---
publish: true
---

| IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY |     |     |            |          |     |           |     |                 |     |     |              |     |        | 1   |
| ------------------------------------------------- | --- | --- | ---------- | -------- | --- | --------- | --- | --------------- | --- | --- | ------------ | --- | ------ | --- |
| Efficient                                         |     |     | Generation |          |     |           | of  | Targeted        |     | and | Transferable |     |        |     |
| Adversarial                                       |     |     |            | Examples |     |           | for | Vision-Language |     |     |              |     | Models |     |
|                                                   |     |     |            |          | Via | Diffusion |     | Models          |     |     |              |     |        |     |
Qi Guo, Shanmin Pang†, Member, IEEE, Xiaojun Jia†, Yang Liu, Senior Member, IEEE, Qing Guo, Senior
|     |     |     |     |     |     |     | Member, | IEEE |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------- | ---- | --- | --- | --- | --- | --- | --- |
4202 ceD 51  ]VC.sc[  4v53301.4042:viXra to generate targeted responses without knowing the models’
| Abstract—Adversarial |     |     | attacks, | particularly |     | targeted | transfer- |     |     |     |     |     |     |     |
| -------------------- | --- | --- | -------- | ------------ | --- | -------- | --------- | --- | --- | --- | --- | --- | --- | --- |
basedattacks,canbeusedtoassesstheadversarialrobustnessof internal information, resulting in greater harm [22], [23]. Fur-
large visual-language models (VLMs), allowing for a more thor- thermore, targeted attacks on black-box models present more
| ough examination |     | of potential |     | security | flaws | before | deployment. |     |     |     |     |     |     |     |
| ---------------- | --- | ------------ | --- | -------- | ----- | ------ | ----------- | --- | --- | --- | --- | --- | --- | --- |
challengesthanuntargetedattacks[24],[25].Asaresult,when
| However, | previous | transfer-based |     | adversarial |     | attacks | incur high |           |                 |            |     |          |     |                |
| -------- | -------- | -------------- | --- | ----------- | --- | ------- | ---------- | --------- | --------------- | ---------- | --- | -------- | --- | -------------- |
|          |          |                |     |             |     |         |            | assessing | the adversarial | robustness |     | of VLMs, |     | it is critical |
costsduetohighiterationcountsandcomplexmethodstructure.
|     |     |     |     |     |     |     |     | to consider | more | threatening | and | challenging | black-box | and |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ---- | ----------- | --- | ----------- | --------- | --- |
Furthermore,duetotheunnaturalnessofadversarialsemantics,
the generated adversarial examples have low transferability. targeted attacks [16]. AttackVLM [16] is the first work to
| These issues | limit | the     | utility | of existing | methods    |     | for assessing |         |                 |            |     |      |         |           |
| ------------ | ----- | ------- | ------- | ----------- | ---------- | --- | ------------- | ------- | --------------- | ---------- | --- | ---- | ------- | --------- |
|              |       |         |         |             |            |     |               | explore | the adversarial | robustness | of  | VLMs | in both | black-box |
| robustness.  | To    | address | these   | issues,     | we propose |     | AdvDiffVLM,   |         |                 |            |     |      |         |           |
which uses diffusion models to generate natural, unrestricted and targeted scenarios using query attacks with transfer-based
and targeted adversarial examples via score matching. Specifi- priors. However, due to the large number of queries required
cally,AdvDiffVLMusesAdaptiveEnsembleGradientEstimation and the complex method structure, this method is inefficient,
| (AEGE) | to modify | the | score | during | the diffusion |     | model’s re- |               |     |          |                 |     |       |            |
| ------ | --------- | --- | ----- | ------ | ------------- | --- | ----------- | ------------- | --- | -------- | --------------- | --- | ----- | ---------- |
|        |           |     |       |        |               |     |             | which reduces | its | validity | and suitability |     | for a | comprehen- |
versegenerationprocess,ensuringthattheproducedadversarial
|          |       |                  |             |                 |          |            |             | sive assessment | of          | the limitations |           | of VLMs. | Another  | attack    |
| -------- | ----- | ---------------- | ----------- | --------------- | -------- | ---------- | ----------- | --------------- | ----------- | --------------- | --------- | -------- | -------- | --------- |
| examples | have  | natural          | adversarial |                 | targeted | semantics, | which       |                 |             |                 |           |          |          |           |
|          |       |                  |             |                 |          |            |             | method          | that can be | used in         | black-box | and      | targeted | scenarios |
| improves | their | transferability. |             | Simultaneously, |          | to         | improve the |                 |             |                 |           |          |          |           |
quality of adversarial examples, we use the GradCAM-guided is the transfer-based attack [26]–[29]. However, this type of
Mask Generation (GCMG) to disperse adversarial semantics attack method is slow to generate adversarial examples due to
| throughout | the | image | rather | than | concentrating |     | them in | a           |           |              |     |             |              |     |
| ---------- | --- | ----- | ------ | ---- | ------------- | --- | ------- | ----------- | --------- | ------------ | --- | ----------- | ------------ | --- |
|            |     |       |        |      |               |     |         | its complex | structure | and numerous |     | iterations. | Furthermore, |     |
singlearea.Finally,AdvDiffVLMembedsmoretargetsemantics
|     |     |     |     |     |     |     |     | because | it adds unnatural | adversarial |     | semantics, |     | the transfer- |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ----------------- | ----------- | --- | ---------- | --- | ------------- |
intoadversarialexamplesaftermultipleiterations.Experimental
abilityofadversarialexamplesispoor.Unrestrictedadversarial
| results show | that | our | method | generates | adversarial |     | examples |     |     |     |     |     |     |     |
| ------------ | ---- | --- | ------ | --------- | ----------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
5x to 10x faster than state-of-the-art (SOTA) transfer-based examples [30]–[33] can incorporate more natural adversarial
adversarial attacks while maintaining higher quality adversarial targeted semantics into the image, thereby improving the
| examples.    | Furthermore, |                  | compared    |               | to previous | transfer-based |             |               |             |                 |              |             |           |          |
| ------------ | ------------ | ---------------- | ----------- | ------------- | ----------- | -------------- | ----------- | ------------- | ----------- | --------------- | ------------ | ----------- | --------- | -------- |
|              |              |                  |             |               |             |                |             | image quality | and         | transferability | of           | adversarial | examples. | For      |
| adversarial  | attacks,     | the              | adversarial | examples      |             | generated      | by our      |               |             |                 |              |             |           |          |
|              |              |                  |             |               |             |                |             | example,      | AdvDiffuser | [32]            | incorporates | PGD         | [34]      | into the |
| method       | have better  | transferability. |             |               | Notably,    | AdvDiffVLM     | can         |               |             |                 |              |             |           |          |
|              |              |                  |             |               |             |                |             | reverse       | process of  | the diffusion   | model        | to          | generate  | targeted |
| successfully | attack       | a variety        |             | of commercial |             | VLMs           | in a black- |               |             |                 |              |             |           |          |
box environment, including GPT-4V. The code is available at adversarial examples with better transferability against classi-
https://github.com/gq-max/AdvDiffVLM fication models. However, applying PGD to the latent image
Index Terms—Adversarial Attack, Visual Language Models, in the reverse process is not suitable for the more difficult
Diffusion Models, Score Matching. task of attacking VLMs. At the same time, performing PGD
|       |      |      |              |       |         |     |               | on each       | step of the | reverse              | process     | incurs   | high costs.  |          |
| ----- | ---- | ---- | ------------ | ----- | ------- | --- | ------------- | ------------- | ----------- | -------------------- | ----------- | -------- | ------------ | -------- |
|       |      |      |              |       |         |     |               | In this       | paper, we   | propose              | AdvDiffVLM, |          | an efficient | frame-   |
|       |      | I.   | INTRODUCTION |       |         |     |               |               |             |                      |             |          |              |          |
|       |      |      |              |       |         |     |               | work that     | leverages   | diffusion            | models      | to       | generate     | natural, |
| LARGE | VLMs | have | shown        | great | success |     | in tasks like |               |             |                      |             |          |              |          |
|       |      |      |              |       |         |     |               | unrestricted, | and         | targeted adversarial |             | examples | through      | score    |
image-to-text [1]–[3] and text-to-image generation [4], matching. Score matching, initially proposed by Hyva¨rinen
[5]. Particularly in image-to-text generation, users can use et al. [38], is a computationally simple probability density
imagestogenerateexecutablecommandsforrobotcontrol[6], estimation method. It was later introduced into the field of
which has potential applications in autonomous driving sys- imagegenerationbySongetal.[39],demonstratingitsability
tems [7], [8], visual assistance systems [9], and content mod- to guide image generation toward specific target semantics by
eration systems [10]. However, VLMs are highly susceptible modifying the score function. Furthermore, Song et al. [40]
to adversarial attacks [11], [12], which can result in life and combinedscorematchingwithadiffusionmodel,significantly
property safety issues [13], [14]. As a result, it is critical to enhancing image quality. Inspired by these developments,
evaluate the adversarial robustness [15]–[18] of these VLMs we investigate the use of score matching to effectively and
before deployment. efficiently attack VLMs, aiming to embed richer adversarial
The early research on assessing the adversarial robustness target semantics compared to existing methods like AdvD-
of VLMs concentrated on white-box and untargeted scenar- iffuser [32]. Specifically, we derive a score generation the-
ios[19]–[21].Black-boxandtargetedattackscancausemodels ory tailored for VLM attacks and propose the AEGE based

IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY 2
Fig.1. Comparisonofdifferenttransfer-basedattacksandourmethodonVLMs.(a)Comparisonofattackperformance.WeselectBLIP2[2]andImg2LLM[35]
astherepresentationmodelsofVLMs.Weselectexistingtransfer-basedattacksinconjunctionwithAttackVLM[16]ascomparisonmethods,includingEns[36],
SVRE[27],CWA[26],SSA[37]andSIA[28].WereporttheCLIPtar score,whichisthesimilaritybetweentheresponsegeneratedbytheinputimages.(b)
Comparisonofimagequality.Weenlargethelocalareaoftheadversarialexamplestoenhancevisualeffects.Itisevidentthatadversarialexamplesgenerated
bytransfer-basedattacksexhibitnotablenoise.Ourmethodhasbettervisualeffects.Magnifyimagesforimprovedcontrast.
on this theoretical foundation. Furthermore, to improve the GCMG.Incontrasttotraditionalapplications,ourmethod
naturalness of the outputs, we propose the GMGC mod- allows modifications to be made across the entire image
ule, which effectively distributes adversarial target semantics while minimizing alterations to key areas, thus balancing
acrosstheexamples.Thispreventstheconcentrationofadver- attack capability with image quality.
sarial features in specific regions, thereby improving overall • Extensive experiments show that our method generates
image quality. In addition, we embed more target semantics targetedadversarialexamplesfasterthanSOTAadversar-
into adversarial examples through multiple iterations, further ial attack methods in attacking VLMs, and the generated
enhancing the visual quality of the generated outputs. As adversarial examples exhibit better transferability. In ad-
demonstrated in Figure 1, AdvDiffVLM outperforms existing dition,ourresearchidentifiesvulnerabilitiesinbothopen-
attack methods by generating targeted adversarial examples source and commercial VLMs, offering insights toward
moreefficientlywhileachievingsuperiortransferability.More- developing more robust and trustworthy VLMs.
over, the generated adversarial examples exhibit enhanced
naturalness,establishingAdvDiffVLMasamoreeffectivetool
for evaluating the adversarial robustness of VLMs. II. RELATEDWORK
We summarize our contributions as follows:
A. Visual-Language Models (VLMs)
• We explore existing adversarial attack techniques against
VLMs and conduct research on more realistic and chal- Large language models (LLMs) [41]–[43] have demon-
lenging scenarios, specifically focusing on targeted and strated great success in a variety of language-related tasks.
transferable attacks. Furthermore, we propose the AdvD- The knowledge contained within these powerful LLMs has
iffVLM framework to efficiently generate targeted and aided the development of VLMs. There are several strategies
transferable adversarial examples for VLMs. and models for bridging the gap between text and visual
• We present a score calculation method that embeds modalities [44], [45]. Some studies [2], [46] extract visual
adversarial target semantics into the diffusion model, informationfromlearnedqueriesandcombineitwithLLMsto
supported by theoretical analysis. Additionally, we pro- enhance image-based text generation. Models like LLaVA [3]
pose an adaptive ensemble method to better estimate and MiniGPT-4 [47] learn simple projection layers to align
the score. Building on this theoretical foundation and visual encoder features with LLM text embeddings. Some
adaptive ensemble method, we propose the AEGE that works [5] train VLMs from scratch, which promotes better
combines the generated gradient with score matching, alignment of visual and textual modalities. In this paper, we
embedding adversarial target semantics naturally in the focus on the adversarial robustness of these VLMs, with the
inverse generation process of the diffusion model. goal of discovering security vulnerabilities and encouraging
• WeapplyaninnovativeuseofGradCAMandproposethe the development of more robust and trustworthy VLMs.

IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY 3
B. Adversarial Attacks in VLMs and diversity. AdvDiffuser [32] incorporates the PGD [34]
methodintothediffusionmodel’sreverseprocess,resultingin
Adversarialattacks areclassified aswhite-box orblack-box
high-quality adversarial examples without restrictions. In this
attacks based on adversary knowledge, as well as targeted or
study, we explore using the diffusion model for generating
untargeted attacks based on adversary objectives [48]–[50].
unrestricted adversarial examples, focusing on modifying the
Studies have investigated the robustness of VLMs, focusing
score in the diffusion model’s reverse process rather than
on adversarial challenges in visual question answering [51]
adding noise to the latent image. We discuss the differences
and image captioning [19]. However, most studies focus on
between our method and AdvDiffuser in Section IV-D.
traditionalCNN-RNN-basedmodels,whichmakeassumptions
about white-box access or untargeted goals, limiting their ap-
plicability in real-world scenarios. Recently, AttackVLM [16]
III. PRELIMINARIES
implemented both transfer-based and query-based attacks on
large open-source VLMs with black-box access and targeted A. Diffusion Models
goals. Nonetheless, this method is time-consuming due to its
In this work, we use diffusion models [4], [55], [56]
reliance on numerous VLM queries. In addition, [52] studied
to generate unrestricted and targeted adversarial examples.
the adversarial robustness of VLMs using ensemble transfer-
In a nutshell, diffusion models learn a denoising process
based attacks, assuming untargeted goals. In this paper, we
from x ∼ N (x ;0,I) to recover the data x ∼ q(x )
investigatetheadversarialrobustnessofVLMsagainsttargeted T T 0 0
with a Markov chain and mainly include two processes:
transfer-basedattacks.Initially,weevaluateVLM’srobustness
forward process and reverse process. Forward process de-
against current SOTA transfer-based attacks in conjunction
fines a fixed Markov chain. Noise is gradually added to
with AttackVLM. We then examine the limitations of current
the image x over T time steps, producing a series of
methods and implement targeted improvements, culminating 0
noisy images {x ,x ,··· ,x }. Specifically, noise is added
in the proposal of AdvDiffVLM. 1√2 T√
by q(x |x ) := α¯ x +ϵ 1−α¯ ,ϵ ∼ N(0,1), where
Our method is most closely related to AttackVLM, as both t 0 t 0 1 t 1
α := 1−β , α :=
(cid:81)t
α and β is a fixed variance to
aim to conduct adversarial attacks on VLMs. However, there t t t s=1 s t
control the step sizes of the noise. The purpose of the reverse
aretwonotabledifferences.First,whileAttackVLMgenerates
process is to gradually denoise from x to obtain a series of
adversarial examples by estimating gradients through black- T
{x˜ ,x˜ ,··· ,x˜ },andfinallyrestorex .Itlearnsthede-
box model outputs, our approach leverages the transferability T−1 T−2 1 0
noisingprocessthroughadenoisingmodelε ,andthetraining
of adversarial examples to effectively attack multiple VLMs. θ
objective is L :=E ∥ε (x ,t)−ϵ ∥2.
This makes our method more versatile for attacking diverse simple t∼[1,T],ϵ1∼N(0,I) θ t 1
VLMs. Furthermore, AttackVLM relies on extensive black-
box queries to estimate gradients, making its generation
B. Problem Settings
process time-intensive. In contrast, our method utilizes the
diffusion model’s generation process, enabling faster creation Then we give the problem setting of this paper. We denote
of adversarial examples. the victim VLM model as f , and aim to induce f to output
ξ ξ
Second, in terms of methodology, AttackVLM combines the target response. This can be formalized as
gradient-basedattackswithblack-boxquerytechniques,using
max CS(g (f (x ;c )),g (c ))
the gradient-based component to initialize black-box queries ψ ξ adv in ψ tar
(1)
effectively. In contrast, our approach adopts an unconstrained s.t. D(x,x )≤ϵ
adv
adversarialexamplegenerationframeworkgroundedingener-
ativemodels.Byintegratingadaptivegradientestimationwith where x∈R3×H×W represents the original image, x adv and
scorematching,weembedtheadversarialexamplegeneration c tar respectively refer to adversarial example and adversarial
process directly within the diffusion model’s workflow. To targettext,andg ψ (·)denotestheCLIPtextencoder.Moreover,
further enhance the quality of adversarial examples, we incor- D(x,x adv ) ≤ ϵ places a bound on a distance metric, and
porate a GradCAM-guided masking technique, which refines CS(·,·) refers to the cosine similarity metric. Finally, c in
the generated outputs. denotes the input text.
Since f is a black-box model, we generate adversarial
ξ
examples on the surrogate model ϕ and transfer them to f .
C. Unrestricted Adversarial Examples ψ ξ
In addition, inspired by [16], matching image-image features
Researchers are increasingly interested in unrestricted ad-
can lead to better results, we define the problem as,
versarial examples, as the l norm distance fails to capture
p
human perception [30]–[33], [53], [54]. Some approaches use max CS(ϕ (x ),ϕ (x ))
ψ adv ψ tar
(2)
generativemethodstocreateunrestrictedadversarialexamples. s.t. D(x,x )≤ϵ
adv
Forexample,[30]and[31]modifythelatentrepresentationof
GANstoproduceunrestrictedadversarialexamples.However, where x represents the target image generated by c .
tar tar
due to the limited interpretability of GANs, the generated We use stable diffusion [4] to implement the text-to-image
adversarialexamplesareofpoorquality.Diffusionmodels[55] generation.ϕ referstoCLIPimageencoder.Ourstudyisthe
ψ
are SOTA generative models based on likelihood and theoret- most realistic and challenging attack scenarios, i.e., targeted
ical foundations, sampling data distribution with high fidelity and transfer scenarios.

| IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY |     |     |     |     |     |     |     |     |     |     |     |     |     | 4   |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
bution.ByofferingscoreguidancetowardssolvingEq.2,faster
|     |     |     |     |     |     |     | convergence | is       | expected.      | Therefore     |                  | score    | information | can be       |
| --- | --- | --- | --- | --- | --- | --- | ----------- | -------- | -------------- | ------------- | ---------------- | -------- | ----------- | ------------ |
|     |     |     |     |     |     |     | considered  | in       | the design     | of            | new              | improved | attack      | method.      |
|     |     |     |     |     |     |     | Second,     | existing | transfer-based |               | attacks          |          | introduce   | unnatural    |
|     |     |     |     |     |     |     | adversarial | noises   | with           | limited       | transferability. |          |             | Unrestricted |
|     |     |     |     |     |     |     | adversarial | examples |                | can introduce |                  | more     | natural     | adversarial  |
targetedsemantics,increasingtransferability.Theseimplythat
|     |     |     |     |     |     |     | new transfer-based |          | targeted  |             | attacks    | can consider |                | unrestricted |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | -------- | --------- | ----------- | ---------- | ------------ | -------------- | ------------ |
|     |     |     |     |     |     |     | adversarial        | attacks. |           |             |            |              |                |              |
|     |     |     |     |     |     |     |                    |          | IV.       | METHODOLOGY |            |              |                |              |
|     |     |     |     |     |     |     | The                | main     | framework | of          | AdvDiffVLM |              | is illustrated | in           |
Fig. 2. The CLIPimg score varies with the step sizes. Here, CLIPimg is Figure3.First,weinputtheoriginalimage,x,followedbythe
| the similarity | between | the adversarial | examples | and | the adversarial | target |         |         |      |       |         |           |         |        |
| -------------- | ------- | --------------- | -------- | --- | --------------- | ------ | ------- | ------- | ---- | ----- | ------- | --------- | ------- | ------ |
|                |         |                 |          |     |                 |        | forward | process | x t∗ | ∼ q(x | t∗ |x ) | to obtain | a noisy | image, |
images, which is calculated by the visual encoder of CLIP ViT-B/32. We 0
|                                                          |     |     |     |     |     |     | x . Subsequently, |     | we  | apply | the reverse | denoising |     | process. At |
| -------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ----------------- | --- | --- | ----- | ----------- | --------- | --- | ----------- |
| chooseSSA[37]astherepresentativeoftransfer-basedattacks. |     |     |     |     |     |     | t∗                |     |     |       |             |           |     |             |
eachstep,wefirstobtainthemaskmfromtheGCMGmodule.
C. Rethinking Transfer-based Attacks This mask is then used to fuse the original noisy image, x ,
t
|                |     |         |                 |     |       |               | withtheadversarialnoisyimage,x˜ |     |     |     |     | .Next,weapplytheAEGE |     |     |
| -------------- | --- | ------- | --------------- | --- | ----- | ------------- | ------------------------------- | --- | --- | --- | --- | -------------------- | --- | --- |
| Transfer-based |     | attacks | can effectively |     | solve | Eq.2. In this |                                 |     |     |     | t   |                      |     |     |
context, we assess the robustness of VLMs against current module to obtain gradient information, which we then use to
SOTA transfer-based attacks, in conjunction with Attack- calculate the score. Finally, we derive the next step of the
VLM. Specifically, we consider ensemble methods including noisy image based on the score matching method.
Ens [36], SVRE [27] and CWA [26], data augmentation In the following, we begin by presenting the motivation
|         |           |     |              |       |                  |     | behind our | approach |     | and a theoretical |     | analysis | of  | our method. |
| ------- | --------- | --- | ------------ | ----- | ---------------- | --- | ---------- | -------- | --- | ----------------- | --- | -------- | --- | ----------- |
| methods | including | SSA | [37] and SIA | [28], | and combinations |     |            |          |     |                   |     |          |     |             |
of both1. We primarily employ the simple ensemble version This is followed by a comprehensive explanation of the
of data augmentation attacks, as relying on a single surrogate proposed AdvDiffVLM framework. Lastly, we highlight the
|             |     |            |              |     |                |     | key distinctions |     | between | our | method | and | AdvDiffuser, | em- |
| ----------- | --- | ---------- | ------------ | --- | -------------- | --- | ---------------- | --- | ------- | --- | ------ | --- | ------------ | --- |
| model tends | to  | yield poor | performance. | For | hyperparameter |     |                  |     |         |     |        |     |              |     |
settings, in all attacks, the value range of adversarial example phasizing their unique features and contributions.
ϵ=16/255
| pixels is | [0,1].                      | We set | the perturbation | budget | as            |     |               |     |                 |     |          |     |     |     |
| --------- | --------------------------- | ------ | ---------------- | ------ | ------------- | --- | ------------- | --- | --------------- | --- | -------- | --- | --- | --- |
|           |                             |        |                  |        |               |     | A. Motivation |     | and Theoretical |     | Analysis |     |     |     |
| undertheℓ | norm.ThenumberofiterationsN |        |                  |        | forallattacks |     |               |     |                 |     |          |     |     |     |
|           | ∞                           |        |                  |        | I             |     |               |     |                 |     |          |     |     |     |
is set to 300. In addition, we use the MI-FGSM [36] method With the growing deployment of VLMs in critical applica-
tionssuchasautonomousdrivingandcontentmoderation,en-
| and set | µ = | 1. Furthermore, | for | SVRE, | internal | step size |     |     |     |     |     |     |     |     |
| ------- | --- | --------------- | --- | ----- | -------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
β = 16/255/10 and internal decay factor µ = 1. For suring their robustness against adversarial attacks has become
| inter |     |     |     |     |     | 2   |     |     |     |     |     |     |     |     |
| ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
CWA, CSE step size β = 16/255/15 and inner gradient essentialformaintainingsystemsecurityandreliability.While
cse
|             |     |        |                 |        |     |            | existing | approaches | have | made | notable | progress |     | in evaluating |
| ----------- | --- | ------ | --------------- | ------ | --- | ---------- | -------- | ---------- | ---- | ---- | ------- | -------- | --- | ------------- |
| ascent step | r   | = 250. | For SSA, tuning | factor | ρ   | = 0.5, the |          |            |      |      |         |          |     |               |
number of spectrum transformations N = 20 and standard VLM robustness, they still face fundamental limitations in
t
|           |       |         |          |            |     |              | terms of | efficiency |     | and effectiveness. |     | High | computational |     |
| --------- | ----- | ------- | -------- | ---------- | --- | ------------ | -------- | ---------- | --- | ------------------ | --- | ---- | ------------- | --- |
| deviation | σ s = | 16/255. | For SSA, | the number |     | of the block |          |            |     |                    |     |      |               |     |
s = 3 and the number of image for gradient calculation overhead and limited transferability hinder the ability to
| N =20. |     |     |     |     |     |     | comprehensivelyassessrobustnessacrossdiverseVLMs.This |     |     |     |     |     |     |     |
| ------ | --- | --- | --- | --- | --- | --- | ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
grad
|              |     |          |                |         |     |          | challenge | motivates |     | our work | to develop |     | an efficient, | high- |
| ------------ | --- | -------- | -------------- | ------- | --- | -------- | --------- | --------- | --- | -------- | ---------- | --- | ------------- | ----- |
| The outcomes |     | of these | transfer-based | attacks | on  | VLMs are |           |           |     |          |            |     |               |       |
depicted in Figure 1. As illustrated, current transfer-based quality, and transferable method for generating adversarial
|         |                 |     |         |                  |     |         | examples, | thereby | facilitating |     | a more | effective | evaluation | of  |
| ------- | --------------- | --- | ------- | ---------------- | --- | ------- | --------- | ------- | ------------ | --- | ------ | --------- | ---------- | --- |
| attacks | face challenges |     | such as | slow adversarial |     | example |           |         |              |     |        |           |            |     |
generation, noticeable noise within these examples, and lim- VLM robustness. We achieve this by leveraging insights from
ited transferability. The limitations of existing transfer-based diffusion models and score matching techniques.
Specifically,wefocusonmodelingadversarialattacksfrom
attacksonVLMsareanalyzedasfollows:First,existingSOTA
transfer-based attacks only access the original image during a generative perspective, considering how to utilize the data
distribution(score)ofthegenerativemodeltoproducenatural,
| the optimization |            | of Eq.2. | Consequently,     |     | they employ    | small |              |     |          |             |     |           |     |               |
| ---------------- | ---------- | -------- | ----------------- | --- | -------------- | ----- | ------------ | --- | -------- | ----------- | --- | --------- | --- | ------------- |
|                  |            |          |                   |     |                |       | unrestricted | and | targeted | adversarial |     | examples. |     | Additionally, |
| steps and        | strategies | like     | data augmentation |     | to tentatively | ap-   |              |     |          |             |     |           |     |               |
proach the optimal solution, necessitating numerous iterations as indicated in [57], learning to model the score function is
|     |     |     |     |     |     |     | equivalent | to modeling |     | the | negative | of the | noise, | suggesting |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ----------- | --- | --- | -------- | ------ | ------ | ---------- |
andresultinginhighattackcosts.AsshowninFigure2,using
a larger step size results in pronounced fluctuations during that score matching and denoising are equivalent processes.
the optimization process. This issue may be mitigated by Thus, our method derives from integrating diffusion models
|            |        |       |                   |     |          |              | and score  | matching,     |     | positioning   | it  | as a         | novel approach | for          |
| ---------- | ------ | ----- | ----------------- | --- | -------- | ------------ | ---------- | ------------- | --- | ------------- | --- | ------------ | -------------- | ------------ |
| leveraging | score, | which | provides insights |     | into the | data distri- |            |               |     |               |     |              |                |              |
|            |        |       |                   |     |          |              | generating | high-quality, |     | unrestricted, |     | transferable |                | and targeted |
1https://github.com/xiaosen-wang/SIT for SIA and https://github.com/ adversarial examples.
thu-ml/Attack-Bard for others. we adapt the targeted attacks to untargeted Formally, we want to obtain distribution meeting the con-
| attacks to | better | align with | our scenario. | Additionally, | to  | ensure a fair |             |     |             |         |     |            |          |        |
| ---------- | ------ | ---------- | ------------- | ------------- | --- | ------------- | ----------- | --- | ----------- | ------- | --- | ---------- | -------- | ------ |
|            |        |            |               |               |     |               | dition that | the | adversarial | example |     | has target | semantic | infor- |
comparisonwithourmodel,wemodifythesurrogatemodelsbyensembling
various CLIP visual encoders, including ResNet-50, ResNet-101, ViT-B/16, mation during the reverse generation process
| and ViT-B/32. | Finally, | we modify | the loss | function | to cosine | similarity loss |     |     |     |     |     |     |     |     |
| ------------- | -------- | --------- | -------- | -------- | --------- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
tomakeitconsistentwithAttackVLM. p(x t−1 |x t ,f ξ (x adv ;c in )=c tar ) (3)

IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY 5
Fig.3. ThemainframeworkoftheAdvDiffVLMforefficientlygeneratingtransferableunrestrictedadversarialexamples.AdvDiffVLMmainlyincludestwo
components:AEGEandGCMG.DetailsarerespectivelydescribedinSecs.IV-BandIV-C.PleaserefertoSectionIVforspecificsymbolmeanings.
where x represents the latent image of the diffusion model. B. Adaptive Ensemble Gradient Estimation (AEGE)
t
Next,westartfromtheperspectiveofscorematching[40]and Since f is a black-box model and cannot obtain gra-
ξ
consider the score ∇logp(x |x ,c ) of this distribution,
t−1 t tar dient information, we use surrogate model to estimate
w
th
h
e
e
o
r
r
e
em
∇
,
is the abbreviation for ∇ xt . According to Bayes ∇logp
fξ
(c
tar
|x
t
). As a scalable method for learning joint
representationsbetweentextandimages,CLIP[58]canlever-
age pre-trained CLIP models to establish a bridge between
(cid:16) (cid:17)
∇logp(x |x ,c )=∇log p(ctar|xt−1,xt)·p(xt−1|xt) images and text. Therefore we use the CLIP model as the
t−1 t tar p(ctar|xt)
=∇logp(c |x ,x )+∇logp(x |x ) surrogate model to estimate the gradient.
tar t−1 t t−1 t
−∇logp(c |x ) Specifically, we first add noise to the original image x by
tar t
=∇logp(c tar |x t−1 )+∇logp(x t |x t−1 ,c tar ) t∗ steps through the forward process q(x t∗ |x 0 ) to obtain x t∗ ,
−∇logp(x t |x t−1 )+∇logp(x t−1 |x t )−∇logp(c tar |x t )where x 0 = x. Then, at each step of reverse process, we
=∇logp(x |x ,c )−∇logp(x |x ) change score:
t t−1 tar t t−1
+∇logp(x t−1 |x t )−∇logp(c tar |x t )
(4)
score=−(√
1−
1
α¯t
ε
θ
(x˜
t
)+s∇
x˜t
(CS(ϕ
ψ
(x˜
t
),ϕ
ψ
(x
tar
))))
(6)
p(x |x ,c ) and p(x |x ) respectively denote the
t t−1 tar t t−1 where s is the adversarial gradient scale used to control the
add noise process with target text and the add noise process
degreeofscorechangeandx˜ isthelatentimageinthereverse
devoid of target semantics. From an intuitive standpoint, t
process.
whethertargettextispresentornot,theforwardnoiseaddition
We find that gradient estimation using only a single surro-
process follows a Gaussian distribution, and the added noise
gate model is inaccurate. Therefore, we consider using a set
remains consistent, indicating that the gradient solely depends of surrogate models (cid:8) ϕi (cid:9)Nm to better estimate the gradient.
on x t . The difference between x t without target text and Specifically, we make th Ψ e f i o = l 1 lowing improvements to Eq. 6:
x with target text is minimal, as constraints are employed
t
t
t
o
he
en
o
s
r
u
ig
re
in
m
al
in
i
i
m
m
a
a
g
l
e
v
.
a
T
ri
h
a
e
ti
r
o
e
n
fo
o
re
f
,
th
∇
e
l
a
o
d
g
v
p
er
(
s
x
a
t
ri
|
al
x t
e
−
x
1
a
,
m
c
p
ta
le
r )
fr
a
o
n
m
d
score=−(√ εθ
1
(
−
x˜
α
t
¯
)
t
+s∇
x˜t
(w
i
(cid:80)N
i=
m
1
CS(ϕi
ψ
(x˜
t
),ϕi
ψ
(x
tar
)))
(
)
7)
∇logp(x t |x t−1 )areapproximatelyequal.Sothefinalscore where w = (w 1 ,w 2 ,··· ,w Nm ) represents the weight of
is ∇logp(x t−1 |x t )−∇logp(c tar |x t ). cosine loss of different models.
Because score matching and denoising are equivalent pro- Since different images have different sensitivities to sur-
cesses, that is, ∇logp(x
t
) = −√
1−
1
α¯t
ε
θ
(x
t
). Therefore we rogate models, only using simple ensemble cannot obtain
can get score (∇logp(x |x ,c )), optimalsolution. Inspiredby [59],we proposea newadaptive
t−1 t tar
ensemblemethod,andobtainwinEq.7inthefollowingway:
score=−(√ ε 1 θ ( − x t α¯ ) +∇logp fξ (c tar |x t )) (5) w (t)= (cid:80)N j= m 1 exp(τL j (t+1)/L j (t+2)) (8)
t i N exp(τL (t+1)/L (t+2))
m i i
where ε is denoising model, and α¯ is the hyperparameter. where τ refers to the temperature. A larger τ makes all
θ t
Eq.5 demonstrates that the score of p(x t−1 |x t ,c tar ) can be weightscloseto1.L i =CS(ϕi ψ (x˜ t ),ϕi ψ (x tar )).Weinitialize
derived by incorporating gradient information into the inverse {w (t∗)}Nm and {w (t∗ −1)}Nm to 1. Through Eq. 8, we
i i=1 i i=1
process of the diffusion model. Consequently, adversarial reduce the weight of surrogate models with fast-changing
semantics can be incrementally embedded into adversarial lossestoensurethatgradientestimationsofdifferentsurrogate
examples based on the principle of score matching. models are updated simultaneously.

| IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY |     |     |     |     |     |     |                         |     |             |                   |               | 6   |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | ----------------------- | --- | ----------- | ----------------- | ------------- | --- |
|                                                   |     |     |     |     |     |     | Algorithm               | 1:  | The overall | algorithm         | of AdvDiffVLM |     |
|                                                   |     |     |     |     |     |     | Input:Originalimagex,Nm |     |             | surrogatemodelsϕi |               |     |
θ ,adversarial
guidancescales,reversegenerationprocesstimestept∗,
maskareasizek,perturbationthresholdδ,temperatureτ,
adversarialtargetimagextar,NumberofiterationsN.
|     |     |     |     |     |     |     | Output:adversarialexamplex |                    |                | adv.  |       |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------------- | ------------------ | -------------- | ----- | ----- | --- |
|     |     |     |     |     |     |     | Initialize{wi}N            |                    | m              |       |       |     |
|     |     |     |     |     |     |     | 1                          | i=                 | 1 =1,CAM,x0=x; |       |       |     |
|     |     |     |     |     |     |     | Samplext∗                  | ∼q(xt∗|x0),letx˜t∗ |                | =x¯t∗ | =xt∗; |     |
2
|     |     |     |     |     |     |     | forn←1,···,N |     | do  |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- |
3
|     |     |     |     |     |     |     | 4 fort←t∗,···,1do |                        |     |     |                     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | ---------------------- | --- | --- | ------------------- | --- |
|     |     |     |     |     |     |     | 5                 | GetmaskmaccordingtoCAM |     |     | ;                   |     |
|     |     |     |     |     |     |     |                   |                        |     |     | // Mask generation; |     |
xt∼q(xt|x0);
6
| Fig.4. ThepipelineoftheAEGE. |     |     |     |     |     |     |     | xˆt=m⊙xt+(1−m)⊙x˜t; |     |     |     |     |
| ---------------------------- | --- | --- | --- | --- | --- | --- | --- | ------------------- | --- | --- | --- | --- |
7
|     |     |     |     |     |     |     |     |     |     | // Mask-based | combination; |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------- | ------------ | --- |
(cid:80)N
m exp(τLj(t+1)/Lj(t+2))
Figure4presentsthedetailedvisualizationresultsofAEGE. wi= j= 1 ;
8
|     |     |     |     |     |     |     |     |     | Nmexp(τLi(t+ | 1 )/ | L i( t + 2) ) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------ | ---- | ------------- | --- |
Specifically, both the target image and the current adversarial / / W e i g ht o ptimization;
example are independently input into N visual encoders g=∇ (wi (cid:80)N m CS(ϕi (xˆt),ϕi (xtar)));
|     |     |     |     | m   |     |     | 9   |     | xˆt i= 1 | ψ   | ψ   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | --- | --- |
to obtain N cosine similarity values. Here, xˆ represents // Ensemble gradient estimation;
|     | m   |     |     |     | t   |     |     |               |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --- | --- | --- |
|     |     |     |     |     |     |     |     | g=clip(g,−δ,δ | );  |     |     |     |
the current adversarial example derived from the mask, as 10 √
|     |     |     |     |     |     |     |     | score=ε | (xˆt)/ 1−α¯t+s·g; |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ----------------- | --- | --- | --- |
described in Sec. IV-C. The gradient, denoted as grad, is then 11 θ
|     |     |     |     |     |     |     |     |     |     | //  | Score calculation; |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |
√
computed by applying weights w to these values. Finally, this 12 x˜t−1=(xˆt+(1−αt)·score)/ αt;
|     |     |     |     |     |     |     |     |     |     |     | // Score matching; |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |
weightedresultiscombinedwiththenoisepredictionvalueto
end
| obtain the | final score. |     |     |     |     |     | 13  |     |     |     |     |     |
| ---------- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
14 end
| Finally,             | we set the | perturbation | threshold |             | δ, and then | clip | 15 Returnx     | adv =x˜0 |                  |     |     |     |
| -------------------- | ---------- | ------------ | --------- | ----------- | ----------- | ---- | -------------- | -------- | ---------------- | --- | --- | --- |
| the adversarial      | gradient   | to ensure    | the       | naturalness | of the      | syn- |                |          |                  |     |     |     |
| thesized adversarial | examples.  |              |           |             |             |      |                |          |                  |     |     |     |
|                      |            |              |           |             |             |      | D. Differences |          | from AdvDiffuser |     |     |     |
C. GradCAM-guided Mask Generation (GCMG) BothourmethodandAdvDiffuser[32]produceunrestricted
We detailed AEGE earlier but observed that relying solely adversarial examples using the diffusion model. Here, we
on it generates obvious adversarial features in specific areas, discuss the distinctions between them, highlighting our con-
| leading to | poor visual | effects. | To balance | visual | quality | and | tributions. |     |     |     |     |     |
| ---------- | ----------- | -------- | ---------- | ------ | ------- | --- | ----------- | --- | --- | --- | --- | --- |
attack capabilities, we propose GCMG, which uses a mask to Tasks of varying difficulty levels: AdvDiffuser is oriented
combine the forward noisy image x and the generated image towards classification models, while our research targets the
t
x˜ t . This combination distributes adversarial semantics across more intricate Vision-Language Models (VLMs). Initially,
the image, enhancing the natural visual quality of adversarial within the realm of classification tasks, each image is asso-
examples. ciated with a singular label. Conversely, in the image-to-text
First,weutilizeGradCAM[60]toderivetheclassactivation tasks, images may be linked to numerous text descriptions.
map CAM of x with respect to ground-truth label y. CAM When faced with an attack targeting a singular description,
assistsinidentifyingimportantandnon-importantareasinthe VLMshavethecapabilitytogenerateanalternatedescription,
image. Subsequently, we clip the CAM values to the range therebyneutralizingtheattack’seffectiveness.Asaresult,our
| [0.3,0.7] | and normalize | them | to obtain | the probability |     | matrix |               |     |                    |     |     |     |
| --------- | ------------- | ---- | --------- | --------------- | --- | ------ | ------------- | --- | ------------------ | --- | --- | --- |
|           |               |      |           |                 |     |        | task presents | a   | greater challenge. |     |     |     |
P. We sample according to the P to obtain the coordinate Different theoretical foundations and implementation
(x,y), and then set the k × k area around (x,y) to be 1 methods: AdvDiffuser utilizes PGD [34] to introduce high-
and remain other areas to obtain mask m. Here, m has the frequency adversarial noise, while our method employs score
same shape as x˜ . This approach disperses more adversarial matching to incorporate target semantics. These theoretical
t
| features in | non-important | areas | and less | in important | areas | of  |              |      |                |     |                 |         |
| ----------- | ------------- | ----- | -------- | ------------ | ----- | --- | ------------ | ---- | -------------- | --- | --------------- | ------- |
|             |               |       |          |              |       |     | distinctions | lead | to differences | in  | implementation: | Without |
adversarial examples, improving the natural visual effect of considering the mask, AdvDiffuser operates on the latent
adversarial examples. image x˜ , adding adversarial noise directly to x˜ . In contrast,
|     |     |     |     |     |     |     |     | t   |     |     | t   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
At each step t, we combine x t and x˜ t as following: our method modifies the predicted noise ε θ (x˜ t ), obtaining a
|     |         |           |     |     |     |     | score that | encodes | adversarial | target | semantics. | For clarity, |
| --- | ------- | --------- | --- | --- | --- | --- | ---------- | ------- | ----------- | ------ | ---------- | ------------ |
|     | xˆ =m⊙x | +(1−m)⊙x˜ |     |     |     | (9) |            |         |             |        |            |              |
t t t we include a visual comparison, as shown in Figure 5. Fur-
where m denotes the final mask, ⊙ refers to Hadamard thermore, our approach obviates the need for initiating with
Product. Afterwards, we can obtain new score by integrating Gaussian noise, initially introducing noise to x through t∗
ε θ√ (xˆ t ) with the estimated gradient and then use x˜ t−1 = steps, followed by the application of adversarial gradient to
− 1−α¯ ×score for sampling. modify score, thereby facilitating more efficient generation of
t
Finally, we take the generated adversarial example as x , adversarial examples.
0
and iterate N times to embed more target semantics into it. Distinct schemes of GradCAM utilization: The Grad-
WeprovideacompletealgorithmicoverviewofAdvDiffVLM CAM mask utilized by AdvDiffuser leads to restricted mod-
in Algorithm 1. ification of crucial image areas, rendering it inadequate for

| IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY |     |     |     |     |     |     |            |               |           |           |            |          |                |               | 7     |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | ---------- | ------------- | --------- | --------- | ---------- | -------- | -------------- | ------------- | ----- |
|                                                   |     |     |     |     |     |     | Evaluation |               | metrics:  | Following | [16],      | we       | adopt          | CLIP          | score |
|                                                   |     |     |     |     |     |     | between    | the generated |           | responses | from       | victim   | models         | and           | pre-  |
|                                                   |     |     |     |     |     |     | defined    | targeted      | texts, as | computed  | by         | ViT-B/32 |                | text encoder, |       |
|                                                   |     |     |     |     |     |     | refered    | as CLIP       | . We      | adopt     | the method |          | of calculating |               | the   |
tar
|     |     |     |     |     |     |     | attack success |     | rate (ASR) | in  | [52], positing |     | that | an attack | is  |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --- | ---------- | --- | -------------- | --- | ---- | --------- | --- |
deemedsuccessfulsolelyiftheimagedescriptionincludesthe
targetsemanticmainobject.Inordertomeasurethequalityof
|         |                       |     |             |     |                |         | adversarial     | examples | and             | the        | perceptibility |                 | of applied  | pertur-      |       |
| ------- | --------------------- | --- | ----------- | --- | -------------- | ------- | --------------- | -------- | --------------- | ---------- | -------------- | --------------- | ----------- | ------------ | ----- |
|         |                       |     |             |     |                |         | bations,        | we use   | four evaluation |            | metrics:       | SSIM            | [61],       | FID          | [62], |
|         |                       |     |             |     |                |         | LPIPS [63]      | and      | BRISQUE         | [64].      |                |                 |             |              |       |
|         |                       |     |             |     |                |         | Implementation  |          | details:        |            | Since          | our adversarial |             | diffusion    |       |
|         |                       |     |             |     |                |         | sampling        | does     | not require     | additional |                | training        | to          | the original |       |
|         |                       |     |             |     |                |         | diffusion       | model,   | we use          | the        | pre-trained    | diffusion       |             | model        | in    |
|         |                       |     |             |     |                |         | our experiment. |          | We adapt        | LDM        | [4]            | with DDIM       |             | sampler      | [56]  |
|         |                       |     |             |     |                |         | (using T        | = 200    | diffusion       | steps)     | and            | select          | the version | trained      |       |
| Fig. 5. | Different theoretical |     | foundations | and | implementation | methods |                 |          |                 |            |                |                 |             |              |       |
onImageNet.Additionally,weusefourversionsofCLIP[58],
| betweenAdvDiffuserandou√rmethod |          |       | .W √      | here“Sampling”referstox˜t−1 |        |        | =                                                    |     |     |     |     |     |     |     |     |
| ------------------------------- | -------- | ----- | --------- | --------------------------- | ------ | ------ | ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
| (cid:0) x˜t−(1−αt)·ε            | (x˜t,t)/ | 1−α¯t | (cid:1) / | αt and “Score               | Match” | refers | to                                                   |     |     |     |     |     |     |     |     |
|                                 | θ        |       |           |                             |        |        | namelyResNet-50,ResNet-101,ViT-B/16,andViT-B/32,each |     |     |     |     |     |     |     |     |
x˜t−1=(x˜t+(1−αt)·score)
trainedon400millionunpublishedimage-textpairs.Forother
|     |     |     |     |     |     |     | hyperparameters, |     | we use | s = | 35,δ | = 0.0025,t∗ |     | = 0.2,k | =   |
| --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | ------ | --- | ---- | ----------- | --- | ------- | --- |
TABLEI
THEDETAILSOFVICTIMVLMS,INCLUDECODEANDCONFIGURATION.
|             |                                       |      |     |     |         |     | 8,τ = 2 | and N    | = 10. | All the | experiments |     | are conducted |     | on  |
| ----------- | ------------------------------------- | ---- | --- | --- | ------- | --- | ------- | -------- | ----- | ------- | ----------- | --- | ------------- | --- | --- |
|             |                                       |      |     |     |         |     | a Tesla | A100 GPU | with  | 40GB    | memory.     |     |               |     |     |
| Models      |                                       | Code |     |     | Version |     |         |          |       |         |             |     |               |     |     |
| Unidifusser | https://github.com/thu-ml/unidiffuser |      |     |     |         | /   |         |          |       |         |             |     |               |     |     |
B. Main Experiments
| BLIP2     | https://github.com/salesforce/LAVIS      |     |     | (blip2 | opt,pretrain | opt2.7b) |     |     |     |     |     |     |     |     |     |
| --------- | ---------------------------------------- | --- | --- | ------ | ------------ | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| MiniGPT-4 | https://github.com/Vision-CAIR/MiniGPT-4 |     |     |        | (Vicuna7B)   |          |     |     |     |     |     |     |     |     |     |
LLaVA https://github.com/haotian-liu/LLaVA (Vicuna,llava-v1.5-7b) In this subsection, we evaluate the effectiveness of our
Img2LLM https://github.com/salesforce/LAVIS (img2prompt vqa,base) methodintargetedandtransferablescenarios.Specifically,we
firstquantitativelycomparethetransferabilityofourapproach
againstbaselinemethodsonbothopen-sourceandcommercial
| image-based | attacks.    | Addressing |         | this issue, | we introduc      | the |       |           |       |            |     |             |         |     |        |
| ----------- | ----------- | ---------- | ------- | ----------- | ---------------- | --- | ----- | --------- | ----- | ---------- | --- | ----------- | ------- | --- | ------ |
|             |             |            |         |             |                  |     | VLMs. | Following | this, | we present |     | qualitative | results |     | of our |
| GCMG.       | Contrary to | utilizing  | GradCAM |             | results directly | as  | a     |           |       |            |     |             |         |     |        |
methodappliedtotheseVLMs.Lastly,weanalyzethemodel’s
| mask, we      | employ      | them as    | a directive  | to           | generate        | the mask |                   |     |                |         |           |                |     |             |     |
| ------------- | ----------- | ---------- | ------------ | ------------ | --------------- | -------- | ----------------- | --- | -------------- | ------- | --------- | -------------- | --- | ----------- | --- |
|               |             |            |              |              |                 |          | complexity        | and | demonstrate    | its     | practical | efficiency.    |     |             |     |
| further. This | not only    | guarantees |              | a likelihood | of modification |          |                   |     |                |         |           |                |     |             |     |
|               |             |            |              |              |                 |          | Quantitative      |     | results        | on open | source    | VLMs.          |     | To validate |     |
| across all    | image areas | but        | also secures | minimal      | alteration      | of       |                   |     |                |         |           |                |     |             |     |
|               |             |            |              |              |                 |          | the effectiveness |     | of AdvDiffVLM, |         | we        | quantitatively |     | evaluate    |     |
significantareas,strikingabalancebetweenimagequalityand
thetransferabilityofadversarialexamplesgeneratedbyAdvD-
attack ability.
|     |     |     |     |     |     |     | iffVLM | and baseline | methods |     | on various | open | source | VLMs. |     |
| --- | --- | --- | --- | --- | --- | --- | ------ | ------------ | ------- | --- | ---------- | ---- | ------ | ----- | --- |
AsshowninTableII,allmethodsdemonstratefavorableattack
|                 |       | V. EXPERIMENTS |     |     |     |     |            |        |                |               |        |              |        |           |     |
| --------------- | ----- | -------------- | --- | --- | --- | --- | ---------- | ------ | -------------- | ------------- | ------ | ------------ | ------ | --------- | --- |
|                 |       |                |     |     |     |     | results in | gray   | box scenarios. |               | In the | transfer     | attack | scenario, |     |
| A. Experimental | Setup |                |     |     |     |     |            |        |                |               |        |              |        |           |     |
|                 |       |                |     |     |     |     | our method | yields | the            | best results. |        | For example, |        | on BLIP2, |     |
Datasets and victim VLMs: Following [52], we use ourmethodimprovesCLIP andASRby0.0200and10.9%
tar
NeurIPS’17 adversarial competition dataset, compatible with , respectively, when compared to SIA-CWA. Furthermore,
ImageNet,foralltheexperiments.Inaddition,weselect1,000 our method generates adversarial examples much faster than
text descriptions from the captions of the MS-COCO dataset baselines. Specifically, when compared to AdvDiffuser, SIA
asouradversarialtargettextsandthenuseStableDiffusion[4] andSSAmethods,ourmethodgeneratesadversarialexamples
to generate 1,000 adversarial targeted images. For the victim 5x to 10x faster. Experimental results show that our method
|            |             |     |        |                |     |           | generates | adversarial | examples |     | with | better | transferability |     | at a |
| ---------- | ----------- | --- | ------ | -------------- | --- | --------- | --------- | ----------- | -------- | --- | ---- | ------ | --------------- | --- | ---- |
| VLMs, SOTA | open-source |     | models | are evaluated, |     | including |           |             |          |     |      |        |                 |     |      |
Unidiffuser [5], BLIP2 [2], MiniGPT-4 [47], LLaVA [3] and faster rate, demonstrating its superiority. To ensure statistical
Img2LLM [35]. The details are shown in Table I. Among significance,werepeateachexperimentthreetimesandreport
them, Unidiffuser is a gray-box model, and the others are the standard deviation.
black-box models. Additionally,ithasbeenobservedthatAdvDiffuserexhibits
Baselines: We compare with AdvDiffuser [32] and other suboptimal performance in challenging attack scenarios, par-
SOTA transfer-based attackers described in Section III-C. ticularlyagainstVLMs.Thisisattributedtoitsdirectapplica-
Since AdvDiffuser is used for classification models, we use tion of GradCAM as the mask, which restricts the modifiable
cosinesimilaritylossinsteadofclassificationlossforadversar- area for adversarial examples in demanding tasks, thereby
ialattacksonVLMs.Forafaircomparison,weimplementthe diminishingattackeffectiveness.Simultaneously,AdvDiffuser
ensemble version of AdvDiffuser, including simple ensemble employs high-frequency adversarial noise to alter semantics.
and adaptive ensemble, which are denoted as AdvDiffuser , Thisadversarialnoise,beinginherentlyfragile,issignificantly
ens
AdvDiffuser respectively.Forhyperparameters(inAd- mitigatedduringthediffusionmodel’sreverseprocess,further
adaptive
vDiffuser), we choose T =200,σ =0.4,I =25. diminishing its attack potential on complex tasks. These ob-

| IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY |     |     |     |     |     |     |     |     |     |     |     | 8   |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
TABLEII
COMPARISONWITHEXISTINGSOTAATTACKMETHODS,WHERETHEBESTRESULTISBOLDED.WEALSOREPORTTHESTANDARDDEVIATIONOFTHE
RESULTS. NOTETHATWEUSEFOURVERSIONSOFTHECLIPVISUALENCODER,INCLUDINGRESNET50,RESNET101,VIT-B/16ANDVIT-B/32,AS
SURROGATEMODELS.SINCEUNIDIFFUSERUSESVIT-B/32ASTHEVISUALENCODER,ITISAGRAYBOXSCENARIO,WHICHWEINDICATEWITH*.IN
ADDITION,WEPROVIDETHEAVERAGETIME(S)FOREACHSTRATEGYTOCRAFTASINGLEx adv.THE SHADEDPARTS REPRESENTOURPROPOSED
METHOD.
|     |     | Unidiffuser* |     | BLIP2 |     | MiniGPT-4 |     | LLaVA |     | Img2LLM |     |     |
| --- | --- | ------------ | --- | ----- | --- | --------- | --- | ----- | --- | ------- | --- | --- |
CLIPtar↑ ASR↑ CLIPtar↑ ASR↑ CLIPtar↑ ASR↑ CLIPtar↑ ASR↑ CLIPtar↑ ASR↑ Time(s)
Original 0.4770±0.0017 0.0%±0.00% 0.4931±0.0027 0.0%±0.00% 0.4902±0.0030 0.0%±0.00% 0.5190±0.0052 0.0%±0.00% 0.5288±0.0039 0.0%±0.00% /
Ens 0.7353±0.0012 99.1%±0.14% 0.5085±0.0019 0.9%±0.11% 0.4980±0.0035 1.8%±0.14% 0.5366±0.0052 3.5%±0.20% 0.5297±0.0027 4.5%±0.22% 69
SVRE 0.7231±0.0020 100.0%±0.00% 0.5190±0.0023 2.4%±0.15% 0.5107±0.0029 2.2%±0.12% 0.5385±0.0049 4.6%±0.18% 0.5292±0.0035 3.8%±0.17% 125
CWA 0.7568±0.0016 100.0%±0.00% 0.5249±0.0022 5.2%±0.27% 0.5211±0.0033 3.8%±0.20% 0.5493±0.0057 7.1%±0.26% 0.5346±0.0042 5.4%±0.04% 101
SSA-Ens 0.7275±0.0031 100.0%±0.00% 0.5539±0.0050 9.2%±0.42% 0.5175±0.0052 10.1%±0.33% 0.6098±0.0056 37.5%±0.49% 0.5629±0.0050 19.6%±0.31% 879
SSA-SVRE 0.7217±0.0039 100.0%±0.00% 0.5776±0.0046 18.7%±0.40% 0.5395±0.0056 16.5%±0.47% 0.6005±0.0063 40.2%±0.57% 0.5625±0.0067 18.4%±0.39% 1012
SSA-CWA 0.7485±0.0024 100.0%±0.00% 0.5888±0.0041 23.3%±0.57% 0.5407±0.0057 20.6%±0.45% 0.6152±0.0070 40.7%±0.52% 0.5634±0.0061 20.4%±0.33% 1225
SIA-Ens 0.7377±0.0058 100.0%±0.00% 0.5956±0.0074 49.6%±1.40% 0.5605±0.0064 40.4%±1.06% 0.7158±0.0085 84.7%±1.80% 0.6337±0.0073 27.0%±1.27% 483
SIA-SVRE 0.7302±0.0066 100.0%±0.00% 0.6102±0.0068 50.1%±0.79% 0.5782±0.0080 46.4%±1.17% 0.7122±0.0079 88.3%±1.73% 0.6305±0.0087 35.4%±1.54% 596
SIA-CWA 0.7498±0.0053 100.0%±0.00% 0.6135±0.0085 51.8%±1.18% 0.5810±0.0064 47.8%±1.24% 0.7194±0.0086 89.5%±2.95% 0.6401±0.0078 40.6%±1.30% 732
AdvDiffuserens 0.6774±0.0037 86.7%±1.93% 0.5396±0.0034 8.6%±0.26% 0.5371±0.0041 8.2%±0.37% 0.5507±0.0071 25.3%±0.37% 0.5395±0.0063 11.5%±0.25% 574
AdvDiffuseradaptive 0.6932±0.0029 88.9%±1.75% 0.5424±0.0062 10.4%±0.35% 0.5391±0.0046 9.6%±0.30% 0.5595±0.0064 27.4%±0.41% 0.5502±0.0060 14.8%±0.32% 602
AdvDiffVLM 0.7502±0.0072 100.0%±0.00% 0.6435±0.0101 66.7%±1.86% 0.6145±0.0096 58.6%±2.07% 0.7206±0.0113 91.2%±2.35% 0.6521±0.0107 43.8%±1.92% 139
Fig.6. Visualizationoftheattackresultsofourmethodonvariousopen-sourceVLMs.Weshowtheadversarialtargettextabovetheimage,anddisplaythe
imagecaptionresultsoforiginalimageandadversarialexamplebelowtheimage.
TABLEIII OpenAI’s GPT-4V2, Google’s Gemini3, Microsoft’s Copilot4,
THERESULTOFATTACKINGCOMMERCIALVLMS.WEREPORTASRAND and Baidu’s ERNIE Bot5. We choose SIA-CWA to represent
PROVIDETHEAVERAGETIME(S)FOREACHSTRATEGYTOCRAFTA
|     |     |     |     |     |     | baselines | and ASR | as  | an evaluation | metric. | We choose | 100 |
| --- | --- | --- | --- | --- | --- | --------- | ------- | --- | ------------- | ------- | --------- | --- |
SINGLEx adv.THEBESTRESULTISBOLDED.
|     |        |        |         |          |         | images from   | the               | NeurIPS’17   | adversarial | competition  |          | dataset |
| --- | ------ | ------ | ------- | -------- | ------- | ------------- | ----------------- | ------------ | ----------- | ------------ | -------- | ------- |
|     |        |        | .       |          |         | and 100       | text descriptions |              | from        | the MS-COCO  | dataset  | as      |
|     | GPT-4V | Gemini | Copilot | ERNIEBot | Time(s) |               |                   |              |             |              |          |         |
|     |        |        |         |          |         | target texts. | Table             | III presents | the         | experimental | results. | Our     |
Noattack 0% 0% 0% 0% / methodoutperformsSIA-CWAintermsofattacksuccessrate,
| SIA-CWA | 35% |     | 12% 25% | 50% | 732 |     |     |     |     |     |     |     |
| ------- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
AdvdiffVLM 37% 17% 26% 58% 139 demonstrating its superior transferability.
QualitativeresultsonopensourceVLMs.Wethenpresent
|                |          |                |     |          |           | visualizations | depicting    |       | the outcomes   | of our         | method’s       | attacks |
| -------------- | -------- | -------------- | --- | -------- | --------- | -------------- | ------------ | ----- | -------------- | -------------- | -------------- | ------- |
|                |          |                |     |          |           | on open        | source VLMs, |       | as illustrated | in Figure      | 6. Considering |         |
|                |          |                |     |          |           | the image      | caption      | task, | we focus       | on two models: | Unidiffuser    |         |
| servations     | validate | the advantages | of  | our GCMG | and score |                |              |       |                |                |                |         |
| matching idea. |          |                |     |          |           |                |              |       |                |                |                |         |
2https://chat.openai.com/
3https://gemini.google.com/
| Quantitative |     | results | on commercial | VLMs. | We con- |     |     |     |     |     |     |     |
| ------------ | --- | ------- | ------------- | ----- | ------- | --- | --- | --- | --- | --- | --- | --- |
4https://copilot.microsoft.com/
duct a quantitative evaluation of commercial VLMs such as 5https://yiyan.baidu.com/

IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY 9
Fig.7. Visualizationresultsofusingthesameadversarialexampletoattack
variousvictimmodels.Wherethetoponeistheoriginalimage,andthebottom
oneistheadversarialexample.Thetargetsemanticsismarkedingreen.
andBLIP2.ConsideringtheVQAtask,wefocusonMiniGPT-
4, LLaVA and Img2LLM. In the case of MiniGPT-4, the
input text is configured as “What is the image showing?”. Fig. 8. Convergence analysis of different methods, with the horizontal axis
For LLaVA, the input text is set to “What is the main
denotingthenumberofiterationsNI.
contain of this image?”, and the prefix “The main contain TABLEIV
is” is omitted in the output. For Img2LLM, the input text COMPARATIVERESULTSUSINGTHESINGLESURROGATEMODEL.
is configured as “What is the content of this image?”. Our
Method Unidiffuser BLIP2 LLaVA Img2LLM
method demonstrates the capability to effectively induce both
SSA-Ens 0.7356 0.5024 0.5522 0.5363
gray-box and black-box VLMs to produce adversarial target
SIA-Ens 0.7473 0.5330 0.5923 0.5393
semantics. For example, in the case of LLaVA’s attack, we AdvDiffuseradaptive 0.6930 0.5011 0.5238 0.5297
define the adversarial target text as “A cake that has various AdvDiffVLM 0.6982 0.5279 0.5793 0.5402
gelatinsinit.”LLaVAgeneratetheresponse“Themaincontain
isaclose-upviewofapartiallyeatencakewithchocolateand
proach. Let N and N respectively denote the total number
whitefrosting.”asthetargetoutput,whiletheoriginalimage’s s d
of parameters of the ensemble surrogate models and the
contentisdescribedas“Themaincontainisabird,specifically
diffusion model. Since N corresponds to the parameters of
a seagull, walking on the beach near the water.”. s
the ensemble models, it follows that N ≪ N . In terms of
Inaddition,wevisualizetheoutputsofvariousvictimmod- d s
space complexity, our method and AdvDiffuser both exhibit
els with the same adversarial example, as shown in Figure 7.
space complexity of O(N + N ), whereas SSA exhibits
The visualization demonstrates that the adversarial example d s
O(N ). Since SIA processes N images in parallel, its space
successfully induces all victim models to produce the target s t
complexityisO(N ·N ).Thus,thespacecomplexitiesofour
semantics. t s
method, AdvDiffuser, and SSA are comparable and notably
Qualitative results on commercial VLMs. We finally
lower than SIA. For time complexity, all methods exhibit
show screenshots of successful attacks on various commercial
linear time complexity, depending on the number of iterations
VLMs image description tasks, including Google’s Gemini,
and the duration of each iteration. AdvDiffuser and SSA
Microsoft’s Copilot, Baidu’s ERNIE Bot, and OpenAI’s GPT-
involveinternalloopswithineachiteration,resultinginlonger
4V, as shown in Figure 9. These models are large-scale visual
iteration durations. SIA processes multiple images in parallel,
languagemodelsdeployedcommercially,andtheirmodelcon-
which slightly increases iteration durations. In contrast, our
figurations and training datasets have not been made public.
method not only achieves the shortest iteration duration but
Moreoever, compared with open source VLMs, these models
alsorequiresfeweriterations,makingitthemosttime-efficient
areequippedwithmorecomplexdefensemechanisms,making
among the compared methods.
them more difficult to attack. However, as shown in Figure 9,
our method successfully induces these commercial VLMs to To further demonstrate the time efficiency of our method,
generate target responses. For example, in GPT-4V, we define we conduct the convergence analysis. We select CWA, SSA,
theadversarialtargettextas“akidisdoingaskateboardtrick and SIAas comparison methods. The experimentalresults are
downsomestairs.”GPT-4Vgeneratestheresponse“Themain depicted in Figure 8. Our method converges to a flat trend
content of this image is a skateboarder performing a trick when N I = 200 (N = 10), whereas other methods achieve
on a skateboard ramp...”, while the semantics of the original
convergenceatN
I
=300.Moreover,ourmethodachievesthe
imageis“Abirdstandingonabranch.”Moreover,ourmethod same attack effect with fewer iterations.
is also applicable to various languages. For example, we Thecomprehensiveexperimentsandanalysesabovedemon-
use English to generate adversarial examples but successfully stratethatourmethoddeliverssuperiorattackperformanceon
attack ERNIE Bot, which operates in Chinese. both open-source and commercial VLMs. Moreover, it offers
Computational resource analysis. We compare the com- notable advantages in terms of resource efficiency and faster
putational complexity of competing methods with our ap- convergence speed.

IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY 10
Fig.9. ScreenshotsofsuccessfulattacksagainstvariouscommercialVLMsAPI’simagedescription.Wegivetheadversarialtargettextontherightsideof
theimage.Inaddition,wemarkthemainobjectsoftheadversarialtargetinredandthemainobjectsintheAPI’sresponseingreen.
TABLEV
COMPARISONRESULTSOFDEFENSEEXPERIMENTSWITHSOTAMETHODSIA.WEUSECLIPtarEVALUATIONMETRICANDREPORTTHEREDUCTION
RESULTSOFCLIPtarWHERETHEBESTRESULTISBOLDED.OTHERWISE,THEPARENTHESESREPRESENTTHEHYPERPARAMETERS(INTHEIRPAPER).
Defensemodels Attackmethods Unidiffuser BLIP2 MiniGPT-4 LLaVA Img2LLM
| SIA-Ens | 0.7204↓0.0173 | 0.5602↓0.0454 | 0.5273↓0.0432 | 0.7034↓0.0124 | 0.6284↓0.0053 |
| ------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| SIA-CWA | 0.7281↓0.0217 | 0.5798↓0.0435 | 0.5442↓0.0468 | 0.7063↓0.0131 | 0.6375↓0.0026 |
BitReduction(4)
AdvDiffVLM 0.7397↓0.0105 0.6320↓0.0115 0.6261↓0.0084 0.7168↓0.0038 0.6501↓0.0020
| SIA-Ens | 0.7192↓0.0185 | 0.5571↓0.0485 | 0.5192↓0.0513 | 0.6968↓0.0190 | 0.6230↓0.0107 |
| ------- | ------------- | ------------- | ------------- | ------------- | ------------- |
STL(k=64,s=8,λ=0.2) SIA-CWA 0.7233↓0.0265 0.5733↓0.0500 0.5385↓0.0525 0.7001↓0.0193 0.6314↓0.0087
AdvDiffVLM 0.7329↓0.0173 0.6267↓0.0168 0.5997↓0.0148 0.7145↓0.0061 0.6471↓0.0050
| SIA-Ens | 0.6734↓0.0642 | 0.5345↓0.0711 | 0.5002↓0.0703 | 0.6542↓0.0616 | 0.6020↓0.0317 |
| ------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| SIA-CWA | 0.6801↓0.0697 | 0.5525↓0.0708 | 0.5273↓0.0637 | 0.6550↓0.0644 | 0.6088↓0.0313 |
JPEGCompression(p=50)
AdvDiffVLM 0.6896↓0.0606 0.6218↓0.0217 0.5865↓0.0380 0.6983↓0.0223 0.6354↓0.0167
| SIA-Ens | 0.6087↓0.1290 | 0.5134↓0.0922 | 0.4986↓0.0719 | 0.6274↓0.0884 | 0.5771↓0.0566 |
| ------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| SIA-CWA | 0.6114↓0.1384 | 0.5290↓0.0943 | 0.5114↓0.0796 | 0.6331↓0.0863 | 0.5842↓0.0559 |
DISCO(s=3,k=5)
AdvDiffVLM 0.6215↓0.1287 0.5892↓0.0543 0.5727↓0.0418 0.6728↓0.0478 0.6093↓0.0428
| SIA-Ens | 0.5642↓0.1735 | 0.5025↓0.1031 | 0.4878↓0.0827 | 0.6067↓0.1091 | 0.5681↓0.0656 |
| ------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| SIA-CWA | 0.5735↓0.1763 | 0.5176↓0.1057 | 0.5074↓0.0836 | 0.6106↓0.1088 | 0.5692↓0.0709 |
DISCO+JPEG
AdvDiffVLM 0.5924↓0.1578 0.5859↓0.0576 0.5650↓0.0495 0.6724↓0.0482 0.6081↓0.0440
| SIA-Ens | 0.4921↓0.2456 | 0.5048↓0.1008 | 0.4919↓0.0786 | 0.5356↓0.1802 | 0.5372↓0.0965 |
| ------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| SIA-CWA | 0.4942↓0.2556 | 0.5099↓0.1136 | 0.5025↓0.0885 | 0.5360↓0.1835 | 0.5388↓0.1013 |
DiffPure(t*=0.15)
AdvDiffVLM 0.5837↓0.1665 0.5527↓0.0908 0.5506↓0.0639 0.5857↓0.1349 0.5711↓0.0810

| IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY |     |     |     |     |     |     |     |     |     |     |     |     | 11  |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
TABLEVI
DEFENSERESULTSWITHDIFFPURE.THESETTINGARETHESAMEASTABLEIIEXCEPTTHEADVERSARIALEXAMPLESAREPURIFIEDBYDIFFPURE.IN
THISTABLE,CLIPtarEVALUATESTHESIMILARITYBETWEENTHERESULTSOFPURIFIEDEXAMPLESANDTHETARGETTEXTS.
|     |     |     | Unidiffuser* |     |     | BLIP2 |     | MiniGPT-4 |     | LLaVA |     | Img2LLM |     |
| --- | --- | --- | ------------ | --- | --- | ----- | --- | --------- | --- | ----- | --- | ------- | --- |
CLIPtar ↑ ASR↑ CLIPtar ↑ ASR↑ CLIPtar ↑ ASR↑ CLIPtar ↑ ASR↑ CLIPtar ↑ ASR↑
Original 0.4802 0.0% 0.4924 0.0% 0.4831 0.0% 0.5253 0.0% 0.5302 0.0%
Ens 0.4833 0.0% 0.4929 0.0% 0.4840 0.0% 0.5263 0.0% 0.5332 0.0%
SVRE 0.4846 0.7% 0.4953 0.0% 0.4852 0.0% 0.5264 0.0% 0.5312 0.0%
CWA 0.4873 2.1% 0.4973 0.0% 0.4901 1.0% 0.5272 0.8% 0.5307 0.0%
SSA-Ens 0.4914 0.9% 0.5024 0.0% 0.4916 0.0% 0.5280 1.2% 0.5322 0.0%
SSA-SVRE 0.4899 2.1% 0.4984 0.2% 0.4918 0.0% 0.5273 1.2% 0.5356 0.0%
SSA-CWA 0.4868 2.5% 0.4997 0.0% 0.4997 0.0% 0.5283 2.8% 0.5367 0.7%
SIA-Ens 0.4921 3.7% 0.5048 1.2% 0.4919 1.1% 0.5356 2.5% 0.5372 1.6%
SIA-SVRE 0.4930 3.9% 0.5012 1.8% 0.5011 1.6% 0.5349 4.2% 0.5380 2.5%
SIA-CWA 0.4942 5.8% 0.5099 2.6% 0.5025 2.2% 0.5360 4.0% 0.5388 1.5%
AdvDiffuserens 0.4920 4.2% 0.4933 2.6% 0.4906 2.4% 0.5325 3.7% 0.5310 2.7%
AdvDiffuseradaptive 0.4922 4.5% 0.5001 3.2% 0.5001 3.2% 0.5336 3.4% 0.5325 2.8%
AdvDiffVLM 0.5837 22.4% 0.5527 10.2% 0.5506 12.6% 0.5857 18.0% 0.5711 10.5%
C. More Experiments
TABLEVII
QUALITYCOMPARISONOFADVERSARIALEXAMPLESUNDERFOUR
Experiment with a single surrogate model. All experi- EVALUATIONMETRICS.THEBESTRESULTISBOLDED.
| ments in  | the previous | subsection          |     | use the | ensemble    | surrogate |     |        |     |       |        |               |     |
| --------- | ------------ | ------------------- | --- | ------- | ----------- | --------- | --- | ------ | --- | ----- | ------ | ------------- | --- |
|           |              |                     |     |         |             |           |     | Method |     | SSIM↑ | LPIPS↓ | FID↓ BRISQUE↓ |     |
| models to | improve      | the transferability |     | of      | adversarial | examples. |     |        |     |       |        |               |     |
To further illustrate the effect of the single surrogate model, SSA-Ens 0.6687 0.3320 110.5 66.89
|     |     |     |     |     |     |     |     | SSA-SVRE |     | 0.6610 | 0.3325 | 112.6 70.05 |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | ------ | ------ | ----------- | --- |
we conduct the comparative experiment and select ViT-B/32 SSA-CWA 0.6545 0.3673 123.4 67.67
| as the surrogate | model. |     | The experimental |     | results | are | shown |         |     |        |        |             |     |
| ---------------- | ------ | --- | ---------------- | --- | ------- | --- | ----- | ------- | --- | ------ | ------ | ----------- | --- |
|                  |        |     |                  |     |         |     |       | SIA-Ens |     | 0.6925 | 0.2990 | 117.3 55.61 |     |
in Table IV. As shown, our method outperforms SSA and SIA-SVRE 0.6920 0.3042 120.0 57.42
AdvDiffuser but is slightly less effective than SIA. This is SIA-CWA 0.6892 0.3306 125.3 56.02
because the score estimated by the single surrogate model AdvDiffuserens 0.6520 0.3074 115.5 14.61
|          |               |      |          |        |              |     |     | AdvDiffuseradaptive |     | 0.6471 | 0.3096 | 126.7 15.32 |     |
| -------- | ------------- | ---- | -------- | ------ | ------------ | --- | --- | ------------------- | --- | ------ | ------ | ----------- | --- |
| deviates | significantly | from | the true | score. | Furthermore, |     | the |                     |     |        |        |             |     |
transferabilityofasinglesurrogatemodelissignificantlylower AdvDiffVLM 0.6992 0.2930 107.4 32.96
| compared | to that | of the | ensemble | surrogate | models. |     |     |     |     |     |     |     |     |
| -------- | ------- | ------ | -------- | --------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
Againstadversarialdefensemodels.Ourmethodachieves both CLIP and CLIP reduction results of our methods
|          |                    |     |     |           |        |     |      |            | tar            | tar  |              |                 |     |
| -------- | ------------------ | --- | --- | --------- | ------ | --- | ---- | ---------- | -------------- | ---- | ------------ | --------------- | --- |
|          |                    |     |     |           |        |     |      | outperform | the baselines. | This | demonstrates | the superiority | of  |
| superior | attack performance |     | on  | both open | source | and | com- |            |                |      |              |                 |     |
mercial VLMs. In recent years, various adversarial defense our method against defense methods compared to baselines.
methods have been proposed to mitigate the threat of adver- To better evaluate the resistance of our method against ad-
sarial examples. Defense methods can be broadly categorized versarialdefensemethods,wefurtherindetailshowtheresults
into adversarial training and data preprocessing. Due to the of the SOTA defense method, namely DiffPure, in Table VI.
Itcanbefoundthatourmethodoutperformsbaselinesinboth
| high time | and resource |     | costs and | instability |     | of adversarial |     |     |     |     |     |     |     |
| --------- | ------------ | --- | --------- | ----------- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- |
training [65], it has not been applied to VLM defense. In gray-boxandblack-boxsettings.Forexample,onUnidiffuser,
contrast, data preprocessing is model-independent and highly for CLIP score, our method is 0.0895 higher than SIA-
tar
adaptable, making it a popular defense strategy across various CWA. On BLIP2, for CLIP tar score, our method is 0.0428
models. To demonstrate the effectiveness of our method in higher than SIA-CWA. Furthermore, in all cases, the attack
successrateofourmethodsishigherthanthebaselines.These
| resisting | data preprocessing |     | attacks, |     | we conduct | extensive |     |     |     |     |     |     |     |
| --------- | ------------------ | --- | -------- | --- | ---------- | --------- | --- | --- | --- | --- | --- | --- | --- |
experimentsonBitReduction[66],STL[67],JPEGCompres- experimentalresultsdemonstratethatourmethodoutperforms
sion[68],DISCO[69],JPEG+DISCO,andDiffPure[70].The baselines in evading the DiffPure defense method.
data preprocessing techniques we used are grouped into three We can break the SOTA defense method Diffpure with an
categories:introducingrandomness,denoising,anddatarecon- attack success rate of more than 10% in a completely black-
struction. Bit Reduction introduces randomness by modifying box scenario, exposing the flaws in current defense methods
image bits; JPEG Compression performs denoising through and raising new security concerns for designing more robust
blurring operations; and other methods involve reconstructing deep learning models.
adversarial examples using various reconstruction networks Image quality comparison. The image quality of adver-
and techniques. We report the CLIP metric. At the same sarial examples is also particularly important. Adversarial
tar
time, we report the CLIP reduction results, which more examples with poor image quality can be easily detected. We
tar
accurately reflect the ability of adversarial examples to re- further evaluate the image quality of the generated adversarial
sist defense methods. The experimental results are shown in examples using four evaluation metrics: SSIM, FID, LPIPS,
Table V. It can be observed that, for all defense methods, andBRISQUE.AsshowninTableVII,comparedtobaselines,

IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY 12
Fig. 10. Visualization of adversarial perturbations generated by different
attack methods. Note that the first row represents adversarial examples, and
the second row represents adversarial perturbations. We choose SIA-CWA
and AdvDiffuseradaptive as representatives of baselines. We amplify the
perturbationvaluesforbettervisualization.
the adversarial examples generated by our method exhibit
Fig. 11. Comparison results of different ablation methods. Here, “Single”
higher image quality. Specifically, our results are significantly
meansusing asingleViT-B/32 tocalculatethe loss,“Ens”means usingthe
better than the baselines in terms of SSIM, LPIPS, and simpleensemblestrategy,and“w/omask”meansnotusingGCMGmodule.
FID evaluation metrics. For the BRISQUE metric, AdvD-
TABLEVIII
iffuser outperforms our method. This is because BRISQUE
COMPARISONOFIMAGEQUALITYOFADVERSARIALEXAMPLESBEFORE
is a reference-free image quality assessment algorithm and ANDAFTERUSINGTHEGCMGMODULE.THEBESTRESULTISBOLDED.
is sensitive to blur, noise, color change, etc. As shown in
Method SSIM↑ LPIPS↓ FID↓ BRISQUE↓
Figure10,theadversarialexamplesgeneratedbyAdvDiffuser
lackobviousabnormalitiesintheseelements,soitsresultsare w/omask 0.7129 0.2687 111.9 16.92
Ours 0.7188 0.2358 96.1 16.80
marginallybetterthanourmethod.However,asshowninFig-
ure10,theperturbationintroducedbyourmethodissemantic,
while AdvDiffuser significantly alters the non-salient area,
Does GCMG module help trade-off image quality and
resultinginpoorvisualeffects.Thisshowsthattheadversarial
attack capability? Next, we explore the role of the GCMG
examples generated by AdvDiffuser are unsuitable for more
module in balancing image quality and transferability. We
complexscenarios,suchasattackingVLMs.Inaddition,itcan
compare this with the w/o mask method, and the results are
beseenthattheadversarialexamplesgeneratedbythetransfer-
presented in Figure 11. As shown in Figure 11(a) and (b),
based methods exhibit significant noise, indicating that our
the use of the GCMG module results in a slight decrease in
method has obvious superiority in terms of image quality.
the transferability and robustness of the adversarial examples.
However,asshowninFigure11(c),theabsenceoftheGCMG
module leads to the adversarial examples exhibiting obvious
D. Ablation Experiments
target features, and the use of the GCMG module enhances
To further understand the effectiveness of AdvDiffVLM, the visual quality of the adversarial example. In addition,
we discuss the role of each module. We set N = 1 to TableVIIIfurthershowsthattheGCMGmodulecanimprove
more conveniently discuss the impact of each module. We the visual quality of adversarial examples. The experimental
consider three cases, including using only a single ViT-B/32 resultsdemonstratethatGCMGeffectivelybalancesthevisual
tocalculatetheloss,usingasimpleensemblestrategy,andnot quality and attack capability of the adversarial examples.
using the GCMG module, named Single, Ens, and w/o mask Then we perform ablation experiments on different config-
respectively. urations of GCMG. Firstly, we conduct experiments on the
Is AEGE module beneficial for boosting the attack effects of the value range of the CAM. The CAM value
capability? We first explore whether the AEGE module range is used to balance image quality with attack capability.
could help improve the transferability and robustness of ad- Cropping the lower boundary increases the probability of
versarial examples. We divide the AEGE module into two selectingnon-criticalareas,whilecroppingtheupperboundary
approaches, Single and Ens, and maintain all other conditions reduces the probability of selecting critical areas, enhancing
constant. The results are shown in Figure 11(a) and (b). the overall quality of adversarial examples. To determine
It is observable that the ensemble method exhibits better an optimal range, we test several intervals: [0,1], [0,0.3],
performance in transferability and robustness compared to [0.7,1], [0.2,0.8], [0.3,0.7], and [0.4,0.6]. The results, shown
the single loss method. Furthermore, the performance of the in Figure 12 (a), indicate that adjusting the CAM value
adaptive ensemble method is enhanced compared to the basic range allows for a trade-off between attack capability and
ensemble method. The experimental results demonstrate that image quality. Based on these results, we select the range
the AEGE module enhances the transferability and robustness [0.3,0.7] to achieve an optimal balance between quality and
of adversarial examples. attack effectiveness. Then we conduct experiments on the

IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY 13
Fig. 12. Ablation study of the different configurations of obtaining m. We
adopt the CLIPtar and LPIPS scores to show the impact of transferability
andimagequalitywithfourVLMs.
Fig.14. Transferabilityofadversarialexamplesonvariousblack-boxVLMs
asN changesfrom1to12.
results are shown in Figure 13. It is evident that all pa-
rameters influence the trade-off between transferability and
image quality. Increasing values for parameters s, t∗, and
δ enhance transferability but diminish the visual quality of
adversarial examples. This is because larger values for these
parameters result in a greater perturbation, allowing for the
embedding of more adversarial semantics into the image.
Conversely, increasing the value of k produces adversarial
examples with improved visual effects but reduces transfer-
ability. The reason is that larger values of k result in a larger
generated mask, making it more challenging to modify the
important areas in the image. To achieve an optimal trade-
off between transferability and image quality, we empirically
select s=35,t∗ =0.2,k=8 and δ =0.0025.
Fig.13. Theimpactofinnerloophyperparameters.WeadopttheCLIPtarand
LPIPSscorestoshowtheimpactoftransferabilityandimagequalitywithfour The impact of outer loop hyperparameter. Next, we
VLMs.AhigherCLIPtarvalueindicatesbetterperformance,whereasalower investigate the impact of the outer hyperparameter N on the
LPIPSvaluesignifiesbetterresults.Weonlyvaryoneofthehyperparameters
transferability of adversarial examples. We conduct experi-
at a time, and then fix the other three hyperparameters to the preset values
showninSectionV-A.Note:theresultsofCLIPtar arepresentedusingbar ments on BLIP2, MiniGPT-4, LLaVA, and Img2LLM with
graphs,whileLPIPSresultsaredepictedusingdot-linegraphs. s = 35,t∗ = 0.2,k = 8, and δ = 0.0025. The experimental
results are shown in Figure 14. It can be found that N
effects of different attention generation methods. In the field
improves the transferability of adversarial examples, but the
of adversarial attacks, GradCAM is a commonly used method
improvement gradually fades. Specifically, the increase in
for generating masks, as demonstrated in [32], [71], [72]. To
transferability is limited after N = 6,6,8,10 for BLIP2,
further illustrate the role of various attention mechanisms in
MiniGPT-4, Img2LLM, and LLaVA. Given that increasing N
mask generation, we compare CAM [73], GradCAM [60],
increases the computational cost, we choose N =10 to strike
GradCAM++[74],andLayerCAM[75],withresultsshownin
a balance between transferability and cost.
Figure 12 (b). Our findings indicate that differences in attack
andtransfercapabilitiesamongadversarialexamplesgenerated
bythesemechanismsareminimal,withvariationsinCLIPand
VI. CONCLUSION
LPIPS values within 0.001. Moreover, we observe a trade-off
between attack capability and image quality. After weighing Inthiswork,weproposeAdvDiffVLM,anunrestrictedand
both factors and for consistency with previous studies, we targeted adversarial example generation method for VLMs.
opted to use GradCAM for mask generation in this paper. We design the AEGE based on the idea of score matching.
It embeds the target semantics into adversarial examples,
which can generate targeted adversarial examples that ex-
E. Hyperparameter Studies
hibit enhanced transferability in a more efficient manner. To
In this subsection, we explore the impact of hyperparame- balance adversarial example quality and attack effectiveness,
ters, including inner loop hyperparameters s, t∗, k, and δ and we propose the GCMG module. Additionally, we enhance
outer loop hyperparameter N. the embedding of target semantics into adversarial examples
The impacts of inner loop hyperparameters. We first through multiple iterations. Extensive experiments show that
discuss the impacts of inner loop hyperparameters (including our method generates targeted adversarial examples 5x to 10x
the s, t∗, k, and δ). We set N = 1 and conduct tests on times faster than baseline methods while achieving superior
Unidiffuser, BLIP2, LLaVA and Img2LLM. The experimental transferability.

| IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY |     |     |     |     |     |     |     |     |     |     |     |     |     | 14  |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
IMPACTSTATEMENTS
|     |     |     |     |     |     |     | [17] X. Jia, | Y. Chen, | X.  | Mao, R. Duan, | J. Gu, | R. Zhang, | H.  | Xue, Y. Liu, |
| --- | --- | --- | --- | --- | --- | --- | ------------ | -------- | --- | ------------- | ------ | --------- | --- | ------------ |
andX.Cao,“Revisitingandexploringefficientfastadversarialtraining
Our research mainly aims to discover vulnerabilities in via law: Lipschitz regularization and auto weight averaging,” IEEE
open-sourcelargeVLMsandcommercialVLMssuchasGPT- TransactionsonInformationForensicsandSecurity,2024.
|               |          |     |                |      |        |            | [18] X. Jia, | Y. Zhang, | X.  | Wei, B. | Wu, K. | Ma, J. Wang, | and | X. Cao, |
| ------------- | -------- | --- | -------------- | ---- | ------ | ---------- | ------------ | --------- | --- | ------- | ------ | ------------ | --- | ------- |
| 4V, providing | insights |     | for developing | more | robust | and trust- |              |           |     |         |        |              |     |         |
“Improvingfastadversarialtrainingwithprior-guidedknowledge,”IEEE
worthy VLMs. However, our attack methods can be abused to TransactionsonPatternAnalysisandMachineIntelligence,2024.
evade actual deployed commercial systems, causing potential [19] N. Aafaq, N. Akhtar, W. Liu, M. Shah, and A. Mian, “Language
modelagnosticgray-boxadversarialattackonimagecaptioning,”IEEE
| negative | social impacts. |        | For example, | criminals |        | may use our |              |     |                |           |     |           |      |              |
| -------- | --------------- | ------ | ------------ | --------- | ------ | ----------- | ------------ | --- | -------------- | --------- | --- | --------- | ---- | ------------ |
|          |                 |        |              |           |        |             | Transactions |     | on Information | Forensics | and | Security, | vol. | 18, pp. 626– |
| methods  | to cause        | GPT-4V | APIs         | to output | target | responses,  |              |     |                |           |     |           |      |              |
638,2022.
[20] Y.Xu,B.Wu,F.Shen,Y.Fan,Y.Zhang,H.T.Shen,andW.Liu,“Exact
| causing | serious | harm. |     |     |     |     |             |        |     |                  |     |                |        |          |
| ------- | ------- | ----- | --- | --- | --- | --- | ----------- | ------ | --- | ---------------- | --- | -------------- | ------ | -------- |
|         |         |       |     |     |     |     | adversarial | attack | to  | image captioning |     | via structured | output | learning |
withlatentvariables,”inProceedingsoftheIEEE/CVFConferenceon
ComputerVisionandPatternRecognition,2019,pp.4135–4144.
REFERENCES
[21] R.LapidandM.Sipper,“Iseedeadpeople:Gray-boxadversarialattack
onimage-to-textmodels,”inProceedingsoftheEuropeanConferenceon
[1] J.Li,D.Li,C.Xiong,andS.Hoi,“Blip:Bootstrappinglanguage-image
MachineLearningandPrinciplesandPracticeofKnowledgeDiscovery
pre-trainingforunifiedvision-languageunderstandingandgeneration,”
inDatabases,2023.
| inInternationalConferenceonMachineLearning. |     |     |     |     | PMLR,2022,pp. |     |     |     |     |     |     |     |     |     |
| ------------------------------------------- | --- | --- | --- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
[22] A.E.Baia,V.Poggioni,andA.Cavallaro,“Black-boxattacksonimage
12888–12900. activitypredictionanditsnaturallanguageexplanations,”inProceedings
[2] J.Li,D.Li,S.Savarese,andS.Hoi,“BLIP-2:bootstrappinglanguage- of the IEEE/CVF International Conference on Computer Vision, 2023,
| image | pre-training | with | frozen image | encoders | and | large language |     |     |     |     |     |     |     |     |
| ----- | ------------ | ---- | ------------ | -------- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
pp.3686–3695.
| models,” | in International |     | Conference | on Machine | Learning, | 2023, pp. |              |          |     |               |          |     |          |            |
| -------- | ---------------- | --- | ---------- | ---------- | --------- | --------- | ------------ | -------- | --- | ------------- | -------- | --- | -------- | ---------- |
|          |                  |     |            |            |           |           | [23] L. Zhu, | T. Wang, | J.  | Li, Z. Zhang, | J. Shen, | and | X. Wang, | “Efficient |
1–13.
|     |     |     |     |     |     |     | query-based |     | black-box | attack against | cross-modal |     | hashing | retrieval,” |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | --------- | -------------- | ----------- | --- | ------- | ----------- |
[3] H.Liu,C.Li,Q.Wu,andY.J.Lee,“Visualinstructiontuning,”inThirty- ACM Transactions on Information Systems, vol. 41, no. 3, pp. 1–25,
| seventh | Conference | on  | Neural Information |     | Processing | Systems, 2023. | 2023. |     |     |     |     |     |     |     |
| ------- | ---------- | --- | ------------------ | --- | ---------- | -------------- | ----- | --- | --- | --- | --- | --- | --- | --- |
[Online].Available:https://openreview.net/forum?id=w0H2xGHlkw
[24] P.N.WilliamsandK.Li,“Black-boxsparseadversarialattackviamulti-
[4] R.Rombach,A.Blattmann,D.Lorenz,P.Esser,andB.Ommer,“High-
objectiveoptimisation,”inProceedingsoftheIEEE/CVFConferenceon
resolutionimagesynthesiswithlatentdiffusionmodels,”inProceedings
ComputerVisionandPatternRecognition,2023,pp.12291–12301.
oftheIEEE/CVFConferenceonComputerVisionandPatternRecogni- [25] H.Zhu,X.Sui,Y.Ren,Y.Jia,andL.Zhang,“Boostingtransferabilityof
tion,2022,pp.10684–10695. targetedadversarialexampleswithnon-robustfeaturealignment,”Expert
[5] F.Bao,S.Nie,K.Xue,C.Li,S.Pu,Y.Wang,G.Yue,Y.Cao,H.Su,and
SystemswithApplications,vol.227,p.120248,2023.
| J. Zhu, | “One transformer |     | fits all distributions |     | in multi-modal | diffusion |     |     |     |     |     |     |     |     |
| ------- | ---------------- | --- | ---------------------- | --- | -------------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
[26] H.Chen,Y.Zhang,Y.Dong,andJ.Zhu,“Rethinkingmodelensemble
atscale,”inInternationalConferenceonMachineLearning,2023,pp.
|     |     |     |     |     |     |     | in transfer-based |     | adversarial | attacks,” |     | in The Twelfth |     | International |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ----------- | --------- | --- | -------------- | --- | ------------- |
1692–1717. Conference on Learning Representations, 2024. [Online]. Available:
[6] S. H. Vemprala, R. Bonatti, A. Bucker, and A. Kapoor, “Chatgpt for https://openreview.net/forum?id=AcJrSoArlh
robotics:Designprinciplesandmodelabilities,”IEEEAccess,2024.
|              |          |                 |                 |                 |            |               | [27] Y. Xiong, | J.                | Lin, M.  | Zhang, J.      | E. Hopcroft, | and          | K. He,     | “Stochastic |
| ------------ | -------- | --------------- | --------------- | --------------- | ---------- | ------------- | -------------- | ----------------- | -------- | -------------- | ------------ | ------------ | ---------- | ----------- |
| [7] G. Liao, | J. Li,   | and X.          | Ye, “Vlm2scene: | Self-supervised |            | image-text-   |                |                   |          |                |              |              |            |             |
|              |          |                 |                 |                 |            |               | variance       | reduced           | ensemble | adversarial    |              | attack for   | boosting   | the adver-  |
| lidar        | learning | with foundation | models          | for             | autonomous | driving scene |                |                   |          |                |              |              |            |             |
|              |          |                 |                 |                 |            |               | sarial         | transferability,” |          | in Proceedings | of           | the IEEE/CVF | Conference | on          |
understanding,” in Proceedings of the AAAI Conference on Artificial ComputerVisionandPatternRecognition,2022,pp.14983–14992.
Intelligence,vol.38,no.4,2024,pp.3351–3359. [28] X. Wang, Z. Zhang, and J. Zhang, “Structure invariant transformation
[8] C. Cui, Y. Ma, X. Cao, W. Ye, Y. Zhou, K. Liang, J. Chen, J. Lu, for better adversarial transferability,” in Proceedings of the IEEE/CVF
| Z. Yang, | K.-D. | Liao | et al., “A survey | on multimodal |     | large language |     |     |     |     |     |     |     |     |
| -------- | ----- | ---- | ----------------- | ------------- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
InternationalConferenceonComputerVision,2023,pp.4607–4619.
| models | for autonomous |     | driving,” | in Proceedings | of  | the IEEE/CVF |               |     |        |             |     |               |     |           |
| ------ | -------------- | --- | --------- | -------------- | --- | ------------ | ------------- | --- | ------ | ----------- | --- | ------------- | --- | --------- |
|        |                |     |           |                |     |              | [29] Y. Wang, | Y.  | Wu, S. | Wu, X. Liu, | W.  | Zhou, L. Zhu, | and | C. Zhang, |
WinterConferenceonApplicationsofComputerVision,2024,pp.958– “Boostingthetransferabilityofadversarialattackswithfrequency-aware
979. perturbation,”IEEETransactionsonInformationForensicsandSecurity,
[9] P.Nagesh,B.Prabha,S.B.Gole,G.Rao,andN.V.Ramana,“Visual vol.19,pp.6293–6304,2024.
| assistance | for | visually | impaired people | using | image | caption and text |     |     |     |     |     |     |     |     |
| ---------- | --- | -------- | --------------- | ----- | ----- | ---------------- | --- | --- | --- | --- | --- | --- | --- | --- |
[30] Y.Song,R.Shu,N.Kushman,andS.Ermon,“Constructingunrestricted
to speech,” in AIP Conference Proceedings, vol. 2512, no. 1. AIP Advances in Neural
|     |     |     |     |     |     |     | adversarial | examples |     | with generative | models,” | in  |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------- | -------- | --- | --------------- | -------- | --- | --- | --- |
Publishing,2024. InformationProcessingSystems,vol.31,2018,pp.8322–8333.
[10] W.Wang,J.Huang,J.-t.Huang,C.Chen,J.Gu,P.He,andM.R.Lyu, [31] Z. Zhao, D. Dua, and S. Singh, “Generating natural adversarial
“An image is worth a thousand toxic words: A metamorphic testing examples,” in International Conference on Learning Representations,
| framework | for | content | moderation | software,” | in 2023 | 38th IEEE/ACM |     |     |     |     |     |     |     |     |
| --------- | --- | ------- | ---------- | ---------- | ------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- |
2018.[Online].Available:https://openreview.net/forum?id=H1BLjgZCb
International Conference on Automated Software Engineering (ASE). [32] X.Chen,X.Gao,J.Zhao,K.Ye,andC.-Z.Xu,“Advdiffuser:Natural
IEEE,2023,pp.1339–1351. adversarialexamplesynthesiswithdiffusionmodels,”inProceedingsof
[11] D.Han,X.Jia,Y.Bai,J.Gu,Y.Liu,andX.Cao,“Ot-attack:Enhancing theIEEE/CVFInternationalConferenceonComputerVision,2023,pp.
| adversarialtransferabilityofvision-languagemodelsviaoptimaltrans- |     |     |     |     |     |     | 4562–4572. |     |     |     |     |     |     |     |
| ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
portoptimization,”arXivpreprintarXiv:2312.04403,2023.
|     |     |     |     |     |     |     | [33] A. S. | Shamsabadi, | R.  | Sanchez-Matilla, |     | and A. Cavallaro, |     | “Colorfool: |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ----------- | --- | ---------------- | --- | ----------------- | --- | ----------- |
[12] S.Gao,X.Jia,X.Ren,I.Tsang,andQ.Guo,“Boostingtransferabilityin Semantic adversarial colorization,” in Proceedings of the IEEE/CVF
vision-languageattacksviadiversificationalongtheintersectionregion Conference on Computer Vision and Pattern Recognition, 2020, pp.
| ofadversarialtrajectory,”arXivpreprintarXiv:2403.12445,2024. |     |     |     |     |     |     | 1151–1160. |     |     |     |     |     |     |     |
| ------------------------------------------------------------ | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
[13] X.Gu,X.Zheng,T.Pang,C.Du,Q.Liu,Y.Wang,J.Jiang,andM.Lin, [34] A.Madry,A.Makelov,L.Schmidt,D.Tsipras,andA.Vladu,“Towards
“Agentsmith:Asingleimagecanjailbreakonemillionmultimodalllm
|     |     |     |     |     |     |     | deep | learning | models | resistant | to adversarial | attacks,” | in  | International |
| --- | --- | --- | --- | --- | --- | --- | ---- | -------- | ------ | --------- | -------------- | --------- | --- | ------------- |
agentsexponentiallyfast,”arXivpreprintarXiv:2402.08567,2024. Conference on Learning Representations, 2018. [Online]. Available:
[14] J. Zheng, C. Lin, J. Sun, Z. Zhao, Q. Li, and C. Shen, “Physical 3d https://openreview.net/forum?id=rJzIBfZAb
adversarial attacks against monocular depth estimation in autonomous [35] J. Guo, J. Li, D. Li, A. M. H. Tiong, B. Li, D. Tao, and S. Hoi,
driving,” in Proceedings of the IEEE/CVF Conference on Computer “Fromimagestotextualprompts:Zero-shotvisualquestionanswering
VisionandPatternRecognition,2024,pp.24452–24461.
|     |     |     |     |     |     |     | with | frozen | large language | models,” | in  | Proceedings | of the | IEEE/CVF |
| --- | --- | --- | --- | --- | --- | --- | ---- | ------ | -------------- | -------- | --- | ----------- | ------ | -------- |
[15] M. T. West, S.-L. Tsang, J. S. Low, C. D. Hill, C. Leckie, L. C. Conference on Computer Vision and Pattern Recognition, 2023, pp.
| Hollenberg,S.M.Erfani,andM.Usman,“Towardsquantumenhanced |     |     |     |     |     |     | 10867–10877. |     |     |     |     |     |     |     |
| -------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
adversarial robustness in machine learning,” Nature Machine Intelli- [36] Y.Dong,F.Liao,T.Pang,H.Su,J.Zhu,X.Hu,andJ.Li,“Boosting
gence,vol.5,no.6,pp.581–589,2023. adversarialattackswithmomentum,”inProceedingsoftheIEEECon-
[16] Y. Zhao, T. Pang, C. Du, X. Yang, C. Li, N.-M. Cheung, and M. Lin, ferenceonComputerVisionandPatternRecognition,2018,pp.9185–
| “Onevaluatingadversarialrobustnessoflargevision-languagemodels,” |     |     |     |     |     |     | 9193. |     |     |     |     |     |     |     |
| ---------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
inThirty-seventhConferenceonNeuralInformationProcessingSystems, [37] Y. Long, Q. Zhang, B. Zeng, L. Gao, X. Liu, J. Zhang, and J. Song,
2023. “Frequency domain model augmentation for adversarial attack,” in

| IEEETRANSACTIONSONINFORMATIONFORENSICSANDSECURITY |     |     |     |     |     |     |     |     |     |     |     |     |     |     | 15  |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
European Conference on Computer Vision. Springer, 2022, pp. 549– [60] R. R. Selvaraju, M. Cogswell, A. Das, R. Vedantam, D. Parikh, and
566. D. Batra, “Grad-cam: Visual explanations from deep networks via
[38] A. Hyva¨rinen and P. Dayan, “Estimation of non-normalized statistical gradient-based localization,” in Proceedings of the IEEE International
models by score matching.” Journal of Machine Learning Research, ConferenceonComputerVision,2017,pp.618–626.
vol.6,no.4,2005. [61] Z. Wang, A. C. Bovik, H. R. Sheikh, and E. P. Simoncelli, “Image
[39] Y. Song and S. Ermon, “Generative modeling by estimating gradients quality assessment: from error visibility to structural similarity,” IEEE
transactionsonimageprocessing,vol.13,no.4,pp.600–612,2004.
| of the | data | distribution,” | Advances | in  | neural | information | processing |     |     |     |     |     |     |     |     |
| ------ | ---- | -------------- | -------- | --- | ------ | ----------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
systems,vol.32,2019. [62] M.Heusel,H.Ramsauer,T.Unterthiner,B.Nessler,andS.Hochreiter,
[40] Y. Song, J. Sohl-Dickstein, D. P. Kingma, A. Kumar, S. Ermon, and “Ganstrainedbyatwotime-scaleupdateruleconvergetoalocalnash
B. Poole, “Score-based generative modeling through stochastic differ- equilibrium,” in Advances in Neural Information Processing Systems,
entialequations,”arXivpreprintarXiv:2011.13456,2020. vol.30,2017,pp.6629–6640.
|     |     |     |     |     |     |     |     | [63] R. Zhang, | P.  | Isola, | A. A. Efros, | E. Shechtman, | and | O. Wang, | “The |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | ------ | ------------ | ------------- | --- | -------- | ---- |
[41] T.Brown,B.Mann,N.Ryder,M.Subbiah,J.D.Kaplan,P.Dhariwal,
A.Neelakantan,P.Shyam,G.Sastry,A.Askelletal.,“Languagemod- unreasonable effectiveness of deep features as a perceptual metric,” in
els are few-shot learners,” Advances in neural information processing Proceedings of the IEEE Conference on Computer Vision and Pattern
systems,vol.33,pp.1877–1901,2020. Recognition,2018,pp.586–595.
[42] C. Raffel, N. Shazeer, A. Roberts, K. Lee, S. Narang, M. Matena, [64] A.Mittal,A.K.Moorthy,andA.C.Bovik,“Blind/referencelessimage
|          |     |         |       |                 |     |            |             | spatial | quality | evaluator,” | in 2011 | conference | record | of the | forty fifth |
| -------- | --- | ------- | ----- | --------------- | --- | ---------- | ----------- | ------- | ------- | ----------- | ------- | ---------- | ------ | ------ | ----------- |
| Y. Zhou, | W.  | Li, and | P. J. | Liu, “Exploring |     | the limits | of transfer |         |         |             |         |            |        |        |             |
learningwithaunifiedtext-to-texttransformer,”TheJournalofMachine asilomar conference on signals, systems and computers (ASILOMAR).
LearningResearch,vol.21,no.1,pp.5485–5551,2020. IEEE,2011,pp.723–727.
[43] W.-L. Chiang, Z. Li, Z. Lin, Y. Sheng, Z. Wu, H. Zhang, L. Zheng, [65] X. Jia, Y. Zhang, X. Wei, B. Wu, K. Ma, J. Wang, and X. Cao,
“Improvingfastadversarialtrainingwithprior-guidedknowledge,”IEEE
S.Zhuang,Y.Zhuang,J.E.Gonzalezetal.,“Vicuna:Anopen-source
TransactionsonPatternAnalysisandMachineIntelligence,2024.
chatbotimpressinggpt-4with90%*chatgptquality,”Seehttps://vicuna.
[66] W.Xu,D.Evans,andY.Qi,“Featuresqueezing:Detectingadversarial
lmsys.org(accessed14April2023),2023.
[44] S.Yin,C.Fu,S.Zhao,K.Li,X.Sun,T.Xu,andE.Chen,“Asurveyon examples in deep neural networks,” arXiv preprint arXiv:1704.01155,
| multimodal |     | large language | models,” |     | arXiv preprint | arXiv:2306.13549, |     | 2017.        |       |          |             |            |              |     |            |
| ---------- | --- | -------------- | -------- | --- | -------------- | ----------------- | --- | ------------ | ----- | -------- | ----------- | ---------- | ------------ | --- | ---------- |
|            |     |                |          |     |                |                   |     | [67] B. Sun, | N.-h. | Tsai, F. | Liu, R. Yu, | and H. Su, | “Adversarial |     | defense by |
2023.
stratifiedconvolutionalsparsecoding,”inProceedingsoftheIEEE/CVF
| [45] J. Wu, | W. Gan, | Z. Chen, | S. Wan, | and | S. Y. Philip, | “Multimodal | large |            |     |             |            |         |              |     |           |
| ----------- | ------- | -------- | ------- | --- | ------------- | ----------- | ----- | ---------- | --- | ----------- | ---------- | ------- | ------------ | --- | --------- |
|             |         |          |         |     |               |             |       | conference |     | on computer | vision and | pattern | recognition, |     | 2019, pp. |
languagemodels:Asurvey,”in2023IEEEInternationalConferenceon
| BigData(BigData). |     |     |     |     |     |     |     | 11447–11456. |     |     |     |     |     |     |     |
| ----------------- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
IEEE,2023,pp.2247–2256.
[46] J.-B.Alayrac,J.Donahue,P.Luc,A.Miech,I.Barr,Y.Hasson,K.Lenc, [68] G. K. Dziugaite, Z. Ghahramani, and D. M. Roy, “A study of the
|     |         |              |     |          |         |            |          | effect | of jpg | compression | on adversarial |     | images,” | arXiv | preprint |
| --- | ------- | ------------ | --- | -------- | ------- | ---------- | -------- | ------ | ------ | ----------- | -------------- | --- | -------- | ----- | -------- |
| A.  | Mensch, | K. Millican, | M.  | Reynolds | et al., | “Flamingo: | a visual |        |        |             |                |     |          |       |          |
arXiv:1608.00853,2016.
languagemodelforfew-shotlearning,”Advancesinneuralinformation
|     |     |     |     |     |     |     |     | [69] C.-H. | Ho and | N. Vasconcelos, | “Disco: | Adversarial |     | defense | with local |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | ------ | --------------- | ------- | ----------- | --- | ------- | ---------- |
processingsystems,vol.35,pp.23716–23736,2022.
implicitfunctions,”AdvancesinNeuralInformationProcessingSystems,
| [47] D. | Zhu, J. | Chen, X. | Shen, | X. Li, | and M. | Elhoseiny, | “Minigpt-4: |     |     |     |     |     |     |     |     |
| ------- | ------- | -------- | ----- | ------ | ------ | ---------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
Enhancingvision-languageunderstandingwithadvancedlargelanguage vol.35,pp.23818–23837,2022.
|     |     |     |     |     |     |     |     | [70] W. Nie, | B. Guo, | Y.  | Huang, C. Xiao, | A. Vahdat, | and | A. Anandkumar, |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | ------- | --- | --------------- | ---------- | --- | -------------- | --- |
models,”arXivpreprintarXiv:2304.10592,2023.
“Diffusionmodelsforadversarialpurification,”inInternationalConfer-
[48] J.C.Costa,T.Roxo,H.Proenc¸a,andP.R.Ina´cio,“Howdeeplearning
enceonMachineLearning,2022,pp.1–23.
| sees | the world: | A survey | on  | adversarial | attacks | & defenses,” | IEEE |     |     |     |     |     |     |     |     |
| ---- | ---------- | -------- | --- | ----------- | ------- | ------------ | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
[71] Y.ZhuandY.Jiang,“Anon-globaldisturbancetargetedadversarialex-
Access,2024.
amplealgorithmcombinedwithc&wandgrad-cam,”NeuralComputing
[49] S.Han,C.Lin,C.Shen,Q.Wang,andX.Guan,“Interpretingadversarial andApplications,vol.35,no.29,pp.21633–21644,2023.
examplesindeeplearning:Areview,”ACMComputingSurveys,vol.55,
|     |     |     |     |     |     |     |     | [72] M. Yoshida, |     | H. Namura, | and M. | Okuda, | “Adversarial | examples | for |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | ---------- | ------ | ------ | ------------ | -------- | --- |
no.14s,pp.1–38,2023.
imagecropping:Gradient-basedandbayesian-optimizedapproachesfor
| [50] J. Gu, | X. Jia, | P. de | Jorge, W. | Yu, | X. Liu, A. | Ma, Y. | Xun, A. Hu, |     |     |     |     |     |     |     |     |
| ----------- | ------- | ----- | --------- | --- | ---------- | ------ | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
effectiveadversarialattack,”IEEEAccess,2024.
A.Khakzar,Z.Lietal.,“Asurveyontransferabilityofadversarialex-
[73] B.Zhou,A.Khosla,A.Lapedriza,A.Oliva,andA.Torralba,“Learning
amplesacrossdeepneuralnetworks,”arXivpreprintarXiv:2310.17626, deepfeaturesfordiscriminativelocalization,”inProceedingsoftheIEEE
2023.
conferenceoncomputervisionandpatternrecognition,2016,pp.2921–
| [51] X. | Xu, X. | Chen, C. | Liu, A. | Rohrbach, | T.  | Darrell, | and D. Song, |     |     |     |     |     |     |     |     |
| ------- | ------ | -------- | ------- | --------- | --- | -------- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
2929.
“Foolingvisionandlanguagemodelsdespitelocalizationandattention
|     |     |     |     |     |     |     |     | [74] A. Chattopadhay, |     | A.  | Sarkar, P. Howlader, | and | V. N. | Balasubramanian, |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------------- | --- | --- | -------------------- | --- | ----- | ---------------- | --- |
mechanism,”inProceedingsoftheIEEEConferenceonComputerVision
“Grad-cam++:Generalizedgradient-basedvisualexplanationsfordeep
andPatternRecognition,2018,pp.4951–4961. convolutional networks,” in 2018 IEEE winter conference on applica-
[52] Y.Dong,H.Chen,J.Chen,Z.Fang,X.Yang,Y.Zhang,Y.Tian,H.Su, tionsofcomputervision(WACV). IEEE,2018,pp.839–847.
andJ.Zhu,“Howrobustisgoogle’sbardtoadversarialimageattacks?”
[75] P.-T.Jiang,C.-B.Zhang,Q.Hou,M.-M.Cheng,andY.Wei,“Layercam:
arXivpreprintarXiv:2309.11751,2023.
|     |     |     |     |     |     |     |     | Exploring | hierarchical |     | class activation | maps | for localization,” |     | IEEE |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ------------ | --- | ---------------- | ---- | ------------------ | --- | ---- |
[53] Q. Li, Q. Hu, H. Fan, C. Lin, C. Shen, and L. Wu, “Attention-sa: TransactionsonImageProcessing,vol.30,pp.5875–5888,2021.
| Exploiting | model-approximated |     |                | data      | semantics | for adversarial | attack,” |     |     |     |     |     |     |     |     |
| ---------- | ------------------ | --- | -------------- | --------- | --------- | --------------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
| IEEE       | Transactions       |     | on Information | Forensics |           | and Security,   | pp. 1–1, |     |     |     |     |     |     |     |     |
2024.
[54] D.-T.Peng,J.Dong,M.Zhang,J.Yang,andZ.Wang,“Csfadv:Critical
| semantic | fusion | guided | least-effort | adversarial |     | example | attacks,” IEEE |     |     |     |     |     |     |     |     |
| -------- | ------ | ------ | ------------ | ----------- | --- | ------- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
TransactionsonInformationForensicsandSecurity,vol.19,pp.5940–
5955,2024.
[55] J.Ho,A.Jain,andP.Abbeel,“Denoisingdiffusionprobabilisticmodels,”
AdvancesinNeuralInformationProcessingSystems,vol.33,pp.6840–
6851,2020.
[56] J.Song,C.Meng,andS.Ermon,“Denoisingdiffusionimplicitmodels,”
| in  | International | Conference |     | on Learning |     | Representations, | 2021. |     |     |     |     |     |     |     |     |
| --- | ------------- | ---------- | --- | ----------- | --- | ---------------- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
[Online].Available:https://openreview.net/forum?id=St1giarCHLP
[57] C.Luo,“Understandingdiffusionmodels:Aunifiedperspective,”arXiv
preprintarXiv:2208.11970,2022.
| [58] A. Radford, |     | J. W. Kim, | C. Hallacy, | A.  | Ramesh, | G. Goh, | S. Agarwal, |     |     |     |     |     |     |     |     |
| ---------------- | --- | ---------- | ----------- | --- | ------- | ------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
G.Sastry,A.Askell,P.Mishkin,J.Clarketal.,“Learningtransferable
| visual                       | models       | from     | natural        | language                | supervision,”   | in       | International |     |     |     |     |     |     |     |     |
| ---------------------------- | ------------ | -------- | -------------- | ----------------------- | --------------- | -------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- |
| ConferenceonMachineLearning. |              |          |                | PMLR,2021,pp.8748–8763. |                 |          |               |     |     |     |     |     |     |     |     |
| [59] Z. Cai,                 | Y.           | Tan, and | M. S. Asif,    | “Ensemble-based         |                 | blackbox | attacks       |     |     |     |     |     |     |     |     |
| on dense                     | prediction,” |          | in Proceedings |                         | of the IEEE/CVF |          | Conference    | on  |     |     |     |     |     |     |     |
ComputerVisionandPatternRecognition,2023,pp.4045–4055.
