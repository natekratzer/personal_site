---
output: hugodown::md_document
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "How to Use Census Microdata to Analyze High Speed Internet in Kentucky"
subtitle: ""
summary: ""
authors: [admin]
tags: []
categories: []
date: 2020-08-29
lastmod: 2020-08-29
featured: false
draft: true

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
rmd_hash: be99b62e50668fda

---

Getting the Data
================

The easiest way to get census microdata is through the Integrated Public Use Microdata Series (IPUMS) hosted by the University of Minnesota. While you can get the data directly from the Census Bureau, IPUMS has made it much easier to compare across multiple years and to select the variables you want. IPUMS also provides a codebook that is easy to refer to and notes any important changes from year to year.

I've put the data for just Kentucky up on GitHub, so I'll read it in from there.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='http://tidyverse.tidyverse.org'>tidyverse</a></span>) 
<span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='https://github.com/rstudio/gt'>gt</a></span>)
<span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='http://r-survey.r-forge.r-project.org/survey/'>survey</a></span>)

<span class='k'>df</span> <span class='o'>&lt;-</span> <span class='nf'>read_csv</span>(<span class='s'>"https://raw.github.com/natekratzer/raw_data/master/ky_high_speed_internet.csv"</span>)
</code></pre>

</div>

Cleaning the Data
=================

When downloading the data it's all numeric, even for variables that are categorial - they've been coded and our first step in the analysis will be using the code book to translate them. I won't show all the codebooks, but for this first variable let's take a look at what IPUMS has to say. For all years NA is coded as 00 and No high speed internet is coded as 20. Prior to 2016 there are detailed codes for the type of internet access, while for 2016 and after the code is collapsed.

![](high_speed_code.png)

I'll use a `case_when()` statement to recode high speed interent access into a categorical variable.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># High Speed Internet</span>
<span class='k'>df</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(
    hspd_int = <span class='nf'>case_when</span>(
      <span class='k'>CIHISPEED</span> <span class='o'>==</span> <span class='m'>00</span> <span class='o'>~</span> <span class='m'>NA_character_</span>,
      <span class='k'>CIHISPEED</span> <span class='o'>==</span> <span class='m'>20</span> <span class='o'>~</span> <span class='s'>"No"</span>,
      <span class='k'>CIHISPEED</span> <span class='o'>&gt;=</span> <span class='m'>10</span> <span class='o'>&amp;</span> <span class='k'>CIHISPEED</span> <span class='o'>&lt;</span> <span class='m'>20</span> <span class='o'>~</span> <span class='s'>"Yes"</span>,
      <span class='kc'>TRUE</span> <span class='o'>~</span> <span class='m'>NA_character_</span>
    )
  )
</code></pre>

</div>

Getting wrong answers by not knowing the data
---------------------------------------------

Now that we have a high speed internet category we can group the data and count up how many responses are in each group. I'll also pivot the dataframe to make it easy to calculate percent with high speed internet.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Count numbers with and without high speed internet</span>
<span class='k'>df_group</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>hspd_int</span>, <span class='k'>YEAR</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>summarize</span>(count = <span class='nf'>n</span>(), .groups = <span class='s'>"drop"</span>)

<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='k'>YEAR</span>, names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_NA = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>))) 
</code></pre>

</div>

While it looks like we have our answers there are two things that are wrong. First, the Census data is weighted. Instead of a count of responses we want to weight them using the person weights the census provides. We can fix that with a pretty simple change - use [`sum(PERWT)`](https://rdrr.io/r/base/sum.html) instead of `n()` in getting the count of people with and without high speed internet.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Count numbers with and without high speed internet</span>
<span class='k'>df_group</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>hspd_int</span>, <span class='k'>YEAR</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>))

<span class='c'>#&gt; `summarise()` regrouping output by 'hspd_int' (override with `.groups` argument)</span>


<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='k'>YEAR</span>, names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_NA = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>)))
</code></pre>

</div>

This is better. The second problem is harder to spot. There are 3 hints in the data:

