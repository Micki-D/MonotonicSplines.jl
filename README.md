# MonotonicSplines.jl

[![Documentation for stable version](https://img.shields.io/badge/docs-stable-blue.svg)](https://bat.github.io/MonotonicSplines.jl/stable)
[![Documentation for development version](https://img.shields.io/badge/docs-dev-blue.svg)](https://bat.github.io/MonotonicSplines.jl/dev)
[![License](http://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat)](LICENSE.md)
[![Build Status](https://github.com/bat/MonotonicSplines.jl/workflows/CI/badge.svg?branch=main)](https://github.com/bat/MonotonicSplines.jl/actions?query=workflow%3ACI)
[![Codecov](https://codecov.io/gh/bat/MonotonicSplines.jl/branch/main/graph/badge.svg)](https://codecov.io/gh/bat/MonotonicSplines.jl)
[![Aqua QA](https://raw.githubusercontent.com/JuliaTesting/Aqua.jl/master/badge.svg)](https://github.com/JuliaTesting/Aqua.jl)


This package provides high-performance, GPU- and
[AD](https://en.wikipedia.org/wiki/Automatic_differentiation)-friendly
monotonic spline functions in Julia for use in
[Normalizing Flows](https://en.wikipedia.org/wiki/Flow-based_generative_model),
resp. parameter transformations in general.

This package currently includes the *monotonic rational quadratic splines* as defined in ["Neural Spline Flows, Durkan et al. 2019"](https://arxiv.org/abs/1906.04032).

This package was created with high performance as the overall goal and features automatic code differentiation, powered by [Zygote.jl](https://fluxml.ai/Zygote.jl/latest/), and GPU compatibility, powered by [KernelAbstractions.jl](https://juliagpu.github.io/KernelAbstractions.jl/stable/).

## Documentation

* [Documentation for stable version](https://bat.github.io/MonotonicSplines.jl/stable)
* [Documentation for development version](https://bat.github.io/MonotonicSplines.jl/dev)

## Installation


Please see the Documentation linked above for details.
To install `MonotonicSplines.jl`, start Julia and run 

```Julia
julia> using Pkg
julia> pkg.add("MonotonicSplines.jl")
```
and 
```Julia
julia> using MonotonicSplines
```
to use the functions the package provides.

## Example usage for rational quadratic spline functions

Obtain `widths`, `heights`, and `derivatives` parameters to characterize a set of rational quadratic spline functions after the parameterization described in ["Neural Spline Flows, Durkan et al."](https://arxiv.org/abs/1906.04032):
```Julia
julia> widths, heights, derivatives = get_params(params_raw, n_dims_to_transform) 
```
`params_raw` is in general the output of a neural net, applied to a set of samples, and has the shape of a ``3(K-1) *n_dims_to_transform x n_samples`` -matrix.

Here, `K` is the number of spline segments in the rational quadratic spline functions, `n_dims_to_transform` is the number of spline functions per sample (i.e. the number of components of a sample that are supposed to be transformed via a spline function) and `n_samples` is the number of samples. 

One column of this matrix `params_raw` is then the output of the neural net, with one sample vector `x` as the input, and will be processed into the spline parameters to characterize the rational quadratic spline functions used to transform the desired `n_dims_to_transform` components of the input sample `x`.

```Julia
julia> rqs_splines = RQSpline(widths, heights, derivatives)
```
Create `RQSpline` object, holding the parameters to characterize ``n_dims_to_transform x n_samples`` spline functions.

```Julia
julia> Y = rqs_splines(X)
```
Apply the rational quadratic spline functions to the matrix ``n_dims_to_transform x n_samples`` `X`. One column in `X` corresponds to a single sample. 

Note that if you have a set of samples `S` of which you wish to transform `n_dims_to_transform` components each, only apply `rqs_splines` to the partial sample set `X`, with only the `n_dims_to_transform` components of each sample that are supposed to be transformed.

`Y` is a ``n_dims_to_transform x n_samples`` matrix, where the ``i,j`` -th component is the transformed value of the ``i,j`` -th entry in `X`. 

For details of the implementation, see the [Documentation for stable version](https://bat.github.io/MonotonicSplines.jl/stable).
