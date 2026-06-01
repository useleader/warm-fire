---
publish: true
---

1
Attacks in Adversarial Machine Learning:
A Systematic Survey from the Life-cycle Perspective
Baoyuan Wu, Senior Member, IEEE, Zihao Zhu, Li Liu, Member, IEEE, Qingshan Liu, Senior Member, IEEE,
Zhaofeng He, Member, IEEE, Siwei Lyu, Fellow, IEEE
Abstract—Adversarial machine learning (AML) studies the model could give totally different predictions on two visually
adversarial phenomenon of machine learning, which may make similar images, as one of them is perturbed by imperceptible
inconsistent or unexpected predictions with humans. Some
and malicious noises [86], while human’s prediction will not
paradigmshavebeenrecentlydevelopedtoexplorethisadversarial
be influenced by such noises. We call this phenomenon as
phenomenon occurring at different stages of a machine learning
system, such as backdoor attack occurring at the pre-training, adversarial phenomenon, indicating the adversary between
in-training and inference stage; weight attack occurring at MLmodelsandhumans.Unlikeregularmachinelearningwhich
the post-training, deployment and inference stage; adversarial focuses on improving the consistency between ML models and
attack occurring at the inference stage. However, although these
humans, adversarial machine learning (AML) focuses on
adversarial paradigms share a common goal, their developments
exploringtheadversarialphenomenon.Unfortunately,duetothe
are almost independent, and there is still no big picture of AML.
Inthiswork,weaimtoprovideaunifiedperspectivetotheAML black-box mechanism of modern ML models (especially deep
community to systematically review the overall progress of this neuralnetworks),itisdifficulttoprovidehuman-understandable
field.WefirstlyprovideageneraldefinitionaboutAML,andthen explanations for their decisions, neither the consistency nor
propose a unified mathematical framework to covering existing
the inconsistency with humans. Consequently, the existence of
attack paradigms. According to the proposed unified framework,
the adversarial phenomenon in ML becomes one of the main
we build a full taxonomy to systematically categorize and review
existingrepresentativemethodsforeachparadigm.Besides,using obstacles to obtain the trust of humans. Meanwhile, due to
this unified framework, it is easy to figure out the connections the importance and challenges of the adversarial phenomenon,
and differences among different attack paradigms, which may it has attracted many researchers’ attention, and AML has
inspire future researchers to develop more advanced attack become an emerging topic in the ML community.
paradigms.Finally, tofacilitatethe viewingof thebuilttaxonomy
As shown in Figure 1, we divide the full life-cycle of the
and the related literature in adversarial machine learning, we
further provide a website, i.e., http://adversarial-ml.com, where machine learning system into five stages, i.e.pre-training stage,
the taxonomies and literature will be continuously updated. training stage, post-training, deployment stage, and inference
stage. In the literature of AML, several different paradigms
Index Terms—Adversarial machine learning, training-time
adversarial attack, deployment-time adversarial attack, inference- have been developed to explore the adversarial phenomenon at
time adversarial attack different stages, mainly including backdoor attacks occurring
at the pre-training, training, and inference stage, weight attacks
occurring at the post-training, deployment, and inference stage,
I. INTRODUCTION
and adversarial examples occurring at the inference stage1.
MACHINE learning (ML) aims to learn a machine/model Although with the same goal, the developments of these attack
from the data, such that it can act like humans when paradigms are almost independent. Without a comprehensive
handling new data. It has achieved huge progress on lots of development of different paradigms, we believe that it will
importantapplicationsinthelastdecade,especiallywiththerise be difficult to comprehensively and deeply understand the
of deep learning from 2006, such as computer vision, natural adversarial phenomenon of ML, making it difficult to truly
language processing, speech recognition, etc. It seems that improve the adversarial robustness of ML. In this survey, we
ML has been powerful enough to satisfy humans’ expectations attempt to provide a unified perspective on the attack aspect of
in practice. However, a disturbing phenomenon found in recent AML on the image classification task, based on the life-cycle
years shows that sometimes ML models may make abnormal of the machine learning system.
predictionsthatareinconsistentwithhumans.Forexample,the Our Goal and Contributions. After years of prosperous
but isolated development with a bottom-up manner (i.e., from
Baoyuan Wu and Zihao Zhu are with the School of Data Sci- different perspectives to the same destination), we believe
ence, The Chinese University of Hong Kong, Shenzhen, China, email:
it is important to make a comprehensive top-down review
wubaoyuan@cuhk.edu.cn, zihaozhu@link.cuhk.edu.cn. Li Liu is with the
HongKongUniversityofScienceandTechnology(Guangzhou),China,email: of current progress in the AML area. To achieve this goal,
avrillliu@hkust-gz.edu.cn.QingshanLiu,email:qsliu@nuist.edu.cn.Zhaofeng we firstly provide a general definition of AML, and then
HeiswiththeSchoolofArtificialIntelligence,BeijingUniversityofPostsand
present a unified mathematical framework to cover diverse
Telecommunications,Beijing,China,email:zhaofenghe@bupt.edu.cn.Siwei
LyuiswiththeDepartmentofComputerScienceandEngineering,University formulations of existing branches. Moreover, according to
atBuffalo,StateUniversityofNewYork(SUNY),Buffalo,NY14260USA,
email:siweilyu@buffalo.edu. 1Notethatinourpaper,weuseadversarialattackstorefertoalltheseattacks
inAML.Toavoidconfusionwithtraditionalinference-timeadversarialattacks,
Correspondingauthor:BaoyuanWu(wubaoyuan@cuhk.edu.cn). weuseadversarialexamplestodenoteinference-timeadversarialattacks.
4202
naJ
4
]GL.sc[
2v75490.2032:viXra

2
the unified framework, we build systematic categorizations
of existing diverse works. Although there been several surveys
about adversarial examples (e.g., [5], [290]) or backdoor
learning (e.g., [80], [247]), the main point that distinguishes
our survey from existing surveys is the unified definition and
1
mathematical framework of AML, which could bring in two
5 2
main contributions to the community. 1) The systematic
perspectiveprovidedbytheunifiedframeworkcouldhelpusto
quickly overview the big picture of AML, to avoid one-sided
AML
or biased understanding of AML. 2) According to the unified
life-cycle
framework, the intrinsic connections among different AML
4 3
branches are built to provide a broader view for researchers in
each individual branch, such that the developments of different
branches could be calibrated to accelerate the overall progress
of AML.
Organization. The remaining contents are organized as
follows: Section II introduces the general definition, unified
mathematical formulation, and three learning paradigms of
AML. Section III reviews adversarial attacks occurring at the
pre-training stage, mainly including data-poisoning based back-
Fig.1. Thefulllife-cycleofAdversarialMachineLeaning
door attacks. Section IV covers adversarial attacks occurring
at the training stage, mainly including training-controllable
basedbackdoorattacks.InsectionV,weinvestigateadversarial ε to indicate benign and adversarial data/model, respectively.
attacks occurring at the post-training stage, mainly including The detailed notations are summarized in Table I.
weight attacks via parameter-modification. Section V reviews
Definition 1 (Adversarial Machine Learning (AML)). AML
adversarial attacks occurring at the deployment stage, mainly
is an emerging sub-area of machine learning to study the
including weight attacks via bit-flip. Section VII reviews
adversarial phenomenon of machine learning. AML is defined
adversarial attacks occurring at the deployment stage, mainly
upon three basic conditions, including stealthiness S, benign
includingadversarialexamples.Exceptforimageclassification,
consistency C and adversarial inconsistency I, as follows:
we also review adversarial attacks in other scenarios (e.g.,
1) Stealthiness S(x ,x ;w ,w ): It captures the condition
diffusion models and large language models) in Section 0 ε 0 ε
that the change between benign and adversarial samples,
VIII. Section IX and X present the applications and further
or that between benign and adversarial weights should be
discussions of adversarial attacks respectively, followed by the
stealthy, while the specific definition and formulation of
summary in Section XI. This survey only covers the attack
stealthiness will lead to different variants.
part of AML. For the defense part, the readers can refer to our
2) Benign consistency C(x ,y ;w ,w ): It captures the
other survey [249]. 0 0 0 ε
condition of prediction consistency on the benign data pair
(x ,y ) between human and benign or adversarial models.
0 0
II. WHATISADVERSARIALMACHINELEARNING
3) Adversarial inconsistency I(x ,y ;w ,w ): It captures
ε ε 0 ε
the condition of prediction inconsistency on the adversarial
TABLEI
BASICNOTATIONS. data pair (x ε ,y ε ) between human and benign or adversarial
Notation NameandDescription models, which reflects the goal of adversarial machine
Benigndataset,i.e.,{(x(i),y(i))}N0,where(x(i),y(i))iscalledabenign learning.
0 0 i=1 0 0
D0 data(BD),withx(
0
i)beingabenignsample(BS)andy
0
(i)beingabenign
label. General formulation. Based on Definition 1, a general
Dε A ad d v v e e r r s s a a r r ia ia l l d d a a ta ta ( s A et D ,i ), .e w ., it { h (x x ε ( ε ( i i ) ) , b y e ε ( i i n )) g } a N i= n ε 1 a , d w ve h r e sa re ria ( l x s ( ε a i) m , p y l ε ( e i) ( ) A i S s ) c a a n ll d ed yε ( a i n ) formulation of AML could be written as follows:
beinganadversariallabel.
fw0 (·)
B
ca
e
l
n
le
ig
d
n
B
m
M
o
,
d
w
el
it
(
h
BM
the
):
w
if
e
o
ig
n
h
e
t
m
w
o
0
delisregularlytrainedbasedonD0,thenitis ar
x
g
ε,
m
wε
in S(x
0
,x
ε
;w
0
,w
ε
)+C(x
0
,y
0
;w
0
,w
ε
)+ (1)
fwε (·) A m d o v d e ifi rs e a d r , ia o l r m on o e de m l o (A de M l ) i : s I t f ra o in n e e d b b e a n s i e g d n o m n o D de ε l( ( B or M m ) i f x w tu 0 re (· o ) f is D m ε a a l n ic d io D us 0 ly ), I(x ε ,y ε ;w 0 ,w ε ).
thenthemodified/trainedmodeliscalledAM,withtheweightwε
Specification of each term will lead to different AML
paradigms.
A. General Definition and Formulation
Notations. We denote a machine learning model as f :X →
w
Y,withwindicatingmodelweight,X ⊂Rd theinputspace(d B. Three Attack Paradigms at Different Stages of AML
indicating the input dimension), and Y ⊂R the output space, The life-cycle of a machine learning system mainly consists
respectively. A data pair is denoted as (x,y), with x ∈ X of five stages, including pre-training, training, post-training,
being the input sample and y ∈ Y being the corresponding deployment, and inference stage. According to the stages
label. Furthermore, for clarity, we introduce subscripts 0 and at which the adversarial phenomenon exists, AML can be

3
TABLEII
SUMMARYOFALLSPECIFIEDFORMULATIONSINTHREEATTACKPARADIGMSOFAML.
Condition Specifications Description
S(x0,xε;w0,wε)
A
A
M
M
L
L
.
.
S
S
w
x:
:
D
D
x
w
(x
(w
0,
0
x
,
ε
w
)
ε) S
S
t
t
e
e
a
a
l
l
t
t
h
h
i
i
n
n
e
e
s
ss
so
o
f
f
w
sa
e
m
ig
p
h
l
t
e
p
p
e
e
r
r
t
t
u
u
r
r
b
b
a
a
t
t
i
i
o
o
n
n
:
:
e
e
n
n
c
c
o
o
u
u
r
r
a
a
g
g
i
i
n
n
g
g
s
s
m
m
a
a
l
l
l
l
d
d
i
i
f
f
f
f
e
e
r
r
e
e
n
n
c
c
e
e
b
b
e
e
t
t
w
w
e
e
e
e
n
n
w
x0
0
a
a
n
n
d
d
x
w
ε
ε
C(x0,y0;w0,wε)
AML.CA:LCA (fwε (x0),y0)
B
y0
enignconsistency1:predictionconsistencyonBSbetweenAMandhuman,whichencouragesfwε (x0)tobe
AML.CB:LCB (fw0 (x0),y0)
B
y0
enignconsistency2:predictionconsistencyonBSbetweenBMandhuman,whichencouragesfw0 (x0)tobe
I(xε,yε;w0,wε)
AML.IA:LIA (fwε (xε),yε)
t
A
o
d
b
v
e
er
t
s
h
a
e
ri
t
a
a
l
rg
in
et
co
la
n
b
s
e
is
l
te
y
n
ε
cy1:predictioninconsistencyonASbetweenAMandhuman,whichencouragesfwε (xε)
AML.IB:LIB (fw0 (xε),yε)
t
A
o
d
b
v
e
er
t
s
h
a
e
ri
t
a
a
l
rg
in
et
co
la
n
b
s
e
is
l
te
y
n
ε
cy2:predictioninconsistencyonASbetweenBMandhuman,whichencouragesfw0 (xε)
R1(fw0 (x0),fw0 (xε)) RegularizationforencouragingsomekindsofsimilaritybetweenBSandASaccordingtoBM
R2(fwε (x0),fwε (xε)) RegularizationforencouragingsomekindsofsimilaritybetweenBSandASaccordingtoAM
Others R3(fwε (x0),fw0 (x0)) RegularizationforencouragingsomekindsofsimilaritybetweenBMandAMonBS
Zx(xε) Constraintonxε,suchastherepresentationdomain
Zw(wε) Constraintonwε,suchassparsityorbounded
categorized to three attack paradigms, as shown in Figure 1: model is deployed in the hardware device (e.g., intelligent
mobile or camera). In this case, the attack can modify the
1) Backdoor attacks: It aims to generate an adversarial model parameters in the memory by flipping bit values in
model f (·) (also called backdoored model), such that at the the discrete space, dubbed as weight attack injection via
wε
inference stage, it performs well on benign data x , while bit-flip. Both these attacks include weight attack activation
0
predicts the adversarial sample x as the target label y . It is at the inference stage. Its formulation could be obtained by
ε ε
implementedbymanipulatingthetrainingdatasetorthetraining specifying the general formulation (1) of AML as follows:
procedure by the attacker. According to whether the attacker

has control over the training process, the backdoor attack S(x 0 ,x ε ;w 0 ,w ε )= D x (x 0 ,x ε )+D w (w 0 ,w ε ),
can further be divided into data-poisoning based backdoor C(x ,y ;w ,w )=L (f (x ),y )+L (f (x ),y ),
0 0 0 ε CB w0 0 0 CA wε 0 0
attackandtraining-controllablebasedbackdoorattack.TheI(x
,y ;w ,w )= L (f (x ),y ).
former mainly focuses on the poisoned sample injection at
ε ε 0 ε IA wε ε ε
(3)
the pre-training stage, while the latter mainly focuses on the
training-controllable based backdoor injection at the training Since the weight attacks require both the benign model
stage. Both these attacks include backdoor activation at the f (·) and the benign data (x ,y ) as inputs, and outputs the
inference stage. Its formulation is derived by specifying the ad w v ε ersarial model f (·) or th 0 e a 0 dversarial sample x , both
general formulation (1) as follows: w and w occur in w a ε bove equations. Similar to D (x ε ,x ),
ε 0 x 0 ε
  S(x 0 ,x ε ;w 0 ,w ε )= D x (x 0 ,x ε ), D
m
2
o
(
d
w
el
0 ,
w
w
e
ε
i
)
gh
e
t
nc
w
oura
s
g
h
e
o
s
ul
t
d
he
b
s
e
tea
c
l
l
t
o
h
s
i
e
nes
to
s t
t
h
h
a
e
t t
b
h
e
e
ni
a
g
d
n
ve
m
rsa
o
r
d
ia
e
l
l
C(x ,y ;w ,w )= L (f (x ),y ), (2) ε
0 0 0 ε CA wε 0 0 weight w . L (f (x ),y ) ensures that the attacked model
I(x
,y ;w ,w )= L (f (x ),y ).
0 CB w0 0 0
ε ε 0 ε IA wε ε ε f
w0
(·) must perform normally on benign data (x
0
,y
0
), which
D x (x 0 ,x ε ) encourages the stealthiness that the poisoned is a hard constraint. The effects of L CA (f wε (x 0 ),y 0 ) and
sample x ε should be similar with the benign sample x 0 . L IA (f wε (x ε ),y ε ) have been described in the above paragraph.
L (f (x ),y ) ensures that the prediction on x by f (·) 3) Adversarial examples: It describes that given a benign
CA wε 0 0 0 wε
should be consistent with the ground-truth label y 0 which model f w0 (·), the attacker aims at slightly modifying one
is annotated by humans. L IA (f wε (x ε ),y ε ) ensures that the benignsamplex 0 toobtainacorrespondingadversarialsample
prediction on x ε by f wε (·) should be the adversarial label x ε , such that the prediction f w0 (x ε ) is different with the
y ε , which is inconsistent with y 0 . Since the backdoor attack ground-truth label y 0 or same with the adversarial label y ε .
doesn’t require a benign model f (·) as input, the benign Differentfrombackdoorattacksandweightattacks,adversarial
w0
model weight w doesn’t occur in the above equations. examples only happen at the inference stage. Its formulation
0
2) Weight attacks: It describes that given the benign could be obtained by specifying the general formulation (1) of
model f (·) trained on the benign dataset D , the attacker AML as follows:
w0 0
aims at slightly modifying the model parameters to obtain

S(x ,x ;w ,w )= D (x ,x ),
an adversarial model f
wε
(·). Consequently, at the inference  0 ε 0 ε x 0 ε
stage, its predictions on adversarial inputs or target benign C(x 0 ,y 0 ;w 0 ,w ε )= L CB (f w0 (x 0 ),y 0 ), (4)
inputs become the target label y ε , while the predictions on I(x ε ,y ε ;w 0 ,w ε )= L IB (f w0 (x ε ),y ε ).
other benign inputs are still their ground-truth labels. Weight
attacks can occur both at the post-training and deployment Since the inference-time adversarial example is conducted only
stages. At the post-training stage, the attacker has the authority on the benign model, the adversarial model weight w doesn’t
ε
to directly modify the parameters of benign model in the occur in above equations. L (f (x ),y ) encourages that
IB w0 ε ε
continuous space, dubbed as weight attack injection via the prediction on x by f (·) should be the adversarial label
ε w0
parameter-modification. At the deployment stage, the benign y , which is inconsistent with y .
ε 0

4
For clarity, we summarize all specified formulations pre- a) Visible trigger: Visible trigger means that the mod-
sentedinEqs.(2),(3),(4)inTableII,aswellassomeadditional ification of the original sample x −x can be realized by
ε 0
regularization or constraints. human visual perception, but it will not interfere with human’s
prediction, i.e., a human can always predict the correct label
III. ATTACKATTHEPRE-TRAININGSTAGE regardless of whether there is a trigger or not. The first visible
Before training large-scale deep models, i.e., pre-training trigger is adopted by BadNets [88] designed, which generates
stage, it is necessary to collect the training dataset and then poisoned image x ε by stamping a small visible grid patch or
preprocess the dataset to adapt the model. In practice, the user a sticker (e.g., yellow square, bomb, flower) on the benign
may download an open-sourced dataset from an unverified image x 0 . Since that, the triggers with similar visible patterns
source or buy data from an untrustworthy third-party data have been widely used in many subsequent works [141], [173],
supplier. Considering the data scale, it is difficult to thoroughly [196]. Besides, in the backdoor attacks against the 3D cloud
check the data quality, or distinguish malicious noises from point classification task, the visible additional 3D points are
random noises. In this scenario, the attacker has the chance to adopted as the backdoor trigger in a few existing works [129],
manipulatepartialdatatogeneratepoisonedsamplestoachieve [257].
themaliciousgoal.Aftertrainingwiththepoisoneddataset,the b) Invisible trigger: Although the visible trigger seems
backdoor can be injected into the trained model. We refer to to be harmless from a human’s perspective, its high-frequency
this type of attack as data-poisoning based backdoor attack. presenceinmultiplesampleswiththesamelabelmaystillraise
Data-poisoning based backdoor attacks can be separated human suspicion. Therefore, some invisible triggers have been
into two independent tasks: malicious data poisoning (i.e., developed to make backdoor attacks less detectable by human
generating poisoned samples) and normal model training inspection, while maintaining the high attack success rate.
respectively. Since the attacker can only access and manipulate There are four main strategies to achieve trigger invisibility.
thetrainingdataset,whilethetrainingprocessisoutofcontrol, • Alpha blending: Blended [41] firstly adopts the alpha
we mainly focus on the former task (dubbed poisoned data blendingstrategytofusethetriggerintothebenignimage.
injection) in this section. Specifically, the g 1 function in Eq. (5) is specified as
the α-blending function, i.e., x =αx +(1−α), with
ε 0
α∈[0,1],andthetriggervisibilityisnegativelycorrelated
A. Formulation and Categorization
with α.
Formulation. Data-poisoning based backdoor generation
• Digital steganography: [8], [13]: it is the technology
focuses on generating poisoned samples D = {(x ,y )},
ε ε ε of concealing secret information into some digital media
which can be formulated as follows:
(e.g.,image,video),whileavoidingobviouschangesinthe
(cid:0) (cid:1)
x =g g (ε),g (D ) , y =g (y ) (5) media.Bytreatingthetriggerassecretinformation,digital
ε 3 1 2 0 ε 4 0
steganography is a perfect tool to generate an invisible
where g (·) denotes the generation of triggers, g (·) denotes
1 2 trigger. For example, Li et al. [126] utilize the widely
the selection of benign samples to be poisoned, g (·) denotes
3 usedsteganographyalgorithm,i.e.,theleastsignificantbit
the fusion of triggers and selected samples, g (·) denotes the
4 (LSB) substitution [27], to insert the trigger information
generation of target labels of poisoned samples.
into the least significant bit of one pixel to avoid the
Categorization. As shown in Figure 2, according to the
visible change in the RGB space. The sample-specific
specifications of (g ,g ,g ,g ), we categorize existing works
1 2 3 4 backdoor attack (SSBA) method [134] adopts a double-
of data-poisoning based backdoor attacks into the following
loop auto-encoder [220] that is firstly proposed for digital
four sub-branches:
steganography, to merge the trigger information into the
1) Backdoorattackswithdifferenttriggersaccordingtog (·),
1 benign image, such that invisible and sample-specific
which will be introduced in Section III-B;
triggers could be generated.
2) Backdoor attacks with different selection strategies ac-
• Adversarialperturbation:sincemosttypesofadversarial
cording to g (·), which will be introduced in Section
2 perturbations are imperceptible to humans, they can be
III-C;
used as effective tools to generate invisible triggers. One
3) Backdoorattackswithdifferentfusionstrategiesaccording
typical example is AdvDoor [293], which adopts the
to g (·,·), which will be introduced in Section III-D;
3 targeted universal adversarial perturbation (TUAP) [162]
4) Backdoor attacks with different target classes according
asthetrigger.Theinvisibilityandthestablemappingfrom
to g (·), which will be introduced in Section III-E.
4 the TUAP to the target class satisfied the requirement
of the backdoor attack with invisible triggers. Besides,
B. Trigger Generation adversarial perturbation is a commonly used technique
Triggergenerationaimstogeneratetriggers(i.e.,g (·)inEq. in label-consistent backdoor attacks (e.g., [176], [200]).
1
(5) that are used to fuse with the benign samples. According to The general idea is that the original feature of a target
thecharacteristicsoftriggers,existingworkscanbecategorized imageiserasedbyinvisibleadversarialperturbation,while
from the following perspectives. the feature of a source image with trigger is inserted.
1) Visible v.s. Invisible Trigger: According to the trigger Consequently, the generated poisoned image has a similar
visibility with respect to human’s visual perception, the trigger visualappearancetothetargetimage,butasimilarfeature
can be categorized into visible and invisible trigger. to source image with trigger, and is labeled as the target

5
Fig.2. Taxonomyofbackdoorattacksatthepre-training,training,andinferencestage.
| class. |     |     |     |     |     |     | 3) Manually | designed |     | Trigger | v.s. | Learnable |     | Trigger: |
| ------ | --- | --- | --- | --- | --- | --- | ----------- | -------- | --- | ------- | ---- | --------- | --- | -------- |
Slight transformation: as human eyes are insensitive to Accordingtohowtriggersaregenerated,wecategorizeexisting
•
slight spatial or color distortion, some slight transforma- works into manually designed and learnable trigger.
tions are adopted as triggers, such as the image warping a) Manually designed trigger: : Manually designed
in [174], or style transfer in [45]. trigger means that the trigger is manually specified by the
|                 |             |              |          |              |             |     | attacker, such | as        | grid square | trigger   | [88], | cartoon   | pattern | [41],    |
| --------------- | ----------- | ------------ | -------- | ------------ | ----------- | --- | -------------- | --------- | ----------- | --------- | ----- | --------- | ------- | -------- |
| 2) Non-semantic |             | v.s.         | Semantic | Trigger:     | According   |     | to             |           |             |           |       |           |         |          |
|                 |             |              |          |              |             |     | random noise   | [196],    | ramp        | signal    | [14], | 3D binary | pattern | [237],   |
| whether         | the trigger | has          | semantic | meaning,     | the trigger | can |                |           |             |           |       |           |         |          |
|                 |             |              |          |              |             |     | etc. When      | designing | these       | triggers, | the   | attacker  | often   | doesn’t  |
| be categorized  | into        | non-semantic |          | and semantic | trigger.    |     |                |           |             |           |       |           |         |          |
|                 |             |              |          |              |             |     | take into      | account   | the benign  | training  |       | dataset   | to be   | poisoned |
a) Non-semantic trigger: Non-semantic trigger means or any particular model, thus there is no guarantee about the
| that the | trigger | has no | semantic | meaning, | such as | a small |              |                  |     |     |       |           |     |     |
| -------- | ------- | ------ | -------- | -------- | ------- | ------- | ------------ | ---------------- | --- | --- | ----- | --------- | --- | --- |
|          |         |        |          |          |         |         | stealthiness | or effectiveness |     | of  | these | triggers. |     |     |
checkerboard grid or random noise. Since most of the existing b) Learnable trigger: Learnable trigger also called
backdoor attacks adopt non-semantic triggers, here we don’t optimization-based trigger, denotes that the trigger is generated
| expand the | descriptions. |     |     |     |     |     |                    |     |              |     |          |      |            |        |
| ---------- | ------------- | --- | --- | --- | --- | --- | ------------------ | --- | ------------ | --- | -------- | ---- | ---------- | ------ |
|            |               |     |     |     |     |     | through optimizing |     | an objective |     | function | that | is related | to the |
b) Semantic trigger: Semantic trigger means that the benign sample or a model, to achieve some particular goals
trigger corresponds to some semantic objects with particular (e.g., enhancing the stealthiness or attack success rate). For
attributes contained in the benign sample, such as the red car example, in label-consistent attacks [200], the trigger is often
|               |     |              |      |        |                |      | generated | by minimizing |     | the distance |     | between | the | poisoned |
| ------------- | --- | ------------ | ---- | ------ | -------------- | ---- | --------- | ------------- | --- | ------------ | --- | ------- | --- | -------- |
| in one image, | or  | a particular | word | in one | sentence. This | kind |           |               |     |              |     |         |     |          |
of semantic trigger is first adopted in backdoor attacks against sampleandthetargetbenignsampleintheoriginalinputspace,
severalsecurity-criticalnaturallanguageprocessing(NLP)tasks and the distance between the poisoned sample and the benign
(e.g., sentiment analysis [26] or text classification [26], toxic source sample in the feature space of a pre-trained model.
comment detection [294], neural machine translation (NMT), Besides, another typical optimized trigger is the universal
|           |       |              |      |                 |          |     | adversarial | perturbation |     | w.r.t. | the | target | class (e.g., | [293], |
| --------- | ----- | ------------ | ---- | --------------- | -------- | --- | ----------- | ------------ | --- | ------ | --- | ------ | ------------ | ------ |
| and [42], | where | a particular | word | or a particular | sentence | was |             |              |     |        |     |        |              |        |
used as the trigger. Then, the semantic trigger was extended [306], [301]), which is optimized based on a set of benign
into the computer vision tasks, where some particular semantic samples and a pre-trained model.
objects in the benign image were treated as the trigger, such 4) Digital v.s. Physical Trigger: According to the scenario
as “cars with racing stripe” [10]. VSSC [230] first proposes to in which the trigger works, existing works can be categorized
edit the source image by image editing methods to generate to digital and physical trigger.
semantictriggersthatareinharmonywiththeremainingvisual a) Digital trigger: Most existing backdoor attack works
content in the image to ensure visual stealthiness. Since the onlyconsiderthedigitaltrigger,i.e.,thetriggerinbothtraining
semantic trigger is chosen among the objects that already exist and inference stages only exist in digital space.
in the benign image, a unique feature of this kind of backdoor b) Physical trigger: In contrast, the physical backdoor
attack is that the input image is not modified, while only attack where some physical objects are used as the trigger
the label is changed to the target class, which increases the at the inference stage has been rarely studied. There are a
stealthiness, compared to backdoor attacks with non-semantic few attempts focusing on some particular tasks, such as face
triggers. recognition or autonomous driving. For example, the work

6
[243]presentsadetailedempiricalstudyofthebackdoorattack D. Trigger Fusion Strategy
against the face recognition model in the physical scenario.
According to different fusion strategies that fuse the triggers
Seven physical objects at different facial locations are used
andselectedsamples,i.e.,g (·,·)inEq.(5),wecancategorize
3
as triggers, and the studies reveal that the trigger location is
existing works from the following perspectives.
criticaltotheattackperformance.Thephysicaltransformations
1) Additive v.s. Non-additive Trigger: According to the
for backdoors (PTB) method [272] also studies the physical
fusion method of trigger and image, the trigger can be
backdoorattackagainstfacerecognition,andintroducesdiverse
categorized into additive and non-additive trigger.
transformations (e.g., distance, rotation, angle, brightness, and
a) Additive trigger: Additive trigger means that the
Gaussian noise) on poisoned facial images, to enhance the
poisoned image is the additive fusion of the benign image and
robustness to distortions in the physical scenario. Besides, the
trigger.Specifically,thefusionfunctiong inEq.(5)isspecified
work [92] explores physical backdoor attacks against lane- 2
asanadditivefunction,i.e.,x =αg (x )+(1−α)g (ε)with
detection models. It designed a set of two traffic cones with ε 0 0 1
α ∈ (0,1). Since most existing triggers belong to this type,
specific shapes and positions as the trigger and changed the
and there is no much variation of g , here we don’t expand
outputlaneinpoisonedsamples.VSSC[230]achievesphysical 1
the details.
attack by automatically editing original images in the digital
b) Non-additivetrigger: Non-additivetriggerdenotesthat
space.
thepoisonedimageisnotthedirectadditivefusionofthetrigger
and the input image, but is generated by some types of non-
additive transformation function. There are two types of trans-
C. Sample Selection Strategy
formations in existing works, including the color/style/attribute
This part focuses on selecting appropriate samples to be transformation, and the spatial transformation. In terms of the
poisoned from the benign dataset D , i.e., g (D ) in Eq. (5). formertype,FaceHack[197]utilizesaparticularfacialattribute
0 2 0
There are two types of strategies adopted by existing works: as the trigger in the face recognition task, such as the age,
random selection and non-random selection strategies. expressionormakeup;DFTS[45]usesaparticularimagestyle
1) RandomSelectionStrategy: Randomselectionisthemost as the trigger. In terms of the latter type, WaNet [174] utilizes
widelyadoptedstrategyinthefieldofbackdoorattacks.Justas image warping as the trigger based on a warping function and
its name implies, the attacker often randomly selects samples a pre-defined warping field.
to be poisoned, disregarding the varying importance of each 2) Sample-agnostic v.s. Sample-specific Trigger: According
poisonedsampleintermsofbackdoorinjection.Theproportion to whether the trigger is dependent on the image, the trigger
of poisoned samples to all training samples, i.e., |D |/|D |, can be categorized into sample-agnostic and sample-specific
ε 0
is called the poisoning ratio. Since most existing backdoor trigger.
attacks adopted this strategy, we don’t expand the details. a) Sample-agnostic trigger: Sample-agnostic trigger
2) Non-random Selection Strategy: Recent studies have means that the trigger x ε −x 0 is independent with the benign
started to explore the importance of different samples for sample x 0 , i.e., x( ε i) −x( 0 i) = x( ε j) −x( 0 j),∀i ̸= j. It could
backdoor attacks and propose different non-random selection be implemented by setting the fusion function g 2 as a linear
strategies to select samples to be poisoned instead of selecting function. Since most existing backdoor attacks adopted this
randomly. Filtering-and-updating strategy (FUS) [255] adopts type, here we don’t expand the details.
forgetting events [223] to indicate the contribution of each b) Sample-specifictrigger: Sample-specifictriggermeans
poisoned sample and iteratively filters and updates a sample thatthetriggerx −x isrelatedtothebenignsamplex ,i.e.,
ε 0 0
pool.Learnablepoisoningsampleselectionstrategy(LPS)[313] x(i)−x(i) ̸=x(j)−x(j),∀i̸=j. It could be implemented by
ε 0 ε 0
learns the mask through a min-max optimization, where the designing a particular fusion function g , or trigger generation
2
inner problem maximizes loss w.r.t. the mask to identify hard function g . In terms of the fusion function, one typical choice
1
poisonedsamplesbyimpedingthetrainingobjective,whilethe isutilizingtheimagesteganographytechnique.Forexample,Li
outer problem minimizes the loss w.r.t. the model parameters. et al. [126] adopt the least significant bit (LSB) steganography
An improved filtering and updating strategy (FUS++) [138] technique [27] to insert the binary code of the trigger into the
combines the forgetting events and curvature of different benign image. Since the least significant bits vary in different
samples to conduct a simple yet efficient sample selection benign images, x −x is specific to each x . The SSBA
ε 0 0
strategy. The representation distance (RD) score is proposed attack [134] adopts a double-loop auto-encoder [220] based
in [253] to identify the poisoning samples that are more steganography technique, to merge the trigger information into
crucial to the success of backdoor attacks. Wu et al. [253] thebenignimagetoobtainspecificx −x foreachx .Another
ε 0 0
propose a confidence-based scoring methodology to measure technique is transformation, where each benign sample after
thecontributionofeachpoisonedsamplebasedonthedistance a particular transformation is treated as one poisoned sample
posteriors and proposed a greedy search algorithm to find the (e.g., [174], [45]), such that x −x is dependent with x .
ε 0 0
most informative samples for backdoor injection. Proxy-Free In terms of the trigger generation function, g could take the
1
Strategy (PFS) [137] utilizes a pre-trained feature extractor to benign sample x as one of the input arguments to generate
0
computethecosinesimilaritybetweencleanandcorresponding sample-specific trigger. For example, Poison ink [289] extracts
poisoned samples and then selects poisoned samples with high a black and white edge image from one benign image, then
similarity and diversity. colorizes the edge image with a particular color as the trigger.

