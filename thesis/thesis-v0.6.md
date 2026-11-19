---
title: 'From Correctness to the Quantum Oracle'
subtitle: 'Exact Modular Rounding, Structured Cryptanalysis, and Conditional Security of C1-K4'
author: 'Samir Bouftass'
date: 'Research manuscript — 18 November 2026 — Version 0.6'
lang: en-US
documentclass: report
classoption: [oneside]
fontsize: 11pt
papersize: a4
geometry:
  - top=29mm
  - bottom=28mm
  - left=30mm
  - right=28mm
colorlinks: true
linkcolor: MidnightBlue
urlcolor: MidnightBlue
toc: true
toc-depth: 2
numbersections: true
header-includes: |
  \usepackage{amsmath,amssymb,amsthm,mathtools}
  \usepackage{microtype}
  \usepackage{booktabs}
  \usepackage{enumitem}
  \usepackage{fancyhdr}
  \usepackage{float}
  \definecolor{MidnightBlue}{RGB}{20,51,87}
  \fancypagestyle{plain}{\fancyhf{}\fancyfoot[C]{\thepage}\renewcommand{\headrulewidth}{0pt}}
  \pagestyle{fancy}
  \fancyhf{}
  \fancyhead[L]{\small From Correctness to the Quantum Oracle}
  \fancyhead[R]{\small Thesis II · Version 0.6}
  \fancyfoot[C]{\thepage}
  \setlength{\headheight}{14pt}
  \setlist{itemsep=0.2em,topsep=0.35em}
  \emergencystretch=2em
  \newtheorem{thm}{Theorem}[chapter]
  \newtheorem{lemma}[thm]{Lemma}
  \newtheorem{prop}[thm]{Proposition}
  \newtheorem{definition}[thm]{Definition}
---

**Research manuscript, Chapters 1–12.** Samir Bouftass is responsible for the scientific claims and publication decisions. ChatGPT (OpenAI) assisted with drafting, computations and consistency checks.

# Abstract {-}

Modular reduction followed by integer division is a simple operation with a complicated probabilistic afterlife. Once quotient values are published, a cryptographic proof must account for the remainders that were discarded, the carries that connect different computations, and the algebraic structure of the matrix operators. When rounded encryption can fail with small but nonzero probability, a second question appears: which definitions of correctness and failure finding are sufficient for a key-encapsulation mechanism, in the classical random-oracle model and in the quantum random-oracle model?

This thesis studies these questions through one explicitly frozen research construction, C1-K4-P12-CBD1, over the negacyclic ring $R=\mathbb Z_{4096}[x]/(x^{256}+1)$ with module rank four and independent centered-binomial secret and error coefficients. It develops the quotient–remainder calculus needed to describe its observations, states the complete public-key encryption algorithms, and organizes the argument from an exact decryption-error identity to conditional security statements. In the current companion manuscripts, the averaged worst-message decryption-failure parameter satisfies $2^{-8192}\le\delta_{wc}<2^{-212}$. Two explicit decisional Module-LWE source distributions are required for the passive-security reduction; replacing them with an unstructured LWE assumption without justification would change the claim.

Concrete cryptanalysis is treated as evidence with a defined scope, not as a hardness proof. The documented MATZOV-style and adapted rotation-aware coded-dual model evaluations report about $2^{213.657}$ and $2^{206.211}$ operations at specified search points, respectively. The thesis distinguishes those costs from source-advantage bounds, reduction overheads, and implementation costs. Chapters 8–10 examine the Fujisaki–Okamoto transformation, implicit rejection, coherent random-oracle simulations, and two types of find-failing-plaintext experiment. Chapter 9 develops a conservative conditional QROM inequality from the companion study in a single-user, key-first ideal-oracle experiment; a sharper use of the contemporary FFP-NG theory remains an open hypothesis-matching problem.

The principal methodological contribution of the thesis is a transparent chain of claims, from exact arithmetic to conditional quantum security, with every assumption, simulation, computational estimate, and remaining obligation stated separately. **No unconditional difficulty theorem, concrete-expander security proof, or 128-bit post-quantum certification is claimed.**

**Keywords:** modular rounding; quotient–remainder calculus; Module-LWE; decryption correctness; Fujisaki–Okamoto; QROM; decryption failures; structured cryptanalysis; reproducibility.

# Manuscript status and scope {-}

This edition covers the twelve chapters of the research manuscript. Chapters 2–8 give the arithmetic, the correctness theorem, the classical reductions and the attack-model estimates. Chapters 9–10 give the conditional quantum reduction and the failure-finding experiments. Chapters 11–12 collect the assumptions, evidence and open questions.

The source editions are [S1] v1.1 (18 November 2026), [S2] V4.2, [S3] V4.3 and [S4] V4.4 (13 November 2026). The design-specific contributions are the joint error and probability analysis, the exact source simulations and the application of specified models to this parameter set. Euclidean division, negacyclic multiplication and the modular FO transformations are established tools.

# Introduction

## The problem behind a small arithmetic operation

A map of the form

$$
F_D(z)=\left\lfloor\frac{[z]_P}{D}\right\rfloor,
\qquad D\mid P,
$$

is often described informally as rounding. Here $[z]_P$ is the least nonnegative representative of an integer modulo $P$. The map discards the remainder modulo $D$, retaining only the quotient. In an isolated scalar calculation its behavior is elementary. In cryptography, however, the quotient participates in subsequent products, different inputs share secret random variables, and observed outputs may reveal information about values that are no longer visible.

The loss of a remainder is therefore not equivalent to the addition of an independent error. For a deterministic input $z$, write

$$
[z]_P=D F_D(z)+[z]_D.
$$

The two terms are exactly linked. If $z$ itself depends on a secret, a public linear operator, and a small random error, then its remainder inherits those dependencies. An analyst may bound a remainder by its range, or may average it in a carefully justified conditional order; replacing it by a fresh random variable with a convenient law is a different model and needs its own proof.

A cryptographic construction introduces several distinct requirements. **Correctness** asks whether a ciphertext produced by honest encryption decrypts to the intended message. **Passive security** asks whether public information and challenge ciphertexts are distinguishable under an explicitly defined attack experiment. **Chosen-ciphertext security** asks whether the mechanism remains secure when an attacker can make permitted decapsulation queries. **Quantum security** further requires that random-oracle access and reduction algorithms be interpreted coherently, with realistic control over query budgets and reduction losses. These requirements are related, but none can be inferred from the mere smallness of a numerical noise term.

The object studied here was selected precisely because it makes the interfaces visible. Its encryption algorithm uses rounding both in the public key and in the ciphertext; it introduces a uniformly sampled remainder to lift a rounded public value; and it uses the same ephemeral secret in two ciphertext components. A valid proof must preserve these joint laws, not merely the marginal distribution of each coefficient.

## Research questions

The thesis is organized around five questions.

**RQ1 — Exact arithmetic.** Which quotient, remainder, and carry identities can be used without approximating the distributions? How does a randomized lift restore a uniform ring element when—and only when—the quotient has the required joint law with the public view?

**RQ2 — Correctness with dependent remainders.** Can a nontrivial explicit decryption-failure bound be proved while retaining the dependence of the actual remainders on the secrets and errors? Does the failure probability vanish, and at which point should the maximum over messages be taken?

**RQ3 — Conditional passive security.** Which exact source problems are sufficient for a game-based IND-CPA reduction? How do seeded matrix generation, ring structure, and the adversary's resource budget affect the statement?

**RQ4 — Cryptanalysis and evidence.** What is learned from reproducible primal, dual, dual-hybrid, and rotation-aware estimates? What cannot be concluded from a search minimum, a measured arithmetic count, or a heuristic lattice-cost model?

**RQ5 — From PKE to KEM in the QROM.** How do nonzero decryption failures enter Fujisaki–Okamoto reductions? Which conditional end-to-end theorem is actually available for the specified oracle experiment, and what additional analysis is needed for a sharper, concrete security claim?

These questions are intended to prevent a common logical error: moving directly from a correctness bound and a high modeled attack cost to a numerical claim of security. Between those endpoints lie assumptions about source hardness, the exact experiment, the implementation of oracle access, and the units by which security loss is measured.

## Mathematical setting and notation

The principal case study uses

$$
R=\mathbb Z_P[x]/(x^n+1),\qquad
(n,k,P,Q,C)=(256,4,4096,4,2).
$$

The ring elements are represented as arrays of $256$ coefficients. Multiplication is negacyclic, so $x^{256}=-1$; multiplication by a ring element corresponds to a structured signed Toeplitz-like coefficient matrix, not to an independent scalar matrix. The module rank is $k=4$.

The common small-coefficient distribution is $\chi=\mathrm{CBD}_1$, defined by the difference of two independent uniformly random bits:

$$
\Pr[E=-1]=\Pr[E=1]=\frac14,
\qquad \Pr[E=0]=\frac12.
$$

Unless otherwise stated, all coefficients drawn from $\chi$ are mutually independent across their stated coordinates, and key-generation randomness is independent of encryption randomness. For vectors and matrices, $F_D$ acts coefficientwise. The expression $a^Tb$ denotes the inner product over the ring, with the resulting polynomial interpreted coefficientwise when it enters a quantizer.

The principal source problem needed later is explicitly distributional. For $j\in\{4,5\}$, a sample from $\mathrm{DMLWE}_{4,j}^{\mathrm{CBD1}}$ has either of the following forms:

$$
(B,Bt+e) \quad\text{or}\quad (B,y),
$$

with

$$
B\leftarrow U(R^{j\times4}),\quad
 t\leftarrow\chi^{1024},\quad
 e\leftarrow\chi^{256j},\quad
 y\leftarrow U(R^j).
$$

Both possibilities expose the same uniform public matrix $B$. In the first, the same secret $t$ is shared by all $j$ rows. The exact secret and error law, modulus, module rank, and row count are part of the assumption; they are not replaceable by the parameters of a different Module-LWE construction without an additional reduction.

The notation $\operatorname{Adv}_{\mathrm{CPA}}$ and $\operatorname{Adv}_{\mathrm{CCA}}$ denotes a distinguishing-success **bias** $|\Pr[\widehat\beta=\beta]-1/2|$. By contrast, the $\epsilon_{4,j}$ terms for the source distributions are **acceptance-probability gaps**. Keeping these conventions explicit is essential when the thesis imports generic transformations whose advantages use different normalizations.

## The frozen C1-K4 construction

The reference encryption profile, named **C1-K4-P12-CBD1**, is not altered in this manuscript. It consists of the following algorithms.

**Key generation.** Sample $A\leftarrow U(R^{4\times4})$ and independent $s,e_0\leftarrow\chi^{1024}$. Compute

$$
 b=F_4(As+e_0).
$$

Publish $pk=(A,b)$ and retain $sk=s$ privately. The compact public-key encoding stores a seed for the matrix rather than all of $A$; this is a separately specified ideal-oracle representation, not a distributional identity for an arbitrary fixed expander.

**Encryption.** For a message polynomial $\mu\in\{0,1\}^{256}$, sample

$$
 r,e_1\leftarrow\chi^{1024},\qquad
 e_2\leftarrow\chi^{256},\qquad
 \rho\leftarrow U(\{0,1,2,3\}^{1024}),
$$

independently, and compute

$$
\begin{aligned}
 v&=4b+\rho,\\
 c_1&=F_2(A^Tr+e_1),\\
 c_2&=F_2(v^Tr+e_2+2048\mu).
\end{aligned}
$$

Both ciphertext components use the *same* ephemeral vector $r$. The variable $v$ is an internal randomized lift: it cannot be replaced by an independently sampled vector in the real experiment. When the simulated quotient is uniform, however, the quotient–remainder bijection can give precisely the distributional coupling required by a security hybrid.

**Decryption.** Compute

$$
 z=[2c_2-(2c_1)^Ts]_{4096}
$$

and decode coefficient $i$ as one exactly when $1024<z_i<3072$; ties are assigned zero. This exact convention matters for failure witnesses and equality at the decoding boundaries.

In the specified packing, the seeded public key contains a one-byte profile identifier, a 32-byte seed, and 1024 ten-bit quotient coefficients: $1313$ bytes in total. The ciphertext contains $1280$ eleven-bit quotient coefficients, hence $1760$ bytes. These are format sizes under the stated bit packing, not estimates of complete key-storage requirements or a security claim.

## The first nontrivial theorem: exact decryption error

Define the actual rounding remainders of an honest execution:

$$
\begin{aligned}
 r_0&=[As+e_0]_4,\\
 r_1&=[A^Tr+e_1]_2,\\
 r_2&=[v^Tr+e_2]_2.
\end{aligned}
$$

The message is encoded by $2048\mu$, which is divisible by two; it therefore does not change $r_2$ at fixed encryption coins. Euclidean division in each quantizer gives the exact congruence

$$
 z\equiv2048\mu+\varepsilon\pmod{4096},
$$

where

$$
\boxed{\displaystyle
\varepsilon=(e_0+\rho-r_0)^Tr+(r_1-e_1)^Ts+e_2-r_2.}
$$

The principal bilinear terms cancel because $(As)^Tr=(A^Tr)^Ts$ in the commutative ring $R$. No step in this identity assigns independent distributions to the remainders. The identity is exact even though the original formulas involve reduction and integer division.

For any output coefficient, $|\varepsilon_i|<1024$ guarantees correct decoding of that coefficient. This is a sufficient condition, not a necessary one: an integer representative of larger magnitude can wrap modulo $4096$ into the intended decoding region. The union event

$$
\mathcal F=\left\{\max_{0\le i<256}|\varepsilon_i|\ge1024\right\}
$$

therefore contains all honest decryption failures but need not coincide with the event of failure. A proof that estimates $\Pr[\mathcal F]$ still needs to justify its averaging order and its treatment of correlated coefficients.

For an honestly generated key $K=(pk,sk)$ define

$$
 d(K)=\max_\mu\Pr_\omega\bigl[
 \operatorname{Dec}(sk,\operatorname{Enc}(pk,\mu;\omega))\ne\mu\mid K
 \bigr],
$$

and let $\delta_{wc}=\mathbb E_Kd(K)$. Notice that the maximum over messages is **inside** the expectation over keys. This quantity is not a worst-case bound over every possible key.

The companion correctness analysis applies exponential moments in a carefully chosen conditional order. It removes the final binary remainder by a pointwise bound, averages independent encryption errors while their offsets are fixed, and only later averages secret coefficients and the key-generation errors. The bound uses finite rational quantities at $u=4/3$:

$$
 a=\frac{744601}{663552},\qquad
 b=\frac{589}{576},\qquad
 c=\frac{49}{48}.
$$

The resulting certificate has the form

$$
\delta_{wc}\le
256\left(1+\frac43\right)
\left(\frac{744601}{663552}\frac{589}{576}\right)^{1024}
\frac{49}{48}\left(\frac34\right)^{1024}
<2^{-212}.
$$

An explicit positive-probability execution produces an honest failure for the all-zero message, with event probability $2^{-8192}$. The current conclusion is therefore

$$
\boxed{2^{-8192}\le\delta_{wc}<2^{-212}.}
$$

The lower bound is scientifically important despite its tiny size: it rules out perfect correctness and prevents transformations from discarding decryption-failure terms merely for convenience. The full proof, including the conditional factorization and integer rational certificate, belongs in the correctness chapter, not in this introductory overview.

## From exact distributions to passive security

The passive-security argument has two separate hybrids. First, it replaces the real public quotient with a uniform quotient, using a four-row source sample. Second, it uses a five-row source sample to simulate both ciphertext components under the *same* ephemeral secret. A fifth source row is necessary to carry the scalar $v^Tr+e_2$ component while the first four carry $A^Tr+e_1$.

The second hybrid must generate its retained lifted vector $V$ together with the quotient $b=F_4(V)$; choosing independent $V$ and $b$ would fail to reproduce the real joint law. The exact quotient–remainder coordinate map explains why the correct coupling exists when $V$ is uniform. In the ideal full-matrix model, the companion proof yields, with the success-bias convention used here,

$$
\operatorname{Adv}_{\mathrm{CPA}}(\mathcal A)
\le \epsilon_{4,4}(\mathcal B_1)
+\epsilon_{4,5}(\mathcal B_2).
$$

This is a **conditional reduction**, not a theorem that the two source problems are hard. In the seeded ideal-matrix-oracle formulation, the precise statement must also account for any relevant oracle exposure or for a key-first coherent simulation, depending on the game. The presence of a 256-bit seed by itself does not justify replacing the ideal matrix oracle with SHAKE256 in a standard-model hardness proof.

## What the attack estimates mean

The research dossier evaluates several candidate classical attacks. In the documented cost models, the expanded MATZOV-style search finds a point around $\log_2T=213.657$, while the adapted rotation-aware coded-dual evaluation reports a point around $\log_2T=206.211$. The coded-dual calculation requires two stated adaptations to a pinned public implementation: the correct $\mathrm{CBD}_1$ zero mass, $\Pr[E=0]=1/2$, and a corrected low endpoint for the rotation-aware search bracket.

These are meaningful reproducible **estimated attack costs within defined models and search domains**. They are not complete attacks, not global optima, and above all not lower bounds on all classical or quantum adversaries. A statement such as “the smallest modeled attack found has cost $2^{206.211}$” does not entail “the construction has 206 bits of security.” The latter would require an assumption and analysis excluding cheaper attacks, including structural, algebraic, combinatorial, and quantum alternatives.

The distinction is particularly important in a module ring: expanding multiplication into coefficients reveals linear algebra with constraints inherited from the ring. Treating these coefficients as independent in a generic LWE estimate can hide attack strategies exploiting algebraic symmetry. Recent work on concrete gaps between LWE and structured variants reinforces the need to test, rather than assume, equivalence of attack costs [R6].

A separate issue is the cost of the reductions themselves. Arithmetic operation counts for a simulator, bit complexity in a security theorem, and operations in a heuristic lattice-estimation model do not automatically share a common work unit. Subtracting their logarithms produces a defensible security level only after the resource models are aligned and a quantified source-hardness bound is supplied at the reduction's actual amplified budget. The present dossier makes no such certification.

## Why decryption failures matter in the QROM

The original Fujisaki–Okamoto transformation [R10] and its modular analysis [R1] offer a systematic route from public-key encryption to key encapsulation and chosen-ciphertext security [R1]. When decryption may fail, their treatment depends on the form of rejection, the key-derivation interface, the exact security game, and the available failure-finding guarantees. In a quantum random-oracle model, the reduction must preserve coherent oracle access rather than assuming a classical adversary's query history can be inspected without disturbance.

The compact C1-K4 wrapper uses deterministic re-encryption driven by an oracle-derived tape, byte-exact checking, and an implicit-rejection fallback. The companion QROM manuscript sets a **single-user, key-first, fresh-ideal-oracle** experiment with classical ciphertext and decapsulation queries, but quantum-accessible hash and matrix oracles. It establishes rigidity through re-encryption checking and a spreadness parameter bounded below by

$$
\gamma\ge1280\log_2(4/3)=531.247999\ldots.
$$

In that stated model it derives a **conservative conditional QROM advantage inequality** from a published extractable-oracle failure-search lemma, a coherent graph simulation of the ciphertext-dependent key derivation and rejection branches, and quantum versions of the two exact Module-LWE source assumptions. That result is stronger than merely identifying a possible route, but narrower than a proof for real SHAKE256 instantiation, multiple users, arbitrary oracle-correlated advice, or 128-bit post-quantum security.

A separate failing-plaintext study analyzes two types of probability. The first bounds mean failure under independent secret keys; the second controls correlations when the encryption tape is shared across two independent keys. The current companion manuscript reports the same-tape second-moment estimate

$$
\mathbb E_\omega[p_m(\omega)^2]<2^{-247},
\qquad \sigma_{\delta ik}<2^{-123.5},
$$

where $p_m(\omega)=\Pr_K[\operatorname{Dec}(sk,\operatorname{Enc}(pk,m;\omega))\ne m]$ for an independent honest key, and $\sigma_{\delta ik}^2=\max_m\operatorname{Var}_{\omega} p_m(\omega)$ for an honest tape. This is a sharper diagnostic than the elementary mean-only variance bound, but it is not by itself a numerical security level for the complete KEM.

Recent literature has also developed FFP-NG analysis for structured lattice settings and families of errors including centered binomial distributions [R5]. It is therefore incorrect to claim that such results are inherently restricted to discrete Gaussians. The unanswered question is more precise: whether the hypotheses and transformations in that work match **this exact** compressed, lifted C1-K4 interface. The sharper FFP-NG route is open; the conservative conditional route does not depend on closing it.

## Contribution and evidence ledger

We distinguish five types of assertion throughout the thesis. A statement labelled **EXACT** follows from algebra, finite arithmetic, or probability under its written sampling laws; it must be accompanied by a proof, not just a passing script. A **CONDITIONAL** statement is a game-based implication with named source assumptions and resource budgets. An **ESTIMATED** result is a computed point in a specified cryptanalytic cost model. An **EXPERIMENTAL** result is a finite observation whose implementation and randomness must be reproducible. An **OPEN** item is an unresolved theorem, model transfer, or engineering/security obligation.

| Subject | Current thesis status | Scope boundary |
|---|---|---|
| Quotient–remainder and carry identities | Exact under stated divisibility conditions | No hardness follows from the identities alone |
| C1-K4 decryption-error identity | Exact algebraic statement | Correctness still requires probability analysis |
| $2^{-8192}\le\delta_{wc}<2^{-212}$ | Proved in Chapter 5 with an integer certificate | Average over honest keys; not a uniform-per-key guarantee |
| IND-CPA reduction | Conditional on two exact DMLWE source distributions | Does not establish source hardness |
| $1313$-byte public key; $1760$-byte ciphertext | Exact for stated ideal-model encodings | No concrete-expander security theorem |
| MATZOV and coded-dual costs | Model-dependent, reproducible search points | Not attack optima or security lower bounds |
| Conservative QROM bound | Conditional in a narrowly specified ideal-oracle experiment | Not general QROM or PQ-128 certification |
| Same-tape failure second moment | Companion probability theorem | Does not by itself solve FFP-NG |
| Sharper FFP-NG applicability | Open matching problem | No impossibility or automatic reduction claimed |

This ledger is part of the proof discipline, not a rhetorical qualification. Readers should be able to locate the assumptions, source version, data, and evidence used for each conclusion.

## Structure of the complete thesis

The twelve chapters are organized in four parts.

**Part I — Arithmetic, distributions, and structure.** Chapter 2 now develops quotient–remainder coordinates, carries, composition, entropy, randomized lifting, and the distinction between floor-LWR and other rounding conventions. Chapter 3 now explains polynomial and module operators, negacyclic convolution, structured coefficient matrices, and the difference between a module source and iid scalar LWE.

**Part II — The C1-K4 public-key encryption layer.** Chapter 4 now specifies the frozen algorithms, serialization, randomness, threat models, and exact source distributions. Chapter 5 now reconstructs the decryption-error identity, moment argument, rational correctness certificate, and explicit nonzero failure event. Chapter 6 now gives the joint-law IND-CPA hybrids and seeded-matrix interface. Chapter 7 now examines primal, dual, dual-hybrid, and coded-dual cost models, reproducibility, and structural caveats.

**Part III — Classical and quantum transformations.** Chapter 8 now reconstructs the conditional classical Fujisaki–Okamoto route, implicit rejection, and nonperfect correctness with explicit classical game interfaces and query allowances. Chapter 9 now fixes the QROM experiment, coherent oracle simulation, rigidity, spreadness, and conservative conditional reduction. Chapter 10 now studies find-failing-plaintext games, second moments, search lemmas, experiments, and the current FFP-NG matching problem.

**Part IV — Scientific synthesis.** Chapter 11 now consolidates the register of exact, conditional, estimated, and open claims and the relationship between parameters, reductions, and cryptanalysis. Chapter 12 now gives the final conclusions and a research agenda without upgrading estimates into theorems.

Companion materials provide the exact finite rational checks, executable verification instructions, pinned versions, and a release manifest linking each chapter to its sources. These supplements are arithmetic evidence and do not formally certify the quantum reduction.

## Closing perspective

The central thesis is methodological as much as cryptographic. A mathematical treatment of modular rounding is valuable only when it respects the information that quotient extraction discards. A correctness theorem is valuable only when its exact quantifiers survive the transition to a security game. A reduction is informative only when its source assumptions, oracle model, advantage conventions, and computational budget are explicit. A cryptanalytic estimate is useful only when its search domain and heuristic status are visible.

C1-K4 supplies one concrete laboratory in which these four disciplines intersect. The resulting thesis does not claim to have resolved the general hardness of structured rounding problems. It aims instead to establish a reliable, reproducible route from exact arithmetic and nonperfect correctness to carefully qualified classical and quantum security conclusions.


# Quotient--Remainder Coordinates and the Arithmetic of Rounding

## Why the discarded remainder must remain visible

Consider a positive modulus $M$ and a divisor $D$ of $M$. The map which replaces a canonical residue by the integer obtained after division by $D$ is deceptively simple. In a cryptographic construction it can be applied to a noisy secret product, to a public key, or to a ciphertext. A description such as "discard the low bits" is not an adequate probability model for the resulting protocol: those bits may be deterministic functions of the public matrix, hidden secrets, and errors, and the same hidden randomness may be present in later calculations.

This chapter develops the exact scalar calculus before introducing any computational hardness assertion. The distinction is deliberate. Euclidean division is an arithmetic identity; the joint law of quotient and remainder is a probabilistic property of the input; and the security of a rounded observation is an additional computational question. We will see that the same quantizer may be used in two systems whose public operators, secret distributions, or hardness assumptions differ.

