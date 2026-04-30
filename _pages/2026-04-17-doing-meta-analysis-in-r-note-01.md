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

<style>
@import url("https://code.cdn.mozilla.net/fonts/fira.css");

.dmar-bookdown {
  --background-color: #FFFEFA;
  --text-color: #000;
  --highlight-color: #277DB0;
  --grey-color: #6C6C6C;
  --line-color: #eee;
  --bg-box: #f5f5f5;
  --box-border-color: #eee;
  --code-bg: #f8f8f8;
  --footer-text: #E6E5E1;
  background: var(--background-color);
  color: var(--text-color);
  font-family: "Fira Sans", "Segoe UI", Arial, sans-serif;
  line-height: 1.6;
  margin-left: calc(50% - 50vw);
  margin-right: calc(50% - 50vw);
  padding: 0 1rem 1.5rem;
}

.dmar-bookdown * {
  box-sizing: border-box;
}

.dmar-bookdown a {
  color: var(--highlight-color);
  overflow-wrap: break-word;
  text-decoration: none;
}

.dmar-bookdown a:hover {
  text-decoration: underline;
}

.dmar-bookdown strong {
  font-weight: 500;
}

.dmar-bookdown .container-fluid {
  margin: 0 auto;
  max-width: 72rem;
}

.dmar-bookdown .row {
  align-items: start;
  column-gap: 3.25rem;
  display: grid;
  grid-template-columns: minmax(0, 47rem) minmax(14rem, 18rem);
  justify-content: center;
}

.dmar-bookdown main {
  margin-top: 1rem;
  min-width: 0;
}

.dmar-bookdown .sidebar {
  max-width: 18rem;
}

.dmar-bookdown .sidebar h1,
.dmar-bookdown .sidebar h2 {
  line-height: 1.3;
  margin: 1.5rem 0 0.5rem;
}

.dmar-bookdown .sidebar h1 {
  font-size: 1.1rem;
}

.dmar-bookdown .sidebar h2 {
  font-size: 0.9rem;
}

.dmar-bookdown .sidebar small {
  color: var(--grey-color);
}

.dmar-bookdown .sidebar ul {
  list-style: none;
  margin: 0;
  padding-left: 0;
}

.dmar-bookdown .sidebar li {
  font-size: 0.9rem;
  line-height: 1.5;
  margin-bottom: 0.5rem;
}

.dmar-bookdown .book-extra {
  border-top: 1px solid #ccc;
  font-size: 0.9rem;
  margin-top: 0.5rem;
  padding-top: 0.5rem;
}

.dmar-bookdown .book-extra p {
  margin: 0 0 0.75rem;
}

.dmar-bookdown h1,
.dmar-bookdown h2,
.dmar-bookdown h3,
.dmar-bookdown h4,
.dmar-bookdown h5 {
  line-height: 1.3;
}

.dmar-bookdown main h1 {
  font-size: 2rem;
  margin: 0 0 0.5rem;
}

.dmar-bookdown main h2 {
  font-size: 1.5rem;
  margin: 2rem 0 1rem;
}

.dmar-bookdown main p {
  margin: 0 0 1rem;
}

.dmar-bookdown main hr {
  border: 0;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
  margin: 0 0 1rem;
}

.dmar-bookdown .anchor {
  display: none;
  font-size: max(0.5em, 1rem);
  margin-left: 0.5rem;
}

.dmar-bookdown h1:hover .anchor,
.dmar-bookdown h2:hover .anchor {
  display: inline;
}

.dmar-bookdown img.cover {
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
  float: right;
  margin: 0 1rem 0 1rem;
  max-width: 45%;
  width: 250px;
}

.dmar-bookdown .repo-badge {
  height: 28px;
  max-width: 100%;
}

.dmar-bookdown .video-frame {
  margin: 1rem auto 0;
  max-width: 580px;
}

