---
title: Go 语言圣经笔记  第六章：方法
date: 2024-08-09 15:22:07
tags:
 - Go
 - Go语言圣经
---


# 第六章 方法
- 在函数声明时， 在其名字前面放上一个变量， 就是一个方法， 这个附加的参数会将该函数附加到这种类型上，相当于我们对于这种类型建立了一种独立的方法。


```go
// 建立方法样例

package geometry

import "math"

type Point struct {X, Y float64}

func Distance(p, q Point) float64 {
	return math.Hypot(q.X - p.X, q.Y - p.Y)
}

func (p Point) Distance(q Point) float64 { // p 是方法的接收器
	return math.Hypot(q.X - p.X, q.Y - p.Y)
}
```
- 可以让代码更简洁

## 嵌入结构体扩展类型
- 看下面例子
```go
type Point struct{ X, Y float64 }

type ColoredPoint struct {
    Point
    Color color.RGBA
}
```
- 则可以直接访问 ColoredPoint.X
- 其逻辑是先从第一层取找元素或者方法， 如果没有就去第二层找， 以此类推, 如果同一层有冲突， 报错
- 注意， 这种关系并不是继承， 不是 ColorPoint is a Point, 而是 ColorPoint has a Point

- 同时， 我们可以将方法绑定到方法变量上， 写法及示例如下。
```go
type Point struct (X, Y float64)
func (p Point) Add(q Point) Point {return Point{p.X + q.X, p.Y + q.Y}}
func (p Point) Sub(q Point) Point {return Point{p.X - q.X, p.Y - q.Y}}
type Path []Point
func (path Path) TranslateBy(offset Point) {
	var op func(p, q Point) Point
	if add {
		op = Point.Add
	} else {
		op = Point.Sub  // 这里可以赋予不同的值， 来使得后面的代码简略
	}
	for i := range path {
		path[i] = op.(path[i], offset)	
	}
}
```

## 示例， bit数组

```go
// intset
package main

import (
	"bytes"
	"fmt"
)

type IntSet struct {
	words []uint64 
}

func  (s *IntSet) Has(x int) bool {
	word, bit := x / 64, uint(x % 64)
	return word < len(s.words) && s.words[word] & (1 << bit) != 0 
}

// 按位或
func (s *IntSet) Add(x int) {
	word, bit := x / 64, uint(x % 64)
	for word >= len(s.words) {
		s.words = append(s.words, 0)
	}
	s.words[word] |= 1 << bit
}

// 合并
func (s *IntSet) UnionWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] |= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

func (s *IntSet) String() string {
	var buf bytes.Buffer 
	buf.WriteByte('{')
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < 64; j++ {
			if word & (1 << uint(j)) != 0 {
				if buf.Len() > len("{") {
					buf.WriteByte(' ')
				}
				fmt.Fprintf(&buf, "%d", 64 * i + j)
			}
		}
	}
	buf.WriteByte('}')
	return buf.String()
}

func main(){
	var x, y IntSet 
	x.Add(1)
	x.Add(144)
	x.Add(9)
	fmt.Println(x.String()) // "{1 9 144}"
	y.Add(9)
	y.Add(42)
	fmt.Println(y.String()) // "{9 42}"
	x.UnionWith(&y)
	fmt.Println(x.String()) // "{1 9 42144}")
}
```


### 练习6.1
- 为bit数组实现下面这些方法

```go
func (*IntSet) Len() int      // return the number of elements
func (*IntSet) Remove(x int)  // remove x from the set
func (*IntSet) Clear()        // remove all elements from the set
func (*IntSet) Copy() *IntSet // return a copy of the set
```

