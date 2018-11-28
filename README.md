# Churn causality  
This repo contains scripts and material which aim to uncover the causal relation between churn and variuos factors such as competition, network congestion etc.

# To do  
* Working with non-linear models:
    * Learning Directed Graphical Models from Nonlinear and Non-Gaussian Data: https://drive.google.com/open?id=11f0WbAZ5JIqFCrRPp0IZI95lb7J-rH4K  - not sure if that works
    * Look slike mgm is the way to go in general but it stil needs more refinment. Work is done at: https://github.com/benoslab/causalMGM
* Latent variabels: (instrumental variables, front door criterion). Instrumental is found via daggity. Front door - need to find some other option.
* Grouping variables (e.g. competition)  
* Model fit and validation (can't do cross validation)  
* Deal with bidirectional dependencies (These will arise a lot!) - these can be canonalized = add Latent variables in between
* Blog posts:
    * Why use DAG for causal inference with concerete example
    * Using DAG for causal inference with mixed domain data
* Consider starting a packacge/adding to an exiting one  
* Do write-up for churn indetification.