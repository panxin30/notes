1. 查询索引
`show index from brokerwork_t002002_mt4_1.mt4_trades; \G`
2. 新建索引
`alter table brokerwork_t002414_mt4_1.mt4_trades add index index_comment_cmd(comment,cmd);`


explain的用法
MySQL的EXPLAIN命令显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。  
**一、通过expalin可以得到**  
1、表的读取顺序  
2、表的读取操作的操作类型  
3、哪些索引可以使用  
4、哪些索引被实际使用  
5、表之间的引用  
6、每张表有多少行被优化器查询

`explain SELECT ticket, login, symbol, cmd, volume, open_time, close_time FROM brokerwork_T002414_MT4_1.mt4_trades WHERE comment = 'r2132568833-37130232-eTojhUNE'AND cmd = 6;`