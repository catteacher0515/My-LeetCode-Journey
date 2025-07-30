好的，KMP 算法绝对值得一篇全面、深入的沉淀笔记。它不仅是一个高效的算法，其“预处理”和“状态机”的思想更是计算机科学的精髓。

我们这就把你代码中详尽的注释、我们之前聊过的所有细节、以及那个生动的“特工”类比，完美地融合在一起，形成一篇专属于你的、完整的 KMP 算法学习笔记。

-----

````markdown
# 字符串查找王者：KMP 算法深度沉淀

KMP 算法（Knuth-Morris-Pratt 算法）是字符串匹配领域的里程碑。它通过对“模式串”进行巧妙的预处理，生成一张“失配跳转表”（`next` 数组），然后在匹配过程中，当遇到不匹配的字符时，能够进行“智能”的、大步长的滑动，避免了文本串指针 `i` 的回溯，从而将时间复杂度从暴力匹配的 $O(m \cdot n)$ 优化到了卓越的 $O(m+n)$。

## 1. 核心思想：KMP 特工行动

理解 KMP 的最佳方式，是把它想象成一个特工故事。

-   **任务**: 你是代号为“KMP”的王牌特工，需要在一段极长的**敌方加密电报 (`text`)**中，找出一段特定的**秘密暗号 (`pattern`)**。
-   **传统困境 (朴素查找)**: 普通特工每次匹配失败，只会把暗号纸条向右挪一格，然后从头开始比对，浪费了大量已知信息。
-   **KMP 的策略 (两阶段行动)**:
    1.  **备课阶段 (`buildNext` 方法)**: 在出发前，特工把自己锁在安全屋里，不看长电报，只反复研究手里的“秘密暗号”。他分析其内部的重复结构，制作出一份精密的“**失配行动预案**”（`next` 数组）。
    2.  **执行阶段 (`kmpSearch` 方法)**: 特工带着这份“预案”潜入敌方。他的左手指 `i` 在长电报上**永不后退**地前进，右手指 `j` 在暗号上移动。一旦发生失配，他无需回退左手，只需查阅“预案”，将右手指 `j` “闪现”到一个新的、更优的位置，继续比对。

---

## 2. “备课”阶段：`buildNext` 方法详解 (制作“失配行动预案”)

这是 KMP 算法的精髓，也是最难理解的部分。它的任务就是生成那张“失配快速跳转表” `next`。

**`next[i]` 的含义**：代表在 `pattern` 的子串 `pattern[0...i]` 中，“**最长的、相等的、不重叠的前后缀**”的长度。这是对模式串自我重复性的量化。

