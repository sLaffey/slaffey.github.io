# KMP 算法
## 题外话

&emsp;&emsp;由于 Hashnode 要求标题不能全部大写（如果强行上传会被改成小写），而且它不识别中文，导致一些算法名称、竞赛名称等会被改成小写，看起来很不舒服。所以我以后可能会采取后加英文题目的解决方式。

## 前言

&emsp;&emsp;KMP 算法一直是笔者的噩梦。今天又翻开蓝皮看了一遍，终于是看懂了。在此写一篇博客加深印象。

## 算法应用

&emsp;&emsp;KMP 算法重点解决的是字符串匹配问题。对于这类问题，我们很容易就能打出暴力算法或者用字符串哈希在与 KMP 算法同等规模下求解。

&emsp;&emsp;但是笔者在模拟赛中被 Z 函数（扩展 KMP 算法）薄纱，因此不得不滚回来学习 KMP。

## 算法流程

&emsp;&emsp;文章以下部分默认要解决的问题为：尝试找出所有长度为 $m$ 的字符串 $B$ 在长度为 $n$ 的字符串 $A$ 中出现的位置。

### 对 $B$ 进行自身匹配

&emsp;&emsp;$next[i]$ 被定义为在字符串 $B$ 中，令 $B[1 \sim next[i]] = B[i - next[i] + 1 \sim i]$ 的最大的数，若不存在则为 $0$。

&emsp;&emsp;很明显我们可以暴力求出所有 $next[i]$，但是这样复杂度规模难以接受。我们需要考虑优化。

&emsp;&emsp;这里笔者采取类似《算法竞赛进阶指南》中的思路，将所有满足 $B[1 \sim j] = B[i - j + 1 \sim i]$ 的 $j$ 称为 $next[i]$ 的候选项。

#### 引理

&emsp;&emsp;若已知 $j_0$ 为 $next[i]$ 的候选项，则 $next[j_0]$ 为所有小于 $j_0$ 候选项中最大的一个。

#### 证明

&emsp;&emsp;以下公式较多，建议手搓几张图方便理解，笔者也会放图。

&emsp;&emsp;首先证明 $next[j_0]$ 也为 $next[i]$ 的候选项。

&emsp;&emsp;由 $next$ 定义可知 $B[1 \sim next[j_0]] = B[j_0 - next[j_0] + 1 \sim j_0]$，又由候选项定义可知 $B[1 \sim j_0] = B[i - j_0 + 1 \sim i]$，所以 $B[j_0 - next[j_0] + 1 \sim j_0] = B[i - next[j_0] + 1 \sim i]$，所以得到 $B[1 \sim next[j_0]] = B[i - next[j_0] + 1 \sim i]$。命题得证。

&emsp;&emsp;放在图中就是灰色部分相等。

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668779999639/txsBZuyj4.png align="left")

&emsp;&emsp;然后我们证明 $next[j_0]$ 为最大的小于 $j_0$ 的候选项。

&emsp;&emsp;用反证法。假设它不是最大，则一定存在候选项 $j_1$ 使 $next[j_0] < j_1 < j_0$。

&emsp;&emsp;由于 $j_0$ 是候选项，所以由定义，可知 $B[j_0 - j_1 + 1 \sim j_0] = B[i - j_1 + 1 \sim i]$。又由于 $j_1$ 是候选项，可知 $B[1 \sim j_1] = B[i - j_1 + 1 \sim i]$。因此可以推出 $B[1 \sim j_1] = B[j_0 - j_1 + 1 \sim j_0]$。发现没，这就是 $next[j_0]$ 啊！又由于 $j_1 > next[j_0]$，根据 $next$ 的定义这个显然是不成立的，所以反命题证伪，原命题成立。

&emsp;&emsp;我们还是放到图中来讲。由定义，前后两部分是相等的，因此我们取一段红色部分，显然也相等。这时发现后面的红色部分是与蓝色部分相等的。因此前面红蓝相等，即 $j_1$ 应满足 $next[j_0]$ 的条件，但 $j_1 > next[j_0]$，因此命题不成立。

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668780710698/b4hwExlv5.png align="left")

&emsp;&emsp;综上，我们证明了这个引理。

#### 优化

&emsp;&emsp;利用引理我们可以优化计算 $next$ 的过程。很明显，若 $j$ 是 $next[i - 1]$ 的候选项，那么当 $B[j + 1] = B[i]$ 时，$j + 1$ 是 $next[i]$ 的候选项。我们只要不断用 $next[i - 1], next[next[i - 1]], \dots$ 去尝试匹配即可。

&emsp;&emsp;核心代码：

```cpp
for (int i = 2, j = 0; i <= m; i++) {
    while (j > 0 && b[i] != b[j + 1]) j = Next[j];
    if (b[i] == b[j + 1]) j++;
    Next[i] = j;
}
```

### 对 $A$ 和 $B$ 进行匹配

&emsp;&emsp;定义 $f[i]$ 为 $A$ 的后缀与 $B$ 的前缀能匹配的最长长度。

&emsp;&emsp;我们可以采取和 $next$ 类似的方法求解 $f$。只需将 $A$ 和 $B$ 匹配即可。

```cpp
for (int i = 1, j = 0; i <= n; i++) {
    while (j > 0 && (j == m || a[i] != b[j + 1]) j = Next[j];
    if (a[i] == b[j + 1]) j++;
    f[i] = j;
    if (j == m) {
        cout << i << ' ';
    }
}
```