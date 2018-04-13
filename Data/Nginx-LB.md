## Mô hình:
![](image/1.PNG)

### Trên LB:

* Trỏ 2 tên miền systemtip.tk và tipsystem.tk vào IP public của LB và tạo 2 file cấu hình cho 2 website trong thư mục conf.d/ cho 2 tên miền đó để chạy: 
* Ví dụ: 
File systemtip.tk.conf

      server {
         listen 80;
         server_name systemtip.tk www.systemtip.tk;
         location / {
           return 301 https://systemtip.tk$request_uri; # khogn co $reques_uri thi khi nhap https://systemtip.tk/private se ve                       systemtip.tk
         }
       }
       server {
         listen 443 ssl;
         server_name systemtip.tk www.systemtip.tk;
         ssl_certificate /etc/letsencrypt/live/systemtip.tk/fullchain.pem;
         ssl_certificate_key /etc/letsencrypt/live/systemtip.tk/privkey.pem;
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_prefer_server_ciphers on;
         ssl_ciphers AES256+EECDH:AES256+EDH:!aNULL;
         error_log  /var/log/nginx/systemtip.tk/systemtip.tk.error.log error;
         access_log  /var/log/nginx/systemtip.tk/systemtip.tk.access.log  main;
         
       location /{
        #allow 42.114.168.71;
        #deny all;
        proxy_pass http://system; #phải phù hợp với tên được định nghĩa trong upstream
        proxy_next_upstream error http_403;
        proxy_set_header HOST $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

       }

        location /private{
              index index.html index.php;
              auth_basic "private site";#linh dong theo location
              auth_basic_user_file /etc/nginx/.htpasswd; #linh dong theo location

                 }

            }

File tipsystem.tk.conf

      server {
         listen 80;
         server_name tipsystem.tk www.tipsystem.tk;
         location / {
           return 301 https://tipsystem.tk$request_uri; # khogn co $reques_uri thi khi nhap https://tipsystem.tk/private se ve                  tipsystem.tk
         }
      }
      server {
         listen 443 ssl;
         server_name tipsystem.tk www.tipsystem.tk;
         ssl_certificate /etc/letsencrypt/live/tipsystem.tk/fullchain.pem;
         ssl_certificate_key /etc/letsencrypt/live/tipsystem.tk/privkey.pem;
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_prefer_server_ciphers on;
         ssl_ciphers AES256+EECDH:AES256+EDH:!aNULL;
         error_log  /var/log/nginx/tipsystem.tk/tipsystem.tk.error.log error;
         access_log  /var/log/nginx/tipsystem.tk/tipsystem.tk.access.log  main;

     location /{
        #allow 42.114.168.71;
        #deny all;
        proxy_pass http://system; # phải phù hợp với tên được định nghĩa trong upstream
        proxy_set_header HOST $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

     }

     location /private{
        index index.html index.php; # tren LB
        auth_basic "private site";#linh dong theo location
        auth_basic_user_file /etc/nginx/.htpasswd; #linh dong theo location

       }
       }

* Khai báo upstream để khi có request đến 2 tên miền này để đẩy cho BE xử lý. Mỗi upstream là một cụm BE của con LB quản lý.
+ Khai báo phần thêm phần include trỏ vào file upstream hoặc khai báo thằng upstream vào html block: 
html {
....
include /etc/nginx/upstream;
......
}
Tạo file với tên là upstream: vim /etc/nginx/upstream.

      upstream system {
          server 10.140.0.2:80 max_fails=3 fail_timeout=10s;
          server 10.146.0.3:80 weight=2 max_fails=3 fail_timeout=10s;
      }
* Note: 
- Ở đây cấu hình thêm phần thuật toán sử dụng và health check cho 2 con backend. Ở đây theo thuật toán cân bằng tải trọng với tham số weight định nghĩa. Backend 2 có weight=2 và Backend 1 bằng 1. Vậy khi có 3 request đến thì 1 request ban đầu vào backend 1, 2 request tiếp vào Backend 2.
- Cấu hình health check: với hai tham số là max_fails=3 fail_timeout=10s. Ví dụ nếu BE2 2 chết thì BE không nhận được đợi phản backend 2. Nó sẽ đếm thời gian timeout, quá một lần thời gian timeout là 1 lần fails. Vậy mất 30s để 3 lần fails. Và khi đó request lại bị đẩy lại backend 2 vì tham số weight cấu hình, vậy mất thêm 30s nữa cho lần 2 => Khoảng 1p sau thì LB mới đẩy lại request cho BE1 xử lý.
- Trường hợp BE2 vấn sống nhưng code web bị hỏng báo lỗi 403 hay lỗi mã lỗi gì đấy, phản hồi từ BE2 về LB vẫn con nhưng đó là một respone lỗi, vậy để LB phân biệt được các respone bị lỗi và respone bình thường thì chỉ cần cấu hình thêm tham số: "proxy_next_upstream error http_403;" trong phần dưới để LB nhận biết respone lỗi và đẩy request đến backend tiếp

      location /{
        #allow 42.114.168.71;
        #deny all;
        proxy_pass http://system; # phải phù hợp với tên được định nghĩa trong upstream
        proxy_set_header HOST $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

             }

# Trên 2 Backend: 
Bản chất 2 con này chỉ để chứa code web và xử lý request
Cấu hình cho 2 tên miền chạy 2 website giống hệt nhau trên cả 2 Backend y như cấu hình trong phần vhost
