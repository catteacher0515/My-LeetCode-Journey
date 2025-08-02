好的，N 皇后问题是回溯算法最经典的案例，非常值得我们用一篇完整的笔记把它彻底沉淀下来。

这篇笔记将**完美融合你的代码注释、我们之前讨论过的生动类比、以及一个完整的 `n=4` 的推演过程**，让你对回溯的每一步都了如指掌。

**文件名建议**：`Algorithm-Backtracking-N-Queens.md`

-----

````markdown
# 经典回溯问题：N皇后 (N-Queens) - 学习沉淀

N皇后问题是学习**回溯 (Backtracking)** 算法思想的终极“道场”。它的代码清晰地展示了回溯算法“**选择 -> 探索 -> 撤销**”的核心三部曲，是解决“组合搜索”问题的通用框架。

## 1. 核心思想：会“反悔”的“谨慎”国王

> **故事背景：一场“互相看不顺眼”的舞会**
> 国王要在一个 N×N 的棋盘式舞池里，安排 N 位脾气古怪的皇后。她们互相之间不能处于同一行、同一列、或者同一条对角线上。国王的任务，就是找出所有能让她们和平共处的座位安排方案。

国王的策略，就是回溯算法的策略：
1.  **一行一行地安排**，从第0行开始。
2.  在当前行，**一列一列地尝试**所有可能的座位。
3.  每尝试一个座位，都进行**安全检查** (`isValid`)，确保不会和楼上已入座的皇后冲突。
4.  如果安全，就**做出选择**（让皇后先坐下），然后**向下探索**（把安排下一行的任务委托给“下属”）。
5.  当下属（递归调用）返回后，无论他成功与否，都**撤销选择**（让皇后站起来），以便在当前行尝试下一个位置。

这个“**前进-碰壁-退回-换路**”的过程，就是回溯的灵魂。

## 2. Java 代码实现（附详细注释）

