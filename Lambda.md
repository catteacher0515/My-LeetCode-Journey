# 📝 Lambda 表达式转换核心笔记与实战

## 核心思想

> Lambda 表达式是**函数式接口**匿名实现的“语法糖”。其本质是用更少的代码，将**“行为（一段代码）”**像数据一样在方法间传递，从而让代码更简洁、表达力更强。

## 关键原则

> **转换的唯一前提：目标接口必须是函数式接口。**
>
> 一个接口只有在“有且仅有一个抽象方法”时，才有资格成为函数式接口，也才有资格被 Lambda 表达式所替代。这是所有转换的“准入资格证”。

---

## 三步转换法 (The 3-Step Conversion Method)

这是一个通用的、绝不会出错的转换流程。

### 第一步：识别靶心 (Identify the Target)
- **做什么？** 在代码中找到 `new InterfaceName() { ... }` 这段匿名类实现。
- **找什么？** 明确两点：
  1.  **接口是谁？** (例如 `Runnable`, `Comparator`)
  2.  **唯一的抽象方法是什么？** (例如 `run()`, `compare(p1, p2)`)，并看清其**参数列表**和**返回值类型**。

### 第二步：翻译蓝图 (Translate the Blueprint)
- **做什么？** 根据第一步识别出的方法签名，编写一个基础的、未简化的 Lambda 表达式。
- **怎么做？** 严格遵循 `(参数列表) -> { 方法体 }` 的格式进行“直译”。
  - `()` 中填写核心方法的参数列表。
  - `{}` 中填写核心方法的方法体代码。
- **例 (排序):** `compare(Person p1, Person p2)` -> `(p1, p2) -> { return p1.age - p2.age; }`

### 第三步：简化打磨 (Simplify & Polish)
- **做什么？** 应用语法糖，将基础 Lambda 变得更优雅、更地道。
- **怎么做？** 依次检查以下简化规则：
  1.  **单行方法体**: 如果方法体只有一行 `return` 语句，可以省略 `{}` 和 `return` 关键字。
      - `(p1, p2) -> { return p1.age - p2.age; }` **→** `(p1, p2) -> p1.age - p2.age`
  2.  **单参数**: 如果参数只有一个，可以省略参数的 `()`。
      - `(s) -> s.length()` **→** `s -> s.length()`
  3.  **方法引用**: 如果 Lambda 体只是在调用一个已存在的方法，使用 `::`。
      - `s -> s.length()` **→** `String::length`
      - `x -> System.out.println(x)` **→** `System.out::println`

---

## 常见函数式接口“速查表”

这张表总结了我们练习过的接口，是你日常开发中的得力助手。

| 接口 (Interface) | 核心方法 (Core Method) | 用途描述（“我能做什么”） | Lambda 范例 |
| :--- | :--- | :--- | :--- |
| `Runnable` | `void run()` | 执行一个无参、无返回值的动作。 | `() -> System.out.println("Hi")` |
| `Consumer<T>` | `void accept(T t)` | **消费**一个数据，不产生任何返回。 | `user -> System.out.println(user)`|
| `Predicate<T>` | `boolean test(T t)` | **断言**一个数据，返回`true`或`false`。 | `s -> s.isEmpty()` |
| `Function<T, R>` | `R apply(T t)` | **转换**一个数据，从 T 类型变为 R 类型。| `s -> s.length()` |
| `Comparator<T>`| `int compare(T o1, T o2)` | **比较**两个数据，定义它们的顺序。 | `(p1, p2) -> p1.getAge() - p2.getAge()`|

---

## 进阶技巧与底层逻辑

- **进阶技巧**:
  - **方法引用 `::`**: 这是 Lambda 的终极形态。当你的 Lambda 只是一个“搬运工”（把参数直接传给另一个方法）时，优先使用方法引用。
  - **`Comparator` 辅助方法**: 在排序时，放弃手写 `(a, b) -> ...`，优先使用 `Comparator.comparing(User::getAge).thenComparing(User::getName)`，代码会更安全、更易读、更易于扩展。

- **底层逻辑**:
  > **类型推断 (Type Inference)**
  >
  > 你不需要为 Lambda 表达式明确指定类型，是因为 Java 编译器非常智能。它会根据**上下文环境**（比如 `list.sort()` 方法需要一个 `Comparator` 参数），来推断出你的 Lambda 应该是什么类型，并检查你的 Lambda 签名是否与该函数式接口的抽象方法匹配。这就是为什么简洁的语法能够成功运行的根本原因。

