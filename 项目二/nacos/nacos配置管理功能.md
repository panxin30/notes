## **各微服务统一从Nacos Server中获取各自的配置，并监听配置的变化。**
一、准备Nacos服务

二、新建一个SpringBoot项目：yl-nacos-comfig

三、在pom中添加nacos配置中心的依赖

四、为我们的项目创建bootstrap.yml配置文件，并添加如下的配置

五、在Nacos的配置管理里添加一个配置

六、读取配置

七、不同环境读取不同配置

八、指定命名空间

九、读取多个配置

# 四、为我们的项目创建bootstrap.yml配置文件，并添加如下的配置
```
spring:
  application:
    name: yl-nacos-config
  cloud:
    nacos:      
      config:        
        server-addr: localhost:8849
        file-extension: yaml
spring:
  application:
    name: service-examination
  cloud:
    nacos:
      discovery:
        server-addr: @config.addr@
        namespace: @config.ns@
        cluster-name: @config.cluster@
      config:
        server-addr: @config.addr@
        namespace: @config.ns@
        cluster-name: @config.cluster@
        prefix: ${spring.application.name}
        file-extension: yml
        shared-configs:
        - data-id: common.yml
          refresh: true

```
# 说明：

spring.application.name 项目名称

spring.cloud.nacos.config.server-addr 配置Nacos服务的地址。从这个地址的配置中心获取配置

spring.cloud.nacos.config.file-extension 配置中心数据的内容格式

# Note：

spring-cloud-starter-alibaba-nacos-config 在加载配置的时候，**加载了以 dataid 为 spring.application.name.{file-extension:properties} 的基础配置**。对应以上的配置，它会去 nacos server中加载data id为service-examination的配置集。 

# Note: 

若没有指定spring.cloud.nacos.confifig.group配置,则默认为DEFAULT_GROUP。

# 五、在Nacos的配置管理里添加一个配置
```
DataID的命名规则：${prefix}-${spring.profiles.active}.${file-extension}

${prefix}默认是spring.application.name的值，也可以使用spring.cloud.nacos.config.prefix来配置。

${spring.profiles.active}当前对应环境的profile。如果该项配置没有值，连前面的小横杠-也没有。

${file-extension}配置内容的数据格式，目前支持properties、yaml/yml，通过spring.cloud.nacos.config.file-extension配置。
```
-------------------------------------------------------------------
# 动态更新配置
	可以通过配置spring.cloud.nacos.config.refresh.enabled=false/true来关闭或开启动态更新配置
# 自定义namespace配置
	再没有指定namespace时，默认使用的是nacos上public这个ns
	必须放在bootstrap.yml中
```
spring:
  cloud:
    nacos:
	  config:
	    namespace: id#在nacos控制台获取
```
# 自定义Groupid配置
	在没有指定dataid的配置下，默认使用的是DEFAULT_GROUP
	必须放在bootstrap.yml中
	增加配置时Group的值一定要和spring.cloud.nacos.config.group配置的值一样
```
spring:
  cloud:
    nacos:
	  config:
	    group: DEFAULT_GROUP
```
# 自定义DataId
```
spring:
  application:
    name: service2
  cloud:
    nacos:
	  config:
	    server-addr: 127.0.0.1:8848
# 1. data id在默认组，不支持配置的动态刷新
		ext-config[0]:
		  data-id: test-config1.yml
# 2. data id不在默认组，不支持配置的动态刷新
		ext-config[1]:
		  data-id: test-config2.yml
		  group: TEST_GROUP
# 3. data id不在默认组，也支持动态刷新
		ext-config[2]:
		  data-id: test-config3.yml
		  group: DEV_GROUP
		  refresh: true
```
通过测试得出以下结果：
	1. spring.cloud.nacos.config.ext-config[n].data-id可以获取多个DataId的配置
	2. spring.cloud.nacos.config.ext-config[n].group可以获取自定义DataId所在的组，默认使用的是DEFAULT_GROUP。
	3. spring.cloud.nacos.config.ext-config[n].refresh用来动态刷新对应的DataId的配置
# 自定义共享DataId配置
```
spring:
  cloud:
    nacos:
      config:
        file-extension: yml
        shared-dataids: common.yml #多个文件支持用逗号隔开，优先级后面高于前面，必须带文件扩展名
          refreshable-dataids: common.yml
```
# nacos目前提供三种获取配置的方式
	1. spring.cloud.nacos.config.shared-dataids获取多个共享配置
	2. spring.cloud.nacos.config.ext-config[n].data-id通过配置多个config[n]来拉取多个配置
	3. 通过内部相关规则(应用名、扩展名)获取对应的DataId的配置
三者共存时候，优先级3>2>1


