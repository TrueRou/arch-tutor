# Environment & OnlyIn

### Transform to Environment

在Forge中, 为了标识某一方法只在逻辑服务端执行, 我们通常会添加@OnlyIn(Dists.DEDICATED\_SERVER)

在Fabric中, 为了标识某一方法只在逻辑服务端执行, 我们通常会添加@Environment(EnvType.SERVER)

为了统一这两个相同的需求, Architectury为我们添加了自动处理的映射关系.

在实际代码中, 我们只需要添加Environment注解即可, Architrctury会对Forge平台进行映射处理

![LightOverlay](https://s2.loli.net/2022/04/02/rU8y6HdeiV4oaXP.png)

