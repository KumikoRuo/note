# Effective Java

[TOC]

## 创建和销毁对象

### 静态工厂方法代替构造器

静态工厂方法（不同于设计模式中的工厂方法）

~~~java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
~~~



相对于构造器的优势：

1. 具有方法名，可以明确地反映出创建意图，代码可读性更好。而构造器的参数列表却无法做到这一点

   > Dart 等编程语言已支持命名构造函数的语法，Java 只能通过编程技巧来弥补语言上的设计缺陷。

2. 可以先将实例构建好，并放入缓存中。每次调用静态工厂方法时，直接返回缓存中的实例即可。这样就可以避免创建实例的成本。

3. 可以返回任何派生类

   这种优势的一种应用就是 

   ~~~java
   public class Foo {
       private class Bar extends Foo {
           
       }
       
       public static Foo() {
           return new Bar();
       }
   }
   ~~~

4. 返回实例的类型可以随着参数而变化

   

