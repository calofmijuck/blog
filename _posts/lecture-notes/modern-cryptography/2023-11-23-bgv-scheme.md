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
title: 17. BGV Scheme
date: 2023-11-23
github_title: 2023-11-23-bgv-scheme
---

## Homomorphisms

> **Definition.** Let $(X, \ast), (Y, \ast')$ be sets equipped with binary operations $\ast$, $\ast'$. A map $\varphi : X \ra Y$ is said to be a **homomorphism** if
> 
> $$
> \varphi(a \ast b) = \varphi(a) \ast' \varphi(b)
> $$
> 
> for all $a, b \in X$.

A homomorphism *sort of* preserves the structure between two sets.[^1]

We will mainly consider **additive homomorphisms** where

$$
\varphi(a + b) = \varphi(a) + \varphi(b),
$$

and **multiplicative homomorphisms** where

$$
\varphi(ab) = \varphi(a)\varphi(b).
$$

## Homomorphic Encryption

> **Definition.** A **homomorphic encryption scheme** defined over $\mc{M}$ consists of an encryption algorithm $E$ and a decryption algorithm $D$ such that
> 
> $$
> D\big( E(x) + E(y) \big) = x + y
> $$
> 
> or
> 
> $$
> D\big( E(x) \cdot E(y) \big) = x \cdot y.
> $$

The **decryption $D$ is a homomorphism**. From ciphertexts of $x$ and $y$, this scheme can compute the ciphertext of $x + y$ or $x \cdot y$.

There are mainly $3$ categories of homomorphic encryption.

- **Partial** Homomorphic Encryption
	- These schemes can evaluate *some* functions on encrypted data.
	- Textbook RSA had a *homomorphic property*.
- **Somewhat** Homomorphic Encryption (SHE)
	- Both addition and multiplication are supported.
	- But there is a limit on the number of operations.
- **Fully** Homomorphic Encryption (FHE)
	- Any function can be evaluated on encrypted data.
	- There is a method called *bootstrapping* that compiles SHE into FHE.

### A Warm-up Scheme

This is a sample scheme, which is insecure.

> Choose parameters $n$ and $q$ as security parameters.
> 
> 1. Set secret key $\bf{s} = (s_1, \dots, s_n) \in \Z^n$.
> 2. For message $m \in \Z_q$, encrypt it as follows.
> 	- Randomly choose $\bf{a} = (a_1, \dots, a_n) \la \Z_q^n$.
> 	- Compute $b = -\span{\bf{a}, \bf{s}} + m \pmod q$.
> 	- Output ciphertext $\bf{c} = (b, \bf{a}) \in \Z_q^{n+1}$.
> 3. To decrypt $\bf{c}$, compute $m = b + \span{\bf{a}, \bf{s}} \pmod q$.

Correctness is trivial. Also, this encryption algorithm has the *additive homomorphism* property. If $b_1, b_2$ are encryptions of $m_1, m_2$, then

$$
b_1 = -\span{\bf{a}_1, \bf{s}} + m_1, \quad b_2 = -\span{\bf{a}_2, \bf{s}} + m_2
$$

in $\Z_q$. Thus,

$$
b_1 + b_2 = -\span{\bf{a}_1 + \bf{a}_2, \bf{s}} + m_1 + m_2.
$$

Decrypting the ciphertext $(b_1 + b_2, \bf{a}_1 + \bf{a}_2)$ will surely give $m_1 + m_2$.

But this scheme is not secure. After $n$ queries, the plaintext-ciphertext pairs can be transformed into a linear system of equations

$$
\bf{b} = -A \bf{s} + \bf{m},
$$

where $\bf{a}_i$ are in the rows of $A$. This system can be solved for $\bf{s}$ with non-negligible probability.[^2]

## Lattice Cryptography

Recall that schemes like RSA and ElGamal rely on the hardness of computational problems. The hardness of those problems make the schemes secure. There are other (known to be) *hard* problems using **lattices**, and recent homomorphic encryption schemes use **lattice-based** cryptography.

> **Definition.** For $\bf{b}_i \in \Z^n$ for $i = 1, \dots, n$, let $B = \braces{\bf{b}_1, \dots, \bf{b}_n}$ be a basis. The set
> 
> $$
> L = \braces{\sum_{i=1}^n a_i\bf{b}_i : a_i \in \Z}
> $$
> 
> is called a **lattice**. The set $B$ is a basis over $L$.

It is essentially a linear combination of basis elements, with *integer coefficients*.

### Bounded Distance Decoding Problem (BDD)

Let $L$ be a lattice with basis $B$. Given

$$
\bf{t} = B\bf{u} + \bf{e} \notin L
$$

for a small error $\bf{e}$, the problem is to find the closest lattice point $B\bf{u} \in L$.

It is known that all (including quantum) algorithms for solving BDD have costs $2^{\Omega(n)}$.

This problem is easy when we have a *short* basis, where the angles between vectors are closer to $\pi/2$. For example, given $\bf{t}$, find $a_i \in \R$ such that

$$
\bf{t} = a_1 \bf{b}_1 + \cdots a_n \bf{b}_n
$$

and return $B\bf{u}$ as

$$
B\bf{u} = \sum_{i=1}^n \lfloor a_i \rceil \bf{b}_i.
$$

Then this $B\bf{u} \in L$ is pretty close to $\bf{t} \notin L$.

## Learning with Errors Problem (LWE)

This is the problem we will mainly use for homomorphic schemes.

Let $\rm{LWE}_{n, q, \sigma}(\bf{s})$ denote the LWE distribution, where
- $n$ is the number of dimensions,
- $q$ is the modulus,
- $\sigma$ is the standard deviation of error.

Also $D_\sigma$ denotes the discrete gaussian distribution with standard deviation $\sigma$.

> Let $\bf{s} = (s_1, \dots, s_n) \in \Z_q^n$ be a secret.
> 
> - Sample $\bf{a} = (a_1, \dots, a_n) \la \Z_q^n$ and $e \la D_\sigma$.
> - Compute $b = \span{\bf{a}, \bf{s}} + e \pmod q$.
> - Output $(b, \bf{a}) \in \Z_q^{n+1}$.
> 
> This is called a **LWE instance**.

### Search LWE Problem

> Given many samples from $\rm{LWE}_{n, q, \sigma}(\bf{s})$, find $\bf{s}$.

### Decisional LWE Problem (DLWE)

> Distinguish two distributions $\rm{LWE}_{n, q, \sigma}(\bf{s})$ and $U(\Z_q^{n+1})$.

It is known that the two versions of LWE problem are **equivalent** when $q$ is a prime bounded by some polynomial in $n$.

LWE problem can be turned into **assumptions**, just like the DL and RSA problems. As in DL and RSA, the LWE problem is not hard for any parameters $n, q$. The problem is harder if $n$ is large and $q$ is small.

## The BGV Scheme

**BGV scheme** is by Brakerski-Gentry-Vaikuntanathan (2012). The scheme is defined over the finite field $\Z_p$ and can perform arithmetic in $\Z_p$.

> Choose security parameters $n$, $q$ and $\sigma$. It is important that $q$ is chosen as an **odd** integer.
> 
> **Key Generation**
> - Set secret key $\bf{s} = (s_1, \dots, s_n) \in \Z^n$.
> 
> **Encryption**
> - Sample $\bf{a} \la \Z_q^n$ and $e \la D_\sigma$.
> - Compute $b = -\span{\bf{a}, \bf{s}} + m + 2e \pmod q$.
> - Output ciphertext $\bf{c} = (b, \bf{a}) \in \Z_q^{n+1}$.
> 
> **Decryption**
> - Compute $r = b + \span{\bf{a}, \bf{s}} \pmod q$.
> - Output $m = r \pmod 2$.

Here, it can be seen that

$$
r = m + 2e \pmod q.
$$

For correctness, $e \ll q$, and

$$
\abs{r} = \abs{m + 2e} < \frac{1}{2}q.
$$

Under the LWE assumption, it can be proven that the scheme is semantically secure, i.e,

$$
E(\bf{s}, 0) \approx_c E(\bf{s}, 1).
$$

### Addition in BGV

Addition is easy!

> Let $\bf{c} = (b, \bf{a})$ and $\bf{c}' = (b', \bf{a}')$ be encryptions of $m, m' \in \braces{0, 1}$. Then, $\bf{c}_\rm{add} = \bf{c} + \bf{c}'$ is an encryption of $m + m'$.

*Proof*. Decrypt $\bf{c}_\rm{add} = (b + b', \bf{a} + \bf{a}')$. If

$$
r = b + \span{\bf{a}, \bf{s}} = m + 2e \pmod q
$$

and

$$
r' = b' + \span{\bf{a}', \bf{s}} = m' + 2e' \pmod q,
$$

then we have

$$
r_\rm{add} = b + b' + \span{\bf{a} + \bf{a}', \bf{s}} = r + r' = m + m' + 2(e + e') \pmod q.
$$

If $\abs{r + r'} < q/2$, then $m + m' = r_\rm{add} \pmod 2$.

### Multiplication in BGV

#### Tensor Product

For multiplication, we need **tensor products**.

> **Definition.** Let $\bf{a} = (a_1, \dots, a_n)^\top, \bf{b} = (b_1, \dots, b_n)^\top$ be vectors. Then the **tensor product** $\bf{a} \otimes \bf{b}$ is a vector with $n^2$ dimensions such that
> 
> $$
> \bf{a} \otimes \bf{b} = \big( a_i \cdot b_j \big)_{1 \leq i, j \leq n}.
> $$

We will use the following property.

> **Lemma.** Let $\bf{a}, \bf{b}, \bf{c}, \bf{d}$ be $n$-dimensional vectors. Then,
> 
> $$
> \span{\bf{a}, \bf{b}} \cdot \span{\bf{c}, \bf{d}} = \span{\bf{a} \otimes \bf{c}, \bf{b} \otimes \bf{d}}.
> $$

*Proof*. Denote the components as $a_i, b_i, c_i, d_i$.

$$
\begin{aligned}
\span{\bf{a} \otimes \bf{c}, \bf{b} \otimes \bf{d}} &= \sum_{i=1}^n\sum_{j=1}^n a_ic_j \cdot b_id_j \\
&= \paren{\sum_{i=1}^n a_ib_i} \paren{\sum_{j=1}^n c_j d_j} =  \span{\bf{a}, \bf{b}} \cdot \span{\bf{c}, \bf{d}}.
\end{aligned}
$$

#### Multiplication

Let $\bf{c} = (b, \bf{a})$ and $\bf{c}' = (b', \bf{a}')$ be encryptions of $m, m' \in \braces{0, 1}$. Since

$$
r = b + \span{\bf{a}, \bf{s}} = m + 2e \pmod q
$$

and

$$
r' = b' + \span{\bf{a}', \bf{s}} = m' + 2e' \pmod q,
$$

we have that

$$
r_\rm{mul} = rr' = (m + 2e)(m' + 2e') = mm' + 2e\conj \pmod q.
$$

So $mm' = r_\rm{mul} \pmod 2$ if $e\conj$ is small.

However, to compute $r_\rm{mul} = rr'$ from the ciphertext,

$$
\begin{aligned}
r_\rm{mul} &= rr' = (b + \span{\bf{a}, \bf{s}})(b' + \span{\bf{a}', \bf{s}}) \\
&= bb' + \span{b\bf{a}' + b' \bf{a}, \bf{s}} + \span{\bf{a} \otimes \bf{a}', \bf{s} \otimes \bf{s}'}.
\end{aligned}
$$

Thus we define $\bf{c}_\rm{mul} = (bb', b\bf{a}' + b' \bf{a}, \bf{a} \otimes \bf{a}')$, then this can be decrypted with $(1, \bf{s}, \bf{s} \otimes \bf{s})$ by the above equation.

> Let $\bf{c} = (b, \bf{a})$ and $\bf{c}' = (b', \bf{a}')$ be encryptions of $m, m'$. Then,
> 
> $$
> \bf{c}_\rm{mul} = \bf{c} \otimes \bf{c}' = (bb', b\bf{a}' + b' \bf{a}, \bf{a} \otimes \bf{a}')
> $$
> 
> is an encryption of $mm'$ with $(1, \bf{s}, \bf{s} \otimes \bf{s})$.

Not so simple as addition, we need $\bf{s} \otimes \bf{s}$.

#### Problems with Multiplication

The multiplication described above has two major problems.

- The dimension of the ciphertext has increased to $n^2$.
	- At this rate, multiplications get inefficient very fast.
- The *noise* $e\conj$ grows too fast.
	- For correctness, $e\conj$ must be small compared to $q$, but it grows exponentially.
	- We can only perform $\mc{O}(\log q)$ multiplications.

### Dimension Reduction

First, we reduce the ciphertext dimension. In the ciphertext $\bf{c}_\rm{mul} = (bb', b\bf{a}' + b' \bf{a}, \bf{a} \otimes \bf{a}')$, $\bf{a} \otimes \bf{a}'$ is causing the problem, since it must be decrypted with $\bf{s} \otimes \bf{s}'$.

Observe that the following dot product is calculated during decryption.

$$
\tag{1} \span{\bf{a} \otimes \bf{a}', \bf{s} \otimes \bf{s}'} = \sum_{i = 1}^n \sum_{j=1}^n a_i a_j' s_i s_j.
$$

The above expression has $n^2$ terms, so they have to be manipulated. The idea is to switch these terms as encryptions of $\bf{s}$, instead of $\bf{s} \otimes \bf{s}'$.

Thus we use encryptions of $s_is_j$ by $\bf{s}$. If we have ciphertexts of $s_is_j$, we can calculate the expression in $(1)$ since this scheme is *homomorphic*. Then the ciphertext can be decrypted only with $\bf{s}$, as usual. This process is called **relinearization**, and the ciphertexts of $s_i s_j$ are called **relinearization keys**.

#### First Attempt

> **Relinearization Keys**: for $1 \leq i, j \leq n$, perform the following.
> - Sample $\bf{u}_{i, j} \la \Z_q^{n}$ and $e_{i, j} \la D_\sigma$.
> - Compute $v_{i, j} = -\span{\bf{u}_{i, j}, \bf{s}} + s_i s_j + 2e_{i, j} \pmod q$.
> - Output $\bf{w}_{i, j} = (v_{i, j}, \bf{u}_{i, j})$.
> 
> **Linearization**: given $\bf{c}_\rm{mul} = (bb', b\bf{a}' + b' \bf{a}, \bf{a} \otimes \bf{a}')$ and $\bf{w}_{i, j}$ for $1 \leq i, j \leq n$, output the following.
> 
> $$
> \bf{c}_\rm{mul}^\ast = (b_\rm{mul}^\ast, \bf{a}_\rm{mul}^\ast) = (bb', b\bf{a}' + b'\bf{a}) + \sum_{i=1}^n \sum_{j=1}^n a_i a_j' \bf{w}_{i, j} \pmod q.
> $$

Note that the addition $+$ is the addition of two $(n+1)$-dimensional vectors. By plugging in $\bf{w}_{i, j} = (v_{i, j}, \bf{u}_{i, j})$, we actually have

$$
b_\rm{mul}^\ast = bb' + \sum_{i=1}^n \sum_{j=1}^n a_i a_j' v_{i, j}
$$

and

$$
\bf{a}_\rm{mul}^\ast = b\bf{a}' + b'\bf{a} + \sum_{i=1}^n \sum_{j=1}^n a_i a_j' \bf{u}_{i, j}.
$$

Now we check correctness. $\bf{c}_\rm{mul}^\ast$ should decrypt to $mm'$ with only $\bf{s}$.

$$
\begin{aligned}
b_\rm{mul}^\ast + \span{\bf{a}_\rm{mul}^\ast, \bf{s}} &= bb' + \sum_{i=1}^n \sum_{j=1}^n a_i a_j' v_{i, j} +  \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \sum_{i=1}^n \sum_{j=1}^n a_i a_j' \span{\bf{u}_{i, j}, \bf{s}} \\
&= bb' + \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \sum_{i=1}^n \sum_{j=1}^n a_i a_j' \paren{v_{i, j} + \span{\bf{u}_{i, j}, \bf{s}}}.
\end{aligned}
$$

Since $v_{i, j} + \span{\bf{u}_{i, j}, \bf{s}} = s_i s_j + 2e_{i, j} \pmod q$, the above expression further reduces to

$$
\begin{aligned}
&= bb' + \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \sum_{i=1}^n \sum_{j=1}^n a_i a_j' \paren{s_i s_j + 2e_{i, j}} \\
&= bb' + \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \span{\bf{a} \otimes \bf{a}', \bf{s} \otimes \bf{s}'} + 2\sum_{i=1}^n\sum_{j=1}^n a_i a_j' e_{i, j} \\
&= rr' + 2e\conj \pmod q,
\end{aligned}
$$

and we have an encryption of $mm'$.

However, we require that

$$
e\conj = \sum_{i=1}^n \sum_{j=1}^n a_i a_j' e_{i, j} \ll q
$$

for correctness. It is highly unlikely that this relation holds, since $a_i a_j'$ will be large. They are random elements of $\Z_q$ after all, so the size is about $\mc{O}(n^2 q)$.

#### Relinearization

We use a method to make $a_i a_j'$ smaller. The idea is to use the binary representation.

Let $a[k] \in \braces{0, 1}$ denote the $k$-th least significant bit of $a \in \Z_q$. Then we can write

$$
a = \sum_{0\leq k<l} 2^k \cdot a[k]
$$

where $l = \ceil{\log q}$. Then we have

$$
a_i a_j' s_i s_j = \sum_{0\leq k <l} (a_i a_j')[k] \cdot 2^k s_i s_j,
$$

so instead of encryptions of $s_i s_j$, we use encryptions of $2^k s_i s_j$.

For convenience, let $a_{i, j} = a_i a_j'$. Now we have triple indices including $k$.

> **Relinearization Keys**: for $1 \leq i, j \leq n$ and $0 \leq k < \ceil{\log q}$, perform the following.
> - Sample $\bf{u}_{i, j, k} \la \Z_q^{n}$ and $e_{i, j, k} \la D_\sigma$.
> - Compute $v_{i, j, k} = -\span{\bf{u}_{i, j, k}, \bf{s}} + 2^k \cdot s_i s_j + 2e_{i, j, k} \pmod q$.
> - Output $\bf{w}_{i, j, k} = (v_{i, j, k}, \bf{u}_{i, j, k})$.
> 
> **Linearization**: given $\bf{c}_\rm{mul} = (bb', b\bf{a}' + b' \bf{a}, \bf{a} \otimes \bf{a}')$, $\bf{w}_{i, j, k}$ for $1 \leq i, j \leq n$ and $0 \leq k < \ceil{\log q}$, output the following.
> 
> $$
> \bf{c}_\rm{mul}^\ast = (b_\rm{mul}^\ast, \bf{a}_\rm{mul}^\ast) = (bb', b\bf{a}' + b'\bf{a}) + \sum_{i=1}^n \sum_{j=1}^n \sum_{k=0}^{\ceil{\log q}} a_{i, j}[k] \bf{w}_{i, j, k} \pmod q.
> $$

Correctness can be checked similarly. The bounds for summations are omitted for brevity. They range from $1 \leq i, j \leq n$ and $0 \leq k < \ceil{\log q}$.

$$
\begin{aligned}
b_\rm{mul}^\ast + \span{\bf{a}_\rm{mul}^\ast, \bf{s}} &= bb' + \sum_{i, j, k} a_{i, j}[k] \cdot v_{i, j, k} +  \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \sum_{i, j, k} a_{i, j}[k] \cdot \span{\bf{u}_{i, j, k}, \bf{s}} \\
&= bb' + \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \sum_{i, j, k} a_{i, j}[k] \paren{v_{i, j, k} + \span{\bf{u}_{i, j, k}, \bf{s}}}.
\end{aligned}
$$

Since $v_{i, j, k} + \span{\bf{u}_{i, j, k}, \bf{s}} = 2^k \cdot s_i s_j + 2e_{i, j, k} \pmod q$, the above expression further reduces to

$$
\begin{aligned}
&= bb' + \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \sum_{i, j, k} a_{i, j}[k] \paren{2^k \cdot s_i s_j + 2e_{i, j, k}} \\
&= bb' + \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \sum_{i, j} a_{i, j}s_i s_j + 2\sum_{i, j, k} a_{i, j}[k] \cdot e_{i, j, k} \\
&= bb' + \span{b\bf{a}' + b'\bf{a}, \bf{s}} + \span{\bf{a} \otimes \bf{a}', \bf{s} \otimes \bf{s}'} + 2e\conj \\
&= rr' + 2e\conj \pmod q,
\end{aligned}
$$

and we have an encryption of $mm'$. In this case,

$$
e\conj = 2\sum_{i=1}^n\sum_{j=1}^n \sum_{k=0}^{\ceil{\log q}} a_{i, j}[k] \cdot e_{i, j, k}
$$

is small enough to use, since $a_{i, j}[k] \in \braces{0, 1}$. The size is about $\mc{O}(n^2 \log q)$, which is a lot smaller than $q$ for practical uses. We have reduced $n^2 q$ to $n^2 \log q$ with this method.

### Noise Reduction

Now we handle the noise growth. For correctness, we required that

$$
\abs{r} = \abs{m + 2e} < \frac{1}{2}q.
$$

But for multiplication, $\abs{r_\rm{mul}} = \abs{rr' + 2e\conj}$, so the noise grows very fast. If the initial noise size was $N$, then after $L$ levels of multiplication, the noise is now $N^{2^L}$.[^3] To reduce noise, we use **modulus switching**.

Given $\bf{c} = (b, \bf{a}) \in \Z_q^{n+1}$, we reduce the modulus to $q' < q$ which results in a smaller noise $e'$. This can be done by scaling $\bf{c}$ by $q'/q$ and rounding it.

> **Modulus Switching**: let $\bf{c} = (b, \bf{a}) \in \Z_q^{n+1}$ be given.
> 
> - Find $b'$ closest to $b \cdot (q' /q)$ such that $b' = b \pmod 2$.
> - Find $a_i'$ closest to $a_i \cdot (q'/q)$ such that $a_i' = a_i \pmod 2$.
> - Output $\bf{c}' = (b', \bf{a}') \in \Z_{q'}^{n+1}$.

In summary, $\bf{c}' \approx \bf{c} \cdot (q'/q)$, and $\bf{c}' = \bf{c} \pmod 2$ component-wise.

We check if the noise has been reduced, and decryption results in the same message $m$. Decryption of $\bf{c}'$ is done by $r' = b' + \span{\bf{a}', \bf{s}} \pmod{q'}$, so we must prove that $r' \approx r \cdot (q'/q)$ and $r' = r \pmod 2$. Then the noise is scaled down by $q'/q$ and the message is preserved.

Let $k \in \Z$ such that $b + \span{\bf{a}, \bf{s}} = r + kq$. By the choice of $b'$ and $a_i'$,

$$
b' = b \cdot (q'/q) + \epsilon_0, \quad a_i' = a_i \cdot (q'/q) + \epsilon_i
$$

for $\epsilon_i \in\braces{0, 1}$. Then

$$
\begin{aligned}
b' + \span{\bf{a}', \bf{s}} &= b' + \sum_{i=1}^n a_i's_i \\
&= b \cdot (q'/q) + \epsilon_0 + \sum_{i=1}^n \paren{a_i \cdot (q'/q) + \epsilon_i} s_i \\
&= (q'/q) \paren{b + \sum_{i=1}^n a_i s_i} + \epsilon_0 + \sum_{i=1}^n \epsilon_i s_i \\
&= (q'/q) \cdot (r + kq) + \epsilon_0 + \sum_{i=1}^n \epsilon_i s_i \\
&= r \cdot (q'/q) + \epsilon_0 + \sum_{i=1}^n \epsilon_i s_i +  kq'.
\end{aligned}
$$

We additionally assume that $\bf{s} \in \Z_2^n$, then the error term is bounded by $n+1$, and $n \ll q$.[^4] Set

$$
r' = r \cdot (q'/q) + \epsilon_0 + \sum_{i=1}^n \epsilon_i s_i,
$$

then we have $r' \approx r \cdot (q'/q)$.

Next, $b + \span{\bf{a}, \bf{s}} = b' + \span{\bf{a}', \bf{s}} \pmod 2$ component-wise. Then

