# C1-K4-P12-CBD1: Exact Correctness, Conditional Security, and Reproducible Attack Estimates

**Samir Bouftass**  
**Article 1 — revised preprint v1.1 — 18 November 2026**  
*Research assistance: ChatGPT (OpenAI); responsibility and authorship statement at the end.*

## Abstract

We specify and analyse C1-K4-P12-CBD1, a fixed module-based encryption profile over \(\mathbb Z_{4096}[x]/(x^{256}+1)\). Public-key rounding, ciphertext compression, and randomized lifting introduce dependent remainders. An exact error identity and an ordered exponential-moment argument retain those dependencies and establish an averaged worst-message decryption-failure probability \(2^{-8192}\le\delta_{wc}<2^{-212}\). The upper bound is certified by finite rational arithmetic; the lower bound follows from an explicit positive-probability failure event.

Two joint-distribution simulations reduce classical IND-CPA security to the exact decisional source problems \(\mathrm{DMLWE}_{4,4}^{\mathrm{CBD1}}\) and \(\mathrm{DMLWE}_{4,5}^{\mathrm{CBD1}}\). A 1313-byte public key uses a seeded matrix in an explicitly stated ideal-oracle model; a ciphertext contains 1760 bytes. We provide an explicitly scoped classical random-oracle KEM argument with declared game and query-budget conditions. Quantum random-oracle composition is treated separately in a companion manuscript [5].

Reproducible cost-model evaluations give a MATZOV-style estimate of approximately \(2^{213.657}\) operations at the lowest point found in the documented search and a corrected rotation-aware coded-dual estimate of approximately \(2^{206.211}\). The latter requires both a CBD1 distribution adjustment and a search-bracket correction. These are estimated costs of specified attacks, not lower bounds on adversarial work. Neither the source assumptions, these estimates, nor the reduction-operation counts certify 128-bit security.

## 1. Scope and status of the claims

This article concerns one frozen profile. Its results have four distinct statuses: exact algebra and probability bounds; conditional reductions; model-dependent attack estimates; and implementation obligations. The distinctions are necessary because an attack estimate does not prove the difficulty assumed by a reduction.

The construction uses a module matrix with independent uniform ring entries in its abstract model. Expanding its multiplication into coefficients produces structured signed Toeplitz blocks. It does not produce a matrix of independent scalar LWE coefficients, and it is not the single overlapping Hankel source used in other MODDIV variants. No equivalence to those sources or to an arbitrary LWR instance is assumed.

The rank-four profile is specified here in full. Results for a rank-three predecessor are not its numerical certificates. The companion manuscripts [5] and [6] address quantum random-oracle composition and failure finding, respectively; [7] treats quotient–remainder arithmetic. These drafts do not supply an unconditional hardness theorem for the source distributions defined below, and none is imported into this article. This paper is a research preprint, not a security evaluation or an externally refereed publication.

## 2. Ring, parameters, and sampling laws

Let
\[
R=\mathbb Z_P[x]/(x^n+1),\qquad
(n,k,P,Q,C)=(256,4,4096,4,2).
\]
Polynomials are arrays of exactly \(n\) coefficients in increasing degree order. All residues \([z]_D\) are canonical, in \(\{0,\ldots,D-1\}\). For \(D\mid P\), define the coefficientwise map
\[
F_D(z)=\left\lfloor\frac{[z]_P}{D}\right\rfloor.
\]
Let \(\chi=\mathrm{CBD}_1\), the difference of two independent fair bits:
\[
\Pr[E=-1]=\Pr[E=1]=\tfrac14,\qquad \Pr[E=0]=\tfrac12.
\]
Sampling from a power of \(\chi\) means independent sampling of every coefficient. Key-generation and encryption draws are independent unless a dependency is explicitly indicated.

For integer coefficient arrays, write \(a\star b\) for negacyclic multiplication before reduction modulo \(P\). Its coefficients satisfy
\[
(a\star b)_i=\sum_{j=0}^{i}a_jb_{i-j}
-\sum_{j=i+1}^{n-1}a_jb_{n+i-j}.
\]
The minus sign follows from \(x^n=-1\). A multiplication block has entries \(a_{i-j}\) when \(i\ge j\), and \(-a_{n+i-j}\) otherwise. For any matrix \(A\in R^{k\times k}\),
\[
(As)^Tr=(A^Tr)^Ts.
\]
Indeed, both sides expand as \(\sum_{u,v}A_{uv}s_vr_u\). Neither symmetry nor invertibility of \(A\) is needed.

## 3. The base public-key encryption scheme

### 3.1 Key generation

Sample \(A\) uniformly from \(R^{4\times4}\), with independent ring entries. Sample \(s,e_0\leftarrow\chi^{1024}\), viewed as four polynomials each. Compute
\[
b=F_4(As+e_0),\qquad pk=(A,b),\qquad sk=s.
\]
The vector \(b\) has 1024 coefficients in \(\{0,\ldots,1023\}\). Section 7 specifies the separate seeded representation of \(A\).

