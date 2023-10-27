---
share: true
toc: true
math: false
categories:
  - Lecture Notes
  - Internet Security
tags:
  - network
  - security
  - lecture-note
title: 01. Security Introduction
date: 2023-09-10
github_title: 2023-09-10-security-intro
image:
  path: /assets/img/posts/Lecture Notes/Internet Security/is-01-cryptosystem.png
attachment: 
  folder: assets/img/posts/Lecture Notes/Internet Security
---

> Every program has at least two purposes: the one for which it was written, and another for which it wasn't. - Alan J. Perlis

## Security Overview

### Security

**Security** may mean different things.
- Emotional security
- Physical security: physical separation of assets
- Resource exhaustion: mitigating DoS attacks
- **System security**
- **Network security**
- Cryptography
- Social Engineering: email pranksters (impersonating)

In this course, we are mainly interested in system/network security!

There are two categories in **IT Security**, (though the boundary is blurry)
- **Computer** (system) **security** uses automated tools and mechanisms to protect the **data in a computer**, against hackers, malware, etc.
- **Internet** (network) **security** prevents, detects, and corrects security violations that involve the **transmission of information** in a network.

In internet security, we assume that:
- Everything on the network can be an attack target.
- Every transmitted bit can be tapped (eavesdropped).

### Modeling in Network Security

- Basically, we have a sender and a receiver, and they communicate through the internet.
- **Sender and receiver want to communicate *securely***.
- But the adversary can attack the communication channel. For instance,
	- tapping, eavesdropping, snooping messages
	- inserting, modifying, deleting, replaying messages
	- poisoning data
	- impersonate and pretend to be someone else
- Conventionally, we use the following names:
	- Alice and Bob for the two parties participating in the communication.
	- Eve (or Mallory, Oscar) for the adversary.

## Security Attacks

This is only an overview, so the attacks are introduced briefly.

### Computer/Network Attacks

- Malware: malicious software
	- virus, worm, Trojan, spyware, ransomware
