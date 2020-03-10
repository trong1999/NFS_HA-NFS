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
  + <img src="https://i.imgur.com/eqfocUM.png">
  + Tắt server primary xem trên server slave có hiển thị VIP không nhé:
  + <img src="https://i.imgur.com/WyIK0dt.png">
- cài đặt nfs lên hai con server
  + **yum install nfs-utils**
  + **systemctl start nfs.service**
- Tạo thư mục /share được chia sẻ bởi nfs
  + **mkdir /share**
- Vào file /etc/exports để xác định địa chỉ được export và phân quyền cho địa chỉ đó.
  + **vim /etc/exports**
  + add : **/share            192.168.5.10(rw,sync,no_root_squash,no_all_squash)**
  + Địa chỉ ip trên là từ đâu, đó là địa chỉ client được mount
  + Ở đây, ip có quyền được read and write, và được chấp nhận cả quyền root và tất cả các quyền khác.
- Sau đó chúng ta sẽ vào firewall để enable service
  + **firewall-cmd --permanent --zone=public --add-service=nfs**
  + **firewall-cmd --reload**
- Trên nfs-client, ta cũng sẽ setup như sau
  + **yum install nfs-utils**
  + **systemctl start nfs.service**
- Tạo thư mục mount
  + **mkdir /share**
- Sau đó vào file /etc/fstab để add địa chỉ mount
  + **192.168.5.20:/share    /share   nfs defaults 0 0**
  + Sau đó save và exit
  + Cuối cùng thì active **mount -a**
