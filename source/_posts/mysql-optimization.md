title: "性能优化之MySQL优化"
date: 2015-04-21 16:25:05
categories: mysql
tags:
---


## 1.使用命令 

	show variables like 'slow_query_log';  
查看到当前没有开启慢查询

## 2.使用命令

show variables like '%log%';  
也没有开启 log_queries_not_using_indexs

## 3.set global log_queries_not_using_indexes=on;

## 4.show variables like 'long_query_time'; 

查看到long_query_time的值为10，意思是慢查询日志中会记录超过十秒的记录;<!--more-->  

默认情况下，MySQL是不会记录超过一定执行时间的SQL语句的。要开启这个功能，我们需要修改MySQL的配置文件，windows下修改my.ini,Linux  下修改my.cnf文件,在[mysqld]最后增加如下命令：

复制代码 代码如下:  

	slow_query_log		
	long_query_time = 1

## 5.开启慢查询日志

	set global slow_query_log=on;  
导入mysql官方提供的sakila数据库：

将文件解压，得到三个文件，

打开cmd：使用命令 mysql -uroot -p < "sakila-schema.sql所在的路径"  

使用命令 mysql -uroot -p < "sakila-data.sql所在的路径"  

## 6.mysql 慢查询日志分析工具

	mysqldumpslow  
	pt-query-digest

## 7.如何通过慢查询日志发现有问题的SQL

**查询次数多且每次查询所占用的时间长的SQL**  
 
> 通常为pt-query-digest分析的前几个查询

**IO大的SQL**

>注意pt-query-digest分析中的Rows exammine项

**未命中索引的SQL** 
>注意pt-query-digest分析中的Rows	examine和Rows Send的对比

## 8.使用explain查询SQL的执行计划

**explain返回各列的含义**

> - **table**：显示这一行的数据是关于哪张表的  
- **type**：这是重要的列，显示连接使用了何种类型。从最好到最差的连续类型为const,eq_reg,ref,range,index和ALL(，没有where从句，表扫描)。   
- **possible_keys**:显示可能应用在这张表中的索引。如果为空，没有可能的索引  
- **key**:实际使用的索引。如果为NULL，则没有使用索引  
- **key_len**:使用的索引的长度。在不损失精确性的情况下，长度越短越好  
- **ref**：显示索引的哪一列被使用了，如果可能的话，是一个常数  
- **rows**：mysql认为必须检查的用来返回请求数据的行数  

## 9.Max()的优化方法

**查询最后支付时间----优化MAX()函数**

	explain select max(payment_date) from payment;  
可以查看到：type:ALL是表扫描操作;没有任何索引，rows非常大，IO效率非常低。

**如何优化这个SQL：通常情况下建立索引：**

	create index idx_paydate on payment(payment_date);

这样在次执行

	explain select max(payment_date) from payment;  

可以看到Extra：select tables optimized away;可以通过索引进行操作，大大减少了IO操作

## 10.Count()的优化方法

在一条SQL中同时查出2006年和2007年电影的数量----优化count()函数

错误方式：

	SELECT COUNT(release_year='2006' OR release_year='2007') FROM film;

无法分开计算2006和2007年的电影数量

	select count(*) FROM film WHERE release_year='2006' AND release_year='2007';

release_year不可能同时为2006和2007，因此有逻辑错误

正确的方式：

	SELECT COUNT(release_year='2006' OR NULL) AS '2006年电影数量',COUNT(release_year='2007' OR NULL) AS '2007年电影数量' FROM film;


现在说说count(*)和count(id)的区别:

> 新建一张表：create table t(id int);

>插入数据：insert into t values (1),(2),(null);

>当我们使用命令：select count(*),count(id) from t;

>显示出count(*)为3，而count(id)为2;

>说明count(*)包含了null的，count(id)不包含值为null的



## 11.子查询的优化

- 通常情况下，需要把子查询优化为join查询，但在优化时要注意关联键是否有一对多的关系，要注意重复数据。
我们新建一张表t1  
- create table t1(tid int);

