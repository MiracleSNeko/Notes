# 高阶数据结构与算法



## 1. 数据结构



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

