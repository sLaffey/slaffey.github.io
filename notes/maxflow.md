# 网络流简介

网络流定义在一张有向图 $G = (V, E)$ 上，位于 $G$ 中的每一条边都有一个属性「容量」记作 $c(u, v)$，同时取两个点 $s, t$ 作为网络流的「源点」与「汇点」，于是我们想象一股水流由源点流入汇点，这就是网络流的基本模型。

我们规定每条边上的「流」不得超过其容量大小，且每个结点入流量一定等于出流量，特别地，$s, t$ 不受此条限制。对于不在 $E$ 中的边，可以直接令 $c(u, v) = 0$。

形式化地，流被定义为一个从边集 $E$ 到整数集或实数集的函数，并满足容量限制与流量守恒，即上述性质。

- 容量限制：$\forall u, v \in V, 0 \leq f(u, v) \leq c(u, v)$。
- 流量守恒：$\forall u \in V, f(u) = \sum \limits_{v \in V} f(u, v) - \sum \limits_{v \in V} f(v, u) = 0 \ \text{if} \ u \neq s, t$。

其中 $f(u)$ 被称作一个点的「净流量」。

对于网络 $G = (V, E)$ 和其上的流 $f$，我们定义 $f$ 的流量 $|f|$ 为点 $s$ 的净流量 $f(s)$，显然地，$|f| = -f(t)$。

最大流问题，即是研究在这样一个给定的图上，所有流的最大流量。

# 最大流问题的求解

## Ford-Fulkerson 增广

### 概述

Ford-Fulkerson 增广是一系列算法思想的总称，采用 Ford-Fulkerson 增广的算法基于贪心的思想，在图中不断寻找增广路径以求解最大流。

增广路径（Augmenting Path）指满足从 $s$ 到 $t$，且其上流量仍小于容量的路径。在 Ford-Fulkerson 增广算法中，我们对于每一条增广路径，都将其上流量增加到最大，这样即可得到最大流量。

值得指出的是，增广路径中可能包含反向的边。此时我们对路径进行增广时，应当从该反向边上已有流量中减去一部分。这一操作称为「退流」或「抵消」（cancellation）。

为了实现访问反向边的操作，我们对于每一条有向边都双向建边，并约定二者流量互为相反数。

显然在寻找增广路径时，我们只需考虑还可以继续增广，即还有「剩余容量」的边。为此我们定义剩余容量 $c_f(u, v) = c(u, v) - f(u, v)$。对于反向边，方便起见，我们也定义其剩余容量 $ c_f(v, u) = f(u, v) $，即还可以容纳的反向流量。

对于图 $G$，我们定义所有剩余容量大于 $0$ 的边组成的子图为「残量网络」$G_f$。形式化地，$G_f = (V, E_f)$，其中 $E_f = \{  (u, v) \ | \ c_f(u, v) > 0  \} $。

这样，我们完成了 Ford-Fulkerson 增广的基本框架，在设计的算法中，我们只需要在残量网络中不断寻找增广路径，并对其进行增广，当算法完成时，我们就得到了一个最大流。

Ford-Fulkerson 增广的正确性可以用反证法容易地证明，不过更主流的做法是将其放在最大流-最小割定理中一同证明。

### 时间复杂度分析

我们先来分析一下 Ford-Fulkerson 增广的复杂度上界。

我们假设所有流量与容量均为整数，那么显然增广总轮数不应超过 $|f|$，而单次增广复杂度为 $O(|E|)$，则总的复杂度为 $O(|E||f|)$。

## 正确性与最大流-最小割定理

首先引入「割」的定义，如果有 $\{ S, T \}$ 是 $V$ 的划分（即 $S \cup T = V  \wedge S \cap T = \varnothing $），且有 $s \in S, t \in T $，则称 $\{S, T\}$ 是 $V$ 的一个 $s-t$ 割。

我们定义割的容量 $c(S, T)$ 为所有由 $S$ 指向 $T$ 的边的容量之和，即 $c(S, T) = \sum \limits_{u \in S, v \in T} c(u, v)$，也记作 $||S, T||$。所有割中容量最小的称为最小割。

### 流值引理（Flow-value Lemma）