1.  There is a very high percentage of NA responses. There are more NA answers than there are people who say they don't have high speed access.
2.  Percent of of people with high speed access is going down over time, while the number of NA answers is going up.
3.  These numbers look very high for Kentucky.

A sensible guess is that people who say they don't have internet access at all aren't then asked about high speed internet and show up as an NA value when we want to code them as not having high speed interent.

So let's get to know the data a bit better by adding in internet access. We'll do the same analysis, but I'll add internet as another id variable just like year. We can see right away that the answers we have above are only including cases where individuals have internet.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>df</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(
    int = <span class='nf'>case_when</span>(
      <span class='k'>CINETHH</span> <span class='o'>==</span> <span class='m'>0</span> <span class='o'>~</span> <span class='m'>NA_character_</span>,
      <span class='k'>CINETHH</span> <span class='o'>==</span> <span class='m'>1</span> <span class='o'>|</span> <span class='k'>CINETHH</span> <span class='o'>==</span> <span class='m'>2</span> <span class='o'>~</span> <span class='s'>"Yes"</span>,
      <span class='k'>CINETHH</span> <span class='o'>==</span> <span class='m'>3</span> <span class='o'>~</span> <span class='s'>"No"</span>,
      <span class='kc'>TRUE</span> <span class='o'>~</span> <span class='m'>NA_character_</span>
    )
  )

<span class='k'>df_group</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>hspd_int</span>, <span class='k'>int</span>, <span class='k'>YEAR</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>), .groups = <span class='s'>"drop"</span>)

<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>YEAR</span>, <span class='k'>int</span>), names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_na = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>)))

</code></pre>

</div>

So what we were looking at was the percentage of people with internet who have high speed internet. What we want is the percentage of all people who have high speed internet. We can fix the way we create our categories by saying that anyone who has no internet also has no high speed internet.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>df</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(
    int = <span class='nf'>case_when</span>(
      <span class='k'>CINETHH</span> <span class='o'>==</span> <span class='m'>0</span> <span class='o'>~</span> <span class='m'>NA_character_</span>,
      <span class='k'>CINETHH</span> <span class='o'>==</span> <span class='m'>1</span> <span class='o'>|</span> <span class='k'>CINETHH</span> <span class='o'>==</span> <span class='m'>2</span> <span class='o'>~</span> <span class='s'>"Yes"</span>,
      <span class='k'>CINETHH</span> <span class='o'>==</span> <span class='m'>3</span> <span class='o'>~</span> <span class='s'>"No"</span>,
      <span class='kc'>TRUE</span> <span class='o'>~</span> <span class='m'>NA_character_</span>
    ),
    hspd_int = <span class='nf'>case_when</span>(
      <span class='k'>CIHISPEED</span> <span class='o'>==</span> <span class='m'>00</span> <span class='o'>&amp;</span> <span class='k'>int</span> != <span class='s'>"No"</span> <span class='o'>~</span> <span class='m'>NA_character_</span>,
      <span class='k'>CIHISPEED</span> <span class='o'>==</span> <span class='m'>20</span> <span class='o'>|</span> <span class='k'>int</span> <span class='o'>==</span> <span class='s'>"No"</span> <span class='o'>~</span> <span class='s'>"No"</span>,
      <span class='k'>CIHISPEED</span> <span class='o'>&gt;=</span> <span class='m'>10</span> <span class='o'>&amp;</span> <span class='k'>CIHISPEED</span> <span class='o'>&lt;</span> <span class='m'>20</span> <span class='o'>~</span> <span class='s'>"Yes"</span>,
      <span class='kc'>TRUE</span> <span class='o'>~</span> <span class='m'>NA_character_</span>
    )
  )

<span class='c'># Count numbers with and without high speed internet</span>
<span class='k'>df_group</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>hspd_int</span>, <span class='k'>YEAR</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>), .groups = <span class='s'>"drop"</span>)

<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>YEAR</span>), names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_na = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>)))
</code></pre>

</div>

These results look much better, although still quite a few NA results.

Group Quarters in the Census
----------------------------

