---
output: hugodown::md_document
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Internet Access in Louisville"
subtitle: ""
summary: ""
authors: [admin]
tags: [internet, data science, R, louisville]
categories: [Louisville, Data Science]
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
rmd_hash: b2c36fc40f8bdedd

---

Overall
=======

This report uses census Microdata from IPUMS to look at internet access in the Louisville MSA. Data is available from 2013 to 2018. Louisville ranks towards the bottom of our peer cities in percent of people with home internet access, although the differences among the cities are relatively small (The top city, Grand Rapids, is only 4 percentage points ahead of Louisville).

<div class="highlight">

<img src="figs/unnamed-chunk-1-1.png" width="700px" style="display: block; margin: auto;" />

</div>

<div class="highlight">

</div>

History
=======

Louisville has increased its internet access over time, including a large jump in 2016. (We do not currently have an explanation for this jump, so if you know of something that happened in that time period let us know). In the past few years access has leveled off at just above 90 percent of the population

<div class="highlight">

<img src="figs/unnamed-chunk-3-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Poverty
=======

There is an access gap between poor and nonpoor individuals in having internet access at home, although this gap has shrunk a little as overall access has increased.

<div class="highlight">

<img src="figs/unnamed-chunk-4-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Race
====

We also see a gap between Black and White households although this has closed dramatically since 2014.

<div class="highlight">

<img src="figs/unnamed-chunk-5-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Age
===

Finally, we see an age related gap, as individuals over the age of 65 are much less likely to be in a household with internet access.

<div class="highlight">

<img src="figs/unnamed-chunk-6-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Devices
=======

-   80% of Louisville households have a computer in thier household, unlike internet access this is down a little bit from 2013 (82%).
-   69% have a tablet in their household.
-   85% have either a computer or a tablet.
-   89% of households have a smartphone.
-   94% have either a smartphone or internet access at home.

JCPS additional capacity
========================

As schools transitioned to being online, JCPS distibuted:

-   20,833 Chromebooks
-   517 Hotspots

This is not a large enough distribution to change our standing relative to peer cities, but it does increase internet access among JCPS students.

This blog was originally written for the [Greater Louisville Project](https://greaterlouisvilleproject.org/blog/internet-access/) and is being posted here to reach an additional audience. Please refer to the Greater Louisville Project [post](https://greaterlouisvilleproject.org/blog/internet-access/) when referencing this work.

