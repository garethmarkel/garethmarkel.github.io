---
title: "FiniteElementChains.jl"
excerpt: "Using neural networks to solve PDE constrained optimization problems while enforcing physical laws"
collection: portfolio
---

This software uses neural networks to solve PDE constrained optimization problems while enforcing physical laws by interpolating the neural network solution onto a finite element space. It can be found at this [GitHub page](https://github.com/garethmarkel/FiniteElementChains.jl).

Physics informed machine learning has been an area of intense research and study in recent years. One particularly prominent approach, that of physics informed neural networks (PINNs)--uses a neural network to learn the solution to an inverse PDE problem. It does this by adding a penalized "physics loss" (the physical law in question, e.g. the heat equation) to the loss and enforcing that law at a selection of "collocation points."

This has some drawbacks:

- How do you select the collocation points?
- How do you impose boundary conditions?
- How do you quantify how consistent the solution is between collocation points?

This package imposes the above using finite element spaces--during training, the neural network is evaluated via the finite element residual that would be obtained by interpolating the neural network onto an appropriate finite element space. Neural networks trained this way can outperform pure FEM solutions by orders of magnitude on a given mesh, and this can be used to solve inverse problems in a "one loop" framework. Additionally, the physical consistency of the solution can be evaluated using the finite element residual.