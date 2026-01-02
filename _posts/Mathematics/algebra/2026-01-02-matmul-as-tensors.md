---
share: true
toc: true
math: true
categories:
  - Mathematics
  - Algebra
path:  _ posts/mathematics
tags:
  - math
  - algebra
title: Matrix Multiplication as $(1, 1)$-Tensors
date: 2026-01-02
github _ title: 2026-01-02-matmul-as-tensors
image:
  path: /assets/img/posts/mathematics/algebra/matmul-tensor.png
---

## Introduction

Matrices have various applications throughout many fields of science. However, when we first learn matrices and their operations, the most non-intuitive concept is the **multiplication** of matrices.

Compared to addition and *scalar* multiplication, which are very intuitive, matrix multiplication has a rather strange definition.

> **Definition.** (Matrix Multiplication) For $A = (a _ {ij}) _ {n\times n}$ and $B = (b _ {ij}) _ {n\times n}$, their product is defined as $C = AB = (c _ {ij}) _ {n\times n}$ where
>
> $$
> c _ {ij} = \sum _ {k=1}^n a _ {ik}b _ {kj}.
> $$

This is often interpreted as "row times column", with the following diagram:

$$
\begin{aligned}
& \begin{bmatrix} & b _ {1j} & & & \\
& b _ {2j} & & & \\
& \vdots & & & \\
& b _ {nj} & & & \\
\end{bmatrix} \\
\begin{bmatrix} \\
a _ {i1} & a _ {i2} & \cdots & a _ {in}  \\
\\ \\
\end{bmatrix} & \begin{bmatrix}
& & & & \\
& c _ {ij} & & &  \\
& & & & \\
& & & &
\end{bmatrix}
\end{aligned}
$$

But why is matrix multiplication defined this way, rather than multiplying element-wise just like addition? What is the meaning behind all this strange "row times column"? It is well-known that matrix multiplications can represent the **composition of linear transformations**, and we are pretty sure that one can derive the above formula so that the desired property holds. Still, why does it have this specific structure?

In this article, we explore the deep reason behind the definition of matrix multiplication using **tensors**.

We assume that all vector spaces are over a field $K$, and are *finite-dimensional*.

## Linearity

### Linear Maps

> **Definition.** Let $(V, \oplus, \odot)$ and $(W, \boxplus, \boxdot)$ be vector spaces over $K$. A map $f : V \to W$ is **linear** if for all $v _ 1, v _ 2 \in V$ and $\lambda \in K$,
>
> $$
>f \paren{(\lambda \odot v _ 1) \oplus v _ 2} = (\lambda \boxdot f(v _ 1)) \boxplus f(v _ 2).
> $$

We will drop the special notation for addition and scalar multiplication, and just write $+$ and $\cdot$. It should be clear from context and not cause any confusion. Then we can write the above definition in a familiar way.

> For all $v _ 1, v _ 2 \in V$ and $\lambda \in K$,
>
> $$
>f(\lambda v _ 1 + v _ 2) = \lambda f(v _ 1) + f(v _ 2).
> $$

Some definitions are in order.

> **Definition.** (Isomorphism) A bijective linear map is a **isomorphism** of vector spaces. If vector spaces $V$ and $W$ are isomorphic, we write $V \approx W$.

> **Definition.** The set of linear maps from $V$ to $W$ is denoted as $\rm{Hom}(V, W)$.

> **Definition.** (Endomorphism) An **endomorphism** of $V$ is a linear map from $V$ to itself. We write $\rm{End}(V) = \rm{Hom}(V, V)$.

### Bilinearity

A map is *bilinear* if it is linear in its respective argument. Formally,

> **Definition.** (Bilinearity) Let $V, W, Z$ be vector spaces over $K$. A map $f : V \times W \to Z$ is **bilinear** if
> 1. For any $w \in W$, $f(\lambda v _ 1 + v _ 2, w) = \lambda f(v _ 1, w) + f(v _ 2, w)$ for all $v _ 1, v _ 2 \in V$ and $\lambda \in K$.
> 2. For any $v \in V$, $f(v, \lambda w _ 1 + w _ 2) = \lambda f(v, w _ 1) + f(v, w _ 2)$ for all $w _ 1, w _ 2 \in W$ and $\lambda \in K$.

