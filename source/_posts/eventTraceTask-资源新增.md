---
title: eventTraceTask 资源新增
date: 2024-12-27 11:31:51
tags:
---
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