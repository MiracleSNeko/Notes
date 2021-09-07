# NOIP基础数据结构与算法

>   https://www.bilibili.com/video/BV1qj411f7NL

## 1. 基础数据结构

### 1.1 二叉堆

插入维护时间为 $O(N)$ 。层数 $h = O(log\ n)$，$i$ 层 $2^i$ 个数字 $h-i$ 深, $\sum 2^i * (n-i) = O(log \ n)$

```c++
int w[200002];

void down(int k)
{
    int l = 2*k, r = 2*k + 1;
    if(w[l] < w[k] || w[r] < w[k])
    {
        // 和较小的孩子交换
        int t = (w[l] < w[r])? l: r;
        swap(w[k], w[t]);
        down(t);
    } 
}

void up(int k)
{
    while (k > 1 && w[k] < w[k>>1])
    {
        // 和父亲交换
        swap(w[k], w[k>>1]);
        up(k>>1);
    }
}
```

```go
const MAX_LEN int = 110000

type ArrayHeap struct {
	Data [MAX_LEN]int
	Len  int
	Cmp  func(ah *ArrayHeap, i, j int) bool
	Init int
}

func (ah *ArrayHeap) Down(k int) {
	if k > (*ah).Len {
		return
	}
	l, r := 2*k, 2*k+1
	if (*ah).Cmp(ah, l, k) || (*ah).Cmp(ah, r, k) {
        if (*ah).Cmp(ah, l, r) {	// 一开始写成 (*ah).Cmp(ah, l, k) ，裂开
			(*ah).Data[k], (*ah).Data[l] = (*ah).Data[l], (*ah).Data[k]
			ah.Down(l)
		} else {
			(*ah).Data[k], (*ah).Data[r] = (*ah).Data[r], (*ah).Data[k]
			ah.Down(r)
		}
	}
}

func (ah *ArrayHeap) Up(k int) {
	for k > 1 && (*ah).Cmp(ah, k, k>>1) {
		(*ah).Data[k], (*ah).Data[k>>1] = (*ah).Data[k>>1], (*ah).Data[k]
		ah.Up(k >> 1)
	}
}

func (ah *ArrayHeap) Add(val int) bool {
	if (*ah).Len+1 == MAX_LEN {
		return false
	} else {
		(*ah).Len++
		(*ah).Data[(*ah).Len] = val
		(*ah).Up((*ah).Len)
		return true
	}
}

func (ah ArrayHeap) Top() int {
	return ah.Data[1]
}

func (ah *ArrayHeap) Pop() {
	l := (*ah).Len
	(*ah).Data[1], (*ah).Data[l] = (*ah).Data[l], (*ah).Init
	(*ah).Len--
	(*ah).Down(1)
}

func (ah *ArrayHeap) Peek() int {
	ret := (*ah).Data[1]
	(*ah).Pop()
	return ret
}

func NewHeap(cmp func(ah *ArrayHeap, i, j int) bool, init int) ArrayHeap {
	data := [MAX_LEN]int{0}
	for i := 1; i < MAX_LEN; i++ {
		data[i] = init
	}
	return ArrayHeap{
		data,
		0,
		cmp,
		init,
	}
}

func minCmp(ah *ArrayHeap, i, j int) bool {
	return (*ah).Data[i] < (*ah).Data[j]
}

func maxCmp(ah *ArrayHeap, i, j int) bool {
	return (*ah).Data[i] > (*ah).Data[j]
}
```



### 1.2 左偏树

定义一个树的斜深度为从根节点开始一直向右走到叶子节点的步数。左偏树是特殊的堆，满足左儿子大小不小于右儿子，关键操作：合并

每个节点斜深度 $O(log\ n)$

```c++
struct tree
{
    int l, r, size, w; // 权值
} t[110000];
// k1, k2 为根, 合并到 k1 为根, k2 合并为 k2 右儿子
int merge(int k1, int k2)
{
    if (k1 == 0 || k2 == 0) return k1 + k2;
    if (t[k1].w > t[k2].w) swap(k1, k2); // 小根堆
    t[k1].r = merge(t[k1].r, k2);
    // 满足左偏树性质
    if(t[t[k1].r].size > t[t[k1].l].size) swap(t[k1].l, t[k1].r);
    // 更新大小
    t[k1].size = t[t[k1].l].size + t[t[k1].r].size + 1;
    return k1;
}

// 插入变成和一个单节点左偏树合并
// 弹顶变成合并根的左右儿子
```

