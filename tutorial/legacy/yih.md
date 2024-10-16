# 「AcWing 2022 CSP-J 模拟赛」Yih 的线段树

[AcWing?](https://www.acwing.com/solution/content/132477/)

update on 8.15：感谢 [@long_hao](http://www.hifumi.cn) 指出文章中存在大量的 Yuzu 数写成 Fuzu 数的问题（好丢脸），另外完善了关于 $1$ 的处理，并去掉了代码中不必要的 `long long`。

推荐去看他的[超简洁题解](http://www.hifumi.cn/index.php/2022/08/14/acwing_csp-j2022_mo_ni_sai_4582_yih_de_xian_duan_shu/)。

## 前言

这是一篇没有使用二进制推导，而是根据线段树的形态总结规律的题解。

大家可能会觉得这个公式不够简洁，如果抱有这样想法也没关系，可以将这篇题解作为另一种思路的参考。

题意略。

## 思考过程

这道题刚拿到时可能没什么思路，还可能会想到奇偶性判断（然而结果是罚时），我们先暴力推出 $ n \leq 300$ 的 Yuzu 数，并画出前几个数的线段树。

在画线段树时可以用[辅助网站](https://csacademy.com/app/graph_editor/)，把 `build()` 代码复制下来略加修改即可。

```cpp
void build(int x, int l, int r) {
    if(l == r) return;
    int mid = (l + r) >> 1;
    printf("%d %d\n", x, x << 1);
    build(x << 1, l, mid);
    printf("%d %d\n", x, x << 1 | 1);
    build(x << 1 | 1, mid + 1, r);
}
```

这里贴出部分 $ n $ 较小时的线段树。

![tree.png](https://s2.loli.net/2022/08/14/AmCgY8EcKW5yfGq.png)

![图片.png](https://s2.loli.net/2022/08/14/yavugfLQ9lWYTZD.png)

![图片.png](https://s2.loli.net/2022/08/14/oLIhc3tM2zlO51f.png)

![图片.png](https://s2.loli.net/2022/08/14/oB8SOkG73WTbZs5.png)

![图片.png](https://s2.loli.net/2022/08/14/lDwIZe5WugNJ6i7.png)

不难发现：

1. 线段树的节点总是成对增加，而且总是先递归左子树，再递归右子树。
2. Yuzu 数总是线段树最后一层最右侧的节点编号，而且总是等于将线段树补全成完全二叉树时的节点编号。

第二条显然成立。第一条可以根据线段树原理与代码归纳。如 4 -> 5 与 5 -> 6 时的线段树形态变化，先在最左侧的 4 号节点增加了两个叶子，又在根节点右子树最左侧的 12 号节点增加了两个叶子。通过代码可以总结。

从这里我们也可以看出，$11$ 这个数并不是 Yuzu 数，因此“奇数是 Yuzu 数”的假设不成立。

由上面提到的性质，并举几个例子可以看出，**线段树复制自己作为右子树，自身作为左子树，并加上一个父节点链接，结果仍是一个线段树**。为了方便，后文我们将这个操作形成的线段树简称为生成线段树。

这是很重要的性质，我们将根据它推导 Yuzu 数之间互相推出的公式。

## 公式推导

假设我们已知一个线段树，它的 Yuzu 数为 $x$。很明显它的生成线段树的 Yuzu 数是左子树补全成满二叉树时节点数量与 $x$ 的和再加上父节点时的数量。

设 Yuzu 数为 $x$ 的线段树补全成满二叉树后的节点数为 $\operatorname{count}(x)$，生成线段树的 Yuzu 数为 $\operatorname{next}(x)$。我们会发现求 $\operatorname{count}(x)$ 可以利用树的深度实现。于是定义 $\operatorname{dep}(x)$ 为 Yuzu 数为 $x$ 的线段树的深度，并将 $\operatorname{count}(x)$ 的定义改为深度为 $x$ 的满二叉树的节点数量。

不难得到以下公式：

1. $\operatorname{dep}(x) = \lfloor \log_2x \rfloor + 1$
2. $\operatorname{count}(x) = 2^x - 1 $

则生成二叉树的左子树节点数即为 $\operatorname{count}(x)$，右子树节点数为 $x$，再加上父节点，由此可以得到以下公式。

$$
\operatorname{next}(x) = \operatorname{count}(\operatorname{dep}(x)) + x + 1
$$

至此，我们完美解决这道题，只需将暴力求出的 Yuzu 数前几项拿来递推就可以了……嘛？

写出程序后把推出的 Yuzu 数和打表打出的对比，发现有很多缺失。比如在把 $1, 3, 5, 7, 9$ 拿来递推时第一个出现的缺失为 $17$。我们试着建出这个线段树。

![图片.png](https://s2.loli.net/2022/08/14/G8qWA7p2XkMalgU.png)

你发现了什么？

在最左侧建立节点的线段树是不能被复制得出的。因此它的 Yuzu 数也无法由别的 Yuzu 数推出，不过这个线段树的 Yuzu 数明显很好求，根据深度求即可。

对应到 Yuzu 数上就是**对于所有的 $x = \operatorname{count}(i) + 2 \ \ (i \in \mathbb{N^*})$，$x$ 一定是 Yuzu 数且不能被其它 Yuzu 数推出**。

于是我们在代码中预处理出 $[1, 10^9]$ 之间无法被推出的 Yuzu 数，并用它们推出其余的 Yuzu 数，预处理完毕后即可 $\operatorname{O}(1)$ 回答询问。

另外，$1$ 也是 Yuzu 数，但是既不能被推出，也不能被直接求出。

值得注意的是，我在代码实现中判断并不是特别严密，可能会有一些推出的 Yuzu 数超出题目数据范围，不过这并不影响代码正确性。

## 简要结论

定义：

- $\operatorname{dep}(x) = \lfloor\log_2x\rfloor + 1$

- $\operatorname{count}(x) = 2^x - 1$

- $\operatorname{next}(x) = \operatorname{count}(\operatorname{dep}(x)) + x + 1$

则有：

1. 如果 $x$ 是 Yuzu 数，则 $\operatorname{next}(x)$ 一定也是 Yuzu 数。
2. 所有的 $x = \operatorname{count}(i) + 2 \ (i \in \mathbb{N^*})$ 是 Yuzu 数且同时不可被其他 Yuzu 数由上一条结论推出。
3. $1$ 是 Yuzu 数，且不可由前两条结论推出。

## AC Code
```cpp
#include <cstdio>
#include <cmath>
#include <queue>
#include <unordered_map>
using namespace std;

int dep(int x)
{
    return floor(log2(x)) + 1;
}

int power(int a, int b)
{
    int ans = 1, base = a;
    while (b) {
        if (b & 1) {
            ans *= base;
        }
        base *= base;
        b >>= 1;
    }
    return ans;
}

int count(int x)
{
    return power(2, x) - 1;
}

int Next(int x)
{
    return count(dep(x)) + x + 1;
}

queue<int> q;

unordered_map<int, bool> v;

int main()
{
    q.push(1);
    int a = 1, b = count(a) + 2;
    while (b <= 1e9) {
        q.push(b);
        a++;
        b = count(a) + 2;
    }
    while (!q.empty()) {
        int x = q.front();
        q.pop();
        if (v[x]) continue;
        v[x] = true;
        if (x > 1e9) {
            continue;
        }
        q.push(Next(x));
    }
    int q;
    scanf("%d", &q);
    while (q--) {
        int x;
        scanf("%d", &x);
        if (v[x]) {
            printf("Y\n");
        }
        else {
            printf("N\n");
        }
    }
    return 0;
}
```