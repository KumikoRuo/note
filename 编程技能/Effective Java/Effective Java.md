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

2. 可以返回任何派生类实例。下面就是该优势的一种应用：隐藏实现类

   ~~~java
   public class Foo {
       private class Bar extends Foo {
           
       }
       
       public static Foo() {
           return new Bar();
       }
   }
   ~~~

   > 在 Java 8 之前，接口不支持默认方法。因此按照之前的惯例，接口 Type 通常会有一个名为 Types 的、不可实例化的伴生类，通过静态工厂方法来提供集合工具。例如 Collection 接口与 Collections 工具类。

3. **封装复杂的对象创建逻辑**，具体来说有：

   1. 可以先将实例构建好，并放入缓存中。每次调用静态工厂方法时，直接返回缓存中的实例即可。这样就可以避免创建实例的成本。
   2. 所返回的派生类可以随着参数而变化，例如 EnumSet 的静态工厂方法在 OpenJDK 中的实现。
   3. 单例与多例模式
   4. 可以在运行期通过反射技术来动态加载类



缺点就是很难发现静态工厂方法，但是可以通过惯用名称来弥补这一缺点

- from：类型转换方法

  ~~~java
  Date date = Date.from(18502105L);
  ~~~

- valueOf：类似 from，类型转换方法

- getType/newType/type：类型转换方法，其中 type 为工厂方法所返回的类型

  ~~~java
  Fi1eStore fs = Fi1es.getFi1eStore(path);
  BufferedReader br = Fi1es.newufferedReader(path);
  List<Comp1aint> litany = Col1ections.list(1egacyLitany);
  ~~~

- of：聚合方法

  ~~~java
  Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
  ~~~

- instance/getInstance

  ~~~java
  Stackwalker luke -stackwa1ker.getTnstance(options);
  ~~~

- create/newInstance：与 instance 类似，但是确保每次都返回一个新的实例



### 遇到多个构造器参数时要考虑使用 Builder

在 Java 中，静态工厂和构造器都无法很好地应对有着大量可选参数的场景。

> JS、Dart 等编程语言提供命名参数的语法来应对这种场景。



一种解决方案是使用 telescoping constructor （伸缩式构造器）模式：

```java
public class NutritionFacts {
    private final int servingSize; // (mL) required
    private final int servings; // (per container) required
    private final int calories; // (per serving) optional
    private final int fat; // (g/serving) optional
    private final int sodium; // (mg/serving) optional
    private final int carbohydrate; // (g/serving) optional
    
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }
    
    public NutritionFacts(int servingSize, int servings,int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
    
    public NutritionFacts(int servingSize, int servings,int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    
    public NutritionFacts(int servingSize, int servings,int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

该方案的缺点：

- 若我们只想设置 fat 参数，那么还得设置一些不必要的参数：

  ```java
  NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
  ```

- 可读性、易用性很差，这就导致在调用时很容易犯错，例如混淆参数顺序。



我们还有 JavaBeans 方案：

```java
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize = -1; // Required; no default value
    private int servings = -1; // Required; no default value
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;
    
    public NutritionFacts(int servingSize, int servings) {
        // ...
    }
    // Setters
}
```

```java
NutritionFacts cocaCola = new NutritionFacts(-1, -1);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

这种方案也有缺点：

1. 不同于构造器这种原子化 initialization ，该方案通过调用多个 setter 方法来分步构建对象。在此期间，JavaBeans 或违反参数间的依赖和约束关系（不一致性），若此时其他代码（尤其在并发场景下）误用该未初始化完成的对象，将引发难以追踪的 Bug。
2. setter 无法初始化 final 成员变量，而且不可变对象要求禁止暴露修改状态的 setter ，最终 JavaBeans 无法设计为不可变的（immutable）。



建议使用 Builder 方案，它完美地解决了上述所有痛点

```java
public class ResourcePoolConfig {
    private String name;
    private int maxTotal;
    private int maxIdle;
    private int minIdle;

    private ResourcePoolConfig(Builder builder) {
        this.name = builder.name;
        this.maxTotal = builder.maxTotal;
        this.maxIdle = builder.maxIdle;
        this.minIdle = builder.minIdle;
    }
    //...省略getter方法...
    
    public static class Builder {
        // 内部也要暂时维护一份数据
        private String name;
        private int maxTotal = 8;
        private int maxIdle = 8;
        private int minIdle = 0;

        public ResourcePoolConfig build() {
            // 校验逻辑放到这里来做，包括必填项校验、依赖关系校验、约束条件校验等
            if (StringUtils.isBlank(name)) {
                throw new IllegalArgumentException("...");
            }
            if (maxIdle > maxTotal) {
                throw new IllegalArgumentException("...");
            }
            if (minIdle > maxTotal || minIdle > maxIdle) {
                throw new IllegalArgumentException("...");
            }
            
            return new ResourcePoolConfig(this);
        }

        public Builder setName(String name) {
            this.name = name;
            return this;
        }

        public Builder setMaxTotal(int maxTotal) {
            this.maxTotal = maxTotal;
            return this;
        }

        public Builder setMaxIdle(int maxIdle) {
            this.maxIdle = maxIdle;
            return this;
        }

        public Builder setMinIdle(int minIdle) {
            this.minIdle = minIdle;
            return this;
        }
    }
}
```

