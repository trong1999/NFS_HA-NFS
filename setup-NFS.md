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
