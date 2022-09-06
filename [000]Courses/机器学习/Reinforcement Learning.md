# 强化学习

# 基本概念
PDF 概率密度函数p(x) 定义域上和(积分)为1

Ex 期望 E(fx) = Σ fx*px 

随机抽样

agent 智能体 用于执行操作的对象

state s 状态 比如指某一帧的状态

action a 行为 根据state产生的动作

policy Π 策略 记作 Π(a|s)

状态转移 通常认为是随机的，一半来自于环境 p(s'|s,a)

随机性来源：① 通过policy函数随机抽样的action ② 状态转移 S' = p(*|a,s)

reward r 奖励 作出action后给到的reward

return 回报 Ut = Rt + γ * Rt+1 + γ^2 * Rt+2 + ...    γ：折扣率

Ut 的随机性：① 动作a ② 下一个状态s ； for i>=t, Rt ~ Si and Ai => Ut 越大越好

## 动作价值函数 action-value function
对Ut计算期望，消除随机性，对当前动作打分

QΠ(st, at) = E[Ut|st, at]

Q*(st, at) = max(Π) QΠ(st, at) 有了Q* 可以找到让其最大化的a

## 状态价值函数 state-value function
VΠ(st) = EA[QΠ(st, A)] = Σa Π(a|st) * QΠ(st, a)

VΠ 表达了当前情况的好坏，评价Π的好坏


### 环境和安装包
import gym

reset() 重置环境

render() 渲染环境

state,reward,done,info = step(action) 执行动作

一般学习 policy 或者 Q 函数

# 价值学习 Value-based
DQN : Deep Q-network

目标： 获得最高分

最优动作价值函数 Q*(st,at) = max(Π) E[Ut|St=st,At=at]

Q* 给动作a打分，找到分最高的a -> DQN一种价值学习方法 Q(s,a;w) 近似出一个Q*(s,a)

## TD算法（Temporal Difference）
目标 令TD error->0 <- 梯度下降

观测当前st at -> 当前DQN预测 -> 微分 ->  获得新的st+1, rt -> 计算 yt -> 梯度下降

Q(st,at;w) ~= rt + γ * Q(st+1,at+1;w)（通过Ut回报公式推导）

TD target: yt = rt + γ * max(a) Q(st+1,a;wt)

Loss: Lt = 0.5*[Q(st,at;w)-yt]^2

梯度下降 减小loss wt+1 = wt - α * d(Lt)/d(w)|w=wt

# 策略学习 policy-based
神经网路近似策略函数 Π(a|s) -> 策略网络




