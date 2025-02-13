# CF2067D Object Identification

## 题目大意

交互题。给定一个范围为 $1 \sim n$ 的整数序列 $x_1, x_2, \ldots, x_n$（但不一定是 $1 \sim n$ 的排列），交互器同时持有相同范围的整数序列 $y_1, y_2, \ldots, y_n$，题目保证对所有的 $i$ 有 $x_i \neq y_i$ 且所有有序数对 $(x_i, y_i)$ 互不相同。

交互器会将这两个序列确定为以下两种之一的对象，你需要通过询问得出是哪一种：

- 对象 A：一个 $n$ 个点 $n$ 条边的有向图，第 $i$ 条边为 $x_i \to y_i$。
- 对象 B：平面直角坐标系下的 $n$ 个点，第 $i$ 个点坐标为 $(x_i, y_i)$。

你有 $2$ 次机会发出询问，每次询问需要指定两个范围为 $1 \sim n$ 的数字 $i, j$，且需满足 $i \neq j$。对每次询问，交互器将给出单独一个数字作为答案：

- 若交互器确定的对象种类为 A，你得到点 $i$ 到点 $j$ 的最短路径长度，若不存在该路径，则得到 $0$。
- 若交互器确定的对象种类为 B，你得到点 $i$ 到点 $j$ 的曼哈顿距离，即 $|x_i - x_j| + |y_i - y_j|$。

一个测试点有最多 $1000$ 组数据，对于每组数据，有 $3 \leq n \leq 2 \times 10^5$。

## 解题思路

我们分两种情况讨论：

若 $x_1, x_2, \ldots, x_n$ **不为** $1 \sim n$ 的一个排列，则一定存在一个数 $a$ 满足 $1 \leq a \leq n$ 且 $a \not \in \{x_n\}$。此时若交互器选择对象 A，则对任意一个不同于 $a$ 的点 $b$ 有询问 $(a, b)$ 的答案是 $0$，也即二者不连通。据此我们可以询问一次，并立即得到答案。

若 $x_1, x_2, \ldots, x_n$ **为** $1 \sim n$ 的一个排列，则我们一定可以找到 $i, j$ 满足 $x_i = 1, x_j = n$。此时可以对这两个点进行正反两次询问。若询问 $(i, j)$ 与询问 $(j, i)$ 的答案不一致，则可以判断为对象 A；若询问 $(i, j)$ 与询问 $(j, i)$ 的答案相同，设其答案为 $a$，则当交互器选择对象 A 时，一定有 $a \leq \lfloor n/2 \rfloor$，当交互器选择对象 B 时，一定有 $a \geq |x_i - x_j| = n - 1$。据此也可以判断得到答案。

## 参考代码

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int MAXN = 2e5 + 10;
struct obj {
    int x, i;
} a[MAXN];

inline int query(int i, int j)
{
    cout << "? " << i << ' ' << j << endl;
    int ans = -1;
    cin >> ans;
    return ans;
}

inline void answer(char c)
{
    cout << "! " << c << endl;
}

int main()
{
    ios_base::sync_with_stdio(false);
    int t;
    cin >> t;
    while (t--) {
        int n;
        cin >> n;
        for (int i = 1; i <= n; i++) {
            cin >> a[i].x;
            a[i].i = i;
        }
        sort(a + 1, a + n + 1, [](const obj &a, const obj &b) -> const bool { return a.x < b.x; });
        bool answered = false;
        for (int i = 1, p = 1; p <= n; p++) {
            while (a[i].x < p && i < n) i++;
            if (a[i].x != p) {
                int t = 1;
                while (a[t].i == p) t++;
                int ans1 = query(a[t].i, p), ans2 = query(p, a[t].i);
                if (ans1 == 0 || ans2 == 0) {
                    answer('A');
                }
                else {
                    answer('B');
                }
                answered = true;
                break;
            }
        }
        if (!answered) {
            int ans1 = query(a[1].i, a[n].i), ans2 = query(a[n].i, a[1].i);
            if (ans1 == ans2 && ans1 > 0 && ans1 >= a[n].x - a[1].x) {
                answer('B');
            }
            else {
                answer('A');
            }
        }
    }
    return 0;
}
```