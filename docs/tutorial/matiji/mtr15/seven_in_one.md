# 一串七

## 题目大意

给出平面上 $n$ 个不重合的点和一个整数 $k$，求能穿过至少 $k$ 个点的直线的个数。

## 解题思路

暴力枚举即可，注意去重。

具体来说，枚举前两个点以确定直线，并枚举第三个点统计在这条直线上的点的个数。时间复杂度 $O(n^3)$。

## 参考代码

```cpp
#include <iostream>
using namespace std;

const int MAXN = 410;
int n, m;
long long x[MAXN], y[MAXN];

inline bool isLine(const int &i, const int &j, const int &k)
{
    return abs((x[i] - x[j]) * (y[j] - y[k])) == abs((x[j] - x[k]) * (y[i] - y[j]));
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        cin >> x[i] >> y[i];
    }
    if (m == 1) {
        cout << -1 << endl;
        return 0;
    }

    int ans = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = i + 1; j <= n; j++) {
            int cnt = 2;
            bool flag = true;
            for (int k = 1; k < j; k++) {
                if (k == i) continue;
                if (isLine(i, j, k)) {
                    flag = false;
                }
            }
            if (!flag) continue;
            for (int k = j + 1; k <= n; k++) {
                if (isLine(i, j, k)) {
                    cnt++;
                }
            }
            if (cnt >= m) ans++;
        }
    }

    cout << ans << endl;
    return 0;
}
```