The census data includes individuals living in group quarters (mostly prisons, senior living centers, and dorms, but includes any sort of communal living arrangement). However, all census questions about appliances and utilities (the category that internet access falls under) are NA for group quarters. So we'll add one more line to filter out individuals living in group quarters (a common practice when working with microdata). The code below adds a filter for Group Quarters. Since this table is showing correct results I'll also add a little additional formatting to make it stand out from the others.

I'll also note that the way the Census Bureau constructs weights is very convenient for getting totals. While I'm focusing on the percent of people who have internet access, the Yes and No columns are accurate estimates of the population with and without access.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Count numbers with and without high speed internet</span>
<span class='k'>df_group</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span>(<span class='k'>GQ</span> <span class='o'>==</span> <span class='m'>1</span> <span class='o'>|</span> <span class='k'>GQ</span> <span class='o'>==</span><span class='m'>2</span> <span class='o'>|</span> <span class='k'>GQ</span> <span class='o'>==</span> <span class='m'>5</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>hspd_int</span>, <span class='k'>YEAR</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>), .groups = <span class='s'>"drop"</span>)

<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>YEAR</span>), names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_na = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>)))
</code></pre>

</div>

That removed about half of our NA values. It might be nice to know a bit more about the missing data, but at around 3 percent of observations it's unlikely to change our substantive conclusions. I suspect these are cases where there wasn't an answer for that question. We'll keep an eye on NA values as we do the analysis, because as we get into questions like how internet access varies by race, income, age, and education we'll want to know if NA answers are more or less likely in any of those categories.

Checking against data.census.gov
--------------------------------

To do a quick check against the way the census bureau itself analyzes the data I looked at data.census.gov for 2018 in Kentucky. An important note is that their data is for households, and so their numeric counts look quite different because I'm counting number of people. They also have a breakdown where cellular is included in broadband, which I do not want, as a cell phone is not really an adequate work or study device. So to get to what I have we need to add "Broadband such as cable, fiber optic or DSL" and "Satellite Internet service", which gets us to 70.8% compared to the 70.5% in this analysis. The difference is small and most likely the result of their analysis being weighted to the household level rather than the person level. (Internet is measured at the household level and the same for every person in the household, but by choosing to weight it at the person level I am a) letting us talk in terms of people, b) giving more weight to larger households, c) making it possible to breakdown internet access by categories that do vary by household, like age).

![](data_census_gov.png)

Analysis
========

Going forward we're going to want to filter by group quarters, so let's apply that filter to our main dataframe.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>df</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
    <span class='nf'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span>(<span class='k'>GQ</span> <span class='o'>==</span> <span class='m'>1</span> <span class='o'>|</span> <span class='k'>GQ</span> <span class='o'>==</span><span class='m'>2</span> <span class='o'>|</span> <span class='k'>GQ</span> <span class='o'>==</span> <span class='m'>5</span>) 
</code></pre>

</div>

Standard Errors
---------------

Know that we know the data we'd also like to know how uncertain our sample is so that we know if movements over time are real or just a result of noisy data. There are a few ways to do this. The `survey` package does an excellent job with complex survey designs, but does require learning a new syntax to use. The alternative I'll use here is a method known as bootstrap. IPUMS suggests using bootstrap might be the best way to get standard errors on census microdata. The basic idea of the bootstrap is to resample the existing data and use the sampling error from that as an estimate for sampling error in the overall population. Let's do an example with high speed internet in 2018 to see how it works. The output here will be the mean and standard deviation for Kentucky. (We'll use the standard error to calculate confidence intervals once we start displaying actual results.)

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'>#set seed</span>
<span class='nf'><a href='https://rdrr.io/r/base/Random.html'>set.seed</a></span>(<span class='m'>42</span>)

