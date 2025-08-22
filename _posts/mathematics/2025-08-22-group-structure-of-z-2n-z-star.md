---
share: true
toc: true
math: true
categories:
  - Mathematics
path: _posts/mathematics
tags:
  - math
  - cryptography
title: Group Structure of $(\mathbb{Z}/2^n \mathbb{Z})^*$
date: 2025-08-22
github_title: 2025-08-22-group-structure-of-z-2n-z-star
---

To compute the rotation automorphism homomorphically, we use the fact that $(\Z/2^n\Z)^* \simeq \span{-1, 5}$. I couldn't find a clear proof of this result online, so I just accepted the fact although it wasn't very satisfying.

After more than a year, I got a chance to revisit the rotation automorphism and I figured that I should clear things up once and for all. So I decided to compile a proof, drawn from many sources.

## Theorem

> **Theorem 1.** $(\Z/2^n \Z)^*$ is the direct product of a cyclic group of order $2$ and cyclic group of order $2^{n-2}$, for all $n \geq 2$.

The above theorem is from Corollary 20 (2) of Section 9.5 in Abstract Algebra, 3rd Edition, Dummit and Foote.

> **Theorem 2.** $(\Z/2^n\Z)^* \simeq \span{-1, 5}$ for $n \geq 3$.

## Observations

### Order of $5$ Modulo $2^n$

> **Proposition.** $5^{2^{n-3}} \equiv 1 + 2^{n-1} \pmod {2^n}$ for $n \geq 3$.

*Proof*. This is an easy proof with induction. Omitted.

> **Lemma.** $5$ has order $2^{n-2}$ in $(\Z/2^n \Z)^*$, for $n \geq 2$.

*Proof*. We will use strong induction. For $n = 2, 3$, the lemma can be checked by direct computation. Now assume that the order of $5$ is $2^{k-2}$ in $(\Z/2^k\Z)^*$, for all $3 \leq k \leq n$.

Let $r$ be the order of $5$ modulo $2^{n+1}$. Then $2^{n-2} \mid r$. This is from the fact that

$$
5^r \equiv 1 \pmod {2^{n+1}} \implies 5^r \equiv 1 \pmod {2^n}.
$$

Therefore $r$ must be a multiple of $2^{n-2}$. (order of $5$ modulo $2^n$) But from the above proposition, $5^{2^{n-2}} \equiv 1 + 2^n \pmod {2^{n+1}}$, so $r \neq 2^{n-2}$. The next candidate of $r$ is $2^{n-1}$ since it should be a multiple of $2^{n-2}$. Observe that

$$
5^{2^{n-1}} = \paren{5^{2^{n-2}}}^2 = (1 + 2^{n})^2 \equiv 1 \pmod {2^{n+1}},
$$

completing the proof.

### Group is Not Cyclic

> **Proposition.** Let $G = \span{x}$ be a cyclic group of finite order $n < \infty$. For each divisor $a$ of $n$, there exists a unique subgroup of $G$ with order $a$.

*Proof*. Since $a \mid n$, set $d = n /a$. Then $\span{x^d}$ is a subgroup of order $a$, showing existence.

For uniqueness, suppose $H \neq \span{x^d}$ is another subgroup of $G$ with order $a$. Since subgroups of cyclic groups are also cyclic, $H = \span{x^k}$ where $k$ is the smallest positive integer with $x^k \in H$. Then from

$$
\frac{n}{d} = a = \abs{H} = \frac{n}{\gcd(n, k)},
$$

$d = \gcd(n, k)$. So $k$ is a multiple of $d$, resulting in $x^k \in \span{x^d}$. Therefore, $H \leq \span{x^d}$, but the two groups have the same order, so $H = \span{x^d}$.

> **Lemma.** $(\Z/2^n \Z)^*$ is not cyclic for any $n \geq 3$.

*Proof*. $(\Z/2^n\Z)^*$ has two distinct subgroups of order $2$. For $n \geq 3$,

$$
(2^n - 1)^2 \equiv (-1)^2 \equiv 1 \pmod {2^n}
$$

and

$$
(2^{n-1}-1)^2 = 2^{2n-2} - 2^n + 1 \equiv 1 \pmod {2^n}.
$$

