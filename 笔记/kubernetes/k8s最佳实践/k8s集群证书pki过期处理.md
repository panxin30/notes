参考:https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/
# 高可用k8s集群更新证书后，需要将新证书拷贝到所有主节点
我是整个/etc/kubernets拷贝覆盖其他主节点。先做好备份。
同时重启下服务kubelet和下面4个
`kube-apiserver|kube-controller-manager|kube-scheduler|etcd`都重启下。
`docker ps | grep -v pause | grep -E "etcd|scheduler|controller|apiserver" | awk '{print $1}' | xargs docker restart`

# **k8s v1.14.3**
`cd /etc/kubernetes/pki && openssl x509 -in apiserver.crt -text -noout` 
`openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text  |grep Not`
确认证书是否过期
docker 18.09.7
现象：
kubectl get ns
The connection to the server x.x.x.x:6443 was refused - did you specify the right host or port?
1.以为拷贝到家目录的config失效，又重新拷贝了一份`cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`,没用
2. 查看`tail -f /var/log/syslog`
3. 查看kubelet日志`journalctl -xeu kubelet`
结果`Part of the existing bootstrap client certificate is expired`
处理：
```
cd /etc/kubernetes/pki/
mv {apiserver.crt,apiserver-etcd-client.key,apiserver-kubelet-client.crt,front-proxy-ca.crt,front-proxy-client.crt,front-proxy-client.key,front-proxy-ca.key,apiserver-kubelet-client.key,apiserver.key,apiserver-etcd-client.crt} ~/
kubeadm init phase certs all    ?????
cd /etc/kubernetes/
mv {admin.conf,controller-manager.conf,kubelet.conf,scheduler.conf} ~/
kubeadm init phase kubeconfig all
#重启kube-apiserver,kube-controller-manager ,kube-scheduler，etcd这4个容器
docker ps | grep -v pause |grep -E 'kube-apiserver|kube-controller-manager|kube-scheduler'
docker ps | grep -v pause | grep -E "etcd|scheduler|controller|apiserver" | awk '{print $1}' | xargs docker restart
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
**使用 kubeadm 进行证书管理**
https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/
# **版本1.15后**
`kubeadm alpha certs check-expiration`检查证书是否过期
`kubeadm alpha certs renew`命令手动更新证书
`kubeadm alpha certs renew all`
# **版本1.23**
`kubeadm certs check-expiration`检查证书是否过期
`kubeadm certs renew`**未经验证**
`kubeadm certs renew all`**未经验证**
**做好备份，注意观察kubelet的状态（可以用docker restart kube-api等）**
小坑：
`kubeadm alpha certs renew`并不会更新kubelet证书（kubelet.conf文件里面写的客户端证书），因为kubelet证书是默认开启自动更新的

但是在执行`kubeadm init`的master节点的kubelet.conf文件里面的证书是以base64编码写死在conf文件的（和controller-manager.conf一样），在用kubeadm命令更新master证书时需要手动将kubelet.conf文件的`client-certificate-data`和`client-key-data`改为：

~~~yaml
client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
~~~
* 这个问题在`v1.17`版得到了解决[https://github.com/kubernetes/kubeadm/issues/1753]
# 验证
`kubectl get ns`
如果原证书已过期，则此时会报错

~~~
[root@FAT-K8S-M1 kubernetes]# kubectl get poderror: You must be logged in to the server (Unauthorized)
~~~

使用新授权文件即可

~~~
cp /etc/kubernetes/admin.conf ~/.kube/config
~~~
~~~
#重启kube-apiserver,kube-controller-manager ,kube-scheduler，etcd这4个容器
docker ps | grep -v pause |grep -E 'kube-apiserver|kube-controller-manager|kube-scheduler|etcd'
docker ps | grep -v pause | grep -E "etcd|scheduler|controller|apiserver" | awk '{print $1}' | xargs docker restart
查看`tail -f /var/log/syslog`
查看kubelet日志`journalctl -xeu kubelet`
~~~

在升级1.15的集群证书，查看kubelet状态，报错没有bootstrap-kubelet.conf，从从节点拷贝的，1.15集群在建立的时候主节点就没有这个文件（从其他1.15集群看的。）
这个命令用 CA （或者 front-proxy-CA ）证书和存储在 /etc/kubernetes/pki 中的密钥执行更新。
如果您运行了一个 HA 集群，这个命令需要在所有控制面板节点上执行。

# **针对kubeadm 1.13.0（不包含1.13.0） 以下处理**
移动证书和配置【注意！必须移动，不然会使用现有的证书，不会重新生成】
```
cd /etc/kubernetes
mkdir ./pki_bak
mkdir ./pki_bak/etcd
mkdir ./conf_bak
mv pki/apiserver* ./pki_bak/
mv pki/front-proxy-client.* ./pki_bak/
mv pki/etcd/healthcheck-client.* ./pki_bak/etcd/
mv pki/etcd/peer.* ./pki_bak/etcd/
mv pki/etcd/server.* ./pki_bak/etcd/
mv ./admin.conf ./conf_bak/
mv ./kubelet.conf ./conf_bak/
mv ./controller-manager.conf ./conf_bak/
mv ./scheduler.conf ./conf_bak/
```
创建证书
`kubeadm alpha phase certs all --apiserver-advertise-address=${MASTER_API_SERVER_IP} --apiserver-cert-extra-sans=主机内网ip,主机公网ip`

运行如上命令会重新生成以下证书
```
#-- /etc/kubernetes/pki/apiserver.key
#-- /etc/kubernetes/pki/apiserver.crt

