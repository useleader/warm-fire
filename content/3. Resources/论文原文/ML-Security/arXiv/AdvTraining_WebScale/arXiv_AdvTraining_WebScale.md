---
publish: true
---

|     |     |     |           | Revisiting |             | Adversarial |     | Training  | at Scale  |     |     |
| --- | --- | --- | --------- | ---------- | ----------- | ----------- | --- | --------- | --------- | --- | --- |
|     |     |     | ZeyuWang* |            | XianhangLi* |             |     | HongruZhu | CihangXie |     |     |
∗equalcontribution
UCSantaCruz
4202 rpA 12  ]VC.sc[  2v72740.1042:viXra
Abstract
AdvXL
Themachinelearningcommunityhaswitnessedadras-
ticchangeinthetrainingpipeline,pivotedbythose“foun-
Previous
1000(cid:1)
| dation models” |     | with unprecedented |     | scales. |     | However, | the |     |     |     |     |
| -------------- | --- | ------------------ | --- | ------- | --- | -------- | --- | --- | --- | --- | --- |
Data
| field of        | adversarial        | training | is       | lagging     | behind,        | predomi-   |     |     |     |     |     |
| --------------- | ------------------ | -------- | -------- | ----------- | -------------- | ---------- | --- | --- | --- | --- | --- |
| nantly centered |                    | around   | small    | model sizes | like           | ResNet-50, |     |     |     |     |     |
| and tiny        | and low-resolution |          | datasets |             | like CIFAR-10. |            | To  |     |     |     |     |
bridgethistransformationgap,thispaperprovidesamod-
AdvXL
ernre-examinationwithadversarialtraining,investigating
Previous
itspotentialbenefitswhenappliedatscale.Additionally,we
5(cid:1)
| introduce | an efficient | and | effective | training | strategy |     | to en- |     |     |     |     |
| --------- | ------------ | --- | --------- | -------- | -------- | --- | ------ | --- | --- | --- | --- |
Model
ableadversarialtrainingwithgiantmodelsandweb-scale
| dataatanaffordablecomputingcost. |         |             |     | Wedenotethisnewly |       |             |     |     |                     |     |     |
| -------------------------------- | ------- | ----------- | --- | ----------------- | ----- | ----------- | --- | --- | ------------------- | --- | --- |
| introducedframeworkasAdvXL.      |         |             |     |                   |       |             |     |     | (a)Scalecomparison. |     |     |
| Empirical                        | results | demonstrate |     | that              | AdvXL | establishes |     |     |                     |     |     |
AdvXL
| new state-of-the-art |                 | robust | accuracy |          | records | under    | Au- |     | +11.4% |        |            |
| -------------------- | --------------- | ------ | -------- | -------- | ------- | -------- | --- | --- | ------ | ------ | ---------- |
|                      |                 |        |          |          |         |          |     |     |        | +14.2% | Prior Best |
| toAttack             | on ImageNet-1K. |        | For      | example, | by      | training | on  |     |        |        |            |
DataComp-1Bdataset,ourAdvXLempowersavanillaViT-
| gmodeltosubstantiallysurpassthepreviousrecordsofl |     |     |     |     |     |     | ∞ - |     |     |     |     |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
, l -, and l -robust accuracy by margins of 11.4%, 14.2% +12.9%
| 2                      | 1         |           |                              |                 |            |     |         |     |     |     |     |
| ---------------------- | --------- | --------- | ---------------------------- | --------------- | ---------- | --- | ------- | --- | --- | --- | --- |
| and12.9%,respectively. |           |           | ThisachievementpositsAdvXLas |                 |            |     |         |     |     |     |     |
| a pioneering           | approach, |           | charting                     | a new           | trajectory |     | for the |     |     |     |     |
| efficient              | training  | of robust | visual                       | representations |            | at  | signif- |     |     |     |     |
https:
| icantly larger | scales. | Our | code | is available |     | at  |     |     |     |     |     |
| -------------- | ------- | --- | ---- | ------------ | --- | --- | --- | --- | --- | --- | --- |
//github.com/UCSC-VLAA/AdvXL.
(b)Performancecomparison.
|     |     |     |     |     |     |     |     | Figure 1.              | Our AdvXL increases              | significantly | in terms of both |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------------- | -------------------------------- | ------------- | ---------------- |
|     |     |     |     |     |     |     |     | modelsizeanddatascale, | whichbringsasubstantialboostover |               |                  |
1.Introduction
|     |     |     |     |     |     |     |     | prior best                           | results of l , l , | and l robustness | on ImageNet-1K, |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------ | ------------------ | ---------------- | --------------- |
|     |     |     |     |     |     |     |     |                                      | ∞ 2                | 1                |                 |
|     |     |     |     |     |     |     |     | eventhoughourmodelisonlytrainedtobel |                    |                  | ∞ -robust.      |
Thelandscapeofmachinelearning,particularlydeeplearn-
ing,haswitnessedatransformativeshiftwiththeadventof However,amidstthisevolution,adversarialtraining[19,
large-scale models and datasets. This paradigmatic shift, 39]—apivotalstrategyaimedatsecuringmodelrobustness
exemplified by the inception of “foundation models” such against adversarial attacks — has faced significant scala-
asLargeLanguageModels(LLMs)[6,14,41,55,56],has bility challenges in this foundation model era. Adversar-
redefined the boundaries of what is achievable in various ial training, typically employed in small models such as
domainsofartificialintelligence.Excitingly,paralleldevel- ResNet-50 [23] trained on small datasets like CIFAR-10
opmentshavealsobeenobservedincomputervision—re- [28], involves repeatedly generating adversarial examples
centadvancementsinscalingdatasetsandmodelsizeshave throughon-the-flyattacksduringthetrainingprocess. This
mirroredthefeasibilityof“LLM-like”scalingforbuilding iterativeandintensiveproceduredemandssubstantialcom-
exceptionallystrongvisualrecognitionmodels[12,16,64]. putationalresources,thusmakingitchallengingtoscaleup.

Contrasting with these challenges, recent endeavors in data augmentation, and training duration on model robust-
adversarialtraininghaveindeedshownintriguingglimpses ness, predominantly on smaller datasets like CIFAR-10
of promise from data scaling by incorporating 50 mil- [7,20,25,33,40,42]. Otherresearcheffortshaveexplored
lionadditionalimagestosustainstate-of-the-artrobustness deeper nuances of adversarial training recipes tailored for
records on CIFAR-10 [57]. Additionally, other adversar- ImageNet-1K [3, 11, 49, 52, 60–62]. Recent works also
ial training works [34, 52] attain impressive performance investigatetherobustnessofnovelnetworkdesignslikeVi-
with model scaling using larger models like Swin-L [35] sionTransformer(ViT)[9,17,21,52]. Inparticular,Singh
and ConvNeXt-L [37] on ImageNet-1K. These observa- et al. [52] achieve the best generalized robustness by en-
tions, coupled with the burgeoning success of foundation hancingViTandConvNeXTwithConvolutionalStem.
models, instigatesacriticalquestion: cantheprinciplesof Despite its effectiveness, adversarial training is notori-
modelanddatascaling,alreadyproveneffectiveinvanilla ouslyresource-intensive,limitingitsscalability. Toaddress
training,betransferabletoadversarialtraining?Moreover, this challenge, researchers have pursued more resource-
howeffectivelydoessuchscalingtranslatetorobustnessim- efficient adversarial training methodologies. Examples in-
provementinadversarialtraining? clude Free Adversarial Training [51] and Fast Adversar-
In response to these questions, we re-examine adver- ial Training [58], both aimed at reducing training costs
sarial training at a previously uncharted foundation-model while preserving model robustness. However, these ap-
scale. In terms of model scaling, we increased the model proacheshavepredominantlyfocusedonsmallernetworks
parameters from the previously largest 200M size to 1B; and datasets, leaving a noticeable gap concerning large-
for data scaling, we adversarially train models on vari- scale models. In this work, we aim to significantly ex-
ousdatasetsspanningfromthemedium-sizeImageNet-1K pandthehorizonsofscalingadversarialtrainingtounprece-
with around 1M images to the web-scale dataset compris- dentedlevelsofefficiencyandeffectiveness.
ing more than 1B images. Additionally, to make the scal-
2.2.ScalingVisionFoundationModels
ing of adversarial training computationally affordable, we
introduceanefficientapproachwithastraightforwardtwo- Paralleltolarge-scalelanguagemodels,exemplifiedbyin-
stage training schedule, i.e., first lightweight pre-training, novations like GPT series [41], similar efforts have been
thenintensivefine-tuning. Wenamethisefficientandscal- made for vision models, particularly with the scaling of
ableadversarialtrainingframeworkasAdvXL. ViTs [12, 16, 64]. Liu et al. [36] effectively trained the
Collectively, extensive experiments showcase that these SwinV2-G model, housing an astounding 3B parameters,
scaling endeavors successfully result in substantial im- by employing residual-post-norm and scaled cosine atten-
provements over the previous state-of-the-art methods on tion. Similarly, Dehghani et al. [12] have shown substan-
adversarial robustness. For example, by training a one- tialperformanceenhancementsbyscalingViTsto22Bpa-
billion parameter model on a one-billion image dataset, rameters,mirroringthescalingtrendswitnessedinlanguage
we establish a new state-of-the-art record for l -robust models.
∞
accuracy of 71.0% under AutoAttack on ImageNet-1K, Despitetheburgeoningscalingeffortsinvisionfounda-
marking a substantial enhancement in model robustness. tionmodels,theexplorationofadversarialtraininghastra-
Notably,AdvXLdemonstratesexceptionalgeneralizability ditionallybeenlimitedtosmallorbasemodelsizes. Recent
whentestedagainstunseenattacks,improvinguponthepre- scaling effort has led to noteworthy performance improve-
vious best l - and l -robust accuracy of models trained to ments,evidencedbytheachievementsonRobustBench[9]
2 1
bel
∞
-robustbymarginsof∼14%and∼13%,respectively. withlargermodelslikeSwin-LandConvNeXt-L[34,52].
These results underscore the pivotal role of (significantly) Diverging from these antecedent initiatives, our work ex-
scaled adversarial training in enhancing model robustness ploresadversarialtrainingatanevenmuchlargerscale,up
againstdiverseadversarialthreats. to the training of a one-billion-parameter model on one-
billion samples, thereby pioneering the frontiers of adver-
2.RelatedWork sarialtrainingintounchartedterritory.
2.1.AdversarialTraining 3.AdvXL
Adversarialtraininghasemergedasapivotaldefensemech- In this section, we introduce AdvXL, a novel training
anismagainstadversarialattacksinmachinelearning. Ini- framework designed for adversarially robust visual repre-
tially introduced by Goodfellow et al. [19], this method- sentation learning at scale. We first revisit the fundamen-
ology involves training models on crafted adversarial ex- tal concept of adversarial attacks and adversarial training
amples designed to provoke model misclassification. Sub- in Sec. 3.1. Following this, in Sec. 3.2, we present a two-
sequent studies have extended this foundation, examin- stageefficientadversarialtrainingpipelinecharacterizedby
ing facets such as the impact of batch size, learning rate, a coarse-to-fine, weak-to-strong approach. In Sec. 3.3, we

showcasehowtoleverageCLIP[47]textencoderasatool Coarse-to-finetraining. Wefirstexplorevariousstrategies
for enabling us to learn with web-crawled images, where for image token reduction in the initial pre-training stage.
apreciselabelisusuallymissingbutwithacorresponding Following [30, 31], three distinct approaches are investi-
| textdescription,forscaledadversarialtraining. |     |     |     |     |     |     |     | gated:           |     |                                  |     |     |     |     |
| --------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | -------------------------------- | --- | --- | --- | --- |
|                                               |     |     |     |     |     |     |     | • RandomMasking. |     | Thismethod,asdescribedin[24,32], |     |     |     |     |
3.1.AdversarialTraining involvesdividinganimageintonon-overlappingpatches
|             |          |              |              |              |         |              |        | (e.g., 16×16), |               | subsequently | masking  |     | a random          | propor-   |
| ----------- | -------- | ------------ | ------------ | ------------ | ------- | ------------ | ------ | -------------- | ------------- | ------------ | -------- | --- | ----------------- | --------- |
| Adversarial | examples |              | are uniquely | crafted      |         | inputs that, | de-    |                |               |              |          |     |                   |           |
|             |          |              |              |              |         |              |        | tion of        | these patches | (e.g.,       | 75%).    | The | model             | only pro- |
| spite their | visual   | similarity   |              | to authentic | samples |              | within |                |               |              |          |     |                   |           |
|             |          |              |              |              |         |              |        | cesses the     | visible       | patches,     | reducing |     | the computational |           |
| specific    | norm     | constraints, | are          | engineered   |         | to deceive   | ma-    |                |               |              |          |     |                   |           |
costby50%or75%,dependingonthemaskingratio.
| chine  | learning | models   | into producing |         | inaccurate |           | predic- |                  |     |             |       |           |          |           |
| ------ | -------- | -------- | -------------- | ------- | ---------- | --------- | ------- | ---------------- | --- | ----------- | ----- | --------- | -------- | --------- |
|        |          |          |                |         |            |           |         | • Block Masking. |     | Inspired    | by    | [4], this | approach | retains   |
| tions. | These    | examples | play a         | crucial | role in    | assessing | the     |                  |     |             |       |           |          |           |
|        |          |          |                |         |            |           |         | tokens from      | a   | consecutive | large | block     | within   | the image |
robustnessofamodelinscenarioswheremaliciousmanip-
|     |     |     |     |     |     |     |     | whilediscardingothers. |     |     | Thismethodleveragesthecom- |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------------- | --- | --- | -------------------------- | --- | --- | --- |
ulationsmayoccur.
|     |     |     |     |     |     |     |     | mon placement |     | of objects | in  | the central | regions | of im- |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ---------- | --- | ----------- | ------- | ------ |
Adversarial Training is central to fortifying a model ages, potentially preserving semantic meaningful tokens
| against | such | adversarial | inputs. | This | technique | involves | a   |     |     |     |     |     |     |     |
| ------- | ---- | ----------- | ------- | ---- | --------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
whilesignificantlyreducingthecomputationalcostfrom
| strategic | training | process | designed | to  | enhance | the | model’s |     |     |     |     |     |     |     |
| --------- | -------- | ------- | -------- | --- | ------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
lengthyinputs.
| robustness | to  | adversarial | attacks. | The | mathematical |     | foun- | Resizing. |                                         |     |     |     |     |     |
| ---------- | --- | ----------- | -------- | --- | ------------ | --- | ----- | --------- | --------------------------------------- | --- | --- | --- | --- | --- |
|            |     |             |          |     |              |     |       | •         | Imageresizingisanothermethodforreducing |     |     |     |     |     |
dationofATisencapsulatedasanoptimizationproblem:
|     |     |     |     |     |     |     |     | the image | token | length. | Compared | to  | masking, | resizing |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ----- | ------- | -------- | --- | -------- | -------- |
retainsmoreimageinformation,especiallyhigh-levelse-
|     |     |     |     |     |     |     |     | mantics. | For | instance, | resizing | an image | to  | 112 × 112 |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | --------- | -------- | -------- | --- | --------- |
(cid:88)
min max L(f (x +δ),y ), (1) iscomputationallyakintoapplyinga75%maskingratio
|     |     |     |     | θ   | i   | i   |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
θ δ:∥δ∥p≤ϵp to an image resized to 224×224. In our approach, we
(xi,yi)∈D
|     |     |     |     |     |     |     |     | choose | anti-aliasing | bilinear | interpolation |     | to  | better pre- |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------------- | -------- | ------------- | --- | --- | ----------- |
whereθrepresentstheparametersforanetworkf .Theob- servetheimagequality.
θ
| jectiveistotrainthenetworkf |     |     |     | suchthatitmaintainscon- |     |     |     |                     |     |              |     |             |       |        |
| --------------------------- | --- | --- | --- | ----------------------- | --- | --- | --- | ------------------- | --- | ------------ | --- | ----------- | ----- | ------ |
|                             |     |     |     | θ                       |     |     |     | A visual comparison |     | illustrating |     | these image | token | reduc- |
sistent predictions under adversarial perturbations δ, i.e., tion strategies is presented in Fig. 2. These strategies are
| within | an l -ball | of radius | ϵ   | centered | around | each | input |                                                       |     |     |     |     |     |     |
| ------ | ---------- | --------- | --- | -------- | ------ | ---- | ----- | ----------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|        | p          |           | p   |          |        |      |       | evaluatedtodiscerntheirefficacyinachievingtrainingac- |     |     |     |     |     |     |
samplex
|     | i . |     |     |     |     |     |     | celerationwhileretainingcriticalimagesemantics. |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- | --- | --- |
AdversarialTraininghasprovenhighlyeffectivetosafe-
|               |        |          |             |            |     |           |     | Weak-to-strongtraining. |       |          | Anothercriticalfactorforaccel- |             |          |            |
| ------------- | ------ | -------- | ----------- | ---------- | --- | --------- | --- | ----------------------- | ----- | -------- | ------------------------------ | ----------- | -------- | ---------- |
| guard         | models | against  | adversarial | threats    | [5, | 19, 53].  | In  |                         |       |          |                                |             |          |            |
|               |        |          |             |            |     |           |     | erating adversarial     |       | training | involves                       | managing    |          | the number |
| our approach, |        | we adopt | the widely  | recognized |     | PGD-based |     |                         |       |          |                                |             |          |            |
|               |        |          |             |            |     |           |     | of gradient             | steps | used to  | craft                          | adversarial | samples. | Gen-       |
AdversarialTraining(PGD-AT)methodfortheinnermax-
erallyspeaking,increasingthenumberofgradientstepsre-
| imization | problem, | renowned |     | for its | robust | performance |     |     |     |     |     |     |     |     |
| --------- | -------- | -------- | --- | ------- | ------ | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
sultsinstrongerattacksandenhancesadversarialrobustness
| and computational |     | efficiency. |     | For the | outer | minimization |     |                |          |               |     |        |        |          |
| ----------------- | --- | ----------- | --- | ------- | ----- | ------------ | --- | -------------- | -------- | ------------- | --- | ------ | ------ | -------- |
|                   |     |             |     |         |       |              |     | but inevitably | inflates | computational |     | costs. | It has | been re- |
problem,wetypicallyemployoptimizationalgorithmslike
portedthatformingarobustnetworkwithadversarialtrain-
| Stochastic | Gradient | Descent | or  | AdamW | [38], | using | cross- |     |     |     |     |     |     |     |
| ---------- | -------- | ------- | --- | ----- | ----- | ----- | ------ | --- | --- | --- | --- | --- | --- | --- |
ingcantakesignificantlylonger,rangingfrom3to30times
entropyasthelossfunctionL.
|     |     |     |     |     |     |     |     | more than                                             | building | a non-robust |     | equivalent | [51]. | As a re- |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------- | -------- | ------------ | --- | ---------- | ----- | -------- |
|     |     |     |     |     |     |     |     | sult, previousstudies[44,51,59]haveproposedstrategies |          |              |     |            |       |          |
3.2.Two-stageTraining
|     |     |     |     |     |     |     |     | like recycling | gradient | information |     | or employing |     | a small |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | -------- | ----------- | --- | ------------ | --- | ------- |
Our adversarial training framework hinges on a two-stage generatornetworktomitigatethesignificantcomputational
process: a lightweight pre-training stage and an intensive burdeninadversarialtraining.
fine-tuningstage. Duringthepre-trainingstage, themodel Our exploration reveals that applying a small number
is trained with inputs at reduced token length and weaker of PGD steps (e.g., PGD-1) during the pre-training stage
attacks,spanningarelativelyextendedduration. Then,dur- and subsequently increasing these steps during the fine-
ing the subsequent fine-tuning stage, the model is trained tuningphase(e.g.,PGD-3)sufficientlysecurestrongrobust-
withinputsatfullresolutionandstrongerattacks,following ness, i.e., thismethodproveseffectivecomparedtoinitiat-
acomparativelyshorterschedule. Comparedtothevanilla ingtrainingwithstrongattacks. Importantly,thisapproach
one-stage adversarial training pipeline, this coarse-to-fine contributesanotableadditionalspeedup,enhancingtheeffi-
(w.r.t. input), weak-to-strong (w.r.t. adversarial attacker), ciencygainedfromthecoarse-to-finetrainingpipeline(e.g.,
two-stage training pipeline significantly reduces the over- up to 2×), as solving the inner optimization of adversarial
alltrainingcost,renderingitcomputationallyaffordablefor trainingoftenrequiresoptimizationwithmultipleiterations
| furtherscalingup. |     |     |     |     |     |     |     | andisextremelytime-consuming. |     |     |     |     |     |     |
| ----------------- | --- | --- | --- | --- | --- | --- | --- | ----------------------------- | --- | --- | --- | --- | --- | --- |

(a) Original (b) Random Masking (c) Block Masking (d) Resizing
Figure2.Illustrationofdifferentapproachestoimagetokenreduction.
Fine-tuning. Echoing findings from prior research [31, optimizationproblem,
32], we find that further adversarially training our model min (cid:88) max L (cid:0) fI,fT,I +δ,T (cid:1) , (3)
with full-resolution inputs and stronger attacks for a short i i
schedule yields considerable improvement and delivers a
θI
(Ii,Ti)∈D
δ:∥δ∥p≤ϵp
morefavorableaccuracy-to-timetrade-off.Comparedtothe where θI represents the parameters of the image encoder
pre-training stage, the fine-tuning phase is notably shorter, fI. To elucidate this integration further, Fig. 3 provides
often reduced by one or two orders of magnitude. There- a visual representation illustrating the incorporation of the
fore,eventhougheachsamplemayentailanotablyhigher CLIPencoderinadversarialtraining.
numberofimagetokens(e.g.,4×byswitchingbacktofull
imageresolution)andrequiremoregradientsteps(e.g.,2× 4.Experiment
byswitchingbacktothestrongPGD-3attacker)inthisfine-
In this section, we first introduce the datasets used for ad-
tuningphase,theoverallcomputationdoesnotincreasesig-
versarialtraining,alongwiththedetailsofthetrainingand
nificantly.
evaluationsetupinSec.4.1. InSec.4.2, wedelveintothe
3.3.CLIPEmbeddingforWeb-CrawledImages ablation results, exploring key elements in our two-stage
training pipeline. Furthermore, we investigate the perfor-
Previous works have leveraged the zero-shot generaliza- manceofadversarialtrainingasthemodel,data,andsched-
tion capability of pre-trained CLIP text encoder [47] to ule scale synergistically in Sec. 4.3. Finally, we compare
aid a range of downstream tasks, including object detec- and contrast the efficiency and efficacy of AdvXL against
tion [22, 66, 67] and segmentation [29, 48] in an open- priorartsinSec.4.5.
vocabulary setting. Similarly, we hereby propose to em-
4.1.Implementation
ploy CLIP text encoder to extract classifier weights when
trainingonweb-crawledlarge-scaledatasetswithopentext Dataset. We utilize four different datasets as the training
descriptions, such as LAION-400M [50] and DataComp- set for adversarial training, which are ImageNet-1K and
1B [18]. Moreover, adversarial training on these gigan- ImageNet-21K [13] — two well-curated labeled datasets
tic datasets enables the model to transcend pre-defined for supervised training, as well as LAION-400M [50] and
categories and directly learn intricate class relationships DataComp-1B [18] — two weakly labeled datasets with
throughnaturallanguagesupervision. naturallanguagecaptionscrawledfromtheInternet.
Specifically,weadoptthecontrastivelossfrom[47,54], Specifically, ImageNet-1K comprises approximately
formulatedas: 1.28M images from 1000 classes, while ImageNet-21K
consistsofaround13Mimagesfrom19kclasses. LAION-
(cid:16) (cid:17)
L fI,fT,Ii,Ti = 400M is the first publicly available web-scale dataset con-
  sisting of 400M image-text pairs. It is filtered by CLIP
− 2 1 n (cid:88) i log (cid:80) e j x e p xp (cid:0) h (cid:16) T i hT i ·h · I i h / I j τ / (cid:1) τ (cid:17) +log (cid:80) e j x e p xp (cid:0) h (cid:16) I i h · I i h · T i h / T j τ / (cid:1) τ (cid:17) a D n a d taC N o S m FW p-1B cri i t s eri a on m , o b re ut re is ce s n t t ill da r t e a l s a e t t iv w el i y th n a o b n o -c u u t r 1 at . e 3 d B .
(2) samplesfilteredfromacandidatepoolof12.8Bimage-text
pairs from Common Crawl, which has been recorded to
where n represents the batch size; τ is a learnable tem- yieldsuperiorperformanceforcontrastivetraining.
(cid:12) (cid:12)
perature parameter; hI i = fI(I i )/(cid:12)fI(I i )(cid:12) and hT i = To summarize, our choices of training datasets cover a
(cid:12) (cid:12)
fT (T
i
)/(cid:12)fT (T
i
)(cid:12) denote the normalized projected fea- wide range of representative datasets, spanning from ∼1M
tures of an image-text pair (I
i
, T
i
). Note that we opt for to ∼1B samples, from well-curated labeled data to non-
CLIPA-trainedtextencoder[30,31]astheinitialfT weight curatedwebdata. Thisdiverseselectionenablesacompre-
and keep it frozen during training. In this case, the adver- hensive investigation into the adversarial training concern-
sarialtrainingframeworkcanbedescribedasthefollowing ingdatascalingbehaviors.

Image Encoder
|     |             |     |     |     |     |     |     |     |     |     | ℎ"  | ℎ"  | ℎ" ℎ" |     |
| --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | --- |
|     | Adv Perturb |     |     |     |     |     |     |     |     |     | !   |     |       |     |
# $ %
ℎ&
!
|     |     |     | A fox standing on a rock |     |     |     |     |     |     |     | ℎ&  |     |     |     |
| --- | --- | --- | ------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
#
|     |     |     |  in the woods |     |     |     |     | Text Encoder |     |     |     |     |     |     |
| --- | --- | --- | ------------- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- |
ℎ&
$
ℎ&
%
Figure3.IllustrationofleveragingCLIPembeddinginadversarialtraining.Thegraylinedenotestheadversarialexamplegenerationflow.
| Training.                  | By default,     |     | our training |               | initiates | with     | a pre-   |     |               |     |            |         |       |        |
| -------------------------- | --------------- | --- | ------------ | ------------- | --------- | -------- | -------- | --- | ------------- | --- | ---------- | ------- | ----- | ------ |
|                            |                 |     |              |               |           |          |          |     | Approach      |     | Ratio/Size | Compute | Clean | PGD-20 |
| training                   | stage utilizing |     | an image     | size          | of 112    | × 112    | and      |     |               |     |            |         |       |        |
|                            |                 |     |              |               |           |          |          |     | baseline      |     | 224/0%     | 1.0×    | 75.5  | 54.5   |
| PGD-1withastepsizeof4/255. |                 |     |              | Subsequently, |           | themodel |          |     |               |     |            |         |       |        |
|                            |                 |     |              |               |           |          |          |     | RandomMasking |     | 224/50%    | 0.5×    | 72.0  | 51.9   |
| undergoes                  | a fine-tuning   |     | stage        | employing     | an        | image    | size of  |     |               |     |            |         |       |        |
|                            |                 |     |              |               |           |          |          |     | RandomMasking |     | 224/75%    | 0.25×   | 67.3  | 46.5   |
| 224×224                    | and PGD-3       |     | with a       | step size     | of 4/255. |          | Our pri- |     |               |     |            |         |       |        |
|                            |                 |     |              |               |           |          |          |     | BlockMasking  |     | 224/50%    | 0.5×    | 72.3  | 52.0   |
maryfocuscentersonViT[15],renownedforitsscalability BlockMasking 224/75% 0.25× 70.6 49.3
[15,24,32,47]yetrelativelyunderexploredintherealmof Resizing 160/0% 0.5× 74.7 53.9
| adversarial | training. | Note | that | the current | best | ViT | model |     |          |     |        |       |      |      |
| ----------- | --------- | ---- | ---- | ----------- | ---- | --- | ----- | --- | -------- | --- | ------ | ----- | ---- | ---- |
|             |           |      |      |             |      |     |       |     | Resizing |     | 112/0% | 0.25× | 73.0 | 52.5 |
onImageNet-1KinRobustBenchisonlyViT-B/16[9], in- Resizing 96/0% 0.18× 70.0 49.9
dicatingplentyofroomforfurtherscaling.
(a)Imagetokenreduction.
OnImageNet-1KandImageNet-21K,ourrecipeclosely
followspriorworks[24],whichsuccessfullytrainsViTson Stage Step Stepsize Compute Clean PGD-20
ImageNetatscalefromscratch. Specifically, weadoptthe 1 4/255 1.0× 73.0 52.5
AdamW optimizer [38] with a short-term linear learning Pre-training 2 3/255 1.5× 72.1 52.6
rate warmup followed by a cosine learning rate schedule. 3 3/255 2.0× 71.8 52.5
1.0×
Our data augmentation strategy integrates RandAug [10], 1 4/255 75.0 50.6
MixUp[65]andCutMix[63]. Additionally,weincorporate Fine-tuning 2 4/255 1.5× 73.2 52.3
stochastic depth [27] and weight decay for model regular- 3 4/255 2.0× 73.0 52.5
ization. On web-scale datasets such as LAION-400M and (b)Attackstrength.
| DataComp-1B,        | our | training | recipe | aligns | with | methodolo- |     |     |     |          |            |       |        |     |
| ------------------- | --- | -------- | ------ | ------ | ---- | ---------- | --- | --- | --- | -------- | ---------- | ----- | ------ | --- |
| giesoutlinedin[31]. |     |          |        |        |      |            |     |     |     | Approach | Ratio/Size | Clean | PGD-20 |     |
Thespecificsofourtrainingschedulesaretailoredtoin- 160/0% 74.4 43.2
w/oTuning
| dividual | datasets, | where | the total | number | of  | training | sam- |     |     |     |        |      |     |      |
| -------- | --------- | ----- | --------- | ------ | --- | -------- | ---- | --- | --- | --- | ------ | ---- | --- | ---- |
|          |           |       |           |        |     |          |      |     |     |     | 112/0% | 68.5 |     | 39.3 |
plesservesastheprimarymetric,followingaparadigmakin 160/0% 74.7 53.9
wTuning
| toCLIPtraining[31,32,47]. |          |                |     | Forinstance,ourdefaultpre- |     |          |      |     |     |     |        |      |     |      |
| ------------------------- | -------- | -------------- | --- | -------------------------- | --- | -------- | ---- | --- | --- | --- | ------ | ---- | --- | ---- |
|                           |          |                |     |                            |     |          |      |     |     |     | 112/0% | 73.0 |     | 52.5 |
| training                  | schedule | on ImageNet-1K |     | spans                      | a   | total of | 256M |     |     |     |        |      |     |      |
(c)Fine-tuning.
samples,whichcorrespondsto200epochsoftraining.
Table1.AblatingdesignchoiceswithViT-B/16onImageNet-1K.
| Evaluation.   | In      | our analysis, |           | we primarily |           | use robust | ac-    |       |                      |     |                                  |          |      |               |
| ------------- | ------- | ------------- | --------- | ------------ | --------- | ---------- | ------ | ----- | -------------------- | --- | -------------------------------- | -------- | ---- | ------------- |
|               |         |               |           |              |           |            |        | We    | report clean         | and | PGD-20 robust                    | accuracy | (%). | If not speci- |
| curacy under  | PGD-20  |               | attack    | with a       | step size | of 1/255   | as     |       |                      |     |                                  |          |      |               |
|               |         |               |           |              |           |            |        | fied, | thedefaultsettingis: |     | 112×112imagesizeforpre-training, |          |      |               |
| the principal | metric. | When          | comparing |              | against   | other      | state- |       |                      |     |                                  |          |      |               |
224×224imagesizeforfine-tuning;PGD-1forpre-training,and
of-the-artmethods,wefollowRobustBench[9]andusethe PGD-3 for fine-tuning; 200 epochs for pre-training length, 20
robustaccuracyevaluatedonasubsetofselected5000im- epochsforfine-tuning.Defaultsettingsaremarkedin gray.Inta-
agesoftheImageNet-1KvalidationsetunderAutoAttack.
ble(a)and(b),notethatfull-resolutionfine-tuningisincluded.In
AutoAttackisastandardizedadversarialrobustnessbench- table(b),whentuningthePGDstepandstepsizeinpre-training,
mark that consists of an ensemble of white- and black- wefixthemtobe3and4/255respectivelyinfine-tuning; When
boxattacks,includingAPGDforcross-entropyandtargeted tuningthePGDstepandstepsizeinfine-tuning,wefixthemtobe
1and4/255respectivelyinpre-training.
| DLR loss, | FAB-attack   |                         | [8] and | the black-box |     | Square | At-   |     |     |     |     |     |     |     |
| --------- | ------------ | ----------------------- | ------- | ------------- | --- | ------ | ----- | --- | --- | --- | --- | --- | --- | --- |
| tack [2]. | The attack   | radii                   | are     | ϵ = 4/255,    |     | ϵ = 2, | and ϵ |     |     |     |     |     |     |     |
|           |              |                         |         | ∞             |     | 2      | 1     |     |     |     |     |     |     |     |
| =75forl   | ∞ ,l 2 ,andl | 1 attacks,respectively. |         |               |     |        |       |     |     |     |     |     |     |     |

| 4.2.DesignChoices |     |     |     |     |     |     | 4.3.ScalingBehavior |     |     |     |     |     |     |
| ----------------- | --- | --- | --- | --- | --- | --- | ------------------- | --- | --- | --- | --- | --- | --- |
Wefirstconductanablationstudyonthedesignchoicesof Theaccelerationoutlinedpreviouslyallowsustodelveinto
AdvXLusingViT-B/16onImageNet-1K,withrobustaccu- the performance implications of scaling AdvXL within an
racyunderPGD-20servingastheprimarymetricforadver- affordable computational budget. In particular, we scruti-
sarial robustness. We maintain the default baseline setting nizethescalingbehavioralongthreeprincipalaxesbelow,
(seethecaptionofTab.1). Anyalterationsareconfinedto inlinewiththeapproachestablishedbyLietal.[32]:
thespecificfactorsunderexamination. • Model scaling. We substitute the ViT-B/16 model with
ViT-L/16orViT-H/14,whichhas∼2×or∼4×numberof
| TokenReduction. |     | Ourinvestigationdelvesintothreedis- |     |     |     |     |     |     |     |     |     |     |     |
| --------------- | --- | ----------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
parameters,respectively.
| tinctstrategiesforreducingimagetokenlength: |                |          |         |         |         | 1)random     |                |       |                                       |        |                       |         |     |
| ------------------------------------------- | -------------- | -------- | ------- | ------- | ------- | ------------ | -------------- | ----- | ------------------------------------- | ------ | --------------------- | ------- | --- |
|                                             |                |          |         |         |         |              | • Datascaling. |       | WesubstitutethetrainingsetofImageNet- |        |                       |         |     |
| masking,                                    | which          | randomly | removes | a       | portion | of input to- |                |       |                                       |        |                       |         |     |
|                                             |                |          |         |         |         |              | 1K with        | three | much                                  | larger | datasets, excessively | expand- |     |
| kens; 2)                                    | block masking, |          | which   | retains | a large | consecutive  |                |       |                                       |        |                       |         |     |
ingthetotalnumberoftrainingsamplesuptomorethan
| block of | the input | grid; | 3) resizing, | which | preserves | most |     |     |     |     |     |     |     |
| -------- | --------- | ----- | ------------ | ----- | --------- | ---- | --- | --- | --- | --- | --- | --- | --- |
high-level semantic information. As shown in Tab. 1a, all ∼1B.ThesedatasetsincludeImageNet-21K[13],asuper-
setofImageNet-1K;LAION-400M[50],andDataComp-
| three methods | exhibit |     | substantial | computational |     | speedups. |     |     |     |     |     |     |     |
| ------------- | ------- | --- | ----------- | ------------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- |
Notably,imageresizingdemonstratessuperiorperformance 1B[18],twoweb-scaledatasets.
|             |             |     |            |         |     |                | • Schedule | scaling. |     | To delineate | the influence | of  | large |
| ----------- | ----------- | --- | ---------- | ------- | --- | -------------- | ---------- | -------- | --- | ------------ | ------------- | --- | ----- |
| among these | strategies, |     | presumably | because |     | it suffers the |            |          |     |              |               |     |       |
leastfromlossofinformation. Forinstance,resizingthein- dataset size from that of extended training duration, we
put image to 112×112 leads to a 75% reduction in total conduct training on ImageNet-21K with the same num-
computation, with only a minor decrease of 2.5% in clean berofseensamplesastrainingonImageNet-1K.
accuracyand2.0%inPGD-20robustaccuracy.Weselectan
|     |     |     |     |     |     |     | By meticulously |     | traversing |     | these three | scaling axes, | we  |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | ---------- | --- | ----------- | ------------- | --- |
imagesizeof112×112forpre-trainingasthedefaultset-
scrutinizetheirindividualeffectsonAdvXL’sperformance.
| ting due | to its satisfactory |     | balance | between | efficiency | and |              |     |          |         |                |        |      |
| -------- | ------------------- | --- | ------- | ------- | ---------- | --- | ------------ | --- | -------- | ------- | -------------- | ------ | ---- |
|          |                     |     |         |         |            |     | The findings | are | detailed | in Tab. | 2, culminating | in the | fol- |
performance.
lowinginsights.
Attack Strength. Tab. 1b scrutinizes the impact of vary- Model scaling. The evaluation of larger model sizes re-
ingattackstepsandstepsizesduringpre-trainingandfine-
vealsdiscernibleimprovementsinbothcleanaccuracyand
tuning. Intriguingly, we observe that the number of PGD adversarial robustness. For instance, as shown in the
| steps for | pre-training | does | not | need to | align | with that for |           |            |      |     |                  |           |     |
| --------- | ------------ | ---- | --- | ------- | ----- | ------------- | --------- | ---------- | ---- | --- | ---------------- | --------- | --- |
|           |              |      |     |         |       |               | first and | the second | rows | of  | Tab. 2, ViT-L/16 | surpasses |     |
fine-tuning. Forinstance, adoptingPGD-1forpre-training ViT-B/16 by 1.8% clean accuracy (from 73.0% to 74.8%)
yields nearly equivalent robustness compared to PGD-3, and1.8%PGD-20-robustaccuracy(from52.5%to54.7%)
| while reducing | the | computation |     | by 100%. | This | suggests |               |     |                 |     |                |           |     |
| -------------- | --- | ----------- | --- | -------- | ---- | -------- | ------------- | --- | --------------- | --- | -------------- | --------- | --- |
|                |     |             |     |          |      |          | when training |     | on ImageNet-1K. |     | Interestingly, | ViT-H/14, |     |
thatdespiteexposuretoweakerattacksduringpre-training despite its superior clean accuracy and tripled computa-
| (e.g., with | PGD-1), | a   | short-term | but stronger |     | adversarial |                 |     |              |     |                 |        |         |
| ----------- | ------- | --- | ---------- | ------------ | --- | ----------- | --------------- | --- | ------------ | --- | --------------- | ------ | ------- |
|             |         |     |            |              |     |             | tional expense, |     | demonstrates |     | only a slightly | better | perfor- |
fine-tuning(e.g.,withPGD-3)issufficientforthemodelto mance(0.2%higherPGD-20robustness)comparedtoViT-
securestrongrobustnessagainstadversarialattacks. There- L/16whentrainingonImageNet-1K,asshowninthethird
fore,weopttousePGD-1forpre-trainingandPGD-3for
|     |     |     |     |     |     |     | row of Tab. | 2.  | However, | it notably | surpasses | ViT-L/16 | by  |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | -------- | ---------- | --------- | -------- | --- |
fine-tuninginourdefaultsetting. a substantial margin (2.2% in PGD-20 robustness) when
|     |     |     |     |     |     |     | training | on the | larger | ImageNet-21K | dataset | (as shown | in  |
| --- | --- | --- | --- | --- | --- | --- | -------- | ------ | ------ | ------------ | ------- | --------- | --- |
Fine-tuning. Tab.1coutlinestheimpactoffullresolution thefifthandsixthrowsofTab.2).Thisobservationsuggests
fine-tuning with stronger attacks for an extra 20 epochs that larger models necessitate a larger training set to fully
| on the ImageNet-1K |     | dataset. |     | For 112 | × 112 | PGD-1 pre- |          |                  |     |      |                |              |     |
| ------------------ | --- | -------- | --- | ------- | ----- | ---------- | -------- | ---------------- | --- | ---- | -------------- | ------------ | --- |
|                    |     |          |     |         |       |            | leverage | their potential. |     | This | finding aligns | with conclu- |     |
training,a224×224PGD-3fine-tuningelevatescleanaccu- sionsinpriorstudies[26],advocatingforequivalentscaling
racyby4.5%andPGD-20robustaccuracyby13.2%. This ofmodelsizeandthevolumeoftrainingtokens.
| fine-tuning | phase | substantially |     | narrows | the | performance |     |     |     |     |     |     |     |
| ----------- | ----- | ------------- | --- | ------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- |
gap between reduced-length pre-training and full-length Schedule scaling. Initial experiments demonstrated that
training, demanding only around 60% of the pre-training extendingthetrainingscheduleforViT-L/16onImageNet-
computationalresources. Extendingthepre-trainingsched- 1Kyieldeddiminishinggains,possiblyduetothecompar-
ule by the corresponding compute yields significantly in- atively “limited” scale of ImageNet-1K. However, results
ferior results, highlighting the distinct advantage of fine- inTab.2showsthatwithlargerandmorediversedatasets,
tuninginachievingasuperiorperformance-computetrade- trainingwithadditionalsamplesyieldsnon-trivialenhance-
off. Therefore, weconsistentlyintegrateashort-termfine- ments.Evenwitha20×augmentationinthetrainingsched-
tuningstagepostpre-training. ule using a one-billion sample dataset, such as training a

