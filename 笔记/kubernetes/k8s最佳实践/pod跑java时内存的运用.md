使用java跑得容器老是重启，分析下容器中的java内存使用情况`VM.native_memory`
# jvm 性能调优工具之 jcmd，jdk1。7以后增加的
https://cloud.tencent.com/developer/article/1130026
`jcmd pid VM.native_memory summary scale=MB` 需要后端程序添加支持
目前给容器配置的内存是700M，可以看到堆就把内存用完了，堆外还有命令需要内存


可以看到整个memory主要包含了Java Heap、Class、Thread、Code、GC、Compiler、Internal、Other、Symbol、Native Memory Tracking、Arena Chunk这几部分；其中reserved表示应用可用的内存大小，committed表示应用正在使用的内存大小



