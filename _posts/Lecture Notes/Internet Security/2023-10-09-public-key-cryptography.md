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
title: 07. Public Key Cryptography
date: 2023-10-09
github_title: 2023-10-09-public-key-cryptography
---

In symmetric key cryptography, we have a problem with key sharing and management. More info in the first few paragraphs of [Key Exchange (Modern Cryptography)](../../modern-cryptography/2023-10-03-key-exchange).

## Public Key Cryptography

We use **two** keys for public key cryptography. The keys are called *public key* and *private key*. These two keys are related to each other, but it is almost impossible to calculate the private key from the public key.

- **Public key** is *public*, and anyone can use it to encrypt messages or verify signatures.
- **Private key** (or secret key) is only kept by the owner. It is used to decrypt messages or create signatures.

We will denote public keys as $pk$ and private keys as $sk$.

These keys are created to be used in **trapdoor one-way functions**.

### One-way Function

A **one-way function** is a function that is easy to compute, but hard to compute the pre-image of any output. Here are some common examples.

- *Cryptographic hash functions*: [Hash Functions (Modern Cryptography)](../../modern-cryptography/2023-09-28-hash-functions#collision-resistance).
- *Factoring a large integer*: It is easy to multiply to integers even if they're large, but factoring is very hard.
- *Discrete logarithm problem*: It is easy to exponentiate a number, but it is hard to find the discrete logarithm.

But a one-way function is not enough. Suppose that $f$ is a one way function with a public key $pk$. It will be easy to encrypt a message $m$ as $f(pk, m)$, but recovering $m$ is hard even for the intended recipient.

### Trapdoor One-way Function

A **trapdoor one-way function** has a *trapdoor*. It is computationally difficult to find the preimage, but with the trapdoor, the inverting is easy.

In public key cryptography, the trapdoor is the *private key* that makes it easy to invert the one-way function $f$. So the recipient can efficiently invert $f$ and recover the message $m$.

### Encryption and Decryption

In public key cryptography, encryption and decryption are done as follows.

Suppose that Alice wants to send a secret message to Bob. Alice must encrypt the message using **Bob's public key**, so that only Bob can decrypt the message.

> 1. Alice takes a plaintext and encrypts it using Bob's public key.
> 2. The ciphertext is sent to Bob.
> 3. Bob uses his private key to decrypt the ciphertext.

Mathematically, let $pk, sk$ be Bob's public key and private key.

> 1. Alice computes the ciphertext $c = f(pk, m)$ of the message $m$.
> 2. $c$ is sent to Bob.
> 3. Bob computes $m = f^{-1}(sk, c)$ and recovers $m$.

### Authentication

Public key cryptography can be used also for **authentication**. If some ciphertext can be decrypted with Alice's public key, we can verify that the message was from Alice.

We will learn more about this when we learn digital signatures.

### Applications of Public Key Cryptography

- **Encryption and decryption**: for private communication.
- **Digital signatures**: authentication, as explained above.
	- This was not possible with symmetric cryptography since both parties have the key, so does not satisfy non-repudiation.
- **Key exchange**
	- We assumed that in symmetric cryptography, there was a secure channel to share the secret key.
	- We use public key cryptography to exchange and agree on the secret key for the symmetric cipher.
	- Public key cryptography takes longer to calculate, so it is preferable to use symmetric ciphers.

But a problem still remains. How does one verify that this key is indeed from that identity? In the example above, how does Alice know that this public key is from Bob and not someone else's? This problem will be solved using **public key infrastructure**.

## Diffie-Hellman Key Exchange

Choose a large prime $p$ and a generator $g$ of $\mathbb{Z}_p^{ * }$. The description of $g$ and $p$ will be known to the public.

> 1. Alice chooses some $x \in \mathbb{Z}_p^{ * }$ and sends $g^x \bmod p$ to Bob.
> 2. Bob chooses some $y \in \mathbb{Z}_p^{ * }$ and sends $g^y \bmod p$ to Alice.
> 3. Alice and Bob calculate $g^{xy} \bmod p$ separately.
> 4. Eve can see $g^x \bmod p$, $g^y \bmod p$ but cannot calculate $g^{xy} \bmod p$.

Refer to [Diffie-Hellman Key Exchange (Modern Cryptography)](../../modern-cryptography/2023-10-03-key-exchange#diffie-hellman-key-exchange-dhke).

## Message Integrity

A function $H$ takes an input of arbitrary length message and outputs a fixed length string. The output is called **message digest**, *tag*, *fingerprint* or **hash**.

Here, the $H$ is called a **hash function**. This function is many-to-one, but it is usually computationally infeasible to find a collision.

**Desirable Properties of $H$**.

- $H$ should be easy to calculate.
- It should be hard to recover $m$ from $H(m)$. (one-wayness)
- It should be computationally difficult to find a collision. (collision resistance)
- The output should seem random.

Using this function, we can check whether if the message was tampered during transmission.

### Message Authentication Code (MAC)

We assume that Alice and Bob already share a secret $k$. Alice wants to send a message $m$ to Bob.

> 1. Alice signs the message using the key and calculates the tag $t = H(k, m)$.
> 2. Alice sends the message and tag.
> 3. Bob calculates the tag $t'$ from the received message. If $t'$ does not match with $t$, Bob detects that the message was modified.

We only care about message integrity in MACs, so the message is not encrypted.

### Properties of MAC

- MACs are based on symmetric keys, so communicating parties must share a key.
- MACs should be able to accept messages of arbitrary length.
- MACs should output a fixed-length string.
- MACs should provide message integrity. Any manipulations in transit will be detected, and receiving party is assured of the origin of the message.
- MACs **do not** support non-repudiation.
	- Since both parties have the secret, any two party can create the message.

## Digital Signatures

**Digital signatures** achieve *integrity*, *non-repudiation* and *authentication*. We leverage public key cryptography.

Suppose Alice wants to **sign** a message $m$. Alice has public key $pk$ and private key $sk$.

> 1. Alice calculates $\sigma = D(sk, m)$ and sends $m \parallel \sigma$.
> 2. Bob receives it and calculates $E(pk, \sigma)$ and compares it with $m$.
> 	- The key $pk$ here is Alice's public key.

- Since the signature can be decrypted using Alice's public key, it must have been signed using Alice's private key.
	- Thus the message must have been from Alice.
- Verification is done using Alice's public key, so anyone can verify the message.
- Messages are usually long, so we take a hash function $H$ to shorten it, and sign $H(m)$ instead.
