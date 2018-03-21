title: MTT用户进桌逻辑--mtt_desk.user_enter()
categories:
  - pyq
  - 工作
date: 2017-07-11 15:49:15
---

## MTT用户进桌逻辑--mtt_desk.user_enter()

### 触发情况
* 增购时，用户不在牌桌内，重新进桌(mtt_desk.addon())
* 重购时，用户不在牌桌内，重新进桌(mtt_desk.rebuy())
* 玩家并桌进来(mtt_desk.send_in_player())
* 用户发起请求进桌(开赛，断线重连)(mtt_game.redirect_user_enter_desk())

<!-- more -->

### 流程

```flow
st=>start: Start
cond1=>condition: 是否原先就在桌子内？
cond2=>condition: 是否旁观？
cond3=>condition: 坐下成功？
op1=>operation: 获取玩家托管状态
op2=>operation: 先保存到旁观组中
op3=>operation: 保存到旁观用户组中
op4=>operation: 从旁观用户组中移除该用户
op5=>operation: 保存用户桌子信息(xuser_tmp)
op6=>operation: 设置玩家托管状态，是否桌内(如果是需要审批的报名,是系统帮他进桌,要设置为False)
op7=>operation: 加入来过该牌局用户列表
sub1=>subroutine: 用户坐下(mtt_desk.user_auto_sit_down()):>/2017/07/11/MTT用户坐下逻辑/#more
e=>end
st->cond1
cond1(yes)->op1->cond2
cond1(no)->op2->cond2
cond2(yes)->op3->op5->op6->op7->e
cond2(no)->sub1->cond3
cond3(yes)->op5->op6->op7->e
cond3(no)->op4->op5->op6->op7->e
```