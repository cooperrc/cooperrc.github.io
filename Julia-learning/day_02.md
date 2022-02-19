# Day_02: a meta-Julia GitHub actions journey

Today the goal was to build Julia notebooks in an automatic deployment
with GitHub actions. I have used this workflow for all of my [Python
notebooks](https://cooperrc.github.io#open-educational-resources), but I
hadn't added the _extra_ Julia requirement. I wasn't sure at first how
to use Julia's _excellent_ `Pkg` tools from bash. 

## Starting with what works

My action to deploy the Jupyter Book I took directly from the
[Jupyter Book
project](https://jupyterbook.org/publish/gh-pages.html?highlight=actions).

Here was my existing `build-docs.yml`

```yaml
name: deploy-book

# Only run this when the master branch changes
on:
  push:
    branches:
    - master

# This job installs dependencies, build the book, and pushes it to `gh-pages`
jobs:
  deploy-book:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    # Install dependencies
    - name: Set up Python 3.9
      uses: actions/setup-python@v1
      with:
        python-version: 3.9

    - name: Install dependencies
      run: |
        pip install -r requirements.txt
    # Build the book
    - name: Build the book
      run: |
        jupyter-book build .
    # Push the book's HTML to github-pages
    - name: GitHub Pages action
      uses: peaceiris/actions-gh-pages@v3.5.9
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./_build/html
```

This action executes automatically as I merge into the master branch and
deploys in my personal github-pages site. Its a great way to share
content. 

## Adding Julia + IJulia to the workflow

Yaml files can be finicky, so when I copy-pasted some another github
action from the excellent [QuantEcon with
Julia](https://julia.quantecon.org/intro.html) I had some
troubleshooting to deal with for whitespaces vs tabs. The main lines I
needed were:

```yaml
    - name: Set up Julia
      uses: julia-actions/setup-julia@v1
      with: 
        version: 1.7.2
    - name: Install IJulia and setup project
      shell: bash
      run: |
        julia -e 'using Pkg; Pkg.add("IJulia");'
        julia --project=Julia-learning --threads auto -e 'using Pkg; Pkg.instantiate();'
```

Where I would use `julia-actions` to get access to Julia. Then, install
`IJulia` using the `julia -e 'using Pkg; Pkg.add("IJulia")'`. Nice!

The result was that I had the new [Day_01](./day_01.md) post added to
the sections in the Learning Julia table of contents. 

## Including plots

The page existed, but yesterday I was workin  on plotting with the
`Plots` package. I received this error in the notebook:

```julia
ArgumentError: Package Plots not found in current path:
- Run `import Pkg; Pkg.add("Plots")` to install the Plots package.


Stacktrace:
 [1] require(into::Module, mod::Symbol)
   @ Base ./loading.jl:967
 [2] eval
   @ ./boot.jl:373 [inlined]
 [3] include_string(mapexpr::typeof(REPL.softscope), mod::Module, code::String, filename::String)
   @ Base ./loading.jl:1196
```

So, I updated the Julia build with `Plots`:

```yaml
    - name: Install IJulia and setup project
      shell: bash
      run: |
        julia -e 'using Pkg; Pkg.add("IJulia");'
        julia -e 'using Pkg; Pkg.add("Plots");' # <---------  the magic word!
        julia --project=Julia-learning --threads auto -e 'using Pkg; Pkg.instantiate();'
```

Once I had `Plots` added, the figures and _even_ animations showed up
without widgets or any other magic! _Mostly because the animations are
cleverly built as gifs_. 

## Wrapping up

Today I didn't get any real Julia development work, but I was able to
build scripts to
- install Julia packages via `julia -e 'using Pkg; ...'`
- build Jupyter Books using the Julia kernel
- display Julia figures and animations on my website
