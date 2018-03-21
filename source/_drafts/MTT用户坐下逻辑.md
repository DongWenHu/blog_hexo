title: MTT用户坐下逻辑--mtt_desk.user_auto_sit_down()
categories:
  - pyq
  - 工作
date: 2017-07-11 17:21:57
---

## MTT用户坐下逻辑--mtt_desk.user_auto_sit_down()

### 触发情况
* 站起时增购(mtt_desk.addon())
* 站起时重购(mtt_desk.rebuy())
* 玩家进桌(mtt_desk.user_enter())

<!-- more -->

### 流程

```flow
st=>start: Start
cond1=>condition: 是否在桌子内？
cond2=>condition: 用户已经是坐下状态？
cond3=>condition: 桌子是否有空位？
cond4=>condition: 是否成功？
cond5=>condition: 是否是新玩家？
op1=>operation: 获取玩家位置状态
(1:小盲与大盲之间 2:庄家与大盲之间 3:小盲位)
op2=>operation: 兑换筹码(携带筹码或初始化筹码)
op3=>operation: 添加用户到用户组并设置座位id
op4=>operation: 设置玩家位置状态
设置用户状态为坐下
清除用户操作超时次数
清除用户动作码
旁观用户中删除该用户
op5=>operation: 更新用户排名
op6=>operation: player_mgr.update用户赛事状态，桌子号，桌子id
op7=>operation: 参赛玩家人数+1
op8=>operation: 更新桌子
e=>end
e2=>end

st->cond1
cond1(yes)->cond2
cond1(no)->e2
cond2(yes)->e2
cond2(no)->cond3
cond3(yes)->op1->op2->op3->cond4
cond3(no)->e
cond4(yes)->op4->op5->op6->cond5
cond4(no)->e
cond5(yes)->op7->op8->e
cond5(no)->op8->e
```