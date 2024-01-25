# 添加物品和功能

![效果图](https://s2.loli.net/2022/04/07/FMYQtNmyU5n9Wdb.png)

观察图片可以猜测, 本节运用到的知识内容:

* **DeferredRegister** - 创建并注册游戏元素
* **I18n** - 为元素添加本地化和材质
* **CompoundTag** - 为物品添加功能
* **ClientTooltipEvent** - 为物品添加Tooltips (使用封装好的事件)

下面我们来基于Architectury API来编写我们的第一个物品 —— 经验储存器.

### 创建并注册物品

在 **trou.arch.item** 包下创建 **ItemExpContainer** 类, 使其继承自**原版的**Item类

```java
位于common: trou/arch/item/ItemExpContainer.java
public class ItemExpContainer extends Item {
    public ItemExpContainer() {
        super(new Properties().stacksTo(1).tab(CreativeModeTab.TAB_TOOLS));
    }
}
```

{% hint style="info" %}
熟悉Forge开发的读者在这里可能产生疑问, 读者可能尝试设置RegistryName和TranslationKey, 结果发现没有这样的方法。

实际上, Forge为原版的Item进行了许多层封装, 读者可以尝试查阅forge模块中的Item类, 会发现它实现了IForgeItem, 继承自ForgeRegistryEntry. Forge为了易用性舍弃了轻量的结构. Forge将注册名的设置封装在了物品内部, 这与原版的逻辑不同.

而Fabric中并没有额外的封装, 而是基于Minecraft原本的样子. 读者可能意识到, 实际上Architectury是更倾向于原版, 或者说Fabric的. 所以我们在开发中, RegistryName实际上是在注册的时候提供的。

Architectury这里采用原版(Fabric)的架构, 而为Forge进行兼容是合理的。在跨平台开发中，往往要**基于低**而**适配高**，这也是可理解的。正如DisplayPort转HDMI容易, 而HDMI转DisplayPort困难的道理。
{% endhint %}

下面我们要注册这个物品, 我们采用Architectury封装的**DeferredRegister**进行注册. 相同的, 如果读者要注册其他的方块、附魔、BlockEntityType等元素, 也可以使用相同的方法. 读者可以查阅Registry中的常量

在 **trou.arch.object** 包下创建 **ModItems** 类

```java
位于common: trou/arch/object/ModItems.java
public class ModItems {
    private static final DeferredRegister<Item> ITEMS = DeferredRegister.create(Arch.MOD_ID, Registry.ITEM_REGISTRY);
    public static final RegistrySupplier<Item> EXP_CONTAINER_ITEM = ITEMS.register("exp_container_item", ItemExpContainer::new);
    public static void register() {
        ITEMS.register(); //DeferredRegister的register方法需要在init阶段被调用
    }
}
```

为了注册物品, 我们调用`DeferredRegister.create` 来创建一个注册器

针对每一个物品, 我们需要调用注册器的`register`方法来注册, 需要传入注册名和构造方法

之后, 我们在模组的**init**阶段调用`ModItems.register()`方法

<mark style="color:red;">**因为Forge的某些限制, 我们需要在forge模块中手动注册一次Architectury的事件总线**</mark>

```java
位于forge: trou/arch/forge/ArchForge.java
@Mod(Arch.MOD_ID)
public class ArchForge {
    public ArchForge() {
        //这里一定要注册上Architectury的EventBus
        EventBuses.registerModEventBus(Arch.MOD_ID, FMLJavaModLoadingContext.get().getModEventBus());
        Arch.init();
    }
}
```

之后分别进入fabric和forge客户端，应该都能看到我们的紫黑块物品了。

{% hint style="info" %}
笔者这里储存了ITEMS.register返回的RegistrySupplier, 如果在代码的别处需要用到ItemExpContainer的实例时, 可以调用RegistrySupplier的get方法.

不难理解, 其实RegistrySupplier充当了保存对象和它的注册名的功能
{% endhint %}

### 为物品添加本地化和材质

common模块中的assets在编译的时候会被合并进forge和fabric模块中, 所以我们只需按照熟悉的方式将本地化文件和材质放在common模块中就可以了.

```json
位于common: assets/arch/lang/zh_cn.json
{
  "item.arch.exp_container_item": "经验储存器",
  "tooltip.exp_container": "已储存经验: %s exp"
}

位于common: assets/arch/lang/en_us.json
{
  "item.arch.exp_container_item": "Experience Container",
  "tooltip.exp_container": "Stored experience: %s exp"
}

位于common: assets/arch/models/item/exp_container_item.json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "arch:item/exp_container_item"
  }
}

位于common: assets/arch/textures/item/exp_container_item.png
// 在这里放置物品的材质
```

### 为物品添加功能

根据我们的需求, 需要覆盖物品的use方法, 并编写我们的逻辑, 这里相信有经验的读者可以完成

```java
位于common: trou/arch/item/ItemExpContainer.java
@Override
public InteractionResultHolder<ItemStack> use(@NotNull Level level, @NotNull Player player, @NotNull InteractionHand usedHand) {
    ItemStack stack = player.getItemInHand(usedHand);
    if (level.isClientSide || usedHand == InteractionHand.OFF_HAND) return InteractionResultHolder.fail(stack);
    CompoundTag tag = stack.hasTag() ? stack.getTag() : new CompoundTag();
    assert tag != null;
    if (player.isShiftKeyDown()) {
        player.giveExperiencePoints(tag.getInt("exp"));
        tag.putInt("exp", 0);
    } else {
        int exp = player.totalExperience;
        player.giveExperiencePoints(-exp);
        tag.putInt("exp", tag.getInt("exp") + exp);
    }
    stack.setTag(tag);
    return InteractionResultHolder.sidedSuccess(stack, level.isClientSide());
}
```

### 监听Tooltip事件



**ArchtecturyAPI**建立了一套位于Forge和Fabric之上的事件封装层. 在Forge中, 我们创建方法并添加@SubscribeEvent注解来让Forge注册我们的事件. 而在**ArchtecturyAPI**中, 我们使用一种更为"静态"的方式



ArchitecturyAPI为我们提供了一些常用的事件, 在上一章节可以找到.&#x20;

这里我们需要监听**ClientTooltipEvent**事件, 首先创建一个类来注册所有的事件

```java
位于common: trou/arch/object/ModEvents.java
public class ModEvents {
    public static void register() {
        // 需要传入的是事件处理器的方法
        ClientTooltipEvent.ITEM.register(ItemExpContainer::append);
    }
}

位于common: trou/arch/Arch.java
public class Arch {
    public static void init() {
        ModEvents.register(); // 在init阶段注册事件
    }
}
```

然后我们按照代码提示, 在ItemExpContainer类中补全append方法来处理事件

```java
位于common: trou/arch/item/ItemExpContainer.java
public static void append(ItemStack stack, List<Component> lines, TooltipFlag flag) {
    if (stack.getItem() instanceof ItemExpContainer) {
        CompoundTag tag = stack.hasTag() ? stack.getTag() : new CompoundTag();
        assert tag != null;
        lines.add(new TranslatableComponent("tooltip.exp_container", tag.getInt("exp")));
    }
}
```

{% hint style="info" %}
ClientTooltipEvent.ITEM.register背后实际上是利用@ExpectPlatform来处理的

针对两个不同的平台, API分别写了相应的事件处理器的实现, 因此简化了我们对于事件的注册

对于一个特定的需求, 首先我们要检查API是否已经为我们提供了相应的方法

如果没有提供相应的方法, 我们再去手动针对两个平台进行实现
{% endhint %}

最后进入游戏, 我们的第一个物品已经成功被加入了
