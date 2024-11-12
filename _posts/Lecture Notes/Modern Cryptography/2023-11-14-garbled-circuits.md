---
share: true
toc: true
math: true
categories:
  - Lecture Notes
  - Modern Cryptography
tags:
  - lecture-note
  - cryptography
  - security
title: 15. Garbled Circuits
date: 2023-11-14
github_title: 2023-11-14-garbled-circuits
---

A simple solution for two party computation would be to use oblivious transfers as noted [here](./2023-11-09-secure-mpc.md#ot-for-computing-14.-secure-multiparty-computation#ot-for-computing-$2$-ary-function-with-finite-domain$-ary-function-with-finite-domain). However, this method is inefficient. We will look at **Yao's protocol**, presented in 1986, for secure two-party computation.

The term **garbled circuit** was used by Beaver-Micali-Rogaway (BMR), presenting a multiparty protocol using a similar approach to Yao's protocol.

## Yao's Protocol

This protocol is for **general secure two party computation**. By general, it means that the protocol can securely compute any functionality. The protocol works on boolean circuits using AND/OR gates, which can be extended to arbitrary circuits, such as addition, multiplication, etc. This protocol takes **constant number of rounds**, and is secure for semi-honest parties.

A plain circuit would be evaluated by giving raw values $0/1$ to the input wires. These inputs will be evaluated through the gates, and the output is fed to another gate, and so on. But for *secure computation*, we require that **no party learns the values of any internal wires**.

**Yao's protocol** is a compiler which transforms a circuit so that all information is hidden except for the final output.

## Garbled Circuits

A **garbled circuit** is an *encrypted circuit*, with a pair of keys for each wire. For each gate, a key is given for each of the input wires. Using the keys, it is possible to compute the key of the gate output, but nothing else can be learned. For this process, we will use **oblivious transfer**.

### Constructing a Garbled Circuit

The garbler first encrypts the circuit. First, assign two keys, called **garbled values**, to each wire of the circuit.

Suppose we have an AND gate, where $C = \rm{AND}(A, B)$. For the wire $A$, the garbler assigns $A_0, A_1$, each for representing the bit $0$ and $1$. Note that this mapping is known only to the garbler. Similar process is done for wires $B$ and $C$.

Then we have the following garbled values, as in columns 1 to 3. Now, encrypt the values of $C$ with a semantically secure scheme $E$, and obtain the $4$th column. Then, permute the rows in random order so that it is indistinguishable.

|$A$|$B$|$C$|$C = \rm{AND}(A, B)$|
|:-:|:-:|:-:|:-:|
|$A_0$|$B_0$|$C_0$|$E(A_0 \parallel B_0, C_0)$|
|$A_0$|$B_1$|$C_0$|$E(A_0 \parallel B_1, C_0)$|
|$A_1$|$B_0$|$C_0$|$E(A_1 \parallel B_0, C_0)$|
|$A_1$|$B_1$|$C_1$|$E(A_1 \parallel B_1, C_1)$|

For evaluation, the **last column** will be given to the other party as the representation of the **garbled gate**. The inputs will be given as $A_x$ and $B_y$, but the evaluator will have no idea about the actual value of $x$ and $y$, hiding the actual input value. Although he doesn't know the underlying bit values, the evaluator is able to compute $C_z$ where $z = x \land y$. Similarly, the evaluator will not know whether $z$ is $0$ or $1$, hiding the output or intermediate values.

The above *garbling* process is done for all gates. For the last output gate, the garbler keeps a **output translation table** to himself, that maps $0$ to $C_0$ and $1$ to $C_1$. This is used for recovering the bit, when the evaluation is done and the evaluator sends the final garbled value.

> In summary, given a boolean circuit,
> 1. Assign garbled values to all wires in the circuit.
> 2. Construct garbled gates using the garbled values.

Note that the evaluator learns nothing during the evaluation.

### Evaluating a Garbled Circuit

There is a slight problem here. In some encryption schemes, a ciphertext can be decrypted by an incorrect key. If the above encryptions are in arbitrary order, how does the evaluator know if he decrypted the correct one?

One method is to add **redundant zeros** to the $C_k$. Then the last column would contain $E\big( A_i \pll B_j, C_k \pll 0^n \big)$. Then when the evaluator decrypts these ciphertexts, the probability of getting redundant zeros with an incorrect key would be negligible. But with this method, all four ciphertexts have to be decrypted in the worst case.

Another method is adding a bit to signal which ciphertext to decrypt. This method is called **point-and-permute**. The garbler chooses a random bit $b_A$ for each wire $A$. Then when drawing $A_0, A_1$, set the first bit (MSB) to $b_A$ and $1 - b_A$, respectively. Next, the ciphertexts are sorted in the order of $b_A$ and $b_B$. Then the evaluator can exploit this information during evaluation.

For example, if the evaluator has $X$ and $Y$ such that $\rm{MSB}(X) = 0$ and $\rm{MSB}(Y) = 1$, then choose the second ($01$ in binary) ciphertext entry to decrypt.

This method does not reduce security, since the bits $b_A$, $b_B$ are random. Also, now the evaluator doesn't have to decrypt all four ciphertexts, reducing the evaluation load.

## Protocol Description

> Suppose we have garbler Alice and evaluator Bob.
> 
> 1. Alice garbles the circuit, generating garbled values and gates.
> 2. Garbled gate tables and the garbled values of Alice's inputs are sent to Bob.
> 3. For Bob's input wire $B$, Alice and Bob run an 1-out-of-2 OT protocol.
> 	- Alice provides $B_0$ and $B_1$ to the OT.
> 	- Bob inputs his input bit $b$ to the OT, and Bob now has $B_b$.
> 4. Bob has garbled values for all input wires, so evaluates the circuit.
> 5. Bob sends the final garbled output to Alice.
> 6. Alices uses the output translation table to recover the final result bit.

Note that OT can be done in *parallel*, reducing the round complexity.

### Why is OT Necessary?

Suppose Alice gave both $B_0$ and $B_1$ to Bob. Bob doesn't know which one represents $0$ or $1$, but he can just run the evaluation for both inputs.

Suppose we have a $2$-input AND gate $C = \rm{AND}(A, B)$. Bob already has $A_x$ from Alice, so he evaluates for both $B_0$ and $B_1$, obtaining $C_{x\land 0}$ and $C_{x \land 1}$. If these are the same, Bob learns that $x = 0$. If different, $x = 1$.

So we need an OT to make sure that Bob only learns one of the garbled values.

### Performance

- We need about $2$ to $4$ rounds.
	- Depends on the implementation of the OT.
	- Need additional rounds if the final output should be sent to a party.
	- Anyways, takes constant number of rounds.
- Need $m$ oblivious transfers, where $m$ is the number of inputs of Bob.
	- These can be carried out in parallel.
- Suppose that there are $N$ gates.[^1]
	- $8N$ symmetric encryptions are required to build a garbled circuit.
	- $2N$ decryptions are required to compute the circuit.
	- We need to communicate the data of $\mc{O}(N)$ gates.

## Summary of Yao's Protocol

Let $f$ be a given public function that Alice and Bob want to compute, in circuit representation. Let $(x_1, \dots, x_n)$ and $(y_1, \dots, y_m)$ be inputs provided by Alice and Bob, respectively.

Alice generates a garbled circuit $G(f)$ by assigning garbled values for each wire. Then gives Bob $G(f)$ and the garbled values of her inputs. Then Alice and Bob run several OTs in parallel for the garbled values of Bob's inputs.

Bob computes $G(f)$ and obtains a key of $f(x_1, \dots, x_n, y_1, \dots, y_m)$, which is sent to Alice and Alice recovers the final result.

## Proof of Security (Semi-honest)

We show that if the underlying OT is secure, then Yao's protocol is secure. If both parties are honest or corrupted, there is nothing to show, so we only show for the cases where one party is corrupted.

### Alice is Corrupted

Alice's view only consists of the messages it receives during the oblivious transfers. Since the OT is secure, OT will have its own simulator $\mc{S}$ for the sender of the OT. To simulate Alice, we can use the same simulator $\mc{S}$.

In the OT-hybrid model, we assume an ideal OT. In this case, Alice receives no messages during the oblivious transfers. Then to simulate Alice, an empty transcript will be sufficient.

### Bob is Corrupted

This case is harder to show. The simulator must construct a fake garbled circuit that is indistinguishable to the real one. But the simulator doesn't know the inputs of Alice, so it cannot generate a real circuit.

Bob's view contains his inputs $(y_1, \dots, y_m)$ and the final output $z = (z_1, \dots, z_k)$. Thus, the simulator generates a fake garbled circuit that **always** outputs $z$. To do this, the garbled values for the wires can be chosen randomly, and use them for encryption keys. But the encrypted message is fixed to the (intermediate) output. For instance, make the gate table consists of $E\big( A_i \pll B_j, C_0 \big)$ for fixed $C_0$. In this way, the simulator can control the values of output wires and get $z$ for the final output.

The output translation tables can be generated using this method. An entry of the table would be $(z_i, C_0)$ where $C_0$ is the garbled value used for generating the gate table. As for $1-z_i$, any random garbled value can be used.

Lastly for communicating garbled values, Alice's input wires can be set to any two garbled values of the wire. Bob's input wires should be simulated by the simulator of the OT, which will result in any one of the two values on the wire.

## The BMR Protocol

This is a multiparty variant of Yao's protocol.

For each wire of the circuit, two random *super-seeds* (garbled values) are used. Each party generates a seed, and the super-seed of the wire is the concatenation of all seeds generated by the parties.

For example, for input wire $A$, let

$$
A_0 = a_0^1 \pll \cdots \pll a_0^n, \quad A_1 = a_1^1 \pll \cdots \pll a_1^n,
$$

where $a_0^k, a_1^k$ are seeds generated by party $P_k$.

Then for garbling gates, the super-seeds of the output wire is encrypted by the super-seeds of the input wires. As an example, suppose that we use $A_b = a_b^1 \pll \cdots \pll a_b^n$ to encrypt an output value $B$. Then we could use a secure PRG $G$ and set

$$
B \oplus G(a_b^1) \oplus \cdots \oplus G(a_b^n)
$$

as the garbled value.

[^1]: Why???