对于任意割与任意流，流通过割的流量即为流的流量。

#### 证明

利用数学归纳法，通过对割的一方的大小归纳来证明。

不妨设归纳奠基情况为 $S = \{s\}$，此时流值引理显然成立。

我们将任意一点由 $T$ 移向 $S$，借由流量守恒，不难证明定理仍然成立。

### 弱对偶性（Weak Duality）

对任意流与任意割，总有 $|f| \leq ||S, T||$。

#### 证明

由容量限制及流值引理，我们不难得到 $|f| = \sum \limits_{u \in S, v \in T}f(u, v) \leq \sum \limits_{u \in S, v \in T}c(u, v) = ||S, T||$。

### 最大流-最小割定理（Maxflow-Mincut Theorem）

最大流-最小割定理：最大流的流量等于最小割的容量。

最大流-最小割定理可以与 Ford-Fulkerson 增广的正确性一同证明。

#### 增广路定理（Ford-Fulkerson 增广正确性）

一个流是最大流当且仅当图中没有增广路径。

#### 证明

我们采用一个逻辑技巧来证明，通过证明以下三个命题等价成立：

对于任意流 $f$：

1. 存在容量与 $f$ 流量相等的割。
2. $f$ 是一个最大流。
3. 对于流 $f$，图上不存在增广路径。

##### Pr. 1 -> 2

由弱对偶性，不妨设该割为 $\{S, T\}$，则有 $|f| = ||S, T|| \geq |f'|$，其中 $f'$ 为任意流。所以 $f$ 是一个最大流。

##### Pr. 2 -> 3

不妨考虑反证，即证明 $\sim 3. \Rightarrow \sim 2.$。

若图上仍有增广路径，则显然该流仍有增长流量的空间，不是最大流。

##### Pr. 3 -> 1

取一个割 $\{S, T\}$，其中 $S$ 是通过非满流边与 $s$ 连接的点的集合，令 $s \in S$，则由于不存在增广路径，有 $t \in T$。

则由定义，这个割的容量就与通过割的流量相等，再由流值引理，也与流量相等。

综上所述，我们证明了一定存在一个与最大流相等的割，由弱对偶性不难证明该割为最小割，于是最大流-最小割定理得证。同时我们也证明了增广路定理，由此 Ford-Fulkerson 增广的正确性也得以证明。

## 最大流算法的两种实现

### Edmonds-Karp 算法

#### 算法流程

证明正确性之后，我们的问题就转变成了如何在残量网络 $G_f$ 中寻找增广路。一个比较朴素的想法是，直接通过 BFS 寻找，基于这种想法的最大流求解算法即为 Edmonds-Karp 算法。

具体地，我们在 $G_f$ 上 BFS，如果能从 $s$ 出发到达 $t$，说明找到了一条增广路。此时我们将该条路径上所有正向边加上所能增广的最大流量，并对其反边减去对应容量。不断重复这一过程直到不存在增广路径。

#### 时间复杂度分析

##### 最短路非递减引理

最短路非递减引理是说，每轮增广后，残量网络上任意节点到源点的最短路径长度一定不减。

考虑反证，我们不妨设 $v$ 是某轮增广后最短路递减的点中最短路长度最小的点，同时记 $d_{f}(v)$ 为增广前最短路长度，$d_{f'}(v)$ 为增广后长度，则有 $d_f(v) > d_{f'}(v)$。

