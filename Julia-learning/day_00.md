# Day_00: Get set up 
_Turns out Julia uses base-1 indexing so I'm off to a rocky start ;)_
1. conda create julia
2. conda install -c conda-forge julia
3. enter Julia command with `julia`
4. `] add IJulia` adds Julia to the Python/conda environment (I think)

Playing with interactive code:

```julia
x = [1, 2, 3]
```

creates and displays the `3-element Vector{Int64}`. 

Trying to get dot product:

```julia
> x*x
ERROR: MethodError: no method matching *(::Vector{Int64}, ::Vector{Int64})
```

The default is a column vector, e.g. $3\times1$. So trying to multiply
these together gives me a mismatched indices error. 

Correcting the approach:

```julia
> transpose(x)*x
14
```

I can even reverse it and get the outer product
```julia
> x*transpose(x)
3×3 Matrix{Int64}:
 1  2  3
 2  4  6
 3  6  9
```

## Wrapping up

I didn't spend a long time in Julia today, but I _really_ like what I see so far. I still don't understand how you would deal with environments/dependencies. 
It seems like you should be using the latest version of Julia (1.7.2 in this case) and the latest version of Jupyter (3.2.5). It looks like I should be using 
the `pkg>` mode to setup+manage environments within Julia REPL [[Set up Julia environments]](https://towardsdatascience.com/how-to-setup-project-environments-in-julia-ec8ae73afe9c).
