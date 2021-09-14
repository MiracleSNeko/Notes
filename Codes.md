# 代码集 （暂存，最终将删除）

## 1. LeetCode Cpp

```c++
#include <bits/stdc++.h>
#define IN_LC 1
#define USE_YCOMBINATOR 2017

#if USE_YCOMBINATOR >= 2014
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
#endif

#if USE_YCOMBINATOR == 2017
template <typename Fn>
YCombinator(Fn) -> YCombinator<Fn>;
#elif USE_YCOMBINATOR == 2014
template <typename Functor>
decltype(auto) MakeYCombinator(Functor func)
{
    return YCombinator<Functor>(func);
}
#elif USE_YCOMBINATOR == 2011
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
#define PAIR(...) std::make_pair(__VA_ARGS__)

#define FORINC(i, st, ed) for (auto i = st; i < ed; ++i)
#define FORINC_STP(i, st, ed, stp) for (auto i = st; i < ed; i += stp)
#define FORDEC(i, st, ed) for (auto i = st; i > ed; --i)
#define FORDEC_STP(i, st, ed, stp) for (auto i = st; i > ed; i -= stp)

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
        TMP_TV(i64)
        using BitSet = std::bitset<v>;
        TMP_T using Vec = std::vector<T>;
        TMP_T using Mat = std::vector<std::vector<T>>;
        TMP_T using VecVec = std::vector<std::vector<T>>;
        TMP_T using Heap = std::priority_queue<T>;
    }

    // 2. Tools Template and Constant
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

        constexpr i32 Direction[4][2] = {{1, 0}, {0, -1}, {0, 1}, {-1, 0}};
        constexpr i32 Mod = 1e9 + 7;
        constexpr i32 Inf = 0x3f3f3f3f;
    }

    // 3. Functions
    inline namespace // Generic
    {
        TMP_T2 const Vec<T2> PreSum(const Vec<T1> &vec)
        {
            T2 tmp = 0;
            Vec<T2> ret({tmp});
            for (auto &i : vec)
            {
                tmp += i;
                ret.emplace_back(tmp);
            }
            return ret;
        }

        TMP_T const Vec<T> PreSum(const Vec<T> &vec)
        {
            return PreSum<T, T>(vec);
        }

        TMP_T2 const T2 Acc(const Vec<T1> &vec, T2 st = 0)
        {
            return std::accumulate(ALL(vec), st);
        }

        TMP_T const T Acc(const Vec<T> &vec, T st = 0)
        {
            return std::accumulate(ALL(vec), st);
        }

        // Wrapper of std::lower_bound for Vec<T> with default Cmp
        TMP_T const size_t LowerBS(const Vec<T> &vec, const T &target)
        {
            return std::lower_bound(ALL(vec), target) - vec.begin();
        }

        // Wrapper of std::upper_bound for Vec<T> with default Cmp
        TMP_T const size_t UpperBS(const Vec<T> &vec, const T &target)
        {
            return std::upper_bound(ALL(vec), target) - vec.begin();
        }

    }
    namespace LinearAlgebra
    {
        TMP_T constexpr decltype(auto) NewMat(int n, int m, T init) { return Mat<T>(n, Vec<T>(m, init)); }
        TMP_T constexpr decltype(auto) MatSize(const Mat<T> &mat) { return TUPLE(mat.size(), mat[0].size()); }
    }
    namespace GraphAlgo
    {
        /**
         * @brief 链式前向星数据结构
         */
        TMP_T struct LFSEdge
        {
            i32 next, to;
            T weight;
        };
        /**
         * @brief 链式前向星存图，加 Dijkstra、SPFA 求最短路
         */
        template <typename T, i32 MaxSize>
        class ListForwardStar
        {
        public:
            ListForwardStar(i32 pointNum) : cntP(pointNum), cntE(0) { head.fill(-1); }

            /**
             * @brief 加边
             */
            void AddEdge(i32 u, i32 v, T w) noexcept
            {
                edge[cntE].to = v;         // 终点
                edge[cntE].weight = w;     //.权值
                edge[cntE].next = head[u]; // 以 u 为起点的最后一条边的编号
                                          // 也就是与这个边起点相同的上一条边的编号
                head[u] = cntE++;          // 更新以 u 为起点的上一条边的编号
                                          // 也就是刚插入的这条边
            }

            /**
             * @brief SPFA 求最短路。输入源点，返回从源点到各个点最小距离
             */
            Vec<T> SPFASolve(i32 source, T InitInf = Inf)
            {
                auto dist = Vec<T>(cntP, InitInf);
                auto inque = BitSet<MaxSize>();
                dist[source] = 0;
                std::queue<i32> q({source});
                inque.flip(source);
                while (!q.empty())
                {
                    i32 u = q.front();
                    q.pop();
                    inque.flip(u);
                    for (i32 i = head[u]; i != -1; i = edge[i].next)
                    {
                        i32 v = edge[i].to;
                        T w = edge[i].weight;
                        if (dist[v] > dist[u] + w)
                        {
                            dist[v] = dist[u] + w;
                            if(inque[v] == 0)
                            {
                                q.push(v);
                                inque.flip(v);
                            }
                        }
                    }
                }
                return dist;
            }

            std::array<LFSEdge<T>, MaxSize> edge;
            std::array<i32, MaxSize> head;
            i32 cntP;
            i32 cntE;
        };
    }

    namespace StringAlgo
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
        str Manacher(const str &s)
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
            for (auto i = 0; i < l - 1; ++i)
            {
                mp[i] = (mx > i) ? std::min(mp[(idx << 1) - i], mx - i) : 1;
                while (i - mp[i] >= 0 && i + mp[i] < (i32)(ma.size()) && ma[i + mp[i]] == ma[i - mp[i]])
                    ++mp[i];
                if (mp[i] > mp[maxpos])
                    maxpos = i;
                if (i + mp[i] > mx)
                    mx = i + mp[i], idx = i;
            }
            str ret;
            for (auto i = maxpos - mp[maxpos] + 1; i < maxpos + mp[maxpos]; ++i)
                if (ISDIGIT(ma[i]))
                    ret += ma[i];
            return ret;
        }

    }
    namespace Mathematic
    {
        /**
         * @brief gcd
         */
        i32 Gcd(i32 a, i32 b) { return b == 0 ? a : Gcd(b, a % b); }

        /**
         * @brief 快速幂
         */
        i64 FastPow(i64 x, const i64 n)
        {
            if (n == 0)
                return 1;
            i64 ret = 1;
            while (n)
            {
                x *= x;
                if (n & 1)
                {
                    ret *= x;
                }
            }
            return ret;
        }

        /**
         * @brief 带 mod 快速幂
         */
        i64 FastPow(i64 x, const i64 n, const i64 mod)
        {
            if (n == 0)
                return 1 % mod;
            i64 ret = 1;
            while (n)
            {
                x = x * x % mod;
                if (n & 1)
                {
                    ret = ret * x % mod;
                }
            }
            return ret % mod;
        }

        /**
         * @brief x 进制转 y 进制
         */
        str Transform(i32 x, i32 y, str valBaseX)
        {
            assert(x >= 2 && x <= 36 && y >= 2 && y <= 36);
            str valBaseY;
            i64 sum = 0;
            for (auto c : valBaseX)
            {
                if (c == '-')
                    continue;
                if (ISDIGIT(c))
                    sum = sum * x + (c - '0');
                else
                    sum = sum * x + (c - 'A' + 10);
            }
            while (sum)
            {
                i8 tmp = sum % y;
                sum /= y;
                if (tmp < 9)
                    tmp += '0';
                else
                    tmp += 'A' - 10;
                valBaseY += tmp;
            }
            if (valBaseY.empty())
                valBaseY = "0";
            if (*valBaseX.begin() == '-')
                valBaseY = "-" + valBaseY;
            return valBaseY;
        }

        namespace PrimeSieve
        {
            /**
            * @brief 埃拉托色尼筛，O(Nlog N)
            */
            TMP_TV(i32)
            Vec<i64> Eratosthenes = []() -> Vec<i64>
            {
                std::array<i64, v> prime;
                i32 tot = 0;
                BitSet<v + 1> notPrime;
                notPrime.flip(0), notPrime.flip(1);
                FORINC(i, 2, v + 1)
                {
                    if (!notPrime[i])
                        prime[tot++] = i;
                    if (i * i <= v)
                        FORINC_STP(j, i * i, v + 1, i)
                        {
                            notPrime.flip(i);
                        }
                }
                return Vec<i64>(prime.begin(), prime.begin() + tot);
            }();

            /**
            * @brief 欧拉筛，O(N)
            */
            TMP_TV(i32)
            Vec<i64> Euler = []() -> Vec<i64>
            {
                std::array<i64, v> prime;
                i32 tot = 0;
                BitSet<v + 1> notPrime;
                FORINC(i, 2, v + 1)
                {
                    if (!notPrime[i])
                        prime[tot++] = i;
                    FORINC(j, 0, tot)
                    {
                        i64 p = prime[j];
                        if (i * p <= v)
                        {
                            notPrime.flip(p * i);
                            if (i % p == 0)
                                break;
                        }
                    }
                }
                return Vec<i64>(prime.begin(), prime.begin() + tot);
            }();
        }
    }

    // 4. Data Structure
    inline namespace
    {
        using Veci = Vec<i32>;
        using Vecl = Vec<i64>;
        using Vecf = Vec<f32>;
        using Vecd = Vec<f64>;
        using Pointi = std::pair<i32, i32>;
        using Pointl = std::pair<i64, i64>;
        using Pointf = std::pair<f32, f32>;
        using Pointd = std::pair<f64, f64>;
        TMP_T using MinHeap = std::priority_queue<T, Vec<T>, std::greater<T>>;
        TMP_T using MaxHeap = std::priority_queue<T, Vec<T>, std::less<T>>;

        TMP_TV(i32)
        class DisjointSet
        {
        public:
            DisjointSet()
            {
                rank.fill(0), parent.fill(-1);
            }

            /**
             * @brief 找祖先
             */
            i32 Find(i32 x) { return x == parent[x] ? x : (parent[x] = Find(parent[x])); }

            /**
             * @brief 按秩合并，检查 x y 是否在一个连通分量，在合并会成环，返回 false
             */
            bool Union(i32 x, i32 y)
            {
                i32 fx = Find(x), fy = Find(y);
                if (fx == fy)
                    return false;
                if (rank[fx] > rank[fy])
                    parent[fy] = fx;
                else
                {
                    parent[fx] = fy;
                    if (rank[fx] == rank[fy])
                        ++rank[fy];
                }
                return true;
            }

            // 祖先数组，提供修改
            std::array<i32, v> parent;

        private:
            std::array<i32, v> rank;
        };
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



