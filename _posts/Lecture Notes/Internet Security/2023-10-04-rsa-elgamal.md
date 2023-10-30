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
title: 06. RSA and ElGamal Encryption
date: 2023-10-04
github_title: 2023-10-04-rsa-elgamal
---

## Exponential Inverses

Suppose we are given integers $a$ and $N$. For any integer $x$ that is relatively prime to $N$, we choose $b$ so that

$$

\tag{$*$}
ab \equiv 1 \pmod{\phi(N)}.
$$

Then we have

$$
x^{ab} \equiv x^{1 + k\phi(N)} \equiv x \pmod N
$$

by Euler's generalization.

> **Definition.** The integer $b$ satisfying $(\ast)$ is called the **exponential inverse of $a$ modulo $N$**.

Using exponential inverses will be a key idea in the RSA cryptosystem.

## RSA Cryptosystem

This is an explanation of *textbook* RSA encryption scheme.

### Key Generation

- We pick two large primes $p, q$ and set $N = pq$.
- Select $(e, d)$ so that $ed \equiv 1 \pmod{\phi(N)}$.
- Set $(N, e)$ as the **public key** and make it public.
- Set $d$ as the **private key** and keep it secret.

### RSA Encryption and Decryption

Suppose we want to encrypt a message $m \in \mathbb{Z}_N$.

- **Encryption**
	- Using the public key $(N, e)$, compute the ciphertext $c = m^e \bmod N$.
- **Decryption**
	- Recover the original message by computing $c^d \bmod N$.

### Correctness of RSA?

Since $ed \equiv 1 \pmod{\phi(N)}$, we have

$$
c^d \equiv m^{ed} \equiv m \pmod N
$$

by the properties of exponential inverses.

Wait, but the properties requires that $\gcd(m, N) = 1$. So it seems like we can't use some values of $m$. Furthermore, it should be computationally infeasible to recover $d$ using $e$ and $N$.

### Regarding the Choice of $N$

If $N$ is prime, it is very easy to find $d$. Since the relation $ed \equiv 1 \pmod {(N-1)}$ holds, we directly see that $d$ can be computed efficiently using the extended Euclidean algorithm.

The next simplest case would be setting $N = pq$ for two large primes $p$ and $q$. We expose $N$ to the public but hide primes $p$ and $q$. Now suppose the attacker wants to compute $d$ using $(N, e)$. The attacker knows that $ed \equiv 1 \pmod {\phi(N)}$, and $\phi(N) = (p-1)(q-1)$. So to calculate $d$, the attacker must know $\phi(N)$, which requires the **factorization of $N$**.

If the factorization $N = pq$ is known, finding $d$ is easy. But factoring large prime numbers (especially a product of two primes of similar size) is known to be very difficult.[^1] No one has formally proven this, but we believe and assume that it is hard.[^2]

## Chinese Remainder Theorem in RSA

Assume that the message $m$ is not divisible by both $p$ and $q$. By Fermat's little theorem, we have $m^{p-1} \equiv 1 \pmod p$ and $m^{q-1} \equiv 1 \pmod q$.

Therefore, for decryption in RSA, the following holds. Note that $N = pq$.

$$
c^d \equiv m^{ed} \equiv m^{1 + k\phi(N)} \equiv m \cdot (m^{p-1})^{k(q-1)} \equiv m \cdot 1^{k(q-1)} \equiv m \pmod p.
$$

A similar result holds for modulus $q$. This does not exactly recover the message yet, since $m$ could have been chosen to be larger than $p$. The above equation is true, but during actual computation, one may get a result that is less than $p$. *This may not be equal to the original message*.[^3]

Since $N = pq$, we use the Chinese remainder theorem. Instead of computing $c^d \pmod N$, we can compute

$$
c^d \equiv m \pmod p, \qquad c^d \equiv m \pmod q
$$

independently and solve the system of equations to recover the message.

## Can I Encrypt $p$ with RSA?

Now we return to the problem where $\gcd(m, N) \neq 1$. The probability of $\gcd(m, N) \neq 1$ is actually $\frac{1}{p} + \frac{1}{q} - \frac{1}{pq}$, so if we take large primes $p, q \approx 2^{1000}$ as in RSA2048, the probability of this occurring is roughly $2^{-999}$, which is negligible. But for completeness, we also prove for this case.

$e, d$ are still chosen to satisfy $ed \equiv 1 \pmod {\phi(N)}$. Suppose we want to decrypt $c \equiv m^e \pmod N$.

We will also use the Chinese remainder theorem here.

Since $\gcd(m, N) \neq 1$ and $N = pq$, we have $p \mid m$. So if we compute in $\mathbb{Z}_p$, we will get $0$,

$$
c^d \equiv m^{ed} \equiv 0^{ed} \equiv 0 \pmod p.
$$

We also do the computation in $\mathbb{Z}_q$ and get

