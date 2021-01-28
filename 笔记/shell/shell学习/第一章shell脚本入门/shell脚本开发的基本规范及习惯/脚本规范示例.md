## Bash 老司机也可能忽视的 10 大编程细节 
https://www.leiphone.com/news/201703/i49ztcRDDymM7Id5.html
## 1\. 等号两边慎用空格
一般常用的 Bash 变量都是字符串，很少见到有数组的。另外，虽然解释器也接受小写，但 Bash 中默认是将变量名全部大写的。
## 2\. 用 ${} 限定变量名
为了避免出现问题，最好的办法是每次引用时都在变量两边加上括号
## 3\. 区分全局变量、局部变量和环境变量
Bash 有三种变量：全局变量、局部变量和环境变量。其中最常用的是环境变量。
实际上每个 Linux 进程都有许多预设的环境变量（运行 env 命令可查看），Bash 中对环境的变量的应用非常简单。例如，想要查看 MYVAR 环境变量的值，可以运行下面这条命令：
> echo "$MYVAR"

想要设置环境变量，可以用这条命令：
> export MYVAR=2

需要注意的是，一旦在进程中设置了环境变量，那么这个环境变量会在所有与其相关的子进程中生效，例如下面这个例子：
> export MYVAR=2; python test.py

$MYVAR 环境变量也会在 test.py 脚本中生效。
另一种是全局变量，如下所示这样的赋值语句实际上就是在定义全局变量：
> MYVAR=2

全局变量就像其他编程语言一样，会在整个代码中生效。
最后一种是局部变量，这种变量通常只在一个循环语句或者 Bash 函数中有效。一般不常用。
## 4\. 活用命令替换
```
for i in `seq 1 10`; do echo $i; done
```
通过反引号（即键盘上Tab键上方的按键，注意不是单引号）将 seq 命令的输出结果，嵌入了 for 循环中直接使用。通过类似这种命令替换的方式，我们可以大大减少代码冗余，同时减少代码的出错几率。常见的替换方式有如下两种：
```
OUTPUT=`command`
or
OUTPUT=$(command)
```
## 5\. if 的注意事项
if 语句的判定条件同时支持单中括号（\[\]）和双中括号（\[\[\]\]），他们都可以用来隔离表达式和 if 关键词。但这里推荐使用双中括号，因为它的容错率更高，而且支持更多功能。
*****
除了使用双中括号之外，还可以用 test 命令的运行结果作为 if 语句的判断条件，例如：
> test -e /tmp/awesome.txt

如果 awesome.txt 文件存在，则命令返回 0，否则返回错误码。
实际上，除了常见的 test 命令，所有返回固定数值的命令都可以作为 if 语句的判断条件。例如下面的代码：
> if grep peanuts food-list.txt
> then
> echo "allergy allert!"
> fi
利用 grep 搜索关键词，然后根据结果打印警告信息。
## 7\. 用双引号引用变量
## 8\. 关于返回值
> create\_user && make\_home\_directory

这条语句，只有 create\_user 返回 0 时，才会执行 make\_home\_directory。
而
> create\_user; make\_home\_directory

则表示无论 create\_user 的返回值是什么，都会执行 make\_home\_directory。
> create\_user || make\_home\_directory

表示只有当 create\_user 返回非 0 值时，才会执行 make\_home\_directory。
## 9\. 使用后台任务
在 Bash 中，可以通过在命令后添加 & 符号实现后台多任务。例如：
> long\_running\_command &

把进程放入后台后，还可以通过 fg 命令将其切换到前台。如果后台命令过多，可以先通过 jobs 命令查看进程的 job ID，然后用 fg+job ID 的方式将指定的后台进程切换到前台。
另外，还可以通过 wait 命令控制多任务的执行顺序。例如：
> long\_running\_command1
> wait
> long\_running\_command2

表示在命令 1 执行结束后才执行命令 2。
## 10\. 活用 set 命令
对 Bash 环境进行一些额外设置，-e 表示出现错误就停止。
类似的，在其他语言中，使用没初始化的变量也会报错，但 Bash 不会。例如下面的代码：
`rm -rf "$DIRECTORY/\*"` 
如果 $DIRECTORY 没有提前初始化，Bash 也并不会停下来，而是直接以空字符串对待，那么这句命令的含义就变成了：尝试删除根目录下的所有文件，结果将非常严重。
这时就可以用 
> set -u

表示 Bash 不执行未定义的变量。
还有
> set -x 

表示每条命令执行之前必须先打印命令内容。此外还可以通过 set -o 显示所有可以设置的选项。
