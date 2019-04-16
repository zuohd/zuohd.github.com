---
layout: page
date: 2016-04-01 19:37:48 +0800 
title: 观察者模式(Observer)
tags: [Observer, Design Patten]
---

## 定义

观察者模式又名“订阅-发布”模式，定义了一种对象之间的一对多的依赖，让多个观察者对象同时监听某一个主题对象，当主题对象发生状态变化时，它的所有依赖者（即观察者）都会收到通知并更新自己的行为。

## 适用场景

微信中的订阅号、订阅博客和QQ微博中的关注好友，这些都属于观察者模式的应用。

## UML 图

![observerDesignPatten](http://www.plantuml.com/plantuml/png/PP1FJyCm3CNl-HG-Ra91dAk1Xa12NE1GyG6Sr1E5DfuS1ua_V7SorepItABdztlFpZOgYgPnCF6r3tiK7C4F6EErglGkATrx3ct_GN1Xq4KLz7fqG9lS2tdzD6UGtUYVLuykahoH0A-pU4jQYm-i_UvzjJRVoxZYE-GzxJjK-dMPu_I6WnVAugdLbWLQpNA6xYW_2xQW_iBjgQXmix8fSVDbRTLe2dUOLi45jLT5QRYCbacR6c_YL19O2HohyWYygvPeG4YJ18YpsokeIlztykN4diy-R4kXI-Fm0m00)

可以看出，在观察者模式的结构图中有以下角色：

- 抽象主题角色（Subject）：抽象主题把所有观察者的引用保存在一个列表中，并提供增加和删除观察者对象的操作，抽象主题角色又叫做抽象被观察者角色，一般由抽象类或接口实现。
- 抽象观察者角色（Observer）：为所有具体观察者定义一个接口，在得到主题通知时更新自己，一般由抽象类或接口实现。
- 具体主题角色（ConcreateSubject）：实现抽象主题接口，具体主题角色又叫做具体被观察者角色。
- 具体观察者角色（ConcreateObserver）：实现抽象观察者角色所要求的接口，以便使自身状态与主题的状态相协调。

## 代码实现

想象客户订阅报纸的场景，顾客可是时时看到最新的报纸，而报纸可以作为一个主题，即subject，顾客就是观察者，需要及时拿到最新的报纸。

``` csharp
namespace ObserverDesignPattens
{
    public abstract class Customer
    {
        public abstract string Update(NewspaperOffice subject);
    }

    public class CustomerA:Customer
    {
        public override string Update(NewspaperOffice subject)
        {
            return $"A {subject.Message} {subject.Name}";
        }
    }

    public class CustomerB : Customer
    {
        public override string Update(NewspaperOffice subject)
        {
            return $"B {subject.Message} {subject.Name}";
        }
    }

    public  class NewspaperOffice
    {
        private readonly List<Customer> _customers=new List<Customer>();
        public string Name { get; set; }
        public string Message { get; set; }

        public NewspaperOffice(string name,string message)
        {
            this.Name = name;
            this.Message = message;
        }
        public void AddCustomer(Customer customer)
        {
            this._customers.Add(customer);
        }
        public void RemoveCustomer(Customer customer)
        {
            this._customers.Remove(customer);
        }
        public ArrayList ComeNewspaper()
        {
            var result = new ArrayList();
            foreach (var customer in _customers)
            {
                if (customer != null) { result.Add(customer.Update(this)); }
            }
            return result;
        }
    }
}

```

测试代码如下：

```csharp

namespace DesignPattens.Tests
{
    [TestClass]
    public class ObserverDesignPattensTest
    {
        [TestMethod]
        public void Observer_should_work()
        {
            var newspaperOffice=new NewspaperOffice("newspaper","subscribed");
            var a=new CustomerA();
            var b=new CustomerB();
            newspaperOffice.AddCustomer(a);
            newspaperOffice.AddCustomer(b);
            var result = newspaperOffice.ComeNewspaper();
            Assert.AreEqual(2,result.Count);
            Assert.IsTrue(result.Contains("A subscribed newspaper"));
            Assert.IsTrue(result.Contains("B subscribed newspaper"));
        }
    }
}

```

## 总结

相应的一些语言如C#和Java也内置了观察者模式的实现，Java采用了 observable 类，有兴趣的朋友可以看看其实现源码，虽然相似但考虑了线程安全，包装了 `synchronized`关键字；C# 中拖过委托和代理的方式实现了观察者模式,其代码如下：

```csharp
namespace CharpObserverDesignPattens
{
    public delegate string NotifyEventHandler();
    public abstract class Customer
    {
        public abstract string Update();
    }

    public class CustomerA : Customer
    {
        private readonly NewspaperOffice _newspaperOffice;
        public string Subject { get; set; }
        public CustomerA(NewspaperOffice newspaperOffice)
        {
            this._newspaperOffice = newspaperOffice;
        }
        public override string Update()
        {
            this.Subject = "A";
            return $"{Subject} {_newspaperOffice.Message} {_newspaperOffice.Name}";
        }
    }

    public class CustomerB : Customer
    {
        private readonly NewspaperOffice _newspaperOffice;
        public string Subject { get; set; }
        public CustomerB(NewspaperOffice newspaperOffice)
        {
            this._newspaperOffice = newspaperOffice;
        }
        public override string Update()
        {
            this.Subject = "B";
            return $"B {_newspaperOffice.Message} {_newspaperOffice.Name}";
        }
    }

    public class NewspaperOffice
    {
        public event NotifyEventHandler NotifyEvent;
        public string Name { get; set; }
        public string Message { get; set; }

        public NewspaperOffice(string name, string message)
        {
            this.Name = name;
            this.Message = message;
        }
        public void AddCustomer(NotifyEventHandler no)
        {
            NotifyEvent+=no;
        }
        public void RemoveCustomer(NotifyEventHandler no)
        {
            NotifyEvent-=no;
        }
        public void ComeNewspaper()
        {
            NotifyEvent?.Invoke();
        }
    }
}

```

以下是测试代码：

```csharp
namespace DesignPattens.Tests
{
    [TestClass]
    public class SecondObserverDesignPattensTest
    {
        [TestMethod]
        public void Observer_should_work()
        {
            var newspaperOffice=new NewspaperOffice("newspaper","subscribed");
            var a=new CustomerA(newspaperOffice);
            var b=new CustomerB(newspaperOffice);
            newspaperOffice.AddCustomer(new NotifyEventHandler(a.Update));
            newspaperOffice.AddCustomer(new NotifyEventHandler(b.Update));
            newspaperOffice.ComeNewspaper();
            Assert.IsTrue(a.Subject.Equals("A"));
            Assert.IsTrue(b.Subject.Equals("B"));
        }
    }
}
```
由此可以看出语言内部实现的方式精简了代码。