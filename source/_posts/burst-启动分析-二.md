title: burst 启动分析（二） master子进程
categories:
  - pyq
  - 工作
date: 2017-07-13 18:23:09
---

## Burst 启动分析（二） master子进程
### 流程分析
<!-- more -->

```flow
st=>start: Start
op1=>operation: 设置进程标题
op2=>operation: 捕获进程信号
终止/强制结束/安全结束: 等待子进程结束再退出
KILL-HUP:重载所有子进程
op3=>operation: 等待proxy进程启动完成
op4=>operation: 连接proxy监听master的服务
(目前该服务暂没任何处理)
op5=>operation: 监听子进程状态
1. 重新拉起挂掉的proxy和workers
2. 替换workers
sub1=>subroutine: 启动proxy子进程
sub2=>subroutine: 启动worker子进程
e=>end

st->op1->op2->sub1->op3->sub2->op4->op5->e
```
