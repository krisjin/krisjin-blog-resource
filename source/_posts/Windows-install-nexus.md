title: "Windows上安装nexus私有仓库"
date: 2015-04-09 16:33:34
categories: nexus
tags:
---



## 1. Nexus简介
---
Nexus是一个Maven仓库管理器，用来搭建私有仓库服务器。建立公司/组织的私有仓库的的好处是便于管理，节省公网带宽，利用内网下载依赖项速度快，还有一个非常有用的功能就是能有效管理内部项目的SNAPSHOT版本，实现各个模块间的共享。 <!--more-->

## 2. 下载Nexus
---
从http://www.sonatype.org/nexus/archived/#step2top 中选择指定的版本下载到本地。、

## 3. 安装Nexus
--- 
解压nexus-2.8.0-bundle.zip，解压后的有两个目录：

1. nexus-2.8.0-05（该目录包含了Nexus运行所需要的文件，如启动脚本、依赖jar包等）
2. sonatype-work（该目录包含Nenus生成的配置文件、日志文件、仓库文件等。）


配置Path环境变量：
将E:\nexus-2.8\nexus-2.8.0-05\bin 添加到Path中


启动Nexus:

因为我用的是win7 64位OS，所以到E:\nexus-2.8\nexus-2.8.0-05\bin\jsw\windows-x86-64 下运行start-nexus.bat（以管理员身份运行）。

启动成功后我们访问下管理界面：
在浏览器输入：http://localhost:8081/nexus/ 显示界面如下：

![](/img/nexus.png)

这时你可以单击界面右上角的Login进行登录，Nexus默认管理用户名和密码为admin/admin123


## 4. Nexus索引
这时你使用Nexus搜索插件得不到任何结果，为了能够搜索Maven中央库，首先需要设置Nexus中的Maven Central仓库下载远程索引。如下图：  
![](/img/nexus-index.png)

单击左边导航栏的Repositories，可以link到这个页面，选择Central，点击Configuration，里面有一个Download Remote Indexes配置，默认状态是false，将其改为true，‘Save’后，单击Administration==> Scheduled Tasks, 就有一条更新Index的任务，这个是Nexus在后天运行了一个任务来下载中央仓库的索引。由于中央仓库的内容比较多，因此其索引文件比较大，Nexus下载该文件也需要比较长的时间。请读者耐心等待把。如果网速不好的话，可以使用其他人搭建好的的Nexus私服。后面会介绍。下图为Nexus后台运行的task图：

![](/img/nexus-index-config.png)