All variables in this chapter are classical and finite. Unless otherwise stated, $M=DL$ with $D,L\ge 1$, and $[x]_M$ denotes the unique element of $\{0,\ldots,M-1\}$ congruent to the integer $x$ modulo $M$. The quotient and remainder are

$$
Q_{M,D}(x)=\left\lfloor\frac{[x]_M}{D}\right\rfloor,
\qquad R_{M,D}(x)=[x]_M-DQ_{M,D}(x).
$$

Thus $Q_{M,D}$ takes values in $\{0,\ldots,L-1\}$ and $R_{M,D}$ in $\{0,\ldots,D-1\}$. We sometimes write $Q(x)$ and $R(x)$ when $M,D$ are fixed. Neither $R_{M,D}(x)$ nor any similar remainder is an independent noise source unless a sampling argument proves that property.

## Euclidean coordinates, fibers, and probability laws

**Theorem 2.1 (Exact coordinates).** For every integer $x$ there is a unique pair $(q,r)$ with $0\le q<L$ and $0\le r<D$ such that

$$
[x]_M=Dq+r.
$$

It is given by $q=Q_{M,D}(x)$ and $r=R_{M,D}(x)$. Consequently the map

$$
\mathbb Z_M\longrightarrow \{0,\ldots,L-1\}\times\{0,\ldots,D-1\},
\qquad [x]_M\mapsto (Q(x),R(x))
$$

is bijective.

*Proof.* Euclidean division of $[x]_M$ by $D$ produces a unique quotient and remainder in the required ranges. The bound $[x]_M<DL$ forces the quotient below $L$. Conversely $(q,r)$ recovers the residue $Dq+r$. $\square$

The coordinate bijection is a bijection of sets. In general it is **not** a product-group isomorphism: the addition law has a carry. Each quotient $q$ has the fiber

$$
Q^{-1}(q)=\{Dq,Dq+1,\ldots,Dq+D-1\},
$$

which contains exactly $D$ residues. In particular, if $X$ is uniform on $\mathbb Z_M$, then

$$
\Pr[Q(X)=q,R(X)=r]=\frac1M=\frac1L\frac1D.
$$

This proves more than uniformity of both marginals.

**Theorem 2.2 (Exact independence under a uniform input).** When $X\leftarrow U(\mathbb Z_M)$, the pair $(Q(X),R(X))$ consists of two independent uniform random variables on alphabets of sizes $L$ and $D$.

*Proof.* The coordinate bijection sends a uniform point on $M=LD$ elements to a uniform pair on $L\cdot D$ elements. The displayed factorization proves independence. $\square$

**Corollary 2.3 (Randomized lifting).** If $U\leftarrow U(\mathbb Z_L)$ and $\rho\leftarrow U(\mathbb Z_D)$ independently, then

$$
V=DU+\rho
$$

is uniform modulo $M$, with $Q(V)=U$ and $R(V)=\rho$ exactly.

A critical qualification is that $V$ and $U$ are not independent when $L>1$: indeed $U=Q(V)$. This is why an encryption simulator must retain the joint transcript $(U,V)$ rather than generate independent uniform versions of those two values.

The qualification becomes sharper when an adversary has auxiliary public information. Suppose $Z$ is any finite public view, and $X$ is uniform on $\mathbb Z_M$ **independently of $Z$**. The bijection shows that $(Q(X),R(X))$ is a uniform pair independent of $Z$; the reverse simulation may sample $U$ independently of $Z$, then sample $\rho$ independently of $(U,Z)$. If $U$ is only marginally uniform, no such conclusion follows. For example, take $M=4,D=L=2$, a uniform public bit $Z$, and set $U=Z$. An independent uniform bit $\rho$ makes $2U+\rho$ marginally uniform on four residues, but its quotient reveals $Z$. Marginal uniformity does not imply uniformity *conditional on the public transcript*.

We shall use exactly this conditional qualification in Chapter 6. In C1-K4 the real public quotient $b$ need not be uniform; its lifting $4b+\rho$ therefore need not be uniform either. It becomes the exact uniform lift needed by the **second hybrid game only after the first hybrid has replaced the public quotient by a uniformly random one**.

## Addition and aggregate carries

Let $[x]_M=Dq_x+r_x$ and $[y]_M=Dq_y+r_y$, with $0\le r_x,r_y<D$. Define the ordinary addition carry

$$
C_D(r_x,r_y)=\left\lfloor\frac{r_x+r_y}{D}\right\rfloor\in\{0,1\}.
$$

**Theorem 2.4 (Addition).** The exact quotient and remainder of $x+y$ are

$$
Q(x+y)=\bigl[q_x+q_y+C_D(r_x,r_y)\bigr]_L,
\qquad R(x+y)=[r_x+r_y]_D.
$$

*Proof.* Divide $r_x+r_y$ by $D$ to obtain $D C_D+t$, where $0\le t<D$. Then $x+y\equiv D(q_x+q_y+C_D)+t\pmod{DL}$, and the claimed coordinates follow from uniqueness. $\square$

This carry is the precise obstruction to additivity. If $D>1$ and $L>1$, then $Q(1)=Q(D-1)=0$ while $Q(D)=1$; hence $Q$ is not a group homomorphism from $(\mathbb Z_M,+)$ to $(\mathbb Z_L,+)$. Conversely $D=1$ makes $Q$ the identity, and $L=1$ makes it the zero map. The lack of a homomorphism is a fact about arithmetic and does not establish one-wayness or cryptographic security.

For $t$ summands the exact carry can exceed one.

**Theorem 2.5 (Multiple summands).** If $[x_i]_M=Dq_i+r_i$ for $1\le i\le t$, then

$$
Q\!\left(\sum_{i=1}^t x_i\right)
=\left[\sum_{i=1}^t q_i+
\left\lfloor\frac{\sum_{i=1}^t r_i}{D}\right\rfloor\right]_L,
\qquad
R\!\left(\sum_{i=1}^t x_i\right)
=\left[\sum_{i=1}^t r_i\right]_D.
$$

The aggregate carry is an integer in $\{0,\ldots,t-1\}$.

*Proof.* Apply Euclidean division to the sum of the remainders, which is between $0$ and $t(D-1)$. Separate its quotient and remainder, and reduce the sum of the large quotients modulo $L$. $\square$

For signed perturbations it is often more useful to keep one input intact. With $[x]_M=Dq+r$ and any signed integer $e$, the same calculation gives

$$
Q_{M,D}(x+e)
=\left[q+\left\lfloor\frac{r+e}{D}\right\rfloor\right]_L,
\quad R_{M,D}(x+e)=[r+e]_D.
$$

The carry $\lfloor(r+e)/D\rfloor$ may be negative. Even a tiny perturbation can change a quotient when $x$ is near a cell boundary: at $M=16,D=4$, one has $Q(3)=0$ and $Q(3+1)=1$, whereas $Q(0)=0$ and $Q(0-1)=3$. The latter example demonstrates the modular wraparound that would be lost by replacing canonical residues with centered representatives mid-calculation.

## Multiplication and products of remainders

**Theorem 2.6 (Multiplication).** For the same $x,y$ as above,

$$
Q(xy)=\left[
Dq_xq_y+q_xr_y+q_yr_x+
\left\lfloor\frac{r_xr_y}{D}\right\rfloor
\right]_L,
\qquad R(xy)=[r_xr_y]_D.
$$

*Proof.* Expand $(Dq_x+r_x)(Dq_y+r_y)$. Its quotient on division by $D$ is $Dq_xq_y+q_xr_y+q_yr_x+\lfloor r_xr_y/D\rfloor$, while its remainder is $[r_xr_y]_D$. Reduction modulo $DL$ changes the quotient by a multiple of $L$ only. $\square$

One cannot normally determine the quotient of a product from the input quotients alone. Take $D>1,L>1$ and compare $(x,y)=(D,0)$ and $(D,1)$. Both inputs have the quotient pair $(1,0)$, but the product quotients are respectively $0$ and $1$. The missing information is an ordinary remainder, not an independently sampled error. In polynomial convolution, the same phenomenon occurs across many coupled products and signed additions.

## Nested truncation and discarded-bit composition

For aligned divisors, consecutive quotient extractions can be composed without error. Write $M=D_1D_2L$ and let $q_1=Q_{M,D_1}(x)$, $r_1=R_{M,D_1}(x)$. Divide the first quotient again as $q_1=D_2q_2+r_2$.

**Theorem 2.7 (Composition of aligned divisions).** The nested quotient and the combined remainder satisfy

$$
Q_{D_2L,D_2}(Q_{M,D_1}(x))=Q_{M,D_1D_2}(x),
\qquad R_{M,D_1D_2}(x)=D_1r_2+r_1.
$$

*Proof.* The identity $[x]_M=D_1(D_2q_2+r_2)+r_1$ is a Euclidean division by $D_1D_2$, since $0\le D_1r_2+r_1<D_1D_2$. $\square$

This law will explain why it is meaningful to speak of staged low-bit removal in an implementation. It does not justify truncating the separate terms of a sum or product before their carries have been combined: in general $Q(x+y)\ne Q(x)+Q(y)$, even when all divisors align.

## Min-entropy and a precise leakage interpretation

For a discrete variable $X$, the (classical) min-entropy is

$$
H_\infty(X)=-\log_2\max_x\Pr[X=x].
$$

For finite side information $Z$, the average conditional min-entropy is

$$
\widetilde H_\infty(X\mid Z)
=-\log_2 p_{\mathrm{guess}}(X\mid Z),\qquad
p_{\mathrm{guess}}(X\mid Z)
=\sum_z\max_x\Pr[X=x,Z=z].
$$

The word "average" is essential: this definition concerns an average optimal guessing probability, not the worst possible conditioning event. Quantum side information is not treated in this chapter.

**Theorem 2.8 (Entropy of a quotient).** For arbitrary finite $X$ supported on $\mathbb Z_M$ and arbitrary correlated classical $Z$,

$$
\widetilde H_\infty(Q(X)\mid Z)
\ge\widetilde H_\infty(X\mid Z)-\log_2D.
$$

In particular, $H_\infty(Q(X))\ge H_\infty(X)-\log_2D$.

*Proof.* For each $z$, each quotient fiber has exactly $D$ elements, so

$$
\max_q\sum_{x:Q(x)=q}\Pr[X=x,Z=z]
\le D\max_x\Pr[X=x,Z=z].
$$

Sum over $z$ and apply $-\log_2$ to both sides. $\square$

**Proposition 2.9 (Residual entropy after publishing a quotient).** Under the same hypotheses,

$$
\widetilde H_\infty(X\mid Z,Q(X))
\ge\widetilde H_\infty(X\mid Z)-\log_2L.
$$

*Proof.* Condition on the quotient and sum the optimal guessing masses over its $L$ possible values:

$$
\begin{aligned}
p_{\mathrm{guess}}(X\mid Z,Q(X))
&=\sum_z\sum_q\max_{x:Q(x)=q}\Pr[X=x,Z=z]\\
&\le L\sum_z\max_x\Pr[X=x,Z=z].
\end{aligned}
$$

Taking negative logarithms yields the claim. $\square$

These statements answer different questions. Theorem 2.8 bounds the entropy *of the quotient*, whereas Proposition 2.9 bounds the information remaining about the *original variable after quotient disclosure*. They cannot be interchanged. Neither guarantees a particular observed quotient has high conditional entropy. For example, let $M=4,D=L=2$ with probabilities $\Pr[X=0]=\Pr[X=1]=2/5$, $\Pr[X=2]=1/5$, $\Pr[X=3]=0$. After observing $Q(X)=1$, the value $X=2$ is determined. The average bound is not contradicted.

For a uniform $X$, each conditional fiber is uniform and $\widetilde H_\infty(X\mid Q(X))=\log_2D$ exactly. This describes information retained in a uniform input, not the computational difficulty of recovering a secret from many correlated outputs.

## Exact boundary with Learning With Rounding

Learning With Rounding was introduced by Banerjee, Peikert and Rosen [R9]. The floor convention used in this chapter is defined below.

A standard floor-quantizer for a source modulus $q$ and target modulus $p$ is

$$
\operatorname{Round}^{\mathrm{floor}}_{q\rightarrow p}(x)
=\left\lfloor\frac{p}{q}[x]_q\right\rfloor.
$$

With $q=M$, $p=L$, and $M=DL$,

$$
Q_{M,D}(x)=\operatorname{Round}^{\mathrm{floor}}_{M\rightarrow L}(x).
$$

This equality is literal and does not require a cryptographic assumption. It explains why an exact-divisor MODDIV quantizer is not a new primitive solely by virtue of containing modular reduction followed by integer division. But the expression *MODDIV is LWR* is incomplete. The full distributional problem must also match the law of the public operator, secret, errors, number of observations, basis, and rounding convention.

Nearest-integer rounding can be different. At $(M,D,L,x)=(16,4,4,3)$, floor rounding gives zero, but nearest rounding of $3\cdot4/16=3/4$ gives one. Matching alphabet sizes does not remove that discrepancy.

Likewise, applying the same scalar quotient to the outputs of an independent matrix, a ring multiplication, or a single overlapping Hankel matrix does not make the resulting computational distributions identical. A $2\times2$ Hankel matrix has equal off-diagonal entries with probability one, whereas an independently sampled $2\times2$ matrix over $\mathbb Z_M$ has that equality with probability $1/M$. A change of basis also need not commute with coefficientwise rounding. These distinctions will be formalized in Chapter 3.

A useful one-way implication can nevertheless be stated. If $(A,Y_0)$ and $(A,Y_1)$ are computationally indistinguishable, then for any efficiently computable deterministic $f$, the pairs $(A,f(Y_0))$ and $(A,f(Y_1))$ are computationally indistinguishable. A distinguisher against the latter could be run on the former after applying $f$. The claim requires **joint** indistinguishability with the public view $A$. Marginal indistinguishability of individual coordinates is insufficient: $(Z,Z)$ with $Z$ uniform has uniform marginals but visible equality across coordinates. Moreover, applying $Q$ to $Y=As+e$ produces $Q(As+e)$, not $Q(As)$; the exact signed carry identity above shows why the error cannot simply be erased.

## Non-divisible moduli and the limits of equal-fiber arguments

If $M$ is not divisible by $D$, write $M=aD+r$, with $0<r<D$. The quotient $\lfloor[x]_M/D\rfloor$ then has outputs $0,\ldots,a$, with $a$ full fibers of size $D$ and one last fiber of size $r$. A uniform $X$ yields probabilities $D/M$ on the first $a$ quotient values and $r/M$ on the last. The quotient and remainder are no longer jointly uniform on a full Cartesian product of the preceding type. The arithmetic Euclidean decomposition still holds, but equal-fiber independence and the simple uniform-lift construction must be reconsidered.

It would be inaccurate to conclude that quotient and remainder are necessarily dependent in *all* non-divisible situations. When $D>M$ the quotient is constant, hence independent of the remainder in the ordinary probabilistic sense, despite failure of the nondegenerate product-uniformity statement. C1-K4 uses the divisible powers of two $4096/4$ and $4096/2$, so its exact simulation avoids this complication.

## What has and has not been established

The statements in this chapter are unconditional arithmetic identities, or elementary probability/entropy theorems with their sampling laws displayed. Their applications to C1-K4 concern coordinatewise quotients and actual remainders. They do not establish hardness of any LWR or Module-LWE distribution, do not imply statistical independence of secret-dependent rounding remainders, and do not certify a public-key cryptosystem.

The following invariant will be used throughout the next chapters: whenever a reduction passes from a quotient to a lifted value, it must preserve the **complete joint distribution with the visible public transcript**. Theorem 2.2 permits that transition under uniformity and independence hypotheses; it never permits the transition for an arbitrary real-key quotient by declaration alone.

*Source and audit boundary.* This chapter expands the elementary arithmetic and distributional results of companion manuscript [S3]. Its proofs are included here so that the thesis does not require access to an internal archive. The computational implications will be treated separately in Chapters 6--10.


# Structured Polynomial and Module Operators

## The operator is part of the probability experiment

Arithmetic rounding does not specify the public linear transformation to which it is applied. The distribution of that transformation is as important as the scalar quantizer. A matrix with independent uniform scalar entries, a matrix built from negacyclic multiplication, a Toeplitz matrix, and a Hankel matrix may all map a coefficient vector to a coefficient vector; their constraints and their distributions are nevertheless different.

This chapter makes the operator model of C1-K4 explicit. The goal is not to replace Module-LWE by an informal unstructured LWE model. It is to show which computations are algebraically exact, which correlations remain in the public coefficient matrix, and how these correlations affect the statement of a security assumption and the interpretation of a cryptanalytic estimate.

## The negacyclic coefficient ring

Let $P\ge2$ and $n\ge1$ and consider

$$
R_{P,n}=\mathbb Z_P[x]/(x^n+1).
$$

Our fixed case is $R=R_{4096,256}$. Each ring element has a unique coefficient representation

$$
a=a_0+a_1x+\cdots+a_{n-1}x^{n-1},\qquad a_i\in\mathbb Z_P.
$$

The coefficient representation is a *basis choice*. For multiplication we first consider ordinary integer coefficient lifts, perform the signed convolution associated with $x^n=-1$, and finally reduce coefficients modulo $P$.

For $a,b$ represented by integer arrays, write $a\star b$ for this integer negacyclic convolution. Its $i$th coefficient is

$$
(a\star b)_i=
\sum_{j=0}^{i}a_jb_{i-j}
-\sum_{j=i+1}^{n-1}a_jb_{n+i-j},
\qquad 0\le i<n.
$$

The equality in $R$ is $ab=[a\star b]_P$ coefficientwise. The minus sign is not cosmetic: a term with degree at least $n$ crosses the relation $x^n=-1$. If an implementation uses an ordinary cyclic convolution, or flips the sign of a wrapped term, it implements a different algebra.

For $n=4$, multiplication by $a$ on a column of coefficients has matrix

$$
C(a)=\begin{pmatrix}
a_0&-a_3&-a_2&-a_1\\
a_1&a_0&-a_3&-a_2\\
a_2&a_1&a_0&-a_3\\
a_3&a_2&a_1&a_0
\end{pmatrix}\pmod P.
$$

The $(i,j)$ entry is $a_{i-j}$ for $i\ge j$ and $-a_{n+i-j}$ for $i<j$, with all subscripts in their stated ranges. Every coefficient of $a$ is reused in several positions. An $n\times n$ coefficient matrix $C(a)$ therefore has only $n$ independently sampled ring coefficients when $a$ is uniform, not $n^2$ independent scalar entries. Its rows are signed shifts of one another.

**Proposition 3.1 (Exact coefficient expansion).** The coefficient vector of $ab\in R$ is $C(a)\mathbf b\pmod P$, where $\mathbf b=(b_0,\ldots,b_{n-1})^T$.

*Proof.* Group the integer negacyclic convolution by the coefficient $b_j$. The coefficient multiplying $b_j$ in output position $i$ is the stated entry of $C(a)$. $\square$

The algebra permits an exact fast transform or an integer NTT-based implementation, provided its output is congruent to the signed convolution modulo 4096. A prime used internally for an exact convolution is an implementation modulus; it is **not** a silent change of the public cryptosystem's modulus.

## Module matrices and transpose identities

Let $k\ge1$ and $A=(A_{uv})\in R^{k\times k}$. The action on a module vector $s\in R^k$ is

$$
(As)_u=\sum_{v=1}^k A_{uv}s_v.
$$

After expanding every ring multiplication in coefficients, the map $s\mapsto As$ becomes a $(kn)\times(kn)$ scalar matrix consisting of $k^2$ signed negacyclic blocks $C(A_{uv})$. When $A$ is uniform in $R^{k\times k}$, its $k^2n$ coefficients are independent, but the entries of its expanded $(kn)^2$ scalar matrix are not.

For C1-K4, $k=4$ and $n=256$. Thus the expanded operator has dimension $1024\times1024$, while the sampled ring matrix contains $16\cdot256=4096$ independent 12-bit coefficients before any seed representation. Counting 1024 scalar rows and columns does **not** make the coefficient operator an iid uniform matrix over $\mathbb Z_{4096}$.

The transpose in the encryption algorithm acts on **module indices**, without replacing a polynomial by an inverse or reciprocal polynomial. Because $R$ is commutative, for $s,r\in R^k$,

$$
(As)^Tr
=\sum_{u,v}A_{uv}s_vr_u
=\sum_{v,u}A_{uv}r_us_v
=(A^Tr)^Ts.
$$

**Proposition 3.2 (Bilinear cancellation).** For every matrix $A\in R^{k\times k}$ and all $s,r\in R^k$,

$$
(As)^Tr-(A^Tr)^Ts=0\quad\text{in }R.
$$

No symmetry or invertibility assumption on $A$ is necessary.

This identity is the algebraic cancellation in the C1-K4 decryption equation. It is worth isolating before any discussion of probability: the cancellation is **deterministic**, while the residual terms contain random secrets, errors, and remainders. Chapter 5 will use both facts without conflating them.

## Toeplitz, Hankel, and negacyclic relations

An ordinary $m\times n$ Toeplitz matrix has entries $T_{ij}=t_{i-j}$; a Hankel matrix has entries $H_{ij}=h_{i+j}$. Both reuse coefficients. If $J$ reverses the columns of an $m\times n$ matrix, then $HJ$ is Toeplitz because $(HJ)_{ij}=h_{i+n-1-j}$ depends only on $i-j$. In an equation $Hs=y$, the corresponding Toeplitz form is $(HJ)(Js)=y$; transforming the matrix without transforming the secret would not preserve the instance.

The signed negacyclic block $C(a)$ is a *structured multiplication matrix*. It is Toeplitz: each diagonal is constant, and the bands on opposite sides of the wrap are linked by a prescribed sign. It is not a freely sampled ordinary Toeplitz matrix, and it is not a single overlapping Hankel ribbon. The public C1-K4 matrix comprises 16 independently sampled ring elements, each expanded to one signed multiplication block; no single polynomial supplies every entry of the whole $1024\times1024$ scalar operator.

A simple statistical test distinguishes many structured laws from an iid matrix. For a $2\times2$ Hankel matrix, the off-diagonal entries coincide with probability one. For an iid uniform $2\times2$ matrix over $\mathbb Z_P$, they coincide with probability $1/P$. This does not prove that the corresponding computational rounding problems are inequivalent under reductions; it merely prevents a claim of literal identity of their public matrix distributions.

Structural differences also matter when reasoning about attacks. Some primal and dual models unfold a module instance into a scalar lattice and then apply generic heuristics. That is a useful **model**, not a theorem stating that structure can never be exploited. Recent cryptanalytic literature exhibits ways in which ring/module structure can change attack costs [R6]. Our later attack chapter must therefore state both the mathematical source and the approximations of the estimator.

## A distinctive algebraic feature modulo two

C1-K4 uses the power-of-two modulus $4096=2^{12}$. Its ring reduction modulo 2 is

$$
\overline R=\mathbb F_2[x]/(x^{256}+1)
=\mathbb F_2[x]/((x+1)^{256}),
$$

where the second equality follows from the Frobenius identity in characteristic two. This is not a product of 256 unrelated scalar fields. It is a local ring with one maximal ideal generated by $x+1$.

The full ring $R=\mathbb Z_{4096}[x]/(x^{256}+1)$ is likewise local, with maximal ideal $\mathfrak m=(2,x+1)$. Indeed, $2$ is nilpotent, so it belongs to every maximal ideal, and after reducing modulo 2 the quotient has only the maximal ideal $(x+1)$. Its residue field is $R/\mathfrak m\simeq\mathbb F_2$.

**Theorem 3.3 (Invertibility test for the module matrix).** A matrix $A\in R^{4\times4}$ is invertible over $R$ if and only if its scalar evaluation $\overline A(1)\in\mathbb F_2^{4\times4}$ is invertible over $\mathbb F_2$.

*Proof.* A square matrix over a commutative ring is invertible if and only if its determinant is a unit. In a local ring an element is a unit exactly when its image in the residue field is nonzero. Applying the quotient $R\to R/\mathfrak m\simeq\mathbb F_2$ to $\det A$ gives $\det\overline A(1)$. $\square$

For a uniform $A\in R^{4\times4}$, each entry of $\overline A(1)$ is an independent fair bit. Consequently the probability of invertibility is

$$
\Pr[A\in\mathrm{GL}_4(R)]
=\frac{(2^4-1)(2^4-2)(2^4-2^2)(2^4-2^3)}{2^{16}}
=\frac{20160}{65536}=\frac{315}{1024}.
$$

The formula uses that an invertible binary $4\times4$ matrix can be formed by choosing four linearly independent columns successively. The equality can be checked by enumerating all $2^{16}$ binary matrices. It is an exact structural fact, **not an attack** on the noisy source: an invertible public operator does not reveal the secret once errors and quotient maps are present.

The statement also supplies a warning against importing a generic scalar-rank estimate. The determinant and invertibility of a module operator have algebraic restrictions inherited from its ring; a random iid $1024\times1024$ scalar matrix over $\mathbb F_2$ has a different law.

## Secret and error distributions and first-bit observations

The centered binomial law used throughout the frozen candidate is

$$
\chi=\mathrm{CBD}_1,\qquad
\Pr[E=-1]=\Pr[E=1]=\tfrac14,\quad
\Pr[E=0]=\tfrac12.
$$

Different coefficients of a sampled secret or error polynomial are independent unless otherwise stated. The distribution is symmetric, mean zero, and has variance $1/2$. It is not a uniform secret in $R$, nor is it a discrete Gaussian. Its parity is uniform because

