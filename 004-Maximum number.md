### **1. 你的代码分析 (Analysis of Your Code)**

你使用了插入排序的框架，并巧妙地将核心的比较逻辑替换成了我们为这道题定制的规则。

  * **优点**：
      * **思路正确**：你准确地抓住了这道题的本质——它是一个自定义排序问题，而不是一个简单的数字排序问题。
      * **学以致用**：你成功地将刚学过的“插入排序”算法，应用到了一个新的、更复杂的问题场景中，这本身就是一次非常成功的练习。
  * **瓶颈**：
      * **时间复杂度**：插入排序的平均和最坏时间复杂度是 $O(n^2)$。虽然核心比较逻辑是对的，但对于这道题的数据规模（`n` 最大可达100），$O(n^2)$ 的算法可能会有超时的风险，而且也不是最优解。

### **2. 优化思路详解 (Detailed Optimization Path)**

你的核心“比较规则”是完全正确的，唯一需要优化的就是实现这个排序的“框架”。我们不必自己手动实现一个 $O(n^2)$ 的插入排序，可以直接利用 Java 内置的、更高效的排序工具。

> **“为什么”要优化？**
> 因为手动实现排序容易出错，且插入排序的效率不够高。我们需要一个平均时间复杂度为 $O(n \\log n)$ 的排序算法。

> **“如何”优化？**
> Java 的 `Arrays.sort()` 方法是一个非常强大的工具。它不仅能给基本类型排序，还允许我们传入一个自定义的比较器 (Comparator)，让它按照我们天马行空的“新规则”来进行排序。
> 我们的任务，就从“实现整个排序过程”，简化为“**只告诉 `Arrays.sort()` 我们的新规则是什么**”。

### **3. 最优解代码 (Optimal Solution Code)**

这份代码使用了 `Arrays.sort()` 和一个简洁的 Lambda 表达式来实现自定义比较器，这是解决此类问题的“标准姿势”。

```java
import java.util.Arrays;
import java.util.Comparator;

class Solution {
    public String largestNumber(int[] nums) {
        // 1. 将 int[] 转换成 String[]，为字符串比较做准备
        String[] arr = new String[nums.length];
        for (int i = 0; i < nums.length; i++) {
            arr[i] = String.valueOf(nums[i]);
        }

        // 2. 使用自定义比较器进行排序
        // 这是整个算法的灵魂
        Comparator<String> customComparator = (a, b) -> (b + a).compareTo(a + b);
        Arrays.sort(arr, customComparator);

        // 3. 处理特殊情况：如果排序后最大的是 "0"，说明原数组全是0
        if (arr[0].equals("0")) {
            return "0";
        }

        // 4. 将排序后的字符串数组拼接成一个结果
        StringBuilder sb = new StringBuilder();
        for (String s : arr) {
            sb.append(s);
        }
        return sb.toString();
    }
}
```

### **4. 总结与沉淀卡片 (Summary & Precipitation Card)**

这是你可以直接复制到你的 GitHub 笔记 `My-LeetCode-Journey` 仓库里的 Markdown 卡片。

-----

### 📝 LeetCode 沉淀卡片

  - **题号与标题**: [179. 最大数 (Largest Number)](https://leetcode.cn/problems/largest-number/)

  - **核心思想**: 将整数排序问题，转化为**自定义规则的字符串排序问题**，目标是拼接出字典序最大的字符串。

  - **关键技巧**: **自定义比较器 (Custom Comparator)**。对于任意两个数字（已转换为字符串）`a` 和 `b`，我们不直接比较它们的大小，而是比较拼接后的字符串 `(a + b)` 和 `(b + a)` 的字典序。若 `(b + a)` 的字典序更大，则 `b` 应该排在 `a` 的前面。这是解决此类问题的“金钥匙”。

  - **复杂度分析**:

      - **暴力解** (全排列)：Time $O(n\! \\cdot l)$，Space $O(n \\cdot l)$ (其中 `l` 是数字平均长度)
      - **最优解** (自定义排序)：
          - Time $O(n \\cdot l \\cdot \\log n)$。排序 `n` 个元素需要 $O(n \\log n)$ 次比较，每次字符串比较和拼接的成本与数字长度 `l` 相关。
          - Space $O(n \\cdot l)$。需要额外空间来存储字符串数组。

  - **同类题型**:

      - [LeetCode 953. 验证外星语词典](https://leetcode.cn/problems/verifying-an-alien-dictionary/): 核心是根据给定的、非标准的字母表顺序，来判断单词列表是否有序，需要完全自定义的比较逻辑。
      - [LeetCode 1356. 根据数字二进制下 1 的数目排序](https://leetcode.cn/problems/sort-integers-by-the-number-of-1-bits/): 排序规则是复合的：优先按二进制中1的个数排，个数相同再按数值大小排。也是自定义比较器的经典应用。

-----
