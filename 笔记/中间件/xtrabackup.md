官网 https://www.percona.com/doc/percona-xtrabackup/2.4/installation.html
选项参数 https://www.percona.com/doc/percona-xtrabackup/LATEST/xtrabackup_bin/xbk_option_reference.html
### Xtrabackup优点

　    1）备份速度快，物理备份可靠
　　2）备份过程不会打断正在执行的事务（无需锁表）
　　3）能够基于压缩等功能节约磁盘空间和流量
　　4）自动备份校验
　　5）还原速度快
　　6）可以流传将备份传输到另外一台机器上
　　7）在不增加服务器负载的情况备份数据
　　8）物理备份工具，在同级数据量基础上，都要比逻辑备份性能要好的多。几十G到不超过TB级别的条件下。但在同数据量级别，物理备份恢复数据上有一定优势。
 ### ubuntu 18.04 安装
1 授权
`grant reload,process,lock tables,replication client on *.* to bkpuser@localhost identified by 's3cret';`
2 全量备份
`time /usr/bin/xtrabackup --user=bkpuser --password='s3cret' --host=localhost --backup --compress --compress-threads=2 --target-dir=/opt/mysql-backup/${TIMES}/`
--compress    //此选项指示xtrabackup压缩备份的InnoDB数据文件，会生成 *.qp 文件。
整个备份过程：连接数据库，开始拷贝redo log，拷贝innodb表文件，锁表、拷贝非innodb表文件，停止拷贝redo log，解锁。
3 恢复-单独准备一套mysql不启动，/var/lib/mysql/清空
如果没有qpress，需要先安装apt install qpress
解压`xtrabackup  --decompress --target-dir=/home/ubuntu/2020011617`
删除*.qp `find /data/backup -name "*.qp" | xargs rm`
恢复 `xtrabackup --copy-back --target-dir=/home/ubuntu/2020011617`
chown -R mysql.mysql /var/lib/mysql/
systemctl start mysql

