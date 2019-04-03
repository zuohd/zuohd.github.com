---
layout: page
date: 2016-04-01 19:37:48 +0800 
title: 建造者模式(Builder)
tags: [Builder, Design Patten]
---

## 定义

我们在软件系统中经常要创建一个复杂的对象，并且这个复杂对象由各个子对象按照一定的步骤顺序组装完成，还以我们前面的组装汽车为例：我们去买车，汽车本身是一个复杂的对象，它是由引擎、轮胎、车盖、车灯、内饰等组装而成的，而这个组装的过程不可能由消费者来完成，而是建一个建造者类对象，建造者只要返回给消费者组装好的汽车就行了。那么从我们下了订单，到汽车厂家依据订单生产组装汽车，就可以用建造者模式来解决。

## 适用场景

建造者模式将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。客户端不需要知道产品构建的具体细节，从而降低了客户端和具体产品之间的耦合度。
在建造者模式中，客户端直接与指挥者打交道，指挥者将客户端组装汽车的请求话费为对各个零件的组装请求，再将这些请求分配给具体的建造者角色，具体建造者是完成具体品牌车型的组装工作的，却不为客户知道；我们说“构造者模式主要用于分步骤构建一个复杂的对象”，分步骤是一个固定的过程，而复杂对象的各个部分是经常变化的，如不同品牌车型安装不同的引擎和车灯；

## UML 图

![builder-design-patten](http://www.plantuml.com/plantuml/svg/TL91IofH6DtVN_7XIf4HtbP4V5gqiHQrQ1F4D7FJ0TzCtBdQmWGZ9Gp9I5T5q4FCLf5j2dNwDJFNQTLVw4vtVEygkHg-SyvtxlaEax6fIcZvJmbf9TMs8QUIAbBoXarJDYM6MKZ3_X5Ns2XUiGmTVkjwL10Gm34KNz3k5AtZQ0pLa7h08LMZ86opPQEaOm9L152TSpK8B0MtRXOC4_FL5NpEgc2ociuRXI9Tflaji84f_tn7CYsAWGJ296iziMcFNJZ-wAyt6N_Kx_rwqnzU-wsNpsdRVHlF9Ws_t_IQhwxJtJFPoF6T1tSyzBgdivk6WF_jiSwL-txBBWTUfn-sk6OGh56BLndFMIRFKmwcQDZ2ps7Qvwmza6FupsUptcEum-u68fF8Wa8uTQ8eaDtS3SS5U3keB0J4Un7PGZv4BY02gWrRlz1_rqI2ZvAR06G-pJjEQWIh52y9PMRUYgAaGrwg0Qn32BGY4mydbuuosDJvJ_e5)

## 代码实现

```csharp
using System.Collections.Generic;

namespace BuilderDesignPattens
{
    public class Car
    {
        private readonly List<string> _parts;

        public Car()
        {
            this._parts=new List<string>();
        }

        public void Add(string part)
        {
            _parts.Add(part);
        }

        public List<string> Show()
        {
            return _parts;
        }
    }
    public abstract class Builder
    {
        public abstract void BuildEngine();
        public abstract void BuildCover();
        public abstract void BuildLight();
        public abstract Car GetCar();
    }

    public class BMWBuilder : Builder
    {
        private Car bmwCar=new Car();
        public override void BuildEngine()
        {
            bmwCar.Add("Engine");
        }

        public override void BuildCover()
        {
            bmwCar.Add("Cover");
        }

        public override void BuildLight()
        {
            bmwCar.Add("Light");
        }

        public override Car GetCar()
        {
            return bmwCar;
        }
    }

    public class RoverBuilder : Builder
    {
        private Car roverCar = new Car();
        public override void BuildEngine()
        {
            roverCar.Add("Engine");
        }

        public override void BuildCover()
        {
            roverCar.Add("Cover");
        }

        public override void BuildLight()
        {
            roverCar.Add("Light");
        }

        public override Car GetCar()
        {
            return roverCar;
        }
    }
    public class Director
    {
        public void Construct(Builder builder)
        {
            builder.BuildEngine();
            builder.BuildCover();
            builder.BuildLight();
        }
    }
}

```

此处的`director`是指挥者角色，负责调度安排组装厂家即建造者，有的消费者买了宝马，有了消费者买了路虎，所以就有了以下调用方法。

``` csharp
namespace DesignPattens.Tests
{
    [TestClass]
    public class BuilderPattenTest
    {
        [TestMethod]
        public void Builder_should_work()
        {
            var director=new Director();
            Builder bmw=new BMWBuilder();
            Builder rover=new RoverBuilder();
            director.Construct(bmw);
            director.Construct(rover);
            Assert.IsInstanceOfType(bmw,typeof(Builder));
            Assert.IsTrue(bmw.GetCar().Show().Contains("Engine"));
            Assert.IsTrue(rover.GetCar().Show().Contains("Engine"));
            Assert.IsFalse(bmw.GetCar().Show().Contains("other"));
            Assert.IsFalse(rover.GetCar().Show().Contains("other"));
        }
    }
}

```

## 总结

抽象工厂模式解决了“系列产品”的需求变化，而建造者模式解决的是 “产品部分”的需求变化；由于建造者隐藏了具体产品的组装过程，所以要改变一个产品的内部表示，只需要再实现一个具体的建造者就可以了，从而能很好地应对产品组成组件的需求变化。