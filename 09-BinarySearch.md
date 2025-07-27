好的，二分查找是算法学习的基石之一，我们必须把它牢固地沉淀下来。

这次的沉淀，我们会把我们刚刚修正代码时学到的“**递归实现要点**”也包含进去，让这次的笔记更有价值。

-----

### **1. 核心思想：高效的“排除法”**

二分查找的本质，是一种基于“**分而治之 (Divide and Conquer)**”思想的高效查找策略。

**它的核心前提**：**数组必须已经排好序。**

**核心操作**：它从不进行无效的查找。每查找一次，都直接去当前范围的**正中间**进行比较。根据比较结果（偏大、偏小、或正好命中），它可以**立即排除掉一半**不可能存在答案的区域，从而使查找范围指数级地缩小。

> **经典类比**：查字典或“猜数字”游戏。我们从不从第一页或数字1开始，而是永远从中间开始，每一次提问都能排除掉一半的可能性。

-----

### **2. 两种实现思路：迭代与递归**

我们刚刚已经分析过，解决二分查找有两种经典的“交通工具”：

1.  **思路一：迭代法 (循环)**

      * **比喻**：“勤奋的员工”。
      * **逻辑**：使用 `left` 和 `right` 两个指针来维护一个查找区间 `[left, right]`。在一个 `while` 循环中，不断地计算中间点 `mid`，进行比较，然后通过更新 `left = mid + 1` 或 `right = mid - 1` 来收缩查找区间，直到找到目标或区间为空。
      * **优点**：空间复杂度为 $O(1)$，没有函数调用开销，性能略好，且绝对不会有“栈溢出”风险。**这是在工程实践中的首选。**

2.  **思路二：递归法**

      * **比喻**：“喜欢委托的经理”。
      * **逻辑**：函数只负责检查当前 `[left, right]` 区间的中间点。如果没找到，它就把“查找左半部分”或“查找右半部分”这个结构完全一样、但规模更小的任务，**委托给一个新的、和自己一样的函数**去执行。
      * **优点**：代码结构与“分而治之”的思想高度统一，对于某些人来说逻辑更清晰、代码更优雅。

-----

### **3. 两种核心代码实现**

#### **解法一：迭代法 (更推荐)**

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;

        while (left <= right) {
            // 使用 (right - left) / 2 可以防止 left + right 整数溢出
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] < target) {
                left = mid + 1; // 目标在右侧，抛弃左半部分
            } else {
                right = mid - 1; // 目标在左侧，抛弃右半部分
            }
        }
        
        // 如果循环结束仍未找到
        return -1;
    }
}
```

#### **解法二：递归法**

```java
class Solution {
    // LeetCode 调用的“壳函数”
    public int search(int[] nums, int target) {
        return binarySearchRecursive(nums, target, 0, nums.length - 1);
    }
    
    // 带有边界参数的“核函数”
    private int binarySearchRecursive(int[] nums, int target, int left, int right) {
        // 1. 终止条件（Base Case）永远放在最前面
        if (left > right) {
            return -1;
        }
        
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            // 2. 将子问题的结果 return 回去
            return binarySearchRecursive(nums, target, mid + 1, right);
        } else {
            return binarySearchRecursive(nums, target, left, mid - 1);
        }
    }
}
```

-----

### **4. 总结与沉淀卡片**

-----

### 📝 LeetCode 沉淀卡片

  - **题号与标题**: [704. 二分查找 (Binary Search)](https://leetcode.cn/problems/binary-search/)

  - **核心思想**: 利用数组的**有序性**，通过不断将查找范围**对半分割**，来指数级地减少比较次数，从而实现高效查找。

  - **关键技巧**:

    1.  **分而治之 (Divide and Conquer)**：通过维护 `left`, `right` 两个指针来定义当前查找区间 `[left, right]`，并计算中间点 `mid`。
    2.  **区间排除**：根据 `nums[mid]` 与 `target` 的大小关系，**每次排除掉一半**不可能存在答案的区间。
    3.  **（本次犯错的关键沉淀点）递归实现要点**：
          - 使用“**主函数+辅助函数**”的模式来满足 LeetCode 的方法签名。
          - 递归函数第一件事就是检查**终止条件 `(left > right)`**。
          - 参数 `left` 和 `right` 在方法签名中定义后，**不能**在方法体内重复定义。

  - **复杂度分析**:

      - **时间复杂度**: $O(\\log n)$ (每次都将问题规模减半)
      - **空间复杂度**:
          - 迭代法：$O(1)$
          - 递归法：$O(\\log n)$ (用于存储递归调用栈)

  - **同类题型**:

      - [LeetCode 35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/): 二分查找的直接应用，找不到时需要返回应该插入的位置。
      - [LeetCode 278. 第一个错误的版本](https://leetcode.cn/problems/first-bad-version/): 在一个概念性的、非数组的有序序列（版本号）上应用二分查找。
      - [LeetCode 34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/): 二分查找的重要变体，要求在找到目标后，继续向左或向右收缩边界，以找到目标的首次和末次出现位置。

-----
