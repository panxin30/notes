## github访问不了，大概率是DNS被污染了
## 如何查域名对应IP？
需要查以下三个域名：
*   `github.com`
*   `assets-cdn.github.com`
*   `github.global.ssl.fastly.net`

# 查找域名映射IP
进入https://www.ipaddress.com/  -->点击 IP Address Lookup
查询对应ip写死在hosts中

# 如果还访问不了
就进入 https://tool.chinaz.com/dns/ 输入域名github.com
选择一个速度最快的ip重新修改hosts
