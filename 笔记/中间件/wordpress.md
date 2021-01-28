[https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/](https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/)
### nginx直接做为web服务器
centos 6.9 nginx1.12.x php7.3 wordpress5.02 mysql5.7.x
部署成功后，为了美观在设置-->固定链接-->自定义结构 /%category%/%post_id%/
[http://webqa.lwork.com/broker-work-price/](http://webqa.lwork.com/broker-work-price/)

vim /etc/php.ini
```
cgi.fix_pathinfo = 0 修改
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