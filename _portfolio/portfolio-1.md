---
title: "PRScs.jl"
excerpt: "Using Bayesian regression to create polygenic risk score weights from GWAS estimates"
collection: portfolio
---

Genotype data is incredibly high dimensional--$n>>p$. This is problematic for many traditional
machine learning tools. One approach that has found success is to take SNP associations from
GWA studies, and use an external reference panel to correct the part of the bias due to other 
ommitted SNPs. Here, I do this with Bayesian regression. I implement two families of prior for
this--first, I reimplement  Ge et al's [Strawderman-Berger](https://github.com/getian107/PRScs/blob/master/README.md)
prior, then I derive and implement an additional Dirichlet-Laplace prior. Please see the readme,
and a forthcoming blog post, for more information.