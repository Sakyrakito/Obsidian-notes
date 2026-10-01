>[!info]
>код для удобного дебага. Этот код полностью поддерживает `std::set`, `std::map`, `std::unordered_set`, `std::unordered_map`, а также `std::vector`, `std::list`, `std::deque` и `std::array`. Просто передать в функцию `watch()`

```C++
// ---------------- DEBUG ------------------
// 1. Предварительные объявления (чтобы вложенные структуры находили друг друга)
template <typename T1, typename T2>
std::ostream &operator<<(std::ostream &os, const std::pair<T1, T2> &p);

template <typename... Args>
std::ostream &operator<<(std::ostream &os, const std::tuple<Args...> &t);

template <typename T>
auto operator<<(std::ostream &os, const T &c)
    -> typename std::enable_if<!std::is_same<T, std::string>::value, decltype(c.begin(), c.end(), os)>::type;

// 2. Печать std::pair
template <typename T1, typename T2>
std::ostream &operator<<(std::ostream &os, const std::pair<T1, T2> &p) {
    return os << "(" << p.first << ", " << p.second << ")";
}

// 3. Печать std::tuple
template <typename Tuple, std::size_t... Is>
void print_tuple(std::ostream &os, const Tuple &t, std::index_sequence<Is...>) {
    ((os << (Is == 0 ? "" : ", ") << std::get<Is>(t)), ...);
}

template <typename... Args>
std::ostream &operator<<(std::ostream &os, const std::tuple<Args...> &t) {
    os << "(";
    print_tuple(os, t, std::index_sequence_for<Args...>{});
    return os << ")";
}

// 4. Печать контейнеров (кроме std::string)
template <typename T>
auto operator<<(std::ostream &os, const T &c)
    -> typename std::enable_if<!std::is_same<T, std::string>::value, decltype(c.begin(), c.end(), os)>::type
{
    os << "{";
    bool first = true;
    for (const auto &x : c) {
        os << (first ? "" : ", ") << x;
        first = false;
    }
    return os << "}";
}

#define watch(...) debug && std::cerr << "{" << #__VA_ARGS__ << "} = " \
    << std::make_tuple(__VA_ARGS__) << std::endl

const bool debug = true;
// -----------------------------------------
```