---
title: Go 语言圣经笔记  第七章：接口
date: 2024-08-09 15:39:35
tags: [Go, Go语言圣经]
---

# 第七章 接口
- 接口类型是对其他类型行为的概括和抽象， 通过使用接口， 我们可以写出更加灵活和通用的函数
- 接口类型不会暴露出他所代表的对象的内部值的结构， 只会暴露出自己的方法， 也就是你并不知道他是怎么做的，只知道他是做什么的

### 练习7.1
-  使用来自ByteCounter的思路，实现一个针对单词和行数的计数器。你会发现bufio.ScanWords非常的有用。
-  
```go

type WordCounter int
type LineCounter int

func (c *WordCounter) Write(p []byte) (int, error) {
	var sc = bufio.NewScanner(bytes.NewReader(p))
	sc.Split(bufio.ScanWords)
	for sc.Scan() {
		*c++
	}
	return int(*c), nil
}

func (c *LineCounter) Write(p []byte) (int, error) {
	var sc = bufio.NewScanner(bytes.NewReader(p))
	sc.Split(bufio.ScanLines)
	for sc.Scan() {
		*c++
	}
	return int(*c), nil
}

```

### 练习7.2 
-  写一个带有如下函数签名的函数CountingWriter，传入一个io.Writer接口类型，返回一个把原来的Writer封装在里面的新的Writer类型和一个表示新的写入字节数的int64类型指针。
-  

```go
// 传入一个io.Writer接口类型，返回一个把原来的Writer封装在里面的新的Writer类型和一个表示新的写入字节数的int64类型指针。
// f返回的是一个新的类型， 和Printf的封装不是很一样
package main

import (
	"fmt"
	"io"
	"os"
)

type CountWriter struct {
	Writer io.Writer 
	Count int64 
} 

func (cw *CountWriter) Write (content []byte) (int, error) {
	n, err := cw.Writer.Write(content)
	if err != nil {
		return n, err
	}
	cw.Count += int64(len(content))
	return n, nil
}


func CountingWriter(w  io.Writer)(io.Writer, *int64) {
	cw := CountWriter{Writer: w}
	return &cw, &cw.Count // 
}

func main() {
	cw, counter := CountingWriter(os.Stdout)
	fmt.Fprintf(cw, "%s", "Print somethind to the screen...") // 在这里调用Write 的时候计数器才加的
	fmt.Println(*counter)
}
```

### 练习7.3
-  为在gopl.io/ch4/treesort (§4.4)的*tree类型实现一个String方法去展示tree类型的值序列。
-  跳过
-  

## 接口类型
- 一些常见接口

```go
package io 
type Reader interface { // 关键词是type 
	Read(p []byte) (n int, err error)
}

type Closer interface {
	Close() error 
}

type ReadWriter interface { // 可以使用已有的接口类型来进行组合
	Reader
	Writer 
}
```
- 发现可以使用已有的接口类型简写命名接口， 这种方式叫做接口内嵌


### 练习7.4
-  strings.NewReader函数通过读取一个string参数返回一个满足io.Reader接口类型的值（和其它值）。实现一个简单版本的NewReader，用它来构造一个接收字符串输入的HTML解析器
```go
// strings.NewReader函数通过读取一个string参数返回一个满足io.Reader接口类型的值（和其它值）。实现一个简单版本的NewReader，用它来构造一个接收字符串输入的HTML解析器
// 我的理解是， 构建一个一个NewReader函数， 输入html 字符串， 返回一个自定义reader， 然后传到html.Parse() 中即可
package main

import (
	"fmt"
	"io"
	"os"

	"golang.org/x/net/html"
)

/*
type Reader interface {
	Read(p []byte) (n int, err error)
}


*/

/*
返回的是一个string.Reader

func (r *Reader) Read(b []byte) (n int, err error) {
	if r.i >= int64(len(r.s)) {
		return 0, io.EOF
	}
	r.prevRune = -1
	n = copy(b, r.s[r.i:])
	r.i += int64(n)
	return
}
*/

type MyReader struct {
	s string
	i int64
}

func (r *MyReader) Read(b []byte) (n int, err error) {
	if r.i >= int64(len(r.s)) {
		return 0, io.EOF
	}
	n = copy(b, r.s[r.i:])
	r.i += int64(n)
	return n, nil 
}


func NewReader(s string) *MyReader {
	return &MyReader{s, 0}
}

func main() {
	readernow := NewReader("<html><head></head><body><a href=\"this is a test\">aaa</a></body></html>")
	doc, err := html.Parse(readernow)
	if err != nil {
		fmt.Fprintf(os.Stderr, "ex7.4: %v\n", err)
		os.Exit(1)
	}
	for _, n := range visit(nil, doc) {
		fmt.Println(n)
	}
}

func visit(links []string, n *html.Node) []string {
	if n.Type == html.ElementNode && n.Data == "a" {
		for _, a := range n.Attr {
			if a.Key == "href" {
				links = append(links, a.Val)
			}
		}
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling { 
		links = visit(links, c) // 递归调用， 
	}
	return links
}


```

