---
layout: page
title: Multilayer FDR control with e-values
description:
img: assets/img/multilayer_fdr.png 
importance: 1
category:
related_publications:
---

## Introduction

Sometimes we want to test hypotheses within groups or partitions, i.e. testing the effect of multiple drugs with many patients in each drug-specific cohort. In literature, there are well-established methods to control false discovery rates (FDR) in groups, i.e. the group-weighted Benjamini-Hochberg (weighted BH) procedure (see, [Benjamini & Cohen, 2017](https://academic.oup.com/biostatistics/article/18/1/91/2555340)). Now, a key weakness of this procedure is its invalidity when the independence assumption is violated. In practice, the independence assumption is usually strong. Thus, we are curious to explore FDR control within partitions under dependence scenarios. 

For this project, we consider e-values as an alternative to p-values since e-values allow for dependence (see, [Wang & Ramdas, 2022](https://academic.oup.com/jrsssb/article/84/3/822/7056146)). Specifically, we shall discuss e-values, their applications, transformations, and finally an e-filter adaptation to the p-filter procedure proposed in [Ramdas et al, 2019](https://arxiv.org/abs/1703.06222) (The p-filter allows for multi-layer FDR control for grouped hypotheses).

[Wang & Ramdas, 2022](https://academic.oup.com/jrsssb/article/84/3/822/7056146) propose an interesting example in Section 8, aiming to detect cryptocurrencies with a positive return. We take inspiration from this example, but in the interest of time and conciseness, modify their example in several meaningful ways. First, we do not have access to the data, so we simulate year-to-year rate of change in stock price (see details below). Next, we do not consider this example in an investment setting, i.e. we set their investment parameter $$\lambda$$ equal to 1. That means that we hold constant our investment and thus interested in identifying which coins seem promising in what year.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/multilayer_fdr.png" title="Hypotheses across partitions" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This figure was taken from Ramdas et al., 2019 and visualizes various partitions and groupings of hypotheses
</div>


## Generating data and e-values

E-values are heavily context-dependent. Depending on the analyst's level of familiarity with the setting, this may be good or bad. For this simulation, we adapt a modified version of the example in Section (8) in [Wang & Ramdas, 2022](https://academic.oup.com/jrsssb/article/84/3/822/7056146). In essence, the authors propose an e-value approach to detecting cryptocurrencies with positive expected return. There are $$K$$ different cryptocurrencies ("coins") over $$T$$ periods of time.

We are modifying the authors' example as follows: Since we don't have access to their data, we consider two distinct data-generating processes to obtain our year-to-year stock rate of change. First, we consider a scenario where all values are sampled i.i.d. from a truncated normal distribution, $$TN(\mu = 1,\sigma = 1;\text{trunc-null},\infty)$$. This will yield the year-to-year percentage positive change in the price of stock of a specific cryptocurrency, e.g. $$X_{i,j} > 1$$ if a coin's value had increased. Second, we are considering a sequential dependence scenario inspired by the authors' example. Rather than sampling data i.i.d. from a truncated normal with mean 1, we introduce sequential dependence by letting a previous realization become the next period's mean.

$$
\text{Initialize} X_{k,t_0} = 1. \text{Then,} 
$$ \\
$$
X_{k,1} \sim TN(\mu = X_{k,t_0}, \sigma = 1; \text{trunc-null}, \infty),
$$
$$
X_{k,2} \sim TN(\mu = X_{k,1}, \sigma = 1; \text{trunc-null}, \infty),\, \ldots,\, 
$$
$$
X_{k,T} \sim TN(\mu = X_{k,T-1}, \sigma = 1; \text{trunc-null}, \infty)
$$

In the i.i.d scenario, we will calculate e-values as the cumulative product of $$X_{ij}$$: 

$$
E_{k,t} = \prod_{j=1}^{t} (X_{k,j}), t=1,...,T.
$$

This is equivalent to the authors' derivation of e-values, $$E_{k,t} = \prod_{j=1}^{t} (1 - \lambda + \lambda X_{k,j}), t=1,...,T.$$, when setting the investment parameter $$\lambda = 1$$.

```{r}
# Inputs
nrow = 4; ncol = 5
trunc_null = 0; trunc_signal = 4

# Matrix shell
X_iid <- matrix(NA, nrow = nrow, ncol = ncol)

# Sampling Loop
for (k in 1:nrow) {
  for (t in 1:ncol) {
    if (k %% 2 == 0){
      # Sample a signal from truncated normal distribution
      X_iid[k, t] <- rtruncnorm(1, a = trunc_signal, mean = 1, sd = 1)
    } else{
      # Sample a null from truncated normal distribution
      X_iid[k, t] <- rtruncnorm(1, a = trunc_null, mean = 1, sd = 1)
    }
    
  }
}

# Print
X_iid
```

```{r}
# Inputs
nrow = 4; ncol = 5
trunc_null = 0; trunc_signal = 4

# Matrix shell
X_dep <- matrix(NA, nrow = nrow, ncol = ncol)

# Sampling Loop
for (k in 1:nrow) {
  # Initialize the mean for the current row
  row_mean <- 1
  
  for (t in 1:ncol) {
    if (k %% 2 == 0){
      # Sample from a truncated normal distribution with lower bound of 0
      X_dep[k, t] <- rtruncnorm(1, a = trunc_signal, mean = row_mean, sd = 1)
    } else {
      # Sample from a truncated normal distribution with lower bound of 0
      X_dep[k, t] <- rtruncnorm(1, a = trunc_null, mean = row_mean, sd = 1)
    }
    
    # Update the mean for the next sample in the same row
    row_mean <- X_dep[k, t]
  }
}

# Print
X_dep
```

The matrices `X_iid` and `X_dep` both represent the year-to-year percentage positive change in the price of stock of a specific cryptocurrency. However, `X_dep` represents sequentially dependent data. Note that we have also introduced signals into the matrices by varying the truncation cutoff. For even coins, i.e. `k %% 2 == 0`, our realizations of year-to-year growth are significantly higher than for the other coins, symbolizing a signal. 

$$
E_{k,t} = \prod_{j=1}^{t} (X_{k,j}), \quad t=1,...,T.
$$

```{r}
# E-values dependent
E_dep <- X_dep

# E-values iid
E_iid <- matrix(NA, nrow = nrow, ncol = ncol)

# Calculate E_iid matrix via cumulative product
for (i in 1:nrow) {
  for (j in 1:ncol) {
      E_iid[i, j] <- prod(X_iid[i, 1:j])
  }}
    

# Print
E_iid
```

Since the `X_dep` entries are already sequentially dependent, we consider them as e-values themselves, on grounds of the dependence. Our entries in the `X_iid` matrix are converted to e-values using the cumulative product, as outlined previously. The purpose of these different approaches is to create dependent e-values in two different ways. 

We have now constructed our matrix of e-values, both for the iid (`E_iid`) and the sequential dependence case (`E_dep`). Immediately, we can see that the `E_iid` values explode with time. There is significant difference between earlier and later time periods. This makes sense since we derived the e-values using the cumulative product. In contrast, the `E_dep` values are relatively homogeneous across columns. There are visible differences across rows, owing to the truncation specific to signals and nulls. 

Both ways of constructing e-values have their pros and cons. The `E_iid` e-values that were derived are very sensitive to larger initial quantities. This could be especially valuable to an analyst only interested in controlling FDR on the group-level, i.e. the coin (more on that in the section: `Group-specific p-values`). In contrast, taking the `E_dep` e-values at face value may be valuable if an analyst does care about controlling FDR over multiple partitions, e.g., time. The `E_iid` e-values may be less fit for this finer analysis since the exploding values in later time periods dillute the accurate encoding of information. This discussion illustrates well how important it is to carefully weigh the pros and cons of using e-values vs. p-values. 

## Transforming e-values to p-values

A key strength of e-values is their flexibility, i.e. we have many ways to create e-values, we may combine them, and there are multiple ways to transform e-values to p-values. Generally speaking, we can define a transformation of e-values to p-values as a e-to-p calibrator, i.e.

Let $$ f : [0, \infty) \rightarrow [0, 1] $$ be a decreasing function. It is an e-to-p calibrator if, for any e-variable E, $$f(E)$$ is a p-variable (i.e., f transforms e-values to p-values).

Although this flexibility is great, it comes with the danger of losing valuable information if this calibration is not done in a way that matches the context of the e-values. In the following, we contrast two ways to convert e-values to p-values and illustrate their respective advantages and drawbacks. The conversion is as follows (see Remark 1 in Wang & Ramdas, 2021):

$$
p_{k,t} = \frac{1}{e_{k,t}}
$$

```{r}
# Calculate idiosyncratic p-values
P_iid <- matrix(1/E_iid, nrow = nrow, ncol = ncol)
P_dep <- matrix(1/E_dep, nrow = nrow, ncol = ncol)

# Print
P_iid
P_dep

# Truncate values greater than 1
P_iid[P_iid > 1] <- 1
P_dep[P_dep > 1] <- 1

# Print
P_iid
P_dep

# Unlist
Pvec_iid <- as.vector(P_iid)
Pvec_dep <- as.vector(P_dep)
```

Looking at the `P_iid` matrix, we can immediately see a problem with this conversion. If an e-value is less than 1, it yields a p-value over '1', thus yielding invalid probabilities. The solution is to truncate values that exceed 1 at 1, which does give us valid p-values, but we lose information. Intuitively, this could mean that this conversion is better fit for situations where e-values tend to be larger.

## Group-specific p-values

An interesting application of e-values in this setting--in line with the authors' example--is to reduce the analysis back to the group-setting, similar to the authors' goal. Taking inspiration from the authors, we can define a group-specific p-value as:

$$
P_{k,j} := \max (E_{k,i})^{-1}
$$

Although we lose a layer of information, i.e. we can no longer partition across additional groups (which we will consider later in the `p-filter` example), we can get good insight into each group hypothesis. To continue the previous arguement, here the exploding e-values are less of an issue because we are only consider the maximum row-wise e-value. We care less about the between-column heterogeneity of e-values and only hope to make valid inference on the coin-level. Note that because we are choosing the maximum e-value, we are also minimizing the risk that our raw p-value exceeds 1. This is a hint that this procedure might be a good fit for this scenario. Alternatively, we could combine the e-values by obtaining the average which is valid under dependence. In an iid scenario, we could also add e-values.

We think this e-value design coupled with group-wise inference could be relevant to scenarios where signals of varying strength are present. The exploding nature of the cumulative product would, given enough iterations, also magnify signals of lesser magnitude. This could be a great fit in a scenario where the analyst puts a lot of emphasis on finding weaker signals, i.e. a geneticist interested in locating loci associated with a certain disease, even those that have less association (Think: Genome-wide association studies).

We can see that the p-values are indeed lower for the signal (recall that even row indices are classified as signals) coins. The differences in p-values are far more pronounced in the `Pg_iid` vector, owing to the large maximum e-values. The group-wise p-values for the `Pg_dep` case are high, but do not vary as drastically between nulls and signals. 

## An `e-filter` approach

Although the previous finding is promising in terms of conducting inference under dependence, what happens if we are interested controlling for FDR beyond the group-level? That question is why we will explore the `pfilter` function. The `p-filter` is designed to guarantee FDR control across multiple partitions of hypotheses. Our endeavor in exploring the `p-filter` in an e-value framework is to reformulate the computational task of running `e-filter` into a `p-filter` task so that we can use the existing code. 

```{r}
pfilter = function(P,alphas,groups){
	# P in [0,1]^n = vector of p-values
	# alphas in [0,1]^M = vector of target FDR levels
	# groups is a n-by-M matrix; 
	#	groups[i,m] = which group does P[i] belong to,
	#		for the m-th grouping
	
	n = length(P)
	M = length(alphas)
	G = apply(groups,2,max) # G[m] = # groups, for grouping m
	
	
	Simes = list()
	for(m in 1:M){
		Simes[[m]]=rep(0,G[m])
		for(g in 1:G[m]){
			group = which(groups[,m]==g)
			Simes[[m]][g]=min(sort(P[group])*length(group)/(1:length(group)))
		}
	}
	
		
	# initialize
	thresh = alphas
	Sh = 1:n
	for(m in 1:M){
		pass_Simes_m = which(is.element(groups[,m],which(Simes[[m]]<=thresh[m])))
		Sh = intersect(Sh,pass_Simes_m)
	}
	done = FALSE


	while(!done){
		thresh_old = thresh
		for(m in 1:M){
			# which groups, for the m-th grouping, 
			#	have any potential discoveries?
			Shm = sort(unique(groups[Sh,m]))

			# run BH on Simes[[m]], constraining to Shm
			Pvals_m = rep(1.01,G[m]); # >1 for groups not in Dm
			Pvals_m[Shm] = Simes[[m]][Shm]
			khatm = max(0,which(sort(Pvals_m)<=(1:G[m])/G[m]*alphas[m]))
			thresh[m] = khatm/G[m]*alphas[m]
			Sh = intersect(Sh,
				which(is.element(groups[,m],which(Pvals_m<=thresh[m]))))
		}
		if(all(thresh_old==thresh)){done = TRUE}
	}
	
	Sh_temp = Sh;
	Sh = rep(0,n); Sh[Sh_temp] = 1
	Sh

}
```

Next, we can specify at what levels we want to control overall and group-level FDR, respectively. To control overall FDR, the first column of "groups" places each p-value into its own group to control group-level FDR, the second column of "groups" assigns each p-value to its group (group 1, 2, 3, or 4). Finally, we can take a look at the discoveries. A "1" indicates that the p-value was selected (identified as a likely true signal)

First, let us run this `p-filter` procedure on our i.i.d. p-values. 

```{r}
# Inputs
fdr_overall = 0.2; fdr_gr = 0.3

# Levels of FDR control
alphas = c(fdr_overall, fdr_gr)

# Specify groups
groups = cbind(1:length(Pvec_iid), 
               c(1,2,3,4,1,2,3,4,1,2,3,4,1,2,3,4,1,2,3,4))

# Find discoveries
Discoveries1_iid = pfilter(Pvec_iid,alphas,groups)

# Print
Discoveries1_iid
```

We can see that the discoveries in the iid case are concentrated in the later time periods, but do accurately correspond with signals and nulls. An issue here is that the earlier signals are not identified. This is because the earlier e-values have not yet "exploded."

```{r}
# Inputs
fdr_overall = 0.2; fdr_gr = 0.3

# Levels of FDR control
alphas = c(fdr_overall, fdr_gr)

# Specify groups
groups = cbind(1:length(Pvec_dep), 
               c(1,2,3,4,1,2,3,4,1,2,3,4,1,2,3,4,1,2,3,4))

# Find discoveries
Discoveries1_dep = pfilter(Pvec_dep,alphas,groups)

# Print
Discoveries1_dep
```

In the `Pvec_dep` case, we see that we have no discoveries at the specified levels of `alpha`. As we saw before, this is because (i) the p-values are already very high and (ii) the additional layer of FDR control (group-specific and overall) takes its toll. Alternatively, we could adjust our `alpha` values. However, it is also important to note that our simulation sample is very small and therefore the sequential dependence relationship has little room to "unfold". In a real-world scenario, the sequential dependence relationship would likely become more pronounced over more time periods.  

All in all, this simulation is an example of how difficult it is to conduct valid inference under dependence. P-values come with a precise probabilistic meaning, but fail under dependence. E-values are heavily context-dependent and can be harmful in the wrong application, but allow for flexibility under dependence. Future research in the replicability and use of of e-values under dependence sounds very intriguing to us after conducting this project. 
