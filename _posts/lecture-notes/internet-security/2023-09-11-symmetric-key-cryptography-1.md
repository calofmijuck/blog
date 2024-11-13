---
share: true
toc: true
math: true
categories:
  - Lecture Notes
  - Internet Security
path: _posts/lecture-notes/internet-security
tags:
  - security
  - lecture-note
  - cryptography
title: 02. Symmetric Key Cryptography (1)
date: 2023-09-11
github_title: 2023-09-11-symmetric-key-cryptography-1
---

## Symmetric Encryption

- Alice and Bob use the same key for encryption and decryption.
- This was the only type of encryption before the invention of public-key cryptography.

### Requirements

- A strong encryption algorithm, which is known to the public.
	- Kerckhoff's principle!
- A secret key known only to sender and receiver.
- We assume the **existence of a a secure channel for distributing the key**.[^1]
- **Correctness requirement**
	- Let $m$, $k$ denote the message and the key.
	- For encryption/decryption algorithm $E$ and $D$,
	- $D(k, E(k, m)) = m$.

## Cryptographic Attacks

In increasing order of the power of the attacker,

- **Ciphertext only attacks**: the attacker has ciphertexts, and tries to obtain information.
- **Known plaintext attack**: the attacker has a collection of plaintext/ciphertext pairs.
- **Chosen plaintext attack**: the attacker has a collection of plaintext/ciphertext pairs *for any plaintext chosen by the attacker*.
- **Chosen ciphertext attack**: the attacker has a collection of plaintext/ciphertext pairs *for any ciphertext chosen by the attacker*.

## Requirements for a Secure Cipher

The following two properties should hold for a secure cipher.
- **Diffusion** hides the relationship between the ciphertext and the plaintext.
	- It should be hard to obtain the plaintext from the ciphertext.
	- Changing a single bit of the plaintext affects several bits of the ciphertext, and vice versa.
- **Confusion** hides the relationship between the ciphertext and the key.
	- It should be hard to obtain the key from the ciphertext.
	- Each bit of the ciphertext should depend on several parts of the key.

## Primitives

### Substitution Cipher

In **substitution cipher**, encryption is done by replacing units of plaintext with ciphertext, with a fixed algorithm.

#### Caesar Cipher

- Encryption is done by $E(x) = x + 3 \pmod{26}$.
- $E(\texttt{A}) = \texttt{D}$, $E(\texttt{Z}) = \texttt{D}$, etc.
- Decryption can be done by $D(x) = x - 3 \pmod{26}$.
- This scheme is not secure, since we can try all $26$ possibilities.

#### Affine Cipher

- Set two integers $a, b$ for the key.
	- In Caesar cipher, $a = 1$ and $b = 3$.
- Encryption: $E(x) = ax + b \pmod m$.
- Decryption: $D(x) = a^{-1}(x - b) \pmod m$.
- If we use the $26$ alphabets, there are $12$ possible values for $a$, and $26$ possible values for $b$.
	- $a^{-1}$ does not exist for all $m$.
	- We need that $\gcd(a, m) = 1$. The number of possible $a$ values is $\phi(m)$.
- This scheme is not secure either, since we can try all possibilities and check if the message makes sense.

#### Monoalphabetic Substitution Cipher

- The key is any permutation $\pi$ defined on the set $\Sigma = \lbrace \texttt{A}, \texttt{B}, \dots, \texttt{Z} \rbrace$.
	- There are $26!$ possible keys.
	- Note that permutations are bijections.
- Encryption is done by replacing each letter $x$ by $\pi(x)$.
- Decryption is done by replacing each letter $x$ by $\pi^{-1}(x)$.
- This scheme is still not secure, since we can try all possibilities on a *modern* computer.

To attack this scheme, we use frequency analysis. Calculate the frequency of each letter and compare it with the actual distribution of English letters. We could also use *bigrams* (2-letters) for calculating the frequency.

#### Vigenère Cipher

