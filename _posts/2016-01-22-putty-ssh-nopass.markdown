---
layout: post
title:  "实现putty的SSH无密码自动登陆"
date:   2016-01-22 14:21:39 +0800
categories: jekyll update
---



1.在服务器上生成秘钥对

	ssh-keygen -t rsa 

一路回车，不用输入密码。生成公私秘钥对。

可以看见当前目录下有两个文件，分别是 id_rsa  id_rsa.pub

现在把这个两个文件（.pub为公钥，另一个为私钥）


2.服务器上导入公钥

将公钥 id_rsa.pub 移到 ~/.ssh目录下，

将公钥输入到authorized_keys

	cat id_rsa.pub >> authorized_keys

题外话: 如果.ssh 不存在，请新建。同理，authorized_keys文件也可新建。

另外要注意. 目录，..目录 以及 authorized_keys的权限，这是我设置的权限.

.目录和..目录都为700,authorized_keys 文件为600


3.本地导入私钥

将第一步生成的私钥`id_rsa`复制到本地

使用puttygen.exe 制作private.ppk

打开puttygen.exe，点击file，Load private key



将本机上的id_rsa导入，


点击，Save private key

生成使用与putty使用的私钥对。假设生成的文件为private.ppk.


4.导入要登陆的会话中

先在session中load你的ip地址以及端口号:

然后 connect-> SSH->Auth 找到右测导入刚才我们生成的private.ppk



