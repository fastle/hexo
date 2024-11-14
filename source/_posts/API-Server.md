---
title: API Server
date: 2024-11-14 13:34:36
tags:
---
注意：不要把apiserver当作controller与agent交互的中间对象来使用，保持单向性原则，一方修
改，另一方得到配置，避免同时修改一个字段。apiserver放的是配置，应当只有controller下发配
置，agent得到配置执行。agent通过status更新的最好是状态，对于agent需要上报的数据，应该
走数据通道。


