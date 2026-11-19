# Closing the Loop on AI-Assisted Delivery

*Why your tickets say done when the software isn't, and a control-loop fix.*

## The premise

**Run every delivery program as a closed control loop.** Fix an envelope: the full set of tickets slated for delivery, each tied to a written definition of done. Measure the real state of the software against it on a fixed rhythm, and name and measure the error that AI tools and haste add to reported progress. At scheduled points, correct course by publishing a new design together with the tests that will prove it.

Quality does not come from getting it right the first time. It comes from shrinking a measured gap at every pass.

## AI made us faster. It also made our dashboards lie.

With AI IDEs like Cursor, my teams merge more code per week than ever. Tickets close, demos go green, and the burndown looks healthy. Yet the platform is not proportionally closer to done.

The tools are not the problem. They raise output faster than they raise evidence. On one migration I mapped, about 280 tickets were tracked and 63% were closed. None of the 46 packages that actually defined delivery had started.

That gap between reported progress and real progress is what this article is about, and how to close it.

## Why the usual signals fail

Every common progress signal can read "done" while the software is not:

- **Ticket closure** measures paperwork, not behavior. A closed ticket says someone stopped working on it.
- **Merged is not live.** Code behind a switched-off feature flag passes every check and changes nothing for a customer.
- **One green screen.** A fix makes one report correct while another screen showing the same number still disagrees.
- **The false oracle.** "Make it match that screen" is a prompt AI handles very well. If that screen is wrong, you now have two wrong screens that agree.
- **Self-graded tests.** When the same hand, human or AI, writes both the code and its test, they agree by construction.

These errors are not random. They all lean the same way: toward looking finished. A bias with a direction is something you can measure and correct, but only if your process admits it exists.

## The envelope: run delivery as a closed loop

Control engineers face this problem daily. When a system has bias, they do not hope the error goes away. They measure the output, compute the error against a target, adjust, and measure again.

Software delivery maps onto that loop almost one to one:

| Control idea | In software delivery |
| --- | --- |
| Setpoint | The envelope: every ticket slated for delivery, each package with a written definition of done |
| Plant | Code, feature flags, data, and every screen that shows the result |
| Sensors | Evidence of behavior: end-to-end tests, parity checks, demos marked pass or fail, flag telemetry |
| Error | The gap between the envelope and what the sensors measure |
| Controller | The accountable technical leader, making course corrections |
| Actuator | The next packages, gates, rollbacks, staffing, retooling, or a new design |

The envelope is fixed in advance, and that matters. You cannot declare victory on whatever happened to finish. Progress is always measured against the same, written target.

![Delivery as a closed loop: the envelope and the AI margin of error feed the corrected gap; the course correction publishes the next design with its tests; the software is observed by sensors that feed back into the gap.](images/closed-loop.png)

The top row decides and acts; the bottom row measures. The dashed path is what makes the loop learn: each new design adds the tests that will judge it.

## Name the AI margin of error, then measure it

Because the bias has a direction, give it a name: the AI margin of error. The gap you steer on is not target minus reported progress. It is:

```math
\hat{\varepsilon}_k = r - y_k + \delta_{AI,k}, \qquad \delta_{AI,k} = y_k\,\hat{\delta}_k
```

The margin is not a guess. At each correction point, pick about 20 tickets marked Done at random, not chosen by the people who closed them. Run each against an end-to-end test. The share that fails is your measured margin.

Two details keep it honest:

- **Not measured is its own number.** A Done ticket with no test that covers it is reported separately, never as a pass. A high count is a coverage gap, not a clean result.
- **Read trends, not small differences.** With 20 samples the figure is coarse. Compare its direction across correction points.

This is not an argument against AI. AI is a powerful actuator. Its error simply has to be in the equation.

## The model, in equations

Every quantity below is computable from data a program already has: the envelope, the weekly sensor readings and the audit sample. Worked-example values are illustrative.

### Symbols

