# Generic

## 概述

**泛型 —— 参数化类型**



在 Java 5 之前的集合类是不安全的，因为编译器允许向集合中插入错误的类型：

```java
class Apple {}
class Orange {}
ArrayList apples = new ArrayList();
apples.add(new Apple());
apples.add(new Apple());
apples.add(new Orange());
for (Object apple : apples) {
   	Apple a = (Apple) apple;		// 运行时会报错
}
```

这种类型错误只有在运行期才能检查出。现在使用泛型特性编写更加安全的代码，因为使用泛型可以在编译期检查出语法错误。A compiler error is much better than a class cast exception at runtime.

```java
ArrayList<Apple> apples = new ArrayList<Apple>();
apples.add(new Orange());		
Apple apple = (Apple)apples.get(0)		// 编译期报错
```



事实上，对于泛型的翻译有两种策略：

- **同构翻译（homogeneous translation）**：所有类型都共享一种实现
- **异构翻译（heterogeneous translation）**：每种类型组合都创建一份特化

C++ 和Java 的策略分别处于异构翻译和同构翻译的极端。而 C#/CLR 处于两者中间，为共享布局的引用类型同构翻译，为值类型异构翻译。

类型擦除的问题：

- **难以获取泛型参数的具体值**：程序员可能会想要知道一个 `List` 到底是 `List<String>` 还是 `List<Integer>`，想要拿到实际泛型参数相关的信息，而因为类型擦除，实际上并不能做到这一点。可以通过传递 Class 对象补偿这一点损失
- **不能布局特化**：当前基于擦除的实现要求泛型参数类型必须拥有公共运行时表示（common runtime representation），在 Java 里这意味着只能是一组引用类型，而不能为原始类型。这导致我们只能用 `List<Integer>`，而用不了针对 `int` 进行布局特化的 `List<int>`，底层存放的全是对 `Integer` 对象的引用，这造成了巨大的内存浪费，同时对现代 CPU 的缓存策略极端不友好，大量间接寻址产生大量 cache miss，造成性能下降。这也是当前 Java 泛型擦除最大的问题。
- **运行时错误**：泛型参数会被擦除，即`List<String>` 会被擦除为 `List`，同时可以通过一些手段（强制转换，raw type 等）将其他类型的值放入这个 List ，但在编译期并不会发现到这个类型错误，直到运行期才能发现。

类型擦除的优势：

- **兼容性**。它完全维护了二进制兼容性和源代码兼容性。不会因引入泛型后让社区生态产生巨大的分歧
- **类型系统的自由度**：虽然 Java 和 JVM 常常被绑定在一起，但它们是各自独立的，有着各自的规范。由于通过类型擦除实现泛型，像 Scala 这样的语言可以以与 Java 泛型高度协同的方案实现远远超出 Java 类型系统表达能力的类型系统，同时保持高度的互操作性。

模板（异构翻译）的问题：

- 模板的每个实例都要有着不同的代码，这意味着模板展开会导致**代码膨胀**。产生更大的硬盘和内存占用



## 语法

使用类型推导来简化泛型的编写：

```java
ArrayList<Apple> apples = new ArrayList<Apple>();
ArrayList<Apple> apples2 = new ArrayList<>();
ArrayList<Apple> apples3 = new ArrayList();	// 错误的，必须有<>，否则会被认为是原生类型的

var apple3 = new ArrayList<Apple>();
var apple4 = new ArrayList<>();		// 这会推断出ArrayList<Object>，不推荐使用	
```





