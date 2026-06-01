---
publish: true
---

|     |     |        |         |     |     | PAC-Private |               | Algorithms∗ |     |     |          |           |     |     |     |
| --- | --- | ------ | ------- | --- | --- | ----------- | ------------- | ----------- | --- | --- | -------- | --------- | --- | --- | --- |
|     |     | Mayuri | Sridhar |     |     |             | Hanshen       | Xiao        |     |     | Srinivas | Devadas   |     |     |     |
|     |     | MIT    | CSAIL   |     |     |             | NVIDIA/Purdue | University  |     |     |          | MIT CSAIL |     |     |     |
Cambridge, MA, 02139 West Lafayette, Indiana, 47907 Cambridge, MA, 02139
Email: mayuri@mit.edu Email: hsxiao@purdue.edu Email: devadas@mit.edu
Abstract—Provable privacy typically requires involved analysis be tightly computed in a few simple applications such as
and is often associated with unacceptable accuracy loss. While aggregation or linear queries. Further, Maximal Leakage
many empirical verification or approximation methods, such (MaxL) [3] requires a white-box analysis of the likelihood
functions,whichisoftencomplex.Moreover,toensurethese
| as Membership |     | Inference |     | Attacks | (MIA) | and | Differential |     |     |     |     |     |     |     |     |
| ------------- | --- | --------- | --- | ------- | ----- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
PrivacyAuditing(DPA),havebeenproposed,thesedonotoffer input-independent indistinguishability guarantees, artificial
rigorous privacy guarantees. In this paper, we apply recently- modifications are typically required to decompose most
|          |          |               |     |     |         |       |            | algorithms | into multiple |     | simpler | and analyzable |     | components, |     |
| -------- | -------- | ------------- | --- | --- | ------- | ----- | ---------- | ---------- | ------------- | --- | ------- | -------------- | --- | ----------- | --- |
| proposed | Probably | Approximately |     |     | Correct | (PAC) | Privacy to |            |               |     |         |                |     |             |     |
suchasmeanestimationormajorityvotingtoenabletractable
| give formal, | mechanized, |     | simulation-based |     |     | proofs | for a range |     |     |     |     |     |     |     |     |
| ------------ | ----------- | --- | ---------------- | --- | --- | ------ | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
of practical, black-box algorithms: K-Means, Support Vector analysis; Differentially-Private Stochastic Gradient Descent
Machines (SVM), Principal Component Analysis (PCA) and (DP-SGD) [4] and PATE [5] are representative examples.
|        |          |     |         |       |         |            |       | Unfortunately, | artificial |     | modifications |             | usually | come  | with   |
| ------ | -------- | --- | ------- | ----- | ------- | ---------- | ----- | -------------- | ---------- | --- | ------------- | ----------- | ------- | ----- | ------ |
| Random | Forests. | To  | provide | these | proofs, | we present | a new |                |            |     |               |             |         |       |        |
|        |          |     |         |       |         |            |       | limits on      | algorithms | and | data          | structures, | and     | often | with a |
simulationalgorithmthatefficientlydeterminesanisotropicnoise
|     |     |     |     |     |     |     |     | significant | compromise |     | on utility. |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ---------- | --- | ----------- | --- | --- | --- | --- |
perturbationrequiredforanygivenlevelofprivacy.Weprovide
Asaconsequence,thelackofpowerfulriskquantification
| a proof        | of correctness |               | for this    | algorithm | and        | demonstrate    | that      |                 |             |          |                  |             |        |              |           |
| -------------- | -------------- | ------------- | ----------- | --------- | ---------- | -------------- | --------- | --------------- | ----------- | -------- | ---------------- | ----------- | ------ | ------------ | --------- |
|                |                |               |             |           |            |                |           | tools heavily   | restricts   |          | the study        | and         | design | of defensive |           |
| anisotropic    | noise          | has           | substantive | benefits  |            | over isotropic | noise.    |                 |             |          |                  |             |        |              |           |
|                |                |               |             |           |            |                |           | methods         | for leakage | control, | as               | the privacy |        | implications | of        |
| Stable         | algorithms     |               | are easier  | to        | privatize, | and            | we demon- |                 |             |          |                  |             |        |              |           |
|                |                |               |             |           |            |                |           | many operations |             | are not  | well-understood. |             | Even   | for          | perturba- |
| strate privacy |                | amplification |             | resulting | from       | introducing    | regular-  |                 |             |          |                  |             |        |              |           |
tion,themostpopularandstraightforwardprivacy-preserving
| ization in | these | algorithms; |     | meaningful | privacy | guarantees | are |     |     |     |     |     |     |     |     |
| ---------- | ----- | ----------- | --- | ---------- | ------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
technique,theminimalnoisetoproducerequiredsecuritypa-
| obtained | with | small losses | in  | accuracy. | We  | propose | new tech- |           |         |         |      |          |           |             |     |
| -------- | ---- | ------------ | --- | --------- | --- | ------- | --------- | --------- | ------- | ------- | ---- | -------- | --------- | ----------- | --- |
|          |      |              |     |           |     |         |           | rameters, | largely | remains | open | for most | practical | algorithms. |     |
niques in order to reduce instability in algorithmic output and In addition, the definition of sensitive information varies
| convert       | intractable | geometric   |               | stability | verification |             | into efficient |                  |              |            |           |             |           |            |       |
| ------------- | ----------- | ----------- | ------------- | --------- | ------------ | ----------- | -------------- | ---------------- | ------------ | ---------- | --------- | ----------- | --------- | ---------- | ----- |
|               |             |             |               |           |              |             |                | across different |              | processing | tasks     | and         | different | individual |       |
| deterministic | stability   |             | verification. |           | Thorough     | experiments | are            |                  |              |            |           |             |           |            |       |
|               |             |             |               |           |              |             |                | preferences.     | For example, |            | for image | data,       | people    | may        | worry |
| included,     | and         | we validate | our           | provable  | adversarial  |             | inference      |                  |              |            |           |             |           |            |       |
|               |             |             |               |           |              |             |                | about whether    | the          | adversary  | can       | reconstruct |           | sensitive  | face  |
hardness against state-of-the-art empirical attacks. features; for health data, the privacy objective can be the
|     |     |     |     |     |     |     |     | relationship | between | certain | associations |     | between |     | patients |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | ------- | ------- | ------------ | --- | ------- | --- | -------- |
1. Introduction and diseases; and in anonymous communication, identities
|               |     |         |            |     |                |     |            | are sensitive. | Universal  |         | risk | quantification |     | is thus   | highly |
| ------------- | --- | ------- | ---------- | --- | -------------- | --- | ---------- | -------------- | ---------- | ------- | ---- | -------------- | --- | --------- | ------ |
|               |     |         |            |     |                |     |            | desirable      | to capture | diverse | and  | customized     |     | concerns. |        |
| The expansion |     | of data | collection |     | and increasing |     | complexity |                |            |         |      |                |     |           |        |
of data processing are happening at unprecedented rates. Besides provable analyses, there is also a long line of
Concerns on information leakage are receiving increas- works focusing on empirical defenses against adversarial
ing attention, while privacy preservation is simultaneously inference. Privacy verification has been extensively studied,
|            |     |            |     |                   |     |               |     | in particular, | for | membership |     | inference | attacks | (MIA) | [6], |
| ---------- | --- | ---------- | --- | ----------------- | --- | ------------- | --- | -------------- | --- | ---------- | --- | --------- | ------- | ----- | ---- |
| challenged | by  | fast-paced |     | and sophisticated |     | advancements. |     |                |     |            |     |           |         |       |      |
Efficient and widely-applicable risk quantification has be- [7],[8].Forexample,manyoperationssuchasregularization
come a fundamental and urgent problem in privacy research. [6], [9], data augmentation [10], [11], and model compres-
Most existing provable privacy analyses of data processing sion [12] are empirically shown to resist certain kinds of
requirestrongassumptions.Forexample,DifferentialPrivacy attacks. However, qualitative analysis for those strategies is
1, challenging and largely remains open, especially in involved
| (DP) [1] | requires | bounded |     | sensitivity |     | which | can only |                 |             |     |        |     |                    |     |     |
| -------- | -------- | ------- | --- | ----------- | --- | ----- | -------- | --------------- | ----------- | --- | ------ | --- | ------------------ | --- | --- |
|          |          |         |     |             |     |       |          | data processing | algorithms. |     | Though |     | carefully-designed |     | em- |
∗An earlier version of this work was published at IEEE S&P 2025. We pirical simulations can provide meaningful approximations
havesincefixederrataincludinganimplementationbugandtheoptimal of privacy risks with respect to specific adversarial strategies,
choiceofrotationmatricesforPCA(seeClaim1).Experimentalresultsare a rigorous proof is desired to show worst-case guarantees
comparableorimproved;thealgorithmsandconclusionsremainunchanged.
1.InthecontextofDP,sensitivitycapturestheworst-caseinfluenceof against arbitrary adversaries. Closing this gap remains a key
anindividualontheoutput,whichisingeneralNP-hardtocompute[2]. and open problem in security and privacy research.

One recent effort to technically address the risk quantifi- terior guarantees and demonstrate that our privatized
cation for black-box data processing is PAC Privacy [13]. algorithms more than satisfy these guarantees against
From a statistical inference perspective, [13] develops a state-of-the-art attacks.
new framework to semantically interpret privacy risk as
concreteinferencehardnessforacomputationally-unbounded 2. Background
adversarytorecoversensitiveinformationsatisfyingacertain
criterion, which can be arbitrarily selected. A set of new We first introduce the PAC Privacy model to describe
tools are also established in [13] to provably convert the information leakage and privacy risk in general. Let X
objective inference hardness into simulatable quantities, denote the sensitive input, which is randomly generated
which enables high-confidence estimation from end-to-end from a (possibly black-box) distribution D, and M denote
black-box simulations to provide a privacy proof. However, a (possibly black-box) processing mechanism, where the
as a theoretical solution to conduct privacy analysis for a output, M(X), is released and observed by an adversary.
black-box processing, there are two important aspects of We challenge the adversary as to whether they can return
PAC Privacy which have not been systematically explored. some estimation X˜ satisfying a certain criterion, which can
First, how can we efficiently determine the (near-)optimal bedescribedbysomeindicatorfunctionρ.Suchaninference
anisotropicnoise2 toaddtoeachexposedoutput,andprovide challenge can be used to capture arbitrary privacy concerns
an associated privacy proof? Second, how can we stabilize and customized leakage control that a user is comfortable
a black-box data processing algorithm to provably produce with. For example, to capture a membership inference attack
a stronger privacy guarantee or a sharpened utility-privacy [6], ρ can be selected as ρ(X˜,X) = 1 if X˜ predicts the
tradeoff? membership of some particular datapoint u correctly in X;
0
In this paper, we contribute an initial comprehensive ρ may also capture data reconstruction [14], [15] where we
study to answer these questions, as summarized below. may set ρ(X˜,X)=1 iff ∥X−X˜∥ ≤1, i.e., the adversary
2
1) Novel algorithm for efficient simulation proofs: We can recover the input with error in l -norm smaller than 1.
2
present an algorithm in Section 4 that adds anisotropic For side-channel attacks on a cryptographic protocol [16],
noise and is more computationally efficient than the where X corresponds to the secret key, ρ can capture the
algorithm (Algorithm 1) in [13] which requires running colliding bits between X and X˜.
Singular Value Decomposition (SVD) on the entire Now, given the data entropy, determined by D, and the
output dimension which can be prohibitively expensive. objective inference task, we can define the optimal a priori
Efficiency is further enhanced through faster conver- success rate (1−δρ) that an adversary can return a satisfied
o
gence; our algorithm only needs to accurately estimate estimation before they observe the release M(X), i.e.,
variance of output along each direction, as opposed to
δρ =inf Pr (ρ(X˜ ,X)̸=1).
converging on a covariance matrix as in [13]. We prove o X˜
oX∼D
o
the correctness of our algorithm.
Similarly, we can define the posterior success rate (1−
2) Efficient privatization for black-box algorithms:
δ) to capture the probability for an adversary to return a
We implement PAC-private versions of several classic
satisfied estimation after observing the release. With the
algorithms.Weprovidenoiseestimatesandutilitytrade-
above preparation, we can now formally define PAC Privacy.
offs, demonstrating that these algorithms can generally
achievemeaningfulprivacywithasmalllossesinutility. Definition 1 ((δ,ρ,D) PAC Privacy [13]). For a processing
We show that adding anisotropic noise has significant function M : X∗ → O, some data distribution D, and
utilitybenefitsoveraddingisotropicnoiseusingl -norm an inference criterion function ρ(·,·), we say M satisfies
2
estimation. (δ,ρ,D)-PAC Privacy if the following experiment is impossi-
3) Sharpening privacy-utility tradeoffs: We first charac- ble:
terize the root of instability in these classic algorithms, A user generates data X from distribution D and sends
andseparatethemintotwolargecategories–superficial M(X) to an informed adversary. The adversary who knows
and intrinsic. Then, we provide novel canonicalization D and M is asked to return an estimation Xˆ on X such
techniquestoimproveuponsuperficialinstability,while that with probability at least (1−δ), ρ(Xˆ,X)=1.
exploring classic and novel techniques, based on regu- Equivalently, M can be defined as (∆ δ,ρ,D) PAC-
f
larizationanddataaugmentationtechniques,toimprove advantage private if the posterior advantage measured in
intrinsic instability. In particular, we show how the use f-divergence satisfies
of a random unitary matrix in Principal Component δ 1−δ
Analysis (PCA) can essentially eliminate superficial
∆
f
δ =D
f
(1
δ
∥1
δo
ρ)=δ
o
ρf(
δρ
)+(1−δ
o
ρ)f(
1−δρ
),
o o
instability.
where (1−δρ) represents the optimal prior success rate,
4) Empiricalverificationforend-to-endprivacy:Finally, o
we provide experimental support, based on simulated δρ =inf Pr (ρ(X′,X)̸=1),
o X′∈X∗
attacks to validate our privacy guarantees. We convert X∼D
the theoretical mutual information guarantees into pos- a p n a d ram 1 δ ete a r n s d δ 1 δ a o ρ nd re δ p ρ r , es r e e n s t pe tw ct o ive B ly e . rn H o e u r l e li , d D istri i b s u s ti o o m n e s o f f -
o f
2.Noisevaryingacrossoutputdimensions. divergence.

