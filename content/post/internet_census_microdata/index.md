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
rmd_hash: 09960f6d95246c8e

---

Getting the Data
================

The easiest way to get census microdata is through the Integrated Public Use Microdata Series (IPUMS) hosted by the University of Minnesota. While you can get the data directly from the Census Bureau, IPUMS has made it much easier to compare across multiple years and to select the variables you want. IPUMS also provides a codebook that is easy to refer to and notes any important changes from year to year.

I've put the data for just Kentucky up on GitHub, so I'll read it in from there and then we can take a quick look at it. I like [`skimr::skim()`](https://docs.ropensci.org/skimr/reference/skim.html) for taking a look at the data. When downloading the data it's all numeric, even for variables that are categorial - they've been coded and our first step in the analysis will be using the code book to translate them.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='http://tidyverse.tidyverse.org'>tidyverse</a></span>)

<span class='c'>#&gt; Warning: replacing previous import 'vctrs::data_frame' by 'tibble::data_frame' when loading 'dplyr'</span>

<span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='https://renkun.me/formattable'>formattable</a></span>)

<span class='k'>df</span> <span class='o'>&lt;-</span> <span class='nf'>read_csv</span>(<span class='s'>"https://raw.github.com/natekratzer/raw_data/master/ky_high_speed_internet.csv"</span>)

<span class='k'>skimr</span>::<span class='nf'><a href='https://docs.ropensci.org/skimr/reference/skim.html'>skim</a></span>(<span class='k'>df</span>)

</code></pre>

|                                                  |        |
|:-------------------------------------------------|:-------|
| Name                                             | df     |
| Number of rows                                   | 270037 |
| Number of columns                                | 23     |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |        |
| Column type frequency:                           |        |
| numeric                                          | 23     |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |        |
| Group variables                                  | None   |

**Variable type: numeric**

| skim\_variable |  n\_missing|  complete\_rate|          mean|            sd|        p0|       p25|      p50|           p75|         p100| hist  |
|:---------------|-----------:|---------------:|-------------:|-------------:|---------:|---------:|--------:|-------------:|------------:|:------|
| YEAR           |           0|               1|  2.015510e+03|  1.710000e+00|    2013.0|    2014.0|     2016|  2.017000e+03|  2.01800e+03| ▇▃▃▅▅ |
| SERIAL         |           0|               1|  5.311484e+05|  9.272230e+03|  512037.0|  524218.0|   530769|  5.377690e+05|  5.53112e+05| ▃▇▇▆▂ |
| CBSERIAL       |           0|               1|  6.775195e+11|  9.528205e+11|       4.0|  571782.0|  1141246|  2.017001e+12|  2.01801e+12| ▇▁▁▁▅ |
| HHWT           |           0|               1|  9.482000e+01|  7.631000e+01|       1.0|      48.0|       75|  1.180000e+02|  1.16600e+03| ▇▁▁▁▁ |
| STATEFIP       |           0|               1|  2.100000e+01|  0.000000e+00|      21.0|      21.0|       21|  2.100000e+01|  2.10000e+01| ▁▁▇▁▁ |
| DENSITY        |           0|               1|  1.013530e+03|  1.343600e+03|      60.1|     108.2|      275|  2.088500e+03|  4.86960e+03| ▇▁▂▁▁ |
| METRO          |           0|               1|  1.640000e+00|  1.590000e+00|       0.0|       0.0|        1|  3.000000e+00|  4.00000e+00| ▇▆▂▂▆ |
| MET2013        |           0|               1|  1.052762e+04|  1.391265e+04|       0.0|       0.0|        0|  3.114000e+04|  3.69800e+04| ▇▁▂▁▃ |
| PUMA           |           0|               1|  1.524210e+03|  7.607400e+02|     100.0|     900.0|     1702|  2.000000e+03|  2.80000e+03| ▅▃▇▅▅ |
| GQ             |           0|               1|  1.130000e+00|  5.500000e-01|       1.0|       1.0|        1|  1.000000e+00|  4.00000e+00| ▇▁▁▁▁ |
| CINETHH        |           0|               1|  1.270000e+00|  7.700000e-01|       0.0|       1.0|        1|  1.000000e+00|  3.00000e+00| ▁▇▁▁▂ |
| CIHISPEED      |           0|               1|  9.660000e+00|  6.120000e+00|       0.0|      10.0|       10|  1.300000e+01|  2.00000e+01| ▃▁▇▂▂ |
| PERNUM         |           0|               1|  2.000000e+00|  1.240000e+00|       1.0|       1.0|        2|  3.000000e+00|  1.20000e+01| ▇▁▁▁▁ |
| PERWT          |           0|               1|  9.848000e+01|  8.085000e+01|       1.0|      49.0|       77|  1.220000e+02|  1.21100e+03| ▇▁▁▁▁ |
| SEX            |           0|               1|  1.510000e+00|  5.000000e-01|       1.0|       1.0|        2|  2.000000e+00|  2.00000e+00| ▇▁▁▁▇ |
| AGE            |           0|               1|  4.119000e+01|  2.362000e+01|       0.0|      20.0|       42|  6.000000e+01|  9.30000e+01| ▇▇▇▇▂ |
| RACE           |           0|               1|  1.300000e+00|  1.190000e+00|       1.0|       1.0|        1|  1.000000e+00|  9.00000e+00| ▇▁▁▁▁ |
| RACED          |           0|               1|  1.301500e+02|  1.216300e+02|     100.0|     100.0|      100|  1.000000e+02|  9.90000e+02| ▇▁▁▁▁ |
| HISPAN         |           0|               1|  5.000000e-02|  3.900000e-01|       0.0|       0.0|        0|  0.000000e+00|  4.00000e+00| ▇▁▁▁▁ |
| HISPAND        |           0|               1|  5.550000e+00|  4.126000e+01|       0.0|       0.0|        0|  0.000000e+00|  4.98000e+02| ▇▁▁▁▁ |
| EDUC           |           0|               1|  5.790000e+00|  3.120000e+00|       0.0|       4.0|        6|  7.000000e+00|  1.10000e+01| ▅▁▇▃▅ |
| EDUCD          |           0|               1|  6.047000e+01|  3.104000e+01|       1.0|      40.0|       63|  7.100000e+01|  1.16000e+02| ▃▂▇▃▅ |
| POVERTY        |           0|               1|  2.755700e+02|  1.719900e+02|       0.0|     126.0|      268|  4.610000e+02|  5.01000e+02| ▅▅▅▃▇ |