In other words,

1. The map $v \mapsto f(v, w)$ is linear for fixed $w \in W$.
2. The map $w \mapsto f(v, w)$ is linear for fixed $v \in V$.

Note that this is *not* the same as linear maps in $V \times W \to Z$, since that would mean

$$
f\paren{\lambda(v _ 1, w _ 1) + (v _ 2, w _ 2)} = \lambda f(v _ 1, w _ 1) + f(v _ 2, w _ 2)
$$

for all $v _ 1, v _ 2 \in V$, $w _ 1, w _ 2 \in W$ and $\lambda \in K$. One can check that $(x, y) \mapsto x+ y$ is linear but not bilinear, while the map $(x, y) \mapsto xy$ is bilinear but not linear.

**Example.** Famous examples of bilinear maps.

 - The standard inner product $\span{\cdot, \cdot} : \R^n \times \R^n \to \R$, where $\span{v, w} = v\trans w$.
 - Determinants of $2 \times 2$ matrices $\det _ 2 : \R^2 \times \R^2 \to \R$, where

	$$\rm{det} _ 2 \paren{\begin{bmatrix} a \\ c\end{bmatrix}, \begin{bmatrix} b \\ d\end{bmatrix}} = \det \paren{\begin{bmatrix} a & b \\ c & d \end{bmatrix}} = ad - bc.$$

## Dual Spaces

### Dual Vector Space

To define tensors, we need *dual spaces*.

> **Definition.** (Dual Space) The **dual vector space** of $V$ is defined as
>
> $$
> V^\ast = \rm{Hom}(V, K)
> $$
>
> where $K$ is considered as a vector space over itself.

The dual vector space is the *vector space of linear maps from $V$ to the underlying field $K$*. Its elements are called **linear functionals**, **covectors**, or **one-forms** on $V$.

We can also consider the *double dual* of a vector space.

> **Definition.** The double dual vector space of $V$ is defined as
>
> $$
>V^{\ast\ast} = \rm{Hom}(V^\ast, K).
> $$

We can continue on, but it is unnecessary due to the following result: $V \approx V^{\ast\ast}$.

> **Theorem.** $V$ and $V^{\ast\ast}$ are *naturally isomorphic*.

*Proof*. Define $\psi : V \ra V^{\ast\ast}$ as

$$
\psi(v)(f) = f(v)
$$

for $v \in V$ and $f \in V^\ast$. Then $\psi$ is a natural isomorphism, independent of the choice of basis on $V$.

Furthermore, $V \approx V^\ast$ (finite-dimensional) but they are not naturally isomorphic since an isomorphism from $V \ra V^\ast$ requires a choice of basis. More about this in another article.

### Dual Basis

Since $V$, $V^\ast$ and $V^{\ast\ast}$ are all isomorphic, they all have the same dimension.

> **Lemma.** $\dim V = \dim V^\ast = \dim V^{\ast\ast}$.

Therefore, the bases of $V$ and $V^\ast$ have the same number of elements. We can construct a basis of $V^\ast$ from a basis of $V$.

> **Definition.** (Dual Basis) Let $V$ be a $d$-dimensional vector space with basis $\mc{B} = \braces{e _ 1, \dots, e _ d}$. The **dual basis** is the unique basis
>
> $$
>\mc{B}' = \braces{f^1, \dots, f^d} \subseteq V^\ast
> $$
>
> such that
>
> $$
>f^i (e _ j) = \delta^i _ j = \begin{cases} 1 & (i = j) \\ 0  & (i \neq j) \end{cases}.
> $$

Note that the superscripts are not exponents.

## Tensor Spaces

Tensors are not just high-dimensional arrays of numbers. We approach tensors from an algebraic perspective.

### Tensors

