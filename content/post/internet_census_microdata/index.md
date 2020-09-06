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
rmd_hash: 6df71d29ebb99e12

---

Getting the Data
================

The easiest way to get census microdata is through the Integrated Public Use Microdata Series (IPUMS) hosted by the University of Minnesota. While you can get the data directly from the Census Bureau, IPUMS has made it much easier to compare across multiple years and to select the variables you want. IPUMS also provides a codebook that is easy to refer to and notes any important changes from year to year.

I've put the data for just Kentucky up on GitHub, so I'll read it in from there.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='http://tidyverse.tidyverse.org'>tidyverse</a></span>) 
<span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='https://renkun.me/formattable'>formattable</a></span>)
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

<span class='c'># Getting the data display ready using transmute, which combines mutate and select</span>
<span class='c'># Formattable is a bit nicer than kable and has some options for nice tables that we'll look at later</span>
<span class='c'># I really like the gt package for tables, but right now it doesn't work with hugodown or blogdown. </span>
<span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(
    Year = <span class='k'>YEAR</span>,
    `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_hspd</span>, digits = <span class='m'>1</span>),
    `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_NA</span>, digits = <span class='m'>1</span>),
    Yes = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>Yes</span>, digits = <span class='m'>0</span>),
    No = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>No</span>, digits = <span class='m'>0</span>),
    `NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>`NA`</span>, digits = <span class='m'>0</span>)
  ) <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>()

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
Year
</th>
<th style="text-align:right;">
Percent Yes
</th>
<th style="text-align:right;">
Percent NA
</th>
<th style="text-align:right;">
Yes
</th>
<th style="text-align:right;">
No
</th>
<th style="text-align:right;">
NA
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
86.9%
</td>
<td style="text-align:right;">
27.5%
</td>
<td style="text-align:right;">
28,337
</td>
<td style="text-align:right;">
4,273
</td>
<td style="text-align:right;">
12,387
</td>
</tr>
<tr>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
85.7%
</td>
<td style="text-align:right;">
26.9%
</td>
<td style="text-align:right;">
28,080
</td>
<td style="text-align:right;">
4,699
</td>
<td style="text-align:right;">
12,089
</td>
</tr>
<tr>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
86.4%
</td>
<td style="text-align:right;">
25.7%
</td>
<td style="text-align:right;">
28,743
</td>
<td style="text-align:right;">
4,518
</td>
<td style="text-align:right;">
11,488
</td>
</tr>
<tr>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
81.8%
</td>
<td style="text-align:right;">
20.2%
</td>
<td style="text-align:right;">
29,233
</td>
<td style="text-align:right;">
6,491
</td>
<td style="text-align:right;">
9,015
</td>
</tr>
<tr>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
80.6%
</td>
<td style="text-align:right;">
19.4%
</td>
<td style="text-align:right;">
29,356
</td>
<td style="text-align:right;">
7,084
</td>
<td style="text-align:right;">
8,769
</td>
</tr>
<tr>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
80.5%
</td>
<td style="text-align:right;">
17.3%
</td>
<td style="text-align:right;">
30,264
</td>
<td style="text-align:right;">
7,347
</td>
<td style="text-align:right;">
7,864
</td>
</tr>
</tbody>
</table>

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

<span class='c'># Otput to formattable</span>
<span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(
    Year = <span class='k'>YEAR</span>,
    `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_hspd</span>, digits = <span class='m'>1</span>),
    `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_NA</span>, digits = <span class='m'>1</span>),
    Yes = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>Yes</span>, digits = <span class='m'>0</span>),
    No = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>No</span>, digits = <span class='m'>0</span>),
    `NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>`NA`</span>, digits = <span class='m'>0</span>)
  ) <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>()

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
Year
</th>
<th style="text-align:right;">
Percent Yes
</th>
<th style="text-align:right;">
Percent NA
</th>
<th style="text-align:right;">
Yes
</th>
<th style="text-align:right;">
No
</th>
<th style="text-align:right;">
NA
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
85.9%
</td>
<td style="text-align:right;">
26.8%
</td>
<td style="text-align:right;">
2,763,511
</td>
<td style="text-align:right;">
454,185
</td>
<td style="text-align:right;">
1,177,599
</td>
</tr>
<tr>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
84.3%
</td>
<td style="text-align:right;">
26.4%
</td>
<td style="text-align:right;">
2,739,012
</td>
<td style="text-align:right;">
509,102
</td>
<td style="text-align:right;">
1,165,343
</td>
</tr>
<tr>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
85.6%
</td>
<td style="text-align:right;">
24.8%
</td>
<td style="text-align:right;">
2,847,329
</td>
<td style="text-align:right;">
478,782
</td>
<td style="text-align:right;">
1,098,981
</td>
</tr>
<tr>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
80.2%
</td>
<td style="text-align:right;">
19.3%
</td>
<td style="text-align:right;">
2,873,926
</td>
<td style="text-align:right;">
707,754
</td>
<td style="text-align:right;">
855,294
</td>
</tr>
<tr>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
79.8%
</td>
<td style="text-align:right;">
18.3%
</td>
<td style="text-align:right;">
2,902,850
</td>
<td style="text-align:right;">
734,608
</td>
<td style="text-align:right;">
816,731
</td>
</tr>
<tr>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
79.0%
</td>
<td style="text-align:right;">
15.9%
</td>
<td style="text-align:right;">
2,969,937
</td>
<td style="text-align:right;">
789,528
</td>
<td style="text-align:right;">
708,937
</td>
</tr>
</tbody>
</table>

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

