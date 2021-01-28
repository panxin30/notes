PromQL 查询结果主要有 3 种类型：

*   瞬时数据 (Instant vector): 包含一组时序，每个时序只有一个点，例如：`http_requests_total`
*   区间数据 (Range vector): 包含一组时序，每个时序有多个点，例如：`http_requests_total[5m]`
*   纯量数据 (Scalar): 纯量只有一个数字，没有时序，例如：`count(http_requests_total)`

**聚合**
一般来说，如果描述样本特征的标签(label)在并非唯一的情况下，通过PromQL查询数据，会返回多条满足这些特征维度的时间序列。而PromQL提供的聚合操作可以用来对这些时间序列进行处理，形成一条新的时间序列：
`sum(http_request_total)`