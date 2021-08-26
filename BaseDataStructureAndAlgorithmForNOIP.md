# NOIP基础数据结构与算法

>   https://www.bilibili.com/video/BV1qj411f7NL

## 1. 基础数据结构

### 二叉堆

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

### 左偏树

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

