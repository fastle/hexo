---
title: Go 语言圣经笔记  第二章：程序结构
date: 2024-08-01 19:40:19
tags: [Go, Go语言圣经]
---

# 第二章
- 实体的第一个字母的大小写决定其可见性是否跨包， 如果是大写开头， 说明是导出的， 可以被自己包之外的其他程序所调用
- 包名称永远是小写纯字母
- 名称的作用域越大，就使用越长且更有意义的名称
- 驼峰式命名法，首字母缩写词往往使用相同的大小写
- go中不允许出现未被定义的变量， 所有类型的变量都应当有直接可用的零值

```go
package main

import "fmt"

func main() {
	const freezingF, boilingF = 32.0, 212.0
	fmt.Printf("freezing %g C\n",fToC(freezingF))
	fmt.Printf("boiling %g C\n", fToC(boilingF))
}


func fToC(f float64) float64{
	return (f -32) * 5 / 9
}
```


```go
package main

import "fmt"

const boilingF = 212.0

func main() {
	var f = boilingF
	var c = (f - 32) * 5 / 9
	fmt.Printf("boiling point = %g F or %g C\n", f, c)
}

```

```go
// 第四版
package main

import (
	"flag"
	"fmt"
	"strings"
)

var n = flag.Bool("n", false, "omit trailing newline")
var sep = flag.String("s", " ", "separator")


func main(){
	flag.Parse()
	fmt.Print(strings.Join(flag.Args(), *sep))
	if !*n {
		fmt.Println()
	}
}


```
- flag包简介：https://www.cnblogs.com/sparkdev/p/10812422.html

### 类型声明
- type name underlying-type
- 一般会放在函数外面全包使用， 若首字母大写则可导出包外

```go
// 进行摄氏温度和华氏温度的转换
package main


type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC Celsius = 0
	BoilingC Celsius = 100
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c * 9 / 5 + 32)}  // 构造时若两个底层是相同类型可以直接构造
func FtoC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9)}
```


- 命名类型之后类似于继承，可以重新定义类型的行为， 类似于下面
```go
func (c Celsius) String() string() {return fmt.Sprintf("%g°C", c)} // fmt 在将元素输出时，会优先调用函数的toString（）方法
```


### 包
- 每个包对应一个独立的命名空间， 需要明确指出包来调用， 只有名字以大写字母开头的信息才是导出的， （汉字不导出）

- 可以将之前的代码分成两个文件， 并且导出包

```go
// 用于进行摄氏度与华氏度之间的转换   tempconv.go
package tempconv 


type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC Celsius = 0
	BoilingC Celsius = 100
)

```

```go 
package tempconv // conv.go

// 摄氏度转华氏度
func CToF(c Celsius) Fahrenheit { return Fahrenheit(c * 9 / 5 + 32)}  // 构造时若两个底层是相同类型可以直接构造

// 华氏度转摄氏度
func FtoC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9)}
```

- flag 

```go
// 练习2.1 注意函数复用
// 进行摄氏温度和华氏温度以及绝对温度的转换
package main


type Celsius float64
type Fahrenheit float64
type Kelvin float64 

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC Celsius = 0
	BoilingC Celsius = 100
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c * 9 / 5 + 32)}  // 构造时若两个底层是相同类型可以直接构造
func FtoC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9)}
func KtoC(k Kelvin) Celsius {return Celsius(k + Kelvin(AbsoluteZeroC))}
func KtoF(k Kelvin) Fahrenheit {return Fahrenheit(CToF(KtoC(k)))}
func CtoK(c Celsius) Kelvin {return Kelvin(c - AbsoluteZeroC)}
func FtoK(f Fahrenheit) Kelvin {return CtoK(FtoC(f))}
```


### 导入包

```go

// 导入tempconv包
package main

import (
	"fmt"
	"os"
	"strconv"

	"./learning/tempconv" // go 调用不同位置的包 ，https://blog.csdn.net/Working_hard_111/article/details/139982343
)

func main(){
	for _, arg := range os.Args[1:]{
		t, err := strconv.ParseFloat(arg, 64)
		if err != nil {
			fmt.Fprintf(os.Stderr, "cf: %v\n", err)
			os.Exit(1)
		}
		f := tempconv.Fahrenheit(t)
		c := tempconv.Celsius(t)
		fmt.Printf("%s = %s, %s = %s\n", f, tempconv.FtoC(f), c, tempconv.CToF(c))
	}
}
```

- 包的初始化， 使用init（）函数， 该函数不能被调用或者引用， 每个文件中init初始化函数在程序执行的时候直接调用
```go
// 用来统计输入数的二进制1数目
package popcount

var pc [256]byte 

func init(){
	for i := range pc { // 直接可以将slice当参数
		pc[i] = pc[i / 2] + byte(i & 1) // byte 可以返回1的个数, pc[i] 表示数字i 二进制时1的位置个数
	}
}

func PopCount(x uint64) int{
	return int(pc[byte(x >> (0 * 8))] +
		pc[byte(x >> (1 * 8))] +
		pc[byte(x >> (2 * 8))] +
		pc[byte(x >> (3 * 8))] +
		pc[byte(x >> (4 * 8))] +
		pc[byte(x >> (5 * 8))] +
		pc[byte(x >> (6 * 8))] +
		pc[byte(x >> (7 * 8))])
}
```


#### 练习2.3
- 重写PopCount函数，用一个循环代替单一的表达式。比较两个版本的性能。（11.4节将展示如何系统地比较两个不同实现的性能。）

```go
// 用来统计输入数的二进制1数目
package popcount

var pc [256]byte 

func init(){
	for i := range pc { // 直接可以将slice当参数
		pc[i] = pc[i / 2] + byte(i & 1) // byte 可以返回1的个数, pc[i] 表示数字i 二进制时1的位置个数
	}
}

func PopCount(x uint64) int{

	ans := 0
	for i := 0 ; i < 8; i++ {  // 写成循环形式
		ans += int(byte(x >> (i * 8)))
	}
	return ans
}
```

#### 练习2.4 
-  用移位算法重写PopCount函数，每次测试最右边的1bit，然后统计总数。比较和查表算法的性能差异。

```go
// 用来统计输入数的二进制1数目
package popcount

var pc [256]byte 

func init(){
	for i := range pc { // 直接可以将slice当参数
		pc[i] = pc[i / 2] + byte(i & 1) // byte 可以返回1的个数, pc[i] 表示数字i 二进制时1的位置个数
	}
}

func PopCount(x uint64) int{

	ans := 0
	for ; x != 0 ; x >>= 1{   // 每次右移一位
		if x & 1 == 1{
			ans ++
		}
	}
	return ans
}

```

#### 练习2.5
- 表达式x&(x-1)用于将x的最低的一个非零的bit位清零。使用这个算法重写PopCount函数，然后比较性能。

```go
// 用来统计输入数的二进制1数目
package popcount

var pc [256]byte 

func init(){
	for i := range pc { // 直接可以将slice当参数
		pc[i] = pc[i / 2] + byte(i & 1) // byte 可以返回1的个数, pc[i] 表示数字i 二进制时1的位置个数
	}
}

func PopCount(x uint64) int{

	ans := 0
	for ; x != 0 ; x = x & (x - 1){   // x - lowbit(x)
			ans ++
	}
	return ans
}
```

### 作用域
- 作用域不等于生命周期， 作用域是编码阶段的概念，生命周期是运行时的概念
- go 中编译器会一层层地向外搜寻合适的范围， 
- for, if, switch 会产生新的词法域
- 这个部分要注意好的编码习惯， 尽量不用相同的变量名， 但是go是允许使用相同的变量名的