# 「2024 昆明邀请赛」Italian Cuisine

## 题意描述

按照逆时针顺序给出一个多边形上的所有点，并给出其内部一个圆。考虑连接多边形的两个顶点，并使其与圆不相交，问多边形不含圆的一侧的最大面积是多少？

数据规模为 $10^5$。

## 解题思路

采用类似旋转卡壳的思路。逆时针依次考虑多边形的每一个顶点，对于每个顶点都考虑使这一侧不含圆且面积最大的分割方案。形象地讲，就是维护一个线段在多边形内部不断旋转寻找最优方案。具体地说，需要维护两个指针分别指向该线段的两个顶点，两个顶点依次向后更新，保证总是最优。

## 参考代码

```cpp
#include <iostream>
#include <cmath>
using namespace std;

using point_t = long long;
constexpr point_t eps = 1e-8;

// 点与向量
template <typename T>
struct point {
    T x, y;

    const bool operator == (const point &a) const { return abs(x - a.x) <= eps && abs(y - a.y) <= eps; }
    const point operator + (const point &a) const { return (point) {x + a.x, y + a.y}; }
    const point operator - (const point &a) const { return (point) {x - a.x, y - a.y}; }
    const point operator - () const { return (point) {-x, -y}; }
    const point operator * (const T &k) const { return (point) {x * k, y * k}; }
    const point operator / (const T &k) const { return (point) {x / k, y / k}; }
    const T operator * (const point &a) const { return x * a.x + y * a.y; } // 点积
    const T operator ^ (const point &a) const { return x * a.y - y * a.x; } // 叉积
    const T len2() const { return x * x + y * y; }
    const T dis2(const point &a) { return ((*this) - a).len2(); }
};

using Point = point<point_t>;
const int MAXN = 1e5 + 10;
Point pts[MAXN];

int main()
{
#ifndef ONLINE_JUDGE
    freopen("./in", "r", stdin);
#endif
    ios_base::sync_with_stdio(false);
    int t;
    cin >> t;
    while (t--) {
        int n;
        cin >> n;
        Point cir;
        int r;
        cin >> cir.x >> cir.y >> r;
        for (int i = 0; i < n; i++) {
            cin >> pts[i].x >> pts[i].y;
        }
        long long ans = 0, now = 0;
        for (int i = 0, j = 1; i < n; i++) {
            while (true) {
                int next_j = (j + 1) % n;
                long long cross = ((pts[next_j] - pts[i]) ^ (cir - pts[i]));
                if (cross <= 0 || (__int128_t) cross * cross < (__int128_t) r * r * pts[i].dis2(pts[next_j])) {
                    break;
                }
                cross = (pts[j] - pts[i]) ^ (pts[next_j] - pts[i]);
                now += abs(cross);
                j = next_j;
            }
            ans = max(ans, now);
            int next_i = (i + 1) % n;
            long long cross = (pts[j] - pts[i]) ^ (pts[next_i] - pts[i]);
            now -= abs(cross);
        }
        cout << ans << '\n';
    }
    return 0;
}
```