### 1.3 ST 表

给出一个数列 A，多次询问区间信息（最小，最大）

用 `ST[i][j]` 表示区间 $(i - 2^j, i]$ 间的信息，显然区间可分成 $(i-2^j + 1, i - 2^{j-1}]$ 和 $(i - 2^{j-1}, i]$

预处理 $O(Nlog\ N)$， 查询 $O(1)$

```c++
void init()
{
    for (int i = 1; i <= n; ++i) f[i][0] = a[i];
    for (int j = 1; j <= 20; ++j)
        for (int i = (1 << j); i <= n; ++i)
        	f[i][j] = min(f[i][j-1], f[i - (1 << (j-1))][j]);
}
```

查询 $[l, r]$ 时先找到满足刚好不够覆盖区间的最大 $2^x$ ，然后用 $[l, l+2^x]$ 和 $[r-2^x, r]$ 两个区间去覆盖 $[l, r]$

```c++
int query(int l, int r)
{
    int x = log(r- l + 1)/log2;
    return min(f[r][x], f[l + (1 << x) ][x])
}
```



## 2. 算法

### 2.1 带余快速幂

```go
const MOD := 1e9 + 7

func fastPowWithMod(x, n int64) (ret int64) {
	ret = 1
	for n > 0 {
		if n&1 == 1 {
			ret = (ret * x) % MOD
		}
		x = (x * x) % MOD
		n /= 2
	}
	return
}
```

### 2.2 树上LCA

```c++
// f[i][j] 表示 i 向上走 2^j 步到达的点
void init()
{
    for (int i = 1; i <= n; ++i) f[i][0] = p[i];
    for (int j = 1; j <= 20; ++j)
        for (int i = 1; i <= n; ++i)
            f[i][j] = f[f[i][j-1]][j-1];
}
int lca(int u, int v)
{
    // 第一步 u v 跳到同一层
    if (dep[u] > dep[v]) swap(u, v);
    for (int j = 20; j >= 0; --j) if (dep[f[v][j]] >= dep[u]) v = f[v][j];
    if (u == v) return u;
    // 第二步 从同一层往上跳。妙啊！
    for (int j = 20; j >= 0; --j) if (f[u][j] != f[v][j]) u = f[u][j], v = f[v][j];
	return f[u][0];
}
```

树上 LCA 可以带一些信息

```c++
// f[i][j] 表示 i 向上走 2^j 步到达的点
// g[i][j] 表示 i 向上走 2^j 步的信息和
// merge 表示一个抽象操作，可以是 max min sum 等
// 一边跳一边把信息累积进去
void init()
{
    for (int i = 1; i <= n; ++i) f[i][0] = p[i];
    for (int j = 1; j <= 20; ++j)
        for (int i = 1; i <= n; ++i)
            f[i][j] = f[f[i][j-1]][j-1], g[i][j] = merge(g[i][j-1],g[f[i][j-1]][j-1]);
}
Info lca(int u, int v)
{
    // 第一步 u v 跳到同一层
    if (dep[u] > dep[v]) swap(u, v);
    Info ret = 0;
    for (int j = 20; j >= 0; --j) if (dep[f[v][j]] >= dep[u]) 
        ret = merge(ret, g[v][j]), v = f[v][j];
    if (u == v) 
        ret = merge(ret, a[u]), return ret;
    // 第二步 从同一层往上跳。妙啊！
    for (int j = 20; j >= 0; --j) if (f[u][j] != f[v][j]) 
        ret = merge(merge(ret, g[u][j]), g[v]][j]), u = f[u][j], v = f[v][j];
   	ret = merge(merge(ret, g[u][0]), g[v][0]);
    u = f[u][0], ret = merge(ret, a[u]);
	return ret;
}
```

一个例子：求 u -> v 的点权和，可以用 s[u] 表示 u 到根的点权和，那么 $s_{u,v} = s[u] + s[v] - 2*s[lca(u, v)] + w[lca(u, v)]$

那么 s[u] 应该怎么维护? 
