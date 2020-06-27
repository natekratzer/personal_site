---
output: hugodown::md_document
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Using Hugodown Academic"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-06-27
lastmod: 2020-06-27
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
rmd_hash: 0eb93df15ed84aaa

---

I'm getting attempting to get a website/blog up using hugodown and the academic theme. As a first post, I'm writing about the process of setting up the site.

I started at the [hugodown site](https://hugodown.r-lib.org/index.html) and installed hugodown

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>devtools</span>::<span class='nf'><a href='https://rdrr.io/pkg/devtools/man/remote-reexports.html'>install_github</a></span>(<span class='s'>"r-lib/hugodown"</span>)</code></pre>

</div>

Then when I tried to create an academic site I got a helpful error message telling me how to install hugo first, so the next two steps are to first install hugo and then create the academic site. The `create_site_academic()` function errored out informing me that `Error: 'ui_silence' is not an exported object from 'namespace:usethis'` which was resolved by installing the latest version of `usethis` from CRAN.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>hugodown</span>::<span class='nf'><a href='https://rdrr.io/pkg/hugodown/man/hugo_install.html'>hugo_install</a></span>(<span class='s'>'0.66.0'</span>)
<span class='k'>hugodown</span>::<span class='nf'><a href='https://rdrr.io/pkg/hugodown/man/create_site_academic.html'>create_site_academic</a></span>()</code></pre>

</div>

At that point I was able to preview the default site

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>hugodown</span>::<span class='nf'><a href='https://rdrr.io/pkg/hugodown/man/hugo_start.html'>hugo_start</a></span>()</code></pre>

</div>

When using the `use_post` function to create a this post I discovered it seems to set the working directory into the content folder already

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'>#This errors with the directory not found</span>
<span class='k'>hugodown</span>::<span class='nf'><a href='https://rdrr.io/pkg/hugodown/man/use_post.html'>use_post</a></span>(<span class='s'>"content/post/using_hugodown"</span>)

<span class='c'>#This works</span>
<span class='k'>hugodown</span>::<span class='nf'><a href='https://rdrr.io/pkg/hugodown/man/use_post.html'>use_post</a></span>(<span class='s'>"post/using_hugodown"</span>)</code></pre>

</div>

At this point I tried knitting this post to see if the site would update and had an error of `Unknown extension: task_lists`. I tried updating `knitr` and had the same error. Next I tried installing the latest version of [pandoc](https://pandoc.org/installing.html)

