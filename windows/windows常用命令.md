services.msc 服务
control.exe 控制面板
gpedit.msc  本地组策略
regedit.exe 注册表
systeminfo
eventvwr.msc 日志
`netstat -ano | findstr 端口号`

# **更改系统区域设置**
*   Windows 8.1/Windows 10/Windows Server 2012/Windows Server 2012 R2/Windows 2016/Windows Server 2019：单击屏幕左下角的Windows徽标，单击k控制面板，然后单击区域。在区域对话框中，单击管理选项卡，然后单击非 Unicode 程序的语言下的**更改系统区域设置。**
# win2019启用telnet
`dism /online /Enable-Feature /FeatureName:TelnetClient`
# 普通用户开启修改hosts权限
https://blog.csdn.net/A_Bear/article/details/82916561
# 关闭windows update medic service
以管理员运行命令行cmd
`REG add “HKLM\\SYSTEM\\CurrentControlSet\\Services\\WaaSMedicSvc” /v “Start” /t REG\_DWORD /d “4” /f`
# 完全禁用win10自动更新
https://blog.csdn.net/weixin_44545251/article/details/101017904
# 卸载和删除Windows service
通过管理员权限运行CMD，输入如下命令
```
Net Stop ServiceName

sc delete ServiceName
```
其中 ServiceName为你要杀掉的服务名
上面如果还是没有用，那就继续尝试

找到系统注册表，删掉服务的注册表信息，通常路径在：`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services` 找到你的Service服务的名字，然后把整个文件夹删掉

# win10关闭安全中心和defender
用了一个软件关闭了defender
关闭安全中心
关闭security center
```
1、输入“regedit.exe”点击“确定”进入注册表编辑器。  
2、找到如下路径：计算机＼HKEY\_LOCAL\_MACHINESYSTEMCurrentControlSetServiceswscsvc  
3、双击start。  
4、可以看到数值数据为“2”（启动）。  
5、将“2”（启动）变更为“4”（禁用），点击“确定”。
```
