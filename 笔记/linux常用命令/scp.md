## 1.scp基本格式

~~~
scp local_file user@host:/dir
~~~

## 2.scp复制文件使用指定端口

scp默认连接的远端主机22端口，如果ssh不是使用标准的22端口（以16022为例）则使用-P（P大写）指定：

~~~
# 本地拷贝到远程
scp -P 16022 local_file user@host:/dir
# 远程拷贝到本地
scp -P 38529 10.28.188.87:/root/demo_account.sql .
~~~

## 3.从远端主机将文件复制到另一台远端主机

scp不仅可以将文件从本机复制到远端机器，还可以将文件从远端机复制到本机，还可以将文件从一台远端机复制到另一台远端机：

~~~
scp user1@host1:file1 user2@host2:file
~~~

这也就是说当情况如：A/B/C三台主机，A和B、B和C网络相通但A和C网络不通，要求把文件从A复制到C上。

那么可行的操作是把文件从A复制到B，再把文件从B复制到C；还可行的情况是直接在B上把文件从A复制到C，在B上执行：

~~~
scp userA@hostA:fileA userC@hostC:fileC
~~~