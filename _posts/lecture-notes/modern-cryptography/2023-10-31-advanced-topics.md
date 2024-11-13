---
share: true
toc: true
math: true
categories:
  - Lecture Notes
  - Modern Cryptography
path: _posts/lecture-notes/modern-cryptography
tags:
  - lecture-note
  - cryptography
  - security
title: 11. Advanced Topics
date: 2023-10-31
github_title: 2023-10-31-advanced-topics
---

## Ciphertext Indistinguishability

- By **Shafi Goldwasser** and **Silvio Micali**
	- Turing Award in 2012

An adversary should not be able to...

- **(Semantic Security)** gain any partial information about a secret.
- **(Ciphertext Indistinguishability)** distinguish pairs of ciphertexts based on the chosen messages.

They showed that

- These two definitions are equivalent under chosen-plaintext attack.
- Encryption schemes must be randomized.

> **Definition.** A symmetric key encryption scheme $E$ is **semantically secure** if for any efficient adversary $\mc{A}$, there exists an efficient $\mc{A}'$ such that for any efficiently computable functions $f$ and $h$,
> 
> $$
> \bigg\lvert \Pr\left[ \mc{A}\big( E(k, m), h(m) \big) = f(m) \right] - \Pr\left[ \mc{A}'\big( h(m) \big) = f(m) \right] \bigg\lvert
> $$
> 
> is negligible.

## Commitment Schemes

A commitment scheme is for committing a value, and opening it later. The committed value cannot be forged.

> **Definition.** A **commitment scheme** for a finite message space $\mc{M}$ is a pair of efficient algorithms $\mc{C} = (C, V)$ satisfying the following.
> 
> - For a message $m \in \mc{M}$ to be committed, $(c, o) \la C(m)$, where $c$ is the **commitment string**, and $o$ is an **opening string**.
> - $V$ is a deterministic algorithm that $V(m, c, o)$ is either $\texttt{accept}$ or $\texttt{reject}$.
> - **Correctness**: for all $m \in \mc{M}$, if $(c, o) \la C(m)$ then $V(m, c, o) = \texttt{accept}$.

Suppose Alice wants to commit a message $m$. She computes $(c, o) \la C(m)$, and sends the commitment string $c$ to Bob, and keeps the opening string $o$ to herself. After some time, Alice sends the opening string $o$ to open the commitment, then Bob will verify the commitment by computing $V(m, c, o)$.

### Secure Commitment Schemes

The scheme must satisfy the following properties. First, the commitment must open to a single message. This is called the **binding** property. Next, the commitment must not reveal any information about the message. This is called the **hiding** property.

> **Definition.** A commitment scheme $\mc{C} = (C, V)$ is **binding** if for every efficient adversary $\mc{A}$ that outputs a $5$-tuple $(c, m _ 1, o _ 1, m _ 2, o _ 2)$, the probability
> 
> $$
> \Pr[m _ 1 \neq m _ 2 \land V(m _ 1, c, o _ 1) = V(m _ 2, c, o _ 2) = \texttt{{accept}}]
> $$
> 
> is negligible.

The hiding property is defined as a security game.

> **Definition.** Let $\mc{C} = (C, V)$ be a commitment scheme. Given an adversary $\mc{A}$, define two experiments.
> 
> **Experiment $b$**.
> 1. $\mc{A}$ sends $m _ 0, m _ 1 \in \mc{M}$ to the challenger.
> 2. The challenger computes $(c, o) \la C(m _ b)$ and sends $c$ to $\mc{A}$.
> 3. $\mc{A}$ computes and outputs $b' \in \braces{0, 1}$.
> 
> Let $W _ b$ be the event that $\mc{A}$ outputs $1$ in experiment $b$. The **advantage** of $\mc{A}$ with respect to $\mc{C}$ is defined as
> 
> $$
> \Adv{\mc{A}, \mc{C}} = \abs{\Pr[W _ 0] - \Pr[W _ 1]}.
> $$
> 
> If the advantage is negligible for all efficient adversaries $\mc{A}$, then the commitment scheme $\mc{C}$ has the **hiding** property.

Next, the definition of secure commitment schemes.

> **Definition.** A commitment scheme $\mc{C} = (C, V)$ is **secure** if it is both hiding and binding.

### Non-binding Encryption Schemes

A semantically secure cipher does not always yield a secure commitment scheme. One might be tempted to use a secure cipher $(E, D)$ as follows.

- For $m \in \mc{M}$, choose $k \la \mc{K}$ and set $\big( E(k, m), k \big) \la C(m)$.
- $V(m, c, k)$ accepts if and only if $D(k, c) = m$.

However, it may be feasible to find another $k' \in \mc{K}'$ such that $D(k, c) \neq D(k', c)$. As an example, consider the one-time pad. It is easy for the committer to manipulate the message. $c = m \oplus k$, so later set $k' = k \oplus m \oplus m'$ as the opening string, then $c \oplus k' = m'$, resulting in a different message.

## Constructions of Commitment Schemes

### Commitment from Secure PRGs

To commit a bit, we can use a secure PRG. The following is due to Naor.

> Let $G : \mc{S} \ra \mc{R}$ be a secure PRG where $\left\lvert \mc{R} \right\lvert \geq \left\lvert \mc{S} \right\lvert^3$ and $\mc{R} = \braces{0, 1}^n$. Suppose that Bob wants to commit a bit $b _ 0 \in \braces{0, 1}$.
> 
> 1. Alice chooses a random $r \in \mc{R}$ and sends it to Bob.
> 2. Bob chooses a random $s \in \mc{S}$ and computes $c \la C(s, r, b _ 0)$, where
> 
> 	$$
> 	C(s, r, b _ 0) = \begin{cases} G(s) & (b _ 0 = 0) \\ G(s) \oplus r & (b _ 0 = 1). \end{cases}
> 	$$
> 
> 	Then Bob outputs $(c, s)$ as the commitment and the opening string.
> 3. During opening, Bob sends $(b _ 0, s)$ to Alice.
> 4. Alice accepts if and only if $C(s, r, b _ 0) = c$.

Correctness is obvious, since Alice recomputes $C(s, r, b _ 0)$.

The hiding property follows since $G(s)$ and $G(s) \oplus r$ are indistinguishable if $G$ is a secure PRG.

The binding property follows if $1 / \left\lvert \mc{S} \right\lvert$ is negligible. For Bob to open $c$ as both $0$ and $1$, he must find two seeds $s _ 0, s _ 1 \in \mc{S}$ such that $c = G(s _ 0) = G(s _ 1) \oplus r$. Then $r = G(s _ 0) \oplus G(s _ 1)$. There are at most $\left\lvert \mc{S} \right\lvert^2$ possible $r \in \mc{R}$ values that this can happen. The probability that Alice chooses such $r$ is

$$
\left\lvert \mc{S} \right\lvert^2 / \left\lvert \mc{R} \right\lvert \leq \left\lvert \mc{S} \right\lvert^2 / \left\lvert \mc{S} \right\lvert^3 = 1 / \left\lvert \mc{S} \right\lvert
$$

by assumption.

The downside of the above protocol is that it has to be interactive.

#### Coin Flipping Protocol

A bit commitment scheme can be used for a **coin flipping protocol**. Suppose that Alice and Bob are flipping coins, when they are physically distant from each other.

> 1. Bob chooses a random bit $b _ 0 \la \braces{0, 1}$.
> 2. Execute the commitment protocol.
> 	- Alice obtains a commitment string $c$ of $b _ 0$.
> 	- Bob keeps an opening string $o$.
> 3. Alice chooses a random bit $b _ 1 \la \braces{0, 1}$, and sends it to Bob.
> 4. Bob reveals $b _ 0$ and $s$ to Alice, she verifies that $c$ is valid.
> 5. The final outcome is $b = b _ 0 \oplus b _ 1$.

After step $2$, Alice has no information about $b _ 0$ because of the hiding property. Her choice of $b _ 1$ is unbiased, and cannot affect the final outcome. Next, in step $4$, $b _ 0$ cannot be manipulated by the binding property.

Thus, $b _ 0$ and $b _ 1$ are both random, so $b$ is either $0$ or $1$ each with probability $1/2$.[^1]

### Commitment Scheme from Hashing

> Let $H : \mc{X} \ra \mc{Y}$ be a collision resistant hash function, where $\mc{X} = \mc{M} \times \mc{R}$. $\mc{M}$ is the message space, and $\mc{R}$ is a finite nonce space. For $m \in \mc{M}$, the derived commitment scheme $\mc{C} _ H = (C, V)$ is defined as follows.
> 
> - $C(m)$: choose random $o \la \mc{R}$, set $c = H(m, o)$ and output $(c, o)$.
> - $V(m, c, o)$: output $\texttt{accept}$ if and only if $c = H(m, o)$.

Correctness is obvious.

The binding property follows since $H$ is collision resistant. If it is easy to find a $5$-tuple $(c, m _ 1, o _ 1, m _ 2, o _ 2)$ such that $c = H(m _ 1, o _ 1) = H(m _ 2, o _ 2)$, $H$ is not collision resistant.

The hiding property follows if $H$ is modeled as a random oracle, or has a property called **input hiding**. For adversarially chosen $m _ 1, m _ 2 \in \mc{M}$ and random $o \la \mc{R}$, the distributions of $H(m _ 1, o)$ and $H(m _ 2, o)$ are computationally indistinguishable.

Additionally, this scheme is **non-malleable** if $H$ is modeled as a random oracle and $\mc{Y}$ is sufficiently large.[^2]

### Commitment Scheme from Discrete Logarithms

> Let $G = \left\langle g \right\rangle$ be a cyclic group of prime order $q$. Let $h$ be chosen randomly from $G$.
> 
> - $C(m)$: choose random $o \la \mathbb{Z} _ q$ and $c \la g^m h^o$ and return $(c, o)$.
> - $V(m, c, o)$: output $\texttt{accept}$ if and only if $c = g^m h^o$.

Correctness is obvious.

The binding property follows from the DL assumption. If an adversary finds $m _ 1, m _ 2$, $o _ 1, o _ 2$ such that $c = g^{m _ 1} h^{o _ 1} = g^{m _ 2} h^{o _ 2}$, then $h = g^{(m _ 2 - m _ 1)/(o _ 1 - o _ 2)}$, solving the discrete logarithm problem for $h$.

The hiding property follows since $h$ is uniform in $G$ and $o$ is also uniform in $\mathbb{Z} _ q$. Then $g^m h^o$ is uniform in $G$, not revealing any information.

## Post Quantum Cryptography

Quantum computers use **qubits** and **quantum gates** for computation. A **qubit** is a *quantum bit*, a **superposition** of two states $\ket{0}$ and $\ket{1}$.

$$
\ket{\psi} = \alpha \ket{0} + \beta \ket{1}
$$

where $\alpha, \beta \in \mathbb{C}$ and $\left\lvert \alpha \right\lvert^2 + \left\lvert \beta \right\lvert^2 = 1$. The quantum gates are usually orthogonal matrices.

The *superposition* may give the false impression that a quantum computer tries all possible solutions in parallel, but the actual magic comes from **complex amplitudes**.

Quantum computers use **quantum interference**, carefully choreograph computations so that wrong answers *cancel out* their amplitudes, while correct answers combine. This process increases the probability of measuring correct results. Naturally, only a few special problems allow this choreograph.

A scheme is **post-quantum secure** if it is secure against an adversary who has access to a quantum computer. Post-quantum cryptography is about classical algorithms that are believed to withstand quantum attacks.

AES is probably safe, since it still takes $\mc{O}(2^{n/2})$ to solve it. (Grover's algorithm) Also, lattice-based cryptography is another candidate.

## Shor's Algorithm

But factorization and discrete logarithms are not safe. The core idea is that a quantum computer is very good at detecting periodicity. This is done by using the **quantum Fourier transform** (QFT).

### Quantum Factorization

Let $n \in \mathbb{Z}$ and $0\neq g \in \mathbb{Z} _ n$. Let $\gamma _ g : \mathbb{Z} \ra \mathbb{Z} _ n$ be defined as $\gamma _ g(\alpha) = g^\alpha$. This function is periodic, since $g^{\phi(n)} = 1$ by Euler's generalization. Also, the order of $g$ will certainly divide the period.

Thus, find a period $p$, and let $t$ be the smallest positive integer such that $g^{p/2^t} \neq 1$. Then $\gcd(n, g^{p/2^t} - 1)$ is a non-trivial factor of $n$ with probability about $1/2$ over the choice of $g$. See Exercise 16.10.[^3]

Shor's algorithm factors $n$ in $\mc{O}(\log^3 n)$ time. RSA is not a secure one-way trapdoor function for quantum computers.

### Quantum Discrete Logarithms

Let $G = \left\langle g \right\rangle$ be a cyclic group of prime order $q$. Let $u = g^\alpha$. Consider the function $f : \mathbb{Z}^2 \ra G$ defined as

$$
f(\gamma, \delta) = g^\gamma \cdot u^\delta.
$$

The period of this function is $(\alpha, -1)$, since for all $(\gamma, \delta) \in \mathbb{Z}^2$,

$$
f(\gamma + \alpha, \delta - 1) = g^{\gamma} \cdot g^\alpha \cdot u^\delta \cdot u^{-1} = g^\gamma \cdot u^\delta = f(\gamma, \delta).
$$

This period can be found in $\mc{O}(\log^3 q)$ time. The DL assumption is false for quantum computers.

(Detailed explanation to be added...)

[^1]: There is one caveat. Bob gets to know the final result before Alice. If the outcome is not what he desired, he could abort the protocol in some way, like sending an invalid $c$, and go over the whole process again.
[^2]: A commitment scheme is **malleable** if a commitment $c = (c _ 1, c _ 2)$ of a message $m$ can be transformed into a commitment $c' = (c _ 1, c _ 2 + \delta)$ of a message $m + \delta$.
[^3]: A Graduate Course in Applied Cryptography.