</div>

Cleaning the Data
=================

I won't show all the codebooks, but for this first variable let's take a look at what IPUMS has to say. For all years N/A is coded as 00 and No high speed internet is coded as 20. Prior to 2016 there are detailed codes for the type of internet access, while for 2016 and after the code is collapsed.

![](high_speed_code.png)

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
  <span class='nf'>summarize</span>(count = <span class='nf'>n</span>())

<span class='c'>#&gt; `summarise()` regrouping output by 'hspd_int' (override with `.groups` argument)</span>


<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='k'>YEAR</span>, names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_NA = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>))) 

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

<span class='c'># Using the GT package to format the table</span>
<span class='c'># df_wide %&gt;%</span>
<span class='c'>#   gt() %&gt;%</span>
<span class='c'>#   tab_header("Weighted (but still wrong) High Speed Internet Results") %&gt;%</span>
<span class='c'>#   fmt_number(columns = vars(No, Yes, `NA`),</span>
<span class='c'>#              decimals = 0) %&gt;%</span>
<span class='c'>#   fmt_percent(columns = vars(percent_hspd, percent_NA),</span>
<span class='c'>#               decimals = 1) %&gt;%</span>
<span class='c'>#   cols_label(</span>
<span class='c'>#     YEAR = "Year",</span>
<span class='c'>#     percent_hspd = "Percent Yes",</span>
<span class='c'>#     percent_NA = "Percent NA"</span>
<span class='c'>#   ) %&gt;%</span>
<span class='c'>#   cols_move(</span>
<span class='c'>#     columns = vars(percent_hspd, Yes, No, `NA`, percent_NA),</span>
<span class='c'>#     after = vars(YEAR)</span>
<span class='c'>#   )</span>
</code></pre>

</div>

This is better. The second problem is harder to spot. There are 3 hints in the data:

1.  There are more NA values than there are people who said they don't have access, and in general a very high percentage of NA responses.
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
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>))

<span class='c'>#&gt; `summarise()` regrouping output by 'hspd_int', 'int' (override with `.groups` argument)</span>


<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>YEAR</span>, <span class='k'>int</span>), names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)),
         percent_NA = (<span class='k'>`NA`</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>)))


<span class='c'># Using the GT package to format the table</span>
<span class='c'># df_wide %&gt;%</span>
<span class='c'>#   gt() %&gt;%</span>
<span class='c'>#   tab_header("Exploring Internet Results") %&gt;%</span>
<span class='c'>#   fmt_number(columns = vars(No, Yes, `NA`),</span>
<span class='c'>#              decimals = 0) %&gt;%</span>
<span class='c'>#   fmt_percent(columns = vars(percent_hspd, percent_NA),</span>
<span class='c'>#               decimals = 1) %&gt;%</span>
<span class='c'>#   cols_label(</span>
<span class='c'>#     YEAR = "Year",</span>
<span class='c'>#     percent_hspd = "Percent Yes",</span>
<span class='c'>#     percent_NA = "Percent NA",</span>
<span class='c'>#     int = "Internet Access"</span>
<span class='c'>#   ) %&gt;%</span>
<span class='c'>#   cols_move(</span>
<span class='c'>#     columns = vars(percent_hspd, Yes, No, `NA`, percent_NA),</span>
<span class='c'>#     after = vars(YEAR)</span>
<span class='c'>#   ) %&gt;%</span>
<span class='c'>#   tab_row_group(</span>
<span class='c'>#     group = "Yes",</span>
<span class='c'>#     rows = int == "Yes"</span>
<span class='c'>#   )</span>
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
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>))