<span class='c'># Filter to just 2018</span>
<span class='c'># Exclude NA values</span>
<span class='c'># Recode as numeric vector of 1 and 0</span>
<span class='c'># The numeric 1 and 0 form will make it much easier to get means without pivoting, which matters a lot when doing this 1000 times</span>
<span class='k'>df2018</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span>(<span class='k'>YEAR</span> <span class='o'>==</span> <span class='m'>2018</span> <span class='o'>&amp;</span> <span class='o'>!</span><span class='nf'><a href='https://rdrr.io/r/base/NA.html'>is.na</a></span>(<span class='k'>hspd_int</span>)) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(hspd_num = <span class='nf'>if_else</span>(<span class='k'>hspd_int</span> <span class='o'>==</span> <span class='s'>"Yes"</span>, <span class='m'>1</span>, <span class='m'>0</span>)) <span class='o'>%&gt;%</span>
  <span class='nf'>select</span>(<span class='k'>hspd_num</span>, <span class='k'>PERWT</span>)

<span class='c'># Write a function so I can map over it.</span>
<span class='c'># In this case, we need the function to do the same thing X number of times and assign an ID that we can use as a grouping variable</span>
<span class='k'>create_samples</span> <span class='o'>&lt;-</span> <span class='nf'>function</span>(<span class='k'>sample_id</span>){
  <span class='k'>df_out</span> <span class='o'>&lt;-</span> <span class='k'>df2018</span>[<span class='nf'><a href='https://rdrr.io/r/base/sample.html'>sample</a></span>(<span class='nf'><a href='https://rdrr.io/r/base/nrow.html'>nrow</a></span>(<span class='k'>df2018</span>), <span class='nf'><a href='https://rdrr.io/r/base/nrow.html'>nrow</a></span>(<span class='k'>df2018</span>), replace = <span class='kc'>TRUE</span>) , ] <span class='o'>%&gt;%</span>
    <span class='nf'>as_tibble</span>()
  <span class='k'>df_out</span><span class='o'>$</span><span class='k'>sample_id</span> <span class='o'>&lt;-</span> <span class='k'>sample_id</span>
  <span class='nf'><a href='https://rdrr.io/r/base/function.html'>return</a></span>(<span class='k'>df_out</span>)
}

<span class='k'>nlist</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/list.html'>as.list</a></span>(<span class='nf'><a href='https://rdrr.io/r/base/seq.html'>seq</a></span>(<span class='m'>1</span>, <span class='m'>5000</span>, by = <span class='m'>1</span>))
<span class='k'>samples</span> <span class='o'>&lt;-</span> <span class='k'>purrr</span>::<span class='nf'><a href='https://purrr.tidyverse.org/reference/map.html'>map_df</a></span>(<span class='k'>nlist</span>, <span class='k'>create_samples</span>)

<span class='k'>sample_summary</span> <span class='o'>&lt;-</span> <span class='k'>samples</span> <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>sample_id</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(ind_weight = <span class='k'>PERWT</span> <span class='o'>/</span> <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>),
         hspd_weight = <span class='k'>hspd_num</span> <span class='o'>*</span> <span class='k'>ind_weight</span>) <span class='o'>%&gt;%</span> <span class='c'># PERWT is population and doesn't sum to 1. Rescale it to sum to one</span>
  <span class='nf'>summarize</span>(group_mean = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>hspd_weight</span>),
            weight_check = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>ind_weight</span>), .groups = <span class='s'>"drop"</span>) <span class='c'># Check that my weights add up to one</span>

<span class='k'>display_tbl</span> <span class='o'>&lt;-</span> <span class='nf'>tibble</span>(
  mean = <span class='nf'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span>(<span class='k'>sample_summary</span><span class='o'>$</span><span class='k'>group_mean</span>),
  sd = <span class='nf'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span>(<span class='k'>sample_summary</span><span class='o'>$</span><span class='k'>group_mean</span>)
) 
</code></pre>

</div>

We can also take a look at our bootstrap graphically. We want to check that the distribution of the sample is roughly normal. If it's not, that means we didn't do enough bootstrap samples for the Central Limit Theorem to kick in.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'>#Check that the distribution is normal and than the middle of the distribution is close to the 70.5% we estimated had internet access above</span>
<span class='k'>plt</span> <span class='o'>&lt;-</span> <span class='nf'>ggplot</span>(<span class='k'>sample_summary</span>, <span class='nf'>aes</span>(<span class='k'>group_mean</span>)) <span class='o'>+</span>
  <span class='nf'>geom_density</span>() <span class='o'>+</span> <span class='nf'>theme_bw</span>() <span class='o'>+</span>
  <span class='nf'>labs</span>(title = <span class='s'>"Bootstrapped means of High Speed Internet Access"</span>,
       x = <span class='s'>"Mean"</span>, 
       y = <span class='s'>"Kernel Density"</span>)