7
Since the edge image is specific in each benign image, the a) Label-inconsistent: Label-inconsistent denotes that the
trigger is also sample-specific. poisoned sample is generated based on benign samples from
3) Static v.s. Dynamic Trigger: According to whether the other classes (i.e., not target class), but its label is changed
|         |         |                  |     |          |             |        | to the target | class, | such | that | the visual | content is inconsistent |
| ------- | ------- | ---------------- | --- | -------- | ----------- | ------ | ------------- | ------ | ---- | ---- | ---------- | ----------------------- |
| trigger | changes | across different |     | samples, | the trigger | can be |               |        |      |      |            |                         |
categorized into static and dynamic trigger. with its label. Since most existing backdoor attacks adopted
a) Static trigger: Static triggerdenotes that the trigger is this setting, here we don’t present more details.
fixed across the training samples, including the pattern and b) Label-consistent: Label-consistent (also called clean-
location. Most early backdoor attacks, such as BadNets [88], label attack) means that the poisoned sample is generated
Blended[41],SIG[14],adoptstatictriggers.However,poisoned based on benign samples from the target class, and the original
samples with static triggers are likely to show very stable label is not changed, such that the visual content of the
and discriminative characteristics compared to benign samples. poisoned sample is consistent with its label. Consequently,
Consequently, these characteristics could be easily identified it is more stealthy than the label-inconsistent poisoned sample
and utilized by the defender. under human inspection. Zhao et al. [301] evaluate the label-
b) Dynamic trigger: Dynamic trigger assumes that there consistent attack against video recognition tasks. The benign
is variation or randomness of the trigger across the poisoned target video is first attacked by adversarial attacks, and the
samples, which could be implemented by adding randomness universal adversarial perturbation w.r.t. the target class is
into the fusion function g or the trigger transformation g . generated based on several benign videos as the trigger, then
|     |     |     | 2   |     |     |     | 1   |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Compared with the static trigger, it may be more difficult to theattackedtargetvideoandthetriggerarecombinedtoobtain
form stable mapping from the dynamic trigger to the target thepoisonedvideo.Refool[149]generatesthereflectionimage
class, but it will be more stealthy to evade the defense. For as the trigger, then combines it with one benign target image
example, the random backdoor attack [196] randomly samples to obtain one poisoned image. Due to the transparency of
the trigger pattern from a uniform distribution and the trigger the reflection image, the poisoned image has a similar visual
location from a pre-defined set for each poisoned sample. appearance to the original benign image. The hidden trigger
|           |        |       |       |         |                     |     | attack [195] | fuses | one | benign | target image | and one source |
| --------- | ------ | ----- | ----- | ------- | ------------------- | --- | ------------ | ----- | --- | ------ | ------------ | -------------- |
| The DeHiB | method | [274] | which | attacks | the semi-supervised |     |              |       |     |        |              |                |
learning models also poisons the unlabeled data by inserting image with trigger through a strategy like adversarial attack:
triggers at a random location. In Refool [149], some hyper- given a pre-trained model, enforcing the feature representation
parameters for generating the reflection trigger are randomly of the combined image (i.e., the poisoned image) to be close
sampledfromsomeuniformdistributions.Thecompositeattack to that of the source image with trigger, while encouraging
[141] defines the trigger as the composition of two existing that the combined image and the benign target image look
objects in benign images, without restrictions on the objects’ similar in the original RGB space. Consequently, the model
appearances or locations. couldlearnthemappingfromthesourceimagewiththetrigger
|           |       |            |                |     |             |          | to the target | class   | based      | on     | the generated | poisoned images,        |
| --------- | ----- | ---------- | -------------- | --- | ----------- | -------- | ------------- | ------- | ---------- | ------ | ------------- | ----------------------- |
|           |       |            |                |     |             |          | and it is     | likely  | to predict | any    | image from    | the source class        |
| E. Target | Label | Generation |                |     |             |          |               |         |            |        |               |                         |
|           |       |            |                |     |             |          | with the      | trigger | as the     | target | class. The    | invisible poison attack |
| According | to    | different  | target classes |     | of poisoned | samples, |               |         |            |        |               |                         |
[176]firstlytransformsavisibletriggerimagetoanoiseimage
i.e., g 4 (·) in Eq. (5), we can categorize existing works from withlimitedmagnitudethroughapre-trainedauto-encoder,then
| the following | perspectives. |     |     |     |     |     |     |     |     |     |     |     |
| ------------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
insertsthenoisedtriggerintoonebenigntargetimagetoobtain
1) Single-target v.s. Multi-target: one poisoned image with a similar appearance. The sleeper
a) Single-target class: Single-target class describes that agent attack [213] proposes a new setting that given a fixed
| all poisoned | training | samples | are | labeled | as one | single target |              |          |      |          |                |                 |
| ------------ | -------- | ------- | --- | ------- | ------ | ------------- | ------------ | -------- | ---- | -------- | -------------- | --------------- |
|              |          |         |     |         |        |               | trigger, the | attacker | aims | to learn | a perturbation | on the training |
classatthetrainingstage,andallpoisonedsamplesareexpected set, such that for any model trained on this perturbed training
to be predicted as that target class at the inference stage. It is set, the label-consistent backdoor from the source class to the
also called all-to-one setting. Most existing backdoor attacks target class can be activated by the trigger. It is formulated as
adopt this setting, thus we don’t expand the details. a bi-level minimization problem w.r.t. the data perturbation
| b)  | Multi-target | classes: | means | that | there | are multiple |         |            |     |               |        |     |
| --- | ------------ | -------- | ----- | ---- | ----- | ------------ | ------- | ---------- | --- | ------------- | ------ | --- |
|     |              |          |       |      |       |              | and the | parameters | of  | the surrogate | model. |     |
targetclasses.Furthermore,accordingtothenumberoftriggers,
| there are | two sub-settings. |     | One is | all-to-all | setting | [88], where |     |     |     |     |     |     |
| --------- | ----------------- | --- | ------ | ---------- | ------- | ----------- | --- | --- | --- | --- | --- | --- |
IV. ATTACKATTHETRAININGSTAGE
| with the | same trigger, | samples | from | different | source | classes |     |     |     |     |     |     |
| -------- | ------------- | ------- | ---- | --------- | ------ | ------- | --- | --- | --- | --- | --- | --- |
willbepredictedasdifferenttargetclasses.Theothersettingis The training stage involves the training loss, training algo-
multiple target classes together with multiple triggers. It could rithm and executing the training procedure. In practice, due
beachievedbysimplyextendingonesingletriggerinthesingle- to the lack of the computational resource, the user usually
targetclasssettingtomultipletriggers.Besides,theconditional outsourcesthetrainingprocesstoathird-partytrainingplatform,
backdoor generating network (c-BaN) method [196] and the or downloads a pretrained model from unverified sources,
Marksman attack [58] propose to learn a class conditional or cannot control the whole training process (e.g., federated
trigger generator, such that the attacker could generate a class learning [114]). These situations leave the attacker a chance to
conditional trigger to fool the model to any arbitrary target injectbackdooratthetrainingstage.Inadditiontomanipulating
class, rather than a pre-defined target class. the triggers or labels as did in data-poisoning based backdoor
2) Label-consistent v.s. Label-inconsistent: attacks, this threat model assumes that the attacker has the

8
total control over the whole training process and outputs a also controls the loss to achieve two goals: encouraging the
backdoored model, dubbed as training-controllable based feature representation of poisoned samples to be close to the
backdoor attack. average feature representation of the benign samples of the
target class; and encouraging the sparsity of the trigger. The
BaN attack [196] also adopts such a sequential structure, but
A. Formulation and Categorization
the trigger generator maps the random noise to the trigger. Its
Formulation. According to Eq. (2), the general formulation
extension, i.e., c-BaN, adopts a class conditional generator,
of training-controllable based backdoor attack, is as follows:
such that the trigger generator would be specific to each target
argmin
1 (cid:88)N0
λ L
(cid:0)
f
(x(i)),y(i)(cid:1) class in the setting of multi-target classes.
{x( ε i)}N i= ϵ 1 ∈Zx,wε∈Zw N 0 i=1 CA CA wε 0 0
+
1 (cid:88)Nε (cid:2)
D
(cid:0) x(i),x(i)(cid:1)
+λ L
(cid:0)
f
(x(i)),y(i)(cid:1) C. Full access v.s. Partial access of training data
N ε i=1 x 0 ε IA IA wε ε ε 1) Full access of training data: Most existing backdoor
+λ R (cid:0) f (x(i)),f (x(i)) (cid:1)(cid:3) , (6) attacks focus on centralized learning, i.e., the attacker has full
r2 2 wε 0 wε ε
access of training data, such that any training data could be
where λ CA ,λ IA ,λ r2 ≥0 are trade-off hyper-parameters. Since manipulated.
theattackerneedstopoisonthedatasetandcontrolthetraining 2) Partialaccessoftrainingdata: Incontrast,inthescenario
process, we treat both x ε and w ε as optimized variables. ofdistributedlearningorfederatedlearning(FL)[114],whichis
Categorization. We categorize existing works from the follow- designedforacceleratingthetrainingprocessorprotectingdata
ing four perspectives, including the number of attack stages, privacy,theparticipationofmultipleclientsmeansahigherrisk
whether the full training data can be accessed, whether the full of backdoor attack, though the attacker can only access partial
training process is controlled, and the controlled components training data. In the following, we mainly review the works of
of the training procedure by the attacker. backdoor attacks against federated learning. For example, [15]
and [10] propose a strategy that the malicious agents scaled up
B. One-stage v.s. Two-stage Training thelocalmodelupdateswhichcontainedbackdoorinformation,
to dominate the global model update, such that the backdoor
In this threat model, the attacker will conduct two tasks,
could be injected into the global model. The distributed
including generating poisoned samples x , and training the
ϵ backdoor attack (DBA) [260] designs a distributed backdoor
model (i.e., learning w ).
ϵ attack mechanism that multiple attackers insert backdoor into
1) Two-stage training: If this two tasks are conducted
theglobalmodelwithdifferentlocaltriggers,andshowsthatthe
sequentially (i.e., separating the problem (6) into two sub-
backdoor activated by the global trigger (i.e., the combination
problems w.r.t. x and w , respectively), then we call it
ϵ ϵ of all local triggers) has very high attack success rate in the
two-stage training backdoor attack. In this case, any off-the-
finaltrainedmodel.Wangetal.[229]proposeanewbackdoor
shelf data-poisoning based backdoor attack strategy could be
attack paradigm in the FL scenario, called edge-case backdoor
adoptedtofinishthefirsttask,whiletheattackermainlyfocuses
attack, which focuses on predicting the data points sampled
onthemanipulationofthetrainingprocess,ofwhichthedetails
from the tail of the input data distribution to a target label,
we will leave to Section IV-E.
without any modification of the input features. Chen et al.
2) One-stage training: In contrast, if these two tasks are
[29] demonstrates the effectiveness of vanilla backdoor attacks
conducted jointly (i.e., optimizing x and w jointly through
ϵ ϵ against federated meta-learning. Fung et al. [78] conducts
solving the problem (6)), then we call it one-stage training
the backdoor attack against federated learning in the sybil
backdoorattack.Comparedtothetwo-stagetrainingattack,itis
setting [63], where the adversary achieves the malicious goal
expected to couple the trigger and the model parameters more
by joining the federated learning using multiple colluding
tightly in the final backdoored model of the one-stage training
aliases. It demonstrates that the attack success rate increased
attack. The input-aware backdoor attack [173] proposes to
withthenumberofsybils(i.e.,maliciousclientswithpoisoned
jointly learn the model parameters and a generative model that
samples).TheNeurotoxinattackmethod[297]aimstoimprove
generates the trigger for each training sample. It also controls
the duration of backdoor effect during the federated learning
the training process that if adding Gaussian noise onto the
procedure, by restricting the gradients of poisoned samples to
poisoned samples, then their labels are corrected back to the
ensure that the coordinates of large gradient norms between
ground-truth labels in the loss function. LIRA [57] and WB
poisoned gradients and benign gradients (sent from the server)
[56] propose a bi-level minimization problem to jointly learn
are not overlapped, such that the backdoor effect would not
the trigger generation network and the backdoored model, with
be erased.
the only difference that LIRA adopts the ℓ norm while WB
∞
utilized the Wasserstein distance to ensure the stealthiness of
D. Full control v.s. Partial control of training process
triggers, respectively. Zhong et al. [307] designs a sequential
structurewithatriggergenerator(e.g.,aU-Netbasednetwork) 1) Full control of training process: In the conventional
and the victim model, and they are trained jointly. The trigger training paradigm, the training process is often finished at one
generator learns a multinomial distribution with three states stage by one trainer, and then the trained model is directed
{0,−1,+1}indicatingtheintensitymodificationoneachpixel, deployed. In this case, the attacker has the chance to fully
andthenatriggerissampledfromthisdistribution.Theattacker control the training process. Since most training-controllable

9
backdoorattacksbelongtothiscase,herewedon’trepeattheir 2) Control training algorithm: The bit-per-pixel attack (Bp-
details. pAttack) [239] firstly adopts image quantization and dithering
2) Partial control of training process: However, sometimes to generate stealthy triggers, then utilizes the contrastive
the training process is separated to several stages by different supervised learning to train the backdoored model, with the
trainers. Consequently, the attacker can only control a partial modificationthattheadversarialexample(usinganyadversarial
training process. One typical training paradigm that emerges in attackmethod)ofeachbenigntrainingexampleisalsoselected
recent years is firstly pre-training on a large-scale dataset, and as its negative sample. The deep feature space trojan (DFST)
then fine-tuning on a small dataset for different downstream attack [45] designs an iterative attack process between data
tasks, especially in the natural language processing field. In poisoning and a controlled detoxication step. The detoxication
this case, the attacker controls the pre-training process and step mitigates the backdoor effect of the simple features of the
aims to train a backdoored pre-trained model. However, the trigger, such that the model is enforced to learn more subtle
main challenge is how to maintain the backdoor effect after and complex features of the trigger in the next round data-
the possible fine-tuning for different downstream tasks. Along poisoningbasedtraining.InbothWaNet[174]andInput-Aware
with the popularity of the pre-training and then fine-tuning [173], a cross-trigger training mode is adopted in the training
paradigm,thebackdoorinsertedinapopularpre-trainedmodel procedure: if adding a trigger onto the training sample, then its
will cause long-term and widespread threats. There have been label is changed to the target class; if further adding a random
a few attempts. For example, Shen et al. [203] propose to noise onto the poisoned sample with the trigger, then its label
map some particular tokens (e.g., the classification token in is changed back to the correct label; the probability of adding
BERT [111]) to a target output representation in the pre- trigger and random noise is controlled by the attacker. It is
trained NLP model for the poisoned text with trigger, such claimed in [173], [174] that this training mode could enforce
that the backdoor could be activated in downstream tasks trigger nonreusablity and help to evade the defense like Neural
through the token representation. The poisoned prompt tuning Cleanse [228].
attack [65] proposes to learn a poisoned soft prompt for a 3) Control indices or order of poisoned samples: The data-
specific downstream task based on a fixed pre-trained model, efficient backdoor attack [256] controls the choice of which
and when the user uses the pre-trained model and the poisoned samplestopoisonaccordingtoafiltering-and-updatingstrategy,
prompt together, then the backdoor would be activated by the which shows improved attack performance compared to the
trigger in the corresponding downstream task. The layer-wise random selection strategy. The batch ordering backdoor (BOB)
weight poisoning (LWP) attack [123] studies the setting that attack [209] only controls the batch orders in each epoch
the backdoored pre-trained model is obtained by retraining a duringtheSGDtrainingprocesstoinjectthebackdoor,without
benignpre-trainedmodelbasedonthepoisoneddatasetandthe any manipulations on the features or labels. The key idea is
benign training dataset of the downstream task. To enhance the choosing training samples to mimic the gradients of a jointly
backdoor resistance to fine-tuning for downstream tasks, LWP trained surrogate model based on a poisoned dataset.
defines the backdoor loss of each layer, such that the backdoor
effect is injected in both lower and higher layers. Another one
V. ATTACKATTHEPOST-TRAININGSTAGE
common training paradigm is firstly training a large model,
After training the model at the training stage, a benign
then conducting model compression to obtain a lightweight
trained model will be obtained at the post-training stage. In
model via model quantization [153] or model pruning [127].
this scenario, the attacker can directly modify the parameters
The work [222] presents a new threat model in which the
of the benign model to inject trojan, dubbed weight attack
attacker controls the training of the large model, and produces
injection via parameter-modification.
a benign large model, but the model after compression became
a backdoored model that could be activated by the trigger. It is
implemented by jointly taking the uncompressed and possible
A. Formulation and Categorization
compressed models into account during the training.
Formulation. According to Eq. (3), the general formulation
of weight attack injection via parameter-modification, is
E. Controlling different components of the training procedure argmin D (w ,w )+ 1 (cid:88)N0 (cid:2) λ L (cid:0) f (x(i)),
Existing training-controllable backdoor attacks could also xε∈Zx,wε∈Zw w 0 ε N 0 i=1 CB CB w0 0
be categorized according to the controlled training component y(i)(cid:1) +λ L (cid:0) f (x(i)),y(i)(cid:1)(cid:3) + 1 (cid:88)Nε (cid:2) D (cid:0) x(i),x(i)(cid:1)
during the training procedure, such as training loss, training 0 CA CA wε 0 0 N ε i=1 x 0 ε
algorithm, order of poisoned samples. +λ L (cid:0) f (x(i)),y(i)(cid:1) +λ R (cid:0) f (x(i)),f (x(i)) (cid:1)
IA IA wε ε ε r2 2 wε 0 wε ε
1) Control training loss: For example, the work [307] adds +λ R (cid:0) f (x(i)),f (x(i)) (cid:1)(cid:3) , (7)
two terms into the training loss function to ensure stealthiness,
r3 3 wε 0 w0 0
including the number of perturbed pixels in poisoned image, where λ ,λ ,λ ,λ ,λ ≥ 0 are trade-off hyper-
CA CB IA r2 r3
and the intermediate layer’s activation difference between parameters. The second term is often specified as a hard
benign and poisoned samples, while the original trigger constraint to ensure the consistency condition AML.C ,
B
(cid:0) (cid:1)
is sampled from a multinomial distribution, of which the i.e., L f (x ),y = δ(argmaxf (x ) = y ), where
CB w0 0 0 w0 0 0
parameters are generated by a generator. δ(a) = 0 if a is true, otherwise δ(a) = ∞. It is a default

10
requirement in weight attack, thus it is omitted hereafter in B. Weight Bit-flip without Trigger
this section. Weight Bit-Flip without trigger aims to change the predic-
| Categorization. |     | According | to  | whether | the | attacker | has knowl- |     |     |     |     |     |     |     |     |
| --------------- | --- | --------- | --- | ------- | --- | -------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
tionsofoneparticularbenignsampleorasetofbenignsamples
edge of parameter values of the model, we can categorize through only manipulating the model weights from w to w ,
|          |       |                |     |     |           |        |        |           |             |     |          |        |         |        | 0 ε    |
| -------- | ----- | -------------- | --- | --- | --------- | ------ | ------ | --------- | ----------- | --- | -------- | ------ | ------- | ------ | ------ |
| existing | works | into white-box |     | and | black-box | weight | attack |           |             |     |          |        |         |        |        |
|          |       |                |     |     |           |        |        | while the | predictions |     | of other | benign | samples | should | not be |
injection.
|     |     |     |     |     |     |     |     | influenced. | The | targeted | bit-flip | attack | (T-BFA) | method | [193] |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | -------- | -------- | ------ | ------- | ------ | ----- |
aimstopredictsomeselectedsamplesasthetargetclassthrough
|              |     |        |        |           |     |     |     | flipping | a few | weight | bits, | and models |     | this task | as a binary |
| ------------ | --- | ------ | ------ | --------- | --- | --- | --- | -------- | ----- | ------ | ----- | ---------- | --- | --------- | ----------- |
| B. White-box |     | Weight | Attack | Injection |     |     |     |          |       |        |       |            |     |           |             |
optimizationproblem,whichissolvedbyasearchingalgorithm.
| White-box |        | weight           | attack | injection | means  | that   | the attacker |              |             |             |         |      |           |          |              |
| --------- | ------ | ---------------- | ------ | --------- | ------ | ------ | ------------ | ------------ | ----------- | ----------- | ------- | ---- | --------- | -------- | ------------ |
|           |        |                  |        |           |        |        |              | The targeted | attack      | with        | limited |      | bit-flips | (TA-LBF) | method       |
| has       | access | to the parameter |        | values    | of the | bengin | model. Liu   |              |             |             |         |      |           |          |              |
|           |        |                  |        |           |        |        |              | [12] uses    | the similar | formulation |         | with | T-BFA,    | but      | could attack |
etal.[151]observethattheoutputsofDNNmodelwithReLU one single selected sample, and utilized a powerful integer
functionsarelinearlyrelatedtosomeparameters.Basedonthis
|     |     |     |     |     |     |     |     | programming | method |     | called | ℓ p -Box | ADMM | [248] | to achieve |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ------ | --- | ------ | -------- | ---- | ----- | ---------- |
observation, two simple weight attack methods were proposed successful targeted attack with only a few bits flipped.
toachievetargetedpredictionsofsomeselectedbenignsamples:
thesinglebiasattack(SBA)simplyenlargesonebiasparameter
|      |            |               |     |               |     |        |               | C. Weight | Bit-flip | with | Trigger |     |     |     |     |
| ---- | ---------- | ------------- | --- | ------------- | --- | ------ | ------------- | --------- | -------- | ---- | ------- | --- | --- | --- | --- |
| that | is related | to the output |     | corresponding |     | to the | target class; |           |          |      |         |     |     |     |     |
the gradient descent attack (GDA) modifies some weights The weight attack with trigger aims to obtain an adversarial
using gradient descent algorithm. Zhao et al. [300] propose an model f through slightly perturbing the benign model
wε
ADMM based framework for solving the optimization problem weights w 0 , such that f wε (·) will be activated by any sample
of weight attack with two constraints: 1) the classification of with a particular trigger that is designed by attacker or
|     |     |     |     |     |     |     |     |     |     |     | w   |     | f (·) |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- |
the other images should be unchanged and 2) the parameter optimized together with ε , while wε performs normally
modifications should be minimized. on benign samples. This type of weight attacks seems to be
|              |     |        |        |           |     |     |     | similar to                         | backdoor | attacks, |     | but with | the                    | main difference | that   |
| ------------ | --- | ------ | ------ | --------- | --- | --- | --- | ---------------------------------- | -------- | -------- | --- | -------- | ---------------------- | --------------- | ------ |
|              |     |        |        |           |     |     |     | w ε isobtainedthroughmanipulatingw |          |          |     |          | 0 ,whilebackdoorattack |                 |        |
| C. Black-box |     | Weight | Attack | Injection |     |     |     |                                    |          |          |     |          |                        |                 |        |
|              |     |        |        |           |     |     |     | trains w                           | from     | scratch. | For | example, |                        | the Trojaning   | attack |
ε
|     |          |              |     |          |           |        |        | [148] designs | a   | sequential | weight | attack | method | with | 3 stages: |
| --- | -------- | ------------ | --- | -------- | --------- | ------ | ------ | ------------- | --- | ---------- | ------ | ------ | ------ | ---- | --------- |
| In  | contrast | to white-box |     | setting, | black-box | weight | attack |               |     |            |        |        |        |      |           |
injectionassumesthattheattackerdoesnothaveanyknowledge firstly generates the trigger through maximizing its activation
ofparametervalues.SubnetReplacementAttack(SRA)method on some selected neurons related to the target class; then
[187] generates a very narrow subnet given the architecture recoverssometrainingdatathroughreverseengineering;finally
information of the target model, where the subnet is explicitly retrains the model based on the recovered training data and
|         |     |              |            |       |     |      |              | its poisoned | version | with | the | generated | trigger | to  | achieve the |
| ------- | --- | ------------ | ---------- | ----- | --- | ---- | ------------ | ------------ | ------- | ---- | --- | --------- | ------- | --- | ----------- |
| trained | to  | be sensitive | to trigger | only, | and | then | replaces the |              |         |      |     |           |         |     |             |
corresponding parts of the target model with the generated targeted attack. It is very practical since no training data is
subnet. required. The targeted bit trojan (TBT) attack [192] relaxs the
|     |     |                            |     |     |     |     |     | above setting |      | to that | some          | training | data   | points     | are accessed, |
| --- | --- | -------------------------- | --- | --- | --- | --- | --- | ------------- | ---- | ------- | ------------- | -------- | ------ | ---------- | ------------- |
|     |     |                            |     |     |     |     |     | but restricts | the  | weight  | modifications |          | from   | continuous | to bit        |
|     | VI. | ATTACKATTHEDEPLOYMENTSTAGE |     |     |     |     |     |               |      |         |               |          |        |            |               |
|     |     |                            |     |     |     |     |     | flip. TBT     | also | adopts  | a sequential  |          | attack | procedure  | with 3        |
The deployment stage of machine learning life-cycle means steps: firstly identifying the significant neurons corresponding
that the trained model is deployed in the hardware device to the target class according to the gradient magnitude; then
| (e.g., | smartphone, | server), | where | the | model | weight | is stored | in         |          |         |     |            |     |                |        |
| ------ | ----------- | -------- | ----- | --- | ----- | ------ | --------- | ---------- | -------- | ------- | --- | ---------- | --- | -------------- | ------ |
|        |             |          |       |     |       |        |           | generating | triggers | through |     | maximizing |     | the activation | of the |
the memory with a binary form. In this scenario, the attacker identified significant neurons; finally searching and flipping
can flip the bits of the model weights in the memory space a few critical bits to inject the backdoor with the generated
via physical fault injection techniques to obtain an adversarial trigger, while keeping the accuracy on some benign samples.
model, dubbed weight attack injection via bit-flip. The ProFlip attack [30] adopts the same 3-step procedure with
|                |     |                    |     |     |     |     |     | TBT, with   | different | algorithm    |         | for   | each individual |        | stage. The |
| -------------- | --- | ------------------ | --- | --- | --- | --- | --- | ----------- | --------- | ------------ | ------- | ----- | --------------- | ------ | ---------- |
|                |     |                    |     |     |     |     |     | adversarial | weight    | perturbation |         | (AWP) | method          | [81]   | proposes   |
| A. Formulation |     | and Categorization |     |     |     |     |     |             |           |              |         |       |                 |        |            |
|                |     |                    |     |     |     |     |     | to slightly | perturb   | the          | weights | of    | a trained       | benign | model      |
Formulation. The general formulation of weight attack injec- to inject backdoor through enforcing the prediction of the
tion via bit-flip has the same form as parameter-modification, poisoned sample by the perturbed model to the target class,
i.e., Eq. (7), with one main difference that there is binary and encouraging the consistency between the prediction of the
constraint w.r.t.w . benign sample by the benign model and that by the perturbed
ε
Categorization. According to whether the benign sample is model. The anchoring attack [296] has the same goal with
modified by adding a trigger or not, existing weight attacks AWP, but with a different objective function that enforcing the
can be generally partitioned into two categories, including: 1) predictionofthepoisonedsamplebytheperturbedmodeltothe
weight bit-flip without trigger, where x = x , with the target class and that of the benign sample to the ground-truth
|     |     |     |     |     |     | ε   | 0   |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
goal that f (x )=y ̸=y ; 2) weight bit-flip with trigger, class, as well as encouraging the logit consistency between
|     |     | wε ε | ϵ 0 |     |     |     |     |     |     |     |     |     |     |     |     |
| --- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
where x ̸= x , with the goal that f (x ) = y ̸= y and the benign and perturbed models on the benign sample. The
|     | ϵ   | 0   |     |     | wε  | ε   | ϵ 0 |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
f wε (x 0 )=y 0 . handcrafted backdoor attack [94] proposes a layer-by-layer

11
weight modification procedure from the bottom to top layer, • Transfer-based adversarial examples: the generated
followingtherulesthatthereisanegligiblecleanaccuracydrop, adversarial perturbation is not designed for any specific
and the activation separation between benign and poisoned victimmodel,andthegoalistoenhancetheprobabilityof
samples is increased. The initial trigger could also be adjusted directly fooling other unknown models without repeated
to increase the activation separation during the modification queries (see Section VII-D).
procedure. Note that although it is named as backdoor, but
it actually is the weight bit flip with trigger. The subnet B. White-Box Adversarial Examples
replacementattack(SRA)[186]firstlytrainsabackdoorsubnet,
Inthissection,wecategorizewhite-boxadversarialexamples
whichshowshighactivationforpoisonedsampleswithtriggers
from two different perspectives, including perturbation types
whilelowactivationforbenignsamples,thenrandomlyreplaces
and output types. According to different perturbation types,
one subnet in the benign model by the backdoor subset and
we can categorize white-box adversarial examples into the
cut off the connection to the remaining part of the model. SRA
following five types.
only needs to know the victim model architecture, rather than
1) Optimization-based vs. Learning-based Perturbation:
model weights required in other weight attacks.
a) Optimization-based perturbation: Early works in this
field mainly focused on directly optimizing the problem (8) to
VII. ATTACKATTHEINFERENCESTAGE
generateoneadversarialexamplex oradversarialperturbation
ε
The inference stage is the last stage in the life cycle of
ε for each individual benign sample x . According to different
0
machine learning system. Normally, at this stage, test samples
specificationsofD (x,x ),existingoptimization-basedworks
w ε
are queried through the deployed model to get the predictions.
could be partitioned into two categories:
Like other stages, several adversarial phenomena can occur
at this stage to achieve malicious goals. First, to accomplish • ℓ ∞ -normandgradientsignbasedmethods.Specifically,
the attacker set D (x,x ) = ∥x−x ∥ to restrict the
the whole attack process of backdoor attacks, the attackers w ε ε ∞
upperboundoftheperturbationstoensurethestealthiness.
needtogeneratepoisonedtestsamplestoactivatethebackdoor
However, since the ℓ -norm is non-differentiable, it
injected into the backdoored model, dubbed backdoor attack ∞
is infeasible to solve the optimization problem using
activation. Similarly, weight attackers also need to activate the
the widely used gradient-based methods. To tackle this
effectiveness of weight attacks by specific samples, dubbed
difficulty,theℓ -normcouldbemovedfromtheobjective
weight attack activation. Another scenario is after obtaining a ∞
function to be a constraint, as follows:
benign model, the attacker has access to modify any benign
sample slightly to mislead the model into predicting wrong argmin L (f (x ),y ), s.t. ∥x−x ∥ ≤ϵ, (9)
IB w0 ε ε ε ∞
labels, called adversarial example. Since the core technical of xε∈Zx
backdoor attacks and weight attacks have been discussed in where ϵ > 0 is an attacker-determined upper bound of
Sections III - VI, we just detail adversarial examples in this the perturbation, which is also called perturbation budget.
section. A series of gradient sign-based methods are proposed
to solve the above problem (9). The first attempt is fast
A. Formulation and Categorization gradient sign method (FSGM) [86], where only one step
Formulation. According to Eq. (4), the general formulation is moved from x 0 following the sign of gradient with
of adversarial examples at the inference stage is as follows: the step size ϵ to obtain x ε . Consequently, the magnitude
of each entry in the adversarial perturbation is ϵ, i.e.,
argmin D (x,x )+λ L (f (x ),y )+ (8)
w ε CB CB w0 0 0 the perturbation budget is fully utilized. However, due
xε∈Zx
to the likely non-smoothness of the decision boundary
λ L (f (x ),y )+λ R (f (x ),f (x )),
IB IB w0 ε ε r1 1 w0 0 w0 ε of f (·), the one-step gradient sign direction might be
w0
where λ
CB
,λ
IB
,λ
r1
≥0 are trade-off hyper-parameters. The inaccuratetoreducethevalueofL
IB
(f
w0
(x
ε
),y
ε
).Thus,
second term L CB (f w0 (x 0 ),y 0 ) is also specified as a hard it is extended to iterative FSGM (I-FGSM) [116], where
constraint, thus it is omitted hereafter in this section. there are multiple updating steps with smaller step size,
Categorization. As shown in Figure 3, we present a hierarchi- such that the gradient sign direction of each step is more
cal taxonomy of existing inference-time adversarial examples. accurate.NotethatI-FGSMisrenamedasanotherfamous
Specifically, according to the accessed information of the name by [158]), called projected gradient descent (PGD).
attacker, there are three categories at the first level, as follows: Then, several extensions of I-FGSM are proposed to
• White-box adversarial examples: the attacker has suf- improve attack performance or adversarial transferability,
ficient information about the victim model, including such as momentum iterative FSGM (MI-FGSM) [60],
architecture and weights, such that the attacker can easily Nesterov accelerates gradient (NI-FGSM) [140], Auto-
generate adversarial perturbations to cross the decision PGD [50], etc.
boundary (see Section VII-B). • ℓ 2 -norm and gradient based methods. Another widely
• Black-box adversarial examples: the attacker can only adopted specification of D w (x,x ε ) is ℓ 2 norm, i.e.,
access the query feedback returned by the victim model, D (x,x )=∥x−x ∥2. For example, in the DeepFool
w ε ε 2
such that the attacker has to gradually adjust the perturba- method [163], it adopts the ℓ norm, but transforms the
2
tion to cross the invisible decision boundary (see Section loss function L (f (x ),y ) to a hard constraint that
IB w0 ε ε
VII-C). argmaxf (x )̸=y . It designs an iterative algorithm
w0 ε 0

