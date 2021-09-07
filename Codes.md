# 代码集 （暂存，最终将删除）

## 1. LeetCode Cpp

```c++
#include <bits/stdc++.h>
#define IN_LC 1

template <typename Functor>
class YCombinator
{
public:
    template <typename Function>
    YCombinator(Function &&func) : func(static_cast<Function &&>(func)) {}

    template <typename... Args>
    decltype(auto) operator()(Args &&...args) const
    {
        return func(*this, (Args &&) args...);
    }

private:
    Functor func;
};

#if __cplusplus >= 201703
template <typename Fn>
YCombinator(Fn) -> YCombinator<Fn>;
#elif __cplusplus >= 201402
template <typename Functor>
decltype(auto) MakeYCombinator(Functor func)
{
    return YCombinator<Functor>(func);
}
#else
#define Lambda(NAME, CAP, PROTO, BODY)                     \
    [&]() {                                                \
        std::function<auto PROTO>(NAME) = CAP PROTO{BODY}; \
        return (NAME);                                     \
    }()
#endif

inline namespace My
{
    // 0. Tools Macro
    inline namespace
    {
#define TMP_T template <typename T>
#define TMP_TV(T) template <T v>
#define TMP_T2 template <typename T1, typename T2>
#define TMP_TV2(T) template <T v1, T v2>
#define TUPLE(...) std::make_tuple(__VA_ARGS__)

#define CHEATING_HEAD std::ios::sync_with_stdio(false)
#define ALL(...) begin(__VA_ARGS__), end(__VA_ARGS__)
#define ISDIGIT(c) (c) >= '0' && (c) <= '9'

#define DEBUG(...) std::cout << "[ " << #__VA_ARGS__ << " ] = " << (__VA_ARGS__) << std::endl;
#define DEBUGARRAY(arr)                                                               \
    do                                                                                \
    {                                                                                 \
        i32 len = arr.size();                                                         \
        for (i32 i = 0; i < len; ++i)                                                 \
        {                                                                             \
            std::cout << "[ " << #arr << "(" << i << ") ] = " << arr[i] << std::endl; \
        }                                                                             \
    } while (0)

    }

    // 1. RustLike Redefine
    inline namespace
    {
        using i8 = int8_t;
        using i16 = int16_t;
        using i32 = int32_t;
        using i64 = int64_t;
        using u8 = uint8_t;
        using u16 = uint16_t;
        using u32 = uint32_t;
        using u64 = uint64_t;
        using f32 = float_t;
        using f64 = double_t;
        using str = std::string;
        TMP_T using Vec = std::vector<T>;
        TMP_T using Mat = std::vector<std::vector<T>>;
        TMP_T using VecVec = std::vector<std::vector<T>>;
        TMP_T using Heap = std::priority_queue<T>;
    }

    // 2. Tools Template
    inline namespace
    {
        TMP_TV2(i32)
        constexpr auto Range = []()
        {
            auto ret = std::array<i32, v2 - v1>();
            for (i32 i = v1; i < v2; ++i)
                ret[i - v1] = i;
            return ret;
        }();
        TMP_TV(i32)
        constexpr auto RangeTo = []()
        {
            auto ret = std::array<i32, v>();
            for (i32 i = 0; i < v; ++i)
                ret[i] = i;
            return ret;
        }();
    }

    // 3. Functions
    namespace LinearAlgebra
    {
        TMP_T constexpr decltype(auto) NewMat(int n, int m) { return Mat<T>(n, Vec<T>(m)); }
        TMP_T constexpr decltype(auto) MatSize(const Mat<T> &mat) { return TUPLE(mat.size(), mat[0].size()); }
    }
    namespace GraphAlgo
    {
        decltype(auto) EdgesToAdjList(const VecVec<i32> &edges, const i32 n)
        {
            auto graph = VecVec<i32>(n, Vec<i32>());
            for (auto &&e : edges)
            {
                auto st = e[0], ed = e[1];
                graph[st].push_back(ed);
                graph[ed].push_back(st);
            }
            return graph;
        }
    }
    namespace WeightedGraphAlgo
    {
        TMP_T struct Vertex
        {
            i32 idx;
            T weight;
            Vertex(i32 idx, T w) : idx(idx), weight(w) {}
        };
        TMP_T decltype(auto) EdgesToAdjList(const VecVec<i32> &edges, const Vec<T> &weights)
        {
            auto n = weights.size();
            auto graph = VecVec<Vertex<T>>(n, Vec<Vertex<T>>());
            for (auto &&e : edges)
            {
                auto st = e[0], ed = e[1];
                graph[st].push_back(Vertex(ed, weights[ed]));
                graph[ed].push_back(Vertex(st, weights[st]));
            }
            return graph;
        }
    }
    namespace String
    {
        /**
         * @brief 统计目标串中有多少模式串
         */
        const i32 KMP(const str &target, const str &pattern)
        {
            i32 ans = 0;
            i32 it = 0, ip = 0; // idx target, idx pattern
            i32 lt = target.size(), lp = pattern.size();

            // 预处理 next 数组
            i32 i = 0, j = -1;
            Vec<i32> nxt(lp + 1, -1);
            while (i < lp)
            {
                while (j != -1 && pattern[i] != pattern[j])
                    j = nxt[j];
                if (pattern[++i] == pattern[++j])
                    nxt[i] = nxt[j];
                else
                    nxt[i] = j;
            }

            while (it < lt)
            {
                while (ip != -1 && pattern[ip] != target[it])
                    ip = nxt[ip];
                ++ip, ++it;
                if (ip >= lp)
                    ++ans, ip = nxt[ip];
            }
            return ans;
        }

        /**
         * @brief 马拉车算法，求最长回文字串
         */
        const str &Manacher(const str &s)
        {
            str ma = "^#";
            for (auto c : s)
            {
                ma += c;
                ma += '#';
            }
            ma += '$';

            i32 l = ma.size();
            Vec<i32> mp(l, 0);
            i32 mx = 0, idx = 0, maxpos = 0;
            for (auto i : RangeTo<l - 1>)
            {
                mp[i] = (mx > i) ? std::min(mp[(idx << 1) - i], mx - i) : 1;
                while (i - mp[i] >= 0 && i + mp[i] < (i32)(ma.size()) && ma[i + mp[i]] == ma[i - mp[i]])
                    ++mp[i] if (mp[i] > mp[maxpos])
                        maxpos = i;
                if (i + mp[i] > mx)
                    mx = i + mp[i], idx = i;
            }
            str ret;
            for (auto i : Range<maxpos - mp[maxpos] + 1, maxpos + mp[maxpos]>)
                if (ISDIGIT(ma[i]))
                    ret += ma[i];
            return ret;
        }
    }
}

#if IN_LC == 0
inline namespace LeetCodeDataStructure
{
    struct TreeNode
    {
        int val;
        TreeNode *left;
        TreeNode *right;
        TreeNode() : val(0), left(nullptr), right(nullptr) {}
        TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
        TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
    };
}
#endif
```



