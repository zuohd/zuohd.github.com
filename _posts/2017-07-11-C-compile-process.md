---
layout: post
title:  "从C源文件到可执行文件过程浅析"
date:   2017-07-11 08:35:48 +0800
categories: technology
tags: [programming]

---

## 创建一个C文件 `firstC.c`

``` c

#include <stdio.h>
#include <stdlib.h>

int main()
{
        printf("Hello world!\n");
        exit(0);
}

```
1.预处理阶段，生成`.i`文件

``` shell

gcc –E firstC.c >firstC.i

```

2.编译阶段，生成同名的汇编文件，文件以后缀`.s`结尾

``` shell

gcc –S firstC.i 

```
执行`cat first.s`此时会看到汇编文件内容:
``` output
	.file	"firstC.c"
	.section	.rodata
.LC0:
	.string	"Hello world!"
	.text
	.globl	main
	.type	main, @function
main:
.LFB2:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	$.LC0, %edi
	call	puts
	movl	$0, %edi
	call	exit
	.cfi_endproc
.LFE2:
	.size	main, .-main
	.ident	"GCC: (GNU) 4.8.5 20150623 (Red Hat 4.8.5-28)"
	.section	.note.GNU-stack,"",@progbits
```
3.汇编阶段，生成同名目标文件，文件后缀以`.o`结尾

``` shell

gcc –C firstC.s 

```
4.链接阶段，生成可执行文件
``` shell

gcc firstC.o -o firstC

```
5.在Linux终端执行文件firstC查看输出结果
``` shell

./firstC

```

当然我们也可以一步到位执行命令`gcc firstC.c`，此时gcc会默认生成可执行文件`a.out`，如果想指定可执行文件名称需追加-o参数：
``` shell

gcc firstC.c -o hello

```
>如果使用`make firstC`将生成与源文件同名的可执行文件


