DevOps这个词，其实就是Development和Operations两个词的组合。它的英文发音类似于“迪沃普斯”。得(de)沃普斯

### devops平台搭建工具项目管理（PM）：jira。运营可以上去提问题，可以看到各个问题的完整的工作流，待解决未解决等；
### 代码管理：gitlab。jenkins或者K8S都可以集成gitlab，进行代码管理，上线，回滚等；
### 持续集成CI（Continuous Integration）：gitlab ci。开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。
### 持续交付CD（Continuous Delivery）：gitlab cd。完成单元测试后，可以把代码部署到连接数据库的 Staging 环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境中。
### 镜像仓库：VMware Harbor，私服nexus。
### 容器：Docker。
### 编排：K8S。
### 服务治理：Consul。
### 脚本语言：Python。
### 日志管理：Cat+Sentry，还有种常用的是ELK。
### 系统监控：Prometheus。
### 负载均衡：Nginx。
### 网关：Kong，zuul。
### 链路追踪：Zipkin。
### 产品和UI图：蓝湖。
### 公司内部文档：Confluence。
### 报警：推送到工作群。