插入一条数据insert into t1 values(1);

进行子查询：

	select * from t where t.id in (select * from t1.tid from t1);

返回t表id在t1表中的数据

- 优化成join的形式：  

		select t.id from t join t1 on t.id=t1.tid;

这两种形式返回的结果是一样的

- 需要注意的是：
如果在t1表中，添加一条数据：insert into t1 values(1);

然后在分别执行这两种形式的查询：

发现使用select * from t where t.id in (select * from t1.tid from t1);查询出来的结果是一条数据。而select t.id from t join t1 on t.id=t1.tid;是两条数据，说明有重复数据，我们可以使用distinct去重

	select distinct t.id from t join t1 on t.id=t1.tid;这样就返回一条记录了

## 12.优化group by查询

	explain SELECT actor.first_name,actor.last_name,count(*) FROM sakila.film_actor INNER JOIN sakila.actor USING(actor_id) GROUP BY film_actor.actor_id;  

using可用在join语句相同字段连接，起到和ON相同作用，inner join 和left join中都可以使用

示例：LEFT JOIN 正常写法：

	SELECT t1.id,t2.name FROM t1 LEFT JOIN t2 ON t1.id=t2.id WHERE ....  

其实也可以这么写：

	SELECT t1.id,t2.name FROM t1 LEFT JOIN t2 USING(id) WHERE ....


上面的explain执行结果，可以看到extra为Using temporary，Using filesort  用到了临时表,文件排序的方式对表进行了全表扫描.

我们应该避免这种方式。改写如下：

	Explain SELECT actor.first_name,actor.last_name,c.cnt  FROM sakila.actor INNER JOIN(SELECT actor_id,COUNT(*) AS cnt FROM sakila.film_actor GROUP BY actor_id) AS c USING(actor_id);
## 13.优化limit查询

limit常用于分页处理，时常会伴随order by从句使用，因此大多时候会使用Filesorts，这样会造成大量的IO问题

	SELECT film_id,description FROM sakila.film ORDER BY title LIMIT 50,5;  

这个语句进行explain操作，发现会进行全表扫描，并且有文件排序的方式

优化方式：

**步骤1.使用有索引的列或主键进行Order By操作** 
 
	SELECT film_id,description FROM sakila.film ORDER BY film_id LIMIT 50,5  

使用这种方式可以得到相同的结果，但是explain执行计划却完全不同，type=index,rows=55  

这种方式并不是最优的，比如将limit 50,5改为limit 500,5;rows=505，rows会随着limit而改变，如果有上万条数据，那么响应速度回变慢

**步骤2.记录上次返回的主键，在下次查询时使用主键过滤**  

	SELECT film_id,description FROM sakila.film WHERE film_id>55 and film_id<=60 ORDER BY film_id LIMIT 1,5;  

这种方式rows=5，并且不会随着limit的增长而增长  

## 14.索引优化

- 在where从句，group by从句，order by 从句，on从句中出现的列  
- 索引字段越小越好  
- 离散度越大的列放到联合索引的前面    
	
		SELECT * FROM payment WHERE staff_id=2 AND customer_id =584;
  
是index(staff_id,customer_id)好?还是index(customer_id,staff_id)好?  

首先我们先看一下payment表的数据结构是什么样的

	desc payment;
  
然后我们使用语句:select count(distinct customer_id),count(distinct staff_id) from payment;这样我们可以看出他们的离散程度。他们的唯一值越多，说明他们的离散度越好，他们的可选择性就越高。  

这个查询可以看出 customer_id=599，而staff_id=2;说明customer_id离散度更高一些，可选择性更高，因此要建立联合索引时就把customer_id放在最前面。

所以应该使用index(customer_id,staff_id)

## 15.如何选择合适的列建立索引?

**索引不是越多越好**

