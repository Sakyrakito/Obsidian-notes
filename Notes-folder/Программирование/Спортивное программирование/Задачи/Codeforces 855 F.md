# Дата - 2025-10-23

[Ссылка на задачу](https://codeforces.com/problemset/problem/855/F?locale=ru&mobile=false)

Tags: #спортивное_программирование #корневая_декомпозиция #структуры_данных

---
## О чём задача:

Даны q запросов. Запросы могут быть двух типов:
1. l, r, k - для всех чисел от l до r сделать a\[i] = min(a\[i], k), если a\[i] уже инициализирована, иначе сделать a\[i] = k. Если k меньше 0, то сделать то же самое, но a\[i] = max(a\[i], k).
2. l, r - найти сумму всех чисел от l до r. Если числа нет (не инициализировано), то они считается нулём. Сумма считается как число снизу + число сверху. Если нет хотя бы 1 число, то сумма в этой координате равна 0.

---
## Решение задачи:  

Применим [[корневая декомпозиция (sqrt-декомпозиция)|корневую декомпозицию]]. Для каждого блока будем хранить:
1. Массив ans, где храниться ответ
2. 2 значения upper и lower для, которые хранят минимальный y > 0, который покрывает весь блок и минимальный |y| для y < 0, который тоже покрывает весь блок соответственно.
3. Словари low, up, где для каждого y храним сколько раз он встречается (только для минимальных значений).
4. Словари vnotup и vnotdown, они как low и up, но хранят значения для тех координат, которые не имеют соседа с другой стороны
5. Значение notboth, хранит количество x-координат, которые не инициализированы ни снизу, ни сверху.
6. Массивы minup и mindown, для значений от 1 до 10^5. Они хранят значения для тех отрезков, которые полностью не покрывают блок.
   
Обновление неполного блока (на примере добавления y > 0, для y < 0 будет симметрично):
1. Если все значения minup, mindown, lower, upper равны INF, то мы сюда впервые добавляем отрезок. Понижаем notboth блока на 1 и обновляем vnotup.
2. Иначе если minup и upper равные INF, значит, что x снизу существует, но не сверху. Обновляем vnotup, up, low, ans.
3. Иначе если mindown и lower = INF, значит что нет x при y < 0. Просто обновляем vnotdown. 
4. Если если обе линии существуют, обновляем ans и up

Обновление целого блока (тоже для y > 0):
1. Удаляем все элементы из vnotup
2. Обновляем up, low, ans
3. Для notboth ставим = 0
4. Обновляем vnotdown - удаляем все элементы, которые больше y и ставим их = y. 
5. В конце обновляем up и ans
6. Так же обновляем upper

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
#include<unordered_set>
#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 1e5, INF = 1e17, MAXA = 1e6 + 3, MOD = 1e9 + 7;
const long double PI = acos(-1);

/*
    Класс реализует sqrt-декомпозицию для задачи, где необходимо обрабатывать
    отрезки по оси X и учитывать линии (или "змейки") с разными координатами y.
    Идея — разбить ось X на блоки и хранить агрегированные данные по каждому блоку.
*/
class SqrtDec {
public:
    SqrtDec(int n) {
        // Инициализация: размер массива, размер блока
        N = n;
        gsize = sqrtl(n / 2);

        // Вектора для хранения состояния каждого блока
        ans.resize(1000);                // ответ для блока
        upper.assign(1000, INF);         // минимальное y > 0, покрывающее весь блок
        lower.assign(1000, INF);         // минимальное |y| для y < 0, покрывающее блок
        low.resize(1000);                // map для y < 0
        up.resize(1000);                 // map для y > 0
        vnotup.resize(1000);             // map для "дыр" сверху (где нет линии y > 0)
        vnotdown.resize(1000);           // map для "дыр" снизу (где нет линии y < 0)
        notBoth.resize(1000);            // количество координат без линии ни сверху, ни снизу
        minup.assign(n + 1, INF);        // min y>0 для каждой координаты
        mindown.assign(n + 1, INF);      // min |y| для y<0 для каждой координаты

        // Изначально все x-координаты "не покрыты" (нет линий)
        for (int i = 1; i <= n; i++) {
            int j = i / gsize;
            notBoth[j]++;
        }
    }

    /*
        Локальный запрос для блока: считает вклад координат [l, r]
        Использует minup/mindown и границы блока upper/lower.
    */
    int query(int i, int l, int r) {
        int sm = 0;
        for (int j = l; j <= r; j++) {
            // Если по обе стороны (y>0 и y<0) есть линии — добавляем их вклад
            if (min(upper[i], minup[j]) < INF && min(lower[i], mindown[j]) < INF)
                sm += min(upper[i], minup[j]) + min(lower[i], mindown[j]);
        }
        return sm;
    }

    // Возврат полного ответа для блока (используется при полном покрытии)
    int querufull(int i) {
        return ans[i];
    }

    /*
        Основная функция обновления: добавляет новую линию на отрезке [l, r].
        val > 0 — линия сверху (y>0)
        val < 0 — линия снизу (y<0)
        Делится на три части: левый неполный блок, полный блоки, правый неполный блок.
    */
    void updateSum(int l, int r, int val) {
        int i = l / gsize;
        int j = r / gsize;

        if (val > 0) { // линия выше оси
            if (i == j)
                updateUp(i, l, r, val);
            else {
                updateUp(i, l, gend(i), val);
                i++;
                while (i < j) {
                    updatefullup(i, val); // обновление для целого блока
                    i++;
                }
                updateUp(j, gbegin(j), r, val);
            }
        }
        else { // линия ниже оси
            if (i == j)
                updateDown(i, l, r, -val);
            else {
                updateDown(i, l, gend(i), -val);
                i++;
                while (i < j) {
                    updatefulldown(i, -val);
                    i++;
                }
                updateDown(j, gbegin(j), r, -val);
            }
        }
    }

    /*
        Запрос суммы на отрезке [l, r].
        Для частично покрытых блоков — вызов query(),
        для полностью покрытых — querufull().
    */
    int getSum(int l, int r) {
        int i = l / gsize;
        int j = r / gsize;
        if (i == j)
            return query(i, l, r);
        else {
            int sm = query(i, l, gend(i));
            i++;
            while (i < j) {
                sm += querufull(i);
                i++;
            }
            sm += query(j, gbegin(j), r);

            return sm;
        }
    }

private:
    int gsize = sqrt(1e5 / 2); // размер блока
    int N;

    // Основные структуры данных для каждого блока
    vector<map<int, int>> low, up;       // хранят распределение высот (y<0 и y>0)
    vector<map<int, int>> vnotup, vnotdown; // хранят координаты без линий сверху/снизу
    vector<int> notBoth, minup, mindown, upper, lower, ans;

    // Возвращает начальный и конечный индекс блока
    int gbegin(int g) { 
        return g * gsize;
    }

    int gend(int g) { 
        return min(gbegin(g) + gsize - 1, N - 1);
    }

    /*
        Обновление блока частично: добавление линии сверху (y > 0).
        Проходим по каждой координате и корректируем minup, ans, карты up/low и т.д.
    */
    void updateUp(int i, int l, int r, int val) {
        if (val >= upper[i])
            return;

        for (int j = l; j <= r; j++) {
            if (minup[j] <= val)
                continue;

            // Если в точке ещё не было линий вообще
            if (minup[j] == INF && mindown[j] == INF && upper[i] == INF && lower[i] == INF) {
                notBoth[i]--;
                vnotdown[i][val]++;
            }
            // Есть только линия снизу
            else if (minup[j] == INF && upper[i] == INF) {
                int temp = min(lower[i], mindown[j]);
                vnotup[i][temp]--;
                if (vnotup[i][temp] == 0)
                    vnotup[i].erase(temp);

                up[i][val]++;
                low[i][temp]++;
                ans[i] += val;
                ans[i] += min(lower[i], mindown[j]);
            }
            // Есть только линия сверху
            else if (mindown[j] == INF && lower[i] == INF) {
                int temp = min(upper[i], minup[j]);
                vnotdown[i][temp]--;
                if (vnotdown[i][temp] == 0)
                    vnotdown[i].erase(temp);
                vnotdown[i][val]++;
            }
            // Есть линии с обеих сторон — обновляем ans и карты
            else {
                ans[i] += val - min(upper[i], minup[j]);
                up[i][min(minup[j], upper[i])]--;
                up[i][val]++;
            }
            minup[j] = val;
        }
    }

    /*
        Обновление целого блока для линий сверху (y > 0).
        Все точки блока полностью перекрыты новой линией.
    */
    void updatefullup(int i, int val) {
        if (val >= upper[i])
            return;

        int sm = 0;
        int ln2 = 0;

        // Обработка точек без верхней линии
        for (auto it = vnotup[i].begin(); it != vnotup[i].end(); it++) {
            ans[i] += (it->first) * (it->second) + (it->second) * val;
            ln2 += it->second;
            low[i][it->first] += it->second;
        }

        vnotup[i].clear();

        // Удаляем точки, где нижняя линия выше текущего y
        int temp = 0;
        auto it = vnotdown[i].upper_bound(val);
        for (; it != vnotdown[i].end(); it++)
            temp += it->second;

        it = vnotdown[i].upper_bound(val);
        vnotdown[i].erase(it, vnotdown[i].end());
        vnotdown[i][val] += temp + notBoth[i];
        notBoth[i] = 0;

        // Пересчитываем up-множество
        int ln = 0;
        it = up[i].upper_bound(val);
        for (; it != up[i].end(); it++) {
            sm += it->first * it->second;
            ln += it->second;
        }

        it = up[i].upper_bound(val);
        up[i].erase(it, up[i].end());
        ans[i] += ln * val - sm;
        up[i][val] += ln + ln2;
        upper[i] = val;
    }

    /*
        Аналог updateUp, но для линий снизу (y < 0).
    */
    void updateDown(int i, int l, int r, int val) {
        if (val >= lower[i])
            return;

        for (int j = l; j <= r; j++) {
            if (mindown[j] <= val)
                continue;

            if (minup[j] == INF && mindown[j] == INF && upper[i] == INF && lower[i] == INF) {
                notBoth[i]--;
                vnotup[i][val]++;
            }
            else if (mindown[j] == INF && lower[i] == INF) {
                int temp = min(upper[i], minup[j]);
                vnotdown[i][temp]--;
                if (vnotdown[i][temp] == 0)
                    vnotdown[i].erase(temp);

                low[i][val]++;
                up[i][temp]++;
                ans[i] += val;
                ans[i] += min(upper[i], minup[j]);
            }
            else if (minup[j] == INF && upper[i] == INF) {
                int temp = min(lower[i], mindown[j]);
                vnotup[i][temp]--;
                if (vnotup[i][temp] == 0)
                    vnotup[i].erase(temp);
                vnotup[i][val]++;
            }
            else {
                ans[i] += val - min(lower[i], mindown[j]);
                low[i][min(mindown[j], lower[i])]--;
                low[i][val]++;
            }
            mindown[j] = val;
        }
    }

    /*
        Полное обновление блока при добавлении линии снизу (y < 0).
        Симметрично функции updatefullup().
    */
    void updatefulldown(int i, int val) {
        if (val >= lower[i])
            return;

        int sm = 0;
        int ln2 = 0;

        // Обрабатываем точки без нижней линии
        for (auto it = vnotdown[i].begin(); it != vnotdown[i].end(); it++) {
            ans[i] += (it->first) * (it->second) + (it->second) * val;
            ln2 += it->second;
            up[i][it->first] += it->second;
        }

        vnotdown[i].clear();

        // Переносим точки, где верхняя линия выше текущей
        int temp = 0;
        auto it = vnotup[i].upper_bound(val);
        for (; it != vnotup[i].end(); it++)
            temp += it->second;

        it = vnotup[i].upper_bound(val);
        vnotup[i].erase(it, vnotup[i].end());
        vnotup[i][val] += temp + notBoth[i];
        notBoth[i] = 0;

        // Пересчёт low-множества
        int ln = 0;
        it = low[i].upper_bound(val);
        for (; it != low[i].end(); it++) {
            sm += it->first * it->second;
            ln += it->second;
        }

        it = low[i].upper_bound(val);
        low[i].erase(it, low[i].end());
        ans[i] += ln * val - sm;
        low[i][val] += ln + ln2;
        lower[i] = val;
    }
};

/*
    Основная функция:
    Обрабатывает q запросов двух типов:
    1 t=1 l r val — обновление (добавление линии)
    2 t=2 l r     — запрос суммы
*/
void solve() {
    SqrtDec sq(1e5 + 5);
    int q; cin >> q;
    while (q--) {
        int t, l, r;
        cin >> t >> l >> r;
        r--;
        if (t == 1) {
            int val; cin >> val;
            sq.updateSum(l, r, val);
        }
        else {
            cout << sq.getSum(l, r) << '\n';
        }
    }
}

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0); cout.tie(0);
    int t;
    t = 1;
    while (t--)
        solve();

    return 0;
}
```
---