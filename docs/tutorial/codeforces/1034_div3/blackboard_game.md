# Blackboard Game

不难发现局面中任意模 $4$ 不同余的一组数不影响结果，因此只考虑四种情况输赢即可。

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
        int n;
        cin >> n;
        const int res[4] = {0, 0, 0, 1};
        cout << (res[(n - 1) % 4] ? "Bob\n" : "Alice\n");
    }
    return 0;
}
```