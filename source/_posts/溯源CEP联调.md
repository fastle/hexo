---
title: 溯源CEP联调
date: 2024-12-23 14:28:51
tags:
---
# 提出的问题
1. 多 agent
2. 重启后不需要重新读入


# idea

1. 资源-Controller 可以有多种实现， 我们现在只考虑cep引擎怎么更改资源配置。




# 设计

1. 放到task.uhs.com 中， [资源配置文档](https://doc.weixin.qq.com/smartsheet/s3_AKsAogbwALQeMeOyFjrR36OY3fYGM?scode=AD4AGAfCAAwMRvcO1XAKsAogbwALQ&tab=q979lj&viewId=vukaF8)

2. 参考FileTraceDel 的实现， 其也为实现单agent 监听。

3.EventTraceTask















# Think
- 代码一定分为三部分
- config_channel 中的部分是自动生成的, 不太应该就行更改， 如果要更改需要停止其自动生成
- agent-go 中实现注册Informer
- controller 注册Informer 
- 之前这块不是在 controller-go 中写的， 到时候前端发个url给我说要下发， 获取信息还是维持之前的 url 。我这边应该也能判断是否入库。
-   


我已经理解了一切。

# 调用

## agent 端
- 在 Informer 中调用溯源函数。
- 传两个参数 (guid string, taskID string)
- 需要把 taskID 存到上传事件信息中。

## controller 端
- 由 cep 调用。
- 引用 "uhs/controller/controller-go/eventtrace"
- 函数 "SendTraceTasks(info)(err error)" 传递

- 硬

## web
- get， post


# eventTraceTask 资源新增

## 部署
### agent
1. 重新编译 uhs-data.exe, curator.exe替换。 重新启动
### controller 
1. 重新编译 config_channel 
2. 增加 agent 资源权限。更改 `/fsdata/hscs/helms/config-channel/templates/configmap.yaml`
添加
```yaml
- apiGroups: [task.uhs.com]
  resources: ["eventtracetasks", "eventtracetasks/status"]
  verbs: ["list", "watch", "create", "update", "get", "delete"]
```
3. 参考 [5.1.3 config_channe手动替换部署方法](https://10.100.8.145/pages/viewpage.action?pageId=39157957)
4. 若部署脚本执行失败，重新执行一次试试。 若部署完无法查看资源，执行。
```
kubectl apply -f /data/deployment/uhs-packages/nodeport.yaml
```

## 调用

### agent

目录在 `/agent/beats3/libbeat/processors/edr_trace/trace_task.go`

实现其中的 `dealTraceTask(guid string, taskID string)` 函数，该函数会在每次资源 Create 或 Update 的时候执行一次。 期望行为是根据 `guid` 进行溯源， 在上传的事件中添加字段保存 `taskID` 信息。

## controller

目录在 `/controller/controller-go/pkg/tracetask/deal_cep_trace.go`

调用其中的 `SendTraceTasks(infos []TraceInfo) (err error)` 函数, 其中 TraceInfo 字段如下， 代表一个溯源需求。
```go
type TraceInfo struct {
	AgentID  string
    Guid string
}
```

