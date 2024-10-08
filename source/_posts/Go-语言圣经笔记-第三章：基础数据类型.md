---
title: Go 语言圣经笔记  第三章：基础数据类型
date: 2024-08-02 11:46:12
tags: [Go, Go语言圣经]
---

# 第三章

- Go语言数据类型分为四类： 基础类型、复合类型、引用类型和接口类型
- 基础类型： 数字、 字符串、 bool型、 
- 复合类型：数组、结构体、
- 引用类型： 指针、 切片、 字典、 函数、 通道
- 接口类型： 第七章

## 数据类型
- Go 在运算时要求比较严格，只允许相同类型的进行运算
- 整型分为 有符号和无符号， 每种都分为 8，16，32，64位
- 还有一种对应CPU平台的类型， int和 uint
- 还用一种无符号的整数类型uintptr, 没有具体的bit大小但是足以容纳指针
- 浮点数转整数的方式是丢弃小数部分， 然后向数轴方向折断
- 浮点数只有两种， float32 和float64
- Nan 的比较总是不成立， 但是！= 会成立
- 浮点数输出可以有%e 科学计数法， %f 小数点， 两种方法， 使用%g可以自动生成

## 运算符

- 基本上和c++ 相同
- &^  为位清空操作



## 实例

```go
package main

import (
	"fmt"
	"math"
)

const (
	width, height = 600, 320
	cells         = 100
	xyrange       = 30.0
	xyscale       = width / 2 / xyrange
	zscale        = height * 0.4
	angle         = math.Pi / 6
)
var sin30 = math.Sin(angle) // Go的常量是在编译之前就能确定的常量
var cos30 = math.Cos(angle)



func main() {
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' "+
        "style='stroke: grey; fill: white; stroke-width: 0.7' "+
        "width='%d' height='%d'>", width, height)
	for i:= 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay := corner(i + 1, j)
			bx, by := corner(i, j)
			cx, cy := corner(i, j + 1)
			dx, dy := corner(i + 1, j + 1)
			fmt.Printf("<polygon points='%g,%g %g,%g %g,%g %g,%g'/>\n",
                ax, ay, bx, by, cx, cy, dx, dy)
		}
	}
	fmt.Println("</svg>")
}

func corner(i, j int) (float64, float64) { // 返回网格顶点的坐标参数
	x := xyrange * (float64(i) / cells - 0.5)
	y := xyrange * (float64(j) / cells - 0.5)
	z := f(x, y)
	sx := width / 2 + (x - y) * cos30 * xyscale
	sy := height / 2 + (x + y) * sin30 * xyscale - z * zscale
	return sx, sy
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y)
	return math.Sin(r) / r
}
```


### 练习3.1
- 如果f函数返回的是无限制的float64值，那么SVG文件可能输出无效的多边形元素（虽然许多SVG渲染器会妥善处理这类问题）。修改程序跳过无效的多边形。

```go
\\练习3.1 更改的代码

func main() {
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' "+
        "style='stroke: grey; fill: white; stroke-width: 0.7' "+
        "width='%d' height='%d'>", width, height)
	for i:= 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay := corner(i + 1, j)
			bx, by := corner(i, j)
			cx, cy := corner(i, j + 1)
			dx, dy := corner(i + 1, j + 1)
			if math.IsNaN(ax) || math.IsNaN(ay) || math.IsNaN(bx) || math.IsNaN(by) || math.IsNaN(cx) || math.IsNaN(cy) || math.IsNaN(dx) || math.IsNaN(dy) {
				fmt.Fprintf(os.Stderr, "NAN")
			} else {
				fmt.Printf("<polygon points='%g,%g %g,%g %g,%g %g,%g'/>\n",
                ax, ay, bx, by, cx, cy, dx, dy)
			}
		}
	}
	fmt.Println("</svg>")
}

```

### 练习3.2
- 试验math包中其他函数的渲染图形。你是否能输出一个egg box、moguls或a saddle图案?

```go
// 练习3.2 更改一下z轴函数即可


func corner(i, j int) (float64, float64) {
	x := xyrange * (float64(i) / cells - 0.5)
	y := xyrange * (float64(j) / cells - 0.5)
	//z := f(x, y)
	z := eggBox(x, y)
	sx := width / 2 + (x - y) * cos30 * xyscale
	sy := height / 2 + (x + y) * sin30 * xyscale - z * zscale
	return sx, sy
}


func eggBox(x, y float64) float64 {
	return math.Sin(x) + math.Sin(y) / 10
}
```

