# Dubbo

参考链接：https://www.jianshu.com/p/4df584e9b493

## Dubbo SPI

Java 原生的SPI 是 `ServiceLoader` 会加载META-INF/services 接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类。

Dubbo 的SPI从 JDK 标准的 SPI (Service Provider Interface) 扩展点发现机制加强而来，Dubbo 改进了 JDK 标准的 SPI 的以下问题：

- JDK 标准的 SPI 会一次性实例化扩展点所有实现，如果有扩展实现初始化很耗时，但如果没用上也加载，会很浪费资源。
- 如果扩展点加载失败，连扩展点的名称都拿不到了。比如：JDK 标准的 ScriptEngine，通过 getName() 获取脚本类型的名称，但如果 RubyScriptEngine 因为所依赖的 jruby.jar 不存在，导致 RubyScriptEngine 类加载失败，这个失败原因被吃掉了，和 ruby 对应不起来，当用户执行 ruby 脚本时，会报不支持 ruby，而不是真正失败的原因。
- 增加了对扩展点 IoC 和 AOP 的支持，一个扩展点可以直接 setter 注入其它扩展点。

1、首先我们先定义一个接口，取名为`Robot`，这里需要加一个`com.alibaba.dubbo.common.extension.SPI`的注解

```java
/**
 * 演示jdk spi的使用
 *
 * @author hui.wang
 * @since 31 January 2019
 */
@SPI
public interface Robot {

    /**
     * 测试接口
     */
    void sayHello();
}
```

2、接着我们定义两个实现类，分别为`Bumblebee`和`OptimusPrime`，和上面代码一致，这里不再列出代码

3、接着在`META-INF/dubbo`文件夹下创建一个文件，名称为 `Robot` 的全限定名`com.hui.wang.dubbo.learn.spi.Robot`，内容如下：

```txt
bumblebee = com.hui.wang.dubbo.learn.spi.Bumblebee
optimusPrime = com.hui.wang.dubbo.learn.spi.OptimusPrime
```

现在开始测试，没有写UT测试，写了main方法

```java
public class DubboSPITest {

    public static void main(String[] args) {
        ExtensionLoader<Robot> extensionLoader = ExtensionLoader.getExtensionLoader(Robot.class);

        Robot optimusPrime = extensionLoader.getExtension("optimusPrime");
        optimusPrime.sayHello();

        /**
         * dubbo SPI支持默认设计
         * 配置 {@link com.alibaba.dubbo.common.extension.SPI}的value属性即可
         */
        Robot bumblebee = extensionLoader.getDefaultExtension();
        bumblebee.sayHello();
    }
}
```

打印结果为

```txt
Hello, I am Bumblebee.
Hello, I am Optimus Prime.
```

## Dubbo SPI AOP

保持上面的代码不变。

编写一个`Robot`实现类，取名为`RobotWrapper`，具体代码如下：

```java
/**
 * <b>关于 dubbo spi aop 的使用</b>
 *
 * @author hui.wang09
 * @since 22 February 2019
 */
public class RobotWrapper implements Robot{

    private Robot robot;

    public RobotWrapper(Robot robot) {
        this.robot = robot;
    }

    @Override
    public void sayHello() {
        System.out.println("========================");
        System.out.println("before");
        System.out.println("========================");
        robot.sayHello();
        System.out.println("========================");
        System.out.println("after");
        System.out.println("========================");
    }
}
```

接着编辑在`META-INF/dubbo`文件夹下，名为 Robot 的全限定名com.hui.wang.dubbo.learn.spi.Robot文件，内容如下：

```txt
bumblebee=com.hui.wang.dubbo.learn.jdkspi.Bumblebee
optimusPrime=com.hui.wang.dubbo.learn.jdkspi.OptimusPrime
robotWrapper=com.hui.wang.dubbo.learn.dubbospi.RobotWrapper
```

这里新增了`robotWrapper=com.hui.wang.dubbo.learn.dubbospi.RobotWrapper`配置

编写测试类，代码如下：

```java
    /**
     * <b>演示dubbo spi aop</b>
     * Wrapper class:
     * robotWrapper=com.hui.wang.dubbo.learn.dubbospi.RobotWrapper
     *
     * optimusPrime:
     * optimusPrime=com.hui.wang.dubbo.learn.jdkspi.OptimusPrime
     */
    @Test
    public void testRobotWrapper() {
        ExtensionLoader<Robot> extensionLoader = ExtensionLoader.getExtensionLoader(Robot.class);

        Robot optimusPrime = extensionLoader.getExtension("optimusPrime");
        optimusPrime.sayHello();
        System.out.println(optimusPrime.getClass());
    }
```

打印结果如下：

```txt
========================
before
========================
Hello, I am Optimus Prime.
========================
after
========================
class com.hui.wang.dubbo.learn.dubbospi.RobotWrapper
```

从打印结果可以看出dubbo spi aop的使用和执行过程，dubbo spi 在加载扩展点时，如果加载到的扩展点有拷贝构造函数，则判定为扩展点 Wrapper 类。Wrapper 类有些类似 AOP，即 Wrapper 代理了扩展点。

