```
server {
    listen       9999;        #端口
    server_name  localhost;   #服务名
    index  index.html;
    charset utf-8; # 避免中文乱码
    root    /data1/tony;  #显示的根索引目录，注意这里要改成你自己的，目录要存在

    location / {
        autoindex on;             #开启索引功能
        autoindex_exact_size off; # 关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
        autoindex_localtime on;   # 显示本机时间而非 GMT 时间
    }
}
```
使用  http://8.136.251.226:9999/      访问

