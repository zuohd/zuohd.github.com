---
layout: page
date: 2016-04-01 19:37:48 +0800 
title: 工厂模式(Factory)
tags: [Factory, Design Patten]
---

## 定义

前面说到简单工厂模式的缺点是难以扩展，如果要加入新的汽车零件，我们需要修改简单工厂方法，这样会造成简单工厂方法的实现逻辑越来越复杂。这次我们考虑使用工厂模式把零部件的创建推迟到子类中

## 适用场景

在工厂方法模式中，工厂类与具体产品具有平行的等级结构，它们之间是一对一关系：
1.Creator 类：充当抽象工厂角色，任何具体工厂都必须继承该抽象类；

2.EngineerFactory和WheelFacotory、CoverFactory：充当具体工厂角色，用来创建具体汽车零部件对象；

3.AutomotivePart：充当抽象汽车零部件角色，具体零部件的抽象类。任何具体零件都应该继承该类。

4.Engineer、Wheel、Cover：充当具体汽车零部件角色，实现抽象汽车零部件中定义的抽象方法，由具体工厂类创建，它们之间有一一对应的关系。

## UML 图

![factory](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuKfCAYufIamkKKZEIImkLd0iBSdFpI_9B4iD02gevgg6oTS2gSoyT0NVnEnK3KskMYvKbJOrkhgkUUcPUQcfHLYamZDIKtEmY3ldbm7Lepl2kIWriII_2DbsLAqG5QceWc1NGRKYEySjCHaeKWcJWZi7JHpk79Wu5ve1UdfsC16hfnspd-xUnGKs1o4DviBam6OXvC1Sc1oNSqvcQbu9K1XgkTMB3StFfwn0YADw3IRja9gN0emq0000)

## 代码实现

```csharp
namespace FactoryPatten
{
    public abstract class AutomotivePart
    {
        public abstract string Make();
    }

    public class Engineer : AutomotivePart
    {
        public override string Make()
        {
            return this.GetType().Name;
        }
    }

    public class Wheel : AutomotivePart
    {
        public override string Make()
        {
            return this.GetType().Name;
        }
    }

    public class Cover : AutomotivePart
    {
        public override string Make()
        {
            return this.GetType().Name;
        }
    }
    public abstract class Creator
    {
        public abstract AutomotivePart CreateAutomotivePart();
    }

    public class EngineerFactory : Creator
    {
        public override AutomotivePart CreateAutomotivePart()
        {
            return new Engineer();
        }
    }

    public class WheelFactory : Creator
    {
        public override AutomotivePart CreateAutomotivePart()
        {
            return new Wheel();
        }
    }

    public class CoverFactory:Creator
    {
        public override AutomotivePart CreateAutomotivePart()
        {
            return new Cover();
        }
    }
}

```

## 总结

工厂方法模式通过面向对象编程中的多态性来将对象的创建延迟到具体工厂中，从而解决了简单工厂所存在的问题，也很好的符合了开闭原则（即对扩展开放，对修改关闭）。