$$
c^d \equiv m^{ed} \equiv m^{1 + k\phi(N)} \equiv m\cdot (m^{q-1})^{k(p-1)} \equiv m \cdot 1^{k(p-1)} \equiv m \pmod q.
$$

Here, we used the fact that $m^{q-1} \equiv 1 \pmod q$. This holds because if $p \mid m$, $m$ is a multiple of $p$ that is less than $N$, so $m = pm'$ for some $m'$ such that $1 \leq m' < q$. Then $\gcd(m, q) = \gcd(pm', q) = 1$ since $q$ does not divide $p$ and $m'$ is less than $q$.

Therefore, from $c^d \equiv 0 \pmod p$ and $c^d \equiv (m \bmod q) \pmod q$, we can recover a unique solution $c^d \equiv m \pmod N$.

Now we must argue that the recovered solution is actually equal to the original $m$. But what we did above was showing that $m^{ed}$ and $m$ in $\mathbb{Z}_N$ are mapped to the same element $(0, m \bmod q)$ in $\mathbb{Z}_p \times \mathbb{Z}_q$. Since the Chinese remainder theorem tells us that this mapping is an isomorphism, $m^{ed}$ and $m$ must have been the same elements of $\mathbb{Z}_N$ in the first place.

Notice that we did not require $m$ to be relatively prime to $N$. Thus the RSA encryption scheme is correct for any $m \in \mathbb{Z}_N$.

## Correctness of RSA with Fermat's Little Theorem

Actually, the above argument can be proven only with Fermat's little theorem. In the above proof, the Chinese remainder theorem was used to transform the operation, but for $N = pq$, the situation is simple enough that this theorem is not necessarily required.

Let $M = m^{ed} - m$. We have shown above only using Fermat's little theorem that $p \mid M$ and $q \mid M$, for any choice of $m \in \mathbb{Z}_N$. Then since $N = pq = \mathrm{lcm}(p, q)$, we have $N \mid M$, so $m^{ed} \equiv m \pmod N$. Hence the RSA scheme is correct.

So we don't actually need Euler's generalization for proving the correctness of RSA...?! In fact, the proof given in the original paper of RSA used Fermat's little theorem.

## Discrete Logarithms

This is an inverse problem of exponentiation. The inverse of exponentials is logarithms, so we consider the **discrete logarithm of a number modulo $p$**.

Given $y \equiv g^x \pmod p$ for some prime $p$, we want to find $x = \log_g y$. We set $g$ to be a generator of the group $\mathbb{Z}_p$ or $\mathbb{Z}_p^*$, since if $g$ is the generator, a solution always exists.

Read more in [discrete logarithm problem (Modern Cryptography)](../../modern-cryptography/2023-10-03-key-exchange#discrete-logarithm-problem-dl).

## ElGamal Encryption

This is an encryption scheme built upon the hardness of the DLP.

> 1. Let $p$ be a large prime.
> 2. Select a generator $g \in \mathbb{Z}_p^*$.
> 3. Choose a private key $x \in \mathbb{Z}_p^*$.
> 4. Compute the public key $y = g^x \pmod p$.
> 	- $p, g, y$ will be publicly known.
> 	- $x$ is kept secret.

### ElGamal Encryption and Decryption

Suppose we encrypt a message $m \in \mathbb{Z}_p^*$.

> 1. The sender chooses a random $k \in \mathbb{Z}_p^*$, called *ephemeral key*.
> 2. Compute $c_1 = g^k \pmod p$ and $c_2 = my^k \pmod p$.
> 3. $c_1, c_2$ are sent to the receiver.
> 4. The receiver calculates $c_1^x \equiv g^{xk} \equiv y^k \pmod p$, and find the inverse $y^{-k} \in \mathbb{Z}_p^*$.
> 5. Then $c_2y^{-k} \equiv m \pmod p$, recovering the message.

The attacker will see $g^k$. By the hardness of DLP, the attacker is unable to recover $k$ even if he knows $g$.

#### Ephemeral Key Should Be Distinct

If the same $k$ is used twice, the encryption is not secure. Suppose we encrypt two different messages $m_1, m_2 \in \mathbb{Z} _ p^{ * }$. The attacker will see $(g^k, m_1y^k)$ and $(g^k, m_2 y^k)$. Then since we are in a multiplicative group $\mathbb{Z} _ p^{ * }$, inverses exist. So

$$
m_1y^k \cdot (m_2 y^k)^{-1} \equiv m_1m_2^{-1} \equiv 1 \pmod p
$$

which implies that $m_1 \equiv m_2 \pmod p$, leaking some information.

[^1]: If one of the primes is small, factoring is easy. Therefore we require that $p, q$ both be large primes.
[^2]: There is a quantum polynomial time (BQP) algorithm for integer factorization. See [Shor's algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm).
[^3]: This part of the explanation is not necessary if we use abstract algebra!