- A polyalphabetic substitution
- Given a key length $m$, take key $k = (k_1, k_2, \dots, k_m)$.
- For the $i$-th letter $x$, set $j = i \bmod m$.
- Encryption is done by replacing $x$ by $x + k_{j}$.
- Decryption is done by replacing $x$ by $x - k_j$.

To attack this scheme, find the key length by [*index of coincidence*](https://en.wikipedia.org/wiki/Index_of_coincidence). Then use frequency analysis.

#### Hill Cipher

- A polyalphabetic substitution
- A key is a *invertible* matrix $K = (k_{ij})_{m \times m}$ where $k_{ij} \in \mathbb{Z}_{26}$.
- Encryption/decryption is done by multiplying $K$ or $K^{-1}$.

This scheme is vulnerable to known plaintext attack, since the equation can be solved for $K$.

### Transposition Cipher

- Positions held by units of plaintext are shifted using some permutation.
- Also known as permutation cipher.

#### Columnar Cipher

- Set the number of columns $n$.
- For the key, use a permutation defined on a set of $n$ elements.
	- There are $n!$ possible keys.
- Write the plaintext message in row major order.
- To encrypt, reorder the columns by the chosen permutation.
	- Then the ciphertext is taken by taking letters in column major order.

##### Example

Suppose we encrypt the following text:

$$
\texttt{CRYPTOGRAPHY INTERNET SECURITY}
$$

Choose a key $\sigma = (1, 4, 5, 2, 3, 6)$. Then

$$
\begin{matrix} \\
4 & 3 & 6 & 5 & 2 & 1 \\ \hline
\texttt{C} & \texttt{R} & \texttt{Y} & \texttt{P} & \texttt{T} & \texttt{O} \\
\texttt{G} & \texttt{R} & \texttt{A} & \texttt{P} & \texttt{H} & \texttt{Y} \\
\texttt{I} & \texttt{N} & \texttt{T} & \texttt{E} & \texttt{R} & \texttt{N} \\
\texttt{E} & \texttt{T} & \texttt{S} & \texttt{E} & \texttt{C} & \texttt{U} \\
\texttt{R} & \texttt{I} & \texttt{T} & \texttt{Y}
\end{matrix}
$$

Now reorder the columns,

$$
\begin{matrix} \\
1 & 2 & 3 & 4 & 5 & 6 \\ \hline
\texttt{O} & \texttt{T} & \texttt{R} & \texttt{C} & \texttt{P} & \texttt{Y} \\
\texttt{Y} & \texttt{H} & \texttt{R} & \texttt{G} & \texttt{P} & \texttt{A} \\
\texttt{N} & \texttt{R} & \texttt{N} & \texttt{I} & \texttt{E} & \texttt{T} \\
\texttt{U} & \texttt{C} & \texttt{T} & \texttt{E} & \texttt{E} & \texttt{S} \\
&& \texttt{I} & \texttt{R} & \texttt{Y} & \texttt{T} 
\end{matrix}
$$

The ciphertext is

$$
\texttt{OYNU THRC RRNTI CGIER PPEEY YATST}.
$$

The decryption process is the reverse of this operation. It seems to be breakable by inspecting the $i$-th letter of each block and reordering the letters to check if any reordering makes sense.

### Exclusive OR (XOR)

- A bitwise operation $x \oplus y = x + y \pmod 2$.
- For the message $m$, key $k \in \lbrace 0, 1 \rbrace^n$,
	- Encryption is done by $E(k, x) = x \oplus k$.
	- Decryption is done by $D(k, y) = y \oplus k$.
	- Correctness: $D(k, E(k, x)) = (x \oplus k) \oplus k = x$.
- Vulnerable to known plaintext attack, since if $c = m \oplus k$, then $k = c \oplus m$.

#### A crucial property of XOR.

> **Theorem.** Suppose that the message $M$ has an arbitrary distribution over $\lbrace 0, 1 \rbrace^n$. If the key $K$ is independently uniformly distributed over $\lbrace 0, 1 \rbrace^n$, then $C = M \oplus K$ is also uniformly distributed.

*Proof*. Let $n = 1$.

$$
\begin{align*}
\Pr[C = 0] &= \Pr[M = 0 \land K = 0] + \Pr[M = 1 \land K = 1] \\ &= \Pr[M = 0] \cdot \Pr[K = 0] + \Pr[M = 1] \cdot \Pr[K = 1] \\
&= \frac{1}{2}\left(\Pr[M = 0] + \Pr[M = 1]\right) \\
&= \frac{1}{2}.
\end{align*}
$$

The case for $C = 1$ is similar.

### One-Time Pad (OTP)

Let $m \in \left\lbrace 0, 1 \right\rbrace^n$ be the message to encrypt. Then choose a *random* key $k \in \left\lbrace 0, 1 \right\rbrace^n$, and XOR $k$ and $m$.

- Encryption: $E(k, m) = k \oplus m$.
- Decryption: $D(k, c) = k \oplus c$.

This scheme is **provably secure**. See also [one-time pad (Modern Cryptography)](../modern-cryptography/2023-09-07-otp-stream-cipher-prgs.md#one-time-pad-(otp)).

## Perfect Secrecy

> **Definition.** Let $(E, D)$ be a cipher defined over $(\mathcal{K}, \mathcal{M}, \mathcal{C})$. We assume that $\lvert \mathcal{K} \rvert = \lvert \mathcal{M} \rvert = \lvert \mathcal{C} \rvert$. The cipher is **perfectly secure** if for all $m \in \mathcal{M}$ and $c \in \mathcal{C}$,
>
> $$
> \Pr[\mathcal{M} = m \mid \mathcal{C} = c] = \Pr[\mathcal{M} = m].
> $$
>
> Or equivalently, for all $m_0, m_1 \in \mathcal{M}$, $c \in \mathcal{C}$,
>
> $$
> \Pr[E(k, m_0) = c] = \Pr[E(k, m_1) = c]
> $$ 
>
> where $k$ is chosen uniformly in $\mathcal{K}$.

In other words, the adversary learns nothing from the ciphertext.

With this definition, we can show that **OTP is perfectly secure**. For all $m \in \mathcal{M}$ and $c \in \mathcal{C}$,

$$
\Pr[E(k, m) = c] = \frac{1}{\lvert \mathcal{K} \rvert}
$$

since for each $m$ and $c$, $k$ is determined uniquely.

### Conditions for Perfect Secrecy

> **Theorem.** If $(E, D)$ is perfectly secure, $\lvert \mathcal{K} \rvert \geq \lvert \mathcal{M} \rvert$.

*Proof*. Assume not, then we can find some message $m_0 \in \mathcal{M}$ such that $m_0$ is not a decryption of some $c \in \mathcal{C}$. This is because the decryption algorithm $D$ is deterministic and $\lvert \mathcal{K} \rvert < \lvert \mathcal{M} \rvert$.

For the proof in detail, check [Shannon's Theorem (Modern Cryptography)](../modern-cryptography/2023-09-07-otp-stream-cipher-prgs.md#shannon's-theorem).

### Two-Time Pad is Insecure

It is not secure to use the same key twice. If for the key $k$ and two messages $m_1$, $m_2$,

$$
c_1 \oplus c_2 = (k \oplus m_1) \oplus (k \oplus m_2) = m_1 \oplus m_2.
$$

So some information is leaked, even though we cannot actually recover $m_i$ from the above equation.

## Two Types of Symmetric Ciphers

- **Stream cipher**: encrypt one bit/byte at a time
	- Generating a random key is difficult.
	- No message integrity or authentication.
	- Ex. RC4
- **Block cipher**: encrypt a block of bits at a time
	- Can provide integrity or authentication.
	- Block ciphers usually have feedback between blocks, so errors during transmission will be propagated during the decryption process.
	- Ex. DES, AES

### Stream Cipher

We start with a secret key called **seed** with size $s$, and generate a random stream using a **pseudo random generator**. (PRG) The PRG is a function $\mathsf{Gen}: \lbrace 0, 1 \rbrace^s \rightarrow \lbrace 0, 1 \rbrace^n$, so use $\mathsf{Gen}(k)$ as the key for the one-time pad.

Stream cipher does not have perfect secrecy, since the key length is shorter than the message length. It is known that the security of stream ciphers depend on the security of PRGs.

### Linear Feedback Shift Register (LFSR)

The seed can be used in a **linear feedback shift register** (LFSR) to generate the actual key for the stream cipher. There are $n$ stages (or states) and the generated key stream is periodic with maximal period $2^n - 1$.

The links between stages may be different. But in general, if one is given $2n$ output bits of LFSR, one can solve the $n$-stage LFSR.

To alleviate this problem, we can combine multiple LFSRs with a $k$-input binary boolean function, so that we have high non-linearity, long period, and low correlation with the input bits.

## Case Study: Wi-Fi WEP

- Wi-Fi 802.11b WEP (Wired Equivalent Privacy)
	- Encryption in the link layer
	- **Misuse of the stream cipher RC4.**

### WEP

#### Encryption Overall

- Plaintext: Message + CRC
- CRC is padded to verify the integrity of the message.
	- CRC is $32$ bits
	- Not for attacks, but for error correction
- Initialization vector (IV): $24$ bit
- Key: $104$ bit number to build the keystream
- IV and the key is used to build the keystream $k_s$
	- IV + Key is $128$ bits
- Encryption: $c = k_s \oplus (m \parallel \mathrm{CRC}(m))$

#### Encryption Process

1. Compute CRC for the message
	- CRC-32 polynomial is used
2. Compute the keystream from IV and the key
	- IV is concatenated with the key.
	- $128$ bit input is given to the key generation algorithm.
3. Now encrypt the plaintext with XOR.
	- The IV is prepended to the ciphertext, since the receiver needs it to decrypt.

#### Decryption Process

1. Compute the keystream from IV and the key
	- Extract the IV from the incoming frame
2. Decrypt the ciphertext with XOR
3. Verify the extracted message with the CRC

### Initialization Vector

- The IV is not encrypted, and carried in plaintext.
- IV is only $24$ bits, so around $16$ million possible IVs.
- **IV must be different for every message transmitted.**
- 802.11 standard doesn't specify how IV is calculated.
	- Usually increment by $1$ for each frame.
	- No restrictions on reusing the IV.

#### IV Collision

- The key is fixed, and the period of IV is $2^{24}$.
- Same IV leads to same key stream.
- So if the adversary takes two frames with the same IV to obtain the XOR of two plaintext messages.
	- $c_1 \oplus c_2 = (p_1 \oplus k_s) \oplus (p_2 \oplus k_s) = p_1 \oplus p_2$
- Since network traffic contents are predictable, messages can be recovered.
	- We are in the link layer, so HTTP, IP, TCP headers will be contained in the encrypted payload.
	- The header formats are usually known.

#### CRC Algorithm

Given a bit string (defined in the specification), the sender performs long division on the data. The remainder is the result of the CRC, which is appended to the data. The receiver will check by performing long division, and the remainder should be $0$ if there were no bit errors during transmission.

### Message Modification

- CRC is actually a linear function.
	- $\mathrm{CRC}(x \oplus y) = \mathrm{CRC}(x) \oplus \mathrm{CRC}(y)$.
	- The remainder of $x \oplus y$ is equal to the sum of the remainders of $x$ and $y$, since $\oplus$ is effectively an addition over $\mathbb{Z}_2$.
- CRC function doesn't have a key, so it is forgeable.
- **RC4 is transparent to XOR**, and messages can be modified.
	- Let $c = k_s \oplus (m \parallel \mathrm{CRC}(m))$.
	- If we XOR $(x \parallel \mathrm{CRC}(x))$, where $x$ is some malicious message.
	- $c \oplus (x \parallel \mathrm{CRC}(x)) = k_s \oplus (m\oplus x \parallel \mathrm{CRC}(m\oplus x))$.
	- The receiver will decrypt and get $(m\oplus x \parallel \mathrm{CRC}(m\oplus x))$.
	- CRC check by the receiver will succeed.

[^1]: This assumption will be removed when we learn public key cryptography.
