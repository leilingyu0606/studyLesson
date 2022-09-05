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

价值函数 value function

对Ut计算期望，消除随机性

# 价值学习