# The Orders of Growth

*Pascal's Triangle as the Universal Growth Encoder for Power Functions*

## Abstract

We present a unified framework that reveals Pascal's triangle as encoding the discrete growth structure of all positive-integer power functions. The core mechanism — the binomial expansion of forward differences, followed by telescoping recovery — has been implicit in the finite difference calculus since Newton and Gregory (1670s). What has not been presented as a unified pedagogical object is the full picture: Pascal's triangle dictates the step-by-step growth of every integer power, the discrete growth polynomials converge to continuous derivatives as resolution increases, and the resulting hierarchy of representations — discrete sums, integrals, and complex exponentiation — forms a natural classification of power functions by the "order" of accumulation required to express them. We develop this framework (the *Russella framework*) in detail, prove a recurrence connecting successive growth polynomials, and situate the decomposition alongside related classical results including Stirling numbers and Faulhaber's formulas.

---

## 1. Motivation: The Odd Numbers Mystery

The Pythagoreans knew, at least as early as the 5th century BC, that perfect squares are sums of consecutive odd numbers (Nicomachus, *Introductio Arithmeticae*, c. 100 AD):

$$1 + 3 + 5 + \cdots + (2x - 1) = x^2$$

Geometrically, each odd number $2k+1$ forms an L-shaped gnomon that, when wrapped around a $k \times k$ square, produces a $(k+1) \times (k+1)$ square. The algebraic content is immediate: the $n$-th term of the sequence (0-indexed) is $1 + 2n$, the polynomial whose coefficients $[1, 2]$ are the first two entries of row 2 of Pascal's triangle.

This raises a natural question: **What is the $r = 3$ version of this identity? The $r = 4$ version? Is there a universal pattern that generates the growth sequence for every power $x^r$?**

The answer has been implicit in the mathematical literature since the development of finite difference calculus by Gregory (1670) and Newton (1687). The forward difference $\Delta[n^r] = (n+1)^r - n^r$ expands via the binomial theorem into a polynomial whose coefficients are binomial entries — a fact that appears throughout the classical literature on finite differences (Jordan, 1939; Graham, Knuth, and Patashnik, 1994, Section 2.6). Telescoping these differences to recover $x^r$ is an instance of the discrete fundamental theorem of calculus.

What has not been presented as a single unified object is the full framework: the explicit role of Pascal's triangle as a universal growth encoder, the convergence of discrete growth polynomials to continuous derivatives, the recursive structure connecting growth polynomials across powers, and the classification of power functions into a hierarchy of accumulative representations. That synthesis is the contribution of this paper.

---

## 2. The Growth Decomposition

### 2.1 The Discrete Increment via the Binomial Theorem

For any $n \in \mathbb{Z}_{\geq 0}$ and $r \in \mathbb{Z}^+$, the binomial theorem gives:

$$(n + 1)^r = \sum_{j=0}^{r} \binom{r}{j} n^j$$

The $j = r$ term is $n^r$. Subtracting it from both sides:

$$(n+1)^r - n^r = \sum_{j=0}^{r-1} \binom{r}{j} n^j$$

Define the **growth polynomial**:

$$P_r(n) \;=\; \sum_{j=0}^{r-1} \binom{r}{j}\, n^j \;=\; (n+1)^r - n^r$$

This is the forward difference $\Delta[n^r]$: the exact amount by which $n^r$ increases when $n$ advances by one unit.

### 2.2 Recovery by Telescoping

Summing the forward differences from $n = 0$ to $n = x - 1$ telescopes:

$$\sum_{n=0}^{x-1} \bigl[(n+1)^r - n^r\bigr] = x^r - 0^r = x^r$$

Substituting the binomial expansion:

$$\boxed{\;x^r = \sum_{n=0}^{x-1} P_r(n) = \sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j\;}$$

for all $r \in \mathbb{Z}^+$ and $x \in \mathbb{Z}^+$. $\blacksquare$

We call this the **growth decomposition** of $x^r$: the expression of a power function as the cumulative sum of its step-by-step increments, with all coefficients drawn from Pascal's triangle.

---

## 3. Pascal's Triangle as Growth Encoder

