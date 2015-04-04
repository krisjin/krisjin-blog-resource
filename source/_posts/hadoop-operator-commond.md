title: "Hadoop基础操作命令"
date: 2015-04-04 16:00:26
categories: Hadoop
tags:
---


 Hadoop HDFS提供了一组命令集来操作文件，它既可以操作Hadoop分布式文件系统，也可以操作本地文件系统。但是要加上theme(Hadoop文件系统用hdfs://,本地文件系统用file://)

#### 1. 启动Hadoop
	./bin/start-all.sh

#### 2. 停止Hadoop
	./bin/stop-all.sh

#### 3. 查看目录
	./bin/hadoop fs -ls /user/kris/output
#### 4. 创建目录
	./bin/hadoop fs -mkdir /user/kris/test
#### 5. 删除目录及目录下所有文件
	./bin/hadoop fs -rmr /user/kris/test

#### 6. 删除文件
	./bin/hadoop fs -rmr /user/kris/test/testfile

#### 7. 上传文件
上传一个本机文件到hdfs

	./bin/hadoop fs -put /home/kris/redis.txt /user/kris/

#### 8. 下载文件
从hdfs下载一个文件

	./bin/hadoop fs -get /user/kris/redis.txt /home/kris/script

#### 9. 查看文件
	./bin/hadoop fs -cat /user/kris/output-redis/part-r-00000