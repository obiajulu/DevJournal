---
layout: post
title:  "End of Summer Recap"
date:   2016-08-16 22:01:43 +0530
categories: journal entry
author: "obiajulu"
---
Wowww, the summer has flown by fast and Google SoC with it! It seems like yesterday that I started developing for the Julia Language under my GSoC mentors Mauro Werder ([@mauro3](https://github.com/mauro3)) and Jiahao Chen ([@jiahao](https://github.com/jiahao)). My project, in a nutshell, was to implement new ODE solvers for the [ODE.jl package](https://github.com/obiajulu/ODE.jl), and further develop [IVPTestSuite.jl](https://github.com/mauro3/IVPTestSuite.jl) which is a package for ODE solver benchmarking that @mauro3 made. Here is [my project proposal](https://josephobiajulu.files.wordpress.com/2016/08/obiajulu_julia_prop.pdf) if you are interested in reading more. 

Now, with only one week to go before the final code submission, I wanted to take this moment to pause and look back on all of the work the was done over the summer and some of the work still left to do. So, without further ado, here is my GSoC in recap:

# Github footprints... commit and PR history for the summer:

I used Git for revision control and code management throughout the summer, and used Github to host online repos (indeed, the first challenge of the summer was learning the ins-and-outs of Git). Other than all of the great management and collaborative features, Git and Github make it super simple to track progress and productivity. 

Below is an accumulative list of most of the commits I made this summer organized by what aspect of the project I was working on, and the PRs I opened. 

### Commit History
- Experimenting with basic Adam Bashforth Solver: [https://github.com/obiajulu/ODE.jl/commits/ob/a-b?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/ob/a-b?author=obiajulu)
- Adam Moulton Solvers: [https://github.com/obiajulu/ODE.jl/commits/ob/a-b_adaptive?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/ob/a-b_adaptive?author=obiajulu)
- Radau Solver: [https://github.com/obiajulu/ODE.jl/commits/radau?author=obiajulu](https://github.com/obiajulu/ODE.jl/commits/radau?author=obiajulu)
- IVPTestSuite preliminaries: [https://github.com/mauro3/IVPTestSuite.jl/commits/master?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/master?author=obiajulu)  
- Terminalmode: [https://github.com/mauro3/IVPTestSuite.jl/commits/terminalmode?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/terminalmode?author=obiajulu)
- Notebookmode: [https://github.com/mauro3/IVPTestSuite.jl/commits/notebookmode?author=obiajulu](https://github.com/mauro3/IVPTestSuite.jl/commits/notebookmode?author=obiajulu)

### PRs this summer: 
- Still open: [https://github.com/pulls?q=is%3Apr+author%3Aobiajulu+is%3AOpen](https://github.com/pulls?q=is%3Apr+author%3Aobiajulu+is%3AOpen)
- Merged/Closed [https://github.com/pulls?q=is%3Apr+author%3Aobiajulu+is%3Aclosed](https://github.com/pulls?q=is%3Apr+author%3Aobiajulu+is%3Aclosed)

But my Github footprint only tells part of the story. I will do my best to fill in the rest, and write about what I was up to.

# Two New ODE solvers, and one on the way...

The goal of my Google SoC was to create new ODE solvers and then enhance the testing suite (IVPTestSuite.jl) used to benchmark them. I focused mostly on implementing solvers which built upon the classical Adam-Bashforth method (which `ode_ms` implements), all of which are especially suited for nonstiff problems. The classical AB method for nonstiff problems is a simple fixed step size method (this means the user specifies how many integrator steps to take over the time span, as opposed to adaptive methods, which use some algorithm to increase or decrease the step size to fit the desired accuracy). I decided to implement both a slightly more accurated fixed step method and, for the bulk of my time, an Adam __Moulton__ method which had variable step size, variable order (where order determines the degree of polynomial interpolation between time step points). I named these `ode_am` and `ode113` respectively

### `ode_am`: a step in the right direction ([code](https://github.com/obiajulu/ODE.jl/blob/ob/a-b_adaptive/src/adams_methods.jl#L61), [commits](https://github.com/obiajulu/ODE.jl/commits/ob/a-b_adaptive?author=obiajulu), [PR](https://github.com/JuliaLang/ODE.jl/pull/106))

This solver is the most basic Adam Moulton method. Like the Adam Bashforth solver `ode_ms`, the user specifies the step size for the time range and can set the order for the solver (which is typically 4 or 5). Unlike `ode_ms`, the solver has what is called a Corrector-Evaluation (CE) step which yields more accurate results. In a nutshell, after predicting the next `y` and using this to predict `dy` value at `t+dt`, we cycle back and use this predicted `dy` to arrive at a _more_ precise calculation of `y` and then again a more precise calculation of `dy`. Below we show the yielding increase in performance from the CE step by comparing `ode_ms` to `ode_am`:

### `ode113`: a fully adaptive Adam Moulton solver ([code](https://github.com/obiajulu/ODE.jl/blob/ob/a-b_adaptive/src/adams_methods.jl#L173), [commits](https://github.com/obiajulu/ODE.jl/commits/ob/a-b_adaptive?author=obiajulu), [PR](https://github.com/JuliaLang/ODE.jl/pull/106))

We now reach the aspect of my GSoC project which took the bulk of my time for the summer: implementing a variable order, variable step size Adam Moulton ODE solver. To say the solver is variable order is to mean that it varies the order of the underlying polynomial interpolation which all Adam methods are based on, so as to minimize the total work the algorithm must do while maintaining low error. We approximate this error each step by examining the difference in the `y` value from using the current order and the order one (or two above) and one below. If the difference in orders varies too much, then we can be sure increaseing the order will yields noticeably better results; however, if the `y` values at different orders are very close, the order may be lowered without incurring significant error. The solver also varies the step size based on this error approximation, to likewise minizime the runtime while still having an error within the specified error tolerance.

Currently, the solver is functional and performs comparably (as well or better) to the other nonstiff solvers in ODE.jl, such as `ode78` and `ode45`. Benchmark results below confirm this:

#### __function evaluations for Pleidas problem `ode113` vs `ode45`__
The real strength of Adam Bashforth methods is the minimal amount of derivative function evaluation one needs. For each iteration step, this function needs to be evaluated usually only twice (really twice for each attempt of a step, and steps are usually accepted). We present a table of the function evaluations necessary for `ode45` vs `ode113` used to solve the same Pleiadas problem above.

| tol | ode113 | ode45 | ode78 |
|---|---|---|---|
|1e-7 | 1115 | 2456 | 1996|
|1e-8 | 1327 | 3890 | 4444|
|1e-9 | 1557 | 6170 | 7320|
|1e-10 | 1859 | 9776 | 10698|
|1e-11 | 2237 | 15488 | 15133|
|1e-12 | 2875 | 24548 | 21076|
|1e-13 | 3869 | 38900 | 29008|
|1e-14 | 6179 | 61652 | 39605|

### `ode_radau` : (WIP) an adaptive implicit Runge-Kutta ([code](https://github.com/obiajulu/ODE.jl/blob/radau/src/radau.jl), [commits](https://github.com/obiajulu/ODE.jl/commits/radau?author=obiajulu), [PR](https://github.com/obiajulu/ODE.jl/pull/3))

Unlike `ode_ms` and `ode113`, `radua` is a solver aimed towards solving stiff ODEs. The basic theorey behindtween the solver is a classical implicit Runge-Kunta tableau and the use of Newton Iteration to solve the generated system of nonlinear equations at each step. Further details can be found in Hairer’s paper below.

Also, anticipating the move of ODE.jl towards iterative formulations of solvers, we are implementing this solver in a near iterative form:

{% highlight ruby %}
function ode_radau(f,y0,tspan,order)
  #set up state, `st`
  
  while !done_radau(st)
     stats = trialstep!(st)
     err, stats, st = errorcontrol!(st)
     if err < 1
         stats, st = ordercontrol!()
         accept = true
     else
         rollback!()
     end
  return status()
  end
end
{% endhighlight %}

This solver is still a WIP, and the goal is to finish before school starts. More on that later.

### Side bar: Theory behind the solvers

For those more interested in the theoretical numerical analysis of these these solvers, we recommend the following sources, which we used while implementing the solvers:

- Hairer, Nørsett, and Wanner's [Solving Ordinary Differential Equations I: Nonstiff Problems](http://www.springer.com/us/book/9783540566700) especially Chapter 2
- Hairer and Wanner's [Solving Ordinary Differential Equations II:Stiff Problems](http://link.springer.com/book/10.1007%2F978-3-642-05221-7), especially Chapter 8
- [Wikiversity article on Adam Bashforth and Adam Moulton methods](https://en.wikiversity.org/wiki/Adams-bashforth_and_Adams-moulton_methods)
- [Stiff differential equationsolved by Radau methos](http://ac.els-cdn.com/S037704279900134X/1-s2.0-S037704279900134X-main.pdf?_tid=d6bd8b8a-64bd-11e6-af6f-00000aacb35e&acdnat=1471467899_2d9205f1c3fe6f027801c84794f8835f) 
- [My Google SoC proposal](https://josephobiajulu.files.wordpress.com/2016/08/obiajulu_julia_prop.pdf)

# Enhancements to IVP Benchmarking 

With all of these ODE solvers, a natural question is "which ones works the best?" As may be expected, the answer is not so straight forward. Different problems call for different tools, and the best ODE solver to use will depend heavily on what ODE you are hoping to solve. Thus, there is clearly a need of testing the performance of ODE solvers against various test problems, as a means of both improving or developing a solver as well as determining which solver to use on real world ODEs. To address these needs, my mentor @mauro3 built IVPTestSuite.jl roughly two years ago, which was inspired by two other IVP test suites made by [Ernst Hairer & Gerhard Wanner](https://www.unige.ch/~hairer/testset/testset.html) and [INdAM Bari](https://archimede.dm.uniba.it/~testset/testsetivpsolvers/). Unfortunately, other developers didn't much join in on the project then, but steadily the need for IVPTestSuite became more and more apparent. Thus, I found myself hoping to help address this need and work to further develop the package this summer.  

### Getting the engines running again ([commits](https://github.com/mauro3/IVPTestSuite.jl/commits/master?author=obiajulu), [PR](https://github.com/mauro3/IVPTestSuite.jl/pull/5),[PR](https://github.com/mauro3/IVPTestSuite.jl/pull/10),[PR](https://github.com/mauro3/IVPTestSuite.jl/pull/11), [PR](https://github.com/mauro3/IVPTestSuite.jl/pull/10))

My initial involvement working on @mauro3's IVPTestSuite.jl was helping to update the package to be compatible with Julia v0.4 and v0.5. This was also a means for me to learn the innards of the package. Also, one of the early tasks I had was changing the underlying plotting package from Winston to PyPlot. I would work on these tasks in the afternoons, after spending the morning making progress on the new solvers. I worked on them for a week or two. 

There was also a push to remove the instances of metaprogramming which were present in the previous version of the package. Switching the implementation to something that avoid this was also one of the tasks I did earlier on. 

Next, I moved on to adding a new test case into the test suite: the Pleidas test problem (often abbreviated to "PLEI"). The problem is to model the movement of the Pleidas star constellation, and it is one of the traditional non-stiff problems from Hairer et al. This would come in much use later when I was developing `ode113` because the only other non-stiff test case problem before PLEI was threebody. There is still a need for more test cases, and this could be future work for someone who is hoping get involved. 

### Experimenting with terminal mode ([commits](https://github.com/mauro3/IVPTestSuite.jl/commits/terminalmode?author=obiajulu))

The motivation for working on a Julia terminal mode for the IVPTestSuite was all of the benchmarking necessary to review Pawel's (@pwl) "[WIP] Adding iterators" PR. @mauro3 and I were, for a while, the only ones who knew IVPTestSuite.j well enough to run these important benchmarks, and we came across some serious performance issues. However, others were not able to run the scripts for themselves. Actually, duirng JuliaCon (more on that later) I threw together an experimental branch called `terminalmode` which allowed the user to run the test suites and generate corresponding plots all from the Julia terminal. The script for testing these, which still works since the experimental branch is still up,  was:

{% highlight ruby %}
Pkg.checkout("IVPTestSuite.jl", "teriminalmode")
using IVPTestSuite.QuickSuites
runtestsuite(ODEsolverfns = [ODE.ode113,ODE.ode45, ODE.ode78], abstols = 10.0.^(-5:-1:-14))
plottestsuite()
{% endhighlight %}

### A more natural interface: notebook mode ([commits](https://github.com/mauro3/IVPTestSuite.jl/commits/notebookmode?author=obiajulu), (PR)[pending])

The terminal mode did the job it was supposed to do, though, it was a bit hacky. For a more long term solution, we explored the idea of having the primary interface with IVPTestSuite be through an IJulia notebook. There is currently a pull request open for this ongoing project and a gist which shows what the notebook looks like. 

{% gist 95277dc3c353b2245fcb6f3269615019 %}

While this may indeed be a valid long term solution, I have my reservations because it seems running the benchmark scripts in the notebook yields different results from in the terminal. I am still looking into this issue.



# JuliaCon
Another great part of my Google Summer of Code experience was attending the 3rd annual [JuliaCon](www.juliacon.org) in MIT's StataCenter in Early June. It is a conference run by Julia developers, for Julia developers (and users), where there were "cutting-edge technical talks, hands-on workshops, [and] a chance to rub shoulders with Julia's creators." I was invited to give a short talk about my work, which I embedded below.

### My Talk
<center><iframe width="700" height="500" src="https://www.youtube.com/embed/dONbskqVMVs" frameborder="0" allowfullscreen></iframe></center>

### Meeting people from Alan to Yingbo
While at JuliaCon, I met a bright high school student, by the name of Yingbo ([@yingbo_ma](www.github.com/yingbo_ma)), who was an ODE.jl user. After talking things over with him and my second mentor @Jiahao, we made room for him to join us in the lab for the rest of my stay and even after I left. It’s been great collaborating with him this summer. During JuliaCon, I was also able to talk face-to-face with @pwl, who is a pot-Doctoral student in Applied Mathematics in Germany and a main contributor to ODE.jl. We worked on his imfamous [PR 49](https://github.com/JuliaLang/ODE.jl/pull/49), "[WIP] Adding iterators"]. , “[WIP] Adding iterators”]. I was able to learn a lot about the interative formulation for ODE solvers from him, and later toyed around with implementing multistep methods (like the Adam Bashforth or simple Adam Moulton method) in iterative form. I was also able to stay after JuliaCon for two or so weeks, working out of the Edelmanlab, where Julia was born. Yingbo and I were able to grab a meal with Prof. Edelman and talking to him more about Julia, its past and future direction.

# Oh, the places we'll go... Work left to do

### Finishing `ode_radau` solver before school starts
The major goal of GSoC which I was not able to finish (yet!) was finishing the implementation of the `ode_radua` solver. However, my senior year at college doesn't start until September 14th, and I am planning to get `ode_radau` to a stable state by then. My mentor @mauro3 has more freetime in September, so I think we can do it. 

### Polishing off the IVPTestSuite notebookmode
As mentioned above, there is a discrepancy  between the benchmark results of IVPTestSuite when run in terminal mode versus notebook mode. I would like to get to the bottom of this, as well as possibly add more interactive plotting functionality by switching from PyPlot to Plotly. 
