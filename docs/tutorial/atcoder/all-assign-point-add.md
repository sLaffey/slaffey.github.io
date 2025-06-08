# ABC278D All Assign Point Add

## Solution

&emsp;&emsp;题目是一个简单的区间覆盖修改 + 单点查询问题。可以直接线段树在 $O(n + q \log n)$ 的复杂度内解决。

&emsp;&emsp;注意到后两种操作可以用普通的数组，而区间覆盖只有全部覆盖一种范围。我们可否利用这些性质优化时间复杂度呢？

&emsp;&emsp;答案是肯定的。类比线段树的懒标记思想，我们可以在执行覆盖时先不进行任何操作，在之后用到这个数字时再进行修改。这可以通过再开一个标记数组实现。

&emsp;&emsp;然而，我们每次覆盖都要清空原有的标记，如果朴素地做复杂度将会达到 $O(nq)$，这是我们不能接受的。为了优化，可以引入「时间戳」的思想。每次清空标记时，将当前时间戳加上 $1$；判断是否进行过修改时，判断这个位置的标记是否和当前时间戳相等即可。这些操作都是 $O(1)$ 的。因此总的复杂度可以做到 $O(n + q)$。

## Code

```cpp
#include <iostream>
using namespace std;

typedef long long ll;

const int MAXN = 2e5 + 10;
ll a[MAXN];
int v[MAXN];
int cov, top;

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    // freopen("out", "w", stdout);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);
    const char endl = '\n';

    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    int q;
    cin >> q;
    while (q--) {
        int t, p, x;
        cin >> t;
        switch (t) {
            case 1:
                cin >> cov;
                top++;
                break;
            case 2:
                cin >> p >> x;
                if (v[p] != top) {
                    v[p] = top;
                    a[p] = cov;
                }
                a[p] += x;
                break;
            case 3:
                cin >> p;
                if (v[p] != top) {
                    v[p] = top;
                    a[p] = cov;
                }
                cout << a[p] << endl;
                break;
        }
    }

    return 0;
}
```