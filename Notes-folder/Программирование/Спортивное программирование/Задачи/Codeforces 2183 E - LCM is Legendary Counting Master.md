# Дата - 2026-01-18

[Ссылка на задачу]([https://codeforces.com/contests](https://codeforces.com/contest/2183/problem/E))

Tags: #спортивное_программирование #дп #математика #комбинаторика #теория_чисел 

---
## О чём задача:

Дан массив $a$ длинны $n$, где каждый элемент - целое число из отрезка $[0, m]$. Он называется хорошим, если выполняются эти 2 условия:
1. $a_{1} < a_{2} < a_{3} < \dots <a_{n}$
2. $\frac{1}{lcm(a_{1},a_{2})} + \frac{1}{lcm(a_{2}, a_{3})} + \dots + \frac{1}{lcm(a_{n - 1}, a_{n})} + \frac{1}{lcm(a_{n}, a_{1})} \geq 1$

Мы должны посчитать количество способов заменить все 0 в $a$ таким образом, чтобы массив стал хорошим.

$$
2 \leq n \leq m \leq 3000
$$
$$
0 \leq a_{i} \leq m
$$

---
## Решение задачи:  

Так как $lcm(x, y) = \frac{x * y}{gcd(x, y)}$, то $\frac{1}{lcm(x, y)} = \frac{gcd(x, y)}{x*y}$.

Теперь у нас условие 2:
$\frac{gcd(a_{1}, a_{2})}{a_{1}*a_{2}} + \frac{gcd(a_{2},a_{3})}{a_{2}*a_{3}} + \dots + \frac{gcd(a_{n - 1}, a_{n})}{a_{n - 1}, a_{n}} + \frac{gcd(a_{n}, a_{1})}{a_{n} * a_{1}} \geq 1$

Так как $a_{i} < a_{i + 1}$, то $gcd(a_{i}, a_{i + 1}) \leq a_{i + 1} - a_{i}$ 
Это значит, что $\sum_{i = 1}^{n - 1}\left( \frac{gcd(a_{i}, a_{i + 1})}{a_{i} *a_{i + 1}} \right) + \frac{gcd(a_{n}, a_{1})}{a_{n}*a_{1}} \leq \sum_{i = 1}^{n - 1}\left( \frac{a_{i + 1} - a_{i}}{a_{i} *a_{i + 1}} \right) + \frac{a_{1}}{a_{n}*a_{1}}$
Раскроем правую сторону как $\frac{a_{2} - a_{1}}{a_{2}*a_{1}} = \frac{1}{a_{1}} - \frac{1}{a_{2}}$ и получим: 
$\sum_{i = 1}^{n - 1}\left( \frac{a_{i + 1} - a_{i}}{a_{i} *a_{i + 1}} \right) + \frac{a_{1}}{a_{n}*a_{1}} = \frac{1}{a_{1}}$

Теперь мы знаем, что $\sum_{i = 1}^{n - 1}\left( \frac{gcd(a_{i}, a_{i + 1})}{a_{i} *a_{i + 1}} \right) + \frac{gcd(a_{n}, a_{1})}{a_{n}*a_{1}} \leq \frac{1}{a_{1}}$
То есть наша сумма никогда не может быть больше, чем 1.

Теперь, если мы возьмём $a_{1} \geq 2$, по получим, что наша сумма будет $\leq \frac{1}{2}$, но по условию нам нужно $\geq 1$, а значит $a_{1} = 1$ и ничего другого.

Мы использовали $gcd(a_{i}, a_{i + 1}) \leq a_{i + 1} - a_{i}$, соответственно $\frac{gcd(a_{i}, a_{i + 1})}{a_{i} *a_{i + 1}} \leq \frac{a_{i + 1} - a_{i}}{a_{i + 1} * a_{i}}$, но чтобы наша сумма была равна 1, то должно выполняться $gcd(a_{i}, a_{i + 1}) = a_{i + 1} - a_{i}\ для\ всех\ i<n$.

$gcd(a_{i}, a_{i + 1})$ - это какой-то делитель $a_{i}$, обозначим его как $d$. Тогда выражение приобретает вид: $d = a_{i + 1} - a_{i} \to a_{i + 1} = a_{i} + d$, то есть следующий элемент - это предыдущий + делитель предыдущего.

По сути так мы считаем ответ для каждого элемента и в конце будет наш ответ. Будем считать это с помощью дп.

Заведём $dp[i][x]$, где $i$ - длинна текущего отрезка, $x$ - элемент, на который заканчивается наш отрезок.
Переход: $dp[i + 1][x +d] +=dp[i][x]$, где $d$ - это делитель числа $x$. 

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
    int n, m;
    cin >> n >> m;
    vector<int> a(n);
    for (int i = 0; i < n; i++) cin >> a[i];

    // первый элемент должен быть 1
    if (a[0] > 1) { 
        cout << "0\n";
        return;
    }

    vector<vector<int>> dp(n, vector<int>(m + 1));
    dp[0][1] = 1; // длинны 1 только 1 способ - это первый элемент 1
    for (int i = 0; i < n - 1; i++) {
        // если текущий элемент не 0, то мы смотрим только его делители
        if (a[i] != 0) {
            int x = a[i];

            for (int d = 1; d * d <= x; d++) {
                if (x % d != 0)
                    continue;

                int d1 = d;
                int d2 = x / d;

                if (x + d1 <= m) {
                    dp[i + 1][x + d1] += dp[i][x];
                    dp[i + 1][x + d1] %= MOD;
                }

                if (d2 != d1 && x + d2 <= m) {
                    dp[i + 1][x + d2] += dp[i][x];
                    dp[i + 1][x + d2] %= MOD;
                }
            }
            continue;
        }

        // если текущий элемент 0, то мы смотрим варианты, когда мы заменили 0 на число x
        for (int x = 1; x <= m; x++) {
            if (dp[i][x] == 0)
                continue;

            for (int d = 1; d * d <= x; d++) {
                if (x % d != 0)
                    continue;

                int d1 = d;
                int d2 = x / d;

                if (x + d1 <= m) {
                    dp[i + 1][x + d1] += dp[i][x];
                    dp[i + 1][x + d1] %= MOD;
                }

                if (d2 != d1 && x + d2 <= m) {
                    dp[i + 1][x + d2] += dp[i][x];
                    dp[i + 1][x + d2] %= MOD;
                }
            }
        }
    }

    // если последний элемент не 0, то смотрим сколько способов в него прийти
    // иначе складываем все возможные варианты последнего элемента
    int ans = 0;
    if (a[n - 1] != 0)
        ans = dp[n - 1][a[n - 1]];
    else
        for (int i = 1; i <= m; i++)
            ans = (ans + dp[n - 1][i]) % MOD;
    cout << ans << '\n';
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