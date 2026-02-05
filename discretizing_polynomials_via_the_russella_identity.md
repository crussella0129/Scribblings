# Discretizing Polynomials via the Russella Identity

## Introduction

The Russella Identity establishes a fundamental connection between continuous power functions and discrete summation through the binomial coefficients of Pascal's triangle. It reveals that every positive-integer power $x^r$ can be decomposed into a finite sum of polynomial increments, and that the coefficients of these increment polynomials are precisely the binomial coefficients from the $r$-th row of Pascal's triangle.

The simplest instance of this principle is the classical fact that perfect squares are sums of consecutive odd numbers:

$$1 + 3 + 5 + \cdots + (2x - 1) = x^2$$

The Russella Identity generalizes this to all positive-integer powers, extends naturally to real and complex exponents via integration and analytic continuation, and provides a unified framework — the **Nature of Growth** — for understanding how power functions accumulate their values step by step.

---

## 1. Statement of the Identity

**Theorem (Russella Identity).** For $r \in \mathbb{Z}^+$ and $x \in \mathbb{Z}^+$:

$$x^r = \sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j} n^j$$

Equivalently, defining the increment polynomial $P_r(n) = \displaystyle\sum_{j=0}^{r-1} \binom{r}{j} n^j$:

$$x^r = \sum_{n=0}^{x-1} P_r(n)$$

where $P_r(n)$ is a polynomial of degree $r - 1$ in $n$, whose coefficients are the first $r$ entries of the $r$-th row of Pascal's triangle (i.e., all entries except the trailing 1).

---

## 2. Proof of the Core Identity

The proof proceeds in two steps: the binomial theorem provides the increment structure, and telescoping completes the summation.

### Step 1: The Discrete Increment via the Binomial Theorem

For any $n \in \mathbb{Z}_{\geq 0}$ and $r \in \mathbb{Z}^+$, the binomial theorem gives:

$$(n + 1)^r = \sum_{j=0}^{r} \binom{r}{j} n^j$$

The $j = r$ term of this sum is $\binom{r}{r} n^r = n^r$. Isolating it and subtracting $n^r$ from both sides:

$$(n+1)^r - n^r = \sum_{j=0}^{r-1} \binom{r}{j} n^j = P_r(n)$$

This is the **forward difference** (discrete derivative) of the function $f(n) = n^r$. It tells us exactly how much $n^r$ increases when $n$ advances by one unit. The coefficients of this increment polynomial are drawn directly from Pascal's triangle.

### Step 2: Recovery by Telescoping

We now sum the forward differences from $n = 0$ to $n = x - 1$. The sum telescopes:

$$\sum_{n=0}^{x-1} \bigl[(n+1)^r - n^r\bigr] = \bigl(x^r - (x-1)^r\bigr) + \bigl((x-1)^r - (x-2)^r\bigr) + \cdots + \bigl(1^r - 0^r\bigr) = x^r - 0^r = x^r$$

Substituting the binomial expansion from Step 1:

$$\boxed{\;x^r = \sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j = \sum_{n=0}^{x-1} P_r(n)\;}$$

$\blacksquare$

### Interpretation: The Nature of Growth

The identity decomposes $x^r$ into its step-by-step construction. Rather than computing $x^r$ as a monolithic evaluation, we build it incrementally: start at $0^r = 0$ and add $P_r(n)$ at each step $n = 0, 1, 2, \ldots, x-1$. The polynomial $P_r(n)$ captures the **nature of growth** of $x^r$ — it is the exact amount by which the function increases at each integer step.

---

## 3. The Connection to Pascal's Triangle

Pascal's triangle encodes the binomial coefficients $\binom{r}{j}$:

```
Row 0:                  1
Row 1:                1   1
Row 2:              1   2   1
Row 3:            1   3   3   1
Row 4:          1   4   6   4   1
Row 5:        1   5  10  10   5   1
```