$$
\Pr[E\bmod2=0]=\frac12,
\qquad \Pr[E\bmod2=1]=\frac14+\frac14=\frac12.
$$

For a source observation $Y=Bt+e\in R^j$, with $e$ consisting of independent CBD1 coefficients, the parity of each coefficient of $Y$ is therefore uniform conditional on $(B,t)$. Moreover the parity coefficients are jointly independent under this conditioning. Thus the reduction of *this unrounded source observation* modulo 2 is a fully uniform mask. It would be erroneous to extrapolate from that observation that the full twelve-bit source is uniform, or that the public-key encryption scheme has no higher-bit or carry correlations.

A useful elementary weight calculation follows immediately. If $t\leftarrow\chi^{1024}$, each coefficient is nonzero with probability $1/2$, so

$$
\#\{i:t_i\ne0\}\sim\operatorname{Binomial}(1024,1/2).
$$

This determines the distribution of the global nonzero weight; it does not exclude hybrid attacks that exploit sparse *subsets* or the nonuniformity of conditioned secrets.

## The exact decisional source families

For $j\in\{4,5\}$ define the source distribution $\mathrm{DMLWE}_{4,j}^{\chi}$ over pairs $(B,Y)$ as follows. Draw

$$
B\leftarrow U(R^{j\times4}),\qquad
 t\leftarrow\chi^{4n},\qquad
 e\leftarrow\chi^{jn}
$$

independently, interpreting the coefficient vectors as elements of $R^4$ and $R^j$. The real sample is $(B,Bt+e)$; the uniform sample is $(B,U)$ with $U\leftarrow U(R^j)$ independently of $B$. Let $\epsilon_{4,j}(\mathcal D)$ be the absolute difference in acceptance probabilities of a distinguisher on those two joint samples.

The names $\mathrm{DMLWE}_{4,4}^{\chi}$ and $\mathrm{DMLWE}_{4,5}^{\chi}$ record **different joint source distributions**, not two arbitrary descriptions of the same sample count. The latter contains five ring observations that share the same secret $t$; the former contains four. The four-row source gives $1024$ coefficient observations and the five-row source gives $1280$. A reduction that uses five rows cannot simply be replaced by a four-row hypothesis without another argument.

These are *assumptions*, not hardness theorems proved here. The dimensions, modulus, secret distribution, and operator law are fixed rather than varied through an asymptotic family. As a result, a computational source assumption at this specific instance should be stated with an explicit advantage and resource budget; an asymptotic theorem for a different distribution cannot be imported by naming it "Module-LWE".

## Coefficientwise rounding does not erase the operator law

Let $Q_{P,D}$ act on every coefficient of a ring polynomial. As Chapter 2 shows, $Q_{P,D}(a+b)$ generally differs from $Q_{P,D}(a)+Q_{P,D}(b)$, and a basis change before rounding can change the output. Consequently the observable $Q_{P,D}(Bt+e)$ retains features of the coefficient basis, the structure of $B$, the secret distribution, and the actual remainders.

Post-processing can transfer indistinguishability in one direction: if the **joint source** $(B,Bt+e)$ is computationally indistinguishable from $(B,U)$ and the quotient map is efficiently computable, then $(B,Q(Bt+e))$ is indistinguishable from $(B,Q(U))$. When $D\mid P$, the latter quotient vector is exactly uniform over its quotient alphabet. This reasoning does not establish the source assumption. Nor does it replace $Q(Bt+e)$ by $Q(Bt)$, because the signed carry from $e$ remains.

The promised distinction is now clear. *Algebraic cancellation* (Proposition 3.2) is unconditional. *Exact expansion* (Proposition 3.1) exposes correlations. *Distributional simulation* requires matching entire public transcripts. *Security* then depends on the two precise decisional source problems and further transformations. Chapter 4 gives the encryption scheme to which those distinctions will be applied.

*Source and audit boundary.* The ring-specific cryptographic setting is fixed in companion manuscript [S1]. The emphasis on the operator law and the distinction from single-Hankel and iid scalar sources is necessary for all later reductions and attack discussions. No equivalence between those source families is claimed here.


# A Complete Mathematical Specification of C1-K4

## Freezing the research instance

A correctness theorem has little force unless it can be tied to one unambiguous algorithm. A security reduction has even less force if a matrix distribution, a rounding convention, or the use of a common ephemeral secret changes between games. This chapter records the frozen public-key encryption (PKE) layer of **C1-K4-P12-CBD1** and separates its abstract mathematical specification from the extra engineering decisions required for an interoperable implementation or a key-encapsulation mechanism (KEM).

The frozen tuple is

$$
(n,k,P,Q,C)=(256,4,4096,4,2),\qquad
R=\mathbb Z_{4096}[x]/(x^{256}+1).
$$

The symbol $P$ is the ring modulus, $Q$ the divisor used to publish the public-key quotient, and $C$ the divisor used to compress ciphertext components. It is useful to keep the names distinct: $P$ is not a prime, and "division" here is exact integer division of canonical coefficient representatives, not multiplication by an inverse modulo $P$.

| Quantity | Frozen value | Interpretation |
|:--|:--|:--|
| $n$ | $256$ | Coefficients per ring polynomial |
| $k$ | $4$ | Module rank |
| $P$ | $4096=2^{12}$ | Coefficient modulus |
| $Q$ | $4$ | Public-quotient divisor |
| $C$ | $2$ | Ciphertext-quotient divisor |
| $\chi$ | $\mathrm{CBD}_1$ | Each secret and explicit error coefficient |
| Message | $\{0,1\}^{256}$ | One bit per coefficient |
| Public matrix | $A\in R^{4\times4}$ | Uniform ring entries in the abstract model |
| Lifting alphabet | $\{0,1,2,3\}$ | Independent remainder per public quotient coefficient |

The ring is represented in the power basis $1,x,\ldots,x^{255}$ with increasing coefficient order. All quotient maps act **coefficientwise**. We use the notation

$$
F_D(z)=\left\lfloor\frac{[z]_P}{D}\right\rfloor,
\qquad D\mid P,
$$

with $[z]_P$ the canonical residue. A polynomial coefficient quotient is identified with its canonical integer representative in $\{0,\ldots,P/D-1\}$; it is not silently lifted back to $\mathbb Z_P$ until the algorithm explicitly does so. Products are negacyclic and are reduced modulo $P$ at the required steps.

## Sampling conventions and all sources of randomness

A coefficient $E\leftarrow\chi=\mathrm{CBD}_1$ is generated as $E=B_0-B_1$ from independent uniform bits $B_0,B_1$, giving $-1$ with probability $1/4$, $0$ with probability $1/2$, and $1$ with probability $1/4$. Sampling $\chi^{256}$ means choosing each coefficient independently; sampling $\chi^{1024}$ means four independent polynomials of 256 independent coefficients each. No pair of different named random arrays shares coins unless the scheme explicitly reuses the same variable in an expression.

Key generation draws $A$, $s$, and $e_0$ independently. Honest encryption draws $r$, $e_1$, $e_2$, and $\rho$ independently of one another and independently of all key-generation randomness. The **same sampled $r$** is used in both ciphertext components. That reuse is not an implementation optimization; it is part of the joint distribution whose security is studied.

The uniform remainder $\rho$ has $1024$ coefficients, each uniformly distributed in $\{0,1,2,3\}$. It does not replace the unknown remainder of $As+e_0$; rather, it creates an explicit randomized lift of the published quotient. Confusing those two remainders would invalidate both the error equation and the second security hybrid.

## Key generation: the public quotient

The abstract key-generation algorithm is:

1. Draw $A\leftarrow U(R^{4\times4})$, with independent uniform ring elements.
2. Draw $s,e_0\leftarrow\chi^{1024}$ independently, viewed as elements of $R^4$.
3. Compute $b=F_4(As+e_0)\in\{0,\ldots,1023\}^{1024}$.
4. Output the mathematical public key $pk=(A,b)$ and PKE secret key $sk=s$.

Equivalently, let the *actual* public-key remainder be

$$
r_0=[As+e_0]_4.
$$

Then the exact coefficientwise equality in the ring is

$$
4b\equiv As+e_0-r_0\pmod{4096}.
$$

The distribution of $r_0$ is determined by $A,s,e_0$. Nothing in key generation makes $r_0$ independent of those inputs. For fixed $A,s,e_0$, it is a fixed array, not a fresh noise variable.

## Encryption: the retained lift and shared ephemeral vector

For a message $\mu=(\mu_0,\ldots,\mu_{255})\in\{0,1\}^{256}$ and public key $(A,b)$, perform:

1. Sample $r,e_1\leftarrow\chi^{1024}$, $e_2\leftarrow\chi^{256}$, and $\rho\leftarrow U(\{0,1,2,3\}^{1024})$ independently.
2. Construct the **internal lift** $v=4b+\rho\in R^4$ coefficientwise. The lift is not transmitted.
3. Set $c_1=F_2(A^Tr+e_1)\in\{0,\ldots,2047\}^{1024}$.
4. Set $c_2=F_2(v^Tr+e_2+2048\mu)\in\{0,\ldots,2047\}^{256}$.
5. Output $c=(c_1,c_2)$.

Here $2048\mu$ is the polynomial $\sum_{i=0}^{255}2048\mu_i x^i$. The module transpose $A^T$ exchanges the four module indices, not the order or signs of the polynomial coefficients. A ring multiplication in $v^Tr$ denotes the sum of four negacyclic products.

In the real system $b=F_4(As+e_0)$ is not assumed uniform. Consequently $v=4b+\rho$ is **not** generally uniform in $R^4$ in the honest encryption distribution. Chapter 2 supplies a uniform-lift identity only for the hybrid game in which $b$ is already an independent uniform quotient. It is essential not to insert uniformity of $v$ into a proof about the unmodified real-key distribution.

Define the actual ciphertext remainders by

$$
r_1=[A^Tr+e_1]_2,\qquad
r_2=[v^Tr+e_2]_2.
$$

The second remainder does not depend on $\mu$ when the other coins are fixed, because $2048\mu$ is even. The remainders $r_1,r_2$ still depend on their respective pre-rounding expressions; in particular $r_2$ need not be independent of $e_2$.

The two exact identities are

$$
2c_1\equiv A^Tr+e_1-r_1\pmod{4096},
$$

$$
2c_2\equiv v^Tr+e_2+2048\mu-r_2\pmod{4096}.
$$

They are the starting point for the correctness analysis in Chapter 5.

## Decryption: thresholds and exact ties

Parsing first checks the expected coefficient counts, sizes, and canonical encodings. For a valid ciphertext, compute the polynomial

$$
z=[2c_2-(2c_1)^Ts]_{4096}.
$$

For each coefficient $z_i\in\{0,\ldots,4095\}$, return bit

$$
\widehat\mu_i=
\begin{cases}
1,&1024<z_i<3072,\\
0,&\text{otherwise}.
\end{cases}
$$

The inequalities are strict. In particular the two midpoint ties at $z_i=1024$ and $z_i=3072$ are both decoded as zero. An invalid PKE ciphertext encoding returns a distinguished failure. A KEM wrapper uses its separately specified implicit-rejection branch for a failure or a re-encryption mismatch; a bare PKE does not manufacture a fallback shared key.

For the ideal no-error input $s=0$, $e_0=e_1=e_2=0$, $r=0$, and any legal $\rho$, we obtain $c_1=0$, $c_2=1024\mu$, and $z=2048\mu$. The decoding then returns $\mu$ exactly. This elementary example confirms the message scale, but it says nothing about the probability of correctness for honestly sampled nonzero secrets and errors.

The distinction between a bit-level *decode tie* and an invalid *byte encoding* must be preserved in tests. A canonical ciphertext whose $z_i$ is exactly a threshold remains a valid ciphertext input; the decoder returns zero for that position. A noncanonical byte array should be rejected before arithmetic. Those are separate cases.

## Encodings, exact public sizes, and random tapes

Each public quotient coefficient lies between $0$ and $1023$, so it occupies exactly ten bits. There are $4\cdot256=1024$ such coefficients. Each ciphertext quotient coefficient lies between $0$ and $2047$, so it occupies eleven bits. There are $1024+256=1280$ ciphertext coefficients.

With a one-byte version/profile field and a 32-byte matrix seed, the stated public-key payload is

$$
\begin{aligned}
|pk|&=1+32+\frac{1024\cdot10}{8}=1313\ \text{bytes},\\
|c|&=\frac{1280\cdot11}{8}=1760\ \text{bytes}.
\end{aligned}
$$

Both packed streams are byte aligned. The mathematical unseeded matrix contains $16\cdot256\cdot12/8=6144$ bytes of coefficients. Adding the unseeded public quotient gives $7424$ bytes, before version fields or transport metadata. The 1313-byte figure is **not** a lossless compression of every possible full independent matrix into 32 bytes; it uses the seeded ideal-oracle matrix-generation model described below.

The public payload orders the profile byte, the seed and the four quotient polynomials. The ciphertext orders the four polynomials of $c_1$ and then the polynomial $c_2$. Within each polynomial, coefficients run in increasing degree. Each field contributes its least significant bit first, starting at bit zero of the first byte. Every 1760-byte string parses to exactly one ciphertext vector. A string of any other length fails parsing. Assignment of a concrete public profile identifier and byte-level encoding of oracle domains belongs to an interoperability specification.

One honest encryption tape may be generated from $2816$ independent uniform bytes. The first $2304$ bytes provide one CBD1 coefficient each as bit zero minus bit one, in this order: $1024$ coefficients for $r$, $1024$ for $e_1$, and $256$ for $e_2$. The remaining $512$ bytes provide $1024$ independent uniform nibbles, low nibble first in each byte, each reduced modulo four, for the coefficients of $\rho$. The unused high bits of the CBD1 source bytes and the unused upper two nibble bits do not create correlations: different independent bytes and distinct nonoverlapping nibbles are independent.

This mapping is many-to-one from uniform tapes to encryption coins. Injectivity is neither asserted nor needed for the correctness proof. The exact tape distribution matters in the deterministic re-encryption used by a Fujisaki--Okamoto wrapper.

## Ideal matrix expansion versus a concrete hash

In the abstract model, $A$ is an independent uniform $4\times4$ matrix over $R$. For a compact public key, sample a uniform $256$-bit seed $\sigma_A$ and query a distinct domain-separated ideal matrix oracle $H_A$ at injective encodings of $(\sigma_A,u,v)$ for every pair of module indices. Each response consists of $512$ bytes. Interpret it as $256$ little-endian 16-bit words and reduce each word modulo 4096 to form one ring element.

A uniform 16-bit word has precisely $16$ preimages for every low twelve-bit result, so reduction modulo 4096 produces an exactly uniform coefficient. Distinct oracle inputs produce independent words in the ideal-oracle experiment. When simulating a supplied uniform coefficient $a_i$, the simulator may independently sample four discarded high bits to reconstruct a uniformly distributed 16-bit raw word with low twelve bits $a_i$. This reproduces the **full raw oracle response**, not merely its visible reduced coefficient.

For a newly sampled seed, an adversary that previously queried $q_A^{\mathrm{pre}}$ different candidate seed values may cause a seed collision with probability at most

$$
\epsilon_A=\min(1,q_A^{\mathrm{pre}}2^{-256}).
$$

Outside that bad event, the matrix-oracle transcript can be simulated with its required joint law. The analysis aborts on a collision event; it must not silently resample seeds until the simulation succeeds, since that changes the probability space.

No proof that a fixed concrete expander (including SHAKE256) is indistinguishable from an independent ideal matrix oracle follows from this argument. An implementation using a concrete expander adopts an additional assumption and must fix its input/output byte conventions. This separation is central to the security discussion of Chapter 6.

## What the scheme specification does not establish

The base PKE permits honest encryption of 256-bit messages, but it does not prove that decrypting always succeeds. Its exact correctness is analyzed next; the theorem will show a tiny but *nonzero* averaged worst-message failure probability. Nor does the design, by itself, imply chosen-plaintext indistinguishability. The proof route invokes the two precise Module-LWE decision sources $\mathrm{DMLWE}_{4,4}^{\chi}$ and $\mathrm{DMLWE}_{4,5}^{\chi}$ defined in Chapter 3.

For subsequent classical and quantum KEM analysis, the intended wrapper derives the deterministic encryption tape from an independent oracle $G(pk,\mu)$, derives the successful key from $H(pk,\mu,c)$, and uses a secret fallback value for ciphertexts that fail canonical parsing or byte-exact deterministic re-encryption. Chapters 8 and 9 must specify the corresponding security games and the exact oracle-domain conventions. Neither a fixed 2816-byte tape specification nor a numerical correctness bound is enough to conclude chosen-ciphertext or quantum security.

Finally, this is a **research profile**, not a production deployment specification. Constant-time implementation, side-channel behavior, secret-key serialization, authenticated version negotiation, multi-user analyses, and a complete end-to-end concrete security certification are outside the proven scope of the present algorithms. Their omission is documented rather than hidden behind a numerical attack estimate.

The abstract profile follows [S1]. It fixes sizes, coin laws and serialization order. Concrete identifiers and a constant-time implementation require an implementation specification.


# Exact Decryption Error and Nonperfect Correctness

## Correctness is a quantified probability statement

A scheme may decrypt almost every honestly generated ciphertext correctly without being perfectly correct. The distinction becomes cryptographically important when a later transformation must answer decapsulation queries or simulate a random oracle: rare failures may be exploitable, or may contribute explicit terms to a reduction. This chapter establishes the correctness statement for the *frozen mathematical C1-K4 PKE*, with all remainder dependencies retained.

Three separate objects must not be conflated. The first is an **exact integer expression** for a lifted decryption error. The second is a **sufficient event** for correct decoding. The third is a **probability bound** under the actual secret, error, and lifting laws. A residue equation modulo $4096$ cannot by itself be used to infer a small integer error; an actual tail bound is needed.

The statement proved here concerns honestly sampled public keys and ciphertexts. It is neither a guarantee for every adversarially selected public key nor a bound on arbitrary malformed ciphertexts. Those are different questions addressed by a future CCA analysis.

## Exact remainders and the cancellation identity

Let $A,s,e_0,r,e_1,e_2,\rho,\mu$ be the actual variables of Chapter 4, with

$$
\begin{aligned}
b&=F_4(As+e_0),& v&=4b+\rho,\\
c_1&=F_2(A^Tr+e_1),&
c_2&=F_2(v^Tr+e_2+2048\mu).
\end{aligned}
$$

Define actual, not resampled, coefficientwise remainders:

$$
r_0=[As+e_0]_4,\qquad
r_1=[A^Tr+e_1]_2,\qquad
r_2=[v^Tr+e_2]_2.
$$

The symbol $r$ without a subscript denotes the ephemeral *module secret*; $r_0,r_1,r_2$ denote three different remainder arrays. The order of operations is always reduce the pre-rounding expression to a canonical residue modulo 4096, then divide; because $4$ and $2$ divide 4096, the remainder may equivalently be taken directly modulo its small divisor. The message does not change $r_2$ because $2048\mu$ is even.

**Theorem 5.1 (Exact error identity).** For each honest execution define the integer-polynomial lift

$$
\boxed{\ 
\varepsilon=(e_0+\rho-r_0)^Tr
+(r_1-e_1)^Ts+e_2-r_2.
\ }
$$

Products in this display are integer negacyclic convolutions, summed over the four module components. If $z$ is the value computed before bit decoding, then

$$
\boxed{\ z\equiv 2048\mu+\varepsilon\pmod{4096}.\ }
$$

*Proof.* Coefficientwise Euclidean division yields

$$
\begin{aligned}
4b&\equiv As+e_0-r_0,\\
2c_1&\equiv A^Tr+e_1-r_1,\\
2c_2&\equiv v^Tr+e_2+2048\mu-r_2
\end{aligned}\qquad\pmod{4096}.
$$

Substitute $v=4b+\rho$ and use the first equation to obtain

$$
v\equiv As+e_0-r_0+\rho\pmod{4096}.
$$

Hence, using the specified decryption expression,

$$
\begin{aligned}
z&\equiv 2c_2-(2c_1)^Ts\\
&\equiv 2048\mu+(As)^Tr+(e_0+\rho-r_0)^Tr
+e_2-r_2\\
&\qquad -(A^Tr)^Ts-(e_1-r_1)^Ts\\
&\equiv 2048\mu+\varepsilon\pmod{4096}.
\end{aligned}
$$

The last step uses the unconditional bilinear cancellation of Proposition 3.2. No distributional independence has been used. The integer negacyclic lift is a valid choice of representatives because reduction modulo 4096 commutes with addition and multiplication. $\square$

The equation does **not** say that $\varepsilon$ is independent of any remainder. In particular, $r_0$ depends on $(A,s,e_0)$, $r_1$ on $(A,r,e_1)$, and $r_2$ on $(v,r,e_2)$. A proof which redraws them as fresh independent uniform residues would be a proof about a different encryption experiment.

## Decode geometry and the worst-message parameter

For a bit $\mu_i=0$, the ideal center of $z_i$ is zero modulo 4096; for $\mu_i=1$, it is 2048. The decoder assigns one precisely to the open modular interval $(1024,3072)$, with both endpoint ties assigned to zero.

**Lemma 5.2 (Sufficient correctness region).** If $|\varepsilon_i|<1024$, then bit $i$ is decoded correctly for either message bit. Consequently any honest decryption failure belongs to the event

$$
\mathcal F=\left\{\max_{0\le i<256}|\varepsilon_i|\ge1024\right\}.
$$

*Proof.* If $\mu_i=0$ and $|\varepsilon_i|<1024$, the circular residue of $\varepsilon_i$ lies strictly closer to zero than to 2048. If $\mu_i=1$, shifting by 2048 puts the residue inside $(1024,3072)$. Both conclusions follow directly from the strict decoder thresholds. $\square$

The implication is one-way. A large integer error can wrap modulo 4096 to a correctly decoded bit. Thus $\mathcal F$ is a sufficient *failure-covering event*, not an exact equality with the failure event.

For an honestly generated key pair $K=(pk,sk)$ define

$$
d(K)=\max_{\mu\in\{0,1\}^{256}}
\Pr_{\omega}\bigl[
\operatorname{Dec}(sk,\operatorname{Enc}(pk,\mu;\omega))\ne\mu
\mid K\bigr],
$$

where $\omega$ denotes fresh honest encryption randomness. Define the averaged worst-message failure probability

$$
\boxed{\quad \delta_{wc}=\mathbb E_{K}\bigl[d(K)\bigr].\quad}
$$

**A common worst message.** Couple all messages using the same key and tape. The shift $2048\mu_i$ is even, so $r_2$ and every integer error coefficient are independent of the message in this coupling. For $t=[\varepsilon_i]_{4096}$ away from $1024,3072$, shifting by 2048 exchanges the two decoding regions, and the failure indicator is the same for bits zero and one. At either boundary both shifted residues decode to zero, so only bit one fails. Taking a union over coordinates gives $F_m(K,\omega)\subseteq F_{\mathbf1}(K,\omega)$ pointwise. Thus, writing $p_K(m)=\Pr_\omega(F_m\mid K)$ and $\mathbf1$ for the message with every coefficient one,
$$
d(K)=p_K(\mathbf1),\qquad
\delta_{wc}=\mathbb E_K p_K(\mathbf1)=\max_m\mathbb E_K p_K(m).
$$
This identity uses this decoder's tie rule. The definition still takes the maximum inside the key average. Individual keys may exceed that average. For the exponential bound below each failure also lies in the message-independent event $\mathcal F$.

A technical point is worth making explicit: the PKE key pair $(pk,sk)$ does not retain the latent key-generation error $e_0$. If multiple $e_0$ values lead to the same key pair, then conditional error probabilities are averaged over those latent draws. Bounding the same-coins event for the extended key-generation variables and then averaging is still valid; it must not be mistaken for a claim that $e_0$ is an independently resampled decryption input.

## Why a deterministic absolute-value bound is insufficient

At a fixed output coefficient, each of the two module inner products comprises $4\cdot256=1024$ signed products. The bounds $|e_0|\le1$, $0\le\rho,r_0\le3$, $|r|\le1$, $0\le r_1\le1$, $|e_1|,|s|\le1$, $|e_2|\le1$, and $0\le r_2\le1$ give a very loose deterministic bound of order thousands, far above the decision threshold 1024.

Using only the worst magnitude of each summand loses the cancellations that result from the symmetric CBD1 secrets. Conversely, declaring all 1024 summands independent would be unjustified because the remainder arrays are computed from coupled products. The useful middle path is to average **independent primitive coins in a controlled order**, while maximizing over the dependent remainder offsets pointwise.

## Exact moment functions

Fix a rational $u>1$. Let $E,R,S$ be independent CBD1 variables and let $T$ be independent uniform on $\{0,1,2,3\}$. Define

$$
\begin{aligned}
a(u)&=\mathbb E_{E,R,T}\,u^{(|E|+T)R},\\
b(u)&=\mathbb E_{S}\left[\max_{c\in\{0,1\}}
\mathbb E_E\,u^{S([c+E]_2-E)}\right],\\
c(u)&=\mathbb E_E\,u^E.
\end{aligned}
$$

The expectation over $S$ comes *after* the maximum over $c$ in $b(u)$. Moving that maximum outside the secret expectation would be a different expression. The offset $c$ is not asserted to be random or independent of the secrets: taking the maximum handles every value it may have in a conditional calculation.

**Lemma 5.3 (Domination of a hidden modulo-four remainder).** For integer $e$, $t\in\{0,1,2,3\}$, and independent $R\leftarrow\chi$, $T\leftarrow U(\{0,1,2,3\})$,

