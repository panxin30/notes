**在开启了 TLS 的集群中，每当与集群交互的时候少不了的是身份认证，使用 kubeconfig（即证书） 和 token 两种认证方式是最简单也最通用的认证方式**
1、部署:
`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-
flannel.yml`
2、将Service改为NodePort,或者用ingress
kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kube-system
3、认证:
认证时的账号必须为ServiceAccount:被dashboard pod拿来由kubernetes进行认证;
**token登录**:
(1)创建ServiceAccount,根据其管理目标,使用rolebinding或clusterrolebinding绑定至合理role或
clusterrole;
(2)获取到此ServiceAccount的secret,查看secret的详细信息,其中就有token;
1. 集群权限
`kubectl create serviceaccount dashboard-admin -n kube-system`
`kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin`
`kubectl describe secret dashboard-admin-token-44k7c`
2. 单个名称空间权限
`kubectl create serviceaccount def-ns-admin -n default`
`kubectl create rolebinding def-ns-admin --clusterrole=admin --serviceaccount=default:def-ns-admin`
`kubectl describe secret def-ns-admin-token-44k7c`

**kubeconfig登录**: 把ServiceAccount的token封装为kubeconfig文件
(1)创建ServiceAccount,根据其管理目标,使用rolebinding或clusterrolebinding绑定至合理role或
clusterrole;
(2)kubectl get secret | awk '/^ServiceAccount/{print $1}'
KUBE_TOKEN=$(kubectl get secret SERVCIEACCOUNT_SERRET_NAME -o jsonpath={.data.token} |
base64 -d)
(3)生成kubeconfig文件
kubectl config set-cluster --kubeconfig=/PATH/TO/SOMEFILE
kubectl config set-credentials NAME --token=$KUBE_TOKEN --kubeconfig=/PATH/TO/SOMEFILE
kubectl config set-context
kubectl config use-context



























        