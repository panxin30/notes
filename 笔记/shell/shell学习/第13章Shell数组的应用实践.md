### **数组的定义**
**方法一：**
```
tony@z6:~$ array=(1 2 3 4 5) #用小括号将数组内容赋值给数组变量，数组元素用"空格"分隔
tony@z6:~$ echo ${array[*]} #输出上面定义的数组的所有元素
1 2 3 4 5
```
**方法二：**
```
tony@z6:~$ array=([1]=one [2]=two [3]=three [4]=four [5]=five)
tony@z6:~$ echo ${array[*]} 
one two three four five
tony@z6:~$ echo ${array[1]}
one
tony@z6:~$ echo ${array[2]}
two
```
**方法三：动态地定义数组变量，并使用命令的输出结果为数组的内容**
`array=($(命令)) 或者array=(`命令`)`
```
tony@z6:~$ mkdir array -p
tony@z6:~$ touch array/{1..5}.txt
tony@z6:~$ ls -l array/
总用量 0
-rw-rw-r-- 1 tony tony 0 5月   6 10:24 1.txt
-rw-rw-r-- 1 tony tony 0 5月   6 10:24 2.txt
-rw-rw-r-- 1 tony tony 0 5月   6 10:24 3.txt
-rw-rw-r-- 1 tony tony 0 5月   6 10:24 4.txt
-rw-rw-r-- 1 tony tony 0 5月   6 10:24 5.txt
tony@z6:~$ array=($(ls array/))
tony@z6:~$ echo ${array[*]} #使用"* " 或"@"打印整个数组的内容
1.txt 2.txt 3.txt 4.txt 5.txt
tony@z6:~$ echo ${array[0]}#打印单个数组元素${数组名[下标]}，未指定时，从0开始
1.txt
tony@z6:~$ echo ${array[1]}
2.txt
tony@z6:~$ echo ${array[2]}
3.txt
tony@z6:~$ echo ${#array[*]} # 获得数组长度
5
```
### **Shell数组的重要命令**
循环打印的基本语法
```
#!/bin/bash
arr=(
    10.0.0.11
    10.0.0.22
    10.0.0.33
)
#c-for循环语法
for ((i=0;i<${#arr[*]};i++))
do
    echo "${arr[$i]}"
done
echo "----------------------"
# 普通for循环语法
for n in ${arr[*]}
do
    echo "$n"
done
```