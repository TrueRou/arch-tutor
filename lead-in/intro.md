# 初识 Architectury

Architectury 是一套跨加载器的模组开发工具链，依靠这套工具链，读者可以快速且舒适的**一次性**开发出模组的**Forge**和**Fabric**版本，而无需构建多套开发环境。

Architectury分为**Architectury Plugin**和**Architectury API**。

Architectury Plugin是一个Gradle插件，意在快速而方便的帮助开发者构建出一个包含Forge模块、Fabric模块和Common模块的项目。

在开发过程中，开发者只需把重心放在Common模块的实现，而为了适配两个不同加载器的**额外代码**则需要编写在forge和fabric模块中，最后构建时Architectury会帮助你进行混合。

开发者的编码组织方式类似于下图:

![](https://s2.loli.net/2022/03/28/3KqQpujYFETZin2.png)

起初开发者们认为这种开发方式的确带来了很大的便利。但因为Fabric和Forge本身存在较大的差异，开发整仍需花费大量时间在Forge模块和Fabric模块的开发上，而且开发者们仿佛都在**重复地**做同一类事情​：

* 注册物品: Common调用注册的方法。Forge里面实现一遍, Fabric里面实现一遍
* 监听事件: Common调用监听事件的方法。Forge里面实现一遍, Fabric里面实现一遍
* 网络发包: Common调用发包的方法。Forge里面实现一遍, Fabric里面实现一遍

所以Architectury API横空出世。

Architectury API在Forge和Fabric上封装了一个抽象层，开发者实际开发中可以跟这些API层打交道，组织方式类似于下图:

![](https://s2.loli.net/2022/03/28/JMs24kZCXvUaAKh.png)

所以实际上我们的开发实际上主要是围绕Architectury API进行的。

在Architectury的工具链中，还有一个重要角色就是Architectury Loom，相信之前尝试过利用Fabric开发的读者应该会想到Fabric Loom, 负责构建开发环境时反编译Minecraft源码并且反混淆的工具。

这里Architectury创造性的把这套东西搬到了Forge, 替代了Forge Gradle，感兴趣的读者可以自行了解。

可以想到的，为了便于管理，在我们的开发环境搭建中也使用Architectury Loom，构建环境时Architectury会自动为我们进行必要的操作，不需要我们手动干涉。

Architectury Loom也可以作为Forge Loom独立使用，如果看不惯Forge Gradle的读者也可以尝试独立使用它构建Forge环境，但是需要注意的是，这套框架还处于测试阶段，可能出现无法构建或者杂七杂八的问题，具体的信息可以在这里找到: [https://github.com/architectury/architectury-loom](https://github.com/architectury/architectury-loom)

相信读者已经基本了解了Architectury，下面我们开始尝试基于Architectury开发模组。请注意，构建过程中需要**通畅**的网络，请读者自行解决，不过鉴于forge-cdn和fabric最近的情况，读者应该不必担心。
