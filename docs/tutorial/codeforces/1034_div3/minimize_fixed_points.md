# Minimize Fixed Points

## 题目大意

如果一个 $1 \sim n$ 的排列 $p$ 满足对所有 $2 \leq i \leq n$，有 $\gcd(p_i, i) > 1$，则称这个排列是好的排列。找出任意一个好的排列，使这个排列中满足 $p_i = i$ 的 $i$ 的取值数量最小。

## 解题思路

观察样例可以发现构造方案与质数有关，只有大到在排列中没有倍数的质数才会被迫保持在原地。

考虑构造，如果按筛法思路对每个质数的所有倍数都做交换，很容易会出现不合法的情况。因此需要一种不重不漏的分类方案，仅在每个类内做交换。考虑将每个数的最大质因数作为它的代表，构造时仅将所有代表相同的数进行轮换。可以证明，这种交换方案不会漏掉任何一个在排列中有倍数的质数，也不会有重复交换的情况。

实操中我采用了先筛法预处理所有质数，再暴力预处理所有数的最大质因数的值并分类的做法，性能有待优化，但胜在实现简单且数据范围较小可以通过。

## 参考代码

```cpp
#include <iostream>
using namespace std;

const int MAXN = 1e5 + 10;
bool v[MAXN];
int p[MAXN], top;

int Head[MAXN], Next[MAXN], val[MAXN], tot;

inline void add(const int &x, const int &y)
{
    val[++tot] = y;
    Next[tot] = Head[x];
    Head[x] = tot;
}

inline void Eth()
{
    int n = 1e5;
    for (int i = 2; i <= n; i++) {
        if (v[i]) continue;
        else {
            p[++top] = i;
            for (int j = i; (long long) i * j <= n; j++) {
                v[i * j] = true;
            }
        }
    }
    for (int i = n; i >= 2; i--) {
        for (int j = top; j >= 1; j--) {
            if (i % p[j] == 0) {
                add(j, i);
                break;
            }
        }
    }
}

int a[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);
    int t;
    cin >> t;
    Eth();
    while (t--) {
        int n;
        cin >> n;
        for (int i = 1; i <= n; i++) {
            a[i] = i;
        }
        for (int x = 1; x <= top && p[x] <= n; x++) {
            for (int i = Head[x], pre = Head[x]; i; i = Next[i]) {
                if (val[i] > n) break;
                swap(a[val[i]], a[val[pre]]);
                pre = i;
            }
        }
        for (int i = 1; i <= n; i++) {
            cout << a[i] << ' ';
        }
        cout << '\n';
    }
    return 0;
}
```