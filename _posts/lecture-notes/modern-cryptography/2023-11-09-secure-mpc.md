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
title: 14. Secure Multiparty Computation
date: 2023-11-09
github_title: 2023-11-09-secure-mpc
---

## Secure Multiparty Computation (MPC)

Suppose we have a function $f$ that takes $n$ inputs and produces $m$ outputs.

$$
(y_1, \dots, y_m) = f(x_1, \dots, x_n).
$$

$N$ parties $P_1, \dots, P_N$ are trying to evaluate this function with a protocol. Each $x_i$ is submitted by one of the parties, and each output $y_j$ will be given to one or more parties.

In **secure multiparty computation** (MPC), we wish to achieve some security functionalities.

- **Privacy**: no party learns anything about any other party's inputs, except for the information in the output.
- **Soundness**: honest parties compute correct outputs.
- **Input independence**: all parties must choose their inputs independently of other parties' inputs.

Security must hold even if there is any adversarial behavior in the party.

### Example: Secure Summation

Suppose we have $n$ parties $P_1, \dots, P_n$ with private values $x_1, \dots, x_n$. We would like to *securely* compute the sum $s = x_1 + \cdots + x_n$.

> 1. Choose $M$ large enough so that $M > s$.
> 2. $P_1$ samples $r \la \Z_M$ and computes $s_1 = r + x_1 \pmod M$ and sends it to $P_2$.
> 3. In the same manner, $P_i$ computes $s_i = s_{i-1} + x_i \pmod M$ and sends it to $P_{i+1}$.
> 4. As the final step, $s_n$ is returned to $P_1$, where he outputs $s = s_n - r \pmod M$.

This protocol seems secure since $r$ is a random noise added to the actual partial sum. But the security actually depends on how we model adversarial behavior.

Consider the case where parties $P_2$ and $P_4$ team up (collusion). These two can share information between them. They have the following:

- $P_2$ has $s_1$, $s_2$, $x_2$.
- $P_4$ has $s_3$, $s_4$, $x_4$.

Using $s_2$ and $s_3$, they can compute $x_3 = s_3 - s_2$ and obtain the input of $P_3$. This violates privacy. Similarly, if $P_i$ and $P_j$ team up, the can compute the partial sum

$$
s_{j - 1} - s_{i} = x_{i+1} + \cdots + x_{j-1}
$$

which leaks information about the inputs of $P_{i+1}, \dots, P_{j-1}$.

## Modeling Adversaries for Multiparty Computation

The adversary can decide not to follow the protocol and perform arbitrarily.

- **Semi-honest** adversaries follows the protocol and tries to learn more information by inspecting the communication.
- **Malicious** adversaries can behave in any way, unknown to us.

Semi-honest adversaries are similar to *passive* adversaries, whereas malicious adversaries are similar to *active* adversaries.

We can also model the **corruption strategy**. Some parties can turn into an adversary during the protocol.

- In **static** corruptions, the set of adversarial parties is fixed throughout the execution.
- In **adaptive** corruptions, the adversary corrupts parties during the execution, based on the information gained from the protocol execution.

We can decide how much computational power to give to the adversary. For *computational security*, an adversary must be efficient, only polynomial time strategies are allowed. For *information-theoretic security*, an adversary has unbounded computational power.

We will only consider **semi-honest** adversaries with **static** corruptions.

## Defining Security for Multiparty Computation

The idea is the following.

> An attack on the protocol in the **real world** is equivalent to some attack on the protocol in an **ideal world** in which no damage can be done.

In the **ideal world**, we use a trusted party to implement a protocol. All parties, both honest and corrupted, submit their input to the trusted party. Since the trusted party is not corrupted, the protocol is safe.

In the **real world**, there is no trusted party and parties must communicate with each other using a protocol.

Thus, a secure protocol must provide security in the real world that is equivalent to that in the ideal world. The definition is saying the following: **there is no possible attack in the ideal world, so there is no possible attack in the real world**. This kind of definition implies privacy, soundness and input independence.

> For every efficient adversary $\mc{A}$ in the real world, there exists an *equivalent* efficient adversary $\mc{S}$ (usually called a **simulator**) in the ideal world.

### Semi-Honest & Static Corruption

- The *view* of a party consists of its input, random tape and the list of messages obtained from the protocol.
	- The view of an adversary is the union of views of corrupted parties.
	- If an adversary learned anything from the protocol, it must be efficiently computable from its view.
