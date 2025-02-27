---
title: '[Bug 493919] [FR 45476]【Agent】【UES-5.0 R6】Windows桌面和linux图形化桌面无法收到用户确认的弹窗'
date: 2025-02-17 09:46:21
tags:
---

# 基本信息
【问题版本】

Svn Revision: 24891

Branch: UES_DEV_R6

Last Changed Date: 2025-02-11 17:47:48 +0800 (Tue, 11 Feb 2025)

Build Date: 20250212003138

【问题描述】

环境：controller 10.160.44.70 root/hillstone  hillstone/Hillstone@123        agent ： windows 10.160.51.164  Administrator/hillstone      linux桌面版：10.160.51.170  root/hillstone

1.选中通用文件列表的一个文件，创建文件分发任务，Windows文件存储位置修改为C:\UESDownload\FileDistribution\

2.用户提醒选择用户确认后分发，创建任务

3.Windows和linux桌面无法收到用户确认的弹窗，任务状态一直是正在分发



【期望结果】

Windows和linux桌面收到用户确认的弹窗，任务正常运行

# 直接登录测试环境进行复现
- 选定任务分发
- 会显示一直分发， 

# 定位过程
- 打log发现无法输出， 怀疑没注册监听事件。
- 查找分支合并时文件变动， 发现curator启动程序代码没合进来， 修改之。



![alt text](/images/image.png)