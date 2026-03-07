# Дата - 2026-02-18

Tags: #спортивное_программирование #геометрия #математика #потоки

---
## О чём задача:

На плоскости даны n точек. Каждый $x$ и каждый $y$ уникальный. Мы должны построить $\frac{n}{3}$ треугольников, потом вокруг каждого треугольника делаем описанный прямоугольник и считаем его периметр. Наша цель максимизировать сумму всех периметров прямоугольников.
$n$ делиться на 3.

$$
3 \leq n \leq 99'999
$$

$$
1 \leq x, y \leq n
$$

---
## Решение задачи:  

Периметр прямоугольника считается так:
$P=2 * (x_{max} - x_{min} + y_{max} - y_{min})$

Смотря на это, нам хочется максимизировать $x_{max}$ и минимизировать $x_{min}$, чтобы периметр был как можно больше. Для $y$ тоже самое.

Давайте посортим наши точки по $x$. Теперь каждая из первых $\frac{n}{3}$ точек будет принадлежать разным треугольникам, чтобы максимизировать сумма по $x$, а последние $\frac{n}{3}$ точек тоже будут принадлежать разным треугольникам и таким образом мы максимизируем $x_{max} - x_{min}$. С $y$ тоже самое. Теперь мы можем посчитать суммарный периметр, который у нас будет в конце.

Пусть $k = \frac{n}{3}$, тогда $P = 8 * k * k$.

Теперь как нам сделать треугольники так, чтобы суммарный периметр был равен текущему значению?

Давайте в моменте, когда мы сортили точки по $x$ каждой присвоим тип $rx - (0, 1, 2)$, если это первые $k$ точек, то тип - 0, вторые - 1, последние - 3. Потом по $y$ сделаем тоже самое $ry$.

Теперь каждая точка имеет тип $(rx, ry)$. И нам нужно построить треугольники так, чтобы каждый из них имел точки, которые суммарно имеют тип (0, 1, 2). 

Согласно [[Теорема Биркгофа - фон Неймана|теореме Биркгофа - фон Неймана]] мы всегда можем разделить точки таким образом. Потому что, если мы построим матрицу $(3 * 3)$ и запишем туда значения $(rx, ry)$, то эта матрица будет является двустохастической. 

Но давайте представим пары $(rx, ry)$ в виде двудольного графа, где слева $rx$, а справа $ry$, а ребро между ними, если в матрице есть значение на пересечении $(rx, ry)$. Этот граф будет являться [[Регулярный граф|регулярным]].

Теперь, чтобы построить каждый треугольник давайте добавим исток(S) и сток(T). Теперь проведём рёбра из S в $rx$ пропускной способностью 1 и из $ry$ в T тоже пропускной способностью 1. Будем делать это $k$ раз и на каждом шаге находить максимальный поток в этом графе (он всегда должен быть равен 3) и после каждого запуска посмотрим все рёбра между $(rx, ry)$ которые мы использовали и из них сделаем треугольник. Таким образом каждый треугольник будет содержать типы (0, 1, 2).

Так же задачу можно решить с помощью нахождения максимального паросочетания или простым перебором все возможных перестановок (0, 1, 2) и проверкой, что есть тройка вершин, соответствующих данной перестановке. 

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
#include<chrono>
//#pragma GCC optimize("Ofast,unroll-loops")
//#pragma GCC target("avx,avx2,fma")
#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 3e5 + 3;
const int INF = 1e18;
const int MAXA = 1e7 + 3;
const int MOD = 1e9 + 7;

const double PI = acosl(-1ll);

struct Point {
    int x = 0, y = 0, id = 0;
    int rx, ry;

    Point(int x, int y, int id, int rx, int ry)
        : x(x), y(y), id(id), rx(rx), ry(ry)
    {}

    Point() {}
};

struct Edge {
    int to, cap, rev;

    Edge(int _to, int _cap, int _rev) :
        to(_to),
        cap(_cap),
        rev(_rev)
    {
    }
};

struct Dinic {
    vector<vector<Edge>> g;
    vector<int> level, it;
    bool check[100'000] = { false };

    Dinic(int sz) :
        g(sz),
        level(sz, -1),
        it(sz, 0)
    {}

    void addEdge(int u, int v, int c) {
        g[u].emplace_back(v, c, g[v].size());
        g[v].emplace_back(u, 0, g[u].size() - 1);
    }

    void addUndir(int u, int v, int c) {
        addEdge(u, v, c);
        addEdge(v, u, c);
    }

    bool bfs(int s, int t) {
        fill(level.begin(), level.end(), -1);
        level[s] = 0;
        queue<int> q;
        q.push(s);
        while (!q.empty()) {
            int v = q.front();
            q.pop();

            for (auto& e : g[v]) {
                if (e.cap > 0 && level[e.to] == -1) {
                    level[e.to] = level[v] + 1;
                    if (e.to == t)
                        return true;

                    q.push(e.to);
                }
            }
        }
        return level[t] != -1;
    }

    int dfs(int v, int t, int f) {
        if (v == t)
            return f;

        for (int& i = it[v]; i < g[v].size(); i++) {
            Edge& e = g[v][i];

            if (e.cap > 0 && level[e.to] == level[v] + 1) {
                int p = dfs(e.to, t, min(f, e.cap));

                if (p > 0) {
                    e.cap -= p;
                    g[e.to][e.rev].cap += p;
                    return p;
                }
            }
        }

        return 0;
    }

    int maxFlow(int s, int t) {
        int flow = 0;

        while (bfs(s, t)) {
            fill(it.begin(), it.end(), 0);

            while (int p = dfs(s, t, INF))
                flow += p;
        }

        return flow;
    }
};

void solve() {
    int n; cin >> n;
    
    vector<Point> p(n);
    for (int i = 0; i < n; i++) {
        cin >> p[i].x >> p[i].y;
        p[i].id = i + 1;
    }

    int k = n / 3;

    sort(p.begin(), p.end(), [](Point a, Point b) {return a.x < b.x; });

    for (int i = 0; i < n; i++) {
        p[i].rx = i / k;
    }

    sort(p.begin(), p.end(), [](Point a, Point b) {return a.y < b.y; });

    for (int i = 0; i < n; i++) {
        p[i].ry = i / k;
    }

    vector<int> cells[3][3];
    for (int i = 0; i < n; i++) {
        cells[p[i].rx][p[i].ry].push_back(p[i].id);
    }

    vector<vector<int>> triangles;
    for (int step = 0; step < k; step++) {
        Dinic dinic(8);

        for (int i = 0; i < 3; i++) {
            dinic.addEdge(0, i + 1, 1);
            dinic.addEdge(i + 4, 7, 1);
            
            for (int j = 0; j < 3; j++) {
                if (!cells[i][j].empty()) {
                    dinic.addEdge(i + 1, j + 4, 1);
                }
            }
        }

        int flow = dinic.maxFlow(0, 7);

        vector<int> curTriangle;
        for (int i = 0; i < 3; i++) {
            for (auto& e : dinic.g[i + 1]) {
                if (e.to >= 4 && e.to <= 6 && e.cap == 0) {
                    int ryIdx = e.to - 4;
                    curTriangle.push_back(cells[i][ryIdx].back());
                    cells[i][ryIdx].pop_back();
                    break;
                }
            }
        }

        triangles.push_back(curTriangle);
    }

    cout << 8 * k * k << '\n';
    for (auto tr : triangles) {
        cout << tr[0] << ' ' << tr[1] << ' ' << tr[2] << '\n';
    }

} // теорема Биркгофа — фон Неймана

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