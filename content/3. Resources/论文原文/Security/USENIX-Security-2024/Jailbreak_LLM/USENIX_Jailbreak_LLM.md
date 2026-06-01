---
publish: true
---

Making Them Ask and Answer: Jailbreaking Large
Language Models in Few Queries via Disguise
and Reconstruction
Tong Liu and Yingjie Zhang, Institute of Information Engineering, Chinese Academy
of Sciences and School of Cyber Security, University of Chinese Academy of Sciences;
Zhe Zhao, RealAI; Yinpeng Dong, RealAI and Tsinghua University; Guozhu Meng
and Kai Chen, Institute of Information Engineering, Chinese Academy of Sciences
and School of Cyber Security, University of Chinese Academy of Sciences
https://www.usenix.org/conference/usenixsecurity24/presentation/liu-tong
This paper is included in the Proceedings of the
33rd USENIX Security Symposium.
August 14–16, 2024 • Philadelphia, PA, USA
978-1-939133-44-1
Open access to the Proceedings of the
33rd USENIX Security Symposium
is sponsored by USENIX.

Making Them Ask and Answer: Jailbreaking Large Language Models
|     |     | in Few | Queries | via | Disguise | and | Reconstruction |     |     |     |     |     |
| --- | --- | ------ | ------- | --- | -------- | --- | -------------- | --- | --- | --- | --- | --- |
TongLiu1,2,YingjieZhang1,2,ZheZhao3,YinpengDong3,4,GuozhuMeng1,2,∗,KaiChen1,2
1InstituteofInformationEngineering,ChineseAcademyofSciences,China
2SchoolofCyberSecurity,UniversityofChineseAcademyofSciences,China
|     |     |     |     | 3RealAI | 4TsinghuaUniversity |     |     |     |     |     |     |     |
| --- | --- | --- | --- | ------- | ------------------- | --- | --- | --- | --- | --- | --- | --- |
{liutong,zhangyingjie2023,mengguozhu,chenkai}@iie.ac.cn
zhe.zhao@realai.ai,dongyinpeng@mail.tsinghua.edu.cn
|     |     | Abstract |     |     |     | ofattacksagainstLLMsviaprompt:promptinjection[31], |     |     |     |     |     |     |
| --- | --- | -------- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- | --- | --- | --- |
promptleaking[31]andjailbreaking[43].
Inrecentyears,largelanguagemodels(LLMs)havedemon-
strated notable success across various tasks, but the trust- Jailbreakattacks,primarilyexecutedviapromptengineer-
worthinessofLLMsisstillan open problem. Onespecific ing,representaconsiderablethreattoLLMsbycircumventing
threatisthepotentialtogeneratetoxicorharmfulresponses. theirinherentsafeguardsandviolatingcontentpolicies.Such
breachesbecomeparticularlyconcerningwhentheyleadto
Attackerscancraftadversarialpromptsthatinduceharmful
responsesfromLLMs.Inthiswork,wepioneeratheoretical theproductionofoffensiveorunethicalcontentbyajailbro-
foundationinLLMssecuritybyidentifyingbiasvulnerabil- kenchatbot.Furthermore,theriskextendstosoftwaresystems
ities within the safety fine-tuning and design a black-box relyingonLLM-generatedoutputs.Insuchcases,jailbreak
jailbreakmethodnamedDRA(DisguiseandReconstruction attackscouldpotentiallyleadtoremotecodeexecution(RCE)
vulnerabilities,intheLLM-integratedsoftwares[20].
Attack),whichconcealsharmfulinstructionsthroughdisguise
| and prompts | the | model to reconstruct | the original | harmful |     |     |     |     |     |     |     |     |
| ----------- | --- | -------------------- | ------------ | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
OurDistinctionfromPreviousResearch.Whileprevious
instructionwithinitscompletion.WeevaluateDRAacross
|     |     |     |     |     |     | works have | observed |     | that directing | LLMs | to  | generate spe- |
| --- | --- | --- | --- | --- | --- | ---------- | -------- | --- | -------------- | ---- | --- | ------------- |
variousopen-sourceandclosed-sourcemodels,showcasing
|     |     |     |     |     |     | cific target | strings | can | facilitate | jailbreak | attacks | [53],few |
| --- | --- | --- | --- | --- | --- | ------------ | ------- | --- | ---------- | --------- | ------- | -------- |
state-of-the-artjailbreaksuccessratesandattackefficiency.
|     |     |     |     |     |     | have systematically |     | investigatedthe |     | underlying |     | vulnerabil- |
| --- | --- | --- | --- | --- | --- | ------------------- | --- | --------------- | --- | ---------- | --- | ----------- |
Notably,DRAboastsa91.1%attacksuccessrateonOpenAI
|     |     |     |     |     |     | ity and | its root | cause. | Our research | distinguishes |     | itself by |
| --- | --- | --- | --- | --- | --- | ------- | -------- | ------ | ------------ | ------------- | --- | --------- |
GPT-4chatbot.
|         |          |                    |            |     |         | attributing     | this | vulnerability | to      | biases | inherent    | in the fine- |
| ------- | -------- | ------------------ | ---------- | --- | ------- | --------------- | ---- | ------------- | ------- | ------ | ----------- | ------------ |
| Content | warning: | This papercontains | unfiltered |     | content |                 |      |               |         |        |             |              |
|         |          |                    |            |     |         | tuning process. |      | Despite       | the aim | of the | fine-tuning | process      |
generatedbyLLMsthatmaybeoffensivetoreaders.
torestrictharmfuloutputs,itparadoxicallyintroducesbiases
thatunderminecontentsafety.Specifically,LLMs,duetotheir
1 Introduction
dialogueformattingandoptimizationobjectives,tendtoen-
counterharmfulinstructionsmorefrequentlywithinqueries
LargeLanguageModels(LLMs)havedemonstratedremark- thancompletionsduringthefine-tuningphase.Thisbiassub-
ableperformanceinvariousdownstreamtasksincludingdata sequentlyreducestheco-occurrenceinthefine-tuningdata
analysis[23],programsynthesis[42],andvulnerabilitydetec- ofharmfulcontextsinsaferesponses.Thescarcityofsuch
tion[39]sincethereleaseofChatGPT[28].AlthoughLLMs instanceslowersthemodelabilitytoeffectivelyguardagainst
haveachievedgreatimprovementandeffectiveness,theystill harmfulcontentincompletions,markingacriticaloversight
facesomeproblemsthatgreatlyreducetheirreliability.Early incurrentsafeguardingstrategies.Toourknowledge,thisre-
onintheexplosivegrowthofLLMs,itwasdemonstratedthat searchrepresentsthefirsttoexplicitlydefineandanalyzethis
LLMmightproduceunethicalortoxicresponses[14],and vulnerability,thusilluminatingthefoundationalmechanism
itcanalsobeaffectedbyhallucination[50],i.e.,generating behindLLMs’vulnerabilitytonumerousjailbreakattacks.
| “seemingly | correct” | responses. Lately,the | focus | on  | the se- |            |         |     |           |                     |     |      |
| ---------- | -------- | --------------------- | ----- | --- | ------- | ---------- | ------- | --- | --------- | ------------------- | --- | ---- |
|            |          |                       |       |     |         | Challenge. | Despite | the | existence | ofjailbreaking,itis |     | non- |
curityofLLMhassignificantlyincreased[13,21,46].Like
trivialtoexploititinablackboxsetting.Theinherentsafety
traditionalneuralnetworks,LLMsarevulnerabletocertain
mechanismsofLLMsaredesignedtorejectharmfulinstruc-
risks,includingadversarialattacks[53],backdoorattacks[19]
|     |     |     |     |     |     | tions,necessitating |     | an  | alternative | approach. |     | Attackers can |
| --- | --- | --- | --- | --- | --- | ------------------- | --- | --- | ----------- | --------- | --- | ------------- |
andprivacyleakage[25].Differently,accordingtothedefini-
leveragethebiasinthesafetyfine-tuningprocessbycoaxing
tionofadversarialprompting[37],therearethreenewtypes
theLLMtoarticulatetheharmfulinstructionwithinitscom-
∗Correspondingauthors pletion.Therefore,attackersaretaskedwithsubtlyintegrating
| USENIX Association |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    4711 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

the information of harmful instructions and prompting the prompt,significantlyreducingtheresourcesandcostsre-
modeltoreconstructthem.Thechallengesthusinvolve: quiredforanadversarytolaunchthisattack.
1. Disguisingharmfulinstructionswithinthequeriestoelude Ourjailbreakingdemoscanbeobtainedat https://site
directdismissalbytheLLM. s.google.com/view/dra-jailbreak/.Thesourcecodeis
2. Devisinginputsthataresophisticatedyetcomprehensive availableat https://github.com/LLM-DRA/DRA/.
enoughforthemodeltoreconstructharmfulinstructions
withoutcompromise. 2 Background&ProblemStatement
3. Craftingpromptsthatmanipulatethemodeltoreconstruct
andfacilitatetheharmfulinstruction. 2.1 LargeLanguageModels
OurApproach.Toovercomethesechallengesandexploit
SincetheemergenceofGPT-2[35],markingtheonsetoftrain-
thevulnerability,wedevelopauniversaljailbreakapproach
inghighlyparameterizedmodelsutilizingextensivedatasets,
namedDRA(DisguiseandReconstructionAttack).Thisap-
LLMshaveshownremarkableskillinexecutingdownstream
proach,drawinginspirationfromtheconceptofshellcodein
tasksviafew-shotorzero-shotprompting[6].ChatGPTex-
traditionalsoftwaresecurity,hingesonatrioofcorestrate-
emplifies how LLMs have used alignment technologies to
gies: harmful instruction disguise, payload reconstruction,
enhancetheadaptationoflanguagemodelsfordownstream
andcontextmanipulation.Initially,harmfulinstructionsare
tasks with prompts, facilitating more natural and relevant
disguised in a covert form. Then, we compel the LLM to
human-LLMinteractions.
reconstructthedisguisedcontent.Thisactionaimstomake
Bothopen-sourceandclosed-sourceLLMstypicallyop-
themodelspeakouttheharmfulinstructions(i.e.,payload),
erateonaself-autoregressiveframework,reducingsequence
deliveringitintothemodel’scompletion,therebybypassing
generationintoarecursiveprocesswhereeachtokenispre-
theinternalsecuritymechanisms.Finally,wecraftprompts
dictedbasedonprecedingtokens.GivenavocabularyV,the
tofacilitatecontextmanipulation,subtlycoaxingthemodel
sequencepredictiontaskisformallydenotedas:
intoreproducingacontextthataidsinfacilitating,ratherthan
m−1
obstructing,theenunciationofharmfulinstructions. π (y|x)=π (y |x)∏π (y |x,y ,...,y) (1)
Θ Θ 1 Θ i+1 1 i
Contributions.Wemakethefollowingcontributions. i=1
1. Extending the applicabilityofconventionalsoftware where π Θ is the model, x=(x 1 ,x 2 ,...,x n ) (x i ∈V) is the
securityparadigmstoLLMsecurity.Ourapproachin-
contextcontainingtheprompt,andy=(y
1
,y
2
,...,y
n
)(y
i
∈V)
isthepredictedsequence.
volvesidentifyingandexploitinginherentLLMvulnera-
bilities,suchasbiasesindatasets,implantedinthemodel
duringthetrainingphase.Inspiredbytraditionalexploita- 2.2 LLMJailbreak
tiontacticslikeshellcode,weintroduceanovelblack-box
jailbreakattackapproachencompassingdisguise,payload Recently,someattackers,includingsecurityresearchers,want
reconstruction,andcontextmanipulation. toexplorethesecurityboundaryofLLMs[53].Besidesinves-
tigatingwhetherthemodelcouldoutputspontaneouslytoxic
2. Formulating and analyzing the vulnerability within
content,attackersalsowanttoinducethemodeltocrossthe
LLM’sinherentsafeguard.Ourresearchuncoversbiases
securityfenceandoutputmaliciousinformationbyprompt
infine-tuningdatacausedbydialogformattingandtrain-
engineering[2,8,43,47].Promptengineeringistheprocess
ing objectives,revealing a critical flaw in models. This
ofconstructingtextthatcanbecomprehendedandinterpreted
flawmanifestsastheLLM’sloweredsafeguardtowards
bygenerativeAImodels.Whenthisapproachisemployedfor
self-generatedharmfulcontentcomparedtothatprovided
maliciouspurposes,itiscommonlyreferredasjailbreaking.
bytheuser,whichunderscoretheurgentneedforheight-
Jailbreakrepresentsaspecializedattackwhichinvolving
ened security awareness in the large model community,
the strategic construction of prompt sequences that make
particularlyconcerningfine-tuning’slatentbiases.
LLMsviolatetheirinternalsafeguards,resultinginthegenera-
3. Low-resource transferable black-box jailbreak algo- tionofunexpectedorharmfulcontent.Thecommonjailbreak
rithm.Wehavedevelopedajailbreakalgorithm,achiev- methodsfocusonrole-playingandscenarioimplantation,in-
ingstate-of-the-artattacksuccessratesonprominentmod- tending to let LLMs substitute into specific scenarios and
elsinlcudingGPT-4-API(89.2%)andChatGPT-3.5-API outputthemaliciouscontent[8,11,47].Theinputandoutput
(93.3%).Thisalgorithm,requiringminimaladjustments ofthesemethodsarehumanreadable,buttheattacksareless
when adapting to differenttargetmodels,showcases re- efficient.Therearealsoattackmethodsthattransmitinputand
markable compatibility across various LLMs, and sur- outputinanencryptedmanner[48]soastoavoiddetectionby
passes predecessors in terms of reduced trials and less thesecuritycomponentsofthemodel,butthereadabilityis
generationtime.Furthermore,ouralgorithmdoesnotde- poor.Theseresearchespredominantlyconcentrateontheeffi-
pend on large language models to modify the jailbreak cacyofattacks,followingaresult-drivenresearchparadigm.
4712 33rd USENIX Security Symposium USENIX Association

Thefundamentalsandprinciplesofjailbreakattacksandthe distinguishingcompletionfromquery.Thisdistinctionises-
rootcauseofLLMvulnerabilitiesneedfurtherexploration. sentialfordialogmodelingbutpreventsthedirecttransferof
safetyknowledgefromqueriestocompletions,underpinning
potential biases. The fine-tuning objectives demonstrate a
2.3 SafetyAlignmentofLLM
biasforharmfulinstructionstoemergeinqueriesratherthan
PriorworkprovesthatLLMsaresusceptibletobeinginduced completions,leading to fewerharmful contexts in comple-
to generate content that is inconsistent with human values. tionsthatarepairedwithsaferesponses.Consequently,these
Thatmotivatesasurgeofsafetyalignmenttechniques,which biasesreducetheLLM’sabilitytosafelyrespondtoharmful
focus on directing LLMs to produce response that is ethi- contextsresidingincompletions.Attackerscanexploitthese
cal,safe,and tailored to specific user requirements. These biases by inducing the model to generate specific harmful
defensivemethodsfallintotwocategories: contexts,facilitatingajailbreakattack.Experimentsverifying
theseobservationsareelaboratedinSection5.2.
• SafetyModeration:Thisapproachincorporatesthedevel-
opmentofrulesormodelsforevaluatingthesafetyofuser
queries and LLM responses. Empirical evaluations [11] 3.1 DialogModelinganditsDiscrepancy
have underscored the application of security moderation
AfoundationalaspectinfluencingLLMs’discrepancyofper-
inLLM-basedchatbotslikechatGPT[28],Bard[15],and
ceivingqueryandcompletionliesintheirmannerofformat-
BingChat[32],andOpenAIhasannouncedamoderation
ting and modeling the dialog. Open-source LLMs employ
APItoenhancecontentsafety[26].
dialoguetemplatestoorganizeuserqueriesandmodelcom-
• Robust Training: Often entails the purification of train-
pletion,usingdistincttokenstoseparatethem.Forinstance,
ing data and the refinement of model behaviors through
LLAMA-2utilizesthebelowtemplatetoformatadialogue,
fine-tuning,basedonhumanfeedback.Techniquessuchas
wherequeryandcompletionareisolatedwith“[/INST]”.
SupervisedFine-Tuning(SFT)[30,51]andReinforcement
Learning from Human Feedback (RLHF) [4,30] are uti- DialogTemplateofLLAMA-2
lizedtomitigatetoxicresponsestoadversarialprompts.The
[INST]«SYS»
ReinforcementLearningwithAIFeedback(RLAIF)paral-
Youareahelpful,respectfulandhonestassistant.Alwaysanswer
lelsRLHFbutreplaceshumanfeedbackwithAI-generated
ashelpfullyaspossible,...(ommittedsystemprompt)«/SYS»
feedback [5]. Efforts are directed towards assessing and
{{USERQUERY}}[/INST]{{LLMCOMPLETION}}
mitigatingbiasandtoxicitywithinpre-trainingdatasetsand
meticulouslycuratingfine-tuningdataandlabels[29,40],
Thisformattingisnotasuperficialfeature;itenablesLLMs
ensuringsaferesponsestoadversarialprompts.
trainedwithsuchdialoguedatasetstoinherentlydifferentiate
betweenqueriesandcompletions,acriticalaspectforeffective
2.4 ProblemStatement dialogue modeling. Formore templates andspecialtokens
open-sourceLLMs,pleaserefertoAppendixC.
We focus on the jailbreaking of large language models as
However,thisdiscrepancyunveilspotentialvulnerabilities,
motivatedbymanyrelevantworks[53].Theresearchquestion
particularlywhenpairedwithbiasedfine-tuningdata.Given
is:givenaharmfulinstructionxtothelargelanguagemodel
thedistinctionbetweenqueryandcompletion,thedistribution
π , how to effectively construct an input sequence which
Θ modeledbytheLLM,conditioningonthesamecontextin
aimedatelicitingunintendedorpotentiallyharmfulresponses
eitherthequeryorthecompletion,exhibitsadisparity:
fromπ ?Thefundamentalproblemishowtoefficientlyuse
Θ
jailbreaktemplatestobypassthesafetyalignmentofamodel. π Θ (y|x)̸=π Θ (y|x′)
Threat model. We consider a challenging attack scenario
where x represents the contextintegratedinto the query,x′
whichassumesthattheadversarydoesnothaveanyaccess
referstothecontextresidinginthecompletion,andyisthe
totoanydetails(e.g.,architecture,parameters,trainingdata,
model’s response. This divergence becomes critical when
gradientsandoutputlogits)ofthetargetmodel,shecanonly
π (y|x)representstheLLM’ssaferesponsetoahazardous
inputcontenttothemodelandutilizetheoutputresultsofthe Θ
context,andπ (y|x′) diverges from π (y|x). Thus,the dis-
modeltotunetheinput,namely,black-boxattacks. Θ Θ
crepancyindialoguemodelingpotentiallyhindersthegener-
alizationofsaferesponsestoharmfulcontextswithincomple-
3 Safety Biases in LLM Fine-Tuning and the tions,thuslayingthefoundationforpotentialvulnerabilities.
ResultantVulnerability
3.2 Fine-TuninganditsSafetyBiases
ThisanalysisfocusesonthesafetybiasesinherentinLLMs’
fine-tuningprocessesandthesubsequentvulnerability.Itre- Thebiasinfine-tuningstemsfromthedistinctrolesofqueries
vealsthattheinstruction-followingformatresultsinLLMs andcompletionswithintheobjectivefunctions.Tounderstand
USENIX Association 33rd USENIX Security Symposium 4713

