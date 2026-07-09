# Дата - 2026-07-09

[Ссылка на задачу](https://codeforces.com/contest/2238/problem/C)

Tags: #спортивное_программирование #деревья #дп 

---
## О чём задача:

Дано дерево размера $n$ с корнем в вершине 1. Для каждой вершины $v$ от 1 до n рассмотрим её поддерево.
Для каждого значения $h$ рассмотрим множество вершин в поддереве $v$, которые расположены на расстоянии ровно $h$ от $v$. Такое множество мы называем **гильдией**. Например, для $h = 0$ гильдия состоит только из вершины $v$.

Две гильдии считаются разными, если есть вершина, которая входит в одну гильдию, но не входит в другую. 

Мы хотим узнать сколько существует разных не пустых гильдий в этом дереве. То есть для каждой $v$ от 1 до $n$ посчитать количество различных гильдий в её поддереве и в конце сложить ответ по всем вершинам.

$$
2 \leq n \leq 2 * 10^5
$$

---
## Решение задачи:  

Давайте для каждой вершины $v$ посчитает расстояние от корня до неё, пусть это будет $depth_{v}$.
Теперь для каждой вершины $v$ найдём самую глубокую вершину в её поддереве, запишем это значение как $maxH_{v}$.

$$
\LARGE
maxH_{v} = depth_{v}
$$

если это лист, иначе:

$$
\LARGE
maxH_{v} = \underset{to \in child(v)}{max(maxH_{to})}
$$

Для каждой вершины $v$ у нас есть 2 типа гильдий:
1. Полностью лежат в каком-то ребёнка
2. Лежат в нескольких детях

Нам нужно посчитать количество гильдий второго типа и прибавить их к ответу. Гильдии первого типа мы уже посчитали, когда считали ответ для детей.

Давайте посмотрим на все $maxH_{to}$. Среди них возьмём 2 самых больших. Пусть максимум из них равен $x$, а минимум $y$. Тогда количество вершин первого типа равно $x - y$. Так как все эти вершины полностью лежат в одном поддереве. Остальные вершины у нас лежат как минимум в двух разных детях, это значит, что они принадлежат второму типу и мы можем объединить их в новую гильдию.
А количество таких вершин - это $y - depth_{v}$ и это значение мы должны прибавить к ответу.

Таким образом ответ получается так:

$$
\LARGE
ans_{v} = 1(сама\ вершина\ v) + (y - depth_{v}) + \sum_{to \in child(v)}{ans_{to}}
$$

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
#include<functional>
#include<random>
#include<ctime>
#include<bit>
#include<unordered_set>
#include<cstring>
#include<chrono>
  
//#pragma GCC optimize("Ofast,unroll-loops")
//#pragma GCC target("avx2")
#define int long long
  
using namespace std;
  
mt19937 rnd(time(NULL));
  
const int MAXN = 5e5 + 3;
const int INF = 1e18;
const int MAXA = 5e6 + 3;
const int MOD = 998244353;
  
void solve() {
    int n; cin >> n;
    vector<vector<int>> g(n + 1);
    for (int i = 2; i <= n; i++){
        int p; cin >> p;
        g[p].push_back(i);
        g[i].push_back(p);
    }
  
    vector<int> h(n + 1);
    vector<int> ans(n + 1);
    vector<int> maxH(n + 1);
    function<void(int, int)> dfs = [&](int u, int p){
        ans[u] = 1;
        maxH[u] = h[u];
  
        for (auto v: g[u]){
            if (v == p){
                continue;
            }
  
            h[v] = h[u] + 1;
            dfs(v, u);
  
            maxH[u] = max(maxH[u], maxH[v]);
            ans[u] += ans[v];
        }
  
        vector<int> vc;
        for (auto v: g[u]){
            if (v == p){
                continue;
            }
  
            vc.push_back(maxH[v] - h[u]);
        }
  
        sort(vc.begin(), vc.end());
  
        if (vc.size() >= 2){
            ans[u] += vc[vc.size() - 2];
        }
    };
  
    dfs(1, 0);
  
    cout << ans[1] << '\n';
}
  
signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
    //freopen("basis.in", "r", stdin);
    //freopen("basis.out", "w", stdout);
    int t = 1;
    cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}
```
---