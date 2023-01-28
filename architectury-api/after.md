# 在这之后的事情

### 结束了，吗？

纵观Architectury, 我们尚未提到的有Hooks和Fluids.

Hooks表示ArchAPI已经预先为你写好的一部分"ExpectPlatform"代码, 这部分可能随着Architectury的不断迭代, 以及开发者不断增加的新需求逐渐扩张, 笔者这里不打算铺开详细说明.

当读者遇到可能要用到ExpectPlatform来实现某种需求时, 不如先查阅一下ArchitecturyAPI是否已经实现了你的需求.&#x20;

<figure><img src="https://s2.loli.net/2023/01/29/zS67e9QyP2flpM8.png" alt=""><figcaption><p>Hooks</p></figcaption></figure>

至于Fluids, 有经验的读者可能意识到了问题. Forge独创性地实现了自己的一套FluidStack系统, 其不基于原版, 是一套独立的概念. 很明显这种独立性极强的特性是很难做到跨加载器的.

为了解决这个问题, ArchitecturyAPI基于Forge的想法重新实现了Fluid系统, 使之可以在Fabric上兼容, 感兴趣的读者可以直接阅读ArchitecturyAPI的相关文档

{% embed url="https://docs.architectury.dev/api/fluid" %}
Architectury API Fluid
{% endembed %}

### 学到了，吗？

与其说本篇教程是Architectury的开发指南, 不如说是学习新设计模式的指南.

笔者的意图是让读者以Minecraft这一熟悉的领域, 扩展到实际开发中更为宽广的领域.

通过接触跨加载器模组开发, 读者应该对跨平台开发的原理有了一定程度的了解, 这种一次编写, 到处实现的思路将贯彻跨平台开发的始终.&#x20;

或许读者曾使用过跨平台开发框架, 但未必能与其原理有深入接触, 笔者希望能借此机会使读者能亲手实现跨平台方法, 亲身体会跨平台的设计模式. 相信读者在之后的开发道路中一定会因此而获益

经由强大的Architectury, 读者还可以发挥想象, 可以尝试亲手实现跨加载器的能源系统, 编写跨加载器的科技Mod, 甚至有机会将自己的实现合并到Architectury仓库中. 有了成型的设计思想, 相信读者能够凭借自己的能力获取需要的知识, 实现自己的需求

请记住, 你学到的不仅仅是知识本身, 更是获取知识的方式.&#x20;
