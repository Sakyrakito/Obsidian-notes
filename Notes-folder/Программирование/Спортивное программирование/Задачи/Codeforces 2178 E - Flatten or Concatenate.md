# Дата - 2025-12-30

[Ссылка на задачу](https://codeforces.com/contest/2178/problem/E)

Tags: #спортивное_программирование #бинарный_поиск #интарактив

---
## О чём задача:

Есть 2 массива $a$ и $b$, которые изначально состоят из 1 элемента $2^k$.
К этим массивам мы применили произвольное (возможно 0) количество операций следующих типов:
1. Сглаживание - выбрать произвольный массив, в нём максимальный элемент $x$, удалить его и на его индекс записать 2 элемента $\frac{x}{2}$.
2. Конкатенация - объединить 2 массива ($a + b$), то есть записать сначала элементы из $a$, потом из $b$ и заменить исходный массивы на получившийся.

После всех операций у нас получился какой-то массив и нам нужно узнать, какой элемент максимальный в нём.

У нас есть 300 запросов, на каждый запрос мы даём отрезок $[l, r]$ и получаем сумму на этом отрезке.

Нам дано $n$ - длинна окончательного массива.
$1\leq n\leq 10^5$

---
## Решение задачи:  

Заметим, что после применения операции первого типа сумма нашего массива не измениться, а после применения второго типа она увеличиться в 2 раза.

Давайте забинарим то, где мы объединили массивы. Ну то есть найдём отрезок, где сумма будет равна половине суммы массива. У нас получились отрезки $[l, mid]$ и $[mid + 1, r]$. Их суммы равны, а значит максимальный элемент будет лежать в отрезке, длинна которого меньше. Мы выбрали такой отрезок и для него снова запускаем бинарный поиск и т.д., пока размер отрезка не будет равен 1. 

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

int ask(int l, int r) {
    cout << "? " << l << ' ' << r << endl;
    int sm;
    cin >> sm;
    return sm;
}

int bs(int ll, int rr, int sm) {
    int l = ll, r = rr, ans = ll;
    while (l <= r) {
        int mid = (l + r) / 2;
        
        int sm1 = ask(ll, mid);
        int sm2 = sm - sm1;

        if (sm1 == sm2) {
            ans = mid;
            break;
        }
        else if (sm1 > sm2) {
            r = mid - 1;
        }
        else {
            l = mid + 1;
        }
    }

    return ans;
}

void solve() {
    int n; cin >> n;
    int sm = ask(1, n);

    int l = 1, r = n, ans = -1;
    while (l <= r) {
        if (l == r) {
            ans = sm;
            break;
        }

        int ind = bs(l, r, sm);
        int ln1 = ind - l + 1;
        int ln2 = (r - l + 1) - ln1;

        if (ln1 < ln2) {
            r = ind;
        }
        else
            l = ind + 1;
        sm /= 2;
    }

    cout << "! " << ans << endl;
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