Compute
|     |     | Case |     | Model |     | Dataset |     | Samples@Resolution |     | Adv.Steps |     |     | Clean | PGD-20 |
| --- | --- | ---- | --- | ----- | --- | ------- | --- | ------------------ | --- | --------- | --- | --- | ----- | ------ |
(1e10)
Baseline ViT-B/16 ImageNet-1K 256M@112+38.4M@224 1/3 0.5 73.0 52.5
modelscaling ViT-L/16 ImageNet-1K 256M@112+38.4M@224 1/3 1.7 74.8 54.7
modelscaling ViT-H/14 ImageNet-1K 256M@112+38.4M@224 2/3 5.7 76.5 54.9
+ImageNet-21K
model+datascaling ViT-L/16 256M@112+38.4M@224 1/3 1.7 75.8 56.1
+ImageNet-21K
model+data+schedulescaling ViT-L/16 789M@112+38.4M@224 1/3 3.4 77.2 58.3
model+data+schedulescaling ViT-H/14 +ImageNet-21K 789M@84+38.4M@224 2/3 8.1 79.0 60.5
model+data+schedulescaling ViT-L/16 +LAION-400M 2.56B@112+38.4M@224 1/3 8.8 80.5 62.2
model+data+schedulescaling ViT-H/14 +DataComp-1B 5.12B@84+38.4M@224 2/3 38.6 83.3 68.2
Table2.ScalingbehaviorofAdvXL.Foreachmodel,wereportitstrainingset,thenumberoftrainingsamplesitusedandtheirresolution,
itsPGDnumberofsteps(inpre-trainingandfine-tuning,respectively),thetotaltrainingcompute(in1e10GFLOPS),cleanaccuracy,and
PGD-20-robustaccuracy.“+”onthedatasetmeansanyadditionaldatasetusedduringtrainingbesidesImageNet-1K.Wescalealongthree
aspects:model,data,andscale,andobserveconsistentimprovementintermsofbothcleanaccuracyandrobustness.
|     | Dataset |     | Model    |     | Clean | PGD-20 |     |              |        |       |            |     |             |         |
| --- | ------- | --- | -------- | --- | ----- | ------ | --- | ------------ | ------ | ----- | ---------- | --- | ----------- | ------- |
|     |         |     |          |     |       |        |     | reduced-size | inputs | posed | challenges | as  | the feature | size of |
|     |         |     | ViT-B/16 |     | 73.0  | 52.5   |     |              |        |       |            |     |             |         |
ImageNet-1K the last stage may even be smaller than the window size.
|     |     |     | ConvNeXT-B |     | 73.9 | 54.2 |     |     |     |     |     |     |     |     |
| --- | --- | --- | ---------- | --- | ---- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
ViT-L/16 80.5 62.2 Forexample,whenemployingcommonconfigurationslike
+LAION-400M
ConvNeXT-L 77.9 58.5 apatchsize4×4andawindowsize7×7,usinga112×112
|     |     |     |     |     |     |     |     | inputwould | leadtoa |     | finalstagefeaturesize |     | of3×3. | This |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | ------- | --- | --------------------- | --- | ------ | ---- |
Table3.ArchitecturecomparisonbetweenViTandConvNeXT.
|     |     |             |     |       |        |     |     | mismatch       | hindered | effective | training     | without | architectural |     |
| --- | --- | ----------- | --- | ----- | ------ | --- | --- | -------------- | -------- | --------- | ------------ | ------- | ------------- | --- |
|     |     | TextEncoder |     | Clean | PGD-20 |     |     |                |          |           |              |         |               |     |
|     |     |             |     |       |        |     |     | modifications, |          | and thus, | we primarily | focus   | on comparing  |     |
|     |     | Base        |     | 80.6  | 62.2   |     |     |                |          |           |              |         |               |     |
ViTandConvNeXT.
|     |     | Large |     | 80.5 | 62.2 |     |     |     |     |     |     |     |     |     |
| --- | --- | ----- | --- | ---- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Toensurefairevaluation,wemaintainconsistencywith
|     |     | Huge |     | 80.6 | 62.3 |     |     |     |     |     |     |     |     |     |
| --- | --- | ---- | --- | ---- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
thesametwo-stagetrainingrecipedetailedinSec.4.3dur-
Table4.CLIPtextencodersize.
|     |     |     |     |     |     |     |     | ingtheperformancecomparison. |     |     |     | Theresults, | presentedin |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------------------- | --- | --- | --- | ----------- | ----------- | --- |
ViT-H/14 model on DataComp-1B for 5.12B samples (the Tab. 3, demonstrate that ConvNeXT does outperform ViT
lastrowofTab.2),thereisnotanobservedsaturationpoint.
|          |          |           |       |          |            |     |           | on a relatively |          | small | scale. However, |         | this advantage | di-    |
| -------- | -------- | --------- | ----- | -------- | ---------- | --- | --------- | --------------- | -------- | ----- | --------------- | ------- | -------------- | ------ |
|          |          |           |       |          |            |     |           | minishes        | as the   | scale | increases,      | leading | us to keep     | ViT as |
| Data     | scaling. | Our       | AdvXL | also     | exhibits   |     | favorable |                 |          |       |                 |         |                |        |
|          |          |           |       |          |            |     |           | the default     | backbone |       | for comparisons | against | other          | state- |
| outcomes | with     | web-scale |       | datasets | LAION-400M |     | and       |                 |          |       |                 |         |                |        |
of-the-artmodels.
| DataComp-1B.      |     | This    | trend | could potentially |          | pave         | the way |       |                                           |     |     |     |     |     |
| ----------------- | --- | ------- | ----- | ----------------- | -------- | ------------ | ------- | ----- | ----------------------------------------- | --- | --- | --- | --- | --- |
|                   |     |         |       |                   |          |              |         | Also, | wecouldadoptalargerpre-trainedCLIPtexten- |     |     |     |     |     |
| for adversarially |     | trained |       | models            | to rival | foundational |         |       |                                           |     |     |     |     |     |
models like CLIP [47] and Flamingo [1]. Notably, we coderincontrastivelearning,asitisfrozenandintroduces
|           |      |         |        |                |     |      |         | little computational |     | overhead. | Tab. | 4 shows | the | result of |
| --------- | ---- | ------- | ------ | -------------- | --- | ---- | ------- | -------------------- | --- | --------- | ---- | ------- | --- | --------- |
| find that | data | scaling | itself | is beneficial, |     | even | without | a                    |     |           |      |         |     |           |
prolonged training schedule. As shown in the second and training ViT-L/16 on LAION-400M with various CLIPA
textencoders.Ascanbeseen,theperformanceisrobusttoa
| the fourth | rows         | of  | Tab.             | 2, by substituting |       | ImageNet-1K |     |                                    |     |     |     |     |               |     |
| ---------- | ------------ | --- | ---------------- | ------------------ | ----- | ----------- | --- | ---------------------------------- | --- | --- | --- | --- | ------------- | --- |
|            |              |     |                  |                    |       |             |     | widerangeofCLIPtextencoderchoices. |     |     |     |     | Thus,wesimply |     |
| with       | ImageNet-21K |     | to adversarially |                    | train | ViT-L/16,   | we  |                                    |     |     |     |     |               |     |
observe an uptick of 1.0% in clean accuracy and a 1.4% usethesame-scaletextencodertotheimageencoder(i.e.a
ViT-LimageencoderwithaLargetextencoder).
| increase   | in  | robustness,  | notwithstanding |          |             | identical | training |     |     |     |     |     |     |     |
| ---------- | --- | ------------ | --------------- | -------- | ----------- | --------- | -------- | --- | --- | --- | --- | --- | --- | --- |
| durations. |     | When coupled |                 | with our | preliminary |           | findings |     |     |     |     |     |     |     |
4.5.ComparisonwithSOTAModels
suggestingdiminishedreturnsfromextendedscheduleson
ImageNet-1K, we conclude that the richness and diversity The comparison presented in Tab. 5 evaluates our models
| brought | by  | data scaling | stand | as  | pivotal | elements | in the |         |       |        |             |                |     |          |
| ------- | --- | ------------ | ----- | --- | ------- | -------- | ------ | ------- | ----- | ------ | ----------- | -------------- | --- | -------- |
|         |     |              |       |     |         |          |        | against | prior | works, | focusing on | l ∞ robustness |     | at ϵ ∞ = |
successofadversarialtrainingatscale. 4/255. Following [52], we include l robustness at ϵ = 2
|     |     |     |     |     |     |     |     |                  |     |     |                | 2   |                | 2    |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | --- | -------------- | --- | -------------- | ---- |
|     |     |     |     |     |     |     |     | l                |     | ϵ   |                |     |                |      |
|     |     |     |     |     |     |     |     | and 1 robustness |     | at  | 1 = 75. Models |     | listed exhibit | over |
4.4.ArchitectureChoice
|     |     |     |     |     |     |     |     | 80M parameters |     | and | are sorted | based on | their l | robust- |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --- | ---------- | -------- | ------- | ------- |
∞
WehavealsoablatedalternativearchitecturessuchasCon- nessunderAutoAttack.
vNeXT[37]andSwin-Transformer[35],twoleadingback- AdvXL emerges as the top performer owing to its un-
bones on RobustBench ImageNet leaderboard [34, 52]. precedented scale in adversarial training. Our highly ef-
However, our attempts to train a Swin-Transformer with ficient two-stage training paradigm facilitates this scala-

