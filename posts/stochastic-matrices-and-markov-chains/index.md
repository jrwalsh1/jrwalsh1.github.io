<!--
.. title: Stochastic Matrices and Markov Chains
.. slug: stochastic-matrices-and-markov-chains
.. date: 2015-08-14 00:55:26 UTC-07:00
.. tags: mathjax, draft
.. category:
.. link:
.. description:
.. type: text
-->

My own work has dealt a lot with Monte Carlo sampling: either for multi-dimensional integrals or for simulating processes.  Typically the systems I work on are dynamic, continuous systems rather than finite state systems.  The evolution operator in continuous systems is often cast as a probability distribution: one generates an event time by sampling according to some probability distribution -- a good example is radioactive decay, where the distribution is simply Poisson.  At the heart most Monte Carlo simulation problems I work on are Poission processes, which have lots of interesting analytic properties that may be derived by using unitarity.

There is a good analog with finite state systems in Markov processes.  In this case, the state of the system can be represented probabilistically by a vector:

$$
\vec{s} = \left( \begin{matrix} p_1 \\\ p_2 \\\ \vdots \\\ p_n \end{matrix} \right) \,, \quad \sum_i p_i = 1\,.
$$

Unitarity enforces the condition that the sum of the elements is 1.  We will label fixed states $i$ as $\vec{s}_i$, with $(\vec{s}_i)_{j} = \delta_{ij}.$  One step in the chain is a transition between states with a specific probability:

   * probability to transition from state $i$ to state $j$ is $p_{ij}$, with $\sum_j p_{ij} = 1$ (unitarity).

These transition probabilities can be put together into an evolution matrix, $U$,

<div> $$
U_{ji} = p_{ij}
$$ </div>
<br>

This is a matrix of nonnegative entries where sum of each column is 1 -- which is a left stochastic matrix.  For a right stochastic matrix, the rows sum to 1.  Several basic properties of stochastic matrices help us understand what happens in Markov processes.

Let's look at the eigenvectors and eigenvalues of $U$, which we label $\vec{v}_i$ and $\lambda_i$ respectively.  The eigenvalues of a matrix and its transpose are equal, so consider $U^{\rm T}$ (a right stochastic matrix); label its eigenvectors $\vec{w}_i$.  Then

<div> $$
U^{\rm T} \vec{w}_i = \lambda_i \vec{w}_i \,.
$$ </div>

If $\vec{u}^{\rm T}_j$ is a row vector of $U^{\rm T}$,

<div> $$
\vec{u}_j \cdot \vec{w}_i = \lambda_i \vec{w}_i
$$ </div>
