1.   当shell脚本运行时，它会先查找系统环境变量ENV，该变量指定了环境文件（**加载顺序**通常是/etc/profile,~/.bash_profile,~/.bashrc,/etc/bashrc等），在加载了上述变量文件后，shell开始执行shell脚本中的内容
  shell脚本是**从上到下，从左到右依次执行**每一行的命令及语句，如果在shell脚本中遇到了子脚本（脚本嵌套），会先执行子脚本的内容，完成后再返回父脚本继续执行父脚本内后续的命令及语句。
  通常情况下，在执行shell脚本时，**会向系统内核请求启动一个新进程**，以便在该进程中执行脚本的命令及子shell脚本
2. shell脚本的执行通常可以采用一下几种方式
    1）bash script-name 或sh script-name推荐使用
    2）给脚本加x位，容易忘记加权限
    3）source script-name 或  . script-name：读入或加载指定shell脚本（如san.sh），然后，依次执行指定shell脚本文件san.sh中的所有语句。这些语句将在当前父shell脚本father.sh进程中运行（其他几种模式都会启动新的进程执行子脚本），因此使用source或 . 可以将san.sh自身脚本中的变量值或函数等的返回值传递到当前父shell脚本father.sh中使用
```
cat testsource.sh
userdir=`pwd`
sh testsource.sh
echo $userdir # 结果为空
#################
source testsource.sh
echo $userdir
/home/oldboy
```



















