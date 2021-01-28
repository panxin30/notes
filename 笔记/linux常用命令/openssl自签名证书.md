# 第一种方式
1.通过openssl生成私钥
`(umask 077; openssl genrsa -out tony.key 2048)`
2. 使用私钥生成自签名的cert证书文件
`openssl req -new -x509 -days 3650 -key tony.key -out tony.crt -subj "/C=CN/ST=GD/L=SZ/O=vihoo/OU=dev/CN=domain/EmailAddress=yy@vivo.com"`
如果对上面参数具体的说明不太了解的，可以使用不带参数的方式，通过命令行步骤生成，参考第二种方式。
# 第二种方式
1.通过openssl生成私钥
`(umask 077; openssl genrsa -out tony.key 2048)`
2. 根据私钥生成证书申请文件csr
`openssl req -new -key tony.key -out tony.csr`
这里根据命令行向导来进行信息输入
3. 使用私钥对证书申请进行签名从而生成证书
`openssl x509 -req -in tony.csr -out tony.crt -signkey tony.key -days 3650`

### **以上生成得到的tony.crt证书，格式都是pem的。**

