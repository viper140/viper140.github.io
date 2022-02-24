---
title: '可能被你误解的 DRY 原则'
date: 2020-11-13T01:00:00+08:00
tags: ['编程思想', '代码规范']
author: '王峰'
---

我相信你或多或少听说过 DRY 原则，以至于看到标题后嗤之以鼻。但我猜你理解的肯定有所偏差，本文会带你重新回顾一下 DRY 原则，修正多数人的理解误区。

<!--more-->

<!-- slide -->

## 1 引言

**DRY，Don't Repeat Yourself**：多个地方表达相同的含义。

### 1.1 如何理解 DRY 原则

DRY 针对的是`知识和意图`的复制，强调多个地方表达的东西其实是相同的，只是表达方式不同。

**理解误区**：有些人将 DRY 固化为编码规范，这是狭隘的。至少，别把它理解为“不要复制粘贴代码”，它和你想的不一样。

### 1.2 Repeat Yourself  带来了哪些问题

- 以文档为例，假如同一个文档有多个副本，变更其中一个，就必须记得变更其他，维护负担很重。
- 以代码为例，同一段代码抄来抄去，如果遇到需求变更，迟早会改漏。如果换个人维护，就更别指望了。

## 2 DRY 原则描述了哪些重复现象

### 2.1 代码重复

“代码重复”就是很多人对 DRY 全部的理解，但还不准确，“复制粘贴”代码只是代码重复的一种特例，很多情况下，重复是不明显的：

```python
# 定义账户类
class MyAccount(object):
    __slots__ = ['fee', 'balance']
    def __init__(self, fee, balance):
        self.fee = fee
        self.balance = balance

# 定义打印函数
def printAccount(account):
    if account.fee < 0:
        print('fee:%10.2f' % -account.fee)
    else:
        print('fee:%10.2f' % account.fee)

    if account.balance < 0:
        print('balance:%10.2f' % -account.balance)
    else:
        print('balance:%10.2f' % account.balance)

# 函数调用
myAccount = MyAccount(100, -300)
printAccount(myAccount)
```

以上代码没有复制粘贴，但仍有两处重复。

- 负数处理的逻辑重复
- 打印逻辑的重复

修改后：

  ```python
  def printAccount(account):
    myPrint('fee', account.fee)
    myPrint('balance', account.balance)

  def myPrint(label, amount):
    print('%s: %10.2f' %(label, formatMoney(amount)))

  def formatMoney(amount):
    return abs(amount)
  ```

> Q：思考一下，所有代码的重复都是知识的重复么？

### 2.2 文档重复

这里的文档是广义上的，还包括注释等。

比如方法的注释把方法中的逻辑分支都描述了一遍，函数的意图就被描述了两次（注释、代码各一次）。只要经过两次需求变更，代码和注释就会变得不一致。

<!-- slide vertical = true -->

```java
private static boolean isAllTicketsStatusOk(FlightOrder order, BigOrder bigOrder) {
  /*
    * 三种情况，都能申请邮递：
    * 1. 退票单 || 该机票已经退了
    * 2. 飞后寄 && 已乘机（只作用于新单，新单乘机后，自行递归原单进行处理）
    * 3. 立即寄 && 已出票、已改签、已乘机（由于打印行程单任务可能拖延时间，故加入已乘机）
    */
  Predicate<FlightOrderTicket> isTicketStatusOk = item -> item.getStatus() == Returned
      || bigOrder.getDistInfo().getDistMode() == DistMode.DistAfterRide && item.getStatus() == Ride
      // Changed 表达的是原单
      || bigOrder.getDistInfo().getDistMode() == DistMode.DistImmediately && item.getStatus().in(Changed, TicketSuccess, Ride);

  boolean isAllTicketsStatusOk = order.getOrderTickets().stream()
      .allMatch(isTicketStatusOk);

  LOG.debug("isAllTicketsStatusOk: {}", isAllTicketsStatusOk);

  return isAllTicketsStatusOk;
}
```

<!-- slide vertical = true -->

写注释是好习惯，这里绝对不是让你为了规避 DRY 原则，把注释全部删掉。

但是，**注释掩盖不了糟糕的代码**。

如果是为了掩饰方法中糟糕或者晦涩难懂的代码，这时候应该重构代码。

<!-- slide vertical = true -->

推荐：

