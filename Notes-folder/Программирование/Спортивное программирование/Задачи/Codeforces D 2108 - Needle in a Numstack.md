# Дата - 2025-08-31

[Ссылка на задачу](https://codeforces.com/contest/2108/problem/D?locale=en)

Tags: #спортивное_программирование #интарактив #бинарный_поиск #перебор #реализация

---
## О чём задача:
Даны 2 числа n и k.
Есть 2 массива A и B. О них знаем:
1. |A| + |B| = n
2. |A| >= k && |B| >= k
3. Массивы состоят только из чисел от 1 до k
4. Если взять любые k подряд идущих элементов из массива A, то они все будут различными. Если взять любые k подряд идущих элементов из массива B, то они все также будут различными.
   
Так же дан массив С = А + В (элементы записаны по очереди, сначала из А, потом из В).
Мы можем задать 250 запросов. Каждый вопрос содержит индекс i. В ответ вы получите i-й элемент склеенного массива C.

Надо найти изначальную длину А и В.

---
## Решение задачи:  
Спросим первые k элементов, потом последние k элементов. Это будут массивы А и В, так как они циклически повторяются.

Построим массивы *a* и *b* длинны *n* состоящие из элементов, которые мы нашли на предыдущем шаге и сравним все элементы между k-м и (n - k)-м. Если какие-то из них не равны, то записываем этот индекс в массив *diff*. Потом пускаем по этому массиву [[бинарный поиск]]. Смотрим, например, элемент под индексом *m* и сравниваем его с массивом *A*, если они равны, то это всё ещё массив *А*, значит двигаем левую границу, иначе правую. И в конце проверить, что это единственный способ найти конкатенацию массивов.

--- 
## Код на C++:

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
#include<unordered_set>
//#pragma GCC optimize("Ofast,unroll-loops")
//#pragma GCC target("avx,avx2,fma")
//#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 3e5 + 3, INF = 1e18, MAXA = 1e6 + 3, MOD = 1e9 + 7;
const long double PI = acos(-1);

void solve() {
    int n, k;
    cin >> n >> k;

    auto ask = [](int i) {
        cout << "? " << i << endl;
        int res;
        cin >> res;
        return res;
    };

    static vector<int16_t> a, b, c;
    static vector<int> diff;
    a.resize(n + 1);
    b.resize(n + 1);
    c.resize(n + 1);
    diff.clear();

    auto ans = [&](vector<int> vc) {
        if (vc.size() != 1) {
            int lenA = vc[0];
            int lenB = vc[1];

            for (int i = 1; i <= lenA; i++)
                c[i] = a[i];
            for (int i = lenA + 1; i <= n; i++)
                c[i] = b[i];

            int r = k;

            while (r + 1 <= n - k && c[r + 1] == c[r - k + 1])
                r++;
           
            int l = n - k + 1;
            while (l - 1 >= k + 1 && c[l - 1] == c[l + k - 1])
                l--;

            if (r >= l) // есть несколько способов восстановить массивы А и В
                vc = { -1 };
        }

        cout << "! ";
        for (auto x : vc)
            cout << x << ' ';
        cout << endl;
    };

    for (int i = 1; i <= k; i++)
        a[i] = ask(i);
    for (int i = k + 1; i <= n; i++)
        a[i] = a[i - k];

    for (int i = n; i > n - k; i--)
        b[i] = ask(i);
    for (int i = n - k; i >= 1; i--)
        b[i] = b[i + k];

    for (int i = k + 1; i <= n - k; i++)
        if (a[i] != b[i])
            diff.push_back(i);

    if (diff.size() == 0) {
        if (n == 2 * k)
            ans({ k, k });
        else
            ans({ -1 });
    }
    else {
        int l = -1, r = diff.size();
        while (r - l > 1) {
            int m = (r + l) / 2;
            int pos = diff[m];
            int x = ask(pos);

            if (x == a[pos])
                l = m;
            else
                r = m;
        }

        int la = k;
        if (l != -1)
            la = diff[l];

        ans({ la, n - la });
    }
}

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
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
