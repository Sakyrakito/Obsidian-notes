# Дата - 2025-09-03

[Ссылка на задачу](https://codeforces.com/contest/2107/problem/D)

Tags: #спортивное_программирование #bfs #графы #деревья #жадные_алгоритмы #реализация

---
## О чём задача:
Дано дерево. Мы можем выбрать любые 2 вершины *u* и *v* посчитать между ними расстояние *d* и пометить все вершины от *u* до *v* заблокированными. По заблокированным вершинам нельзя ходить, значит между *u* и *v* не должно их быть изначально. Потом мы выписываем тройку значений *{ d, u, v }*. Наша задача в том, чтобы пройтись так по всем вершинам и выписать все тройки, пока есть не заблокированные вершины. Но надо не просто выписать, а сделать так, чтобы все они были лексикографически максимальны. 

---
## Решение задачи:  
Сначала находим диаметр с максимальными *u*, потом из него запускаем bfs и находим второй конец *v* тоже максимальный. Выписываем это тройку и помечаем все вершины на пути заблокированными. Потом проходим по каждой вершине на пути и рекурсивно вызываем этот алгоритм от всех незаблокированных соседей. Всё это заносим в приоритетную очередь и оттуда выводим.

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
//#pragma GCC optimize("Ofast,unroll-loops")
//#pragma GCC target("avx,avx2,fma")
#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 3e5 + 3, INF = 1e18, MAXA = 1e6 + 3, MOD = 1e9 + 7;
const long double PI = acos(-1);

vector<int> par, dist;
vector<bool> blocked;
map<tuple<int, int, int>, vector<tuple<int, int, int>>> child;
vector<vector<int>> g;

tuple<int, int, int> getAns(int start) {
    vector<int> used, q;
    // bfs
    dist[start] = 1;
    par[start] = 0;
    q.push_back(start);
    used.push_back(start);
    for (int i = 0; i < q.size(); i++) {
        for (auto v : g[q[i]]) {
            if (!blocked[v] && dist[v] == 0) {
                dist[v] = dist[q[i]] + 1;
                q.push_back(v);
                used.push_back(v);
                par[v] = q[i];
            }
        }
    }

    int u = q[0];
    for (int x : q)
        if (dist[x] > dist[u] || (dist[x] == dist[u] && x > u))
            u = x;

    for (int x : used)
        dist[x] = par[x] = 0;
    used.clear();
    q.clear();

    // bfs
    dist[u] = 1;
    q.push_back(u);
    used.push_back(u);
    par[u] = 0;
    for (int i = 0; i < q.size(); i++) {
        for (auto v : g[q[i]]) {
            if (!blocked[v] && dist[v] == 0) {
                dist[v] = dist[q[i]] + 1;
                par[v] = q[i];
                used.push_back(v);
                q.push_back(v);
            }
        }
    }

    int w = q[0];
    for (int x : q) {
        if (dist[x] > dist[w] || (dist[x] == dist[w] && x > w))
            w = x;
    }

    vector<int> path;
    for (int cur = w; cur; cur = par[cur])
        path.push_back(cur);
    for (int x : used)
        dist[x] = par[x] = 0;

    int ln = path.size();
    int maxEnd = max(u, w);
    int minEnd = (maxEnd == u ? w : u);
    
    for (int x : path)
        blocked[x] = true;

    tuple<int, int, int> res = { ln, maxEnd, minEnd };
    for (int x : path) {
        for (int v : g[x]) {
            if (!blocked[v]) {
                tuple<int, int, int> chil = getAns(v);
                child[res].push_back(chil);
            }
        }
    }

    return res;
}

void solve() {
    int n;
    cin >> n;
    g.assign(n + 1, {});
    blocked.assign(n + 1, false);
    par.assign(n + 1, 0);
    dist.assign(n + 1, 0);
    child.clear();

    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }

    tuple<int, int, int> ans = getAns(1);
    priority_queue<tuple<int, int, int>> pq;
    pq.push(ans);

    while (!pq.empty()) {
        auto [ln, mx, mn] = pq.top();
        pq.pop();
        cout << ln << ' ' << mx << ' ' << mn << ' ';
        for (auto ch : child[{ln, mx, mn}])
            pq.push(ch);
    }
    cout << '\n';
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