In [13], D is selected to be the KL-divergence and it is
f
shown that,
(cid:0) (cid:1)
∆
KL
δ =D
KL
(1
δ
∥1
δo
ρ)≤MI X;M(X) , (1)
where MI(·,·) represents mutual information and
D
KL W
(
e
1
n δ
∥
ow
1
δo d
ρ)
efi
=
ne
δ
t
l
h
n
e
(
s δ
δ
to ρ a
)
nd
+
ar
(
d
1
M
−
e
δ
m
)
b
ln
er
(
s 1 h
1
−
−
ip δ
δ
o ρ I
)
n
.
ferenceAttack
(MIA) [6], formalized to match PAC Privacy below.
Definition 2 (Membership Inference Attack). Given a finite
data pool U = {u ,u ,··· ,u } and some processing
1 2 N
mechanismM,X isann-subsetofUrandomlyselected.An
informed adversary is asked to return an n-subset Xˆ as the
membership estimation of X after observing M(X). We say
Figure1.Asimple4-stepprocesstoautomaticallyprivatizeablack-box
M is resistant to (1−δ i ) individual membership inference algorithm M. We first measure the stability of M by computing Yi =
for the i-th datapoint u
i
, if for an arbitrary adversary, M(Xi)onvaryingsubsetsofdataXi.Wethenusethevarianceofthe
Pr (1 =1 )≤1−δ . Here, 1 outputdistributionYi toestimatetherequirednoisenecessarytoprivatize
X←U,X˜←M(X) ui∈X ui∈Xˆ i ui∈X M. Finally, we release a noisy version of the learned vector on a new,
(1 ui∈Xˆ ) is an indicator which equals 1 if u i is in X (Xˆ). randomsubsetXj.
In this paper, we will use PAC Privacy to provably noise is added. We denote X as the complete training
train
and automatically measure the privacy risk. We will also dataset which X is sampled from. We denote
j
qualitatively (and occasionally quantitatively) compare our
|X |
resultstopriorworkwithDifferentialPrivacy(DP);itsformal r := j
definition is presented below. |X train |
Definition 3 ((ϵ,δ¯) Differential Privacy [1]). Given a data as our subsampling rate; that is, r is the fraction of the
universe X∗, we say that two datasets S,S′ ⊆ X∗ are data that will be used to learn our released output Y . For
j
adjacent, denoted as S ∼S′, if S =S′∪s or S′ =S ∪s our experiments, we choose r =0.5, or 50%. Let D be the
for some additional datapoint s. A randomized processing uniformly random distribution over all possible r|X |-
train
function M is said to be (ϵ,δ¯)-differentially-private (DP) sized subsets of X ; as such, X is sampled from D.
train j
if for any pair of adjacent datasets S,S′ and any set o in We represent each x∈X as a vector, with its l -norm
j 2
the output space O of M, it holds that Pr(M(S) ∈ o) ≤ bounded by a known constant. In order to provide a private
eϵ·Pr(M(S′)∈o)+δ¯. representation of Y = M(X), we follow the framework
of PAC Privacy [13] to determine the minimal noise we
We can interpret DP in a context of the posterior success
must add to the vector Y. We aim to minimize the noise to
rate for successful membership inference. In the same setup
maximize utility of the output vector Y. Following the steps
of Definition 2, if n= N 3, i.e., each datapoint is included
2 described in Figure 1, we can use the stability of M on
in X with probability 1/2, and M satisfies (ϵ,δ¯)-DP, then
distinct X ’s (which are drawn from the same distribution
by [17], [18], the posterior success rate (1−δ ) is upper i
i D as X ) to determine the required noise to provide privacy
bounded by j
for M(X ).
1−δ¯ j
1−δ ≤1− . (2) To do this, we first compute M on distinct subsampled
i 1+eϵ
datasets X ...X , which are independent and identically
1 m
distributed subsets of X , drawn from the same distri-
3. Automatic Privatization train
bution as X , which means |X | = |X |. We denote m
j i j
as the round complexity of the noise estimation process.
3.1. A Template for Provable Privacy
The value of m is induced by the required precision of our
convergence guarantee. This computation produces output
In this section, we present a formal template for privatiz-
vectors Y ...Y . We describe this in further detail in Sec-
1 m
ing black-box algorithms using PAC Privacy. The key steps
tion 4. We can then use the variance of the Y ’s (along with
i
of this technique are summarized in Figure 1.
appropriate security parameters) to estimate the minimum
In particular, we consider any black-box algorithm M.
noise required to add to the output of M(X ), in order
j
The goal of our template is to release Y = M(X) for a
to provide a meaningful bound on the mutual information,
secret input X . We want to bound the posterior advantage
j which in turn bounds the posterior advantage. An illustrative
the adversary gains upon observing Y =M(X ) by adding
j j instantiation of this procedure on the K-Means algorithm is
noise to Y . Y is an arbitrary learned statistic about the
j j provided in Figure 2.
input X that is exposed to the adversary after appropriate
j In our analysis, we make the conservative assumption
that the adversary knows D, meaning the adversary knows
3.Forthegeneralcase,onecanperformsimilarreasoningbysolving
X and the sampling strategy, which is not typically
aconstrainedlinearprogramwithrespecttoTypeIandTypeIIerrorsas train
describedbyEqn.(1)in[17]. true in the real world. Importantly, the computed posterior

Figure2.ToinstantiatePAC-PrivateK-Means,wefirststartwithourinputdataset,Xtrain,ofsizen.WeapproximatethedistributionDbydrawingm
samplesX1...Xm,whereeachXi isarandomsubsetofXtrain,with|Xi|=n/2.WethencomputethelearnedcentroidsofeachXi.Thevariance
amongthesecentroidsdeterminesthenoiserequiredtoprivatizeouralgorithm.Whenthecentroidsareclosetogether,thealgorithmisstableandthus,the
requirednoisetoprivatizeitissmall.Finally,wegenerateanew,randomsubsetXj andcomputethecorrespondingcentroids;Xj isthesecretsetthatthe
adversarywantstodiscover.Weaddtherequirednoiseandpubliclyreleasetheperturbedcentroidsinthelaststep.Thenoiseguaranteesthattheadversary
hasaboundedadvantageinidentifyingwhetheranyindividualdatapointwasusedtoconstructthefinalreleasedcentroids.
advantage will hold for this adversary or a weaker one Mutual PosteriorSuccessRate(po)
Information Priorp=50% Priorp=1%
with only partial or no knowledge of X . Note that the
train 1/128 56.241% 2.477%
sampledX j ishiddenfromtheadversary,andthereforethe 1/64 58.815% 3.213%
participation of a particular data element in X is unknown 1/32 62.434% 4.364%
j
to the adversary. 1/16 67.490% 6.200%
1/8 74.464% 9.171%
The posterior advantage holds for an arbitrary inference
1/4 83.789% 14.057%
task ρ on the input dataset X. The classic membership 1/2 95.181% 22.177%
attack by [6] defines a specific ρ, where the goal is to 1 100% 35.729%
determine whether a fixed sample x was included in X; that 2 100% 58.103%
is,ρ(X¯,X)=1ifX¯ correctlypredictsthemembershipofx. 4 100% 92.582%
Other attacks may include reconstruction attacks (recovering TABLE1.MUTUALINFORMATIONCANBERELATEDTOTHE
X), norm estimation, or others, e.g., [14], [15]. THEORETICALMAXIMALPOSTERIORSUCCESSRATEFORDIFFERENT
PRIORSUCCESSRATESUSINGEQUATION(3).
We make a few key observations about this template:
1) M is treated as a black-box. The magnitude of the
below as
addednoisedependssolelyontheoutputdistributionof
(cid:18) (cid:19) (cid:18) (cid:19)
Y i ’s. This allows us to generate a privacy template for p ln p o +(1−p )ln 1−p o ≤MI(X ;Y ) (3)
complex black-box algorithms in an instance-specific o p o 1−p i i
(i.e., specific to X ) manner.
train where p is the prior success rate and p is the posterior
2) We observe that the magnitude of added noise only o
success rate. Table 1 provides the theoretical maximal
impacts the posterior advantage of the inference task.
posterior success rates for two different prior success rates
We make no assumptions on the specific inference
of 50% and 1%. The prior success rate p for a subsampling
task of the adversary; rather, PAC privacy allows us to
rate r equals max(r,1−r) for an individual membership
bound the mutual information between the output Y
inference task; we choose r =0.5 to minimize p to 50%.4
and the secret input X, bounding the maximal posterior
However,pcanbemuchlowerforageneralizedmembership
advantage. We further discuss the relationship between
inference task for the same r (e.g., 1%) (cf. Appendix C).
mutual information and the posterior advantage for WecanuseEquation(2)tointerpret(ϵ,δ¯)-DPparameters
specific membership inference attacks in Section 8.
as posterior success rates. For example, a (0.36,0)-DP
3) Inordertomeaningfullymeasurethe(co)varianceacross
((2.98,0)-DP) corresponds to a posterior success rate of
Y ’s, the outputs on varying inputs X must lie on the
i i 58.815% (95.181%) for a prior of 50%. This is useful in
same output space. In particular, we must canonicalize
calibrating mutual information in Table 1 with a DP ϵ.
ouroutputs.Thatis,ifweassumeY isalearnedvector,
i
it must remain in the same order and even simpler,
3.2. Privacy vs. Utility
the same length. For certain tasks, like regression, this
appears obvious, since a learned weight vector has
In this section, we discuss techniques to reduce the
fixed dimension and order. However, for unsupervised
instability in the outputs of our algorithms. That is, we
learning tasks (e.g., clustering), or certain classification
first classify varying causes of instability in the output
algorithms, this becomes non-trivial.
distributions for a black-box algorithm M:
We can then use Equation (1) to convert the mutual
information guarantee to the maximal posterior advantage
4.Thepriorsuccessrateofpositiveidentificationofindividualmember-
for an arbitrary inference task. We can expand Equation (1) shipequalsr.

1) Intrinsicinstability:Wedenoteanalgorithm’sintrinsic k’th column and X[i][j] to represent the element at row i
instability as instability that cannot be reduced without and column j. For a vector v, we denote v[i] as the i’th
semantically changing the output of the algorithm. element. In this algorithm, we compute a matrix G, with
2) Superficial instability: We denote an algorithm’s su- dimension m×d, and σ [k] := Var(G ) represents our
m k
perficial instability as an instability in the output that estimate for the variance of M(X) along direction A . We
k
does not reflect a semantic difference in the output. choose m such that m is large enough that our convergence
This can be addressed by canonicalizing our outputs, criterion, represented by f is satisfied. In particular, for our
τ
by representing them in a consistent manner. experiments, we choose f to be the maximal element-wise
τ
In this work, we explore techniques to reduce both types of difference between σ and σ with τ = 10−6, where
t t−1
instability in a set of widely-used algorithms. σ represents the empirical estimate of the variance in the
t
We first consider a simple example of superficial in- directions A for i∈[1,d] at round t. After computing Σ ,
i B
stability in unsupervised learning algorithms. In general, weaddGaussiannoiseB ∼N(0,Σ )AT toeachelementof
B
unsupervised learning algorithms provide a mechanism for the output M(X). For our experiments, we choose A=I .
d
clustering. However, by definition, these clusters do not
have labels. Thus, an algorithm could return the same set of Algorithm 1 Anisotropic Noise Determination of Determin-
clusters in any order; while the ordered vector appears very istic Mechanism M
different, the true result is semantically the same. In this Input: The input distribution D represented by X
train
case, canonicalizing the output is near-trivial; we can simply and a sampling strategy, τ as the precision required for
assign arbitrary labels to each cluster and choose labels to convergence, f as the convergence function, deterministic
τ
minimize the variance across Y i ’s. mechanism M:Xn →Yd, mutual information requirement
We now consider an example of intrinsic instability. In β, d×d unitary projection matrix A.
this, we consider the random forest algorithm [19]. The goal
1) m:=1, σ := null, G:=null.
0
of this algorithm is to classify different classes within a
2) while m≤2∥f (σ ,σ )≥τ:
τ m−1 m
dataset, by constructing several decision trees. Each decision
a) Draw X ∼D.
tree chooses a subset of features to train on; then, each m
b) y :=M(X ).
level of the tree splits the dataset into subsets in order to m m
c) Compute g =[y ·A ,y ·A ,...,y ·A ].
minimize entropy or Gini impurity [19]. These algorithms m m 1 m 2 m d
d) Row append g to G:
are known to be unstable, since small changes to the input m
datasetcanleadtosignificantchangesinthethresholdvalues. GT :=g .
m m
In Section 5, we discuss how we modify this algorithm to
e) Compute the vector σ where σ is a vector of
provide meaningful guarantees in our framework. m m
Finally, we note that in the classic non-private setting length d and σ m [k] is the empirical variance of G k .
for these algorithms, stability is useful primarily as a proxy f) m:=m+1.
for understanding the generalizability of these algorithms. 3) Calculate the required noise in each direction i as
However, in our setting, stability directly affects the utility
of the privatized algorithm, since it is inversely correlated (cid:112) σ [i] (cid:80) d (cid:112) σ [j]
m m
with the total added noise. This implies that efficiently j=1
e := for i∈[1,d].
privatizing these algorithms involves an inner optimization i 2β
problem, similar to the hyperparameter search typically done
4) Return a diagonal matrix Σ , where Σ [i][i]=e .
using cross-validation. We discuss heuristic strategies for B B i
this search and empirical results in Section 6.
Theorem 1. For an arbitrary deterministic mechanism M,
4. Efficiently Computing Anisotropic Noise
a public unitary matrix A, and Gaussian noise of the form
B ∼N(0,Σ )AT, where σ =Var(M(X)·A ) and Σ is
B i i B
In this section, we formally describe a “best of both
a diagonal matrix with entries
worlds” algorithm that is as efficient as the isotropic noise
additionalgorithmof[13]whilecomputinganisotropicnoise √ (cid:80) d √
σ σ
that minimally affects utility. We then prove that the noise i j
j=1
mechanism satisfies the mutual information guarantees. e := ,
i 2β
4.1. Noise Determination and Guarantees
the output M(X)+B satisfies
The algorithm is described in full in Algorithm 1. We MI(X;M(X)+B)≤β.
denote n as the number of input elements, A as a unitary
projectionmatrix,τ astheprecisionrequiredforconvergence,
f as a function measuring whether our estimator for the
τ
variance has converged, and d as the output dimension. Proof. We first recall Theorem 3 of [13]; this theorem states
For any matrix X, we use the notation X to denote the that
k