.dmar-bookdown .video-frame iframe {
  aspect-ratio: 580 / 327;
  border: 0;
  border-radius: 5px;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
  display: block;
  width: 100%;
}

.dmar-bookdown main ul {
  list-style-type: square;
  margin-bottom: 0;
  padding-left: 25px;
}

.dmar-bookdown main li {
  margin-bottom: 0.5rem;
}

.dmar-bookdown main li:first-child {
  margin-top: 0.5rem;
}

.dmar-bookdown .boxempty {
  background: var(--bg-box);
  border: 3px solid var(--box-border-color);
  border-radius: 0.25rem;
  margin-bottom: 10px;
  padding: 1em 1em 1em 1.1em;
  position: relative;
}

.dmar-bookdown .chapter-nav {
  display: flex;
  justify-content: space-between;
  margin-top: 2rem;
}

.dmar-bookdown .chapter-nav .empty,
.dmar-bookdown .chapter-nav .next {
  border: 1px solid #eee;
  border-radius: 0.2rem;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
  padding: 0.5rem;
}

.dmar-bookdown .chapter-nav .empty {
  border: 0;
  box-shadow: none;
}

.dmar-bookdown .chapter-nav .next a::after {
  content: " »";
}

.dmar-bookdown .series-return {
  border-top: 1px solid var(--line-color);
  font-size: 0.95rem;
  margin-top: 1.5rem;
  padding-top: 1rem;
}

.dmar-bookdown .reading-meta {
  border-bottom: 1px solid var(--line-color);
  color: var(--grey-color);
  font-size: 0.9rem;
  margin-bottom: 1.25rem;
  padding-bottom: 0.75rem;
}

.dmar-bookdown .reading-meta p {
  margin: 0 0 0.25rem;
}

.dmar-bookdown .sidebar-chapter {
  border-left: 1px solid var(--line-color);
  padding-left: 1rem;
}

.dmar-bookdown nav[data-toggle="toc"] .nav-link {
  display: block;
  line-height: 1.45;
  padding: 0.35rem 0.5rem;
}

.dmar-bookdown nav[data-toggle="toc"] li {
  margin-bottom: 0.25rem;
}

.dmar-bookdown nav[data-toggle="toc"] .nav-link:hover {
  text-decoration: underline;
}

.dmar-bookdown nav[data-toggle="toc"] .nav-link.active {
  background-color: #eee;
  border-radius: 0.2rem;
}

.dmar-bookdown footer {
  background: var(--highlight-color);
  color: var(--footer-text);
  font-size: 0.9rem;
  margin: 3rem -1rem 0;
}

.dmar-bookdown footer a {
  color: var(--footer-text);
  text-decoration: underline;
}

.dmar-bookdown .footer-grid {
  column-gap: 2rem;
  display: grid;
  grid-template-columns: 1fr 1fr;
  margin: 0 auto;
  max-width: 95rem;
  padding: 1rem;
}

.dmar-bookdown .footer-grid p {
  margin: 0;
}

@media (min-width: 768px) {
  .dmar-bookdown .sidebar-chapter {
    max-height: 100vh;
    overflow-y: auto;
    position: sticky;
    top: 0;
  }
}

@media (min-width: 1200px) {
  .dmar-bookdown {
    font-size: 18px;
  }
}

@media (max-width: 991.98px) {
  .dmar-bookdown .row {
    display: block;
  }

  .dmar-bookdown .sidebar {
    max-width: 100%;
  }

  .dmar-bookdown .sidebar-chapter {
    border-left: 0;
    border-top: 1px solid var(--line-color);
    margin-top: 2rem;
    padding-left: 0;
    padding-top: 1rem;
  }
}

@media (max-width: 767.98px) {
  .dmar-bookdown .sidebar-chapter {
    display: none;
  }

  .dmar-bookdown img.cover {
    display: block;
    float: none;
    margin: 0 auto 1rem;
    max-width: 250px;
    width: 80%;
  }

  .dmar-bookdown .footer-grid {
    grid-template-columns: 1fr;
    row-gap: 1rem;
  }
}
</style>