> **Definition.** (Tensor) Let $V$ be a vector space over $K$. A **$(p, q)$-tensor** $T$ on $V$ is a *multi-linear map*
>
> $$
>T : \underbrace{V^\ast \times \cdots \times V^\ast} _ {p \text{ copies}} \times \underbrace{V \times \cdots \times V} _ {q \text{ copies}} \to K.
> $$
>
> We write the set of $(p, q)$-tensors on $V$ as
>
> $$
>T _ q^p V = \underbrace{V \otimes \cdots \otimes V} _ {p\text{ copies}} \otimes \underbrace{V^\ast \otimes \cdots \otimes V^\ast} _ {q\text{ copies}}.
> $$

Thus, **tensors** are **multi-linear maps**.

**Remark.** The order of $V$ and $V^\ast$ are swapped in the definition of tensor and the set of tensors. Although this is just a matter of notation, it can be understood as follows.

A $(p, q)$-tensor $T$ eats $p$ covectors and $q$ vectors to map them to a scalar. Notice that

- An element of $V \approx V^{\ast\ast} = \rm{Hom}(V^\ast,K)$ eats a covector and maps it to a scalar.
- An element of $V^\ast = \rm{Hom}(V, K)$ eats a vector and maps it to a scalar.

Thus, a $(p, q)$-tensor $T$ is sort of an element of $p$ copies of $V$ ($\approx V^{\ast\ast}$) and $q$ copies of $V^\ast$, which kind of justifies the notation in $T _ q^p V$.

### Tensor Spaces

The set $T _ q^pV$ can be equipped with a $K$-vector space structure by defining addition and multiplication component-wise, since tensors are linear maps. Now we have a **tensor space** as a $K$-vector space.

**Example.** Some famous examples of tensor spaces.

- $T _ 1^0 V = V^\ast$ is the set of linear maps $T : V \ra K$, which agrees with the definition of $V^\ast$.
- $T _ 1^1(V) = V \otimes V^\ast$ is the set of bilinear maps $T : V^\ast \times V \to K$.

### Tensor Products

> **Definition.** (Tensor Product) Let $T \in T _ q^pV$ and $S \in T _ s^r V$. The **tensor product** of $T$ and $S$ is the tensor $T \otimes S \in T _ {q+s}^{p+r} V$ defined as
>
> $$
>\begin{aligned}
>&(T\otimes S)(w _ 1, \dots, w _ p, w _ {p+1}, \dots, w _ {p+r}, v _ 1, \dots, v _ q, v _ {q+1}, \dots, v _ {q+s})  \\
>&\quad= T(w _ 1, \dots, w _ p, v _ 1, \dots, v _ q) \cdot S(w _ {p+1}, \dots, w _ {p+r}, v _ {q+1}, \dots, v _ {q+s}).
> \end{aligned}
> $$
>
> for $w _ i \in V^\ast$ and $v _ i \in V$.

The definition seems complicated but is actually very natural. $T \otimes S$ should eat $p+r$ covectors and $q+s$ vectors. Considering the arguments of $T$ and $S$, give $p$ covectors and $q$ vectors to $T$ and the remaining to $S$. Multiply the resulting scalar in $K$.

Note that the $\otimes$ here is different from $\otimes$ used in the definition of $T _ q^p V$.

### Tensors as Components

With these tools, we can completely determine a tensor with its *components*, just like how linear maps are completely determined by its values on the basis vectors.

> **Definition.** Let $V$ be a finite-dimensional $K$-vector space with basis $\mc{B} = \braces{e _ 1, \dots, e _ d}$ and dual basis $\mc{B}' = \braces{f^1, \dots, f^{d}}$.
>
> The **components** of $T \in T _ q^p V$ are defined to be the numbers
>
> $$
>T^{a _ 1 \dots a _ p}  _ {b _ 1\dots b _ q} = T(f^{a _ 1}, \dots, f^{a _ p}, e _ {b _ 1}, \dots, e _ {b _ q}) \in K
> $$
>
> where $1 \leq a _ i, b _ j \leq d = \dim V$.

Notice that $a _ i, b _ j$ range from $1$ to $\dim V$. Since we have $p$ copies of $V^\ast$ and $q$ copies of $V$, the value of $T$ at *every possible combination of basis vectors* are the components.

