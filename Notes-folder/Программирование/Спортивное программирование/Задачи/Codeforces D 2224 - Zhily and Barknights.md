# Дата - 2026-06-20

[Ссылка на задачу](https://codeforces.com/contest/2224/problem/D?locale=en)

Tags: #спортивное_программирование #теория_чисел #бинарный_поиск #математика #сортировки

---
## О чём задача:

Даны 2 массива $a$ и $b$, содержащие по $n$ целых положительных чисел. Пусть $b'$ - перестановка массива $b$, выбранная равномерно случайным образом из всех $n!$ возможных перестановок. Определим $c_{i} = a_{i} * b'_{i}$ для $1\leq i\leq n$

Найти ожидаемое число инверсий массива $c$. Ответ вывести по модулю 998 244 353

$$
1 \leq n \leq 2000
$$

$$
1 \leq a_{i}, b_{i} \leq 10^9
$$

---
## Решение задачи:  

Для начала что такое инверсии. Это пара индексов $i, j$, такие, что $i < j$ и $a_{i} > a_{j}$. То есть левый элемент больше правого.

Нам по сути нужно найти количество пар индексов $i, j, k, m$, такие, что $a_{i} * b_{k} > a_{j} * b_{m}$ и $i < j$ и $k \neq m$.
Давайте преобразуем это выражение так, чтобы элементы одинаковых массивов были в одинаковый сторонах, а именно:

$$
\frac{a_{i}}{a_{j}} > \frac{b_{m}}{b_{k}}
$$

Так как $n \leq 2000$, то обе части можно посчитать за $O(n^2)$.
Просто переберём все возможные пары индексов и запишем в 2 разных массива.

Пусть все возможные числа из левой части неравенства мы записали в массив $f$, а из правой части в массив $s$. 
Отсортируем массив $s$ по возрастанию.
Будем перебирать все числа из массива $f$ и бинарным поиском найдём количество элементов в массиве $s$, которые меньше чем $f_{i}$. Пусть их будет $kol$, тогда к ответу мы должны добавить $kol * (n - 2)!$.

Почему умножаем на $(n - 2)!$? Пусть мы зафиксировали индексы $m$ и $k$. тогда количество перестановок массива $b$, где эти элементы зафиксированы равно $(n - 2)!$.

Но стоит обратить внимание, что мы не можно просто поделить числа и хранить вещественные, так как будет неправильно работать бинарный поиск и сравнение. Поэтому нужно хранить числа в виде дробей и написать отдельный компаратор, который будет их сравнивать.

--- 
## Код:

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
#include <functional>
#define int long long
using namespace std;

const int MAXN = 2e5 + 3;
const int MOD = 998'244'353;
const int INF = 1e18;
const double eps = 1e-9;

int binpow(int x, int n) {
    int res = 1 % MOD;
    while (n > 0) {
        if (n % 2)
            res = res * x % MOD;
        x = x * x % MOD;
        n /= 2;
    }
    return res;
}

int inverse_element(int x) {
    return binpow(x, MOD - 2);
}

int divide(int a, int b) {
    return a * inverse_element(b) % MOD;
}

int fact[MAXN + 1], ifact[MAXN + 1];

void fill_fact() {
    fact[0] = 1;
    for (int i = 1; i <= MAXN; i++) {
        fact[i] = fact[i - 1] * i % MOD;
    }
}

void fill_ifact() {
    ifact[MAXN] = binpow(fact[MAXN], MOD - 2);
    for (int i = MAXN - 1; i >= 0; i--) {
        // 1/i! = (i+1) * 1/(i+1)!
        ifact[i] = ifact[i + 1] * (i + 1) % MOD;
    }
}

int C(int n, int k) {
    if (k < 0 || k > n || n < 0) return 0;
    return fact[n] * ifact[k] % MOD * ifact[n - k] % MOD;
}

int A(int n, int k) {
    if (k < 0 || k > n || n < 0) return 0;
    return fact[n] * ifact[n - k] % MOD;
}

int slow(vector<int> a, vector<int> b) {
    int n = a.size();

    vector<int> p(n);
    for (int i = 0; i < n; i++) p[i] = i;

    int inv = 0;
    int cnt = 0;

    vector<int> c(n);

    do {
        cnt++;

        for (int i = 0; i < n; i++) {
            c[i] = a[i] * b[p[i]];
        }

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (c[i] > c[j]) {
                    inv++;
                    inv %= MOD;
                }
            }
        }

    } while (next_permutation(p.begin(), p.end()));

    return divide(inv, fact[n]);
    //return inv;
}

struct Fraction {
    int x;
    int y;

    bool operator<(const Fraction& other) const {
        return x * other.y < other.x * y;
    }
};

void solve() {
	int n; cin >> n;
	vector<int> a(n);
	for (int i = 0; i < n; i++) {
		cin >> a[i];
	}
	vector<int> b(n);
	for (int i = 0; i < n; i++) {
		cin >> b[i];
	}

    vector<Fraction> f, s;
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            f.push_back({ a[i], a[j] });
        }
    }

    for (int i = 0; i < b.size(); i++) {
        for (int j = 0; j < b.size(); j++) {
            if (i == j)
                continue;

            s.push_back({ b[i], b[j] });
        }
    }

    sort(s.begin(), s.end());

    int ans = 0;
    for (auto x : f) {
        int ind = lower_bound(s.begin(), s.end(), x) - s.begin();

        ans += ind;
        ans %= MOD;
    }
    ans *= fact[max(n - 2, 0ll)];
    ans %= MOD;

    /*int z = slow(a, b);
    cout << "slow: " << z << '\n';
    cout << "ans: " << ans << '\n';*/

    assert(ans >= 0);

    cout << divide(ans, fact[n]) << '\n';
}

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
    //freopen("generation.in", "r", stdin);
    //freopen("generation.out", "w", stdout);
    fill_fact();
    fill_ifact();
    int t;
    t = 1;
    cin >> t;
    while (t--)
        solve();
    return 0;
}

```
---