<div class="dmar-bookdown">
  <div class="container-fluid">
    <div class="row">
      <main id="content">
        <div class="reading-meta">
          <p>This is the first note in my reading series on <em>Doing Meta-Analysis in R: A Hands-on Guide</em>.</p>
          <p>来源页面：<a href="https://doing-meta.guide/">https://doing-meta.guide/</a></p>
        </div>

        <section id="section-welcome">
          <h1>欢迎！<a class="anchor" aria-label="anchor" href="#section-welcome">#</a></h1>
          <hr>
          <p><a href="https://www.routledge.com/Doing-Meta-Analysis-with-R-A-Hands-On-Guide/Harrer-Cuijpers-Furukawa-Ebert/p/book/9780367610074" target="_blank" rel="noopener"><img src="https://doing-meta.guide/images/cover.png" width="250" alt="Doing Meta-Analysis with R book cover" class="cover"></a>欢迎来到 <strong>《使用 R 进行 Meta 分析：实操指南》</strong>（<em>Doing Meta-Analysis with R: A Hands-On Guide</em>）的在线版本。</p>
          <p>本书旨在以一种易于上手的方式介绍如何在 <em>R</em> 中开展 Meta 分析。书中涵盖了 Meta 分析的关键步骤，包括结局指标的合并、森林图、异质性诊断、亚组分析、Meta 回归、控制发表偏倚的方法，以及偏倚风险评估和绘图工具。</p>
          <p>书中也讨论了若干更高级但同样非常重要的主题，例如网络 Meta 分析、多层级或三层级 Meta 分析、贝叶斯 Meta 分析方法，以及结构方程模型（SEM）Meta 分析。</p>
          <p>本书涉及的编程与统计背景知识保持在<strong>非专家</strong>水平。该书的<strong>纸质版</strong>已由 <a href="https://www.routledge.com/Doing-Meta-Analysis-with-R-A-Hands-On-Guide/Harrer-Cuijpers-Furukawa-Ebert/p/book/9780367610074">Chapman &amp; Hall/CRC Press</a>（Taylor &amp; Francis）出版。</p>
        </section>

        <section id="section-open-source-repository">
          <h2>开源仓库<a class="anchor" aria-label="anchor" href="#section-open-source-repository">#</a></h2>
          <hr>
          <p>本书使用 <a href="https://rmarkdown.rstudio.com/docs/"><strong>{rmarkdown}</strong></a> 和 <a href="https://bookdown.org/"><strong>{bookdown}</strong></a> 构建，公式则通过 <a href="http://docs.mathjax.org/en/latest/index.html">MathJax</a> 渲染。我们用于编译本指南的全部材料和源代码都可以在 <strong>GitHub</strong> 上找到。你可以自由地 fork、分享并复用其中的内容。不过，这个仓库主要定位为“只读”；通常不会接受 PR（如需联系我们，请参见下文以及前言中的说明）。</p>
          <p><a href="https://github.com/MathiasHarrer/Doing-Meta-Analysis-in-R"><img class="repo-badge" src="https://img.shields.io/badge/View%20Repository-100000?style=for-the-badge&amp;logo=github&amp;logoColor=white" alt="View Repository"></a></p>
        </section>

        <section id="section-how-to-use-the-guide">
          <h2>如何使用本指南<a class="anchor" aria-label="anchor" href="#section-how-to-use-the-guide">#</a></h2>
          <hr>
          <p>本教程简要介绍了这份指南，以及如何将它用于你自己的 Meta 分析项目。</p>
          <div class="video-frame">
            <iframe src="https://www.youtube.com/embed/i1b5c-dVfkU" title="YouTube video player" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
          </div>
        </section>

        <section id="section-contributing">
          <h2>参与贡献<a class="anchor" aria-label="anchor" href="#section-contributing">#</a></h2>
          <hr>
          <p>本指南是一个开源项目，我们特别感谢那些在本指南部分章节中提供额外内容的专业贡献者。</p>
          <ul>
            <li><a href="https://twitter.com/mcguinlu"><strong>Luke A. McGuinness</strong></a>，布里斯托大学：第 15 章《偏倚风险图》。</li>
          </ul>
          <p>如果你也想为本指南贡献内容，欢迎给 <strong>Mathias</strong>（<a href="mailto:mathias.harrer@fau.de">mathias.harrer@fau.de</a>）发送电子邮件，告诉我们你计划补充的内容。</p>
        </section>

        <section id="section-citing-this-guide">
          <h2>引用本指南<a class="anchor" aria-label="anchor" href="#section-citing-this-guide">#</a></h2>
          <hr>
          <p>建议的引用格式如下：</p>
          <div class="boxempty">
            <p>Harrer, M., Cuijpers, P., Furukawa, T.A., &amp; Ebert, D.D. (2021). <em>Doing Meta-Analysis with R: A Hands-On Guide</em>. Boca Raton, FL and London: Chapman &amp; Hall/CRC Press. ISBN 978-0-367-61007-4.</p>
          </div>
          <p>可将该参考文献下载为 <a href="https://www.protectlab.org/meta-analysis-in-r/data/citation.bib">BibTeX</a> 或 <a href="https://www.protectlab.org/meta-analysis-in-r/data/citation.ris">.ris</a>。</p>
        </section>

        <section id="section-cite-the-packages">
          <h2>请同时引用相关软件包<a class="anchor" aria-label="anchor" href="#section-cite-the-packages">#</a></h2>
          <hr>
          <p>在本指南中，我们介绍并使用了多个 <em>R</em> 软件包。我们之所以能够免费使用这些软件包，是因为世界各地的专家通常在无偿的情况下投入了大量时间与精力进行开发。如果你在自己的 Meta 分析中使用了本书提到的某些软件包，我们强烈建议你也在报告中引用这些软件包。</p>
          <p>在本指南中，每当引入一个新软件包时，我们也会提供可用于引用它的参考文献信息。你也可以运行 <code>citation("package")</code> 来获取推荐的引用条目。谢谢！</p>
        </section>

        <div class="chapter-nav">
          <div class="empty"></div>
          <div class="next"><a href="https://doing-meta.guide/preface">前言</a></div>
        </div>

        <div class="series-return">
          <a href="/posts/2026/04/doing-meta-analysis-in-r-series/">Return to the series introduction</a>
        </div>
      </main>

      <aside class="sidebar sidebar-chapter">
        <nav id="toc" data-toggle="toc" aria-label="On this page">
          <h2>On this page</h2>
          <ul class="nav">
            <li><a class="nav-link active" href="#section-welcome">欢迎！</a></li>
            <li><a class="nav-link" href="#section-open-source-repository">开源仓库</a></li>
            <li><a class="nav-link" href="#section-how-to-use-the-guide">如何使用本指南</a></li>
            <li><a class="nav-link" href="#section-contributing">参与贡献</a></li>
            <li><a class="nav-link" href="#section-citing-this-guide">引用本指南</a></li>
            <li><a class="nav-link" href="#section-cite-the-packages">请同时引用相关软件包</a></li>
          </ul>
          <div class="book-extra"></div>
        </nav>
      </aside>
    </div>
  </div>

  <footer>
    <div class="footer-grid">
      <p>《<strong>Doing Meta-Analysis in R</strong>: A Hands-on Guide》由 Mathias Harrer、Pim Cuijpers、Toshi A. Furukawa 和 David D. Ebert 撰写。</p>
      <p>本书由 <a href="https://bookdown.org/">bookdown</a> R 软件包构建。</p>
    </div>
  </footer>
</div>
