---
title: Go time.Tick 和 time.After 内存泄露
tags:
  - 随笔
  - Go
date: 2024-09-26 13:41:57
---
time.Tick 和没结束的 timer.After 在函数结束时不会被gc， 于是导致内存泄漏

解决方法是使用 time.Ticker 并进行正常关闭

Go1.23 修复啦， 一旦不引用就可以gc