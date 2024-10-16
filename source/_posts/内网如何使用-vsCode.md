---
title: 内网如何使用 VSCode
tags:
  - 随笔
date: 2024-10-16 16:53:15
---

两个需求， 能使用 ssh 功能， 能使用 Go 的扩展。 结果坑还不少

# 扩展安装

强烈建议在安装扩展前， 在内网中将 VSCode 安装为最新版。

之后在 [扩展市场](https://marketplace.visualstudio.com/vscode) 找到你要安装的扩展， 点击右边的 **Download Extension**, 下载到本地， 传到内网。

![](/images/vscode_extention.png)

然后在 VSCode 的扩展界面上选择 **从 VSIX 安装**

![](/images/vsix.png)

这样版本符合的话就能安装了。

但是安装上了不代表能正常使用

# Remote SSH 调教

问题出在远程机器无法正常下载 .vscode-server

然后我从这个[博客](https://blog.csdn.net/blxmove/article/details/118605830) 找到了解决方法。 

# Go 扩展调教

问题出在扩展功能需要下一些 Golang 的包。

解决方法是挑几个重要的包安装， 有些包就不安了， 因为我们内网添加代理时增添包比较麻烦。


