# 理解IOC和DI
本文尝试剖析IOC和DI这两个概念。
## 程序目标
首先，我们程序的开发从最简单到完善，存在两个极端：

- 最简单程度：hello world
只是输出一句hello world，怎么写都行，能输出出来就行了，这个就是目的，其他的都不用考虑。
- 完善程度：这是系统、平台、框架的要求
  - 稳定性
  - 可维护性
  - 可扩展性
这三个目标中，可扩展性和另外两个存在一定的冲突:
  - 扩展意味着添加和变动，而这个动作就可能破坏稳定性。
  - 扩展意味着需要维护的内容更多，不一致的现象也越多，从而越难维护。

IOC和DI是实现这种终极目标的方式，是一种设计思想，根据不同的需求有不同的实现。

### 实现效果

那么如果实现了第二个的目标，效果是什么呢？拿下面的例子体验下：
大家用手机的时候，有好东西要分享，不管是相册中的、微信中的、抖音中的、网页中的，会出现一个选择列表，比如现在常见的

- 微信好友
- 微信朋友圈
- 蓝牙
- 钉钉
- 抖音
- ...
这里面的方式都是安装了相应的程序，才会出现，卸载之后，就会相应的去掉。所有按照约定实现了分享功能的程序，都可以接入，不需要手机系统为这些程序做什么开发。

如果咱们开发了一个程序，然后产品经理每天都来提需求，每个功能完成后，能像分享这样，加入程序中就行，其他部分不需要为这个功能做任何改动，那么引入问题的概率就大大降低了。

那么，这个效果是怎么实现的呢？以下是我想到的流程

```plantuml
:完善模式;
start
fork
:发起分享;
fork again
partition 容器 {
:注册分享功能;
}
endfork
partition 容器 {
    :获取可以分享到的目标列表;
}
:从列表中选择目标;
partition 容器 {
:发送分享到目标;
}
stop
```

这个其实就是IOC和DI思想的一种呈现。

## 概念

### IOC—Inversion of Control: “控制反转”
这个概念通常需要搞清楚这几个问题
- 谁控制谁
- 什么是反转
- 什么是正转

### DI—Dependency Injection: “依赖注入”
这个概念通常需要搞清楚这几个问题
- 谁依赖于谁
- 为什么需要依赖
- 谁注入谁
- 注入了什么

### 剖析

为了理解，咱们分析下上面提到的分享程序的两个极端实现。

拿微信为例，其内部需要支持的分享模式：
  1. 发送给好友
  2. 分享到朋友圈
  3. 在看
  4. 收藏
  5. 发送邮件
  6. 浏览器打开
  7. 分享到手机QQ
  8. 分享到QQ空间

##### 简单模式
从简单模式出发，在没有REGIST，IOC和DI的情况下，流程是这样的：

```plantuml
:简单模式;
start
partition 加载 {
:把上面的8中分享的模块放入一个列表;
}
:发起分享;
:从8中模块，罗列给用户;
:用户选择其中一个;
partition 使用 {
:根据用户选择，构建该模块的一个实例，然后调用;
}
stop
```

一开始，没有任何模块支持分享，开发一个支持分享的模块"发送给好友"后，涉及到的其他工作：

- "加载"阶段，主程序需要知道有这个"发送给好友"的存在，所以需要放到主程序执行过程中的某段代码中，以便主程序知道
- ”使用“阶段，拿到用户的选择后，主程序需要加载相应的程序代码来执行，需要添加相应的调用逻辑

这样就诞生了下面几个问题：

- 如果这个"发送邮件"不用了，就需要在相应的位置移除代码。
- 新的"分享给上帝"加入，就需要添加到"加载"的列表中
- 然后其他的功能2,3,4,...都需要这样做。
  每一个上面的操作都可能引入问题，当然，需要修改的位置的源代码需要公开是必要的。

这种实现下，解答两个概念的部分问题
- 主程序控制其他模块
  - 主程序加载模块，调用其代码，用完后需要释放相应的资源等
  - 主程序中添加了多少模块，就支持哪些模块
- 模块依赖于主程序
  - 没有主程序，8个分享模块就不能被使用
  - 两者通过把分享模块的使用设定在主程序中而构成这种关系

##### 完善模式
该模式下的流程图在上面已经画出来了，不管是微信程序内的分享，还是分享到程序外，都是一样的模式，区别只是ioc和di的实现和使用不同。
相应的回答几个概念问题:
- 容器控制主程序。
  主程序能使用哪些模块，由容器来决定，主程序只负责调用
  - 容器中有哪些模块，取决于有哪些模块注册了
  - 看上去是模块的注册控制了容器能提供的服务，从而控制了主程序能使用哪些模块，像是被反转了。
- 主程序依赖容器提供所需要的功能模块
  - 主程序把需要的模块通知容器
  - 容器负责把各种功能模块实例化
  - 容器把“需要的模块”“注入”给主程序使用
- 容器的功能：
  - 承接各种模块的管理
  - 负责给各种需求提供合适的模块实现

## 讨论
上面的例子只是容器向程序提供需要的模块，那么进一步的，模块所需要的其他模块也可以由容器提供，这样就构成了完善了IOC/DI体系。
另外，容器提供模块需要能创建和销毁其实例，举例
- 分享的时候，如果是分享到其他app，则可能直接打开该应用，这个打开的过程，就需要加载该app的程序，然后运行，然后调用其接口
- 手机上打开地图，会提示大家选择高德、百度、腾讯等应用，相应的会打开大家选择的app
当然这部分不再这片文章的讨论中。

## 结语
IOC/DI其实是一种思想的两种表述。从使用者的需求角度去考虑，更容易理解。
另外也不必强行去解释那几个控制和依赖的问题，关键是理解怎么样消除可扩展性和稳定性、可维护性之间的冲突。




