# Bài toán :
- Xây dựng hệ thống NFS có sử dụng High Avaiable
# Solution : 
- Mô hình như sau
  + <img src="https://i.imgur.com/f3k6dAX.jpg">
- Chúng ta sử tạo VIP bằng cách sử dụng keepalived
- Tạo hai con server NFS-clients Primary và Slave
  + Nếu một trong hai con down thì đường truyền vẫn không bị gián đoạn
- Tạo hai con storage để làm NFS-servers
  + Một trong hai con down thì địa chỉ mount sẽ không bị mất vì trên hai con này ta sử dụng glusterfs để replicate volume
# Bắt tay vào làm ngay
- Cấu hình keepalive kết hợp VIP
  + **yum install epel-release -y**
  + **yum update -y**
  + **yum install gcc kernel-headers kernel-devel -y**
  + **yum install keepalived -y**
  + **echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/sysctl.conf** ==> cho phép tạo VIP trên card mạng
  + **sysctl -p** => kiểm tra xem có đúng bằng 1 không
- Sau đó vào file **/etc/keepalive/keepalive.conf** _ để làm gì?
  + Trên con primary
    + <img src="https://i.imgur.com/4bfOfsk.png">
  + Trên con slave 
    + <img src="https://i.imgur.com/NQ2A1ay.png">
  + Trên cả hai con ta start: **systemctl start keepalived.service**
- Sau đó kiểm tra
  + **ip a**
  + <img src="https://i.imgur.com/K0omq1y.png">
  + Tắt server primary xem trên server slave có hiển thị VIP không nhé:
  + <img src="https://i.imgur.com/YEIejF4.png">
 ## Setup nfs lên hai con server
  + **yum install nfs-utils**
  + **systemctl start nfs.service**
- Tạo thư mục /data/nfs được chia sẻ bởi nfs
  + **mkdir /data/nfs**
- Vào file /etc/exports để xác định địa chỉ được export và phân quyền cho địa chỉ đó.
  + **vim /etc/exports**
  + add : **/data/nfs            192.168.5.30(rw,sync,no_root_squash,no_all_squash)**
  + Địa chỉ ip trên là từ đâu, đó là địa chỉ client được mount
  + Ở đây, ip có quyền được read and write, và được chấp nhận cả quyền root và tất cả các quyền khác.
- Sau đó chúng ta sẽ vào firewall để enable service
  + **firewall-cmd --permanent --zone=public --add-service=nfs**
  + **firewall-cmd --reload**
## Setup nfs-client, ta cũng sẽ setup như sau
  + **yum install nfs-utils**
  + **systemctl start nfs.service**
- Việc tạo disk và cấu hình sẽ tương tự với glusterfs
- Sau đó cài đặt **yum install -y pcs fence-agents-all** trên cả hai server
- Tạo authen trên các node cluster
  + **echo "1234" | passwd --stdin hacluster**
  + <img src="https://i.imgur.com/HnuDdv9.png">
  + <img src="https://i.imgur.com/fzZzx4y.png">
### Trên server
- Tạo filesystem recource vì phải cần một storege share
  + **pcs resource create nfsshare Filesystem device=/dev/sdb1  directory=/data/nfs fstype=xfs --group nfsgrp**
- Tạo nfsserver resource
  + **pcs resource create nfsd nfsserver nfs_shared_infodir=/data/nfs/nfsinfo --group nfsgrp**
- Tạo exports resource
  + **pcs resource create nfsroot exportfs clientspec="192.168.5.30/24" options=rw,sync,no_root_squash directory=/data/nfs fsid=0 --group nfsgrp**
- Tạo NFS Addr2 (cho phép địac chỉ ip card mạng ảo NFS)
  + **pcs resource create nfsip IPaddr2 ip=192.168.5.10 cidr_netmask=32 --group nfsgrp**
### Trên client

- Tạo thư mục mount trên **client**
  + **mkdir /mnt/nfsshare**
- Sau đó vào file /etc/fstab để add địa chỉ mount
  + **192.168.5.10:/    /mnt/nfsshare   nfs defaults 0 0**
  + Địa chỉ server này là VIP, vì client sẽ làm việc trực tiếp với VIP.
  + Sau đó save và exit
  + Cuối cùng thì active **mount -a**
- Tạo file trong thư mục mount **touch /mnt/nfsshare/data/nfs/demo5**
  + <img src="https://i.imgur.com/uZWjvxX.png">
- Qua server kiểm tra
  + <img src="https://i.imgur.com/R629JDm.png">
- Bây giờ ta sẽ tắt server primary đi để xem hệ thống sẽ hoạt động thế nào ?
  + Thì VIP sẽ nhảy sang con slave => vì thế nó sẽ không ảnh hưởng gì đến việc mount từ client đến server.
  + Vào check **/data/nfs**
  + <img src="https://i.imgur.com/lTtHYJA.png">
- Nếu trên client bạn gặp lỗi này
  + <img src="https://i.imgur.com/I60jyMv.png">
  + Ta sẽ fix như sau
  + <img src="https://i.imgur.com/HgquLcf.png">
#### Vậy là đã xong
