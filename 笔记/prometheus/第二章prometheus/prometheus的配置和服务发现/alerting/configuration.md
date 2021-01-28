**全局配置**
[Alertmanager](https://github.com/prometheus/alertmanager)通过命令行标志和配置文件进行配置。虽然命令行标志配置了不可变的系统参数，但配置文件定义了禁止规则，通知路由和通知接收器。
[可视化编辑器](https://prometheus.io/webtools/alerting/routing-tree-editor)可以帮助构建路由树。

如果想要查看所有命令，请使用命令`alertmanager -h`。

`Alertmanager`能够在运行时动态加载配置文件。如果新的配置有错误，则配置中的变化不会生效，同时错误日志被输出到终端。通过发送`SIGHUP`信号量给这个进程，或者通过HTTP POST请求`/-/reload`，Alertmanager配置动态加载到内存。

提供的[有效示例文件](https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml)显示上下文中的用法。

全局配置指定在所有其他配置上下文中有效的参数。它们还可用作其他配置节的默认值。
```
global:
  # ResolveTimeout is the time after which an alert is declared resolved
  # if it has not been updated.
  [ resolve_timeout: <duration> | default = 5m ]

  # The default SMTP From header field.
  [ smtp_from: <tmpl_string> ]
  # The default SMTP smarthost used for sending emails, including port number.
  # Port number usually is 25, or 587 for SMTP over TLS (sometimes referred to as STARTTLS).
  # Example: smtp.example.org:587
  [ smtp_smarthost: <string> ]
  # The default hostname to identify to the SMTP server.
  [ smtp_hello: <string> | default = "localhost" ]
  [ smtp_auth_username: <string> ]
  # SMTP Auth using LOGIN and PLAIN.
  [ smtp_auth_password: <secret> ]
  # SMTP Auth using PLAIN.
  [ smtp_auth_identity: <string> ]
  # SMTP Auth using CRAM-MD5. 
  [ smtp_auth_secret: <secret> ]
  # The default SMTP TLS requirement.
  [ smtp_require_tls: <bool> | default = true ]

  # The API URL to use for Slack notifications.
  [ slack_api_url: <secret> ]
  [ victorops_api_key: <secret> ]
  [ victorops_api_url: <string> | default = "https://alert.victorops.com/integrations/generic/20131114/alert/" ]
  [ pagerduty_url: <string> | default = "https://events.pagerduty.com/v2/enqueue" ]
  [ opsgenie_api_key: <secret> ]
  [ opsgenie_api_url: <string> | default = "https://api.opsgenie.com/" ]
  [ hipchat_api_url: <string> | default = "https://api.hipchat.com/" ]
  [ hipchat_auth_token: <secret> ]
  [ wechat_api_url: <string> | default = "https://qyapi.weixin.qq.com/cgi-bin/" ]
  [ wechat_api_secret: <secret> ]
  [ wechat_api_corp_id: <string> ]

  # The default HTTP client configuration
  [ http_config: <http_config> ]

# Files from which custom notification template definitions are read.
# The last component may use a wildcard matcher, e.g. 'templates/*.tmpl'.
# 从中读取自定义通知模板定义的文件。 最后一个组件可以使用通配符匹配器
templates:
  [ - <filepath> ... ]

# The root node of the routing tree. 路由树的根节点
route: <route>

# A list of notification receivers. 通知接收者列表.
receivers:
  - <receiver> ...

# A list of inhibition rules. 禁止规则列表
inhibit_rules:
  [ - <inhibit_rule> ... ]
```