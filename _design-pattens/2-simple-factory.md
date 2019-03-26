---
layout: page
date: 2016-04-01 19:37:48 +0800 
title: 简单工厂模式(Simple Factory)
tags: [Simple Factory, Design Patten]
---

## 定义

简单工厂模式可以理解为负责生产对象的一个类，平时编程中如果类中有调用`new`关键字创建一个对象时，我们就说这个类依赖这个对象，也就是说它们的耦合度很高，当需求改变时，我们不得不修改此类的源码，那么简单工厂的思想就是去封装改变，即把容易改变的地方抽出来构造一个简单工厂类。

## 适用场景

客户端要一辆车，如果是自己造的话，就比较麻烦，需要调用服务端构造好的轮胎、发动机、车盖子，还要自己充气，喷漆等流程操作，而且一旦需求有变化，如流程顺序改变或车的尺寸改变，就需要频繁更改客户端代码。简单工厂的好处就是把这些造车逻辑封装到服务端，客户消费端只需要提要求即可，这样就封装了改变，如果 web服务一样，客户端并不应该过多改变。

## UML 图

![SimpleFactory](http://www.plantuml.com/plantuml/png/LKyzRy8m4DtlLzmOP8mwTwY8ebPYwDI1sRLFOE6SaNsXQgFyzzea25Zkz7plutcRnIGP-W3vvFbiakdH1fDpAvByzo2qiv6__1_X2rmFs9dqOQp4z3k26ejUl9q-pObQBgUzbsCNGqnDBWPyG5l1wlKIlOFfRZ62VkU3PqAwBj3x8r70dpf1T_522MsTe0gzUIAlyO6ynWVo2bIjKXLF-Q1QvsQFPYcZTbpAXt334y3zQxZPlDMQzGcZdlDkOFPPq9mEMs8tzE4V)

## 代码实现

没有使用简单工厂的代码可能是这样的：

```csharp
    public class Car
    {
        private Engine engine;
        private Wheel wheel;
        private Cover cover;

        public Car(Engine engine, Wheel wheel, Cover cover)
        {
            this.engine = engine;
            this.wheel = wheel;
            this.cover = cover;
        }

        public void MakeCar(string color)
        {
            cover.Print(color);
            this.AddEngine();
            this.InstallWheel();
        }

        public void AddEngine() { }
        public void InstallWheel() { }
    }

    public class Engine { }

    public class Wheel { }

    public class Cover
    {
        public void Print(string color)
        {
            Console.WriteLine($"The car is {color}");
        }
    }
```

客户端代码需要写成这样：

```csharp
class Program
    {
        static void Main(string[] args)
        {
            var engine = new Engine();
            var wheel = new Wheel();
            var cover = new Cover();
            
            var redCar=new Car(engine,wheel,cover);
            redCar.MakeCar("red");
            var blakCar=new Car(engine,wheel,cover);
            blakCar.MakeCar("Black");
            Console.Read();
        }
    }
```

而简单工厂方法就是把对象依赖的部分和容易改变的部分从客户端拿掉，因为客户不关心车市如何造出来的，消费者只要提需求，就应该拿到符合要求的车。
那么我们可以加入一个工厂类，里面有一个静态方法：

```csharp
    public class CarFactory
    {
        public static void MakeAnCarWithColor(string color)
        {
            var engine = new Engine();
            var wheel = new Wheel();
            var cover = new Cover();

            var car = new Car(engine, wheel, cover);
            car.MakeCar(color);
        }
    }
```

这样客户端的方法调用就改造为：

```harp
    class Program
    {
        static void Main(string[] args)
        {
            CarFactory.MakeAnCarWithColor("Red");
            CarFactory.MakeAnCarWithColor("Black");

            Console.Read();
        }
    }
```

## 总结

简单工厂有利有弊，好处是客户端不用关心实际生产对象逻辑，全权交给工厂类处理；坏处是工厂类一旦挂掉，系统崩溃，这就需要我们做个权衡。
