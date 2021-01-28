#### 语法格式：

httpd \[-D name\] \[-d directory\] \[-f file\]  
         \[-C "directive"\] \[-c "directive"\]  
         \[-w\] \[-k start|restart|stop|shutdown\]  
         \[-k install|config|uninstall\] \[-n service\_name\]  
         \[-v\] \[-V\] \[-h\] \[-l\] \[-L\] \[-t\] \[-T\] \[-S\]

#### 参数选项： 
\-l  
输出一个静态编译在服务器中的模块的列表。它不会列出使用LoadModule指令动态加载的模块。  
\-t  
仅对配置文件执行语法检查。程序在语法解析检查结束后立即退出，或者返回"0"(OK)，或者返回非0的值(Error)。如果还指定了"-D DUMP\_VHOSTS"，则会显示虚拟主机配置的详细信息。  
\-v  
显示httpd的版本，然后退出。  
\-V  
显示httpd和APR/APR-Util的版本和编译参数，然后退出。  

**YUM方式安装mod\_ssl.so模块**  
`yum -y install mod_ssl`