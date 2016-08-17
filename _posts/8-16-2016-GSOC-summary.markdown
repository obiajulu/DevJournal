---
layout: post
title:  "End of Summer Recap"
date:   2016-08-16 22:01:43 +0530
categories: jekyll update
author: "obiajulu"
---
Woww, the summer has flown by fast and Google SoC with it! Here, with only one week to go before the final code submission, I wanted to take this moment to pause and look back on all of the work the was done and some of the work still left to do. So, without further ado, here is my GSoc in recap:

# Two New ODE Solvers, and one on the way...

### `ode4am`: a 

### `ode113`: a fully adaptive Adam Moulton solver

### `radau` : (WIP) an adaptive implicit Runge-Kutta

# Enhancements to IVP Benchmarking

With all of these ODE solvesr, a natural question is "which ones works the best?" As may be expected, the answer is not so straight forward. Different problems call for different tools, and the best ODE solver to use will depend heavily on what ODE you are hoping to solve. Thus, there is clearly a need of testing the performance of ODE solvers against various test problems, as a means of both improving or developing a solvers as well as determining which solver to use on real world ODEs. To address these needs, my mentor Mauro built IVPTestSuite.jl roughly two years ago. Unforunately, other developers didn't join in on the project then, but steadily the need for IVPTestSuite back more and more apparent. Thus, I found myself hoping to help address this need and work to further develop the package this summer.  

### Getting the engines running again

My initial involvement working on Mauro's IVPTestSuite.jl was helping to update the package to be compatible with Julia v0.4 and v0.5. This was also a means for me to learn the innards of the package. Also, one of the early tasks I had was changing the underlying plotting package from Winston to PyPlot. I would work on these tasks in the afternoons, after spending the morning making progress on the new solvers. I worked on them for a week or two. 

There was also a push to remove the the instances of metaprogramming which were present in the previous version of the package. Switching the implementation to something that avoid this was also one of the tasks I did earlier on. 

Next, I moved on to adding a new testcase into the test suite: the Pleidas test problem (often abbreviated to "PLEI"). The problem is to model the movement of the Pleidas star constellation, and it is one of the traditional non-stiff problems from Hairer et al. This would come in much use later when I was developing `ode113` because the only other non-stiff testcase problem before PLEI was threebody. There is still a need for more test cases, and this could be future work for someone who is hoping get involved. 

### Experimenting with terminal mode 

The motivation for working on a Julia terminal mode for the IVPTestSuite was all of the benchmarking necessary to review @pawel's "[WIP] Adding iterators" PR. Mauro and I were, for a while, the only ones who knew IVPTestSuite.j well enough to run these important benchmarks, and we came across some serious performance issues. However, others were not able to run the scripts for themselves. Actually, duing JuliaCon (more on that later) I threw together an experimental branch call `terminalmode` which allowed the user to run the testsuites and generate corresponding plots all from the Julia terminal. The script for testing these, which still works since the experimental branch is still up,  was:

{% highlight ruby %}
Pkg.checkout("IVPTestSuite.jl", "teriminalmode")
using IVPTestSuite.QuickSuites
runtestsuite(ODEsolverfns = [ODE.ode113,ODE.ode45, ODE.ode78], abstols = 10.0.^(-5:-1:-14))
plottestsuite()
{% endhighlight %}

### A more natural interface: notebook mode

The terminal mode did the job it was supposed to do, though, it was a bit hacky. For a more long term solution, we explored the idea of having the primary interface with IVPTestSuite be through a IJulia notebook. There is currently a pull request open for this ongoing project and a gist which shows what the notebook looks like. 

{% gist 95277dc3c353b2245fcb6f3269615019 %}

While this may indeed be a valid long term solution, I have my reservations because it seems running the benchmark scripts in the notebook yields different results from in the terminal. I am still looking into this issue.

# JuliaCon
### My Talk
<center><iframe width="700" height="500" src="https://www.youtube.com/embed/dONbskqVMVs" frameborder="0" allowfullscreen></iframe></center>

### Meeting people from Alan to Yingbo

### Imfamous PR 49 "[WIP] Adding iterators"

# Commit and PR History:
- Experimenting with basic Adam Bashforth Solver: [https://github.com/obiajulu/ODE.jl/commits/ob/a-b?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/ob/a-b?author=obiajulu)
- Adam Moulton Solvers: [https://github.com/obiajulu/ODE.jl/commits/ob/a-b_adaptive?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/ob/a-b_adaptive?author=obiajulu)
- Radau Solver: [https://github.com/obiajulu/ODE.jl/commits/radau?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/radau?author=obiajulu)
- IVPTestSuite preliminaries and new test case: [https://github.com/mauro3/IVPTestSuite.jl/commits/master?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/master?author=obiajulu)  
- Terminalmode: [https://github.com/mauro3/IVPTestSuite.jl/commits/terminalmode?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/terminalmode?author=obiajulu)
- Notebookmode: [https://github.com/mauro3/IVPTestSuite.jl/commits/notebookmode?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/notebookmode?author=obiajulu)
Authored PRs: [https://github.com/pulls?q=is%3Apr+author%3Aobiajulu](https://github.com/pulls?q=is%3Apr+author%3Aobiajulu)

# Oh, the places we'll go

### Finishing `radau` solver before school starts
The major goal of GSoC which I was not able to finish (yet!) was finishing the implementation of the `radua` solver. However, my senior year at college doesn't start until September 14th, and I am planning to get `radau` to a stable state by then. My mentor @mauro3 has more freetime in September, so I think we can do it. 

### Polishing off the IVPTestSuite notebookmode
As mentioned above, there is a discripency between the benchmark results of IVPTestSuite when run in terminal mode versus notebook mode. I would like to get to the bottom of this, as well as possibly add more interactive plotting functionality by switching from PyPlot to Plotly. 
