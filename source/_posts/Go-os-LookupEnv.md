---
title: Go-os.LookupEnv
tags:
  - 随笔
  - Go
date: 2024-10-14 10:55:49
---

```go
func LookupEnv(key string)(string, bool)
```

检索由键命名的操作系统环境变量的值。如果变量存在于环境中，则返回值(可能为空)并且布尔值为真。否则返回值为空，布尔值为假。

我遇到的代码是。

```go
_, err := os.LookupEnv("LOCALIC")
 
if !err {
  in = bufio.NewReader(os.Stdin)
  out = bufio.NewWriter(os.Stdout)
} else {
  f1, _ := os.Open("input.txt")
  in = bufio.NewReader(f1)

  outf, _ := os.OpenFile("output.txt", os.O_RDWR, fs.ModeAppend)
  outf.Truncate(0)

  out = bufio.NewWriter(outf)
}
```

其中 `LOCALIC` 是一个环境变量。 可能是自定义的。用于区分不同环境下的输入输出方向。 