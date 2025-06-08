# 输入法

## 题目大意

给一个包含 $n$ 个单词的词库，第 $i$ 个记作 $s_i$，保证只包含小写字母。

维护一个字符串，支持两种操作：

- 在字符串的最右边插入一个字母。
- 在字符串的最右边删去一个字母。

题目进行 $m$ 次操作，每次操作后，你需要输出词库中以当前字符串为前缀的单词数量。

数据范围：$1 \leq n, m \leq 10^5, 1 \leq \sum_{i=1}^n|s_i| \leq 10^6$。

## 解题思路

Trie 板子题，注意一下静态数组模拟指针进行动态开点即可。

## 参考代码

```cpp
#include <iostream>
using namespace std;

const int MAXS = 1e6 + 10, MAXN = 1e5 + 10;
int cnt[MAXS], Next[MAXS][30], tot = 1;
int pre[MAXS];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        string s;
        cin >> s;
        for (int i = 0, p = 1; i < s.length(); i++) {
            int c = s[i] - 'a';
            if (Next[p][c] == 0) {
                Next[p][c] = ++tot;
            }
            p = Next[p][c];
            cnt[p]++;
        }
    }
    pre[1] = 1;
    cnt[1] = n;

    int p = 1;
    while (m--) {
        int op;
        char c;
        cin >> op;
        if (op == 1) {
            cin >> c;
            int q = c - 'a';
            if (Next[p][q] == 0) {
                Next[p][q] = ++tot;
            }
            pre[Next[p][q]] = p;
            p = Next[p][q];
            cout << cnt[p] << '\n';
        }
        else if (op == 2) {
            p = pre[p];
            cout << cnt[p] << '\n';
        }
    }

    return 0;
}
```
