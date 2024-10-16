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

