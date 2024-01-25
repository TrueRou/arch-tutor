# 一切的开始

### 构建开发环境

类似于fabric, architectury plugin也提供了相对应的example-mod来方便我们搭建环境。

首先来到Architectury-Templates的Releases页面，选择合适的版本

{% embed url="https://github.com/architectury/architectury-templates/releases" %}

{% hint style="info" %}
目前Architectury仅支持1.16+的版本

Architectury提供下面这几种Templates可供选择

* 1.18.2-forge.zip: 1.18.2 的基于**Forge-Loom**的**Forge开发环境**
* 1.18.2-architectury.zip: 1.18.2 跨加载器的开发环境
* 1.18.2-architectury-mixin.zip: 1.18.2 使用Mixin的跨加载器开发环境

相信阅读过初识Architectury章节的读者已经明白自己需要下载的版本了

这里我们不着重介绍Mixin, 感兴趣的读者可以阅读笔者先前的文章
{% endhint %}

本文基于Minecraft 1.18.2版本，并且我们不需要Mixin, 所以我们下载**1.18.2-architectury.zip**

将下载到的文件进行解压，**直接**使用Intellij IDEA来导入文件夹中的**build.gradle**文件

类似于Fabric和Forge的环境构建，这里Gradle会帮助我们下载必要的文件, 并且自动使用Loom进行反混淆，读者需要稍加等待

{% hint style="info" %}
build.gradle中的一些常量现在移动到了**gradle.properties**中

<img src="https://s2.loli.net/2022/04/01/iaMLz2vOkusrQoB.png" alt="image.png" data-size="original">
{% endhint %}

构建完成后, Gradle会自动为我们生成针对两个平台的运行配置

<div align="left">

<img src="https://s2.loli.net/2022/04/01/FxjZpD4ufm3IQNl.png" alt="">

</div>

{% hint style="info" %}
在Architectury项目创建完成后, 请尽量避免移动项目的位置或者给项目的目录改名. 可能会产生找不到architectury-transformer-agent.jar的问题

如果产生了这个问题, 可以自行修改启动配置中-javaagent的路径
{% endhint %}

恭喜你, 你已经构建好了一个完整的开发环境

### 简析目录结构

重新审视我们构建好的开发环境, 我们的目录结构类似这样

* common
  * src/main/java/net.examplemod
    * ExampleMod.java
    * ExampleExpectPlatform.java
* forge
  * src/main/java/net.examplemod.forge
    * ExampleModForge.java
    * ExampleExpectPlatformImpl.java
* fabric
  * src/main/java/net.examplemod.fabric
    * ExampleModFabric.java
    * ExampleExpectPlatformImpl.java

经过简单的观察, 我们发现

* 实例代码在主类**ExampleMod**中进行了物品的注册和一些基本操作
* **ExampleModForge**中添加了@Mod注解, 并调用**ExampleMod**中的init
* **ExampleModFabric**类实现了ModInitializer方法, 并调用**ExampleMod**中的init
* 实例代码在**ExampleExpectPlatform**中写了一个**"没有方法体"**的方法 getConfigDirectory
* **common**模块中的某些代码调用了getConfigDirectory
* **forge**模块中的ExampleExpectPlatformImpl用**forge独有**的方式补全了方法体
* **fabric**模块中的ExampleExpectPlatformImpl用**fabric独有**的方式补全了方法体

可以发现, 我们模组的**共性**部分在common模块中, 而**非共性**部分则分别在forge和fabric包中展现.

针对一个需要考虑兼容性的方法, 我们往往先定义"接口", 然后为接口写不同的实现(Impl), 这是Architechtury以及大部分跨平台程序设计中所共有的思想

实际上**ExampleExpectPlatform**在这里类似于接口.

### 开始旅程

实例代码已经为我们提供了一个清晰的思路, 现在我们已经可以删除掉他们, 开始我们自己的旅途了.&#x20;

在common, forge, fabric三个模块中分别建立trou.arch(.forge/.fabric)包, 并创建对应的Arch, ArchForge, ArchFabric类.

```java
位于common: trou/arch/Arch.java
public static final String MOD_ID = "arch";

public class Arch {
    public static void init() {
        System.out.println("Hello Architectury!");
    }
}
```

```java
位于forge: trou/arch/forge/ArchForge.java
@Mod(Arch.MOD_ID)
public class ArchForge {
    public ArchForge() {
        Arch.init();
    }
}
```

```java
位于fabric: trou/arch/fabric/ArchFabric.java
public class ArchFabric implements ModInitializer {
    @Override
    public void onInitialize() {
        Arch.init();
    }
}
```

根据读者以往开发Forge或Fabric的经验修改必要的文件名和参数变量, 分别启动游戏, 应该可以看到Hello Architectury!消息输出在控制台中

### 可能出现的问题

因为涉及到很多transform和黑科技, 对规范性的要求比较高, 读者可能会出现一些疏忽, 下面是一些可能出现的常见问题 ~~(笔者搭建环境的时候出现的问题)~~

#### 开发中出现错误, 其中有examplemod字样

检查是否有遗漏的examplemod没有修改, 请主要检查

* fabric.mod.json
* gradle.properties
* mods.toml
* (accesswidener)
* (mixins.json)

![Architectury Templates](https://s2.loli.net/2022/04/02/uEsX9nicqLZWAb3.png)

#### 运行Fabric调试, 模组没有加载

请检查自己是否修改了fabric.mod.json, 其中定义了Mod主类入口

打开fabric模块中的resources/fabric.mod.json, 修改entrypoints

```
"entrypoints": {
    "main": [
      "trou.arch.fabric.ArchFabric"
    ]
  },
```

#### 运行Forge调试, 抛出Modules generated错误

请检查自己的Forge主类是否放在了forge模块的对应**forge包**下

我们项目的ArchForge应位于forge\src\main\java\trou\arch**\forge\\**ArchForge.java

### 实验性内容: 使用[**Parchment**](https://github.com/ParchmentMC/Parchment)

{% hint style="danger" %}
此项内容读者可以选择性阅读. Architectury并未主动支持Parchment. 以下内容均出自笔者尝试, 不能保证是否出现问题, 如果读者想要使用, 出现了编译类问题, 请首先尝试移除Parchment
{% endhint %}

众所周知, 现在Forge和Fabric默认使用Mojang官方混淆表, 但官方混淆表并没有放出参数的混淆信息, 这导致高版本默认反编译的参数表是很难阅读的, Parchment解决了这一问题, Parchment提供了参数表的反混淆方式. 这里笔者不过多赘述Parchment本身, 国内也有相关的介绍和使用教程, 读者可以自行翻阅查看.

这里我们尝试在Architectury中引入Parchment

首先打开项目**根目录**的build.gradle, 在subprojects中按需**添加**如下内容

```
subprojects {
    repositories {
        maven { url = 'https://maven.parchmentmc.org' }
    }

    dependencies {
        minecraft "com.mojang:minecraft:${rootProject.minecraft_version}"
        mappings loom.layered() {
            officialMojangMappings()
            parchment("org.parchmentmc.data:parchment-1.18.2:2022.03.13@zip")
        }
        // mappings loom.officialMojangMappings()
    }
}
```

之后刷新Gradle项目, 等待Parchment下载并自动部署后, 再次查看参数表, 已经反混淆过了.
