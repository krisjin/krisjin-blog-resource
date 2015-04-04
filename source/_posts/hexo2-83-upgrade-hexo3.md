title: "hexo2.8.3 到3.0使用问题"
date: 2015-04-04 11:32:15
categories: hexo
tags:
---


为了在单位和家里都可以很方面的用hexo 写博客，博客和源码都放在github。这样就方便很多，每次写的随时条。换了不同的地域或电脑，直接重新pull一下，就可以获取最新的博客源码。  

但是，有的时候从不能很顺利的。node和 hexo 都会更新的，如果在另一台电脑重新搭建环境的话，用的node和hexo都是最新的，这就会和之前的有版本冲突。下面就来解决一下这个问题，我之前用的hexo2.8.3版本，新的环境是hexo3.0  <!--more-->

1. 搭建好环境  （安装node，mygit）
2. 下载博客源码到本地  
3. 先将_config.yml 保存另一个位置(执行hexo init 时会被覆盖掉)
4. 安装hexo(npm install -g hexo)
5. hexo init
4. 安装依赖（npm install）

现在执行hexo g 和hexo s 都可以正常的运行，可以看到显示的博客了。

当你执行 hexo deploy 时会报错：

![](/img/deploy-git.png)


在网上找到了这样的答案：

	In version 3, your should use git instead of github.
	And your need to run
	
	npm install hexo-deployer-git --save

就执行上面的命令

成功执行后还需要修改_config.yml 文件

	deploy:
	  type: github
	  repository: https://github.com/krisjin/krisjin.github.io.git
	  branch: master

将type: github 改为 type: git


这时在去执行hexo d 就没问题了。


为了和之前的版本不会产生冲突，这里修改.gitignore 将package.json添加进去，当然这不是好的解决方法。

也可以把之前的打个tag