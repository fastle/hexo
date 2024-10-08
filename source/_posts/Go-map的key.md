---
title: Go map的key
tags:
  - 随笔
  - Go
date: 2024-09-26 11:56:15
---
boolean, numeric, string, pointer, channel, interface 均可作为key

其中 interface 的相等判定是动态类型和动态值都要相等

然后 structs 和 arrays 如果只含有上面这些类型的元素， 那么也可以作为key

slices， maps， functions 不可以作为key