通常情况下建立索引可以优化我们的查询效率，但是会降低我们的写入效率，也就是说建立索引会增强查询，会影响insert，update，delete这种写入操作。但是往往不是这样的，过多的索引不但会影响我们的写入效率，同时也会影响查询，这是由于数据库进行查询分析的时候，首先要选择哪个索引进行查询，如果索引越多，分析的过程就越慢，这样就会影响查询的效率。因此我们要维护和删除索引。

**索引的维护及优化---重复及冗余索引**  
重复索引是指相同的列以相同的顺序建立同类型的索引，如下表primary key 和ID列上的索引就是重复索引(主键已经是一个唯一索引了)：

	create table test(id int not null primary key,name varchar(10) not null,title varchar(50) not null ,unique(id))engine=innodb;  

冗余索引是指多个索引的前缀列相同，或者是在联合索引中包含了主键的索引，下面这个例子中key(name,id)就是一个冗余索引。

	create table test(id int null primary key,name varchar(50) not null,title varchar(50) not null,key(name,id))engine=innodb;  

**查找重复及冗余索引**  

	SELECT a.TABLE_SCHEMA AS '数据库名',a.table_name as '表名',a.index_name AS '索引1',b.INDEX_NAME AS '索引  2',a.COLUMN_NAME AS '重复列名' FROM STATISTICS a JOIN STATISTICS b ON a.TABLE_SCHEMA=b.TABLE_SCHEMA AND   a.TABLE_NAME=b.TABLE_NAME AND a.SEQ_IN_INDEX=b.SEQ_IN_INDEX AND a.COLUMN_NAME=b.COLUMN_NAME WHERE a.SEQ_IN_INDEX = 1 AND a.INDEX_NAME <> b.INDEX_NAME;

**工具：使用pt-duplicate-key-checker工具**  

**删除不用的索引，目前mysql中只能使用慢查询日志配合pt-index-usage工具来进行索引使用情况的分析**

## 16.数据库结构优化

**选择合适的数据类型**

数据类型的选择，重点在于"合适"二字，如何确定选择的数据类型是否合适？

>1. 使用可以存下你的数据的最小的数据类型  
2. 使用简单的数据类型。Int要比varchar类型在mysql处理上简单  
3. 尽可能的时候not null定义字段  
4. 尽量少用text类型，非用不可是最好考虑分表  
使用bigint来存储ip地址，利用INET_ATON(),INET_NTOA()两个函数来进行转换

	CREATE TABLE sessions(id INT AUTO_INCREMENT NOT NULL,ipaddress BIGINT,PRIMARY KEY(id));

	INSERT INTO session(ipaddress) VALUES(INET_ATON('192.168.0.1'));

	SELECT INET_NTOAA(ipaddress) FROM sessions;

**表的范式化和反范式化**

范式化是指数据库设计的规范，目前说到范式化一般是指第三范式，也就是要求数据库中不存在非关键字段对任意候选关键字段的传递函数依赖则符合第三范式

反范式化是指为了查询效率的考虑把原本符合第三范式的表适当的增加冗余，已达到优化查询效率的目的，反范式化是一种以空间来换取时间的操作。

**表的垂直拆分**

所谓的垂直拆分，就是把原来一个有很多列的表拆分成多个表，这解决了表的宽度问题。通常垂直拆分可以按以下原则进行：

>1. 把不常用的字段单独存放到一个表中。  
2. 把大字段独立存放到一个表中。  
3. 把经常一起使用的字段放到一起。  

**表的水平拆分**

表的水平拆分是为了解决单表的数据量过大的问题，水平拆分的表每一个表的结构都是完整一致的。

常用的水平拆分方法为：

>1. 对id进行hash运算，如果要拆分成5个表则使用mod(id,5)取出0-4个值
2. 针对不同的hashID把数据存到不同的表中。

挑战：1.快分区表进行数据查询2.统计及后台报表操作

## 17.操作系统配置优化

