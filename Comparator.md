# 📝 自定义排序裁判：Comparator 核心笔记与实战

## 核心思想

> `Comparator` 的核心思想是**将“比较两个对象的逻辑”从“排序算法本身”中分离出来**。它是一个可插拔的、独立的“规则手册”，让通用的排序算法（如 `TimSort`）可以处理任何类型的自定义排序需求。

## 关键合同：`compare` 方法的返回值

这是所有 `Comparator` 的基石，是与排序算法沟通的唯一语言。

> `int compare(T o1, T o2)`
>
> -   返回**负数** (如 -1): `o1` 排在 `o2` **前面**。
> -   返回**正数** (如 1): `o1` 排在 `o2` **后面**。
> -   返回**零 (0)**: `o1` 和 `o2` 顺序不变（依赖排序稳定性）。
>
> **核心口诀：负前正后 (负数 o1 在前，正数 o1 在后)**

---

## 从零编写 `compare` 逻辑的三步心法

1.  **明确“尺子” (Clarify the "Ruler")**: 你要根据哪个属性来排序？（例如：`age`, `score`, `length`）
2.  **明确“方向” (Clarify the "Direction")**: 你想要升序还是降序？
3.  **选择“工具” (Choose the "Tool")**: 根据属性类型，选择最合适的代码工具来表达比较。
    -   **数字（升序）**: `Integer.compare(key1, key2)` (最安全) 或 `key1 - key2` (有溢出风险)。
    -   **数字（降序）**: `Integer.compare(key2, key1)` 或 `key2 - key1`。
    -   **对象/字符串（升序）**: `key1.compareTo(key2)`。
    -   **对象/字符串（降序）**: `key2.compareTo(key1)`。
    -   **多级排序**: 使用 `if-else` 逻辑链，先判断主要规则，若结果为 `0` 再判断次要规则。

---

## `Comparator` 的三种实现方式

这是 Java 语法的演进之路，从笨重到优雅。

#### 1. 经典方式：匿名内部类 (Java 7 及以前)
最原始、最啰嗦，但逻辑清晰，适用于任何复杂场景。
```java
list.sort(new Comparator<User>() {
    @Override
    public int compare(User o1, User o2) {
        return o1.getAge() - o2.getAge();
    }
});
````

#### 2\. 现代方式：Lambda 表达式 (Java 8+)

替代函数式接口匿名类的语法糖，代码更简洁。

```java
list.sort((o1, o2) -> o1.getAge() - o2.getAge());
```

#### 3\. 专业方式：静态辅助方法 (Java 8+，**强烈推荐**)

可读性最强、最安全、最灵活的声明式方法。

```java
list.sort(Comparator.comparingInt(User::getAge));
```

-----

## 高级技巧：构建复杂排序规则链

使用 `Comparator` 的静态辅助方法，可以像搭积木一样构建任意复杂的排序逻辑。

**场景**：先按分数降序，分数相同按名字升序，如果连名字都可能为 `null`，把 `null` 放在最后。

```java
import static java.util.Comparator.*;

Comparator<User> myComplexComparator = 
    comparingInt(User::getScore).reversed()         // 1. 按分数降序
    .thenComparing(User::getName, nullsLast(naturalOrder())); // 2. 然后按名字升序, null在后

