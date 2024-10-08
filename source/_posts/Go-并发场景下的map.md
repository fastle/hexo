---
title: Go 并发场景下的map
tags:
  - 随笔
  - Go
date: 2024-09-13 16:02:18
---
# 原生 map
原生 map 是线程不安全的，需要加锁。

```go
func main() {
	m := make(map[string]int, 2)
	m["dd"] = 22
	go func() {
		for {
			m["ff"] = 1
		}
	}()
	go func() {
		for {
			_ = m["dd"]
		}
	}()
	time.Sleep(1 * time.Hour)
}
```

使用上面代码会报 `fatal error: concurrent map read and map write` 的错误。

最原始的解决方法是在 CRUD 操作时全部都加 Mutex 锁， 然后我们发现并发读并无不妥， 所以可以改成 sync.RWMutex 提升效率。

另外标准库中提供了一种官方实现的线程安全的 map 结构，即 sync.Map 。官方建议仅在每个 key 只写入一次但是要读许多次， 并且多个 goroutine 使用的 key 不相交时使用。 其底层逻辑为使用两个 map 一个为只读的 Read ， 另一个是可写的 Dirty， 两者之间的元素可以进行动态转换， 从而规避加锁操作。