For the monomial $x^r$, the increment polynomial $P_r(n)$ uses the $r$-th row of Pascal's triangle **with the trailing 1 removed**:

| Power | Row $r$ of Pascal's triangle | Coefficients of $P_r(n)$ | Increment polynomial |
|---|---|---|---|
| $x^1$ | 1, **1** | 1 | $1$ |
| $x^2$ | 1, 2, **1** | 1, 2 | $1 + 2n$ |
| $x^3$ | 1, 3, 3, **1** | 1, 3, 3 | $1 + 3n + 3n^2$ |
| $x^4$ | 1, 4, 6, 4, **1** | 1, 4, 6, 4 | $1 + 4n + 6n^2 + 4n^3$ |
| $x^5$ | 1, 5, 10, 10, 5, **1** | 1, 5, 10, 10, 5 | $1 + 5n + 10n^2 + 10n^3 + 5n^4$ |

The dropped entry (bolded above) is always $\binom{r}{r} = 1$, corresponding to the $n^r$ term that cancels when computing the forward difference $(n+1)^r - n^r$.

### Why Pascal's Triangle Appears

The binomial theorem expresses $(n+1)^r$ as $\sum_{j=0}^r \binom{r}{j} n^j$. Pascal's triangle is the table of these binomial coefficients. When we subtract $n^r$ to isolate the growth increment, we remove the final coefficient, leaving the first $r$ entries of row $r$ as the coefficients of $P_r(n)$.

---

## 4. Worked Examples

### Example 1: $x^1$ — Counting

**Increment polynomial:** $P_1(n) = \binom{1}{0} = 1$

**Verification:**

$$\sum_{n=0}^{x-1} 1 = x = x^1 \quad \checkmark$$

The first power is simply counting: $x$ is the sum of $x$ ones. Growth is constant.

---

### Example 2: $x^2$ — Squares as Sums of Odd Numbers

**Increment polynomial:** $P_2(n) = \binom{2}{0} + \binom{2}{1}\, n = 1 + 2n$

**The increment sequence** $P_2(0),\; P_2(1),\; P_2(2),\; \ldots$:

$$1,\; 3,\; 5,\; 7,\; 9,\; 11,\; \ldots$$

These are the **odd numbers**. The $n$-th term (0-indexed) is $2n + 1$, which is the $(n+1)$-th odd number.

**Algebraic verification:**

$$\sum_{n=0}^{x-1} (1 + 2n) = \sum_{n=0}^{x-1} 1 + 2\sum_{n=0}^{x-1} n = x + 2 \cdot \frac{(x-1)x}{2} = x + x^2 - x = x^2 \quad \checkmark$$

**Numerical verification:**

| $x$ | Sum | Result |
|-----|-----|--------|
| 1 | $1$ | $1 = 1^2$ |
| 2 | $1 + 3$ | $4 = 2^2$ |
| 3 | $1 + 3 + 5$ | $9 = 3^2$ |
| 4 | $1 + 3 + 5 + 7$ | $16 = 4^2$ |
| 5 | $1 + 3 + 5 + 7 + 9$ | $25 = 5^2$ |

This is the most elementary case: **$x^2$ equals the sum of the first $x$ odd numbers.**

---

### Example 3: $x^3$ — Cubes

**Increment polynomial:** $P_3(n) = 1 + 3n + 3n^2$

**The increment sequence:**

$$1,\; 7,\; 19,\; 37,\; 61,\; 91,\; \ldots$$

**Algebraic verification:**

$$\sum_{n=0}^{x-1} (1 + 3n + 3n^2) = x + 3 \cdot \frac{(x-1)x}{2} + 3 \cdot \frac{(x-1)x(2x-1)}{6}$$

$$= x + \frac{3x(x-1)}{2} + \frac{x(x-1)(2x-1)}{2} = x + \frac{x(x-1)\bigl[3 + (2x-1)\bigr]}{2}$$

