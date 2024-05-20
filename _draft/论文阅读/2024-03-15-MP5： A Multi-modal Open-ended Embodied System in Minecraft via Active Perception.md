---
title: MP5： A Multi-modal Open-ended Embodied System in Minecraft via Active Perception
author: Jing
date: 2024-03-15
tags:
  - CVPR2024
  - Embodied_AI
  - 泛读
---
[论文](https://arxiv.org/pdf/2312.07472.pdf)
[作者知乎](https://zhuanlan.zhihu.com/p/686000830)
和朱松纯老师的讲座不谋而合
# 1. 领域
具身智能
# 2. 应用
通往真正强人工智能的必经之路。
它的应用十分广泛：机器人技术、自动驾驶汽车等。
它使用到了多种技术：深度学习、大语言模型（LLM）、视觉传感器等
# 3. motivation
- 具身智能这项任务希望，用**类似人类的方式**解决长期开放世界任务的具体系统。
- 问题：现有方法会遇到有任务引起的 `logic-aware decomposition` 和 `context-aware execution` 的复合困难
- 解决的任务分为两种：`process dependency` 和 `context dependency`
# 4. 论文提出的方案
- 论文解决方案：提出 MP5，一个 open-ended multimodal embodied system 
- 好处：`decompose feasible sub-objectives, design sophisticated context-aware executionsituation-aware plans, perform embodied action control, with frequent communication with a goal-conditioned active perception scheme.`
- 创新点：提出了MP5—— a novel embodied system developed within Minecraft
- MP5包括 5 个交互模块：
	- Parser：将 long-horizon task 分解为一系列子问题
	- Percipient：回答有关观测到的图像的各种问题（是LoRA-enabled Multimodal LLM）
	- Planner：根据当前情况安排子目标行动顺序，并细化子目标
	- Performer：执行动作并于环境互动
	- Patroller：检查 Percipient、Planner、Performer 的响应，验证当前的计划 / 行动，或者针对潜在的更好的策略进行反馈
# 5. 效果
MP5 能够鲁棒地完成 long-horizon reasoning 和 complex context understanding 
在 diamond-level 的任务上完成 22% 的胜率，在需要 complex scene understanding 的任务中达到 91% 的胜率
# 6. 前人做到的
- 大型语言模型通过将任务分解为一些列可行的子任务，来解决 process-dependent 的问题。
	- 缺点：假设 agent 是 all-seeing 的（知道所有他们的状态和所处的环境），简化了 context-dependent challenge
	- 解决 context-dependent challenge 应该：*感知能力是开放式的、有选择性的，并给出针对不同目的定制的结果（例如任务规划和行动执行）；感知模块可以与其他模块一起兼容地调度（例如，规划和执行模块）通过统一的接口，作为一个集成的系统*
# 7. 本文工作
专门为 Minecraft 设计和训练了 MineLLM，Parser、Percipient、Planner、Performer、Patroller都是大语言模型
# 8. 感想
对我来讲很新颖（但是在 Minecraft 上做 embodied agent 已经非常常见了，看到从 2022 年就开始做这样的工作），在 Minecraft 游戏作为 embodied agent 的主体环境，搭建了一个 embodied system ，与听到的朱松纯老师做的“通通”具身智能思路相同。
但是总觉得它还有很多缺陷：Minecraft 毕竟只是一款游戏，放到现实世界中是否会有 domain adaptation 的问题；所有的部分只用 LLM 会不会忽略一些其他的信息；完成的任务单线程而且比较单一，而且准确率不是很高，不像人灵活；LLM 耗费资源很多，不像人脑比较节能。

Generalist agents : 通用智能体
