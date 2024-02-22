# 一、ocserv服务器修改配置
**默认ocserv已经在该服务器安装**
# 修改hostname
`hostnamectl  set-hostname vpn.51sw.cc`
# 在hosts中添加
`内网ip vpn.51sw.cc`
# 解析vpn.51sw.cc到这台服务器IP地址
# 安装freeipa客户端到该服务器
```
#服务器是ubuntu18.04
apt update
apt install -y freeipa-client
```
# 配置freeipa客户端
```
root@vpn:~# ipa-client-install

DNS discovery failed to determine your DNS domain
Provide the domain name of your IPA server (ex: example.com): 51sw.cc
Provide your IPA server name (ex: ipa.example.com): freeipa.51sw.cc
The failure to use DNS to find your IPA server indicates that your resolv.conf file is not properly configured.
Autodiscovery of servers for failover cannot work with this configuration.
If you proceed with the installation, services will be configured to always access the discovered server for all operations and will not fail over to other servers in case of failure.
Proceed with fixed values and no DNS discovery? [no]: yes
Client hostname: vpn.51sw.cc
Realm: 51SW.CC
DNS Domain: 51sw.cc
IPA Server: freeipa.51sw.cc
BaseDN: dc=51sw,dc=cc

Continue to configure the system with these values? [no]: yes
Synchronizing time
No SRV records of NTP servers found and no NTP server or pool address was provided.
Using default chrony configuration.
Attempting to sync time with chronyc.
Time synchronization was successful.
User authorized to enroll computers: admin
Password for admin@51SW.CC: 
Successfully retrieved CA cert
    Subject:     CN=Certificate Authority,O=51SW.CC
    Issuer:      CN=Certificate Authority,O=51SW.CC
    Valid From:  2022-01-15 09:36:13
    Valid Until: 2042-01-15 09:36:13

Enrolled in IPA realm 51SW.CC
Created /etc/ipa/default.conf
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
Configured /etc/krb5.conf for IPA realm 51SW.CC
trying https://freeipa.51sw.cc/ipa/json
[try 1]: Forwarding 'schema' to json server 'https://freeipa.51sw.cc/ipa/json'
trying https://freeipa.51sw.cc/ipa/session/json
[try 1]: Forwarding 'ping' to json server 'https://freeipa.51sw.cc/ipa/session/json'
[try 1]: Forwarding 'ca_is_enabled' to json server 'https://freeipa.51sw.cc/ipa/session/json'
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_dsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
[try 1]: Forwarding 'host_mod' to json server 'https://freeipa.51sw.cc/ipa/session/json'
Could not update DNS SSHFP records.
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring 51sw.cc as NIS domain.
Client configuration complete.
The ipa-client-install command was successful
```
# 完成后在https://freeipa.51sw.cc登录

查看主机那项有没有增加主机名`vpn.51sw.cc`，有，且注册为true，就成功了
# 二、freeipa 服务端操作
```
kinit admin     #这里需要输入admin密码
ipa service-add HTTP/vpn.example.com
```
# 三、修改ocserv配置文件（开启以下选项）
```
#/etc/ocserv/ocserv.conf

#用户认证
auth = pam

#用户设置
config-per-group = /etc/ocserv/config-per-user
default-group-config = 

#这里默认没有开启，需要创建
mkdir  /etc/ocserv/config-per-user
```
# 四、 freeipa网页端创建用户