$$= x + \frac{x(x-1)(2x+2)}{2} = x + x(x-1)(x+1) = x + x(x^2 - 1) = x^3 \quad \checkmark$$

**Numerical verification:**

| $x$ | Sum | Result |
|-----|-----|--------|
| 1 | $1$ | $1 = 1^3$ |
| 2 | $1 + 7$ | $8 = 2^3$ |
| 3 | $1 + 7 + 19$ | $27 = 3^3$ |
| 4 | $1 + 7 + 19 + 37$ | $64 = 4^3$ |

Note that the coefficients $[1, 3, 3]$ are the first three entries of row 3 of Pascal's triangle.

---

### Example 4: $x^4$ — Fourth Powers

**Increment polynomial:** $P_4(n) = 1 + 4n + 6n^2 + 4n^3$

**The increment sequence:**

$$1,\; 15,\; 65,\; 175,\; 369,\; 671,\; \ldots$$

**Numerical verification:**

| $x$ | Sum | Result |
|-----|-----|--------|
| 1 | $1$ | $1 = 1^4$ |
| 2 | $1 + 15$ | $16 = 2^4$ |
| 3 | $1 + 15 + 65$ | $81 = 3^4$ |
| 4 | $1 + 15 + 65 + 175$ | $256 = 4^4$ |
| 5 | $1 + 15 + 65 + 175 + 369$ | $625 = 5^4$ |

The coefficients $[1, 4, 6, 4]$ are the first four entries of row 4 of Pascal's triangle.

---

## 5. Discretization of General Polynomials

Since the Russella Identity works for every monomial $x^r$ (with $r \geq 1$), linearity of summation extends it to arbitrary polynomials.

**Corollary.** Let $p(x) = a_0 + a_1 x + a_2 x^2 + \cdots + a_d x^d$ be a polynomial of degree $d$. Then for $x \in \mathbb{Z}^+$:

$$p(x) = p(0) + \sum_{n=0}^{x-1} Q(n)$$

where $Q(n) = p(n+1) - p(n)$ is the forward difference of $p$, which can be expressed explicitly as:

$$Q(n) = \sum_{k=1}^{d} a_k \cdot P_k(n) = \sum_{k=1}^{d} a_k \sum_{j=0}^{k-1} \binom{k}{j}\, n^j$$

**Proof.** Each monomial $x^k$ with $k \geq 1$ satisfies $x^k = \sum_{n=0}^{x-1} P_k(n)$, so:

$$p(x) = a_0 + \sum_{k=1}^{d} a_k x^k = a_0 + \sum_{k=1}^{d} a_k \sum_{n=0}^{x-1} P_k(n) = a_0 + \sum_{n=0}^{x-1} \sum_{k=1}^{d} a_k P_k(n) \quad \blacksquare$$

When $p(0) = 0$ (i.e., $a_0 = 0$), this simplifies to $p(x) = \sum_{n=0}^{x-1} Q(n)$, a pure discrete summation.

### Concrete Example

Consider $p(x) = 2x^3 + 5x^2$. Here $p(0) = 0$, so:

$$Q(n) = 2 \cdot P_3(n) + 5 \cdot P_2(n) = 2(1 + 3n + 3n^2) + 5(1 + 2n) = 7 + 16n + 6n^2$$

**Verification at $x = 3$:**

$$Q(0) + Q(1) + Q(2) = 7 + (7 + 16 + 6) + (7 + 32 + 24) = 7 + 29 + 63 = 99$$

$$p(3) = 2(27) + 5(9) = 54 + 45 = 99 \quad \checkmark$$

Any polynomial evaluated at positive integers can be computed as a running accumulation of its discrete increments, with all coefficients determined by Pascal's triangle.

---

## 6. Connection to Discrete Calculus

The Russella Identity is the power-function instance of the **fundamental theorem of discrete calculus** (also called the summation analog of the fundamental theorem of calculus).

### The Forward Difference Operator

Define the forward difference operator $\Delta$ by:

