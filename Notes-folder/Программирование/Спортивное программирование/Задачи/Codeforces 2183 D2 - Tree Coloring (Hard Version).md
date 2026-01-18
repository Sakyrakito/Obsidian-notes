# Дата - 2026-01-17

[Ссылка на задачу](https://codeforces.com/contest/2183/problem/D2)

Tags: #спортивное_программирование #графы #деревья #конструктив #жадные_алгоритмы 

---
## О чём задача:

Дано дерево с корнем в 1. Нам нужно покрасить каждую из вершин. В каждый момент времени мы выбираем подмножество вершин и красим его если для любой пары вершин из подмножества $(u, v)$ выполняется:
1. Между вершинами $u$ и $v$ нет ребра
2. Эти вершины находятся на разном расстоянии от корня

Нужно вывести минимальное время, за которое покрасим дерево и в каждый момент времени подмножество вершин, которое красим.

$$
Количество\ наборов\ входных\ данных\ 1\leq t\leq 10^4
$$
$$
Количество\ вершин\ 2 \leq n \leq 2\ \cdot\ 10^5
$$

---
## Решение задачи:  

Для начала найдём сколько нам потребуется времени.

Запустим bfs/dfs и найдём расстояние от корня до каждой вершины. Так как мы не можем выбирать вершины, которые лежат на одной глубине, то ответ как минимум $cnt[d[i]]$, где $cnt$ - количество вершин на глубине $d[i]$. 
Но мы так же не можем выбирать вершины, которые имеют ребро между собой, поэтому пройдём по каждой вершине и посмотрим сколько у неё соседей. Чтобы удалить вершину и всех её соседей, то уйдёт $g[i].size()$ операций, где $g[i]$ - список соседей вершины $i$. 
Но так как у вершины 1 нет предка, то для неё ответ $g[1].size() + 1$.
Из всех этих значений взять максимум и это будет минимально нужное время.

Теперь восстановим ответ.

В первом запросе мы выбираем вершину 1. Теперь будем проходить по каждой глубине и смотреть все вершины на ней. Будем жадно раскидывать вершины в запросы. 
Если мы не можем закинуть вершину $v$ в i-ый запрос то это значит, то последняя добавленная вершина в этот запрос - это предок нашей текущей вершины, значит мы делаем:
1. Если у нас есть ещё хотя бы 1 запрос, который мы не посетили, то кидаем нашу вершину туда, так как там 100% не её предок и там нет вершины с такой же глубиной.
2. Иначе мы смотрим на все вершины текущей глубины и смотрим то, куда мы её добавили. Пусть мы смотрим вершину $u$. Если вершина перед $u$ не является предком вершины $v$ и последняя вершина из последнего запроса не является предком $u$, то мы свапаем эти вершины местами и у нас всё норм

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
    vector<vector<int>> g(n + 1);
    for (int i = 1; i < n; i++) {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u); // строим граф
    }

    vector<int> d(n + 1, INF), p(n + 1);
    vector<vector<int>> vInD(n + 1);
    auto bfs = [&]() {
        d[1] = 0;
        queue<int> q;
        q.push(1);
        while (!q.empty()) {
            int u = q.front();
            q.pop();

            vInD[d[u]].push_back(u); // для каждой глубины запоминаем какие на ней вершины

            for (auto v : g[u]) {
                if (d[u] + 1 < d[v]) {
                    d[v] = d[u] + 1; // ищем глубину вершин
                    p[v] = u; // запоминаем предка
                    q.push(v);
                }
            }
        }
        };

    bfs();

    int ans = 0;
    vector<int> cnt(n + 1);
    for (int i = 1; i <= n; i++) {
        cnt[d[i]]++;
        ans = max(ans, cnt[d[i]]);
    }

    for (int i = 2; i <= n; i++) {
        ans = max(ans, (int)g[i].size());
    }

    ans = max(ans, (int)g[1].size() + 1);

    vector<vector<int>> res(ans);

    vector<int> id(n + 1, -1); // для каждой вершины в какой запрос мы её отправили
    res[0].push_back(1); // первая вершина в первый запрос
    id[1] = 0;

    set<int> free;
    for (int i = 0; i < ans; i++)
        free.insert(i); // запросы, куда мы ещё не пихали вершины глубины d[i]

    for (int curD = 1; curD <= n; curD++) {
        for (auto v : vInD[curD]) {
            int newId = *free.begin();
            
            if (newId != id[p[v]]) { // если можем добавить, то добавляем
                id[v] = newId;
                free.erase(newId);
                continue;
            }
            
            if (free.size() > 1) { // если есть хотя бы 2 запроса, то добавляем во второй
                newId = *(++free.begin());
                id[v] = newId;
                free.erase(newId);
                continue;
            }

            id[v] = newId;
            for (auto u : vInD[curD]) { // смотрим все вершины и ищем, с какой можем свапнуть
                if (id[p[u]] != id[v] && id[p[v]] != id[u]) {
                    swap(id[u], id[v]);
                    break;
                }
            }
        }

        for (auto v : vInD[curD]) { // восстанавливаем свободные запросы и ответ
            free.insert(id[v]);
            res[id[v]].push_back(v);
        }
    }

    cout << ans << '\n';
    for (int i = 0; i < ans; i++) {
        cout << res[i].size() << ' ';
        for (auto x : res[i])
            cout << x << ' ';
        cout << '\n';
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