### 1、启用：-XX:NativeMemoryTracking=detail
```
-XX:NativeMemoryTracking=[off | summary | detail] 

# off: 默认关闭 

# summary: 只统计各个分类的内存使用情况. 

# detail: Collect memory usage by individual call sites.

打开NMT会带来5%-10%的性能损耗
```