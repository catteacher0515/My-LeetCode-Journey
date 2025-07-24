### **1. 你的代码分析 (暴力解法)**

你的思路非常直接：对于每一个数，都向后遍历，数一数有多少个比它小的。这是一个完全正确的、符合题意的暴力解法。

  - **优点**: 逻辑清晰，代码直观，是思考这道题的绝佳起点。
  - **瓶颈**:
      - **时间复杂度**: $O(n^2)$。两层嵌套循环导致在数据量大时，计算量会急剧增加，导致超时。
      - **语法修正**: 你的代码逻辑正确，但 `main` 方法中缺少 `return` 语句，并且返回类型应为 `List<Integer>` 而非 `int[]`。

**修正后的暴力解代码 (仅用于对比)：**

```java
class BruteForceSolution {
    public List<Integer> countSmaller(int[] nums) {
        List<Integer> resultList = new ArrayList<>();
        if (nums == null || nums.length == 0) {
            return resultList;
        }

        for (int i = 0; i < nums.length; i++) {
            int count = 0;
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[j] < nums[i]) {
                    count++;
                }
            }
            resultList.add(count);
        }
        return resultList;
    }
}
```

-----

### **2. 优化思路详解 (归并排序解法)**

这道题的巧妙解法，是将一个“计数问题”，转化为一个“**排序过程中的副产品**”。

我们知道，**归-并排序**的 `merge` 操作会合并两个**已经有序**的子数组。这个“**已有序**”的特性，就是我们解题的关键。

**核心洞察**:
当我们合并左半边 `L` 和右半边 `R` 这两个有序数组时，假设我们正准备把左半边的 `L[i]` 这个元素放进最终的有序数组里。此时，我们看了一眼右半边的 `R` 数组，发现已经有 `j` 个元素（`R[0]` 到 `R[j-1]`）因为比 `L[i]` 小，而被先放了进去。

这意味着什么？

> **这意味着，我们刚刚拿出的这个 `L[i]`，比它原始位置右边的 `j` 个元素都要大！**

因此，我们可以在 `merge` 的过程中，**顺便**为每个来自左侧数组的元素，累加上它右侧小于它的元素个数。当整个归并排序完成时，所有元素的计数值也就都计算完毕了。

-----

### **3. 最优解代码 (归并排序解法)**

```java
import java.util.ArrayList;
import java.util.List;

class Solution {
    // 用一个内部类来封装值、原始下标和计数值
    private class Pair {
        int val;
        int originalIndex;
        int count;

        Pair(int val, int originalIndex) {
            this.val = val;
            this.originalIndex = originalIndex;
            this.count = 0;
        }
    }

    public List<Integer> countSmaller(int[] nums) {
        if (nums == null || nums.length == 0) {
            return new ArrayList<>();
        }
        int n = nums.length;
        Pair[] pairs = new Pair[n];
        for (int i = 0; i < n; i++) {
            pairs[i] = new Pair(nums[i], i);
        }

        mergeSortAndCount(pairs, 0, n - 1);

        // 创建一个counts数组，按原始下标顺序存放结果
        int[] counts = new int[n];
        for (Pair p : pairs) {
            counts[p.originalIndex] = p.count;
        }

        // 将结果转换为List<Integer>
        List<Integer> resultList = new ArrayList<>(n);
        for (int count : counts) {
            resultList.add(count);
        }
        return resultList;
    }

    private void mergeSortAndCount(Pair[] pairs, int left, int right) {
        if (left >= right) return;
        
        int mid = left + (right - left) / 2;
        mergeSortAndCount(pairs, left, mid);
        mergeSortAndCount(pairs, mid + 1, right);
        merge(pairs, left, mid, right);
    }

    private void merge(Pair[] pairs, int left, int mid, int right) {
        Pair[] temp = new Pair[right - left + 1];
        int i = left;     // 左半边指针
        int j = mid + 1;  // 右半边指针
        int k = 0;        // 临时数组指针
        
        while (i <= mid && j <= right) {
            if (pairs[i].val <= pairs[j].val) {
                // 这是关键：当左边的元素入队时，
                // 我们知道右边已经有 j - (mid + 1) 个元素比它小了
                pairs[i].count += j - (mid + 1);
                temp[k++] = pairs[i++];
            } else {
                // 当右边的元素入队时，我们不做任何计数
                temp[k++] = pairs[j++];
            }
        }

        // 处理剩余元素
        while (i <= mid) {
            // 此时右边所有元素都已入队，所以右边有 (right - mid) 个元素比它小
            pairs[i].count += right - mid;
            temp[k++] = pairs[i++];
        }
        while (j <= right) {
            temp[k++] = pairs[j++];
        }

        // 复制回原数组
        System.arraycopy(temp, 0, pairs, left, temp.length);
    }
}
```

-----

### **4. 总结与沉淀卡片 (Summary & Precipitation Card)**

-----

### 📝 LeetCode 沉淀卡片

  - **题号与标题**: [315. 计算右侧小于当前元素的个数 (Count of Smaller Numbers After Self)](https://leetcode.cn/problems/count-of-smaller-numbers-after-self/)

  - **核心思想**: 将一个直观的“对每个元素向右查找计数”的问题，转化为利用\*\*“分而治之”**思想，在**归并排序的合并过程中\*\*巧妙地完成计数。

  - **关键技巧**: **改造归并排序 (Modified Merge Sort)**。
    在合并两个有序子数组（左`L`和右`R`）时，利用它们已经有序的特性。当我们将左侧数组的元素 `L[i]` 准备放入最终位置时，我们能够**瞬间知道**右侧数组中已经有多少个元素（设为 `j` 个）因为比 `L[i]` 小而被先放了进去。这个 `j` 的值，就是 `L[i]` 在其右侧子数组中拥有的“逆序对”数量。通过在递归的每一层累加这个计数值，最终得到全局的答案。

  - **复杂度分析**:

      - **暴力解**：Time $O(n^2)$, Space $O(n)$ (用于返回列表)
      - **归并排序解**：Time $O(n \\log n)$, Space $O(n)$ (用于归并的临时数组和`Pair`对象)

  - **同类题型**:

      - [LeetCode 493. 翻转对](https://leetcode.cn/problems/reverse-pairs/): “逆序对”问题的经典变体，同样是考察对归并排序 `merge` 过程的深度理解和改造能力。
      - [LeetCode 912. 排序数组](https://leetcode.cn/problems/sort-an-array/): 掌握标准的归并排序是解决此类问题的基础。

-----
