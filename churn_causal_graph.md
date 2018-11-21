Causal graph
================
Iyar Lin
21 November, 2018

-   [Causal graph](#causal-graph)
    -   [Variables](#variables)
    -   [Outages](#outages)

Causal graph
============

Variables
---------

![d\_t\\in \\{0,1\\}](https://latex.codecogs.com/png.latex?d_t%5Cin%20%5C%7B0%2C1%5C%7D "d_t\in \{0,1\}") - Subscriber disconnected at tenure ![t](https://latex.codecogs.com/png.latex?t "t")
![T \\in \[0,\\infty)](https://latex.codecogs.com/png.latex?T%20%5Cin%20%5B0%2C%5Cinfty%29 "T \in [0,\infty)") - Subscriber life time
comp - Competition measurements
cong - Network congestion measurements
dem - Demographics (credit score, population density, age)
pop - Population count in beam
pool - Perspective customers pool (count)
plan - Plan set offered
subs - Number of subscribers on the network (that share bandwidth)
t\_s - Time since satellite lunched
cap - Maximum capacity

![](churn_causal_graph_files/figure-markdown_github/plot%20causal%20graph-1.png)

A simpler model might be:

![](churn_causal_graph_files/figure-markdown_github/plot%20simpler%20causal%20graph-1.png)

Outages
-------

It's possible that more than congenstion outages causes disconnects.
