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
title: 04. Modular Arithmetic (1)
date: 2023-09-25
github_title: 2023-09-25-modular-arithmetic-1
---

**Number theory** is a branch of mathematics devoted primarily to the study of the integers. **Modular arithmetic** is heavily used in cryptography.

## Divisibility

> **Definition.** Let $a, b, c \in \mathbb{Z}$ such that $a = bc$. Then,
> 
> 1. $b$ and $c$ are said to **divide** $a$, and are called **factors** of $a$.
> 2. $a$ is said to be a **multiple** of $b$ and $c$.

> **Notation.** For $a, b \in \mathbb{Z}$, we write $a \mid b$ if $a$ divides $b$. If not, we write $a \nmid b$.

These are simple lemmas for checking divisibility.

> **Lemma.** Let $a, b, c \in \mathbb{Z}$.
> 
>  1. If $a \mid b$ and $a \mid c$, then $a \mid (b + c)$.
>  2. If $a \mid b$, then $a \mid bc$.
>  3. If $a \mid b$ and $b \mid c$, then $a \mid c$.

## Prime Numbers

> **Definition.** Integer $n \geq 2$ is **prime** if it is only divisible by $1$ and itself. If it is not prime, then it is **composite**.

Note that $1$ is neither prime nor composite.

### Primality Tests

It is hard to verify if some given number is prime. Many encryption schemes heavily rely on this fact.

The following is a simple algorithm to check if a given integer is prime.

```c
bool naive_prime_test(int n) {
    if (n < 2) {
        return false;
    }

    for (int i = 2; i < sqrt(n); ++i) {
        if (n % i == 0) {
            return false;
        }
    }
    return true;
}
```

However, this algorithm has complexity $\mathcal{O}(\sqrt{n})$, which is slow. We have better algorithms like Fermat's test, Miller-Rabin test, Pollard's rho algorithm... (Not covered in this lecture)

## Division Algorithm

> **Theorem.** (Euclidean Division) For $a, b \in \mathbb{Z}$ with $b \neq 0$, there exist unique integers $q, r$ with $0 \leq r < \left\lvert b \right\rvert$ such that $a = bq + r$.

*Proof.* [By induction](https://en.wikipedia.org/wiki/Euclidean_division#Proof).

Other proofs use the well-ordering principle.

## Modulo Operation

There are two ways to think about 'mod': as a function, and as a congruence.

### Modulo as a Function

As a function, $a \bmod b$ return the remainder of $a$ divided by $b$. This operation is commonly denoted `%` in many programming languages.[^1]

### Modulo as a Congruence

As a congruence, it means that $a, b$ are in the same *equivalence class*.[^2]

> **Definition.** For $a, b, n \in \mathbb{Z}$ and $n \neq 0$, $a \equiv b \pmod n$ if and only if $n \mid (a - b)$.

Properties of modulo operation.

> **Lemma.** Suppose that $a \equiv b \pmod n$ and $c \equiv d \pmod n$. Then, the following hold.
> 
> 1. $a + c \equiv (b + d) \pmod n$.
> 2. $ac \equiv bd \pmod n$.
> 3. $a^k \equiv b^k \pmod n$.
> 4. $a \equiv (a \bmod n) \pmod n$.

*Proof.* Trivial. :)

The last one is very useful in computing. For example, if $a, b$ are very large integers, using the identity

$$
(a + b)^k \equiv ((a + b) \bmod n)^k \pmod n
$$

allows us to reduce the size of the numbers before exponentiation.

## Modular Arithmetic

For modulus $n$, **modular arithmetic** is operation on $\mathbb{Z}_n$.

### Residue Classes

For each positive integer $n$, we can partition $\mathbb{Z}$ into $n$ cells according to whether the remainder is $0, 1, 2, \dots, n - 1$ when the integer is divided by $n$. These cells are the **residue classes modulo $n$ in $\mathbb{Z}$**.

We write each residue class as follows.

$$
\overline{k} = [k] = \left\lbrace m \in \mathbb{Z} : m \bmod n = k\right\rbrace
$$

Consider the relation

$$
R = \left\lbrace (a, b) : a \equiv b \pmod m \right\rbrace \subset \mathbb{Z} \times \mathbb{Z}
$$

then $R$ has the following properties.

- **Reflexive**: $\forall a \in \mathbb{Z}$, $(a, a) \in R$.
- **Symmetric**: $\forall a, b \in \mathbb{Z}$, if $(a, b) \in R$, then $(b, a) \in R$.
- **Transitive**: $\forall a, b, c \in \mathbb{Z}$, if $(a, b), (b, c) \in R$ then $(a, c) \in R$.