$$\Delta f(n) = f(n+1) - f(n)$$

This is the discrete analog of the derivative $\frac{d}{dn} f(n)$.

For power functions, the Russella Identity gives us an explicit formula:

$$\Delta[n^r] = (n+1)^r - n^r = \sum_{j=0}^{r-1} \binom{r}{j}\, n^j$$

Compare with the continuous derivative:

$$\frac{d}{dn}[n^r] = r\, n^{r-1}$$

### The Discrete Fundamental Theorem

Just as the fundamental theorem of calculus states:

$$f(x) - f(0) = \int_0^x f'(t)\, dt$$

the discrete fundamental theorem states:

$$f(x) - f(0) = \sum_{n=0}^{x-1} \Delta f(n)$$

The Russella Identity is exactly this theorem applied to $f(n) = n^r$:

$$x^r - 0^r = \sum_{n=0}^{x-1} \Delta[n^r] = \sum_{n=0}^{x-1} P_r(n)$$

### Leading-Term Correspondence

The discrete increment $P_r(n)$ and the continuous derivative $r\,n^{r-1}$ are closely related. The leading term of $P_r(n)$ is:

$$\binom{r}{r-1}\, n^{r-1} = r\, n^{r-1}$$

which is exactly the continuous derivative. The remaining terms $\binom{r}{0} + \binom{r}{1}n + \cdots + \binom{r}{r-2}n^{r-2}$ are lower-order corrections that account for the finite step size. In the limit of infinitesimal steps, these corrections vanish, and the discrete framework reduces to the continuous one.

This can be seen precisely: as a step size $h \to 0$,

$$\frac{(n+h)^r - n^r}{h} = \sum_{j=0}^{r-1} \binom{r}{j} n^j h^{j-1} \;\;\xrightarrow{h \to 0}\;\; r\,n^{r-1}$$

Only the $j = r-1$ term survives, recovering the power rule of continuous calculus.

---

## 7. The Continuous Generalization

### From Summation to Integration

For $r \in \mathbb{Z}^+$, the Russella Identity expresses $x^r$ as a discrete sum. When $r$ is extended to $\mathbb{R}^+$, the discrete sum no longer applies (the binomial coefficients $\binom{r}{j}$ still make sense, but the finite telescoping argument breaks down for non-integer exponents). However, the continuous analog via the fundamental theorem of calculus holds for all $r \in \mathbb{R}^+$:

$$x^r = \int_0^x r\, n^{r-1}\, dn$$

**Proof.** By the power rule of integration:

$$\int_0^x r\, n^{r-1}\, dn = \left[ n^r \right]_0^x = x^r - 0^r = x^r \quad \blacksquare$$

### From Integration to Exponentiation

For $r \in \mathbb{C}$ (including negative, irrational, and complex exponents), $x^r$ is most generally defined via the exponential:

$$x^r = e^{r \ln x}$$

This definition agrees with the integral form when $r \in \mathbb{R}^+$ and with the discrete sum when $r \in \mathbb{Z}^+$, providing a single unified expression valid across all of $\mathbb{C}$ (with appropriate branch cuts for $\ln x$).

### The Chain of Equivalences

The full Russella Identity, as expressed in the main identity image, is:

$$x^r = \sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j} n^j = \int_0^x r\, n^{r-1}\, dn = e^{r \ln x}$$

Each form is the natural representation for a different domain of $r$:

- The **double sum** is native to $r \in \mathbb{Z}^+$ (discrete/combinatorial).
- The **integral** is native to $r \in \mathbb{R}^+$ (continuous/analytic).
- The **exponential** is native to $r \in \mathbb{C}$ (complex/universal).

Moving left to right generalizes the domain; moving right to left discretizes the representation.

---

## 8. The Polynomial Set Classification

The domains over which $x^r$ can be expressed via each form define a hierarchy of **polynomial sets**:

