---
share: true
toc: true
math: true
categories:
  - Lecture Notes
  - Internet Security
tags:
  - lecture-note
  - security
  - cryptography
  - number-theory
title: 05. Modular Arithmetic (2)
date: 2023-10-04
github_title: 2023-10-04-modular-arithmetic-2
---

## Exponentiation by Squaring

Suppose we want to calculate $a^n$ where $n$ is very large, like $n \approx 2^{1000}$. A naive multiplication would take $\mathcal{O}(n)$ multiplications. We will ignore integer overflow for simplicity.

```c
int naive_exponentiation(int a, int n) {
    int result = 1;
    for (int i = 0; i < n; ++i) {
        result *= a;
    }
    return result;
}
```

Using the above implementation, computing $3^{2^{63} - 1}$ takes almost forever...

Instead, we use **exponentiation by squaring** method. Notice the following,

$$
a^n = \begin{cases}
(a^2)^{\frac{n}{2}} & (n \text{ is even})\\
a \cdot (a^2)^{\frac{n-1}{2}} & (n \text{ is odd})
\end{cases}.
$$

Therefore, the exponent is reduced by half for every multiplication. Here is the implementation. The base cases are to be handled separately.

```c
int exponentiation_by_squaring(int a, int n) {
    if (n == 0) {
        return 1;
    } else if (n == 1) {
        return a;
    }

    int result = 1;
    if (n % 2 == 0) {
        return exponentiation_by_squaring(a * a, n / 2);
    } else {
        return a * exponentiation_by_squaring(a * a, (n - 1) / 2);
    }
}
```

The above code executes about $\mathcal{O}(\log n)$ multiplications. Now we can actually get an answer for $3^{2^{63} - 1}$.

Alternatively, here is an iterative version of the above for those who want to save some memory.

```c
int exponentiation_by_squaring_iterative(int a, int n) {
    int result = 1;
    int base = a, exponent = n;
    while (exponent > 0) {
        if (n % 2 == 1) {
            result *= base;
        }

        base *= base;
        exponent /= 2;
    }
    return result;
}
```

For even better (maybe faster) results, we need the help of elementary number theory.

## Fermat's Little Theorem

> **Theorem.** Let $p$ be prime. For $a \in \mathbb{Z}$ such that $\gcd(a, p) = 1$,
> 
> $$
> a^{p-1} \equiv 1 \pmod p.
> $$

*Proof*. (Using group theory) The statement can be rewritten as follows. For $a \neq 0$ in $\mathbb{Z}_p$, $a^{p-1} = 1$ in $\mathbb{Z}_p$. Since $\mathbb{Z}_p^*$ is a (multiplicative) group of order $p-1$, the order of $a$ should divide $p-1$. Therefore, $a^{p-1} = 1$ in $\mathbb{Z}_p$.

Here is an elementary proof not using group theory.

*Proof*. (Elementary) Let $S = \left\lbrace 0, 1, \dots, p-1 \right\rbrace$. Consider a map $f : S \rightarrow S$ defined as $x \mapsto ax \bmod p$ ($a \neq 0$).

We will show that $f$ is injective. Suppose that $ax \equiv ay \pmod p$ for distinct $x, y \in S$. Since $\gcd(a, p) = 1$, $a$ has a multiplicative inverse, thus $x \equiv y \pmod p$. Then $x, y$ should be same elements of $S$.

By injectivity, $f(i)$ are distinct for all $i \in S$, so $f$ is a permutation on $S$. Therefore, the product of all elements of $S$ must be equal to the product of all $f(i)$ for $i \in S$.

$$
(p-1)! \equiv f(1)f(2)\cdots f(p-1) \equiv a^{p-1} \cdot (p-1)!\pmod p.
$$

Since $\gcd(i, p) = 1$ for all $i \in S$, we can multiply the multiplicative inverse for all $i \in S$ and we get $a^{p-1} \equiv 1 \pmod p$.

## Euler's Totient Function

For composite modulus, we have Euler's generalization. Before proving the theorem, we first need to define Euler's totient function.

> **Definition.** Let $n \in \mathbb{N}$. Define $\phi(n)$ as the number of positive integers $k \leq n$ such that $\gcd(n, k) = 1$.

For direct calculation, we use the following formula.

