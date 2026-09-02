# Jumpy forecasts in a multi-out setup

> **Claim.** When a variable is harder to predict than the main variable(s) it depends on, this can lead to *jumpy* forecasts in a multi-out setup.

## Setup: a trivial prognostic variable

We'll illustrate this at the level of forecast error in a very simple toy model of a single prognostic and diagnostic variable. 

Consider discrete time $t \in \mathbb{N}$ and a variable $\phi$ with trivial dynamics:

$$\phi(t+1)=\phi(t).$$
 
Still this can lead to non-trivial things if we have a non-trivial iterative scheme to step through the dynamics, that we assume to be an approximation and hence introduces an error.

### A non-trivial multi-step scheme

We introduce a non-trivial iterative scheme by uniquely splitting

$$t = (R-1)\,N + s, \qquad 1 \le s \le N,$$

where all quantities are integers (remainder theorem applied to $t-1$: $t-1 = qN + r$ with $0 \le r < N$, and $q = R-1$, $r = s-1$). Here $R$ is the **rollout** index and $s$ the **multi-out step**. 
so the dynamics read, in multi-step form,

$$\phi\big((R-1)N + s\big) = \phi\big((R-2)N + N\big).$$

### Introducing approximation error

We now create approximate dynamics by introducing a small error. We assume two types: (a) one per rollout step (error interpreting the input), and (b) one per multi-out step $s$ (error forecasting a given amount of time ahead):

$$\tilde\phi\big((R-1)N + s\big) = \tilde\phi\big((R-2)N + N\big) + \epsilon_s + \zeta_R.$$

Here $\epsilon_s$ and $\zeta_R$ may a priori depend arbitrarily on $s$ and $R$. We consider the simple case where the input is perfectly interpreted ($\zeta_R = 0$) and the per-step error within a rollout window is linear, $\epsilon_s = s\epsilon$: 

$$\tilde\phi\big((R-1)N + s\big) = \tilde\phi\big((R-2)N + N\big) + s\epsilon.$$

Assuming exact initialisation, $\tilde\phi(0) = \phi(0)$, we obtain the following relation to the exact dynamics:

$$\tilde\phi\big((R-1)N + s\big) = \phi\big((R-1)N + s\big) + \big((R-1)N + s\big)\epsilon.$$

### Adding a diagnostic variable

Now introduce an extra, *diagnostic* variable $\chi$, whose value is not computed recursively from an earlier timestep but from $\phi$ at the same time: $\chi(t) = f(\phi(t), t)$. A trivial example is $\chi(t) = \phi(t)$; still, assuming a non-trivial recursive representation and a separate error-based approximation makes things interesting:

$$\chi\big((R-1)N + s\big) = \phi\big((R-2)N + N\big),$$

$$\tilde\chi\big((R-1)N + s\big) = \tilde\phi\big((R-2)N + N\big) + \delta_s,$$

where $\delta_s$ is some $s$-dependent error. Taking it linear, $\delta_s = s\delta$, we get:

$$\tilde\chi\big((R-1)N + s\big) = \chi\big((R-1)N + s\big) + (R-1)N\epsilon + s\delta.$$

### Error growth and 'jumpiness'

This lets us read off the error growth in time. Defining $\Delta_\phi(t) = \tilde\phi(t) - \phi(t)$ and similarly for $\chi$:

$$\Delta_\phi\big((R-1)N + s\big) = \big((R-1)N + s\big)\epsilon,$$

$$\Delta_\chi\big((R-1)N + s\big) = (R-1)N\epsilon + s\delta.$$

The point is that $\Delta_\phi$ is monotonic (it equals $t\epsilon$), whereas $\Delta_\chi$ need not be — it can be *jumpy*. Even when $\epsilon$ and $\delta$ share the same sign (say positive), non-monotonic behaviour can appear. Indeed, for fixed $R$ the error is monotonic in $s$, so a jump can only occur between rollout windows, namely when

$$\Delta_\chi\big((R-1)N + N\big) > \Delta_\chi(RN + 1),$$

which is equivalent to

$$\delta > \frac{N}{N-1}\,\epsilon.$$

