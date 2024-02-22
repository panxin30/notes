# 0-简介

Openconnect VPN服务端简称ocserv（Openconnect VPN Server），ocserv是一款开源的，兼容Cisco Anyconnect VPN的VPN服务端软件。目前状况下通讯较为稳定，干扰较小。主要优势是多平台的支持，Windows、Android、iOS都能找到它的客户端。
参考:
# Set up OpenConnect VPN Server (ocserv) on Ubuntu 20.04
## Step 0:阿里云申请的免费证书
## Step 1: Installing OpenConnect VPN Server on Ubuntu 20.04
Log into your Ubuntu 20.04 server. Then use apt to install the ocserv package,which is included in Ubuntu repository since 16.04.

`sudo apt install ocserv`
Once installed, the OpenConnect VPN server is automatically started. You can check its status with:

`systemctl status ocserv`

## Step 2: Editing OpenConnect VPN Server Configuration File
Edit ocserv configuration file.

`sudo vim /etc/ocserv/ocserv.conf`
First, configure password authentication. By default, password authentication through PAM (Pluggable Authentication Modules) is enabled, which allows you to use Ubuntu system accounts to login from VPN clients. This behavior can be disabled by commenting out the following line.

`auth = "pam[gid-min=1000]"`
If we want users to use separate VPN accounts instead of system accounts to login, we need to add the following line to enable password authentication with a password file.

`auth = "plain[passwd=/etc/ocserv/ocpasswd]"`
After finishing editing this config file, we will see how to use ocpasswd tool to generate the /etc/ocserv/ocpasswd file, which contains a list of usernames and encoded passwords.

```
tcp-port = 443
udp-port = 443
Then find the following two lines. We need to change them.

server-cert = /etc/ssl/vpn.51sw.cc.pem
server-key = /etc/ssl/vpn.51sw.cc.key
```
Then, set the maximal number of clients. Default is 16. Set to zero for unlimited.

`max-clients = 16`
Set the number of devices a user is able to login from at the same time. Default is 2. Set to zero for unlimited.

`max-same-clients = 2`
Next, find the following line. Change false to true to enable MTU discovery, which can optimize VPN performance.

`try-mtu-discovery = false`
After that, set the default domain to vpn.example.com.

`default-domain = vpn.51sw.cc
The IPv4 network configuration is as follows by default. This will cause problems because most home routers also set the IPv4 network range to 192.168.1.0/24.
We can use another private IP address range (10.10.10.0/24) to avoid IP address collision, so change the value of ipv4-network to

`ipv4-network = 10.10.10.0/24`

Change DNS resolver address. You can use Google’s public DNS server.

`dns = 8.8.8.8`
Note: It’s a good practice to run your own DNS resolver on the same server, especially if you are a VPN provider. If there’s a DNS resolver running on the same server, then specify the DNS as

`dns = 10.10.10.1`
10.10.10.1 is the IP address of OpenConnect VPN server in the VPN LAN. This will speed up DNS lookups a little bit for clients because the network latency between the VPN server and the DNS resolver is eliminated.

Then comment out all the route parameters (add # symbol at the beginning of the following four lines), which will set the server as the default gateway for the clients.
```
route = 10.10.10.0/255.255.255.0
route = 192.168.0.0/255.255.0.0
route = fef4:db8:1000:1001::/64

no-route = 192.168.5.0/255.255.255.0
```
Save and close the file  Then restart the VPN server for the changes to take effect.

`sudo systemctl restart ocserv`

## Step 4: Creating VPN Accounts
Now use the ocpasswd tool to generate VPN accounts.

`sudo ocpasswd -c /etc/ocserv/ocpasswd username`
You will be asked to set a password for the user and the information will be saved to /etc/ocserv/ocpasswd file. To reset password, simply run the above command again.

## Step 5: Enable IP Forwarding
In order for the VPN server to route packets between VPN client and the outside world, we need to enable IP forwarding. Edit sysctl.conf file.

`sudo vim /etc/sysctl.conf`
Add the following line at the end of this file.

`net.ipv4.ip_forward = 1`
Save and close the file. Then apply the changes with the below command. The -p option will load sysctl settings from /etc/sysctl.conf file. This command will preserve our changes across system reboots.

`sudo sysctl -p`

## Step 6: Configure IP Masquerading 打开NAT功能
`sudo iptables -t nat -A POSTROUTING -j MASQUERADE` #重启后会失效，需要保存
## 保存iptables规则
`sudo apt install iptables-persistent`
每当设置了新的iptables规则后，使用如下命令保存规则即可，规则会根据ipv4和ipv6分别保存在了/etc/iptables/rules.v4和/etc/iptables/rules.v6文件中。
`sudo netfilter-persistent  save`
由于 iptables-persistent 在安装时已经把它作为一个服务设置为开机启动了，它在开机后会自动加载已经保存的规则，所以也就达到了永久保存的目的。