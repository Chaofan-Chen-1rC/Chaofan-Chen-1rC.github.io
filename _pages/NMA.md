---
title: "Network Meta-analysis1"
date: 2026-04-17
permalink: /reading-notes/dmar-note-12/
tags:
  - meta-analysis
  - R
  - reading notes
  - doing-meta-analysis-in-r
---


#meta 111
- Doing meta analysis with R 
- netmeta文章
- 贝叶斯方法

[12 Network Meta-Analysis](https://doing-meta.guide/netwma#netwma) Source from: [Doing Meta-Analysis in R](https://doing-meta.guide/)

- 第12章系统介绍了Network meta analysis 的核心概念与 R 语言实现。内容涵盖：
	1. 直接与间接证据的整合、
	2. 可传递性与一致性假设，
	3. 详细演示了如何通过 netmeta 包进行频率网络元分析，
	4. 以及利用 gemtc 包完成贝叶斯网络元分析。
- 章节重点包括：
	1. 网络模型构建、
	2. 异质性与不一致性诊断、
	3. 治疗排序（P值/SUCRA）
	4. 结果可视化，
	5. 并拓展至网络元回归分析，

==该方法学为综合比较多种干预措施效果提供全面方法学指导。==

## Introduction

**对临床试验或其他类型的干预研究，通常旨在评估某一特定治疗方法的真实效应量**，纳入的研究均是将同类型干预措施与相似的对照组（例如安慰剂组），来判断该特定治疗方法是否有效。

**但许多领域不是只存在一种“决定性的”治疗方案**，而是存在多种治疗方法，如偏头痛有多种药物和非药物治疗。特别一些成熟的研究领域，**证明疗法有效不再那么重要**，我们更希望**找出针对某种特定适应症的最有效疗法**。

这就引发了新的问题。要在**传统元分析**中评估多种治疗的相对有效性，需要有足够多的两种治疗之间的直接对比研究。但实际常缺少此类直接比较，大多研究是与“效力较弱”对照组比较。因此，**传统元分析无法就多种治疗相对有效性建立确凿证据**。

不过，尽管可能缺乏两个或多个治疗方案之间比较的**直接证据**，但**间接证据**通常是存在的。不同的治疗方案的研究可能都使用了**相同的对照组**。

>[!info] 例：
>A VS. C 和 B VS. C，A 和 B 均使用了相同的对照组 C，A 和 B 就存在了间接比较。

**网络元分析**可用于纳入此类间接比较，它可以**将多个直接和间接的治疗比较整合到一个统一的模型中**，从而使我们能够同时比较多种干预措施的效果[^1]。网络元分析也被称为**混合治疗比较元分析**（Valkenhoef 等，2012）。过去十年间，该方法在生物医学及其他学科的应用研究中得到日益广泛的应用。

然而，这一方法也面临额外挑战与潜在风险，尤其在**异质性（heterogeneity）**及所谓网络不一致性（network inconsistency）方面（Salanti等，2014）。**

==**因此，首先讨论其核心组成部分和假设十分重要，下面我们将分步阐述基本要点以便理解。**==

> [!faq] this is
> 当我们对临床试验或其他类型的**干预研究**进行荟萃分析时，通常会估算某一特定治疗方法的实际效果大小。我们会纳入那些将相同类型的干预措施与类似对照组（例如安慰剂）进行比较的研究。在其他条件相同的情况下，**这有助于评估某一特定类型的治疗方法是否有效**。
> 
> 但现实情况是，很多领域，例如偏头痛的治疗，干预方式不止一种，在一些成熟的领域，一种治疗有没有效可能没有那么重要了，相反，重要的是，我们想找出一种针对特定结局最佳的干预方式。


## 12.1 What Are Network Meta-Analyses?
### **12.1.1** **直接证据与间接证据**

首先，我们需要理解什么是治疗的“网络”。假设我们从某项随机对照研究（RCT）中提取了数据，该研究比较了治疗A与另一种条件B（如等待名单对照组）的效果。我们可以用图形来表示这种比较：
![[Pasted image 20251114093044.png]]

我们的图有两个核心组成部分。
- 第一个是两个圆圈（称为节点），代表试验i中的两种条件A和B。
- 第二个组件是连接这两个节点的线，称为边。边表示A和B之间的关系。我们可以用效应量$\hat{\theta}_{i,A,B}$ 来描述A和B之间的关系。

现在，假设我们还从另一项研究j中获得了数据。这项研究也使用了对照组B与另一种治疗C。在研究j中，治疗C也与B进行了比较。我们可以将这一信息添加到我们的图中：
![[Pasted image 20251114093522.png]]

我们创建了第一个小网络，图中包含两个在“真实”试验里直接观察到的效应量估计值：$\hat{\theta}_{i,A,B}$（比较A与B）和$\hat{\theta}_{j,C,B}$（比较C与B）。这类信息叫**直接证据**，用 $\hat{\theta}^{direct}_{B,A}$ 和 $\hat{\theta}^{direct}_{B,C}$ 表示，B是参考组，条件B在符号中排在前面，因为两项研究都以它为对照组。

在新图中，所有节点（条件）要么直接连接，要么间接连接。条件B（对照组）与所有其他节点**直接连接**。从B到其他两个节点A和C只需“一步”：B→A，B→C。相反，A和C之间并无直接连接，它们都连接到B：A→B和C→B。

但是A与C之间存在一条**间接连接**。因为B充当了连接这两个条件的纽带或桥梁：A → B → C。因此，存在关于A和C之间关系的**间接证据**，可以从网络结构中推导出来：
![[Pasted image 20251114094108.png]]
使用直接观察到的边的信息，我们可以计算A和C之间**未直接观察到的间接比较的效应量**，用 $\hat{\theta}^{indirect}_{A,C}$表示这个未观察到的间接效应量。该效应量估计可以使用以下公式计算（Dias等，2018，第1章）：
$\begin{equation} \hat\theta_{\text{A,C}}^{\text{indirect}} = \hat\theta_{\text{B,A}}^{\text{direct}} - \hat\theta_{\text{B,C}}^{\text{direct}} \tag{12.1} \end{equation}$


**网络元分析的核心在于将直接证据与间接证据同时整合在同一个模型中**，借此信息可估计每种纳入治疗的（相对）效应。通过添加间接证据，即使对于该特定比较存在直接证据，我们也可以提高效应量估计的精确度。总体而言，**网状元分析具有以下几个好处**：

1. 能将一组相关研究的全部信息整合到一个分析里，而传统元分析需把每个比较分别做独立的元分析。

2. 可纳入传统元分析无法纳入的间接证据（通过间接与共有的对照组进行比较）成对元分析只能整合试验中实际比较的直接证据。

3. 若假设成立且结果明确，网络元分析能推断出目标人群的**首选治疗类型（治疗排序）。**

**网络元分析中需注意的问题：**

- 首先，**间接效应量估计值的方差计算**，采用如下公式。要计算间接比较的方差，我们将直接比较的方差相加。基于间接证据估计出的效应量，其方差总是大于基于直接证据估计的效应量，因而精确度也较低（[Dias et al. 2018, chap. 1](https://doing-meta.guide/references#ref-dias2018network)）。

$\begin{equation} \text{Var} \left(\hat\theta_{\text{A,C}}^{\text{indirect}} \right) = \text{Var} \left(\hat\theta_{\text{B,A}}^{\text{direct}} \right) + \text{Var} \left(\hat\theta_{\text{B,C}}^{\text{direct}} \right) \tag{12.2} \end{equation}$


- 通过直接证据计算出的间接证据的效应量值的公式（公式12.1），只有在满足**可传递性假设（transitivity）**时（从统计学的角度来看，这一假设等同于**网络一致性（network consistency）**（Efthimiou et al. 2016）)，才成立。

==下面，我们将解释这两个术语的含义以及它们为何重要。==

### 12.1.2 可传递性 & 一致性[](https://doing-meta.guide/netwma#transitivity-consistency)

网状元分析无疑是标准元分析方法的有价值的扩展，但其有效性仍有争议。对网状元分析的大多数批评集中在间接证据的使用上（Edwards等，2009；Ioannidis，2006），尤其是在存在直接证据的情况下。

关键问题在于，随机试验中参与者被随机分配到治疗组（如A或B），但网络元分析中的试验条件（治疗组合）本身并非随机选择。这本身不会导致网络元分析模型产生偏倚（Dias等，2018）。只有当某一特定比较在试验中的选择或未被选择依赖于该比较的真实效应时，模型才会出现偏倚（Dias等，2013）。以下展开具体说明。

**可传递性假设**的核心原则是，我们可以合并直接证据（例如， A vs. B 和 C vs. B 的证据）来生成关于一个相关比较（例如 A vs. C）的间接证据。其与可交换性概念相关。

**可交换性假设**是指任一比较项 i 的真实效应量 $\theta_i$，均是从一个"总体 "真实效应量分布中随机且独立抽取的结果，即只要是比较A 和 B 的研究，所得出的结果全部是从一个“总体”的真实效应量分布中随机独立抽取的结果，是可交换的。

在网状元分析里，关键假设是无论比较是否被实际评估或该比较是否“缺失，其效应都是可交换的。
因为原始研究中的干预选择永远不可能是随机的，因此也不可能在一个实验中纳入所有的研究方法，干预总是特定选择的，只要这个选择不是因为特定干预有或没有效果而被选择，就不会出现偏倚。

关键假设在于，比如 A - B 这种比较的效果在各试验间具有可交换性——无论试验是否实际评估了这种比较，还是“缺失”。在网状荟萃分析中，当某个比较 i 的效应估计值 $\hat\theta_i$ 是从总体真实效应分布中随机独立抽取时，就满足可交换性，无论该效应量是通过直接证据还是间接证据得出的。

当协变量或效应修饰因素在相关研究中分布不均时，可传递性假设可能被违反（Song等，2009）。**它无法用统计方法检验，但可通过纳入人群、方法和目标条件相似的研究降低风险（Salanti等，2014）。** 可传递性的统计表现是一致性，不一致性是其对立面（Efthimiou等，2016；Cipriani等，2013）。

**一致性**是指基于直接证据得到的某个比较（例如 A vs. B）的相对效应，与基于间接证据得到的该比较的相对效应之间**没有差异**（Schwarzer, Carpenter, and Rücker 2015, chap. 8）

$\begin{equation} \theta_{\text{A,B}}^{\text{indirect}} = \theta_{\text{A,B}}^{\text{direct}} \tag{12.3} \end{equation}$

常采用**网络热图（net heat plots）** 和 **节点拆分法（node splitting）** 来诊断，我们将在下面的部分中更详细地描述这些方法。

==所以：==
1. 检验可传递性（通过纳入人群，例如平均年龄，性别等特征、方法和目标条件相似的研究降低风险）
2. 检验一致性（直接比较和间接比较之间的相对效应是否存在差异）

### 12.1.3 Network Meta-Analysis 模型[](https://doing-meta.guide/netwma#netw-which-model)

到这里，我们已经介绍了网络元分析模型基本理论与假设。此前，我们借助一个仅有三个节点和边的简单网络来辅助说明。但在实际应用中，网状元分析纳入的治疗数量往往更多，这会使网络迅速变得更为复杂，如下面例子所示：
![[Pasted image 20251114102052.png]]

随着网络中治疗数量S的增加，我们需要估计的（直接和间接）成对比较数量C会急剧上升：
![[Pasted image 20251114102308.png]]

因此，为高效且内部一致地汇总网状元分析中的所有可用网络数据，我们需要一种计算模型。目前已开发出多种用于网络元分析的统计方法（Efthimiou等，2016），**后续我们将讨论基于频率论模型和贝叶斯多层次模型及其在R中的实现。**

>[!info] **我应该使用哪种建模方法？**
>尽管网络元分析模型的统计方法存在差异，但好消息是当样本量足够大时，都应产生相同的结果（Shim等，2019）。一般而言，没有哪种网状元分析方法绝对更优，我们可根据方法是否直观，或实现该方法的R包功能来选择（Efthimiou等，2016）。  
>在大多数学科中，基于频率论推断的方法仍比贝叶斯方法更常见，部分人可能**更易理解由频率论模型得出的结果**。不过，**R中实现频率论网络元分析的方法（后文介绍）不支持元回归**，而贝叶斯模型支持。  
>==实践中，一种有效策略是：主要分析选一种方法，敏感性分析用另一种方法。若二者结论一致，则可增强我们对研究结果可信度的信心。==

## 12.2 Frequentist Network Meta-Analysis[](https://doing-meta.guide/netwma#frequentist-ma)

接下来，我们将介绍如何使用 `{netmeta}` 包进行 Network Meta-Analysis（NMA）（Rücker 等人，2020 年）。该包能够在频率学框架内估计 NMA 模型。 `{netmeta}` 所采用的方法源自图论技术，这些技术最初是为电气网络而开发的（Rücker，2012 年）。

>[!info] 概率的频率主义解释
>频率学是一种常见的用于解释某一事件 E 的概率的理论方法。频率学方法通过描述在我们多次重复某个过程（例如实验）的情况下事件 E 预期出现的频率来定义事件 E 的概率（阿罗诺和米勒 2019 年，第 1.1.1 章）。频率学的理念是许多定量研究者日常所采用的统计方法的核心所在，比如显著性检验、置信区间的计算或者 p 值的确定等。

### 12.2.1 图论模型[](https://doing-meta.guide/netwma#the-graph-theoretical-model)

下面介绍`netmeta`包中 NMA 模型的构建要点：

假设我们已从若干原始研究中收集了效应量数据，接下来需遍历全部K项试验，统计治疗比较总数 M（由这 K 项试验产生的 **两两处理比较（pairwise comparisons）** 总数，若一项三臂试验比较 A、B、C，则该试验贡献 3 组比较：A-B、A-C、B-C）。随后计算每个比较 m 的效应量 $\hat\theta_m$，并将所有效应量汇总在向量 $\boldsymbol{\hat\theta} = (\hat\theta_1, \hat\theta_2, \dots, \hat\theta_M)$ 中。为了进行 NMA，我们现在需要一个模型来描述这个观察到的效应大小向量 $\boldsymbol{\hat\theta}$ 是如何产生的：在`{netmeta}`中，使用如下模型 ([Schwarzer, Carpenter, and Rücker 2015, chap. 8](https://doing-meta.guide/references#ref-schwarzer2015meta))：

$\begin{equation} \boldsymbol{\hat\theta} =\boldsymbol{X} \boldsymbol{\theta}_{\text{treat}} + \boldsymbol{\epsilon} \tag{12.4} \end{equation}$

我们假设所观测到的影响量向量 $\boldsymbol{\hat\theta}$ 是由等式的右侧——即我们的模型所生成的。第一部分 $\boldsymbol{X}$ 是一个 $m×n$ 的设计矩阵，其中列代表不同的处理方式，行代表处理方式之间的比较数量。在该矩阵中，一个处理方式的比较由同一行中的 1 和 -1 定义，其中列的位置对应于正在比较的处理方式。

简单解释就是：==把比较关系编码成矩阵==

| 符号  | 意义                                            |
| --- | --------------------------------------------- |
| $m$ | 所有研究拆分后得到的 **成对处理比较**（pairwise comparisons）总数 |
| $n$ | 网络中涉及的 **干预（treatments）** 数目                  |

- **维度**：$X$ 是一个 $m \times n$ 的矩阵。
    
    - **行**（$m$）对应一条具体的治疗比较。
    - **列**（$n$）对应一个具体的治疗。

==行的构造规则：==

对于任意一次比较「治疗 A vs B」，在对应行里：

- 把治疗 A 所在列记为 **1**；
    
- 把治疗 B 所在列记为 **−1**；
    
- 其余治疗所在列记为 0。

> **含义**：  
> 1 表示该行把治疗 A 作为“效果方向”的正向；  
> −1 表示与之比较的治疗 B；  
> 0 表示本行并未涉及其他治疗。

##### 例子

假设网络中有 3 个治疗：A、B、C。三种可能的比较与 XXX 如下：

| 比较     | A 列 | B 列 | C 列 |
| ------ | --- | --- | --- |
| A vs B | 1   | −1  | 0   |
| A vs C | 1   | 0   | −1  |
| B vs C | 0   | 1   | −1  |

写成矩阵形式：

$X=\begin{pmatrix} 1 & -1 & 0\\ 1 & 0 & -1\\ 0 & 1 & -1 \end{pmatrix}.$

==用矩阵表示观测效应量与真实治疗效应的关系==（再次回到公式12.4）：

$\begin{equation} \boldsymbol{\hat\theta} =\boldsymbol{X} \boldsymbol{\theta}_{\text{treat}} + \boldsymbol{\epsilon} \tag{12.4} \end{equation}$

其中

- 该公式中最重要的部分是向量 $\boldsymbol{\theta}_{\text{treat}}$ 是 长度为 $n$ 的向量，存放每行治疗观测的相对**真实效应**。这个向量正是我们的网络荟萃分析模型所需要估计的内容，因为它使我们能够确定我们网络中哪些治疗方式是最有效的。
    
- $\boldsymbol{\epsilon}$ 是抽样误差（通常服从零均值、已知或可估计的方差）。参数 $\boldsymbol{\epsilon}$ 是一个向量，其中包含了所有比较中的抽样误差 $\epsilon_m$ 。每个比较的抽样误差被假定是从均值为零、方差为 $\sigma^2_m$ 的高斯正态分布中随机抽取得到的：
$\begin{equation} \epsilon_m \sim \mathcal{N}(0,\sigma_m^2) \tag{12.4} \end{equation}$
    
- 即矩阵中每条观测获得一个相对效应 + 抽样误差，遍历$\boldsymbol{X}$ 后获得等式左侧的 $\hat{\theta}$
 
==为什么要用这种编码?==

| 目的              | 解释                                                                    |
| --------------- | --------------------------------------------------------------------- |
| **统一表示双臂与多臂试验** | 把多臂试验拆解为若干成对比较后，用同一套 1 / −1 / 0 规则即可编码全部研究。                           |
| **方便矩阵运算**      | 估计 $\boldsymbol{\theta}_{\text{treat}}$ 的过程中，可直接用线性模型或广义最小二乘等矩阵方法求解。  |
| **自动保证“平衡约束”**  | 由于每行 1 与 −1 相抵，β\betaβ 的估计不会受多臂试验内部对比非独立性的影响；配合协方差矩阵还能正确处理多臂试验产生的相关性。 |
为了说明该模型公式（see [Schwarzer, Carpenter, and Rücker 2015, 189](https://doing-meta.guide/references#ref-schwarzer2015meta)），假设我们的 NMA包含 K = 5 项研究。每项研究都包含一个独特的治疗对比（即 K = M）。这些对比分别是 A - B、A - C、A - D、B - C 和 B - D。这便形成了一个（已观测到的）对比向量: $\boldsymbol{\hat\theta} = (\hat\theta_{1\text{，A,B}}，\hat\theta_{2\text{，A,C}}，\hat\theta_{4\text{，A,D}}，\hat\theta_{4\text{，B,C}}，\hat\theta_{5\text{，B,D}})^\top$ 。我们的目标是估计我们网络中包含的所有四种情况的真实效应大小，$\boldsymbol{\theta}_{\text{treat}} = (\theta_{\text{A}}，\theta_{\text{B}}，\theta_{\text{C}}，\theta_{\text{D}})^\top$ 。如果我们将这些参数代入我们的模型公式，就会得到以下方程：
$\begin{align} \boldsymbol{\hat\theta} &= \boldsymbol{X} \boldsymbol{\theta}_{\text{treat}} + \boldsymbol{\epsilon} \notag \\ \begin{bmatrix} \hat\theta_{1\text{,A,B}} \\ \hat\theta_{2\text{,A,C}} \\ \hat\theta_{3\text{,A,D}} \\ \hat\theta_{4\text{,B,C}} \\ \hat\theta_{5\text{,B,D}} \\ \end{bmatrix} &= \begin{bmatrix} 1 & -1 & 0 & 0 \\ 1 & 0 & -1 & 0 \\ 1 & 0 & 0 & -1 \\ 0 & 1 & -1 & 0 \\ 0 & 1 & 0 & -1 \\ \end{bmatrix} \begin{bmatrix} \theta_{\text{A}} \\ \theta_{\text{B}} \\ \theta_{\text{C}} \\ \theta_{\text{D}} \\ \end{bmatrix} + \begin{bmatrix} \epsilon_{1} \\ \epsilon_{2} \\ \epsilon_{3} \\ \epsilon_{4} \\ \epsilon_{5} \\ \end{bmatrix} \tag{12.5} \end{align}$

注：$\boldsymbol{\hat{\theta}}$ 可能是多个值，取决于矩阵中 $n$ 的数量

值得注意的是，在目前的形式下，这个模型公式从数学的角度来看是有问题的。现在，这个模型被过度参数化了。我们的模型中有太多的参数 $\boldsymbol{\theta}_{\text{treat}}$，无法根据手头的信息进行估计。

这与设计矩阵$\boldsymbol{X}$的**不满秩**有关。在我们的例子中，当矩阵的列并非全部**相互独立**时，该矩阵就没有**满秩**；或者换句话说，当独立列的数量少于总列数（n）时，就会出现这种情况。因为我们正在处理一个**治疗网络**，所以很明显，治疗组合之间不会完全相互独立。例如，治疗 D 的列（第四个列）可以被描述为前三个列的**线性组合**。**因此其不可逆。** 无法直接使用（加权）最小二乘法来估计$\boldsymbol{\theta}_{\text{treat}}$。

netmeta包中实现的图论方法会建立一个Moore-Penrose伪逆矩阵，允许使用**加权最小二乘法**来计算我们网络模型的拟合值。该程序同样能处理**多臂试验**（即试验中包括不止一个两两比较）。多臂比较是相关的，因为至少有一种情况被比较了不止一次。这意味着多组研究比较的精确度被人为地提高了——除非我们的模型考虑了这一点。

此外，该模型还允许我们纳入**研究间异质性**的估计（$s^2_m + \hat\tau^2$）。在 netmeta 包中，$τ^2$的值是采用DerSimonian-Laird 估计量的改编版本来估计的（Jackson, White, and Riley 2013，另见第4.1.2.1章）

还可以计算  $I^2$ 的等效指标，该指标在此处代表了我们网络中的**不一致性**。与 Higgins 和 Thompson 的公式（见第 5.1.2 章）类似，此版本的 $I^2$ 也是由 Q 统计量推导得出。然而，在网络元分析中，Q为网络中的总异质性（也记为$Q_{total}$）。因此，公式如下所示：

$\begin{equation} I^2 = \text{max} \left(\frac{Q_{\text{total}}-\text{d.f.}} {Q_{\text{total}}}, 0 \right) \tag{12.6} \end{equation}$
为防止得到负值。Q 统计量有时会因抽样误差导致 $Qtotal<d.f.$，此时按照定义 $I^{2}$ 应视为 0。

我们网络中的自由度是：

$\begin{equation} \text{d.f.} = \left( \sum^K_{k=1}p_k-1 \right)- (n-1) \tag{12.7} \end{equation}$

其中，$K$ 为研究总数，$p$ 为某研究K中的条件数，$n$ 为我们的网络模型中的治疗总数

假设：
- K=3 个试验，治疗臂数分别为 2、3、2（即 p1=2, p2=3, p3=2）
- 网络共有 n=4种治疗
则：
- $d.f.=[(2+3+2)−1]−(4−1)=  6−1−3=2.$

因此，对应的 $Q_{\text{total}}​$ 应当在 2 个自由度的 $\chi^2$ 分布下进行显著性检验。
### 12.2.2 Frequentist Network Meta-Analysis in _R_[](https://doing-meta.guide/netwma#frequentist-network-meta-analysis-in-r)

下面，我们将使用`{netmeta}`进行 NMA

首先安装包，然后从库中加载它：
```R
# 首先安装包
install.packages("netmeta")
# 然后从库中加载它
library(netmeta)
```

#### 12.2.2.1 数据准备[](https://doing-meta.guide/netwma#data-preparation-1)

我们使用 `TherapyFormats` 数据集。该数据集是基于一项真实的 NMA 建模的，该分析评估了**不同形式的认知行为疗法对抑郁症的疗效**（P. Cuijpers, Noma, et al. 2019）。所有纳入的研究都是在后测时测量了对抑郁症状影响的随机对照试验。纳入比较的效应量以两种干预方式之间的**标准化均值差**（SMD）表示。

>[!info] ** “TherapyFormats” 数据集**
> `TherapyFormats`  是 **{dmetar}** 包的一部分. 如果你已经安装了 **{dmetar}**, 并且加载了他, 运行 `data(TherapyFormats)` 将会将该数据集自动保存在你的 _R_ environment. 然后数据集就可以使用了。
library(dmetar)
data(TherapyFormats)

```r
library(dmetar)
data(TherapyFormats)

head(TherapyFormats[1:5])

```
```
##          author     TE  seTE treat1 treat2
## 1  Ausbun, 1997  0.092 0.195    ind    grp
## 2  Crable, 1986 -0.675 0.350    ind    grp
## 3  Thiede, 2011 -0.107 0.198    ind    grp
## 4 Bonertz, 2015 -0.090 0.324    ind    grp
## 5     Joy, 2002 -0.135 0.453    ind    grp
## 6   Jones, 2013 -0.217 0.289    ind    grp
```

- **要使用** **netmeta**，我们数据集中的所有效应量必须已经预先计算好，也就是数据集中需要包含Te 和 seTE 两列数据，我们可以使用 `metafor` 或 `meta` 包 pairwise() 完成这些计算。  
- treat1和treat2表示正在比较的两个条件。  
- studlab列包含独特的研究标签，表示从哪个研究中提取了特定的处理比较。本列有助于检查**多组研究（即有多个比较的研究）**。我们可以使用`table()`和`as.matrix`来实现这一点：
```r
as.matrix(table(TherapyFormats$author))
```
```
## [...]
## Bengston, 2004              1
## Blevins, 2003               1
## Bond, 1988                  1
## Bonertz, 2015               1
## Breiman, 2001               3
## [...]
```

我们的 `TherapyFormats` 数据集仅包含一项多臂试验，即 Breiman, 2001 所进行的那项试验。从我们的观察来看，该试验包含了三项对比，而其他所有试验都只包含一项对比。

==**在准备网络元分析数据时，需要注意：**==

- 在数据集中包含一个研究标签列。

- 为每个单独的研究赋予一个唯一名称。

- 为包含两个及以上比较的研究赋予完全相同的标签名称。

#### 12.2.2.2 模型拟合[](https://doing-meta.guide/netwma#model-fitting-2)

==我们现在可以使用`netmeta`拟合我们的第一个NMA模型。最重要的参数如下所示：==

- TE：数据集中包含每个比较的**效应量的列**的名称。

- seTE：包含每个比较的**标准误差的列**的名称。

- treat1：数据集中包含**第一个干预名称的列**的名称。

-  treat2：数据集中包含**第二个干预名称的列**的名称。

- studlab：从中提取比较的研究标签。尽管此参数是可选的，但建议始终指定，这是**检测网络中是否存在多臂试验的唯一方法**。
---
- data：数据集的名称。

- sm：用于指定我们所使用的效应量类型，包括："RD"（风险差）、"RR"（风险比）、"OR"（比值比）、"HR"（风险比）、"MD"（均值差）、"SMD"（标准化均值差）等。

- fixed：指示是否应进行固定效应模型的网络元分析（TRUE/FALSE）

- random：指示是否应该使用随机效应模型（TRUE/FALSE）

- reference.group：指示应使用哪种治疗作为参考治疗用于所有其他治疗。

- tol.multiarm：用于多臂研究的效应量的比较，处理多臂试验中效应量的内在一致性（允许设置容差阈值）。

- details.chkmultiarm：是否打印效应大小不一致的多臂比较的估计值（TRUE/FALSE）。

- sep.trts：用于指定在比较标签中使用的分隔符字符（例如“ vs. ”）。

我们将结果保存为`m.netmeta`，并设定常规护理 ("cau") 作为参照组。我们假设采用`固定效应模型`是适合的，代码如下：
```r
m.netmeta <- netmeta(TE = TE,
                     seTE = seTE,
                     treat1 = treat1,
                     treat2 = treat2,
                     studlab = author,
                     data = TherapyFormats,
                     sm = "SMD",
                     fixed = TRUE,
                     random = FALSE,
                     reference.group = "cau",
                     details.chkmultiarm = TRUE,
                     sep.trts = " vs ")
summary(m.netmeta)
```
```
## Original data (with adjusted standard errors for multi-arm studies):
## 
##                    treat1 treat2    TE seTE seTE.adj narms multiarm
## [...]
## Burgan, 2012          ind    tel -0.31 0.13   0.1390     2         
## Belk, 1986            ind    tel -0.17 0.08   0.0830     2         
## Ledbetter, 1984       ind    tel -0.00 0.23   0.2310     2         
## Narum, 1986           ind    tel  0.03 0.33   0.3380     2         
## Breiman, 2001         ind    wlc -0.75 0.51   0.6267     3        *
## [...]
## 
## Number of treatment arms (by study):
##                          narms
## Ausbun, 1997                 2
## Crable, 1986                 2
## Thiede, 2011                 2
## Bonertz, 2015                2
## Joy, 2002                    2
## [...]
## 
## Results (fixed effects model):
## 
##                treat1 treat2   SMD         95%-CI      Q leverage
## Ausbun, 1997      grp    ind  0.06 [ 0.00;  0.12]   0.64     0.03
## Crable, 1986      grp    ind  0.06 [ 0.00;  0.12]   3.05     0.01
## Thiede, 2011      grp    ind  0.06 [ 0.00;  0.12]   0.05     0.03
## Bonertz, 2015     grp    ind  0.06 [ 0.00;  0.12]   0.01     0.01
## Joy, 2002         grp    ind  0.06 [ 0.00;  0.12]   0.02     0.00
## [....]
## 
## Number of studies: k = 182
## Number of treatments: n = 7
## Number of pairwise comparisons: m = 184
## Number of designs: d = 17
## 
## Fixed effects model
## 
## Treatment estimate (sm = 'SMD', comparison: other treatments vs 'cau'):
##         SMD             95%-CI      z  p-value
## cau       .                  .      .        .
## grp -0.5767 [-0.6310; -0.5224] -20.81 < 0.0001
## gsh -0.3940 [-0.4588; -0.3292] -11.92 < 0.0001
## ind -0.6403 [-0.6890; -0.5915] -25.74 < 0.0001
## tel -0.5134 [-0.6078; -0.4190] -10.65 < 0.0001
## ush -0.1294 [-0.2149; -0.0439]  -2.97   0.0030
## wlc  0.2584 [ 0.2011;  0.3157]   8.84 < 0.0001
##
## 
## Quantifying heterogeneity / inconsistency:
## tau^2 = 0.26; tau = 0.51; I^2 = 89.6% [88.3%; 90.7%]
## 
## Tests of heterogeneity (within designs) and inconsistency (between designs):
##                       Q d.f.  p-value
## Total           1696.84  177 < 0.0001
## Within designs  1595.02  165 < 0.0001
## Between designs  101.83   12 < 0.0001
```

输出结果包含丰富信息。

1. 首先展示了各项比较的效应量，其中**星号标记的多臂研究**已进行标准误校正（考虑效应量相关性）。随后是各研究的治疗臂数量概览。

2. 下一个表格向我们展示了**拟合值**，用于我们（固定效应）网络元分析模型中的每个比较。其中Q值列尤为关键，它揭示了各比较对网络不一致性的贡献程度。例如，Crable（1986）的研究Q值高达3.05，表明其贡献较大。

3. 网络的基本信息也展示在输出中。

4. 然后，我们进入了网络元分析的核心部分：**治疗的效应估计（“cau”**）进行比较，因此cau未显示效应值。值得注意的是，网络中存在高度异质性/不一致性（I²=89.6%），这提示**固定效应模型可能并不适用**。

5. 最后的**异质性检验**将总异质性分解为设计内异质性与设计间不一致性。**设计**指试验中包含的治疗条件组合（如A-B或A-B-C）。**设计内异质性**指完全相同设计的不同研究间存在的真实效应差异，而**设计间变异**反映网络中的不一致性（即直接比较和间接比较的结果冲突）。当前结果显示两者均达显著水平（p < 0.001）。

- 这提示随机效应模型可能更合适。为了进一步验证这一点，我们可以使用**full design-by-treatment interaction random-effects model**计算总不一致性（J. Higgins等，2012）。为此，我们只需将**m.netmeta** 对象输入 `decomp.design()`中：

```r
decomp.design(m.netmeta)
```
```
## Q statistics to assess homogeneity / consistency
##  [...]
## Design-specific decomposition of within-designs Q statistic
## 
##      Design      Q df  p-value
##  cau vs grp   82.5 20 < 0.0001
##  cau vs gsh    0.7  7   0.9982
##  cau vs ind  100.0 29 < 0.0001
##  cau vs tel   11.4  5   0.0440
##  [...]
## 
## Between-designs Q statistic after detaching of single designs
## 
##    Detached design      Q df  p-value
##  [...]
##         ind vs wlc  77.23 11 < 0.0001
##         tel vs wlc  95.45 11 < 0.0001
##         ush vs wlc  95.81 11 < 0.0001
##  gsh vs ind vs wlc 101.78 10 < 0.0001
##
## Q statistic to assess consistency under the assumption of
## a full design-by-treatment interaction random effects model
## 
##                    Q df p-value tau.within tau2.within
## Between designs 3.82 12  0.9865     0.5403      0.2919
```

- 在输出结果中，首先呈现给我们的是一系列 $Q$ 值，这些值显示了每个设计对我们的模型中“内部”和“不同设计”之间的异质性/不一致性所做出的单独贡献。

- 输出结果的重要部分位于最后一部分（用于评估在假设存在完全的设计与治疗交互随机效应模型下的一致性的 $Q$ 统计量）。

- 我们看到，当假设存在完全的设计与治疗随机效应模型时，$Q$ 值大幅下降（之前为 101.83，现在为 3.82），并且不同设计之间的不一致性不再显著（$p$ = 0.986），直接比较和间接比较的结果一致，证据可信，因为已经建立了设计内的随机效应。这还表明，随机效应模型可能被用于（至少部分地）解释我们网络模型中的不一致性和异质性。

#### 12.2.2.3 网络模型的进一步检验[](https://doing-meta.guide/netwma#further-examination-of-the-network-model)
##### 12.2.2.3.1 网络证据图[](https://doing-meta.guide/netwma#the-network-graph)
在使用 `netmeta` 包拟合网络元分析模型后，可以生成**网络图**。这可以通过 `netgraph()` 来实现。

`netgraph()` 函数有许多参数，您可以在控制台中运行 `?netgraph` 来查看这些参数。然而，大多数这些参数都有非常合理的默认值，因此不需要过多指定。

首先，我们在函数输入我们拟合的模型 `m.netmeta`。由于我们在模型中使用了简短的标签（treat1 和 treat2），所以在绘制时应将其替换为完整的版本（存储在 treat1.long 和 treat2.long 中）。这可以通过 `labels` 参数来实现，我们需要提供所有治疗的完整名称。治疗标签应与存储在 `m.netmeta$trts` 中的标签的顺序相同。
```r
# Show treatment order (shortened labels)
m.netmeta$trts
```
```
## [1] "cau" "grp" "gsh" "ind" "tel" "ush" "wlc"
```
```r
# Replace with full name (see treat1.long and treat2.long)
long.labels <- c("Care As Usual", "Group", 
                 "Guided Self-Help", 
                 "Individual", "Telephone", 
                 "Unguided Self-Help", 
                 "Waitlist")

netgraph(m.netmeta, 
         labels = long.labels)
```
![[Pasted image 20251114143331.png]]
![[Pasted image 20251114162540.png]]
这个网络图传输了多种信息。

1. 首先，我们可以看到我们网络中的比较的整体结构。这使我们能够更好地理解原始数据中哪些治疗方法相互进行了比较。

2. 此外，我们还可以看到图表中的边有不同的粗细。粗细程度代表我们在网络中发现特定比较的频率。例如，我们看到guided self-help 在许多试验中与 waiting list 进行了比较。

3. 我们还在网络中看到了多臂试验，它由一个阴影三角形表示。这是布雷曼的研究，该研究比较了引导式自助疗法、个体治疗和等待名单。

- ==网络图函数还允许绘制三维图，这有助于更好地理解复杂的网络结构。该函数需要安装并加载`{rgl}`包。要生成三维图，我们只需将 `dim` 参数设置为“3d”。==
![[Pasted image 20251114162616.png]]
##### 12.2.2.3.2 可视化直接和间接证据[](https://doing-meta.guide/netwma#visualizing-direct-and-indirect-evidence)
在接下来的步骤中，让我们看看用于估计每个比较的直接和间接证据的比例。`{dmetar}` 中的`direct.evidence.plot` 函数就是为此目的而开发的。
该函数会为我们生成一个图表，**该图表展示了用于每个估计比较的直接证据和间接证据所占的百分比**。`direct.evidence.plot` 函数仅需要我们提供的已拟合的网络荟萃分析模型 `m.netmeta` 作为输入：
```r
library(dmetar)

d.evidence <- direct.evidence.plot(m.netmeta)
plot(d.evidence)
```
![[Pasted image 20251114143932.png]]

如我们所见，在我们的网络模型中存在若干需要仅依靠间接证据来推断的估计值。该图表还为我们提供了两个额外的指标：每个估计比较的最小并行度和平均路径长度。根据 König, Krahn, and Binder ([2013](https://doing-meta.guide/references#ref-konig2013visualizing)) 的研究，**mean path length > 2 表明对这种比较估计应特别谨慎地解读(由于走过的路径长，证据多数由间接比较的证据支持）**。
##### 12.2.2.3.3 效果估计表格[](https://doing-meta.guide/netwma#effect-estimate-table)
接下来，我们可以查看我们网络中针对所有可能的治疗比较的估计值。

为此，我们可以使用保存在 `m.netmeta$TE.fixed` 中的矩阵（如果使用固定效应模型）或者 `m.netmeta$TE.random` 中的矩阵（如果使用随机效应模型）。我们需要进行一些预处理步骤，以使矩阵更易于阅读。首先，我们从我们的 `m.netmeta` 对象中提取数据，并将矩阵中的数字四舍五入到两位小数。
```r
result.matrix <- m.netmeta$TE.fixed
result.matrix <- round(result.matrix, 2)
```
鉴于我们矩阵中的上“三角形”会包含冗余信息，我们将使用以下代码将该三角形区域替换为空值：
```r
result.matrix[lower.tri(result.matrix, diag = FALSE)] <- NA
```

| 步骤  | 函数/语法                                    | 具体含义                                                                                                                 |
| --- | ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| 1   | `lower.tri(result.matrix, diag = FALSE)` | 生成一个与 `result.matrix` 同尺寸的逻辑矩阵。  <br>- 元素在**下三角**(含行索引 > 列索引) 位置记为 `TRUE`。  <br>- `diag = FALSE` 表示主对角线位置设为 `FALSE`。 |
| 2   | `result.matrix[...]`                     | 用第 1 步得到的布尔索引挑出 **下三角（不含对角线）** 的所有元素。                                                                                |
| 3   | `<- NA`                                  | 将这些被选中的元素统一赋值为 `NA`。                                                                                                 |

这给出了以下结果：
```r
result.matrix
```
```
##     cau  grp   gsh  ind   tel   ush   wlc
## cau   0 0.58  0.39 0.64  0.51  0.13 -0.26
## grp  NA 0.00 -0.18 0.06 -0.06 -0.45 -0.84
## gsh  NA   NA  0.00 0.25  0.12 -0.26 -0.65
## ind  NA   NA    NA 0.00 -0.13 -0.51 -0.90
## tel  NA   NA    NA   NA  0.00 -0.38 -0.77
## ush  NA   NA    NA   NA    NA  0.00 -0.39
## wlc  NA   NA    NA   NA    NA    NA  0.00
```
如果我们想在我们的研究论文中报告这些结果，一个不错的做法是**同时将每个效应量估计值的置信区间也包含在内**。这些置信区间可以通过与之前相同的方式获得，即使用 `m.netmeta` 中的 `lower.fixed` 和 `upper.fixed`（或者 `lower.random` 和 `upper.random`）矩阵。

**更便捷的导出所有估计效应值的方法是使用 `netleague` 函数**。该函数会生成一个类似于我们之前创建的表格的表。然而，在 `netleague` 生成的矩阵中：

- **上三角部分只会显示我们网络中可用的直接比较的合并效应值**，有点像如果我们对每个比较都进行了常规的元分析所能得到的结果。由于我们没有所有比较的直接证据，矩阵上三角部分的一些单元格将为空。
- **`netleague` 生成的矩阵的下三角部分包含每个比较的估计效应值**（即使对于只有间接证据可用的比较也是如此）。

`netleague` 的输出可以轻松导出为一个`.csv` 文件。它可以用于在一个表格中报告我们网络元分析的综合结果。使用此函数的另一个优点是，效应值估计值和置信区间将在每个单元格中一起显示。假设我们想要生成这样一个治疗效应值表格，并将其保存为一个名为 “netleague.csv” 的 .csv 文件。这可以通过以下代码实现：
```r
# Produce effect table
netleague <- netleague(m.netmeta, 
                       bracket = "(", # use round brackets
                       digits=2)      # round to two digits

# Save results (here: the ones of the fixed-effect model)
write.csv(netleague$fixed, "netleague.csv")
```
![[Pasted image 20251114163432.png]]
##### 12.2.2.3.4 治疗排名[](https://doing-meta.guide/netwma#treatment-ranking)
我们可以回答的问题：哪种干预方式获得了最大的效应量？在 `{netmeta}` 中实现的 `netrank` 可以实现。**该函数允许我们生成治疗方法的排名**，表明哪种治疗方法可能产生最大的益处。

`netrank` 函数与 `netmeta` 工具自身所采用的模型一样，也是基于频率学方法的。这种频率学方法使用 `P-Score` 来对治疗方案进行排序，该分数衡量的是一个治疗方案优于另一个治疗方案的确定性程度，这一确定性程度是针对所有竞争治疗方案进行平均计算得出的。**P 分数已被证明与 SUCRA 等价**（Rücker 和 Schwarzer，2015），我们将在关于贝叶斯网络荟萃分析的章节中对其进行详细介绍。

**`netrank` 函数需要我们的 `m.netmeta` 模型**作为输入。**此外，我们还应指定 `small.values` 参数**，该参数用于确定在比较中较小（即负）的影响大小是表示有益（“良好”）效果还是有害（“不良”）效果。在这里，我们使用 small.values = "good"，表明该指标为低优指标（即该干预的值越低，对抑郁症状的改善越有效）。

```r
netrank(m.netmeta, small.values = "good") # 或者设置为 small.values = "desirable"
```
```
##     P-score
## ind  0.9958
## grp  0.8184
## tel  0.6837
## gsh  0.5022
## ush  0.3331
## cau  0.1669
## wlc  0.0000
```
我们可以看到 `ind` 表现出最大的益处（最高的`P-Score`）相反，`wlc`（wlc）的 P 值为零，这似乎与我们的直觉相符，即仅仅让患者等待治疗并非最佳选择。

**然而，绝不能仅仅因为某种治疗方法在排名中得分最高，就自动得出它是“最佳”的结论**([Mbuagbaw et al. 2017](https://doing-meta.guide/references#ref-mbuagbaw2017approaches))。为了更好地直观呈现我们网络中的不确定性，可以制作一个森林图，在该图中，一种条件被用作对照组。

在 `{netmeta}` 中，可以通过使用 **`forest()`** 功能来实现这一目标。 `{netmeta}` 中的 `forest()` 功能与 `{meta}` 包中的功能工作方式非常相似。在森林图中我们需要使用 `reference.group` 参数来指定参考组。我们再次使用常规的`cau`。
```r
forest(m.netmeta, 
       reference.group = "cau",
       sortvar = TE,
       xlim = c(-1.3, 0.5),
       smlab = paste("Therapy Formats vs. Care As Usual \n",
                     "(Depressive Symptoms)"),
       drop.reference.group = TRUE,
       label.left = "Favors Intervention",
       label.right = "Favors Care As Usual",
       labels = long.labels)
```
![[Pasted image 20251114150234.png]]
**该森林图表明，除了个体治疗之外，还有其他表现优异的治疗方式**。我们还注意到，一些置信区间存在重叠现象。这使得做出明确的决策变得较为困难。虽然个体治疗似乎确实能产生最佳效果，但除了 `waitlist` 几种治疗方式也比常规护理能带来显著的益处。
#### 12.2.2.4 评估结果的有效性[](https://doing-meta.guide/netwma#evaluating-the-validity-of-the-results)
##### 12.2.2.4.1 网状热图[](https://doing-meta.guide/netwma#net-heat-plot)
**可使用 `netmeta` 内置的函数 `netheat` 绘制** **网络热图**。网络热图对于评估我们网络模型中的不一致性以及哪些设计导致了这种不一致性非常有帮助。仅需输入拟合好的网络元分析对象即可生成图形：
```r
netheat(m.netmeta)
```
![[Pasted image 20251114150547.png]]
该函数会生成一个二维热图，其中每一行中的设计都会与其他列中的设计进行比较。重要的是，行和列所代表的是具体的设计，而非我们网络中的单个治疗比较。因此，该图还为我们在多臂研究中使用的设计（该研究比较了引导式自助疗法、个体治疗以及等待名单）设置了行和列。这个网络热图具有两个重要特征：
- **灰色方框**。灰色方框表示一个治疗比较对于另一个治疗比较的估计有多重要。方框越大，该比较就越重要。分析这种关系的一个简单方法是依次查看图中的行，并在每一行中检查哪些方框最大。常见的发现是，热图的对角线上方框较大，因为这意味着使用了直接证据。例如，可以看到一个特别大的方框位于“cau 与 grp”行和“cau 与 grp”列的交叉点处。
- **颜色背景**。这些彩色背景代表着同一行设计中不一致程度的大小，这种不一致程度可归因于同一列的设计。字段颜色的范围可以从深红色（表示强烈的不一致）到蓝色（表示该设计提供的证据支持了该行的证据）。netheat 函数使用一种算法将行和列分类为具有较高或较低不一致程度的簇。在我们的图表中，几个不一致的字段显示在左上角。例如，在“ind vs wlc”这一行中，我们看到“cau vs grp”列中的条目显示为红色。这意味着“cau vs grp”为“ind vs wlc”估计所提供的证据是不一致的。另一方面，我们看到“gsh vs wlc”列中的字段有深蓝色的背景，这表明该设计的证据支持了“ind vs wlc”这一行设计的证据。

**核心概念**

- **行／列**：代表一整套 _design_（同一个研究中出现的治疗组合）。
    
- **灰色方块**：表示「列 design 对行 design 的估计有多重要」。
    
- **底色渐变**：表示这份贡献（列 → 行）带来的 **一致性/不一致性** 程度。
 
 **例子拆解：**

- **行 `ind vs wlc`，列 `cau vs grp` → 深红**
    
    - _解释_：在估计 _ind vs wlc_ 时，需要部分借助 _cau vs grp_ 的间接路径。但这两类设计给出的效应方向或大小显著不一致，提示可能存在效应修饰因子。
        
- **行 `ind vs wlc`，列 `gsh vs wlc` → 深蓝**
    
    - _解释_：把 _gsh vs wlc_ 的直接结果当作网络中的间接证据时，与 _ind vs wlc_ 的直接结果非常吻合，为该估计提供了支撑。
    
- netheat 图用颜色告诉你 **“把列的某一份直接证据抽掉，仅靠剩下的（间接 + 其它直接）证据重新估计同一行的比较，效应量估计会不会出现显著差异？”**——差异显著就以红／蓝显示，表示 **冲突（inconsistency）**；差异不显著就接近白／浅色，表示二者 **一致**。即把列的比较拿掉，重新估计同一行的比较，结果会不会不一致，程度怎么样？可能贡献越大的研究剔除掉之后不一致程度越大，因为贡献大！

**我们应当提醒自己，这些结果是基于固定效应模型得出的，因为我们正是利用该模型来构建我们的网络荟萃分析模型的。** 然而，就目前我们所了解到的情况来看，使用固定效应模型已变得越来越不恰当——因为存在太多的异质性和设计不一致性。

因此，**让我们来查看一下假设使用随机效应模型时，网络热图会如何变化**。我们可以通过将 `netheat` 中的 `random` 参数设置为 `TRUE` 来实现这一操作：
![[Pasted image 20251114153441.png]]
我们发现，这使得我们网络中的不一致性显著降低。现在没有出现带有深红色背景的字段了，这表明一旦使用随机效应模型，我们模型的整体一致性就有了显著提高。

**因此，我们可以得出结论，随机效应模型更适合我们的数据。在实践中，这意味着我们在设置comb时使用 `netmeta` 重新运行模型。随机为 `TRUE`（`comb.fixed` to `FALSE`），我们只报告基于随机效应模型的分析结果。** 我们在这里省略了这一步，因为我们之前提出的所有分析也可以以完全相同的方式应用于随机效应网络模型。
##### 12.2.2.4.2 网络切分法[](https://doing-meta.guide/netwma#net-splitting)

另一种检查我们网络一致性的方法是“网络拆分”。这种方法将我们的网络估计值分为直接证据和间接证据的贡献部分，这使我们能够控制网络中各个比较估计值中的不一致性。要应用“网络拆分”技术，我们只需将拟合模型提供给“网络拆分”函数即可：

```r
netsplit(m.netmeta)
```
```
## Separate indirect from direct evidence using back-calculation method
## 
## Fixed effects model: 
## 
##  comparison  k prop     nma  direct  indir.    Diff     z  p-value
##  grp vs cau 21 0.58 -0.5767 -0.3727 -0.8628  0.4901  8.72 < 0.0001
##  gsh vs cau  8 0.22 -0.3940 -0.5684 -0.3442 -0.2243 -2.82   0.0048
##  ind vs cau 30 0.71 -0.6403 -0.7037 -0.4863 -0.2174 -3.97 < 0.0001
##  tel vs cau  6 0.35 -0.5134 -0.7471 -0.3867 -0.3604 -3.57   0.0004
##  ush vs cau  9 0.35 -0.1294 -0.1919 -0.0953 -0.0966 -1.06   0.2903
##  [...]
## 
## Legend:
##  [...]
##  Diff       - Difference between direct and indirect estimates
##  z          - z-value of test for disagreement (direct vs. indirect)
##  p-value    - p-value of test for disagreement (direct vs. indirect)
```

**输出中所呈现的最重要信息是基于直接证据和间接证据得出的影响评估值之间的差异（即 `Diff`），以及这种差异是否具有显著性（通过 `p-value` 列来体现）。当 `p` 值小于 0.05 时，意味着直接评估值和间接评估值之间存在显著的分歧（不一致）。**

从输出结果中我们可以看到，确实存在许多对比数据，**这些数据表明直接证据和间接证据之间存在显著的不一致之处（当采用固定效应模型时）。** 通过森林图来直观呈现网络切分法结果是一种很好的方式：
```r
netsplit(m.netmeta) %>% forest()
```
![[Pasted image 20251114154106.png]]
改为随机效应可能会好起来。
##### 12.2.2.4.3 比较调整漏斗图[](https://doing-meta.guide/netwma#comparison-adjusted-funnel-plots)

评估网络元分析模型的发表偏倚难度较大，常规元分析的多数技术无法直接应用。不过，**比较调整漏斗图可用于评估网络元分析的发表偏倚风险**（Salanti等，2014），适用于对发表偏倚影响有特定假设的情况。

例如，具有“新颖”发现的研究更易发表，即便样本量小，科学界追求“突破性”结果（如新疗法优于标准疗法），导致数据出现类似小研究效应的情况。在比较新、旧疗法的漏斗图中，效应量可能不对称分布，因为“不理想”的结果（新疗法并不优于旧疗法）被搁置，小样本时新疗法需更大才值得发表，理论上会产生标准元分析中的不对称漏斗图。

但这种模式仅在效应量编码方式一致时才出现，如检验“新vs旧”假设时，需确保效应量解释一致（正效应量表示“新”治疗更优，负效应量相反），可通过定义治疗从旧到新“排序”并据此确定效应符号实现。

在 netmeta 包中，`funnel()`用于生成比较调整漏斗图以评估网络元分析的发表偏倚，其核心参数包括：

- `order`：定义治疗排序的向量，用于统一效应量方向（如新疗法 vs 旧疗法）

- `pch`：指定用于漏斗图中研究的符号。

- `col`：指定用于区分不同比较的颜色。我们在此处指定的颜色数量必须与漏斗图中唯一比较的数量相同。

- `linreg`：当设置为 `TRUE` 时，会进行 Egger 漏斗图不对称性检验，并将其 `p` 值显示在图中。
```r
funnel(m.netmeta, 
      order = c("wlc", "cau", "ind", "grp", # from old to new
                "tel", "ush", "gsh"), 
      pch = c(1:4, 5, 6, 8, 15:19, 21:24), 
      col = c("blue", "red", "purple", "forestgreen", "grey", 
              "green", "black", "brown", "orange", "pink", 
              "khaki", "plum", "aquamarine", "sandybrown", 
              "coral", "gold4"), 
      linreg = TRUE)
```
![[Pasted image 20251114154516.png]]
- 若假设成立，小样本（标准误高）研究在漏斗图中零线周围应呈不对称分布，因为比较新、旧疗法时，未发现新疗法更优的小样本研究发表可能性低，会系统性缺失于漏斗一侧。

- 但实际漏斗图对称，`Egger` 检验不显著（$p$=0.402），表明网络中不存在明显的小研究效应，至少不是因“创新性”治疗更易发表导致。

>[!info] 使用 {netmeta} 进行网络荟萃分析：总结性论述
>我们：1. 展示了 `{netmeta}` 所使用统计模型背后的核心思想，2. 描述了如何运用这种方法构建网络荟萃分析模型、3. 如何可视化和解读结果，4. 以及如何评估研究结果的有效性。再怎么强调都不为过，网络荟萃分析中的（临床）决策不应仅仅基于单一的测试或指标。相反，我们必须睁大眼睛审视我们的模型及其结果，检查我们发现的模式是否一致，并考虑到与某些估计值相关的巨大不确定性。

## 12.3 Bayesian Network Meta-Analysis[](https://doing-meta.guide/netwma#bayesian-net-ma)

接下来，我们将介绍如何基于贝叶斯层次框架进行网络荟萃分析。我们将使用的 R 软件包名为 `{gemtc}`（[Valkenhoef et al. 2012](https://doing-meta.guide/references#ref-van2012automating)）。

但首先，让我们先探讨一下贝叶斯推断的**一般原理**，以及**可用于网络荟萃分析的贝叶斯模型的类型**。
### 12.3.1 Bayesian Inference[](https://doing-meta.guide/netwma#bayesian-inference)

### 12.3.2 The Bayesian Network Meta-Analysis Model[](https://doing-meta.guide/netwma#bayesian-net-ma-model)
#### 12.3.2.1 Pairwise Meta-Analysis[](https://doing-meta.guide/netwma#pairwise-meta-analysis)
#### 12.3.2.2 Extension to Network Meta-Analysis[](https://doing-meta.guide/netwma#extension-to-network-meta-analysis)
### 12.3.3 Bayesian Network Meta-Analysis in _R_[](https://doing-meta.guide/netwma#bayesian-network-meta-analysis-in-r)

现在，让我们使用 `{gemtc}` 包来执行我们的第一个贝叶斯网络元分析。与往常一样，我们必须首先安装包，然后从库中加载它。
```r
library(gemtc)
```

`{gemtc}` 这个包依赖于 `{rjags}`（[Plummer 2019](https://doing-meta.guide/references#ref-rjags)），该软件用于我们之前描述的 Gibbs 抽样过程（Chapter [12.3.1](https://doing-meta.guide/netwma#bayesian-inference)）。然而，在安装并加载这个包之前，我们首先需要安装另一个名为 JAGS（“仅仅是一个吉布斯抽样器”的缩写）的软件。该软件适用于 Windows 和 Mac 系统，并且可以从互联网上免费下载。

完成上述步骤后，我们就可以安装并加载 `{rjags}` 这个包了。

```r
#install.packages("rjags")
library(rjags)
```
#### 12.3.3.1 Data Preparation[](https://doing-meta.guide/netwma#data-preparation-2)

使用的数据和 `frequentist network meta-analysis` 一样，但需要稍微调整数据结构以更好的适配 `gemtc` 包。

原始的 `TherapyFormats` 数据集包含 `TE` 和 `seTE` 两列，分别包含了标准化均值差值和标准误差，每行代表一次比较。如果我们要在 `{gemtc}` 中使用此类相对效应数据，就必须对数据框进行重新排列，**使每行代表一个单独的治疗组**。

此外，为了在比较中明确哪项治疗被设定为参考组，我们需要在效应量列中填写“NA”（无）。我们已将此重新整理的数据集以 “`TherapyFormatsGeMTC`” 为名保存下来。

`TherapyFormatsGeMTC` 数据集实际上是一个包含两个元素的列表，其中一个元素被称为“数据”。这个元素就是我们用于拟合模型所需的数据框架。让我们来了解一下它：
```r
library(dmetar)
data(TherapyFormatsGeMTC)

head(TherapyFormatsGeMTC$data)
```
```
##          study   diff std.err treatment
## 1 Ausbun, 1997  0.092   0.195       ind
## 2 Ausbun, 1997     NA      NA       grp
## 3 Crable, 1986 -0.675   0.350       ind
## 4 Crable, 1986     NA      NA       grp
## 5 Thiede, 2011 -0.107   0.198       ind
## 6 Thiede, 2011     NA      NA       grp
```
与频率学网状meta分析的数据整理格式不同，同一研究的数据被切分为两行，被对照组的diff 和 std.err 被设置为 NA。

`{gemtc}` 这个包还要求我们数据框中的列要正确标注名称。如果我们使用的是基于连续变量的效应量（例如平均差或标准化平均差），那么我们的数据集就必须包含以下这些列：

- study：每项研究的唯一标识（同netmeta包中的studlab列）。

- treatment：治疗的标签或缩写代码。

- diff：比较计算的效应量（如SMD）。参照治疗行设为NA，与之比较的治疗行则保存实际效应量。参照类别是按研究定义的，而不是按比较定义的。这意味着在多臂研究中，我们仍然只有一个参照治疗，所有其他治疗都与之比较。

- std.err：效应量的标准误。它也在参照组中设置为 NA，仅在比较治疗行中定义。

需注意，针对其他类型的数据（如二元结局数据），也存在不同的数据录入格式。对于不同效应量数据类型，数据集应如何结构化，具体可参考`gemtc包`的文档说明。您可以通过运行`?mtc.model`来访问它。然后滚动到`“Details”`部分。
#### 12.3.3.2 Network Graph[](https://doing-meta.guide/netwma#network-graph)
现在我们的数据已经准备就绪，我们将其输入到 `mtc.network` 函数中。这会生成一个类别为 `mtc.network` 的对象，我们可以将该对象用于后续的建模步骤。

由于我们使用的是预先计算好的效应量数据，所以我们必须在 `mtc.network` 的 `data.re` 参数中指定我们的数据集。对于原始效应量数据（例如均值、标准差和样本量），我们会使用 data.ab 参数。

可选的 treatments 参数可用于向 `{gemtc}` 提供网络中包含的所有处理的实际名称。此信息应以包含 id 和 descripition 的数据框形式准备。我们已经创建了这样一个数据框，并将其保存为 `TherapyFormatsGeMTC` 中的 `treat.codes`：
```r
TherapyFormatsGeMTC$treat.codes
```
```
##    id        description
## 1 ind         Individual
## 2 grp              Group
## 3 gsh   Guided Self-Help
## 4 tel          Telephone
## 5 wlc           Waitlist
## 6 cau      Care As Usual
## 7 ush Unguided Self-Help
```

我们利用这个`TherapyFormatsGeMTC$treat.codes`以及我们的效应量数据集`TherapyFormatsGeMTC` 构建我们的 `mtc.network` 对象。我们将它保存为 `network` 这一名称：
```r
network <- mtc.network(data.re  = TherapyFormatsGeMTC$data,
                       treatments = TherapyFormatsGeMTC$treat.codes)
```
将结果对象插入到 `summary` 函数中已经为我们提供了一些关于网络的有趣信息：
```r
summary(network)
```
```
## $Description
## [1] "MTC dataset: Network"
## 
## $`Studies per treatment`
## ind grp gsh tel wlc cau ush 
##  62  52  57  11  83  74  26  
## 
## $`Number of n-arm studies`
## 2-arm 3-arm 
##   181     1 
## 
## $`Studies per treatment comparison`
##     t1  t2 nr
## 1  ind tel  4
## 2  ind wlc 18
## 3  grp ind  7
## [...]
```
我们还可以使用 “`plot`” 函数来生成网络图。与由`“{netmeta}”` 包生成的网络图一样，边的粗细与我们为该比较所纳入的研究数量有关：
```r
plot(network, 
     use.description = TRUE) # Use full treatment names
```
![[Pasted image 20251114173604.png]]
我们可以试着使用**Fruchterman-Reingold algorithm**创建更好的图，这个图是具有随机性的，所以需要设置随机种子以确保结果可复现；

我们使用的R包是 **{igraph}**，A detailed description of the different styling options can be found in the online **{igraph}** [manual](https://igraph.org/r/doc/plot.common.html).

```r
library(igraph)
set.seed(12345) # set seed for reproducibility

plot(network, 
     use.description = TRUE,            # Use full treatment names
     vertex.color = "white",            # node color
     vertex.label.color = "gray10",     # treatment label color
     vertex.shape = "sphere",           # shape of the node
     vertex.label.family = "Helvetica", # label font
     vertex.size = 20,                  # size of the node
     vertex.label.dist = 2,             # distance label-node center
     vertex.label.cex = 1.5,            # node label size
     edge.curved = 0.2,                 # edge curvature
     layout = layout.fruchterman.reingold)
```
![[Pasted image 20251114200436.png]]
#### 12.3.3.3 模型编译[](https://doing-meta.guide/netwma#model-compilation)
使用我们创建的 `mtc.network`，我们可以现在制定和编译我们的模型，使用 `gemtc` 包可以自动的进行贝叶斯推论的流程，如参数先验分布的选择。
#### 12.3.3.4 Markov Chain Monte Carlo Sampling[](https://doing-meta.guide/netwma#markov-chain-monte-carlo-sampling)
#### 12.3.3.5 Assessing Model Convergence[](https://doing-meta.guide/netwma#bayesian-model-convergence)
#### 12.3.3.6 Assessing Inconsistency: The Nodesplit Method[](https://doing-meta.guide/netwma#assessing-inconsistency-the-nodesplit-method)
#### 12.3.3.7 Generating the Network Meta-Analysis Results[](https://doing-meta.guide/netwma#generating-the-network-meta-analysis-results)
### 12.3.4 Network Meta-Regression[](https://doing-meta.guide/netwma#network-meta-regression)




[^1]: Dias, Sofia, Alex J Sutton, AE Ades, and Nicky J Welton. 2013. “Evidence Synthesis for Decision Making 2: A Generalized Linear Modeling Framework for Pairwise and Network Meta-Analysis of Randomized Controlled Trials.” _Medical Decision Making_ 33 (5): 607–17.
