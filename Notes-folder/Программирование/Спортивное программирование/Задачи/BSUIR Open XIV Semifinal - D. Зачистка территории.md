# Дата - 2026-04-11

[Ссылка на задачу](https://solve.bsuir.by/contests)

Tags: #спортивное_программирование #структуры_данных 

---
## О чём задача:
Дано поле $n \times m$. Каждая клетка имеет значение $0 \leq a_{i} \leq 10^9$. Нам выбрать прямоугольник размера $A \times B$ и сделать все элементы в нём равными. Элемент можно только уменьшить или ничего не делать. Наша работа - это то, на сколько мы уменьшили каждый элемент. Нужно выбрать такой прямоугольник в поле, чтобы минимизировать работу.

$$
\large 1 \leq n,m \leq 6000
$$

$$
\large 1 \leq A \leq n, 1 \leq B \leq m
$$

---
## Решение задачи:  
Так как элементы мы можем только уменьшать, то лучше всего найти минимум и все элементы, которые больше, уменьшить до него. Делать меньше минимума не выгодно.

Тогда мы можем перебрать все прямоугольники, найти минимум и суммарная работа, которая будет потрачена в этом прямоугольнике - это $sum - min * A * B$.
То есть сумма на прямоугольнике минус минимум умноженный на размер.

Теперь, из-за больших ограничений нам нужно придумать алгоритм, который будет быстро искать минимум на прямоугольнике.

Есть 2 решения:
1. Написать двумерный sparse table и получать минимум на отрезке за O(1)
2. Так как прямоугольник не меняет размеры, то можно искать минимум с помощью стеков

Первый способ плох тем, что ест много памяти и строится за O(n\*m\*log(n)) (Хоть и не ловит TL). Но возникнет проблема с памятью, поэтому нужно её грамотно чистить. Решение довольно комплексное, поэтому не буду его объяснять, но самое главное - это то, то нам нужно посчитать только отрезки длинны A, а не всех степеней двоек.

Второй способ лучше, так как работает за линейное время и памяти тоже ест мало. Суть в том, что давайте посчитаем минимум для каждой строки размера B, после этого на новом массиве посчитаем минимум по столбцам размера A. И в конечной матрице в каждой клетке у нас будет храниться минимум на прямоугольнике.

Ниже представлены 2 реализации.
1. Sparse table
2. С помощью стека (дека)

--- 
## Код 1:

```c++
#include<iostream>
#include<vector>
#include<map>
#include<unordered_map>
#include<set>
#include<bitset>
#include<stack>
#include<queue>
#include<string>
#include<list>
#include<algorithm>
#include<cmath>
#include<numeric>
#include<iterator>
#include<iomanip>
#include<cassert>
#include<functional>
#include<random>
#include<ctime>
#include<bit>
#include<unordered_set>
#include<cstring>
#include<chrono>
//#pragma GCC optimize("Ofast,unroll-loops")
//#pragma GCC target("avx,avx2,fma")
//#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 5e5 + 3;
const int INF = 1e18;
const int MAXA = 5e6 + 3;
const int MOD = 1e9 + 7;

int logg2[6001];

int A, B;

void calcLog() {
    int k = 2;
    while (k <= 6000) {
        logg2[k]++;
        k <<= 1;
    }
    for (int i = 1; i <= 6000; i++) {
        logg2[i] += logg2[i - 1];
    }
}


struct STable {
    int n;
    int m;
    int maxA;
    int maxB;
    vector<vector<vector<int>>> st;

    STable(vector<vector<int>>& v) {
        n = v.size();
        m = v[0].size();
        maxA = logg2[A];
        maxB = logg2[B];
        st.assign(15, vector<vector<int>>(n));
        for (int i = 0; i < n; i++) {
            st[0][i].resize(m);
            for (int j = 0; j < m; j++) {
                st[0][i][j] = v[i][j];
            }
        }
        fB();
    }

    void fB() {
        int maxK = 2;
        for (int k = 1; k <= maxB; k++) {
            // st[k].resize(n);
            for (int i = 0; i < n; i++) {
                st[k][i].resize(m - maxK + 1);
                for (int j = 0; j + maxK - 1 < m; j++) {
                    st[k][i][j] = min(st[k - 1][i][j], st[k - 1][i][j + (maxK >> 1)]);
                }
            }
            maxK *= 2;
            st[k - 1].clear();
            st[k - 1].shrink_to_fit();
        }
    }

    void build() {
        st[0].resize(n);
        for (int i = 0; i < n; i++) {
            st[0][i].resize(m);
            for (int j = 0; j < m - B + 1; j++) {
                st[0][i][j] = query(i, j, j + B - 1);
            }
        }
        int maxK = 2;
        for (int k = 1; k <= maxA; k++) {
            st[k].resize(n - maxK + 1);
            for (int i = 0; i + maxK - 1 < n; i++) {
                st[k][i].resize(m - B + 1);
                for (int j = 0; j < m - B + 1; j++) {
                    st[k][i][j] = min(st[k - 1][i][j], st[k - 1][i + (maxK >> 1)][j]);
                }
            }
            st[k - 1].clear();
            st[k - 1].shrink_to_fit();
            maxK *= 2;
        }
    }

    int query2(int x, int y) {
        int k = logg2[A];
        return min(st[k][x][y], st[k][x + A - 1 - (1 << k) + 1][y]);
    }

    int query(int x, int l, int r) {
        int k = logg2[r - l + 1];
        return min(st[k][x][l], st[k][x][r - (1 << k) + 1]);
    }
};

long long mod = 1e9;

void solve() {
    long long n, m, seed, x, y;
    cin >> n >> m >> A >> B >> seed >> x >> y;
    vector<vector<int>> v(n, vector<int>(m));
    vector<vector<long long>> pref(n + 1, vector<long long>(m + 1, 0));
    calcLog();

    long long prevS = seed;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            // cin >> v[i][j];
            long long z = (prevS * x + y) % mod;
            if (z < 0) {
                z += mod;
            }
            v[i][j] = z;
            prevS = v[i][j];
        }
    }
    STable st(v);
    st.build();

    long long minAns = 1e18;
    pair<int, int> ans;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            pref[i + 1][j + 1] = pref[i][j + 1] + pref[i + 1][j] - pref[i][j] + v[i][j];

            int x1 = i - A + 1;
            int y1 = j - B + 1;

            if (x1 < 0 || y1 < 0)
                continue;

            long long minV = st.query2(x1, y1);
            long long inSum = pref[x1][y1] - pref[x1 + A][y1] - pref[x1][y1 + B] + pref[x1 + A][y1 + B];
            // cout << inSum << " " << minV << endl;
            if (inSum - minV * A * B < minAns) {
                minAns = inSum - minV * A * B;
                ans = { x1 + 1, y1 + 1 };
            }
        }
        if (i - A >= 0)
            pref[i - A].clear();

        if (i > 0)
            v[i - 1].clear();
    }

    cout << minAns << endl;
    cout << ans.first << " " << ans.second << endl;
}


signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
    //freopen("basis.in", "r", stdin);
    //freopen("basis.out", "w", stdout);
    int t = 1;
    //cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}
```
---

## Код 2:
```c++
#include<iostream>
#include<vector>
#include<map>
#include<unordered_map>
#include<set>
#include<bitset>
#include<stack>
#include<queue>
#include<string>
#include<list>
#include<algorithm>
#include<cmath>
#include<numeric>
#include<iterator>
#include<iomanip>
#include<cassert>
#include<functional>
#include<random>
#include<ctime>
#include<bit>
#include<unordered_set>
#include<cstring>
#include<chrono>
//#pragma GCC optimize("Ofast,unroll-loops")
//#pragma GCC target("avx,avx2,fma")
#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 5e5 + 3;
const int INF = 1e18;
const int MAXA = 5e6 + 3;
const int MOD = 1e9;

void solve() {
    long long n, m, A, B, seed, x, y;
    cin >> n >> m >> A >> B >> seed >> x >> y;
    vector<vector<int>> v(n, vector<int>(m));
    vector<vector<int>> pref(n + 1, vector<int>(m + 1));

    deque<pair<int, int>> dq;
    long long prevS = seed;
    for (int i = 0; i < n; i++) {
        dq.clear();

        for (int j = 0; j < m; j++) {
            long long z = (prevS * x + y) % MOD;
            if (z < 0) {
                z += MOD;
            }
            v[i][j] = z;
            prevS = v[i][j];

            if (j - B >= 0) {
                while (!dq.empty() && dq.front().first <= j - B) {
                    dq.pop_front();
                }
            }

            while (!dq.empty() && dq.back().second >= v[i][j]) {
                dq.pop_back();
            }

            dq.push_back({ j, v[i][j] });

            if (j - B + 1 >= 0) {
                v[i][j - B + 1] = dq.front().second;
            }

            pref[i + 1][j + 1] = pref[i][j + 1] + pref[i + 1][j] - pref[i][j] + v[i][j];
        }
    }

    int ans = INF;
    pair<int, int> ansP{ 0, 0 };
    for (int j = 0; j < m - B + 1; j++) {
        dq.clear();

        for (int i = 0; i < n; i++) {
            if (i - A >= 0) {
                while (!dq.empty() && dq.front().first <= i - A) {
                    dq.pop_front();
                }
            }

            while (!dq.empty() && dq.back().second >= v[i][j]) {
                dq.pop_back();
            }

            dq.push_back({ i, v[i][j] });

            if (i - A + 1 >= 0) {
                v[i - A + 1][j] = dq.front().second;

                int x = i + 1;
                int y = j + B;
                int sm = pref[x][y] - pref[x - A][y]
                    - pref[x][y - B] + pref[x - A][y - B];

                int mn = dq.front().second;

                if (sm - mn * A * B < ans) {
                    ans = sm - mn * A * B;
                    ansP = { i - A + 2, j + 1 };
                }
                else if (sm - mn * A * B == ans) {
                    if (make_pair(i - A + 2, j + 1) < ansP)
                        ansP = { i - A + 2, j + 1 };
                }
            }
        }
    }

    cout << ans << '\n';
    cout << ansP.first << ' ' << ansP.second;
}

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
    //freopen("basis.in", "r", stdin);
    //freopen("basis.out", "w", stdout);
    int t = 1;
    //cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}
```
---