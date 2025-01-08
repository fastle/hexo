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
- [x] winlog 配置文件写入
- [x] 日志采集设置与采集开关
- [x] 管道新增模块
- [x] 单测编写
- [x] 常见EventID 对应Channel 映射（规则已存在）。
- [ ] 常见EventID 对应Channel 映射（规则未存在）。

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
package main

import (
    "github.com/fsnotify/fsnotify"
    "log"
    "path/filepath"
)

// update是一个自定义的函数，用于处理文件变动后的逻辑，这里只是示例，可按实际需求修改
func update(filePath string) {
    log.Printf("文件 %s 发生变动，执行更新操作...\n", filePath)
    // 在这里添加具体的更新逻辑，比如重新读取文件内容等
}

func watchFiles(file1, file2 string) {
    watcher, err := fsnotify.NewWatcher()
    if err!= nil {
        log.Fatal(err)
    }
    defer watcher.Close()

    done := make(chan struct{})
    // 用于去重，避免重复处理同一个文件变动事件
    processed := make(map[string]struct{})

    go func() {
        for {
            select {
            case event, ok := <-watcher.Events:
                if!ok {
                    return
                }
                // 只关注写入、创建、重命名、删除等可能导致文件内容变化的事件
                if event.Op&(fsnotify.Write|fsnotify.Create|fsnotify.Rename|fsnotify.Remove)!= 0 {
                    absPath, err := filepath.Abs(event.Name)
                    if err!= nil {
                        log.Printf("获取文件绝对路径出错: %v\n", err)
                        continue
                    }
                    if _, ok := processed[absPath];!ok {
                        if absPath == file1 || absPath == file2 {
                            update(absPath)
                            processed[absPath] = struct{}{}
                        }
                    }
                }
            case err, ok := <-watcher.Errors:
                if!ok {
                    return
                }
                log.Printf("监听文件出错: %v\n", err)
            }
        }
    }()

    // 添加要监听的文件
    err = watcher.Add(file1)
    if err!= nil {
        log.Fatal(err)
    }
    err = watcher.Add(file2)
    if err!= nil {
        log.Fatal(err)
    }

    <-done
}

func main() {
    file1 := "path/to/file1"  // 替换为实际的第一个文件路径
    file2 := "path/to/file2"  // 替换为实际的第二个文件路径
    watchFiles(file1, file2)
}
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
- ruhe ping jia wang zuo  zuo pai pai zuo ta le da zhou 
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
- 因为加解密， 所以最后不能进行流式输入

- 



# winlog
- 存在事件ID复用
1. **Event ID 1000在不同场景下的复用**
   - 在Windows应用程序日志（Application Log）中，事件ID 1000通常与应用程序错误有关。当一个应用程序（如Microsoft Word）发生崩溃时，会在应用程序日志中记录一个事件ID为1000的事件。这个事件的详细信息可能包含应用程序崩溃的模块名称、异常代码等内容，用于帮助开发人员定位问题。
   - 然而，在某些系统进程（如Windows系统自带的一些服务）出现内部错误并记录日志时，也可能会使用事件ID 1000。例如，Windows更新服务（wuauserv）在更新过程中遇到某些非预期的错误，导致其内部的一个子模块出错，也可能会在系统日志中记录事件ID为1000的事件。不过，此时事件的详细描述会与应用程序崩溃时有所不同，可能会涉及更新相关的组件名称、更新文件的版本等信息。
2. **Event ID 7000在服务启动中的复用**
   - 在Windows服务相关的日志中，事件ID 7000通常用于表示服务启动相关的事件。当一个用户安装并首次启动一个自定义开发的Windows服务时，会记录事件ID为7000的事件，详细信息可能包括服务名称、服务启动类型（自动、手动等）、启动时的依赖关系等。
   - 但在系统更新或者系统组件重新配置后，某些系统关键服务（如Windows防火墙服务）重新启动时，同样会记录事件ID 7000的事件。这里的事件详细描述可能会涉及服务重启的原因（如因为安全策略更新而重启）、服务重启后的状态与之前状态的对比等信息。
