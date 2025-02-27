---
layout: post
title: Markov Cohort Models
---
* TOC
{:toc}

### Overview
This page shows how to employ a Markov cohort model for cost-effectiveness analysis using R. The analysis replicates a [paper](https://www.ncbi.nlm.nih.gov/pubmed/10169387) that is commonly used for [teaching purposes](https://www.amazon.com/Decision-Modelling-Economic-Evaluation-Handbooks/dp/0198526628) and compares combination HIV therapy using zidovudine and lamivudine to zidovudine monotherapy.

### Setting Up a Markov Model
In a Markov model patients move from one mutually exclusive disease state to another over a series of discrete time periods. A fully specified model consists of 1) disease states, 2) the probability of transitioning from one stage to the next, 3) treatment costs in each stage, and 4) quality of life weights by stage. 

#### States

In our example, HIV patients can be in one of the following 4 states at a given point in time:

* State A: 200 < cd4 < 500 cells/mm$$^3$$,
* State B: cd4 < 200 cells/mm$$^3$$,
* State C: Aids,
* State D: Death.

#### Transition Matrix

At any given points in time, the row vector $$z_{kt} = [z_{1kt}\; z_{2kt}\; z_{3kt}\; z_{4kt}]$$ represents the total number of HIV patients in each of the 4 states using treatment $$k$$ at time $$t$$. This vector, $$z_{kt}$$, changes over time according to a transition matrix, $$P_{kt}$$,

$$
\begin{aligned}
\underbrace{z_{kt}}_{1 \times 4} &= \underbrace{z_{k,t-1}}_{1 \times 4} \underbrace{P_{kt}}_{4 \times 4},
\end{aligned}
$$

where

$$
\begin{aligned}
P_{kt} &=
\begin{bmatrix}
p_{11} & p_{12} & p_{13} & p_{14}\\
p_{21} & p_{22} & p_{23} & p_{24}\\
p_{31} & p_{32} & p_{33} & p_{34}\\
p_{41} & p_{42} & p_{43} & p_{44}
\end{bmatrix}.
\end{aligned}
$$

The probability of transitioning from state $$i$$ to state $$j$$ at time $$t$$ is denoted by $$p_{ij}$$. Under monotherapy (treatment $$0$$), the estimated transition matrix is constant and given by,

$$
\begin{aligned}
P_0 &=
\begin{bmatrix}
0.721 & 0.202 & 0.067 & 0.010\\
0 & 0.581 & 0.407 & 0.012\\
0 & 0 & 0.750 & 0.250\\
0 & 0 & 0 & 1
\end{bmatrix}.
\end{aligned}
$$

Transition probabilities for combination therapy (treatment $$1$$) are based on an estimated relative risk of disease progression of $$0.509$$ from a meta-analysis of 4 comparative trials. The relative risk is assumed to reduce the probability of transitioning to a worse state (i.e. $$p_{12}, p_{13}, p_{14}, p_{23}, p_{24}, p_{34}$$) by a factor of .509.

We set up the transition probabilities in R.


```r
P.0 <- matrix(c(.721, .202, .067, .01, 
              0, .581, .407, .012,
              0, 0, .75, .25,
              0, 0, 0, 1),
            ncol = 4, nrow = 4, byrow = TRUE)
P.1 <- matrix(c(.858, .103, .034, .005,
             0, .787, .207, .006,
             0, 0, .873, .127,
             0, 0, 0, 1),
             ncol = 4, nrow = 4, byrow = TRUE)
```

#### Costs

Treatment costs are due to a) direct medical and community expenses and b) the prices of each drug. The drug costs of zidovudine and lamivudine are &pound;2278 and &pound;2086 respectively. Medical and community expenses in each state are as follows:

* State A: &pound;2256
* State B: &pound;3052
* State C: &pound;9007
* State D: &pound;0

In R, we have:


