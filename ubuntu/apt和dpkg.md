update的作用是从/etc/apt/source.list文件中定义的源中去同步包的索引文件，即运行这个命令其实并没有更新软件，而是相当于windows下面的检查更新，获取的是软件的状态。
而upgrade则是更据update命令同步好了的包的索引文件，去真正地更新软件。

## ubuntu 16.0.4 改名字
hostname 名字         临时
vim /etc/hostname

## centos 7 改名字
hostnamectl set-hostname 名字

## yum (rpm) 和 apt-get的对应关系

![](../images/screenshot_1552272816637.png)
查看dpkg的帮助。
选择 dpkg -l来查看软件的状态。
dpkg --remove只是删除安装的文件，但不删除配置文件。而dpkg --purge则安装文件和配置文件都删除。
###  **apt 常用命令：**
  list - 根据名称列出软件包
  search - 搜索软件包描述
  show - 显示软件包细节
  install - 安装软件包
  remove - 移除软件包
  autoremove - 卸载所有自动安装且不再使用的软件包
  update - 更新可用软件包列表
  upgrade - 通过 安装/升级 软件来更新系统
  full-upgrade - 通过 卸载/安装/升级 来更新系统
  edit-sources - 编辑软件源信息文件
### **确认文件位置**
# dpkg -L命令
*   如果已知软件使用apt-get install命令安装的，可以使用这种方法