### 练习3.3
- 根据高度给每个多边形上色，那样峰值部将是红色（#ff0000），谷部将是蓝色（#0000ff）。

```go
\\ 练习3.3
package main

import (
	"fmt"
	"math"
	"os"
)

const (
	width, height = 600, 320
	cells         = 100
	xyrange       = 30.0
	xyscale       = width / 2 / xyrange
	zscale        = height * 0.4
	angle         = math.Pi / 6
)
var sin30 = math.Sin(angle) // Go的常量是在编译之前就能确定的常量
var cos30 = math.Cos(angle)



func main() {
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' "+
        "style='stroke: grey; fill: white; stroke-width: 0.7' "+
        "width='%d' height='%d'>", width, height)
	for i:= 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay, az := corner(i + 1, j)
			bx, by, bz := corner(i, j)
			cx, cy, cz := corner(i, j + 1)
			dx, dy, dz := corner(i + 1, j + 1)
			if math.IsNaN(ax) || math.IsNaN(ay) || math.IsNaN(bx) || math.IsNaN(by) || math.IsNaN(cx) || math.IsNaN(cy) || math.IsNaN(dx) || math.IsNaN(dy) {
				fmt.Fprintf(os.Stderr, "NAN")
			} else {
				//将z映射到一个较大范围

				fmt.Printf("<polygon style='fill: ")
				
				avgz := int((az + bz + cz + dz) * 10.0 + 8.0) * 18
				
				redv, bluev := 0, 0 
				if avgz <= 255 {
					redv = 0
					bluev = 255 - avgz
				} else {
					redv = avgz - 255
					bluev = 0
				}
				if redv > 255 {
					redv = 255
				}
				if bluev > 255{
					bluev = 255
				}
				
				fmt.Printf("#%02X00", redv)
				fmt.Printf("%02X", bluev)	
				fmt.Printf("' points='%g,%g %g,%g %g,%g %g,%g'/>\n",ax, ay, bx, by, cx, cy, dx, dy)
				
			}
		}
	}
	fmt.Println("</svg>")
}
$$
func corner(i, j int) (float64, float64, float64) {
    x := xyrange * (float64(i)/cells - 0.5)
    y := xyrange * (float64(j)/cells - 0.5)

    z := f(x, y)
    sx := width/2 + (x-y)*cos30*xyscale
    sy := height/2 + (x+y)*sin30*xyscale - z*zscale
    return sx, sy, z
}

func f(x, y float64) float64 {
    r := math.Hypot(x, y) 
    return math.Sin(r) / r
}
```
### 练习3.4
- 参考1.7节Lissajous例子的函数，构造一个web服务器，用于计算函数曲面然后返回SVG数据给客户端。
- 服务器必须设置Content-Type头部： `w.Header().Set("Content-Type", "image/svg+xml")`

```go
// 直接返回给浏览器
package main

import (
	"fmt"
	"io"
	"log"
	"math"
	"net/http"
	"os"
)

const (
	width, height = 600, 320
	cells         = 100
	xyrange       = 30.0
	xyscale       = width / 2 / xyrange
	zscale        = height * 0.4
	angle         = math.Pi / 6
)
var sin30 = math.Sin(angle) // Go的常量是在编译之前就能确定的常量
var cos30 = math.Cos(angle)



func main() {
	handler := func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Type", "image/svg+xml")
		getXML(w)
	}
	http.HandleFunc("/", handler)
		log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func getXML(out io.Writer){
	fmt.Fprintf(out, "<svg xmlns='http://www.w3.org/2000/svg' "+
	"style='stroke: grey; fill: white; stroke-width: 0.7' "+
	"width='%d' height='%d'>", width, height)
	for i:= 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay := corner(i + 1, j)
			bx, by := corner(i, j)
			cx, cy := corner(i, j + 1)
			dx, dy := corner(i + 1, j + 1)
			if math.IsNaN(ax) || math.IsNaN(ay) || math.IsNaN(bx) || math.IsNaN(by) || math.IsNaN(cx) || math.IsNaN(cy) || math.IsNaN(dx) || math.IsNaN(dy) {
				fmt.Fprintf(os.Stderr, "NAN")
			} else {
				fmt.Fprintf(out, "<polygon points='%g,%g %g,%g %g,%g %g,%g'/>\n",ax, ay, bx, by, cx, cy, dx, dy)
			}
		}
	}
	fmt.Fprintln(out, "</svg>")
}

func corner(i, j int) (float64, float64) {
	x := xyrange * (float64(i) / cells - 0.5)
	y := xyrange * (float64(j) / cells - 0.5)
	z := f(x, y)
	sx := width / 2 + (x - y) * cos30 * xyscale
	sy := height / 2 + (x + y) * sin30 * xyscale - z * zscale
	return sx, sy
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y)
	return math.Sin(r) / r
}
```


