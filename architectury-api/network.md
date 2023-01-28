# 网络通信和热键

关于网络通信, ArchitecturyAPI提供了类似Fabric的Packet和类似Forge的Channel这样两种解决方案, 这里笔者将带领读者以两个具体实例切入这两种解决方案, 读者可以实际感受后选择自己喜欢的方式来进行网络通信.

### 使用NetworkManager进行网络通信

本节将使用类似Fabric的NetworkManager来写一个处理客户端鼠标滚动事件的实例. 这里笔者给上文我们提到的经验储存器添加了一个小功能, 可以让玩家用Shift+滚动来调整单次经验的存取量.

很显然, 我们的需求涉及一个C2S的网络包请求. 熟悉模组开发的读者应该清楚, 滚轮事件是由客户端发出的, 其产生的影响 (这里是改变经验储存器的一个NBT值) 是体现在逻辑服务端的. 一个携带数据的网络包需要从玩家的客户端发出, 到达逻辑服务器产生影响.

按照需求, 我们在trou.arch下创建network包, 并且创建ModPackets类

```java
位于common: trou/arch/network/ModPackets.java
public class ModPackets {
    public static final ResourceLocation EXP_CONTAINER_MODE = new ResourceLocation("arch", "exp_container_change_mode");

    public static void register() {
        NetworkManager.registerReceiver(NetworkManager.Side.C2S, EXP_CONTAINER_MODE, (buf, context) -> {
            Player player = context.getPlayer();
            int deltaMode = buf.readInt();
            ItemStack stack = player.getItemInHand(InteractionHand.MAIN_HAND);
            ItemExpContainer.raiseMode(stack, deltaMode);
        });
    }

    public static void sendExpContainerMousePacket(int deltaMode) {
        FriendlyByteBuf buf = new FriendlyByteBuf(Unpooled.buffer());
        buf.writeInt(deltaMode);
        NetworkManager.sendToServer(EXP_CONTAINER_MODE, buf);
    }
}
```

{% hint style="info" %}
FriendlyByteBuf是针对ByteBuf的一个封装, 其中包含了常用的Minecraft对象的序列化和反序列化, 读者需要时可以自行查看

需要注意的是, 由于ByteBuf的特性, 数据被写入的顺序和读取的顺序必须一致
{% endhint %}

显然, 我们调用了NetworkManager提供的方法并且完成了包的发送和接收, 包的数据由一个ByteBuf携带, 相信读者已经对这种模式不陌生了.

注册包的接收端时需要提供Side, 由于我们是由逻辑客户端发向逻辑服务端, 所以我们选择Side.C2S

context包含了一些随包携带的常用参数, 包含发起的玩家和Environment, 方便我们直接进行使用和判断.

最后, 不要忘记在主类中**调用**我们ModPackets的**register方法**

### 处理raiseMode方法

接下来我们要处理逻辑服务端触发的raiseMode方法, 我们希望通过这个方法改变手中经验储存器的单次存取量.

```java
位于common: trou/arch/item/ItemExpContainer.java
public static void raiseMode(ItemStack stack, int value) {
    if (!(stack.getItem() instanceof ItemExpContainer)) return;
    CompoundTag tag = stack.hasTag() ? stack.getTag() : new CompoundTag();
    assert tag != null;
    int newValue = Math.max(tag.getInt("mode") + value, 0); // 限制不为负
    tag.putInt("mode", newValue);
}
```

相信不用笔者过多赘述, 读者可以理解其中的逻辑.

我们还需要修改存取经验时的逻辑, 使其按照我们设定的数值进行存取

```java
位于common: trou/arch/item/ItemExpContainer.java
@Override
public InteractionResultHolder<ItemStack> use(@NotNull Level level, @NotNull Player player, @NotNull InteractionHand usedHand) {
    ItemStack stack = player.getItemInHand(usedHand);
    if (level.isClientSide || usedHand == InteractionHand.OFF_HAND) return InteractionResultHolder.fail(stack);
    CompoundTag tag = stack.hasTag() ? stack.getTag() : new CompoundTag();
    assert tag != null;
    int mode = tag.getInt("mode");
    if (player.isShiftKeyDown()) {
        int storedExp = tag.getInt("exp");
        int amount = Math.min(mode, storedExp);
        player.giveExperiencePoints(amount);
        tag.putInt("exp", storedExp - amount);
    } else {
        int ownedExp = player.totalExperience;
        int cost = Math.min(ownedExp, mode);
        player.giveExperiencePoints(-cost);
        tag.putInt("exp", tag.getInt("exp") + cost);
    }
    stack.setTag(tag);
    return InteractionResultHolder.sidedSuccess(stack, level.isClientSide());
}
```

最后使单次存取量可以在Tooltip显示, 并添加相关i18n词条

