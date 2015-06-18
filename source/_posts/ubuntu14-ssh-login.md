title: "ubuntu14 ssh登录问题？"
date: 2015-04-04 11:49:33
tags: [ubuntu]
---

1. 关闭防火墙  
 sudo ufw disable

2. Ubuntu系统上安装、启动sshd服务  
sudo apt-get install openssh-server
sudo /etc/init.d/ssh restart  

3. ssh还不能登录上，就修改sshd的默认配置  
ssh出现permission denied (publickey)问题:  
修改/etc/ssh/sshd-config文件.    
将其中的PermitRootLogin no修改为yes  
PubkeyAuthentication yes修改为no  
AuthorizedKeysFile .ssh/authorized_keys前面加上#屏蔽掉，  
PasswordAuthentication no修改为yes就可以了。