## 复数
- 附属包含complex64 和 complex128, 注意分别对应的是float32 和 float64 是两倍的关系

- Mandelbrot图像， 对每个点进行$z_{k+1} = z^2_k + c$迭代测试， 迭代次数越多出范围的颜色越深形成的图形

```go
\\ web示例
// png格式的mandelbrot 图像
package main

import (
	"image"
	"image/color"
	"image/png"
	"log"
	"math/cmplx"
	"net/http"
)

func main(){
	const (
		xmin, ymin, xmax, ymax = -2, -2,  2, 2
		width, height = 1024, 1024
	)
	img := image.NewRGBA(image.Rect(0,0,width, height))
	for py := 0; py < height; py++ {
		y := float64(py) / height * (ymax - ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px) / width * (xmax - xmin) + xmin 
			z := complex(x, y)
			img.Set(px, py, mandelbrot(z))
		}
	}
	handler := func(w http.ResponseWriter, r *http.Request) {
		png.Encode(w, img)
	}
	http.HandleFunc("/", handler)
		log.Fatal(http.ListenAndServe("localhost:8000", nil))

}

func mandelbrot(z complex128) color.Color{
	const iterations = 200
	const contrast = 15
	var v complex128 
	for n := uint8(0); n < iterations; n++ {
		v = v * v + z
		if cmplx.Abs(v) > 2 {
			return color.Gray{255 - contrast * n}
		}
	}
	return color.Black
}

```

### 练习3.5
- 实现一个彩色的Mandelbrot图像，使用image.NewRGBA创建图像，使用color.RGBA或color.YCbCr生成颜色。

```go
// 随便调调参， 颜色还挺好看
// 练习3.5 实现彩色效果
package main

import (
	"image"
	"image/color"
	"image/png"
	"log"
	"math/cmplx"
	"net/http"
)

func main(){
	const (
		xmin, ymin, xmax, ymax = -2, -2,  2, 2
		width, height = 1024, 1024
	)
	img := image.NewRGBA(image.Rect(0,0,width, height))
	for py := 0; py < height; py++ {
		y := float64(py) / height * (ymax - ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px) / width * (xmax - xmin) + xmin 
			z := complex(x, y)
			img.Set(px, py, mandelbrot(z))
		}
	}
	handler := func(w http.ResponseWriter, r *http.Request) {
		png.Encode(w, img)
	}
	http.HandleFunc("/", handler)
		log.Fatal(http.ListenAndServe("localhost:8000", nil))

}

func mandelbrot(z complex128) color.Color{
	const iterations = 200
	const contrast = 15
	var v complex128 
	for n := uint8(0); n < iterations; n++ {
		v = v * v + z
		if cmplx.Abs(v) > 2 {
			return color.RGBA{50, 100, 100 + n, 255 - contrast * n} // 生成RGB效果
		}
	}
	return color.RGBA{200, 200, 100, 0}
}

```

### 练习3.6
- 升采样技术可以降低每个像素对计算颜色值和平均值的影响。简单的方法是将每个像素分成四个子像素，实现它。
- 升采样技术， 这里要求分成四个像素， 已知本来像素的中心点和宽度， 计算其他四个中心点其实不难， 结果取个平均值

