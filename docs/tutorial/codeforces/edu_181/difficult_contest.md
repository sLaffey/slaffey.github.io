# Difficult Contest

## 题目大意

给一个字符串，重新排列它使得其中不存在形如 `FFT` 或 `NTT` 的连续子串，若没有则维持原状。

## 解题思路

容易想到直接按字母从大到小排列即可。

## 参考代码

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const char *t1 = "FFT", *t2 = "NTT";
string s;

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
        cin >> s;

        auto Find = [&](const char *t) {
            for (int i = 0; i < s.length() - 2; i++) {
                if (s[i] == t[0] && s[i + 1] == t[1] && s[i + 2] == t[2]) {
                    return true;
                }
            }
            return false;
        };
        
        if (Find(t1) || Find(t2)) {
            sort(s.begin(), s.end(), [](const char &a, const char &b) { return a > b; });
        }
        cout << s << '\n';
    }
    return 0;
}
```