### 练习7.5
-  io包里面的LimitReader函数接收一个io.Reader接口类型的r和字节数n，并且返回另一个从r中读取字节但是当读完n个字节后就表示读到文件结束的Reader。实现这个LimitReader函数：

```go
// io包里面的LimitReader函数接收一个io.Reader接口类型的r和字节数n，并且返回另一个从r中读取字节但是当读完n个字节后就表示读到文件结束的Reader。实现这个LimitReader函数：s
package main

import (
	"io"
	"strings"
)

type LimitType struct {
	n int64
	i int64 
	w io.Reader
}

func (r *LimitType) Read(p []byte) (n int, err error) {
	if r.i >= r.n {
		return 0, io.EOF 
	}
	if r.i + int64(len(p)) > r.n {
		p = p[:r.n - r.i]
	} 
	n, err = r.w.Read(p)
	r.i += int64(n)
	return 
}

func LimitReader(r io.Reader, n int64) io.Reader {
	return &LimitType{n, 0, r}
}

func main(){
	r := LimitReader(io.Reader(strings.NewReader(string("hello world"))), 5)
	for {
		b := make([]byte, 1)
		n, err := r.Read(b)
		if err != nil {
			break
		}
		println(string(b[:n]))
	}
}
```

### 练习7.6
-  对tempFlag加入支持开尔文温度。
```go
// 对tempFlag加入支持开尔文温度。
/*
package flag

// Value is the interface to the value stored in a flag.
type Value interface {
    String() string
    Set(string) error
}


*/
package main

import (
	"flag"
	"fmt"
)


type Celsius float64
type Fahrenheit float64
type Kelvin float64 

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC Celsius = 0
	BoilingC Celsius = 100
)


type celsiusFlag struct{ Celsius }

func (f *celsiusFlag) Set(s string) error {
    var unit string
    var value float64
    fmt.Sscanf(s, "%f%s", &value, &unit)
    switch unit {
    case "C", "°C":
        f.Celsius = Celsius(value)
        return nil
    case "F", "°F":
        f.Celsius = FToC(Fahrenheit(value))
        return nil
	case "K", "°K":
		f.Celsius = KToC(Kelvin(value))
		return nil 
	}
	return fmt.Errorf("invalid temperature %q", s)
}
func (c Celsius) String() string { return fmt.Sprintf("%g°C", c)}

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c * 9 / 5 + 32)}
func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9)}
func KToC(k Kelvin) Celsius {return Celsius(k + Kelvin(AbsoluteZeroC))}
func KToF(k Kelvin) Fahrenheit {return Fahrenheit(CToF(KToC(k)))}

func CToK(c Celsius) Kelvin {return Kelvin(c - AbsoluteZeroC)}

func FToK(f Fahrenheit) Kelvin {return CToK(FToC(f))}
func CelsiusFlag(name string, value Celsius, usage string) *Celsius {
    f := celsiusFlag{value}
    flag.CommandLine.Var(&f, name, usage) // 因为这里已经实现了String() 和Set() 所以可以调用
    return &f.Celsius
}

var temp = CelsiusFlag("temp", 20.0, "the temperature")

func main() {
    flag.Parse()
    fmt.Println(*temp)
}
```
### 练习7.7
- 解释为什么帮助信息在它的默认值是20.0没有包含°C的情况下输出了°C。
- 因为我们定义实现了Celsius.string， 在输出时调用了， 所以会直接输出
- 


