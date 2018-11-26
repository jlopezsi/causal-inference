DAG use case
================
Iyar Lin
26 November, 2018

-   [Intro](#intro)
    -   [Confounding variable example](#confounding-variable-example)
    -   [Blocking variable](#blocking-variable)
    -   [Collider nodes (advanced)](#collider-nodes-advanced)

Intro
=====

This script demonstrates the need and use case for DAGs.

Confounding variable example
----------------------------

Let's assume we measure the following variables:

-   churn: Did a subscriber churn in the first 6 months? (Constructing it as a continuous variable for demonstration purposes)
-   cong: Network congestion (e.g. fill rate)
-   comp: Competition levels (e.g. distance to fiber)

Let's also assume that the graph below depicts the causal relatioships between the above variables:

``` r
g <- dagitty("dag {
churn [outcome]
cong [exposure]
cong -> churn [beta = 0.15]
comp -> cong [beta = -0.4]
comp -> churn [beta = 0.9]
}")

plot(graphLayout(g))
```

![](dag_use_case_files/figure-markdown_github/define%20model1%20and%20plot-1.png)

``` r
N <- 100000
```

We'll also assume that the relationships are linear with the following coefficients:

![\\beta\_{cong\\rightarrow churn} = 0.15](https://latex.codecogs.com/png.latex?%5Cbeta_%7Bcong%5Crightarrow%20churn%7D%20%3D%200.15 "\beta_{cong\rightarrow churn} = 0.15") ![\\beta\_{comp\\rightarrow cong} = -0.4](https://latex.codecogs.com/png.latex?%5Cbeta_%7Bcomp%5Crightarrow%20cong%7D%20%3D%20-0.4 "\beta_{comp\rightarrow cong} = -0.4") ![\\beta\_{comp\\rightarrow churn} = 0.9](https://latex.codecogs.com/png.latex?%5Cbeta_%7Bcomp%5Crightarrow%20churn%7D%20%3D%200.9 "\beta_{comp\rightarrow churn} = 0.9")

The effect of increasing congestion on churn ![\\delta \\text{churn} / \\delta \\text{cong}](https://latex.codecogs.com/png.latex?%5Cdelta%20%5Ctext%7Bchurn%7D%20%2F%20%5Cdelta%20%5Ctext%7Bcong%7D "\delta \text{churn} / \delta \text{cong}") is:

![\\delta \\text{churn} / \\delta \\text{cong} = \\beta\_{cong\\rightarrow churn} = 0.15](https://latex.codecogs.com/png.latex?%5Cdelta%20%5Ctext%7Bchurn%7D%20%2F%20%5Cdelta%20%5Ctext%7Bcong%7D%20%3D%20%5Cbeta_%7Bcong%5Crightarrow%20churn%7D%20%3D%200.15 "\delta \text{churn} / \delta \text{cong} = \beta_{cong\rightarrow churn} = 0.15")

Below I simulate a dataset from the above graph with 100000 observations

``` r
sim_data <- simulateSEM(g, N = N, standardized = T)
```

If we were to plot the relation between congestion and churn we would get:

``` r
sim_data %>%
  ggplot(aes(cong, churn)) +
  geom_point(alpha = 0.05) + geom_smooth(method = "lm")
```

![](dag_use_case_files/figure-markdown_github/plot%20bi-variate%20relation-1.png)

Fitting the linear model:

![churn = \\beta\_1cong + \\epsilon](https://latex.codecogs.com/png.latex?churn%20%3D%20%5Cbeta_1cong%20%2B%20%5Cepsilon "churn = \beta_1cong + \epsilon")

``` r
model <- lm(churn ~ cong, data = sim_data)
```

We get the following coefficients:

``` r
pandoc.table(coefficients(model))
```

<table style="width:29%;">
<colgroup>
<col width="19%" />
<col width="9%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">cong</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.002297</td>
<td align="center">-0.21</td>
</tr>
</tbody>
</table>

We get the opposite relationship than what we'd expect! Notionaly we know that may arise due to competition being a confounding variable. We usually fix confounding variables by "controlling" for as many things we can.

If we fit the model:

![churn = \\beta\_1cong + \\beta\_2comp + \\epsilon](https://latex.codecogs.com/png.latex?churn%20%3D%20%5Cbeta_1cong%20%2B%20%5Cbeta_2comp%20%2B%20%5Cepsilon "churn = \beta_1cong + \beta_2comp + \epsilon")

``` r
model <- lm(churn ~ cong + comp, data = sim_data)
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
<th align="center">cong</th>
<th align="center">comp</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.0003722</td>
<td align="center">0.1518</td>
<td align="center">0.9011</td>
</tr>
</tbody>
</table>

Obtaining the true congestion effect.

Blocking variable
-----------------

Next let's assume we also measure:

-   CX: Customer experience (e.g. buffering events/calls to care)

We'll assume the graph below depicts the causal relatioships between the all measured variables:

``` r
g <- dagitty("dag {
churn [outcome]
cong [exposure]
cong -> CX [beta = -0.3]
CX -> churn [beta = -0.5]
comp -> cong [beta = -0.4]
comp -> churn [beta = 0.9]
}")

plot(graphLayout(g))
```

![](dag_use_case_files/figure-markdown_github/define%20model%202-1.png)

We'll also assume that the relationships are linear with the following coefficients:

![\\beta\_{cong\\rightarrow CX} = -0.3](https://latex.codecogs.com/png.latex?%5Cbeta_%7Bcong%5Crightarrow%20CX%7D%20%3D%20-0.3 "\beta_{cong\rightarrow CX} = -0.3") ![\\beta\_{CX\\rightarrow churn} = -0.5](https://latex.codecogs.com/png.latex?%5Cbeta_%7BCX%5Crightarrow%20churn%7D%20%3D%20-0.5 "\beta_{CX\rightarrow churn} = -0.5") ![\\beta\_{comp\\rightarrow cong} = -0.4](https://latex.codecogs.com/png.latex?%5Cbeta_%7Bcomp%5Crightarrow%20cong%7D%20%3D%20-0.4 "\beta_{comp\rightarrow cong} = -0.4") ![\\beta\_{comp\\rightarrow churn} = 0.9](https://latex.codecogs.com/png.latex?%5Cbeta_%7Bcomp%5Crightarrow%20churn%7D%20%3D%200.9 "\beta_{comp\rightarrow churn} = 0.9")

The effect of increasing congestion on churn ![\\delta \\text{churn} / \\delta \\text{cong}](https://latex.codecogs.com/png.latex?%5Cdelta%20%5Ctext%7Bchurn%7D%20%2F%20%5Cdelta%20%5Ctext%7Bcong%7D "\delta \text{churn} / \delta \text{cong}") in this case is:

![\\delta \\text{churn} / \\delta \\text{cong} = \\beta\_{cong\\rightarrow CX} \\cdot \\beta\_{CX\\rightarrow churn} = 0.15](https://latex.codecogs.com/png.latex?%5Cdelta%20%5Ctext%7Bchurn%7D%20%2F%20%5Cdelta%20%5Ctext%7Bcong%7D%20%3D%20%5Cbeta_%7Bcong%5Crightarrow%20CX%7D%20%5Ccdot%20%5Cbeta_%7BCX%5Crightarrow%20churn%7D%20%3D%200.15 "\delta \text{churn} / \delta \text{cong} = \beta_{cong\rightarrow CX} \cdot \beta_{CX\rightarrow churn} = 0.15")

This time around if we use all the measures variables in our regression:

![churn = \\beta\_1cong + \\beta\_2comp + \\beta\_3CX + \\epsilon](https://latex.codecogs.com/png.latex?churn%20%3D%20%5Cbeta_1cong%20%2B%20%5Cbeta_2comp%20%2B%20%5Cbeta_3CX%20%2B%20%5Cepsilon "churn = \beta_1cong + \beta_2comp + \beta_3CX + \epsilon")

``` r
sim_data <- simulateSEM(g, N = N, standardized = T)
model <- lm(churn ~ ., data = sim_data)
pandoc.table(coefficients(model))
```

<table style="width:60%;">
<colgroup>
<col width="19%" />
<col width="13%" />
<col width="12%" />
<col width="13%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">CX</th>
<th align="center">comp</th>
<th align="center">cong</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.00003214</td>
<td align="center">-0.4998</td>
<td align="center">0.9014</td>
<td align="center">0.001201</td>
</tr>
</tbody>
</table>

We can see that the coefficient of congestion is about 0. This happens because conditioning CX blocks the path from cong to churn.

If we instead fit the model:

![churn = \\beta\_1cong + \\beta\_2comp + \\epsilon](https://latex.codecogs.com/png.latex?churn%20%3D%20%5Cbeta_1cong%20%2B%20%5Cbeta_2comp%20%2B%20%5Cepsilon "churn = \beta_1cong + \beta_2comp + \epsilon")

``` r
model <- lm(churn ~ cong + comp, data = sim_data)
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
<th align="center">cong</th>
<th align="center">comp</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">-0.00006752</td>
<td align="center">0.1516</td>
<td align="center">0.9004</td>
</tr>
</tbody>
</table>

Obtaining the true congestion effect.

Collider nodes (advanced)
-------------------------

Next let's assume we also measure:

-   population: How many potential subscribers in beam footprint
-   demo: Demographics (rich population, dense population etc)

We'll assume the graph below depicts the causal relatioships between the all measured variables:

``` r
g <- dagitty("dag {
churn [outcome]
cong [exposure]
cong -> CX [beta = -0.3]
CX -> churn [beta = -0.5]
comp -> cong [beta = -0.4]
comp -> churn [beta = 0.9]
population -> comp [beta = 0.2]
population -> cong [beta = 0.3]
demo -> comp [beta = 0.3]
demo -> churn [beta = -0.1]
}")

plot(graphLayout(g))
```

![](dag_use_case_files/figure-markdown_github/define%20model%203-1.png)

This time around neither conditioning on comp neither using all variables will help us. We'll have to use grpahical model math to discover that we need to condition on:

``` r
print(adjustmentSets(g))
```

    ## { comp, demo }
    ## { comp, population }

So running the model:

![churn = \\beta\_1cong + \\beta\_2comp + \\beta\_3 demo + \\epsilon](https://latex.codecogs.com/png.latex?churn%20%3D%20%5Cbeta_1cong%20%2B%20%5Cbeta_2comp%20%2B%20%5Cbeta_3%20demo%20%2B%20%5Cepsilon "churn = \beta_1cong + \beta_2comp + \beta_3 demo + \epsilon")

``` r
sim_data <- simulateSEM(g, N = N, standardized = T)
model <- lm(churn ~ cong + comp + demo, data = sim_data)
```

We get the following coefficients:

``` r
pandoc.table(coefficients(model))
```

<table style="width:57%;">
<colgroup>
<col width="19%" />
<col width="11%" />
<col width="12%" />
<col width="13%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">cong</th>
<th align="center">comp</th>
<th align="center">demo</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">-0.001032</td>
<td align="center">0.148</td>
<td align="center">0.8984</td>
<td align="center">-0.09993</td>
</tr>
</tbody>
</table>

Obtaining the true congestion effect.
