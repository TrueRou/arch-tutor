# ExpectPlatform

### ExpectPlatform

读者应该已经对ExpectPlatform有了一个基本的认识, ExpectPlatform是Architectury Plugin的一个重要注解, 它可以根据平台的不同将**一个方法**映射到**不同的实现**上去.

Architectury对ExpectPlatform修饰的方法有一些规范和要求:&#x20;

* 被修饰的方法必须是**公开**且**静态**的
* 方法的实现, 命名必须是**原方法名+Impl**
* 方法的实现, 应该存放到forge模块, 相同包下名为forge的包中
* 方法的实现, 应该存放到fabric模块, 相同包下名为fabric的包中

在common模块中创建类Storyteller, 并创建一个**公开静态**方法, 添加ExpectPlatform注解

```java
位于common: trou/arch/Storyteller.java
@ExpectPlatform
public static void tellStory() {
    // 方法体为空, Architectury会根据不同的模组加载器自动填充
}
```

在fabric和forge模块中**相同位置**创建类Storyteller**Impl**, 为它添加实现

```java
位于fabric: trou/arch/fabric/StorytellerImpl.java
public class StorytellerImpl {
    public static void tellStory() {
        System.out.println("I'm fabric, your new friend.");
    }
}
```

```java
位于forge: trou/arch/forge/StorytellerImpl.java
public class StorytellerImpl {
    public static void tellStory() {
        System.out.println("I'm forge, your old friend.");
    }
}
```

在common包中, 我们就可以按需调用tellStory方法, 编译时Architectury会根据平台的不同补齐方法

```java
位于common: trou/arch/Arch.java
public static void init() {
    System.out.println("Hello Architectury!");
    Storyteller.tellStory();
}
```

此时分别运行Forge和Fabric客户端, 输出的信息应该有所不同.

{% hint style="info" %}
我们的文件结构应该是这样的:&#x20;

ExpectPlatform方法: common\src\main\java\trou\arch\Storyteller.java

Forge平台的实现: forge\src\main\java\trou\arch**\forge\\**StorytellerImpl.java

Fabric平台的实现: fabric\src\main\java\trou\arch**\fabric\\**StorytellerImpl.java
{% endhint %}

![Forge Side](https://s2.loli.net/2022/04/02/TQ4f6vJWEqDRglF.png)

![Fabric Side](https://s2.loli.net/2022/04/02/Xc6QYMF8Es3hIai.png)

### IDEA的Architectury插件

Architectury提供了IDEA的插件来帮助我们补全ExpectPlatform的实现

{% embed url="https://plugins.jetbrains.com/plugin/16210-architectury" %}
Jetbrins Marketplace
{% endembed %}

<div align="left">

<img src="https://s2.loli.net/2022/04/02/JBgFSlKcz12d7iI.png" alt="Code Hint">

</div>

创建ExpectPlatform方法后, 可以自动在正确的位置创建forge和fabric的实现