## 实现接口条件
- 首先必须实现该接口需要的所有方法.
- 接口赋值规则对且看下面示例
```go
var w io.Writer
w = os.Stdout           // OK: *os.File has Write method
w = new(bytes.Buffer)   // OK: *bytes.Buffer has Write method
w = time.Second         // compile error: time.Duration lacks Write method

var rwc io.ReadWriteCloser
rwc = os.Stdout         // OK: *os.File has Read, Write, Close methods
rwc = new(bytes.Buffer) // compile error: *bytes.Buffer lacks Close method
```
- 甚至可以使用接口类型来定义变量， 并且赋值给接口变量
- 所以接口可以用来实现基于类的语言的接口的效果， 通过抽取共性形成接口， 并且通过组合接口来继承
- 区别是Go语言可以在需要时才定义新的抽象和分组， 并且不用修改原有类型的定义。
### 空接口
- 空接口类型是所有类型的超类型， 所有类型都实现了空接口， 空接口类型可以接收任何类型的值， 也可以返回任何类型的值
- 比如说 在我们经常使用的Printf 和 Fprintf函数中， 就使用空接口来接收任何值

### 接口值
- 概念上一个接口的值包含两部分：一个类型和一个值。 由于Go语言是静态类型的语言， 我们不认为他的类型是一个值
- 下面四个语句中， 变量w得到了三个不同的值，
```go
var w io.Writer //1
w = os.Stdout //2 
w = new(bytes.Buffer)//3
w = nil //1
```

- 引起的问题大概如下， 一个包含nil指针的接口不是nil接口
```go
const debug = true

func main() {
    var buf *bytes.Buffer
    buf = new(bytes.Buffer) 
    f(buf)
}

func f(out io.Writer) {
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
```
- 使用时会报错， out 是一个类型为*bytes.Buffer的指针，值是nil， 但是他并不是nil接口， 这时候会报错




## 常见接口解析

### flag.Value

- flag.Value 定义如下
```go
package flag

// Value is the interface to the value stored in a flag.
type Value interface {
    String() string
    Set(string) error
}
```
- 思考下面这个会休眠特定时间的程序
```go

var period = flag.Duration("period", 1*time.Second, "sleep period")

func main() {
    flag.Parse()
    fmt.Printf("Sleeping for %v...", *period)
    time.Sleep(*period)
    fmt.Println()
}
```
- 其中period是一个Duration类型的值， 并且实现了String和Set方法， 所以可以使用flag.Duration函数来创建period变量

- 我们对之前使用过的tempconv程序进行改编, 定义了String() 和set(), 并通过set分类自动输出转换后的信息， 
```go
// 进行摄氏温度和华氏温度的转换
package main

import (
	"flag"
	"fmt"
)


type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC Celsius = 0
	BoilingC Celsius = 100
)


type celsiusFlag struct{ Celsius }

func (f *celsiusFlag) Set(s string) error {
    var unit string
    var value float64
    fmt.Sscanf(s, "%f%s", &value, &unit) // no error check needed
    switch unit {
    case "C", "°C":
        f.Celsius = Celsius(value)
        return nil
    case "F", "°F":
        f.Celsius = FToC(Fahrenheit(value))
        return nil
    }
    return fmt.Errorf("invalid temperature %q", s)
}

func (f *celsiusFlag)String() string { return fmt.Sprintf("%g°C", f.Celsius) }

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c * 9 / 5 + 32)}  // 构造时若两个底层是相同类型可以直接构造
func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9)}

func CelsiusFlag(name string, value Celsius, usage string) *Celsius {
    f := celsiusFlag{value}
    flag.CommandLine.Var(&f, name, usage) /// 因为这里已经实现了String() 和Set() 所以可以调用
    return &f.Celsius
}

var temp = CelsiusFlag("temp", 20.0, "the temperature")

func main() {
    flag.Parse()
    fmt.Println(*temp)
}
```

### sort.Interface
- 

```go
package sort

type Interface interface {
    Len() int
    Less(i, j int) bool // i, j are indices of sequence elements
    Swap(i, j int)
}

```
### http.Handler 

```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

- ListenAndServe函数接收一个地址和Handler接口类型的值， 并且返回一个错误, 会一直运行， 直到遇见一个错误而失败

### error 
```go
type error interface {
    Error() string
}

