# Stream

[TOC]

## 概述

流（Streams）是「声明式编程范式」（Declarative programming）的实践，通过抽象底层控制流，允许开发者以「做什么」而非「如何做」的方式来表达处理数据的逻辑。相比较传统的命令式编程，声明式编程在可读性方面上更加优秀，下面是代码对比：

```java
//声明式编程
import java.util.*;
public class Randoms {
    public static void main(String[] args) {
        //内部迭代（internal iteration）
        new Random(47)
            .ints(5, 20)
            .distinct()
            .limit(7)
            .sorted()			
            .forEach(System.out::println);
    }
    
}


//命令式编程
import java.util.*;
public class ImperativeRandoms {
    public static void main(String[] args) {
        Random rand = new Random(47);
        SortedSet<Integer> rints = new TreeSet<>();
        //外部迭代（external iteration）
        while(rints.size() < 7) {				
            int r = rand.nextInt(20);
            if(r < 5) continue;
            rints.add(r);
        }
        System.out.println(rints);
    }
}
```

流式编程的两个核心特征：内部迭代 & 惰性（按需）加载。



流操作的类型有三种：

- 创建流
- 修改流元素（中间操作， Intermediate Operations）
- 消费流元素（终端操作， Terminal Operations）

## 创建流

`java.util.stream.Stream`

- `static <T> Stream<T> of(T... values)`

  yields a stream whose elements are the given values.

- `static <T> Stream<T> empty()`

  yields a stream with no elements.

- `static <T> Stream<T> generate(Supplier<T> s)`

  yields an infinite stream whose elements are constructed by repeatedly invoking the function `s`.

- `static <T> Stream<T> iterate(T seed, UnaryOperator<T> f)`

- `static <T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> f)`

  yield a stream whose elements are `seed`, `f` invoked on `seed`, `f` invoked on the preceding element, and so on. The first method yields an infinite stream. The stream of the second method comes to an end before the first element that doesn't fulfill the `hasNext` predicate.

- `static <T> Stream<T> ofNullable(T t)`

  returns an empty stream if `t` is `null` or a stream containing `t` otherwise.

- `interface Builder<T>`：

  ```java
  Stream.Builder<Integer> builder = Stream.builder();
  builder.add(1);
  Stream<Integer> stream = builder.build();
  ```

  



`java.util.Collection<E>`：

- `default Stream<E> stream()`

- `default Stream<E> parallelStream()` yield a sequential or parallel stream of the elements in this collection.

  ```java
  List<String> words = List.of("Hello", "World");
  Stream<String> stream = words.stream();
  ```

  

`java.util.Spliterators`：

- `static <T> Spliterator<T> spliteratorUnknownSize(Iterator<? extends T> iterator, int characteristics)`

  turns an iterator into a splittable iterator of unknown size with the given characteristics (a bit pattern containing constants such as `Spliterator.ORDERED`)


If you have an `Iterator` and want a stream of its results, use

```java
StreamSupport.stream(Spliterators.spliteratorUnknownSize(
   iterator, Spliterator.ORDERED), false);
```

If you have an `Iterable` that is not a collection, you can turn it into a stream by calling

```java
StreamSupport.stream(iterable.spliterator(), false);
```





`java.util.Arrays` 

- `static <T> Stream<T> stream(T[] array, int startInclusive, int endExclusive)` 

  yields a stream whose elements are the specified range of the array.

`java.util.regex.Pattern`

- `Stream<String> splitAsStream(CharSequence input)`

  yields a stream whose elements are the parts of the input that are delimited by this pattern.

`java.util.Scanner`

- `public Stream<String> tokens()`

  yields a stream of strings returned by calling the `next` method of this scanner.



Streams enable you to execute possibly-parallel aggregate operations over a variety of data sources, including even non-thread-safe collections such as `ArrayList`. This is possible only if we can prevent **interference** with the data source during the execution of a stream pipeline.  For most data sources, preventing interference means ensuring that the data source is not modified at all during the execution of the stream pipeline. The notable exception to this are streams whose sources are concurrent collections,  which are specifically designed to handle concurrent modification.

