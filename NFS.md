## NFS là gì: Network File System
- Dùng để chia sẻ dữ liệu files trên hệ thống, sử dụng TCP và UDP để vận chuyển.
- Cho phép truy cập và lưu trữ dữ liệu trên nhiều disk và thư mục.
- Dùng để phân tán dữ liệu từ xa nhưng như việc chúng ta sử dụng chúng ở mạng cục bộ.
- NFS sử dụng mô hình client-server
  + Server: export directories
  + Client: mount directories từ server.
- NFS là một giao thức cấp độ ứng dụng
### Control NFS
- /etc/export : file configuare
- /ect/hosts : file chưa tên và địa chỉ tất cả clients nhận service.
### Exports filesystem
- Trước khi setup NFS server thì phải kiểm tra xem các hosts được mount đến filesystem
- Sau đó edit file /etc/export, ở đây chúng ta sẽ xác định các host được mount và phân quyền cho chúng.
### Mount filesystem 
- Chúng ta có thể mount bằng câu lệnh
  + **mount -t nfs nfs-domain.com:/share /share** => thư mục được share và thư mục nhận share.
- Và config trong file /etc/fstab
  + **192.168.5.10:/share /share  nfs4  rw,sync**
