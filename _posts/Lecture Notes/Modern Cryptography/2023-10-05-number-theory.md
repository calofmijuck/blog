---
share: true
toc: true
math: true
categories:
  - Lecture Notes
  - Modern Cryptography
tags:
  - lecture-note
  - cryptography
  - number-theory
  - security
title: 8. Number Theory
date: 2023-10-05
github_title: 2023-10-05-number-theory
---


## Background

### Number Theory

Let $n$ be a positive integer and let $p$ be prime.

> **Notation.** Let $\mathbb{Z}$ denote the set of integers. We will write $\mathbb{Z}_n = \left\lbrace 0, 1, \dots, n - 1 \right\rbrace$.

> **Definition.** Let $x, y \in \mathbb{Z}$. $\gcd(x, y)$ is the **greatest common divisor** of $x, y$. $x$ and $y$ are relatively prime if $\gcd(x, y) = 1$.

> **Definition.** The **multiplicative inverse** of $x \in \mathbb{Z}_n$ is an element $y \in \mathbb{Z}_n$ such that $xy = 1$ in $\mathbb{Z}_n$.

> **Lemma.** $x \in \mathbb{Z}_n$ has a multiplicative inverse if and only if $\gcd(x, n) = 1$.

> **Definition.** $\mathbb{Z}_n^\ast$ is the set of invertible elements in $\mathbb{Z}_n$. i.e, $\mathbb{Z}_n^\ast = \left\lbrace x \in \mathbb{Z}_n : \gcd(x, n) = 1 \right\rbrace$.

> **Lemma.** (Extended Euclidean Algorithm) For $x, y \in \mathbb{Z}$, there exists $a, b \in \mathbb{Z}$ such that $ax + by = \gcd(x, y)$.

### Group Theory

> **Definition.** A **group** is a set $G$ with a binary operation $* : G \times G \rightarrow G$, satisfying the following properties.
> 
> - $(\mathsf{G1})$ (Associative) $(a * b) * c = a * (b * c)$ for all $a, b, c \in G$.
> - $(\mathsf{G2})$ (Identity) $\exists e \in G$ such that for all $a\in G$, $e * a = a * e = a$.
> - $(\mathsf{G3})$ (Inverse) For each $a \in G$, $\exists x \in G$ such that $a * x = x * a = e$. In this case, $x = a^{-1}$.

> **Definition.** A group is **commutative** if $a * b = b * a$ for all $a, b \in G$.

> **Definition.** The **order** of a group is the number of elements in $G$, denoted as $\left\lvert G \right\lvert$.

> **Definition.** A set $H \subseteq G$ is a **subgroup** of $G$ if $H$ is itself a group under the operation of $G$. We write $H \leq G$.

> **Theorem.** (Lagrange) Let $G$ be a finite group and $H \leq G$. Then $\left\lvert H \right\lvert \mid \left\lvert G \right\lvert$.

*Proof*. All left cosets of $H$ have the same number of elements. A bijection between any two coset can be constructed. Cosets partition $G$, so $\left\lvert G \right\lvert$ is equal to the number of left cosets multiplied by $\left\lvert H \right\lvert$.

Let $G$ be a group.

> **Definition.** Let $g \in G$. The set $\left\langle g \right\rangle = \left\lbrace g^n : n \in \mathbb{Z} \right\rbrace$ is called the **cyclic subgroup generated by $g$**. The **order** of $g$ is the number of elements in $\left\langle g \right\rangle$, denoted as $\left\lvert g \right\lvert$.

> **Definition.** $G$ is **cyclic** if there exists $g \in G$ such that $G = \left\langle g \right\rangle$.

> **Theorem.** $\mathbb{Z}_p^\ast$ is cyclic.

*Proof*. $\mathbb{Z}_p$ is a finite field, so $\mathbb{Z}_p^\ast = \mathbb{Z}_p \setminus \left\lbrace 0 \right\rbrace$ is cyclic.

> **Theorem.** If $G$ is a finite group, then $g^{\left\lvert G \right\lvert} = 1$ for all $g \in G$. i.e, $\left\lvert g \right\lvert \mid \left\lvert G \right\lvert$.

*Proof*. Consider $\left\langle g \right\rangle \leq G$, then the result follows from Lagrange's theorem.

