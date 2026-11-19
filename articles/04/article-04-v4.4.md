# Article 4 — Find-Failing-Plaintext Analysis of C1-K4-P12-CBD1

**Revision V4.4 — 13 November 2026 — scoped mathematical and reproducibility closure**

**Samir Bouftass**

*Research assistance: ChatGPT (OpenAI). Samir Bouftass is responsible for the scientific content of this archival edition.*

## Abstract

This article studies decryption failures of C1-K4-P12-CBD1 and their role in the FFP-NK and FFP-NG games. Its main contribution is a bound on the second moment of the failure probability over independent keys when encryption randomness is shared.

For the frozen construction over
\[
R=\mathbb Z_{4096}[x]/(x^{256}+1),\qquad k=4,
\]
the exact honest decryption error is
\[
\varepsilon=(e_0+\rho-r_0)^Tr+(r_1-e_1)^Ts+e_2-r_2,
\]
with the actual dependent remainders
\[
r_0=[As+e_0]_4,\qquad r_1=[A^Tr+e_1]_2,\qquad r_2=[v^Tr+e_2]_2.
\]

Article 1, revision V4.1, Sections 4–5 and Corollary 4, establishes
\[
\delta_{wc}<2^{-212}.
\]
The main theorem here is a same-randomness, two-independent-key moment bound. For
\[
p_m(\omega)=
\Pr_K[
\operatorname{Dec}_{sk}(\operatorname{Enc}_{pk}(m;\omega))\ne m
],
\]
we prove
\[
\mathbb E_\omega[p_m(\omega)^2]<2^{-247},
\qquad
\sigma_{\delta ik}<2^{-123.5}.
\]
The proof uses Cauchy–Schwarz to handle distinct output coefficients before factorizing the diagonal two-key case; this avoids a false independence assumption between permuted uses of the common encryption tape.

The mean-only analysis now distinguishes an exact single-threshold bound from an integrated tail bound. For \(a=152(q+1)^2\delta_{ik}\), the latter is \(a(1+\ln(1/a))\) when \(0<a<1\). We also record the stronger conservative alternative using Article 1's averaged worst-message parameter:
\[
\operatorname{Adv}^{\mathrm{FFP\text{-}NK}}
\le\min\{1,152(q+1)^2\delta_{wc}\}.
\]
This follows by applying the search lemma with a fixed hidden target key and averaging afterwards. These are failure-game probability bounds, not KEM security levels.

For FFP-NG, the literature status is also updated. Majenz and Sisinni, IACR ePrint 2025/1997, state an analysis for Ring/Module-LWE with a subgaussian family including centered binomials. This makes the earlier “Gaussian-only” obstruction obsolete. We have not established that their theorem applies unchanged to the exact C1-K4 compression, lifting, modulus, and failure interface, so FFP-NG remains an explicit matching obligation rather than a proved impossibility.

Structured-tape experiments remain experimental. The four public-key-dependent strategies tested at 128 paired trials each produced zero observed failures in either arm. Exact integer controls distinguish complete fixed tapes from resampled families.

The corrected status is therefore
\[
\boxed{
\text{proved failure bounds; sharper FFP-NG route remains open}.
}
\]

Article 2, revision V4.2, separately gives a conservative conditional QROM composition using the direct FFP-CPA bound from \(\delta_{wc}\). Its proof does not require this article's second moment or an FFP-NG reduction. Matching a sharper FFP-NG theorem and obtaining a concrete PQ-128 certificate remain distinct tasks.

---

## 1. Frozen setting

C1-K4-P12-CBD1 uses
\[
P=4096,\quad n=256,\quad k=4,\quad Q=4,\quad C=2,
\]
with coefficientwise CBD1 secrets and errors: \(\Pr[-1]=\Pr[1]=1/4\), \(\Pr[0]=1/2\). All key-secret and key-error coefficients are mutually independent; the encryption values \(r,e_1,e_2\) have independent CBD1 coefficients, and \(\rho\) has independent uniform coefficients in \(\{0,1,2,3\}\), independently of the key. The module products are over the negacyclic ring displayed in the abstract.