而且 Builder 模式也可以支持类层次结构：

```java
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
    final Set<Topping> toppings;
    
    
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
        
        abstract Pizza build();
        
        // Subclasses must override this method to return "this"
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}
```

```java
public class Calzone extends Pizza {
    private final boolean sauceInside;
    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default
        
        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }
        
        // 协变返回类型
        @Override public Calzone build() {
            return new Calzone(this);
        }
        
        @Override protected Builder self() {
            return this;
        }
    }
    
    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

### 用私有构造器或者枚举类型强化 Singleton 属性

单例的两种经典实现：

1. 通过公有变量访问单例对象

   ```java
   public class Elvis {
       public static final Elvis INSTANCE = new Elvis();
       private Elvis() {}
   }
   ```

2. 通过静态工厂访问单例对象

   ```java
   public class Elvis {
       private static final Elvis INSTANCE = new Elvis();
       private Elvis() {}
       
       public static getInstance() {
           return this.INSTANCE;
       }
   }
   ```

   该实现的优势就是提供了灵活性：

   1. `Elvis::getInstace()` 可以作为 `Supplier<Elvis>` 用在函数式编程中
   2. 在保持公有 API 不变的前提下，可以修改其内部实现，例如支持惰性加载、保证在线程仅有一个单例实例等等

   

序列化单例对象的标准做法：

1. 添加 implements Serializable 空接口

2. 将所有实例属性声明为 transient

3. 实现 readResolve 方法（反序列化时会回调该方法，如果有的话），否则每次反序列化时都会创建一个新的对象，这违反了单例的语义。

   ```java
   public class Singleton implements Serializable { 
       private Object readResolve() {  
           return this.INSTANCE; 
       }  
   }
   ```

因为枚举序列化是由 JVM 保证的，所以我们可以通过枚举类型来实现单例，这样就无需考虑序列化问题了

```java
enum EnumSingleton {
    INSTANCE
}
```



### 通过私有构造器强化不可实例化的能力

```java
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        // 避免反射调用构造器
        throw new AssertionError();
    }
}
```

由于所有的构造器都必须（隐式）调用基类的构造器，这种方式使得其不能被派生。

### 先考虑依赖注人来引用资源

不建议使用静态工具类或 Singleton 类来设计持有底层资源的类。

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;
    
    private SpellChecker(...) {
        this.dictionary = ...
    }
    public static INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

这样做的缺点是：

- 难以 Mock 掉所依赖的资源，可测试性差
- 所依赖的资源在类加载时就被构建，难以在不同场景来引用不同的资源，可扩展性差



建议使用依赖注入技术，通过构造器来参数化不同的资源：

```java
public class SpellChecker {
    private final Lexicon dictionary;
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

### 避免创建不必要的对象

最好能复用实例，而不是在每次需要时都创建一个新的，下面来看几个例子：

- 字符串：
  ```java
  String s = new String("bikini");
  ```

  上述代码每次 new 都会创建一个新的 bikini 字符串实例，建议复用字符串常量池中的对象：

  ```java
  String s = "bikini";
  ```

- Pattern：

  ```java
  static boolean isRomanNumeral(String s) {
      return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$";
  }
  ```
  
   matches 方法每次都会在内部为正则表达式创建 Pattern 实例，而其创建成本很高，因为需要将正则表达式编译为一个 FSM。最好将 Pattern 实例缓存起来并复用
  
  ```java
  public class RomanNumerals {
      private static final Pattern ROMAN = Pattern.compile(
          "^(?=.)M*(C[MD]|D?C{0,3})"
          + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
      static boolean isRomanNumeral(String s) {
          return ROMAN.matcher(s).matches();
      }
  }
  ```
  
- Map 的 keySet 方法，每次调用都返回同一个 keys 的 Set 视图实例。

- 自动装箱：尽量避免隐式自动装箱与拆箱。

  ```java
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++) {
      sum += i;
  }
  return sum;
  ```

