---
layout: page
date: 2016-04-01 19:37:48 +0800 
title: 原型模式(Prototype)
tags: [Prototype, Design Patten]
---

## 定义

用一个原型对象来指明要创建的对象类型，然后用复制这个原型对象的方法来创建出更多的同类型对象，如同细胞的分裂或者核裂变不断复制自身的过程。

## 适用场景

原型模式与工厂模式很相似，Clone方法类似于工厂方法，只不过工厂方法模式的工厂方法通过New运算符重新创建一个对象（相当于原型模式的深拷贝实现），而原型模式是通过调用MemberwiseClone方法来对原来的对象进行浅拷贝。

## UML 图

![prototypeDesignPatten](http://www.plantuml.com/plantuml/png/TOz12eD034NtEKMW6qM4dg1ONBahz0Hn9fIXcf5CNBJMkrUe1QLq_LvU-5KoKXlx3j8zo41AFRgEKZgRQMX7uyP5kOOdun6eJQRax6DWJNhHQD4U0qzGuWA3tB3nSDYnR0N1vr3WzAtCEmdhBWfdESomrgCuPJBU_2Z-dVpAvUkqTQ5YySlCDm00)

## 代码实现

```csharp
namespace PrototypeDesignPattens
{
    public abstract class CarPrototype
    {
        public string Id { get; set; }

        protected CarPrototype(string id)
        {
            this.Id = id;
        }

        public abstract CarPrototype Clone();
    }

    public class ConcreatePrototype : CarPrototype {
        public ConcreatePrototype(string id) : base(id)
        {
        }

        public override CarPrototype Clone()
        {
            return (CarPrototype) this.MemberwiseClone();
        }
    }
}
```

此处的代码实现的是浅拷贝的模式，所谓`浅拷贝`是指被复制的对象所有变量的值都与原对象保持相同，此对象里包含的对其他对象的引用还是指向原来的对象保持不变，跟一个影子一样，一个对象变化也会引起另一个对象的改变；`深拷贝`是指被复制的对象所有变量以及引用对象进行复制，即不会跟原来的对象共享任何内容，改变一个对象对另外一个对象没有任何影响。
以下是测试代码：

```csharp
namespace DesignPattens.Tests
{
    [TestClass]
    public class PrototypePattenTest
    {
        [TestMethod]
        public void Prototype_should_be_work()
        {
            CarPrototype bmw1=new ConcreatePrototype("BMW");
            var bmw2 = bmw1.Clone() as ConcreatePrototype;
            var bmw3 = bmw1.Clone() as ConcreatePrototype;
            Assert.AreEqual(bmw2.Id,bmw1.Id);
            Assert.AreEqual(bmw3.Id, bmw2.Id);
        }
    }
}
```

## 总结

.NET下System命名空间提供了IConeable接口，实现此接口下的Clone()方法，我们就可以实现原型模式了。