---
---

## 理论结合实践：一个完整的重构案例

下面我们将运用上述所有知识点，来完整地重构一个真实的排序需求。

### A. 案例场景

假设我们有一个 `User` 类，包含用户名和积分。现在的需求是：对一个 `User` 列表进行排序。

- **排序规则**:
  1.  **首要规则**：按**积分 (score) 降序**排列（分数高的排在前面）。
  2.  **次要规则**：如果积分相同，则按**用户名 (name) 的字典序升序**排列（a, b, c...）。

- **`User` 类定义**:
```java
class User {
    private String name;
    private int score;

    public User(String name, int score) {
        this.name = name;
        this.score = score;
    }

    public String getName() { return name; }
    public int getScore() { return score; }

    @Override
    public String toString() {
        return name + "{score=" + score + "}";
    }
}
````

### B. 演进之路

#### 阶段一：原始的匿名类 (Java 7 风格)

在没有 Lambda 的时代，我们只能通过匿名内部类来实现这个复杂的排序逻辑。代码显得非常笨重和冗长。

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

List<User> userList = new ArrayList<>();
userList.add(new User("alice", 90));
userList.add(new User("bob", 100));
userList.add(new User("cathy", 90));

// 起点：一个非常臃肿的匿名内部类
userList.sort(new Comparator<User>() {
    @Override
    public int compare(User u1, User u2) {
        // 规则1：比较积分
        if (u1.getScore() != u2.getScore()) {
            // 注意是降序，所以是 u2 - u1
            return u2.getScore() - u1.getScore(); 
        } else {
            // 规则2：积分相同，比较用户名（升序）
            return u1.getName().compareTo(u2.getName());
        }
    }
});

System.out.println(userList);
// 期望输出: [bob{score=100}, alice{score=90}, cathy{score=90}]
```

#### 阶段二：现代的 Lambda 表达式

我们可以用 Lambda 替换匿名内部类，代码行数减少，但核心的 `if-else` 逻辑依然存在。

```java
// 第一次转换：变成了多行 Lambda，但依然很啰嗦
userList.sort((u1, u2) -> {
    if (u1.getScore() != u2.getScore()) {
        return u2.getScore() - u1.getScore();
    } else {
        return u1.getName().compareTo(u2.getName());
    }
});
```

*(注：参数类型 `User` 也可以省略，编译器能自动推断)*

#### 阶段三：专业的静态辅助方法与方法引用

这才是现代 Java 解决复杂排序的最终形态。我们不再自己手写 `if-else`，而是像搭积木一样，用声明式的方式来**组合**排序规则。

```java
import java.util.Comparator;

// 最终形态：使用 Comparator 辅助方法和方法引用
userList.sort(
    Comparator.comparingInt(User::getScore).reversed() // 规则1: 按分数降序
              .thenComparing(User::getName)          // 规则2: 然后按名字升序
);

System.out.println(userList);
```

### C. 最终代码与总结

| 演进阶段 | 代码片段 | 优点/缺点 |
| :--- | :--- | :--- |
| **匿名类** | `new Comparator<>() { ... }` | **优点**: 逻辑集中，兼容性好。\<br\>**缺点**: 极其啰嗦，可读性差。|
| **Lambda** | `(u1, u2) -> { if... }` | **优点**: 比匿名类简洁。\<br\>**缺点**: 复杂逻辑依然需要手写 `if-else`。|
| **辅助方法** | `comparing(...).thenComparing(...)`| **优点**: 极度简洁，声明式，可读性强，安全，易于扩展。\<br\>**缺点**: 需要对 API 有一定了解。|

**案例结论**:
这个案例清晰地展示了，Lambda 和它的生态工具（如 `Comparator` 辅助方法）带来的不仅仅是代码行数的减少，更是一种**思维方式的转变**：

  - 从**命令式**（告诉计算机“如何”一步步地做 `if-else` 判断）
  - 转变为**声明式**（告诉计算机我“想要什么”样的排序规则）。

后者代码更简洁、意图更清晰、组合性更强，是更优秀、更专业的编程范式。

```
```
