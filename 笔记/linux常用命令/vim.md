```
:g/^\s*$/d
简单解释一下：
g ：全区命令
/ ：分隔符
^\s*$ ：匹配空行，其中^表示行首，\s表示空字符，包括空格和制表符，*重复0到n个前面的字符，$表示行尾。连起来就是匹配只有空字符的行，也就是空行。
/d ：删除该行

:%s/\n/ /g
/r 是换行不是/n

linux中方法：
1、%s/xx/\r/g
2、%s/xx/^M/g(^M的输入方法是：先按CTRL+V，松开然后按回车键）
# 去掉行尾的所有空格。
:%s/ \*$//
```
## **整篇文章大写转化为小写**
  打开文件后，无须进入命令行模式。键入:ggguG 
解释一下：ggguG分作三段gg gu G
gg=光标到文件第一个字符
gu=把选定范围全部小写
G=到文件结束
## 查看每行结尾是否有空格或乱码
`set invlist`

## linux下查看文件编码格式
```
root@ali-hz-crm3-k8s-master01:~# file /tmp/product.js
/tmp/product.js: ISO-8859 text, with CRLF line terminators
root@ali-hz-crm3-k8s-master01:~# file /tmp/id.js
/tmp/id.js: UTF-8 Unicode (with BOM) text, with CRLF line terminators
```
## 文件编码转换
1.在Vim中直接进行转换文件编码,比如将一个文件转换成utf-8格式
`:set fileencoding=utf-8`
2.enconv 转换文件编码，比如要将一个GBK编码的文件转换成UTF-8编码，操作如下
`enconv -L zh_CN -x UTF-8 filename`