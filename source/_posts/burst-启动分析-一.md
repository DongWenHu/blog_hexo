title: burst 启动分析（一）run方法
categories:
  - pyq
  - 工作
date: 2017-07-13 17:27:15
---

## Burst 启动分析（一）run方法

### 流程分析

<!-- more -->

```flow
st=>start: Start
i=>inputoutput: 启动的环境变量参数
cond1=>condition: 参数为空？
cond2=>condition: 参数的type为proxy？
cond3=>condition: 参数的type为worker？
sub1=>subroutine: 启动master子进程
sub2=>subroutine: 启动proxy子进程
sub3=>subroutine: 启动worker子进程
e=>end
st->i->cond1
cond1(yes)->sub1->e
cond1(no)->cond2
cond2(yes)->sub2->e
cond2(no)->cond3
cond3(yes)->sub3->e
cond3(no)->e
```