<pre class="language-java"><code class="lang-java">位于common: trou/arch/item/ItemExpContainer.java
<strong>public static void appendTooltip(ItemStack stack, List&#x3C;Component> lines, TooltipFlag flag) {
</strong>    if (stack.getItem() instanceof ItemExpContainer) {
        CompoundTag tag = stack.hasTag() ? stack.getTag() : new CompoundTag();
        assert tag != null;
        lines.add(new TranslatableComponent("tooltip.exp_container", tag.getInt("exp")));
        lines.add(new TranslatableComponent("tooltip.exp_container_mode", tag.getInt("mode")));
    }
}
</code></pre>

### 监听鼠标滚动事件

接下来我们来完成监听鼠标滚轮的相关事件, 相信读过上一节的读者已经对此有头绪了. 经过寻找, 我们命中了ClientRawInputEvent.MOUSE\_SCROLLED事件, 这个事件会在玩家进入世界后滚动滚轮触发, 并且在打开GUI等界面时不会触发, 完美符合我们的需求.

修改ModEvents类来监听这个事件, 并且在ItemExpContainer类中编写事件处理器

```java
位于common: trou/arch/object/ModEvents.java
public class ModEvents {
    public static void register() {
        ClientTooltipEvent.ITEM.register(ItemExpContainer::appendTooltip);
        ClientRawInputEvent.MOUSE_SCROLLED.register(ItemExpContainer::mouseScroll);
    }
}
```

显然 ClientRawInputEvent 只会在客户端进入世界后触发, 我们可以直接用 minecraft.player 来拿到非空的本地玩家

```java
位于common: trou/arch/item/ItemExpContainer.java
// 向上滚动 v = 1.0 向下滚动 v = -1.0
public static EventResult mouseScroll(Minecraft minecraft, double v) {
    assert minecraft.player != null; // 断定minecraft.player不为空
    ItemStack heldItem = minecraft.player.getItemInHand(InteractionHand.MAIN_HAND);
    if (heldItem.getItem() instanceof ItemExpContainer) {
        if (minecraft.player.isShiftKeyDown()) {
            ModPackets.sendExpContainerMousePacket((int) v);
            return EventResult.interruptFalse();
        }
    }
    return EventResult.pass();
}
```

{% hint style="info" %}
EventResult.interruptFalse() 会阻碍接下来滚轮的滚动, 使事件冒泡停止

EventResult.pass() 不会影响接下来滚轮的滚动
{% endhint %}

到这里, 我们的功能已经写好了, 读者可以启动游戏验证我们的代码, 确保其在Forge和Fabric均能正常使用

<figure><img src="https://s2.loli.net/2023/01/29/gtjy83Q7NnvPMoI.png" alt=""><figcaption><p>经验储存器</p></figcaption></figure>

### 使用NetworkChannel进行网络通信

本节我们将使用类似Forge的NetworkChannel进行网络通信, 做出一个点击热键即可将储存器中的经验丢给盯着的玩家的功能

NetworkChannel的核心概念是Channel和Message, 当网络通信时, 我们的视觉效果是Message在Channel中传递, 显然, 丢经验这个动作应该属于一条Message

所以我们在network包中创建MessageThrowExp类, 来诠释丢经验这条消息

```java
位于common: trou/arch/network/MessageThrowExp.java
public class MessageThrowExp {
    public final UUID targetPlayerUUID;
    
    // 收到消息时, 会调用这个方法来从buf中取出数据
    public MessageThrowExp(FriendlyByteBuf buf) {
        this(buf.readUUID());
    }

    // 创建消息时提供数据的构造器
    public MessageThrowExp(UUID targetPlayerUUID) {
        this.targetPlayerUUID = targetPlayerUUID;
    }

    // 发送消息时将数据写入buf的方法
    public void encode(FriendlyByteBuf buf) {
        buf.writeUUID(targetPlayerUUID);
    }

    // 收到消息后, 调用的处理事件
    public void apply(Supplier<NetworkManager.PacketContext> contextSupplier) {
        Player self = contextSupplier.get().getPlayer();
        Player target = self.level.getPlayerByUUID(targetPlayerUUID);
        if (target == null) return;
        ItemStack stack = self.getItemInHand(InteractionHand.MAIN_HAND);
        ItemExpContainer.doThrow(stack, target);
    }
}
```

{% hint style="info" %}
请永远使用UUID来作为玩家的标识符, 而不要传递Player对象或者玩家的名字
{% endhint %}

显然, 这里我们要承载的数据是指向的玩家的UUID, 我们需要编写一个构造器来提供成员变量的值, 并且在encode时将值编码到ByteBuf中

apply方法会在ByteBuf的值序列化到实例中后被逻辑服务器调用, 这里可以处理我们的逻辑. 与上文中的context相类似, contextSupplier会提供Player, Environment等可能用到的上下文对象.

完成了Message, 接下来我们需要定义并注册承载它的Channel, 在network包下创建ModChannels类

