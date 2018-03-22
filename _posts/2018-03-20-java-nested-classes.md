---
layout: default
---

# Java 嵌套类

顾名思义，定义在一个类中的类，如下：

```java
class OuterClass {
    ...
    class NestedClass {
        ...
    }
}
```

> 分为两种：静态、非静态。
>
> 被声明为静态的嵌套类称作_静态嵌套类_，非静态嵌套类称作_内部类_。如下：

```java
class OuterClass {
    ...
    static class StaticNestedClass {
        ...
    }
    class InnerClass {
        ...
    }
}
```

内部类可以访问外部类的成员（即使被声明为私有），而静态嵌套类则不可以。嵌套类是外部类的成员，可以被声明为公有、私有、受保护和包私有四种访问级别。

## 那为什么用嵌套类呢？

*  **一种符合逻辑地组织类的方式：**当一个类只对另一个类有用时，把它内嵌到那个类中，使它们保持在一起，是一种合情合理的做法。
*  **提高封装性：**类A、B，B要访问A中成员，而A中成员为私有，通过把B嵌入A中实现B对A中私有成员的轻松访问，并且对于外部而言B自己也能被隐藏起来。
*  **带来更高可读性和可维护性的代码：**在顶层类中嵌入类，使内嵌类更加靠近与它被使用的地方。

## 静态嵌套类

与类成员一样，静态嵌套类与其外部类相关联，像静态方法一样，静态嵌套类不可引用其外部类的实例成员。实际上，静态嵌套类在行为上是顶层类，只不过为了打包便利，才把它内嵌到另一个顶层类中。

要创建静态嵌套类对象，如下：

```
OuterClass.StaticNestedClass nestedObject = new OuterClass.StaticNestedClass();
```

## 内部类

与实例成员一样，内部类与其外部类的实例相关联，可直接访问其外部类的对象的属性和方法，但是其内不可定义静态成员。

为了实例化内部类，必须先实例化外部类，然后用外部类对象创建内部类对象，如下：

```
OuterClass.InnerClass innerObject = outerObject.new InnerClass();
```

## 嵌套中同名遮蔽问题

如下：

```java
public class ShadowTest {

    public int x = 0;

    class FirstLevel {

        public int x = 1;

        void methodInFirstLevel(int x) {
            System.out.println("x = " + x);
            System.out.println("this.x = " + this.x);
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }

    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}
```

输出：

```
x = 23
this.x = 1
ShadowTest.this.x = 0
```

### 参考链接

[https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html).
