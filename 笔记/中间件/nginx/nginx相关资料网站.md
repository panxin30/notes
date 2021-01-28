http://nginx.org/en/linux_packages.html#stable
http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass
https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/



## 屏蔽访问某文件夹
```
    location ~* /(.git|.gitignore){
        deny all;
    }
```


### module-version-x-instead-of-y
https://www.nginx.com/blog/failed-nginx-plus-upgrade-module-version-x-instead-of-y/
### **nginx stream 模块，属于四层转发**
apt 安装的1.16直接有stream模块