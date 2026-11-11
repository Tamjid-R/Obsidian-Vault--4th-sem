# Maths 4543 — Numerical Methods Notes
**Course:** Math 4543, Islamic University of Technology (IUT)  
**Instructor:** Syed Rifat Raiyan, Department of CSE  
**Textbook:** *Numerical Methods with Applications* — Prof. Dr. Autar Kaw  
🔗 http://nm.mathforcollege.com  
📺 YouTube: `numericalmethodsguy`

---

> **How to use these notes:** Read the concept, then the intuition, then the example. After that, try the trial questions at the end without looking at the notes. If you can do that, you truly understand it.

---

## Table of Contents
- [[#1. What Are Numerical Methods?]]
- [[#2. Analytical vs. Numerical Solutions]]
- [[#3. Error Quantification — How Do We Know We're Close Enough?]]
- [[#4. Why Do We Even Need Numerical Methods?]]
- [[#5. The Bisection Method]]
  - [[#5.1 The Core Idea — Intermediate Value Theorem]]
  - [[#5.2 The Algorithm Step-by-Step]]
  - [[#5.3 Worked Example 1 — Solving x³ = 20]]
  - [[#5.4 Worked Example 2 — The Floating Ball Problem]]
  - [[#5.5 Significant Digits Criterion (Scarborough's Criterion)]]
  - [[#5.6 Advantages of Bisection]]
  - [[#5.7 Drawbacks / Pitfalls of Bisection]]
- [[#6. Trial Questions]]

---

## 1. What Are Numerical Methods?

### Definition
Numerical methods are **systematic techniques for approximating** the solutions to mathematical problems that are either impossible or impractical to solve exactly (analytically).

### The Engineering Problem-Solving Pipeline

$$\text{Real Problem} \longrightarrow \text{Mathematical Model} \longrightarrow \underbrace{\text{Solve the Model}}_{\text{← Numerical Methods live here}} \longrightarrow \text{Use the Solution}$$

### 🧠 Intuition (Think of It This Way)
Imagine you need to find the exact centre of a forest. Analytically, you'd need a perfect map and a ruler. Numerically, you'd walk in a straight line, check if you've gone too far or not far enough, then **keep halving your search area** until you're "close enough." That's the essence of numerical methods — *organised trial and error with guaranteed improvement.*

---

## 2. Analytical vs. Numerical Solutions

### The Classic Example: Quadratic Equation $ax^2 + bx + c = 0$

| Property | Analytical Solution | Numerical Solution |
|---|---|---|
| **Formula** | $x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$ | Iterative (e.g., Bisection) |
| **Result** | Exact | Approximate |
| **Effort** | Symbolic manipulation | Repeated simple calculations |
| **Works for** | Degree-2 polynomials easily | Almost any well-behaved function |
| **Limitation** | No formula exists for degree ≥ 5 polynomials | Never perfectly exact |

### 🧠 Intuition
The quadratic formula is a recipe for degree-2 problems. But what if the equation is $x^5 - 3x^3 + e^x = 0$? There's **no recipe** — no one has ever found a general formula. Numerical methods are your backup plan when the recipe book runs out.

---

## 3. Error Quantification — How Do We Know We're Close Enough?

When you're iterating, you need to know **when to stop**. You use errors as your stopping signal.

### 3.1 Percentage Absolute Relative Approximate Error ($|\epsilon_a|$)

$$\boxed{|\epsilon_a| = \left| \frac{x_m^{\text{new}} - x_m^{\text{old}}}{x_m^{\text{new}}} \right| \times 100\%}$$

> **Note:** $x_m$ here is the **same midpoint** from the Bisection Method — $x_m = \frac{x_l + x_u}{2}$, where:
> - $x_l$ = **lower bound** of the current bracket (the left edge of your search window)
> - $x_u$ = **upper bound** of the current bracket (the right edge of your search window)
> - $x_m$ = the **midpoint** between them = your current best guess for the root
>
> $x_m^{\text{new}}$ is the midpoint from the *current* iteration, and $x_m^{\text{old}}$ is the midpoint from the *previous* iteration. You're just measuring how much your guess shifted between two consecutive steps.

- Compares the **current estimate to the previous estimate**.
- You don't need to know the true answer — you only need two consecutive estimates.
- When this number is small, your answer isn't changing much → you're converging.

#### How Do We Use It?

You compute $|\epsilon_a|$ **after every iteration** and compare it to your tolerance $\epsilon_s$:
- If $|\epsilon_a| > \epsilon_s$ → **keep going**, the answer is still changing too much
- If $|\epsilon_a| \leq \epsilon_s$ → **stop**, your answer is stable enough

#### Mini Example — Using the $x^3 = 20$ problem

Recall from Worked Example 1 ($f(x) = x^3 - 20 = 0$, bracket $[1, 4]$):

| Iteration | $x_m$ (midpoint / current guess) |
|:---:|:---:|
| 1 | $x_m^{\text{old}} = 2.5$ |
| 2 | $x_m^{\text{new}} = 3.25$ |

Plug into the formula:

$$|\epsilon_a| = \left| \frac{3.25 - 2.5}{3.25} \right| \times 100\% = \left| \frac{0.75}{3.25} \right| \times 100\% \approx 23.08\%$$

**What this tells you:** Your guess moved by 23% of its current value between iteration 1 and 2. That's a big jump — the answer is still changing a lot, so you keep iterating.

After the next iteration, $x_m$ goes from $3.25$ to $2.875$:

$$|\epsilon_a| = \left| \frac{2.875 - 3.25}{2.875} \right| \times 100\% \approx 13.04\%$$

Still too large. You keep going until $|\epsilon_a|$ drops below your chosen $\epsilon_s$ (e.g., $5\%$ or $0.5\%$).

> **Key insight:** You never need to know the true answer ($\sqrt[3]{20} = 2.7144...$) to use this formula. The formula is *self-referential* — it only looks at how much your own guesses are changing.

### 3.2 True Error and Relative True Error

$$E_t = \text{True value} - \text{Approximate value}$$
$$\epsilon_t = \frac{E_t}{\text{True value}} \times 100\%$$

- Use this only when you **already know** the true answer (e.g., in textbook verification).
- In real-world problems, you never know the true value — that's why you use $|\epsilon_a|$ instead.

### 3.3 Stopping Criteria & Tolerance ($\epsilon_s$)

In numerical iterations, you stop calculating when:
$$\boxed{|\epsilon_a| \leq \epsilon_s}$$

where:
- $|\epsilon_a|$ is the **percentage absolute relative approximate error** computed at the current iteration:
  $$|\epsilon_a| = \left| \frac{x^{\text{new}} - x^{\text{old}}}{x^{\text{new}}} \right| \times 100\%$$
- $\epsilon_s$ is the **pre-specified tolerance** (stopping criterion threshold), expressed as a percentage.
- Alternatively, iterations terminate if the safety limit `max_iter` is exceeded to prevent infinite looping.

#### What is the Tolerance ($\epsilon_s$)?
The tolerance $\epsilon_s$ is a **pre-set numerical boundary** defining the acceptable margin of uncertainty. It answers the question: *"How small must the relative change between consecutive approximations become before the solution is considered converged?"*
- It is determined **before** running the algorithm, either directly by engineering requirements (e.g., $\epsilon_s = 1\%$, $\epsilon_s = 0.1\%$) or derived mathematically using **Scarborough's Criterion** to guarantee a specific number of correct significant figures ($m$ digits):
  $$\epsilon_s = 0.5 \times 10^{2-m}\%$$

#### Does the Tolerance ($\epsilon_s$) Stay the Same?
**Yes, the tolerance $\epsilon_s$ remains strictly constant throughout all iterations.**

It does **not** change from step to step:
1. **$\epsilon_s$ is a fixed benchmark (the target):** It represents the fixed finish line set prior to iteration 1.
2. **$|\epsilon_a|$ is a dynamic variable (the measurement):** Unlike $\epsilon_s$, the approximate error $|\epsilon_a|$ is recomputed after every single iteration. As iterations progress and the estimates converge towards the true root, $|\epsilon_a|$ typically decreases.
3. **Comparison at each step:** At iteration $i$, the algorithm evaluates the check `if |ε_a| <= ε_s`. The threshold $\epsilon_s$ on the right-hand side stays constant, while $|\epsilon_a|$ on the left-hand side drops until the inequality is satisfied.

| Property | Tolerance ($\epsilon_s$) | Absolute Relative Approx. Error ($|\epsilon_a|$) |
|---|---|---|
| **Value Nature** | **Constant** (Fixed throughout all iterations) | **Dynamic** (Recalculated every iteration) |
| **Origin** | Set a priori by the user / specifications | Computed from current and previous estimates ($x^{\text{new}}, x^{\text{old}}$) |
| **Role** | Acceptance threshold / stopping criterion | Progress indicator / error quantification |
| **Equation Side** | Right-hand side of $|\epsilon_a| \leq \epsilon_s$ | Left-hand side of $|\epsilon_a| \leq \epsilon_s$ |

#### Example: Fixed Tolerance vs. Evolving Error
Consider finding the root of $f(x) = x^2 - 5 = 0$ using bisection on $[2, 3]$ with a target of **2 significant digits**.

1. **Calculate Fixed Tolerance ($\epsilon_s$):**
   Using Scarborough's Criterion for $m = 2$:
   $$\epsilon_s = 0.5 \times 10^{2-2}\% = 0.5 \times 10^0\% = 0.5\%$$
   This tolerance $\epsilon_s = 0.5\%$ remains **fixed** for every iteration.

2. **Iteration Tracking Table:**

| Iteration ($i$) | $x^{\text{old}}$ | $x^{\text{new}}$ | $|\epsilon_a| = \left|\frac{x^{\text{new}}-x^{\text{old}}}{x^{\text{new}}}\right|\times 100\%$ | Tolerance $\epsilon_s$ | Status ($|\epsilon_a| \leq \epsilon_s$) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | — | 2.5000 | — | $0.5\%$ | Initial step (N/A) |
| 2 | 2.5000 | 2.2500 | $11.11\%$ | $\mathbf{0.5\%}$ | $11.11\% > 0.5\%$ → Continue |
| 3 | 2.2500 | 2.1250 | $5.88\%$ | $\mathbf{0.5\%}$ | $5.88\% > 0.5\%$ → Continue |
| 4 | 2.1250 | 2.1875 | $2.86\%$ | $\mathbf{0.5\%}$ | $2.86\% > 0.5\%$ → Continue |
| 5 | 2.1875 | 2.2188 | $1.41\%$ | $\mathbf{0.5\%}$ | $1.41\% > 0.5\%$ → Continue |
| 6 | 2.2188 | 2.2344 | $0.70\%$ | $\mathbf{0.5\%}$ | $0.70\% > 0.5\%$ → Continue |
| 7 | 2.2344 | 2.2422 | $\mathbf{0.35\%}$ | $\mathbf{0.5\%}$ | $\mathbf{0.35\% \leq 0.5\% \implies \text{STOP}}$ |

As observed, $\epsilon_s$ stayed at $0.5\%$ across all 7 iterations, while $|\epsilon_a|$ updated dynamically from $11.11\% \to 5.88\% \to \dots \to 0.35\%$, at which point the stopping criterion was met.

### 3.4 Significant Digits Criterion (Scarborough's Criterion)

$$\text{If } |\epsilon_a| \leq 0.5 \times 10^{2-m}\% \text{, then at least } m \text{ significant digits are correct.}$$

| Desired Sig. Digits ($m$) | Required $\lvert \epsilon_a \rvert$ |
| :-----------------------: | :---------------------------------: |
|             1             |             $\leq 5\%$              |
|             2             |            $\leq 0.5\%$             |
|             3             |            $\leq 0.05\%$            |
|             4             |           $\leq 0.005\%$            |

### 🧠 Intuition
Think of it like a GPS. When the "recalculating" message shows a big reroute — you're still converging. When you see tiny sub-metre adjustments — you've essentially arrived. The $|\epsilon_a|$ is how "big" the recalculation step is. Once it's tiny enough, you declare arrival.

---

## 4. Why Do We Even Need Numerical Methods?

Some problems are **analytically impossible**:

### Reason 1 — Integrals with No Closed Form
The standard normal cumulative distribution (used everywhere in statistics and probability):

$$\Phi(x) = \frac{1}{\sqrt{2\pi}} \int_{-\infty}^{x} e^{-t^2/2} \, dt$$

There is **no antiderivative** for $e^{-t^2/2}$. This integral **cannot** be evaluated with pen and paper — you must use numerical integration.

### Reason 2 — Large Systems of Equations
Solving **1000 simultaneous linear equations** with 1000 unknowns by hand is analytically possible but computationally infeasible. Numerical algorithms (like Gaussian elimination or LU decomposition) make it manageable.

### 🧠 Intuition
It's like asking "what is $\pi$ exactly?" You can never write it out fully — but you can use enough decimal places (3.14159...) to build a bridge. Numerical methods give you **enough precision to be useful**, even when perfection is impossible.

---

## 5. The Bisection Method

**Also known as:** Binary Search Method  
**Type:** Bracketing Method (Root-Finding)  
**Goal:** Find $x$ such that $f(x) = 0$

---

### 5.1 The Core Idea — Intermediate Value Theorem

#### The Theorem
> If $f(x)$ is **continuous** on $[x_l, x_u]$, and $f(x_l) \cdot f(x_u) < 0$ (opposite signs), then there is **at least one root** between $x_l$ and $x_u$.

$$f(x_l) \cdot f(x_u) < 0 \implies \exists \, c \in [x_l, x_u] \text{ such that } f(c) = 0$$

#### Sign Analysis

| Product $f(x_l) \cdot f(x_u)$ | Conclusion |
|:---:|---|
| $< 0$ | ✅ At least one root guaranteed (odd number of roots) |
| $> 0$ | ❌ May or may not have roots (even number or zero roots) |
| $= 0$ | $x_l$ or $x_u$ is itself a root |

#### ⚠️ Critical Requirement: CONTINUITY
The theorem **only works if $f(x)$ is continuous** on the interval. If $f(x)$ has a jump discontinuity or vertical asymptote between $x_l$ and $x_u$, the sign change does NOT guarantee a root. The function may appear to change sign across the discontinuity without ever crossing zero.

### 🧠 Intuition
Imagine you're on the ground floor (negative altitude) and a friend is on the roof (positive altitude). Somewhere between you, someone must be at ground level (altitude = 0) — **as long as the building has a continuous staircase** (no teleportation). If the building has a broken staircase (discontinuity), the argument fails.

---

### 5.2 The Algorithm Step-by-Step

```
BISECTION METHOD ALGORITHM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
INPUT:  f(x), x_l, x_u, ε_s (tolerance), max_iter
OUTPUT: Approximate root x_m

STEP 1: Validate bracket
        Check that f(x_l) * f(x_u) < 0
        If not → Invalid bracket, choose different x_l and x_u

STEP 2: Compute midpoint (initial estimate)
        x_m = (x_l + x_u) / 2

STEP 3: Evaluate which half contains the root
        IF f(x_l) * f(x_m) < 0 → root is in LEFT half [x_l, x_m]
             → set x_u = x_m
        IF f(x_l) * f(x_m) > 0 → root is in RIGHT half [x_m, x_u]
             → set x_l = x_m
        IF f(x_l) * f(x_m) = 0 → x_m IS the root → STOP

STEP 4: Compute NEW midpoint
        x_m_new = (x_l + x_u) / 2

STEP 5: Compute approximate error
        |ε_a| = |x_m_new - x_m_old| / |x_m_new| × 100%

STEP 6: Check stopping condition
        IF |ε_a| > ε_s AND iterations < max_iter → Go to STEP 3
        ELSE → STOP; report x_m_new as the root
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 🧠 Intuition — The Phone Book Analogy
You're looking for "Smith" in a phone book. You open to the middle. If Smith comes after the middle, you throw away the first half. Open to the middle of the remaining half again. Repeat. Each step **halves your search space**. That's bisection — guaranteed to find the answer, just not the fastest way.

---

### 5.3 Worked Example 1 — Solving $x^3 = 20$

**Problem:** Find $x$ such that $x^3 = 20$, using $x_l = 1$, $x_u = 4$.  
**Rewrite:** $f(x) = x^3 - 20 = 0$

#### Step 0: Validate the Bracket

$$f(1) = 1^3 - 20 = -19 \quad (\text{negative})$$
$$f(4) = 4^3 - 20 = 44 \quad (\text{positive})$$
$$f(1) \cdot f(4) = (-19)(44) = -836 < 0 \quad ✅ \text{ Valid bracket!}$$

#### Iteration 1

| Variable | Value |
|---|---|
| $x_l$ | 1 |
| $x_u$ | 4 |
| $x_m$ | $\frac{1 + 4}{2} = 2.5$ |
| $|\epsilon_a|$ | N/A (first iteration, no previous estimate) |

#### Iteration 2

$$f(2.5) = (2.5)^3 - 20 = 15.625 - 20 = -4.375$$
$$f(x_l) \cdot f(x_m) = f(1) \cdot f(2.5) = (-19)(-4.375) = +83.125 > 0$$

Root is in the **right half** $[2.5, 4]$ → Set $x_l = 2.5$, $x_u = 4$

$$x_m^{\text{new}} = \frac{2.5 + 4}{2} = 3.25$$

$$|\epsilon_a| = \left| \frac{3.25 - 2.5}{3.25} \right| \times 100\% = \frac{0.75}{3.25} \times 100\% \approx \mathbf{23.08\%}$$

Since $23.08\% > 5\%$, **0 significant digits** are guaranteed correct.

#### Iteration 3

$$f(3.25) = (3.25)^3 - 20 = 34.3281 - 20 = 14.3281$$
$$f(x_l) \cdot f(x_m) = f(2.5) \cdot f(3.25) = (-4.375)(14.3281) < 0$$

Root is in the **left half** $[2.5, 3.25]$ → Set $x_l = 2.5$, $x_u = 3.25$

$$x_m^{\text{new}} = \frac{2.5 + 3.25}{2} = 2.875$$

$$|\epsilon_a| = \left| \frac{2.875 - 3.25}{2.875} \right| \times 100\% = \frac{0.375}{2.875} \times 100\% \approx \mathbf{13.04\%}$$

Since $13.04\% > 5\%$, **0 significant digits** are guaranteed correct.

> **True value:** $x = \sqrt[3]{20} \approx 2.7144$. Our estimate of $2.875$ after 3 iterations has a true relative error of about $5.9\%$. More iterations would tighten this.

---

### 5.4 Worked Example 2 — The Floating Ball Problem (Real-World!)

**Problem:** A plastic ball floats in water. Given:
- Specific gravity $\gamma = 0.6$
- Radius $R = 5.5 \text{ cm} = 0.055 \text{ m}$

The depth $x$ (in metres) to which the ball is submerged satisfies:

$$f(x) = x^3 - 0.165x^2 + 3.993 \times 10^{-4} = 0$$

**Physical bounds:** $0 \leq x \leq 2R = 0.11 \text{ m}$  
Use $x_l = 0$, $x_u = 0.11$.

#### Bracket Validation

$$f(0) = 0 - 0 + 3.993 \times 10^{-4} = +3.993 \times 10^{-4} > 0$$
$$f(0.11) = (0.11)^3 - 0.165(0.11)^2 + 3.993 \times 10^{-4} = -2.662 \times 10^{-4} < 0$$
$$f(0) \cdot f(0.11) < 0 \quad ✅$$

#### Full Iteration Table (10 Iterations)

| Iter | $x_l$ | $x_u$ | $x_m$ | $|\epsilon_a|$ (%) | $f(x_m)$ |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 0.00000 | 0.11000 | 0.05500 | — | $+6.655 \times 10^{-5}$ |
| 2 | 0.05500 | 0.11000 | 0.08250 | 33.33% | $-1.622 \times 10^{-4}$ |
| 3 | 0.05500 | 0.08250 | 0.06875 | 20.00% | $-5.563 \times 10^{-5}$ |
| 4 | 0.05500 | 0.06875 | 0.06188 | 11.11% | $+4.484 \times 10^{-6}$ |
| 5 | 0.06188 | 0.06875 | 0.06531 | 5.263% | $-2.593 \times 10^{-5}$ |
| 6 | 0.06188 | 0.06531 | 0.06359 | 2.702% | $-1.080 \times 10^{-5}$ |
| 7 | 0.06188 | 0.06359 | 0.06273 | 1.370% | $-3.176 \times 10^{-6}$ |
| 8 | 0.06188 | 0.06273 | 0.06230 | 0.690% | $+6.497 \times 10^{-7}$ |
| 9 | 0.06230 | 0.06273 | 0.06252 | 0.344% | $-1.265 \times 10^{-6}$ |
| 10 | 0.06230 | 0.06252 | 0.06241 | 0.172% | $-3.077 \times 10^{-7}$ |

**Answer after 10 iterations:** $x \approx 0.06241 \text{ m} \approx 6.241 \text{ cm}$

The ball is submerged to about **6.24 cm** (just over half the radius — makes sense for something with specific gravity 0.6, which is lighter than water!).

---

### 5.5 Significant Digits Criterion Deep Dive

After Iteration 10, $|\epsilon_a| = 0.172\%$. How many significant digits are guaranteed?

Use Scarborough's Criterion:

$$|\epsilon_a| \leq 0.5 \times 10^{2-m}\%$$

$$0.172 \leq 0.5 \times 10^{2-m}$$

Divide both sides by 0.5:

$$0.344 \leq 10^{2-m}$$

Take $\log_{10}$ of both sides:

$$\log_{10}(0.344) \leq 2 - m$$

$$-0.4634 \leq 2 - m$$

$$m \leq 2 + 0.4634 = 2.4634$$

Largest integer: $m = \mathbf{2}$

✅ After 10 iterations, the answer $x = 0.06241 \text{ m}$ is **guaranteed to have 2 correct significant digits**.

---

### 5.6 Advantages of the Bisection Method

| Advantage | Explanation |
|---|---|
| **Always Convergent** | Since the root is always bracketed, the method can never "miss" the root and diverge. |
| **Predictable Error Bound** | After $n$ iterations, the maximum possible error is $\frac{x_u - x_l}{2^n}$. You can calculate in advance how many iterations you need. |
| **Simple to Implement** | Only requires evaluating $f(x)$ — no derivatives needed. |

#### How Many Iterations Do You Need? (Pre-Calculation)

$$n \geq \frac{\log\left(\frac{x_u - x_l}{\epsilon}\right)}{\log(2)}$$

where $\epsilon$ is your desired absolute error tolerance.

---

### 5.7 Drawbacks / Pitfalls of the Bisection Method

#### Pitfall 1 — Slow Convergence (Linear Rate)

Each iteration only **halves** the interval. This is called **linear convergence** with rate $\frac{1}{2}$. Other methods (like Newton-Raphson) can converge **quadratically** (doubling correct digits each step) — much faster.

#### Pitfall 2 — Insensitive to "How Close" You Are

Even if $x_l$ is already extremely close to the root and $x_u$ is far away, bisection doesn't take advantage of that. It mechanically picks the midpoint every time.

#### Pitfall 3 — Cannot Handle Roots That Touch the Axis

Consider $f(x) = x^2 = 0$. The root is $x = 0$, but:

$$f(-1) = 1 > 0, \quad f(1) = 1 > 0$$
$$f(-1) \cdot f(1) = 1 > 0 \quad \text{(no sign change!)}$$

The function **touches** the x-axis without crossing it. Bisection can **never bracket** this root.

#### Pitfall 4 — Singularities Can Fool the Algorithm

Consider $f(x) = \frac{1}{x}$ with $x_l = -2$, $x_u = 3$:

$$f(-2) = -\frac{1}{2} < 0, \quad f(3) = \frac{1}{3} > 0$$
$$f(-2) \cdot f(3) < 0 \quad \text{(looks like a valid bracket!)}$$

But $f(x) = \frac{1}{x}$ has **no root** — it has a vertical asymptote at $x = 0$. The sign change is caused by the discontinuity, not a root. Bisection will converge toward $x = 0$, incorrectly reporting it as a root.

**Rule:** Always verify that $f(x)$ is **continuous** on $[x_l, x_u]$ before applying bisection.

#### Summary Table

| Drawback | Root Cause |
|---|---|
| Slow (linear convergence) | Only halves the interval per step |
| Insensitive to bracket quality | Always picks midpoint, ignores function shape |
| Can't find "touching" roots | Requires sign change, so $f(x_l)f(x_u) < 0$ is impossible |
| Fooled by singularities | Continuity violated, sign change is spurious |

---

## 6. Trial Questions

Try these on your own — no notes allowed! Use them to test whether you've truly understood the material.

---

### Section A — Conceptual Questions

**Q1.** A student tries to apply bisection to find the root of $f(x) = (x-3)^2$ with $x_l = 0$ and $x_u = 5$. Explain why this will fail.

**Q2.** Explain in your own words why the bisection method is called a *bracketing* method. What is being "bracketed"?

**Q3.** A function $f(x)$ satisfies $f(2) = -5$ and $f(7) = 3$. Can you guarantee a root exists in $[2, 7]$? What additional condition is needed?

**Q4.** You are using bisection with $x_l = 1$, $x_u = 9$, and you want to guarantee an absolute error of less than $0.01$. How many iterations are needed?  
*(Hint: use $n \geq \log\left(\frac{x_u - x_l}{\epsilon}\right) / \log(2)$)*

**Q5.** What is the difference between $|\epsilon_a|$ and $\epsilon_t$? When would you use each?

**Q6.** True or False: "If $f(x_l) \cdot f(x_u) < 0$, there is exactly one root between $x_l$ and $x_u$." Justify your answer.

---

### Section B — Calculation Questions

**Q7.** Use the Bisection Method to find a root of $f(x) = x^3 - x - 1 = 0$ with initial bracket $[1, 2]$ and tolerance $\epsilon_s = 5\%$.  
Perform **at least 4 iterations**. For each iteration, show: $x_l$, $x_u$, $x_m$, $f(x_m)$, and $|\epsilon_a|$.

**Q8.** For $f(x) = \cos(x) - x = 0$, find the root in $[0, 1]$ using bisection for **3 iterations**.  
(Note: use radians for cosine.)

**Q9.** Given the iteration table below, fill in the **missing values**:

| Iter | $x_l$ | $x_u$ | $x_m$ | $f(x_m)$ | $|\epsilon_a|$ (%) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 0 | 2 | 1.0 | $-0.5$ | — |
| 2 | 1.0 | 2 | ? | ? | ? |
| 3 | ? | ? | ? | ? | ? |

The function has $f(0) = +1$, $f(1) = -0.5$, $f(1.5) = +0.375$, $f(2) = +3$.

**Q10.** After 15 iterations of bisection on the interval $[0, 4]$:  
(a) What is the maximum possible absolute error?  
(b) How many significant digits are guaranteed if $|\epsilon_a| = 0.0061\%$?

---

### Section C — Real-World and Trap Questions

**Q11.** A civil engineer models the deflection of a beam as:
$$f(x) = e^{-x} - x = 0$$
Use bisection with $x_l = 0$ and $x_u = 1$ for **4 iterations**.

**Q12.** Explain what happens when you apply bisection to:
$$f(x) = \frac{1}{x - 2} \quad \text{with} \quad x_l = 1, \; x_u = 3$$
Why is the result misleading?

**Q13.** A root-finding problem gives you the following errors across iterations:
$$|\epsilon_a| = 50\%, \; 25\%, \; 12.5\%, \; 6.25\%, \; 3.125\%$$
What method is most likely being used, and why? How many more iterations are needed to guarantee 3 significant digits?

---

### Answers — Section A Quick Key

| Q | Answer |
|---|---|
| Q1 | $f(x) = (x-3)^2 \geq 0$ always. No sign change possible since it touches the axis without crossing. |
| Q2 | The root is "bracketed" (trapped) between $x_l$ and $x_u$ — its guaranteed to be inside the interval. |
| Q3 | Not guaranteed without continuity. If $f$ is continuous on $[2,7]$, then yes, a root is guaranteed. |
| Q4 | $n \geq \log(800)/\log(2) \approx 9.6 \to n = 10$ iterations |
| Q5 | $|\epsilon_a|$ uses consecutive approximations (no true answer needed). $\epsilon_t$ uses the true value (only for verification). |
| Q6 | FALSE. There could be an odd number of roots (1, 3, 5, ...). The sign change guarantees *at least one* root. |

---

## Key Formulas Summary

| Formula | Use |
|---|---|
| $x_m = \frac{x_l + x_u}{2}$ | Midpoint (root estimate) |
| $f(x_l) \cdot f(x_u) < 0$ | Bracket validation |
| $|\epsilon_a| = \left| \frac{x_m^{\text{new}} - x_m^{\text{old}}}{x_m^{\text{new}}} \right| \times 100\%$ | Approximate error |
| $|\epsilon_a| \leq 0.5 \times 10^{2-m}\%$ | Scarborough's criterion ($m$ sig. digits) |
| $\text{Max error after } n \text{ steps} = \frac{x_u - x_l}{2^n}$ | Error bound |
| $n \geq \frac{\log\left(\frac{x_u-x_l}{\epsilon}\right)}{\log 2}$ | Iterations needed for tolerance $\epsilon$ |

---

## Quick Reference: Bisection vs. Other Methods

| Feature | Bisection | Newton-Raphson (preview) |
|---|---|---|
| Convergence | Linear (slow) | Quadratic (fast) |
| Requires derivative? | No | Yes |
| Always converges? | Yes (if bracket valid) | Not always |
| Starting requirement | Valid bracket $[x_l, x_u]$ | Single initial guess near root |

---

*Related notes: [[Numerical Methods Overview]] · [[Error Analysis]] · [[Root Finding Methods]]*
