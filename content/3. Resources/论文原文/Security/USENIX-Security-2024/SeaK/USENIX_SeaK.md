---
publish: true
---

SeaK: Rethinking the Design of a
Secure Allocator for OS Kernel
Zicheng Wang, University of Colorado Boulder & Nanjing University; Yicheng Guang,
Nanjing University; Yueqi Chen, University of Colorado Boulder; Zhenpeng Lin,
Northwestern University; Michael Le, IBM Research; Dang K Le, Northwestern
University; Dan Williams, Virginia Tech; Xinyu Xing, Northwestern University;
Zhongshu Gu and Hani Jamjoom, IBM Research
https://www.usenix.org/conference/usenixsecurity24/presentation/wang-zicheng
This paper is included in the Proceedings of the
33rd USENIX Security Symposium.
August 14–16, 2024 • Philadelphia, PA, USA
978-1-939133-44-1
Open access to the Proceedings of the
33rd USENIX Security Symposium
is sponsored by USENIX.

SeaK: Rethinking the Design of a Secure Allocator for OS Kernel
ZichengWang†‡,YichengGuang‡,YueqiChen†,
ZhenpengLin§,MichaelLe¶,DangKLe§,
DanWilliams⋆,XinyuXing§,ZhongshuGu¶,HaniJamjoom¶,
†UniversityofColoradoBoulder,‡NanjingUniversity,
§NorthwesternUniversity,¶IBMResearch,⋆VirginiaTech
†{zicheng.wang,yueqi.chen}@colorado.edu,‡ guangyichengok@gmail.com,
§{zplin,dang.le,xinyu.xing}@u.northwestern.edu,
¶{mvle,zgu,jamjoom}@us.ibm.com,⋆djwillia@vt.edu
Abstract Thoughthere are security features in place in the Linux
Inrecentyears,heap-basedexploitationhasbecomethemost kernel to counter heap-based exploitation,they can barely
dominantattackagainsttheLinuxkernel.Securingthekernel providetheexpectedprotection. Ononehand,thefeatures
heapisofvitalimportanceforkernelprotection.Thoughthe thatareenabledbydefaultinmostdistrosareoverlyspecific
Linuxkernelallocatorhassomesecuritydesignsinplaceto tocertainexploitationtechniques,makingthembypassable.
counterexploitation,ouranalyticalexperimentsrevealthat Consider freelist randomization as an example: by design,
theycanbarelyprovidetheexpectedresults.Thisshortfall itonlyworksforexploitsleveragingspatialcorruption,like
is rootedin the currentstrategyofdesigning secure kernel heapout-of-boundwriteandread,andfallsshortindefend-
allocatorswhichinsistsonprotectingeveryobjectallthetime. ing againsttemporalcorruption,suchas use-after-free. On
Suchstrategyinherentlyconflictswiththekernelnature. theotherhand,thefeaturesthataredisabledbydefaulthave
Tothisend,weadvocateforrethinkingthedesignofsecure intrinsicweaknessesandfailtodeliverthepromisedsecurity
kernelallocator.Inthiswork,weexploreanewstrategywhich improvements.Toenumerate,structurelayoutrandomization
centers around the “atomic alleviation” concept, featuring faceschallengesinsecurelystoringtherandomseed,while,
flexibilityandefficiencyindesignanddeployment.Recent basedonourexperiments,KFENCErarelyachievesitsgoal
advancements in kernel design and research outcomes on ofseparatingkernelobjects.
exploitationtechniquesenableustoprototypethisstrategyin Inparallel,intheuserspace,thedesignofsecureallocators
atoolnamedSeaK.Weusedreal-worldcasestothoroughly hasbeenwellresearched[19,45,57,59],introducingfeatures
evaluate SeaK. The results validate that SeaK substantially like red zones,poisoning,user tracking,and ad-hoc sanity
strengthensheapsecurity,outperformingallexistingfeatures, checks.Thesefeaturescanbefoundinakernelmechanism
withoutincurringnoticeableperformanceandmemorycost. named slub_debug. Unfortunately,slub_debug is regarded
Besides,SeaKshowsexcellentscalabilityandstabilityinthe primarilyasadebuggingfeatureandservesasthebuilding
productionscenario. blockforsanitizerslikeKASAN,duetoitshighcost.
Inthiswork,weconductedamulti-facetedmeasurementof
1 Introduction existingsecurityfeatures,revealingthefundamentalobstacle
indesigningakernelsecureallocator:theallocatoristhecore
IntheLinuxkernel,exploitationtargetingheapvulnerabili- kernelsubsystemandisinvokedwithhighfrequency.Given
tiessuchasuse-after-free,heapout-of-boundwriteandread, thecurrentstrategyinsecurityfeaturedesign,whichinsists
anduninitializedheapisveryprevalent.IntheGoogleand onprotectingeveryobjectallthetime[51],performanceand
AlphabetVulnerabilityRewardProgramheldin2022[36],42 memoryoverheadaccompanyeachallocationandfree.This
revealedLinuxkernelexploitsareallagainstthekernelheap. level of overhead is unacceptable for the kernel as it must
InthePwn2Owncontestsince2020,allnineshowcasedLinux offerservicesforuserspacewithhighefficiencybutdoesnot
kernelexploitstargetthekernelheap,withawardsreaching haveunlimitedmemory.
upto$300k.Inaddition,fromavarietyofsourcesincluding Sincetheobstacleisintrinsictothekernelnature,weadvo-
industrysummitslikeBlackHatandpersonalblogsoffamous cateforrethinkingthedesignofkernelsecureallocators.In
whitehathackers(e.g.,[1,9]),wecollectedpublicexploits, thiswork,weexploreanewstrategywhichcentersaroundthe
PoCs,andwrite-upsoverthepastfiveyears,andfoundthat “atomicalleviation”(AA)concept.OneAAoffersthemost
143outof173Linuxkernelexploitsareagainstthekernel granularlevelofexploitalleviationbyseparatingaspecific
heap.Hence,securingtheheapisofparamountimportance type ofkernelobject. We can orchestrate particularsetsof
forkernelprotection. AAstomeetdistinctsecurityneeds,focusingonlyoncritical
USENIX Association 33rd USENIX Security Symposium 1171

objectsinsteadofeveryobjectindiscriminately.Besides,the
spatial overlapping
| enforcementandretirementofAAsdonotbothertorecom- |     |     |     |     |     |     |     |           |          |     | vuln. obj |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --------- | -------- | --- | --------- | --- | --- | --- |
|                                                  |     |     |     |     |     |     |     | vuln. obj | vic. obj |     | free      |     |     |     |
pileandrebootthekernel,supportingcontinuousprotection temporal
free slot
| upgrading. |     |     |     |     |     |     |     |     |     |     |     | overlapping |     |     |
| ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | --- |
allocate
| This strategy |     | had been | previously |     | infeasible | until re- |     | sensitive data |     |     |     |     |     |     |
| ------------- | --- | -------- | ---------- | --- | ---------- | --------- | --- | -------------- | --- | --- | --- | --- | --- | --- |
vic. obj
| cent advancements |     | in the | extended | Berkeley | Packet | Filter |     |     |     |     |     |     |     |     |
| ----------------- | --- | ------ | -------- | -------- | ------ | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
Figure1:Kernelheapexploitationwhichcreatesoverlapping
(eBPF)[29,35,49,54,66,71,72].UsingeBPF,weprototyped
betweenspatial/temporalmemorycorruptionfromthevulner-
thisnewstrategyinatoolnamedSeaK,representingSecure
ableobjectandsensitivedatainthevictimobject.
| allocatorforKernel. |     | Wefurtherdevelopedtechniquesthat |     |     |     |     |     |     |     |     |     |     |     |     |
| ------------------- | --- | -------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
canautomaticallyidentifyallocationandfreesitesofobjects
selectedforseparationinacertainscenario,andsynthesized 2 Background
eBPFprogramsthatconstructseparationatruntime.Thesep-
aration is shipped with guard pages and up to 43 entropy Inthissection,wewilldescribetheheapmanagementand
randomization.
heapexploitationtechniquesintheLinuxkernel.
| This strategy |     | wouldn’tbe | reliable | security-wise |     | without |     |     |     |     |     |     |     |     |
| ------------- | --- | ---------- | -------- | ------------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
theoutcomesofresearchonexploitationtechniquesinrecent
years. We applied SeaK to two scenarios to illustrate how 2.1 KernelHeapManagement
| itenhances | kernelheapsecurity. |     |     | Given | thatnotallobjects |     |     |     |     |     |     |     |     |     |
| ---------- | ------------------- | --- | --- | ----- | ----------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
areworthprotecting,recentworks[21,22,65]havebeenfo- IntheLinuxkernel,thetermofheapoftenreferstothemem-
cusingonidentifyingsecurity-sensitiveobjectsintheLinux ory located in the direct mapping region which has 64TB
|     |     |     |     |     |     |     | intotalandmapsallphysicalmemory. |     |     |     |     | Theheapiscoordi- |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------------------- | --- | --- | --- | --- | ---------------- | --- | --- |
kernel.Advancementsinthisdirectionpavethewayforde-
ployingSeaKtoseparateandprotectthesecriticalobjects.In natelymanagedbythebuddyallocatorandtheSLAB/SLUB
allocator,whichroughlyworkasfollows.
anotherscenariowherethekernelpatchingprocessislengthy,
weshowcasehowSeaKcanseparatecorruptionsintroduced
|     |     |     |     |     |     |     | Buddy | Allocator. |     | The buddy | allocator | partitions | physical |     |
| --- | --- | --- | --- | --- | --- | --- | ----- | ---------- | --- | --------- | --------- | ---------- | -------- | --- |
byN-dayvulnerabilities,therebyprotectingthekernelfrom
memoryintopagesandmanagesobjectslargerthanonepage.
potentialexploitationbeforepatchesareavailable. Duringallocation,iftherequestedsize(e.g.,2pages)isun-
WeevaluatedSeaKintermsofsecurityimprovement,per-
available,alargerchunk(e.g.,8pages)isrepeatedlyhalved
formance and memory overhead, and scalability, by using (e.g.,8pagesto4pagesto2pages)untilachunkoftheexact
real-worldcases.OurexperimentalresultsvalidatethatSeaK
sizeisproduced.Thetwohalvesarereferredtoasbuddies:
caneffectivelythwartstate-of-the-artkernelheapexploitation,
onesatisfiestheallocationrequestandtheotherremainsfree.
eventhenewestDirtyCredattack[44],outperformingexisting Whentheallocatedbuddyislaterfreedanditscounterpart
securityfeatures.Inthemeanwhile,theperformanceoverhead
buddyisnotused,theywillbemergedbackandreclaimed
ofasingleAAisnegligible-lessthan1%onaveragewhen (e.g.,two2-pagebuddiesbacktoone4-pagechunk).
separatingthemostfrequentlyusedobject;andthememory
overheadisnotnoticeablewhenseparatingthemostdurable SLAB/SLUBAllocator.TheSLAB/SLUBallocatormanages
|              |     |          |          |               |     |             | smallobjects |     | by  | acquiring | pages from | the buddy | allocator. |     |
| ------------ | --- | -------- | -------- | ------------- | --- | ----------- | ------------ | --- | --- | --------- | ---------- | --------- | ---------- | --- |
| object. SeaK | is  | scalable | as there | is no obvious |     | increase in |              |     |     |           |            |           |            |     |
Thesepagesareknownasslabcachesandaredividedinto
performanceoverheadandmemoryoverheadwhenmorethan
|     |     |     |     |     |     |     | slots | - eachslotaims |     | to  | holdone object. | Objects | storedin |     |
| --- | --- | --- | --- | --- | --- | --- | ----- | -------------- | --- | --- | --------------- | ------- | -------- | --- |
64AAsareorchestrated.
|     |     |     |     |     |     |     | the | same | slab cache | either | share the | same | type or have | a   |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ---------- | ------ | --------- | ---- | ------------ | --- |
Toourknowledge,SeaKisthefirstpracticalsecurealloca-
torinopen-sourcekernels. SeaKhasbeendeployedonthe similarsize.Whenslotsarefree,theyarelinkedinasingly-
|     |     |     |     |     |     |     | linked | list | called | the freelist. | The freelist | works | in a | LIFO |
| --- | --- | --- | --- | --- | --- | --- | ------ | ---- | ------ | ------------- | ------------ | ----- | ---- | ---- |
authors’machine,havingsupporteddailyresearchandeduca-
|     |     |     |     |     |     |     | (Last | In, | First Out) | fashion: | retrieving | the first | slot | in the |
| --- | --- | --- | --- | --- | --- | --- | ----- | --- | ---------- | -------- | ---------- | --------- | ---- | ------ |
tionactivitiesfor2.5months,andrunningstablytodate.In
freelistforallocationandaddingtheslotbacktothebeginning
summary,thisworkmakesthefollowingcontributions:
|     |     |     |     |     |     |     | of  | the list | during | free. If | all slots in | these pages | are | freed, |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------ | -------- | ------------ | ----------- | --- | ------ |
• Amulti-facetedmeasurementtodisclosetheinherentob-
theSLAB/SLUBallocatorreturnsthesepagestothebuddy
| staclesofdesigningasecureallocatorforOSkernel. |     |     |     |     |     |     | allocator. |     |     |     |     |     |     |     |
| ---------------------------------------------- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
• Theintroductionofanewandpracticalstrategytosecure
kernelheapanditscoreconcept-atomicalleviation.
2.2 KernelHeapExploitation
| • Open-source |       | design | and implementation |        | of     | the strategy |     |        |                     |     |            |         |            |     |
| ------------- | ----- | ------ | ------------------ | ------ | ------ | ------------ | --- | ------ | ------------------- | --- | ---------- | ------- | ---------- | --- |
| in SeaK       | which | is the | first practical    | secure | kernel | alloca-      |     |        |                     |     |            |         |            |     |
|               |       |        |                    |        |        |              | In  | recent | years, exploitation |     | techniques | against | the kernel |     |
tor. SeaK can remediate vulnerability before patches are heaphavebeenthoroughlydiscussedinbothacademia[21,
available.
22,67,68,70]andindustry[38].Duetothespacelimit,we
• AcomprehensiveevaluationofSeaKwhichvalidatesthe won’telaborateonconcreteexploitsbutinsteadsummarize
effectivenessandefficiencyofthenewstrategy. thecommonideasbehindvariousexploitationtechniques.
| 1172    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

