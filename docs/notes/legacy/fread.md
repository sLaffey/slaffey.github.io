# fread 优化快读

一般来讲，对于输入量特别大的题目，普通快读就足够了，但是实际上还有更快的输入方式——`fread`。

### 代码模板

模板采用了函数对象的形式封装快读。

```cpp
#include <cstdio>
#include <cctype>
#include <ctime>
using namespace std;

class QuickRead {

private:
    static const int BUFFER_SIZE = 1e6 + 10;
    static const int READ_MAX = 1e6;
    char ReadBuffer[BUFFER_SIZE];
    int p = READ_MAX;

    inline const char& get()
    {
        if (p == READ_MAX) { // refresh the buffer
            fread(ReadBuffer + 1, 1, READ_MAX, stdin);
            p = 0;
        }
        return ReadBuffer[++p];
    }

public:
    inline const int operator() ()
    {
        int ans = 0, inv = 1;
        char c = get();
        while (isspace(c)) {
            c = get();
        }
        if (c == '-') {
            inv = -1;
            c = get();
        }
        while (isdigit(c)) {
            ans *= 10;
            ans += c - '0';
            c = get();
        }
        return ans * inv;
    }
} read;

int main()
{
    freopen("in", "r", stdin);
    int beginTime = clock();
    int n;
    for (int i = 1; i <= 1e8; i++) {
        n = read();
    }
    int endTime = clock();
    printf("%.3lfms\n", (double) (endTime - beginTime) / CLOCKS_PER_SEC * 1000);
}
```

看起来 $10^6$ 的缓存数组很大，但实际上占用内存仅有 `1MiB` 左右，完全可以放心使用。

### 测试环境

| Key | Value |
| --- | --- |
| CPU | Intel Core i5-1035G1 |
| OS | Ubuntu LTS 22.04 64bit |
| PC | Dell Inspiron 5493 |
| Compiler | g++-11.2.0-posix-x86_64-linux-gnu |
| Args | `g++ <input> -o <output> -Wall -Wextra -std=c++20 -m64 (-O2)` |

### 测试结果

| O2 | fread | qread | scanf | cin | cin (optimized) |
| --- | --- | --- | --- | --- | --- |
| No | `3650.354ms ~ 5058.915ms` | `4212.445ms ~ 4458.994ms` | `7143.911ms ~ 8037.850ms` | `22570.459ms` | `6047.171ms ~ 6891.675ms` |
| Yes | `1018.914ms ~ 1149.519ms` | `2603.585ms ~ 2733.011ms` | `8014.751ms ~ 8308.432ms` | `24607.168ms` | `5830.896ms ~ 6773.609ms` |

注：

- 每种输入方式测试了至少三次，取最大与最小值。
- `cin` 由于实在太慢没有多次测试，仅供参考。
- `cin (optimized)` 使用 `ios_base::sync_with_stdio(false)` 与 `cin.tie(0), cout.tie(0)` 进行优化。
- 输入数据为 $1 \sim 10^8$ 的递增序列。

### 结果分析

不难看到，在未吸氧的情况下，普通快读较快且较稳定，但在吸氧以后，全员被 `fread` 暴打。

另外，优化后的 `cin` 效率也很吸引人，和 `scanf` 不相上下，甚至比前者快。

### 附录

附 1：测试使用快读代码。

```cpp
inline int read() {
    int now = 0, nev = 1;
    char c = getchar();
    while (c < '0' || c > '9') {
        if (c == '-')
            nev = -1;
        c = getchar();
    }
    while (c >= '0' && c <= '9') {
        now = (now << 1) + (now << 3) + (c & 15);
        c = getchar();
    }
    return now * nev;
}
```

附 2：也可以不使用函数对象封装直接使用。

```cpp
const int BUFFER_SIZE = 1e6 + 10;
const int READ_MAX = 1e6;
char ReadBuffer[BUFFER_SIZE];
int p = READ_MAX;

inline const char& get()
{
    if (p == READ_MAX) { // refresh the buffer
        fread(ReadBuffer + 1, 1, READ_MAX, stdin);
        p = 0;
    }
    return ReadBuffer[++p];
}

inline const int read()
{
    int ans = 0, inv = 1;
    char c = get();
    while (isspace(c)) {
        c = get();
    }
    if (c == '-') {
        inv = -1;
        c = get();
    }
    while (isdigit(c)) {
        ans *= 10;
        ans += c - '0';
        c = get();
    }
    return ans * inv;
}
```

附 3：使用命名空间封装也是可以的。

```cpp
namespace QuickRead {
    const int BUFFER_SIZE = 1e6 + 10;
    const int READ_MAX = 1e6;
    char ReadBuffer[BUFFER_SIZE];
    int p = READ_MAX;

    inline const char& get()
    {
        if (p == READ_MAX) { // refresh the buffer
            fread(ReadBuffer + 1, 1, READ_MAX, stdin);
            p = 0;
        }
        return ReadBuffer[++p];
    }

    inline const int read()
    {
        int ans = 0, inv = 1;
        char c = get();
        while (isspace(c)) {
            c = get();
        }
        if (c == '-') {
            inv = -1;
            c = get();
        }
        while (isdigit(c)) {
            ans *= 10;
            ans += c - '0';
            c = get();
        }
        return ans * inv;
    }
}

int main()
{
    using QuickRead::read;
}
```

附 4：

~~贴个现在常用的板子在这方便复制~~

```cpp
namespace FastInput {
    const int SIZE = 1e6 + 10;
    const int RMAX = 1e6;
    char Buffer[SIZE];
    int p = RMAX;

    inline const char& get()
    {
        if (p == RMAX) {
            fread(Buffer + 1, 1, RMAX, stdin);
            p = 0;
        }
        return Buffer[++p];
    }

    inline int read()
    {
        int ans = 0, nev = 1;
        char c = get();
        while (c > '9' || c < '0') {
            if (c == '-') nev = -1;
            c = get();
        }
        while (c >= '0' && c <= '9') {
            ans = (ans << 1) + (ans << 3) + (c ^ 48);
            c = get();
        }
        return ans * nev;
    }
}
```