|     |     |     |     |     |     |     |     | Params | Compute |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------- | --- | --- | --- |
Model Dataset Samples@Resolution Pre-trained Adv.Steps Source Clean l∞ l2 l1
|     |            |     |     |          |     |     |     | (M) | (1e10)   |      |      |           |
| --- | ---------- | --- | --- | -------- | --- | --- | --- | --- | -------- | ---- | ---- | --------- |
|     | RobArch-L  |     |     | 128M@224 |     |     | 3   | 104 | 1.3 [43] | 73.5 | 48.9 | 39.5 14.7 |
|     | ViT-B/16   |     |     | 384M@224 |     |     | 2   | 87  | 2.7 [49] | 76.6 | 53.5 | - -       |
|     | ConvNeXT-B |     |     | 384M@224 |     |     | 3   | 89  | 2.4 [34] | 76.0 | 55.8 | 44.7 21.2 |
|     | Swin-B     |     |     | 384M@224 |     |     | 3   | 88  | 2.4 [34] | 76.2 | 56.2 | 47.9 23.9 |
ImageNet-1K
✓
ConvNeXt-B+ConvStem 320M@224 3 89 2.0 [52] 75.2 56.3 49.4 23.6
ConvNeXt-L+ConvStem 128M@224 ✓ 3 198 1.8 [52] 77.0 57.7 47.0 22.2
ConvNeXt-L+ConvStem 128M@224(320eval) ✓ 3 198 1.8 [52] 78.2 59.4 56.2 33.8
|     | ConvNeXt-L |     |     | 384M@224 |     |     | 3   | 198 | 5.3 [34] | 78.0 | 58.5 | - - |
| --- | ---------- | --- | --- | -------- | --- | --- | --- | --- | -------- | ---- | ---- | --- |
|     | Swin-L     |     |     | 384M@224 |     |     | 3   | 197 | 5.3 [34] | 78.9 | 59.6 | - - |
ViT-H/14 +DataComp-1B 5.12B@84+38.4M@224+6.4M@336 2/3 304 39.6 ours 83.9 69.8 69.8 46.0
ViT-g/14 +DataComp-1B 5.12B@84+38.4M@224+6.4M@336 2/3 1013 63.4 ours 83.9 71.0 70.4 46.7
Table 5. Comparison to SOTA l -robust models on ImageNet. For each model we report the training set it used, the number and
∞
resolutionoftrainingsamplesitused,ifitusespre-trainedweightsornot,thenumberofPGDstepsinAT(inpre-trainingandfine-tuning,
respectively),thenumberofparametersofeachmodel,thetotaltrainingcompute(in1e10
GFLOPS),itssource,itscleanaccuracyand
l ∞ ,l 2 ,l 1 -robustaccuracywithϵ ∞ =4/255,ϵ 2 =2,ϵ 1 =75(AutoAttack). Notethatforthemodelinitializedwithpre-trainedweight,the
pre-trainingcomputeisnotincluded. Forunavailablemetricsofthosepubliclyunavailablemodels,weuse“-”tofillintheblank. “+”on
thedatasetmeansanyadditionaldatasetusedduringtrainingbesidesImageNet-1K.OurAdvXLsuccessfullysecuresnewstate-of-the-art
recordsonallthreerobustnessmetricsthankstoitsunprecedentedmodelanddatascale.
bility without incurring excessive computational expenses. ImageNet-1K dataset. In this work, we break new ground
For instance, our largest ViT-g/14 model trained on the by scaling adversarial training to web-scale datasets con-
DataComp-1B dataset achieves outstanding results with a taining over 1B samples. Our AdvXL approach com-
computing budget of merely about 12× that of the previ- prises two core components: 1) a coarse-to-fine, weak-to-
ous best results from [34]. Despite this relatively modest strong, two-stage training paradigm to mitigate the com-
computationalinvestment,ourmodeloutperformsthemby putational cost of scaling up; 2) the utilization of a pre-
animpressive11.4%intermsofl -robustaccuracyunder trained CLIP text encoder enabling training on web-scale
∞
AutoAttack. Wewouldliketostressthattrainingwithfull datasets. Through scaling along model, data, and sched-
resolutionandstrongattackson5.12Bsamples,withoutour ule dimensions, we successfully establish a new state-of-
efficiencydesign,wouldincur∼20×thecomputationalcost the-artrecordofl -robustaccuracyunderAutoAttack,sur-
∞
ofourapproach(equatingto∼250×thecomputeofthepre- passingthepreviousbestbyamarginof∼10%. Addition-
vious best results), rendering such an endeavor computa- ally, training on those gigantic datasets demonstrates in-
tionallyinfeasible. creasedgeneralizabilityagainstunseenattacksduringtrain-
Even more noteworthy is the exceptional generalizabil- ing, aligning with observations from various foundation
|     |           |              |          |        |            | models | [1, 6, | 45–47]. | We envision | our work | as a | stepping |
| --- | --------- | ------------ | -------- | ------ | ---------- | ------ | ------ | ------- | ----------- | -------- | ---- | -------- |
| ity | showcased | by our AdvXL | ViT-g/14 | models | trained on |        |        |         |             |          |      |          |
theweb-scaleDataComp-1Bdataset,securingl -robustac- stoneforadversarialtrainingtoentertheeraoffoundation
2
curacy of 70.4% and l -robust accuracy of 46.7%. These models,inspiringfurtherlarge-scaleadversarialtrainingen-
1
deavors.
| figures | represent | an absolute            | improvement      | of  | about 13% |              |     |                                     |     |     |     |     |
| ------- | --------- | ---------------------- | ---------------- | --- | --------- | ------------ | --- | ----------------------------------- | --- | --- | --- | --- |
| over    | the       | best previous results. | This observation |     | indicates |              |     |                                     |     |     |     |     |
|         |           |                        |                  |     |           | Broadimpact. |     | Ourmethoddeliversover5×speedup,sig- |     |     |     |     |
thatscalingmodel,data,andschedulecollectivelynotonly
nificantlyreducingwall-clocktimefortrainingmodelswith
significantlyenhancesrobustnessagainstknownattacksbut
|     |     |     |     |     |     | hundreds | of  | millions | or even billions | of  | parameters | on  |
| --- | --- | --- | --- | --- | --- | -------- | --- | -------- | ---------------- | --- | ---------- | --- |
alsofortifiesthemodelagainstunseenattacksduringtrain-
|     |     |     |     |     |     | billion-scale | datasets |     | (e.g., on the | order | of thousands | of  |
| --- | --- | --- | --- | --- | --- | ------------- | -------- | --- | ------------- | ----- | ------------ | --- |
ing. Ourfindingsonscalingadversarialtrainingilluminate
|     |     |     |     |     |     | TPU/GPU-days). |     | AdvXL | not only | facilitates | rapid | proto- |
| --- | --- | --- | --- | --- | --- | -------------- | --- | ----- | -------- | ----------- | ----- | ------ |
thepathtowardstheevolutionofnext-generationrobustvi-
|      |         |                        |     |       |                | typingand | accelerated |     | researchcyclesbutalso |     | contributes |     |
| ---- | ------- | ---------------------- | --- | ----- | -------------- | --------- | ----------- | --- | --------------------- | --- | ----------- | --- |
| sual | models, | potentially propelling | the | field | of adversarial |           |             |     |                       |     |             |     |
tosubstantialenergyandcarbonemissionssavings,acriti-
trainingintotheeraoffoundationmodels.
calconsiderationinlarge-scalemodeltraining.
| 5.DiscussionandConclusion |     |     |     |     |     | Acknowledge |     |     |     |     |     |     |
| ------------------------- | --- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- |
Adversarial training has traditionally been confined to This work is partially supported by a gift from Open Phi-
smallnetworksanddatasets,predominantlyResNet-50and lanthropy. We thank Center for AI Safety, TPU Research
CIFAR-10. Until recently, there have been few attempts Cloud(TRC)program,andGoogleCloudResearchCredits
to train adversarially robust models on the medium-size programforsupportingourcomputingneeds.