3. **Event ID 4624（用户登录）在不同登录方式下的复用**
   - 在安全日志中，事件ID 4624主要表示用户成功登录。当用户通过本地账户在本地计算机上登录时，会记录事件ID 4624的事件，详细信息可能包括登录的用户名、登录的时间、登录的方式（如交互式登录）等。
   - 当用户通过远程桌面协议（RDP）远程登录到另一台计算机时，同样会记录事件ID 4624的事件。不过，此时事件的详细描述会包含远程登录的源IP地址、远程登录的会话ID等额外信息，以区分本地登录和远程登录这两种不同的登录场景。





# 
```go
func buildDataset {

}
```


# 一份奶茶算上配送费30块，喝不起 






















# winlog 
- 存在事件id 复用

1. **Event ID 1000在不同场景下的复用**



以下是一种基于Kubernetes（k8s）实现多台机器通过资源进行通信，由Controller下发命令，Agent监听并完成任务的架构设计示例，同时尽量考虑避免常见问题：

### 整体架构概述
该架构主要包含三个核心组件：Controller（控制端）、Agent（代理端，运行在各个节点机器上）以及共享的资源存储（可以利用Kubernetes自身的资源对象等作为媒介来实现），它们之间通过合理的机制交互协作，以完成命令下发与执行的流程。

### 一、Controller组件设计
1. **功能职责**
    - 负责生成具体的任务命令，这些命令可以基于业务逻辑进行定制，例如配置变更、服务启动/停止指令等。
    - 监控任务的执行状态，需要和Agent保持通信，收集任务执行反馈，便于后续决策（如重试、告警等）。
    - 与Kubernetes API Server进行交互，将任务相关的信息以合适的资源形式存储到集群中，方便Agent获取。
2. **实现要点及避免的问题**
    - **配置管理**：采用配置文件或者环境变量等方式加载自身的配置信息，配置项例如连接Kubernetes API Server的认证信息、默认的任务超时时间等。为避免配置错误导致无法正常连接集群或者不合理的任务超时设定影响执行效果，在启动时进行配置合法性校验，并且提供配置热更新功能（可以通过监听配置变更的ConfigMap等机制实现），以便在运行中修正配置问题。
    - **与API Server交互**：使用Kubernetes官方提供的客户端库（如Go语言的 `client-go`）来与API Server通信，确保按照API规范操作资源。避免频繁创建过多的连接实例浪费资源，可采用连接池技术复用连接；同时处理好连接异常情况，设置合理的重试策略，防止因网络闪断等原因导致命令无法下发到集群资源存储处。
    - **任务调度与状态管理**：基于任务队列来管理待下发的任务，保证任务的顺序性和并发控制（例如限制同时下发的任务数量，避免集群资源瞬间压力过大）。维护一个任务状态的存储（可以是内存中的数据结构结合持久化存储如数据库等），准确记录任务从创建、下发到执行各个阶段的状态，避免出现任务状态丢失或者不一致的情况，对于长时间处于异常状态的任务要能及时告警。

### 二、Agent组件设计
1. **功能职责**
    - 运行在各个Kubernetes节点机器上，监听特定的资源变化（例如自定义资源或者ConfigMap、Secret等的更新），这些资源变化即代表Controller下发的命令。
    - 解析获取到的命令内容，并按照命令要求在本地节点执行相应的操作，操作可能涉及到调用系统命令、操作容器运行时（如Docker、containerd）等。
    - 将任务执行的结果（成功、失败及详细的错误信息等）反馈给Controller，以便Controller掌握整体执行情况。
2. **实现要点及避免的问题**
    - **监听机制**：利用Kubernetes的Watch机制（通过客户端库监听资源对象的变更事件）来实时获取命令相关资源的更新情况，避免采用轮询方式消耗过多的系统资源和网络带宽。同时，要做好对Watch连接异常的处理，当连接断开时要有自动重连机制，防止遗漏命令更新事件。
    - **命令解析与执行**：对获取到的命令内容进行严格的格式校验，按照预先定义好的命令格式规范进行解析，避免因命令格式错误导致执行出错。在执行命令时，要考虑命令执行的超时处理（可以通过设置系统信号或者定时器等方式实现），防止命令执行时间过长导致节点资源被长时间占用，影响其他服务；另外，执行涉及系统权限的操作时，遵循最小权限原则，避免因权限过大带来安全隐患。
    - **反馈机制**：通过可靠的通信方式（比如向Controller暴露的API接口发送HTTP请求或者使用消息队列等）将任务执行结果反馈回去，要处理好反馈过程中的网络异常情况，采用重试、缓存结果待网络恢复后发送等策略，确保结果能准确传达给Controller，避免出现Controller因未收到反馈而错误判断任务执行情况。

