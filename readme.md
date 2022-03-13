# Some Useful Commands That Iam Using From Time To Time 

### Install mysql

```jsx
#Install mysql 
sudo apt install mysql-server
sudo mysql_secure_installation
- N 
set any password 
- Y
- N
- Y
- Y
-------
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'STRONG_PASSWORD_HERE';
FLUSH PRIVILEGES;
```

### Import and Export MySql

```powershell
Export : mysqldump -u root -p db > db.sql #one db
             : mysqldump -u root -p --all-databases > alldb.sql # all dbs
             : for DB in $(mysql -e 'show databases' -s --skip-column-names); do 
                    mysqldump -u root -p  $DB > "$DB.sql";
                  done

Import : mysql -u root -p db < db.sql
```

### Change Mysql Password

```jsx
# option1
$ sudo mysql -u root # I had to use "sudo" since is new installation

mysql> USE mysql;
mysql> UPDATE user SET plugin='mysql_native_password' WHERE User='root';
mysql> FLUSH PRIVILEGES;
mysql> exit;

$ sudo service mysql restart

#option2
$ sudo mysql -u root # I had to use "sudo" since is new installation

mysql> USE mysql;
mysql> CREATE USER 'YOUR_SYSTEM_USER'@'localhost' IDENTIFIED BY 'YOUR_PASSWD';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'YOUR_SYSTEM_USER'@'localhost';
mysql> UPDATE user SET plugin='auth_socket' WHERE User='YOUR_SYSTEM_USER';
mysql> FLUSH PRIVILEGES;
mysql> exit;

$ sudo service mysql restart
```

### Transfer database from server to another

```jsx
mysqldump --all-databases -uuser -ppassword -hserver -P3306 --force | mysql -hremoteserver -uremoteuser -premotepassword -P3306 --force
```

### SSL Certificate

```jsx
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
sudo certbot certonly --nginx
OR
certbot certonly --noninteractive --agree-tos --cert-name SITE_TLD -d SITE_TLD -d www.SITE_TLD --register-unsafely-without-email --webroot -w /var/www/html/SITE_TLD
```

### Crontab Usefull Commands

```jsx
* * * * * cd /var/www/html/example.com/ && git reset --hard HEAD && git clean -f -d && git pull origin master --allow-unrelated-histories
* * * * * cd /var/www/html/blog.example.com/wp-content/themes/blog/ && git reset --hard HEAD && git clean -f -d && git pull origin master --allow-unrelated-histories
* * * * * cd /var/www/html/example.com && php artisan queue:restart && php artisan queue:work --queue=high,medium,default,low >> /dev/null 2>&1
* * * * * cd /var/www/html/example.com && php artisan schedule:run >> /dev/null 2>&1
* * * * * cd /var/www/html/example.com/storage/logs && chmod -R 777 *
```

### Useful Headers

```jsx
<link rel="stylesheet" type="text/css" href="https://nafezly.com/css/cust-fonts.css">
<link rel="stylesheet" type="text/css" href="https://nafezly.com/css/responsive-font.css">
<link rel="stylesheet" type="text/css" href="https://nafezly.com/css/fontawsome.min.css">

<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css">

<script src="https://unpkg.com/vue@next"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"></script>
```

### Nginx Optimized Config file ( /etc/nginx/nginx.conf )

```powershell
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
#include /etc/nginx/conf.d/*;
events {
    worker_connections 768;
    # multi_accept on;
}

http {
#   access_by_lua_file anti_ddos_challenge.lua;
    #client_max_body_size = 250M;
    ##
    # Basic Settings
    ##
    client_max_body_size 250M;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;

     gzip_vary on;
     gzip_proxied any;
     gzip_comp_level 6;
     gzip_min_length    256;
     gzip_buffers 16 8k;
     gzip_http_version 1.1;
     gzip_types text/plain text/css application/javascript application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript image/svg+xml image/svg image/jpg image/png;

    ##
    # Virtual Host Configs
    ##

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

#mail {
#   # See sample authentication script at:
#   # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#   # auth_http localhost/auth.php;
#   # pop3_capabilities "TOP" "USER";
#   # imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#   server {
#       listen     localhost:110;
#       protocol   pop3;
#       proxy      on;
#   }
# 
#   server {
#       listen     localhost:143;
#       protocol   imap;
#       proxy      on;
#   }
#}
```

### Default Nginx File

```jsx
server {
    listen 80;
    listen [::]:80;

    root /var/www/html/first-project/public;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name example.com;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }

    location ~ /\.ht {
            deny all;
    }
}
```

### Nginx File After SSL

```jsx
server {
    listen 80;
    access_log off;
    root /var/www/html/example.com/public;
    index index.php index.html index.htm index.nginx-debian.html;
    client_max_body_size 1000M;
    fastcgi_read_timeout 8600;
    proxy_cache_valid 200 365d;
    location ~ \.(env|log|htaccess)$ {
        deny all;
    }
    location ~*\.(?:js|jpg|jpeg|gif|png|css|tgz|gz|rar|bz2|doc|pdf|ppt|tar|wav|bmp|rtf|swf|ico|flv|txt|woff|woff2|svg|mp3|jpe?g,eot|ttf|svg)$ {
        access_log off;
        expires 360d;
        add_header Access-Control-Allow-Origin *;
    add_header Pragma public;
        add_header Cache-Control "public";
        add_header Vary Accept-Encoding; 
        try_files $uri $uri/ /index.php?$query_string;
    }
    location / {
    add_header Access-Control-Allow-Origin *;
        if ($host ~* ^(www)) {
          rewrite ^/(.*)$ https://example.com/$1 permanent;
        }
                if ($scheme = http) {
                        return 301 https://example.com$request_uri;
                }
        try_files $uri $uri/ /index.php?$query_string;
    access_log off;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }
   listen 443 ssl; # managed by Certbot
   server_name example.com www.example.com;
   ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
   ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
   include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
   ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

### Install Nginx

```powershell
sudo apt update
sudo apt install nginx
sudo ufw app list
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
sudo ufw status
```

### Install Laravel Extensions

```powershell
sudo add-apt-repository universe
sudo apt install php openssl php-fpm php-common php-curl php-json php-mbstring php-mysql php-xml php-zip php-gd php-cli
```

### Setting Up Firewall

```powershell
sudo ufw app list
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