- 方法名准确的描述方法要做什么，方法内每行代码都写的像注释一样清晰易懂，注释则可以移除。
- 对于 if-else 分支多的场景，不要试图用注释把复杂的逻辑讲清楚，多借鉴一下设计模式来优化代码结构，比如**策略模式**、**模板方法模式**等。
![img](https://github.com/viper140/viper140.github.io/blob/master/_posts/2020/11/imgs/if-else.png)

<!-- slide vertical = true -->

### 2.3 数据重复

其实就是常说的数据冗余。

<!-- slide vertical = true -->

```java
class Line {
  Point x;
  Point y;
  double length;
}
```

x、y 两点即可确定连线的长度，length 字段明显重复了，应该改成方法：

```java
double length() {

}
```

<!-- slide vertical = true -->

即使不在同一个类，也可能构成重复。

举个例子，假设一个行程 route，包含多个航段 segment，route 有的距离，segment 上也有距离。

<!-- slide vertical = true -->

```java
class Route {
  List<Segment> segments;
  int distance;
}

class Segment {
  int distance;
}
```

<!-- slide vertical = true -->

route 上的距离是其下所有 segment 距离之和，定义成字段，就重复了，改成方法。

```java
class Route {
  List<Segment> segments;

  int getDistance() {
    return this.segments.stream()
          .map(item -> item.getDistance())
          .sum();
  }
}

class Segment {
  int distance;
}
```

<!-- slide vertical = true -->

数据库实体类比较特殊，有时候要考虑**性能因素**，采取了冗余措施。

比如下面例子的 amount 字段。

<!-- slide vertical = true -->

```java
@Table
class FlightOrder {
  BigDecimal amount;
  List<FlightOrderPassenger> passengers;
}

@Table
class FlightOrderPassenger {
  BigDecimal amount;
  List<FlightOrderPassengerTicket> tickets;
  FlightOrder belongedOrder;
}

@Table
class FlightOrderPassengerTicket {
  BigDecimal amount;
  FlightOrderPassenger belongedPassenger;
}
```

<!-- slide vertical = true -->

将更新冗余字段的逻辑**封装在类内部**，**集中处理**。

```java
@Table
class FlightOrderPassengerTicket {
  BigDecimal amount;
  FlightOrderPassenger belongedPassenger;

  void setAmount(BigDecimal amount) {
    this.amount = amount;
    this.resetOrderAmout();
  }

  void addAmount (BigDecimal amount) {
    this.amount = this.amount + amount;
    this.resetOrderAmout();
  }

  private resetOrderAmout() {
    var passenger = this.belongedPassenger;
    passenger.amount = passenger.getTickets().stream()
      .reduce(BigDecimal.ZERO, BigDecimal::add);

    var order = this.belongedPassenger.belongedOrder;
    order.amount = order.getPassengers().stream()
      .reduce(BigDecimal.ZERO, BigDecimal::add);
  }
}
```

<!-- slide vertical = true -->

推荐：

- 合理使用**方法替换属性**，来去除重复。
- 不得不冗余的地方，将相关的逻辑尽可能控制在局部，来减少重复带来的风险。

<!-- slide vertical = true -->

### 2.4 表征重复

主要描述的和外部打交道的时候，不可避免的重复。代码必须持有外部系统已经蕴含的知识（表征）。包括 API、数据 schema，错误码等等。

<!-- slide vertical = true -->

#### 2.4.1 API 的重复

对于 API，客户端代码、API 定义、服务端代码，两两之间存在重复。

<!-- slide vertical = true -->

推荐：

- 使用 swagger 等 API 管理工具、框架。
- 使用 lib 包，可以封装实体类，甚至更进一步，把远程调用的代码也封装进来。

以上两种方式，都消除了 API 定义、服务端代码之间的重复，不足是无法消灭客户端重复，但也可以非常便利的手动触发完成重复消除。

<!-- slide vertical = true -->

#### 2.4.2 数据源的重复

实体类对数据表的定义和数据库实际的表结构存在重复。

<!-- slide vertical = true -->

推荐：

- 借助 orm 框架，自动实现对象和关系型数据库的映射。这是一种方式，但需要谨慎对待，数据问题无小事。
- 合理采用 json 字符串类型字段，那么它是什么就完全由客户端决定了，消除了数据表那边的定义。

<!-- slide vertical = true -->

### 2.5 开发人员重复

准确的说，是不同服务（开发人员）对同一需求都有自己的实现。它最大的问题是不在同一服务很难实现共用，尤其是前后端，存在语言壁垒。

<!-- slide vertical = true -->

最容易产生重复的就是校验规则，手机号、邮箱校验等，不同服务不同的开发人员都有自己的实现。

<!-- slide vertical = true -->

推荐：

- 加强沟通。
- 建立知识库。
（不得不说上面两句话是无比正确的废话）
- 同语言的服务抽取通用 lib。
- 同语言的服务，在项目构建工具的帮助下，在同一个仓库中组织起来，依赖公共组件服务。

但是，**无法破除跨语言的壁垒**。

<!-- slide -->

## 3 总结

<!-- slide vertical = true -->

- DRY 原则描述的重复是 `知识和意图` 的重复。包含 `代码重复`、`文档重复`、`数据重复`、`表征重复`、`开发人员重复`。
- 这些类型的重复，很多都需要消灭，但不是全部：
  - 有些重复并不一定是的`知识和意图` 的重复，消灭反而讲不通。
  - 有些`知识`的重复，因为性能等方面的考量，予以保留，但应妥善对待。
  - 还有些重复没法根除。

<!-- slide -->

## 4 最后的忠告

<!-- slide vertical = true -->

规则终究是规则，思想终究是思想。实践起来困难重重，忠告：

- 不要打着 DRY 的旗号，提前抽象，请遵循 **Rule of three** 原则（三次原则）。
- 不要过度追求 DRY，破坏了**内聚性**，这两者很难两头都握住，很遗憾的告诉你，没有规则可言，多向经验丰富的程序员讨教。