| Symbol | Meaning | Source |
| --- | --- | --- |
| $r$ | Setpoint: fraction of envelope packages meeting their definition of done ($r = 1$) | Fixed before the first correction |
| $N$ | Packages in the envelope | Envelope list |
| $k$ | Correction point: $k = 1, 2, 3$; acceptance at $k = 4$ | Calendar |
| $y_k$ | Reported progress: share of packages reported Done | Tracker and sensors |
| $y^{*}_k$ | True progress | Hidden; estimated below |
| $\delta_k$ | AI margin: probability that a reported-Done item fails its vertical test | Audit |
| $n, f, u$ | Audit sample size, failures, items with no covering test | Audit |
| $\hat{\varepsilon}_k$ | Corrected gap | Computed |
| $\tau$ | Acceptance tolerance on the gap | Agreed up front |

### 1. Reported progress, true progress and the corrected gap

```math
y_k = \frac{1}{N}\sum_{i=1}^{N} d_{i,k}, \qquad d_{i,k} = \begin{cases} 1 & \text{package } i \text{ reported Done at } k \\ 0 & \text{otherwise} \end{cases}
```

```math
y^{*}_k = y_k\,(1-\delta_k)
```

```math
\hat{\varepsilon}_k = r - y^{*}_k = \underbrace{(r - y_k)}_{\text{reported gap}} + \underbrace{y_k\,\hat{\delta}_k}_{\text{AI margin of error}}
```

The reported gap under-reads the true gap by exactly $y_k\,\hat{\delta}_k$. That term is the AI margin of error, counted once.

### 2. Estimating δ from the audit, with its interval

Point estimate, excluding unmeasured items from the denominator (they are reported separately, never as passes):

```math
\hat{\delta}_k = \frac{f}{m}, \qquad m = n - u
```

95% Wilson interval ($z = 1.96$):

```math
\hat{\delta}_{L,U} = \frac{\hat{\delta} + \dfrac{z^2}{2m} \mp z\sqrt{\dfrac{\hat{\delta}(1-\hat{\delta})}{m} + \dfrac{z^2}{4m^2}}}{1 + \dfrac{z^2}{m}}
```

Propagated to the gap ($\hat{\varepsilon}$ increases with $\delta$):

```math
\hat{\varepsilon}_k \in \Big[\, r - y_k\,(1-\hat{\delta}_L),\;\; r - y_k\,(1-\hat{\delta}_U) \,\Big]
```

Worked example: $y = 0.63$, $n = 20$, $f = 6$, $u = 0$.

| Quantity | Computation | Value |
| --- | --- | --- |
| Reported gap | 1 − 0.63 | 0.370 |
| $\hat{\delta}$ | 6 / 20 | 0.300 |
| 95% interval for $\delta$ | Wilson, $m = 20$ | [0.146, 0.519] |
| AI margin | 0.63 × 0.30 | 0.189 |
| True progress $y^{*}$ | 0.63 × 0.70 | 0.441, interval [0.303, 0.538] |
| Corrected gap $\hat{\varepsilon}$ | 1 − 0.441 | 0.559, interval [0.462, 0.697] |

The tracker says 37% remains. The corrected gap says 56%, and plausibly up to 70%.

### 3. Sizing the audit

Exact upper bound on $\delta$ when the audit finds zero failures in $n$ items, at confidence $1-\alpha$:

```math
p_U = 1 - \alpha^{1/n}
```

Sample size needed to demonstrate $\delta < p_0$ with zero failures:

```math
n \ge \frac{\ln \alpha}{\ln(1-p_0)}
```

General acceptance plan $(n, c)$, accepting when at most $c$ of $n$ fail. Its operating characteristic, producer's risk and consumer's risk:

```math
P_a(p) = \sum_{j=0}^{c} \binom{n}{j}\, p^{j}(1-p)^{n-j}, \qquad \alpha = 1 - P_a(p_{\text{AQL}}), \qquad \beta = P_a(p_{\text{LTPD}})
```