```go

// 练习3.6
package main

import (
	"image"
	"image/color"
	"image/png"
	"log"
	"math/cmplx"
	"net/http"
)

func main(){
	const (
		xmin, ymin, xmax, ymax = -2, -2,  2, 2
		width, height = 1024, 1024
	)
	img := image.NewRGBA(image.Rect(0,0,width, height))
	for py := 0; py < height; py++ {
		y := float64(py) / height * (ymax - ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px) / width * (xmax - xmin) + xmin 

			xn := []float64{x - (xmax - xmin) / width / 4, x + (xmax - xmin) / width / 4}
			yn := []float64{y - (ymax - ymin) / height / 4, y + (ymax - ymin) / height / 4}
			var rnow, gnow, bnow, anow uint32 // 因为有相加操作， 所以要大一点
			//fmt.Fprintf(os.Stderr, "%g\n", xn[0])
			for _, xnow := range xn {
				for _, ynow := range yn {
					rtmp, gtmp, btmp, atmp := mandelbrot(complex(xnow, ynow)).RGBA()
					//fmt.Fprintf(os.Stderr, "%d\n", atmp)
					rnow += rtmp >> 8
					gnow += gtmp >> 8
					bnow += btmp >> 8
					anow += atmp >> 8
				}
			}
			rnow /= 4
			gnow /= 4
			bnow /= 4
			anow /= 4
			//fmt.Fprintf(os.Stderr, "%d\n", anow)
			img.SetRGBA(px, py, color.RGBA{uint8(rnow), uint8(gnow), uint8(bnow), uint8(anow)})
		}
	}
	handler := func(w http.ResponseWriter, r *http.Request) {
		png.Encode(w, img)
	}
	http.HandleFunc("/", handler)
		log.Fatal(http.ListenAndServe("localhost:8000", nil))

}

func mandelbrot(z complex128) color.Color{
	const iterations = 200
	const contrast = 15
	var v complex128 
	for n := uint8(0); n < iterations; n++ {
		v = v * v + z
		if cmplx.Abs(v) > 2 {
			return color.Gray{255 - contrast * n}
		}
	}
	return color.Black
}

```

### 练习3.7 
- 另一个生成分形图像的方式是使用牛顿法来求解一个复数方程，例如$z^4-1=0$。每个起点到四个根的迭代次数对应阴影的灰度。方程根对应的点用颜色表示。
- $f(z) = z^4-1$, 已知幂函数为解析函数， 故 $f'(z) = 4z^3$

```go

// 练习3.7
package main

import (
	"image"
	"image/color"
	"image/png"
	"log"
	"math/cmplx"
	"net/http"
)

func main(){
	const (
		xmin, ymin, xmax, ymax = -2, -2,  2, 2
		width, height = 1024, 1024
	)
	img := image.NewRGBA(image.Rect(0,0,width, height))
	for py := 0; py < height; py++ {
		y := float64(py) / height * (ymax - ymin) + ymin
		for px := 0; px < width; px++ {
			x := float64(px) / width * (xmax - xmin) + xmin 
			z := complex(x, y)
			img.Set(px, py, newton(z))
		}
	}
	handler := func(w http.ResponseWriter, r *http.Request) {
		png.Encode(w, img)
	}
	http.HandleFunc("/", handler)
		log.Fatal(http.ListenAndServe("localhost:8000", nil))

}

func newton(z complex128) color.Color{ // x_{i + 1} = x_i - f(x) / f'(x)
	const iterations = 200
	const contrast = 15
	var v complex128 
	v = z
	eps := 1e-8
	ans1, ans2, ans3, ans4 := complex(1, 0), complex(-1, 0), complex(0, 1), complex(0, -1)
	for n := uint8(0); n < iterations; n++ {
		v = v - f(v) / diff(v)
		if cmplx.Abs(v - ans1) < eps || cmplx.Abs(v - ans2) < eps || cmplx.Abs(v - ans3) < eps || cmplx.Abs(v - ans4) < eps {
			return color.Gray{255 - contrast * n}
		}
	}
	return color.Black
}

func f(z complex128) complex128 {
	return z * z * z * z - complex(1,0)
}

func diff(z complex128) complex128 {
	return 4 * z * z * z 
}
```


