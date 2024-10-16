# 「POJ 1738」连通图

男人八题第一题就这么难的吗（

状态转移方程：

$$
f[i] = \sum _{j = 1}^{j < i} f[j] \times f[i - j] \times (2^j - 1) \times C^{j - 1} _{i - 2}
$$

注意用高精就完了。

（实际上我也不是很懂这个状转，之后再补讲解罢）

```cpp
#include <cstdio>
#include <cstring>
using namespace std;

const int& max(const int &a, const int &b) { return a > b ? a : b; }

int out[8000], top;

struct hipre {
    long long val[1000];
    int len;
    static const unsigned int MOD = 1e8;

    hipre()
    {
        len = 0;
        memset(val, 0, sizeof val);
    }

    hipre(long long x)
    {
        len = 0;
        while (x) {
            val[++len] = x % MOD;
            x /= MOD;
        }
    }

    const hipre& operator = (const hipre &a)
    {
        memset(val + 1, 0, sizeof(long long) * len);
        for (int i = 1; i <= a.len; i++) {
            val[i] = a.val[i];
        }
        len = a.len;
        return *this;
    }

    hipre& operator += (const hipre &a) 
    {
        len = max(len, a.len) + 1;
        for (int i = 1; i <= len; i++) {
            val[i] += a.val[i];
            val[i + 1] += val[i] / MOD;
            val[i] %= MOD;
        }
        while (len && !val[len]) len--;
        return *this;
    }

    const hipre operator + (const hipre &a) const 
    {
        hipre c = *this;
        c += a;
        return c;
    }

    const hipre operator * (const hipre &a) const 
    {
        hipre c;
        c.len = len + a.len;
        for (int i = 1; i <= len; i++) {
            for (int j = 1; j <= a.len; j++) {
                c.val[i + j - 1] += val[i] * a.val[j];
            }
        }

        for (int i = 1; i <= c.len; i++) {
            c.val[i + 1] += c.val[i] / MOD;
            c.val[i] %= MOD;
        }

        while (c.len && !c.val[c.len]) c.len--;

        return c;
    }

    void print() const
    {
        top = 0;
        for (int i = 1; i <= len; i++) {
            long long x = val[i];
            for (int j = 1; j <= 8; j++) {
                out[++top] = x % 10;
                x /= 10;
            }
        }

        while (!out[top]) top--;

        for (int i = top; i > 0; i--) {
            printf("%d", out[i]);
        }
    }
};

typedef long long ll;

const ll& max(const ll &a, const ll &b) { return a > b ? a : b; }

const int MAXN = 55;
ll C[MAXN][MAXN];
hipre f[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
#endif
    for (int i = 0; i <= 50; i++) {
        C[i][0] = 1;
        for (int j = 1; j <= i; j++) {
            C[i][j] = C[i - 1][j] + C[i - 1][j - 1];
        }
    }

    f[1] = f[2] = 1;
    for (int i = 3; i <= 50; i++) {
        for (int j = 1; j < i; j++) {
            f[i] += f[j] * f[i - j] * ((1LL << j) - 1) * C[i - 2][j - 1];
        }
    }

    int n;
    while (scanf("%d", &n) != EOF && n) {
        f[n].print();
        printf("\n");
    }

    return 0;
}
```