Pascal's triangle encodes the binomial coefficients $\binom{r}{j}$:

```
Row 0:                  1
Row 1:                1   1
Row 2:              1   2   1
Row 3:            1   3   3   1
Row 4:          1   4   6   4   1
Row 5:        1   5  10  10   5   1
```

For the monomial $x^r$, the growth polynomial $P_r(n)$ uses the $r$-th row of Pascal's triangle **with the trailing 1 removed**:

| Power | Row $r$ of Pascal's triangle | Coefficients of $P_r(n)$ | Growth polynomial |
|---|---|---|---|
| $x^1$ | 1, **1** | [1] | $1$ |
| $x^2$ | 1, 2, **1** | [1, 2] | $1 + 2n$ |
| $x^3$ | 1, 3, 3, **1** | [1, 3, 3] | $1 + 3n + 3n^2$ |
| $x^4$ | 1, 4, 6, 4, **1** | [1, 4, 6, 4] | $1 + 4n + 6n^2 + 4n^3$ |
| $x^5$ | 1, 5, 10, 10, 5, **1** | [1, 5, 10, 10, 5] | $1 + 5n + 10n^2 + 10n^3 + 5n^4$ |

The dropped entry (bolded) is always $\binom{r}{r} = 1$, corresponding to the $n^r$ term that cancels when computing the forward difference. What remains are the first $r$ entries of row $r$ — a truncated row of Pascal's triangle — serving as the complete description of how $x^r$ grows at each integer step.

### 3.1 Comparison to Stirling Numbers

The growth decomposition is not the only way to bridge discrete and continuous representations of power functions. The **Stirling numbers of the second kind** $S(r, k)$ (Stirling, *Methodus Differentialis*, 1730) provide an alternative decomposition:

$$x^r = \sum_{k=0}^{r} S(r, k)\, x^{(k)}$$

where $x^{(k)} = x(x-1)(x-2)\cdots(x-k+1)$ is the falling factorial. Falling factorials are the "natural monomials" of discrete calculus because they satisfy $\Delta[x^{(k)}] = k\, x^{(k-1)}$, perfectly mirroring the continuous power rule $\frac{d}{dx}[x^k] = k\, x^{k-1}$.

The two decompositions serve different purposes:
- The **Stirling decomposition** changes basis from ordinary powers to falling factorials, making discrete calculus operations (summation, differencing) algebraically clean.
- The **growth decomposition** stays in the ordinary power basis and makes the *step-by-step accumulation* of $x^r$ explicit. Its coefficients are readable directly from Pascal's triangle without computing Stirling numbers.

The growth decomposition answers "how does $x^r$ build up, one step at a time?" The Stirling decomposition answers "how do I express $x^r$ in the natural coordinates of discrete calculus?"

---

## 4. Worked Examples

### 4.1 $x^2$ — Squares as Sums of Odd Numbers

**Growth polynomial:** $P_2(n) = 1 + 2n$

The sequence $P_2(0),\, P_2(1),\, P_2(2),\, \ldots$ is:

$$1,\; 3,\; 5,\; 7,\; 9,\; 11,\; \ldots$$

the odd numbers. Verification:

$$\sum_{n=0}^{x-1} (1 + 2n) = x + 2 \cdot \frac{(x-1)x}{2} = x + x^2 - x = x^2 \quad \checkmark$$

| $x$ | Sum | Result |
|-----|-----|--------|
| 1 | $1$ | $1 = 1^2$ |
| 2 | $1 + 3$ | $4 = 2^2$ |
| 3 | $1 + 3 + 5$ | $9 = 3^2$ |
| 4 | $1 + 3 + 5 + 7$ | $16 = 4^2$ |
| 5 | $1 + 3 + 5 + 7 + 9$ | $25 = 5^2$ |

This is the Pythagorean result: **$x^2$ equals the sum of the first $x$ odd numbers.** The growth decomposition reveals it as the $r = 2$ instance of a universal pattern.

### 4.2 $x^3$ — Cubes

**Growth polynomial:** $P_3(n) = 1 + 3n + 3n^2$. Coefficients $[1, 3, 3]$: first three entries of row 3.

