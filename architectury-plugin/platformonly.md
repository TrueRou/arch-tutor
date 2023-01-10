# PlatformOnly

### PlatformOnly

PlatformOnly是一个用于限定某一方法只能在某一平台执行的注解.

* @PlatformOnly("forge")修饰的方法**只能**在Forge模块调用(编译后不会出现在Fabric代码中)
* @PlatformOnly("fabric")修饰的方法**只能**在Fabric模块调用(编译后不会出现在Forge代码中)

被PlatformOnly修饰的方法如果在common包直接调用会产生警告 (需要安装插件)

![Warning](https://s2.loli.net/2022/04/02/Hp85GEtrSfOKueB.png)

如果我们尝试在Forge模块中调用被@PlatformOnly("fabric")修饰的方法, 游戏会抛出反射异常

![InvocationTargetException](https://s2.loli.net/2022/04/02/1DZfJ9xFXowS5mn.png)