Thus, $R$ is an **equivalence relation** and each residue class $[k]$ is an **equivalence class**.

We write the set of residue classes modulo $n$ as

$$
\mathbb{Z}_n = \left\lbrace \overline{0}, \overline{1}, \overline{2}, \dots, \overline{n-1} \right\rbrace.
$$

Note that $\mathbb{Z}_n$ is closed under addition and multiplication.

### Identity

> **Definition.** For a binary operation $\ast$ defined on a set $S$, $e$ is the **identity** if
> 
> $$
> \forall a \in S,\, a * e = e * a = a.
> $$

In $\mathbb{Z}_n$, the additive identity is $0$, the multiplicative identity is $1$.

### Inverse

> **Definition.** For a binary operation $\ast$ defined on a set $S$, let $e$ be the identity. $x$ is the **inverse of $a$** if
> 
> $$
> x * a = a * x = e.
> $$
> 
> We write $x = a^{-1}$.

In the language of modular arithmetic, $x$ is the inverse of $a$ if

$$
ax \equiv 1 \pmod n.
$$

The inverse exists if and only if $\gcd(a, n) = 1$.

> **Lemma**. For $n \geq 2$ and $a \in \mathbb{Z}$, its inverse $a^{-1} \in \mathbb{Z}_n$ exists if and only if $\gcd(a, n) = 1$.

*Proof*. We use the Extended Euclidean Algorithm. There exists $u, v \in \mathbb{Z}$ such that

$$
au + nv = \gcd(a, n).
$$

($\impliedby$) If $\gcd(a, n) = 1$, then $au + nv = 1$, so $au = 1 - nv \equiv 1 \pmod n$. Thus $a^{-1} = u$.

($\implies$) Suppose that $x = a^{-1}$ exists. Then $ax \equiv 1 \pmod n$, so $ax = 1 + kn$ for some $n \in \mathbb{Z}$. Then $ax - nk = 1$. $\gcd(a, n)$ must divide the LHS, so $\gcd(a, n) = 1$.

## Euclidean Algorithm

### Greatest Common Divisor

> **Definition.** Let $a, b \in \mathbb{Z} \setminus \left\lbrace 0 \right\rbrace$ . The **greatest common divisor** of $a$ and $b$ is the largest integer $d$ such that $d \mid a$ and $d \mid b$. We write $d = \gcd(a, b)$.

> **Definition.** If $\gcd(a, b) = 1$, we say that $a$ and $b$ are **relatively prime**.

### Euclidean Algorithm

Euclidean Algorithm is an efficient way to find $\gcd(a, b)$. It relies on the following lemma.

> **Lemma.** For $a, b \in \mathbb{Z}$ and $b \neq 0$, $\gcd(a, b) = \gcd(b, a \bmod b)$.

*Proof*. By the division algorithm, there exists $q, r \in \mathbb{Z}$ such that $a = bq + r$. Here, $r = a \bmod b$.

Let $d = \gcd(a, b)$. Then $d \mid a$ and $d \mid b$, so $d \mid (a - bq)$. Thus $d \leq \gcd(b, r)$. Conversely, let $d' = \gcd(b, r)$. Then $d' \mid b$ and $d' \mid (a - bq)$, so $d' \mid a$. Thus $d' \leq \gcd(a, b)$. Thus $d = d'$.

The following code computes the greatest common divisor.

```c
int gcd(int a, int b) {
	if (b == 0) {
        return a;
    } else {
        return gcd(b, a % b);
    }
}
```

### Extended Euclidean Algorithm

We can extend the Euclidean algorithm to compute $u, v \in \mathbb{Z}$ such that

$$
ua + vb = \gcd(a, b).
$$

Basically, we use the Euclidean algorithm and solve for the remainder (which is the $\gcd$).

#### Calculating Modular Multiplicative Inverse

We can use the extended Euclidean algorithm to find modular inverses. Suppose we want to calculate $a^{-1}$ in $\mathbb{Z}_n$. We assume that the inverse exist, so $\gcd(a, n) = 1$.

Therefore, we use the extended Euclidean algorithm and find $x, y \in \mathbb{Z}$ such that

$$
ax + ny = 1.
$$

Then $ax \equiv 1 - ny \equiv 1 \pmod n$, thus $x$ is the inverse of $a$ in $\mathbb{Z}_n$.

[^1]: Note that in C standards, `(a / b) * b + (a % b) == a`.
[^2]: $a$ and $b$ are in the same coset of $\mathbb{Z}/n\mathbb{Z}$.
