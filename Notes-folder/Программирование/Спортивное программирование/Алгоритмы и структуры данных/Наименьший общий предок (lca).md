>[!info]
>Алгоритм на нахождение наименьшего общего предка в дереве. 
>Быть осторожным, так как тут вершины от 0 до n - 1.
>Вызываем сначала dfs от корня, а lca как l = lca(a, b)

```c++
vector<int> tin(MAXN), tout(MAXN);
int timer, l;
vector < vector<int> > up(MAXN);
vector<int> g[MAXN];

void dfs(int v, int p = 0) {
    tin[v] = ++timer;
    up[v][0] = p;
    for (int i = 1; i <= l; ++i)
        up[v][i] = up[up[v][i - 1]][i - 1];
    for (size_t i = 0; i < g[v].size(); ++i) {
        int to = g[v][i];
        if (to != p)
            dfs(to, v);
    }
    tout[v] = ++timer;
}

bool upper(int a, int b) {
    return tin[a] <= tin[b] && tout[a] >= tout[b];
}

int lca(int a, int b) {
    if (upper(a, b))  return a;
    if (upper(b, a))  return b;
    for (int i = l; i >= 0; --i)
        if (!upper(up[a][i], b))
            a = up[a][i];
    return up[a][0];
}
```