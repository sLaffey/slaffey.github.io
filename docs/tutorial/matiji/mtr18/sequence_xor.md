# 序列异或

## 题目大意

给定两个长度为 $n$ 的整数序列 $a, b$。定义一个数 $x$ 是异或数当且仅当存在一个 $1 \sim n$ 的排列 $\pi$，使得对所有 $1 \leq i \leq n$，有 $a_i \oplus b_{\pi_i} = x$。求出所有异或数。

数据范围：$1 \leq n \leq 2000, 0 \leq a_i, b_i \leq 2^{30}$。

## 解题思路

先来考虑 $a, b$ 序列内部分别两两不同的情况。不难发现所有可能的异或数不会超过 $n^2$ 个。若我们枚举 $a_1$ 与序列 $b$ 中所有数异或的值，则这个范围还可以缩减到 $n$ 个。由于序列不重复，可以证明对单个 $x$ 也有唯一的合法排列 $\pi$。因此只需要对每个 $a_i$ 枚举其对应的 $b_i$，得到 $n^2$ 个可能的异或数，并统计其重复数量，最后重复 $n$ 个的即为答案。

若 $a, b$ 序列内部可能出现重复，此时对单个 $x$ 的合法排列不唯一，但由于题目不要求统计重复数字，只要去重再行统计即可。可以证明，若去重后 $a, b$ 长度不一致，则一定不存在异或数。

## 参考代码

```cpp
#include <iostream>
#include <algorithm>
#include <unordered_map>
#include <vector>
using namespace std;

const int MAXN = 2e3 + 10;
int a[MAXN], ta, b[MAXN], tb, c[MAXN];
int n;
unordered_map<int, int> v;
int ans[MAXN], top;

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);
    cin >> n;
    c[0] = -1;
    for (int i = 1; i <= n; i++) {
        cin >> c[i];
    }
    sort(c + 1, c + n + 1);
    for (int i = 1; i <= n; i++) {
        if (c[i] != c[i - 1]) {
            a[++ta] = c[i];
        }
    }
    for (int i = 1; i <= n; i++) {
        cin >> c[i];
    }
    sort(c + 1, c + n + 1);
    for (int i = 1; i <= n; i++) {
        if (c[i] != c[i - 1]) {
            b[++tb] = c[i];
        }
    }
    if (ta != tb) {
        cout << 0 << endl;
        return 0;
    }
    n = ta;

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            int t = a[i] ^ b[j];
            v[t]++;
            if (i == n && v[t] == n) {
                ans[++top] = t;
            }
        }
    }
    sort(ans + 1, ans + top + 1);
    cout << top << endl;
    for (int i = 1; i <= top; i++) {
        cout << ans[i] << endl;
    }
    return 0;
}
```

> 碎碎念：最近状态奇差……三道题明明都很简单但是一直卡到比赛快结束才做出来，唉唉