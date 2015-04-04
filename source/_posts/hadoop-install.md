title: "Ubuntu14安装hadoop"
date: 2015-04-04 14:50:39
categories: Hadoop
tags:
---

Hadoop的安装比较繁琐，有如下几个原因：其一，Hadoop有非常多的版本；其二，官方文档不尽详细，有时候
更新脱节，Hadoop发展的太快了；其三，网上流传的各种文档，或者是根据某些需求定制，或者加入了不必须要
的步骤，或者加入容易令人误解的步骤。其实安装是很重要的步骤，只有安装好了，才能谈及下一步。
在本书撰写的时候，选用Hadoop的stable版安装。
笔者的登录用户名是brian，大家可以根据自己的登录名更改命令，后面凡是出现brian的地方，都用自己的登录
用户名替换掉。<!--more-->


## 1.操作系统

操作系统是Ubuntu14

如果操作系统其他版本的Ubuntu，在图形界面上会略有一点区别，但对安装影响不大。不同发行版的Linux的安
装Hadoop的过程基本类似，没太大的差别。
## 2.Hadoop的版本
Hadoop当前的stable版是1.2.1。
## 3.下载Hadoop

#### 3.1 在Hadoop的主页上提供了多个下载链接。
http://www.apache.org/dyn/closer.cgi/hadoop/common/

#### 3.2 任选一个下载站点如下：
http://mirror.esocc.com/apache/hadoop/common/
#### 3.3 选择stable版，其实stable版就是1.2.1版：

http://mirror.esocc.com/apache/hadoop/common/stable/  
在这个目录下有多个文件，是针对不同的linux发行版的，不需要全部下载。

#### 3.4 下载hadoop-1.2.1.tar.gz 和 hadoop-1.2.1.tar.gz.mds  
打开命令终端，下文的命令都是在终端里执行，为方便起见，命令都用引号引起。  
将 stable版本的Hadoop的两个文件下载到“~/setup/hadoop”目录下，也就是”/home/brian/setup/hadoop”目录，  
命令如下：  
##### 3.4.1 “mkdir -p ~/setup/hadoop”
mkdir 命令是创建新目录。”-p”参数的意思是，假如hadoop目录的上级目录不存在，也创建上级目录。在
终端里执行“man mkdir”，可以看到对这个命令的更详细的解释，按一下q键重新返回终端。
在命令终端里，”～”表示当前登录用户的主目录。比如说，在开机的时候，登录用户是brian，那么在命
令终端里，”~”就表示目录”/home/brian”，如果开机时候，登录用户是john，那么”~”就表示”/home/john”目录。

##### 3.4.2 "cd ~/setup/hadoop"
cd就是change directory的缩写，切换当前目录。

##### 3.4.3 "wget http://mirror.esocc.com/apache/hadoop/common/stable/hadoop-1.2.1.tar.gz.mds"  
wget是下载文件的命令行工具，”man wget”有详细说明。  

##### 3.4.4 "wget http://mirror.esocc.com/apache/hadoop/common/stable/hadoop-1.2.1.tar.gz"  

##### 3.4.5 "md5sum hadoop-1.2.1.tar.gz"  
md5sum 命令，计算一个文件的md5码。开源社区在提供源码下载的时候，会同时提供下载文件的md5
码。md5码是根据文件内容生成的32位字符串，不同的文件的md5码是不同的，如果下载出错，下载文件的md5码
跟正常文件的md5码是不一样的，由此检测下载是否正常，只有在极其罕见的情况下，才会出现不同的文件有相同
md5码。hadoop-1.2.1.tar.gz是一个比较大的文件，需要检查下载的文件是否完整，执行这个命令之后，会出现形
如"8D 79 04 80 56 17 C1 6C B2 27 D1 CC BF E9 38 5A hadoop-1.2.1.tar.gz"的字符串，前面的一串字符串就是32位的
md5校验码。

##### 3.4.6 "cat hadoop-1.2.1.tar.gz.mds"，  
cat命令，cat是catenate的缩写，在标准输出上打印文件内容，通常标准输出就是屏幕。这个命令会在屏
幕上打印hadoop-1.2.1.tar.gz.mds 的内容，也就是一些校验码，在里面找到"md5"这一行，如果跟md5sum出来的一
致，则表明下载文件完整的，否则需要重新下载。