$$
\mathbb E_{R,T}u^{(e+T-t)R}
\le \mathbb E_{R,T}u^{(|e|+T)R}.
$$

The same upper bound holds if the exponent on the left is negated.

*Proof.* Write $f(y)=\mathbb E_R u^{yR}$. Symmetry of CBD1 makes $f$ even, while $f''(y)=(\log u)^2\mathbb E_R[R^2u^{yR}]\ge0$ makes it convex. Consequently

$$
g(h)=\frac14\sum_{j=0}^3 f(h+j)
$$

is convex and symmetric about $h=-3/2$. As $t$ ranges over $\{0,1,2,3\}$, the left side is $g(e-t)$. A convex function attains its maximum over this four-point interval at an endpoint. For $e\ge0$ the farthest endpoint from $-3/2$ is $t=0$, and for $e\le0$ it is $t=3$; in the second case reflect the uniform $T$ and use evenness to obtain $g(|e|)$. Negating the original exponent is absorbed by symmetry of $R$. $\square$

The lemma is a **pointwise bound in the actual hidden remainder $t$**. It does not assert that $r_0$ is uniform, or that an error and its remainder are independent.

## Joint exponential bound without independent-remainder replacement

**Theorem 5.4 (Ordered exponential-moment bound).** In the frozen C1-K4 experiment, for every output coefficient $i$,

$$
\mathbb E\,u^{\varepsilon_i}
\le (a(u)b(u))^{1024}c(u),
$$

$$
\mathbb E\,u^{-\varepsilon_i}
\le u\,(a(u)b(u))^{1024}c(u).
$$

It follows that

$$
\boxed{\quad
\delta_{wc}\le
256(1+u)(a(u)b(u))^{1024}c(u)u^{-1024}.
\quad}
$$

*Proof.* Fix a coefficient $i$ and use the integer negacyclic representation from Theorem 5.1. Every coefficient of a vector in each module inner product occurs in precisely one paired term for this chosen output coordinate; the convolution contributes a fixed sign to that term. There are $1024$ such pairs in the first product and $1024$ in the second. The proof does not require independence across different output coordinates.

**First**, bound the final remainder before attempting to average $e_2$. Since $r_{2,i}\in\{0,1\}$, one has $u^{-r_{2,i}}\le1$ and $u^{r_{2,i}}\le u$. The positive tail therefore loses no factor and the negative tail loses at most $u$. After this pointwise step the remaining $e_{2,i}$ may be averaged independently, yielding $\mathbb E u^{\pm e_{2,i}}=c(u)$. No independence between $e_2$ and $r_2$ was assumed.

**Second**, condition on $A,s,e_0,r,\rho$. The remainder in a coefficient of $r_1$ is the parity of a fixed coefficient of $A^Tr$ plus its corresponding independent coefficient of $e_1$. Thus for each matched signed contribution the conditional moment, as a function of its secret coefficient $S$, is bounded by

$$
\max_{c\in\{0,1\}}\mathbb E_E
u^{S([c+E]_2-E)}
$$

Here $S$ denotes the fixed secret coefficient times the sign of its negacyclic term and the sign of the chosen tail. Its distribution is CBD1 only at the final, unconditional secret average. Conditional on $A,s,e_0,r,\rho$, independent coefficients of $e_1$ give a product of these factors. Each depends on its signed secret coefficient alone.

**Third**, hold $A,s,e_0$ fixed, so that the *actual* $r_0$ is fixed. In the first inner product, average each independent pair consisting of an ephemeral CBD1 coefficient of $r$ and a uniform lifting coefficient of $\rho$. Lemma 5.3 bounds the contribution for every fixed value of $r_0$, leaving a factor depending only on the corresponding $|e_0|$. There is no need to assign any probabilistic law to $r_0$.

**Finally**, average the mutually independent coefficients of $s$ and $e_0$. The first family of bounded factors gives $a(u)^{1024}$ and the second family gives $b(u)^{1024}$, with $c(u)$ already obtained from $e_2$. This produces the two moment inequalities, including the extra factor $u$ in the negative tail.

Apply Markov's inequality to $u^{\varepsilon_i}$ and $u^{-\varepsilon_i}$ at the threshold 1024, then take a union bound over all 256 coefficients:

$$
\Pr[\mathcal F]\le
\sum_{i=0}^{255}\left(
\frac{\mathbb E u^{\varepsilon_i}}{u^{1024}}+
\frac{\mathbb E u^{-\varepsilon_i}}{u^{1024}}
\right)
\le256(1+u)(ab)^{1024}cu^{-1024}.
$$

The same-coins covering event in Lemma 5.2 permits the worst-message maximum before the final honest-key average. $\square$

The order of integration is the core of the result. Reordering it casually can condition an "independent" coefficient on a remainder that depends on it. Here the dependent remainders are removed by pointwise inequalities or offset maxima *before* primitive independent coins are averaged.

## A finite rational certificate

Choose $u=4/3$. Because CBD1 and the lifting alphabet are finite and $u$ is rational, the moment functions are rational finite sums. Direct exact evaluation gives

$$
\boxed{\quad
a=\frac{744601}{663552},\qquad
b=\frac{589}{576},\qquad
c=\frac{49}{48}.
\quad}
$$

The certified upper expression is

$$
B=256\cdot\frac73
\left(\frac{744601}{663552}\frac{589}{576}\right)^{1024}
\frac{49}{48}\left(\frac34\right)^{1024}.
$$

**Corollary 5.5 (Explicit upper bound).** The averaged worst-message failure probability satisfies

$$
\delta_{wc}\le B<2^{-212}.
$$

*Proof.* Theorem 5.4 gives the first inequality. Write the displayed rational as $B=N/D$ in lowest terms. The second inequality is the integer comparison $2^{212}N<D$, which is checked exactly by the accompanying short Python certificate and does not depend on floating-point evaluation. Its decimal presentation is $\log_2 B\approx-212.5264474805$. $\square$

The verification script recalculates $a,b,c$ from the primitive finite laws, constructs $B$ using rational arithmetic, and asserts the strict integer comparison. It does not try to estimate a $2^{-212}$ event by Monte Carlo simulation; doing so with ordinary experiment counts would be scientifically meaningless.

## A legal positive-probability failure event

A very small upper bound does not imply perfect correctness. An explicit honest randomness event establishes a strictly positive lower bound.

**Proposition 5.6 (Nonperfect correctness).** The same profile satisfies

$$
\boxed{\quad\delta_{wc}\ge2^{-8192}.\quad}
$$

*Proof.* Use the all-zero message. In key generation set every coefficient of $s$ to zero and every coefficient of $e_0$ to $-1$. In encryption set all coefficients of $e_1,e_2,\rho$ to zero; set all coefficients of the first three module components of $r$ to $1$ and all coefficients of its fourth component to $0$. These are all legal CBD1 and lift choices with positive probabilities; do not restrict $A$.

Because $As=0$, the public quotient and remainder are $b=1023$ and $r_0=3$ at every coefficient. Thus $v=4092$. Since $v$ is even and $e_2=0$, one has $r_2=0$; the entire product $(r_1-e_1)^Ts$ vanishes because $s=0$. Each active module component contributes $-4$ times the negacyclic product of the all-one polynomial with itself. The coefficient of degree zero in that product is $1-255=-254$. Therefore

$$
\varepsilon_0=3(-4)(-254)=3048.
$$

With $\mu_0=0$ this gives $z_0=3048$, which lies strictly between 1024 and 3072; the decoder incorrectly returns one.

The independent probability factors for the prescribed values have binary exponents: $1024$ for $s=0$, $2048$ for $e_0=-1$, $2048$ for $\rho=0$, $1024$ for $e_1=0$, $256$ for $e_2=0$, $1536$ for the first three all-one components of $r$, and $256$ for the last all-zero component. The sum is

$$
1024+2048+2048+1024+256+1536+256=8192.
$$

The event therefore has probability exactly $2^{-8192}$, without any condition on $A$. It causes failure for a fixed message; the maximum over messages is at least the failure probability of that message. Averaging over keys establishes the claim. $\square$

Combining Corollary 5.5 and Proposition 5.6 gives the two-sided statement

$$
\boxed{\qquad 2^{-8192}\le\delta_{wc}<2^{-212}.\qquad}
$$

The wide gap between the lower and upper exponents is not a contradiction. The lower bound comes from a deliberately contrived, extremely rare event and proves only nonperfect correctness. The upper bound is a conservative tail estimate protecting *all* honest messages on average over honestly generated keys.

## Consequences for multiple encryptions and individual keys

Let $d(K)$ be as defined above. For every $\theta>0$, Markov's inequality on the nonnegative variable $d(K)$ gives

$$
\Pr_K[d(K)>\theta]
\le\min\!\left(1,\frac{\delta_{wc}}{\theta}\right)
\le\min\!\left(1,\frac{2^{-212}}{\theta}\right).
$$

 The important quantifier is the **probability over honest key generation**: no worst-case-key bound follows.

For at most $h$ honest encryptions under one honestly generated key, with fresh independent encryption coins and possibly adaptively selected messages, a conditional union bound gives total failure probability at most $h\,d(K)$ for that key, capped by one. Averaging over keys yields

$$
\Pr[\text{at least one honest failure}]
\le\min(1,h\delta_{wc})
\le\min(1,h2^{-212}).
$$

The point is that adaptively selected messages do not require a new correctness theorem when $d(K)$ already maximizes over all messages.

These bounds still do not control how a chosen-ciphertext adversary selects malformed ciphertexts, queries the decapsulation oracle, or searches across coins for failure-triggering plaintexts. Chapters 8--10 will introduce the relevant games. In particular, average correctness, same-tape second moments, and failure finding under quantum oracle access are distinct mathematical objects.

## Proof obligations carried forward

The present chapter closes the **exact arithmetic and averaged correctness** layer for the frozen abstract PKE. It does not assert perfect correctness; it does not prove the hardness of the Module-LWE sources; and it does not transform an upper bound on honest failures into an IND-CCA or QROM theorem. The latter require precise oracle and adversary experiments and a separate reduction.

For the manuscript's reproducibility, the certificate accompanying these chapters verifies the finite sums and integer comparison. It is a certificate of a *finite arithmetic step*, not a machine-checked proof of the probability-conditioning argument. An independent examiner should still scrutinize the proof of Lemma 5.3 and the integration order of Theorem 5.4; a passing arithmetic script cannot validate them.

The bounds and witness follow [S1]. The common worst-message identity in Section 5.3 also uses the fixed tie rule.




# Joint-Distribution Hybrids and Conditional Passive Security

## The security question after correctness

Chapter 5 established a small but nonzero probability that honestly generated ciphertexts decrypt incorrectly. This is a correctness statement, not an indistinguishability statement. We now examine what an attacker can learn from a public quotient and from the two compressed ciphertext components. The proof objective is deliberately conditional: to turn an algorithm that distinguishes the ciphertext challenge into algorithms that distinguish **specified decisional source distributions**. The chapter does not assert that these sources are difficult.

The issue is not simply the number of secret coefficients. C1-K4 is a module construction in a negacyclic ring, with a single ephemeral secret $r$ reused in two ciphertext components. Moreover, encryption samples a fresh remainder $\rho$ and forms a lift $v=4b+\rho$ of the public quotient. These facts make a componentwise argument inadequate. An exact game hop must simulate the *joint* public-key and challenge-ciphertext law, including the conditional relation of $v$ to $b$.

Throughout this chapter, set

$$
R=\mathbb Z_{4096}[x]/(x^{256}+1),\qquad
\chi=\mathrm{CBD}_1,\qquad k=4,
$$

and understand $F_d$ as the coefficientwise map $a\mapsto\lfloor[a]_{4096}/d\rfloor$. All coefficient samples of $\chi$ are mutually independent when no dependency is indicated. We use the exact algorithms in Chapter 4: for uniform $A\in R^{4\times4}$, sample $s,e_0\leftarrow\chi^{1024}$ and set $b=F_4(As+e_0)$; given a message $\mu\in\{0,1\}^{256}$, sample independent $r,e_1\leftarrow\chi^{1024}$, $e_2\leftarrow\chi^{256}$ and $\rho\leftarrow U(\{0,1,2,3\}^{1024})$, then set

$$
\begin{aligned}
v&=4b+\rho,\\
c_1&=F_2(A^Tr+e_1),\\
c_2&=F_2(v^Tr+e_2+2048\mu).
\end{aligned}
$$

None of these formulas includes a replacement of an actual remainder by an independent ideal noise variable. That distinction remains essential even though the following proof uses the uniform-lift identity from Chapter 2.

## A precise IND-CPA convention

Consider the standard **single-challenge public-key IND-CPA experiment**. The challenger draws one honest key and publishes $pk=(A,b)$. An adversary $\mathcal A$, with public encryption and its own randomness available, returns two messages $\mu_0,\mu_1\in\{0,1\}^{256}$. The challenger draws a uniform bit $\beta$, independently encrypts $\mu_\beta$, and gives the resulting ciphertext to $\mathcal A$, which outputs $\widehat\beta$.

We fix the success-bias convention

$$
\operatorname{Adv}_{\mathrm{CPA}}(\mathcal A)
=\left|\Pr[\widehat\beta=\beta]-\frac12\right|.
$$

For a decisional source distinguisher $\mathcal B$, by contrast, $\epsilon(\mathcal B)$ denotes the **absolute acceptance-probability gap** between two source distributions. The two conventions differ in general. Our hybrid reductions output the CPA adversary's *success indicator*, which makes each adjacent difference a source gap without silently adding a factor of two.

The uniformity of $\beta$ and the equality of the message lengths are used in the final game. We make no assertion here for chosen-ciphertext queries, multi-user composition, quantum oracles, or secret-key leakage. Those are separate experiments.

## The two exact source distributions

For $j\in\{4,5\}$, define the decisional small-secret Module-LWE source

$$
\mathcal L_{4,j}:=(B,Bt+e),\qquad
\mathcal U_{4,j}:=(B,y),
$$

where

$$
B\leftarrow U(R^{j\times4}),\quad
t\leftarrow\chi^{1024},\quad
e\leftarrow\chi^{256j},\quad
y\leftarrow U(R^j).
$$

The same four-polynomial secret $t$ is shared by every row of $Bt+e$. The variable $y$ is independent of $B$. Define

$$
\epsilon_{4,j}(\mathcal B)=
\left|\Pr[\mathcal B(\mathcal L_{4,j})=1]
-\Pr[\mathcal B(\mathcal U_{4,j})=1]\right|.
$$

These are **exact** source problems for CBD1, the modulus $4096$, degree $256$, and module ranks shown. They are not freely interchangeable with coefficientwise iid LWE, Gaussian-secret Module-LWE, or a generic noisy Hankel sample. In particular, expanding $B$ into a scalar matrix introduces negacyclic structure. The first source has four rows and $1024$ coefficient observations; the second has five rows and $1280$ observations. The augmented running times and oracle accesses of the reduction algorithms matter when the hardness assumptions are instantiated.

## The public quotient cannot be replaced coefficient by coefficient

We first state a coupling that will be used only in the appropriate intermediate game.

**Lemma 6.1 (exact conditional uniform lift).** Suppose $A\leftarrow U(R^{4\times4})$ and $V\leftarrow U(R^4)$ are independent. Define $b=F_4(V)$ and $\rho=[V]_4$, coefficientwise. Then $b$ is uniform in $\{0,\ldots,1023\}^{1024}$, $\rho$ is uniform in $\{0,1,2,3\}^{1024}$, and $(A,b,\rho)$ has the product law. Conversely, if $(A,b,\rho)$ has this product law, the element $4b+\rho$ is uniform in $R^4$, and

$$
(A,F_4(V),V)\overset{d}= (A,b,4b+\rho).
$$

*Proof.* Each residue $v\in\{0,\ldots,4095\}$ has the unique Euclidean decomposition $v=4q+r$, with $0\le q<1024$ and $0\le r<4$. The map $v\leftrightarrow(q,r)$ is a bijection from a set of cardinality $4096$ to the Cartesian product of sets of cardinalities $1024$ and $4$. Uniformity therefore transports to the product law. Apply this bijection independently to all $1024$ coefficients. Independence from $A$ is preserved by deterministic transformation. $\square$

The qualification **conditional** refers to the public view. The lemma remains valid after message selection as long as the adversary only sees $(A,b)$ and independent randomness before the challenge. It does *not* imply that the actual lift $4F_4(As+e_0)+\rho$ in a real key is uniform, because the actual $b$ may be distinguishable from uniform. The reduction must replace the public quotient first; only then may it use the uniform lift in the challenge simulation.

## Game zero and the first transition

Write $\mathsf G_0$ for the real CPA experiment. The challenger generates $(A,b)$ honestly and encrypts with the common ephemeral secret and a fresh uniform lift remainder. Write $\mathsf G_1$ for the identical experiment except that $b$ is sampled uniformly and independently of $A$; thereafter every encryption uses that chosen public $b$ and a fresh independent lift remainder. In particular, the challenge ciphertext in $\mathsf G_1$ is still **honestly encrypted relative to this hybrid public key**.

**Lemma 6.2 (four-row hybrid).** There is a distinguisher $\mathcal B_4$ for the exact source $(\mathcal L_{4,4},\mathcal U_{4,4})$ for which

$$
\left|\Pr[\mathsf G_0:\widehat\beta=\beta]
-\Pr[\mathsf G_1:\widehat\beta=\beta]\right|
\le\epsilon_{4,4}(\mathcal B_4).
$$

*Proof.* Given $(B,z)$, the distinguisher sets $A=B$ and publishes $b=F_4(z)$. It runs $\mathcal A$ with its own random coins, draws $\beta$, and performs public encryption of the selected message using fresh $r,e_1,e_2,\rho$; no decryption secret is needed. It returns one exactly if $\widehat\beta=\beta$. If $z=Bt+e$ under the real source law, the public quotient has precisely the real key law with $s=t,e_0=e$; the challenge is generated publicly in the same fashion as $\mathsf G_0$. If $z=y$ is uniform independent of $B$, then $F_4(y)$ is uniform independent of $A$, yielding $\mathsf G_1$. The acceptance gap is the displayed game gap. $\square$

A frequent incorrect shortcut would replace $b$ by a uniform value **without** changing the dependent encryption law. That is not the game above. The distinguisher explicitly encrypts using the value of $b$ that was published, thereby preserving the distribution of the entire public view and challenge in each branch.

## The fifth row and the second transition

The transition from $\mathsf G_1$ to $\mathsf G_2$ replaces the source response underlying the challenge ciphertext by uniform ring values. Its simulator must retain one matrix component that plays the role of the lift $v$; otherwise it would destroy the correlation between the published quotient $b$ and the $c_2$ input.

**Lemma 6.3 (five-row hybrid).** There exists a distinguisher $\mathcal B_5$ for $(\mathcal L_{4,5},\mathcal U_{4,5})$ such that

$$
\left|\Pr[\mathsf G_1:\widehat\beta=\beta]
-\Pr[\mathsf G_2:\widehat\beta=\beta]\right|
\le\epsilon_{4,5}(\mathcal B_5),
$$

and in $\mathsf G_2$ the challenge bit is statistically hidden, even from a computationally unbounded adversary given the public view.

*Proof.* Parse the five-row source matrix and response as

$$
B=\begin{pmatrix}A^T\\ V^T\end{pmatrix},
\qquad z=(z_1,\ldots,z_4,z_5),
$$

where $A\in R^{4\times4}$ and $V\in R^4$. The simulator publishes $pk=(A,F_4(V))$, retains $V$, and waits for $\mathcal A$ to select its two messages. It samples $\beta$ and returns

$$
c_1=F_2(z_1,\ldots,z_4),\qquad
c_2=F_2(z_5+2048\mu_\beta).
$$

In the real-source branch $z_1,\ldots,z_4$ are the four coordinates of $A^Tr+e_1$, while $z_5=V^Tr+e_2$, with the *same* secret $r$ and fresh independent CBD1 errors. Since $V$ is initially uniform independently of $A$, Lemma 6.1 shows that publishing $F_4(V)$ and retaining $V$ is exactly the distribution of $b$ and its randomized lift in $\mathsf G_1$. This is a statement about the joint law, including the fact that the adversary selects its messages from the published key.

In the uniform-source branch all five response ring elements are jointly uniform and independent of $(A,V)$. Adding $2048\mu_\beta$ is a bijection on the last response ring element; its law remains uniform, jointly with the other four responses. Finally, $F_2$ maps each uniform modulus-$4096$ coefficient to a uniform value among $2048$ quotient values. Therefore the joint challenge distribution is independent of $\beta$, conditioned on the entire public view. The simulator again outputs the CPA success indicator. $\square$

The proof explains why **five** rows appear even though key generation uses four: the fifth row represents the scalar inner product with the lifted public vector. Treating these two ciphertext components as separately randomized samples with unrelated secrets would not preserve the scheme.

## Conditional IND-CPA theorem

**Theorem 6.4 (exact-source, full-matrix passive-security reduction).** For every classical CPA adversary $\mathcal A$ there exist classical source distinguishers $\mathcal B_4,\mathcal B_5$, constructed above and run at their respective augmented resources, such that

$$
\boxed{
\operatorname{Adv}_{\mathrm{CPA}}(\mathcal A)
\le \epsilon_{4,4}(\mathcal B_4)+\epsilon_{4,5}(\mathcal B_5).}
$$

*Proof.* Insert the three games $\mathsf G_0,\mathsf G_1,\mathsf G_2$. By Lemmas 6.2 and 6.3 and the triangle inequality, the difference between the first and last success probabilities is bounded by the sum of the two source gaps. The last game's challenge distribution is independent of $\beta$, and therefore its success probability is $1/2$. $\square$

This is a **reduction**, not a numerical security guarantee. The simulation identities hold under the stated sampling laws. A small passive distinguishing advantage follows if both source gaps are small at the respective augmented budgets. A proof of hardness for those particular small-secret distributions, or an assumption made explicit at a desired security budget, is an additional premise. No single sample is implicitly expanded into arbitrary independent Module-LWE oracle samples: each source distinguisher consumes its one supplied matrix-response instance.

## Seeded matrices: ideal oracle and classical conflict event

The compact public key does not transmit all sixteen ring entries of the full matrix. A 32-byte seed $\sigma_A$ is expanded through a matrix oracle $H_A$ with domain-separated and injectively encoded addresses $(\sigma_A,i,j)$ for $0\le i,j<4$. A response consists of $512$ uniform bytes, parsed as $256$ little-endian 16-bit words and reduced modulo $4096$. The low twelve bits are uniform; the upper four bits are discarded when a ring coefficient is formed.

For a uniformly sampled independent matrix $A$, programming an ideal-oracle response consistent with $A$ requires sampling the unused upper four bits of every raw coefficient independently. This ensures that an adversary allowed to query $H_A$ sees the full correct oracle distribution, not merely the correct reduced matrix. The programmed oracle must be consistent across repeated queries.

Suppose a *classical* adversary or outer simulator made at most $q_A^{\mathrm{pre}}$ distinct relevant oracle queries before a fresh uniform seed was chosen. The event that one of those queries targets the chosen seed has probability at most

$$
\epsilon_A=\min\{1,q_A^{\mathrm{pre}}2^{-256}\}.
$$

The two source branches use the same pre-query simulation and the same abort rule. The pre-seed transcript and the independently drawn seed can be coupled identically across branches. On the common conflict event the difference of their conditional success probabilities has absolute value at most one, so discarding it costs at most one conflict probability per transition. The safe simulation aborts on that event; it does not resample seeds until a conflict disappears. Outside the conflict, fresh addresses can be patched consistently to the source-supplied matrix. Applying the allowance to each source transition yields the classical ideal-matrix-oracle bound

$$
\boxed{
\operatorname{Adv}_{\mathrm{CPA}}^{\mathrm{seed}}(\mathcal A)
\le\epsilon_{4,4}(\mathcal B_4)+\epsilon_{4,5}(\mathcal B_5)
+2\epsilon_A.}
$$

Here the superscript *seed* denotes the ideal oracle construction, not the security of any specific SHAKE256 implementation. In a strictly key-first experiment in which the oracle is freshly initialized and no prior matrix-oracle query exists, the classical pre-query conflict term is zero. A coherent quantum query to a pre-existing oracle would require a different argument; this chapter supplies none. The quantum source simulation and coherent oracle programming are deliberately reserved for Chapter 9.

**Lemma 6.5 (encoding arithmetic).** The seeded public-key payload contains $1+32+1024\cdot10/8=1313$ bytes, and the ciphertext payload contains $(1024+256)\cdot11/8=1760$ bytes.

*Proof.* The public quotient $b$ has $1024$ coefficients each in $\{0,\ldots,1023\}$ and thus exactly ten bits each. The ciphertext has $1280$ coefficients each in $\{0,\ldots,2047\}$ and thus exactly eleven bits each. Both totals are byte-aligned. Adding one profile byte and a 32-byte matrix seed gives the public payload. $\square$

These byte counts concern canonical payloads. They do not include transport wrappers, a full KEM secret-key serialization, or a confirmation tag. The seed is not a lossless compression of an arbitrary full uniform matrix: the ideal oracle changes the specification of how a matrix is generated from a seed, and a concrete expander requires its own analysis.

## Structural caveats and a useful modulo-two diagnostic

Expanding the module multiplication into coefficients yields signed negacyclic blocks; the ordinary iid-scalar-LWE hypothesis cannot be imported by simply equating the coefficient dimension with $1024$. Similarly, interpreting a rounded public quotient as an independent LWE sample changes its law. It is scientifically more precise to give the exact source assumption a name and test it than to conceal it beneath a broad hardness slogan.

