---
title: Go 语言圣经笔记  第四章：复合数据类型
date: 2024-08-08 11:47:28
tags: [Go, Go语言圣经]
---

# 第四章


## 数组
- 默认情况下，数组的每个元素都被初始化为元素类型对应的0值。
- 示例

```go

q := [3]int{1, 2, 3}

type Currency int
const (
	USD Currency = iota
	EUR
	GBP
	RMB
)
symbol := [4]string{USD: "$", EUR: "€", GBP: "£", RMB: "¥"}
fmt.Println(RMB, symbol[RMB])


```
- 数组比较时， 只有各元素都可比较且一致时， 才认为数组相等
- 这里有个数组比较的例子

```go
package main

import (
	"crypto/sha256"
	"fmt"
)

func main(){
	c1 := sha256.Sum256([]byte("x"))
	c2 := sha256.Sum256([]byte("X"))
	fmt.Printf("%x\n%x\n%t\n%T\n", c1, c2, c1 == c2, c1)
}
```

### 练习4.1
- 编写一个函数，计算两个SHA256哈希码中不同bit的数目。（参考2.6.2节的PopCount函数。)

```go
package main

import (
	"crypto/sha256"
	"fmt"
)

var pc [256]byte

func init() {
    for i := range pc {
        pc[i] = pc[i/2] + byte(i&1)
    }
}

func main() {
	c1 := sha256.Sum256([]byte("x"))
	c2 := sha256.Sum256([]byte("X"))
	ans := 0
	for i := 0; i < len(c1); i++ {
		ans += int(pc[c1[i]^c2[i]])
	}
	fmt.Println(ans)
}
```

### 练习4.2
- 编写一个程序，默认情况下打印标准输入的SHA256编码，并支持通过命令行flag定制，输出SHA384或SHA512哈希算法。

```go
//编写一个程序，默认情况下打印标准输入的SHA256编码，并支持通过命令行flag定制，输出SHA384或SHA512哈希算法。

package main

import (
	"crypto/sha256"
	"crypto/sha512"
	"flag"
	"fmt"
)

var hashMethod = flag.Int("s", 256, "hash method default:256 other:384, 512")
func main() {
	flag.Parse()
	var s string
	fmt.Printf("输入字符串")
	fmt.Scanln(&s)
	switch *hashMethod{
		case 256:
			fmt.Printf("%x 1\n", sha256.Sum256([]byte(s)))
		case 384:
			fmt.Printf("%x 2\n", sha512.Sum384([]byte(s)))
		case 512:
			fmt.Printf("%x 3\n", sha512.Sum512([]byte(s)))
		default:
			fmt.Printf("输入错误"u
}
```

## Slice

### 共通点
- 语法相近,slice只是没有固定长度。

### 区别
- slice的第一个元素不一定是数组的第一个元素。
- slice的容量是指从slice开始地址到数组结束地址的距离。 使用cap函数可以获取slice的容量。
- slice的容量和长度可以不一样。多个slice可以指向同一个数组。
- slice不能直接判断是否相等，但是可以通过比较其长度和元素是否相等来判断。
- 
```

```