```java
位于common: trou/arch/network/ModChannels.java
public class ModChannels {
    public static final NetworkChannel CHANNEL = NetworkChannel.create(new ResourceLocation("arch", "exp_container"));

    public static void register() {
        CHANNEL.register(MessageThrowExp.class, MessageThrowExp::encode, MessageThrowExp::new, MessageThrowExp::apply);
    }
}
```

{% hint style="info" %}
根据读者对Channel的不同理解, 你可以在一个Mod使用多个Channel承载不同功能, 也可以只使用一个Channel来承载该Mod所有的Message, 这只是设计模式的差别, 读者可以根据自己的喜好来决定.

可以看到这里笔者创建的Channel的标识符中提到了exp\_container, 即为经验储存器创建了一个单独的Channel
{% endhint %}

最后, 我们需要在主类中调用ModChannels.register()方法

### 处理doThrow方法

相信这里不必笔者多说, 读者应该已经想出了相关逻辑

```java
位于common: trou/arch/item/ItemExpContainer.java
public static void doThrow(ItemStack stack, Player target) {
    if (!(stack.getItem() instanceof ItemExpContainer)) return;
    CompoundTag tag = stack.hasTag() ? stack.getTag() : new CompoundTag();
    assert tag != null;
    int storedExp = tag.getInt("exp");
    int mode = tag.getInt("mode");
    int amount = Math.min(mode, storedExp);
    target.giveExperiencePoints(amount);
    tag.putInt("exp", storedExp - amount);
}
```

当执行这个方法的时候, 会从经验提取器中提取指定量的经验, 给予目标玩家

### 添加一个热键

ArchitecturyAPI为我们提供了通用的添加热键的方式.

创建trou.arch.client包, 并创建ModKeys类, 创建我们的KeyMapping

```java
位于common: trou/arch/client/ModKeys.java
public class ModKeys {
    public static final KeyMapping THROW_EXP = new KeyMapping("key.arch.throw_exp", InputConstants.Type.KEYSYM, InputConstants.KEY_Z, "category.arch");

    public static void register() {
        KeyMappingRegistry.register(THROW_EXP);
    }
}
```

KeyMapping的构造器需要四个参数

1. 热键的本地化名称: 将显示在选项-控制中
2. 热键的类型: 可选值为 KEYSYM, SCANCODE, MOUSE. 分别对应键盘, 组合键, 鼠标
3. 热键的值: 具体取决于类型, 这里InputConstants.KEY\_Z代表键盘的Z键
4. 热键类别的本地化名称: 也将显示在选项-控制中

接下来我们要为热键赋予功能, 根据ArchitecturyAPI文档中的介绍, 我们要监听ClientTickEvent, 并且使用一个while来判断热键是否被按下.

<figure><img src="https://s2.loli.net/2023/01/29/argPSelXFoEHBzq.png" alt=""><figcaption><p>ArchitecturyAPI文档</p></figcaption></figure>

这里我们在ModEvents中监听事件, 并且编写事件处理器

```java
位于common: trou/arch/object/ModEvents.java
public class ModEvents {
    public static void register() {
        ...
        ClientTickEvent.CLIENT_POST.register(ModKeys::handleThrowExpKey);
    }
}

位于位于common: trou/arch/client/ModKeys.java
public static void handleThrowExpKey(Minecraft minecraft) {
    while (THROW_EXP.consumeClick()) {
        if (minecraft.player == null) return;
        // minecraft.crosshairPickEntity 可以获取到玩家光标瞄准的实体
        if (minecraft.crosshairPickEntity instanceof Player) {
            UUID playerUUID = minecraft.crosshairPickEntity.getUUID();
            // Channel提供了发送Message的几个方法
            // 这里使用了我们创建的Message的构造器来提供playerUUID数据
            ModChannels.CHANNEL.sendToServer(new MessageThrowExp(playerUUID));
        }
    }
}
```

最后, 我们在主类中调用ModKeys.register方法来注册热键, 为了防止问题, 我们让热键在事件之前注册

此时进入游戏, 你应该能看到添加的热键了, 并且能够使用热键给玩家丢经验了

<figure><img src="https://s2.loli.net/2023/01/29/cMLXQseYhElibv3.png" alt=""><figcaption></figcaption></figure>

### 总结

阅读上文的代码后, 你的主类应该是这样的

```java
位于common: trou/arch/Arch.java
public class Arch {
    public static final String MOD_ID = "arch";

    public static void init() {
        System.out.println("Hello Architectury!");
        Storyteller.tellStory();
        ModItems.register();
        ModKeys.register();
        ModEvents.register();
        ModPackets.register();
        ModChannels.register();
    }
}
```

本章我们主要介绍了两种网络通信的方式, 使用NetworkManager较为简便, 使用NetworkChannel较为灵活, 这里读者可以根据自己的需求来选择.

如果你对Fabric熟悉, 或需求较为单一, 推荐使用NetworkManager

如果你对Forge熟悉, 或需求较为复杂, 希望可读性强一些, 推荐使用NetworkChannel
