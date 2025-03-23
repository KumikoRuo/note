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

相对于 public 构造器的优势：

1. 具有方法名，可以明确地反映出创建意图，可读性更好。
2. 可以先将实例构建好，并放入缓存中。每次调用静态工厂方法时，直接返回缓存中的实例即可，避免创建实例的成本。
3. 可以返回任何派生实例

