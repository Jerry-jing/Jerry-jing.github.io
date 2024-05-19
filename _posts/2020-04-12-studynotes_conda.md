---
title: Conda 学习笔记
date: 2020-04-12
categories: [科研]
tags: [学习笔记, 技术, Python]
img_path: /assets/img/
---



Conda 几乎是深度学习领域必备的工具，以后做科研一定会用到。今天就系统学习一下。


Conda 官方文档：<https://docs.conda.io>


## 一、Conda相关概念

我觉得很多初学者分不清 Conda、Anaconda 等之间的区别，这里就先把概念整理明白吧。

![conda](Conda_vs_Miniconda_vs_Anaconda.png){:w='500'}

Conda 是一个包管理工具，最主要的功能是方便管理一个计算机里安装的不同环境。使用的场景举例：
- 写某代码需要尝试不同版本的 Python 或库；
- 某代码需要使用新版本的 Python 或库，但不想更新掉旧版本，因为自己有些别的代码是依赖旧版本的；
- 有的大佬身兼数职，既搞深度学习，又搞前端开发，他想独立地管理这两个不太相关的领域的库；
- 运行陌生的代码，想单独找个环境，用完即删除；等等。

对于人工智能领域的程序员，Conda 几乎是必备的，因为大部分代码是用 Python 写的，而且代码并不是从头实现，需要用到各种不同的人、公司开发的库，且这些库版本迭代非常快，如 TensorFlow、PyTorch 等，这时包管理就显得非常重要了。

Anaconda 和 Miniconda 简言之是安装 Python + Conda 环境的两种方式，即只要装了它，就相当于把 Python 和 Conda 都装好了。Miniconda 是最精简版本，几乎只有 Conda；Anaconda 更像懒人包，把大量常用的库预装好了（类似于《上古卷轴5》一些贴吧大神做的 Mod 整合包），包管理还有图形界面，运作更加商业化。一般来说，就装 Miniconda 即可，更不容易被花里胡哨的东西迷惑双眼。需要用到什么包自己手动安装，而不是用 Anaconda 懒人包里的东西，这样对代码的理解可能更深。

在 Linux/Mac 系统里，Python 和 Conda 都是在终端运行的应用程序，即可以在终端敲 `python` `conda`，后面跟一系列子命令运行的。


## 二、安装后

安装过程不再叙述，基本就是“下一步、下一步、下一步、完成”。还要强调，装了 Anaconda / Miniconda 就是装了 Python，没必要单独去官网装一个 Python 了。如果安装前就有一个 Python，官方文档说了，卸不卸载随意，只要能分的清。

以 Miniconda 在 Mac 系统为例，安装完毕后，所有 Python （包括解释器 `python.app` ）和 Conda 的文件全部默认都在用户目录 `~/Miniconda3/` 里面，管理的包也都在这里面。这个文件夹的文件是怎么组织的参见[文档](https://docs.conda.io/projects/conda/en/latest/user-guide/concepts/environments.html#conda-directory-structure)。


安装完毕后，Conda 会创建一个默认的环境 `base`，Mac 系统的命令行左边会出现一个 `(base)` 前缀，表示当前处在此环境中。`conda` 和 `python` 都已配置到环境变量中。

如何在指定的环境运行 Python 代码？
- 在终端中，激活此环境（让左边的括号变成此环境，见下文），运行 py 文件；
- 在 IDE 如 VSCode 中，找一找，图形界面里总有地方可以选择（指定了环境的）解释器的，选择后运行即可。

> 请注意，Conda 中多个环境可共用一个 Python 解释器，因此运行 Python 代码需要指定环境而不是解释器。
{: .prompt-warning }

## 三、Conda 必备操作


Conda 作为一个包管理工具，最主要的逻辑就是一个两层的关系：上一层为环境，下一层为该环境里安装的包。所有必备操作（指实现此软件核心功能必须有的操作）都是围绕这两层进行的：

- 对环境的操作
    - 检索：`conda info --envs`
    - 创建：`conda create --name NAME python=x.x`
      - 复制：`conda create --name NAME --clone XXX`
    - 切换：`conda activate NAME` (`conda deactivate` 等效于 `conda activate base`)
    - 删除：`conda remove --name NAME --all`
- 对包的操作
    - 检索：`conda list`
    - 安装：`conda install PACKAGE_NAME`
    - 卸载：`conda remove PACKAGE_NAME`



## 四、Conda 进阶操作

这些操作可能是非必需的，但能在效率上锦上添花。

- 对环境的操作
  - 复制：在 `conda create` 命令加选项 `--clone NAME`
- 其他全局操作
  - 增加下载源：`conda config --add channels` 加 URL。在默认的境外下载源抽风时使用，推荐国内清华大学 TUNA 开源镜像。


其他待更新。