References [16] YuxinFang,WenWang,BinhuiXie,QuanSun,LedellWu,
|     |     |     |     |     |     |     |     | Xinggang |     | Wang, | Tiejun | Huang, | Xinlong Wang, | and Yue |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | ----- | ------ | ------ | ------------- | ------- |
[1] Jean-BaptisteAlayrac, JeffDonahue, PaulineLuc, Antoine Cao. Eva:Exploringthelimitsofmaskedvisualrepresenta-
Miech,IainBarr,YanaHasson,KarelLenc,ArthurMensch,
|                    |     |                  |     |     |       |           |     | tionlearningatscale. |     |               | InCVPR,2023. |          | 1,2       |     |
| ------------------ | --- | ---------------- | --- | --- | ----- | --------- | --- | -------------------- | --- | ------------- | ------------ | -------- | --------- | --- |
| KatherineMillican, |     | MalcolmReynolds, |     |     | etal. | Flamingo: | a   |                      |     |               |              |          |           |     |
|                    |     |                  |     |     |       |           |     | [17] YongganFu,      |     | ShunyaoZhang, |              | ShangWu, | ChengWan, | and |
visuallanguagemodelforfew-shotlearning.NeurIPS,2022.
|     |     |     |     |     |     |     |     | Yingyan | Lin. | Patch-fool: |     | Are vision | transformers | always |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ---- | ----------- | --- | ---------- | ------------ | ------ |
7,8
|     |     |     |     |     |     |     |     | robustagainstadversarialperturbations? |     |     |     |     | InICLR,2022. | 2   |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- | ------------ | --- |
[2] MaksymAndriushchenko, FrancescoCroce, NicolasFlam- [18] SamirYitzhakGadre,GabrielIlharco,AlexFang,Jonathan
| marion,andMatthiasHein. |             |        | Squareattack: |        | aquery-efficient |     |       |                                               |          |        |          |      |               |              |
| ----------------------- | ----------- | ------ | ------------- | ------ | ---------------- | --- | ----- | --------------------------------------------- | -------- | ------ | -------- | ---- | ------------- | ------------ |
|                         |             |        |               |        |                  |     |       | Hayase,                                       | Georgios |        | Smyrnis, | Thao | Nguyen,       | Ryan Marten, |
| black-box               | adversarial | attack | via           | random | search.          | In  | ECCV, |                                               |          |        |          |      |               |              |
|                         |             |        |               |        |                  |     |       | MitchellWortsman,DhrubaGhosh,JieyuZhang,etal. |          |        |          |      |               | Dat-         |
| 2020.                   | 5           |        |               |        |                  |     |       |                                               |          |        |          |      |               |              |
|                         |             |        |               |        |                  |     |       | acomp:                                        | In       | search | of the   | next | generation of | multimodal   |
[3] YutongBai,JieruMei,AlanLYuille,andCihangXie. Are datasets. InNeurIPS,2022. 4,6
transformersmorerobustthancnns? NeurIPS,2021. 2 [19] Ian Goodfellow, Jonathon Shlens, and Christian Szegedy.
| [4] HangboBao,LiDong,SonghaoPiao,andFuruWei. |     |     |     |     |              |     | BEit: |            |       |                |     |             |           |          |
| -------------------------------------------- | --- | --- | --- | --- | ------------ | --- | ----- | ---------- | ----- | -------------- | --- | ----------- | --------- | -------- |
|                                              |     |     |     |     |              |     |       | Explaining |       | and harnessing |     | adversarial | examples. | In ICLR, |
| BERTpre-trainingofimagetransformers.         |     |     |     |     | InICLR,2022. |     | 3     |            |       |                |     |             |           |          |
|                                              |     |     |     |     |              |     |       | 2015.      | 1,2,3 |                |     |             |           |          |
[5] BattistaBiggio,IginoCorona,DavideMaiorca,BlaineNel- [20] SvenGowal,ChongliQin,JonathanUesato,TimothyMann,
Sˇrndic´,
son, Nedim Pavel Laskov, Giorgio Giacinto, and and Pushmeet Kohli. Uncovering the limits of adversarial
FabioRoli. Evasionattacksagainstmachinelearningattest trainingagainstnorm-boundedadversarialexamples. arXiv
| time.   | InECMLPKDD,2013. |     | 3     |      |        |         |      |                                |     |        |        |         |          |               |
| ------- | ---------------- | --- | ----- | ---- | ------ | ------- | ---- | ------------------------------ | --- | ------ | ------ | ------- | -------- | ------------- |
|         |                  |     |       |      |        |         |      | preprintarXiv:2010.03593,2020. |     |        |        |         | 2        |               |
| [6] Tom | Brown, Benjamin  |     | Mann, | Nick | Ryder, | Melanie | Sub- |                                |     |        |        |         |          |               |
|         |                  |     |       |      |        |         |      | [21] Jindong                   | Gu, | Volker | Tresp, | and Yao | Qin. Are | vision trans- |
biah,JaredDKaplan,PrafullaDhariwal,ArvindNeelakan- formersrobusttopatchperturbations? InECCV,2022. 2
tan,PranavShyam,GirishSastry,AmandaAskell,etal.Lan- [22] Xiuye Gu, Tsung-Yi Lin, Weicheng Kuo, and Yin Cui.
| guagemodelsarefew-shotlearners. |       |       |            | NeurIPS,2020. |     |        | 1,8  |                        |     |     |                  |     |            |              |
| ------------------------------- | ----- | ----- | ---------- | ------------- | --- | ------ | ---- | ---------------------- | --- | --- | ---------------- | --- | ---------- | ------------ |
|                                 |       |       |            |               |     |        |      | Open-vocabulary        |     |     | object detection |     | via vision | and language |
| [7] Tianlong                    | Chen, | Sijia | Liu, Shiyu | Chang,        | Yu  | Cheng, | Lisa |                        |     |     |                  |     |            |              |
|                                 |       |       |            |               |     |        |      | knowledgedistillation. |     |     | InICLR,2022.     |     | 4          |              |
Amini,andZhangyangWang.Adversarialrobustness:From [23] KaimingHe,XiangyuZhang,ShaoqingRen,andJianSun.
self-supervisedpre-trainingtofine-tuning. InCVPR,2020. Deep residual learning for image recognition. In CVPR,
| 2             |          |              |        |          |           |           |         | 2016.        | 1   |        |       |         |              |           |
| ------------- | -------- | ------------ | ------ | -------- | --------- | --------- | ------- | ------------ | --- | ------ | ----- | ------- | ------------ | --------- |
| [8] Francesco | Croce    | and Matthias |        | Hein.    | Minimally | distorted |         |              |     |        |       |         |              |           |
|               |          |              |        |          |           |           |         | [24] Kaiming | He, | Xinlei | Chen, | Saining | Xie, Yanghao | Li, Piotr |
| adversarial   | examples | with         | a fast | adaptive | boundary  |           | attack. |              |     |        |       |         |              |           |
Dolla´r,andRossGirshick.Maskedautoencodersarescalable
| InICML,2020. |     | 5   |     |     |     |     |     | visionlearners. |     | InCVPR,2022. |     | 3,5 |     |     |
| ------------ | --- | --- | --- | --- | --- | --- | --- | --------------- | --- | ------------ | --- | --- | --- | --- |
[9] Francesco Croce, Maksym Andriushchenko, Vikash Se- [25] Dan Hendrycks, Kimin Lee, and Mantas Mazeika. Using
hwag, Edoardo Debenedetti, Nicolas Flammarion, Mung pre-trainingcanimprovemodelrobustnessanduncertainty.
| Chiang,PrateekMittal,andMatthiasHein. |     |     |     |     | Robustbench: |     | a   |              |     |     |     |     |     |     |
| ------------------------------------- | --- | --- | --- | --- | ------------ | --- | --- | ------------ | --- | --- | --- | --- | --- | --- |
|                                       |     |     |     |     |              |     |     | InICML,2019. |     | 2   |     |     |     |     |
standardizedadversarialrobustnessbenchmark.InNeurIPS,
|     |     |     |     |     |     |     |     | [26] Jordan | Hoffmann, |     | Sebastian | Borgeaud, | Arthur | Mensch, |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --------- | --- | --------- | --------- | ------ | ------- |
2021. 2,5 Elena Buchatskaya, Trevor Cai, Eliza Rutherford, Diego
[10] EkinDogusCubuk,BarretZoph,JonShlens,andQuocLe. delasCasas,LisaAnneHendricks,JohannesWelbl,Aidan
Randaugment: Practicalautomateddataaugmentationwith Clark, Tom Hennigan, Eric Noland, Katherine Millican,
| areducedsearchspace. |     |     | InNeurIPS,2020. |     | 5   |     |     |     |     |     |     |     |     |     |
| -------------------- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
GeorgevandenDriessche,BogdanDamoc,AureliaGuy,Si-
[11] Edoardo Debenedetti, Vikash Sehwag, and Prateek Mittal. monOsindero,KarenSimonyan,ErichElsen,OriolVinyals,
Alightrecipetotrainrobustvisiontransformers. InSaTML, Jack William Rae, and Laurent Sifre. An empirical analy-
2023. 2 sis of compute-optimal large language model training. In
[12] Mostafa Dehghani, Josip Djolonga, Basil Mustafa, Piotr NeurIPS,2022. 6
Padlewski, Jonathan Heek, Justin Gilmer, Andreas Peter [27] GaoHuang,YuSun,ZhuangLiu,DanielSedra,andKilianQ
Steiner,MathildeCaron,RobertGeirhos,IbrahimAlabdul- Weinberger.Deepnetworkswithstochasticdepth.InECCV,
| mohsin,etal. |     | Scalingvisiontransformersto22billionpa- |     |     |     |     |     | 2016. | 5   |     |     |     |     |     |
| ------------ | --- | --------------------------------------- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- |
rameters. InICML,2023. 1,2 [28] AlexKrizhevsky. Learningmultiplelayersoffeaturesfrom
[13] Jia Deng, Wei Dong, Richard Socher, Li-Jia Li, Kai Li, tinyimages. Technicalreport,2009. 1
andLiFei-Fei. Imagenet: Alarge-scalehierarchicalimage [29] Boyi Li, Kilian Q Weinberger, Serge Belongie, Vladlen
database. InCVPR,2009. 4,6 Koltun, and Rene Ranftl. Language-driven semantic seg-
[14] Jacob Devlin, Ming-Wei Chang, Kenton Lee, and Kristina mentation. InICLR,2022. 4
Toutanova. BERT:Pre-trainingofdeepbidirectionaltrans- [30] XianhangLi,ZeyuWang,andCihangXie. Clipa-v2: Scal-
InNAACL,2019. ingcliptrainingwith81.1arXivpreprintarXiv:2306.15658,
| formersforlanguageunderstanding. |              |       |        |           |     |             | 1   |       |     |     |     |     |     |     |
| -------------------------------- | ------------ | ----- | ------ | --------- | --- | ----------- | --- | ----- | --- | --- | --- | --- | --- | --- |
| [15] Alexey                      | Dosovitskiy, | Lucas | Beyer, | Alexander |     | Kolesnikov, |     | 2023. | 3,4 |     |     |     |     |     |
Dirk Weissenborn, Xiaohua Zhai, Thomas Unterthiner, [31] XianhangLi,ZeyuWang,andCihangXie. Aninversescal-
MostafaDehghani,MatthiasMinderer,GeorgHeigold,Syl- inglawforcliptraining. InNeurIPS,2023. 3,4,5
vainGelly,JakobUszkoreit,andNeilHoulsby. Animageis [32] YanghaoLi,HaoqiFan,RonghangHu,ChristophFeichten-
worth16x16words: Transformersforimagerecognitionat hofer,andKaimingHe.Scalinglanguage-imagepre-training
| scale. | InICLR,2021. | 5   |     |     |     |     |     | viamasking. |     | InCVPR,2023. |     | 3,4,5,6 |     |     |
| ------ | ------------ | --- | --- | --- | --- | --- | --- | ----------- | --- | ------------ | --- | ------- | --- | --- |

