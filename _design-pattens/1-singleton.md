---
layout: page
date: 2016-04-01 19:37:48 +0800 
title: 单例模式(Singleton)
tags: [Singleton, Design Patten]
---


## 定义

确保一个类只有一个实例，并提供一个访问它的全局访问点。

## 适用场景

当我们系统中某个对象只需要一个实例的情况，如操作系统中只能有一个任务管理器，同一时间内只允许一个实例对其进行操作。

## UML 图

![Singleton](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuIhEpimhI2nAp5L8paaiBdOiAIdAJ2ejIVLCpiyBpgnALJ3W0aie16VcfUUabkJdLNFfbkPb5fQcUkO1JSbBJhM2It7fmfaQcbpQxgabC3KQcaYgQhaSKlDIWBO30000)

Singleton 类通过定义一个私有变量 uniqueInstance 来记录单例类的唯一实例；构造函数之所以定义为私有，是为了防止外界使用 new 关键字来实例化该类；提供访问 uniqueInstance的唯一全局访问点正是公有方法 GetInstance。
另外我们这里之所以定义私有变量为静态是因为每个线程都有自己的线程栈，静态变量在不同的线程间是共享的，定义为静态主要是为了在多线程确保共享一个实例。

## 代码实现

``` csharp
   public class Singleton
   {
       private static Singleton uniqueSingleton;

       private Singleton()
       {
           
       }

       public Singleton GetInstance()
       {
           if (uniqueSingleton is null)
           {
               uniqueSingleton=new Singleton();
           }
           return uniqueSingleton;
       }
   }
```

这段代码在单线程中不存在问题，但在多线程中的静态方法对变量操作时会得到多个 Singleton 实例，因为在两个线程同时运行 GetInstance方法时，此时两个线程判断(uniqueSingleton is null)均为真，这时候两个线程都会创建 Singleton的实例，违背了单例模式的初衷，解决的方法之一就是加锁，方法如下：

```csharp
    public class Singleton
    {
        private static Singleton uniqueSingleton;
        private static readonly object Locker = new object();
        private Singleton()
        {

        }

        public Singleton GetInstance()
        {
            lock (Locker)
            {
                if (uniqueSingleton is null)
                {
                    uniqueSingleton = new Singleton();
                }
            }

            return uniqueSingleton;
        }
    }
```

上面的代码确实可以解决多线程安全的问题，但如果我们能再思考一下，只要第一个线程创建了 Singleton实例，那么后面的就只需要判断 uniqueSingleton 是否为空即可，上面的方法增加了系统开销，损失了性能，这里我们再加入一层判断，如果 uniqueSington为空再进入加锁的流程，改进代码如下：

```csharp
    public class Singleton
    {
        private static Singleton uniqueSingleton;
        private static readonly object Locker = new object();
        private Singleton()
        {

        }

        public Singleton GetInstance()
        {
            if (uniqueSingleton is null)
            {
                lock (Locker)
                {
                    if (uniqueSingleton is null)
                    {
                        uniqueSingleton = new Singleton();
                    }
                }
            }

            return uniqueSingleton;
        }
    }
```

## 总结

单例模式的思考方向首先是确保任何一个时刻只有一个类实例，然后还要为外界提供一个唯一访问点，这个访问点可以是公共方法，也可以是公共的属性。