Both $2^n-1$ and $2^{n-1} - 1$ have order $2$ modulo $2^n$ and they are distinct since $n \geq 3$. By the above proposition, $(\Z/2^n\Z)^*$ cannot be cyclic.

## Proof of Theorem 1

*Proof*. $(\Z/2^n\Z)^*$ is a finitely generated abelian group, so the fundamental theorem of finitely generated abelian groups applies here. We know that group has order $2^{n-1}$, and from the above results,

- $(\Z/2^n \Z)^*$ has an element of order $2^{n-2}$,
- $(\Z/2^n \Z)^*$ is not cyclic for $n \geq 3$.

Thus, for $n \geq 3$, the only possible case is $(\Z/2^n\Z)^* \simeq \Z _ 2 \times \Z_{2^{n-2}}$. As for $n = 2$, $(\Z/4\Z)^* \simeq \Z_2 \times \Z_1$ is pretty obvious.

*Note*. I'm still looking for an elementary proof that doesn't use the fundamental theorem. This sort of feels like nuking a mosquito.

## More Observations

> **Lemma.** Suppose that $H$ and $K$ are normal subgroups of $G$ and $H \cap K = \braces{1}$. Then $HK \simeq H \times K$.

*Proof*. Construct an isomorphism $\varphi : H \times K \ra HK$ such that $(h, k) \mapsto hk$.

Since $H, K \unlhd G$, observe that $hkh\inv k \inv \in K \cap H = \braces{1}$ and $hk = kh$. Therefore,

$$
\varphi(h, k) \cdot \varphi(h',k') = hkh'k' = hh' kk' = \varphi\paren{(h, k)\cdot (h', k')}
$$

and $\varphi$ is a homomorphism.

Next, if $\varphi(h, k) = hk = 1$, we have $h = k\inv \in H\cap K = \braces{1}$. Then $h = k = 1$, showing that $\ker \varphi$ is trivial and $\varphi$ is injective.

Surjectivity of $\varphi$ is trivial. $\varphi$ is an isomorphism and $HK \simeq H \times K$.

> **Proposition.** As subgroups of $(\Z/2^n\Z)^*$, $\span{-1} \cap \span{5} = \braces{1}$ for $n \geq 3$.

*Proof*. It suffices to show that $-1 \notin \span{{5}}$. Suppose that $-1 \in \span{5}$. Since $\span{5}$ is cyclic, it has a unique element of order $2$. Since $5$ has order $2^{n-2}$, it must be the case that $-1 \equiv 5^{2^{n-3}} \pmod {2^n}$.

Then we have

$$
-1 \equiv 5^{2^{n-3}} \equiv 1 + 2^{n-1} \pmod {2^n},
$$

which gives $2^{n-1} + 2 \equiv 0 \pmod {2^n}$. But for $n \geq 3$, this is impossible since $0 < 2^{n-1} + 2 < 2^n$. Contradiction.

*Note*. If $-1 \in \span{5}$, then maybe $5$ would have generated the whole group. But the group isn't cyclic, so we have a contradiction?

## Proof of Theorem 2

*Proof*. Since we are dealing with commutative groups, all subgroups are normal. We have $\span{-1}, \span{5} \unlhd (\Z/2^n\Z)^*$ and $\span{-1} \cap \span{5} = \braces{1}$. Therefore,

$$
(\Z/2^n\Z)^* \simeq \Z_2 \times \Z_{2^{n-2}} = \span{-1} \times \span{5} \simeq \span{-1}\span{5}.
$$

This means that we can uniquely write all elements of $(\Z/2^n\Z)^*$ as $(-1)^a 5^b$ for ${} 0 \leq a < 2 {}$, $0 \leq b < 2^{n-2}$. From commutativity, this exactly equals the subgroup generated by $-1$ and $5$, which is $\span{-1, 5}$. This concludes the proof.

## Notes

The theorem wasn't so trivial after all, but I'm still happy to have resolved a long overdue task.

## References

- My notes taken from abstract algebra class
- <https://math.stackexchange.com/q/459815>
- <https://math.stackexchange.com/q/3881641>
- <https://math.stackexchange.com/a/4910312/329909>
