>[!info]
>Скользящая таблица на произведение по модулю. Эта структура поддерживает запросы на отрезке за O(1) но только не изменения на отрезке. Отрезок [l; r] ($0 <= l <= r < n$)

```c++
template<typename AccumType>
struct DisjointSparseTable {
    vector<vector<AccumType>> sparseTable;

    template<typename InputType>
    DisjointSparseTable(const vector<InputType>& arr) {
        int n = arr.size();
        int height = highestBit(n - 1) + 1;
        int extendedSize = (1 << height);

        sparseTable.assign(height, vector<AccumType>(extendedSize, 1));

        for (int lvl = 0; lvl < height; lvl++) {
            int curLen = (1 << lvl);
            for (int center = curLen; center < extendedSize; center += 2 * curLen) {

                sparseTable[lvl][center] = (center < n ? AccumType(arr[center]) : 0ll) % MOD;
                for (int i = center + 1; i < center + curLen; i++) {
                    sparseTable[lvl][i] = sparseTable[lvl][i - 1] * (i < n ? AccumType(arr[i]) : 0ll) % MOD;
                }

                sparseTable[lvl][center - 1] = (center - 1 < n ? AccumType(arr[center - 1]) : 0) % MOD;
                for (int i = center - 2; i >= center - curLen; i--) {
                    sparseTable[lvl][i] = (i < n ? AccumType(arr[i]) : 0) * sparseTable[lvl][i + 1] % MOD;
                }
            }
        }
    }

    int highestBit(unsigned int number) {
        if (number == 0) return 0;
#ifdef _MSC_VER
        unsigned long index;
        _BitScanReverse(&index, number);
        return static_cast<int>(index);
#else
        return 31 - __builtin_clz(number); // тут смотреть на тип данных
#endif
    }

    AccumType query(int left, int right) { // [left, right]
        if (left == right)
            return sparseTable[0][left];

        int lvl = highestBit(left ^ right);
        return sparseTable[lvl][left] * sparseTable[lvl][right] % MOD;
    }
};


// вот так создаётся DisjointSparseTable<int> maxCntSparce(a);
```