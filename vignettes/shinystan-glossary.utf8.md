---
title: 'ShinyStan: Glossary'
date: "2019-04-08"
output: 
  rmarkdown::html_vignette:
    toc: true
vignette: >
  %\VignetteIndexEntry{Glossary}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

# General MCMC

## \(n_{eff}\) (ESS)

<em>Quick definition</em>

\(n_{eff}\) is an estimate of the effective number of independent draws from 
the posterior distribution of the estimand of interest. Because the draws 
within a chain are not independent if there is autocorrelation, the 
effective sample size will be smaller than the total number of iterations.

<br><br>
<em>More details</em>

<p>
Samples in a Markov chain are only drawn with the marginal distribution 
\(p(\theta | y,x)\) after the chain has converged to its equilibrium distribution. 
There are several methods to test whether an MCMC method has failed to converge; 
unfortunately, passing the tests does not guarantee convergence. The recommended 
method for Stan is to run multiple Markov chains, initialized randomly with a 
diffuse set of initial parameter values, discard the warmup/adaptation samples, 
then split the remainder of each chain in half and compute the potential 
scale reduction statistic \(\hat{R}\). 
</p>

<p>
If the effective sample size is too low to make inferences with the desired 
precision, double the number of iterations and start again, including rerunning 
warmup and everything. Often, a small effective sample size is the result of too 
few warmup iterations. At most, this rerunning strategy will consume about 
50% more cycles than guessing the correct number of iterations at the outset.
</p>

<p>
The estimation of effective sample size is described in detail in the 'Markov
Chain Monte Carlo Sampling' chapter of the
<a href="http://mc-stan.org/documentation/">
  Stan Modeling Language User's Guide and Reference Manual.</a>
</p>


## \(se_{mean}\) (mcse) 

<em>Quick definition</em>

The standard error of the mean of the posterior draws (not to be confused
with the standard deviation of the posterior draws) is the uncertainty
associated with the Monte Carlo approximation. This quantity approaches 0 as
the sample size goes to infinity, whereas the standard deviation of the 
posterior draws approaches the standard deviation of the posterior distribution.

<br><br>
<em>More details</em>

<p>
When estimating a mean based on a sample of \(M\) independent draws, the 
estimation error is proportional to \(1/M\). If the draws are positively 
correlated, as they typ￼ically are when drawn using MCMC methods, the error
is proportional to \(1/\sqrt{n_{eff}}\) where \(n_{eff}\) is the effective 
sample size. Thus it is standard practice to also monitor (an estimate of) 
the effective sample size until it is large enough for the estimation or
inference task at hand.
</p>

## \(\hat{R}\) (Rhat)

<em>Quick definition</em>

One way to monitor whether a chain has converged to the equilibrium 
distribution is to compare its behavior to other randomly initialized chains. 
This is the motivation for the Gelman and Rubin potential scale reduction 
statistic \(\hat{R}\). The \(\hat{R}\) statistic measures 
the ratio of the average variance of samples within each chain to the variance
of the pooled samples across chains; if all chains are at equilibrium, these 
will be the same and \(\hat{R}\) will be one. If the chains have not converged 
to a common distribution, the \(\hat{R}\) statistic will be greater than one.

<br><br>
<em>More details</em>

<p>
Gelman and Rubin’s recommendation is that the independent Markov chains be 
initialized with diffuse starting values for the parameters and sampled until 
all values for \(\hat{R}\) are below 1.1. Stan allows users to specify initial 
values for parameters and it is also able to draw diffuse random 
initializations itself.
</p>

<p>
Details on the computatation of \(\hat{R}\) and some of its limitations can be 
found in the 'Markov Chain Monte Carlo Sampling' chapter of the
<a href="http://mc-stan.org/documentation/">
  Stan Modeling Language User's Guide and Reference Manual.</a>
<p>


*****

# NUTS/HMC

## HMC and NUTS (very briefly)

This is a very brief overview. For more details see the Stan manual and 
<a href="https://arxiv.org/abs/1701.02434"> Betancourt, M. (2017). A conceptual introduction to Hamiltonian Monte Carlo.</a>

### Hamiltonian Monte Carlo (HMC)
Hamiltonian Monte Carlo (HMC) is a Markov chain Monte Carlo (MCMC) method that
uses the derivatives of the density function being sampled to generate
efficient transitions spanning the posterior. It uses an approximate Hamiltonian 
dynamics simulation based on numerical integration which is then corrected by 
performing a Metropolis acceptance step.

