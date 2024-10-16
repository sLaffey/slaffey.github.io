# 「NOI 2015」软件包管理器

&emsp;&emsp;复习数据结构，从树剖开始罢。

&emsp;&emsp;因为 Hashnode 不允许标题全大写，就加了个后缀。

### Solution

&emsp;&emsp;一道板子题，不过有几个需要注意的点。

1. 线段树维护每个区间中已安装软件包的数量，树上修改操作直接修改节点的值即可，不用累加处理。
2. 安装时直接修改根节点到这个节点的链，卸载时直接对子树进行修改。
3. 答案可以由修改前和修改后的根节点的值相减取绝对值得到。
4. 编号最好改为从 $1$ 开始。

### Code

```cpp
#include <cstdio>
using namespace std;

inline void swap(int &a, int &b) { int t = a; a = b, b = t; }

const int MAXN = 1e5 + 10;

int Head[MAXN], Next[MAXN], to[MAXN], tot;

inline void add(const int &x, const int &y)
{
    to[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
}

int dep[MAXN];
int Size[MAXN], son[MAXN], top[MAXN];
int fa[MAXN];
int id[MAXN], id_top;

void dfs_size(int x)
{
    Size[x] = 1;
    int maxp = 0;
    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];

        fa[y] = x;
        dep[y] = dep[x] + 1;

        dfs_size(y);

        Size[x] += Size[y];
        if (Size[y] > Size[maxp]) {
            maxp = y;
        }
    }

    son[x] = maxp;
}

void dfs_top(int x, int v)
{
    id[x] = ++id_top;
    top[x] = v;

    if (son[x]) dfs_top(son[x], v);
    else return;

    for (int i = Head[x]; i; i = Next[i]) {
        int y = to[i];
        if (y == son[x]) continue;

        dfs_top(y, y);
    }
}

struct stree {
    int val;
    int add;
    int l, r;
} tree[MAXN << 2];

#define self tree[p]
#define ls (p << 1)
#define rs (p << 1 | 1)
#define lson tree[ls]
#define rson tree[rs]

void st_build(int p, int l, int r)
{
    self.l = l, self.r = r;
    if (l == r) {
        return;
    }

    int mid = l + r >> 1;
    st_build(ls, l, mid);
    st_build(rs, mid + 1, r);

    return;
}

inline void pushdown(int p)
{
    if (self.add != -1) {
        lson.add = self.add;
        lson.val = (lson.r - lson.l + 1) * self.add;

        rson.add = self.add;
        rson.val = (rson.r - rson.l + 1) * self.add;

        self.add = -1;
    }
}

void st_modify(int p, int l, int r, bool v)
{
    if (l <= self.l && self.r <= r) {
        self.add = v;
        self.val = v ? self.r - self.l + 1 : 0;
        return;
    }

    pushdown(p);
    if (l <= lson.r) st_modify(ls, l, r, v);
    if (r >= rson.l) st_modify(rs, l, r, v);

    self.val = lson.val + rson.val;

    return;
}

void modify(int x, int y)
{
    while (top[x] != top[y]) {
        if (dep[top[x]] < dep[top[y]]) {
            swap(x, y);
        }
        st_modify(1, id[top[x]], id[x], 1);
        x = fa[top[x]];
    }
    if (dep[x] > dep[y]) {
        swap(x, y);
    }
    st_modify(1, id[x], id[y], 1);
}

void modify(int x)
{
    st_modify(1, id[x], id[x] + Size[x] - 1, 0);
}

int install(int x)
{
    int ans = tree[1].val;
    modify(1, x);
    return tree[1].val - ans;
}

int uninstall(int x)
{
    int ans = tree[1].val;
    modify(x);
    return ans - tree[1].val;
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    int n;
    scanf("%d", &n);
    for (int i = 2; i <= n; i++) {
        int fa;
        scanf("%d", &fa);
        add(fa + 1, i);
    }

    dfs_size(1);
    dfs_top(1, 1);
    st_build(1, 1, n);

    int q;
    scanf("%d", &q);
    while (q--) {
        static char op[15];
        int x;
        scanf("%s%d", op, &x);
        x++;

        switch (op[0]) {
            case 'i':
                printf("%d\n", install(x));
                break;
            case 'u':
                printf("%d\n", uninstall(x));
                break;
            default:
                printf("nen hen hen aaaaaaaaaaaaaaaaaaaaa\n");
                return 0;
        }
    }

    return 0;
}
```