- If a protocol is secure, it must be possible in the ideal world to generate something indistinguishable from the real world adversary's view.
	- In the ideal world, the adversary's view consists of inputs/outputs to and from the trusted party.
	- An adversary in the ideal world must be able to generate a view equivalent to the real world view. We call this ideal world adversary a **simulator**.
	- If we show the existence of a simulator, a real world adversary's ability is the same as an adversary in the ideal world.

> **Definition.** Let $\mc{A}$ be the set of parties that are corrupted, and let $\rm{Sim}$ be a simulator algorithm.
> - $\rm{Real}(\mc{A}; x_1, \dots, x_n)$: each party $P_i$ runs the protocol with private input $x_i$. Let $V_i$ be the final view of $P_i$. Output $\braces{V_i : i \in \mc{A}}$.
> - $\rm{Ideal}_\rm{Sim}(x_1, \dots, x_n)$: output $\rm{Sim}(\mc{A}; \braces{(x_i, y_i) : i \in \mc{A}})$.
> 
> A protocol is **secure against semi-honest adversaries** if there exists a simulator such that for every subset of corrupted parties $\mc{A}$, its views in the real and ideal worlds are indistinguishable.

## Oblivious Transfer (OT)

This is a building block for building any MPC.

Suppose that the sender has data $m_1, \dots, m_n \in \mc{M}$, and the receiver has an index $i \in \braces{1, \dots, n}$. The sender wants to send exactly one message and hide others. Also, the receiver wants to hide which message he received.

This problem is called 1-out-of-$n$ **oblivious transfer** (OT).

### 1-out-of-2 OT Construction from ElGamal Encryption

We show an example of 1-out-of-2 OT using the ElGamal encryptions scheme. We use a variant where a hash function is used in encryption.

It is known that $k$-out-of-$n$ OT is constructible from 1-out-of-2 OTs.

> Suppose that the sender Alice has messages $x_0, x_1 \in \braces{0, 1}\conj$, and the receiver Bob has a choice $\sigma \in \braces{0, 1}$.
> 
> 1. Bob chooses $sk = \alpha \la \Z_q$ and computes $h = g^\alpha$, and chooses $h' \la G$.
> 2. Bob sets $pk_\sigma = h$ and $pk_{1-\sigma} = h'$ and sends $(pk_0, pk_1)$ to Alice.
> 3. Alice encrypts each $x_i$ using $pk_i$, obtains two ciphertexts.
> 	- $\beta_0, \beta_1 \la \Z_q$.
> 	- $c_0 = \big( g^{\beta_0}, H(pk_0^{\beta_0}) \oplus x_0 \big)$, $c_1 = \big( g^{\beta_1}, H(pk_1^{\beta_1}) \oplus x_1 \big)$.
> 4. Alice sends $(c_0, c_1)$ to Bob.
> 5. Bob decrypts $c_\sigma$ with $sk$ to get $x_\sigma$.

Correctness is obvious.

Alice's view contains the following: $x_0, x_1, pk_0, pk_1, c_0, c_1$. Among these, $pk_0, pk_1$ are the received values from Bob. But these are random group elements, so she learns nothing about $\sigma$. The simulator can choose two random group elements to simulate Alice.

Bob's view contains the following: $\sigma, \alpha, g^\alpha, h', c_0, c_1, x_\sigma$. He only knows one private key, so he only learns $x_\sigma$, under the DL assumption. (He doesn't have the discrete logarithm for $h'$) The simulator must simulate $c_0, c_1$, so it encrypts $x_\sigma$ with $pk_\sigma$, and as for $x_{1-\sigma}$, a random message is encrypted with $pk_{1-\sigma}$. This works because the encryption scheme is semantically secure, meaning that it doesn't reveal any information about the underlying message.

The above works for **semi-honest** parties. To prevent malicious behavior, we fix the protocol a bit.

> 1. Alice sends a random $w \la G$ first.
> 2. Bob must choose $h$ and $h'$ so that $hh' = w$. $h$ is chosen the same way, and $h' = wh\inv$ is computed.
> 
> The remaining steps are the same, except that Alice checks if $pk_0 \cdot pk_1 = w$.