#-- /etc/kubernetes/pki/apiserver-etcd-client.key
#-- /etc/kubernetes/pki/apiserver-etcd-client.crt

#-- /etc/kubernetes/pki/apiserver-kubelet-client.key
#-- /etc/kubernetes/pki/apiserver-kubelet-client.crt

#-- /etc/kubernetes/pki/front-proxy-client.key
#-- /etc/kubernetes/pki/front-proxy-client.crt

#-- /etc/kubernetes/pki/etcd/healthcheck-client.key
#-- /etc/kubernetes/pki/etcd/healthcheck-client.crt

#-- /etc/kubernetes/pki/etcd/peer.key
#-- /etc/kubernetes/pki/etcd/peer.crt

#-- /etc/kubernetes/pki/etcd/server.key
#-- /etc/kubernetes/pki/etcd/server.crt
```
不移动证书会有如下提示
```
#[certificates] Using the existing apiserver certificate and key.
#[certificates] Using the existing apiserver-kubelet-client certificate and key.
#[certificates] Using the existing front-proxy-client certificate and key.
#[certificates] Using the existing etcd/server certificate and key.
#[certificates] Using the existing etcd/peer certificate and key.
#[certificates] Using the existing etcd/healthcheck-client certificate and key.
#[certificates] Using the existing apiserver-etcd-client certificate and key.
#[certificates] valid certificates and keys now exist in "/etc/kubernetes/pki"
#[certificates] Using the existing sa key.
```
生成新配置文件
`kubeadm alpha phase kubeconfig all --apiserver-advertise-address=${MASTER_API_SERVER_IP}`
将新生成的admin配置文件覆盖掉原本的admin文件
```
mv $HOME/.kube/config $HOME/.kube/config.old
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
sudo chmod 777 $HOME/.kube/config
```
完成后重启kube-apiserver,kube-controller,kube-scheduler,etcd这4个容器
如果有多台master，则将第一台生成的相关证书拷贝到其余master即可。
# **bw的kubernetes1.12集群过期处理**
1. 备份/etc/kubernetes文件夹
2. 该集群是kubeadm直接搭建，但etcd集群是自建，不是kubeadm初始化的那种，所以执行更新证书的时候不能`renew all`必须分别执行下面三个命令
```
kubeadm alpha phase certs renew apiserver
kubeadm alpha phase certs renew apiserver-kubelet-client
kubeadm alpha phase certs renew front-proxy-client
```
3. 更新1.12集群的kubeconfig，这里更新需要删除原有的配置文件，不然会更新不成功
```
rm -rf admin.conf controller-manager.conf kubelet.conf scheduler.conf
```
4. 更新所有kubeconfig
```
kubeadm alpha phase kubeconfig all --apiserver-advertise-address=${MASTER_API_SERVER_IP}
```
5. 更新管理配置文件
`cp -av /etc/kubernetes/admin.conf /root/.kube/config`
6. 重启kube-apiserver,kube-controller-manager ,kube-scheduler这3个容器
`docker ps | grep -E 'kube-apiserver|kube-controller-manager|kube-scheduler'`
7. 重启kubelet
8. 如果有多台master，则将第一台生成的相关证书拷贝到其余master即可。