Write \([x]_d\) for the coefficientwise least nonnegative residue and \(F_d(x)=\lfloor[x]_{4096}/d\rfloor\). The exact construction is
\[
b=F_4(As+e_0),\quad v=4b+\rho,\quad
c_1=F_2(A^Tr+e_1),\quad
c_2=F_2(v^Tr+e_2+2048\mu),
\]
where \(\mu\in\{0,1\}^{256}\). Decryption computes
\[
z=[2c_2-(2c_1)^Ts]_{4096}
\]
and outputs bit one exactly when \(1024<z_i<3072\); both boundary ties decode to zero. These are Article 1 V4.1's conventions. The matrix is generated independently of the small secrets and errors. The bounds below eliminate its remainders by maxima and do not require matrix entries to be mutually independent.

An encryption tape can equivalently mean the complete tuple \((r,e_1,e_2,\rho)\) with this product law, or the uniform 2816-byte tape mapped to that tuple as specified in Article 1. The first 2304 bytes each supply the difference of their two low bits for CBD1, and the final 512 bytes each supply two independent two-bit lift values from their nibbles. All probabilities below use this honest pushforward law; a deliberately chosen tuple is not sampled with uniform weight over the tuple support.

The message does not affect the actual remainder \(r_2\) at fixed coins because \(C\mid P/2\). Hence the same containing failure event can be used for all messages. This does not assert that actual failure is message-independent: at the two decoding ties it depends on the message bit.

For an honest ciphertext,
\[
z\equiv(P/2)\mu+\varepsilon\pmod P,
\]
where
\[
\varepsilon=(e_0+\rho-r_0)^Tr+(r_1-e_1)^Ts+e_2-r_2.
\]

Every honest failure is contained in
\[
\mathcal F=\{\max_i|\varepsilon_i|\ge P/4\}.
\]

---

## 2. FFP-NK random variable

For a fixed message \(m\) and complete valid encryption tape \(\omega\), define
\[
p_m(\omega)
=
\Pr_K[
\operatorname{Dec}_{sk}(\operatorname{Enc}_{pk}(m;\omega))\ne m
].
\]

In the FFP-NK experiment considered here, the adversary outputs \(m\), its tape is the final oracle value at \(m\), and an independent target key is drawn for the failure test. The adversary does not choose that target key. Its success advantage is the failure probability, not half a distinguishing gap.

The independent-key error rate is
\[
\delta_{ik}
=
\max_m\mathbb E_\omega[p_m(\omega)].
\]

Define explicitly
\[
\delta_{wc}=\mathbb E_K\!\left[\max_m\Pr_\omega(F\mid K,m)\right].
\]
The maximum is inside this key expectation. Since Article 1 proves this averaged worst-message bound,
\[
\delta_{ik}\le\delta_{wc}<2^{-212}.
\]

The variance parameter is
\[
\sigma_{\delta ik}^2
=
\max_m
\operatorname{Var}_{\omega}[p_m(\omega)].
\]

The generic inequality \(0\le p\le1\) gives
\[
\sigma_{\delta ik}^2\le\delta_{ik}<2^{-212},
\]
hence
\[
\sigma_{\delta ik}<2^{-106}.
\]

The next section improves this bound.

---

## 3. Two-independent-key second-moment theorem

**Theorem 1 (second moment).** For every fixed message \(m\) under the laws in Section 1, \(\mathbb E_\omega[p_m(\omega)^2]<2^{-247}\). Consequently \(\sigma_{\delta ik}<2^{-123.5}\).

**Proof.** Fix \(u>1\). Take two independently generated key pairs \(K^{(1)},K^{(2)}\), but use the same encryption tape \(\omega\). For fixed \(m\),
\[
\mathbb E_\omega[p_m(\omega)^2]
=
\Pr_{\omega,K^{(1)},K^{(2)}}[F_1\wedge F_2].
\]

A direct factorization across two arbitrary output coefficients would be unjustified because distinct negacyclic coefficients reuse the same tape under different permutations. We therefore insert a Cauchy–Schwarz step.

For an output index \(i\) and sign \(\alpha\in\{\pm1\}\), define
\[
g_{i,\alpha}(\omega)
=
\mathbb E_K
u^{\alpha\varepsilon_i(K,\omega)}.
\]
For arbitrary \(i,j,\alpha,\beta\),
\[
\mathbb E_\omega
g_{i,\alpha}(\omega)g_{j,\beta}(\omega)
\le
\sqrt{
\mathbb E_\omega g_{i,\alpha}(\omega)^2\,
\mathbb E_\omega g_{j,\beta}(\omega)^2
}.
\]
Thus it suffices to bound the diagonal case: the same output coefficient and same sign, evaluated under two independent keys but the same encryption tape.

