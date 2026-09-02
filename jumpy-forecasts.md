# Jumpy forecasts in a multi-out setup

> **Claim.** When a variable is harder to predict than the main variable(s) it depends on, this can lead to *jumpy* forecasts in a multi-out setup.

## Setup: a trivial prognostic variable

We'll illustrate this at the level of forecast error in a very simple toy model of a single prognostic and diagnostic variable. 

Consider discrete time $t \in \mathbb{N}$ and a variable $\phi$ with trivial dynamics:

$$\phi(t+1)=\phi(t)\,.$$
 
Still this can lead to non-trivial things if we have a non-trivial iterative scheme to step through the dynamics, that we assume to be an approximation and hence introduces an error.

### A non-trivial multi-step scheme

We introduce a non-trivial iterative scheme by uniquely splitting

$$t = (R-1)N + s\,, \qquad 1 \le s \le N\,,$$

where all quantities are integers (remainder theorem applied to $t-1$, i.e. $t-1 = qN + r$ with $0 \le r < N$, and $q = R-1$, $r = s-1$). Here $R$ is the **rollout** index and $s$ the **multi-out step**. 
so the dynamics read, in multi-step form,

$$\phi\left((R-1)N + s\right) = \phi\left((R-2)N + N\right)\,.$$

### Introducing approximation error

We now create approximate dynamics by introducing a small error. We assume two types: (a) one per rollout step (error interpreting the input), and (b) one per multi-out step $s$ (error forecasting a given amount of time ahead):

$$\tilde\phi\left((R-1)N + s\right) = \tilde\phi\left((R-2)N + N\right) + \epsilon_s + \zeta_R\,.$$

Here $\epsilon_s$ and $\zeta_R$ may a priori depend arbitrarily on $s$ and $R$. We consider the simple case where the input is perfectly interpreted ($\zeta_R = 0$) and the per-step error within a rollout window is linear, $\epsilon_s = s\epsilon$: 

$$\tilde\phi\left((R-1)N + s\right) = \tilde\phi\left((R-2)N + N\right) + s\epsilon\,.$$

Assuming exact initialisation, $\tilde\phi(0) = \phi(0)$, we obtain the following relation to the exact dynamics:

$$\tilde\phi\left((R-1)N + s\right) = \phi\left((R-1)N + s\right) + \left((R-1)N + s\right)\epsilon\,.$$

### Adding a diagnostic variable

Now introduce an extra, *diagnostic* variable $\chi$, whose value is not computed recursively from an earlier timestep but from $\phi$ at the same time: $\chi(t) = f(\phi(t), t)$. A trivial example is $\chi(t) = \phi(t)$; still, assuming a non-trivial recursive representation and a separate error-based approximation makes things interesting:

$$\chi\left((R-1)N + s\right) = \phi\left((R-2)N + N\right)\,,$$

$$\tilde\chi\left((R-1)N + s\right) = \tilde\phi\left((R-2)N + N\right) + \delta_s\,,$$

where $\delta_s$ is some $s$-dependent error. Taking it linear, $\delta_s = s\delta$\,, we get:

$$\tilde\chi\left((R-1)N + s\right) = \chi\left((R-1)N + s\right) + (R-1)N\epsilon + s\delta\,.$$

### Error growth and 'jumpiness'

This lets us read off the error growth in time. Defining $\Delta_\phi(t) = \tilde\phi(t) - \phi(t)$ and similarly for $\chi$:

$$\Delta_\phi\left((R-1)N + s\right) = \left((R-1)N + s\right)\epsilon\,,$$

$$\Delta_\chi\left((R-1)N + s\right) = (R-1)N\epsilon + s\delta\,.$$

The point is that $\Delta_\phi$ is monotonic (it equals $t\epsilon$), whereas $\Delta_\chi$ need not be — it can be *jumpy*. Even when $\epsilon$ and $\delta$ share the same sign (say positive), non-monotonic behaviour can appear. Indeed, for fixed $R$ the error is monotonic in $s$, so a jump can only occur between rollout windows, namely when the error on the last step in a rollout window is larger than the error on the first step of the next window:

$$\Delta_\chi\left((R-1)N + N\right) > \Delta_\chi(RN + 1)\,,$$

which is equivalent to

$$\delta > \frac{N}{N-1}\,\epsilon\,.$$

## Interactive explorer

The above is interactively visualized (tune N, epsilon, delta) at [jumpy-forecasts.html](https://dietervdb-meteo.github.io/notes/jumpy-forecasts.html).

<!--
## Appendix — a more non-trivial example (to be reintegrated later)

A variable with growth dynamics:

$$\phi(t+1) = (t+1)\phi(t), \qquad \phi(t) = t!\,\phi_0.$$
 We will now consider the following approximate N-out model for this variable. First we uniquely split t = (R - 1)  N + s   where 1 <= s <= N, here all are unique integers (via the remainder theorem for t-1, i.e. t-1 = q N + r, with 0 <= r < N and q = R -1, r = s - 1 ). 
We can then rewrite the above dynamics in a 'multi-step' fashion: [R is rollout, s is multi-out step]

$$\phi\left((R-1)N + s\right) = \phi\left((R-2)N + N\right)\prod_{k=1}^{s}\left((R-1)N + k\right)$$

```text

e.g. 
    N = 1 -> s = 1 and thus R = t + 2 so phi(t+1) = \phi(t) (t+1)
    N=2 
     t=1 -> R = 1, s = 1 so phi(1) = phi(0)  = 1! phi(0)
     t=2 -> R = 1, s = 2 so phi(2) = 2 phi(0)= 2! phi(0)
     t=3 -> R= 2,  s=1 so phi(3) = 3 phi(2) = 3! phi(0) 
     t=4 -> R = 2, s=2 so phi(4) = 12 phi (2) = 4! phi(0)
     ETC
   ETC
```
-->
