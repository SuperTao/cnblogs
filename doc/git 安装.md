每次安装git都要找资料花时间，将过程记录一下，方便以后使用。

###### git安装

1.windows

https://git-for-windows.github.io/

2.Linux

ubuntu

　　apt-get install git

fedora

　　yum install git-core

###### 参考文档

http://www.cnblogs.com/monodin/p/3268679.html

https://git-scm.com/book/zh/v1/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git

###### 配置

1.配置Name和Email

git config --global user.name "your name"

git config --global user.email "your email address" 

2.生成Public/Private RSA Key

ssh-keygen -C "your email address" -t rsa

3.将生成的ssh秘钥添加到github上

c:\users\username\.ssh\id_rsa.pub