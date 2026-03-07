# Дата - 2026-01-23

[Ссылка на задачу](https://codeforces.com/contest/2181/problem/G)

Tags: #спортивное_программирование #бинарный_поиск #математика

---
## О чём задача:

Дан массив $a$ размера $n$. За каждый ход мы можем выбрать максимум $(n - 1)$ различных элементов и отнять от i-го и ((i + 1) % n)-го единицу (выбрали 2, 3, 4 - значит отнимаем от 2, 3, 3, 4, 4, 5). Элементы записаны по кругу.

За какое минимальное количество раундом мы сможем каждый элемент в $a$ сделать равным 0?

Ответ всегда существует.

$$
1 \leq t \leq 10^4
$$
$$
2 \leq n \leq 5* 10^5
$$
$$
0 \leq a_{i} \leq 10^9
$$

---
## Решение задачи:  

Запишем вектор $(x_{1}, x_{2}, x_{3}, \dots, x_{n})$ - количество раз, которые мы выбрали i-ый элемент. 
Теперь можем записать это в виде системы линейных уравнений:

$$
\begin{cases}
x_{1} +x_{n} = a_{1} \\
x_{1} + x_{2} = a_{2} \\
x_{2} + x_{3} = a_{3} \\
\dots \\
x_{n - 1} + x_{n} = a_{n}
\end{cases}
$$

Решим эту систему линейных уравнений в виде матриц:

$$
\begin{pmatrix}
1, 0, 0, \dots, 1 \\
1, 1, 0, \dots, 0 \\
0, 1, 1, \dots, 0 \\
\dots \\
0, 0, \dots, 1, 1
\end{pmatrix}
*
\begin{pmatrix}
x_{1} \\
x_{2} \\
x_{3} \\
\dots \\
x_{n}
\end{pmatrix} =
\begin{pmatrix}
a_{1} \\
a_{2} \\
a_{3} \\
\dots \\
a_{n}
\end{pmatrix}
$$

Запишем это в виде расширенной матрицы и решим методом Гаусса. Сначала применим прямой ход Гаусса, а потом обратный, чтобы только на главной диагонали были нули.
И там получится, что есть $n \% 2 =1$, то система имеет единственное решение, иначе у нас $x_{n}$ - это свободный член и мы должны забинарить его так, чтобы минимизировать максимальный $x_{i}$. Там получится:

$$
\begin{cases}
x_{1} = b_{1} - x_{n} \\
x_{2} = b_{2} + x_{n} \\
x_{3} = b_{3} - x_{n} \\
x_{4} = b_{4} + x_{n} \\
\dots \\
x_{n - 1} = b_{n - 1} - x_{n}
\end{cases}
$$

Где $b_{i}$ - свободный член после обратного прохода Гаусса

После того, как мы знаем каждый $x_{i}$, то мы можем посчитать ответ на задачу. Так как мы не можем выбирать каждый элемент больше 1-го раза, то ответ как минимум $max(x_{i})$. 
Но так же мы не можем выбирать больше, чем $(n - 1)$ элемент на каждом ходу, поэтому ответ:

$$
ans = max
\begin{cases}
max(x_{i}) \\
\Large \left\lceil \frac{\sum(x_{i})}{n - 1}  \right\rceil
\end{cases}
$$

--- 
## Код:

```C++
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
//#pragma GCC optimize("Ofast,unroll-loops")
//#pragma GCC target("avx,avx2,fma")
#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 3e5 + 3, INF = 2e18, MAXA = 1e6 + 3, MOD = 998'244'353;
const double PI = acos(-1);

void solve() {
    int n; cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; i++)
        cin >> a[i];

    if (n == 2) {
        cout << a[0] << '\n';
        return;
    }

	// прямой ход Гаусса
    for (int i = 1; i < n; i++)
        a[i] -= a[i - 1];

    vector<int> b(n);
    b[n - 1] = a[n - 1] / 2;

	// обратный ход Гаусса
    for (int i = 0; i < n - 1; i++)
        if (i % 2 == 0)
            b[i] = a[i] - b[n - 1];
        else
            b[i] = a[i] + b[n - 1];
    
    if (n % 2 == 1) {
        int ans = 0;
        for (int i = 0; i < n; i++) {
            ans = max(ans, b[i]);
        }
        int sm = accumulate(b.begin(), b.end(), 0ll);
        ans = max(ans, (sm + n - 2) / (n - 1));
        cout << ans << '\n';
    }
    else {
        auto can = [&](long long M) -> bool {
            long long L = -INF, R = INF;

            for (int i = 0; i < n; i++) {
                if (i % 2 == 0) {
                    L = max(L, b[i] - M);
                    R = min(R, b[i]);
                }
                else {
                    L = max(L, -b[i]);
                    R = min(R, M - b[i]);
                }
                if (L > R) return false;
            }
            return true;
        };

        long long lo = 0, hi = INF;
        long long ans = -1;

        while (lo <= hi) {
            long long mid = (lo + hi) / 2;
            if (can(mid)) {
                ans = mid;
                hi = mid - 1;
            }
            else {
                lo = mid + 1;
            }
        }

        int xn = 0;
        for (int i = 0; i < n; i++)
            if (i % 2 == 0)
                xn = max(xn, b[i] - ans);
            else
                xn = max(xn, -b[i]);

        int res = 0, sm = 0;
        for (int i = 0; i < n; i++) {
            if (i % 2 == 0)
                b[i] = b[i] - xn;
            else
                b[i] = b[i] + xn;
            res = max(res, b[i]);
            sm += b[i];
        }
        res = max(res, (sm + n - 2) / (n - 1));
        cout << res << '\n';
    }
}

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0); //cout.tie(0);
    //freopen("basis.in", "r", stdin);
    //freopen("basis.out", "w", stdout);
    int t;
    t = 1;
    cin >> t;
    while (t--)
        solve();
    return 0;
}
```
---
