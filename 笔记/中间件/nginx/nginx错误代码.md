*   对于3xx，重定向，301表示，请求的网页已永久移动到新位置，302表示，临时性重定向，303表示临时性重定向，且总是使用 GET 请求新的 URI。304表示，自从上次请求后，请求的网页未修改过。
*   **对于4xx，客户端错误**。404，服务器无法理解请求的格式，客户端不应当尝试再次使用相同的内容发起请求，401，请求未授权，403，禁止访问，404，找不到如何与 URI 相匹配的资源。
*    **对于5xx，服务器错误**。500 未知错误，502 网关错误，503 服务器超载，可能是停机或维护
504(网关超时)  服务器作为网关或代理，但是没有及时从上游服务器收到请求。(都还没到服务器)

**400 Bad Request**
1、语义有误，当前请求无法被服务器理解。除非进行修改，否则客户端不应该重复提交这个请求。
2、请求参数有误。
**401 Unauthorized**
当前请求需要用户验证。该响应必须包含一个适用于被请求资源的 WWW-Authenticate 信息头用以询问用户信息。
**402 Payment Required**
该状态码是为了将来可能的需求而预留的。
**403 Forbidden**
在网站访问过程中，常见的错误提示，表示资源不可用。 服务器理解客户的请求，但拒绝处理它。没权限
**404 Not Found**
意味着链接指向的网页不存在，即原始网页的URL失效。没这个文件


