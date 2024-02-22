[https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/](https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/)
### nginx直接做为web服务器
centos 6.9 nginx1.12.x php7.3 wordpress5.02 mysql5.7.x
部署成功后，为了美观在设置-->固定链接-->自定义结构 /%category%/%post_id%/
[http://webqa.lwork.com/broker-work-price/](http://webqa.lwork.com/broker-work-price/)
# wordpress 修改上传文件限制大小
vim /etc/php.ini
```
cgi.fix_pathinfo = 0 修改
### wordpress 修改上传文件限制大小，有可能nginx那边还有限制
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300 #可选
```
nginx的配置
```server {
    listen       80;
    server_name  webqa.lwork.com;
    access_log /var/log/nginx/webqa.lwork.com.log main;

    root  /home/www_lwork_com_qa;
    index  index.php ;

    location / {
        try_files $uri $uri/ /index.php?$args;
}

    location ~* \.php$ {
    include fastcgi.conf;
    fastcgi_intercept_errors on;
    fastcgi_pass    127.0.0.1:9000;
}
}
```
## 安装插件
```
放置的位置在下面代码之后：
if ( !defined('ABSPATH') )
        define('ABSPATH', dirname(__FILE__) . '/');

define('WP_TEMP_DIR', ABSPATH.'wp-content/tmp');
define("FS_METHOD", "direct");  
define("FS_CHMOD_DIR", 0777);  
define("FS_CHMOD_FILE", 0777); 
```
### **下载失败。 文件流的目标目录不存在或不可写**。
解决办法是：空间更换完毕后自行修改wp-config里的此项属性所对应的缓存目录的绝对路径，目标文件夹权限设为777。
或者删除类似这样的代码 `define('WP_TEMP_DIR', ABSPATH.'wp-content/tmp');`
# ubuntu 18.04 安装配置wordpress
1、ubuntu 18.04+nginx1.14.0+mysql5.7.33+wordpress5.7+php7.2
2 、将软件包索引和系统软件包更新为最新版本
```
sudo apt update
sudo apt upgrade
```
3、安装mysql
```
apt install mysql-server
   20  cd /data/
   22  mkdir mysql
# 修改mysql存储路径
   23  cp -r /var/lib/mysql/* /data/mysql
   24  systemctl stop mysql
   25  vim /etc/mysql/mysql.conf.d/mysqld.cnf 
   27  chown -R mysql.mysql mysql/
   28  vim /etc/apparmor.d/usr.sbin.mysqld 
   29  systemctl restart apparmor
   30  systemctl restart mysql
```
创建用户及库
```
CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'change-with-strong-password';
FLUSH PRIVILEGES;
EXIT;
```
批量修改数据库域名
```
UPDATE wp_options SET option_value = replace(option_value, 'http://old.com', 'http://new.com') WHERE option_name = 'home' OR option_name = 'siteurl';
UPDATE wp_posts SET guid = replace(guid, 'http://old.com','http://www.newurl');
UPDATE wp_posts SET post_content = replace(post_content, 'http://old.com', 'http://new.com');
UPDATE wp_postmeta SET meta_value = replace(meta_value,'http://old.com','http://new.com');
UPDATE wp_usermeta SET meta_value = replace(meta_value, 'http://old.com', 'http://new.com');
UPDATE wp_comments SET comment_content = REPLACE (comment_content, 'http://old.com', 'http://new.com');
UPDATE wp_comments SET comment_author_url = REPLACE (comment_author_url, 'http://old.com','http://new.com');
```
4、安装php
```
   33  sudo apt install php7.2-cli php7.2-fpm php7.2-mysql php7.2-json php7.2-opcache php7.2-mbstring php7.2-xml php7.2-gd php7.2-curl
   35  systemctl status php7.2-fpm.service 
# 修改php.ini的2个参数 cgi.fix_pathinfo=0,upload_max_filesize = 51M
   37  vim /etc/php/7.2/fpm/php.ini 
   38  systemctl restart php7.2-fpm.service
```
5、安装nginx
`apt install nginx`
6、安装wordpress
`wget https://wordpress.org/latest.tar.gz`
7、配置
参考: https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/
```
server {
    listen 80;
    server_name www.paaswork.com;

    root /data/www_paaswork_com;
    index index.php;

    # log files
    access_log /var/log/nginx/paaswork.com.access.log;
    error_log /var/log/nginx/paaswork.com.error.log;

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_intercept_errors on;
        proxy_connect_timeout   180;
        proxy_send_timeout      180;
        proxy_read_timeout      180;
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }

    location ~* /(.git|.gitignore){
        deny all;
    }

    location /wp-admin {
      if ($reject = on) {
        return 403;
        break;
       }
    }

    location /check {
            root /etc/nginx/check;
    }

    location /images/bg_5XX.png {
            proxy_set_header Host leanwork-static.oss-cn-hangzhou.aliyuncs.com;
            proxy_pass https://leanwork-static.oss-cn-hangzhou.aliyuncs.com/static/images/bg_5XX.png;
            proxy_redirect off;
    }
    error_page   404 http://paaswork.com;
    error_page   403 444 500 502 503 504 http://$host/error.html?sc=$status;
    location /error.html {
            proxy_set_header Host leanwork-static.oss-cn-hangzhou.aliyuncs.com;
            proxy_pass https://leanwork-static.oss-cn-hangzhou.aliyuncs.com/static/5XX.html$is_args$args;
            proxy_redirect off;
    }
}
```

