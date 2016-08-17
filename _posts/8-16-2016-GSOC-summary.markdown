---
layout: post
title:  "End of Summer Recap"
date:   2016-08-16 22:01:43 +0530
categories: jekyll update
author: "obiajulu"
---
Woww, the summer has flown by fast and Google SoC with it! Here, with only one week to go before the final code submission, I wanted to take this moment to pause and look back on all of the work the was done and some of the work still left to do. So, without further ado, here is my GSoc in recap:

# Two New ODE Solvers, and one on the way...
## `ode4am`: a 
## `ode113`: a fully adaptive Adam Moulton solver
## `radau` : (WIP) an adaptive implicit Runge-Kutta

# Enhancements to IVP Benchmarking
## Getting the engines running again
## Experimenting with terminal mode 

The motivation for working on a Julia terminal mode for the IVPTestSuite was all of the benchmarking necessary to review @pawel's "[WIP] Adding iterators" PR. Mauro and I were, for a while, the only ones who knew IVPTestSuite.j well enough to run these important benchmarks, and we came across some serious performance issues. However, others were not able to run the scripts for themselves. Actually, duing JuliaCon (more on that later) I threw together an experimental branch call `terminalmode` which allowed the user to run the testsuites and generate corresponding plots all from the Julia terminal. The script for testing these, which still works since the experimental branch is still up,  was:

{% highlight ruby %}
Pkg.checkout("IVPTestSuite.jl", "teriminalmode")
using IVPTestSuite.QuickSuites
runtestsuite(ODEsolverfns = [ODE.ode113,ODE.ode45, ODE.ode78], abstols = 10.0.^(-5:-1:-14))
plottestsuite()
{% endhighlight %}

## More natural notebook mode

# JuliaCon
## My Talk
<center><iframe width="700" height="500" src="https://www.youtube.com/embed/dONbskqVMVs" frameborder="0" allowfullscreen></iframe></center>
## Meeting people from Alan to Yingbo
## Imfamous PR 49 "[WIP] Adding iterators"

# TL;DR: 
##Commit History:
- Experimenting with basic Adam Bashforth Solver: [https://github.com/obiajulu/ODE.jl/commits/ob/a-b?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/ob/a-b?author=obiajulu)
- Adam Moulton Solvers: [https://github.com/obiajulu/ODE.jl/commits/ob/a-b_adaptive?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/ob/a-b_adaptive?author=obiajulu)
- Radau Solver: [https://github.com/obiajulu/ODE.jl/commits/radau?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/radau?author=obiajulu)
- IVPTestSuite preliminaries and new test case: [https://github.com/mauro3/IVPTestSuite.jl/commits/master?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/master?author=obiajulu)  
- Terminalmode: [https://github.com/mauro3/IVPTestSuite.jl/commits/terminalmode?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/terminalmode?author=obiajulu)
- Notebookmode: [https://github.com/mauro3/IVPTestSuite.jl/commits/notebookmode?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/notebookmode?author=obiajulu)
Authored PRs: [https://github.com/pulls?q=is%3Apr+author%3Aobiajulu](https://github.com/pulls?q=is%3Apr+author%3Aobiajulu)
- 
# Oh, the places we'll go
## Finishing `radau` solver before school starts
The major goal of GSoC which I was not able to finish (yet!) was finishing the implementation of the `radua` solver. However, my senior year at college doesn't start until September 14th, and I am planning to get `radau` to a stable state by then. My mentor @mauro3 has more freetime in September, so I think we can do it. 
