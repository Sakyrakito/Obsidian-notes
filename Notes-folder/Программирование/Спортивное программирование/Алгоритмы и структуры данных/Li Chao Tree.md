>[!info]
>Дерево Ли Чао для поиска прямой, на которой наш x даёт максимальный результат.
>Перед использованием нужно сжать координаты, это можно сделать так:
>sort(x_unique.begin(), x_unique.end());
>x_unique.erase(unique(x_unique.begin(), x_unique.end()), x_unique.end());

```c++

struct Line
{
    int k = 0, b = -INF;

    int eval(int x)
    {
        return k * x + b;
    }
};

class LiChaoTree
{
  public:
    LiChaoTree(vector<int>& x)
    : x_unique(x)
    {
        n = x.size();
        tree.resize(max(1ll, n * 4));
    }

    void add(int k, int b)
    {
        if (n == 0){
            return;
        }
        addTree({k, b}, 1, 0, n - 1);
    }

    int get_max(int x)
    {
        if (n == 0){
            return -INF;
        }

        int idx = lower_bound(x_unique.begin(), x_unique.end(), x) - x_unique.begin();
        return getMaxTree(idx, 1, 0, n - 1);
    }

  private:
    vector<Line> tree;
    vector<int> x_unique;
    int n;

    void addTree(Line cur, int v, int l, int r)
    {
        if (tree[v].b == -INF)
        {
            tree[v] = cur;
            return;
        }

        int mid = (l + r) / 2;

        bool mid_better = (cur.eval(x_unique[mid]) > tree[v].eval(x_unique[mid]));
        bool left_better = (cur.eval(x_unique[l]) > tree[v].eval(x_unique[l]));

        if (mid_better)
        {
            swap(tree[v], cur);
        }

        if (l == r)
        {
            return;
        }

        if (left_better != mid_better)
        {
            addTree(cur, v * 2, l, mid);
        }
        else
        {
            addTree(cur, v * 2 + 1, mid + 1, r);
        }
    }

    int getMaxTree(int x, int v, int l, int r){
        if (tree[v].b == -INF){
            return -INF;
        }

        int res = tree[v].eval(x_unique[x]);

        if (l == r){
            return res;
        }

        int mid = (l + r) / 2;

        if (x <= mid){
            res = max(res, getMaxTree(x, v * 2, l, mid));
        }
        else{
            res = max(res, getMaxTree(x, v * 2 + 1, mid + 1, r));
        }

        return res;
    }
};

```