There is one elementary but informative observation at the lowest binary layer. If $E\sim\mathrm{CBD}_1$ then $E\bmod2$ is a fair bit, because $\Pr[E\text{ odd}]=1/4+1/4=1/2$. Consequently the parity of a raw Module-LWE response $Bt+e$, conditioned on $B,t$, is a uniform vector over $\mathbb F_2$. This prevents a distinguisher that sees **only** the raw response parity from learning the secret in that conditional view. It does not say that the full response modulo $4,8,\ldots,4096$ is uniform or that a quotient transcript contains no exploitable correlations.

The module ring is also singular-looking over $\mathbb F_2$: $x^{256}+1=(x+1)^{256}$. For a uniformly random $4\times4$ matrix over $\mathbb F_2[x]/((x+1)^{256})$, invertibility is determined by the determinant at $x=1$. Hence the fraction of invertible matrices is

$$
\frac{|\operatorname{GL}(4,2)|}{2^{16}}
=\frac{(16-1)(16-2)(16-4)(16-8)}{65536}
=\frac{20160}{65536}.
$$

That fraction is an exact structural fact and a check on simulations. It is **not** a secret-recovery attack in the presence of CBD1 errors or quotient compression. Chapter 7 will distinguish diagnostics of this kind from attack cost estimates.

## What the reduction does not establish

The chapter's proven content is the sequence of distribution-preserving hybrid simulations. It does not establish: unconditional decisional Module-LWE hardness at $(4,4)$ or $(4,5)$; a reduction from worst-case lattices to these exact parameters; the security of a specific non-ideal matrix expander; indistinguishability under decryption queries; quantum chosen-ciphertext security; or any fixed-bit security level. Attack-model numbers must not be substituted for the source gaps in Theorem 6.4.

The source simulations follow [S1]. The full-matrix theorem and the classical seed-conflict allowance concern distinct public interfaces. Chapter 7 reports the estimator budgets separately.

# Structured Cryptanalysis, Cost Models, and Reproducible Evidence

## Why a reduction is not an attack estimate

The conditional theorem in Chapter 6 relates CPA distinguishing advantage to two explicitly specified Module-LWE source distinguishing advantages. It does not determine the numerical difficulty of either source. Conversely, a cryptanalytic model may predict the resource cost of one family of attacks without bounding the best attack that might exist. A responsible concrete-security discussion must therefore keep two different statements apart:

1. A **reduction statement** of the form $\operatorname{Adv}_{\mathrm{CPA}}\le\epsilon_{4,4}+\epsilon_{4,5}+2\epsilon_A$, whose value depends on assumptions about all relevant source distinguishers at an augmented resource budget.
2. An **attack-estimation statement** that a specified optimization procedure found certain parameters yielding a specified modeled workload $T$ for a named technique.

The inequality in the first statement is a security implication once its premises are supplied; the number in the second is a possible attack cost under a selected model. It is not the contrapositive of the first. A model predicting $T=2^{206.21}$ does not imply that every adversary below that cost has negligible success probability.

The present chapter records the attack evidence associated with the fixed C1-K4 instance, explains how it was calculated, and marks the boundary between an executable arithmetic check, a model evaluation, an actual attack, and a source-hardness assumption. The numbers in the tables are inherited from the pinned Article 1 supplementary dossier [S1]. We do not relabel archived Sage logs as new executions or declare exhaustive coverage where only local search was performed.

## The target geometry and observation budgets

Recall the exact source families

$$
B\in R^{j\times4},\quad
R=\mathbb Z_{4096}[x]/(x^{256}+1),\quad
j\in\{4,5\},\quad t,e\sim\mathrm{CBD}_1.
$$

Each row provides $256$ coefficient observations, so the two decision problems contain respectively $1024$ and $1280$ coefficient observations of a $1024$-coefficient secret. A coefficient embedding retains the negacyclic constraints. It would be wrong to say that the resulting coefficient matrix consists of $1024\times1024$ or $1280\times1024$ independent scalar entries. The set of structured matrices has a different distribution and different exploitable symmetries from the unstructured ensemble.

For either source a basic attacker can discard rows or individual observations. This yields a monotonicity observation: **access to more observations cannot make the *optimal* achievable distinguishing task harder for an attacker**, because the attacker may ignore the extras. It says nothing about how a particular restricted estimator reacts to a changed value of its internal sample-count parameter. Such an estimator may use a suboptimal regime or fail to search a profitable choice of samples.

A second distinction concerns the domain of advantage. Passive security of the PKE is reduced to distinguishing the Module-LWE *source samples*, while the KEM transforms involve oracle queries and possibly decryption failures. The attack numbers here target the source-like task in specified classical cost models. They are not directly values for the final IND-CCA or QROM advantage functions.

## Inventory of the reported model values

The corrected companion dossier preserves four reference evaluations:

| Model and configuration | Reported $\log_2 T$ | Epistemic status |
|:--|--:|:--|
| Classical primal, uSVP baseline | $227.176$ | Estimated cost at the model's selected point |
| Classical plain dual baseline | $235.060$ | Estimated cost at the model's selected point |
| MATZOV-style dual-hybrid, expanded local search | $213.656698256$ | Minimum found in the documented search domain |
| Rotation-aware coded-dual with CBD1 and corrected bracket | $206.210581344$ | Archived full Sage cost at one documented point |

All four entries are **model-based log-work estimates**, not observed times for a complete attack on C1-K4 and not proven lower bounds. Even taking the smallest *reported* entry does not identify the best conceivable classical attack. The source of the corrected coded-dual result is a pinned Sage adapter and its execution record, not a general theorem that coded-dual optimizes to that cost.

The raw CBD1 secret and error distribution has

$$
\Pr[X=-1]=\Pr[X=1]=\tfrac14,
\quad \Pr[X=0]=\tfrac12,\quad
\mathbb E X=0,\quad\operatorname{Var}(X)=\tfrac12.
$$

The corresponding standard deviation $\sqrt{1/2}$, the source modulus $4096$, and the secret coefficient dimension $1024$ are inputs to the chosen lattice-cost models. A different norm convention, a different embedding, or a revised sieving-cost hypothesis can move the estimates. A change of such conventions is a change of the attack model, not a change to the frozen C1-K4 PKE itself.

## The MATZOV-style dual-hybrid evaluation

The MATZOV-style adaptation represented in the supplemental program combines a lattice-reduction cost with guessing, enumeration, and Fourier-analysis terms. The exact objective is defined by its pinned implementation and parameter conventions [R3, S1]; it is not inferred here from an informal asymptotic expression. Its inputs include a classical reduction exponent $0.292$, a lattice block size $\beta$, an integer precision parameter $p$, enumeration dimension $k_{\mathrm{enum}}$, Fourier dimension $k_{\mathrm{fft}}$, the resulting lattice dimension $k_{\mathrm{lat}}$, and the number of input observations $m$.

The documented program performs the following deterministic procedure, independently for the two candidate sample counts $m=1024$ and $m=1280$:

1. Scan $\beta=700,701,\ldots,900$ at the initial tuple $(p,k_{\mathrm{enum}},k_{\mathrm{fft}})=(3,11,113)$.
2. Starting from the best such point, perform four rounds of coordinate descent. During each round the current best point determines a local window $\beta\pm40$, $k_{\mathrm{enum}}\pm20$, and $k_{\mathrm{fft}}\pm25$, clipped to legal limits; also scan the integer $p\in\{2,\ldots,10\}$.
3. Evaluate each changed coordinate with other coordinates fixed at their values at the start of that coordinate sweep, update the best value, and report the final tuple and objective.

The reported $m=1024$ point is

$$
(\beta,p,k_{\mathrm{enum}},k_{\mathrm{fft}},k_{\mathrm{lat}},d)
=(731,3,6,114,904,1928),
$$

with

$$
\boxed{\log_2 T_{\mathrm{MATZOV}}=213.656698255976\ldots.}
$$

With $m=1280$ fixed, the second run gives
$$
\log_2 T=221.970828625273,\qquad
(\beta,p,k_{\mathrm{enum}},k_{\mathrm{fft}},k_{\mathrm{lat}},d)=(760,3,10,113,901,2181).
$$
 Discarding the fifth row recovers the four-row marginal. The earlier value $220.538\ldots$ came from a narrower search whose parameter boundary was accidentally decisive. It remains useful as a provenance record, but it must not be advertised as the best known point in the same model once a lower admissible point has been found. The refined result is also not an optimum proof: coordinate descent need not find the global minimum, and the $m=1280$ case was not exhaustively optimized over every algorithmic choice.

**Proposition 7.1 (a reproducible result is search-relative).** Let $\mathcal S$ be the finite set of parameter combinations actually evaluated by a program, and let $C:\mathcal S\to\mathbb R$ be its deterministic cost function. Finding $\widehat\theta\in\mathcal S$ with $C(\widehat\theta)=c$ proves only that

$$
\min_{\theta\in\mathcal S}C(\theta)\le c.
$$

Even a certified minimizer over $\mathcal S$ cannot yield a lower bound for the optimum over a larger attack family $\mathcal T\supseteq\mathcal S$.

*Proof.* The first inequality follows because $\widehat\theta\in\mathcal S$. Enlarging the search space can only decrease or preserve its minimum. No relation to a cryptographic security lower bound follows unless a theorem says all attackers are represented and correctly costed by $C$, which is not assumed. $\square$

This modest proposition is worth stating because the word “best” is sometimes used ambiguously. In this thesis it always means **best among the tested and documented alternatives**, unless an explicit optimality theorem is present.

## Rotation-aware coded dual and the critical search bracket

The coded-dual cost model originates in [R8]. The rotation-aware calculation of [R6], full version (a), is based on a public Sage code snapshot on the branch `eprint-2026-279` at commit `7cce6a5572420004da31be58f26954cedfc885d0` [R7]. The accompanying adapter is pinned to that snapshot, whose threshold-integral argument already uses $\beta_1$ in the affected call. The two adaptations below are additional profile-specific changes. The reproduced point in the historical audit uses a model success threshold $0.6$, a separate false-candidate budget, and the rotation-aware calculation at

$$
(m,n_{\mathrm{enu}},n_{\mathrm{fft}},k_{\mathrm{fft}},n_{\mathrm{lat}},\beta_0,\beta_1)
=(770,17,135,14,872,701,701).
$$

This procedure requires **two distinct adaptations**, and provenance is lost if only one is reported.

First, the zero mass of the coefficient distribution must be set to $p_0=\Pr[X=0]=1/2$ for CBD1; a code path designed for another noise law cannot be assumed correct for CBD1. Second, the binary search for an integer sample/repetition parameter $R$ must begin below the success-probability threshold. The historical upstream bracket chose $R_{\min}=262144$, but the reported success probability there was already approximately $0.795804$, which exceeds the target $0.6$. Such a lower endpoint does not bracket the crossing and therefore cannot justify a claimed least admissible $R$.

With the revised lower endpoint $R_{\min}=1$, the archived checked values are

$$
\eta(137828)=0.599998202351\ldots<0.6,
$$

$$
\eta(137829)=0.600000555101\ldots\ge0.6.
$$

Thus the reported threshold-crossing candidate is $R=137829$, giving

$$
\boxed{\log_2 T_{\mathrm{coded\ dual}}=206.21058134400172.}
$$

For this tuple the threshold function has a direct monotonicity check. If $Z\sim\mathrm{Bin}(1024,1/2)$, let $h_z=\binom z{17}/\binom{1024}{17}$, with $h_z=0$ for $z<17$. The model averages $1-(1-h_Z)^R$. Its increment is $\mathbb E[h_Z(1-h_Z)^R]>0$ for finite positive integer $R$, since $0<h_Z<1$ has positive probability. The adjacent probabilities therefore locate the first admissible integer for this bracket at this tuple. The full cost $206.210581344\ldots$ comes from the archived corrected Sage record. The earlier log at $R=262145$ reports $206.558064614\ldots$. The independent Python check reproduces the threshold probabilities. Global cost optimization remains open.

The public coded-dual repository also records a separate parameter mismatch in an inherited rotation utility. A reproduction must therefore pin the source commit and record all deviations from that code. This thesis retains the two modifications explicitly made for the C1-K4 adapter and does not silently import unrelated patches from later branches. A future comparison should investigate how each additional patch affects the same reported point before treating the resulting figures as directly comparable.

## A five-row source is not analyzed by a four-row estimate

The five-row source has $1280$ coefficient observations. The scalar count $m=770$ fits within four rows. To transfer a particular partial-row implementation one must also show that the chosen observations support the modeled rotations and shared preprocessing. The scalar count alone does not give that construction. Projection of a full five-row sample onto its first four rows preserves the four-row source law exactly.

To avoid confusion, distinguish the following statements:

- An algorithm restricted to $770$ observations can be run against either source as long as it can select the relevant observations.
- The same algorithm, when extended or reoptimized for $1280$ observations, might have a different cost; no exhaustive such search is reported here.
- The optimal feasible attack cost with $1280$ observations cannot be larger than the optimal cost when an attacker is forced to use only $1024$ observations, because ignoring observations is allowed.

These statements are compatible. The last is about all available algorithms; the first two are about a constrained cost-model search. This separation prevents an estimator's local result from being mistaken for a general property of the source.

## Local rings and response parity

The reduction of the modulus-$4096$ ring modulo $2$ is

$$
R/2R\simeq\mathbb F_2[x]/(x^{256}+1)
=\mathbb F_2[x]/((x+1)^{256}).
$$

This is a finite local ring with unique maximal ideal generated by $x+1$. A $4\times4$ matrix over it is invertible exactly when its reduction modulo that maximal ideal is invertible, equivalently when the binary matrix obtained by evaluating each polynomial entry at $x=1$ lies in $\mathrm{GL}(4,2)$.

**Proposition 7.2 (exact binary-matrix invertibility frequency).** If $A$ is a uniformly random $4\times4$ matrix over $R$, then the coefficientwise reduction modulo $2$, followed by evaluation at $x=1$, is a uniform binary $4\times4$ matrix, and

$$
\Pr[A\text{ is invertible modulo }2]
=\frac{20160}{65536}=0.3076171875.
$$

*Proof.* Uniform ring coefficients reduce to independent uniform binary coefficients; evaluation at $1$ is a surjective linear functional on each polynomial, hence its value is uniform. The number of invertible $4\times4$ binary matrices is

$$
|\mathrm{GL}(4,2)|
=(2^4-1)(2^4-2)(2^4-2^2)(2^4-2^3)=20160.
$$

There are $2^{16}=65536$ binary matrices. Invertibility over a finite local ring is equivalent to invertibility modulo its maximal ideal, completing the proof. $\square$

For independent CBD1 errors, parity is also uniform: $\Pr[E\bmod2=0]=\Pr[E\bmod2=1]=1/2$. Hence if a distinguisher views *only* $(B,(Bt+e)\bmod2)$, its response parity is uniformly masked even after $B,t$ have been fixed. This does not establish uniformity of the response modulo $4$ or of compressed quotients. Carries from higher residues still depend on the structured products and errors.

A second simple diagnostic is secret sparsity. The number of nonzero CBD1 coefficients in a $1024$-coefficient secret has the distribution

$$
W\sim\operatorname{Binomial}(1024,1/2).
$$

The exact probability of an unusually light secret can be computed by summing binomial coefficients. Such a tail estimate helps calibrate one particular brute-force strategy; it cannot exclude hybrid guessing of selected coordinates, adaptive sparse subspace attacks, or algebraic attacks. The higher-modulus relationships, not the uniform parity alone, must be studied to assess those possibilities.

## Resource accounting and the invalid “effective bits” subtraction

The supplementary word-operation schedule counts $63{,}200$ abstract operations for re-encryption, bounded by $2^{16}$, plus a fixed precomputation allowance $2^{17}$. Its bookkeeping yields

$$
\kappa\le\bar\kappa=1+4\cdot2^{16}+2^{17}=393217,
\qquad\log_2(12\bar\kappa)=22.1699286704\ldots.
$$

An auxiliary number-theoretic-transform prime is used only to implement exact convolution; the cryptosystem modulus remains $4096$. These model counts are not measured cycle counts and do not certify the cost of any concrete or quantum simulation. Different arithmetic units, memory models and oracle-query conventions can make the reduction budget incomparable with the work units used by a lattice-attack estimator.

It is therefore invalid to form $206.2106-22.1699\approx184.04$ and call the difference a *proved security level*. To turn such a subtraction into a formal implication one would need, at minimum, a bound on every relevant source distinguisher's success probability as a function of work, an explicit simulator whose work can be expressed in the **same** units, and rigorous control of all other game losses. The first ingredient does not follow from an attack estimator, and the remaining ingredients are not established by the archived word schedule alone.

The correct conclusion is weaker and more useful: **the current attacks give candidate points for further cryptanalysis and help prioritize parameters and audit questions**. No numerical certification of $128$-bit classical or postquantum security follows.

## Evidence classes and reproduction protocol

For clarity, this chapter uses the following independent evidence classes.

| Evidence class | What it certifies | What it does not certify |
|:--|:--|:--|
| Exact arithmetic test | Equality or inequality for finite tested inputs | A universal probability theorem unless the enumeration is exhaustive |
| Rational certificate | A specified integer/rational comparison | The correctness of preceding probabilistic conditioning |
| Deterministic parameter search | Evaluated objective at documented candidate points | Global optimality or completeness over all algorithms |
| Sage reproduction of a coded-dual model | Model output for one fixed software and code patch set | A realized cryptanalytic attack |
| Actual distinguishing experiment | A measured advantage on sampled test inputs | A worst-case security lower bound |
| Computational hardness premise | Conditional input to a reduction | Its own unconditional validity |

A reproducible release should provide its versioned source commit, hash manifest, command line, software environment, and *freshly generated* output data. If a verification script reads a JSON file, the pipeline must regenerate that file before comparing it to expected values; otherwise a stale output can falsely make the test appear to pass. Historical logs are retained as historical logs, never renamed as new experiments. This discipline is particularly important because the pure-Python MATZOV run and the Sage coded-dual adapter have different executables and dependencies.

The original chapter verifier embeds reference values for the two model costs. Fresh runs in the reproducibility supplement evaluate the expanded MATZOV search and the coded-dual threshold probabilities. Recomputing the complete coded-dual cost requires the pinned Sage code and adapter.

## Cryptanalytic obligations still open

The present evidence does not close a global coded-dual optimizer over the full observation budgets, an exhaustive structured attack search, a proof ruling out $2$-adic or boolean-algebraic distinguishers, a worst-case-to-average-case reduction for these exact source distributions, or a complete quantum cryptanalytic cost analysis. The structure of $R/2R$ means that algebraic checks deserve attention, but does not itself furnish an attack.

The expanded MATZOV values were regenerated from [S1]'s supplementary code. The coded-dual cost retains its archived Sage provenance. The model descriptions are [R3], [R6] and [R8], with the pinned implementation [R7].

# Classical Fujisaki–Okamoto Transformation with Imperfect Correctness

## The missing bridge between passive and active security

An IND-CPA reduction is not enough to explain the security of a key-encapsulation mechanism (KEM) that exposes a decapsulation oracle. In C1-K4, the gap is especially important because Chapter 5 proved that **honest decryption is not perfectly correct**. This chapter fixes the classical random-oracle wrapper, explains how its randomized encryption becomes deterministic once an oracle is fixed, and reconstructs the conservative game accounting recorded in the corrected companion manuscript [S1]. The chapter makes clear which steps are elementary, which use a particular version of the modular Fujisaki–Okamoto (FO) analysis, and which remain conditional on the exact game interface.

The historical FO idea is to derive encryption coins from a message through a random oracle, encrypt that message, and then check by re-encryption during decryption. Implicit rejection returns a pseudorandom-looking key rather than exposing the internal reject symbol. The resulting proof is not a single algebraic identity: it is a sequence of transitions among a deterministic-encryption game, a plaintext-checking game, and a key-indistinguishability game. The corrected modular analysis of Hofheinz, Hövelmanns and Kiltz [R1] makes the role of nonperfect correctness explicit. Its errata and concrete query conventions cannot be bypassed by simply writing “FO security follows”.

This chapter uses **classical, not superposition, random-oracle queries**. One honestly generated key is considered. The main numerical envelope below is scoped to a **challenge-first** experiment, in which the challenger supplies the challenge before any additional attacker oracle interactions that could depend on its unknown message. A broader experiment with pre-challenge queries requires distinct accounting and is not silently included. Quantum-accessible oracles, the key-first coherent model, and its different bounds belong to Chapter 9.

## The complete compact KEM wrapper

Let $\mathcal M=\{0,1\}^{256}$ with $M=|\mathcal M|=2^{256}$. In addition to the ideal matrix oracle $H_A$ of Chapter 6, take two independent classical random oracles with injectively and unambiguously encoded tuple inputs:

$$
G:\bigl(pk,m\bigr)\longmapsto \{0,1\}^{8\cdot2816},
\qquad
H:\bigl(pk,m,c\bigr)\longmapsto\{0,1\}^{256}.
$$

Here $c$ in $H$ ranges over all finite byte strings, including malformed inputs. The three oracle families $H_A,G,H$ have disjoint domain identifiers. The outputs of $G$ are interpreted as uniform encryption tapes according to Chapter 4: $2304$ bytes supply independent CBD1 coefficients by subtracting the two low bits of each byte, and $512$ bytes supply independent two-bit lift coefficients from nibbles. The leftover bits do not need to be invertible. Correctness and distributional arguments depend on the **pushforward distribution** of the tape, not on an injective tape-to-coins map.

For fixed public key and fixed oracle $G$, define the deterministic encryption function

$$
\operatorname{Enc}_1(pk,m)
=\operatorname{Enc}\bigl(pk,m;G(pk,m)\bigr).
$$

The classical wrapper consists of the following three procedures.

**Key generation.** Generate the base C1-K4 PKE key pair as in Chapter 4. Independently sample a uniform $256$-bit fallback secret $z_0$. Retain the base decryption secret, the necessary public key data for re-encryption, and $z_0$ in the KEM secret key. Publish the $1313$-byte public-key payload, with its ideal seed/matrix-oracle interpretation.

**Encapsulation.** Choose $m\leftarrow U(\mathcal M)$. Compute

$$
c=\operatorname{Enc}_1(pk,m),\qquad K=H(pk,m,c),
$$

and return $(c,K)$. The ciphertext payload is exactly $1760$ bytes. No confirmation tag is appended in this frozen variant.

