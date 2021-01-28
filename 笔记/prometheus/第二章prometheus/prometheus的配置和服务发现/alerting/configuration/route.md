**route**
路由块定义路由树中的节点及其子节点。如果未设置，则其可选配置参数将从其父节点继承。

每个警报都会在配置的顶级路由中进入路由树，该路由必须匹配所有警报（即没有任何已配置的匹配器）。然后它遍历子节点。如果`continue`设置为false，则在第一个匹配的子项后停止。如果`continue`在匹配节点上为true，则警报将继续与后续兄弟节点匹配。如果警报与节点的任何子节点都不匹配（没有匹配的子节点，或者不存在），则根据当前节点的配置参数处理警报。
~~~
route:
[ receiver: <string> ] 
# 默认的receiver，如果某条告警没有被一个route匹配，则发送给默认的接收器

# The labels by which incoming alerts are grouped together. For example,
# multiple alerts coming in for cluster=A and alertname=LatencyHigh would
# be batched into a single group.
# 这里的标签列表是接受到告警信息后的重新分组标签,
# To aggregate by all possible labels use the special value '...' as the sole label name, for example:
# group_by: ['...'] 
# This effectively disables aggregation entirely, passing through all 
# alerts as-is. This is unlikely to be what you want, unless you have 
# a very low alert volume or your upstream notification system performs 
# its own grouping.
[ group_by: '[' <labelname>, ... ']' ]

# Whether an alert should continue matching subsequent sibling nodes.
# 警报是否应继续匹配后续兄弟节点
[ continue: <boolean> | default = false ]

# A set of equality matchers an alert has to fulfill to match the node.
# 一组平等匹配警报必须满足以匹配节点
match:
  [ <labelname>: <labelvalue>, ... ]

# A set of regex-matchers an alert has to fulfill to match the node.
# 一组正则表达式匹配器必须满足警报以匹配节点
match_re:
  [ <labelname>: <regex>, ... ]

# How long to initially wait to send a notification for a group
# of alerts. Allows to wait for an inhibiting alert to arrive or collect
# more initial alerts for the same group. (Usually ~0s to few minutes.)
# 最初等待为一组警报发送通知的时间。 允许等待禁止警报到达或收集同一组的更多初始警报
# 在一个新的告警分组被创建后，等待至少group_wait时间来初始化通知，这种方式可以确保有足够的时间为同一分组获取多条告警，然后一起触发这条警报。
[ group_wait: <duration> | default = 30s ]

# How long to wait before sending a notification about new alerts that
# are added to a group of alerts for which an initial notification has
# already been sent. (Usually ~5m or more.)
# 在第一条警报信息发送后，等待group_interval时间来发送新的一组警报信息。
[ group_interval: <duration> | default = 5m ]

# How long to wait before sending a notification again if it has already
# been sent successfully for an alert. (Usually ~3h or more).
# 如果某条警报信息已经发送成功，则等待repeat_interval时间重新发送它们。
[ repeat_interval: <duration> | default = 4h ]

# Zero or more child routes.
# 上面的所有属性都由所有子路由继承，并且可以在每个子路由上覆盖。
routes:
  [ - <route> ... ]
~~~

#### 示例

~~~
# The root route with all parameters, which are inherited by the child
# routes if they are not overwritten.
route:
  receiver: 'default-receiver'
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  group_by: [cluster, alertname]
  # All alerts that do not match the following child routes
  # will remain at the root node and be dispatched to 'default-receiver'.
  routes:
  # All alerts with service=mysql or service=cassandra
  # are dispatched to the database pager.
  - receiver: 'database-pager'
    group_wait: 10s
    match_re:
      service: mysql|cassandra
  # All alerts with the team=frontend label match this sub-route.
  # They are grouped by product and environment rather than cluster
  # and alertname.
  - receiver: 'frontend-pager'
    group_by: [product, environment]
    match:
      team: frontend
~~~
[https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml](https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml)
```
global:
  # The smarthost and SMTP sender used for mail notifications.
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'
  # The auth token for Hipchat.
  hipchat_auth_token: '1234556789'
  # Alternative host for Hipchat.
  hipchat_api_url: 'https://hipchat.foobar.org/'

# The directory from which notification templates are read.
templates: 
- '/etc/alertmanager/template/*.tmpl'

# The root route on which each incoming alert enters.
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s
  # When the first notification was sent, wait 'group_interval' to send a batch
  # of new alerts that started firing for that group.
  group_interval: 5m
  # If an alert has successfully been sent, wait 'repeat_interval' to
  # resend them.
  repeat_interval: 3h 

  # A default receiver
  receiver: team-X-mails

  # All the above attributes are inherited by all child routes and can 
  # overwritten on each.

  # The child route trees.
  # 子路由树
  routes:
  # This routes performs a regular expression match on alert labels to
  # catch alerts that are related to a list of services.
# 此路由在警报标签上执行正则表达式匹配，以捕获与服务列表相关的警报。
  - match_re:
      service: ^(foo1|foo2|baz)$
    receiver: team-X-mails
   # The service has a sub-route for critical alerts, any alerts
   # that do not match, i.e. severity != critical, fall-back to the
   # parent node and are sent to 'team-X-mails'
    routes:
    - match:
        severity: critical
      receiver: team-X-pager
  - match:
      service: files
    receiver: team-Y-mails

    routes:
    - match:
        severity: critical
      receiver: team-Y-pager

  # This route handles all alerts coming from a database service. If there's
  # no team to handle it, it defaults to the DB team.
# 此路由处理来自数据库服务的所有警报。 如果没有团队可以处理它，则默认为数据库团队。
  - match:
      service: database
    receiver: team-DB-pager
   # Also group alerts by affected database.
    group_by: [alertname, cluster, database]
    routes:
    - match:
        owner: team-X
      receiver: team-X-pager
      continue: true
    - match:
        owner: team-Y
      receiver: team-Y-pager


# Inhibition rules allow to mute a set of alerts given that another alert is firing.
# We use this to mute any warning-level notifications if the same alert is already critical.
inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  # Apply inhibition if the alertname is the same.
  equal: ['alertname', 'cluster', 'service']


receivers:
- name: 'team-X-mails'
  email_configs:
  - to: 'team-X+alerts@example.org'

- name: 'team-X-pager'
  email_configs:
  - to: 'team-X+alerts-critical@example.org'
  pagerduty_configs:
  - service_key: <team-X-key>

- name: 'team-Y-mails'
  email_configs:
  - to: 'team-Y+alerts@example.org'

- name: 'team-Y-pager'
  pagerduty_configs:
  - service_key: <team-Y-key>

- name: 'team-DB-pager'
  pagerduty_configs:
  - service_key: <team-DB-key>
  
- name: 'team-X-hipchat'
  hipchat_configs:
  - auth_token: <auth_token>
    room_id: 85
    message_format: html
    notify: true
```
