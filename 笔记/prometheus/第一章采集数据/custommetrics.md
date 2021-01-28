### 概述
kubernetes的监控指标分为两种：
**Core metrics(核心指标)**：由kubelet、metrics-server以及由API server提供的api组成，仅提供Node和Pod的CPU累积使用率、内存实时使用率、Pod的资源占用率及容器的磁盘占用率，再由metrics-server提供给 Dashboard、HPA 控制器等使用。（核心指标不适合委托给第三方）
**Custom Metrics(自定义指标)**：由Prometheus Adapter提供API custom.metrics.k8s.io，自定义指标本身不能被k8s所解析，从其他数据源（如 Node Exporter 等）获取自定义指标，由此可支持任意Prometheus采集到的指标。
### **部署**
Prometheus可以采集其它各种指标，但是prometheus采集到的metrics并不能直接给k8s用，因为两者数据格式不兼容，因此还需要另外一个组件(kube-state-metrics)，将prometheus的metrics数据格式转换成k8s API接口能识别的格式，转换以后，因为是自定义API，所以还需要用Kubernetes aggregator在主API服务器中注册，以便直接通过/apis/来访问。
**Prometheus作为一个监控使用也作为一些特殊资源指标的提供者来使用，不过这些指标不是内建的标准核心指标，即自定义指标，要想把监控采集的数据转成指标格式需要个特殊的组件叫k8s-prometheus-adapter**
文件清单：
**node-exporter**：prometheus的export，收集Node级别的监控数据
**prometheus**：监控服务端，从node-exporter拉数据并存储为时序数据。
**kube-state-metrics**：将prometheus中可以用PromQL查询到的指标数据转换成k8s对应的数
**k8s-prometheus-adpater**：聚合进apiserver，即一种custom-metrics-apiserver实现

**关于k8s-prometheus-adapter**

**监控采集到的很多指标数据整合进k8s中，让k8s能用，作为判断，比如HPA判断是否需要伸缩pod规模的一个标准。**