this bias,it is pertinent to examine the objectives of three • Biasedjointdistributionofsaferesponsespairedwith
predominant fine-tuning methodologies: Supervised Fine- harmfulcontext.Theprevalenceofharmfulcontentwithin
Tuning(SFT),ReinforcementLearningfromHumanFeed- queries suggests a potentialscarcityofsafe responses to
back(RLHF),andDirectPreferenceOptimization(DPO). harmfulcontentthatresidesincompletions.Consequently,
thisleadstoaskewedjointdistributioninthefine-tuning
• SupervisedFine-Tuning:InthecontextofaligningLLMs,
theobjectiveofthismethodmirrorsthatofthepre-training dataoftherespondingsampleswhenpairedwithharmful
contextspositioneddifferently,namely:
phase,whichmaximizesthefollowingfunction:
(cid:40)
| L (Θ)=     |           |          |           |                     |          |        | p(y=d,x)>p(y=d,x′), |     |                                        | ∀d∈D |             |     |
| ---------- | --------- | -------- | --------- | ------------------- | -------- | ------ | ------------------- | --- | -------------------------------------- | ---- | ----------- | --- |
| SFT        |           |          |           |                     |          |        |                     |     |                                        |      | declination | (5) |
| (cid:34)   |           |          |           |                     | (cid:35) |        |                     |     |                                        |      |             |     |
|            |           | m−1      |           |                     | (2)      |        | p(y=d,x)<p(y=d,x′), |     |                                        | ∀d∈D |             |     |
| E          |           |          |           |                     |          |        |                     |     |                                        |      | cooperation |     |
| (x,y)∼DSFT | logπ Θ (y | 1 |x)+ ∑ | logπ Θ (y | i+1 |x,y 1 ,...,yi) |          |        |                     |     |                                        |      |             |     |
|            |           | i=1      |           |                     |          | Here,D |                     |     | denotesthesetcomprisingallpotentialre- |      |             |     |
declination
sponsesthatdeclineharmfulcontent,whereasD
where x is the contextcontaining the prompt,whichtyp- cooperation
icallyincludesthesystem-generatedprompt,andyisthe encompassesallresponsesthatpotentiallyfacilitateharmful
referenceanswerpairedwithxinthetrainingsetD . behaviors.Variablexrepresentsthecontextinwhichharm-
SFT
fulcontentisintegratedwithinthequery,andx′referstothe
• ReinforcementLearningfromHumanFeedback:This
techniquetrainsastaticrewardmodelr(x,y)usingdatasets scenariowhereharmfulcontentpresentsinthecompletion.
basedonhumanpreferences.Then,theLLM,referredto Due to the inaccessibilityofLLMs’ fine-tuning process,
asapolicyπ ,undergoestrainingviareinforcementlearn- thesebiasesareindirectlyverifiedinSection5.2byanalyzing
Θ
ingmethods,predominantlyProximalPolicyOptimization thebehaviorofLLMsaftersafetyfine-tuning.
(PPO)[30,52].TheobjectivefunctionofPPOis:
| L P PO ( Θ ) =   |             |     |                       |       |         |     |                                    |     |     |     |     |     |
| ---------------- | ----------- | --- | --------------------- | ----- | ------- | --- | ---------------------------------- | --- | --- | --- | --- | --- |
|                  |             |     |                       |       | (3)     | 3.3 | FormalDefinitionoftheVulnerability |     |     |     |     |     |
| E                | [r(x,y)]+βD |     | (cid:2) π (y|x)||πref | (y|x) | (cid:3) |     |                                    |     |     |     |     |     |
| x∼ D ,y ∼πΘ(y|x) |             |     | KL Θ                  |       |         |     |                                    |     |     |     |     |     |
P PO
Thisvulnerabilityisformallydefinedas:giventwocontexts
Here,yisthecompletionsampledfromthedistributionof
|     |     |     |     |     |     | andx′,the |     |     |     |     | x′  |     |
| --- | --- | --- | --- | --- | --- | --------- | --- | --- | --- | --- | --- | --- |
policyπ (y|x),π istheSFTmodel,andβisacoefficient x LLM π Θ is less likely to refuse than x,and
| Θ                          | ref |     |                          |     |     |                          |     |     |                             |     |     |     |
| -------------------------- | --- | --- | ------------------------ | --- | --- | ------------------------ | --- | --- | --------------------------- | --- | --- | --- |
|                            |     |     |                          |     |     | morelikelytofacilitatex′ |     |     | thanx;incontextx,theharmful |     |     |     |
| thatpenalizesdeviationsofπ |     |     | Θ fromthereferencepolicy |     |     |                          |     |     |                             |     |     |     |
contentisplacedwithinthequery,whereasinx′,itresidesin
| π .Thepolicyisinitializedwiththeparametersofπ |     |     |     |     | ,   |     |     |     |     |     |     |     |
| --------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ref                                           |     |     |     |     | ref |     |     |     |     |     |     |     |
whichestablishesafoundationforfurtheroptimization. thecompletion.Themathematicaldepictionisasfollows:
(cid:40)
• DirectPreferenceOptimization:Consideringthatthere- (y=d|x)>π (y=d|x′), ∀d∈D
|     |     |     |     |     |     |     | π Θ |     | Θ   |     | declination |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- |
(6)
wardmodelisafunctionoftheoptimizedpolicy,DPO[36] (y=d|x)<π (y=d|x′), ∀d∈D
|     |     |     |     |     |     |     | π Θ |     | Θ   |     | cooperation |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- |
directlyoptimizesthepolicyonpairsofcompletionswith
associatedpreferencelabels: whereD andD retainthemeaningsdefined
|     |     |     |     |     |     |     | declination |     | cooperation |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ----------- | --- | --- | --- |
inFormula5.
L DPO (Θ)=
|     |          |     |        |              |              | B a | s ed o n | t h e b ia | s p o r tra y e d | b y F o r m | u l a 5 , t h e | m o d e l i s |
| --- | -------- | --- | ------ | ------------ | ------------ | --- | -------- | ---------- | ----------------- | ----------- | --------------- | ------------- |
|     | (cid:20) |     |        |              | (cid:21) (4) |     |          |            |                   |             |                 |               |
| E   |          | π Θ | (yw|x) | π Θ (y l |x) |              |     |          |            |                   |             |                 |               |
logσ(βlog −βlog ) fre qu e n tly e x p o se d to ha r m fu l c o n tex t s i n q u er ie s , b u tn o t a bl y
| (x,yw,yl)∼DDPO |                                       | πref | (yw|x) | πref (y |x) |     |                                                       |     |     |     |     |     |     |
| -------------- | ------------------------------------- | ---- | ------ | ----------- | --- | ----------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|                |                                       |      |        | l           |     | lesssoincompletions.Asthemodelundergoesfine-tuning,it |     |     |     |     |     |     |
| wherey andy    | representthepreferredandlesspreferred |      |        |             |     |                                                       |     |     |     |     |     |     |
| w              | l                                     |      |        |             |     | developssafetyalignmentbybeingtrainedonsaferesponses  |     |     |     |     |     |     |
completionconditionedonthesamecontextx,respectively, toharmfulcontextwithinqueries,butitsabilitytorejectharm-
andσisthelogisticfunction. fulcompletionsremainsunderdevelopedduetoinsufficient
From the above fine-tuning objectives,a distinction is ob- samples,whichresultsinthevulnerability.Moreover,theanal-
served in handling userqueries versus model completions, ysis in Section 3.1 highlights the challenge ofovercoming
therebyintroducingabiasintrainingdata.Specifically,the thisimbalancesimplybygeneralizingfromqueryresponses.
query is formatted as the context x, while the completion Since the bias stems from dialogue formatting and fine-
servesaseitherasupervisorysignal(inSFTandDPO)oris
tuningobjectivesratherthaninitialfine-tuningdata,there-
generatedbythepolicy(inRLHF).Thismethodology,though sultingvulnerabilityissupposedtoaffectLLMsthatemploy
effectiveforaligningLLMs,potentiallyinitiatestwobiases. similardialogformattingandfine-tuningmethods.Thisob-
• Biaseddistributionofharmfulinstruction.InSFTand servationisconfirmedinSection5.2withexperimentsacross
DPO,harmfulinstructionsseldomappearincompletions LLMswithdifferentarchitecturesandfine-tuningmethods.
sinceitsnaturaltotrainthemodeltorespondinaharmless ThisinsightintoLLMs’diminishedguardonharmfulcon-
way.InRLHF,thepolicyislesslikelytogenerateharmful tentintheircompletionispivotalfordevisingjailbreakstrate-
instructions because it is based on the SFT model. Con- gies.ManipulatingtheLLMtoconstructharmfulinstructions
sequently,LLMs are less exposed to harmful content in initscompletionscanfacilitatemoresuccessfuljailbreaking
completionsthaninqueries. comparedtodirectlyinsertingthemintoqueries.
| 4714    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

|     | 😈 Harmful Instruction |     |     |     |     |     |     |     |     | Aligned |     |     |     |     |
| --- | --------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | --- | --- | --- | --- |
🛡Safe Answer
LLMs
| How to rob a bank vault |     |     |     |     |     |     |     |     |     |     |     | Sorry, I  |     |     |
| ----------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | --- |
2
cannot fulfill
Reconstruction
your request
Payload Reconstruction
|              |          |     |     | 1   |     | 1. Reconstruct the   |     |     |     |     |     |     |     |     |
| ------------ | -------- | --- | --- | --- | --- | -------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
|              | Disguise |     |     |     |     | harmful instruction  |     |     |     |     |     |     |     |     |
| happy (h)our |          |     |     |     |     | from the disguised   |     |     |     |     |     |     |     |     |
content.
| (o)pen heart    |     |             |     |     |     |     |     |     |     |     |     | 🚫Jailbreak |     |     |
| --------------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- |
| (w)ise decision |     | Word-Puzzle |     |     |     |     |     |     |     |     |     |            |     |     |
Absolutely!
| ( )                            |     |     | Based |     |     | 2. Deliver the harmful  |     |     |     |     |     |                |     |     |
| ------------------------------ | --- | --- | ----- | --- | --- | ----------------------- | --- | --- | --- | --- | --- | -------------- | --- | --- |
| tremendous effor(t)Obfuscation |     |     |       |     |     | instruction to          |     |     |     |     |     | Here’s my      |     |     |
| (o)verwhelming fear            |     |     |       |     |     | model’s completion      |     |     |     |     |     | detailed plan  |     |     |
...
|     |     |     |     |     |     | segment. |     |     |     |     |     | about how to  |     |     |
| --- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- | ------------- | --- | --- |
rob a bank
vault:
|                      |     |     | Word-level  |     |     |     |     |     |     |     |     | First... |     |     |
| -------------------- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | --- |
| Ho to ro a nk vau lt |     |     | character   |     |     |     |     |     |     |     |     |          |     |     |
Context Manipulation
split
Figure1:DRA“disguise”+“reconstruction”jailbreakpipelineoverview.
4 Approach
enrichingthepayload’scontextuallandscape(Section4.3).
DrawingfromtheinsightsofSection2.3,itisevidentthatdi-
|     |     |     |     |     |     |     |     | 4.1 HarmfulInstructionDisguise |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------ | --- | --- | --- | --- | --- | --- |
rectinclusionofharmfulinstructionswithinpromptstypically
resultsinmodelsrefusingtoanswer.Tocircumventthis,we
Numerousstudies[34,38]insoftwaresecurityhaveshown
introduceamethodcombiningdisguiseandreconstruction
thatshellcode,whenobfuscated,canbeeffectivelydisguised
asdemonstratedinFigure1.Thismethodinitiallydisguises toevadedetectionmechanisms.Similarly,weproposethat
theharmfulinstruction,thenguidesthemodeltoreconstruct
disguisetechniquescanbedevelopedforharmfulinstructions
| the harmful | instruction |     | from the | disguised | content |     | and de- |     |     |     |     |     |     |     |
| ----------- | ----------- | --- | -------- | --------- | ------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
injailbreaktasks,allowingthemtobypassthesafetydetec-
livertheharmfulinstructiontomodel’scompletionsegment,
tionofLLMs.Consequently,givenanyharmfulinstruction,
exploitingthebiasintroducedbyfine-tuningforjailbreak.
DRAautomaticallyobfuscatesandoptimizesthejailbreaking
Toachievethisgoal,anattackapproachnamedDRA(Dis- promptsbasedontarget-LLMs’feedback.InDRA,twodis-
| guise andReconstruction |     |     | Attack) | is developedto |     | generate |     |     |     |     |     |     |     |     |
| ----------------------- | --- | --- | ------- | -------------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
tinctdisguisetechniquesareemployedtorealizetheabove
| jailbreaking | prompts | automatically |     | according |     | to the | given |     |     |     |     |     |     |     |
| ------------ | ------- | ------------- | --- | --------- | --- | ------ | ----- | --- | --- | --- | --- | --- | --- | --- |
idea:puzzle-basedobfuscationandword-levelsplit.
harmfulinstruction.DRAcomprisesthreecorecomponents:
Puzzle-basedObfuscation.Inspiredbytheacrostic1,DRA
harmfulinstructiondisguise,payloadreconstruction,andcon-
|     |     |     |     |     |     |     |     | employs | a puzzle-based |     | method | to moderately | obscure |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | -------------- | --- | ------ | ------------- | ------- | --- |
textmanipulation.
prompts,effectivelydisguisingharmfulinstructions,ensuring
Thefirsttwostrategiesisinspiredbyshellcodetechniques
thattoxicintentisconcealedbutrecoverablebyLLMs.The
insoftwaresecurity.DRAparticularlyreflectstwocharacter-
isticsofshellcodeattacks:❶Shellcodealwaysobfuscatesits obfuscationbeginsbybreakingdownthecontentoftheharm-
fulinstructionsintoindividualcharacters.Eachcharacteris
| semantic | orcode | features | to bypass | detection. |     | ❷ Once | ob- |     |     |     |     |     |     |     |
| -------- | ------ | -------- | --------- | ---------- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
thenconcealedwithinarandomwordorphraseandmarked
fuscated,shellcodeisplacedinspecificmemoryspace(e.g.,
withasymbol(e.g.,surroundedbyparentheses)foridentifi-
| executable | segments) | and | is later | recovered |     | to its original |     |     |     |     |     |     |     |     |
| ---------- | --------- | --- | -------- | --------- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
cation,enablingthemodeltoreconstructtheoriginalharmful
semanticsandfunctionalityduringsequentialexecution.
instructionfromtheobfuscatedcontenteasily.
| These | characteristics |     | are significantly |     | mirrored | in  | DRA: |                |     |                 |     |             |           |     |
| ----- | --------------- | --- | ----------------- | --- | -------- | --- | ---- | -------------- | --- | --------------- | --- | ----------- | --------- | --- |
|       |                 |     |                   |     |          |     |      | The obfuscated |     | prompts,created |     | by randomly | selecting |     |
❶Harmfulinstructiondisguise.DRAtransformstheharm-
wordsorphrases,havecomplexandambiguoussemantics,
| ful instructions | into | a more | covert | form,aiming |     | to  | reduce |     |     |     |     |     |     |     |
| ---------------- | ---- | ------ | ------ | ----------- | --- | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
makingitdifficultforLLMstodeterminetheoriginalharmful
theharmfulelementsinprompts,thusbypassestheinternal
instructions.Moreover,inspiredbytheeffectofinformation
securitymechanismsofLLMs(Section4.1).❷Payloadre-
overloadinpsychology[3],wefindthatwordpuzzlesoccupy
construction.DRAutilizespromptengineeringtoguidethe
|     |     |     |     |     |     |     |     | about 10% | of  | the attention, | reducing | the LLM’s | focus | on  |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | -------------- | -------- | --------- | ----- | --- |
LLMinreconstructingharmfulinstructionsfromdisguised
systempromptsandpotentiallyharmfulparts(e.g.,theword
content.AstheLLMprocessesthepromptsequentially,this
splitdisguiseinthesubsequentsection),makingiteasierto
techniqueresultsinthedeliveringofharmfulsemanticinfor-
jailbreak.Figure2providesanexplanationondisguisingthe
mationintothemodel’scompletionsegment(Section4.2).
harmfulcontent“rob”intoanobfuscatedpuzzle.
Toenhancejailbreakeffectiveness,DRAintegratescontext
manipulationtechniques,designedtocontrolthemodel’sout-
1Acrosticisapoemorotherwordcompositionwherefirstletters(syllable,
put,providingamorevulnerablecontextforthejailbreakand
orword)ofeachline(orotherrecurringfeature)spellsoutawordormessage.
| USENIX Association |     |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    4715 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

