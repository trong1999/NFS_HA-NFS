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
  + **gluster volume create glusterfsvolumne replica 2 storage1.org:/data/glusterfs/glustervolume      storage2.org:/data/glusterfs/glustervolume force**
  + Vậy **/data/glusterfs/glustervolume** là gì ? Nó là folder của volume
- **/data/glusterfs** ở đâu ra ?
  + đầu tiền ta phải add thêm disk thêm sao server đó là **/dev/sdb**
  + **fdisk /dev/sdb**
  + <img src="https://i.imgur.com/8KKP9iP.png">
  + **mkfs.ext4 /dev/sdb1** => format partition
  + <img src="https://i.imgur.com/dJA2DvQ.png">
  + <img src="https://i.imgur.com/ckxu3Qc.png">
  + Sau đó sẽ tạo **/data/glusterfs** và lưu vào **/etc/fstab**
  + <img src="https://i.imgur.com/CsrZ1Df.png">
- Tạo volume 
  + **gluster volume create glusterfsvolumne replica 2 storage1.org:/data/glusterfs/glustervolume storage2.org:/data/glusterfs/glustervolume**
  + <img src="https://i.imgur.com/jJtXFrx.png">
  + Sau đó start và kiểm tra
  + <img src="https://i.imgur.com/p8QJImM.png">
- Default thì gluster cho phép tất cả các kết nối đến volume, ở đây ta sẽ chỉ cho hai server kết nối vào thôi
  + **gluster volume set glusterfsvolumne auth.allow 192.168.5.2,192.168.5.3**
- Check volume info
  + Trên storage1: <img src="https://i.imgur.com/MiF3lQi.png">
  + Trên storage2: <img src="https://i.imgur.com/XuONjId.png">
  
- Cài đặt Glusterfs-client
  + Trước khi ta cài đặt cho client ta cần phải check xem version trên server, mục đích là để cho version ở hai bản tương thích với nhau, điều này rất quan trọng để có mount được.
  + <img src="https://i.imgur.com/OYjOeWx.png">
  + **rpm -i glusterfs-7.3-1.el7.x86_64.rpm**
  + **rpm -i glusterfs-fuse-7.3-1.el7.x86_64.rpm**
  + <img src="https://i.imgur.com/KaxrXec.png">
  + Sau khi cài đặt xong, ta sẽ tạo một folder để mount volume : **mkdir -p /mnt/glusterfsvolumne**
  + Tiếp theo là sẽ mount **mount -t glusterfs storage1.org:/glusterfsvolumne /mnt/glusterfsvolumne**
  + Ở đây ta có thể dùng storage1 hoặc storage2
  + Check mount
  + <img src="https://i.imgur.com/qB01fcQ.png">
- Replication
  + Trên storage1: **mount -t glusterfs storage2.org:/glusterfsvolumne /mnt**
  + Trên storage2: **mount -t glusterfs storage1.org:/glusterfsvolumne /mnt**
  + Các file trên **client** như sau:
  + <img src="https://i.imgur.com/85TvvNc.png">
  + Check xem trên 2 **storage** thế nào 
  + <img src="https://i.imgur.com/HXqCWf1.png">
  + <img src="https://i.imgur.com/HXqCWf1.png">
- **Thử tắt đi storage1.org và thêm vào Glusterfs-client một file xem thế nào** 
  + <img src="https://i.imgur.com/kgMPCYK.png">
  + Ta thấy đã có file **demo.txt** rồi nhé.
  + Đáng lưu ý là trong file **/etc/fstab** ta vẫn không thay đổi gì => điều này có được là vì ở bước trước ta đã tạo thêm một disk để cho cả hai storage sẽ replication dữ liệu với nhau.
  
  
  
  
  
  
