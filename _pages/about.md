---
permalink: /
title: "About Me"
author_profile: true
redirect_from:
- /about/
- /about.html
---

## Background

I am a quantitative scientist specializing in the development and application of computational methods for complex technical problems. My work focuses on translating advances in statistics, mathematics, and machine learning into practical tools for scientific, engineering, and financial applications.

My recent work includes a large-scale analysis of the genotypes of ~80,000 sibling pairs spanning 6 countries (forthcoming in *AER: Insights*), a Julia package to efficiently integrate neural networks into finite element analysis, and research on the theoretical limits of genetic prediction for certain cancers.

I hold a Ph.D. in economics from George Mason University, with fields in experimental economics and econometric methods for genetic research. For my dissertation, *Essays in Social Genomics*, I leveraged causal inference, high performance computing, and an extremely large biomedical datasets to tackle questions related to labor economics, healthcare economics, and human capital formation.

## Current work

In my professional role, I am a senior data scientist leading the development of machine learning and AI capabilities within a US federal agency, including information retrieval systems, evaluation frameworks, and decision-support tools for engineering and operational stakeholders.

## Research areas

1. Causal inference, econometrics, statistical learning
2. Numerical methods, optimization, and scientific computing
3. Time series methods and forecasting
4. Computational and statistical genetics
5. Machine learning, deep learning and artificial intelligence

## Selected Projects

#### [Nature, Nurture, and Socioeconomic Outcomes: New Evidence from Sib Pairs and Molecular Genetic Data (forthcoming, American Economic Review: Insights)](https://www.aeaweb.org/articles?id=10.1257/aeri.20250281&&from=f)
##### This paper uses extremely large scale biomedical data to identify 80,000 pairs of siblings (from a sample of over 1,000,000 participants spanning 6 countries), and compute the proportion of their genomes that each sibling pair shares. This similarity is randomly assigned to any pair of siblings (excluding full twins), which makes it the ultimate natural experiment, and cleanly identifies the effect of genetic similarity on trait similarity for siblings in our sample. We compute this effect for 15 traits, ranging from risk tolerance to smoking initiation, and find evidence of sizeable genetic influences on many outcomes.

#### [FiniteElementChains.jl](https://garethmarkel.github.io/FiniteElementChains.jl/dev/)
##### This package provides a set of tools for solving inverse problems constrained by partial differential equations using neural networks. The PDE constraints are enforced using the finite element method.

#### [PRScs.jl](https://garethmarkel.github.io/posts/prs-cs-jl/)
##### This package provides a set of tools for enhancing the predictive power of polygenic scores created using GWAS weights, by using a Bayesian sparse high dimensional regression framework to regularize the raw GWAS weights,
