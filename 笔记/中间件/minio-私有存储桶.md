 文档:http://docs.minio.org.cn/minio/baremetal/

**域名的作用**

默认情况下，如果要访问某个对象，地址为：`http://IP:9001/bucket/xxx.txt`，如果您在搭建的时候添加了域名参数`MINIO_DOMAIN`，域名做好解析后，您可以使用这样的方式访问到对象：`http://bucket.xxx.com/1.txt`，相当于就是将bucket映射为主机名称（域名前缀）

bucket的概念，翻译成中文就是“桶”，我们的对象（文件）就是存放在这个“桶里面”

使用之前我们需要先进行设置`mc config host add`????