12
Fig.3. Taxonomyofinference-timeadversarialexamples.
to minimize the ℓ 2 distance while moving towards the in mobile phone, or the human detection model in video
decision boundary of the benign class y , where in surveillance system). The whole attack procedure consists
0
each step the distance between the current solution and of 3 stages, including: a)generating perturbation in digital
the decision boundary has to be approximated. In the space; b) transforming the digital perturbation to a physical
C&W-ℓ method [23], L (f (x ),y ) is specified as perturbation (e.g., poster or sticker); c) digitizing the physical
|     | 2   |     | IB  | w0  | ε ε |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
a differentiable loss function (e.g., cross entropy loss perturbation back into the digital space via camera or scanner,
or hinge loss). Consequently, the problem (8) could be and then fooling the attack model. In short, there are two
directlysolvedbyanyoff-the-shelfgradient-basedmethod, transformations between the initial digital perturbation at the
together with the projection to the constraint space Z x . firststageandthefinaldigitalperturbationfedintotheattacked
b) Learning-based perturbation: model, including digital-to-physical (D2P) and physical-to-
|     |     |     |     | In  | addition | to directly |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | -------- | ----------- | --- | --- | --- | --- | --- | --- | --- |
digital
optimizing the problem (8), some methods attempt to utilize (P2D) transformations. Consequently, some distortions
the learning-based method to generate adversarial samples will be introduced on the perturbation, which may cause the
|                   |     |               |     |            |     |             | attack failure. | To  | achieve | a successful | physical | attack, | the |
| ----------------- | --- | ------------- | --- | ---------- | --- | ----------- | --------------- | --- | ------- | ------------ | -------- | ------- | --- |
| or perturbations. |     | Specifically, | it  | is assumed |     | that x ε or | ε is            |     |         |              |          |         |     |
generated by a parametric model with x as the input, i.e., attackerhastoencouragethegeneratedperturbationtoberobust
0
|     |     |     |     |     |     |     | to distortions. | Besides, | due | to these | distortions, | it is | difficult |
| --- | --- | --- | --- | --- | --- | --- | --------------- | -------- | --- | -------- | ------------ | ----- | --------- |
x =g (x ), or x =x +g (x ). (10) to require the invisibility of perturbation by restricting the ℓ -
|           | ε            | θ   | 0 ε      | 0             | θ   | 0     |                                                          |     |     |     |     |     | p   |
| --------- | ------------ | --- | -------- | ------------- | --- | ----- | -------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|           |              |     |          |               |     | θ,    | normofperturbationasdidinmostdigitalattacks.Instead,itis |     |     |     |     |     |     |
| Then, the | task becomes |     | to learn | the parameter |     | which | can                                                      |     |     |     |     |     |     |
oftenrequiredthattheadversarialexamplesshouldlooknatural
| be formulated | as  | follows: |     |     |     |     |     |     |     |     |     |     |     |
| ------------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
1 or realistic in the physical world. Since most existing works
|     |     | (cid:88)n | (cid:2) (x( i),g | (x( | i)))+ |     |     |     |     |     |     |     |     |
| --- | --- | --------- | ---------------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
arg min D (11) b e l o n g t o d ig i ta l a t t a c k , in t h e f o l lo w i n g w e o n l y in tr o d u c e
|     | n   |     | w 0 | θ   | 0   |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
θ i=1 ex i s t in g p hy s ic a l at t a c k s . A st h e s p e c ia l r e qu ire m e n t in p h y s ic a l
|     |     |     | λ L | (f (g | (x( i))),y | (i)) (cid:3) | ,   |     |     |     |     |     |     |
| --- | --- | --- | --- | ----- | ---------- | ------------ | --- | --- | --- | --- | --- | --- | --- |
IB IB w0 θ 0 ε attack is the robustness to the distortions from D2P and P2D
|         |          |     |            |             |     |             | transformations, | we  | categorize | existing | physical | attacks | into |
| ------- | -------- | --- | ---------- | ----------- | --- | ----------- | ---------------- | --- | ---------- | -------- | -------- | ------- | ---- |
| where L | is often | set | as the GAN | (generative |     | adversarial |                  |     |            |          |          |         |      |
IB
|     |     |     |     |     |     |     | two types: | According | to  | the robustness | type, | we categorize |     |
| --- | --- | --- | --- | --- | --- | --- | ---------- | --------- | --- | -------------- | ----- | ------------- | --- |
network)lossoritsvariants,suchasadvGAN[258],PhysGAN
|                 |     |        |        |        |        |     | existing physical | attacks |     | into two types: |     |     |     |
| --------------- | --- | ------ | ------ | ------ | ------ | --- | ----------------- | ------- | --- | --------------- | --- | --- | --- |
| [115], CGAN-Adv |     | [281], | AP-GAN | [298], | AC-GAN |     | [212],            |         |     |                 |     |     |     |
MAG-GAN [34], LG-GAN [308], AdvFaces [53], etc. Robustness to the D2P distortion. It is observed in
•
2) Digital v.s. Physical Perturbation: [201] that the digital-to-physical distortion is partially
a) Digital perturbation: Digital perturbation means that caused by the insufficient color space of the printer,
the whole attack procedure, including perturbation generation and RGB values out of the color space are clipped to
and attacking the victim model, is conducted in the digital bring in color distortion. To tackle it, the concept of non-
space. Since there is no perturbation distortion, the attacker printabilityscore(NPS)isproducedin[201]toencourage
can precisely manipulate the perturbation value and only needs adversarial perturbations to be in the printable color
to pay attention to generating better perturbations with higher space. Similarly, the adversarial generative nets (AGNs)
stealthiness and attack success rate. [202] trains a generative model using GANs to generate
b) Physical perturbation: Physical perturbation is firstly adversarial textures on an eyeglass frame, and restricts
studied in [116], aims to attack against the model deployed the texture within the printable color space to resist color
in the physical scenario (e.g., the face recognition model distortion.ThemethodSLAP[155]adoptstheprojectorto

13
project the digital adversarial perturbation onto real-world a randomized rendering function to model the scaling,
objects to get physical adversarial examples. It extends translating, and augmentation transformations together.
the NPS concept to the projectable colors, by considering The work [67] attacks the road sign in the physical world
the factors of projector distance, ambient light, camera by simultaneously considering rotation, scaling, and color
exposure, as well as color and material properties of the shift in the EOT loss, as well as a random background
projection surface. The work [245] adopts a conditional sampled in the physical world. The ERCG method [302]
variational autoencoder (CVAE) to learn adversarial per- designs rescaling and perspective transformations based
turbation sets based on the pair of benign and adversarial on the estimated location and perspective of the target
images. Based on the multi-illumination dataset of scenes object in the image into EOT loss. In [198], instead of
captured in the wild, the CVAE can generate adversarial printing adversarial perturbations on a sticker as did in
perturbations that are robust to different color distortions. most other physical attack works, the adversarial light
Thework[104]utilizedthegenerativeadversarialnetwork signal that illuminates the objects is generated to achieve
(GAN) [85] to simulate the color distortion between the physical attacks, implemented by LEDs. To improve
original digital image and the corresponding physical the robustness to environmental and camera imaging
image obtained through the D2P and P2D transformations conditions, a set of experimentally determined affine or
and without spatial transformations. Consequently, the polynomial transformations applied per color channel is
trained GAN model can generate images with similar adopted in the EOT loss. AdvPattern [240] transforms
color distortion in physical scenario. Then, the generated the original image by changing the position, brightness,
color-distorted image is used as the input to generate or blurring to improve the robustness to environmental
adversarialperturbations.However,duetothetimecostof distortions in person re-ID tasks. AdvHat [113] attacks
manually preparing the physical images through printing face recognition by pasting a printed adversarial sticker
andscanning,thetrainingsetofGANcannotbetoolarge, on the hat. It simulates the spatial distortion from the
likely causing the overfitting to the attacked image, the off-plane bending of the hat by a parabolic transformation
printingandscanningdevicesandtheattackedmodel.The in 3D space. The curriculum adversarial attack (CAA)
class-agnostic and model-agnostic meta learning (CMML) [305]pastesaprintedadversarialstickerontheforehead.It
method [75] adopts a GAN model to simulate the color designsastickertransformationmoduletosimulatesticker
distortion, and trains the GAN model based on limited deformation (e.g., off-plane bending and 3D rotations)
training physical images to improve the generalization andstickerpositiondisturbance,andafacetransformation
to different classes and different attacked models. The module to simulate the variations of poses, lighting
curriculumadversarialattack(CAA)method[305]designs conditions, and internal facial variations. PhysGAN [115]
a D2P module based on a multi-layer perception model, aims to attack the autonomous driving system by taking a
to simulate two types of chromatic aberration of stickers, smallsliceofvideoastheinput,ratherthan2Dindividual
including the fabrication error induced by printers and the images,suchthatthegeneratedadversarialroadsigncould
photographing error caused by cameras. continuously fool the moving vehicle. The EOT loss is
• Robustness to the physical-to-digital (P2D) distortion. extended to attack the automatic speech recognition in the
Two major sources of the P2D distortion include the rela- physical world. In [188], the expectation loss is defined
tive location variation between the physical perturbation over different kinds of reverberations that are generated
and the digitizing device (e.g., camera or phone), and by an acoustic room simulator. In [273], the expectation
the environmental variations (e.g., ambient or camera loss is defined over impulse responses recorded in diverse
light). The formal variation can be modeled as spatial environments to resist environmental reverberations, and
transformations, and the expectation over transformation the Gaussian noise is also considered to simulate the
(EOT) w.r.t. the original adversarial loss is proposed thermal noise caused in both the playback and recording
in [9] to encourage the robustness to different spatial devices.
P2D distortions. Then, EOT loss is extended in several 3) Sample-Specific vs. Sample-Agnostic Perturbation: Ac-
subsequent works with different transformations. For cording to the perspective that the adversarial perturbation
example, RP2 [70] extends EOT loss by adding physical is specific or agnostic to the benign sample, existing attack
images to the transformation sets. The RP2 is further methods could be partitioned to the following two categories:
extended in [69] from the classification to the detection
(cid:40)
task by adding more constraints on object positions. The Sample-specific perturbation: x( ε i)−x 0 (i) ̸=x( ε j)−x( 0 j);
work [265] aims to attack human detectors in the physical Sample-agnostic perturbation:x(i)−x(i) =x(j)−x(j),
ε 0 ε 0
world. It enriches EOT loss by utilizing the thin plate (12)
spline (TPS) transformation to model non-rigid object where i ̸= j are the indices of two different samples. While
deformation(e.g.,theT-shirt),aswellascolortransforma- most existing works belong to the sample-specific type, here
tion.Theuniversalphysicalcamouflageattack(UPC)[97] we mainly review the works of sample-agnostic perturbation,
modelsthenon-rigiddeformationbyaseriesofgeometric whichisalsocalleduniversaladversarialperturbation(UAP).
transformations (i.e., cropping, resizing, affine tomogra- According to whether some benign samples are utilized or not,
phy). The ShapeShifter method [37] adds the masking existing UAP methods can be categorized to the following two
operation into the EOT loss. The work [254] proposes types: data-dependent and data-free methods.

14
a) Data-dependent UAP: The existence of UAP is firstly decisionboundary(i.e.,increasingtheclassificationloss),while
discoveredby[162]intheCNN-basedimageclassificationtask, keeping the perturbed image on a transformation manifold, to
which extends the general formulation (8) from one individual ensure the inconspicuousness of the generated transformation.
benign sample to a set of benign samples, as follows: ThestAdvmethod[259]proposestominimizethelocalspatial
distortion that is defined based on the flow vector between the
n
argmin ∥ε∥2+ λ CB (cid:88) L (f (x(i)+ε),y(i)). (13) benign and adversarial images. The work [68] empirically in-
2 m CB w0 0 0
ε vestigates the vulnerability of neural network–based classifiers
i=1
to geometric transformations, using the gradient-based attack
OnevariantofUAP,calledclassdiscriminativeUAP(CD-UAP)
method or grid search to find the adversarial transformation.
[287], aims to find a common perturbation that is adversarial
Twosuggestionsforimprovingtherobustnessarealsoprovided,
for benign samples from some particular target classes, while
including inserting the adversarial transformation into the
ineffective for benign samples from other classes.
adversarial training process [158], and the majority vote with
b) Data-free UAP: To improve the generalization of
multiple random geometric transformations at inference. The
UAP to new benign samples, some works attempt to generate
generalizeduniversaladversarialperturbations(GUAP)method
UAP without utilizing any benign sample, i.e., data-free. For
[295] utilizes the learning-based attack method to generate
example, the Fast Feature Fool method [164] shows that the
the universal spatial transformation. The work [?] adopts the
perturbation with higher activation at all the convolution layers
issersteindistance[?]tomeasurethecostofmovingpixelmass
in the attacked model could fool multiple benign samples
between images, rather than the widely used ℓ distance. It
simultaneously. The generalizable data-free UAP (GD-UAP) p
can cover multiple geometric transformations, such as scaling,
[165] proposes to generate UAP by searching the perturbation
rotation, and translation.
with the maximal activation norms of all layers in the attacked
b) Style transformations: Style transformations mean to
model. The prior driven uncertainty approximation (PD-UA)
change the style or color of the global or local region of the
method [144] proposes to generate UAP by maximizing the
image. The ReColorAdv method [117] proposes to globally
model uncertainty, including the Epistemic uncertainty and the
change the color of the benign image to achieve the attack
Aleatoricuncertainty,basedontheassumptionthatlargermodel
goal, in both RGB and CIELUV color spaces. The adversarial
uncertainty corresponded to stronger attack performance. In
camouflage(AdvCam)method[67]combinesthestylelossthat
addition to the image-classification task, UAP has also studied
is firstly used on image style transfer [82] with the adversarial
in many other applications, such as image retrieval [121],
loss,togenerateadversarialimagewithnaturalstylesthatlooks
object detection [97], [118], face recognition [2], and speech
legitimate to human eyes.
recognition [170], [262], etc.
5) Densevs.SparsePerturbation: Accordingtoperturbation
4) Additivevs.Non-AdditivePerturbation: Accordingtothe
cardinality, each attack method belongs to one of the following
relationship between x and x , existing attack methods could
ε 0 types:
be partitioned into the following two categories:
(cid:40)
(cid:40) Dense perturbation: ∥x ε −x 0 ∥ 0 =|x 0 |;
Additive perturbation: x =x +ε; (15)
ε 0 (14) Sparse perturbation: ∥x −x ∥ <|x |.
ε 0 0 0
Non-additive perturbation: x =h(x ),
ε 0
For example, if all pixels in one image are perturbed, then
where h(·) denotes a non-additive transformation function. the perturbation is dense. While most existing attack methods
While most existing works adopted additive perturbation, here adoptthesettingofdenseperturbation(orcalleddenseattack),
we mainly review the works of non-additive perturbation. some works find that perturbing partial entries of one benign
Note that since the non-additive transformation causes the sample (e.g., partial pixels in one image) could also achieve
global distortion compared to the benign sample x 0 , the the attack goal, which is called sparse perturbation or sparse
ℓ p -norm is often no longer adopted to specify the distance attack. Compared with dense perturbation where the attacker
metric D 1 (x 0 ,x ε ), instead by other forms, such as the style only needs to determine the perturbation magnitude of each
loss in AdvCam [67], or the geodesic distance in Manifool entry,thesparseattackershouldalsodeterminetheperturbation
[109], etc. Existing non-additive methods mainly adopt two positions.Accordingtothestrategyofdeterminingperturbation
types of transformations, i.e., geometric (or spatial) and style positions, existing sparse attack methods are partitioned into
transformations. three categories, including: manual, heuristic search and
a) Geometrictransformations: Geometrictransformations optimization-based strategies.
mainly include rotation, translation, and affine transformations. a) Manual strategy: Manual strategy means that the at-
The fact that small rotations and translations on images can tacker manually specifies the perturbed positions. For example,
changethepredictionsofconvolutionalneuralnetworks(CNNs) LaVAN [110] experimentally demonstrates that an adversarial
is firstly observed in [72]. However, it focuses on measuring and visible local patch located in the background in one image
theinvarianceofCNNstoanygeometrictransformation,rather could also fool the model. This manual strategy qualitatively
thandesigninginconspicuousadversarialtransformations.Some demonstrates that the background pixels are also important for
laterworkspaymoreattentiontoensuretheinconspicuousness the prediction, but cannot provide more exact and quantitative
of the generated geometric transformations. For example, analysis of the sensitivity of each pixel.
the Manifool method [109] generates adversarial geometric b) Heuristic search strategy: Heuristic search strategy
transformations by perturbing the original image towards the means that the perturbed entries are gradually determined

15
accordingtosomeheuristiccriterion.Forexample,theJacobian- 6) Untargeted vs. Targeted Attack: Untargeted attack aims
basedsaliencymapattack(JSMA)[180]anditsextensions[23] to fool the model to give an incorrect prediction (i.e., different
perturb the pixels corresponding to large values in the saliency with y ) on x , while targeted attack aims to fool the model
0 ε
map. The CornerSearch method [49] firstly sorts all candidate to predict x as a target class y . Their difference could
ε ε
pixels according to their changes to the model output, then be reflected by the specification of L (f (x ),y ) in the
IB w0 ε ε
iteratively samples perturbed pixels following a probability general formulation (8), as follows:
distribution related to the sorted index. The LocSearchAdv (cid:40)
Untargeted attack:L (f (x ),y )=−L(f (x ),y );
algorithm [167] conducts sparse attack in a black-box setting. IB w0 ε ε w0 ε 0
It designs a greedy local search strategy that given the current Targeted attack: L (f (x ),y )=L(f (x ),y ),
IB w0 ε ε w0 ε ε
perturbed pixels, then new candidate pixels are searched from (16)
a small square centered at each perturbed pixel, according to where L(·,·) could be any widely used loss function, such as
black-box attack performance. cross-entropy loss or hinge loss. The difference between the
above two loss specifications doesn’t influence the choice of
c) Optimization-based strategy: Optimization-based strat-
the adopted optimization algorithm. Thus, in most adversarial
egy aims at optimizing the magnitudes and positions of
example works, experiments of both untargeted and targeted
perturbationssimultaneously.Forexample,theOne-Pixelattack
attacksareconductedsimultaneouslytoverifytheeffectiveness
[216] tries to perturb only one pixel to achieve the attack goal.
of the proposed objective function or the proposed algorithm.
The pixel coordinates and the RGB values are concatenated to
However, according to the reported results in existing works,
form a vector that needs to be optimized. Then, the differential
there is a remarkable gap in the attack performance between
evolution (DE) algorithm [25] is adopted to search for a good
these two attacks. Especially in black-box and transfer-based
concatenate vector that achieves the attack goal. In [299], the
attacks, the targeted attack is often much more challenging
sparsity of perturbations is encouraged by the ℓ norm, along
0 than the untargeted attack, as the adversarial region of a
with the adversarial loss. The alternating direction method
particulartargetclassismuchnarrowerthanthatofallincorrect
of multipliers (ADMM) algorithm [18] is then adopted to
classes. Thus, a few recent attempts focus on improving
optimize this problem. However, there is no constraint on
the targeted attack performance in black-box and transfer-
perturbationmagnitudes,causingthelearnedperturbationmight
based attack scenarios. For example, the work [124] replaces
be visible. The Pointwise attack method [199] extends the
the cross entropy loss by the Poincare distance metric to
Boundary attack method [19] (a dense black-box attack) from
obtain the self-adaptive gradient magnitude during iterative
ℓ norm to ℓ norm to enforce sparsity. It firstly adds a salt-
2 0 attack to alleviate noise curing, and adds a triplet loss to
and-pepper noise that could fool the model, then repeatedly
enforce adversarial example away from the original class,
removes the noise of one pixel if the model is still fooled. The
leading to more transferable targeted adversarial examples.
GreedyFool method [145] develops a two-stage algorithm to
The transferable targeted perturbation (TTP) [168] designs a
minimize the ℓ norm. The first stage increases the perturbed
0 generative adversarial network w.r.t. the target class, which
pixels according to the distortion map, and the second stage
can produce highly transferable targeted perturbation. The
gradually reduces the perturbed pixels according to the attack
work[304]findthatexistingiterativetransferattacks(e.g.,MI-
performance with different perturbation magnitudes on these
FSGM [60], DI-FSGM [261]) could give much better targeted
pixels. The SparseFool method [159] adopts the ℓ norm to
1 transferattackresultswithsufficientiterations,andproposedto
encourage sparsity, and develops an iterative algorithm that
adopt the logit loss to further improve the attack performance.
picks one coordinate (i.e., one pixel) to perturb based on
7) Factorized vs. Structured Output: Most existing attacks
the linear approximation of the decision boundary. Moreover,
areevaluatedontaskswithfactorizedoutputs(e.g.,thediscrete
[264] explores the group-wise sparsity in adversarial examples,
label in DNN-based image classification task), such that it is
encouraged by the ℓ norm [284]. The learned perturbations
2,1 easy to compute (for white-box attacks) or estimate (for black-
gathered together to the local regions that are highly related to
box attacks) the gradients w.r.t. the input. However, there are
discriminative regions. The ℓ norm is also used in [241] to
2,1 alsosomeDNN-basedtaskswithstructuredoutputs(e.g.,image
enforce the temporal sparsity for attacking the model for the
captioning, scene text recognition), which predict a sequential
video-based task, i.e., only partial frames are perturbed. The
label for each input (image or audio). The dependency among
sparse adversarial attack via perturbation factorization (SAPF)
outputs may bring in additional challenge for adversarial
method [71] provides a new perspective that the perturbation
examples. According to the type of target outputs, existing
on each pixel is factorized to the product of the perturbation
attacks against tasks with structured outputs are partitioned
magnitudeandabinaryselectionfactor.Ifthebinaryfactoris1,
into the following two categories:
then the corresponding pixel is perturbed, otherwise not. Then,
a) Attackwithcompletetargetoutputs: whereacomplete
thesparseattackisformulatedasamixedintegerprogramming
sentence is set as the target output. In this case, the gradient
(MIP), and the sparsity degree is exactly controlled via the
w.r.t. the input can be easily computed as did in the regular
cardinality constraint on selection factors. This MIP problem
learningoftheattackedmodel,andtheattackmethodsdesigned
is efficiently solved by the ℓ -Box ADMM algorithm [248].
p for models with factorized outputs can be naturally applied.
According to different output types, we can further For example, [269] sets a complete and irrelevant caption
categorize white-box adversarial examples into the following as the target caption to attack the dense captioning model,
two types. and directly uses the Adam-based attack method proposed

16
in [23]. [204] proposes to construct the complete target a successful adversarial perturbation. Thus, once a successful
caption by replacing noun, numeral, or relation words in the adversarialperturbationisobtained,theattackcouldstop.Inthe
original caption with other words of the same type. Then, following, according to the information utilized by the attacker,
the visual-semantic embedding based image captioning model we summarize existing score-based adversarial examples from
is attacked by minimizing the hinge loss defined based on two categories, including query-based and combination-based
the constructed target caption. [24] attacked the audio-to-text adversarial examples.
a) Query-basedadversarialexamples:
model by setting complete target sentences, and generated Query-basedmeth-
the adversarial audio input by the fast gradient sign attack ods treat the attack task as a black-box optimization problem,
method [86] and its iterative variant [116]. The works [267] such that many black-box optimization approaches can be
and [268] also change some words in the output sequence applied. Accordingly, existing query-based attack methods can
to set a complete target output, and utilize the sequential be further partitioned into two categories, including random
|               |     |     |               |     |             |        |            | search and gradient-estimation-based |     |     |     | methods. |     |     |
| ------------- | --- | --- | ------------- | --- | ----------- | ------ | ---------- | ------------------------------------ | --- | --- | --- | -------- | --- | --- |
| factorization |     | of  | the posterior |     | probability | w.r.t. | the target |                                      |     |     |     |          |     |     |
sequence to simplify the optimization of adversarial loss. • Random search methods update the adversarial pertur-
b) Attack with partial target outputs: where only partial bation based on some random search strategies. SimBA
outputs in the target output sequence is set as the target. [89] randomly picks one direction to add or subtract
The work [31] firstly proposed the target keyword attack by the perturbation at each step, from a set of orthonormal
requiring some keywords to appear in the output sequence, basis vectors, which is specified as the Cartesian basis or
while not restricting their specific locations. It is implemented discrete cosine basis. The ECO attack [161] proposes to
by the hinge loss to maximize the probability of the target determine the perturbation among the vertices of the ℓ
∞
versionoftheneighborhoodsetB
keywords. The work [270] proposed exact structured attack x,ϵ ,leadingtoadiscrete
by requiring some specific keywords to appear at specific set maximization problem, which is then approximately
locations in the output sequence. It is more strict than the solved by the local search algorithm [73]. The Square
target keyword attack, and the complete target output could be attack [7] also searches the perturbation among vertices
seen as a special case of it. It is implemented by treating the of the ℓ version of B , but within a randomly sampled
|     |     |     |     |     |     |     |     | ∞   |     |     | x,ϵ |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
attack as a structured output prediction problem with hidden local patch at each step. The PRFA attack [139] extends
variables, where the targeted words are treated as observed theSquareattacktoattackagainstobjectdetectionmodels
random variables, while the output words of other unrestricted in black-box manner, by perturbing multiple local patches
locations are treated as hidden variables. Then, two structured in parallel for better efficiency. The PPBA attack [122]
output learning methods, including generalized expectation searches the perturbation in a low-dimensional and low-
maximization [16] and structured SVM with latent variables frequency subspace constructed by the discrete cosine
[280], are adopted to generate the adversarial examples. transform (DCT) [4] and its inverse transform, through
|     |     |     |     |     |     |     |     | an accelerated | random |     | walk | optimization | algorithm | with |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | ------ | --- | ---- | ------------ | --------- | ---- |
C. Black-Box Adversarial Examples the effective probability of historical searches.
|             |           |             |             |           |                |             |             | Gradient-estimation-based |       |           |              | methods      | update | the adver-    |
| ----------- | --------- | ----------- | ----------- | --------- | -------------- | ----------- | ----------- | ------------------------- | ----- | --------- | ------------ | ------------ | ------ | ------------- |
|             | According | to          | the type    | of        | query feedback |             | returned by | •                         |       |           |              |              |        |               |
|             |           |             |             |           |                |             |             | sarial perturbation       |       | based     | on gradient, |              | which  | is estimated  |
| the         | attacked  | model,      | existing    | black-box |                | adversarial | examples    |                           |       |           |              |              |        |               |
|             |           |             |             |           |                |             |             | based on                  | query | feedback  | or           | some         | kinds  | of prior. The |
| could       | be        | further     | partitioned | into      | two            | categories, | including   |                           |       |           |              |              |        |               |
|             |           |             |             |           |                |             |             | NES attack                | [100] | estimates |              | the gradient |        | based on the  |
| score-based |           | adversarial |             | examples  | with           | continuous  | feedback    |                           |       |           |              |              |        |               |
naturalevolutionstrategy[244],whereseveralperturbation
| (e.g., | the | posterior | probability |     | in [0,1]), | and decision-based |     |             |         |      |            |     |               |        |
| ------ | --- | --------- | ----------- | --- | ---------- | ------------------ | --- | ----------- | ------- | ---- | ---------- | --- | ------------- | ------ |
|        |     |           |             |     |            |                    |     | vectors are | sampled | from | a Gaussian |     | distribution. | Bandit |
adversarialexampleswithdiscretefeedback(e.g.,thediscrete
|     |     |     |     |     |     |     |     | [101] extends | this | gradient | estimation |     | by embedding | both |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | ---- | -------- | ---------- | --- | ------------ | ---- |
label).
thespatialprior(neighboringpixelshavesimilargradients)
|         | 1) Score-basedAdversarialExamples: |     |         |           |            | Forthiscategory,the |         |                  |              |       |                |      |            |             |
| ------- | ---------------------------------- | --- | ------- | --------- | ---------- | ------------------- | ------- | ---------------- | ------------ | ----- | -------------- | ---- | ---------- | ----------- |
|         |                                    |     |         |           |            |                     |         | and the temporal |              | prior | (the gradients |      | between    | consecutive |
| general | formulation                        |     | (8) is  | specified | as follows |                     |         |                  |              |       |                |      |            |             |
|         |                                    |     |         |           |            |                     |         | iterations       | are similar) | to    | obtain         | more | consistent | gradients.  |
|         |                                    |     | (cid:0) |           | (cid:1)    | (cid:0)             | (cid:1) |                  |              |       |                |      |            |             |
min δ x ∈B +max 0,△ , (17) NAttack [133] extends the NES attack by restricting the
|     |     |     | ε   | x0,ϵ |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
xε∈Zx
|     |     |     |           |     |     |     |     | vectors sampled |     | from | the Gaussian |     | distribution | into the |
| --- | --- | --- | --------- | --- | --- | --- | --- | --------------- | --- | ---- | ------------ | --- | ------------ | -------- |
|     | B   |     | {x′|∥x′−x |     |     |     |     |                 |     |      |              |     |              |          |
where x0,ϵ = 0 ∥ p ≤ ϵ} defines a neighborhood feasiblespace(i.e.,theallowedsearchspaceofadversarial
set around x , with ϵ > 0 and the norm p being attacker perturbation). The AdvFlow method [160] further extends
0
| determined |     | scalars. | The | distance | function | D   | is specified as |         |              |     |              |     |              |        |
| ---------- | --- | -------- | --- | -------- | -------- | --- | --------------- | ------- | ------------ | --- | ------------ | --- | ------------ | ------ |
|            |     |          |     |          |          | x   |                 | NAttack | by replacing |     | the Gaussian |     | distribution | with a |
| (cid:0)    | ∈B  | (cid:1)  |     |          |          |     |                 |         |              |     |              |     |              |        |
δ x ,andδ(a)=0ifaistrue,otherwiseδ(a)=∞.It complexdistributionwhichiscapturedbythenormalizing
ε x,ϵ
servesasahardconstrainttolimitadversarialperturbation,and flow model [219] pre-trained on benign data, such that
the attacker has to minimize the hinge loss through searching the generated adversarial sample is more close to the
(cid:0) (cid:1)
within B . The hingeloss max 0,△ is specifiedas follows: benign sample. ZO-signSGD [146] proposes to update
x,ϵ
|     |            |     |       |     |             |     |      | the perturbations |     | along | with | the gradient | sign | direction |
| --- | ----------- | --- | ----- | --- | ----------- | --- | ---- | ----------------- | --- | ----- | ---- | ------------ | ---- | --------- |
|     | Untargeted: |     | △=f(x |     | ,y )−maxf(x |     | ,j); |                   |     |       |      |              |      |           |
 ε 0 ε rather than the estimated gradient direction, and provides
|     |            |     |          |       | j̸=y0   |     | (18)  |                                                   |          |     |        |             |     |                 |
| --- | ---------- | --- | -------- | ----- | ------- | --- | ----- | ------------------------------------------------- | -------- | --- | ------ | ----------- | --- | --------------- |
|     |            |     |          |       |         |     |       | a theoretical                                     | analysis |     | of the | convergence |     | rate. It is not |
|     | Targeted: |     | △=maxf(x |       | ,j)−f(x |     | ,y ), |                                                   |          |     |        |             |     |                 |
|     |            |     |          |       | ε       | ε   | ε     |                                                   |          |     |        |             |     |                 |
|     |            |     |          | j̸=yε |         |     |       | onlymemory-efficient,butalsoshowscomparableoreven |          |     |        |             |     |                 |
with y ∈ Y being the target label. Note that the hinge loss betterattackefficiencyinpractice.SignHunter[6]designs
ε
is non-negative, and 0 is the minimal value, corresponding to a divide-and-conquer strategy to accelerate the estimation

