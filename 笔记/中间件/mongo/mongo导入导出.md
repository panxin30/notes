## **恢复数据**
-----------------
①：备份后,新增的数据在恢复时不会被删除。
②：如果存在同样的数据，恢复的时候会被覆盖掉，不会重复插入。
③：如果备份后某条数据被update了，恢复的时候备份后倘若某条记录被更新了，无法恢复到备份点的状态
##  **所以在生产环境中，可用备份数据还原到开发环境，再修改对应数据，再插入或修改到生产**
```
mongorestore -h127.0.0.1:27017 -d ops-db -c t_module_permission -u ops -p abc123  /root/ops-db/t_module_permission.bson
mongorestore -h10.47.43.22:27017 -ubrokerwork -po6RJzAF1LELzutEv -dbrokerwork /root/brokerwork20170416/brokerwork
load 在mongodb命令行下可以直接导入一个文件 load("1.js")
备注：mongodump 如果不指定 -d 参数，刚会备份整个 MongoDB 实例。
mongorestore -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 --drop 文件存在路径
mongorestore -h127.0.0.1 --port 27017 -uadmin -pWAer8R59G6 --authenticationDatabase=admin -dtest --drop /root/test/
### --drop
#### 恢复的时候，先删除当前数据(包括索引)，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除，慎用哦！所以使用drop的导入的时候应该bson和json应该一块导入
```
直接执行文件,文件名改为js结尾(文件里面是一些执行语句)
-----------------------------
mongo 127.0.0.1:27017/msc-db -umsc -pabc123 flag.js
## **数据备份**
AWS的mongo集群导出需要加
--forceTableScan -d database_name
导出的是文件夹
`mongodump -h127.0.0.1:27017 -d msc-db -c t_tenant_i18n -u msc -p abc123 --oplog -o msc-db/`

--oplog只适合于副本集，备份时候记录期间的数据变化，恢复时会更新这些变化的数据
*****
mongodump -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 -o 文件存在路径  
**mongodump 仅备份数据库中的文档，不备份索引，所以我们还原后，需要重新生成索引**。
如果没有用户谁，可以去掉-u和-p。  
如果导出本机的数据库，可以去掉-h。  
如果是默认端口，可以去掉--port。  
如果想导出所有数据库，可以去掉-d。  
2,导出所有数据库  
`mongodump -h 127.0.0.1 -u cine -p 111111 -o /www/web/tmp`  
3,导出指定数据库  
`mongodump -h 127.0.0.1 -d cine -o /www/web/tmp/`
------------------------------------------

**用于集群**
```
mongodump -h127.0.0.1 --port 20000 --authenticationDatabase=admin --gzip -dfeedworkBid -cLEANWORK_XAUUSD_W1 -uadmin -pWAer8R59G6 -o /root/

mongorestore --gzip  导入压缩的备份
mongorestore -h10.47.43.22 -urakeback -pGqTEt12E -drcr_t000138 --authenticationDatabase=admin --gzip dump_t000138/rcr_t000138/finish_tickets.bson.gz
```
**不要把mongoimport，mongoexport用于 生产环境的备份**

\-》因为mongoimport，mongoexport只是导入导出JSON格式数据

\-〉无法支持和保留所有的类型的数据

\-〉原因是：Mongo内部完整的数据是用BSON格式去表示的，而JSON只是BSON的子集，所以有些数据格式，JSON无法表示出来
JSON虽然具有较好的跨版本通用性，但其只保留了数据部分，不保留索引，账户等其他基础信息
## **导出的是文件**
mongoexport/mongoimport
将collection导出为json/csv格式数据/将数据导入数据库,mongoimport恢复会直接覆盖这张表
`mongoexport -h10.25.205.183:27017 -u owsc -pDS3x8mHP  -d owsc -c t_structural_setting -q '{"tenantId":"T001172", "productId":"TW"}' -o chaxun.txt`
`mongoimport -h10.28.185.68:27017 -dbrokerwork -ct_customer_profiles -ubrokerwork -po6RJzAF1LELzutEv tony.txt`

## **--csv 以EXCEL格式导出**

## **导出某个字段**
```
mongoexport -h10.25.205.183:27017 -uowsc -pDS3x8mHP -dowsc -ct_tenant -f _id,tenantName -q '{}' --csv -o tony.csv
mongodump -h10.28.185.68 --port 27017 -ubrokerwork -p'o6RJzAF1LELzutEv' -dbrokerwork -ct_account_base_info -q '{"tenantId" : "T001194"}'  -o T001194
```
带条件的导出

统计客户数
```
for db in `cat 2.txt`
do
	   echo $db >>1.csv
	   sed  "s/117/${db}/g" 1.js >2.js
	   mongo 10.47.43.22:27017/brokerwork -ubrokerwork -po6RJzAF1LELzutEv 2.js >> 1.csv
done
```
下面是1.js
```
db = db.getSiblingDB('brokerwork');
cursor = db.t_customer_profiles.count({"tenantId" : "117","enabled" : true,"createTime" : {"$gte":NumberLong(1483200000000)},"createTime" : {"$lte":NumberLong(1490976000000)}});
printjson(cursor);
```
直接打印出结果
```
db.t_template.find({"tenantId":"T002105","type":"SMS","templateType" : "29298"},{ "id": 1, "title": 1,"templateType":1 }).forEach(function(item){
        print(JSON.stringify(item));
 })

db.t_tenant_product.find({"productId":{"$in":["BW","TW"]}}).forEach(function(products){
	db.t_tenant.find({"_id":products.tenantId}).forEach(function(tenants){
	 print(tenants.tenantName,products.tenantId,products.productId,products.productDomain,products.customerDomain);
		});
});
```
# 
```
db.t_form_field.find({'tenantId':'T000000',"sysDefault" : true,'message.vi-VN':{$exists:true}}).forEach(function(v){
    db.t_form_field.update({'tenantId':'T001630','message.vi-VN':{$exists:false},'formId':v.formId,'fieldId':v.fieldId},{$set:{'message':v.message}});
})
```