<span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(
    Internet = <span class='k'>int</span>,
    Year = <span class='k'>YEAR</span>,
    `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_hspd</span>, digits = <span class='m'>1</span>),
    `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_na</span>, digits = <span class='m'>1</span>),
    Yes = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>Yes</span>, digits = <span class='m'>0</span>),
    No = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>No</span>, digits = <span class='m'>0</span>),
    `NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>`NA`</span>, digits = <span class='m'>0</span>)
  ) <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>()

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
Internet
</th>
<th style="text-align:right;">
Year
</th>
<th style="text-align:right;">
Percent Yes
</th>
<th style="text-align:right;">
Percent NA
</th>
<th style="text-align:right;">
Yes
</th>
<th style="text-align:right;">
No
</th>
<th style="text-align:right;">
NA
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
Yes
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
85.9%
</td>
<td style="text-align:right;">
5.4%
</td>
<td style="text-align:right;">
2,763,511
</td>
<td style="text-align:right;">
454,185
</td>
<td style="text-align:right;">
185,297
</td>
</tr>
<tr>
<td style="text-align:right;">
Yes
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
84.3%
</td>
<td style="text-align:right;">
5.6%
</td>
<td style="text-align:right;">
2,739,012
</td>
<td style="text-align:right;">
509,102
</td>
<td style="text-align:right;">
193,284
</td>
</tr>
<tr>
<td style="text-align:right;">
Yes
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
85.6%
</td>
<td style="text-align:right;">
6.1%
</td>
<td style="text-align:right;">
2,847,329
</td>
<td style="text-align:right;">
478,782
</td>
<td style="text-align:right;">
216,933
</td>
</tr>
<tr>
<td style="text-align:right;">
Yes
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
80.2%
</td>
<td style="text-align:right;">
3.5%
</td>
<td style="text-align:right;">
2,873,926
</td>
<td style="text-align:right;">
707,754
</td>
<td style="text-align:right;">
128,501
</td>
</tr>
<tr>
<td style="text-align:right;">
Yes
</td>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
79.8%
</td>
<td style="text-align:right;">
3.3%
</td>
<td style="text-align:right;">
2,902,850
</td>
<td style="text-align:right;">
734,608
</td>
<td style="text-align:right;">
124,375
</td>
</tr>
<tr>
<td style="text-align:right;">
Yes
</td>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
79.0%
</td>
<td style="text-align:right;">
3.2%
</td>
<td style="text-align:right;">
2,969,937
</td>
<td style="text-align:right;">
789,528
</td>
<td style="text-align:right;">
125,088
</td>
</tr>
<tr>
<td style="text-align:right;">
No
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
866,163
</td>
</tr>
<tr>
<td style="text-align:right;">
No
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
841,712
</td>
</tr>
<tr>
<td style="text-align:right;">
No
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
753,210
</td>
</tr>
<tr>
<td style="text-align:right;">
No
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
595,545
</td>
</tr>
<tr>
<td style="text-align:right;">
No
</td>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
560,373
</td>
</tr>
<tr>
<td style="text-align:right;">
No
</td>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
452,152
</td>
</tr>
<tr>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
126,139
</td>
</tr>
<tr>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
130,347
</td>
</tr>
<tr>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
128,838
</td>
</tr>
<tr>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
131,248
</td>
</tr>
<tr>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
131,983
</td>
</tr>
<tr>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
131,697
</td>
</tr>
</tbody>
</table>

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

<span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(
    Year = <span class='k'>YEAR</span>,
    `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_hspd</span>, digits = <span class='m'>1</span>),
    `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_na</span>, digits = <span class='m'>1</span>),
    Yes = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>Yes</span>, digits = <span class='m'>0</span>),
    No = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>No</span>, digits = <span class='m'>0</span>),
    `NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>`NA`</span>, digits = <span class='m'>0</span>)
  ) <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>()

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
Year
</th>
<th style="text-align:right;">
Percent Yes
</th>
<th style="text-align:right;">
Percent NA
</th>
<th style="text-align:right;">
Yes
</th>
<th style="text-align:right;">
No
</th>
<th style="text-align:right;">
NA
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
67.7%
</td>
<td style="text-align:right;">
7.1%
</td>
<td style="text-align:right;">
2,763,511
</td>
<td style="text-align:right;">
1,320,348
</td>
<td style="text-align:right;">
311,436
</td>
</tr>
<tr>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
67.0%
</td>
<td style="text-align:right;">
7.3%
</td>
<td style="text-align:right;">
2,739,012
</td>
<td style="text-align:right;">
1,350,814
</td>
<td style="text-align:right;">
323,631
</td>
</tr>
<tr>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
69.8%
</td>
<td style="text-align:right;">
7.8%
</td>
<td style="text-align:right;">
2,847,329
</td>
<td style="text-align:right;">
1,231,992
</td>
<td style="text-align:right;">
345,771
</td>
</tr>
<tr>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
68.8%
</td>
<td style="text-align:right;">
5.9%
</td>
<td style="text-align:right;">
2,873,926
</td>
<td style="text-align:right;">
1,303,299
</td>
<td style="text-align:right;">
259,749
</td>
</tr>
<tr>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
69.2%
</td>
<td style="text-align:right;">
5.8%
</td>
<td style="text-align:right;">
2,902,850
</td>
<td style="text-align:right;">
1,294,981
</td>
<td style="text-align:right;">
256,358
</td>
</tr>
<tr>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
70.5%
</td>
<td style="text-align:right;">
5.7%
</td>
<td style="text-align:right;">
2,969,937
</td>
<td style="text-align:right;">
1,241,680
</td>
<td style="text-align:right;">
256,785
</td>
</tr>
</tbody>
</table>

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

