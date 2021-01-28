*   [场景一：让镜像变成像命令一样使用]
*   [场景二：应用运行前的准备工作]
`ENTRYPOINT`的目的和`CMD`一样，都是在指定容器启动程序及参数。`ENTRYPOINT`在运行时也可以替代，不过比`CMD`要略显繁琐，需要通过`docker run`的参数`--entrypoint`来指定。
当指定了`ENTRYPOINT`后，`CMD`的含义就发生了改变，不再是直接的运行其命令，而是将`CMD`的内容作为参数传给`ENTRYPOINT`指令，换句话说实际执行时，将变为：
~~~
<ENTRYPOINT> "<CMD>"
~~~