### 3.2 Encryption

For \(\mu\in\{0,1\}^{256}\), independently sample
\[
r,e_1\leftarrow\chi^{1024},\qquad e_2\leftarrow\chi^{256},\qquad
\rho\leftarrow U(\{0,1,2,3\}^{1024}).
\]
Set
\[
v=4b+\rho,\qquad
c_1=F_2(A^Tr+e_1),\qquad
c_2=F_2(v^Tr+e_2+2048\mu).
\]
Return \(c=(c_1,c_2)\). Both components use the same \(r\). The lift \(v\) remains internal. Its real-key law is not asserted to be uniform. Removing \(\rho\) would change the scheme and invalidate the uniform-lifting simulation in Section 6.

### 3.3 Decryption

Check the prescribed lengths and coefficient ranges. For a valid ciphertext compute
\[
z=[2c_2-(2c_1)^Ts]_{4096}.
\]
Decode coefficient \(i\) as one exactly when \(1024<z_i<3072\); otherwise decode it as zero. Both distance ties are assigned zero. Invalid encodings return failure at the base-PKE interface and use the specified fallback branch at the KEM interface.

## 4. Exact error geometry and the correctness definition

Define the actual remainders
\[
r_0=[As+e_0]_4,\qquad
r_1=[A^Tr+e_1]_2,\qquad
r_2=[v^Tr+e_2]_2.
\]
Divisibility of \(P\) by 4 and 2 permits these expressions before or after reduction modulo \(P\). Since \(2\mid2048\), \(r_2\) is unchanged by the message at fixed coins. This is not independence of \(r_2\) and \(e_2\).

**Lemma 1 — Exact error identity.** For every honest execution,
\[
z\equiv2048\mu+\varepsilon\pmod{4096},\qquad
\varepsilon=(e_0+\rho-r_0)^Tr+(r_1-e_1)^Ts+e_2-r_2.
\]
Integer negacyclic products give a valid integer lift of each \(\varepsilon_i\).

*Proof.* Euclidean division gives \(4b\equiv As+e_0-r_0\), \(2c_1\equiv A^Tr+e_1-r_1\), and \(2c_2\equiv v^Tr+e_2+2048\mu-r_2\). Substitute these identities into decryption and cancel the two main bilinear products using Section 2. No distributional independence is used. \(\square\)

If \(|\varepsilon_i|<1024\), circular distance modulo 4096 gives the correct decoded bit. Consequently every failure is contained in
\[
\mathcal F=\{\max_i|\varepsilon_i|\ge1024\}.
\]
The containment need not be equality, since a large integer lift can wrap to a correctly decoded residue.

For an honestly generated key define
\[
d(pk,sk)=\max_\mu\Pr_\omega[
\operatorname{Dec}(sk,\operatorname{Enc}(pk,\mu;\omega))\ne\mu\mid pk,sk],
\qquad
\delta_{wc}=\mathbb E_{pk,sk}d(pk,sk).
\]
This averages the maximum over messages; it does not take a maximum over all keys. The event \(\mathcal F\) contains all message failures under a same-coins coupling. Its message independence therefore allows the maximum to precede the final key average. If multiple key-generation draws yield the same key pair, average those latent draws conditionally on that pair.

## 5. Exact correctness certificate

### 5.1 Exponential moments with dependent remainders

For \(u>1\), let \(E,R,S\) be independent with law \(\chi\), and let \(T\) be independent uniform on \(\{0,1,2,3\}\). Define
\[
a(u)=\mathbb E_{E,R,T}u^{(|E|+T)R},\qquad
b(u)=\mathbb E_S\max_{c\in\{0,1\}}\mathbb E_Eu^{S([c+E]_2-E)},\qquad
c(u)=\mathbb E_Eu^E.
\]
The maximum in \(b(u)\) is inside the expectation over \(S\).

**Lemma 2 — Hidden-remainder domination.** For an integer \(e\), \(t\in\{0,1,2,3\}\), and symmetric \(R\) independent of uniform \(T\),
\[
\mathbb E_{R,T}u^{(e+T-t)R}\le
\mathbb E_{R,T}u^{(|e|+T)R}.
\]
The same bound holds with the exponent negated.

*Proof.* The function \(f(y)=\mathbb E_Ru^{yR}\) is even and convex. Writing \(g(x)=\tfrac14\sum_{h=0}^3 f(x+h-3/2)\), we obtain an even convex function. The left-hand side is \(g(e+3/2-t)\). Over the possible shifts \(t\), its maximum is at the endpoint farthest from that midpoint. This is \(t=0\) when \(e\ge0\) and \(t=3\) when \(e\le0\). Reflect \(T\) in the second case. Symmetry of \(R\) absorbs the sign. The inequality is pointwise in the hidden remainder \(t\). \(\square\)

