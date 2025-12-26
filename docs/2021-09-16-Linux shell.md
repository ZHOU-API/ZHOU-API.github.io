---
title: 'Linux shell'
layout: post
tags:
  - linux
category: 运维
---
Linux shell脚本常用记录.

<!--more-->

# 定时脚本

```shell
nohup [command] > /dev/null 2>&1 &
```

\> 直接把内容生成到指定文件，会覆盖原来文件中的内容[ls > test.txt],
\>> 尾部追加，不会覆盖原有内容 [ls >> test.txt],
< 将指定文件的内容作为前面命令的参数[cat < [text.sh](http://text.sh/)]

# 删除脚本

```shell
#!/bin/bash
find /home/visual/app/csv -mtime +30 -name "*.csv" | xargs -i rm -rf {}
find /home/visual/app/csv -type d -empty -delete
```

「 find 」查找

「 /home/visual/app/csv」指定的目录

「 -mtime +30 」30天前的（天数可自定义）

「 -name "*.csv 」所有.csv结尾的文件

「 | xargs -i  rm -rf {} 」固定写法

「 -type 」根据指定类型查找(-,d,l,s,f,b)

# 定时任务

## 进入编辑页

```shell
crontab -e
```



## 编辑cron表达式

```shell
0 2 * * * cmd
```



## 查看定时任务

```shell
crontab -l
```

