---
title: "Bayesian Meta-analysis Notes"
permalink: /notes/bayesian-meta-analysis/
layout: single
author_profile: false
toc: true
toc_sticky: true
classes: wide
---

- 需要自己指定先验分布（prior distributions）  
- 均值分布
- tau2分布
- 抽样误差已经以seTE或vi表示

贝叶斯meta分析（Bayesian meta-analysis），基于**贝叶斯推论（Bayes’ theorem）**，与频率学派不同，我们可以设置**先验分布参数**（prior distributions），包括**无信息，弱信息与信息先验**，**标准设置为正态分布的 $$\mu$$ + Half-Cauchy distribution 的 $$\tau^2$$**。[^1]
## Bayes’ theorem：  
## the idea of prior distributions：  

## [13.1 贝叶斯层次模型 The Bayesian Hierarchical Model](https://doing-meta.guide/bayesian-ma#bayes-hierarchical-model)

贝叶斯meta分析使用的模型是 **Bayesian hierarchical model** ([Röver 2017](https://doing-meta.guide/references#ref-rover2017bayesian); [Julian Higgins, Thompson, and Spiegelhalter 2009](https://doing-meta.guide/references#ref-higgins2009re)). 参考：**(Chapter [12.3.2](https://doing-meta.guide/netwma#bayesian-net-ma-model)).**
### 模型结构和原理

在贝叶斯荟萃分析中使用的分层模型，其核心宗旨与“传统”的随机效应模型是相同的。所有荟萃分析模型都具有固有的**分层**结构（或称多层结构）：

1. **第一层：研究内部 (Within-Study Level)**
    
    - 我们假设研究 $$k$$ 中报告的观测效应值 $$\hat\theta_k$$ 是该研究中“真实”效应 $$\theta_k$$ 的估计。
    - 观测效应值 $$\hat\theta_k$$ 由于抽样误差 $$\epsilon_k$$ 而偏离 $$\theta_k$$。
    - 该层级假设 $$\hat\theta_k$$ 是从一个以 $$\theta_k$$ 为均值、以 $$\sigma_k^2$$ 为方差的总体分布中抽样得出的，通常表示为：$$\hat\theta_k \sim \mathcal{N}(\theta_k,\sigma_k^2)$$。其中 $$\mathcal{N}$$ 表示**正态**分布 (normal distribution)。
2. **第二层：研究之间 (Between-Study Level)**
    
    - 我们假设这些研究特定的真实效应值 $$\theta_k$$ 本身是从一个总体的真实效应值分布中抽样得出的。
    - 该分布的均值 $$\mu$$ 是我们想要估计的汇总（pooled）“真实”效应。
    - 该分布的方差 $$\tau^2$$ 代表研究间的异质性 (between-study heterogeneity)。
    - 该层级表示为：$$\theta_k \sim \mathcal{N}(\mu,\tau^2)$$。
    - **固定效应模型** (fixed-effect model) 是该模型的一个特例，此时我们假设研究间异质性 $$\tau^2 = 0$$，意味着所有研究共享一个单一的真实效应值（即所有研究 $$k$$ 都有 $$\theta_k = \mu$$）。

### 贝叶斯特性

上述分层公式本身并没有独特的“贝叶斯”之处。使其成为贝叶斯模型的是增加了以下方程：

$$ (\mu, \tau^2) \sim p(.) \quad \text{和} \quad \tau^2 > 0 $$

- 第一行定义了参数 $$\mu$$（汇总真实效应）和 $$\tau^2$$（研究间异质性方差）的**先验分布** (prior distributions) $$p(.)$$。
- 这允许我们**先验地**（_a priori_）指定我们认为真实的汇总效应大小 $$\mu$$ 和研究间异质性 $$\tau^2$$ 可能是什么样子，以及我们对此的确定程度。
- 贝叶斯分层模型通常假定 $$\mu$$ 和 $$\tau^2$$ 的**弱信息先验** (weakly informative priors)。
    - 例如，对于 $$\mu$$ 可以使用 $$\mu \sim \mathcal{N}(0,1)$$。
    - 对于非负的 $$\tau$$ ( $$\tau^2$$ 的平方根)，常推荐使用 **Half-Cauchy** (半柯西) 分布，因为它只对正值定义且尾部较厚，这可以表示 $$\tau$$ 值很高虽然不太可能但仍然存在可能性。例如，可以使用 $$\tau \sim \mathcal{HC}(0,0.5)$$。

在实践中，贝叶斯方法使用基于**马尔可夫链蒙特卡洛** (Markov Chain Monte Carlo, MCMC) 的抽样程序（例如 Gibbs sampling 或 No-U-Turn sampling）来估计模型参数。

### 贝叶斯方法的优势

与频率论方法相比，贝叶斯荟萃分析具有几个明显的优势：

- 贝叶斯方法可以直接对我们对 $$\tau^2$$ 估计的**不确定性**进行建模。
- 当纳入的研究数量较少时，贝叶斯方法在估计汇总效应方面可能更优越。
- 贝叶斯方法为 $$\mu$$ 和 $$\tau^2$$ 产生完整的**后验分布** (posterior distributions)。这使得能够计算 $$\mu$$ 或 $$\tau^2$$ 小于或大于某个特定值的**确切概率**。这与频率论方法形成对比，频率论方法只计算置信区间。
- 贝叶斯方法允许在计算荟萃分析时整合**先验知识**和假设。

---

_可以将分层模型想象成一个共享知识的系统：每个单独的研究（第一层）都在努力寻找自己的真实效应，但它们都被假定为从一个共同的、更大的真实效应池（第二层，均值 $$\mu$$）中汲取信息。贝叶斯方法通过纳入先验分布，允许分析师在观察任何数据之前，就对这个共同池的性质 ($$\mu$$ 和 $$\tau^2$$) 进行合理的假设。_


在第 10 章中，我们了解到，**每一种元分析模型都具有内在的“多层”结构，因而也是分层结构**：
- **在第一层**，我们有各个个体参与者。这一层的数据通常以每项研究 k 的计算效应量 $$\hat\theta_k$$ 的形式呈现给我们。我们假定参与者在**第二层**内嵌于各项研究之中，并且我们元分析中不同研究的真实效应量 $$\theta_k$$ 遵循其自身的分布。这种真实效应的分布具有均值 $$\mu$$（我们想要估计的合并“真实”效应）和方差 $$\tau^2$$，代表研究间的异质性。

- 让我们尝试将其形式化。在第一层面上，我们假设研究 k 中报告的观测效应量 $$\hat\theta_k$$ 是该研究中“真实”效应 $$\theta_k$$ 的估计值。观测效应 $$\hat\theta_k$$ 与 $$\theta_k$$ 之间的偏差是由于抽样误差 $$\epsilon_k$$ 引起的。这是因为我们假定 $$\hat\theta_k$$ 是从研究 $$k$$ 所基于的总体中抽取（抽样）得到的。这个总体可以被视为一个均值为 $$\theta_k$$（即该研究的“真实”效应）且方差为 $$\sigma^2$$ 的分布。

- 在第二步中，我们假设真实的效应量 $$\theta_k$$ 实际上只是一个更大范围的真实效应量分布的样本值。这个分布的均值 $$\mu$$ 就是我们想要估计的综合效应量。每个研究中的真实效应值 $$\theta_k$$ 与 $$\mu$$ 存在差异，这是因为整个分布还具有方差 $$\tau^2$$，这表明了不同研究之间的异质性。**综合起来，这就形成了以下这两个方程：**
$$\begin{align} \hat\theta_k &\sim \mathcal{N}(\theta_k,\sigma_k^2) \notag \\ \theta_k &\sim \mathcal{N}(\mu,\tau^2) \tag{13.1} \end{align}$$

在此，我们使用 $$\mathcal{N}$$ 来表示左侧的参数是来自正态分布的样本。有人可能会认为对于第二个方程来说，这一假设过于严格（朱利安·希金斯、汤普森和斯皮格尔哈特 2009 年）。但正如之前所提到的，此处所示的这种表述方式是大多数情况下所采用的。正如之前所述，固定效应模型只是这种模型的一个特殊情况，在 这种模型中我们假设 $$\tau^2 = 0$$，这意味着不存在研究间的异质性，并且所有研究都共享一个单一的真实效应量（即对于所有研究 k：$$\theta_k = \mu$$）。

**我们还可以通过采用边际形式来简化这个公式：**
$$\begin{equation} \hat\theta_k \sim \mathcal{N}(\mu,\sigma_k^2 + \tau^2) \tag{13.2} \end{equation}$$

你可能已经发现这些公式看起来很像我们在讨论**随机效应**（第4.1.2章）和三水平元分析（第10.1章）模型时定义的公式。事实上，这个公式并没有什么特别的“贝叶斯”。然而，当我们**添加以下方程**（Williams, Rast, and Burkner 2018）时，**情况就会发生变化**：

$$\begin{align} (\mu, \tau^2) &\sim p(.) \notag \\ \tau^2 &> 0 \tag{13.3} \end{align}$$

第一行尤为重要。它为参数 $$\mu$$ 和 $$\tau^2$$ 定义了**先验分布参数**。这使我们能够事先确定我们认为真实的合并效应量 $$\mu$$ 和研究间的异质性 $$\tau^2$$ 可能呈现的形态，以及我们对此的确定程度。第二个方程添加了一个约束条件，即**研究间的异质性方差必须大于零**。然而，这个公式并未明确指定用于 $$\mu$$ 和 $$\tau^2$$ 的具体类型的先验分布。它只是告诉我们**假设**使用了某种先验分布。我们将在**后面更详细地探讨贝叶斯元分析模型中合理的、具体的先验分布参数**。

主要涉及 **MCMC** 采样，比如 **Gibbs sampling** 。在 $${brms}$$ 包中，我们将在本章中使用，所谓的**No-U-Turn采样**（NUTS, Hoffman and Gelman 2014）。

> **为什么使用贝叶斯呢？**

**原因是贝叶斯元分析有一些明显的优势：**
- 贝叶斯方法允许在我们对$$\tau^2$$的估计中直接建模不确定性（**更稳定，结合先验和后验给出一个区间内的概率**）。
- 它们在估计综合效应方面也有优势，特别是当纳入的研究数量很少时（这在实践中很常见）。
- 贝叶斯方法能够为 $$\mu$$ 和 $$\tau^2$$ 生成完整的后验分布。这使得我们能够计算出 $$\mu$$ 或 $$\tau^2$$ 小于或大于某个特定值的确切概率。这与频率主义方法形成对比，在频率主义方法中，我们仅计算置信区间。然而，（95%）置信区间只是表明，如果数据采样重复进行无数次，总体参数（如 $$\mu$$ 或 $$\tau^2$$）的真实值会在置信区间范围内出现的概率为 95%。**它们并不能告诉我们真实参数位于两个指定值之间的概率**。
- 贝叶斯方法允许我们在计算元分析时整合 **prior knowledge**和 **assumptions**
## [13.2 Setting Prior Distributions](https://doing-meta.guide/bayesian-ma#priors)

Generally, a good approach is to use **weakly informative** priors ([Williams, Rast, and Bürkner 2018](https://doing-meta.guide/references#ref-williams2018bayesian))。**弱信息先验**可以与非信息先验进行对比。非信息先验是先验分布的最简单形式。它们通常基于均匀分布，用来表示所有的值都是同样可信的。非信息先验是先验分布的最简单形式。它们通常基于均匀分布，用来表示所有的值都是同样可信的。另一方面，弱信息先验概率则要复杂一些。它们依赖于某种分布，这种分布表明我们对某些值的可信度高于其他值持有较弱的信念。然而，他们仍然没有对从数据中估计的参数值做出任何具体的陈述。

直觉上，这很有意义。例如，在许多元分析中，假设真正的效果介于 $$SMD = -2.0和2.0$$ 之间似乎是合理的，但不太可能是 $$SMD = 50$$。基于这个基本原理，我们的$$\mu$$先验的一个好的起点可能是均值为0，方差为1的正态分布（反映了我们的假设，即真实效果最有可能接近于零，但可能会变化）。**这意味着我们给予一个大约95%的先验概率，真实的合并效应大小 $$\mu$$ 介于- 2.0和2.0之间**：

$$\begin{equation} \mu \sim \mathcal{N}(0,1) \tag{13.4} \end{equation}$$

我们必须指定的下一个先验是 $$\tau^2$$ 这个有点困难，因为我们知道 $$\tau^2$$ 应始终为非负数，但可以（接近）零。这种情况的推荐分布，通常用于方差，例如 $$\tau^2$$ 是半柯西先验。 Half-Cauchy 是 Cauchy 的一种特例，它仅针对分布的一个 “一半”（即正侧）定义。

Half-Cauchy 分布由两个参数控制。第一个是 location 参数 $$x0$$，它指定分布的峰值。第二个是$$s$$、scaling 参数。它控制分布的尾部有多严重（即它“扩散”到更高的值多少）。半柯西分布用 $$HC(x0,s)$$。
![Pasted image 20251115093713.png](figures/Pasted%20image%2020251115093713.png)
上面的图表显示了不同s值的半柯西分布，其中$$x_0$$的值固定为0

半柯西分布通常具有较为厚重的尾部特征，这使其特别适合作为 $$\tau$$ 的先验分布。厚重的尾部特性意味着我们仍会赋予 $$\tau$$ 的较高值一定的概率，同时又假定较低的值更有可能出现。

在许多元分析中，$$\tau$$（$$\tau^2$$ 的平方根）大约在 $$0.3$$ 左右，或者至少处于相近的范围内。因此，为了指定半柯西先验分布，我们可以使用 s = 0.3。这确保了小于 $$\tau$$ = 0.3 的值有 50%的概率（威廉姆斯、拉斯特和布尔克纳 2018 年）。我们可以使用 `{extraDistr}` 包中的 `phcauchy` 函数实现的半柯西分布函数来验证这一点（沃洛德佐克 2020 年）。
```r
library(extraDistr)
phcauchy(0.3, sigma = 0.3)
```

然而，这已经是一个关于 $$\tau$$ 真实值的相当具体的假设了。一种更为保守的方法（我们将在此实际示例中采用这种方法）是将 s 设为 0.5；这样会使分布变得更平缓。一般来说，建议在进行分析时始终使用不同的先验设定来进行敏感性分析，以检查这些设定是否会对结果产生显著影响。**以 s = 0.5 作为半柯西分布的参数，我们可以这样写出我们的 $$\tau$$ 先验分布**：
$$\begin{equation} \tau \sim \mathcal{HC}(0,0.5) \tag{13.5} \end{equation}$$

**注意：** 贝叶斯随机效应 meta 分析：永远给 $$\tau$$（标准差）设先验，而不是 τ²（方差）。
现在我们可以把层次模型的公式和上述的规范放在一起。这就引出了我们**可以用于贝叶斯元分析的完整模型**：

$$\begin{align} \hat\theta_k &\sim \mathcal{N}(\theta_k,\sigma_k^2) \notag \\ \theta_k &\sim \mathcal{N}(\mu,\tau^2) \notag \\ \mu &\sim \mathcal{N}(0,1) \notag \\ \tau &\sim \mathcal{HC}(0,0.5) \tag{13.5} \end{align}$$
**也就是**：观测到的效应量 $${\theta}_k = \mu + \tau^2 + \epsilon$$ 
指定**研究间异质性 $$\tau$$ 是half-Cauchy distribution**，个体水平的抽样误差已经是以seTE表明了，估计的 $$\mu$$ 可以指定一个正态分布的0，误差为1，即使用的是弱先验。
## [13.3 Bayesian Meta-Analysis in _R_](https://doing-meta.guide/bayesian-ma#bayes-ma-R)

**下面大部分时间在R中操作**  

既然我们已经为我们的荟萃分析定义了贝叶斯模型，现在就该在 R 语言中将其实现出来了。在这里，我们使用 `{brms}` 包（Bürkner 2017b，2017a）来拟合我们的模型。`{brms}` 包是一个非常灵活且强大的工具，可用于拟合贝叶斯回归模型。它可用于各种各样的应用，包括多层（混合效应）模型、广义线性模型、多元模型和广义加性模型等等，这只是其中的一部分。大多数这些模型都需要个体层面的数据，**但 `{brms}` 也可以用于荟萃分析**，在这种分析中我们处理的是（加权的）研究层面的数据 。

在开始拟合模型之前，我们首先**需要安装并加载 {brms} 包**：
```r
library(brms)
```

### [13.3.1 Fitting the Model](https://doing-meta.guide/bayesian-ma#fitting-the-model)

> **基本流程**
> 1. 设置先验
> 2. 拟合模型
> 3. 检查模型拟合
> 	- Rhat
> 	- ppcheck
> 4. 生成后验分布密度图
> 5. 查看特定smd下的概率
> 6. 整理模型结果
> 7. 绘制森林图


在我们的实践示例中，我们再次使用了 `ThirdWave` 数据集，该数据集包含了来自一项关于“第三波”心理疗法对大学生影响的元分析的研究信息（第 4.2.1 章）。***当准备自己的数据时：*** 需要提前计算好TE和seTE。

在拟合模型之前，让我们首先指定总体效应量 $$\mu$$ 和研究间异质性 $$\tau$$ 的先验分布。此前，我们已定义:
$$\begin{align}\mu \sim \mathcal{N}(0, 1)\end{align}$$ 以及 $$\tau \sim \mathcal{HC}(0，0.5)$$

我们可以使用先验函数来指定这些分布。该函数有两个参数。在第一个参数中，我们指定我们想假设的先验分布，包括分布参数。在第二个参数中，我们必须定义先验的类别。对于 $$\mu$$，合适的类别是 $$Intercept$$，因为它是一个固定的人群层面效应。对于 $$\tau$$，类别是 $$sd$$，因为它是一个方差（或者更确切地说，是一个标准差）。***我们可以使用先验函数分别定义这两个先验***，然后将它们连接起来，并将结果对象保存为 $$priors$$。
```r
priors <- c(prior(normal(0,1), class = Intercept),
            prior(cauchy(0,0.5), class = sd))
```

现在，我们可以继续进行模型的构建了。为此，**我们使用了 {brms} 库中的 `brm` 函数**。该函数有许多参数，但对我们而言，只有少数几个是重要的。

在公式的参数中，指定了模型的公式。{brms} 包使用了一种**回归公式表示法**，在这种表示法中，结果（在我们的例子中，是一个观测到的效果大小）**y 是由一个或多个预测变量 x 来预测的**。一个波浪号（~）用于表示存在预测关系：y ~ x。
荟萃分析具有一定的特殊性，因为我们没有一个变量能够预测效应大小（除非我们在进行荟萃回归时这样做）。**这意味着 x 必须替换为 1，表示一个仅包含截距项的模型**。此外，**我们不能直接使用每个研究的效应大小作为 y 的值。我们还必须给具有更高精度（即样本量）的研究赋予更大的权重**。这可以通过使用 **`y|se(se_y)`** 而不是仅仅使用 y 来实现，其中 `se(se_y)` 部分代表我们数据集中每个效应大小 y 的标准误差。

如果我们想要采用随机效应模型，那么最后一步就是在公式右侧添加一个随机效应项（1|study）。这表示 y 中的效果值被假定为嵌套在研究之中，而这些研究中的真实效果本身则是从一个总体的真实效果集中随机抽取的。如果我们要使用固定效应模型，就可以直接省略这个项。随机效应模型的通用完整公式如下：`y|se(se_y) ~ 1 + (1|随机)`。要了解更多关于 brm 模型中公式设置的信息，您可以在控制台中输入 **？brmsformula** 来打开相关文档。

其他参数的设定较为简单明了。在“prior”部分，我们明确了要为我们的模型设定的先验概率。在我们的示例中，我们可以直接将之前创建的先验对象代入其中。“iter”参数则指定了马尔可夫链蒙特卡罗算法的迭代次数。模型越复杂，这个数字就应该越大。然而，迭代次数越多，函数完成运算所需的时间也就越长。最后，我们还必须指定“data”，在这里我们只需提供数据集的名称即可。
***我们将已拟合好的贝叶斯元分析模型保存为名为“m.brm”的文件。其代码如下所示***：
```r
m.brm <- brm(TE|se(seTE) ~ 1 + (1|Author),
             data = ThirdWave,
             prior = priors,
             iter = 4000)
```

>如果你用 `metafor::escalc()` 得到 `yi`/`vi`，在 `brms` 里写 `yi | se(sqrt(vi)) ~ 1 + (1|study)` 即可。  
>验证一下，是不是sei = 上述公式换算的se

模型拟合的时间会比频率学的长，耐心等待几分钟。
### [13.3.2 Assessing Convergence & Model Validity](https://doing-meta.guide/bayesian-ma#assessing-convergence-model-validity)
分析结果之前需要确保模型收敛（MCMC采样找到了最优解）。
more iterations (`iter`).可以解决模型不收敛的问题。
为了评估模型是否收敛，以及是否有效，我们做两件事情：
1. 检查参数估计的 $$\hat{R}$$ 值
	1. $$\hat{R}$$ 值是**Potential Scale Reduction Factor** (PSRF)）潜在尺度收缩因子Chapter [12.3.3.5](https://doing-meta.guide/netwma#bayesian-model-convergence)，判断标准是低于**1.01**
	2. ```r
	   summary(m.brm)
	   ```
```
## Family: gaussian 
# > summary(m.brm)
# Family: gaussian 
# Links: mu = identity; sigma = identity 
# Formula: TE | se(seTE) ~ 1 + (1 | Author) 
# Data: ThirdWave (Number of observations: 18) 
# Draws: 4 chains, each with iter = 4000; warmup = 2000; thin = 1;
# total post-warmup draws = 8000
# 
# Multilevel Hyperparameters:
#   ~Author (Number of levels: 18) 
# Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
# sd(Intercept)     0.29      0.10     0.12     0.51 1.00     2406     3350
# 
# Regression Coefficients:
#   Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
# Intercept     0.57      0.10     0.39     0.77 1.00     3913     3053
# 
# Further Distributional Parameters:
#   Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
# sigma     0.00      0.00     0.00     0.00   NA       NA       NA
# 
# Draws were sampled using sampling(NUTS). For each parameter, Bulk_ESS
# and Tail_ESS are effective sample size measures, and Rhat is the potential
# scale reduction factor on split chains (at convergence, Rhat = 1).
# > 
```
- 小于1再解释结果，不行就增加iter的次数
2. 后验预测检查
	1. 如果数据拟合的比较好，后验的一些随机抽样应该是和观测到的估计趋势相符，我们可以用下述代码检查这一点：
	2. ```r
	   pp_check(m.brm)
	   ```
![Pasted image 20251115165052.png](figures/Pasted%20image%2020251115165052.png)
### [13.3.3 Interpreting the Results](https://doing-meta.guide/bayesian-ma#interpreting-the-results)

看 **Group-Level Effects（Multilevel Hyperparameters）**，这个结果代表了我们拟合的随机效应的结果，效应量嵌套在单个研究中，而研究间是存在异质性的。~Author （n = 18）代表了每个研究都有自己的真实值，我们只是抽样的这些观测到的带有研究间异质性的一个均值估计 $${\mu}$$ 。研究间异质性的估计值 `sd(Intercept)` 为 $$\tau = 0.29$$，这与我们最初设定先验值时的“最佳猜测”非常接近。通过 ranef 函数，我们还可以提取出每个研究的“真实”效应量与合并效应量之间的估计偏差：

***研究间异质性 sd（截距）*** 的估计值为 τ= 0.29。0.29，因此与我们设置先验时的初始 “最佳猜测 ”非常接近。
![Pasted image 20251116220616.png](figures/Pasted%20image%2020251116220616.png)

使用 **ranef** 函数，我们还可以提取每项研究的 “真实 ”效应大小与集合效应的估计偏差（***每个研究相对于整体均值 μ 的“偏移量”***）：即mu为0.57，Call et al.就是0.57 + 0.06828349。

```r
ranef(m.brm)
```
```
# $Author
# , , Intercept
# 
#                           Estimate Est.Error         Q2.5       Q97.5
# Call et al.             0.06828349 0.1983268 -0.320593098  0.47551679
# Cavanagh et al.        -0.14058214 0.1767110 -0.510772720  0.19742287
# DanitzOrsillo           0.48140015 0.2868260 -0.007816358  1.09853352
# de Vibe et al.         -0.31765641 0.1438968 -0.609201679 -0.04047291
# Frazier et al.         -0.11375961 0.1495281 -0.419185805  0.17194045
# Frogeli et al.          0.03738813 0.1706535 -0.303662200  0.37204036
# Gallego et al.          0.08882019 0.1838263 -0.263123264  0.46186364
# Hazlett-Stevens & Oren -0.02759904 0.1782860 -0.389536155  0.31632311
# Hintz et al.           -0.20330035 0.1679345 -0.556108193  0.10940672
# Kang et al.             0.28359611 0.2443915 -0.142962095  0.81216193
# Kuhlmann et al.        -0.30532451 0.1889994 -0.693266573  0.03480816
# Lever Taylor et al.    -0.10713962 0.1896207 -0.506374259  0.25667118
# Phang et al.           -0.02253504 0.1917887 -0.414563484  0.35306199
# Rasanen et al.         -0.07658234 0.1954864 -0.483310211  0.29392009
# Ratanasiripong         -0.02449663 0.2252375 -0.472122309  0.42968417
# Shapiro et al.          0.39639818 0.2545663 -0.042040744  0.94894863
# Song & Lindquist        0.02124340 0.1810978 -0.333594946  0.38333853
# Warnecke et al.         0.01420085 0.1966047 -0.383552034  0.39706551
```
输出的下一部分我们可以解释的是“Population-Level Effects”（群体水平效应）,`现在变成了Regression Coefficients`。***这一部分代表我们建模的“固定”群体参数。在我们的例子中，这是μ，即我们元分析的整体效应量***。
![Pasted image 20251116220546.png](figures/Pasted%20image%2020251116220546.png)
在输出中，我们看到估计值是一个（经过偏差校正的）标准化均值差（SMD）为0.57，95%可信区间（95% CrI）范围为0.39到0.76。这表明，本元分析中研究的干预措施具有中等规模的整体效应。  

**sigma？是否代表了个体水平的抽样误差？**

由于这是一个贝叶斯模型，我们在这里找不到任何**p值**。但我们的例子应该强调，即使不依赖经典的显著性检验，我们也可以做出合理的推断。**贝叶斯元分析的一个优势是，我们可以对我们想要估计的参数进行概率建模**，而这在频率学派的元分析中是无法实现的。***贝叶斯模型不仅估计感兴趣的参数，还为τ²和μ生成整个后验分布***，我们可以非常方便地访问这些分布。我们只需要使用posterior_samples函数即可：
```r
post.samples <- posterior_samples(m.brm, c("^b", "^sd"))
names(post.samples)
```
```
[1] "b_Intercept"          "sd_Author__Intercept"
```
提取的数据框，包含两列：b_截距，即集合效应大小的后验样本数据；sd_作者_截距，即研究间异质性数据τ. 我们将这两列重新命名为 smd 和 tau，使其名称更具信息性：
```r
names(post.samples) <- c("smd", "tau")
```
使用 `post.samples` 的数据样本，我们现在可以生成**后验分布**的密度图。我们使用 `{ggplot2}` 包进行绘图：
```r
ggplot(aes(x = smd), data = post.samples) +
  geom_density(fill = "lightblue",                # set the color
               color = "lightblue", alpha = 0.7) +  
  geom_point(y = 0,                               # add point at mean
             x = mean(post.samples$smd)) +
  labs(x = expression(italic(SMD)),
       y = element_blank()) +
  theme_minimal()

ggplot(aes(x = tau), data = post.samples) +
  geom_density(fill = "lightgreen",               # set the color
               color = "lightgreen", alpha = 0.7) +  
  geom_point(y = 0, 
             x = mean(post.samples$tau)) +        # add point at mean
  labs(x = expression(tau),
       y = element_blank()) +
  theme_minimal()
```
![posterior_plot 3.png](figures/posterior_plot%203.png)
我们看到后验分布遵循单峰分布，大致为正态分布，在$$\mu$$和$$\tau$$的估计值附近达到峰值。

贝叶斯方法为我们感兴趣的参数创建了实际的抽样分布，这意味着我们可以精确计算μ或τ大于或小于某个特定值的概率。假设我们在之前的文献中发现，如果干预效果的标准化均值差（SMD）低于0.30，则效果不再具有实际意义。***因此，我们可以基于我们的模型计算元分析中真实整体效应小于SMD = 0.30的概率*** （SMCID的验证）。

这可以通过查看经验累积分布函数（ECDF）来实现。ECDF允许我们选择一个特定值X，并根据提供的数据返回某个值x小于X的概率。在我们的例子中，μ的后验分布的ECDF如下图所示：
```r
smd.ecdf <- ecdf(post.samples$smd)
smd.ecdf(0.3)
```
```
> [1] 0.002125
```
![P_smd_0.3.png](figures/P_smd_0.3.png)
我们看到，在0.21%的情况下，我们的综合效应小于0.30的概率非常非常低。假设切点值（0.3）是有效的，这意味着我们在荟萃分析中发现的干预措施的总体效果很可能是有意义的。（可以确定标准化临床最小差异（smcid），然后使用贝叶后验概率计算小于这个值的概率是多少，如果非常小（例如0.002），可以证明我们的结果是有临床意义的）。

### [13.3.4 Generating a Forest Plot](https://doing-meta.guide/bayesian-ma#generating-a-forest-plot)
正如您所看到的，贝叶斯模型允许我们提取其抽样后验分布（即一项研究的多次抽样的分布）。这对于直接评估模型中特定值的概率非常有帮助。我们还可以利用这一特性创建**增强的森林图**（第 6 章），森林图不仅信息量大，而且赏心悦目：

遗憾的是，目前还没有可以直接从 {brms} 模型创建森林图的软件包。但是我们可以使用 `{tidybayes}` 软件包（Kay 2020）的函数来创建森林图。因此，在继续之前，让我们首先加载这个软件包和其他一些软件包：
```r
library(tidybayes)
library(dplyr)
library(ggplot2)
library(ggridges)
library(glue)
library(stringr)
library(forcats)
```
在生成森林图之前，我们必须准备好数据。特别是，**我们需要单独提取每项研究的后验分布（因为森林图也描述了每项研究的具体效应大小）**。为此，我们可以使用 {tidybayes} 软件包中的 spread_draws 函数。该函数需要三个参数作为输入：拟合的{brms}模型、对结果进行索引的随机效应因子，以及我们想要提取的参数（此处为 b_截距，因为我们想要提取固定项：效应大小）。

使用管道运算符，我们可以直接操作输出结果。使用 {dplyr} 中的 `mutate` 函数，我们将汇总效应大小 b_Intercept 加到每项研究的估计偏差中（**即我们想估计的固定效应加上研究间异质性（每个研究或者每个研究的每次取样与总体mu的偏差），对同一研究的截距不停抽样，并考虑到研究的异质性是在该群体中随机抽取的，最后展示抽样的结果，即密度曲线**），从而计算出每项研究的实际效应大小。我们将结果保存为 study.draws：
```r
study.draws <- spread_draws(m.brm, r_Author[Author,], b_Intercept) %>% 
  mutate(b_Intercept = r_Author + b_Intercept)
```
接下来，我们希望以类似的方式生成汇总效果的分布（因为在森林图中，汇总效果通常显示在最后一行）。因此，我们稍微调整了前面的代码，去掉了第二个参数，只获得了池化的效果。对mutate的调用只添加了一个名为“Author”的额外列。我们将结果保存为pool .effect.draw。
```r
pooled.effect.draws <- spread_draws(m.brm, b_Intercept) %>% 
  mutate(Author = "Pooled Effect")
```
接下来，我们结合`study.draws`和pooled.effect.在一个数据帧中一起绘制。然后我们再次启动管道，首先调用ungroup，然后使用mutate来(1)清理研究标签（即用空格替换点），(2)根据效应大小（从高到低）重新排序研究因子水平。结果是我们绘图所需的数据，我们将其保存为forest.data。
```r
forest.data <- bind_rows(study.draws, 
                         pooled.effect.draws) %>% 
  ungroup() %>%
  mutate(Author = str_replace_all(Author, "[.]", " ")) %>% 
  mutate(Author = reorder(Author, b_Intercept))
```
```
# A tibble: 152,000 × 6
# Groups:   Author [19]
# Author      r_Author .chain .iteration .draw b_Intercept
# <chr>          <dbl>  <int>      <int> <int>       <dbl>
#   1 Call.et.al.   0.231       1          1     1       0.786
# 2 Call.et.al.  -0.0303      1          2     2       0.548
# 3 Call.et.al.   0.144       1          3     3       0.694
# 4 Call.et.al.  -0.0408      1          4     4       0.646
# 5 Call.et.al.  -0.0732      1          5     5       0.517
# 6 Call.et.al.   0.0305      1          6     6       0.528
# 7 Call.et.al.  -0.0985      1          7     7       0.455
# 8 Call.et.al.   0.213       1          8     8       0.800
# 9 Call.et.al.   0.179       1          9     9       0.723
# 10 Call.et.al.   0.0439      1         10    10       0.673
# ℹ 151,990 more rows
# ℹ Use `print(n = ...)` to see more rows
```
最后，森林图还应该显示每个研究的效应大小（SMD和可信区间）。为此，我们使用了新生成的森林。data数据集，按Author分组，***然后使用`mean_qi`函数计算这些值***。我们将输出保存为forest.data.summary：
`mean_qi(b_Intercept)`这是 `{tidybayes}` 包的函数：`mean_qi()` 会对每组的 `b_Intercept` 进行： 
- **mean()**：后验分布的平均值
- **qi()**：计算后验的**credible interval**（默认是 95%，即 2.5% ~ 97.5%）
```r
forest.data.summary <- group_by(forest.data, Author) %>% 
  mean_qi(b_Intercept)
```
现在我们准备使用`{ggplot2}`包生成森林图。生成该图的代码如下所示：
```r
forest.data.summary <- group_by(forest.data, Author) %>% 
  mean_qi(b_Intercept)

ggplot(aes(b_Intercept, 
           relevel(Author, "Pooled Effect", 
                   after = Inf)), 
       data = forest.data) +
  
  # Add vertical lines for pooled effect and CI
  geom_vline(xintercept = fixef(m.brm)[1, 1], 
             color = "grey", size = 1) +
  geom_vline(xintercept = fixef(m.brm)[1, 3:4], 
             color = "grey", linetype = 2) +
  geom_vline(xintercept = 0, color = "black", 
             size = 1) +
  
  # Add densities
  geom_density_ridges(fill = "darkgray", 
                      rel_min_height = 0.01, 
                      col = NA, scale = 1,
                      alpha = 0.8) +
  geom_pointintervalh(data = forest.data.summary, 
                      size = 1) +
  
  # Add text and labels
  geom_text(data = mutate_if(forest.data.summary, 
                             is.numeric, round, 2),
            aes(label = glue("{b_Intercept} [{.lower}, {.upper}] "), 
                x = Inf), hjust = "inward") +
  labs(x = "Standardized Mean Difference", # summary measure
       y = element_blank()) +
  theme_bw()
```
![Pasted image 20251116231904.png](figures/Pasted%20image%2020251116231904.png)
需要注意的是，新版本的tidybayes已经不支持 `geom_pointintervaljh`，需要指定为 `geom_pointinterval`，并设置：
```r
geom_pointinterval(data = forest.data.summary, 
                     aes(xmin = .lower, xmax = .upper),
                     size = 1)
```
完整绘图代码如下：
```r
ggplot(aes(b_Intercept, 
           relevel(Author, "Pooled Effect", 
                   after = Inf)), 
       data = forest.data) +
  
  # Add vertical lines for pooled effect and CI
  geom_vline(xintercept = fixef(m.brm)[1, 1], 
             color = "grey", size = 1) +
  geom_vline(xintercept = fixef(m.brm)[1, 3:4], 
             color = "grey", linetype = 2) +
  geom_vline(xintercept = 0, color = "black", 
             size = 1) +
  
  # Add densities
  geom_density_ridges(fill = "darkgray", 
                      rel_min_height = 0.01, 
                      col = NA, scale = 1,
                      alpha = 0.8) +
  geom_pointinterval(data = forest.data.summary, 
                     aes(xmin = .lower, xmax = .upper),
                     size = 1) +
  
  # Add text and labels
  geom_text(data = mutate_if(forest.data.summary, 
                             is.numeric, round, 2),
            aes(label = glue("{b_Intercept} [{.lower}, {.upper}] "), 
                x = Inf), hjust = "inward") +
  labs(x = "Standardized Mean Difference", # summary measure
       y = element_blank()) +
  theme_bw()
```
> **观察到的效应量与基于模型的效应量**
> 这里有一件非常重要的事情需要提及。森林图中所显示的影响大小并不代表原始研究中实际观察到的影响大小（TE，例如DanitzOrsillo的原始TE为1.7911700，估计后为1.05），而是基于贝叶斯模型对一项研究的“真实”影响大小（$${\theta}_k$$）的估计值。森林图中显示的点相当于我们在使用 `ranef` 提取随机效应时所看到的针对每个研究的估计值（只是这些值是围绕合并效应进行中心化的）。  
> 此外，观察那些影响大小非常大的研究（例如“丹尼茨·奥尔西洛”和“夏皮罗等人”这样的异常值），我们会发现基于模型的影响大小比其最初观察到的值更接近总体效应 $$\hat\mu$$。  
> 这种向均值的收缩是具有共同总体分布的分层模型（如元分析随机效应模型）的典型特征。在估计过程中，贝叶斯模型会将关于研究 k 的影响效应的信息与所有元分析中估计的 K 个影响大小的总体分布信息相结合，从而“补充”相关信息。这种“力量借用”现象意味着具有极端效应的研究结果会向平均值靠拢（卢恩等人，2012 年，第 10.1 章）。对于那些提供的信息相对较少（即标准误差较大的研究）而言，这种现象更为明显。

> **为什么会发生 shrinkage？——“借力（borrowing of strength）”**
> - 单看某一篇研究：如果它样本量很小 / 标准误很大，它自己的信息其实很“虚”，非常不稳定。
> - 贝叶斯分层模型不会“完全相信”这一篇的 yₖ，而是做了一个折中：  
> - $$\hat{\theta}_k\approx w⋅yk+(1−wk)⋅μ$$  
> - wₖ：一个权重，取决于 σₖ² 和 τ²
> - 如果这个研究非常不精确（σₖ 很大），wₖ 就变小，说明：  
> - “我们少信一点这篇自己的数据，多信一点整体 meta 分析得出的分布”  
> - 这就是所谓 **“borrowing of strength（借力）”**：单个研究的信息被“其它研究的整体分布”所补充、修正。
> - 即让效力更强的研究稀释一下误差比较大的研究，往回也就是往均值的方向拉一些。
## [13.4 Questions & Answers](https://doing-meta.guide/bayesian-ma#questions-answers-12)
## [13.5 Summary](https://doing-meta.guide/bayesian-ma#summary-12)

- 贝叶斯meta分析基于贝叶斯层次模型，该模型的核心原则与“传统的”随机效应模型相同（即抽样误差，研究间异质性），区别在于我们会对mu和tau2给一个（无信息，弱信息，信息）先验分布。
- 最好用弱信息先验，我们有小把握，一个值（范围可能更加可信比其他值（范围））。
- 为了确定不同研究之间异质性方差 $$\tau^2$$ 的先验分布，可以使用**半柯西分布**。半柯西分布特别适用于此任务，因为它们仅定义于正值范围内，并且具有更重的尾部特征。这可以用来表示 $$\tau^2$$ 的极高值不太可能发生，但仍然有可能出现。
- 在拟合贝叶斯元分析模型时，重要的是：(1)始终检查模型是否包含足够的迭代以收敛（例如通过检查$$\hat{R}$$值），以及(2)使用不同的先验规范进行敏感性分析，以评估对结果的影响。

## 14 有不懂的你就看这里
1. [Bayesian meta-analysis in brms | A. Solomon Kurz](https://solomonkurz.netlify.app/blog/bayesian-meta-analysis/)（Bayesian meta-analysis in brms 笔记）
2. [Chapter 13 Bayesian Meta-Analysis | Doing Meta-Analysis in R](https://doing-meta.guide/bayesian-ma#fitting-the-model)
3. [LaTeX 代码](https://chatgpt.com/c/69168707-3c84-832e-ac17-732409bed2a4)
4. [brms: Bayesian Regression Models using 'Stan'](https://cran.r-project.org/web/packages/brms/brms.pdf?utm_source=chatgpt.com)

## 15 借鉴文献
1. [和DwR的教程很相似的部分，绘制了总体效应量 和 研究间异质性，以及研究内的异质性的后验分布，考虑到了效应量间的异质性（三水平层次模型）；计算了累计概率曲线；研究效应量提取也可借鉴](https://www.nature.com/articles/s44271-024-00124-2)
	- [原文PDF](references/%E8%B4%9D%E5%8F%B6%E6%96%AF%E8%8C%83%E6%96%87.pdf)
![Pasted image 20251117014856.png](figures/Pasted%20image%2020251117014856.png)
2. Advanced Bayesian Multilevel Modeling with the R Package brms

#meta 


[^1]: The ﬁnal level of the model contained **weakly informative priors**. A standard normal prior was used for the pooled effect, while the prior for **τ2 was a Half-Cauchy distribution with location and scale parameters set to 0 and 0.5**, respectively.  
原文：*A systematic review and Bayesian metaanalysis provide evidence for an effect of acute physical activity on cognition in young adults*
