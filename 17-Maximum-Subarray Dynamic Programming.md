当然，我们这就把“最大子数组和”问题，以及解决它的、极其经典的动态规划思想（**卡丹算法 Kadane's Algorithm**），完整地沉淀下来。

这篇笔记会帮你彻底巩固你对“动态规划”这个概念的第一次成功实践。

**文件名建议**：`Algorithm-DP-Maximum-Subarray.md`

-----

````markdown
# 动态规划入门：最大子数组和 (Kadane's Algorithm)

“最大子数组和”是学习动态规划（Dynamic Programming, DP）思想最经典的入门问题之一。它的解法——卡丹算法，完美地体现了DP“**利用上一步的最优结果，推导出当前的最优结果**”的核心精髓。

## 1. 核心思想：投资鬼才的每日决策

> **问题**: 在一个记录了每日股票涨跌值（可正可负）的数组中，找到**连续**的一段时间进行投资，能获得的最大收益是多少？

这个问题的暴力解法是找出所有可能的连续子数组，然后计算它们的和，这是一个 $O(n^2)$ 的方法。而动态规划提供了一个 $O(n)$ 的线性时间解法。

**DP的思考方式**：
作为投资鬼才，你站在第 `i` 天。你不需要回顾全部历史，只需要问自己一个简单的问题：
> **“对于我‘到今天为止’的这段连续投资，是（A）从今天重新开始算收益更高，还是（B）把我今天的涨跌，加到我‘到昨天为止’的那段连续收益上更高？”**

你只依赖于**昨天的状态**，来做出**今天的最优决策**。这就是动态规划。

## 2. 算法步骤与状态定义

为了实现上述思想，我们需要两个关键的“账本”：

1.  **`maxEndingHere` (当前连续收益)**
    -   **DP状态定义**：`dp[i]`，即**在所有以 `i` 位置元素为结尾的子数组中，能得到的最大和**。
    -   **类比**：你“到今天为止”的最佳连续收益。这份收益报告，是你做明天决策的唯一依据。

2.  **`maxSoFar` (历史最高收益)**
    -   **作用**：一个全局变量，用来记录我们从开始到现在，见过的所有`maxEndingHere`中的最大值。
    -   **类比**：你记事本上的“历史最高连续收益纪录”。

**算法流程**：
1.  初始化 `maxEndingHere` 和 `maxSoFar` 为数组的第一个元素。
2.  从数组的第二个元素开始，向后遍历（一天一天地做决策）。
3.  在第 `i` 天，根据我们的核心决策，更新 `maxEndingHere`：
    `maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);`
4.  然后，用今天刚算出的 `maxEndingHere`，去挑战“历史最高纪录” `maxSoFar`：
    `maxSoFar = Math.max(maxSoFar, maxEndingHere);`
5.  遍历结束后，`maxSoFar` 中保存的就是最终答案。

## 3. Java 代码实现 (空间优化版)

这份代码就是卡丹算法的经典实现。它通过只使用一个变量 `maxEndingHere` 来记录前一个状态 `dp[i-1]`，巧妙地将动态规划所需的 $O(n)$ 空间优化到了 $O(1)$。

```java
public class MaxSubArray {
    public static int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        
        // maxSoFar: 全局最大和，我们的最终答案
        // maxEndingHere: 以当前元素结尾的最大子数组和
        int maxSoFar = nums[0];
        int maxEndingHere = nums[0];
        
        // 从第二个元素开始遍历
        for (int i = 1; i < nums.length; i++) {
            // 核心决策：是“另起炉灶”，还是“继承家业”？
            maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
            
            // 挑战“历史最高纪录”
            maxSoFar = Math.max(maxSoFar, maxEndingHere);
        }
        
        return maxSoFar;
    }
}
````

-----

## 4\. 总结与沉淀卡片

### 📝 算法沉淀卡片：最大子数组和

  - **题号与标题**: [53. 最大子数组和 (Maximum Subarray)](https://leetcode.cn/problems/maximum-subarray/)

  - **核心思想**: **动态规划 (Dynamic Programming)**。通过一次遍历，维护“以当前元素结尾的最大子数组和”，并用它来不断更新“全局最大子数组和”。

  - **关键技巧**:

    1.  **状态定义**: `dp[i]` 的定义是“**以 `nums[i]` 为结尾**的最大和”，而不是“前 `i` 个元素”的最大和，这是解决本题的关键。
    2.  **状态转移方程**: `dp[i] = max(nums[i], dp[i-1] + nums[i])`。这个方程优雅地体现了“另起炉灶”与“继承家业”的决策。
    3.  **空间优化**: 观察到 `dp[i]` 只与 `dp[i-1]` 相关，因此可以用一个变量代替整个 `dp` 数组，将空间复杂度从 $O(n)$ 降至 $O(1)$。

  - **复杂度分析**:

      - **时间复杂度**: $O(n)$ (仅需一次数组遍历)
      - **空间复杂度**: $O(1)$ (只使用常数个额外变量)

  - **同类题型**: (考察动态规划基础思想)

      - [LeetCode 70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/): DP 入门的“Hello World”，状态转移方程更简单。
      - [LeetCode 198. 打家劫舍](https://leetcode.cn/problems/house-robber/): 经典的 DP 问题，状态转移的决策比本题稍微复杂一点（偷或不偷）。

<!-- end list -->

```
```