| Polynomial Set | Notation | Domain of $r$ | Accumulative Form |
|---|---|---|---|
| **Summable** | $\mathbb{S}_{\text{poly}}$ | $r \in \mathbb{Z}^+$ | $\displaystyle\sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j = x^r$ |
| **Integrable** | $\mathbb{I}_{\text{poly}}$ | $r \in \mathbb{R}^+ \setminus \mathbb{Z}^+$ | $\displaystyle\int_0^x r\, n^{r-1}\, dn = x^r$ |
| **Contour Integrable** | $\mathbb{CI}_{\text{poly}}$ | $r \in \mathbb{C} \setminus \mathbb{R}^+ \setminus \mathbb{Z}^+ \setminus \{-1\}$ | $\displaystyle\frac{1}{2\pi i} \oint z^r\, e^{x/z}\, dz = x^r$ |
| **Universal** | $\mathbb{U}_{\text{poly}}$ | $r \in \mathbb{C}$ | $\displaystyle\int_0^{\infty} t^{r-1} e^{-t}\, dt = \Gamma(r),\quad e^{r\ln x} = x^r$ |

Each successive set strictly contains the previous one, and each representation is the simplest **accumulative function** (i.e., a function that builds $x^r$ by accumulating contributions) valid on its domain.

### The Piecewise Unification

These representations can be unified into a single piecewise-defined function:

$$x^r = \begin{cases} \displaystyle\sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j, & r \in \mathbb{Z}^+ \\[12pt] \displaystyle\int_0^x r\, n^{r-1}\, dn, & r \in \mathbb{R}^+ \\[12pt] e^{r \ln x}, & r \in \mathbb{C} \end{cases}$$

All three branches agree wherever their domains overlap: for $r \in \mathbb{Z}^+$, the discrete sum, the integral, and the exponential all yield the same value $x^r$.

---

## 9. Why the Discrete and Continuous Forms Agree

It is not a coincidence that the discrete sum and the integral give the same result for $r \in \mathbb{Z}^+$. Here we prove their agreement directly.

**Claim.** For $r \in \mathbb{Z}^+$ and $x \in \mathbb{Z}^+$:

$$\sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j = \int_0^x r\, n^{r-1}\, dn$$

**Proof.** Both sides equal $x^r$:

- The left side equals $x^r$ by the telescoping argument (Section 2).
- The right side equals $x^r$ by the fundamental theorem of calculus.

Therefore they are equal. $\blacksquare$

But one can also see a deeper structural reason. The discrete sum computes:

$$\sum_{n=0}^{x-1} P_r(n) = \sum_{n=0}^{x-1} \bigl[(n+1)^r - n^r\bigr]$$

The integral computes:

$$\int_0^x r\, n^{r-1}\, dn = \int_0^x \frac{d}{dn}[n^r]\, dn$$

Both are instances of the same principle — **recovering a function from its rate of change** — applied in different calculi. The discrete sum uses forward differences (step size 1); the integral uses infinitesimal differences (step size $dn$). For polynomial functions evaluated at integers, both methods yield exact results.

---

## 10. Structural Observations

### 10.1 The Increment Polynomial $P_r(n)$ Encodes Finite Growth

The polynomial $P_r(n) = (n+1)^r - n^r$ answers the question: *"By how much does $n^r$ grow when $n$ increases by 1?"* Its coefficients — the truncated rows of Pascal's triangle — are a combinatorial encoding of this growth.

