# Java 字符串拼接指南：从 + 到 StringBuilder

在 Java 开发中，字符串拼接是一项极其常见的操作。选择正确的拼接方式，不仅能让代码更简洁，还能在关键时刻极大地影响程序性能。本笔记旨在梳理几种主流的字符串拼接方法，并明确它们各自的最佳适用场景。

---

## 1. `+` 运算符：最直观的选择

这是最基础、也是代码写起来最方便的方式。

### 语法
```java
String part1 = "Hello";
String part2 = "World";
int version = 3;

// 字符串和字符串拼接
String message1 = part1 + " " + part2; // 得到 "Hello World"

// 字符串和非字符串拼接 (非字符串类型会自动转换)
String message2 = "Version: " + version; // 得到 "Version: 3"
````

### 优点

  - **代码简洁**：语法非常直观，可读性极高。
  - **灵活方便**：可以和任何数据类型混合使用。

### 缺点

  - **性能问题（在循环中）**：`String` 对象是**不可变的**。每次使用 `+` 拼接，编译器在背后都会创建一个临时的 `StringBuilder` 对象来完成工作。在**循环**中大量使用 `+` 会频繁创建和销毁临时对象，造成显著的性能开销。

### 适用场景

  - **非循环**环境下的、少量（通常是1-3个）字符串的拼接。
  - 当代码的简洁和可读性是首要考虑因素时。

-----

## 2\. `StringBuilder`：最高效的选择

`StringBuilder` 是 Java 官方专门为了解决 `+` 拼接性能问题而设计的“**字符串建造者**”。

### 语法

```java
// 1. 创建一个 StringBuilder 对象
StringBuilder sb = new StringBuilder();

// 2. 使用 append 方法进行拼接
sb.append("Hello");
sb.append(", ");
sb.append("World");
sb.append("! The year is ");
sb.append(2025);

// 3. 最后，使用 toString() 方法得到最终的 String 对象
String result = sb.toString(); // "Hello, World! The year is 2025"
```

### 优点

  - **性能极高**：`StringBuilder` 内部是可变的，所有 `append` 操作都不会创建新的对象，只在最后 `toString()` 时生成一次。
  - **功能强大**：除了 `append`，还提供了 `insert`、`delete`、`reverse` 等多种灵活的字符串操作方法。
  - **支持链式调用**：
    ```java
    String result = new StringBuilder()
                        .append("Hello")
                        .append(", ")
                        .append("World")
                        .toString();
    ```

### 缺点

  - **线程不安全**：在单线程环境下速度更快，但在多线程环境下，如果多个线程同时修改同一个 `StringBuilder`，可能导致数据错乱。
  - 代码比 `+` 略长。

### 适用场景

  - **循环中**的字符串拼接。
  - 需要进行**大量、复杂**的字符串构建操作。
  - **绝大多数单线程场景下，`StringBuilder` 都是拼接字符串的首选。**

-----

## 3\. `StringBuffer`：线程安全的选择

`StringBuffer` 是 `StringBuilder` 的一个“双胞胎哥哥”，它们的功能几乎完全一样。

### 与 `StringBuilder` 的唯一区别

  - `StringBuffer` 的所有修改方法（如 `append`）都是**线程安全 (thread-safe)** 的，方法内部加了同步锁。
  - 这意味着，你可以在多线程环境下安全地使用同一个 `StringBuffer` 对象。
  - 代价是，因为有加锁和解锁的开销，它的**性能比 `StringBuilder` 稍差**。

### 适用场景

  - **多线程**环境下，需要共享和修改同一个字符串构建器的场景。
  - **注意**：在现代编程中，这种场景相对较少。**99% 的情况下，你都应该优先使用 `StringBuilder`**。

-----

## 4\. `String.join()`：优雅的连接者

这是 Java 8 引入的一个非常方便的静态方法，专门用来**将一个字符串数组或集合，用指定的分隔符连接起来**。

### 语法

```java
import java.util.Arrays;
import java.util.List;

String[] parts = {"Welcome", "to", "Java"};
// 第一个参数是分隔符，第二个参数是要连接的集合
String result1 = String.join(" ", parts); // "Welcome to Java"

List<String> list = Arrays.asList("apple", "banana", "orange");
String result2 = String.join(", ", list); // "apple, banana, orange"
```

### 优点

  - **代码极其优雅**：对于“用分隔符连接集合”这个特定任务，它的代码最简洁，意图最明确。
  - **性能不错**：底层也是用 `StringBuilder` 实现的。

### 缺点

  - **功能单一**：它只能做“用分隔符连接”这一件事。

### 适用场景

  - 当你有一个**字符串数组或列表**，想用一个**固定的分隔符**把它们拼接成一个长字符串时。

-----

## 总结与沉淀

| 方法                | 优点                                   | 缺点                             | 最佳适用场景                               |
| ------------------- | -------------------------------------- | -------------------------------- | ------------------------------------------ |
| **`+` 运算符** | 语法最简洁，可读性高                   | **循环中性能差** | 简单的、非循环的拼接                       |
| **`StringBuilder`** | **性能极高**，功能强大，支持链式调用   | 线程不安全                         | **绝大多数单线程场景，特别是循环中** |
| **`StringBuffer`** | **线程安全** | 性能略低于 `StringBuilder`       | 极少数的多线程共享场景                     |
| **`String.join()`** | 语法优雅，意图明确                     | 功能单一                           | **用固定分隔符连接数组或列表** |

```
```
