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
rmd_hash: 1cdd1911ad15c5fe

---

Getting the Data
================

The easiest way to get census microdata is through the Integrated Public Use Microdata Series (IPUMS) hosted by the University of Minnesota. While you can get the data directly from the Census Bureau, IPUMS has made it much easier to compare across multiple years and to select the variables you want. IPUMS also provides a codebook that is easy to refer to and notes any important changes from year to year.

I've put the data for just Kentucky up on GitHub, so I'll read it in from there and then we can take a quick look at it. I like [`skimr::skim()`](https://docs.ropensci.org/skimr/reference/skim.html) for

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='http://tidyverse.tidyverse.org'>tidyverse</a></span>)

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