  程序在 for 循环内构造了约 2^31 个多余的 Long 实例，严重损耗性能。



不建议实现轻量级对象池来复用小对象，因为：

1. 实现对象池使得代码实现复杂度增加
2. 对于小对象来说，现代 JVM 都能够很好地应对它们的创建与销毁。其 GC 性能能够很轻松超过对象池的



### 消除过期的对象引用

下面我们来看一个例子：

```java
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    /**
    * Ensure space for at least one more element, roughly
    * doubling the capacity each time the array needs to grow.
    */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

当栈先增长再收缩后，即使元素已经 pop 出去，但内部数组仍持有该元素的引用，导致 GC 无法回收，从而内存泄漏。解决方法很简单，就是清空对象引用（赋值 null）：

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

清空对象引用应是一种例外而不是规范，当我们自己手动管理内存时，就要警惕这种内存泄漏问题了。



此外，对于长期驻留内存的静态 Map 也有类似的内存泄漏和资源堆积耗尽的问题，建议采用以下防护措施：

1. 设置大小上限

   ```java
   if (map.size() > UPPER_SIZE) {
       throw new Exception("deny to put element to map");
   }
   map.put(key, value);
   ```

2. 老化机制（后台线程定时清理）

3. 使用 WeakHashMap

### 避免使用 finalizer 方法和 cleaner 方法

避免使用 finalizer，原因在于：

1. Finalizer 的清理时间是不可预测的，甚至都不会执行
2. 各个 JVM 对于 Finalizer 的实现有差异，导致程序使用 Finalizer 后，其可移植性较差
3. 在 Java 中，内存资源的清理是由 JVM 负责的，而网络、文件等系统资源的清理通常是在 try-with-resources 以及 try-finally 中，因此使用 Finalizer 是没必要的



但是在某些场景下还是可以使用 Finalizer 的：

1. 安全兜底，以防资源拥有者忘记释放资源（例如调用 close 方法）。在 Java 库中， FileInputStream、ThreadPoolExecutor 和 java.sql.Connection 等都使用到了 Finalizer 
2. 

## 通用方法

### equals



### hashCode



### toString



### clone



### Comparable



## 类和接口

## 泛型

## 枚举和注解

## Lambda 与 Stream

## 方法

## 通用编程

## 异常

Java 异常分为三类：受检异常（checked exception）、运行时异常（runtime exception）、错误（error）



禁止将异常应用到正常的程序控制流程中，下面给出一个反例：

```java
try {
    int i = 0;
    while (true) {
        range[i++].climb();
    }
} catch (ArrayIndexOutOfBoundException e) {
    
}

// do other thing
```

上述代码为了避免 JVM 的数组越界检查而造成的性能消耗，直接通过捕获越界异常来终止数组的遍历。这样做是错误的，因为：

1. 模糊了代码的意图
2. JVM 会优化数组越界检查，反而 try-catch 块会阻碍 JVM 要执行的特定优化
3. 使用异常来控制正常的流程会掩盖程序中真正的 bug

> 一个设计良好的 API 不能迫使其客户端在正常的控制流程中使用异常。



只有在特定的、不可预知的条件下才可调用的 state-dependent 方法，通常有一个与之关联的 state-testing 方法，来表明是否可以调用该 state-dependent 方法，例如 Iterator 迭代器中的 next() 以及 hashNext() 方法

```java
for (Iterator<Foo> itr = collection.iterator(); i.hasNext(); ) {
    Foo foo = i.next();
    ...
}
```

如果 Iterator 缺少 hasNext 方法，客户端将被迫这样做：

```java
try {
    Iterator<Foo> itr = collection.iterator();
    while (true) {
        Foo foo = i.next;
        ...
    }
} catch (NoSuchElementException e) {
    
}
```

值得说明的是，可以让 state-dependent 在不能执行时返回特殊值（例如， null、empty Optional 、-1 等）来代替 state-testing 方法。

## 并发

## 序列化



## 拾遗

### SPF

Service Provider Framework 由以下四部分组成：

- Service Interface
- Provider Registration API：注册 Service Provider Interface（如果有的话） 或 Service Interface 实例
- Service Access API：获取 Service Interface 实例 
- Service Provider Interface（可选的）：创建 Service Interface 实例的工厂类接口

API 与 SPI 的区别就在于接口由实现方定义，还是由调用方定义。



下面给出一个例子：

~~~java
// Service Interface
public interface ServiceInterface {
    void serve();
}

public class ServiceInterfaceImpl implements ServiceInterface {
    @Override
    public void serve() {
        System.out.println("This is ServiceInterfaceImpl");
    }
}

// Provider Registration API & Service Access API
public class ServiceManager {
    private ServiceManager() {}
    
    private static final Map<String, Provider> providers = new ConcurrentHashMap<>();
    
    // Provider registration API
    public static void registProvider(String name, Provider p) {
        providers.put(name, p);
    }
    
    // ServiceInterface access API
    public static ServiceInterface newInstance(String name) {
        Provider p = providers.get(name);
        return p == null ? null : p.newService();
    }
}

// Service Provider Interface
public interface ServiceProvider {
    ServiceInterface newService();
}

public class EntityProvider implements ServiceProvider {
    static {
        ServiceManager.registProvider("ServiceInterfaceImpl", new EntityProvider());
    }

    @Override
    public ServiceInterface newService() {
        return new ServiceImpl();
    }
}
~~~

再给个 JDBC 的例子：

- Service Interface：Connection
- Provider Registration API：Drivertanager.registerDriver
- Service Access API：Drivertanager.getConnection
- Service Provider Interface（可选的）：Drive

ServiceLoader 为JDK 原生提供的 SPF 



### ResourceBundle

ResourceeBundle 类读取 properties 配置文件