```

### 类型断言
- 类型断言是Go语言中一个重要的特性， 它允许我们检查一个接口的值是否属于某个类型， 并且获取该类型的值表示为x.(T)
- 这里的T有两种可能， 一种是具体的类型， 这种情况的话， 成功则会从返回x的动态值， 失败的话则返回panic
- 另一种是接口， 若成功则并不获取动态值， 

- 如果采用下面这种写法， 则在失败的时候不会崩溃， 并且能能够获取到错误信息。
```go
var w io.Writer = os.Stdout
f, ok := w.(*os.File)      // success:  ok, f == os.Stdout
b, ok := w.(*bytes.Buffer) // failure: !ok, b == nil
```

### 通过断言识别错误类型
- 如何判断处理返回的错误的类型， 这是一个问题
- 一个幼稚的实现会通过检查错误信息里面是否含有某个字符串来检查
- 使用专门的类型os.PathError来表示结构化的错误值， 
- 建议即时处理， 不然如果调用类似fmt.Errorf 之类的方法后， 结构信息就没了。

- 接口有两种风格， 一种突出了满足这个接口的具体类型之间的相似性， 但是隐藏了各个具体类型的布局与各自特有的功能
- 另一种充分利用了接口值能够容纳各种具体类型的能力， 把接口作为这些类型的联合来使用。
- 
### sort.Interface
```go
type Interface interface {
	Len() int 
	Less(i, j int) bool
	Swap(i, j int) 
}
```
- 教材实例
```go
package main

import (
	"fmt"
	"os"
	"sort"
	"text/tabwriter"
	"time"
)


type Track struct {
	Title  string
	Artist string
	Album  string
	Year   int
	Length time.Duration
}

var tracks = []*Track{
    {"Go", "Delilah", "From the Roots Up", 2012, length("3m38s")},
    {"Go", "Moby", "Moby", 1992, length("3m37s")},
    {"Go Ahead", "Alicia Keys", "As I Am", 2007, length("4m36s")},
    {"Ready 2 Go", "Martin Solveig", "Smash", 2011, length("4m24s")},
}

func length(s string) time.Duration {
    d, err := time.ParseDuration(s)
    if err != nil {
        panic(s)
    }
    return d
}

func PrintTracks(tracks []*Track) {
	const format = "%v\t%v\t%v\t%v\t%v\t\n"
	tw := new(tabwriter.Writer).Init(os.Stdout,0, 8, 2, ' ', 0)
	fmt.Fprintf(tw, format, "Title", "Artist", "Album", "Year", "Length")
	fmt.Fprintf(tw, format, "-----", "------", "-----", "----", "------")
	for _, t := range tracks {
		fmt.Fprintf(tw, format, t.Title, t.Artist, t.Album, t.Year, t.Length)
	}
	tw.Flush() // 计算各列宽度， 输出表格
}
type byArtist []*Track  // 两种类型直接转， 但是会对应不同函数
func (x byArtist) Len() int {return len(x)}
func (x byArtist) Less(i, j int) bool {return x[i].Artist < x[j].Artist}

func (x byArtist) Swap(i, j int) {x[i], x[j] = x[j], x[i]}


type customSort struct {
	t []*Track 
	less func(x, y *Track) bool // 只需要自定义less比较方法
}

func (x customSort) Len() int {return len(x.t)}

func (x customSort) Less(i, j int) bool {return x.less(x.t[i], x.t[j])}

func (x customSort) Swap(i, j int) {x.t[i], x.t[j] = x.t[j], x.t[i]}

func main() {
	sort.Sort(byArtist(tracks))
	PrintTracks(tracks)
	sort.Sort(sort.Reverse(byArtist(tracks)))
	PrintTracks(tracks)
	sort.Sort(customSort{tracks, func(x, y *Track)bool {
		if x.Title != y.Title {
			return x.Title < y.Title
		}
		if x.Year != y.Year {
			return x.Year < y.Year
		}
		if x.Length != y.Length {
			return x.Length < y.Length
		}
		return false
	}})
	PrintTracks(tracks)
}
```
### 练习7.8
-  很多图形界面提供了一个有状态的多重排序表格插件：主要的排序键是最近一次点击过列头的列，第二个排序键是第二最近点击过列头的列，等等。定义一个sort.Interface的实现用在这样的表格中。比较这个实现方式和重复使用sort.Stable来排序的方式。
```go
// 设置一个键值slice， 每次追加一个键值
package main

