# Count Good Numbers

## 题目大意

所有质因子都至少是两位数的数称为好的数。计算 $[l, r]$ 中所有好的数的数目。

## 解题思路

一位数的质数只有 $2, 3, 5, 7$，用容斥原理容易算出。

## 参考代码

```cpp
#include <iostream>
using namespace std;

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);

    
    int t;
    cin >> t;
    while (t--) {
        long long l, r;
        cin >> l >> r;
        long long cnt = 0;
        for (int i = 1; i < (1 << 4); i++) {
            auto count = [&](int x) {
                static const int p[4] = {2, 3, 5, 7};
                long long ans = 0, base = 1, cnt = 0;
                for (int i = 0; i < 4; i++) {
                    if ((x >> i) & 1) {
                        base *= p[i];
                        cnt++;
                    }
                }
                return (r / base - (l - 1) / base) * ((cnt & 1) ? 1 : -1);
            };
            cnt += count(i);
        }
        cout << (r - l + 1 - cnt) << '\n';
    }
    return 0;
}
```