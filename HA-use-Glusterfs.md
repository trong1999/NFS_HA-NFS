## Glusterfs là gì?
- Glusterfs được tạo ra để giải quyết bài toán ngày càng có số lượng file lưu trữ rất lớn
- Thêm storage dễ dàng hơn.
- Có hiệu năng sử dụng cao hơn (HA)
- Cần lưu ý một số yếu tố sau:
  + node: servers được cài đặt gluster
  + brick: folder; mount point; file system trên một node dùng để chia sẻ với các node khác và một node có thể có nhiều brick;
  brick được dùng để gán dữ liệu vào volume
- volume: là một khối chứa nhiều brick
- client: là các máy kết nối đến server, có thể là gluster-client, nfs-client
### Có các dạng volume sau 
- Distributed Volume
  + Ví dụ có 4 file thì: file1 và file2 được lưu ở brickA; file3 và file4 được lưu ở brickB
  + Dữ liệu sẽ bị mất khi 1 trong 2 brick bị hỏng
- Replicated Volume
  + Ví dụ có 2 file thì trên cả brickA và brickB đều lưu 2 file này
  + Có tính dự phòng cao
- Striped Volume
  + Ví dụ có 1 file thì nó sẽ được chia nhỏ ra và lưu trữ trên các brick khác nhau
  + Một brick bị hỏng thì file cũng không hoàn chỉnh
- Distributed Replicated Volume
  + là sự kết hợp giữa kỹ thuật 1 (Distributed Volume) và kỹ thuật 2 (Replicated Volume). 
  Các file được phân tán trên các Brick trong cùng một Volume
  + Khi một volume bị hỏng thì dữ liệu cũng sẽ bị mất
## Tiếp theo sẽ đi đến việc cấu hình Glusterfs
- Đầu tiên ta sẽ add tất cả các host vào file **/etc/hosts** => trên cả gluster-server và gluster-client
  + <img src="https://i.imgur.com/kcLa5Ik.png">
- Sau đó sẽ cài các packages rồi mới đến cài glusterfs-server.
  + **yum install wget**
  + **yum install centos-release-gluster -y**
  + **yum install epel-release -y**
  + **yum install glusterfs-server -y**
- Tiếp theo là mở port trên firewall
  + **firewall-cmd --zone=public --add-port=24007-24008/tcp --permanent**
  + **firewall-cmd --zone=public --add-port=24009/tcp --permanent**
  + **firewall-cmd --zone=public --add-service=nfs --add-service=samba --add-service=samba-client --permanent**
  + **firewall-cmd --zone=public --add-port=111/tcp --add-port=139/tcp --add-port=445/tcp** 
  + **firewall-cmd --zone=public --add-port=965/tcp --add-port=2049/tcp** 
  + **firewall-cmd --zone=public --add-port=38465-38469/tcp --add-port=631/tcp --add-port=111/udp** 
  + **firewall-cmd --zone=public --add-port=963/udp --add-port=49152-49251/tcp --permanent**
- Distribute Volume Setup
  + Trên storage nào cũng được: **gluster peer probe storage2.org**
  + <img src="https://i.imgur.com/HSHx7Bz.png">
- Tạo volume tên share trên 2 storage 
  + **gluster volume create share replica 2 transport tcp storage1.org:/data storage2.org:/data force**
- Default thì gluster cho phép tất cả các kết nối đến volume, ở đây ta sẽ chỉ cho hai server kết nối vào thôi
  + **gluster volume set Host auth.allow 192.168.5.2,192.168.5.3**
- Check volume info
  + Trên storage1: <img src="https://i.imgur.com/QIqIqMu.png">
  + Trên storage2: <img src="https://i.imgur.com/tMcVch0.png">
  
- Cài đặt Glusterfs-client
  + **yum install glusterfs-client**
- Tạo thư mục **mkdir /share**
- Sau đó vào **/etc/fstab** để mount 
  + ****
  
  
  
  
  
  
  
  