<span class='k'>plt</span>

</code></pre>
<img src="figs/unnamed-chunk-10-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Checking our results against the survey package
-----------------------------------------------

Above we found a mean of 0.705 for 2018 and and standard error of 0.0029 based on our bootstrap analysis. It's worth checking that this is the same result we'd get using an analytic approach (instead of bootstrap). So here's the code to take our same `df2018` dataframe and use the survey package.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='http://r-survey.r-forge.r-project.org/survey/'>survey</a></span>)

<span class='c'># Here we're assuming a simple design. </span>
<span class='c'># Survey requires the creation of a design object and then has functions that work with that object.</span>
<span class='c'># You can get more complicated, which is when the survey package would be most useful.</span>
<span class='k'>svy_df</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/pkg/survey/man/svydesign.html'>svydesign</a></span>(ids = <span class='o'>~</span> <span class='m'>1</span>, weights = <span class='o'>~</span><span class='k'>PERWT</span>, data = <span class='k'>df2018</span>)

<span class='c'># Taking the mean and standard error from our design object</span>
<span class='k'>hint_tbl</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/pkg/survey/man/surveysummary.html'>svymean</a></span>(<span class='o'>~</span><span class='k'>hspd_num</span>, design = <span class='k'>svy_df</span>)

<span class='k'>hint_tbl</span> <span class='o'>&lt;-</span> <span class='nf'>as_tibble</span>(<span class='k'>hint_tbl</span>)
<span class='nf'><a href='https://rdrr.io/r/base/names.html'>names</a></span>(<span class='k'>hint_tbl</span>) <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='s'>"mean"</span>, <span class='s'>"sd"</span>) <span class='c'>#The names weren't coerced correctly when transforming into a tibble. </span>
</code></pre>

</div>

These results are very similar. Following the IPUMS recommendation we'll continue on with the bootstrap, but it's good to know the results are the same for practical purposes. So now instead of just doing 2018, we'll need to do every year. We've already one the mean values for every year, and they're still saved in the `df_wide` variable right now. So let's write a function for bootstrap that will let us find standard errors for every year or for any other grouping we choose.

Writing a bootstrap function
----------------------------

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Create a helper function</span>
<span class='c'># It needs to have a way to recieve the dataframe from the function that calls it, so we've added a second argument</span>
<span class='k'>create_samples</span> <span class='o'>&lt;-</span> <span class='nf'>function</span>(<span class='k'>sample_id</span>, <span class='k'>df</span>){
  
  <span class='k'>df_out</span> <span class='o'>&lt;-</span> <span class='k'>df</span>[<span class='nf'><a href='https://rdrr.io/r/base/sample.html'>sample</a></span>(<span class='nf'><a href='https://rdrr.io/r/base/nrow.html'>nrow</a></span>(<span class='k'>df</span>), <span class='nf'><a href='https://rdrr.io/r/base/nrow.html'>nrow</a></span>(<span class='k'>df</span>), replace = <span class='kc'>TRUE</span>) , ] <span class='o'>%&gt;%</span>
    <span class='nf'>as_tibble</span>()
  
  <span class='k'>df_out</span><span class='o'>$</span><span class='k'>sample_id</span> <span class='o'>&lt;-</span> <span class='k'>sample_id</span>
  
  <span class='nf'><a href='https://rdrr.io/r/base/function.html'>return</a></span>(<span class='k'>df_out</span>)
}

