ln -s 源文件/目录  目标文件/目录

[root@nc2 test]# ls
file
[root@nc2 test]# ln -s file link
[root@nc2 test]# ls -l
total 0
-rw-r--r-- 1 root root 0 Nov 11 14:41 file
lrwxrwxrwx 1 root root 4 Nov 11 14:42 link -> file
[root@nc2 test]# rm link
[root@nc2 test]# ls
file

ln -s /home/brokerwork/logs/ali-hk-bw-msc10/bw-user ali-hk-bw-msc10

ali-hk-bw-msc7 -> /home/brokerwork/logs/ali-hk-bw-msc7/pub-auth
rm ali-hk-bw-msc7


#### 这里有两点要注意：
第一，ln命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化；
第二，ln的链接又分软链接和硬链接两种，软链接就是ln –s 源文件 目标文件，它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接 ln 源文件 目标文件，没有参数-s， 它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化。
ln指令用在链接文件或目录，如同时指定两个以上的文件或目录，且最后的目的地是一个已经存在的目录，则会把前面指定的所有文件或目录复制到该目录中。若同时指定多个文件或目录，且最后的目的地并非是一个已存在的目录，则会出现错误信息。

3．命令参数：
必要参数:
-b 删除，覆盖以前建立的链接
-d 允许超级用户制作目录的硬链接
-f 强制执行
-i 交互模式，文件存在则提示用户是否覆盖
-n 把符号链接视为一般目录
-s 软链接(符号链接)
-v 显示详细的处理过程