```r
c.zidovudine <- 2278
c.lamivudine <- 2086.50
c.0 <- c(2756 + c.zidovudine, 3052 + c.zidovudine, 9007 + c.zidovudine, 0)
c.1 <- c(c.0[1:3] + c.lamivudine, 0)
```


#### Quality of Life Weights

Treatment effects are typically measured by quality-adjusted life-years (QALYs). However, we follow the original paper which measured effectiveness with (unadjusted) life-years. In mathematical terms, this means that the 4 states are weighted with the row vector $$[1\; 1\; 1\; 0]$$. 

```r
qolw <- c(1, 1, 1, 0)
```

### Cohort Simulation in R
A cohort simulation uses the Makov model to measure the experiences of a hypothetical cohort, say 1000 patients, over a set period of time (i.e. 20 years), under each treatment option. 

We load an R function that simulates the costs and effects of an intervention given a transition matrix ```P```; an initial row vector ```z0``` containing the number of patients in each state at time $$0$$; the number of cycles ```ncycles```, the costs in each state; the quality of life weights; and a discount factor for future costs. (Note that, like the original paper, the discount factor is only applied to costs, although a discount could be applied to life-years as well.) 


```r
source("_rmd-posts/markov.R")
```

At each cycle, the function calculates the number of patients in each state ```z[t, ]```; total costs per patient, ```c[t]```; and life-years ```e[t]```. The algorithm is very simple but appears more complicated because it allows for both time constant and time-varying transition matrices and costs. The time-varying transition matrices are necessary because lamivudine is only assumed to be given for the first two-years of treatment. 

Using the function we simulate outcomes under both treatments.

```r
ncycles <- 20
z0 <- matrix(c(1000, 0, 0, 0), nrow = 1)
sim0 <- MarkovCohort(P = P.0,  z0 = c(1000, 0, 0, 0), ncycles = ncycles,
                    costs = c.0, qolw = qolw, 
                    discount = 0.06)
sim1 <- MarkovCohort(P = c(replicate(2, P.1, simplify = FALSE), 
                              replicate(ncycles - 2, P.0, simplify = FALSE)),
                     z0 = c(1000, 0, 0, 0), ncycles = ncycles,
                     costs = c(replicate(2, c.1, simplify = FALSE),
                               replicate(ncycles - 2, c.0, simplify = FALSE)),
                     qolw = qolw, discount = 0.06)
```

### Decision Analysis
Armed with the simulation results, we can plot survival curves by treatment.

```r
library("ggplot2")
theme_set(theme_bw())
surv.df <- data.frame(surv = c(sim0$e, sim1$e),
                      cylce = rep(seq(1, ncycles), 2),
                       lab = rep(c("Monotherapy", "Combination therapy"), 
                       each = ncycles))
ggplot2::ggplot(dat = surv.df, aes(x = cylce, y = surv, col = lab)) + geom_line() + 
  xlab("Cycle") + ylab("Fraction surviving") +
  theme(legend.title=element_blank()) + 
  theme(legend.position = "bottom")
```

<img src="/figs/survival-1.png" title="plot of chunk survival" alt="plot of chunk survival" style="display: block; margin: auto;" />
Due to the estimated relative risk of disease progression, the survival curve for combination therapy lies above the curve for monotherapy. That said, costs are higher for combination therapy because of 1) additional drug costs for lamivudine and 2) patients being treated longer due to increased survival.

To summarize the cost-effectiveness of the intervention we can use the incremental cost-effectiveness ratio (ICER), or,

$$
\begin{aligned}
\frac{c_1 - c_0}{e_1 - e_0},
\end{aligned}
$$

where $$c_k$$ and $$e_k$$ refer to the costs and effects under treatment $$k$$ respectively. In R,

```r
icer <- (sum(sim1$c) - sum(sim0$c))/(sum(sim1$e) - sum(sim0$e))
print(icer)
```

```
## [1] 6276.083
```
which suggest that a decision-maker should only approve combination therapy if he or she is willing to pay over ``6276`` pounds for an additional life-year gained.
