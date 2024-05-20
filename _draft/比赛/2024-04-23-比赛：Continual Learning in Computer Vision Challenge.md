---
title: Continual Learning in Computer Vision Challenge
author: Jing
date: 2024-04-23
---
项目地址：[5th CLVISION CVPR Workshop - Challenge (google.com)](https://sites.google.com/view/clvision2024/challenge)
codalab 地址：[CodaLab - Competition (upsaclay.fr)](https://codalab.lisn.upsaclay.fr/competitions/17780#learn_the_details-overview)
baseline 地址：[ContinualAI/clvision-challenge-2024：CVPR 的第 5 届 CLVISION 研讨会：挑战的回购 (github.com)](https://github.com/ContinualAI/clvision-challenge-2024)
# 1. 比赛概述
- 两个问题：==持续学习==和==无标签数据使用==
- 目标：通过利用无标签数据来解决重复类增量问题（CIR）
- 重复类增量（CIR）包括各种情况，主要有两个特点：
	- ==以前观察到的类别可以以任意的重复模式重新出现在 new experience 中==
	- ==不是每个 experence 都有所有类别==
- 每个 scenario 被分为 50 个 experiences，在每个 experiences 中，我们都有一个训练环节，可以使用一组有标签和无标签的样本。在此期间，我们可以使用标注数据训练、更新或调整我们的模型，同时使用未标注数据提供支持。
- 一旦训练结束，有标签和无标签的数据都将不可用。
- 根据不同的 scenario，在未来的 experiences 中，我们将有可能获得已见类、未见类或干扰类的样本。
- *干扰类是？*
- 官方提出 3 个 scenarios 来测试参与者所制定策略的稳健性，每个 scenarios 都会出现一个标记流数据和一个未标记流数据
	- 未标记流数据中的样本可能属于：和标记流数据一样的类别；所有标注类别的样本（包括未看到的标注类）；干扰类既不存在于标记流数据中，也不存在于评估集中
- 我们要基于 DevKit 制定*策略*（改变数据加载过程和于比赛相关的 modules 都是不允许的），在模型完成对整个经验流的训练后，在评估测试集上达到较高的平均准确率，该测试集包含来自 LS 的未见样本（均衡数量）。
- 每个 scenario 都是相同算法。
# 2. 指标
最终准确率，在 evaluation test set 中评估
没有干扰类
convergence rate of accuracy over experiences、
# 3. Scenarios
- 包含 3 个 scenarios ，基于 ImageNet ，具有固定类数的计算机视觉数据集。
- 每个 scenarios 由 50 个 experiences 组成，由 500 张标记图像和 1000 张未标记图像。
	- experiences 个数：50
	- 每个 experiences 的 labelled samples 个数：500
	- 每个 experiences 的 unlabelled samples 个数：1000
	- ==Samples are equally balanced among present classes in each experience.==
# 4. 限制
- 模型：必须使用 ResNet 18 作为模型的基本体系，允许添加额外模块，例如门控模块，不能超过比赛允许的最大 GPU 内存，不能使用预训练权重初始化。
- 缓冲区大小（即存储的示例总数）都不应超过 200；缓冲区不得用于存储数据集图像，可以存储其他形式的数据表示或者统计数据（如模型的内部表示）
- GPU 不能超过 8000 M
	- 正常 ResNet18 的 max gpu memory allocated 为 3000 - 3100 M
# 5. 时间
预选阶段： 2024 年 5 月 6 日
# 6. 数据集详细情况
train：168128 张
test：6500 张
一共 130 个类别，但是只有 ==100 个类别（以标记或未标记的形式出现）最终放到 test 中==。场景 3 中，剩余的 30 类作未标记数据出现，但是在最终的场景中，可学习类和干扰类的数量可能会不同。
# 7. 结果
.pkl 
```python 
dict_keys(['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24', '25', '26', '27', '28', '29', '30', '31', '32', '33', '34', '35', '36', '37', '38', '39', '40', '41', '42', '43', '44', '45', '46', '47', '48', '49'])
```
每个键中有 6500 个值
可能有 99 个类
# 7. 往届优秀
[第四届CLVISION CVPR研讨会 - 挑战赛结果 (google.com)](https://sites.google.com/view/clvision2023/challenge/challenge-results)
## 7.1 第一名
>主要提点：momentum-based approach、test time augmentation
>方法结合了 Hard Attention to the Task 和 Supervised Contrasive Learning 的优点；使用可训练的硬掩码对网络进行分区，掩码于 experience ID 对齐；引入了 HAT（hard attention to the task）；整个网络是两阶段训练的：第一阶段表示学习，用对比监督损失；第二阶段分类学习，用分类损失；测试时加入 tta（预测全部 100 个类），最终结果是对不同的 experiences 进行加权平均；
- model Implementation
	- HAT with modifications：
		- Each task corresponds to an experience.
		- Initialization and scaling improvement.
		- ==Overcoming catastrophic forgetting with hard attention to the task.== 论文中
- two-stage training
	- Contastive supervised learning（SupCon）
	- Classification（on the classes of the current experience）
		- All replay samples are marked as a single OOD class.（out of distribution）
		- > 在经验回放（Experience Repaly）策略中，所有用于辅助模型保持对旧类记忆的未标记数据样本都被标记未同一个”超出范围“的类别。可以确保模型不会将这些样本错误地分类为旧类中的任何一个类别
		- ![[Pasted image 20240425150341.png]]
	- ==Supervised contrastive learning.==
- prediction
	- Given an experience（ID）, the model can only generate the logits of the classes from that experience
	- Logits from recent experiences to cover all classes
		- Momentum using logits from older experiences
		- *We adopt a momentum-based approach that utilizes the last three experiences for every logic of a  class.*
	- Test time augmentation
- replicas
	- Fragments: each model is in charge of a chunk of experiences.
	- Ensemble: multiple models for the same chunk of experiences.
	- ![[cl_replicas.png]]
## 7.2 第二名
> 提出了于集成学习策略相结合的参数隔离方法；对分类器进行调整，*保持模型在不同训练阶段获得的能量水平相同*；用自监督学习来捕捉更一般的表征；建立一个共享提示库，促进不同任务和类别之间的互动，促进知识融合。





## 7.3 第三名
> 在选定的 experiences 上学习一组特征提取器；启发式方法确定是否要在本次 experience 中学习新的 FE；

# 8. 持续学习中的概念
- Replay Embeddings：回放嵌入
	- 在回放嵌入中，除了原始的分类头之外，还引入了一个额外的”超出经验类别“ OOD 的 logit 头
- energy level：
	- 在 machine learning 中，energy level 通常指模型预测的置信度或确定性










更新了  pickle 配置
问题：
- scenaio_1.pkl 里面放的什么？定义一个增量学习场景，以便在训练过程中逐渐增加新的类别。
	- 每个 experience 有 500 个 labelled samples
- 每一个 scenaio 里面都有 50 个 experience，一个 experience 里面有一个 train dataset、一个 unlabeled dataset
- stream length : number of experiences in the stream
- ==没有标签数据的作用==：
	- 知识蒸馏：使用旧模型对未标记数据进行预测，然后将这些预测作为新模型的目标，以帮助新模型记住旧知识
	- 经验回放（Experience Replay）：将旧类的未标记数据保存到经验回放池中，并在每个新的训练阶段中定期重新引入这些数据，以帮助模型保持对旧类的记忆
	- 模型增强（Model Augmentation）：使用旧类的未标记数据对模型进行增强，以提高模型的泛化能力
	- 元学习（Meta-Learning）：在未标记数据上进行元学习，以优化模型的学习过程，使其能够在增量学习中更好地适应新类别。
	- 模型整合（Model Fusion）：将旧模型和新模型的输出结合，以利用旧模型对未标记数据的预测
	- 在比赛中，每个经验都提供了一组标记和未标记的数据。在训练阶段，你可以使用标记数据来更新或适应模型，同时使用未标记数据来辅助模型保持对旧类的记忆。一旦训练阶段结束，标记和未标记数据都不可访问，模型必须依靠其在先前经验中学到的知识来处理未来的经验。
	- 由于未标记数据可能包含新类别的样本，因此在处理未标记数据时需要谨慎，以确保模型不会错误地学习或记忆这些新类别。通常，未标记数据的使用是有限的，并且只在特定的经验中进行。
- 三个场景
	- 场景1： LS 和 US 在每个 experience 中都有相同的类
	- 场景2：US 包含与 LS 相同的类，以及整个 LS 中过去或未来的类
	- 场景3：US 包含与 LS 相同的类，以及整个 LS 中过去或未来的类，以及 LS 中不存在干扰类


- *干扰类？*
	- distractor classes 是不需要被学习或分类的类别
- *test 中是否有 train 没有的类别？超出 label 范围的数据有没有？*
	- 
- *unlabel 的数据是否是 label 数据的子集？*
	- 是的，train.txt 中有 168128 张图片，test.txt 中有 6500 张图片，总共 .JPEG 图片 174628 张
	- 代码里面 unlabelled dataset 也是 labelled dataset 的子集

==Class Incremental Learning== With ==Few-Shots== Based on Linear Programming for Hyperspectral Image Classification
==Jing Bai==
IEEE Transactions on Cybernetics 2020

Incremental Land Cover Classification via Label Strategy and Adaptive Weights
Bo Ren; Zhao Wang; ==Biao Hou==
IEEE Transactions on Geoscience and Remote Sensing 2023

Hyperspectral Image Classification Using Geometric Spatial–Spectral Feature Integration: A Class Incremental Learning Approach
==Jing Bai==
IEEE Transactions on Geoscience and Remote Sensing 2023

Incremental laplacian regularization extreme learning machine for online learning
Lixia Yang, ==Shuyuan Yang==
Applied Soft Computing 2017

Incremental Semi-Supervised classification of data streams via self-representative selection
Zhixi Feng, Min Wang, ==Shuyuan Yang==
Applied Soft Computing 2016

Continual learning for cross-modal image-text retrieval based on domain-selective attention
Rui Yang, Shuang Wang, Yu Gu, Jihui Wang, Yingzhi Sun, Huan Zhang, Yu Liao, Licheng Jiao
==Pattern Recognition 2024==

Lightweight Hyperspectral Image Classification with Lifelong Learning
Anran Yuan, ==Jing Bai==, Zhu Xiao, Zhongrui Wang, Jianqing Li, Licheng Jiao

Knowledge Decomposition and Replay: A Novel Cross-modal Image-Text Retrieval Continual Learning Method
Rui Yang, Shuang Wang, Huan Zhang, Siyuan Xu, YanHe Guo, Xiutiao Ye, Biao Hou, Licheng Jiao
ACMMM 2023

Deep Continuous Matching Network for more Robust Multi-Modal Remote Sensing Image Patch Matching
Rufan Zhou; Dou Quan; Chonghua Lv; Yanhe Guo; Shuang Wang; Yu Gu; Licheng Jiao
IGARSS 2023



数据集知识蒸馏
A Category-Aware Curriculum Learning for Data-Free Knowledge Distillation
Xiufang Li; Licheng Jiao
IEEE Transactions on Multimedia 

广义少样本学习和持续学习之间的关联
无数据知识蒸馏
D3K: Dynastic Data-Free Knowledge Distillation
Xiufang Li; Qigong Sun; Licheng Jiao
IEEE Transactions on Multimedia 2023

