---
layout: apps
title: Survival Distributions in R
---
* TOC
{:toc}

### Overview

This page summarizes common parametric distributions in R, based on the R functions shown in the table below.

<table class="t1">
<caption> <b> Parametric survival distributions in R </b> </caption>
<thead>
<tr><th>Distribution</th><th>Density</th><th>CDF</th><th>Hazard</th><th>Cumulative hazard</th><th>Random sample</th></tr>
</thead>
<tbody>
<tr><th>Exponential</th><td>dexp</td><td>pexp</td><td>flexsurv::hexp</td><td>flexsurv::Hexp</td><td>rexp</td></tr>
<tr><th>Weibull (AFT) </th><td>dweibull</td><td>pweibull</td><td>flexsurv::hweibull</td><td>flexsurv::Hweibull</td><td>rweibull</td></tr>
<tr><th>Gamma</th><td>dgamma</td><td>pgamma</td><td>flexsurv::hgamma</td><td>flexsurv::Hgamma</td><td>rgamma</td></tr>
<tr><th>Lognormal </th><td>dlnorm</td><td>plnorm</td><td>flexsurv::hlnorm</td><td>flexsurv::Hlnorm</td><td>rlnorm</td></tr>
<tr><th>Gompertz </th><td>flexsurv::dgompertz</td><td>flexsurv::pgompertz</td><td>flexsurv::hgompertz</td><td>flexsurv::Hgompertz</td><td>flexsurv::rgompertz</td></tr>
<tr><th>Log-logistic </th><td>flexsurv::dllogis</td><td>flexsurv::pllogis</td><td>flexsurv::hllogis</td><td>flexsurv::Hllogis</td><td>flexsurv::rllogis</td></tr>
<tr><th>Generalized gamma (Prentice 1975) </th><td>flexsurv::dgengamma</td><td>flexsurv::pgengamma</td><td>flexsurv::hgengamma</td><td>flexsurv::Hgengamma</td><td>flexsurv::rgengamma</td></tr>
</tbody>
</table>

### General Survival Distributions
Survival function:
$$\begin{aligned}
S(t) = Pr(T > t) = \exp\left(-H(t)\right)
\end{aligned}$$

Hazard function:
$$\begin{aligned}
h(t) = f(t)/S(t)
\end{aligned}$$

Cumulative hazard function:
$$\begin{aligned}
H(t) = \int_0^t h(z)dz = -log S(t) 
\end{aligned}$$

### Exponential Distribution
Notation: 
$$\lambda > 0$$ (rate)

Density:
$$f(t) = \lambda e^{-\lambda t}$$

Survival:
$$S(t) = e^{-\lambda t}$$

Hazard:
$$h(t) = \lambda$$

Cumulative hazard:
$$h(t) = \lambda t$$

Mean: 
$$1/\lambda$$

Median:
$$\ln(2)/\lambda$$