$$
r + kq = b + \span{\bf{a}, \bf{s}} = b' + \span{\bf{a}', \bf{s}} = r' + kq' \pmod 2.
$$

Since $q, q'$ are odd, $r = r' \pmod 2$.

### Modulus Chain

Let the initial noise be $\abs{r} \approx N$. Set the maximal level $L$ for multiplication, and set $q_{L} = N^{L+1}$. Then after each multiplication, switch the modulus to $q_{k-1} = q_k/N$ using the above method.

Multiplication increases the noise to $N^2$, and then modulus switching decreases the noise back to $N$, allowing further computation.

So we have a modulus chain,

$$
N^{L+1} \ra N^L \ra \cdots \ra N.
$$

When we perform $L$ levels of computation and reach modulus $q_0 = N$, we cannot perform any multiplications. We must apply [bootstrapping](./2023-12-08-bootstrapping-ckks.md#bootstrapping).

Note that without modulus switching, we need $q_L > N^{2^L}$ for $L$ levels of computation, which is very large. Since we want $q$ to be small (for the hardness of the LWE problem), modulus switching is necessary. We now only require $q_L > N^{L+1}$.

### Multiplication in BGV (Summary)

- Set up a modulus chain $q_k = N^{k+1}$ for $k = 0, \dots, L$.
- Given two ciphertexts $\bf{c} = (b, \bf{a}) \in \Z_{q_k}^{n+1}$ and $\bf{c}' = (b', \bf{a}') \in \Z_{q_k}^{n+1}$ with modulus $q_k$ and noise $N$.

- (**Tensor Product**) $\bf{c}_\rm{mul} = \bf{c} \otimes \bf{c}' \pmod{q_k}$.
	- Now we have $n^2$ dimensions and noise $N^2$.
- (**Relinearization**)
	- Back to $n$ dimensions and noise $N^2$.
- (**Modulus Switching**)
	- Modulus is switched to $q_{k-1}$ and noise is back to $N$.

## BGV Generalizations and Optimizations

### From $\Z_2$ to $\Z_p$

The above description is for messages $m \in \braces{0, 1} = \Z_2$. This can be extend to any finite field $\Z_p$. Replace $2$ with $p$ in the scheme. Then encryption of $m \in \Z_p$ is done as

$$
b = -\span{\bf{a}, \bf{s}} + m + pe \pmod q,
$$

and we have $r = b + \span{\bf{a}, \bf{s}} = m + pe$, $m = r \pmod p$.

### Packing Technique

Based on the Ring LWE problem, plaintext space can be extended from $\Z_p$ to $\Z_p^n$ by using **polynomials**.

With this technique, the number of linearization keys is reduced from $n^2 \log q$ to $\mc{O}(1)$.

## Security and Performance of BGV

- Security depends on $n$ and $q$.
	- $(n, \log q) = (2^{10}, 30), (2^{13}, 240), (2^{16}, 960)$.
	- $q$ is much larger than $n$.
	- We want $n$ small and $q$ large enough to be correct.
- BGV is a **somewhat** homomorphic encryption.
	- The number of multiplications is limited.
- Multiplication is expensive, especially linearization.
	- Parallelization is effective for optimization, since multiplication is basically performing the same operations on different data.

[^1]: A homomorphism is a *confused name changer*. It can map different elements to the same name.
[^2]: The columns $\bf{a}_i$ are chosen random, so $A$ is invertible with high probability.
[^3]: Noise: $N \ra N^2 \ra N^4 \ra \cdots \ra N^{2^L}$.
[^4]: This is how $\bf{s}$ is chosen in practice.