import (
	"fmt"
	"os"
	"sort"
	"text/tabwriter"
	"time"
)

type Track struct {
	Title  string
	Artist string
	Album  string
	Year   int
	Length time.Duration
}

var tracks = []*Track{
	{"Go", "Delilah", "From the Roots Up", 2012, length("3m38s")},
	{"Go", "Moby", "Moby", 1992, length("3m37s")},
	{"Go Ahead", "Alicia Keys", "As I Am", 2007, length("4m36s")},
	{"Ready 2 Go", "Martin Solveig", "Smash", 2011, length("4m24s")},
}

func PrintTracks(tracks []*Track) {
	const format = "%v\t%v\t%v\t%v\t%v\t\n"
	tw := new(tabwriter.Writer).Init(os.Stdout,0, 8, 2, ' ', 0)
	fmt.Fprintf(tw, format, "Title", "Artist", "Album", "Year", "Length")
	fmt.Fprintf(tw, format, "-----", "------", "-----", "----", "------")
	for _, t := range tracks {
		fmt.Fprintf(tw, format, t.Title, t.Artist, t.Album, t.Year, t.Length)
	}
	tw.Flush() // 计算各列宽度， 输出表格
}

func length(s string) time.Duration {
    d, err := time.ParseDuration(s)
    if err != nil {
        panic(s)
    }
    return d
}


type table struct {
	t    []*Track
	keys []string // keys to sort by
}

func (t table) Len() int {
	return len(t.t)
}

func (t table) Less(i, j int) bool {
	for p := len(t.keys) - 1; p >= 0; p--{
		fmt.Println(t.keys[p])
		switch t.keys[p] {
		case "Title":
			if t.t[i].Title != t.t[j].Title {
				return t.t[i].Title < t.t[j].Title
			}
		case "Artist":
			if t.t[i].Artist != t.t[j].Artist {
				return t.t[i].Artist < t.t[j].Artist
			}
		case "Album":
			if t.t[i].Album != t.t[j].Album {
				return t.t[i].Album < t.t[j].Album
			}
		case "Year":
			if t.t[i].Year != t.t[j].Year {
				return t.t[i].Year < t.t[j].Year
			}
		}
	}
	return false // all keys are equal
}

func (t table) Swap(i, j int) {
	t.t[i], t.t[j] = t.t[j], t.t[i]
}

func setPrime(t *table, key string) {
	t.keys = append(t.keys, key)
}
func main() {
	table := table{tracks, []string{}}
	setPrime(&table, "Year")
	setPrime(&table, "Title")
	sort.Sort(table)
	PrintTracks(table.t)
}

```

### 练习7.9
- 使用html/template包（§4.6）替代printTracks将tracks展示成一个HTML表格。将这个解决方案用在前一个练习中，让每次点击一个列的头部产生一个HTTP请求来排序这个表格。

```go
//使用html/template包（§4.6）替代printTracks将tracks展示成一个HTML表格。将这个解决方案用在前一个练习中，让每次点击一个列的头部产生一个HTTP请求来排序这个表格。

package main

import (
	"fmt"
	"html/template"
	"io"
	"log"
	"net/http"
	"sort"
	"time"
)

type Track struct {
	Title  string
	Artist string
	Album  string
	Year   int
	Length time.Duration
}

var tracks = []*Track{
	{"Go", "Delilah", "From the Roots Up", 2012, length("3m38s")},
	{"Go", "Moby", "Moby", 1992, length("3m37s")},
	{"Go Ahead", "Alicia Keys", "As I Am", 2007, length("4m36s")},
	{"Ready 2 Go", "Martin Solveig", "Smash", 2011, length("4m24s")},
}