We can reconstruct a tensor from its components by the tensor product of vectors and covectors from the basis and the dual basis:

$$
T = \underbrace{\sum _ {a _ 1=1}^{\dim V} \cdots \sum _ {b _ q = 1}^{\dim V}} _ {p+q \text{ sums}} T^{a _ 1 \dots a _ p}  _ {b _ 1\dots b _ q} e _ {a _ 1} \otimes \cdots \otimes e _ {a _ p} \otimes f^{b _ 1} \otimes \cdots \otimes f^{b _ q}.
$$

Here, $e _ {a _ i}$ are considered as elements of $T _ 0^1 V \approx V$ and $f^{b _ j}$ as elements of $T _ {1}^0 V \approx V^\ast$.

## Vectors as Tensors

Now that we have components for tensors in $T _ q^p V$ using a basis for $V$ and dual basis of $V^\ast$, we can write vectors and matrices as tensors.

### Vectors

It is well-known that any vector can be represented as a linear combination of the basis vectors. i.e.,

$$
v = v^1 e _ 1 + \cdots + v^d e _ d \in V
$$

where $v^j$ are components of the vector $v$ with respect to each basis vector $e _ j$. We observe that vectors are indeed tensors in $T _ 0^1 V = V$.

Conventionally, we often write vectors as *columns* of numbers.

$$
v = \sum v^i e _ i \quad \longleftrightarrow \quad v = \begin{bmatrix} v^1 \\ \vdots \\ v^d \end{bmatrix}
$$

### Covectors

As for covectors $w \in V^\ast$, we can also represent them as a linear combination. i.e.,

$$
w = w _ 1 f^1 + \cdots + w _ d f^d \in V^\ast
$$

where $w _ i$ are components of the covector $w$ with respect to each dual basis vector $f^i$. We also observe that covectors are tensors in $T _ 1^0 V = V^\ast$.

Also, we often write covectors as *rows* of numbers.

$$
w = \sum w _ i f^i \quad \longleftrightarrow \quad w = \begin{bmatrix} w _ 1 & \dots & w _ d \end{bmatrix}
$$

## Matrices as Tensors

As for matrices we need to prove a few things beforehand. We limit the discussion to square matrices.

First, we know that **matrices are linear transformations**, so every matrix can be considered as a linear map $V \to V$, thus an element of $\rm{End}(V)$.

> **Lemma.** $T _ 1^1 V = V \otimes V^\ast \approx \rm{End}(V^\ast)$.

*Proof*. We must construct an invertible linear map between a tensor $T \in V \otimes V^\ast$ and a linear map $f : V^\ast \to V^\ast$.

For $w \in V^\ast$, simply define $f(w) = T(\cdot, w)$, then $f(w)$ is a map from $V$ to $K$, where $f$ and $f(w)$ can be shown to be linear by the bilinearity of $T$. Thus $f(w) \in V^\ast$ and $f \in \rm{End}(V^\ast)$.

This correspondence is invertible, since given a linear map $f : V^\ast \to V^\ast$, we can define $T$ as $T(v, w) = f(w)(v)$ for $v \in V$ and $w \in V^\ast$. Then $f(w) \in V^\ast$ and $f(w)(v) \in K$, so $T : V \times V^\ast \to K$. Linearity of $T$ follows directly from the linearity of $f$ and $f(w)$.

> **Corollary.** For finite dimensional $K$-vector space $V$, $T _ 1^1 V \approx \rm{End}(V)$.

This directly follows from the fact that $V\approx V^\ast$.

### Matrices as $(1, 1)$-Tensors

Finally, since matrices are elements of $\rm{End}(V)$, we can conclude that **square matrices are $(1, 1)$-tensors**. Therefore, we can write a tensor $\phi \in T _ 1^1 V$ with respect to the chosen basis as

$$
\phi = \sum _ {i=1}^{\dim V} \sum _ {j=1}^{\dim V} \phi^i _ j \; e _ i \otimes f^j,
$$

where $\phi^i _ j = \phi(f^i, e _ j)$. Now it is *very tempting* to think of $\phi^i _ j$ as a **square array of numbers**.

