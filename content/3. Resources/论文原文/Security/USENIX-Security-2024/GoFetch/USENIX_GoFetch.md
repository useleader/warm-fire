---
publish: true
---

GoFetch: Breaking Constant-Time
Cryptographic Implementations Using
Data Memory-Dependent Prefetchers
Boru Chen, University of Illinois Urbana-Champaign; Yingchen Wang,
University of Texas at Austin; Pradyumna Shome, Georgia Institute of Technology;
Christopher Fletcher, University of California, Berkeley; David Kohlbrenner,
University of Washington; Riccardo Paccagnella, Carnegie Mellon University;
Daniel Genkin, Georgia Institute of Technology
https://www.usenix.org/conference/usenixsecurity24/presentation/chen-boru
This paper is included in the Proceedings of the
33rd USENIX Security Symposium.
August 14–16, 2024 • Philadelphia, PA, USA
978-1-939133-44-1
Open access to the Proceedings of the
33rd USENIX Security Symposium
is sponsored by USENIX.

GoFetch: Breaking Constant-Time Cryptographic Implementations
|     |          |                        | Using        | Data     | Memory-Dependent |                          |                     |     | Prefetchers |                       |              |     |     |     |
| --- | -------- | ---------------------- | ------------ | -------- | ---------------- | ------------------------ | ------------------- | --- | ----------- | --------------------- | ------------ | --- | --- | --- |
|     | BoruChen |                        | YingchenWang |          |                  |                          | PradyumnaShome      |     |             | ChristopherW.Fletcher |              |     |     |     |
|     |          | UIUC                   |              | UTAustin |                  |                          | GeorgiaTech         |     |             |                       | UCBerkeley   |     |     |     |
|     |          | DavidKohlbrenner       |              |          |                  |                          | RiccardoPaccagnella |     |             |                       | DanielGenkin |     |     |     |
|     |          | UniversityofWashington |              |          |                  | CarnegieMellonUniversity |                     |     |             |                       | GeorgiaTech  |     |     |     |
Abstract theprogram’sinstructionmemoryaccesspattern.Thishasled
tothedevelopmentofawiderangeofdefenses—including
Microarchitecturalside-channelattackshaveshakenthefoun-
|     |     |     |     |     |     |     |     | the ubiquitous |     | constant-time | programming |     | model | [53,62], |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | ------------- | ----------- | --- | ----- | -------- |
dationsofmodernprocessordesign.Thecornerstonedefense
informationflow-basedtracking[42,80,95],andmore—all
againsttheseattackshasbeentoensurethatsecurity-critical
ofwhichseektopreventsecretdatafrombeingusedasan
programsdonotusesecret-dependentdataasaddresses.Put
addresstomemory/control-flowinstructions.
simply:donotpasssecretsasaddressesto,e.g.,datamemory
Recently,however,Augury[84]demonstratedthatApple
| instructions. |     | Yet,the discoveryofdata |     | memory-dependent |     |     |     |     |     |     |     |     |     |     |
| ------------- | --- | ----------------------- | --- | ---------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
M-seriesCPUsunderminethisprogrammingmodelbyintro-
prefetchers(DMPs)—whichturnprogramdataintoaddresses
ducingaDataMemory-dependentPrefetcher(DMP)thatwill
directlyfromwithinthememorysystem—callsintoquestion
|     |     |     |     |     |     |     |     | attempt | to prefetch | addresses | found | in  | the contents | of pro- |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ----------- | --------- | ----- | --- | ------------ | ------- |
whetherthisapproachwillcontinuetoremainsecure.
grammemory.Thus,intheory,Apple’sDMPleaksmemory
| This | paper | shows that | the security | threat | from | DMPs | is  |     |     |     |     |     |     |     |
| ---- | ----- | ---------- | ------------ | ------ | ---- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
contentsviacachesidechannels,evenifthatmemoryisnever
significantlyworsethanpreviouslythoughtanddemonstrates
passedasanaddresstoamemory/control-flowinstruction.
thefirstend-to-endattacksonsecurity-criticalsoftwareusing
|     |     |     |     |     |     |     |     | Despite | the Apple | DMP’s | novelleakage |     | capabilities,its |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | --------- | ----- | ------------ | --- | ---------------- | --- |
theApplem-seriesDMP.Undergirdingourattacksisanew
restrictivebehaviorhaspreventeditfrombeingusedinattacks.
| understanding |     | of how DMPs | behave | which | shows,among |     |     |     |     |     |     |     |     |     |
| ------------- | --- | ----------- | ------ | ----- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Inparticular,AuguryreportedthattheDMPonlyactivates
otherthings,thattheAppleDMPwillactivateonbehalfof
|     |     |     |     |     |     |     |     | in the presence |     | of a rather | idiosyncratic |     | program | memory |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------- | --- | ----------- | ------------- | --- | ------- | ------ |
anyvictimprogramandattemptto“leak”anycacheddata
accesspattern(wheretheprogramstreamsthroughanarray
| that resembles |     | a pointer. | From this | understanding, |     | we  | de- |     |     |     |     |     |     |     |
| -------------- | --- | ---------- | --------- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ofpointersandarchitecturallydereferencesthosepointers).
signanewtypeofchosen-inputattackthatusestheDMPto
Thisaccesspatternisnottypicallyfoundinsecuritycritical
performend-to-endkeyextractiononpopularconstant-time
softwaresuchasside-channelhardenedconstant-timecode—
implementationsofclassical(OpenSSLDiffie-HellmanKey
hencemakingthatcodeimpervioustoleakagethroughthe
Exchange,GoRSAdecryption)andpost-quantumcryptogra-
DMP.WiththeDMP’sfullsecurityimplicationsunclear,in
phy(CRYSTALS-KyberandCRYSTALS-Dilithium).
thispaperweaddressthefollowingquestions:
|     |     |     |     |     |     |     |     | Do DMPs | create | a critical | security | threat | to  | high-value |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ------ | ---------- | -------- | ------ | --- | ---------- |
1 Introduction
|     |     |     |     |     |     |     |     | software? | Can | attacks | use DMPs | to bypass | side-channel |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | ------- | -------- | --------- | ------------ | --- |
countermeasuressuchasconstant-timeprogramming?
Foroveradecade,modernprocessorshavefacedamyriad
ofmicroarchitecturalside-channelattacks,e.g.,throughthe
|     |     |     |     |     |     |     |     | 1.1 OurContribution |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------- | --- | --- | --- | --- | --- | --- |
caches[64,92],TLBs[43,79,83],branchpredictors[6,36],
on-chip interconnects [32,65,86], memory management This paper answers the above questions in the affirmative,
units [44,51,82], speculative execution [52,55], voltage- showing how Apple’s DMP implementation poses severe
frequencyscaling[78,88,89]andmore. riskstotheconstant-timecodingparadigm.Inparticular,we
The most prominent class of these attacks occurs when demonstrate end-to-endkeyextraction attacksagainstfour
theprogram’smemoryaccesspatternbecomesdependenton state-of-the-artcryptographicimplementations,alldeploying
secretdata.Forexample,cacheandTLBside-channelattacks constant-timeprogramming.
arise when the program’s data memory access pattern be- Analyzing DMP Activation Patterns. We start by re-
comessecretdependent.Otherattacks,e.g.,thosemonitoring examining the findings in Augury [84], here we find that
on-chipinterconnects,canbeviewedsimilarlywithrespectto Augury’sanalysisoftheDMPactivationmodelwasoverly
| USENIX Association |     |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1117 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

restrictive and missed several DMP activation scenarios. theirthreatmodel.TheGoCryptoteamconsidersthisattack
Throughnewreverseengineering,wefindthattheDMPac- tobelowseverity.TheCRYSTALSteamagreedthatpinning
tivates on behalf of potentially any program,and attempts totheIcestormcoreswithoutDMPcouldbetheshort-term
todereferenceanydatabroughtintocachethatresemblesa solutionandhardwarefixesareneededinthelongterm.
pointer.Thisbehaviorplacesasignificantamountofprogram
dataatrisk,andeliminatestherestrictionsreportedbyprior
2 Background
work.Finally,goingbeyondAppleweconfirmtheexistence
of a similarDMP on Intel’s latest 13th generation (Raptor
Lake)architecturewithmorerestrictiveactivationcriteria. CacheArchitecture. Modernprocessorsuseahierarchy
BreakingConstantTimeCryptography. Next,weshow ofcachestoreducememoryaccesslatency.Typically,higher-
howtoexploittheDMPtobreaksecurity-criticalsoftware. levelcachesaresmallerandfastertoaccess,whilelower-level
Wedemonstratethewidespreadpresenceofcodevulnerable cachesarelargerbutslowertoaccess.Forexample,theApple
toDMP-aidedattacksinstate-of-the-artconstant-timecryp- processorswestudyinthispaperhavetwocachelevels,acore-
tographicsoftware,spanningclassicaltopost-quantumkey privateL1andasharedL2.Thesecachesareset-associative,
exchangeandsigningalgorithms.Ourkeyinsightisthatwhile meaningthattheycontainafixednumberofcachesets,each
theDMPonlydereferencespointers,anattackercancraftpro- ofwhichcanfitafixednumberofcachelines.Cachelines
graminputssothatwhenthoseinputsmixwithcryptographic arethebasicunitforcachetransactions.Multi-levelcaches
secrets, the resulting intermediate state can be engineered haveaninclusionpolicythatdetermineshowthepresenceof
to look like a pointer if and only if the secret satisfies an acachelineinonelevelaffectsitspresenceinotherlevels.
attacker-chosenpredicate.Forexample,imaginethatapro- MostofourexperimentswereconductedontheAppleM1’s
gram has secret s,takes x as input and computes and then 4Firestorm(performance)cores,whicharetheonlyonesto
storesy=s⊕xtoitsprogrammemory.Theattackercancraft haveaDMP.EachFirestormcorehasa128KByte,8wayset-
differentxandinferpartial(orevencomplete)information associativeL1datacachewith64Bytecachelinesandthese
aboutsbyobservingwhethertheDMPisabletodereference 4Firestormcoressharea12MByte,12wayset-associative
y.Wefirstusethisobservationtobreaktheguaranteesofa L2datacachewith128Bytecachelines.ThesharedL2cache
standardconstant-timeswapprimitive[54]recommendedfor isinclusiveoftheL1caches,i.e.everycachelinepresentin
useincryptographicimplementations.Wethenshowhowto theL1isalsopresentintheL2[94].
breakcompletecryptographicimplementationsdesignedto CacheSide-ChannelAttacks. Inacacheside-channelat-
besecureagainstchosen-inputattacks. tack,anattackerinfersavictimprogram’ssecretbyobserving
SummaryofContribution. Wecontributethefollowing. thesideeffectsofthevictimprogram’ssecret-dependentac-
1. Reverse Engineering Apple and Intel DMPs. We cessestotheprocessorcache.Theseattackstypicallyconsist
reverseengineertheDMPfoundonAppleCPUsanddis- ofthreesteps,duringwhichtheattacker(i)bringsthecache
covernewactivationcriteria(Section4). intoaknownstate,(ii)letsthevictimexecute,and(iii)checks
2. DevelopingDMPExploitationTechniques. Usingour thestateofthecachetolearninformationaboutthevictim’s
newunderstandingoftheDMP,wedevelopanewtypeof executionduringstep(ii).Twotechniquescommonlyusedto
victim-agnosticchosen-inputattackandassociatedattack mountcacheside-channelattacksareFlush+Reload[92]and
primitives (e.g.,eviction set construction) that does not Prime+Probe[64].InFlush+Reload,anattackerthatshares
requiretheattackerandvictimtosharememory.Weuse memorywithavictimflushesindividualsharedcachelines
these primitives to mount a proof-of-concept attack on and later reloads them to figure out if the victim accessed
constant-timeswapoperations(Section5). them.InPrime+Probe,theattackerbuildsanevictionsetof
3. BreakingConstant-TimeCryptography. Undergirded addressesthatmaptothesamecachesetasthevictim’starget
byourchosen-inputattackframework,inSections6and7 cacheline,primesthecachesetwiththeevictionset,andlater
wedevelopend-to-endkey-extractionattacksonconstant- probesittofigureoutwhetherthevictimaccessedthetarget
timeimplementationsofclassicalcryptography(OpenSSL line/displacedalineintheevictionset.
Diffie-HellmanKeyExchangeandGoRSAdecryption) Classical Prefetchers. Prefetchers are a hardware opti-
andpost-quantumcryptography(CRYSTALS-Kyberand mizationusedtohidememoryaccesslatency.Prefetcherslive
CRYSTALS-Dilithium). inthememorysystem,typicallybetweentheL1andL2or
betweentheL2andDRAM,andworkbypre-loadingdata
intothecachebeforeitisrequestedbythecore.Inparticular,
1.2 Disclosure
givenaprogrammemoryaccesspattern,classicalprefetchers
WedisclosedtoApple,OpenSSL,GoCrypto,andtheCRYS- trytopredictthenextaddressestheprogramwillaccessbased
TALS team. Apple is investigating our PoC. OpenSSL re- onitsaccesspattern(anaddresstrace)thusfar.
ported that local side-channel attacks (i.e., ones where an ClassicalPrefetcherSecurityImplications. Severalprior
attackerprocess runs on the same machine) falloutside of works have analyzed the security implications of classical
1118 33rd USENIX Security Symposium USENIX Association

Dereferenced by code
prefetchers[17,26,27,31,76,91,98].Theseworksdemon- Prefetched by stream
Dereferenced by DMP
stratethat,throughunintendedinteractionswithprefetchers, prefetcher
|     |     |     |     |     |     |     | Augury |     |     |     | …   |     | … ptr[M-1] |
| --- | --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | ---------- |
victimprogramscancreatecachestatechangesthatcanbe ptr[0] ptr[1] ptr[2] ptr[N-1] ptr[N]
| measured | by the | attacker | to leak | information. | Fortunately, |     |     |     |     |     |     |     |     |
| -------- | ------ | -------- | ------- | ------------ | ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
|          |        |          |         |              |              |     |     | *   | *   | *   | *   | *   | *   |
leakagethroughtheseattacksislimitedtothevictim’saccess
| patternandcanbemitigatedthroughconstant-timeprogram- |     |     |     |     |     |     |           |       |       |       | …     |        | …        |
| ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | --------- | ----- | ----- | ----- | ----- | ------ | -------- |
|                                                      |     |     |     |     |     |     | This Work | dummy | dummy | dummy | dummy | ptr[N] | ptr[M-1] |
mingpracticesthatensuretheprogrammemoryaccesspattern
| doesnotdependonsecrets.                |     |     |     |     |         |     |     |     |     |     |     | *   | *   |
| -------------------------------------- | --- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
| DataMemory-DependentPrefetchers(DMPs). |     |     |     |     | DMPsare |     |     |     |     |     |     |     |     |
…
aclassofprefetchersdesignedtoprefetchirregularmemory This Work ptr[0] ptr[1] ptr[2] ptr[M-1]
accesspatterns.Incontrasttoclassicalprefetchers,whichonly * * * *
takethememoryaccesspatternasaninput,DMPsalsotake
intoaccountthecontentsofdatamemorydirectlytodetermine This Work ptr[0] ptr[1] … ptr[7] Loaded alongside
(within same cache line)
| what to prefetch. |     | The computer | architecture |     | literature | and |     |     |     |     |     |     |     |
| ----------------- | --- | ------------ | ------------ | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
|                   |     |              |              |     |            |     |     | *   | *   |     | *   |     |     |
industrypatentsproposedseveraltypesofDMPs[7,8,16,24,
|     |     |     |     |     |     |     | Figure 1: | We  | compare | memory | access | patterns | and subse- |
| --- | --- | --- | --- | --- | --- | --- | --------- | --- | ------- | ------ | ------ | -------- | ---------- |
29,50,84,96,97],whichdifferintheirregularaccesspatterns
quentprefetches.Thefirstrowrepresentstheactivationpat-
thattheyaredesignedtospeedup(e.g.,linked-listtraversals,
ternreportedbyAugury[84]:astreamingdereferenceaccess
sparsematrixtraversals).
patterncausestheDMPtodereferenceout-of-boundspointers.
| DMPSecurityImplications. |     |     | Vicarteetal.werethefirstto |     |     |     |     |     |     |     |     |     |     |
| ------------------------ | --- | --- | -------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Inthesecondrow,weshowthatarchitectural/program-level
performananalysisofthesecurityimplicationsofDMPs[72].
dereferencesareunnecessary;weseeDMPactivationseven
Intheworstcase,theyfoundthatproposed(butnotknown
whenthetrainingarraycontainsnon-pointervalues.Inthe
| to be implemented) |     | indirect | memory | prefetchers |     | could be |     |     |     |     |     |     |     |
| ------------------ | --- | -------- | ------ | ----------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
thirdrow,weshowthattheDMPevendereferencesthein-
| used to build | universal | read | gadgets | that | leak a program’s |     |     |     |     |     |     |     |     |
| ------------- | --------- | ---- | ------- | ---- | ---------------- | --- | --- | --- | --- | --- | --- | --- | --- |
boundspointersthatarearchitecturallyaccessed(but,again,
| entire memory, |     | similar to | Spectre | [52,60]. | More | recently, |                   |     |             |     |              |       |        |
| -------------- | --- | ---------- | ------- | -------- | ---- | --------- | ----------------- | --- | ----------- | --- | ------------ | ----- | ------ |
|                |     |            |         |          |      |           | notdereferenced). |     | Finally,the |     | lastrowshows | thata | single |
AugurydemonstratedthatmodernAppleprocessorsemploy
accesstoamemorylocationresultsinallpointersstoredin
a type of DMP referred to as an Array-of-Pointers (AoP) theincidentcachelinebeingdereferenced.
DMP[84].WedescribethisDMP’sbehaviorinmoredetail
inSection4.1.
4 MicroarchitecturalCharacterization
3 ThreatModelandSetup
4.1 RevisitingDMPDataAccessPatterns
Inthispaperweassumeatypicalmicroarchitecturalattack
scenario,where the victim and attackerhave two different In this section,we investigate the access patterns required
toactivatetheM1DMP.WeshowthattheM1DMPderef-
processesco-locatedonthesamemachine.
Software. Forourcryptographicattacks,we assume the erencesmorepointersandwithfewerprogramassumptions
attackerrunsunprivilegedcodeandisabletointeractwith thanwasclaimedbyAugury[84].Figure1summarizesthe
subsection’sfindings.
| the victim | via nominal | software | interfaces,triggering |     |     | it to |     |     |     |     |     |     |     |
| ---------- | ----------- | -------- | --------------------- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
perform private key operations. Next, we assume that the Augury. WebeginbyreviewingtheM1DMPactivation
patternandmethodologydescribedinAugury.Augury’scode,
| victim is | constant-time |     | software | that does | not exhibit | any |     |     |     |     |     |     |     |
| --------- | ------------- | --- | -------- | --------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
(known)microarchitecturalside-channelleakage.Finally,we summarizedinListing1(left),firstallocatesanarray(aop)
assumethattheattackerandthevictimdonotsharememory, oflengthMandfillsaopwithpointerstomemoryaddresses
butthattheattackercanmonitoranymicroarchitecturalside thatcorrespondtouniqueL2cachelines.Next,itevictsthese
channelsavailabletoit,e.g.,cachelatency.Aswetestunpriv- cachelinesfromtheL2viacachethrashing(byloadinganar-
rayeighttimesthesizeofthecache).Thecodethenaccesses
ilegedcode,weonlyconsidermemoryaddressescommonly
allocatedtouserspace(EL0)programsbymacOS. (loads)anddereferencesthefirstNelementsoftheaop,where
Hardware. Unlessotherwisespecified,wefocusonApple N≤M.Wecallaop[0],...,aop[N-1]thein-boundspoint-
ersandaop[N],...,aop[M-1]theout-of-boundspointers.
| hardware. | TheM1-basedexperimentsofSection |     |     |     | 4arerun |     |     |     |     |     |     |     |     |
| --------- | ------------------------------- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
onaMacMiniwithanAppleM1runningmacOS13.5.For AuguryinferredtheDMP’sactivitybyaddingcodeafter
ourinvestigationintotheM2/M3microarchitecture,weused thelooptotimehowlongitwouldtaketodereferencepointers
aMacMiniwithanAppleM2(runningmacOS14.2.1)and intheaop.Wecallthesetestaccesses.Themainfindingwas
a MacBookPro withan Apple M3 (running macOS 14.2). thatthelatencyoftestaccessesforout-of-boundspointersin
Finally,wheninvestigatingIntel’sDMPimplementation,we someindexrange[N,N+δ)correspondedtoL2cachehits.
usedanIntelCorei9-13900K(RaptorLake)CPU,running Thisisnoteworthybecausethecodeitselfneverdereferenced
Ubuntu23.04withkernelversion6.2.0. pointerslocatedafteraop[N].Auguryattributedthisbehavior
| USENIX Association |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1119 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

