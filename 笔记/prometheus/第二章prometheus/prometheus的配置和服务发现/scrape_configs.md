
**scrape_configs**
`scrape_config`:定义收集规则。 其中每一个scrape_config对象对应一个数据采集的Job，每一个Job可以对应多个Instance，即配置文件中的targets。 在高级配置中，这可能会改变。
目标可以通过`<static_configs>`参数静态配置，也可以使用其中一种支持的服务发现机制动态发现。
此外，`<relabel_configs>`允许在抓取之前对任何目标及其标签进行高级修改。
其中`<job_name>`在所有scrape配置中必须是唯一的。
```
# The job name assigned to scraped metrics by default.
# 默认分配给已抓取指标的job名称。
job_name: <job_name>

# How frequently to scrape targets from this job.
# 从job中抓取目标的频率.
[ scrape_interval: <duration> | default = <global_config.scrape_interval> ]

# Per-scrape timeout when scraping this job.
# 抓取此job时，每次抓取超时时间.
[ scrape_timeout: <duration> | default = <global_config.scrape_timeout> ]

# The HTTP resource path on which to fetch metrics from targets.
# 从目标获取指标的HTTP资源路径.
[ metrics_path: <path> | default = /metrics ]

# honor_labels controls how Prometheus handles conflicts between labels that are
# already present in scraped data and labels that Prometheus would attach
# server-side ("job" and "instance" labels, manually configured target
# labels, and labels generated by service discovery implementations).
#
# If honor_labels is set to "true", label conflicts are resolved by keeping label
# values from the scraped data and ignoring the conflicting server-side labels.
#
# If honor_labels is set to "false", label conflicts are resolved by renaming
# conflicting labels in the scraped data to "exported_<original-label>" (for
# example "exported_instance", "exported_job") and then attaching server-side
# labels.
#
# Setting honor_labels to "true" is useful for use cases such as federation and
# scraping the Pushgateway, where all labels specified in the target should be
# preserved.
#
# Note that any globally configured "external_labels" are unaffected by this
# setting. In communication with external systems, they are always applied only
# when a time series does not have a given label yet and are ignored otherwise.
# honor_labels控制Prometheus如何处理已经存在于已抓取数据中的标签与Prometheus将附加服务器端的标签之间的冲突（"job"和"instance"标签，手动配置的目标标签以及服务发现实现生成的标签）。
# 如果honor_labels设置为"true"，则通过保留已抓取数据的标签值并忽略冲突的服务器端标签来解决标签冲突。
# 如果honor_labels设置为"false"，则通过将已抓取数据中的冲突标签重命名为"exported_ <original-label>"（例如"exported_instance"，"exported_job"）然后附加服务器端标签来解决标签冲突。 这对于联合等用例很有用，其中应保留目标中指定的所有标签。
# 请注意，任何全局配置的"external_labels"都不受此设置的影响。 在与外部系统通信时，它们始终仅在时间序列尚未具有给定标签时应用，否则将被忽略。
[ honor_labels: <boolean> | default = false ]

# honor_timestamps controls whether Prometheus respects the timestamps present
# in scraped data.
#
# If honor_timestamps is set to "true", the timestamps of the metrics exposed
# by the target will be used.
#
# If honor_timestamps is set to "false", the timestamps of the metrics exposed
# by the target will be ignored.
[ honor_timestamps: <boolean> | default = true ]

# Configures the protocol scheme used for requests.
# 配置用于请求的协议方案.
[ scheme: <scheme> | default = http ]

# Optional HTTP URL parameters.
# 可选的HTTP URL参数.
params:
  [ <string>: [<string>, ...] ]

# Sets the `Authorization` header on every scrape request with the
# configured username and password.
# password and password_file are mutually exclusive.
# 使用配置的用户名和密码在每个scrape请求上设置`Authorization`标头。 password和password_file是互斥的。
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# Sets the `Authorization` header on every scrape request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file`.
[ bearer_token: <secret> ]

# Sets the `Authorization` header on every scrape request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token`.
[ bearer_token_file: /path/to/bearer/token/file ]

###############################################
###############################################
# Configures the scrape request's TLS settings.
# 配置scrape请求的TLS设置.
tls_config:
  [ <tls_config> ]
# 用于验证API服务器证书的CA证书。
[ ca_file: <filename> ]
# 用于服务器的客户端证书身份验证的证书和密钥文件。
[ cert_file: <filename> ]
[ key_file: <filename> ]
# ServerName扩展名，用于指示服务器的名称。
# https://tools.ietf.org/html/rfc4366#section-3.1
[ server_name: <string> ]
# 禁用服务器证书的验证。
[ insecure_skip_verify: <boolean> ]
###############################################
###############################################


# Optional proxy URL.
# 可选的代理URL.
[ proxy_url: <string> ]

# List of Kubernetes service discovery configurations.
# Kubernetes服务发现配置列表。
kubernetes_sd_configs:
  [ - <kubernetes_sd_config> ... ]

# List of labeled statically configured targets for this job.
# 此job的标记静态配置目标列表。
static_configs:
  [ - <static_config> ... ]

# List of target relabel configurations.对服务发现的目标进行重新标记
# 目标重新标记配置列表。
relabel_configs:
  [ - <relabel_config> ... ]

# List of metric relabel configurations.抓取目标后，被保存之前。可以确定哪些指标需要保存或丢弃
# 度量标准重新配置列表。
metric_relabel_configs:
  [ - <relabel_config> ... ]

# Per-scrape limit on number of scraped samples that will be accepted.
# If more than this number of samples are present after metric relabelling
# the entire scrape will be treated as failed. 0 means no limit.
# 对每个将被接受的样本数量的每次抓取限制。
# 如果在度量重新标记后存在超过此数量的样本，则整个抓取将被视为失败。 0表示没有限制。
[ sample_limit: <int> | default = 0 ]

#还有一些其他配置如：自动发现下列服务
azure_sd_configs:
consul_sd_configs:
dns_sd_configs:
ec2_sd_configs:
openstack_sd_configs:
file_sd_configs:
gce_sd_configs:
marathon_sd_configs:
nerve_sd_configs:
serverset_sd_configs:
triton_sd_configs:
``` 

`static_configs`参数进行静态配置
**static_configs**
```
    scrape_configs:
    - job_name: prometheus
      static_configs:
      - targets:
        - localhost:9090
```
，也可以使用其中一种受支持的服务发现机制进行动态发现。
