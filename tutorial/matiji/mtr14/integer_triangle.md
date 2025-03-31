# 整数三角形

## 题目大意

给 $n$ 个整数点，求任取三个点计算得到的三角形面积为整数的概率，共线视为面积为 $0$。

## 解题思路

任取三个点 $A, B, C$，记 $s = \overrightarrow{AB}, t = \overrightarrow{AC}$，于是题目条件也即 $| \vec{s} \times \vec{t} \ |$ 为偶数。而 $| \vec{s} \times \vec{t} \ |$ 的奇偶性取决于三点坐标的奇偶性。根据横纵坐标的奇偶性可以将点分为四类。于是事先维护四类点的个数，并暴力枚举三角形三个顶点分类的所有组合，就可以统计出符合条件的所有情况数。记为 $s$，那么题目所求即为 $c \equiv s \times \binom{n}{3}^{-1} \pmod p$，其中 $p = 998244353$，可以求逆元解决。

## 参考代码

```cpp
#include <iostream>
using namespace std;

const int MAXN = 1e5 + 10, MOD = 998244353;
int sum[5];
int n;

long long power(long long a, int b)
{
    long long base = a % MOD, ans = 1;
    while (b) {
        if (b & 1) {
            ans = (ans * base) % MOD;
        }
        base = (base * base) % MOD;
        b >>= 1;
    }
    return ans;
}

struct Point {
    int x, y;
    const Point operator - (const Point &a) const {
        return {x - a.x, y - a.y};
    }
    const long long operator ^ (const Point &a) const {
        return x * a.y - a.x * y;
    }
} p[4] = {{0, 0}, {0, 1}, {1, 0}, {1, 1}};

int cnt[4];
int typ[3];
long long ans = 0;

void dfs(int x)
{
    if (x == 3) {
        if (((p[typ[2]] - p[typ[0]]) ^ (p[typ[1]] - p[typ[0]])) % 2 == 0) {
            long long mul = 1;
            for (int i = 0; i < 4; i++) {
                for (int j = 1; j <= cnt[i]; j++) {
                    mul = mul * (sum[i] - j + 1) % MOD;
                    mul = mul * power(j, MOD - 2) % MOD;
                }
            }
            ans = (ans + mul) % MOD;
        }
        return;
    }
    for (int i = (x > 0 ? typ[x - 1] : 0); i < 4; i++) {
        if (cnt[i] >= sum[i]) continue;
        cnt[i]++;
        typ[x] = i;
        dfs(x + 1);
        cnt[i]--;
    }
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin >> n;
    for (int i = 1; i <= n; i++) {
        long long x, y;
        cin >> x >> y;
        x = (x & 1);
        y = (y & 1);
        if (x == 0 && y == 0) sum[0]++;
        else if (x == 0 && y == 1) sum[1]++;
        else if (x == 1 && y == 0) sum[2]++;
        else if (x == 1 && y == 1) sum[3]++;
    }
    dfs(0);
    ans = (ans * 6) % MOD;
    ans = (ans * power(((long long) n * (n - 1) % MOD) * (n - 2) % MOD, MOD - 2)) % MOD;
    cout << ans << endl;
    return 0;
}
```