```java
List<String> wordList = new ArrayList(...);
Stream<String> words = wordList.stream();
// ERROR -- interference
// throws java.util.ConcurrentModificationException
words.forEach(s -> {
    if (s.length() < 12) 
        wordList.remove(s);
});
```

## 中间操作

### Transformation

- `Stream<T> filter(Predicate<? super T> predicate)`

  yields a stream containing the elements of this stream that fulfill the predicate.

- `<R> Stream<R> map(Function<? super T,? extends R> mapper)`

  yields a stream containing the results of applying `mapper` to the elements of this stream.

- `<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)`

  yields a stream obtained by concatenating the results of applying `mapper` to the elements of this stream. (Note that each `mapper` result is a stream.)

- `<R> Stream<R> mapMulti(BiConsumer<? super T,? super Consumer<R>> mapper)` 

  For each stream element, the `mapper` is invoked, and all elements passed to the `Consumer` during invocation are added to the result stream.



- `Stream<T> distinct()`

  yields a stream of the distinct elements of this stream.

- `Stream<T> sorted()`

- `Stream<T> sorted(Comparator<? super T> comparator)`

  yield a stream whose elements are the elements of this stream in sorted order. The first method requires that the elements are instances of a class implementing `Comparable`.

- `Stream<T> peek(Consumer<? super T> action)`

  yields a stream with the same elements as this stream, passing each element to `action` as it is consumed.



### Extracting Substreams and Combining Streams

- `Stream<T> limit(long maxSize)`

  yields a stream with up to `maxSize` of the initial elements from this stream.

- `Stream<T> skip(long n)`

  yields a stream whose elements are all but the initial `n` elements of this stream.

- `Stream<T> takeWhile(Predicate<? super T> predicate)`

  yields a stream whose elements are the initial elements of this stream that fulfill the predicate.

  ```java
  List<Integer> numberList= Arrays.asList(1,20,35,2,5);
  numberList.stream()
      .takeWhile(num -> num <= 20)
      .forEach(System.out::println);
  // 1 20
  ```

- `Stream<T> dropWhile(Predicate<? super T> predicate)`

  yields a stream whose elements are the elements of this stream except for the **initial** ones that fulfill the predicate.

  ```java
  List<Integer> numberList= Arrays.asList(1,20,35,2,5);
  numberList.stream()
      .dropWhile(num -> num <= 20)
      .forEach(System.out::println);
  // 35 2 5
  ```

- `static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)`

  yields a stream whose elements are the elements of `a` followed by the elements of `b`.

  ```java
  Stream<String> combined = Stream.concat(
     graphemeClusters("Hello"), graphemeClusters("World"));
     // Yields the stream ["H", "e", "l", "l", "o", "W", "o", "r", "l", "d"]
  ```





## 终端操作

### Reductions

Reductions are *terminal operations*. They reduce the stream to a nonstream value

- `Optional<T> max(Comparator<? super T> comparator)`

- `Optional<T> min(Comparator<? super T> comparator)`

  yield a maximum or minimum element of this stream, using the ordering defined by the given comparator, or an empty `Optional` if this stream is empty. 

- `Optional<T> findFirst()`

- `Optional<T> findAny()`

  yield the first, or any, element of this stream, or an empty `Optional` if this stream is empty. `findAny` method is effective when you parallelize the stream

- `boolean anyMatch(Predicate<? super T> predicate)`

- `boolean allMatch(Predicate<? super T> predicate)`

- `boolean noneMatch(Predicate<? super T> predicate)`

  return `true` if any, all, or none of the elements of this stream match the given predicate. 



### Collecting

The `Collectors` class provides a large number of factory methods for common collectors

These calls give you a list or set, but you cannot make any further assumptions. The collection might not be mutable, serializable, or thread-safe. If you want to control which kind of collection you get, use the following call instead:

```java
TreeSet<String> result = stream.collect(Collectors.toCollection(TreeSet::new))
```



### Grouping & Partitioning



## Optional

Optional 是 Java 8 引进的一个新特性，我们通常认为 Optional 是用于解决 Java 臭名昭著的空指针异常问题。Brian Goetz （Java语言设计架构师）对 Optional 设计意图的原话如下：

