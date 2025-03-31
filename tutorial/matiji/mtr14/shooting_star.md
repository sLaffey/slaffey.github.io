# 流星

## 题目大意

求空间任意两条直线的距离。

## 解题思路

对于两条异面直线，设两直线上分别任取一点得到向量为 $\vec{r}$，两直线方向向量分别为 $\vec{s}$，$\vec{t}$。则直线距离可以表示为 $\vec{r}$ 在 $\vec{s} \times \vec{t}$ 上的投影。

对于同一平面的直线，距离用 $\vec{s}$ 或 $\vec{t}$ 与 $\vec{r}$ 围成的平行四边形面积表示。

## 参考代码

```cpp
#include <iostream>
#include <cmath>
#include <iomanip>
using namespace std;


struct Point {
    long double x, y, z;

    const Point operator ^ (const Point &a) const {
        return {y * a.z - a.y * z, a.x * z - x * a.z, x * a.y - a.x * y};
    }

    const Point operator - (const Point &a) const {
        return {x - a.x, y - a.y, z - a.z};
    }

    const double Module() const {
        return sqrt(x * x + y * y + z * z);
    }

    friend istream& operator >> (istream& is, Point &a) {
        is >> a.x >> a.y >> a.z;
        return is;
    }
};

const long double MixMul(const Point &a, const Point &b, const Point &c)
{
    return a.x * (b.y * c.z - c.y * b.z) - a.y * (b.x * c.z - c.x * b.z) + a.z * (b.x * c.y - c.x * b.y);
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    int t;
    cin >> t;
    while (t--) {
        Point a, b, c, d;
        cin >> a >> b >> c >> d;
        Point r = c - a, s = b - a, t = d - c;
        cout << fixed << setprecision(3);
        long double m = MixMul(r, s, t);
        if (m == 0) {
            cout << abs((r ^ s).Module() / s.Module()) << endl;
        }
        else cout << abs(MixMul(r, s, t) / (s ^ t).Module()) << endl;
    }
    return 0;
}
```