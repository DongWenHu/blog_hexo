title: burst 启动分析（三） proxy子进程
categories:
  - pyq
  - 工作
date: 2017-07-13 18:44:18
---

## Burst 启动分析（三） proxy子进程

### 流程分析
<!-- more -->

```flow
st=>start: Start
op1=>operation: 设置进程标题
op2=>operation: 捕获进程信号
终止/强制结束/安全结束: stop所有监听的reactor
KILL-HUP:忽略
op3=>operation: 启动监听master，listenUNIX
暂时对客户端无任何处理

sub1=>subroutine: 启动监听worker，listenUNIX
(burst worker进程会与之建立连接)
sub2=>subroutine: 启动对外监听，logice server，listenTCP
(就是每个logicServer的服务)
sub3=>subroutine: 启动admin服务，listenUNIX/listenTCP
e=>end

st->op1->op2->op3->sub1->sub2->sub3->e
```

### 各个server 时序分析

![](/img/pyq_sequence.png)

```sequence
participant 用户
participant gateway
participant 客户端(maple worker)
participant 对外监听(logic server)
participant 监听worker
participant 客户端(burst worker)

客户端(burst worker)->监听worker: 建立连接
Note over 监听worker: 放到空闲worker列表里去
Note over 监听worker: 尝试从任务队列去申请task

用户->gateway: 发送请求
gateway->客户端(maple worker): 转发请求
客户端(maple worker)->对外监听(logic server): 取模分发请求
客户端(maple worker)->gateway: 向gateway索取任务
Note over 对外监听(logic server): 添加任务,\n当新消息来得时候，应该先检查有没有空闲的worker，\n如果没有的话，才放入消息队列

对外监听(logic server)->监听worker: 有空闲worker发送任务
监听worker->客户端(burst worker): 发送任务
Note over 客户端(burst worker): 处理任务(bp路由)

客户端(burst worker)-->监听worker: 任务处理返回(write_to_worker,write_to_client)

监听worker-->对外监听(logic server): 任务处理返回(write_to_worker,write_to_client)
对外监听(logic server)-->客户端(maple worker): 任务处理返回(write_to_worker,write_to_client)
客户端(maple worker)-->gateway: 任务处理返回(write_to_worker,write_to_client)
gateway-->用户: 任务处理返回(write_to_client)
```