17
of gradient signs, by flipping the gradient signs of all target model, then adopts the queried inputs and the
pixels within a range together, and then repeating such labels returned by the target model to tune the surrogate
a group flipping operation on the whole image, the first models. The learnable black-box attack (LeBA) [275]
half, the second half, the first quadrant, and so on. combines the query-based method SimBA [89] and the
|                      |                |           |                   |              |                   |           | transfer-based    |          | method        | TIMI [61]   | conducts      | on             | surrogate   |
| -------------------- | -------------- | --------- | ----------------- | ------------ | ----------------- | --------- | ----------------- | -------- | ------------- | ----------- | ------------- | -------------- | ----------- |
| b) Combination-based |                |           | methods:          |              | Combination-based |           |                   |          |               |             |               |                |             |
|                      |                |           |                   |              |                   |           | models,           | proposes | to            | update      | surrogate     | models         | using a     |
| methods incorporate  |                | some      | kinds of          | priors       | learned           | from sur- |                   |          |               |             |               |                |             |
|                      |                |           |                   |              |                   |           | high-Order        | gradient | approximation |             | algorithm.    |                | The consis- |
| rogate models        | into           | the query | procedure         | for          | the target        | model,    |                   |          |               |             |               |                |             |
|                      |                |           |                   |              |                   |           | tency sensitivity |          | guided        | ensemble    | attack        | (CSEA)         | [283]       |
| to enhance           | the efficiency |           | of finding        | a successful | adversarial       |           |                   |          |               |             |               |                |             |
|                      |                |           |                   |              |                   |           | proposes          | to learn | a linear      | combination |               | of an ensemble | of          |
| solution. According  |                | to the    | type of surrogate |              | models, the       | priors    |                   |          |               |             |               |                |             |
|                      |                |           |                   |              |                   |           | surrogate         | models   | with          | diversified | model         | architectures  | to          |
| can be further       | partitioned    |           | into two          | categories.  |                   |           |                   |          |               |             |               |                |             |
|                      |                |           |                   |              |                   |           | approximate       | the      | target        | model,      | and meanwhile |                | update the  |
• Priors from static surrogate models, which means that surrogate models by encouraging the same response to
thesurrogatemodelsarefixedduringtheattackprocedure. different adversarial samples. The black-box attack via
For example, the Subspace attack method [91] adopts surrogate ensemble search (BASES) [22] designs a bi-
the gradients of several surrogate models as the basis level optimization by alternatively updating adversarial
vectorstoestimatethegradientforupdatingtheadversarial perturbation based on the linear combination of several
perturbation against the target model in each step of surrogate models and the linear combination weight of
the iterative attack procedure. The prior-guided random each surrogate model according to query feedback. The
gradient-free (P-RGF) method [59] improves the random Simulator attack [157] utilizes meta learning to learn a
gradient-free method [171] by combining the surrogate generalized simulator (i.e., surrogate model), which can
gradient with randomly sampled unit vectors to obtain be fine-tuned by limited query feedback. Similar to CG-
more accurate gradient estimation. TREMBA [99] trains Attack,themetaconditionalgenerator(MCG)attack[279]
anauto-encodertogenerateadversarialperturbationbased alsolearnsCAD,withthedifferencethatMCGproposeda
on surrogate models, and adopts the decoder (i.e., the metalearningframeworktocaptureboththeexample-level
projection from a low-dimensional latent space to the and model-level adversarial transferability (introduced
original input space) as a prior, such that the perturbation later in Section VII-D), such that the CAD could be
for the target model could be efficiently searched in the adjusted for each benign sample, and the surrogate model
low-dimensionallatentspace.TheCG-Attack[77]captures could be updated based on query feedback.
| the conditional |     | adversarial | distribution |     | (CAD) | by the | c-                |     |         |     |                |     |         |
| --------------- | --- | ----------- | ------------ | --- | ----- | ------ | ----------------- | --- | ------- | --- | -------------- | --- | ------- |
|                 |     |             |              |     |       |        | 2) Decision-based |     | Attack: | For | this category, | the | general |
Glow model, which could map a Gaussian distribution formulation (8) is specified as follows
| to a complex |     | distribution. | The | c-Glow | model | is firstly |     |     |     |         |     |     |         |
| ------------ | --- | ------------- | --- | ------ | ----- | ---------- | --- | --- | --- | ------- | --- | --- | ------- |
|              |     |               |     |        |       |            |     |     |     | (cid:0) |     |     | (cid:1) |
trainedbasedonsurrogatemodels.Then,themappingpart min D (x ,x )+δ C(f (x ),y)=1 , (19)
|     |     |     |     |     |     |     |     | x   | 0 ε |     | w0  | ε   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
xε∈Zx
| of this | c-Glow | model, | i.e., the | mapping | from | Gaussian |     |     |     |     |     |     |     |
| ------- | ------ | ------ | --------- | ------- | ---- | -------- | --- | --- | --- | --- | --- | --- | --- |
distribution to perturbation distribution, is fixed, while the where C(f (x ),y) indicates the adversarial criterion, which
w0 ε
Gaussian distribution is refitted based on query feedback is true if the attack goal is achieved, otherwise false. Specifi-
| of the           | target | model,        | to approximate | the | target CAD,  | such  | cally, |     |     |     |     |     |     |
| ---------------- | ------ | ------------- | -------------- | --- | ------------ | ----- | ------ | --- | --- | --- | --- | --- | --- |
| that adversarial |        | perturbations | for            | the | target model | could |        |     |     |     |     |     |     |
(cid:40)
|     |     |     |     |     |     |     | Untargeted | attack: | C(f | (x ),y)=I(f(x |     | ;w)̸=y | );  |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ------- | --- | ------------- | --- | ------ | --- |
be efficiently sampled. The meta square attack (MSA) w0 ε ε 0
attack [278] utilizes meta learning to learn a sampling Targeted attack: C(f (x ),y)=I(f(x ;w)=y ),
|     |     |     |     |     |     |     |     |     |     | w0 ε |     | ε   | ε   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- |
(20)
| distribution |     | of the hyper-parameters |     | in  | Square attack | [7] |        |     |     |        |     |     |     |
| ------------ | --- | ----------------------- | --- | --- | ------------- | --- | ------ | --- | --- | ------ | --- | --- | --- |
|              |     |                         |     |     |               |     | I(a)=1 |     |     | I(a)=0 |     |     |     |
(e.g.,thesquarepatch’ssize,locationandcolor)basedon with if a is true and otherwise, and y ∈Y
ε
surrogate models, and the meta distribution is fine-tuned denotes the target label. Besides, as defined above, δ(a)=0
based on query feedback to provide more suitable hyper- if a is true, otherwise δ(a) = ∞, which serves as a hard
parametersforattackingthetargetmodel.Theeigenblack- constraint to ensure that each immediate solution x should be
ε
|                     |     |     |              |     |                  |      | a feasible solution | that | satisfies | the | attack | goal. With | this hard |
| ------------------- | --- | --- | ------------ | --- | ---------------- | ---- | ------------------- | ---- | --------- | --- | ------ | ---------- | --------- |
| box attack(EigenBA) |     |     | [309]studies | a   | differentsetting | that |                     |      |           |     |        |            |           |
the surrogate and target models share one backbone that constraint, the attacker has to search the better solution (i.e.,
is accessible to the attacker, while the classifier layers are correspondingtosmallerD (x ,x ))withinthefeasiblespace
|     |     |     |     |     |     |     |     |     | x   | 0 ε |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
differentandthetargetmodel’sclassifierlayerisunknown. defined by δ(·), also dubbed adversarial space. However, the
EigenBA proposes to calculate the updating direction of main challenge is that such an adversarial space is invisible
each step according to the right singular vectors of the to the attacker. Existing works mainly focused on designing
Jacobian matrix of the shared and white-box backbone. efficient search strategies, subject to the invisible adversarial
Priors from adaptive surrogate models, which means space. The search strategies in existing decision-based attack
•
random
that the surrogate models are updated based on the methods are summarized as two categories, including
query feedback during the attack procedure, such that search and gradient-estimation-based methods.
the gap between surrogate and target models could be a) Random search methods: Random search methods
alleviated. For example, the hybrid batch attack method determinethesearchdirectionandstepsizearoundtheinvisible
[218] generates candidate adversarial examples based decision boundary using some heuristic strategies, based on a
on surrogate models as the initial point to query the randomsampler.ThefirstattemptcalledBoundarymethod[19]

18
samples the search direction based on the normal distribution (MCG) [279], as follows: “adversarial perturbations around
and dynamically adjusts the step size according to the ratio different benign examples may have some similar properties".
of adversarial solutions among all sampled solutions. The Meanwhile, some other works also implied or utilized similar
Evolutionary method [62] extends the Boundary method by ideas. According to the assumption of “similar properties",
replacing the normal distribution with a Gaussian distribution, existing works are partitioned into the following categories.
of which the parameters and the step size are automatically a) Different benign examples have a common adversarial
|          |       |                  |     |           |     |         |           | perturbation: |     | i.e., ∃i̸=j, |     |     |     |     |     |     |
| -------- | ----- | ---------------- | --- | --------- | --- | ------- | --------- | ------------- | --- | ------------ | --- | --- | --- | --- | --- | --- |
| adjusted | using | the evolutionary |     | strategy. |     | Another | extension |               |     | ,            |     |     |     |     |     |     |
of Boundary called customized iteration and sampling attack f(x(i)+ε)=y(i), f(x(j)+ε)=y(j).
(21)
(CISA) [206] replaces the initial adversarial perturbation by a 0 ε 0 ε
transferable perturbation generated based on surrogate models, This assumption is actually the basis of UAP (universal
and adjusts the sampling distribution and step size based adversarial perturbations) [162] and its variants.
|               |     |          |               |     |                |     |        | b)  | Adversarial | perturbations |     | of  | different | benign | examples |     |
| ------------- | --- | -------- | ------------- | --- | -------------- | --- | ------ | --- | ----------- | ------------- | --- | --- | --------- | ------ | -------- | --- |
| on historical |     | queries. | The geometric |     | decision-based |     | attack |     |             |               |     |     |           |        |          |     |
(GeoDA) [191] constructs the search direction by estimating could be generated by a common parametric model: , i.e.,
| the normal | direction |     | of the | decision | boundary, | utilizing | the | ∃i̸=j, |     |     |     |     |     |     |     |     |
| ---------- | --------- | --- | ------ | -------- | --------- | --------- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
assumptionoflowcurvatureofthedecisionboundary.Thesign f(x(i)+g (x(i)))=y(i), f(x(j)+g (x(j)))=y(j). (22)
|     |     |     |     |     |     |     |     | 0   | θ   | 0   | ε   | 0   | θ 0 |     | ε   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
flipattack(SFA)[40]randomlysearchesthenewsolutionatthe
|         |        |          |        |            |          |          |     | The generative |       | adversarial | perturbations |          | (GAP)       | attack |            | [184] |
| ------- | ------ | -------- | ------ | ---------- | -------- | -------- | --- | -------------- | ----- | ----------- | ------------- | -------- | ----------- | ------ | ---------- | ----- |
| surface | of the | ℓ ∞ ball | around | the benign | example, | followed | by  |                |       |             |               |          |             |        |            |       |
|         |        |          |        |            |          |          |     | adopts the     | above | assumption  |               | to train | a generator |        | to produce |       |
randomsignflipsofsomedimensionsofthenewsolution.The
ℓ ball’s radius is gradually decreased along search iterations universal or sample-specific adversarial perturbations. It is
∞
|               |     |          |                 |           |            |            |            | further         | extended | in [169]         | and | [168]         | to boost | the          | adversarial |     |
| ------------- | --- | -------- | --------------- | --------- | ---------- | ---------- | ---------- | --------------- | -------- | ---------------- | --- | ------------- | -------- | ------------ | ----------- | --- |
| to ensure     | the | decrease | of perturbation |           | norms.     | Similarly, | the        |                 |          |                  |     |               |          |              |             |     |
|               |     |          |                 |           |            |            |            | transferability |          | across different |     | data domains, |          | by utilizing |             | the |
| Ray searching |     | attack   | (RayS)          | [32] also | determines |            | the search |                 |          |                  |     |               |          |              |             |     |
ℓ training mechanism of generative adversarial networks [85].
| direction  | at the                    | surface | of the  | ∞ ball, | while    | the | step size | is       |                           |          |               |     |                         |        |     |     |
| ---------- | ------------------------- | ------- | ------- | ------- | -------- | --- | --------- | -------- | ------------------------- | -------- | ------------- | --- | ----------------------- | ------ | --- | --- |
|            |                           |         |         |         |          |     |           | c)       | Adversarialexamplesw.r.t. |          |               |     | differentbenignexamples |        |     |     |
| determined | through                   | binary  | search. |         |          |     |           |          |                           |          |               |     |                         |        |     |     |
|            |                           |         |         |         |          |     |           | follow a | common                    | marginal | distribution: |     | i.e.,                   | ∃i̸=j, |     |     |
| b)         | Gradient-estimation-based |         |         |         | methods: |     | Gradient- |          |                           |          |               |     | ,                       |        |     |     |
estimation-based methods determine the search direction ∃i̸=j,x(i),x(j) ∼P(x ). (23)
|               |            |              |       |        |             |         |            |                 |          |              | ε   | ε              | ε   |         |       |     |
| ------------- | ---------- | ------------ | ----- | ------ | ----------- | ------- | ---------- | --------------- | -------- | ------------ | --- | -------------- | --- | ------- | ----- | --- |
| by estimating |            | the gradient |       | w.r.t. | the current |         | solution   | in              |          |              |     |                |     |         |       |     |
|               |            |              |       |        |             |         |            | This assumption |          | is adopted   |     | by AdvFlow     |     | [160],  | where | the |
| the update    | procedure. |              | Ilyas | et al. | [100]       | propose | to firstly |                 |          |              |     |                |     |         |       |     |
|               |            |              |       |        |             |         |            | common          | marginal | distribution |     | of adversarial |     | samples | P(x   | )   |
estimate the continuous score of the current solution based ε
|              |              |          |                |             |            |       |             | is modeled | by          | the normalizing |      | flow           | model         | [219],    | which  | is    |
| ------------ | ------------ | -------- | -------------- | ----------- | ---------- | ----- | ----------- | ---------- | ----------- | --------------- | ---- | -------------- | ------------- | --------- | ------ | ----- |
| on the       | returned     | hard     | labels,        | by querying |            | a few | randomly    |            |             |                 |      |                |               |           |        |       |
|              |              |          |                |             |            |       |             | capable    | to capture  | complex         | data | distributions. |               |           |        |       |
| perturbed    | points       | around   | the            | current     | solution.  |       | Then, the   |            |             |                 |      |                |               |           |        |       |
|              |              |          |                |             |            |       |             | d)         | Adversarial | perturbations   |      |                | around        | different | benign |       |
| natural      | evolutionary | strategy |                | approach    | is adopted |       | to estimate |            |             |                 |      |                |               |           |        |       |
|              |              |          |                |             |            |       |             | samples    | follow      | a common        |      | conditional    | distribution: |           | ,      | i.e., |
| the gradient |              | using    | the continuous |             | score.     | The   | opt-based   |            |             |                 |      |                |               |           |        |       |
∃i̸=j,
| black-box | attack | (Opt-attack) |     | [43] | proposes | a   | continuous |     |     |     |     |     |     |     |     |     |
| --------- | ------ | ------------ | --- | ---- | -------- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
objective function to find the search direction leading to x( i),x( j) ∼P(x |x ). (24)
|     |     |     |     |     |     |     |     |     |     | ε   | ε   | ε   | 0   |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
the minimal ℓ 2 norm of perturbations, which can be solved For example, the CG-Attack method [77]) proposes to learn a
| by the | zero-order | optimization |     | method |     | with the | gradient |        |             |              |     |     |      |         |             |     |
| ------ | ---------- | ------------ | --- | ------ | --- | -------- | -------- | ------ | ----------- | ------------ | --- | --- | ---- | ------- | ----------- | --- |
|        |            |              |     |        |     |          |          | common | conditional | distribution |     | P(x | |x ) | via the | conditional |     |
|        |            |              |     |        |     |          |          |        |             |              |     |     | ε 0  |         |             |     |
estimation-based on the randomized gradient-free method. Glow model [156] (a variant of the flow-based model). This
| The Sign-OPT |     | method     | [44] | improves | the         | performance | of     |            |             |             |         |               |     |        |           |       |
| ------------ | --- | ---------- | ---- | -------- | ----------- | ----------- | ------ | ---------- | ----------- | ----------- | ------- | ------------- | --- | ------ | --------- | ----- |
|              |     |            |      |          |             |             |        | assumption | is          | further     | relaxed | in the        | MCG | method |           | [279] |
| Opt-attack   | by  | estimating | the  | sign     | of gradient | instead     | of the |            |             |             |         |               |     |        |           |       |
|              |     |            |      |          |             |             |        | that the   | conditional | adversarial |         | distributions |     | around | different |       |
gradient itself. The HopSkipJumpAttack method [33] proposes benign examples are similar but might be slightly different.
| to estimate               | the    | gradient | at the  | boundary        | point         | using          | the Monte  |                  |      |             |               |           |             |            |             |       |
| ------------------------- | ------ | -------- | ------- | --------------- | ------------- | -------------- | ---------- | ---------------- | ---- | ----------- | ------------- | --------- | ----------- | ---------- | ----------- | ----- |
|                           |        |          |         |                 |               |                |            | MCG proposes     |      | a meta      | learning      | framework |             | to firstly | learn       | a     |
| Carlo estimation          |        | method.  | The     | query-efficient |               | boundary-based |            |                  |      |             |               |           |             |            |             |       |
|                           |        |          |         |                 |               |                |            | meta conditional |      | adversarial | distribution, |           | which       | could      | be          | fine- |
| black-box                 | attack | (QEBA)   |         | [120]           | proposes      | to             | accelerate |                  |      |             |               |           |             |            |             |       |
|                           |        |          |         |                 |               |                |            | tuned to         | more | accurately  | capture       | the       | conditional |            | adversarial |       |
| gradient-estimation-based |        |          | methods |                 | by estimating | the            | gradient   |                  |      |             |               |           |             |            |             |       |
|                           |        |          |         |                 |               |                |            | distribution     | for  | new benign  | examples.     |           |             |            |             |       |
in the low-dimensional subspace instead of the original space. e) Different benign examples have similar gradients to
| The qFool | method   | [150] | approximates |     | the                | gradient | of the |                    |     |                |     |         |             |     |     |     |
| --------- | -------- | ----- | ------------ | --- | ------------------ | -------- | ------ | ------------------ | --- | -------------- | --- | ------- | ----------- | --- | --- | --- |
|           |          |       |              |     |                    |          |        | search adversarial |     | perturbations: |     | , i.e., | ∃i̸=j,      |     |     |     |
| current   | solution | by    | the gradient |     | of its neighboring |          | points |                    |     |                |     |         |             |     |     |     |
|           |          |       |              |     |                    |          |        |                    | ∂J  | (x(i),y(i))    |     | ∂J      | (x(j),y(j)) |     |     |     |
at the decision boundary, utilizing the low curvature of the G1 0 ε ≈ G2 0 ε ,
(25)
decision boundary at the vicinity of adversarial points. ∂x(i) ∂x(j)
|     |     |     |     |     |     |     |     | where J | (·,·) | denotes | the | adversarial | objective |     | function |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ----- | ------- | --- | ----------- | --------- | --- | -------- | --- |
G1
|                   |     |             |     |          |     |     |     | w.r.t. theattackedmodelG |      |           | ,andx(i) |             | indicatestheimmediate |     |          |     |
| ----------------- | --- | ----------- | --- | -------- | --- | --- | --- | ------------------------ | ---- | --------- | -------- | ----------- | --------------------- | --- | -------- | --- |
| D. Transfer-based |     | Adversarial |     | Examples |     |     |     |                          |      |           | 1        |             |                       |     |          |     |
|                   |     |             |     |          |     |     |     | solution                 | when | searching | the      | adversarial | sample                |     | of x(i). | For |
0
According to the level of the adversarial transferability, we example, the meta attack [64] trains a meta model that could
categorize existing transfer-based adversarial examples into directly generate a gradient w.r.t. the input sample, which is
example-level and model-level transferability. then fine-tuned on each new benign sample and new model to
1) Example-level Adversarial Transferability: The concept quickly generate effective gradients, rather than querying the
example-leveladversarialtransferabilityisfirstlyandexplicitly newmodeltoestimategradients,suchthattheattackefficiency
defined in a recent work called meta conditional generator and effectiveness is supposed to be improved.

19
2) Model-level Adversarial Transferability: Model-level c) Loss perspective: Some works attempt to design novel
adversarial transferability tells that adversarial perturbations loss functions to generate more transferable adversarial pertur-
generated based on one model may be also adversarial for bations, rather than the widely used cross entropy loss defined
another model. There have been several attempts to improve based on the model prediction and ground-truth or adversarial
the model-level transferability from different perspectives. labels.Forexample,thefeaturedistributionattack(FDA)[102]
a) Data perspective: Inspired by the data augmentation firstly trains an auxiliary binary classifier of the intermediate
technique that alleviates the overfitting of the trained model layer features w.r.t. the target class, then maximizes the
to the training dataset, the attacker firstly conducts a random posterior probability predicted by this classifier to generate
transformationonthebenignsample,thengeneratesadversarial adversarialexamples.Itislaterextendedfromoneintermediate
perturbations based on the transformed sample w.r.t. the layer to multiple layers in [103]. The intermediate level attack
surrogate model, such that the generated adversarial sample projection attack (ILAP) [98] maximized the difference of
doesn’t overfit to the surrogate model too much. For example, intermediate layer features between adversarial and benign
the diverse inputs iterative FGSM (DI2-FGSM) algorithm inputs, while keeping close to an existing adversarial example
[261] proposes to insert random resizing and padding into intheintermediatefeaturespace.Thefeatureimportance-aware
the input sample. A more common setting is replacing the attack(FIA)[238]disruptsimportantobject-awareintermediate
benignsamplebyasetofvariantswithrandomtransformations, features in the surrogate model, and the feature importance
such as random scaling (i.e., resizing) [140], random mixup is calculated by averaging the gradients w.r.t. feature maps
[234], random translation [61], as well as adding random of the surrogate model. The neuron attribution-based attack
Gaussian noises [252]. The object-based diverse input (ODI) (NAA) [292] extends FIA by measuring feature importance
method [20] aims to improve the transferability of targeted by neuron attribution. The work [304] find that maximizing
adversarial samples through a complex transformation that is the logit w.r.t. the target class using I-FGSM method with
implemented by firstly printing the original adversarial sample sufficient iterations could generate adversarial examples with
on 3D objects’ surface, then rendering these 3D objects in a hightargetedtransferability.Theinteraction-reducedattack(IR)
varietyofrenderingenvironmentstoobtaindiversetransformed [236] empirically verifies that “the adversarial transferability
adversarial samples. Besides, the spectrum simulation iterative and the interactions inside adversarial perturbations are nega-
FGSM (S2I-FGSM) [154] designs a spectrum transformation tively correlated", and proposes an interaction loss to generate
based on discrete cosine transform (DCT) and inverse discrete high transferable perturbations. In addition to the above losses
cosine transform (IDCT) techniques, to generate more diverse defined based on intermediate layers, the reverse adversarial
inputs than the transformations in the spatial domain. perturbation attack (RAP) [189] proposes a novel min-max
b) Optimization perspective: To avoid the underfitting loss, where the adversarial example is perturbed by adding a
of the one-step gradient sign method (i.e., FGSM [86]) and reverse adversarial perturbation. It encouraged to search for
the overfitting of the multi-step gradient sign method (i.e., I- flat local minimums which are more robust to model changes,
FGSM[116])tothesurrogatemodel,somevariantsofgradient- leading to higher transferability.
based optimization algorithms are introduced to generate
powerful adversarial examples with good transferability, such
asthemomentum-basedgradient(e.g.,MI-FGSM[60])andits
variants (e.g., VMI-FGSM [232] which consider the gradient d) Surrogate model perspective: Some attempts focus on
variance in the vicinity of the current data point into the choosing or adjusting surrogate models to improve transferabil-
momentum, EMI-FGSM [235] which calculates the average ity. For example, the work [214] empirically demonstrates that
gradient of multiple points in the vicinity of the current data the slightly robust surrogate model (i.e., adversarially training
point,andSVRE-MI-FGSM[263]whichextendsMI-FSGMto with moderate perturbation budget) could generate highly
the ensemble attack with reduced gradient variance), as well as transferable adversarial perturbations. The Ghost networks
the Nesterov accelerated gradient (e.g., NI-FGSM) [140] and attack [130] firstly perturbs a fixed surrogate model by densely
itsvariant(e.g.,PI-FSGM[235]whichreplacetheaccumulated inserting dropout layers and randomly adjusting residual
momentum in NI-FGSM [140] by a new momentum that only weight to generate multiple surrogate models, then adopts the
accumulates the local gradient of the previous step). Besides, longitudinal ensemble that each step updating of adversarial
there are also a few attempts to modify the gradients. For perturbation is calculated based on one randomly selected
examples, the SGM (skip gradient method) attack [251] find surrogate model. The intrinsic adversarial attack (IAA) [312]
that backpropagating more gradients from the skip connections hypothesizes that samples at the low-density region of the
than the residual modules in the ResNet-like models could ground truth data distribution where models are not well
generate higher transferable adversarial examples. The linear trained are more transferable. Thus, it proposes to maximize
backpropagation attack (LinBP) [90] proposes to omit the the matching between the gradient of adversarial samples and
ReLU layers in backpropagation pass to improve adversarial the direction toward the low-density regions. The distribution-
transferability. The meta gradient adversarial attack (MGAA) relevant attack (DRA) [311] fine-tunes the surrogate model to
[285]utilizesmetalearningtolearnageneralizedmetagradient encourage the gradient similarity between the model and the
by treating the attack against one model as one individual task, ground truth data distribution, according to the hypothesis that
such that the meta gradient can be quickly fine-tuned to find adversarial samples away from the original distribution of the
effective adversarial perturbations for new models. benign sample are highly transferable.

20
VIII. ADVERSARIALMACHINELEARNINGINOTHER searching for discrete tokens via a gradient search algorithm.
SCENARIOS PromptInject method [181] investigates two types of prompt
injection to misalign the goals of GPT-3, where goal hijacking
Recently, diffusion models and large language models
misaligned the original goal of a prompt to a new goal of
have shown superior understanding and generative abilities in
printing a target phrase, and prompt leaking aimed to output
visual and language fields respectively, which have stimulated
theoriginalprompt.BadPromptmethod[21]conductsbackdoor
widespread attention in the AI community. Despite their
attack to continuous prompts and proposes an adaptive trigger
extraordinary capabilities, recent studies have presented the
optimizationalgorithmtoautomaticallyselectthemosteffective
vulnerabilities of these models under malicious attacks. In this
and invisible trigger for each sample. BadGPT [205] aims
section, we mainly detail proposed attacks on diffusion models
to attack against RL fine-tuning paradigm of LLMs via
(see Section VIII-A) and large language models (see Section
backdooring reward model. Wan et al. [226] show that the
VIII-B).
poisoning dataset used for pertaining LLMs by bay bad-of-
words approximation will cause test errors even for held-out
A. Attack on Diffusion Models tasks that were not poisoned during training time. HOUYI
Diffusion models are a class of deep generative models that method[147]appliesasystematicapproachtopromptinjection
learn forward and reverse diffusion processes via progressive on LLMs by drawing from SQL injection and XSS attacks.
noise-addition and denoising. Chou et al. [46] first study PoisonPrompt method [276] compromises both hard and soft
the robustness of diffusion model against backdoor attacks prompt-based LLMs by a bi-level optimization-based backdoor
and propose BadDiffusion, which implants backdoor into the attackwithtwoprimaryobjectives:first,tooptimizethetrigger
diffusion processes by specific triggers and target images. At used for activating the backdoor behavior, and second, to train
theinferencestage,thebackdooreddiffusionmodelwillbehave the prompt tuning task. AutoDAN method [310] automatically
just like an untampered generator for regular data inputs, while generates interpretable prompts to jailbreak LLMs which can
falsely generating some targeted outcome designed by the bad bypass perplexity-based filters while maintaining a high attack
actor upon receiving the implanted trigger signal. TrojDiff [38] success rate like manual jailbreak attacks. TrojLLM method
designs transitions to diffuse a pre-defined target distribution [271] implements a black-box backdoor attack by universal
to the Gaussian distribution biased by a specific trigger, and API-driven trigger discovery and progressive prompt poisoning.
then proposes a parameterization of the generative process to Unlike other methods, it assumes that the attack can only
reverse the trojan diffusion process via an effective training query LLMs-based APIs, while having no access to the inner
objective.Unliketheabovetwomethods,TargetPromptAttack workings of LLMs, such as architecture, parameters, gradients,
(TPA) and Target Attribute Attack (TAA) [215] aims to inject and so on.
backdoor into the pre-trained text encoder of the text-to-image
synthesis diffusion models. By inserting a single character IX. APPLICATIONS
trigger into the prompt, the attacker can trigger the model to Attackparadigmsmentionedabovearedouble-edgedswords.
generateimageswithpredefinedattributesorimagesthatfollow Ontheonehand,theycanindeedcompromisemachinelearning
a hidden, potentially malicious description. Chou et al. [47] system to achieve malicious goals. But on the other hand, such
further propose VillanDiffusion, which is a unified backdoor negative effectiveness can be turned into goodness for some
attack framework for diffusion models that covers mainstream specific tasks. For example, backdoor attacks can be used
unconditional and conditional diffusion models (denoising- for copyright protection and adversarial attacks can be used
based and score-based) and various training-free samplers for for privacy protection. In this section, we will introduce the
holistic evaluations. positive applications of different attack paradigms.
B. Attack on Large Language Models A. Backdoor Attacks for Copyright Protection
Large language models (LLMs) are a type of pre-trained Adi et al. [1] propose to watermark deep neural networks
language model notable for their abilities to achieve general- by backdoor attack to identify models as the intellectual
purpose language understanding and generation, which have property of a particular vendor. Sommer et al. [211] design a
made remarkable progress toward achieving artificial general backdoorbasedverificationmechanismformachineunlearning,
intelligence. However, LLMs are still vulnerable to malicious in which each user can utilize backdoor techniques to verify
attacks. Xu et al. [266] explore the universal vulnerability whether the MLaaS provider deleted their training data from
of the pre-training paradigm of LLMs by either injecting the backdoored model by checking the attack success rate
backdoor triggers with poisoned samples or searching for using its own trigger with the target label. Li et al. [135]
adversarial triggers using only plain text. Then these triggers propose a backdoor based dataset copyright protection method
can be used to control the outputs after fine-tuning on the byfirstadoptingdata-poisoningbasedbackdoorattackandthen
downstream tasks. Poisoned Prompt Tuning (PPT) method conducting ownership verification by verifying whether the
[66] aims to embed backdoor into the soft prompt via prompt backdoored model has targeted backdoor behaviors. Li et al.
tuningonthepoisoneddatasetandthebackdoorwillbeloaded [136] design a black-box dataset ownership verification based
into LLMs by using the soft prompt. PromptAttack method on targeted backdoor attacks and pair-wise hypothesis tests.
[207] constructs malicious prompt templates by automatically However,theembeddedbackdoorcanbemaliciouslyexploited

21
by the attackers to manipulate model predictions. Li et al. largerflexibilityandstrongercapabilitytoevadebackdoor
| [131] propose |     | a untargeted |     | backdoor | watermarking |     | scheme | to defenses. |     |     |     |     |     |     |     |
| ------------- | --- | ------------ | --- | -------- | ------------ | --- | ------ | ------------ | --- | --- | --- | --- | --- | --- | --- |
alleviate this problem, where the backdoored model behaviors In terms of the target class, the single-target setting
•
are not deterministic. ROWBACK method [28] improves the is still the dominant setting in backdoor attacks, while
robustness of watermarking by redesigning the trigger set the multi-target setting is often evaluated as extended
based on adversarial examples and modifying the marking experiments in a few works. However, we think that some
mechanism to ensure thorough distribution of the embedded interestingpointsofthelattersettingdeservetobestudied.
watermarks.MIBmethod[95]leveragesbackdoortoeffectively First, how many target classes could be embedded into
infer whether a data sample was used to train an ML model or a dataset, and what is the relationship between attack
notbypoisoningasmallnumberofsamplesandonlyblack-box performance and target class numbers? It will study the
access to the target model. capability of backdoor injection of one model. Second,
|                |     |          |     |         |            |             |     | what      | is the     | difference | between          |         | single-target |          | and multi-  |
| -------------- | --- | -------- | --- | ------- | ---------- | ----------- | --- | --------- | ---------- | ---------- | ---------------- | ------- | ------------- | -------- | ----------- |
|                |     |          |     |         |            |             |     | target    | backdoored |            | models?          | It will | be helpful    | to       | develop     |
| B. Adversarial |     | Examples | for | Privacy | Protection |             |     |           |            |            |                  |         |               |          |             |
|                |     |          |     |         |            |             |     | effective |            | defenses   | for multi-target |         | classes       | attacks, | while       |
| The increasing |     | leakage  | and | misuse  | of visual  | information |     |           |            |            |                  |         |               |          |             |
|                |     |          |     |         |            |             |     | most      | existing   | defenses   | are              | mainly  | designed      |          | for single- |
raisessecurityandprivacyconcerns.Infact,thenegativeeffects targetclassattacks.Third,whatistherelationshipbetween
| of adversarial |     | attacks | can | be utilized | positively |     | to protect |     |           |     |           |         |     |                |     |
| -------------- | --- | ------- | --- | ----------- | ---------- | --- | ---------- | --- | --------- | --- | --------- | ------- | --- | -------------- | --- |
|                |     |         |     |             |            |     |            | the | backdoors | of  | different | targets | in  | one backdoored |     |
privacy.Ohetal.[108]proposedagametheoreticalframework
|     |     |     |     |     |     |     |     | model? | It  | will be | useful | to design | advanced | attack | and |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | --- | ------- | ------ | --------- | -------- | ------ | --- |
between a social media user and a recogniser to explore defense methods in multi-target settings.
| adversarial | image | perturbations |     | for | privacy | protection, | where |                                               |     |     |     |     |     |     |          |
| ----------- | ----- | ------------- | --- | --- | ------- | ----------- | ----- | --------------------------------------------- | --- | --- | --- | --- | --- | --- | -------- |
|             |       |               |     |     |         |             |       | 2) Training-controllablebasedBackdoorAttacks: |     |     |     |     |     |     | Compared |
the user perturbs the image to confuse the recognizer and the to the above threat model, the training-controllable based
| recognizer | chooses | blue | strategy | as  | a countermeasure. |     | Privacy- |          |         |      |          |      |          |         |         |
| ---------- | ------- | ---- | -------- | --- | ----------------- | --- | -------- | -------- | ------- | ---- | -------- | ---- | -------- | ------- | ------- |
|            |         |      |          |     |                   |     |          | backdoor | attacks | have | not been | well | studied, | as they | require |
preservingFeatureExtractionbasedonAdversarialTraining(P-
|             |            |         |     |             |          |        |            | the control | of       | the training | process, | which          |          | seems          | to be less |
| ----------- | ---------- | ------- | --- | ----------- | -------- | ------ | ---------- | ----------- | -------- | ------------ | -------- | -------------- | -------- | -------------- | ---------- |
| FEAT) [55]  | method     | employs |     | adversarial | training | to     | strengthen |             |          |              |          |                |          |                |            |
|             |            |         |     |             |          |        |            | practical.  | However, | along        | with     | the popularity |          | of pre-trained |            |
| the privacy | protection |         | of  | an encoder  | in a     | neural | network,   |             |          |              |          |                |          |                |            |
|             |            |         |     |             |          |        |            | large-scale | models,  | the          | backdoor | threat         | in these | models         | could      |
ensuring reduced privacy leakage without compromising task widely spread to downstream tasks of different domains, and
| accuracy. | Adversarial |     | Privacy-preserving |        | Filter      | (APF) | [291] |          |           |        |     |         |                |     |          |
| --------- | ----------- | --- | ------------------ | ------ | ----------- | ----- | ----- | -------- | --------- | ------ | --- | ------- | -------------- | --- | -------- |
|           |             |     |                    |        |             |       |       | the main | challenge | is how | to  | improve | the resistance |     | to fine- |
| method    | protects    | the | online             | shared | face images | from  | being |          |           |        |     |         |                |     |          |
tuninganddomaingeneralization.Besides,duetothesufficient
| maliciously     | used | by         | an end-cloud |     | collaborated |     | adversarial |            |                |     |         |       |          |      |          |
| --------------- | ---- | ---------- | ------------ | --- | ------------ | --- | ----------- | ---------- | -------------- | --- | ------- | ----- | -------- | ---- | -------- |
|                 |      |            |              |     |              |     |             | capability | of large-scale |     | models, | it is | expected | that | multiple |
| attack solution |      | to satisfy | requirements |     | of privacy,  |     | utility and |            |                |     |         |       |          |      |          |
backdoorswithdifferenttypescouldbeinserted,posingserious
non-accessibility. Text-space adversarial attack method (AaaD) challenges for defense.
| [128] method | explores |     | the utilization |        | of adversarial |                | attacks | to        |         |     |     |     |     |     |     |
| ------------ | -------- | --- | --------------- | ------ | -------------- | -------------- | ------- | --------- | ------- | --- | --- | --- | --- | --- | --- |
| protect data | privacy  | on  | social          | media. | It focuses     | on obfuscating |         |           |         |     |     |     |     |     |     |
|              |          |     |                 |        |                |                |         | B. Weight | Attacks |     |     |     |     |     |     |
users’attributesbygeneratingsemanticallyandvisuallysimilar
wordperturbations,provingitseffectivenessagainstattributein- We note that most weight attack methods still stay in
|                  |                 |             |             |        |             |               |        | theoretical | analysis. | To         | the    | best of | our knowledge, |     | there   |
| ---------------- | --------------- | ----------- | ----------- | ------ | ----------- | ------------- | ------ | ----------- | --------- | ---------- | ------ | ------- | -------------- | --- | ------- |
| ference attacks. |                 | Adversarial |             | Visual | Information | Hiding        | (AVIH) |             |           |            |        |         |                |     |         |
|                  |                 |             |             |        |             |               |        | hasn’t been | any       | successful | attack | against | intelligent    |     | systems |
| method           | [217] generates |             | obfuscating |        | adversarial | perturbations |        |             |           |            |        |         |                |     |         |
to obscure the visual information of the data. Meanwhile, it in real scenarios. We think the main reason is that the
|           |     |        |            |     |              |           |     | success | of weight | attack | is built | upon | physical | fault | injection |
| --------- | --- | ------ | ---------- | --- | ------------ | --------- | --- | ------- | --------- | ------ | -------- | ---- | -------- | ----- | --------- |
| maintains | the | hidden | objectives | to  | be correctly | predicted | by  |         |           |        |          |      |          |       |           |
models. techniques that can precisely manipulate each bit in memory,
|     |     |     |                |     |     |     |     | such as          | Rowhammer    | attack     | [3],    | or Laser | Beam    | attack    | [112].   |
| --- | --- | --- | -------------- | --- | --- | --- | --- | ---------------- | ------------ | ---------- | ------- | -------- | ------- | --------- | -------- |
|     |     |     |                |     |     |     |     | These techniques |              | often      | require | some     | special | equipment | or       |
|     |     |     | X. DISCUSSIONS |     |     |     |     |                  |              |            |         |          |         |           |          |
|     |     |     |                |     |     |     |     | computer         | architecture | knowledge, |         | which    | is      | difficult | for most |
A. Backdoor Attacks AI researchers. However, this barrier can be tackled through
1) Data-poisoning based Backdoor Attacks: cross-disciplinary cooperation, and the practical security threat
|                   |       |          |          |          |          |              |          | of weight      | attack | deserves | more | attention | in  | the future. |     |
| ----------------- | ----- | -------- | -------- | -------- | -------- | ------------ | -------- | -------------- | ------ | -------- | ---- | --------- | --- | ----------- | --- |
| • In              | terms | of       | the      | trigger  | type,    | visible/non- |          |                |        |          |      |           |     |             |     |
| semantic/manually |       |          | designed | triggers | have     | been         | widely   |                |        |          |      |           |     |             |     |
|                   |       |          |          |          |          |              |          | C. Adversarial |        | Examples |      |           |     |             |     |
| adopted           | in    | existing | works.   |          | However, | along        | with the |                |        |          |      |           |     |             |     |
development of backdoor defense, the characteristics 1) White-box Adversarial Attacks: After thorough explo-
of these types have been thoroughly explored and then rationofwhite-boxadversarialattacks,therehavebeenmassive
utilized to develop more effective defense methods. Thus, and diverse methods, and now it is rare to see new white-
we think that future backdoor attacks are more likely box attack methods. Instead, white-box adversarial examples
to adopt invisible/semantic/learnable triggers, to evade have been used as useful tools for other tasks, such as
existing backdoor defenses. adversarialtraining(e.g.,usingwhite-boxadversarialexamples
In terms of the fusion strategy, additive/static/sample- togenerateadversarialexamplesduringthetrainingprocedure),
•
agnostic triggers have been widely adopted in existing backdoor attack (e.g., using universal adversarial perturbation
works, but we think that non-additive/dynamic/sample- as the trigger, or erasing the original information of poisoned
specific triggers will be the future trend, due to the samples in label-consistent attacks), transfer-based attacks

