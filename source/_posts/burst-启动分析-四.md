title: burst 启动分析（四） worker子进程
categories:
  - pyq
  - 工作
date: 2017-07-14 16:16:21
---

## Burst 启动分析（四） worker子进程

### 流程分析
<!-- more -->

```flow
st=>start: Start
con1=>condition: 执行任务出现异常？
op1=>operation: 设置进程标题
op2=>operation: 捕获进程信号
终止/强制结束/安全结束: 不做处理
KILL-HUP:不做处理
op3=>operation: create_worker
create_app_worker
op4=>operation: 连接proxy监听worker的服务
op04=>operation: create_conn
create_app_conn
op041=>operation: 启动监控task的耗时线程
默认30s，超时强制从子线程退出worker
op5=>operation: 等待读取任务消息
op6=>operation: before_first_request
before_app_first_request
before_app_request
before_request
op7=>operation: 根据bp路由cmd执行对应任务函数
op8=>operation: 返回的错误码为-10001
op9=>operation: before_response
before_app_response
op10=>operation: write给worker server
op11=>operation: after_app_response
after_response
op12=>operation: after_request
after_app_request
e=>end

st->op1->op2->op2->op3->op4->op04->op041->op5->op6->op7->con1
con1(yes)->op8->op9->op10->op11->op12->op5
con1(no)->op9->op10->op11->op12->op5
```