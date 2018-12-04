Common analysis mistakes (Or: why we've been doing it wrong all along
================
Iyar Lin
04 December, 2018

-   [Intro](#intro)
-   [Common analysis strategies](#common-analysis-strategies)
    -   [Looking on bi-variate relationship](#looking-on-bi-variate-relationship)
    -   ["Controlling" for as many variables as possible](#controlling-for-as-many-variables-as-possible)
-   [Using DAGs is the solution](#using-dags-is-the-solution)

Intro
=====

When trying to estimate the impact of some decisions several methods are usually utilized, all of which are flawed and produce biased estimates.

Common analysis strategies
==========================

Looking on bi-variate relationship
----------------------------------

Let's assume we measure the following variables:

-   revenue: Overall sales revenue for a given year
-   RD: How much we spent on Reaserach and Development in the previous year
-   MS: Relative share of the market for a given year

Let's also assume that the graph below depicts the causal relationships between the above variables:

``` r
g <- dagitty("dag {
revenue [outcome]
RD [exposure]
RD -> revenue [beta = 0.15]
MS -> RD [beta = -0.4]
MS -> revenue [beta = 0.9]
}")

plot(graphLayout(g))
```

![](common_analysis_mistakes_files/figure-markdown_github/define%20model1%20and%20plot-1.png)

``` r
N <- 100000
```

We'll also assume that the relationships are linear with the following coefficients:

![\\beta\_{RD\\rightarrow revenue} = 0.15](https://latex.codecogs.com/png.latex?%5Cbeta_%7BRD%5Crightarrow%20revenue%7D%20%3D%200.15 "\beta_{RD\rightarrow revenue} = 0.15") ![\\beta\_{MS\\rightarrow RD} = -0.4](https://latex.codecogs.com/png.latex?%5Cbeta_%7BMS%5Crightarrow%20RD%7D%20%3D%20-0.4 "\beta_{MS\rightarrow RD} = -0.4") ![\\beta\_{MS\\rightarrow revenue} = 0.9](https://latex.codecogs.com/png.latex?%5Cbeta_%7BMS%5Crightarrow%20revenue%7D%20%3D%200.9 "\beta_{MS\rightarrow revenue} = 0.9")

The effect of increasing RD on revenue ![\\delta \\text{revenue} / \\delta \\text{RD}](https://latex.codecogs.com/png.latex?%5Cdelta%20%5Ctext%7Brevenue%7D%20%2F%20%5Cdelta%20%5Ctext%7BRD%7D "\delta \text{revenue} / \delta \text{RD}") is:

![\\delta \\text{revenue} / \\delta \\text{RD} = \\beta\_{RD\\rightarrow revenue} = 0.15](https://latex.codecogs.com/png.latex?%5Cdelta%20%5Ctext%7Brevenue%7D%20%2F%20%5Cdelta%20%5Ctext%7BRD%7D%20%3D%20%5Cbeta_%7BRD%5Crightarrow%20revenue%7D%20%3D%200.15 "\delta \text{revenue} / \delta \text{RD} = \beta_{RD\rightarrow revenue} = 0.15")

Below I simulate a dataset from the above graph with 100000 observations

``` r
sim_data <- simulateSEM(g, N = N, standardized = T)
```

If we were to plot the relation between RD and revenue we would get:

``` r
sim_data %>%
  ggplot(aes(RD, revenue)) +
  geom_point(alpha = 0.05) + geom_smooth(method = "lm")
```

![](common_analysis_mistakes_files/figure-markdown_github/plot%20bi-variate%20relation-1.png)

Fitting the linear model:

![revenue = \\beta\_1RD + \\epsilon](https://latex.codecogs.com/png.latex?revenue%20%3D%20%5Cbeta_1RD%20%2B%20%5Cepsilon "revenue = \beta_1RD + \epsilon")

``` r
model <- lm(revenue ~ RD, data = sim_data)
```

We get the following coefficients:

``` r
pandoc.table(coefficients(model))
```

<table style="width:32%;">
<colgroup>
<col width="19%" />
<col width="12%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">RD</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">-0.001196</td>
<td align="center">-0.2135</td>
</tr>
</tbody>
</table>

We get the opposite relationship than what we'd expect!

"Controlling" for as many variables as possible
-----------------------------------------------

Notionally we know that may arise due to MS being a confounding variable. Often times we fix confounding variables by "controlling" for as many things we can.

Let's assume we also measure:

-   CS: Customer satisifaction

We'll assume the graph below depicts the causal relationships between the all measured variables:

``` r
g <- dagitty("dag {
revenue [outcome]
RD [exposure]
RD -> CS [beta = 0.3]
CS -> revenue [beta = 0.5]
MS -> RD [beta = -0.4]
MS -> revenue [beta = 0.9]
}")

plot(graphLayout(g))
```

![](common_analysis_mistakes_files/figure-markdown_github/define%20model%202-1.png)

We'll also assume that the relationships are linear with the following coefficients:

![\\beta\_{RD\\rightarrow CS} = 0.3](https://latex.codecogs.com/png.latex?%5Cbeta_%7BRD%5Crightarrow%20CS%7D%20%3D%200.3 "\beta_{RD\rightarrow CS} = 0.3") ![\\beta\_{CS\\rightarrow revenue} = 0.5](https://latex.codecogs.com/png.latex?%5Cbeta_%7BCS%5Crightarrow%20revenue%7D%20%3D%200.5 "\beta_{CS\rightarrow revenue} = 0.5") ![\\beta\_{MS\\rightarrow RD} = -0.4](https://latex.codecogs.com/png.latex?%5Cbeta_%7BMS%5Crightarrow%20RD%7D%20%3D%20-0.4 "\beta_{MS\rightarrow RD} = -0.4") ![\\beta\_{MS\\rightarrow revenue} = 0.9](https://latex.codecogs.com/png.latex?%5Cbeta_%7BMS%5Crightarrow%20revenue%7D%20%3D%200.9 "\beta_{MS\rightarrow revenue} = 0.9")

The effect of increasing RD on revenue ![\\delta \\text{revenue} / \\delta \\text{RD}](https://latex.codecogs.com/png.latex?%5Cdelta%20%5Ctext%7Brevenue%7D%20%2F%20%5Cdelta%20%5Ctext%7BRD%7D "\delta \text{revenue} / \delta \text{RD}") in this case is:

![\\delta \\text{revenue} / \\delta \\text{RD} = \\beta\_{RD\\rightarrow CS} \\cdot \\beta\_{CS\\rightarrow revenue} = 0.15](https://latex.codecogs.com/png.latex?%5Cdelta%20%5Ctext%7Brevenue%7D%20%2F%20%5Cdelta%20%5Ctext%7BRD%7D%20%3D%20%5Cbeta_%7BRD%5Crightarrow%20CS%7D%20%5Ccdot%20%5Cbeta_%7BCS%5Crightarrow%20revenue%7D%20%3D%200.15 "\delta \text{revenue} / \delta \text{RD} = \beta_{RD\rightarrow CS} \cdot \beta_{CS\rightarrow revenue} = 0.15")

If we fit a regression using all variables available:

![revenue = \\beta\_1RD + \\beta\_2MS + \\beta\_3CS + \\epsilon](https://latex.codecogs.com/png.latex?revenue%20%3D%20%5Cbeta_1RD%20%2B%20%5Cbeta_2MS%20%2B%20%5Cbeta_3CS%20%2B%20%5Cepsilon "revenue = \beta_1RD + \beta_2MS + \beta_3CS + \epsilon")

We'll obtain the following coefficient estimates:

``` r
sim_data <- simulateSEM(g, N = N, standardized = T)
model <- lm(revenue ~ ., data = sim_data)
pandoc.table(coefficients(model))
```

<table style="width:60%;">
<colgroup>
<col width="19%" />
<col width="12%" />
<col width="12%" />
<col width="15%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">CS</th>
<th align="center">MS</th>
<th align="center">RD</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">-0.00002264</td>
<td align="center">0.5012</td>
<td align="center">0.8999</td>
<td align="center">-0.001053</td>
</tr>
</tbody>
</table>

We can see that the coefficient of RD is about 0. This happens because conditioning on CS blocks the path from RD to revenue.

If we instead fit the model:

![revenue = \\beta\_1RD + \\beta\_2MS + \\epsilon](https://latex.codecogs.com/png.latex?revenue%20%3D%20%5Cbeta_1RD%20%2B%20%5Cbeta_2MS%20%2B%20%5Cepsilon "revenue = \beta_1RD + \beta_2MS + \epsilon")

``` r
model <- lm(revenue ~ RD + MS, data = sim_data)
```

We get the following coefficients:

``` r
pandoc.table(coefficients(model))
```

<table style="width:44%;">
<colgroup>
<col width="19%" />
<col width="12%" />
<col width="12%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">RD</th>
<th align="center">MS</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">-0.0003675</td>
<td align="center">0.1475</td>
<td align="center">0.9026</td>
</tr>
</tbody>
</table>

Obtaining the true RD effect.

Using DAGs is the solution
==========================

Next let's assume we also measure:

-   population: How many potential customers in the market
-   demo: Demographics (rich population, dense population etc)

We'll assume the graph below depicts the causal relationships between the all measured variables:

``` r
g <- dagitty("dag {
revenue [outcome]
RD [exposure]
RD -> CS [beta = 0.3]
CS -> revenue [beta = 0.5]
MS -> RD [beta = -0.4]
MS -> revenue [beta = 0.9]
population -> MS [beta = -0.5]
population -> RD [beta = 0.7]
demo -> MS [beta = 0.3]
demo -> revenue [beta = 0.2]
}")

plot(graphLayout(g))
```

![](common_analysis_mistakes_files/figure-markdown_github/define%20model%203-1.png)

This time around neither conditioning on MS:

``` r
sim_data <- simulateSEM(g, N = N, standardized = T)
model <- lm(revenue ~ RD + MS, data = sim_data)
pandoc.table(coefficients(model))
```

<table style="width:44%;">
<colgroup>
<col width="19%" />
<col width="12%" />
<col width="12%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">RD</th>
<th align="center">MS</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.001441</td>
<td align="center">0.1959</td>
<td align="center">0.9946</td>
</tr>
</tbody>
</table>

neither using all variables will help us:

``` r
sim_data <- simulateSEM(g, N = N, standardized = T)
model <- lm(revenue ~ ., data = sim_data)
pandoc.table(coefficients(model))
```

<table style="width:92%;">
<colgroup>
<col width="19%" />
<col width="12%" />
<col width="12%" />
<col width="18%" />
<col width="12%" />
<col width="16%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">CS</th>
<th align="center">MS</th>
<th align="center">RD</th>
<th align="center">demo</th>
<th align="center">population</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.0001593</td>
<td align="center">0.4999</td>
<td align="center">0.9001</td>
<td align="center">-0.0002181</td>
<td align="center">0.1999</td>
<td align="center">0.0003666</td>
</tr>
</tbody>
</table>

We'll have to use graphical model math (backdoor criteria in this case) to discover that we need to condition on:

``` r
print(adjustmentSets(g))
```

    ## { MS, demo }
    ## { MS, population }

So running the model:

![revenue = \\beta\_1RD + \\beta\_2MS + \\beta\_3 demo + \\epsilon](https://latex.codecogs.com/png.latex?revenue%20%3D%20%5Cbeta_1RD%20%2B%20%5Cbeta_2MS%20%2B%20%5Cbeta_3%20demo%20%2B%20%5Cepsilon "revenue = \beta_1RD + \beta_2MS + \beta_3 demo + \epsilon")

``` r
sim_data <- simulateSEM(g, N = N, standardized = T)
model <- lm(revenue ~ RD + MS + demo, data = sim_data)
```

We get the following coefficients:

``` r
pandoc.table(coefficients(model))
```

<table style="width:57%;">
<colgroup>
<col width="19%" />
<col width="12%" />
<col width="12%" />
<col width="12%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">RD</th>
<th align="center">MS</th>
<th align="center">demo</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.0003796</td>
<td align="center">0.1493</td>
<td align="center">0.8988</td>
<td align="center">0.1998</td>
</tr>
</tbody>
</table>

Obtaining the true RD effect.
