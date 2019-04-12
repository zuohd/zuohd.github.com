---
layout: page
date: 2016-04-01 19:37:48 +0800 
title: 策略模式(Stragety)
tags: [Stragety, Design Patten]
---

## 定义

策略模式定义了算法族，分别封装起来，让他们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。
该模式涉及三个角色：

- 环境上下文角色（Context）：持有一个Stragety类的引用；
- 抽象策略角色（Stragety）：这是一个抽象角色，通常由一个抽象类或接口来实现，此角色给出所有具体策略类要实现的接口方法；
- 具体策略角色（ConcreateStragety）:包装了相关算法和行为

## 适用场景

1.一些系统需要动态的在各个算法中切换，那么这些算法可以包装到一个个的策略类中，如一些权限控制、计算器类、税收计算场景都可以考虑使用策略模式，方便未来扩展。
2.一个对象有许多行为，如果不适用模式会造成很多`if-else`判断，就可以抽象出行为到具体的策略类中。

## UML 图

![stragetyDesignPatten](http://www.plantuml.com/plantuml/png/TOyz3e9W38RtdgAZnaWyW0CQ5mOdl4250etGp_GhclXpTn7YmE3Ojy_xD4sZatdVjH0lebSowh1eASQTkycvTywAeBay63SWwcmL5OpPgU6Edx32E61QOzw4-vWjbl26qO3ZV2IbcksBteAKcBF_k3_boOtRLvBWWIlgswanM-nDwcQUhyFi8gIivV3O1m00)

## 代码实现

```csharp
namespace StragetyDesignPattens
{
    public interface ITheme
    {
        string ShowTheme();
    }

    public class DefaultTheme:ITheme
    {
        public string ShowTheme()
        {
            return "Default theme";
        }
    }

    public class BrightTheme:ITheme
    {
        public string ShowTheme()
        {
            return "Bright theme";
        }
    }

    public class ThemeManager
    {
        private ITheme _theme;

        public ThemeManager(ITheme theme)
        {
            this._theme = theme;
        }

        public string ShowTheme()
        {
            return this._theme.ShowTheme();
        }
    }
}
```

使用策略者模式代码如下：

```csharp
namespace DesignPattens.Tests
{
    [TestClass]
    public class StragetyPattenTest
    {
        [TestMethod]
        public void TestMethod1()
        {
            ThemeManager manager=new ThemeManager();
            manager.SetTheme(new DefaultTheme());
            Assert.AreEqual("Default theme",manager.ShowTheme());
            manager.SetTheme(new BrightTheme());
            Assert.AreEqual("Bright theme",manager.ShowTheme());
        }
    }
}
```

当然这里我们修改主题使用了 `new BrightTheme()`,如果要修改为其他主题就要修改该行代码，我们可以用依赖注入或者将此类配置到Xml中利用反射动态修改主题,因此更换主题也只需要修改配置文件即可。

## 总结

其实策略不一定命名为 Stragety，Context 不一定叫Context，可以根据实际情况自己命名,如我们的ThemeManager。策略模式充分体现了 `开闭原则`（对扩展开发，对修改关闭）。
