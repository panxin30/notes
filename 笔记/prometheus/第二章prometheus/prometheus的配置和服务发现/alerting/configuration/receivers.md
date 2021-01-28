Receiver是一个或多个通知集成的命名配置。
~~~
# The unique name of the receiver.
name: <string>

# Configurations for several notification integrations.
详细见官网receivers
email_configs:
  [ - <email_config>, ... ]
hipchat_configs:
  [ - <hipchat_config>, ... ]
pagerduty_configs:
  [ - <pagerduty_config>, ... ]
pushover_configs:
  [ - <pushover_config>, ... ]
slack_configs:
  [ - <slack_config>, ... ]
opsgenie_configs:
  [ - <opsgenie_config>, ... ]
webhook_configs:
  [ - <webhook_config>, ... ]
victorops_configs:
  [ - <victorops_config>, ... ]
wechat_configs:
  [ - <wechat_config>, ... ]
~~~

例子
```
    receivers:
    - name: 'ops'
      webhook_configs:
      - url: 'http://dingtalk-hook.monitoring.svc.cluster.local:5000'
        send_resolved: true
```