var trackTable = template.Must(template.New("Track").Parse(`
<h1> Tracks </h1>
<table>
<tr style='text-align: left'>
    <th onclick="submitform('Title')">Title
        <form action="" name="Title" method="post">
            <input type="hidden" name="orderby" value="Title"/>
        </form>
    </th>
    <th>Artist
        <form action="" name="Artist" method="post">
            <input type="hidden" name="orderby" value="Artist"/>
        </form>
    </th>
    <th>Album
        <form action="" name="Album" method="post">
            <input type="hidden" name="orderby" value="Album"/>
        </form>
    </th>
    <th onclick="submitform('Year')">Year
        <form action="" name="Year" method="post">
            <input type="hidden" name="orderby" value="Year"/>
        </form>
    </th>
    <th onclick="submitform('Length')">Length
        <form action="" name="Length" method="post">
            <input type="hidden" name="orderby" value="Length"/>
        </form>
    </th>
</tr>
{{range .T}}
<tr>
    <td>{{.Title}}</td>
    <td>{{.Artist}}</td>
    <td>{{.Album}}</td>
    <td>{{.Year}}</td>
    <td>{{.Length}}</td>
</tr>
{{end}}
</table>

<script>
function submitform(formname) {
    document[formname].submit();
}
</script>
`))

func length(s string) time.Duration {
    d, err := time.ParseDuration(s)
    if err != nil {
        panic(s)
    }
    return d
}


type table struct {
	T    []*Track
	keys []string // keys to sort by
}

func (t table) Len() int {
	return len(t.T)
}

func (t table) Less(i, j int) bool {
	for p := len(t.keys) - 1; p >= 0; p--{
		fmt.Println(t.keys[p])
		switch t.keys[p] {
		case "Title":
			if t.T[i].Title != t.T[j].Title {
				return t.T[i].Title < t.T[j].Title
			}
		case "Artist":
			if t.T[i].Artist != t.T[j].Artist {
				return t.T[i].Artist < t.T[j].Artist
			}
		case "Album":
			if t.T[i].Album != t.T[j].Album {
				return t.T[i].Album < t.T[j].Album
			}
		case "Year":
			if t.T[i].Year != t.T[j].Year {
				return t.T[i].Year < t.T[j].Year
			}
		}
	}
	return false // all keys are equal
}

func (t table) Swap(i, j int) {
	t.T[i], t.T[j] = t.T[j], t.T[i]
}

func setPrime(t *table, key string) {
	t.keys = append(t.keys, key)
}


func printTracks(w io.Writer, x *table) {
        trackTable.Execute(w, x)
}
func main() {
	table := table{tracks, []string{}}
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request){
		if err := r.ParseForm(); err != nil {
            fmt.Printf("ParseForm: %v\n", err)
        }
		for k, v := range r.Form {
			if k == "orderby" {
				setPrime(&table, v[0])
			}
		}
		sort.Sort(table)
		printTracks(w, &table)
	})
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

```

### 练习7.10
- sort.Interface类型也可以适用在其它地方。编写一个IsPalindrome(s sort.Interface) bool函数表明序列s是否是回文序列，换句话说反向排序不会改变这个序列。假设如果!s.Less(i, j) && !s.Less(j, i)则索引i和j上的元素相等。
```go
func IsPalindrome(s sort.Interface) bool {
	l := s.Len()
	for i := 0; i < l / 2; i++ {
		if s.Less(i, l - i - 1) != s.Less(l - i - 1, i) {
			return false
		}
	}
	return true 
}
```

### http.Handle 接口

```go
package http

type Handler interface {
	ServeHTTP(w ResponseWriter, r *Request)
}
// L
func ListenAndServe(address string, h Handler) error 
```

- 实现一个简单的电子商务网站如下

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	db := database{"shoes": 50, "socks": 5}
	log.Fatal(http.ListenAndServe("localhost:8000", db))
}

type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	for item, price := range db {
		fmt.Fprintf(w, "%s: %s\n", item, price)
	}
}
```

- 指定参数方法

```go

```

- 下面的程序中，我们创建一个ServeMux并且使用它将URL和相应处理/list和/price操作的handler联系起来，这些操作逻辑都已经被分到不同的方法中。然后我们在调用ListenAndServe函数中使用ServeMux为主要的handler。

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	db := database{"shoes": 50, "socks": 5}
	mux := http.NewServeMux()
	mux.Handle("/list", http.HandlerFunc(db.list))
	mux.Handle("/price", http.HandlerFunc(db.price))
	log.Fatal(http.ListenAndServe("localhost:8000", mux))
}

type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars


func (db database) list (w http.ResponseWriter, r *http.Request) {
	for item, price := range db {
		fmt.Fprintf(w, "%s: %s\n", item, price)
	}
}

func (db database) price(w http.ResponseWriter, r *http.Request) {
	item := r.URL.Query().Get("item") // 获取参数方法
		price, ok := db[item]
		if ! ok{
			w.WriteHeader(http.StatusNotFound) // 404
			fmt.Fprintf(w, "no such item: %q\n", item)
			return
		}
		fmt.Fprintf(w, "%s\n", price)
}


```

- 可以简写为
```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    http.HandleFunc("/list", db.list)
    http.HandleFunc("/price", db.price)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```


### 练习7.11
-  增加额外的handler让客户端可以创建，读取，更新和删除数据库记录。例如，一个形如 /update?item=socks&price=6 的请求会更新库存清单里一个货品的价格并且当这个货品不存在或价格无效时返回一个错误值。（注意：这个修改会引入变量同时更新的问题）

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"strconv"
)

func main() {
	db := database{"shoes": 50, "socks": 5}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}

type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	switch r.URL.Path {
	case "/list":
		for item, price := range db {
			fmt.Fprintf(w, "%s: %s\n", item, price)
		}
	case "/price":
		item := r.URL.Query().Get("item") // 获取参数方法
		price, ok := db[item]
		if ! ok{
			w.WriteHeader(http.StatusNotFound) // 404
			fmt.Fprintf(w, "no such item: %q\n", item)
			return
		}
		fmt.Fprintf(w, "%s\n", price)
	case "/create": 
		item := r.URL.Query().Get("item")
		price, err := strconv.ParseFloat(r.URL.Query().Get("price"), 32)
		if err != nil {
			w.WriteHeader(http.StatusBadRequest)
			fmt.Fprintf(w, "invalid price: %s\n", r.URL.Query().Get("price"))
			return
		}
		db[item] = dollars(price)
	case "/modify":
		item := r.URL.Query().Get("item")
		price, err := strconv.ParseFloat(r.URL.Query().Get("price"), 32)
		if err != nil {
			w.WriteHeader(http.StatusBadRequest)
			fmt.Fprintf(w, "invalid price: %s\n", r.URL.Query().Get("price"))
			return 
		}
		db[item] = dollars(price)
	case "/delete":
		item := r.URL.Query().Get("item")
		delete(db, item) // delete 为Go内置， 按照指定的键将元素从map中删除， 若删除的键为nil或者在map中不存在， 则不进行任何操作。
	default:
		w.WriteHeader(http.StatusNotFound)
		fmt.Fprintf(w, "no such page: %s\n", r.URL)	
	}
}
```

### 练习7.12
-  修改/list的handler让它把输出打印成一个HTML的表格而不是文本。html/template包（§4.6）可能会对你有帮助。

```go
// 练习7.12
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
)

func main() {
	db := database{"shoes": 50, "socks": 5}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}

type dollars float32
var itemTable = template.Must(template.New("Items").Parse(`
	<h1>Items</h1>
<table>
    <tr>
        <th> Item </th>
        <th> Price </th>
    </tr>
    {{ range $k, $v := . }}
        <tr>
            <td>{{ $k }}</td>
            <td>{{ $v }}</td>
        </tr>
    {{end}}
</table>
`))
func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	switch r.URL.Path {
	case "/list":
		itemTable.Execute(w, db)
		for item, price := range db {
			fmt.Fprintf(w, "%s: %s\n", item, price)
		}
	case "/price":
		item := r.URL.Query().Get("item") // 获取参数方法
		price, ok := db[item]
		if ! ok{
			w.WriteHeader(http.StatusNotFound) // 404
			fmt.Fprintf(w, "no such item: %q\n", item)
			return
		}
		fmt.Fprintf(w, "%s\n", price)
	
	default:
		w.WriteHeader(http.StatusNotFound)
		fmt.Fprintf(w, "no such page: %s\n", r.URL)	
	}
}
```

### error 接口

```go
type error interface {
    Error() string
}
```

### 接口示例 表达式求值
- 创建一个接口， 并进行相关测试