### 三、共享资源存储设计
1. **资源选择与定义**
可以选择自定义资源（Custom Resources，通过CRD定义）来承载具体的命令内容和相关的任务元数据（如任务ID、优先级等），或者利用ConfigMap（适合简单的配置类命令）、Secret（用于包含敏感信息的命令场景，如包含密码的数据库操作命令等）等已有的Kubernetes资源类型，根据实际业务需求灵活决定。
2. **数据一致性与版本控制**
不管选择哪种资源形式，要确保资源数据在多副本环境下的一致性，Kubernetes本身对于资源存储有一定的分布式一致性保障机制，但在更新频繁等特殊场景下，可以考虑添加额外的版本控制字段（类似乐观锁机制，通过比对版本号来决定是否更新成功），避免出现数据冲突问题。同时，对于历史命令记录等可以根据业务需要进行合理的保留和清理，防止存储资源无限增长。

### 四、通信安全与可靠性保障
1. **安全方面**
    - 在Controller与Agent通信以及它们和Kubernetes API Server交互过程中，使用TLS加密传输数据，避免命令等敏感信息在网络传输中被窃取或篡改。
    - 对Controller和Agent进行身份认证，例如基于Kubernetes的RBAC（基于角色的访问控制）机制，确保只有合法授权的组件才能进行相应的操作（Controller有权限下发命令到指定资源，Agent有权限读取和更新对应节点相关的资源等），防止非法访问带来的安全风险。
2. **可靠性方面**
    - 整个架构的各个组件都要具备良好的容错能力，如Controller可以部署多个副本实现高可用，通过负载均衡等方式将请求均匀分发；Agent所在的节点若出现故障（机器宕机、网络故障等），在节点恢复后能够自动重新连接到集群，继续监听任务命令并执行后续操作，确保整体任务执行不受单点故障过多影响。

通过上述这样一个较为完善的架构设计，在Kubernetes多台机器环境下，能较好地实现Controller下发命令，Agent监听并执行的流程，同时通过一系列机制避免常见的配置、通信、安全以及执行方面的问题，保障系统的稳定可靠运行。不过在实际落地中，还需要根据具体的业务场景、集群规模等因素进一步优化和调整各部分的细节实现。 


#### 注意事项
1. 在 agent 重启之后，是否需要重新获取配置文件。
2. 多 agent 如何设计。（不同客户端需要的并不同， 所以应该分开设计） 


以下是使用Go语言结合Kubernetes的`client-go`库，基于上述架构实现的一个简单关键代码示例，包含了Controller和Agent的核心逻辑部分，这里假设使用自定义资源（Custom Resource）来作为命令传递的媒介。

### 1. 定义自定义资源（Custom Resource Definition，CRD）
首先需要定义自定义资源的结构以及相关的配置，示例代码如下（简化示意，实际应用中可能更复杂）：

```go
// 引入必要的库
import (
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    "k8s.io/apimachinery/pkg/runtime/schema"
    "k8s.io/client-go/kubernetes/scheme"
    "k8s.io/client-go/rest"
    "k8s.io/client-go/tools/clientcmd"
    "k8s.io/client-go/tools/record"
    "sigs.k8s.io/controller-runtime/pkg/client"
    "sigs.k8s.io/controller-runtime/pkg/controller"
    "sigs.k8s.io/controller-runtime/pkg/handler"
    "sigs.k8s.io/controller-runtime/pkg/manager"
    "sigs.k8s.io/controller-runtime/pkg/reconcile"
    "sigs.k8s.io/controller-runtime/pkg/source"
)

// 定义命令自定义资源结构
type Command struct {
    metav1.TypeMeta   `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty"`
    Spec              CommandSpec   `json:"spec,omitempty"`
    Status            CommandStatus `json:"status,omitempty"`
}

// 命令规格，包含具体要执行的命令内容等
type CommandSpec struct {
    Command string `json:"command"`
}

// 命令状态，用于记录命令执行情况
type CommandStatus struct {
    Phase   string `json:"phase"`
    Message string `json:"message"`
}

