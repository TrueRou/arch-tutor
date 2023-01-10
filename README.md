# 缘由和启发

### 本文的想法来源

前些天在Curseforge的Library区闲逛，发现了一直在前排榜单上的**Architectury API**，笔者稍加了解以后决定起笔这篇文章。类似于**Architectury**这种的跨平台思路最近还是挺流行的，比如flutter，uno什么的，但是在Minecraft开发领域并不多见，查了一下Curseforge上依赖**Architectury API**的Mod，发现还挺多的，可能也算比较流行了。

### 本文的受众对象

* **有跨加载器开发模组的需求 (如果没有这个需求的话就没必要阅读了)**
* 对Minecraft及其生态链有一个较为进阶的了解
* **有利用Fabric或Forge开发Mod的经验 (本文不会讲解模组开发的基础部分)**
* 有独立思维解决问题的能力

### 本文的组织方式

本文不会详细记述如何开发一个完整的Forge或Fabric模组，只会抽出开发中的几个关键点进行诠释，并加以笔者感兴趣的实例进行补充讲解。

在Architectury Plugin部分中，注重于介绍**Architectury**的特性和使用方式。所以，这部分**不是**思路引领性的，**而是**知识引领性的。

而Architectury API部分中，注重于介绍**Architectury**的实际使用和开发经验。主要以实例的角度，这部分是知识引领性的，熟悉笔者写作习惯的读者应该更乐于去接受这种方式。

对于希望快速上手的读者, 可以阅读完前言并搭建完环境后, 直接阅读Architectury API部分.

### 本文基于

* Minecraft: 1.18.2
* Architectury: 4.1.32
* Fabric Loader: 0.13.3
* Fabric API: 0.48.0+1.18.2
* Forge: 1.18.2-40.0.17
* (Parchment: 1.18.2-2022.03.13)
