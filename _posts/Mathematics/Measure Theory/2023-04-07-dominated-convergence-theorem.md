---
share: true
toc: true
math: true
categories: [Mathematics, Measure Theory]
tags: [math, analysis, measure-theory]
title: "07. Dominated Convergence Theorem"
date: "2023-04-07"
github_title: "2023-04-07-dominated-convergence-theorem"
image:
  path: /assets/img/posts/Mathematics/Measure Theory/mt-07.png
attachment:
  folder: assets/img/posts/Mathematics/Measure Theory
---

## Almost Everywhere

지난 글에서 measure가 0인 집합 위에서 적분하면 결과가 0이 됨을 확인했습니다. 적분 입장에서 보면 measure가 0인 곳에서의 적분은 의미가 없다고 생각할 수 있겠죠? 그러면 앞으로 그런걸 무시해도 된다고 하면 어떨까요?

**정의.** (Almost Everywhere) $P = P(x)$ 가 어떤 성질이라 하자.[^1] 만약 measure가 0인 집합 $N$이 존재하여 성질 $P$가 모든 $x \in E \setminus N$ 에서 성립하면, $P$가 $E$의 **거의 모든 점에서** 성립한다고 한다.

**표기법.** 위를 편의상 ‘$P$ $\mu$-a.e. (**almost everywhere**) on $E$’로 적겠습니다.

### Markov's Inequality

확률론과도 연관이 깊은 정리 하나를 소개하고 가겠습니다.

