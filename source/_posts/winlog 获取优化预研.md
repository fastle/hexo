---
title: 事件查看优化预研
date: 2024-12-10 10:19:17
tags: 
 - FR
---

# 背景

当前规则团队新增的规则需要手动修改从 Windows 获取的事件， 非常不方便。 因此需要进行自动化。

从以下三个方面入手。

## 自动化动态调整日志的订阅

## 自动化修改事件的过滤规则

## 自动化修改系统相关配置开关和设定
- 该部分实现后默认关闭， 设定为只有管理员从 UI 下发才能修改。


# 参考 FD
- 代京 [https://rd.hillstonenet.com/doc/detail/unique-161393](FR33088-FR33089-FR33084-windows&linux数据流&用户态事件过滤机制-FD.docx)

# 定位

edr_trace 溯源

LRU filebeat/input/hillstone/lruCache

winlog 日志按照条件过滤 /beats3/uhs-log/ues/uhs-data-main.yml

bugID 436881 数据过滤 增加EDR winlog 事件的自动过滤配置。  r18546 r18629

beats3/uhs-log/ues/uhs-data-input-windows.yml
beats3/libbeat.processors/enrich_process/process_fields.go

- [x] 文件变动监听
- [x] 解密解码
- [x] winlog 日志采集需求统计
- [ ] winlog 配置文件写入
- [ ] 日志采集设置与采集开关
- [x] 管道新增模块
- [ ] 单测编写
- [ ] 测试

# problem

1. cal.go 271-282 怀疑有 ticker 泄露。

1. 有的日志后加 enrich  有的日志没有， 如何统一。（如何识别要进行额外的操作）
2. 对应规则的注意事项。（什么样的规则用 winlog。规则解码出来后如何找到对应的日志相关信息。）
3. 加在哪里是合理可行的。（是否云铠智铠共用）



1. rules to 其它日志采集是否有类似实现。




# Notes

filebeat/input/hillstone/start_windows.go 具有 过滤字样  164 但是看起来这块是云铠的

decode_xml_wineventlog 没用


gowin32.OpenWTSServer函数主要用于打开一个 Windows 终端服务（Terminal Services，WTS）服务器的连接。这个功能在管理远程桌面会话和服务器相关操作中非常有用。

框架实现上已经实现了热重载， 理论上只需要更新 input 文件即可。

## 通过查看

## ioa.yml 
Line 115739: security


## uhs-data-input-windows.yml 
- event_id 中 负号前缀 表示 不包含。
- event_id 信息不能超过 22 个（最好不要超过21个）


## 


## 文件监控 fsnotify 
- 在beats 中可以使用， 用于监控某个文件的变动


- once.Do 只执行一次。


```go
go func() {
        for {
            select {
            case event, ok := <-watcher.Events:
                if!ok {
                    return
                }
                // 判断事件类型是否是写入（变动）
                if event.Op&fsnotify.Write == fsnotify.Write {
                    fmt.Println("配置文件发生变动:", event.Name)
                    // 获取变动后的文件绝对路径
                    absPath, err := filepath.Abs(event.Name)
                    if err!= nil {
                        log.Printf("获取文件绝对路径失败：%v", err)
                        continue
                    }
                    var updatedConfig1Data, updatedConfig2Data []byte
                    if absPath == configFile1 {
                        updatedConfig1Data, err = ioutil.ReadFile(configFile1)
                        if err!= nil {
                            log.Printf("重新读取配置文件1失败：%v", err)
                            continue
                        }
                    }
                    if absPath == configFile2 {
                        updatedConfig2Data, err = ioutil.ReadFile(configFile2)
                        if err!= nil {
                            log.Printf("重新读取配置文件2失败：%v", err)
                            continue
                        }
                    }
                    update(updatedConfig1Data, updatedConfig2Data)
                }
            case err, ok := <-watcher.Errors:
                if!ok {
                    return
                }
                log.Println("监控文件出错:", err)
            }
        }
    }()
```



在 Go 语言中使用gopkg.in/yaml.v3库实现流式读取
基本原理
流式读取是一种逐块处理数据的方式，而不是一次性将整个文件内容读取到内存中。对于大型的yaml文件，这种方式可以有效减少内存占用。gopkg.in/yaml.v3库提供了相应的功能来实现yaml文件的流式读取和解码。
示例代码
假设yaml文件内容如下（为了演示流式读取，假设文件内容较多，有多个person记录）：
```yaml
       people:
         - name: "John"
           age: 30
           hobbies:
             - reading
             - coding
         - name: "Alice"
           age: 25
           hobbies:
             - dancing
             - painting
```
首先，定义一个结构体来接收yaml数据：
```go
type Person struct {
    Name    string   `yaml:"name"`
    Age     int      `yaml:"age"`
    Hobbies []string `yaml:"hobbies"`
}
```
以下是流式读取yaml文件的代码：
```go
package main

import (
    "fmt"
    "gopkg.in/yaml.v3"
    "os"
)

func main() {
    file, err := os.Open("example.yaml")
    if err!= nil {
        fmt.Println("打开文件出错：", err)
        return
    }
    defer file.Close()
    decoder := yaml.NewDecoder(file)
    var people []Person
    for {
        var person Person
        err := decoder.Decode(&person)
        if err == yaml.EOF {
        break
        } else if err!= nil {
        fmt.Println("解码出错：", err)
        return
        }
        people = append(people, person)
    }
    fmt.Println("读取到的人员信息：")
    for _, p := range people {
        fmt.Printf("姓名：%s，年龄：%d，爱好：%v\n", p.Name, p.Age, p.Hobbies)
    }
}
```
在这个示例中：
首先打开yaml文件，然后使用yaml.NewDecoder函数创建一个解码器decoder，它会从打开的文件中逐块读取yaml数据。
通过一个循环，不断调用decoder.Decode方法来读取并解码每一个person结构体的数据。当遇到文件结束符（yaml.EOF）时，循环结束。
如果在解码过程中出现错误（除了文件结束符），则打印错误信息并返回。
最后，将读取到的所有person信息打印出来。



# 对于