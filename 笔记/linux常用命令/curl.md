time curl 可以看出用l多少时间

`curl -X POST https://prod-uploadpic-nei.lwork.com:8443`
`curl ipinfo.io`


实例
```
curl -H 'X-Api-Token: PUB:T002414:a7ac7105-ab6d-47df-b3c5-86b858be2290:68de2c70-0d41-450a-9559-6c7fdec484a5' -H 'Origin:https://broker.cxmtrading.com' -H 'content-type': "application/json" --data '{"conditions":[{"condition":"EQUALS","field":"status","logicType":"AND","value":"failed"},{"condition":"BETWEEN","field":"time","logicType":"AND","value":["2020-10-19 00:00","2020-10-19 23:59"]}],"keyword":"","lang":"","ownerType":"all","pageSize":20,"pager":1,"reportId":"","serverId":"3329","sortingColumn":"","sortingDirection":"","userId":6452}' -X POST 'http://172.17.128.212:31103/api/v3/report/list/RCR_REPORT'
```
# 返回状态码
`curl -I http://consul.tools.lwork.com/` 
# 
# 一、基础请求  
1、Get 请求  
命令格式：`curl requesturl`  
例如：`curl https://kunpeng.csdn.net/ad/template/161?positionId=427`

2、Post 请求  
命令格式：`curl -X POST requesturl`  
例如：`curl -X POST https://msg.csdn.net/v1/web/message/view/unread`

# 二、指定ip发送请求  
1、http命令格式：  
`curl -H 'Host:requestHost' http://ip:port/requestPath`  
或`curl -x ip:port http://requestHost/requestPath`  
例如：  
`curl -H 'Host:kunpeng.csdn.net' http://101.201.173.208:80/ad/template/161?positionId=427`  
`curl -x '101.201.173.208:80' http://kunpeng.csdn.net/ad/template/161?positionId=427`

2、https命令格式：  
`curl -H 'Host:requestHost' https://ip/requestPath`  
或`curl ip https://requestHost/requestPath -k`  
例如：  
`curl -H 'Host:kunpeng.csdn.net' https://101.201.173.208/ad/template/161?positionId=427 -k`  
`curl '101.201.173.208' https://kunpeng.csdn.net/ad/template/161?positionId=427 -k`
# 三、带参数的POST请求  
命令格式：

```
curl -X POST https://requestHost/requestPath 
-H 'headKey: headValue'
--data "dataValue"
```

1、**head请求参数用 -H表示**（一个横杆）  
2、如果命令需要换行，在换行处加 反斜杠  
3、**body请求参数用 --data**表示（两个横杆）；请求内容有引号时，加反斜杠\\

例如：

```
curl --http1.0 --next --no-keepalive -X POST "https://www. domain.com/requestUri" \
    -H 'Content-Type: application/json' \
    -H 'User-agent: test' \
    -H "token: tokenValue" \
 --data "{\"jsons\":[{\"id\":\"1\",\"value\":1}],\"type\":\"M\",\"name\":\"fei\"}"
```