<span class='c'>#Need to be able to take in grouping variables so that the summaries can be specific to the groups</span>
<span class='k'>bootstrap_pums</span> <span class='o'>&lt;-</span> <span class='nf'>function</span>(<span class='k'>df</span>, <span class='k'>num_samples</span>, <span class='k'>group_vars</span>) {
  
  <span class='k'>nlist</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/list.html'>as.list</a></span>(<span class='nf'><a href='https://rdrr.io/r/base/seq.html'>seq</a></span>(<span class='m'>1</span>, <span class='k'>num_samples</span>, by = <span class='m'>1</span>))
  <span class='k'>samples</span> <span class='o'>&lt;-</span> <span class='k'>purrr</span>::<span class='nf'><a href='https://purrr.tidyverse.org/reference/map.html'>map_df</a></span>(<span class='k'>nlist</span>, <span class='k'>create_samples</span>, <span class='k'>df</span>)
  
  <span class='k'>sample_summary</span> <span class='o'>&lt;-</span> <span class='k'>samples</span> <span class='o'>%&gt;%</span>
    <span class='nf'>group_by</span>( <span class='k'>sample_id</span>, <span class='nf'>across</span>( {{<span class='k'>group_vars</span>}} )) <span class='o'>%&gt;%</span>
    <span class='nf'>mutate</span>(ind_weight = <span class='k'>PERWT</span> <span class='o'>/</span> <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>),
           hspd_weight = <span class='k'>hspd_n</span> <span class='o'>*</span> <span class='k'>ind_weight</span>) <span class='o'>%&gt;%</span> <span class='c'># PERWT sums to population instead of to 1. Rescale it to sum to 1.</span>
    <span class='nf'>summarize</span>(group_mean = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>hspd_weight</span>), .groups = <span class='s'>"drop"</span>) <span class='c'># Not dropping .groups here results in problems in the next group_by call.</span>
  
  <span class='k'>sample_sd</span> <span class='o'>&lt;-</span> <span class='k'>sample_summary</span> <span class='o'>%&gt;%</span>
    <span class='nf'>group_by</span>( <span class='nf'>across</span>( {{ <span class='k'>group_vars</span> }} )) <span class='o'>%&gt;%</span>
    <span class='nf'>summarize</span>(sd = <span class='nf'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span>(<span class='k'>group_mean</span>), .groups = <span class='s'>"drop"</span>)
}

<span class='c'># We do need to prep the data a little so that we're not carrying through the whole dataframe.</span>
<span class='k'>df_in</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
   <span class='nf'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span>(<span class='o'>!</span><span class='nf'><a href='https://rdrr.io/r/base/NA.html'>is.na</a></span>(<span class='k'>hspd_int</span>)) <span class='o'>%&gt;%</span>
   <span class='nf'>mutate</span>(hspd_n = <span class='nf'>if_else</span>(<span class='k'>hspd_int</span> <span class='o'>==</span> <span class='s'>"Yes"</span>, <span class='m'>1</span>, <span class='m'>0</span>)) <span class='o'>%&gt;%</span>
   <span class='nf'>select</span>(<span class='k'>hspd_n</span>, <span class='k'>PERWT</span>, <span class='k'>YEAR</span>)

<span class='c'># And finally we can call the function</span>
<span class='k'>boot_results</span> <span class='o'>&lt;-</span> <span class='nf'>bootstrap_pums</span>(df = <span class='k'>df_in</span>, num_samples = <span class='m'>100</span>, group_vars = <span class='k'>YEAR</span>)
</code></pre>

</div>

