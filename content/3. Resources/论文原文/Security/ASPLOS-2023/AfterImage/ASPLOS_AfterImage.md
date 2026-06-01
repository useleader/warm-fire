---
publish: true
---

|     | Leaking           |          | Control |     | Flow | Information       |              | via | the Hardware |                   | Prefetcher |     |     |     |
| --- | ----------------- | -------- | ------- | --- | ---- | ----------------- | ------------ | --- | ------------ | ----------------- | ---------- | --- | --- | --- |
|     |                   | YunChen* |         |     |      |                   | LingfengPei* |     |              | TrevorE.Carlson   |            |     |     |     |
|     | SchoolofComputing |          |         |     |      | SchoolofComputing |              |     |              | SchoolofComputing |            |     |     |     |
NationalUniversityofSingapore NationalUniversityofSingapore NationalUniversityofSingapore
Abstract
Modernprocessordesignsuseavarietyofmicroarchitec-
tural methods to achieve high performance. Unfortunately, Train Trigger Observe
Pre-analysis
|                   |     |      |       |                |     |      |         |     | Prefetcher |     | Prefetcher |     |     | Cache |
| ----------------- | --- | ---- | ----- | -------------- | --- | ---- | ------- | --- | ---------- | --- | ---------- | --- | --- | ----- |
| new side-channels |     | have | often | been uncovered |     | that | exploit |     |            |     |            |     |     |       |
1202 peS 1  ]RC.sc[  1v47400.9012:viXra
| these enhanced |     | designs. | One | area that | has | received | little |     |     |     |     |     |     |     |
| -------------- | --- | -------- | --- | --------- | --- | -------- | ------ | --- | --- | --- | --- | --- | --- | --- |
attentionfromasecurityperspectiveistheprocessor’shard- Figure1:AfterImageOverview.
wareprefetcher,acriticalcomponentusedtomitigateDRAM
latency in today’s systems. Prefetchers, like branch predic- such as AES keys [38]. Recently, the branch predictor has
receivedsignificantattentionforitsabilitytoamountspecu-
tors,holdcriticalstaterelatedtotheexecutionoftheappli-
lativeexecutionattacks[11,27,31,50,57]byinducingthe
| cation,andhavethepotentialtoleaksecretinformation. |     |     |     |     |     |     | But |     |     |     |     |     |     |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
uptonow, therehasnotbeenademonstrationofageneric processortoexecutemispredictedinstructions. Furthermore,
anumberofothermicroarchitecturalfeatureshavebeennewly
| prefetcher | side-channel |     | that | could be | actively | exploited | in  |     |     |     |     |     |     |     |
| ---------- | ------------ | --- | ---- | -------- | -------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
today’shardware. identifiedasvulnerable,fromTLBsandLineFillBuffers,toa
varietyofotherhardwarestructures[20,42,48,57].
Inthispaper,wepresentAfterImage,anewside-channel
that exploits the Intel Instruction Pointer-based stride Hardwareprefetchershavealsobeeninvestigatedasapo-
prefetcher. Weobservethat,whentheexecutionoftheproces- tentialside-channelandcovertchannel,expandingthislistof
sorswitchesbetweendifferentprivatedomains,theprefetcher vulnerablehardwarestructures[14,51]. Prefetchersworkby
trainedbyonedomaincanbetriggeredinanother. Tothebest preloading data into the cache before it is requested by the
|     |     |     |     |     |     |     |     | processor | to mitigate | the effects | of  | extremely | long | DRAM |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ----------- | ----------- | --- | --------- | ---- | ---- |
ofourknowledge,thisworkisthefirsttopubliclydemonstrate
amethodologythatisbothalgorithm-agnosticandalsoable loadlatencies. InthevastmajorityofcurrentIntelprocessors,
toleakkerneldataintouserspace.AfterImageisdifferentfrom anInstructionPointer-basedstride(IP-stride)prefetchercan
befound[15]asitisasmallstructurethatcanprovideasig-
previousworks,asitleaksdataonthenon-speculativepathof
execution. Becauseofthis,alargeclassofworkthathasfo- nificantperformancebenefit. Thisprefetcherlearnsrepetitive
stridesbetweenloadaddressesrequestedbythesameIP.When
cusedonprotectingtransient,branch-outcome-baseddatawill
beunabletoblockthisside-channel. Byreverse-engineering theprefetcherobtainssufficientconfidence,itwillprefetchthe
theIP-strideprefetcherinmodernIntelprocessors,wehave nextaddressintothecache,whichisthesumofthecurrent
addressandthepreviouslylearnedstride.
| successfully | developed |     | three | variants of | AfterImage |     | to leak |     |     |     |     |     |     |     |
| ------------ | --------- | --- | ----- | ----------- | ---------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
controlflowinformationacrosscoderegions,processesand Shinetal.[51]wasthefirsttoverifytheexistenceofapo-
tentialdatabreachintroducedbytheIntelIP-strideprefetcher.
| the user-kernel |     | boundary. | We  | find a high | level | of accuracy |     |     |     |     |     |     |     |     |
| --------------- | --- | --------- | --- | ----------- | ----- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
inleakinginformationwithourmethodology(from91%,up TheywereabletoextractthesecretvalueoftheECDHalgo-
to99%),andproposetwomitigationtechniquestoblockthis rithm[5]inOpenSSL[6]throughacacheside-channel.While
side-channel,oneofwhichcanbeusedonhardwaresystems aninterestingproof-of-concept,thisworkwasonlyapplicable
today. toaveryspecificprogrambehavior,i.e. tablelook-ups,andis
|     |     |     |     |     |     |     |     | notageneralside-channel. |     | Onesubsequentwork[14]devel- |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------ | --- | --------------------------- | --- | --- | --- | --- |
1. Introduction
opedacovertchannelwiththeuseoftheIP-strideprefetcher.
Byiterativelytrainingandevictingtherecentlytrainedentries,
| Modern | systems | leverage | a   | variety of | methods | to achieve |     |            |                  |      |      |                |     |      |
| ------ | ------- | -------- | --- | ---------- | ------- | ---------- | --- | ---------- | ---------------- | ---- | ---- | -------------- | --- | ---- |
|        |         |          |     |            |         |            |     | the sender | and the receiver | were | able | to communicate |     | with |
highperformance,fromnewinstructionstomicroarchitectural
|               |     |                                        |     |     |     |     |     | each other. | This method | was | designed | solely | for | use as a |
| ------------- | --- | -------------------------------------- | --- | --- | --- | --- | --- | ----------- | ----------- | --- | -------- | ------ | --- | -------- |
| enhancements. |     | Unfortunately,manyofthesetechniquesde- |     |     |     |     |     |             |             |     |          |        |     |          |
covertchannelandrequiresboththesenderandreceiverto
signedtospeedupprocessorshavealsoresultedinexposing
|                        |        |       |              |                   |     |           |     | be controlled               | by the | attacker; | it cannot                   | be  | extended | to leak |
| ---------------------- | ------ | ----- | ------------ | ----------------- | --- | --------- | --- | --------------------------- | ------ | --------- | --------------------------- | --- | -------- | ------- |
| new microarchitectural |        |       | side-channel | attacks           |     | [19, 33]. | The |                             |        |           |                             |     |          |         |
|                        |        |       |              |                   |     |           |     | informationasaside-channel. |        |           | Thesefirstattemptstoexploit |     |          |         |
| processor              | cache, | as an | example,     | can significantly |     | improve   |     |                             |        |           |                             |     |          |         |
theIP-strideprefetcherareeitheralgorithm-specific,orcan
| workloadperformance. |            |          | However,itstiminginformationcan |              |       |     |      |         |                  |          |      |        |     |           |
| -------------------- | ---------- | -------- | ------------------------------- | ------------ | ----- | --- | ---- | ------- | ---------------- | -------- | ---- | ------ | --- | --------- |
|                      |            |          |                                 |              |       |     |      | only be | used as a covert | channel; | they | cannot | be  | used as a |
| expose               | the memory | activity |                                 | of programs, | which | has | been |         |                  |          |      |        |     |           |
general-purposeside-channelforalargeclassofapplications.
widelyexploitedastimingside-channelstoleakinformation
Inthispaper,wedemonstrateagenerichardwareprefetcher-
basedside-channel,calledAfterImage,thatcanbeexploited
*Theseauthorscontributedequallytothiswork.

in today’s processors1 to extract secret information across 2. AfterImageOverview
| program      | and user-kernel | boundaries. |         | With       | the use | of load |                                          |     |     |     |     |          |     |
| ------------ | --------------- | ----------- | ------- | ---------- | ------- | ------- | ---------------------------------------- | --- | --- | --- | --- | -------- | --- |
|              |                 |             |         |            |         |         | TheoverviewofAfterImageisshowninFigure1. |     |     |     |     | Thereare |     |
| instructions | that overlap    | the         | key set | of address | bits    | of the  |                                          |     |     |     |     |          |     |
fourmainstepsneededtoleakinformationviathisprefetcher
destinationIP,andashorttrainingphasewithdifferentstrides
| fortheifandelsecasesofabranch, |     |     |     | wecanpersuadethe |     |     | side-channel. |     |                                       |     |     |     |     |
| ------------------------------ | --- | --- | --- | ---------------- | --- | --- | ------------- | --- | ------------------------------------- | --- | --- | --- | --- |
|                                |     |     |     |                  |     |     | Pre-analysis: |     | Theattackerlocatesvulnerablecodeinthe |     |     |     |     |
hardwaretoleakinformationfromthenextexecutionofan
application,orfromthekernelitself(SeeFigure1). victimprocess,andgeneratesalocalversionofthetargeted
|     |     |     |     |     |     |     | load instructions. |     | These | loads masquerade |     | as the | target |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | ----- | ---------------- | --- | ------ | ------ |
Toaccomplishthis,weshowthat,whentheexecutionofthe
loadsandsharethesamehardwareentryintheprefetcher.
processorswitchesbetweendifferentdomains,suchasuser-
|     |     |     |     |     |     |     | Train | Prefetcher: | The | attacker then | trains | the IP-stride |     |
| --- | --- | --- | --- | --- | --- | --- | ----- | ----------- | --- | ------------- | ------ | ------------- | --- |
kernelstateswitching,theIP-strideprefetchertrainedbyone
prefetcherlocallybyexecutingastridedaddresssequenceto
| domaincanbetriggeredinanotherdomain. |     |     |     |     | Thisprovidesan |     |     |     |     |     |     |     |     |
| ------------------------------------ | --- | --- | --- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
opportunitytoleakthecontrolfloworexecutioninformation obtainasufficientlevelofconfidenceintheprefetcher.
TriggerPrefetcher:Whenthevictimexecutesthetargeted
oftheapplication,andthereforethepotentialsecret,evenin
thepresenceoftraditionalhardwareisolationtechniques. code region, the prefetcher will be automatically triggered
withthepreviouslytrainedstride.
Insummary,wemakethefollowingmajorcontributions:
|                                               |       |            |               |     |             |        | Observe                        | Cache: | The             | attacker detects        | the           | existence | of     |
| --------------------------------------------- | ----- | ---------- | ------------- | --- | ----------- | ------ | ------------------------------ | ------ | --------------- | ----------------------- | ------------- | --------- | ------ |
| • We describe                                 | a new | prefetcher | side-channel, |     | AfterImage, |        |                                |        |                 |                         |               |           |        |
|                                               |       |            |               |     |             |        | one or more                    |        | strides through | cache                   | side-channels |           | (e.g., |
| thatisabletoleakthecontrolflowofapplications. |       |            |               |     |             | Thenew |                                |        |                 |                         |               |           |        |
|                                               |       |            |               |     |             |        | Prime+Probe,Flush+Reload,etc.) |        |                 | andinfersthecontrolflow |               |           |        |
side-channelattackdoesnotrelyonchangestothecontrol
toleakthesecretdata.
flowgraph(i.e.,changethestatusofbranchpredictor)and
TheflexibilityoftrainingtheIP-strideprefetcherwithar-
| does not | rely on branch-speculative |     |     | execution, |     | defeating |     |     |     |     |     |     |     |
| -------- | -------------------------- | --- | --- | ---------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- |
manyrecentdefencesagainsttransientside-channels. Asa bitrary stride values provides a series of advantages. First,
weexplicitlyuseastridevaluewithacachelinegranularity,
side-channelattack,AfterImagecanachieveahighattack
whichalignswiththeuseoftraditionalcacheside-channels
successrate(from91%to99%).
suchasPrime+Probe[38]andFlush+Reload[63],sincethey
• Toaccomplishtheside-channelattack,wedevelopedacus-
|     |     |     |     |     |     |     | can only | detect | information | as fine as | the cache | line | size. |
| --- | --- | --- | --- | --- | --- | --- | -------- | ------ | ----------- | ---------- | --------- | ---- | ----- |
tomsetofmicrobenchmarksthatrevealedcriticalfeatures
of the Intel IP-stride prefetcher, including the number of Moreover,weonlydetectwhetherthesespecificstridesexist
|     |     |     |     |     |     |     | ornot. Thisallowsforagenericimplementation,andrequires |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------ | --- | --- | --- | --- | --- | --- |
entries,thestructureofeveryentry,strideandconfidence
significantlylesseffortbytheattackertoextractpotentialse-
updatingpolicy,indexingstrategy,replacementpolicy,and
|                     |                 |                                |                        |     |     |      | cret information                 |     | compared | with previous         | works | that | utilize |
| ------------------- | --------------- | ------------------------------ | ---------------------- | --- | --- | ---- | -------------------------------- | --- | -------- | --------------------- | ----- | ---- | ------- |
| pagecheckingpolicy. |                 | Byusingthesemicrobenchmarks,we |                        |     |     |      |                                  |     |          |                       |       |      |         |
|                     |                 |                                |                        |     |     |      | time-seriesanalyzingmethods[51]. |     |          | Inaddition,thismethod |       |      |         |
| presentan           | in-depthstudyof |                                | theIP-strideprefetcher |     |     | used |                                  |     |          |                       |       |      |         |
in modern Intel processors. Reverse engineering critical alsoimprovestheresolutionandaccuracyofAfterImagecom-
|     |     |     |     |     |     |     | paredtopriorworks. |     | Ourproof-of-conceptisbuiltuponthe |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --------------------------------- | --- | --- | --- | --- |
featureslikethenewabilitytoprefetchpastthevirtualpage
|     |     |     |     |     |     |     | Prime+Probe | and | Flush+Reload | cache | side-channel, |     | which |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ------------ | ----- | ------------- | --- | ----- |
boundary,andthefactthathigh-confidencematcheswillal-
seessuccessrateofatleast91%andashighas99%.
waystrigger,haveallowedustoconstructthisside-channel.
|     |     |     |     |     |     |     | AfterImage | is  | not susceptible | to the | prevailing | defenses |     |
| --- | --- | --- | --- | --- | --- | --- | ---------- | --- | --------------- | ------ | ---------- | -------- | --- |
• WeevaluatethreeattackvariantsthatexploitthenovelIP-
strideprefetchervulnerability: (a)cross-coderegion/same againsttransientexecution[11,28,34,37,55,65,43]. Many
ofthesedefenseswillselectivelydisablethespeculativeloads
address-spaceattack,(b)cross-processattack,and(c)cross
byserializingloadinstructionsorintroducinginvisiblespec-
user-kernelboundaryattack.
|               |           |     |          |                |     |       | ulation if                             | the malicious | code | is trying to | transiently    |     | access |
| ------------- | --------- | --- | -------- | -------------- | --- | ----- | -------------------------------------- | ------------- | ---- | ------------ | -------------- | --- | ------ |
| • We propose, | implement | and | evaluate | a light-weight |     | miti- |                                        |               |      |              |                |     |        |
|               |           |     |          |                |     |       | addressesthatmightresultindataleakage. |               |      |              | However,After- |     |        |
gationstrategyforfuturehardwaredesignsthatshowsan
extremelysmall(0.8%)slowdownforapplicationswhich Imagedoesnotdeviatefromthethenormalcontrolflowor
dataflowoftheoriginalprogramandwillnotactivelychange
| cantakeadvantageofthisprefetcher. |     |     |     | Inaddition,wealso |     |     |                                    |     |     |     |                  |     |     |
| --------------------------------- | --- | --- | --- | ----------------- | --- | --- | ---------------------------------- | --- | --- | --- | ---------------- | --- | --- |
|                                   |     |     |     |                   |     |     | thepipelinestateoftheoftheprogram. |     |     |     | Thesedefensesare |     |     |
proposeamitigationthatcanbeusedontoday’shardware
topreventthisattack. thereforecurrentlyvulnerabletotheAfterImageside-channel.
Intherestofthispaper,wefirstprovideashortoverview
|     |     |     |     |     |     |     | 3. Background |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --- | --- | --- | --- | --- |
ofhowAfterImageworks(Section2)andprovidesomeaddi-
tionalbackgroundinformationforthiswork(Section3). Next, 3.1. PrefetchingonIntelMicroprocessors
westudytheIP-strideprefetcherpresentintoday’shardware
Prefetchingisawidelyadoptedtechniqueinmodernproces-
(Section4),anddiscuss(Section5)andevaluate(Section7)
theproposedside-channelandourdefenseoptions.Finally,we sorsusedtomitigatethelatencygapbetweentheCPUand
discusseffectivenessofexisteddefensesinSection8,outline thememorysubsystem. PrefetcherscanhidethelongDRAM
relatedworkinSection9,andconcludeinSection10. latencybypredictingandpreloadingdatafromslowermem-
oryintothehighspeedcachebeforethedataisrequestedby
1AfterImagehasbeendisclosedtoIntel,andapprovedfordistribution. theCPU.Intelprocessorsprovidebothsoftwareprefetching

instructioninterfacesanddedicatedprefetchinghardwarecom- IntelPrefetcher Location Pattern
DataCacheUnit
ponents. Software prefetching requires use of programmer L1-D Nextcacheline
(DCU)
knowledge or compiler information by inserting PREFETCH Instructionpointer Loadinstructionswith
|     |     |     |     |     |     |     | (IP)-stride | L1-D | regularstride |     |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ---- | ------------- | --- |
instructionsintotheprogramwithanexplicitmemoryaddress,
|                                                       |     |     |     |     |     |     | Dataprefetchlogic |     | 128-bytes-aligned |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ----------------- | --- |
| whilehardwareprefetchersautomaticallypredictthememory |     |     |     |     |     |     |                   | L2  |                   |     |
|                                                       |     |     |     |     |     |     | (DPL)             |     | paircacheline     |     |
accessaddressbylearningtherun-timememoryaccesspat- Severalcachelinesbackward
|     |     |     |     |     |     |     | Streamer | L2  | orforward |     |
| --- | --- | --- | --- | --- | --- | --- | -------- | --- | --------- | --- |
terns.Thespeculationthatoccursinthehardwareprefetcheris
differentfrombranchpredictorspeculation. Iftheprediction Table1:DocumentedIntelHardwarePrefetchers.
| of prefetching                                   | is wrong, | the useless | memory              |     | accesses | may |           |     |     |     |
| ------------------------------------------------ | --------- | ----------- | ------------------- | --- | -------- | --- | --------- | --- | --- | --- |
| wastebandwidthorpollutethecache.                 |           |             | However,thedatawill |     |          |     |           |     |     |     |
| notbeusedbytheprocessorandwillnotaffectthenormal |           |             |                     |     |          |     | Processor |     |     |     |
executionoftheprogram.
Prefetched
Intelhasdescribedfourhardwareprefetchersintheirpro- Load request queue
| cessor designs | [15], and | their | features | are listed | in  | Table 1. |                     |       |     |     |
| -------------- | --------- | ----- | -------- | ---------- | --- | -------- | ------------------- | ----- | --- | --- |
|                |           |       |          |            |     |          | IP Last Addr Stride | Conf. |     |     |
Thedatacacheunit(DCU)prefetcher,alsoknownasnext-line
|            |                |                  |     |          |     |           | IP Last Addr Stride | Conf. |     | FIFO |
| ---------- | -------------- | ---------------- | --- | -------- | --- | --------- | ------------------- | ----- | --- | ---- |
| prefetcher | [52], attempts | to automatically |     | prefetch |     | a single, |                     |       |     |      |
subsequent cache line. Data prefetch logic (DPL), i.e., the IP Last Addr Stride Conf.
……
adjacentprefetcher,regardsdataas128-bytesalignedblocks. IP Last Addr Stride Conf.
Send prefetched
Acachemisstooneofthetwocachelinesinthisblockwill
|           |                 |            |     |          |            |     | IP-stride prefetcher |     | request to cache |     |
| --------- | --------------- | ---------- | --- | -------- | ---------- | --- | -------------------- | --- | ---------------- | --- |
| trigger a | prefetch to the | pair line. | The | Streamer | prefetcher |     |                      |     |                  |     |
Figure2:GeneralarchitectureoftheIP-strideprefetcher.
| records sequential | positive | and | negative | offset | streams | and |     |     |     |     |
| ------------------ | -------- | --- | -------- | ------ | ------- | --- | --- | --- | --- | --- |
prefetches the next or previous several cache lines, respec- consequentdelay. Thepagecheckingissuewillbediscussed
tively. Previouswork[49]hasshownthattheStreamerwill indetailinSection4.
| maintainthestatusafteracontextswitchaswell. |     |     |     |     | However,it |      |                                |     |     |     |
| ------------------------------------------- | --- | --- | --- | --- | ---------- | ---- | ------------------------------ | --- | --- | --- |
|                                             |     |     |     |     |            | 3.2. | Cachetimingside-channelattacks |     |     |     |
islocatedattheL2cacheandindexedbyphysicalmemoryad-
dress.Itwilldynamicallydecidehowmanycachelinesshould Wheneverthetimetakenfortheprocessortoperformcertain
| be prefetched | based on | the system | status | (e.g., | bandwidth, |     |     |     |     |     |
| ------------- | -------- | ---------- | ------ | ------ | ---------- | --- | --- | --- | --- | --- |
operationsisdependentonsecretvalues,timingside-channels
streamingdirection,etc.). Therefore,theoperationofthese canexist[62]. Instruction-basedtimingside-channelsrelyon
threehardwareprefetchersdonothaveasmuchflexibilityas the correlation between the secret and the number of CPU
| theInstructionPointer(IP)-basedstridedprefetcher,i.e. |     |     |     |     |     | the                                        |     |     |     |             |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | ------------------------------------------ | --- | --- | --- | ----------- |
|                                                       |     |     |     |     |     | cyclesneededtoexecuteaninstructionsegment. |     |     |     | Cache-based |
IP-strideprefetcher. timingside-channelsexploitthelatencygapbetweenthecache
ThebasicstructureoftheIP-strideprefetcherisshownin
|     |     |     |     |     |     | andmemorysubsystems. |     | Whenthesecretvalueisrelatedto |     |     |
| --- | --- | --- | --- | --- | --- | -------------------- | --- | ----------------------------- | --- | --- |
Figure2.Thisprefetcherkeepstrackofloadinstructionswith thememoryaccessbehaviorofthesystem,attackershavethe
regularstridesfromthesameIP.Itsoperationiscomposedof potentialtoextractthesecretbyobservingtimingdifferences.
Whenaloadinstruction
threesteps. 1. IndexandReplace. Flush+Reload[63]isoneofthecachetimingside-channels
ispresent,itwillbeindexedintoanentrywiththesameIPtag. thattakesadvantageofsharedmemorybetweendifferentpro-
| If no such | IP tag exists, | a victim | entry | will be | selected | and     |                                               |     |     |     |
| ---------- | -------------- | -------- | ----- | ------- | -------- | ------- | --------------------------------------------- | --- | --- | --- |
|            |                |          |       |         |          | cesses. | Theattackerfirstflushesoutthesharedmemoryfrom |     |     |     |
evicted. 2. Update. Ifthedifferencebetweenthecurrentad- cacheintoDRAM.Afterthevictimperformsitsnormalex-
dressandtheLastAddrisequaltotheStride,theConfidence ecution, the attacker then reloads the shared memory and
will be increased, otherwise it will be decreased. However, observesthetimingdifferences, whereaddressesthathitin
iftheConfidencedropsbelowathreshold,theStridevalue thecacheindicateanaccessbythevictim.
| willbeupdatedtoanewstrideaswell. |     |     |     | 3. Prefetch. |     | Ifthe |     |     |     |     |
| -------------------------------- | --- | --- | --- | ------------ | --- | ----- | --- | --- | --- | --- |
ThePrime+Probe[38,41]cacheside-channeldoesnotre-
Confidence exceeds a certain threshold, a prefetch request quiretheexistenceofsharedmemory. Theattackerprimes
willbesenttothenextaddresswhichisthesumofthecurrent the cache sets with its own data, and probes whether these
accessaddressandtherecordedStridevalue.
cachesetsarestilloccupiedafterthevictimprogramhasbeen
AlthoughsomeoftheparametersandstructuresofIntel’s scheduled. Prime+ProbeismoregeneralthanFlush+Reload
IP-strideprefetcherhasbeendocumentedpreviously[15,25],
butitislessnoise-resilient,sinceanyactivityinthesystemcan
importantinformationaboutthereplacementandupdatepoli- evicttheprimingdataaswell. Therefore,undercircumstances
cieshavenotbeensharedpublicly. Inthiswork,wesharethe without interference from a significant number of memory
resultsofthereverseengineeringprocessandpresentthemin accesses,theattackercandeterminethesecretthroughtheuse
| Section4. |     |     |     |     |     | ofaninclusiveLLC-basedPrime+Probe. |     |     |     |     |
| --------- | --- | --- | --- | --- | --- | ---------------------------------- | --- | --- | --- | --- |
Therearealsootherimportantparametersusedduringthe Priming the complete LLC cache might not be easy to
operationofhardwareprefetchers. Forexample,cross-page achieve by accessing a chunk of data whose size is larger
prefetching will be restrained to avoid page faults and the thantheLLC.Thisisbecausemostrecentmicroarchitectures

|                                                    |     |     |     |     |     |     | void | idx_detect_train(int | stride, int train) |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | ---- | -------------------- | ------------------ | --- |
| dividetheLLCintosmallerslicestoreducecontentionand |     |     |     |     |     |     | 1    |                      |                    |     |
2 {
theslice-selectionalgorithmisnotpubliclyknown. Theevic- for(int i = 0; i < train; i++)
3
4 {
tionset,asaformaltermofprimingdata,isacollectionof
|                                                    |     |     |     |     |     |     | 5 IP_1: | int temp0 | = array[i * stride]; |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | ------- | --------- | -------------------- | --- |
| addressesthatmaptothesamecachesetandslicethatguar- |     |     |     |     |     |     |         | }         |                      |     |
6
anteesitscompleteeviction[26,53,58]. Aminimaleviction 7 // Not shown: add IP offset using NOPs
|     |     |     |     |     |     |     | IP_2:int | temp1 = array[r]; |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------- | ----------------- | --- | --- |
8
| set (MES)      | has the     | number | of                       | elements | equal | to the cache | 9 } |     |     |     |
| -------------- | ----------- | ------ | ------------------------ | -------- | ----- | ------------ | --- | --- | --- | --- |
| associativity. | Forexample, |        | iftheLLCofaprocessorhas4 |          |       |              |     |     |     |     |
slices(normallycorrespondstothenumberofcores)andits
|     |     |     |     |     |     |     | Listing1: | Microbenchmarkpseudo-codefordetectingthein- |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------- | ------------------------------------------- | --- | --- |
associativityis16,eachMEShas16elementstocoveracache
dexingmechanismoftheIP-strideprefetcher.
setonaspecificslice.
| ThegenericmodelforconstructingtheMES[53]systemati- |     |     |     |     |     |     | 250 |     |     |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
callytestsamongalargecandidateaddresspooltoseewhether
200
anelementcanbeevictedbythemanditerativelyshrinkits
emiT sseccA
| size,whichisanextremelytime-consumingprocess.However, |     |     |     |     |     |     | 150 |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
theLLCisindexedbyapartofthebitsofphysicaladdresses,
100
whichmeansthatthecomputedMESscanbeusedasatem-
| plateandcanbereuseddirectly. |          |         |     | Othermethodsaimtoexploit |           |            | 50  |     |     |     |
| ---------------------------- | -------- | ------- | --- | ------------------------ | --------- | ---------- | --- | --- | --- | --- |
| the CBox                     | hardware | counter | per | slice                    | stored in | Model Spe- |     |     |     |     |
0
cificRegisters(MSRs)toreverse-engineertheslice-selection 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16
| function | [36]. This | method | may | not be | effective | on newer |     |     |     |     |
| -------- | ---------- | ------ | --- | ------ | --------- | -------- | --- | --- | --- | --- |
#Matched least significant bits with IP_1
generationofIntelprocessorsduetotheCBoxmodifications.
Figure3: TheCoffeeLakeIP-strideprefetchertriggeringare-
Intheprimingphase,theattackeroccupiestheLLCcache
|                                       |     |     |     |                        |     |             | sult on IP_2 | when trained    | with IP_1. Note | that an Access     |
| ------------------------------------- | --- | --- | --- | ---------------------- | --- | ----------- | ------------ | --------------- | --------------- | ------------------ |
| withtheMESsandrecordstheiraccesstime, |     |     |     |                        |     | whichshould |              |                 |                 |                    |
|                                       |     |     |     |                        |     |             | Time higher  | than 120 cycles | means that      | the prefetcher has |
| resultinallcachehitlatencies.         |     |     |     | Intheprobingphase,MESs |     |             |              |                 |                 |                    |
notbeentriggeredtoprefetchtheaddressintocache.
areaccessedagainandtheirnewaccesstimeiscomparedwith
the baseline. If the time variance is larger than a threshold, whetherthereareanyotherfactorsthatshouldbetakeninto
thisMEShaselementsthathavebeenevictedanditishighly accountduringindexing.
possiblethatthevictimhasloadeddataintothiscachesetand
|     |     |     |     |     |     |     | Weuse | a microbenchmark, | similarto | that shown in List- |
| --- | --- | --- | --- | --- | --- | --- | ----- | ----------------- | --------- | ------------------- |
slice. ing 1, which first trains the IP-stride prefetcher using IP_1
withaconstantmultiplecacheline-sizedstrideandthenac-
4. UnderstandingIntel’sIP-stridePrefetcher
|     |     |     |     |     |     |     | cesses ther-th | cacheline | inthe array with | IP_2 (e.g., mov |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --------- | ---------------- | --------------- |
array[r],rax).IP_2isalsooffsetsuchthattheleastsignificant
| The IP-stride | prefetcher, |     | as implemented |     | in  | Intel’s Sandy |                         |     |                               |     |
| ------------- | ----------- | --- | -------------- | --- | --- | ------------- | ----------------------- | --- | ----------------------------- | --- |
|               |             |     |                |     |     |               | n-bitsmatchthoseofIP_1. |     | IftheloadatIP_2canactivatethe |     |
Bridgemicroarchitecture,waspreviouslydescribedinanIntel
whitepaper[15]. Haswell,anewergenerationmicroarchitec- prefetchertoprefetcharray[r+stride],i.e. maptothesame
entrywithIP_1,weconcludethattheindexingoftheIP-stride
| ture, usesenhanceddataprefetchers[25], |     |     |     |     | butthedetailsof |     |     |     |     |     |
| -------------------------------------- | --- | --- | --- | --- | --------------- | --- | --- | --- | --- | --- |
theseupdatesremainundocumented. Priorworks[14,51,59] prefetcherisdependentonthesen-bitsofIP.Figure3shows
thattheIP_2loadcantriggertheprefetcherifitslowest8-bits
| have attempted |              | to reverse-engineer |             | a number |          | of the charac- |                         |     |                            |     |
| -------------- | ------------ | ------------------- | ----------- | -------- | -------- | -------------- | ----------------------- | --- | -------------------------- | --- |
|                |              |                     |             |          |          |                | arethesameasthatofIP_1, |     | confirmingourunderstanding |     |
| teristics      | of the Intel | IP-stride           | prefetcher. |          | However, | in this        |                         |     |                            |     |
work,wetakeanadditionalsteptoreverse-engineerthemajor thattheIP-strideprefetcherusestheleastsignificant8-bitsto
|            |     |               |            |     |        |             | indextheentry. | Furthermore, | thisexampleverifiesthatthe |     |
| ---------- | --- | ------------- | ---------- | --- | ------ | ----------- | -------------- | ------------ | -------------------------- | --- |
| components | of  | the IP-stride | prefetcher |     | in the | Haswell and |                |              |                            |     |
CoffeeLakemicroarchitectures. Tothebestofourknowledge, IP-strideprefetcherlacksatagfieldtoverifythefullIP.
this is the first work to reveal the index, update and trigger 4.2. ConfidenceandStrideDetails
| mechanismsindetail. |     | Weadditionallyinvestigatetheeffects |     |     |     |     |     |     |     |     |
| ------------------- | --- | ----------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
ofcross-pageaddressprefetching,determinethenumberof IntheIP-strideprefetcherimplementation,theconfidence
entriesinthehistorytable,andreverseengineertheIP-stride determines whether a prefetch should be triggered and the
prefetcher’sreplacementpolicy. stride determines the offset of the prefetch. Investigating
howthesetwofieldsareupdatedormaintainedarecrucialin
4.1. IndexingintotheIP-stridePrefetcher
|     |     |     |     |     |     |     | understandinghowtheIP-strideprefetcheroperates. |     |     | Withthis |
| --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | -------- |
TounderstandhowtheIP-strideprefetcherisindexedinhard- knowledge, it can then be possible develop the AfterImage
ware,westartbyinvestigatingthedetailsoftheprefetcher’s side-channel. To reverse-engineer these details, we design
historytable. SinceoldergenerationsofIntelprocessorsare ourexperimentsbytrainingtheprefetcherintwophaseswith
indexedwiththeleastsignificant8-bitsoftheloadinstruction differentstrides,andobservingitsbehaviorateachstep.
address,wefirstaimtoverifywhetherthisisstilltrueinnewer Accordingtoourbasictest,theconfidencehastwobits
processorgenerations. Moreover,wewouldliketoinvestigate andthethresholdis2(weusetheforloopshowninListing1,

| void | const_ip_load(int | index, void* array) |     |     |     |     |     |            |     |     |     |
| ---- | ----------------- | ------------------- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- |
| 1    |                   |                     |     |     |     |     |     | AfterImage |     |     |     |
2 {
| IP_1: | int temp = | array[index]; |     |          |     |     | 21  | 28  |     |     | 17  |
| ----- | ---------- | ------------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
| 3     |            |               |     | Prefetch |     |     |     |     | 9   |     |     |
4 }
|     |     |     |     |         |     |     | +7    | +7  | +7  |     | +5  |
| --- | --- | --- | --- | ------- | --- | --- | ----- | --- | --- | --- | --- |
| 5   |     |     |     | Memory  |     | +7  | +7 +7 | -19 | +5  | +5  |     |
void policy_cs(int st_1, int st_2, int tr_1, int tr_2) 0 7 14 21 2 7 12
| 6   |     |     |     | Sequence 1 |     |     |     |     |     |     |     |
| --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
7 {
| char* | array = mmap(4096, | ...); |     |     |     |     |     |     |     |     |     |
| ----- | ------------------ | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
8
| 9 int    | i, offset;      |                 |     |     |     |                   |     |     |                   |     |     |
| -------- | --------------- | --------------- | --- | --- | --- | ----------------- | --- | --- | ----------------- | --- | --- |
|          |                 |                 |     |     |     | Training phase 1  |     |     | Training phase 2  |     |     |
| 10 for(i | = 0; i < tr_1;  | i++)            |     |     |     | with stride = 7   |     |     | with stride = 5   |     |     |
|          | const_ip_load(i | * st_1, array); |     |     |     |                   |     |     |                   |     |     |
11
12 flush(array); (a)Twotrainingphaseswitharandomoffsetinbetween.
| for(i | = 0; i < tr_2; | i++) |     |     |     |     |     |     |     |     |     |
| ----- | -------------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
13
| 14  | const_ip_load(offset | + i * st_2, array); |     |     |     |     |     |     |     |     |     |
| --- | -------------------- | ------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
15 // test whether the stride now is updated to st_2 Prefetch 21 28 33 36 41
| 16 time(array[i | * st_2]); |     |     |     |     |     |     |     |     |     |     |
| --------------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
17 // test whether the stride now is still st_1 +7 +7 +7 +5 +5
time(array[offset + (i -1 ) * st_2 + st_1]); Memory  +7 +7 +7 +5 +5 +5
| 18   |     |     |     |            | 0   | 7   | 14  | 21  | 26  | 31  | 36  |
| ---- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
| 19 } |     |     |     | Sequence 2 |     |     |     |     |     |     |     |
Listing2:Microbenchmarkpseudo-codefordetectingthecon- Training phase 1  Training phase 2
|     |     |     |     |     |     | with stride = 7 |     |     | with stride = 5 |     |     |
| --- | --- | --- | --- | --- | --- | --------------- | --- | --- | --------------- | --- | --- |
fidenceandstrideupdatingpolicyoftheIP-strideprefetcher.
(b)Thesecondtrainingphasestartsimmediatelyafterthefirst.
| and set different                       | values | for the train. After | training, we |            |              |     |           |        |           |            |     |
| --------------------------------------- | ------ | -------------------- | ------------ | ---------- | ------------ | --- | --------- | ------ | --------- | ---------- | --- |
|                                         |        |                      |              | Figure 4:  | Experimental |     | results   | of the | IP-stride | prefetcher |     |
| checkarray[i*stride]toseeifitiscached). |        |                      | Thestridehas |            |              |     |           |        |           |            |     |
|                                         |        |                      |              | triggering | mechanism    |     | on Coffee | Lake.  | Even      | though     | the |
(1+12)bits,withthemostsignificantbitisusedtodifferentiate
prefetcherseesanewstridevalue,itwillcontinuetoprefetch
betweennegativeandpositivestrides,whiletheothertwelve
usingthemostrecentlytrainedstride,enablingAfterImage.
bitsreflectthemaximumstride,whichcannotbemorethan
2KiB(1<<12). ItshouldbenotedthatthestrideofIntel’s be updated as current address − last address, and once
IP-strideprefetcherdoesnotneedtoaligntoacacheline[25]. the newly computed stride is different from the previous
As a result, the prefetcher requires up to 13 bits to deliver stride, the confidence will be reset to 1 at the same time.
the stride. However, because we train the prefetcher using Therefore,foraloadinstructionwithanIPandarequestto
cache-line-sizeddataoffsetsinthiswork,astrideof7means current address,theworkflowinsidetheprefetcherisshown
| thatthestriderecordedintheIP-strideprefetcherhasalength |     |     |     | inAlgorithm1. |     |     |     |     |     |     |     |
| ------------------------------------------------------- | --- | --- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- |
of7×64bytes,or7cachelinesintotal. The complete update policy for the confidence and
Afterdeterminingtheprefetcher’ssupportedconfidence stride can be described as follows. After a new entry is
and stride values, we use a microbenchmark (Listing 2) made, the stride and confidence are set to the learning
to reverse-engineer the confidence and stride update policy. value and 1, respectively, at the second cache miss. If the
The microbenchmark first trains the IP_1 tr_1 times (tr_1 strideremainsconstantinthethirditeration,theconfidence
> 2 to guarantee the confidence is equal or larger than the adds1(nowis2)andtriggerstheprefetchertoprefetchthe
threshold)withstridest_1,andthenusesanewstride(st_2) current address + stride. Iftheconfidenceislargerthan1,
totraintheprefetchertr_2times. Finally,theresultsfromthe theprefetcherwillprefetchcurrent address + stride,even
microbenchmarkrunallowsustodeterminewhichstrideis itfindsthecurrent address − last address (cid:54)= strideinthe
currentlybeingusedintheprefetcher,withtheresultslisted nextiteration. Inthemeantime,duetothecurrent address −
inFigure4. last address (cid:54)= stride, the prefetcher updates the stride to
Inourexperiment,st_1andst_2aresetas7and5,respec- currentaddress−lastaddress,andthenresettheconfidence
tively. Normally,astrideofst_1willbetriggeredevenafter to1(afterprefetching).
| thefirstiterationofthesecondloop. |     | Thismeansthatregard- |     |     |     |     |     |     |     |     |     |
| --------------------------------- | --- | -------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
4.3. PageBoundaryChecking
lessofwhetherthenewstrideisidenticaltotherecordedstride,
iftheconfidencereachestherequiredthreshold,theprefetcher Modern processors often use virtual memory management
| alwaysissuesanewprefetchrequest. |     | Werefertothisbehav- |     |             |        |     |                    |     |     |          |       |
| -------------------------------- | --- | ------------------- | --- | ----------- | ------ | --- | ------------------ | --- | --- | -------- | ----- |
|                                  |     |                     |     | with memory | paging |     | support. According |     | to  | previous | stud- |
iorasthekeycomponentofAfterImage. Thisallowsforthe ies[14,25],whenaninstructionloadsdata(current address)
triggeringoftheprefetchertooccurunconditionally,allowing fromanewpage,theprefetcherinvalidatestheentryandwill
theresulttoappearinaseparateexecutioncontext. needtore-learnthestride. Tounderstandhowcross-pageac-
Fortheseconditerationofthesecondloop,nomatterhow cessesaffectthisprefetcherinnewergenerations,andtoallow
large the value of tr_1 is, neither st_1 nor st_2 is triggered. AfterImage to be applied to a broad set of scenarios across
Whenwegettothethirditeration, thest_2isfinallyactive. differentexecutioncontexts,wenextbroadenourinvestigation
However, we discover that if we set the offset directly to toreverse-engineertheundocumentedelementsofthepage
| thestrideofthesecondphase, |     | i.e. startthesecondtraining |     | checkingpolicy. |     |     |     |     |     |     |     |
| -------------------------- | --- | --------------------------- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- |
earlier,theprefetcherwillbecomefullytrainedatthesecond ThebenchmarkshowninListing3firstbuildstwomemory
iteration. These results imply that the stride will always pools,namedrecl_arrayandlock_array. recl_arrayis

|                                              |     |     |     |     |     |     | void two_ip_loads(int |     |     | index, | void* array1, | void* array2) |     |
| -------------------------------------------- | --- | --- | --- | --- | --- | --- | --------------------- | --- | --- | ------ | ------------- | ------------- | --- |
| Algorithm1:Confidenceandstrideupdatingpolicy |     |     |     |     |     | 1   |                       |     |     |        |               |               |     |
2 {
|                                                |                                     |                    |     |     |     |     | IP_1: | int | temp0 = | array1[index]; |     |     |     |
| ---------------------------------------------- | ----------------------------------- | ------------------ | --- | --- | --- | --- | ----- | --- | ------- | -------------- | --- | --- | --- |
| andtriggeringstrategyoftheIP-strideprefetcher. |                                     |                    |     |     |     | 3   |       |     |         |                |     |     |     |
|                                                |                                     |                    |     |     |     | 4   | IP_2: | int | temp1 = | array2[index]; |     |     |     |
|                                                | Data:                               | IP,current_address |     |     |     | 5   | }     |     |         |                |     |     |     |
| 1                                              | ifIPtagexistedinthehistorytablethen |                    |     |     |     | 6   |       |     |         |                |     |     |     |
7
distance=current address−last address void page_policy(int offset, int stride)
| 2   |     |     |     |     |     | 8   |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
9 {
3 ifconfidence≥2then
|     |     |     |     |     |     | 1 0 | char* | recl_array |     | = mmap(n | * 4096, | ...); |     |
| --- | --- | --- | --- | --- | --- | --- | ----- | ---------- | --- | -------- | ------- | ----- | --- |
Prefetchcurrent address+stride char* lock_array = mmap(n * 4096, MAP_LOCKED, ...);
| 4   |     |                        |     |     |     | 11  |      |          |           |            |     |     |     |
| --- | --- | ---------------------- | --- | --- | --- | --- | ---- | -------- | --------- | ---------- | --- | --- | --- |
|     |     | ifdistance!=stridethen |     |     |     | 1 2 | / /  | d o n ot | cro s s p | ag e       |     |     |     |
| 5   |     |                        |     |     |     |     | f or | ( in t i | = 0 ; i   | < 4 ; i++) |     |     |     |
1 3
6 stride=distanceconfidence=1 14 two_ip_loads(i * stride, recl_array, lock_array);
|     |     | else                |     |     |     | 1 5 |                        |     |     |             |           |              |     |
| --- | --- | ------------------- | --- | --- | --- | --- | ---------------------- | --- | --- | ----------- | --------- | ------------ | --- |
| 7   |     |                     |     |     |     | 1 6 | two_ip_loads(offset,   |     |     | recl_array, |           | lock_array); |     |
| 8   |     | ifconfidence!=3then |     |     |     | 17  | time(recl_array[offset |     |     | +           | stride]); |              |     |
|     |     |                     |     |     |     |     | time(lock_array[offset |     |     | +           | stride]); |              |     |
| 9   |     | confidence+=1       |     |     |     | 1 8 |                        |     |     |             |           |              |     |
1 9 }
end
10
| 11  |      | end                    |     |     |     |                                               |     |                |     |             |     |                    |     |
| --- | ---- | ---------------------- | --- | --- | --- | --------------------------------------------- | --- | -------------- | --- | ----------- | --- | ------------------ | --- |
|     |      |                        |     |     |     | Listing                                       | 3:  | Microbenchmark |     | pseudo-code |     | for detecting      | the |
| 12  | else |                        |     |     |     | pagecheckingstrategyoftheIP-strideprefetcher. |     |                |     |             |     |                    |     |
| 13  |      | ifdistance!=stridethen |     |     |     |                                               |     |                |     |             |     |                    |     |
|     |      |                        |     |     |     |                                               |     | PhysicalAddr   |     | VirtualAddr |     | PrefetchTargetAddr |     |
|     |      | stride=distance        |     |     |     | Offset                                        |     |                |     |             |     |                    |     |
14 recl_array lock_array recl_array lock_array recl_array lock_array
|     |     |               |     |     |     | 1 P    | a g e   | s a m e | d i f f e r e n t | d i f f e r e | n t d i f f e | r e n t (cid:51) | (cid:51) |
| --- | --- | ------------- | --- | --- | --- | ------ | ------- | ------- | ----------------- | ------------- | ------------- | ---------------- | -------- |
| 15  |     | confidence=1  |     |     |     |        |         |         |                   |               |               |                  |          |
|     |     |               |     |     |     | 2 P    | a g e s | s a m e | d i f f e r e n t | d i f f e r e | n t d i f f e | r e n t (cid:51) | (cid:55) |
|     |     | else          |     |     |     |        |         |         |                   |               |               | (cid:51)         | (cid:55) |
| 16  |     |               |     |     |     | 3Pages |         | same    | different         | different     | different     |                  |          |
|     |     |               |     |     |     | 4Pages |         | same    | different         | different     | different     | (cid:51)         | (cid:55) |
| 17  |     | confidence+=1 |     |     |     |        |         |         |                   |               |               |                  |          |
ifconfidence==2then Table2:TheCoffeeLakeIP-strideprefetchertriggeringresults
18
|     |     | Prefetchcurrent |     | address+stride |     |                                             |                      |     |        |     |          |        |     |
| --- | --- | --------------- | --- | -------------- | --- | ------------------------------------------- | -------------------- | --- | ------ | --- | -------- | ------ | --- |
| 19  |     |                 |     |                |     | ondifferentlogicpagesandphysicalpageframes. |                      |     |        |     |          |        |     |
| 20  |     | end             |     |                |     |                                             |                      |     |        |     |          |        |     |
|     |     |                 |     |                |     | 1                                           | v oid n_ip_loads(int |     | index, | int | N, void* | array) |     |
| 21  |     | end             |     |                |     |                                             | {                    |     |        |     |          |        |     |
2
|     | end |     |     |     |     | 3   | IP_1: | int | temp1 = | array[4096 | + index]; |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----- | --- | ------- | ---------- | --------- | --- | --- |
| 22  |     |     |     |     |     |     | ...   |     | ...     |            |           |     |     |
4
| 23  | else                                       |     |     |     |     | 5   | IP_N: | int | tempN = | array[4096*N | +   | index]; |     |
| --- | ------------------------------------------ | --- | --- | --- | --- | --- | ----- | --- | ------- | ------------ | --- | ------- | --- |
|     | Create_New_Entry(IP,confidence=0,stride=0) |     |     |     |     | 6   | }     |     |         |              |     |         |     |
| 24  |                                            |     |     |     |     | 7   |       |     |         |              |     |         |     |
end
| 25  |     |     |     |     |     | 8   | void num_entry(int |     | N,  | int stride, | int | offset) |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- | ----------- | --- | ------- | --- |
{
9
|     |     |     |     |     |     | 10  | //contains |     | N pages |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ------- | --- | --- | --- | --- |
allocated without MAP_LOCKED, which is a resource-saving 11 char* array = mmap(N * 4096, MAP_LOCKED, ...);
|                                                    |      |               |              |               |         | 12  | for(int            | i            | = 0; i | < 5; i++)  |            |     |     |
| -------------------------------------------------- | ---- | ------------- | ------------ | ------------- | ------- | --- | ------------------ | ------------ | ------ | ---------- | ---------- | --- | --- |
| pool                                               | that | automatically | reclaim used | physical page | frames. |     |                    |              |        |            |            |     |     |
|                                                    |      |               |              |               |         | 13  |                    | n_ip_loads(i |        | * stride,  | N, array); |     |     |
| Thelock_array,ontheotherhand,willalwayslockthepage |      |               |              |               |         | 14  |                    |              |        |            |            |     |     |
|                                                    |      |               |              |               |         | 15  | n_ip_loads(offset, |              |        | N, array); |            |     |     |
frame. WeleverageIP_1andIP_2totraintheprefetcherwith
|     |     |     |     |     |     | 16  | for(int | i                 | = 0; i | < N; i++) |          |           |     |
| --- | --- | --- | --- | --- | --- | --- | ------- | ----------------- | ------ | --------- | -------- | --------- | --- |
|     |     |     |     |     |     | 17  |         | time(array[4096*i |        | +         | offset + | stride]); |     |
givenstrideononepage(e.g.,p-thpage),andthenaccessthe
18 }
nextoffset-thpage,andverifywhetherthetargetaddress(i.e.,
recl_/lock_array[p+offset+stride])isinthecacheornot.
Listing4:Microbenchmarkpseudo-codefordeterminingnum-
Table2showstheresultofthisexperiment.Thefirstcolumn
berofentriesoftheIP-strideprefetcher.
istheoffsetbetweenthetestingpageandthetrainingpage.
|     |     |     |     |     |     | page | mapping | hits | in TLB, | the | prefetcher | will be | activated |
| --- | --- | --- | --- | --- | --- | ---- | ------- | ---- | ------- | --- | ---------- | ------- | --------- |
Thesecondandthirdcolumnindicateswhetherthesetesting
pageshavethesamephysicalorvirtualaddresswiththetrain- immediately. However,forpagesfurtherintothefuture,i.e.
offset>1,prefetchingwillbeprohibited.Weinferthatthenext
| ingpageornot. |     | Thelastcolumnrepresentsthetestingresults, |     |     |     |     |     |     |     |     |     |     |     |
| ------------- | --- | ----------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
pageisspecialbecauseoftheuseofthenext-pageprefetcher
| i.e. | successfullyprefetchedornot. |     |     | Despitethefactthatthe |     |     |     |     |     |     |     |     |     |
| ---- | ---------------------------- | --- | --- | --------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
destinationaddressinrecl_arrayspansseverallogicalpage thatwasintroducedintheHaswellmicroarchitecture[4].
|     |     |     |     |     |     | As  | a result, | the | prefetcher |     | uses the | page frame | to deter- |
| --- | --- | --- | --- | --- | --- | --- | --------- | --- | ---------- | --- | -------- | ---------- | --------- |
boundaries,theprefetcherdoesnotinvalidatetheentryIP_1
andtheyareallsuccessfullytriggered. Ifthephysicalpage minewhetherthenewaddresscrossesthepageboundaryand
processesthenextpageseparately.
frameiscrossed,theprefetchercaninvalidatetheentryandre-
| learnthestrideandconfidence. |     |     | Morespecifically,ifthenewly |     |     |     |     |     |     |     |     |     |     |
| ---------------------------- | --- | --- | --------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
4.4. NumberofPrefetchEntries
accessedpageisthenextpageofthelearningpage((p+1)-th
page)andthispagemissesintheTLB,thefirstaccessforthis The number of entries represents the maximum number of
pagewillcreatethepagetableentryandwillnotimpactthe IPsandtheircorrespondingstridethattheprefetchercanre-
prefetcherstatus(e.g.,decreasetheconfidence). Thesecond member. This will affect how we search for a matched IP
memoryaccessonthe(p+1)-thpagethencandirectlyactivate whentargetIPisunknown,aswewillshowin5.3,sincethe
the prefetcher to prefetch current address + stride. If the hardwarecapacityisfixed.

if(secret)
|     |     | 30-inputs 26-inputs |     | 1     |                         |     |     |     |
| --- | --- | ------------------- | --- | ----- | ----------------------- | --- | --- | --- |
|     |     |                     |     | 2 int | temp0 = array[address]; |     |     |     |
| 250 |     |                     |     | else  |                         |     |     |     |
3
| 200 |     |     |     | 4 int | temp1 = array[address]; |     |     |     |
| --- | --- | --- | --- | ----- | ----------------------- | --- | --- | --- |
emiT sseccA
150
100
Listing5:Targetminimalvulnerablecoderegionofthevictim.
50
|     |     |     |     | Listing4byaddinganumberofjmpinstructions. |     |     | Thenumber |     |
| --- | --- | --- | --- | ----------------------------------------- | --- | --- | --------- | --- |
0
0 1 2 3 4 5 6 7 8 9 1011121314151617181920212223242526272829 oftestIPsisincreasedto32and32pageframesareallocated
|     |     | #Inputs |     | forthetrainingofeachIP.Thefirst24IPswillbetrainedon |     |     |     |     |
| --- | --- | ------- | --- | --------------------------------------------------- | --- | --- | --- | --- |
Figure5: TheIP-strideprefetchertriggeringresultsfor26IPs
variouspageframestooccupythewholetable,andthenthe
and30IPstodeterminetheprefetcher’snumberofentrieson cacheswillbeflushed. Next,thefirst8IPswillbere-trained
CoffeeLake. toupdatethemtoamorerecentlyusedposition. Following
|     |     |     |     | that, we train | another 8 | new IPs to evict | some entries | out |
| --- | --- | --- | --- | -------------- | --------- | ---------------- | ------------ | --- |
250
|     |     |     |     | andflushthecacheonceagain. |     | Finally,weexecutethese32 |     |     |
| --- | --- | --- | --- | -------------------------- | --- | ------------------------ | --- | --- |
200
emiT sseccA loadinstructionsagaintoreadarandomcachelineLinthe
| 150 |     |     |     | correspondingpageandseeifthe(L+stride)-thcacheline |                                         |     |     |     |
| --- | --- | --- | --- | -------------------------------------------------- | --------------------------------------- | --- | --- | --- |
|     |     |     |     | isprefetched.                                      | ThefirsteightIPsshouldhavebeenevictedif |     |     |     |
100
| 50  |     |     |     | theprefetcherusesaFIFOpolicy. |                    | Ifnot,theseIPaddresses |                |     |
| --- | --- | --- | --- | ----------------------------- | ------------------ | ---------------------- | -------------- | --- |
|     |     |     |     | should still                  | be able to trigger | prefetching.           | The experimen- |     |
0
1 2 3 4 5 6 7 8 91011121314151617181920212223242526272829303132 talresultisshowninFigure6. Weobservethattheevicted
|     |     | #Inputs |     | IPsarebetweenthe9thand16thposition,indicatingthatthe |     |     |     |     |
| --- | --- | ------- | --- | ---------------------------------------------------- | --- | --- | --- | --- |
Figure6: TheIP-strideprefetchertriggeringresultfor32IPs, IP-strideprefetcherisusingaformoftheLRUreplacement
with 8-16th IPs revisited, to demonstrate the prefetcher’s re- strategy. Inaddition,becausethereplacementshavealways
placementpolicyonCoffeeLake. beencontiguous,itfollowsthatitwillmostlikelynotusea
|     |     |     |     | tree-basedpseudo-LRU(PLRU)replacementpolicy. |     |     |     | Further, |
| --- | --- | --- | --- | -------------------------------------------- | --- | --- | --- | -------- |
Weconstructamicrobenchmark(SeeListing4)thatexe-
asatrueLRUimplementationcanbeexpensivetoimplement
cutesaloopwithavaryingnumberofloadinstructions.Every
inhardware,wesuspectthatthehardwareisimplementinga
| load’sleastsignificant8-bitsofitsIPareunique, |     |     | andtheir |     |     |     |     |     |
| --------------------------------------------- | --- | --- | -------- | --- | --- | --- | --- | --- |
Bit-PLRU-basedreplacementpolicy.
| dataaccesspatternsareconstant. |     | Aftertrainingeachofthese |     |     |     |     |     |     |
| ------------------------------ | --- | ------------------------ | --- | --- | --- | --- | --- | --- |
IPsondifferentpageframes(toavoidfalse-positives),were-
4.6. IP-stridePrefetcheracrossDifferentGenerations
accessthemandtesttheaccesstimetopage_t[offset+stride]
|     |     |     |     | Using our | microbenchmarking | methodology, | we  | reverse- |
| --- | --- | --- | --- | --------- | ----------------- | ------------ | --- | -------- |
(0≤t≤n)todetermineiftheycanstillactivatetheprefetcher.
TheexperimentalresultisshowninFigure5. SpecificIPs engineertheIP-strideprefetcheronHaswellandCoffeeLake
|                                        |     |     |                   | anddemonstratethesameconclusions. |     |     | Becausethemicroar- |     |
| -------------------------------------- | --- | --- | ----------------- | --------------------------------- | --- | --- | ------------------ | --- |
| arenolongerabletotriggertheprefetcher. |     |     | Morespecifically, |                                   |     |     |                    |     |
ifthenumberoftestIPsis26,thefirsttwoIPswillnolonger chitecturesofSkyLakeandKabyLakeareidenticaltothatof
CoffeeLake[9],weinferthattheHaswell,SkyLake,Kaby
| beabletoactivatetheprefetcher.               |     | IfthenumberoftestIPsis |           |           |                                |     |             |      |
| -------------------------------------------- | --- | ---------------------- | --------- | --------- | ------------------------------ | --- | ----------- | ---- |
|                                              |     |                        |           | Lake, and | Coffee Lake microarchitectures |     | all use the | same |
| 30,thefirstsixIPscannottriggertheprefetcher. |     |                        | Asaresult |           |                                |     |             |      |
oftheprefetcher’srestrictedsize,someIPsgetevicted. Thus, IP-strideprefetcher,implyingthattheyareallvulnerableto
AfterImage.
thenumberofprefetcherentriesisthenumberofIPsthatcan
stillactivatetheprefetcheraftertrainingallofthem,whichis Figure7depictsthemicroarchitectureofIntel’scurrentIP-
|     |     |     |     | strideprefetcher. | Comparedtotheoldergeneration[15],the |     |     |     |
| --- | --- | --- | --- | ----------------- | ------------------------------------ | --- | --- | --- |
24,inourcase.
optimizedprefetcherdecreasesthenumberofentriesfrom256
4.5. PrefetcherReplacementPolicy
to24,andthedirect-mappedindexingapproachisreplaced
withafullyassociativeindexingthatmostlikelyusesaBit-
| Since thecapacity | of IP-strideprefetcher |     | is limited, | i.e. 24 |     |     |     |     |
| ----------------- | ---------------------- | --- | ----------- | ------- | --- | --- | --- | --- |
Thelast address,stride,and
as seen in in Section 4.4, when new load instructions are PLRUreplacementpolicy.
|     |     |     |     | confidenceallhavethesamestructureasbefore. |     |     | Thepage |     |
| --- | --- | --- | --- | ------------------------------------------ | --- | --- | ------- | --- |
present,someentriesshouldbereplacedfollowingaspecific
checkingpolicyhasalsobeenmodifiedtoallowprefetching
| replacementpolicy. | Generally,wehavefoundthataslongas |     |     |                     |                                   |     |     |     |
| ------------------ | --------------------------------- | --- | --- | ------------------- | --------------------------------- | --- | --- | --- |
|                    |                                   |     |     | inthenextpageframe. | Ourreverse-engineeringeffortshave |     |     |     |
theloadinstructionofinterestwereexecutedrecently,itwill
|     |     |     |     | been the first, | to the best | of our knowledge, | to present | the |
| --- | --- | --- | --- | --------------- | ----------- | ----------------- | ---------- | --- |
remaininthehardwareandlatertriggeraprefetch,whichis
sufficienttosatisfyAfterImageinmostcases. However,we pagepolicy,confidenceandstrideupdatestrategyoftheIntel
IP-strideprefetcher.
provideadditionalinformationabouttheIP-strideprefetcher
thatmaybenefitfutureresearchers.
5. LeakingSecretswiththeIP-stridePrefetcher
| As we only | see the most | recent IPs | being evicted, | to de- |     |     |     |     |
| ---------- | ------------ | ---------- | -------------- | ------ | --- | --- | --- | --- |
terminewhethertheprefetcher’sreplacementpolicyisFirst- Inthissection, wepresenthowAfterImageleakscontrol
In-First-Out(FIFO)orleastrecentlyused(LRU),weupdate flowinformationacrossdifferentregionsofisolationthrough

|             |     |     |                 |          |          |            |         |     | for(int    | i = 0; i < | 3; i ++) |     |     |     |
| ----------- | --- | --- | --------------- | -------- | -------- | ---------- | ------- | --- | ---------- | ---------- | -------- | --- | --- | --- |
|             |     |     | 8-bits          | 12-bits  | 13-bits  |            | 2-bits  | 1   |            |            |          |     |     |     |
|             |     |     |                 |          |          |            |         | 2   | {          |            |          |     |     |     |
| Load Buffer |     |     | IP Last Address |          | Stride   | Confidence |         |     | IP offset1 |            |          |     |     |     |
3
|     |     |                    |     |                      |     |     |     | 4   | // to      | match if-path   |             |     |     |     |
| --- | --- | ------------------ | --- | -------------------- | --- | --- | --- | --- | ---------- | --------------- | ----------- | --- | --- | --- |
|     |     |                    |     |                      |     |     |     | 5   | int        | temp0 = array[i | * stride1]; |     |     |     |
|     |     |                    |     |                      |     |     |     |     | IP offset2 |                 |             |     |     |     |
|     |     |                    |     | IP-stride Prefetcher |     |     |     | 6   |            |                 |             |     |     |     |
|     | TLB | Invalid signal     |     | 24 entries           |     |     |     | 7   | // to      | match else-path |             |     |     |     |
|     |     | IP’s lowest 8-bits |     |                      |     |     |     |     | int        | temp1 = array[i | * stride2]; |     |     |     |
|     |     |                    |     | Fully associative    |     |     |     | 8   |            |                 |             |     |     |     |
|     |     |                    |     |                      |     |     |     | 9   | }          |                 |             |     |     |     |
Page walking
Gadget
PrefetchGenerator Listing 6: used in variant 1 and variant 2. Note that
if-elseconditionsarenotrequiredinthegadgetusedbythe
L1D Cache attacker, onlytheinstructionaddressesoftheloadsneedto
|     | 8-way 32 KB |     |                   |     |     |     |     |        |     |                  |         | (stride1 |     | stride2) |
| --- | ----------- | --- | ----------------- | --- | --- | --- | --- | ------ | --- | ---------------- | ------- | -------- | --- | -------- |
|     |             |     | Prefetchedrequest |     |     |     |     | match. | The | use of different | strides |          |     | and      |
allowtheattackertodifferentiatethetwocases.
strideprefetchertoprefetchdatalocatedatcurrentaddress+
Figure7: Thereverse-engineeredmicroarchitectureofIntel’s
|          |           |            |       |     |          |     |       | stride. | Thekeytothismethodisthatevenifthebaseaddress |     |     |     |     |     |
| -------- | --------- | ---------- | ----- | --- | -------- | --- | ----- | ------- | -------------------------------------------- | --- | --- | --- | --- | --- |
| advanced | IP-stride | prefetcher | found | in  | Haswell, | Sky | Lake, |         |                                              |     |     |     |     |     |
thevictimusesisunknown,onecandistinguishthecontrol
KabyLake,andCoffeeLake.
flow,andthereforethesecretcondition,bythestridevalue
| thecarefultrainingoftheIP-strideprefetcher. |     |     |     |     |     | WebaseAfter- |     | alone. |     |     |     |     |     |     |
| ------------------------------------------- | --- | --- | --- | --- | --- | ------------ | --- | ------ | --- | --- | --- | --- | --- | --- |
ImageonthecharacteristicsdiscussedinSection4andthree To accomplish the attack, both Prime+Probe [38] and
keyobservationswithrespecttotheIP-strideprefetcher’sstate, Flush+Reload[63]canbeadoptedtoeffectivelyobservethe
to be discussed below, each of which forms the basis for a stridetriggeredbythevictimthatisnowpresentinthecache.
variantoftheside-channel. Westartwiththetargetedcode IntermsofaPrime+Probeimplementation,aftermis-training,
regionexample,asshowninListing5. Whentheprogram’s we prime the LLC sets (with methods introduced in Sec-
controlflowisdeterminedbyasecretvalue,andloadinstruc- tion3.2)andrecordtheaccesstimeforeachminimaleviction
tionsexistintheif-elsebranch,AfterImagecanextractitand set (MES) to an array called prime_time. After victim ex-
leakit. Inthefollowingsections,wedetailthethreekeyob- ecution, we probe the LLC on a cache-line granularity and
servations of this work, and introduce the three variants of record the access time for MESs to another array, named
AfterImage: (a) attack the victim from same address space probe_time. WithEq.1,theattackercansimplyobservethe
butdifferentcoderegions,(b)attackthevictimfromdiffer- timingvariations(tv)inthesystemtocollectwhichcachesets
entaddressspace,and(c)attacktheOperatingSystem(OS) haveelementsthathavebeenevicted,i.e.,whichcachesets
acrosstheuser-kernelboundary. Whensuccessfullyexecuted, havebeenaccessedbythevictim.
thesetechniquesrendernoobservablecontrolflowchanges
or anomalous branch-speculative executions, and therefore tv=prime_time−probe_time (1)
havetheabilitytobypassseveralrecentlyproposeddefenses
|     |     |     |     |     |     |     |     | 5.2. | AfterImagevariant2 |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ------------------ | --- | --- | --- | --- | --- |
againsttransientattacks,e.g.,controlflowintegrity[28,34],
andaccesscontrolrestrictions[43,54,60].
|     |     |     |     |     |     |     |     | Observation |     | 2: The | IP-stride | prefetcher | is  | shared by |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ------ | --------- | ---------- | --- | --------- |
5.1. AfterImagevariant1 several processes that operate on the same physical
|     |     |     |     |     |     |     |     | core. | Process | B may | utilize | the entry | of the | IP-stride |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | ------- | ----- | ------- | --------- | ------ | --------- |
Observation1: TheIP-strideprefetchertrainedbyIP1 prefetchertrainedbyprocessAiftheirleastsignificant
| canbetriggeredbyIP2,aslongastheleastsignificant |     |     |     |     |     |     |     | 8-bitsofIPmatch. |     |     |     |     |     |     |
| ----------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- |
8-bitsofIP2matchthoseofIP1.
AfterImagevariant2demonstratestheabilitytoleakthe
Thegoalofthisvariantistoleakthevictim’scontrolflow controlflowofthevictimfromadifferentaddressspace. We
|      |           |      |                |      |         |        |     | implement | variant | 2 using | the | Flush+Reload |     | side-channel, |
| ---- | --------- | ---- | -------------- | ---- | ------- | ------ | --- | --------- | ------- | ------- | --- | ------------ | --- | ------------- |
| from | different | code | regions in the | same | address | space. | We  |           |         |         |     |              |     |               |
firstdesignagadget(seeListing6)thatconsistsoftwoload where the secret-dependent load instructions of the victim
instructions with different IPs. The least significant 8-bits accessesdatainsharedmemory.
of IPs of these two load instructions are tailored to match To attack the victim from another process, we build the
thememoryaccessinstructionsinif-pathandelse-pathinthe gadgetusedinvariant1inouraddressspaceandagaintrain
victim’s code region, respectively, i.e., line 3 and line 8 in thetwoIPswithdistinctstridesequences(e.g.,8forIP1aand
Listing5. Byexecutingthegadgetwithtwoconstant-strided 11forIP2a)(step 1 inFigure8).Aftertrainingtheprefetcher
memoryaddresssequencesfortwoloadinstructions,wepush to a high confidence, the shared pages is then flushed us-
twoentriesintotheIP-strideprefetcherforthesetwoIPsand ing clflush instruction. If the victim process executes the
saturate their confidence counters for their specific strides. secret-dependentbranches(step 2 inFigure8),basedonthe
Based on the Observation 1, the victim will trigger the IP- Observation2,asthelowest8-bitsofsecret-dependentmem-

|     |       |                           |     |     |                  |                  |     |     | void IP_matching(void |                          | * addr, | int group_num) |     |     |
| --- | ----- | ------------------------- | --- | --- | ---------------- | ---------------- | --- | --- | --------------------- | ------------------------ | ------- | -------------- | --- | --- |
|     |       | Attacker                  |     |     |                  | Victim           |     | 1   |                       |                          |         |                |     |     |
|     |       |                           |     |     |                  |                  |     | 2   | { asm                 | (                        |         |                |     |     |
|     | f o r | ( i n t i = 0; i< n; i++) |     |     | i f ( s ec r et) |                  |     |     | " g                   | r o u p 0 : "            |         |                |     |     |
|     |       |                           |     |     | {                |                  |     | 3   |                       |                          |         |                |     |     |
|     | {     |                           |     |     |                  |                  |     | 4   | / /                   | j u m p t o the specific | group   |                |     |     |
|     | . o   | f f s et 1                |     |     | I P 1 v :        | Memory[address]; |     |     |                       |                          |         |                |     |     |
|     | IP1a: | Memory[i* stride1];       |     |     | }                |                  |     | 5   | "cmp                  | 0, group_num"            |         |                |     |     |
|     | . o   | ff s et2                  |     |     | e l s e          |                  |     |     | " j                   | n e g r o up 1 "         |         |                |     |     |
|     | IP    | 2 a : Memory[i* stride2]; |     |     | {                |                  |     | 6   |                       |                          |         |                |     |     |
|     |       |                           |     |     | I P 2v:          | Memory[address]; |     | 7   | / /                   | c re a t e 2 4 IPs       |         |                |     |     |
|     | }     |                           |     |     |                  |                  |     |     | ".rept                | 24"                      |         |                |     |     |
|     |       |                           |     |     | }                |                  |     | 8   |                       |                          |         |                |     |     |
|     |       |                           |     |     |                  |                  |     | 9   | "mov                  | (addr), rax"             |         |                |     |     |
❶
|     |     | ❸   |     |     |     |     |     | 10  | //train | each IP on   | different | page |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | ------------ | --------- | ---- | --- | --- |
|     |     |     |     |     |     | ❷   |     |     | "add    | 0x1000 addr" |           |      |     |     |
11
12 ".endr"
|     |     |     | Shared Memory |     |     |     |     |     | "group1:" |     |     |     |     |     |
| --- | --- | --- | ------------- | --- | --- | --- | --- | --- | --------- | --- | --- | --- | --- | --- |
13
14 ...
15 ) }
| Figure8: |          | Variant2: | Mis-trainingtheIP-strideprefetcherfrom |       |     |                |     |     |     |     |     |     |     |     |
| -------- | -------- | --------- | -------------------------------------- | ----- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
| another  | process. | Note      | that code                              | shown | in  | the attacker’s | re- |     |     |     |     |     |     |     |
Listing7:IPmatchingfunction.
gionrepresentsourgadgetandthatIP1amatchesIP1vand
IP2amatchesIP2v.
|     |     |     |     |     |     |     |     | stridedmemoryaddresses. |     |     | Aftertraining,weusetheclflush |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------- | --- | --- | ----------------------------- | --- | --- | --- |
instructiontowritebackthesharedmemoryfromalllevelsof
Shared Page
|     |     |     |     | head |     |     |     | cache. | Next,wetriggerasystemcallandswitchintokernel   |     |     |     |     |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | ------ | ---------------------------------------------- | --- | --- | --- | --- | --- |
|     |     |     |     | ……   |     |     |     | mode.  | Duringthesyscallservice,ifthebranchinourtarget |     |     |     |     |     |
8th cache block
Victim Reload Attacker vulnerablecodesegmentisresolvedastaken,thefollowing
……
|     |     |     |     |     |     |     |     | load | instruction | can trigger | a prefetch |     | with our | designated |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ----------- | ----------- | ---------- | --- | -------- | ---------- |
16th cache block stride, which completely happens in the kernel mode. The
……
tail final step is to reload the shared memory and detect the ac-
cesstimetoseewhetherthetrainedaccess,andstridedoffset,
Figure9:Exampleofstridedetectioninasharedmemory.The
existsinthecache.
| attacker |      | detects that | both the | 8th and  | 16th     | cache | line in the |             |     |          |          |               |     |         |
| -------- | ---- | ------------ | -------- | -------- | -------- | ----- | ----------- | ----------- | --- | -------- | -------- | ------------- | --- | ------- |
|          |      |              |          |          |          |       |             | IPmatching. |     | SinceIPs | ofsystem | callfunctions |     | arenor- |
| shared   | page | are cached,  | and      | observes | a stride | at    | 8, which    |             |     |          |          |               |     |         |
matches the stride that is used to train the prefetcher. Note mallyunknowntotheuserorhardtodetermine,wecannot
|     |     |     |     |     |     |     |     | directly | deduce | the offset | in  | the gadget. | In this | case, we |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------ | ---------- | --- | ----------- | ------- | -------- |
thattrainingoftheprefetchstridebytheattackerisnotshown.
designedanIPsearchmethodtocreatethecorrectlymatched
oryinstructions’IPs(IP1vorIP2v)willbeindexedtoanentry
IPasthetrainingobject,whichhasthesamefunctionofthe
thatistrainedbyIP1aorIP2a,theIP-strideprefetcherwill gadget. This IP should have the same least significant 8-
prefetchthecachelinefromcurrent address + strideinto bitswiththeloadinstructionofthesyscallfunction, which
thecache. Aftervictimexecutedtheinterestedbranch,wecan fortunately shrinks the searching space to 256 possibilities.
reloadthesharedmemoryandcheckwhetherthepre-setstride Due to the hardware capacity limitation, we search for the
| existsamongthecachedlines. |     |     |     | BecausetheIP1aandIP2aare |     |     |     |     |     |     |     |     |     |     |
| -------------------------- | --- | --- | --- | ------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
IPingroups,24IPsasagrouptofitthesizeoftheIP-stride
trainedwithdifferentstrides,wereadilydiscoverthevictim’s prefetcherasindicatedinSection4.
run-timecontrolflow(seeFigure9).
|     |     |     |     |     |     |     |     | Listing7showstheIPcreatingandsearchingfunction. |     |     |     |     |     | For |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- | --- | --- |
eachround,wetrainonesinglegroupofIPsbyloopingthis
5.3. AfterImagevariant3
|     |     |     |     |     |     |     |     | function | several | times with | strided | addr | values | and a fixed |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------- | ---------- | ------- | ---- | ------ | ----------- |
Observation3: Whenaprocessswitchesbetweenuser group_num. 24 IPs in this group will be trained simultane-
ouslywiththesamestrideondifferentpages.Whenthecorrect
|     | and kernel | privilege | modes, | the | trained | entries | of the |     |     |     |     |     |     |     |
| --- | ---------- | --------- | ------ | --- | ------- | ------- | ------ | --- | --- | --- | --- | --- | --- | --- |
IP-strideprefetcherisretained. groupgettrained,thesyscallcantriggerastridedfootprintin
|     |     |     |     |     |     |     |     | thecacheifitexecutestheveryloadinstruction. |     |     |     |     |     | Thisprocess |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------- | --- | --- | --- | --- | --- | ----------- |
canberepeatedformultipletimesuntilamatchedgroupis
Thestrictisolationbetweenkernelandusermodeprotects
privileged hardware and system status from being exposed foundincaseoftoomanynottakenduringthetesting.
to users. Sensitive information in the kernel represents the ThisIPmatchingmethodisgeneralwhenthevictim’sIP
datathatshouldnotbevisibletoanarbitraryuser,e.g.,tokens, is difficult to be accessed. It completely runs as a normal
passwords, andencryptionkeys. Thisvariantdemonstrates privatefunctionwithouttouchinganydatatowhichitisnot
howtheIP-strideprefetchercanbridgetheisolationgapbe- privileged. Speciallyforkernelfunctions,oncethethesystem
tweenkernelanduserspacewhichleadstoaside-channelto is booted, the IPs of instructions will not be changed. This
potentiallyleakkernelsecrets. searchingprocessamongdifferentgroupsisnotnecessaryif
Our third variant is built on top of Flush+Reload. To es- anotherroundofAfterImageislaunched.
tablish this kernel-user side-channel, we first create a load Attacking. Todemonstrateitsfeasibility,webuildasimple
instructionwiththesameleastsignificant8bitsoftheIPas kernelfunctionasastraightforwardexampletodemonstrate
foundinthetargetcodeofthekernelfunction(inthisexample, how the prefetcher can leak information between user and
asystemcall). WetrainthisIPintheuser’saddressspacewith kernelspace. Listing8presentsthecustomizedkernelfunc-

|     | void vulnerable_syscall1(void* |     | memory_space) |     |     |     |     |     |         |     |     |         |     |
| --- | ------------------------------ | --- | ------------- | --- | --- | --- | --- | --- | ------- | --- | --- | ------- | --- |
| 1   |                                |     |               |     |     |     |     |     | i7-4770 |     |     | i7-9700 |     |
2 {
|     | int num | = random(); |     |     |     | Architecture |     |     | Haswell |     |     | CoffeeLake |     |
| --- | ------- | ----------- | --- | --- | --- | ------------ | --- | --- | ------- | --- | --- | ---------- | --- |
3
| 4   | if(num) |            |                            |     |     | CPUcores       |     |     |     | 4   |     | 8    |     |
| --- | ------- | ---------- | -------------------------- | --- | --- | -------------- | --- | --- | --- | --- | --- | ---- | --- |
| 5   | {       |            |                            |     |     | LastLevelCache |     |     | 8MB |     |     | 12MB |     |
|     | char    | *address = | get_address(memory_space); |     |     |                |     |     |     |     |     |      |     |
6
| 7   | Memory[address]; |     |     |     |     | OperatingSystem |     |     | Ubuntu18.04 |     | Ubuntu18.04 |     |     |
| --- | ---------------- | --- | --- | --- | --- | --------------- | --- | --- | ----------- | --- | ----------- | --- | --- |
}
| 8   |        |     |     |     |     |     | ASLR  |     | Level-2Enabled |     | Level-2Enabled |         |     |
| --- | ------ | --- | --- | --- | --- | --- | ----- | --- | -------------- | --- | -------------- | ------- | --- |
| 9   | return | 0;  |     |     |     |     |       |     |                |     |                |         |     |
|     |        |     |     |     |     |     | KASLR |     | Enabled        |     |                | Enabled |     |
10 }
|     |     |     |     |     |     |     | DRAM |     | DDR4,2x4G |     | DDR4,2x8G |     |     |
| --- | --- | --- | --- | --- | --- | --- | ---- | --- | --------- | --- | --------- | --- | --- |
Listing8:Thecustomizedkernelfunction.
Table3:Architectureandsystemconfigurations.
tion. Inthisfunction,numrepresentsthesecretinthekernel
|                    |                                           |                                  |     |     |     |          | Boundary     |     |     | Flush+Reload |     | Prime+Probe |     |
| ------------------ | ----------------------------------------- | -------------------------------- | --- | --- | --- | -------- | ------------ | --- | --- | ------------ | --- | ----------- | --- |
| anddeterminestheif |                                           | branch,inwhichaloadinstructionis |     |     |     |          |              |     |     |              |     |             |     |
|                    |                                           |                                  |     |     |     |          |              |     |     | (cid:51)     |     | (cid:51)    |     |
|                    |                                           |                                  |     |     |     | variant1 | coderegion   |     |     |              |     |             |     |
| followed.          | Thesyscallfunctionsharesmemorywithuservia |                                  |     |     |     |          |              |     |     |              |     |             |     |
|                    |                                           |                                  |     |     |     | variant2 | processspace |     |     | (cid:51)     |     |             | -   |
amemory_spaceparameterthatallowsFlush+Reloadtotake
|     |     |     |     |     |     | variant3 | user-kernel |     |     | (cid:51) |     |     | -   |
| --- | --- | --- | --- | --- | --- | -------- | ----------- | --- | --- | -------- | --- | --- | --- |
place.
AfterthetargetgroupofIPsgetswell-trainedintheafore- Table4:Allvariants’featuresandcurrentsupportedmeasure-
|                                      |        |              | clflush |                 | mentmodels. |     |     |     |     |     |     |     |     |
| ------------------------------------ | ------ | ------------ | ------- | --------------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
| mentioned                            | phase, | the attacker | calls   | instruction to  |             |     |     |     |     |     |     |     |     |
| flushthesharedchunkofdataoutofcache. |        |              |         | Thenitcallsthis |             |     |     |     |     |     |     |     |     |
syscallandpassesthecontrolrighttothekernel. Ifthebranch 1 and 2, we directly obtain the last 8 bits of the IP of the
|     |     |     |     |     | victim’s |     | load instructions |     | from | disassembly |     | tools, | such as |
| --- | --- | --- | --- | --- | -------- | --- | ----------------- | --- | ---- | ----------- | --- | ------ | ------- |
istaken,thefollowingloadinstructionswillbeexecutedonthe
samesharedmemoryspace. TheIP-strideprefetcherautomat- objdump. Forvariant3,weuseamoregeneralIPmatching
icallychecksitshistorytableandfindsamatchedentrywitha method,discussedindetailinSection5.3.
highconfidencevalue. Therefore,itwillsendprefetchrequest Inaddition,astheAddressSpaceLayoutRandomization
tothenextaddress,whichiscurrentaddress + stride. When (ASLR) or Kernel ASLR (KASLR) on Linux has at least
|     |     |     |     |     | a   | granulatiry | on  | one page | (assuming |     | a page-size |     | of 4KiB), |
| --- | --- | --- | --- | --- | --- | ----------- | --- | -------- | --------- | --- | ----------- | --- | --------- |
thesyscallserviceisfinished,processgoesbacktotheuser
state. Theattackerreloadsthedatatoseewhichaddressesare thesetechniqueswillnotchangetheleastsignificant12bits
cached. Iftwoaddresseswithourselectedstridearebothhit, oftheIPs. SincetheIP-strideprefetcherusesthelowest8-bits
wecaninferthatthisbranchhasbeentakenbythekernel,and to index its history, ASLR does not impact IP matching or
| viceversa. |     |     |     |     | gadgetbuilding. |              |      |       |              |     |        |     |         |
| ---------- | --- | --- | --- | --- | --------------- | ------------ | ---- | ----- | ------------ | --- | ------ | --- | ------- |
|            |     |     |     |     |                 | Interference | from | other | prefetchers. |     | Except | for | the IP- |
5.4. AfterImageMitigationStrategies
|     |     |     |     |     | stride | prefetcher, |     | the other | three | hardware |     | prefetchers | can |
| --- | --- | --- | --- | --- | ------ | ----------- | --- | --------- | ----- | -------- | --- | ----------- | --- |
introducefalsepositivesintotheresultsbyloadingadditional
Weconsiderdefendingagainstvariants2and3,astheyposea
|     |     |     |     |     | cache | lines | unexpectedly. |     | Fortunately, |     | these | prefetchers | do  |
| --- | --- | --- | --- | --- | ----- | ----- | ------------- | --- | ------------ | --- | ----- | ----------- | --- |
higherthreattothesystemandapplications,whichissimilar
|                |     |                                      |     |     | not             | have | the address | range                                 | reach | compared |     | to the | IP-stride |
| -------------- | --- | ------------------------------------ | --- | --- | --------------- | ---- | ----------- | ------------------------------------- | ----- | -------- | --- | ------ | --------- |
| toSpectre[28]. |     | Weproposetwocountermeasuresthatcanbe |     |     |                 |      |             |                                       |       |          |     |        |           |
|                |     |                                      |     |     | prefetcher[59]. |      |             | TheDCU(next-line)prefetcherprefetches |       |          |     |        |           |
implementedduringacontextswitch,similartothedefenses
|                     |     |                                |     |     | onlythenextcacheline.       |     |     |     | DPL(adjacent)prefetcherprefetches |                          |     |     |     |
| ------------------- | --- | ------------------------------ | --- | --- | --------------------------- | --- | --- | --- | --------------------------------- | ------------------------ | --- | --- | --- |
| ofpreviouswork[12]. |     | Onemitigationthatcanbedeployed |     |     |                             |     |     |     |                                   |                          |     |     |     |
|                     |     |                                |     |     | thepreviousornextcacheline. |     |     |     |                                   | Andthestreamerprefetches |     |     |     |
immediately,istotraineachentryoftheIP-strideprefetcherto
thepreviousornextseveralsequentialcachelines.Noiseintro-
evictallpotentialentriesthatcouldbeusedasaside-channel.
ducedbytheseprefetcherscanbeeasilydistinguishedwhen
Whilethisstepdoesrequiremicroarchitectureknowledgeand
usinglargemultiplesofcachelinesastheprefetchtrigger(see
willaddadditionaldelayduringcontextswitches,itprovides
thenextparagraphfordetails).
| animmediatedefenseapplicabletotoday’shardware. |     |     |     | Asan |     |        |            |     |        |          |     |           |       |
| ---------------------------------------------- | --- | --- | --- | ---- | --- | ------ | ---------- | --- | ------ | -------- | --- | --------- | ----- |
|                                                |     |     |     |      |     | Choice | of stride. | Due | to the | presence | of  | the three | other |
alternative,wealsoproposeamorelightweightdefensethat
hardwareprefetchers,weuseastridethatisgreaterthanfour
| requires | hardware | support. | More concretely, | we propose a |             |     |                                           |     |     |     |     |     |     |
| -------- | -------- | -------- | ---------------- | ------------ | ----------- | --- | ----------------------------------------- | --- | --- | --- | --- | --- | --- |
|          |          |          |                  |              | cachelines. |     | Additionally,theuseofuncommonstridevalues |     |     |     |     |     |     |
privilegedclear-ip-prefetcherinstructionthatcanbeused
|             |                                            |               |             |                  | (e.g.,                                          | a larger | prime | number) |     | will provide |     | additional | noise    |
| ----------- | ------------------------------------------ | ------------- | ----------- | ---------------- | ----------------------------------------------- | -------- | ----- | ------- | --- | ------------ | --- | ---------- | -------- |
| on          | a context switch                           | to invalidate | all entries | in the IP-stride |                                                 |          |       |         |     |              |     |            |          |
|             |                                            |               |             |                  | resiliencebecausetheycanbeeasilydifferentiated. |          |       |         |     |              |     |            | Inourex- |
| prefetcher. | SeeSection7forthenewinstructionevaluation. |               |             |                  |                                                 |          |       |         |     |              |     |            |          |
periments,wegenerallytraintheprefetcherwithstridevalues
| 6.  | ExperimentalSetup |     |     |     | of7,11and13. |              |     |               |     |     |             |     |       |
| --- | ----------------- | --- | --- | --- | ------------ | ------------ | --- | ------------- | --- | --- | ----------- | --- | ----- |
|     |                   |     |     |     |              | Side-channel |     | preparations. |     | The | Prime+Probe |     | side- |
Experimentalenvironment. WeperformProof-of-Concept channel requires the calculation of minimal eviction sets
(PoC)side-channelexperimentsonHaswellandCoffeeLake (MESs) to occupy the LLC cache. We utilize the slice-
machines. ThearchitecturedetailsandOSconfigurationof selection algorithm found in the Haswell microarchitec-
thesetwomachinesisshowninTable3.
ture[26]togeneratetheMESstocovermultiplecachesets.
Gadgetbuilding. Wehaveusedseveralmethodstobuild Ifprobingisperformedwitharegularpattern,itwilltrigger
thegadgettotraintheIP-strideprefetcher. Intermsofvariant prefetchingandintroducemanyfalsepositives. Therefore,we

untouched sets secret-related loads prefetched sets untouched sets secret-related loads prefetched sets untouched lines secret-related loads
| noise              |     | LLC hit threshold |     |     |                    |     |                   |     |     |                 |                  |           |     |                   |     |
| ------------------ | --- | ----------------- | --- | --- | ------------------ | --- | ----------------- | --- | --- | --------------- | ---------------- | --------- | --- | ----------------- | --- |
|                    |     |                   |     |     | noise              |     | LLC hit threshold |     |     |                 | prefetched lines |           |     | LLC hit threshold |     |
| 600                |     |                   |     |     | 800                |     |                   |     |     | 300             |                  |           |     |                   |     |
| 500                |     |                   |     |     | noitairaV emiT 700 |     |                   |     |     |                 |                  |           |     |                   |     |
| noitairaV emiT 400 |     |                   |     |     | 600                |     |                   |     |     | emiT sseccA 250 |                  |           |     |                   |     |
| 300                |     |                   |     |     | 500                |     |                   |     |     | 200             |                  |           |     |                   |     |
| 200                |     |                   |     |     | 400                |     |                   |     |     |                 |                  |           |     |                   |     |
| 100                |     |                   |     |     | 300                |     |                   |     |     | 150             |                  |           |     |                   |     |
|                    |     |                   |     |     | 200                |     |                   |     |     |                 |                  | if-path   |     |                   |     |
| 0                  |     |                   |     |     | 100                |     |                   |     |     | 100             |                  |           |     |                   |     |
| -100               |     |                   |     |     | 0                  |     |                   |     |     |                 |                  |           |     |                   |     |
| -200               |     |                   |     |     | -100               |     |                   |     |     | 50              |                  |           |     |                   |     |
|                    |     |                   |     |     | -200               |     |                   |     |     | 0               |                  | else-path |     |                   |     |
-300
| 0   | 10 20 | 30         | 40 50 | 60  | 0   | 10  | 20         | 30 40 | 50  | 60  | 0 10 | 20  | 30         | 40 50 | 60  |
| --- | ----- | ---------- | ----- | --- | --- | --- | ---------- | ----- | --- | --- | ---- | --- | ---------- | ----- | --- |
|     |       | #Cache Set |       |     |     |     | #Cache Set |       |     |     |      |     | #Cache Set |       |     |
(a)attackif-path. (b)attackround-by-roundwithPrime+Probe. (c)attackround-by-roundwithFlush+Reload.
(a)singlebitextractionfromif-path(b)round-by-roundextractionfromreal
| Figure10: | AttackresultsofAfterImagevariant1: |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| --------- | ---------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
executionflowwithPrime+Probe.(c)round-by-roundextractionfromrealexecutionflowwithFlush+Reload.
organizeourMESelements,eachwiththesizeofonecache
|                                     |                      |     |         |                     |      |         |     |             | untouched sets |     | secret-related loads |     | prefetched sets |     |     |
| ----------------------------------- | -------------------- | --- | ------- | ------------------- | ---- | ------- | --- | ----------- | -------------- | --- | -------------------- | --- | --------------- | --- | --- |
|                                     |                      |     |         |                     |      |         |     |             | noise          |     | LLC hit threshold    |     |                 |     |     |
| line,intoalinked-listdatastructure. |                      |     |         | Weperformtheprobing |      |         |     |             | 400            |     |                      |     |                 |     |     |
| traversal                           | in a pointer-chasing |     | manner, | and                 | will | prevent | the |             | 350            |     |                      |     |                 |     |     |
|                                     |                      |     |         |                     |      |         |     | emiT sseccA | 300            |     |                      |     |                 |     |     |
hardwareprefetcherfromgeneratingrequests[15,56].
250
| For the | same | reason, | Flush+Reload | can | also | introduce |     |     |     |     |     |     |     |     |     |
| ------- | ---- | ------- | ------------ | --- | ---- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
200
| noise. When | we  | reload a | shared | page, | instead | of loading |     |     | 150 |     |     |     |     |     |     |
| ----------- | --- | -------- | ------ | ----- | ------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
if-path
| thecachelinesequentially,weusethemodernversionofthe |     |     |     |     |     |     |     |     | 100 |     |     |     |     |     |     |
| --------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
50
| Fisher–Yates | shuffle | algorithm | [16] | to randomize |     | the index |     |     |     |     |     |     |     |     |     |
| ------------ | ------- | --------- | ---- | ------------ | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
0
sequence in the searching range (i.e., [0,63]). In addition, 0 10 20 30 40 50 60
we add mfence and lfence when we measure the memory #Cache Set
instructionexecutiontimetoensurethatmeasuredmemory
Figure11:AttackresultofAfterImagevariant2.
andcacheaccesstimeisaccurate[14].
We note that the insight of this paper, i.e. the IP-stride length unit. The y-axis shows the time taken, between the
probingphaseandprimingphase,toaccesseachMESofthe
| prefetcher | can be | intentionally | trained | to  | leak information, |     |     |     |     |     |     |     |     |     |     |
| ---------- | ------ | ------------- | ------- | --- | ----------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
anditsdemonstrationisourfocus. Thechoiceoftheunder- cache set. As depicted in the figure, most cache sets have
|     |     |     |     |     |     |     |     | not been | accessed | as their | access | latency | is  | lower than | 120 |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | -------- | -------- | ------ | ------- | --- | ---------- | --- |
lyingtimingside-channelusedtoobservecacheactivitiesis
|                  |        |     |             |     |               |     |     | cycles. | Thetwocachesetswiththehighesttimedeltahasa |     |     |     |     |     |     |
| ---------------- | ------ | --- | ----------- | --- | ------------- | --- | --- | ------- | ------------------------------------------ | --- | --- | --- | --- | --- | --- |
| not as critical. | Except | for | Prime+Probe | and | Flush+Reload, |     |     |         |                                            |     |     |     |     |     |     |
othermeasuringmethodssuchasFlush+Flush[23]canapply clearstrideof7,demonstratingthatthetargetedloadonthe
if-pathwasexecuted,andtriggeredanIP-strideprefetchthat
| tothisworkaswell. |     | AshasbeenintroducedinSection3.2, |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ----------------- | --- | -------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Prime+Probeislessnoise-resilientthanFlush+Reloadbyna- wastrainedtobeadistanceof7cachelines.
Notethattheprefetchercanprefetchdatathatisbeyondthe
| ture. We | experimentally |     | discover | that the | process | of  | con- |     |     |     |     |     |     |     |     |
| -------- | -------------- | --- | -------- | -------- | ------- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
text switching and user-kernel mode switching in variant 2 boundaryofanarray[25],thereforetheattackerisstillable
and3tendstobringintolerablenoisetoaPrime+Probeside- toobservethestrideeventhecurrentaddressisclosetothe
channel, i.e., over half of MESs are touched by the system. endofarrayofthevictim(buttheprefetchedaddressshould
Therefore,wepresentourexperimentresultsofvariant1with notcrosspageboundary). Asthebranchpredictortypically
Prime+Probe and Flush+Reload, and variant 2 and 3 with exhibitsahighaccuracyinitspredictions,wegenerallysee
Flush+Reloadinthenextsection. Table4presentsalistof onlyonestridewhenthevictimexecutesoneofthebranches.
| thesuccessfulexperimentsconductedacrossthevariantsof |     |     |     |     |     |     |     |         |      |              |     | gadget |          |                |     |
| ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ------- | ---- | ------------ | --- | ------ | -------- | -------------- | --- |
|                                                      |     |     |     |     |     |     |     | We then | call | the proposed |     |        | to train | the prefetcher |     |
AfterImage. forbothpathsandtrytoconsistentlyleakcontrolflowround-
|     |     |     |     |     |     |     |     | by-round. | The | synchronization |     | with | the victim | can | rely on |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | --------------- | --- | ---- | ---------- | --- | ------- |
7. Experimentalresults
simultaneousmultithreadingoronaccuratetime-multiplexing.
|     |     |     |     |     |     |     |     | From Figure | 10b | and | Figure | 10c, we | observe | clear | signals |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | --- | ------ | ------- | ------- | ----- | ------- |
Inthissection,wepresentourexperimentresults,fromvariant
1to3. Forvariant1,weshowtheresultsforbothifandelse (strides) after the victim performed the branch. During the
|     |     |     |     |     |     |     |     | first round, | we  | see that | the victim | took | the | else-path. | The |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | -------- | ---------- | ---- | --- | ---------- | --- |
branch, onaPrime+ProbeandFlush+Reloadmeasurement.
|     |     |     |     |     |     |     |     | victimthenexecutedtheif-pathinthefollowingcycle. |     |     |     |     |     |     | Ifthe |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- | --- | ----- |
Forvariant2and3,wepresentFlush+Reloadmeasurement.
Intheend,wetestoursuccessrateforallofthethreevariants. branchissecurity-related,wethenknowthesecretisB’10.
|        |           |                  |     |        |     |         |     | From | Figure | 10, we | observe | some | noise | in the | resulting |
| ------ | --------- | ---------------- | --- | ------ | --- | ------- | --- | ---- | ------ | ------ | ------- | ---- | ----- | ------ | --------- |
| Figure | 10a shows | the experimental |     | result | of  | leaking | the |      |        |        |         |      |       |        |           |
if-pathviaPrime+Probeafteroneroundofobservation. The output,butourdetectionmethodusesaknownstridelength,
allowingforahighaccuracy.
x-axisrepresentsthecachesetnumberinourobservingpage
(4KiBpagewith64cachelines),whosedistancedirectlyrep- TheProof-of-Concept(PoC)implementationforvariant2
resentstheoffsetbetweenmemoryaddressesinacache-line- demonstratesacross-processattack. Intheattackerprocess,

450
400
350
300
250
200
150
100
50
0
0 10 20 30 40 50 60
emiT
sseccA
untouched lines secret-related loads prefetched lines
noise LLC hit threshold 1.8 1.6 1.078 1.411.4
1 1 . . 2 4 1.076
1
0.8
0.6
0.4
0.2
0
if-path
#Cache Set
Figure12:AttackresultofAfterImagevariant3.
stridesforthetrainingofifandelse-patharesetto7and13,
respectively,asinvariant1. Figure11showstheexperiment
resultsextractedbytheattackerprocess, withthecacheset
againonthex-axis, andtheaccesstimeforeachsetonthe
y-axis. Usingthistechnique,weareabletodetectthecontrol
flowdifferencesbyobservingthestridesforlow-access-time
cachesets.
IntermsofAfterImagevariant3,weperformtheIPsearch
asdescribedinSection5.3,andcreate20groupsof24load
instructions to guarantee the 256 possibilities are covered2.
Whenamatchedgroupisfound,weperformtheside-channel
workflow,ashasbeenintroducedin5.3. Inthisexample,we
settrainingstrideintheuserspaceto11cachelines. Inthe
userreloadingphase,thedetectedstride,showninFigure12,
indicatesthatthekernelfunctionexecutedtheif-pathandthus
thevalueofnumis1.
Tofurtherevaluatetheattacksuccessrate,weevaluatethe
threevariantswith200roundsonasetofsampledata. We
conductthisevaluationontheplatformdescribedinTable3,
whichisbootedinthenormalmodewithoutadditionaluser
programsrunning. Theattacksuccessrateofvariant1,variant
2,andvariant3are99%,97%,and91%,respectively.
Hardware Mitigation Results. To demonstrate the low
overheadofourproposedinstruction-baseddefence,weim-
plemented it using ChampSim [2], a cycle-level simulator
andtheplatformusedforevaluatinganumberofcomputer
architecturechampionships[1,30,39,40].
WeconfigureChampSimtomodelaCoffeeLake-likepro-
cessor, and implement the Intel IP-stride prefetcher (based
onourreverse-engineeringdetailedinSection4)toemulate
frequentprefetcherflushingduetocontextswitching(every
10µs)ondifferentapplicationsinordertoemulateaworst-
caseflushingscenario. Ateachcontextswitch,weflushthe
IP-stride prefetcher, and allow it to re-learn the stride. The
timetoresettheprefetcherishighlydependentbythewrite
portsofprefetcher. Forinstance,ifthewehave2writeports
and24entries, whichmeans, everycyclewecanoverwrite
two entries. To clean all slots, we need 12 cycles. To ap-
proach the worst case, we set the prefetcher to have only 1
writeport,andthuswewillneed24cyclestoresetallentries.
2Sincethereare480instructionsarecreatedintotal,someIPsmayfall
intomorethanonegroupandanyofthemcanserveasthetrainingobject.
CPI
dezilamroN
baseline IP-stride Selective flushing
Figure13: Performanceimpactforproposedmitigation. Note
that the baseline configuration represents a processor with-
out prefetching, IP-stride denotes the processor with an IP-
stride prefetcher, configured as reverse-engineered, and se-
lective flushing is the processor that enables our proposed
defensewithaworst-caseperiodof10µµµs.
Usingthistechnique,wethenmeasuretheperformanceofthe
SPEC CPU2006 and 2017 [7, 8] benchmarks, with a focus
on prefetching-sensitive applications. We run 30 billion in-
structionsforeachapplicationunlessitcompletesearlyand
thenreportthenormalizedIPC.Theoverheadintroducedby
theproposeddefense(SeeFigure13)isnegligible, withan
averageperformancereductionofonly0.7%forprefetching-
sensitiveapplications(thatutilizetheIP-strideprefetcher)and
0.2%acrossalltestedapplications. Overheadislowbecause
of the extremely low number of training iterations needed
for this prefetcher (which we expect to translate to kernel
functionsaswell).
8. Discussion
8.1. ImpactfromExistingDefenses
WefirstdiscusstheeffectivenessofAfterImageinthepresence
ofexistingdefensesthataimtomitigatethethreatbroughtby
microarchitecturalside-channelattacks.
Control flow integrity. Many existing defenses are pro-
posed based on control flow integrity [11, 28, 34]. These
protectionmodelswillautomaticallycheckwhethertherun-
timecontrolflowisdeviatingfromthethecontrolflowgraph.
Ifthemaliciousbranchisdiscovered,thosepreviousproposals
willenabletheirdefense(e.g.,obfuscatedexecution,disabling
thespeculativeexecution,etc.). However,AfterImagedoes
notrequirespeculativecontrol-flowchanges;onlythebackend
prefetcherisaffectedbyAfterImage,andthisoccursduring
thenon-speculativepathofexecution.
Performancecounter-basedmonitoring. Leveragingper-
formancecountersprovidedbyIntel,thedefendermightbe
able to identify abnormalities or potential malicious activi-
tiesinvulnerablehardwarecomponentsduringrun-time(e.g.,
micro-opcache,BPU,L1cache,LLC,etc.)[13,48,66]. Fur-
thermore,theperformancecounterprovidesaperformance-
efficientwayofmonitoringthesystem.However,thesampling
frequencyandaccuracyofperformancemonitorprovidedby
Intel [47] may not be enough to capture the training activi-

250 based fencing/delaying solutions [34, 43, 54, 60] to restrict
MEM[r + stride1]
|     |     |     |     |     | thespeculativeloadingofsecret-relateddata. |     |     | Morespecific, |
| --- | --- | --- | --- | --- | ------------------------------------------ | --- | --- | ------------- |
200
emiT sseccA thesedefenseswillselectivelyserializingthespeculativeload
150
|     |     |     |     |     | instructions                                | by adding lfence-type | instructions | or directly |
| --- | --- | --- | --- | --- | ------------------------------------------- | --------------------- | ------------ | ----------- |
|     |     |     |     |     | delayingthespeculativeloadsintheissuequeue. |                       |              | Thesede-    |
100
M E M
MEM[r] MEM[t][t +  st r ide2] fenses are mainly effective in prohibiting the unauthorized
50
accessofsecretdataincaseoftransientexecutionattacks. Af-
0
If-matched else-matched terImage,however,doesn’trelyonbranch-speculativeexecu-
|                  |                                               |                                     |     |     | tiontoleakthesecret. | Inaddition,basedonourobservations, |                  |                |
| ---------------- | --------------------------------------------- | ----------------------------------- | --- | --- | -------------------- | ---------------------------------- | ---------------- | -------------- |
| Figure14:        | DetectingcontrolflowinAfterImagevariant1with- |                                     |     |     |                      |                                    |                  |                |
|                  |                                               |                                     |     |     | serializing          | load instructions                  | will not prevent | the prefetcher |
| outcacheprobing. |                                               | Theprefetchercannolongerbetriggered |     |     |                      |                                    |                  |                |
fromgeneratingprefetchrequests.
ifthevictimexecutesthetargetload(andbranch).
8.2. MitigationOptions
ties,sinceAfterImagerequiresjusttwotothreeiterationsof
trainingataminimum,andnoprefetchrequestwillbegener-
|     |     |     |     |     | There are | some generic | defenses that | could be deployed to |
| --- | --- | --- | --- | --- | --------- | ------------ | ------------- | -------------------- |
atedbeforeitiswell-trained. Moreconcretely,Spectreneeds mitigate all variants of AfterImage. The most straightfor-
around26,000cyclestomis-traintheBPU[29],butweneed
|     |     |     |     |     | ward protection | is to disable | the IP-stride | prefetcher. This |
| --- | --- | --- | --- | --- | --------------- | ------------- | ------------- | ---------------- |
justthreecachemisses,i.e.,1000cyclesto2000cycles(ifthe
|     |     |     |     |     | defense, | however, will | result in a significant | performance |
| --- | --- | --- | --- | --- | -------- | ------------- | ----------------------- | ----------- |
firstaccessresultsinpagefault). overhead. If possible, redesigning the application to avoid
| Protected | cache. | Other solutions | based on | randomiza- |     |     |     |     |
| --------- | ------ | --------------- | -------- | ---------- | --- | --- | --- | --- |
speculativeaccessesofsecretscanalsopreventthisissue[3],
tion [44, 45], randomize cache indexing, which may hide ifthesourceoftheapplicationisavailable. Further,oblivious
| the stride | if the | attacker would like | to construct | the mini- |           |                |              |                     |
| ---------- | ------ | ------------------- | ------------ | --------- | --------- | -------------- | ------------ | ------------------- |
|            |        |                     |              |           | execution | [46, 64] could | prevent data | leakage by removing |
mal eviction setin the Prime+Probe. AfterImage, however, anycontrolflowandmostdatadependencies. Nevertheless,
supportsusingFlush+Reloadtoobservethecachestatusvari- theuseofobliviouscodefacespracticaldifficulties,asitleads
| ation, which | also | saves the trouble | of computing | eviction |     |     |     |     |
| ------------ | ---- | ----------------- | ------------ | -------- | --- | --- | --- | --- |
tosignificantoverheadinmanyapplicationsandmaynotwork
sets. Otherworks[13,37,66]havebeenproposedtoprevent correctlyduetothelimitationsofapplicationorprogrammer
Flush+Reloadbytrackingabnormalevents(e.g.,alargenum-
|     |     |     |     |     | experience[42]. | Theuseofasecuretimerthatcanobfuscate |     |     |
| --- | --- | --- | --- | --- | --------------- | ------------------------------------ | --- | --- |
ber of fetching, a large number of LLC/L1D cache misses, thecacheaccesslatencybyaddingnoise[24,35]isanother
etc.) andbyprohibitingdataorinstructionfetching. However, way to mitigate AfterImage. But they have to be built on
theimportantthingtonoteisthattheattackercanuseAfterIm-
specifickernelorextendedISA,whichareoftencostlytoim-
ageinotherwaysbesideFlush+ReloadorPrime+Probe. For plement. Inaddition,cachingpagetableentriesofsensitive
example,accordingtoourreverseengineeringresults,after
|     |     |     |     |     | data in | a isolated cache | rather than traditional | caches (e.g., |
| --- | --- | --- | --- | --- | ------- | ---------------- | ----------------------- | ------------- |
thevictimtouchesawell-trainedIPintheprefetcher,theIP’s CATalyst[32])canalsomitigateAfterImage. However,hav-
confidence will be updated and it will no longer be able to ingaseparatecacheforpagetablepagescanintroduceahigh
| triggertheprefetcher. |     | Theattackercanre-executeallofthe |     |     |                         |     |                                 |     |
| --------------------- | --- | -------------------------------- | --- | --- | ----------------------- | --- | ------------------------------- | --- |
|                       |     |                                  |     |     | overheadinhardware[21]. |     | Ourproposedmitigationstrategies |     |
well-trained IPs to determine which IPs are no longer trig- canbeimplementedtoday,oralow-overheadversioncanbe
gered and infer the victim’s control flow, i.e., checking the implementedinfuturehardwaredesigns(SeeSection5.4).
| prefetcherstatus. |     | Byusingthismethod,insteadofreloading |     |     |     |     |     |     |
| ----------------- | --- | ------------------------------------ | --- | --- | --- | --- | --- | --- |
orprobingthewholepage(memory),theattackeronlyneeds 9. RelatedWork
| totestthelatencyofasingledestinationaddress. |     |     |     | Toverify |                    |              |         |                    |
| -------------------------------------------- | --- | --- | --- | -------- | ------------------ | ------------ | ------- | ------------------ |
|                                              |     |     |     |          | Microarchitectural | side-channel | attacks | exploit shared re- |
thefeasibilityofthemeasurementmodel,werunAfterImage
sourcesintheprocessortoleaksensitiveinformationacross
| variant1again. | InsteadofusingPrime+Probe,however,we |     |     |     |     |     |     |     |
| -------------- | ------------------------------------ | --- | --- | --- | --- | --- | --- | --- |
randomlyaccesstwomemoryaddressesusingif-matchedIP privilege domains [19, 38]. Examples such as caches, the
branchpredictorunit(BPU)andprefetchershavebeensuc-
| andelse-matchedIPinthegadget. |     |     | Theexperimentalresult |     |     |     |     |     |
| ----------------------------- | --- | --- | --------------------- | --- | --- | --- | --- | --- |
is shown in Figure 14. In this example, we find that the if- cessfullydemonstratedaspotentialsourcesofrisk.
matchedIPnolongertriggerstheprefetcher,whichimplies
9.1. CacheSide-Channels
| thatthevictimhasexecutedtheif-path.                  |     |     | Onelimitationofthis |     |     |     |     |     |
| ---------------------------------------------------- | --- | --- | ------------------- | --- | --- | --- | --- | --- |
| measurementisthepotentiallyhigherfalse-positiverate. |     |     |                     | For |     |     |     |     |
Cacheside-channels,e.g.,Evict+Time[38],Flush+Flush[23],
example, the prefetcher also might not be triggered due to Flush+Reload[63],Prime+Probe[38,41],utilizethetiming
memoryaccessesatthecontextswitchorbecauseofalarge variance caused by different memory behaviors. The large
amount of activity. This technique, however, allows us to latencydifferencebetweenacachehitandacachemissallows
bypassthedefenseagainstFlush+ReloadorPrime+Probe. forhighresolution,lownoiseobservations. Inaddition,the
Accesscontrol. Thereareseveralworksaimedatcreating hierarchicalstructureofthecacheallowsforinformationto
amemorysafetyzonetopreventunauthorizedaccesstoconfi- be leaked across different shared timing level, from private
dentialdata. Mostofthemleveragehardwareand/orsoftware- datacacheandinstructioncache,tothelastlevelcacheshared

byallthecores[10,38,63]. Inmanyworks, secret-related prefetcher in many aspects. For example, they are shared
activitiescanbeexhibitedonthetimingofthecachebynature amongallapplicationsoperatingonthesamecore,andthey
orbyinduction[42,48]. Therefore,cachetimingisoftenthe bothhaveconfidencecountermechanisms. Kocheretal.[27]
basisofmanifestingnewside-channels. proposedSpectretoexploitconditionalbranchmisprediction
and poison indirect branches. Although the mispredicted
9.2. PrefetchingSide-Channels
branch will not be committed in the end, system informa-
TheworkofShinetal.[51]istheclosestworkwithoursin tion,suchascacheddata,canbeaffected.Ourimplementation
leakinginformationthroughtheIP-strideprefetcherontopof issimilartoSpectre’sinmistrainingahardwarecomponentin
cache side-channels. They observe that, when the program theprocessorandexploitingcachetimingtoanalyzeresults.
itselfshowsastablememoryaccessbehavior,e.g. tablelook- Nevertheless,comparedtotheBPU,thehardwareprefetcher
ups,theIP-strideprefetchercanbetriggeredandleavespecial willnotintroduceadditionalinstructionsforexecutionorcause
footprintinthecache. Basedonthisinsight,theyareableto pipelineflushevents;itonlybringsloaddataintothecache.
leakthesecretvalueofonespecificalgorithmoftheOpenSSL TheIP-strideprefetchertakesarelativelyshortamountoftime
library[6],theECDHalgorithm[5]. Instead,inthiswork,we totraincomparedtotheBPU.Also,sinceBPUusestheleast
presentagenericmethodologywhichcanextractinformation 20bitsofIP,Spectrestillneedsseveralroundsoftestingto
from a variety of workloads using the IP-stride prefetcher. bypassASLR,whiletheIP-strideprefetcherusestheleast8
Detecting only the selected strides in the cache saves huge bitsandisnotaffected.
amountofanalyzingeffortfortheobserver. Forexample,this
state-of-the-artwork[51]canonlyfindtwocachelinesthatis 9.4. OtherMicroarchitecturalSide-Channels
relatedtothesecret,sothattheysampletheactivitiesofthese
twocachelinesintotime-seriesdataandthenadoptclustering Apartfromthetraditionalcache,BPUandprefetcher,some
algorithmtoanalyzetheircorrelationwiththesecret. otherarchitecturalcomponentshaverecentlyreceivedatten-
From the software perspective, a software prefetch side- tion as well. Ren et al. [48] reveals the characteristics of
channel [22] aims to bypass Supervisor Mode Access Pre- the micro-op cache in Intel and AMD processors and ex-
vention(SMAP)andKASLR.Toinferthemappingbetween ploits them as a timing channel to transmit secret informa-
virtual address and physical address in the system, they ex- tion. TheTLB,orTranslationLook-asideBuffer,isanother
ploitsprefetchinstructions’capabilityofleakingtimingin- sharedresourcesintheprocessor,canalsoleakinformation.
formationontheexacttranslationlevelofthevirtualaddress. AnOS-leveladversarycaninducepagefaultstoobservethe
However,themainpurposeofthatworkisdifferentfromAfter- page-levelaccesspatternsofthevictim[61]. TLBleed[20]
Image. Moreover,theyimplementtheirattackusingsoftware furthercombinesmachinelearningmethodsthatimprovesits
interface,whilewedirectlymanipulatehardwareprefetcher. granularity. CacheOut[57]detailshowanattackercanleak
In many previous security-related works, prefetching is informationcrossmultiplesecurityboundariesbyusingthe
regardedasahindrance,limitingtheabilityoftheattackerto line-fill buffer (LFB). Recently, a front-end side-channel is
interprettheresultsofthevictim. Theworkthatintroduces proposed [42]. They found that during an interrupt, some
Flush+Reload[63]mentionsthatresultsfromdataprefetching instructions’ execution time will change when instructions
shouldbeidentifiedandfilteredoutwhenanalyzingtheresults. aroundthem,andtheirvirtualaddressesvaries.Theymanaged
Tromeretal.[56]proposetheuseofpointer-basedtraversal toexploitthistimingdifferenceonIntelSGXtodistinguish
method in Prime+Probe to suppress prefetching. Wang et betweeninstruction-wiseidenticalbranchesandextractsecret
al. [59] eliminate prefetching effect from Prime+Probe by state.
constructingevictionsetswithanextrawarm-upsectionand
designsspecialprimingandprobingsequencesonthespecific 10. Conclusion
machines. Someworks[17,18]evenexploittheprefetcher
asadefenceagainstacovertchannelandcacheside-channel In this work, we observe that Intel IP-stride prefetcher is a
byobfuscatingthecommunicationsynchronizationorcache sharedhardwareresource. Weleveragethisfeaturetointro-
footprintitself. duce a novel side-channel attack that can leak control flow
namedAfterImage. Toaccomplishtheattack,wepresentan
9.3. BranchPredictor-basedSide-Channels
in-depth study of the Intel IP-stride prefetcher, revealing a
The branch predictor (BPU, branch prediction unit), an- numberofundocumenteddetails. AfterImageisabletoleak
other shared structure, has been exploited to conduct spec- victim’scontrolflowcross(a)coderegions,(b)processspaces,
ulative execution based attacks and has been studied exten- and(c)crossuser-kernelboundary. WeshowthatAfterImage
sively [11, 27, 31, 48, 50, 57]. In these works, the branch achievesasuccessrateofupto99%, dependingonvariant.
predictorcanbemis-trainedbytheadversarytoforcespec- Wefinallydiscusseffectivenessofexisteddefensesandpro-
ulativeexecutionofmispredictedinstructions. Wefindthat posetwomitigationtechniquestoblockthisside-channel,one
themicroarchitectureoftheIntelBPUresemblesitshardware ofwhichcanbeusedonhardwaresystemstoday.

References [26] Gorka Irazoqui, Thomas Eisenbarth, and Berk Sunar. Systematic
|     |     |     |     |     | reverseengineeringofcachesliceselectioninintelprocessors. |     |     |     |     | In  |
| --- | --- | --- | --- | --- | --------------------------------------------------------- | --- | --- | --- | --- | --- |
[1] 3rddataprefetchingchampionship(DPC). https://dpc3.compas. EuromicroConferenceonDigitalSystemDesign,pages629–636,2015.
cs.stonybrook.edu/?finalprograms. [27] PaulKocher,JannHorn,AndersFogh,,DanielGenkin,DanielGruss,
[2] ChampSim.https://github.com/ChampSim/ChampSim. WernerHaas,MikeHamburg,MoritzLipp,StefanMangard,Thomas
[3] Guidelines for mitigating timing side channels against cryp- Prescher,MichaelSchwarz,andYuvalYarom. Spectreattacks: Ex-
https://software.intel.com/ ploitingspeculativeexecution. InIEEESymposiumonSecurityand
| tographic | implementations. |     |     |     |     |     |     |     |     |     |
| --------- | ---------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
content/www/us/en/develop/articles/software-security- Privacy(S&P’19),2019.
guidance/secure-coding/mitigate-timing-side-channel- [28] EsmaeilMohammadianKoruyeh,ShirinHajiAminShirazi,KhaledN
crypto-implementation.html. Khasawneh,ChengyuSong,andNaelAbu-Ghazaleh.Speccfi:Mitigat-
[4] InconsistencyinTLBmisscounters.https://software.intel.com/ ingspectreattacksusingcfiinformedspeculation.InIEEESymposium
en-us/forums/software-tuning-performance-optimization- onSecurityandPrivacy(S&P),pages39–53,2020.
platform-monitoring/topic/593830. [29] CongmiaoLiandJean-LucGaudiot.Challengesindetectingan“eva-
[5] National institute of standards and technology. 2013. FIPS PUB sivespectre”.IEEEComputerArchitectureLetters,pages18–21,2020.
186-4digitalsignaturestandard(DSS). https://csrc.nist.gov/ [30] Chit-KwanLinandStephenJTarsa.Branchpredictionisnotasolved
publications/detail/fips/186/4/final. problem:Measurements,opportunities,andfuturedirections. arXiv
preprintarXiv:1906.08170,2019.
[6] OpenSSL,cryptographyandSSL/TLStoolkit.http://www.openssl.
org. [31] Moritz Lipp, Michael Schwarz, Daniel Gruss, Thomas Prescher,
[7] SPECCPU2006.https://www.spec.org/cpu2006/. WernerHaas,AndersFogh,JannHorn,StefanMangard,PaulKocher,
|     |     |     |     |     | DanielGenkin,etal. |     | Meltdown:Readingkernelmemoryfromuser |     |     |     |
| --- | --- | --- | --- | --- | ------------------ | --- | ------------------------------------ | --- | --- | --- |
[8] SPECCPU2017.https://www.spec.org/cpu2017/.
|     |     |     |     |     | space. | InUSENIXSecuritySymposium(USENIXSecurity),pages |     |     |     |     |
| --- | --- | --- | --- | --- | ------ | ----------------------------------------------- | --- | --- | --- | --- |
[9] AndreasAbelandJanReineke.nanobench:Alow-overheadtoolfor
| runningmicrobenchmarksonx86systems.InInternationalSymposium |     |     |     |     | 973–990,2018. |     |     |     |     |     |
| ----------------------------------------------------------- | --- | --- | --- | --- | ------------- | --- | --- | --- | --- | --- |
[32] FangfeiLiu,QianGe,YuvalYarom,FrankMckeen,CarlosRozas,
onPerformanceAnalysisofSystemsandSoftware(ISPASS),2020.
GernotHeiser,andRubyB.Lee.Catalyst:Defeatinglast-levelcache
| [10] OnurAciiçmez. | Yetanothermicroarchitecturalattack: |     |     | Exploiting |              |         |          |            |              |         |
| ------------------ | ----------------------------------- | --- | --- | ---------- | ------------ | ------- | -------- | ---------- | ------------ | ------- |
|                    |                                     |     |     |            | side channel | attacks | in cloud | computing. | In Symposium | on High |
I-Cache. InWorkshoponComputerSecurityArchitecture(CSAW), PerformanceComputerArchitecture(HPCA),pages406–418,2016.
page11–18,2007.
|     |     |     |     |     | [33] XiaoxuanLou, |     | TianweiZhang, | JunJiang, | andYinqianZhang. | A   |
| --- | --- | --- | --- | --- | ----------------- | --- | ------------- | --------- | ---------------- | --- |
[11] AtriBhattacharyya,AndrésSánchez,EsmaeilMKoruyeh,NaelAbu-
surveyofmicroarchitecturalside-channelvulnerabilities,attacks,and
Ghazaleh,ChengyuSong,andMathiasPayer.SpecROP:Speculative defensesincryptography.2021.
| exploitationofROPchains. |     | InSymposiumonResearchinAttacks, |     |     |     |     |     |     |     |     |
| ------------------------ | --- | ------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
[34] KevinLoughlin,IanNeal,JiachengMa,ElisaTsai,OfirWeisse,Satish
IntrusionsandDefenses(RAID),pages1–16,2020.
Narayanasamy,andBarisKasikci.Dolma:Securingspeculationwith
| [12] Thomas Bourgeat, | Ilia | Lebedev, Andrew | Wright, | Sizhuo Zhang, |     |     |     |     |     |     |
| --------------------- | ---- | --------------- | ------- | ------------- | --- | --- | --- | --- | --- | --- |
theprincipleoftransientnon-observability.InUSENIXSecuritySym-
Arvind,andSrinivasDevadas.MI6:Secureenclavesinaspeculative posium(USENIXSecurity),2021.
out-of-orderprocessor.InSymposiumonMicroarchitecture(MICRO),
[35] RobertMartin,JohnDemme,andSimhaSethumadhavan.Timewarp:
page42–56,2019.
Rethinkingtimekeepingandperformancemonitoringmechanismsto
[13] MarcoChiappetta,ErkaySavas,andCemalYilmaz.Realtimedetec- mitigateside-channelattacks. InInternationalSymposiumonCom-
tionofcache-basedside-channelattacksusinghardwareperformance puterArchitecture(ISCA),pages118–129,2012.
counters.Appl.SoftComput.,page1162–1174,2016.
|     |     |     |     |     | [36] Clémentine | Maurice, | Nicolas | Le Scouarnec, | Christoph | Neumann, |
| --- | --- | --- | --- | --- | --------------- | -------- | ------- | ------------- | --------- | -------- |
[14] PatrickCroninandChengmoYang.Afetchingtale:Covertcommuni- Olivier Heen, and Aurélien Francillon. Reverse engineering intel
cationwiththehardwareprefetcher. InInternationalSymposiumon last-levelcachecomplexaddressingusingperformancecounters. In
HardwareOrientedSecurityandTrust(HOST),pages101–110,2019. SymposiumonRecentAdvancesinIntrusionDetection(RAID),pages
48–65,2015.
| [15] JackDoweck. | WhitepaperinsideIntel®Core™microarchitecture |     |     |     |     |     |     |     |     |     |
| ---------------- | -------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
andsmartmemoryaccess.IntelCorporation,pages72–87,2006. [37] SamiraMirbagher-Ajorpaz,GillesPokam,EsmaeilMohammadian-
[16] RichardDurstenfeld.Algorithm235:Randompermutation.Commun. Koruyeh,ElbaGarza,NaelAbu-Ghazaleh,andDanielA.Jiménez.
|     |     |     |     |     | Perspectron: | Detectinginvariantfootprintsofmicroarchitecturalat- |     |     |     |     |
| --- | --- | --- | --- | --- | ------------ | --------------------------------------------------- | --- | --- | --- | --- |
ACM,pages420–422,1964.
tackswithperceptron.InSymposiumonMicroarchitecture(MICRO),
[17] HongyuFang,SaiSantoshDayapule,FanYao,MilošDoroslovacˇki,
pages1124–1137,2020.
| and Guru Venkataramani. |     | Prefetch-guard: | Leveraging | hardware |     |     |     |     |     |     |
| ----------------------- | --- | --------------- | ---------- | -------- | --- | --- | --- | --- | --- | --- |
prefetchestodefendagainstcachetimingchannels.InSymposiumon [38] DagArneOsvik,AdiShamir,andEranTromer. Cacheattacksand
countermeasures:thecaseofaes.InCryptographers’trackattheRSA
HardwareOrientedSecurityandTrust(HOST),pages187–190,2018.
conference,pages1–20,2006.
| [18] AdiFuchsandRubyBLee. |     | Disruptiveprefetching:impactonside- |     |     |     |     |     |     |     |     |
| ------------------------- | --- | ----------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
channelattacksandcachedesigns.InSystemsandStorageConference [39] SamuelPakalapatiandBiswabandanPanda. Bouquetofinstruction
pointers:Instructionpointerclassifier-basedspatialhardwareprefetch-
(SYSTOR),pages1–12,2015.
ing. InInternationalSymposiumonComputerArchitecture(ISCA),
[19] QianGe,YuvalYarom,DavidCock,andGernotHeiser.Asurveyof
pages118–131,2020.
microarchitecturaltimingattacksandcountermeasuresoncontempo- [40] Pierre-YvesPéneau,DavidNovo,FlorentBruguier,GillesSassatelli,
| raryhardware. | JournalofCryptographicEngineering,pages1–27, |     |     |     |     |     |     |     |     |     |
| ------------- | -------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
andAbdoulayeGamatié.Performanceandenergyassessmentoflast-
2018.
|     |     |     |     |     | level cache | replacement |     | policies. In | Conference on Embedded | &   |
| --- | --- | --- | --- | --- | ----------- | ----------- | --- | ------------ | ---------------------- | --- |
[20] BenGras,KavehRazavi,HerbertBos,andCristianoGiuffrida.Trans- DistributedSystems(EDiS),pages1–6,2017.
lationleak-asidebuffer:Defeatingcacheside-channelprotectionswith
[41] ColinPercival.Cachemissingforfunandprofit,2005.
| TLBattacks. | InUSENIXSecuritySymposium(USENIXSecurity), |     |     |     |     |     |     |     |     |     |
| ----------- | ------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
pages955–972,2018. [42] IvanPuddu,MoritzSchneider,MiroHaller,andSrdjanCˇapkun.Frontal
|     |     |     |     |     | attack: | leakingcontrol-flowinsgxviathecpufrontend. |     |     |     | USENIX |
| --- | --- | --- | --- | --- | ------- | ------------------------------------------ | --- | --- | --- | ------ |
[21] BenGras,KavehRazavi,ErikBosman,HerbertBos,andCristiano
Giuffrida. Aslrontheline: Practicalcacheattacksonthemmu. In SecuritySymposium(USENIXSecurity),2021.
NetworkandDistributedSystemSecuritySymposium(NDSS),page26,
|     |     |     |     |     | [43] ZhenxiaoQi, | QianFeng, |     | YueqiangCheng, | MengjiaYan, | PengLi, |
| --- | --- | --- | --- | --- | ---------------- | --------- | --- | -------------- | ----------- | ------- |
2017. HengYin, andTaoWei. Spectaint: Speculativetaintanalysisfor
[22] DanielGruss,ClémentineMaurice,AndersFogh,MoritzLipp,and discoveringspectregadgets.2021.
| StefanMangard. | Prefetchside-channelattacks: |     | BypassingSMAP |     |     |     |     |     |     |     |
| -------------- | ---------------------------- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- |
[44] MoinuddinK.Qureshi.Ceaser:Mitigatingconflict-basedcacheattacks
andkernelASLR.InConferenceonComputerandCommunications viaencrypted-addressandremapping.InSymposiumonMicroarchi-
Security(CCS),pages368–379,2016.
tecture(MICRO),page775–787,2018.
[23] DanielGruss,ClémentineMaurice,KlausWagner,andStefanMangard. [45] MoinuddinK.Qureshi.Newattacksanddefenseforencrypted-address
Flush+Flush: A fast and stealthy cache attack. In Conference on cache.InInternationalSymposiumonComputerArchitecture(ISCA),
| DetectionofIntrusionsandMalware,andVulnerabilityAssessment |     |     |     |     | page360–371,2019. |     |     |     |     |     |
| ---------------------------------------------------------- | --- | --- | --- | --- | ----------------- | --- | --- | --- | --- | --- |
(DIMVA),pages279–299,2016.
[46] AshayRane,CalvinLin,andMohitTiwari.Raccoon:Closingdigital
[24] W.-M.Hu.Reducingtimingchannelswithfuzzytime.InSymposium
|     |     |     |     |     | side-channels | through | obfuscated | execution. | In USENIX | Security |
| --- | --- | --- | --- | --- | ------------- | ------- | ---------- | ---------- | --------- | -------- |
onResearchinSecurityandPrivacy,pages8–20,1991. Symposium(USENIXSecurity),pages431–446,2015.
[25] Intel.Intel®64andia-32architecturesoptimizationreferencemanual. [47] JamesReinders.Vtuneperformanceanalyzeressentials.IntelPress,
| IntelCorporation,2019. |     |     |     |     | 2005. |     |     |     |     |     |
| ---------------------- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- |

[48] XidaRen,LoganMoody,MohammadkazemTaram,MatthewJordan,
DeanMTullsen,andAshishVenkat.Iseedeadµops:Leakingsecrets
viaintel/amdmicro-opcaches.2021.
| [49] AdityaRohan,BiswabandanPanda,andPrakharAgarwal. |     |     |     | Reverse |
| ---------------------------------------------------- | --- | --- | --- | ------- |
engineeringthestreamprefetcherforprofit.InEuropeanSymposium
onSecurityandPrivacyWorkshops(EuroS&PW),pages682–687,
2020.
[50] MichaelSchwarz,MoritzLipp,DanielMoghimi,JoVanBulck,Julian
| Stecklina,ThomasPrescher,andDanielGruss. |     |                           | Zombieload:Cross- |     |
| ---------------------------------------- | --- | ------------------------- | ----------------- | --- |
| privilege-boundarydatasampling.          |     | InConferenceonComputerand |                   |     |
CommunicationsSecurity(CCS),pages753–768,2019.
| [51] YoungjooShin,             | HyungChanKim,                               | DokeunKwon,               | JiHoonJeong, |     |
| ------------------------------ | ------------------------------------------- | ------------------------- | ------------ | --- |
| andJunbeomHur.                 | Unveilinghardware-baseddataprefetcher,ahid- |                           |              |     |
| densourceofinformationleakage. |                                             | InConferenceonComputerand |              |     |
CommunicationsSecurity(CCS),page131–145,2018.
[52] AlanJaySmith.Sequentialprogramprefetchinginmemoryhierarchies.
Computer,pages7–21,1978.
| [53] WeiSongandPengLiu. |     | Dynamicallyfindingminimalevictionsets |     |     |
| ----------------------- | --- | ------------------------------------- | --- | --- |
canbequickerthanyouthinkforside-channelattacksagainstthellc.
InSymposiumonResearchinAttacks,IntrusionsandDefenses(RAID),
pages427–442,2019.
[54] MohammadkazemTaram,AshishVenkat,andDeanTullsen.Context-
sensitivefencing:Securingspeculativeexecutionviamicrocodecus-
tomization.InConferenceonArchitecturalSupportforProgramming
LanguagesandOperatingSystems(ASPLOS),pages395–410,2019.
[55] MCanerTol,BerkGulmezoglu,KorayYurtseven,andBerkSunar.
| Fastspec: | Scalablegenerationanddetectionofspectregadgetsus- |     |     |     |
| --------- | ------------------------------------------------- | --- | --- | --- |
ingneuralembeddings.IEEEEuropeanSymposiumonSecurityand
Privacy(EuroS&P),2021.
[56] EranTromer,DagArneOsvik,andAdiShamir.Efficientcacheattacks
| onaes,andcountermeasures. |     | JournalofCryptology,pages37–71, |     |     |
| ------------------------- | --- | ------------------------------- | --- | --- |
2010.
[57] StephanvanSchaik,MarinaMinkin,AndrewKwong,DanielGenkin,
| andYuvalYarom. | Cacheout: | Leakingdataonintelcpusviacache |     |     |
| -------------- | --------- | ------------------------------ | --- | --- |
evictions.2021.
| [58] PepeVila,BorisKöpf,andJoséF.Morales. |     |                                     | Theoryandpracticeof |     |
| ----------------------------------------- | --- | ----------------------------------- | ------------------- | --- |
| findingevictionsets.                      |     | InIEEESymposiumonSecurityandPrivacy |                     |     |
(S&P),pages39–54,2019.
[59] DaimengWang,ZhiyunQian,NaelAbu-Ghazaleh,andSrikanthV
Krishnamurthy.Papp:Prefetcher-awareprimeandprobeside-channel
attack.InDesignAutomationConference(DAC),pages1–6,2019.
[60] GuanhuaWang,SudiptaChattopadhyay,IvanGotovchits,TulikaMi-
| tra,andAbhikRoychoudhury. |     | oo7: Low-overheaddefenseagainst |     |     |
| ------------------------- | --- | ------------------------------- | --- | --- |
spectreattacksviaprogramanalysis.IEEETransactionsonSoftware
Engineering,2019.
[61] WenhaoWang,GuoxingChen,XiaoruiPan,YinqianZhang,XiaoFeng
Wang,VincentBindschaedler,HaixuTang,andCarlAGunter.Leaky
cauldrononthedarkland:Understandingmemoryside-channelhaz-
ardsinsgx.InConferenceonComputerandCommunicationsSecurity
(CCS),pages2421–2434,2017.
[62] MengWu,ShengjianGuo,PatrickSchaumont,andChaoWang.Elimi-
natingtimingside-channelleaksusingprogramrepair.InInternational
SymposiumonSoftwareTestingandAnalysis(ISSTA),page15–26,
2018.
[63] YuvalYaromandKatrinaFalkner.FLUSH+RELOAD:Ahighreso-
| lution,lownoise,l3cacheside-channelattack. |     |     | InUSENIXSecurity |     |
| ------------------------------------------ | --- | --- | ---------------- | --- |
Symposium(USENIXSecurity),pages719–732,2014.
| [64] Jiyong | Yu, Lucas Hsiung, | Mohamad El’Hajj, | and Christopher | W   |
| ----------- | ----------------- | ---------------- | --------------- | --- |
Fletcher.Dataobliviousisaextensionsforsidechannel-resistantand
| highperformancecomputing. |     | InNetworkandDistributedSystem |     |     |
| ------------------------- | --- | ----------------------------- | --- | --- |
SecuritySymposium(NDSS),2019.
[65] JiyongYu,MengjiaYan,ArtemKhyzha,AdamMorrison,JosepTor-
| rellas,andChristopherWFletcher. |     | Speculativetainttracking(stt)a |     |     |
| ------------------------------- | --- | ------------------------------ | --- | --- |
comprehensiveprotectionforspeculativelyaccesseddata.InSympo-
siumonMicroarchitecture(MICRO),pages954–968,2019.
[66] TianweiZhang,YinqianZhang,andRubyBLee.Cloudradar:Areal-
timeside-channelattackdetectionsysteminclouds.InSymposiumon
ResearchinAttacks,Intrusions,andDefenses(RAID),pages118–140,
2016.