**Theorem 3 — Joint-error moment bound.** For any fixed matrix, or any random matrix independent of the joint draw of \(s,e_0,r,e_1,e_2,\rho\), under the sampling laws of Sections 2–3,
\[
\mathbb E u^{\varepsilon_i}\le(a(u)b(u))^{1024}c(u),\qquad
\mathbb E u^{-\varepsilon_i}\le u(a(u)b(u))^{1024}c(u).
\]
It follows that
\[
\delta_{wc}\le256(1+u)(a(u)b(u))^{1024}c(u)u^{-1024}.
\]

*Proof.* Fix one output coefficient. In a negacyclic inner product each input coordinate occurs exactly once, paired with one coordinate of the other factor and with a fixed sign. There are 1024 such pairs in each vector inner product.

First remove \(-r_{2,i}\) pointwise. Its positive exponential contributes at most 1, and its negative exponential at most \(u\). Average the remaining \(e_{2,i}\) to obtain \(c(u)\). This step never conditions \(r_2\) to be independent of \(e_2\).

Next fix \(A,s,e_0,r,\rho\). Each relevant coefficient of \(e_1\) appears once and its remainder is \([c_j+e_{1,j}]_2\) for a fixed offset \(c_j\). Average the independent \(e_{1,j}\)'s, then bound each factor by the maximum over its offset. These factors depend on the paired signed secret coefficients but no longer depend on \(r\).

With \(A,s,e_0\) fixed, \(r_0\) is fixed. Average the independent actual pairs of ephemeral coefficients and lift coefficients in the first inner product. Lemma 2 bounds every factor without assigning an independent law to \(r_0\). The resulting factors depend only on the associated \(|e_{0,j}|\).

Only now average the mutually independent coefficients of \(s,e_0\). The first factors yield \(a(u)^{1024}\); the second yield \(b(u)^{1024}\), since the secret law absorbs every negacyclic sign. Reversing all signs gives the inequality for the negative exponential. Markov's inequality for the two tails, followed by a union bound over 256 outputs, gives the claim. Output coefficients need not be independent. Section 4 justifies the worst-message maximum. \(\square\)

### 5.2 Finite rational evaluation

**Corollary 4 — Rank-four upper bound.** At \(u=4/3\), finite sums give
\[
a=\frac{744601}{663552},\qquad b=\frac{589}{576},\qquad c=\frac{49}{48}.
\]
Therefore
\[
B=256\,\frac73\left(\frac{744601}{663552}\cdot\frac{589}{576}\right)^{1024}
\frac{49}{48}\left(\frac34\right)^{1024},\qquad
\delta_{wc}\le B<2^{-212}.
\]
For the reduced rational \(B=N/D\), the certificate is the integer inequality \(2^{212}N<D\). Its approximate display value is
\[
\log_2 B=-212.5264474805\ldots.
\]
The integer comparison, rather than a rounded logarithm or a Monte Carlo estimate, establishes the upper bound.

### 5.3 Positive-probability failure

**Proposition 5 — Correctness is not perfect.** For the same profile,
\[
\delta_{wc}\ge2^{-8192}.
\]

*Proof.* Use the all-zero message. Set every coefficient of \(s,e_1,e_2,\rho\) to zero and every coefficient of \(e_0\) to \(-1\). Set all coefficients of the first three polynomials of \(r\) to 1 and those of its fourth polynomial to 0. For every \(A\), this gives \(b=1023\), \(v=4092\), \(r_0=3\), and \(r_2=0\). At coefficient zero,
\[
\varepsilon_0=3(-4)(1-255)=3048,
\]
which is decoded as one. The independent event probabilities have exponents
\[
1024+2048+2048+1024+256+1536+256=8192
\]
for \(s,e_0,\rho,e_1,e_2\), the first three components of \(r\), and its fourth component, respectively. This event implies a zero-message failure without restricting \(A\). Averaging over keys and then using the worst-message maximum proves the claim. \(\square\)

Combining the bounds gives
\[
\boxed{2^{-8192}\le\delta_{wc}<2^{-212}.}
\]
Markov's inequality also gives \(\Pr[d(pk,sk)>\theta]\le\min(1,2^{-212}/\theta)\). For at most \(h\) fresh honest encryptions under one generated key, even with adaptively chosen messages, a conditional union bound gives a total failure probability at most \(\min(1,h2^{-212})\). Neither statement gives uniform correctness for maliciously selected keys.

## 6. Exact source assumptions and conditional IND-CPA security

### 6.1 The source distributions

For \(j\in\{4,5\}\), define \(\mathrm{DMLWE}_{4,j}^{\chi}\) by distinguishing
\[
(B,Bt+e)
\quad\text{from}\quad(B,y),
\]
where \(B\leftarrow U(R^{j\times4})\), \(t\leftarrow\chi^{1024}\), \(e\leftarrow\chi^{256j}\), and \(y\leftarrow U(R^j)\) are initially independent. The same secret \(t\) is used in every row. Let \(\epsilon_{4,j}(\mathcal B)\) denote the absolute acceptance-probability gap of a distinguisher.

