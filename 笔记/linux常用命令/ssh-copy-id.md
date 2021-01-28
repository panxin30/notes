linux下自带的命令，可用于自动打通服务器，前提是已经创建过key，并且在默认路径下。
`ssh-keygen -t rsa -b 4096`
ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.204.6.236
`id_dsa         -->私钥(钥匙)`
`id_dsa.pub     -->公钥(锁)`


[https://www.cnblogs.com/Hi-blog/p/9482418.html](https://www.cnblogs.com/Hi-blog/p/9482418.html)