>[!info]
>алгоритм по нахождению максимального потока

```c++
struct Edge {
    int to, cap, rev;
    
    Edge(int _to, int _cap, int _rev) :
        to(_to),
        cap(_cap),
        rev(_rev) 
    {}
};

struct Dinic {
    vector<vector<Edge>> g;
    vector<int> level, it;

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

        for (int i = it[v]; i < g[v].size(); i++) {
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
```