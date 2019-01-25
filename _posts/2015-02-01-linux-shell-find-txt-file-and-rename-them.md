---
layout: post
title:  "Linux shell 查找目录下所有txt文件并去掉扩展名"
date:   2015-02-01 12:37:48 +0800
categories: Linux shell
tags: [Linux shell]
---
 今天看Linux脚本编程的时候，想到一个需求：查找/home/shell/homework下(包含子目录)所有的文本文件并去掉其扩展名，遂有了下面的想法：

``` shell
#!/bin/bash
func(){
for file in $(ls $1); do
file=$1/$file

if [ -d $file ]; then
func $file; else
if [ ${file##*.} = txt ]; then
mv $file ${file%.*}; fi
fi
done
}

func /home/shell/homework

```