$$\sum_{n=0}^{x-1} (1 + 3n + 3n^2) = x + \frac{3x(x-1)}{2} + \frac{x(x-1)(2x-1)}{2} = x + \frac{x(x-1)(2x+2)}{2} = x + x(x^2 - 1) = x^3 \quad \checkmark$$

| $x$ | Sum | Result |
|-----|-----|--------|
| 1 | $1$ | $1 = 1^3$ |
| 2 | $1 + 7$ | $8 = 2^3$ |
| 3 | $1 + 7 + 19$ | $27 = 3^3$ |
| 4 | $1 + 7 + 19 + 37$ | $64 = 4^3$ |

### 4.3 Summary Table

Every positive-integer power $x^r$ is the partial sum of its growth sequence. The table below summarizes the first five orders:

| $r$ | Growth polynomial $P_r(n)$ | Growth sequence | Partial sums |
|-----|---|---|---|
| 1 | $1$ | $1, 1, 1, 1, 1, \ldots$ | $1, 2, 3, 4, 5, \ldots$ |
| 2 | $1 + 2n$ | $1, 3, 5, 7, 9, \ldots$ | $1, 4, 9, 16, 25, \ldots$ |
| 3 | $1 + 3n + 3n^2$ | $1, 7, 19, 37, 61, \ldots$ | $1, 8, 27, 64, 125, \ldots$ |
| 4 | $1 + 4n + 6n^2 + 4n^3$ | $1, 15, 65, 175, 369, \ldots$ | $1, 16, 81, 256, 625, \ldots$ |
| 5 | $1 + 5n + 10n^2 + 10n^3 + 5n^4$ | $1, 31, 211, 781, 2101, \ldots$ | $1, 32, 243, 1024, 3125, \ldots$ |

---

## 5. Discretization of General Polynomials

Since the growth decomposition works for every monomial $x^r$ with $r \geq 1$, linearity extends it to arbitrary polynomials.

**Corollary.** Let $p(x) = a_0 + a_1 x + a_2 x^2 + \cdots + a_d x^d$ be a polynomial of degree $d$. Then for $x \in \mathbb{Z}^+$:

$$p(x) = p(0) + \sum_{n=0}^{x-1} Q(n)$$

where $Q(n) = p(n+1) - p(n) = \sum_{k=1}^{d} a_k \cdot P_k(n)$.

**Proof.** Each monomial $x^k$ with $k \geq 1$ satisfies $x^k = \sum_{n=0}^{x-1} P_k(n)$, so:

$$p(x) = a_0 + \sum_{k=1}^{d} a_k\, x^k = a_0 + \sum_{k=1}^{d} a_k \sum_{n=0}^{x-1} P_k(n) = p(0) + \sum_{n=0}^{x-1} \sum_{k=1}^{d} a_k\, P_k(n) \quad \blacksquare$$

When $p(0) = 0$, this simplifies to $p(x) = \sum_{n=0}^{x-1} Q(n)$: a pure discrete summation with all coefficients determined by Pascal's triangle.

**Example.** For $p(x) = 2x^3 + 5x^2$:

$$Q(n) = 2P_3(n) + 5P_2(n) = 2(1 + 3n + 3n^2) + 5(1 + 2n) = 7 + 16n + 6n^2$$

At $x = 3$: $\;Q(0) + Q(1) + Q(2) = 7 + 29 + 63 = 99 = 2(27) + 5(9)$. $\checkmark$

---

## 6. The Discrete-Continuous Bridge

### 6.1 Two Calculi, One Principle

The growth decomposition is the power-function instance of the **fundamental theorem of discrete calculus** (cf. Graham, Knuth, and Patashnik, *Concrete Mathematics*, Section 2.6). The discrete and continuous fundamental theorems are structurally identical:

