>[!info]
>Дерево отрезков на:
>Дан массив целых чисел a длины n. Поступает q запросов восьми типов:
> - 1. l r x. Для каждого i на отрезке от l до r включительно нужно заменить ai на min(ai,x).
> - 2. l r x. Для каждого i на отрезке от l до r включительно нужно заменить ai на max(ai,x).
> - 3. l r x. Для каждого i на отрезке от l до r включительно нужно заменить ai на x.
> - 4. l r x. Для каждого i на отрезке от l до r включительно нужно заменить ai на ai+x.
> - 5. l r. Необходимо вывести сумму элементов массива a на отрезке от l до r включительно.
> - 6. l r. Необходимо вывести минимум элементов массива a на отрезке от l до r включительно.
> - 7. l r. Необходимо вывести максимум элементов массива a на отрезке от l до r включительно.
> - 8. l r. Необходимо вывести НОД (наибольший общий делитель) элементов массива a на отрезке от l до r включительно.

>[!warning]
>На счёт взятия по модулю не уверен, не тестил с ним, правильный ли даёт результат

```C++
struct Node {
    int maxv = -INF;
    int cntMax = 0;
    int secondMax = -INF;

    int minv = INF;
    int cntMin = 0;
    int secondMin = INF;

    int diffGcd = 0;
    int sum = 0;
    int add = 0;
    int ln = 0;
    int assignment = -INF;
};

class SegmentTreeBeats {
public:
    SegmentTreeBeats(int n) {
        a.resize(n + 1);
        tree.resize(n * 4);
    }

    SegmentTreeBeats(int n, vector<int>& vc)
        : SegmentTreeBeats(n)
    {
        for (int i = 1; i <= n; i++)
            a[i] = vc[i];
    }

    void build(int v) {
        buildTree(v, 1, a.size() - 1);
    }

    int getMin(int l, int r) {
        return getMinTree(1, l, r, 1, a.size() - 1);
    }

    int getMax(int l, int r) {
        return getMaxTree(1, l, r, 1, a.size() - 1);
    }

    int getSum(int l, int r) {
        return getSumTree(1, l, r, 1, a.size() - 1);
    }

    int getGcd(int l, int r) {
        return getGcdTree(1, l, r, 1, a.size() - 1);
    }

    void updateMin(int x, int l, int r) {
        updateMinTree(1, x, l, r, 1, a.size() - 1);
    }

    void updateMax(int x, int l, int r) {
        updateMaxTree(1, x, l, r, 1, a.size() - 1);
    }

    void updateAdd(int x, int l, int r) {
        updateAddTree(1, x, l, r, 1, a.size() - 1);
    }

    void updateAssignment(int x, int l, int r) {
        updateAssignmentTree(1, x, l, r, 1, a.size() - 1);
    }

    void updateMod(int x, int l, int r) {
        updateModTree(1, x, l, r, 1, a.size() - 1);
    }

private:
    vector<int> a;
    vector<Node> tree;

    void concatenation(int v) {
        tree[v].ln = tree[v * 2].ln + tree[v * 2 + 1].ln;

        tree[v].sum = tree[v * 2].sum + tree[v * 2 + 1].sum;
        
        // ------------------------MAX--------------------------
        if (tree[v * 2].maxv > tree[v * 2 + 1].maxv) {
            tree[v].maxv = tree[v * 2].maxv;
            tree[v].cntMax = tree[v * 2].cntMax;
            tree[v].secondMax = max(tree[v * 2].secondMax, tree[v * 2 + 1].maxv);
        }
        else if (tree[v * 2].maxv < tree[v * 2 + 1].maxv) {
            tree[v].maxv = tree[v * 2 + 1].maxv;
            tree[v].cntMax = tree[v * 2 + 1].cntMax;
            tree[v].secondMax = max(tree[v * 2 + 1].secondMax, tree[v * 2].maxv);
        }
        else {
            tree[v].maxv = tree[v * 2].maxv;
            tree[v].cntMax = tree[v * 2].cntMax + tree[v * 2 + 1].cntMax;
            tree[v].secondMax = max(tree[v * 2].secondMax, tree[v * 2 + 1].secondMax);
        }

        // ------------------------MIN--------------------------
        if (tree[v * 2].minv < tree[v * 2 + 1].minv) {
            tree[v].minv = tree[v * 2].minv;
            tree[v].cntMin = tree[v * 2].cntMin;
            tree[v].secondMin = min(tree[v * 2].secondMin, tree[v * 2 + 1].minv);
        }
        else if (tree[v * 2].minv > tree[v * 2 + 1].minv) {
            tree[v].minv = tree[v * 2 + 1].minv;
            tree[v].cntMin = tree[v * 2 + 1].cntMin;
            tree[v].secondMin = min(tree[v * 2 + 1].secondMin, tree[v * 2].minv);
        }
        else {
            tree[v].minv = tree[v * 2].minv;
            tree[v].cntMin = tree[v * 2].cntMin + tree[v * 2 + 1].cntMin;
            tree[v].secondMin = min(tree[v * 2].secondMin, tree[v * 2 + 1].secondMin);
        }

        tree[v].diffGcd = std::gcd(tree[v * 2].diffGcd, tree[v * 2 + 1].diffGcd);
        int anyLeft = tree[v * 2].secondMax; // any value that's not equal to left child max and min
        int anyRight = tree[v * 2 + 1].secondMax; // any value that's not equal to right child max and min
        if (anyLeft != -INF && anyLeft != tree[v * 2].minv
            && anyRight != -INF && anyRight != tree[v * 2 + 1].minv)
        {
            tree[v].diffGcd = std::gcd(tree[v].diffGcd, anyLeft - anyRight); // connect two spanning trees
        }

        int any = -1; // any value that's not equal to current vertex max and min
        if (anyLeft != -INF && anyLeft != tree[v * 2].minv)
            any = anyLeft;
        else if (anyRight != -INF && anyRight != tree[v * 2 + 1].minv)
            any = anyRight;

        for (int val : {tree[v * 2].minv, tree[v * 2].maxv, tree[v * 2 + 1].minv, tree[v * 2 + 1].maxv}) {
            if (val != tree[v].minv && val != tree[v].maxv) {
                if (any != -1)
                    tree[v].diffGcd = std::gcd(tree[v].diffGcd, val - any);
                else
                    any = val;
            }
        }

        tree[v].add = 0;
    }

    void pushMinEq(int v, int x) {
        if (tree[v].ln == 0)
            return;
        if (tree[v].maxv <= x)
            return;
        
        if (tree[v].minv >= x) {
            pushAssignment(v, x);
            return;
        }

        tree[v].sum -= (tree[v].maxv - x) * tree[v].cntMax;
        
        if (tree[v].minv == tree[v].maxv) {
            tree[v].minv = x;
            tree[v].secondMax = -INF;
            tree[v].secondMin = INF;
        }
        else if (tree[v].secondMin == tree[v].maxv)
            tree[v].secondMin = x;

        tree[v].maxv = x;
    }

    void pushMaxEq(int v, int x) {
        if (tree[v].ln == 0)
            return;
        if (tree[v].minv >= x)
            return;

        if (tree[v].maxv <= x) {
            pushAssignment(v, x);
            return;
        }

        tree[v].sum += (x - tree[v].minv) * tree[v].cntMin;

        if (tree[v].maxv == tree[v].minv) {
            tree[v].maxv = x;
            tree[v].secondMax = -INF;
            tree[v].secondMin = INF;
        }
        else if (tree[v].secondMax == tree[v].minv)
            tree[v].secondMax = x;

        tree[v].minv = x;
    }

    void pushAdd(int v, int x) {
        if (tree[v].ln == 0)
            return;

        if (tree[v].maxv == tree[v].minv) {
            pushAssignment(v, tree[v].minv + x);
            return;
        }

        tree[v].add += x;
        tree[v].maxv += x;
        tree[v].minv += x;
        if (tree[v].secondMax != -INF)
            tree[v].secondMax += x;

        if (tree[v].secondMin != INF)
            tree[v].secondMin += x;

        tree[v].sum += x * tree[v].ln;
    }

    void pushAssignment(int v, int x) {
        if (tree[v].ln == 0)
            return;

        tree[v].maxv = x;
        tree[v].secondMax = -INF;
        tree[v].cntMax = tree[v].ln;

        tree[v].minv = x;
        tree[v].secondMin = INF;
        tree[v].cntMin = tree[v].ln;

        tree[v].assignment = x;
        tree[v].sum = x * tree[v].ln;
        tree[v].diffGcd = 0;
    }

    void pushToChildren(int v) {
        if (tree[v].ln == 0)
            return;

        if (tree[v].assignment != -INF) {
            pushAssignment(v * 2, tree[v].assignment);
            pushAssignment(v * 2 + 1, tree[v].assignment);
            tree[v].assignment = -INF;
        }
        else {
            pushAdd(v * 2, tree[v].add);
            pushAdd(v * 2 + 1, tree[v].add);
            tree[v].add = 0;

            pushMinEq(v * 2, tree[v].maxv);
            pushMinEq(v * 2 + 1, tree[v].maxv);

            pushMaxEq(v * 2, tree[v].minv);
            pushMaxEq(v * 2 + 1, tree[v].minv);
        }
    }

    void buildTree(int v, int tl, int tr) {
        tree[v].assignment = -INF;
        tree[v].add = 0;
        if (tl == tr) {
            tree[v].sum = a[tl];

            tree[v].maxv = a[tl];
            tree[v].cntMax = 1;
            tree[v].secondMax = -INF;

            tree[v].minv = a[tl];
            tree[v].cntMin = 1;
            tree[v].secondMin = INF;

            tree[v].diffGcd = 0;
            tree[v].ln = 1;
            tree[v].add = 0;
            tree[v].assignment = -INF;
            return;
        }

        int m = (tl + tr) / 2;
        buildTree(v * 2, tl, m);
        buildTree(v * 2 + 1, m + 1, tr);
        concatenation(v);
    }

    int getMinTree(int v, int l, int r, int tl, int tr) {
        if (tr < l || tl > r)
            return INF;

        if (tl >= l && tr <= r)
            return tree[v].minv;

        pushToChildren(v);

        int m = (tl + tr) / 2;
        return min(getMinTree(v * 2, l, r, tl, m), 
            getMinTree(v * 2 + 1, l, r, m + 1, tr));
    }

    int getMaxTree(int v, int l, int r, int tl, int tr) {
        if (tr < l || tl > r)
            return -INF;

        if (tl >= l && tr <= r)
            return tree[v].maxv;

        pushToChildren(v);

        int m = (tl + tr) / 2;
        return max(getMaxTree(v * 2, l, r, tl, m), 
            getMaxTree(v * 2 + 1, l, r, m + 1, tr));
    }

    int getSumTree(int v, int l, int r, int tl, int tr) {
        if (tr < l || tl > r)
            return 0;

        if (tl >= l && tr <= r)
            return tree[v].sum;

        pushToChildren(v);

        int m = (tl + tr) / 2;
        return getSumTree(v * 2, l, r, tl, m) +
            getSumTree(v * 2 + 1, l, r, m + 1, tr);
    }

    int getGcdTree(int v, int l, int r, int tl, int tr) {
        if (tr < l || tl > r)
            return 0;

        if (tl >= l && tr <= r) {
            int ans = tree[v].diffGcd;
            if (tree[v].secondMax != -INF)
                ans = std::gcd(ans, tree[v].secondMax - tree[v].maxv);

            if (tree[v].secondMin != INF)
                ans = std::gcd(ans, tree[v].secondMin - tree[v].minv);

            ans = std::gcd(ans, tree[v].maxv);
            return ans;
        }

        pushToChildren(v);

        int m = (tl + tr) / 2;
        return std::gcd(getGcdTree(v * 2, l, r, tl, m), 
            getGcdTree(v * 2 + 1, l, r, m + 1, tr));
    }

    void updateMinTree(int v, int x, int l, int r, int tl, int tr) {
        if (tl > r || tr < l || tree[v].maxv <= x)
            return;

        if (tl >= l && tr <= r && tree[v].secondMax < x) {
            pushMinEq(v, x);
            return;
        }

        pushToChildren(v);

        int m = (tl + tr) / 2;
        updateMinTree(v * 2, x, l, r, tl, m);
        updateMinTree(v * 2 + 1, x, l, r, m + 1, tr);
        concatenation(v);
    }

    void updateMaxTree(int v, int x, int l, int r, int tl, int tr) {
        if (tl > r || tr < l || tree[v].minv >= x)
            return;

        if (tl >= l && tr <= r && tree[v].secondMin > x) {
            pushMaxEq(v, x);
            return;
        }

        pushToChildren(v);

        int m = (tl + tr) / 2;
        updateMaxTree(v * 2, x, l, r, tl, m);
        updateMaxTree(v * 2 + 1, x, l, r, m + 1, tr);
        concatenation(v);
    }

    void updateAddTree(int v, int x, int l, int r, int tl, int tr) {
        if (tr < l || tl > r)
            return;

        if (tl >= l && tr <= r) {
            pushAdd(v, x);
            return;
        }

        pushToChildren(v);

        int m = (tl + tr) / 2;
        updateAddTree(v * 2, x, l, r, tl, m);
        updateAddTree(v * 2 + 1, x, l, r, m + 1, tr);
        concatenation(v);
    }

    void updateAssignmentTree(int v, int x, int l, int r, int tl, int tr) {
        if (tr < l || tl > r)
            return;

        if (tl >= l && tr <= r) {
            pushAssignment(v, x);
            return;
        }

        pushToChildren(v);
        int m = (tl + tr) / 2;
        updateAssignmentTree(v * 2, x, l, r, tl, m);
        updateAssignmentTree(v * 2 + 1, x, l, r, m + 1, tr);
        concatenation(v);
    }

    void updateModTree(int v, int x, int l, int r, int tl, int tr) {
        if (tr < l || tl > r || tree[v].maxv < x)
            return;

        if (tl == tr) {
            pushToChildren(v);
            tree[v].maxv %= x;
            tree[v].minv %= x;
            tree[v].sum %= x;
            return;
        }

        pushToChildren(v);
        int m = (tl + tr) / 2;
        updateModTree(v * 2, x, l, r, tl, m);
        updateModTree(v * 2 + 1, x, l, r, m + 1, tr);
        concatenation(v);
    }
};
```