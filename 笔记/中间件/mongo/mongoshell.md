引用：https://www.docs4dev.com/docs/zh/mongodb/v3.6/reference/tutorial-write-scripts-for-the-mongo-shell.html
# **shell命令行模式执行mongo命令**
### **1. 交互式 mongo shell**

大部分的 mongodb 教程，在第一章都会讲解这种方式。
mongo 127.0.0.1:27017
use test
db.users.findOne()

### **2. mongo --eval 运行一段脚本**

**不进入交互模式，直接在 OS 的命令行下运行一段mongodb脚本,打印出所有dbname。**

mongo 127.0.0.1:27017/test --eval "printjson(db.users.findOne())"
`mongo 127.0.0.1:27017/admin -uadmin -pWAer8R59G6 --eval "db.adminCommand('listDatabases')"`
### **3. 在OS命令行下，运行一个js文件**

mongo 127.0.0.1:27017/test userfindone.js

userfindone.js 的内容：
printjson(db.users.findOne());


### **4. 在mongo shell 交互模式下，运行一个js文件**

mongo test
load("/root/mongojs/userfindone.js")
load() 参数中的文件路径，既可以是相对路径，也可以是绝对路径。
在mongo shell下查看当前工作路径的方法： pwd( )