Create Overlapping. Figure 1 illustrates an essential step (C2)featuresthataredesignedforprotectionbutaredisabled
inexploitationwhichistocreateoverlappingbetweenheap bydefaultinmostdistros;(C3)featuresthatarecommonly
corruptionandsensitivedatasuchasfunctionpointersand employedinuserspacesecureallocatorsandintegratedinto
credentials.Theoverlappingcanbecategorizedintospatial slub_debug[37].
overlappingandtemporaloverlappingaccordingtothenature Inthissection,wewillevaluatethesecurityandoverhead
ofvulnerabilities.Toexploitheapout-of-boundwrite/read,at- ofthesefeatures,followedbyinvestigationintotheparticular
tackersmanipulateheaplayout[22,68]toplacevictimobjects obstacleinsecurekernelallocatordesign.
thatcontainsensitivedataadjacenttothevulnerableobject.
Bytriggeringthevulnerability,thecorruptionhasaspatial
3.1 SecurityAnalysis
overlappingwiththesensitivedata,allowingattackerstotam-
perwithitandachieveIPcontrolorprivilegeescalation.To
By-defaultEnabledFeatures(C1).Inthiscategory,❶freel-
exploituse-after-free,attackersfirstfreethevulnerableobject
istrandomization [26] randomizes the orderofslots in the
whichstillhasadanglingpointerreferringtoit.Thefreedslot
freelist of the SLAB/SLUB allocator so that attackers can
isrecycledbacktoSLAB/SLUBallocator. Then,attackers
hardlyaccuratelypredicttheslabcachelayout.❷Thefreelist
sprayvictimobjectscontainingsensitivedatatoreclaimthe
obfuscation[27]aimstohinderattackersfrommanipulating
samememory,leveragingtheLIFOfeature.Bydereferencing
the allocator into returning a memory under the attackers’
thedanglingpointer,attackerstamperwithsensitivedatain
control.❸Theheapzeroing[52]onallocationinitializesthe
thevictimobjectandobtainexploitableprimitives.
slotduringallocationsothatattackerscannotreadsensitive
Cross-cacheExploitation.Ifthevictimobjectandthevulner- informationbelongingtotheobjectthatpreviouslyoccupied
ableobjectareinthesameslabcache,itisoftenreferredtoas theslot.Thesefeaturesraisethebarofexploitationbutare
within-cacheexploitation.Cross-cacheexploitationstandsfor toospecifictocertainvulnerabilitiesandattacktechniques
thesituationwherethevictimobjectandthevulnerableobject andthuscanbeeasilybypassed[2–4,7,10].Takingfreelist
areindifferentslabcaches.Toexploitaheapout-of-bound randomizationasanexample,itisdesignedonlyagainstvul-
write/readinacross-cacheexploitationmanner,attackersma- nerabilitiesthatcausespatialoverlappinglikeout-of-bound
nipulatetheheaplayoutatthebuddyallocatorlevel,ensuring write,anddoesn’tworkforvulnerabilitiesthatcausetemporal
thattwopages—theonecontainingthevictimobjectand overlappinglikeuse-after-free.Evenforspatialoverlapping,
theothercontainingthevulnerableobjectareadjacentthough itcanbebypassedthroughheapgrooming[39].
they belong to distinct caches. To exploit a use-after-free
By-defaultDisabledFeatures(C2).Inthiscategory,❶struc-
inacross-cacheexploitationmanner,attackersfirstfreeall
ture layout randomization [25] shuffles the field orders in
vulnerableobjectswithinthesamecachesothatthecacheis
structures each time the kernel boots up so that attackers
reclaimedtothebuddyallocator.Then,theyallocateanumber
cannotpredicttheoffsetbetweensensitivedataandthestart
ofvictimobjects,forcingthebuddyallocatortore-halvethe
ofcorruption.However,thekernelmustrevealtherandom
justreclaimedslabcachepages-previouslystoringvulner-
seedtosupportcompilingthird-partykernelmodules.How
ableobjects-forstoringvictimobjectsinstead.Thus,there
tosecurelystoretheseedcontinuestobeachallengenowa-
isatemporaloverlappingbetweenthevulnerableandvictim
days[33].Besides,itonlyrandomizesstructuresthatcontain
objects.
onlyfunctionpointersandcannotcoverallstructuraltypes.
Same-type Exploitation. In most exploits, regardless of ❷KFENCEallocatesobjectsfromapre-reservedpool.Each
within-cache or cross-cache,the vulnerable object and the objecttakespagesandissurroundedbyredzonesandguard
victimobjectareofdistincttypessothedifferencebetween pages,whichcandetectout-of-boundaccess.Whentheob-
theirsemantics allows successful tampering. However,the ject is freed, the corresponding page is unmapped so that
newestDirtyCredattack[44]demonstratesthatvulnerableob- use-after-freecanalsobedetected.However,toreduceover-
jectsandvictimobjectscanbeofthesametypeinexploitation. head,KFENCErandomlysamplesobjectsforseparation,no
Technically,twoobjectsofthesametype(e.g.,struct cred matterwhethertheobjectissecurity-relatedornot.Further,
butcarryingdifferentprivilegelevels(e.g.,cred.uid == 1000 thesamplingonlyhappensinthefastpathofallocationandit
andcred.uid == 1000)canbeoverlappedtoachieveprivilege onlysamplesoneobjectinacertaintimewindow.Thus,in
escalation.Thisattackbypassesallexistingsecurityfeatures ourexperiment,KFENCEcanonlyprotect0.005%-0.35%
inthekernelallocator,evenslub_debug. sensitiveobjectsevenafteritscapabilityismaximized.Fea-
turesinthiscategoryaredesignedforprotectionbutfallshort
ofprovidingassuredsecurityimprovement,presumablythe
3 ObstaclesinExistingDesigns
reasonwhytheyaredisabledbydefault.
ThenewestsecurityfeaturesintheLinuxkernelallocatorare Lightweight “Debugging” Features (C3). This category
inthreecategories:(C1)featuresthatareenabledbydefault includes❶slub_debugwhichisprimarilyregardedasade-
in mainstream Linux distros such as Ubuntu and CentOS; buggingfeature.Itprovidesafullspectrumofsecurityfea-
USENIX Association 33rd USENIX Security Symposium 1173

turestypicallyfoundinuserspacesecureallocators:redzones
|     |     |     |     | LMbench | C1 C2 | C3  |
| --- | --- | --- | --- | ------- | ----- | --- |
aroundobjectsenabledbyZflag,poisoningandusertracking
|     |     |     |     | Simplesyscall | 0.35% 1.06% | 0.90% |
| --- | --- | --- | --- | ------------- | ----------- | ----- |
offreedmemorybyPandUflagsrespectively,andadditional
|     |     |     |     | Simpleread | 0.98% 3.73% | 0.70% |
| --- | --- | --- | --- | ---------- | ----------- | ----- |
sanitychecksbyFflag.ComparedwithKASAN,slub_debug
|     |     |     |     | Simplewrite | 0.41% 1.71% | 2.46% |
| --- | --- | --- | --- | ----------- | ----------- | ----- |
islightweightbutforprotection,itistooheavy.Wewillshow
|     |     |     |     | Selecton100fd’s | -0.64% 1.21% | 0.04% |
| --- | --- | --- | --- | --------------- | ------------ | ----- |
thisinthefollowing.
|                         |     |     |     | Signalhandlerinstall  | -1.35% -1.88% | -1.17%  |
| ----------------------- | --- | --- | --- | --------------------- | ------------- | ------- |
|                         |     |     |     | Signalhandleroverhead | 0.75% 3.29%   | 169.16% |
| 3.2 OverheadMeasurement |     |     |     | fork+exit             | 0.60% 1.76%   | 168.17% |
|                         |     |     |     | fork+execve           | 2.42% 1.56%   | 177.22% |
BenchmarksandSettings.Weusetwobenchmarkstoevalu- fork+/bin/sh-c 1.21% 2.32% 151.55%
ate the overhead of existing security features. One is LM- UDPlatency 3.91% 4.97% 144.34%
| bench [47]                        | which is a micro-benchmark |                  | widely used for  |                  |              |         |
| --------------------------------- | -------------------------- | ---------------- | ---------------- | ---------------- | ------------ | ------- |
|                                   |                            |                  |                  | TCP/IPconnection | -2.74% 5.25% | 129.81% |
| system                            | call level measurement.    | Another          | is Phoronix Test |                  |              |         |
|                                   |                            |                  |                  | AF_UNIXbandwidth | -0.20% 0.27% | 52.16%  |
| Suite[11]whichisamacro-benchmark. |                            | Itrunsreal-world |                  |                  |              |         |
|                                   |                            |                  |                  | Pipebandwidth    | 0.80% 1.16%  | -1.98%  |
applicationsandweuseittomeasuretheimpactofSeaKon
|     |     |     |     | Phoronix | C1 C2 | C3  |
| --- | --- | --- | --- | -------- | ----- | --- |
theoverallsystem.Inparticular,fromPhoronixTestSuite,we
|     |     |     |     | Sockperf(Msgs/sec) | -0.27% -0.61% | 57.58% |
| --- | --- | --- | --- | ------------------ | ------------- | ------ |
choosesevenapplications:OpenSSL,7zip-compress,FFm-
peg,Redis,SQLite,andApacheasrepresentativesofdifferent OSBench(Ns/Event) -0.08% -1.00% 6.25%
|     |     |     |     | 7-ZipCompress(MIPS) | -0.34% 0.54% | -0.39% |
| --- | --- | --- | --- | ------------------- | ------------ | ------ |
workloadstocomprehensivelytestprocessor,OS,andsystem.
Ourexperimentswereconductedusingabare-metalma- FFmpegLive(FPS) -0.14% 0.28% 1.25%
|     |     |     |     | OpenSSLSHA256(B/s) | 0.01% 0.04% | 0.01% |
| --- | --- | --- | --- | ------------------ | ----------- | ----- |
chinerunningUbuntu22.04LTSwithanIntel(R)Core(TM)
i7-6700CPU@3.40GHzand16GBRAM.Webuiltavanilla RedisSET(Reqs/sec) -0.37% 0.47% 0.55%
kernelimageasthebaselineusingtheLinuxkernelv5.15- SQLiteSpeedtest(sec) 0.52% 1.34% 4.05%
thelatestLong-termsupport(LTS)versionatthetimeofex- Apache100(Reqs/sec) -0.50% -0.42% 46.29%
perimentation.Thevanillakerneldoesn’tcontainanysecurity
featuresdescribedinthissection.Then,webuiltanotherthree Table 1: Performance overhead of security features in different
images:onewithallthreefeaturesinC1,onewiththetwo categories.SeeSection3.1formoredetailsofeachcategory.
featuresinC2,andonewithslub_debuginC3.Tominimize
| fluctuation | and rule out outliers,we | repeatedly | ran bench- |     |     |     |
| ----------- | ------------------------ | ---------- | ---------- | --- | --- | --- |
marksuntiltheoverheadoftherecentfiveexecutionshada as indicating that slub_debug has no overhead. In fact,for
benchmarksthataredependentonkernelservices,theover-
coefficientofvariationsmallerthan3.5%-adefaultsetting
in Phoronix. Between each round,we rebooted the whole headarisessignificantly.
systemtoensureacleanenvironmentforthebenchmark.
MemoryOverhead.Figure2presentsthememoryoverhead
whenrunningLMbench(Phoronix’sresultaremovedto[18]
PerformanceOverhead.Table1presentstheperformance
overhead.Thougharecentstudy[55]revealsthatfreelistran- due to space limit). From the figure, we can observe that
vanilla,C1,andC3exhibitsimilarmemoryusagewhileC2
domizationinC1showcases45%peakoverheadincertain
situations(i.e.,big-selectandbig-fork)becauseofpoorlocal- consumesanadditional400MBofmemory.Thisisbecause
ity,ourmeasurementoverbothLMbenchandPhoronixindi- KFENCEinC2allocatesobjectsfromapre-reservedpool
andeachobjectseparatedbyKFENCEoccupiespagesthat
catesthattheoveralloverheadofC1isnotobvious:-2.74%
to3.91%whichiswithinreasonablefluctuationrange.For aresurroundedbyredzonesandguardpages.
thetwofeaturesinC2thataredisabledbydefault,theirover-
headisslightlynoticeable:-1.88%to5.25%forLMbench,
3.3 BehindSecurity/OverheadTrade-off
| and -1% | to 1.34% forreal-world | applications | in Phoronix. |     |     |     |
| ------- | ---------------------- | ------------ | ------------ | --- | --- | --- |
Whilethisextentofoverheadistolerable,aswediscussedin Summarizingfromsecurityanalysisandoverheadmeasure-
thesecurityanalysis(Section3.1),bothfeaturesinC2have ment,existingsecuritydesignsintheLinuxkernelallocator
inherentweaknessesthatpreventthemfromproducingthe eitherfailtoprovidesubstantialsecurityenhancement(C1and
expectedsecureenhancement. C2)orareimpracticalduetoexcessivememoryandperfor-
Forslub_debuginC3,itsoverheadisprohibitivelyhigh, manceoverhead(C2andC3).Tohaveadeeperunderstanding
reaching 177.22% forLMbench and 57.58% forPhoronix. oftherootobstacle,weselectedApachefromPhoronox-a
Notethat,inthetable,somebenchmarks(e.g.,7zip-compress) casewithnoticeableoverhead-asourtargetandconducteda
| shownegativeoverheadbecausetheirexecutiontimepredom- |     |     |     | casestudy. |     |     |
| ---------------------------------------------------- | --- | --- | --- | ---------- | --- | --- |
inantlytakesplaceinuserspacewithminimalkernelinvolve- Table2presentsthestatisticscollectedbybpftraceduring
ment.Thesenegativenumbersshouldn’tbemis-interpreted theexecutionofApache.Fromthetable,wecanobservethat,
| 1174    33rd USENIX Security Symposium |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | ------------------ | --- |

criticalobjectsratherthanprotectingeveryobjectindiscrimi-
nately.Weexploredthisnewstrategyandprototypeitinthe
designofSeaK.Inthissection,wewillfirstdescribetheeBPF
ecosystemontopofwhichwebuildSeaK,followedbythe
threatmodelandthedesignoverview.
4.1 TheeBPFEcosystem
TheextendedBerkeleyPacketFilter(eBPF)isanin-kernel
virtualmachinethatallowsprivilegeduserstorunprograms
inthekernelspace.TheeBPFprogramsarewrittenintheC
Figure2:Increaseofmemoryusageofexistingsecurityfeatures
languageandcompiledtobytecodeusingtheLLVMeBPF
whenrunningLMbench.
backend.PrivilegeduserscaninstallthecompiledeBPFpro-
gramsintothekernelandattachthemtoarbitraryinstructions,
Vanilla C1 C2 C3 monitoringandmodifyingkernelbehaviors.
Inst.#(million) 1,661,099 1,641,186 1,673,330 2,708,439 ThesafetyofeBPFprogramsisguaranteedbyastaticver-
Invoked#/sec 324,194 321,347 321,634 214,793 ifierwhichestablishesthreeproperties: (1)memorysafety,
TimeProportion 17.1% 17.1% 17.7% 41.4% ensuring that the program only accesses pre-defined mem-
TimePerReq(µs) 17.8 17.7 17.7 33.2 orylocations,(2)informationflowsecurity,ensuringthatno
secretkernelstateisexposed,and(3)allexecutionmustter-
Table2:StatisticscollectedthroughtheApacheBenchmarkinfive
minate.Overdecadesofdevelopment,eBPFhassignificantly
minutes.“Inst.#(million)”indicatesthe#ofinstructionsexecuted
improvedinexpressiveness.TheeBPFhelperfunctionsal-
withinthekernel,representedinmillions.“Invoked#”isthe#of
lowtheinstalledeBPFprogramstointeractwithotherkernel
timestheallocatoriscalled.“TimeProportion”isthefractionof
subsystems. The BPF maps can store arbitrary data. They
timespentwithintheallocatorinsidethekernel.“TimePerReq”
alloweBPFprogramstocommunicatewitheachotherand
averagestimerequiredtohandlearequest.
withprivilegeduserspaceprocessesbylookingupandupdat-
ingthestoreddatathroughkeys.Forthesakeofefficiency,
inagiventime,thekernelwithsecurityfeaturesexecutesa avarietyofoptimizationtechniqueshasbeenadoptedinthe
substantiallyhighernumberofinstructions,particularlyfor eBPFecosystem.Forinstance,theeBPFbytecodeisexecuted
C3whichexhibitsa63.1%increasecomparedtovanilla.This byaJust-In-Time(JIT)engineratherthanaslowinterpreter
isattributedtoC3’smostcompletespectrumofsecurityfea- toachievenative-machinecodelevelperformance.Further,
tures,whichintroducesnumerousadditionalinstructionsto theinstructionsonceattached,areoverwrittentocallorjmp
thekernelallocator.Further,weobservedthatthekernelallo- instructionsinsteadofthepreviouslyusedint3interrupt.As
catorisinvokedveryfrequently,reaching214,793to324,194 such,thetimespentonswitchingcontexttoeBPFprograms
timespersecond.Asaresult,inC3,thetimeproportiondedi- issaved.
catedtotheallocatordoubles,jumpingfrom17.1%to41.4%. eBPF for Security. The eBPF ecosystem has undergone
Thisbloatingnotonlydeceleratesthekernel,asevidencedby swift evolution across commodity OS kernels, including
thereducednumberofallocatorinvocationsinagiventime, Linux, Windows, FreeBSD, and macOS [5,6]. In the se-
butalsoimpactstheentiresystem,leadingtolongerresponse curity area,PET [66] instruments eBPF programs to error
timesforApachetohandlearequest. sites,preventingkernelvulnerabilitiesfrombeingtriggered.
Conclusion.Basedonouranalyticalexperiments,wehavethe Sifter [32] filters malicious syscall with eBPF to make at-
followingconclusion.First,theallocatorisacoresubsystem tacksurfacesinsecurity-criticalkernelmodulesunreachable.
inthekernelandisfrequentlyused.Second,eachtimethe RapidPatch [31] allowsRTOS developers to hot-patchem-
kernelinvokestheallocator,itincurstheoverheadassociated beddeddevicefirmwareusingeBPF.
with executing the additional code introduced by security
features.Giventhesefacts,thecurrentstrategyofprotecting
4.2 Threatmodel
everyobjectallthetimecanhardlyworkwithinthekernel
whichmustofferservicesforuserspaceefficiently. SeaKhasthefollowingassumptionsaboutthecapabilitiesof
attackersanddefenders.
4 DesignOverview Attacker.Attackerspossessavulnerabilitythatcorruptsthe
kernel heap. This vulnerability can be exploited using the
Drawingonlessonslearnedfromevaluatingexistingfeatures, newest kernel exploitation techniques. It can be present in
amorerealisticstrategywouldbeconservingresourceson anykernelsubsystem,includingtheeBPFsubsystemontop
USENIX Association 33rd USENIX Security Symposium 1175

|                      |     | eBPF Synthesizer |     | User Space   |     |     | 1 // essential             |           | utilities |          |      |     |
| -------------------- | --- | ---------------- | --- | ------------ | --- | --- | -------------------------- | --------- | --------- | -------- | ---- | --- |
|                      |     |                  |     |              |     |     | 2 int alloc_handler(struct |           |           | pt_regs* | ctx, |     |
|                      |     |                  |     |              |     |     | 3 u64                      | kpi_type) | {...}     |          |      |     |
|                      | …   |                  |     | Kernel Space |     |     |                            |           |           |          |      |     |
| 0xffffffff81c32ff3:  |     |                  |     |              |     |     | 4 int free_handler(struct  |           |           | pt_regs* | ctx, |     |
alloc
| callq  <kmalloc> |     | handler |     |     | dedicated  |     | 5 u64 | kpi_type) | {...} |     |     |     |
| ---------------- | --- | ------- | --- | --- | ---------- | --- | ----- | --------- | ----- | --- | --- | --- |
BPF
|                      | …   |     |      |     | region |     | 6   |     |     |     |     |     |
| -------------------- | --- | --- | ---- | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
| 0xffffffff81c331db:  |     |     | Maps |     |        |     |     |     |     |     |     |     |
free  “guard pages +  7 SEC("kprobe/?") // attach to allocation site
| callq  <kfree> |     | handler |     |     | 43 entropy” |     |                     |     |         |          |      |     |
| -------------- | --- | ------- | --- | --- | ----------- | --- | ------------------- | --- | ------- | -------- | ---- | --- |
|                |     |         |     |     |             |     | 8 int probe_alloc_? |     | (struct | pt_regs* | ctx) | {   |
Runtime Kernel
|     |     |     |     |     |     |     | 9 return           | alloc_handler(ctx, |     |        | ?);          |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | ------------------ | --- | ------ | ------------ | --- |
|     |     |     |     |     |     |     | 10 }               |                    |     |        |              |     |
|     |     |     |     |     |     |     | 11 SEC("kprobe/?") |                    | //  | attach | to free site |     |
Figure3:Overviewofoneatomicalleviation(AA)inSeaK.The
|     |     |     |     |     |     |     | 12 int probe_free_? |     | (struct | pt_regs* | ctx) | {   |
| --- | --- | --- | --- | --- | --- | --- | ------------------- | --- | ------- | -------- | ---- | --- |
eBPFprogramsynthesizerintheuserspaceinstallsaneBPFprogram 13 return free_handler(ctx, ?);
| withallochandlersandfreehandlersintothekernelspace,separating |     |     |     |     |     |     | 14 } |     |     |     |     |     |
| ------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- |
objectsofinterest.
Listing1:Thesnippetofsynthesistemplate.Oncethe"?"isfilled
in,itcanbedirectlycompiledandinstalledintothekernelspace.
ofwhichSeaKisbuilt,aslongastheeBPFprogramscanbe
installed.Further,weassumeattackersareunprivilegedusers
sothattheycannotdirectlydisableSeaK.
orrequiresupdates,theeBPFprogramcanexitgracefullyand
bereinstalledlaterifneededagain.
Defender.Defendersareprivilegedusersorsystemadmin-
istratorssothatSeaKisgrantedtherightprivilegetoinstall Likesecureallocatorsinuserspace(e.g., [19,45,57,59]),to
preventspatialoverlapping,thededicatedregionisequipped
eBPFprogramsintothekernel.TheinstalledeBPFprograms
withguardpagesandrandomizationwithupto43entropy.
arefreefrombugsandtheirsafetyisguaranteedbythesound
|     |     |     |     |     |     |     | To prevent | temporal | overlapping, |     | SeaK currently | enforces |
| --- | --- | --- | --- | --- | --- | --- | ---------- | -------- | ------------ | --- | -------------- | -------- |
verified.Weassumethatdefendersareawareofthevulnerabil-
themostrestrictiveseparationpolicytodealwiththenewest
itythreatbyeithercatchingitsexploitinthewildorobtaining
necessaryinformation(e.g.,vulnerabilityreports)frompublic exploitation techniques (Section 2.2): only recycle within
objectsthatareallocatedfromthesamesite,havingthesame
| resources | including | the dashboard | of  | Syzkaller– |     | the most |     |     |     |     |     |     |
| --------- | --------- | ------------- | --- | ---------- | --- | -------- | --- | --- | --- | --- | --- | --- |
size,carryingthesameprivilegelevel,andinthesamezone.
widelyusedkernelfuzzerdevelopedbyGoogle,theNational
VulnerabilityDatabase,andmore. Advantages.SeaKisflexiblefromthefollowingperspectives.
❶Design-wise,oneAAoffersthemostgranularlevelofal-
|     |     |     |     |     |     |     | leviation. | We can | strategically |     | orchestrate | different sets of |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ------ | ------------- | --- | ----------- | ----------------- |
4.3 SeaKataGlance
|     |     |     |     |     |     |     | AAs to meetdistinctsecurity |     |          |            | needs. Itallows | efficientuse |
| --- | --- | --- | --- | --- | --- | --- | --------------------------- | --- | -------- | ---------- | --------------- | ------------ |
|     |     |     |     |     |     |     | of resources                | by  | focusing | on crucial | objects,rather  | than an      |
Theatomicalleviation(AAforshort)isthekeyconceptin
indiscriminateseparation.❷Deployment-wise,AAscanbe
SeaK.OneAAisresponsibleforseparatingaspecifictype
enabledontheflywithoutdisruptingrunningcomputationser-
ofkernelobjects-objectsofinterest.Figure3illustratesthe
vices,thusmaintainingsystemavailability.❸Evolution-wise,
designofoneAAinSeaK.
|     |     |     |     |     |     |     | The separation |     | policy | enforced | in AA can | be dynamically |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --- | ------ | -------- | --------- | -------------- |
Inuserspace,theeBPFsynthesizerfirstanalyzestheker-
upgradedwhennewexploitationtechniquesaredisclosed.In
nelsourcecodetopinpointtheallocationandfreesitesfor
comparison,theimplementationofexistingsolutionsisfixed
| objects | ofinterest. | Then,itinvestigates |     | debug | information |     |     |     |     |     |     |     |
| ------- | ----------- | ------------------- | --- | ----- | ----------- | --- | --- | --- | --- | --- | --- | --- |
onceintegrated.
tomapthesource-code-levelsitesintocorrespondingbinary
addressesofcallinstructionsthatinvokememoryallocation
andfreefunctions(e.g.,kmalloc/kfree).Afterthis,thesynthe- 5 TechnicalDetails
| sizerproduces |     | an eBPF program | that | can | be installed | into |     |     |     |     |     |     |
| ------------- | --- | --------------- | ---- | --- | ------------ | ---- | --- | --- | --- | --- | --- | --- |
kernelspacetoachievealleviation. Inthissection,wewillpresentmoretechnicaldetailsofAA
inSeaK,fromtheeBPFsynthesizerintheuserspacetothe
Inthekernelspace,theeBPFprogramshouldersacomplete
allocation and free logic for objects of interest: The alloc runtimeseparationinthekernelspace.
handlerattachedtotheallocationsitesinterceptstheoriginal
allocationandobtainsmemoryfromadedicatedregionthatis 5.1 eBPFProgramSynthesis
separated;thefreehandlerattachedtothefreesitesprevents
thememoryindedicatedregionsfrombeingdirectlyreturned Toseparateobjectsofinterest,aneBPFprogramisgenerated.
tothebuddysystemandrecyclesmemoryonlyforobjects TheessentialelementsneededtoconstructtheeBPFprogram
thatareallowedbythesecuritypolicy.Thestatusofdedicated include the binary addresses ofthe allocation site andfree
regionsismaintainedbyBPFmaps.Ifonededicatedregion site,as well as the prototype of the kernel function that is
runsoutofspace,additionalmemorywillbeassigned.When called forallocation orfree. Readers can referto List 2 in
thethreatlandscapechangesandtheAAisnolongeruseful Appendixfortheillustrationofasynthesizedprogram.
| 1176    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

| Synthesis     | Template.    | List      | 1 shows | the              | template   | SeaK uses |         |     |                        |              |     |     |
| ------------- | ------------ | --------- | ------- | ---------------- | ---------- | --------- | ------- | --- | ---------------------- | ------------ | --- | --- |
|               |              |           |         |                  |            |           | alloc   |     | Key: ip-size-priv-zone |              |     |     |
| to synthesize | eBPF         | programs. | All     | the              | "?" in the | template  |         |     |                        |              |     |     |
|               |              |           |         |                  |            |           | handler |     | Value: region          | region-index |     |     |
| represent     | the elements | that      | need    | to be customized |            | per site. |         |     |                        |              |     |     |
Inline7andline11,SECdecorationdenotesthelocationsof free  Key: addr obj2region
theallocationandfreesitesintheformatoffunc+offset.In handler Value: ip-size-priv-zone
thisformat,funcisthesymbolofthekernelfunctionwhere
rand
allocationandfreearecalled,offsetrepresentstheoffsetof … guard  object guard  object …
|     |     |     |     |     |     |     |     | page offset |     | page |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ---- | --- | --- |
thecallinstructionfromthestartoffunc.Weusefunc+offset
|     |     |     |     |     |     |     |     | guard  | guard  |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------ | --- | --- | --- |
instead of the absolute address of the call instruction (i.e., … object …
|     |     |     |     |     |     |     |     | page |     | page |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | --- | ---- | --- | --- |
0xffffffff81c331db)becauseofKernelAddressSpaceLayout
dedicated regions
| Randomization | (KASLR) |     | [28] which | randomizes |     | the base |     |     |     |     |     |     |
| ------------- | ------- | --- | ---------- | ---------- | --- | -------- | --- | --- | --- | --- | --- | --- |
addressofkernelimageduringboottime.Usingfunc+offset,
|     |     |     |     |     |     |     | Figure 4: | Run-timeseparationinthekernelspace.Theattached |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------- | ---------------------------------------------- | --- | --- | --- | --- |
theeBPFprogramcanworkacrossmachineswithdifferent mallochandlersandfreehandlersleveragetwoBPFmapstomanage
| baseaddresseswithoutfurthermodification. |       |         |            |     |               |       | dedicatedregions. |     |     |     |     |     |
| ---------------------------------------- | ----- | ------- | ---------- | --- | ------------- | ----- | ----------------- | --- | --- | --- | --- | --- |
| In lines                                 | 8 and | 12, the | "?" serves | as  | a placeholder | for a |                   |     |     |     |     |     |
uniqueidentifiertodifferentiatemultipleallocationsitesand
free sites. In line 9 and 13,the "?" is used to differentiate kmalloc/kfree series functions at the source code level are
prototypesofallocationandfreefunctionsbeingcalled.This occasionallyinlinedintokmem_cache_alloc/kmem_cache_freese-
KernelProgrammingInterface(KPI)information(kpi_type) riesatthebinarylevelthroughcompileroptimization.There-
is needed so that the eBPF programs (line 1-5) can deter- fore,wesynthesizethekpi_typepartintheeBPFprograms
minehowtoobtaintherequestedsizeoftheallocationand accordingtotheseriesusedinthekernelimageratherthan
theaddressofobjecttobefreedfromtherun-timecontext. thesourcecodetoeliminateinaccuracies.
| For example,we |     | differentiate | kmalloc | and | kmem_cache_alloc |     |     |     |     |     |     |     |
| -------------- | --- | ------------- | ------- | --- | ---------------- | --- | --- | --- | --- | --- | --- | --- |
becausetheallocationsizeisstoredinthe1stparameterof
kmallocbutinthe2ndparameterofkmem_cache_alloc,passed 5.2 Run-timeSeparation
through$rdiand$rsi,respectively,bythex64convention.
Here,wedelveintomoredetailsofhoweBPFprogramsand
DeterminingAllocationandFreeSites.Giventheobject
BPFmapsseparateobjectsofinterest.
| type, SeaK | finds | the allocation |     | sites and | free sites | by first |     |     |     |     |     |     |
| ---------- | ----- | -------------- | --- | --------- | ---------- | -------- | --- | --- | --- | --- | --- | --- |
DataStructures.AsillustratedinFigure4,inoneAA,an
| searching | at the source | code | level | and then | converting | the |     |     |     |     |     |     |
| --------- | ------------- | ---- | ----- | -------- | ---------- | --- | --- | --- | --- | --- | --- | --- |
sitesintothecorrespondingbinaryaddressesintheformatof alloc handleris attached to each allocation site,and a free
handlerisattachedtoeachfreesite.Thesynthesizerdescribed
func+offset.
inSection5.1isresponsibleforidentifyingthesesitesand
Atthesourcecodelevel,theSLAB/SLUBallocatoruses
synthesizingthecorrespondinghandlers.
twoseriesofkernelfunctionsforallocationandfree.Oneis
kmalloc/kfreeseriesforobjectsinthegeneralcache.Theother To track the status of dedicated regions that store sepa-
ratedobjects,eachAAhastwoBPFmaps:region-indexand
| is kmem_cache_alloc/kmem_cache_free |     |     |     | series | for special | cache. |     |     |     |     |     |     |
| ----------------------------------- | --- | --- | --- | ------ | ----------- | ------ | --- | --- | --- | --- | --- | --- |
Forthebuddysystem,thefunctionsforallocationandfreeare obj2region.Theregion-indexmapisusedtolocatededicated
regions.Itskeyiscustomizedaccordingtothesecuritypolicy.
alloc_pages/free_pagesseries.Thecodelinesthatcallthese
Itsvaluerepresentsthededicatedregionwhichistheaddress
| functions | are allocation |     | and free | sites. | We further | narrow |     |     |     |     |     |     |
| --------- | -------------- | --- | -------- | ------ | ---------- | ------ | --- | --- | --- | --- | --- | --- |
downthescopeandidentifythesitesspecifictotheobjectsof of either a struct kmem_cache object if the requested size is
|     |     |     |     |     |     |     | smallerthanonepageorastruct |     |     | pageobjectiftherequested |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------------------- | --- | --- | ------------------------ | --- | --- |
interestbyanalyzingthereturnvaluesorargumentsofthese
functioncalls.Becausetheirreturnvaluesorargumentsare sizeislargerthanonepage.Thekeyoftheobj2regionmap
|     |     |     |     |     |     |     | is the address | ofan | individualobjectthatis |     | separatedinto |     |
| --- | --- | --- | --- | --- | --- | --- | -------------- | ---- | ---------------------- | --- | ------------- | --- |
alwayspointersreferencingtheobjectsallocatedorfreed,by
dedicatedregions.Itsvalueindicateswhichregiontheobject
analyzingthem,wecaneasilyfigureoutwhethertheobjectis
ofinterest.Technically,weperformause-defanalysisandre- belongstoandcanbeusedtoindextheregion-indexmap.
solvememoryalias.Alongtheuse-defchainofreturnedvalue Evolving with Exploitation Techniques. In Figure 4,we
orarguments,wetrackinstructionsrelevanttotypecasting,
useip-size-priv-zoneasthekeytoindexthededicatedregion.
pointerdereferencing,andargumentpassing.Theoperands
Assuch,thememoryisrecycledonlywithinobjectsthatare
oftheseinstructionsexplicitlyrevealthetypeofvariables.By allocated from the same site (i.e., ip), with the same size,
usingthisinformation,wecaneasilyinferandconcludethe
|     |     |     |     |     |     |     | carrying | the same | privilege level,and | in  | the same zone. | It  |
| --- | --- | --- | --- | --- | --- | --- | -------- | -------- | ------------------- | --- | -------------- | --- |
typeofeachallocatedorfreedobject. is the most restrictive policy that can alleviate the newest
Atthebinarylevel,sitesinthesourcecodethatallocate exploitationtechniquesthathavebeendisclosedthusfar.As
or free objects of interest are mapped to binary addresses newexploitationtechniquesevolveinthefuture,thispolicy
via debugging information in the kernel image. Note that, canbestrengthenedaccordingly.
| USENIX Association |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1177 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

Guard Page and Randomization. Each separated object perkeyandaread-copy-update(RCU)counterpervaluein
occupies multiple pages. Even for the object the real size BPFmapstoachieveconcurrencycontrol.
ofwhichissmallerthanonepage,ittakesoveratleastone WhenoneAAisnolongerneededorastrongerpolicyisde-
page.Thesepagesarefurthersurroundedbyguardpagesthat veloped,itexitsgracefullywithoutcausingmemoryleakage.
are not mapped. The offset of the object from the start of Morespecifically,SeaKcreatesakerneltaskthatperiodically
thepagesisrandomized.Therandomizationentropywithin scanstheentirekernelmemorytocheckforpointersreferring
the page is 7,considering pointer alignment in x64. Since to the separated objects indexed in the obj2region map. If
the dedicatedregion can be locatedanywhere in the direct nomorepointersrefertotheseparatedobjects,theoccupied
mappingarea,theoverallentropyis21forthe16MBDMA memorycanbesafelyreturnedtothebuddysystem.Other-
zoneor43forthe64TBnormalzone,whichismuchhigher wise,the AA stays in the kernel until the obj2region map
thanthewidely-usedKASLRwhichis8.Givensuchahigh becomesempty.
entropy,thechanceforattackerstobrute-forcetheaddressof
separatedobjectsinoneshotisnearlyzero.
6 Application
Workflow.Now,wedescribehowtheaforementioneddesigns
Inthissection,weshowcasehowtoorchestrateasetofAAs
areusedthroughthelifecycleofanobjectthatisseparated.
tomeetsecurityrequirementsindifferentscenarios.
Whentheobjectisallocated,theallochandlerwillbeex-
ecuted.Accordingtothesecuritypolicy,itobtainstheexe-
6.1 SeparatingSecurity-SensitiveObjects
cutioncontext:(1)theaddressofallocationsitesfrom$rip
register,(2) the request size from $rdi for kmalloc,$rsi for Givennotallobjectsaresecurity-sensitive,protectingeach
kmem_cache_alloc,$rsi*PAGE_SIZEforalloc_pages.(3)thepriv- oneindiscriminatelyisunnecessary.Kernelobjectssuchas
ilegelevelofthecurrentprocessthroughthehelperfunction, cred,msg_msg,andkey_payloadarewell-knowntobesecurity-
(4)therequestedzonewhichisspecifiedinGFPflags-$rsi sensitivebecausetheycontaindatalikecredentialsandfunc-
for kmalloc, $rdx for kmem_cache_alloc, $rdi for alloc_pages. tionpointers.Separatingthemisonekeytaskinpriorworks,
This context information is concatenated to form the key. including xMP [53],kalloc_type [30], AutoSlab [40],and
The allochandleruses the keyto lookupthe region-index slab_virtual[64].SeaKcancompletethistaskinastraightfor-
map to examine if there is already a dedicated region that wardway:eachAAseparatesonespecificsensitivetype.
fits.Ifnodedicatedregionisavailable,theallochandlercre- Aseriesofresearcheffortshavebeenundertakentoidentify
ates a new slab cache or a new buddy through the helper security-sensitiveobjectsintheLinuxkernel:SLAKE[22]
functionandrecordsrelatedinformationintheregion-index collectsobjectswithfunctionpointers;ELOISE[21]focuses
map.Otherwise,theallochandlerretrievestheslabcacheor on elastic objects that can provide stronger write and read
buddystructurefromtheregion-indexmap,storingtheobject primitives; AlphaExp [65] finds objects based on within-
andsetting guardpages throughhelp functions. Following cacheexploitationscheme.Asexploitationtechniquesevolve,
this,the address of the object is used as the key to update moreobjectswillbeidentifiedassecurity-sensitive.Bycon-
theobj2regionmap.Finally,theallochandleroverwritesthe structingandinstallingmoreAAs,SeaKcaneasilyadaptto
returnaddresstodirectlyjumptothenextinstructionofallo- these newly discovered objects. This flexibility is a signif-
cationcallandthusskiptheoriginalallocation. icant advantage over prior works because their design and
Whentheobjectisfreed,thefreehandlerwillbeexecuted.
implementationarefixed.Moreover,SeaKcanenforcesepa-
Itfirstobtainsobject’saddressfromthe$rdiregisterandlooks rationontheflywhilepriorworksrequirerecompilationand
itupintheobj2regionmaptodetermineiftheobjecttosepa- rebootingwhichdisruptssystemavailability.
rated.Ifnot,thekernelcontinuestheroutinefreeoperation.
Otherwise,thefreehandlerlooksuptheregion-indexmap, 6.2 SeparatingVulnerabilityCorruptions
obtains the dedicatedregion,andfrees the objectfrom the
regionusingahelperfunction.Notethat,thisfreeoperation Due to the lack of manpower and design complexity, the
doesnotindeedreturnthememory.Instead,thememoryis patching process is quite lengthy for the Linux kernel. A
recycled forthe subsequent allocations of objects with the studytwoyearsago[61]revealedthattheaveragepatching
sameip-size-priv-zonekey,asthesecuritypolicyrequires. windowintheLinuxkernelwas66days,andthesituationis
gettingworse[8].Duringthistimewindow,patchesarenot
ConcurrencyandExit.Sinceobjectsofinterestcanbeallo- availablefortheseN-dayvulnerabilitiesandattackershave
catedandfreedatmanykernelsites,theallochandlersand thefullfreedomtodevelopexploitsandlaunchattacks.
freehandlersattachedtothesesiteswillconcurrentlyaccess Tomitigatethisthreat,wecanconstructAAstoseparate
thesharedregion-indexandobj2regionmaps.Therefore,it corruptionsintroducedbyvulnerabilities,therebylimitingthe
is essential to ensure the atomicity of map read and write. damage.Forvulnerabilitiessuchasheapout-of-boundwrite
Fortunately,thecurrenteBPFecosystemprovidesaspinlock andread,corruptionhappenswhenthereisbeyond-boundary
1178 33rd USENIX Security Symposium USENIX Association

access.OneAAcanbeinstalledtoseparatetheoverflowed andbpf_create_buddytocreatededicatedregions,bpf_get_zone
object.Thus,guardpagescancatchthespatialoverlapping toobtaintherequestedzone(e.g.,normalzoneorDMAzone),
caused by the vulnerability,and due to randomization,the bpf_cache_allocandbpf_buddy_alloctoallocatememoryinthe
overlappingcannotpreciselytamperwithtargetedkerneldata. dedicatedregions,andbpf_set_pt_presenttosetguardpages.
Forvulnerabilitiessuchasuse-after-free,corruptionhappens ThetwoBPFmaps,region-indexandobj2region,usedinthe
when a dangling pointerdereferences a freedobject. After eBPFprogramarebothoftypeBPF_MAP_TYPE_HASH,whichsup-
separatingthefreedobjectusingoneAA,theheapmemory ports quick lookup and update. The maximum number of
entriesforbothmapsissetto214,allowingthemanagement
isrecycledonlyforobjectsthatareallowedbytheseparation
policy,whichisrestrictiveenoughtofailnewestexploitation of up to 16,384 dedicated regions. Our implementation is
attempts.Furthermore,SeaKcreatesataskthatperiodically basedonv5.15thelatestLong-termsupport(LTS)kernelver-
scans the entire kernel memory to check for the existence sionwhenwedidourexperiments.Itcanbeeasilymigrated
ofdanglingpointersandstartsrecyclingonlywhenthereis tootherversionswithminormodifications.Weleveragethe
nodanglingpointer.PresentedwithmultipleN-dayvulner- LLVMtoolchaintocompileeBPFprograms.
abilities,weemployasetofAAstoprotectthekerneluntil
SupportforLoadableKernelModules(LKMs).Insome
patchesforthesevulnerabilitiesarereleased.
|     |     |     |     |     | corner cases,objects |     | of  | interest | are allocated |     | and freed by |
| --- | --- | --- | --- | --- | -------------------- | --- | --- | -------- | ------------- | --- | ------------ |
loadablekernelmodules.Withoutloadingthesemodules,the
attachedaddresscannotbedetermined.Todealwiththisis-
7 Implementation
|     |     |     |     |     | sue, we       | first install | an eBPF  | program |         | that attaches | to the    |
| --- | --- | --- | --- | --- | ------------- | ------------- | -------- | ------- | ------- | ------------- | --------- |
|     |     |     |     |     | load_module() | kernel        | function | to      | monitor | which         | module is |
TheimplementationofSeaKincludes416linesofCcodefor
loaded.Oncethemoduleisloaded,itsendsoutsignals,allow-
eBPFtemplates,152linesofCcodeforthenewhelperfunc-
tions in thekernel,and649 linesofPython code foreBPF ingseparation-purposedeBPFprogramstobeinstalled.Note
|                   |     |           |              |              | that SeaK | does not | need | the absolute |     | address | of the kernel |
| ----------------- | --- | --------- | ------------ | ------------ | --------- | -------- | ---- | ------------ | --- | ------- | ------------- |
| program synthesis | and | framework | integration. | In addition, |           |          |      |              |     |         |               |
SeaKhas3998linesofC++codebasedonLLVMinfrastruc- moduleastheattachedsitesarebasedonsymbols.
ture,topinpointallocationandfreesitesforobjectsofinterest SupportforApplicationScenarios.Toidentifyvulnerable
andsupportthescenarioofseparatingvulnerabilitycorrup-
|     |     |     |     |     | objects | thatintroduce | corruption,we |     | employthe |     | approach |
| --- | --- | --- | --- | --- | ------- | ------------- | ------------- | --- | --------- | --- | -------- |
tionbyidentifyingvulnerableobjects.SeaKisimplemented
inpriorwork[43]toanalyzereportsgeneratedbysanitizers
overLinuxandcanbemigratedtootheropen-sourcekernels suchasKASAN,KMSAN,KCSAN,andmore.Wepayspe-
thankstotheconsistentdesignoftheeBPFecosystemacross
cialattentiontothesoundnessofouranalysisbyincluding
platforms.SeaKisopen-sourceinGPLv2Licence1.
situationsnotpreviouslyconsidered.Morespecifically,some
eBPF Program Synthesis. Given the type of objects for kernelobjectsareordinaryarraysanddonotbelongtoany
separation,SeaKanalyzesthekernelimagewithdebuginfor- structureoruniontype(e.g.,char* p = kmalloc(0x10)).Ifwe
naivelytreatchar*asthevulnerabletype,SeaKwillinevitably
mationtopinpointallocationsitesandfreesitesinorderto
synthesizeeBPFprograms.WeuseLLVMinfrastructureto isolateanumberofirrelevantarraysinthekernel,resulting
inunnecessaryoverhead.Throughinvestigation,weobserve
searchforthesesitesofinterestatthesourcecodeleveland
useBinaryNinjatomaptheresultstobinaryaddresses.More that these arrays are either referenced by a pointerfield in
specifically,we provide a list of kernel allocation and free astructure(e.g.,bitmap_ip.members)sothattheycanbeused
acrosssystemcalls,orusedasatemporarybufferthatwill
| KPIs (e.g., kfree, | kmem_cache_free, |     | kfree_skbmem, | etc) for Bi- |     |     |     |     |     |     |     |
| ------------------ | ---------------- | --- | ------------- | ------------ | --- | --- | --- | --- | --- | --- | --- |
naryNinja.BinaryNinjacangettheaddressesofallsymbols, bepassedasafunctionargument,whichcanbetrackedby
|     |     |     |     |     | our analysis. | Therefore,for |     | each | array | in the | analysis,we |
| --- | --- | --- | --- | --- | ------------- | ------------- | --- | ---- | ----- | ------ | ----------- |
cross-referencethemtotheircallsites,retrievetheaddress
ofeachcallsite,andthenwritethemintoafile. Afterthat, create an anonymous type type_name+offset for differentia-
eachaddressinthisfileispassedtollvm-symbolizertocreate tion.Here,type_nameindicatestheassociatedstructuretype
|                                                    |     |     |     |     | (e.g.,struct | bitmap_ip) | and |        | records | the | offset of the |
| -------------------------------------------------- | --- | --- | --- | --- | ------------ | ---------- | --- | ------ | ------- | --- | ------------- |
| anallocationsitemappingbetweenthekernelimagebinary |     |     |     |     |              |            |     | offset |         |     |               |
andthesourcecode.ThemappingitselfisstoredasaPython pointerfield(e.g.,members).wepatchtheLLVMcompilerto
dumpbitcodesbeforeanyoptimizationpasses,thusprevent-
| dictionary within | a .pickle | file. | We can | easily look up the |     |     |     |     |     |     |     |
| ----------------- | --------- | ----- | ------ | ------------------ | --- | --- | --- | --- | --- | --- | --- |
ingcompileroptimizationfrominfluencingtheaccuracyof
mappingfortheexactcallinstructionsthatallocateandfree
| objects of interest. | All | the steps | and sub-steps | mentioned | ouranalysis. |     |     |     |     |     |     |
| -------------------- | --- | --------- | ------------- | --------- | ------------ | --- | --- | --- | --- | --- | --- |
abovearewrappedinaPythonscripttofullyautomatethe
entireanalysisworkflow.
8 Evaluation
NewHelperFunctions&BPFMaps.Tosupportseparat-
| ing objects ofinterest,we |     | extendthe | eBPF | mechanism by |     |     |     |     |     |     |     |
| ------------------------- | --- | --------- | ---- | ------------ | --- | --- | --- | --- | --- | --- | --- |
addingnewhelperfunctions,includingbpf_create_slab_cache Inthissection,weusereal-worldcasestoevaluateSeaKin
|     |     |     |     |     | terms of | security | improvement, |     | performance |     | and memory |
| --- | --- | --- | --- | --- | -------- | -------- | ------------ | --- | ----------- | --- | ---------- |
1https://github.com/a8stract-lab/SeaK overhead,aswellasscalabilityandstability.
| USENIX Association |     |     |     |     |     |     | 33rd USENIX Security Symposium    1179 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

Exploits SensitiveObjectType C1 C2 C3 SeaK theexploitationofspatialcorruptionbasedexploitation(e.g.,
2021-4154(exp1)[41] msgseg,pipe_buffer / exp4,exp6). Second,freelistobfuscationandheapzeroing
| 2021-22600(exp3)[16] |     | msg_msg,pipe_buffer |     |     | (cid:35) (cid:35)/(cid:35)(cid:72) | (cid:32) (cid:32) |         |                   |     |            |         |      |          |
| -------------------- | --- | ------------------- | --- | --- | ---------------------------------- | ----------------- | ------- | ----------------- | --- | ---------- | ------- | ---- | -------- |
|                      |     |                     |     |     |                                    |                   | are not | activated because |     | no exploit | tampers | with | freelist |
| 2022-0185(exp4)[24]  |     | msg_msg,pipe_buffer |     |     | (cid:35)/(cid:35)(cid:72)          |                   |         |                   |     |            |         |      |          |
(cid:35) (cid:32) (cid:32) pointerandreliesonuninitializedvalueforKASLRbypass-
2022-27666(exp6)[73] xattr,xfrm_policy (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) (cid:32) ing. Instead,theyusethereadcapabilityintroducedbythe
| 2022-29582(exp9)[56] |     |     | msgseg,tls_context |     | (cid:35) (cid:35)/(cid:35)(cid:72) | (cid:32) (cid:32) |     |     |     |     |     |     |     |
| -------------------- | --- | --- | ------------------ | --- | ---------------------------------- | ----------------- | --- | --- | --- | --- | --- | --- | --- |
vulnerabilitytoleakkernelbaseaddress.
| 2022-1786(exp13)[69] |     |     | timerfd_ctx |     | (cid:35) (cid:35)/(cid:35)(cid:72) | (cid:32) (cid:32) |     |     |     |     |     |     |     |
| -------------------- | --- | --- | ----------- | --- | ---------------------------------- | ----------------- | --- | --- | --- | --- | --- | --- | --- |
ForC2,structurelayoutrandomizationfailstopreventall
| 2022-20409(exp15)[42] |     |     | cred |     | (cid:35) (cid:35)/(cid:35)(cid:72) | (cid:32) (cid:32) |     |     |     |     |     |     |     |
| --------------------- | --- | --- | ---- | --- | ---------------------------------- | ----------------- | --- | --- | --- | --- | --- | --- | --- |
(cid:35) (cid:35)(cid:35)(cid:72) (cid:35) (cid:32) exploitsbecauseitonlyrandomizesstructureswitheveryfield
Table 3: Results ofSeaK’s security improvementforseparating asafunctionpointer.Noneofthesensitiveobjectsmisusedin
security-sensitiveobjects.The“Exploits”columnincludesCVEIDs
theexploitfallintothiscategory.KFENCEsamplesobjects
andtheinternalexploitIDfromGoogle.The“SensitiveObjectType”
|     |     |     |     |     |     |     | for separation. | During | the | experiment, | we  | maximized | the |
| --- | --- | --- | --- | --- | --- | --- | --------------- | ------ | --- | ----------- | --- | --------- | --- |
meansthetypeofobjectsmisusedintheexploit. indicatesfailing capabilityofKFENCEbyminimizingitssamplinginterval
| topreventexploitation, |     |                                                          | standsforworkingocca(cid:35)sionallyduetothe |     |     |     |         |              |     |        |        |                |     |
| ---------------------- | --- | -------------------------------------------------------- | -------------------------------------------- | --- | --- | --- | ------- | ------------ | --- | ------ | ------ | -------------- | --- |
|                        |     |                                                          |                                              |     |     |     | to 1 ms | andexpanding | its | memory | poolto | its limitof512 |     |
| samplingnature,        |     | mea(cid:72)(cid:35)nssucceedinginpreventingexploitation. |                                              |     |     |     |         |              |     |        |        |                |     |
MB.Wediscoveredthatitcanprotectonly0.005%-0.35%
(cid:32)
sensitiveobjectsinTable3.
8.1 SecurityAnalysis ForC3,slub_debugsuccessfullythwartsmostexploitsex-
ceptforexp15.ThisexploitperformsDirtyCredattack[44]
ToevaluatethesecurityimprovementofSeaK,wedrawacom-
whichoverlapstwocredcarryingdifferentlevelsofprivilege,
parisonbetweenitandexistingsecurityfeatures,nomatter therebyevadingdetectionthroughredzone,poisoning,and
whethertheyareenabledbydefaultornot.
usertracking.
Incomparison,SeaKpreventsallexploits,showcasingthe
| Dataset | & Criteria. | We  | built | two datasets | corresponding |     |     |     |     |     |     |     |     |
| ------- | ----------- | --- | ----- | ------------ | ------------- | --- | --- | --- | --- | --- | --- | --- | --- |
to the two illustrative scenarios. The firstdatasetis forthe strongestsecurityimprovement.Ononehand,guardpages
andoffsetrandomizationhinderexploitationutilizingspatial
scenarioofseparatingsecurity-sensitiveobjects.Itincludes
sevenexploitscollectedbyGoogleandAlphabetVulnerabil- overlapping,achieving the same effect as slub_debug. On
ityRewardProgram[36].Wedidn’tincludeallcasesbecause theotherhand,SeaKonlyrecyclesobjectsallocatedfromthe
samesite,havingthesamesize,carryingthesameprivilege
theremaininglackspubliclyavailable,functionalexploitcode.
Weselectedthisprogramasthedatasourcebecausethepro- level,andinthesamezone.Therefore,itcanhandlenotonly
gramreportclearlyspecifieswhichsecurity-sensitiveobject common temporal overlapping attacks but also the newest
isusedineachcase,whichsavesourtimeandavoidsinaccu- same-typeexploitationlikeDirtyCred.Moreover,whennew
raciesinidentifyingwhichtypeofobjectforseparation. attacksemergeinthefuture,SeaKsupportsupdatingtherecy-
clingpolicycorrespondinglytokeeppacewiththeevolving
Theseconddatasetisforthescenarioofseparatingvulner-
| abilitycorruption.Thisdatasetisconstructedusingvulnera- |     |     |     |     |     |     | threatlandscape. |     |     |     |     |     |     |
| ------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- |
bilitiesreportedbySyzkaller[15].EverycasefromSyzkaller
|             |     |          |               |     |          |            | Separating | VulnerabilityCorruption. |     |     | Table | 4 shows | the |
| ----------- | --- | -------- | ------------- | --- | -------- | ---------- | ---------- | ------------------------ | --- | --- | ----- | ------- | --- |
| encompasses | a   | report,a | configuration |     | file,and | a PoC pro- |            |                          |     |     |       |         |     |
sampledresultsforseparatingvulnerabilitycorruptionagainst
gram,allaidinginthereproductionofthevulnerability.We
theseconddataset.Theresultsshareasimilaritywiththefirst
randomlyselected50vulnerabilitiesreportedafterkernelver-
scenario:C1cannotrestrictcorruption,soisstructurelayout
| sion v4.15 | 2, successfully |     | reproducing |     | 46 of them. | These |     |     |     |     |     |     |     |
| ---------- | --------------- | --- | ----------- | --- | ----------- | ----- | --- | --- | --- | --- | --- | --- | --- |
randomizationinC2;KFENCEinC2worksoccasionallyif
| include | 30 reported | by  | KASAN,1 | by  | KMSAN,1 | by UB- |     |     |     |     |     |     |     |
| ------- | ----------- | --- | ------- | --- | ------- | ------ | --- | --- | --- | --- | --- | --- | --- |
thevulnerableobjectissampled.slub_debugcanseparateall
SAN,5byBUG_ONmacro,2byGPF,and7byWARNING
corruptionswhenproactiveattacksarenotpresent.
macro.Thediversitywithinthesevulnerabilitiesensuresthat
FPandFN.Toseparatevulnerabilitycorruption,SeaKiden-
thedatasetisrepresentative.
tifiesthevulnerableobjectanditsallocationandfreesites.
SeparatingSecurity-SensitiveObjects.Table3showsthe
Duringthisprocess,SeaKcanhaveFalsePositive(FP)-iden-
| results of | separating | security-sensitive |     |     | objects | against the |     |     |     |     |     |     |     |
| ---------- | ---------- | ------------------ | --- | --- | ------- | ----------- | --- | --- | --- | --- | --- | --- | --- |
tifyingobjectsthatarenotvulnerableorsitesthatallocating
| firstdataset. | In  | general,SeaK | outperforms |     | C1  | (i.e.,freelist |     |     |     |     |     |     |     |
| ------------- | --- | ------------ | ----------- | --- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- |
irrelevantobjects,andFalseNegative(FN)-missingvulnera-
randomization+freelistobfuscation+heapzeroing),C2(i.e.,
bleobjectsorsitesthatallocatevulnerableobjects.SeaKcan
structurelayoutrandomization+KFENCE),andalsoC3(i.e.,
accommodateFPbyseparatingmoreobjectsatthecostof
slub_debug=UFPZ).
|     |     |     |     |     |     |     | a minorincrease | in  | overhead,thanks |     | to its | scalability(See |     |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --------------- | --- | ------ | --------------- | --- |
ForC1,allexploitscanbypassby-defaultenabledfeatures
Section8.2).TheeliminationofFNreliesonthesoundness
init.First,freelistrandomizationessentiallycannotprevent
oftheanalysis.Toachievethis,weusedthestate-of-the-art
temporalcorruptionbasedexploitation(e.g.,exp1,exp3,exp9,
|     |     |     |     |     |     |     | techniques | in implementation |     | (See | Section | 7). However,it |     |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ----------------- | --- | ---- | ------- | -------------- | --- |
exp13,exp15)andisbypassedusingheapgrooming[39]in
isimportanttonotethat,todate,nowhole-programanalysis
issoundwhenappliedtorealprogramminglanguages[46].
2LLVMisunabletocompilekernelearlierthanthisversion,andmany
Therefore,thouoghempiricallySeaKdidn’toverlookanyvul-
eBPFfeaturesusedinSeaKdidnotexistatthetime.
| 1180    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

| SYZTitle                  |     |     | C1 C2 C3                  | TypeofIdentifiedVulnerableObject |                |     | SeaK     |
| ------------------------- | --- | --- | ------------------------- | -------------------------------- | -------------- | --- | -------- |
| GPF-delayed_uprobe_remove |     |     | /                         |                                  | delayed_uprobe |     |          |
| WARNING-call_rcu          |     |     | (cid:35)/(cid:35)(cid:72) |                                  | route4_filter  |     |          |
|                           |     |     | (cid:35) (cid:32)         |                                  |                |     | (cid:32) |
WARNING-ODEBUGbug-tcf_queue_work (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) route4_filter,workqueue_struct (cid:32)
KASAN-uaf-read-route4_get (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) Qdisc,route4_bucket,route4_filter,route4_head... (cid:32)
UBSAN-shift-oob-dummy_hub_control (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) urb,usb_ctrlrequest,usb_device,usb_hcd (cid:32)
KASAN-uaf-read-hci_send_acl (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) hci_chan,hci_conn,hci_dev,l2cap_conn,work_struct (cid:32)
BUG-corruptedlist-kobject_add_internal (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) hci_conn (cid:32)
KMSAN-uninit-value-geneve_xmit (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) netdev,sk_buff (cid:32)
KASAN-slab-oob-write-decode_data (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) tty_ldisc,tty_struct (cid:32)
|     |     |     | (cid:35) (cid:35)(cid:35)(cid:72) (cid:32) |     |     |     | (cid:32) |
| --- | --- | --- | ------------------------------------------ | --- | --- | --- | -------- |
Table4:SampledresultsofSeaK’ssecurityimprovementforseparatingvulnerabilitycorruption.MoreresultscanbefoundinTable9.The
“SYZTitle”columnisthebugreporttitleminusanuninformativepreposition.Structuresinboldin“TypeofIdentifiedVulnerableObject”
columnaregroundtruth.TheremainingsareFPs.SeaKhasnoFNinalltestcases. indicatesfailingtopreventcorruptionfromdamaging
otherkernelobjects, standsforworkingoccasionallyduetothesamplingnature,(cid:35) meanssucceedinginseparatingcorruption.
|     | (cid:72)(cid:35) |     |     | (cid:32) |     |     |     |
| --- | ---------------- | --- | --- | -------- | --- | --- | --- |
nerableobjectinalltestcases(SeeTable4and 9),wemust substantialamountofobjectslingeringinmemory,thuseasily
cautionusersabouttheriskofFNs-SeaKmightoccasionally exhaustingkernelmemory.
failtoseparatevulnerableobjects.
|     |     |     |     | Based on | the profiling | results, we consider | three repre- |
| --- | --- | --- | --- | -------- | ------------- | -------------------- | ------------ |
Comparison with Other Works. Beyond C1, C2, C3, in sentativesituations-Cold,Hot,andDurable.“Cold”refers
Section6.1,wementionedseveralworksalsoforobjectsepa- to an object whose allocation and free rarely happen. We
useKASAN-uaf-l2cap_chan_close[60]fromthedatasetfor
rationpurpose.Amongthem,xMP[53]andkalloc_type[30]
are comparable to SeaK security-wise. However,AutoSlab separatingvulnerabilitycorruptionasanexampleofthissit-
|     |     |     |     | uation,because | its vulnerable | object - struct | l2cap_chan is |
| --- | --- | --- | --- | -------------- | -------------- | --------------- | ------------- |
inadvertentlysimplifiescross-cacheexploitation[40]because
allocatedonlyinfunctionl2cap_chan_createintheBluetooth
itseparateskernelobjectsperslabcachewhicheasesrecy-
clingatthebuddysystemlevel.Googleslab_virtual’snewest module.“Hot”indicatesthattheobjectisfrequentlyallocated
andfreed,buthasashortlifespan.Ourprofilingshowsthat
implementation[63]preventstemporal-corruption-basedex-
ploitationbutpartiallyworksforspatial-corrution-basedex- struct seq_operationsisthehottestinbothnoworkloadsitu-
ationandrunningLMbenchsituation-allocated32.7times
ploitation.Besides,itcannothandleDMAobjectswhichmust
and47.34timespersecondrespectively.“Durable”standsfor
resideinDMAzone.Onadifferentnote,PET[66]prevents
vulnerabilitytriggering,focusingonadifferentstageofthe anobjectthathasalonglifespanandamoderateoperationfre-
|     |     |     |     | quency.Ourprofilingsuggestsstruct |     | cred,struct | sk_filter, |
| --- | --- | --- | --- | --------------------------------- | --- | ----------- | ---------- |
exploitationchainfromSeaK.Itcanmissvulnerabilitiestrig-
geredthroughdifferentpathsoratdifferentsites[43]. struct fdtable,becausetheyhave5s,20s,and30slifespan.
|     |     |     |     | Further,weincludestruct | fileintoourmeasurementbecause |     |     |
| --- | --- | --- | --- | ----------------------- | ----------------------------- | --- | --- |
“Everythingisafile”.
8.2 OverheadMeasurement PerformanceOverhead.Table8.2presentstheperformance
overheadofthethreerepresentativesituations.Intuitively,one
wouldanticipatetheoverheadof“Hot”isthehighest,given
| Benchmarks | andSettings. | Weemployedidenticalbench- |     |     |     |     |     |
| ---------- | ------------ | ------------------------- | --- | --- | --- | --- | --- |
marks and settings including variances for measuring the the correlation between performance and the frequency of
overheadofSeaKasthoseutilizedfortheexistingsecurity allocationandfree.However,thedataillustratesthat“Hot”
doesn’tshowanysignificantincreasecomparedtotheother
| features outlined | in Section | 3.2. In addition | to the vanilla |     |     |     |     |
| ----------------- | ---------- | ---------------- | -------------- | --- | --- | --- | --- |
kernelimage,webuiltahardenedkernelimageincorporating situations.Specifically,theoverheadrangesfrom-2.42%to
theSeaKextensions.BothimageshavetheBPFJITengine 2.70% for LMbench, and -1.73% to 1.64% for Phoronix,
enabled. Note that,we needn’t build a different kernel for whichiswithinreasonablemarginoffluctuationandaligns
each individualAA. Once the SeaK extension is there,any withthedatafor“Cold”and“Durable”.Therefore,wecon-
|     |     |     |     | clude that | SeaK has negligible | overhead,regardless | of the |
| --- | --- | --- | --- | ---------- | ------------------- | ------------------- | ------ |
numberofAAscanbeinstalled.
Weusedbpftracetoprofilethelifespanofkernelobjects benchmark and the use frequency of objects that are sepa-
rated.
for20minutesintwosituations-noworkloadandrunning
LMbench.Theprofilingrevealedthatmostkernelobjectsare Tofurthervalidateourconclusionandhaveadeeperun-
allocatedandfreedquicklywhileasmallfractionofobjects derstandingofSeaK’sperformance,wemeasuredthelatency
havelonglifespan.Noobjectshavebothalonglifespanand causedbycriticaloperationsinSeaK.Thepotentialoverhead,
frequent operations, as this combination would result in a ifthereisany,willcomefromeBPFprogramexecutionand
| USENIX Association |     |     |     |     | 33rd USENIX Security Symposium    1181 |     |     |
| ------------------ | --- | --- | --- | --- | -------------------------------------- | --- | --- |

|     | LMbench(ms)   |     | Vanilla       | Cold Hot |       | Durable | File          | MBytes |     |     |     |     |     |     |
| --- | ------------- | --- | ------------- | -------- | ----- | ------- | ------------- | ------ | --- | --- | --- | --- | --- | --- |
|     | Simplesyscall |     | 0.1942 -1.68% | -0.67%   | 0.06% | 0.08%   | -0.29% -0.94% | 1200   |     |     |     |     |     |     |
|     | Simpleread    |     | 0.2946 0.20%  | -0.58%   | 0.49% | -0.48%  | 0.03% -0.45%  |        |     |     |     |     |     |     |
|     | Simplewrite   |     | 0.2502 -2.67% | -2.42%   | 0.51% | 0.15%   | 0.56% -0.18%  |        |     |     |     |     |     |     |
1000
| Selecton100fd’s       |     |     | 1.0718 0.26%  | 0.20%  | -0.16% | -0.49% | -0.10% -0.01% |     |     |     |     |     |     |     |
| --------------------- | --- | --- | ------------- | ------ | ------ | ------ | ------------- | --- | --- | --- | --- | --- | --- | --- |
| Signalhandlerinstall  |     |     | 0.2538 -1.28% | -1.32% | 0.17%  | -0.33% | 0.11% 0.02%   | 800 |     |     |     |     |     |     |
| Signalhandleroverhead |     |     | 0.8815 -0.90% | -1.53% | 0.12%  | 1.54%  | 0.35% -0.33%  |     |     |     |     |     |     |     |
Vanilla
|     | fork+exit |     | 99.6357 0.83% | 2.49% | -0.49% | -2.82% | -3.44% -2.43% | 600 |     |     |     |     |     |     |
| --- | --------- | --- | ------------- | ----- | ------ | ------ | ------------- | --- | --- | --- | --- | --- | --- | --- |
Cold
|                  | fork+execve    |           | 283.2725 1.51% | 0.23%    | 2.32%  | 1.82%   | -1.76% 3.34%  |     | Hot           |     |     |     |     |          |
| ---------------- | -------------- | --------- | -------------- | -------- | ------ | ------- | ------------- | --- | ------------- | --- | --- | --- | --- | -------- |
|                  | fork+/bin/sh-c |           | 678.1250 2.93% | 2.70%    | 2.35%  | 0.23%   | -1.16% 2.28%  | 400 | Durable(avg.) |     |     |     |     |          |
|                  | UDPlatency     |           | 5.8852 1.25%   | -1.10%   | 0.07%  | -0.73%  | -1.37% -0.32% |     | file          |     |     |     |     |          |
| TCP/IPconnection |                |           | 10.1259 0.13%  | 0.78%    | 0.51%  | -0.01%  | 2.04% 1.62%   | 200 | 64AAs         |     |     |     |     |          |
| AF_UNIXbandwidth |                | 9460.5067 | 0.67%          | -0.56%   | 0.71%  | 0.92%   | -1.85% -1.26% |     | Max Point     |     |     |     |     |          |
|                  | Pipebandwidth  | 4569.4767 | 0.87%          | -1.37%   | -1.03% | 1.94%   | 0.56% -3.02%  | 0   |               |     |     |     |     |          |
|                  |                |           |                |          |        |         |               | 0   |               | 200 | 400 | 600 |     | 800 1000 |
|                  | Phoronix       |           | Vanilla        | Cold Hot |        | Durable | File          |     |               |     |     |     |     |          |
Time Elapsed (s)
| Sockperf(Msgs/sec) |     |     | 739608 -0.04% | -1.73% | -1.30% | 0.75% | 0.63% 0.93% |     |     |     |     |     |     |     |
| ------------------ | --- | --- | ------------- | ------ | ------ | ----- | ----------- | --- | --- | --- | --- | --- | --- | --- |
Figure5:IncreasedmemoryusageofSeaKwhenrunningLMbench.
| OSBench(Ns/Event) |     |     | 78.28 -0.92% | -0.30% | -0.23% | -1.18% | -0.15% -2.23% |     |     |     |     |     |     |     |
| ----------------- | --- | --- | ------------ | ------ | ------ | ------ | ------------- | --- | --- | --- | --- | --- | --- | --- |
7-ZipCompress(MIPS) 29521 -1.31% -0.95% 1.07% 0.60% 1.62% 0.97% Thelinesofallsituationsalmostcompletelyoverlapwitheachother,
| FFmpegLive(FPS)    |     |            | 178.08 0.45% | -1.29% | 1.63%  | 1.57% | 0.86% 0.68%  |             |     |         |          |                |          |        |
| ------------------ | --- | ---------- | ------------ | ------ | ------ | ----- | ------------ | ----------- | --- | ------- | -------- | -------------- | -------- | ------ |
|                    |     |            |              |        |        |       |              | exceptfor64 |     | AAs (in | purple). | Durable (avg.) | averages | memory |
| OpenSSLSHA256(B/s) |     | 1225189783 | 0.28%        | -0.31% | -0.05% | 0.02% | 0.23% -0.08% |             |     |         |          |                |          |        |
usageofcred,sk_filter,andfdtable.
| RedisSET(Reqs/sec)   |     |     | 1932771 1.49% | -1.21% | -3.36% | -0.28% | 0.30% 1.03%  |            |     |             |     |         |            |          |
| -------------------- | --- | --- | ------------- | ------ | ------ | ------ | ------------ | ---------- | --- | ----------- | --- | ------- | ---------- | -------- |
| SQLiteSpeedtest(sec) |     |     | 62.63 0.57%   | 1.64%  | -1.41% | -0.88% | 1.44% -0.41% |            |     |             |     |         |            |          |
| Apache100(Reqs/sec)  |     |     | 48216 -0.63%  | -0.95% | -0.40% | 0.49%  | 0.68% 0.18%  |            |     |             |     |         |            |          |
|                      |     |     |               |        |        |        |              | the memory |     | usage lines | for | “Cold”, | “Hot”, and | ‘Durable |
Table5:PerformanceoverheadofSeaK.Vanillaindicatesthebase-
(avg.)‘almostcompletelyoverlapwithVanilla.Ontheone
line.The“Cold”,“Hot”,and“Durable”columnsarethreerepresenta-
hand,itisbecauseAAoffersthemostgranularlevelofsepa-
tivesituationsexplainedinSection8.2.Especially,threecolumnsin
rationratherthanindiscriminatelyprotecteveryobjectallthe
“Durable”indicatecred,sk_filter,andfdtablefromlefttoright.
time.Therefore,itselfdoesn’timposeexcessivememoryover-
head.Ontheotherhand,nokernelobjecthasalonglifespan
|     | Operations |     | Time(ns) |     | Operations | Time(ns) |     |       |            |                      |     |     |               |      |
| --- | ---------- | --- | -------- | --- | ---------- | -------- | --- | ----- | ---------- | -------------------- | --- | --- | ------------- | ---- |
|     |            |     |          |     |            |          |     | while | also being | frequentlyallocated. |     |     | Therefore,the | mem- |
createregion 13,320.93 idlehandler 1,479.55 oryconsumedbySeaKiseitherquicklyreclaimed,asseenin
|     | allocatefromregion |     | 1,545.04 | allochandler |     | 7,108.49 |     |     |     |     |     |     |     |     |
| --- | ------------------ | --- | -------- | ------------ | --- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
“Hot”and“Cold”,orsufficientlysmalltoremainunnoticeable,
|     | freetoregion |     | 121.03 |     | freehandler | 1,597.02 |     |     |     |     |     |     |     |     |
| --- | ------------ | --- | ------ | --- | ----------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
asin“Durable”.
|     | setguardpage |     | 280.64 |     |     |     |     |     |     |     |     |     |     |     |
| --- | ------------ | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ComparisonwithOtherWorks.BesidesC1,C2,C3,Sec-
Table6:LatenciesofcriticaloperationsinSeaK.Theidlehandler tion 6.1 mentioned several protections from academia and
representsanemptyeBPFprogram;theallochandlerreferstothe commercialproductsthatarealsoforobjectseparation.While
timeineBPFprogramsminusallocationhelperfunction;thefree theperformanceofkalloc_type[30]andAutoSlab[40]can
handleristimeineBPFprogramsminusfreehelperfunction.
notbeevaluatedduetotheirclosed-sourcenature,acompari-
soncanbemadeamongSeaK,xMP[53],slab_virtual[63],
|     |     |     |     |     |     |     |     | and PET | [66], | based | on publicly | available | results | and our |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ----- | ----- | ----------- | --------- | ------- | ------- |
thecreationofdedicatedregions.Therefore,weinsertedrdtsc
measurementinthesamesetting:xMP’saverageoverhead
instructionsbeforeandaftertheseoperationstocollectthe
forisolatingcredusingLMbenchis22.34%(calculatedfrom
| data. | The results | are | presented | in  | Table | 6. As we | can see, |     |     |     |     |     |     |     |
| ----- | ----------- | --- | --------- | --- | ----- | -------- | -------- | --- | --- | --- | --- | --- | --- | --- |
paper’sdata),exceedingSeak’s1%.ThePhoronixresultsof
| all latencies |                | are below | 0.02     | ms, significantly |        | smaller            | than |              |             |          |         |             |              |             |
| ------------- | -------------- | --------- | -------- | ----------------- | ------ | ------------------ | ---- | ------------ | ----------- | -------- | ------- | ----------- | ------------ | ----------- |
|               |                |           |          |                   |        |                    |      | both are     | negligible. | The      | average | performance |              | overhead of |
| the           | minimum        | time      | required | for a             | system | call - 0.1942      | ms   |              |             |          |         |             |              |             |
|               |                |           |          |                   |        |                    |      | slab_virtual |             | is 1.09% | which   | is larger   | than SeaK-64 | case’s      |
| for           | simple syscall |           | in Table | 8.2. Among        |        | all operations,the |      |              |             |          |         |             |              |             |
0.4%.PET’sperformanceoverheadis3%andmemoryover-
| most | time-consuming |     | one | is “create | region”,taking |     | 0.013 |     |     |     |     |     |     |     |
| ---- | -------------- | --- | --- | ---------- | -------------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
headis5.6%(915MB/16GB),asshowninthepaper,while
ms.Itoccursonlywhenadedicatedregioniscreated-the
Seak’soverheadisminimalinboth.
AAperformsallocationforthefirsttime.Suchasmallover-
headisunlikelytoimpactreal-worldapplications-asdoubly
8.3 Scalability&StabilityAnalysis
confirmedinTable8.2.
MemoryOverhead.Theadditionalmemorybroughtinby TodeploySeaKintherealworld,itisessentialtoevaluate
SeaK comprises the guard pages and the unused memory itsscalabilitywhenmultipleAAsareused.Tomeasurethis,
within the page(s) that hold the object. As two separated wegatheredallAAsfromoursecurityanalysisandrandomly
objects can share one guard page, the maximum memory selectedadditionalsecurity-sensitiveobjectsidentifiedin[65],
wastageis8KBforeach16Bobject.However,suchextreme toobtainintotalof64uniqueAAs.Weincreasedthenumber
memoryinflationisveryrare,asourprofilingindicatesthat of installed AAs exponentially,without any specific order,
87.2% objects in the kernel are at least 64 bytes. Figure 5 andpresentedtheaveragedperformanceoverheadinTable7
presentstheincreasedmemoryusageforvarioussituations (More complete results are in Table 8 in Appendix). The
whenrunningLMbench.Unlikeexistingfeatures(Figure2), performanceshowsnonoticeabledegradationasthenumber
| 1182    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

LMbench 2AAs 4AAs 8AAs 16AAs 32AAs 64AAs Inthefuture,wewillreleasenewversionstoleveragethese
featuresspecifictodifferentplatforms.
| Avg.     | -0.32% | 0.0%   | -0.55% | 0.01%  | 0.20% | 0.04% |           |                  |      |             |     |          |
| -------- | ------ | ------ | ------ | ------ | ----- | ----- | --------- | ---------------- | ---- | ----------- | --- | -------- |
| Phoronix | 2AAs   | 4AAs   | 8AAs   | 16AAs  | 32AAs | 64AAs |           |                  |      |             |     |          |
|          |        |        |        |        |       |       | Extending | to stack. Though | SeaK | is designed | as  | a kernel |
| Avg.     | -0.28% | -0.74% | -0.33% | -0.31% | 0.71% | 0.22% |           |                  |      |             |     |          |
secureallocator,itcanbeextendedtopreventstackattack.For
example,wecanattacheBPFprogramstofunctionswhere
Table7:PerformanceofSeaKscalingupto64AAs.Moredetailed
stackoverflowmighthappen,recordingthevalueofthereturn
resultsareinTable8.
addressinBPFmapsatthefunctionentryandexaminingits
integrityatthefunctionexit.Wewillintegratethisextension
intoSeaKinthefuture.
| of AAs grows, | regardless |     | of the | benchmark | used. | This is |     |     |     |     |     |     |
| ------------- | ---------- | --- | ------ | --------- | ----- | ------- | --- | --- | --- | --- | --- | --- |
becauseonesingleAA,nomatterforcoldorhotobjects,has
negligibleoverhead.Regardingmemoryoverhead,Figure5
10 RelatedWorks
showsthatthememoryusageof64AAsisonparwithVanilla,
| with occasional | variances |     | up to | 100MB. | Such intermittent |     |     |     |     |     |     |     |
| --------------- | --------- | --- | ----- | ------ | ----------------- | --- | --- | --- | --- | --- | --- | --- |
Heapsecurityfeaturesfromtheupstreamkernelhavebeen
| overheadis | negligible | formodern |     | OS which | has | access to |     |     |     |     |     |     |
| ---------- | ---------- | --------- | --- | -------- | --- | --------- | --- | --- | --- | --- | --- | --- |
evaluatedinSection3andthosefromacademiaandcommer-
theentirephysicalmemory-oftenmultiplesof4GB-owing
toSDRAMtechnology.Toconclude,SeaKisscalablewhen cialproductshavebeencomparedwithSeaKinSection8.
Theremainingrelatedworksaresecureallocatorinuser
multipleAAsarepresent.
Regardingstability,weenabledall64AAsonthemachine space, which employs safe design principles and random-
izedmechanismstomitigatesheapvulnerabilities.Typically,
| used for daily | research |     | and education | activities, |     | including |     |     |     |     |     |     |
| -------------- | -------- | --- | ------------- | ----------- | --- | --------- | --- | --- | --- | --- | --- | --- |
theseallocatorsseparatein-placemetadatafromtheobject
Overleaf,Outlook,Zoommeetings,ChatGPT,Dockercon-
tainers for CTF challenges, and plugining/unplugining pe- and randomize both the allocation and reuse of memory.
|     |     |     |     |     |     |     | Notable | examples include | DieHard | [19] | and its successor |     |
| --- | --- | --- | --- | --- | --- | --- | ------- | ---------------- | ------- | ---- | ----------------- | --- |
ripherals.Overnearly2.5months,themachinemaintained
consistentstabilitywithoutencounteringanyissues. DieHarder [48],which rely on abundant memory space to
allocatemorememorythanrequiredandrandomizeobjectlo-
cations.FreeGuard[59]enhancesperformancebyintegrating
9 Discussion&FutureDirections thesecurityfeaturesofBIBOP-styleallocatorswiththerapid
allocationcapabilitiesoffreelist-styleallocators.Guarder[57]
eBPFAlternativesandSecurityIssues.SeaKemploysthe refinesFreeGuardbyfine-tuningthelevelofentropy,while
|     |     |     |     |     |     |     | SlimGuard | [45] improves | memory | efficiency | through | fine- |
| --- | --- | --- | --- | --- | --- | --- | --------- | ------------- | ------ | ---------- | ------- | ----- |
eBPFecosystembecauseitissafe,expressive,efficient,and
grainedmemoryclassmanagement.WhileSeaKbenefitsfrom
| only privileged | users | can | install | eBPF programs. |     | Without |     |     |     |     |     |     |
| --------------- | ----- | --- | ------- | -------------- | --- | ------- | --- | --- | --- | --- | --- | --- |
itsuser-spacecounterparts,certainfeaturesarenotapplicable
consideringtheseadvantages,therearealternativesofeBPF
tokernelspace.Forexample,thekerneldoesn’thaveinfinite
ecosystemlikeKprobe.Kprobestoresrawdataratherthan
structuresdatainkernelmemoryandreliesonauserspacepro- memorywhichiscommonlyassumedintheseworks.
Inadditiontogeneral-purposedsecureallocators,thereare
cesstobeinchargeofdatasharingbetweenmodules.Starting
fromKprobe,weneedtoreinventthewheelsalreadyprovided specializedallocatorsfocusingsolelyonmitigatinguse-after-
freevulnerabilities.Cling[50]ensuresthatfreedobjectsare
bytheeBPFecosystem.Thisrequiressignificantengineering
reusedonlyiftheirtypesmatch,identifiedthroughruntime
| efforts and | inevitably | incurs | higher | performance |     | overhead |     |     |     |     |     |     |
| ----------- | ---------- | ------ | ------ | ----------- | --- | -------- | --- | --- | --- | --- | --- | --- |
duetoadditionalkernel-userswitchesfordatasharing. callstackanalysis.MarkUs[17]employsgarbagecollection
principlestofreeobjectswhennodanglingpointersreference
ArecentworkEPF[34]revealsthatattackerscanmisuse
|     |     |     |     |     |     |     | them. Oscar | [62] and FFmalloc |     | [20] operate | under | the as- |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ----------------- | --- | ------------ | ----- | ------- |
BPFcodeasgadgetsforcodereuseattacks.Furthermore,the
sumptionofunlimitedmemoryspace,allocatingobjectsonce
eBPFecosystemwaspreviouslyreportedtocontainvulnera-
|     |     |     |     |     |     |     | and never | reusing them. | Vik [23] | assigns | IDs to | allocated |
| --- | --- | --- | --- | --- | --- | --- | --------- | ------------- | -------- | ------- | ------ | --------- |
bilities[12–14].Thoughaddressingthesesecurityissuesare
orthogonaltoSeaK,itremainsworthwhiletopayattentionto objectsandpermitsonlypointerswithmatchingIDstoac-
cesstheobject,optimizingoverheadthroughARMhardware
thedownsidesofeBPFinthedeploymentofSeaK.
features.Thesespecializedallocatorsaregenerallyunsuitable
HardwareSupport.SeaKreliesonrandomizationandguard
forkernelspacebecausetheirassumptionsdonotapplythere.
pagestoseparateobjectswithindedicatedregions.Whilethe Besides,theyareoverlyspecifictocertainvulnerabilitytype
entropy stands strong at 43,it can be further strengthened whileSeaKgenerallyworks.
throughhardwarefeaturesorhypervisorfeaturesifpresent.
| Some promising                                 |     | memory | protection | features | include | Intel |               |     |     |     |     |     |
| ---------------------------------------------- | --- | ------ | ---------- | -------- | ------- | ----- | ------------- | --- | --- | --- | --- | --- |
| MPK,Armprotectiondomain,andRISC-VDonky[58],and |     |        |            |          |         |       | 11 Conclusion |     |     |     |     |     |
Xen.However,consideringthatmanydevices,especiallyem-
beddedones,lackthesefeatures,wechoosenottointegrate ThisworkpresentsSeaK,thefirstpracticalsecureallocatorin
themintoSeaKatthistimetomaintainitsbroadapplicability. thekernelthatsubstantiallystrengthensheapsecuritywithout
| USENIX Association |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1183 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

introducingnoticeableoverhead.Itisanchoredinanewstrat- [17] Ainsworth,SamandJones,TimothyM. MarkUs:Drop-inuse-
egyofdesigningasecurekernelallocator,centeringaround after-freepreventionforlow-levellanguages. In2020IEEE
SymposiumonSecurityandPrivacy(SP),2020.
the“atomicalleviation”concept.Thisnewstrategyisderived
fromin-depthanalysesofexistingkernelheapsecurityfea- [18] Anonymous Author(s). Raw results ofevaluation. https:
tures andis furthervalidatedby comprehensive evaluation //tinyurl.com/3ytyevpb,October2022.
ofSeaKintermsofsecurityimprovement,performanceand [19] Berger,EmeryD.andZorn,BenjaminG. DieHard:Probabilis-
memoryoverhead,scalability,andstability. ticMemorySafetyforUnsafeLanguages. InProceedingsof
the27thACMSIGPLANConferenceonProgrammingLan-
guageDesignandImplementation(PLDI),2006.
References
[20] BrianWickmanandHongHuandInsuYunandDaeHeeJang
[1] AlexanderPopov’sblog. https://a13xp0p0v.github.io/. andJungWonLimandSanidhyaKashyapandTaesooKim.
PreventingUse-After-FreeAttackswithFastForwardAlloca-
[2] AnalysisandExploitationofaLinuxKernelVulnerability.-
tion. In30thUSENIXSecuritySymposium(USENIXSecurity),
https://perception-point.io/analys
| PerceptionPoint. |     |     |     |     | 2021. |     |     |     |
| ---------------- | --- | --- | --- | --- | ----- | --- | --- | --- |
is-and-exploitation-of-a-linux-kernel-vulnerab
|     |     |     |     |     | [21] YueqiChen,ZhenpengLin,andXinyuXing. |     | ASystematic |     |
| --- | --- | --- | --- | --- | ---------------------------------------- | --- | ----------- | --- |
ility-2/.
InProceed-
StudyofElasticObjectsinKernelExploitation.
| [3] bevx-talk. | https://duasynt.com/slides/bevx-talk.pdf. |     |     |     |     |     |     |     |
| -------------- | ----------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
ingsofthe27thACMSIGSACConferenceonComputerand
[4] CVE-2016-6187:ExploitingLinuxkernelheapoff-by-one- CommunicationsSecurity(CCS),2020.
| VitalyNikolenko. | https://duasynt.com/blog/cve-2016 |     |     |     |                             |                           |     |     |
| ---------------- | --------------------------------- | --- | --- | --- | --------------------------- | ------------------------- | --- | --- |
|                  |                                   |     |     |     | [22] YueqiChenandXinyuXing. | SLAKE:FacilitatingSlabMa- |     |     |
-6187-heap-off-by-one-exploit.
nipulationforExploitingVulnerabilitiesintheLinuxKernel.
InProceedingsofthe26thACMSIGSACConferenceonCom-
[5] eBPFforWindows:MainPage—microsoft.github.io.https:
puterandCommunicationsSecurity(CCS),2019.
| //microsoft.github.io/ebpf-for-windows/. |     |     |     | [Accessed |     |     |     |     |
| ---------------------------------------- | --- | --- | --- | --------- | --- | --- | --- | --- |
06-Feb-2023]. [23] Cho,Haehyun and Park,Jinbum and Oest,Adam and Bao,
TiffanyandWang,RuoyuandShoshitaishvili,YanandDoupé,
| [6] eBPF | Implementation | for | FreeBSD | :: FreeBSD |     |     |     |     |
| -------- | -------------- | --- | ------- | ---------- | --- | --- | --- | --- |
Presentations and Papers — papers.freebsd.org. AdamandAhn,Gail-Joon. ViK:PracticalMitigationofTem-
https://papers.freebsd.org/2018/bsdcan/hayakaw poralMemorySafetyViolationsthroughObjectIDInspection.
InProceedingsofthe27thACMInternationalConferenceon
| a-ebpf_implementation_for_freebsd/. |     |     |     | [Accessed |     |     |     |     |
| ----------------------------------- | --- | --- | --- | --------- | --- | --- | --- | --- |
ArchitecturalSupportforProgrammingLanguagesandOper-
06-Feb-2023].
atingSystems(ASPLOS),2022.
| [7] Lexfo’s | security blog | - CVE-2017-11176: |       | A step-by-   |                                                         |     |                       |     |
| ----------- | ------------- | ----------------- | ----- | ------------ | ------------------------------------------------------- | --- | --------------------- | --- |
|             |               |                   |       |              | [24] clubby789. CVE-2022-0185:ACaseStudy-Ataleondiscov- |     |                       |     |
| step        | Linux Kernel  | exploitation      | (part | 1/4). https: |                                                         |     |                       |     |
|             |               |                   |       |              | eringaLinuxkernelprivesc,2022.                          |     | https://www.hackthebo |     |
//blog.lexfo.fr/cve-2017-11176-linux-kernel-
x.com/blog/CVE-2022-0185:_A_case_study.
exploitation-part1.html.
|     |     |     |     |     | [25] Kees Cook. | security things | in linux v4.13, | 2017. |
| --- | --- | --- | --- | --- | --------------- | --------------- | --------------- | ----- |
https://syzkaller.appspot.com
[8] Upstreambuglifetimes. https://outflux.net/blog/archives/2017/09/05/
/upstream/graph/lifetimes.
security-things-in-linux-v4-13/.
| [9] VitalyNikolenko’sblog. |     | https://duasynt.com/. |     |     |                 |                 |                 |       |
| -------------------------- | --- | --------------------- | --- | --- | --------------- | --------------- | --------------- | ----- |
|                            |     |                       |     |     | [26] Kees Cook. | security things | in linux v4.14, | 2017. |
[10] ZDI-17-240|ZeroDayInitiative. https://www.zerodayin https://outflux.net/blog/archives/2017/11/14/
itiative.com/advisories/ZDI-17-240/. security-things-in-linux-v4-14/.
[11] PhoronixTestSuite,2015. http://www.phoronix-test-s [27] KeesCook. [v4]mm:AddSLUBfreelistpointerobfuscation.
| uite.com/. |     |     |     |     | https://patchwork.kernel.org/project/linux-harde |     |     |     |
| ---------- | --- | --- | --- | --- | ------------------------------------------------ | --- | --- | --- |
ning/patch/20170726041250.GA76741@beast/,2022.
https://cve.m
| [12] CVE | - CVE-2021-4204 | — cve.mitre.org. |     |     |                                                            |     |     |     |
| -------- | --------------- | ---------------- | --- | --- | ---------------------------------------------------------- | --- | --- | --- |
|          |                 |                  |     |     | [28] JakeEdge. Kerneladdressspacelayoutrandomization,2013. |     |     |     |
itre.org/cgi-bin/cvename.cgi?name=CVE-2021-4204,
https://lwn.net/Articles/569635/.
2022.
|          |                 |                  |     |               | [29] Yoann Ghigoff,Julien | Sopena,Kahina                      | Lazri,Antoine | Blin, |
| -------- | --------------- | ---------------- | --- | ------------- | ------------------------- | ---------------------------------- | ------------- | ----- |
| [13] CVE | - CVE-2022-0264 | — cve.mitre.org. |     | https://cve.m |                           |                                    |               |       |
|          |                 |                  |     |               | andGillesMuller.          | BMC:AcceleratingMemcachedusingSafe |               |       |
itre.org/cgi-bin/cvename.cgi?name=CVE-2022-0264,
| 2022. |     |     |     |     | In-kernelCachingandPre-stackProcessing. |     | In18thUSENIX |     |
| ----- | --- | --- | --- | --- | --------------------------------------- | --- | ------------ | --- |
SymposiumonNetworkedSystemsDesignandImplementation
| [14] CVE-CVE-2022-23222—cve.mitre.org. |     |     |     | https://cve.m |     |     |     |     |
| -------------------------------------- | --- | --- | --- | ------------- | --- | --- | --- | --- |
(NSDI),2021.
itre.org/cgi-bin/cvename.cgi?name=CVE-2022-23222,
|     |     |     |     |     | [30] PierreH. https://twitter.com/pedantcoder/status/14 |     |     |     |
| --- | --- | --- | --- | --- | ------------------------------------------------------- | --- | --- | --- |
2022.
70585072361172993?lang=en,2021.
| [15] Syzkaller: | an unsupervised                      | coverage-guided |     | kernel fuzzer, |                    |                     |        |         |
| --------------- | ------------------------------------ | --------------- | --- | -------------- | ------------------ | ------------------- | ------ | ------- |
|                 |                                      |                 |     |                | [31] Yi He,Zhenhua | Zou,Kun Sun,Zhuotao | Liu,Ke | Xu,Qian |
| 2023.           | https://github.com/google/syzkaller. |                 |     |                |                    |                     |        |         |
Wang,ChaoShen,ZhiWang,andQiLi.RapidPatch:Firmware
[16] Rajvardhan Agarwal. CVE-2021-22600, 2022. HotpatchingforReal-TimeEmbeddedDevices.InProceedings
https://github.com/r4j0x00/exploits/tree/mas ofthe31thUSENIXSecuritySymposium(USENIXSecurity),
ter/CVE-2021-22600.
2022.
| 1184    33rd USENIX Security Symposium |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

