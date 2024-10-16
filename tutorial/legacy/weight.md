# 「NOIP 2014」联合权值

**排版不咋地，凑合看罢——2022.5.2注**

# 暴力做法


  这道题的标签十分诡异，尽管洛谷标了树形DP和LCA，但最终我的做法只能说勉强可以说是树形DP。
 	
  首先读题，n 个点 n-1 条边的图明显是一棵树，由于题目没有给出树根，所以我们默认用1号节点作为树根。
 	
  我们可以先试着用搜索暴力探索题目的解，这里很多人喜欢用深搜，但我认为对这道题来说用广搜显然更为合适。为什么？且看分析：

​	这道题需要求出**距离为2**的点，这类点在树中可分为两类：

  - 爷爷节点与孙子节点之间的距离为2
  - 同一个节点之间的距离为2

​	也就是说，我们需要知道**一个节点的所有孩子**以及它的父亲节点。后者维护一个数组即可实现，而前者相比于一搜到底的深搜，显然逐层扩展的广搜记录起来更为方便。

​	另外在写代码时还应注意样例给出的信息：分析样例可以发现**这道题将2个点的不同顺序组合视为不同情况**。

**但是样例很坑的一点是它只有一条链，所以代码不太好测。**

​	至此，一个搜索代码就成形了：

```cpp
#include <cstdio>
#include <queue>
#include <vector>
using namespace std;

const int MAXN = 4e5 + 10, MOD = 10007;//因为是双向建边所以要开两倍
int head[MAXN], nxt[MAXN], to[MAXN];
int tot;
int we[MAXN], fa[MAXN];
bool v[MAXN];
int sumw, maxw;
queue<int> q;

int max(const int &a, const int &b)
{
    return (a > b) ? a : b;
}

void add(int x, int y)
{
    to[++tot] = y;
    nxt[tot] = head[x];
    head[x] = tot;
}

void bfs(int start)
{
    q.push(start);
    while (!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = true;
        vector<int> son;//记录当前节点的所有孩子的编号
        for (int i = head[x]; i; i = nxt[i]) {
            int y = to[i];
            if (!v[y]) {
                q.push(y);
                fa[y] = x;
                //先处理爷爷节点与孙子节点间的情况
                sumw += (we[y] * we[fa[x]]) * 2;
                maxw = max(maxw, we[y] * we[fa[x]]);
                sumw %= MOD;
                
		        //注意变量更新的前后顺序
                for (int i = 0; i < son.size(); i++) {
                        int k = son[i];
                	sumw += (we[k] * we[y]) * 2;
                	maxw = max(maxw, (we[k] * we[y]));
                	sumw %= MOD;
                }
                son.push_back(y);
            }
        }
    }
}

int main()
{
    int n;
    scanf("%d", &n);
    for (int i = 1; i <= n - 1; i++) {
        int x, y;
        scanf("%d%d", &x, &y);
        add(x, y);
        add(y, x);
    }
    for (int i = 1; i <= n; i++) {
        scanf("%d", &we[i]);
    }

    bfs(1);

    printf("%d %d", maxw, sumw);
    return 0;
}

```

​	这样的代码在洛谷上可以拿到70分。

# 优化

​	这之后我就在这里卡了很久，因为翻洛谷题解也是数论，遍历，深搜（树形DP）等，我就没怎么细看。直到当时听了一半 AcWing 讲解的 [@Star](http://oj.hyoicloud.cn/user/176) 
 看到我的代码后说出了三个字：**前缀和**

​	当时就茅厕顿开 ~~（雾）~~，依旧来分析一下，我们用于计算一个节点的所有孩子间的联合权值的代码使用了一个 for 循环，导致代码十分的低效，先回顾下计算过程。

$$
Sumw\ \gets Sumw + Son[0]\ \times w[y]\ \times 2  \\
$$
$$
Sumw\ \gets Sumw + Son[1]\ \times w[y]\ \times 2  \
$$
$$
Sumw\ \gets Sumw + Son[2]\ \times w[y]\ \times 2  \
$$
$$
\dots
$$

​	我们可以使用乘法分配律使其更为清晰：

$$
Sumw \ \gets Sumw + (\ Son[0]\ + \ Son[1]\ +\ Son[2]\ +\ ...)\ \times \ w[y]\ \times \ 2
$$

​	答案此时已经呼之欲出了：**前缀和优化**。

​	我们可以在遍历一个节点的子节点时动态维护一个变量存储前缀和（即当前节点前面的节点的权值之和），这样在计算所有子节点的联合权值之和时可以直接拿它相乘。由于我们还需要求出最大的联合权值，因此需要另一个变量存储前面节点权值的最大值，vector数组就不需要了。

```cpp
#include <cstdio>
#include <queue>
using namespace std;

const int MAXN = 4e5 + 10, MOD = 10007;//因为是双向建边所以要开两倍
int head[MAXN], nxt[MAXN], to[MAXN];
int tot;
int we[MAXN], fa[MAXN];
bool v[MAXN];
int sumw, maxw;
queue<int> q;

int max(const int &a, const int &b)
{
    return (a > b) ? a : b;
}

void add(int x, int y)
{
    to[++tot] = y;
    nxt[tot] = head[x];
    head[x] = tot;
}

void bfs(int start)
{
    q.push(start);
    while (!q.empty()) {
        int s = 0, rmax = 0;//s：权值和，rmax：权值最大值
        int x = q.front();
        q.pop();
        v[x] = true;
        for (int i = head[x]; i; i = nxt[i]) {
            int y = to[i];
            if (!v[y]) {
                q.push(y);
                fa[y] = x;
                sumw += (we[y] * we[fa[x]]) * 2;
                maxw = max(maxw, we[y] * we[fa[x]]);
                sumw %= MOD;//防止爆int

                sumw += (s * we[y]) * 2;
                maxw = max(maxw, rmax * we[y]);
                sumw %= MOD;

                //更新变量
                s += we[y];
                rmax = max(rmax, we[y]);
                s %= MOD;
            }
        }
    }
}

int main()
{
    int n;
    scanf("%d", &n);
    for (int i = 1; i <= n - 1; i++) {
        int x, y;
        scanf("%d%d", &x, &y);
        add(x, y);
        add(y, x);
    }
    for (int i = 1; i <= n; i++) {
        scanf("%d", &we[i]);
    }

    bfs(1);

    printf("%d %d", maxw, sumw);
    return 0;
}
```

​	最后关于标签，其实可以把 sumw 与 maxw 两个变量当做状态 ~~（虽然也没见过用广搜的树形DP~~

​	这道题遗憾的是没能自己想出来，率先做出这道题的是 [@Star](http://oj.hyoicloud.cn/user/176) ，可以去看他的[深搜树形DP做法题解](http://oj.hyoicloud.cn/article/95)