// 定义GroupVersionKind用于注册自定义资源到Kubernetes API
var (
    CommandKind       = "Command"
    CommandAPIVersion = "example.com/v1"
    CommandGVK        = schema.GroupVersionKind{
        Group:   "example.com",
        Version: "v1",
        Kind:    CommandKind,
    }
)

// 注册自定义资源到Scheme，方便客户端与API Server交互
func init() {
    SchemeBuilder := runtime.NewSchemeBuilder(addKnownTypes)
    err := SchemeBuilder.AddToScheme(scheme.Scheme)
    if err!= nil {
        panic(err)
    }
}

// 添加已知类型到Scheme
func addKnownTypes(scheme *runtime.Scheme) error {
    scheme.AddKnownTypes(schema.GroupVersion{Group: "example.com", Version: "v1"},
        &Command{})
    metav1.AddToGroupVersion(scheme, schema.GroupVersion{Group: "example.com", Version: "v1"})
    return nil
}
```

### 2. Controller核心代码

```go
// Controller结构体，管理命令的下发和状态监控等
type CommandController struct {
    client.Client
    recorder record.EventRecorder
}

// 实现Reconcile接口的Reconcile方法，处理自定义资源的变更逻辑
func (r *CommandController) Reconcile(ctx context.Context, req reconcile.Request) (reconcile.Result, error) {
    // 获取对应的命令资源实例
    command := &Command{}
    err := r.Get(ctx, req.NamespacedName, command)
    if err!= nil {
        return reconcile.Result{}, client.IgnoreNotFound(err)
    }

    // 这里可以添加具体的命令处理逻辑，例如解析命令并决定后续操作等
    // 简单示例，先打印命令内容
    log.Printf("Received command: %s", command.Spec.Command)

    // 更新命令状态为已处理（实际应用中根据执行结果更新更准确的状态）
    command.Status.Phase = "Processed"
    command.Status.Message = "Command received and processed"
    err = r.Status().Update(ctx, command)
    if err!= nil {
        return reconcile.Result{}, err
    }

    return reconcile.Result{}, nil
}

// 设置并启动Controller
func SetupController(mgr manager.Manager) error {
    // 创建Controller实例，传入客户端和事件记录器
    controller := &CommandController{
        Client:    mgr.GetClient(),
        recorder:  mgr.GetEventRecorderFor("command-controller"),
    }

    // 创建Controller并注册资源以及对应的处理逻辑
    c, err := controller.NewControllerManagedBy(mgr).
        For(&Command{}).
        Build()
    if err!= nil {
        return err
    }

    // 配置资源的事件源，这里是监听Command资源的所有变更
    err = c.Watch(&source.Kind{Type: &Command{}}, &handler.EnqueueRequestForObject{})
    if err!= nil {
        return err
    }

    return nil
}

// 获取Kubernetes配置并启动Manager，进而启动Controller
func main() {
    // 加载Kubernetes配置，可以是本地kubeconfig文件或者集群内配置
    config, err := clientcmd.BuildConfigFromFlags("", "")
    if err!= nil {
        panic(err)
    }

    // 创建Manager实例
    mgr, err := manager.New(config, manager.Options{})
    if err!= nil {
        panic(err)
    }

    // 注册自定义资源到Manager的Scheme
    err = addKnownTypes(mgr.GetScheme())
    if err!= nil {
        panic(err)
    }

    // 启动Controller
    err = SetupController(mgr)
    if err!= nil {
        panic(err)
    }

    // 启动Manager，开始监听资源变更并处理
    if err := mgr.Start(context.Background()); err!= nil {
        panic(err)
    }
}
```
chiseled 是切割的意思。
所以，

### 3. Agent核心代码

```go
// Agent结构体，用于监听命令并执行
type Agent struct {
    client.Client
    nodeName string
}

// 监听命令的方法，利用Watch机制监听自定义资源的更新
func (a *Agent) ListenForCommands(ctx context.Context) error {
    // 创建一个Watch对象，监听Command资源
    watcher, err := a.Client.Watch(ctx, &metav1.ListOptions{})
    if err!= nil {
        return err
    }
    defer watcher.Stop()

    for event := range watcher.ResultChan() {
        if command, ok := event.Object.(*Command); ok {
            // 简单判断是否是本节点相关的命令（实际可根据更准确的标识判断）
            if command.ObjectMeta.Labels["node"] == a.nodeName {
                // 执行命令逻辑，这里简单打印
                log.Printf("Agent on node %s executing command: %s", a.nodeName, command.Spec.Command)
                // 执行完后可以更新命令状态反馈给Controller（此处省略详细更新代码）
            }
        }
    }

    return nil
}