### `solveNQueens` 方法 (国王)
这是程序的入口，负责初始化棋盘，并发起第一次回溯调用。
```java
public static List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    if (n <= 0) {
        return result;
    }

    // 初始化棋盘
    char[][] board = new char[n][n];
    for (int i = 0; i < n; i++) {
        // 你的注释：调用 Arrays 工具箱的 fill 方法，
        // 把字符 . 填充到 board 棋盘的第 i 行的每一个格子里
        Arrays.fill(board[i], '.');
    }

    // 你的注释：下达第一个总命令：“来，从第 0 行开始，帮我找出所有解法！”
    backtrack(board, 0, result);
    return result;
}
````

### `backtrack` 方法 (大臣)

这是递归和回溯的核心，是那个不断“尝试、探索、反悔”的棋手。

```java
private static void backtrack(char[][] board, int row, List<List<String>> result) {
    // 终止条件：如果成功安排好了最后一行，row 会等于 n
    if (row == board.length) {
        // 你的注释：我们找到了一个完整的解！拍照存档！
        result.add(constructSolution(board));
        return;
    }

    // 尝试在当前行的每一列放置皇后
    for (int col = 0; col < board.length; col++) {
        // 进行安全检查
        if (isValid(board, row, col)) {
            // 1. 做出选择：让皇后在 (row, col) 落座
            board[row][col] = 'Q';

            // 2. 向下探索：把安排下一行 (row + 1) 的任务交给“下属”
            backtrack(board, row + 1, result);

            // 3. 撤销选择：让皇后站起来，恢复现场，以便尝试当前行的下一个位置
            // 你的注释：这是回溯的灵魂，保证了我们能不遗漏地探索所有可能性。
            board[row][col] = '.';
        }
    }
}
```

### `isValid` 方法 (安全检查官)

> **你的理解**: 从当前点 `(row, col)`，沿着**正上方、左上方、右上方**三个方向回溯检查，因为下方的行都还是空的。

```java
private static boolean isValid(char[][] board, int row, int col) {
    // 检查同一列
    for (int i = 0; i < row; i++) {
        if (board[i][col] == 'Q') return false;
    }

    // 检查左上对角线
    for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
        if (board[i][j] == 'Q') return false;
    }

    // 检查右上对角线
    for (int i = row - 1, j = col + 1; i >= 0 && j < board.length; i--, j++) {
        if (board[i][j] == 'Q') return false;
    }
    
    return true;
}
```

> **你的记忆方法（三步思考法）**：
>
> 1.  **确定“起点”**：检查点永远是 `(row - 1, ...)`。左上是 `col - 1`，右上是 `col + 1`。
> 2.  **确定“方向”**：左上是 `i--, j--`；右上是 `i--, j++`。
> 3.  **确定“边界”**：向上 `i >= 0`；向左 `j >= 0`；向右 `j < n`。

### `constructSolution` 方法 (记录员)

这个辅助方法负责将 `char[][]` 棋盘的当前“快照”，转换成题目要求的 `List<String>` 格式。

```java
private static List<String> constructSolution(char[][] board) {
    List<String> solution = new ArrayList<>();
    for (char[] row : board) {
        solution.add(new String(row));
    }
    return solution;
}
```

## 3\. `n=4` 时的推演过程

> **你的推演过程**：
>
> 1.  **首席大臣 (`row=0`)** 尝试 `(0, 0)`，落子 `Q`，并派**二号大臣**去处理 `row=1`。
> 2.  **二号大臣 (`row=1`)** 尝试 `(1, 0)` (失败)，`(1, 1)` (失败)，最终在 `(1, 2)` 成功落子 `Q`，并派**三号大臣**去处理 `row=2`。
> 3.  **三号大臣 (`row=2`)** 尝试了所有列，发现都无法安全落子，走入了**死胡同**。他任务失败，返回。
> 4.  **第一次回溯**：**二号大臣**得知 `(1, 2)` 这个选择会导致死胡同。他**撤销**了 `(1, 2)` 的棋子，将其变回 `.`，然后继续自己的循环，尝试下一个位置 `(1, 3)`...
> 5.  这个过程不断地“**前进-碰壁-退回-换路**”，直到最终有一个分支能成功走到底（`row == 4`），这时就找到了一个解。然后继续回溯，寻找所有其他的解。

## 4\. 总结与沉淀卡片

-----

### 📝 算法沉淀卡片：N皇后问题 (回溯法)

  - **核心思想**: **回溯法 (Backtracking)**，一种“聪明的”深度优先搜索 (DFS)。通过“**选择-探索-撤销**”的范式，系统性地遍历解空间树，找到所有符合条件的解。

  - **关键技巧**:

    1.  **递归框架**: 使用递归函数 `backtrack(状态)` 来代表探索过程。函数参数（如 `row`）用来定义当前所处的决策阶段。
    2.  **约束与剪枝**: `isValid` 函数是回溯算法的性能关键。它能在探索的早期就判断出当前路径是否可行，从而“剪掉”大量无效的搜索分支。
    3.  **选择的撤销**: 在递归调用返回后，必须将状态（棋盘）恢复到做出选择之前的样子 (`board[row][col] = '.';`)，以便在当前层级尝试其他选择。

  - **复杂度分析**: (`N` = 皇后数量)

      - **时间复杂度**: 约为 $O(N\!)$。虽然有剪枝，但本质上仍然是指数级的，因为需要探索大量的可能性。
      - **空间复杂度**: $O(N)$ 或 $O(N^2)$。递归深度最多为 `N`，所以栈空间是 $O(N)$。棋盘本身需要 $O(N^2)$ 的空间。

  - **同类题型**:

      - [LeetCode 46. 全排列](https://leetcode.cn/problems/permutations/)
      - [LeetCode 78. 子集](https://leetcode.cn/problems/subsets/)
      - [LeetCode 39. 组合总和](https://leetcode.cn/problems/combination-sum/)

-----

```
```
