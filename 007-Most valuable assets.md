### **1. 你的思路与一个“经典”错误**

你的整体解题思路（“两步走”：先计算总和存入新数组，再找最大值）非常清晰，逻辑上完全正确。

你遇到的问题，是一个非常细节、但也极其重要的**语法点**，几乎所有初学者在处理二维数组时都会遇到。

**错误代码回顾：**

```java
// 你的内层循环
for(int j = 0; j < accounts.length; j++){ // 错误！
    nums[i] += accounts[i][j];
}
```

**错误分析**：
这里的 `accounts.length` 获取的是**客户的数量（行数）**，但内层循环的目的是遍历单个客户的**银行账户数量（列数）**。当行数和列数不同时，就会导致逻辑错误或程序因“数组下标越界”而崩溃。

#### **沉淀点：二维数组的“长”和“宽”**

为了彻底搞懂这个问题，我们可以把二维数组想象成一个“**书架**”。

  * `int[][] accounts`：代表**整个书架**。
  * `accounts.length`：代表这个书架总共有多少**层**（**行数**）。
  * `accounts[i]`：代表书架的**第 `i` 层**（一个一维数组）。
  * `accounts[i].length`：代表第 `i` 层书架上，具体放了多少**本书**（**第 `i` 行的列数**）。

> **金科玉律**：处理二维数组 `T[][] array` 时，永远记住：
>
>   * `array.length`   -\> **行数**
>   * `array[i].length` -\> **第 `i` 行的列数**

把这个规则记下来，以后所有二维数组问题都不会再犯同样的错误。

-----

### **2. 优化思路详解**

你的“两步走”逻辑很清晰。一个更优化的思路是“**一步到位**”，即在计算每个客户总资产的同时，就去和当前的最大值进行比较。

这样做的好处是，可以省去创建一个额外的一维数组 `nums` 来存储所有总资产的步骤，从而把**空间复杂度从 $O(m)$ 降到 $O(1)$**（`m`是客户数量）。

-----

### **3. 最优解代码 (一步到位)**

```java
class Solution {
    public int maximumWealth(int[][] accounts) {
        // 用来记录最大资产的“擂主”，初始为0
        int maxWealth = 0;

        // 外层循环：遍历每一个客户（每一行）
        for (int[] customerAccount : accounts) {
            
            // 计算当前客户的总资产
            int currentWealth = 0;
            for (int money : customerAccount) {
                currentWealth += money;
            }

            // 打擂台：如果当前客户更富有，就更新擂主
            maxWealth = Math.max(maxWealth, currentWealth);
        }

        // 所有客户都比较完毕，返回最终的擂主
        return maxWealth;
    }
}
```

-----

### **4. 总结与沉淀卡片**

-----

### 📝 LeetCode 沉淀卡片

  - **题号与标题**: [1672. 最富有客户的资产总量](https://leetcode.cn/problems/richest-customer-wealth/)

  - **核心思想**: 遍历二维数组，通过累加每一行的值来计算单个客户的财富，并持续追踪、更新所遇到的最大财富值。

  - **关键技巧**: **嵌套循环遍历二维数组**。
    核心是理解并正确使用 `array.length` (获取**行数**) 和 `array[i].length` (获取**第 `i` 行的列数**)。这是处理所有二维数组问题的基础。

  - **复杂度分析**: (`m` = 客户数, `n` = 银行数)

      - **我的解法** (两步走)：Time $O(m \\cdot n)$, Space $O(m)$ (用于存储每个客户的总财富)
      - **最优解法** (一步到位)：Time $O(m \\cdot n)$, Space $O(1)$ (只使用固定几个变量)

  - **同类题型**: (用于巩固二维数组遍历)

      - [LeetCode 1572. 矩阵对角线元素的和](https://leetcode.cn/problems/matrix-diagonal-sum/): 经典的二维数组遍历，需要精确的下标计算。
      - [LeetCode 867. 转置矩阵](https://leetcode.cn/problems/transpose-matrix/): 考察对二维数组行列互换的理解和操作。

-----
