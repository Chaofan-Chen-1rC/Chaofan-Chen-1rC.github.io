---
title: "Bayesian meta analysis in Brms 2"
date: 2026-04-24
permalink: /pages/2026/04/Bayesian-meta-analysis-in-Brms-2/
tags:
  - meta-analysis
  - R
  - reading notes
  - book notes
---

# Bayesian Meta-Analysis with R, Stan, and brms

*Meta 分析是贝叶斯多层模型的一个特例*

## 导航

**站点导航**

- [作品（Works）](https://vuorre.com/publications.html)
- [博客（Blog）](https://vuorre.com/blog.html)
- [RSS](https://vuorre.com/blog.xml)
- [GitHub](https://github.com/mvuorre/mvuorre.github.io)
- [Bluesky](https://bsky.app/profile/matti.vuorre.com)

**文章目录**

- 引言
- 数据
- 模型
- 贝叶斯估计
- 结果比较
- 讨论

原网页侧栏还包含页面操作：
[编辑此页](https://github.com/mvuorre/mvuorre.github.io/edit/main/posts/bayesian-meta-analysis/index.qmd)、
[查看源码](https://github.com/mvuorre/mvuorre.github.io/blob/main/posts/bayesian-meta-analysis/index.qmd)、
[报告问题](https://github.com/mvuorre/mvuorre.github.io/issues/new)

## 原文信息

- 作者：[Matti Vuorre](https://orcid.org/0000-0001-5052-066X)
- 机构：[Tilburg University](https://www.tilburguniversity.edu)
- 发布日期：2016-09-29
- 标签：`r`、`statistics`
- 原文链接：[https://vuorre.com/posts/bayesian-meta-analysis/](https://vuorre.com/posts/bayesian-meta-analysis/)
- 说明：本文依据原文页面标注的 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可翻译整理，供学习与研究参考；原作者为 Matti Vuorre。

## 引言

近来，关于 Meta 分析的讨论很多。这里我想快速说明，贝叶斯多层模型完全可以很好地满足 Meta 分析的需要，而且借助 `rstan` 和 `brms` 包，在 R 中实现起来也很容易。正如你将看到的，当你无法或不愿意为 Meta 分析效应量估计值设定先验分布时，Meta 分析其实就是贝叶斯多层模型的一个特例。

这篇文章的灵感来自 Wolfgang Viechtbauer 的[网站](http://www.metafor-project.org/doku.php/tips:rma_vs_lm_lme_lmer?s%5B%5D=lme4)。在该网站中，他比较了使用他那款出色的频率学派包 [metafor](http://www.metafor-project.org/doku.php/metafor) 与多层建模“瑞士军刀” [lme4](https://cran.r-project.org/web/packages/lme4/index.html) 拟合 Meta 分析模型的结果。结果表明，尽管可以使用 `lme4` 拟合 Meta 分析模型，但它与传统 Meta 分析模型的结果略有差异，因为两者对各研究方差的处理方式略有不同。

这里是我们将使用的包：

```r
library(knitr)
library(kableExtra)
library(metafor)
library(scales)
library(lme4)
library(brms)
library(tidyverse)
```

## 数据

这里我只关注一个简单的效应量随机效应 Meta 分析，并使用前述[网站](http://www.metafor-project.org/doku.php/tips:rma_vs_lm_lme_lmer?s%5B%5D=lme4)中的同一组示例数据。该数据包含在 `metafor` 包中，用于描述尽责性与用药依从性之间的关系。效应量是由相关系数 `r` 经 `z` 转换得到的。

表 1. 示例数据（`metafor` 包中的 `dat.molloy2014`）

| study | year | ni | ri | yi | vi | sei |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Axelsson et al. (2009) | 2009 | 109 | 0.19 | 0.19 | 0.01 | 0.10 |
| Axelsson et al. (2011) | 2011 | 749 | 0.16 | 0.16 | 0.00 | 0.04 |
| Bruce et al. (2010) | 2010 | 55 | 0.34 | 0.35 | 0.02 | 0.14 |
| Christensen et al. (1995) | 1995 | 72 | 0.27 | 0.28 | 0.01 | 0.12 |
| Christensen et al. (1999) | 1999 | 107 | 0.32 | 0.33 | 0.01 | 0.10 |
| Cohen et al. (2004) | 2004 | 65 | 0.00 | 0.00 | 0.02 | 0.13 |
| Dobbels et al. (2005) | 2005 | 174 | 0.17 | 0.18 | 0.01 | 0.08 |
| Ediger et al. (2007) | 2007 | 326 | 0.05 | 0.05 | 0.00 | 0.06 |
| Insel et al. (2006) | 2006 | 58 | 0.26 | 0.27 | 0.02 | 0.13 |
| Jerant et al. (2011) | 2011 | 771 | 0.01 | 0.01 | 0.00 | 0.04 |
| Moran et al. (1997) | 1997 | 56 | -0.09 | -0.09 | 0.02 | 0.14 |
| O'Cleirigh et al. (2007) | 2007 | 91 | 0.37 | 0.39 | 0.01 | 0.11 |
| Penedo et al. (2003) | 2003 | 116 | 0.00 | 0.00 | 0.01 | 0.09 |
| Quine et al. (2012) | 2012 | 537 | 0.15 | 0.15 | 0.00 | 0.04 |
| Stilley et al. (2004) | 2004 | 158 | 0.24 | 0.24 | 0.01 | 0.08 |
| Wiebe & Christensen (1997) | 1997 | 65 | 0.04 | 0.04 | 0.02 | 0.13 |

## 模型

我们将对这些观测到的效应量及其标准误拟合一个随机效应 Meta 分析模型。参照 R 包 Metafor 手册（第 6 页）的记号，该模型可以写作：

$$y_i \sim N(\theta_i, \sigma_i^2)$$

其中，每个记录到的效应量 $y_i$ 都来自一个正态分布；该分布以该研究的“真实”效应量 $\theta_i$ 为中心，标准差等于该研究观测到的标准误 $\sigma_i$。

接下来，我们假定各研究的真实效应量近似来源于所有研究总体中的某个潜在效应量。我们将这一总体效应量记为 $\mu$，其标准差记为 $\tau$，于是各研究真实效应量满足：

$$\theta_i \sim N(\mu, \tau^2)$$

现在我们有两个值得关注的参数：$\mu$ 告诉我们，在其他条件相同的情况下，在相似研究总体中我可以期望的“真实”效应是什么；$\tau$ 告诉我们该效应在不同研究之间有多大的变异。

我认为，将这个模型写成另一个混合效应模型会更直接一些（Metafor 手册，第 6 页）：

$$y_i \sim N(\mu + \theta_i, \sigma^2_i)$$

其中，$\theta_i \sim N(0, \tau^2)$，即各研究的真实效应服从均值为 0、研究间异质性为 $\tau^2$ 的正态分布。这里稍微有点令人困惑的一点在于，我们是已知 $\sigma_i$ 的，而这正是 Meta 分析与其他更常见回归模型之间的重要区别。

### 使用 metafor 进行估计

非常简单！

```r
library(metafor)
ma_out <- rma(data = dat, yi = yi, sei = sei, slab = dat$study)
summary(ma_out)
```

```text
随机效应模型（k = 16；tau^2 估计量：REML）

  logLik  deviance       AIC       BIC      AICc
  8.6096  -17.2191  -13.2191  -11.8030  -12.2191

tau^2（估计的总异质性大小）：0.0081（SE = 0.0055）
tau（tau^2 估计值的平方根）：      0.0901
I^2（总异质性 / 总变异）：         61.73%
H^2（总变异 / 抽样变异）：         2.61

异质性检验：
Q(df = 15) = 38.1595, p-val = 0.0009

模型结果：

estimate      se    zval    pval   ci.lb   ci.ub
  0.1499  0.0316  4.7501  <.0001  0.0881  0.2118  ***

---
显著性代码：0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

## 贝叶斯估计

到目前为止，一切都很好，我们仍然完全处在标准 Meta 分析的框架内。但我想提出的是，与其使用专门的 Meta 分析软件，不如直接把上述模型视为另一个普通的回归模型，并像拟合其他任何多层回归模型那样来拟合它。也就是说，使用 [Stan](http://mc-stan.org/)，通常经由 [brms](https://cran.r-project.org/package=brms) 接口来完成。

采用贝叶斯方法后，我们可以为总体层面的参数 $\mu$ 和 $\tau$ 指定先验分布，并且通常会希望使用一些非常温和的正则化先验。这里我们使用 `brms` 的默认先验（下方输出中也会打印出来）。

### 使用 brms 进行估计

下面是使用 `brms` 拟合该模型的方法：

```r
brm_out <- brm(
  yi | se(sei) ~ 1 + (1 | study),
  data = dat,
  cores = 4,
  file = "metaanalysismodel"
)
```

```text
分布族：gaussian
链接函数：mu = identity
公式：yi | se(sei) ~ 1 + (1 | study)
数据：dat（观测数：16）
抽样：4 条链，每条 iter = 2000；warmup = 1000；thin = 1；
      warmup 后总抽样数 = 4000

先验：
Intercept ~ student_t(3, 0.2, 2.5)
<lower=0> sd ~ student_t(3, 0, 2.5)

多层超参数：
~study（水平数：16）
              Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
sd(Intercept)     0.10      0.04     0.04     0.18 1.00     1398     2247

回归系数：
          Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
Intercept     0.15      0.03     0.08     0.22 1.00     1845     1720

其他分布参数：
      Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
sigma     0.00      0.00     0.00     0.00   NA       NA       NA

样本使用 sampling(NUTS) 抽取。对每个参数而言，Bulk_ESS 与 Tail_ESS
是有效样本量指标，而 Rhat 是链拆分后的潜在尺度缩减因子
（当收敛时，Rhat = 1）。
```

这些结果与使用 `metafor` 得到的结果是相同的。请注意其中的 Student's *t* 先验分布，它们足够弥散，因此不会对后验分布施加明显影响。

## 结果比较

现在我们可以比较这两种估计方法的结果。当然，贝叶斯方法有一个巨大的优势，即它给出的不是单一数值，而是一整套合理取值的完整分布。

```text
警告：参数 'pars' 已弃用。请改用 'variable'。

警告：移除了 2 行包含缺失值或超出尺度范围的观测（`geom_bar()`）。
移除了 2 行包含缺失值或超出尺度范围的观测（`geom_bar()`）。

警告：点点记号（`..density..`）已在 ggplot2 3.4.0 中弃用。
ℹ 请改用 `after_stat(density)`。
```

![平均效应量与变异性的后验分布图](https://vuorre.com/posts/bayesian-meta-analysis/index_files/figure-html/unnamed-chunk-2-1.png)

*图 1. 平均效应量（左上）与变异性（右上）的后验分布样本直方图。左下图展示了平均效应（x 轴）与标准差（y 轴）的二维后验分布，颜色越浅表示该取值越可信。每幅图中的虚线表示极大似然点估计及其 95% 置信限（二维图中仅显示点估计）。*

从数值输出，尤其是从图形中，我们可以看到，这两种推断方式给出了相同的数值结果。不过请记住，贝叶斯估计真正允许我们讨论的是概率，而这通常也正是我们在解释结果时最想讨论的内容。

例如，平均效应量大于 0.2 的概率是多少？大约是 8%：

```r
hypothesis(brm_out, "Intercept > 0.2")
```

```text
class b 的假设检验：
             Hypothesis Estimate Est.Error CI.Lower CI.Upper Evid.Ratio Post.Prob Star
1 (Intercept)-(0.2) > 0    -0.05      0.03     -0.1     0.01       0.09      0.08
---
'CI'：单侧假设使用 90% CI，双侧假设使用 95% CI。
'*'：对于单侧假设，若后验概率超过 95%；
     对于双侧假设，若被检验值落在 95% CI 之外。
点假设的后验概率默认假定先验概率相等。
```

### 森林图

森林图展示了每个 $\theta_i$ 的完整后验分布。Meta 分析效应量 $\mu$ 也显示在最底部一行。下面我会展示相当多的代码，以便你可以基于 `brms` 的输出自行绘制森林图：

```r
library(tidybayes)
library(ggdist)
# Study-specific effects are deviations + average
out_r <- spread_draws(brm_out, r_study[study, term], b_Intercept) %>%
  mutate(b_Intercept = r_study + b_Intercept)
# Average effect
out_f <- spread_draws(brm_out, b_Intercept) %>%
  mutate(study = "Average")
# Combine average and study-specific effects' data frames
out_all <- bind_rows(out_r, out_f) %>%
  ungroup() %>%
  # Ensure that Average effect is on the bottom of the forest plot
  mutate(study = fct_relevel(study, "Average")) %>%
  # tidybayes garbles names so fix here
  mutate(study = str_replace_all(study, "\\.", " "))
# Data frame of summary numbers
out_all_sum <- group_by(out_all, study) %>%
  mean_qi(b_Intercept)
# Draw plot
out_all %>%
  ggplot(aes(b_Intercept, study)) +
  # Zero!
  geom_vline(xintercept = 0, size = .25, lty = 2) +
  stat_halfeye(.width = c(.8, .95), fill = "dodgerblue") +
  # Add text labels
  geom_text(
    data = mutate_if(out_all_sum, is.numeric, round, 2),
    aes(label = str_glue("{b_Intercept} [{.lower}, {.upper}]"), x = 0.75),
    hjust = "inward"
  ) +
  # Observed as empty points
  geom_point(
    data = dat %>% mutate(study = str_replace_all(study, "\\.", " ")),
    aes(x = yi),
    position = position_nudge(y = -.2),
    shape = 1
  )
```

```text
警告：在线条中使用 `size` 美学映射已在 ggplot2 3.4.0 中弃用。
ℹ 请改用 `linewidth`。
```

![示例模型结果的森林图](https://vuorre.com/posts/bayesian-meta-analysis/index_files/figure-html/draw-forest-plot-1.png)

*图 2. 示例模型结果的森林图。实心点和区间表示后验均值及 80%/95% 可信区间。空心点表示观测到的效应量。*

请关注 Moran et al. (1997) 的观测效应量（空心圆）：与其他所有研究相比，这是一个异常结果。你或许会把它描述为*难以置信的（incredible）*，而贝叶斯估计程序也确实做了这一点，因此其最终得到的后验分布已不再等同于观测到的效应量本身。相反，它向平均效应量发生了收缩。

再看上表，这项研究只有 56 名参与者，因此我们理应对该研究的观测效应量（ES）更加谨慎；在其他研究的背景下，我们或许也应相应调整对这项研究的判断。因此，在综合所有其他研究之后，我们对该研究效应量的最佳猜测不再是观测均值本身，而是某个更接近总体平均效应的值。

如果这种“收缩”看起来有些激进，不妨看看 Quine et al. (2012)。这项研究的样本量大得多（537），因此标准误更小；同时，它的结果也整体上更接近平均效应量估计值。因此，观测到的平均效应量与后验分布的均值几乎完全一致。这同样是一个相当理想的性质。

## 讨论

这些不同方法通常以不同名目呈现出来（回归、Meta 分析、ANOVA，等等），这很容易让像我这样的初学者“只见树木，不见森林”。我也觉得，这其实是应用统计学学生的普遍体验：每一种实验、情境和问题，似乎都对应一种不同的统计方法（更糟的是，“我该用哪种检验？”），而学生往往看不到这些方法彼此之间的联系。因此，我认为应当把注意力放在“模型（回归模型）”本身上；但现实中，人们常常更看重那种按决策树来选择统计方法的思路（McElreath，2020）。

因此，我认为我们已经陷入了一种局面：例如，Meta 分析被视为与其他建模工作相互分离，仿佛它与重复测量 *t* 检验之类的方法无关。实际上，我认为心理学中的应用统计学往往呈现为一堆彼此脱节的技巧和模型，这会导致困惑，也会降低我们正确实施合适方法的效率。

### 贝叶斯多层模型

随着我对统计学的学习不断深入，我经常注意到：某些只在特定情境下使用的技术，实际上只是某种更一般建模方法的特例。我在这里把这种一般方法称为*贝叶斯多层模型*（McElreath，2020）。如果你被迫只能选择一种统计方法来学习，那它应该是贝叶斯多层模型，因为它能让你完成并理解大多数事情，也能帮助你看到这些方法在底层机制上究竟有多么相似。

## 参考文献

McElreath, Richard. 2020. *Statistical Rethinking: A Bayesian Course with Examples in R and Stan*. Second edition. CRC Texts in Statistical Science. Boca Raton: Taylor and Francis, CRC Press.

## 复用说明

原文页面标注的复用许可为 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)。

## 引用方式

BibTeX 引用如下：

```bibtex
@online{vuorre2016,
  author = {Vuorre, Matti},
  title = {Bayesian {Meta-Analysis} with {R,} {Stan,} and Brms},
  date = {2016-09-29},
  url = {https://vuorre.com/posts/bayesian-meta-analysis/},
  langid = {en}
}
```

如需署名引用，原文建议按如下方式引用：

Vuorre, Matti. 2016. “Bayesian Meta-Analysis with R, Stan, and Brms.” September 29, 2016. [https://vuorre.com/posts/bayesian-meta-analysis/](https://vuorre.com/posts/bayesian-meta-analysis/).