```go
// intset
package main

import (
	"bytes"
	"fmt"
)

type IntSet struct {
	words []uint64 
}

func  (s *IntSet) Has(x int) bool {
	word, bit := x / 64, uint(x % 64)
	return word < len(s.words) && s.words[word] & (1 << bit) != 0 
}

// 按位或
func (s *IntSet) Add(x int) {
	word, bit := x / 64, uint(x % 64)
	for word >= len(s.words) {
		s.words = append(s.words, 0)
	}
	s.words[word] |= 1 << bit // 
}

// 合并
func (s *IntSet) UnionWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] |= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

func (s *IntSet) String() string {
	var buf bytes.Buffer 
	buf.WriteByte('{')
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < 64; j++ {
			if word & (1 << uint(j)) != 0 {
				if buf.Len() > len("{") {
					buf.WriteByte(' ')
				}
				fmt.Fprintf(&buf, "%d", 64 * i + j)
			}
		}
	}
	buf.WriteByte('}')
	return buf.String()
}

func (s *IntSet) Len() int {     // return the number of elements
	ans := 0
	for _, e := range s.words {
		for e != 0 {
			ans ++
			e &= e - 1
		}
	}
	return ans 
}
func (s *IntSet) Remove(x int) { // remove x from the set
	word, bit := x / 64, uint(x % 64)
	bit = ^bit // Go 中取反的写法
	s.words[word] |= 1 << bit // 

}
func (s *IntSet) Clear() {       // remove all elements from the set
	s.words = nil
}
func (s *IntSet) Copy() *IntSet {// return a copy of the set
	ans := new(IntSet)
	for _, e := range s.words {
		ans.words = append(ans.words, e)
	}
	return ans
}
func main(){
	var x, y IntSet 
	x.Add(1)
	x.Add(144)
	x.Add(9)
	fmt.Println(x.String()) // "{1 9 144}"
	y.Add(9)
	y.Add(42)
	fmt.Println(y.String()) // "{9 42}"
	x.UnionWith(&y)
	fmt.Println(x.String()) // "{1 9 42144}")
}

```

### 练习6.2
- 定义一个变参方法(*IntSet).AddAll(...int)，这个方法可以添加一组IntSet，比如s.AddAll(1,2,3)。
```go
func (s *IntSet) AddAll(x ...int) {
	for _, v := range x {
		s.Add(v)
	}
}

```

### 练习6.3 
- (*IntSet).UnionWith会用|操作符计算两个集合的并集，我们再为IntSet实现另外的几个函数IntersectWith（交集：元素在A集合B集合均出现），DifferenceWith（差集：元素出现在A集合，未出现在B集合），SymmetricDifference（并差集：元素出现在A但没有出现在B，或者出现在B没有出现在A）。

```go
func (s *IntSet) IntersectWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] &= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

func (s *IntSet) DifferenceWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] &= ^tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

func (s *IntSet) SymmetricDifference(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] ^= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}
```

### 练习6.4
-  实现一个Elems方法，返回集合中的所有元素，用于做一些range之类的遍历操作。

```go
func (s *IntSet) Elems() []int {
	var ans []int
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < 64; j++ {
			if word & (1 << uint(j)) != 0 {
				ans = append(ans, 64 * i + j) 
			}
		}
	}
	return ans 
}

```

### 练习6.5
- 我们这章定义的IntSet里的每个字都是用的uint64类型，但是64位的数值可能在32位的平台上不高效。修改程序，使其使用uint类型，这种类型对于32位平台来说更合适。当然了，这里我们可以不用简单粗暴地除64，可以定义一个常量来决定是用32还是64，这里你可能会用到平台的自动判断的一个智能表达式：32 << (^uint(0) >> 63)

```go
// 根据机器型号， 来进行操作, 直接把字长设置成wordSize = 32 << (^uint(0) >> 32 & 1)
package main

import (
	"bytes"
	"fmt"
)

const (
	wordSize = 32 << (^uint(0) >> 32 & 1)
)

type IntSet struct {
	words []uint64 
}

func  (s *IntSet) Has(x int) bool {
	word, bit := x / wordSize, uint(x % wordSize)
	return word < len(s.words) && s.words[word] & (1 << bit) != 0 
}

// 按位或
func (s *IntSet) Add(x int) {
	word, bit := x / wordSize, uint(x % wordSize)
	for word >= len(s.words) {
		s.words = append(s.words, 0)
	}
	s.words[word] |= 1 << bit
}

// 合并
func (s *IntSet) UnionWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] |= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

func (s *IntSet) String() string {
	var buf bytes.Buffer 
	buf.WriteByte('{')
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < wordSize; j++ {
			if word & (1 << uint(j)) != 0 {
				if buf.Len() > len("{") {
					buf.WriteByte(' ')
				}
				fmt.Fprintf(&buf, "%d", wordSize * i + j)
			}
		}
	}
	buf.WriteByte('}')
	return buf.String()
}

func main(){
	var x, y IntSet 
	x.Add(1)
	x.Add(144)
	x.Add(9)
	fmt.Println(x.String()) // "{1 9 144}"
	y.Add(9)
	y.Add(42)
	fmt.Println(y.String()) // "{9 42}"
	x.UnionWith(&y)
	fmt.Println(x.String()) // "{1 9 42144}")
}
```
- 一个对象的变量或者方法对于调用方是不可见的话， 一般就被定义为封装， 封装有时候也被叫做数据隐藏，
- Go 只有一种控制可见性的方法， 大写首字母的标识符会被定义他们的包导出， 小写字母的则不会被导出。
- 同时适用于struct 或者一个类型的方法