[32] Hsin-WeiHung,YingtongLiu,andArdalanAmiriSani.Sifter: [49] Sujin Park,Diyu Zhou,Yuchen Qian,Irina Calciu,Taesoo
protectingsecurity-criticalkernelmodulesinandroidthrough Kim,andSanidhyaKashyap. Application-InformedKernel
attacksurfacereduction. InProceedingsofthe28thAnnualIn- SynchronizationPrimitives. In16thUSENIXSymposiumon
ternationalConferenceonMobileComputingAndNetworking, OperatingSystemsDesignandImplementation(OSDI),2022.
2022.
[50] PeriklisAkritidis.Cling:AMemoryAllocatortoMitigateDan-
https:
[33] NurHussein. Randomizingstructurelayout,2017. glingPointers. In19thUSENIXSecuritySymposium(USENIX
| //lwn.net/Articles/722293/. |             |               |     |           |             |      | Security),2010.     |                                       |     |     |     |
| --------------------------- | ----------- | ------------- | --- | --------- | ----------- | ---- | ------------------- | ------------------------------------- | --- | --- | --- |
| [34] Di                     | in,Vaggelis | Atlidakis,and |     | Vasileios | P Kemerlis. | EPF: |                     |                                       |     |     |     |
|                             |             |               |     |           |             |      | [51] JessePolhemus. | Vasileioskemerliswinsannsfcareeraward |     |     |     |
Evilpacketfilter. InUSENIXAnnualTechnicalConference for adaptive hardening, debloating, and hardware-assisted
(USENIXATC23),2023. protection. https://cs.brown.edu/news/2023/03/21/vas
ileios-kemerlis-wins-nsf-career-award-adaptive
| [35] Kostis | Kaffes,Jack | TigarHumphries,David |     |     | Mazières,and |     |     |     |     |     |     |
| ----------- | ----------- | -------------------- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- |
ChristosKozyrakis. Syrup:User-DefinedSchedulingAcross -hardening-debloating-and-hardware-assisted-prot
| theStack. |     | InProceedingsoftheACMSIGOPS28thSympo- |     |     |     |     | ection/. |     |     |     |     |
| --------- | --- | ------------------------------------- | --- | --- | --- | --- | -------- | --- | --- | --- | --- |
siumonOperatingSystemsPrinciples(SOSP),2021.
|     |     |     |     |     |     |     | [52] Alexander | Potapenko. |     | mm: security: | introduce |
| --- | --- | --- | --- | --- | --- | --- | -------------- | ---------- | --- | ------------- | --------- |
[36] kCTF VRP. Kernel Exploits Recipes Notebook. init_on_alloc=1 and init_on_free=1 boot options.
https://docs.google.com/document/d/1a9uUAISBz https://git.kernel.org/pub/scm/linux/kernel/gi
w3ur1aLQqKc5JOQLaJYiOP5pe_B4xCT1KA/,October2022. t/torvalds/linux.git/commit/?id=6471384af2a65306
96fc0203bafe4de41a23c9ef,2022.
| [37] Imran | Khan.     |     | Linux | SLUB      | Allocator | Internals |     |     |     |     |     |
| ---------- | --------- | --- | ----- | --------- | --------- | --------- | --- | --- | --- | --- | --- |
| and        | Debugging | -   | SLUB  | Debugger, | Part      | 2 of 4.   |     |     |     |     |     |
[53] SergejProskurin,MariusMomeu,SeyedhamedGhavamnia,
https://blogs.oracle.com/linux/post/linux-slub
|     |     |     |     |     |     |     | VasileiosP.Kemerlis,andMichalisPolychronakis. |     |     |     | xMP:Se- |
| --- | --- | --- | --- | --- | --- | --- | --------------------------------------------- | --- | --- | --- | ------- |
-allocator-internals-and-debugging-2,2022. lective Memory Protection for Kernel and User Space. In
[38] AndreyKonovalov. LinuxKernelExploitation,2020. https: Proceedingsofthe41stIEEESymposiumonSecurityandPri-
vacy(S&P),2020.
//github.com/xairy/linux-kernel-exploitation.
[39] AzeriaLabs. Groomingtheioskernelheap,2020. https: [54] ShixiongQi,LeslieMonis,ZitengZeng,Ian-chinWang,and
|     |     |     |     |     |     |     | K.K.Ramakrishnan. | SPRIGHT:ExtractingtheServerfrom |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | ------------------------------- | --- | --- | --- |
//azeria-labs.com/grooming-the-ios-kernel-heap/.
[40] ZhenpengLin. HowAUTOSLABChangestheMemoryUn- ServerlessComputing!High-PerformanceEBPF-BasedEvent-
safetyGame.https://grsecurity.net/how_autoslab_ch Driven,Shared-MemoryProcessing. InProceedingsofthe
ACMSIGCOMMConference(SIGCOMM),2022.
anges_the_memory_unsafety_game,August2021.
|                   |     |                     |     |     | https://github.c |     | [55] Xiang                     | (Jenny) Ren,KirkRodrigues,Luyuan |     |                     | Chen,Camilo |
| ----------------- | --- | ------------------- | --- | --- | ---------------- | --- | ------------------------------ | -------------------------------- | --- | ------------------- | ----------- |
| [41] ZhenpengLin. |     | CVE-2021-4154,2022. |     |     |                  |     |                                |                                  |     |                     |             |
|                   |     |                     |     |     |                  |     | Vega,MichaelStumm,andDingYuan. |                                  |     | AnAnalysisofPerfor- |             |
om/Markakd/CVE-2021-4154/blob/master/WRITEUP.md.
|     |     |     |     |     |     |     | manceEvolutionofLinux’sCoreOperations. |     |     |     | InProceedings |
| --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | ------------- |
https://github.c
| [42] ZhenpengLin. |     | CVE-2022-20409,2022. |     |     |     |     |     |     |     |     |     |
| ----------------- | --- | -------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ofthe27thACMSymposiumonOperatingSystemsPrinciples
om/Markakd/bad_io_uring.
(SOSP),2019.
[43] ZhenpengLin,YueqiChen,YuhangWu,DongliangMu,Chen-
|       |          |          |      |     |        |           | [56] Ruia-ruia. | CVE-2022-29582,2022. |     | https://github.com/R |     |
| ----- | -------- | -------- | ---- | --- | ------ | --------- | --------------- | -------------------- | --- | -------------------- | --- |
| sheng | Yu,Xinyu | Xing,and | Kang | Li. | GREBE: | Unveiling |                 |                      |     |                      |     |
uia-ruia/CVE-2022-29582-Exploit.
| ExploitationPotentialforLinuxKernelBugs. |     |     |     |     | InIEEESympo- |     |     |     |     |     |     |
| ---------------------------------------- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- |
siumonSecurityandPrivacy(S&P),2022. [57] SamSilvestroandHongyuLiuandTianyiLiuandZhiqiang
|                                         |     |     |     |     |                 |     | LinandTongpingLiu. |     | Guarder:ATunableSecureAlloca- |     |     |
| --------------------------------------- | --- | --- | --- | --- | --------------- | --- | ------------------ | --- | ----------------------------- | --- | --- |
| [44] ZhenpengLin,YuhangWu,andXinyuXing. |     |     |     |     | DirtyCred:Esca- |     |                    |     |                               |     |     |
tor. In27thUSENIXSecuritySymposium(USENIXSecurity),
| latingPrivilegeinLinuxKernel. |     |     |     | InProceedingsofthe29th |     |     |     |     |     |     |     |
| ----------------------------- | --- | --- | --- | ---------------------- | --- | --- | --- | --- | --- | --- | --- |
2018.
ACMSIGSACConferenceonComputerandCommunications
Security(CCS),2022. [58] David Schrammel,Samuel Weiser,Stefan Steinegger,Mar-
[45] Liu,BeichenandOlivier,PierreandRavindran,Binoy. Slim- tinSchwarzl,MichaelSchwarz,StefanMangard,andDaniel
Guard:ASecureandMemory-EfficientHeapAllocator. In Gruss. Donky:DomainKeys–EfficientIn-ProcessIsolation
|             |     |             |               |     |            |         | forRISC-Vandx86. | In29thUSENIXSecuritySymposium |     |     |     |
| ----------- | --- | ----------- | ------------- | --- | ---------- | ------- | ---------------- | ----------------------------- | --- | --- | --- |
| Proceedings |     | of the 20th | International |     | Middleware | Confer- |                  |                               |     |     |     |
(USENIXSecurity),2020.
ence(Middleware),2019.
[59] Silvestro,SamandLiu,HongyuandCrosser,CoreyandLin,
| [46] Benjamin |     | Livshits, Manu | Sridharan, |     | Yannis Smaragdakis, |     |     |     |     |     |     |
| ------------- | --- | -------------- | ---------- | --- | ------------------- | --- | --- | --- | --- | --- | --- |
ZhiqiangandLiu,Tongping.FreeGuard:AFasterSecureHeap
| Ondˇrej | Lhoták, | J. Nelson | Amaral, | Bor-Yuh | Evan | Chang, |     |     |     |     |     |
| ------- | ------- | --------- | ------- | ------- | ---- | ------ | --- | --- | --- | --- | --- |
InProceedingsoftheACMSIGSACConferenceon
| SamuelZ.Guyer,UdayP.Khedker,AndersMøller,andDim- |     |     |     |     |     |     | Allocator. |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- |
itriosVardoulakis. InDefenseofSoundiness:AManifesto. In ComputerandCommunicationsSecurity(CCS),2017.
CommunicationsoftheACM(CACM),2015.
|     |     |     |     |     |     |     | [60] syzbot. | WARNING:refcount |     | bug in | l2cap_chan_put, |
| --- | --- | --- | --- | --- | --- | --- | ------------ | ---------------- | --- | ------ | --------------- |
[47] LarryMcVoyandCarlStaelin. LMbench-ToosforPerfor- 2020. https://syzkaller.appspot.com/bug?id=39d35c
manceAnalysis,2015. http://lmbench.sourceforge.net 93d0856ca3134bf97f8bb3f249808c2751.
/.
|     |     |     |     |     |     |     | [61] Seyed | Mohammadjavad | Seyed | Talebi, | Zhihao Yao, |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ------------- | ----- | ------- | ----------- |
[48] Novark,Gene and Berger,Emery D. DieHarder: Securing Ardalan Amiri Sani, Zhiyun Qian, and Daniel Austin.
the Heap. In Proceedings ofthe 17thACM Conference on UndoWorkaroundsforKernelBugs. InProceedingsofthe
ComputerandCommunicationsSecurity(CCS),2010. 30thUSENIXSecuritySymposium(USENIXSecurity),2021.
| USENIX Association |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1185 |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

[62] Thurston H.Y. Dang and Petros Maniatis and David Wag- LMBench 2cases 4cases 8cases 16cases 32cases 64cases
ner. Oscar:APracticalPage-Permissions-BasedSchemefor Simplesyscall 0.70% -0.01% -1.52% -1.20% 0.28% 1.43%
ThwartingDanglingPointers. In26thUSENIXSecuritySym- Simpleread 0.06% 0.16% -0.35% 0.16% 0.78% 0.05%
|     |     |     |     |     |     |     | Simplewrite | 0.55% -2.28% | -2.58% | -2.28% | -0.21% 2.44% |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ------------ | ------ | ------ | ------------ |
posium(USENIXSecurity),2017.
|     |     |     |     |     |     |     | Selecton100fd’s | -0.11% -0.04% | 0.11% | 0.00% | -0.36% 0.01% |
| --- | --- | --- | --- | --- | --- | --- | --------------- | ------------- | ----- | ----- | ------------ |
https://github.com/thejh/linux
[63] Torvalds. Slubvirtual. Signalhandlerinstall -0.77% -1.21% -1.55% -1.21% -1.01% -0.39%
/tree/slub-virtual/. Signalhandleroverhead 0.26% -0.34% -1.14% -0.58% 1.55% 3.29%
|     |     |     |     |     |     |     | fork+exit | -2.68% 3.26% | 0.06% | 3.26% | -2.04% -3.30% |
| --- | --- | --- | --- | --- | --- | --- | --------- | ------------ | ----- | ----- | ------------- |
[64] Eduardo Vela. Making Linux Kernel Exploit Cook- fork+execve -1.95% -0.90% -0.50% -0.90% 3.01% -1.99%
ingHarder. https://security.googleblog.com/2022/08 fork+/bin/sh-c -0.39% 0.06% 0.65% 0.06% 1.07% -2.96%
/making-linux-kernel-exploit-cooking.html, August UDPlatency 0.95% 0.17% -0.34% 0.17% -0.23% 0.92%
| 2022.        |       |          |       |      |        |            | TCP/IPconnection | 1.72% 0.64%  | -0.15% | 0.64% | 1.20% 0.10%   |
| ------------ | ----- | -------- | ----- | ---- | ------ | ---------- | ---------------- | ------------ | ------ | ----- | ------------- |
|              |       |          |       |      |        |            | AF_UNIXbandwidth | -1.09% 0.13% | 0.26%  | 0.13% | -1.54% -0.16% |
| [65] Ruipeng | Wang, | Kaixiang | Chen, | Chao | Zhang, | Zulie Pan, |                  |              |        |       |               |
|              |       |          |       |      |        |            | Pipebandwidth    | -1.45% 1.00% | -0.16% | 1.89% | 0.13% 0.04%   |
QianyuLi,SiliangQin,ShenglinXu,MinZhang,andYang
|     |     |     |     |     |     |     | Avg. | -0.32% 0.05% | -0.55% | 0.01% | 0.20% 0.04% |
| --- | --- | --- | --- | --- | --- | --- | ---- | ------------ | ------ | ----- | ----------- |
Li. AlphaEXP: An expertsystem foridentifying Security- Phoronix 2cases 4cases 8cases 16cases 32cases 64cases
Sensitivekernelobjects.In32ndUSENIXSecuritySymposium Sockperf(Msgs/sec) 0.48% -1.33% -1.65% -1.30% 4.20% 3.75%
(USENIXSecurity),2023. OSBench(Ns/Event) -0.24% -0.16% -0.19% -0.23% 1.45% 0.45%
|                                                     |     |     |     |     |             |     | 7-ZipCompress(MIPS) | -1.88% -1.22% | -0.50% | 1.07%  | -0.29% 0.41% |
| --------------------------------------------------- | --- | --- | --- | --- | ----------- | --- | ------------------- | ------------- | ------ | ------ | ------------ |
| [66] ZichengWang,YueqiChen,andQingkaiZeng.          |     |     |     |     | PET:Prevent |     |                     |               |        |        |              |
|                                                     |     |     |     |     |             |     | FFmpegLive(FPS)     | 0.48% -0.83%  | -0.34% | 1.63%  | 1.97% 0.87%  |
| discoverederrorsfrombeingtriggeredinthelinuxkernel. |     |     |     |     |             | In  |                     |               |        |        |              |
|                                                     |     |     |     |     |             |     | OpenSSLSHA256(B/s)  | -0.10% -0.16% | -0.09% | -0.05% | -0.07% 0.04% |
32ndUSENIXSecuritySymposium(USENIXSecurity),2023.
|     |     |     |     |     |     |     | RedisSET(Reqs/sec)   | 0.94% -3.30% | -3.06% | -3.36% | -1.06% -2.99% |
| --- | --- | --- | --- | --- | --- | --- | -------------------- | ------------ | ------ | ------ | ------------- |
|     |     |     |     |     |     |     | SQLiteSpeedtest(sec) | 0.37% -0.31% | 0.57%  | 1.41%  | 0.00% 0.15%   |
[67] WeiWu,YueqiChen,JunXu,XinyuXing,WeiZou,andXi-
|     |     |     |     |     |     |     | Apache100(Reqs/sec) | -0.30% -0.52% | -0.71% | -0.40% | -0.55% -0.85% |
| --- | --- | --- | --- | --- | --- | --- | ------------------- | ------------- | ------ | ------ | ------------- |
aoruiGong. FUZE:TowardsFacilitatingExploitGeneration Avg. -0.28% -0.74% -0.33% -0.31% 0.71% 0.22%
| for Kernel | Use-After-Free |     | Vulnerabilities. |     | In Proceedings |     |     |     |     |     |     |
| ---------- | -------------- | --- | ---------------- | --- | -------------- | --- | --- | --- | --- | --- | --- |
ofthe27thUSENIXSecuritySymposium(USENIXSecurity), Table8:CompleteresultsfortheperformanceofSeaKscalingup
2018.
to64AAs.CorrespondingsampledresultsareinTable7.
[68] WenXu,JuanruLi,JunliangShu,WenboYang,TianyiXie,
| YuanyuanZhang,andDawuGu. |     |     |     | FromCollisionToExploita- |     |     |     |     |     |     |     |
| ------------------------ | --- | --- | --- | ------------------------ | --- | --- | --- | --- | --- | --- | --- |
tion:UnleashingUse-After-FreeVulnerabilitiesinLinuxKer-
A AdditionalResults
nel. InProceedingsofthe22ndACMSIGSACConferenceon
ComputerandCommunicationsSecurity(CCS),2015.
Duetothespacelimit,weonlypresentsampledresultsinthe
| [69] KyleZeng. | CVE-2022-1786,2022. |     |     | https://blog.kylebot |     |     |     |     |     |     |     |
| -------------- | ------------------- | --- | --- | -------------------- | --- | --- | --- | --- | --- | --- | --- |
text.InGoogleSheet[18],wepresentmoreresultsregarding
.net/2022/10/16/CVE-2022-1786.
memoryusageofexistingfeaturesandSeaKwhenrunning
| [70] KyleZeng,Yueqi |     | Chen,Haehyun |     | Cho,Xinyu | Xing,Adam |     |     |     |     |     |     |
| ------------------- | --- | ------------ | --- | --------- | --------- | --- | --- | --- | --- | --- | --- |
Phoronix.Inaddition,Table8presentsmorecompleteresults
| Doupé,YanShoshitaishvili,andTiffanyBao. |     |     |     |     | PlayingforK |     |     |     |     |     |     |
| --------------------------------------- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- |
regardingSeaK’sscalabilityinperformance.Table9shows
(H)eaps:UnderstandingandImprovingLinuxKernelExploit
theFNandFPresultsofstaticanalysisforseparatingvulner-
| Reliability. | InProceedingsofthe31stUSENIXSecuritySym- |     |     |     |     |     |                    |                                  |     |     |     |
| ------------ | ---------------------------------------- | --- | --- | --- | --- | --- | ------------------ | -------------------------------- | --- | --- | --- |
|              |                                          |     |     |     |     |     | abilitycorruption. | List2showsanexampleofsynthesized |     |     |     |
posium(USENIXSecurity),2022.
eBPFprogram.
[71] YuhongZhong,HaoyuLi,YuJianWu,IoannisZarkadas,Jef-
| freyTao,Evan            |     | Mesterhazy,MichaelMakris,Junfeng |     |                           |     | Yang, |     |     |     |     |     |
| ----------------------- | --- | -------------------------------- | --- | ------------------------- | --- | ----- | --- | --- | --- | --- | --- |
| AmyTai,andRyanStutsman. |     |                                  |     | XRP:In-KernelStorageFunc- |     |       |     |     |     |     |     |
| tionswitheBPF.          |     | In16thUSENIXSymposiumonOperating |     |                           |     |       |     |     |     |     |     |
SystemsDesignandImplementation(OSDI),2022.
[72] YangZhou,ZezhouWang,SowmyaDharanipragada,andMin-
| lanYu. | Electrode:                              | AcceleratingDistributedProtocolswith |     |     |     |     |     |     |     |     |     |
| ------ | --------------------------------------- | ------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| eBPF.  | In20thUSENIXSymposiumonNetworkedSystems |                                      |     |     |     |     |     |     |     |     |     |
DesignandImplementation(NSDI),2023.
| [73] XiaochenZou. |     | CVE-2022-27666,2022. |     |     | https://github.c |     |     |     |     |     |     |
| ----------------- | --- | -------------------- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- |
om/plummm/CVE-2022-27666.
| 1186    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

| SYZTitle                                    | C1 C2 | C3 TypeofIdentifiedVulnerableObject |     |     | SeaK |
| ------------------------------------------- | ----- | ----------------------------------- | --- | --- | ---- |
| BUG__corrupted_list_in_kobject_add_internal | /     | hci_conn                            |     |     |      |
KASAN__use-after-free_Read_in_sctp_auth_free (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) crypto_shash,sctp_endpoint (cid:32)
KASAN__use-after-free_Write_in__sco_sock_close (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) hci_conn (cid:32)
KMSAN__uninit-value_in_geneve_xmit (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) sk_buff,net_device (cid:32)
WARNING__refcount_bug_in_l2cap_chan_put (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) l2cap_chan (cid:32)
KASAN__slab-out-of-bounds_Read_in_tcf_exts_destroy (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) tc_action,tcindex_filter_result (cid:32)
KASAN__use-after-free_Read_in_rdma_listen (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) rdma_id_private (cid:32)
BUG__corrupted_list_in_nft_obj_del (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) nft_object (cid:32)
BUG__corrupted_list_in_nf_tables_commit (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) nft_flowtable,nft_trans (cid:32)
general_protection_fault_in_delayed_uprobe_remove (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) delayed_uprobe (cid:32)
KASAN__use-after-free_Read_in_delayed_uprobe_remove (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) delayed_uprobe (cid:32)
KASAN__use-after-free_Read_in_x25_device_event (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) (cid:32)
net_device,x25_neigh
|     | (cid:35) (cid:35)/(cid:35)(cid:72) | (cid:32) |     |     | (cid:32) |
| --- | ---------------------------------- | -------- | --- | --- | -------- |
WARNING__ODEBUG_bug_in_tcf_queue_work workqueue_struct,route4_filter
KASAN__use-after-free_Read_in_route4_get (cid:35) (cid:35)(cid:35)(cid:72) / (cid:32) Qdisc,route4_bucket,route4_head,sk_buff,tcf_block,tcf_chain, (cid:32)
tcf_proto,route4_filter
|                                      | (cid:35) (cid:35)(cid:35)(cid:72) | (cid:32)      |     |     | (cid:32) |
| ------------------------------------ | --------------------------------- | ------------- | --- | --- | -------- |
| WARNING__ODEBUG_bug_in_route4_change | /                                 | route4_filter |     |     |          |
WARNING_in_call_rcu (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) route4_filter (cid:32)
|                                                             | (cid:35) (cid:35)(cid:35)(cid:72) | (cid:32) snd_pcm_oss_file, | snd_pcm_plugin, | snd_pcm_runtime, | (cid:32) |
| ----------------------------------------------------------- | --------------------------------- | -------------------------- | --------------- | ---------------- | -------- |
| KASAN__slab-out-of-bounds_Write_in_default_read_copy_kernel | /                                 |                            |                 |                  |          |
snd_pcm_substream,snd_pcm_plugin_channel
|                                                             | (cid:35) (cid:35)(cid:35)(cid:72) | (cid:32)          |                 |                  | (cid:32) |
| ----------------------------------------------------------- | --------------------------------- | ----------------- | --------------- | ---------------- | -------- |
|                                                             |                                   | snd_pcm_oss_file, | snd_pcm_plugin, | snd_pcm_runtime, |          |
| KASAN__slab-out-of-bounds_Read_in_default_write_copy_kernel | /                                 |                   |                 |                  |          |
snd_pcm_substream,snd_pcm_plugin_channel
|                                      | (cid:35) (cid:35)(cid:35)(cid:72) | (cid:32)                |     |     | (cid:32) |
| ------------------------------------ | --------------------------------- | ----------------------- | --- | --- | -------- |
| general_protection_fault_in_vb2_mmap | /                                 | video_device,vb2_buffer |     |     |          |
KASAN__use-after-free_Read_in_vb2_mmap (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) video_device,vb2_buffer (cid:32)
KASAN__use-after-free_Read_in__list_add_valid (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) sockaddr,ucma_context,ucma_file,rdma_id_private (cid:32)
KASAN__null-ptr-deref_Read_in_refcount_sub_and_test_checked (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) vb2_vmalloc_buf (cid:32)
KASAN__use-after-free_Write_in__ext4_expand_extra_isize (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) ext4_inode_info,ext4_sb_info,inode (cid:32)
KASAN__use-after-free_Read_in__nf_tables_abort (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) list_head,net,nft_trans,sk_buff,sock,nft_table (cid:32)
BUG__corrupted_list_in__nf_tables_abort (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) list_head,nft_flowtable,nft_rule,nft_set,nft_table (cid:32)
|                              | (cid:35) (cid:35)(cid:35)(cid:72) | (cid:32) pde_opener, proc_dir_entry, | seq_file, | snd_info_private_data, | (cid:32) |
| ---------------------------- | --------------------------------- | ------------------------------------ | --------- | ---------------------- | -------- |
| WARNING_in_snd_info_get_line | /                                 |                                      |           |                        |          |
snd_info_buffer
|                                                | (cid:35) (cid:35)(cid:35)(cid:72) | (cid:32)             |     |     | (cid:32) |
| ---------------------------------------------- | --------------------------------- | -------------------- | --- | --- | -------- |
| KASAN__slab-out-of-bounds_Write_in_decode_data | /                                 | tty_ldisc,tty_struct |     |     |          |
KASAN__use-after-free_Read_in_ip6_hold_safe (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) dst_entry (cid:32)
KASAN__use-after-free_Write_in_ip6_hold_safe (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) dst_entry (cid:32)
KASAN__use-after-free_Read_in_l2cap_chan_close (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) l2cap_chan (cid:32)
KASAN__slab-out-of-bounds_Read_in_cap_inode_getsecurity (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) vfs_cap_data (cid:32)
kernel_BUG_at_fs/userfaultfd.c (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) userfaultfd_ctx (cid:32)
KASAN__use-after-free_Read_in_handle_userfault (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) mm_struct,vm_area_struct,userfaultfd_ctx (cid:32)
KASAN__use-after-free_Read_in__rhashtable_lookup (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) rhashtable (cid:32)
KASAN__use-after-free_Read_in_hci_send_acl (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) hci_conn,hci_dev,l2cap_conn,work_struct,hci_chan (cid:32)
WARNING_in_snd_usbmidi_submit_urb/usb_submit_urb (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) urb (cid:32)
UBSAN__shift-out-of-bounds_in_dummy_hub_control (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) urb,usb_ctrlrequest,usb_device,usb_hcd (cid:32)
KASAN__use-after-free_Read_in_tcp_check_sack_reordering (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) sk_buff (cid:32)
KASAN__use-after-free_Read_in__vb2_perform_fileio (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) vb2_fileio_data,video_device (cid:32)
KASAN__slab-out-of-bounds_Read_in_bitmap_ip_add (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) bitmap_ip->members (cid:32)
KASAN__slab-out-of-bounds_Read_in_bitmap_ip_ext_cleanup (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) net,nlattr,sk_buff,sock,bitmap_ip->members (cid:32)
KASAN__slab-out-of-bounds_Write_in_bitmap_ip_del (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) bitmap_ip->members (cid:32)
KASAN__use-after-free_Read_in__queue_work (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) atomic64_t,work_struct (cid:32)
KASAN__use-after-free_Read_in_ep_scan_ready_list (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) (cid:32)
atomic64_t,work_struct
|     | (cid:35) (cid:35)/(cid:35)(cid:72) | (cid:32) |     |     | (cid:32) |
| --- | ---------------------------------- | -------- | --- | --- | -------- |
KASAN__use-after-free_Read_in_p9_fd_poll list_head,p9_client,p9_trans_fd
WARNING__ODEBUG_bug_in_p9_fd_close (cid:35) (cid:35)/(cid:35)(cid:72) (cid:32) p9_client,p9_trans_fd (cid:32)
|     | (cid:35) (cid:35)(cid:35)(cid:72) | (cid:32) |     |     | (cid:32) |
| --- | --------------------------------- | -------- | --- | --- | -------- |
Table 9: MoreresultsofSeaK’ssecurityimprovementforseparatingvulnerabilitycorruption. Structuresinboldin“TypeofIdentified
VulnerableObject”aregroundtruth.TheremainingisFP.SeaKhasnoFNinalltestcases. indicatesfailingtopreventexploitation, stands
forworkingoccasionallyduetothesamplingnature, meanssucceedinginpreventingex(cid:35)ploitation.
(cid:72)(cid:35)
(cid:32)
| USENIX Association |     |     | 33rd USENIX Security Symposium    1187 |     |     |
| ------------------ | --- | --- | -------------------------------------- | --- | --- |

| 1 // essential |     | utilities |     |            |     |             |          |
| -------------- | --- | --------- | --- | ---------- | --- | ----------- | -------- |
| 2 // define    | a   | map where | an  | alloc_addr |     | corresponds | to a key |
| 3 struct       | {   |           |     |            |     |             |          |
4 ...
| 5 } addr2key | SEC(".maps"); |           |     |                 |     |      |       |
| ------------ | ------------- | --------- | --- | --------------- | --- | ---- | ----- |
| 6 // define  | a             | map where | a   | key corresponds |     | to a | cache |
| 7 struct     | {             |           |     |                 |     |      |       |
8 ...
| 9 } key2cache               | SEC(".maps");                        |                                   |                         |                |                     |           |             |
| --------------------------- | ------------------------------------ | --------------------------------- | ----------------------- | -------------- | ------------------- | --------- | ----------- |
| 10 int alloc_handler(struct |                                      |                                   | pt_regs*                | ctx,           |                     |           |             |
| 11 u64                      | kpi_type)                            | {                                 |                         |                |                     |           |             |
| 12                          | // one                               | key is                            | related                 | to             | one kind            | of object |             |
| 13                          | key =                                | generate_key(ctx);                |                         |                |                     |           |             |
| 14                          | //for                                | kmem_cache_alloc,                 |                         |                |                     |           |             |
| 15                          | //we                                 | should                            | get the                 | kmem_cache     |                     | first.    |             |
| 16                          | //Then                               | we can                            | read                    | the alloc_size |                     | through   | kmem_cache. |
| 17                          | if(kpi_type                          | ==                                | kmem_cache_alloc){      |                |                     |           |             |
| 18                          | cache                                | = (struct                         | kmem_cache*)            |                | PT_REGS_PARM1(ctx); |           |             |
| 19                          | alloc_size                           |                                   | = BPF_CORE_READ(cache,  |                |                     | size);    |             |
| 20                          | alloc_size                           |                                   | = get_size(alloc_size); |                |                     |           |             |
| 21                          | }else                                | if(kpi_type                       | ==                      | kmalloc){      |                     |           |             |
| 22                          | alloc_size                           |                                   | = PT_REGS_PARM1(ctx);   |                |                     |           |             |
| 23                          | alloc_size                           |                                   | = get_size(alloc_size); |                |                     |           |             |
| 24                          | }                                    |                                   |                         |                |                     |           |             |
| 25                          | //use                                | key to                            | find                    | the related    |                     | cache in  | key2cache   |
| 26                          | cache                                | = bpf_look_up_cache(key2cache,    |                         |                |                     | key)      |             |
| 27                          | if(!cache){                          |                                   |                         |                |                     |           |             |
| 28                          | //create                             |                                   | a cache                 | if first       | allocted            |           |             |
| 29                          | cache                                | = generate_cache(key,alloc_size); |                         |                |                     |           |             |
| 30                          | }                                    |                                   |                         |                |                     |           |             |
| 31                          | //allocate                           | an                                | object                  |                |                     |           |             |
| 32                          | alloc_addr                           | = bpf_cache_alloc(cache)          |                         |                |                     |           |             |
| 33                          | update_map(addr2key,alloc_addr,key); |                                   |                         |                |                     |           |             |
34 }
| 35 int free_handler(struct |                                          |                                      | pt_regs*              | ctx,          |            |     |     |
| -------------------------- | ---------------------------------------- | ------------------------------------ | --------------------- | ------------- | ---------- | --- | --- |
| 36 u64                     | kpi_type)                                | {                                    |                       |               |            |     |     |
| 37 //for                   | kmem_cache_free                          |                                      |                       |               |            |     |     |
| 38 //the                   | second                                   | parameter                            |                       | is alloc_addr |            |     |     |
| 39                         | if(kpi_type                              | ==                                   | kmem_cache_free){     |               |            |     |     |
| 40                         | alloc_addr                               |                                      | = PT_REGS_PARM2(ctx); |               |            |     |     |
| 41                         | }else                                    | if (kpi_type                         | ==                    | kfree){       |            |     |     |
| 42                         | alloc_addr                               |                                      | = PT_REGS_PARM1(ctx); |               |            |     |     |
| 43                         | }                                        |                                      |                       |               |            |     |     |
| 44                         | key =                                    | bpf_look_up_key(addr2key,alloc_addr) |                       |               |            |     |     |
| 45                         | //locate                                 | the                                  | object’s              | cache         |            |     |     |
| 46                         | cache                                    | = bpf_look_up_cache(key2cache,key)   |                       |               |            |     |     |
| 47                         | alloc_size                               | = BPF_CORE_READ(cache,size)          |                       |               |            |     |     |
| 48                         | // get                                   | the alloc_size                       |                       | of            | the object |     |     |
| 49                         | bpf_delete_object(alloc_size,alloc_addr) |                                      |                       |               |            |     |     |
50 }
| 51 // attach | to  | allocation |     | site |     |     |     |
| ------------ | --- | ---------- | --- | ---- | --- | --- | --- |
52 SEC("kprobe/single_open+0x2a")
| 53 int probe_alloc_seq |                    | (struct | pt_regs* |                    | ctx) { |     |     |
| ---------------------- | ------------------ | ------- | -------- | ------------------ | ------ | --- | --- |
| 54 return              | alloc_handler(ctx, |         |          | kmem_cache_alloc); |        |     |     |
55 }
| 56 // attach | to  | free | site |     |     |     |     |
| ------------ | --- | ---- | ---- | --- | --- | --- | --- |
57 SEC("kprobe/single_release+0x34")
| 58 int probe_free_seq |                   | (struct | pt_regs* |         | ctx) { |     |     |
| --------------------- | ----------------- | ------- | -------- | ------- | ------ | --- | --- |
| 59 return             | free_handler(ctx, |         |          | kfree); |        |     |     |
60 }
| Listing 2: | A simplifiedexample |     |     | ofa | synthesizedebpfpro- |     |     |
| ---------- | ------------------- | --- | --- | --- | ------------------- | --- | --- |
gram.Theto-be-protectedobjectisseq_operationswhichis
allocatedviakmem_cache_allocinsingle_open()andfreedvia
kfreeinsingle_release.Notethatcommentsinthecodeare
notsynthesizedbutforillustrationpurpose.
1188    33rd USENIX Security Symposium USENIX Association
