# Дата - 2026-01-19

[Ссылка на задачу](https://codeforces.com/contest/2181/problem/J)

Tags: #спортивное_программирование #математика #теория_вероятности 

---
## О чём задача:

Мы в казино. Есть $n$ автоматов. Для каждого автомата известно $p_{i}$ - шанс выиграть на $i$-том автомате. Если выигрываем, то наша ставка удваивается, а проигрываем - обнуляется.

Мы играем $k$ раундов. Владелец казино равновероятно выбрал автомат, на котором мы будем играть **все раунды**, но мы не знаем, что это за автомат. В каждом раунде мы можем поставить любую сумму денег от 0 до всех имеющихся у нас денег. 

Нужно посчитать математическое ожидание нашего выигрыша после $k$ раундов. Изначально у нас в кармане 1000$.

$$
1 \leq n \leq 100\ 000
$$
$$
1 \leq k \leq 30
$$
$$
0 \leq p_{i} \leq 100
$$

---
## Решение задачи:  

Сначала давайте посчитаем мат. ожидание, если мы всегда будем ставить все деньги. Если мы ставим $x$, то получаем $2 * x$. Тогда мат. ожидание прибыли, если у нас 1 раунд равно: 

$$
M = \sum_{1}^{n}{(2 * x * p_{i} * prob_{i}) - x}
$$

Где $prob_{i}$ - вероятность, что нам выбрали автомат $i$.
Вычитаем $x$ - потому что нам нужно получить только прибыль (разность начального капитала и полученного).

Можем подставить вместо $x$ 1000 и вынести её и если будет $k$ раундов, то формула будет выглядеть так:

$$
M = 1000 * \left( \sum_{1}^{n}{((2 * p_{i})^k * prob_{i})} - 1 \right)
$$

Но эта формула не всегда даёт правильный ответ. В некоторых случаях она может давать отрицательные значения, когда на самом деле можно добиться положительного. Пример:
Если у нас много автоматов с маленьким шансом выигрыша и 1 с большим, то формула даст отрицательное значение. Но если мы в первом раунде поставим 0$, то у нас будет 2 исхода:
1. Мы выиграли. Это значит, что шанс того, что у нас автомат с большим шансом выигрыша повышается, а шанс остальных понижается.
2. Мы проиграли. Это значит, что скорее всего у нас автомат с маленьким шансом выигрыша.

Уже тут становиться интуитивно понятно, то нам не всегда стоит ставить все деньги, а иногда ставить 0.
Тогда как нам теперь посчитать мат. ожидание?
Воспользуемся [[Теорема Баейса|теоремой Байеса]]. Пусть $(W, L)$ - количество побед и поражений соответственно. Тогда нам нужно посчитать вероятность:

$$
P(H_{i}|(W,L)) = \frac{P((W,L)|H_{i}) * P(H_{i})}{P((W,L))}
$$

Где:
$H_{i}$ выбран i-тый автомат.
$P((W,L)|H_{i}) = p_{i}^{W} * (1-p_{i})^{L}$
$P(H_{i}) = \frac{1}{n}$
$P((W, L))$ можно не считать отдельно, так как она равна $\sum P((W,L)|H_{i}) * P(H_{i})$ и посчитается сама собой.

Теперь ответ мы будем строить так:

$$
ans =
  \begin{cases} 
0, \\
allIn(W,L), \\
solve(W+1,L) + solve(W, L+1)
  \end{cases}
$$

Где:
$allIn(W, L)$ - наш профит, если мы будем ставить все деньги при условии, что у нас $W$ побед и $L$ поражений.
$solve(W, L)$ - рекурсивная функция, которая возвращает $allIn$ или $0$.

А наша конечная формула выгладит так:

$$
M = 1000* \left(\sum_{i=1}^n {((2 * p_{i})^k * P(H_{i}|(W,L)))} - 1 \right) * P(W, L)
$$

В конце умножаем на $P(W, L)$ так как это вероятность того, что мы вообще дошли до состояния с $W$ победами и $L$ поражениями.

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
    int n, k;
    cin >> n >> k;
    vector<int> cnt(101);
    for (int i = 0; i < n; i++) {
        int p; cin >> p;
        cnt[p]++;
    }

    vector<pair<double, int>> vc;
    for (int i = 0; i <= 100; i++) {
        if (cnt[i] > 0) {
            vc.push_back({ i / (double)100, cnt[i] });
        }
    }

    int initN = n;
    n = vc.size();
    vector<vector<vector<double>>> pwlh(k + 1, vector<vector<double>>(k + 1, vector<double>(n)));
    vector<vector<double>> pwl(k + 1, vector<double>(k + 1));
    for (int W = 0; W <= k; W++) {
        for (int L = 0; L <= k; L++) {

            for (int i = 0; i < n; i++) {
                auto [pi, ci] = vc[i];
                pwlh[W][L][i] = pow(pi, W) * pow(1 - pi, L) * ci / initN;
                pwl[W][L] += pwlh[W][L][i];
            }
        }
    }

    auto allIn = [&](int W, int L) -> double {
        double res = 0;
        for (int i = 0; i < n; i++) {
            auto [pi, ci] = vc[i];
            res += (pow(2 * pi, k - W - L) - 1) * pwlh[W][L][i] / pwl[W][L];
        }

        return 1000 * res * pwl[W][L];
    };

    map<pair<int, int>, double> cashe;
    function<double(int, int)> getAns = [&](int W, int L) -> double {
        if (W + L == k)
            return (double)0;

        pair<int, int> state(W, L);
        if (cashe.find(state) != cashe.end())
            return cashe[state];

        double res = max({ (double)0, allIn(W, L), getAns(W + 1, L) + getAns(W, L + 1) });

        cashe[state] = res;
        return res;
    };

    cout << fixed << setprecision(12);
    cout << getAns(0, 0) << '\n';
}

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0); //cout.tie(0);
    //freopen("basis.in", "r", stdin);
    //freopen("basis.out", "w", stdout);
    int t;
    t = 1;
    //cin >> t;
    while (t--)
        solve();
    return 0;
}
```
---