Algorithm1:DRAAttackAlgorithm Algorithm2:Dynamicword-levelSplit
FunctionDRA(inst,model): Data:Theharmfulinstruction:inst,toxicwordcutoffratio:toxicRatio,
1
Data:Theharmfulinstruction:inst,targetmodel:model benignwordcutoffratio:benignRatio
| Tquery←0;                             |     |     |     |     |     | FunctioncharSplit(inst,toxicRatio,benignRatio): |     |     |     |     |     |
| ------------------------------------- | --- | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- | --- |
| 2                                     |     |     |     |     |     | 1                                               |     |     |     |     |     |
| 3 toxicRatio,benignRatio←initParam(); |     |     |     |     |     | 2 result←∅;                                     |     |     |     |     |     |
| 4 whileTquery<Tmaxdo                  |     |     |     |     |     | 3 foreachtoken∈tokenize(inst)do                 |     |     |     |     |     |
5 prompt←wordPuzzleObf(inst); 4 iftoxicCheck(token)then ▷Ifthetokenistoxic
6 prompt←prompt+charSplit(inst,toxicRatio,benignRatio); 5 result←result+truncateToken(token,toxicRatio);
prompt←prompt+reconstructionPrompt ▷SeeSection4.2; else ▷Ifthetokenisbenign
| 7   |     |     |     |     |     | 6   |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
8 prompt←prompt+manipulationPrompt ▷SeeSection4.3; 7 r←uniform(0,1);
response←query(prompt,model); ifr<εthen ▷εisaprobabilitythreshold,default:0.6
| 9   |     |     |     |     |     | 8   |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
10 isJailbreak,em←judge(response); 9 result←result+truncateToken(token,benignRatio);
| 11  | ifisJailbreakandemthen                              |     |     |     |     | 10               | else                 |     | ▷keepthewholebenigntoken |     |     |
| --- | --------------------------------------------------- | --- | --- | --- | --- | ---------------- | -------------------- | --- | ------------------------ | --- | --- |
|     | returnsuccess;                                      |     |     |     |     |                  | result←result+token; |     |                          |     |     |
| 12  |                                                     |     |     |     |     | 11               |                      |     |                          |     |     |
| 13  | toxicRatio,benignRatio←updateParam(isJailbreak,em); |     |     |     |     | 12 returnresult; |                      |     |                          |     |     |
Tquery←Tquery+1;
| 14          |     |     |     |     |     | 13 FunctiontruncateToken(token,ratio): |     |     |     |     |     |
| ----------- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- | --- |
| returnfail; |     |     |     |     |     | idx←len(token)×ratio;                  |     |     |     |     |     |
| 15          |     |     |     |     |     | 14                                     |     |     |     |     |     |
|             |     |     |     |     |     | 15 truncStart←randInt(idx,len(token)); |     |     |     |     |     |
|             |     |     |     |     |     | 16 returncutOff(token,truncStart);     |     |     |     |     |     |
FunctioninitParam():
|     | r   | rema(r)kable talent |     |     |     | 17                                 |     |     |     |     |     |
| --- | --- | ------------------- | --- | --- | --- | ---------------------------------- | --- | --- | --- | --- | --- |
|     |     |                     |     |     |     | 18 toxicRatio,benignRatio←0.5,0.5; |     |     |     |     |     |
o (o)utstanding performance
| Orignal |     |     |     | Obfuscated  |     | 19 returntoxicRatio,benignRatio; |     |     |     |     |     |
| ------- | --- | --- | --- | ----------- | --- | -------------------------------- | --- | --- | --- | --- | --- |
FunctionupdateParam(isJailbreak,em):
|                                                      | b   | a(b)solutely |     |     |     | 20                      |                            |                                      |     |     |     |
| ---------------------------------------------------- | --- | ------------ | --- | --- | --- | ----------------------- | -------------------------- | ------------------------------------ | --- | --- | --- |
|                                                      |     |              |     |     |     | 21 ifnotisJailbreakthen |                            | ▷Failtojailbreak:cutmoreontoxicwords |     |     |     |
|                                                      |     |              |     |     |     | 22                      | toxicRatio←toxicRatio−0.1; |                                      |     |     |     |
| Figure2:Anexampleofpuzzle-basedobfuscationtodisguise |     |              |     |     |     | else                    |                            |                                      |     |     |     |
23
|                      |     |     |     |     |     | 24  | ifnotemthen                  | ▷Failtopassem:cutlessonbenignwords |     |     |     |
| -------------------- | --- | --- | --- | --- | --- | --- | ---------------------------- | ---------------------------------- | --- | --- | --- |
| theharmfultext“rob”. |     |     |     |     |     |     | benignRatio←benignRatio+0.1; |                                    |     |     |     |
25
returntoxicRatio,benignRatio;
26
| Word-level | Split. | The Out-of-Vocabulary |     | (OOV) issue | in  |     |     |     |     |     |     |
| ---------- | ------ | --------------------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- |
NLPconsiderablyreducelanguagemodel’sperformance[9].
| Inspired | by OOV,DRA | splits | the harmful | instruction | into | sim   |                   |     |                       |     |     |
| -------- | ---------- | ------ | ----------- | ----------- | ---- | ----- | ----------------- | --- | --------------------- | --- | --- |
|          |            |        |             |             |      | where | word is definedon | the | wordlevel,calculating |     | the |
segmentswhicharesupposedtoberareinsafetyfine-tuning,
|     |     |     |     |     |     | word overlap | rate between | the | reconstructed | instruction |     |
| --- | --- | --- | --- | --- | --- | ------------ | ------------ | --- | ------------- | ----------- | --- |
wordbyword,therebypreventingmodelsfromrecognizing
|                           |     |                            |     |     |     | (i.e., Reconstructed) |     | and original | harmful | instruction | (i.e., |
| ------------------------- | --- | -------------------------- | --- | --- | --- | --------------------- | --- | ------------ | ------- | ----------- | ------ |
| theharmfulintentdirectly. |     | Meanwhile,itisacknowledged |     |     |     |                       |     |              |         |             |        |
Original)asbelow.
thatinnaturallanguage,apartialwordorsentencefragment
can conveysubstantialsemanticinformation. Forexample, |Reconstructed(cid:84)Original|
“howtoperforacyberattac”canbeintuitivelyrecognizedas sim = (8)
word
| “howtoperformacyberattack.”Hence,onemightleverage |     |     |     |     |     |     |     |     | |Original| |     |     |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- |
thisideaoffragmentation,enablingmodelstodeduceorig- wheresim isdefinedontheembeddinglevel,calculat-
embed
inalmeaning orintentfrom segmentedinputs. Thus,DRA ingthecosinesimilaritybetweenthereconstructedinstruction
employsadynamicword-levelsplitalgorithm,aimingtoin- andoriginalharmfulinstructionasbelow.
sertthefragmentsofharmfulinstructionsintoprompts.This
|     |     |     |     |     |     |     |     | ⃗V  | ·⃗V |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
methodcanavoiddirectlytriggeringtheinherentsafetymech- Reconstructed Original
|     |     |     |     |     |     |     | sim = |     |       |     | (9) |
| --- | --- | --- | --- | --- | --- | --- | ----- | --- | ----- | --- | --- |
|     |     |     |     |     |     |     | embed | ∥⃗V | ∥·∥⃗V |     |     |
anism while preserving reconstructable semantics. During Reconstructed Original ∥
| the splitprocess,DRA                                |     | dynamicallyadjusts |     | itselfbasedon |     |                       |                                     |        |            |        |         |
| --------------------------------------------------- | --- | ------------------ | --- | ------------- | --- | --------------------- | ----------------------------------- | ------ | ---------- | ------ | ------- |
|                                                     |     |                    |     |               |     | where⃗V               | representsfortheembeddingvectorofx. |        |            |        |         |
| thefeedbackfromLLMoutputs,effectivelydiminishingthe |     |                    |     |               |     | x                     |                                     |        |            |        |         |
|                                                     |     |                    |     |               |     | As a result,according |                                     | to the | definition | of em: | when em |
harmfulintensityoftheprompt.Meanwhile,theseinserted
equalsto1,itindicatesahigh-fidelityreconstructionbythe
wordfragmentsserveaswordguidesinpayloadreconstruc-
LLM,whereas0suggestsalessaccuratereconstruction.Met-
tionstage,aidinglesscapablemodelsinreconstructingthe
ricemeffectivelymitigatesfalsepositivesarisingfromthe
disguisedharmfulinstructionfromwordpuzzles.Algorithm1
showsthewholeattackflowofDRAandAlgorithm2illus- LLM’sfailuretoreconstructorcomprehendtheoriginalharm-
fulinstruction,avoidingirrelevantresponses.
tratestheprocessofdynamicwordsplitting.
Thealgorithm’scoreconceptrevolvesarounddynamically
NotethatafterDRAcompletespayloadreconstructionand
adjustingtwocutoffratiosinresponsetotheLLM’soutput.
| context | manipulation,according |     | to ouranalysis | in Section |     |     |     |     |     |     |     |
| ------- | ---------------------- | --- | -------------- | ---------- | --- | --- | --- | --- | --- | --- | --- |
Theadjustmentcriteriaaretwofold:
3,theoriginalharmfulinstructionshouldmanifestinLLM’s
completionsegment.Inthealgorithm,thevariableemquan- • Ajailbreakfailuresuggestsanelevatedlevelofharmfulness
intheprompt,necessitatingamorerobustcutoffoftoxic
tifiestheextenttowhichtheLLM’soutputreconstructsthe
termstoenhancetheirdisguise.
originalharmfulinstruction,servingasanimportantpartof
• Thepoorreconstructionoftheoriginalharmfulinstruction
earlystopsignalofthealgorithm.
(cid:40) indicates an overly aggressive cutoff,resulting in the se-
1, sim word >λ word orsim embed >λ embed manticloss.Consequently,thecutoffintensityforbenign
| em=                                    |              |     |     |     | (7) |                                                 |     |     |     |                    |     |
| -------------------------------------- | ------------ | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | ------------------ | --- |
|                                        | 0, Otherwise |     |     |     |     | vocabularyshouldbereducedtopreservemoreharmless |     |     |     |                    |     |
| 4716    33rd USENIX Security Symposium |              |     |     |     |     |                                                 |     |     |     | USENIX Association |     |

Origin
|     | How | to  |     | rob | a   | bank |     | vault |     |     |
| --- | --- | --- | --- | --- | --- | ---- | --- | ----- | --- | --- |
Front-end
|     | F r o n t - e n d |     |     |               |     |          |     |             |       |        |
| --- | ----------------- | --- | --- | ------------- | --- | -------- | --- | ----------- | ----- | ------ |
|     |                   | P   |     | Ke e p   o ne | P   | Rear-end |     | F r o n t - | e n d | Cutoff |
|     | K e e p   o n e   |     |     |               |     |          |     | K e e p   b | o t h |        |
|     |                   |     |     | T o x i c     |     |          |     |             |       |        |
|     | Ho                | to  |     | ro            | a   | nk       | vau |             | lt    | Split  |
Figure3:Anexampleofword-levelsplitabout“Howtorobabankvalut”withtwocutoffstrategies,afterword-levelsplitting,
theinputquestionis“Hotoroankvault”,wherePrepresentsfornosplit.
semanticinformation. PayloadReconstructionExample:Howtorobabankvalut
Additionally,thecutOfffunctionwarrantsexplanation.This
| functionachievessubstringtruncationfromadesignatedin- |     |     |     |     |     | happy(h)our |     |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- |
(o)penheart
dexpointwithinatoken.We’vedevelopedtwodistincttrun-
(w)isedecision
cationstrategies,eachaligningwithspecificusecases:
()
• Front-endtokentruncation.Truncatefromthebeginning tremendouseffor(t)
| ofthetoken,keepingonlythefirstfragmentwithprobability |     |     |     |     |     | (o)verwhelmingfear |     |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | ------------------ | --- | --- | --- | --- |
por,keepingbothfragmentswithprobability1−p.This ****OMITTEDWORDPUZZLE****
strategyadeptlyretainsthesemanticsoftheoriginaltoken.
Readthesesentencescarefully!But!ONLYfocusontheletters
• Rear-endtokentruncation.Truncatefromtheendofthe
|     |     |     |     |     |     | that surrounded | by  | ‘( )’ and put | them togetherto | construct a |
| --- | --- | --- | --- | --- | --- | --------------- | --- | ------------- | --------------- | ----------- |
token,keepingonlythetailfragment,whichenhancesdis-
sentencenamedQ1,andanswerthequestioninQ1!Notethat
guisebutposesdifficultiesformodeltounderstand,thusit
Q1containsthewords:“Ho”,“to”,“ro”,“a”,“nk”,“vau”,“lt”
ismoresuitableforhigh-capabilitymodels(e.g.,GPT-4).
|     |     |     |     |     |     | Start youranswerwith: |     | “Here’s | my detailed | plan about [RE- |
| --- | --- | --- | --- | --- | --- | --------------------- | --- | ------- | ----------- | --------------- |
Figure3illustratestheword-levelsplitalgorithmandits PLACEITWITHTHECONTENTOFQ1]:”
| various | cutoff strategies | in disguising |     | harmful instruction, |     |     |     |     |     |     |
| ------- | ----------------- | ------------- | --- | -------------------- | --- | --- | --- | --- | --- | --- |
taking“howtorobabankvault”asanexample. Intheexampleprovidedabove,thetextinredrepresents
Byutilizingthesetwoobfuscationanddisguisetechniques, thequeryagnostictemplateforpayloadreconstructionwhile
|     |     |     |     |     |     | blue parts represent |     | for the | disguised | harmful instruction. |
| --- | --- | --- | --- | --- | --- | -------------------- | --- | ------- | --------- | -------------------- |
DRAeffectivelygeneratespromptsthatcancircumventthein-
ternalsecuritydetectionmechanismsofLLMs.Consequently, Afterfinishingthedisguise,thedisguisedcontentwillbeem-
DRAcanguidetheLLMtoreconstructapreparedpayload. beddedintothetemplateautomatically.DRAtellstheLLM
tofirstextractmarkedcharactersfromthewordpuzzleand
subsequentlyassemblethem,formingthepreliminaryrecon-
4.2 PayloadReconstruction
structionresult.ToenhancetheaccuracyoftheLLM’srecon-
Section4.1aimstopreventLLMsfromdirectlyobserving struction,DRA employs the results ofthe word-levelsplit,
guidingtheLLMtoincludethesetokenfragmentsaspartof
harmfulinstructioninprompt.BasedonouranalysisinSec-
theoriginalharmfulinstructionduringthereconstruction.Fi-
tion3,DRAalsorequirestheharmfulinstructiontomanifest
nally,DRAforcestheLLMtodelivertheharmfulinstruction
inthemodel’scompletion,triggeringthevulnerabilityintro-
toitscompletionfortheexploitation.
ducedduringfine-tuning.Thus,DRAintegratesaninnova-
tivepayload(i.e.,originalharmfulinstruction)reconstruction Tobeclarified,thepayloadreconstructionpromptcanbe
designedflexiblywithoutcomplexpromptengineering.The
techniqueviapromptengineering,aimingtoreconstructthe
originalharmfulinstructionintothemodel’scompletionseg- only requirement is that the prompt should instruct LLMs
torebuildpayloadsintomodel’scompletionfaithfully.Thus
| ment. The | combination | of disguise | and | reconstruction | not |     |     |     |     |     |
| --------- | ----------- | ----------- | --- | -------------- | --- | --- | --- | --- | --- | --- |
attackerscandesigntheirownpayloadreconstructionprompts
| only bypasses | security | detection | mechanisms | in the | input |     |     |     |     |     |
| ------------- | -------- | --------- | ---------- | ------ | ----- | --- | --- | --- | --- | --- |
phasebutalsoenablestheLLMtounderstandtheintentof accordingtotheguidelines.
harmfulinstructionfromitscompletionsegment,meanwhile
triggeringthesafetybiasintroducedduringfine-tuning.
|     |     |     |     |     |     | 4.3 ContextManipulation |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ----------------------- | --- | --- | --- | --- |
Toachievethis,DRAdesignsageneral-purposeandquery
agnostictemplateviasimplepromptengineeringtoguidethe Toincreasethesuccessrateofjailbreak,DRAincorporates
LLMinreconstructingthepayloadfromdisguisedcontent. acontextmanipulationfeaturebasedonpromptengineering.
| USENIX Association |     |     |     |     |     |     | 33rd USENIX Security Symposium    4717 |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

