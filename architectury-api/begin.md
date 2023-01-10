# 再次开始

首先我们先从官方文档出发，看看Architectury API都为我们封装了哪些易于方法的方法和类。

**读者可以暂时跳过这部分, 进入下文的实例环节, 在以后的开发中如果有需要再来查阅.**

{% hint style="info" %}
注: 某些类的描述添加了笔者自己的理解，并且附加了可能的应用方法，以便于读者自行摸索下文的例子中没有提到的使用方法。
{% endhint %}

### 实用的抽象层

| 类名                       | 描述                                                          |
| ------------------------ | ----------------------------------------------------------- |
| Platform                 | 提供了获取当前加载器的一些规范的类(比如配置文件位置, Mod列表等方法)                       |
| Registries               | 提供了注册物品, 方块等元素的一系列方法和接口的类(比如RegistryProvider)               |
| KeyBindings              | 提供了一些关于按键绑定的方法                                              |
| CreativeTabs             | 提供了创建CreativeTab的方法                                         |
| MenuRegistry             | 提供了关于Menu界面(例如统计信息, 设置)的一些注册方法                              |
| RenderTypes              | 统一了两个加载器的RenderTypes                                        |
| ReloadListeners          | 重载Listeners的注册器, 现在已经改成了ReloadListenerRegistry, 但是官方文档还没有更新 |
| CriteriaTriggersRegistry | 提供了一些关于成就触发器的方法, 现在这个类已经弃用, 如果读者要使用, 请调用CriteriaTriggers    |
| ColorHandlers            | 提供了一些关于颜色的方法                                                |
| BlockEntityRenderers     | 提供了一些绑定BlockEntity渲染器的方法                                    |
| BiomeModifications       | 提供了一些关于生物群系的方法                                              |
| PackRepositoryHooks      | 提供了一些关于高版本行为包的钩子方法                                          |

### 网络和网络包的抽象层

| 类名             | 描述                         |
| -------------- | -------------------------- |
| NetworkManager | 提供了一个类似于Fabric的网络包管理系统     |
| NetworkChannel | 提供了一个类似Forge Channel的包收发系统 |

### 抽象出来的一些钩子方法

| 类名               | 描述                                                  |
| ---------------- | --------------------------------------------------- |
| BiomeHooks       | 提供了一套重新封装的生物群系属性. 需要获取生物群系属性的可以调用这个类中的方法            |
| BlockEntityHooks | 将BlockEntity的数据变化推送至客户端的方法, 会调用区块的blockChanged方法来推送 |
| DyeColorHooks    | 提供了将DyeColor转换成颜色的方法                                |
| EntityHooks      | 提供了关于Entity的Collision相关方法, 源文档中提供的已弃用               |
| ExplosionHooks   | 提供了从爆炸(Explosion)获取爆炸位置的方法                          |
| ItemEntityHooks  | 可以获取一个物品实体的lifespan                                 |
| PlayerHooks      | 可以获取玩家是否是fakePlayer                                 |
| ScreenHooks      | 提供了一些关于在屏幕上渲染的方法                                    |

### 抽象出来的一些事件

![事件](https://s2.loli.net/2022/04/07/DxcKzeACnqY41iN.png)

Architectury API提供了一些我们常用的事件, 这里笔者可以根据名称自行理解用途.
