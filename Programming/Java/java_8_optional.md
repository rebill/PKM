## Java 8 Optional 详解

## 概述

工作中经常会有这样的一个经历：

调用一个方法获得的返回值可能为空，需要进行null判断，然后再做一些相应的业务处理，或者直接抛出NullPointerException。

为了减少这样的null值判断，java官方借鉴Google Guava类库的Optional类，在java8 中引入了一个同样名字的Optional类，官方javadoc描述如下：

>Icon
A container object which may or may not contain a non-null value. If a value is present, isPresent() will return true and get() will return the value.

Optional是 java.util  的一个子类，通过以下方式引入
```java
import java.util.Optional;
```

## Optional类简介

### of

*为非Null值创建一个Optional*

of方法通过工厂方法创建Optional实例，需要注意的是传入的参数不能为null，否则抛出NullPointerException。

```java
// 给与一个非空值
Optional<String> username = Optional.of("Unknown");
// 传入参数为null，抛出NullPointerException.
Optional<String> nullValue = Optional.of(null);
```

### ofNullable

*为指定的值创建一个Optional，如果指定的值为null，则返回一个空的Optional。*

可为空的Optional

```java
// 下面创建了一个不包含任何值的Optional实例
// 输出Optional.empty
Optional empty = Optional.ofNullable(null);
```

### isPresent

*如果值存在返回true,否则返回false*
类似下面的代码:
```java
// isPresent方法用来检查Optional实例中是否包含值
if (username.isPresent()) {
    //在Optional实例内调用get()返回已存在的值
    System.out.println(username.get());      //输出：Unknown
}
```

### get

*如果Optional有值则将其返回，否则抛出NoSuchElementException。*

```java
// 执行下面的代码抛出NoSuchElementException
try {
     // 在空的Optional实例上调用get()
     System.out.println(empty.get());
 } catch (NoSuchElementException ex) {
     System.out.println(ex.getMessage());         // 输出：No value present
}
```

## 小试牛刀

看语法介绍都是很枯燥的，我们来一个生动的例子。

先来看看在没有Optional之前我们的代码是怎么写的。

```java
public static String getName(User u) {
    if (u == null)
        return "Unknown";
    return u.name;
}
```

有了Optional之后，你可能会把代码改成下面这样：

```java
public static String getName(User u) {
    Optional<User> user = Optional.ofNullable(u);
    if (!user.isPresent())
        return "Unknown";
    return user.get().name;
}
```

看上去仍然是在原地踏步，实质上是没有任何分别。

## 更优雅Optional

Let me Show you the code:

```java
public static String getName(User u) {
    return Optional.ofNullable(u)
                    .map(user->user.name)
                    .orElse("Unknown");
}
```

## 更复杂的例子

```java
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    if (comp != null) {
        CompResult result = comp.getResult();
        if (result != null) {
            User champion = result.getChampion();
            if (champion != null) {
                return champion.getName();
            }
        }
    }
    throw new IllegalArgumentException("The value of param comp isn't available.");
}
```

由于种种原因（比如：比赛还没有产生冠军、方法的非正常调用、某个方法的实现里埋藏的大礼包等等），我们并不能开心的一路comp.getResult().getChampion().getName()到底。

让我们看看经过Optional加持过后，这些代码会变成什么样子。

```java
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    return Optional.ofNullable(comp)
            .map(c->c.getResult())
            .map(r->r.getChampion())
            .map(u->u.getName())
            .orElseThrow(()->new IllegalArgumentException("The value of param comp isn't available."));
}
```

Optional给了我们一个真正优雅的Java风格的方法来解决null安全问题。

## 更多神奇魅力

为空则不打印可以这么写：

```java
string.ifPresent(System.out::println);
```

检验参数的合法性

```java
public void setName(String name) throws IllegalArgumentException {
    this.name = Optional.ofNullable(name).filter(User::isNameValid)
                        .orElseThrow(()->new IllegalArgumentException("Invalid username."));
}
```

## Optional类深入学习

### orElse

*如果有值则将其返回，否则返回指定的其它值。*
如果Optional实例有值则将其返回，否则返回orElse方法传入的参数。示例如下：

```java
@Test
public void whenOrElseWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElse("Unknown");
    assertEquals("Unknown", name);
}
```

### orElseGet

orElseGet与orElse方法类似，区别在于得到的默认值。orElse方法将传入的字符串作为默认值，orElseGet方法可以接受Supplier接口的实现用来生成默认值。示例如下：

```java
@Test
public void whenOrElseGetWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElseGet(() -> "Unknown");
    assertEquals("Unknown", name);
}
```

### map

*如果有值，则对其执行调用mapping函数得到返回值。如果返回值不为null，则创建包含mapping返回值的Optional作为map方法返回值，否则返回空Optional。*

map方法用来对Optional实例的值执行一系列操作。通过一组实现了Function接口的lambda表达式传入操作。map方法示例如下：

```java
@Test
public void givenOptional_whenMapWorks_thenCorrect() {
    List<String> companyNames = Arrays.asList(
      "paypal", "oracle", "", "microsoft", "", "apple");
    Optional<List<String>> listOptional = Optional.of(companyNames);
  
    int size = listOptional
      .map(List::size)
      .orElse(0);
    assertEquals(6, size);
}
```

### 更多方法

* **ifPresent**  如果Optional实例有值则为其调用consumer ,否则不做处理。
* **orElseThrow**  如果有值则将其返回，否则抛出supplier接口创建的异常。
* **flatMap**  如果有值，为其执行mapping函数返回Optional类型返回值，否则返回空Optional。flatMap与map（Funtion）方法类似，区别在于flatMap中的mapper返回值必须是Optional。调用结束时，flatMap不会对结果用Optional封装。
* **filter**  filter个方法通过传入限定条件对Optional实例的值进行过滤。如果有值并且满足断言条件返回包含该值的Optional，否则返回空Optional。

不再一一介绍。

JDK 9 又新增了几个API

* the **or()** method for providing a supplier that creates an alternative Optional
* the **ifPresentOrElse()** method that allows executing an action if the Optional is present or another action if not
* **stream()** method for converting an Optional to a Stream

## 总结

使用 Optional 时尽量不直接调用 Optional.**get()** 方法，Optional.**isPresent()** 更应该被视为一个私有方法，应依赖于其他像 Optional.**orElse()**、Optional.**orElseGet()**、Optional.**map()** 等这样的方法。

最好的理解 Java 8 Optional 的方法莫过于看它的源代码 [java.util.Optional](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/Optional.java)，阅读了源代码才能真真正正的让你解释起来最有底气，Optional 的方法中基本都是内部调用  isPresent() 判断，真时处理值, 假时什么也不做。

## 参考资料

* [Java8 Optional 详解](https://juejin.im/entry/5ae6b732f265da0b7f44548f)
* [Java8 如何正确使用 Optional](http://www.importnew.com/26066.html)
* [Guide To Java 8 Optional](https://www.baeldung.com/java-optional)
* [使用 Java8 Optional 的正确姿势](http://www.importnew.com/22060.html)
