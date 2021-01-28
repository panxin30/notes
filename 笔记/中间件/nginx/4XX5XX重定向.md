fastcgi_intercept_errors on;
http{}或
server{}中都行
这个指令指定是否允许自定义4xx和5xx错误信息，默认情况下，nginx不支持自定义错误页面，只有这个指令被设置为on，nginx才能将错误自定义！

```
server {
...
error_page   404 500 https://www.test.cn;
...
}
```