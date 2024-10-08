---
title: Go 语言圣经笔记  第八章：Goroutines和Channels
date: 2024-08-09 15:40:42
tags: [Go, Go语言圣经]
---

# 第八章 Goroutines 和Channels 

## Goroutines 
- 在Go语言中，每一个并发的执行单元叫做一个goroutine。
- 使用关键字go创建新的goroutine
- 下面是一个求斐波那契数列的程序， 
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	go spinner(100 * time.Millisecond)
	const n = 45
	fibN := fib(n)
	fmt.Printf("\rFibonacci(%d) = %d\n", n, fibN)
}

func spinner(delay time.Duration) {
/*
time.Duration
持续时间(Duration)表示两个瞬间之间经过的时间
作为int64纳秒计数。该表示限制了
最大可代表的持续时间约为290年。
*/

	for {
		for _, r := range `-\|/` {
			fmt.Printf("\r%c", r)
			time.Sleep(delay)
		}
	}
}
func fib(x int) int {
	if x < 2 {
		return x
	}
	return fib(x-1) + fib(x-2)
}
```
- 值得注意的是， 当主程序返回或者直接终止时， 所有goroutine 都会终止

## 并发的clock服务

- 首先是顺序的始终服务
```go
package main

import (
	"io"
	"log"
	"net"
	"time"
)

func handleConn(c net.Conn) {
	defer c.Close() // 延迟调用
	for {
		_, err := io.WriteString(c, time.Now().Format("15:04:05\n"))
		if err != nil {
			return 
		}
		time.Sleep(1 * time.Second)
	}
}
func main() {
	listener, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err) // e.g., connection aborted
			continue
		}
		go handleConn(conn)
	}
}
```

- 下面是并发的
```go
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err) // e.g., connection aborted
			continue
		}
		handleConn(conn)
	}
```

### 练习8.1
- 修改clock2来支持传入参数作为端口号，然后写一个clockwall的程序，这个程序可以同时与多个clock服务器通信，从多个服务器中读取时间，并且在一个表格中一次显示所有服务器传回的结果，类似于你在某些办公室里看到的时钟墙。如果你有地理学上分布式的服务器可以用的话，让这些服务器跑在不同的机器上面；或者在同一台机器上跑多个不同的实例，这些实例监听不同的端口，假装自己在不同的时区。
```go

```

### 练习8.2
- 实现一个并发FTP服务器。服务器应该解析客户端发来的一些命令，比如cd命令来切换目录，ls来列出目录内文件，get和send来传输文件，close来关闭连接。你可以用标准的ftp命令来作为客户端，或者也可以自己实现一个。

## 并发的echo服务
- 前面的clock服务器在每一个连接都会运行一个goroutine， 而在本节中我们在每个连接中运行多个goroutine
```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"net"
	"strings"
	"time"
)

func echo(c net.Conn, shout string, delay time.Duration) {
	fmt.Fprintln(c, "\t", strings.ToUpper(shout))
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", shout)
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", strings.ToLower(shout))
}

func handleConn(c net.Conn) {
	input := bufio.NewScanner(c)
	for input.Scan() {
		echo(c, input.Text(), 1*time.Second)
	}
	c.Close()
}
func main() {
	listener, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err) // e.g., connection aborted
			continue
		}
		go handleConn(conn)
	}
}
```

```go
\\并发的netcat
package main

import (
	"io"
	"log"
	"net"
	"os"
)

func main() {
    conn, err := net.Dial("tcp", "localhost:8000")
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    go mustCopy(os.Stdout, conn)
	mustCopy(conn, os.Stdin)
}

func mustCopy(dst io.Writer, src io.Reader) {
    if _, err := io.Copy(dst, src); err != nil {
        log.Fatal(err)
    }
}

```
- 也可以同时并行处理echo

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"net"
	"strings"
	"time"
)

func echo(c net.Conn, shout string, delay time.Duration) {
	fmt.Fprintln(c, "\t", strings.ToUpper(shout))
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", shout)
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", strings.ToLower(shout))
}

func handleConn(c net.Conn) {
	input := bufio.NewScanner(c)
	for input.Scan() {
		go echo(c, input.Text(), 1*time.Second)
	}
	c.Close()
}
func main() {
	listener, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err) // e.g., connection aborted
			continue
		}
		go handleConn(conn)
	}
}
```

## Channels
- Channel是一个通信机制，允许goroutine之间传递数据。
- 使用make 函数创建一个channel
- channel 可以发送和接收， 其中从channel 获取的时候可以不获取接收结果。
- channel 可以关闭， 关闭后还可以从其接收，但是不可以发送。
- Channel 分为带缓存的和不带缓存的， 