<span class='c'>#&gt; `summarise()` regrouping output by 'hspd_int' (override with `.groups` argument)</span>


<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>YEAR</span>), names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = <span class='nf'><a href='https://rdrr.io/r/base/Round.html'>round</a></span>(<span class='m'>100</span> <span class='o'>*</span> (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)), <span class='m'>1</span>))

<span class='k'>knitr</span>::<span class='nf'><a href='https://rdrr.io/pkg/knitr/man/kable.html'>kable</a></span>(<span class='k'>df_wide</span>)

</code></pre>

|  YEAR|       No|      Yes|      NA|  percent\_hspd|
|-----:|--------:|--------:|-------:|--------------:|
|  2013|  1320348|  2763511|  311436|           67.7|
|  2014|  1350814|  2739012|  323631|           67.0|
|  2015|  1231992|  2847329|  345771|           69.8|
|  2016|  1303299|  2873926|  259749|           68.8|
|  2017|  1294981|  2902850|  256358|           69.2|
|  2018|  1241680|  2969937|  256785|           70.5|

</div>

These results look much better, although still quite a few N/A results.

Group Quarters in the Census
----------------------------

The census data includes individuals living in group quarters (mostly prisons, senior living centers, and dorms, but includes any sort of communal living arrangement). However, all census questions about appliances and utilities (the category that internet access falls under) are NA for group quarters. So we'll add one more line to filter out individuals living in group quarters (a common practice when working with microdata). The code below adds a filter for Group Quarters and then also adds a column to show the percent of all responses that are NA.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Count numbers with and without high speed internet</span>
<span class='k'>df_group</span> <span class='o'>&lt;-</span> <span class='k'>df</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span>(<span class='k'>GQ</span> <span class='o'>==</span> <span class='m'>1</span> <span class='o'>|</span> <span class='k'>GQ</span> <span class='o'>==</span><span class='m'>2</span> <span class='o'>|</span> <span class='k'>GQ</span> <span class='o'>==</span> <span class='m'>5</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>group_by</span>(<span class='k'>hspd_int</span>, <span class='k'>YEAR</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>summarize</span>(count = <span class='nf'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span>(<span class='k'>PERWT</span>))

<span class='c'>#&gt; `summarise()` regrouping output by 'hspd_int' (override with `.groups` argument)</span>


<span class='c'># Pivot for easier percent calculations</span>
<span class='k'>df_wide</span> <span class='o'>&lt;-</span> <span class='k'>df_group</span>  <span class='o'>%&gt;%</span>
  <span class='nf'>pivot_wider</span>(id_cols = <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span>(<span class='k'>YEAR</span>), names_from = <span class='k'>hspd_int</span>, values_from = <span class='k'>count</span>) <span class='o'>%&gt;%</span>
  <span class='nf'>mutate</span>(percent_hspd = <span class='nf'><a href='https://rdrr.io/r/base/Round.html'>round</a></span>(<span class='m'>100</span> <span class='o'>*</span> (<span class='k'>Yes</span> <span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span>)), <span class='m'>1</span>),
         percent_na = <span class='nf'><a href='https://rdrr.io/r/base/Round.html'>round</a></span>(<span class='m'>100</span> <span class='o'>*</span> (<span class='k'>`NA`</span><span class='o'>/</span> (<span class='k'>Yes</span> <span class='o'>+</span> <span class='k'>No</span> <span class='o'>+</span> <span class='k'>`NA`</span>)), <span class='m'>1</span>))

<span class='k'>knitr</span>::<span class='nf'><a href='https://rdrr.io/pkg/knitr/man/kable.html'>kable</a></span>(<span class='k'>df_wide</span>)

</code></pre>

|  YEAR|       No|      Yes|      NA|  percent\_hspd|  percent\_na|
|-----:|--------:|--------:|-------:|--------------:|------------:|
|  2013|  1320348|  2763511|  185297|           67.7|          4.3|
|  2014|  1350814|  2739012|  193284|           67.0|          4.5|
|  2015|  1231992|  2847329|  216933|           69.8|          5.0|
|  2016|  1303299|  2873926|  128501|           68.8|          3.0|
|  2017|  1294981|  2902850|  124375|           69.2|          2.9|
|  2018|  1241680|  2969937|  125088|           70.5|          2.9|

</div>

That removed about half of our NA values. It might be nice to know a bit more about the missing data, but at around 3 percent of observations it's unlikely to change our substantive conclusions. I suspect these are cases where there wasn't an answer for that question. We'll keep an eye on NA values as we do the analysis, because as we get into questions like how internet access varies by race, income, age, and education we'll want to know if NA answers are more or less likely in any of those categories.

Analysis
========

Race, Poverty, and Geography
----------------------------

Visualizing the Data
====================

Mapping the Data
================

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># Internet Overall</span>

</code></pre>

</div>

