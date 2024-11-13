---
share: true
toc: true
math: true
categories:
  - Algorithms
  - Data Structures
path: _posts/algorithms/data-structures
tags:
  - algorithms
  - data-structures
title: Search Time in Hash Tables
date: 2024-04-12
github_title: 2024-04-12-search-time-hash-tables
---

Here are the expected time complexities of the search operation in hash tables.

## Assumptions

- Let $m$ be the number of buckets in the hash table.
- Let $n$ be the number of entries currently in the hash table.
- Let $\alpha = n/m$ be the *load factor*.
- Elements are uniformly hashed to each bucket of the hash table.

These results imply that the `search` operation takes almost constant time.

## Hashing with Chaining

### Unsuccessful Search

> **Theorem.** The expected time complexity of an unsuccessful search in a hash table using chaining is $\alpha$.

*Proof*. Observe that since elements are hashed uniformly into each bucket, the expected number of elements in each bucket is the same for all buckets. By linearity of expectation, their sum should equal $n$. So each bucket has $\alpha = n/m$ expected number of elements. Thus, on an unsuccessful search, we search at most $\alpha$ elements.

### Successful Search

> **Theorem.** The expected time complexity of a successful search in a hash table using chaining is $1 + \frac{\alpha}{2} - \frac{\alpha}{2n}$.

*Proof*. Suppose we are looking for the element $x$. The number of elements to search is determined by the number of elements inserted after $x$, whose hash collided with $x$.

The probability of collision is $\frac{1}{m}$, since the hash function is uniform by assumption. If $x$ was inserted as the $i$-th element, the number of elements to search equals

$$
1 + \sum _ {j = i + 1}^n \frac{1}{m}.
$$

The additional $1$ comes from searching $x$ itself. Averaging over all $i$ gives the final result.

$$
\begin{aligned}
\frac{1}{n}\sum _ {i=1}^n \paren{1 + \sum _ {j=i+1}^n \frac{1}{m}} &= 1 + \frac{1}{mn} \sum _ {i=1}^n \sum _ {j=i+1}^n 1 \\
&= 1 + \frac{1}{mn}\paren{n^2 - \frac{n(n+1)}{2}} \\
&= 1 + \frac{n(n-1)}{2mn} \\
&= 1+ \frac{(n-1)\alpha}{2n} = 1+ \frac{\alpha}{2} - \frac{\alpha}{2n}.
\end{aligned}
$$

## Hashing with Open Addressing

For open addressing, we first assume that $\alpha < 1$. The case $\alpha = 1$ will be handled separately. Also, we assume no deletion.

### Unsuccessful Search

> **Theorem.** The expected time complexity of an unsuccessful search in a hash table using open addressing is $\frac{1}{1-\alpha}$.

*Proof*. Let the random variable $X$ be the number of probes made in an unsuccessful search. We want to find $\bf{E}[X]$, so we use the identity

$$
\bf{E}[X] = \sum _ {i \geq 1} \Pr[X \geq i].
$$

We want to find a bound for $\Pr[X \geq i]$. For $X \geq i$ to happen, $i - 1$ probes must fail, i.e., it must probe to an occupied bucket. On the $j$-th probe, there are $m - j + 1$ buckets left to be probed, and $n - j + 1$ elements not probed yet. Thus the $j$-th probe fails with probability $\frac{n - j + 1}{m - j + 1} < \frac{n}{m}$. Therefore,

$$
\begin{aligned}
\Pr[X \geq i] &= \frac{n}{m} \cdot \frac{n - 1}{m - 1} \cdot \cdots \cdot \frac{n - (i - 2)}{m - (i - 2)} \\
&\leq \paren{\frac{n}{m}}^{i-1} = \alpha^{i - 1}.
\end{aligned}
$$

Now we have

$$
\bf{E}[X] = \sum _ {i \geq 1} \Pr[X \geq i] \leq \sum _ {i\geq 1} \alpha^{i-1} = \frac{1}{1 - \alpha}.
$$

### Successful Search

> **Theorem.** The expected time complexity of a successful search in a hash table using open addressing is $\frac{1}{\alpha} \log \frac{1}{1- \alpha}$.

*Proof*. On a successful search, the sequence of probes is exactly the same as the sequence of probes when that element was inserted.

Suppose that an element $x$ was the $i$-th inserted element. At the moment of insertion, the load factor is ${} \alpha _ i = (i-1)/m {}$. By the above theorem, the expected number of probes must have been ${} 1/(1 -\alpha _ i) = \frac{m}{m-(i-1)} {}$. Averaging this over all $i$ gives

$$
\begin{aligned}
\frac{1}{n} \sum _ {i=1}^n \frac{m}{m - (i - 1)} &= \frac{m}{n} \sum _ {i=0}^{n-1} \frac{1}{m - i} \\
&\leq \frac{1}{\alpha} \int _ {m-n}^m \frac{1}{x}\,dx \\
&= \frac{1}{\alpha} \log \frac{1}{1-\alpha}.
\end{aligned}
$$

### When the Hash Table is Full ($\alpha = 1$)

First of all, on an unsuccessful search, all $m$ buckets should be probed.

On a successful search, set $m = n$ on the above argument, then the average number of probes is

$$
\frac{1}{m} \sum _ {i=1}^m \frac{m}{m - (i - 1)} = \sum _ {i=1}^m \frac{1}{i} = H _ m,
$$

where $H _ m$ is the $m$-th harmonic number.
