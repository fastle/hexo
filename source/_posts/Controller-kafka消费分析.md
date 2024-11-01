---
title: Controller kafka消费分析
date: 2024-10-29 10:32:56
tags:
---

来表演一下盲人摸象。

# controller-correlation-service

该模块从 Yaml 文件读入解析规则， 并构建返回 Application。 调用了 analyze.Application , 但是有点奇怪， 搜不到其他使用 Application 的。




然后也找不到调用这个库的。 可能起到个初始化作用 ？


基本路径

rulePath -> applicationManager.add(rules)

# licenseApplicationFeignClient

该模块



# controller-threat-graph-service 

参与事件溯源

该模块比较好观察 kafka 信息。 ThreatGraphApplicationService 中消费事件 id 溯源事件。

使用 Spring 中 spring-kafka 

名称为 `graphTaskConsumer`, 监听 topic `threat_graph_task_queue` （但是这个在R4图中没清晰看到）， 监听器工厂为 `threatGraphTaskFactory`

`threatGraphTaskFactory` 处于 `controller.config.kafka` , 在这里设置了配置信息， 并设置为单线程。




# 当前Event Topic 

controller-flink-job/flink-job-util/.../ues/flink/common/util/threat 

UES_SYSMON "ues_sysmon" "sysmon事件"
UES_OSQUERY "ues_osquery" "osquery事件"
UES_WINEVENT "ues_winevent"  "winevent事件"
UES_OTHER "ues_other" "othershi事件"
THREAT_LOG "THREAT_LOG" "行为事件"
SUSP_LOG "susp_log" "可疑事件"
BRUTE_FORCE_EVENT "brute_force_event" 
LIBRARY_UPDATE_TOPIC library_update
EDR_THREAT_LOG "edr_threat_log" 威胁事件 // 
UHS_EDR "uhs_edr"
GRAPH_ENGINE_EVENT "graph_engine_event"    // 没有任何使用记录
THREAT_GRAPH_TASK "threat_graph_task_queue" 通知威胁溯源 
CORRELTION_PRE_LOG "correlation_pre_log" 关联分析预处理日志
GRAPH_LOG "edr_graph" 图引擎处理日志
ICA_MATCH_TOPIC "ica_match" 图引擎数据


当前 add kafka 需要修改

1. controller-flink-job/flink-job-util/.../ues/flink/common/util/threat  增加常量
2. controller-flink-job-mgnt/.../job/application/FlinkJobMgntApplicationService 增加 create Topics // 这个地方的createOrModifyTopics(org.apache.kafka.clients.admin.NewTopic... topics) 不知道有没有用到
3. 输出到


除了通知威胁溯源， 其他全都是从 flink 里面走的


连接kafka 用到 AbstractKafkaFactory.createKafkaSource

点进去是调用 Flink 的KafkaSource， 设置GroupID是提出来的


[KafkaSource使用指南](https://shigure.me/archives/flink%E6%96%B0%E7%89%88kafka%E8%BF%9E%E6%8E%A5%E5%99%A8%E7%9A%84kafkasource%E4%B8%8Ekafkasink%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)