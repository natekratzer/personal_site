---
output: hugodown::md_document
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "test_post"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-09-06
lastmod: 2020-09-06
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
rmd_hash: 4e23c6f1012be86b

---

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='http://tidyverse.tidyverse.org'>tidyverse</a></span>)

<span class='c'>#&gt; Warning: replacing previous import 'vctrs::data_frame' by 'tibble::data_frame' when loading 'dplyr'</span>

<span class='c'>#&gt; ── <span style='font-weight: bold;'>Attaching packages</span><span> ───────────────────────────────────────────────────────────────────────────────────────────────────────────────── tidyverse 1.3.0 ──</span></span>

<span class='c'>#&gt; <span style='color: #00BB00;'>✔</span><span> </span><span style='color: #0000BB;'>ggplot2</span><span> 3.3.0     </span><span style='color: #00BB00;'>✔</span><span> </span><span style='color: #0000BB;'>purrr  </span><span> 0.3.4</span></span>
<span class='c'>#&gt; <span style='color: #00BB00;'>✔</span><span> </span><span style='color: #0000BB;'>tibble </span><span> 3.0.3     </span><span style='color: #00BB00;'>✔</span><span> </span><span style='color: #0000BB;'>dplyr  </span><span> 1.0.0</span></span>
<span class='c'>#&gt; <span style='color: #00BB00;'>✔</span><span> </span><span style='color: #0000BB;'>tidyr  </span><span> 1.1.1     </span><span style='color: #00BB00;'>✔</span><span> </span><span style='color: #0000BB;'>stringr</span><span> 1.4.0</span></span>
<span class='c'>#&gt; <span style='color: #00BB00;'>✔</span><span> </span><span style='color: #0000BB;'>readr  </span><span> 1.3.1     </span><span style='color: #00BB00;'>✔</span><span> </span><span style='color: #0000BB;'>forcats</span><span> 0.5.0</span></span>

<span class='c'>#&gt; ── <span style='font-weight: bold;'>Conflicts</span><span> ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──</span></span>
<span class='c'>#&gt; <span style='color: #BB0000;'>✖</span><span> </span><span style='color: #0000BB;'>dplyr</span><span>::</span><span style='color: #00BB00;'>filter()</span><span> masks </span><span style='color: #0000BB;'>stats</span><span>::filter()</span></span>
<span class='c'>#&gt; <span style='color: #BB0000;'>✖</span><span> </span><span style='color: #0000BB;'>dplyr</span><span>::</span><span style='color: #00BB00;'>lag()</span><span>    masks </span><span style='color: #0000BB;'>stats</span><span>::lag()</span></span>


<span class='k'>df</span> <span class='o'>&lt;-</span> <span class='nf'>tibble</span>(
  x = <span class='m'>1</span><span class='o'>:</span><span class='m'>4</span>,
  y = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='s'>"a"</span>, <span class='s'>"a"</span>, <span class='s'>"b"</span>, <span class='s'>"b"</span>)

)

<span class='k'>make_groups</span> <span class='o'>&lt;-</span> <span class='nf'>function</span>(<span class='k'>df</span>, <span class='k'>group_var</span>){
  <span class='k'>df</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
    <span class='nf'>group_by</span>({{<span class='k'>group_var</span>}}) <span class='o'>%&gt;%</span>
    <span class='nf'>summarize</span>(mean = <span class='nf'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span>(<span class='k'>x</span>))
}

<span class='k'>df</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>make_groups</span>(<span class='k'>y</span>)

<span class='c'>#&gt; `summarise()` ungrouping output (override with `.groups` argument)</span>


<span class='nf'><a href='https://rdrr.io/r/base/print.html'>print</a></span>(<span class='k'>df</span>)

<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 2 x 2</span></span>
<span class='c'>#&gt;   y      mean</span>
<span class='c'>#&gt;   <span style='color: #555555;font-style: italic;'>&lt;chr&gt;</span><span> </span><span style='color: #555555;font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span><span> a       1.5</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>2</span><span> b       3.5</span></span>
</code></pre>

</div>