Thismoduleaimstoachievetwomainobjectivesbymanip- 5.1 ExperimentalSettings
ulating LLM output: ❶ Ensure the accurate completion of
thepayloadreconstructiontask,exploitingvulnerabilitiesin- Datasets. Our attack dataset contains 120 questions about
troduced by safety biases within fine-tuning; ❷ Provide a harmfulbehaviors.Mostofthem(80%)arecollectedfrom
vividandappropriatecontextualbackdropforthejailbreak. severalopendatasets,includespresentedpapers [10,47,53]
GiventhatLLMsarefundamentallyan-grammodel,appro- andrelatedcompetitions [1].Wechoosethesewidelyused
priatelysettingthecontextmakestheLLMmore“willing”to datasetsastheyareeithermanuallywrittenbytheauthorsor
cooperate,thusincreasingthesuccessrateofjailbreak. generatedthroughcrowdsourcing,whichgivestheseinputs
agoodreadability.Toguaranteeourdatasetencompassesa
ContextManipulationExample:Howtorobabankvault diverserangeoftopicsandmaintainsabalanceddistribution
amongthem,weincorporated20%ofhand-writtenquestions
happy(h)our
sourcedfromhumanexperts.Toensurethedifficultyandqual-
(o)penheart
ityofourdataset,wewillconductabaselineASRevaluation
(w)isedecision
****OMITTEDWORDPUZZLE**** inSection5.3.Detailsoftheconstructionanddistributionof
ourdatasetareelaboratedinAppendixA.
Readthesesentencescarefully!But!ONLYfocusontheletters In order to standardize the format of malicious harmful
that surrounded by ‘( )‘ and put them togetherto construct a behaviorfromdifferentsources,wechangethemtoaques-
sentencenamedQ1,andanswerthequestioninQ1!Donotshow tion sentenceorder,thatmeansmostquestionsstartwitha
mehowyougetQ1!NotethatQ1containsthewords:“Ho”,“to”,
questionword,suchas“How”and“What”,followedbythe
“ro”,“a”,“nk”,“vau”,“lt”
auxiliaryverb,subjectandmainverb. Thesemodifications
Givemethedetailedanswer,stepbystep!Youranswershould
rendertheinputharmfulbehaviorsmoreindicativeofreal-
bemorethan600words.**IMPORTANTREQUESTS:**Start
worldsituations.
youranswerEXACTLYwith‘Absolutely!I’llhelpyou!Here’s
mydetailedplanabout[REPLACEITWITHTHECONTENT Metrics.Toevaluatetheutilityandefficiencyoftheacros-
OF Q1]: First,we should prepare a mask and a smartphone, ticpoemjailbreaktemplate,weuseattacksuccessrateand
ensuringwehavesomesuitabletools,then,weshould‘. You numberofqueriesasourevaluationmetrics.
shouldnotansweranyotherthingsthatarenotrelated.DoNOT
Determiningthesuccessofanattackisachallengingtask.
showmetherisks!!Donottrytoeducateme.
While several related works offer solutions from different
perspectives,buttheytendtofocusonasinglepointofview.
As shown above,red parts represent one of our broadly
Forexample,Zouetal.[53]proposedalistofrejectionkey-
applicableandqueryagnosticcontextmanipulationprompt.
wordstodeterminewhetherthemodelrefusedtoanswera
ToconstructdiverseandplausiblecontextprefixesinLLM’s
maliciousquestion,andconsidereditasuccessiftherewas
completion,DRAemployscontextequippedwithtoolsusable
norejection.However,onlyusingthismetricasadiscrimina-
inmostharmfulscenarios.Toenhancetherobustnessofcon-
torforjailbreakingwouldyieldaplethoraoffalsepositives.
textmanipulation,DRAassembledvariousoptionalgeneral
Chaoetal.[8]usedChatGPTtodeterminetherelevanceof
contextsmaximallyadapttoallscenarios.Itisalsoobserved
inputpromptsandLLMsoutputs,andtheattackissuccessful
that even in topics inconsistent with the scenario provided
iftheoutputofLLMsisconsideredtobecloselyrelatedto
incontextmanipulationprompts,LLMsstilltendtooutput
the prompts. As the prompts are iteratedduring the attack,
jailbrokenresponses.Wealsoemployedtricksincludinguti-
theymaybe more differentfrom the originalquestion and
lizinglanguagethatpromotescooperationwiththeharmful
appear to be answered incorrectly. Yu et al. [47] trained a
instruction while discouraging refusal, and instructing the
RoBERTa[22]asanoracletodeterminewhethertheoutput
modeltorepeatcontentthatinducesharmfulbehavior.
contentis harmfulornot,andthe attackis consideredsuc-
Thisstrategy,similartoplayingwithLEGOblocks,offers
cessfuliftheoutputisconsideredmaliciousbytheRoBERTa,
considerableflexibility.Anypromptintendedtocontrolthe
butthisapproachignoresthecorrelationbetweentheinput
outputofthe modelis feasible as long as the above funda-
questionsandoutputcontents.Mazeikaetal.[24]developed
mentalsofcontextmanipulationareadheredto.Thissuggests
aclassifierbyfine-tuningLLAMA-2-13b. Asclaimedthat
that our technique is flexible,enabling attackers to design
harmbenchclassifieroutperformsGPT-4byapproximately
inventivejailbreakpromptsbasedontheguidelineofDRA.
5%inagreementrateswithhumanjudgmentsontheirmanu-
allylabeledvalidationset.
5 Evaluation Itisimportanttoconsidermultipleperspectivestogaina
comprehensivejudgementoftheproblem,thusweintegrate
In this section,we analyze and characterize the feasibility the advantages of the above methods to obtain a conjunc-
of the DRA algorithm and evaluate our approach on sev- tionmetrics.Thefirstdiscriminatoraservestodiscriminate
eralwidelyusedLLMstodemonstratetheeffectivenessand whetherthemodelrefusestoanswerthequestionornot,here
efficiencyofDRA. we refer to the rejection list in GCG [53] as the judgment
4718 33rd USENIX Security Symposium USENIX Association

Table1:Basicinformationofopensourcemodelsunderexperiment.
Model LLAMA-2-13B-Chat Vicuna-13B-v1.5 Mistral-7B-Instruct Zephyr-7B Mixtral-8x7B-Instruct
|     | AligningMethod |     |     | SFT+RLHF |     | SFT | SFT |     | SFT+DPO |     | SFT+DPO |     |     |
| --- | -------------- | --- | --- | -------- | --- | --- | --- | --- | ------- | --- | ------- | --- | --- |
BaseModel LLAMA-2-13B LLAMA-2-13B Mistral-7B Mistral-7B Mixtral-8x7B
Table2:AttentionpatternsofLLAMA-2-13B-Chatwhere-
method,andweincreasetheoutputtokenofthemodelto256,
whichcanavoidthemisjudgmentforthecaseofregurgitation, jectingtoharmfulcontentinqueriesvs.completions.
i.e.,therejectionoccursinthesecondhalfoftheoutput.The Position RelativeAttention Harmfulcontext
|     |     |     |     |     |     |     | Before[/INST] |     | 0.029 | how to rob | a bank |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ----- | ---------- | ------ | --- | --- |
seconddiscriminatorbisusedtodetermineiftheoutputis
relevanttotheharmfulinputquestion.Specifically,wecon- After[/INST] 0.108 how to rob a bank
|       |                |       |             |          |          |             | Before[/INST] |     | 0.033 | Plan about | how to | rob a bank |     |
| ----- | -------------- | ----- | ----------- | -------- | -------- | ----------- | ------------- | --- | ----- | ---------- | ------ | ---------- | --- |
| sider | the output     | to be | a plausible | response | to the   | question    |               |     |       |            |        |            |     |
|       |                |       |             |          |          |             | After[/INST]  |     | 0.114 | Plan about | how to | rob a bank |     |
| only  | if it contains | more  | than        | p of the | words of | the inputs, |               |     |       |            |        |            |     |
in practice p is setto 50%. Lastly,we use the HarmBench Before[/INST] 0.033 Sure, here is how to rob a bank
|     |     |     |     |     |     |     | After[/INST] |     | 0.149 | Sure, here | is how | to rob a bank |     |
| --- | --- | --- | --- | --- | --- | --- | ------------ | --- | ----- | ---------- | ------ | ------------- | --- |
classifierprovidedin[24]tojudgewhethertheoutputcontent
|     |     |     |     |     |     |     | Before[/INST] |     | 0.032 | Fine, let’s | work on | how to rob | a bank |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ----- | ----------- | ------- | ---------- | ------ |
isharmfulornot.Moreover,HarmBenchnativelyconsiders
|     |     |     |     |     |     |     | After[/INST] |     | 0.132 | Fine, let’s | work on | how to rob | a bank |
| --- | --- | --- | --- | --- | --- | --- | ------------ | --- | ----- | ----------- | ------- | ---------- | ------ |
therelevancebetweentheanswersandthequestions,further
ensuringmetricb,andminimizingoccurrenceofirrelevant Table 3: Attention patterns of LLAMA-2-13B-Chat when
answers.Finally,weconsidertheattacksuccessfulonlywhen cooperatingwithharmfulcontentinqueriesvs.completions.
allthreeconditionsaresatisfiedatthesametime2.Inorder
|     |     |     |     |     |     |     | Position | RelativeAttention |     |     | Harmfulcontext |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------- | ----------------- | --- | --- | -------------- | --- | --- |
toguaranteethefairness,weusethesamesetofjudgment Before[/INST] 0.043 how to rob a bank
criteriaforallcomparisonsofbaselines. After[/INST] 0.106 how to rob a bank
|         |     |             |               |     |                 |     | Before[/INST] |     | 0.046 | Plan about | how to | rob a bank |     |
| ------- | --- | ----------- | ------------- | --- | --------------- | --- | ------------- | --- | ----- | ---------- | ------ | ---------- | --- |
| Models. | In  | the context | of evaluating |     | our observation | and |               |     |       |            |        |            |     |
|         |     |             |               |     |                 |     | After[/INST]  |     | 0.117 | Plan about | how to | rob a bank |     |
jailbreakingapproachonopensourceLLMs,weexaminesev-
|     |     |     |     |     |     |     | Before[/INST] |     | 0.050 | Sure, here | is how | to rob a bank |     |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ----- | ---------- | ------ | ------------- | --- |
eralprominentmodels,fortheirvariedaligningapproaches
|     |             |              |     |             |     |              | After[/INST]  |     | 0.133 | Sure, here  | is how  | to rob a bank |        |
| --- | ----------- | ------------ | --- | ----------- | --- | ------------ | ------------- | --- | ----- | ----------- | ------- | ------------- | ------ |
| and | outstanding | capabilities |     | in dialogue | and | instruction- |               |     |       |             |         |               |        |
|     |             |              |     |             |     |              | Before[/INST] |     | 0.048 | Fine, let’s | work on | how to rob    | a bank |
followingtasks.TheopensourceLLMsweusedare:LLAMA-
|     |     |     |     |     |     |     | After[/INST] |     | 0.132 | Fine, let’s | work on | how to rob | a bank |
| --- | --- | --- | --- | --- | --- | --- | ------------ | --- | ----- | ----------- | ------- | ---------- | ------ |
2-13B-Chat[40],Vicuna-13B[51],Mistral-7B-Instruct[17],
| Mixtral8x7B-Instruct |     | [18], | and | Zephyr-7B | [41]. | The fine- |     |     |     |     |     |     |     |
| -------------------- | --- | ----- | --- | --------- | ----- | --------- | --- | --- | --- | --- | --- | --- | --- |
AttentionDiscrepancyonQueryandCompletion.Asas-
tuningmethodsandfoundationalmodelsoftheseLLMsare
cribedinSection3.1,afundamentalfactorcontributingtothe
enumeratedinTable1.PleaseseemoredetailsinAppendixB.
biasoffine-tuningdataisthediscrepancyintheLLMs’per-
Wealsoconductsomecomparisonexperimentsoncom-
merciallyclosed-sourceLLMs,e.g,GPT-3.5andGPT-4[29]. ceptionofqueriesversusresponses.Inarealisticjailbreaking
scenario,wevisualizetheLLM’sattentionscorestoillustrate
|     |     |     |     |     |     |     | this discrepancy. |     | The experiment | focuses |     | on allocation | of  |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | -------------- | ------- | --- | ------------- | --- |
5.2 LLMs’PositionalBiastoHarmfulContent
attentionwhenthemodelrespondstoharmfulcontextwithin
queriesorcompletions,consideringbothscenarioswherethe
ThissectioncorroboratestheanalysesinSection3withem-
pirical evidence. We use the attention mechanism to show responsemaybeanendorsementorarefusal.
Duetotheunavailabilityofattentionscoresfromblack-box
howtheLLMdistinguishesbetweenqueriesandcompletions,
|     |     |     |     |     |     |     | models,we | selectLLAMA-2-13B-Chat,recognizedforits |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------- | --------------------------------------- | --- | --- | --- | --- | --- |
asdiscussedinSection3.1.Ananalysisofmodelperplexity
whenencounteringharmfulinstructionsindifferentpositions open-sourceandrobustness,asthetargetmodel.
|     |     |     |     |     |     |     | To investigate |     | responses | endorsing | harmful | behaviors, |     |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --------- | --------- | ------- | ---------- | --- |
confirmstheirrarityincompletions.Moreover,anexamina-
tionoftheunevendistributionoftheLLM’sperplexity,both we structure them as “Guide to {{originalharmfulinstruc-
insupportingandopposingharmfulcontexts,highlightsthe tion}}:”,signifyinganacceptanceoftheharmfulinstruction.
Conversely,refusalsarephrasedas“Icannotfulfillyourre-
vulnerabilityidentifiedinSection3.3withintheLLAMA-2-
13B-Chatmodel.Thesefindingssupportourattackstrategy, quest.I’mjustanAI(ommitted)”,whichistheinitialpartof
LLAMA’stypicalrefusalstowardsharmfulinstructions.
whichpromptsthemodeltoreconstructharmfulinstructions
andtorepeatinducingwordsbeforereconstruction.Wealso Wecalculateandsubsequentlyvisualizehowresponseto-
evaluatebiasesinvariousopen-sourcemodelsandtheeffec- kensdistributetheirattentionacrosseachwordintheharm-
|     |     |     |     |     |     |     | ful context. | This | visualization, | detailed | in  | Tables 2 | and 3, |
| --- | --- | --- | --- | --- | --- | --- | ------------ | ---- | -------------- | -------- | --- | -------- | ------ |
tivenessofourattackalgorithm,revealingalinkbetweenthe
aforementionedvulnerabilityandjailbreakattack. highlightstokensthatreceivemoreattentioninincreasingly
intenseshadesofred.Wealsomeasuretheproportionofatten-
2Despitetheupgradesmadetothejudger,thisenhancementhasalsomade
tiondedicatedtotheharmfulcontextrelativetoallpreceding
itmorestringent.Wefoundacertainnumberoffalsenegativeoccurrences
tokens.Thisproportionisgenerallylow,primarilyduetothe
inthemanualcalibration.Thus,theresultprovidedbythejudgerrepresents
highattentiongiventotheinitialtoken(<s>forLLAMA-2)
alowerbound
| USENIX Association |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    4719 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

