"Correlation is not causation". So what is?
================
Iyar Lin
03 January, 2019

Intro
=====

Machine learning applications have been growing in volume and scope rapidly over the last few years. What's Causal inference, how is it different than plain good ole' ML and when should you consider using it? In this report I try giving a short and concrete answer by using an example.

A typical data science task
===========================

Imagine we're tasked by the marketing team to find the effect of raising marketing spend on sales. We have at our disposal records of marketing spend (mkt), visits to our website (visits), sales, and competition index (comp). Below are the first few rows of the dataset:

<table style="width:46%;">
<colgroup>
<col width="11%" />
<col width="12%" />
<col width="11%" />
<col width="11%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">mkt</th>
<th align="center">visits</th>
<th align="center">sales</th>
<th align="center">comp</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">282.5</td>
<td align="center">2977</td>
<td align="center">379</td>
<td align="center">3.635</td>
</tr>
<tr class="even">
<td align="center">338.8</td>
<td align="center">3149</td>
<td align="center">308</td>
<td align="center">4.515</td>
</tr>
<tr class="odd">
<td align="center">303.9</td>
<td align="center">2485</td>
<td align="center">369</td>
<td align="center">3.092</td>
</tr>
<tr class="even">
<td align="center">558.8</td>
<td align="center">3117</td>
<td align="center">191</td>
<td align="center">5.22</td>
</tr>
<tr class="odd">
<td align="center">334.4</td>
<td align="center">4038</td>
<td align="center">286</td>
<td align="center">4.281</td>
</tr>
<tr class="even">
<td align="center">297.7</td>
<td align="center">2854</td>
<td align="center">441</td>
<td align="center">3.592</td>
</tr>
</tbody>
</table>

We'll simulate a dataset using a set of equations (also called structural equations):