### 不带缓存的Channels 
- 发送时会使得发送者的goroutine阻塞， 直到另一个goroutine 在相同的Channel上进行接收操作。
- 反之，如果一个goroutine先进行接收操作， 照样会堵塞
- 当值传送成功之后， 两个goroutine 裁可以进行后面的语句。

```go
package main

import (
	"io"
	"log"
	"net"
	"os"
)

func main() {
    conn, err := net.Dial("tcp", "localhost:8000")
    if err != nil {
        log.Fatal(err)
    }
	done:= make(chan struct{})
	go func(){
		io.Copy(os.Stdout, conn)
		log.Println("done")
		done <- struct{}{} //这里主要起到同步的作用， 如果
	}()
	
    mustCopy(os.Stdout, conn)
	conn.Close() // 这里关闭后可以让用户端 收到关闭通知
	<- done // 如果这里先完成的话， 会先进入堵塞等待状态
}

func mustCopy(dst io.Writer, src io.Reader) {
    if _, err := io.Copy(dst, src); err != nil {
        log.Fatal(err)
    }
```

### 串联的Channels (pipeline)
- Channel可以传递任意类型的值， 以下是个示例， 由三个goroutine 使用两个channel 串联起来， 第一个是计数器，第二个是平方， 第三个是输出

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	naturals := make(chan int)
	squares := make(chan int)

	go func() {
		for x := 0; x < 100; x++ {
			naturals <- x // 每次进去一个数字， 这边就会堵塞
			
			fmt.Printf("make %d\n", x)
		}
	}()

	go func() {
		for {
			x := <-naturals
			squares <- x * x
			fmt.Printf("squares %d\n", x)
		}
	}()
	for {
		fmt.Println(<-squares)
		time.Sleep(time.Second)
	}
}
```
- 我们加上标记后， 可以看到输出的顺序。
- 试图关闭一个关闭的通道会宕机
- 在通知接收方goroutine所有数据都发送完毕的时候可以关闭通道， 关闭通道并不是必须的

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	naturals := make(chan int)
	squares := make(chan int)

	go func() {
		for x := 0; x < 100; x++ {
			naturals <- x // 每次进去一个数字， 这边就会堵塞
			
		//	fmt.Printf("make %d\n", x)
		}
		close(naturals)
	}()

	go func() {
		for {
			x := <-naturals
			squares <- x * x
		}
		close(squares)
	}()
	for {
		fmt.Println(<-squares)
		time.Sleep(time.Second)
	}
}
```
### 单向通道
- 可以定义只输入或者输出的管道， 作用是避免误用

```go
package main

import "fmt"

func counter(out chan<- int) { // 这里是定义了类型， 为单向输入或者输出型的，
	for x := 0; x < 100; x++ {
		out <- x
	}
	close(out)
}

func squarer(out chan<- int, in <-chan int) {
	for v := range in {
		out <- v * v
	}
	close(out)
}

func printer(in <-chan int) {
	for v := range in {
		fmt.Println(v)
	}
}

func main() {
	naturals := make(chan int)
	squares := make(chan int)
	go counter(naturals)
	go squarer(squares, naturals)
	printer(squares)
}
```

### 缓冲通道
- 缓冲通道内部是一个队列， 知道这个就好
- 有一种Bug，叫做goroutines泄露， 这是因为goroutines没有被关闭， 通道一直被堵塞没有被关闭， 导致goroutines一直运行， 直到程序退出

### 并行循环
- 由一些完全独立的子问题组成的问题称为高度并行， 高度并行的问题最容易实现并行。
- 并行循环常见例子， 对于循环中每个运算， 都可以开一个goroutine运行，这样做经常会出现一个错误，就是循环结束后程序直接就结束了， 我们的goroutine当然也就没有执行完成。
- 解决方法是设置一个共享通道， 读出循环个数次（当其中由错误中断的时候可能有问题）
- 书上又给出了一个例子， 使用通道获取出现的第一个错误并返回， 而这样也会堵塞引起goroutine泄露的错误。
- 解决方法给出了两种， 一种是开足够大的缓冲通道， 另一种是返回时建立一个goroutine来读完通道，
- 


### 并发的Web爬虫
- 这里再提一下， 对于匿名函数一般两种方法传参， 一种是后面括号直接跟着， 另一种是赋值给函数变量。然后再调用
```go

package main

import (
	"fmt"
	"os"

	"gopl.io/ch5/links"
)

func crawl(url string) []string {
	fmt.Println(url)
	list, err := links.Extract(url)
	if err != nil {
		fmt.Println(err)
	}
	return list 
}
func main() {
	worklist := make(chan []string)
	go func(){
		worklist <- os.Args[1:]
	}()

	seen := make(map[string]bool)
	for list := range worklist {
		for _, link := range list {
			if !seen[link] {
				seen[link] = true
				go func(link string) {
					worklist <- crawl(link)
				}(link)
			}
		}
	}
}
```
- 该程序会不断爬取相关网页， 书上说会执行若干秒后出现错误，1是出现某网页解析错误， 二是连接数目过多性能不够。 这里两个我都没有遇到， 三是该程序永远不会结束， 这是因为worklist没有关闭， 循环会一直堵塞， 直到程序退出。
- 第一个改进是减少并行goroutine的数量， 采用空闲槽获取令牌来限制并发数量
- 对于第三个的改进是通过计数器来判断worklist是否结束

