## tr命令实现字符转换功能，其功能类似于sed命令，但是，tr命令比sed命令简单
`tr [选项] 字符串1 字符串2 <输入文件`
tony@z6:~$ cat guoqi20200610.txt 
T000006,T000013,T000017,T000021,T000024,T000029,T000033,T000036
`cat guoqi20200610.txt | tr ',' '\n' > 2.txt`
`tr ',' '\n' < guoqi20200610.txt > 2.txt`
![](../../images/screenshot_1551925687318.png)

-d选项只需跟一个字符串，它表示删除字符串中出现的所有字符

-s选项用于删除所有重复出现的字符序列，只保留一个，即将重复出现的字符串压缩为一个字符


-c选项用于选定字符串1中字符集的补集，即反选字符串1中的字符集
tr命令也可以加上字符串1和字符串2，将字符串1用字符串2来替换

