title: ubuntu14安装mysql
date: 2015-03-17 14:49:55
categories: mysql
tags: [ubuntu,mysql]
---

使用ubuntu14搭建Java项目的运行环境确实方便不少，下面就在Ubuntu14下搭建Mysql。  
输入 **sudo apt-get install mysql-server** 命令即可<!--more-->  

![](/img/mysql-install-1.png)

输入root密码

![](/img/mysql-install-2.png)

安装成功页面

成功安装之后输入：**mysql -u root -p** 登录mysql  


## Ubuntu下mysql安装目录

- /usr/bin               客户端程序和mysql_install_db
- /var/lib/mysql         数据库和日志文件
- /var/run/mysqld        服务器
- /etc/mysql             配置文件my.cnf
- /usr/share/mysql       字符集，基准程序和错误消息
- /etc/init.d/mysql      启动mysql服务器

## 卸载Mysql 
 sudo apt-get autoremove --purge mysql-server-5.5  
根据自己安装的版本进行卸载


## 更改默认字符集为 UTF8‘
为防止中文乱码，建议将数据库默认编码方式统一改为 utf8，默认是 latin1

	[mysql]
	default-character-set=utf8
	 
	[mysqld]
	character-set-server=utf8

## 添加启动服务

	cp support-files/mysql.server /etc/init.d/mysqld   
	update-rc.d mysqld defaults

移除启动服务： 

	update-rc.d -f mysqld remove

## 启动 MySQL 服务

	service mysqld start
	
	停止 MySQL 服务：
	service mysqld stop
	
	重启 MySQL 服务：
	service mysqld restart

## 设置 root 用户密码

mysqladmin -u root password 'root'

##  登录 MySQL
	
	mysql -u root –p

随后需输入 root 用户密码（root）

## 清理 MySQL 用户

	select host, user, password from mysql.user;
	delete from mysql.user where password = '';
	flush privileges;

为了提高安全性，建议删除所有空密码的用户，仅保留 root@localhost 一个用户，以后可针对不同的数据库，创建不同的用户。


## 远程访问 MySQL

若需要通过 root 用户远程访问 MySQL，则需要修改 mysql.user 表的 host 为“%”。
	
	update mysql.user set host = '%' where user = 'root';  
	flush privileges;
