---
title: Go 语言圣经笔记  第五章：函数
date: 2024-08-09 14:53:50
tags: 
 - Go
 - Go语言圣经
---


# 第五章
- 函数的四种写法——你懂么?
```go
func add(x int, y int) int {return x + y}
func sub(x, y int) int {return x - y})
func first(x int, _ int) int {return x}
func zero(int, int ) int {return 0}


```

- 函数的类型被称为函数的签名， 由两个部分，参数列表和返回值列表决定。
```go
func Sin(x float64) float // 该函数没有函数体， 为函数声明，表示功能不是由GO实现的， 定义了函数签名（可能是汇编语言）
```
- 遍历dom树的递归函数
```go
// 遍历dom树查找herf
// 遍历dom树查找herf
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)

func main(){
	//fmt.Println(fetch())
	doc, err := html.Parse(os.Stdin)  // html.Parse 输入是io.Reader 常见来源有 os.Open, strings.NewReader, http.Request.body, bytes.Buffer
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	for _, link := range visit(nil, doc) { // 
		fmt.Println(link)
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
- 输出整个dom树结构，

### 练习5.1
- 修改findlinks代码中遍历n.FirstChild链表的部分，将循环调用visit，改成递归调用。
```go
// 递归子节点和兄弟节点

package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)

func main(){
	doc, err := html.Parse(os.Stdin)  
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	for _, link := range visit(nil, doc) { // 
		fmt.Println(link)
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
	if n.FirstChild != nil {
		links = visit(links, n.FirstChild)
	}
	if n.NextSibling != nil {
		links = visit(links, n.NextSibling)
	}
	return links
}


```

### 练习5.2
-  编写函数，记录在HTML树中出现的同名元素的次数。
  
```go
// 编写函数，记录在HTML树中出现的同名元素的次数。
// 同名元素是指 Node.Data 相同的， 使用map统计即可
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)


func main(){
	//fmt.Println(fetch())
	doc, err := html.Parse(os.Stdin)  // html.Parse 输入是io.Reader 常见来源有 os.Open, strings.NewReader, http.Request.body, bytes.Buffer
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	for name, total := range count(make(map[string]int), doc) { // 
		fmt.Printf("%s, %d\n", name, total)
	}
}

func count(m map[string]int, n *html.Node) map[string]int{
	if n.Type == html.ElementNode{
		m[n.Data]++
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling { 
		count(m, c)
	}
	return m
}


```

### 练习5.3
-  编写函数输出所有text结点的内容。注意不要访问`<script>`和`<style>`元素，因为这些元素对浏览者是不可见的。
  
```go
// text节点判断方法—— 为html.text Node, 输出非英文有的是乱码
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)

func main(){
	//fmt.Println(fetch())
	doc, err := html.Parse(os.Stdin)  // html.Parse 输入是io.Reader 常见来源有 os.Open, strings.NewReader, http.Request.body, bytes.Buffer
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	for _, link := range visit(nil, doc) { // 
		fmt.Println(link)
	}
}

func visit(links []string, n *html.Node) []string {
	if n.Type == html.TextNode  {
		fmt.Println(n.Data)
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling { 
		if c.Data == "style" || c.Data == "script" {
			continue
		}
		links = visit(links, c) 
	}
	return links
}

```

### 练习5.4
-  扩展visit函数，使其能够处理其他类型的结点，如images、scripts和style sheets。
-  
```go

// 扩展visit函数，使其能够处理其他类型的结点，如images、scripts和style sheets。
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)

