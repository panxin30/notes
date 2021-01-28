官网：[https://github.com/helm/helm](https://github.com/helm/helm)
              helm.sh
![](../../images/screenshot_1565838579374.png)
当前版本，3.1.2

**helm把k8s资源打包到一个chart中，制作并测试完成各个chart和chart本身的依赖关系
并利用chart仓库对外进行分发
还实现可配置发布，通过values文件完成配置和发布
同时支持应用程序版本管理，简化k8s部署的版本控制，如果你的chart更新，helm自动支持滚动更新机制，还支持一键回滚。**

**官方可用的Chart列表**:
https://hub.kubeapps.com/
### **安装helm**
wget https://get.helm.sh/helm-v3.1.2-linux-amd64.tar.gz
tar zxf helm-v3.1.2-linux-amd64.tar.gz
cd linux-amd64
cp helm /usr/bin/
### 
helm repo add stable https://charts.helm.sh/stable
helm search repo stable
helm常用命令:
**release管理**:
install
delete
upgrade/rollback
list
history:release的历史信息;
status:获取release状态信息; `helm status name`
**chart管理**:
create
pull 下载一个chart包
get
show `helm inspect stable/jenkins` `helm search memcached`
package
verify
lint #**,**
helm repo update              # Make sure we get the latest list of charts
### **安装和使用**
go语言在后端服务器，并发编程开发方面
requirements.yaml这个chart的依赖，会自动下载依赖，也可以手动放在charts这个目录中
values.yaml主要为templates中的模板中的自定义值提供默认值

{{.values.imageRegistry}} {{.values.dockerTag}}
**顶级字段，在values.yaml中顶格写**



