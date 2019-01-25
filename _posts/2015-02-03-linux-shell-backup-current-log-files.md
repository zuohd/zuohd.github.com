---
layout: post
title:  "Linux shell 定期备份log文件"
date:   2015-02-03 12:37:48 +0800
categories: technology
tags: [Linux shell]

---
>实现将/home/shell/homework/logs目录下所有当前周生成的log文件备份至/home/shell/homework/currentweeklogs,不能改变源文件的创建修改日期，每周日备份。
                
1.新建shell文件cpweeklogs.sh

``` shell
#!/bin/bash

function cp_current_week_logs(){
last_sunday_d=$(date -d 'last sunday' +%Y%m%d)
next_monday_d=$(date -d 'next monday' +%Y%m%d)

for file in `find $1 -name "*.log"`; do
file_name=${file##*/}
file_ymd=`echo ${file_name%.*} | cut -c 1-4,6-7,9-10`
if [ $file_ymd -lt $next_monday_d -a $file_ymd -gt $last_sunday_d ]; then
        target_logs_dir="/home/shell/homework/currentweeklogs"
        if [ ! -e $target_logs_dir/$file_name ]; then
        cp -p $file $target_logs_dir
        fi
fi
done
}

cp_current_week_logs /home/shell/homework/logs

```
2.`crontab -e` 进入cron定时任务编辑模式,此处记得为cpweeklogs.sh 赋予执行权限

``` shell
* * * * 0 /home/shell/cpweeklogs.sh #every week sunday scheduler

```

3.可通过查看/var/log/cron 文件验证监控定时任务是否正常执行
                
``` shell
cd /var/log
cat cron

```
