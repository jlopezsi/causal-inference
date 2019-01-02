Identifying average causal effects with latent confounding variable proxies
================
Iyar Lin
17 December, 2018

-   [Intro](#intro)
-   [Both proxies are conditionally independant given U](#both-proxies-are-conditionally-independant-given-u)

Intro
=====

When using the Backdoor criteria to find the adjustment set required to estimate the average causal effect of setting a variable ![X=x](https://latex.codecogs.com/png.latex?X%3Dx "X=x") on some other variable ![Y](https://latex.codecogs.com/png.latex?Y "Y") we may discover that we need to condition on an unmeasured (latent) variable. Often times we'll have proxy variables (i.e. noisy measurements of the latent variable) at hand. In this script I demonstrate how to utilize the latent variable proxies in the structural equation settings to estimate the ATE as presented at [Kuroki and Pearl, 2014](https://pdfs.semanticscholar.org/19da/be9fdbe82d74a584f5ece882f5825e4effdb.pdf)

Both proxies are conditionally independant given U
==================================================

We consider the following graphical model:

``` r
g <- dagitty(
  "dag {
  U [latent, pos=\"0,1\"]
  W [pos=\"2.000,1.000\"]
  X [exposure,pos=\"1.000,0.000\"]
  Y [outcome,pos=\"3.000,0.000\"]
  Z [pos=\"-1.000,0.000\"]
  U -> W [beta = 0.3]
  U -> X [beta = -0.5]
  U -> Y [beta = 0.9]
  U -> Z [beta = -0.2]
  X -> Y [beta = 0.3]
  }"
)

ggdag(tidy_dagitty(g)) + theme_dag_blank()
```

![](ATE_with_latent_variable_proxies_files/figure-markdown_github/plot%20model%20graph-1.png)

Here, ![Z](https://latex.codecogs.com/png.latex?Z "Z") and ![W](https://latex.codecogs.com/png.latex?W "W") serve as conditionally independent proxies of ![U](https://latex.codecogs.com/png.latex?U "U") given ![U](https://latex.codecogs.com/png.latex?U "U"). The average causal effect (ACE) in this toy model is 0.3.

One would think conditioning on ![\\{Z,W\\}](https://latex.codecogs.com/png.latex?%5C%7BZ%2CW%5C%7D "\{Z,W\}") would suffice to "account for the confounder" but unfortunately that is not the case. Below I simulate 100K observations from the model above and show the resulting linear regression coefficients when fitting the model ![Y = \\beta\_1X + \\beta\_2Z + \\beta\_3W + \\epsilon](https://latex.codecogs.com/png.latex?Y%20%3D%20%5Cbeta_1X%20%2B%20%5Cbeta_2Z%20%2B%20%5Cbeta_3W%20%2B%20%5Cepsilon "Y = \beta_1X + \beta_2Z + \beta_3W + \epsilon"):

``` r
sim_data <- simulateSEM(g, N = 100000)
naive_model <- lm(Y ~ X + Z + W, data = sim_data)

pandoc.table(coef(naive_model))
```

<table style="width:61%;">
<colgroup>
<col width="19%" />
<col width="13%" />
<col width="13%" />
<col width="13%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">X</th>
<th align="center">Z</th>
<th align="center">W</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.000882</td>
<td align="center">-0.1076</td>
<td align="center">-0.1253</td>
<td align="center">0.1998</td>
</tr>
</tbody>
</table>

Below I simulate the dataset 1000 times for a variety of sample sizes, demonstrate how to estimate the ACE correctly and show the estimator mean and standard deviation as a function of the sample size:

``` r
ACE <- data.frame(N = c(100, 1000, 10000, 100000), 
                  ACE_mean = NA, 
                  ACE_SD = NA)

for(i in 1:nrow(ACE)){
  sample_size <- ACE$N[i]
  ACE_vec = replicate(1000, expr = {
    sim_data <- simulateSEM(g, N = sample_size)
    Sigma <- cov(sim_data)
    sigma_x_y <- Sigma[2, 3]
    sigma_w_z <- Sigma[1, 4]
    sigma_x_w <- Sigma[2, 1]
    sigma_y_z <- Sigma[3, 4]
    sigma_x_x <- Sigma[2, 2]
    sigma_x_z <- Sigma[2, 4]
    
    ACE <- (sigma_x_y*sigma_w_z - sigma_x_w*sigma_y_z)/
      (sigma_x_x*sigma_w_z - sigma_x_w*sigma_x_z)
    ACE})
  ACE$ACE_mean[i] <- mean(ACE_vec)
  ACE$ACE_SD[i] <- sd(ACE_vec)
}

pandoc.table(ACE)
```

    ## 
    ## -----------------------------
    ##    N      ACE_mean   ACE_SD  
    ## -------- ---------- ---------
    ##   100     -0.08779    13.81  
    ## 
    ##   1000     0.5755     8.523  
    ## 
    ##  10000     0.3231    0.1309  
    ## 
    ##  100000    0.3006    0.03243 
    ## -----------------------------

It's quite astounding to see one needs at least 10K observations to get an "OK" estimator.
