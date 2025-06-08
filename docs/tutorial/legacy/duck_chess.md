# 「THUPC 2019」鸭棋

&emsp;&emsp;2022.9.23 晚十时许，Laffey 突发奇想，要写一个模拟（屎山）。于是他找到了[鸭棋](https://www.luogu.com.cn/problem/P5380)。

&emsp;&emsp;2022.9.24 晚九时许，经过总共约 3.5 h 的代码编写，Laffey 成功 AC。调试时间并不算太长。

&emsp;&emsp;来欣赏总长 8.1 KB 的代码罢！

```cpp
#include <cstdio>
using namespace std;

inline const int abs(const int &a) { return a > 0 ? a : -a; }
inline const int& min(const int &a, const int &b) { return a > b ? b : a; }
inline const int& max(const int &a, const int &b) { return a > b ? a : b; }

enum {
    null = 0,
    blue_captain = 1,
    blue_guard = 2,
    blue_elephant = 3,
    blue_horse = 4,
    blue_car = 5,
    blue_duck = 6,
    blue_soldier = 7,
    red_captain = 8,
    red_guard = 9,
    red_elephant = 10,
    red_horse = 11,
    red_car = 12,
    red_duck = 13,
    red_soldier = 14,
    blue = false,
    red = true
};

const char table[15][20] = {
    "Error type.",
    "blue captain",
    "blue guard",
    "blue elephant",
    "blue horse",
    "blue car",
    "blue duck",
    "blue soldier",
    "red captain",
    "red guard",
    "red elephant",
    "red horse",
    "red car",
    "red duck",
    "red soldier"
};

int board[11][10] = {
    {0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
    {0, red_car, red_horse, red_elephant, red_guard, red_captain, red_guard, red_elephant, red_horse, red_car},
    {0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
    {0, red_duck, 0, 0, 0, 0, 0, 0, 0, red_duck},
    {0, red_soldier, 0, red_soldier, 0, red_soldier, 0, red_soldier, 0, red_soldier},
    {0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
    {0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
    {0, blue_soldier, 0, blue_soldier, 0, blue_soldier, 0, blue_soldier, 0, blue_soldier},
    {0, blue_duck, 0, 0, 0, 0, 0, 0, 0, blue_duck},
    {0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
    {0, blue_car, blue_horse, blue_elephant, blue_guard, blue_captain, blue_guard, blue_elephant, blue_horse, blue_car}
};

int xs, ys, xt, yt;
bool turn = red;
bool end = false;

bool group(int piece) { return piece / 8; }

bool check(int xs, int ys, int xt, int yt, bool ignore)
{
    // check whether the option is valid or not

    // check if the game is ended
    if (end) {
        return false;
    }

    // check if the start & end positions are on the board
    auto inb = [](const int &x, const int &y) -> const bool {
        return x > 0 && x < 11 && y > 0 && y < 10;
    };
    if (!(inb(xs, ys) && inb(xt, yt))) {
        return false;
    }

    // check if there is a chess piece
    if (!board[xs][ys]) return false;

    // check if there is a chess piece in the same group at the moving-to position
    if (board[xt][yt] && group(board[xs][ys]) == group(board[xt][yt])) return false;

    int st = board[xs][ys], ed = board[xt][yt];

    // check if turn is right
    if (!ignore)
        if (turn != group(st)) {
            return false;
        }

    // check for captain
    int hx = abs(xt - xs), hy = abs(yt - ys);
    if (st == red_captain || st == blue_captain) {
        // check the position
        if (abs(xt - xs) + abs(yt - ys) > 1) {
            return false;
        }
    }
    // check for guard
    else if (st == red_guard || st == blue_guard) {
        // check the position
        if (hx != 1 || hy != 1) {
            return false;
        }
    }
    // check for elephant
    else if (st == red_elephant || st == blue_elephant) {
        // check if there is a chess piece
        if (board[xs + xt >> 1][ys + yt >> 1]) {
            return false;
        }
        // check the position
        if (hx != 2 || hy != 2) {
            return false;
        }
    }
    // check for horse
    else if (st == red_horse || st == blue_horse) {
        // check if there is a chess piece
        if ((board[xs + xt >> 1][ys] && hx == 2) || (board[xs][ys + yt >> 1] && hy == 2)) {
            return false;
        }
        // check the position
        if (!(hx == 1 && hy == 2) && !(hx == 2 && hy == 1)) {
            return false;
        }
    }
    // check for car
    else if (st == red_car || st == blue_car) {
        // check the position
        if (xs != xt && ys != yt) {
            return false;
        }
        // check if there is a chess piece
        for (int i = min(xs, xt) + 1; i < max(xs, xt); i++) {
            if (board[i][ys]) {
                return false;
            }
        }
        for (int i = min(ys, yt) + 1; i < max(ys, yt); i++) {
            if (board[xs][i]) {
                return false;
            }
        }
    }
    // check for duck
    else if (st == red_duck || st == blue_duck) {
        // check the position
        if (!((hx == 3 && hy == 2) || (hx == 2 && hy == 3))) {
            return false;
        }
        // check if there is a chess piece
        int mx = xs + xt >> 1, my = ys + yt >> 1;
        if (yt > ys && xt > xs) {
            if (hx == 3) {
                if (board[xs + 1][ys] || board[xs + 2][ys + 1]) {
                    return false;
                }
            }
            else {
                if (board[xs][ys + 1] || board[xs + 1][ys + 2]) {
                    return false;
                }
            }
        }
        else if (yt > ys && xt < xs) {
            if (hx == 3) {
                if (board[xs - 1][ys] || board[xs - 2][ys + 1]) {
                    return false;
                }
            }
            else {
                if (board[xs][ys + 1] || board[xs - 1][ys + 2]) {
                    return false;
                }
            }
        }
        else if (yt < ys && xt > xs) {
            if (hx == 3) {
                if (board[xs + 1][ys] || board[xs + 2][ys - 1]) {
                    return false;
                }
            }
            else {
                if (board[xs][ys - 1] || board[xs + 1][ys - 2]) {
                    return false;
                }
            }
        }
        else if (yt < ys && xt < xs) {
            if (hx == 3) {
                if (board[xs - 1][ys] || board[xs - 2][ys - 1]) {
                    return false;
                }
            }
            else {
                if (board[xs][ys - 1] || board[xs - 1][ys - 2]) {
                    return false;
                }
            }
        }
    }
    // check for soldier
    else if (st == red_soldier || st == blue_soldier) {
        // check the position
        if (hx + hy > 2 || hx == 2 || hy == 2) {
            return false;
        }
    }
    // throw error
    else {
        printf("Error from [func] check(): The type of the chess piece at start position is invalid.\n");
    }

    return true;
}

bool checkmate(int x, int y)
{
    // another "long long" function
    // nen hen hen aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
    int p = board[x][y];
    int tg;

    if (group(p) == blue) {
        tg = red_captain;
    }
    else {
        tg = blue_captain;
    }

    // search for captain
    int xt = 0, yt = 0;
    for (int i = 1; i < 11 && !xt; i++) {
        for (int j = 1; j < 10; j++) {
            if (board[i][j] == tg) {
                xt = i, yt = j;
                break;
            }
        }
    }

    // check
    return check(x, y, xt, yt, true);
}

void move()
{
    int &st = board[xs][ys], &ed = board[xt][yt];
    if (ed) {
        printf("%s;", table[ed]);
        if (ed == blue_captain || ed == red_captain) {
            end = true;
        }
    }
    else {
        printf("NA;");
    }
    ed = st;
    st = null;
    
    bool cm = false;
    for (int i = 1; i < 11 && !cm; i++) {
        for (int j = 1; j < 10; j++) {
            if (board[i][j]) {
                if (checkmate(i, j)) {
                    cm = true;
                    break;
                }
            }
        }
    }
    printf(cm ? "yes;" : "no;");

    printf(end ? "yes\n" : "no\n");
    turn = !turn;
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in", "r", stdin);
    freopen("out", "w", stdout);
#endif
    int q;
    scanf("%d", &q);
    while (q--) {
        scanf("%d%d%d%d", &xs, &ys, &xt, &yt);
        xs++, ys++, xt++, yt++;
        if (!check(xs, ys, xt, yt, false)) {
            printf("Invalid command\n");
        }
        else {
            int st = board[xs][ys], ed = board[xt][yt];
            // output the type
            printf("%s;", table[st]);
            move();
        }
    }

    return 0;
}
```