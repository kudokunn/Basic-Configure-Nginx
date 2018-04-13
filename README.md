## Thông tin Nginx
* Nginx là một máy chủ web và cũng có thể được sử dụng làm reserved proxy, load balancer hoặc HTTP cache. 
* Nó được phát hành ngày 04/11/2004 và là một chương trình mã nguồn mở được viêt bằng ngôn ngữ C. Các hệ điều hành có thể dùng Nginx: BSD variants, HP-UX, IBM AIX, Linux, macOS, Solaris, Windows.
* Chức năng cơ bản của nginx:
- Làm reverse proxy có thể che giấu sự tồn tại và các đặc điểm của các servers backend thực sự được dùng.
- Load balancing, có thể chia đều các yêu cầu của các máy khách tới các servers.
- Proxy nginx có thể được dùng như là một application firewall để chống đỡ các cuộc tấn công (như Tấn công từ chối dịch vụ) vào các ứng dụng web.
- Proxy caching nội dung web server gửi và "rót" từ từ đến các máy khách hoạt động chậm. Máy chủ mạng không phải đợi máy khác
- Nhằm giúp giảm tải máy chủ mạng proxy có thể cache các nội dung tĩnh như hình ảnh, tập tin.
## Cài đặt: có thể cài đặt nginx nhanh chóng bằng các lệnh cài từ kho package
Centos: yum install -y nginx
Ubuntu: apt-get install nginx -y
Khởi chạy dịch vụ và enable: systemctl start nginx && systemctl enable nginx
Nhập ip của server cài Nginx. Nếu hiện tittle của Nginx đã cài đặt thành công
## Cấu hình:
### 1. Cấu hình vhost [tại đây](Data/vhost.md)

### 2. Cấu hình SSL [tại đây](Data/SSL.md)

### 3. Cấu hình Load balancing [tại đây](Data/Nginx-LB.md)