<br><br>
<em>Algorithm summary</em>

The Hamiltonian Monte Carlo algorithm starts at a specified initial set of
parameters; in Stan, this value is either user-specified or generated
randomly. Then, for a given number of iterations, a new momentum vector is
sampled and the current value of the parameters is updated using the leapfrog
integrator with discretization time <code>stepsize</code> and number of 
steps <code>n_leapfrog</code> according to the Hamiltonian dynamics. 
Then a Metropolis acceptance step is applied, and a decision is made whether 
to update to the new state or keep the existing state.

### No-U-Turn Sampler (NUTS)

The no-U-turn sampler (NUTS) automatically selects an appropriate 
<code>n_leapfrog</code> in each iteration in order to allow the 
proposals to traverse the posterior without doing unnecessary work. 
The motivation is to maximize the expected squared jump distance 
(see, e.g., 
Roberts et al. (<a href = "http://www.jstor.org/stable/2245134">1997</a>)) 
at each step and avoid the random-walk behavior that arises in random-walk 
Metropolis or Gibbs samplers when there is correlation in the posterior. 
For a precise definition of the NUTS algorithm see Hoffman and Gelman 
(<a href = "http://arxiv.org/abs/1111.4246">2011</a>, 
<a href = "http://www.stat.columbia.edu/~gelman/research/published/nuts.pdf">2014</a>)

<br><br>
 <em>Algorithm summary</em> 

NUTS generates a proposal by starting at an initial position determined 
by the parameters drawn in the last iteration. It then generates an independent 
unit-normal random momentum vector. It then evolves the initial system both 
forwards and backwards in time to form a balanced binary tree. At each iteration 
of the NUTS algorithm the <code>treedepth</code> is increased by one, doubling 
<code>n_leapfrog</code> and effectively doubling the computation time. 
The algorithm terminates in one of two ways, either

<ul>
  <li>the NUTS criterion (i.e., a U-turn in Euclidean space on a subtree) 
  is satisfied for a new subtree or the completed tree, or</li>
  <li>the depth of the completed tree hits the maximum depth allowed.</li>
</ul>
 
Rather than using a standard Metropolis step, the final parameter value 
is selected via multinomial sampling among the Hamiltonian trajectories.

<br><br>
Configuring the no-U-turn sampler involves putting a cap on the <code>treedepth</code>
that it evaluates during each iteration. This is controlled through a maximum depth parameter. The number of leapfrog steps taken is then bounded by 2 to the power 
of the maximum depth minus 1.
<br><br>

<p>
For more details see 
<a href="https://arxiv.org/abs/1701.02434"> Betancourt, M. (2017). A conceptual introduction to Hamiltonian Monte Carlo.</a>
</p>


## Acceptance statistic 

<em>Quick definition</em>

The acceptance statistic used by NUTS for the Metropolis correction.   
In the original NUTS implementation a slice sampling step was used to sample a 
state from each Hamiltonian trajectory and <code>accept_stat</code> was the acceptance 
probability averaged over samples in the slice. In more recent versions of Stan
the NUTS algorithm uses multinomial sampling over the states for each Hamiltonian 
trajectory.

For HMC without NUTS <code>accept_stat</code> is the standard Metropolis 
acceptance probability.

<br><br>
<em>More details</em>

<p>
If the leapfrog integrator were perfect numerically, there would no need to do 
any more randomization per transition than generating a random momentum vector. 
Instead, what is done in practice to account for numerical errors during 
integration is to apply a Metropolis acceptance step. If the proposal is not 
accepted, the previous parameter value is returned for the next draw and used 
to initialize the next iteration.
</p>

<p>
By setting the target acceptance parameter to a value closer to 1 (its value
must be strictly less than 1 and its default value is 0.8), adaptation will be
forced to use smaller step sizes. This can improve sampling efficiency
(effective samples per iteration) at the cost of increased iteration times.
Raising the target will also allow some models that would otherwise get
stuck to overcome their blockages.
</p>

## Divergent

<em>Quick definition</em>

The number of leapfrog transitions with diverging error. Because NUTS terminates 
at the first divergence this will be either 0 or 1 for each iteration. 
The average value of <code>divergent</code> over all iterations is therefore 
the proportion of iterations with diverging error.

<br><br>
<em>More details</em>