In that diagonal case, the paired tape coordinates coincide, and the conditional order can be chosen so that all factorizations are legitimate.

### 3.1 Final-coordinate factor

The remainder \(r_2\) depends on \(e_2\), so it is **not** fixed before averaging \(e_2\). Instead use a pointwise maximum inside that expectation. For two keys and signs \(\alpha,\alpha'\), define
\[
C_2(u)
=
\max_{\alpha,\alpha'}
\mathbb E_{E_2}
\max_{t,t'\in\{0,1\}}
u^{\alpha(E_2-t)+\alpha'(E_2-t')}.
\]

### 3.2 Ciphertext-error factor

Condition on both keys, the shared \(r,\rho\), and the corresponding hidden offsets. The coefficients of \(e_1\) are independent. For independent key-secret coefficients \(S,S'\sim\mathrm{CBD1}\),
\[
B_2(u)
=
\max_{\alpha,\alpha'}
\mathbb E_{S,S'}
\max_{c,c'\in\{0,1\}}
\mathbb E_{E_1}
u^{
\alpha S([c+E_1]_2-E_1)
+
\alpha'S'([c'+E_1]_2-E_1)
}.
\]
The intermediate coefficient factor is a function \(b_{\alpha,\alpha'}(S,S')\) of the two secret coefficients, not the already averaged scalar \(B_2\). It no longer depends on the shared ephemeral tape after maximizing the offsets. The expectation over \(S,S'\) is postponed until the key-generation/lifting factors have also had their hidden remainders removed. This order is essential.

### 3.3 Key-generation/lifting factor

Condition on both keys so that the two \(r_0\) values are fixed, then average the independent shared tape pairs
\[
(R,T)=(r_j,\rho_j).
\]
For two independent key-generation errors \(E,E'\sim\mathrm{CBD1}\),
\[
A_2(u)
=
\max_{\alpha,\alpha'}
\mathbb E_{E,E'}
\max_{t,t'\in\{0,1,2,3\}}
\mathbb E_{R,T}
u^{R(\alpha(E+T-t)+\alpha'(E'+T-t'))}.
\]

For clarity, the full conditioning order is as follows. For a fixed output index \(i\), put
\[
\pi_i(j)=(i-j)\bmod n,\qquad
\tau_i(j)=\begin{cases}+1,&j\le i,\\-1,&j>i.\end{cases}
\]
Then \((fg)_i=\sum_j\tau_i(j)f_{\pi_i(j)}g_j\). This explicitly identifies the signed permutations in each module component.

1. Hold both keys and \((r,e_1,\rho)\) fixed. Remove both final remainders pointwise, then average the shared \(e_{2,i}\); the cost is \(C_2\). The remainder cannot be held fixed while averaging \(e_{2,i}\).
2. With both keys and \((r,\rho)\) fixed, the offsets \([A^Tr]_2\) are fixed before averaging the independent coordinates of shared \(e_1\). Maximize those offsets after each coordinate expectation. This leaves a product of \(b_{\alpha\tau_i(j),\alpha\tau_i(j)}(s_{\pi_i(j)},s'_{\pi_i(j)})\), which still depends on the two secrets.
3. For each shared pair \((r_j,\rho_{\pi_i(j)})\), average its honest distribution with the two keys fixed; their \(r_0\) values are already fixed. Maximize the two remainder values *outside* that expectation. This leaves a product of functions \(a_{\alpha\tau_i(j),\alpha\tau_i(j)}(e_{0,\pi_i(j)},e'_{0,\pi_i(j)})\). These functions no longer depend on the secrets or either matrix.
4. Only now average all key-secret and key-error coefficients. They are independent across keys, module components and coordinates, and the two permutations are bijections. The remaining \(a\)- and \(b\)-products therefore separate. Each averaged factor is bounded by \(A_2\) or \(B_2\), respectively.

Here \(a\) and \(b\) are precisely the inner maxima in the displayed definitions of \(A_2\) and \(B_2\), before averaging \((E,E')\) or \((S,S')\). The outer maxima over signs give bounds uniform in every negacyclic sign. In particular, no independence between \(r_0\) and \(s\), or between the two error blocks before these maxima, is assumed. We obtain
\[
\mathbb E_\omega g_{i,\alpha}(\omega)^2
\le C_2(u)(A_2(u)B_2(u))^{kn}.
\]
By Cauchy–Schwarz, the same upper bound applies to arbitrary pairs \(i,j,\alpha,\beta\).

A union bound over \(n^2\) output-index pairs and four sign pairs, followed by exponential Markov at total threshold \(P/2=2048\), gives
\[
\boxed{
\mathbb E[p_m(\omega)^2]
\le
4n^2u^{-P/2}
(A_2(u)B_2(u))^{kn}C_2(u)
}.
\]

At
\[
u=\frac65,
\]
exact finite enumeration gives
\[
A_2=
\frac{99036386572201}{83980800000000},
\qquad
B_2=
\frac{489677}{480000},
\qquad
C_2=
\frac{3721}{2500}.
\]

With \(n=256\), \(kn=1024\), and \(P/2=2048\),
\[
4\cdot256^2
(A_2B_2)^{1024}
C_2
\left(\frac65\right)^{-2048}
<2^{-247}.
\]

Therefore
\[
\boxed{
\mathbb E_\omega[p_m(\omega)^2]<2^{-247}
}
\]
and, since variance is at most the second moment,
\[
\boxed{
\sigma_{\delta ik}<2^{-123.5}.
}
\]

The displayed bound is a rational number. Its strict comparison with \(2^{-247}\) is certified by an integer numerator/denominator comparison; no floating-point logarithm is used in that test. Its base-two logarithm is approximately
\[
-247.024841645.
\]

---

## 4. Interpreting the variance parameter

The variance theorem in [HHM22, full version, Theorem 11] contains a leading summand of the form
\[
\sqrt{c_{\mathrm{search}}q_G\sigma_{\delta ik}},
\qquad c_{\mathrm{search}}=304.
\]

Using only the simplified leading term
\[
3\sqrt{304\,q_G\,2^{-123.5}},
\]
the corresponding negative base-two logarithms are approximately:

| \(q_G\) | bound bits |
|---:|---:|
| \(1\) | 56.0 |
| \(2^{16}\) | 48.0 |
| \(2^{32}\) | 40.0 |
| \(2^{64}\) | 24.0 |

These numbers are evaluations of one positive summand only. Omitting the other terms does **not** produce a complete upper bound on advantage. The symbol \(\sigma_{\delta ik}\) denotes a standard deviation: the second moment bound \(2^{-247}\) cannot be substituted for it. No variance-based KEM bound is instantiated here; Section 5 provides directly stated failure-game upper bounds.

---

## 5. Mean-only search bounds and the conservative alternative

Use the extractable-oracle search lemma [HHM22, full version, Lemma 5, equation (51), p. 34; published version, Lemma 4, equation (30), p. 439]. For a relation \(\mathcal R\subseteq\mathcal X\times\mathcal Y\), set
\[
\Gamma_{\mathcal R}=\max_x\#\{y:(x,y)\in\mathcal R\}.
\]
An adversary making \(q\) queries to \(\mathrm{eCO.RO}\) before its final output satisfies
\[
\Pr[(x,\mathrm{eCO.RO}(x))\in\mathcal R]
\le D_q\frac{\Gamma_{\mathcal R}}{|\mathcal Y|},
\qquad D_q=152(q+1)^2.
\]
The additional final oracle evaluation is accounted for by \(q+1\). The lemma allows extraction queries, but it is a statement in its specified extractable-oracle model, with the fixed auxiliary function and initial state independent of the fresh oracle. Other oracle calls in a KEM proof must first be translated into this budget. Uniform oracle outputs are mapped to the honest tape law from Section 1.

### 5.1 Threshold and integrated bounds from \(\delta_{ik}\)

Let \(F_{\mathrm{out}}=p_m(\mathrm{eCO.RO}(m))\) for the output of one fixed adversary. For every \(m\), Markov gives
\[
\Pr_\omega[p_m(\omega)\ge t]\le\delta_{ik}/t.
\]
Thus the relation \(p_m(\omega)\ge t\) has density at most \(\delta_{ik}/t\), uniformly in \(m\). The search lemma yields
\[
\Pr[F_{\mathrm{out}}\ge t]\le\min\{1,a/t\},
\qquad a=D_q\delta_{ik}.
\]
This works for adaptively selected messages because the relation density uses a maximum over its inputs. The adversary is unchanged as \(t\) varies.

For \(0<a<1\), a single threshold \(t=\sqrt a\) and
\(F_{\mathrm{out}}\le t+(1-t)\mathbf1_{F_{\mathrm{out}}\ge t}\) give
\[
\mathbb E F_{\mathrm{out}}\le 2\sqrt a-a.
\]
Integrating the tails of the *same* random variable gives the stronger bound
\[
\mathbb E F_{\mathrm{out}}\le
\begin{cases}
a(1+\ln(1/a)),&0<a<1,\\
1,&a\ge1,\\
0,&a=0.
\end{cases}
\]
Since the target key in FFP-NK is fresh and independent, its failure success probability equals \(\mathbb E F_{\mathrm{out}}\). The earlier expression involving \(2q\sqrt{304\delta_{ik}}\) was only an asymptotic diagnostic and did not track the final oracle evaluation consistently; it is replaced by the exact threshold expression above.

### 5.2 Direct bound using \(\delta_{wc}\)

The averaged worst-message parameter already proved in Article 1 gives a simpler alternative. Sample the target key \(K\) before the experiment and keep it hidden; this is equivalent to drawing it after the output. For a fixed key define
\[
d(K)=\max_m\Pr_\omega(F\mid K,m).
\]
The relation \(\mathcal R_K=\{(m,\omega):\text{decryption fails under }K\}\) has density at most \(d(K)\). Apply the same search lemma for each fixed \(K\), then average:
\[
\operatorname{Adv}^{\mathrm{FFP\text{-}NK}}
\le\mathbb E_K\min\{1,D_qd(K)\}
\le\min\{1,D_q\delta_{wc}\}.
\]
This step uses \(\mathbb E_K\max_m\), not \(\max_m\mathbb E_K\). Replacing \(\delta_{wc}\) here by \(\delta_{ik}\) without another argument would be unjustified. The auxiliary oracle model and any side information must remain independent of this hidden target key and of the fresh oracle as required by the search lemma. In the actual FFP-NK game the target key is fresh, so this conditioning is legitimate.

Taking the common certified upper bound \(\delta_{ik},\delta_{wc}\le2^{-212}\):

| \(q\) | \(-\log_2\) threshold upper bound | \(-\log_2\) integrated upper bound | \(-\log_2\) direct upper bound |
|---:|---:|---:|---:|
| \(1\) | 100.376 | 195.607 | 202.752 |
| \(2^{16}\) | 85.376 | 165.836 | 172.752 |
| \(2^{32}\) | 69.376 | 134.129 | 140.752 |
| \(2^{64}\) | 37.376 | 70.992 | 76.752 |

The last column uses a stronger hypothesis, available for C1-K4. It does not claim to follow from \(\delta_{ik}\) alone, nor to be optimal. None of these columns gives KEM security bits. In particular, an FFP-NK bound is not itself a bound for key-dependent FFP-CPA. Article 2 V4.2 gives the latter by a separate fixed-key application using \(\delta_{wc}\), and states the full conservative composition and translated query counts in its Theorem 4. Its theorem does not depend on resolving Section 7 below.

---

## 6. Generic bad tapes: experimental evidence only

Experiments found highly structured tapes for which decryption failures can become common over fresh keys. A representative family uses several all-one module components together with highly structured lift values.

The interpretation must distinguish two experimental objects:

1. a **complete fixed tape**, where every ephemeral coefficient and lift coefficient is frozen before testing keys;
2. a **family of tapes**, where a procedure resamples some coordinates between trials.

Only the first directly estimates the fixed-\(\omega\) quantity \(p_m(\omega)\).

The V4.4 supplementary package retains the two experimental series **separately**:

1. `original_experiments/ffp_structured_tape_experiment.py` and `original_experiments/FFP_EXPERIMENT_RESULTS.json`, reproducing the earlier family-of-tapes experiment with seed `20261113`; its FFT products are now checked individually against exact integer convolution;
2. `independent_controls/ffp_independent_fixed_tape_controls.py` and `independent_controls/INDEPENDENT_FIXED_TAPE_RESULTS.json`, using integer negacyclic convolution, zero message, explicit fixed/fresh lift modes, seed `2026111302`, and 128 trials per row.

The implementations and tape conventions differ and are not merged into one statistical sample. The first series resamples some tape coordinates and therefore probes a family; the second explicitly distinguishes fixed-zero-lift tapes from fresh-uniform-lift controls.

The reproduced counts are:

| Experiment | Active all-one components | Failures / trials, first arm | Second arm |
|---|---:|---:|---:|
| Original family, active lifts zero | 2 | 0/128 | not run |
| Original family, active lifts zero | 3 | 128/128 | not run |
| Original family, active lifts zero | 4 | 128/128 | not run |
| Complete fixed tape, all lifts zero | 2 | 0/128 | 0/128 |
| Complete fixed tape, all lifts zero | 3 | 128/128 | 128/128 |
| Complete fixed tape, all lifts zero | 4 | 128/128 | 128/128 |
| Fresh uniform lift control | 2, 3, or 4 separately | 0/128 each | 0/128 each |

For the controls, both arms use independent keys with exactly the same tape in each pair; neither tape was selected from a public key. Calling the first arm “matching” is only a historical JSON label and does not make it an FFP-NG attack. These counts are finite observations, not probabilities equal to zero or one.

The observed failures demonstrate bad tuples in these families. They do not estimate the mass of such tuples under honest random-oracle output. A family with nonrandom \(r\) cannot be used to estimate honest decryption failure. The analytical moment bounds, not these Monte Carlo counts, address honest tape probabilities.

---

## 7. FFP-NG: corrected state of the art

In the one-pair FFP-NG game, a bit \(b\) selects the target key: it is the key matching the public key given to the adversary when \(b=0\), and a fresh independent key when \(b=1\). The adversary chooses a valid pair \((m,\omega)\), receives its failure bit under the selected key, and guesses \(b\). For a fixed pair-generation rule and a decision rule that uses only the failure bit with a fixed orientation, the better of the two orientations has bias \(\tfrac12|p_0-p_1|\). A general adversary may also use its public-key-dependent view in the final guess, so this marginal statistic is not an upper bound on its advantage. This factor of two differs from a raw probability gap and from FFP-NK's success probability.

The earlier version of this article emphasized a PVW/discrete-Gaussian reduction and stated too broadly that the known proof technology was Gaussian-specific. That statement is outdated.

Christian Majenz and Fabrizio Sisinni, *Provable decryption failure security for practical lattice-based PKE*, IACR ePrint 2025/1997, state that they generalize FFP-NG analysis to widely used Ring-LWE and Module-LWE schemes and consider a family of subgaussian distributions including centered binomials.

This is relevant to C1-K4, but applicability is not automatic. The present work has not verified that the theorem's exact hypotheses match:

- C1-K4's coefficientwise CBD1 source laws;
- the exact module and modulus;
- its compression and lifting maps;
- the dependent remainders entering the failure predicate;
- the compact wrapper's FFP interface.

Accordingly,
\[
\boxed{
\text{FFP-NG for the exact C1-K4 interface remains open in this article.}
}
\]

CBD1 has the exact splitting identity
\[
\mathrm{CBD1}
\overset d=
B_+ + B_-,
\]
where
\[
B_+\leftarrow\{0,1\},
\qquad
B_-\leftarrow\{-1,0\}
\]
are independent and uniform. This notation is used consistently throughout this revision. The splitting identity is a distributional fact, not a new FFP-NG reduction.

---

## 8. FFP-NG attack search

Exactly four public-key-dependent rules are implemented: midpoint signs (`signb`), parity (`parityb`), a three-valued threshold (`threshold`), and signs of local differences (`difference`). The threshold rule may produce zeros; no separate sparse-selection search is implemented or claimed.

Each rule uses 128 independent key pairs and the zero message. The tape is derived from \(pk_0\), then held identical for its two key evaluations. Each of the four rules gave \(0/128\) failures on each arm; the recorded empirical bias \(|f_0-f_1|/(2\cdot128)\) is zero. This is a plug-in statistic, not a bound on true advantage. The samples are paired, not two pooled independent sets of 256 observations. No negligible-probability, universal no-attack, or security conclusion follows from this sample size.

The independent hostile audit also evaluated explicitly fixed tapes and integer-convolution controls. In V4.4 those controls remain delivered under `independent_controls/` with their script, seed, protocol, and JSON output. They remain separate from the original experiments under `original_experiments/` because the tape conventions and implementations differ.

The correct conclusion is:

\[
\boxed{
\text{no tested elementary strategy produced an observed FFP-NG advantage at the stated sample size.}
}
\]

---

## 9. Final classification

### PROVED

1. The exact C1-K4 failure geometry is retained.
2. \(\delta_{ik}<2^{-212}\).
3. The Cauchy–Schwarz-repaired two-key proof gives
   \[
   \mathbb E[p_m(\omega)^2]<2^{-247}.
   \]
4. Consequently,
   \[
   \sigma_{\delta ik}<2^{-123.5}.
   \]
5. The exact threshold and integrated mean-only FFP-NK bounds, including
   \[
   a(1+\ln(1/a))
   \]
   follow in the stated search-lemma model for \(0<a<1\).
6. The direct failure-game bound \(\min\{1,152(q+1)^2\delta_{wc}\}\) holds under the independent-target-key model of Section 5.2.

### EXPERIMENTALLY OBSERVED

1. Structured generic bad tapes exist in the tested families.
2. No tested elementary public-key-dependent strategy produced an observed FFP-NG advantage at the stated sample size.

### OPEN

1. Exact applicability of the newer Ring/Module-LWE FFP-NG theorem to C1-K4.
2. A complete FFP-NG reduction specialized to the exact compact interface, if the published theorem does not directly instantiate.
3. A practical FFP-NG attack.
4. A sharper end-to-end composition using a specialized FFP-NG theorem. The conservative conditional composition already established in Article 2 V4.2 is not an open obligation here.
5. A concrete PQ-128 QROM certificate for the compact 1760-byte KEM.

---

## 10. Verdict

\[
\boxed{
\textbf{ARTICLE 4 — SCOPED INTERNAL CLOSURE / PARTIAL FFP ANALYSIS}
}
\]

The second-moment proof is closed in its stated model, with explicit conditioning and an exact rational certificate. The finite-query threshold bound and probability-game conventions are corrected. The experiments are reproducible from clean output directories and their scope is explicit. Articles 1–3 are not modified by this revision.

This is an internal mathematical review, not external peer review or a claim of PQ-128 security. No impossibility theorem is proved. The exact applicability of a sharper FFP-NG theorem remains open; the separate conservative conditional QROM composition in Article 2 V4.2 is acknowledged.

## References

- Samir Bouftass and ChatGPT (OpenAI), *C1-K4-P12-CBD1: Exact Correctness, Conditional Security, and Reproducible Attack Estimates*, Article 1, revision V4.1, 13 November 2026. Sections 3–5 define the exact laws, error identity, and averaged worst-message bound. Companion manuscript included unchanged under `dependencies/` in the standalone supplement.
- Samir Bouftass and ChatGPT (OpenAI), *C1-K4 in the QROM: Structural Bounds and a Conservative Conditional Reduction*, Article 2, revision V4.2, 13 November 2026. Theorem 4 states the conditional conservative composition. Companion manuscript included unchanged under `dependencies/`.
- Kathrin Hövelmanns, Andreas Hülsing, and Christian Majenz, *Failing Gracefully: Decryption Failures and the Fujisaki–Okamoto Transform*, ASIACRYPT 2022, LNCS 13794, pp. 414–443. https://doi.org/10.1007/978-3-031-22972-5_15. Full version: https://arxiv.org/abs/2203.10182v1, Lemma 5, equation (51), p. 34; Theorem 11, p. 33. The corresponding published search lemma is Lemma 4, equation (30), p. 439; numbering differs.
- Christian Majenz and Fabrizio Sisinni, *Provable decryption failure security for practical lattice-based PKE*, Cryptology ePrint Archive 2025/1997, 2025. https://eprint.iacr.org/2025/1997. Only the public notice and abstract are used for the literature update; this article does not claim that the full theorem hypotheses have been matched to C1-K4.