> Optional is intended to provide a limited mechanism for library method return types where there needed to be a clear way to represent “no result," and using null for such was overwhelmingly likely to cause errors.

Optional 的机制类似于 Java 的检查异常，**强迫 API 调用者面对没有返回值的现实**。不要将Optiona对象作为字段、参数，以及实现 if-else 逻辑。



An `Optional<T>` is a container that either holds an object of type `T` (present) or nothing at all (empty)



Creating Optional Values：

- `static <T> Optional<T> of(T value)`

- `static <T> Optional<T> ofNullable(T value)`

  yield an `Optional` with the given value. If `value` is `null`, the first method throws a `NullPointerException` and the second method yields an empty `Optional`.

- `static <T> Optional<T> empty()`

  yields an empty `Optional`.



Getting an Optional Value：

- `T orElse(T other)`

  yields the value of this `Optional`, or `other` if this `Optional` is empty.

- `T orElseGet(Supplier<? extends T> other)`

  yields the value of this `Optional`, or the result of invoking `other` if this `Optional` is empty.

- `<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)`

  yields the value of this `Optional`, or throws the result of invoking `exceptionSupplier` if this `Optional` is empty.



Consuming an Optional Value：

- `void ifPresent(Consumer<? super T> action)`

  If this `Optional` is present, passes its value to `action`.

- `void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`

  If this `Optional` is present, passes its value to `action`, else invokes `emptyAction`.

  ```java
  optionalValue.ifPresentOrElse(
      v -> System.out.println("Found " + v),
      () -> logger.warning("No match")
  );
  ```



If you don't use `Optional` values correctly, you have no benefit over the “something or `null`” approach of the past：

- `T get()`

- `T orElseThrow()`

  yield the value of this `Optional`, or throw a `NoSuchElementException` if it is empty.

- `boolean isEmpty()`

- `boolean isPresent()`

  return `true` if this `Optional` is empty or not empty.

If you find yourself using the `get`, `isPresent`, or `isEmpty` methods, try to rethink your approach and use one of the strategies of the preceding method (orElse, orElseGet, ifPresent ...). because 

```java
Optional<T> optionalValue = . . .;
optionalValue.get().someMethod()
```

is no safer than

```java
T value = . . .;
value.someMethod();
```

and

```java
if (optionalValue.isPresent()) 
    optionalValue.get().someMethod();
```

is no easier than

```java
if (value != null) 
    value.someMethod();
```



Pipelining Optional Values：

- `<U> Optional<U> map(Function<? super T,? extends U> mapper)`

  yields an `Optional` whose value is obtained by applying the given function to the value of this `Optional` if present, or an empty `Optional` otherwise.

- `Optional<T> filter(Predicate<? super T> predicate)`

  yields an `Optional` with the value of this `Optional` if it fulfills the given predicate, or an empty `Optional` otherwise.

  ```java
  Optional<Path> transformed = optionalString
      .filter(s -> s.endsWith(".txt"))
      .map(Path::of);
  ```

- `Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)`

  yields this `Optional` if it is nonempty, or the one produced by the supplier otherwise.



`<U> Optional<U> flatMap(Function<? super T,? extends Optional<? extends U>> mapper)`yields the result of applying `mapper` to the value in this `Optional` if present, or an empty optional otherwise.

该方法建模了其他编程语言中可空链式调用（Optional Chaining）的语法：

```java
Optional<Double> result = inverse(arg);
result.ifPresent(x -> {
    result = squareRoot(x);
})

// 等价于
Optional<Double> result = inverse(arg).flatMap(x -> squareRoot(x));
```



`Stream<T> stream()` If a value is present, returns a sequential Stream containing only that value, otherwise returns an empty Stream.

```java
Stream<String> ids = . . .;

Stream<User> users = ids.map(Users::lookup)
    .filter(Optional::isPresent)
    .map(Optional::get);

// 等价于
Stream<User> users = ids.map(Users::lookup)
    .flatMap(Optional::stream);		// 这里的 flatMap 是 Stream 中的方法，而不是 Optional 的
```