list.sort(myComplexComparator);
```

  - `.reversed()`: 将升序比较器反转为降序。
  - `.thenComparing()`: 添加次级排序规则。
  - `nullsFirst()` / `nullsLast()`: 优雅地处理 `null` 值，避免 `NullPointerException`。

-----

## 与 `Comparable` 的区别

这是另一个核心概念，两者经常被混淆。

| 特性 | `Comparable<T>` (内部比较器) | `Comparator<T>` (外部比较器) |
| :--- | :--- | :--- |
| **定义位置** | 在**类定义中**实现 (`class User implements Comparable`) | 在**外部**独立实现 |
| **核心目的** | 定义一个类的**自然顺序**或**默认顺序** | 定义一个类的**特定顺序**或**临时顺序** |
| **方法签名** | `int compareTo(T other)` (自己和别人比) | `int compare(T o1, T o2)` (裁判评判两者) |
| **数量** | 一个类**只能有一个**自然顺序 | 一个类可以有**无数个**外部比较器 |
| **类比** | 人的**身高** (与生俱来，默认的比较标准) | 比赛的**规则** (今天比体重，明天比年龄) |

-----

-----

## 理论结合实践：一个完整的重构案例

下面我们将运用上述所有知识点，来完整地重构一个真实的排序需求。

### A. 案例场景

我们有一个 `User` 类，需要对一个 `User` 列表进行排序。

  - **排序规则**:

    1.  **首要规则**：按**积分 (score) 降序**排列（分数高的排在前面）。
    2.  **次要规则**：如果积分相同，则按**用户名 (name) 的字典序升序**排列（a, b, c...）。

  - **`User` 类定义**:

<!-- end list -->

```java
class User {
    private String name;
    private int score;

    public User(String name, int score) { this.name = name; this.score = score; }
    public String getName() { return name; }
    public int getScore() { return score; }

    @Override
    public String toString() { return name + "{score=" + score + "}"; }
}
```

### B. 演进之路

#### 阶段一：原始的匿名类 (Java 7 风格)

这是我们的起点，代码逻辑交织在 `if-else` 中，显得非常笨重。

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Comparator;

List<User> userList = new ArrayList<>();
userList.add(new User("alice", 90));
userList.add(new User("bob", 100));
userList.add(new User("cathy", 90));

userList.sort(new Comparator<User>() {
    @Override
    public int compare(User u1, User u2) {
        // 应用“三步心法”：尺子是 score/name, 方向是 降序/升序
        if (u1.getScore() != u2.getScore()) {
            // 工具：降序用 key2 - key1
            return u2.getScore() - u1.getScore(); 
        } else {
            // 工具：升序用 key1.compareTo(key2)
            return u1.getName().compareTo(u2.getName());
        }
    }
});

// 期望输出: [bob{score=100}, alice{score=90}, cathy{score=90}]
// 注意：alice 和 cathy 都是90分，按名字字母序，a 在 c 前面，所以 alice 在 cathy 前面。
System.out.println(userList);
```

#### 阶段二：现代的 Lambda 表达式

我们可以用 Lambda 替换匿名内部类，代码行数减少，但核心的 `if-else` 逻辑依然存在。

```java
userList.sort((u1, u2) -> {
    if (u1.getScore() != u2.getScore()) {
        return u2.getScore() - u1.getScore();
    } else {
        return u1.getName().compareTo(u2.getName());
    }
});
```

#### 阶段三：专业的静态辅助方法与方法引用

这才是最终的、最值得推荐的形态。我们不再描述“如何做”，而是声明“要什么”。

```java
userList.sort(
    Comparator.comparingInt(User::getScore).reversed() // 规则1: 按分数降序
              .thenComparing(User::getName)          // 规则2: 然后按名字升序
);
```

### C. 总结

| 演进阶段 | 代码片段 | 优点/缺点 |
| :--- | :--- | :--- |
| **匿名类** | `new Comparator<>() { ... }` | **优点**: 逻辑集中，兼容性好。\<br\>**缺点**: 极其啰嗦，可读性差。|
| **Lambda** | `(u1, u2) -> { if... }` | **优点**: 比匿名类简洁。\<br\>**缺点**: 复杂逻辑依然需要手写 `if-else`。|
| **辅助方法** | `comparing(...).thenComparing(...)`| **优点**: 极度简洁，声明式，可读性强，安全，易于扩展。\<br\>**缺点**: 需要对 API 有一定了解。|

这个案例清晰地展示了，`Comparator` 的现代用法带来的不仅是代码行数的减少，更是一种**思维方式的转变**：从**命令式**（告诉计算机“如何”做 `if-else`）转变为**声明式**（告诉计算机我“想要什么”样的排序规则）。

```
```