For small $n$, the lower-order terms (the "corrections" from Pascal's triangle) dominate. For large $n$, the leading term $r\, n^{r-1}$ dominates, and $P_r(n)$ approaches the continuous derivative. This transition from combinatorial to analytic behavior is the quantitative content of the Nature of Growth.

### 10.2 Every Positive-Integer Power Generates a Summable Sequence

The identity reveals that every perfect power sequence $1^r, 2^r, 3^r, \ldots$ is the sequence of **partial sums** of the increment sequence $P_r(0), P_r(1), P_r(2), \ldots$:

| $r$ | Increment sequence | Partial sums (= perfect $r$-th powers) |
|-----|------|------|
| 1 | $1, 1, 1, 1, \ldots$ | $1, 2, 3, 4, \ldots$ |
| 2 | $1, 3, 5, 7, \ldots$ | $1, 4, 9, 16, \ldots$ |
| 3 | $1, 7, 19, 37, \ldots$ | $1, 8, 27, 64, \ldots$ |
| 4 | $1, 15, 65, 175, \ldots$ | $1, 16, 81, 256, \ldots$ |

Each increment sequence is generated by a polynomial of one degree lower than the power it sums to, and all such polynomials have their coefficients dictated by Pascal's triangle.

### 10.3 The Constant First Term

Every increment polynomial satisfies $P_r(0) = \binom{r}{0} = 1$, regardless of $r$. This reflects the fact that $1^r - 0^r = 1$ for all $r \in \mathbb{Z}^+$: the first step of growth is always 1.

### 10.4 Recursive Structure of the Increments

The increment polynomials satisfy a clean recurrence across powers.

**Claim.** $P_{r+1}(n) = (1 + n) \cdot P_r(n) + n^r$

**Proof via Pascal's rule on the coefficients.** We have $P_{r+1}(n) = \sum_{j=0}^{r} \binom{r+1}{j} n^j$. Applying Pascal's rule $\binom{r+1}{j} = \binom{r}{j} + \binom{r}{j-1}$ (where $\binom{r}{-1} = 0$):

$$P_{r+1}(n) = \sum_{j=0}^{r} \binom{r}{j}\, n^j + \sum_{j=1}^{r} \binom{r}{j-1}\, n^j$$

The first sum is $P_r(n) + n^r$ (since it includes the $j = r$ term $\binom{r}{r} n^r = n^r$ beyond the $P_r$ range). The second sum, after re-indexing $k = j - 1$, is $n \cdot \sum_{k=0}^{r-1} \binom{r}{k} n^k = n \cdot P_r(n)$. Combining:

$$P_{r+1}(n) = P_r(n) + n^r + n \cdot P_r(n) = (1 + n) \cdot P_r(n) + n^r \quad \blacksquare$$

**Proof from first principles.** Directly from $P_r(n) = (n+1)^r - n^r$:

$$(1 + n) \cdot P_r(n) + n^r = (1+n)\bigl[(n+1)^r - n^r\bigr] + n^r = (n+1)^{r+1} - n^{r+1} = P_{r+1}(n) \quad \blacksquare$$

**Verification.** Taking $P_2(n) = 1 + 2n$:

$$P_3(n) = (1+n)(1 + 2n) + n^2 = 1 + 3n + 2n^2 + n^2 = 1 + 3n + 3n^2 \quad \checkmark$$

---

## 11. Summary

The Russella Identity,

$$x^r = \sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j = \int_0^x r\, n^{r-1}\, dn = e^{r \ln x}$$

provides three equivalent windows into the nature of power functions:

1. **Discrete (Summable):** For positive-integer exponents, $x^r$ is a finite sum of polynomial increments whose coefficients are the binomial entries of Pascal's triangle. This discretizes the polynomial, expressing a closed-form evaluation as an explicit accumulation process.

2. **Continuous (Integrable):** For positive-real exponents, the discrete sum generalizes to an integral of the continuous derivative $r\,n^{r-1}$, which is the leading term of the discrete increment polynomial.

3. **Universal (Complex):** For all complex exponents, $x^r = e^{r \ln x}$ provides the fully general definition, from which the integral and sum forms are recovered as special cases.

The key insight is that **Pascal's triangle encodes the discrete growth structure of all power functions.** The classical fact that "the sum of the first $x$ odd numbers is $x^2$" is not an isolated curiosity — it is the $r = 2$ case of a universal pattern in which every positive-integer power $x^r$ is the cumulative sum of a polynomial sequence whose coefficients are read directly from Pascal's triangle.
