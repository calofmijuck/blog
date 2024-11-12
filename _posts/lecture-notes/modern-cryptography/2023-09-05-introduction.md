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
  - security
  - cryptography
title: 0. Introduction
date: 2023-09-05
github_title: 2023-09-05-introduction
---

## Classical Cryptography

> The "art" of writing or solving codes - Oxford

- *Art* focusing on ensuring private communication between two parties.
- Two parties share secret information in advance. (symmetric key encryption)

## Symmetric Ciphers

Some notations first:
- $\mathcal{M}$ is the plaintext space (messages)
- $\mathcal{K}$ is the key space
- $\mathcal{C}$ is the ciphertext space
- $E: \mathcal{K} \times \mathcal{M} \rightarrow \mathcal{C}$ is the encryption algorithm (sometimes $\mathsf{Enc}_k$)
- $D:\mathcal{K} \times \mathcal{C} \rightarrow \mathcal{M}$ is the decryption algorithm (sometimes $\mathsf{Dec}_k$)

In a **symmetric cipher**, a key $k \in \mathcal{K}$ is used both for encryption and decryption, giving the following correctness requirement.

$$\forall k \in \mathcal{K},\, \forall m \in \mathcal{M},\, D(k, E(k, m)) = m$$

### Caesar Cipher (ROT)

Let $\Sigma$ be the set of lowercase english alphabets.

- $\mathcal{M} = \mathcal{C} = \Sigma^\ast$, where $\Sigma^\ast$ is the [Kleene star](https://en.wikipedia.org/wiki/Kleene_star).
- $\mathcal{K} = \mathbb{Z}_{26}$ (Caesar used $3 \in \mathcal{K}$)
- $E(k, x) = x + k \pmod{26}$ for each letter $x$ of $m \in \mathcal{M}$.
- $D(k, y) = y - k \pmod{26}$ for each letter $y$ of $c \in \mathcal{C}$.

This scheme is not safe since we can try all $26$ keys and find the sentence that makes sense.

#### An Improved Attack Method

Guessing the key and checking the plaintext is hard to automate, since computers don't know what sentences make sense.[^1] In some cases, the message may be invalid in normal English, while the plaintext characters follow the same distribution.

Let $p_i \in [0, 1]$ be the frequency of the $i$-th letter in normal English text. Then it is known that

$$ \sum_{i=0}^{25} p_i^2 \approx 0.065$$

Now, let $q_i \in [0, 1]$ be the frequency of the $i$-th letter in the given ciphertext. For the key $k \in \mathcal{K}$, $p_i \approx q_{i+k}$ for all $i$, since the $i$-th letter is mapped to the $(i + k)$-th letter. (Addition is done modulo $26$). So for each index $j \in [0, 25]$, we compute

$$\sum_{i=0}^{25} p_i q_{i+j}$$

and choose $j$ that gives the closest value to $0.065$.

### Substitution Cipher

Let $\Sigma$ be the set of lowercase english alphabets.

- $\mathcal{M} = \mathcal{C} = \Sigma^\ast$
- $\mathcal{K}$ is the set of all permutations on $\Sigma$.
- $E(k, x) = k(x)$ for each letter $x$ of $m \in \mathcal{M}$.
- $D(k, y) = k^{-1}(y)$ for each letter $y$ of $c \in \mathcal{C}$.

Note that $\lvert \mathcal{K} \rvert = 26! \approx 10^{26.6} \approx 2^{88.4}$, so it is a bit hard to try all cases. To attack this scheme, we use **frequency analysis** on the ciphertext. The frequency of letters in the ciphertext will be similar to the actual distribution of letters in the English language.[^2] For example, if $\texttt{w}$ appears the most in the ciphertext, we can guess that this would have been an $\texttt{e}$.

Also, we could try $n$-letter frequency analysis. For $2$, $3$-letter frequency analysis, we can easily spot prepositions that are used very often, such as $\texttt{the}$. Sometimes, we can guess words from the context. If the message is a letter or an email, the message will probably start with a greeting.

### Vigenère Cipher

A stronger version of Caesar cipher, where the key is a string of length $n$.

- [Wikipedia](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher)

We can guess the key length $n$, and perform frequency analysis on every $mn$-th letter ($m \in \mathbb{N}$) of the ciphertext. If $n$ is very large, we won't have enough letters to perform frequency analysis.

## Modern Cryptography

> **Cryptography** is the practice and study of techniques for secure communication in the presence of adversarial behavior.[^3]

- Modern cryptography developed into more of a science.
	- Schemes are developed and analyzed rigorously and systematically.
	- Goal is to give a rigorous proof that some scheme is secure.
	- Formally define concepts and make appropriate assumptions.
- Covers much broader scope than just encryption/decryption.
	- Data integrity, authentication, zero-knowledge proofs, etc.

### Principles of Modern Cryptography

#### Formal Definitions

For the proper design and analysis of cryptographic primitives, we must formally define what security is. Through a formal definition, we can understand what threats are possible and how much security is desired. Also, definitions enable us to evaluate and analyze the schemes, according to the definitions.

#### Precise Assumptions

Proofs of security are built on precise assumptions, such as the questions in computational complexity theory. For example, many proofs require that $\mathcal{P} ≠ \mathcal{NP}$. We must make such assumptions explicit and precise, so that we can validate the assumptions and compare them. This allows us to modularize the assumptions, which enables us to replace any assumptions if they turn out to be false in the future.

#### Proof of Security

We must provide rigorous proof that the scheme is indeed secure under the assumptions made. With a rigorous proof, we can be sure that some scheme is secure.

### Goals

- **Data privacy**: Messages should be read only by the sender and the receiver.
- **Data integrity**: Messages should not be modified by any adversaries.
- **Data authenticity**: Messages should really be from the sender.

## Advanced Cryptography

- Cryptography beyond encryption and signatures, that also protect the computation itself, not just data.
- Several techniques such as
	- Zero-knowledge proofs
	- Secure multi-party computation
	- Homomorphic encryption
	- Differential privacy

We need advanced cryptography since our private information is being used in return for services, such as maps, health services. These data should be protected!

[^1]: The computer can look up a dictionary to check if some English word exists. Or maybe LLMs can do the job...
[^2]: We assume that this distribution is already known.
[^3]: Rivest, Ronald L. (1990). "Cryptography". In J. Van Leeuwen (ed.). Handbook of Theoretical Computer Science. Vol. 1. Elsevier.
