title: CentOS上搭建Git Server
date: 2015-03-06 13:50:06
categories: git
tags: [git server,centos]
---

经过了一年，公司的项目终于采用了Git作为代码的版本管理，利用了一上午的时间终于弄完了，在这里做个记录供以后使用。<!--more-->


### 1.Git服务器安装准备
使用的Centos5的操作系统，git server 使用的git-1.9.0.tar.gz，。下面列出安装依赖的软件，需要提前安装。

- curl-devel
- expat-devel
- gettext-devel
- openssl-devel
- zlib-devel
- perl-devel(perl)

### 2.安装Git

基于上面的依赖软件都安装完后，下面开始安装git服务器，

*wget http://git-core.googlecode.com/files/git-1.9.0.tar.gz* 下载

将下载后git 解压并进入，使用下面命令进行安装。  

	#make prefix=/usr/local all
	#make prefix=/usr/local install

使用**git --version**查看是否安装成功。


### 3.安装gitosis

gitosis为Git用户权限管理系统,通过管理服务端的/home/git/.ssh/authorized_key文件来执行对用户权限的管理，是一个python模块包。  

使用下面命令执行安装
	
	#yum install python python-setuptools
	#git clone git://github.com/res0nat0r/gitosis.git
	#cd gitosis/
	#python setup.py install

*显示Finished processing dependencies for gitosis==0.2即表示成功*


### 4.Git客户端安装

