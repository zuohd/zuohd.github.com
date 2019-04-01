---
layout: post
title: UML 类图注解说明 
date:   2015-03-01 12:37:48 +0800
categories: technology
tags: [UML]

---

我们有时候在看别人的源码的时候，或者去理解一个设计模式，经常要看类图。软件开发者通过 UML 沟通对象的设计，而对于简单任务来讲又很少画它，所以在此记录下来，方便查看。

*内容概览*

* Do not remove this line (it will not be displayed)
{:toc}

>类图用于描述系统中所包含的类以及它们之间的关系，帮助人们简化对系统的理解，它是系统分析和设计阶段的重要产物，也是系统编码和测试的重要模型依据。

# 类

类封装了数据和行为，是面向对象的重要组成部分，他是具有相同属性、操作、关系的对象的集合。一个类可以有多种职责，但设计好的类应该具备 *单一职责*。类通常可以分为 3 种：实体类（Entity Class）、控制类（Control Class）、边界类（Boundary Class）

- 实体类：对应系统中的每个实体，即所说的 Domain，实体类既包括存储和传输数据的类，也包括操作数据的类，一般使用数据库的表或文件来存储。如用户类、商品类，由此可以看出实体类来源于需求说明的中的名词。
- 控制类：控制类一般体现业务逻辑，提供相应的操作，即通常所说的 Service，将控制类抽象出来可以实现数据库与呈现层之间的解耦。如用户注册类、商品管理类，由此可以看出控制类一般是个动宾结构转换来的名词。
- 边界类：对应系统中外部用户交互的对象，即所说的 View/View Model，其实就是呈现层如界面类、菜单类、窗体类。

在面向对象分析和设计的初级阶段，通常先识别出实体类，绘制初始类图，此时的类图也被称为领域模型，包括实体类以及它们之间的关系。

## 类的 UML 图示

在 UML 中，类使用包含类名、属性和操作且带有分隔线的长方形来表示，如定义一个 Employee 类，它包含属性 name、age、ID以及操作 ChangeDepartment()，在 UML 类图中如下表示：