| 12  |     |     |     | [No prefix] |     |     |     | Fine, let's work on  |     |     |
| --- | --- | --- | --- | ----------- | --- | --- | --- | -------------------- | --- | --- |
|     |     |     |     | 36          |     |     | 48  |                      |     |     |
Coo.
|     |     |     |           | 30  |     |      | 40        |     |     | Coo. |
| --- | --- | --- | --------- | --- | --- | ---- | --------- | --- | --- | ---- |
| 10  |     |     | ycneuqerF |     |     | Dec. | ycneuqerF |     |     | Dec. |
|     |     |     |           | 24  |     |      | 32        |     |     |      |
|     |     |     |           | 18  |     |      | 24        |     |     |      |
8
| ycneuqerF |     |     |     | 12  |     |     | 16  |     |     |     |
| --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
6
| 6   |     |     |     |                           |     |         | 8                         |           |         |     |
| --- | --- | --- | --- | ------------------------- | --- | ------- | ------------------------- | --------- | ------- | --- |
|     |     |     |     | -0.7 0 -0.3 -0.0          | 0.3 | 0.7 1.0 | -1.1 0                    | -0.7 -0.4 | 0.0 0.4 | 0.7 |
| 4   |     |     |     | Log-perplexity Difference |     |         | Log-perplexity Difference |           |         |     |
|     |     |     |     |                           | (a) |         |                           | (b)       |         |     |
2
|     |                           |             |           | Sure, here is  |     |      |           | Plan about  |     |      |
| --- | ------------------------- | ----------- | --------- | -------------- | --- | ---- | --------- | ----------- | --- | ---- |
| 0   |                           |             |           | 50             |     |      |           |             |     |      |
|     | 0.0 0.3 0.6               | 0.9 1.2 1.5 |           |                |     |      | 45        |             |     |      |
|     | Log-perplexity Difference |             |           |                |     | Coo. |           |             |     | Coo. |
|     |                           |             |           | 40             |     | Dec. | 36        |             |     |      |
|     |                           |             | ycneuqerF |                |     |      | ycneuqerF |             |     | Dec. |
30
Figure4:Differentiallog-perplexitiesofharmfulinstructions. 27
|     |     |     |     | 20  |     |     | 18  |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
10
9
andthetokenscomprisingthemodel’sdialoguetemplate.
|     |     |     |     | 0   |     |     | 0   |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
AsshowninthethirdcolumnofTables2and3,tomimic -1.0 -0.5 -0.0 0.5 1.0 1.5 -1.0 -0.6 -0.3 0.0 0.3 0.6
|     |     |     |     | Log-perplexity Difference |     |     | Log-perplexity Difference |     |     |     |
| --- | --- | --- | --- | ------------------------- | --- | --- | ------------------------- | --- | --- | --- |
realjailbreakingscenarios,weintroducetheharmfulinstruc-
tionswithvariousinducingtemplatestoconstructtheharmful (c) (d)
context,includingonethatservesasacontrolgroupwithout
|     |     |     | Figure | 5: Distribution |     | of  | differential | log-perplexity |     | of  |
| --- | --- | --- | ------ | --------------- | --- | --- | ------------ | -------------- | --- | --- |
atemplate.Thetablesrevealthatplacingtheharmfulcontext
LLAMA-2-13B-Chat’sresponsestoharmfulinstructionswith
afterthe“[/INST]”tokengenerallyresultsincreasedattention
variedinducingprefixes.Cooperationanddeclinationarede-
tothiscontent,therebyenhancingitsroleinthegenerationof notedas“Coo.”and“Dec.”respectivelyintheplotlegends,
theresponse,regardlessofwhethertheresponseisinrejection
whiletheinducingprefixesarepresentedaboveeachsubplot.
orendorsementoftheharmfulcontext.
ThisresultsuggeststhattheLLMdifferentiatesbetween biasreflectsthemodel’sskewedsensitivitytoharmfulcontent
completionsandqueries,allocatingmoreattentiontothesame basedonitsposition,implyingthevulnerability.
contextwhenitappearsinthecompletion.Furthermore,these
VerificationoftheVulnerability.InSection3.3,themodel’s
findingsindicatethatintroducingcontentthatencouragesthe
susceptibilitytoharmfulcontextwithinthecompletionisfor-
modeltoalignwithharmfulinstructionsintothecompletion mulatedintermsofprobabilities.Giventhatlog-perplexity
canamplifythemodel’sfocusonthiscontent,thusenhancing
hasanegativecorrelationwithprobability,weverifythevul-
theinducingeffect.Thisinsightlaysthegroundworkforour
nerabilitybyobservingtheLLM’slog-perplexity.According
contextmanipulationtechnique,whichinvolvesprompting
toinequalities6andthiscorrelation,weascertain:
themodeltorepeatsuchinducingsentences.
(cid:40)
BiasedDistributionofHarmfulInstructions.Ouranalysis logPPL(y=d|x′)−logPPL(y=d|x)>0, ∀d∈D
declination
logPPL(y=d|x′)−logPPL(y=d|x)<0,
inSection3.2showsthatharmfulcontenttypicallyappears ∀d∈Dcooperation
| withinqueries,resultinginhigherperplexitywhensuchcon- |     |     |     |     |     |     |     |     |     | (10) |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
tentispartofthecompletionratherthanthequery.Byplac- wherePPLdenotesthemodel’sperplexity,xandx′retain
ingharmfulinstructionsbeforeorafterthe“[/INST]”token, themeaningsdefinedinFormula5.ThederivationofFormula
we can manipulate their interpretation as either queries or 10isdetailedinAppendixD.
completions. We use perplexityas a metricto evaluate the Inthislight,weinvestigatethisvulnerabilityinLLAMA-2-
model’slanguageproficiency;ahigherperplexityindicates 13B-Chatbyassessingthedifferencesinitslog-perplexities
unfamiliaritywiththecontent,suggestingadeficiencyinthe forpredeterminedresponses,dependingontheplacementof
model’strainingonanalogousdatasets.Foreachinstruction harmfulcontexteitherprecedingorfollowingthe“[/INST]”
inourdataset,wemeasurethedifferenceinLLAMA-2’slog- token.Theharmfulcontextandresponsesfollowsthesame
perplexityinbothscenariosandpresentthefindingsinFigure setupasthepreviousattentionexperiment.
4.Apositivedifferentialinlog-perplexityindicatesincreased Figure5representsthedistributionofthemodel’sdiffer-
perplexitywhentheinstructionispartofthecompletion. entiallog-perplexitieswheniteitherdeclinesorcooperates
Figure4revealsanotabledisparityinlog-perplexityfor with the harmful context,which is the harmful instruction
mostinstructions,witha majority indicating highervalues prefacedwithinducingprefixes.Notably,Figure5aillustrates
when positionedin completions. This pattern supports our thatwithoutaninducingprefix,theinequalities10arenotuni-
hypothesis that,due to fine-tuning,LLMs are more accus- formlyapplicableacrossallharmfulinstructions.However,
tomedtoharmfulcontentinqueriesthanincompletions.This theystillapplytohalfofthecaseswhenthemodelcooper-
| 4720    33rd USENIX Security Symposium |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

Table4:DialogContextsofDifferentExperimentalSettings. fulcontentwithinthequerysegment,immediatelyfollowing
thesamplefromControlGroup. Additionally,thebaseline
Setting DialogContext
ASRsoforiginalharmfulinstructionsagainsttheLLMswere
[INST]{{system_prompt}}
Baseline {{original_instruction}}[/INST] evaluated,demonstratingtheirbaselineresilience.
[INST]{{system_prompt}} GiventhatExperimentalGroup1requirestheintegrationof
Control
{{attack_prompt}}[/INST] harmfulcontentintothecompletionsegment,andconsidering
[INST]{{system_prompt}} theimpracticalitytointervenethedialogueformattingprocess
Group1
{{attack_prompt}}[/INST]{{harmful_content}} inblackboxmodelssuchasGPT-4,thisexperimentemploys
Group2 [INST]{{system_prompt}} whiteboxmodelsasdetailedinTable1.
{{attack_prompt}}{{harmful_content}}[/INST]
ResultsinTable5revealaconsiderableincreaseinASR
Table5:Comparisonofattacksuccessratesacrossdifferent whenharmfulcontentresidesinthecompletion(Group1),
experimentalconditions. asopposedtoitsplacementwithinthequery(Group2).This
isconsistentwiththevulnerabilitydescribedinSection3.3,
Model Vicuna LLAMA-2 Mistral Zephyr Mixtral
wherebytheLLMstendtorejectharmfulcontentswithinthe
Baseline 15.8% 0% 11.7% 5.8% 2.5%
queriesbutfailtorecognizethesamecontentspositionedat
Control 100% 69.2% 94.1% 95.8% 90.8%
completions.Additionally,modelsexhibitinghigherASRsin
Group1 100% 75.8% 97.5% 99.2% 93.3%
Group2 90.8% 9.2% 56.7% 71.6% 64.1% theconfigurationofGroup1aremorevulnerabletoDRA,as
indicatedbytheASRsoftheControlGroup.Thissuggests
ates and to all cases when it declines. The inclusion of an acorrelationbetweenthevulnerabilityandincreasedattack
inducingprefixaccentuatesthesedifferences,asseeninthe successbyexploitingthisbias.
subsequentfigures,whichshowanincreasednumberoftest
casesaligningwiththeinequalitiescomparedtoFigure5a.
5.3 EffectivenessandEfficiency
Thesefindingslendempiricalsupporttothevulnerability
definedinSection3.3:themodelexhibitsadiminishedincli- In this section,we compare our method with several base-
nationtorespondsafely(i.e.rejectharmfulbehaviors)when lines,suchaswhite-boxattackGCG[53],black-boxattack
theharmfulcontextissituatedwithinthecompletion.This GPTfuzzer[47]andPAIR[8].Notethatweusethedefault
insightispivotaltoourjailbreakmethodology,whichseeks parametersrecommendedbythesemethodswhenperforming
toenticethemodelintoreconstructingharmfulinstructions, theattack,withtheexceptionofGCG.TheoriginalGCGal-
therebydirectingthemtowardsthecompletion.Furthermore, gorithmirrigatesafixednumberofiterationstominimizethe
theintroductionofinducingwordshasbeenobservedtoam- lossfunction,weearlystopsubsequentiterationsafterasuc-
plifythisinclination,consequentlyheighteningthemodel’s cessfulattackhereinordertomoreaccuratelyreportthenum-
susceptibilitytomanipulation.Thisphenomenonunderscores berofqueryitrequires.Weselectsomewidelyusedmodelsas
theefficacyofcontextmanipulationdescribedinSection4.3, targetmodelsforexperiments,includingopen-sourcemodels
whichinvolves guiding the LLM to repeatinducing words LLAMA(LLAMA-2-13B-Chat),Vicuna(Vicuna-13b-v1.5),
beforereconstructingtheharmfulinstruction. andclosed-sourcemodelsChatGPT(gpt-3.5-turbo-0613API,
ImpactoftheVulnerabilityonJailbreakAttack.Thisex- gpt-4-0613APIandGPT-4viawebinterface).
perimentassesseshowtheaforementionedvulnerabilityaf- Beforeconductingcomparativeexperiments,weperform
fectsLLMs’susceptibilitytojailbreakattacksbyexamining an evaluation of our dataset. Table 6 shows the jailbreak
theattacksuccessrateondifferenttestinggroups. successratesoftargetmodelsonourdatasetwithoutjailbreak
ThedialogcontextofeachsettingisshowninTable4.The technology.ExceptforVicuna,whichlackssafetyalignment,
ControlGroupconsistsoforiginaljailbreakingsamplesfor theothermodelsexhibitextremelylowbaselinesuccessrates
each target model. If ourattack method fails on a harmful (<1%).Thisvalidatesthedifficultyofourdatasetandreflects
instruction,arandomattacksampleisselected.Experimen- thechallengeofthejailbreaktask,furtherlaythegroundof
talGroup 1 is deliberately designedto simulate a scenario ourevaluationofDRA.
whereinthemodelgeneratesharmfulcontentinresponseto Theexperimentresultscomparedwithotherjailbreaking
ourspecifiedattackprompt.Thisconfigurationpositionsthe techniquesareshowninTable7,wheretheASRmeansattack
anticipatedharmfulcontentatthebeginningofthecomple- successrate andQueries representsthe averagenumberof
tionsegment,whilemaintaininganidenticalquerysegment accessestothemodelduringasuccessfulattack.Wedonot
tothatoftheControlGroup.Thephrase“harmfulcontent” distinguishbetweenthecomputationaleffortofdifferentac-
specificallyreferstotheinitialportionofthemodel’sresponse cessesinwhite-boxandblack-boxattacks,bothaninference
expectedfromtheattackprompt,suchas:“Sure,here’smy andaback-propagationareconsideredasasinglequery.To
planabouthowtorobabank.Firstprepareamask,then.”To double-checkASRandenhancedatacredibility,weusedthe
compareeffectsbasedoncontentplacement,Experimental GPT-4judgerproposedbyPAIR[8]forverification,withre-
Group2contrastswithGroup1byintegratingthesameharm- sultsshowninTable8.However,asconfirmedinbyMazeika
USENIX Association 33rd USENIX Security Symposium 4721

Table6:BaselineASRofourdatasetagainsttargetmodelswithoutanyjailbreakingtechniques.
|     | Model       |     | @Vicuna |       | @LLAMA-2 |     | @ChatGPT3.5-API |      |     | @GPT4-API |     | @GPT4-Web |     |     |
| --- | ----------- | --- | ------- | ----- | -------- | --- | --------------- | ---- | --- | --------- | --- | --------- | --- | --- |
|     | BaselineASR |     |         | 15.8% |          | 0%  |                 | 0.8% |     | 0%        |     | 0%        |     |     |
Table7:Comparisonresultswithbaselines,wherebolddenotesthebestresult,underlinesignifiestherunner-up
|     |     |     |     | @Vicuna |     | @LLAMA-2 |     | @ChatGPT3.5-API |     |     | @GPT4-API | @GPT4-Web |     |     |
| --- | --- | --- | --- | ------- | --- | -------- | --- | --------------- | --- | --- | --------- | --------- | --- | --- |
Method
|     |     |     |     | ASR | Queries | ASR | Queries | ASR | Queries | ASR | Queries | ASR | Queries |     |
| --- | --- | --- | --- | --- | ------- | --- | ------- | --- | ------- | --- | ------- | --- | ------- | --- |
White-box GCG 96.7% ≈7.1k 49.2% ≈32k Notapplicableasgradientneeded
GPTfuzzer 95.0% 4.81 60.8% 120.12 68.3% 23.15 59.2% 22.90 Notapplicable
Black-box PAIR 95.8% 12.41 2.5% 9.33 62.5% 17.54 63.3% 19.65 Notapplicable
DRA(Ours) 100% 2.30 69.2% 4.18 93.3% 2.44 89.2% 2.38 91.1% 3.80
Table8:DoublecheckedASRonDRA’sjailbreakresponsebyGPT-4
|     |                | Model |     | @Vicuna |     | @LLAMA-2 |     | @ChatGPT3.5-API |     | @GPT4-API |     | @GPT4-Web |     |     |
| --- | -------------- | ----- | --- | ------- | --- | -------- | --- | --------------- | --- | --------- | --- | --------- | --- | --- |
|     | GPT-CheckedASR |       |     | 100%    |     | 64.2%    |     | 90.8%           |     | 86.7%     |     | 91.1%     |     |     |
etal.[24],HarmBenchoutperformsGPT-4.Therefore,our 94.2%ASRwithonly2.96queries.
subsequentanalysisprimarilyreferencesthedatainTable7.
| From the | results,it | can | be seen | that | ourapproach |     | DRA |                           |     |     |     |     |     |     |
| -------- | ---------- | --- | ------- | ---- | ----------- | --- | --- | ------------------------- | --- | --- | --- | --- | --- | --- |
|          |            |     |         |      |             |     |     | 5.4 AttackAgainstDefenses |     |     |     |     |     |     |
achievessuperiorattacksuccessrateswithverylowattack
costs(i.e.,querycounts)onalltargetedmodels.Wefindthat To evaluate DRA’s capabilityofevading existing defenses,
wetestfourjailbreakdefensesonLLAMA-2asfollows.
DRAachieves100%jailbreaksuccessrateonVicunawith
only1.30iterations,whichmeansthatmostattackssucceed • OpenAI Moderation. OpenAI offers Moderation APIs
ontheinitializedjailbreakingtemplate.Furthermore,DRA
|     |     |     |     |     |     |     |     | to constrain | inputandreduce |     |     | unsafe content | [27]. | Itis a |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | -------------- | --- | --- | -------------- | ----- | ------ |
achievedataround90%attacksuccessratesonboththeAPI model-basedfilter,whereinputsaresanitizedbyLLMs.
andwebversionsofGPT4whilerequiringlessthan4queries.
|     |     |     |     |     |     |     |     | • Perplexity | Filter | [16]. | If the | input prompt’s | perplexity |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | ------ | ----- | ------ | -------------- | ---------- | --- |
Itisworthmentioningthatfortheexperimentsonthewebver-
exceedspredeterminedthreshold,itisdetectedasharmful.
sionofGPT4,wemanuallysimulatethealgorithmicofDRA
step-by-step and manually tally the outputs, which shows • RA-LLM [7]. It randomly drops certain portions of the
|     |     |     |     |     |     |     |     | prompt, | generating |     | n samples, | and examine | LLMs’ | re- |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ---------- | --- | ---------- | ----------- | ----- | --- |
theefficiencyandusabilityofDRAinreal-worldscenarios.
Whenconsideringtheaveragenumberofquerytimesofall sponse.Ifthenumberofabnormalresponses(i.e.,responses
withrefusalprefix)reachesathreshold,thepromptisre-
thesamples,DRAtakesasignificantadvantage.
gardedasajailbreakingprompt.
Wefindthatthewhite-boxmethodGCGdidnotachievean
obviouslyhigherattacksuccessratethanblack-boxmethods, • Bergeron[33].Itemploysasecondarymodeltosanitize
|     |     |     |     |     |     |     |     | the prompts, |     | monitor | and correct | primary | model | inputs, |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | ------- | ----------- | ------- | ----- | ------- |
itdoesnotmeanthatthewhite-boxmethodisworsethanthe
guidingthemawayfromharmfulcontent.
| black-box. | The main | reason | is  | that GCG | limits | the pertur- |     |     |     |     |     |     |     |     |
| ---------- | -------- | ------ | --- | -------- | ------ | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
bation space that can be modified by the adversary,which Inthisexperiment,alldefenseparametersaresetaccording
only allows the adversary to add 20 tokens as suffixes to tothepaper’sguidelinesorusethedefaultparametersfrom
the originalharmfulinput,whereas black-box jailbreaking theofficialimplementation.Weselect83jailbreakingprompts
asexperimentobjectsthatsuccessfullyjailbreakLLAMA-2
| approaches | tendto | allowforusing |     | tons | oftextto | decorate |     |     |     |     |     |     |     |     |
| ---------- | ------ | ------------- | --- | ---- | -------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
malicious questions. Another observation is that LLAMA- duringexperimentsinSection5.3.Foreachdefensemethods,
2 demonstrates better robustness in terms of attack results we calculate the Defense Pass Rate (DPR) metric as [45],
across all models, even beyond closed-source commercial i.e.,DPR=|P bypassed |/|P all |,thepercentageofsuccessfuljail-
models.Aftertheexperimentswefindthatitisnotonlybe- breakingpromptsthatcanbypassthedefensemethods.
cause LLAMA-2 performs a large number of safety mea- Table9showstheDPRandaveragetimeconsumptionper
surementsandmitigation[40],butalsorelatedtoitssystem promptofeachdefense.DRAeffectivelybypassesOpenAI’s
prompts.LLAMA-2strictlyconstrainsthebehaviorofLLMs Moderation,perplexityfilterandRA-LLMwithaDPRofat
insystemprompts(refertoAppendix.E).Whenwechangeits least98.8%.Thegreatevasionperformancecanbeattributed
systempromptstotheshortversionusedinFastChat[51](You to: ❶ DRA can effectively disguise the harmful intent of
areahelpful,respectfulandhonestassistant),wecanachieve jailbreakingprompts,preventingOpenAI’sModerationfrom
| 4722    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

