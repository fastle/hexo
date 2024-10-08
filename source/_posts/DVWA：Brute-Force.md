---
title: DVWA：Brute Force
date: 2024-09-25 16:23:07
tags:
 - 网络攻防
 - DVWA
---

暴力破解基本逻辑是使用弱密码和枚举法， 

# Low 
在 low 模式下进行攻击获取流程为， 首先发送验证信息并进行抓包， 然后使用 Intruder 进行爆破。 

# Medium
使用了符号转义， 可以防止 sql 注入， 不影响我们直接爆破。

# High
使用了 token ， 每次查询失败会 sleep 随机0到3秒。

对于 token ， 需要在爆破时分析 response 的返回值， 找到 token 的位置， 然后在爆破时将 token 添加到 payload 中。

# Impossible
在 high 的基础上对用户登录次数有所限制， 三次后会锁柱15秒。 就比较难爆破了。


