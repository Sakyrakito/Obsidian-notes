>[!info] 
>корневая декомпозиция на сумму

C++:
```c++
class SqrtDec {
public:
	SqrtDec(const vector<int>& a) {
		gsize = sqrtl((int)a.size() / 2);
		arr = a;
		sm.assign((a.size() + gsize - 1) / gsize, INF);
		for (int g = 0; g < (int)sm.size(); g++)
			calcGroup(g);
	}

	void update(int i, int v) {
		int g = i / gsize;
		sm[g] -= arr[i];
		arr[i] = v;
		sm[g] += arr[i];
	}

	int get_sum(int l, int r) {
		int gl = l / gsize, gr = r / gsize;
		int lsm = 0;
		if (gl == gr) {
			for (int i = l; i <= r; i++)
				lsm += arr[i];
		}
		else {
			for (int i = l; i < gend(gl); i++)
				lsm += arr[i];
			for (int i = gl + 1; i < gr; i++)
				lsm += sm[i];
			for (int i = gbegin(gr); i <= r; i++)
				lsm += arr[i];
		}
		return lsm;
	}

private:
	int gsize = sqrt(1e5 / 2);
	vector<int> arr, sm;

	int gbegin(int g) {
		return g * gsize;
	}

	int gend(int g) {
		return min(gbegin(g) + gsize, (int)arr.size());
	}

	void calcGroup(int g) {
		sm[g] = 0;
		for (int i = gbegin(g); i < gend(g); i++)
			sm[g] += arr[i];
	}
};
```