| Plan $n$ | Failures allowed $c$ | $P_a$ at $\delta$ = 2% | at 10% | at 20% |
| --- | --- | --- | --- | --- |
| 20 | 0 | 0.668 | 0.122 | 0.012 |
| 20 | 1 | 0.940 | 0.392 | 0.069 |
| 50 | 2 | 0.922 | 0.112 | 0.001 |

- 0 failures in 20 proves only $\delta < 10.9\%$ at 90% confidence, or $\delta < 13.9\%$ at 95%.
- 0 failures in 60 proves $\delta < 4.9\%$ at 95%.
- Demonstrating $\delta < 5\%$ at 95% needs $n \ge \ln 0.05 / \ln 0.95 = 58.4$, so 59 clean items.
- Plan (20, 1) accepts work with a 10% margin 39% of the time: fine for trend tracking, too weak for final acceptance.

### 4. The vertical suite and its ratchet

```math
T_k = f\big(\text{design}_k,\ \hat{\varepsilon}_k\big), \qquad S_k = \big(S_{k-1} \cup T_k\big) \setminus R_k, \qquad S_0 = \varnothing
```

$R_k$ is the set of tests retired at correction $k$, each with a written reason; $R_k = \varnothing$ after the final freeze.

```math
y^{\text{carry}}_k = \frac{\#\{\text{passing tests in } S_{k-1}\}}{|S_{k-1}|}, \qquad y^{\text{new}}_k = \frac{\#\{\text{passing tests in } T_k\}}{|T_k|}
```

```math
\hat{\varepsilon}^{\,\text{suite}}_k = 1 - y^{\text{crit}}_k, \qquad y^{\text{crit}}_k = \text{pass rate on UAT-critical tests in } \begin{cases} T_1 & k = 1 \\ S_{k-1} & k \ge 2 \end{cases}
```

Falling $y^{\text{carry}}$ means regression; rising $y^{\text{new}}$ means progress on the current design. Neither can mask the other.

### 5. Convergence and compounding

Contraction ratio of the loop at each correction:

```math
\rho_k = \frac{\hat{\varepsilon}_k}{\hat{\varepsilon}_{k-1}}
```

$\rho_k < 1$: the loop is closing the gap. Growth counts as real, not sampling noise, when the intervals separate:

```math
\hat{\varepsilon}^{\,L}_{k} > \hat{\varepsilon}^{\,U}_{k-1} \;\;\Longrightarrow\;\; \text{change the controller}
```

Corrections needed to reach $\tau$ at a constant ratio, and the ratio required to reach $\tau$ within $K$ corrections:

```math
k^{*} = \left\lceil \frac{\ln\!\left(\tau / \hat{\varepsilon}_0\right)}{\ln \rho} \right\rceil, \qquad \rho_{\text{req}} = \left(\frac{\tau}{\hat{\varepsilon}_0}\right)^{1/K}
```

Worked example: $\hat{\varepsilon}_0 = 0.559$, $\hat{\varepsilon}_1 = 0.42$, so $\rho = 0.75$. With $\tau = 0.05$, $k^{*} = \lceil \ln(0.0894)/\ln(0.75) \rceil = \lceil 8.44 \rceil = 9$ corrections.

With only $K = 3$ corrections scheduled, $\rho_{\text{req}} = (0.05/0.559)^{1/3} = 0.45$. Every correction must remove more than half of the remaining gap. Computed at the first correction point, this tells you early that either the controller or the scope has to change.

### 6. Reliability growth: why a growing gap indicts the process

