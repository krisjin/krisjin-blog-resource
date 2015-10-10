title: Mysql性能优化检查
date: 2015-10-08 15:05:44
categories: mysql
tags: [mysql]
---


# 1. 寻找运行缓慢的SQL语句
在mysql系统或者终端运行以下命令：

	show full processlist



# 2. 生成一个查询执行计划（Query Execution Plan,QEP）
当Mysql要执行一个SQL查询的时候，它首先会对该SQL语句进行语法检查，然后构造一个QEP，QEP决定了Mysql从底层存储引擎获取信息方式。如果想要查看Mysql查询优化器为SQL语句构造的QEP，需要在select语句前加上前缀explain:

	explain select * from t_advert where id =100000\G