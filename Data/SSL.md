Điều kiện: Cần tên miền trỏ DNS về IP của server để dùng SSL

## BƯỚC 1. CÀI ĐẶT LET’S ENCRYPT CLIENT

* Bước đầu tiên để có thể lấy được SSL miễn phí từ Let’s Encrypt đó là cài đặt phần mềm letsencrypt trên máy chủ của bạn. Hiện tại, Let’s Encrypt chỉ có thể được cài đặt thông qua Repository từ Github nhưng trong tương lai có thể Let’s Encrypt sẽ được cài đặt thông qua Package Manager.

* Cài đặt Git

* Cập nhật các gói phần mềm của hệ thống:

* Debian/Ubuntu :

        sudo apt-get update 
        sudo apt-get -y install git
        
* CentOS

        yum -y update
        yum -y install git

* Sao chép mã nguồn Let’s Encrypt vào máy chủ của bạn, chúng ta sẽ lưu trong thư mục opt

        git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt

## BƯỚC 2. LẤY CHỨNG CHỈ SSL MIỄN PHÍ TỪ LET’S ENCRYPT

* Dành cho bạn đang dùng NGINX:

* Có nhiều cách để lấy một chứng chỉ tuy nhiên chúng ta sẽ lấy nó bằng plugin Standalone.

* Đảm bảo cổng 80 đang mở và không sử dụng

* Plugin Standalone cung cấp cho bạn một công cụ để lấy một chứng chỉ SSL từ Let’s Encrypt. Nó chạy như một webserver nhỏ thông qua port 80 để Let’s Encrypt CA có thể kết nối và xác mình server của bạn. Nó sẽ chạy như một webserver bình thường do đó nếu port 80 cần được mở và không sử dụng. Nếu bạn đã cài NGINX thì cần tắt nó đi tạm thời để giải phóng cổng 80. Chạy lệnh sau để tắt NGINX:

* Tắt NGINX (Debian/Ubuntu/CentOS)

      service nginx stop

* Bây giờ bạn đã có thể sử dụng plugin Standalone được rồi.

* Nếu bạn đang dùng CentOS thì cần chạy các lệnh sau mới có thể dùng được plugin Standalone, Debian/Ubuntu thì bỏ qua nhé lên thẳng bước 3.

* Đầu tiên bạn cần kiểm tra phiên bản Python trên VPS CentOS: python –version
* Nếu kết quả là phiên bản lớn hơn hoặc bằng 2.7 thì bạn có thể bỏ qua còn nếu nhỏ hơn 2.7 thì chạy các lệnh sau để cập nhật Python lên phiên bản 2.7, chạy từng cái một nhé, hơi lâu khoảng 5”.

        yum -y update

        yum groupinstall "Development tools"

        yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel

        cd /opt

        wget --no-check-certificate https://www.python.org/ftp/python/2.7.6/Python-2.7.6.tar.xz

        tar xf Python-2.7.6.tar.xz 

        cd Python-2.7.6

        ./configure --prefix=/usr/local

        make && sudo make altinstall

* Sau đó kiểm tra xem đã là phiên bản 2.7 chưa bằng lệnh /usr/local/bin/python2.7 –version . Nếu đã lên 2.7 thì đi tiếp bước 3.

## BƯỚC 3. CHẠY LET’S ENCRYPT

Trước khi sử dụng Let’s Encrypt, bạn cần duyệt tới thư mục chứa Let’s Encrypt Client mà đã Clone lúc trước từ Github.
      cd /opt/letsencrypt

Bây giờ bạn có thể sử dụng plugin Standalone bằng cách chạy lệnh:

    ./letsencrypt-auto certonly --standalone

Có thể nó sẽ hỏi bạn nhập mật khẩu của user mà bạn đang đăng nhập vào SSH. Nếu hỏi thì nhập vào.

Chờ một lúc cho Let’s Encrypt khởi chạy, bạn sẽ được hỏi một số thông tin, nhập chính xác nhé:

Nhập địa chỉ email để phòng khi mất private key còn có chỗ mà lấy lại:
alt text

Sau đó phải chấp nhận điều khoản của Let’s Encrypt. Chọn Agree
Tiếp tục, bạn cần nhập tên miền mà bạn muốn dùng SSL. Lưu ý nếu bạn muốn dùng nhiều SSL ví dụ cho cả nodejs.vn và www.nodejs.vn thì phải nhập cả hai vào, cách nhau bởi dấu cách.
alt text

Nếu thành công, bạn sẽ nhận được thông báo như thế này:

IMPORTANT NOTES:

    – If you lose your account credentials, you can recover through e-mails sent to contact@crazytut.com

    – Congratulations! Your certificate and chain have been saved at /etc/letsencrypt/live/example.com/fullchain.pem. Your cert will expire on 2016-03-15. To obtain a new version of the certificate in the future, simply run Let’s Encrypt again.

    – Your account credentials have been saved in your Let’s Encrypt configuration directory at /etc/letsencrypt. You should make a secure backup of this folder now. This configuration directory will also contain certificates and private keys obtained by Let’s Encrypt so making regular backups of this folder is ideal.

    – If like Let’s Encrypt, please consider supporting our work by: Donating to ISRG / Let’s Encrypt: https://letsencrypt.org/donate Donating to EFF: https://eff.org/donate-le
    Bạn nhớ chú ý dòng chữ màu đỏ ở trên, đó là thư mục chưa file SSL của bạn và ngày hết hạn chứng chỉ.

File SSL gồm có:

      cert.pem: Chứng chỉ SSL cho tên miền của bạn
      chain.pem: Chứng chỉ của Let’s Encrypt
      fullchain.pem: cert.pem và chain.pem được gộp lại thành một
      privkey.pem: Private key của chứng chỉ
      Lưu ý là thư mục lưu các file SSL của bạn tại đường dẫn /etc/letencrypt/live/domain.com/
Bước 4: Cấu hình SSL và restart nginx:
Sử dụng file config của bài tạo vhost để cấu hình luôn ssl cho hai web site đó như sau

           server { 
           listen 80; 
           server_name systemtip.tk www.systemtip.tk;
           location / { 
             return 301 https://systemtip.tk$request_uri; # khogn co $reques_uri thi khi nhap https://systemtip.tk/private se ve systemtip.tk
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
           root /data/systemtip.tk; 
           index index.php index.html index.htm; 
           error_log  /var/log/nginx/systemtip.tk/systemtip.tk_error.log error;
           access_log  /var/log/nginx/systemtip.tk/systemtip.tk_access.log  main;
       
           location ~ \.php {
                fastcgi_pass unix:/var/run/php_fpm.sock;
                fastcgi_index   index.php;
                include         /etc/nginx/fastcgi_params;
                fastcgi_param   SCRIPT_FILENAME $document_root/$fastcgi_script_name; 
           }

     location /{
        index index.html index.php;
        #allow 42.114.168.71;
        #deny all;
     }

     location /private{
        index index.html index.php;
        auth_basic "private site";#linh dong theo location
        auth_basic_user_file /etc/nginx/.htpasswd; #linh dong theo location

     }

}

