---
title: Go 时间戳获取转换
date: 2024-09-26 10:57:02
tags:
 - Go
 - 随笔
---

# 获取时间

time.Now() 返回 time.Time 类型的时间， Time.Unix() 返回当前时间的秒级时间戳， 起始时间为2006-01-02 15：04：05， Go 语言诞生的时间。

时间转字符串
```go
time.Now().Format("2006-01-02")
```

时间戳转时间字符串
```go
timeTemplate1 := "2006-01-02 15:04:05"
t := int64(1546926630)     
timeStr := time.Unix(t, 0).Format(timeTemplate1)
```

时间字符串解码

```go
time1 := "2015-03-20 08:50:29"
t1, err := time.Parse("2006-01-02 15:04:05", time1)
```