<p>
When numerical issues arise during the evaluation of the parameter
Jacobians or the model log density, an exception is raised in the
underlying code and the current expansion of the Hamiltonian forward
and backward in time is halted. This is marked as a divergent
transition.
</p>

<p>
The primary cause of divergent transitions in Euclidean HMC (other
than bugs in the model code) is numerical instability in the leapfrog
integrator used to simulate the Hamiltonian evaluation. The
fundamental problem is that a fixed step size is being multiplied by
the gradient at a particular point, to determine the next simulated
point. If the stepsize is too large, this can overshoot into
ill-defined portions of the posterior.  
</p>

<p>
<strong>
If there are (post-warmup) divergences then the results may be biased and 
should not be used.
</strong>
</p>

<p>
In some cases, simply lowering the initial step size and increasing
the target acceptance rate will keep the step size small enough that
sampling can proceed.  
</p>

<p>
The exact cause of each divergent transition is printed as a warning 
message in the output console. This can be useful in cases where managing 
the step size is insufficient.  In such cases, a reparameterization is 
often required so that the posterior curvature is more manageable; 
see the section about Neal's Funnel in the Stan manual for an example.
</p>

For more details see 
<a href="https://arxiv.org/abs/1701.02434"> Betancourt, M. (2017). A conceptual introduction to Hamiltonian Monte Carlo.</a>

## Energy

<em>Quick definition</em>

The <code>energy</code> is the value of the Hamiltonian (up to an additive 
constant) at each sample.

<br><br>
<em>More details</em>

<p>
While divergences can identify light tails and incomplete exploration of the target distribution, the energy diagnostic can identify overly heavy tails that are also challenging for sampling. Informally, the energy diagnostic for HMC quantifies the heaviness of the tails of the posterior distribution. The energy diagostic plot 
shows overlaid histograms of the (centered) marginal energy distribution
and the first-differenced distribution. Keep an eye out for 
discrepancies between these distributions.
</p>

For more details see 
<a href="https://arxiv.org/abs/1701.02434"> Betancourt, M. (2017). A conceptual introduction to Hamiltonian Monte Carlo.</a>

## Step_size

<em>Quick definition</em>

The integrator step size used in the Hamiltonian simulation.

<br><br>
<em>More details</em>

<p>
All implementations of HMC use numerical integrators requiring a step size
(equivalently, discretization time interval).
</p>
<p>
If <code>step_size</code> is too large, the leapfrog integrator will be 
inaccurate and too many proposals will be rejected. If <code>step_size</code> 
is too small, too many small steps will be taken by the leapfrog integrator 
leading to long simulation times per interval. Thus the goal is to balance the 
acceptance rate between these extremes.
</p>

## Number of leapfrog

<em>Quick definition</em>

The number of leapfrog steps (calculations) taken during the 
Hamiltonian simulation.

<br><br>
<em>More details</em>

<p>
If <code>n_leapfrog</code> is too small, the trajectory traced out in each 
iteration will be too short and sampling will devolve to a random walk. If 
<code>n_leapfrog</code> is too large, the algorithm will do too much work 
on each iteration.
</p>

## Treedepth

<em>Quick definition</em>

The depth of tree used by NUTS.
  
<br><br>  
<em>More details</em>

<p>
Configuring NUTS involves putting a cap on the depth of the 
trees that it evaluates during each iteration. This is controlled through 
a maximum depth parameter. <code>n_leapfrog</code> is then bounded 
by 2 to the power of the maximum depth minus 1.
</p>

<p>
Tree depth is an important diagnostic tool for
NUTS. For example, a <code>treedepth = 0</code> occurs when the first leapfrog 
step is immediately rejected and the initial state returned, indicating extreme
curvature and poorly-chosen <code>stepsize</code> (at least relative to the 
current position). 
</p>

<p>
On the other hand, <code>treedepth = max_treedepth</code> equal to the maximum depth
indicates that NUTS is taking many leapfrog steps and being terminated
prematurely to avoid excessively long execution time. 
</p>

<p>
Taking very many steps may be a sign of poor adaptation, 
may be due to targeting a very high acceptance rate, 
or may simply indicate a difficult posterior from which to sample. 
In the latter case, reparameterization may help with efficiency. But
in the rare cases where the model is correctly specified and a large number of
steps is necessary, the maximum depth should be increased to ensure that that
the NUTS tree can grow as large as necessary.
</p>