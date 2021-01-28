使用国内镜像站
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
sudo curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
sudo tee /etc/apt/sources.list.d/kubernetes.list <<-'EOF'  
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main  
EOF
sudo apt-get update  
sudo apt-cache madison kubeadm
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo apt-mark hold package 标记该软件包不被自动更新
```
### 安装指定版本
```
apt -y install kubelet=1.15.4-00 kubeadm=1.15.4-00 kubectl=1.15.4-00
```
## **systemctl list-unit-files --type=service 查看服务是否开机自启**
```
root@saas-test:~# cat 1.txt 
kube-apiserver:v1.15.4
kube-controller-manager:v1.15.4
kube-scheduler:v1.15.4
kube-proxy:v1.15.4
pause:3.1
etcd:3.3.10
coredns:1.3.1
root@saas-test:~# cat 1.sh 
for imageName in $(cat 1.txt) ; do
        docker pull registry.aliyuncs.com/google_containers/$imageName
        docker tag registry.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
        docker rmi registry.aliyuncs.com/google_containers/$imageName
done
```
### 安装k8s
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 
```
### 安装网络插件
https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-kubernetes-api-datastore-50-nodes-or-less
### 单机版去除主节点污点
`kubectl taint nodes --all node-role.kubernetes.io/master-`
安装指定版本 --kubernetes-version v1.15.4