### Weibull Distribution
Notation: 
$$\kappa > 0$$ (shape), $$\eta > 0$$ (scale), $$\Gamma(x)$$ = [gamma function](https://en.wikipedia.org/wiki/Gamma_function)

Density:
$$f(t) = \frac{\kappa}{\eta}\left(\frac{t}{\eta}\right)^{\kappa -1}e^{-(x/\eta)^\kappa}$$

Survival:
$$S(t) = e^{-(t/\eta)^\kappa}$$

Hazard:
$$h(t) = \frac{\kappa}{\eta}\left(\frac{t}{\eta}\right)^{\kappa -1}$$

Cumulative hazard:
$$H(t) = \left(\frac{t}{\eta}\right)^\kappa$$

Mean:
$$\eta \Gamma(1 + 1/\kappa)$$

Median:
$$\eta (\ln(2))^{1/\kappa}$$

Notes: The exponential distribution is a special case of the Weibull with $$\kappa = 1$$ and $$\lambda = 1/\eta$$

### Gamma Distribution
Notation: 
$$a > 0$$ (shape), $$b > 0$$ (rate), $$\gamma(k, x) = \int_0^x z^{k-1}e^{-z}dz$$ is the lower [incomplete gamma function](https://en.wikipedia.org/wiki/Incomplete_gamma_function)

Density:
$$f(t) = \frac{b^a}{\Gamma(a)}t^{a -1}e^{-bt}$$

Survival:
$$S(t) = 1 - \frac{\gamma(a, bt)}{\Gamma(a)}$$

Mean: $$a/b$$

Notes: When $$a=1$$, the gamma distribution simplifies to the exponential distribution with rate parameter $$b$$. 

### Lognormal Distribution
Notation: 
$$\mu \in (-\infty, \infty)$$ (mean), $$\sigma > 0$$ (standard deviation), $$\Phi(t)$$ is the [CDF of the standard normal distribution](https://en.wikipedia.org/wiki/Normal_distribution#Cumulative_distribution_function) 

Density:
$$f(t) = \frac{1}{t\sigma\sqrt{2\pi}}e^{-\frac{(\ln t - \mu)^2}{2\sigma^2}}$$

Survival: 
$$1- \Phi\left(\frac{\ln t - \mu}{\sigma}\right)$$

Mean:
$$e^{\mu + \sigma^2/2}$$

Median:
$$e^\mu$$

### Gompertz Distribution
Notation: $$a \in (-\infty, \infty)$$ (shape), $$b > 0$$ (rate)

Density: 
$$f(t) = be^{at}\exp\left[-\frac{b}{a}(e^{at}-1)\right]$$

Survival:
$$S(t) = \exp\left[-\frac{b}{a}(e^{at}-1)\right]$$

Hazard:
$$h(t) = be^{at}$$

Cumulative Hazard:
$$H(t) = \frac{b}{a}\left(e^{at}-1\right)$$

Median:
$$\frac{1}{b}\ln\left[-(1/a)\ln(1/2) + 1\right]$$

Notes:
When $$a=0$$ the Gompertz distribution is equivalent to the exponential with constant hazard and rate $$b$$.

### Log-logistic Distribution
Notation: 
$$\kappa>0$$ (shape), $$\eta > 0$$ (scale)

Density: 
$$\begin{aligned}
f(t) =\frac{(\kappa/\eta)(t/\eta)^{\kappa-1}}{\left(1 + (t/\eta)^\kappa\right)^2}
\end{aligned}$$

Survival:
$$\begin{aligned}
S(t) = \frac{1}{(1+(t/\eta)^\kappa)}
\end{aligned}$$

Hazard:
$$\begin{aligned}
h(t) =\frac{(\kappa/\eta)(t/\eta)^{\kappa-1}}{\left(1 + (t/\eta)^\kappa\right)}
\end{aligned}$$

Median:
$$\eta$$

Mean: 
If $$\kappa > 1$$,
$$\begin{aligned}
\frac{\eta (\pi/\kappa)}{\sin(\pi/\kappa)};
\end{aligned}$$
else undefined

### Generalized Gamma Distribution
Notation: 
$$\mu \in (-\infty, \infty)$$ (location parameter), $$\sigma > 0$$ (scale parameter), $$Q \in (-\infty, \infty)$$ (shape parameter), $w = (log(t) - \mu)/\sigma$, and $u = Q^{-2}e^{Qw}$ 

Density: 
$$\begin{aligned}
f(t) = \frac{|Q|(Q^{-2})^{Q^{-2}}}{\sigma t \Gamma(Q^{-2})} \exp\left[Q^{-2}\left(Qw-e^{Qw}\right)\right]
\end{aligned}$$  

Survival:
$$\begin{aligned}
S(t) =
\begin{cases} 1 - \frac{\gamma(Q^{-2}, u)}{\Gamma(Q^{-2})} \text{ if } Q \neq 0  \\
       1 - \Phi(w) \text{ if } Q = 0 
\end{cases}
\end{aligned}$$  

Notes: Simplifies to lognormal when $$Q=0$$, Weibull when $$Q=1$$, exponential when $$Q=\sigma=1$$, and gamma when $$Q = \sigma$$