```go
\\ 整体代码结构较复杂， 这里仅展示前面信息的定义
package main

import (
	"fmt"
	"math"
	"strconv"
	"strings"
	"testing"
	"text/scanner"
)

type Expr interface {
	Eval(env Env) float64
}

type Var string
type literal float64
type Env map[Var]float64

type unary struct { // 一元运算符
	op rune
	x  Expr //当前定义下， 可以放var or literal
}

func (u unary) Eval(env Env) float64 {
	switch u.op {
	case '+':
		return +u.x.Eval(env)
	case '-':
		return -u.x.Eval(env)
	}
	panic(fmt.Sprintf("unsupported unary operator: %q", u.op))
}

type binary struct { // 二元运算符
	op   rune
	x, y Expr
}

func (u binary) Eval(env Env) float64 {
	switch u.op {
	case '+':
		return u.x.Eval(env) + u.y.Eval(env)
	case '-':
		return u.x.Eval(env) - u.y.Eval(env)
	case '*':
		return u.x.Eval(env) * u.y.Eval(env)
	case '/':
		return u.x.Eval(env) / u.y.Eval(env)
	}
	panic(fmt.Sprintf("unsupported binary operator: %q", u.op))
}

type call struct {
	fn   string
	args []Expr
}

func (u call) Eval(env Env) float64 {
	switch u.fn {
	case "pow":
		return math.Pow(u.args[0].Eval(env), u.args[1].Eval(env))
	case "sin":
		return math.Sin(u.args[0].Eval(env))
	case "sqrt":
		return math.Sqrt(u.args[0].Eval(env))
	}
	panic(fmt.Sprintln("unsupported binary operator "+  u.fn))
}


func (v Var) Eval(env Env) float64 {
	return env[v]
}

func (l literal) Eval(env Env) float64 {
	return float64(l)
}

func TestEval(t *testing.T) { // 测试用例
	tests := []struct {
		expr string
		env Env
		want string 
	}{
		{"sqrt(A / pi)", Env{"A": 87616, "pi": math.Pi}, "167"},
        {"pow(x, 3) + pow(y, 3)", Env{"x": 12, "y": 1}, "1729"},
        {"pow(x, 3) + pow(y, 3)", Env{"x": 9, "y": 10}, "1729"},
        {"5 / 9 * (F - 32)", Env{"F": -40}, "-40"},
        {"5 / 9 * (F - 32)", Env{"F": 32}, "0"},
        {"5 / 9 * (F - 32)", Env{"F": 212}, "100"},
	}
	var prevExpr string 
	for _, test := range tests {
		if test.expr != prevExpr {
			fmt.Printf("\n%s\n", test.expr)
			prevExpr = test.expr
		}
		expr, err := Parse(test.expr)
		if err != nil {
			t.Error(err)
			continue
		}
		got := fmt.Sprintf("%.6g", expr.Eval(test.env))
		fmt.Printf("\t%v => %s\n", test.env, got)
		if got != test.want {
			t.Errorf("%s, Eval() in %v = %q, want %q\n", test.expr, test.env, got, test.want)
		}
	}

}

```

### 练习7.13 - 7.16
-  量有点大， 先略过
```go

```

### xml解码示例

```go
// Xmlselect prints the text of selected elements of an XML document.
package main

import (
	"encoding/xml"
	"fmt"
	"io"
	"os"
	"strings"
)

func main() {
    dec := xml.NewDecoder(os.Stdin)
    var stack []string // stack of element names
    for {
        tok, err := dec.Token()
        if err == io.EOF {
            break
        } else if err != nil {
            fmt.Fprintf(os.Stderr, "xmlselect: %v\n", err)
            os.Exit(1)
        }
        switch tok := tok.(type) { // 类型分支写法
        case xml.StartElement:
            stack = append(stack, tok.Name.Local) // push
        case xml.EndElement:
            stack = stack[:len(stack)-1] // pop
        case xml.CharData:
            if containsAll(stack, os.Args[1:]) {
                fmt.Printf("%s: %s\n", strings.Join(stack, " "), tok)
            }
        }
    }
}

// containsAll reports whether x contains the elements of y, in order.
func containsAll(x, y []string) bool {
    for len(y) <= len(x) {
        if len(y) == 0 {
            return true
        }
        if x[0] == y[0] {
            y = y[1:]
        }
        x = x[1:]
    }
    return false
}

```