```go
package main

import (
	"fmt"
	"os"

	"gopl.io/ch5/links"
)
var tokens = make(chan struct{}, 20)

func crawl(url string) []string {
	fmt.Println(url)
	tokens <- struct{}{} // 有槽才能启动
	list, err := links.Extract(url)
	<- tokens // 释放槽
	if err != nil {
		fmt.Println(err)
	}
	return list 
}
func main() {
	worklist := make(chan []string)
	go func(){
		worklist <- os.Args[1:]
	}()

	seen := make(map[string]bool)
	for list := range worklist {
		for _, link := range list {
			if !seen[link] {
				seen[link] = true
				go func(link string) {
					worklist <- crawl(link)
				}(link)
			}
		}
	}
}
```
- 另一种限制并发的思路是使用常驻的crawler goroutine 
```go
package main

import (
	"fmt"
	"os"

	"gopl.io/ch5/links"
)

func crawl(url string) []string {
	fmt.Println(url)
	list, err := links.Extract(url)
	if err != nil {
		fmt.Println(err)
	}
	return list 
}
func main() {
	worklist := make(chan []string)
	unseenLinks := make(chan string)
	go func(){
		worklist <- os.Args[1:]
	}()

	for i := 0; i < 20; i++ {
		go func() {
			for link := range unseenLinks { // 公用通道， for range 可以在通道被关闭的时候正常退出， 
				foundLinks := crawl(link) 
				go func() {worklist <- foundLinks}()
			}
		}()
	}

	seen := make(map[string]bool)
	for list := range worklist {
		for _, link := range list {
			if !seen[link] {
				seen[link] = true
				unseenLinks <- link //
			}
		}
	}
}
```

### select 多路复用
- 一个简单的倒计时发射程序如下

```go
package main

import (
	"fmt"
	"time"
)

func launch(){
	fmt.Println("Lift off!")
}
func main() {
	fmt.Println("Commencing countdown. Press return to abort.")
	tick := time.Tick(1 * time.Second)
	for countdown := 10; countdown > 0; countdown--{
		fmt.Println(countdown)
		<-tick // 等计时器滴答
	}
	launch()
}




```
- 现在我们想在倒计时结束前， 可以允许按下回车键来取消发射
- 朴素来想， 我们现在西药新建一个goroutine 来监听stdin并新设置一个channel 传递信息， 然后在循环中同时监听tick 和新通道
- 这时候可以用select来解决，
- select 会等待case中有能够执行的case 时再去执行， 这时候其他通信不会进行
- 多个程序同时满足时，select会随机选择一个。
- 使用select实现我们上面要求的功能

```go
package main

import (
	"fmt"
	"os"
	"time"
)

func launch(){
	fmt.Println("Lift off!")
}
func main() {
	fmt.Println("Commencing countdown. Press return to abort.")
	abort := make(chan struct{})
	go func(){
		os.Stdin.Read(make([]byte, 1))
		abort <- struct{}{} 
	}()
	tick := time.Tick(1 * time.Second)
	for countdown := 10; countdown > 0; countdown--{
		fmt.Println(countdown)
		select {
		case <-tick:
		case <-abort:
			fmt.Println("Launch aborted!")
			return
		}
	}
	launch()
}




```

### 示例: 并发的目录遍历
- 朴素来说是这样
```go
package main

import (
	"flag"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
)

func walkDir(dir string, fileSize chan<- int64) {
	for _, entry := range dirents(dir) {
		if entry.IsDir() {
			subdir := filepath.Join(dir, entry.Name())
			walkDir(subdir, fileSize)
		} else {
			fileSize <- entry.Size() // 增加
		}
	}
}

func dirents(dir string) []os.FileInfo {
	entries, err := ioutil.ReadDir(dir)
	if err != nil {
		fmt.Fprintf(os.Stderr, "du1: %v\n", err)
	}
	return entries 
}
func main() {
	flag.Parse()
	roots := flag.Args()
	if len(roots) == 0{
		roots = []string{"."}
	}
	fileSizes := make(chan int64)
	go func(){
		for _, root := range roots {
			walkDir(root, fileSizes)
		}
		close(fileSizes)
	}()

	var nfiles, nbytes int64
    for size := range fileSizes {
        nfiles++
        nbytes += size
    }
    printDiskUsage(nfiles, nbytes)
}

func printDiskUsage(nfiles, nbytes int64) {
	fmt.Printf("%d files %.1f GB\n", nfiles, float64(nbytes) / 1e9)
}
```

