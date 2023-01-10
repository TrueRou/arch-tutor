# ArchitecturyTarget

### ArchitecturyTarget <a href="#architecturytarget" id="architecturytarget"></a>

利用**ArchitecturyTarget**类的**getCurrentTarget**方法, 可以直接获取当前运行的模组加载器平台.

{% hint style="info" %}
**ArchitecturyTarget.getCurrentTarget()需要在common包中被调用**

* 当forge环境执行到该语句时, 方法的返回值为forge
* 当fabric环境执行到该语句时, 方法的返回值为fabric
{% endhint %}

下面我们来用ArchitecturyTarget修改我们上一章节编写的Storyteller代码.

修改StoryTeller类, 去掉ExpectPlatform注解, 直接基于当前运行的加载器环境进行判断

```java
public class Storyteller {
    public static final String FORGE_STORY = "I'm forge, your old friend.";
    public static final String FABRIC_STORY = "I'm fabric, your new friend.";
    public static void tellStory() {
        final String platform = ArchitecturyTarget.getCurrentTarget();
        System.out.println(platform.equals("forge") ? FORGE_STORY : FABRIC_STORY);
    }
}
```

现在应该能得到与之前相同的效果. 读者可以根据需要取舍使用ExpectPlatform和ArchitecturyTarget
