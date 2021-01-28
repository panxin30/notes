### tar 本职只做归档(不压缩)，但可调用gzip对归档结果进行压缩
参数
-c 创建tar包  `tar zcf test.tar.gz 目标文件或目录`
-z 调用gzip
-j 调用 bzip2
-x 解包
-t 查看 `tar ztf test.tar.gz` 查看里面的文件
-C 指定解压位置
-f 使用归档文件
-- remove 打包后删除原文件
-- newer-mtime 2020-03-19备份该日期以后的

tar zcf brokerwork.tgz t_* 
`t_account_base_info t_certificates_info t_financial_info #三个文件夹`
tar zxf brokerwork.tgz 解压的时候报错
```
gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now
```
先用tar xf brokerwork.tgz
再用tar zxf brokerwork.tgz