> **Lemma.** For $n \in \mathbb{N}$, the following holds.
> 
> $$
> \phi(n) = n \cdot \prod_{p \mid n} \left( 1 - \frac{1}{p} \right)
> $$
> 
> where $p$ is a prime number dividing $n$.

So to calculate $\phi(n)$, we need to **factorize** $n$. From the formula above, we have some corollaries.

> **Corollary.** For prime numbers $p, q$ and $k \in \mathbb{N}$, the following hold.
> 1. $\phi(p) = p - 1$.
> 2. $\phi(pq) = (p-1)(q-1)$.
> 3. $\phi(p^k) = p^{k-1}(p-1)$.

### Reduced Set of Residues

Let $n \in \mathbb{N}$. The **complete set of residues** was denoted $\mathbb{Z}_n$ and

$$
\mathbb{Z}_n = \left\lbrace 0, 1, \dots, n-1 \right\rbrace.
$$

We also often use the **reduced set of residues**.

> **Definition.** The **reduced set of residues** is the set of residues that are relatively prime to $n$. We denote this set as $\mathbb{Z}_n^*$.
> 
> $$
> \mathbb{Z}_n^* = \left\lbrace a \in \mathbb{Z}_n \setminus \left\lbrace 0 \right\rbrace : \gcd(a, n) = 1 \right\rbrace.
> $$

Then by definition, we have the following result.

> **Lemma.** $\left\lvert \mathbb{Z}_n^* \right\lvert = \phi(n)$.

We can also show that $\mathbb{Z}_n^*$ is a multiplicative group.

> **Lemma.** $\mathbb{Z}_n^*$ is a multiplicative group.

*Proof*. Let $a, b \in \mathbb{Z}_n^{ * }$. We must check if $ab \in \mathbb{Z}_n^{ * }$. Since $\gcd(a, n) = \gcd(b, n) = 1$, $\gcd(ab, n) = 1$. This is because if $d = \gcd(ab, n) > 1$, then a prime factor $p$ of $d$ must divide $a$ or $b$ and also $n$. Then $\gcd(a, n) \geq p$ or $\gcd(b, n) \geq p$, which is a contradiction. Thus $ab \in \mathbb{Z}_n^{ * }$.

Associativity holds trivially, as a subset of $\mathbb{Z}_n$. We also have an identity element $1$, and inverse of $a \in \mathbb{Z}_n^*$ exists since $\gcd(a, n) = 1$.

Now we can prove Euler's generalization.

## Euler's Generalization

> **Theorem.** Let $a \in \mathbb{Z}$ such that $\gcd(a, n) = 1$. Then
> 
> $$
> a^{\phi(n)} \equiv 1 \pmod n.
> $$

*Proof*. Since $\gcd(a, n) = 1$, $a \in \mathbb{Z}_n^{ * }$. Then $a^\left\lvert \mathbb{Z}_n^{ * } \right\lvert = 1$ in $\mathbb{Z}_n$. By the above lemma, we have the desired result.

*Proof*. (Elementary) Set $f : \mathbb{Z}_n^* \rightarrow \mathbb{Z}_n^*$ as $x \mapsto ax \bmod n$, then the rest of the reasoning follows similarly as in the proof of Fermat's little theorem.

Using the above result, we remark an important result that will be used in RSA.

> **Lemma.** Let $n \in \mathbb{N}$. For $a, b \in \mathbb{Z}$ and $x \in \mathbb{Z}_n^*$, if $a \equiv b \pmod{\phi(n)}$, then $x^a \equiv x^b \pmod n$.

*Proof*. $a = b + k\phi(n)$ for some $k \in \mathbb{Z}$. Then

$$
x^a \equiv x^{b + k\phi(n)} = (x^{\phi(n)})^k \cdot x^b \equiv x^b \pmod n
$$

by Euler's generalization.

## Groups Based on Modular Arithmetic

> **Definition.** A **group** is a set $G$ with a binary operation $* : G \times G \rightarrow G$, satisfying the following properties.
> 
> - $(\mathsf{G1})$ The binary operation $*$ is **closed**.
> - $(\mathsf{G2})$ The binary operation $*$ is **associative**, so $(a * b) * c = a * (b * c)$ for all $a, b, c \in G$.
> - $(\mathsf{G3})$ $G$ has an **identity** element $e$ such that $e * a = a * e = a$ for all $a \in G$.
> - $(\mathsf{G4})$ There is an **inverse** for every element of $G$. For each $a \in G$, there exists $x \in G$ such that $a * x = x * a = e$. We write $x = a^{-1}$ in this case.

