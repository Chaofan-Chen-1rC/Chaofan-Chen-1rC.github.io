---
title: "Doing Meta-Analysis in R | Note 1: Welcome!"
date: 2026-04-17
permalink: /reading-notes/dmar-note-01/
classes: wide
tags:
  - meta-analysis
  - R
  - reading notes
  - doing-meta-analysis-in-r
---

# 欢迎！

来源页面：[https://doing-meta.guide/](https://doing-meta.guide/)

以下内容为页面可见内容的中文译文，并尽量保留了原页面的结构与链接。

<aside class="sidebar__right">
  <nav class="toc">
    <header>
      <h4 class="nav__title"><i class="fa fa-file-text"></i> 本页内容</h4>
    </header>
    <ul class="toc__menu" id="markdown-toc">
      <li><a href="#welcome" id="markdown-toc-welcome">欢迎！</a></li>
      <li><a href="#open-source-repository" id="markdown-toc-open-source-repository">开源仓库</a></li>
      <li><a href="#how-to-use-the-guide" id="markdown-toc-how-to-use-the-guide">如何使用本指南</a></li>
      <li><a href="#contributing" id="markdown-toc-contributing">参与贡献</a></li>
      <li><a href="#citing-this-guide" id="markdown-toc-citing-this-guide">引用本指南</a></li>
      <li><a href="#cite-the-packages" id="markdown-toc-cite-the-packages">请同时引用相关软件包</a></li>
    </ul>
  </nav>
</aside>

<a id="welcome"></a>
## 欢迎！

欢迎来到 **《使用 R 进行 Meta 分析：实操指南》**（*Doing Meta-Analysis with R: A Hands-On Guide*）的在线版本。

本书旨在以一种易于上手的方式介绍如何在 *R* 中开展 Meta 分析。书中涵盖了 Meta 分析的关键步骤，包括结局指标的合并、森林图、异质性诊断、亚组分析、Meta 回归、控制发表偏倚的方法，以及偏倚风险评估和绘图工具。

书中也讨论了若干更高级但同样非常重要的主题，例如网络 Meta 分析、多层级或三层级 Meta 分析、贝叶斯 Meta 分析方法，以及结构方程模型（SEM）Meta 分析。

本书涉及的编程与统计背景知识保持在**非专家**水平。该书的**纸质版**已由 [Chapman & Hall/CRC Press](https://www.routledge.com/Doing-Meta-Analysis-with-R-A-Hands-On-Guide/Harrer-Cuijpers-Furukawa-Ebert/p/book/9780367610074)（Taylor & Francis）出版。

<a id="open-source-repository"></a>
## 开源仓库

本书使用 [**{rmarkdown}**](https://rmarkdown.rstudio.com/docs/) 和 [**{bookdown}**](https://bookdown.org/) 构建，公式则通过 [MathJax](http://docs.mathjax.org/en/latest/index.html) 渲染。我们用于编译本指南的全部材料和源代码都可以在 **GitHub** 上找到。你可以自由地 fork、分享并复用其中的内容。不过，这个仓库主要定位为“只读”；通常不会接受 PR（如需联系我们，请参见下文以及前言中的说明）。

[查看 GitHub 仓库](https://github.com/MathiasHarrer/Doing-Meta-Analysis-in-R)

<a id="how-to-use-the-guide"></a>
## 如何使用本指南

本教程简要介绍了这份指南，以及如何将它用于你自己的 Meta 分析项目。

[YouTube 视频教程](https://www.youtube.com/embed/i1b5c-dVfkU)

<a id="contributing"></a>
## 参与贡献

本指南是一个开源项目，我们特别感谢那些在本指南部分章节中提供额外内容的专业贡献者。

- [**Luke A. McGuinness**](https://twitter.com/mcguinlu)，布里斯托大学：第 15 章《偏倚风险图》。

如果你也想为本指南贡献内容，欢迎给 **Mathias**（mathias.harrer@fau.de）发送电子邮件，告诉我们你计划补充的内容。

<a id="citing-this-guide"></a>
## 引用本指南

建议的引用格式如下：

> Harrer, M., Cuijpers, P., Furukawa, T.A., & Ebert, D.D. (2021). *Doing Meta-Analysis with R: A Hands-On Guide*. Boca Raton, FL and London: Chapman & Hall/CRC Press. ISBN 978-0-367-61007-4.

可将该参考文献下载为 [BibTeX](https://www.protectlab.org/meta-analysis-in-r/data/citation.bib) 或 [.ris](https://www.protectlab.org/meta-analysis-in-r/data/citation.ris)。

<a id="cite-the-packages"></a>
## 请同时引用相关软件包

在本指南中，我们介绍并使用了多个 *R* 软件包。我们之所以能够免费使用这些软件包，是因为世界各地的专家通常在无偿的情况下投入了大量时间与精力进行开发。如果你在自己的 Meta 分析中使用了本书提到的某些软件包，我们强烈建议你也在报告中引用这些软件包。

在本指南中，每当引入一个新软件包时，我们也会提供可用于引用它的参考文献信息。你也可以运行 `citation("package")` 来获取推荐的引用条目。谢谢！

《Doing Meta-Analysis in R: A Hands-on Guide》由 Mathias Harrer、Pim Cuijpers、Toshi A. Furukawa 和 David D. Ebert 撰写。

本书由 [bookdown](https://bookdown.org/) R 软件包构建。