## Adaptive

上述的SPI存在一个问题，即扩展点对应的实现类不能在程序运行时动态指定，就是`extensionLoader.getExtension`方法写死了扩展点对应的实现类，不能在程序运行期间根据运行时参数进行动态改变。

大致解决方案是设置一个代理类，代理类根据程序运行期间URL的参数，动态加载响应的实现类并代理。

dubbo 自适应机制的使用是通过注解的方式，注解为`com.alibaba.dubbo.common.extension.Adaptive`。直接上代码示例：

首先定义一个接口，名称为`AdaptiveExt`，使用`SPI`和`Adaptive`修饰：

```java
@SPI
public interface AdaptiveExt {

    @Adaptive
    String echo(String msg, URL url);
}
```

创建三个实现类，分别为`DubboAdaptiveExtImpl`、`SpringAdaptiveExtImpl`和`SpringBootAdaptiveExtImpl`

```java
public class DubboAdaptiveExtImpl implements AdaptiveExt {

    @Override
    public String echo(String msg, URL url) {
        return "dubbo";
    }
}
```

```java
public class SpringAdaptiveExtImpl implements AdaptiveExt {

    @Override
    public String echo(String msg, URL url) {
        return "spring";
    }
}
```

```java
public class SpringBootAdaptiveExtImpl implements AdaptiveExt{

    @Override
    public String echo(String msg, URL url) {
        return "spring boot";
    }
}
```

接着在META-INF/dubbo文件夹下创建一个文件，名称为`Robot`的全限定名`com.hui.wang.dubbo.learn.dubbo.adaptive.AdaptiveExt`，内容如下：

```undefined
dubbo=com.hui.wang.dubbo.learn.dubbo.adaptive.DubboAdaptiveExtImpl
spring=com.hui.wang.dubbo.learn.dubbo.adaptive.SpringAdaptiveExtImpl
springboot=com.hui.wang.dubbo.learn.dubbo.adaptive.SpringBootAdaptiveExtImpl
```

到这里，示例的基本代码搭建完成，现在开始进入使用和测试

在URL里面指定参数，代码为：

```java
@Test
public void test1() {
    ExtensionLoader<AdaptiveExt> extExtensionLoader = ExtensionLoader.getExtensionLoader(AdaptiveExt.class);
	// adaptiveExt是一个代理类
    AdaptiveExt adaptiveExt = extExtensionLoader.getAdaptiveExtension();
    // adaptive.ext是默认参数（取类名加.分割） adaptive.ext=spring 用于指定配置文件中的实现类
    URL url = URL.valueOf("test://localhost/test?adaptive.ext=spring");
    System.out.println(adaptiveExt.echo("d", url));
}
```

打印结果为：

```txt
spring
```

配置`Adaptive`注解的`value`值，代码如下：

```java
@SPI
public interface AdaptiveExt {
	// value 用于自定义参数 myAdaptiveName相当于adaptive.ext
    @Adaptive(value = {"myAdaptiveName"})
    String echo(String msg, URL url);
}
```

测试代码：

```java
@Test
public void test4() {
    ExtensionLoader<AdaptiveExt> extExtensionLoader = ExtensionLoader.getExtensionLoader(AdaptiveExt.class);
    AdaptiveExt adaptiveExt = extExtensionLoader.getAdaptiveExtension();
    URL url = URL.valueOf("test://localhost/test?myAdaptiveName=spring");
    System.out.println(adaptiveExt.echo("d", url));
}
```

打印结果为:

```txt
spring
```

在方法上打上`@Adaptive`注解，注解中的`value`与链接中的参数的`key`一致，链接中的`key`对应的`value`就是spi中的`name`,获取相应的实现类。

`@Adaptive`注解使用在实现类上，代码如下：

```java
@Adaptive
public class DubboAdaptiveExtImpl implements AdaptiveExt {

    @Override
    public String echo(String msg, URL url) {
        return "dubbo";
    }
}
```

测试代码：

```java
@Test
public void test4() {
    ExtensionLoader<AdaptiveExt> extExtensionLoader = ExtensionLoader.getExtensionLoader(AdaptiveExt.class);
    AdaptiveExt adaptiveExt = extExtensionLoader.getAdaptiveExtension();
    URL url = URL.valueOf("test://localhost/test?myAdaptiveName=spring");
    System.out.println(adaptiveExt.echo("d", url));
}
```

打印结果为:

```txt
dubbo
```

这里可以看到实现类上有`@Adaptive`注解后，默认调用该类进行调用。

到这里基本上就是dubbo 自适应机制的使用，即`@Adaptive`注解的使用，在下一篇我们将介绍`@Adaptive`注解背后的源码实现

## Activate

Activate注解标识一个扩展是否被激活和使用，可以放在定义的类上和方法上，dubbo用它在SPI扩张类定义上，标识这个扩展实现激活的条件和时机，`Activate`就是定义扩展点实现类激活的条件，当程序运行的参数满足这些条件时，自动激活这些扩展点实现类。调用方法是`ExtensionLoader#getActivateExtension`。常见的用途如：filter链，接下来我们看看`Activate`注解的使用。