> **Corollary.** (Fermat's Little Theorem) If $x \in \mathbb{Z}_p^\ast$, $x^{p-1} = 1$.

*Proof*. $\mathbb{Z}_p^\ast$ has $p-1$ elements.

> **Corollary.** (Euler's Generalization) If $x \in \mathbb{Z}_n^\ast$, $x^{\phi(n)} = 1$.

*Proof*. $\mathbb{Z}_n^\ast$ has $\phi(n)$ elements, where $\phi(n)$ is the Euler's totient function.

---

Schemes such as Diffie-Hellman rely on the hardness of the DLP. So, *how hard is it*? How does one compute the discrete logarithm?

There are group-specific algorithms that exploit the algebraic features of the group, but we only cover generic algorithms, that works on any cyclic group. A trivial example would be the exhaustive search, where if $\left\lvert G \right\lvert = n$ and given a generator $g \in G$, find the discrete logarithm of $h \in G$ by computing $g^i$ for all $i = 1, \dots, n - 1$. Obviously, it has running time $\mathcal{O}(n)$. We can do better than this.

## Baby Step Giant Step Method (BSGS)

Let $G = \left\langle g \right\rangle$, where $g \in G$ has order $q$. $q$ need not be prime for this method. We are given $u = g^\alpha$, $g$, and $q$. Our task is to find $\alpha \in \mathbb{Z}_q$.

Set $m = \left\lceil \sqrt{q} \right\rceil$. $\alpha$ is currently unknown, but by the division algorithm, there exists integers $i,j$ such that $\alpha = i \cdot m + j$ and $0\leq i, j < m$. Then $u = g^\alpha = g^{i\cdot m + j} = g^{im} \cdot g^j$. Therefore,

$$
u(g^{-m})^i = g^j.
$$

Now, we compute the values of $g^j$ for $j = 0, 1,\dots, m - 1$ and keep a table of $(j, g^j)$ pairs. Next, compute $g^{-m}$ and for each $i$, compute $u(g^{-m})^{i}$ and check if this value is in the table. If a value is found, then we found $(i, j)$ such that $i \cdot m + j = \alpha$.

We see that this algorithm takes $2\sqrt{q}$ group operations on $G$ in the worst case, so the time complexity is $\mathcal{O}(\sqrt{q})$. However, to store the values of $(j, g^j)$ pairs, a lot of memory is required. The table must be large enough to contain $\sqrt{q}$ group elements, so the space complexity is also $\mathcal{O}(\sqrt{q})$.

To get around this, we can build a smaller table by choosing a smaller $m$. But then $0 \leq j < m$ but $i$ must be checked for around $q/m$ values.

There is actually an algorithm using constant space. **Pollard's Rho** algorithm takes $\mathcal{O}(\sqrt{q})$ times and $\mathcal{O}(1)$ space.

## Groups of Composite Order

In Diffie-Hellman, we only used large primes. There is a reason for using groups with prime order. We study what would happen if we used composite numbers.

Let $G$ be a cyclic group of composite order $n$. First, we start with a simple case.

### Prime Power Case: Order $n = q^e$

Let $G = \left\langle g \right\rangle$ be a cyclic group of order $q^e$.[^1] ($q > 1$, $e \geq 1$) We are given $g,q, e$ and $u = g^\alpha$ and we will find $\alpha$. ($0 \leq \alpha < q^e)$

For each $f = 0, \dots, e$, define $g_f = g^{(q^f)}$. Then

$$
(g_f)^{(q^{e-f})} = g^{(q^f) \cdot (q^{e-f})} = g^{(q^e)} = 1.
$$

So $g_f$ generates a cyclic subgroup of order $q^{e-f}$. In particular, $g_{e-1}$ generates a cyclic subgroup of order $q$. Using this fact, we will reduce the given problem into a discrete logarithm problem on a group having smaller order $q$.

We proceed with recursion on $e$. If $e = 1$, then $\alpha \in \mathbb{Z}_q$, so we have nothing to do. Suppose $e > 1$. Choose $f$ so that $1 \leq f \leq e-1$. We can write $\alpha = i\cdot q^f + j$, where $0 \leq i < q^{e-f}$ and $0 \leq j < g^f$. Then

$$
u = g^\alpha = g^{i \cdot q^f + j} = (g_f)^i \cdot g^j.
$$

Since $g_f$ has order $q^{e-f}$, exponentiate both sides by $q^{e-f}$ to get

$$
u^{(q^{e-f})} = (g_f)^{q^{e-f} \cdot i} \cdot g^{q^{e-f} \cdot j} = (g_{e-f})^j.
$$

Now the problem has been reduced to a discrete logarithm problem with base $g_{e-f}$, which has order $q^f$. We can compute $j$ using algorithms for discrete logarithms.

After finding $j$, we have

$$
u/g^j = (g_f)^i
$$

which is also a discrete logarithm problem with base $g_f$, which has order $q^{e-f}$. We can compute $i$ that satisfies this equation. Finally, we can compute $\alpha = i \cdot q^f + j$. We have reduced a discrete logarithm problem into two smaller discrete logarithm problems.

To get the best running time, choose $f \approx e/2$. Let $T(e)$ be the running time, then

$$
T(e) = 2T\left( \frac{e}{2} \right) + \mathcal{O}(e\log q).
$$

The $\mathcal{O}(e\log q)$ term comes from exponentiating both sides by $q^{e-f}$. Solving this recurrence gives

$$
T(e) = \mathcal{O}(e \cdot T_{\mathrm{base}} + e\log e \log q),
$$

where $T_\mathrm{base}$ is the complexity of the algorithm for the base case $e = 1$. $T_\mathrm{base}$ is usually the dominant term, since the best known algorithm takes $\mathcal{O}(\sqrt{q})$.

Thus, computing the discrete logarithm in $G$ is only as hard as computing it in the subgroup of prime order.

### General Case: Pohlig-Hellman Algorithm

Let $G = \left\langle g \right\rangle$ be a cyclic group of order $n = q_1^{e_1}\cdots q_r^{e_r}$, where the factorization of $n$ into distinct primes $q_i$ is given. We want to find $\alpha$ such that $g^\alpha = u$.

For $i = 1, \dots, r$, define $q_i^\ast = n / q_i^{e_i}$. Then $u^{q_i^\ast} = (g^{q_i^\ast})^\alpha$, where $g^{q_i^\ast}$ will have order $q_i^{e_i}$ in $G$. Now compute $\alpha_i$ using the algorithm for the prime power case.

Then for all $i$, we have $\alpha \equiv \alpha_i \pmod{q_i^{e_i}}$. We can now use the Chinese remainder theorem to recover $\alpha$. Let $q_r$ be the largest prime, then the running time is bounded by

$$
\sum_{i=1}^r \mathcal{O}(e_i T(q_i) + e_i \log e_i \log q_i) = \mathcal{O}(T(q_r) \log n + \log n \log \log n)
$$

group operations. Thus, we can conclude the following.

> The difficulty of computing discrete logarithms in a cyclic group of order $n$ is determined by the size of the largest prime factor.

### Consequences

- For a group with order $n = 2^k$, the Pohlig-Hellman algorithm will easily compute the discrete logarithm, since the largest prime factor is $2$. The DL assumption is false for this group.
- For primes of the form $p = 2^k + 1$, the group $\mathbb{Z}_p^\ast$ has order $2^k$, so the DL assumption is also false for these primes.
- In general, $G$ must have at least one large prime factor for the DL assumption to be true.
- By the Pohlig-Hellman algorithm, discrete logarithms in groups of composite order is a little harder than groups of prime order. So we often use a prime order group.

## Information Leakage in Groups of Composite Order

Let $G = \left\langle g \right\rangle$ be a cyclic group of composite order $n$. We suppose that $n = n_1n_2$, where $n_1$ is a small prime factor.

By the Pohlig-Hellman algorithm, the adversary can compute $\alpha_1 \equiv \alpha \pmod {n_1}$ by computing the discrete logarithm of $u^{n_2}$ with base $g^{n_2}$.

Consider $n_1 = 2$. Then the adversary knows whether $\alpha$ is even or not.

> **Lemma.** $\alpha$ is even if and only if $u^{n/2} = 1$.

*Proof*. If $\alpha$ is even, then $u^{n/2} = g^{\alpha n/2} = (g^{\alpha/2})^n = 1$, since the group has order $n$. Conversely, if $u^{n/2} = g^{\alpha n/2} = 1$, then the order of $g$ must divide $\alpha n/2$, so $n \mid (\alpha n /2)$ and $\alpha$ is even.

This lemma can be used to break the DDH assumption.

> **Lemma.** Given $u = g^\alpha$ and $v = g^\beta$, $\alpha\beta \in \mathbb{Z}_n$ is even if and only if $u^{n/2} = 1$ or $v^{n/2} = 1$.

*Proof*. $\alpha\beta$ is even if and only if either $\alpha$ or $\beta$ is even. By the above lemma, this is equivalent to $u^{n/2} = 1$ or $v^{n/2} = 1$.

Now we describe an attack for the DDH problem.

> 1. The adversary is given $(g^\alpha, g^\beta, g^\gamma)$.
> 2. The adversary computes the parity of $\gamma$ and $\alpha\beta$ and compares them.
> 3. The adversary outputs $\texttt{accept}$ if the parities match, otherwise output $\texttt{{reject}}$.

If $\gamma$ was chosen uniformly, then the adversary wins with probability $1/2$. But if $\gamma = \alpha\beta$, the adversary always wins, so the adversary has DDH advantage $1/2$.

The above process can be generalized to any groups with small prime factor. See Exercise 16.2[^2] Thus, this is another reason we use groups of prime order.

- DDH assumption does not hold in $\mathbb{Z}_p^\ast$, since its order $p-1$ is always even.
- Instead, we use a prime order subgroup of $\mathbb{Z}_p^\ast$ or prime order elliptic curve group.

## Summary of Discrete Logarithm Algorithms

|Name|Time Complexity|Space Complexity|
|:-:|:-:|:-:|
|BSGS|$\mathcal{O}(\sqrt{q})$|$\mathcal{O}(\sqrt{q})$|
|Pohlig-Hellman|$\mathcal{O}(\sqrt{q_\mathrm{max}}$|$\mathcal{O}(1)$|
|Pollard's Rho|$\mathcal{O}(\sqrt{q})$|$\mathcal{O}(1)$|

- In generic groups, solving the DLP requires $\Omega(\sqrt{q})$ operations.
	- By *generic groups*, we mean that only group operations and equality checks are allowed. Algebraic properties are not used.
- Thus, we use a large prime $q$ such that $\sqrt{q}$ is large enough.

## Candidates of Discrete Logarithm Groups

We need groups of order prime, and we cannot use $\mathbb{Z}_p^\ast$ as itself. We have two candidates.

- Use a subgroup of $\mathbb{Z}_p^\ast$ having prime order $q$ such that $q \mid (p-1)$ as in Diffie-Hellman.
- Elliptic curve group modulo $p$.

### Reduced Residue Class $\mathbb{Z}_p^\ast$

There are many specific algorithms for discrete logarithms on $\mathbb{Z}_p^\ast$.

- Index-calculus
- Elliptic-curve method
- Special number-field sieve (SNFS)
- **General number-field sieve** (GNFS)

GNFS running time is dominated by the term $\exp(\sqrt[3]{\ln p})$. If we let $p$ to be an $n$-bit prime, then the complexity is $\exp(\sqrt[3]{n})$. Suppose that GNFS runs in time $T$ for prime $p$. Since $\sqrt[3]{2} \approx 1.26$, doubling the number of bits will increase the running time of GNFS to $T^{1,26}$.

Compare this with symmetric ciphers such as AES, where doubling the key size squares the amount of work required.[^3] NIST and Lenstra recommends the size of primes that gives a similar level of security to that of symmetric ciphers.

|Symmetric key length|Size of prime (NIST)|Size of prime (Lenstra)|
|:-:|:-:|:-:|
|80|1024|1329|
|128|3072|4440|
|256|15360|26268|

All sizes are in bits. Thus we need a very large prime, for example $p > 2^{2048}$, for security these days.

### Elliptic Curve Group over $\mathbb{Z}_p$

Currently, the best-known attacks are generic attacks, so we can use much smaller parameters than $\mathbb{Z}_p^\ast$. Often the groups have sizes about $2^{256}$, $2^{384}$, $2^{512}$.

[^1]: We didn't require $q$ to be prime!
[^2]: A Graduate Course in Applied Cryptography
[^3]: Recall that the best known attack was only 4 times faster than brute-force search.