- Bots that automate malicious tasks
- [Buffer Overflow](https://en.wikipedia.org/wiki/Buffer_overflow) (BOF)
- Denial of Service (DoS)
	- Distributed DoS (DDoS) if numerous hosts are used
- Network-based attacks (upcoming)
- Physical Attacks
	- [Van Eck phreaking](https://en.wikipedia.org/wiki/Van_Eck_phreaking)
	- Energy weapons (electromagnetic waves)
- Password Attacks
	- Password guessing, dictionary attacks, brute force attacks
- Information gathering attacks
	- through phone, web, SNS (watch out what you post)
	- Phishing with cloned websites
		- the information you enter will be sent to the attacker
	- (Port) Scanning: searching for open ports on a server
- [Side Channel Attacks](https://en.wikipedia.org/wiki/Side-channel_attack): attacks based on extra information rather than the flaws in the design of the protocol or algorithm itself.
	- Timing information, power consumption can be used
	- Data remanence: reading sensitive data after they have been deleted

### Network-based Attacks

- Cryptographic attacks: decrypting ciphertext, finding the key
- Spoofing: ARP, DNS, cache poisoning
- Session hijacking
- Impersonation, man-in-the-middle (MITM) attacks
- Network domain specific attacks: wireless, web, mobile, IoT etc.

There are two types of attacks in security attacks
- **Active attacks**: modify the content of messages
	- Ex. (D)DoS, MITM, poisoning, smurf attack, system attacks.
	- *Prevention* is important since the active attacks concern *data integrity* and *availability*.
- **Passive attacks**: does not modify information, but observes the content or copies it.
	- Ex. eavesdropping, port scanning (idle scan secretly scans).
	- *Detection* is important since passive attacks are a danger to *confidentiality*.

## Security Services and Mechanisms

### CIA Triad

What kind of security services do we want? The basic network security services must support the following. These are also known as the **CIA triad**.

- **Confidentiality**: the data must be kept secret (privacy)
- **Integrity**: the data must not be modified during transmission (consistency, accuracy, trustworthiness)
- **Availability**: information should be consistently and readily accessible

Additionally, we also need:
- **Authentication**: a way to authenticate users (ID, passwords)
- **Non-repudiation**: ensure that no party can deny that it sent or received a message or approved some information
	- Assurance that someone cannot deny the validity of message or information

### Attacks Against CIA Triad

- Confidentiality: snooping, traffic analysis
- Integrity: modification, masquerading, replaying, repudiation
- Availability: denial of service

### More Security Services

- **Access control**: controlling privileges to access assets
	- identification, authentication (credential validation), authorization
- **Anonymity**: name or identification is hidden
- **Accountability**: any actions of an entity can be traced uniquely to that entity
	- similar to responsibility of an entity to some event or incident
- **Security audit**: assessment or evaluation of an organization's security systems
- **Privacy**: keeping data safe in transit and in storage
- **Digital forensics**: recovering data from digital devices

### Security Mechanisms

There are many ways of achieving security.

- **Cryptography**: encryption/decryption of data
- **Credential**: ID, password, certificates
- **Message digest**: usage of hash functions and message authentication codes (MAC)
- **Traffic padding**: to keep traffic size equal
	- It may be desirable to not leak *any* information, so one might add padding to the traffic, so the traffic is indistinguishable by the adversary (prevents side-channel attacks)
- **Digital signatures**: provides authenticity of digital messages or documents
- **Trusted Third Party** (TTP): a safe third-party that we can trust
	- If we have a TTP, a lot of problems go away. We can always ask the TTP for the truth.
	- But TTP can become a *single point of failure* (SPOF), and security architectures may become too dependent on the TTP.
- **Append-only server**: keeps track of all modifications, good for auditing
	- Blockchain is a kind of append-only data structure.

## Cryptography

> **Cryptography** is the study of mathematical techniques for securing digital information, systems, and distributed computations against adversarial attacks.[^1]

**Cryptanalysis** is the study of methods for obtaining the meaning of encrypted information without access to the key.

### Basics of a Cryptosystem

![is-01-cryptosystem.png](../../../assets/img/posts/Lecture%20Notes/Internet%20Security/is-01-cryptosystem.png#)

- A **message** in *plaintext* is given to an **encryption algorithm**.
- The encryption algorithm uses an **encryption key** to create a *ciphertext*.
- The ciphertext if given to a **decryption algorithm**.
- The decryption algorithm uses a **decryption key** to recover the original plaintext.
- The encryption/decryption keys are only known to the sender/receiver.

### Classification of Cryptosystems

There are two criteria for classifying cryptosystems.

- How are the keys used?
	- **Symmetric** cryptography uses a single key for both encryption and decryption.
	- **Public key** cryptography uses different keys for encryption and decryption, respectively.
- How are plaintexts processed?
	- **Block cipher**
	- **Stream cipher**

### Kerckhoffs' Principle

There are two choices to achieve the security of a cryptosystem.

1. Keep the encryption/decryption scheme secret. (security through obscurity)
2. Keep the key secret.

But in real life, we use the second method and keep the key secret.

> The cipher method must not be required to be secret, and it must be able to fall into the hands of the enemy without inconvenience.[^1]

**Kerckhoffs' principle** demands that *security rely solely on the secrecy of the key*. Even if everything about the system is publicly known, except for the key.

Why? Here are some of the arguments in favor of Kerckhoffs' principle.

1. It is significantly easier to maintain the secrecy of a short key than to keep an encryption scheme secret.
	- Information about the scheme might be leaked or reverse engineered.
2. In case the secret information is exposed, it is much easier to replace the key, than to replace the encryption scheme.
	- Generating a new random key is relatively easy, but generating a new, *secure* encryption scheme is non-trivial.
3. The public can review an encryption scheme to check for vulnerabilities.
	- *Standardization* of schemes is possible, supporting compatibility between different users.
	- It is beneficial to use strong schemes that have gone through public scrutiny.

## Threat Modeling

What should we consider when we are designing secure systems? We should consider what attacks are possible. **Threat modeling** is the process of systematically identifying the threats faced by a system.

1. Identify the values of assets.
2. Enumerate the *attack surfaces*.
3. Hypothesize attackers.
	- What kinds of assets would they want?
	- Are they able to attack through vulnerable surfaces?
4. Survey mitigations.
5. Balance costs vs. risks.

We consider the case of a smartphone.

### Identifying Assets

In a smartphone, assets (things of value) would be
- Saved credentials such as passwords
- *Personally identifiable information* (PII) such as social security number
- Contacts, pictures, sensitive documents, credit card data
- Access to sensors such as camera, microphone, network traffic or location
- The device itself

### Attack Surfaces

- Physically stealing the device
- Tricking the user to install malicious applications
- Passive eavesdropping on the network
- Backdoors in the OS

### Hypothetical Attackers

For example,

|Attacker|Abilities|Goals|
|:-:|-|-|
|Thief|Steal the phone|Take the device|
|FBI|Lot of things...|Obtain evidence from the device|
|Eavesdropper|Observe network traffic|Steal information|

### Surveying Mitigations

Next, we survey how to mitigate the attacks.

Suppose we are mitigating theft. One could:
- Apply strong authentication using passwords or biometrics
	- But this is annoying to the user
- Use full device encryption
- Use remote device tracking and format the device
	- May not work if the device is disconnected from the internet

For blocking eavesdroppers, one could apply HTTPS everywhere or use a VPN. But it's hard to check if apps are actually using HTTPS or not, and VPNs may slow down connection.

### Cost vs. Risk Analysis

- How costly is the mitigation?
	- Applying strong password is not very costly.
- How likely is the attack?
	- Attacks from FBI are very unlikely for an average person.

[^1]: J. Katz, Introduction to Modern Cryptography