Now that we have our bootstrap standard errors we can combine them with the data and plot them. We'll use 95% confidence intervals, which we get by multiplying the

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>df_plt</span> <span class='o'>&lt;-</span> <span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>full_join</span>(<span class='k'>boot_results</span>, by = <span class='s'>"YEAR"</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(Year = <span class='k'>YEAR</span>,
            Percent = <span class='m'>100</span> <span class='o'>*</span> <span class='k'>percent_hspd</span>,
            me = <span class='m'>100</span> <span class='o'>*</span> <span class='m'>1.96</span> <span class='o'>*</span> <span class='k'>sd</span>)
  
<span class='k'>plt_int</span> <span class='o'>&lt;-</span> <span class='nf'>ggplot</span>(<span class='k'>df_plt</span>, <span class='nf'>aes</span>(x = <span class='k'>Year</span>, y = <span class='k'>Percent</span>)) <span class='o'>+</span>
  <span class='nf'>geom_errorbar</span>(<span class='nf'>aes</span>(ymin = <span class='k'>Percent</span> <span class='o'>-</span> <span class='k'>me</span>, ymax = <span class='k'>Percent</span> <span class='o'>+</span> <span class='k'>me</span>), width = <span class='m'>.1</span>) <span class='o'>+</span>
  <span class='nf'>geom_line</span>() <span class='o'>+</span>
  <span class='nf'>geom_point</span>() <span class='o'>+</span>
  <span class='nf'>theme_bw</span>() <span class='o'>+</span>
  <span class='nf'>labs</span>(title = <span class='s'>"High Speed Internet Access"</span>) <span class='o'>+</span>
  <span class='nf'>theme</span>(legend.position = <span class='s'>"bottom"</span>)

<span class='k'>plt_int</span>

</code></pre>
<img src="figs/unnamed-chunk-13-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Race, Poverty, Age
------------------

### Race

We'll build a table by race and year. It's a long table, so I've added some color to both the `Percent Yes` column and the `Percent NA` column. For the NA column I'm using red to pick out cases where the NA values were particularly high, because we want to see if there's a pattern there. For the Percent Yes column I'm checking to see where the values are particularly low.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Let's build a table first and then we'll do the standard errors</span>

<span class='c'># Coding a race variable using case_when</span>
<span class='k'>df</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(race = <span class='nf'>case_when</span>(
            <span class='k'>RACE</span> <span class='o'>==</span> <span class='m'>1</span> <span class='o'>~</span> <span class='s'>"White"</span>,
            <span class='k'>RACE</span> <span class='o'>==</span> <span class='m'>2</span> <span class='o'>~</span> <span class='s'>"Black"</span>,
            <span class='k'>RACE</span> <span class='o'>&gt;</span> <span class='m'>3</span> <span class='o'>&amp;</span> <span class='k'>RACE</span> <span class='o'>&lt;</span> <span class='m'>7</span> <span class='o'>~</span> <span class='s'>"Asian"</span>,
            <span class='k'>HISPAN</span> <span class='o'>&gt;</span> <span class='m'>0</span> <span class='o'>&amp;</span> <span class='k'>HISPAN</span> <span class='o'>&lt;</span> <span class='m'>5</span> <span class='o'>~</span> <span class='s'>"Hispanic"</span>,
            <span class='kc'>TRUE</span> <span class='o'>~</span> <span class='s'>"All Others"</span>
          ))

<span class='k'>df_group</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>hspd_int</span>, <span class='k'>race</span>, <span class='k'>YEAR</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>), .groups = <span class='s'>"drop"</span>)

<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>race</span>, <span class='k'>YEAR</span>), names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_na = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>)))

</code></pre>

</div>

While we do see high NA values in some years

Now let's add standard errors and graph the data.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># We do need to prep the data a little so that we're not carrying through the whole dataframe.</span>
<span class='k'>df_in</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span>(<span class='o'>!</span><span class='nf'><a href='https://rdrr.io/r/base/NA.html'>is.na</a></span>(<span class='k'>hspd_int</span>)) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(hspd_n = <span class='nf'>if_else</span>(<span class='k'>hspd_int</span> <span class='o'>==</span> <span class='s'>"Yes"</span>, <span class='m'>1</span>, <span class='m'>0</span>)) <span class='o'>%&gt;%</span>
  <span class='nf'>select</span>(<span class='k'>hspd_n</span>, <span class='k'>PERWT</span>, <span class='k'>YEAR</span>, <span class='k'>race</span>)

<span class='c'># And we can call the bootstrap function</span>
<span class='k'>boot_results</span> <span class='o'>&lt;-</span> <span class='nf'>bootstrap_pums</span>(df = <span class='k'>df_in</span>, num_samples = <span class='m'>100</span>, group_vars = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>YEAR</span>, <span class='k'>race</span>))

