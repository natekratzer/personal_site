---
output: hugodown::md_document
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Using Hugodown Academic"
subtitle: "A personal website using: R, hugodown, Academic Theme, GitHub, and Netlify"
summary: "I'm attempting to get a website/blog up using hugodown and the academic theme. As a first post, I'm writing about the process of setting up the site."
authors: [admin]
tags: [hugodown, internet]
categories: [R]
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
rmd_hash: c87a4767ba1e57d7

---

I'm attempting to get a website/blog up using hugodown and the academic theme. As a first post, I'm writing about the process of setting up the site. This post was written in pieces as I worked on my website, so until I go back and make some edits it's more of a first-hand account than a tutorial. Hopefully it's still useful.

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

At this point I tried knitting this post to see if the site would update and had an error of `Unknown extension: task_lists`. I tried updating `knitr` and had the same error. Next I tried installing the latest version of [pandoc](https://pandoc.org/installing.html), which worked and I was able to knit this post. Rerunning `hugo_start` showed me that the post had been added.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>hugodown</span>::<span class='nf'><a href='https://rdrr.io/pkg/hugodown/man/hugo_start.html'>hugo_start</a></span>()</code></pre>

</div>

At this point I decided to work on [deploying](https://hugodown.r-lib.org/articles/deploy.html) the site.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>hugodown</span>::<span class='nf'><a href='https://rdrr.io/pkg/hugodown/man/use_netlify_toml.html'>use_netlify_toml</a></span>()</code></pre>

</div>

I used netlify as recommended, bought my own domain name and then waited for the DNS to propogate. No issues with this part of the process.

The next step is customizing the Academic Theme. For the most part I did this by opening up files and reading the comments. There is also [documentation](https://sourcethemes.com/academic/docs/) online. The first thing I did was work with the `content/home` folder and turn off some of the extra features. For example, if you open content/home/hero you'll be see this markdown at the top. I set active to false to get rid of the banner.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Hero widget.</span>
<span class='k'>widget</span> <span class='o'>=</span> <span class='s'>"hero"</span>  <span class='c'># See https://sourcethemes.com/academic/docs/page-builder/</span>
<span class='k'>headless</span> <span class='o'>=</span> <span class='k'>true</span>  <span class='c'># This file represents a page section.</span>
<span class='k'>active</span> <span class='o'>=</span> <span class='k'>false</span>  <span class='c'># Activate this widget? true/false</span>
<span class='k'>weight</span> <span class='o'>=</span> <span class='m'>10</span>  <span class='c'># Order that this section will appear.</span></code></pre>

</div>

In order to update from being Nelson Bighetti you need to look in `content/authors/admin`. By default the admin folder is linked, although if you add additional authors there is an option in `content/home/about.md` to change which author is displayed. While the initial folder structure of the academic theme is pretty overwhelming I found that just approaching it a file at a time and reading the comments let me do most of what I wanted to do.

Adding google analytics was also fairly painless, I got the tracking number from the google analytics site and then put it in the config.toml file. The other code that Google talks about being required for this to work is already built into the academic theme, so you only have to add the tracking ID.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Enable analytics by entering your Google Analytics tracking ID</span>
<span class='k'>googleAnalytics</span> <span class='o'>=</span> <span class='s'>"&lt;trackingIDfromgoogle&gt;"</span></code></pre>

</div>

I also found that while turning off the extra pages by setting them to false was easy, those pages still showed up across the top of the site and in the dropdown menu. That menu is controlled by `config/_default/menus.toml`, and is to easy to change once you finally find it.

The two big lessons I learned from this process: - Update everything if you want hugodown to work on the first try - Expect to spend some time getting to know the academic theme. It's a complicated folder structure and frustrating when you're trying to do something specific, but it's also reasonably well documented and easy to change once you find out where the right file is in all the folders.