- 会经历很久的运行时间，然后输出信息， 我们希望中途输出更多相关信息， 但不希望会直接输出一大坨
- 使用上一章的select 进行改造

```go

var verbose = flag.Bool("v", false, "show progress")

func main() {
	flag.Parse()
	roots := flag.Args()
	if len(roots) == 0{
		roots = []string{"."}
	}
	fileSizes := make(chan int64)
	go func(){
		for _, root := range roots {
			walkDir(root, fileSizes)
		}
		close(fileSizes)
	}()
	var tick <-chan time.Time 
	if *verbose {
		tick = time.Tick(10 * time.Millisecond)
	}
	var nfiles, nbytes int64
loop:
    for  {
        select {
		case size, ok := <-fileSizes:
			if !ok {
				break loop //该处为关闭后返回的信息。loop 可以goto 和break
			}
			nfiles++
			nbytes += size 
		case <-tick:
			printDiskUsage(nfiles, nbytes)
		}
    }
    printDiskUsage(nfiles, nbytes)
}

```
- 然后对遍历操作进行并发改造
- 速度明显变快了
```go
package main

import (
	"flag"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"sync"
	"time"
)

func walkDir(dir string, n *sync.WaitGroup, fileSize chan<- int64) {
	defer n.Done() // 正好每个都能减到
	for _, entry := range dirents(dir) {
		if entry.IsDir() {
			n.Add(1)
			subdir := filepath.Join(dir, entry.Name())
			go walkDir(subdir, n, fileSize)
		} else {
			fileSize <- entry.Size() // 传输到主逻辑增加。
		}
	}
}

func dirents(dir string) []os.FileInfo {
	entries, err := ioutil.ReadDir(dir)
	if err != nil {
		fmt.Fprintf(os.Stderr, "du1: %v\n", err)
	}
	return entries 
}

var verbose = flag.Bool("v", false, "show progress")

func main() {
	flag.Parse()
	roots := flag.Args()
	if len(roots) == 0{
		roots = []string{"."}
	}
	fileSizes := make(chan int64)
	var n sync.WaitGroup
	
	for _, root := range roots {
		n.Add(1) // 组的数目
		go walkDir(root, &n,  fileSizes)
	}
	go func(){
		n.Wait()
		close(fileSizes)
	}()


	var tick <-chan time.Time 
	if *verbose {
		tick = time.Tick(10 * time.Millisecond)
	}
	var nfiles, nbytes int64
loop:
    for  {
        select {
		case size, ok := <-fileSizes:
			if !ok {
				break loop //该处为关闭后返回的信息。loop 可以 goto 和 break 棘啊
			}
			nfiles++
			nbytes += size 
		case <-tick:
			printDiskUsage(nfiles, nbytes)
		}
    }
    printDiskUsage(nfiles, nbytes)
}

func printDiskUsage(nfiles, nbytes int64) {
	fmt.Printf("%d files %.1f GB\n", nfiles, float64(nbytes) / 1e9)
}
```

### 并发的退出
- 使用通道来广播退出信息， 方法是平常不往里面传值， 终止时把通道关闭， 就能从通道获取零值 获取已退出信息
- 要注意后台goroutine 针对信息快速终止

### 示例 聊天服务
```go
package main

import (
	"bufio"
	"log"
	"net"
)

func main() {
	listener, err := net.Listen("tcp", "localhost:8080")
	if err != nil {
		log.Fatal(err)
	}
	go broadcaster()
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err)
			continue
		}
		go handleConn(conn)
	}
}

type client chan<- string 

var (
	entering = make(chan client)
	leaving = make(chan client)
	messages = make(chan string)
)
/*
broadcaster监听来自全局的客户端,到来离开信息， 并且接收消息并且广播到所有客户端
*/
func broadcaster() {
	clients := make(map[client]bool) 
	for {
		select {
		case msg := <-messages:
			for cli := range clients{
				cli <- msg
			}
		case cli := <-entering:
			clients[cli] = true 
		
		case cli := <-leaving:
			delete(clients, cli)
			close(cli) // 注意 删除了不代表关闭了， 关闭了才算退出
		}
	}
}

func handleConn(conn net.Conn) {
	ch := make(chan string)
	go clientWriter(conn, ch)
	who := conn.RemoteAddr().String()
	ch <- "You are " + who
	messages <- who + " has arrived"
	entering <- ch
	input := bufio.NewScanner(conn)
	for input.Scan() {
		messages <- who + ": " + input.Text()
	}
	leaving <- ch
	messages <- who + " has left"
	conn.Close()
}

func clientWriter(conn net.Conn, ch <-chan string ){
	for msg := range ch {
		fmt.Fprintf(conn, msg)
	}
}
```