<span class='k'>df_plt</span> <span class='o'>&lt;-</span> <span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>full_join</span>(<span class='k'>boot_results</span>, by = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='s'>"race"</span>, <span class='s'>"YEAR"</span>)) <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(Year = <span class='k'>YEAR</span>,
            Race = <span class='k'>race</span>,
            Percent = <span class='m'>100</span> <span class='o'>*</span> <span class='k'>percent_hspd</span>,
            me = <span class='m'>100</span> <span class='o'>*</span> <span class='m'>1.96</span> <span class='o'>*</span> <span class='k'>sd</span>) <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span>(<span class='k'>Race</span> != <span class='s'>"All Others"</span>) <span class='c'># When plotting All Others overlaps White and having five lines makes it quite hard to read. </span>

<span class='c'># At this point I'll introduce a function to plot multiple groups over time, since we'll use this again </span>

<span class='k'>plt_by</span> <span class='o'>&lt;-</span> <span class='nf'>function</span>(<span class='k'>df</span>, <span class='k'>group_var</span>, <span class='k'>title_text</span> = <span class='s'>"High Speed Internet Access by Race and Ethnicity"</span>) {
  
  <span class='k'>plt</span> <span class='o'>&lt;-</span> <span class='nf'>ggplot</span>(data = <span class='k'>df</span>, <span class='nf'>aes</span>(x = <span class='k'>Year</span>, y = <span class='k'>Percent</span>, group = {{<span class='k'>group_var</span>}}, colour = {{<span class='k'>group_var</span>}})) <span class='o'>+</span>
    <span class='nf'>geom_errorbar</span>(<span class='nf'>aes</span>(ymin = <span class='k'>Percent</span> <span class='o'>-</span> <span class='k'>me</span>, ymax = <span class='k'>Percent</span> <span class='o'>+</span> <span class='k'>me</span>), width = <span class='m'>.1</span>) <span class='o'>+</span>
    <span class='nf'>geom_point</span>() <span class='o'>+</span>
    <span class='nf'>geom_line</span>() <span class='o'>+</span>
    <span class='nf'>theme_bw</span>() <span class='o'>+</span>
    <span class='nf'>labs</span>(title = <span class='k'>title_text</span>, x = <span class='s'>"Year"</span>, y = <span class='s'>"Percent"</span>) <span class='o'>+</span>
    <span class='nf'>theme</span>(legend.position = <span class='s'>"bottom"</span>)

  <span class='k'>plt</span>
}

<span class='k'>plt_race</span> <span class='o'>&lt;-</span> <span class='nf'>plt_by</span>(<span class='k'>df_plt</span>, <span class='k'>Race</span>)

<span class='k'>plt_race</span>

</code></pre>
<img src="figs/unnamed-chunk-15-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Poverty Status
--------------

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Coding a race variable using case_when</span>
<span class='k'>df</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(poverty = <span class='nf'>case_when</span>(
            <span class='k'>POVERTY</span> <span class='o'>&lt;=</span> <span class='m'>100</span> <span class='o'>~</span> <span class='s'>"In Poverty"</span>,
            <span class='k'>POVERTY</span> <span class='o'>&gt;</span> <span class='m'>100</span> <span class='o'>&amp;</span> <span class='k'>POVERTY</span> <span class='o'>&lt;=</span> <span class='m'>200</span> <span class='o'>~</span> <span class='s'>"Near Poverty"</span>,
            <span class='kc'>TRUE</span> <span class='o'>~</span> <span class='s'>"Not in Poverty"</span>
          ))

<span class='k'>df_group</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>hspd_int</span>, <span class='k'>poverty</span>, <span class='k'>YEAR</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>), .groups = <span class='s'>"drop"</span>)

<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>poverty</span>, <span class='k'>YEAR</span>), names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_na = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>)))
</code></pre>

</div>

Age
---

Geography
---------

Urban v. Suburban v. Rural

Mapping the Data
================

All of Kentucky
---------------

Children 5-18
-------------

We'll also take a count of children 5-18 in Kentucky

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Internet Overall</span>

</code></pre>

</div>