![sales = \\beta\_1vists + \\beta\_2comp + \\epsilon\_1](https://latex.codecogs.com/png.latex?sales%20%3D%20%5Cbeta_1vists%20%2B%20%5Cbeta_2comp%20%2B%20%5Cepsilon_1 "sales = \beta_1vists + \beta_2comp + \epsilon_1")

![vists = \\beta\_3mkt + \\epsilon\_2](https://latex.codecogs.com/png.latex?vists%20%3D%20%5Cbeta_3mkt%20%2B%20%5Cepsilon_2 "vists = \beta_3mkt + \epsilon_2")

![mkt = \\beta\_4comp + \\epsilon\_3](https://latex.codecogs.com/png.latex?mkt%20%3D%20%5Cbeta_4comp%20%2B%20%5Cepsilon_3 "mkt = \beta_4comp + \epsilon_3")

with ![\\{\\beta\_1, \\beta\_2, \\beta\_3\\, \\beta\_4\\} = \\{0.3, -0.9, 0.5, 0.6\\}](https://latex.codecogs.com/png.latex?%5C%7B%5Cbeta_1%2C%20%5Cbeta_2%2C%20%5Cbeta_3%5C%2C%20%5Cbeta_4%5C%7D%20%3D%20%5C%7B0.3%2C%20-0.9%2C%200.5%2C%200.6%5C%7D "\{\beta_1, \beta_2, \beta_3\, \beta_4\} = \{0.3, -0.9, 0.5, 0.6\}")

All data presented in graphs or used to fit models below is simulated from the above equations.

Our goal is to predict the effect of raising marketing spend on sales which is 0.15 (from the set of equations above, using product decomposition we get ![\\beta\_1 \\cdot \\beta\_3 = 0.3 \\cdot 0.5 = 0.15](https://latex.codecogs.com/png.latex?%5Cbeta_1%20%5Ccdot%20%5Cbeta_3%20%3D%200.3%20%5Ccdot%200.5%20%3D%200.15 "\beta_1 \cdot \beta_3 = 0.3 \cdot 0.5 = 0.15")).

Common analysis approaches
==========================

First approach: plot bi-variate relationship
--------------------------------------------

Many of us would start off by plotting a scatter plot of sales by marketing:

![](correlation_is_not_causation_so_what_is_files/figure-markdown_github/plot%20scatter%20plot-1.png)

We can see that the relationship seen in the graph is actually the opposite of what we'd expected! It looks like increasing marketing actually decreases sales. Indeed, not only correlation isn't causation, at times it can even show the opposite.

Fitting a simple linear model ![sales = r\_0 + r\_1mkt + \\epsilon](https://latex.codecogs.com/png.latex?sales%20%3D%20r_0%20%2B%20r_1mkt%20%2B%20%5Cepsilon "sales = r_0 + r_1mkt + \epsilon") would yield the following coefficients (note ![r](https://latex.codecogs.com/png.latex?r "r") is a regression coefficient where's ![\\beta](https://latex.codecogs.com/png.latex?%5Cbeta "\beta") is a true parameter in the structural equations):

<table style="width:32%;">
<colgroup>
<col width="19%" />
<col width="12%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">mkt</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">513.5</td>
<td align="center">-0.3976</td>
</tr>
</tbody>
</table>

Confirming that we get a vastly different effect than the one we were looking for (0.15).

Second approach: Use ML model with all available features
---------------------------------------------------------

One might postulate that looking on a bi-variate relation amounts to using only 1 predictor variable, but if we were to use all of the available features we might be able to find a more accurate estimate.

When running the regression ![sales = r\_0 + r\_1mkt + r\_2visits + r\_3comp + \\epsilon](https://latex.codecogs.com/png.latex?sales%20%3D%20r_0%20%2B%20r_1mkt%20%2B%20r_2visits%20%2B%20r_3comp%20%2B%20%5Cepsilon "sales = r_0 + r_1mkt + r_2visits + r_3comp + \epsilon") we get the following coefficients:

<table style="width:62%;">
<colgroup>
<col width="19%" />
<col width="15%" />
<col width="13%" />
<col width="13%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">mkt</th>
<th align="center">visits</th>
<th align="center">comp</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">596.7</td>
<td align="center">0.009642</td>
<td align="center">0.02849</td>
<td align="center">-90.06</td>
</tr>
</tbody>
</table>

Now it looks like marketing has almost no effect at all! Since we simulated the data from a set of linear equations we know that using more sophisticated models (e.g. XGBoost, GAMs) can't produce better results (I entourage the skeptic reader to try this out by re-running the [Rmd script](https://github.com/IyarLin/causal-inference/blob/master/correlation_is_not_causation_so_what_is.Rmd) used to produce this report).

Maybe we should consider the relation between features too...
=============================================================

When consulting the marketing team we learn that in highly competitive markets the team would usually increase marketing spend (this is reflected in the coefficient ![\\beta\_4 = 0.6](https://latex.codecogs.com/png.latex?%5Cbeta_4%20%3D%200.6 "\beta_4 = 0.6") above). So it's possible that competition is a "confounding" factor since when we observe high marketing spend there's also high competition thus leading to lower sales.

Also, we notice that marketing probably affects visits to our site and those visits in turn affect sales.

We can visualize these feature inter-dependencies with a directed a-cyclic graph (DAG):

![](correlation_is_not_causation_so_what_is_files/figure-markdown_github/plot%20DAG2-1.png)

So it would make sense to account for the confounding competition by adding it to our regression. Adding visits to our model however somehow "blocks" or "absorbs" the effect of marketing on sales so we should omit it from our model.

Fitting the model ![sales = r\_0 + r\_1mkt + r\_2comp + \\epsilon](https://latex.codecogs.com/png.latex?sales%20%3D%20r_0%20%2B%20r_1mkt%20%2B%20r_2comp%20%2B%20%5Cepsilon "sales = r_0 + r_1mkt + r_2comp + \epsilon") yields the coefficients below:

<table style="width:44%;">
<colgroup>
<col width="19%" />
<col width="12%" />
<col width="12%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">(Intercept)</th>
<th align="center">mkt</th>
<th align="center">comp</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">662.2</td>
<td align="center">0.1524</td>
<td align="center">-90.89</td>
</tr>
</tbody>
</table>

Now we finally got the right effect estimate!

The way we got there was a bit shaky though. We came up with general concepts around "confounding" and "blocking" of features. Trying to apply those to datasets consisting of tens of variables with complicated relationships would probably prove tremendously hard.

So now what? Causal inference!
==============================

So far we've seen that trying to estimate the effect of marketing spend on sales by examining bi-variate plots can fail bad. We've also seen that standard ML practices of throwing all available features into our model can fail too. It would seem we need to carefully construct the set of covariates included in our model in order to obtain the true effect.

In causal inference this covariate set is also termed "adjustment set". Given a model DAG we can utilize various algorithms that rely on rules in similar spirit to the considerations we mentioned above such as "confounding" and "blocking", to find the correct adjustment set.

Backdoor criteria
-----------------

One of the most basic algorithms that can obtain the correct adjustment set is the "Backdoor-criteria" developed by J. Pearl. In a nutshell it seeks adjustment sets that block every "spurious" paths between our "exposure" variable (e.g. marketing) and "outcome" variable (e.g. sales) while keeping directed baths open.

Consider for example the DAG below where we're interested in finding the effect of x5 on x10:

![](correlation_is_not_causation_so_what_is_files/figure-markdown_github/plot%20large%20DAG-1.png)

Using the backdoor-criterion (implemented in the R package "dagitty") we can find the correct adjustment sets:

![](correlation_is_not_causation_so_what_is_files/figure-markdown_github/plot%20adjustemnt%20sets-1.png)

How to obtain model DAGs?
-------------------------

Finding the model DAG can be admittedly challenging. It can be done using any combination of the following:

-   Use domain knowledge
-   Given a few candidate model DAGs one can perform statistical tests to compare their fit to the data at hand
-   Use search algorithms (e.g. those implemented in the R "mgm" package)

I'll touch upon this subject in more breadth in a future post.

Further reading
---------------

To anyone curious to learn a bit more about the questions I've tried to answer in this report I'll recommend reading the light-weight Technical Report by Pearl: [The Seven Tools of Causal Inference with Reflections on Machine Learning](https://ftp.cs.ucla.edu/pub/stat_ser/r481.pdf).

For a more in-depth introduction to Causal inference and the DAG machinery I'd recommend getting Pearl's short book: [Causal Inference in Statistics - A Primer](https://www.amazon.com/Causal-Inference-Statistics-Judea-Pearl/dp/1119186846)