首先定义一个接口，取名为`ActivateExt`，代码如下：

```dart
/**
 * {@link com.alibaba.dubbo.common.extension.Activate} 使用
 *
 * @author hui.wang09
 * @since 31 January 2019
 */
@SPI
public interface ActivateExt {

    String echo(String msg);
}
```

定义五个实现类，如下：

```java
/**
 * {@link ActivateExt} impl
 *
 * @author hui.wang09
 * @since 31 January 2019
 */
@Activate(group = {"default_group"})
public class DefaultActivateExtImpl implements ActivateExt{

    @Override
    public String echo(String msg) {
        return msg;
    }
}
```

```java
/**
 * {@link ActivateExt} impl
 *
 * @author hui.wang09
 * @since 31 January 2019
 */
@Activate(group = {"group1", "default_group"})
public class GroupActivateExtImpl implements ActivateExt{

    @Override
    public String echo(String msg) {
        return msg;
    }
}
```

```java
/**
 * {@link ActivateExt} impl
 *
 * @author hui.wang09
 * @since 31 January 2019
 */
@Activate(order = 2, group = {"order"})
public class OrderActivateExtImpl implements ActivateExt{

    @Override
    public String echo(String msg) {
        return msg;
    }
}
```

```java
/**
 * {@link ActivateExt} impl
 *
 * @author hui.wang09
 * @since 31 January 2019
 */
@Activate(order = 1, group = {"order"})
public class OrderActiveExtImplMore implements ActivateExt{

    @Override
    public String echo(String msg) {
        return msg;
    }
}
```

```java
/**
 * {@link ActivateExt} impl
 *
 * @author hui.wang09
 * @since 31 January 2019
 */
@Activate(value = {"myKey"}, group = {"value"})
public class ValueActivateExtImpl implements ActivateExt{

    @Override
    public String echo(String msg) {
        return msg;
    }
}
```

接着在`META-INF/dubbo`文件夹下创建一个文件，名称为`ActivateExt`的全限定名`com.hui.wang.dubbo.learn.dubbo.activate.ActivateExt`，内容如下：

```csharp
group=com.hui.wang.dubbo.learn.dubbo.activate.GroupActivateExtImpl
order1=com.hui.wang.dubbo.learn.dubbo.activate.OrderActiveExtImplMore
order2=com.hui.wang.dubbo.learn.dubbo.activate.OrderActivateExtImpl
value=com.hui.wang.dubbo.learn.dubbo.activate.ValueActivateExtImpl
com.hui.wang.dubbo.learn.dubbo.activate.DefaultActivateExtImpl
```

到这里，前期的准备完成，下面开始我们的测试使用阶段

**使用一:**  关于`ActivateExt`注解的`group`的使用，代码如下：

```java
@Test
public void testGroup() {
    ExtensionLoader<ActivateExt> extExtensionLoader = ExtensionLoader.getExtensionLoader(ActivateExt.class);
    URL url = URL.valueOf("test://localhost/test");

    List<ActivateExt> list = extExtensionLoader.getActivateExtension(url, new String[]{}, "default_group");
    list.forEach(o -> System.out.println(o.echo(o.getClass().getName())));
}
```

打印结果为：

```css
com.hui.wang.dubbo.learn.dubbo.activate.GroupActivateExtImpl
com.hui.wang.dubbo.learn.dubbo.activate.DefaultActivateExtImpl
```

这里可以看到命中了`Activate`注解`group`为`default_group`的扩展点实例。

**使用二：** 关于`Activate`注解的`value`的使用，代码如下：

```java
@Test
public void testValue() {
    ExtensionLoader<ActivateExt> extExtensionLoader = ExtensionLoader.getExtensionLoader(ActivateExt.class);
    URL url = URL.valueOf("test://localhost/test");

    /**
         * 对应 @Activate(value = {"myKey"})
         */
    url = url.addParameter("myKey", "test");

    List<ActivateExt> list = extExtensionLoader.getActivateExtension(url, new String[]{}, "value");
    list.forEach(o -> System.out.println(o.echo(o.getClass().getName())));
}
```

打印结果为：

```css
com.hui.wang.dubbo.learn.dubbo.activate.ValueActivateExtImpl
```

可以看到命中了`Activate`注解`group`为`value`，且`value`为`myKey`的扩展点实例。

**使用三：** 关于`Activate`注解`order`的使用，代码如下：

```java
@Test
public void test3() {
    ExtensionLoader<ActivateExt> extExtensionLoader = ExtensionLoader.getExtensionLoader(ActivateExt.class);
    URL url = URL.valueOf("test://localhost/test");

    List<ActivateExt> list = extExtensionLoader.getActivateExtension(url, new String[]{}, "order");
    list.forEach(o -> System.out.println(o.echo(o.getClass().getName())));
}
```

打印结果为:

```css
com.hui.wang.dubbo.learn.dubbo.activate.OrderActiveExtImplMore
com.hui.wang.dubbo.learn.dubbo.activate.OrderActivateExtImpl
```

可以看到命中了`Activate`注解`group`为`order`的扩展点实例，且`order`低的优先级高