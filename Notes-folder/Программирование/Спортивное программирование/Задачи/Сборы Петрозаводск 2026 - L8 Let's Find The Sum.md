# Дата - 2026-02-07

Tags: #спортивное_программирование #математика

---
## О чём задача:

Посчитать:

$$
\sum_{i=1}^{n}{\sum_{j=1}^{n}{\left\lfloor \frac{n}{i*j} \right\rfloor}}
$$

$$
1 \leq n \leq 10^9
$$

Количество тестов $t \leq 100$.

---
## Решение задачи:  

Простой перебор не зайдёт, от за $10^{18}$ работает. Воспользуемся известным фактом:

$$
\large \left\lfloor \frac{n}{x} \right\rfloor = k
$$

\- это равно количеству целых k таких, что 

$$
\large x * k \leq n
$$

Применим это к $\large x = ij$:

$$
\large \left\lfloor \frac{n}{ij} \right\rfloor = количеству\ k \geq 1\ таких, что\ ijk \leq n
$$

Зафиксируем $k$.
Тогда остаётся посчитать количество пар $(i, j)$ таких, что:

$$
\large ij \leq \left\lfloor \frac{n}{k} \right\rfloor 
$$

Обозначим:

$$
\large m = \left\lfloor \frac{n}{k} \right\rfloor
$$

Тогда число подходящих пар:

$$
\large f(m) = \sum_{i=1}^{m}{\left\lfloor \frac{m}{i} \right\rfloor }
$$

Перепишем всю сумму:

$$
\large S(n) = \sum_{k = 1}^{n}{f\left( \left\lfloor \frac{n}{k} \right\rfloor  \right)}
$$

Но тут всё ещё $n$ слагаемых, нужно ускорять.
Заметим, что 

$$
\large v = \left\lfloor \frac{n}{k} \right\rfloor 
$$

принимает всего $\sqrt{n}$ различных значений.
Тогда:

$$
\large k \in \left[ \left\lfloor \frac{n}{v + 1} \right\rfloor + 1, \left\lfloor \frac{n}{v} \right\rfloor   \right] 
$$

А количество таких $k:cnt = r - l + 1$.
И теперь мы можем перебирать не все значения, а только $\sqrt{n}$ и для каждого считать отрезок, на котором оно находиться. Типа:

```C++
int ans = 0;
for (int l = 1; l <= n; ) {
    int v = n / l;
	int r = n / v;
	ans += (r - l + 1) * f(v);
	l = (r + 1);
}
return ans;
```

Таким же образом мы можем считать $f(v)$, но тогда у нас решение не зайдёт по времени. Его асимптотика будет примерно $O(n^{2/3})$, но константа у него слишком большая, поэтому нужно как-то оптимизировать функцию $f(v)$.

Во-первых давайте используем мемоизацию, то есть будем запоминать ответ для каждого $v$, и если мы до этого его уже считали, то не будем пересчитывать.

Рассмотрим все точки $(i, j)$, где:

$$
\large
\begin{align}
i \geq 1 \\
j \geq 1 \\
i*j\leq n
\end{align}
$$

Это [[Программирование/Спортивное программирование/Задачи/Visualisation for problems/Сборы Петрозаводск 2026 - L8 Let's Find The Sum|точки под гиперболой]]. Обозначим

$$
\large sq = sqrt(n)
$$

В квадрата $sq * sq$ все точки подходят и их количество равно площади этого квадрата.
Осталось посчитать точки справа от квадрата и сверху от квадрата. Они симметричны, поэтому считаются вдвое.

Фиксируем $i \leq sq$. 
Для данного $i$:
- Максимальный $j = \left\lfloor  \frac{n}{i}  \right\rfloor$
- из них $sq$ уже учтены в квадрате
Новых точек:

$$
\large \frac{n}{i} - sq
$$

Такие точки есть справа и сверху, поэтому умножаем это выражение на 2.
Итоговая формула:

$$
\large \sum_{i=1}^{n}{\left\lfloor \frac{n}{i} \right\rfloor} = \underbrace{sq^2}_{квадрат} + 2 * \sum_{i=1}^{sq}{\left( \left\lfloor \frac{n}{i} - sq \right\rfloor  \right)}
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
#include<cstring>
//#pragma GCC optimize("Ofast,unroll-loops")
//#pragma GCC target("avx,avx2,fma")
#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 3e5 + 3;
const int INF = 1e18;
const int MAXA = 3e6 + 3;
const int MOD = 1e9 + 7;

const double PI = acosl(-1ll);

unordered_map<int, int> memo;

int calc(int n) {
    if (memo.count(n))
        return memo[n];

    int sq = sqrtl(n);
    int res = sq * sq;

    for (int i = 1; i <= sq; i++) {
        res += 2 * (n / i - sq);
    }

    return memo[n] = res;
}

void solve() {
    int n;
    cin >> n;

    int ans = 0;
    for (int l = 1; l <= n;) {
        int v = n / l;
        int r = n / v;

        ans += (r - l + 1) * calc(v);

        l = (r + 1);
    }

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