## 4.安装Java JDK

#### 4.1 在这里有jdk 1.7的下载
http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html
如果是CPU是32位，选择下载Linux x86，如果CPU是64位的，选择Linux x64。一般来说，如果计算机是双
核的，肯定支持64位操作系统。或者可以运行”uname -a”命令看一下，在笔者的笔记本上运行这个命令结果如下：
Linux brian-i3 2.6.32-51-generic #113-Ubuntu SMP Wed Aug 21 19:46:35 UTC 2013 x86_64 GNU/Linux
后面的x86_64表明系统是64位的。   
在这个页面，找 ” Java SE Development Kit 7u40”，注意，这里有一个选项，必须选择”Accept License
Agreement”，接受License才能下载。  
下载的jdk 1.7，存放到 “/home/brian/setup/java-jdk-1.7/”目录。  
下载的文件是”java-jdk-7u40-linux-i586.tar.gz”，java jdk的版本常常有更新，次版本号有可能会比40更高一点。
  
#### 4.2 "sudo su -"
切换到root用户，参考”man sudo”。这个命令会切换到root用户，也就是最高权限的用户。因为后面要执行的  
jdk安装操作是在/usr/local目录下进行的，用root用户更方便。

##### 4.3 "cd /usr/local/lib"

#### 4.4 "tar -zxvf /home/brian/setup/java-jdk-1.7/java-jdk-7u40-linux-i586.tar.gz"
tar 是 linux 下的打包和解压命令行工具，具体细节可以参考”man tar”。这个命令将 java-jdk-7u40-linuxi586.
tar.gz压缩包解压到当前目录下。解压缩完毕之后，执行"ls"，能看到当前目录下有一个新目录叫"jdk1.7.0_40"

#### 4.5 配置环境变量：  

##### 4.5.1 “gedit /etc/profile”
gedit是linux下类似Windoes的记事本的编辑器，文件/etc/profile是linux下的配置文件。本命令会打开这个
配置文件，以备编辑。

##### 4.5.2 添加配置
在/etc/profile文件末尾加上如下的三行代码：  
export JAVA_HOME=/usr/local/lib/jdk1.7.0_40  
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar  
export PATH=$PATH:$JAVA_HOME/bin  
保存文件，然后退出。  
Linux系统，开机后会自动执行/etc/profile配置文件。export命令设置或者显示环境变量。上述三行代码，分
别设置了JAVA_HOME, CLASSPATH, PATH这三个环境变量。

##### 4.5.3 "chown root:root -R /usr/local/lib/jdk1.7.0_40"  
chown命令，更改目录或者文件的拥有者。这条命令将 jdk1.7.0_40目录的拥有者改为root组的root用户。”-
R”参数是递归的意思，将 jdk1.7.0_40目录下连同子目录都进行更改。

##### 4.5.4 "chmod 755 -R /usr/local/lib/jdk1.7.0_40"  
chmod命令，更改目录和文件的模式。本命令将jdk1.7.0_40的模式改为拥有者可以读写执行，同组用户和
其他用户可读可执行不可写。“-R”参数同上，也是递归的意思。  

#####4.5.5 "source /etc/profile"  
如果更改了/etc/profile配置文件，它只会在新的终端里生效，现在正在使用的终端是不会生效的。如果想让
它在正使用的终端也生效，需要用source命令运行一下配置文件。这条命令会让4.4.2的三个环境变量立即生效。这
条命令也可以简写成”. /etc/profile”。

##### 4.5.6 "java -version"
这条命令检查jdk安装是否成功。运行这条命令，只要没有报错就表明安装成功了。

## 5. 安装hadoop 

#### 5.1 "su brian"
su命令，切换用户。安装jdk用的是root用户。现在切回brian用户。

#### 5.2 "mkdir -p ~/usr/hadoop"
创建Hadoop的安装目录

#### 5.3 "cd ~/usr/hadoop"

#### 5.4 "tar -xvzf ~/setup/hadoop/hadoop-1.2.1.tar.gz"
解压缩完毕后，就有目录~/usr/hadoop/hadoop-1.2.1，这是hadoop的主目录。

