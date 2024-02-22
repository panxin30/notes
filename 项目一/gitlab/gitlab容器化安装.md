```
docker run -d \
    -p 8443:443 -p 8090:80 -p 8022:22 \
    --restart always --name gitlab \
    -v /usr/local/gitlab/etc:/etc/gitlab \
    -v /usr/local/gitlab/log:/var/log/gitlab \
    -v /usr/local/gitlab/data:/var/opt/gitlab \
    --privileged=true \
    twang2218/gitlab-ce-zh

docker run --detach \
    --hostname 10.1.1.78 \
    --publish 8001:22 --publish 8002:80 --publish 8003:443 \
    --name gitlab --restart always \
    --volume /var/opt/gitlab/config:/etc/gitlab \
    --volume /var/opt/gitlab/logs:/var/log/gitlab \
    --volume /var/opt/gitlab/data:/var/opt/gitlab 8bc3815e6043
参数名称	参数说明
detach	指定容器运行于前台还是后台
hostname	指定主机地址，如果有域名可以指向域名
publish	指定容器暴露的端口,左边的端口代表宿主机的端口，右边的是代表容器的端口
name	给容器起一个名字，
restart always	总是重启
volume	数据卷，在docker中是最重要的一个知识点.
```