### 练习3.8
- 通过提高精度来生成更多级别的分形。使用四种不同精度类型的数字实现相同的分形：complex64、complex128、big.Float和big.Rat。（后面两种类型在math/big包声明。Float是有指定限精度的浮点数；Rat是无限精度的有理数。）它们间的性能和内存使用对比如何？当渲染图可见时缩放的级别是多少？

### 练习3.9
- 编写一个web服务器，用于给客户端生成分形的图像。运行客户端通过HTTP参数指定x、y和zoom参数。

```go
// 实现http传参， 先处理参数再绘制

package main

import (
	"fmt"
	"image"
	"image/color"
	"image/png"
	"log"
	"math/cmplx"
	"net/http"
	"strconv"
)

func main(){
	const (
	//	xmin, ymin, xmax, ymax = -2, -2,  2, 2
		width, height = 1024, 1024
	)
	params := map[string] float64 { // 使用map直接存， 要是在之后使用多个if判断， 耗时反而会更久
		"xmin": -2, 
		"xmax": 2,
		"ymin": -2,
		"ymax": 2,
		"zoom": 1,
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request){
			for name := range params {
				s := r.FormValue(name)
				if s == "" {
					continue
				}
				f, err := strconv.ParseFloat(s, 64)
				if err != nil {
					http.Error(w, fmt.Sprintf("query param %s: %s", name, err), http.StatusBadRequest)
					return
				}
				params[name] = f  // 读取信息的方式， 来自Gopl-homework
			}
			if params["xmax"] <= params["xmin"] || params["ymax"] <= params["ymin"] {
				http.Error(w, fmt.Sprintf("min coordinate greater than max"), http.StatusBadRequest)
					return 
			}
			xmin, xmax, ymin, ymax, zoom := params["xmin"],params["xmax"],params["ymin"],params["ymax"],params["zoom"]
			lenX, lenY := xmax - xmin, ymax - ymin
			midX, midY := xmin + lenX / 2, ymin + lenY / 2
			xmin, xmax, ymin, ymax = midX - lenX / 2 / zoom, midX + lenX / zoom / 2, midY - lenY / 2 / zoom, midY + lenY / 2 / zoom
			//fmt.Fprintf(os.Stderr, "%g %g %g %g\n", xmin, xmax, ymin, ymax)
			img := image.NewRGBA(image.Rect(0,0,width, height))
			for py := 0; py < height; py++ {
				y := float64(py) / height * (ymax - ymin) + ymin
				for px := 0; px < width; px++ {
					x := float64(px) / width * (xmax - xmin) + xmin 
					z := complex(x, y)
					img.Set(px, py, mandelbrot(z))
				}
			}
			png.Encode(w, img)
	})
	log.Fatal(http.ListenAndServe("localhost:8000", nil))

}

func mandelbrot(z complex128) color.Color{
	const iterations = 200
	const contrast = 15
	var v complex128 
	for n := uint8(0); n < iterations; n++ {
		v = v * v + z
		if cmplx.Abs(v) > 2 {
			return color.Gray{255 - contrast * n}
		}
	}
	return color.Black
}
```

- Go 同C含有表达式短路机制

## 字符串
- 首先 Go中的字符串是不允许被修改的
- 字符串S[i]表示读第i个字节， 并不一定是第i个字符
- Unicode 编码标准优良， 可以不用解码检查前后缀
- 在程序内部使用rune序列可以更加方便， 大小一致， 便于切割
  
- 书上的示例
```go

// 三个示例写一个文件里了

// 将看起来像是 系统目录的前缀删除， 并将看起来像后缀名的部分删除
func basename (s string) string {
	for i := len(s); i >= 0; i-- {
		if s[i] == '/' {
			s = s[i + 1: ]
			break
		}
	}
	for i := len(s) - 1; i >= 0; i-- {
		if s[i] == '.' {
			s = s[:i]
			break
		}
	}
	return s
}

// 使用strings.LastIndex 
func basename1 (s string) string {
	slash := strings.LastIndex(s, "/")
	s = s[slash + 1:]
	if dot := strings.LastIndex(s, "."); dot >= 0 {
		s = s[:dot]
	} 
	return s
}

// comma 

func comma(s string) string {
	n := len(s)
	if n <= 3 {
		return s
	}
	return comma(s[:n - 3] + "," + s[n - 3:])
}

```

