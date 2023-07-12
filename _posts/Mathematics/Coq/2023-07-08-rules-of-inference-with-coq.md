---
share: true
toc: true
math: true
categories: [Mathematics, Coq]
tags: [math, coq, proof-verification]
title: "Rules of Inference with Coq"
date: "2023-07-08"
github_title: "2023-07-08-rules-of-inference-with-coq"
---

This is a list of proofs with Coq, for each rule of inference stated in [List of Rules of Inference (Wikipedia)](https://en.wikipedia.org/wiki/List_of_rules_of_inference)

Some rules that need [the law of excluded middle](https://en.wikipedia.org/wiki/Law_of_excluded_middle) are at the end of the section.

## Rules for Negation

```coq
Lemma reductio_ad_absurdum : forall P Q : Prop,
  (P -> Q) -> (P -> ~Q) -> ~P.
Proof.
  intros. intros HP.
  apply H0 in HP as HNQ.
  apply H in HP as HQ.
  contradiction.
Qed.

Lemma ex_contradictione_quodlibet : forall P Q : Prop,
  P -> ~P -> Q.
Proof. intros. contradiction. Qed.

Lemma double_negation_introduction : forall P : Prop,
  P -> ~~P.
Proof. auto. Qed.
```

## Rules for Conditionals

```coq
Lemma modus_ponens : forall P Q : Prop,
  (P -> Q) -> P -> Q.
Proof. auto. Qed.

Lemma modus_tollens : forall P Q : Prop,
  (P -> Q) -> ~Q -> ~P.
Proof. auto. Qed.
```

## Rules for Conjunctions

```coq
Lemma conjuction_introduction : forall P Q : Prop,
  P -> Q -> P /\ Q.
Proof. auto. Qed.

Lemma conjunction_elimination_left : forall P Q : Prop,
  P /\ Q -> P.
Proof. intros P Q [HP HQ]; auto. Qed.

Lemma conjunction_elimination_right : forall P Q : Prop,
  P /\ Q -> Q.
Proof. intros P Q [HP HQ]; auto. Qed.

Lemma conjunction_commutative : forall P Q : Prop,
  P /\ Q -> Q /\ P.
Proof.
  intros. destruct H; split; auto.
Qed.

Lemma conjunction_associative : forall P Q R : Prop,
  (P /\ Q) /\ R -> P /\ (Q /\ R).
Proof.
  intros. destruct H as [H H1]; destruct H; split; auto.
Qed.
```

## Rules for Disjunctions

```coq
Lemma disjunction_introduction_left : forall P Q : Prop,
  P -> P \/ Q.
Proof. auto. Qed.

Lemma disjunction_introduction_right : forall P Q : Prop,
  Q -> P \/ Q.
Proof. auto. Qed.

Lemma disjunction_elimination : forall P Q R : Prop,
  (P -> R) -> (Q -> R) -> (P \/ Q) -> R.
Proof.
  intros. destruct H1; auto.
Qed.

Lemma disjunctive_syllogism_left : forall P Q : Prop,
  (P \/ Q) -> ~P -> Q.
Proof.
  intros. destruct H; auto. contradiction.
Qed.

Lemma disjunctive_syllogism_right : forall P Q : Prop,
  (P \/ Q) -> ~Q -> P.
Proof.
  intros. destruct H; auto. contradiction.
Qed.

Lemma constructive_dilemma : forall P Q R S : Prop,
  (P -> R) -> (Q -> S) -> (P \/ Q) -> (R \/ S).
Proof.
  intros. destruct H1; auto.
Qed.

Lemma disjunction_commutative : forall P Q : Prop,
  P \/ Q -> Q \/ P.
Proof. intros. destruct H; auto. Qed.

Lemma disunction_associative : forall P Q R : Prop,
  (P \/ Q) \/ R -> P \/ (Q \/ R).
Proof.
  intros. destruct H as [H | H]; auto.
  destruct H; auto.
Qed.
```

## Rules for Biconditionals

```coq
Lemma biconditional_introduction : forall P Q : Prop,
  (P -> Q) -> (Q -> P) -> (P <-> Q).
Proof. split; auto. Qed.

Lemma biconditional_elimination_left_mp : forall P Q : Prop,
  (P <-> Q) -> P -> Q.
Proof.
  intros. destruct H; auto.
Qed.

Lemma biconditional_elimination_right_mp : forall P Q : Prop,
  (P <-> Q) -> Q -> P.
Proof.
  intros. destruct H; auto.
Qed.

Lemma biconditional_elimination_left_mt : forall P Q : Prop,
  (P <-> Q) -> ~P -> ~Q.
Proof.
  intros. destruct H; auto.
Qed.

Lemma biconditional_elimination_right_mt : forall P Q : Prop,
  (P <-> Q) -> ~Q -> ~P.
Proof.
  intros. destruct H; auto.
Qed.

Lemma biconditional_elimination_disjunction : forall P Q : Prop,
  (P <-> Q) -> (P \/ Q) -> (P /\ Q).
Proof.
  intros. destruct H, H0; auto.
Qed.

Lemma biconditional_elimination_disjunction_not : forall P Q : Prop,
  (P <-> Q) -> (~P \/ ~Q) -> (~P /\ ~Q).
Proof.
  intros. destruct H, H0; auto.
Qed.
```

## Other Rules

```coq
Lemma exportation : forall P Q R : Prop,
  (P /\ Q) -> R <-> (P -> Q -> R).
Proof.
  split; auto.
  destruct H; auto.
Qed.

Lemma distributive_disjunction : forall P Q R : Prop,
  P \/ (Q /\ R) <-> (P \/ Q) /\ (P \/ R).
Proof.
  split; intros.
  - destruct H as [H | [H1 H2]]; split; auto.
  - destruct H as [H1 H2]; destruct H1, H2; auto.
Qed.

Lemma distributive_conjunction : forall P Q R : Prop,
  P /\ (Q \/ R) <-> (P /\ Q) \/ (P /\ R).
Proof.
  split; intros.
  - destruct H as [H [H1 | H1]]; auto.
  - destruct H as [[H1 H2] | [H1 H2]]; auto.
Qed.

Lemma material_implication_converse : forall P Q : Prop,
  (~P \/ Q) -> (P -> Q).
Proof.
  intros. destruct H; auto. contradiction.
Qed.

Lemma resolution : forall P Q R : Prop,
  (P \/ Q) -> (~P \/ R) -> (Q \/ R).
Proof.
  intros. destruct H, H0; auto. contradiction.
Qed.
```

## Rules that require Excluded Middle

We declare the law of excluded middle as an axiom.

```coq
Axiom excluded_middle : forall P : Prop, P \/ ~P.

Lemma double_negation_elimination : forall P : Prop,
  ~~P -> P.
Proof.
  intros. destruct (excluded_middle P); auto. contradiction.
Qed.

Lemma material_implication : forall P Q : Prop,
  (P -> Q) -> (~P \/ Q).
Proof.
  intros. destruct (excluded_middle P); auto.
Qed.

Lemma reductio_ad_absurdum_neg : forall P Q : Prop,
  (~P -> Q) -> (~P -> ~Q) -> P.
Proof.
  intros. destruct (excluded_middle P); auto.
  apply H in H1 as HQ.
  apply H0 in H1 as HNQ.
  contradiction.
Qed.
```

---

I was supposed to be reading the [source](https://github.com/snu-sf/promising-seq-coq) for the paper [Sequential Reasoning for Optimizing Compilers Under Weak Memory Concurrency](https://dl.acm.org/doi/abs/10.1145/3519939.3523718) but I got carried away...