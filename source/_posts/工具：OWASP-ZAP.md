---
title: 工具：OWASP ZAP
tags:
  - 随笔
  - 网络攻防
  - tools
date: 2024-09-29 14:50:12
---


# 工具介绍
OWASP 是一个开源的、非营利性的全球性安全组织， 使命是使应用软件更加安全， 使企业和组织能够对应用安全风险做出更清晰的决策。

ZAP 全称 OWASP Zed Attack Proxy, 是世界上最受欢迎的免费安全工具之一， 作用在浏览器和 web 之间， 对流量进行拦截、检查、发送。

# 安装

有官方网站， 有安装包， Windows 和 Linux 安装的话需要 JRE11+， MAC 则不需要。 另外有官方的 docker 环境。

# 使用

## 会话机制

ZAP 的每个“project”都是一个会话， 每次打开 ZAP 都会新建一个会话， 可以选择关闭时是否保存。

![](images/zapnew.png)

## Getting Started
当前情况下，ZAP 可以使用自动扫描和手动扫描。 自动扫描执行傻瓜式的渗透测试， 可以进行自动扫描无需配置代理。

自动扫描结束后可以看到警报。

这里简单试一下我们之前做过的 DVWA 靶场， 扫描结果如下。

![](images/zap_dvwa.png)