func main(){
	doc, err := html.Parse(os.Stdin)
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	for _, link := range visit(nil, doc) { // 
		fmt.Println(link)
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
	if n.Type == html.ElementNode && (n.Data == "img" || n.Data == "script") {
		for _, a := range n.Attr {
			if a.Key == "src" {
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


## 多返回值
- 多返回值可以做返回值
- 多返回值函数可以做参数， 下面两种写法等同

```go
log.Println(findLinks(url))
```

```go
links, err := findLinks(url)
log.Println(links, err)
```

- 新版本findlinks

```go
// 遍历dom树查找herf
package main

import (
	"fmt"
	"net/http"
	"os"

	"golang.org/x/net/html"
)

func main(){
	for _, url := range os.Args[1:] {
		links, err := findLinks(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "findlinks2: %v\n", err)
			continue
		}
		for _, link := range links {
			fmt.Println(link)
		}
	}
}

func findLinks(url string) ([]string, error) {
	resp, err := http.Get(url)
	if err != nil {
		return nil, err 
	}
	if resp.StatusCode != http.StatusOK {
		resp.Body.Close()
		return nil, fmt.Errorf("getting %s: %s", url, resp.Status)
	}
	doc, err := html.Parse(resp.Body)
	resp.Body.Close() // Go的垃圾回收不包括
	if err != nil {
		return nil, err
	}
	return visit(nil, doc), nil // 返回值有好几种， 
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

- 如果一个函数的所有返回值都有显示的变量名， 那么该函数的return语句可以省略变量名。 bare return

## 练习5.6
-  修改gopl.io/ch3/surface（§3.2）中的corner函数，将返回值命名，并使用bare return。
-  

```go
func corner(i, j int) (sx float64,sy float64) {
	x := xyrange * (float64(i) / cells - 0.5)
	y := xyrange * (float64(j) / cells - 0.5)
	z := f(x, y)
	sx = width / 2 + (x - y) * cos30 * xyscale
	sy = height / 2 + (x + y) * sin30 * xyscale - z * zscale
	return 
}

```

## 错误处理
- 所有错误在本层分层时， 都需要添加本层的前缀， 错误信息
- 错误一般分为五种
- 传播错误， 错误会使得整个功能失败。 整个错误返回给调用者
- 错误时偶然性，不可预知的问题产生。 重试时， 我们需要限制重试的时间间隔或者重试的次数， 避免无限制的重试 （例子：下面的wait函数）
- 错误时整个程序无法运行、需要输出错误并且结束程序， 只应该在main中执行。 
- 错误时， 只需要输出错误， 不需要结束程序。  log.printf("message", error)
- 直接忽略掉错误。
- 文件结尾错误一般不需要报错
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func main() {

}

func WaitForServer(url string) error {
	const timeout = 1 * time.Minute
	deadline := time.Now().Add(timeout)
	for tries := 0; time.Now().Before(deadline); tries++ {
		_, err := http.Head(url)
		if err == nil {
			return nil
		}
		log.Printf("server not responding (%s); retrying...", err)
		time.Sleep(time.Second << uint(tries)) 
	}
	return fmt.Errorf("server %s failed to respond after %s", url, timeout)
}
```

## 函数变量
- Go语言中，函数也是值， 可以赋值给变量， 函数变量可以作为参数传递给其他函数
- 例子

```go
func square(n int) int { return n * n }
func negative(n int) int { return -n }
func product(m, n int) int { return m * n }
f := square
fmt.Println(f(3))
f = negative
fmt.Println(f(3))
fmt.Printf("%T\n", f))
f = product // 这里会报错， 因为两种函数不是同一类型（类型由参数列表和返回值列表决定）
```

- %*s 会在字符串之前填充一些空格，后面写数目

- 利用函数变量重写outline如下

```go
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)
var depth int 
func main(){
	//fmt.Println(fetch())
	doc, err := html.Parse(os.Stdin) 
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	forEachNode(doc, startElement, ElementNode)
}


func forEachNode(n *html.Node, pre, post func(n *html.Node)) {
	if pre != nil {
		pre(n)
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		forEachNode(c, pre, post)
	}
	if post != nil {
		post(n)
	}
}

func startElement(n *html.Node) {
	if n.Type == html.ElementNode {
		fmt.Printf("%*s<%s>\n", depth * 2, "", n.Data)
		depth++
	}
}

func ElementNode(n *html.Node) {
	if n.Type == html.ElementNode {
		depth--
		fmt.Printf("%*s<%s>\n", depth * 2, "", n.Data)
	}
}
```


### 练习5.7
-  完善startElement和endElement函数，使其成为通用的HTML输出器。要求：输出注释结点，文本结点以及每个元素的属性（< a href='...'>）。使用简略格式输出没有孩子结点的元素（即用<img/>代替<img></img>）。编写测试，验证程序输出的格式正确。（详见11章）

```go
// 在前括号信息内增加所需信息， 并且通过有无子节点判断是否增添后括号
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)
var depth int 
func main(){
	//fmt.Println(fetch())
	doc, err := html.Parse(os.Stdin) 
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	forEachNode(doc, startElement, ElementNode, leaveNode)
}


func forEachNode(n *html.Node, pre, post, now func(n *html.Node)) {
	if n.FirstChild == nil {
		now(n)
		return 
	}
	if pre != nil {
		pre(n)
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		forEachNode(c, pre, post, now)
	}
	if post != nil && n.FirstChild != nil{
		post(n)
	}
}

func startElement(n *html.Node) {
	if n.Type == html.ElementNode {
		fmt.Printf("%*s<%s ", depth * 2, "", n.Data)
		for _, a := range n.Attr {
			fmt.Printf("%s=%s ", a.Key, a.Val)
		}
		depth++
		fmt.Printf(">\n")
	}
}

func ElementNode(n *html.Node) {
	if n.Type == html.ElementNode {
		depth--
		fmt.Printf("%*s</%s>\n", depth * 2, "", n.Data)
	}
}

func leaveNode(n *html.Node) {
	if n.Type == html.ElementNode {
		fmt.Printf("%*s<%s/ ", (depth + 1) * 2, "", n.Data)
		for _, a := range n.Attr {
			fmt.Printf("%s=%s ", a.Key, a.Val)
		}
		fmt.Printf(">\n")
	}
}
```

### 练习5.8
- 修改pre和post函数，使其返回布尔类型的返回值。返回false时，中止forEachNoded的遍历。使用修改后的代码编写ElementByID函数，根据用户输入的id查找第一个拥有该id元素的HTML元素，查找成功后，停止遍历。

```go
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)
var depth int 
func main(){
	//fmt.Println(fetch())
	doc, err := html.Parse(os.Stdin) 
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	_, ok := ElementByID(doc, "lg", startElement, ElementNode)
	if !ok {
		fmt.Println("not found")
	}

}

func ElementByID(n *html.Node, id string,  pre, post func(n *html.Node,id string) bool) (*html.Node, bool) {
	if pre != nil {
	    ok := pre(n, id)
		if ok {
			return n, ok
		}
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		ans1, ok := ElementByID(c, id, pre, post)
		if ok {
			return ans1, ok
		}
	}
	if post != nil {
		post(n, id)
	}
	return nil, false
}


func startElement(n *html.Node, id string) bool{
	if n.Type == html.ElementNode {
		fmt.Printf("%*s<%s>\n", depth * 2, "", n.Data)
		depth++
	}
	for _, a := range n.Attr {
		if a.Key == "id" && a.Val == id{
			fmt.Printf("%*s found here\n", (depth + 1) *2, "")
			return true
		}
	}
	return false
}

func ElementNode(n *html.Node, id string) bool{
	if n.Type == html.ElementNode {
		depth--
		fmt.Printf("%*s</%s>\n", depth * 2, "", n.Data)
	}
	return false
}

```

### 练习5.9
- 编写函数expand，将s中的"foo"替换为f("foo")的返回值。
```go
// 编写函数expand，将s中的"foo"替换为f("foo")的返回值

package main

import (
	"fmt"
	"strings"
)

func main() {

	s := "fooofffofofooffooofofofofofo"
	fmt.Printf("%s\n%s\n", s, expand(s, f))
}

func expand(s string, f func(string) string) string {
	newS := f("foo")
	return strings.Replace(s, "foo", newS, -1)
}

func f(s string) string {
	return "?" + s + "?"
}
```

## 匿名函数
- 命名函数只能在包级别进行声明， 匿名函数函数后面没有名称， 能够获取到整个词法环境
- 示例

```go
package main

func main() {
	f := squares()
	for i := 0; i < 10; i++ {
		println(f())
	}
}

func squares() func() int {
	var x int
	return func() int {
		x++
		return x * x
	}
}
```
- 拓扑排序

```go
package main

import (
	"fmt"
	"sort"
)

var prereqs = map[string][]string{
	"algorithms": {"data structures"},
	"calculus":   {"linear algebra"},
	"compilers": {
		"data structures",
		"formal languages",
		"computer organization",
	},
	"data structures":       {"discrete math"},
	"databases":             {"data structures"},
	"discrete math":         {"intro to programming"},
	"formal languages":      {"discrete math"},
	"networks":              {"operating systems"},
	"operating systems":     {"data structures", "computer organization"},
	"programming languages": {"data structures", "computer organization"},
}

func main() {
	for i, course := range topoSort(prereqs) {
		fmt.Printf("%d:\t%s\n", i+1, course)
	}
}

// topoSort 实现了对有向无环图（DAG）的节点进行拓扑排序。
// m 是一个映射，其中每个键代表图中的一个节点，对应的值是该节点指向的其他节点的列表。
// 返回值是节点的拓扑排序列表。
func topoSort(m map[string][]string) []string {
    // order 用于存储拓扑排序的结果。
    var order []string
    // seen 用于记录已经访问过的节点，以避免重复访问。
    seen := make(map[string]bool)

    // visitAll 是一个递归函数，用于遍历节点并将其按拓扑顺序添加到 order 中。
    var visitAll func(items []string)
    visitAll = func(items []string) {
        for _, item := range items {
            // 如果节点尚未被访问，则递归访问其依赖项，并将该节点添加到排序顺序中。
            if !seen[item] {
                seen[item] = true
                visitAll(m[item])
                order = append(order, item)
            }
        }
    }

    // keys 用于存储 m 中所有键（节点）的列表，以便进行排序。
    var keys []string
    for key := range m {
        keys = append(keys, key)
    }

    // 对节点进行排序，以便按照一定的顺序访问它们。
    sort.Strings(keys)

    // 使用排序后的节点列表调用 visitAll，以确保节点的处理顺序符合排序结果。
    visitAll(keys)  // 这里是访问入口， 从这里开始拓扑排序

    // 返回拓扑排序结果。
    return order
}
```

### 练习5.10
- 重写topoSort函数，用map代替切片并移除对key的排序代码。验证结果的正确性（结果不唯一）。

```go
// 重写topoSort函数，用map代替切片并移除对key的排序代码。验证结果的正确性（结果不唯一）。

package main

import (
	"fmt"
)

var prereqs = map[string][]string{
	"algorithms": {"data structures"},
	"calculus":   {"linear algebra"},
	"compilers": {
		"data structures",
		"formal languages",
		"computer organization",
	},
	"data structures":       {"discrete math"},
	"databases":             {"data structures"},
	"discrete math":         {"intro to programming"},
	"formal languages":      {"discrete math"},
	"networks":              {"operating systems"},
	"operating systems":     {"data structures", "computer organization"},
	"programming languages": {"data structures", "computer organization"},
}

func main() {
	for i, course := range topoSort(prereqs) {
		fmt.Printf("%d:\t%s\n", i+1, course)
	}
}
func topoSort(m map[string][]string) []string {
    var order []string
    seen := make(map[string]bool)

    var visitAll func(m map[string][]string, s string)
    visitAll = func(m map[string][]string, s string) {
		if seen[s] {  // 多入口， 把判断改到循环开始
			return
		}
		seen[s] = true
		order = append(order, s)
		items := m[s]
        for _, item := range items {
			//fmt.Println(s + "!" + item + "!")
            visitAll(m , item)
        }
    }
    for key := range m {
		visitAll(m, key)
    }
    return order
}

```
### 练习5.11
- 现在线性代数的老师把微积分设为了前置课程。完善topSort，使其能检测有向图中的环。

```go
// 使用带度数的拓扑排序，最后要是有度数不为零的，就是环上的。

package main

import (
	"fmt"
	"sort"
)

var prereqs = map[string][]string{
	"algorithms": {"data structures"},
	"calculus":   {"linear algebra"},
	"compilers": {
		"data structures",
		"formal languages",
		"computer organization",
	},
	"data structures":       {"discrete math"},
	"databases":             {"data structures"},
	"discrete math":         {"intro to programming"},
	"formal languages":      {"discrete math"},
	"networks":              {"operating systems"},
	"operating systems":     {"data structures", "computer organization"},
	"programming languages": {"data structures", "computer organization"},
}

func main() {
	for i, course := range topoSort(prereqs) {
		fmt.Printf("%d:\t%s\n", i+1, course)
	}
}
func topoSort(m map[string][]string) []string {
	du := make(map[string]int)
	for _, v := range m {
		for _, tmp := range v{
			du[tmp] ++
		}
	}

    var order []string
    var visitAll func(now string)
    visitAll = func(now string) {
		order = append(order, now)
        items := m[now]
		for _, item := range items{
            du[item] --
            if du[item] == 0 {
			//	fmt.Println("?")
				visitAll(item)
			}
        }
    }
    var keys []string
    for key := range m {
        if du[key] == 0 {
			keys = append(keys, key)
		//	fmt.Println(key + "!")
		}
    }
    sort.Strings(keys)
    for _, key := range keys {
        visitAll(key)
    }
	for _, v := range du {
		if v != 0 {
			panic("有环")
		}
	}
    return order
}
```

### 练习5.12
- gopl.io/ch5/outline2（5.5节）的startElement和endElement共用了全局变量depth，将它们修改为匿名函数，使其共享outline中的局部变量。

```go

// 将outline2 中的startElement和endElement改成匿名函数

package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)
func main(){
	//fmt.Println(fetch())
	doc, err := html.Parse(os.Stdin) 
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlink: %v\n", err)
		os.Exit(1)
	}
	var depth int
	forEachNode(doc, func(n *html.Node){
		if n.Type == html.ElementNode {
			fmt.Printf("%*s<%s>\n", depth * 2, "", n.Data)
			depth++
		}
	}, func(n *html.Node){
		if n.Type == html.ElementNode {
			depth--
			fmt.Printf("%*s</%s>\n", depth * 2, "", n.Data)
		}
	})
}


func forEachNode(n *html.Node, pre, post func(n *html.Node)) {
	if pre != nil {
		pre(n)
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		forEachNode(c, pre, post)
	}
	if post != nil {
		post(n)
	}
}

```

### 练习5.13
- 修改crawl，使其能保存发现的页面，必要时，可以创建目录来保存这些页面。只保存来自原始域名下的页面。假设初始页面在golang.org下，就不要保存vimeo.com下的页面。
- 暂时略过

### 练习5.14
- 使用breadthFirst遍历其他数据结构。比如，topoSort例子中的课程依赖关系（有向图）、个人计算机的文件层次结构（树）；你所在城市的公交或地铁线路（无向图）。
- 暂时略过

- 作用域问题， 由于匿名函数使用的变量常常是传址


## 可变参数
- 在声明可变参数函数时， 需要在参数列表的最后一个参数类型前面加上省略号...
- 例子

```go
package main

import "fmt"

func main() {
	fmt.Println(sum(1, 2, 3, 4, 5))
	fmt.Println(sum())
	fmt.Println(sum(1))
}

func sum(vals ...int) int {
	total := 0
	for _, val := range vals {
		total += val
	}
	return total
}
```

### 练习5.15
-  编写类似sum的可变参数函数max和min。考虑不传参时，max和min该如何处理，再编写至少接收1个参数的版本。
```go
// 编写类似sum的可变参数函数max和min。考虑不传参时，max和min该如何处理，再编写至少接收1个参数的版本。
package main

import "fmt"


func main(){
	fmt.Println(max(1,2,3,4,5,6,7,8,9,10))
	fmt.Println(min())
}

func max(x ...int) int {
	if len(x) == 0 {
		return 0
	}
	max := x[0]
	for _, v := range x {
		if v > max {
			max = v
		}
	}
	return max
}

func min(x ...int) int {
	if len(x) == 0 {
		return 0
	}
	min := x[0]
	for _, v := range x {
		if v < min {
			min = v
		}
	}
	return min
}

```

### 练习5.16
- 编写多参数版本的strings.Join。

```go
// 将 多个字符串数组拼接成一个字符串
package main

import (
	"fmt"
)
func main() {
	strs := []string{"a", "b", "c"}
	strs2 := []string{"d", "e", "f"}
	fmt.Println(myJoin(","))
	fmt.Println(myJoin("?", strs))
	fmt.Println(myJoin("?", strs, strs))
	fmt.Println(myJoin("?", strs, strs2, strs))

}

func myJoin(s string, elems ...[]string) string {
	if len(elems) == 0 {
		return ""
	}
	var str string
	for _, elem := range elems {
		for _, e := range elem {
			str += e + s
		}
	}
	return str[:len(str)-len(s)]
}
```

### 练习5.17
- 编写多参数版本的ElementsByTagName，函数接收一个HTML结点树以及任意数量的标签名，返回与这些标签名匹配的所有元素。下面给出了2个例子：

```go
package main

import (
	"fmt"
	"net/http"
	"os"

	"golang.org/x/net/html"
)

func main() {
	for _, url := range os.Args[1:] {
		resp, err := http.Get(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
			continue
		}
		if resp.StatusCode != http.StatusOK {
			resp.Body.Close()
			fmt.Fprintf(os.Stderr, "fetch: %s: %v\n", url, resp.Status)
			continue 
		}
		doc, err := html.Parse(resp.Body)
		resp.Body.Close() 
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: parsing %s: %v\n", url, err)
			continue
		}
		images := ElementsByTagName(doc, "img")
		headings := ElementsByTagName(doc, "h1", "h2", "h3", "h4")
		fmt.Println(images)
		fmt.Println(headings)
	}
}

func ElementsByTagName(doc *html.Node, name ...string) []*html.Node {
	return visit(nil, doc, name)
}

func visit(links []*html.Node, n *html.Node, v []string) []*html.Node{
	if n.Type == html.ElementNode {
		for _, a := range v {
			if n.Data == a {
				links = append(links,  n)
				return links
			}
		}
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling { 
		links = visit(links, c, v) // 递归调用， 
	}
	return links
}

```

## Deffered 函数
- 一般判断出错的方法如下
```go
package main

import (
	"fmt"
	"net/http"
	"strings"

	"golang.org/x/net/html"
)

func main() {

}
func title(url string) error {
	resp, err := http.Get(url)
	if err != nil {
		return err
	}
	// Check Content-Type is HTML (e.g., "text/html;charset=utf-8").
	ct := resp.Header.Get("Content-Type")
	if ct != "text/html" && !strings.HasPrefix(ct, "text/html;") { 
		resp.Body.Close() // 多次调用关闭， 确保各种情况都会正常退出， 但是很麻烦
		return fmt.Errorf("%s has type %s, not text/html", url, ct)
	}
	doc, err := html.Parse(resp.Body)
	resp.Body.Close()
	if err != nil {
		return fmt.Errorf("parsing %s as HTML: %v", url, err)
	}
	visitNode := func(n *html.Node) {
		if n.Type == html.ElementNode && n.Data == "title" && n.FirstChild != nil {
			fmt.Println(n.FirstChild.Data)
		}
	}
	forEachNode(doc, visitNode, nil)
	return nil
}


func forEachNode(n *html.Node, pre, post func(n *html.Node)) {
	if pre != nil {
		pre(n)
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		forEachNode(c, pre, post)
	}
	if post != nil {
		post(n)
	}
}

```
- 可以使用defer函数， 在调用普通函数或者方法的前面加上defer ， 当执行到该条语句时， 函数和参数表达式得到计算， 但函数并不执行。当函数返回时， 函数和参数表达式被执行。
- 执行顺序和声明顺序相反
- 在一些复杂的情况下， 可以使用defer函数， 确保函数在程序退出时执行。
- 示例
```go
package main

import (
	"fmt"
	"net/http"
	"strings"

	"golang.org/x/net/html"
)

func main() {

}
func title(url string) error {
	resp, err := http.Get(url)
	if err != nil {
		return err
	}
	defer resp.Body.Close() // 
	ct := resp.Header.Get("Content-Type")
	if ct != "text/html" && !strings.HasPrefix(ct, "text/html;") { 
		return fmt.Errorf("%s has type %s, not text/html", url, ct)
	}
	doc, err := html.Parse(resp.Body)
	if err != nil {
		return fmt.Errorf("parsing %s as HTML: %v", url, err)
	}
	visitNode := func(n *html.Node) {
		if n.Type == html.ElementNode && n.Data == "title" && n.FirstChild != nil {
			fmt.Println(n.FirstChild.Data)
		}
	}
	forEachNode(doc, visitNode, nil)
	return nil
}


func forEachNode(n *html.Node, pre, post func(n *html.Node)) {
	if pre != nil {
		pre(n)
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		forEachNode(c, pre, post)
	}
	if post != nil {
		post(n)
	}
}

```

- defer 也可以用来记录何时进入和退出函数

```go
package main

import (
	"log"
	"time"
)

func main() {
	bigSlowOperation()
}

func bigSlowOperation() {
	defer trace("bigSlowOperation")() // 如果不加括号的的话， 则表示在退出时调用trace

	time.Sleep(10 * time.Second)
}

func trace(msg string) func() {
	start := time.Now()
	log.Printf("enter %s", msg)
	return func() {
		log.Printf("exit %s (%s)", msg, time.Since(start))
	}
}
```
- 注意defer函数是在函数结束后才执行， 而不是其他代码域
- 使用defer 改进fetch ， 将http相应信息写入本地文件而不是标准输出流 
```go

// fetch 改进版 将http相应信息写入本地文件而不是标准输出流

package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"path"
)
// 
func main() {
	for _, url := range os.Args[1:] {
		local, n, err := fetch(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
			return 
		}
		fmt.Printf("%s %s %d\n", url, local, n)
	}
}

func fetch(url string) (filename string, n int64, err error) {
	resp, err := http.Get(url)
    if err != nil {
        return "", 0, err
    }
    defer resp.Body.Close() // 延迟关闭
    local := path.Base(resp.Request.URL.Path)
    if local == "/" {
        local = "index.html"
    }
    f, err := os.Create(local)
    if err != nil {
        return "", 0, err
    }
    n, err = io.Copy(f, resp.Body)
    if closeErr := f.Close(); err == nil {
        err = closeErr
    }
    return local, n, err
}
```


### 练习5.18
- 不修改fetch的行为，重写fetch函数，要求使用defer机制关闭文件。

```go
//  不修改fetch的行为，重写fetch函数，要求使用defer机制关闭文件。

package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"path"
)

func main() {
	for _, url := range os.Args[1:] {
		local, n, err := fetch(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
			return 
		}
		fmt.Printf("%s %s %d\n", url, local, n)
	}
}

func fetch(url string) (filename string, n int64, err error) {
	resp, err := http.Get(url)
    if err != nil {
        return "", 0, err
    }
    defer resp.Body.Close() // 延迟关闭
    local := path.Base(resp.Request.URL.Path)
    if local == "/" {
        local = "index.html"
    }
    f, err := os.Create(local)
    if err != nil {
        return "", 0, err
    }
    n, err = io.Copy(f, resp.Body)
	defer func() {   // defer 执行顺序在return 之后， 但是在返回值赋值给调用方之前
		// 为什么defer能调用返回值，因为这里返回值是有名的， defer 函数只能访问有名返回值
		if closeErr := f.Close(); err == nil {
			err = closeErr
		} 
	}()
    return local, n, err
}
```

## panic 异常 （宕机）
- 一般来说， panic异常是只能在运行时才能检查到的错误， 比如说数组访问越界， 空指针引用， 当panic发生时， 程序中断运行， 并立即执行在goroutine中被延迟的函数， 然后崩溃输出日志信息
-  一般来说， 不应用 panic 检查哪些运行时会检查的信息。而且只有比较严重的错误才应用panic
-  为了方便诊断问题，runtime包允许程序员输出堆栈信息。在下面的例子中，我们通过在main函数中延迟调用printStack输出堆栈信息。
-  
- regexp.MustCompile 可以用来检查输入的合法性
```go
package main

import (
	"fmt"
	"os"
	"runtime"
)

func main() {
    defer printStack()
    f(3)
}
func printStack() {
    var buf [4096]byte
    n := runtime.Stack(buf[:], false)
    os.Stdout.Write(buf[:n])
}
func f(x int) {
	fmt.Printf("f(%d)\n", x+0/x) 
	defer fmt.Printf("defer %d\n", x)
	f(x - 1)
}
/*
输出第一部分
f(3)
f(2)
f(1)
defer 1
defer 2
defer 3  // 发生异常、 之前延迟的defer先被调用， 然后再触发panic
panic: runtime error: integer divide by zero


printStack() 的输出为
goroutine 1 [running]:
main.printStack()
        D:/bq/Go_learning_tools/learning/defer2/defer2.go:15 +0x2e
panic({0xb27a80?, 0xbcb9f0?})
        C:/Program Files/Go/src/runtime/panic.go:770 +0x132
main.f(0xb5b098?)
        D:/bq/Go_learning_tools/learning/defer2/defer2.go:19 +0x118
main.f(0x1)
        D:/bq/Go_learning_tools/learning/defer2/defer2.go:21 +0xfe
main.f(0x2)
        D:/bq/Go_learning_tools/learning/defer2/defer2.go:21 +0xfe
main.f(0x3)
        D:/bq/Go_learning_tools/learning/defer2/defer2.go:21 +0xfe
main.main()
        D:/bq/Go_learning_tools/learning/defer2/defer2.go:11 +0x35
*/
```

## Recover 捕获异常
- recover 函数用来捕获panic异常， 如果没有panic异常， recover返回nil， 如果有panic异常， recover返回panic的值， 并且恢复panic， 恢复后程序继续运行
- recover函数只能在defer函数中调用， 否则会panic

### 练习5.19
- 使用panic和recover编写一个不包含return语句但能返回一个非零值的函数。

```go
// 练习5.19   使用panic和recover编写一个不包含return语句但能返回一个非零值的函数。
package main

import "fmt"

func main() {
	fmt.Println(f())
}

func f() (res int) {
	defer func() {
		if err := recover(); err != nil {
			res = 1
		}
	}()
	panic("panic")
}
```