![Class diagram](https://www.plantuml.com/plantuml/svg/JSqn3i8m38NXtQVm24AzGr73q8M54rZqAKIAtIenI1NYxeHsONkVtl3trQNuKpEH9y_W8KkjgDpjYqEIM4I9wBxveakULu3VnDpTl2IBV1N5ZwCvdoF-dZ4qd5vpcdXuY4K4B2seC3ySwKCzR6hrBm00)

对应的 C# 代码如下：

``` csharp
public class Employee
{
        private string Name;
        private int Age;
        private string ID;

        public void ChangeDepartment()
        {

        }
}
```

看到图中字段和方法前有不同的表示符号，代表的是可见性，包括公有（public）、私有（private）、受保护（protected） 3种，[plant uml](http://plantuml.com/zh/class-diagram)分别用 `+、-、#` 代表。

## 类与类之间的关系

### 泛化（继承）

泛化关系用一条带箭头的实线表示两个类之间是一种继承关系，如下图：

![继承关系](https://www.plantuml.com/plantuml/svg/NO_12i8m44Jl-nLxL2G_81ugsaCFNWG_O6cMD91iIh8gelwxiLH1vpG3RmQcAKfPpp40gDV0UEH0wfEKmOy53Uu2bO9xJovJ181-ogV26U-0kEf4SUluRI_9Msn95qjuVi0sZY7TcETu8277kUvC2vk7iUagvekjiPE1irUIdl-OkVkBL3KijXvu0G00)

员工和管理人员继承了用户类的属性和方法。

### 实现

实现关系表示的是类与接口的关系，类实现的接口，如下图：

![实现关系](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuIh9BCb9LV3CAqajIajCJYrMq5LmpaaiBbPmoKnCBqhCvSg4IW4LXIHM19SK9QQdbbHo9OCL1QGMv-Sbvc1Ak89q5HHb5gSYoo6RO5EZfmVJZqyD3gCSKlDIWEu70000)

### 关联

关联是类与类之间常用的一种关系，代码中通常将一个类的对象作为另一个类的成员变量。

1.双向关联

如下图所示顾客和商品之间的关系：

![双向关联](https://www.plantuml.com/plantuml/svg/FSyn3i8m343HtQVm1IAOEu7AmePWH0mcjQX5caIniu2uEorjsIphVKlUgL6nEWJGfyHCXGRqWLJtPaNkrVZeKxp8Yt4B9XOOpnp894Nj9IiwR2UE1w7760FyQcohMXguu1iGNIwfgzwqESt3zGQVLIwh6VebQzP-SlzRpgswoRNudfGtcaA7bW1s7BlnZoy0)

```csharp
namespace ClassDiagram
{
    public class Customer
    {
        private Product[] products;
    }

    public class Product
    {
        private Customer customer;
    }
}

```

2.单向关联

如下图所示顾客和地址之间的关系：

![单向关联](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuIhEpimhI2nAp5L8paaiBdOiAIdAJ2ejIVLCpiyBpgnALJ3W0aieE9SMb-JdfXOfL7CfAEWcfgGKfHONMpb2QAvQBYwu0brTEmL7g7n6OZ6NGsfU2j0g0000)

```csharp
namespace ClassDiagram
{
    public class Customer
    {
        private Address address;
    }

    public class Address
    {
    }
}

```

3.自关联

树节点之间就是自关联关系如下图表示：

![自关联](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuIhEpimhI2nAp5L8paaiBdOiAIdAJ2ejIVLCpiyBpgnALJ3WSWP9AFZbfwHMSodeAXIN9e7OLY22gvQB2qGhwEhQAO15WTfA4ZFpYhaSKlDIW1O00000)

```csharp
public class Node
{
    private Node subNode;
}

```

4.多重性关联

多重性关联表示两个关联对象在数量上的对应关系，如一个界面Form可以拥有0个或多个按钮，下图表示：

![多重性关联](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuKhEIImkLd3BBygjvb9Gq4qgBId9pxDo0bF5qR5SjLmWEGKbGpqzGoK5NLqx1IK3FJqj9GM84Yw7rBmKe3a0)

```csharp
public class Form
{
    private Button[] buttons;
}

public class Button
{

}
```

| 表示方式  | 多重性说明   |
| ---- | -----  |
| 1..1    |表示另一个类的一个对象只与该类的一个对象有关系 |
| 0..*    |表示另一个类的一个对象与该类的零个或多个对象有关系 |
| 1..*    |表示另一个类的一个对象与该类的一个或多个对象有关系 |
| 0..1    |表示另一个类的一个对象没有或只与该类的一个对象有关系 |
| m..n    |表示另一个类的一个对象与该类最少m，最多n个对象有关系（m<=n） |

### 聚合

聚合关系表示整体与部分的关系，在聚合关系中，成员对象是整体对象的一部分，但是成员对象可以脱离整体对象而存在。
在 UML 中，聚合关系用带空心的菱形表示，如：轮胎是汽车的组成部分，但是轮胎可以脱离汽车而单独存在。

![聚合](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuKhEIImkLd1EBAhcKb3GLIZFI4tDiGe7aK2-DaX20yo3o6Y21ODJIy1YA39MPVcPALorN40T2lcwkdOAO4chvFoyaiJCl68kXzIy5A250000)

```csharp

namespace ClassDiagram
{
    public class Car
    {
        private Wheel wheel;

        #构造注入
        public Car(Wheel wheel)
        {
            this.wheel = wheel;
        }

        #设值注入
        public void SetWheel(Wheel wheel)
        {
            this.wheel = wheel;
        }
    }

    public class Wheel
    {        
    }
}

```

### 组合

组合关系和聚合很像，也是类整体与部分的关系，但是在组合关系中整体对象可以控制成员对象的生命周期，一旦整体对象不存在，成员对象也就不复存在，他们之间是“不求同年同月同日生，但求同年同月同日死的”结拜兄弟关系。组合关系在 UML 中带实心的菱形表示，如头和眼睛就是组合关系：眼睛是头的一部分，头没了，眼睛也就死翘翘了。

![组合](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuNBEIImkLl18JKofvbBGLKYjJBLohKpbKj06YsbeSjLoSO5O2bgwkdQkGDcCn68kXzIy5A1c0000)

```csharp
namespace ClassDiagram
{
    public class Head
    {
        private Eye eye;
        public Head()
        {
            this.eye = new Eye();
        }
    }

    public class Eye
    {

    }
}

```

### 依赖

最后再来说说依赖关系，也就是一个类与另一个类是调用关系，即一个类的实现需要另外一个类的协助，实际代码中我们应该避免互相依赖。依赖关系用带箭头的虚线表示，箭头指向被调用者，如下图：

![依赖](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuN9KqDEpKt1Ii59mJip9uN98pKi12WC0)

## 各种关系的强弱关系

*泛化==实现>组合>聚合>关联>依赖*

- 聚合：天下没有不散的宴席，聚散离合都无所谓，所以用空心的菱形，虚的很！
- 组合：我忘你亡，共生共死，是一根绳子上的蚂蚱，所以用实心的菱形，实打实嘛
