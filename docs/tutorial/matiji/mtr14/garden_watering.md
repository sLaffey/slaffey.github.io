# 花园浇水

$n$ 的范围很小，直接枚举中间最高点的坐标，向两边扩展即可。

```cpp
#include <iostream>
using namespace std;

const int MAXN = 1e3 + 10;
int a[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        int l = i, r = i;
        while (a[l - 1] <= a[l] && l > 1) l--;
        while (a[r + 1] <= a[r] && r < n) r++;
        ans = max(ans, r - l + 1);
    }

    cout << ans << endl;
    return 0;
}
```