These are exact small-secret, small-error, power-of-two-modulus assumptions. The coefficient secret dimension is 1024. The four-row source supplies 1024 coefficient observations and the five-row source 1280. A generic unstructured-LWE theorem is not imported merely by counting those coefficients.

### 6.2 Uniform lifting with the public view

If \(V\) is uniform in \(R^4\) and independent of \(A\), then
\[
(A,F_4(V),V)\overset d=
(A,b,4b+\rho),
\]
where \(b\) and \(\rho\) are independent uniform quotient and remainder vectors. This follows coordinatewise from the bijection \(v\leftrightarrow(\lfloor v/4\rfloor,[v]_4)\). The conditional remainder remains uniform after an adversary selects messages from \((A,b)\) and independent coins. Independence from the whole public view, not merely from \(b\), is required.

### 6.3 Two exact simulations

**Theorem 6 — Abstract full-matrix CPA reduction.** With
\(\operatorname{Adv}_{CPA}(\mathcal A)=|\Pr[\widehat\beta=\beta]-1/2|\), there are source distinguishers at the actual augmented simulation budget such that
\[
\operatorname{Adv}_{CPA}(\mathcal A)\le
\epsilon_{4,4}(\mathcal B_1)+\epsilon_{4,5}(\mathcal B_2).
\]

*Proof.* In the first transition replace the real public quotient \(b\) by an independent uniform quotient vector. Given a four-row source sample \((A,y)\), publish \((A,F_4(y))\), run the adversary, choose its challenge bit, and encrypt publicly. The real and uniform source cases give precisely the two games. Return the adversary's success indicator. The simulator does not need a decryption secret.

For the second transition, write a five-row source matrix as
\[
B=\begin{pmatrix}A^T\\V^T\end{pmatrix},\qquad y=(y_{0:4},y_4).
\]
Publish \((A,F_4(V))\), retaining \(V\). After message selection, return
\[
c_1=F_2(y_{0:4}),\qquad c_2=F_2(y_4+2048\mu_\beta).
\]
In the real source, the common secret is the required ephemeral \(r\), and the five independent errors supply \(e_1,e_2\). Uniform lifting gives the exact conditional law of the retained \(V=4b+\rho\) after message selection. Any other local encryptions use their own fresh lifts.

In the uniform source, all five responses are jointly uniform independently of the full matrix. Message translation preserves uniformity, and equal-sized floor cells preserve uniform quotient outputs. The challenge is consequently independent of \(\beta\) given the public view. The two source gaps bound the two changes in success probability; the last game's success probability is \(1/2\). \(\square\)

This theorem transfers source hardness conditionally. It does not prove the source problems hard. Its two reductions use their respective single source samples; no unlimited-sample oracle is assumed.

## 7. Seeded matrices, encodings, and exact byte counts

### 7.1 Matrix-expander model

The compact key uses a uniform 256-bit seed and an independent ideal matrix oracle \(H_A\). Injective, domain-separated inputs identify the seed and matrix coordinates. Each response has 512 bytes, interpreted as 256 little-endian 16-bit words, reduced modulo 4096. Distinct entry inputs receive independent responses.

Each 12-bit coefficient has 16 preimages among 16-bit words. To program a supplied uniform matrix, independently sample the discarded four high bits of every word. This reproduces the entire raw-response distribution, including bits an adversary can query but the matrix parser ignores.

With at most \(q_A^{pre}\) prior queries, a freshly drawn seed conflicts with a previously exposed seed with probability at most
\[
\epsilon_A=\min(1,q_A^{pre}2^{-256}).
\]
Abort on that bad event instead of resampling until success. Outside it the joint simulation is exact. Applying the allowance to both simulations gives
\[
\operatorname{Adv}_{CPA}^{seed}\le\epsilon_{4,4}+\epsilon_{4,5}+2\epsilon_A.
\]
For correctness with a seeded key, fix the public matrix-oracle table when conditioning on a key, or equivalently retain its expanded matrix in the public state. The fixed-matrix proof in Section 5 then applies, followed by averaging over generated keys and the public oracle. On the bad event the simulator gives the same fixed output in both source branches. The lost difference of acceptance probabilities on that common event is at most its probability. A fixed concrete expander such as SHAKE256 requires a separate modeling assumption. A short seed is not a lossless representation of every independent full matrix.

### 7.2 Payload sizes

Store a one-byte profile identifier, the 32-byte seed, and the 1024 ten-bit public quotients. Store 1280 eleven-bit ciphertext quotients. Thus
\[
|pk|=1+32+1024\cdot10/8=1313\text{ bytes},\qquad
|c|=1280\cdot11/8=1760\text{ bytes}.
\]
The unseeded matrix alone occupies \(16\cdot256\cdot12/8=6144\) bytes; with the public quotient vector it occupies 7424 bytes before metadata.