$$
\phi = \sum _ {i=1}^{\dim V} \sum _ {j=1}^{\dim V} \phi^i _ j \; e _ i \otimes f^j \quad \longleftrightarrow \quad \phi =
\begin{bmatrix}
\phi _ 1^1 & \phi _ 2^1 & \cdots & \phi _ d^1 \\
\phi _ 1^2 & \phi _ 2^2 & \cdots & \phi _ d^2 \\
\vdots & \vdots & \ddots & \vdots \\
\phi _ 1^d & \phi _ 2^d & \cdots & \phi _ d^d
\end{bmatrix}
$$

The convention is to consider the top index $i$ as a *row* index and bottom index $j$ as a *column* index.

### Matrix Multiplication

However, the above *arrangement* is a pure convention.

Consider $\phi \in \rm{End}(V)$ also as a tensor $\phi \in T _ 1^1 V$. Abusing the notation, we can write

$$
\phi(w, v) = w(\phi(v))
$$

for $v \in V$ and $w \in V^\ast$. For clarity, $\phi : V^\ast \times V \to K$ on the left hand side. As for the right hand side, $\phi : V \to V$. $w$ eats a vector $\phi(v) \in V$ and maps it to $K$ as it is an element of $V^\ast$.

Thus, the components of $\phi \in \rm{End}(V)$ are $\phi _ j^i = \phi(f^i, e _ j) = f^i(\phi(e _ j))$.

Since **matrix multiplication represents the composition of linear transformation**, for $\phi, \psi \in \rm{End}(V)$, let us consider the components of $\phi \circ \psi$ as a $(1,1)$-tensor. We have

$$
\begin{aligned}
(\phi \circ \psi) _ j^i &= (\phi \circ \psi)(f^i, e _ j) \\
&= f^i\big( (\phi\circ\psi)(e _ j) \big) \\
&= f^i\big( \phi(\psi (e _ j)) \big) \\
&\overset{(\ast)}{=} f^i\paren{ \phi\paren{\sum _ k \psi^k _ j e _ k} } \\
&\overset{(\star)}{=} \sum _ k \psi _ j^k f^i \big( \phi(e _ k) \big) \\
&= \sum _ k \psi _ j^k \phi _ k^i.
\end{aligned}
$$

- $(\ast)$ follows by $\psi(e _ j) = \sum _ k \psi _ j^k e _ k$. This holds since for $w \in V^\ast$,

	$$
	\begin{aligned}
    \psi(w, e _ j) &= \sum _ {k, l} \psi _ l^k \; (e _ k \otimes f^l)(w, e _ j) \\
    &= \sum _ {k, l} \psi _ l^k \; e _ k(w) \cdot f^l(e _ j) \\
    &= \sum _ {k} \psi _ j^k e _ k (w)
    \end{aligned}
	$$

	because $f^l (e _ j) = \delta _ {j}^l$, leaving only the terms on $l=j$.
- $(\star)$ follows by the linearity of $f^i$ and $\phi$.

Doesn't this expression look familiar? Since $K$ is a field,

$$
\sum _ {k} \psi _ j^k \phi _ k^i = \sum _ k \phi^i _ k \psi _ j^k,
$$

which implies that the $(i, j)$-th component of $(1, 1)$-tensor $\phi \circ \psi$ exactly matches the $(i, j)$-th entry of the matrix product $\phi \psi$.

Now we know why matrix multiplication is defined as "row times column". Tensor spaces, tensor products and dual spaces were behind all this.

## Notes

- The above argument should be generalizable to rectangular matrices, although I haven't checked yet.
- Now we know why matrix multiplication is defined in a strange way, but the question is now why tensors are defined in such a strange way, involving vector spaces and their duals.
- (Not sure) Matrix transpose should be interpretable with tensors?

## References

- [Geometrical Anatomy of Theoretical Physics](https://youtube.com/playlist?list=PLPH7f _ 7ZlzxTi6kS4vCmv4ZKm9u8g5yic&si=dJj6nmoAc944YOgl), Lecture 8