**정리.** (Markov's Inequality) $u \in \mathcal{L}^{1}(E, \mu)$ 라 하자. 모든 $c > 0$ 에 대하여

$$\mu\left( \lbrace \lvert u \rvert \geq c\rbrace \cap E \right) \leq \frac{1}{c} \int_E \lvert u \rvert \,d{\mu}$$

이다.

**증명.** $\displaystyle\int_E \lvert u \rvert \,d{\mu} \geq \int_ {E\cap \lbrace \lvert u \rvert\geq c\rbrace} \lvert u \rvert \,d{\mu} \geq \int_ {E\cap \lbrace \lvert u \rvert\geq c\rbrace} c \,d{\mu} = c \mu\left( \lbrace \lvert u \rvert \geq c\rbrace \cap E \right)$.

아래 정리는 measure가 0인 집합에서의 적분은 무시해도 됨을 알려줍니다. $u(x) \neq 0$ 인 점들이 존재하더라도, 이 점들의 집합의 measure가 0이면 적분값에 영향을 줄 수 없습니다.

**정리.** $u\in \mathcal{L}^{1}(E, \mu)$ 일 때, 다음은 동치이다.

1. $\displaystyle\int_E \lvert u \rvert \,d{\mu} = 0$.

2. $u = 0$ $\mu$-a.e. on $E$.

3. $\mu\left( \lbrace x \in E : u(x) \neq 0\rbrace \right) = 0$.

**증명.**

(2 $\iff$ 3) $E\cap\lbrace u\neq 0\rbrace$ 가 measurable이므로 정의에 의해 당연하다.

(2 $\implies$ 1) $\displaystyle\int_E \lvert u \rvert \,d{\mu} = \int_ {E \cap \lbrace \lvert u \rvert > 0\rbrace} \lvert u \rvert \,d{\mu} + \int_ {E \cap \lbrace \lvert u \rvert = 0\rbrace} \lvert u \rvert \,d{\mu} = 0 + 0 = 0$.

(1 $\implies$ 3) Markov's inequality를 사용하면

$$\mu\left( \left\lbrace \lvert u \rvert \geq \frac{1}{n}\right\rbrace \cap E \right) \leq n\int_E \lvert u \rvert \,d{\mu} = 0$$

이다. 이제 $n\rightarrow\infty$ 일 때 continuity of measure를 사용하면 $\mu\left( \lbrace \lvert u \rvert > 0\rbrace \cap E \right) = 0$ 이다.

위 정리의 결과를 생각해 보면 다음이 성립함도 알 수 있습니다.

**참고.** $A, B$가 measurable이라 하자. $B \subseteq A$ 이고 $\mu\left( A \setminus B \right) = 0$ 이면 모든 $f \in \mathcal{L}^{1}(A, \mu)$ 에 대하여

$$\int_A f \,d{\mu} = \int_B f \,d{\mu}$$

이다.

또한 적분값이 유한하다면, 거의 모든 점에서 함숫값이 유한해야 할 것입니다!

**정리.** $u \in \mathcal{L}^{1}(E, \mu)$ 이면 $u(x) \in \mathbb{R}$ $\mu$-a.e. on $E$ 이다. 즉, $u(x) = \infty$ 인 집합의 measure가 0이다.

**증명.** $\mu\left( \lbrace \lvert u \rvert \geq 1\rbrace\cap E \right) \leq \displaystyle\int_E \lvert u \rvert \,d{\mu} < \infty$.[^2] 그러므로

$$\begin{aligned}        \mu\left( \lbrace \lvert u \rvert = \infty\rbrace \cap E \right) & = \mu\left( \bigcap_ {n=1}^\infty \lbrace x \in E : \lvert u(x) \rvert \geq n\rbrace \right) \\                                               & = \lim_ {n \rightarrow\infty} \mu\left( \lbrace \lvert u \rvert \geq n\rbrace \cap E \right) \leq \limsup_ {n\rightarrow\infty} \frac{1}{n} \int_E \lvert u \rvert \,d{\mu} = 0    \end{aligned}$$

이다.

적분 가능하다면 어차피 함숫값이 무한한 영역은 적분값에 영향을 주지 않으므로, 함숫값이 유한한 곳에서만 적분해도 될 것입니다.

**따름정리.** $u \in \mathcal{L}^{1}(E, \mu)$ 이면 $\displaystyle\int_E u \,d{\mu} = \int_ {E \cap \lbrace \lvert u \rvert < \infty\rbrace} u \,d{\mu}$ 이다.

### Linearity of the Lebesgue Integral

드디어 일반적인 경우에서 적분의 선형성을 증명합니다!

**정리.** $f_1, f_2 \in \mathcal{L}^{1}(E, \mu)$ 이면 $f_1 + f_2 \in \mathcal{L}^{1}(E, \mu)$ 이고

$$\int_E \left( f_1 + f_2 \right) \,d{\mu} = \int_E f_1 \,d{\mu} + \int_E f_2 \,d{\mu}$$

이다.

**증명.** $\lvert f_1 + f_2 \rvert \leq \lvert f_1 \rvert + \lvert f_2 \rvert$ 임을 이용하면 $f_1+f_2 \in \mathcal{L}^{1}(E, \mu)$ 인 것은 당연하다. 이제 $f = f_1 + f_2$ 로 두고

$$N = \left\lbrace x : \max\left\lbrace f_1^+, f_1^-, f_2^+, f_2^-, f^+, f^-\right\rbrace = \infty \right\rbrace$$

으로 정의하자. 함수들이 모두 적분 가능하므로 위 정리에 의해 $\mu(N) = 0$ 이다. 그러므로 $E \setminus N$ 에서는 무한한 값이 없으므로 이항을 편하게 할 수 있다. 즉,

$$f^+ - f^- = f_1^+ - f_1^- + f_2^+ - f_2^- \implies f^+ + f_1^- + f_2^- = f^- + f_1^+ + f_2^+$$

이다. 그러면

$$\int_ {E\setminus N} f^+ \,d{\mu} + \int_ {E\setminus N} f_1^- \,d{\mu} + \int_ {E\setminus N} f_2^- \,d{\mu} = \int_ {E\setminus N} f^-\,d{\mu} + \int_ {E\setminus N} f_1^+\,d{\mu} + \int_ {E\setminus N} f_2^+ \,d{\mu}$$

이고, $\mu(N) = 0$ 임을 이용하여 $N$ 위에서의 적분값을 더해주면

$$\int_ {E \setminus N} f \,d{\mu} = \int_ {E \setminus N} f_1 \,d{\mu} + \int_ {E \setminus N} f_2 \,d{\mu} \implies \int_ {E} f \,d{\mu} = \int_ {E} f_1 \,d{\mu} + \int_ {E} f_2 \,d{\mu}$$

를 얻는다.

## Convergence Theorems Revisited

이제 이를 응용하여 수렴정리를 다시 적어보겠습니다. 지난 글에서는 모든 점에서 특정 성질이 성립할 것이 요구되었으나 이제는 거의 모든 점에서만 성립하면 됩니다. 증명은 해당 성질이 성립하지 않는 집합을 빼고 증명하면 됩니다.

**정리.** (단조 수렴 정리) $f_n$이 measurable이고 $0 \leq f_n(x) \leq f_ {n+1}(x)$ $\mu$-a.e. 라 하자.

$$\lim_ {n\rightarrow\infty} f_n(x) = f(x)$$

로 두면,

$$\lim_ {n \rightarrow\infty} \int_E f_n \,d{\mu} = \int_E f \,d{\mu}.$$

이다.

**정리.** (Fatou) $f_n$이 measurable이고 $f_n(x) \geq 0$ $\mu$-a.e. 라 하자. 다음이 성립한다.

$$\int_E \liminf_ {n\rightarrow\infty} f_n \,d{\mu} \leq \liminf_ {n\rightarrow\infty} \int_E f_n \,d{\mu}.$$

비슷한 느낌으로 다음과 같은 명제를 생각할 수도 있습니다.

**명제.** $E$ 위의 measurable function $f, g$에 대하여 $\lvert f \rvert \leq \lvert g \rvert$ $\mu$-a.e. on $E$ 이면,

$$\int \lvert f \rvert \,d{\mu} \leq \int \lvert g \rvert \,d{\mu}$$

이므로, $g \in \mathcal{L}^{1}(E, \mu)$ 이면 $f \in \mathcal{L}^{1}(E, \mu)$ 이다.

## Equivalence Classes of Functions in $\mathcal{L}^p$

사실 $\mu$-a.e. 를 정의한 이유가 또 있습니다. 이는 뒤에서 $\mathcal{L}^p$ 공간을 공부하며 명확해질 것입니다. 두 함수 간의 거리를 정의한다면, 함수의 차를 적분한 값을 떠올릴 수 있을 것입니다. 그런데 거리 함수(metric) 정의상, 거리가 0이려면 거리를 잰 두 대상이 같아야 합니다! 그런데 르벡 적분의 경우 실제로 함수가 같지 않지만 거의 모든 점에서 함숫값이 일치하여 차의 적분값이 0이 되어버릴 수도 있습니다.

**정의.** Measurable set $E$에 대하여, $\mathcal{L}^{1}(E, \mu)$의 함수에 대한 relation $\sim$을 다음과 같이 정의한다.

> $f \sim g \iff f = g$ $\mu$-a.e. on $E$.

그러면 $\sim$은 equivalence relation이고 다음과 같이 적을 수 있다.

$$[f] = \lbrace g \in \mathcal{L}^{1}(E, \mu) : f \sim g\rbrace.$$

이처럼 equivalence relation을 정의하면 equivalence class의 대표에 대해서만 생각해도 충분합니다. 사실상 거의 모든 점에서 함숫값이 같다면 같은 함수로 보겠다는 뜻이 됩니다.

## Dominated Convergence Theorem

마지막 수렴정리를 소개하고 수렴정리와 관련된 내용을 마칩니다. 지배 수렴 정리(dominated convergence theorem, DCT)로 불립니다.

![mt-07.png](../../../assets/img/posts/Mathematics/Measure%20Theory/mt-07.png)

**정리.** (지배 수렴 정리) Measurable set $E$와 measurable function $f$에 대하여, $\lbrace f_n\rbrace$이 measurable function의 함수열이라 하자. $E$의 거의 모든 점 위에서 극한 $f(x) = \displaystyle\lim_ {n \rightarrow\infty} f_n(x)$ 가 $\overline{\mathbb{R}}$에 존재하고 (점별 수렴) $\lvert f_n \rvert \leq g \quad \mu$-a.e. on $E$ ($\forall n \geq 1$) 를 만족하는 $g \in \mathcal{L}^{1}(E, \mu)$ 가 존재하면,

$$\lim_ {n \rightarrow\infty} \int_E \lvert f_n - f \rvert \,d{\mu} = 0$$

이다.

**참고.**

1. $f_n, f \in \mathcal{L}^{1}(E, \mu)$ 이다.

2. 적분의 성질에 의해

	$$\lvert \int f_n \,d{\mu} - \int f \,d{\mu} \rvert \leq \int \lvert f_n - f \rvert \,d{\mu}$$

	이므로 위 정리의 결론은 곧

	$$\lim_ {n \rightarrow\infty} \int f_n \,d{\mu} = \int f \,d{\mu}$$

	를 의미한다.

**증명.** 다음과 같은 집합을 정의한다.

$$A = \left\lbrace \displaystyle x \in E : \lim_ {n \rightarrow\infty} f_n(x) \text{가 존재하고}, f_n(x), f(x), g(x) \in \mathbb{R}, \lvert f_n(x) \rvert \leq g(x)\right\rbrace.$$

그러면 가정에 의해 $\mu\left( E\setminus A \right) = 0$ 이다. 이제 $x \in A$ 에 대해서만 생각해도 충분하다. 그러면

$$2g - \lvert f_n - f \rvert \geq 2g - \bigl(\lvert f_n \rvert + \lvert f \rvert \bigr) \geq 0$$

이다. $\lvert f_n - f \rvert \rightarrow 0$, $2g - \lvert f_n - f \rvert \rightarrow 2g$ 이므로, Fatou’s lemma를 적용하면

$$\begin{aligned}        2 \int_E g \,d{\mu} = \int_A 2g \,d{\mu} & = \int_A \liminf_ {n \rightarrow\infty} \big(2g - \lvert f_n - f \rvert\big) \,d{\mu} \\                                               & \leq \liminf_ {n \rightarrow\infty} \left( 2 \int_A g \,d{\mu} - \int_A \lvert f_n - f \rvert \,d{\mu} \right) \\                                               & = 2\int_A g \,d{\mu} - \limsup_ {n \rightarrow\infty} \int_A \lvert f_n - f \rvert \,d{\mu} \leq 2 \int_A g \,d{\mu}    \end{aligned}$$

이다. 따라서

$$2 \int_A g \,d{\mu} - \limsup_ {n \rightarrow\infty} \int_A \lvert f_n - f \rvert \,d{\mu} = 2 \int_A g \,d{\mu}$$

이고, 가정에 의해 $\displaystyle 0 \leq \int_A g \,d{\mu} < \infty$ 이므로 $\displaystyle\limsup_ {n \rightarrow\infty} \int_A \lvert f_n - f \rvert \,d{\mu} = 0$ 이다.

[^1]: 예를 들어, ‘$f(x)$가 연속이다’ 등.
[^2]: Continuity of measure를 사용하기 위해서는 첫 번째 집합의 measure가 유한해야 한다.