[Jelinski–Moranda (1972)](https://ifs.host.cs.st-andrews.ac.uk/Books/SE9/Web/DepSecAssur/GrowthModels.html): with $N_0$ initial faults and per-fault rate $\phi$, the failure intensity after $i-1$ repairs is

```math
\lambda_i = \phi\,\big(N_0 - (i-1)\big)
```

A simple imperfect-repair extension: each fix introduces $q$ new faults on average. The expected residual after $m$ fixes is

```math
\mathbb{E}[R_m] = N_0 - m\,(1-q)
```

If $q \ge 1$, the residual does not fall however many fixes ship, and shipping more of them (raising $m$, the gain) cannot help. Changing the controller means lowering $q$ through policy, pattern and tests. Littlewood and Verrall (1973) formalize the same negative growth.

### 7. Kalman filter: true progress every week

State $x_k$ is true progress. Each reading is debiased before use:

```math
x_k = x_{k-1} + u_k + w_k,\;\; w_k \sim \mathcal{N}(0, Q) \qquad\qquad z_k = y_k\,(1-\hat{\delta}) = x_k + v_k,\;\; v_k \sim \mathcal{N}(0, R_k)
```

Predict, then update:

```math
\hat{x}^{-}_k = \hat{x}_{k-1} + u_k, \qquad P^{-}_k = P_{k-1} + Q
```

```math
K_k = \frac{P^{-}_k}{P^{-}_k + R_k}, \qquad \hat{x}_k = \hat{x}^{-}_k + K_k\,\big(z_k - \hat{x}^{-}_k\big), \qquad P_k = (1 - K_k)\,P^{-}_k
```

$R_k$ is large for a weekly reading and small in an audit week. Worked example: $\hat{x}_0 = 0.44$, $P_0 = 0.0049$, $Q = 0.0004$, planned $u = 0.02$ per week, weekly $R = 0.0036$, audit $R = 0.0009$.

| Week | Reading $z$ | $R$ | Gain $K$ | Estimate $\hat{x}$ | Std. dev. $\sqrt{P}$ |
| --- | --- | --- | --- | --- | --- |
| 1 | 0.47 | 0.0036 | 0.60 | 0.466 | 0.046 |
| 2 | 0.49 | 0.0036 | 0.41 | 0.488 | 0.039 |
| 3 (audit) | 0.50 | 0.0009 | 0.68 | 0.503 | 0.025 |

The audit week gets the highest gain and cuts the uncertainty most. [Kim and Reinschmidt (2010)](https://pure.psu.edu/en/publications/probabilistic-forecasting-of-project-duration-using-kalman-filter/) applied the same filter to earned-value forecasting.

### 8. Capture–recapture: problems nobody found

The vertical tests find $n_1$ problems, an independent reviewer finds $n_2$, and $m$ are found by both. Lincoln–Petersen and Chapman's bias-corrected estimates of the total, with Chapman's variance:

```math
\hat{N}_{LP} = \frac{n_1 n_2}{m}, \qquad \hat{N}_{C} = \frac{(n_1+1)(n_2+1)}{m+1} - 1, \qquad \widehat{\mathrm{Var}}\big(\hat{N}_C\big) = \frac{(n_1+1)(n_2+1)(n_1-m)(n_2-m)}{(m+1)^2\,(m+2)}
```

```math
\hat{N}_{\text{undetected}} = \hat{N} - (n_1 + n_2 - m)
```

Worked example: $n_1 = 8$, $n_2 = 6$, $m = 4$. $\hat{N}_{LP} = 12$ and $\hat{N}_C = 11.6$ (standard deviation 1.8). Ten distinct problems were found, so about 2 remain undetected. The estimate is valid only if the two checks are independent, which is what the different-hands rule enforces. See [Ahrefors et al. (2004)](https://lup.lub.lu.se/search/publication/282e2518-f7d2-49e0-bd9c-49934bcf9849) for its use in software inspection.

### 9. The acceptance criterion

```math
\text{Accept} \iff y^{\text{crit}}_4 = 1 \;\wedge\; \hat{\delta}_{U,4} \le \delta_{\max} \;\wedge\; \hat{\varepsilon}^{\,U}_4 \le \tau
```

The final audit is sized with section 3 so that $\hat{\delta}_U$ can actually fall below $\delta_{\max}$: to accept at $\delta_{\max} = 5\%$, at least 59 clean items.

## Every correction ships with the tests that prove it

Unit tests cannot see the failures that matter most here. Each layer passes on its own, while two screens still show different totals. The sensor that catches this is a **vertical test**: one business journey through every layer, checking that every screen and report agrees.

The key move is that the test set changes with each correction. Every new design is published together with the vertical tests that will judge it, written before the code ships. Six rules keep that honest:

1. **Ratchet, don't replace.** Tests carry forward from one correction to the next. Retiring one needs a written reason.
2. **Report two numbers.** The pass rate on carried-forward tests shows regression; the pass rate on new tests shows progress. A new design cannot hide a regression.
3. **Expected values come from written policy,** never from what the system returns today.
4. **Different hands.** Whoever implements a change, human or AI, is never the only author of its expected values.
5. **Keep it small.** Three to eight vertical journeys per correction, covering only what the new design touches.
6. **Failures at baseline are the measurement, not the defect.** A known failing test is honest error. A false pass is the real problem.

## Two clocks, never confused

The loop runs on two time scales over the same software. Mixing them up is a common way programs drift.

| Clock | Cadence | Owner | What it does |
| --- | --- | --- | --- |
| Sensing | Weekly | Team leads | Demos marked pass or fail, plus a run of the vertical tests. Early warning, no redesign |
| Course correction | Monthly, on fixed dates | Accountable technical leader | Re-measure the corrected gap, make the calls, publish the next design and its tests |
| Acceptance | Once, at the end | Accountable technical leader | Run the frozen test set. Accept or reject against an agreed tolerance. No new design |

On the migration above, that meant three correction points a month apart, then final acceptance. Anything outside tolerance at the end becomes a deferred item with a written policy. Nothing quietly disappears from scope.

## When the gap grows, change the controller, not the gain

Sometimes a correction makes the measured gap bigger. The instinct is to push the same fix harder: another patch to the one screen, another retry, another hotfix. In control terms, that is turning up the gain on a loop whose model is wrong.

The rule is simple. If the gap grew after a correction, change the controller before shipping more of the same. That can mean a different policy, a different design pattern, better sensors, different staffing or a smaller scope. Then publish the revised design and its tests.

One exception does not wait for the next correction point. Irreversible actions, such as switching a production data path or migrating customer money, need interlocks that act continuously. A monthly review that finds an irreversible mistake is too late.

## What it looked like on a real migration

The context was a platform migration done strangler-style, with billing totals, policy status and generated documents shared across many screens. The team used AI IDEs heavily. At the baseline measurement:

- About 280 tickets were tracked and 63% were closed. On a dashboard, that reads as two-thirds done.
- All 46 packages that defined delivery were still unstarted.
- The new data layer was merged in several waves, all switched off. No data type had yet been cut over, soaked and rolled back.
- A critical bug showed customers a different total on two screens. That is exactly the multi-screen disagreement vertical tests are built to catch.

The loop reframed the program in its first weeks. Ticket closure became a footnote and per-area evidence became the measure. The first set of vertical tests was built before the first correction point, so it could serve as the baseline. Switching on the new data layer in production was gated behind a non-production pilot and a timed rollback drill.

Results from the correction points themselves will be the subject of a follow-up.

## How to start next week

1. **Fix the envelope.** List every ticket slated for delivery and give each package a one-line written definition of done.
2. **Choose your sensors.** Decide what counts as evidence for each area, and demote ticket closure to a footnote.
3. **Put the clock on the calendar.** Schedule three or four correction points and one acceptance date now, not later.
4. **Build the baseline tests first.** Three to eight vertical journeys, built and run before the first correction point.
5. **Audit the margin.** At each correction point, test 20 random Done tickets and publish the share that fails.
6. **Correct, don't just review.** At each point: measure, decide, and publish the next design together with its tests.
7. **Gate what can't be undone.** Production cutovers and money paths wait for their pilot and rollback drill.
8. **Watch for compounding.** If the gap grew, change the approach before shipping more of the same.

## The point

AI IDEs raise the rate of change. Quality depends on whether your evidence keeps up with it.

The loop does not ask you to be right the first time. It asks for an honest measure, a named bias, and the discipline to correct on schedule. Ship each design together with the test that will judge it, and the gap closes pass by pass.