5. Algorithms
1
| MI(X;M(X)+B)≤ |     |     |     | lndet(I | +Σ  |     | Σ−1). |     |     |     |     |     |     |     |     |
| ------------- | --- | --- | --- | ------- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
2 d M(X) B In this section, we discuss several classic algorithms and
|     |      |           |     |     |     |     |     | the required | modifications |             | to  | automatically |           | privatize | them.    |
| --- | ---- | --------- | --- | --- | --- | --- | --- | ------------ | ------------- | ----------- | --- | ------------- | --------- | --------- | -------- |
| We  | then | note that |     |     |     |     |     |              |               |             |     |               |           |           |          |
|     |      |           |     |     |     |     |     | For          | all the       | algorithms, |     | we first      | normalize |           | our data |
MI(X;M(X)+B)=MI(X;M(X)A+BA),
|     |     |     |     |     |     |     |     | and separate | it  | into | a training | dataset |     | and a test | dataset. |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | ---- | ---------- | ------- | --- | ---------- | -------- |
since A is unitary and public. We note that data representation has a significant impact
|     |            |     |             |       |         |     |             | on both         | stability | and         | sensitivity. |              | Data can | be normalized |      |
| --- | ---------- | --- | ----------- | ----- | ------- | --- | ----------- | --------------- | --------- | ----------- | ------------ | ------------ | -------- | ------------- | ---- |
| By  | Hadamard’s |     | inequality, | since | Σ M(X)A |     | is positive |                 |           |             |              |              |          |               |      |
|     |            |     |             |       |         |     |             | using different |           | techniques; |              | in practice, |          | we consider   | many |
semi-definite,
normalizationtechniquesandchoosethenormalizationwhich
|     | det(Σ |     | )≤det(diag(Σ |     |     | )), |     |     |     |     |     |     |     |     |     |
| --- | ----- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
M(X)A M(X)A provides the best privatized accuracy for each algorithm and
where diag(Σ ) is the diagonal matrix with the i’th is computationally efficient. We then measure “accuracy”
M(X)A
(alsoreferredtoasutility)onthetestdataset.Allrandomized
| element    | σ i . By | construction, |          | BA has | variance | Σ   | B , which is |            |     |          |         |       |     |     |     |
| ---------- | -------- | ------------- | -------- | ------ | -------- | --- | ------------ | ---------- | --- | -------- | ------- | ----- | --- | --- | --- |
|            |          |               |          |        |          |     |              | algorithms | are | run with | a fixed | seed. |     |     |     |
| a diagonal | matrix   | with          | elements | e      | . Thus,  |     |              |            |     |          |         |       |     |     |     |
i
|     |     |     | 1   |     |     |     |     | 5.1. Clustering: |     | K-Means |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | ------- | --- | --- | --- | --- | --- |
Σ−1)
| MI(X;M(X)A+BA)≤ |     |     |     | lndet(I | +Σ      |       |       |     |         |            |     |            |            |     |           |
| --------------- | --- | --- | --- | ------- | ------- | ----- | ----- | --- | ------- | ---------- | --- | ---------- | ---------- | --- | --------- |
|                 |     |     | 2   |         | d       | M(X)A | B     |     |         |            |     |            |            |     |           |
|                 |     |     | 1   |         |         |       |       | The | K-Means | clustering |     | algorithm, | originally |     | developed |
|                 |     |     | ≤   | lndet(I | +diag(Σ |       | )Σ−1) |     |         |            |     |            |            |     |           |
d M(X)A B by Lloyd in 1982, aims to partition an input set X into K
2
1 (cid:89) 2β non-overlapping subsets or clusters [20], [21]. Each subset
|     |     |     | =   | ln (1+σ |     | √   | √ ) |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
2 i σ (cid:80) σ i∈[1,K] is defined by its centroid, µ . The objective is to
|     |     |     |     |     |     | i   | j j |     |     |     |     |     | i   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
i √ minimize the sum-of-squares within each cluster, over all
|     |     |     | 1(cid:88) |          | 2β       | σ i |     | the clusters, | i.e., |        |          |     |     |     |     |
| --- | --- | --- | --------- | -------- | -------- | --- | --- | ------------- | ----- | ------ | -------- | --- | --- | --- | --- |
|     |     |     | =         | ln(1+    |          | √   | )   |               |       |        |          |     |     |     |     |
|     |     |     | 2         |          | (cid:80) | σ   |     |               |       |        |          |     |     |     |     |
|     |     |     |           |          |          | j j |     |               |       |        | n        |     |     |     |     |
|     |     |     |           | i        | √        |     |     |               |       |        | (cid:88) |     |     |     |     |
|     |     |     |           |          |          |     |     |               |       | argmin | min∥x    |     | −µ  | ∥2. |     |
|     |     |     | 1(cid:88) | 2β       | σ i      |     |     |               |       |        |          |     | i j |     |     |
|     |     |     | ≤         |          | √        |     |     |               |       | µ      |          | µj  |     |     |     |
|     |     |     | 2         | (cid:80) | σ        |     |     |               |       |        | i=0      |     |     |     |     |
j j
i
|     |     |     |     |     |     |     |     | That | is, the | classic | algorithm | outputs |     | a list of | centroids |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ------- | ------- | --------- | ------- | --- | --------- | --------- |
≤β,
|     |     |     |     |     |     |     |     | corresponding |     | to each | cluster. | We  | observe | that | K-Means |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ------- | -------- | --- | ------- | ---- | ------- |
where the fifth inequality uses the fact that ln(1+x)≤x. requires minimal changes to fit into our PAC privacy frame-
|     |     |     |     |     |     |     |     | work.Thex | ’sarethesecretinput,andthelearnedcentroids |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ------------------------------------------ | --- | --- | --- | --- | --- | --- |
i
|     |     |     |     |     |     |     |     | µ i ’s are | the exposed |     | output. |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | ----------- | --- | ------- | --- | --- | --- | --- |
Theorem1showsthatifAlgorithm1computestheexact
|               |         |               |                |            |             |              |            | In order           | to           | canonicalize |               | the output, |            | we simply       | order      |
| ------------- | ------- | ------------- | -------------- | ---------- | ----------- | ------------ | ---------- | ------------------ | ------------ | ------------ | ------------- | ----------- | ---------- | --------------- | ---------- |
| variance      | of M(X) |               | over the       | directions | A           | for i∈[1,d], | we         |                    |              |              |               |             |            |                 |            |
|               |         |               |                |            | i           |              |            | these centroids    |              | by inferring |               | appropriate |            | cluster labels. | For        |
| can privatize |         | any black-box |                | mechanism  |             | M. The       | primary    |                    |              |              |               |             |            |                 |            |
|               |         |               |                |            |             |              |            | supervised         | learning,    | we           | do            | this by     | choosing   | the             | label that |
| advantage     | of      | Algorithm     | 1              | is that    | it avoids   | building     | the        |                    |              |              |               |             |            |                 |            |
|               |         |               |                |            |             |              |            | is best associated |              | with         | each          | cluster.    | We         | then measure    | test       |
| covariance    | matrix  |               | and subsequent |            | SVD         | (as in       | Algorithm  |                    |              |              |               |             |            |                 |            |
|               |         |               |                |            |             |              |            | accuracy           | by comparing |              | the           | inferred    | cluster    | label           | with the   |
| 1 of [13]),   | while   | determining   |                | sufficient | anisotropic |              | noise for  |                    |              |              |               |             |            |                 |            |
|               |         |               |                |            |             |              |            | true class         | label        | on the       | test dataset. |             |            |                 |            |
| privacy.      | The     | optimal       | A for minimal  |            | noise can   | be           | determined |                    |              |              |               |             |            |                 |            |
|               |         |               |                |            |             |              |            | The                | K-Means      | algorithm    |               | is not      | inherently | designed        | for        |
byestimatingthecovariancematrixandusingSVD.However,
we note that Algorithm 1 computes an empirical estimate unbalanced datasets. To improve stability and generalization,
|        |           |        |       |          |           |     |              | we explore    | oversampling |     | techniques  |       | such | as SMOTE | to  |
| ------ | --------- | ------ | ----- | -------- | --------- | --- | ------------ | ------------- | ------------ | --- | ----------- | ----- | ---- | -------- | --- |
| on the | variance, | rather | than  | the true | variance. |     | In practice, |               |              |     |             |       |      |          |     |
|        |           |        |       |          |           |     |              | automatically | balance      |     | the classes | [22]. |      |          |     |
| using  | A = I     | allows | us to | estimate | σ for     | τ = | 10−6 with    |               |              |     |             |       |      |          |     |
d
| reasonably | small | m   | (cf. Section |     | 7) for | most | algorithms. |     |     |     |     |     |     |     |     |
| ---------- | ----- | --- | ------------ | --- | ------ | ---- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
However, unstable algorithms (e.g., decision trees) on large 5.2. Classification: SVM
| datasets | are | very computationally |     |     | expensive | even | in this |     |     |     |     |     |     |     |     |
| -------- | --- | -------------------- | --- | --- | --------- | ---- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
model. We therefore discuss an alternative, more efficient Consider the multiclass support vector machine algo-
approach where we compute the exact variance with a small rithm [23], [24]. The linear support vector machine problem
change in how the distribution D is constructed — see solves the following optimization problem:
| Appendix | B.          |     |                 |     |        |        |          |     |       |     | n         |     |      |       |     |
| -------- | ----------- | --- | --------------- | --- | ------ | ------ | -------- | --- | ----- | --- | --------- | --- | ---- | ----- | --- |
|          |             |     |                 |     |        |        |          |     | 1     |     | (cid:88)  |     |      |       |     |
|          |             |     |                 |     |        |        |          | min | wTw+C |     | max(0,1−y |     | (wTx | +b)). | (4) |
| We       | can further |     | provide tighter |     | bounds | on the | required |     |       |     |           |     | i    | i     |     |
|          |             |     |                 |     |        |        |          | w,b | 2     |     |           |     |      |       |     |
noise when considering specific inference tasks, e.g., in- i=1
dividual membership inference attacks (cf. Definition 2). Here, the x i ’s are the features, and the y i ’s are the labels;
We use these individual privacy guarantees to provide a these both correspond to the secret inputs. The learned
quantitative comparison between PAC and DP for mean weight vector w, b is the exposed output.
estimation. A complete description of the modifications We use the regularization weight C to trade off between
required to Algorithm 1 for individual privacy is provided the hinge loss and the norm of the learned weight vector.
in Appendix A. Without modification, the standard value of C used is 1. To

accommodatemulticlassstrategies,weconsideraone-versus- to us a set of d′ basis vectors with dimensionality d. We
rest classification strategy. That is, we train K classifiers for can freely optimize over the matrix M as long as it remains
K different classes [25]. unitary, since M is simply a linear map.
After the weight vector has been trained, we can use
Claim 1. The optimal choice for M is of the form M =
it to compute a “per-class” score for a new point x . The
i VUT, where U,V are the left and right singular vectors of
predicted label yˆ is the class with the highest score. Similar
i BAT.
to K-Means, we measure the accuracy of the test dataset
by computing the class label predicted by SVM to the true Proof. This is a simple extension of the orthogonal Pro-
label. crustes problem [27]. That is, M is chosen to minimize
We note that this algorithm may or may not have a lot
min ∥A−MB∥2 ,
of superficial instability. That is, the learned weight vectors F
M;MTM=I
are inherently ordered by the labels of their corresponding
classes; it thus requires almost no modification to fit into the for fixed matrices A and B of dimension d′ × d. We
PACprivacyframework.However,theremaybeseveralnear- suppress the unitary requirement on M for succinctness
optimal solutions with no obvious ordering when regulariza- in the remainder of this argument.
tion is not applied appropriately. Strong regularization (low Expanding the definition of the Frobenius norm gives us
values of C) can reduce the algorithm’s intrinsic instability, that:
though it may come with a utility tradeoff. min ∥A−MB∥2 ,
F
M
5.3. Dimensionality Reduction: PCA =min Tr[(A−MB)T(A−MB)],
M
=min Tr[ATA−BTMTA−ATMB+BTMTMB],
Consider the classic dimensionality reduction algorithm,
M
principal component analysis (PCA) [26]. PCA is used to =min ∥A∥2 +∥B∥2 −2Tr[ATMB],
decomposeamultivariatedatasetintoorthogonalcomponents M F F
that explain the most variance. where the last step uses the fact that M is unitary. This is
Unlike the other algorithms considered, PCA is not equivalent to maximizing Tr[MBAT] by the cyclic property
independently used for a regression or classification task; of matrix traces. For unitary M this is maximized at M =
rather,itistypicallyasubroutine.Weconsideraninitialdata VUT, where U and V are the left and right singular vectors
matrix X ∈Rm×d, with m samples of dimension d, where of BAT.
X ⊂ X is secret. We then reduce the dimensionality
train
of X to be in Rm×d′ using PCA; that is, we compute the
top d′ principal components and denote them as a matrix By canonicalizing our output, we reduce the superficial
S ∈Rd′×d. S is the exposed output. instability; “nearby” subspaces are represented by “nearby”
basis vectors. Without the appropriate canonicalization, the
WeobservethatPCAhassignificantsuperficialinstability.
PCA algorithm is very unstable and difficult to privatize.
In particular, we consider the subspace defined by the basis
vectors [0,1] and [1,0]; this subspace is R2. However, there ForagiveninputX test ,afterrunningPCA,theprojection
of X is computed as X ST, representing the best
are an infinite number of basis vectors with the same span; test test
in fact, any two linearly-independent vectors span R2. This projection of X test into the learned rank-d′ subspace. We
can then “restore” X into the original rank-d subspace
implies that two calls to the PCA algorithm can return the test
by computing X STS, also known as the PCA inverse
same subspace, represented by significantly different basis test
transform; we denote this matrix as X′ and calculate the
vectors.
restoration error as
Inordertocanonicalizethebasisvectors,weconsidertwo
v in e s c t t a o n r c s es as of S t 1 he an P d CA S 2 a , lg w o h ri e th re m S a i nd ∈ d R en d′ o × te d t f h o e r r i et = urn 1 e , d 2. ba W si e s Restoration error (RE) := ∥X ∥ ′ X − te X st t ∥ est ∥ . (5)
observethatwecanchooseaunitarymatrixM andcompute
We use the restoration error as a proxy of our accuracy
MS as an equivalent representation of the basis chosen by
2 metric for other downstream tasks, since low RE would
S . The goal is now to choose M in order to minimize the
2 imply high success rate on any secondary task.
distancesbetweenS andS ;weusethisformulationandthe
1 2
propertiesofsingularvaluedecomposition(SVD)tocompute
5.4. Boosting: Random Forest
the optimal M. We note that the SVD decomposition is
unique up to the sign of the right and left singular vectors.
As mentioned in Section 3, random forest algorithms
Consider the following optimization problem:
typically involve both superficial and intrinsic instability,
min ∥A−MB∥2 , making them an interesting case study for our template.
F
M;MTM=I Wefirstdescribetheclassicrandomforestalgorithm[19]
whereA,B arematricesofbasisvectors,withdimensionality and then describe our modifications for canonicalization.
d′×d and M is any unitary matrix. We first observe that The classic random forest algorithm is an ensemble learning
this models our PCA problem exactly; that is, PCA returns technique which combines several weak classifiers (decision

trees) to make an ensemble model which performs better regularization and data augmentation defenses suggested in
than any of the individual trees. Typically, these decision [9] and [10]. First, we define a data augmentation defense.
trees are trained on subsets of the provided dataset and the That is, we first discretize the possible split values of each
final classification is the plurality vote of the individual trees. level. Thus, the possible split values of a feature L[i] are
For each tree, the algorithm chooses a feature (or subset of in the range [0,1], evenly divided into 1/p segments of
features) to split on. Then, the “value” to split on is chosen length p, where p is a tunable hyperparameter (typically
to minimize a metric – in our case, we use the metric of 0.01). Then, rather than just calculating the entropy of the
weighted entropy. The provided dataset is the secret input, split, we calculate our final split value as
and the learned trees are the exposed output: a number of
v :=argmin (1−w )H +w (H +H ).
trees with corresponding structures and weights. v 1 v 1 v−p v+p
In our setting, we require that the trees all have the We denote the tuple (p,w ) as our augmentation regulariza-
1
same structure. To simplify this, we ensure that all the trees tion parameter. If the entropies of the neighbors (v−p and
are complete and split on the same order of features. Thus, v+p) are also low, then this suggests that the split value
our random forest algorithm has two hyperparameters: the v is robust to small amounts of perturbation. Increasing the
number of trees (a classic requirement for an ensemble weight w forces the algorithm to choose a split that is more
1
model) and the ordered list of the features to split on for robust.
each tree, denoted here as L. The structure of a tree is fully Second, we add l regularization, which adds a penalty
1
determined from the ordered set of features; that is, if there of the form |w v| for any split value v. This follows the
2
are d features, then the tree will have exactly 2d leaf nodes. classicl regularizationscheme,whereweencouragesparsity
1
Each node at level i will split on feature L[i]; the exact in the learned split vector. This is especially important in
value of the split threshold is determined by computing the our setting; intuitively, we only want the complete tree to
minimum weighted entropy across all possible values of the learn a non-degenerate split if the change in entropy is
feature L[i]. significant and thus, the learned split is stable. Thus, our
In particular, each possible “split” on the feature L[i] overall regularization parameter is of the form (p,w ,w ).
1 2
at value v splits the dataset into two sets S (v) (containing
r
elements where feature L[i] has value ≥ v) and S l (v) 6. Experiments
(containing elements where feature L[i] has value < v).
We can calculate the entropy of each split as 6.1. Datasets
(cid:88)
H(S)= −p j logp j , Irisdataset:TheIrisdatasetisavailableintheUCIrvine
j Machine Learning Repository [29]. It is a classic dataset
used in machine learning for supervised and unsupervised
where p is the empirical probability of element j (the
j learning tasks. The goal is to classify three class of irises;
frequency of item j in S divided by |S|). The total entropy
there are 50 instances of each class and 4 features; its small
of a split can be calculated as
size makes privatization difficult. We use 100 datapoints as
H :=|S (v)|H(S (v))+|S (v)|H(S (v)). our training set and 50 as the test dataset.
v l l r r
Rice dataset: The Rice dataset is available in the UC
Wethenchoosethesplitthatminimizestheweightedentropy. Irvine Machine Learning Repository [30]. It contains 3,810
We consider the choice of ordered features akin to early instances of rice, from two distinct species: Osmancik and
workinbaggingschemes[28],wheresubsetsoffeatureswere Cammeo; the goal is to classify the species of rice. Each
chosenforeachtree.Wepassinallthedatatoeachdecision example contains 7 features such as area, perimeter and
treeandoutputasimplemajorityvoteofthetreesasthefinal eccentricity. We use 70% of the dataset for training and the
decision of our random forest. In this setting, the features x i remaining 30% as the test dataset.
and the labels y i for our training data represent our secret Dry Bean dataset: The Dry Bean dataset is available
input. The coefficients of the learned trees represent the in the UC Irvine Machine Learning Repository. This dataset
exposed output. As with the prior algorithms, after the trees contains seven different types of dry beans; there are 13,611
areexposed,wecanmeasurethetestaccuracybycomparing instancesofdatawith16featureseach[31].Examplefeatures
the learned classification of a test data point to its true label. include area, perimeter and eccentricity. We use 70% of the
We note that classic regularization schemes on decision datasetfortrainingandtheremaining30%asthetestdataset.
trees (or random forests) focus on pruning the depth of the CIFAR-10 dataset: Finally, we consider the CIFAR-10
tree or allowing the tree to split on a maximum number of dataset [32]. This dataset consists of 60,000 images across
features at each level [19]. Neither of these are consistent 10 classes. The classes represent varying objects (e.g., “cat”
with our framework. In particular, the former does not allow or “deer”) and there are 6,000 images per class. Each image
for an efficient canonicalization since the trees will have isrepresentedasalength-3072vector.Weuse50,000images
different structures. The latter is irrelevant for us, since our as the training dataset and the remaining 10,000 as the test
trees split on a single feature at each level. dataset. We only use CIFAR-10 for the PCA algorithm;
We use two techniques intended to increase the stability images are not particularly appropriate for K-Means, SVM,
of the random forest algorithm, following the form of and Random Forest.

6.2. Experimental Design can be used, with different privacy-utility tradeoffs; we do
|     |     |     |     |     |     |     | not explore | those here.) |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ------------ | --- | --- | --- |
Foreachofourexperiments,wefollowthetemplatefrom To make a meaningful comparison, we compare DP and
Figure 1. We first choose our required privacy guarantee, PAC fixing the posterior success rate. A given posterior
represented by an upper bound on the mutual information success rate for membership inference can be translated to
(MI) between the input and output to our algorithm, M. a particular ϵ-DP guarantee using Equation (2). Similarly,
|     |     |     | MI  |     | 1   | = 2−7 |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- |
In our experiments, we vary between and mutual information bounds and posterior success rates are
128
4=22. We then estimate the stability of M, on the training related by Equation (3) (also see Table 1). We can therefore
data X . To do this, we repeatedly randomly sample compare the expected l distance between DP and PAC
| train |     |     |     |     |     |     |     |     | 2   |     |     |
| ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
X ⊂ X where each X from i = 1...m satisfies estimated means and the true means for the same success
| i             | train |       | i        |        |       |            |       |             |     |     |     |
| ------------- | ----- | ----- | -------- | ------ | ----- | ---------- | ----- | ----------- | --- | --- | --- |
| |X i |:=0.5|X | train | |. We | denote m | as the | round | complexity | rates | in Table 2. |     |     |     |
of the algorithm and we increment m until our estimator As discussed in Section 4, we compare DP with PAC for
satisfiesourprecisionrequirements,asshowninAlgorithm1. bothindividualprivacyandarbitraryinferencetasks(denoted
We then compute the stability of M as a function of the as global privacy). We observe that both PAC individual and
|     | M(X | )   |     | X   | ...X |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
variance of i over all subsets 1 m . We use this global privacy, the error for small values of MI on Iris is
to compute the noise required to privatize M, which we larger than its corresponding DP error. When the dataset
denote as ∆(M,MI). grows larger (and thus, mean estimation more stable), both
For all of our algorithms (Mean, K-Means, SVM, PCA, individualandglobalPACprivacyguaranteesshowempirical
| and Random | Forest), |     | we implement |     | the noise | estimation |             |              |                     |              |     |
| ---------- | -------- | --- | ------------ | --- | --------- | ---------- | ----------- | ------------ | ------------------- | ------------ | --- |
|            |          |     |              |     |           |            | performance | improvements | over the worst-case | differential |     |
algorithm of Section 4 to determine additive noise. privacy guarantees. Isotropic noise with PAC global privacy
We measure the utility of the baseline and both the increases the l -norm distance after privatization by up to
2
| isotropic | and anisotropic |     | privatized |     | versions | of M. In | 2.5×. |     |     |     |     |
| --------- | --------------- | --- | ---------- | --- | -------- | -------- | ----- | --- | --- | --- | --- |
particular, we first run M on the entire X and calculate Itisimportanttonotethatthethreeprivacyguaranteesare
train
the accuracy of M(X train ) on the test dataset X test . This notequivalentinthesemanticsense.Inparticular,PACglobal
provides our accuracy metric for the baseline non-private privacy provides a posterior bound for arbitrary inference
algorithm; we denote this as the “baseline accuracy” of M. tasks, e.g., the generalized membership attack discussed
Then,weconstructtwoprivatizedalgorithmsby,respectively, in Appendix C. While PAC individual privacy provides
adding the required anisotropic noise and isotropic Gaussian better mean estimates than PAC global privacy across all
noise to each element of the trained vector M(X ). The the datasets; it is a weaker privacy guarantee, focusing on
j
required noise is sampled from a Gaussian with zero mean individual membership. On the other hand, the differential
and variance determined by Algorithm 1. This creates two privacy results provide a worst-case guarantee, while both
privatized trained vectors, M P (X j ) (anisotropic noise) and sets of PAC results are instance-based.
| M (X | ) (isotropic | noise); | we then | compute |     | the accuracy |     |     |     |     |     |
| ---- | ------------ | ------- | ------- | ------- | --- | ------------ | --- | --- | --- | --- | --- |
Q j
| of M (X         | ) and | M (X      | ) on X      | ; these | are,      | respectively, | 6.4. | K-Means |     |     |     |
| --------------- | ----- | --------- | ----------- | ------- | --------- | ------------- | ---- | ------- | --- | --- | --- |
| P               | j     | Q         | j           | test    |           |               |      |         |     |     |     |
| the anisotropic | and   | isotropic | “privatized |         | accuracy” | of M for      |      |         |     |     |     |
trial.
a single Our results are averaged over 1000 trials for Aspreviouslydiscussed,weexpectK-Meanstobeeasily
each setting. compatible with the PAC Privacy framework; results are
| We  | now provide |     | results across | varying |     | datasets and | provided | in Figure 3. |     |     |     |
| --- | ----------- | --- | -------------- | ------- | --- | ------------ | -------- | ------------ | --- | --- | --- |
algorithms. All code used is provided at https://github.com/ We observe that the baseline accuracy on our test set
| mayuri95/pac | algs. |     |     |     |     |     |          |                       |                    |             |       |
| ------------ | ----- | --- | --- | --- | --- | --- | -------- | --------------------- | ------------------ | ----------- | ----- |
|              |       |     |     |     |     |     | is above | 90% for the Iris      | and Rice datasets. | Even on     | small |
|              |       |     |     |     |     |     | datasets | like Iris, we observe | a negligible       | gap between | the   |
6.3. Warmup: Estimating the Mean baseline and privatized algorithms for MI ≥ 2−4. On the
|     |     |     |     |     |     |     | Rice | dataset, the centroids | are quite stable | and thus, | we see |
| --- | --- | --- | --- | --- | --- | --- | ---- | ---------------------- | ---------------- | --------- | ------ |
Mean estimation is simple enough that we can provide no meaningful difference between privatized accuracy and
a quantitative, head-to-head comparison between PAC and the non-private baseline for all MI.
DP, since DP does not require significant changes beyond l - IntheDryBeandataset,theunderlyingbaselineaccuracy
2
norm clipping for bounded sensitivity. For our experiments, is quite low (≈73%) and the anisotropic privatized accuracy
we do a search to find the optimal clipping threshold to is below 50% for all MI<2. As discussed in Section 5, we
minimize the overall distance between the privatized mean posit that the low baseline accuracy for Dry Bean is due to
estimate and the true mean. For a given clipping threshold the class imbalance. Thus, we explore improving stability
C and dataset size n, the global sensitivity for the mean using oversampling techniques, as seen in Figure 4.
C/n. ϵ-DP We observe negligible differences due to oversampling
| estimate | is  | The | required | noise | to provide | an  |     |     |     |     |     |
| -------- | --- | --- | -------- | ----- | ---------- | --- | --- | --- | --- | --- | --- |
guarantee is then a Laplacian with scale C/(nϵ) [33]. for the Iris and Rice datasets, since the original datasets
In contrast to DP, which can use the entire dataset, do not have a class imbalance. However, in the Dry Bean
PAC requires an input distribution, which we derive from dataset, we first observe a significant increase (≈12%) in
subsampling. PAC does not require clipping. We chose the the baseline accuracy. Improved stability also means that
subsampling rate r =0.5 to minimize prior success rate for the algorithm becomes easier to privatize and we observe
an individual membership inference attack. (Any 0<r <1 utilityof≈80%atMI=2−3.Thus,stabilitytechniqueslike

|     |         |        |     | ϵ=1.64;   |     |     | ϵ=0.73;   |     |           | ϵ=0.36; |     |     |
| --- | ------- | ------ | --- | --------- | --- | --- | --------- | --- | --------- | ------- | --- | --- |
|     | Dataset | Metric |     | 1−δ=0.84; |     |     | 1−δ=0.67; |     | 1−δ=0.59; |         |     |     |
|     |         |        |     | MI=1/4    |     |     | MI=1/16   |     |           | MI=1/64 |     |     |
(1.18×10−5,0.0092)
|     | Iris DifferentialPrivacy |     |     |     |     |     | (0.0023,0.021) |     |     | (0.008,0.042) |     |     |
| --- | ------------------------ | --- | --- | --- | --- | --- | -------------- | --- | --- | ------------- | --- | --- |
Iris PACIndividualPrivacy (0.011,0.018) (0.010,0.030) (0.010,0.057)
Iris PACGlobalPrivacy (0.010,0.034) (0.010,0.066) (0.011,0.13)
Rice DifferentialPrivacy (5.68×10−6,5.35×10−4) (5.68×10−6,0.0012) (5.68×10−6,0.0025)
Rice PACIndividualPrivacy (4.74×10−5,5.66×10−5) (4.77×10−5,7.90×10−5) (4.68×10−5,1.36×10−4)
|     |                             |     | (4.73×10−5,1.56×10−4) |     |     | (4.65×10−5,3.01×10−4) |     |     | (4.78×10−5,5.93×10−4) |     |     |     |
| --- | --------------------------- | --- | --------------------- | --- | --- | --------------------- | --- | --- | --------------------- | --- | --- | --- |
|     | Rice PACGlobalPrivacy       |     |                       |     |     |                       |     |     |                       |     |     |     |
|     |                             |     | (3.81×10−5,2.44×10−4) |     |     | (3.81×10−5,5.43×10−4) |     |     | (3.81×10−5,0.0011)    |     |     |     |
|     | DryBean DifferentialPrivacy |     |                       |     |     |                       |     |     |                       |     |     |     |
DryBean PACIndividualPrivacy (2.79×10−5,3.05×10−5) (2.76×10−5,3.62×10−5) (2.87×10−5,5.27×10−5)
DryBean PACGlobalPrivacy (2.77×10−5,8.98×10−5) (2.80×10−5,1.70×10−4) (2.84×10−5,3.42×10−4)
TABLE2.QUANTITATIVECOMPARISONOFDPVS.PACPRIVACYFORPRIVATEMEANESTIMATION.WEPROVIDEPACPRIVACYESTIMATESFORBOTH
GLOBALANDINDIVIDUALGUARANTEESASDISCUSSEDINSECTION4.DPUSESδ¯=0FORVARYINGPOSTERIORSUCCESSPROBABILITIES,(1−δ).DP
CELLSPROVIDEl2DISTANCEAFTERCLIPPINGANDAFTERCLIPPINGANDPRIVATIZATION;PACCELLSPROVIDEl2DISTANCEAFTERSUBSAMPLING
ANDAFTERSUBSAMPLINGANDANISTROPICPRIVATIZATIONUSINGALGORITHM1.ALLRESULTSAREAVERAGEDOVER1000TRIALS.
Figure3.WeplottheaccuracyoftheK-Meansalgorithmwithoutprivati- Figure4.WeobserveasignificantimprovementintheDryBeanbaseline
zationinblue.Wethenshowtheanisotropicprivatizationinorangeand accuracyfrom≈73%to≈85%.Thealgorithmalsobecomeseasierto
| isotropicprivatizationingreen.Theaccuracyismeasuredacrossmutual |     |                                        |     |     |     | privatize. |     |     |     |     |     |     |
| --------------------------------------------------------------- | --- | -------------------------------------- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- |
| informationvaryingfrom2−7                                       |     | to22.Asexpected,weobservebetterutility |     |     |     |            |     |     |     |     |     |     |
usinganisotropicnoiseacrossalldatasetsandmutualinformationvalues.
TheRicedatasetistheeasiesttoprivatize,whiletheDryBeandatasetis dataset is a more stark example of this phenomenon — the
thehardest. noise added is so large that the privatized utility does not
|     |     |     |     |     |     | achieve | > 50% | until MI | > 1. While | the | gap is not | large, |
| --- | --- | --- | --- | --- | --- | ------- | ----- | -------- | ---------- | --- | ---------- | ------ |
oversamplingallowustobothincreasethebaselineutilityof the anisotropic utility is consistently better than the isotropic
| the | underlying algorithm | and the | ease of | privatization. | We  |     |     |     |     |     |     |     |
| --- | -------------------- | ------- | ------- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
counterpart.
consider this a crucial win-win situation for PAC Privacy, We observe a similar trend on the Rice dataset for low
where privacy and utility are both improved. MI values. However, the Rice dataset is easier to privatize
|     |     |     |     |     |     | and | the anisotropic | privatized | algorithm | achieves | over | 70% |
| --- | --- | --- | --- | --- | --- | --- | --------------- | ---------- | --------- | -------- | ---- | --- |
MI=2−4.
| 6.5. | Support Vector | Machines | (SVM) |     |     | accuracy | at        |               |         |         |            |     |
| ---- | -------------- | -------- | ----- | --- | --- | -------- | --------- | ------------- | ------- | ------- | ---------- | --- |
|      |                |          |       |     |     |          | There are | many possible | reasons | for the | difference | in  |
Our initial results on SVM, without any additional performance between the baseline and privatized algorithms.
regularization (C =1.0), are summarized in Figure 5. We consider two main cases, corresponding to superficial
We first consider the Iris dataset. For sufficiently large and intrinsic instability, respectively. We observe that many
MI (> 1), the utility loss due to privatization is minimal. sources of superficial instability can be resolved by regular-
However, whenwe tighten themutual information guarantee, ization. That is, regularization provides a technique to order
the magnitude of required noise increases until the utility multiple solutions which provide similar utility, by simply
2−4,
impact is quite severe – at an MI guarantee of the choosingthesimplestone(lowestnorm).However,increasing
privatized algorithm has a utility ≈ 55%. The Dry Bean regularizationtoomuchcaninterferewiththebaselineresults

accuracy,acrossallpossiblemutualinformationbounds–that
|     |     |     |     |     |     |     |     | is, the baseline |             | accuracy       | drops            | from ≈95%    |               | to ≈78%.    | This       |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------- | ----------- | -------------- | ---------------- | ------------ | ------------- | ----------- | ---------- |
|     |     |     |     |     |     |     |     | shows that       | the         | regularization |                  | is strong    | enough        | to          | overpower  |
|     |     |     |     |     |     |     |     | the loss         | in utility; | that           | is, for          | the chosen   |               | value       | of C, the  |
|     |     |     |     |     |     |     |     | optimization     | problem     |                | prefers          | a stable     | low-norm      |             | solution   |
|     |     |     |     |     |     |     |     | more than        | the         | ≈ 17%          | increase         | in accuracy. |               | However,    | we         |
|     |     |     |     |     |     |     |     | observe          | that the    | privatized     | version          | of           | the algorithm |             | shows a    |
|     |     |     |     |     |     |     |     | significant      | increase    | in             | accuracy—over    |              | 70%           | for MI≥2−3. |            |
|     |     |     |     |     |     |     |     | We               | observe     | better         | results          | with the     | Rice          | and         | Dry Bean   |
|     |     |     |     |     |     |     |     | datasets.        | In these    | settings,      | we               | observe      | small         | losses      | of ≈2%     |
|     |     |     |     |     |     |     |     | in the baseline  |             | accuracy.      | For              | the Rice     | dataset,      | we          | observe    |
|     |     |     |     |     |     |     |     | negligible       | losses      | due            | to privatization |              | for all       | values      | of MI,     |
|     |     |     |     |     |     |     |     | indicating       | that        | the increased  |                  | stability    | provides      |             | privacy at |
|     |     |     |     |     |     |     |     | nearly no        | additional  | cost.          | The              | Dry          | Bean dataset  |             | achieves   |
2−3;
|     |     |     |     |     |     |     |     | accuracy      | ≈ 74%      | at       | MI =       | in       | contrast,  |         | the na¨ıve |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | ---------- | -------- | ---------- | -------- | ---------- | ------- | ---------- |
|     |     |     |     |     |     |     |     | SVM algorithm |            | had a    | privatized | accuracy |            | of only | 68% at     |
|     |     |     |     |     |     |     |     | MI=4.         | This again | suggests | that       | larger   | datasets,  |         | which can  |
|     |     |     |     |     |     |     |     | be stabilized | more       | easily,  | can        | provide  | meaningful |         | privacy    |
Figure5.Withoutadditionalregularization,weobservethatitisdifficultto
privatizetheIrisdataset(significantutilitylossforMI≤20)andnearly guarantees with relatively small utility costs.
impossibletoprivatizetheDryBeandataset.TheRicedatasetiseasierto
privatizeandshowsminimalutilitylossesforMI>2−2.
|     |     |     |     |     |     |     |     | 6.6. Principal |     | Component |     | Analysis |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --------- | --- | -------- | --- | --- | --- |
– intuitively, we can prioritize simple solutions over those We then consider the principal component analysis
| with higher | utility. | Thus, | this | cannot | successfully |     | resolve |     |     |     |     |     |     |     |     |
| ----------- | -------- | ----- | ---- | ------ | ------------ | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
algorithmfordimensionalityreduction.Forthisalgorithm,we
| issues where | the          | underlying |     | algorithm     |     | is unstable | due to  |                   |                 |             |              |     |              |      |         |
| ------------ | ------------ | ---------- | --- | ------------- | --- | ----------- | ------- | ----------------- | --------------- | ----------- | ------------ | --- | ------------ | ---- | ------- |
|              |              |            |     |               |     |             |         | evaluate          | its performance |             | by measuring |     | the distance |      | between |
| inherent     | instability, | without    |     | a significant |     | utility     | impact. |                   |                 |             |              |     |              |      |         |
|              |              |            |     |               |     |             |         | the reconstructed |                 | test matrix | X′           | and | the original | test | matrix  |
We experiment with the stability of the SVM algorithm X as defined in Equation (5). Here, we normalize the
test
| by increasing | the    | regularization. |     | We  | vary | the regularization |        |               |              |         |         |              |     |         |         |
| ------------- | ------ | --------------- | --- | --- | ---- | ------------------ | ------ | ------------- | ------------ | ------- | ------- | ------------ | --- | ------- | ------- |
|               |        |                 |     |     |      |                    |        | data by       | individually | scaling |         | each feature |     | between | 0 and   |
| parameter     | C from | Equation        |     | (4) | and  | our results        | are in |               |              |         |         |              |     |         |         |
|               |        |                 |     |     |      |                    |        | 1, a standard |              | feature | scaling | technique.   | We  | first   | explore |
Figure 6. the underlying rank of the datasets.5 These results are
|     |     |     |     |     |     |     |     | summarized | in  | Figure | 7.  |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ------ | --- | --- | --- | --- | --- |
Figure 6. Regularization for SVM affects our datasets in different ways. Figure 7. We measure the percentage of explained variance by the top
IntheIrisdataset,thebaselinealgorithmsuffersasignificantutilityloss principalcomponentsforeachdataset.TheRicedatasethasatotalof7
features,whiletheDryBeandatasethas16features.TheCIFAR-10dataset
duetothestrongregularization(from>90%to≈78%).However,the
gapbetweentheprivatizedandbaselineaccuracydecreases,suggestingthat has3072features—weonlyplottheexplainedvarianceforthetop20
strongregularizationisoptimalfortightmutualinformationguarantees.In dimensions,whichaccountfor≈70%ofthetotalvariance.
theRiceandDryBeandatasets,weobserveasmalllossinbaselineutility In general, we expect that the PCA algorithm will be
forsignificantimprovementsinprivacy.
|     |                |     |          |          |     |         |          | inherently | unstable | at  | tight MI | guarantees |     | when | there are |
| --- | -------------- | --- | -------- | -------- | --- | ------- | -------- | ---------- | -------- | --- | -------- | ---------- | --- | ---- | --------- |
| We  | first consider |     | the Iris | dataset; | the | results | here are |            |          |     |          |            |     |      |           |
severalprincipalcomponentswithsimilar“importance”.That
=10−6.
| plotted for | C   |     | We first | observe | that | the | non-private |     |     |     |     |     |     |     |     |
| ----------- | --- | --- | -------- | ------- | ---- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
version of the algorithm shows a significant decrease in 5.WedonotusetheIrisdatasetduetoitssmalldimension.

| is, if there | are          | two principal | components |           | that | both explain | ≈         |     |     |     |     |     |     |
| ------------ | ------------ | ------------- | ---------- | --------- | ---- | ------------ | --------- | --- | --- | --- | --- | --- | --- |
| 1% of the    | underlying   |               | variance,  | we expect | that | either       | could     |     |     |     |     |     |     |
| be returned  | arbitrarily, |               | even for   | extremely |      | similar      | datasets. |     |     |     |     |     |     |
| We first     | investigate  | d=1           | in Figure  |           | 8.   |              |           |     |     |     |     |     |     |
Figure9.WerunPCAwithvaryingnumbersofcomponents(dintheplots)
forthedifferentdatasets.Withlarged,thebaselinerestorationerrordropsto
nearzeroforRiceandDryBean.Further,forallMIandanisotropicnoise,
|     |     |     |     |     |     |     |     | we can privatize | these algorithms | with minimal | utility | loss. In | contrast, |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------- | ---------------- | ------------ | ------- | -------- | --------- |
wechoosed=3forCIFAR-10inordertoprovidemeaningfulprivatized
|     |     |     |     |     |     |     |     | utility; however, | the baseline | RE remains high | due to | the relatively | low |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------- | ------------ | --------------- | ------ | -------------- | --- |
dimension.Increasingdfurthersignificantlyreducesthestability,making
Figure8.Weobservethatouralgorithmisrelativelystableonallthedatasets.
theprivatizedalgorithmunusable.
WeobservealowRE(<25%)fortheRiceandDryBeandatasetsdueto
thesignificanceofthetopeigenvector.Meanwhile,CIFAR-10hasalarger
baselineRE(≈40%),andshowshigherprivatizedREatlowMI.
|                                                |      |         |               |     |        |                  |     | algorithm | dropssignificantly     | (the l norm | ofthe | noise           | added |
| ---------------------------------------------- | ---- | ------- | ------------- | --- | ------ | ---------------- | --- | --------- | ---------------------- | ----------- | ----- | --------------- | ----- |
| AsobservedinFigure7,mostofthevarianceintheRice |      |         |               |     |        |                  |     |           |                        | 1           |       |                 |       |
|                                                |      |         |               |     |        |                  |     | from d    | = 3 to d = 4 increases | by 100×).   |       | The anisotropic |       |
| and Dry                                        | Bean | dataset | are explained |     | in the | first component. |     |           |                        |             |       |                 |       |
algorithm’sREford=3variesfrom≈300%forMI=2−7
| Thus, we | observe | in  | Figure 8 | that the | restoration |     | error is |     |     |     |     |     |     |
| -------- | ------- | --- | -------- | -------- | ----------- | --- | -------- | --- | --- | --- | --- | --- | --- |
22,
<25% for all mutual information values and we can largely to ≈ 35.5% for MI = which is a small improvement
|         |               |        |               |       |           |           |      | over d=1. |     |     |     |     |     |
| ------- | ------------- | ------ | ------------- | ----- | --------- | --------- | ---- | --------- | --- | --- | --- | --- | --- |
| recover | the original  | matrix | for           | both  | datasets. | Moreover, | the  |           |     |     |     |     |     |
| cost of | privatization |        | is negligible | since | the       | algorithm | con- |           |     |     |     |     |     |
sistently identifies the top eigenvector. In contrast, less than 6.7. Random Forest
| 50% of   | the variance     |       | of the CIFAR-10 |               | dataset | is explained |        |          |             |            |        |            |     |
| -------- | ---------------- | ----- | --------------- | ------------- | ------- | ------------ | ------ | -------- | ----------- | ---------- | ------ | ---------- | --- |
| by the   | first component. |       | Thus,           | this          | shows   | a much       | higher |          |             |            |        |            |     |
|          |                  |       |                 |               |         |              |        | Finally, | we consider | the random | forest | algorithm. | As  |
| baseline | restoration      | error | ≈ 40%.          | Additionally, |         | CIFAR-10     |        |          |             |            |        |            |     |
discussedinSection5,therandomforestalgorithmisknown
| is the hardest |     | dataset | to privatize | and | achieves | RE  | ≈54% |     |     |     |     |     |     |
| -------------- | --- | ------- | ------------ | --- | -------- | --- | ---- | --- | --- | --- | --- | --- | --- |
tobeunstableandisquitedifficulttoadapttoourframework.
at MI=2−4.
We then consider the same algorithm with higher di- We first test the na¨ıve algorithm with no additional
|           |     |         |        |          |            |     |          | regularization | on the Iris | and Rice datasets.6 |     |     |     |
| --------- | --- | ------- | ------ | -------- | ---------- | --- | -------- | -------------- | ----------- | ------------------- | --- | --- | --- |
| mensions, | as  | seen in | Figure | 9. Here, | we observe |     | that all |                |             |                     |     |     |     |
OurresultsaresummarizedinFigure10.Weuseasingle
| the datasets | show | a significant |     | decrease | in restoration |     | error |     |     |     |     |     |     |
| ------------ | ---- | ------------- | --- | -------- | -------------- | --- | ----- | --- | --- | --- | --- | --- | --- |
for the non-private baseline; this is expected since we are tree for the Iris dataset with depth 3. We use 3 trees for the
increasingthenumberofdimensionskeptandthus,capturing Rice dataset, with depth 3.
|         |              |     |                 |     |         |     |     | As expected, | the privatized | version | of  | random | forest |
| ------- | ------------ | --- | --------------- | --- | ------- | --- | --- | ------------ | -------------- | ------- | --- | ------ | ------ |
| more of | the variance |     | in the original |     | matrix. |     |     |              |                |         |     |        |        |
For the Rice dataset, we observe that the anisotropic without additional regularization shows significant instability
noise has anisotropic privatized RE below 5% for all values for our datasets. Further investigation shows that there are
of MI. The Dry Bean dataset shows similar results with several possible causes for the instability within a tree:
anisotropic privatized RE below 9% for all MI. These both • When there are a small number of samples that are
represent significant improvements in utility compared to the in a path, the optimal “threshold” value to split on is
d = 1 setting; this suggests that we can privatize PCA on unstable. We resolve this by providing regularization
| such large | datasets | with | large | enough | dimensions | to  | capture | penalties. |     |     |     |     |     |
| ---------- | -------- | ---- | ----- | ------ | ---------- | --- | ------- | ---------- | --- | --- | --- | --- | --- |
most of the variance. In both of these cases, we observe the • The exact threshold value to split on can be noisy due
benefit of anisotropic noise — the corresponding isotropic to the exact set of points that are observed. To improve
algorithmoftenhasmuchworseresultsinhigherdimensions. stability, we consider a fixed set of threshold values
TheCIFAR-10dataset,incontrast,canonlybeprivatized with finite precision.
| for d=3; | we  | note that | the baseline |     | RE for | d=3 | remains |     |     |     |     |     |     |
| -------- | --- | --------- | ------------ | --- | ------ | --- | ------- | --- | --- | --- | --- | --- | --- |
relatively high at ≈ 33.7%. In particular, the eigenvectors 6.Forthisalgorithm,wedonotusetheDryBeandatasetforcomputa-
for d>3 have similar “importance” and the stability of the tionalefficiency.SeeAppendixBforresultsonthisdataset.

Figure12.WechooseourtrialcomplexitymforAlgorithm1bymeasuring
Figure10.Thena¨ıverandomforestalgorithmshowssignificantinstability the change in our variance estimate in each direction A k. When all of
ontheIrisandRicedatasets.TheIrisdatasetachievesover90%accuracyin thedirectionsarestabilizedwithin10−6,wereturnthecurrentvariance
thenon-privatecase,butshowsadramaticlossinutility(downto≈55%
estimate.
forMI=2−4)afterprivatization.TheRicedatasetshowsbetterresults,
|     |     |     |     |     |     | accuracy | due to | privatization |     | over | all values | of MI; | at MI= |
| --- | --- | --- | --- | --- | --- | -------- | ------ | ------------- | --- | ---- | ---------- | ------ | ------ |
with≈91%accuracyinthebaselineand≈74%accuracyafteranisotropic
privatizationatMI=2−4. 2−7, the privatized accuracy is 79.9%.
|     |     |     |     |     |     | For | the Rice | dataset, | we  | observe | a   | small | increase of |
| --- | --- | --- | --- | --- | --- | --- | -------- | -------- | --- | ------- | --- | ----- | ----------- |
The threshold values are sometimes unstable due to the 0.7%inbaselineaccuracyduetoregularization.Additionally,
•
|             |           |            |             |         |              | the improvement |      | in         | stability | is significant; |            | there | is thus a |
| ----------- | --------- | ---------- | ----------- | ------- | ------------ | --------------- | ---- | ---------- | --------- | --------------- | ---------- | ----- | --------- |
| non-uniform |           | spread of  | the feature | values. | To address   |                 |      |            |           |                 |            |       |           |
|             |           |            |             |         |              | negligible      | loss | in utility | between   | the             | privatized | and   | baseline  |
| this, we    | calculate | a weighted | average     | of      | the entropy. |                 |      |            |           |                 |            |       |           |
We choose these three techniques in order to address the algorithm after the increased regularization for all MI values.
issue that the trees cannot be pruned while maintaining a This again shows that unstable algorithms can be privatized
|                   |          |             |                |        |             | with little | utility | cost | when        | the dataset | is       | sufficiently | large     |
| ----------------- | -------- | ----------- | -------------- | ------ | ----------- | ----------- | ------- | ---- | ----------- | ----------- | -------- | ------------ | --------- |
| canonical         | ordering | that can    | be compared    | across | iterations. |             |         |      |             |             |          |              |           |
|                   |          |             |                |        |             | and stable. | Similar | to   | our K-Means |             | results, | this is      | a win-win |
| We now experiment |          | with adding | regularization |        | of the form |             |         |      |             |             |          |              |           |
(p,w ,w ) (cf. Section 5) for the Iris and Rice datasets. situation where improvements in stability increase both the
1 2
|     |     |     |     |     |     | baseline | utility | and allow | us  | to efficiently |     | privatize | complex |
| --- | --- | --- | --- | --- | --- | -------- | ------- | --------- | --- | -------------- | --- | --------- | ------- |
algorithms.
|     |     |     |     |     |     | 7. Convergence  |             |                | of Algorithm |               | 1          |               |              |
| --- | --- | --- | --- | --- | --- | --------------- | ----------- | -------------- | ------------ | ------------- | ---------- | ------------- | ------------ |
|     |     |     |     |     |     | In this         | section,    | we             | discuss      | our           | empirical  | convergence   |              |
|     |     |     |     |     |     | guarantee.      | We          | observe        | that         | Theorem       | 1 provides |               | a mutual     |
|     |     |     |     |     |     | information     | bound       | when           | the          | variances     | are        | estimated     | exactly.     |
|     |     |     |     |     |     | For practical   | guarantees, |                | we           | choose        | trial      | complexity    | large        |
|     |     |     |     |     |     | enough          | such        | that each      | element      |               | of the     | variance      | estimate     |
|     |     |     |     |     |     | convergeswith   |             | veryhigh       | precision    |               | (τ =10−6). | Inparticular, |              |
|     |     |     |     |     |     | we run          | our noise   | estimation     |              | algorithm     |            | and estimate  | our          |
|     |     |     |     |     |     | variance        | vector      | after          | every        | 10 instances. |            | The algorithm | is           |
|     |     |     |     |     |     | considered      | to have     | converged      |              | when          | none       | of the        | estimates in |
|     |     |     |     |     |     | our output      | vector      | have           | changed      | by            | more than  | τ.            | We choose    |
|     |     |     |     |     |     | τ sufficiently  | small       | such           | that         | the           | impact     | of adding     | noise in     |
|     |     |     |     |     |     | the order       | of τ        | is negligible. |              | Results       | are shown  | in            | Figure 12.   |
|     |     |     |     |     |     | Figure          | 12          | provides       | the          | change        | in         | the first     | element      |
|     |     |     |     |     |     | of our variance |             | vector         | for          | varying       | algorithms |               | on the Iris  |
Figure 11. The Iris dataset has a ≈ 4% loss in baseline accuracy due dataset. We observe that the trial complexity varies across
| to regularization; | however, | we achieve | over 85% | privatized | utility for |     |     |     |     |     |     |     |     |
| ------------------ | -------- | ---------- | -------- | ---------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
MI > 2−6. In the Rice dataset, there is a small increase in baseline algorithms and datasets. Stable algorithms (e.g., SVM with
accuracy.Additionally,theprivatizedaccuracyincreasestoachieveover strongregularization)convergenearly100×fasterthanSVM
90%accuracyforallvaluesofMIduetotheimprovementsinstability. without regularization.
We first consider the Iris dataset. As seen in Figure 11, Convergence on the complete covariance matrix would
we add significant regularization; this decreases our baseline require an order of magnitude more trials. In Appendix
accuracy to ≈ 88%. However, we observe small losses in B, we describe an alternate approach to true variance

| computation | that | can be | more | efficient | than | a convergence- |     |     |     |     |     |     |     |     |     |
| ----------- | ---- | ------ | ---- | --------- | ---- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
based approach.
| 8. Empirical   |               | Privacy   | Estimation  |                  |         |               |           |     |     |     |     |     |     |     |     |
| -------------- | ------------- | --------- | ----------- | ---------------- | ------- | ------------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
| Although       | PAC           | Privacy   |             | allows           | us to   | provably      | bound     |     |     |     |     |     |     |     |     |
| the posterior  | advantage     |           | for any     | attack,          | in      | this section, | we        |     |     |     |     |     |     |     |     |
| focus on       | membership    |           | inference   | attacks          | for     | concreteness  | and       |     |     |     |     |     |     |     |     |
| validation.    | The           | objective | of          | membership       |         | inference     | attacks   |     |     |     |     |     |     |     |     |
| (MIA) is       | defined       | in [6]    | as follows: | given            | a       | machine       | learning  |     |     |     |     |     |     |     |     |
| model and      | a single      | datapoint |             | x, the           | goal    | is to         | determine |     |     |     |     |     |     |     |     |
| whether        | x was         | used to   | train       | the model.       |         |               |           |     |     |     |     |     |     |     |     |
| For            | our purposes, | we        | define      | our              | machine | learning      | model     |     |     |     |     |     |     |     |     |
| as the trained | output        | vector    |             | Y , representing |         | some          | statistic |     |     |     |     |     |     |     |     |
i
| about our   | input      | data X         | i ⊂X          | train .    | In K-Means, |              | this vector |            |           |           |           |     |           |            |         |
| ----------- | ---------- | -------------- | ------------- | ---------- | ----------- | ------------ | ----------- | ---------- | --------- | --------- | --------- | --- | --------- | ---------- | ------- |
| corresponds | to         | the list       | of centroids; |            | for         | SVM,         | the vector  |            |           |           |           |     |           |            |         |
| represents  | the list   | of hyperplanes |               | separating |             | the classes. | We          |            |           |           |           |     |           |            |         |
| vary the    | mutual     | information    |               | bound      | and         | focus        | on the Iris |            |           |           |           |     |           |            |         |
| dataset.    | We observe | that           | our           | model      | is trained  | on           | a subset    |            |           |           |           |     |           |            |         |
|             |            |                |               |            |             |              |             | Figure 13. | Empirical | posterior | advantage |     | from LIRA | over 1,000 | trials. |
X where |X | = 0.5(|X |) in all of our experiments. The empirical posterior advantage of the subsampled algorithms are all
| j   | j   |     | train |     |     |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
This indicates that for any particular datapoint x, the prior atmost11%.Fortheprivatizedalgorithms,theempiricaladvantagesare
Pr[x ∈ X ] = 0.5. Table 1 gives a theoretical maximal alwaysbelowthetheoreticalposterioradvantagesofTable1.TheK-Means
j
|           |            |       |     |             |     |        |           | results show | the largest | average | reduction |     | in posterior | advantage | due to |
| --------- | ---------- | ----- | --- | ----------- | --- | ------ | --------- | ------------ | ----------- | ------- | --------- | --- | ------------ | --------- | ------ |
| posterior | advantage, | which | can | be compared |     | to the | empirical |              |             |         |           |     |              |           |        |
privatizationof≈5%overallvaluesofMI.
| advantage | observed. |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| --------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
We consider the Likelihood-Ratio Attack (LIRA) as guarantees, we restrict ourselves to qualitative comparisons
described in [7] and adapt it to the K-Means and SVM between our PAC-privatized algorithms and state-of-the-art
algorithms; details are provided in Appendix D. Our results DP algorithms for the various problems.
ontheIrisdatasetaresummarizedinFigure13.PACPrivacy We first consider K-Means. Early work developed
| is necessarily |            | conservative; |          | in Figure | 13,     | the | empirical  |         |       |      |         |            |           |     |        |
| -------------- | ---------- | ------------- | -------- | --------- | ------- | --- | ---------- | ------- | ----- | ---- | ------- | ---------- | --------- | --- | ------ |
|                |            |               |          |           |         |     |            | DPLloyd | [35], | a DP | version | of Lloyd’s | algorithm |     | for K- |
| posterior      | advantages |               | (denoted | as        | p ) for | the | privatized |         |       |      |         |            |           |     |        |
e Means clustering. Intuitively, DPLloyd adds Laplacian noise
algorithms are significantly lower than the upper bounds each time the approximate centroids are computed. We
given by Table 1 across all mutual information bounds. observe that even for a simple algorithm, providing DP
| We          | observe | a decrease      | in  | the privatized |             | posterior | advan-     |            |             |         |            |                 |              |            |          |
| ----------- | ------- | --------------- | --- | -------------- | ----------- | --------- | ---------- | ---------- | ----------- | ------- | ---------- | --------------- | ------------ | ---------- | -------- |
|             |         |                 |     |                |             |           |            | required   | significant | changes | to         | the algorithmic |              | structure, | e.g.,    |
| tage across | all     | the algorithms. |     | In             | the K-Means |           | algorithm, |            |             |         |            |                 |              |            |          |
|             |         |                 |     |                |             |           |            | fixing the | number      | of      | iterations | for             | convergence. |            | Further, |
we observe the most significant change, from a baseline p e we observe that K-Means is known to be sensitive to the
of 9.6% to 2% at MI=1/128. In the SVM algorithms, for initialcentroidschosen;thus,DPLloydrequiresanewprivate
C = 1.0, we observe an average decrease of ≈ 3% in p ; initialization procedure as well.
e
| for C =10−6, |      | the difference |     | is ≈2%. |     |     |     |              |               |            |            |              |         |            |        |
| ------------ | ---- | -------------- | --- | ------- | --- | --- | --- | ------------ | ------------- | ---------- | ---------- | ------------ | ------- | ---------- | ------ |
|              |      |                |     |         |     |     |     | We           | then consider |            | SVM        | classifiers; | there   | has        | been a |
|              |      |                |     |         |     |     |     | long history | of            | developing | SVMs       | with         | DP      | guarantees | [36],  |
| 9. Related   | Work |                |     |         |     |     |     |              |               |            |            |              |         |            |        |
|              |      |                |     |         |     |     |     | [37]. In     | general,      | these      | techniques | use          | the SVM | algorithm  | to     |
computetheoptimalweightvectorandaddappropriatenoise
Recent work in PAC Privacy addressed the formalization to provide DP guarantees. However, as observed by [38],
|            |              |           |          |            |            |          |           | large training   | sets | led   | to large | weight      | vectors      | with | increased |
| ---------- | ------------ | --------- | -------- | ---------- | ---------- | -------- | --------- | ---------------- | ---- | ----- | -------- | ----------- | ------------ | ---- | --------- |
| of privacy | guarantees   |           | provided | by         | heuristic  | encoding | algo-     |                  |      |       |          |             |              |      |           |
|            |              |           |          |            |            |          |           | noise. Moreover, |      | often | there    | were strong | restrictions |      | on the    |
| rithms,    | and provided | efficient |          | estimation | strategies |          | for these |                  |      |       |          |             |              |      |           |
specialized encoding techniques [34]. objective function (e.g., convexity) to enable tight bounds
Quantitative comparisons to DP for generic mean esti- on the noise. [38] suggests a novel method in order to solve
|             |               |      |            |     |          |            |         | the dual     | problem | of SVM,          | which | approaches |          | the non-private |          |
| ----------- | ------------- | ---- | ---------- | --- | -------- | ---------- | ------- | ------------ | ------- | ---------------- | ----- | ---------- | -------- | --------------- | -------- |
| mation      | were provided |      | in Section |     | 6.3. For | more       | complex |              |         |                  |       |            |          |                 |          |
|             |               |      |            |     |          |            |         | SVM accuracy |         | for sufficiently |       | large      | training | sets.           | However, |
| algorithms, | small         | ϵ-DP | guarantees | are | harder   | to provide | and     |              |         |                  |       |            |          |                 |          |
ofteninvolvesignificantchangestoalgorithmimplementation. note that this is still a white-box mechanism to achieve
Even with white-box changes, the resulting algorithm often privacy, i.e., Laplacian noise was added in each iteration
requires a large dataset with small data dimension to provide and in each iteration, an inner loop is required to choose the
|            |         |             |     |     |           |     |          | pairs of | dual variables |     | to update. |     |     |     |     |
| ---------- | ------- | ----------- | --- | --- | --------- | --- | -------- | -------- | -------------- | --- | ---------- | --- | --- | --- | --- |
| meaningful | utility | guarantees. |     | In  | contrast, | PAC | provides |          |                |     |            |     |     |     |     |
instance-specificguaranteeswithreasonableutilityloss,even Random forests with differential privacy have been
when subsampling small datasets of ≈ 1,000 datapoints with explored less extensively. [39] suggests that differentially-
large output dimension. Given the substantial algorithmic private random forests can be constructed by allocating a
differencesbetweenDPwhite-boxedalgorithmsandthePAC privacy budget across trees, and then across levels of each
black-box approach, and the semantic difference in privacy tree. Each tree is “complete” when either all the features are

| used, the | remaining |     | samples | all belong | to  | the same | class or |             |     |         |         |              |      |       |              |
| --------- | --------- | --- | ------- | ---------- | --- | -------- | -------- | ----------- | --- | ------- | ------- | ------------ | ---- | ----- | ------------ |
|           |           |     |         |            |     |          |          | [2] X. Xiao | and | Y. Tao, | “Output | perturbation | with | query | relaxation,” |
when a maximum height is reached. [40] expands this work Proceedings of the VLDB Endowment, vol. 1, no. 1, pp. 857–869,
2008.
| and constructs |     | differentially-private |     |     | median | forests, | which |     |     |     |     |     |     |     |     |
| -------------- | --- | ---------------------- | --- | --- | ------ | -------- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
also improve the stability of the data structure. However, [3] I. Issa, A. B. Wagner, and S. Kamath, “An operational approach
they still observe significant utility losses for sufficiently toinformationleakage,”IEEETransactionsonInformationTheory,
| small ϵ. |     |          |     |                 |     |           |        | vol.66,no.3,pp.1625–1657,2019. |           |         |             |     |                |     |             |
| -------- | --- | -------- | --- | --------------- | --- | --------- | ------ | ------------------------------ | --------- | ------- | ----------- | --- | -------------- | --- | ----------- |
|          |     |          |     |                 |     |           |        | [4] M.                         | Abadi, A. | Chu, I. | Goodfellow, |     | H. B. McMahan, |     | I. Mironov, |
| Finally, | we  | consider | DP  | for identifying |     | principal | compo- |                                |           |         |             |     |                |     |             |
K.Talwar,andL.Zhang,“Deeplearningwithdifferentialprivacy,”
nents.[41]constructsanear-optimaltechniqueforidentifying
inProceedingsofthe2016ACMSIGSACconferenceoncomputer
| principal | components |     | while | providing | DP. | Their | technique |     |     |     |     |     |     |     |     |
| --------- | ---------- | --- | ----- | --------- | --- | ----- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
andcommunicationssecurity,2016,pp.308–318.
requires datasets with a large number of datapoints, but [5] N.Papernot,S.Song,I.Mironov,A.Raghunathan,K.Talwar,and
the noise scales with the original dimension, making it U´.Erlingsson,“Scalableprivatelearningwithpate,”arXivpreprint
impractical for datasets with large dimension, even if the arXiv:1802.08908,2018.
| true rank          | is constant. |             | We further | observe      |        | that their     | resulting    |                                    |         |           |          |          |                    |             |           |
| ------------------ | ------------ | ----------- | ---------- | ------------ | ------ | -------------- | ------------ | ---------------------------------- | ------- | --------- | -------- | -------- | ------------------ | ----------- | --------- |
|                    |              |             |            |              |        |                |              | [6] R. Shokri,                     | M.      | Stronati, | C. Song, | and      | V. Shmatikov,      | “Membership |           |
| utility guarantees |              | are         | upon       | a secondary  |        | classification | task,        |                                    |         |           |          |          |                    |             |           |
|                    |              |             |            |              |        |                |              | inference                          | attacks | against   | machine  | learning | models,”           | in          | 2017 IEEE |
|                    |              |             |            |              |        |                |              | symposiumonsecurityandprivacy(SP). |         |           |          |          | IEEE,2017,pp.3–18. |             |           |
| rather than        | the          | restoration | of         | the original | matrix |                | task that we |                                    |         |           |          |          |                    |             |           |
evaluate on. In practice, we expect the latter to be a stronger [7] N. Carlini, S. Chien, M. Nasr, S. Song, A. Terzis, and F. Tramer,
guarantee since a perfectly restored matrix would provide “Membershipinferenceattacksfromfirstprinciples,”in2022IEEE
the best utility guarantee on any secondary task. SymposiumonSecurityandPrivacy(SP). IEEE,2022,pp.1897–
1914.
|     |     |     |     |     |     |     |     | [8] H. Hu, | Z.  | Salcic, L. | Sun, G. | Dobbie, | P. S. | Yu, and | X. Zhang, |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ---------- | ------- | ------- | ----- | ------- | --------- |
10. Conclusions “Membershipinferenceattacksonmachinelearning:Asurvey,”ACM
ComputingSurveys(CSUR),vol.54,no.11s,pp.1–37,2022.
We have shown how PAC Privacy can be applied to [9] M.Nasr,R.Shokri,andA.Houmansadr,“Machinelearningwithmem-
bershipprivacyusingadversarialregularization,”inProceedingsof
| privatize | black-box | algorithms |     | by giving |     | a template | that can |     |     |     |     |     |     |     |     |
| --------- | --------- | ---------- | --- | --------- | --- | ---------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
the2018ACMSIGSACconferenceoncomputerandcommunications
| be applied | to  | virtually | any | algorithm. | Using | Algorithm | 1 to |     |     |     |     |     |     |     |     |
| ---------- | --- | --------- | --- | ---------- | ----- | --------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
security,2018,pp.634–646.
| add anisotropic |     | noise | is critical | to  | improving | privacy-utility |     |              |     |              |     |       |           |              |      |
| --------------- | --- | ----- | ----------- | --- | --------- | --------------- | --- | ------------ | --- | ------------ | --- | ----- | --------- | ------------ | ---- |
|                 |     |       |             |     |           |                 |     | [10] Y. Kaya | and | T. Dumitras, |     | “When | does data | augmentation | help |
tradeoffs. withmembershipinferenceattacks?”inInternationalconferenceon
An exciting aspect of PAC Privacy that is demonstrated machinelearning. PMLR,2021,pp.5345–5355.
| most clearly |     | by our | K-Means | results | is  | the potential | win- |     |     |     |     |     |     |     |     |
| ------------ | --- | ------ | ------- | ------- | --- | ------------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
[11] Y.Yin,K.Chen,L.Shou,andG.Chen,“Defendingprivacyagainst
win situation in the algorithm tradeoff space. Stability with moreknowledgeablemembershipinferenceattackers,”inProceedings
respect to input changes is a desirable feature of algorithms, ofthe27thACMSIGKDDConferenceonKnowledgeDiscovery&
DataMining,2021,pp.2026–2036.
becausestablealgorithmsgeneralizebettertonewinputsand
have better worst cases. Concomitantly, stable algorithms [12] Y.Wang,C.Wang,Z.Wang,S.Zhou,H.Liu,J.Bi,C.Ding,and
S.Rajasekaran,“Againstmembershipinferenceattack:Pruningisall
| require | less additive |     | noise | on their | outputs | for | privatization. |     |     |     |     |     |     |     |     |
| ------- | ------------- | --- | ----- | -------- | ------- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
youneed,”inInternationalJointConferenceonArtificialIntelligence,
| One | aspect | that | is worth | exploring |     | in the | future is |     |     |     |     |     |     |     |     |
| --- | ------ | ---- | -------- | --------- | --- | ------ | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
2021.
| the privacy-utility |     | tradeoff |     | at different |     | subsampling | rates. |     |     |     |     |     |     |     |     |
| ------------------- | --- | -------- | --- | ------------ | --- | ----------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
[13] H.XiaoandS.Devadas,“Pacprivacy:Automaticprivacymeasurement
| Additional | future | work | includes |     | using | the compositional |     |     |     |     |     |     |     |     |     |
| ---------- | ------ | ---- | -------- | --- | ----- | ----------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
andcontrolofdataprocessing,”inAdvancesinCryptology–CRYPTO
properties of mutual information [42] to tackle unstable 2023: 43rd Annual International Cryptology Conference, CRYPTO
algorithms such as Stochastic Gradient Descent (SGD). 2023, Santa Barbara, CA, USA, August 20–24, 2023, Proceedings,
Algorithms can be broken down into phases, and noise is Part II. Berlin, Heidelberg: Springer-Verlag, 2023, p. 611–644.
|          |         |         |        |     |                   |     |         | [Online].Available:https://doi.org/10.1007/978-3-031-38545-2 |     |     |     |     |     |     | 20  |
| -------- | ------- | ------- | ------ | --- | ----------------- | --- | ------- | ------------------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- |
| added at | the end | of each | phase. | It  | is conservatively |     | assumed |                                                              |     |     |     |     |     |     |     |
[14] B.Balle,G.Cherubin,andJ.Hayes,“Reconstructingtrainingdata
| that the | noisy | outputs | are | exposed, | similar | to DP-SGD | [4], |     |     |     |     |     |     |     |     |
| -------- | ----- | ------- | --- | -------- | ------- | --------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
withinformedadversaries,”in2022IEEESymposiumonSecurityand
| although   | in PAC | Privacy    |     | the phases | can | be         | hundreds of |                |     |                         |     |           |           |          |      |
| ---------- | ------ | ---------- | --- | ---------- | --- | ---------- | ----------- | -------------- | --- | ----------------------- | --- | --------- | --------- | -------- | ---- |
|            |        |            |     |            |     |            |             | Privacy(SP).   |     | IEEE,2022,pp.1138–1156. |     |           |           |          |      |
| iterations | and    | each phase | can | be treated |     | as a black | box.        |                |     |                         |     |           |           |          |      |
|            |        |            |     |            |     |            |             | [15] J. Hayes, | S.  | Mahloujifar,            | and | B. Balle, | “Bounding | training | data |
reconstructionindp-sgd,”arXivpreprintarXiv:2302.07225,2023.
11. Acknowledgements
[16] D.Gruss,J.Lettner,F.Schuster,O.Ohrimenko,I.Haller,andM.Costa,
“Strongandefficientcache{Side-Channel}protectionusinghardware
transactionalmemory,”in26thUSENIXSecuritySymposium(USENIX
We thank the anonymous reviewers for their detailed and Security17),2017,pp.217–233.
| constructive | feedback. |     | This | work | was supported |     | in part by |     |     |     |     |     |     |     |     |
| ------------ | --------- | --- | ---- | ---- | ------------- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
[17] P.Kairouz,S.Oh,andP.Viswanath,“Thecompositiontheoremfor
| grants from | Cisco | Systems |     | and Capital | One. | Mayuri | Sridhar |     |     |     |     |     |     |     |     |
| ----------- | ----- | ------- | --- | ----------- | ---- | ------ | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
differentialprivacy,”inInternationalconferenceonmachinelearning.
| was supported |     | by the | NDSEG | fellowship |     | from the | DoD and |     |     |     |     |     |     |     |     |
| ------------- | --- | ------ | ----- | ---------- | --- | -------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
PMLR,2015,pp.1376–1385.
a MathWorks fellowship. [18] T.Humphries,S.Oya,L.Tulloch,M.Rafuse,I.Goldberg,U.Hengart-
ner,andF.Kerschbaum,“Investigatingmembershipinferenceattacks
|     |     |     |     |     |     |     |     | under | data | dependencies,” | in  | 2023 | IEEE 36th | Computer | Security |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | ---- | -------------- | --- | ---- | --------- | -------- | -------- |
References
FoundationsSymposium(CSF),2023,pp.473–488.
|     |     |     |     |     |     |     |     | [19] L. Breiman, |     | “Random | forests,” | Machine | Learning, | vol. | 45, no. 1, |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | ------- | --------- | ------- | --------- | ---- | ---------- |
[1] C. Dwork, “Differential privacy,” in International colloquium on pp. 5–32, 2001. [Online]. Available: http://dx.doi.org/10.1023/A%
| automata,languages,andprogramming. |     |     |     |     | Springer,2006,pp.1–12. |     |     | 3A1010933404324 |     |     |     |     |     |     |     |
| ---------------------------------- | --- | --- | --- | --- | ---------------------- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- |

[20] S.Lloyd,“Leastsquaresquantizationinpcm,”IEEETransactionson [39] A.PatilandS.Singh,“Differentialprivaterandomforest,”in2014
InformationTheory,vol.28,no.2,pp.129–137,1982. InternationalConferenceonAdvancesinComputing,Communications
andInformatics(ICACCI),2014,pp.2623–2630.
[21] D.ArthurandS.Vassilvitskii,“K-means++:Theadvantagesofcareful
seeding,”vol.8,012007,pp.1027–1035. [40] S. Consul and S. A. Williamson, “Differentially private random
forestsforregressionandclassification,”2021.[Online].Available:
[22] N. V. Chawla, K. W. Bowyer, L. O. Hall, and W. P. Kegelmeyer, https://api.semanticscholar.org/CorpusID:235365701
“Smote:syntheticminorityover-samplingtechnique,”J.Artif.Int.Res.,
[41] K.Chaudhuri,A.Sarwate,andK.Sinha,“Near-optimaldifferentially
vol.16,no.1,p.321–357,jun2002.
|     |     |     |     |     |     |     | private | principal | components,” | in  | Advances in | Neural | Information |
| --- | --- | --- | --- | --- | --- | --- | ------- | --------- | ------------ | --- | ----------- | ------ | ----------- |
[23] R.-E. Fan, K.-W. Chang, C.-J. Hsieh, X.-R. Wang, and C.-J. Lin, Processing Systems, F. Pereira, C. Burges, L. Bottou, and
“Liblinear: A library for large linear classification,” Journal of K.Weinberger,Eds.,vol.25. CurranAssociates,Inc.,2012.[Online].
| Machine | Learning | Research, | vol. | 9, no. | 61, pp. | 1871–1874, 2008. |                                                |     |     |     |     |                        |     |
| ------- | -------- | --------- | ---- | ------ | ------- | ---------------- | ---------------------------------------------- | --- | --- | --- | --- | ---------------------- | --- |
|         |          |           |      |        |         |                  | Available:https://proceedings.neurips.cc/paper |     |     |     |     | files/paper/2012/file/ |     |
[Online].Available:http://jmlr.org/papers/v9/fan08a.html f770b62bc8f42a0b66751fe636fc6eb0-Paper.pdf
[24] C.-C. Chang and C.-J. Lin, “Libsvm: A library for support vector [42] H. Xiao, “Automated and Provable Privatization for Black-Box
machines,”ACMTrans.Intell.Syst.Technol.,vol.2,no.3,may2011. Processing,”Ph.D.dissertation,MassachusettsInstituteofTechnology,
August2024.
[Online].Available:https://doi.org/10.1145/1961189.1961199
[25] F. Pedregosa, G. Varoquaux, A. Gramfort, V. Michel, B. Thirion, [43] J. Platt, “Probabilistic outputs for support vector machines and
O.Grisel,M.Blondel,P.Prettenhofer,R.Weiss,V.Dubourg,J.Van- comparisonstoregularizedlikelihoodmethods,”Adv.LargeMargin
Classif.,vol.10,062000.
| derplas, | A. Passos, | D.  | Cournapeau, | M.  | Brucher, | M. Perrot, and |     |     |     |     |     |     |     |
| -------- | ---------- | --- | ----------- | --- | -------- | -------------- | --- | --- | --- | --- | --- | --- | --- |
E.Duchesnay,“Scikit-learn:MachinelearninginPython,”Journalof
| MachineLearningResearch,vol.12,pp.2825–2830,2011. |     |     |     |     |     |     | Appendix | A.  |     |     |     |     |     |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- | --- |
[26] K. Pearson, “Liii. on lines and planes of closest fit to systems of Individual Privacy Guarantees
pointsinspace,”TheLondon,Edinburgh,andDublinPhilosophical
MagazineandJournalofScience,vol.2,no.11,pp.559–572,1901. We can provide tighter bounds on the noise required
[Online].Available:https://doi.org/10.1080/14786440109462720
|            |       |           |           |        |              |            | to privatize | algorithms | when | considering |     | specific | inference |
| ---------- | ----- | --------- | --------- | ------ | ------------ | ---------- | ------------ | ---------- | ---- | ----------- | --- | -------- | --------- |
| [27] G. H. | Golub | and C. F. | Van Loan, | Matrix | computations | (3rd ed.). |              |            |      |             |     |          |           |
tasks.
USA:JohnsHopkinsUniversityPress,1996.
|     |     |     |     |     |     |     | In particular, | for | a d-dimensional |     | mean | estimation | mech- |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --------------- | --- | ---- | ---------- | ----- |
[28] T.K.Ho,“Therandomsubspacemethodforconstructingdecision anism M, we may decompose M as M ,··· ,M , where
forests,”IEEETransactionsonPatternAnalysisandMachineIntelli- 1 d
gence,vol.20,no.8,pp.832–844,1998. M i isthei-thcoordinateaverageofinputX.Wemayfollow
|     |     |     |     |     |     |     | the composition | results | (Theorem |     | 7) of [13] | to upper | bound |
| --- | --- | --- | --- | --- | --- | --- | --------------- | ------- | -------- | --- | ---------- | -------- | ----- |
[29] R.A.Fisher,“Iris,”UCIMachineLearningRepository,1988,DOI: (cid:0) (cid:1)
|     |     |     |     |     |     |     | the mutual | information | of  | MI X;M(X) |     | by the | sum of the |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ----------- | --- | --------- | --- | ------ | ---------- |
https://doi.org/10.24432/C56C76.
|     |     |     |     |     |     |     | KL divergence | bound | of  | each M | (X). |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------- | ----- | --- | ------ | ---- | --- | --- |
[30] I. Cınar and M. Koklu, “Classification of rice varieties using i
|            |              |           |               |     |         |                | In particular, | for | individual | privacy | where | the | adversary |
| ---------- | ------------ | --------- | ------------- | --- | ------- | -------------- | -------------- | --- | ---------- | ------- | ----- | --- | --------- |
| artificial | intelligence | methods,” | International |     | Journal | of Intelligent |                |     |            |         |       |     |           |
x∗
SystemsandApplicationsinEngineering,2019.[Online].Available: aims to infer whether a datapoint is selected in the input
https://api.semanticscholar.org/CorpusID:208105752 set or not, rather than calculating the empirical variance over
O¨zkan, all possible subsets, we can simply compute the average
| [31] M. Koklu | and | I. A. | “Multiclass |     | classification | of dry beans |     |     |     |     |     |     |     |
| ------------- | --- | ----- | ----------- | --- | -------------- | ------------ | --- | --- | --- | --- | --- | --- | --- |
using computer vision and machine learning techniques,” Comput. expected distance between sets which contain the point x∗
| Electron. | Agric., | vol. | 174, p. 105507, |     | 2020. [Online]. | Available: |          |          |                |     |             |              |     |
| --------- | ------- | ---- | --------------- | --- | --------------- | ---------- | -------- | -------- | -------------- | --- | ----------- | ------------ | --- |
|           |         |      |                 |     |                 |            | and sets | which do | not. Formally, |     | as in [13], | to calculate | the |
https://api.semanticscholar.org/CorpusID:219762890
|     |     |     |     |     |     |     | privacy guarantee |     | for an | individual | point | x∗, we | compute |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ------ | ---------- | ----- | ------ | ------- |
[32] A.Krizhevsky,G.Hintonetal.,“Learningmultiplelayersoffeatures
| fromtinyimages,”2009. |     |     |     |     |     |     | MI(x∗,M(X)[i]+B[i]) |     |     |     |     |     |     |
| --------------------- | --- | --- | --- | --- | --- | --- | ------------------- | --- | --- | --- | --- | --- | --- |
(M(X)[i]+B[i]∥M(X¯[i])+B[i])
| [33] C. Dwork | and | A.  | Roth, “The | algorithmic |     | foundations of | ≤E  | X∼X¯D |     |     |     |     |     |
| ------------- | --- | --- | ---------- | ----------- | --- | -------------- | --- | ----- | --- | --- | --- | --- | --- |
KL
d i f fer e n ti al p riv a c y ,” F o un d . Tr e n d s Th eo r . C o mp u t . S c i . , vo l . 9 , (cid:2)(cid:13) (cid:13)
|     |     |     |     |     |     |     | E   | (cid:13)M(X)[i]−M(X¯)[i] |     |     | 2(cid:3) |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------ | --- | --- | -------- | --- | --- |
n o . 3 – 4 , p . 2 1 1 – 40 7 , a u g 2 0 1 4 . [O n l in e] . A v a il a b l e : h t tp s: X∼X¯ (cid:13)
|                              |     |     |     |     |     |     | ≤   |     |     |     |     | .   |     |
| ---------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| //doi.org/10.1561/0400000042 |     |     |     |     |     |     |     |     | 2e  |     |     |     |     |
i
[34] H.Xiao,G.E.Suh,andS.Devadas,“FormalPrivacyProofofHeuristic
|     |     |     |     |     |     |     | where X | and X¯ are | drawn | from | D and satisfy | the | constraint |
| --- | --- | --- | --- | --- | --- | --- | ------- | ---------- | ----- | ---- | ------------- | --- | ---------- |
Encoding:ThePossibilityandImpossibilityofLearnableObfuscation,”
|     |     |     |     |     |     |     | thatx∗ ∈X | andx∗ | ∈/ X¯.Forthetightestguaranteesonnoise |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------- | ----- | ------------------------------------- | --- | --- | --- | --- |
inComputerandCommunicationsSecurityConference,October2024.
X¯
|     |     |     |     |     |     |     | for a given | X, we | choose | and | X as adjacent |     | datasets. |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ----- | ------ | --- | ------------- | --- | --------- |
[35] D.Su,J.Cao,N.Li,E.Bertino,andH.Jin,“Differentiallyprivate
k-means clustering,” in Proceedings of the Sixth ACM Conference Now, when we add independent Gaussian noises B ,
[1:d]
onDataandApplicationSecurityandPrivacy,ser.CODASPY’16. in a form N(0,e ), for i=1,2,··· ,d, to each coordinate,
|     |     |     |     |     |     |     |     | i (cid:0) |     |     | (cid:1) |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | --- | ------- | --- | --- |
NewYork,NY,USA:AssociationforComputingMachinery,2016,p. to ensure that MI X;M(X)+B is upper bounded by β,
26–37.[Online].Available:https://doi.org/10.1145/2857705.2857708
|                    |     |                |     |     |             |                 | it suffices | to select | e [1:d] | such that |     |     |     |
| ------------------ | --- | -------------- | --- | --- | ----------- | --------------- | ----------- | --------- | ------- | --------- | --- | --- | --- |
| [36] K. Chaudhuri, |     | C. Monteleoni, | and | A.  | D. Sarwate, | “Differentially |             |           |         |           |     |     |     |
p ri va t e e m p ir ic a l r i s k m i n im iz a t ion,”J.Mach.Learn.Res.,vol.12, d
(cid:88) σ i
| no . n | u ll , p. 1 0 | 6 9 – 1 1 0 9 ,j | u l 20 1 1 . |     |     |     |     |     |     | ≤β, |     |     |     |
| ------ | ------------- | ---------------- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
2e
i
[37] B.I.P.Rubinstein,P.L.Bartlett,L.Huang,andN.Taft,“Learning i=1
i n a l ar g e f u n c ti o n s p a c e : P riv a c y- p re s e r v in g m e ch an is m s f or sv m E (cid:2)(cid:13) (cid:13)M(X)[i]−M(X¯)[i] (cid:13) 2(cid:3)
l ear ni n g ,” J o u r n a l o f P r i v a cy a n d C o n fi d e n ti al ity , vo l. 4 , no . 1, Ju l. where σ i := X∼X¯ (cid:13) . This en-
2012. [Online]. Available: https://journalprivacyconfidentiality.org/ ables us to compute the anisotropic noise as in Theorem 1
index.php/jpc/article/view/612 where the optimal e is of the form
i
|     |     |     |     |     |     |     |     |     | √   |     | √   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
[38] Y.Zhang,Z.Hao,andS.Wang,“Adifferentialprivacysupportvector (cid:80)d
machineclassifierbasedondualvariableperturbation,”IEEEAccess, σ i σ j
|                            |     |     |     |     |     |     |     |     | e = | j=1 | .   |     |     |
| -------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| vol.7,pp.98238–98251,2019. |     |     |     |     |     |     |     |     | i   |     |     |     |     |
2β

Finally, we note that we take the maximum e over all subset and therefore the membership information of all data
i
individualdatapointsx∗ toprovideanindividualmembership elements! The correlation between the memberships of the
guarantee for any chosen point. data elements decreases exponentially as m increases.
| Appendix     | B.       |           |     |              |     |                |     |            |               |     |          |               |               |              |
| ------------ | -------- | --------- | --- | ------------ | --- | -------------- | --- | ---------- | ------------- | --- | -------- | ------------- | ------------- | ------------ |
|              |          |           |     |              |     |                |     | B.2. Noise | Determination |     |          | using         | True Variance |              |
| An Alternate |          | Approach  |     | to Computing |     | Noise          |     |            |               |     |          |               |               |              |
|              |          |           |     |              |     |                |     | A slightly | modified      |     | noise    | determination |               | algorithm is |
|              |          |           |     |              |     |                |     | presented  | in Algorithm  |     | 2, which | takes         | as input      | the distri-  |
| The          | approach | described |     | in Section   | 4   | uses empirical |     |            |               |     |          |               |               |              |
convergence guarantees to provide a tight variance estimate bution D; in this model, this is represented exactly by the
for M(X), rather than the high-probability guarantee of uniform distribution over the set S. We denote S[k] as the
|                   |                 |               |              |                |              |          |           | k’th element  | of               | the set      | S. All    | other terms   | are        | as before.    |
| ----------------- | --------------- | ------------- | ------------ | -------------- | ------------ | -------- | --------- | ------------- | ---------------- | ------------ | --------- | ------------- | ---------- | ------------- |
| [13] which        | requiresgreater |               | round        | complexity.    |              | In this  | section,  |               |                  |              |           |               |            |               |
| we describe       | a               | different     | approach     | to             | constructing | D,       | which     |               |                  |              |           |               |            |               |
|                   |                 |               |              |                |              |          |           | Algorithm     | 2 Anisotropic    |              | Noise     | Determination |            | of M Using    |
| has predictable   |                 | computational |              | cost           | and directly |          | satisfies |               |                  |              |           |               |            |               |
|                   |                 |               |              |                |              |          |           | True Variance |                  |              |           |               |            |               |
| the requirements  |                 | of Theorem    |              | 1 by providing |              | a method | to        |               |                  |              |           |               |            |               |
|                   |                 |               |              |                |              |          |           | Input:        | The input        | distribution |           | D represented |            | by the set of |
| compute           | the true        | variance      | of           | M(X).          |              |          |           |               |                  |              |           |               |            |               |
|                   |                 |               |              |                |              |          |           | subsets       | S, deterministic |              | mechanism | M:Xn          | →Yd,       | mutual        |
|                   |                 |               |              |                |              |          |           | information   | requirement      |              | β, d×d    | unitary       | projection | matrix        |
| B.1. Constructing |                 | a             | Distribution |                | D            |          |           |               |                  |              |           |               |            |               |
A.
|             |          |             |     |             |         |         |     | 1) m:=|S|, | G:=[0]        |     |     |     |     |     |
| ----------- | -------- | ----------- | --- | ----------- | ------- | ------- | --- | ---------- | ------------- | --- | --- | --- | --- | --- |
| First,      | consider | sampling    | X   | ···X        | ⊂X      | , which | are |            |               |     | m×d |     |     |     |
|             |          |             |     | 1 m         |         | train   |     | 2) for     | k =1,2,...,m: |     |     |     |     |     |
| independent | and      | identically |     | distributed | subsets | of      | X   |            |               |     |     |     |     |     |
train
with |X | = |X | as before for a fixed choice of m. We a) X :=S[k].
|     | i   | j   |     |     |     |     |     |     | k       |        |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | ------ | --- | --- | --- | --- |
|     |     |     |     |     |     |     |     | b)  | Compute | y =M(X |     | ).  |     |     |
then define S as the set {X 1 ···X m }; throughout, we let k k
D denote the uniform distribution over S. Constructing S 3) for each k ∈[1,...m] and i∈[1,...d], set
| to be all | possible | r|X | |     | subsets of | X     | matches | the |     |     |     |            |     |     |     |
| --------- | -------- | --- | ----- | ---------- | ----- | ------- | --- | --- | --- | --- | ---------- | --- | --- | --- |
|           |          |     | train |            | train |         |     |     |     |     | G[k][i]:=y | ·A  | .   |     |
construction of D in Section 4. As before, our noisy release k i
must be drawn from D. However, this choice of S is too 4) Compute the variance vector σ where σ is a vector
|     |     |     |     |     |     |     |     |     |     |     |     | m   |     | m   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
large for the true variance to be exactly computed. of length d and σ [i] is the variance of G .
|     |     |     |     |     |     |     |     |     |     |     | m   |     |     | i   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Varying choices of S affect the prior of our adversary. 5) Calculate the required noise in each direction i as
| That is, | if we | choose | S to be | m uniformly |     | random | subsets |     |     |           |     |           |     |     |
| -------- | ----- | ------ | ------- | ----------- | --- | ------ | ------- | --- | --- | --------- | --- | --------- | --- | --- |
|          |       |        |         |             |     |        |         |     |     | (cid:112) | d   | (cid:112) |     |     |
of size r|X train |, then a randomly chosen element x i may σ [i] (cid:80) σ [j]
|     |     |     |     |     |     |     |     |     |     | m   |     | m   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
occur in over 50% of these subsets; the number of subsets j=1
|           |          |            |         |          |              |            |      |           | e :=       |     |        |           | for i∈[1,d]. |            |
| --------- | -------- | ---------- | ------- | -------- | ------------ | ---------- | ---- | --------- | ---------- | --- | ------ | --------- | ------------ | ---------- |
| x occurs  | in would | follow     | the     | binomial | distribution |            | of m |           | i          |     |        |           |              |            |
| i         |          |            |         |          |              |            |      |           |            |     | 2β     |           |              |            |
| tosses of | a fair   | coin. This | implies | that     | certain      | membership |      |           |            |     |        |           |              |            |
|           |          |            |         |          |              | m,         |      | 6) Return | a diagonal |     | matrix | Σ , where | Σ            | [i][i]=e . |
attacks would have a prior over 50%; for large the prior B B i
| will be overwhelmingly |     |     | close | to 50%. |     |     |     |     |     |     |     |     |     |     |
| ---------------------- | --- | --- | ----- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
However, the choice of S is free; that is, our privacy In practice, we construct S using m = 1024 possible
| guaranteesholdaslongasthechosenX |      |          |     |            | isdrawnuniformly |                |     |            |         |         |          |              |         |             |
| -------------------------------- | ---- | -------- | --- | ---------- | ---------------- | -------------- | --- | ---------- | ------- | ------- | -------- | ------------ | ------- | ----------- |
|                                  |      |          |     |            | j                |                |     | subsets    | made up | of 512  | distinct | pairs        | (X i ,X | i+1 ), 1≤i≤ |
| at random                        | from | the same | set | S that the | X                | are subsampled |     |            |         |         |          |              |         |             |
|                                  |      |          |     |            | i                |                |     | 512, where | each    | pair is | disjoint | as described | above.  | This has    |
from (and the prior is appropriately calculated). For instance, a prior for individual membership of exactly 50%, provides
a simple instantiation of S for r =0.5 is as follows: empirical stability in performance (cf. Section B.3), and
1) Choose a random 0.5|X train | subset of X train that we has exponentially small correlation between data elements.
denote as X . This choice of m=1024 allows us to efficiently privatize
1
2) Choose X :=X \X . large datasets (e.g., Dry Bean with over 10,000 datapoints)
|     |     | 2   | train | 1   |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
3) S = {X ,X }, i.e., m = 2, and D corresponds to even for complex algorithms like decision trees, which was
1 2
S.
drawing randomly from intractable for the prior approach. Moreover, the variance in
For this choice of S and D, we observe that the prior for performance across different S is small. We provide further
any individual datapoint is exactly 50% since X and X resultsfortheDryBeandatasetwhenprivatizingtheRandom
|                 |     |       |               |           |       | 1              | 2       |                  |     |           |     |     |     |     |
| --------------- | --- | ----- | ------------- | --------- | ----- | -------------- | ------- | ---------------- | --- | --------- | --- | --- | --- | --- |
|                 |     |       |               |           |       |                |         | Forest algorithm |     | in Figure | 14. |     |     |     |
| are constructed |     | to be | disjoint.     | Moreover, | this  | set is         | of size |                  |     |           |     |     |     |     |
| 2 and computing |     | the   | true variance | is        | easy. | The adversary, |         |                  |     |           |     |     |     |     |
who is assumed to know D and S, has a prior probability B.3. Tradeoffs in Choosing m
| of 1 of | correctly | guessing |     | the chosen | subset, | and | in this |     |     |     |     |     |     |     |
| ------- | --------- | -------- | --- | ---------- | ------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
m
1,
case it is which is the same as guessing whether an Smallerchoicesofmshowamuchlargervarianceinthe
2
individual data element x is in the chosen subset or not. noise estimates. For example, for m=2, different random
a
Knowing the chosen subset gives away the membership instantiations of the set S can produce very different Σ
B
information of all data elements in X , regardless of the from Algorithm 2. Increasing m leads to greater stability of
train
value of m. However, for the m=2 case, knowing a single Σ B . Note that there are privacy guarantees even for m=2;
element that was chosen (or not) gives away the chosen we can bound the posterior advantage of the adversary in

