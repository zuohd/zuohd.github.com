---
layout: post
title:  位运算注解 
date:   2019-03-11 08:35:48 +0800
categories: technology
tags: [算法]

---
# 什么是位运算

程序中所有数在计算机内存中都是以二进制的形式存储的，位运算说穿了就是直接对整数在内存中的二进制位进行操作。比如，and 运算本来是一个逻辑运算符，但整数与整数之间也可以进行 and 运算。举个例子，6 的二进制是 110，11 的二进制是 1011 ，那么6 and 11 的结果就是2，它是二进制对应位进行逻辑运算的结果 （0 表示 False，1 表示 True，空位都当0处理）：

``` output
110 AND 1011 --> 0010(b) --> 2(d)
```

由于位运算直接对内存数据进行操作，不需要转成十进制，因此处理速度非常快。
下面看看位运算符号的含义：

| 符号  | 描述   | 运算规则 |
| ---- | -----  | --------|
| &    |与     |两个位都为1时，结果才为1|
|\\|   |或     |两个位都为0时，结果才为0|
|^     |异或   |两个位相同为0，不同为1|
|~     |取反   |0变1，1变0|
|<<    |左移   |各二进位全部左移若干位，高位丢弃，低位补零|
|>>    |右移   |各二进位全部右移若干位，对无符号数高位补零，有符号数各编译器处理方法不同，有的补符号位（算数右移），有的补0（逻辑右移）|

## XOR-异或的特点

>异或：相同为0，不同为1。也可用不进位加法来理解。

``` output
x ^ 0 =x

x^ 1s =~x #1s为全部二进制都为1的数

x^ (~x)=1s

x ^ x=0 #interestring and important!

a^ b =c =>a^c=b,b^c=a #实现a和b的交换

a ^b ^c=a ^(b^c)=(a^b)^c #异或结合律？

```

## 实战常用的位运算操作

- X&1==1 OR ==0 用来判断奇数偶数，因为 X&1会取到 X的最后一位，为 1 则为奇数，为0 则为偶数；

- X = X & (X-1) 用于对X的最低位1进行清零：如 X=1100000，则X-1=1011111，两者与的结果是1000000，再赋值给X

- X & -X 得到最低位的1

## 复杂的位运算操作

1.将X的最右边的 n 位清零 `X&(~0<<n)`

2.获取X的第 n 位值 `(X>>n)&1`

3.获取X的第 n 位幂值 `X&(1<<(n-1))`

4.仅将第 n 位 置为1 `X|(1<<n)`

5.仅将第 n 位 置为0 `X&(~(1<<n))`

6.将 X 最高位至第 n 位（包含）清零 `X&((1<<n)-1)`

7.将第 n 位至第 0 位（包含）清零 `X&(~((1<<(n+1))-1))`

## 位计算的应用

1.编写一个函数，输入一个无符号整数，返回其二进制表达式中数字位数为1的个数。

两种思路，一种是让输入数X与2取模，看余数是否为1，为1则计数count加1，然后X等于X>>1,这里的算法复杂度虽然为O（1），但最大循环次数为32次；另一种就是让X等于X&(x-1),即将X的最低位1置0，然后循环条件只要x不为0，就执行清除最低位1的操作，直到循环退出为止，这样得到的循环次数就是X中二进制的1的个数,当然这里最坏的情况也是32次，即X的二进制都为1的情况。

### 方案一

``` python
class Solution(object):
    def hammingWeight(self, n):
        """
        :type n: int
        :rtype: int
        """
        rst=0
        mask=1
        for i in range(32):
            if n&mask:
                rst+=1
            mask=mask<<1
        return rst
```

### 方案二

```python
class Solution(object):
    def hammingWeight(self, n):
        """
        :type n: int
        :rtype: int
        """
        rst=0
        while (n!=0):
            n=n&(n-1)
            rst+=1
        return rst
```

2.给定任意一个数，判断其是不是2的指数。如4=2^2,8=2^3,但9不是：

依然两种思路：一种是不断与2取模，把商这样不断与2取模；另一种就是符合条件的数其实是`n个0...1...n个0`的格式，这样我们就写判断x!=0 and (x&(x-1)==0)即可。这里我们只给出方案二：

```python
class Solution(object):
    def isPowerOfTwo(self, n):
        """
        :type n: int
        :rtype: bool
        """
        return n!=0 and (n&(n-1)==0)
```

3.给定一个非负整数 num，对于 `0<=i<=num`范围中的每个数字 i，计算器二进制数中1的个数，并将它们作为数组返回。如给定数为2，则输出为0、1、2中1的个数至数组中，结果为[0,1,1]。

两种思路：一种循环0到num，根据前面的思路计算每个循环i的1位个数，填充到数组；
一种是初始化一个数组，长度为`num+1`，所有元素默认值为0，循环i变量从0到num，每次循环为 `array[i]=count[i&(i-1)]+1`,以下代码就是这种方案：

``` python
class Solution(object):
    def countBits(self, num):
        """
        :type num: int
        :rtype: List[int]
        """
        list=[0]*(num+1)
        for n in xrange(1,num+1):
            list[n]+=list[n&(n-1)]+1
        return list
```

4.如何将 n 个皇后放置在 n*n 的棋盘上，并且使皇后彼此之间不能相互攻击。
思路：将棋盘看作二进制位，如n为4就是4为，二进制为1代表已经有皇后在，二进制为0代表没有皇后在，采用DFS遍历算法，代码有注释。

``` python
class Solution(object):
    def totalNQueens(self, n):
        """
        :type n: int
        :rtype: int
        """
        if n<1:return []
        self.count=0
        self.DFS(n,0,0,0,0)
        return self.count
    def DFS(self,n,row,cols,pie,na):
        # 递归终止条件
        if row>=n:
            self.count+=1
            return
        bits=(~(cols|pie|na))&((1<<n)-1) #得到当前所有的空位
        while bits:
            p=bits&-bits #取到最低位的1
            self.DFS(n,row+1,p|cols,(p|pie)<<1,(p|na)>>1)
            bits=bits&(bits-1) #去掉最低位的1
```