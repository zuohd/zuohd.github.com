---
layout: page
date: 2016-04-01 19:37:48 +0800 
title: 抽象工厂模式(Abstract Factory)
tags: [Abstract Factory, Design Patten]
---

## 定义

工厂模式解决了简单工厂的缺点，简单工厂随着产品的增加需要修改简单工厂方法。而工厂方法模式每个具体工厂类只完成单个产品的创建，所以它有很好的扩展性。但现实生活中，一个工厂只创建一个产品的可能性很小，大部分工厂会创建一系列产品，而设计这样的产品工厂方法模式不太适用，那么我们的抽象工厂模式可以解决这样的问题。
**抽象工厂模式提供一个创建产品的接口来创建一组相关产品，而不需要或关心实际生产中的具体产品是什么，这样客户就可以从具体产品中解耦。**

## 适用场景

抽象工厂模式将创建具体产品延迟到具体的工厂子类中，这样讲对象的创建封装起来，可以减少客户端与具体产品的依赖，从而使系统耦合度低，从而有利于后期的维护和扩展，这是抽象工厂模式的优点。但抽象工厂也有缺点，很难支持新种类产品的添加，这是因为抽象工厂接口中已经确定了可以被创建的产品集合，如果需要添加新产品，就必须修改抽象工厂的接口，这样就会涉及到抽象工厂类以及所有实现类的修改，违背了`开闭原则`，由此我们可以看出抽象工厂适用的场景：

1.一个系统不依赖具体产品的创建，组合，这也是所有工厂模式的前提；
2.这个系统有多个系列产品，而系统中只消费某一系列产品；
3.系统要求提供一个产品类的库，所有产品以同样的接口出现，客户端不依赖具体实现。

## UML 图

![AbstractFactory](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuKfCAYufIamkKKZEIImkLd1EB783ylABAgkvgc5ojSxvPQb52hw9sQbWbeOcabnuHcfg3bWEp0BArN8XJB6fqTLryWs7yj0b8brW4bpe3W5RXEm689MPbmws26G7kdgmvVZKWms4EWVCHXA9T868W0s1gW6p8I9GDH0HBeHtzFJiO8PPFUsO_NJtBWn9i1wuPAGBGX9cAIv959rSpZcPgNabG1b425gzsD3yVCeAX1JOE0nHi3amKOVKl1HWh0C0)

## 代码实现

```csharp
namespace AbstractFactoryPatten
{
    public abstract class CarFactory
    {
        public abstract Cover MakeCover();
        public abstract Wheel MakeWheel();
    }

    public class BMWFactory:CarFactory
    {
        public override Cover MakeCover()
        {
            return new BMWCover();
        }

        public override Wheel MakeWheel()
        {
            return new BMWWheel();
        }
    }

    public class BYDFactory:CarFactory
    {
        public override Cover MakeCover()
        {
            return new BYDCover();
        }

        public override Wheel MakeWheel()
        {
            return new BYDWheel();
        }
    }

    public abstract class Cover
    {
        public abstract string Make();
    }

    public abstract class Wheel
    {
        public abstract string Make();
    }

    public class BMWCover : Cover
    {
        public override string Make()
        {
            return this.GetType().Name;
        }
    }

    public class BMWWheel:Wheel
    {
        public override string Make()
        {
            return this.GetType().Name;
        }
    }

    public class BYDCover : Cover
    {
        public override string Make()
        {
            return this.GetType().Name;
        }
    }

    public class BYDWheel:Wheel
    {
        public override string Make()
        {
            return this.GetType().Name;
        }
    }
}

```

## 总结

.NET中实现抽象工厂模式的例子就是`System.Data.Common.DbProviderFactory`，为了连接多种类型的数据库，每一个都有实现类，如`SqlClientFactory`就是处理 SQL Server 相关操作方法的类。
