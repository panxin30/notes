# 
**pom.xml > /home\_dir/.m2/settings.xml > /maven\_dir/conf/settings.xml 。**
但是settings定位不同,它倾向于提供一些公共的附属信息,而不是个性化的构建信息.它会尽量融合到你的pom中.
# 本地仓库

本机u s e r . h o m e / . m 2 
本地仓库地址可以在settings.xml里边指定
# 远程仓库

pom可以配置多个repository,如果好多项目共用的话,可以在settings文件配置profile,这样新项目就不需要重复配置repository了

# 中央仓库

maven必须至少知道一个远程仓库,中央仓库就是默认的远程仓库,不需要显示配置在maven的 super pom中配置的  
兜底用的,找不到的jar会找他

如果中央仓库慢可以用mirrors来替换它,它的id是central,在mirrorOf标签中配置它的标签就是替换了
在settings.xml中进行配置
```
<mirror>
  <id>alimaven</id>
  <name>aliyun maven</name>
  <mirrorOf>central</mirrorOf>
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror>

```
# 仓库在哪里配置

可以在settings OR pom.xml中配置  
可以嵌入到profile中,也可以单独通过repository配置然后profile通过id引用

# Servers是个什么东西

如果需要用户名和密码,就需要配置下server,不适合放在pom中,一般定义在settings中,由pom去引用

仓库的下载和部署是在pom.xml文件中的repositories和distributionManagement元素中定义的。

# server如何跟repository关联

该 id 与 distributionManagement 中 repository 元素的 id 相匹配

# .m2/setting.xml
```
    <server>
      <id>maven-releases</id>
      <username>admin</username>
      <password>test</password>
    </server>
    <server>
      <id>maven-snapshots</id>
      <username>admin</username>
      <password>test</password>
    </server>
  </servers>

<distributionManagement>
    <repository>
        <id>maven-releases</id>
        <name>Releases</name>
        <url>http://192.168.60.11:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>maven-snapshots</id>
        <name>Snapshot</name>
        <url>http://192.168.60.11:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```
# pom.xml
```
        <distributionManagement>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <name>User Project SNAPSHOTS</name>
            <url>http://192.168.60.11:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
        <repository>
            <id>maven-releases</id>
            <name>User Project Release</name>
            <url>http://192.168.60.11:8081/repository/maven-releases/</url>
        </repository>
    </distributionManagement>

    <repositories>
        <repository>
            <id>maven-releases</id>
            <name>nexus</name>
            <url>http://192.168.60.11:8081/repository/maven-releases/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
        </repository>
        <repository>
            <id>maven-snapshots</id>
            <name>nexus</name>
            <url>http://192.168.60.11:8081/repository/maven-snapshots</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
        </repository>
    </repositories>
```
# settings和pom之间的关系

settings偏向于全局配置  
一般pom的优先高于settings,但是他们之间的信息是相互引用的

# 依赖仓库的配置方式

*   中央仓库，这是默认的仓库
*   镜像仓库，通过 sttings.xml 中的 settings.mirrors.mirror 配置
*   全局profile仓库，通过 settings.xml 中的 settings.repositories.repository 配置
*   项目仓库，通过 pom.xml 中的 project.repositories.repository 配置
*   项目profile仓库，通过 pom.xml 中的 project.profiles.profile.repositories.repository 配置
*   本地仓库

# 通过pom中\<repositories>配置
**通过setting.xml方式配置会对所有maven项目生效**
**如果只想在本项目中配置一个maven仓库，可以通过在pom.xml中配置\<repositories>标签来实现。**

在自己的maven项目的pom.xml中添加如下配置，就配置好了一个仓库。这时候，maven会优先采用这个配置，而不会去读setting.xml中的配置了。这样配置好后，maven就会自动从aliyun下载jar包了。
```
<repositories>
  <repository>
    <id>aliyun-releases</id>
    <name>阿里云仓库(name可以随便起)</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </repository>
</repositories>

```

repositories标签下可以配置多个repository，maven会按出现顺序使用，如果第1个可用，就用第一个，如果不可用，就依次往下找
上面配置\<repository>时\<id>是给mirrorOf用的。

# maven仓库配置的其他选项
```
<!--releases和snapshots中有个enabled属性，是个boolean值，默认为true，
表示是否需要从这个远程仓库中下载稳定版本或者快照版本的构建，
一般使用第三方的仓库，都是下载稳定版本的构建。-->
<repository>
  <id>aliyun-releases</id>
  <url>https://maven.aliyun.com/repository/public</url>
  <releases>
    <enabled>true</enabled>
  </releases>
  <snapshots>
    <enabled>false</enabled>
  </snapshots>
</repository>
```


# maven默认的内置仓库的配置位置
```
<!--
可以从以下文件中找到maven仓库的默认配置如下,具体看你自己安装的版本
apache-maven-3.6.1\lib\maven-model-builder-3.6.1.jar\org\apache\maven\model\pom-4.0.0.xml
-->
<repositories>
  <repository>
   <id>central</id>
   <name>Central Repository</name>
   <url>https://repo.maven.apache.org/maven2</url>
   <layout>default</layout>
   <snapshots>
    <enabled>false</enabled>
   </snapshots>
  </repository>
</repositories>
```