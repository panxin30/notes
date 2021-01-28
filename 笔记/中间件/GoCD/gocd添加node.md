引用[https://neveryu.github.io/2018/07/20/yarn/](https://neveryu.github.io/2018/07/20/yarn/)
在 Node 生态系统中，依赖通常安装在项目的**node_modules**文件夹中。然而，这个文件的结构和实际依赖树可能有所区别，因为重复的依赖可以合并到一起。`npm`客户端把依赖安装到`node_modules`目录的过程具有不确定性。这意味着当依赖的安装顺序不同时，`node_modules`目录的结构可能会发生变化。这种差异可能会导致类似“我的电脑上可以运行，别的电脑上不行”的情况，并且通常需要花费大量时间定为与解决。
1.服务器安装npm 未完成
2.使用npm 安装yarn，不全局安装，会在当前目录安装 未完成
3.将安装好的yarn拷贝到指定目录
`cp -r yarn/ /data1/ops/homego/node-v11.8.0-linux-x64/lib/node_modules`
4.软连接到node的bin下
`ln -s ../lib/node_modules/yarn/bin/yarn.js yarn`
1.下载需要的版本
`https://nodejs.org/en/download/releases/`
`cd /data1/ops/homego #改目录是挂载到k8s的gocd客户端的` 
`wget https://nodejs.org/download/release/v11.8.0/node-v11.8.0-linux-x64.tar.gz`
`tar zxf node-v11.8.0-linux-x64.tar.gz`