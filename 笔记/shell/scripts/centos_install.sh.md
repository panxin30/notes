```
#/bin/bash
os=`cat /etc/redhat-release |grep -i Centos`

if [ "$os" = "" ];then
        echo "$0 just can use it on system CentOS"
fi

mongodb() {
echo "begin install mongodb"
echo "[mongodb-org-3.4]" >/etc/yum.repos.d/mongodb-org-3.4.repo
echo "name=MongoDB Repository" >>/etc/yum.repos.d/mongodb-org-3.4.repo
echo 'baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/' >>/etc/yum.repos.d/mongodb-org-3.4.repo
echo "gpgcheck=1" >>/etc/yum.repos.d/mongodb-org-3.4.repo
echo "enabled=1" >>/etc/yum.repos.d/mongodb-org-3.4.repo
echo "gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc" >>/etc/yum.repos.d/mongodb-org-3.4.repo

#sudo yum update -y

sudo yum install -y mongodb-org

sudo chkconfig mongod on

sudo service mongod start

echo "mongodb install complete"
exit

}
jdk() {
echo "begin install java jdk"
#yum install -y java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64
yum remove openjdk* -y
wget http://databackup.lwork.com/tools/jdk/jdk-linux-x64.rpm 

rpm -ivh jdk-linux-x64.rpm
java -version |grep -i version|grep "1.8"

echo "java jdk install complete"

exit
}


rabbitmq() {
echo "begin install erlang"
wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
rpm --import https://packages.erlang-solutions.com/rpm/erlang_solutions.asc
yum install -y erlang socat

echo "begin install rabbitmq-server"                               
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc

rpm -ivh http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.6/rabbitmq-server-3.6.6-1.el7.noarch.rpm

sudo chkconfig rabbitmq-server on
sudo service rabbitmq-server start
}

mysqld() {
echo "begin install mysqld"
echo "[mysql57-community]" >/etc/yum.repos.d/mysql-community.repo
echo "name=MySQL 5.7 Community Server" >>/etc/yum.repos.d/mysql-community.repo
echo "baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64/" >>/etc/yum.repos.d/mysql-community.repo
echo "enabled=1" >>/etc/yum.repos.d/mysql-community.repo
echo "gpgcheck=0" >>/etc/yum.repos.d/mysql-community.repo

#sudo yum update

sudo yum install -y mysql-community-server

systemctl enable mysqld.service

service mysqld start

echo "mysql install complete
exit"
}

redis() {
echo "begin install redis"
wget http://download.redis.io/releases/redis-3.2.3.tar.gz

tar zxvf redis-3.2.3.tar.gz 
cd redis-3.2.3
mkdir -p /usr/local/redis

make PREFIX=/usr/local/redis install -j8

mkdir -p /usr/local/redis/etc

mkdir -p /usr/local/redis/var/
mkdir -p /usr/local/redis/logs/

cp redis.conf /usr/local/redis/etc/

echo '#!/bin/bash
#
# redis - this script starts and stops the redis-server daemon
#
# chkconfig:   - 85 15 
# description:  Redis is a persistent key-value database
# processname: redis-server
# config:      /usr/local/redis/etc/redis.conf
# config:      /etc/sysconfig/redis
# pidfile:     /var/run/redis.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

redis="/usr/local/redis/bin/redis-server"
prog=$(basename $redis)

REDIS_CONF_FILE="/usr/local/redis/etc/redis.conf"

[ -f /etc/sysconfig/redis ] && . /etc/sysconfig/redis

lockfile=/var/lock/subsys/redis

start() {
    [ -x $redis ] || exit 5
    [ -f $REDIS_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $redis $REDIS_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    stop
    start
}

reload() {
    echo -n $"Reloading $prog: "
    killproc $redis -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload}"
        exit 2
esac' >/etc/init.d/redis

sudo chmod +x /etc/init.d/redis

sudo  sed -i s/"daemonize no"/"daemonize yes"/g /usr/local/redis/etc/redis.conf

sudo /etc/init.d/redis start

sudo chkconfig --add redis
sudo chkconfig --level 3 redis on
echo "redis install complete"
exit
}

nginx () {
echo "begin nginx install"
echo '[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/x86_64/
gpgcheck=0
enabled=1 
' >/etc/yum.repos.d/nginx.repo

#sudo yum update -y

sudo yum install -y nginx

sudo chkconfig nginx on
sudo systemctl enable nginx.service
sudo service nginx start

echo "nginx install complete"
exit
}
init () {
echo "begin init"
#ssh key
wget http://databackup.lwork.com/script/authorized_keys
mkdir -p /root/.ssh
mv authorized_keys /root/.ssh/
chmod -R 600 /root/.ssh/authorized_keys

service sshd restart
#ntp
echo "exclude=kernel*" >>/etc/yum.conf
date -R
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
date;hwclock -r 
ntpdate time.windows.com && hwclock -w 
#hostname
echo -n "Pls input hostname:"
read myname
if [ ! -n "${myname}" ]; then
echo "Usage:no hostname input";
exit;
else
hostname ${myname}
echo ${myname} >/etc/hostname

fi
#login info
wget http://databackup.lwork.com/script/aliyun_motd.sh
sh aliyun_motd.sh

echo "init complete"
}

consul () {
echo "begin install consul"
wget https://releases.hashicorp.com/consul/0.6.4/consul_0.6.4_linux_amd64.zip
yum install -y  unzip
unzip consul_0.6.4_linux_amd64.zip
mv consul /usr/bin
echo "consul install complete"
}

zabbix_agent (){

echo "begin install zabbix_agent"
wget http://databackup.lwork.com/tools/zabbix_agent/zabbix_agent_linux.tgz
wget http://databackup.lwork.com/tools/zabbix_agent/zabbix_agentd
chmod +x zabbix_agentd
mv zabbix_agentd /etc/init.d/
tar zxvf zabbix_agent_linux.tgz
mv zabbix /usr/local/
adduser zabbix -s /sbin/nologin
/etc/init.d/zabbix_agentd start
echo "zabbix_agent install complete!"
}

ops_agent ()
{
echo "begin install ops agent"

curl -s http://tools.lwork.com/download/install_agent.sh | bash

echo "ops agent install complete"
}

echo_help() {
        echo "invalid argument!"
        echo "Usage:only support mongodb|rabbitmq|jdk|mysqld|redis|nginx|init|consul|zabbix_agent|ops_agent"
}

if [ $# -lt 1 ];then
        echo_help
        exit 1
fi
case $1 in
mongodb) mongodb;;
rabbitmq) rabbitmq;;
jdk) jdk;;
mysqld) mysqld;;
redis) redis;;
nginx) nginx;;
init) init;;
consul) consul;;
zabbix_agent) zabbix_agent;;
ops_agent) ops_agent;;
*) echo_help
   exit 2;;
esac
```