22
(i.e., generating transferable adversarial examples on white- this task. More importantly, we still lack a clear theoretical
box surrogate models), privacy protection (i.e., erasing the understanding of the intrinsic reason and characteristics of
information of main objects in one sample to evade the third- adversarial transferability, though several effective heuristic
party detection system), etc. strategies or assumptions have been developed. We suggest
2) Black-box Adversarial Attacks: that a solid theoretical analysis should consider the factors of
data distribution, model architectures, loss landscape, decision
• In terms of the score-based black-box adversarial surface, and get inspiration from the theory about model
examples, the combination-based methods have shown
generalizationacrossdifferentdatadistributions.Besides,better
much better performance than the query-based methods,
understanding about adversarial transferability will be also
demonstrating the benefit of utilizing the surrogate priors.
beneficial to design more robust models in practice.
We think that the potentials of further improving attack
performancelieinshrinkingthegapbetweensurrogateand
targetmodels,extractingmoreusefulpriorsfromsurrogate D. Comparisons of Three Attack Paradigms
models, and effectively combining surrogate priors and
Until now, the differences among the three attack paradigms
queryfeedback.Besides,thereportedresultsinsomelatest
have been clearly described, and it is found that their de-
works (e.g., CG-Attack [77] and MCG [279]) are very
velopments are almost independent. But there are still a few
good, and the median query number even achieves 1 and
interactions. For example, although without explicit claims, it
the attack success rate achieves 100% at some easy cases
is obvious that the design of triggers or poisoned samples in
(e.g., small data dimension, untargeted attack, regularly
backdoorattacksgotinspirationfrominference-timeadversarial
trained target model). It seems that the performance of
attacks, such as invisible triggers, or non-additive triggers.
score-based attacks is hitting the ceiling. However, it is
Besides, some adversarial examples were directly utilized
notable that all these evaluations are conducted under
in backdoor attacks, such as using the targeted universal
the setting of no defense. As shown in recent black-
adversarial perturbation [162] as backdoor trigger [293], or
box defense methods, e.g., RND (slightly perturbing the
using inference-time adversarial examples to erase the original
query input) [190] and AAA (slightly perturbing the
benign features in label-consistent backdoor attack [301].
query feedback) [36], several SOTA black-box adversarial
However, since these three attack paradigms occur in different
examples significantly degraded. It implies that future
stages of a machine learning system, it is difficult to integrate
score-basedadversarialexamplesshouldbedefense-aware.
them to implement a unified attack. In contrast, we think
• In terms of the decision-based black-box adversarial
that their interactions may be more close when taking the
examples, all existing decision-based methods are query-
defense into account. Specifically, when designing a defense to
based. Although some priors are also extracted from
improvethemodelrobustnesstooneparticularattackparadigm,
surrogate models, they are used as fixed priors, without
one should consider whether the robustness to other attack
interactionwithqueryfeedbacklikethecombination-based
paradigmswillbeharmedornot.Forexample,itisvaluableto
methodsinscore-basedadversarialexamples.Besides,due
study the risk of adversarial training [106], [158] to backdoor
to the less feedback information, its attack performance
attack, and whether the adversarially trained model is more
is much poor than the score-based black-box adversarial
vulnerable to weight attack. Some recent backdoor defense
examples. We think it is valuable to explore combination-
works [39], [96] shows that the backdoor injection could be
based decision-based adversarial examples.
inhibited through replacing the standard end-to-end supervised
• One commonality of score-based and decision-based
training by some well-designed secure training algorithms.
adversarial examples is that targeted adversarial examples
However, whether the robustness of such trained model to
are much more challenging than untargeted adversarial
adversarial examples or weight attacks is also improved or
examples, reflected by more queries and lower attack
not should be further studied. In summary, we think that it is
performance, mainly due to the smaller region of one
valuable to consider the above three attack paradigms from a
target class than the non-ground-truth class region. We
systematic perspective, otherwise, the security of a machine
think one feasible solution to improve targeted attack
learning system cannot be really improved.
performance is accurately modeling the adversarial per-
turbation distribution conditioned on benign sample and
target class. XI. SUMMARY
3) Transfer-based Adversarial Examples: Compared with In this survey, we have proposed a unified definition and
white-box and black-box adversarial examples, the transfer- mathematical formulation about adversarial machine learning
based adversarial examples have no requirement about the (AML), covering three main attack paradigms, including
attacked model, posing higher practical threats. According to backdoor attack at the training stage, weight attack at the
the reported evaluations in existing transfer-based adversarial deployment stage and adversarial attack at the testing stage.
examples,weobservedthattheattackperformanceiswellifthe This unified framework provided a systematic perspective of
trainingdatasetsandarchitecturesbetweensurrogateandtarget AML, which could not only help readers to quickly obtain a
models are similar, otherwise the attack performance is very comprehensive understanding of this field, but also calibrate
poor.Itimpliesthatimprovingadversarialtransferabilityacross different paradigms to accelerate the overall development of
datasets and model architectures is still the main challenge of AML.

23
| Supplementary      | materials.    |            | Due to | the space    | limit, | several    |                                  |                                            |         |                          |             |           |     |
| ------------------ | ------------- | ---------- | ------ | ------------ | ------ | ---------- | -------------------------------- | ------------------------------------------ | ------- | ------------------------ | ----------- | --------- | --- |
|                    |               |            |        |              |        |            | [24] Nicholas                    | Carlini and David                          | Wagner. | Audio                    | adversarial | examples: |     |
|                    |               |            |        |              |        |            | Targetedattacksonspeech-to-text. |                                            |         | InIEEES&PWorkshops,2018. |             |           |     |
| additional         | but important | contents   | will   | be presented |        | as supple- |                                  |                                            |         |                          |             |           |     |
|                    |               |            |        |              |        |            | [25] UdayKChakraborty.           | Advancesindifferentialevolution,volume143. |         |                          |             |           |     |
| mentary materials, |               | including: |        |              |        |            |                                  |                                            |         |                          |             |           |     |
Springer,2008.
|                |     |              |           |     |      |           | [26] Alvin | Chan, Yi Tay, Yew-Soon |     | Ong, and | Aston | Zhang. | Poison |
| -------------- | --- | ------------ | --------- | --- | ---- | --------- | ---------- | ---------------------- | --- | -------- | ----- | ------ | ------ |
| 1) Comparisons | of  | three attack | paradigms |     | from | different |            |                        |     |          |       |        |        |
attacksagainsttextdatasetswithconditionaladversariallyregularized
| perspectives; |     |     |     |     |     |     | autoencoder. | InFindingsofEMNLP,2020. |     |     |     |     |     |
| ------------- | --- | --- | --- | --- | --- | --- | ------------ | ----------------------- | --- | --- | --- | --- | --- |
2) Summary of all mentioned weight attack methods; [27] Chin-Chen Chang, Ju-Yuan Hsiao, and Chi-Shiang Chan. Finding
optimalleast-significant-bitsubstitutioninimagehidingbydynamic
3) Atableofrelatedtoolboxesorbenchmarksinthecommunity
|     |     |     |     |     |     |     | programmingstrategy. | PatternRecognition,36(7):1583–1595,2003. |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------- | ---------------------------------------- | --- | --- | --- | --- | --- |
of adversarial machine learning; [28] NandishChattopadhyayandAnupamChattopadhyay. Rowback:Robust
4) Summary of the associations between each individual attack watermarkingforneuralnetworksusingbackdoors. InICMLA,2021.
|     |     |     |     |     |     |     | [29] Chien-Lun | Chen, Leana | Golubchik, | and | Marco Paolieri. | Backdoor |     |
| --- | --- | --- | --- | --- | --- | --- | -------------- | ----------- | ---------- | --- | --------------- | -------- | --- |
method and its categorizations. attacksonfederatedmeta-learning. InNeurIPS,2020.
|     |     |     |     |     |     |     | [30] HuiliChen,ChengFu,JishenZhao,andFarinazKoushanfar. |     |     |     |              |     | Proflip: |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------- | --- | --- | --- | ------------ | --- | -------- |
|     |     |     |     |     |     |     | Targetedtrojanattackwithprogressivebitflips.            |     |     |     | InICCV,2021. |     |          |
REFERENCES [31] Hongge Chen, Huan Zhang, Pin-Yu Chen, Jinfeng Yi, and Cho-Jui
|     |     |     |     |     |     |     | Hsieh. | Attackingvisuallanguagegroundingwithadversarialexamples: |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------ | -------------------------------------------------------- | --- | --- | --- | --- | --- |
InACL,2018.
[1] YossiAdi,CarstenBaum,MoustaphaCisse,BennyPinkas,andJoseph Acasestudyonneuralimagecaptioning.
|         |         |               |                               |             |              |      | [32] JinghuiChenandQuanquanGu. |     |                   | Rays:Araysearchingmethodfor |     |     |     |
| ------- | ------- | ------------- | ----------------------------- | ----------- | ------------ | ---- | ------------------------------ | --- | ----------------- | --------------------------- | --- | --- | --- |
| Keshet. | Turning | your weakness | into                          | a strength: | Watermarking | deep |                                |     |                   |                             |     |     |     |
|         |         |               | InUSENIX,pages1615–1631,2018. |             |              |      | hard-labeladversarialattack.   |     | InACMSIGKDD,2020. |                             |     |     |     |
neuralnetworksbybackdooring.
[2] AkshayAgarwal,RichaSingh,MayankVatsa,andNaliniRatha. Are [33] Jianbo Chen, Michael I Jordan, and Martin J Wainwright. Hop-
|     |     |     |     |     |     |     | skipjumpattack:Aquery-efficientdecision-basedattack. |     |     |     |     | InIEEES&P, |     |
| --- | --- | --- | --- | --- | --- | --- | ---------------------------------------------------- | --- | --- | --- | --- | ---------- | --- |
image-agnosticuniversaladversarialperturbationsforfacerecognition
pages1277–1294.IEEE,2020.
| difficulttodetect? |     | InBTAS.IEEE,2018. |     |     |     |     |     |     |     |     |     |     |     |
| ------------------ | --- | ----------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
[3] Michel Agoyan, Jean-Max Dutertre, Amir-Pasha Mirbaha, David [34] JinyinChen,HaibinZheng,HuiXiong,ShijingShen,andMengmeng
|           |           |          |           |       |     |                | Su. | Mag-gan:Massiveattackgeneratorviagan. |     |     | InformationSciences, |     |     |
| --------- | --------- | -------- | --------- | ----- | --- | -------------- | --- | ------------------------------------- | --- | --- | -------------------- | --- | --- |
| Naccache, | Anne-Lise | Ribotta, | and Assia | Tria. | How | to flip a bit? |     |                                       |     |     |                      |     |     |
536:67–90,2020.
InIOLTS,2010.
[4] Nasir Ahmed, T_ Natarajan, and Kamisetty R Rao. Discrete cosine [35] SiminChen,HanlinChen,MirazulHaque,CongLiu,andWeiYang.
transform. IEEETransactionsonComputers,100(1):90–93,1974. Thedarksideofdynamicroutingneuralnetworks:Towardsefficiency
|                               |     |     |                                  |     |     |     | backdoorinjection. | InCVPR,2023. |     |     |     |     |     |
| ----------------------------- | --- | --- | -------------------------------- | --- | --- | --- | ------------------ | ------------ | --- | --- | --- | --- | --- |
| [5] NaveedAkhtarandAjmalMian. |     |     | Threatofadversarialattacksondeep |     |     |     |                    |              |     |     |     |     |     |
[36] SizheChen,ZhehaoHuang,QinghuaTao,YingwenWu,CihangXie,
| learningincomputervision:Asurvey. |     |     |     | IEEEAccess,6:14410–14430, |     |     |     |     |     |     |     |     |     |
| --------------------------------- | --- | --- | --- | ------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
2018. andXiaolinHuang. Adversarialattackonattackers:Post-processto
[6] AbdullahAl-DujailiandUna-MayO’Reilly. Signbitsareallyouneed mitigateblack-boxscore-basedqueryattacks. InNeurIPS,2022.
[37] Shang-TseChen,CoryCornelius,JasonMartin,andDuenHorngPolo
| forblack-boxattacks. |     | InICLR,2019. |     |     |     |     |     |     |     |     |     |     |     |
| -------------------- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
[7] MaksymAndriushchenko,FrancescoCroce,NicolasFlammarion,and Chau. Shapeshifter:Robustphysicaladversarialattackonfasterr-cnn
MatthiasHein. Squareattack:aquery-efficientblack-boxadversarial objectdetector. InECMLPKDD,2018.
|                        |                                            |              |     |     |     |      | [38] Weixin                        | Chen, Dawn Song, | and | Bo Li.       | Trojdiff: | Trojan attacks | on  |
| ---------------------- | ------------------------------------------ | ------------ | --- | --- | --- | ---- | ---------------------------------- | ---------------- | --- | ------------ | --------- | -------------- | --- |
| attackviarandomsearch. |                                            | InECCV,2020. |     |     |     |      |                                    |                  |     |              |           |                |     |
|                        |                                            |              |     |     |     |      | diffusionmodelswithdiversetargets. |                  |     | InCVPR,2023. |           |                |     |
| [8] DonovanArtz.       | Digitalsteganography:hidingdatawithindata. |              |     |     |     | IEEE |                                    |                  |     |              |           |                |     |
InternetComputing,5(3):75–80,2001. [39] WeixinChen,BaoyuanWu,andHaoqianWang. Effectivebackdoor
[9] Anish Athalye, Logan Engstrom, Andrew Ilyas, and Kevin Kwok. defense by exploiting sensitivity of poisoned samples. In NeurIPS,
2022.
| Synthesizingrobustadversarialexamples. |     |     |     | InICML,2018. |     |     |     |     |     |     |     |     |     |
| -------------------------------------- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
[10] EugeneBagdasaryan,AndreasVeit,YiqingHua,DeborahEstrin,and [40] WeilunChen,ZhaoxiangZhang,XiaolinHu,andBaoyuanWu.Boosting
VitalyShmatikov. Howtobackdoorfederatedlearning. InAISTATS, decision-basedblack-boxadversarialattackswithrandomsignflip. In
| 2020.        |              |     |         |                 |      |           | ECCV,2020.  |                  |     |              |     |          |       |
| ------------ | ------------ | --- | ------- | --------------- | ---- | --------- | ----------- | ---------------- | --- | ------------ | --- | -------- | ----- |
|              |              |     |         |                 |      |           | [41] Xinyun | Chen, Chang Liu, | Bo  | Li, Kimberly | Lu, | and Dawn | Song. |
| [11] Jiawang | Bai, Baoyuan | Wu, | Zhifeng | Li, and Shu-tao | Xia. | Versatile |             |                  |     |              |     |          |       |
weightattackviaflippinglimitedbits.arXivpreprintarXiv:2207.12405, Targetedbackdoorattacksondeeplearningsystemsusingdatapoisoning.
| 2022. |     |     |     |     |     |     | arXivpreprintarXiv:1712.05526,2017. |     |     |     |     |     |     |
| ----- | --- | --- | --- | --- | --- | --- | ----------------------------------- | --- | --- | --- | --- | --- | --- |
[42] XiaoyiChen,AhmedSalem,DingfanChen,MichaelBackes,Shiqing
[12] JiawangBai,BaoyuanWu,YongZhang,YimingLi,ZhifengLi,and
|             |                                                    |     |     |     |     |     | Ma,QingniShen,ZhonghaiWu,andYangZhang. |     |     |     |     | Badnl:Backdoor |     |
| ----------- | -------------------------------------------------- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- | -------------- | --- |
| Shu-TaoXia. | Targetedattackagainstdeepneuralnetworksviaflipping |     |     |     |     |     |                                        |     |     |     |     |                |     |
limitedweightbits. InICLR,2021. attacksagainstnlpmodelswithsemantic-preservingimprovements. In
| [13] ShumeetBaluja. |     | Hidingimagesinplainsight:Deepsteganography. |     |     |     | In  | ACSAC,2021. |              |            |       |             |         |     |
| ------------------- | --- | ------------------------------------------- | --- | --- | --- | --- | ----------- | ------------ | ---------- | ----- | ----------- | ------- | --- |
|                     |     |                                             |     |     |     |     | [43] Minhao | Cheng, Thong | Le, Pin-Yu | Chen, | Huan Zhang, | Jinfeng | Yi, |
NeurIPS,2017.
[14] MauroBarni,KassemKallas,andBenedettaTondi. Anewbackdoor and Cho-Jui Hsieh. Query-efficient hard-label black-box attack: An
attackincnnsbytrainingsetcorruptionwithoutlabelpoisoning. In optimization-basedapproach. InICLR,2019.
|     |     |     |     |     |     |     | [44] Shuyu | Cheng, Yinpeng | Dong, Tianyu | Pang, | Hang | Su, and Jun | Zhu. |
| --- | --- | --- | --- | --- | --- | --- | ---------- | -------------- | ------------ | ----- | ---- | ----------- | ---- |
ICIP,2019.
[15] ArjunNitinBhagoji,SupriyoChakraborty,PrateekMittal,andSeraphin Improvingblack-boxadversarialattackswithatransfer-basedprior. In
| Calo. | Analyzing | federated | learning | through an | adversarial | lens. In | NeurIPS,2019. |     |     |     |     |     |     |
| ----- | --------- | --------- | -------- | ---------- | ----------- | -------- | ------------- | --- | --- | --- | --- | --- | --- |
ICML,2019. [45] SiyuanCheng,YingqiLiu,ShiqingMa,andXiangyuZhang. Deepfea-
turespacetrojanattackofneuralnetworksbycontrolleddetoxification.
| [16] ChristopherMBishopandNasserMNasrabadi. |     |                |     |     | PatternRecognition |     |              |     |     |     |     |     |     |
| ------------------------------------------- | --- | -------------- | --- | --- | ------------------ | --- | ------------ | --- | --- | --- | --- | --- | --- |
| andMachineLearning.                         |     | Springer,2006. |     |     |                    |     | InAAAI,2021. |     |     |     |     |     |     |
[17] MikelBober-Irizar,IliaShumailov,YirenZhao,RobertMullins,and [46] Sheng-YenChou,Pin-YuChen,andTsung-YiHo. Howtobackdoor
NicolasPapernot.Architecturalbackdoorsinneuralnetworks.InCVPR, diffusionmodels? InCVPR,2023.
|     |     |     |     |     |     |     | [47] Sheng-YenChou,Pin-YuChen,andTsung-YiHo. |     |     |     |     | Villandiffusion:A |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------------------------------- | --- | --- | --- | --- | ----------------- | --- |
2023.
[18] StephenBoyd,NealParikh,EricChu,BorjaPeleato,JonathanEckstein, unifiedbackdoorattackframeworkfordiffusionmodels. InNeurIPS,
| etal. | Distributedoptimizationandstatisticallearningviathealternating |     |     |     |     |     | 2023. |     |     |     |     |     |     |
| ----- | -------------------------------------------------------------- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- |
[48] FrancescoCroce,MaksymAndriushchenko,VikashSehwag,Edoardo
| directionmethodofmultipliers. |     |     | FoundationsandTrendsinMachine |     |     |     |     |     |     |     |     |     |     |
| ----------------------------- | --- | --- | ----------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Debenedetti,NicolasFlammarion,MungChiang,PrateekMittal,and
learning,3(1):1–122,2011.
[19] WielandBrendel,JonasRauber,andMatthiasBethge. Decision-based Matthias Hein. Robustbench: a standardized adversarial robustness
adversarialattacks:Reliableattacksagainstblack-boxmachinelearning benchmark. InNeurIPSDatasetsandBenchmarksTrack,2021.
|         |              |     |     |     |     |     | [49] Francesco | Croce and | Matthias | Hein. | Sparse | and imperceivable |     |
| ------- | ------------ | --- | --- | --- | --- | --- | -------------- | --------- | -------- | ----- | ------ | ----------------- | --- |
| models. | InICLR,2018. |     |     |     |     |     |                |           |          |       |        |                   |     |
[20] JunyoungByun,SeungjuCho,Myung-JoonKwon,Hee-SeonKim,and adversarialattacks. InICCV,2019.
Changick Kim. Improving the transferability of targeted adversarial [50] FrancescoCroceandMatthiasHein. Reliableevaluationofadversarial
examplesthroughobject-baseddiverseinput. InCVPR,2022. robustnesswithanensembleofdiverseparameter-freeattacks.InICML,
2020.
| [21] XiangruiCai,SihanXu,YingZhang,andXiaojieYuan. |     |     |     |     |     | Badprompt: |     |     |     |     |     |     |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
backdoorattacksoncontinuousprompts. InNeurIPS,2022. [51] FrancescoCroceandMatthiasHein. Reliableevaluationofadversarial
[22] Zikui Cai, Chengyu Song, Srikanth Krishnamurthy, Amit Roy- robustnesswithanensembleofdiverseparameter-freeattacks.InICML,
| Chowdhury,      | and | M Salman        | Asif. Black-box |     | attacks | via surrogate | 2020.      |                  |           |            |       |         |      |
| --------------- | --- | --------------- | --------------- | --- | ------- | ------------- | ---------- | ---------------- | --------- | ---------- | ----- | ------- | ---- |
|                 |     |                 |                 |     |         |               | [52] Ganqu | Cui, Lifan Yuan, | Bingxiang | He, Yangyi | Chen, | Zhiyuan | Liu, |
| ensemblesearch. |     | InNeurIPS,2022. |                 |     |         |               |            |                  |           |            |       |         |      |
[23] NicholasCarliniandDavidWagner. Towardsevaluatingtherobustness andMaosongSun. Aunifiedevaluationoftextualbackdoorlearning:
ofneuralnetworks. InIEEES&P,2017. Frameworksandbenchmarks. InNeurIPSDatasetsandBenchmarks

24
| Track,2022. |     |     |     |     | preprintarXiv:2007.10760,2020. |     |     |     |     |     |
| ----------- | --- | --- | --- | --- | ------------------------------ | --- | --- | --- | --- | --- |
[53] DebayanDeb,JianbangZhang,andAnilKJain. Advfaces:Adversarial [81] SiddhantGarg,AdarshKumar,VibhorGoel,andYingyuLiang. Can
facesynthesis. InIJCB.IEEE,2019. adversarialweightperturbationsinjectneuralbackdoors.InACMCIKM,
| [54] GavinWeiguangDing,LuyuWang,andXiaomengJin. |     |     |     | AdverTorch | 2020. |     |     |     |     |     |
| ----------------------------------------------- | --- | --- | --- | ---------- | ----- | --- | --- | --- | --- | --- |
v0.1:Anadversarialrobustnesstoolboxbasedonpytorch.arXivpreprint [82] LeonAGatys,AlexanderSEcker,andMatthiasBethge. Imagestyle
arXiv:1902.07623,2019. transferusingconvolutionalneuralnetworks. InCVPR,pages2414–
| [55] XiaofengDing,HongbiaoFang,ZhilinZhang,Kim-KwangRaymond |     |     |     |     | 2423,2016. |     |     |     |     |     |
| ----------------------------------------------------------- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- |
Choo,andHaiJin. Privacy-preservingfeatureextractionviaadversarial [83] BehnamGhavami,SeydMovi,ZhenmanFang,andLesleyShannon.
training. IEEE Transactions on Knowledge and Data Engineering, Stealthyattackonalgorithmic-protecteddnnsviasmartbitflipping. In
| 34(4):1967–1979,2020. |     |     |     |     | ISQED,2022. |     |     |     |     |     |
| --------------------- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- |
[56] Khoa Doan, Yingjie Lao, and Ping Li. Backdoor attack with [84] XueluanGong,YanjiaoChen,QianWang,HuayangHuang,Lingshuo
imperceptibleinputandlatentmodification. InNeurIPS,2021. Meng,ChaoShen,andQianZhang. Defense-resistantbackdoorattacks
[57] KhoaDoan,YingjieLao,WeijieZhao,andPingLi. Lira:Learnable, againstdeepneuralnetworksinoutsourcedcloudenvironment. IEEE
imperceptibleandrobustbackdoorattacks. InICCV,2021. JournalonSelectedAreasinCommunications,2021.
[58] KhoaDDoan,YingjieLao,andPingLi.Marksmanbackdoor:Backdoor [85] IanGoodfellow,JeanPouget-Abadie,MehdiMirza,BingXu,David
| attackswitharbitrarytargetclass. |     |     | InNeurIPS,2022. |     |               |         |        |                  |     |                |
| -------------------------------- | --- | --- | --------------- | --- | ------------- | ------- | ------ | ---------------- | --- | -------------- |
|                                  |     |     |                 |     | Warde-Farley, | Sherjil | Ozair, | Aaron Courville, | and | Yoshua Bengio. |
[59] Yinpeng Dong, Shuyu Cheng, Tianyu Pang, Hang Su, and Jun Zhu. Generativeadversarialnets. InNIPS,2014.
Query-efficientblack-boxadversarialattacksguidedbyatransfer-based [86] IanGoodfellow,JonathonShlens,andChristianSzegedy. Explaining
TPAMI,44(12):9536–9548,2022.
| prior. |     |     |     |     | andharnessingadversarialexamples. |     |     | InICLR,2015. |     |     |
| ------ | --- | --- | --- | --- | --------------------------------- | --- | --- | ------------ | --- | --- |
[60] YinpengDong,FangzhouLiao,TianyuPang,HangSu,JunZhu,Xiaolin [87] DouGoodman,HaoXin,WangYang,WuYuesheng,XiongJunfeng,
Hu,andJianguoLi. Boostingadversarialattackswithmomentum. In andZhangHuan. Advbox:atoolboxtogenerateadversarialexamples
| CVPR,2018. |     |     |     |     | thatfoolneuralnetworks,2020. |     |     |     |     |     |
| ---------- | --- | --- | --- | --- | ---------------------------- | --- | --- | --- | --- | --- |
[61] YinpengDong,TianyuPang,HangSu,andJunZhu. Evadingdefenses [88] Tianyu Gu, Kang Liu, Brendan Dolan-Gavitt, and Siddharth Garg.
totransferableadversarialexamplesbytranslation-invariantattacks. In Badnets:Evaluatingbackdooringattacksondeepneuralnetworks.IEEE
| CVPR,2019.   |            |             |             |                   | Access,7:47230–47244,2019. |     |     |     |     |     |
| ------------ | ---------- | ----------- | ----------- | ----------------- | -------------------------- | --- | --- | --- | --- | --- |
| [62] Yinpeng | Dong, Hang | Su, Baoyuan | Wu, Zhifeng | Li, Wei Liu, Tong |                            |     |     |     |     |     |
[89] ChuanGuo,JacobGardner,YurongYou,AndrewGordonWilson,and
Zhang, and Jun Zhu. Efficient decision-based black-box adversarial Kilian Weinberger. Simple black-box adversarial attacks. In ICML,
| attacksonfacerecognition. |     | InCVPR,2019. |     |     | 2019. |     |     |     |     |     |
| ------------------------- | --- | ------------ | --- | --- | ----- | --- | --- | --- | --- | --- |
[63] John R Douceur. The sybil attack. In International workshop on [90] Yiwen Guo, Qizhang Li, and Hao Chen. Backpropagating linearly
peer-to-peersystems.Springer,2002.
|     |     |     |     |     | improvestransferabilityofadversarialexamples. |     |     |     | InNeurIPS,2020. |     |
| --- | --- | --- | --- | --- | --------------------------------------------- | --- | --- | --- | --------------- | --- |
[64] Jiawei Du, Hu Zhang, Joey Tianyi Zhou, Yi Yang, and Jiashi Feng. [91] Yiwen Guo, Ziang Yan, and Changshui Zhang. Subspace attack:
Query-efficientmetaattacktodeepneuralnetworks. InICLR,2020. Exploitingpromisingsubspacesforquery-efficientblack-boxattacks.
| [65] WeiDu,YichunZhao,BoqunLi,GongshenLiu,andShilinWang. |     |     |     | Ppt: |     |     |     |     |     |     |
| -------------------------------------------------------- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
InNeurIPS,2019.
Backdoorattacksonpre-trainedmodelsviapoisonedprompttuning. [92] XingshuoHan,GuowenXu,YuanZhou,XuehuanYang,JiweiLi,and
InIJCAI,2022. TianweiZhang. Physicalbackdoorattackstolanedetectionsystemsin
[66] WeiDu,YichunZhao,BoqunLi,GongshenLiu,andShilinWang. Ppt: autonomousdriving. InACMMultimedia,2022.
Backdoorattacksonpre-trainedmodelsviapoisonedprompttuning.
|               |     |     |     |     | [93] Jonathan         | Hayase | and Sewoong  | Oh. Few-shot | backdoor | attacks via |
| ------------- | --- | --- | --- | --- | --------------------- | ------ | ------------ | ------------ | -------- | ----------- |
| InIJCAI,2022. |     |     |     |     | neuraltangentkernels. |        | InICLR,2022. |              |          |             |
[67] RanjieDuan,XingjunMa,YisenWang,JamesBailey,AKaiQin,and [94] SanghyunHong,NicholasCarlini,andAlexeyKurakin. Handcrafted
YunYang. Adversarialcamouflage:Hidingphysical-worldattackswith backdoorsindeepneuralnetworks. InNeurIPS,2022.
naturalstyles. InCVPR,2020. [95] Hongsheng Hu, Zoran Salcic, Gillian Dobbie, Jinjun Chen, Lichao
[68] Logan Engstrom, Brandon Tran, Dimitris Tsipras, Ludwig Schmidt, Sun,andXuyunZhang. Membershipinferenceviabackdooring. arXiv
andAleksanderMadry. Exploringthelandscapeofspatialrobustness. preprintarXiv:2206.04823,2022.
InICML,2019. [96] Kunzhe Huang, Yiming Li, Baoyuan Wu, Zhan Qin, and Kui Ren.
[69] Kevin Eykholt, Ivan Evtimov, Earlence Fernandes, Bo Li, Amir Backdoordefenseviadecouplingthetrainingprocess. InICLR,2022.
Rahmati,FlorianTramèr,AtulPrakash,TadayoshiKohno,andDawn [97] LifengHuang,ChengyingGao,YuyinZhou,CihangXie,AlanLYuille,
Song. Physicaladversarialexamplesforobjectdetectors. InUSENIX ChangqingZou,andNingLiu. Universalphysicalcamouflageattacks
| ConferenceonOffensiveTechnologies,2018. |     |     |     |     |     |     | InCVPR,2020. |     |     |     |
| --------------------------------------- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- |
onobjectdetectors.
[70] KevinEykholt,IvanEvtimov,EarlenceFernandes,BoLi,AmirRahmati, [98] QianHuang,IsayKatsman,HoraceHe,ZeqiGu,SergeBelongie,and
ChaoweiXiao,AtulPrakash,TadayoshiKohno,andDawnSong.Robust Ser-NamLim. Enhancingadversarialexampletransferabilitywithan
physical-worldattacksondeeplearningvisualclassification. InCVPR, intermediatelevelattack. InICCV,2019.
|     |     |     |     |     | [99] Zhichao | Huang and | Tong | Zhang. Black-box | adversarial | attack with |
| --- | --- | --- | --- | --- | ------------ | --------- | ---- | ---------------- | ----------- | ----------- |
2018.
[71] Yanbo Fan, Baoyuan Wu, Tuanhui Li, Yong Zhang, Mingyang Li, transferablemodel-basedembedding. InICLR,2020.
ZhifengLi,andYujiuYang. Sparseadversarialattackviaperturbation [100] AndrewIlyas,LoganEngstrom,AnishAthalye,andJessyLin. Black-
|                                       |              |     |                               |     | boxadversarialattackswithlimitedqueriesandinformation. |     |     |     |     | InICML, |
| ------------------------------------- | ------------ | --- | ----------------------------- | --- | ------------------------------------------------------ | --- | --- | --- | --- | ------- |
| factorization.                        | InECCV,2020. |     |                               |     |                                                        |     |     |     |     |         |
| [72] AlhusseinFawziandPascalFrossard. |              |     | Manitest:Areclassifiersreally |     | 2018.                                                  |     |     |     |     |         |
invariant? InBMVC,2015. [101] AndrewIlyas,LoganEngstrom,andAleksanderMadry. Priorconvic-
[73] Uriel Feige, Vahab S Mirrokni, and Jan Vondrák. Maximizing tions:Black-boxadversarialattackswithbanditsandpriors. InICLR,
| non-monotone | submodular | functions. | SIAM Journal | on Computing, | 2019. |     |     |     |     |     |
| ------------ | ---------- | ---------- | ------------ | ------------- | ----- | --- | --- | --- | --- | --- |
40(4):1133–1153,2011. [102] Nathan Inkawhich, Kevin Liang, Lawrence Carin, and Yiran Chen.
[74] Le Feng, Sheng Li, Zhenxing Qian, and Xinpeng Zhang. Stealthy Transferableperturbationsofdeepfeaturedistributions. InICLR,2020.
[103] NathanInkawhich,KevinLiang,BinghuiWang,MatthewInkawhich,
| backdoorattackwithadversarialtraining. |     |     | InICASSP,2022. |     |     |     |     |     |     |     |
| -------------------------------------- | --- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- |
[75] WeiweiFeng,BaoyuanWu,TianzhuZhang,YongZhang,andYongdong LawrenceCarin,andYiranChen.Perturbingacrossthefeaturehierarchy
Zhang. Meta-attack: Class-agnostic and model-agnostic physical to improve standard and strict blackbox attack transferability. In
| adversarialattack. | InICCV,2021. |     |     |     | NeurIPS,2020. |     |     |     |     |     |
| ------------------ | ------------ | --- | --- | --- | ------------- | --- | --- | --- | --- | --- |
[76] Yu Feng, Benteng Ma, Jing Zhang, Shanshan Zhao, Yong Xia, and [104] Steve TK Jan, Joseph Messou, Yen-Chen Lin, Jia-Bin Huang, and
Dacheng Tao. Fiba: Frequency-injection based backdoor attack in GangWang. Connectingthedigitalandphysicalworld:Improvingthe
medicalimageanalysis. InCVPR,2022. robustnessofadversarialattacks. InAAAI,volume33,2019.
[77] YanFeng,BaoyuanWu,YanboFan,LiLiu,ZhifengLi,andShutao [105] RishiDevJha,JonathanHayase,andSewoongOh. Labelpoisoningis
Xia. Boostingblack-boxattackwithpartiallytransferredconditional allyouneed. InNeurIPS,2023.
adversarialdistribution. InCVPR,2022. [106] XiaojunJia,YongZhang,BaoyuanWu,KeMa,JueWang,andXiaochun
[78] ClementFung,ChrisJMYoon,andIvanBeschastnikh. Thelimitations Cao. Las-at: Adversarial training with learnable attack strategy. In
| offederatedlearninginsybilsettings. |     |     | In23rdInternationalSymposium |     | CVPR,2022. |     |     |     |     |     |
| ----------------------------------- | --- | --- | ---------------------------- | --- | ---------- | --- | --- | --- | --- | --- |
onResearchinAttacks,IntrusionsandDefenses,2020. [107] Wenbo Jiang, Hongwei Li, Guowen Xu, and Tianwei Zhang. Color
[79] Kuofeng Gao, Jiawang Bai, Baoyuan Wu, Mengxi Ya, and Shu-Tao backdoor:Arobustpoisoningattackincolorspace. InCVPR,2023.
Xia. Imperceptibleandrobustbackdoorattackin3dpointcloud. arXiv [108] Seong Joon Oh, Mario Fritz, and Bernt Schiele. Adversarial image
|     |     |     |     |     | perturbation | for | privacy protection–a | game | theory | perspective. In |
| --- | --- | --- | --- | --- | ------------ | --- | -------------------- | ---- | ------ | --------------- |
preprintarXiv:2208.08052,2022.
[80] Yansong Gao, Bao Gia Doan, Zhi Zhang, Siqi Ma, Jiliang Zhang, ProceedingsoftheIEEEInternationalConferenceonComputerVision,
AnminFu,SuryaNepal,andHyoungshickKim. Backdoorattacksand pages1482–1491,2017.
countermeasures on deep learning: A comprehensive review. arXiv [109] CanKanbak,Seyed-MohsenMoosavi-Dezfooli,andPascalFrossard.

