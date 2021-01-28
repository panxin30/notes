### **supervisor**
ubuntu 18.04 apt update && apt install supervisor
```
[program:mongo]
directory=/data/mongodb
command=/usr/bin/mongod --config /etc/mongod.conf
autostart=true
autorestart=true
user=mongodb
```
If mongod is daemonizing itself, it's not the kind of program supervisor is designed to manage. Supervisor expects its supervised programs to run in the foreground so it can monitor if they've exited.

See [supervisord docs](http://supervisord.org/subprocess.html).

在shell命令行下你运行的这个command，你需要用Ctrl-C才能重新获得对终端的控制的命令，才是前台运行。
如果在运行后无需按Ctrl-C即可向您返回shell提示，则在**supervisord**下没有用。所有程序都有可以在前台运行的选项，但是没有“标准方法”可以执行。您需要阅读每个程序的文档。


### **supervise**
`sudo apt update -y`
`sudo apt install -y daemontools`
Daemontools是svscanboot，svscan，supervise，svc，svok，svstat等一系列工具的合集。
### 加入一个新服务
最简单的方式是建立一个文件夹test
在test目录下编写脚本run（注意：名字必须为run。**注意run 脚本里不能后台启动**。）
