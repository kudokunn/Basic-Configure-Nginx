* Đ/N: Vhost (virtual hosts) là một host ảo chạy trên 1 server, nhiều vhost có thể chạy trên một server vật lý, điều này cho phép một server vật lý chạy nhiều website.
* Cấu hình: File cấu hình của nginx trong thư mục /etc/nginx/. Và cài đặt vhosts nên đặt mỗi tên miền của web site là một file cấu hình trong thư mục /etc/nginx/conf.d/. Ví dụ: ở đây sẽ 2 tên miền systemtip.tk và tipsystem.tk cho cùng một IP public của server nên ta sẽ có hai file cấu hình .conf với tên của web: systemtip.tk.conf và tipsystem.tk.conf.
P/s: tên miền .tk là tên miền miễn phí, nhưng phải đăng trí trỏ về 1 IP public. Vậy có thể đăng ký sử dụng các VPS Google Cloud miễn phí một năm để có được server có IP public để trỏ tên miền về. Tham khảo tại đây: http://svuit.vn/news/2017/03/21/tao-vps-google-mien-phi-1-nam-300/

Nội dung cơ bản như sau:
### Doamin: systemtip.tk.conf    
    server{
        listen       80;
        server_name systemtip.tk www.systemtip.tk;
        root /data/systemtip.tk;
        error_log  /var/log/nginx/systemtip.tk/systemtip.tk_error.log error;
        access_log  /var/log/nginx/systemtip.tk/systemtip.tk_access.log  main;

     location /{
        index index.html index.php;
        #allow 42.114.168.71;
        #deny all;           
     }

     location /private{
        index index.html index.php;
        #auth_basic "private site";
        #auth_basic_user_file /etc/nginx/.htpasswd; 

     }

    location ~ \.php {
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index   index.php;
        include         /etc/nginx/fastcgi_params;
        fastcgi_param   SCRIPT_FILENAME $document_root/$fastcgi_script_name;
    }
    }

### Doamin: tipsystem.tk.conf    

      server{
        listen       80;
        server_name tipsystem.tk www.tipsystem.tk;
        root /data/tipsystem.tk;
        error_log  /var/log/nginx/tipsystem.tk/tipsystem.tk_error.log error;
        access_log  /var/log/nginx/tipsystem.tk/tipsystem.tk_access.log  main;

     location /{
        index index.html index.php;
        #allow 42.114.168.71;
        #deny all;           
     }

     location /private{
        index index.html index.php;
        #auth_basic "private site";#linh dong theo location
        #auth_basic_user_file /etc/nginx/.htpasswd; #linh dong theo location

     }

    location ~ \.php {
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index   index.php;
        include         /etc/nginx/fastcgi_params;
        fastcgi_param   SCRIPT_FILENAME $document_root/$fastcgi_script_name;
    }
    }

* Thêm đường dẫn thư mục root /data/systemtip.tk chứa code và đường dẫn của file log /var/log/nginx/systemtip.tk/. Ở đây tạo một file php đơn giản check info php.
* Bản chất hoạt động: Khi client request đến server, server biết được trường hosts có chứa tên website, từ đó server sẽ tìm tên miền đó trong tất cả các file cấu hình có nameserver trùng với tên miền web đó. Nếu trùng thì truy vấn thư mục root chứa code web và tạo các file log access và error đối với tên miền đó
* Tạo xác thực cơ bản: tạo file .htpasswd bằng cách: htpasswd -c pwfile username để lưu mật khẩu người dùng và khai báo directive auth_basic và auth_basic_user_file như sau:
        
        #auth_basic "private site";#linh dong theo location
        #auth_basic_user_file /etc/nginx/.htpasswd; #linh dong theo location

## Thêm: Cài đặt php-fpm để nginx có thể biên dịch file php
Bản thân nginx không hỗ trợ xử lý php. Việc xử lý php sẽ do một service khác đảm nhận. Nginx sẽ forward request đến service này và nhận kết quả về. Service này là php-fpm.
* B1: Cài đặt: yum instal php-fpm -y
* B2: Chỉnh sửa file

    vim /etc/php-fpm.conf

      include=/etc/php-fpm.d/*.conf
      [global]
      pid = /var/run/php-fpm/php-fpm.pid
      error_log = /var/log/php-fpm/error.log
      log_level = warning
      daemonize = yes
      
    vim /etc/php-fpm.d/www.conf
    
        [www]
        listen = /var/run/php_fpm.sock
        listen.allowed_clients = 127.0.0.1
        listen.owner = nginx
        listen.group = nginx
        listen.mode = 0660
        user = nginx
        group = nginx
        pm = dynamic
        pm.max_children = 50
        pm.start_servers = 5
        pm.min_spare_servers = 5
        pm.max_spare_servers = 35
        slowlog = /var/log/php-fpm/www-slow.log
        php_admin_value[error_log] = /var/log/php-fpm/www-error.log
        php_admin_flag[log_errors] = on
        php_value[session.save_handler] = files
        php_value[session.save_path] = /var/lib/php/session
        Tôi chỉ sửa ba chỗ so với config mặc định của www.conf
        listen = /tmp/php_fpm.sock
        listen.owner = nginx
        listen.group = nginx
