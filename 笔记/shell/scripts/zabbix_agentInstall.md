```
#!/bin/bash
#Used for install zabbix-agent
#Used for auto discovery
set -ex

SCRIPT_DIR=$(pwd)
UNIX_NAME=$(uname -a | awk -F ' ' '{print $4}' | awk -F '-' '{print $2}')

Centos (){
#for centos agent
yum -y install gcc gcc-c++
wget http://nchc.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.0.4/zabbix-3.0.4.tar.gz
tar zxf zabbix-3.0.4.tar.gz
groupadd zabbix
useradd -g zabbix -s /sbin/nologin zabbix
cd zabbix-3.0.4
./configure --prefix=/usr/local/zabbix --enable-agent
make && make install
sleep 1
cd -
cp -f ${SCRIPT_DIR}/zabbix-3.0.4/misc/init.d/fedora/core/zabbix_agentd /etc/init.d/
chmod 755 /etc/init.d/zabbix_agentd
sed -i "s/\/usr\/local/\/usr\/local\/zabbix/g" /etc/init.d/zabbix_agentd
chkconfig --add zabbix_agentd
chkconfig --level 3 zabbix_agentd on
sed -i "s/Server=127.0.0.1/Server=10.24.21.65/g" /usr/local/zabbix/etc/zabbix_agentd.conf 
sed -i "s/ServerActive=127.0.0.1/ServerActive=10.24.21.65/g" /usr/local/zabbix/etc/zabbix_agentd.conf
sed -i "s/Hostname=Zabbix\ server/#Hostname=system.hostname/g" /usr/local/zabbix/etc/zabbix_agentd.conf
sed -i "s/#\ HostMetadataItem=/HostMetadataItem=system.uname/g" /usr/local/zabbix/etc/zabbix_agentd.conf
sed -i "s/#\ Timeout=3/Timeout=15/g" /usr/local/zabbix/etc/zabbix_agentd.conf
service zabbix_agentd start

rm -rf zabbix*
}

Ubuntu (){
#for ubuntu agent
#cp misc/init.d/debian/zabbix-agent /etc/init.d
#
#
#update-rc.d zabbix-agent defaults
sudo apt-get update
#sudo apt-get -y install gcc gcc-c++
wget http://nchc.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.0.4/zabbix-3.0.4.tar.gz
tar zxf zabbix-3.0.4.tar.gz
sleep 1
groupadd zabbix
useradd -g zabbix -s /sbin/nologin zabbix
cd zabbix-3.0.4
./configure --prefix=/usr/local/zabbix --enable-agent
make && make install
cd -
sleep 1
cp -f ${SCRIPT_DIR}/zabbix-3.0.4/misc/init.d/debian/zabbix-agent /etc/init.d/
chmod 755 /etc/init.d/zabbix-agent
sed -i "s/\/usr\/local/\/usr\/local\/zabbix/g" /etc/init.d/zabbix-agent
update-rc.d zabbix-agent defaults
sed -i "s/Server=127.0.0.1/Server=10.24.21.65/g" /usr/local/zabbix/etc/zabbix_agentd.conf
sed -i "s/ServerActive=127.0.0.1/ServerActive=10.24.21.65/g" /usr/local/zabbix/etc/zabbix_agentd.conf
sed -i "s/Hostname=Zabbix\ server/#Hostname=system.hostname/g" /usr/local/zabbix/etc/zabbix_agentd.conf
sed -i "s/#\ HostMetadataItem=/HostMetadataItem=system.uname/g" /usr/local/zabbix/etc/zabbix_agentd.conf
sed -i "s/#\ Timeout=3/Timeout=15/g" /usr/local/zabbix/etc/zabbix_agentd.conf
service zabbix-agent start

rm -rf zabbix*
}

if [ ${UNIX_NAME} = 'Ubuntu' ]
then
                Ubuntu
else
                Centos
fi
exit 0


#case $1 in 
#		centos)
#			Centos
#		;;
#		ubuntu)
#			Ubuntu
#		;;
#			*)
#			                            echo "Usage:
#
#
#            注意:
#            "
#		;;
#esac
```