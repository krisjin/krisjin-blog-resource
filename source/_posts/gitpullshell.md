title: Git仓库更新shell
date: 2015-06-01 09:00:21
categories: shell
tags: [git,shell]
---

shell脚本

```bash
		#!/bin/sh
		cd . 
		reps="proA proB"
		arr=($reps)
		len=${#arr[@]}
		
		for ((i=0; i<$len; i++)); do
		    if [ $i -eq 0 ] ;then
				cd ${arr[$i]}
		    else
				cd ../${arr[$i]}
		    fi
		    git pull
		    echo ${arr[$i]} pull end ...............
		done
```