[33] ZichaoLi,LiLiu,ZeyuWang,YuyinZhou,andCihangXie. [50] Christoph Schuhmann, Richard Vencu, Romain Beaumont,
Bag of tricks for fgsm adversarial training. arXiv preprint Robert Kaczmarczyk, Clayton Mullis, Aarush Katta, Theo
arXiv:2209.02684,2022. 2 Coombes,JeniaJitsev,andAranKomatsuzaki.Laion-400m:
[34] Chang Liu, Yinpeng Dong, Wenzhao Xiang, Xiao Yang, Open dataset of clip-filtered 400 million image-text pairs.
HangSu, JunZhu, YuefengChen, YuanHe, HuiXue, and arXivpreprintarXiv:2111.02114,2021. 4,6
Shibao Zheng. A comprehensive study on robustness of [51] Ali Shafahi, Mahyar Najibi, Mohammad Amin Ghiasi,
imageclassificationmodels: Benchmarkingandrethinking. ZhengXu,JohnDickerson,ChristophStuder,LarrySDavis,
arXivpreprintarXiv:2302.14301,2023. 2,7,8 GavinTaylor, andTomGoldstein. Adversarialtrainingfor
[35] ZeLiu,YutongLin,YueCao,HanHu,YixuanWei,Zheng free! NeurIPS,32,2019. 2,3
Zhang, Stephen Lin, and Baining Guo. Swin transformer: [52] NamanDSingh,FrancescoCroce,andMatthiasHein. Re-
Hierarchical vision transformer using shifted windows. In visiting adversarial training for imagenet: Architectures,
ICCV,2021. 2,7 trainingandgeneralizationacrossthreatmodels.InNeurIPS,
[36] Ze Liu, Han Hu, Yutong Lin, Zhuliang Yao, Zhenda Xie, 2023. 2,7,8
YixuanWei,JiaNing,YueCao,ZhengZhang,LiDong,etal. [53] ChristianSzegedy,WojciechZaremba,IlyaSutskever,Joan
Swintransformerv2:Scalingupcapacityandresolution. In Bruna,DumitruErhan,IanGoodfellow,andRobFergus.In-
CVPR,2022. 2 triguingpropertiesofneuralnetworks. InICLR,2014. 3
[37] ZhuangLiu,HanziMao,Chao-YuanWu,ChristophFeicht- [54] Yonglong Tian, Dilip Krishnan, and Phillip Isola. Con-
enhofer,TrevorDarrell,andSainingXie. Aconvnetforthe trastivemultiviewcoding. InECCV,2020. 4
2020s. InCVPR,2022. 2,7 [55] Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier
Martinet,Marie-AnneLachaux,Timothe´eLacroix,Baptiste
[38] IlyaLoshchilovandFrankHutter. Decoupledweightdecay
Rozie`re, Naman Goyal, Eric Hambro, Faisal Azhar, et al.
regularization. InICLR,2019. 3,5
Llama: Open and efficient foundation language models.
[39] Aleksander Madry, Aleksandar Makelov, Ludwig Schmidt,
arXivpreprintarXiv:2302.13971,2023. 1
DimitrisTsipras,andAdrianVladu. Towardsdeeplearning
[56] Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert,
modelsresistanttoadversarialattacks. InICLR,2018. 1
Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov,
[40] Yichuan Mo, Dongxian Wu, Yifei Wang, Yiwen Guo, and
Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, et al.
YisenWang. Whenadversarialtrainingmeetsvisiontrans-
Llama2:Openfoundationandfine-tunedchatmodels.arXiv
formers: Recipes from training to architecture. NeurIPS,
preprintarXiv:2307.09288,2023. 1
2022. 2
[57] ZekaiWang,TianyuPang,ChaoDu,MinLin,WeiweiLiu,
[41] OpenAI. Gpt-4 technical report. ArXiv, abs/2303.08774,
andShuichengYan.Betterdiffusionmodelsfurtherimprove
2023. 1,2
adversarialtraining. InICML,2023. 2
[42] TianyuPang,XiaoYang,YinpengDong,HangSu,andJun
[58] EricWong,LeslieRice,andJ.ZicoKolter.Fastisbetterthan
Zhu. Bagoftricksforadversarialtraining. InICLR,2021. 2
free:Revisitingadversarialtraining. InICLR,2020. 2
[43] ShengYunPeng,WeilinXu,CoryCornelius,KevinLi,Rahul
[59] Chaowei Xiao, Bo Li, Jun-Yan Zhu, Warren He, Mingyan
Duggal, Duen Horng Chau, and Jason Martin. Robarch:
Liu,andDawnSong. Generatingadversarialexampleswith
Designing robust architectures against adversarial attacks.
adversarialnetworks. IJCAI,2018. 3
arXivpreprintarXiv:2301.03110,2023. 8
[60] CihangXieandAlanYuille. Intriguingpropertiesofadver-
[44] OmidPoursaeed,IsayKatsman,BichengGao,andSergeBe- sarialtrainingatscale. InICLR,2020. 2
longie.Generativeadversarialperturbations.InCVPR,2018.
[61] Cihang Xie, Yuxin Wu, Laurens van der Maaten, Alan L.
3
Yuille, and Kaiming He. Feature denoising for improving
[45] Alec Radford, Karthik Narasimhan, Tim Salimans, Ilya adversarialrobustness. InCVPR,2019.
Sutskever,etal. Improvinglanguageunderstandingbygen- [62] CihangXie,MingxingTan,BoqingGong,AlanYuille,and
erativepre-training. OpenAIblog,2018. 8 Quoc V Le. Smooth adversarial training. arXiv preprint
[46] AlecRadford,JeffreyWu,RewonChild,DavidLuan,Dario arXiv:2006.14536,2020. 2
Amodei, Ilya Sutskever, et al. Language models are unsu- [63] Sangdoo Yun, Dongyoon Han, Seong Joon Oh, Sanghyuk
pervisedmultitasklearners. OpenAIblog,2019. Chun, Junsuk Choe, and Youngjoon Yoo. Cutmix: Regu-
[47] Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya larizationstrategytotrainstrongclassifierswithlocalizable
Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, features. InICCV,2019. 5
AmandaAskell,PamelaMishkin,JackClark,etal. Learn- [64] XiaohuaZhai,AlexanderKolesnikov,NeilHoulsby,andLu-
ingtransferablevisualmodelsfromnaturallanguagesuper- casBeyer. Scalingvisiontransformers. InCVPR,2022. 1,
vision. InICML,2021. 3,4,5,7,8 2
[48] Yongming Rao, Wenliang Zhao, Guangyi Chen, Yansong [65] Hongyi Zhang, Moustapha Cisse, Yann N. Dauphin, and
Tang, Zheng Zhu, Guan Huang, Jie Zhou, and Jiwen Lu. DavidLopez-Paz. mixup: Beyondempiricalriskminimiza-
Denseclip: Language-guideddensepredictionwithcontext- tion. InICLR,2018. 5
awareprompting. InCVPR,2022. 4 [66] Yiwu Zhong, Jianwei Yang, Pengchuan Zhang, Chunyuan
[49] Sylvestre-AlviseRebuffi,FrancescoCroce,andSvenGowal. Li,NoelCodella,LiunianHaroldLi,LuoweiZhou,Xiyang
Revisitingadapterswithadversarialtraining.InICLR,2023. Dai, Lu Yuan, Yin Li, et al. Regionclip: Region-based
2,8 language-imagepretraining. InCVPR,2022. 4

| [67] Xingyi                         | Zhou, | Rohit Girdhar,   | Armand    | Joulin,         | Philipp |
| ----------------------------------- | ----- | ---------------- | --------- | --------------- | ------- |
| Kra¨henbu¨hl,                       |       | and Ishan Misra. | Detecting | twenty-thousand |         |
| classesusingimage-levelsupervision. |       |                  |           | InECCV,2022.    | 4       |
