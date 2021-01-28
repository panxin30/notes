## 故障现象gtid主从报错信息

When@@SESSION.GTID_NEXT is set to a GTID, you must explicitly set it to a differentvalue after a COMMIT or ROLLBACK. Please check GTID_NEXT variable manual pagefor detailed explanation. Current @@SESSION.GTID_NEXT is'039a0c0c-9cee-11e6-922c-525400a9bf0e:324065731'.

### 故障可能原因
1 enforce_gtid_consistency为OFF，主库执行create table as select * from xx;
2 myisam引擎表，主库执行insert delayed into xx values...
3 主从引擎不一致，主库innodb引擎一个事务中写入两条数据，传到从库的myisam引擎执行这个事务

### 原因分析
根本原因是每一个GTID需要与一个唯一的事务对应
**针对原因1**，create table as select * from xx会自动转换成row模式，这时会拆成crate table和insert两个事务，这时传到slave时，slave执行完crate以后，多出一个insert事务没有gtid，于是报错；
**针对原因2**，只有myisam引擎支持insert delayed语法，insert delay是异步写入，也就是一旦执行立即返回给客户端成功。mysql内部处理insert delay时，会将多个线程的insert合并后一起执行，但是只生成了一个GTID；于是传到slave后，由于是myisam表，从库的同样只能执行第一条SQL，于是报错
**针对原因3**，主库innodb执行一个事务，只产生一个gtid，myisam不支持事务，事务的第一条执行完以后，第二个sql就没有gtid，于是报错