### 1.15后，集群安装方法有变动
### 阿里云香港ubuntu16.04安装k8s 1.11.2 HA版
## **自动生成的证书只能用一年。(有变动)**
`openssl x509 -in apiserver-etcd-client.crt -text -noout`
### **etcd**
组件负责存储和复制集群状态。 它具有很高的资源要求。 
因此，最佳实践是隔离etcd,通过将副本放在专用节点上。
# 架构说明
由于阿里云Ecs无法安装keepalived，我们采用阿里内部loadbalancer做master的负载，阿里内部loadbalancer无法由vpc内部ecs负载到自己机器，所以还需要另外两台服务器安装haproxy负载到三台master
```
172.16.1.166    k8s-test-01  master,haproxy
172.16.1.167    k8s-test-02  master,haproxy
172.16.1.168    k8s-test-03  master,haproxy
172.16.1.169    k8s-test-04  node
172.16.1.170    k8s-test-api loadblancer ip
```
# 准备工作
- master节点做ssh打通
- 配置hosts解析
```
#master,k8s-test-api 指向本机通过 haproxy 负载
cat >>/etc/hosts<<EOF
172.16.1.166    k8s-test-01
172.16.1.167    k8s-test-02
172.16.1.168    k8s-test-03
172.16.1.169    k8s-test-04
127.0.0.1       k8s-test-api
EOF
# node,k8s-test-api 指向 loadblancer ip
cat >>/etc/hosts<<EOF
172.16.1.166    k8s-test-01
172.16.1.167    k8s-test-02
172.16.1.168    k8s-test-03
172.16.1.169    k8s-test-04
127.16.1.170    k8s-test-api
EOF
```
# 安装docker
```
# 卸载安装指定版本docker-ce
yum remove -y docker-ce docker-ce-selinux container-selinux
#出处https://docs.docker.com/install/linux/docker-ce/centos/#os-requirements
#Centos 7
yum install --setopt=obsoletes=0 \
   docker-ce-17.03.2.ce-1.el7.centos.x86_64 \
   docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch
#ubuntu
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get -y update
apt-get  -y install docker-ce=17.03.1~ce-0~ubuntu-xenial
#
systemctl enable docker
systemctl start docker
```
# 配制haproxy
所有master都安装haproxy
```
#拉取haproxy镜像
docker pull haproxy:1.7.8-alpine
mkdir /etc/haproxy
cat >/etc/haproxy/haproxy.cfg<<EOF
global
  log 127.0.0.1 local0 err
  maxconn 50000
  uid 99
  gid 99
  #daemon
  nbproc 1
  pidfile haproxy.pid

defaults
  mode http
  log 127.0.0.1 local0 err
  maxconn 50000
  retries 3
  timeout connect 5s
  timeout client 30s
  timeout server 30s
  timeout check 2s

listen admin_stats
  mode http
  bind 0.0.0.0:1080
  log 127.0.0.1 local0 err
  stats refresh 30s
  stats uri     /haproxy-status
  stats realm   Haproxy\ Statistics
  stats auth    will:will
  stats hide-version
  stats admin if TRUE

frontend k8s-https
  bind 0.0.0.0:8443
  mode tcp
  #maxconn 50000
  default_backend k8s-https

backend k8s-https
  mode tcp
  balance roundrobin
  server lab1 172.16.1.166:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
  server lab2 172.16.1.167:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
  server lab3 172.16.1.168:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
EOF

#启动haproxy
docker run -d --name my-haproxy \
-v /etc/haproxy:/usr/local/etc/haproxy:ro \
-p 8443:8443 \
-p 1080:1080 \
--restart always \
haproxy:1.7.8-alpine

#查看日志
docker logs my-haproxy
```
# 安装 kubeadm, kubelet 和 kubectl
```
#centos7
#配置源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
#安装
yum install -y kubelet kubeadm kubectl ipvsadm
#ubuntu 16.04
apt-get update && sudo apt-get install -y apt-transport-https
curl -s http://packages.faasx.com/google/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://mirrors.ustc.edu.cn/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl ipvsadm
sudo apt-mark hold kubelet kubeadm kubectl ipvsadm #锁定服务，不更新
```
# 配置第一个master节点
```
#生成配置文件
CP0_IP="172.16.1.166"
CP0_HOSTNAME="k8s-test-01"
cat > kubeadm-config.yaml<<EOF
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.2
apiServerCertSANs:
- k8s-test-01
- k8s-test-02
- k8s-test-03
- k8s-test-api
- 172.16.1.166
- 172.16.1.167
- 172.16.1.168
- 172.16.1.170
- 127.0.0.1

api:
  controlPlaneEndpoint: k8s-test-api:8443

etcd:
  local:
    extraArgs:
      listen-client-urls: "https://127.0.0.1:2379,https://$CP0_IP:2379"
      advertise-client-urls: "https://$CP0_IP:2379"
      listen-peer-urls: "https://$CP0_IP:2380"
      initial-advertise-peer-urls: "https://$CP0_IP:2380"
      initial-cluster: "$CP0_HOSTNAME=https://$CP0_IP:2380"
    serverCertSANs:
      - $CP0_HOSTNAME
      - $CP0_IP
    peerCertSANs:
      - $CP0_HOSTNAME
      - $CP0_IP
controllerManagerExtraArgs:
  node-monitor-grace-period: 10s
  pod-eviction-timeout: 10s
networking:
    # This CIDR is a Calico default. Substitute or remove for your CNI provider.
    podSubnet: "192.168.0.0/16"  
kubeProxy:
  config:
    mode: ipvs
EOF
#初始化
#注意保存返回的 join 命令
kubeadm init --config kubeadm-config.yaml

#打包ca相关文件上传至其他master节点
cd /etc/kubernetes && tar cvzf k8s-key.tgz admin.conf pki/ca.* pki/sa.* pki/front-proxy-ca.* pki/etcd/ca.*
scp k8s-key.tgz k8s-test-02:~/
scp k8s-key.tgz k8s-test-03:~/
ssh k8s-test-02 'tar xf k8s-key.tgz -C /etc/kubernetes/'
ssh k8s-test-03 'tar xf k8s-key.tgz -C /etc/kubernetes/'
```
# 配置第二个master
```
#生成配置文件
CP0_IP="172.16.1.166"
CP0_HOSTNAME="k8s-test-01"
CP1_IP="172.16.1.167"
CP1_HOSTNAME="k8s-test-02"
cat > kubeadm-config.yaml<<EOF
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.2
apiServerCertSANs:
- k8s-test-01
- k8s-test-02
- k8s-test-03
- k8s-test-api
- 172.16.1.166
- 172.16.1.167
- 172.16.1.168
- 172.16.1.170
- 127.0.0.1
api:
  controlPlaneEndpoint: k8s-test-api:8443
etcd:
  local:
    extraArgs:
      listen-client-urls: "https://127.0.0.1:2379,https://$CP1_IP:2379"
      advertise-client-urls: "https://$CP1_IP:2379"
      listen-peer-urls: "https://$CP1_IP:2380"
      initial-advertise-peer-urls: "https://$CP1_IP:2380"
      initial-cluster: "$CP0_HOSTNAME=https://$CP0_IP:2380,$CP1_HOSTNAME=https://$CP1_IP:2380"
      initial-cluster-state: existing
    serverCertSANs:
      - $CP1_HOSTNAME
      - $CP1_IP
    peerCertSANs:
      - $CP1_HOSTNAME
      - $CP1_IP
controllerManagerExtraArgs:
  node-monitor-grace-period: 10s
  pod-eviction-timeout: 10s
networking:
    # This CIDR is a Calico default. Substitute or remove for your CNI provider.
    podSubnet: "192.168.0.0/16"  
kubeProxy:
  config:
    mode: ipvs
EOF

#配置kubelet
kubeadm alpha phase certs all --config kubeadm-config.yaml
kubeadm alpha phase kubelet config write-to-disk --config kubeadm-config.yaml
kubeadm alpha phase kubelet write-env-file --config kubeadm-config.yaml
kubeadm alpha phase kubeconfig kubelet --config kubeadm-config.yaml
systemctl restart kubelet

#添加etcd到集群中
CP0_IP="172.16.1.166"
CP0_HOSTNAME="k8s-test-01"
CP1_IP="172.16.1.167"
CP1_HOSTNAME="k8s-test-02"
KUBECONFIG=/etc/kubernetes/admin.conf kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP1_HOSTNAME} https://${CP1_IP}:2380
kubeadm alpha phase etcd local --config kubeadm-config.yaml

#部署
kubeadm alpha phase kubeconfig all --config kubeadm-config.yaml
kubeadm alpha phase controlplane all --config kubeadm-config.yaml
kubeadm alpha phase mark-master --config kubeadm-config.yaml
```
# 配制第三个master
```
#生成配置文件
CP0_IP="172.16.1.166"
CP0_HOSTNAME="k8s-test-01"
CP1_IP="172.16.1.167"
CP1_HOSTNAME="k8s-test-02"
CP2_IP="172.16.1.168"
CP2_HOSTNAME="k8s-test-03"
cat > kubeadm-config.yaml<<EOF
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.2

apiServerCertSANs:
- k8s-test-01
- k8s-test-02
- k8s-test-03
- k8s-test-api
- 172.16.1.166
- 172.16.1.167
- 172.16.1.168
- 172.16.1.170
- 127.0.0.1

api:
  controlPlaneEndpoint: k8s-test-api:8443

etcd:
  local:
    extraArgs:
      listen-client-urls: "https://127.0.0.1:2379,https://$CP2_IP:2379"
      advertise-client-urls: "https://$CP2_IP:2379"
      listen-peer-urls: "https://$CP2_IP:2380"
      initial-advertise-peer-urls: "https://$CP2_IP:2380"
      initial-cluster: "$CP0_HOSTNAME=https://$CP0_IP:2380,$CP1_HOSTNAME=https://$CP1_IP:2380,$CP2_HOSTNAME=https://$CP2_IP:2380"
      initial-cluster-state: existing
    serverCertSANs:
      - $CP2_HOSTNAME
      - $CP2_IP
    peerCertSANs:
      - $CP2_HOSTNAME
      - $CP2_IP

controllerManagerExtraArgs:
  node-monitor-grace-period: 10s
  pod-eviction-timeout: 10s
networking:
    # This CIDR is a Calico default. Substitute or remove for your CNI provider.
    podSubnet: "192.168.0.0/16"
kubeProxy:
  config:
    mode: ipvs
EOF

#配置kubelet
kubeadm alpha phase certs all --config kubeadm-config.yaml
kubeadm alpha phase kubelet config write-to-disk --config kubeadm-config.yaml
kubeadm alpha phase kubelet write-env-file --config kubeadm-config.yaml
kubeadm alpha phase kubeconfig kubelet --config kubeadm-config.yaml
systemctl restart kubelet

#添加etcd到集群中
CP0_IP="172.16.1.166"
CP0_HOSTNAME="k8s-test-01"
CP2_IP="172.16.1.168"
CP2_HOSTNAME="k8s-test-03"
KUBECONFIG=/etc/kubernetes/admin.conf kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP2_HOSTNAME} https://${CP2_IP}:2380
kubeadm alpha phase etcd local --config kubeadm-config.yaml

#部署
kubeadm alpha phase kubeconfig all --config kubeadm-config.yaml
kubeadm alpha phase controlplane all --config kubeadm-config.yaml
kubeadm alpha phase mark-master --config kubeadm-config.yaml
```
# 配置使用kubectl
```
rm -rf $HOME/.kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 配置实用网络插件（选择本地）
https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-kubernetes-api-datastore-50-nodes-or-less
#查看node节点
kubectl get nodes

#只有网络插件也安装配置完成之后，才能会显示为ready状态
#设置master允许部署应用pod，参与工作负载，现在可以部署其他系统组件
#如 dashboard, heapster, efk等
kubectl taint nodes --all node-role.kubernetes.io/master-
```
# 安装ingress-nginx
官网 https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md
```
# kubectl get po -n ingress-nginx
# kubectl logs -n ingress-nginx nginx-ingress-controller-6d466b5c9d-g9c62
已经用kubeadm自动部署了一套集群。
安装calico插件,再加入工作节点。
出处:https://github.com/kubernetes/ingress-nginx/tree/master/deploy
#The following resources are required for a generic deployment.
#集合rbac,default-backend,ingress一体
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml

# 验证default-backend
kubectl get service --all-namespaces
ingress-nginx   default-http-backend   ClusterIP   10.107.250.86   <none>         80/TCP          2h
curl 10.107.250.86
default backend - 404
```
# 安装dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
----------------------------------------无用
#
mkdir /root/certs
cd /root/certs
openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr
mv server.key server.key.org
openssl rsa -in server.key.org -out server.key
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
#重建Secret kubernetes-dashboard-certs,如果之前装过dashboard.
kubectl  delete secrets -n kube-system kubernetes-dashboard-certs
#导入证书
kubectl create secret generic kubernetes-dashboard-certs --from-file=/root/certs -n kube-system
------------------------无用
#创建管理用户 dashboard需要访问集群的权限，所以使用了clusterrolebinding
cat > admin-user.yaml<<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF
kubectl apply -f admin-user.yaml
#配置ingress
cat > dashboard-ingress.yaml<<EOF
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/secure-backends: "true"
spec:
  tls:
  - hosts:
    - k8s.dashboard.com
    secretName: kubernetes-dashboard-certs
  rules:
  - host: k8s.dashboard.com
    http:
      paths:
      - path:
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
EOF
kubectl apply -f dashboard-ingress.yaml
#获取token用于登录
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}'i)
# 本地绑定hostdashboard所在宿主机IP测试，开放443端口
```
