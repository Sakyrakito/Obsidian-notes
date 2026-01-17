# Дата - 2026-01-03

[Ссылка на задачу](https://codeforces.com/contest/2182/problem/D)

Tags: #спортивное_программирование #комбинаторика

---
## О чём задача:

Даны $n$ людей, каждый из них приготовил $a[i]\ (1\leq i\leq n)$ подарков и общая коробка подарков $(a[0])$.

Мы должны выстроить челов в такую очередь, что они по порядку будут доставать по одному своему подарку или из общей коробки по порядку и так по кругу. Процесс заканчивается тогда, когда настала очередь какого-то чела, а он не может достать подарок (их нет ни в общей коробки, ни у него).

Наша задача посчитать количество перестановок (очередей), что в конце все подарки будут вынуты ($a[0],\ a[1],\ ...,\ a[n] = 0$).

---
## Решение задачи:  

Пускай всего у нас $S$ подарков. 

Они будут выниматься как $p1,p2,p3,...,pn,p1,p2,...pn,p1,p2$. 
Количество полных циклов - $\frac{S}{n}$
Остаток $rem$: $S\ \%\ n$

Тогда посчитаем количество людей у которых $\frac{S}{n} + 1$ подарков, пусть из будет $k$. Тогда эти люди должны стоять в перестановке первыми, так как они будут забирать остаток подарков.

Если есть человек, у которого количество подарков больше $\frac{S}{n} + 1$, то ответ 0.
Если $k > rem$, то ответ 0.

Тогда первыми у нас стоят $k$ людей, но нам надо добить, чтобы вначале стояло $rem$ людей. Оставшихся мы можем выбрать из $n - k$ челов. Количество способов это сделать равно $C_{n - k}^{rem - k}$. 

Тогда первые $rem$ челов могут стоять как угодно и $n - rem$ тоже как угодно и ответ получается:
$$C_{n - k}^{rem - k} * rem! * (n - rem)!$$

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
#include <functional>
#define int long long
using namespace std;

const int MAXN = 2e5 + 3, MOD = 998244353;

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

void solve() {
    int n; cin >> n;
    vector<int> vc(n + 1);
    int sm = 0;
    for (int i = 0; i <= n; i++) {
        cin >> vc[i];
        sm += vc[i];
    }

    int kol = sm / n;
    int rem = sm % n;
    int cnt = 0;
    for (int i = 1; i <= n; i++) {
        if (vc[i] > kol + 1) {
            cout << "0\n";
            return;
        }
        if (vc[i] == kol + 1)
            cnt++;
    }

    if (cnt > rem) {
        cout << "0\n";
        return;
    }

    cout << (C(n - cnt, rem - cnt) * fact[rem] % MOD * fact[n - rem] % MOD) << '\n';
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