| | Discrete calculus | Continuous calculus |
|---|---|---|
| **Rate of change** | $\Delta f(n) = f(n+1) - f(n)$ | $f'(t) = \lim_{h \to 0} \frac{f(t+h) - f(t)}{h}$ |
| **Recovery** | $f(x) - f(0) = \displaystyle\sum_{n=0}^{x-1} \Delta f(n)$ | $f(x) - f(0) = \displaystyle\int_0^x f'(t)\, dt$ |
| **Applied to $n^r$** | $x^r = \displaystyle\sum_{n=0}^{x-1} P_r(n)$ | $x^r = \displaystyle\int_0^x r\,t^{r-1}\, dt$ |

Both recover a function from its rate of change. The discrete version uses unit-step increments; the continuous version uses infinitesimal increments. For integer-valued power functions, both yield $x^r$ exactly.

### 6.2 Leading-Term Correspondence

The growth polynomial $P_r(n)$ and the continuous derivative $r\,n^{r-1}$ are not independent objects. The leading term of $P_r(n)$ is:

$$\binom{r}{r-1}\, n^{r-1} = r\, n^{r-1}$$

which is exactly the continuous derivative. The remaining terms $\binom{r}{0} + \binom{r}{1}n + \cdots + \binom{r}{r-2}n^{r-2}$ are lower-order corrections arising from the finite step size. As the step size $h \to 0$:

$$\frac{(n+h)^r - n^r}{h} = \sum_{j=0}^{r-1} \binom{r}{j}\, n^j\, h^{j-1} \;\;\xrightarrow{h \to 0}\;\; r\,n^{r-1}$$

Only the $j = r - 1$ term survives, recovering the power rule of continuous calculus. The growth polynomial is the *exact finite-step version* of what the derivative approximates infinitesimally.

### 6.3 Convergence in Practice

The following table illustrates the convergence of $P_r(n)$ toward the continuous derivative $r\,n^{r-1}$ for $r = 3$:

| $n$ | $P_3(n) = 1 + 3n + 3n^2$ | $3n^2$ (continuous derivative) | Ratio $P_3(n) / 3n^2$ |
|-----|---|---|---|
| 1 | 7 | 3 | 2.33 |
| 2 | 19 | 12 | 1.58 |
| 5 | 91 | 75 | 1.21 |
| 10 | 331 | 300 | 1.10 |
| 50 | 7651 | 7500 | 1.02 |
| 100 | 30301 | 30000 | 1.01 |

At large $n$, the lower-order terms $(1 + 3n)$ become negligible relative to $3n^2$, and the discrete growth polynomial converges to the continuous derivative. The discrete and continuous calculi are the same object viewed at different resolutions.

---

## 7. Three Representations of $x^r$

The growth decomposition sits at the discrete end of a hierarchy of representations. Three expressions all evaluate to $x^r$, each native to a different domain:

$$\sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j \quad=\quad \int_0^x r\,t^{r-1}\, dt \quad=\quad e^{r \ln x}$$

These are not "equivalences" in a strict algebraic sense — they are three representations that **agree wherever their domains overlap**:

- The **discrete sum** requires $r \in \mathbb{Z}^+$ and $x \in \mathbb{Z}^+$. It is native to the combinatorial world.
- The **integral** requires $r > 0$ and $x > 0$ (both real). It extends the discrete sum to all positive-real exponents. Importantly, it *also* works for $r \in \mathbb{Z}^+$ — it is a strict generalization, not an alternative.
- The **exponential** $e^{r \ln x}$ requires only $x > 0$ and $r \in \mathbb{C}$ (with a branch cut for $\ln x$). It extends the integral to complex exponents and serves as the universal definition of $x^r$.

Each successive representation absorbs the previous one and extends the domain. Moving left to right generalizes; moving right to left discretizes.

---

## 8. The Polynomial Set Classification

The three representations define a hierarchy of **polynomial sets**, classified by the simplest accumulative form that expresses $x^r$ on a given domain of $r$:

| Order | Polynomial Set | Notation | Domain of $r$ | Simplest Accumulative Form |
|---|---|---|---|---|
| I | **Summable** | $\mathbb{S}_{\text{poly}}$ | $r \in \mathbb{Z}^+$ | $\displaystyle x^r = \sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j$ |
| II | **Integrable** | $\mathbb{I}_{\text{poly}}$ | $r \in \mathbb{R}^+$ | $\displaystyle x^r = \int_0^x r\, t^{r-1}\, dt$ |
| III | **Universal** | $\mathbb{U}_{\text{poly}}$ | $r \in \mathbb{C}$ | $x^r = e^{r \ln x}$ |