25
Geometricrobustnessofdeepnetworks:analysisandimprovement. In Shu-TaoXia. Black-boxdatasetownershipverificationviabackdoor
| CVPR,2018. |     |     |     |     | watermarking. | IEEETIFS,2023. |     |     |
| ---------- | --- | --- | --- | --- | ------------- | -------------- | --- | --- |
[110] DannyKarmon,DanielZoran,andYoavGoldberg. Lavan:Localized [137] ZiqiangLi,HongSun,PengfeiXia,BeihaoXia,XueRui,WeiZhang,
andvisibleadversarialnoise. InICML,2018. andBinLi.Aproxy-freestrategyforpracticallyimprovingthepoisoning
| [111] Jacob | Devlin Ming-Wei | Chang Kenton and | Lee Kristina | Toutanova. |                              |                                     |     |     |
| ----------- | --------------- | ---------------- | ------------ | ---------- | ---------------------------- | ----------------------------------- | --- | --- |
|             |                 |                  |              |            | efficiencyinbackdoorattacks. | arXivpreprintarXiv:2306.08313,2023. |     |     |
Bert: Pre-training of deep bidirectional transformers for language [138] ZiqiangLi,PengfeiXia,HongSun,YueqiZeng,WeiZhang,andBin
understanding. InProceedingsofNAACL-HLT,2019. Li.Exploretheeffectofdataselectiononpoisonefficiencyinbackdoor
[112] Yoongu Kim, Ross Daly, Jeremie Kim, Chris Fallin, Ji Hye Lee, attacks. arXivpreprintarXiv:2310.09744,2023.
DonghyukLee,ChrisWilkerson,KonradLai,andOnurMutlu.Flipping [139] SiyuanLiang,BaoyuanWu,YanboFan,XingxingWei,andXiaochun
bits in memory without accessing them: An experimental study of Cao. Parallel rectangle flip attack: A query-based black-box attack
dramdisturbanceerrors. ACMSIGARCHComputerArchitectureNews, againstobjectdetection. InICCV,2021.
42(3):361–372,2014. [140] Jiadong Lin, Chuanbiao Song, Kun He, Liwei Wang, and John E.
[113] StepanKomkovandAleksandrPetiushko. Advhat:Real-worldadver- Hopcroft. Nesterov accelerated gradient and scale invariance for
sarialattackonarcfacefaceidsystem. InICPR,2021. adversarialattacks. InICLR,2020.
[114] Jakub Konecˇny`, H Brendan McMahan, Felix X Yu, Peter Richtárik, [141] Junyu Lin, Lei Xu, Yingqi Liu, and Xiangyu Zhang. Composite
AnandaTheerthaSuresh,andDaveBacon.Federatedlearning:Strategies backdoor attack for deep neural network by mixing existing benign
forimprovingcommunicationefficiency. InNIPSWorkshoponPrivate features. InACMCCS,pages113–131,2020.
Multi-PartyMachineLearning,2016. [142] XiangLing,ShoulingJi,JiaxuZou,JiannanWang,ChunmingWu,Bo
[115] ZelunKong,JunfengGuo,AngLi,andCongLiu. Physgan:Generating Li,andTingWang. Deepsec:Auniformplatformforsecurityanalysis
|     |     |     |     |     |     | 2019 | IEEE Symposium | on Security and |
| --- | --- | --- | --- | --- | --- | ---- | -------------- | --------------- |
physical-world-resilientadversarialexamplesforautonomousdriving. of deep learning model. In
| InCVPR,2020. |     |     |     |     | Privacy(SP),pages673–690.IEEE,2019. |     |     |     |
| ------------ | --- | --- | --- | --- | ----------------------------------- | --- | --- | --- |
[116] Alexey Kurakin, Ian Goodfellow, and Samy Bengio. Adversarial [143] Aishan Liu, Xianglong Liu, Jiaxin Fan, Yuqing Ma, Anlan Zhang,
examples in the physical world. In Artificial intelligence safety and HuiyuanXie,andDachengTao. Perceptual-sensitiveganforgenerating
security,pages99–112.ChapmanandHall/CRC,2018. adversarialpatches. InAAAI,2019.
[117] CassidyLaidlawandSoheilFeizi. Functionaladversarialattacks. In [144] HongLiu,RongrongJi,JieLi,BaochangZhang,YueGao,Yongjian
NeurIPS,2019. Wu, and Feiyue Huang. Universal adversarial perturbation via prior
| [118] Debang | Li, Junge Zhang, | and Kaiqi Huang. | Universal | adversarial |                                 |     |              |     |
| ------------ | ---------------- | ---------------- | --------- | ----------- | ------------------------------- | --- | ------------ | --- |
|              |                  |                  |           |             | drivenuncertaintyapproximation. |     | InICCV,2019. |     |
perturbationsagainstobjectdetection. PatternRecognition,110:107584, [145] HuiLiu,BoZhao,JiabaoGuo,YangAn,andPengLiu. Greedyfool:
2021. Distortion-awaresparseadversarialattack. InNeurIPS,2020.
[119] HaoliangLi,YufeiWang,XiaofeiXie,YangLiu,ShiqiWang,Renjie [146] SijiaLiu,Pin-YuChen,XiangyiChen,andMingyiHong. signsgdvia
Wan, Lap-Pui Chau, and Alex C. Kot. Light can hack your face! zeroth-orderoracle. InICLR,2019.
black-boxbackdoorattackonfacerecognitionsystems. arXivpreprint [147] YiLiu,GeleiDeng,YuekangLi,KailongWang,TianweiZhang,Yepang
arXiv:2009.06996,2020. Liu,HaoyuWang,YanZheng,andYangLiu. Promptinjectionattack
[120] HuichenLi,XiaojunXu,XiaoluZhang,ShuangYang,andBoLi.Qeba: againstllm-integratedapplications. arXivpreprintarXiv:2306.05499,
| Query-efficientboundary-basedblackboxattack. |     |     | InCVPR,2020. |     | 2023. |     |     |     |
| -------------------------------------------- | --- | --- | ------------ | --- | ----- | --- | --- | --- |
[121] JieLi,RongrongJi,HongLiu,XiaopengHong,YueGao,andQiTian. [148] Yingqi Liu, Shiqing Ma, Yousra Aafer, Wen-Chuan Lee, Juan Zhai,
Universalperturbationattackagainstimageretrieval. InICCV,2019. Weihang Wang, and Xiangyu Zhang. Trojaning attack on neural
[122] JieLi,RongrongJi,HongLiu,JianzhuangLiu,BinengZhong,Cheng
|     |     |     |     |     | networks. | InNDSS,2018. |     |     |
| --- | --- | --- | --- | --- | --------- | ------------ | --- | --- |
Deng,andQiTian. Projection&probability-drivenblack-boxattack. [149] Yunfei Liu, Xingjun Ma, James Bailey, and Feng Lu. Reflection
InCVPR,2020. backdoor: A natural backdoor attack on deep neural networks. In
| [123] LinyangLi,DeminSong,XiaonanLi,JiehangZeng,RuotianMa,and |     |     |     |     | ECCV,2020. |     |     |     |
| ------------------------------------------------------------- | --- | --- | --- | --- | ---------- | --- | --- | --- |
Xipeng Qiu. Backdoor attacks on pre-trained models by layerwise [150] YujiaLiu,Seyed-MohsenMoosavi-Dezfooli,andPascalFrossard. A
weightpoisoning. InEMNLP,2021. geometry-inspireddecision-basedattack. InICCV,2019.
[124] Maosen Li, Cheng Deng, Tengjiao Li, Junchi Yan, Xinbo Gao, and [151] Yannan Liu, Lingxiao Wei, Bo Luo, and Qiang Xu. Fault injection
HengHuang. Towardstransferabletargetedattack. InCVPR,2020. attackondeepneuralnetwork. InICCAD,2017.
[125] Shaofeng Li, Hui Liu, Tian Dong, Benjamin Zi Hao Zhao, Minhui [152] Yang Liu, Zhihao Yi, and Tianjian Chen. Backdoor attacks and
Xue,HaojinZhu,andJialiangLu. Hiddenbackdoorsinhuman-centric defensesinfeature-partitionedcollaborativelearning. arXivpreprint
| languagemodels. | InACMCCS,2021. |     |     |     | arXiv:2007.03608,2020. |     |     |     |
| --------------- | -------------- | --- | --- | --- | ---------------------- | --- | --- | --- |
[126] ShaofengLi,MinhuiXue,BenjaminZhao,HaojinZhu,andXinpeng [153] Zechun Liu, Baoyuan Wu, Wenhan Luo, Xin Yang, Wei Liu, and
Zhang. Invisible backdoor attacks on deep neural networks via Kwang-TingCheng. Bi-realnet:Enhancingtheperformanceof1-bit
steganographyandregularization. TDSC,2021. cnnswithimprovedrepresentationalcapabilityandadvancedtraining
[127] TuanhuiLi,BaoyuanWu,YujiuYang,YanboFan,YongZhang,and algorithm. InECCV,2018.
Wei Liu. Compressing convolutional neural networks via factorized [154] YuyangLong,QilongZhang,BohengZeng,LianliGao,XianglongLiu,
convolutionalfilters. InCVPR,2019. JianZhang,andJingkuanSong.Frequencydomainmodelaugmentation
[128] Xiaoting Li, Lingwei Chen, and Dinghao Wu. Turning attacks into foradversarialattack. InECCV,2022.
protection:Socialmediaprivacyprotectionusingadversarialattacks. [155] GiulioLovisotto,HenryTurner,IvoSluganovic,MartinStrohmeier,and
InProceedingsofthe2021SIAMInternationalConferenceonData IvanMartinovic. Slap:improvingphysicaladversarialexampleswith
Mining(SDM),pages208–216.SIAM,2021. short-lived adversarial perturbations. In USENIX, pages 1865–1882,
| [129] XinkeLi,ZhiruiChen,YueZhao,ZekunTong,YabangZhao,Andrew |     |     |     |     | 2021. |     |     |     |
| ------------------------------------------------------------ | --- | --- | --- | --- | ----- | --- | --- | --- |
Lim,andJoeyTianyiZhou. Pointba:Towardsbackdoorattacksin3d [156] YouLuandBertHuang. Structuredoutputlearningwithconditional
| pointcloud. | InICCV,2021. |     |     |     | generativeflows. | InAAAI,2020. |     |     |
| ----------- | ------------ | --- | --- | --- | ---------------- | ------------ | --- | --- |
[130] YingweiLi,SongBai,YuyinZhou,CihangXie,ZhishuaiZhang,and [157] Chen Ma, Li Chen, and Jun-Hai Yong. Simulating unknown target
Alan Yuille. Learning transferable adversarial examples via ghost modelsforquery-efficientblack-boxattacks. InCVPR,2021.
networks. InAAAI,2020. [158] Aleksander Madry, Aleksandar Makelov, Ludwig Schmidt, Dimitris
[131] YimingLi,YangBai,YongJiang,YongYang,Shu-TaoXia,andBoLi. Tsipras,andAdrianVladu. Towardsdeeplearningmodelsresistantto
Untargetedbackdoorwatermark:Towardsharmlessandstealthydataset adversarialattacks. InICLR,2018.
|                      |     |               |     |     | [159] Apostolos | Modas, Seyed-Mohsen | Moosavi-Dezfooli, | and Pascal |
| -------------------- | --- | ------------- | --- | --- | --------------- | ------------------- | ----------------- | ---------- |
| copyrightprotection. |     | NeurIPS,2022. |     |     |                 |                     |                   |            |
[132] Yaxin Li, Wei Jin, Han Xu, and Jiliang Tang. Deeprobust: A Frossard. Sparsefool:Afewpixelsmakeabigdifference. InCVPR,
| pytorch | library for adversarial | attacks and | defenses. | arXiv preprint | 2019. |     |     |     |
| ------- | ----------------------- | ----------- | --------- | -------------- | ----- | --- | --- | --- |
arXiv:2005.06149,2020. [160] Hadi Mohaghegh Dolatabadi, Sarah Erfani, and Christopher Leckie.
[133] YandongLi,LijunLi,LiqiangWang,TongZhang,andBoqingGong. Advflow:inconspicuousblack-boxadversarialattacksusingnormalizing
Nattack: Learning the distributions of adversarial examples for an flows. InNeurIPS,2020.
improvedblack-boxattackondeepneuralnetworks. InICML,2019. [161] SeungyongMoon,GaonAn,andHyunOhSong. Parsimoniousblack-
[134] YuezunLi,YimingLi,BaoyuanWu,LongkangLi,RanHe,andSiwei box adversarial attacks via efficient combinatorial optimization. In
| Lyu. | Invisiblebackdoorattackwithsample-specifictriggers. |     |     | InICCV, | ICML,2019. |     |     |     |
| ---- | --------------------------------------------------- | --- | --- | ------- | ---------- | --- | --- | --- |
2021. [162] Seyed-MohsenMoosavi-Dezfooli,AlhusseinFawzi,OmarFawzi,and
[135] YimingLi,ZiqiZhang,JiawangBai,BaoyuanWu,YongJiang,andShu- PascalFrossard. Universaladversarialperturbations. InCVPR,2017.
[163] Seyed-MohsenMoosavi-Dezfooli,AlhusseinFawzi,andPascalFrossard.
| TaoXia. | Open-sourceddatasetprotectionviabackdoorwatermarking. |     |     |     |     |     |     |     |
| ------- | ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
InNeurIPSWorkshoponDatasetCurationandSecurity,2020. Deepfool:asimpleandaccuratemethodtofooldeepneuralnetworks.
[136] Yiming Li, Mingyan Zhu, Xue Yang, Yong Jiang, Tao Wei, and InCVPR,2016.

26
[164] KRMopuri,UGarg,andRVenkateshBabu. Fastfeaturefool:Adata automaticspeechrecognition. InICML,2019.
independentapproachtouniversaladversarialperturbations. InBMVC, [189] ZeyuQin,YanboFan,YiLiu,LiShen,YongZhang,JueWang,and
2017. BaoyuanWu. Boostingthetransferabilityofadversarialattackswith
[165] Konda Reddy Mopuri, Aditya Ganeshan, and R Venkatesh Babu. reverseadversarialperturbation. InNeurIPS,2022.
|               |     |           |           |     |          |           |             | [190] ZeyuQin,YanboFan,HongyuanZha,andBaoyuanWu.Randomnoise |     |     |     |     |     |     |     |
| ------------- | --- | --------- | --------- | --- | -------- | --------- | ----------- | ----------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| Generalizable |     | data-free | objective | for | crafting | universal | adversarial |                                                             |     |     |     |     |     |     |     |
perturbations. TPAMI,41(10):2452–2465,2018. defenseagainstquery-basedblack-boxattacks. InNeurIPS,2021.
[166] JohnMorris,EliLifland,JinYongYoo,JakeGrigsby,DiJin,andYanjun [191] AliRahmati,Seyed-MohsenMoosavi-Dezfooli,PascalFrossard,and
Qi. Textattack:Aframeworkforadversarialattacks,dataaugmentation, HuaiyuDai. Geoda:ageometricframeworkforblack-boxadversarial
|     |             |          |     |                |        |                 |     | attacks. | InCVPR,2020. |     |     |     |     |     |     |
| --- | ----------- | -------- | --- | -------------- | ------ | --------------- | --- | -------- | ------------ | --- | --- | --- | --- | --- | --- |
| and | adversarial | training | in  | nlp. In EMNLP: | System | Demonstrations, |     |          |              |     |     |     |     |     |     |
2020. [192] AdnanSirajRakin,ZhezhiHe,andDeliangFan. Tbt:targetedneural
[167] NinaNarodytskaandShivaPrasadKasiviswanathan. Simpleblack-box networkattackwithbittrojan. InCVPR,2020.
|                                          |     |     |     |     |                     |     |     | [193] AdnanSirajRakin,ZhezhiHe,JingtaoLi,FanYao,ChaitaliChakrabarti, |     |     |     |     |     |     |     |
| ---------------------------------------- | --- | --- | --- | --- | ------------------- | --- | --- | -------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| adversarialperturbationsfordeepnetworks. |     |     |     |     | CVPRWorkshops,2017. |     |     |                                                                      |     |     |     |     |     |     |     |
[168] Muzammal Naseer, Salman Khan, Munawar Hayat, Fahad Shahbaz and Deliang Fan. T-bfa: Targeted bit-flip adversarial weight attack.
Khan,andFatihPorikli. Ongeneratingtransferabletargetedperturba- TPAMI,44(11):7928–7939,2022.
tions. InICCV,2021. [194] Jonas Rauber, Wieland Brendel, and Matthias Bethge. Foolbox: A
pythontoolboxtobenchmarktherobustnessofmachinelearningmodels.
| [169] Muhammad |     | Muzammal | Naseer, | Salman | H Khan, | Muhammad | Haris |     |     |     |     |     |     |     |     |
| -------------- | --- | -------- | ------- | ------ | ------- | -------- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
Khan,FahadShahbazKhan,andFatihPorikli. Cross-domaintransfer- InICMLWorkshop,2017.
abilityofadversarialperturbations. InNeurIPS,2019. [195] Aniruddha Saha, Akshayvarun Subramanya, and Hamed Pirsiavash.
InAAAI,2020.
[170] PaarthNeekhara,ShehzeenHussain,PrakharPandey,ShlomoDubnov, Hiddentriggerbackdoorattacks.
|     |     |     |     |     |     |     |     | [196] Ahmed | Salem, | Rui Wen, | Michael | Backes, | Shiqing | Ma, and | Yang |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ------ | -------- | ------- | ------- | ------- | ------- | ---- |
JulianMcAuley,andFarinazKoushanfar.Universaladversarialperturba-
tionsforspeechrecognitionsystems. arXivpreprintarXiv:1905.03828, Zhang. Dynamicbackdoorattacksagainstmachinelearningmodels.
| 2019. |     |     |     |     |     |     |     | InEuroS&P,2022.                                                 |     |     |     |     |     |     |     |
| ----- | --- | --- | --- | --- | --- | --- | --- | --------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
|       |     |     |     |     |     |     |     | [197] EshaSarkar,HadjerBenkraouda,GopikaKrishnan,HomerGamil,and |     |     |     |     |     |     |     |
[171] YuriiNesterovandVladimirSpokoiny.Randomgradient-freeminimiza-
tionofconvexfunctions. FoundationsofComputationalMathematics, MichailManiatakos. Facehack:Attackingfacialrecognitionsystems
17(2):527–566,2017. usingmaliciousfacialcharacteristics. IEEET-BIOM,4:361–372,2022.
|                      |      |         |      |                                 |     |      |            | [198] Athena | Sayles,    | Ashish | Hooda, Mohit             | Gupta, | Rahul    | Chatterjee, | and |
| -------------------- | ---- | ------- | ---- | ------------------------------- | --- | ---- | ---------- | ------------ | ---------- | ------ | ------------------------ | ------ | -------- | ----------- | --- |
| [172] Dung           | Thuy | Nguyen, | Tuan | Minh Nguyen,                    | Anh | Tuan | Tran, Khoa | D            |            |        |                          |        |          |             |     |
|                      |      |         |      |                                 |     |      |            | Earlence     | Fernandes. |        | Invisible perturbations: |        | Physical | adversarial |     |
| Doan,andKOKSENGWONG. |      |         |      | Iba:Towardsirreversiblebackdoor |     |      |            |              |            |        |                          |        |          |             |     |
attacksinfederatedlearning. InNeurIPS,2023. examplesexploitingtherollingshuttereffect. InCVPR,2021.
[173] TuanAnhNguyenandAnhTran.Input-awaredynamicbackdoorattack. [199] LukasSchott, JonasRauber,Matthias Bethge,and Wieland Brendel.
Towardsthefirstadversariallyrobustneuralnetworkmodelonmnist.
InNeurIPS,volume33,2020.
| [174] TuanAnhNguyenandAnhTuanTran. |     |     |     |     | Wanet-imperceptiblewarping- |     |     | InICLR,2019. |     |     |     |     |     |     |     |
| ---------------------------------- | --- | --- | --- | --- | --------------------------- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
basedbackdoorattack. InICLR,2021. [200] AliShafahi,WRonnyHuang,MahyarNajibi,OctavianSuciu,Christoph
[175] Maria-Irina Nicolae, Mathieu Sinn, Minh Ngoc Tran, Beat Buesser, Studer, Tudor Dumitras, and Tom Goldstein. Poison frogs! targeted
|         |        |        |          |           |              |     |          | clean-labelpoisoningattacksonneuralnetworks. |     |     |     |     | InNeurIPS,2018. |     |     |
| ------- | ------ | ------ | -------- | --------- | ------------ | --- | -------- | -------------------------------------------- | --- | --- | --- | --- | --------------- | --- | --- |
| Ambrish | Rawat, | Martin | Wistuba, | Valentina | Zantedeschi, |     | Nathalie |                                              |     |     |     |     |                 |     |     |
Baracaldo, Bryant Chen, Heiko Ludwig, Ian M. Molloy, and Ben [201] MahmoodSharif,SrutiBhagavatula,LujoBauer,andMichaelKReiter.
Edwards. Adversarialrobustnesstoolboxv1.0.0,2019. Accessorize to a crime: Real and stealthy attacks on state-of-the-art
[176] RuiNing,JiangLi,ChunshengXin,andHongyiWu. Invisiblepoison: facerecognition. InACMCCS,2016.
|                                                        |     |     |     |     |     |     |     | [202] MahmoodSharif,SrutiBhagavatula,LujoBauer,andMichaelKReiter. |     |     |     |     |     |     |     |
| ------------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| Ablackboxcleanlabelbackdoorattacktodeepneuralnetworks. |     |     |     |     |     |     |     | In                                                                |     |     |     |     |     |     |     |
ICCC,2021. Ageneralframeworkforadversarialexampleswithobjectives. TOPS,
[177] RuiNing,JiangLi,ChunshengXin,HongyiWu,andChonggangWang. 22(3):1–30,2019.
|     |     |     |     |     |     |     |     | [203] LujiaShen,ShoulingJi,XuhongZhang,JinfengLi,JingChen,JieShi, |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
Hibernatedbackdoor:Amutualinformationempoweredbackdoorattack
|                       |     |     |              |     |     |     |     | ChengfangFang,JianweiYin,andTingWang. |     |     |     |     | Backdoorpre-trained |     |     |
| --------------------- | --- | --- | ------------ | --- | --- | --- | --- | ------------------------------------- | --- | --- | --- | --- | ------------------- | --- | --- |
| todeepneuralnetworks. |     |     | InAAAI,2022. |     |     |     |     |                                       |     |     |     |     |                     |     |     |
[178] Ren Pang, Zheng Zhang, Xiangshan Gao, Zhaohan Xi, Shouling Ji, modelscantransfertoall. InACMCCS,2021.
PengCheng,andTingWang. Trojanzoo:Towardsunified,holistic,and [204] Haoyue Shi, Jiayuan Mao, Tete Xiao, Yuning Jiang, and Jian Sun.
|                                       |     |     |     |     |                 |     |     | Learning |     | visually-grounded | semantics | from | contrastive | adversarial |     |
| ------------------------------------- | --- | --- | --- | --- | --------------- | --- | --- | -------- | --- | ----------------- | --------- | ---- | ----------- | ----------- | --- |
| practicalevaluationofneuralbackdoors. |     |     |     |     | InEuroS&P,2022. |     |     |          |     |                   |           |      |             |             |     |
[179] Nicolas Papernot, Fartash Faghri, Nicholas Carlini, Ian Goodfellow, samples. InCOLING,pages3715–3727,2018.
Reuben Feinman, Alexey Kurakin, Cihang Xie, Yash Sharma, Tom [205] JiawenShi,YixinLiu,PanZhou,andLichaoSun. Badgpt:Exploring
Brown, Aurko Roy, Alexander Matyasko, Vahid Behzadan, Karen securityvulnerabilitiesofchatgptviabackdoorattackstoinstructgpt.
arXivpreprintarXiv:2304.12298,2023.
Hambardzumyan,ZhishuaiZhang,Yi-LinJuang,ZhiLi,RyanSheatsley,
|     |     |     |     |     |     |     |     | [206] YuchengShi,YahongHan,QinghuaHu,YiYang,andQiTian. |     |     |     |     |     |     | Query- |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------ | --- | --- | --- | --- | --- | --- | ------ |
AbhibhavGarg,JonathanUesato,WilliGierke,YinpengDong,David
Berthelot,PaulHendricks,JonasRauber,andRujunLong. Technical efficient black-box adversarial attack with customized iteration and
TPAMI,45(2):2226–2245,2023.
| report | on the | cleverhans | v2.1.0 | adversarial | examples |     | library. arXiv | sampling.   |      |                    |      |          |      |          |     |
| ------ | ------ | ---------- | ------ | ----------- | -------- | --- | -------------- | ----------- | ---- | ------------------ | ---- | -------- | ---- | -------- | --- |
|        |        |            |        |             |          |     |                | [207] Yundi | Shi, | Piji Li, Changchun | Yin, | Zhaoyang | Han, | Lu Zhou, | and |
preprintarXiv:1610.00768,2018.
[180] Nicolas Papernot, Patrick McDaniel, Somesh Jha, Matt Fredrikson, ZheLiu. Promptattack:Prompt-basedattackforlanguagemodelsvia
Z Berkay Celik, and Ananthram Swami. The limitations of deep gradientsearch. InNLPCC,2022.
|          |     |             |           |             |       |          |       | [208] RezaShokrietal. |                 | Bypassingbackdoordetectionalgorithmsindeep |     |     |     |     |     |
| -------- | --- | ----------- | --------- | ----------- | ----- | -------- | ----- | --------------------- | --------------- | ------------------------------------------ | --- | --- | --- | --- | --- |
| learning | in  | adversarial | settings. | In EuroS&P, | pages | 372–387. | IEEE, |                       |                 |                                            |     |     |     |     |     |
|          |     |             |           |             |       |          |       | learning.             | InEuroS&P,2020. |                                            |     |     |     |     |     |
2016.
[181] FábioPerezandIanRibeiro.Ignorepreviousprompt:Attacktechniques [209] Ilia Shumailov, Zakhar Shumaylov, Dmitry Kazhdan, Yiren Zhao,
NicolasPapernot,MuratAErdogdu,andRossJAnderson.Manipulating
| forlanguagemodels. |     |     | InNeurIPSworkshop,2022. |     |     |     |     |                             |     |     |                 |     |     |     |     |
| ------------------ | --- | --- | ----------------------- | --- | --- | --- | --- | --------------------------- | --- | --- | --------------- | --- | --- | --- | --- |
|                    |     |     |                         |     |     |     |     | sgdwithdataorderingattacks. |     |     | InNeurIPS,2021. |     |     |     |     |
[182] HuyPhan,CongShi,YiXie,TianfangZhang,ZhuohangLi,Tianming
|       |      |          |       |          |           |     |              | [210] RyanSoklaski,JustinGoodwin,OliviaBrown,MichaelYee,andJason |     |     |     |     |     |     |     |
| ----- | ---- | -------- | ----- | -------- | --------- | --- | ------------ | ---------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| Zhao, | Jian | Liu, Yan | Wang, | Yingying | Chen, and | Bo  | Yuan. Ribac: |                                                                  |     |     |     |     |     |     |     |
Towardsrobustandimperceptiblebackdoorattackagainstcompact Matterer. Tools and practices for responsible ai engineering. arXiv
preprintarXiv:2201.05647,2022.
dnn. InECCV,2022.
|                                                     |     |     |     |     |     |     |           | [211] DavidMarcoSommer,LiweiSong,SameerWagh,andPrateekMittal. |     |     |     |     |     |     |     |
| --------------------------------------------------- | --- | --- | --- | --- | --- | --- | --------- | ------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| [183] HuyPhan,YiXie,JianLiu,YingyingChen,andBoYuan. |     |     |     |     |     |     | Invisible |                                                               |     |     |     |     |     |     |     |
Towardsprobabilisticverificationofmachineunlearning.arXivpreprint
andefficientbackdoorattacksforcompresseddeepneuralnetworks.
| InICASSP,2022.                                                  |                                                        |         |          |              |        |           |           | arXiv:2003.04247,2020.                               |        |                                                |            |                 |       |              |     |
| --------------------------------------------------------------- | ------------------------------------------------------ | ------- | -------- | ------------ | ------ | --------- | --------- | ---------------------------------------------------- | ------ | ---------------------------------------------- | ---------- | --------------- | ----- | ------------ | --- |
|                                                                 |                                                        |         |          |              |        |           |           | [212] YangSong,RuiShu,NateKushman,andStefanoErmon.   |        |                                                |            |                 |       | Constructing |     |
| [184] Omid                                                      | Poursaeed,                                             | Isay    | Katsman, | Bicheng      | Gao,   | and Serge | Belongie. |                                                      |        |                                                |            |                 |       |              |     |
|                                                                 |                                                        |         |          |              |        |           |           | unrestrictedadversarialexampleswithgenerativemodels. |        |                                                |            |                 |       | InNeurIPS,   |     |
| Generativeadversarialperturbations.                             |                                                        |         |          | InCVPR,2018. |        |           |           |                                                      |        |                                                |            |                 |       |              |     |
| [185] XiangyuQi,TinghaoXie,YimingLi,SaeedMahloujifar,andPrateek |                                                        |         |          |              |        |           |           | 2018.                                                |        |                                                |            |                 |       |              |     |
|                                                                 |                                                        |         |          |              |        |           |           | [213] Hossein                                        | Souri, | Liam                                           | Fowl, Rama | Chellappa,      | Micah | Goldblum,    | and |
| Mittal.                                                         | Revisitingtheassumptionoflatentseparabilityforbackdoor |         |          |              |        |           |           |                                                      |        |                                                |            |                 |       |              |     |
|                                                                 |                                                        |         |          |              |        |           |           | TomGoldstein.                                        |        | Sleeperagent:Scalablehiddentriggerbackdoorsfor |            |                 |       |              |     |
| defenses.                                                       | InICLR,2022.                                           |         |          |              |        |           |           |                                                      |        |                                                |            |                 |       |              |     |
|                                                                 |                                                        |         |          |              |        |           |           | neuralnetworkstrainedfromscratch.                    |        |                                                |            | InNeurIPS,2022. |       |              |     |
| [186] Xiangyu                                                   | Qi,                                                    | Tinghao | Xie,     | Ruizhe Pan,  | Jifeng | Zhu, Yong | Yang, and |                                                      |        |                                                |            |                 |       |              |     |
KaiBu. Towardspracticaldeployment-stagebackdoorattackondeep [214] Jacob Springer, Melanie Mitchell, and Garrett Kenyon. A little
|                 |     |                  |      |             |          |         |             | robustness       |           | goes a long     | way: Leveraging | robust | features | for       | targeted |
| --------------- | --- | ---------------- | ---- | ----------- | -------- | ------- | ----------- | ---------------- | --------- | --------------- | --------------- | ------ | -------- | --------- | -------- |
| neuralnetworks. |     | InCVPR,2022.     |      |             |          |         |             |                  |           |                 |                 |        |          |           |          |
|                 |     |                  |      |             |          |         |             | transferattacks. |           | InNeurIPS,2021. |                 |        |          |           |          |
| [187] Xiangyu   | Qi, | Jifeng           | Zhu, | Chulin Xie, | and Yong | Yang.   | Subnet      |                  |           |                 |                 |        |          |           |          |
|                 |     |                  |      |             |          |         |             | [215] Lukas      | Struppek, | Dominik         | Hintersdorf,    | and    | Kristian | Kersting. | Rick-    |
| replacement:    |     | Deployment-stage |      | backdoor    | attack   | against | deep neural |                  |           |                 |                 |        |          |           |          |
networksingray-boxsetting. InICLRworkshop,2021. rollingtheartist:Injectingbackdoorsintotextencodersfortext-to-image
|     |     |     |     |     |     |     |     | synthesis. |     | InICCV,2023. |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ------------ | --- | --- | --- | --- | --- |
[188] YaoQin,NicholasCarlini,GarrisonCottrell,IanGoodfellow,andColin
|         |                |     |         |              |             |     |              | [216] JiaweiSu,DaniloVasconcellosVargas,andKouichiSakurai. |     |     |     |           |     |     | Onepixel |
| ------- | -------------- | --- | ------- | ------------ | ----------- | --- | ------------ | ---------------------------------------------------------- | --- | --- | --- | --------- | --- | --- | -------- |
| Raffel. | Imperceptible, |     | robust, | and targeted | adversarial |     | examples for |                                                            |     |     |     |           |     |     |          |
|         |                |     |         |              |             |     |              | attackforfoolingdeepneuralnetworks.                        |     |     |     | TEC,2019. |     |     |          |

27
[217] ZhigangSu,DaweiZhou,NannanWang,DechengLiu,ZhenWang, learningsystemsinthephysicalworld. InCVPR,2021.
andXinboGao. Hidingvisualinformationviaobfuscatingadversarial [244] DaanWierstra,TomSchaul,TobiasGlasmachers,YiSun,JanPeters,
perturbations. InICCV,2023. andJürgenSchmidhuber.Naturalevolutionstrategies.JMLR,15(1):949–
| [218] FnuSuya,JianfengChi,DavidEvans,andYuanTian. |     |     |     |     |     | Hybridbatch | 980,2014.  |          |        |                  |              |          |        |
| ------------------------------------------------- | --- | --- | --- | --- | --- | ----------- | ---------- | -------- | ------ | ---------------- | ------------ | -------- | ------ |
|                                                   |     |     |     |     |     |             | [245] Eric | Wong and | J Zico | Kolter. Learning | perturbation | sets for | robust |
attacks:Findingblack-boxadversarialexampleswithlimitedqueries.
| InUSENIX,2020. |     |     |     |     |     |     | machinelearning. |     | InICLR,2020. |     |     |     |     |
| -------------- | --- | --- | --- | --- | --- | --- | ---------------- | --- | ------------ | --- | --- | --- | --- |
[219] EstebanGTabakandCristinaVTurner. Afamilyofnonparametric [246] EricWong,FrankSchmidt,andZicoKolter. Wassersteinadversarial
densityestimationalgorithms. CommunicationsonPureandApplied examplesviaprojectedsinkhorniterations. InICML,2019.
|     |     |     |     |     |     |     | [247] Baoyuan | Wu, | Hongrui | Chen, Mingda | Zhang, Zihao | Zhu, | Shaokui |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ------- | ------------ | ------------ | ---- | ------- |
Mathematics,66(2):145–164,2013.
[220] MatthewTancik,BenMildenhall,andRenNg. Stegastamp:Invisible Wei,DanniYuan,andChaoShen. Backdoorbench:Acomprehensive
hyperlinksinphysicalphotographs. InCVPR,2020. benchmarkofbackdoorlearning. InNeurIPSDatasetsandBenchmarks
| [221] Ruixiang | Tang, | Mengnan | Du, | Ninghao | Liu, Fan | Yang, and Xia Hu. | Track,2022. |     |     |     |     |     |     |
| -------------- | ----- | ------- | --- | ------- | -------- | ----------------- | ----------- | --- | --- | --- | --- | --- | --- |
An embarrassingly simple approach for trojan attack in deep neural [248] Baoyuan Wu and Bernard Ghanem. Lp-box admm: A versatile
networks. InKDD,2020. frameworkforintegerprogramming. TPAMI,41(7):1695–1708,2019.
[222] Yulong Tian, Fnu Suya, Fengyuan Xu, and David Evans. Stealthy [249] Baoyuan Wu, Shaokui Wei, Mingli Zhu, Meixi Zheng, Zihao Zhu,
MingdaZhang,HongruiChen,DanniYuan,LiLiu,andQingshanLiu.
| backdoorsascompressionartifacts. |     |     |     | IEEETIFS,2022. |     |     |     |     |     |     |     |     |     |
| -------------------------------- | --- | --- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
[223] MariyaToneva,AlessandroSordoni,RemiTachetdesCombes,Adam Defensesinadversarialmachinelearning:Asurvey,2023.
Trischler,YoshuaBengio,andGeoffreyJ.Gordon. Anempiricalstudy [250] Baoyuan Wu, Xuanchen Yan, andZeyu Qin. Blackboxbench. https:
ofexampleforgettingduringdeepneuralnetworklearning. InICLR, //github.com/SCLBD/BlackboxBench.
[251] DongxianWu,YisenWang,Shu-TaoXia,JamesBailey,andXingjun
2019.
[224] Alexander Turner, Dimitris Tsipras, and Aleksander Madry. Label- Ma. Skip connections matter: On the transferability of adversarial
consistentbackdoorattacks. arXivpreprintarXiv:1912.02771,2019. examplesgeneratedwithresnets. InICLR,2020.
[225] Tsinghua University, Alibaba Security, and RealAI. Adversarial [252] Lei Wu and Zhanxing Zhu. Towards understanding and improving
robustnessbenchmark. https://ml.cs.tsinghua.edu.cn/adv-bench. thetransferabilityofadversarialexamplesindeepneuralnetworks. In
| [226] AlexanderWan,EricWallace,ShengShen,andDanKlein. |     |     |     |     |              | Poisoning | ACML,2020.                                         |     |     |     |     |             |     |
| ----------------------------------------------------- | --- | --- | --- | --- | ------------ | --------- | -------------------------------------------------- | --- | --- | --- | --- | ----------- | --- |
|                                                       |     |     |     |     |              |           | [253] YutongWu,XingshuoHan,HanQiu,andTianweiZhang. |     |     |     |     | Computation |     |
| languagemodelsduringinstructiontuning.                |     |     |     |     | InICML,2023. |           |                                                    |     |     |     |     |             |     |
[227] Boxin Wang, Chejian Xu, Shuohang Wang, Zhe Gan, Yu Cheng, anddataefficientbackdoorattacks. InICCV,2023.
Jianfeng Gao, Ahmed Hassan Awadallah, and Bo Li. Adversarial [254] ZuxuanWu,Ser-NamLim,LarrySDavis,andTomGoldstein. Making
glue: A multi-task benchmark for robustness evaluation of language aninvisibilitycloak:Realworldadversarialattacksonobjectdetectors.
InECCV,2020.
| models. | InNeurIPS,2021. |     |     |     |     |     |     |     |     |     |     |     |     |
| ------- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
[228] BolunWang,YuanshunYao,ShawnShan,HuiyingLi,BimalViswanath, [255] PengfeiXia,ZiqiangLi,WeiZhang,andBinLi.Data-efficientbackdoor
Haitao Zheng, and Ben Y Zhao. Neural cleanse: Identifying and attacks. InIJCAI,2022.
[256] PengfeiXia,ZiqiangLi,WeiZhang,andBinLi.Data-efficientbackdoor
| mitigating |     | backdoor | attacks | in neural | networks. | In 2019 IEEE |          |               |     |     |     |     |     |
| ---------- | --- | -------- | ------- | --------- | --------- | ------------ | -------- | ------------- | --- | --- | --- | --- | --- |
|            |     |          |         |           |           |              | attacks. | InIJCAI,2022. |     |     |     |     |     |
SymposiumonSecurityandPrivacy(SP),pages707–723.IEEE,2019.
[229] HongyiWang,KartikSreenivasan,ShashankRajput,HaritVishwakarma, [257] ZhenXiang,DavidJMiller,SihengChen,XiLi,andGeorgeKesidis.
SaurabhAgarwal,Jy-yongSohn,KangwookLee,andDimitrisPapail- Abackdoorattackagainst3dpointcloudclassifiers. InICCV,2021.
|           |                                                    |     |     |     |     |     | [258] Chaowei | Xiao, | Bo Li, | Jun Yan Zhu, Warren | He, | Mingyan | Liu, and |
| --------- | -------------------------------------------------- | --- | --- | --- | --- | --- | ------------- | ----- | ------ | ------------------- | --- | ------- | -------- |
| iopoulos. | Attackofthetails:Yes,youreallycanbackdoorfederated |     |     |     |     |     |               |       |        |                     |     |         |          |
DawnSong.Generatingadversarialexampleswithadversarialnetworks.
| learning.     | InNeurIPS,2020.  |             |       |            |           |                      |               |       |           |                         |     |           |          |
| ------------- | ---------------- | ----------- | ----- | ---------- | --------- | -------------------- | ------------- | ----- | --------- | ----------------------- | --- | --------- | -------- |
| [230] Ruotong | Wang,            | Hongrui     | Chen, | Zihao      | Zhu, Li   | Liu, Yong Zhang,     | InIJCAI,2018. |       |           |                         |     |           |          |
|               |                  |             |       |            |           |                      | [259] Chaowei | Xiao, | Jun-Yan   | Zhu, Bo Li, Warren      | He, | Mingyan   | Liu, and |
| Yanbo         | Fan,             | and Baoyuan | Wu.   | Robust     | backdoor  | attack with visible, |               |       |           |                         |     |           |          |
|               |                  |             |       |            |           |                      | Dawn          | Song. | Spatially | transformed adversarial |     | examples. | In ICLR, |
| semantic,     | sample-specific, |             | and   | compatible | triggers. | arXiv preprint       |               |       |           |                         |     |           |          |
2018.
arXiv:2306.00816,2023.
[231] TongWang,YuanYao,FengXu,ShengweiAn,HanghangTong,and [260] ChulinXie,KeliHuang,Pin-YuChen,andBoLi. Dba:Distributed
|           |              |                                                    |     |     |     |     | backdoorattacksagainstfederatedlearning. |      |          |                    | InICLR,2019. |             |       |
| --------- | ------------ | -------------------------------------------------- | --- | --- | --- | --- | ---------------------------------------- | ---- | -------- | ------------------ | ------------ | ----------- | ----- |
| TingWang. |              | Aninvisibleblack-boxbackdoorattackthroughfrequency |     |     |     |     |                                          |      |          |                    |              |             |       |
|           |              |                                                    |     |     |     |     | [261] Cihang                             | Xie, | Zhishuai | Zhang, Yuyin Zhou, | Song         | Bai, Jianyu | Wang, |
| domain.   | InECCV,2022. |                                                    |     |     |     |     |                                          |      |          |                    |              |             |       |
[232] XiaosenWangandKunHe. Enhancingthetransferabilityofadversarial ZhouRen,andAlanLYuille. Improvingtransferabilityofadversarial
attacksthroughvariancetuning. InCVPR,2021. exampleswithinputdiversity. InCVPR,2019.
[262] YiXie,ZhuohangLi,CongShi,JianLiu,YingyingChen,andBoYuan.
| [233] XiaosenWang,KunHe,andJohnEHopcroft. |     |     |     |     | At-gan:Agenerative |     |     |     |     |     |     |     |     |
| ----------------------------------------- | --- | --- | --- | --- | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- |
Enablingfastanduniversalaudioadversarialattackusinggenerative
attackmodelforadversarialtransferringongenerativeadversarialnets.
arXivpreprintarXiv:1904.07793,3(4),2019. model. InAAAI,volume35,2021.
[263] YifengXiong,JiadongLin,MinZhang,JohnE.Hopcroft,andKunHe.
| [234] Xiaosen | Wang, | Xuanran | He, Jingdong |     | Wang, and | Kun He. Admix: |     |     |     |     |     |     |     |
| ------------- | ----- | ------- | ------------ | --- | --------- | -------------- | --- | --- | --- | --- | --- | --- | --- |
Stochasticvariancereducedensembleadversarialattackforboosting
| Enhancingthetransferabilityofadversarialattacks. |     |     |     |     |     | InICCV,2021. |     |     |     |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
[235] Xiaosen Wang, Jiadong Lin, Han Hu, Jingdong Wang, and Kun He. theadversarialtransferability. InCVPR,2022.
|                                                            |     |     |     |     |     |     | [264] Kaidi                    | Xu, Sijia | Liu, | Pu Zhao, Pin-Yu | Chen,                        | Huan Zhang, | Deniz |
| ---------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ------------------------------ | --------- | ---- | --------------- | ---------------------------- | ----------- | ----- |
| Boostingadversarialtransferabilitythroughenhancedmomentum. |     |     |     |     |     |     | In                             |           |      |                 |                              |             |       |
|                                                            |     |     |     |     |     |     | Erdogmus,YanzhiWang,andXueLin. |           |      |                 | Structuredadversarialattack: |             |       |
BMVC,2021.
[236] Xin Wang, Jie Ren, Shuyun Lin, Xiangming Zhu, Yisen Wang, and Towardsgeneralimplementationandbetterinterpretability. ICLR,2019.
Quanshi Zhang. A unified approach to interpreting and boosting [265] Kaidi Xu, Gaoyuan Zhang, Sijia Liu, Quanfu Fan, Mengshu Sun,
|                             |     |     |              |     |     |     | HonggeChen,Pin-YuChen,YanzhiWang,andXueLin.     |     |     |     |     | Adversarial  |     |
| --------------------------- | --- | --- | ------------ | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- | ------------ | --- |
| adversarialtransferability. |     |     | InICLR,2021. |     |     |     |                                                 |     |     |     |     |              |     |
|                             |     |     |              |     |     |     | t-shirt!evadingpersondetectorsinaphysicalworld. |     |     |     |     | InECCV,2020. |     |
[237] YulongWang,MinghuiZhao,ShenghongLi,XinYuan,andWeiNi.
Dispersedpixelperturbation-basedimperceptiblebackdoortriggerfor [266] LeiXu,YangyiChen,GanquCui,HongchengGao,andZhiyuanLiu.
imageclassifiermodels. TIFS,17:3091–3106,2022. Exploringtheuniversalvulnerabilityofprompt-basedlearningparadigm.
InNAACL,2022.
[238] ZhiboWang,HengchangGuo,ZhifeiZhang,WenxinLiu,ZhanQin,
|            |     |                                                        |     |     |     |     | [267] Xing | Xu, Jiefu | Chen, | Jinhui Xiao, Lianli | Gao, | Fumin Shen, | and |
| ---------- | --- | ------------------------------------------------------ | --- | --- | --- | --- | ---------- | --------- | ----- | ------------------- | ---- | ----------- | --- |
| andKuiRen. |     | Featureimportance-awaretransferableadversarialattacks. |     |     |     |     |            |           |       |                     |      |             |     |
InICCV,2021. HengTaoShen. Whatmachinesseeisnotwhattheyget:Foolingscene
[239] Zhenting Wang, Juan Zhai, and Shiqing Ma. Bppattack: Stealthy textrecognitionmodelswithadversarialtextimages. InCVPR,2020.
|                                                |           |        |                 |      |                 |           | [268] Xing   | Xu, Jiefu | Chen,                                              | Jinhui Xiao, Zheng | Wang, | Yang Yang, | and |
| ---------------------------------------------- | --------- | ------ | --------------- | ---- | --------------- | --------- | ------------ | --------- | -------------------------------------------------- | ------------------ | ----- | ---------- | --- |
| and                                            | efficient | trojan | attacks against | deep | neural networks | via image |              |           |                                                    |                    |       |            |     |
|                                                |           |        |                 |      |                 |           | HengTaoShen. |           | Learningoptimization-basedadversarialperturbations |                    |       |            |     |
| quantizationandcontrastiveadversariallearning. |           |        |                 |      | InCVPR,2022.    |           |              |           |                                                    |                    |       |            |     |
[240] Zhibo Wang, Siyan Zheng, Mengkai Song, Qian Wang, Alireza forattackingsequentialrecognitionmodels. InACMMultimedia,2020.
[269] XiaojunXu,XinyunChen,ChangLiu,AnnaRohrbach,TrevorDarrell,
| Rahimpour, |     | and Hairong | Qi. | advpattern: | Physical-world | attacks on |     |     |     |     |     |     |     |
| ---------- | --- | ----------- | --- | ----------- | -------------- | ---------- | --- | --- | --- | --- | --- | --- | --- |
andDawnSong.Foolingvisionandlanguagemodelsdespitelocalization
deeppersonre-identificationviaadversariallytransformablepatterns.
|     |     |     |     |     |     |     | andattentionmechanism. |     |     | InCVPR,2018. |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ---------------------- | --- | --- | ------------ | --- | --- | --- |
InICCV,2019.
[241] XingxingWei,JunZhu,ShaYuan,andHangSu. Sparseadversarial [270] YanXu,BaoyuanWu,FuminShen,YanboFan,YongZhang,HengTao
|                         |     |     |              |     |     |     | Shen,andWeiLiu.                              |     | Exactadversarialattacktoimagecaptioningvia |     |              |     |     |
| ----------------------- | --- | --- | ------------ | --- | --- | --- | -------------------------------------------- | --- | ------------------------------------------ | --- | ------------ | --- | --- |
| perturbationsforvideos. |     |     | InAAAI,2019. |     |     |     |                                              |     |                                            |     |              |     |     |
|                         |     |     |              |     |     |     | structuredoutputlearningwithlatentvariables. |     |                                            |     | InCVPR,2019. |     |     |
[242] YuxinWen,JonasGeiping,LiamFowl,HosseinSouri,RamaChellappa,
Micah Goldblum, and Tom Goldstein. Thinking two moves ahead: [271] JiaqiXue,MengxinZheng,TingHua,YilinShen,YepengLiu,Ladislau
Anticipatingotherusersimprovesbackdoorattacksinfederatedlearning. Boloni,andQianLou. Trojllm:Ablack-boxtrojanpromptattackon
|     |     |     |     |     |     |     | largelanguagemodels. |     |     | InNeurIPS,2023. |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------- | --- | --- | --------------- | --- | --- | --- |
arXivpreprintarXiv:2210.09305,2022.
[272] MingfuXue,CanHe,YinghaoWu,ShichangSun,YushuZhang,Jian
[243] EmilyWenger,JosephinePassananti,ArjunNitinBhagoji,Yuanshun
Yao,HaitaoZheng,andBenYZhao. Backdoorattacksagainstdeep Wang,andWeiqiangLiu. Ptb:Robustphysicalbackdoorattacksagainst
Computers&Security,2022.
deepneuralnetworksinrealworld.

28
[273] HiromuYakuraandJunSakuma. Robustaudioadversarialexample deepneuralnetworks. InACM/IEEEDAC,2019.
foraphysicalattack. InIJCAI,2019. [301] ShihaoZhao,XingjunMa,XiangZheng,JamesBailey,JingjingChen,
[274] ZhicongYan,GaoleiLi,YuanTIan,JunWu,ShenghongLi,Mingzhe andYu-GangJiang. Clean-labelbackdoorattacksonvideorecognition
Chen,andHVincentPoor. Dehib:Deephiddenbackdoorattackon models. InCVPR,2020.
|                                                    |     |     |              |     | [302] YueZhao,HongZhu,RuigangLiang,QintaoShen,ShengzhiZhang, |     |     |     |
| -------------------------------------------------- | --- | --- | ------------ | --- | ------------------------------------------------------------ | --- | --- | --- |
| semi-supervisedlearningviaadversarialperturbation. |     |     | InAAAI,2021. |     |                                                              |     |     |     |
[275] JianchengYang,YangzhouJiang,XiaoyangHuang,BingbingNi,and andKaiChen. Seeingisn’tbelieving:Towardsmorerobustadversarial
ChenglongZhao. Learningblack-boxattackerswithtransferablepriors attackagainstrealworldobjectdetectors. InACMCCS,2019.
andqueryfeedback. InNeurIPS,2020. [303] ZhendongZhao,XiaojunChen,YuexinXuan,YeDong,DakuiWang,
[276] Hongwei Yao, Jian Lou, and Zhan Qin. Poisonprompt: Backdoor and Kaitai Liang. Defeat: Deep hidden feature backdoor attacks by
attack on prompt-based large language models. arXiv preprint imperceptible perturbation and latent representation constraints. In
| arXiv:2310.12439,2023. |     |     |     |     | CVPR,2022. |     |     |     |
| ---------------------- | --- | --- | --- | --- | ---------- | --- | --- | --- |
[277] YuanshunYao, HuiyingLi,HaitaoZheng,andBen YZhao. Latent [304] Zhengyu Zhao, Zhuoran Liu, and Martha Larson. On success and
backdoorattacksondeepneuralnetworks. InACMCCS,2019. simplicity:Asecondlookattransferabletargetedattacks. InNeurIPS,
| [278] MaksymYatsura,JanMetzen,andMatthiasHein. |     |     | Meta-learningthe |     | 2021. |     |     |     |
| ---------------------------------------------- | --- | --- | ---------------- | --- | ----- | --- | --- | --- |
searchdistributionofblack-boxrandomsearchbasedadversarialattacks. [305] Xin Zheng, Yanbo Fan, Baoyuan Wu, Yong Zhang, Jue Wang, and
InNeurIPS,2021. ShiruiPan. Robustphysical-worldattacksonfacerecognition. Pattern
[279] FeiYin,YongZhang,BaoyuanWu,YanFeng,JingyiZhang,Yanbo Recognition,133:109009,2023.
Fan,andYujiuYang. Generalizableblack-boxadversarialattackwith [306] HaotiZhong,CongLiao,AnnaCinziaSquicciarini,SencunZhu,and
metalearning. TPAMI,2022. DavidMiller. Backdoorembeddinginconvolutionalneuralnetwork
[280] Chun-NamJohnYuandThorstenJoachims. Learningstructuralsvms InACMCODASPY,2020.
modelsviainvisibleperturbation.
withlatentvariables. InICML,2009. [307] Nan Zhong, Zhenxing Qian, and Xinpeng Zhang. Imperceptible
[281] PingYu,KaitaoSong,andJianfengLu.Generatingadversarialexamples backdoorattack:frominputspacetofeaturerepresentation. InIJCAI,
| withconditionalgenerativeadversarialnet. |     |     | InICPR,2018. |     |     |     |     |     |
| ---------------------------------------- | --- | --- | ------------ | --- | --- | --- | --- | --- |
2022.
[282] Yi Yu, Yufei Wang, Wenhan Yang, Shijian Lu, Yap-Peng Tan, and [308] HangZhou,DongdongChen,JingLiao,KejiangChen,XiaoyiDong,
Alex C Kot. Backdoor attacks against deep image compression via Kunlin Liu, Weiming Zhang, Gang Hua, and Nenghai Yu. Lg-gan:
adaptivefrequencytrigger. InCVPR,2023. Labelguidedadversarialnetworkforflexibletargetedattackofpoint
| [283] JianheYuanandZhihaiHe. |     | Consistency-sensitivityguidedensemble |     |     |                         |     |              |     |
| ---------------------------- | --- | ------------------------------------- | --- | --- | ----------------------- | --- | ------------ | --- |
|                              |     |                                       |     |     | cloudbaseddeepnetworks. |     | InCVPR,2020. |     |
black-box adversarial attacks in low-dimensional spaces. In ICCV, [309] LinjunZhou,PengCui,XingxuanZhang,YinanJiang,andShiqiang
2021. Yang. Adversarialeigenattackonblack-boxmodels. InCVPR,2022.
| [284] MingYuanandYiLin. |     | Modelselectionandestimationinregression |     |     |                                                            |     |     |     |
| ----------------------- | --- | --------------------------------------- | --- | --- | ---------------------------------------------------------- | --- | --- | --- |
|                         |     |                                         |     |     | [310] SichengZhu,RuiyiZhang,BangAn,GangWu,JoeBarrow,Zichao |     |     |     |
withgroupedvariables. JournaloftheRoyalStatisticalSociety:Series Wang,FurongHuang,AniNenkova,andTongSun.Autodan:Automatic
B,68(1):49–67,2006. andinterpretableadversarialattacksonlargelanguagemodels. arXiv
[285] ZhengYuan,JieZhang,YunpeiJia,ChuanqiTan,TaoXue,andShiguang preprintarXiv:2310.15140,2023.
| Shan. | Metagradientadversarialattack. |     | InICCV,2021. |     |                                                             |     |     |     |
| ----- | ------------------------------ | --- | ------------ | --- | ----------------------------------------------------------- | --- | --- | --- |
|       |                                |     |              |     | [311] YaoZhu,YuefengChen,XiaodanLi,KejiangChen,YuanHe,Xiang |     |     |     |
[286] GuoyangZeng,FanchaoQi,QianruiZhou,TingjiZhang,BairuHou, Tian, Bolun Zheng, Yaowu Chen, and Qingming Huang. Toward
Yuan Zang, Zhiyuan Liu, and Maosong Sun. Openattack: An open- understandingandboostingadversarialtransferabilityfromadistribution
sourcetextualadversarialattacktoolkit. InACLandIJCNLP:System perspective. TIP,31:6487–6501,2022.
Demonstrations,2021.
|     |     |     |     |     | [312] Yao | Zhu, Jiacheng | Sun, and Zhenguo | Li. Rethinking adversarial |
| --- | --- | --- | --- | --- | --------- | ------------- | ---------------- | -------------------------- |
[287] ChaoningZhang,PhilippBenz,ToobaImtiaz,andIn-SoKweon. Cd- transferabilityfromadatadistributionperspective. InICLR,2022.
uap:Classdiscriminativeuniversaladversarialperturbation. InAAAI, [313] Zihao Zhu, Mingda Zhang, Shaokui Wei, Li Shen, Yanbo Fan, and
2020. Baoyuan Wu. Boosting backdoor attack with a learnable poisoning
[288] HangfanZhang,JinyuanJia,JinghuiChen,LuLin,andDinghaoWu.
|                                                               |     |     |     |     | sampleselectionstrategy. |     | arXivpreprintarXiv:2307.07328,2023. |     |
| ------------------------------------------------------------- | --- | --- | --- | --- | ------------------------ | --- | ----------------------------------- | --- |
| A3fl:Adversariallyadaptivebackdoorattackstofederatedlearning. |     |     |     |     | In                       |     |                                     |     |
Thirty-seventhConferenceonNeuralInformationProcessingSystems,
2023.
[289] JieZhang,ChenDongdong,QidongHuang,JingLiao,WeimingZhang,
| HuaminFeng,                  | GangHua,                    | andNenghaiYu.                        | Poisonink:Robustand |     |     |     |     |     |
| ---------------------------- | --------------------------- | ------------------------------------ | ------------------- | --- | --- | --- | --- | --- |
| invisiblebackdoorattack.     |                             | TIP,31:5691–5705,2022.               |                     |     |     |     |     |     |
| [290] JiliangZhangandChenLi. |                             | Adversarialexamples:Opportunitiesand |                     |     |     |     |     |     |
| challenges.                  | TNNLS,31(7):2578–2593,2019. |                                      |                     |     |     |     |     |     |
[291] JiamingZhang,JitaoSang,XianZhao,XiaowenHuang,YanfengSun,
InProceedingsof
| andYongliHu. | Adversarialprivacy-preservingfilter. |     |     |     |     |     |     |     |
| ------------ | ------------------------------------ | --- | --- | --- | --- | --- | --- | --- |
the28thACMInternationalConferenceonMultimedia,pages1423–
1431,2020.
[292] JianpingZhang,WeibinWu,Jen-tseHuang,YizhanHuang,Wenxuan
| Wang,YuxinSu,andMichaelRLyu.              |     |     | Improvingadversarialtransfer- |     |     |     |     |     |
| ----------------------------------------- | --- | --- | ----------------------------- | --- | --- | --- | --- | --- |
| abilityvianeuronattribution-basedattacks. |     |     | InCVPR,2022.                  |     |     |     |     |     |
[293] QuanZhang,YifengDing,YongqiangTian,JianminGuo,MinYuan,
| andYuJiang.                                                    | Advdoor:Adversarialbackdoorattackofdeeplearning |                      |     |           |     |     |     |     |
| -------------------------------------------------------------- | ----------------------------------------------- | -------------------- | --- | --------- | --- | --- | --- | --- |
| system.                                                        | InISSTA,2021.                                   |                      |     |           |     |     |     |     |
| [294] XinyangZhang,ZhengZhang,ShoulingJi,andTingWang.          |                                                 |                      |     | Trojaning |     |     |     |     |
| languagemodelsforfunandprofit.                                 |                                                 | InEuroS&P.IEEE,2021. |     |           |     |     |     |     |
| [295] YanghaoZhang,WenjieRuan,FuWang,andXiaoweiHuang.          |                                                 |                      |     | Gener-    |     |     |     |     |
| alizinguniversaladversarialattacksbeyondadditiveperturbations. |                                                 |                      |     |           | In  |     |     |     |
ICDM,2020.
[296] ZhiyuanZhang,LingjuanLyu,WeiqiangWang,LichaoSun,andXu
Sun. Howtoinjectbackdoorswithbetterconsistency:Logitanchoring
| oncleandata.    | InICLR,2021. |                 |                      |       |     |     |     |     |
| --------------- | ------------ | --------------- | -------------------- | ----- | --- | --- | --- | --- |
| [297] Zhengming | Zhang,       | Ashwinee Panda, | Linyue Song, Yaoqing | Yang, |     |     |     |     |
MichaelMahoney,PrateekMittal,RamchandranKannan,andJoseph
| Gonzalez. | Neurotoxin: | durable backdoors | in federated | learning. | In  |     |     |     |
| --------- | ----------- | ----------------- | ------------ | --------- | --- | --- | --- | --- |
ICML,2022.
| [298] Guoping | Zhao, Mingyu                                               | Zhang, Jiajun    | Liu, Yaxian Li, | and Ji-Rong |     |     |     |     |
| ------------- | ---------------------------------------------------------- | ---------------- | --------------- | ----------- | --- | --- | --- | --- |
| Wen.          | Ap-gan:Adversarialpatchattackoncontent-basedimageretrieval |                  |                 |             |     |     |     |     |
| systems.      | GeoInformatica,pages1–31,2022.                             |                  |                 |             |     |     |     |     |
| [299] Pu      | Zhao, Sijia Liu,                                           | Yanzhi Wang, and | Xue Lin. An     | admm-based  |     |     |     |     |
universalframeworkforadversarialattacksondeepneuralnetworks.
InACMMultimedia,2018.
[300] PuZhao,SiyueWang,ChengGongye,YanzhiWang,YunsiFei,and
| XueLin. | Faultsneakingattack:Astealthyframeworkformisleading |     |     |     |     |     |     |     |
| ------- | --------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |

29
APPENDIX
COMPARISONSOFTHREEATTACKPARADIGMS
To further clarify the connections and differences of three
attack paradigms, we provide:
1) Differences on inputs, outputs and formulations of
three attack paradigms are summarized in Table III.
2) Asimpleillustrationofthreeattackparadigmsisshown
in Fig. 4.
Fig.4. AbriefgraphicalillustrationofthreeattackparadigmsofAML.(1)
Abinaryclassificationtask.(2)Backdoorattack:abackdooredmodelfwε (·)
istrainedbasedonthemanipulatedtrainingdataset.(3)Weightattack:locally
modifyingthedecisionboundaryofthebenignmodelfw0 (·)tochangethe
prediction of the target benign sample. (4) Adversarial example: a benign
sample is perturbed to across the decision boundary of the benign model
fw0 (·).
SUMMARYOFWEIGHTATTACKMETHODS
Compared to backdoor attacks and adversarial attacks,
existing weight attacks are insufficient to form a complex
taxonomy. Instead, we present a table to summarize all
mentioned weight attack methods, as shown in Table IV.
To help readers quickly explore adversarial phenomenon of
machine learning, we collect related resources of adversarial
machine learning, including several open-source toolboxes and
benchmarks, as shown in Table V.
ASSOCIATEDCATEGORIZATIONSOFEACHINDIVIDUAL
METHOD
Notethatinthetaxonomiespresentedinthemainmanuscript,
each individual method could belong to multiple categorizes
simultaneously.Tofacilitatethequickreviewofeachindividual
method,weprovidefourtablestosummarizetheassociatedcat-
egorizations for each method in different attack paradigms, as
showninTablesVI,VII,IV,IX,andX,respectively.Moreover,
weprovideawebsite,i.e.,http://adversarial-ml.com,wherethe
taxonomies and related literature are clearly presented. This
website will be well maintained and continuously updated to
cover more literature into the taxonomies.

30
TABLEIII
COMPARISONSAMONGTHREEATTACKPARADIGMSOFAML.
|                |     |              |                  |           |          |     |        | Stealthiness | Benignconsistency |        |     | Adversarialinconsistency |        |
| -------------- | --- | ------------ | ---------------- | --------- | -------- | --- | ------ | ------------ | ----------------- | ------ | --- | ------------------------ | ------ |
| Attackparadigm |     |              | Inputs           |           | Outputs  |     |        |              |                   |        |     |                          |        |
|                |     |              |                  |           |          |     | AML.Sx | AML.Sw       | AML.CA            | AML.CB |     | AML.IA                   | AML.IB |
|                |     | T ra i n i n | g da ta se t D 0 | orcontrol |          |     |        |              |                   |        |     |                          |        |
| Backdoorattack |     |              |                  |           | Dε orfwε | (·) |        |              |                   |        |     |                          |        |
|                |     | of t r a i n | ing p ro c es s  |           |          |     |        |              |                   |        |     |                          |        |
f ( ·)
| Weightattack       |     | w 0          | & a f e w b en       | i g n data& | fwε | (·) |     |     |     |     |     |     |     |
| ------------------ | --- | ------------ | -------------------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|                    |     | co n tr o lo | fd e v i ce m e      | m o ry      |     |     |     |     |     |     |     |     |     |
|                    |     | ( x , y      | , y ) & w e ig       | h ts or ac- |     |     |     |     |     |     |     |     |     |
| Adversarialexample |     | 0 0          | ε                    |             |     | xε  |     |     |     |     |     |     |     |
|                    |     | c es s p e   | r m i s sio no f f w | (·)         |     |     |     |     |     |     |     |     |     |
0
TABLEIV
ABRIEFSUMMARYOFEXISTINGDEPLOYMENT-TIMEADVERSARIALATTACK(i.e.,WEIGHTATTACK)METHODS.
| Category |                                     | Method |     |                                                  |              |               |           | Description/Specification |                   |       |          |           |      |
| -------- | ----------------------------------- | ------ | --- | ------------------------------------------------ | ------------ | ------------- | --------- | ------------------------- | ----------------- | ----- | -------- | --------- | ---- |
|          | ⋄Singlebiasattack(SBA)[151]         |        |     | ⋄Simplyenlargingthebiasparameterofthetargetclass |              |               |           |                           |                   |       |          |           |      |
|          | ⋄Gradientdescentattack(GDA)[151]    |        |     | ⋄argminwε                                        |              | Dw(w0,wε)+λIA |           |                           | LIA (fwε (xε),yε) |       |          |           |      |
|          |                                     |        |     |                                                  |              |               |           |                           |                   |       |          | (x( i)),y | (i)) |
|          |                                     |        |     | ⋄                                                |              | argmin        |           | Dw(w0,wε)                 |                   | + λCA | LCA (fwε | 0         | 0 +  |
|          | ⋄Targetedbit-flipattack(T-BFA)[193] |        |     |                                                  | wε∈{0,1}|wε| |               |           |                           |                   |       |          |           |      |
|          |                                     |        |     |                                                  |              | (j)),yε       | (j)),i̸=j |                           |                   |       |          |           |      |
|          |                                     |        |     | λIA                                              | LIA (fwε     | (xε           |           |                           |                   |       |          |           |      |
Weightattack
|                      |                                |     |     | ⋄ Almost |     | same with | T-BFA, | with | only one difference | that | there is | no binary | constraint |
| -------------------- | ------------------------------ | --- | --- | -------- | --- | --------- | ------ | ---- | ------------------- | ---- | -------- | --------- | ---------- |
| with o u ttr ig ger: | ⋄Faultsneakingattack(FSA)[300] |     |     |          |     |           |        |      |                     |      |          |           |            |
w . r. t . w ε
x ε = x 0
⋄ A l m o st samewithT-BFA,withtwomaindifferences:1)attackonesingledata,rather
|     | ⋄ Targeted   | attack with | limited bit-flips |                                 |          |             |        |                   |              |           |         |             |               |
| --- | ------------ | ----------- | ----------------- | ------------------------------- | -------- | ----------- | ------ | ----------------- | ------------ | --------- | ------- | ----------- | ------------- |
|     |              |             |                   | than                            | all data | of one      | class; | 2) an efficient   | optimization | algorithm | for     | binary      | optimization, |
|     | (TA-LBF)[12] |             |                   | ratherthantheheuristicalgorithm |          |             |        |                   |              |           |         |             |               |
|     |              |             |                   | ⋄ Changing                      |          | the weights | of     | the adversarially | trained      | model     | via bit | flipping to | reduce the    |
⋄Robustnessattack[83]
model’srobustness
|     |     |     |     | ⋄ There | are        | three steps: | 1)            | Selecting | a few neurons     | (i.e.,  | weights) | that contribute | more       |
| --- | --- | --- | --- | ------- | ---------- | ------------ | ------------- | --------- | ----------------- | ------- | -------- | --------------- | ---------- |
|     |     |     |     | to      | the target | class;       | 2) generating |           | an input-agnostic | trigger | xε       | − x0 by         | maximizing |
⋄TargetedbitTrojan(TBT)attack[192] the activation of the selected neurons; 3) argminwε λCA LCA (fwε (x( i)),y (i)) +
0 0
|     |     |     |     |     |          | (j)),yε | (j)),i̸=j |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | -------- | ------- | --------- | --- | --- | --- | --- | --- | --- |
|     |     |     |     | λIA | LIA (fwε | (xε     |           |     |     |     |     |     |     |
⋄Adoptingthesame3-stepprocedurewithTBT,withdifferentalgorithmforeachindividual
⋄ProFlipattack[30]
stage
|              |                       |     |     | ⋄ A | l m o s t s  | a m e w it h  | T BT,withthemaindifferencethatthebenignsamplex0 |     |     |     |     |     | isobtained |
| ------------ | --------------------- | --- | --- | --- | ------------ | ------------- | ----------------------------------------------- | --- | --- | --- | --- | --- | ---------- |
| Weightattack | ⋄Trojaningattack[148] |     |     |     |              |               |                                                 |     |     |     |     |     |            |
|              |                       |     |     | by  | r e ve r s e | e n g in ee r | in g                                            |     |     |     |     |     |            |
withtrigger:
|     |     |     |     | ⋄   | Backdoor |     | injection |     | via | slight | weight |     | perturbation: |
| --- | --- | --- | --- | --- | -------- | --- | --------- | --- | --- | ------ | ------ | --- | ------------- |
xε̸=x0 ⋄ A dversarialWeightPerturbation(AWP) (x( i)),yε (i))+R3(fwε (x( i)),fw0 (x( i)))
|     |       |     |     | argminwε |          | λIA LIA | (fwε      | ε   |            |        |        |            |               |
| --- | ----- | --- | --- | -------- | -------- | ------- | --------- | --- | ---------- | ------ | ------ | ---------- | ------------- |
|     | [81 ] |     |     |          |          |         |           |     |            | 0      | 0      |            |               |
|     |       |     |     | ⋄        | Backdoor |         | injection |     | via        | slight | weight |            | perturbation: |
|     |       |     |     |          |          |         |           | (x( | i)),y (i)) |        |        | (x( i)),yε | (i))          |
⋄Anchoringattack[296] argminwε λCA LCA (fwε + λIA LIA (fwε ε +
|     |     |     |     |             | (x( | i)),gw0    | (x( i))),whereg |            | 0 0             |              |        |             |         |
| --- | --- | --- | --- | ----------- | --- | ---------- | --------------- | ---------- | --------------- | ------------ | ------ | ----------- | ------- |
|     |     |     |     | R3(gwε      |     |            |                 |            | denotesthelogit |              |        |             |         |
|     |     |     |     |             |     | 0          | 0               |            |                 |              |        |             |         |
|     |     |     |     | ⋄ Replacing |     | one subnet | in              | the benign | model by        | the backdoor | subnet | and cutting | off the |
⋄Subnetreplacementattack(SRA)[186]
connectiontoremainingpartofthemodel
|     |     |     |     | ⋄ExtensionofTA-LBFbyintroducingatriggerxε−x0,andoptimizingwε |     |     |     |     |     |     |     |     | andtrigger |
| --- | --- | --- | --- | ------------------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | ---------- |
⋄Triggeredsamplesattack(TSA)[11] xε−x0 jointlybysolvingamixedintegerprogrammingproblem,suchthatthemodified
modelwillbeactivatedbythetrigger
TABLEV
RELATEDOPEN-SOURCETOOLBOXESANDBENCHMARKSINADVERSARIALMACHINELEARNING.
|     |     | Year | Backdoorlearning | Adversarialexample |     | Toolbox |     | Benchmark | Link |     |     |     |     |
| --- | --- | ---- | ---------------- | ------------------ | --- | ------- | --- | --------- | ---- | --- | --- | --- | --- |
BackdoorBench[247] 2022 ✓ ✓ ✓ https://github.com/SCLBD/backdoorbench
BlackboxBench[250] 2022 ✓ ✓ ✓ https://github.com/SCLBD/BlackboxBench
OpenBackdoor[52] 2022 ✓ ✓ ✓ https://github.com/thunlp/OpenBackdoor
|     |     |     |     |     | ✓   |     | ✓   |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ResponsibleAIToolbox[210] 2022 https://github.com/mit-ll-responsible-ai/responsible-ai-toolbox
AdversarialGLUE[227] 2021 ✓ ✓ https://github.com/AI-secure/adversarial-glue
DeepRobust[132] 2021 ✓ ✓ https://github.com/DSE-MSU/DeepRobust
OpenAttack[286] 2021 ✓ ✓ https://github.com/thunlp/OpenAttack
|     |     |     |     |     | ✓   |     | ✓   | ✓   |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
AdversarialRobustnessBenchmark[225] 2021 https://ml.cs.tsinghua.edu.cn/adv-bench
RobustBench[48] 2021 ✓ ✓ ✓ https://github.com/RobustBench/robustbench
| TextAttack[166] |     | 2020 |     |     | ✓   |     | ✓   |     | https://github.com/QData/TextAttack |     |     |     |     |
| --------------- | --- | ---- | --- | --- | --- | --- | --- | --- | ----------------------------------- | --- | --- | --- | --- |
TrojanZoo[178] 2020 ✓ ✓ https://github.com/ain-soph/trojanzoo
|                |     |      |     |     | ✓   |     | ✓   |     |                                      |     |     |     |     |
| -------------- | --- | ---- | --- | --- | --- | --- | --- | --- | ------------------------------------ | --- | --- | --- | --- |
| AutoAttack[51] |     | 2020 |     |     |     |     |     |     | https://github.com/fra31/auto-attack |     |     |     |     |
|                |     |      |     |     | ✓   |     | ✓   |     |                                      |     |     |     |     |
| Advbox[87]     |     | 2020 |     |     |     |     |     |     | https://github.com/advboxes/AdvBox   |     |     |     |     |
AdverTorch[54] 2019 ✓ ✓ https://github.com/BorealisAI/advertorch
| DEEPSEC[142] |     | 2019 |     |     | ✓   |     | ✓   |     | https://github.com/ryderling/DEEPSEC |     |     |     |     |
| ------------ | --- | ---- | --- | --- | --- | --- | --- | --- | ------------------------------------ | --- | --- | --- | --- |
CleverHans[179] 2018 ✓ ✓ https://github.com/cleverhans-lab/cleverhans
|     |     |     |     |     | ✓   |     | ✓   |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
AdversarialRobustnessToolbox[175] 2018 https://github.com/Trusted-AI/adversarial-robustness-toolbox
| Foolbox[194] |     | 2017 |     |     | ✓   |     | ✓   |     | https://github.com/bethgelab/foolbox |     |     |     |     |
| ------------ | --- | ---- | --- | --- | --- | --- | --- | --- | ------------------------------------ | --- | --- | --- | --- |

31
TABLEVI
CATEGORIZATIONOFEXISTINGDATA-POISONINGBASEDBACKDOORATTACKMETHODS.FOREACHCATEGORIZATIONCRITERION“A/B”, DENOTESTHE
|     |     | FORMER“A”, | DENOTESTHELATTER“B”AND |     | REPRESENTSBOTH.  |     | (cid:35) |
| --- | --- | ---------- | ---------------------- | --- | ---------------- | --- | -------- |
|     |     |            | (cid:32)               |     | (cid:72)(cid:35) |     |          |
Data-poisoningbasedbackdoorattack
Visible/ Non- Manually Digital/ Additive/ Static/ Sample- Label- Single/
| Method | Venue | semantic/ | designed/ |          | Non-     | agnostic/ inconsistent |              |
| ------ | ----- | --------- | --------- | -------- | -------- | ---------------------- | ------------ |
|        |       | Invisible |           | Physical | Dynamic  |                        | Multi-target |
|        |       | Semantic  | Leanable  |          | additive | specific /consistent   |              |
IEEE Access
BadNets[88]
2019
(cid:35) (cid:35) (cid:35) (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)(cid:72)
| Blended[41] | arXiv2017 |     |     |     |     |     |     |
| ----------- | --------- | --- | --- | --- | --- | --- | --- |
TrojanNN[148] NDSS2018 (cid:32) (cid:35) (cid:35) (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
Shafahi et al. (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
NeurIPS2018
[200]
(cid:32) (cid:35) (cid:32) (cid:35) (cid:32) (cid:32) (cid:32) (cid:32) (cid:35)
| SIG[14] | ICIP2019 |     |     |     |     |     |     |
| ------- | -------- | --- | --- | --- | --- | --- | --- |
LC[224] arXiv2019 (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35)
Yaoetal.[277] CCS2019 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35)
Saha et al. (cid:35) (cid:35) (cid:32) (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
AAAI2020
[195]
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35)
CODASPY
| Static[306] | 2020 |     |     |     |     |     |     |
| ----------- | ---- | --- | --- | --- | --- | --- | --- |
(cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
| Adaptive[306] | CODASPY |     |     |     |     |     |     |
| ------------- | ------- | --- | --- | --- | --- | --- | --- |
2020
Zhao et al. (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35)
CVPR2020
[301]
(cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35)
| Refool[149] | ECCV2020 |     |     |     |     |     |     |
| ----------- | -------- | --- | --- | --- | --- | --- | --- |
Lietal.[119] arXiv2020 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35)
DeHiB[274] AAAI2021 (cid:32) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
Wenger et al. (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35)
CVPR2021
[243]
(cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
ICLR
| Lietal.[125] | Workshop |     |     |     |     |     |     |
| ------------ | -------- | --- | --- | --- | --- | --- | --- |
2021
(cid:32) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35)
| Steganography | IEEE TDSC |     |     |     |     |     |     |
| ------------- | --------- | --- | --- | --- | --- | --- | --- |
| [126]         | 2021      |     |     |     |     |     |     |
(cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35)
| Regularization | IEEE TDSC |     |     |     |     |     |     |
| -------------- | --------- | --- | --- | --- | --- | --- | --- |
| [126]          | 2021      |     |     |     |     |     |     |
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
| InvisiblePoison | INFOCOM |     |     |     |     |     |     |
| --------------- | ------- | --- | --- | --- | --- | --- | --- |
| [176]           | 2021    |     |     |     |     |     |     |
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35)
IEEE JSAC
| ROBNET[84] | 2021 |     |     |     |     |     |     |
| ---------- | ---- | --- | --- | --- | --- | --- | --- |
(cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:72)(cid:35)
| AdvDoor[293] | ISSTA2021 |     |     |     |     |     |     |
| ------------ | --------- | --- | --- | --- | --- | --- | --- |
PCBA[257] ICCV2021 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35)
PointPBA[129] ICCV2021 (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
PointCPB[129] ICCV2021 (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
(cid:72)(cid:35) (cid:35) (cid:32) (cid:35) (cid:72)(cid:35) (cid:32) (cid:32) (cid:32) (cid:35)
| SSBA[134] | ICCV2021 |     |     |     |     |     |     |
| --------- | -------- | --- | --- | --- | --- | --- | --- |
Phan[183] ICASSP2022 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)1 (cid:32) (cid:32) (cid:35) (cid:35)
Random Back- Euro S&P (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
| door[196] | 2022 |     |     |     |     |     |     |
| --------- | ---- | --- | --- | --- | --- | --- | --- |
(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:72)(cid:35)
| FTrojan[231] | ECCV2022 |     |     |     |     |     |     |
| ------------ | -------- | --- | --- | --- | --- | --- | --- |
Sleeper Agent (cid:32) (cid:35) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35)
NeruIPS2022
[213]
(cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35)
| PTB[272] | C&S2022 |     |     |     |     |     |     |
| -------- | ------- | --- | --- | --- | --- | --- | --- |
IEEE TBBIS (cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
FaceHack[197]
2022
(cid:32) (cid:35) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35)
| Hanetal.[92] | MM2022 |     |     |     |     |     |     |
| ------------ | ------ | --- | --- | --- | --- | --- | --- |
IRBA[79] arXiv2022 (cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)(cid:72) (cid:35)
Wang et al. (cid:32) (cid:35) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35)
IEEETIFS
[237]
(cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
Adap-Blend
ICLR2023
[185]
(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35)
| Yuetal.[282] | CVPR2023 |     |     |     |     |     |     |
| ------------ | -------- | --- | --- | --- | --- | --- | --- |
ColorBackdoor (cid:32) (cid:35) (cid:32) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35)
CVPR2023
[107]
(cid:32) (cid:35) (cid:32) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35)
| VSSC[230] | arXiv2023 |     |     |     |     |     |     |
| --------- | --------- | --- | --- | --- | --- | --- | --- |
FLIP[105] NeurIPS2023 (cid:35) (cid:32) (cid:32) (cid:72)(cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35)(cid:72)
(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)

32
TABLEVII
CATEGORIZATIONOFEXISTINGTRAINING-CONTROLLABLEBASEDBACKDOORATTACKMETHODS.FOREACHCATEGORIZATIONCRITERION“A/B”,
DENOTESTHEFORMER“A”, DENOTESTHELATTER“B”AND REPRESENTSBOTH.FORCRITERION“A/B/C”, , , , DENOTE“A”,“B”,“C”,
(cid:35) “D”RESPECTIVELY.FORTH(cid:32)EMETHODSOFPARTIALLYCONTRO(cid:72)(cid:35)LLINGTHETRAININGDATAANDPROCESS,WEOM(cid:35)IT(cid:32)TRI(cid:71)GG(cid:72)ERSINCETHEREISNO
DISTINGUISHABLETRIGGERLISTEDINTHETABLE.
Data-poisoningbasedbackdoorattack Training-controllablebasedbackdoorattack
Method Venue I V n i v s i i s b i l b e le / s S e e m N m a o a n n n ti - t c ic / d M L e e s a a i n g n u n a a e b l d l l e y / P D h ig y i s t i a c l al / A a d d N d d i o i t t i n i v v - e e / D S y t n at a i m c i / c a S s g p a n e m o c s p i t fi i l c e c - / / in c c o L o n a n s b s i i e s s l t t - e e n n t t M S ul i t n i g -t l a e rg / et Tw O o n -s e ta / ge F tr u a a l i c l n c i / e n s P g s a d r o t a f i t a a l Fu co t l p r l n r a o / t i r n c P o e i a l n s r g s o ti f al tr C a a l i o g n n o o in t r r r d g i o t e h l l r l m o in s g s / /
B [1 h 5 a ] gojietal. ICML2019
(cid:35) (cid:32)
Bagdasaryan AISTATS
etal.[10] 2020
(cid:35) (cid:32)
W [22 a 9 n ] g et al. NeruIPS2020
(cid:35) (cid:32)
[ F 7 u 8 n ] g et al. RAID2020
(cid:35) (cid:32)
[ C 2 h 9 e ] n et al. arXiv2020
(cid:35) (cid:32)
[ L 1 i 5 u 2] et al. arXiv2020
(cid:35) (cid:32)
C ta o c m k p [1 o 4 si 1 t ] eAt- CCS2020
(cid:35) (cid:32) (cid:35) (cid:72)(cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
Tan et al. Euro S&P
[208] 2020
(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35)
DBA[260] ICLR2020
(cid:35) (cid:32)
T [2 r 2 o 1 ja ] nNet KDD2020
(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:72)(cid:35) (cid:32) (cid:35) (cid:32) (cid:32)
N [1 g 7 u 3 y ] enetal. NeruIPS2020
(cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
DFST[45] AAAI2021
(cid:32) (cid:35) (cid:32) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
WaNet[174] ICLR2021
(cid:32) (cid:35) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32)
WB[56] NeruIPS2021
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
BOB[209] NeurIPS2021
(cid:35) (cid:35) (cid:35) (cid:71)
LIRA[57] ICCV2021
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
LWP[123] EMNLP2021
S [2 h 0 e 3 n ] et al. CCS2021 (cid:35) (cid:32) (cid:32)
(cid:35) (cid:32) (cid:32)
HB[177] AAAI2022
(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32)
D [3 E 03 F ] EAT AAAI2022
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
BaN[196] E 20 u 2 ro 2 S&P
(cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
c-BaN[196] E 20 u 2 ro 2 S&P
(cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
RIBAC[182] ECCV2022
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
Feng[74] ICASSP2022
[ Z 3 h 0 o 7 n ] getal. IJCAI2022 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
W [24 e 2 n ] et al. arXiv2022
(cid:35) (cid:32)
B [2 P 3 P 9 A ] TTACK CVPR2022
(cid:35) (cid:35) (cid:35) (cid:35)
FIBA[76] CVPR2022
(cid:32) (cid:35) (cid:35) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32)
M [58 a ] rksman NeurIPS2022
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:72)(cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
Poison Ink IEEE TIP
[289] 2022
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
NTBA[93] ICLR2023
(cid:32) (cid:35) (cid:32) (cid:35) (cid:32) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
EfficFrog[35] CVPR2023
(cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32)
MAB[17] CVPR2023
(cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32)
IBA[172] NeurIPS2023
(cid:32)
A3FL[288] NeruIPS2023
(cid:32)

33
TABLEVIII
CATEGORIZATIONOFEXISTINGEXISTINGDEPLOYMENT-TIMEADVERSARIALATTACK(i.e.,WEIGHTATTACK)METHODS.
Method Venue WeightAttackwithoutTrigger WeightAttackwithTrigger
| SBA[151]             | ICCAD2017 |          |          |
| -------------------- | --------- | -------- | -------- |
| GDA[151]             | ICCAD2017 | (cid:32) |          |
| Trojaningattack[148] | NDSS2018  | (cid:32) |          |
| FSA[300]             | DAC2019   |          | (cid:32) |
(cid:32)
| AWP[81] | CIKM2020 |     |     |
| ------- | -------- | --- | --- |
(cid:32)
| TBT[192] | CVPR2020 |     |     |
| -------- | -------- | --- | --- |
(cid:32)
| TA-LBF[12] | ICLR2021 |     |     |
| ---------- | -------- | --- | --- |
(cid:32)
| Anchoringattack[296] | ICLR2021 |     |     |
| -------------------- | -------- | --- | --- |
(cid:32)
| ProFlip[30] | ICCV2021 |     |     |
| ----------- | -------- | --- | --- |
(cid:32)
| Robustnessattack[83] | ISQED2022 |     |     |
| -------------------- | --------- | --- | --- |
(cid:32)
| SRA[186] | CVPR2022 |     |     |
| -------- | -------- | --- | --- |
(cid:32)
| TSA[11]    | arXiv2022     |     |          |
| ---------- | ------------- | --- | -------- |
| T-BFA[193] | IEEETPAMI2022 |     | (cid:32) |
(cid:32)

34
TABLEIX
CATEGORIZATIONOFEXISTINGWHITE-BOXADVERSARIALEXAMPLES.FORANYCLASSIFICATIONCRITERION“A/B”, DENOTESTHEFORMER“A”,
DENOTESTHELATTER“B”. (cid:35)
(cid:32)
White-boxadversarialexamples
Optimization/ Sample-agnostic/ Additive/ Untargeted/ Factorized/
Method Venue Digtial/Physical Dense/Sparse
Learning-based specific Non-additive Targeted Structured
Sharif et al.
CCS2016
[201]
Kurakin et al. (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
ICLR2017
[116]
UAP[162] CVPR2017 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
JSMA[180] EuroS&P2016 (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
Mopuri et al. BMVA2017 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
[164]
Carlini et al. (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
IEEES&P2017
[23]
LocSearchAdv (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
CVPR2017
[167]
IEEE TPAMI (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
GD-UAP[165]
2018
Carlini et al. (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
IEEESPW2018
[24]
ECML-PKDD (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:32)
ShapeShifter[37]
2018
RP2[70] CVPR2018 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Xuetal.[269] CVPR2018 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Manifool[109] CVPR2018 (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:32)
Shietal.[204] COLING2018 (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35)
AC-GAN[212] NeurIPS2018 (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:32)
advGAN[258] IJCAI2018 (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
stAdv[259] ICLR2018 (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Athalye[9] ICML2018 (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35)
LaVAN[110] ICML2018 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
CGAN-Adv ICPR2018 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
[281]
Zhaoetal.[299] MM2018 (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Chenetal.[31] ACL2018 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
Janetal.[104] AAAI2019 (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:32)
Weietal.[241] AAAI2019 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
PS-GAN[143] AAAI2019 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
ERCG[302] CCS2019 (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Qinetal.[188] ICML2019 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Engstrom et al. ICML2019 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
[68]
Wong et al. (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35)
ICML2019
[246]
ReColorAdv (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35)
NeurIPS2019
[117]
AT-GAN[233] arXiv2019 (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35)
Yakura et al. IJCAI2019 (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
[273]
AdvFaces[53] IJCB2019 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
AGNs[202] TOPS2019 (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
SparseFool[159] CVPR2019 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Xuetal.[270] CVPR2019 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
AdvPattern[240] ICCV2019 (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:32)
CornerSearch ICCV2019 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
[49]
PD-UA[144] ICCV2019 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
Schott et al. ICLR2019 (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
[199]
Xuetal.[264] ICLR2019 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
Suetal.[216] IEEETEC2019 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
CD-UAP[287] AAAI2020 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
Xuetal.[265] ECCV2020 (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35) (cid:35)
Wuetal.[254] ECCV2020 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
SAPF[71] ECCV2020 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
UPC[97] CVPR2020 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
AdvCam[67] CVPR2020 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
LG-GAN[308] CVPR2020 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Xuetal.[267] CVPR2020 (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
PhysGAN[115] CVPR2020 (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:32)
Lietal.[124] CVPR2020 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
GreedyFool[145] NeurIPS2020 (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35)
Xuetal.[268] MM2020 (cid:35) (cid:35) (cid:32) (cid:35) (cid:32) (cid:35) (cid:35)
MAG-GAN[34] Information Sci- (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:32)
ences2020
GUAP[295] ICDM2020 (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Wong et al. ICLR2021 (cid:35) (cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35)
[245]
AdvHat[113] ICPR2021 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
Sayles et al. CVPR2021 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
[198]
TTP[168] ICCV2021 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
CMML[75] ICCV2021 (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35)
Zhaoetal.[304] NeurIPS2021 (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
SLAP[155] USENIXSecurity (cid:35) (cid:35) (cid:32) (cid:35) (cid:35) (cid:32) (cid:35)
2021
Geoinformatica (cid:32) (cid:35) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)
AP-GAN[298]
2022
(cid:35) (cid:32) (cid:32) (cid:35) (cid:35) (cid:35) (cid:35)

35
TABLEX
CATEGORIZATIONOFEXISTINGBLACK-BOXANDTRANSFER-BASEDADVERSARIALEXAMPLES.FORANYCLASSIFICATIONCRITERION“A/B”, DENOTES
|     |     | THEFORMER“A”, | DENOTESTHELATTER“B”. |     | (cid:35) |
| --- | --- | ------------- | -------------------- | --- | -------- |
(cid:32)
|                  |          |     | Black-boxadversarialexamples | Transfer-basedadversarialexamples |     |
| ---------------- | -------- | --- | ---------------------------- | --------------------------------- | --- |
| Method           | Venue    |     | Decison/Score-based          | Example/Model-level               |     |
| Brendeletal.[19] | ICLR2018 |     |                              |                                   |     |
(cid:35)
| Ilyasetal.[100] | ICML2018 |     |          |     |     |
| --------------- | -------- | --- | -------- | --- | --- |
| NES[100]        | ICML2018 |     | (cid:35) |     |     |
| GAP[184]        | CVPR2018 |     | (cid:32) |     |     |
(cid:35)
| MI-FGSM[60] | CVPR2018 |     |     |          |     |
| ----------- | -------- | --- | --- | -------- | --- |
| OPT[43]     | ICLR2019 |     |     | (cid:32) |     |
(cid:35)
| Sign-OPT[44]       | NeurIPS2019 |     |          |     |     |
| ------------------ | ----------- | --- | -------- | --- | --- |
| Subspaceattack[91] | NeurIPS2019 |     | (cid:35) |     |     |
(cid:32)
| Naseeretal.[169] | NeurIPS2019 |     |          |          |     |
| ---------------- | ----------- | --- | -------- | -------- | --- |
|                  |             |     | (cid:35) | (cid:35) |     |
| qFool[150]       | ICCV2019    |     |          |          |     |
| ILAP[98]         | ICCV2019    |     | (cid:35) |          |     |
|                  |             |     | (cid:35) | (cid:32) |     |
| SimBA[89]        | ICML2019    |     |          |          |     |
| ECO[161]         | ICML2019    |     | (cid:32) |          |     |
(cid:32)
| NAttack[133]    | ICML2019 |     |          |     |     |
| --------------- | -------- | --- | -------- | --- | --- |
| Bandit[101]     | ICLR2019 |     | (cid:32) |     |     |
| ZO-signSGD[146] | ICLR2019 |     | (cid:32) |     |     |
(cid:32)
| SignHunter[6] | ICLR2019 |     |          |     |     |
| ------------- | -------- | --- | -------- | --- | --- |
| Dongetal.[62] | CVPR2019 |     | (cid:32) |     |     |
(cid:35)
| DI2-FGSM[261] | CVPR2019 |     |          |          |     |
| ------------- | -------- | --- | -------- | -------- | --- |
| GeoDA[191]    | CVPR2020 |     | (cid:35) | (cid:32) |     |
(cid:35)
| QEBA[120] | CVPR2020 |     |     |     |     |
| --------- | -------- | --- | --- | --- | --- |
(cid:35)
| TREMBA[99]     | ICLR2020    |     |          |          |     |
| -------------- | ----------- | --- | -------- | -------- | --- |
| MetaAttack[64] | ICLR2020    |     | (cid:32) |          |     |
|                |             |     | (cid:35) | (cid:35) |     |
| FDA[102]       | ICLR2020    |     |          |          |     |
| SGM[251]       | ICLR2020    |     | (cid:35) | (cid:32) |     |
|                |             |     | (cid:35) | (cid:32) |     |
| AdvFlow[160]   | NeurIPS2020 |     |          |          |     |
| LinBP[90]      | NeurIPS2020 |     | (cid:32) | (cid:35) |     |
| LeBA[275]      | NeurIPS2020 |     | (cid:35) | (cid:32) |     |
(cid:32)
| Inkawhichetal.[103] | NeurIPS2020        |     |          |          |     |
| ------------------- | ------------------ | --- | -------- | -------- | --- |
| Suyaetal.[218]      | USENIXSecurity2020 |     | (cid:35) | (cid:32) |     |
(cid:32)
| Squareattack[7] | ECCV2020 |     |          |     |     |
| --------------- | -------- | --- | -------- | --- | --- |
| SFA[40]         | ECCV2020 |     | (cid:32) |     |     |
| RayS[32]        | ICDM2020 |     | (cid:35) |     |     |
(cid:35)
| HopSkipJumpAttack[33] | AAAI2021 |     |          |          |     |
| --------------------- | -------- | --- | -------- | -------- | --- |
| EMI-FGSM[235]         | BMVC2021 |     | (cid:35) |          |     |
|                       |          |     | (cid:35) | (cid:32) |     |
| PI-FSGM[235]          | BMCV2021 |     |          |          |     |
| IR[236]               | ICLR2021 |     | (cid:35) | (cid:32) |     |
|                       |          |     | (cid:35) | (cid:32) |     |
| PRFA[139]             | ICCV2021 |     |          |          |     |
(cid:32)
| MGAA[285]            | ICCV2021      |     |          |          |     |
| -------------------- | ------------- | --- | -------- | -------- | --- |
| Naseeretal.[168]     | ICCV2021      |     | (cid:35) | (cid:32) |     |
|                      |               |     | (cid:35) | (cid:35) |     |
| Simulatorattack[157] | CVPR2021      |     |          |          |     |
| VMI-FGSM[232]        | CVPR2021      |     | (cid:32) |          |     |
|                      |               |     | (cid:35) | (cid:32) |     |
| MSA[278]             | NeurIPS2021   |     |          |          |     |
| P-RGF[59]            | IEEETPAMI2021 |     | (cid:32) |          |     |
| CG-Attack[77]        | CVPR2022      |     | (cid:32) |          |     |
(cid:32)
| SVRE-MI-FGSM[263] | CVPR2022      |     |          |          |     |
| ----------------- | ------------- | --- | -------- | -------- | --- |
| CISA[206]         | IEEETPAMI2022 |     | (cid:35) | (cid:32) |     |
(cid:35)
| MCG[279] | IEEETPAMI2022 |     |          |          |     |
| -------- | ------------- | --- | -------- | -------- | --- |
|          |               |     | (cid:32) | (cid:35) |     |