toanewformofprefetcher,withprefetchdistanceδ.
|              |                   |              |                   | )selcyc( ycnetaL sseccA No Pointer | Contain Pointer |     | )selcyc( ycnetaL sseccA No Pointer | Contain Pointer |
| ------------ | ----------------- | ------------ | ----------------- | ---------------------------------- | --------------- | --- | ---------------------------------- | --------------- |
|              |                   |              |                   | 800                                |                 |     | 800                                |                 |
|              |                   |              |                   | 7 0 0                              |                 |     | 700                                |                 |
| uint64_t*    | aop[M];           | uint64_t*    | aop[M];           |                                    |                 |     |                                    |                 |
|              |                   |              |                   | 6 0 0                              |                 |     | 600                                |                 |
| // Fill      | aop with pointers | // Fill      | aop with pointers | 500                                |                 |     | 500                                |                 |
|              |                   |              |                   | 400                                |                 |     | 400                                |                 |
| // to unique | addresses         | // to unique | addresses         | 300                                |                 |     | 300                                |                 |
|              |                   |              |                   | 200                                |                 |     | 200                                |                 |
| // or random | values            | // or random | values            | 100                                |                 |     | 100                                |                 |
|              |                   |              |                   | 0 256257258259260261262263         |                 |     | 0 256257258259260261262263         |                 |
| for (i=0;    | i<N; i++) {       | for (i=0;    | i<N; i++) {       |                                    |                 |     |                                    |                 |
|              |                   |              |                   |                                    | Test Index      |     |                                    | Test Index      |
*aop[i%N]; aop[i%N]; (a) Row 1: Traversing the AoP (b) Row 2: Traversing the AoP
| }                                                  |                   | }          |                   |                                    |                 |                             |                                    |                 |
| -------------------------------------------------- | ----------------- | ---------- | ----------------- | ---------------------------------- | --------------- | --------------------------- | ---------------------------------- | --------------- |
|                                                    |                   |            |                   | withdereferences;out-of-bounds     |                 | without                     | dereferences;                      | out-of-         |
|                                                    |                   |            |                   | pointersareprefetched              |                 | boundspointersareprefetched |                                    |                 |
| // Measure                                         | latency to        | // Measure | latency to        |                                    |                 |                             |                                    |                 |
| // set                                             | of test addresses | // set     | of test addresses |                                    |                 |                             |                                    |                 |
|                                                    |                   |            |                   | )selcyc( ycnetaL sseccA No Pointer | Contain Pointer |                             | )selcyc( ycnetaL sseccA No Pointer | Contain Pointer |
|                                                    |                   |            |                   | 800                                |                 |                             | 800                                |                 |
| Listing1:Left:TheDMPactivationcodepatternstudiedby |                   |            |                   | 700                                |                 |                             | 700                                |                 |
|                                                    |                   |            |                   | 600                                |                 |                             | 600                                |                 |
| Augury[84].Right:TheDMPactivationpatternstudiedin  |                   |            |                   | 500                                |                 |                             | 500                                |                 |
|                                                    |                   |            |                   | 400                                |                 |                             | 400                                |                 |
thiswork.Forboth,assumeN≤M.Bothcodepatternsfill
|                                                |     |     |     | 300        |                |         | 300    |                  |
| ---------------------------------------------- | --- | --- | --- | ---------- | -------------- | ------- | ------ | ---------------- |
|                                                |     |     |     | 200        |                |         | 200    |                  |
| theaopbeforetheloopbeginsanduseamodoperationto |     |     |     | 100        |                |         | 100    |                  |
|                                                |     |     |     | 0          |                |         | 0      |                  |
| inhibitspeculativeexecution.                   |     |     |     | 0 1        | 2 3 4 5 6      | 7       | 0 1 2  | 3 4 5 6 7        |
|                                                |     |     |     |            | Test Index     |         |        | Test Index       |
|                                                |     |     |     | (c) Row 3: | Traversing the | AoP (d) | Row 4: | One load to AoP; |
ObservingDMPActivations. WereproduceAugury’sex- withoutdereferences;in-bounds pointerswithintheincidentcache
|     |     |     |     | pointersareprefetched |     | lineareprefetched |     |     |
| --- | --- | --- | --- | --------------------- | --- | ----------------- | --- | --- |
perimentsbysettingN=256andM=264,choosingasetof
testpointers,andtheneitherfillingtheout-of-boundsregion
Figure2:Median,minimum,andmaximumtestaccesslaten-
withthosepointersorrandomvalues.Whenthepointersare
cies(over32samplesforeachbar)usingtheaccesspatterns
present,atestaccess(dereference)toonetakes∼250cycles,1
ofFigure1.Thexaxiscorrespondstothetestaccesslatency
asshowninFigure2a.Whenthepointersarenotpresent,the
forthepointeratthecorrespondingindexintheaopincaseit
sametestaccessestakesignificantlylonger.Acutoffof300
containsthepointer.Bluebars(NoPointer)areforwhenthe
cycles(reddashline)cleanlydifferentiatesbetweenthetwo
testpointerisnotintheaoparray,whileredbars(Contain
casesandthusDMPactivations.ThiscorrespondstotheL2
Pointer)areforwhenthepointerisinthearray.
hittimeandmatchesAugury’sfindings,consistentwithδ≥8.
| AvoidingArchitecturalPointerDereferencing. |     |     | Todeter- |     |     |     |     |     |
| ------------------------------------------ | --- | --- | -------- | --- | --- | --- | --- | --- |
mineifthearchitecturalpointerdereferencesarerequiredto
loadandnoarchitecturaldereference,asshowninFigure1
triggerDMPactivationsweusethecodeinListing1(right),
(fourthrow).Eventhoughtheprogramonlyloadsoneaop
where the in-bounds region does not contain pointers nor index,otherpointersinthesamecachelinearealsobrought
doestheaoptraversalloopperformanypointerdereferences. intothecache.Figure2dshowsthatwithasingleload,2we
Again,weeitherfilltheout-of-boundsregionwithtestpoint-
|     |     |     |     | observesimilarresultstotraversingtheentire(N |     |     |     | =8)aop |
| --- | --- | --- | --- | -------------------------------------------- | --- | --- | --- | ------ |
ersorrandomvalues.SeeFigure1(secondrow). inFigure2c.Wefurtherrepeattheexperimentbutvarythe
AsseeninFigure2b,whentheout-of-boundsregioncon-
numberofpointersinthecachelinefrom1to8.Inallcases,
tainspointers,testaccessesare<300cyclesdespitenoar-
weobserveDMPactivations/dereferencesforallpointersin
chitecturaldereferencesoccurringtothein-boundspointers.
aop,indicatingthatevenasinglepointercantriggertheDMP.
Fromthis,wededucethatarchitecturaldereferencesarenotre-
quiredfortheDMPtoactivate,i.e.,thattheDMPwillprefetch
4.2 DMPActivationCriteria
out-of-boundspointerswithoutthem.
| In-boundsDMPDereferencing. |     | Wethenfurthercheckif |     |                    |      |        |        |                   |
| -------------------------- | --- | -------------------- | --- | ------------------ | ---- | ------ | ------ | ----------------- |
|                            |     |                      |     | Having established | what | memory | access | patterns activate |
thein-boundspointersarealsodereferencedbytheDMPas
theDMP,thissectioninvestigateswheredatamustresidein
theyarenolongerarchitecturallydereferencedinListing1
thememoryhierarchytobeDMP-searchedforpointers.We
(right).ThisisthememoryaccesspatternoutlinedinFigure1
showthattheDMPdereferencespointersspecificallyonL1
(thirdrow),whereweiterateoveranarraycontainingvalid
cachefillsandfeaturestwomechanismstopreventredundant
pointerswithoutperforminganydereferences.
|     |     |     |     | prefetches: | a history filter | and | a do-not-scan | hint. In this |
| --- | --- | --- | --- | ----------- | ---------------- | --- | ------------- | ------------- |
Figure2cshowsthatforN=8,wecanstillconsistently
section,wemakeuseofstandardevictionsets,i.e.,eviction
differentiatebetweenthetwocases.Thisindicatesthatifthe
setsforindividualcachesets.Wegeneratetheseevictionsets
aopcontainsdatawhichcanbeinterpretedasvalidpointers,
usingstandardtechniquesfrompriorwork[85].3
merelyiteratingoveritissufficienttoactivatetheDMP.
|                        |     |                              |     | HistoryFilter. | Westartbyrerunningtheexperimentsfrom |     |     |     |
| ---------------------- | --- | ---------------------------- | --- | -------------- | ------------------------------------ | --- | --- | --- |
| OneLoad,SinglePointer. |     | Finally,weconsiderhowgeneral |     |                |                                      |     |     |     |
thememoryaccesspatterncanbebyperformingasingledata 2Replacingtheloadwiththestoreinstruction,wefindthatnoneofpoint-
ersintheaccessedcachelinearedereferenced.
1Wecollecttimingmeasurementsbyconfiguringandreadingperformance 3ThisisincontrastwithAugury,which,aswementionedinSection4.1,
counters(PMC2-PMC7)forcyclecountingviakperf. reliedoncachethrashingtopreconditionthecache.
| 1120    33rd USENIX Security Symposium |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

Section4.1usingstandardL2evictionsetstoevictboththe
aoparrayandtheL2cachelinesthatarepointedtobypointers
intheaoparray.WecalltheseL2linesthetargetlines.
WeobservethattheDMPonlyreliablydereferenceseach
pointeronce,onthefirstaccesstoitsaopentry.Thatis,evenif
thepreviouslyprefetchedtargetlineisevictedfromthecache,
alongwithitsaopentry,theDMPnolongeractivateswhen
seeingthatpointerinthefuture. Thisobservationsuggests
thatthedecisiontodereferenceapointerismadebasedonnot
onlytheprogram’saccesspatternbutalsosomeadditional
mechanism.AnApplepatentsuggeststhatthismechanism
might be a history filter that “attempts to identify whether
a given memory pointer candidate likely corresponds to a
candidatethathasbeenrecentlyprefetched,inwhichcasethe
givencandidatemaybediscardedasalikelyduplicate”[47].
Thesamepatentsuggeststhatthisfiltermaybeorganizedas
adirect-mapped128-entryor256-entrystructure.
HistoryFilterReverseEngineering. Tocorroboratethe
historyfilterhypothesis,wedesignanewexperimentwhere
aoponlycontainsasinglepointerptr.First,weaccessaop,
causing the DMP to dereference ptr. We then evict aop
and the target line for ptr from the cache using standard
eviction sets. Next, we read S unique pointers stored in a
differentarray,causingtheDMPtoinspectanddereference
Sadditionalpointers.Finally,were-accessaopandcheckif
thissecondaccesscausestheDMPtodereferenceptr.We
runtheexperiment100timesforeachvalueofSandreport
thesuccessrate(i.e.,thepercentageoftimesthattheDMP
activatedonthesecondaopaccess)inFigure3.
80
70
60
50
40
30
20
10
0
1 2 4 8 16 32 64 128256
Number of different ptrs
)%(
etaR
sseccuS
arrayeighttimesthesizeofthecache)hadasideeffectof
alsoflushingthehistoryfilter.
Wefurtherfindthatthehistoryfilterisaper-corestructure
andisresetifacoreremainsidleforanextendedperiodof
time.Specifically,theDMPreliablyre-activatesevenwhen
S=0ifwe(i)rescheduleourexperimenttoadifferentcore
between the first and the second aop access or(ii) run the
experimentononecorebutleavethecoreidlefor100µsor
morebetweenthefirstandthesecondaopaccess.
L1andL2CacheFills. Theaboveobservationsindicate
thattheDMPactivateswhenanaopentryisaccessedfrom
DRAMandtherecordofitstargetisnotpresentinthehistory
filter.Next,weinvestigateatwhichstageofaDRAMfetch
theDMPscansthedataforpointers.RecallthattheM1has
anL2linesizeof128BytesandanL1linesizeof64Bytes.
Witheachpointercontaining8Bytes,L2linescanthusbe
split into “lower” and “upper” halves,each of which is an
independentL1linethatcanstore64/8=8pointers.Whena
programaccesseseitherthelowerorupperhalf,theaccessed
L1linewillbefilledintoboththeL1andL2caches,while
theotherhalfwillonlybefilledintotheL2cache.4Inorder
to differentiate between L1 and L2 fills,we populate a L2
line size-aligned aop with 16 unique pointers and run the
experimentfromListing1(right)inSection4.1withN=1
andM=16.Beforeeachrepetition,weusecachethrashing
(asinSection4.1)toevicttheaopanditstargetlinesfrom
boththecacheandthehistoryfilter.
Figure4(top)summarizesourfindings,repeatingeachex-
periment100timesandusingthe300cyclethresholdfrom
Section 4.1 forL2 cache hits. Here,we observe thatwhen
the program accesses aop[0],the DMP only dereferences
aop[0],···,aop[7].Werun7morevariantsofthisexperi-
ment,varyingthesingleaop[i]accessfromi=1,···,7and
observethesamebehaviorforeachchoiceofi.Next,werun
8morevariantsofthesameexperiment,thistimemakinga
singleaccesstoaop[i]fori=8,···,15.Inthiscase,weob-
servethataop[8],···,aop[15]arealldereferencedforeach
choiceofi,asshowninFigure4(bottom).Weconcludethat
whenfillinganL2cachelinefromDRAM,theDMPderefer-
encesallpointersinthespecificL1linethatisaccessed,and
Figure 3: The percentage of experiments where the DMP notthoseintheotherhalfoftheL2line.
re-activateswhenptrisre-accessed(SuccessRate,y-axis), Werun8morevariantsoftheaboveexperiment.Forthese,
asafunctionofthenumberofuniquepointersaccessedin beforemakinganaccesstoaop[i]fori=0,···,7,wefirst
betweenthefirstandsecondaccesstoptr(x-axis).Observe makeanaccesstoaop[8].Wethenrepeatthissetupwhileex-
thatSuccessRateincreaseswiththenumberofuniqueinter- ploringtheoppositecase:beforemakinganaccesstoaop[i]
mediatepointeraccesses. fori=8,···,15,wefirstmakeanaccesstoaop[0].Asdis-
cussed above,the first access brings aop[i] from DRAM
totheL2cacheandaop[i]furthermovestotheL1cache
We observe that the DMP only reliably re-activates on
with the second access. We observe that the DMP reliably
ptr when S≥128. This behavior is likely due to the lim-
dereferencesthecontentsoftheL1linecontainingaop[i].
itedcapacityofthehistoryfilter.Thatis,accessingSunique
ThismeansthatL2toL1fillscanalsoactivatetheDMP.
pointersresultsintherecordofptr’stargetgettingevicted
Do-not-scan Hint. The above experiments suggest that
fromthefilterwhenS≥128.WehypothesizethatAugury’s
methodologywasnotaffectedbythehistoryfilterbecause 4Weempiricallyverifythisbysubsequentlytiminganaccesstotheother
itsaggressivecachethrashingtechnique(i.e.,accessingan halfandobservingthatitsaccesslatencycorrespondstothethatofanL2hit.
USENIX Association 33rd USENIX Security Symposium 1121

andrelyoncachethrashingtoensurethattheaopisuncached.
)%( etaR sseccuS 100
|     |     |     |     |     |     |     | Wethen | trytestingdifferentpointervaluesin |     |     |     |     | theaop,and |     |
| --- | --- | --- | --- | --- | --- | --- | ------ | ---------------------------------- | --- | --- | --- | --- | ---------- | --- |
checkingforDMPactivations.
50
|     |     |     |     |     |     |     | 4GByte | Prefetch | Region. |     | We begin | by investigating |     | if  |
| --- | --- | --- | --- | --- | --- | --- | ------ | -------- | ------- | --- | -------- | ---------------- | --- | --- |
theDMPrequirestheretobearelationshipbetweenthead-
0
|                      |     |     |     |     |     |     | dress | of the aop   | entry | and the     | value | of the aop | entry     | (i.e., |
| -------------------- | --- | --- | --- | --- | --- | --- | ----- | ------------ | ----- | ----------- | ----- | ---------- | --------- | ------ |
| )%( etaR sseccuS 100 |     |     |     |     |     |     |       |              |       |             |       | aop        |           |        |
|                      |     |     |     |     |     |     | the   | pointer). We | call  | the address | of    | the        | entry the | en-    |
try’s/pointer’sposition.Tounderstandwhattherequirements
| 50  |     |     |     |     |     |     | areforonepointertobedereferenced,wecarryoutaseriesof |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
experimentsthatvaryapointer’spositionandvalue.Seeone
|     | 0   |       |       |          |       |          | suchexperimentinFigure5whichshowsthatthepointer’s |     |     |     |     |     |     |     |
| --- | --- | ----- | ----- | -------- | ----- | -------- | ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
|     | 0   | 1 2 3 | 4 5 6 | 7 8 9 10 | 11 12 | 13 14 15 |                                                   |     |     |     |     |     |     |     |
positionandvaluemustberelatedforDMPactivationtooc-
Position Offset
Figure4:WhichpointersinanL2linearedereferencedwhen cur.Overall,wediscoverthattheDMPonlydereferencesa
anaccessismadetodatainthatline?Top:thecodeaccesses pointeriftheaopentryandtargetlineareinthesame4GByte-
aop[0].Bottom:thecodeaccessesaop[8].Weconcludethat alignedregion(Figure6).Inotherwords,thattheupper32
theDMPdereferencespointersinthespecificL1line(either bitsoftheiraddressesmatch.Apple’spatent[47]mentions
theupperorlowerhalfoftheL2line)thecodeaccessed. similarpointerdetectionheuristic.
| theDMPsearchesforpointersinL1cachelinesduringL1 |     |     |     |     |     |     |     | 0x47 |     |     |     |     |     |     |
| ----------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0x46
fills,regardlessofwhethertheL1lineisfetchedfromDRAM
| ortheL2.Tocorroboratethishypothesis,wedesignanother |     |     |     |     |     |     |     | 0x45 |     |     |     |     |     |     |
| --------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
0x44
| variant        | of the | single-pointer | experiment |         | from | Section 4.1. |     |      |     |     |     |     |     |     |
| -------------- | ------ | -------------- | ---------- | ------- | ---- | ------------ | --- | ---- | --- | --- | --- | --- | --- | --- |
| The experiment |        | starts         | by loading | the aop | into | the L1 and   |     | 0x43 |     |     |     |     |     |     |
subsequently using eviction sets to either (i) evict the aop 0x42 100
| from | the L1 | or(ii) evict | the aop | from both | the | L1 and L2. | eulav retnioP | 0x41 |     |     |     |     |     |     |
| ---- | ------ | ------------ | ------- | --------- | --- | ---------- | ------------- | ---- | --- | --- | --- | --- | --- | --- |
80
Inbothcases,theexperimentalsoevictsthetargetlinefrom )%( etaR sseccuS
0x40
| thecacheandaccessesaseparatesetof256pointerstoevict |     |            |           |             |         |          |     | 0x3f |     |     |     |     |     | 60  |
| --------------------------------------------------- | --- | ---------- | --------- | ----------- | ------- | -------- | --- | ---- | --- | --- | --- | --- | --- | --- |
| the record                                          | of  | the target | line from | the history | filter. | Finally, |     |      |     |     |     |     |     |     |
|                                                     |     |            |           |             |         |          |     | 0x3e |     |     |     |     |     | 40  |
theexperimentre-accessesaopandtestsifthissecondaop
0x3d
| accesscausestheDMPtore-dereferenceptr. |     |     |     |     |     |     |     |     |     |     |     |     |     | 20  |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
0x3c
| Interestingly, |     | we  | observe | that the | DMP | does not re- |     |     |     |     |     |     |     |     |
| -------------- | --- | --- | ------- | -------- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
0x3b
|             |     | ptr  |                |             |     | aop |     |     |     |     |     |     |     | 0   |
| ----------- | --- | ---- | -------------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| dereference |     | when | the experiment | re-accesses |     | and |     |     |     |     |     |     |     |     |
0x3a
aopwasonlyevictedfromtheL1.However,whentheaopis
0x39
alsoevictedfromtheL2,theDMPre-dereferencesit.This
0x38
| means | that | even if | the previously | prefetched |     | target line is |     |                |           |                |           |                     |           |     |
| ----- | ---- | ------- | -------------- | ---------- | --- | -------------- | --- | -------------- | --------- | -------------- | --------- | ------------------- | --------- | --- |
|       |      |         |                |            |     |                |     | 83x0 93x0 a3x0 | b3x0 c3x0 | d3x0 e3x0 f3x0 | 04x0 14x0 | 24x0 34x0 44x0 54x0 | 64x0 74x0 |     |
evictedfromboththecacheandthehistoryfilter,theDMP
doesnotdereferencethatpointeragainunlessitsaopentryis
Pointer Position
alsoevictedfromboththeL1andL2.Thisbehaviormatches Figure5:Forvariouscombinationsofpointerpositionand
amechanismalsodescribedinthepreviouslyreferencedAp- value,when does the DMP dereference the pointer? Here,
0x380000000
ple patent [47],where the L2 sets a “do-not-scan” hint on we sweep within the region between and
L1cachefillstopreventapreviouslyscannedL1cacheline 0x480000000.Thewhitediagonalshowsthedegeneratecase
from being redundantly re-scanned. Fortunately,in ourex- whenthepointer’svalueequalsitsposition,whichisinvalid.
periments,evictingtheaopfromboththeL1andtheL2is Thelower28bitsoftheaddressesareomittedforbrevity.
sufficienttoclearthe“do-not-scan”hintontheaop.
|     |     |     |     |     |     |     | TopByteIgnore. |     | TheaddressspacestandardsinARMv8 |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --- | ------------------------------- | --- | --- | --- | --- | --- |
4.3 RestrictionsonDereferencedPointers directtheprocessortoignorethetopbyteofthevirtualad-
dress[2].TolearnwhethertheDMPfollowsthisspecification,
In the previous section,we learnedthatthe DMP activates weperformaseriesofexperimentsflippingdifferentupper
onL1fillsanddereferencesthepointersinsideitifandonly bitsinavalidpointer.Wethenperformatestaccesstocheck
ifthosepointers’targetsarenotinthehistoryfilterandthe whether,afterthesebitflips,theDMPstilldereferencesthe
filledlineisnotmarkedwiththe"do-not-scan"hint.Wenow originalpointer.Figure7showstheresults.Weperform16
investigatewhat pointerscanbedereferencedbytheDMP. experiments,whereeachflipsabitintheaddressstartingat
Forthis,weagainuseListing1(right)withN=1andM=1 bit48andendingatbit63.WeobservethattheDMPdoesnot
| 1122    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

Further,perSection4.3,thecachelinethatstoresthepointer
*
|     |        |     |     | (itsposition)mustbeinthesame4GByte(log |     |     | 4GByte = |
| --- | ------ | --- | --- | -------------------------------------- | --- | --- | -------- |
|     |        | …   |     |                                        |     |     | 2        |
| aop | target |     | aop |                                        |     |     |          |
32bits)-alignedregionasthecachelinethatthepointerpoints
to(itstarget).Inotherwords,theDMPcheckswhetherbits
|     | *   | 4GByte Bound |     |            |                       |           |               |
| --- | --- | ------------ | --- | ---------- | --------------------- | --------- | ------------- |
|     |     |              |     | [55:32] of | the candidate pointer | match the | corresponding |
Figure6:Outlineoftheplacementofthetargetlineandthe
aopentry.FollowingtheobservationfromFigure5,iftheaop bitsoftheaddressofthetargetcacheline.Finally,theDMP
checksifthecandidatepointerispresentinthehistoryfilter
entryandtargetlinestraddlea4GByteboundary,theDMP
won’tdereferencethepointer. (Section4.2).Ifbits[55:32]matchandthepointerisnotin
thehistoryfilter,theDMPattemptstoprefetchtwoL2lines.
Specifically,itfirstprefetchesthecachelinetargetedbythe
dereferencetheoriginalpointerifabitintherange[48,55]is 64-bit chunk,ignoring the top byte value. Next,it triggers
flipped.However,ifabitintherange[56,63]isflipped,the theCPU’snextlineprefetcherandfetchestheneighboring
originalpointergetsdereferenced.WeconcludethattheDMP cachelinealsointotheCPU’sL2cache(Section4.3).Both
ignorestheupper8bitsofapointerwhendereferencingit, prefetchedaddressesaretheninsertedtothehistoryfilter.
whichmatchesthe“Top-Byte-Ignore”inARMv8.
Aspartoftheprefetchingprocess,theDMPlooksupthe
|     |     |     |     | translation | lookaside buffer | (TLB) and triggers | page table |
| --- | --- | --- | --- | ----------- | ---------------- | ------------------ | ---------- |
)%( etaR sseccuS 100 walkstoobtainthephysicaladdresscorrespondingtoeach
| 80  |     |     |     | candidatepointer(whichisavirtualaddress[33]).OnaTLB |     |     |     |
| --- | --- | --- | --- | --------------------------------------------------- | --- | --- | --- |
miss,theDMPinsertsthemissingtranslationsintotheTLB.6
60
40
20
4.5 OtherMicroarchitectures
0
48495051525354555657585960616263
Flip Bit Index
WeinvestigatedtheDMPbehavioronothermicroarchitec-
Figure 7: Activation success rate for a pointer when it is turesincludingtheAppleM2/M3andIntel’s13thGeneration
accessedbytheprogram,afterhavingonebitflippedbetween
(RaptorLake)CPUs,anddisplayresultsinFigures8aand8b.
bit48to63.
AstheAppleM3behavessimilarlytotheM2,weomitits
figure.Inthesetwofigures,thex-axisreferstothefouraccess
Auxiliarynext-lineprefetch. Finally,weinvestigatethe patterns shown as the rows in Figure 1,while the y-axis is
amount of data prefetched when the DMP dereferences a theaccesslatencyfortestaccesses.Forsimplicity,weonly
pointer.Wetestthisbyperformingatestaccesstonotonlya show latencies for test accesses to the first pointer in each
pointer’stargetline,butalsotonearbylines.Apartfromthe pattern.TheInteli9-13900K(RaptorLake)showsadistin-
targetline,wealsoobserveL2hitstocachelinesimmediately guishabletimingdifferenceonlyforthefirstaccesspattern
next to the target line. We hypothesize that this is due to fromFigure1,whereastheM2/M3activatesonallthepat-
a next-line prefetcher being triggered alongside the DMP, ternsdiscussedpreviously.WeconcludethatwhileDMPsare
whichmatchestheadjacent-lineprefetchbehaviordescribed
presentonRaptorLakemachines,theyrequiredifferentacti-
inApple’spatent[47]. vationpatterns.Finally,weleavethesystematicinvestigation
andexplorationofIntel’sDMPstofuturework.
4.4 AModelfortheDMP’sBehavior
|     |     |     |     | No Pointer | Contain Pointer | No Pointer | Contain Pointer |
| --- | --- | --- | --- | ---------- | --------------- | ---------- | --------------- |
Wenowsummarizetheprevioustwosubsectionsandmake )selcyc( ycnetaL sseccA 800 )selcyc( ycnetaL sseccA
800
| severalnewobservations. |     |     |     | 700 |     | 700 |     |
| ----------------------- | --- | --- | --- | --- | --- | --- | --- |
|                         |     |     |     | 600 |     | 600 |     |
500
| Step 1: Observing | Cache Line | Data. | The DMP scans |     |     | 500 |     |
| ----------------- | ---------- | ----- | ------------- | --- | --- | --- | --- |
|                   |            |       |               | 400 |     | 400 |     |
the data in an L1 line when that line is filled to the L1,if 300 300
|                                                     |     |     |     | 200 |     | 200 |     |
| --------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| thelineisnotmarkedwiththe“do-not-scan”hint(i.e.,the |     |     |     | 100 |     | 100 |     |
|                                                     |     |     |     | 0   |     | 0   |     |
linehasnotbeenscannedsinceitwasbroughtintothecache; Row 1 Row 2 Row 3 Row 4 Row 1 Row 2 Row 3 Row 4
Section4.2).TheDMPperformsthescanbycheckingeach (a)AppleM2 (b)IntelRaptorLake
pointersize-alignedchunk(thefirst64bits,second64bits,
Figure8:WetestfouraccesspatternsshowninFigure1on
etc.)inthecacheline.5
AppleM2(left)andIntel13thgenerationRaptorLake(right).
| Step2:AddressCheck. | Next,theDMPappliesadditional |     |     |     |     |     |     |
| ------------------- | ---------------------------- | --- | --- | --- | --- | --- | --- |
checksandfilterstoeachchunk(candidatepointer)toseeifit
shouldbedereferenced.Bits[63:56]areignored(Section4.3).
6Priorwork[84]alsoobservesthattheM1DMPfillsTLBentriesfor
5Pointersinaopshouldbe64-bitaligned,whichisalsodiscussedin[84].
pointersintheaop.
| USENIX Association |     |     |     |     | 33rd USENIX Security Symposium    1123 |     |     |
| ------------------ | --- | --- | --- | --- | -------------------------------------- | --- | --- |

5 AttackingConstant-TimeConditionalSwap investigatingthevictim’sprograminadvance.Theattacker
process’goalistoextractthevalueofsecretfromthevictim,
Tomitigatemicroarchitecturalsidechannels,cryptographic
usingmicroarchitecturalsidechannelsandtheDMP.
code follows the constant-time programming principle: A Chosen-InputAttack. Wenowoverviewhowtheattacker
| secret should | not | determine |     | which instructions |     | to  | execute, |     |     |     |     |     |     |     |
| ------------- | --- | --------- | --- | ------------------ | --- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
usestheDMPtoextractsecret.Atahighlevel,theattacker
whichmemorytoaccess,orbeusedasinputforvariable-time populates one of ct-swap’s arrays (a or b—let us assume
instructions[11,13,14,21–23,62]. it chooses b) with a pointer ptr of its choosing, and then
WenowshowhowtheDMPcanbreakcryptographicsecu-
arrangesfortheDMPtodereferencethecontentsoftheother
rityevenwhencodeiswrittentofollowtheconstant-timeprin- array (a) during the conditional swap computation. Then,
ciple.Tointroduceideasandattackertools,thissectionshow-
theattackerusesconventionalcacheside-channelanalysisto
casesaProof-of-Concept(PoC)attackonacoreconstant-time observewhetherptrwasdereferencedbytheDMPdueto
cryptographic primitive [54] called ct-swap which condi- ct-swap’scomputationovera,whichinturnrevealswhether
tionallyswapsthecontentsoftwoarraysaandbbasedon
theswapoccurredandthereforethevalueofsecret.
asecretbitsecret. Westartwithct-swaptosimplifythe OvercomingDMPActivationCriteria. Tocorrectlyat-
presentation.Latersectionswillreusetheideasandprocesses
tributetheDMP’sactivationtoptrbeingmovedfrombto
describedheretobreakrealcryptographiccode. a,theattackermustensurethattheDMP’sactivationcriteria
| Constant | Time     | Swap | Overview.   |     | Listing   | 2 swaps | the    |                    |      |           |     |        |         |          |
| -------- | -------- | ---- | ----------- | --- | --------- | ------- | ------ | ------------------ | ---- | --------- | --- | ------ | ------- | -------- |
|          |          |      |             |     |           |         |        | are only satisfied | when | accessing |     | a (and | not b). | Based on |
| contents | of array | a    | and b based | on  | the value | of      | secret |                    |      |           |     |        |         |          |
Section4.2,onenecessaryprerequisitetoactivatetheDMP
in a constant-time manner. The underlying swap opera- onanaoploadistoevicttheaopfromtheL2cache.Thus,
| tion for | each | 64-bit | entry is | borrowed | from | OpenSSL.7 |     |     |     |     |     |     |     |     |
| -------- | ---- | ------ | -------- | -------- | ---- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
weneedameanstoevicta8(butnotb).
To achieve constant-time behavior, Line 4 in Listing 2 OvercomingAddressSpaceSeparation. Yet,sincethe
| first extends | secret |     | to be | a machine-sized |     | word; | i.e., |     |     |     |     |     |     |     |
| ------------- | ------ | --- | ----- | --------------- | --- | ----- | ----- | --- | --- | --- | --- | --- | --- | --- |
attackerrunsinaseparateprocessfromthevictimandwithout
| 0x0000000000000000 |            |     | or 0xFFFFFFFFFFFFFFFF |      |                 | based | on     |            |           |      |         |     |              |     |
| ------------------ | ---------- | --- | --------------------- | ---- | --------------- | ----- | ------ | ---------- | --------- | ---- | ------- | --- | ------------ | --- |
|                    |            |     |                       |      |                 |       |        | any shared | memory,we | must | replace | the | Flush+Reload | in  |
| the value          | of secret. |     | Next, for             | each | loop iteration, |       | Line 6 |            |           |      |         |     |              |     |
Section4withPrime+Probe.Inparticular,wemustbuildan
ofListing2computesamaskeddeltabetweenthecontents
evictionsettodetectwhetherptrwasdereferencedbythe
|     |     |     | a   | b.  |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
of the current elements of and Finally,Lines 7 and 8 DMPinsidethevictimprocess.However,itisnotclearhow
actuallyconditionallyswapthecontentsofthetwoelements,
|     |     |     |     |     |     |     |     | to build eviction | sets | for ptr’s | target | line | (or a mentioned |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------- | ---- | --------- | ------ | ---- | --------------- | --- |
basedonthevalueofsecret.
|     |     |     |     |     |     |     |     | above),9 aswecannottimeaccessestothesesincetheyare |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- | --- | --- | --- |
void ct-swap(uint64_t secret, uint64_t *a, uint64_t *b, locatedinsidethevictim’saddressspace.
1
size_t len) {
2
| 3 uint64_t |     | delta; |     |     |     |     |     |     |     |     |     |     |     |     |
| ---------- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
4 uint64_t mask = ~(secret-1); 5.2 CompoundEvictionSetConstruction
| 5 for | (size_t | i = | 0; i < len; | i++) | {   |     |     |     |     |     |     |     |     |     |
| ----- | ------- | --- | ----------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
6 delta = (a[i] ^ b[i]) & mask; We now present a novel technique—compound eviction
a[i] = a[i] ^ delta;
| 7   |     |     |     |     |     |     |     | set generation—which |     | solves | the | above problem |     | by using |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------- | --- | ------ | --- | ------------- | --- | -------- |
b[i] = b[i] ^ delta;
8
| }   |     |     |     |     |     |     |     | ct-swap’saccesstoaaswellasDMPdereferencestoptrto |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- | --- |
9
| }   |     |     |     |     |     |     |     | simultaneouslybuildevictionsetsforbothelements. |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- | --- | --- |
10
Listing2:Codesnippetofconstant-timeswap.Thecontents EstablishingaTimingSource. Tostart,weneedtodistin-
ofaandbisconditionallyswappedbasedonsecret. guishbetweenL2hitsandmisses.However,astheattacker
isrunningwithoutelevatedprivileges,itisunabletoaccess
nanosecond-accuratetimersonAppleCPUs,insteadbeing
limitedtothesystem’s42nstimer.Unfortunately,weempir-
| 5.1 AttackOverviewandChallenges |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
icallyfindthatthistimerisnotsufficienttoreliablymount
|     |     |     |     |     |     |     |     | Prime+Probe | attacks. | We sidestep |     | this issue | by  | using the |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | -------- | ----------- | --- | ---------- | --- | --------- |
Emulatingrealisticattackscenarios,weassumethatct-swap
multi-threadtimerapproachof[46,69,73].Here,themain
runsinavictimprocess,separatefromtheattacker’saddress
ideaistouseadedicatedcountingthread,whichconstantlyin-
| space. We | assume | a   | simple butcommon |     | protocolbetween |     |     |     |     |     |     |     |     |     |
| --------- | ------ | --- | ---------------- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
crementsasharedvariablewiththeattackerprocessinatight
| victim and | attacker,where |     | the | victim | takes | input from | the |     |     |     |     |     |     |     |
| ---------- | -------------- | --- | --- | ------ | ----- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
loop.Byloadingthevalueofthesharedvariable,theattacker
attackertopopulatethect-swap’saandbarraysandthen
executesct-swap.Theoutcomeoftheswapisneverdirectly
8TriggeringtheDMPalsorequiresthataisrefilledafteritisevicted.We
revealed,noristhevalueofsecret.Theattackercanlearn
relyonthevictimtoperformthisrefill.Forexample,ct-swapreadsaina
pageoffsets(notrandomizedbyASLR)ofarrayaandbby loop,whichwillcauseeachcachelinemakingupatobeaccessed(refilled)
multipletimes(len>1).
7constant_time_cond_swap_64: https://github.com/openssl/ 9Weassumethatthebaseaddressesofaandbhavedifferentpage-offset
openssl/blob/1751185154ab1f1a796e0f39567fe51c8e24b78d/ bits,sothattheevictionsetforawouldnotevictb,whichalsoholdsforlater
| include/internal/constant_time.h.      |     |     |     |     |     |     |     | attacks. |     |     |     |                    |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | ------------------ | --- | --- |
| 1124    33rd USENIX Security Symposium |     |     |     |     |     |     |     |          |     |     |     | USENIX Association |     |     |

process is thus able to obtain high resolution timestamps, tack on ct-swap to learn the secret secret. For all proof-
allowingustodistinguishL2hitsfrommisses. of-concept attacks, we use three attacker processes. The
GeneratingStandardEvictionSets. Next,weneedtogen- first process establishes a TCP connection with the victim
eratealargenumberofstandardL2evictionsets,i.e.,eviction process and transmits the value of ptr to the victim. The
setstargetedtoindividualL2sets.TheM1has8192(213)L2 victim process upon receiving ptr subsequently executes
cachesets,indexedwith6(upper)bitsfromthephysicalpage ct-swap(a,b,secret)whereaissomedummyvalue,bis
frameand7(lower)bitsfromthepageoffset.Wegenerate full of multiple copies of ptr,and secret is a hardcoded
standardevictionsetsforalltheseL2setsbyextendingthe value.Inparallel,weusethesecondattackerprocesstocon-
techniqueusedinSection4.2(detailedinAppendixA). tinuouslytraverseEV ,evictingafromtheCPU’sL2cache
a
GeneratingCompoundEvictionSets. Withallstandard duringtheexecutionofLine7ofListing2.Finally,thethird
L2evictionsetsinhand,wenowneedtotestwhichofthese attackerprocessprovidesahigh-resolutiontimingsourcevia
arecapableofevictingthetargetofptr.AsdescribedinSec- acountingthreadthatconstantlyincrementsasharedvariable.
Aftertransmittingthevalueofptrtothevictim,ourfirstat-
tion5.1,thisisnon-trivialbecauseobservingthedereference
of ptr via DMP activations requires an eviction set for a tackerprocessusesthePrime+ProbechannelbuiltonEV to
ptr
whichwecannotcreatewithstandardtechniques. monitortheDMPactivation.Weperform3200attacktrials,11
Tosolvethisproblem,wewillbuildandtestwhatwecall forbothvaluesofsecret.Figure9summarizesourfindings,
compoundevictionsets,whichsimultaneouslyevictboththe with the timing distributions for secret=1 and secret=0
targetofptranda.Webuildcandidatecompoundeviction
beingclearlydistinguishable.
| setsaspairs(EV | ,EV | )ofstandardL2evictionsets,where |     |     |     |     |     |     |     |
| -------------- | --- | ------------------------------- | --- | --- | --- | --- | --- | --- | --- |
a ptr
| EV (respectivelyEV |     | )isanevictionsetwhosepage-offset |     |     |     |     |     |     |     |
| ------------------ | --- | -------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| a                  | ptr |                                  |     |     |     | 300 |     |     |     |
secret=1
| bitsarecompatiblewitha(respectivelyptr). |     |     |     |     |     |     | secret=0 |     |     |
| ---------------------------------------- | --- | --- | --- | --- | --- | --- | -------- | --- | --- |
250
Weproceedasfollows.First,theattackerwillplaceptr
200
| inbothaandb.Thisissothatdereferencestoacanoccur |     |     |     |     |     | ycneuqerF |     |     |     |
| ----------------------------------------------- | --- | --- | --- | --- | --- | --------- | --- | --- | --- |
150
regardlessofthesecretvalue.Next,theattackertestswhether
| eachcandidatecompoundset,denoted(EV |     |     | ,EV | ),canevict |     | 100 |     |     |     |
| ----------------------------------- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- |
a ptr
| both a and               | ptr’s target | by priming               | all lines in | EV ptr and |     | 50  |     |     |     |
| ------------------------ | ------------ | ------------------------ | ------------ | ---------- | --- | --- | --- | --- | --- |
| continuouslytraversingEV |              | ,andthenprobing/timingEV |              | .          |     |     |     |     |     |
|                          |              | a                        |              | ptr        |     | 0   |     |     |     |
If the probe results in an L2 miss, the target of ptr filled 600 625 650 675 700 725 750 775 800
Prime+Probe Latency (ticks)
thecacheanddisplacedalineinEV ptr .Thissimultaneously Figure9:Prime+Probelatencyofconstant-timeswapsubrou-
implies that EV evicted a because evicting a is the only tine.Ifptrshowsupina(secret=1),theattackerobservesa
a
highlatency;otherwise(secret=0)itobservesalowlatency.
waythattheDMPwouldhavedereferencedptr.Iftheprobe
| resultsinallL2hits,eitherEV |     |     | orEV werenoteviction |     |     |     |     |     |     |
| --------------------------- | --- | --- | -------------------- | --- | --- | --- | --- | --- | --- |
|                             |     | a   | ptr                  |     |     |     |     |     |     |
setsforaorptr,respectively.
Thecomplexityofcompoundevictionsetfindingispropor-
6 AttackingClassicalCryptography
tionaltothenumberofpossiblecandidates.Withtheknowl-
a ptr,the
| edge of page | offsets of | and | attacker | can reduce |                |      |          |                |     |
| ------------ | ---------- | --- | -------- | ---------- | -------------- | ---- | -------- | -------------- | --- |
|              |            |     |          |            | We demonstrate | that | Go’s RSA | implementation | and |
thenumberofpotentialL2setseachofthemmapstofrom
|                       |     |                          |     |     | OpenSSL’s | Diffie-Hellman | Key Exchange | (DHKE) | imple- |
| --------------------- | --- | ------------------------ | --- | --- | --------- | -------------- | ------------ | ------ | ------ |
| 8192to64.Meanwhile,EV |     | onlyneedstobeasupersetof |     |     |           |                |              |        |        |
a
thestandardeviction.Wegroup810standardevictionsetsas mentation,despitebeingconstant-time,canleaksecretsvia
|     |     |     |     |     | the DMP | side-channel. | Bothsystems | are otherwise | secure |
| --- | --- | --- | --- | --- | ------- | ------------- | ----------- | ------------- | ------ |
oneEV inourPoCs,whichleadsto512candidatesoverall.
| a   |     |     |     |     | againstmaliciousinputs,butfeaturesubroutinesthatactivate |     |     |     |     |
| --- | --- | --- | --- | --- | -------------------------------------------------------- | --- | --- | --- | --- |
Werunourcompoundevictionsetsconstructionalgorithm10
theDMPbasedonthesecretkey.Wedrawinspirationfrom
timesandthemeantimeforallL2evictionsetgenerationis
|     |     |     |     |     | priorchosen-ciphertextside-channelattacks |     |     | [4,5,9,19,20, |     |
| --- | --- | --- | --- | --- | ----------------------------------------- | --- | --- | ------------- | --- |
263.9seconds,while113.6secondsforfindingthecompound
39,40,48,53,61,90,93],andadaptthosetechniquesforthe
evictionset.Overall,wefindthatweareabletoreliablycon-
specificimplementationsconsideredinthissection.
structthesecompoundevictionsetsusingtheabove-described
technique,allowingustoproceedtousingtheDMPinorder
torecoversecretfromwithinct-swap’saddressspace. 6.1 Go’sRSAEncryption
OurtargetedRSAimplementationusesMontgomerymulti-
5.3 Proof-of-ConceptResults
plication,whichimplicitlyblindstheRSAsecretkeyexcept
| Withthecompoundevictionset(EV |     |     | ,EV )foraandptr’s |     |     |     |     |     |     |
| ----------------------------- | --- | --- | ----------------- | --- | --- | --- | --- | --- | --- |
a ptr 11BasedonSection4.2,theattackerhastoresetthehistoryfiltertoachieve
target in hand,we now demonstrate a proof-of-concept at- re-dereferencestothesameptr.InourPoC,whilethemethodsdiscussed
inSection4.2couldhelp,weexperimentallyfindthattheTCPsocketcode
10Thegroupsizeisnot“thebiggerthebetter”,sinceEVaneedstoevict usedinthevictimprocesstoreceiveinputsfromtheattackergeneratesa
arrayabeforethevictimloadsa. sufficientamountoftraffictoresetthehistoryfilter.
| USENIX Association |     |     |     |     |     | 33rd USENIX Security Symposium    1125 |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

|                                                         |     |     |     | // m = c ^ Dp mod                 | P         |           |
| ------------------------------------------------------- | --- | --- | --- | --------------------------------- | --------- | --------- |
| duringasingle,necessarymodularoperation.12Wefindthat    |     |     |     | 1                                 |           |           |
|                                                         |     |     |     | m = bigmod.NewNat().Exp(t0.Mod(c, | P), // t0 | = c mod P |
| anattackercancraftciphertextstoexploitthismodularopera- |     |     |     | 2                                 |           |           |
|                                                         |     |     |     | 3 priv.Precomputed.Dp.Bytes(),    | P)        |           |
tionandextractapartialRSAsecretkeybyobservingDMP 4 // m2 = c ^ Dq mod Q
activations.13 Theycan then use thestandardCoppersmith 5 m2 = bigmod.NewNat().Exp(t0.Mod(c, Q), // t0 = c mod Q
methodtorecovertheentireRSAsecretkey[30,71]. 6 priv.Precomputed.Dq.Bytes(), Q)
Go’sRSA(1.20+)encryptionoverview. RSAisapublic- Listing 3: DMP-vulnerable subroutine in Go’s RSA (1.20)
keycryptosystem.Go(1.20+)RSAimplementationfollows Decrypt.cistheattacker’schallengeciphertextthatcontains
the specification in RFC 8017 [63]. RSA has a public ex- a ptr.t functions as the AoP a in Section 5 becauset =
|     |     |     |     | 0   |     | 0   |
| --- | --- | --- | --- | --- | --- | --- |
ponent e (65537 in Go’s RSA). An RSA secret key con- cmod pwouldactivatetheDMPifandonlyifc<p.Attacker
| sistsoftwoprimes                | pandq,andanintegerd |                 | suchthated≡ |                                                      |     |     |
| ------------------------------- | ------------------- | --------------- | ----------- | ---------------------------------------------------- | --- | --- |
|                                 |                     |                 |             | canthenextract padaptivelybyobservingDMPactivations. |     |     |
| 1mod(p−1)(q−1).                 | AnRSApublickeyis(N  |                 | = p∗q,e).   |                                                      |     |     |
| Withoutlossofgeneralityweassume |                     | p>q.Go’sRSAuses |             |                                                      |     |     |
be1.Then,theattackersetstheremainingbitsofctobeall
theChineseremaindertheorem(CRT)toacceleratedecryp-
0,exceptthelower448bitsthatarefilledwith764-bitptrs.
tion.
The16bitsimmediatelybeforetheptrsarealwayssetto0
| PoC overview. | In our PoC, | we target | Go’s RSA-2048 |     |     |     |
| ------------- | ----------- | --------- | ------------- | --- | --- | --- |
andunused.
(1.20).Similarlyto[93],ourthreatmodelassumesthatthe
victim(server)generatesapairofstaticpublicandsecretkeys.
recovered
Theattackersendsaciphertexttothevictim,andthevictim 0···0 1 0···0 0···0 ptrs
prefix
decryptstheciphertextusingitssecretkey.14 ThepublicN 1024 n-1 560-n 16 448
| is2048bitslong,andthesecret |     | pandqareabout1024bits |     |     | n-th |     |
| --------------------------- | --- | --------------------- | --- | --- | ---- | --- |
Figure10:Challengeciphertextconstructiontoleakthen-th
| long.FactoringN | into pandqbreaksRSA-2048.InourPoC, |     |     |     |     |     |
| --------------- | ---------------------------------- | --- | --- | --- | --- | --- |
the attacker extracts the 560 most significant bits of p by mostsignificantbitofthe pinGo’sRSA-2048.
observingDMPactivations,andthenusestheCoppersmith
methodtobreakRSA-2048. Assumingp>q,iftheattackerobservesnoDMPactivation
DMP-vulnerable subroutine in Go’s RSA. Listing 3 fromt 0 =cmod p,theycanconcludethatc≥pandthen-th
showstheDMP-vulnerablesubroutineinGo’sRSADecrypt. bit of p is 0 with 1− 1 ≈1 probability. On the other
2576−n
DecrypttakesinanRSAsecretkeyandaciphertextc,and hand,iftheattackerobservestheDMPactivation,theycan
m=cd
outputs a plaintext modN. Due to CRT, Decrypt conclude that c< p and the n-th bit of p must be 1. Since
breaksthisexponentiationintotwo:m=cDp mod pandm= 1 becomesnon-negligibleasnapproaches576,westop
2576−n
cDq
| modq,whereD | p andD | q areCRT-relatedparameters. |     | theattackatn=560. |     |     |
| ----------- | ------ | --------------------------- | --- | ----------------- | --- | --- |
Thefirststepofm=cDp mod pistocomputet =cmod p. Experimentalresult. Wenowusethepreviousciphertext
0
Akeyobservationisthatifc<p,t remainsasc.Ontheother constructionstrategyandtheCoppersmithmethodtoextract
0
thefullRSAsecretkey.16Whentargetingeachofthe560top
| hand,ifc≥ p,t | becomesc−l∗p,whichisunpredictable |     |     |     |     |     |
| ------------- | --------------------------------- | --- | --- | --- | --- | --- |
0
becauselisanunknowninteger.Supposeccontainsaptr: bitsof p,wecollect32Prime+Probelatencydatapointsto
• Ifc<p,t 0 =ccontainstheptrandactivatestheDMP. mitigatebackgroundnoise.Themedianofthecollecteddata
• Ifc≥p,t ̸=cisrandomanddoesnotactivatetheDMP. isthencomparedtoaprofiledthreshold:742±38tickswhen
0
ctriggerstheDMPactivationversus664±124tickswhenc
| Inthiscase,wecanextract |     | pbitbybitbyobservingDMP |     |     |     |     |
| ----------------------- | --- | ----------------------- | --- | --- | --- | --- |
activationsresultingfromloadingt .Thisallowsustotreat doesnottrigger.Werepeattheexperimenttargetingbitnif
0
t astheAoPainSection5.15 thecollecteddataareoutliersduetosystemnoise.Theend-to-
0
Challengeciphertextconstruction. Next,weshowhow endattacktakes49minutesonaveragetofinish.Moredetails
to construct c to extract the 560 most significant bits of p aboutcompoundevictionsetgenerationandnoisetolerance
forGo’sRSAareinAppendixB.
(oneatatime).InFigure10,assumetheattackerhasalready
| recoveredthen−1mostsignificantbitsof |     |     | pandtargetsthe |     |     |     |
| ------------------------------------ | --- | --- | -------------- | --- | --- | --- |
n-thbit.Sincepis1024-bit,theattackersetstheleading1024
6.2 OpenSSLDiffie-HellmanKeyExchange
bitsofthe2048-bitctobe0.Theysetthenextn−1bitsof
ctobetherecoveredn−1bitsof p,andthen-thbitofcto Our targeted OpenSSL DHKE implementation utilizes a
window-basedexponentiationalgorithm.Thiscreatesavul-
12UpdatestoGo1.20cryptography:https://words.filippo.io/disp nerabilitygivenDMP:ifanattackercraftsamaliciouspublic
atches/go-1-20-cryptography/
keyandcorrectlyguessesthetargetwindowofthesecretkey,
13InSections6and7,DMPactivationparticularlyreferstoDMPderefer-
amultiplicationsubroutinewillgenerateaptrvalue.Theat-
encestotheattacker-chosenptr.
14TheattackdoesnotapplytotheRSAsignatureschemebecausesigna- tackercanthenexploitDMPactivationstoadaptivelyextract
turesarecalculatedass=hd modN,wherehisthemessagehash.Since theDHsecretkey.
hashisaone-wayfunction,theattackerdoesnothaveprecisecontrolofh.
15t0isaGobigmod.Natwhoseinternalrepresentationisanarrayof64-bit 16The implementation ofthe Coppersmithmethodusedbyourpaper:
integers(on64-bitmachine). https://github.com/mimoo/RSA-and-LLL-attacks
| 1126    33rd USENIX Security Symposium |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | ------------------ | --- |

Cryptography OnlineTime(minutes) OfflineTime(minutes) bn_mul_mont_fixed_topisDMP-vulnerable,andtmpisthe
|           |     | ➊ ➋  |     | ➌   |     |     | AoPainSection5. |          |        |      |     |     |     |     |
| --------- | --- | ---- | --- | --- | --- | --- | --------------- | -------- | ------ | ---- | --- | --- | --- | --- |
| RSA-2048  |     | 5 18 |     | 26  | ∼0  |     |                 |          |        |      |     |     |     |     |
|           |     |      |     |     |     |     | 1 // tmp        | = c^(s0) |        |      |     |     |     |     |
| DH-2048   |     | 5 6  |     | 127 | ∼0  |     |                 |          |        |      |     |     |     |     |
|           |     |      |     |     |     |     | 2 while (bits   | >        | 0) {   |      |     |     |     |     |
| Kyber-512 |     | 6 10 |     | 43  | 286 |     |                 |          |        |      |     |     |     |     |
|           |     |      |     |     |     |     | 3 for (i        | = 0;     | i < w; | i++) |     |     |     |     |
Dilithium-2 5 13 577 274 if (!bn_mul_mont_fixed_top(&tmp, &tmp, &tmp, mont, ctx))
4
|     |     |     |     |     |     |     | goto | err; |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ---- | ---- | --- | --- | --- | --- | --- | --- |
5
| Table 1: | Experimental |     | results | of four cryptographic |     | attack | // bits | -=  | w;  |     |     |     |     |     |
| -------- | ------------ | --- | ------- | --------------------- | --- | ------ | ------- | --- | --- | --- | --- | --- | --- | --- |
6
PoCs. We show the mean of three runs of each PoC. On- // tmp = tmp * c^(si)
7
| linetimereferstotherequiredtimeforaco-locatedattacker |     |     |     |     |     |     | }   |     |     |     |     |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
8
process,whichincludes➊standardevictionsetsgeneration;
Listing4:bn_mul_mont_fixed_topisourDMP-vulnerable
➋compoundevictionsetfinding;➌DMPleakage.Offline subroutineinOpenSSLDHKE(1.1.1q).Thesecretkeysis
|     |     |     |     |     |     |     | broken | into k | windows | s ∥s ∥...∥s |     | with a | window | size |
| --- | --- | --- | --- | --- | --- | --- | ------ | ------ | ------- | ----------- | --- | ------ | ------ | ---- |
time is the post-processing (e.g. lattice reduction) time to 0 1 k−1
completesecretkeyrecovery.Wedonotincludethetimefor w. An attackwho knows s ∥s ∥...∥s can guess s and
|     |     |     |     |     |     |     |     |     |     | 0 1 | i−2 |     |     | i−1 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
theofflinesignaturecollectionphaseofDilithium-2. constructcsuchthatiftheguessiscorrect,tmpcontainsptr
afterLine5.Theattackercanthenextractsadaptively.
| OpenSSLDHKE(1.1.1q)overview. |     |     |     |     | DHKEallowstwo |     |                                 |     |     |     |     |                  |     |     |
| ---------------------------- | --- | --- | --- | --- | ------------- | --- | ------------------------------- | --- | --- | --- | --- | ---------------- | --- | --- |
|                              |     |     |     |     |               |     | Challengepublickeyconstruction. |     |     |     |     | Next,weshowhowto |     |     |
parties,Alice andBob,to agree on a sharedsecretoveran constructthechallengepublickeyc.Allmultiplicationisin
| insecurechannel[34].Thepublicparametersareaprime |     |     |     |     |     | p   |     |     |     |     |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Montgomeryform,soeveryoperandispre-multipliedwith
andageneratorgthatgeneratesacyclicorder-qsubgroupof
apublicconstantR.Assumetheattackeralreadyrecovered
| Z∗.DHKErequires |     |     |     |     |     | p−1.17 |     |     |     |     |     |     |     |     |
| --------------- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
ptobeasafeprimesuchthatq=
p 2 s 0 ∥s 1 ∥...∥s i−2 .Totargets i−1 ,theattackermakesaguessof
| Al ice and                    | Bob | generate | their | own secret | keys x∈Z     | and |                                            |     |     |     |     |     |     |     |
| ----------------------------- | --- | -------- | ----- | ---------- | ------------ | --- | ------------------------------------------ | --- | --- | --- | --- | --- | --- | --- |
|                               |     |          |       |            |              | q   | itsvalueandconstructscbysolvingtheequation |     |     |     |     |     |     |     |
| y∈Z .Alicesendsherpublickeygx |     |          |       |            | ptoBobandBob |     |                                            |     |     |     |     |     |     |     |
| q                             |     |          |       | mod        |              |     |                                            |     |     |     |     |     |     |     |
(cs0∥s1∥...∥si−1)2w
sendshisgy mod ptoAlice.Theybothcomputetheshared ·R≡tmp mod p (1)
| secret(gx)y | mod | p=(gy)x | mod | p.ThesecurityofDHKEre- |     |     |     |     |     |     |     |     |     |     |
| ----------- | --- | ------- | --- | ---------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
wherethe2048-bitattacker-controlledoutputbuffertmpcon-
liesonthecomputationalDiffie–Hellman(CDH)assumption
| thatgivengx | mod | p,gy | mod p,andg,itiscomputationally |     |     |     | tainsaptr. |     |     |     |     |     |     |     |
| ----------- | --- | ---- | ------------------------------ | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
difficulttocomputegxy mod p. LetE denotetheexponent(s ∥s ∥...∥s )∗(2w).Westart
|              |     |                                        |     |     |     |     |                            |     |     | 0   | 1              | i−1 |     |     |
| ------------ | --- | -------------------------------------- | --- | --- | --- | --- | -------------------------- | --- | --- | --- | -------------- | --- | --- | --- |
|              |     |                                        |     |     |     |     | byassumingthattheexponentE |     |     |     | isanoddnumber. |     |     |     |
| PoCoverview. |     | Following[41,61],ourthreatmodelassumes |     |     |     |     |                            |     |     |     |                |     |     |     |
that the victim (server) and attacker (client) do a DH key IfE isanoddnumber,wefirstmoveRtotheright-hand
sideoftheequationbydoinganinverse:
exchange.Thevictim(server)generatesarandom2048-bit
| DHpublicparameter |     | pandsharesitwiththeattacker(client). |     |     |     |     |     |     |     |             |     |     |     |     |
| ----------------- | --- | ------------------------------------ | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- |
|                   |     |                                      |     |     |     |     |     |     | cE  | ≡R−1·tmpmod |     | p   |     | (2) |
Thevictimgeneratesitsownstaticsecretkeys.Theattacker
sendsachallengepublickeyctothevictim,whocomputes
|     |     |     |     |     |     |     | Since p | is a | safe prime, | gcd(p−1,E) |     | = 1 | (E is | odd), |
| --- | --- | --- | --- | --- | --- | --- | ------- | ---- | ----------- | ---------- | --- | --- | ----- | ----- |
cs mod pusingtheOpenSSLwindow-basedexponentiation.
|     |     |     |     |     |     |     | the modular | inverse |     | E−1 (E−1·E | ≡   | 1 mod(p−1)) |     | ex- |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ------- | --- | ---------- | --- | ----------- | --- | --- |
Thewindowsizewis6.Theattackerextractsswindowafter ists due to Fermat’s little theorem,and c can be solved as
| windowbyobservingDMPactivations. |     |     |     |     |     |     | (tmp·R−1)E−1 |     |     |     |     |     |     |     |
| -------------------------------- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
mod p[39].
DMP-vulnerablesubroutineinOpenSSLDHKE. The However,E =(s ∥s ∥...∥s )∗(2w) is an even number.
|               |         |            |         |                    |        |            |                                                |        | 0       | 1 i−1   |         |          |     |        |
| ------------- | ------- | ---------- | ------- | ------------------ | ------ | ---------- | ---------------------------------------------- | ------ | ------- | ------- | ------- | -------- | --- | ------ |
| victim breaks | s       | into k     | windows | s ∥s               | ∥...∥s | with each  |                                                |        |         |         |         |          |     |        |
|               |         |            |         | 0 1                | k−1    |            | Anevennumbercanbefactorizedasanoddnumbermulti- |        |         |         |         |          |     |        |
| window        | of size | w. Listing | 4       | shows a simplified |        | version of |                                                |        |         |         |         |          |     |        |
|               |         |            |         |                    |        |            | plied by                                       | 2n. In | orderto | convert | an even | numberto |     | an odd |
cs
the algorithm thatcomputes mod p window by window, number,we need to eliminate the 2n. Tonelli-Shanks algo-
| where we | replace | most | of the | code with | descriptive | com- |     |     |     |     |     |     |     |     |
| -------- | ------- | ---- | ------ | --------- | ----------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
rithmexplainshowtocalculatethemodularsquarerootfor
ments and only highlight the DMP-vulnerable subroutine n times,but only half of the elements in Z∗ are quadratic
p
| bn_mul_mont_fixed_top. |     |     | To  | start with,a | variable | tmp is |     |     |     |     |     |     |     |     |
| ---------------------- | --- | --- | --- | ------------ | -------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
residues,whichmeansthatagivennumbermightnothave
initializedtocs0.Attheendofeachwhileloopiterationi(i
recursivemodularsquarerootsofdepth-n[74,81].Thisprob-
startsfrom1),tmp=cs0∥...∥si.
|     |     |     |     |     |     |     | lem can | be overcome |     | because | tmp has | 32 64-bit | elements |     |
| --- | --- | --- | --- | --- | --- | --- | ------- | ----------- | --- | ------- | ------- | --------- | -------- | --- |
Duringiterationi,aninvariantholdsafterLine5:tmp=
andonlyoneneedstobetheptr.Wecanadjustanyofthe
(cs0∥s1∥...∥si−1)2w
.Iftheattackeralreadyrecoveredtheprefix other3164-bitelementstoensurethattmp·R−1 mod phas
s ∥s ∥...∥s ,they can guess s and construct c strategi- recursivemodularsquarerootsofdepth-n.Oncethe2nfactor
| 0 1                 | i−2 |                |     | i−1         |     |           |                                 |     |     |     |     |                    |     |     |
| ------------------- | --- | -------------- | --- | ----------- | --- | --------- | ------------------------------- | --- | --- | --- | --- | ------------------ | --- | --- |
| cally. Iftheirguess |     | is correct,tmp |     | willcontain | a   | ptr after |                                 |     |     |     |     |                    |     |     |
|                     |     |                |     |             |     |           | iseliminated,wecanapplytheodd-E |     |     |     |     | caseoutlinedabove. |     |     |
Line 5, triggering the DMP. Consequently, the subroutine Experimentalresult. Foratargetwindowi,thereareonly
2w(64)possiblevaluesofs.Foreachguessofs,wecollect
|     |     |     |     |     |     |     |     |     |     | i   |     |     | i   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
17WhyDiffie-Hellmanpreferssafeprimes:https://www.johndcook.c
32Prime+Probelatencydatapointstomitigatebackground
om/blog/2017/01/12/safe-primes-sylow-theorems-and-cryptog
noise.Werepeatanexperimentifthecollecteddatacontains
raphy/
| USENIX Association |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1127 |     |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- | --- |

outliersduetosystemnoise.Wecomparethemedianofour uponaPKEschemecalledKyber.CPAPKE,whichischosen-
collecteddatawithaprofiledthresholdtodetermineifour plaintextsecure(CPA-secure).KyberisaFujisaki-Okamoto
guessofs i iscorrect.Aftertestingall64values,weexpectto (FO)transformationoftheunderlyingKyber.CPAPKE,which
observe1highPrime+Probelatency(correctguess)and63 turnsaCPA-securePKEintoaIND-CCA2-secureKEM[38].
lowPrime+Probelatencies(incorrectguesses).Ifthenumber Kyber.CPAPKEkeygenerationsamplesthesecretkeys,e∈
ofpositiveandnegativemeasurementsdeviatesfromthis,we Rk fromB ,withη beingasmallinteger.Thepublickey
|     |     |     |     |     |     |     |     | q   | η1  |     | 1   |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
redo the experiment for this window. When the challenge consistsoft∈RkandarandomA∈Rk×k,wheret=As+e.19
|     |     |     |     |     |     |     |     |     |     | q   |     | q   |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
publickeyc triggers the DMP,the Prime+Probe latencyis LeakingeithersorebreaksKyber.
701±65ticks,comparedto641±10whenitdoesnot.The
Kyber.CPAPKEencryptiontakesinthepublickey(t,A),a
experimenttakes2.3hourstocomplete,andweextractthe
256-bitmessageM,andaseedrasthesourceofrandomness.
victimsecretkeys.AppendixBprovidesfurtherdetailsabout M = M M ...M is converted to a polynomial m ∈ R ,
|     |     |     |     |     |     |     |     |     | 0 1 | 255 |     |     |     | p   | q   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
c o m p o u nd e v ic tio n set generation and noise tolerance for ∗⌈q
|        |         |         |     |     |     |     |     | where | m (x)=∑2 |     | 55 M  | ⌋∗xi. Then,it |         | samples r∈Rk |     |
| ------ | ------- | ------- | --- | --- | --- | --- | --- | ----- | -------- | --- | ----- | ------------- | ------- | ------------ | --- |
|        |         |         |     |     |     |     |     |       | p        | i=  | 0 i   | 2             |         |              | q   |
| O p en | S S L D | H K E . |     |     |     |     |     |       |          | ∈Rk |       |               |         |              |     |
|        |         |         |     |     |     |     |     | fromB | η1 ,e    | 1   | fromB | η2 ,ande 2 ∈R | q fromB | η2 ,withη    | 1   |
q
|     |     |     |     |     |     |     |     | andη | beingsmallintegers.Itcomputesu=ATr+e |     |     |     |     |     | ,and |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ------------------------------------ | --- | --- | --- | --- | --- | ---- |
|     |     |     |     |     |     |     |     |      | 2                                    |     |     |     |     |     | 1    |
v=tTr+e
7 AttackingPost-QuantumCryptography 2 +m p .Theciphertextis(u,v).
|                                                 |     |     |     |     |     |     |     | Kyber.CPAPKE                           |     |     | decryption | takes in | the ciphertext |       | (u,v), |
| ----------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | ---------- | -------- | -------------- | ----- | ------ |
|                                                 |     |     |     |     |     |     |     | andthesecretkey(s,e).Itcomputesv−sTu=m |     |     |            |          |                | +eTr+ |        |
| WedemonstratethattheimplementationoftwoCRYSTALS |     |     |     |     |     |     |     |                                        |     |     |            |          |                | p     |        |
cryptographic primitives,Kyber and Dilithium,though de- e −sTe .Coefficientsinm areeither0or⌈q ⌋.Coefficients
|                                                     |     |     |     |     |     |     |     | 2       | 1    |                                        |     | p   |     | 2   |     |
| --------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ------- | ---- | -------------------------------------- | --- | --- | --- | --- | --- |
|                                                     |     |     |     |     |     |     |     | ineTr+e | −sTe | aresmallintegers.Decryptionrecoversthe |     |     |     |     |     |
| signedtobeconstant-time,canleaksecretsviatheDMPside |     |     |     |     |     |     |     |         | 2    | 1                                      |     |     |     |     |     |
channel.18 Kyber is an IND-CCA2-secure (secure against plaintextMbyroundingeachcoefficientofv−sTuto1ifthe
coefficientiscloserto⌈q⌋thanto0;andto0otherwise.
| adaptivechosen-ciphertextattack)NIST-selectedkeyencap- |     |     |     |     |     |     |     |     |     |     | 2   |     |     |     |     |
| ------------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
sulationmechanism(KEM)[12].DilithiumisaNIST-selected Decryptionfailureoccurswithnegligibleprobabilitywhen
digitalsignaturescheme[57].BothKyberandDilithiumrely processingnormalciphertexts.LetM′denotethedecrypted
′̸=M
onthehardnessofModule-LWE(MLWE). plaintext.Ifdecryptionfails,resultinginM i i (thei-thbit
R (Z[x]/xn+1). R isflipped),thishappensonlyifthei-thcoefficientoftheerror
| Notation:                  |                            | denotes    | the | ring    |                    | q denotes   |     |                                |     |                                        |         |     |         |     |     |
| -------------------------- | -------------------------- | ---------- | --- | ------- | ------------------ | ----------- | --- | ------------------------------ | --- | -------------------------------------- | ------- | --- | ------- | --- | --- |
|                            | (Z                         |            |     |         |                    |             |     | vector(eTr+e                   |     | −sTe                                   | )[i]≥⌈q | ⌋.  |         |     |     |
| the ring                   |                            | [x]/xn+1). | Rk  | denotes | the space          | of length-k |     |                                |     | 2                                      | 1       | 4   |         |     |     |
|                            |                            | q          | q   |         |                    |             |     |                                |     |                                        |         |     |         |     |     |
|                            |                            |            |     | .Rk×l   |                    |             |     | PoCoverview.                   |     | WetargettheKyber-512referenceimplemen- |         |     |         |     |     |
| vectorswhoseelementsareinR |                            |            |     | q       | denotesthespaceof  |             |     |                                |     |                                        |         |     |         |     |     |
|                            |                            |            |     | q       |                    |             |     | tation,wheren=256,q=3329,k=2,η |     |                                        |         |     | =3,andη |     | =2. |
| k×l                        | matriceswhoseelementsarein |            |     |         | R . Forapolynomial |             |     |                                |     |                                        |         |     | 1       |     | 2   |
q
p, p[i] denotes the i-thcoefficientof p. Fora vectorv,v[i] Ourthreatmodelassumesthatthevictim(server)andattacker
(client)wanttoderiveasharedsecretusingKyber.Thevic-
| denotesthei-thpolynomialofv,andv[i][j]denotesthe |     |     |     |     |     |     | j-th |     |     |     |     |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
coefficientofv[i].Foravectorv∈Rk (ormatrixA∈Rk×l), tim(server)generatesapairofstaticKybersecretandpublic
|     |     |     |     |     | q   |     | q   |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
keys.Thesecretshastwo(k=2)polynomials,eachwith256
| vT (or | AT) | denotes | its transpose. | ⌈x⌋ | denotes | rounding | x   |     |     |     |     |     |     |     |     |
| ------ | --- | ------- | -------------- | --- | ------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
coefficients.Theattackerguessesavaluefors[i][j]andthen
| to the | closest | integer, | rounding | up in | the case | of ties. | B   |     |     |     |     |     |     |     |     |
| ------ | ------- | -------- | -------- | ----- | -------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
η
craftsaplaintextMcontainingaptr.TheyencryptMusing
| and | S η denote | the centered |     | binomial | and uniform | random |     |     |     |     |     |     |     |     |     |
| --- | ---------- | ------------ | --- | -------- | ----------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
thevictim’spublickeyandsendtheciphertexttothevictim
| distributionrespectively.AnumbersampledfromB |     |     |     |     |     | orS       | is  |                |     |     |     |     |     |     |     |
| -------------------------------------------- | --- | --- | --- | --- | --- | --------- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- |
|                                              |     |     |     |     |     | η         | η   |                |     |     |     |     |     |     |     |
| withintherange[−η,η].Whenwesaythatv∈Rk       |     |     |     |     |     | issampled |     | fordecryption. |     |     |     |     |     |     |     |
q
|       |            |                                          |      |              |     |        |        | DMP-vulnerable                                    |           |             | subroutine | in Kyber.     |            | Kyber’s | DMP- |
| ----- | ---------- | ---------------------------------------- | ---- | ------------ | --- | ------ | ------ | ------------------------------------------------- | --------- | ----------- | ---------- | ------------- | ---------- | ------- | ---- |
| fromB | (S         | ),wemeanthateachcoefficientofpolynomials |      |              |     |        |        |                                                   |           |             |            |               |            |         |      |
|       | η          | η                                        |      |              |     |        |        | vulnerablesubroutineisindcpa_dec,theCPAPKEdecryp- |           |             |            |               |            |         |      |
| in v  | is sampled | from                                     | B (S | ). B denotes | the | set of | sparse |                                                   |           |             |            |               |            |         |      |
|       |            |                                          | η η  | τ            |     |        |        |                                                   |           |             |            |               |            |         |      |
|       |            |                                          |      |              |     |        |        | tion                                              | function. | It decrypts |            | the challenge | ciphertext | that    | en-  |
polynomialsinRwhereτcoefficientsareeither−1or1and
cryptsaplaintextMcontainingaptr,andstoresthedecrypted
therestare0.
M′intoabufferbuf.Ifthedecryptionissuccessful,M′=M
andbufcontainsptr.Otherwise,M′̸=Mandbufdoesnot
| 7.1 | Kyber |     |     |     |     |     |     | containptr. |     |     |     |     |     |     |     |
| --- | ----- | --- | --- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- |
KyberisCCAsecure.Decapsulationwouldrejectamal-
Kyberdecapsulationreliesonadecryptionsubroutine.De- formedciphertextwithoutexposing M′ =M orM′ ̸=M to
| cryption | failure | leaks | the Kybersecretkey |     |     | [35,66–68,75]. |     |     |     |     |     |     |     |     |     |
| -------- | ------- | ----- | ------------------ | --- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
theattacker.However,theattackercanlearndecryptionfail-
| While | Kyberdoes | notexpose |     | decryption | failures | to  | the at- |     |     |     |     |     |     |     |     |
| ----- | --------- | --------- | --- | ---------- | -------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
ureorsuccessbyobservingwhetherptrisdereferencedby
tacker,theattackercanusetheDMPsidechanneltoconstruct
theDMP.Thisbehaviorisnotanimplementationissuebut
adecryptionfailureoracleandthenextractthesecretkey.
|     |     |     |     |     |     |     |     | fundamental |     | to the | FO transform. | As  | a result, | subroutine |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ------ | ------------- | --- | --------- | ---------- | --- |
Kyber overview. A KEM uses a public key encryption indcpa_decisDMP-vulnerableandbufistheAoPainSec-
(PKE)schemetosecuresymmetrickeymaterial.Kyberbuilds
tion5.
18CRYSTALS:CryptographicSuiteforAlgebraicLatticeshttps://pq 19ThesecuritylevelofKyberscaleswithk.AMLWEmatrixfromRk×kis
q
| -crystals.org/index.shtml              |     |     |     |     |     |     |     | analogoustoank×nkmatrixinLWE. |     |     |     |     |                    |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ----------------------------- | --- | --- | --- | --- | ------------------ | --- | --- |
| 1128    33rd USENIX Security Symposium |     |     |     |     |     |     |     |                               |     |     |     |     | USENIX Association |     |     |

Challengeciphertextconstruction. Wedemonstratehow Experimentalresult. InourPoC,thereare392recoverable
toconstructaciphertext(u,v)thatallowstheattackertobuild secretcoefficients. Weconstruct8challengeciphertextsto
adecryptionfailureoracleusingDMPactivations.Recall: adaptivelylearneachcoefficient,asitspotentialvalueranges
from-3to3.See[25]forwhyweneed8ciphertexts.Foreach
u=ATr+e
|     |     |     |     | 1   |     |     | ciphertext,we | collect | 32 Prime+Probe | latency | data points |
| --- | --- | --- | --- | --- | --- | --- | ------------- | ------- | -------------- | ------- | ----------- |
v=tTr+e
+m (3) to mitigate backgroundnoise. We repeatthe experimentif
|     |     |     |     | 2   | p   |     |          |            |                   |     |                  |
| --- | --- | --- | --- | --- | --- | --- | -------- | ---------- | ----------------- | --- | ---------------- |
|     |     |     |     |     |     |     | the data | we collect | contains outliers | due | to system noise. |
Supposetheattackerattemptstolearnthefirstcoefficientof
Wecomparethemedianofourcollecteddatawithaprofiled
thefirstpolynomialins:s[0][0].TheyprepareaplaintextM
thresholdtodeterminetheactivationstatusoftheDMP.When
withaptrinM 0...63 andfilltherestwith0s:M=ptr||00...00. theciphertexttriggerstheDMP(decryptionsucceeds),the
Theymanipulateothervariablestoensurethefollowing:if
Prime+Probelatencyis713±22ticks,comparedto616±14
0<s[0][0],(u,v)decryptstoM;otherwise,itdecryptstoM′
|                           |     |     |     |                           |     |     | ticks when                                        | it does not | (decryption | fails). | The experiment |
| ------------------------- | --- | --- | --- | ------------------------- | --- | --- | ------------------------------------------------- | ----------- | ----------- | ------- | -------------- |
| withthefirstbitflipped(M′ |     |     |     | =M ⊕1).Toachievethis,they |     |     |                                                   |             |             |         |                |
|                           |     |     | 0   | 0                         |     |     | takes59minutestocomplete.Afterthat,wespendanother |             |             |         |                |
cansetr=(0,0)(alength-2vectorofdegree-0polynomials),
|     |     |     |     |     |     |     | 5 hours | on lattice reduction | to extractthe |     | entire secretkey. |
| --- | --- | --- | --- | --- | --- | --- | ------- | -------------------- | ------------- | --- | ----------------- |
=⌈q⌋(adegree-0polynomial),ande
e 2 1 =(1,0).Thisresults More details about compound eviction set generation and
4
| inaciphertextofu=(1,0),v=m |     |     |     | +e  | .   |     |                                       |     |     |     |     |
| -------------------------- | --- | --- | --- | --- | --- | --- | ------------------------------------- | --- | --- | --- | --- |
|                            |     |     |     | p   | 2   |     | noisetoleranceforKyberareinAppendixB. |     |     |     |     |
Decryptioncomputesv−sTu:
|     | v−sTu=m |     | +e  | −sTe |     |     | 7.2 Dilithium |     |     |     |     |
| --- | ------- | --- | --- | ---- | --- | --- | ------------- | --- | --- | --- | --- |
|     |         |     | p   | 2 1  |     |     |               |     |     |     |     |
−(s[0],s[1])T(1,0)
=m p +e 2 Dilithiumreliesonthe"Fiat-ShamirwithAborts"[57],and
q
=m +⌈ ⌋−s[0] its security depends on the privacy of its nonce y [56].
|     |     |     | p   |     |     | (4) |                                                       |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------- | --- | --- | --- | --- |
|     |     |     |     | 4   |     |     | Dilithiumissecureagainstchosen-messageattacks,meaning |     |     |     |     |
apolynomial-timeattackercannotlearnsecretinformation
| Thefirstentryofv−sTuism |                |            |                     | [0]+⌈q⌋−s[0][0],contain- |                |     |                                                 |             |                   |     |              |
| ----------------------- | -------------- | ---------- | ------------------- | ------------------------ | -------------- | --- | ----------------------------------------------- | ----------- | ----------------- | --- | ------------ |
|                         |                |            |                     | p                        | 4              |     |                                                 |             |                   |     |              |
|                         |                |            |                     |                          | ⌈q⌋−s[0][0].   |     | by observing                                    | signatures. | However,Dilithium |     | might gener- |
| ing                     | a deliberately | introduced |                     | la rge erro              | r              | De- |                                                 |             |                   |     |              |
|                         |                |            |                     |                          | 4              |     | atedatainythatresemblesapointer.BymonitoringDMP |             |                   |     |              |
| cryption                | would          | fail       | if ⌈q⌋−s[0][0]≥⌈q⌋. |                          | The ciphertext |     |                                                 |             |                   |     |              |
|                         |                |            | 4                   |                          | 4              |     |                                                 |             |                   |     |              |
activations,anattackercouldobtainknowledgeofy,derive
constructionensuresthatdecryptionfailuredependsonthe
linearequationsinvolvingthesecretkey,andultimatelyex-
valueofs[0][0]:
tracttheentiresecretkey.Priorresearchhasexploredsimilar
|     | ⌈q⌋ |     | ⌈q⌋ |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
• If − s[0][0] < (0 < s[0][0]), M′ = M = attacksthatexploitsidechannelstolearn intermediateval-
|     | 4   |     |     | 4   |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ptr||00...00,andbufactivatestheDMP. uesduringDilithiumsigning,allowingsecretkeyreconstruc-
tion[15,28,49,58,77,87].
• If⌈q⌋−s[0][0]≥⌈q⌋(0≥s[0][0]),M′̸=Mbecausethe
|     | 4   |     | 4   |                          |     |     | Dilithium | overview. | Dilithium | key generation | samples |
| --- | --- | --- | --- | ------------------------ | --- | --- | --------- | --------- | --------- | -------------- | ------- |
|     |     |     | ′   | ⊕1),andbufcannotactivate |     |     |           |           |           |                |         |
firstbitisflipped(M =M 0 the secret key s ∈Rl from S and s ∈Rk from S ,with
|     |     |     | 0   |     |     |     |     | 1   | q η | 2   | q η |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
theDMP.
|     |     |     |     |     |     |     | η being | a small integer. | The public | key consists | of t∈Rk |
| --- | --- | --- | --- | --- | --- | --- | ------- | ---------------- | ---------- | ------------ | ------- |
q
Theattackercanlearntheexactvalueofs[0][0]bytuning andarandomA∈Rk×l,wheret=As +s .Leakingeither
|     |     |     |     |     |     |     |     |     | q   | 1   | 2   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
e andobservingDMPactivations.TotriggerDMPactivation s or s breaks Dilithium. Dilithium also has a public key
| 2   |     |     |     |     |     |     | 1 2 |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
onbuf(e.g.,evictionsetconstruction),weemploythesame compression,whichwediscussin[25].
methodasthechosen-inputattackfromSection5.1. Dilithiumsignaturegenerationusesrejectionsamplingto
We now present a simplified version of our attack. Ky- generatedigitalsignatures[56].InAlgorithm1wepresent
berincludesanextracompressionanddecompressionstep. asimplifiedversionthatfocusesonthepartrelevanttoour
In [25],we detailhowto overcome the compression when attack.Thealgorithmgeneratesasignature(z,c)ofamessage
constructingthechallengeciphertext. Musingthesecretkeys 1 .zisinitializedto⊥(Line2).Ina
Thesecretkeyhas256×2=512coefficients.Ideally,the whileloop,thealgorithmsamplesaprivatenoncey,whichis
alength-lvectorofpolynomialswithcoefficientsrandomly
attackershouldbeabletoapplythesameprocessaboveto
targetanycoefficientofs.However,duetofindingsinSec- sampledfrom[−γ ,γ ](Line4).Then,thealgorithmsamples
1 1
tionSection4.3,theDMPcannotleakeverycoefficientofs:If a random c from B ,and c depends on M (Line 5). c is a
τ
webreaksintochunksof64,theleading8andtrailing7bits sparsepolynomialwithexactlyτnumberof1or-1,andthe
arenotrecoverableviatheDMP.Asaresult,392outof512 non-zeroentrieshaverandomizedpositions.Thealgorithm
coefficientscanberecoveredbyobservingDMPactivations. computesz=y+cs (Line6),butwillonlyaccept(z,c)ifit
1
Wefeedtherecovered392coefficientsas392hintsintothe leaksnosecrets,andreject(z,c)otherwise.Notethatymust
latticereduction toolfrom Mayetal.,torecovertheentire be kept private. Leaking y leaks s = z−y. Leaking partial
|     |     |     |     |     |     |     |     |     |     | 1 c |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
secretkey[59].In[25],weprovidemoredetailsaboutwhy informationofymightalsocompromises 1 [18].
certaincoefficientsarenotrecoverable,andhowweusethe PoC overview. The victim (a Dilithium signing server)
latticereductiontool. generatesapairofDilithiumsecretandpublickeys.Ourthreat
| USENIX Association |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1129 |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

Sign(s ,M) y[0][1]) and z (z[0][0],z[0][1]) as an example. Assume that
1 1
z[0][1]∥z[0][0]formsavalid64-bitptr,pointingtothesame
2 z:=⊥
4GByteregionwhereylives.Ifwebreakptrintotwo32-bit
whilez=⊥do
3
ℓ ha lv e s (p t r ∥ p t r ) , th e n z [ 0 ] [1 ] = p t r a n dz[0][0]=ptr .
| 4   | y←S | // A | length-l | vector | of  | random |     | 1 0 |     |     | 1   |     | 0   |
| --- | --- | ---- | -------- | ------ | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
γ 1 W e c a n d e ri ve t h e r a n g e o f ( y [ 0 ][ 0 ],y [ 0 ][ 1 ]) :
|     | and   | small       | polynomials |            |            |     |     |     |     |     |     |     |     |
| --- | ----- | ----------- | ----------- | ---------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
| 5   | c∈B τ | // A sparse |             | polynomial | (depending |     |     |     |     |     |     |     |     |
on M) with τ number of 1 or -1 y[0][1]=z[0][1]−cs [0][1]∈[ptr −78,ptr +78]
|     |     |     |     |     |     |     |     |     | 1   |     | 1   | 1   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
z:=y+cs
| 6   |             | 1   |       |         |            |     | y[0][0]=z[0][0]−cs |     | [0][0]∈[ptr |     | −78,ptr | +78] | (5) |
| --- | ----------- | --- | ----- | ------- | ---------- | --- | ------------------ | --- | ----------- | --- | ------- | ---- | --- |
|     |             |     |       |         |            |     |                    |     | 1           |     | 0       | 0    |     |
|     | // Reject   | z   | (set  | z to ⊥) | if z leaks |     |                    |     |             |     |         |      |     |
|     | information |     | about | the     | secret     | key |                    |     |             |     |         |      |     |
ThetakeawayfromEquation(5)isthatifz[0][1]∥z[0][0]isa
7 return(z,c)
ptr,y[0][1]∥y[0][0]mightalsobeaptr!
end
8
|           |     |        |           |        |           |          | To elaborate, | we              | know z[0][1] | ∥    | z[0][0] forms | a    | ptr. If |
| --------- | --- | ------ | --------- | ------ | --------- | -------- | ------------- | --------------- | ------------ | ---- | ------------- | ---- | ------- |
| Algorithm |     | 1: The | main body | of the | Dilithium | sign al- |               |                 |              |      | ptr,we        |      |         |
|           |     |        |           |        |           |          | we want       | y[0][1]∥y[0][0] | to also      | form | a             | only | need    |
gorithmisawhileloopthatcreatesasignature(z,c)of y[0][1]=z[0][1]orcs [0][1]=0.Thevalueofy[0][0]isless
1
messageMunderthesecretkeys .Thealgorithmreturns importantbecausecs [0][0]issmall,variationsinwhichwill
|     |     |     |     | 1   |     |     |     |     | 1   |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
(z,c)ifitdoesnotleakanysecretinformation.
onlycausey[0][1]∥y[0][0]tomaptothesameoranadjacent
|     |     |     |     |     |     |     | cachelineasptr.Asaresult,z=y+cs |     |     |     | isDMP-vulnerable. |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------- | --- | --- | --- | ----------------- | --- | --- |
1
IftheattackersetsyastheAoPafromSection5,theycan
modelassumesthatthevictimisasigningserver.Theattacker
learncs [0][1]=0byobservingDMPactivations.Thesame
1
canchoosearbitrarymessagesandrequestdigitalsignatures ideaappliestoallothercoefficientsofyandz.
fromthevictim.Theattackercanparsethesignaturesoffline
|     |     |     |     |     |     |     | OfflineandOnlinesignaturecollection |     |     |     | Intheofflinephase, |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------------------------------- | --- | --- | --- | ------------------ | --- | --- |
andreplaycertainmessageslater.
theattackersendsmrandommessagesforsignatures.Recall
| We  | target | CIRCL’s | implementation |     | of  | deterministic |     |     |     |     |     |     |     |
| --- | ------ | ------- | -------------- | --- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- |
Dilithium-2 (written in Go),where n=256,q=8380417, thatz is a length-4 vectorof256-degree polynomials. The
=217,η=2,andτ=39 [37].20 attackercollectsm′pairsof{(z,c),M}whereforani∈[0,3]
| k=l | =4,γ 1 |     |     |     |     | Dilithium is |     |     |     |     |     |     |     |
| --- | ------ | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
deterministic when the private nonce y in Algorithm 1 is andaneven j∈[0,255],z[i][j+1]∥z[i][j]formsaptr,which
livesinthesame4GByteregionasy.21
generatedwithdeterministicrandomness.Ourattackismoti-
vatedbytheobservation:theservermightnaturallyproduce Intheonlinephase,theattackerre-submitsthem′messages
datathatresemblesaptriny.Whiletheexactvalueofy collectedoffline.IftheattackerobservesaDMPactivation,
shouldremainsecret,theunderlyingMLWEstructureallows theattackercandeducethaty[i][j+1]=z[i][j+1],andderive
anattackertoapproximateythroughz.Ifaptrappearsinz, onelinearequationofs :cs [i][j+1]=0.Aftergatheringat
1 1
theattackerinfersitspresencewithinyandconfirmsthisby
least876linearequations,theattackerusesthelatticereduc-
observingDMPactivations.Successfulconfirmationreveals tiontooltorecovers [59].
1
| partialknowledgeofs |     |     | .   |     |     |     |                                                  |     |     |     |     |     |     |
| ------------------- | --- | --- | --- | --- | --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- | --- |
|                     |     |     | 1   |     |     |     | In[25],wediscussmoredetailsaboutourPoCincludinga |     |     |     |     |     |     |
Our PoC consists of an offline and online signature col- theoreticalboundofmandm′,andhowtoloosesomecondi-
lectionphase.Duringtheofflinephase,theattackersendsm
tionsaboveforthepracticalityofthePoC.
messagetotheserverrequestingsignaturesandcollectsm′
InourPoC,werequestm=4×109
pairsof{(z,c),M},wherezcontainsaptr.Duringtheonline Experimentalresult.
phase,theattackerre-submitsthecollectedm′ messagesto messagesduringtheofflinecollectionphase. Weparsethe
|     |     |     |     |     |     |     | signatures | andcollectm′ | =3×105 |     | ones withthe | property |     |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ------------ | ------ | --- | ------------ | -------- | --- |
theserverforsignatures.Theattackercandistinguishwhich
pair{(z,c),M}causestheptrtoshowupinyviaDMPacti- thatz[i][j+1]∥z[i][j]formsaptr.Intheonlinephase,resub-
vations,andthenderivealinearequationofs .Theattacker mittingthem′messages,weobserveaPrime+Probelatency
1
furtherusesthelatticereductiontooltorecovers [59]. of772±152tickswhenthemessagetriggerstheDMP,com-
1
DMP-vulnerablesubroutineinCIRCLDilithium. The paredto657±106tickswhenitdoesnot.Tominimizefalse
positives,weacceptamessageastriggeringtheDMPonly
| DMP-vulnerablesubroutineisthez=y+cs |     |     |     |     | 1   | inAlgorithm1 |     |     |     |     |     |     |     |
| ----------------------------------- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
Sign.CIRCLusesanarrayofunsigned32-bitintegerstorep- afterobserving 10 consecutive positive signals. The entire
resentapolynomial.Everycoefficientofyandzisstoredas experimenttakes10hours.Anadditional5hoursarespent
anunsigned32-bitinteger.WepickyastheAoPafromSec- onlatticereductiontoextractthefullsecretkey.Moredetails
tion5.Therangeofcoefficientsinyis[−217,217],andthat aboutcompoundevictionsetgenerationandnoisetolerance
ofcoefficientsincs is[−78,78]. forCIRCLDilithiumareinAppendixB.
1
| Let’s | take | the first | two 32-bit | coefficients |     | of y (y[0][0], |     |     |     |     |     |     |     |
| ----- | ---- | --------- | ---------- | ------------ | --- | -------------- | --- | --- | --- | --- | --- | --- | --- |
20CIRCL: Cloudflare’s Interoperable Reusable CryptographicLibrary 21Bothbaseaddressesofzandyare64-bitaligned,sothatentriesateven
https://github.com/cloudflare/circl/ indexesare64-bitalignedaswell.
| 1130    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

8 Countermeasures ifthe attackerhas the ability to secret-dependently write a
pointertomemory,theDMPenablesittolearnpartialorcom-
Thispaperdemonstratesthatinformationdisclosurethrough
|     |     |     |     |     |     |     | plete information | about | that | secret. While | we  | demonstrate |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | ----- | ---- | ------------- | --- | ----------- |
the Apple m-series DMP is significantlygreaterthan previ- end-to-end attacks on fourcryptographic implementations,
ouslybelieved,andputsconstant-timecryptographyatrisk. moreprogramsarelikelyatriskgivensimilarattackstrategies.
AdrasticsolutionwouldbetocompletelydisabletheDMP. GivenourfindingsthatDMPsalsoexistontheAppleM2/M3
However,as doing so will incurheavy performance penal- andIntel13thGenerationCPUs,theproblemseeminglytran-
tiesandislikelynotpossibleonM1andM2CPUs,22inthis
|     |     |     |     |     |     |     | scends specific | processors |     | and hardware | vendors | and thus |
| --- | --- | --- | --- | --- | --- | --- | --------------- | ---------- | --- | ------------ | ------- | -------- |
sectionwediscussalternativedefensiveapproaches. requiresdedicatedhardwarecountermeasures.
| UsingEfficiencyCores. |     |     | AspointedoutbyAugury[84], |     |     |     |     |     |     |     |     |     |
| --------------------- | --- | --- | ------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
theDMPdoesnotactivateoncoderunningonIcestormcores.
Acknowledgments
Thus,asensibleshort-termsecuritypostureistorunallcryp-
| tographic | code | on Icestorm | cores. | This strategy |     | is simple, |     |     |     |     |     |     |
| --------- | ---- | ----------- | ------ | ------------- | --- | ---------- | --- | --- | --- | --- | --- | --- |
ThisworkwaspartiallysupportedbytheAirForceOfficeof
| general,and | does | not require | user | code changes. |     | Yet,it is |     |     |     |     |     |     |
| ----------- | ---- | ----------- | ---- | ------------- | --- | --------- | --- | --- | --- | --- | --- | --- |
ScientificResearch(AFOSR)underawardnumberFA9550-
| brittle because | any | future | Apple part | could | silently | enable |     |     |     |     |     |     |
| --------------- | --- | ------ | ---------- | ----- | -------- | ------ | --- | --- | --- | --- | --- | --- |
20-1-0425;theDefenseAdvancedResearchProjectsAgency
| the DMP | on Icestorm | cores. | Finally,restricting |     | cryptogra- |     |     |     |     |     |     |     |
| ------- | ----------- | ------ | ------------------- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
(DARPA)undercontractnumbersW912CG-23-C-0022and
| phyto run | on Icestorm |     | cores willlikelyincura |     | significant |     |                |     |              |         |            |       |
| --------- | ----------- | --- | ---------------------- | --- | ----------- | --- | -------------- | --- | ------------ | ------- | ---------- | ----- |
|           |             |     |                        |     |             |     | HR00112390029; |     | the National | Science | Foundation | (NSF) |
performancepenalty.
undergrantnumbers1954712,1954521,2154183,2153388,
| Blinding. | Analternativesolutionistoapplycryptographic |     |     |     |     |     |     |     |     |     |     |     |
| --------- | ------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
and1942888;theAlfredP.SloanResearchFellowship;and
blinding-liketechniques.Forexample,byinstrumentingthe
giftsfromIntel,Qualcomm,andCisco.
codetoadd/removemaskstosensitivevaluesbefore/afterbe-
ingstored/loadedfrommemory.Theseideascouldbeapplied
References
| in different | ways | depending | on the | sensitive | program. | For |     |     |     |     |     |     |
| ------------ | ---- | --------- | ------ | --------- | -------- | --- | --- | --- | --- | --- | --- | --- |
instance,inourattackonDiffie-HellmanKeyExchange,one
|     |     |     |     |     |     |     | [1] ArmArmv8-AArchitectureRegisters. |     |     | https://developer.arm.c |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------ | --- | --- | ----------------------- | --- | --- |
cangeneratearandomnumbertomaskthesecretkeyforev-
om/documentation/ddi0595/2021-12/.
erykeyexchange[45].Themajordownsideofthisapproach
[2] ARMCortex-ASeriesProgrammer’sGuideforARMv8-A.https://
isthatitrequirespotentiallyDMP-bespokecodechangesto
developer.arm.com/documentation/den0024/a.
everycryptographicimplementation,aswellasheavyperfor-
[3] DataOperandIndependentTimingInstructionSetArchitecture(ISA)
mancepenaltiesforsomecryptographicschemes.
|     |     |     |     |     |     |     | Guidance. | https://www.intel.com/content/www/us/en/deve |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------- | -------------------------------------------- | --- | --- | --- | --- |
Ad-HocDefenses. Finally,onecanimaginepointdefenses loper/articles/technical/software-security-guidance/
thatinterferewithspecificstepsintheattack.Forexample, best-practices/data-operand-independent-timing-isa-g
uidance.html.
changingvictimstobettervalidateinputsorschedulingpoli-
|                |             |     |           |          |     |           | [4] OnurAciiçmez. |     | Yetanothermicroarchitecturalattack:exploitingi- |     |     |     |
| -------------- | ----------- | --- | --------- | -------- | --- | --------- | ----------------- | --- | ----------------------------------------------- | --- | --- | --- |
| cies to forbid | co-location |     | [70]. The | downside | of  | these ap- |                   |     |                                                 |     |     |     |
cache.InCSAW,2007.
proachesisthattheyaread-hocandleavetherootcause(the
[5] OnurAcıiçmez,ÇetinKayaKoç,andJean-PierreSeifert.Predicting
DMP)unaddressed.
secretkeysviabranchprediction.InCT-RSA,2006.
| Hardware | Support. |     | Longerterm,we | view | the | rightso- |     |     |     |     |     |     |
| -------- | -------- | --- | ------------- | ---- | --- | -------- | --- | --- | --- | --- | --- | --- |
[6] OnurAciicmez,Jean-PierreSeifert,andCetinKayaKoc.Predicting
| lution to | be to | broaden | the hardware-software |     | contract | to  |     |     |     |     |     |     |
| --------- | ----- | ------- | --------------------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
SecretKeysviaBranchPrediction.IACR,2006.
accountfortheDMP.Ataminimum,hardwareshouldexpose
tosoftwareawaytoselectivelydisabletheDMPwhenrun- [7] SamAinsworthandTimothyM.Jones.GraphPrefetchingUsingData
StructureKnowledge.InICS,2016.
ningsecurity-criticalapplications.Thisalreadyhasnascent
|          |            |     |                  |      |            |     | [8] Sam AinsworthandTimothy |     |     | M. Jones. | An Event-TriggeredPro- |     |
| -------- | ---------- | --- | ---------------- | ---- | ---------- | --- | --------------------------- | --- | --- | --------- | ---------------------- | --- |
| industry | precedent. | For | example, Intel’s | DOIT | extensions |     |                             |     |     |           |                        |     |
grammablePrefetcherforIrregularWorkloads.InASPLOS,2018.
specificallymentiondisablingtheirDMPthroughanISAex-
|     |     |     |     |     |     |     | [9] Monjur | Alam,Haider | Adnan | Khan,Moumita | Dey,Nishith | Sinha, |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ----------- | ----- | ------------ | ----------- | ------ |
tension[3].Longerterm,onewouldideallylikefiner-grain
|          |          |           |         |         |          |      | RobertCallan,AlenkaZajic,andMilosPrvulovic. |     |                  |     |                        | One&Done: A |
| -------- | -------- | --------- | ------- | ------- | -------- | ---- | ------------------------------------------- | --- | ---------------- | --- | ---------------------- | ----------- |
| control, | e.g., to | constrain | the DMP | to only | prefetch | from |                                             |     |                  |     |                        |             |
|          |          |           |         |         |          |      | Single-Decryption                           |     | EM-Basedattackon |     | OpenSSL’sConstant-Time |             |
specificbuffersordesignatednon-sensitivememoryregions. blindedRSA.InUSENIXSecurity,2018.
[10] ThomasAllan,BillyBobBrumley,KatrinaFalkner,JoopVandePol,
|     |     |     |     |     |     |     | andYuvalYarom. |     | Amplifyingsidechannelsthroughperformance |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --- | ---------------------------------------- | --- | --- | --- |
9 Conclusions
degradation.InACSAC,2016.
[11] JoséBacelarAlmeida,ManuelBarbosa,GillesBarthe,FrançoisDupres-
InthispaperweshowedthatDMPsposeasignificantsecurity soir,andMichaelEmmi. VerifyingConstant-Timeimplementations.
threattomodernsoftware,breakingawidevarietyofstate- InUSENIXSecurity,2016.
of-the-art cryptographic implementations. At a high level, [12] RobertoAvanzi,JoppeBos,LéoDucas,EikeKiltz,TancrèdeLepoint,
VadimLyubashevsky,JohnMSchanck,PeterSchwabe,GregorSeiler,
22Weobservethatsettingthedataindependenttiming(DIT)[1]bitdisables andDamienStehlé.Crystals-kyberalgorithmspecificationsandsup-
theDMPbehavioronM3,whichisnotthecasewithM1andM2. portingdocumentation(version3.02).NISTsubmissions,2021.
| USENIX Association |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1131 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

[13] GillesBarthe,GustavoBetarte,JuanCampo,CarlosLuna,andDavid [35] JintaiDing,ScottFluhrer,andSaraswathyRv. Completeattackon
Pichardie.System-levelnon-interferenceforconstant-timecryptogra- rlwekeyexchangewithreusedkeys,withoutsignalleakage.InACISP,
phy.InCCS,2014. 2018.
[14] GillesBarthe,BenjaminGrégoire,andVincentLaporte.Securecom- [36] DmitryEvtyushkin,RyanRiley,NaelAbu-Ghazaleh,andDmitryPono-
pilationofside-channelcountermeasures:Thecaseofcryptographic marev. BranchScope: ANewSide-ChannelAttackonDirectional
“constant-time”.InCSF,2018. BranchPredictor.InASPLOS,2018.
[15] AlexandreBerzati,AnderssonCalleViera,MayaChartouny,Steven [37] ArmandoFaz-HernándezandKrisKwiatkowski.IntroducingCIRCL:
Madec,DamienVergnaud,andDavidVigilant.Exploitingintermediate AnAdvancedCryptographicLibrary.Cloudflare,June2019.Available
valueleakageindilithium:atemplate-basedapproach.InCHES,2023. athttps://github.com/cloudflare/circl.v1.3.3AccessedMay,
2023.
[16] AbhishekBhattacharjee. BreakingtheAddressTranslationWallby
AcceleratingMemoryReplays.IEEEMicro,2018. [38] EiichiroFujisakiandTatsuakiOkamoto.Secureintegrationofasym-
metricandsymmetricencryptionschemes.InCRYPTO,1999.
[17] SaraniBhattacharya,ChesterRebeiro,andDebdeepMukhopadhyay.
HardwarePrefetchersLeak:ARevisitofSVFforCache-TimingAt- [39] DanielGenkin,Lev Pachmanov,ItamarPipman,andEran Tromer.
tacks.InMICROW,2012. StealingkeysfromPCsusingaradio:Cheapelectromagneticattacks
onwindowedexponentiation.InCHES,2015.
[18] LeonGrootBruinderinkandPeterPessl.Differentialfaultattackson
deterministiclatticesignatures.CHES,2018. [40] DanielGenkin,AdiShamir,andEranTromer.Rsakeyextractionvia
low-bandwidthacousticcryptanalysis.InCRYPTO,2014.
[19] DavidBrumleyandDanBoneh.Remotetimingattacksarepractical.
InUSENIXSecurity,2005. [41] DanielGenkin,LukeValenta,andYuvalYarom. MaytheFourthBe
WithYou:AMicroarchitecturalSideChannelAttackonSeveralReal-
[20] EladCarmon,Jean-PierreSeifert,andAvishaiWool. Photonicside
WorldApplicationsofCurve25519.InCCS,2017.
channelattacksagainstrsa.InHOST,2017.
[42] Google/LLVM. SpeculativeLoadHardening. https://llvm.org/
[21] SunjayCauligi,CraigDisselkoen,KlausvGleissenthall,DeanTullsen,
docs/SpeculativeLoadHardening.html,2018.
DeianStefan,TamaraRezk,andGillesBarthe.Constant-timefounda-
tionsforthenewspectreera.InPLDI,2020. [43] BenGras,KavehRazavi,HerbertBos,andCristianoGiuffrida.Trans-
lationleak-asidebuffer:Defeatingcacheside-channelprotectionswith
[22] SunjayCauligi,GarySoeller,FraserBrown,Brian Johannesmeyer,
TLBattacks.InUSENIXSecurity,2018.
YunluHuang,RanjitJhala,andDeianStefan.Fact:Aflexible,constant-
timeprogramminglanguage.SecDev,2017. [44] BenGras,KavehRazavi,ErikBosman,HerbertBos,andCristiano
Giuffrida. Aslrontheline:Practicalcacheattacksonthemmu. In
[23] SunjayCauligi,GarySoeller,Brian Johannesmeyer,FraserBrown,
NDSS,2017.
RiadSWahby,JohnRenner,BenjaminGrégoire,GillesBarthe,Ranjit
Jhala,andDeianStefan.Fact:adslfortiming-sensitivecomputation. [45] DaHarkins.Dragonflykeyexchange.RFC7664,November2015.
InPLDI,2019.
[46] LorenzHetterichandMichaelSchwarz. Branchdifferent-spectreat-
[24] MustafaCavus,ResitSendag,andJoshuaJ.Yi.InformedPrefetching tacksonapplesilicon.InDIMVA,2022.
forIndirectMemoryAccesses.ACMTrans.Archit.CodeOptim.,2020.
[47] Tyler J Huberty,Stephan G Meier,andMridulAgarwal. Content-
[25] Boru Chen, Yingchen Wang, Pradyumna Shome, Christopher W. directedprefetchcircuitwithqualityfiltering,February62018. US
Fletcher,DavidKohlbrenner,RiccardoPaccagnella,andDanielGenkin. Patent9,886,385.
GoFetch:Breakingconstant-timecryptographicimplementationsusing
[48] MehmetSinanInci,BerkGulmezoglu,GorkaIrazoqui,ThomasEisen-
datamemory-dependentprefetchers.https://gofetch.fail,2024.
barth,andBerkSunar.Seriously,getoffmycloud!cross-vmrsakey
[26] YunChen,AliHajiabadi,LingfengPei,andTrevorE.Carlson.New recoveryinapubliccloud.IACR,2015.
Cross-CoreCache-AgnosticandPrefetcher-basedSide-Channelsand
[49] EmreKarabulut,ErdemAlkim,andAydinAysu. Single-traceside-
Covert-Channels.InArXiV,2023.
channelattacksonω-smallpolynomialsampling:Withapplicationsto
[27] YunChen,LingfengPei,andTrevorE.Carlson.AfterImage:Leaking ntru,ntruprime,andcrystals-dilithium.InHOST,2021.
ControlFlowDataandTrackingLoadOperationsviatheHardware
[50] Anirudh Mohan Kaushik,Gennady Pekhimenko,and Hiren Patel.
Prefetcher.InASPLOS,2023.
Gretch: A Hardware PrefetcherforGraphAnalytics. ACM Trans.
[28] ZhaohuiChen,EmreKarabulut,AydinAysu,YuanMa,andJiwuJing. Archit.CodeOptim.,2021.
Anefficientnon-profiledside-channelattackonthecrystals-dilithium
[51] TaehunKim,HyeongjinPark,SeokminLee,SeungheeShin,Junbeom
post-quantumsignature.InICCD,2021.
Hur,andYoungjooShin.Devious:Device-drivenside-channelattacks
[29] RobertCooksey,StephanJourdan,andDirkGrunwald. Astateless, ontheiommu.InS&P,2023.
content-directeddataprefetchingmechanism.ACMSIGPLANNotices,
[52] Paul Kocher, Daniel Genkin, Daniel Gruss, Werner Haas, Mike
2002.
Hamburg,MoritzLipp,StefanMangard,ThomasPrescher,Michael
[30] DonCoppersmith.Findingasmallrootofabivariateintegerequation; Schwarz,andYuvalYarom. Spectreattacks:Exploitingspeculative
factoringwithhighbitsknown.InEUROCRYPT,1996. execution.InS&P,2019.
[31] PatrickCroninandChengmoYang.Afetchingtale:Covertcommuni- [53] PaulC.Kocher.TimingAttacksonImplementationsofDiffie-Hellman,
cationwiththehardwareprefetcher.InHOST,2019. RSA,DSS,andOtherSystems.InCRYPTO,1996.
[32] MilesDai,RiccardoPaccagnella,MiguelGomez-Garcia,JohnMc- [54] AdamLangley,MikeHamburg,andSeanTurner. Rfc7748:Elliptic
Calpin,andMengjiaYan. Don’tmesharound:Side-Channelattacks curvesforsecurity,Jan2016.
andmitigationsonmeshinterconnects.InUSENIXSecurity,2022.
[55] MoritzLipp,MichaelSchwarz,DanielGruss,ThomasPrescher,Werner
[33] PeterJDenning.Virtualmemory.ACMComputingSurveys(CSUR), Haas,StefanMangard,PaulKocher,DanielGenkin,YuvalYarom,and
2(3):153–189,1970. MikeHamburg.Meltdown:ReadingKernelMemoryfromUserSpace.
InUSENIXSecurity,2018.
[34] WhitfieldDiffieandMartinEHellman.Newdirectionsincryptography.
In Democratizing Cryptography: The WorkofWhitfieldDiffie and [56] VadimLyubashevsky.Fiat-shamirwithaborts:Applicationstolattice
MartinHellman.2022. andfactoring-basedsignatures.InASIACRYPT,2009.
1132 33rd USENIX Security Symposium USENIX Association

[57] VadimLyubashevsky,LéoDucas,EikeKiltz,TancrèdeLepoint,Peter [77] Hauke Steffen,Georg Land,Lucie Kogelheide,and Tim Güneysu.
Schwabe,GregorSeiler,DamienStehlé,andShiBai.Crystals-dilithium Breakingandprotectingthecrystal:Side-channelanalysisofdilithium
algorithmspecificationsandsupportingdocumentation(version3.1). inhardware.InPQCrypto,2023.
NISTsubmission,2021.
[78] HritvikTaneja,JasonKim,JieJeffXu,StephanvanSchaik,Daniel
[58] SoundesMarzougui,VincentUlitzsch,MehdiTibouchi,andJean-Pierre Genkin,andYuvalYarom.HotPixels:Frequency,Power,andTemper-
Seifert.Profilingside-channelattacksondilithium:Asmallbit-fiddling atureAttacksonGPUsandARMSoCs.InUSENIXSecurity,2023.
leakbreaksitall.InSAC,2022.
[79] AndreiTatar,DaniëlTrujillo,CristianoGiuffrida,andHerbertBos.
[59] AlexanderMayandJulianNowakowski.TooManyHints-WhenLLL TLB;DR:EnhancingTLB-basedattackswithTLBdesynchronized
BreaksLWE.InASIACRYPT,2023. reverseengineering.InUSENIXSecurity,2022.
[60] RossMcilroy,JaroslavSevcik,TobiasTebbi,BenL.Titzer,andToon [80] MohitTiwari,HassanM.G.Wassel,BitaMazloom,ShashidharMysore,
Verwaest. Spectreisheretostay:Ananalysisofside-channelsand FredericT.Chong,andTimothySherwood. CompleteInformation
speculativeexecution.InArXiV,2019. FlowTrackingfromtheGatesUp.InASPLOS,2009.
[61] RobertMerget,MarcusBrinkmann,NimrodAviram,JurajSomorovsky, [81] AlbertoTonelli.Bemerkungüberdieauflösungquadratischercongruen-
JohannesMittmann,andJörgSchwenk.Raccoonattack:Findingand zen.NachrichtenvonderKönigl.GesellschaftderWissenschaftenund
exploitingMost-Significant-Bit-OraclesinTLS-DH(E).InUSENIX derGeorg-Augusts-UniversitätzuGöttingen,1891.
Security,2021.
[82] Stephan Van Schaik,Cristiano Giuffrida,Herbert Bos,and Kaveh
[62] DavidMolnar,MattPiotrowski,DavidSchultz,andDavidWagner.The Razavi. Maliciousmanagementunit: Whystoppingcacheattacks
ProgramCounterSecurityModel:AutomaticDetectionandRemoval insoftwareisharderthanyouthink.InUSENIXSecurity,2018.
ofControl-FlowSideChannelAttacks.InICISC,2005.
[83] StephanVanSchaik,KavehRazavi,BenGras,HerbertBos,andCris-
[63] KathleenMoriarty,BurtKaliski,JakobJonsson,andAndreasRusch. tianoGiuffrida.Revanc:Aframeworkforreverseengineeringhardware
Pkcs# 1: Rsa cryptographyspecifications version 2.2. RFC 8017, pagetablecaches.InEuroSec,2017.
November2016.
[84] JoseRodrigoSanchezVicarte,MichaelFlanders,RiccardoPaccagnella,
[64] DagArneOsvik,AdiShamir,andEranTromer. CacheAttacksand GrantGarrett-Grossman,AdamMorrison,ChristopherWFletcher,and
Countermeasures:TheCaseofAES.InCT-RSA,2006. DavidKohlbrenner.Augury:Usingdatamemory-dependentprefetch-
erstoleakdataatrest.InS&P,2022.
[65] RiccardoPaccagnella,LichengLuo,andChristopherWFletcher.Lord
ofthering(s):SidechannelattacksontheCPUOn-Chipringintercon- [85] PepeVila,BorisKöpf,andJoséFMorales. Theoryandpracticeof
nectarepractical.InUSENIXSecurity,2021. findingevictionsets.InS&P,2019.
[66] YueQin,ChiCheng,andJintaiDing.Anefficientkeymismatchattack [86] JunpengWan,YanxiangBi,ZheZhou,andZhouLi.Meshup:Stateless
onthenistsecondroundcandidatekyber.CryptologyePrintArchive, cacheside-channelattackoncpumesh.InS&P,2022.
2019.
[87] RuizeWang,KalleNgo,JoelGärtner,andElenaDubrova.Single-trace
[67] YueQin,ChiCheng,XiaohanZhang,YanbinPan,LeiHu,andJintai side-channelattacksoncrystals-dilithium: Mythorreality? IACR,
Ding.Asystematicapproachandanalysisofkeymismatchattackson 2023.
lattice-basednistcandidatekems.InASIACRYPT,2021.
[88] Yingchen Wang,Riccardo Paccagnella,Elizabeth Tang He,Hovav
[68] GokulnathRajendran,PrasannaRavi,Jan-PieterD’Anvers,Shivam Shacham,ChristopherWFletcher,andDavidKohlbrenner.Hertzbleed:
Bhasin,andAnupamChattopadhyay. Pushingthelimitsofgeneric TurningPowerSide-ChannelAttacksIntoRemoteTimingAttackson
side-channelattacksonlwe-basedkems-parallelpcoracleattackson x86.InUSENIXSecurity,2022.
kyberkemandbeyond.InCHES,2023.
[89] Yingchen Wang,Riccardo Paccagnella,Alan Wandke,Zhao Gang,
[69] JosephRavichandran,WeonTaekNa,JayLang,andMengjiaYan.Pac- GrantGarrett-Grossman,ChristopherW.Fletcher,DavidKohlbrenner,
man:Attackingarmpointerauthenticationwithspeculativeexecution. andHovavShacham.DVFSfrequentlyleakssecrets:Hertzbleedattacks
InISCA,2022. beyondSIKE,cryptography,andCPU-onlydata.InIEEES&P,2023.
[70] ThomasRistenpart,EranTromer,HovavShacham,andStefanSavage. [90] Zixuan Wang,Mohammadkazem Taram,Daniel Moghimi,Steven
Hey,You,GetoffofMyCloud:ExploringInformationLeakagein Swanson,Dean Tullsen,andJishen Zhao. Nvleak: Off-chip side-
Third-PartyComputeClouds.InCCS,2009. channelattacksvianon-volatilememorysystems.InUSENIXSecurity,
2023.
[71] KeeganRyanandNadiaHeninger. Fastpracticallatticereduction
throughiteratedcompression.InCRYPTO,2023. [91] ChongXiao,MingTang,andSylvainGuilley.Exploitingthemicroar-
chitecturalleakageofprefetchingactivitiesforside-channelattacks.
[72] JoseRodrigoSanchezVicarte,PradyumnaShome,NandeekaNayak,
JournalofSystemsArchitecture,2023.
CarolineTrippel,AdamMorrison,DavidKohlbrenner,andChristo-
pherW.Fletcher.OpeningPandora’sBox:ASystematicStudyofNew [92] YuvalYaromandKatrinaFalkner.Flush+Reload:Ahighresolution,
WaysMicroarchitectureCanLeakPrivateData.InISCA,2021. lownoise,L3cacheside-channelattack.InUSENIXSecurity,2014.
[73] MichaelSchwarz,ClémentineMaurice,DanielGruss,andStefanMan- [93] YuvalYarom,DanielGenkin,andNadiaHeninger. CacheBleed:A
gard.Fantastictimersandwheretofindthem:High-resolutionmicroar- TimingAttackonOpenSSLConstantTimeRSA.IACR,2016.
chitecturalattacksinjavascript.InFC,2017.
[94] JiyongYu,AishaniDutta,TrentJaeger,DavidKohlbrenner,andChristo-
[74] DanielShanks.Fivenumber-theoreticalgorithms.InProceedingsofthe pherW.Fletcher.Synchronizationstoragechannels(S2C):Timer-less
SecondManitobaConferenceonNumericalMathematics(Winnipeg), cacheSide-Channelattacksontheapplem1viahardwaresynchroniza-
1973. tioninstructions.InUSENIXSecurity,2023.
[75] MuyanShen,ChiCheng,XiaohanZhang,QianGuo,andTaoJiang. [95] JiyongYu,MengjiaYan,ArtemKhyzha,AdamMorrison,JosepTorrel-
Findthebadapples:Anefficientmethodforperfectkeyrecoveryunder las,andChristopherW.Fletcher.SpeculativeTaintTracking(STT):A
imperfectscaoracles–acasestudyofkyber.InCHES,2023. ComprehensiveProtectionforSpeculativelyAccessedData.InMICRO,
2019.
[76] YoungjooShin,HyungChanKim,DokeunKwon,JiHoonJeong,and
JunbeomHur. Unveilinghardware-baseddataprefetcher,ahidden [96] XiangyaoYu,ChristopherJ.Hughes,NadathurSatish,andSrinivas
sourceofinformationleakage.InCCS,2018. Devadas.IMP:IndirectMemoryPrefetcher.InMICRO,2015.
USENIX Association 33rd USENIX Security Symposium 1133

[97] Xiangyao Yu,Christopher J. Hughes,and Nadathur Rajagopalan searchspace.Toaddressthis,theattackercandecoupleEV a
Satish. Hardware prefetcherforindirectaccess patterns,February and EV by first targeting sig.z,where the attackercan
ptr
2017.US9582422B2. confirmptrinjection.Notethatthecompoundevictionset
[98] ZhiyuanZhang,MingtianTao,SioliO’Connell,ChitchanokChuengsa- tosig.zsharesthesameEV withthatofy,thushaving
ptr
tiansup,DanielGenkin,andYuvalYarom.BunnyHop:Exploitingthe
therightEV makesgeneratingEV foryefficient.
InstructionPrefetcher.InUSENIXSecurity,2023. ptr a
Noisetolerance. Weobservedseveralsourcesofnoiseor
failurewhencheckingfortheexistenceofptrina.
A Standardevictionsetsgenerationalgorithm First,the backgroundnoise ofthe Prime+Probe channel
couldresultinfalsepositives.Toaddressthis,wetake32la-
Wenowbrieflypresentthemethodtogenerateevictionsets tencyobservations(Sections6and7)andapplythefollowing
coveringallL2sets. Westartwithgeneratingevictionsets strategies forourcryptography targets. To startoff,during
with a fixed page offset,which map to L2 sets differed by theattackprocess,theattackeralsoperformsthebackground
upper6bits.Tothisend,wesweepapoolofpagestoidentify testaccessesbysendingrandommessages.Havingthesere-
newevicttargets,andtestifthefixedoffsetintooneofthem dundantmeasurementsinterleavedwithnormaltestaccesses
hasconflictswiththecurrentevictionsetgroup.Ifthereisno establishesconfidencethatitistheDMPcausinghighlaten-
conflict,itmeansthatthisevicttargetmapstoanewL2set ciesinthenormaltestaccesses.Second,anattackercandetect
whoseevictionsetisnotincludedinthecurrentgroup.With errorsinRSA-2048(Go)andDHKE-2048(OpenSSL),and
thisevicttarget,weusethetechniquesfromVilaetal.[85] isabletorollbackandredotheexperimentinsuchcases.In
togenerateamatchingevictionsetandadditintothegroup. RSA(Go),ifonebitiswrong0/1,theciphertextcwillalways
Finally,foreachofthe64(26)evictionsets,wefixtheupper besmaller/largerthan p,resultingintheattackerrecovering
6bits,andgenerateevictionsetsforeverycombinationofthe consecutive1/0bits,anunlikelypatterninpractice.InDHKE-
lower7bitsforatotalof8192evictionsets. 2048 (OpenSSL),ifan erroneous s is chosen atthe i-th
i−1
window,theattackerwillnotobserveanyDMPsignalforthe
nextwindow,becausethechallengecforsubsequentwindows
B Compound Eviction Sets Generation and
isbasedoncorrectnessofs .Thismethodisnotapplicable
i−1
NoiseToleranceTips
forKyber-512(reference)andDilithium-2(CIRCL),because
therecoveryofeachcoefficientinKyber/Dilithiumisinde-
Compoundevictionsetsgeneration. Togenerateacom-
pendentoftheothers.Anattackercanalwaysrepeattheattack
poundevictionset(EV ,EV ),theattackermustsolvetwo
a ptr severaltimesandtakeamajorityvote.
problems.First,theymustidentifytheaddressofaaswellas
Second,amaychangeitsvirtualaddressatruntime,render-
valid(andquiet)pagestosearchforptr.Second,theymust
ingEV ineffectiveandcausingfalsenegatives.Todetectthis,
a
confirmptrinjectiontoa.
theattackermustcheckknown-goodptrinjectiontoinfer
Forthe firstproblem,bothDHKE-2048 (OpenSSL) and
thevalidityofthecurrentcompoundevictionset.Aslongas
Kyber-512(referenceimplementation)allocateainthesame
thishappensinfrequently,theattackercanthenre-generate
4GByteregionasthedyldcache,whichisanidealtargetfor
thecompoundevictionset.
ptr. The virtualaddress ofthe dyld sharedlibraryis only Third,theintervalbetweeneachloadtoamaybeshorter
randomizedbymacOSatboottimeandtheattackercanre-
thantraversingEV .Onesolutionistodecreasethesizeof
a
coveritwithanotherunprivilegedprocessbyrunningvmmap.
EV untilitmatchesthatofastandardevictionset.Iftravers-
a
RSA-2048(Go)anddeterministicDilithium-2(CIRCL)allo-
ingastandardevictionsetistooexpensive,apossiblesolution
cateainastableaddressregionbeyond0x14000000000.23
istodegradetheperformanceofthevictimprogram[10].
PagesinthisregionarealwaysvalidevenwithASLR,which
makesitanidealtargetforptr.
Forthesecondproblem,inRSA-2048(Go),aslongasthe
ciphertextcissmallerthan pandq,ptrwillbepreservedin
a.InDHKE-2048(OpenSSL),thefirstwindowofsecret,s ,
0
isalways1.Hence,theattackercaninjecttheptrtoaforthe
firstiterationbysolvingEquation(2)withE=1.InKyber-
512(reference),ptrcanbeinjectedbycorrectlyencrypting
messagemwithptr. Dilithium-2(CIRCL)istrickyasthe
attackercannotconfirmptrinjectiontoy(ainDilithium),
but has a pool of messages from which a subset correctly
injects ptr. Moreover, the so-called semi-confirmation in
Dilithiumsignificantlyincreasesthecompoundevictionset
23Thisspecificaddressisafunctionofthetargetprogram.
1134 33rd USENIX Security Symposium USENIX Association