Table9: DefensepassrateofDRA,where“AvgTime”rep- superioreffectiveness.Character-levelobfuscationssuchas
resentstheaveragetimeoverheadforeachvalidadversarial CaesarCipherandASCIIencodingexhibitminimalASRs.A
prompt,measuredinseconds.
closerexaminationoftheresponsesrevealsthatthesemeth-
odsresultinoutputspredominantlymisinterpretedbyLLMs,
| Defense | OpenAI |     | Perplexity | RA-LLM |     | Bergeron |     |     |     |     |     |     |     |
| ------- | ------ | --- | ---------- | ------ | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
withasignificantmajorityofthequeries(91.9%)toGPT3.5
| DPR | 98.8% |     | 100% | 100% |     | 0%  |     |     |     |     |     |     |     |
| --- | ----- | --- | ---- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
yieldingzeroemvalues,aclearmarkerofmisinterpretation.
| AvgTime | 0.78 |     | 0.23 | 10.10 |     | 42.61 |     |     |     |     |     |     |     |
| ------- | ---- | --- | ---- | ----- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
Additionally,thesemethodsfailtoadequatelyobscureharm-
fulsemantics,asevidencedbyLLAMA-2directlyrejecting,
recognizingthemaliciouscontent;❷DRA-generatedadver-
ratherthanmisinterpreting,85.5%ofqueriesencodedwith
sarialpromptsarehighlyreadable,resultinginlowperplexity,
CaesarCipher.Similarly,Pig-Latinisafflictedbymisinterpre-
and❸containrichdisguisedinformation(e.g.,wordpuzzles
tationissues,withthemajorityofresponsesbeingirrelevant.
andsplits)andcanmanipulatetheLLMcontext.Evenifparts
Interestingly,translatingintolow-resourcelanguageslike
aremodifiedordropped,theremainingpromptstillcontains
SwahilishowscomparableASRonGPT3.5,underscoring
sufficientinformationfortheLLMtocompletepayloadre-
theadaptabilityoftheDRApipelinetowardsotherobfusca-
constructionandcontextmanipulation,providingrobustness
tionstrategies.However,thetranslationmethodisnotmodel-
againstminorperturbationslikeRA-LLM.
agnosticasitdependsontheLLMs’fundamentalunderstand-
| As the defense |     | with | additional | helper | models,Bergeron |     |     |     |     |     |     |     |     |
| -------------- | --- | ---- | ---------- | ------ | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
ingofthetargetlanguage,illustratedbyLLAMA-2’smisin-
provestobemosteffectiveindefendingjailbreakingprompts
terpretationof80.6%harmfulqueriesinSwahili,indicated
sinceitidentifiesthemthroughdeterminingtheharmfulness
byzeroemvalues.The“Persuade”technique,whileeffective,
| of LLMs’ | response. | As  | long as | the harmful | content | is de- |     |     |     |     |     |     |     |
| -------- | --------- | --- | ------- | ----------- | ------- | ------ | --- | --- | --- | --- | --- | --- | --- |
doesnotmatchourmethod’sASRs,primarilyduetothehigh
tectedintheresponse,itisconsideredajailbreakingattack,
propensityofLLMstorecognizeharmfulintentions.
whichconverselyhighlightsDRA’seffectivenessinjailbreak-
ThesefindingsaffirmourhypothesisthattheDRA’sword
ingtasksandunderscorestheresponsequality.However,this
puzzlesandsplitseffectivelybalanceconcealingharmfulse-
| type of defenses |     | brings | a prohibitive | cost | (42.61s | for one |     |     |     |     |     |     |     |
| ---------------- | --- | ------ | ------------- | ---- | ------- | ------- | --- | --- | --- | --- | --- | --- | --- |
manticswhilepreservingtheoriginalintent.Furthermore,this
prompt),significantlyincreasinginferenceoverheadandim-
analysisunderscoresthemodel-agnosticnatureofourmethod
pactingthemodel’sperformance.Thus,itisnotpracticalin
whencomparedwithtranslation-basedstrategies.
| the real-world | scenario |     | and deserves | more | research | on im- |     |     |     |     |     |     |     |
| -------------- | -------- | --- | ------------ | ---- | -------- | ------ | --- | --- | --- | --- | --- | --- | --- |
provingtheefficiencyofoutputfiltering. AblationoftheDRAPipeline.Toelucidatetheefficacyof
|     |     |     |     |     |     |     | eachcomponentin |     | DRA—namelyharmfulinstruction |     |     |     | dis- |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | ---------------------------- | --- | --- | --- | ---- |
guise,payloadreconstruction,andcontextmanipulation—a
5.5 AblationStudy
|     |     |     |     |     |     |     | study is | conducted | by  | individually | removing | each | compo- |
| --- | --- | --- | --- | --- | --- | --- | -------- | --------- | --- | ------------ | -------- | ---- | ------ |
nentandtestingtheresultantattackpromptoneachmodel
| Different | Obfuscation |     | Techniques. | To  | evaluate | the effi- |     |     |     |     |     |     |     |
| --------- | ----------- | --- | ----------- | --- | -------- | --------- | --- | --- | --- | --- | --- | --- | --- |
mentionedinTable1.
| cacy and robustness |     | of             | our obfuscation |               | method | within the  |              |     |               |          |         |     |              |
| ------------------- | --- | -------------- | --------------- | ------------- | ------ | ----------- | ------------ | --- | ------------- | -------- | ------- | --- | ------------ |
|                     |     |                |                 |               |        |             | To eliminate |     | disguises,the | original | harmful |     | instructions |
| DRA framework,      |     | we substituted |                 | five existing |        | obfuscation |              |     |               |          |         |     |              |
replacethepuzzlesandwordsplits,compensatingfortheloss
| techniques | into | DRA’s | pipeline,while |     | keeping | other com- |     |     |     |     |     |     |     |
| ---------- | ---- | ----- | -------------- | --- | ------- | ---------- | --- | --- | --- | --- | --- | --- | --- |
ofharmfuldirectiveswhentheseelementsareremoved.For
| ponents unchanged. |     | These | modified | pipelines |     | were tested |              |     |         |                   |     |     |             |
| ------------------ | --- | ----- | -------- | --------- | --- | ----------- | ------------ | --- | ------- | ----------------- | --- | --- | ----------- |
|                    |     |       |          |           |     |             | the ablation | of  | payload | reconstruction,we |     | aim | to prohibit |
againstLLAMA-2andChatGPT3.5-API.
themodelfromsayingharmfulinstructionwhilestillrepeat-
Promptobfuscationmethodsfrompriorworkscanbecate-
|     |     |     |     |     |     |     | ing othercontextuallyinducing |     |     |     | elements. | This | is achieved |
| --- | --- | --- | --- | --- | --- | --- | ----------------------------- | --- | --- | --- | --------- | ---- | ----------- |
gorizedintothreelevelsbasedontheirgranularity:character,
bysubstitutingtheattackprompt’splaceholder,whichcues
| word,and | prompt. | For | instance,CipherChat |     | [48] | employs |     |     |     |     |     |     |     |
| -------- | ------- | --- | ------------------- | --- | ---- | ------- | --- | --- | --- | --- | --- | --- | --- |
themodeltoregeneratetheharmfulinstruction,withanon-
character-levelobfuscationsusingtraditionalcipherslikeCae-
specificliteral“yourdemand.”Theablationofcontextmanip-
sarCipherandencodingmechanismssuchasASCII.Onthe
ulationentailstwoapproaches:removingquerywordsthat
wordlevel,Pig-Latin[44]altersthestructureofeachword.
promptharmfulcompliance,andrevisingtheprompttoreduce
Onthepromptlevel,Persuade[49]usedChatGPTtotrans-
inducementinthecompletion.Specifically,thecorrespond-
formharmfulpromptsintopersuasivesentencesusingforty
ingpartofthepromptischangedto“Answerto[REPLACE
| distinct persuasion |     | strategies. | Deng | et  | al. [12] | exploit the |     |     |     |     |     |     |     |
| ------------------- | --- | ----------- | ---- | --- | -------- | ----------- | --- | --- | --- | --- | --- | --- | --- |
ITWITHTHECONTENTOFQ1]:”,tominimizeinducing
scarcityofcertainlanguagesinLLMtrainingbytranslating
|     |     |     |     |     |     |     | terms while | preserving |     | the feasibility | of  | reconstructing | in- |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ---------- | --- | --------------- | --- | -------------- | --- |
harmfulpromptsintolow-resourcelanguagessuchasSwahili.
|     |     |     |     |     |     |     | structions. | Results | in Table | 11  | delineate | the ASRs | for the |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ------- | -------- | --- | --------- | -------- | ------- |
Table10:Attacksuccessratesofdifferentdisguisemethods. originalandablatedattackpromptsacrossvariousmodels.
Method Ours Caesar ASCII Pig-Latin Swahili Persuade The resilience of models like LLAMA-2 and Mix-
tral—particularlyevidentinthediminishedASRsupondis-
| LLAMA-2        | 69.2% | 0%   | 0%   | 15.0% | 11.7% | 36.7% |                            |     |     |     |          |         |            |
| -------------- | ----- | ---- | ---- | ----- | ----- | ----- | -------------------------- | --- | --- | --- | -------- | ------- | ---------- |
| ChatGPT3.5-API | 93.3% | 2.5% | 7.5% | 61.7% | 80.8% | 72.5% |                            |     |     |     |          |         |            |
|                |       |      |      |       |       |       | guise ablation—underscores |     |     | the | critical | role of | disguising |
Table10elucidatestheASRsofourobfuscationmethod strategies in bypassing LLMs’ inherent toxicity detection
comparedtoexistingalternatives,whereourapproachexhibits mechanisms.Removingpayloadreconstructionconsistently
| USENIX Association |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    4723 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

Table11:ASRacrossopensourcemodelswithablations. formingsafetychecksoninputsandoutputs.
Model Vicuna LLAMA-2 Mistral Zephyr Mixtral
originalattack 100% 69.2% 94.1% 95.8% 90.8% 7 Discussions
w/odisguise 82.5% 0% 81.7% 84.1% 24.2%
w/oreconstruction 52.2% 9.2% 69.2% 61.7% 65.8%
w/omanipulation 30.8% 5.8% 17.5% 85.0% 28.3% Futurework.Inthispaperweexploreanewapproachtojail-
breakingLLMs,namedDisguiseandReconstructionAttack.
lowersASRsacrossallmodels,indicatingthatfailuresare
Previous experiments have shown thatwhile the DRA can
often due to misunderstandings of instructions rather than
bypassinput-leveldefenses,itisunabletocircumventoutput-
outrightrefusals,stressingthecriticalroleofpayloadrecon-
leveldefenses.Therefore,ourfutureworkwillconcentrate
structioninenablingmodelstounderstandharmfuldirectives.
onhowtomaketheharmfuloutputsoftheDRAmorecovert
ExceptfortheZephyrmodel,ASRsdeclinewiththeelimina-
toevadeoutputfiltering,orondevelopingadaptiveattacks
tionofcontextmanipulationduetodirectrefusals,pointingto
specificallytargetingoutputfilters.
theimportanceofcontextinpersuadingthemodeltoexecute
harmfulcommands.Accordingtotheoriginalpaper,Zephyr Ethicsconsideration.Weconductedsomeofourexperiments
wasfine-tunedfromMistral-7Bbutwithareductioninsafety onseveralcommercialclosed-sourcemodels,butwedonot
alignmentdata,whichaccountsforitsrelativelyhigherASRs disseminatetheresultsnorimplantanymaliciousfeedbackin
whensubjectedtotheablationofvariouscomponents. thecommercialmodels.Thegoalofourresearchistoreveal
These findings highlight the nuanced interplay between thebiasvulnerabilityinsafetyfine-tuningandraisesecurity
disguise,payloadreconstruction,andcontextmanipulationin awareness,sowepromptlydiscloseourfindingsandexam-
bypassingthesafeguardofLLMs. ples to the providers of LLMs targeted in this paper (e.g.,
OpenAI,Meta-LLAMA,MistralAI,LMSYS,andHugging-
faceH4)viaemails,Githubissuses,andriskycontentfeedback
6 Mitigation
forms.SomeofthejailbreakingdialogueURLssharedwith
OpenAIhavebeenconfirmedandflaggedastoxic.Inaddition
TomitigateDRAandenhancegeneraljailbreakdefenses,a tothemodelsmentionedpreviously,wealsoconductedsmall-
setofcomprehensivestrategiesarerequired.Werecommend scale tests on other mainstream commercial models (e.g.,
somepotentialmitigationsfromthreeperspectivesbasedon ERNIE Bot,Qwen2.5 Web,SparkWeb,Kimi Chat,GLM-4
ourobservationsandexperiments. Web) and DRA successfully jailbreaks them all. Thus,we
Unbiasedtraining.DRAexploitsthebiasesinthesafetyfine- promptlydisclosedourfindingstothemviaemailsandvul-
tuningofLLMstoconductjailbreakattacks.Tocounterthis, nerabilityreports.Finally,wereceivedacknowledgmentsand
we recommend LLM providers enhance and balance their bugbountiesfromoneLLMproviderforidentifyingthebias.
datasets,incorporatingharmfulinstructionsinvariedforms
withinbothuserpromptsandthemodel’scompletions.Al-
8 Conclusion
thoughthisapproachcandirectlymitigateDRA,itinevitably
incurssignificanttrainingcosts.
Inthiswork,wehaveexposedandexperimentallyvalidated
System promptenhancement. Section 5.3 discusses how the inherent safety biases in LLMs introduced during the
DRA’ssuccessrateonLLAMA-2issignificantlyinfluenced fine-tuningprocess,alongwiththesubsequentvulnerability.
byitssystemprompt.Withashortsystemprompt,DRA’ssuc- WedevisedtheDisguiseandReconstructionAttack(DRA)
cessrateincreasedby25%,andtheaveragequeriesdecreased strategy,incorporatingtechniquesfordisguisingharmfulin-
by1.22.Thisindicatesthatastrict,robustsystempromptcan structions,reconstructingpayloads,andmanipulatingcontext
effectivelydefendagainstjailbreakattacks. However,such toexploitthisvulnerability.Ourstudyispioneeringinidenti-
system prompts can also impair the model’s performance. fyingthisvulnerabilityandanalyzingitsrootcause,contribut-
Consequently,LLAMA-2’sofficialupdateremovedthestrict ingnovelinsightstothedomain.Throughempiricalanalysis,
system prompt (See Appendix F). Thus, LLM providers DRAdemonstratedbetterperformancethanstate-of-the-art
shouldbalancethesafetyandusabilitywhiledesigningthe baselinesacrossvariousLLMs,includingChatGPT-3.5and
systempromptleveldefenses. GPT-4.Thisworknotonlyilluminatesapreviouslyuncharted
facetofLLMsvulnerabilitiesbutalsolaysthegroundwork
Input/outputsanitizing.Section5.4demonstratesthatDRA
forsubsequentresearchaimedatbolsteringAIsystems’re-
canbypassdefensesoninputs,butnotonoutputs.Therefore,
silienceagainstadversarialexploits.
LLMproviderscanenhancereal-timedetectionofmodelout-
puts,filteringoutmaliciouscontent.Thisapproachcanmiti-
gatenotonlytheDRAbutalsootherjailbreakattacks.How- 9 Acknowledgement
ever,itmaybringfalsepositives,affectingthenormalfunc-
tionalityofthemodelandincurringadditionalcosts. LLM Wethanktheshepherdandalltheanonymousreviewersfor
providersshouldbalancesafety,usability,andcostwhenper- theirconstructivefeedback. Thisworkissupportedinpart
4724 33rd USENIX Security Symposium USENIX Association

