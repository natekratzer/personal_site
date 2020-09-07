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
rmd_hash: 43ceaadc731af4b4

---

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Here we're assuming a simple design. </span>
<span class='c'># Survey requires the creation of a design object and then has functions that work with that object.</span>
<span class='c'># You can get more complicated, which is when the survey package would be most useful.</span>
<span class='k'>svy_df</span> <span class='o'>&lt;-</span> <span class='nf'>svydesign</span>(ids = <span class='o'>~</span> <span class='m'>1</span>, weights = <span class='o'>~</span><span class='k'>PERWT</span>, data = <span class='k'>df2018</span>)

<span class='c'># Taking the mean and standard error from our design object</span>
<span class='k'>hint_tbl</span> <span class='o'>&lt;-</span> <span class='nf'>svymean</span>(<span class='o'>~</span><span class='k'>hspd_num</span>, design = <span class='k'>svy_df</span>)

<span class='k'>hint_tbl</span> <span class='o'>&lt;-</span> <span class='nf'>as_tibble</span>(<span class='k'>hint_tbl</span>)
<span class='nf'><a href='https://rdrr.io/r/base/names.html'>names</a></span>(<span class='k'>hint_tbl</span>) <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='s'>"mean"</span>, <span class='s'>"sd"</span>) <span class='c'>#The names weren't coerced correctly when transforming into a tibble. </span>
</code></pre>

</div>

