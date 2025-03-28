# Club of Young Aircraft Builders (easy version)

## 题目大意

你有一个 $n$ 个数的数列 $s_1, s_2, \ldots, s_n$，你可以进行 $m$ 次操作，每次操作任选一个数 $1 \leq i \leq n$，并令 $s_i$ 的值加一。同时给出一个数 $c$，对于所有的 $1 \leq i \leq n$，当 $\sum _{j = i}^n s_j \geq c$ 时，你将不能再对 $s_i$ 进行操作。当所有操作进行完时，你需要保证对所有 $1 \leq i \leq n$ 都有 $\sum _{j = i}^n s_j \geq c$ 成立。

你有一个操作序列 $a_1, a_2, \ldots, a_n$，其中 $a_i$ 表示第 $i$ 次操作数的下标。现在其中的某些值丢失了，丢失的操作用 $0$ 表示。你现在需要补全这个操作序列，并统计所有满足要求的操作序列个数，由于答案可能很大，你只需输出模 $10^9 + 7$ 的结果。

数据范围为 $1 \leq n \leq 100, 1 \leq c \leq 100, c \leq m \leq n \times c$。

在这道题中，所有的操作都丢失了，也就是题目保证所有 $a_i = 0$。

## 解题思路

### DP 做法

考虑对 $s_1$ 操作，显然在操作序列中，对 $s_1$ 的操作只可能在前 $c$ 次里进行，如若不然，则一定有 $\sum_{i = 1}^{n} s_i \geq c$ 成立，无法再对 $s_1$ 进行操作。因此需要在前 $c$ 次操作中任选 $k_1$ 个对 $s_1$ 执行。此时再考虑对 $s_2$ 进行操作，$s_1$ 选取的 $k$ 个操作此时可以忽略，类似地，现在需要在不包括已经进行的操作中任选 $k_2$ 个对 $s_2$ 执行。由此得到启发，可以设计状态 $dp[i, j]$ 表示对**后** $i$ 个数进行总计 $j$ 次操作的满足要求的个数。（这样比较方便转移）

于是我们可以得到状态转移方程：

$$
dp[i, j] = \sum_{k=0}^{c} \binom{c}{k} dp[i - 1, j - k]
$$

### 数学做法

我们可以发现 $s_n$ 一定会进行 $c$ 次操作。逐个向前考虑，每一个数都应决定