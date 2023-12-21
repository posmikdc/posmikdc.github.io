---
layout: page
title: Multilayer FDR control with e-values
description: How can we control for false discoveries across partioned data?
img: assets/img/12.jpg
importance: 1
category: work
related_publications: e0instein1956investigations, einstein1950meaning
---

## Introduction

Sometimes we want to test hypotheses within groups or partitions, i.e. testing the effect of multiple drugs with many patients in each drug-specific cohort. In literature, there are well-established methods to control false discovery rates (FDR) in groups, i.e. the group-weighted Benjamini-Hochberg (weighted BH) procedure (see, [Benjamini & Cohen, 2017](https://academic.oup.com/biostatistics/article/18/1/91/2555340)). Now, a key weakness of this procedure is its invalidity when the independence assumption is violated. In practice, the independence assumption is usually strong. Thus, we are curious to explore FDR control within partitions under dependence scenarios. 

For this project, we consider e-values as an alternative to p-values since e-values allow for dependence (see, [Wang & Ramdas, 2019](https://academic.oup.com/jrsssb/article/84/3/822/7056146)). Specifically, we shall discuss e-values, their applications, transformations, and finally an e-filter adaptation to the p-filter procedure proposed in Ramdas et al, 2019 (The p-filter allows for multi-layer FDR control for grouped hypotheses).

[Wang & Ramdas, 2019](https://academic.oup.com/jrsssb/article/84/3/822/7056146) propose an interesting example in Section 8, aiming to detect cryptocurrencies with a positive return. We take inspiration from this example, but in the interest of time and conciseness, modify their example in several meaningful ways. First, we do not have access to the data, so we simulate year-to-year rate of change in stock price (see details below). Next, we do not consider this example in an investment setting, i.e. we set their investment parameter $\lambda$ equal to 1. That means that we hold constant our investment and thus interested in identifying which coins seem promising in what year.

## Generating data and e-values
E-values are heavily context-dependent. Depending on the analyst’s level of familiarity with the setting, this may be good or bad. For this simulation, we adapt a modified version of the example in Section (8) in Wang & Ramdas. In essence, the authors propose an e-value approach to detecting cryptocurrencies with positive expected return. There are K different cryptocurrencies (“coins”) over T periods of time.

We are modifying the authors’ example as follows: Since we don’t have access to their data, we consider two distinct data-generating processes to obtain our year-to-year stock rate of change. First, we consider a scenario where all values are sampled i.i.d. from a truncated normal distribution, $TN(\mu=1,\sigma=1;trunc-null,\infty)$. This will yield the year-to-year percentage positive change in the price of stock of a specific cryptocurrency, e.g. $Xi,j>1$ if a coin’s value had increased. Second, we are considering a sequential dependence scenario inspired by the authors’ example. Rather than sampling data i.i.d. from a truncated normal with mean 1, we introduce sequential dependence by letting a previous realization become the next period’s mean.



















Every project has a beautiful feature showcase page.
It's easy to include images in a flexible 3-column grid format.
Make your photos 1/3, 2/3, or full width.

To give your project a background in the portfolio page, just add the img tag to the front matter like so:

    ---
    layout: page
    title: project
    description: a project with a background image
    img: /assets/img/12.jpg
    ---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images.
Say you wanted to write a little bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, *bled* for your project, and then... you reveal its glory in the next row of images.


<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>


The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}
```html
<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
```
{% endraw %}
