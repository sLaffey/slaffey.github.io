# Prefix Min and Suffix Max

只要自身是前缀最小或者后缀最大就可以。

```cpp
#include <iostream>
using namespace std;

const int MAXN = 2e5 + 10;
int a[MAXN];
int ans[MAXN];

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
        int n;
        cin >> n;
        for (int i = 1; i <= n; i++) {
            cin >> a[i];
            ans[i] = 0;
        }
        for (int i = 1, pre = a[1]; i <= n; i++) {
            pre = min(pre, a[i]);
            if (pre == a[i]) ans[i] = 1;
        }
        for (int i = n, suf = a[n]; i >= 1; i--) {
            suf = max(suf, a[i]);
            if (suf == a[i]) ans[i] = 1;
        }
        for (int i = 1; i <= n; i++) {
            cout << ans[i];
        }
        cout << '\n';
    }
    return 0;
}
```