数据库是基于操作系统的，目前大多数Mysql都是安装在Linux系统之上，所以对于操作系统的一些参数配置也会影响到Mysql的性能，下面就列出一些常用的系统配置

网络方面的配置，要修改/etc/sysctl.conf

	#增加tcp支持的队列数
	
	net.ipv4.tcp_max_syn_backlog = 65535
	
	#减少断开连接时，资源回收
	
	net.ipv4.tcp_max_tw_buckets = 8000
	
	net.ipv4.tcp_tw_reuse = 1
	
	net.ipv4.tcp_tw_recycle = 1
	
	net.ipv4.tcp_fin_timeout = 10
	
	
	#打开文件数的限制，可以使用ulimit -a查看目录的各位限制，可以修改/etc/security/limits.conf文件，增加以下内容以修改打开文件数量的限制
	soft nofile 65535
	hard nofile 65535


除此之外最好在Mysql服务器上关闭iptable,selinux等防火墙软件

## 18.mysql配置文件

Mysql可以通过启动时指定配置文件参数和使用配置文件两种方法进行配置，在大多数情况下配置文件位于/etc/my.cnf或是/etc/mysql/my.cnf，在windows系统配置文件可以是位于C:/windows/my.ini文件，Mysql查找配置文件的顺序可以通过以下方法获得

	$/usr/sbin/mysqld --verbose --help|grep -A l 'Default option'

注意：如果存在多个位置存在配置文件，则后面的会覆盖前面的

Mysql配置文件--常用参数说明

	innodb_buffer_pool-size：非常重要的一个参数，用于配置Innodb的缓冲池，如果数据库中个只有Innodb表，则推荐配置量为总内存的75%.
	
	innodb_buffer_pool_instances:Mysql5.5中新增加参数，可以控制缓冲池的个数，默认情况下只有一个缓冲池
	
	Innodb_log_buffer_size:innodb log 缓冲的大小，由于日志最长每秒钟就会刷新所以一般不用太大
	
	innodb_flush_log_at_trx_commit:关键参数，对innodb的IO效率影响很大。默认值为1,可以取0,1,2三个值，一般建议设为2，但如果数据安全性要求比较高则使用默认值1.
	
	innodb_read_io_threads,innodb_write_io_threads：决定了Innodb读写的IO进程数，默认为4
	
	innodb_file_per_table:关键参数，控制Innodb每一个表使用独立的表空间，默认为off，也就是所有表都会建立在共享表空间中
	
	innodb_stats_on_metadata:决定了Mysql在什么情况下会刷新innodb表的统计信息

## 19.第三方工具：Percon Configuration Wizard

## 20.服务器硬件优化：

**如何选择CPU**：是选择单核更快的CPU还是选择核数更多的cup?

>1. MySQL有一些工作只能用到单核CPU
>
2. Mysql对CPU核数的支持并不是越多越快

mysql5.5使用的服务器不要超过32核

**磁盘IO优化：**

常用RAID级别介绍：

RAID0:也称为条带，就是把多个磁盘链接成一个硬盘使用，这个级别的IO最好，缺点就是一个磁盘坏掉那么所有数据就会丢失

RAID1：也称为镜像，要求至少有两个磁盘，每组磁盘存储的数据相同，IO效果不如RAID0，安全性好

RAID5：也是把多个(最少3个)硬盘合并成一个逻辑盘使用，数据读写时会建立奇偶校验信息，并且奇偶校验信息和相对应的数据分别存储于不同的磁盘上。当RAID5的一个磁盘数据发生损坏后，利用剩下的数据和相应的奇偶校验信息去恢复被损坏的数据。

一般的OLTP型的数据库使用RAID1+0：就是RAID1和RAID0的结合。同时具有两个级别的优缺点。一般建议数据库使用这个级别。

**SNA和NAT是否合适数据库：**

>1. 常用于高可用解决方案
2. 顺序读写效率高，但是随即读写不如人意
3. 数据库随即读写比率很高。

转载自：[Get社区](http://get.jobdeer.com/7055.get)