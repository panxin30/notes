默认的延迟启动是120秒，基本上够了。
未验证
1、打开注册表regedit

2、找到已做成Windows服务bw-report-cronservice

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\bw-report-cronservice

3、添加参数AutoStartDelay（REG_DWORD）单位秒