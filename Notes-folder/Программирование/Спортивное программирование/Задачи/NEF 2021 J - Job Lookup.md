# Дата - 2026-07-14

[Ссылка на задачу](https://codeforces.com/contest/1666/problem/J)

Tags: #спортивное_программирование #деревья #дп

---
## О чём задача:

Есть матрица $C$ размера $n*n$ симметричная относительно главной диагонали. Элементы на главной диагонали равны нулю.

Мы должны построить бинарное дерево поиска из $n$ элементов (для любой вершины $v$ все вершины слева имеют значение строго меньше $v$, а справа строго больше).

Определим расстояние между вершинами $i$ и $j$ как количество рёбер, лежащих на кратчайшем пути из $i$ в $j$. Обозначим его как $d_{ij}$

Нам нужно построить такое дерево, чтобы минимизировать:

$$
\LARGE
\sum_{1 \leq i < j \leq n}{C_{ij} * d_{ij}}
$$

В конце нужно вывести структуру дерева: для каждой вершины от 1 до $n$ вывести его предка. Для корня дерева предок равен 0

$$
1 \leq n \leq 200
$$

$$
0 \leq C_{ij} \leq 10^9
$$

---
## Решение задачи:  

Давайте сначала переосмыслим формулу $\sum C_{ij} * d_{ij}$. 
По сути $d_{ij}$ - это количество рёбер от $i$ до $j$. Это значит, что каждое ребро на пути увеличило свою стоимость на $C_{ij}$. И если мы сложим стоимость каждого ребра, то получим именно $C_{ij} * d_{ij}$. 

Тогда давайте для каждой пары $i$ и $j$ посмотрим через какие рёбра пройдёт их путь и к стоимости каждого ребра на пути добавим $C_{ij}$ и в конце просто сложим стоимости всех рёбер.

Воспользуемся свойством дерева поиска: для любого поддерева с корнем $v$ множество всех вершин этого поддерева образуют непрерывный отрезок \[l, r]. То есть в этом поддереве будут все вершины начиная с l до r и никаких других.

Зная это, мы можем написать дп по всем отрезкам \[l, r]. Будем рассматривать все отрезки в порядке возрастания их длин. Пусть сейчас мы рассматриваем \[l, r]. Нам нужно узнать стоимость ребра, которое соединяет корень этого поддерева со всем остальным деревом. Для этого нам нужно посчитать сумму:

$$
\large
W(l, r) = \sum_{i \in [l, r]}{\sum_{j \notin[l, r]}{C_{ij}}}
$$

То есть все вершины внутри поддерева (это $i$) и все вершины "снаружи" поддерева (это $j$).

Так же нам нужно перебрать корень этого поддерева. Пусть это будет $k$. Для данного корня есть левое и правое поддерево. Все вершины в левом поддереве принадлежат отрезку $[l, k - 1]$, а все вершины в правом принадлежат отрезку $[k + 1, r]$. Тогда ответ для данного отрезка будет:

$$
\large
dp[l][r] = W(l, r) + \underset{k \in [l, r]}{min}(dp[l][k - 1] + dp[k + 1][r])
$$

Теперь проблема в том, что нам нужно быстро посчитать $W(l, r)$. Если делать это жадно, то итоговая асимптотика будет $O(n^4)$ что нам не подходит. Но эту сумму можно узнавать за $O(1)$ с помощью префикс сумм. 
По сути её можно раскрыть так:

$$
\large
W(l, r) = (все\ возможные\ суммы\ с\ [l, r]) - (все\ возможные\ суммы\ между\  [l, r])
$$

Чтобы посчитать все возможные суммы с $[l, r]$ нам достаточно знать суммы в каждой строке матрицы с $l$ до $r$ и просто сложить их все. Это делается обычной префикс суммой.

Чтобы посчитать все возможные суммы между $[l, r]$ нам придётся воспользоваться двумерной префикс суммой. По сути нам нам нужно взять сумму на прямоугольнике $[l, r][l, r]$. Для этого просто построить двумерную префикс сумму на матрице.

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
//#pragma GCC target("avx2")
#define int long long

using namespace std;

mt19937 rnd(time(NULL));

const int MAXN = 5e5 + 3;
const int INF = 1e18;
const int MAXA = 5e6 + 3;
const int MOD = 1e9 + 7;

void solve() {
	int n; cin >> n;
	vector<vector<int>> vc(n + 1, vector<int>(n + 1));
	vector<vector<int>> pref1(n + 1, vector<int>(n + 1));

	for (int i = 1; i <= n; i++){
		for (int j = 1; j <= n; j++){
			cin >> vc[i][j];
			pref1[i][j] = pref1[i][j - 1] + vc[i][j];
		}

		pref1[i][n] += pref1[i - 1][n];
	}

	vector<vector<int>> pref2(n + 1, vector<int>(n + 1)); 

	for (int i = 1; i <= n; i++){
		for (int j = 1; j <= n; j++){
			pref2[i][j] = pref2[i - 1][j] + pref2[i][j - 1] - pref2[i - 1][j - 1] + vc[i][j];
		}
	}

	vector<vector<int>> dp(n + 2, vector<int>(n + 2, INF));
	vector<vector<int>> pred(n + 1, vector<int>(n + 1));

	for (int i = 1; i <= n; i++){
		dp[i][i] = pref1[i][n] - pref1[i - 1][n];
		pred[i][i] = i;
	}

	for (int ln = 2; ln <= n; ln++){
		for (int l = 1; l + ln - 1 <= n; l++){
			int r = l + ln - 1;

			int sm1 = pref1[r][n] - pref1[l - 1][n];
			int sm2 = pref2[r][r] - pref2[r][l - 1] - pref2[l - 1][r] + pref2[l - 1][l - 1];
			int sm = sm1 - sm2;

			int best = -1;
			for (int k = l; k <= r; k++){
				int left = (dp[l][k - 1] == INF ? 0ll : dp[l][k - 1]);
				int right = (dp[k + 1][r] == INF ? 0ll : dp[k + 1][r]);

				if (sm + left + right < dp[l][r]){
					dp[l][r] = sm + left + right;
					best = k;
				}
			}

			pred[l][r] = best;
		}
	}

	vector<int> ans(n + 1);
	
	function<void(int, int, int)> dfs = [&](int l, int r, int p){
		if (l > r){
			return;
		}

		int k = pred[l][r];
		ans[k] = p;

		if (l >= r){
			return;
		}

		dfs(l, k - 1, k);
		dfs(k + 1, r, k);
	};

	dfs(1, pred[1][n] - 1, pred[1][n]);
	dfs(pred[1][n] + 1, n, pred[1][n]);

	for (int i = 1; i <= n; i++){
		cout << ans[i] << ' ';
	}
	cout << '\n';
}

signed main() {
	ios_base::sync_with_stdio(false);
	cin.tie(0); cout.tie(0);
	//freopen("basis.in", "r", stdin);
	//freopen("basis.out", "w", stdout);
	int t = 1;
	// cin >> t;
	while (t--) {
		solve();
	}
	return 0;
}
```
---