$\mathbb{Z}_n$ is an additive group, and $\mathbb{Z}_n^*$ is a multiplicative group.

## Chinese Remainder Theorem (CRT)

> **Theorem.** Let $n_1, \dots, n_k$ integers greater than $1$, and let $N = n_1n_2\cdots n_k$. If $n_i$ are pairwise relatively prime, then the system of equations $x \equiv a_i \pmod {n_i}$ has a unique solution modulo $N$.
> 
> *(Abstract Algebra)* The map
> 
> $$
> x \bmod N \mapsto (x \bmod n_1, \dots, x \bmod n_k)
> $$
> 
>  defines a ring isomorphism
> 
> $$
>  \mathbb{Z}_N \simeq \mathbb{Z}_{n_1} \times \mathbb{Z}_{n_2} \times \cdots \times \mathbb{Z}_{n_k}.
> $$

*Proof*. (**Existence**) Let $N_i = N/n_i$. Then $\gcd(N_i, n_i) = 1$. By the extended Euclidean algorithm, there exist integers $M_i, m_i$ such that $M_iN_i + m_in_i= 1$. Now set

$$
x = \sum_{i=1}^k a_i M_i N_i.
$$

Then $x \equiv a_iM_iN_i \equiv a_i(1 - m_in_i) \equiv a_i \pmod {n_i}$ for all $i = 1, \dots, k$.

(**Uniqueness**) Suppose that we have two distinct solutions $x, y$ modulo $N$. $x, y$ are solutions to $x \equiv a_i \pmod {n_i}$, so $n_i \mid (x - y)$ for all $i$. Therefore we have

$$
\mathrm{lcm}(n_1, \dots, n_k) \mid (x - y).
$$

But $n_i$ are pairwise relatively prime, so $\mathrm{lcm}(n_1, \dots, n_k) = N$ and $N \mid (x-y)$. Hence $x \equiv y \pmod N$.

*Proof*. (**Abstract Algebra**) The above uniqueness proof shows that the map

$$
x \bmod N \mapsto (x \bmod n_1, \dots, x \bmod n_k) 
$$

is injective. By pigeonhole principle, this map must also be surjective. This map is also a ring homomorphism, by the properties of modular arithmetic. We have a ring isomorphism.

### Notes on the Proof of the Chinese Remainder Theorem

The elementary proof given above gives a *direct construction* of the solution. It is clear and easy to understand, and tells us how to find the actual solution.

But when the above proof is used in actual computation, it involves computations of very large numbers. The following is an implementation.

```cpp
// remainder holds the a_i values
// modulus holds the n_i values
int chinese_remainder_theorem(vector<int>& remainder, vector<int>& modulus) {
    int product = 1;
    for (int m : modulus) {
        product *= m;
    }

    int result = 0;
    for (int i = 0; i < (int) modulus.size(); ++i) {
        int N_i = product / modulus[i];
        result += remainder[i] * modular_inverse(N_i, modulus[i]) * N_i;
        result %= product;
    }

    return result;
}
```

The `modular_inverse` function uses the extended Euclidean algorithm to find $M_i$ in the proof. For large moduli and many equations, $N_i = N / n_i$ results in a very large number, which is hard to handle (if your language has integer overflow) and takes longer to compute.

A better way is to construct the solution **inductively**. Find a solution for the first two equations,

$$
\begin{array}{c}
x \equiv a_1 \pmod{n_1} \\
x \equiv a_2 \pmod{n_2}
\end{array} \implies x \equiv a_{1, 2} \pmod{n_1n_2}
$$

and using the result, add the next equation $x \equiv a_3 \pmod{n_3}$ and find a solution.[^1]

Lastly, the ring isomorphism actually tells us a lot and is quite effective for computation. Since the two rings are *isomorphic*, operations in $\mathbb{Z} _ N$ can be done independently in each $\mathbb{Z} _ {n_i}$ and then merged back to $\mathbb{Z} _ N$. $N$ was a large number, so computations can be much faster in $\mathbb{Z} _ {n _ i}$. Specifically, we will see how this fact is used for computations in RSA.

[^1]: I have an implementation in my repository. [Link](https://github.com/calofmijuck/BOJ/blob/4b29e0c7f487aac3186661176d2795f85f0ab21b/Codes/23000/23062.cpp#L38).