$$\mathbb{S}_{\text{poly}} \;\subset\; \mathbb{I}_{\text{poly}} \;\subset\; \mathbb{U}_{\text{poly}}$$

Each set strictly contains the previous one. The classification captures the idea that as the exponent $r$ becomes less "discrete" (moving from integers to reals to complex numbers), the representation required to express $x^r$ becomes correspondingly less combinatorial and more analytic.

The three orders are the **orders of growth**: they measure the level of mathematical machinery needed to accumulate $x^r$ from its infinitesimal or combinatorial components.

---

## 9. The Recursive Structure of Growth

### 9.1 The Recurrence

The growth polynomials satisfy a recurrence connecting successive powers.

**Theorem.** For all $r \geq 1$ and $n \geq 0$:

$$P_{r+1}(n) = (1 + n) \cdot P_r(n) + n^r$$

**Proof (algebraic).** We have $P_{r+1}(n) = \sum_{j=0}^{r} \binom{r+1}{j}\, n^j$. Applying Pascal's rule $\binom{r+1}{j} = \binom{r}{j} + \binom{r}{j-1}$:

$$P_{r+1}(n) = \sum_{j=0}^{r} \binom{r}{j}\, n^j + \sum_{j=1}^{r} \binom{r}{j-1}\, n^j$$

The first sum is $P_r(n) + \binom{r}{r}\,n^r = P_r(n) + n^r$. The second sum, re-indexing $k = j - 1$, is $n \cdot \sum_{k=0}^{r-1} \binom{r}{k}\, n^k = n \cdot P_r(n)$. Combining:

$$P_{r+1}(n) = P_r(n) + n^r + n \cdot P_r(n) = (1 + n) \cdot P_r(n) + n^r \quad \blacksquare$$

**Proof (direct).** From $P_r(n) = (n+1)^r - n^r$:

$$(1+n) \cdot P_r(n) + n^r = (n+1)\bigl[(n+1)^r - n^r\bigr] + n^r = (n+1)^{r+1} - n^{r+1} = P_{r+1}(n) \quad \blacksquare$$

**Verification.** Starting from $P_1(n) = 1$:

$$P_2(n) = (1+n) \cdot 1 + n^1 = 1 + 2n \quad \checkmark$$

$$P_3(n) = (1+n)(1+2n) + n^2 = 1 + 3n + 2n^2 + n^2 = 1 + 3n + 3n^2 \quad \checkmark$$

### 9.2 Interpretation

The recurrence says: *the growth of $x^{r+1}$ at step $n$ is $(1+n)$ times the growth of $x^r$ at step $n$, plus a correction term $n^r$.* This is a multiplicative-additive recurrence — each order of growth builds on the previous one through scaling and correction, mirroring how Pascal's triangle itself is built by summing adjacent entries.

### 9.3 Exponential Generating Function

The growth polynomials have a clean exponential generating function in $r$. Since $P_r(n) = (n+1)^r - n^r$:

$$\sum_{r=0}^{\infty} P_r(n)\, \frac{t^r}{r!} = e^{(n+1)t} - e^{nt} = e^{nt}(e^t - 1)$$

The factor $e^{nt}$ encodes the base point; the factor $(e^t - 1)$ is the generating function of the forward difference operator itself. This connects the growth polynomials to the classical theory of finite differences and the umbral calculus (Rota, Kahaner, and Odlyzko, 1973; Roman, 1984).

### 9.4 Open Questions on the Recurrence

The recurrence $P_{r+1}(n) = (1+n) \cdot P_r(n) + n^r$ invites several questions:

- **Matrix formulation.** The growth polynomials $P_1, P_2, \ldots, P_r$ evaluated at a fixed $n$ form a sequence generated by a linear recurrence. Can this be expressed as a matrix power, and does the matrix have interesting spectral properties?
- **Extension to matrix powers.** If $A$ is a square matrix, can the growth decomposition be extended to $A^r$? The binomial theorem generalizes to matrices when the factors commute, suggesting $P_r(A) = (A + I)^r - A^r$ is well-defined.
- **$q$-analog.** The $q$-binomial coefficients $\binom{r}{j}_q$ are the natural $q$-deformation of the ordinary binomial coefficients. Does the growth decomposition have a $q$-analog, and what does it decompose?

---

## 10. Related Frameworks

The growth decomposition is one of three classical ways to relate power functions and discrete summation. The following table clarifies where it sits:

| Framework | Question answered | Decomposes | Into | Coefficients from |
|---|---|---|---|---|
| **Growth decomposition** | How does $x^r$ build up step by step? | $x^r$ (a value) | Growth increments $P_r(n)$ | Pascal's triangle (binomial coefficients) |
| **Stirling numbers** | How is $x^r$ expressed in discrete-calculus coordinates? | $x^r$ (a polynomial) | Falling factorials $x^{(k)}$ | Stirling numbers $S(r, k)$ |
| **Faulhaber / Bernoulli** | What is the closed form of $\sum n^r$? | $\sum_{n=0}^{x-1} n^r$ (a sum) | Polynomial in $x$ | Bernoulli numbers $B_k$ |

The growth decomposition and Faulhaber's formulas are in a sense dual: the growth decomposition expresses a *value* $x^r$ as a sum of increments, while Faulhaber's formulas express a *sum* $\sum n^r$ as a closed-form polynomial. The Stirling decomposition operates at a different level entirely, changing the polynomial basis rather than decomposing a value into steps.

### Computational Note

The growth decomposition yields an $O(x \cdot r)$ algorithm for computing $x^r$ by accumulation (evaluate $P_r(n)$ at each of $x$ steps, each evaluation costing $O(r)$). This is worse than direct computation by repeated squaring ($O(\log r)$ multiplications) for serial execution. However, the growth decomposition parallelizes trivially: each $P_r(n)$ is independent of the others, yielding a natural map-reduce structure with $O(r)$ depth given $x$ processors. The value is structural rather than computational — it reveals the *architecture* of how $x^r$ accumulates, not a faster way to evaluate it.

---

## 11. Structural Observations

### 11.1 The Constant First Term

Every growth polynomial satisfies $P_r(0) = 1$, regardless of $r$. This reflects the universal fact that $1^r - 0^r = 1$ for all $r \in \mathbb{Z}^+$: the first step of growth is always 1, no matter the power.

### 11.2 Degree Reduction

The growth polynomial $P_r(n)$ has degree $r - 1$. Each order of growth reduces the polynomial degree by 1. Iterating the growth decomposition — decomposing $P_r(n)$ itself into its own growth increments — would reduce the degree further, eventually reaching degree 0 (constant increments) after $r - 1$ iterations. This tower of iterated differences connects to the theory of higher-order forward differences $\Delta^k[n^r]$, which vanish for $k > r$.

---

## 12. Open Questions

The framework suggests several directions:

1. **Physical interpretation.** If $x^r$ models a physical quantity (e.g., energy scaling as the $r$-th power of a variable), what does $P_r(n)$ represent physically? The growth polynomial is the "quantum" of increase at each discrete step — is there a natural physical setting where this decomposition is directly observable?

2. **$q$-deformation.** The $q$-binomial coefficients $\binom{r}{j}_q$ generalize the ordinary binomial coefficients. A $q$-analog of the growth decomposition would replace Pascal's triangle with its $q$-deformed version. What quantity does the resulting $q$-sum compute?

3. **Polynomial sequences.** The growth polynomials $\{P_r\}_{r \geq 1}$ form a polynomial sequence (a sequence of polynomials, one for each degree) with the recurrence $P_{r+1} = (1+n) P_r + n^r$. How does this sequence relate to other classical polynomial sequences (Appell sequences, Sheffer sequences) studied in the umbral calculus?

