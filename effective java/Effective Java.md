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
   4. 可以在运行期动态加载类，这是 SPF 的基础

缺点：

- 如果不提供 public or protected 的构造器，那么其派生类就不能被实例化

- 很难发现静态工厂方法，但是可以通过惯用名称来弥补这一缺点

  - from：类型转换方法

    ~~~java
    Date date = Date.from(18502105L);
    ~~~

  - getType/newType/type：类型转换方法，其中 type 为工厂方法所返回的类型

    ~~~java
    Fi1eStore fs = Fi1es.getFi1eStore(path);
    BufferedReader br = Fi1es.newufferedReader(path);
    List<Comp1aint> 1itany = Col1ections.1ist(1egacyLitany);
    ~~~

  - of：聚合方法

    ~~~java
    Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
    ~~~

  - valueOf：类似 from，类型转换方法

  - instance/getInstance

    ~~~java
    Stackwalker 1uke -stackwa1ker.9etTnstance(options);
    ~~~

  - create/newInstance：与 instance 类似，但是确保每次都返回一个新的实例

    

### 遇到多个构造器参数时要考虑使用构建器

## 通用方法

## 类和接口

## 泛型

## 枚举和注解

## Lambda 与 Stream

## 方法

## 通用编程

## 异常

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
