# Java 8 接口 默认方法

示例代码：

```java
        List<String> stringList = new ArrayList<>();
        
        stringList.stream().forEach(s -> System.out.println(s));
        
        stringList.parallelStream().forEach(s -> System.out.println(s));
```

在这里有一个明显的问题就是：在Java 8之前，List<T>并没有**stream**或**parallelStream**方法，它实现的Collection<T>接口也没有，这样会导致有很多的替代集合框架都用Collection API实现了接口。但给接口加入一个新方法，意味着所有的实体类都必须为其提供一个实现。语言设计者是没法控制Collections所有现有的实现的，现在需要做到你如何改变已发布的接口而不破坏已有的实现：

1. 最简单的解决方案就是让Java 8的设计者把stream方法加入Collection接口，并加入ArrayList类的实现。但是上面也说到，语言设计者是没法控制Collections所有现有的实现的，所以这种方案只能用于自己的接口。
2. 第二种方案就是缺失的方法主体随接口提供了（因此就有了**默认实现**），而不是由实现类提供。

- **默认实现**

例如，在Java 8里，你现在可以直接对List调用sort方法。它是用Java 8 List接口中如下所 示的默认方法实现的，它会调用Collections.sort静态方法：

```java
default void sort(Comparator<? super E> c) {
    Collections.sort(this, c);
}
```

在Java 8中List的任何实体类都不需要显式实现sort，而在以前的Java版本中必须提供sort的实现，否则这些实体类在重新编译时都会失败。