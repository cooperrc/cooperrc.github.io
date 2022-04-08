---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Julia 1.7.2
  language: julia
  name: julia-1.7
---

# Day_08: Filing and tracking issues in GitHub

Today I ran into some strange behavior when I tried to generalize my Lagrangian equations of motion. It boils down to something happening with Symbolic `Vectors`

Take for example

```{code-cell}
using Symbolics

@variables t x[1:2](t) # independent and dependent variables
@variables k # parameters

Dx1 = Differential(x[1])
Dx2 = Differential(x[2])

V = 1/2*k*(x[2]- x[1])^2
```

I define a symbolic function `V` that is

$\frac{1}{2}\left(x_2 - x_1\right)^2$. 

Now, I take a derivative with respect to $x_1$

$\frac{\partial L}{\partial x_1} = -k(x_2 - x_1)$

but instead I get

```{code-cell}
Symbolics.expand_derivatives(Dx1(V))
```

_Weird!_

Expanding the polynomial there are 3 terms
- $1/2 kx_1^2$
- $1/2 kx_2^2$
- $-kx_1x_2$

So, I'll take derivatives of these terms and see what I get

```{code-cell}
Vterms = [1/2*k*x[1]^2,1/2*k*x[2]^2, -k*x[1]*x[2]]
```

```{code-cell}
Symbolics.expand_derivatives.(Dx1.(Vterms))
```

Here is the root of my problem, for some reason the partial derivative `Differential(x[1])` is treating `x[1]` and `x[2]` as the same variable. The result should have been

- $kx_1$
- $0$
- $-kx_2$


## Unindexed variables work
Now, I can compare this to an unindexed set of variables

```{code-cell}
@variables t y1(t) y2(t) # independent and dependent variables
@variables k # parameters

Dy1 = Differential(y1)
Dy2 = Differential(y2)

V = 1/2*k*(y2- y1)^2
```

```{code-cell}
Symbolics.expand_derivatives(Dy1(V))
```

This returns the correct derivative, 

$\frac{\partial V}{\partial y_1} = -k(y_2 - y_1)$

```{code-cell}
@variables z_1(t)
```

```{code-cell}
D = Differential(t)
```

```{code-cell}
D.(x)
```

```{code-cell}
D.([y1,y2])
```

## Symbolics still adding support for arrays

The Symbolics.jl package is incredible. Its still working on supporting differentiation with arrays. I filed this behavior on [github.com/Symbolics.jl issue #571](https://github.com/JuliaSymbolics/Symbolics.jl/issues/571). From what I can see, the main driver behind symbolic differentiation is in the [`diff.jl`](https://github.com/JuliaSymbolics/Symbolics.jl/blob/master/src/diff.jl) file. 

I was getting the error from line 58

```julia
    if symtype(expr) <: AbstractArray
        error("Differentiation of expressions involving arrays and array variables is not yet supported.")
    end
```

but [@shashi](https://github.com/shashi) added some great exceptions that remove this error in [#570](https://github.com/JuliaSymbolics/Symbolics.jl/pull/570) and [#568](https://github.com/JuliaSymbolics/Symbolics.jl/pull/570). There is always more work to be done on projects like this and because Julia is open source, I can dig into the code, offer help and try to add my experiences to the solution. 

+++

## Wrapping up

Today, I found some strange behavior in the Symbolics.jl package. I'll continue to update/add to the Julia projects as issues or hopefully merge some improvements. It would be great to document the process of a PR merge from start-to-finish. 

For now, the work-around appears to be that `jacobian` and `Differential` only support scalar values. So I can create a `Vector` of individual values as such

```julia
@variables x1(t) x2(t)
x = [x1, x2]
```

The development behind Julia and Symbolics.jl is active and inspiring, so I'm sure this work-around won't need to exist much longer.