## Interactive explorer

An interactive version (tune N, epsilon, delta) lives in [jumpy-forecasts.html](jumpy-forecasts.html).
The `<iframe>` below renders it in a real browser or when this file is exported/served as HTML;
in GitHub or the VS Code Markdown preview the script is sanitized, so use the link above instead.

<iframe src="jumpy-forecasts.html" width="100%" height="520" style="border:1px solid #2a3150; border-radius:12px;" title="Jumpy forecasts error growth explorer"></iframe>

### Vega-Lite version (for HedgeDoc)

HedgeDoc renders Vega-Lite inside a fenced ` ```vega-lite ` block (via vega-embed), and the slider *bindings* below stay interactive — so this works inline, with no external file or `<script>`. In the VS Code preview it just shows as a code block; paste it into a HedgeDoc note to get the live sliders. (If your instance only knows the ` ```vega ` keyword, rename the fence — vega-embed detects Vega-Lite from `$schema`.)

```vega-lite
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "description": "Error growth: prognostic vs diagnostic in a multi-out rollout.",
  "width": 640,
  "height": 360,
  "params": [
    {"name": "N",   "value": 2,   "bind": {"input": "range", "min": 1, "max": 12, "step": 1,    "name": "N (multi-out steps): "}},
    {"name": "eps", "value": 0.1, "bind": {"input": "range", "min": 0, "max": 1,  "step": 0.01, "name": "epsilon (per-step error): "}},
    {"name": "del", "value": 0.4, "bind": {"input": "range", "min": 0, "max": 1,  "step": 0.01, "name": "delta (diagnostic error): "}}
  ],
  "layer": [
    {
      "data": {"sequence": {"start": 1, "stop": 49, "step": 1, "as": "tb"}},
      "transform": [{"filter": "(datum.tb % N) == 0"}],
      "mark": {"type": "rule", "strokeDash": [4, 4], "color": "#9aa3c0", "opacity": 0.5},
      "encoding": {"x": {"field": "tb", "type": "quantitative"}}
    },
    {
      "data": {"sequence": {"start": 1, "stop": 49, "step": 1, "as": "t"}},
      "transform": [
        {"calculate": "floor((datum.t - 1) / N) + 1", "as": "R"},
        {"calculate": "((datum.t - 1) % N) + 1", "as": "s"},
        {"calculate": "datum.t * eps", "as": "dphi"},
        {"calculate": "(datum.R - 1) * N * eps + datum.s * del", "as": "dchi"},
        {"fold": ["dphi", "dchi"], "as": ["series", "error"]}
      ],
      "mark": {"type": "line", "point": true},
      "encoding": {
        "x": {"field": "t", "type": "quantitative", "title": "t"},
        "y": {"field": "error", "type": "quantitative", "title": "cumulative error"},
        "color": {
          "field": "series", "type": "nominal", "title": null,
          "scale": {"domain": ["dphi", "dchi"], "range": ["#4c78a8", "#f58518"]},
          "legend": {"labelExpr": "datum.label == 'dphi' ? 'Delta_phi (prognostic)' : 'Delta_chi (diagnostic)'"}
        },
        "tooltip": [
          {"field": "t", "type": "quantitative"},
          {"field": "series", "type": "nominal"},
          {"field": "error", "type": "quantitative", "format": ".3f"}
        ]
      }
    }
  ]
}
```

---

## Appendix — a more non-trivial example (to be reintegrated later)

A variable with growth dynamics:

$$\phi(t+1) = (t+1)\phi(t), \qquad \phi(t) = t!\,\phi_0.$$
 We will now consider the following approximate N-out model for this variable. First we uniquely split t = (R - 1)  N + s   where 1 <= s <= N, here all are unique integers (via the remainder theorem for t-1, i.e. t-1 = q N + r, with 0 <= r < N and q = R -1, r = s - 1 ). 
We can then rewrite the above dynamics in a 'multi-step' fashion: [R is rollout, s is multi-out step]

$$\phi\big((R-1)N + s\big) = \phi\big((R-2)N + N\big)\prod_{k=1}^{s}\big((R-1)N + k\big)$$

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