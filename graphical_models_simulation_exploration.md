Graphical models simulation exploration
================
Iyar Lin
21 November, 2018

-   [Generate simulated dataset](#generate-simulated-dataset)
    -   [Define model](#define-model)
    -   [Generate new observations](#generate-new-observations)
-   [Estimate model](#estimate-model)
    -   [Estimate model structure](#estimate-model-structure)
    -   [Find coeffcients](#find-coeffcients)

Generate simulated dataset
==========================

Define model
------------

``` r
g <- dagitty( "dag {
Z -> W [beta = -0.4]
W -> U [beta = 0.2]
X -> W [beta = 0.35]
X -> Y [beta = -0.9]
}" )

set.seed(1)
plot(graphLayout(g))
```

![](graphical_models_simulation_exploration_files/figure-markdown_github/define%20model%20and%20plot-1.png)

``` r
print(impliedConditionalIndependencies(g))
```

    ## U _||_ X | W
    ## U _||_ Y | X
    ## U _||_ Y | W
    ## U _||_ Z | W
    ## W _||_ Y | X
    ## X _||_ Z
    ## Y _||_ Z

Generate new observations
-------------------------

``` r
sim_data <- simulateSEM(g, N = 100000)
```

Estimate model
==============

Estimate model structure
------------------------

``` r
mfit <- mgm(sim_data, type = rep("g", ncol(sim_data)), level = rep(1, ncol(sim_data)), k = 3, verbatim = T)
```

    ## Note that the sign of parameter estimates is stored separately; see ?mgm

``` r
FactorGraph(mfit, labels = names(sim_data))
```

![](graphical_models_simulation_exploration_files/figure-markdown_github/find%20graph-1.png)

Looks like the nodewise regression model was able to reconstruct correctly the model structure. Now it's up to the data scientist to determine the diretions of the arrows based on domain knowledge.

Find coeffcients
----------------

Assuming one was able to correctly identify the arrow directions I'll now compare the estimated overall effect of X on U running a regular regression, and when adjusting correctly. We know that the true overall effect is:

``` r
pander::pandoc.table(data.frame(variable = c("Z", "W", "X", "Y"), overall_effect = c(-0.08, 0.2, 0.07, 0)))
```

<table style="width:38%;">
<colgroup>
<col width="15%" />
<col width="22%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">variable</th>
<th align="center">overall_effect</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">Z</td>
<td align="center">-0.08</td>
</tr>
<tr class="even">
<td align="center">W</td>
<td align="center">0.2</td>
</tr>
<tr class="odd">
<td align="center">X</td>
<td align="center">0.07</td>
</tr>
<tr class="even">
<td align="center">Y</td>
<td align="center">0</td>
</tr>
</tbody>
</table>

### Naive model

``` r
naive_model <- lm(U ~ ., data = sim_data)
pander::pandoc.table(coefficients(naive_model))
```

<table style="width:82%;">
<colgroup>
<col width="19%" />
<col width="12%" />
<col width="16%" />
<col width="16%" />
<col width="16%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">W</th>
<th align="center">X</th>
<th align="center">Y</th>
<th align="center">Z</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.0001659</td>
<td align="center">0.1971</td>
<td align="center">-0.004581</td>
<td align="center">-0.002226</td>
<td align="center">-0.003075</td>
</tr>
</tbody>
</table>

The naive model only gets ![\\beta\_W](https://latex.codecogs.com/png.latex?%5Cbeta_W "\beta_W") right, missing on the others

### Causal inference

To get X we need to condition on Z only:

``` r
x_model <- lm(U ~ X + Z, data = sim_data)
pander::pandoc.table(coefficients(x_model))
```

<table style="width:47%;">
<colgroup>
<col width="19%" />
<col width="13%" />
<col width="13%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">X</th>
<th align="center">Z</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">0.00001998</td>
<td align="center">0.06698</td>
<td align="center">-0.08209</td>
</tr>
</tbody>
</table>

To get Z it's the same and we can see it above.

We can also see Y can't affect U.