4. **Connection to the orders of topology.** The "orders of growth" classification ($\mathbb{S}_{\text{poly}} \subset \mathbb{I}_{\text{poly}} \subset \mathbb{U}_{\text{poly}}$) parallels the "orders of topology" framework developed in a companion paper, where increasing topological order corresponds to increasing degrees of freedom in cross-sectional geometry. Both frameworks describe hierarchies of structure that become progressively more continuous and general. Whether this parallel is superficial or reflects a deeper mathematical connection is an open question.

---

## 13. Summary

The growth decomposition,

$$x^r = \sum_{n=0}^{x-1} P_r(n) = \sum_{n=0}^{x-1} \sum_{j=0}^{r-1} \binom{r}{j}\, n^j$$

reveals Pascal's triangle as the universal encoder of how power functions grow step by step. The classical fact that "the sum of the first $x$ odd numbers is $x^2$" is not an isolated curiosity — it is the $r = 2$ case of a universal pattern. Every positive-integer power is the cumulative sum of a polynomial growth sequence whose coefficients are read directly from Pascal's triangle.

The discrete growth polynomials $P_r(n)$ converge to the continuous derivative $r\,n^{r-1}$ as resolution increases, bridging discrete and continuous calculus. The resulting hierarchy of representations — discrete sums for integer exponents, integrals for real exponents, exponentiation for complex exponents — defines the **orders of growth**: a classification of power functions by the level of accumulative machinery required to express them.

---

## Historical Note

The mathematical components of this framework are classical. The forward difference $\Delta[n^r] = (n+1)^r - n^r$ and its binomial expansion follow immediately from the binomial theorem and were understood in the development of finite difference calculus by James Gregory (1670) and Isaac Newton (*Principia Mathematica*, 1687; *Methodus Differentialis*). The connection between forward differences and falling factorials via Stirling numbers was established by James Stirling (*Methodus Differentialis*, 1730). The sums-of-powers problem was systematically addressed by Johann Faulhaber (*Academia Algebrae*, 1631) and placed on a general footing by Jacob Bernoulli (*Ars Conjectandi*, 1713). The algebraic foundations of finite difference calculus were developed into the umbral calculus by Gian-Carlo Rota, David Kahaner, and Andrew Odlyzko (1973) and by Steven Roman and Rota (1978). The modern standard reference for finite calculus and its relation to continuous calculus is Graham, Knuth, and Patashnik, *Concrete Mathematics* (1994), Section 2.6.

What is new in this paper is the synthesis: the explicit identification of Pascal's triangle as a universal growth encoder for power functions, the recursive structure $P_{r+1}(n) = (1+n)\,P_r(n) + n^r$ connecting successive growth polynomials, the polynomial set classification ($\mathbb{S}_{\text{poly}} \subset \mathbb{I}_{\text{poly}} \subset \mathbb{U}_{\text{poly}}$), and the unified presentation of discrete sums, integrals, and complex exponentiation as three "orders of growth" that form a natural hierarchy.

---

## References

- Bernoulli, J. *Ars Conjectandi*. Basel, 1713.
- Faulhaber, J. *Academia Algebrae*. Ulm, 1631.
- Graham, R. L., Knuth, D. E., and Patashnik, O. *Concrete Mathematics: A Foundation for Computer Science*. 2nd ed., Addison-Wesley, 1994.
- Jordan, C. *Calculus of Finite Differences*. Budapest, 1939.
- Knuth, D. E. "Johann Faulhaber and Sums of Powers." *Mathematics of Computation*, 61(203), 1993, pp. 277--294.
- Newton, I. *Philosophiae Naturalis Principia Mathematica*. London, 1687.
- Nicomachus of Gerasa. *Introductio Arithmeticae*. c. 100 AD. English translation: M. L. D'Ooge, University of Michigan Press, 1926.
- Roman, S. *The Umbral Calculus*. Academic Press, 1984.
- Roman, S. and Rota, G.-C. "The Umbral Calculus." *Advances in Mathematics*, 31, 1978, pp. 95--188.
- Rota, G.-C., Kahaner, D., and Odlyzko, A. "On the Foundations of Combinatorial Theory VIII: Finite Operator Calculus." *Journal of Mathematical Analysis and Applications*, 42(3), 1973, pp. 684--760.
- Stirling, J. *Methodus Differentialis*. London, 1730.