Bob must choose $h, h'$ such that $hh' = w$. If not, Bob can choose $\alpha' \la \Z_q$ and set $h' = g^{\alpha'}$, enabling him to decrypt both $c_0, c_1$, revealing $x_0, x_1$. Under the DL assumption, Bob cannot find the discrete logarithm of $h'$, which prevents malicious behavior.

### 1-out-of-$n$ OT Construction from ElGamal Encryption

Let $m_1, \dots, m_n \in \mc{M}$ be the messages to send, and let $i$ be an index. We will use ElGamal encryption on a cyclic group $G = \span{g}$ of prime order, with a hash function and a semantically secure symmetric cipher $(E_S, D_S)$.

> 1. Alice chooses $\beta \la \Z_q$, computes $v \la g^\beta$ and sends $v$ to Bob.
> 2. Bob chooses $\alpha \la \Z_q$, computes $u \la g^\alpha v^{-i}$ and sends $u$ to Alice.
> 3. For $j = 1, \dots, n$, Alice computes the following.
> 	- Compute $u_j \la u \cdot v^j = g^\alpha v^{j-i}$ as the public key for the $j$-th message.
> 	- Encrypt $m_j$ as $(g^\beta, c_j)$, where $c_j \la E_S\big( H(g^\beta, u_j^\beta), m_j \big)$.
> 4. Alice sends $(c_1, \dots, c_n)$ to Bob.
> 5. Bob decrypts $c_i$ as follows.
> 	- Compute symmetric key $k \la H(v, v^\alpha)$ where $v = g^\beta$ from step $1$.
> 	- $m_i \la D_S(k, c_i)$.

Note that all ciphertexts $c_j$ were created from the same ephemeral key $\beta \in \Z_q$.

For correctness, we check that Bob indeed receives $m_i$ from the above protocol. Check that $u_i = u\cdot v^i = g^\alpha v^0 = g^\alpha$, then $u_i^\beta = g^{\alpha\beta} = v^\alpha$. Since $c_i = E_S\big( H(g^\beta, u_i^\beta), m_i \big) = E_S\big( H(v, v^\alpha), m_i \big)$, the decryption gives $m_i$.

Now is this oblivious? All that Alice sees is $u = g^\alpha v^{-i}$ from Bob. Since $\alpha \la \Z_q$, $u$ is uniformly distributed over elements of $G$. Alice learns no information about $i$.

As for Bob, we need the **CDH assumption**. Suppose that Bob can query $H$ on two different ciphertexts $c_{j_1}, c_{j_2}$. Then he knows

$$
u_{j_1}^\beta/u_{j_2}^\beta = v^{\beta(j_1 - j_2)},
$$

and by raising both to the $(j_1 - j_2)\inv$ power (inverse in $\Z_q$), he can compute $v^\beta = g^{\beta^2}$. Thus, Bob has computed $g^{\beta^2}$ from $g^\beta$, and this breaks the CDH assumption.[^1] Thus Bob cannot query $H$ on two points, and is unable to decrypt two ciphertexts. He only learns $m_i$.

### OT for Computing $2$-ary Function with Finite Domain

We can use an OT for computing a $2$-ary function with finite domain.

Let $f : X_1 \times X_2 \ra Y$ be a deterministic function with $X_1$, $X_2$ both finite. There are two parties $P_1, P_2$ with inputs $x_1, x_2$, and they want to compute $f(x_1, x_2)$ without revealing their input.

Then we can use $1$-out-of-$\abs{X_2}$ OT to securely compute $f(x_1, x_2)$. Without loss of generality, suppose that $P_1$ is the sender.

$P_1$ computes $y_x =f(x_1, x)$ for all $x \in X_2$, resulting in $\abs{X_2}$ messages. Then $P_1$ performs 1-out-of-$\abs{X_2}$ OT with $P_2$. The value of $x_2$ will be used as the choice of $P_2$, which will be oblivious to $P_1$.[^2]

This method is inefficient, so we have better methods!

[^1]: Given $g^\alpha, g^\beta$, compute $g^{\alpha + \beta}$. Then compute $g^{\alpha^2}, g^{\beta^2}, g^{(\alpha+\beta)^2}$, and obtain $g^{2\alpha\beta}$. Exponentiate by $2\inv \in \Z_q$ to find $g^{\alpha\beta}$.
[^2]: Can $P_1$ learn the value of $x_2$ from the final output $y_{x_2} = f(x_1, x_2)$?
