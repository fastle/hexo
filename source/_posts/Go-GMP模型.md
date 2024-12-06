---
title: Go GMP模型
tags:
  - 随笔
  - Go
date: 2024-09-26 13:56:45
---
G：Goroutine，实际上我们每次调用 go func 就是生成了一个 G。

P：Processor，处理器，一般 P 的数量就是处理器的核数，可以通过 GOMAXPROCS 进行修改。

M：Machine，系统线程。

也就是 M 必须与 P 进行绑定，然后不断地在 M 上循环寻找可运行的 G 来执行相应的任务。

https://zhuanlan.zhihu.com/p/261057034