// 初始化Agent并启动监听
func main() {
    // 获取当前节点名称，可以通过环境变量等方式获取更准确的节点标识
    nodeName := os.Getenv("NODE_NAME")
    if nodeName == "" {
        panic("NODE_NAME environment variable not set")
    }

    // 加载Kubernetes配置，同Controller部分类似
    config, err := clientcmd.BuildConfigFromFlags("", "")
    if err!= nil {
        panic(err)
    }

    // 创建Agent实例，传入客户端和节点名称
    agent := &Agent{
        Client:    client.New(config, client.Options{}),
        nodeName:  nodeName,
    }

    ctx := context.Background()
    // 启动监听命令的逻辑
    err = agent.ListenForCommands(ctx)
    if err!= nil {
        panic(err)
    }
}
```

请注意，以上代码只是一个非常简化的示例，实际应用中还需要完善很多细节，比如：

- 对命令执行结果更准确的反馈，包括处理执行失败情况以及详细错误信息上报给Controller。
- 更好的资源管理，例如连接复用、内存优化等，在Controller和Agent与Kubernetes API Server交互时。
- 更完善的安全机制，如前面提到的TLS加密、RBAC权限控制等代码层面的具体实现。
- 处理各种异常情况以及添加合适的日志记录，便于排查问题等。

但希望这个代码示例能帮助你初步理解基于上述架构在Kubernetes环境下如何实现命令下发与执行的关键流程。 





# 当前无法解析的 winlog 规则 
当前一共有94条规则无法解析， 对于每种 ID 只展示一条对应规则

- HybridConnectionMannager Service Running Detected(对应 channel: ?)
```yaml
logsource:
  product: windows
  service: servicebus
detection:
  selection:
    EventID: 
      - 40300
      - 40301
      - 40302
```
- suspicious AppX Package Digital Signature Detected(对应 channel：System)
```yaml
logsource:
  product: windows
  service: appxpackaging
detection:
  selection:
    EventID: 
      - 157
```
- Suspicious BTTS Transfer Job Download Detected (对应 channel：?)
```yaml
logsource:
    product: windows
    service: bits_client
detection:
    selection:
        EventID: 16403
```
- New BTTS Job Created From Temp Folder Detected 
```yaml
logsource:
    product: windows
    service: bits_client
detection:
    selection:
        EventID: 3
```
- WMI FilterToConsumerBinding Deletion Detected
```yaml
logsource:
    product: windows
    service: wmi
detection:
    selection:
        EventID: 11
```
- Suspicious WMI Command Line Event Consumer Registered Detected
```yaml
logsource:
    product: windows
    service: wmi
    selection:
        EventID: 5861
```
- WMI Persistence Detection
```yaml
logsource:
    product: windows
    service: wmi
detection:
    selection:
        EventID: 
          - 5859
          - 5861
```
- New Process Created Via WMI Call Detected
```yaml
logsource:
    product: windows
    service: wmi
detection:
    selection:
        EventID: 22
```

- Environment Variable Creation via WMI Detected
```yaml
logsource:
    product: windows
    service: wmi
detection:
    selection:
        EventID: 12
```

- Suspicious Creation Via WMI Detected
```yaml
logsource:
    product: windows
    service: wmi
detection:
    selection:
        EventID: 23
```

- CodeIntergrity Blocked Image/Driver Loaded for Policy Violation Detected
```yaml
logsource:
    product: windows
    service: codeintegrity
detection:
    selection:
        EventID: 3077
```
- Blocked Driver Load With Revoke Certificate Detected
```yaml
logsource:
    product: windows
    service: codeintegrity
detection:
    selection:
        EventID: 3023
```

- Revoked Kernel Driver Loaded Detected
```yaml
logsource:
    product: windows
    service: codeintegrity
detection:
    selection:
        EventID: [3021, 3022]