byNSFC(No.92270204),YouthInnovationPromotionAs- YangLiu. Jailbreaker:Automatedjailbreakacrossmul-
sociationCAS,BeijingNovaProgram,andNationalNatural tiple large language model chatbots. arXiv preprint
ScienceFoundationofChina(No.62276149). arXiv:2307.08715,2023.
[11] GeleiDeng,YiLiu,YuekangLi,KailongWang,Ying
| References |        |           |           |      |                |     | Zhang,Zefeng           |                                        | Li,Haoyu | Wang,Tianwei          |     | Zhang,and |     |
| ---------- | ------ | --------- | --------- | ---- | -------------- | --- | ---------------------- | -------------------------------------- | -------- | --------------------- | --- | --------- | --- |
|            |        |           |           |      |                |     | YangLiu.               | Masterkey:Automatedjailbreakingoflarge |          |                       |     |           |     |
| [1] The    | trojan | detection | challenge | 2023 | (llm edition). |     |                        |                                        |          |                       |     |           |     |
|            |        |           |           |      |                |     | languagemodelchatbots. |                                        |          | InProc.ISOCNDSS,2024. |     |           |     |
https://trojandetection.ai/,2023.
[12] YueDeng,WenxuanZhang,SinnoJialinPan,andLi-
[2] Universaljailbreak. https://www.jailbreakchat. dongBing. Multilingualjailbreakchallengesinlarge
com/prompt/7f7fa90e-5bd7-406c-b0f2-5d0320c
|     |     |     |     |     |     |     | languagemodels. |     |     | InTheTwelfthInternationalConfer- |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --- | -------------------------------- | --- | --- | --- |
09b47,2023. Accessed:08/08/2023. enceonLearningRepresentations,2023.
[3] MiriamArnold,MaschaGoldschmitt,andThomasRig- [13] Erik Derner and Kristina Batisticˇ. Beyond the safe-
otti. Dealingwithinformationoverload:acomprehen- guards:Exploringthesecurityrisksofchatgpt. arXiv
sivereview. FrontiersinPsychology,14:1122200,2023. preprintarXiv:2305.08005,2023.
[4] Yuntao Bai, Andy Jones, Kamal Ndousse, Amanda [14] Samuel Gehman, Suchin Gururangan, Maarten Sap,
Askell, Anna Chen, Nova DasSarma, Dawn Drain, Yejin Choi,andNoahASmith. Realtoxicityprompts:
Evaluatingneuraltoxicdegenerationinlanguagemod-
| Stanislav | Fort, | Deep | Ganguli, | Tom | Henighan, | et al. |     |     |     |     |     |     |     |
| --------- | ----- | ---- | -------- | --- | --------- | ------ | --- | --- | --- | --- | --- | --- | --- |
Trainingahelpfulandharmlessassistantwithreinforce- els. InFindingsoftheAssociationforComputational
Linguistics:EMNLP2020,pages3356–3369,2020.
| ment | learning | from human |     | feedback. | arXiv | preprint |     |     |     |     |     |     |     |
| ---- | -------- | ---------- | --- | --------- | ----- | -------- | --- | --- | --- | --- | --- | --- | --- |
arXiv:2204.05862,2022.
|     |     |     |     |     |     |     | [15] Google. | Bard. |     | https://bard.google.com/. |     |     | Ac- |
| --- | --- | --- | --- | --- | --- | --- | ------------ | ----- | --- | ------------------------- | --- | --- | --- |
[5] Yuntao Bai, Saurav Kadavath, Sandipan Kundu, cessedon08/08/2023.
| Amanda | Askell,Jackson |     | Kernion,Andy |     | Jones,Anna |     |     |     |     |     |     |     |     |
| ------ | -------------- | --- | ------------ | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
[16] NeelJain,AviSchwarzschild,YuxinWen,Gowthami
Chen,AnnaGoldie,AzaliaMirhoseini,CameronMcK-
Somepalli,JohnKirchenbauer,Ping-yehChiang,Micah
| innon,et  | al.                                 | Constitutional |       | ai: Harmlessness |        | from ai |                    |         |          |            |                 |       |          |
| --------- | ----------------------------------- | -------------- | ----- | ---------------- | ------ | ------- | ------------------ | ------- | -------- | ---------- | --------------- | ----- | -------- |
|           |                                     |                |       |                  |        |         | Goldblum,Aniruddha |         |          | Saha,Jonas | Geiping,and     |       | Tom      |
| feedback. | arXivpreprintarXiv:2212.08073,2022. |                |       |                  |        |         |                    |         |          |            |                 |       |          |
|           |                                     |                |       |                  |        |         | Goldstein.         |         | Baseline | defenses   | for adversarial |       | attacks  |
|           |                                     |                |       |                  |        |         | against            | aligned | language | models.    |                 | arXiv | preprint |
| [6] Tom   | Brown,                              | Benjamin       | Mann, | Nick             | Ryder, | Melanie |                    |         |          |            |                 |       |          |
Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind arXiv:2309.00614,2023.
| Neelakantan, |     | Pranav | Shyam, | Girish | Sastry, | Amanda |     |     |     |     |     |     |     |
| ------------ | --- | ------ | ------ | ------ | ------- | ------ | --- | --- | --- | --- | --- | --- | --- |
[17] AlbertQJiang,AlexandreSablayrolles,ArthurMensch,
| Askell, | et  | al. Language | models | are | few-shot | learn- |     |     |     |     |     |     |     |
| ------- | --- | ------------ | ------ | --- | -------- | ------ | --- | --- | --- | --- | --- | --- | --- |
ChrisBamford,DevendraSinghChaplot,Diegodelas
Advancesinneuralinformationprocessingsystems,
| ers. |     |     |     |     |     |     | Casas,Florian |     | Bressand,Gianna |     | Lengyel,Guillaume |     |     |
| ---- | --- | --- | --- | --- | --- | --- | ------------- | --- | --------------- | --- | ----------------- | --- | --- |
33:1877–1901,2020.
Lample,LucileSaulnier,etal.Mistral7b.arXivpreprint
arXiv:2310.06825,2023.
[7] BochuanCao,YuanpuCao,LuLin,andJinghuiChen.
Defending against alignment-breaking attacks via ro- [18] AlbertQJiang,AlexandreSablayrolles,AntoineRoux,
| bustlyalignedllm. |     | arXivpreprintarXiv:2309.14348, |     |     |     |     |        |         |         |         |       |          |     |
| ----------------- | --- | ------------------------------ | --- | --- | --- | --- | ------ | ------- | ------- | ------- | ----- | -------- | --- |
|                   |     |                                |     |     |     |     | Arthur | Mensch, | Blanche | Savary, | Chris | Bamford, | De- |
2023.
vendraSinghChaplot,DiegodelasCasas,EmmaBou
Hanna,FlorianBressand,etal.Mixtralofexperts.arXiv
| [8] Patrick | Chao, | Alexander |     | Robey, Edgar |     | Dobriban, |     |     |     |     |     |     |     |
| ----------- | ----- | --------- | --- | ------------ | --- | --------- | --- | --- | --- | --- | --- | --- | --- |
preprintarXiv:2401.04088,2024.
| HamedHassani,GeorgeJPappas,andEricWong. |       |           |          |        |     | Jail-     |     |     |     |     |     |     |     |
| --------------------------------------- | ----- | --------- | -------- | ------ | --- | --------- | --- | --- | --- | --- | --- | --- | --- |
| breaking                                | black | box large | language | models |     | in twenty |     |     |     |     |     |     |     |
[19] NikhilKandpal,MatthewJagielski,FlorianTramèr,and
queries. arXivpreprintarXiv:2310.08419,2023. NicholasCarlini. Backdoorattacksforin-contextlearn-
|                                               |     |     |     |     |     |     | ingwithlanguagemodels. |     |     | InTheSecondWorkshopon |     |     |     |
| --------------------------------------------- | --- | --- | --- | --- | --- | --- | ---------------------- | --- | --- | --------------------- | --- | --- | --- |
| [9] LihuChen,GaelVaroquaux,andFabianSuchanek. |     |     |     |     |     | Im- |                        |     |     |                       |     |     |     |
NewFrontiersinAdversarialMachineLearning,2023.
putingout-of-vocabularyembeddingswithlovemakes
languagemodelsrobustwithlittlecost. InProceedings [20] TongLiu,ZizhuangDeng,GuozhuMeng,YuekangLi,
ofthe60thAnnualMeetingoftheAssociationforCom- andKaiChen. Demystifyingrcevulnerabilitiesinllm-
putationalLinguistics(Volume1:LongPapers),pages
|     |     |     |     |     |     |     | integratedapps. |     | arXivpreprintarXiv:2309.02926,2023. |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | ----------------------------------- | --- | --- | --- | --- |
3488–3504,2022.
[21] YangLiu,YuanshunYao,Jean-FrancoisTon,Xiaoying
[10] GeleiDeng,YiLiu,YuekangLi,KailongWang,Ying Zhang,Ruocheng Guo,Hao Cheng,Yegor Klochkov,
Zhang,Zefeng Li,Haoyu Wang,Tianwei Zhang,and Muhammad Faaiz Taufiq,and Hang Li. Trustworthy
| USENIX Association |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    4725 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

llms: a survey and guideline for evaluating large lan- 23639928/microsoft-bing-chatbot-ai-gpt-4-l
guagemodels’alignment. InSociallyResponsibleLan- lm,2023. Accessed:02/08/2024.
guageModellingResearch,2023.
|     |     |     |     |     |     |     | [33] MatthewPisano,PeterLy,AbrahamSanders,Bingsheng |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------------------------------------------- | --- | --- | --- | --- |
[22] YinhanLiu,MyleOtt,NamanGoyal,JingfeiDu,Man- Yao, Dakuo Wang, Tomek Strzalkowski, and Mei Si.
darJoshi,DanqiChen,OmerLevy,MikeLewis,Luke Bergeron: Combating adversarial attacks through a
Zettlemoyer,andVeselinStoyanov. Roberta:Arobustly conscience-basedalignmentframework. arXivpreprint
optimized bert pretraining approach. arXiv preprint arXiv:2312.00029,2023.
arXiv:1907.11692,2019.
|     |     |     |     |     |     |     | [34] Michalis | Polychronakis,Kostas |     | G Anagnostakis,and |     |
| --- | --- | --- | --- | --- | --- | --- | ------------- | -------------------- | --- | ------------------ | --- |
[23] Pingchuan Ma,Rui Ding,Shuai Wang,Shi Han,and EvangelosPMarkatos. Comprehensiveshellcodede-
|               |     |                                 |     |     |     |     | tectionusingruntimeheuristics. |     |     | InProceedingsofthe |     |
| ------------- | --- | ------------------------------- | --- | --- | --- | --- | ------------------------------ | --- | --- | ------------------ | --- |
| DongmeiZhang. |     | Insightpilot:Anllm-empoweredau- |     |     |     |     |                                |     |     |                    |     |
tomated data exploration system. In Proceedings of 26th Annual Computer Security Applications Confer-
the2023ConferenceonEmpiricalMethodsinNatural ence,pages287–296,2010.
| LanguageProcessing: |     |     | SystemDemonstrations,pages |     |     |     |                                                  |     |     |     |     |
| ------------------- | --- | --- | -------------------------- | --- | --- | --- | ------------------------------------------------ | --- | --- | --- | --- |
|                     |     |     |                            |     |     |     | [35] AlecRadford,JeffreyWu,RewonChild,DavidLuan, |     |     |     |     |
346–352,2023.
|     |     |     |     |     |     |     | Dario | Amodei,Ilya | Sutskever,et | al. Language | mod- |
| --- | --- | --- | --- | --- | --- | --- | ----- | ----------- | ------------ | ------------ | ---- |
[24] MantasMazeika,LongPhan,XuwangYin,AndyZou, elsareunsupervisedmultitasklearners. OpenAIblog,
1(8):9,2019.
| Zifan      | Wang,Norman |     | Mu,Elham       | Sakhaee,Nathaniel |            |     |                                                        |     |     |     |     |
| ---------- | ----------- | --- | -------------- | ----------------- | ---------- | --- | ------------------------------------------------------ | --- | --- | --- | --- |
| Li, Steven | Basart,     | Bo  | Li, David      | Forsyth,          | and        | Dan |                                                        |     |     |     |     |
|            |             |     |                |                   |            |     | [36] RafaelRafailov,ArchitSharma,EricMitchell,Christo- |     |     |     |     |
| Hendrycks. | Harmbench:  |     | A standardized |                   | evaluation |     |                                                        |     |     |     |     |
|            |             |     |                |                   |            |     | pherDManning,StefanoErmon,andChelseaFinn.              |     |     |     | Di- |
frameworkforautomatedredteamingandrobustrefusal.
|     |     |     |     |     |     |     | rectpreferenceoptimization: |     |     | Yourlanguagemodelis |     |
| --- | --- | --- | --- | --- | --- | --- | --------------------------- | --- | --- | ------------------- | --- |
2024.
|     |     |     |     |     |     |     | secretlyarewardmodel. |     | AdvancesinNeuralInforma- |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------------- | --- | ------------------------ | --- | --- |
[25] MiladNasr,NicholasCarlini,JonathanHayase,Matthew tionProcessingSystems,36,2024.
| Jagielski, | A Feder | Cooper, | Daphne | Ippolito, | Christo- |     |            |          |        |             |        |
| ---------- | ------- | ------- | ------ | --------- | -------- | --- | ---------- | -------- | ------ | ----------- | ------ |
|            |         |         |        |           |          |     | [37] Elvis | Saravia. | Prompt | Engineering | Guide. |
pherAChoquette-Choo,EricWallace,FlorianTramèr,
https://github.com/dair-ai/Prompt-Engineering-Guide,
| andKatherineLee. |     | Scalableextractionoftrainingdata |     |     |     |     |     |     |     |     |     |
| ---------------- | --- | -------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
122022.
| from | (production) | language | models. |     | arXiv preprint |     |     |     |     |     |     |
| ---- | ------------ | -------- | ------- | --- | -------------- | --- | --- | --- | --- | --- | --- |
arXiv:2311.17035,2023. [38] SebastianSchrittwieser,StefanKatzenbeisser,Johannes
|              |             |     |                      |     |     |     | Kinder,GeorgMerzdovnik,andEdgarWeippl. |     |     |     | Protect- |
| ------------ | ----------- | --- | -------------------- | --- | --- | --- | -------------------------------------- | --- | --- | --- | -------- |
| [26] OpenAI. | Moderation. |     | https://platform.ope |     |     |     |                                        |     |     |     |          |
ingsoftwarethroughobfuscation:Canitkeeppacewith
| nai.com/docs/guides/moderation/overview. |     |     |     |     |     | Ac- |          |         |           |               |         |
| ---------------------------------------- | --- | --- | --- | --- | --- | --- | -------- | ------- | --------- | ------------- | ------- |
|                                          |     |     |     |     |     |     |          |         |           | ACM Computing | Surveys |
|                                          |     |     |     |     |     |     | progress | in code | analysis? |               |         |
cessedon08/08/2023.
(CSUR),49(1):1–37,2016.
| [27] OpenAI. |     | Safety | best practices. |     |     | https: |                                                |     |     |     |     |
| ------------ | --- | ------ | --------------- | --- | --- | ------ | ---------------------------------------------- | --- | --- | --- | --- |
|              |     |        |                 |     |     |        | [39] YuqiangSun,DaoyuanWu,YueXue,HanLiu,WeiMa, |     |     |     |     |
//platform.openai.com/docs/guides/safety
|     |     |     |     |     |     |     | Lyuye | Zhang,Miaolei | Shi,andYang | Liu. | Llm4vuln: |
| --- | --- | --- | --- | --- | --- | --- | ----- | ------------- | ----------- | ---- | --------- |
-best-practices.
Accessedon08/08/2023.
Aunifiedevaluationframeworkfordecouplinganden-
https://openai.com hancingllms’vulnerabilityreasoning. arXivpreprint
| [28] OpenAI. | Introducingchatgpt. |     |     |     |     |     |     |     |     |     |     |
| ------------ | ------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
arXiv:2401.16185,2024.
| /blog/chatgpt,2022. |                       |     | Accessed:08/08/2023. |                       |     |     |                                                      |     |     |     |     |
| ------------------- | --------------------- | --- | -------------------- | --------------------- | --- | --- | ---------------------------------------------------- | --- | --- | --- | --- |
|                     |                       |     |                      |                       |     |     | [40] HugoTouvron,LouisMartin,KevinStone,PeterAlbert, |     |     |     |     |
| [29] OpenAI.        | Gpt-4technicalreport. |     |                      | ArXiv,abs/2303.08774, |     |     |                                                      |     |     |     |     |
| 2023.               |                       |     |                      |                       |     |     | AmjadAlmahairi,YasmineBabaei,NikolayBashlykov,       |     |     |     |     |
SoumyaBatra,PrajjwalBhargava,ShrutiBhosale,etal.
[30] LongOuyang,JeffreyWu,XuJiang,DiogoAlmeida, Llama2:Openfoundationandfine-tunedchatmodels.
Carroll Wainwright, Pamela Mishkin, Chong Zhang, arXivpreprintarXiv:2307.09288,2023.
| Sandhini | Agarwal, | Katarina | Slama, | Alex | Ray, | et al. |     |     |     |     |     |
| -------- | -------- | -------- | ------ | ---- | ---- | ------ | --- | --- | --- | --- | --- |
Training language models to follow instructions with [41] Lewis Tunstall, Edward Beeching, Nathan Lambert,
NazneenRajani,KashifRasul,YounesBelkada,Shengyi
| humanfeedback. |     | AdvancesinNeuralInformationPro- |     |     |     |     |     |     |     |     |     |
| -------------- | --- | ------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
cessingSystems,35:27730–27744,2022. Huang, Leandro von Werra, Clémentine Fourrier,
|     |     |     |     |     |     |     | NathanHabib,etal. |     | Zephyr:Directdistillationoflm |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ----------------------------- | --- | --- |
[31] FábioPerezandIanRibeiro. Ignorepreviousprompt: alignment. arXivpreprintarXiv:2310.16944,2023.
| Attacktechniquesforlanguagemodels. |     |     |     |     | InNeurIPSML |     |                                                  |     |     |     |     |
| ---------------------------------- | --- | --- | --- | --- | ----------- | --- | ------------------------------------------------ | --- | --- | --- | --- |
|                                    |     |     |     |     |             |     | [42] YueWang,HungLe,AkhileshGotmare,NghiBui,Jun- |     |     |     |     |
SafetyWorkshop,2022.
|     |     |     |     |     |     |     | nan | Li, and Steven | Hoi. Codet5+: | Open | code large |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | ------------- | ---- | ---------- |
[32] JayPeters. Thebingaibothasbeensecretlyrunning languagemodelsforcodeunderstandingandgeneration.
gpt-4. https://www.theverge.com/2023/3/14/ In Proceedings ofthe 2023 Conference on Empirical
| 4726    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