<span class='k'>hspd_table</span> <span class='o'>&lt;-</span> <span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(
    Year = <span class='k'>YEAR</span>,
    `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_hspd</span>, digits = <span class='m'>1</span>),
    `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_na</span>, digits = <span class='m'>1</span>),
    Yes = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>Yes</span>, digits = <span class='m'>0</span>),
    No = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>No</span>, digits = <span class='m'>0</span>),
    `NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>`NA`</span>, digits = <span class='m'>0</span>)
  ) <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>(align = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='nf'><a href='https://rdrr.io/r/base/rep.html'>rep</a></span>(<span class='s'>"l"</span>, <span class='m'>6</span>)),
              <span class='nf'><a href='https://rdrr.io/r/base/list.html'>list</a></span>(
                `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/color_bar.html'>color_bar</a></span>(<span class='s'>"lightblue"</span>),
                `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/color_bar.html'>color_bar</a></span>(<span class='s'>"lightgrey"</span>)
              ))
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

<span class='k'>display_tbl</span> <span class='o'>%&gt;%</span> 
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>()

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
mean
</th>
<th style="text-align:right;">
sd
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
0.705127
</td>
<td style="text-align:right;">
0.002894493
</td>
</tr>
</tbody>
</table>

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

<span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>(<span class='k'>hint_tbl</span>)

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
mean
</th>
<th style="text-align:right;">
sd
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
0.7051774
</td>
<td style="text-align:right;">
0.00293509
</td>
</tr>
</tbody>
</table>

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

<span class='k'>boot_results</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>()

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
YEAR
</th>
<th style="text-align:right;">
sd
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
0.003242816
</td>
</tr>
<tr>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
0.003115310
</td>
</tr>
<tr>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
0.002885018
</td>
</tr>
<tr>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
0.002913172
</td>
</tr>
<tr>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
0.002840172
</td>
</tr>
<tr>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
0.002787895
</td>
</tr>
</tbody>
</table>

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

<span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(
    Year = <span class='k'>YEAR</span>,
    Race = <span class='k'>race</span>,
    `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_hspd</span>, digits = <span class='m'>1</span>),
    `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_na</span>, digits = <span class='m'>1</span>),
    Yes = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>Yes</span>, digits = <span class='m'>0</span>),
    No = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>No</span>, digits = <span class='m'>0</span>),
    `NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>`NA`</span>, digits = <span class='m'>0</span>)
  ) <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>(align = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='nf'><a href='https://rdrr.io/r/base/rep.html'>rep</a></span>(<span class='s'>"l"</span>, <span class='m'>7</span>)),
              <span class='nf'><a href='https://rdrr.io/r/base/list.html'>list</a></span>(
                `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/color_tile.html'>color_tile</a></span>(<span class='s'>"#ff7f7f"</span>, <span class='s'>"lightgreen"</span>),
                `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/color_tile.html'>color_tile</a></span>(<span class='s'>"lightgreen"</span>, <span class='s'>"#ff7f7f"</span>)
              ))

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:left;">
Year
</th>
<th style="text-align:left;">
Race
</th>
<th style="text-align:left;">
Percent Yes
</th>
<th style="text-align:left;">
Percent NA
</th>
<th style="text-align:left;">
Yes
</th>
<th style="text-align:left;">
No
</th>
<th style="text-align:left;">
NA
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
2013
</td>
<td style="text-align:left;">
All Others
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #acd18b">72.3%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #bfbe88">5.7%</span>
</td>
<td style="text-align:left;">
58,286
</td>
<td style="text-align:left;">
22,297
</td>
<td style="text-align:left;">
4,895
</td>
</tr>
<tr>
<td style="text-align:left;">
2014
</td>
<td style="text-align:left;">
All Others
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b1cc8a">70.4%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #aecf8b">4.0%</span>
</td>
<td style="text-align:left;">
65,866
</td>
<td style="text-align:left;">
27,716
</td>
<td style="text-align:left;">
3,909
</td>
</tr>
<tr>
<td style="text-align:left;">
2015
</td>
<td style="text-align:left;">
All Others
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #afce8b">71.1%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #cab387">6.9%</span>
</td>
<td style="text-align:left;">
59,620
</td>
<td style="text-align:left;">
24,258
</td>
<td style="text-align:left;">
6,191
</td>
</tr>
<tr>
<td style="text-align:left;">
2016
</td>
<td style="text-align:left;">
All Others
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a6d78c">74.8%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a9d48c">3.6%</span>
</td>
<td style="text-align:left;">
63,197
</td>
<td style="text-align:left;">
21,335
</td>
<td style="text-align:left;">
3,118
</td>
</tr>
<tr>
<td style="text-align:left;">
2017
</td>
<td style="text-align:left;">
All Others
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #afce8b">70.9%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #9ce18e">2.2%</span>
</td>
<td style="text-align:left;">
68,874
</td>
<td style="text-align:left;">
28,233
</td>
<td style="text-align:left;">
2,227
</td>
</tr>
<tr>
<td style="text-align:left;">
2018
</td>
<td style="text-align:left;">
All Others
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #aecf8b">71.6%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b6c78a">4.9%</span>
</td>
<td style="text-align:left;">
73,910
</td>
<td style="text-align:left;">
29,379
</td>
<td style="text-align:left;">
5,279
</td>
</tr>
<tr>
<td style="text-align:left;">
2013
</td>
<td style="text-align:left;">
Asian
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #90ee90">84.0%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #add08b">3.9%</span>
</td>
<td style="text-align:left;">
42,194
</td>
<td style="text-align:left;">
8,034
</td>
<td style="text-align:left;">
2,064
</td>
</tr>
<tr>
<td style="text-align:left;">
2014
</td>
<td style="text-align:left;">
Asian
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a2db8d">76.3%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #9be28e">2.1%</span>
</td>
<td style="text-align:left;">
41,559
</td>
<td style="text-align:left;">
12,906
</td>
<td style="text-align:left;">
1,189
</td>
</tr>
<tr>
<td style="text-align:left;">
2015
</td>
<td style="text-align:left;">
Asian
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #93ea8f">82.6%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a3da8d">2.9%</span>
</td>
<td style="text-align:left;">
48,766
</td>
<td style="text-align:left;">
10,284
</td>
<td style="text-align:left;">
1,764
</td>
</tr>
<tr>
<td style="text-align:left;">
2016
</td>
<td style="text-align:left;">
Asian
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a0dd8d">77.1%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #ed9081">10.3%</span>
</td>
<td style="text-align:left;">
43,768
</td>
<td style="text-align:left;">
13,001
</td>
<td style="text-align:left;">
6,550
</td>
</tr>
<tr>
<td style="text-align:left;">
2017
</td>
<td style="text-align:left;">
Asian
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #99e48e">80.3%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #90ed8f">1.1%</span>
</td>
<td style="text-align:left;">
52,465
</td>
<td style="text-align:left;">
12,878
</td>
<td style="text-align:left;">
703
</td>
</tr>
<tr>
<td style="text-align:left;">
2018
</td>
<td style="text-align:left;">
Asian
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #95e88f">81.8%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #9fde8d">2.5%</span>
</td>
<td style="text-align:left;">
52,838
</td>
<td style="text-align:left;">
11,733
</td>
<td style="text-align:left;">
1,660
</td>
</tr>
<tr>
<td style="text-align:left;">
2013
</td>
<td style="text-align:left;">
Black
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #bfbe88">64.4%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #dda084">8.8%</span>
</td>
<td style="text-align:left;">
192,209
</td>
<td style="text-align:left;">
106,359
</td>
<td style="text-align:left;">
28,707
</td>
</tr>
<tr>
<td style="text-align:left;">
2014
</td>
<td style="text-align:left;">
Black
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #ceaf86">58.4%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #e39a83">9.4%</span>
</td>
<td style="text-align:left;">
171,066
</td>
<td style="text-align:left;">
121,804
</td>
<td style="text-align:left;">
30,304
</td>
</tr>
<tr>
<td style="text-align:left;">
2015
</td>
<td style="text-align:left;">
Black
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #c5b887">61.8%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #fe7f7f">12.1%</span>
</td>
<td style="text-align:left;">
182,537
</td>
<td style="text-align:left;">
112,665
</td>
<td style="text-align:left;">
40,535
</td>
</tr>
<tr>
<td style="text-align:left;">
2016
</td>
<td style="text-align:left;">
Black
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #c6b787">61.6%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b3ca8a">4.5%</span>
</td>
<td style="text-align:left;">
204,021
</td>
<td style="text-align:left;">
127,215
</td>
<td style="text-align:left;">
15,628
</td>
</tr>
<tr>
<td style="text-align:left;">
2017
</td>
<td style="text-align:left;">
Black
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #c1bc88">63.8%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #9de08d">2.3%</span>
</td>
<td style="text-align:left;">
216,023
</td>
<td style="text-align:left;">
122,491
</td>
<td style="text-align:left;">
8,042
</td>
</tr>
<tr>
<td style="text-align:left;">
2018
</td>
<td style="text-align:left;">
Black
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #c5b887">62.1%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a3da8c">3.0%</span>
</td>
<td style="text-align:left;">
200,403
</td>
<td style="text-align:left;">
122,453
</td>
<td style="text-align:left;">
9,905
</td>
</tr>
<tr>
<td style="text-align:left;">
2013
</td>
<td style="text-align:left;">
Hispanic
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #e89582">47.6%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #aecf8b">4.0%</span>
</td>
<td style="text-align:left;">
21,571
</td>
<td style="text-align:left;">
23,758
</td>
<td style="text-align:left;">
1,904
</td>
</tr>
<tr>
<td style="text-align:left;">
2014
</td>
<td style="text-align:left;">
Hispanic
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #ff7f7f">38.4%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #ff7f7f">12.1%</span>
</td>
<td style="text-align:left;">
15,707
</td>
<td style="text-align:left;">
25,242
</td>
<td style="text-align:left;">
5,658
</td>
</tr>
<tr>
<td style="text-align:left;">
2015
</td>
<td style="text-align:left;">
Hispanic
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #ea9382">46.9%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #e89582">9.8%</span>
</td>
<td style="text-align:left;">
17,236
</td>
<td style="text-align:left;">
19,545
</td>
<td style="text-align:left;">
4,010
</td>
</tr>
<tr>
<td style="text-align:left;">
2016
</td>
<td style="text-align:left;">
Hispanic
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #ccb186">59.3%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #add08b">3.9%</span>
</td>
<td style="text-align:left;">
22,979
</td>
<td style="text-align:left;">
15,756
</td>
<td style="text-align:left;">
1,590
</td>
</tr>
<tr>
<td style="text-align:left;">
2017
</td>
<td style="text-align:left;">
Hispanic
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #d1ac85">57.1%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #90ee90">1.0%</span>
</td>
<td style="text-align:left;">
24,665
</td>
<td style="text-align:left;">
18,517
</td>
<td style="text-align:left;">
421
</td>
</tr>
<tr>
<td style="text-align:left;">
2018
</td>
<td style="text-align:left;">
Hispanic
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #cbb286">59.4%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #92eb8f">1.2%</span>
</td>
<td style="text-align:left;">
30,860
</td>
<td style="text-align:left;">
21,063
</td>
<td style="text-align:left;">
616
</td>
</tr>
<tr>
<td style="text-align:left;">
2013
</td>
<td style="text-align:left;">
White
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b7c689">67.9%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #add08b">3.9%</span>
</td>
<td style="text-align:left;">
2,449,251
</td>
<td style="text-align:left;">
1,159,900
</td>
<td style="text-align:left;">
147,727
</td>
</tr>
<tr>
<td style="text-align:left;">
2014
</td>
<td style="text-align:left;">
White
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b7c689">67.8%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #aecf8b">4.0%</span>
</td>
<td style="text-align:left;">
2,444,814
</td>
<td style="text-align:left;">
1,163,146
</td>
<td style="text-align:left;">
152,224
</td>
</tr>
<tr>
<td style="text-align:left;">
2015
</td>
<td style="text-align:left;">
White
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b0cd8a">70.4%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b1cc8a">4.4%</span>
</td>
<td style="text-align:left;">
2,539,170
</td>
<td style="text-align:left;">
1,065,240
</td>
<td style="text-align:left;">
164,433
</td>
</tr>
<tr>
<td style="text-align:left;">
2016
</td>
<td style="text-align:left;">
White
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b3ca8a">69.3%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a1dc8d">2.7%</span>
</td>
<td style="text-align:left;">
2,539,961
</td>
<td style="text-align:left;">
1,125,992
</td>
<td style="text-align:left;">
101,615
</td>
</tr>
<tr>
<td style="text-align:left;">
2017
</td>
<td style="text-align:left;">
White
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #b3ca8a">69.5%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a4d98c">3.0%</span>
</td>
<td style="text-align:left;">
2,540,823
</td>
<td style="text-align:left;">
1,112,862
</td>
<td style="text-align:left;">
112,982
</td>
</tr>
<tr>
<td style="text-align:left;">
2018
</td>
<td style="text-align:left;">
White
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #afce8b">71.2%</span>
</td>
<td style="text-align:left;">
<span style="display: block; padding: 0 4px; border-radius: 4px; background-color: #a2db8d">2.8%</span>
</td>
<td style="text-align:left;">
2,611,926
</td>
<td style="text-align:left;">
1,057,052
</td>
<td style="text-align:left;">
107,628
</td>
</tr>
</tbody>
</table>

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

<span class='k'>df_wide</span> <span class='o'>%&gt;%</span>
  <span class='nf'>transmute</span>(
    Year = <span class='k'>YEAR</span>,
    Poverty = <span class='k'>poverty</span>,
    `Percent Yes` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_hspd</span>, digits = <span class='m'>1</span>),
    `Percent NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/percent.html'>percent</a></span>(<span class='k'>percent_na</span>, digits = <span class='m'>1</span>),
    Yes = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>Yes</span>, digits = <span class='m'>0</span>),
    No = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>No</span>, digits = <span class='m'>0</span>),
    `NA` = <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/comma.html'>comma</a></span>(<span class='k'>`NA`</span>, digits = <span class='m'>0</span>)
  ) <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/pkg/formattable/man/formattable.html'>formattable</a></span>()

</code></pre>
<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
Year
</th>
<th style="text-align:right;">
Poverty
</th>
<th style="text-align:right;">
Percent Yes
</th>
<th style="text-align:right;">
Percent NA
</th>
<th style="text-align:right;">
Yes
</th>
<th style="text-align:right;">
No
</th>
<th style="text-align:right;">
NA
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
In Poverty
</td>
<td style="text-align:right;">
50.5%
</td>
<td style="text-align:right;">
7.4%
</td>
<td style="text-align:right;">
375,040
</td>
<td style="text-align:right;">
367,142
</td>
<td style="text-align:right;">
58,937
</td>
</tr>
<tr>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
In Poverty
</td>
<td style="text-align:right;">
48.7%
</td>
<td style="text-align:right;">
6.9%
</td>
<td style="text-align:right;">
365,417
</td>
<td style="text-align:right;">
385,039
</td>
<td style="text-align:right;">
55,449
</td>
</tr>
<tr>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
In Poverty
</td>
<td style="text-align:right;">
54.1%
</td>
<td style="text-align:right;">
9.3%
</td>
<td style="text-align:right;">
391,075
</td>
<td style="text-align:right;">
331,204
</td>
<td style="text-align:right;">
74,265
</td>
</tr>
<tr>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
In Poverty
</td>
<td style="text-align:right;">
50.1%
</td>
<td style="text-align:right;">
5.1%
</td>
<td style="text-align:right;">
389,899
</td>
<td style="text-align:right;">
388,031
</td>
<td style="text-align:right;">
41,794
</td>
</tr>
<tr>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
In Poverty
</td>
<td style="text-align:right;">
51.4%
</td>
<td style="text-align:right;">
4.7%
</td>
<td style="text-align:right;">
374,652
</td>
<td style="text-align:right;">
353,598
</td>
<td style="text-align:right;">
36,054
</td>
</tr>
<tr>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
In Poverty
</td>
<td style="text-align:right;">
52.9%
</td>
<td style="text-align:right;">
4.1%
</td>
<td style="text-align:right;">
376,231
</td>
<td style="text-align:right;">
335,489
</td>
<td style="text-align:right;">
30,551
</td>
</tr>
<tr>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
Near Poverty
</td>
<td style="text-align:right;">
58.1%
</td>
<td style="text-align:right;">
4.7%
</td>
<td style="text-align:right;">
485,960
</td>
<td style="text-align:right;">
350,733
</td>
<td style="text-align:right;">
41,305
</td>
</tr>
<tr>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
Near Poverty
</td>
<td style="text-align:right;">
57.2%
</td>
<td style="text-align:right;">
6.5%
</td>
<td style="text-align:right;">
477,329
</td>
<td style="text-align:right;">
357,115
</td>
<td style="text-align:right;">
57,724
</td>
</tr>
<tr>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
Near Poverty
</td>
<td style="text-align:right;">
60.9%
</td>
<td style="text-align:right;">
6.3%
</td>
<td style="text-align:right;">
483,503
</td>
<td style="text-align:right;">
310,406
</td>
<td style="text-align:right;">
53,675
</td>
</tr>
<tr>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
Near Poverty
</td>
<td style="text-align:right;">
60.7%
</td>
<td style="text-align:right;">
3.0%
</td>
<td style="text-align:right;">
489,235
</td>
<td style="text-align:right;">
317,045
</td>
<td style="text-align:right;">
25,027
</td>
</tr>
<tr>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
Near Poverty
</td>
<td style="text-align:right;">
58.4%
</td>
<td style="text-align:right;">
3.3%
</td>
<td style="text-align:right;">
476,175
</td>
<td style="text-align:right;">
339,251
</td>
<td style="text-align:right;">
28,090
</td>
</tr>
<tr>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
Near Poverty
</td>
<td style="text-align:right;">
61.6%
</td>
<td style="text-align:right;">
3.5%
</td>
<td style="text-align:right;">
499,269
</td>
<td style="text-align:right;">
311,347
</td>
<td style="text-align:right;">
29,697
</td>
</tr>
<tr>
<td style="text-align:right;">
2013
</td>
<td style="text-align:right;">
Not in Poverty
</td>
<td style="text-align:right;">
75.9%
</td>
<td style="text-align:right;">
3.3%
</td>
<td style="text-align:right;">
1,902,511
</td>
<td style="text-align:right;">
602,473
</td>
<td style="text-align:right;">
85,055
</td>
</tr>
<tr>
<td style="text-align:right;">
2014
</td>
<td style="text-align:right;">
Not in Poverty
</td>
<td style="text-align:right;">
75.7%
</td>
<td style="text-align:right;">
3.1%
</td>
<td style="text-align:right;">
1,896,266
</td>
<td style="text-align:right;">
608,660
</td>
<td style="text-align:right;">
80,111
</td>
</tr>
<tr>
<td style="text-align:right;">
2015
</td>
<td style="text-align:right;">
Not in Poverty
</td>
<td style="text-align:right;">
77.0%
</td>
<td style="text-align:right;">
3.4%
</td>
<td style="text-align:right;">
1,972,751
</td>
<td style="text-align:right;">
590,382
</td>
<td style="text-align:right;">
88,993
</td>
</tr>
<tr>
<td style="text-align:right;">
2016
</td>
<td style="text-align:right;">
Not in Poverty
</td>
<td style="text-align:right;">
76.9%
</td>
<td style="text-align:right;">
2.3%
</td>
<td style="text-align:right;">
1,994,792
</td>
<td style="text-align:right;">
598,223
</td>
<td style="text-align:right;">
61,680
</td>
</tr>
<tr>
<td style="text-align:right;">
2017
</td>
<td style="text-align:right;">
Not in Poverty
</td>
<td style="text-align:right;">
77.3%
</td>
<td style="text-align:right;">
2.2%
</td>
<td style="text-align:right;">
2,052,023
</td>
<td style="text-align:right;">
602,132
</td>
<td style="text-align:right;">
60,231
</td>
</tr>
<tr>
<td style="text-align:right;">
2018
</td>
<td style="text-align:right;">
Not in Poverty
</td>
<td style="text-align:right;">
77.9%
</td>
<td style="text-align:right;">
2.4%
</td>
<td style="text-align:right;">
2,094,437
</td>
<td style="text-align:right;">
594,844
</td>
<td style="text-align:right;">
64,840
</td>
</tr>
</tbody>
</table>

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

