---
layout: post
title:  深入.Net 垃圾回收和内存管理
date:   2017-02-03 20:35:48 +0800
categories: technology
tags: [.net]
---
*内容概览*

* Do not remove this line (it will not be displayed)
{:toc}

# 一、.NET 对象内存管理  

我们知道托管堆中存放引用类型对象，因此GC的内存管理的目标主要都是引用类型对象。一个对象的生命周期无非就是创建->使用->释放，在 .net 中一个对象的生命周期如下：

- new 创建对象并分配内存

- 对象初始化

- 对象操作、使用

- 非托管资源的清理

- GC 垃圾回收

托管堆中的对象都是按顺序存放的，而且维护着一个指针 NextObjPtr，它指向下一个对象在堆中的分配位置。

![New object main process](http://www.plantuml.com/plantuml/png/DP0nLiGm34HxdsBAv0f9mRv7GGTSmN6qYNPjASzMWENq21BQx_yqCxuqYzNsaeEnPS8hVKAd6oL3YIpOgYPgBVnfOJYV6dyJg1cNQAmoXg5GqVf0p5dJmU0mIbnenafnGreftSV0LtIdJABxiZxri9K4tODQ7oWtGYVQ1s1uYNNo0tZMd2aTIIGpNqCOpYJ2SSmh94Si2v9AixedqmfJiB2nYsuK_VXT-6Llq-tTgfDAjbV1V_jvhZvszBMwXOj7-SVy0000)

例如有下面一个类，其创建过程为：

``` c#
public class User
{
    public int Age { get; set; }
    public string Name { get; set; }

    public string _Name = "123" + "abc";
    public List<string> _Names;
}
```

1.**对象大小估算，共计40个字节**：

- 属性Age值类型Int，4字节；
- 属性Name，引用类型，初始为NULL，4个字节，指向空地址；
- 字段_Name初始赋值了，由前面的文章（.NET面试题解析(03)-string与字符串操作）可知，代码会被编译器优化为_Name=”123abc”。一个字符两个字节，字符串占用2×6+8（附加成员：4字节TypeHandle地址，4字节同步索引块）=20字节，总共内存大小=字符串对象20字节+_Name指向字符串的内存地址4字节=24字节；
- 引用类型字段 **List<string> _Names** 初始默认为NULL，4个字节；
- User对象的初始附加成员（4字节TypeHandle地址，4字节同步索引块）8个字节；

2.**内存申请**： 申请44个字节的内存块，从指针NextObjPtr开始验证，空间是否足够，若不够则触发垃圾回收。

3.**内存分配**： 从指针NextObjPtr处开始划分44个字节内存块。

4.**对象初始化**： 首先初始化对象附加成员，再调用User对象的构造函数，对成员初始化，值类型默认初始为0，引用类型默认初始化为NULL；

5.**托管堆指针后移**： 指针NextObjPtr后移44个字节。

6.**返回内存地址**： 返回对象的内存地址给引用变量。

>GC是垃圾回收（Garbage Collect）的缩写，是.NET核心机制的重要部分。她的基本工作原理就是遍历托管堆中的对象，标记哪些被使用对象（那些没人使用的就是所谓的垃圾），然后把可达对象转移到一个连续的地址空间（也叫压缩），其余的所有没用的对象内存被回收掉。

首先，需要再次强调一下托管堆内存的结构如下图，只有GC堆才是GC的管辖区域，GC堆里面为了提高内存管理效率等因素，又分成多个部分，其中两个主要部分：

![.NET程序内托管堆](http://www.plantuml.com/plantuml/svg/hT51IyCm5C3nS_Cf7la0qkqYLWe3aLr4F7YJ7LwQjpJuceGqOPQnxsvhYoXp7iGSGzxldv1Ddp2ctB7GraGCBMpXA9HfFFjOov1ZOAg2TQyI61LnBQyUxv_XANeJiOCE7HhIq1862NllqZ9vQ6qYAG3wh6QuuTmdYYiuxYtpSannfgePq7tosxjB6FHfOAhbD4vQmiXgIowXCzFs-hgKeYYSrmGlVOk1QkNVu4iiagu6OlQ7CqiiTKEE8YRh7PHoDt8uURT_Ti_PbVn-xvjoogWvyr-HzO-HPddBwRUEYKJkKkB1ewOuVz0Ez8TNY76Tn8QSpXs_0m00)

- **0/1/2代**：代龄（Generation）在后面有专门说到；

- **大对象堆(Large Object Heap)**：大于85000字节的大对象会分配到这个区域，这个区域的主要特点就是：不会轻易被回收；就是回收了也不会被压缩（因为对象太大，移动复制的成本太高了）

# 二、垃圾回收

垃圾简单理解就是没有被引用的对象，其垃圾回收的基本流程包含以下三个步骤：

1.**标记**

先假设所有对象都是垃圾，根据应用程序根指针Root遍历堆上的每一个引用对象，生成可达对象图，对于还在使用的对象（可达对象）进行标记（其实就是在对象同步索引块中开启一个标示位）

>其中Root根指针保存了当前所有需要使用的对象引用，他其实只是一个统称，意思就是这些对象当前还在使用，主要包含：静态对象/静态字段的引用；线程栈引用（局部变量、方法参数、栈帧）；任何引用对象的CPU寄存器；根引用对象中引用的对象；GC Handle table；Freachable队列等。

2.**清除**

针对所有不可达对象进行清除操作，针对普通对象直接回收内存，而对于实现了终结器的对象（实现了析构函数的对象）需要单独回收处理。清除之后，内存就会变得不连续了，就是步骤3的工作了。

3.**压缩**

把剩下的对象转移到一个连续的内存，因为这些对象地址变了，还需要把那些Root跟指针的地址修改为移动后的新地址

![垃圾回收的基本流程](https://www.plantuml.com/plantuml/svg/ZPHTRuCW58QlvLSGhxtezqrZqkwhFw9f5i0pTKCnWElCqly-A5dgfc2zupo7GtXUuavf8dLRSPILkIHDWG_u6w58jTGj6M-L1lawCsNyvqDHf7J78NqhEOTiQzPCS25JNkQ5pYLqjbIB3F2ntvYuhdDHQaXKGHf8gVWwRH6P7Hqo9tS99nHu6Wlw3aplunCYSoz-i7ZXnOyMBxtuoUAL5pzRlFRY5uitNdmmE8gk9PpYk2Bo0xUr1C8AGZd4nj5bpFVy6iEOlhuoUbC6K0-TqLzd7N0kpaDhT15eiDxeqd-25KTNWG5M7Ltx1rXtT1COyCzU3ZL8eW4xVnomQyCLQUBR0wIy1QirGWEjYxkqiWcjBAmrSOFB-hDKfGdPR0Xh7jamePs5jOyCd8Y1XMCO3kp5eiu03TO5OBTPZMGnI69vLnBPH1BPXATiujgzMp1HDOJfqNSKeGlQGPsP7-KF)

>垃圾回收的过程是不是还挺辛苦的，因此建议不要随意手动调用垃圾回收GC.Collect()，GC会选择合适的时机、合适的方式进行内存回收的。

## 关于代龄（Generation）

当然，实际的垃圾回收过程可能比上面的要复杂，如果每次都扫描托管堆内的所有对象实例，这样做太耗费时间而且没有必要。分代(Generation)算法是CLR垃圾回收器采用的一种机制，它唯一的目的就是提升应用程序的性能。分代回收，速度显然快于回收整个堆。分代(Generation)算法的假设前提条件：

1、大量新创建的对象生命周期都比较短，而较老的对象生命周期会更长

2、对部分内存进行回收比基于全部内存的回收操作要快

3、新创建的对象之间关联程度通常较强。heap分配的对象是连续的，关联度较强有利于提高CPU cache的命中率.

.NET将托管堆分成3个代龄区域: Gen 0、Gen 1、Gen 2：

- 第0代，最新分配在堆上的对象，从来没有被垃圾收集过。任何一个新对象，当它第一次被分配在托管堆上时，就是第0代（大于85000的大对象除外）。 
- 第1代，0代满了会触发0代的垃圾回收，0代垃圾回收后，剩下的对象会搬到1代。 
- 第2代，当0代、1代满了，会触发0代、1代的垃圾回收，第0代升为第1代，第1代升为第2代。

![GC generation](/assets/dotnenet-mem-06-generation.png)
大部分情况，GC只需要回收0代即可，这样可以显著提高GC的效率，而且GC使用启发式内存优化算法，自动优化内存负载，自动调整各代的内存大小

## 非托管资源回收

.NET中提供释放非托管资源的方式主要是：Finalize() 和 Dispose()。

### Dispose()

常用的大多是Dispose模式，主要实现方式就是实现IDisposable接口，下面是一个简单的IDisposable接口实现方式

```c#
public class SomeType : IDisposable
{
    public MemoryStream _MemoryStream;
    public void Dispose()
    {
        if (_MemoryStream != null) _MemoryStream.Dispose();
    }
}
```

Dispose需要手动调用，在.NET中有两种调用方式：

```c#
#方式1：显示接口调用
SomeType st1=new SomeType();
//do sth
st1.Dispose();

#方式2：using()语法调用，自动执行Dispose接口
using (var st2 = new SomeType())
{
    //do sth
}
```

第一种方式，显示调用，缺点显而易见，如果程序猿忘了调用接口，则会造成资源得不到释放。或者调用前出现异常，当然这一点可以使用try…finally避免。

一般都建议使用第二种实现方式，他可以保证无论如何Dispose接口都可以得到调用，原理其实很简单，using()的IL代码如下图，因为using只是一种语法形式，本质上还是try…finally的结构。
![using IL](https://images2015.cnblogs.com/blog/151257/201603/151257-20160309235625475-414934067.png)

### Finalize() ：终结器（析构函数）

>首先了解下Finalize方法的来源，她是来自System.Object中受保护的虚方法Finalize，无法被子类显示重写，也无法显示调用，是不是有点怪？。她的作用就是用来释放非托管资源，由GC来执行回收，因此可以保证非托管资源可以被释放。

- 无法被子类显示重写：.NET提供类似C++析构函数的形式来实现重写，因此也有称之为析构函数，但其实她只是外表和C++里的析构函数像而已。

- 无法显示调用：由GC来管理和执行释放，不需要手动执行了，再也不用担心猿们忘了调用Dispose了。

所有实现了终结器（析构函数）的对象，会被GC特殊照顾，GC的终止化队列跟踪所有实现了Finalize方法（析构函数）的对象。

- 当CLR在托管堆上分配对象时，GC检查该对象是否实现了自定义的Finalize方法（析构函数）。如果是，对象会被标记为可终结的，同时这个对象的指针被保存在名为终结队列的内部队列中。终结队列是一个由垃圾回收器维护的表，它指向每一个在从堆上删除之前必须被终结的对象。

- 当GC执行并且检测到一个不被使用的对象时，需要进一步检查“终结队列”来查询该对象类型是否含有Finalize方法，如果没有则将该对象视为垃圾，如果存在则将该对象的引用移动到另外一张Freachable列表，此时对象会被复活一次。

- CLR将有一个单独的高优先级线程负责处理Freachable列表，就是依次调用其中每个对象的Finalize方法，然后删除引用，这时对象实例就被视为不再被使用，对象再次变成垃圾。

- 下一个GC执行时，将释放已经被调用Finalize方法的那些对象实例。

**简单总结一下：** Finalize()可以确保非托管资源会被释放，但需要很多额外的工作（比如终结对象特殊管理），而且GC需要执行两次才会真正释放资源。听上去好像缺点很多，她唯一的优点就是不需要显示调用。

有些编程不建议大家使用Finalize，尽量使用Dispose代替，我觉得可能主要原因在于：第一是Finalize本身性能并不好；其次很多人搞不清楚Finalize的原理，可能会滥用，导致内存泄露。因此就干脆别用了，其实微软是推荐大家使用的，不过是和Dispose一起使用，同时实现IDisposable接口和Finalize（析构函数），其实FCL中很多类库都是这样实现的，这样可以兼具两者的优点。

- 如果调用了Dispose，则可以忽略对象的终结器，对象一次就回收了；
- 如果程序猿忘了调用Dispose，则还有一层保障，GC会负责对象资源的释放；

# 三、性能优化建议

## 尽量不要手动执行垃圾回收的方法：GC.Collect()

垃圾回收的运行成本较高（涉及到了对象块的移动、遍历找到不再被使用的对象、很多状态变量的设置以及Finalize方法的调用等等），对性能影响也较大，因此我们在编写程序时，应该避免不必要的内存分配，也尽量减少或避免使用GC.Collect()来执行垃圾回收，一般GC会在最适合的时间进行垃圾回收。

而且还需要注意的一点，在执行垃圾回收的时候，所有线程都是要被挂起的（如果回收的时候，代码还在执行，那对象状态就不稳定了，也没办法回收了）。

## 推荐Dispose代替Finalize

如果你了解GC内存管理以及Finalize的原理，可以同时使用Dispose和Finalize双保险，否则尽量使用Dispose。