### Install New Site NGINX one line

```powershell
mkdir /var/www/html/example.com ; mkdir /var/www/html/example.com/public ; echo "<?php php_info() ?>" ; rm -rf /etc/nginx/sites-available/example.com ; rm -rf  /etc/nginx/sites-enabled/example.com ; echo "server {
    listen 80;

    root /var/www/html/example.com/public;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name www.example.com example.com;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }

    location ~ /\.ht {
            deny all;
    }
}" > /etc/nginx/sites-available/example.com ; ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/  ; certbot certonly --noninteractive --agree-tos --cert-name example.com -d example.com -d www.example.com --register-unsafely-without-email --webroot -w /var/www/html/example.com  ; rm -rf /etc/nginx/nginx.conf ;  echo "user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
#include /etc/nginx/conf.d/*;
events {
    worker_connections 768;
    # multi_accept on;
}
http {
#   access_by_lua_file anti_ddos_challenge.lua;
    #client_max_body_size = 250M;
    ##
    # Basic Settings
    ##
    client_max_body_size 250M;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;

     gzip_vary on;
     gzip_proxied any;
     gzip_comp_level 6;
     gzip_min_length    256;
     gzip_buffers 16 8k;
     gzip_http_version 1.1;
     gzip_types text/plain text/css application/javascript application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript image/svg+xml image/svg image/jpg image/png;

    ##
    # Virtual Host Configs
    ##

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

#mail {
#   # See sample authentication script at:
#   # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#   # auth_http localhost/auth.php;
#   # pop3_capabilities 'TOP' 'USER';
#   # imap_capabilities 'IMAP4rev1' 'UIDPLUS';
# 
#   server {
#       listen     localhost:110;
#       protocol   pop3;
#       proxy      on;
#   }
# 
#   server {
#       listen     localhost:143;
#       protocol   imap;
#       proxy      on;
#   }
#}" > /etc/nginx/nginx.conf  ; rm -rf /etc/nginx/sites-available/example.com ;
rm -rf /etc/nginx/sites-enabled/example.com  ; echo 'server {
    listen 80;
    listen 443 ssl; # managed by Certbot
    server_name example.com www.example.com;
    access_log off;
    root /var/www/html/example.com/public;
    index index.php index.html index.htm index.nginx-debian.html;
    client_max_body_size 1000M;
    fastcgi_read_timeout 8600;
    proxy_cache_valid 200 365d;
    location ~ \.(env|log|htaccess)$ {
        deny all;
    }
    location ~*\.(?:js|jpg|jpeg|gif|png|css|tgz|gz|rar|bz2|doc|pdf|ppt|tar|wav|bmp|rtf|swf|ico|flv|txt|woff|woff2|svg|mp3|jpe?g,eot|ttf|svg)$ {
        access_log off;
        expires 360d;
        add_header Access-Control-Allow-Origin *;
    add_header Pragma public;
        add_header Cache-Control "public";
        add_header Vary Accept-Encoding; 
        try_files $uri $uri/ /index.php?$query_string;
    }
    location / {
    add_header Access-Control-Allow-Origin *;
        if ($host ~* ^(www)) {
          rewrite ^/(.*)$ https://example.com/$1 permanent;
        }
        if ($scheme != "https") {
            rewrite ^ https://$host$uri permanent;
        }
        try_files $uri $uri/ /index.php?$query_string;
    access_log off;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    } 
   ssl_certificate /etc/letsencrypt/live/example.com-0001/fullchain.pem; # managed by Certbot
   ssl_certificate_key /etc/letsencrypt/live/example.com-0001/privkey.pem; # managed by Certbot
   include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
   ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}' > /etc/nginx/sites-available/example.com ; rm -rf /etc/nginx/sites-enabled/example.com ; ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/ ; service nginx restart
```

### Install Nginx & PHP & MySql & laravel Extensions ⇒ ONE LINE

```powershell
sudo apt update ; sudo apt install nginx ; sudo ufw allow 'Nginx HTTP' ; sudo ufw allow 'Nginx HTTPS' ; sudo ufw allow OpenSSH ; sudo ufw enable ; sudo add-apt-repository universe ; sudo apt install php openssl php-fpm php-common php-curl php-json php-mbstring php-mysql php-xml php-zip php-gd php-cli ; sudo apt install mysql-server 
```

### run laravel application

```powershell
mkdir example.com ; cd example.com ; sudo apt-get install git ; git init ; git remote add origin [] ; git pull origin master ;  cp .env.example .env ;  composer install  ;  php artisan key:generate ; 
- after setting up database 
php artisan migrate --seed ; chmod 777 -R storage 
```

### List dir content with size

```powershell
du -sh * | sort -hr
```

### Close Process On Port Linux

```powershell
sudo kill -9 $(sudo lsof -t -i:9001)
```

### Alert Before Remove

```powershell
var result = confirm('هل أنت متأكد من عملية الحذف ؟');
if (result) {
    //Logic to delete the item
}
```

# Usefull Packages

[https://github.com/spatie/browsershot](https://github.com/spatie/browsershot)