#### 5.5 配置hadoop，参考了http://hadoop.apache.org/docs/stable/single_node_setup.pdf。
按照伪分布式进行配置，也就是用一个机器同时运行NameNode, SecondaryNameNode, DataNode, JobTracker,
TaskTracker 5个任务。

##### 5.5.1 配置文件在~/usr/hadoop/hadoop-1.2.1/conf/目录下
##### 5.5.2 将core-site.xml 文件内容修改成如下：

	<configuration>
	<property>
	<name>fs.default.name</name>
	<value>hdfs://localhost:9000</value>
	</property>
	</configuration>

##### 5.5.3 将mapred-site.xml文件内容修改如下：
 
	<configuration>
	<property>
	<name>mapred.job.tracker</name>
	<value>localhost:9001</value>
	</property>
	</configuration>

#### 5.5.4 将hdfs-site.xml文件内容修改如下：

	<configuration>
	<property>
	<name>dfs.replication</name>
	<value>1</value>
	</property>
	</configuration>

##### 5.5.5 在hadoop-env.sh文件里添加如下一条语句：
export JAVA_HOME=/usr/local/lib/jdk1.7.0_40

## 6. 安装rsync和ssh

#### 6.1 "sudo apt-get install ssh rsync"
这条命令安装ssh和rsync。ssh是一个很著名的安全外壳协议Secure Shell Protocol。rsync是文件同步命令行工具。

#### 6.2 配置ssh免登录

##### 6.2.1 "ssh-keygen -t dsa -f ~/.ssh/id_dsa"
执行这条命令生成ssh的公钥/私钥，执行过程中，会一些提示让输入字符，直接一路回车就可以。

##### 6.2.2 "cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys"
ssh进行远程登录的时候需要输入密码，如果用公钥/私钥方式，就不需要输入密码了。上述方式就是设置公
钥/私钥登录。

##### 6.2.3 “ssh localhost”
第一次执行本命令，会出现一个提示，输入”yes”然后回车即可。

## 7. 启动hadoop

#### 7.1 "cd ~/usr/hadoop/hadoop-1.2.1"

#### 7.2 "./bin/hadoop namenode -format"
格式化NameNode。

#### 7.3 "./bin/start-all.sh"
启动所有节点，包括NameNode, SecondaryNameNode, JobTracker, TaskTracker, DataNode。

#### 7.4 “jps”
检查各进程是否运行，这时，应该看到有6个java虚拟机的进程，分别是Jps, NameNode, SecondaryNameNode,
DataNode, JobTracker, TaskTracker，看到6 个是对的，表明启动成功。如果提示”jps”没安装或者找不到，执行一
次”source /etc/profile”即可。

## 8. 测试hadoop

#### 8.1 "cd ~/usr/hadoop/hadoop-1.2.1"

#### 8.2 "./bin/hadoop fs -put README.txt readme.txt"
将当前目录下的README.txt放到hadoop进行测试，这个README.txt是Hadoop的介绍文件，这里用它做测
试。这条命令将README.txt文件复制到Hadoop的分布式文件系统HDFS，重命名为readme.txt。

#### 8.3 "./bin/hadoop jar hadoop-examples-1.2.1.jar wordcount readme.txt output"
运行hadoop 的examples 的wordcount，测试hadoop 的执行。这条语句用Hadoop 自带的examples 里的
wordcount程序，对readme.txt进行处理，处理后的结果放到HDFS的output目录。

#### 8.4 "./bin/hadoop fs -cat output/part-r-00000"
这条命令查看处理结果，part-r-00000文件存放wordcount的运行结果，cat命令将文件内容输出到屏幕，显示
字符的统计结果。这是一个简单的字符统计，wordcount只是做了简单的处理，所以会看到单词后面有标点符号。
## 9. 练习
笔者做一次完整的安装是50分钟左右，其中下载Hadoop安装包和Java JDK安装包是半小时，操作部分用时20
分钟。新手第一次安装，2~5个小时内完成都是正常的。建议将Hadoop的安装过程按照上述流程走上三遍，熟悉每个
步骤，然后不看流程凭记忆做出来，重复练习多次次，以加深印象。如果再有时间的话，可以逐个研究里面涉及到
的各种命令，诸如wget, ssh, rsync等等。