```

- Unmet WHQL Requirements for loaded Kernel module CodeIntegrity Detected
```yaml
logsource:
    product: windows
    service: codeintegrity
detection:
    selection:
        EventID: [3082,3083]

```

- Revoked Certificate Blocked Image Load in Code Integrity Detected
```yaml
logsource:
    product: windows
    service: codeintegrity
detection:
    selection:
        EventID: 3036
```

- Revoked Image Loaded in Code Integrity Detected
```yaml
logsource:
    product: windows
    service: codeintegrity
detection:
    selection:
        EventID: [3032,3035]
```

- Unsigned Image Loaded in Code Integrity Detected
```yaml
logsource:
    product: windows
    service: codeintegrity
detection:
    selection:
        EventID: 3037
```

- Code Integrity - Unmet Signing level Requirements Detected
```yaml
logsource:
    product: windows
    service: codeintegrity
detection:
    selection:
        EventID: [3033,3034]
```

- USB Device Plugged Detected
```yaml 
logsource:
    product: windows
    service: driver_framework
    definition: Requires enabling and collection of the Microsoft-Windows-DriverFrameworks-UserMode/Operrational eventlog
detection:
    selection:
        EventID: [2003,2100,2102]
```

- GALLIUM Activities Detected
```yaml
logsource:
    product: windows
    service: dns_server_analytic
    definition: Requirements Microsoft-Windows-DNS-Server/Analytical 
detection
    selection:
        EventID: 257
```

- Ngrok Usage with Remote Desktop Service Detected(s)
```yaml
logsource:
    product: windows
    service: terminalservices
detection:
    selection:
        EventID: 21
```

- DNS Server Failed toload ServerLeverPluginDLL Detected
```yaml
logsource:
    product: windows
    service: dns_server
detection:
    selection:
        EventID: [150,770,771]
```

- DNS Sone Transfer Failure Detected
```yaml
logsource:
    product: windows
    service: dns_server
detection:
    selection:
        EventID: 6004
```

-  SMB Exploitation Attempt for Potential CVE-2023-23397
```yaml
logsource:
    product: windows
    service: smbclient
detection:
    selection:
        EventID: [30803,30804,30806]
```

- Suspicious Rejected SMB Guest Logon From IP Detected
```yaml
logsource:
    product: windows
    service: smbclient
detection:
    selection:
        EventID: 31017
```

- DNS Query to Ufile.io by DNS Client Detected
```yaml
logsource:
    product: windows
    service: dns_client
    definition: Requrements Microsoft-Windows-DNS Client Events/Operational
detection:
    selection:
        EventID: 3008
```

- Print Spooler Exploitation CVE-2021-1675 detected
```yaml
logsource:
    product: windows
    service: printservice
detection:
    selection:
        EventID: 316
```

- CVE-2021-1675 Prrint Spooler Explotation Detected 
```yaml
logsource:
    product: windows
    service: printservice
detection:
    selection:
        EventID: 808
```
- 到这是 41 个不同 ID
- Deletion of a Rule from Windows Firewall Exception List Detected 
```yaml
logsource:
    product: windows
    service: firewall_as
detection:
    selection:
        EventID: [2006,2052]
```

- New FireWall Rule Added In Windows Firewall Exception List for Potential Suspicious Application Detected
```yaml
logsource:
    product: windows
    service: firewall_as
detection:
    selection:
        EventID: [2004,2071,2097]
```




- Windows Firewall Configuation Rules Deletion Detected


- 94 rules, 




- 有两种需要enrich 的情况， 一个是使用Security 和 OpenSSH/Operational ,需要增加

```yaml
processors:
    - add_tags:
        tags:["abnormalLogon"]
```

- 另一种是使用Microsoft-Windows-TerminalServices-LocalSessionManager/Operational， 需要增加

```yaml
processors:
    - enrich_process:

```

- DNS Sone Transfer Failure Detected
```yaml
logsource:
    product: windows
    service: dns_server
    definition: Requirements Microsoft-Windows-DNS-Server/Analytical
    detection:
    selection:
        EventID: 6004
        DNSQuestionName:42.42.42.42
        DNSQuestionType: 1
        DNSResponseType: 1
        DNSResponseName: 42.42.42.42
        DNSResponseIP: 42.42.42.42
        DNSResponseTTL: 0
```
- 