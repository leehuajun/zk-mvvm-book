# 简介

## MVVM简介

MVVM [1]是源自Microsoft的设计模式Model-View-ViewModel的缩写。[2] 它是由著名的MVC模式的变体Martin Fowler引入的Presentation Model [3]的一种特殊化。 此模式具有3个角色：`View`，`Model`和`ViewModel`。 View 和 Model 起着与MVC中相同的作用。


MVVM中的ViewModel就像View的特殊控制器一样，负责将Model中的数据公开到View，并为View的用户请求提供所需的操作和逻辑。 ViewModel是View抽象的类型，其中包含View的状态和行为。 但是ViewModel应该不包含对UI组件的引用，并且对View的可视元素一无所知。 因此，View和ViewModel之间有明显的分隔，这也是MVVM模式的关键特征。 从另一个角度讲，也可以说View层就像ViewModel的UI投影。

 

> [1] : WPF Apps With The Model-View-ViewModel Design Pattern http://msdn.microsoft.com/en-us/magazine/dd419663.aspx
>
> [2] : Introduction to Model/View/ViewModel pattern for building WPF apps http://blogs.msdn.com/b/johngossman/archive/2005/10/08/478683.aspx
>
> [3] : Presentation Model http://martinfowler.com/eaaDev/PresentationModel.html

## MVVM的优势

> 将数据和逻辑与表示分离

关键特性是ViewModel对View的可视元素一无所知，从而保证了从View到ViewModel的单向依赖，从而避免了UI和ViewModel之间的相互编程涟漪效应。 因此，它具有以下优点：

- 它适合`design-by-contract`编程。[1] 只要制定好约定（contract）（显示哪些数据以及执行什么操作），ViewModel的UI设计和编码就可以并行且独立地进行。 任何一方都不会阻碍对方的前进。
- 与View的松耦合。 只要不更改约定（contract），就可以不经修改就轻松地随时更改UI设计，而无需修改ViewModel。
- 更好的可重用性。 使用通用的ViewModel为不同的设备设计不同的视图会更加容易。 对于具有更大屏幕的桌面浏览器，可以在一页上显示更多信息。 而对于显示空间有限的智能手机，无需更改（大部分）ViewModel，就可以设计基于向导的逐步操作UI。
- 更好的可测试性。 由于ViewModel不能“看到”表示层，因此开发人员可以在没有UI元素的情况下轻松地对ViewModel类进行单元测试。

> [1] : Design by contract http://en.wikipedia.org/wiki/Design_by_contract

## MVVM & ZK Bind

由于ViewModel不包含对UI组件的引用，因此需要一种在View和ViewModel之间同步数据的机制。 此外，此机制还必须接受来自View层的用户请求，并将该请求桥接到ViewModel层提供的动作和逻辑。 这种机制是MVVM设计模式的核心部分，它要么是由应用程序开发人员自己编写的数据同步代码，要么是框架提供的数据绑定系统。 ZK 6提供了一种称为ZK Bind的数据绑定机制，作为MVVM设计模式的基础架构，而binder [1]在操作整个机制中起着关键作用。 绑定器就像一个代理，负责View和ViewModel之间的通信。

![image-20200605153832368](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607161813.png)

> [1] : binder ZK Developer's [DataBinding/Binder](#_bookmark14)

## 详细操作流程

在这里，我们使用一个简单的场景来说明MVVM如何在ZK中工作。 假定用户单击一个按钮，然后出现“ Hello World”文本。 流程如下：

![image-20200605153922498](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607161819.png)



如前所述，绑定程序在UI和ViewModel之间同步数据。

1. 用户按下屏幕上的按钮（执行操作）。
2. 一个相应的事件被激发到binder。
3. binder在ViewModel中找到相应的动作逻辑（它是Command）并执行它。
4. 动作逻辑从模型层访问数据并更新ViewModel的相应属性。
5. ViewModel通知binder某些属性已更改。
6. 根据已更改的属性，binder从ViewModel加载数据。
7. Binder然后更新相应的UI组件以向用户提供视觉反馈。

