# LEMP Stack CentOS

## Update Core System

```
sudo dnf update -y
```

## Install NGINX

```
sudo dnf install nginx -y
```

### Start dan Enable NGINX

```
sudo systemctl start nginx && sudo systemctl enable nginx
```

### Allow servis HTTP dan HTTPS pada firewalld

```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

## Install MariaDB

```
sudo dnf install @mariadb -y
```

### Start dan Enable MariaDB

```
sudo systemctl start mariadb && sudo systemctl enable mariadb
```

### Mysql Secure Install

```
sudo sudo mysql_secure_installation
```

## Install PHP 8.2

### Install Epel dan REMI Repository

```
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

### Enable Module PHP 8.2

```
sudo dnf module enable php:remi-8.2
```

### Install PHP Extensions

```
sudo dnf -y install php php-fpm php-mysqlnd php-opacache php-gd php-xml php-mbstring php-zip
```

### Start dan Enable php-fpm

```
sudo systemctl start php-fpm && sudo systemctl enable php-fpm
```

### Edit ```nano /etc/php-fpm.d/www.conf```

```
user = nginx
group = nginx
listen = /var/run/php-fpm/php-fpm.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

### Reload php-fpm

```
sudo systemctl reload php-fpm 
```

### Membuat direktori root nginx

```
sudo mkdir -p /home/user/public_html
```


### Membuat file index yang berisi info php ```sudo vim /home/user/public_html/index.php```

```
<?php
phpinfo();
?>
```

### Ubah file permissions

```
sudo chown -R nginx:nginx /home/user/public_html
sudo chmod -R 755 /home/user/public_html
```

### Membuat VirtualHost NGINX ```nano /etc/nginx/conf.d/testsite.com.conf```

```
server {
    listen 80;
    server_name  testsite.com www.testsite.com;

    root   /home/user/public_html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;

    location = /50x.html {
        root /home/user/public_html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### Check dan Restart Nginx

```
sudo nginx -t
sudo systemctl restart nginx
```

### Install LetsEncrypt SSL dan enable HTTPS beserta HTTP/2 virtual host
``` 
sudo dnf install certbot python3-certbot-nginx
sudo certbot --nginx
```