Figure15.Thepriorfromthegeneralizedmembershipattack(Equation(6))
dropsbelow5%fork≥59.ThisindicatesthatlooseMIguaranteescan
bemeaningfulforharderadversarialinferencetasks.
|     |     |     |     |     |     |     |     | the calculated |      | MI. In     | this setting, | the      | adversary | knows | the     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | ---- | ---------- | ------------- | -------- | --------- | ----- | ------- |
|     |     |     |     |     |     |     |     | distribution   | that | the secret | key           | is drawn | from,     | but   | it does |
Figure14.OntheDryBeandataset,withoutregularization,weachieve71% not necessarily have a compact representation as a uniform
accuracyinthebaseline,butsignificantlossesinprivatizedutilityoverall distribution over a sufficiently small S.
MI.Withregularization,duetotheimprovedstability,wecanincreasethe Formally, the secret key is distributed uniformly at
numberoftreesused.Thisprovidesasmallincreaseinbaselineaccuracy 2n
to72%withnegligiblelossesinprivatizedutilityforallMI. random over all possible bit vectors. That is, for any
|                   |       |         |          |           |              |             |      | particular | bit i   | ∈ [1,n],   | we  | have a       | probability | of      | 0.5 on  |
| ----------------- | ----- | ------- | -------- | --------- | ------------ | ----------- | ---- | ---------- | ------- | ---------- | --- | ------------ | ----------- | ------- | ------- |
|                   |       |         |          |           |              |             |      | guessing   | the bit | correctly. | Our | inference    |             | task is | to then |
| guessing          | which | subset  | was      | used. The | prior        | and success | rate |            |         |            |     |              |             |         |         |
|                   |       |         |          |           |              |             |      | construct  | a guess | X′, which  | is  | a bit vector | of          | length  | n, such |
| of reconstruction |       | attacks | decrease | as        | m increases. |             |      |            |         |            |     |              |             |         |         |
X′
To compute the variance of noise estimates, we fix MI= that at least k indices in are classified correctly. For any
|         |          |     |         |            |     |         |         | fixed size | k, our | prior | probability | becomes |     |     |     |
| ------- | -------- | --- | ------- | ---------- | --- | ------- | ------- | ---------- | ------ | ----- | ----------- | ------- | --- | --- | --- |
| 0.5 and | consider | the | K-Means | algorithm; | we  | analyze | the l 2 |            |        |       |             |         |     |     |     |
norm of diag(Σ ), which represents the noise required to k −1(cid:18) (cid:19)(cid:18) (cid:19)n
|     |     | B   |     |     |     |     |     |     |     |     | (cid:88) | n   | 1   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- |
privatize this algorithm. At m=2, over 100 instantiations, p:=1− . (6)
k′
| we observe                 | that         | the           | norm       | of diag(Σ               | ) varies            | from       | 0.003    |                  |            |         |           |                   | 2          |             |          |
| -------------------------- | ------------ | ------------- | ---------- | ----------------------- | ------------------- | ---------- | -------- | ---------------- | ---------- | ------- | --------- | ----------------- | ---------- | ----------- | -------- |
|                            |              |               |            |                         | B                   |            |          |                  |            |         | k′=0      |                   |            |             |          |
| to 0.89.                   | In contrast, |               | at m       | = 128,                  | the norm            | of         | diag(Σ B | )                |            |         |           |                   |            |             |          |
|                            |              |               |            |                         |                     |            |          | In expectation,  |            | we      | guess     | n/2 bits          | correctly. |             | However, |
| varies from                | 0.08         | to            | 0.19.      | This indicates          |                     | that, at   | m = 2,   |                  |            |         |           |                   |            |             |          |
|                            |              |               |            |                         |                     |            |          | as k increases   |            | beyond  | n/2,      | our prior         | success    | probability |          |
| the particular             |              | instantiation |            | of S greatly            | affects             | the        | overall  |                  |            |         |           |                   |            |             |          |
|                            |              |               |            |                         |                     |            |          | drops very       | quickly,   | as      | seen in   | Figure            | 15. When   | we          | consider |
| performance                | of           | the           | privatized | algorithm.              |                     | Similarly, | when     |                  |            |         |           |                   |            |             |          |
|                            |              |               |            |                         |                     |            |          | n = 100          | and        | k = 63, | our       | prior probability |            | drops       | below    |
| we consider                | decision     |               | trees      | without                 | any regularization, |            | we       |                  |            |         |           |                   |            |             |          |
|                            |              |               |            |                         |                     |            |          | 1%, providing    | meaningful |         | posterior | bounds            |            | for large   | values   |
| observethatthenormofdiag(Σ |              |               |            | )variesfrom0.08to2.91at |                     |            |          |                  |            |         |           |                   |            |             |          |
|                            |              |               |            | B                       |                     |            |          | of MI as         | discussed  | in      | Table     | 1. Thus,          | when       | we          | consider |
| m=2.                       | At m=128,    |               | the norm   | of diag(Σ               |                     | ) varies   | between  |                  |            |         |           |                   |            |             |          |
|                            |              |               |            |                         | B                   |            |          | k non-negligibly |            | larger  | than      | n/2, this         | allows     | for         | loose MI |
0.39 and 0.98.
guaranteestostillbemeaningful.Forexample,ageneralized
Small m is computationally more efficient, but provides membership attack with n = 100, k = 70 and a mutual
| weaker             | privacy    | guarantees. |               | It may       | require   | the addition | of       |              |                  |     |            |          |              |           |          |
| ------------------ | ---------- | ----------- | ------------- | ------------ | --------- | ------------ | -------- | ------------ | ---------------- | --- | ---------- | -------- | ------------ | --------- | -------- |
|                    |            |             |               |              |           |              |          | information  | guarantee        |     | of 1,      | provides | a ≤          | 13.81%    | poste-   |
| smaller            | or greater |             | noise         | in different | trials.   | We           | defer a  |              |                  |     |            |          |              |           |          |
|                    |            |             |               |              |           |              |          | rior success | guarantee,       |     | by solving | Equation |              | (3) given | the      |
| fuller exploration |            | of          | the tradeoffs | between      |           | privacy,     | utility, |              |                  |     |            |          |              |           |          |
|                    |            |             |               |              |           |              |          | appropriate  | prior.           |     |            |          |              |           |          |
| computation        | cost       | and         | choice        | of m         | to future | work.        |          |              |                  |     |            |          |              |           |          |
|                    |            |             |               |              |           |              |          | Appendix     | D.               |     |            |          |              |           |          |
| Appendix           | C.         |             |               |              |           |              |          |              |                  |     |            |          |              |           |          |
|                    |            |             |               |              |           |              |          | Adapting     | LIRA             | to  | K-Means    |          | and SVM      |           |          |
| A generalized      |            | membership  |               | attack       |           |              |          |              |                  |     |            |          |              |           |          |
|                    |            |             |               |              |           |              |          | The          | Likelihood-Ratio |     | Attack     | by       | [7] exploits |           | the idea |
We observe that the prior guarantee of 0.5 is specific to that when a model is trained on a particular point x, its
membershipattackswherewetrytoidentifythemembership “confidence”onclassifyingxintoaparticularclassorcluster
of a single, specific datapoint x. Here, we consider a willbehigherthanonapointitisnottrainedon.Theoriginal
generalization of the membership attack where we aim work by [7] exploits this by framing a membership attack as
|     |     |     |     |     |     | any |     |     |     |     | ϕ(M(X | ),x) |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | ---- | --- | --- | --- |
to correctly identify the membership of subset of a hypothesis test. Let j denote the confidence
points, where the subset has size at least k. Consider an score of M(X ) on x. In this work, they sample many
j
attack that attempts to identify some fraction of bits of subsetsX andapproximatethedistributionofϕ(M(X ),x),
|     |     |     |     |     |     |     |     |     | i   |     |     |     |     |     | i   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
a secret key. In this setting, a secret key consisting of n whenx∈/ X .Fortheattack,theythenconsideravaryingset
i
bits is generated from a uniform random distribution and an ofthresholdstwheretheattackconcludesx∈X ifandonly
j
adversarialtaskfocusesonidentifyingatleastkbitscorrectly. if ϕ(M(X j ),x) ≥ t. Each threshold t has corresponding
In this section, we primarily consider the prior for this true and false positive rates, and we simply consider the
attack. In practice, the adversary may be able to view some maximum accuracy over all thresholds t, which represents
side channel, e.g., a timing, power, or cache side channel, the maximum posterior advantage achievable by LIRA.
and this additional exposure via some mechanism M will Unlike Carlini’s work, we do not directly produce con-
provide some posterior advantage that will correspond to fidence values from our algorithms. We instead translate

