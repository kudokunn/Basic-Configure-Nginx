* Mô hình: Hình 1.PNG
* Trên LB:
* Trỏ 2 tên miền systemtip.tk và tipsystem.tk vào IP public của LB và tạo 2 file cấu hình cho 2 tên miền đó để chạy: 
* Ví dụ: systemtip.tk.conf. Quan trọng phần khai báo loạt proxy.....

      server{
        listen       80;
        server_name systemtip.tk www.systemtip.tk;
        error_log  /var/log/nginx/systemtip.tk.error.log error; 
        access_log  /var/log/nginx/systemtip.tk.access.log  main;

      location /{
        #index index.html index.php;
        #allow 42.114.168.71;
        #deny all;
        proxy_pass http://system; # system: tên của khai báo upstream
        proxy_set_header HOST $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
      }
      location ~ \.php {
        fastcgi_pass unix:/var/run/php_fpm.sock;
        fastcgi_index   index.php;
        include         /etc/nginx/fastcgi_params;
        fastcgi_param   SCRIPT_FILENAME $document_root/$fastcgi_script_name;
      }

      }


* Khai báo upstream khi có request đến 2 tên miền này để đẩy cho BE xử lý. Mỗi upstream là một cụm BE để con LB quản lý 
+ Khai báo phần thêm phần include trỏ vào file upstream hoặc khai báo thằng upstream vào html context: include /etc/nginx/upstream 
khai báo upstream trong html: 

      upstream system {
          server 10.140.0.2:80 max_fails=3 fail_timeout=10s;
          server 10.146.0.3:80 weight=2 max_fails=3 fail_timeout=10s;
      }
+ Note: ở đây ta cấu hình thêm phần thuật toán sử dụng và health check cho 2 con backend. 

Trên 2 BE: bản chất 2 con này chỉ để chứa code web và xử lý request
Cấu hình cho 2 tên miền chạy 2 website giống hệt nhau trên cả 2 BE như cấu hình trong phần vhost: server (port, root, log,  => location /( index, allow, deny, authen...) => location ~/php (....)

