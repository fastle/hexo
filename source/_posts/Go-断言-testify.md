---
title: GO 断言(testify)
tags:
  - 随笔
  - Go
date: 2024-09-26 11:25:58
---


Go标准包中为什么没有断言，官方在FAQ里面回答了这个问题。

官方说是为了防止程序员在错误处理上偷懒。

但其实个人感觉引入的话可读性更高嘿嘿


基本使用方法大概如下

```go
func TestTestify(t *testing.T){
    a := assert.New(t)
    a.Equal(v, want)
    a.Nil(err,"Information ...")
}
```