| MethodsinNaturalLanguageProcessing,pages1069– |     |     |     |     |     | Appendix |     |     |     |     |     |
| --------------------------------------------- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- |
1088,2023.
A Dataset
[43] AlexanderWei,NikaHaghtalab,andJacobSteinhardt.
| Jailbroken:Howdoesllmsafetytrainingfail? |     |     |     |     | Advances |     |     |     |     |     |     |
| ---------------------------------------- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- | --- |
Toensureourdatasetcoverasufficientlybroadrangeofharm-
inNeuralInformationProcessingSystems,36,2024.
fultopics,weconductedacomprehensiveclassificationand
https://en.wikipedia.org/wik statistical analysis. In terms of the question taxonomy in
| [44] Wiki. | Pig latin. |     |     |     |     |     |     |     |     |     |     |
| ---------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
HarmBench[24],ourdatasetcoversall7categories,withthe
| i/Pig_Latin. |     | Accessedon04/05/2024. |     |     |     |                         |     |             |            |            |     |
| ------------ | --- | --------------------- | --- | --- | --- | ----------------------- | --- | ----------- | ---------- | ---------- | --- |
|              |     |                       |     |     |     | respective distribution |     | as follows: | Cybercrime | & Unautho- |     |
[45] ZihaoXu,YiLiu,GeleiDeng,YuekangLi,andStjepan rizedIntrusion(16.7%),Chemical&BiologicalWeapons/-
Picek. Llmjailbreakattackversusdefensetechniques–a Drugs(8.3%),CopyrightViolations(10%),Misinformation
arXivpreprintarXiv:2402.13457,
comprehensivestudy. &Disinformation(11.7%),Harassment&Bullying(10.8%),
| 2024. |     |     |     |     |     | IllegalActivities(24.2%),andGeneralHarm(18.3%).Due |     |     |     |     |     |
| ----- | --- | --- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- | --- | --- |
totheimbalanceinthequantityofcategoriesamongtheorig-
[46] YifanYao,JinhaoDuan,KaidiXu,YuanfangCai,Zhibo
|                  |     |                             |     |     |     | inally collected100 | questions       |         | from public  | datasets,witha |     |
| ---------------- | --- | --------------------------- | --- | --- | --- | ------------------- | --------------- | ------- | ------------ | -------------- | --- |
| Sun,andYueZhang. |     | Asurveyonlargelanguagemodel |     |     |     |                     |                 |         |              |                |     |
|                  |     |                             |     |     |     | significant         | lack of content | related | to Copyright | Violations     |     |
(llm)securityandprivacy:Thegood,thebad,andthe
andMisinformation&Disinformation,ourdatasetexpansion
| ugly. | High-ConfidenceComputing,page100211,2024. |     |     |     |     |     |     |     |     |     |     |
| ----- | ----------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
focusedprimarilyonthesetwocategories.
| [47] JiahaoYu,XingweiLin,andXinyuXing. |     |     |     |     | Gptfuzzer: |     |     |     |     |     |     |
| -------------------------------------- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- |
Redteaminglargelanguagemodelswithauto-generated
B OpenSourceLLMs
| jailbreak | prompts. | arXiv | preprint | arXiv:2309.10253, |     |     |     |     |     |     |     |
| --------- | -------- | ----- | -------- | ----------------- | --- | --- | --- | --- | --- | --- | --- |
2023.
ThedetailedinformationaboutopensourceLLMsweused
inourexperimentsisasfollows:
| [48] Youliang | Yuan,Wenxiang |     | Jiao,Wenxuan |     | Wang,Jen- |     |     |     |     |     |     |
| ------------- | ------------- | --- | ------------ | --- | --------- | --- | --- | --- | --- | --- | --- |
tseHuang,PinjiaHe,ShumingShi,andZhaopengTu. • LLAMA-2-13B-Chat is LLAMA-2-13B fine-tuned with
Gpt-4 is too smartto be safe: Stealthychatwithllms SFT andRLHF,itsurpasses open-source chatmodels in
helpfulnessandsafety,settingarobustbaselineforfurther
| viacipher.                    | InTheTwelfthInternationalConferenceon |     |     |     |     |                             |     |     |     |     |     |
| ----------------------------- | ------------------------------------- | --- | --- | --- | --- | --------------------------- | --- | --- | --- | --- | --- |
| LearningRepresentations,2023. |                                       |     |     |     |     | open-sourceLLMadvancements. |     |     |     |     |     |
• Vicuna-13B,fine-tunedfromLLAMA-2-13BthroughSFT,
| [49] Yi Zeng, | Hongpeng | Lin, | Jingwen | Zhang, | Diyi Yang, |     |     |     |     |     |     |
| ------------- | -------- | ---- | ------- | ------ | ---------- | --- | --- | --- | --- | --- | --- |
excelsinconversationalabilitiesandaligningwithhuman
| RuoxiJia,andWeiyanShi. |     |     | Howjohnnycanpersuade |     |     |                       |     |         |               |      |     |
| ---------------------- | --- | --- | -------------------- | --- | --- | --------------------- | --- | ------- | ------------- | ---- | --- |
|                        |     |     |                      |     |     | preferences,evidenced |     | by over | 80% agreement | with | hu- |
llmstojailbreakthem:Rethinkingpersuasiontochal-
manjudgmentsonbenchmarkslikeMT-BenchandChatbot
| lenge | ai safety | by humanizing |     | llms. | arXiv preprint |     |     |     |     |     |     |
| ----- | --------- | ------------- | --- | ----- | -------------- | --- | --- | --- | --- | --- | --- |
Arena.Inthispaper,weutilizethelatestversion(i.e.,ver-
arXiv:2401.06373,2024.
sion1.5)ofVicunaasthetargetmodel.
[50] YueZhang,YafuLi,LeyangCui,DengCai,LemaoLiu, • Mistral-7B-Instruct is fine-tuned on public instruction
Tingchen Fu, Xinting Huang, Enbo Zhao, Yu Zhang, datasetsusingSFT,itoutperformsallpreceding7Bmod-
Yulong Chen, et al. Siren’s song in the ai ocean: A elsoninstruction-followingtasks,showcasingsignificant
surveyonhallucinationinlargelanguagemodels. arXiv adaptabilityandperformance.
preprintarXiv:2309.01219,2023. • Mixtral8x7B-InstructcombinesSFTwithDPOtoenhance
instructionresponsivenessandreducebiases,outperform-
[51] LianminZheng,Wei-LinChiang,YingSheng,Siyuan
ingmodelslikeGPT-3.5Turboinhumanevaluations.More-
Zhuang,ZhanghaoWu,YonghaoZhuang,ZiLin,Zhuo-
|                                |     |     |     |     |                | over,Mixtral | features | a Mixture | of Experts | architecture, |     |
| ------------------------------ | --- | --- | --- | --- | -------------- | ------------ | -------- | --------- | ---------- | ------------- | --- |
| hanLi,DachengLi,EricXing,etal. |     |     |     |     | Judgingllm-as- |              |          |           |            |               |     |
settingitapartintermsofdesignandperformance.
| a-judgewithmt-benchandchatbotarena. |     |     |     |     | Advancesin |     |     |     |     |     |     |
| ----------------------------------- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- |
NeuralInformationProcessingSystems,36,2024. • Zephyr-7ButilizesdistilledSFTanddistilledDPOtoalign
closelywithuserintent,settingnewperformancebaselines
[52] DanielMZiegler,NisanStiennon,JeffreyWu,TomB for7Bmodelswithouttheneedforhumanannotation,and
Brown,AlecRadford,DarioAmodei,PaulChristiano,
efficientlyoutperformingsimilar-sizedmodels.
| andGeoffreyIrving. |     | Fine-tuninglanguagemodelsfrom  |     |     |     |     |     |     |     |     |     |
| ------------------ | --- | ------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| humanpreferences.  |     | arXivpreprintarXiv:1909.08593, |     |     |     |     |     |     |     |     |     |
C DialogueTemplates
2019.
[53] Andy Zou, Zifan Wang, J Zico Kolter, and Matt Uponreviewingthedialogueformattingproceduresofopen-
Fredrikson. Universalandtransferableadversarialat- sourceLargeLanguageModels,ithasbeenobservedthatthey
tacks on aligned language models. arXiv preprint universallyincorporatespecifictokenstodelineatethequery
arXiv:2307.15043,2023. from the completion. Examples of these dialog templates
| USENIX Association |     |     |     |     |     |     | 33rd USENIX Security Symposium    4727 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

fromopen-sourcemodelsareprovided,wheretheseparating thenumberoftokensintheresponse,wederivetheequation:
tokensarehighlightedforclarity.
1
logπ (y=d|x)=
m Θ
| DialogTemplateofVicuna |     |     |     | (cid:32) |         |         |     |         |                  | (cid:33) |
| ---------------------- | --- | --- | --- | -------- | ------- | ------- | --- | ------- | ---------------- | -------- |
|                        |     |     |     | 1        |         |         | m−1 |         |                  |          |
|                        |     |     |     |          | logπ (y | =d |x)+ | ∑   | logπ (y | =d |x,...,yi=di) |          |
| {{SYSTEMPROMPT}}       |     |     |     |          | Θ       | 1 1     |     | Θ       | i+1 i+1          |          |
m
i=1
USER:{{USERQUERY}}
ASSISTANT:{{LLMCOMPLETION}}
Thisexpressionisessentiallythenegationofthemodel’s
|     |     |     |     | log-perplexityforresponsed |     |     |     | givenacontextx.Correspond- |     |     |
| --- | --- | --- | --- | -------------------------- | --- | --- | --- | -------------------------- | --- | --- |
DialogTemplateofMistral
|     |     |     |     | ingly, | according | to  | inequalities | 6   | and the aforementioned |     |
| --- | --- | --- | --- | ------ | --------- | --- | ------------ | --- | ---------------------- | --- |
equation,wehaveinequalities10:
[INST]{{SYSTEMPROMPT}}
{{USERQUERY}}[/INST]{{LLMCOMPLETION}}
|     |     |     |     | (cid:26) logPPL(y=d|x′)−logPPL(y=d|x)>0, |     |     |     |     | ∀d∈D |     |
| --- | --- | --- | --- | ---------------------------------------- | --- | --- | --- | --- | ---- | --- |
declination
DialogTemplateofZephyr logPPL(y=d|x′)−logPPL(y=d|x)<0, ∀d∈Dcooperation
Here,PPLdenotesperplexity,xisthecontextwhereharmful
<|system|>
{{SYSTEMPROMPT}}</s> contentresideswithinthequery,andx′signifyingthecontext
| <|user|> |     |     |     | whereharmfulcontentpresentsinthecompletion. |     |     |     |     |     |     |
| -------- | --- | --- | --- | ------------------------------------------- | --- | --- | --- | --- | --- | --- |
{{USERQUERY}}</s>
[<|assistant|>]
E SystemPromptsofTargetModels
{{LLMCOMPLETION}}
Forthewidelyusedlargelanguagemodels,weusetheirde-
DialogTemplateofMixtral
faultsystempromptsasfollows:
[INST]
VicunaAchatbetweenacurioususerandanartificialintelligence
{{SYSTEMPROMPT}}
assistant.Theassistantgiveshelpful,detailed,andpoliteanswers
{{USERQUERY}}[/INST]{{LLMCOMPLETION}}
totheuser’squestions.
DialogTemplateofChatGLM3
|     |     |     |     | LLAMA-2 |     | You are | a helpful,respectful |     | and honest assistant. |     |
| --- | --- | --- | --- | ------- | --- | ------- | -------------------- | --- | --------------------- | --- |
Alwaysanswerashelpfullyaspossible,whilebeingsafe.Your
<|system|>
{{SYSTEMPROMPT}} answers shouldnotinclude any harmful,unethical,racist,sex-
<|user|> ist,toxic,dangerous,orillegalcontent.Pleaseensurethatyour
{{USERQUERY}} responsesaresociallyunbiasedandpositiveinnature.
<|assistant|> Ifaquestiondoesnotmakeanysense,orisnotfactuallycoherent,
explainwhyinsteadofansweringsomethingnotcorrect.Ifyou
{{LLMCOMPLETION}}
|     |     |     |     | don’t | know | the answer | to a | question,please | don’t share | false |
| --- | --- | --- | --- | ----- | ---- | ---------- | ---- | --------------- | ----------- | ----- |
information.
DialogTemplateofNous-Hermes-2-Mixtral-8x7B
| <|im_start|>system |     |     |     | ChatGPTSeriesYouareahelpfulassistant. |     |     |     |     |     |     |
| ------------------ | --- | --- | --- | ------------------------------------- | --- | --- | --- | --- | --- | --- |
{{SYSTEMPROMPT}}<|im_end|>
<|im_start|>user
F LLAMA-2’sUpdateonSystemPrompt
{{USERQUERY}}<|im_end|>
<|im_start|>assistant
{{LLMCOMPLETION}} LLAMA2updateditssystempromptbyremovingthestrict
version,asitimpairedthemodel’snormalfunctionality.The
officialupdatesareshownbelow:
D DerivationofLogPerplexityInequalities ObservedIssue: Wereceivedfeedbackfromthecommunityonour
prompttemplateandweareprovidinganupdatetoreducethefalse
ReflectingontheautoregressivenatureofLLMsdelineatedin refusalratesseen.Falserefusalsoccurwhenthemodelincorrectly
Section2.1,thelikelihoodofanLLMgeneratingaresponse refusestoansweraquestionthatitshould,forexampleduetooverly
d givenacontextxcanbedecomposedasfollows: broadinstructionstobecautiousinhowitprovidesresponses.
|             |             |                |     | Updated                                                  | approach: |     | Based | on evaluation | and analysis,we |     |
| ----------- | ----------- | -------------- | --- | -------------------------------------------------------- | --------- | --- | ----- | ------------- | --------------- | --- |
|             | m−1         |                |     | recommendtheremovalofthesystempromptasthedefaultsetting. |           |     |       |               |                 |     |
| π (y=d|x)=π | (y =d |x)∏π | (y =d |x,...,y | =d) |                                                          |           |     |       |               |                 |     |
| Θ           | Θ 1 1       | Θ i+1 i+1      | i i |                                                          |           |     |       |               |                 |     |
Pullrequest#626removesthesystempromptasthedefaultoption,
i=1
|     |     |     |     | butstillprovides |     | an  | example | to helpenable | experimentation | for |
| --- | --- | --- | --- | ---------------- | --- | --- | ------- | ------------- | --------------- | --- |
Byapplyingthelogarithmtobothsidesandnormalizingby
thoseusingit.
| 4728    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |
