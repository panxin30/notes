新建目标组，可以选择是关联实例ID或者是关联IP
目标组配置运行检查 http / 80,这个实际就是curl http://ip:80/ 实际就到了nginx的默认页面default，如果没有这个就会显式404，同时运行检查不通过
ALB只有443和80两个端口
ALB监听443端口，关联到目标组，目标组指向具体ip的80端口
---
如果目标组仅包含运行状况不佳的注册目标，则负载均衡器将请求路由到所有这些目标，而不考虑这些目标的运行状况。这意味着，如果在所有已启用的可用区中，所有目标都未通过运行状况检查，则负载均衡器将在失败时开放。失败时开放的效果是根据负载均衡算法，允许传输到所有已启用的可用区中的所有目标的流量，而不考虑这些目标的运行状况。**未经验证**