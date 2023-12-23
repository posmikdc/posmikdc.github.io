---
layout: page
title: CNNs and Uncertainty Quantification
description: 
img: assets/img/cnn_uq.png 
importance: 3
category: theory
---

## Applying Uncertainty Quantification on a Convolutional Neural Net Algorithm 

{% if site.data.repositories.github_repos %}

{% for repo in site.data.repositories.github_repos %} {% include repository/repo.html repository=repo %} {% endfor %}
{% endif %}

## Introduction

Using a Convolutional Neural Net (CNN), I determine whether the pictures are deer, frogs, or trucks. I design a Neural Net that includes three convolutional layers (they pick up on the structure in the picture) and feed the output into dense layers. The classification accuracy is just over 90%. Additionally, I apply Grad CAM to visualize which image structure led to the classification decision. Here is an example using a cat and a dog:

![ChartChat1-3](https://miro.medium.com/max/1186/0*D4FATkIeWp61o9zo.jpg)

## MC Dropout 

Next, I use Monte Carlo (MC) Dropout to construct confidence intervals around my decisions (→ "Uncertainty Quantification). MC Dropout varies the Neural Net architecture to induce some variation into the classification decision, then we can measure the resulting deviation as a confidence interval. If the Neural Net architecture varie  and the classification decisions stay the same, we have constructed a great (→ robust) model and the confidence intervals will be slim. Here is a visual explanation:

![ChartChat1-3](https://docs.aws.amazon.com/prescriptive-guidance/latest/ml-quantifying-uncertainty/images/mc-dropout.png)

## Variational Neural Network

These are the results for my model, where Cat. 1 - 3 correspond to Frogs, Deer, and Trucks respectively. The confidence intervals are reasonable, and especially slim for Cat. 3 (→ Trucks) which means the CNN does an outstanding job at classifying trucks. This makes sense seeing that Trucks are not animals and look considerably less alike than Deer and Frogs (who at least share rudimentary facial features). 

Lastly, I apply a second technique to quantify uncertainty: A variational neural network (VNN). A VNN introduces a hidden layer "Z" which adds uncertainty to the weights assigned during _regular_ backpropagation. The results indicate similar performance as the MC Dropout. 

![ChartChat1-3](https://miro.medium.com/max/848/1*6uuK7GpIbfTb-0chqFwXXw.png)