**Decapsulation.** Reject malformed byte encodings at the internal PKE parsing interface. For a well-formed $c$, calculate $m'=\operatorname{Dec}(sk,c)$, then compute $c'=\operatorname{Enc}_1(pk,m')$ and check exact equality $c'=c$ over the complete canonical byte string. Set

$$
\operatorname{Decap}(sk,c)=
\begin{cases}
H(pk,m',c),& \text{if parsing, decryption and exact re-encryption succeed},\\
H(pk,z_0,c),& \text{otherwise}.
\end{cases}
$$

Repeated rejected ciphertext strings return the same fallback key because $z_0$ is fixed per key and $H$ is a function. Good and fallback branches use **the same** $H$ family with the stated middle argument; adding a branch marker would define a different scheme. The internal decryption may emit a reject symbol, but the external KEM interface always returns a $256$-bit key in this ideal functionality. The specification has not yet frozen a constant-time production implementation or a complete interoperable secret-key format.

An important asymmetry is visible: public encapsulation can be carried out without knowing the PKE decryption secret, while valid decapsulation both decrypts and checks a full re-encryption. The equality check is part of the mathematical construction, not an optional defense.

## Canonical parsing and rigidity

A KEM proof must say what happens to strings that do not represent valid ciphertexts. Here each of the $1280$ ciphertext quotient coefficients lies in $\{0,\ldots,2047\}$ and is stored in an eleven-bit fixed-width field. Because the total bit length is byte-aligned, the canonical encoder has a unique byte representation for every coefficient vector. Every string of length 1760 bytes parses. All other lengths enter fallback with the original raw string as the final $H$ argument. The profile field belongs to the public key. No accepted alternative representation of the same coefficient vector is permitted.

Define a reference deterministic decryption $\operatorname{Dec}_1$ that returns a message only if its parsed base-PKE decryption followed by deterministic re-encryption exactly reproduces the submitted ciphertext; otherwise it returns $\bot$.

**Lemma 8.1 (rigidity).** For every fixed $(pk,sk)$ and fixed function $G$,

$$
\operatorname{Dec}_1(sk,c)=m\ne\bot
\quad\Longrightarrow\quad
c=\operatorname{Enc}_1(pk,m).
$$

*Proof.* The defining non-rejection branch of $\operatorname{Dec}_1$ performs the equality test $\operatorname{Enc}_1(pk,m)=c$. Hence returning $m$ implies the equality. No assumption on injectivity of the random tape, noiseless decryption, or the uniqueness of preimages of the encryption function is needed. $\square$

Rigidity is a property of **accepted** ciphertexts, not of all possible encryptions. An honest ciphertext can fail to decrypt to its generating message, and in that event the equality-check reference decryption may reject it. The lemma therefore does not imply perfect correctness. It also does not establish constant-time behavior: an implementation can be mathematically rigid while leaking a timing or fault side channel.

## Correctness parameters and the right order of quantifiers

For an honestly generated base PKE key pair $K=(pk,sk)$, let

$$
d(K)=\max_{m\in\mathcal M}\Pr_\omega\bigl[
\operatorname{Dec}(sk,\operatorname{Enc}(pk,m;\omega))\ne m\mid K\bigr],
\qquad
\delta_{wc}=\mathbb E_K d(K).
$$

The maximum is taken **before** the expectation over honest keys. Chapter 5 proves

$$
\boxed{2^{-8192}\le\delta_{wc}<2^{-212}.}
$$

The upper bound remains valid for each fixed message after key averaging, but it is not a guarantee that $d(K)<2^{-212}$ for every key. The lower bound proves that the base scheme is not perfectly correct. A correct security proof must retain failure terms; it must not replace $\delta_{wc}$ by zero because the numerical upper bound is tiny.

If a freshly sampled uniform encryption tape is assigned to a distinct oracle input $(pk,m)$, conditioning on the key and any transcript before that input is first evaluated leaves the tape independent and honest, **provided no earlier event has already constrained that input**. This is the basis for charging failure probabilities at fresh tape-oracle inputs in a classical game hop. It is not a license to treat the outputs at previously queried inputs as new independent samples. Such details are why a concrete HHK theorem and its oracle-query conventions matter.

## What a classical IND-CCA experiment asks

An IND-CCA KEM challenger generates the public and secret keys and supplies the adversary with the public key. It obtains a valid encapsulation $(c^*,K_{\mathrm{real}})$ and samples an independent uniform $K_{\mathrm{rand}}\in\{0,1\}^{256}$. A secret bit determines whether the adversary receives $K_{\mathrm{real}}$ or $K_{\mathrm{rand}}$ together with $c^*$. It can make permitted decapsulation queries $c\ne c^*$ and classical random-oracle queries. Its output is a guess for the secret bit.

Our success-bias convention is

$$
\operatorname{Adv}_{\mathrm{CCA}}(\mathcal A)
=\left|\Pr[\widehat\beta=\beta]-\frac12\right|.
$$

There are two notable types of bad event. First, the adversary may learn or correctly predict the unknown encapsulation message $m$ and query the relevant key oracle on $(pk,m,c^*)$. Second, it may exploit a decryption failure to distinguish an expected re-encryption relationship from the true decapsulation response. These events are not the same: the first is a message-search or oracle-guessing event; the second concerns the loss of a simulation that assumes the internal decapsulated value agrees with the original message.

The corrected modular FO analysis separates these mechanisms using intermediate one-way and plaintext-checking interfaces. In this chapter we expose the resource and probability parameters of that route instead of citing a slogan.

## Deterministic encryption, plaintext checking, and game transitions

The first conceptual transformation, often denoted $T$, derives a deterministic PKE from an arbitrary randomized PKE by selecting randomness as $G(pk,m)$. The second, an implicit-rejection transformation often written $U^{\not\perp}$, derives the KEM key from the message and ciphertext and assigns a fallback value to rejected ciphertexts. To use modular FO results, the cryptographic game interfaces must match not only the algebra but also the challenge timing and the random-oracle query behavior.

Fix the challenge key prefix. All other prefixes of each independent random oracle have independent tables. In HHK's corrected full version [R1], Definition 2.1 and Theorem 3.2 apply to the deterministic transform $T$. The plaintext-checking oracle on $(m,c)$ with $m\in\mathcal M$ returns $[\operatorname{Dec}_1(sk,c)=m]$. One may test $c^*$ with this oracle. There is no separate ciphertext-validity oracle ($q_V=0$), so the validity-oracle guessing term in the proof is zero. In the KEM experiment the decapsulation oracle excludes $c^*$. The relevant implicit-rejection transformation is Theorem 3.4, $U^{\not\perp}$, with the ciphertext-dependent key hash. The bounds below use our success-bias convention for CPA and CCA and direct recovery probability for OW-PCA.

## Query allowances and a conservative envelope

Let $q_G,q_H,q_D$ be the explicit classical upper bounds for the adversary's $G$-queries, $H$-queries, and decapsulation queries in the described challenge-first experiment. Define conservative simulation budgets

$$
Q_G=q_G+q_D+1,
\qquad
Q_H=q_H+2q_D+1,
\qquad
Q_{RO}=Q_G+Q_H=q_G+q_H+3q_D+2.
$$

These are deterministic upper budgets. In the implicit-rejection simulation use at most one plaintext check per target-key hash query, hence $q_P\le Q_H$. Let $L\le Q_G+Q_H$ bound the number of distinct target-key messages on the public simulation list, including checks that consult $G$. A private challenge tape occupies a budget slot but does not insert the hidden message into this list. Repeated inputs reuse cached values.

**From deterministic plaintext checking to two CPA tests.** Choose independent uniform candidates $m_0,m_1$. Give them to a base CPA challenger which encrypts $m_b$ using private honest coins. Supply a fresh independent public $G$ table for the simulated transcript. Answer each $(m,c)$ check by comparing $c$ with public deterministic re-encryption, recording every queried $m$ for the target key. Couple the hidden real $G(pk,m_b)$ to the challenge tape until the public list first hits $m_b$. Before a hit, any erroneous plaintext-check answer requires base decryption failure on a fresh listed tape: if the equality test accepts and base decryption gives $m$, both checks accept; if the equality fails the rigid reference check cannot accept $m$. Fix the expanded key (including its matrix in the seeded game) and the public matrix-oracle environment. At first assignment each other $G$ value is independent of the transcript; the conditional error is at most $d(K)$. Charging once per new input and averaging over keys costs at most $L\delta$, for $\delta=\delta_{wc}$. This accounts for the corrected $q_G+q_P$ allowance.

For the list test guess the index of the unique listed candidate and break ties by a fair coin. Twice its signed success bias is the difference between inclusion probabilities of the encrypted and unused candidates. The latter is independent of the simulated transcript, with inclusion probability at most $L/M$. Thus the hit probability is at most $2\beta_D+L/M$. For recovery, guess zero if the output equals $m_0$, and one otherwise. By symmetry of the two hidden candidate labels, twice the signed bias is recovery probability minus the probability of matching the unused candidate, at most $1/M$ for the latter. Both tests run the identical simulation without revealing either candidate to the simulated adversary. Here $\beta_D,\beta_E$ denote their absolute CPA success biases. Replacing original-message recovery by recovery of the decrypted honest challenge costs at most $\delta$ (only messages are legal recovery outputs). Using a loose $2L/M$ yields
$$
p_{\mathrm{OW\text{-}PCA}}\le 2\beta_D+2\beta_E+(L+1)\delta+(2L+1)/M.
$$

**From implicit rejection to plaintext checking.** In a reference game assign an independent uniform key per raw ciphertext, consistently on repeated decapsulation. At a target-key $H(pk,m,c)$ query use the plaintext check: for the uniquely accepted $m$ use the per-ciphertext value, and otherwise use a separate lazy hash-table value. Use a separate random challenge key. Decapsulation of $c^*$ is forbidden. Up to the first explicit guess of the fallback secret or hash query at $(pk,m_b,c^*)$ (here $m_b$ denotes the hidden encapsulated message), these tables couple to the real and random key experiments, with the challenge entry programmed according to the challenge bit. The dummy fallback secret is independent of the reference transcript, giving at most $Q_H/M$ for a correct guess. On a correctly decrypting challenge, a check at its true message returns one and the cached result supplies an OW-PCA output. If it decrypts incorrectly, charge at most $\delta$. This gives the conservative bound
$$
\operatorname{Adv}_{\mathrm{CCA}}\le 2\beta_D+2\beta_E+(L+2)\delta+(2L+Q_H+1)/M.
$$

Since $L\le Q_{RO}$ and $Q_{RO}\ge2$,

$$
(L+2)\delta\le2Q_{RO}\delta,
\qquad
\frac{2L+Q_H+1}{M}\le\frac{3Q_{RO}+1}{M}.
$$

Furthermore, applying the seeded classical CPA inequality from Chapter 6 separately to both derived CPA adversaries gives

$$
\beta_D,\beta_E\le
\epsilon_{4,4}+\epsilon_{4,5}+2\epsilon_A,
$$

where the $\epsilon$ terms dominate the corresponding source distinguishers at the **largest augmented resource budgets**, not just the original attacker's budget. Thus, in the precisely scoped *classical challenge-first ideal-oracle model*, the resulting conservative bound is

$$
\boxed{\begin{aligned}
\operatorname{Adv}_{\mathrm{CCA}}
\;\le\;&4\epsilon_{4,4}+4\epsilon_{4,5}+8\epsilon_A\\
&+2Q_{RO}\,2^{-212}
+(3Q_{RO}+1)2^{-256}.
\end{aligned}}
$$

This expression is an **upper bound conditional on source security, simulator budgets, and the applicable classical transformation-game hypotheses**. It is not a numerical security level in the absence of independent bounds on the source advantages and a verified common resource model. The displayed correctness term uses the strict inequality $\delta_{wc}<2^{-212}$ in its conservative non-strict presentation.

## Why pre-challenge decapsulation is a distinct obligation

The previous game is intentionally restricted. If an adversary can decapsulate ciphertexts before the challenge is formed, a new complication arises: a future challenge message may coincide with a message implicitly targeted through a previous decapsulation or oracle transcript. Treating that complication as though it were already included in $Q_{RO}$ can hide a missing bad event.

For a phase of $d_{\mathrm{pre}}$ decapsulation queries before a fresh uniform challenge message, the companion manuscript records a separate collision allowance of the form

$$
\min\{1,d_{\mathrm{pre}}2^{-256}+\delta_{wc}\}.
$$

The interpretation is that a **successfully decrypting** ciphertext determines at most one decoded message, and a fresh uniform message has probability at most $1/M$ of matching a fixed such candidate. Incorrect honest decryptions require the additional correctness allowance. This is a *separate* collision estimate, not a silent extension of the boxed challenge-first theorem to unrestricted CCA interaction. A complete general-game theorem must integrate this event with all oracle-programming and decryption simulations at the same level of precision.

## What the expression means numerically

Suppose for illustration that a hypothetical reduction has $Q_{RO}=2^{32}$, while ignoring for this calculation the unknown source advantages and the matrix-programming term. The two explicit terms from the displayed envelope are then

$$
2Q_{RO}2^{-212}=2^{-179},
\qquad
(3Q_{RO}+1)2^{-256}<2^{-222}.
$$

This arithmetic demonstrates only that the visible **correctness and message-guessing additions** can be small for a chosen query budget. It does **not** certify that the total CCA advantage is small, because the source terms are not numerically bounded by a hardness theorem. A classical attack estimate such as $2^{206.211}$ from Chapter 7 cannot be inserted as if it were a theorem $\epsilon_{4,j}\le2^{-206.211}$.

There is also no justified conversion from this classical-ROM arithmetic to an attacker's quantum success probability. A quantum query is not the same as a classical indexed lookup; in particular, a superposition query may involve many candidate messages simultaneously. Changing the oracle model changes the proof. The QROM statement belongs to Chapter 9 and uses different reductions and query factors.

## Engineering obligations not hidden by the theorem

Even a complete classical ideal-oracle proof would not guarantee that a deployed implementation is secure. Several concrete choices remain to be fixed and audited: byte-level tuple encodings, domain identifiers, canonical parsing under malformed inputs, secret-key storage, side-channel behavior, constant-time re-encryption and comparison, a concrete matrix expander, and the hash or XOF instantiation of $G$ and $H$.

A particularly common error is to replace the specified fallback $H(pk,z_0,c)$ by an unrelated tagged hash, or to return an observable error instead of an opaque key. Such a modification creates a *different KEM interface* and needs its own security analysis. A second error is to claim that the $2816$-byte tape is a uniform vector over all possible tuples $(r,e_1,e_2,\rho)$: the chosen mapping induces the intended CBD1 and uniform-lift distributions, but is generally many-to-one.

The optional confirmation-tag variant discussed in a companion study would add $32$ bytes, increasing ciphertext length to $1792$ bytes, but this chapter does not silently change the frozen format. A byte count for a tag is not a transformation theorem; its generation, verification and domain separation would require a separate specification.

## Scope of the classical reduction

The classical implication uses one challenge key, independent ideal oracles, a challenge-first transcript and augmented source budgets. Re-encryption gives rigidity. The ciphertext-dependent good and fallback key hashes determine which plaintext checks the simulator must answer. Section 8.7 derives the explicit advantage bound within that experiment. The collision calculation in Section 8.8 is available separately for an analysis allowing pre-challenge decapsulation.

The wrapper follows [S1] and the classical modular transformations are due to [R1]. A quantum-accessible oracle requires a separate coherent simulation. Chapters 9 and 10 address the quantum and failure-finding experiments.

# The Quantum Random-Oracle Boundary: A Conservative Conditional Route

## What changes when an oracle becomes quantum accessible

The classical argument in Chapter 8 makes a sequence of public and private computations while observing an adversary that submits ordinary strings to ideal hash functions. A quantum random oracle is not an ordinary table accessed only through classical calls. If $H:\mathcal X\to\{0,1\}^{\ell}$ is a sampled function, the adversary receives the unitary evaluation interface

$$
U_H:\ |x\rangle|y\rangle\longmapsto|x\rangle|y\oplus H(x)\rangle,
$$

and may evaluate $U_H$ on a superposition of addresses. A proof that programs one address *after* observing some oracle queries must specify the indistinguishability loss of that intervention. Merely remarking that a random 256-bit address is unlikely to have been sampled classically does not suffice. Conversely, a finite patch chosen and installed *before* a fresh quantum adversary is given its oracle does not involve retrospective reprogramming.

This chapter gives a single-user quantum random-oracle reduction for the frozen C1-K4 construction. Proposition 9.2 concerns the two exact CBD1 Module-LWE distributions. Lemma 9.5 links the ciphertext-dependent key derivation and implicit fallback to the explicitly rejecting reference transform. Theorem 9.6 then applies the extractable-oracle result of Hövelmanns, Hülsing and Majenz [R2] to this experiment.

## The complete key-first experiment

**Definition 9.1 (single-user key-first QROM IND-CCA experiment).** Independently sample the ideal matrix oracle $H_A$, the encryption-tape oracle $G$, and the KDF oracle $H$, with injective domain encodings and distinct oracle-family prefixes. Generate one C1-K4 key pair in that environment, with a fresh uniform 256-bit fallback secret $z$. Give the public key to the adversary *before its first oracle query*. The adversary may issue quantum queries to $H_A,G,H$ and classical decapsulation queries, may retain quantum workspace, and chooses two classical 256-bit challenge messages only in the associated PKE subexperiments. In the KEM experiment the challenger samples the encapsulation message $m_*$ uniformly and returns a classical challenge ciphertext $c_*$ together with either the real 256-bit KDF output or an independent random key. The adversary must output a classical bit. The exact challenge ciphertext $c_*$ may not be submitted for decapsulation after challenge delivery.

The adversary has no *earlier* queries or auxiliary advice correlated with the same sampled oracle tables. Its independent side information may be quantum, but the freshness condition is substantive. Oracle domains for $H_A$, $G$ and $H$ are pairwise separated even if an implementation were to use the same primitive under different prefixes. The good and fallback KDF inputs are **not** given artificial branch separators: both are evaluated by $H(pk,\cdot,c)$, as the frozen algorithm specifies.

To prevent an unnoticed change of adversarial power, we make the interface explicit.

| Interface | Input and reply | Allowed adversarial use | Excluded extension |
|---|---|---|---|
| Matrix oracle $H_A$ | Indexed 512-byte responses | Quantum, after $pk$ | Previously fixed oracle-correlated advice |
| Tape oracle $G$ | Encoded $(pk,m)$ to 2816 bytes | Quantum | An oracle with a different tape distribution |
| Key oracle $H$ | Encoded $(pk,m,c)$ to 32 bytes | Quantum | Artificially split good/fallback oracle families |
| Decapsulation | Classical ciphertext byte string to 32 bytes | Classical and adaptive | Quantum superposition of ciphertexts |
| Challenge | A classical 1760-byte ciphertext and key challenge | Once; challenge-first with respect to decapsulation in the reference game | Multi-user or arbitrary pre-challenge decapsulation |

The reference challenger may generate the independent encapsulation challenge when it generates the key and the simulator may withhold it during any initial hash-query phase. Evaluation at the hidden address uses the already sampled functions. Before delivery no decapsulation is allowed. The challenge-first restriction should not be confused with the key-first restriction: the public key comes before *any* oracle call, while the challenge arrives before the decapsulation part of the reference experiment. The latter matters for the application of [R2]. The restriction cannot be removed merely by citing a separate collision estimate.

For source $j\in\{4,5\}$, define a **quantum** decision gap $\epsilon_{4,j}^{Q}(T,\mathbf q)$ for distinguishing $(B,Bt+e)$ from $(B,y)$ with $B\leftarrow U(R^{j\times4})$, $t\leftarrow\chi^{1024}$ and $e\leftarrow\chi^{256j}$. The bound must apply to the *constructed* source distinguisher, including its run time, auxiliary quantum oracles and classical arithmetic. It is not the cost estimate of an attack algorithm and is not an assertion that every such distinguisher is inefficient.

## Reusing the passive hybrids without cloning quantum state

The two hybrids of Chapter 6 are classical data transformations applied to the challenge of a quantum adversary. A source distinguisher need not rewind or copy the adversary's state: it embeds its single source sample, runs the adversary once, and measures the final classical decision. In the first transition it gives $A$ together with $F_4(y)$. In the second, it partitions a five-row source matrix into $(A^T;V^T)$, publishes $(A,F_4(V))$ and forms the challenge from the five source response rows. The same source secret is used in all five rows. Because $V\leftrightarrow(F_4(V),[V]_4)$ is a bijection, the hidden remainder is uniform *conditioned on the published matrix and quotient $(A,F_4(V))$, before the challenge response is formed* as a consequence of the uniform five-row *matrix* law in both response branches. In the structured branch the response also depends on this same $V$.

A subtler point is the 32-byte seed in the compact public key. At the start of the reduction the simulator receives the source matrix $A$, selects a new seed $\sigma$, and installs the raw outputs associated with $\sigma$ into the otherwise fresh oracle $H_A$. Each 12-bit coefficient of $A$ is completed to a 16-bit raw word by four independently uniform high bits. Distinct $(\sigma,i,j)$ addresses are encoded injectively, so the 16 matrix-polynomial responses are jointly uniform with their exact raw-word distribution. The quantum adversary is then invoked *after* this random function has been defined.

This is a distributional coupling, not a proof that SHAKE256 behaves as an independent oracle. No oracle is changed after a query. A coherent implementation can compare an address with the finite programmed set, compute the appropriate raw word through a reversible table evaluation, XOR it into the answer register and uncompute the comparison. The extra reversible gates, memory and calls to the remaining oracle must be charged to $\epsilon_{4,j}^{Q}$.

**Proposition 9.2 (conditional quantum passive transfer).** In the fresh, key-first ideal-oracle model, if the two source quantum gaps at the augmented budgets are bounded by $\epsilon_{4,4}^{Q}$ and $\epsilon_{4,5}^{Q}$, then

$$
\operatorname{Adv}^{Q}_{\mathrm{CPA,bias}}\le
\epsilon_{4,4}^{Q}+\epsilon_{4,5}^{Q}.
$$

The proof is the pair of joint-distribution simulations just described. It is conditional on the source gaps and on the specified fresh-oracle experiment. It does not inherit Chapter 6's classical *prior-query conflict term*, since the quantum simulation selects and patches the seed before the first adversarial oracle query. Allowing historical oracle queries would require a new analysis.

## Two structural properties independent of source hardness

**Lemma 9.3 (rigidity).** Let $\operatorname{Dec}_1$ decrypt and accept only if the exact canonical input ciphertext equals $\operatorname{Enc}_1(pk,m')$, where $m'$ was obtained by decryption. Then

$$
\operatorname{Dec}_1(sk,c)=m'\neq\bot
\quad\Longrightarrow\quad
c=\operatorname{Enc}_1(pk,m').
$$

This follows by definition of the exact re-encryption check. It is not equivalent to perfect correctness, and it does not assert that the map from all 2816-byte tapes to ciphertexts is injective. Indeed, ignored tape bits make that latter statement false.

**Lemma 9.4 (spreadness).** For any honestly generated public key $pk$, any fixed message $m$, and any ciphertext byte string $c$,

$$
\Pr_{\omega}[\operatorname{Enc}(pk,m;\omega)=c]
\le\left(\frac34\right)^{1280}.
$$

To prove it, fix $r$ and the full lift vector $\rho$. Each coefficient of $e_1$ enters one of the 1024 components of $c_1$ and each coefficient of $e_2$ enters one of the 256 components of $c_2$, after a deterministic pre-error offset. The three possibilities $x-1,x,x+1$ under CBD1 have weights $1/4,1/2,1/4$; any floor cell of width two contains at most weight $3/4$, including when the modulus wraps. All 1280 error coefficients are still independent under this conditioning. Multiply the individual masses and then average over $(r,\rho)$. Therefore the spreadness exponent is

$$
\gamma\ge1280\log_2(4/3)
=531.247999\ldots.
$$

This lemma describes the randomized encryption distribution. A *deterministic* encryption with its oracle-derived tape fixed does not have this spreadness bound as an additional independent randomness source.

## The exact good/fallback graph under coherent access

Let $(pk_*,sk_*)$ be the reference challenge key. An explicitly rejecting reference transform uses a tape oracle $g(m)$ and a key oracle $h(m)$, together with deterministic encryption

$$
E(m)=\operatorname{Enc}(pk_*,m;g(m)).
$$

For the target prefix the reference $g$ outputs a uniform full tape and $h$ outputs a uniform key. A fresh secret $z$ is sampled independently of the reference experiment and kept for all simulated decapsulations. Its accepted decapsulation returns $h(m)$ and its rejected decapsulation returns $\bot$. To simulate the **actual** ciphertext-dependent and implicitly rejecting C1-K4 KEM, sample independent auxiliary random oracles $G_0(pk,m)$ and $H_0(pk,m,c)$ and define

$$
\widehat G(pk,m)=
\begin{cases}
 g(m),&pk=pk_*,\\
 G_0(pk,m),&pk\neq pk_*;
\end{cases}
$$

$$
\widehat H(pk,m,c)=
\begin{cases}
 h(m),&pk=pk_*\ \text{and}\ c=E(m),\\
 H_0(pk,m,c),&\text{otherwise}.
\end{cases}
$$

**Lemma 9.5 (conditional graph-oracle coupling).** Conditioned on the table of $g$ and on the complete key/matrix environment, $\widehat H$ is a uniform random function on its encoded domain, independent of $\widehat G$; the pair has the target joint law. Further, the target adversary's oracle queries can be simulated coherently and its *classical* decapsulation transcript has exactly the implicit-rejection law.

*Proof.* Define the graph $\mathcal G=\{(pk_*,m,E(m)):m\in\{0,1\}^{256}\}$. Two distinct messages have different graph addresses because the message is explicitly part of the address, even if their ciphertexts collide. Conditional on $g$, each graph address receives an independent uniform $h(m)$, and each nongraph address receives an independent uniform $H_0$ value. Hence every finite collection of distinct addresses has the product-uniform law. This conditional law is unchanged by the graph itself, so $\widehat H$ is independent of $g$ and of the independent off-prefix $G_0$ table as well, and hence of $\widehat G$. The matrix-oracle environment is preserved on both sides.

For quantum evaluation, reversibly compute the predicate $[pk=pk_*\wedge c=E(m)]$ using coherent access to $g$ and to the fixed matrix oracle. Obtain a candidate answer from $h(m)$ on the graph branch and from $H_0(pk,m,c)$ on the other branch. XOR the selected value into the output register. Reverse all computations and uncompute the temporary predicates and candidate registers. Because no measurement is used, linearity proves correctness on arbitrary superpositions. A repeated address has the same table value; when $m=z$ and a fallback address lies on the graph, the same $\widehat H$ value is used rather than an artificial independent fallback hash.

For a classical decapsulation query, forward the ciphertext to the explicit-rejection reference oracle. If it returns a key, rigidity implies that the ciphertext is $E(m')$ for its accepted message, and the reply equals $\widehat H(pk_*,m',c)$. If it rejects, the target reply is $\widehat H(pk_*,z,c)$ with the one sampled fallback secret $z$. Preserve the raw ciphertext byte string in the fallback hash even if parsing fails. The reference oracle's acceptance flag is never exposed directly to the target adversary. In the real-key branch the challenge key is $h(m_*)=\widehat H(pk_*,m_*,E(m_*))$. In the random-key branch the challenger supplies an independent uniform key with the same ciphertext. Thus the simulated and target transcripts have the same joint law. $\square$

The proof concerns ideal random functions and exact arithmetic in a reversible circuit. It does not furnish a reversible implementation with measured gate counts, nor does it establish the assumptions of an actual compiled hash function. The finite graph may be exponentially large as a mathematical oracle table; a reduction must charge the coherent cost of computing $E(m)$ rather than pretend to store it explicitly.

## Search for honest decryption failures

Let $K$ denote the entire honest key and matrix environment. Chapter 5 gives

$$
d(K)=\max_m\Pr_{\omega}[F(K,m,\omega)],
\qquad \delta_{wc}=\mathbb E_K[d(K)]<2^{-212},
$$

where $F$ is the honest decryption-failure event. Section 5.3 shows for this decoder that the all-one message realizes the maximum for each key, but the conservative QROM search argument needs only the displayed definition.

The extractable-oracle search result of Hövelmanns--Hülsing--Majenz [R2] bounds, for a fixed relation with maximum accepting oracle-output density $d$, the probability that an adversary with $u$ relevant oracle queries produces an accepting address by $152(u+1)^2 d$, capped at one. The published search lemma is Lemma 4, equation (30), while the cited full version uses Lemma 5, equation (51); its final evaluation contributes the $+1$. Extraction calls, oracle-table independence and the intended experiment are part of that theorem's hypotheses and cannot be erased when importing it.

For $c=\operatorname{Enc}(pk,m;g(m))$, a correct base decryption to $m$ always passes re-encryption with the same $g(m)$. If base decryption returns a different value, the checked decryption can only return that value or reject. Thus the failure predicate for the deterministic reference challenge is exactly the base failure predicate on the oracle tape. Fix $K$ *before* sampling the independent fresh tape oracle. Apply the search lemma to the relation $\{(m,\omega):F(K,m,\omega)\}$. The adversary may have $pk$ and may select $m$ adaptively; its fresh oracle output for a fixed address is distributed as an honest encryption tape. The density is at most $d(K)$ for every message. Averaging over $K$ and using the cap gives

$$
\begin{split}
\operatorname{Adv}^{\mathrm{FFP\text{-}CPA}}(u)
&\le\mathbb E_K\min\{1,152(u+1)^2d(K)\}\\
&\le\min\{1,152(u+1)^2\delta_{wc}\}.
\end{split}
$$

The worst-case-correctness alternative of Hövelmanns and Majenz [R11] provides further context. The constants used here come from [R2]. The right-hand side does **not** follow by naively replacing $\mathbb E_K\max_m$ with $\max_m\mathbb E_K$. The key is fixed only for the search-lemma argument and averaged afterwards. The estimate is an *upper bound on failure-finding success in the imported extractable-oracle setting*, not a theorem that every ciphertext is correctly decrypted.

## Composing the conservative conditional bound

Let $q_A,q_G,q_H,q_D$ be upper bounds on matrix-, tape-, KDF- and classical decapsulation calls made by the target quantum adversary. The graph simulation can conservatively make two reference-$g$ calls per target $G$ call, two reference-$g$ and two reference-$h$ calls per target $H$ call, and two of each per simulated fallback. Define

$$
Q_G=2(q_G+q_H+q_D),\qquad
Q_H=2(q_H+q_D),
$$

$$
R_Q=Q_G+Q_H,\qquad U_Q=R_Q+q_D.
$$

The symbol $R_Q$ denotes an oracle-query count, not the polynomial ring $R$. These allowances are deliberately conservative; they count compute--use--uncompute, do not assume gratis controlled calls, and do not include an explicit numerical circuit cost for every arithmetic step. The constructed source algorithms must pay for the additional matrix-oracle, extraction and auxiliary-oracle work.

**Theorem 9.6 (single-user conditional QROM bound).** Suppose the exact C1-K4 source problems have quantum distinguishing gaps bounded by $\epsilon_{4,4}^Q$ and $\epsilon_{4,5}^Q$ at the augmented resources used by the simulations. Assume the key-first, single-user, fresh ideal-oracle, challenge-first and classical-decapsulation interfaces of Definition 9.1, with independent matrix and auxiliary oracle interfaces available to the constructed algorithms and their calls charged to the resource budgets. Then the exact 1760-byte C1-K4 KEM satisfies the following conservative bound:

$$
\boxed{\begin{aligned}
\operatorname{Adv}^{Q}_{\mathrm{CCA,bias}}
\le\min\biggl\{\frac12,\;&
4\sqrt{2U_Q(\epsilon_{4,4}^Q+\epsilon_{4,5}^Q)}
+8U_Q\,2^{-128}\\
&+(q_D+1)\min\{1,152(U_Q+1)^2\,2^{-212}\}\\
&+24q_D(Q_G+4q_D)(3/4)^{640}
\biggr\}.
\end{aligned}}
$$

*Derivation and provenance.* Use [R2], full version, Corollary 8, equations (69)–(70), for the explicitly rejecting, message-only key derivation $\mathrm{FO}^{\bot}_m$ of its Figure 2. Figure 5 gives the ordinary classical challenge and decapsulation interface. Use the reductions relative to the independent matrix and auxiliary oracle families. All games share them, including public encryption and its reversible evaluation. Their gates act on separate tables from the compressed $g,h$ database, so they are available between queries. Their calls, time and memory count toward the constructed adversaries; the source gaps are those in this joint experiment. Lemma 9.5 supplies the explicit-rejection reference adversary, Lemma 9.4 supplies $\gamma$, and Section 9.6 supplies the failure relation. Let $d$ be the reference query depth and $w$ its width. Serializing gives $d\le R_Q$; the IND-CPA branch does not use the alternative OW-CPA term involving $w$. Its bound is

$$
\begin{aligned}
&4\sqrt{(d+q_D)\alpha_{\mathrm{CPA}}}+
8(Q_G+Q_H+q_D)/\sqrt{|\mathcal M|}\\
&\quad +(q_D+1)\operatorname{Adv}_{\mathrm{FFP\text{-}CPA}}
+24q_D(Q_G+4q_D)2^{-\gamma/2}.
\end{aligned}
$$

Here $\alpha_{\mathrm{CPA}}$ is an *acceptance-probability gap*, whereas the CPA quantity in Proposition 9.2 is a *success bias*, so $\alpha_{\mathrm{CPA}}\le2(\epsilon_{4,4}^Q+\epsilon_{4,5}^Q)$. For a fair bit, $|\Pr[\hat b=b]-1/2|=|\Pr[\hat b=1\mid b=1]-\Pr[\hat b=1\mid b=0]|/2$. The reference KEM advantage already uses success bias. Corollary 8 gives the failure adversary at most $U_Q$ calls to eCO.RO and $q_D$ to eCO.E. The search lemma charges the final output evaluation separately and allows these extraction queries. Use $d+q_D\le U_Q$, the message space $|\mathcal M|=2^{256}$, the search bound with $u=U_Q$ and $\delta_{wc}<2^{-212}$, and $2^{-\gamma/2}\le(3/4)^{640}$. The coupling in Lemma 9.5 transfers the reference-game bound to the exact good/fallback hash interface. The result is a direct restatement and parameter substitution of the V4.2 companion's conditional result, not a newly discovered independent quantum-security theorem.

The time overhead includes coherent C1-K4 re-encryption, matrix oracle expansion, the finite seed patch, address comparisons and the extractable-oracle simulation. A schematic ledger is

$$
T_{\mathrm{sim}}=O\big((q_G+q_H+q_D)T_{\mathrm{Enc}}+
(q_A+q_G+q_H+q_D)L_{\mathrm{cmp}}\big)
+T_{\mathrm{eCO}}(R_Q+q_D),
$$

plus arithmetic and source-simulation overhead. Extraction has a resource cost even though its query count does not enter the density multiplier. The source budgets include the quantum workspace for the compressed database and reversible encryption. For variable-length raw ciphertexts the time bound uses the longest input the adversary presents within its running-time limit. Here $L_{\mathrm{cmp}}$ represents reversible comparisons, controls, and table-lookups. Every $\epsilon_{4,j}^Q$ must apply at the resulting resources. No independent statement about the concrete quantum difficulty of these sources is known from this derivation.

## Boundaries and counterfactual changes

The proven structural lemmas of this chapter use the frozen ciphertext and sampling law. Several attractive changes would alter the theorem: adding a 256-bit confirmation tag changes the format to 1792 bytes; using a distinct fallback hash family would change the graph coupling; removing the random lift changes the second source simulation; supplying a quantum decapsulation oracle changes the threat model; instantiating $H_A,G,H$ with SHAKE256 changes the ideal-oracle hypotheses. The conservative theorem does not silently cover any of these changes.

A large conditional exponent in a particular *classical* attack-cost model cannot replace $\epsilon_{4,j}^{Q}$ in the square-root term. Even if every displayed arithmetic factor were checked, the resulting inequality would not certify 128-bit post-quantum security without independent source hardness and resource-conversion premises. The exact question for the closing chapters is whether a stronger *failure-game* analysis can improve the conservative failure term while respecting the same frozen protocol. Chapter 10 treats that question.


# Finding Failures: Shared Tapes, Second Moments, and the FFP Frontier

## Why correctness is not the same experiment as failure finding

Chapter 5 bounds the probability that honestly sampled encryption coins cause an error. A random-oracle KEM permits a qualitatively different adversarial action: an attacker can evaluate an oracle on many messages and try to choose the message whose oracle-derived tape is unusually unfavorable. Whether such selection can exploit a *public key* is another, still more delicate, question. These effects are the reason that a small decryption-failure probability is not automatically a complete chosen-ciphertext security proof.

This chapter separates four random objects: the generated key, the message, the complete encryption tape, and the adversary's observable view. It then reconstructs the independent-key second-moment argument from companion Article [S4]. The theorem is conditional only on the specified distribution of key generation and tapes, not on hardness of Module-LWE. Its relevance to a QROM reduction depends on a separately identified search lemma. The current Ring/Module-LWE literature on public-key-dependent failure finding is considered explicitly rather than dismissed on the basis of noise nomenclature.

## A dictionary of failure parameters

Let $K=(pk,sk,H_A)$ stand for a full generated key environment. Let $\omega$ be a complete honest uniform encryption tape, *including* all ephemeral CBD1 and lift coordinates. Define the honest failure predicate

$$
F(K,m,\omega)=
\mathbf1\{\operatorname{Dec}_{sk}(\operatorname{Enc}_{pk}(m;\omega))\ne m\}.
$$

The parameter used in Chapters 5, 8 and 9 is

$$
d(K)=\max_m\Pr_\omega[F(K,m,\omega)=1],
\qquad
\delta_{wc}=\mathbb E_K d(K).
$$

The maximum is *inside* the key expectation. In this particular C1-K4 decoder, the all-one message is maximal at each fixed key, because only the decision ties at $1024$ and $3072$ break bit symmetry. Section 5.3 proves the pointwise inclusion of failure events for the two bits; enumeration of all 4096 residues also verifies the endpoints. Consequently, $d(K)$ also equals the honest error probability for the all-one message at that key. We retain the general notation $d(K)$ because the proof strategy should not depend on a coincidence that can fail for another decoder.

For a message and tape held fixed, sample an **independent fresh key** and define

$$
p_m(\omega)=\Pr_K[F(K,m,\omega)=1],
\quad
\delta_{ik}=\max_m\mathbb E_{\omega}p_m(\omega),
\quad
\sigma_{\delta ik}^2=
\max_m\operatorname{Var}_{\omega}(p_m(\omega)).
$$

The letter $ik$ refers to an independent key in the FFP-NK setting, not to a key $K$ that is revealed to the adversary. Interchanging the two maxima and expectations without a coupling proof is generally invalid. For the frozen decoder, the all-one message is a simultaneous maximizer for each key, so the equality $\delta_{ik}=\delta_{wc}$ holds under the identical honest distributions. The generic inequality $\delta_{ik}\le\delta_{wc}$ would suffice elsewhere.

| Quantity | What is fixed | What is averaged | What it does not bound |
|---|---|---|---|
| $d(K)$ | One full honest key | Fresh tape, maximum over messages | Every alternative generated key |
| $\delta_{wc}$ | C1-K4 algorithm | Average of $d(K)$ over keys | Worst-case malicious keys |
| $p_m(\omega)$ | One message and one complete tape | Fresh independent keys | Failure rate conditional on a matching public key |
| $\delta_{ik}$ | Message before maximizing | Tape and independent key | Public-key-dependent tape selection |
| $\sigma_{\delta ik}$ | Same message | Standard deviation across complete tapes, maximized over messages | Full IND-CCA advantage |
| FFP-NG advantage | Public key is visible | Random target-key experiment and adversarial coins | Itself a CCA bound without composition |

An elementary but useful observation is $0\le p_m(\omega)\le1$. Hence $\operatorname{Var}p_m\le\mathbb E p_m$ and $\sigma_{\delta ik}<2^{-106}$ follows directly from the $2^{-212}$ correctness theorem. The aim below is a stronger result obtained without pretending that different output coefficients have independent tapes.

## FFP-NK and FFP-NG are different games

**Definition 10.1 (FFP-NK).** The adversary uses the specified extractable tape-oracle interface and outputs only a classical message $m$. The challenger evaluates $\omega=\mathrm{eCO.RO}(m)$ at the end and tests $F(K,m,\omega)$ with a fresh independent honest key $K$. The tape includes all 2816 bytes. The tested key and its environment are independent of the adversary's oracle setup. Success is the probability of the failure event. Allowing the adversary to supply an arbitrary tape would define a different game.

**Definition 10.2 (one-pair FFP-NG).** Draw independent honest keys $K_0,K_1$ and a fair hidden bit $b$. Give $pk_0$ to the adversary. It may submit one chosen pair $(m,\omega)$ of a message and a full tape, receive $F(K_b,m,\omega)$ and output $b'$. Its advantage is $|\Pr[b'=b]-1/2|$. The distinction from FFP-NK is twofold: here the adversary knows $pk_0$ and chooses its tape. Its final decision can use its entire public-key-dependent view.

In FFP-NK, the final oracle tape is independent of the unknown tested key. In FFP-NG, the tape may be a function of $pk_0$ and may correlate with the actual secret through the *public* arithmetic. That correlation is precisely what a hardness or reduction argument must address. The FFP-NG game is not solved by the two-key second moment alone, and an inability to find an attack among a handful of tested heuristics is not a proof of the absence of such a correlation.

## A same-tape, two-independent-key identity

Take two independent copies $K^{(1)},K^{(2)}$ of the complete honest key experiment, including the stipulated matrix environments. Feed them the *same* complete tape $\omega$ and fixed message $m$. Conditional independence of the two keys for each fixed tape implies the exact identity

$$
\mathbb E_\omega[p_m(\omega)^2]
=\Pr_{\omega,K^{(1)},K^{(2)}}
[F(K^{(1)},m,\omega)=1\ \wedge\ F(K^{(2)},m,\omega)=1].
$$

This identity is simple but important. It transforms a variance question into a probability of two simultaneous failures. It does **not** make the two events independent unconditionally, since the tape is shared. Nor may a proof replace the tape by independent tapes for each key: doing so would produce $(\mathbb E p_m)^2$, a different quantity.

The key environment matters when using a compact seeded representation. The theorem uses two independently drawn honest key environments, with the independently specified seed/matrix law, while the same encryption tape is shared. If an implementation or a multi-user experiment forces both keys to share an oracle state with seed collisions, the coupling of the two environments must first be re-established. The two-key estimate must not silently be transplanted to such a variant.

## The diagonal moment calculation and why Cauchy--Schwarz is necessary

Write $\varepsilon_i(K,\omega)$ for the integer error lift of output coefficient $i$. From the Chapter 5 containment, each failure implies that for some coefficient $i$ and sign $a\in\{\pm1\}$, $a\varepsilon_i\ge1024$. For a fixed $u>1$ set

$$
g_{i,a}(\omega)=\mathbb E_K u^{a\varepsilon_i(K,\omega)}.
$$

At two possibly distinct indices and signs, $g_{i,a}$ and $g_{j,b}$ share the tape and need not be independent. Cauchy--Schwarz gives the valid substitute

$$
\mathbb E_\omega[g_{i,a}g_{j,b}]
\le
\sqrt{\mathbb E_\omega[g_{i,a}^2]\,\mathbb E_\omega[g_{j,b}^2]}.
$$

It suffices to dominate each diagonal square $\mathbb E g_{i,a}^2$ uniformly. The same-index diagonal is special: in each negacyclic block, the two keys see the *same signed permutation* of the common tape coordinates. This makes a coordinatewise conditioning argument possible. The negacyclic sign is absorbed by maximizing over the two signs in the following exact finite moments.

Let $E,E',S,S',R$ be independent CBD1 variables; let $T$ be uniform on $\{0,1,2,3\}$; and let $E_1,E_2$ be fresh CBD1 variables shared by the two key calculations. Define

$$
\begin{aligned}
A_2(u)=\max_{a,b\in\{\pm1\}}\mathbb E_{E,E'}\,
\max_{t,t'\in\{0,1,2,3\}}
\mathbb E_{R,T}
 u^{R[a(E+T-t)+b(E'+T-t')]},\\
B_2(u)=\max_{a,b\in\{\pm1\}}\mathbb E_{S,S'}\,
\max_{c,c'\in\{0,1\}}
\mathbb E_{E_1}
 u^{aS([c+E_1]_2-E_1)+bS'([c'+E_1]_2-E_1)},\\
C_2(u)=\max_{a,b\in\{\pm1\}}
\mathbb E_{E_2}\,
\max_{t,t'\in\{0,1\}}
 u^{a(E_2-t)+b(E_2-t')}.
\end{aligned}
$$

The placement of the *inner* maxima is part of the theorem: it prevents replacing correlated derived remainders by independent samples. The offset maxima in $A_2$ and $B_2$ occur after the indicated conditional expectation, while the maximum over the final two remainders in $C_2$ stays within the expectation over $E_2$. Moving either maximum could produce a smaller, unjustified upper bound.

**Theorem 10.3 (same-tape second-moment bound).** Under the two-independent-key and shared-tape law,

$$
\mathbb E_\omega[p_m(\omega)^2]
\le4n^2u^{-P/2}\bigl(A_2(u)B_2(u)\bigr)^{kn}C_2(u),
$$

for any fixed message $m$, with $n=256$, $k=4$, $P=4096$ and $u>1$.

*Proof.* Fix a coefficient and a tail sign. In the simultaneous two-key exponential moment, eliminate the two final remainders pointwise and then average the *shared* $e_2$ using $C_2$, without claiming its independence from those remainders. Condition on both complete key-generation draws (including the latent errors $e_0^{(1)},e_0^{(2)}$), the shared $r,\rho$, and the parity offsets; average each independent shared $e_1$ coefficient and maximize over its pair of fixed offsets. This produces a product of functions of two key-secret coefficients, not yet the scalar $B_2^{kn}$. Next hold both actual $r_0$ vectors fixed under their corresponding keys, average each independent shared pair of $r$ and $\rho$ coefficients matched by the signed permutation for output $i$ and maximize the two hidden public remainder values. This produces a product of functions of the two corresponding key-generation errors. Only then average the independent secret and key-error coefficients across the two keys. The signed permutations are bijections at the same output index, giving $kn=1024$ factors of the bounded $A_2B_2$ product. Thus each diagonal expectation is at most $C_2(A_2B_2)^{kn}$. Use Cauchy--Schwarz for arbitrary pairs of output indices and signs. A two-tail exponential Markov bound at total threshold $2\cdot(P/4)=P/2$, followed by the union bound over $4n^2$ index/sign pairs, gives the stated inequality. $\square$

At the exact rational choice $u=6/5$, enumeration of the finite supports gives

$$
A_2=\frac{99036386572201}{83980800000000},\qquad
B_2=\frac{489677}{480000},\qquad
C_2=\frac{3721}{2500}.
$$

Consequently,

$$
\begin{split}
M_2&=4\cdot256^2\left(\frac65\right)^{-2048}
\left(\frac{99036386572201}{83980800000000}
\frac{489677}{480000}\right)^{1024}\frac{3721}{2500},\\
M_2&<2^{-247},\qquad
\log_2 M_2\approx-247.024841645.
\end{split}
$$

The strict comparison is between *integers* after clearing denominators; it is not a test of a floating-point logarithm. The accompanying verification script reconstructs this exact rational and checks the strict inequality. Since variance is bounded by the second moment,

$$
\boxed{\sigma_{\delta ik}<2^{-123.5}.}
$$

This is a new application-specific probability estimate in [S4] that extends the simpler Chapter 5 first-moment bound. It does not itself provide a small FFP-NG advantage or a quantum LWE theorem.

## Search bounds and the essential scope of the oracle lemma

Consider the HHM search lemma [R2], with $q$ queries to its specified extractable oracle before the adversary's final output. Write $D_q=152(q+1)^2$. In the independent-key FFP-NK experiment, if the key is not known to the attacker and the oracle is fresh, the relation for any fixed target key has maximum tape density $d(K)$. Applying the search lemma and then averaging the key gives the **direct** bound

$$
\operatorname{Adv}_{\mathrm{FFP\text{-}NK}}\le
\min\{1,D_q\delta_{wc}\}.
$$

It uses $\mathbb E_K\max_m$, not merely $\max_m\mathbb E_K$. If only $\delta_{ik}$ were available, one can instead apply Markov to $p_m(\omega)$ with a threshold $t$; for a single fixed adversary this yields

$$
\Pr[p_m(\omega_{\mathrm{out}})\ge t]
\le\min\{1,D_q\delta_{ik}/t\}.
$$

For $a=D_q\delta_{ik}\in(0,1)$, integrating that tail distribution bounds the mean failure success by

$$
\mathbb E\,p_m(\omega_{\mathrm{out}})\le a(1+\ln(1/a)).
$$

For $a=0$ take zero, and for $a\ge1$ cap the bound at one. The threshold and direct formulas use different correctness hypotheses and should not be conflated. The imported FFP-NK variance refinement can inform some decompositions, but the leading variance-dependent summand alone cannot be presented as an end-to-end KEM bound. The advantage normalization also matters: a failure-event probability needs no factor of two, whereas a distinguishing bias generally does.

As an illustration, taking $q=2^{32}$ and the upper bound $\delta_{wc}<2^{-212}$ gives

$$
D_q\delta_{wc}<152(2^{32}+1)^2 2^{-212}
\approx2^{-140.752}.
$$

That exponent describes the stated direct *failure-game component*. It must not be advertised as a 140-bit security certificate for the complete QROM KEM: source hardness, oracle composition and other positive terms remain.

## Generic bad tapes versus public-key-dependent tape selection

The supplementary experiments of [S4] distinguish two mathematically different protocols. In the first, a procedure generates a *family* of structured tapes and resamples selected lift coordinates from trial to trial. In the second, a **complete fixed tape** is held unchanged across independent keys. The latter is the direct empirical analogue of $p_m(\omega)$ for that tape. Mixing the two protocols into a single frequency would silently change the sampled random variable.

The reported finite controls use 128 trials per arm with integer negacyclic convolution. With three all-one active ephemeral module components and all lift coefficients set to zero, failures were observed in all 128 trials in both independent arms; with fresh uniform lifts in their separate controls, the observed counts were zero in the stated arms. These observations demonstrate strongly structured *possible* failure tapes, not their probability under an honest random oracle. Their sampling procedure does not define an FFP-NG attack because the tape was not selected from a matching public key. The proofs in this chapter do not depend on those frequencies.

The separate FFP-NG pilot of [S4] tested four elementary public-key-dependent tape-selection rules: a midpoint-sign rule, a parity rule, a three-valued threshold rule, and a local-difference sign rule. Each was tested on 128 independently generated key pairs, and the reported failure frequencies in both arms were zero. The resulting plug-in bias is zero **for those samples only**. No conclusion about true zero advantage, universal resistance, or negligible asymptotic advantage follows. Adaptive polynomial-time and quantum adversaries cover a far larger class of algorithms than these four rules.

## A contemporary Ring/Module-LWE result and a missing bridge

Majenz--Sisinni [R4] studied a non-generic failure-finding reduction in an LWE setting. Their later work [R5], *Provable Decryption Failure Security for Practical Lattice-Based PKE* (IACR ePrint 2025/1997, also listed as accepted at SAC 2026), explicitly extends the analysis to Ring/Module-LWE families and subgaussian errors including centered binomial laws. Therefore the statement “FFP-NG theory exists only for Gaussian LWE” would be scientifically incorrect. The applicability of [R5] to the **exact** C1-K4 quotient/lift mechanism still requires checking the hypotheses of the version to be cited.

| Interface to match | Frozen C1-K4 specification | Necessary comparison before importing an FFP-NG theorem |
|---|---|---|
| Public operator | Uniform four-by-four matrix over $\mathbb Z_{4096}[x]/(x^{256}+1)$ in the ideal model | Same ring/module generation and public-view assumptions |
| Secret | Independent coefficient CBD1 | Permitted secret support, covariance and reduction interface |
| Explicit key error | Independent coefficient CBD1 | Required noise splitting or subgaussian conditions |
| Public rounding | $F_4(As+e_0)$ | Whether quotient observation and discarded remainder fit the theorem's encryption law |
| Encryption lift | $v=4b+\rho$ with fresh uniform $\rho$ | Joint law of $v$, $b$ and public key; no artificial independence |
| Ciphertext compression | $F_2$ on both components | Treatment of the resulting dependent remainders and failure predicate |
| Shared ephemeral secret | Same $r$ in $c_1,c_2$ | Sample correlations and matrix augmentation permitted by the theorem |
| Failure-selection interface | Attacker may choose tape using public view | Exact FFP-NG game and key/coin quantifiers |
| Security model | Key-first fresh QROM for Chapter 9 | Distinguishing gap, query model and any additional assumptions |

A statement about the distribution family alone is insufficient: C1-K4's compression and randomized lifting may place it outside an otherwise applicable standard PKE template. Conversely, the fact that its coefficients have dependent remainders does not prove the reduction impossible. The right verdict, until every row is matched and the source theorem is instantiated, is **open**, not refuted and not automatically proved.

## Chapter conclusion

The same-tape second moment and the direct extractable-oracle search bound quantify two important ways in which failure probabilities enter a QROM analysis. They demonstrate that the generic bad-tape issue can be bounded in an explicit model without forcing a full FFP-NG reduction into the conservative composition of Chapter 9. They do *not* neutralize a hypothetical key-dependent adversary outside those game interfaces. The sharper match to [R5], with all C1-K4 quotient, lift and compression details exposed, remains a genuine research problem. Chapter 11 will combine the contributions in a ledger that prevents these distinct claims from being mistaken for interchangeable security guarantees.


# Evidence, Assumptions, and the Cost of a Security Claim

## A theorem has an experiment attached to it

A cryptographic manuscript is not adequately summarized by its largest exponent. It must explain what the exponent measures and to which probability experiment its supporting proof applies. The C1-K4 case presents four distinct kinds of knowledge. Exact equalities and rational bounds follow from fixed algebraic laws. Hybrid reductions convert one specified advantage into a sum of other specified advantages. Cryptanalytic cost models estimate known algorithmic strategies under heuristic formulas. Implementation and experimental reports show only what has been executed on particular programs or data. Confusing one category with another can turn a scientifically useful research candidate into an unsupported product claim.

This chapter records the principal results of the first ten chapters with their quantifiers and explicit boundaries. It is part of the scientific argument: a reader should be able to reconstruct both what follows and what does not follow from the definitions, proofs, source versions and recorded computations.

## Frozen parameters, objects, and exact numerical statements

The immutable instance is

$$
R=\mathbb Z_{4096}[x]/(x^{256}+1),\quad
(k,n,P,Q,C)=(4,256,4096,4,2),\quad \chi=\mathrm{CBD}_1.
$$

Secrets, key errors and encryption errors are sampled coefficientwise from $\chi$; the lift $\rho$ is uniform on $\{0,1,2,3\}^{1024}$. The public quotient $b=F_4(As+e_0)$ and the lifted value $v=4b+\rho$ are **not independently sampled** under a real key. The same $r$ appears in both ciphertext components. These choices define the experiment; changing any of them creates a new scheme rather than an optimization of the same theorem.

The exact core statements are:

$$
\varepsilon=(e_0+\rho-r_0)^Tr+(r_1-e_1)^Ts+e_2-r_2,
$$

$$
2^{-8192}\le\delta_{wc}<2^{-212},
\qquad
\mathbb E_\omega[p_m(\omega)^2]<2^{-247},
$$

$$
|pk|=1313\ \mathrm{bytes},\qquad
|ct|=1760\ \mathrm{bytes},
\quad\gamma\ge1280\log_2(4/3).
$$

The correctness parameter is averaged over honest generated keys after maximizing the per-key honest message-failure probability. The same-tape second moment uses two independent complete key environments and a shared honest tape. The byte counts concern canonical packed public payloads in the 256-bit ideal matrix-seed model; they are not totals for a finished KEM API with all secret-key fields, algorithm identifiers and transport framing.

**Claim register (exact statements).**

| Claim | Where it is established | Crucial condition |
|---|---|---|
| Unique quotient/remainder coordinates and carries | Chapter 2; [S3] | Canonical residues and stated divisibility |
| Negacyclic block signs and module transpose | Chapter 3 | $x^{256}=-1$, ring coefficient law |
| Complete C1-K4 algorithms | Chapter 4 | Exact CBD1, lifting and shared $r$ |
| Error congruence and nonperfect correctness | Chapter 5; [S1] | Honest keys and tapes; actual derived remainders |
| Uniform-lift simulation | Chapter 6 | Uniform $V$ *jointly* with the public quotient |
| Rigidity | Chapters 8--9 | Full byte-exact re-encryption comparison |
| Spreadness | Chapter 9; [S2] | Randomized encryption with independent $e_1,e_2$ |
| Same-tape second moment | Chapter 10; [S4] | Two independent key environments and one shared tape |

A script reproducing a fraction proves that the fraction was computed, not that the conditioning argument from which it arose was correct. The latter must be read at the level of the proof. In Chapter 5 the rational upper bound uses $u=4/3$ and $kn=1024$; in Chapter 10 the second-moment bound uses $u=6/5$ with a different sequence of conditional maxima. Their numerically similar forms are not interchangeable.

## A graph of the conditional reductions

The passive-security chain is

$$
\mathrm{DMLWE}_{4,4}^{\chi}
\ \&\ \mathrm{DMLWE}_{4,5}^{\chi}
\Longrightarrow\ \text{conditional C1-K4 IND-CPA}.
$$

This is an implication **under small exact source gaps at the reductions' budgets**, not a theorem proving those gaps small. The four-row source simulates the public quotient; the five-row source simulates the joint challenge with a shared ephemeral secret and a lifted public quotient. In the seeded model the matrix oracle must be included in the source distinguishers' resource accounting. The direction of implication is not reversed by displaying an attack estimate.

The classical wrapper chain, for one key and the challenge-first ROM model of Chapter 8, is

$$
\operatorname{Adv}^{\mathrm{classical}}_{\mathrm{CCA,bias}}
\le4\epsilon_{4,4}+4\epsilon_{4,5}+8\epsilon_A
+2Q\,2^{-212}+(3Q+1)2^{-256},
$$

with $Q=q_G+q_H+3q_D+2$ in that chapter's allowance. Its terms have different origins: source gaps, a prior-query matrix-seed conflict in the applicable classical simulation, a correctness contribution, and random guessing. The inequality cannot be transplanted to a quantum adversary by replacing $q$ with a quantum query count. The quantum key-first source coupling does not even use the same matrix-seed conflict term.

Chapter 9, using the published/full-version extractable-oracle route of [R2], gives a distinct **conditional quantum** inequality with a square root in the quantum source gaps and explicit spreadness and failure-finding contributions. Its $U_Q$ is not equal to the classical $Q$ above. The correct comparison is between *fully stated experiments*, not merely formulas. The sharper FFP-NG route is marked open because the contemporary structured-lattice theorem's hypothesis match to C1-K4 is not established in Chapter 10.

**Claim register (conditional statements).**

| Claim | Security experiment | External premise or unfinished check |
|---|---|---|
| Two-source CPA transfer | Classical or quantum with specified oracle treatment | Small exact decision gaps for $j=4,5$ |
| Classical FO/implicit-rejection bound | Single-user, ideal classical ROM, challenge-first | Corrected [R1] game interfaces and augmented budgets |
| Graph simulation | Fresh key-first QROM, classical decapsulation | Exact reversible oracle coupling and [R2] reference transform |
| Conservative QROM expression | Same quantum experiment as Chapter 9 | Quantum source gaps and [R2] hypotheses/normalization |
| Refined FFP-NG composition | Would require fully specified FFP-NG experiment | Exact instantiation of [R5] or a new reduction |

The words *conditional* and *conservative* have distinct meanings. Conditional names the external premise: the source distributions and imported theorem must satisfy the stated bounds. Conservative means the proof deliberately uses a coarse upper bound that may be far from tight. Neither adjective means incorrect or insignificant; neither alone implies a concrete numerical security level.

## Work, advantage, time, and memory must not be silently identified

A dual or lattice attack estimator returns a number of modeled primitive operations at a point in a parameter search. A security reduction supplies a computational transformation of one adversary into a distinguisher for an assumed-hard source problem. These numbers measure different quantities until a common cost model is fixed. In particular, if a model predicts work $2^{206.211}$ for a named strategy, the model supplies an example of a candidate attack cost, **not** the inequality that every attack takes at least $2^{206.211}$ steps. Subtracting a hypothetical reduction overhead of approximately $2^{22.17}$ does not repair the logical reversal.

The evidence from Chapter 7 is limited as follows.

| Evaluated attack model | Reported $\log_2$ work | Evidence limit |
|---|---:|---|
| Classical primal uSVP | 227.176 | Baseline heuristic |
| Classical plain dual | 235.060 | Baseline heuristic |
| MATZOV expanded local search | 213.656698256 | Best point found in documented scan, not a global minimum |
| Rotation-aware coded-dual | 206.210581344 | Pinned Sage model and two adaptations; historical full cost evaluation |

The MATZOV run at fixed $m=1024$ found the point

$$
(\beta,p,k_{enu},k_{fft},k_{lat},d)=(731,3,6,114,904,1928).
$$

The separately reported probe with $m=1280$ required gives $221.9708\ldots$ at its local point. The larger number cannot establish a higher minimum work requirement for the five-row source: a solver may ignore 256 of its 1280 observations and use the four-row strategy. The coded-dual reported point uses $m=770$ and has a valid checked search-bracket crossing at $R=137829$ in its specified model, but this does not resolve all rotation-compatible selections or untested variants. These statements are about the coverage of the *search*, not the truth of a general hardness conjecture.

The modeled re-encryption schedule counts 63,200 abstract operations and uses a conservative $2^{16}$ bound with $2^{17}$ precomputation allowance. The value $\bar\kappa=393217$ and $\log_2(12\bar\kappa)\approx22.16992867$ are bookkeeping outputs, not processor measurements or a quantum gate count. A reproducible thesis may print them for transparency but must not silently identify their units with those of the attack estimates. A concrete reduction-to-security-bits conversion would additionally require source work/advantage curves, adversary memory bounds, sample complexity and oracle/circuit costs.

## Two numerical examples that should not be overinterpreted

At the classical ROM query allowance $Q=2^{32}$, the explicitly bounded correctness term in Chapter 8 satisfies $2Q2^{-212}=2^{-179}$; the guessing term is $(3Q+1)2^{-256}<2^{-222}$. These small terms leave the source gaps $\epsilon_{4,4},\epsilon_{4,5}$ entirely unknown. Thus the numerical example does not imply an IND-CCA advantage below $2^{-128}$.

At the QROM failure-search allowance $u=2^{32}$, the direct extracted-oracle failure component is less than $2^{-140.75}$, as Chapter 10 explains. It is nevertheless multiplied by $(q_D+1)$ in the composed inequality and accompanied by a square-root source-advantage term and a spreadness term. Furthermore $u$ must be the **translated** extractable-oracle query budget, not an original user-facing query count. Reporting $2^{-140.75}$ in isolation as the system's security would discard the very components a proof is designed to retain.

## Reproducibility hierarchy and versions

The intended public archive should separate three sources of evidence. A proof document gives definitions and inference steps. Executable checks regenerate exact finite fractions, parameter counts and specified model outputs. An immutable reference to public attack code records the cost formula and search environment used at the time of the calculation. A prose report may explain how an audit was conducted, but its verdict is **not** a replacement for these objects.

Chapters 1–8 derive from V0.4. Chapters 9–12 were introduced in V0.5 and revised in this edition. The sources of their conditional quantum and two-key claims are [S2] V4.2, [S4] V4.4 and the forms of the external theorems specified in Chapters 9 and 10.

An audit-ready release should include: exact source Markdown and generated LaTeX, a visually inspected PDF, an editable word-processing edition where possible, the four separately identifiable companion manuscripts and their supplements, clean-output verification commands, pinned code commits, and a SHA-256 manifest. Each experiment must record its seed, sample law, trial count, implementation and output path. Historic Sage output remains labelled historic unless the exact Sage computation is newly executed.

The accompanying checks enumerate finite supports for the moments and rounding cells, compare exact integers and exercise the graph coupling on small finite domains. The proof of the quantum reduction uses the oracle identities and [R2].

## A consolidated decision table

The following table is the minimal truthful executive summary of C1-K4 in this edition.

| Question | Answer in this edition |
|---|---|
| Are the public-key and ciphertext algorithms completely specified? | Yes, for the research ideal-oracle profile. |
| Is honest decryption perfectly correct? | No; an explicit event proves nonzero failure. |
| Is the average worst-message honest failure small? | Yes, an exact bound is proved. |
| Is IND-CPA unconditional? | No, it depends on two precise Module-LWE decision gaps. |
| Is the classical ROM route completely general? | No, the stated experiment is single-user and challenge-first. |
| Is a coherent QROM route described? | Yes, conditional in the single-user, fresh, key-first model of Theorem 9.6. |
| Is the sharper FFP-NG reduction established? | No; applicability of [R5] is unresolved. |
| Do the reported attacks prove a minimum security level? | No. |
| Is SHAKE256 instantiation proved equivalent to the ideal oracles? | No. |
| Can the candidate be called *certified PQ-128*? | No. |

The legitimate contribution is not an unearned security label. It is the preservation of exact arithmetic and dependent remainders all the way to clearly delimited reductions and adversarial models. Chapter 12 uses this ledger to state the final conclusions, limits and next research obligations.


# Conclusions, Open Problems, and the Research Program Beyond C1-K4

## What this thesis has established

The central argument of this thesis begins with the identity $[z]_P=DF_D(z)+[z]_D$ for $D\mid P$. Its quotient seems simple because a remainder has disappeared from the displayed value. The remainder has not disappeared from the probability space. When quotient values enter later bilinear computations, the discarded coordinates continue to influence carries, errors, and correlations. In C1-K4 the effect is visible in three places: key-generation rounding, ciphertext compression, and randomized lifting of the public quotient. Keeping all three remainders in the same probability experiment is the decisive mathematical discipline of the construction.

Chapters 2--5 established the exact foundations and one nontrivial correctness result. With the frozen ring, module rank, modulus and CBD1 laws, honest decryption obeys

$$
z\equiv2048\mu+
(e_0+\rho-r_0)^Tr+(r_1-e_1)^Ts+e_2-r_2
\pmod{4096}.
$$

The dependent-remainder moment proof yields an average-over-keys worst-message bound below $2^{-212}$. A distinct positive-probability event establishes that correctness is not perfect: $\delta_{wc}\ge2^{-8192}$. Together these claims are more informative than a Gaussian approximation or a simulation that happens to show no failures. The lower bound prevents future reductions from casually discarding the correctness term. The upper bound controls it within its precise quantifier order.

Chapters 6--8 moved to cryptography. They exhibited the two decision-source interfaces $\mathrm{DMLWE}_{4,4}^{\mathrm{CBD1}}$ and $\mathrm{DMLWE}_{4,5}^{\mathrm{CBD1}}$, with a shared secret in the latter's five rows and an exact relation between the public quotient and its retained lift. These are conditional security transfers, not hardness results for arbitrary LWE. The classical Fujisaki--Okamoto construction uses a compact 1313-byte public key, a 1760-byte ciphertext, a fresh uniform 256-bit encapsulation message, an exact re-encryption test, and a stable implicit-rejection fallback. Within the single-user, challenge-first classical-ROM game established in Chapter 8, the security implication has explicit source and correctness terms. The hypothesis that the exact sources are hard remains external.

Chapters 9--10 confronted the quantum and failure-finding interfaces. A fresh, key-first quantum random-oracle experiment allows the matrix oracle to be installed before the adversary's first quantum query, rather than trying to reprogram a previously observed oracle. The ciphertext-dependent KDF and fallback use one underlying hash family; a coherent graph simulation of that fact is crucial. The conservative conditional QROM bound imported from [S2] combines quantum passive-security gaps, the spreadness estimate, an extractable-oracle failure search bound and a reference-transform theorem. The sharper same-tape second moment from [S4] gives $\mathbb E[p_m(\omega)^2]<2^{-247}$ under two independent keys. Neither result is a completed reduction from a standard worst-case lattice problem to the exact compact KEM.

The synthesis of Chapter 11 is deliberately less dramatic than a headline exponent. Each claim is assigned an experiment, assumptions and evidence. A cost estimate near $2^{206.211}$ for one specified adapted coded-dual model is meaningful evidence for that attack model, but not a theorem that every classical or quantum adversary requires that many steps. Our objective is an auditable mathematical case study, not a numerical security certification.

## The main contribution and its limits

The contribution is the integration of several separately difficult interfaces under a single frozen algorithmic specification. The exact error identity was not replaced by independent artificial noise. The CPA source reductions preserve the complete joint distribution. The FO analysis retains a nonzero failure probability. The quantum analysis treats oracle access as a coherent operation and distinguishes a preselected oracle patch from adaptive quantum programming. The FFP analysis separates independent-key rare-event search from public-key-dependent tape selection. Finally, the cryptanalytic evidence reports its search domain and provenance.

There are important boundaries to this claim. Euclidean division, Toeplitz/negacyclic algebra, Module-LWE and the FO family are established foundations; the thesis does not claim their invention. The new C1-K4-specific claims concern the application of those foundations to the specified construction, the dependent-remainder correctness estimate, the exact source-game coupling, and the numerical/modeling evidence. The second-moment argument is an imported companion result with its own hypotheses. The QROM theorem in Chapter 9 is a conditional restatement and derivation from [S2] and [R2], under the oracle and resource assumptions stated in Theorem 9.6.

A good cryptographic conclusion should also say what it does *not* establish. This manuscript does not prove the quantum computational hardness of either exact source distribution, the security of SHAKE256 as a concrete substitute for three independent ideal oracles, a concrete 128-bit post-quantum work factor, resistance to all structural/2-adic attacks, multi-user security, superposition decapsulation security, or security against shared prior random-oracle advice. It does not provide a hardened, constant-time, independently tested software implementation.

## Six specific open problems

### Problem 12.1 — Source hardness for the exact small-secret module laws

The passive and quantum reductions terminate in $\mathrm{DMLWE}_{4,4}^{\mathrm{CBD1}}$ and $\mathrm{DMLWE}_{4,5}^{\mathrm{CBD1}}$. A useful next result would be a rigorously quantified hardness reduction or a controlled cryptanalysis of these exact distributions, with $R=\mathbb Z_{4096}[x]/(x^{256}+1)$ and sample counts fixed. The reduction must state whether it is classical or quantum, the law of the public operator, the secret, the additive error, the allowed number of rows, and the advantage/runtime loss. Generic LWE theorems cannot be imported by merely expanding the 1024 secret coefficients into a vector.

**Closure criterion:** a theorem with explicit source and target distributions and resource accounting, or a reproducible counterexample/attack invalidating a claimed equivalence.

### Problem 12.2 — Exact FFP-NG hypothesis matching

The newer work of Majenz--Sisinni [R5] is relevant because it studies Ring/Module-LWE and centered-binomial-like noise. The missing mathematical bridge is to show whether its assumptions match the real C1-K4 quotient $b=F_4(As+e_0)$, the conditional uniform lift $v=4b+\rho$, the two $F_2$ ciphertext compressions and the common ephemeral $r$. This could produce a sharper failure contribution than the coarse $152(u+1)^2\delta_{wc}$ route. It could also reveal a precise obstruction. Neither outcome should be presumed in advance.

**Closure criterion:** a theorem-by-theorem correspondence table followed by a reduction with the exact FFP-NG game, or a documented mismatch that identifies a genuinely missing lemma.

### Problem 12.3 — Complete augmented quantum resource budgets

The QROM source gaps refer to algorithms generated by a transformation, not to the original attacker's nominal query count. A concrete comparison needs a reversible implementation of the oracle graph test, full matrix expansion, encryption arithmetic, extraction, uncomputation and classical decapsulation simulation, together with quantum gate, depth, memory and query budgets. The present symbolic overhead is useful for logic but insufficient for a cost-conversion claim.

**Closure criterion:** an independently checked gate/resource account for the composed reduction, and quantitative source-advantage bounds at those exact augmented resources.

### Problem 12.4 — From an ideal oracle to a real expander

The 32-byte seed is modeled by an ideal random matrix oracle with independent raw 512-byte responses. Instantiating that oracle with SHAKE256 or another concrete primitive would not make the complete matrix uniform in the information-theoretic sense. The correct statement would need a suitable cryptographic assumption, a provable pseudorandom generation construction, or an explicitly distinguished implementation assumption. Domain separation alone does not prove the desired distributional replacement.

**Closure criterion:** a precisely formulated computational replacement game and a justified advantage bound for the concrete generator, including the high raw-word bits visible at its oracle interface.

### Problem 12.5 — Broader games: pre-challenge, multi-user, shared-oracle advice

The classical chapter identifies a bounded pre-challenge ciphertext collision term but does not equate that observation with a complete adaptive pre-challenge CCA proof. The quantum chapter assumes a key-first adversary, fresh oracles and classical decapsulation. A natural extension would allow genuinely adaptive pre-challenge decapsulation, several correlated challenge public keys, or advice based on a previously sampled random oracle. Each extension changes what can be patched or conditioned upon and how failures and collisions accumulate.

**Closure criterion:** a separate game statement and complete reduction for each extension, with no silent inheritance from the narrower single-user experiment.

### Problem 12.6 — Adversarial cryptanalysis and implementation evidence

The existing MATZOV and coded-dual searches are documented points, not a complete minimum over all attacks. More exhaustive structured, 2-adic, hybrid, rotation-aware and quantum analyses should test the exact four- and five-row instances. A trustworthy practical KEM would in parallel require constant-time encoding/decoding, malformed-input fuzzing, deterministic cross-platform tests, correct fallback handling, randomness health checks and a side-channel review.

**Closure criterion:** pinned code and search domains for new attack results, together with independent implementation audits. No test campaign, even a broad one, can by itself prove universal absence of attacks.

## Toward a versioned, reproducible edition

The source editions are identified in the bibliography. The reproducibility dossier contains hashes of the inputs and corrected manuscript, the fresh calculations, their commands and the historical status of the Sage result. Versioning the experiment matters: changes in the decoder, oracle interface or key distribution can change a proof even when the parameter sizes stay fixed.

## Final statement

This thesis argues that **mathematical exactness and security claims must travel together**. The quotient–remainder calculus can be exact while the hardness of its distributions is assumed. A small decryption-failure parameter can be rigorously bounded while the failure-finding game remains a separate adversarial problem. A coherent QROM security implication can be meaningful while the quantum hardness of its source remains unproved. A cryptanalytic estimate can be reproducible while saying nothing universal about the minimum cost of an undiscovered attack.

C1-K4 is therefore best described as a carefully specified and mathematically analyzable research candidate. Its principal value is an explicit route from exact modular arithmetic to conditional cryptographic experiments, accompanied by precise unsolved questions. The scientific task of turning those conditional statements into a robust, concretely assessed system is **future work**. A claim of “PQ-128 certified” would be premature. The research program remains valuable precisely because its boundaries have been made visible enough to be challenged, reproduced and improved.


# References and source manuscripts {-}


**[R1]** Dennis Hofheinz, Kathrin Hövelmanns, Eike Kiltz. “A Modular Analysis of the Fujisaki–Okamoto Transformation.” *TCC 2017*, LNCS 10677, pp. 341–371; corrected full version, 2 November 2021, Definition 2.1, Theorems 3.2 and 3.4 and errata, IACR ePrint 2017/604. <https://eprint.iacr.org/2017/604>

**[R2]** Kathrin Hövelmanns, Andreas Hülsing, Christian Majenz. “Failing Gracefully: Decryption Failures and the Fujisaki–Okamoto Transform.” *ASIACRYPT 2022*, LNCS 13794, pp. 414–443. Published Lemma 4, (30), p. 439. Full version, arXiv:2203.10182v1, Lemma 5 (51), p. 34 and Corollary 8 (69)–(70), pp. 37–38. <https://doi.org/10.1007/978-3-031-22972-5_15> · <https://arxiv.org/abs/2203.10182v1>

**[R3]** MATZOV. “Report on the Security of LWE: Improved Dual Lattice Attack.” Version 2, 2022. <https://doi.org/10.5281/zenodo.6493704>

**[R4]** Christian Majenz, Fabrizio Sisinni. “Provable Security Against Decryption Failure Attacks from LWE.” *CRYPTO 2024*, LNCS 14921, pp. 456–485. <https://doi.org/10.1007/978-3-031-68379-4_14>

**[R5]** Christian Majenz, Fabrizio Sisinni. “Provable Decryption Failure Security for Practical Lattice-Based PKE.” IACR ePrint 2025/1997, preprint, accepted at SAC 2026. The comparison in Section 10.8 refers to the ePrint version. <https://eprint.iacr.org/2025/1997>

**[R6]** Jianhua Hou, Haodong Jiang, Tabitha Ogilvie. “Careful with the Ring! Concrete Hardness Gaps Between LWE and MLWE.” Full version (a), T. Ogilvie, ePrint 2026/279. *CRYPTO 2026*, LNCS 16802, pp. 462–497. <https://doi.org/10.1007/978-3-032-35377-1_15>

**[R7]** *CodedDualAttack*, branch eprint-2026-279, commit 7cce6a557242 (full identifier in Section 7.5). [Pinned source](https://github.com/identitymapping/CodedDualAttack/tree/7cce6a5572420004da31be58f26954cedfc885d0).

**[R8]** Kévin Carrier, Charles Meyer-Hilfiger, Yixin Shen, and Jean-Pierre Tillich. *Assessing the Impact of a Variant of MATZOV’s Dual Attack on Kyber*. CRYPTO 2025, LNCS 16000, pp. 444–476. <https://doi.org/10.1007/978-3-032-01855-7_15>.

**[R9]** Abhishek Banerjee, Chris Peikert and Alon Rosen. “Pseudorandom Functions and Lattices.” EUROCRYPT 2012. <https://doi.org/10.1007/978-3-642-29011-4_42>.

**[R10]** Eiichiro Fujisaki and Tatsuaki Okamoto. “Secure Integration of Asymmetric and Symmetric Encryption Schemes.” CRYPTO 1999, LNCS 1666, pp. 537–554.

**[R11]** Kathrin Hövelmanns and Christian Majenz. “A Note on Failing Gracefully: Completing the Picture for Explicitly Rejecting Fujisaki–Okamoto Transforms Using Worst-Case Correctness.” *PQCrypto 2024*, LNCS 14772, pp. 245–265. <https://doi.org/10.1007/978-3-031-62746-0_11>.

**[S1]** Samir Bouftass. “C1-K4-P12-CBD1: Exact Correctness, Conditional Security, and Reproducible Attack Estimates.” Corrected preprint v1.1, 18 November 2026. Research assistance: ChatGPT (OpenAI); publication review in progress.

**[S2]** “C1-K4 in the QROM: Structural Bounds and a Conservative Conditional Reduction.” Companion manuscript, revision V4.2, 13 November 2026. Current source document, not a claimed externally refereed publication.

**[S3]** “A Quotient–Remainder Calculus for Modular Rounding.” Companion manuscript, revision V4.3, 13 November 2026. Current source document.

**[S4]** “Find-Failing-Plaintext Analysis of C1-K4-P12-CBD1.” Companion manuscript, revision V4.4, 13 November 2026. Current source document.