git客户端使用的基于Window上的mygit,下载**[mygit](https://msysgit.github.io/ "mygit")**并安装。

**客户端密钥生成**  

	ssh-keygen -t rsa
在安装好的mygit操作窗口中输入上面的命令生成密钥，生成的是一个id_rsa.pub文件，将生成的密钥上传到服务器中，上传存放目录可以随便定义。

**Git服务端设置**


*创建Git用户*  

	useradd -r -s /bin/sh -c 'git version control' -d /home/git git

*创建Git仓库目录*  

	mkdir -p /home/git
	chown git:git /home/git

*在服务器端生成管理库*

上面已经创建了用户及对应的home目录，下面我们切换到git用户。
	
	su - git
	gitosis-init < ～/id_rsa.pub


创建成功提示信息：  

Initialized empty Git repository in /home/git//repositories/gitosis-admin.git/ Reinitialized existing Git repository in /home/git/repositories/gitosis-admin.git/ 

执行完后会在/home/git下创建两个目录：

![](/img/gitosis.png)

注：  

1. 生成的gitosis-admin为Git的用户访问权限管理库，gitosis通过这个git库来管理所有git库的访问权限。
2. 通过执行初始化，该公钥的拥有者就能修改用于配置gitosis的那个特殊Git仓库了


### 5.Gitosis管理

上面完成了gitosis及仓库的初始化，下面我们来看看如果使用gitosis进行git的用户权限管理。

在mygit中使用如下命令：
	
	git clone git@192.168.56.1:gitosis-admin.git
将gitosis-admin 下载到本地，在gitosis-admin主要有*keydir*目录和*gitosis.conf*配置文件。

注：

1. gitosis.conf文件用来设置用户、仓库和权限的控制文件，

2. keydir目录则是保存所有具有访问权限用户公钥的地方./keydir/root@vm1.pub:如前所述，该用户具有访问权限，如果新增的开发人员，只需要把生成的密钥放到该目录中就行了。


**权限控制配置文件讲解**

	[gitosis]
	
	[group gitosis-admin]
	writable = gitosis-admin
	members = kris@KRIS-PC
	
	[group test-git] ##组名称
	writable = test-git ##项目名称
	members = kris@KRIS-PC hexun@HEXUN-PC ##项目成员名称
	
	[group testq]
	writable = testq
	members = kris@KRIS-PC hexun@HEXUN-PC




**初始、增加及使用项目git-test** 

	cd /git-repo
	mkdir git-test
	cd git-test
	git init
	touch README
	git add .
	git commit -a -m "init git-test"
	git remote add origin git@192.168.56.1:git-test.git
	git push origin master

注：在新项目git-test里首次推送数据到服务器前，需先设定该服务器地址为远程仓库，但你不用事先到服务器上手工创建该项目的裸仓库— Gitosis 会在第一次遇到推送时自动创建



注意：  
1.**keydir 目录下的pub文件名与gitsois.conf中的用户一致**

2.**gitosis.conf文件清单：**

	[gitosis]

	[group gitosis-admin]
	writable = gitosis-admin
	members = kris@KRIS-PC hexun@HEXUN-PC shenmiao1314@SHENMIAO
	
	
	[group test-git]
	writable = test-git
	members = kris@KRIS-PC hexun@HEXUN-PC
	
	[group testq]
	writable = testq
	members = kris@KRIS-PC hexun@HEXUN-PC
	
	[group hx-nlp]
	writable = hx-nlp
	members = kris@KRIS-PC hexun@HEXUN-PC shenmiao1314@SHENMIAO

3.~/.ssh/authorized_keys 文件清单


	ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA2wFjIZCRESZri1B9GOLJJvOvidB6KykHqQFJSCLQbattGBtD1paFmOaH2l51O2TdEkFVSdqtU5hfdF+QoqxaitcL3uFnAdIsOdYcfi51l5OhnyoZTSfol6SQnZHphv5VV+iMJU9G0UoBjOOdliBZ/oK3T/kPhOcBIM748hDI0GQUJjYuV658WsYUfG+ZgRWA6MgDjbviMYKUtaaZagFhlVRroS/za/NuqNiyoydTneYGastJkMko2AO3dTs4eKrrrf3wPxSNJ6Z6ZDmZavmrepYDOZMTDAfFx/MoaxQ+jf9hhHDkTArCYuybhx+bZM3/9yydZT29ian0uucpTdVfrQ== hexun@HEXUN-PC
	### autogenerated by gitosis, DO NOT EDIT
	command="gitosis-serve kris@KRIS-PC",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDB8RJWPD3P5Dd1f6QonOxSJ3EEvqrJvdKtSyx2xMNnIn1tTk6oYOw4dphJFCLzuYe0GffqSHIJRlP7IxfXTZ+FS0ebLD0bljD7Ir9ywjFscd3paLPofrASwBW7TxJlq8etcdEb4nn8z3+0d4YpgFhOfCkqcnTOxPPwQxYmJCWWfAs/Wo8v0A70LXhO90FoyYWzzo8NAHG10WIXyyAGj9yBlheTZQf3vb8VKhcAFoWpKOoz8NvLOSll5s6RzVKyzU387EA+vG5t6DUMsuB1ZHZrkd/9ZmOYCPZ9i9o1OkZ4vr7mlSa9WPpejTMAZMvA5PPhEvpiyAn+QPE95W+Otxvn kris@KRIS-PC
	command="gitosis-serve hexun@HEXUN-PC",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA81Afc+I0WIeFXlf/ZN04gtQKDT++SCcoX/9uukCuMFPqaGvH+27xi9HwXDgfGl2xQ/sIE1vfaTVsrMaZDFN9BiqUQ5TWKppaNZomvflGUTiIvDZsC5Is/UGpsaQuCOPyUT8ygzOAX2E0zHi2iKo8dWN9e0JE5rVH5zd+B87W0x5Spe14BbdoWt9prNTR96ZtHRMTZQBK7pakKhmZ1mmuB/vd9sh6Lo1QmRvLrEQRfb3glETDUMzu4mrH88qoM+MgHDo4kc+wFnjztnxFmsxv8Vt+5Lo4JTpk4ANALU+MoCG2ifpUbPxhjS6eT34hC/17jB+HkJZTcVBlbY8nLWcgQQ== hexun@HEXUN-PC
	command="gitosis-serve shenmiao1314@SHENMIAO",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAwbld5Mo/va+R/vQE2ITdo/QMa1zpLWPJKSoMhShizIwopgW3UQgSi05YoaJvM1tEgT+iFpOTyu+ckvlDOmqwStk42HJh/b8zBKKvsfSqY5HNeV0VJLdsmZ3Wh00M+CNvZ0n91pp4bvgLGuED+sFm9OiKR5bvpdN2ui7KL2t3jYyk8jTdlgoj5IoQGt1E23nu0EBPuhFBzIp1wlrfdth3KlZa0wlM1+BxA3NtTNH2hJZtZdFBb6+Z33aDC4eo9yQSyIeBmVGDSUKcnHfa6N5Qwly8sapZnETAN4fUpSsc8wKpYackDGbHsWupPx5s5vFYz8/hpqzPbJm9pDIQD7ky5w== shenmiao1314@SHENMIAO

	
	                                   