| our output  | vectors    | to     | approximate | confidence |            | values. For |
| ----------- | ---------- | ------ | ----------- | ---------- | ---------- | ----------- |
| the K-Means | algorithm, |        | we compute  | a          | confidence | metric      |
| ϕ(M(X       | j ),x)     | := 1 − | d(x), where | d(x)       | represents | the         |
normalizeddistancetotheclustertowhichxisassigned.For
| SVM, we   | use Platt | calibration | to   | translate    | the distance | from  |
| --------- | --------- | ----------- | ---- | ------------ | ------------ | ----- |
| the point | to the    | hyperplane  | into | a confidence | metric       | [43], |
i.e.,
1
|     | ϕ(M(X | j ),x,i)= |     |     |     | ,   |
| --- | ----- | --------- | --- | --- | --- | --- |
1+exp(−d(x,i))
| where d(x,i)   |      | represents | the              | distance | between          | x and    |
| -------------- | ---- | ---------- | ---------------- | -------- | ---------------- | -------- |
| the hyperplane |      | for        | class i.         | Then,    | ϕ(M(X            | j ),x) = |
| max ϕ(M(X      |      | ),x,i).    | We observe       | that     | the distribution | of       |
| i              | j    |            |                  |          |                  |          |
| ϕ(M(X          | ),x) | is not     | always Gaussian; |          | we thus          | directly |
j
| approximate | its | CDF through | 1,000 | trials. |     |     |
| ----------- | --- | ----------- | ----- | ------- | --- | --- |
