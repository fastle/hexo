---
title: Go test中运行指定的benchmark
tags:
  - 随笔
  - Go
date: 2024-09-26 11:42:30
---

我遇到的问题是跑benchmark的时候， 非benchmark的测试用例也会跑。

查了查应该加上 -run=none 来禁用Testing