### 练习3.10
-  编写一个非递归版本的comma函数，使用bytes.Buffer代替字符串链接操作。

```go
// 实现非递归版本的comma函数

package main

import "bytes"

func main(){
	for _, s := range []string{"1", "12", "123", "1234", "12345678901234", "123456789012345", "1234567890123456", "12345678901234567", "123456789012345678", "1234567890123456789"} {
		println(comma(s))
	}
}

func comma(s string) string {
	var buf bytes.Buffer
	len := len(s)
	mod1 := len % 3
	if mod1 == 0 && len >= 3 {
		mod1 = 3
	}
	for be := 0; be + mod1 <= len ;{
		buf.WriteString(s[be:be+mod1])
		if be + mod1 != len {
			buf.WriteString(",")
		}
		be += mod1
		mod1 = 3
	}

	return buf.String()
}
```

## 练习3.11
-  完善comma函数，以支持浮点数处理和一个可选的正负号的处理。

```go
// 实现非递归版本的comma函数

package main

import (
	"bytes"
	"strings"
)

func main(){
	for _, s := range []string{"1", "12", "123", "1234", "12345678.901234", "-123456789012.345", "123456789012.3456", "123456789012345.67", "12345678901234567.8", "12.34567890123456789"} {
		println(comma(s))
	}
}

// 更改后的comma 函数支持将浮点数, 小数部分是从左往右
func comma(s string) string {
	var buf bytes.Buffer
	len := len(s)
	if len > 0 && s[0] == '-' {
		buf.WriteByte('-')
		s = s[1:]
	}
	dotPlace := strings.IndexByte(s, '.') 
	if dotPlace > 0 {
		buf.WriteString(comma1(s[:dotPlace]))
		buf.WriteByte('.')
		buf.WriteString(comma2(s[dotPlace + 1:]))
	} else {
		buf.WriteString(comma1(s))
	}
	return buf.String()
}
// 整数部分
func comma1(s string) string {
	var buf bytes.Buffer
	len := len(s)
	mod1 := len % 3
	if mod1 == 0 && len >= 3 {
		mod1 = 3
	}
	for be := 0; be + mod1 <= len ;{
		buf.WriteString(s[be:be+mod1])
		if be + mod1 != len {
			buf.WriteString(",")
		}
		be += mod1
		mod1 = 3
	}
	return buf.String()
}
// 小数部分
func comma2(s string) string {
	var buf bytes.Buffer
	len := len(s)
	
	for be := 0; be < len ; be += 3{
		if be + 3 < len {
			buf.WriteString(s[be:be+3])
			buf.WriteString(",")
		} else {
			buf.WriteString(s[be:])
		}
	}
	return buf.String()
}
```

### 练习3.12
- 编写一个函数，判断两个字符串是否是相互打乱的，也就是说它们有着相同的字符，但是对应不同的顺序。

```go

// 使用两个map记录两个字符串字符出现次数
// 

package main

import "fmt"

func main(){
	s1, s2 := "abc", "cba"
	fmt.Println(cmp(s1, s2))
}

func cmp(s1, s2 string) bool {
	if s1 == s2{
		return false 
	}
	m1, m2 := make(map[string]int), make(map[string]int)
	for i := 0; i < len(s1); i++ {
		m1[string(s1[i])]++
	}
	for i := 0; i < len(s2); i++ {
		m2[string(s2[i])]++
	}
	for k, v := range m1 {
		if m2[k] != v {
			return false
		}
	}
	return true
}
```


## 常量

### iota 常量生成器
- iota 常量生成器， 可以在常量声明中用iota作为常量值， 它会自增， 从0开始
```go
type Weekday int

const (
    Sunday Weekday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)
```
- 常量可以表示为无类型的

### 练习3.13
- 编写KB、MB的常量声明，然后扩展到YB。

```go
// 练习3.13 编写KB、MB的常量声明，然后扩展到YB。
// 拆分二进制
package main

func main() {
	const (
		KB = 1000
		MB = 1000 * KB
		GB = 1000 * MB
		TB = 1000 * GB
		PB = 1000 * TB
		EB = 1000 * PB
		ZB = 1000 * EB
		YB = 1000 * ZB
	)
	//fmt.Println(KB, MB, GB, TB, PB, EB, ZB,YB)
}

```