The public payload orders the profile byte, seed and four quotient polynomials. The ciphertext orders the four polynomials of its first component, then the single polynomial of its second component. Coefficients are in increasing degree order and each fixed-width value contributes its least significant bit first to the byte stream, starting at bit zero. These payloads are byte-aligned. Canonical parsing admits no alternative accepted representation of the same ciphertext. A full KEM secret-key format must also retain the public data needed for re-encryption and a fallback secret; its length is not determined by the two public payload counts.

### 7.3 Uniform encryption tape

The archival allocation uses 2816 bytes. The first 2304 bytes supply the \(4n+4n+n\) CBD1 coefficients: subtract bit 1 from bit 0 of each byte. The remaining 512 bytes supply 1024 nibbles; reduce each nibble modulo four to obtain \(\rho\). Independent uniform bytes give exactly the required laws, since the two nibbles of a byte are independent uniform values. Unused bits are ignored; no injectivity of this tape-to-coins map is claimed.

## 8. Classical random-oracle KEM route and its limits

Take independent random oracles \(G,H\), also separate from \(H_A\), with injectively encoded tuple domains. Let \(G(pk,\mu)\) produce a 2816-byte tape and \(H(pk,\mu,c)\) a 256-bit key. Write
\[
\operatorname{Enc}_1(pk,\mu)=\operatorname{Enc}(pk,\mu;G(pk,\mu)).
\]
Generate one independent uniform 256-bit fallback secret \(z_0\) per key. Encapsulation selects uniform \(\mu\), returns \(c=\operatorname{Enc}_1(pk,\mu)\) and \(H(pk,\mu,c)\). Decapsulation obtains \(\mu'\), re-encrypts it, and returns
\[
\begin{cases}
H(pk,\mu',c),&\text{if parsing succeeds and }\operatorname{Enc}_1(pk,\mu')=c,\\
H(pk,z_0,c),&\text{otherwise}.
\end{cases}
\]
The comparison is byte-exact. Invalid lengths take the fallback branch; repeated rejected ciphertexts receive consistent fallback keys. Good and fallback branches use the same specified \(H\) family with their respective middle argument. The algorithms are an abstract oracle specification, not a claim that the supplementary arithmetic scripts implement a production KEM, a constant-time decapsulator, or the byte-level oracle domains. An interoperable implementation also needs assigned profile and domain tags, exact oracle input encodings, a secret-key format and a side-channel assessment.

The challenge ciphertext \(c^*\) is excluded from all decapsulation queries. A plaintext check takes \((m,c)\) with \(m\in\{0,1\}^{256}\) and returns the bit \([\operatorname{Dec}_1(sk,c)=m]\), where \(\operatorname{Dec}_1\) checks parsing and re-encryption. There is no separate ciphertext-validity oracle. At a fixed challenge key, each injectively prefixed oracle restricts to a single-key random function. Other key prefixes can be simulated by independent tables. Here IND-CCA uses a fair bit to choose between the real encapsulated key and an independent uniform 256-bit key accompanying the same honest challenge ciphertext. The classical route is based on the corrected HHK games [1], transformation \(T\) followed by implicit-rejection \(U^{\not\perp}\), in the single-user challenge-first experiment. Both CPA and CCA advantages here are success biases \(|\Pr[\widehat\beta=\beta]-1/2|\); source advantages remain acceptance gaps. We use the conservative derivation below rather than silently identifying these normalizations or treating faulty decryption as perfect.

Let \(q_G,q_H,q_D\) count explicit adversarial queries, and take
\[
Q_G=q_G+q_D+1,\qquad Q_H=q_H+2q_D+1,\qquad
Q_{RO}=Q_G+Q_H=q_G+q_H+3q_D+2.
\]
These allowances include the honest challenge, re-encryption, and evaluation of both candidate hashes per decapsulation. In the classical HHK simulation the plaintext-checking budget is at most \(Q_H\). Let \(\mathcal L\) contain all distinct messages for the target key at which a simulated explicit query or plaintext check consults \(G\). A private tape draw used to create the challenge does not insert the hidden message into this publicly testable list. Its budget slot is already included in \(Q_G\). Thus \(|\mathcal L|\le L\le Q_{RO}\) for a deterministic budget \(L\). Repeated inputs use the same table entry. In HHK’s implicit-rejection simulation the check for a challenge-ciphertext hash query is cached when that query is answered, so extracting a previously checked message adds no plaintext check. This retains the corrected \((q_G+q_P)\delta\) term from [1] by budgeting plaintext-checking inputs explicitly.

For clarity, the conversions can be bounded without a tight-constant claim. Let \(M=2^{256}\), \(\delta=\delta_{wc}\), and let \(\beta_D,\beta_E\) be the success biases of the two CPA adversaries arising in the \(T\) games. Use a base-CPA challenge encryption of a uniform message with fresh private coins. In the simulated transcript, the public \(G\) table is independent of those coins, and a plaintext check on \((m,c)\) compares \(c\) with the public re-encryption under its table entry. Malformed ciphertexts fail this check. Couple this simulation to the real transformed challenge by reserving its private coins for the real entry at the hidden message. Up to the first consultation of that entry, the two transcripts agree unless a publicly tested encryption decrypts incorrectly. Conditional on the key pair, each newly exposed entry at a different message uses a fresh uniform tape, even after adaptive message selection. A union bound costs \(L\,d(pk,sk)\); averaging gives \(L\delta\).

For the list test, draw two independent uniform candidate messages. Guess the index of the unique list member if there is one and use a fair coin otherwise. Its signed success bias is half the difference between the inclusion probabilities for the encrypted and the unused candidate. The unused candidate is independent of the simulated transcript, so its inclusion probability is at most \(L/M\). Hence the challenge-message hit probability is at most \(2\beta_D+L/M\); we use the looser \(2\beta_D+2L/M\) below. For recovery, guess the first candidate’s index when the output equals that candidate, and the second index otherwise. A match with an unused candidate has probability at most \(1/M\). Original-message recovery therefore costs at most \(2\beta_E+1/M\). In HHK’s plaintext-checking experiment, success targets the decrypted challenge value. This changes the probability by at most \(\delta\), the honest challenge error under a uniform message and tape.

After the \(L\delta\) allowance for incorrect simulated re-encryptions, these bounds give
\[
p_{OW\text{-}PCA}\le
2\beta_D+2\beta_E+(L+1)\delta+(2L+1)/M.
\]
For the implicit-rejection hop, couple the fallback outputs to an independent random value per rejected ciphertext until an explicit hash query guesses \(z_0\). In this latter game the adversary’s candidate guesses are independent of \(z_0\) before a hit. Hash and decapsulation tables agree on an accepted ciphertext through its unique decrypted message. A hash query targeting the honest message and challenge ciphertext gives a plaintext-checking recovery whenever that challenge decrypts correctly. The fallback-guess contribution is at most \(Q_H/M\) for guessing the fallback message. Retaining a further \(\delta\) for an incorrectly decrypting honest challenge yields
\[
\operatorname{Adv}_{CCA}\le
2\beta_D+2\beta_E+(L+2)\delta+(2L+Q_H+1)/M.
\]
This is the proposed conservative envelope for the described classical game hops, not a verbatim tight bound attributed to [1] and not a statement about arbitrary oracle interfaces. Since \(Q_{RO}\ge2\), its last two terms are at most \(2Q_{RO}\delta\) and \((3Q_{RO}+1)/M\). Let \(\epsilon_{4,j}\) bound the corresponding source gaps at the largest augmented resource budget of these CPA reductions. Applying Sections 6–7 to each gives
\[
\boxed{
\operatorname{Adv}_{CCA}\le
4\epsilon_{4,4}+4\epsilon_{4,5}+8\epsilon_A
+2Q_{RO}2^{-212}+(3Q_{RO}+1)2^{-256}.
}
\]
Source budgets include oracle simulation and arithmetic work. This is a conditional inequality for the construction and parameters in the precisely scoped challenge-first classical experiment above. For a phase of \(d_{pre}\) decapsulations before a fresh uniform challenge message, retain the separate collision allowance \(\min(1,d_{pre}2^{-256}+\delta_{wc})\): a ciphertext can decrypt correctly to at most one message, while the expected fraction of failing messages is at most \(\delta_{wc}\). The pre-challenge allowance is separate and is not part of the boxed challenge-first inequality.

The displayed KEM bound is conditional on the precise classical experiment, oracle interfaces and augmented simulation budgets used in the game argument; its use with a concrete implementation would require a fresh interface check. In particular, this calculation is not an end-to-end certification of an implemented KEM. Quantum-accessible oracles and sharper failure-finding bounds are treated separately in [5] and [6]. Recent FFP-NG literature covers Ring/Module-LWE and noise families that include centered binomials [8], but matching that framework to this profile, its quotient compression and its randomized lifting remains an open obligation. No multi-user theorem, concrete-expander theorem, side-channel guarantee, or fixed security level is established here.

## 9. Reproducible known-attack estimates

The following numbers evaluate specified cost models. They do not describe completed attacks on a 1024-dimensional instance, prove source hardness, or exhaust cryptanalysis. The distinction between coefficient-linear algebra and ring structure matters for estimating attacks [3].

| Attack model | Reported \(\log_2\) work | Status |
|---|---:|---|
| Classical primal uSVP baseline | 227.176 | Estimated |
| Classical plain-dual baseline | 235.060 | Estimated |
| MATZOV-style expanded search | 213.656698256 | Lowest point found in the documented domain |
| Rotation-aware coded dual, both adaptations applied | 206.210581344 | Archived Sage evaluation at the stated point |

### 9.1 MATZOV-style search

The port uses the MATZOV-style dual-hybrid cost model [2], with coefficient dimension 1024, modulus 4096, secret/error standard deviation \(\sqrt{1/2}\), and classical reduction exponent 0.292. The standalone search tests sample counts 1024 and 1280 separately. For each, it scans integer \(\beta=700,\ldots,900\) at \((p,k_{enum},k_{fft})=(3,11,113)\), then performs four ordered coordinate rounds.

At the start of each round, the current best point determines the coordinate windows: \(\beta\) within 40, clipped to \([40,m]\); \(k_{enum}\) within 20, clipped below at zero; \(k_{fft}\) within 25, clipped below at zero; and integer \(p\in[2,10]\). Each coordinate is varied while the other coordinates retain their current best values. This is a deterministic local search, not enumeration of the full Cartesian domain or a global optimization proof.

The reported \(m=1024\) point is
\[
(\beta,p,k_{enum},k_{fft},k_{lat},d)=(731,3,6,114,904,1928),
\]
with \(\log_2 T=213.656698255976\ldots\). The narrower historical baseline \(220.538\ldots\) is retained only in archived outputs and is not the preferred MATZOV estimate.

### 9.2 Coded dual: two necessary adaptations

The coded-dual model originates in [9]. Its rotation-aware variant is described in [3], full version (a). The adapter targets the public CodedDualAttack fork [4] at commit
`7cce6a557242`.
This fork already uses \(\beta_1\) in the threshold integral identified in its README. Relative to that snapshot, the adapter applies two further changes in memory:

1. The CBD1 law requires \(p_0=\Pr[X=0]=1/2\).
2. The lower search bracket for \(R\) is reset to 1. At the reported point, the upstream lower bracket 262144 already has success probability about 0.795804, above the target 0.6; it cannot serve as a below-target endpoint.

For
\[
(m,n_{enu},n_{fft},k_{fft},n_{lat},\beta_0,\beta_1)
=(770,17,135,14,872,701,701),
\]
the checked adjacent values are
\[
\eta(137828)=0.599998202351\ldots<0.6
\le0.600000555101\ldots=\eta(137829).
\]
The corrected point uses \(R=137829\) and gives
\[
\log_2T=206.21058134400172.
\]
Both changes are needed to reproduce this figure; changing the CBD1 probability alone is insufficient. The accompanying Sage adapter uses an absolute output path resolved before changing directories. The corrected Sage record is the file `coded_dual_bracket_check.json` under `audit_imports`. The separate `coded_dual_sage_absolute.log` records the earlier bracket and a different cost. A fresh evaluation of the binomial success probabilities at the three stated inputs confirms their ordering. The full Sage cost was not recomputed for this revision.

### 9.3 What the fifth row does and does not imply

The coded-dual cost function at the reported point takes \(m=770\) scalar observations as an input. This count fits the 1024 coefficient observations supplied by four module rows. The scalar count alone does not establish an implementation for every selection of partial ring rows. This fact does not establish that extra observations never improve an attack. No exhaustive coded-dual search over all 1280 observations is claimed. The separate local MATZOV probes are likewise not a proof of completeness for either source interface.

## 10. Reduction budgets are not security lower bounds

The supplied arithmetic budget counts 63,200 word-model operations for one re-encryption under its stated schedule, bounded conservatively by \(2^{16}\), with a fixed precomputation allowance \(2^{17}\). It uses an auxiliary NTT prime for exact convolution; that prime does not replace the scheme modulus 4096. The associated bookkeeping gives
\[
\kappa\le1+4\cdot2^{16}+2^{17}=393217,\qquad
\log_2(12\kappa)=22.1699286704\ldots.
\]
These are modeled arithmetic counts, not measured processor cycles, bit-operation bounds for every implementation, or quantum-oracle simulation costs. The count assumes precomputed transform constants; it is not the instruction count of the Python self-test, which computes powers and updates twiddles dynamically. The displayed \(\log_2(12\kappa)\) is retained as an archived bookkeeping value, not as a certified loss for the revised bound in Section 8. The attack-model work unit and this simulator's word-model unit are different. Subtracting their logarithms has a security interpretation only after explicitly assuming a common resource model and a source work/advantage bound at the reductions' full budgets.

In particular, a modeled attack at \(2^{206.211}\) is not evidence that every attack requires that work. It cannot by subtraction establish an effective security lower bound near 184 bits. This article supplies no such lower bound and no 128-bit certification.

## 11. Reproducibility and verification boundary

The companion archive contains the pure-Python checks, the standalone expanded MATZOV search, the Sage adapter with its public-code lock, expected values, and preserved execution records. The source snapshot is the supplementary-data V4 archive; the current preprint does not retroactively edit or rerun the original Sage work. From a clean copy of that archive, run:

```bash
python -m pip install -r requirements.txt
python scripts/run_pure_python_checks.py
python scripts/k4_matzov_expanded_search.py
```

The first command installs dependencies. The main pipeline regenerates its result JSON files before expected-value validation. The expanded search is separate from the legacy baseline scan. The Sage workflow is specified in `REPRODUCIBILITY.md`; it must not be reported as freshly executed merely because its earlier results are present.

On 18 November 2026 the finite closure checks and the pure-Python reproducibility pipeline were executed anew on a clean extracted copy and passed; the deterministic expanded MATZOV search was rerun and found the documented 213.656698255976 point for 1024 observations. For v1.1 the Python outputs were also regenerated with all six target JSON files initially absent. The Sage coded-dual cost is retained as a historically logged result. A separate dated verification report accompanies the release candidate.

The correctness proof is Sections 4–5, supported by its finite rational certificate. A passing script alone does not prove the distributional arguments in Section 6, validate attack heuristics, or establish security in a different oracle model. The precise public-key/ciphertext interface and canonical encoding must remain fixed when instantiating an external transformation theorem.

## 12. Conclusion

C1-K4-P12-CBD1 has a self-contained exact error analysis, a proved nonzero correctness interval, two explicit conditional source reductions, and reproducible attack estimates. The compact public-key size depends on its stated seeded-matrix model. The corrected estimates retain both the expanded MATZOV search and the two coded-dual adaptations. The construction remains a research candidate. Source hardness and concrete instantiation require separate justification; the conditional classical-ROM KEM argument in Section 8 must be interpreted only within its stated experiment and oracle interface. A secure deployment or PQ-128 certification does not follow.

## References

1. Dennis Hofheinz, Kathrin Hövelmanns, and Eike Kiltz. *A Modular Analysis of the Fujisaki–Okamoto Transformation*. TCC 2017; corrected full version dated 2 November 2021, Cryptology ePrint Archive 2017/604. Classical games: Sections 3.1–3.2; transformation interfaces: Theorems 3.2 and 3.4; query-count correction: the errata. Section 8 above gives its own conservative normalization and correctness allowances. <https://eprint.iacr.org/2017/604>.
2. MATZOV. *Report on the Security of LWE: Improved Dual Lattice Attack*. 2022, version 2. <https://doi.org/10.5281/zenodo.6493704>.
3. Jianhua Hou, Haodong Jiang, and Tabitha Ogilvie. *Careful with the Ring! Concrete Hardness Gaps Between LWE and MLWE*. CRYPTO 2026, LNCS 16802, pp. 462–497. <https://doi.org/10.1007/978-3-032-35377-1_15>. Full version (a), Tabitha Ogilvie, *On the Concrete Hardness Gap Between MLWE and LWE*, ePrint 2026/279. <https://eprint.iacr.org/2026/279>.
4. *CodedDualAttack*, public implementation, branch `eprint-2026-279`, commit `7cce6a557242` (full identifier in the supplied supplementary lock file). [pinned repository](https://github.com/identitymapping/CodedDualAttack/tree/7cce6a5572420004da31be58f26954cedfc885d0). The pinned code snapshot is distinct from the publication and its later revisions.

5. Samir Bouftass. *C1-K4 in the QROM: Structural Bounds and a Conservative Conditional Reduction*. Companion research manuscript, revision V4.2, 13 November 2026. Publication identifier not yet assigned.
6. Samir Bouftass. *Find-Failing-Plaintext Analysis of C1-K4-P12-CBD1*. Companion research manuscript, revision V4.4, 13 November 2026. Publication identifier not yet assigned.
7. Samir Bouftass. *A Quotient–Remainder Calculus for Modular Rounding: Exact Carries, Distributional Laws, Nested Truncation, and the Boundary with Learning With Rounding*. Companion research manuscript, revision V4.3, 13 November 2026. Publication identifier not yet assigned.

8. Christian Majenz and Fabrizio Sisinni. *Provable decryption failure security for practical lattice-based PKE*. Cryptology ePrint Archive, Paper 2025/1997, 2025. <https://eprint.iacr.org/2025/1997>. Its theorem hypotheses require a separate verification for the C1-K4 profile.

9. Kévin Carrier, Charles Meyer-Hilfiger, Yixin Shen, and Jean-Pierre Tillich. *Assessing the Impact of a Variant of MATZOV’s Dual Attack on Kyber*. CRYPTO 2025, LNCS 16000, pp. 444–476. <https://doi.org/10.1007/978-3-032-01855-7_15>.

## Contribution and responsibility statement

Samir Bouftass is listed as the author and is responsible for the scientific claims, attribution, and publication decisions. ChatGPT (OpenAI) provided computational and drafting assistance, including mathematical exposition, code development, consistency checks, and manuscript consolidation; it is acknowledged as a tool, not listed as an academic author. This candidate revises the scientific manuscript V4.1 of 13 November 2026 and the preprint candidate v1.0 of 18 November 2026. The accompanying audit identifies the source files, fresh checks and editorial changes. External peer review and deployment assessment are separate processes.