在增广后的残量网络 $G_{f'}$ 上，我们记 $u$ 为 $s$ 到 $v$ 的最短路径中的上一个点，即 $d_{f'}(u) + 1 = d_{f'}(v)$。

由于 $v$ 是所有最短路长度减小的点中最短路长度最小的点，$u$ 的最短路长度就一定不减，即 $d_{f'}(u) \geq d_f(u)$。

不等式两侧同加 $1$，得到 $d_{f'}(v) \geq d_f(u) + 1$，又由反证假设，有 $d_f(v) > d_f(u) + 1$。

以下讨论本轮增广中 $(u, v)$ 的增广方向。

- 若 $(u, v) \in E_f$，由于我们采用 BFS 算法寻找增广路，于是有 $d_f(u) + 1 \geq d_f(v)$，与假设矛盾。

- 若 $(u, v) \not \in E_f$，由 $u$ 点定义于 $G_{f'}$，所以有 $(u, v) \in G_{f'}$，该边应由 $(v, u)$ 方向退流引入，所以有 $d_f(u) = d_f(v) + 1$，与假设矛盾。

综上，我们证明了最短路非递减引理。

##### 增广总轮数上界证明

我们不妨选定一条增广路上剩余容量最小的边，称作饱和边。

对于饱和边来说，本次增广会使其剩余容量归零，那么下次在这条边上进行增广时就一定是反向退流操作。

当沿 $(u, v)$ 增广时，有 $d_f(v) = d_f(u) + 1$，当沿 $(v, u)$ 增广时，有 $d_{f'}(u) = d_{f'}(v) + 1$，又由最短路非递减引理，有 $d_{f'}(v) \geq d_f(v)$，连接上述各式，得到 $d_{f'}(u) \geq d_f(u) + 2$。也就是说，每条边 $(u, v)$ 在被选为饱和边后，$u$ 到 $s$ 的距离至少增加 $2$，又由于任意结点到 $s$ 的距离最多为 $|V|$，则单条边被选作饱和边的次数为 $O(|V|)$。因此增广总轮数上界为 $O(|V||E|)$。

综上所述，Edmonds-Karp 算法的时间复杂度上界为 $O(|V||E|^2)$。

实际运行中，构造出使 Edmonds-Karp 算法达到其最坏复杂度的图较为困难，并且算法竞赛中较少考察最大流的优化，而更多考察最大流的问题建模，因此很少会有达到复杂度上界的情况。

#### 参考代码

```cpp
// Luogu P3376
#include <iostream>
#include <queue>
#include <cstring>

const int MAXN = 210, MAXM = 5010;
const long long INF = 1LL << 62;

int Head[MAXN], Next[MAXM << 1], to[MAXM << 1], tot = 1;
long long cap[MAXM << 1];

inline void add(const int &x, const int &y, const long long &c)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
    cap[tot] = c;
}

int n, m, s, t;
struct State {
    int x;
    long long min_cap;
};
int pre[MAXN];

inline long long bfs()
{
    memset(pre + 1, 0, sizeof(int) * n);
    queue<State> q;
    q.push({s, INF});
    while (!q.empty()) {
        int x = q.front().x;
        long long min_cap = q.front().min_cap;
        q.pop();

        if (x == t) {
            return min_cap;
        }

        for (int i = Head[x]; i; i = Next[i]) {
            const int &y = to[i];
            if (pre[y] || !cap[i]) continue;
            q.push({y, min(min_cap, cap[i])});
            pre[y] = i;
        }
    }
    return 0;
}

inline long long Edmonds_Karp()
{
    long long max_flow = 0;
    long long aug = bfs();
    while (aug) {
        max_flow += aug;
        int x = t;
        while (x != s) {
            cap[pre[x]] -= aug;
            cap[pre[x] ^ 1] += aug;
            x = to[pre[x] ^ 1];
        }
        aug = bfs();
    }
    return max_flow;
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin >> n >> m >> s >> t;
    for (int i = 1; i <= m; i++) {
        int x, y;
        long long c;
        cin >> x >> y >> c;
        add(x, y, c);
        add(y, x, 0);
    }
    cout << Edmonds_Karp() << endl;
    return 0;
}
```

### Dinic 算法

#### 算法流程

与 Edmonds-Karp 算法不同，Dinic 算法在增广前先对 $G_f$ 进行 BFS 分层，每次增广时仅考虑流向下一层的边。我们称 $G_f$ 去掉指向层数更小的边后的图为层次图 $G_L$。形式化地，我们称 $G_L$ 为 $G_f$ 的层次图，即 $G_L = \{(u, v) \ | \ (u, v) \in E_f \wedge d(u) + 1 = d(v)\}$。

我们在称 $G_L$ 上找到的的最大增广流 $f_b$ 为 $G_L$ 的阻塞流。

于是在执行 Dinic 算法时，我们先 BFS 出层次图 $G_L$，再 DFS 出阻塞流 $f_b$，并将阻塞流并入原先的流中，重复执行直到 BFS 无法到达汇点。此时我们就得到了最大流。

注意到 DFS 的一个缺陷：如果结点 $u$ 有大量的入边和出边，并且每次接受入边流量的时候都遍历所有出边，则这一点消耗时间复杂度最坏会达到 $O(|E|^2)$。因此，我们对每个点维护一个指针，存储这个点第一个还未满流的出边，每次只遍历该边及其以后的边，这一优化称作「当前弧优化」。

#### 时间复杂度分析

##### 单轮增广时间复杂度

可以证明，通过 DFS 进行单轮增广的时间复杂度为 $O(|V||E|)$。

对于阻塞流经过的每条增广路，它们都是在 $G_L$ 上通过一系列当前弧跳转的结果，并且跳转的次数显然不超过 $O(|V|)$。

考虑每条增广路上被清空的饱和边，我们将它们构成的集合记作 $E_1$。饱和边被清空后，由于图的分层性质，不可能会再被反向经过一次，因此有 $E_1 \subseteq E_L$。

考虑所有没有得到增广路的情形，我们将这些路径上最后一条边形成的集合称作 $E_2$，由于其不饱和，所以有 $E_1 \cap E_2 = \varnothing$，同时也有 $E_2 \subseteq E_L$。

所以有 $E_1 \cup E_2 \subseteq E_L$，也即有 $|E_1 \cup E_2| \leq |E|$，且 $E_1 \cup E_2$ 中每一个元素都代表一条 DFS 路径，又由于每条增广路跳转次数都不超过 $O(|V|)$，所以单轮增广时间复杂度为 $O(|V||E|)$。

##### 增广总轮数

注意到层次图的层数不超过 $|V|$，如果我们能证明层次图的层数在增广过程中严格单调，那么就可以证明增广总轮数不超过 $O(|V|)$。

由于 OI-Wiki 上给出的证明需要引入预流推进算法中的部分概念，而 Dinic 的原论文又暂时无法查阅。暂时从略。

#### 参考代码

```cpp
// Luogu P3376
#include <iostream>
#include <queue>
#include <cstring>
using namespace std;

const int MAXN = 210, MAXM = 5010;
const long long INF = 1LL << 62;

int Head[MAXN], Next[MAXM << 1], to[MAXM << 1], tot = 1;
int cap[MAXM << 1];

inline void add(const int &x, const int &y, const int &c)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
    cap[tot] = c;
}

int n, m, s, t;
int dep[MAXN];
int cur[MAXN];

inline bool bfs()
{
    memset(dep + 1, 0, sizeof(int) * n);
    memcpy(cur + 1, Head + 1, sizeof(int) * n);
    queue<int> q;

    q.push(s);
    dep[s] = 1;
    while (!q.empty()) {
        int x = q.front();
        q.pop();

        for (int i = Head[x]; i; i = Next[i]) {
            const int &y = to[i];
            if (cur[x] == Head[x]) cur[x] = i;
            dep[y] = dep[x] + 1;
            q.push(y);
        }
    }

    return (bool) dep[t];
}

long long dfs(const int &x, const long long &flow)
{
    if (x == t || flow == 0) return flow;

    long long res = 0;
    for (int i = cur[x]; i; i = Next[i]) {
        const int &y = to[i];
        if (dep[y] <= dep[x] || !cap[i]) continue;
        cur[x] = i;
        long long aug = dfs(y, min(flow - res, (long long) cap[i]));
        cap[i] -= aug;
        cap[i ^ 1] += aug;
        res += aug;
        if (res == flow) return res;
    }

    return res;
}

inline long long Dinic()
{
    long long max_flow = 0;
    while (bfs()) {
        max_flow += dfs(s, INF);
    }
    return max_flow;
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin >> n >> m >> s >> t;
    for (int i = 1; i <= m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
        add(b, a, 0);
    }
    cout << Dinic() << endl;
    return 0;
}
```

# 参考文献
1. [Coursera: Algorithms II](https://www.coursera.org/learn/algorithms-part2), Prof. Robert Sedgewick

2. [最大流 - OI Wiki](https://oi-wiki.org/graph/flow/max-flow)

3. 《算法导论》, Introduction to Algorithms, C.L.R.S.