### 代码实现
```java
// 构建部分匹配表（next数组）
private static int[] buildNext(String pattern) {
    int m = pattern.length();
    int[] next = new int[m];
    if (m == 0) return next;
    next[0] = 0; // 第一个字符的最长相等前后缀长度为0

    // i 是“后缀”的末尾指针，j 是“前缀”的末尾指针
    for (int i = 1, j = 0; i < m; i++) {
        // 当前字符不匹配，根据预案回退 j
        while (j > 0 && pattern.charAt(i) != pattern.charAt(j)) {
            j = next[j - 1];
        }

        // 当前字符匹配，j 向前移动
        if (pattern.charAt(i) == pattern.charAt(j)) {
            j++;
        }

        // 记录当前位置 i 的最长相等前后缀长度
        next[i] = j;
    }
    return next;
}
````

-----

## 3\. “上课”阶段：`kmpSearch` 方法详解 (执行任务)

这个方法负责带着“预案”去敌方电报中进行实际的匹配。

### 代码实现与你的注释解析

```java
public static int kmpSearch(String text, String pattern) {
    // --- 边界检查 ---
    if (pattern == null || pattern.length() == 0) {
        // 你的注释：如果子串是空的,那么任意位置都可以,那么我们直接返回0也是可以的
        return 0;
    }
    if (text == null || text.length() < pattern.length()) {
        // 你的注释：如果母串是空的,或者母串的长度是小于子串的,那么直接就没有结果,返回-1
        return -1;
    }
    
    int n = text.length();
    int m = pattern.length();

    // --- 准备工作 ---
    int[] next = buildNext(pattern);
    // 你的注释：获取了之前精心制作的 next 数组，这是我们高效行动的关键

    // --- 开始匹配 ---
    for (int i = 0, j = 0; i < n; i++) {
        /*
         * 你的注释：我们派出了两位“侦察兵”，或者说用两根手指在地图上移动：
         * i (文本指针): 一根手指 i，负责在长电报 text 上移动。它的规则是：勇往直前，永不后退。
         * j (模式指针): 另一根手指 j，负责在我们的暗号纸条 pattern 上移动。它的移动会比较“跳跃”。
         */

        // 1. 处理失配：根据 next 数组回退 j
        while (j > 0 && text.charAt(i) != pattern.charAt(j)) {
            j = next[j - 1];
            /*
             * 你的注释：
             * > “智能滑动”: 我们不对 i（电报指针）做任何操作，而是只回退 j（暗号指针）。
             * > 查阅预案: 我们查阅 next[j-1] 条款。这个条款告诉我们：你刚刚匹配成功的部分（长度为j），
             * > 其中有一段长度为 next[j-1] 的前缀，可以完美地对齐到这段成功部分的末尾。
             * > 执行回退: 于是，我们直接把 j 指针“闪现”回 next[j-1] 这个位置。
             */
        }

        // 2. 处理匹配：j 向前移动
        if (text.charAt(i) == pattern.charAt(j)) {
            j++;
        }

        // 3. 判断是否完全匹配
        if (j == m) {
            /*
             * 你的注释：
             * > i 是我们在电报中匹配成功的最后一个字符的下标。
             * > m 是暗号的长度。
             * > 所以，匹配成功的起始位置，自然就是 i - m + 1。
             */
            return i - m + 1;
        }
    }

    return -1; // 未找到匹配
}
```

## 4\. 进阶任务：`kmpSearchAll` 方法详解 (寻找所有匹配)

这个“终极版”方法的绝大部分代码与 `kmpSearch` 完全相同，唯一的区别在于找到匹配后的行动。

### 代码实现

```java
public static List<Integer> kmpSearchAll(String text, String pattern) {
    // ... 边界检查和 next 数组构建与 kmpSearch 相同 ...
    List<Integer> positions = new ArrayList<>();
    int n = text.length();
    int m = pattern.length();
    int[] next = buildNext(pattern);

    for (int i = 0, j = 0; i < n; i++) {
        // 失配逻辑完全相同
        while (j > 0 && text.charAt(i) != pattern.charAt(j)) {
            j = next[j - 1];
        }

        // 匹配逻辑完全相同
        if (text.charAt(i) == pattern.charAt(j)) {
            j++;
        }

        // 【核心区别】
        if (j == m) {
            // 动作一：记录成果
            positions.add(i - m + 1);
            
            // 动作二：智能重启
            j = next[j - 1];
            /*
             * 你的注释：
             * > 这行代码，就是特工在查阅他的“行动预案”的最后一条。
             * > 预案告诉他：“任务完成后，你的暗号指针 j 不需要归零，可以直接‘闪现’到下标 next[m-1] 的位置，
             * > 然后随着 i 的下一次前进，继续寻找下一个可能的匹配！”
             */
        }
    }
    return positions;
}
```

**“智能重启”的本质**：利用 `next[m-1]` 记录的、整个模式串的“最长相等前后缀”信息，让 `j` 指针回退到一个最优的位置，以继续寻找可能存在的、与当前匹配项**重叠**的下一个匹配项。

-----

## 5\. KMP 算法沉淀卡片

### 📝 算法沉淀卡片：KMP 字符串匹配

  - **核心思想**: 通过对**模式串 `pattern`** 进行预处理，生成一张 `next` 跳转表，然后在匹配过程中遇到失配时，利用 `next` 表进行“智能滑动”，避免了文本串指针 `i` 的回溯，从而实现线性时间复杂度。

  - **关键技巧**: **`next` 数组的构建与应用**。

    1.  **构建 (`buildNext`)**: `next[i]` 存储的是 `pattern[0...i]` 的最长相等前后缀的长度。这是“模式串”的自我分析。
    2.  **使用 (`kmpSearch`)**: 当 `text[i]` 和 `pattern[j]` 失配时，`i` 指针不回退，`j` 指针根据 `next[j-1]` 的值进行“智能跳转”，从而跳过无效的比较。

  - **复杂度分析**: (`n` = 文本串长度, `m` = 模式串长度)

      - **时间复杂度**: $O(n + m)$ (构建 `next` 数组 $O(m)$，搜索过程 $O(n)$)
      - **空间复杂度**: $O(m)$ (用于存储 `next` 数组)

  - **同类题型**:

      - [LeetCode 28. 找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/): KMP 的直接应用。
      - [LeetCode 1392. 最长快乐前缀](https://leetcode.cn/problems/longest-happy-prefix/): 题意就是让你实现 `buildNext` 方法，并返回 `next[m-1]`。
      - [LeetCode 459. 重复的子字符串](https://leetcode.cn/problems/repeated-substring-pattern/): 可以通过 KMP 的 `next` 数组的性质来巧妙求解。

<!-- end list -->

```
```
