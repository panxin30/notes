```
Commit agent's Docker container
Run the build inside Docker containers 在Docker容器中运行构建
Define a Docker template 定义Docker模板	
GitHub 项目	
Groovy script to restrict where this project can be run	Groovy脚本，用于限制可以运行此项目的位置
This build requires lockable resources	此构建需要可锁定的资源
Throttle builds
丢弃旧的构建
参数化构建过程
Disable this project 禁用此项目
Execute concurrent builds if necessary 必要时执行并发构建
Restrict where this project can be run 限制此项目可以运行的位置
高级选项
Quiet period
Retry Count
Block build when upstream project is building
Block build when downstream project is building
Use custom workspace
Keep the build logs of dependencies
```
# 构建步骤
你需要向Jenkins解释下一件事是用源代码做什么。在自由模式的构建中，可以通过自定义构建步骤来做这个。构建步骤是Jenkins自由构建过程的基本模块。他们能让你准确地告知Jenkins，你到底想要如何构建项目。
构建作业可能包括一个或多个步骤，有时候可能没有步骤

















