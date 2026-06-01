---
publish: true
---

New Cross-Core Cache-Agnostic and Prefetcher-based
Side-Channels and Covert-Channels
YunChen AliHajiabadi
NationalUniversityofSingapore NationalUniversityofSingapore
LingfengPei TrevorE.Carlson
NationalUniversityofSingapore NationalUniversityofSingapore
ABSTRACT officialarchitecturemanual[26]butonlyhasabrieffunctionality
Inthispaper,werevealtheexistenceofanewclassofprefetcher, descriptionintheHPCoptimizationmanual[2].
theXPTprefetcher,inthemodernIntelprocessorswhichhasnever Inthispaper,wefirstreverse-engineertheXPTprefetcherand
beenofficiallydocumented.Itspeculativelyissuesaload,bypassing revealitsprefetchingmechanism.Weinvestigatetheinteractionof
last-levelcache(LLC)lookups,whenitpredictsthataloadrequest differentcoresthroughtheprefetcher.Ouranalysishasresultedin
willresultinanLLCmiss.WedemonstratethatXPTprefetcheris thefirstside-channelusingtheXPTprefetcher,calledPrefetchX,
sharedamongdifferentcores,whichenablesanattackertobuild whichcanbeexploitedin3𝑟𝑑 generationXeonprocessors1toef-
cross-coreside-channelandcovert-channelattacks.Wepropose fectivelymonitorthevictim’spageactivities.Thisisachievedby
PrefetchX,across-coreattackmechanism,toleakusers’sensitive deliberatelymistrainingtheXPTprefetcheronspecificpagesand
dataandactivities. subsequentlyexaminingtheprefetcher’sstatus.Consequently,the
WeempiricallydemonstratethatPrefetchXcanbeusedtoex- attackercanreconstructthevictim’ssensitiveinformation,like
tractprivatekeysofreal-worldRSAapplications.Furthermore,we cryptographickeys.
showthatPrefetchXcanenableside-channelattacksthatcan Unlikemanystandardthreatmodelsthatnecessitatethevictim
monitorkeystrokesandnetworktrafficpatternsofusers.Ourtwo andattackertosharethesamephysicalcore[10–12,16,27,37,40,
cross-corecovert-channelattacksalsoseealowerrorrateanda 44,50],PrefetchXenablescross-coreside-channelattacks.This
1.7MB/smaximumchannelcapacity.Duetothecache-independent substantiallybroadensthescopeofpotentialtargetsandamplifies
feature of PrefetchX, current cache-based mitigations are not thepotentialsecurityimpact.Inaddition,thePrefetchXattacksdo
effectiveagainstourattacks.Overall,ourworkuncoversasignifi- notrelyonthecachesubsystem;wedonotrelyonthecachesasa
cantvulnerabilityintheXPTprefetcher,whichcanbeexploited sourceofleakagenorasaprimitivetochecktheprefetcher’sstatus.
tocompromisetheconfidentialityofsensitiveinformationinboth Tothebestofourknowledge,wearethefirsttoidentify,explore,
cryptoandnon-crypto-relatedapplicationsamongprocessorcores. andreverse-engineertheXPTprefetcher.Thisdiscoveryhighlights
theexistenceofhiddenchannelsthatcouldexposesecurityrisks
inprocessorarchitectures.
Ourkeycontributionsinthisworkareasfollows:
1 INTRODUCTION
• Weuncoverabriefly-mentionedprefetcher,namedtheXPT
Decadesofresearchindesigningefficientandmodernprocessors
prefetcher,inthe3𝑟𝑑 generationofIntelserverprocessors.
hasresultedinvariousperformanceenhancementsatthemicroar-
Wefullyreverse-engineerthisprefetcher,andprovidea
chitecturallevel,likeout-of-orderexecution,speculativeexecution,
detailedcharacterizationofitsfeaturesandbehaviors(Sec-
multi-coreprocessing,caching,prefetching,andsharingresources
tion4).
amongdifferentcores.However,inrecentyearstherehasbeen
rapiddiscoveryofsecurityvulnerabilitiesarisingfromtheseperfor- • Weconstructfourend-to-endcross-coreside-channelat-
tacksusingtheXPTprefetcherastheleakysource.Theseat-
manceenhancementtechniques[10,12,13,16,18,27,31,37,46,48].
tacksincludeakeystrokeattack,anetworktrafficmonitor-
Thesevulnerabilitiesareexploitedtoeitherinferauser’sprivate
ingattack,andtwodistinctRSAattackstobreaktheprivate
dataandsecretkeys(incaseofside-channelattacks)ortostealthily
exponent(asquare-and-multiplyRSAusedinGnuPG1.4[1]
transferdatainthesystem(inthecaseofcovert-channelattacks).
andatiming-constantMontgomery-LadderRSA[29]used
Oneoftheprominentsourcesofinformationleakageinmodern
inOpenSSL[34]andMbedTLS[33]).Ourresultsdemon-
processors are prefetchers since they can exhibit a footprint of
stratethepracticalityandeffectivenessof PrefetchXin
thedataaccessedbythevictims,similartocaches.Thegoalof
real-worldscenarios2(Section5andSection6).
prefetchingistobringthedataclosertothecoreifithashigh
confidencetobeneededinthenearfuture.Thistechniquecanbe • Wedeveloptwocross-corecovert-channelattackswithlow
errorrates,furtherdemonstratingthehighapplicabilityand
implementedoneitherhardware[7,24]orsoftware[22].Recent
potentialsecurityimplicationsof PrefetchX(Section7).
attacksexploitavarietyofprefetchingmechanisms[10,11,15,17,
20,21,40,44]toleavepersistentmaliciousandsecret-dependent
changesinthesystemthatcanbelaterinferredbytheattacker.
Inthispaper,weexploreaprefetcherinIntelprocessors,called
1ThelatestXeonprocessorsatthetimeofpapersubmission.
eXtendedPredictionTable(XPT),thatislocatedinparalleltothe
2WehaveresponsiblydisclosedourfindingstotheIntelPSIRTteamandhavereceived
LLC. The XPT prefetcher is not documented in the latest Intel approvaltodistributethesedetails.
3202
nuJ
91
]RC.sc[
1v59111.6032:viXra

andvictimintheside-channelattacks,orthesenderandreceiver
L1 L2 LLC
Memory L1 miss L2 miss LLC miss DRAM inthecovert-channelattack.Weaimtotransferorleakprivate
Access Lookup Lookup Lookup Access informationacrosscoresinsideaprocessor4byobservingthepage-
Instructions
accessingactivitiesthroughtheXPTprefetcher.Wealsoassume
XPT
thatthetwopartieshaveshareddatasuchassharedmemoryspace
Prefetcher if predicted as
orsharedlibraries(e.g.OpenSSL[4]),similartoprevalenthardware
LLC miss
attackingscenarios[18,44,47,48].AsweleverageRDTSCtocom-
Normal Memory Access Optimized Memory Access
putetheaccesslatency,whichisrunningwithaconstantfrequency
anddoesnotchangewiththeCPUfrequency[25],wemakeno
Figure1:OverviewoftheXPTprefetcheroperation.
assumptionsontheCPUfrequencyoftargetedcores.
2 PREFETCHXMOTIVATIONANDOVERVIEW
2.3 PrefetchXWorkflow
2.1 TheXPTPrefetcherasaLeakageSource
ThegeneralattackflowofPrefetchXcanbesummarizedasthree
PrefetchXexploitstheXPTprefetcherinIntelprocessors,andthe mainsteps:
rootcauseoftheattackisthespecialmechanismofthiscomponent. Step1:ChannelPreparation.Toconstructaside-channel,the
TheXPTprefetcherresidesinparallelwiththeLLCanditsmain attackerfirstidentifiesthephysicaladdressofthetargetshared
goalistopredictLLCmissesandtosendspeculativerequeststo pagethatthevictim’sinterestingactionswillaccess,whichcan
theDRAMtoconductLLClookupsinadvance.Thiscanreduce beeasilydonebyusingestablishedtechniques[28,50].Thenthe
memoryaccesslatencyinthecaseofacorrectprediction.Hence, attackerprimesatargetedXPTentrycorrespondingtothevictim’s
thememoryaccessessenttotheLLCsubsystemexhibitthreelevels physicaladdressonthesharedpage.Basedonthefindingsofour
ofaccesslatency(weusetheRDTSCtocomputethelatency):(1) reverse-engineeringanalysis(refertoSection4.1),generating32
anLLChit(lessthan160cyclesinoursetupandexperiments),(2) cachemissesonapageissufficienttoprimetheXPTprefetcher
LLCmiss(350+cycles),andfinally(3)optimizedLLCmiss(170-330 tobeginprefetching.WhenusingtheXPTprefetcherasacovert-
cycles). The optimized LLC misses occur in cases that the XPT channel,thesenderandthereceiverleverageapredefinedpagefor
prefetcherhasacorrectpredictionandisenabled.Figure1shows communication.
anoverviewoftheXPTprefetcheralongsidethecachehierarchy. Step2:Performingthetargetaction.Aftertheside-channel
Thistimingvariationistightlycoupledwiththememoryactivities setup, the victim execution is initiated. As the victim runs, the
oftheexecutingprogramthatenableseasystatusmonitoringof prefetcher’sstatusisevictedorresetifthetargetactionisinvoked.
theprefetcher. Inthecaseofcovert-channel,thesendereitherprimesorflushes/e-
Inaddition,ourdetailedexperimentsinSection4revealthat(1) victsanXPTentrytosenddifferentbitsofthesecretmessage(i.e.,
XPTprefetchingcanbetrainedandtriggeredacrossdifferentcores, b’0orb’1).
that(2)theLLCmisspredictionisbasedonamisscounterinthe Step3:ChecktheXPTprefetcherstatus.Theattackerchecks
prefetcherfora4KiBpagegranularity,andthat(3)differententries theprefetcherstatustodeterminewhetherthevictimhasexecuted
intheprefetcherareindexedbasedonthephysicaladdressofthe thetargetfunction(incaseofcovert-channel,tocheckwhatbithas
dataanddistinctbyAddressSpaceID(ASID)andThreadID(TID)3.
beentransmittedbythesender).
In this work, we make two key observations from the XPT Byfollowingthesesteps,PrefetchXisabletoeffectivelyleak
prefetcherbehaviorthatenablesustobuildside-channelandcovert- thevictim’ssensitivepage-relatedactionsthroughside-channel
channelattacks.First,weobservethattheXPTprefetcherentries analysis,whilealsoenablingacovertchannel.Section5andSec-
areevictedbasedontheLeastRecentlyUsed(LRU)policyifan tion6describeourside-channelattacks,whileSection7describes
unmappedpageinthesameASIDisaccessed.Inotherwords, thedetailsofhowtousePrefetchXasacovert-channel.
ifthreadt1trainstheprefetcherwithacertainsetofdatapages
andthreadt2accessesapageoutsidethissetthentheoldestpage 3 BACKGROUND
trainedbyt1isevictedfromtheXPTprefetcher.Second,weob-
3.1 Prefetching
servethatflushingtheTLBresetstheXPTprefetcherstatus.
Wedemonstratethatifprocess1trainstheXPTprefetcherwith Themaingoalofaprefetcheristopredictthedataandinstructions
asharedpageAandprocess2flushesthatpagemappingfrom thatwillberequiredbytheprocessorinthenearfutureandtofetch
TLBthentheXPTprefetcherwillnotbetriggeredfortheprevi- themfromthemainmemoryintothecacheinadvance,improving
ouslytrainedpageA.InSection4.5,weprovidethedetailsofour systemperformance.Thereexistavarietyofprefetchingstyles.
observations. Softwareprefetchingallowsprogrammerstoinsertprefetchingin-
structionsindesiredlocationsofthecode[17,20,25,47].Hardware
2.2 PrefetchXThreatModelandAttack prefetchers,suchasnext-lineprefetcher[24],strideprefetcher[10],
Surface spatialandtemporalprefetchers[8,9,38,42],eachhavedifferent
strategiesforpredictingandfetchingdataandcanbebeneficial
Weassumetwounprivilegedthreads/processesrunningontwo
basedontheapplicationrunninginthehardware.
differentcoresofthesameprocessor,correspondingtotheattacker
4TheinterplaywithIntelSoftwareGuardExtension(SGX)isconsideredtobeoutside
3Note,eachprocesshasauniqueASID,whereasdifferentthreadsinthesameprocess thescopeofthisworkasSGXwasnotdesignedtobesecureagainstside-channel
shareanASIDbutareassigneddifferentTIDs attacks[5]andisnotsupportedinAWS(ourtestplatform).
2

IntelPrefetcher Location Pattern Attacks Virtual Address
DataCacheUnit
L1-D Nextcacheline[24] -
(DCU) TLB miss Page Table PT miss
Instructionpointer Loadinstructionswith [10,11] TLB DRAM
L1-D (PT)
(IP)-stride regularstride[10] [40] TLB write PT write
Dataprefetchlogic 128-bytes-aligned[24] TLB hit
L2 -
(DPL) paircacheline
Severalcachelines Physical Address
Streamer L2 [38]
backwardorforward[38]
XPTprefetcher LLC LLCmisspredictor Thiswork Figure2:Processofapagewalking.
Table1:Intelhardwareprefetchers[14].
somecases,whenonethreadupdatessharedmemory,the
operatingsystem(OS)maylaunchanIPIwhenthethread
AccordingtoIntel’swhitepaper[14],theirprocessordesigns isswitchedtoanothercore,followedbyflushingtheTLB
featurefourhardwareprefetchers,whicharelistedinTable1.Inthis toensurethatthecorrectdataispresented.
work,wefocusontheeXtensionPredictionTable(XPT)prefetcher,
WewilllatershowthatTLBflushimpactstheXPTprefetcher
whichisnotdocumentedbyIntelinanyoftheirhardwarereference
statusandistherootcauseofanumberofourcross-corecovert-
manuals [25, 26], but instead was briefly described in the HPC
optimizationreferencemanual[2]forthe3𝑟𝑑 generationofXeon channelandside-channelattacks.
processors.ThekeydifferenceoftheXPTprefetcherwithothers
isthatitisplacedatthelast-levelcache(LLC)(seeTable1)which 4 REVERSE-ENGINEERINGTHEXPT
makesitapotentialchannelforcross-coreattacks. PREFETCHER
TobetterunderstandtheXPTprefetcher,weperformedextensive
3.2 TranslationLookasideBufferandPage
microbenchmarkingtoreverseengineerit.Inthispaper,weuse
Tables
anAWSEC2m6i.4xlargeinstancepoweredbya16-coreIntel(R)
Translationandpagewalking.Figure2showsthevirtualad- Xeon(R)Platinum8375CCPU(IceLakegeneration,SunnyCove
dresstophysicaladdresstranslationprocessinx86processors.The microarchitecture).
memorymanagementunit(MMU)hasacache,calledtranslation
lookasidebuffer(TLB),thatkeepsthemostrecentlyaccessedpage 4.1 TriggeringtheXPTPrefetcher
mappingsforafasttranslationincaseofaTLBhit.Incaseofa
Basedonourexperiments,theXPTprefetcherisenabledaftera
TLBmiss,theMMUperformsapagewalkoverthepagetables(PT)
fixednumberofLLCcachemisses.Todeterminethenumberof
tofindthemapping.Theoperatingsystem(OS)isresponsiblefor
LLCcachemissesrequiredtotriggertheXPTprefetcher,weusea
maintainingthemappingsfromthevirtualaddressesprovidedby
microbenchmarkoutlinedinListing1.Themicrobenchmarkfirst
eachprocesstothephysicaladdressesindynamicrandom-access
initializesapage(line20)andflushestheinitializeddatafromthe
memory(DRAM).Asthesemappingstypicallyhaveagranularity
cacheusingclflushinstructions(line23).WethentraintheXPT
of4KiB,thepagewalkingprocessinvolvesprimarilytheupper48
prefetcheruptoaspecificnumberofLLCmisses,andfinally,testthe
bitsofthevirtualaddress,referredtoasthepageoffset.Meanwhile,
memoryaccesslatencyforanLLCcachemiss(i.e.,DRAMaccess)
thelower12bitsremainidenticaltothoseofthephysicaladdress
totestifXPTisenabled.Theparameterninthetrain()function
andarecalledthein-pageoffset.
specifies the number of cache misses to be generated, which is
TLBflush.Inmodernprocessors,theTLBisusuallyflushedat
achievedthroughtheuseofthegenerate_random()functionthat
anytimeiftheCPUneedstoupdatetheTLB[25].However,there
generatesnuniquerandomnumbersasindicesforaccessingthe
arethreemaineventsthatrequireaTLBflush:
page.Thisensuresthatthememoryaccessesareirregularandwill
• Context Switch: When the OS switches from one pro- nottriggerotherpotentialprefetchers,suchasthenext-line,IP-
cesstoanother,theTLBneedstobeupdatedwiththenew
stride,adjacent,andstreamerprefetchers.Aftergeneratingncache
virtual-to-physicaladdressmappingsforthenewprocess5.
misses,atest_indexisset,computedatruntimeanddistantfrom
• PageTableChanges:Ifthepagetablesaremodified,such n,totesttheDRAMlatencyoftheaccessatptr[test_index].
aswhenapageisswappedinoroutofmemory,theTLB
TheresultsofourexperimentsarepresentedinFigure3(green
mayneedtobeflushedtoensurethecorrectpagemappings
line)whichrunsourmicrobenchmarkwithanincreasingnumber
areused.
ofLLCmissesduringtraining.ItisevidentthatOnecanseethat
• Inter-ProcessorInterruptforCacheCoherency:Ina theXPTprefetcherbeginstostartrequestingdataearlyafter32
multiprocessorsystem,aninter-processorinterrupt(IPI)is
cachemisses,leadingtoareductioninDRAMaccesslatencyfrom
atypeofinterruptusedtoscheduletasksorsynchronize
approximately350+cyclestoanoptimizedLLCcachemiss(between
databetweendifferentprocessorsorcores.Whenmultiple
160to300cycles)whichhastriggeredtheXPTprefetcher.
threadsrunondifferentcoresandsharememory,main-
AstheXPTislocatedinparalleltotheLLC,itisreasonableto
tainingcachecoherencybetweenthecoresiscritical.In
assumethatitissharedbetweentwodifferentcores.Toverifythis
assumption,wemodifytheListing1torunthetrainandtest
5However,frequentTLBflushesduringcontextswitchesareavoidedinmodernpro-
functionsondifferentcores.TheresultispresentedinFigure3(red
cessorsaseveryTLBentryistaggedwithanASID,ensuringminimalimpacton
processperformance. line),whichissimilartoourobservationfromtriggeringtheXPT
3

void train(char ∗ptr) { TestedScenario IndexingPolicy IsXPTtriggered?
1
| 2   | int n =                      | 32;                    |           |     |     |     |            |     |                    |     |     |
| --- | ---------------------------- | ---------------------- | --------- | --- | --- | --- | ---------- | --- | ------------------ | --- | --- |
| 3   | int random_num[n];           |                        |           |     |     |     | Scenario-1 |     | PhysicalPage       |     | ✓   |
| 4   | generate_random(random_num); |                        |           |     |     |     | Scenario-2 |     | VirtualPage        |     | ✗   |
| 5   | for (int                     | i = 0; i               | < n; i++) | {   |     |     |            |     |                    |     |     |
|     | int                          | index = random_num[i]; |           |     |     |     | Scenario-3 |     | InstructionPointer |     | ✗   |
6
7 char junk = ptr[index ∗ CACHE_LINE]; Table2:TheXPTprefetchertriggeringresultswithdifferent
8 }
indexingpolicies.
9 }
| 10 void | test(char      | ∗ptr) {  |     |          |     |     |     |     |     |     |     |
| ------- | -------------- | -------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
|         | int test_index | = rand() | %   | 10 + 53; |     |     |     |     |     |     |     |
11
| 12  | int start | = rdtsc(); |     |     |     |     |     |     |     |     |     |
| --- | --------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
13 asm volatile("mfence");
| 14  | char junk               | = ptr[test_index |     | ∗ CACHE_LINE]; |     |        |          |     |     |     |     |
| --- | ----------------------- | ---------------- | --- | -------------- | --- | ------ | -------- | --- | --- | --- | --- |
| 15  | asm volatile("lfence"); |                  |     |                |     | 1 void | main() { |     |     |     |     |
int diff = rdtsc() − start; 2 char ∗huge_1GiB_page = (char ∗)mmap(1 << 30,
16
| 17 } |     |     |     |     |     | 3   |     | ... | , MAP_HUGE_1GB|MAP_LOCK); |     |     |
| ---- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------------- | --- | --- |
18 void main() { 4 char ∗huge_2MiB_page = (char ∗)mmap(1 << 21,
|     |               |         |     |     |     |     |     | ... | , MAP_HUGE_2MB|MAP_LOCK); |     |     |
| --- | ------------- | ------- | --- | --- | --- | --- | --- | --- | ------------------------- | --- | --- |
| 19  | int page_size | = 4096; |     |     |     | 5   |     |     |                           |     |     |
20 char ∗ptr = (char ∗)mmap(page_size, ... , MAP_LOCK, ...); 6 char ∗normal_4KiB_page = (char ∗)mmap(8192,
|     | for (int | i = 0; i | < page_size; | i+=64) |     | 7   |     |     |     | ... , MAP_LOCK); |     |
| --- | -------- | -------- | ------------ | ------ | --- | --- | --- | --- | --- | ---------------- | --- |
21
| 22  | ptr[i]         | = 'x'; |     |     |     | 8 char           | ∗ptr; | //refers | to one | of pools above |     |
| --- | -------------- | ------ | --- | --- | --- | ---------------- | ----- | -------- | ------ | -------------- | --- |
| 23  | flushall(ptr); |        |     |     |     | 9 train(ptr);    |       |          |        |                |     |
|     |                |        |     |     |     | test(ptr[4096]); |       | //cross  | 4KiB   | boundary       |     |
| 24  | train(ptr);    |        |     |     |     | 10               |       |          |        |                |     |
| 25  | test(ptr);     |        |     |     |     | 11 }             |       |          |        |                |     |
}
| 26                                                |     |     |     |     |     | Listing     | 3:  | Determine | the | page boundary | of the XPT |
| ------------------------------------------------- | --- | --- | --- | --- | --- | ----------- | --- | --------- | --- | ------------- | ---------- |
| Listing1:DetermininghowtotriggertheXPTprefetcher. |     |     |     |     |     | prefetcher. |     |           |     |               |            |
TheresultsshowninFigure3.
|     |     |     |     |     |     | We  | developed | a microbenchmark |     | (See Listing | 2) to evaluate |
| --- | --- | --- | --- | --- | --- | --- | --------- | ---------------- | --- | ------------ | -------------- |
Cross-Core Trigger Same-Core Trigger theindexingpolicyoftheXPTprefetcher.Ourmicrobenchmark
)selcyc( ycnetaL sseccA 480 involvesallocatingasharedmemorypageandachildprocess.The
|     | 400 |     |     |     | LLC miss |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- | --- |
trainandtestfunctions,asshowninListing1,arereusedinthis
320
|     |     |     |     |     | LLC miss  | experiment.ThemainprocesstrainstheXPTprefetcheronthe |     |     |     |     |     |
| --- | --- | --- | --- | --- | --------- | ---------------------------------------------------- | --- | --- | --- | --- | --- |
240
|     |     |     |     |     | (XPT enabled) | sharedpageandthechildprocessaccessesanuncacheddatablock |     |     |     |     |     |
| --- | --- | --- | --- | --- | ------------- | ------------------------------------------------------- | --- | --- | --- | --- | --- |
160
inthesharedpagetodetermineiftheXPTprefetcherhasbeen
80
|     |     | 32 misses |     |     | LLC hit | triggered. |     |     |     |     |     |
| --- | --- | --------- | --- | --- | ------- | ---------- | --- | --- | --- | --- | --- |
0
0 10 20 30 40 50 Asharedpagehasauniquemappinginthevirtualaddressspaces
Number of LLC misses ofboththemainprocessandthechildprocess(inListing2),be-
|     |     |     |     |     |     | cause | the main | process | and child | process have | distinct address |
| --- | --- | --- | --- | --- | --- | ----- | -------- | ------- | --------- | ------------ | ---------------- |
Figure3:TriggeringtheXPTprefetcherfromeitherthesame spaces.However,thephysicalpagemappingisthesameforboth
coreoradifferentcore.Inbothcases,theXPTprefetcheris processes.Theresultsofourexperiment,asshowninthefirstrow
triggeredafter32LLCmisses.
ofTable2(Scenario-1),indicatethattheXPTprefetcheriscon-
sistentlytriggeredwhentwoprocessesaccessthesamephysical
page,suggestingthattheXPTisindexedbythephysicalpage.We
| void | main()        | {       |     |     |     |                                                          |     |     |     |     |     |
| ---- | ------------- | ------- | --- | --- | --- | -------------------------------------------------------- | --- | --- | --- | --- | --- |
| 1    |               |         |     |     |     | alsotestedthepossibilityofindexingbythevirtualaddressand |     |     |     |     |     |
| 2    | int page_size | = 4096; |     |     |     |                                                          |     |     |     |     |     |
3 char ∗ptr = (char ∗)mmap(page_size, ... , MAP_LOCK, ...); IP(Scenario-2andScenario-3Table2,respectively),buttheresults
4 pid_t sub_process = fork(); indicatethatthesemethodsdonottriggertheXPTprefetcherand
| 5   | if (sub_process | >   | 0) // main | process |     |     |     |     |     |     |     |
| --- | --------------- | --- | ---------- | ------- | --- | --- | --- | --- | --- | --- | --- |
train(ptr);
| 6   |         |              |       |                |     | arenotusedforindexingtheXPTprefetcher. |     |     |     |     |     |
| --- | ------- | ------------ | ----- | -------------- | --- | -------------------------------------- | --- | --- | --- | --- | --- |
| 7   | else if | (sub_process | == 0) | // sub−process |     |                                        |     |     |     |     |     |
8 test(ptr); Togainadeeperunderstandingofhowmanybitsofthephys-
| 9 } |     |     |     |     |     | icaladdressareutilizedforindexingtheXPT,wecreatea32GiB |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ------------------------------------------------------ | --- | --- | --- | --- | --- |
Listing 2: Determining how the XPT prefetcher table is memorypoolsothatwecanmanipulatetheleastsignificant35bits
indexed.Weusesemaphoretocontrolthetrain/testsequence ofphysicaladdresses.Wethenidentifytwophysicalpageswith
identicalleastsignificant𝑀bitsfortheirpageoffsets(i.e.,physical
fromthesamecore(greenline).ThisimpliesthattheXPTprefetcher addressesfrom13𝑡ℎ bittothe (13+𝑀)𝑡ℎ bitarethesame).By
issharedbetweendifferentcores. trainingtheXPTprefetcherononepage,weexpectthatifthetable
|     |     |     |     |     |     | isindexedbythelower𝑀 |     |     | bitsofthepage,anotherpageshould |     |     |
| --- | --- | --- | --- | --- | --- | -------------------- | --- | --- | ------------------------------- | --- | --- |
4.2 IndexingintotheXPTPrefetcher alsotriggertheprefetcher.Nevertheless,wedonotobservethis
Intelcommonlyemploystheinstructionpointer(IP)[10],virtual behaviorevenwhen𝑀isincreasedto20,suggestingthattheXPT
prefetchershouldbeeitherindexedbythefullphysicaladdressor
| address | [26], | or physical | address | [13] as an | index foraccessing |     |     |     |     |     |     |
| ------- | ----- | ----------- | ------- | ---------- | ------------------ | --- | --- | --- | --- | --- | --- |
possessesatagforeachphysicalpage.
| hardware | tables | or caches | in  | its processors. | Our objective is to |     |     |     |     |     |     |
| -------- | ------ | --------- | --- | --------------- | ------------------- | --- | --- | --- | --- | --- | --- |
examineandunderstandtheindexingmechanismemployedbythe
4.3 PageBoundary
XPTprefetcher.GivenitspositionasabypassforLLClookupand
itslocationbetweentheLLCandL2cache,ourinitialfocusisto To determine if the XPT prefetcher is enabled in the case of
determineiftheXPTutilizesphysicaladdressindexing,similarto huge-paging(largerthannormalsizeof4KB)andcross-4KBac-
theLLC[13,26]. cessing.ThebenchmarkshownintheListing3firstmapsthree
4

eXtensionPrediction Table (XPT) Prefetcher Structure
Cross4KiB-PageBoundaryTrigger
PageTableSize Contiguouson Contiguouson IsXPT Physical  Address Space  Thread  Least Recent  LLC Miss
|     |     |     | VirtualAddress | PhysicalAddress | Triggered? |     |     |         |          |          |            |         | Enabled |
| --- | --- | --- | -------------- | --------------- | ---------- | --- | --- | ------- | -------- | -------- | ---------- | ------- | ------- |
|     |     |     |                |                 |            |     |     | Address | ID(ASID) | ID (TID) | Usage(LRU) | Counter |         |
seirtnE 652
|     |     | 1GiB | Yes | Yes |     | ✗   |     | 2ef4800 | 20  | 1   | 1   | 32  | True |
| --- | --- | ---- | --- | --- | --- | --- | --- | ------- | --- | --- | --- | --- | ---- |
|     |     | 2MiB | Yes | Yes |     | ✗   |     |         |     |     |     |     |      |
|     |     |      |     |     |     |     |     | …       | …   | …   | …   | …   | …    |
|     |     | 4KiB | Yes | No  |     | ✗   |     |         |     |     |     |     |      |
Table3:TheXPTprefetchertriggeringresultswithvarious 1af5000 140 1 1 18 False
pagetablessetting.
Figure5:ThearchitectureoftheXPTprefetcher
notingthatthegradualincreaseinlatencywiththelinearslope
|        |            |       |          |     |     |     | betweenthefirstpageandthe256𝑡ℎ |     |     |     | pageisaresultofqueuing |     |     |
| ------ | ---------- | ----- | -------- | --- | --- | --- | ------------------------------ | --- | --- | --- | ---------------------- | --- | --- |
| 1 void | train(char | ∗ptr, | int num) | {   |     |     | delay.6                        |     |     |     |                        |     |     |
| 2      | int n =    | 32;   |          |     |     |     |                                |     |     |     |                        |     |     |
3 int random_num[n]; InordertoaccuratelydeterminethesetassociativityoftheXPT,
generate_random(random_num);
| 4   |          |        |               |     |     |     | weallocatea16GiBmemorypoolandgenerate𝑁 |     |     |     |     |     | +1physical |
| --- | -------- | ------ | ------------- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- | --- | ---------- |
| 5   | for (int | i = 0; | i < num; i++) | {   |     |     |                                        |     |     |     |     |     |            |
6 for (int j = 0; j < n; i++) { pageswithidenticalbitsfromthe13𝑡ℎ bittothe (13+𝑀)𝑡ℎ bit.
7 int idx = random_num[j]; Wetestwithvaryingvaluesof𝑁,specifically𝑁 4,8,16,32,64,
| 8   | char | junk = | ptr[i∗4096+idx∗CACHE_LINE]; |     |     |     |     |     |     |     |     | =   |     |
| --- | ---- | ------ | --------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
}
| 9   |     |     |     |     |     |     | inanattempttoinferthenumberofcacheways.Additionally,we |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------ | --- | --- | --- | --- | --- | --- |
10 }
| 11 }    |     |           |            |     |         |            | define𝑀           | =256/𝑁,whichisusedtocomputethesetnumberofthe |     |     |     |     |     |
| ------- | --- | --------- | ---------- | --- | ------- | ---------- | ----------------- | -------------------------------------------- | --- | --- | --- | --- | --- |
| Listing | 4:  | Determine | the number | of  | entries | of the XPT | correspondingway. |                                              |     |     |     |     |     |
Forexample,ifweguessthattheXPTprefetcherusesa4-way
prefetcher.
set-associativecache,wewillprime5(4+1)entriestomaptothe
sameset,andatleastoneentryshouldbeevicted,therebystopping
)selcyc( ycnetaL sseccA
|     | 480 |     |     |     |     |     | prefetchingonthatparticularpage.However,weobservedthatno |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|     | 430 | 256 |     |     |     |     | evictionoccursevenwhenweincreasedtheguessednumberof      |     |     |     |     |     |     |
LLC miss
|     | 380 |     |     |     |     |     | waysto128.Basedonthesefindings,wehypothesizethattheXPT |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------ | --- | --- | --- | --- | --- | --- |
|     | 330 |     |     |     |     |     | operatesasafully-associativecache.                     |     |     |     |     |     |     |
LLC miss
280
(XPT enabled)
230
4.5 KeyObservationsandLeakageSources
180
|     | 0   | 250 | 500 750 | 1000 1250 | 1500 |     |     |              |            |            |         |      |             |
| --- | --- | --- | ------- | --------- | ---- | --- | --- | ------------ | ---------- | ---------- | ------- | ---- | ----------- |
|     |     |     |         |           |      |     | As  | we discussed | in Section | 2.1, there | are two | main | root causes |
Number of Trained Pages
leadingtodataleaksfromtheXPTprefetecher(XPTevictionsand
TLBflushes).Inthissection,wewilldiscussourkeyobservations.
Figure4:ThenumberofentriesoftheXPTprefetcher.
Table4consistsofsixdistinctscenarios.Thefirsttwoscenarios
memorypoolsnamed(1)huge_1GiB_page,(2)huge_2MiB_page, involvethecreationoftwothreadsrunninginseparateprocesses,
and(3)normal_4KiB_page.Thefirsttwomemorypoolswillbeal- withoneoftheprocesses(Process1)trainedonsharedpageA.
locatedwithhugepagetables(1GiBor2MiB),andthelastonewill Anotherprocess(Process2)isthentrainedonsharedpageAorB,
beallocatedwithanormalpagetable,i.e.,4KiB.Allthesepageswill andweassesstheabilityofProcess1tocontinuetriggeringthe
belabeledwithMAP_LOCKtoavoidunusedpoolsbeingreclaimed XPTprefetcheronpageA.Theresultsofthesetwoscenariosallow
andimpactingtheexperimentalresults.WethentraintheXPT ustoconcludethattheXPTissharedamongmultipleprocesses.In
prefetcheronthe4KiBregionandtestitonanother4KiBregion.In addition,thethirdscenariohighlightsthattheXPTprefetcheris
thiscase,onlynormal_4KiB_pagehasthecrosspageaccesses,and similarlysharedbetweendifferentthreadswithinthesameprocess
| theXPTprefetchershouldbere-trainedonthenewpage.Theresult |     |     |     |     |     |     | space. |     |     |     |     |     |     |
| -------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- |
presentedinTable3,however,showsthatevenhuge_1GiB_page Thefourthandfifthscenariosdemonstratedifferentbehaviors.
andhuge_2MiB_pagecannottriggertheXPTprefetcheraftercross- ThefourthscenariosuggeststhattheXPTprefetchershouldpos-
ingthe4KiBboundary,whichindicatesthattheXPTprefetcherhas sessathreadidentificationtagforcontrollingitsstatus.Asaresult,
a4KiBboundarytagineveryentrythatpreventscross4KiB-page iftwothreadsareexecutinginthesameprocessspace,i.e.,they
prefetch. sharethesameAddressSpaceID(ASID),andoneofthethreads
accessesapagenotpresentintheXPT,thenthewell-learneden-
4.4 NumberofEntriesandSetAssociativity triesassociatedwiththatASIDwillbeinvalidated.Thisraisesthe
WeconstructamicrobenchmarkthattrainstheXPTprefetcher questionofwhetherallentriesrelatedtotheASIDwillbeinvali-
onvaryingnumbersof4KiBpages(Listing4).Bygivingdifferent dated.Thefifthscenariowasconductedtoprovideananswer.In
thisscenario,Thread1trainstheXPTprefetcheronpagesAand
numberstonum,wecancreatenumentriesintheXPTprefetcher.
B,andthenThread2trainstheXPTprefetcheronpageC.The
Aftertraining,wethenusethetestfunctionshowninListing1to
resultsshowedthatonlytheprefetchingofpageAwasstopped.
testifthefirsttrainedpagecanstilltriggertheXPTprefetcher.
TheresultsoftheexperimentarepresentedinFigure4.The
DRAMaccesslatencyincreasessharplytoapproximately350+cy- 6Wehypothesizethatthequeuingdelayobservedisduetotheprefetcher,asthe
clesafteraccessing256pages,andthenremainsstable.Thisindicates latencyremainsunchangedafterprimingmorethan256entries.However,ifthe
delayisattributedtothememorycontroller,weexpecttoseeacontinuousincrease
thatthenumberofentriesintheXPTprefetcheris256.Itisworth inlatencybeyond256entries.
5

Cross Cross Cross Same
Scenario TrainingOperation InvalidatingOperation Invalidate?
Process Thread Core∗ Core
Scenario-1 ✓ ✓ ✓ ✓ Process1,pageA Process2,pageA Process1,pageA,✓trigger
Scenario-2 ✓ ✓ ✓ ✓ Process1,pageA Process2,pageB Process1,pageA,B,✓trigger
Scenario-3 - ✓ ✓ ✓ Thread1,pageA Thread2,pageA Thread1,pageA,✓trigger
Scenario-4 - ✓ ✓ ✓ Thread1,pageA Thread2,pageB Thread1,pageA,✗notrigger
Thread1,pageA,✗notrigger
Scenario-5 - ✓ ✓ ✓ Thread1,pagesA,B Thread2,pageC
Thread1,pageB,✓trigger
Process2
Scenario-6 ✓ ✓ ✓ ✓ Process1,pageA Process1,pageA,✗notrigger
flushestheTLB
✓:Thisconfigurationisenabledforthescenario.-:Thisconfigurationisdisabledforthescenario.
Table4:TheXPTprefetcherinvalidatingresultsondifferentscenarios.Notethatwebuildasharedmemoryregiontoenable
twoprocessestoaccessthesamephysicalpage.∗Inourexperiments,weevaluatevariouscorepairings,e.g.,Core-0andCore-1,
andCore-0andCore-15,toinvestigatethepotentialimpactofcoredistanceontheresults.Ourfindingshowsthatwealways
getsimilarresults,whichimpliesthattheXPTprefetcherremainsunaffectedbycoredistance.
Inanotherexperiment,whereThread1trainstheXPTprefetcher Insummary,whiletheXPTprefetcher’smechanismispromis-
onpagesA,B,andCandthenThread2trainsonpagesDandE, ingtobebeneficialformemory-intensiveapplicationswithirregu-
theprefetchingofpagesAandBwasfoundtohavebeenstopped. larmemoryaccesses,ourkeyobservationsinthissectionleadto
TheseresultsindicatethatathreadwillreplaceentriesintheXPT varioussuccessfulandpracticalside-channelandcovert-channel
trainedbythreadswiththesameASID,usingaLeastRecentUsage attacks.
(LRU)replacementpolicy.
5 EVICTION-BASEDCROSS-CORE
Observation1.Whentwothreads,t1andt2,havethesame
SIDE-CHANNELATTACKS
ASIDandt1trainstheXPTprefetcheroncertainpages.When
t2 starts a page walk, the oldest page trained by t1 will be InlightofourobservationsfromtheXPTprefetcherbehavior,we
evicted. proposeanewside-channelattackthatexploitsthesecharacteristics
toleaksecretkeysfromreal-worldcryptographicapplications.In
Inordertocomprehensivelyexaminetheinfluenceofthepage
thissection,wefocusonObservation1thatexploitsthecontention
managementunit,particularlytheTLB,onthebehavioroftheXPT
andevictionpoliciesoftheXPTprefetcher.
prefetcher,weconductedthesixthscenario.First,Process17trains
onasharedpageA.Then,Process2performsaflushoperationon
5.1 AttackingSquare-and-MultiplyRSA
theTLB.OurresultsindicatethatifaTLBflushoccurs,theXPT
prefetchercannotbetriggered. WefirstleverageObservation1toleaktheprivateexponentofthe
Square-and-MultiplyRSAapplicationusedinGnuPG1.4[1],which
Observation 2. When a page mapping is flushed from the isareal-worldapplicationthathasbeenusedasaproof-of-concept
TLB,thecorrespondingpageintheXPTprefetcherwillalsobe byrecentwork[21].AsshowninListing5,foreveryiteration,if
flushed. theleastsignificantbit(LSB)ofthe𝑒𝑥𝑝isequaltob’1,thebase𝑏
willbeloadedfrommemoryandmultipliedwithtemporaryresult
4.6 SummaryoftheXPTPrefetcherOperations 𝑟 (line3).Otherwise,onlyasquareoperationwillbeexecuted(line
TheXPTprefetcher,depictedinFigure5,operatesasfollows.Upon 4).
accessing a physical page, it checks for a hit in the table. If it AnXPTentrywillbeevictedifanewthreadinitiatesapage
is present and the cache miss counter is 32 or higher, then the walk(seeObservation1).Hence,anattackercantraintheXPT
prefetcheristriggeredandaspeculativeloadisissued.Whena prefetcheronapageotherthanthepagestoring𝑏andthenmeasure
tablemissoccurs,theXPTprefetcherchecksiftheASIDofthe theprefetcherstatusaftereachdecryptioniterationtoknowifthe
physicaladdressisrecorded.Ifpresent,itthencheckstheThread secret-dependentbranch(line2inListing5)isexecutedornot.
ID(TID).IftheTIDisfound,entriesrelatedtothisTIDareretained, Figure6showsanoverviewofourattacktorevealtheRSAprivate
andthenewpagereplacestheoldestentryofotherTIDsunderthe exponent.Forasuccessfulattack,weneedtoaddresstwochallenges:
sameASIDusingtheLRUreplacementpolicy.Iftherearenoother (1)howtosynchronizewiththevictimthreadtoguaranteewecan
TIDsassociatedwiththisASIDoriftheASIDisnotpresentedin measuretheXPTprefetcherstatusatapropertime.(2)Howto
thetable,thepageisallocatedwithanewentrywithacachemiss detectthepageusedtostore𝑏withoutsudo(asweneedtoavoid
counterof1,unlesstheprefetcherisfull,inwhichcasetheoldest trainingonthispage).
entryisreplacedusingtheLRUpolicy. Inthiswork,weusethepthreadandsemaphorelibrariesto
solvethefine-grainedsynchronizationchallengewiththevictim.
Morespecifically,inourexperimentalsetup,boththeattackerand
7ItalsoholdsthescenariothattwodifferentthreadshavethesameASID. victimthreadsarecreatedusingthepthread_createfunction,and
6

| Attacker | Repeating the attack for the next secret bit  |     |     |            |        |     |     |
| -------- | --------------------------------------------- | --- | --- | ---------- | ------ | --- | --- |
|          |                                               |     |     | while (exp | > 0) { |     |     |
1
|                               |     |                               |     | 2 bool check           | = exp ? true | : false;          |        |
| ----------------------------- | --- | ----------------------------- | --- | ---------------------- | ------------ | ----------------- | ------ |
|                               |     |                               |     | 3 if (check)           | {            |                   |        |
| 1 Training XPT                |     | 3 Checking XPT Status         |     |                        |              |                   |        |
|                               |     |                               |     | 4 multiply_and_add(x1, |              | z1, x2, z2, ctx); | //res1 |
|                               |     |                               |     | 5 multiple_and_add(x2, |              | z2, x2, z2);      | //res2 |
| Thread 0 | Process 0 | Core 0 |     | Thread 0 | Process 0 | Core 0 |     | }                      |              |                   |        |
6
7 else {
|               |              |     |               | 8 multiply_and_add(x2, |     | z2, x1, z1, ctx); | //res2 |
| ------------- | ------------ | --- | ------------- | ---------------------- | --- | ----------------- | ------ |
| Domain Switch | 2 Decryption |     | Domain Switch |                        |     |                   |        |
|               |              |     |               | 9 multiple_and_add(x1, |     | z1, x1, z1);      | //res1 |
10 }
| Victim | Thread 1 | Process 0 | Core 1 |     |     | exp = | exp >> 1; |     |     |
| ------ | ----------------------------- | --- | --- | ----- | --------- | --- | --- |
11
12 }
Figure6:OverviewofattackingRSAapplications.Afterthe Listing6:CodesegmentoftheMontgomery-LadderRSA.
attackertrainstheXPTprefetcher,thevictimdecryptsthe
messageandtheattackerthencanchecktheprefetchersta- Toconstructtheattack,wefirstdeterminethepageoffsetsof𝑟𝑒𝑠1
tus. and𝑟𝑒𝑠2usingthetechniquementionedinSection5.1.Bytraining
theXPTprefetcheronthepagestoring𝑟𝑒𝑠2andthencheckingthe
prefetcherstatusonthatpageafterthefirstmultiply_and_add()
functioniscomplete,wecandeterminewhichpagehasbeenac-
| 1 while (exp) { |     |     |     |     |     |     |     |
| --------------- | --- | --- | --- | --- | --- | --- | --- |
if (exp & 0x01) cessedbythevictim.Whenthesecretbitisb’1,apagewalkis
2
3 r = (r ∗ b) mod n;//load b from a shared page initiatedtothepagecontaining𝑟𝑒𝑠1,resultingintheevictionof
| 4 r = (r ∗ r) | mod n; |     |     |     |     |     |     |
| ------------- | ------ | --- | --- | --- | --- | --- | --- |
thetrainedentryfromtheprefetcher.Thisevictioncausestheat-
| 5 exp = exp >> | 1;  |     |     |                                                    |     |     |     |
| -------------- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- |
| 6 }            |     |     |     | tackertoobserveanormalLLCmisswhenaccessinguncached |     |     |     |
Listing5:CodesegmentoftheSquare-and-MultiplyRSAin
dataonthetrainedpage.Incontrast,whenthesecretbitisb’0,the
GnuGP.
pageholding𝑟𝑒𝑠2isaccessedfirst,andthewell-trainedentryis
notevicted,whichresultsinanoptimizedLLCmiss.Similartoour
theattackerutilizesasemaphoretolocktheCPUinordertoevict Square-and-MultiplyRSAattack,themainchallengeisthesynchro-
thesharedlibrary(whichcreatesapagewalkingactionforthevic- nizationwiththevictimbetweenthetwomultiply_and_add()
timtoinsertthepageintotheXPTprefetcherwhenthevictimloads invocationstomeasuretheXPTprefetcherstatus.Synchronization
𝑏)andtraintheXPTprefetcher.AftertrainingtheXPTprefetcher, atfunctioncall-levelgranularityisrelativelystraightforward(e.g.,
theattackerunlockstheCPUviareleasingthesemaphore.The bymanipulatingthesemaphoreatthebeginningandendofthe
victimthenlockstheCPUandrunstheRSAtotheattacker’sex- multiply_and_add()).Section8.1discussestheexperimentalre-
pectedpoint(e.g.,adecryptioniteration),andthenunlockstheCPU sultsofoursuccessfulattacktoleakoneofbyteoftheMontgomery-
whichallowstheattackertomeasuretheprefetcherstatus.This LadderRSAsecretkey.
methodisoftenusedasasimplifiedsynchronizationtechniquein
manyattacks[19,43]andhasbeendemonstratedtonotbeacritical Use-Case1.WeexploitObservation1toreconstructthesecret-
limitationandcanbedealtwithbyaLinuxschedulerattack[19]. dependentbehaviorsinvariousimplementationsofRSAengine
Moreover,concerningthesecondchallenge,theattackcaneasily andbuildaside-channelattack.
uncoverthepageoffsetof𝑏byemployingtheIntelPin-basedtech-
niqueproposedinBinoculars[50].Alternatively,theattackercan 6 TLB-FLUSH-BASEDCROSS-CORE
executethesharedlibrarylocallyandinferthepagemappingvia
SIDE-CHANNELATTACKS
RAMBleed[28].Inthiswork,weleveragethesecondtechniqueto
uncoverpagemappingfromtheuserlevel.InSection8.1,wepresent Ourside-channelattackspresentedinthissectionfocusonscenar-
ourproof-of-concept(PoC)attackresultsthatleadtosuccessful iosinwhichthevictim’ssensitivebehaviorresultsinTLBflushes
(exploitingObservation2).Inparticular,wehaveobservedthat
extractionofonebyteoftheRSAsecretkeyin2minutes.
certaineventsrelatedtodrivers(e.g.,keystrokes,Bluetoothconnec-
tions,networkpackettransmissionsviaanetworkcard,etc.)can
5.2 AttackingMontgomeryLadderRSA
triggerIPIsonIntelprocessorswhenrunningonseparatecores
| The Montgomery | Ladder RSA [29], | implemented | in popular li- |     |     |     |     |
| -------------- | ---------------- | ----------- | -------------- | --- | --- | --- | --- |
withotherthreadsandupdatingsharedmemory.ThiscausesaTLB
brariessuchasOpenSSL1.01[34]andMbedTLS[33],isareal-world flushandresettingtheXPTprefetcherstatus,resultinginanormal
RSAapplicationthatoffersresistanceagainsttimingside-channel LLCmisslatencyforuncacheddataaccess.Inourattacks,werun
attacks.Thekeyaspectofthealgorithm(Listing6)isthatregardless theattackprocessinparallelwiththevictim,andthesynchroniza-
ofthekey,thefunctionmultiply_and_add()isalwaysinvoked
tionisdonebyIPIssignalsgeneratedbythevictim.Figure7depicts
twicewithdifferentinputs.Whiletheexecutiontimeofbothpaths
thePrefetchXflowforTLB-flush-basedside-channelattacks.
remainsconstant,theorderinwhichthesetwocallsareexecuted
variesbasedonthesecretkey.Specifically,ifthekeyisequaltob’1,
6.1 MonitoringKeyboardActivities
thevalues𝑟𝑒𝑠1arecalculatedandstoredinmemoryfirst,followed
bythecalculationandstorageof𝑟𝑒𝑠2.Theorderisreversedforthe Ourfirstattackfocusesonleakingtheprecisetimingofkeystrokes,
secretbitb’0.As𝑟𝑒𝑠1and𝑟𝑒𝑠2arestoredindifferentpages[50],it i.e.,whenthekeyboardisactivatedbythevictim.Thisleakageis
ispossibletoextracttheprivatekeybydistinguishingtheorderin veryimportantsinceitcanassistinreconstructingtypedwords
| whichthepagesareaccessed. |     |     |     | fromusers[32,49]. |     |     |     |
| ------------------------- | --- | --- | --- | ----------------- | --- | --- | --- |
7

|          |     |                      |     |     | void | victim_client()                         | {                      |     |     |     |
| -------- | --- | -------------------- | --- | --- | ---- | --------------------------------------- | ---------------------- | --- | --- | --- |
|          |     | Repeating the attack |     |     | 1    |                                         |                        |     |     |     |
|          |     |                      |     |     | 2    | int sd = socket(AF_INET,SOCK_STREAM,0); |                        |     |     |     |
| rekcattA |     |                      |     |     | 3    | struct sockaddr_in                      | serveraddr,clientaddr; |     |     |     |
Accessing
Training XPT XPT Status Reset 4 set(serveraddr); //set network protocol
|                    |     | uncacheddata       |                    |     | 5   | socklen_t     | len = sizeof(clientaddr);  |                   |     |     |
| ------------------ | --- | ------------------ | ------------------ | --- | --- | ------------- | -------------------------- | ----------------- | --- | --- |
|                    |     |                    |                    |     |     | int acceptfd  | = accept(sd,(struct        | sockaddr          | ∗)& |     |
| Process 0 | Core 0 |     | Process 0 | Core 0 | Process 0 | Core 0 |     | 6   |               |                            |                   |     |     |
|                    |     |                    |                    |     | 7   |               |                            | clientaddr,&len); |     |     |
|                    |     |                    |                    |     | 8   | int recvbytes | = recv(acceptfd,share_buf, |                   |     |     |
TLB flush
|        |     |                            |     |     | 9    |     | sizeof(share_buf),0); |     |     |     |
| ------ | --- | -------------------------- | --- | --- | ---- | --- | --------------------- | --- | --- | --- |
|        |     |                            |     |     | 10 } |     |                       |     |     |     |
| mitciV |     | Triggering the keyboard or |     |     |      |     |                       |     |     |     |
Starts running Listing 7: Code segment of the victim TCP client. Line 6
Sending/receiving packets
establishestheconnectionwiththeclientandline8receives
| Process 1 | Core 1 |     |     | Process 1 | Core 1 |     |     |     |     |     |     |     |
| ------------------ | --- | --- | ------------------ | --- | --- | --- | --- | --- | --- | --- |
thepackets.
| Figure 7: | The PrefetchX | flow for TLB-flush-based |     | side- |     |     |     |     |     |     |
| --------- | ------------- | ------------------------ | --- | ----- | --- | --- | --- | --- | --- | --- |
channelattacks.
Predefined
synchronization protocol
We first consider a victim that receives user input from the Sender Receiver
keyboard(e.g.,getchar,scanf,etc.),andwritestheinputintoa Core-0 Core-1
|     |     |     |     |     |     | Sending b’0 |     | Sending b’1 |     |     |
| --- | --- | --- | --- | --- | --- | ----------- | --- | ----------- | --- | --- |
sharedmemoryregion.Incaseofkeyboardactivation,theXPT
prefetcherresetsandwillnolongertrigger,asdemonstratedby
|     |     |     |     |     |     | Flush the  | Training XPT  |     |     | Accessing an  |
| --- | --- | --- | --- | --- | --- | ---------- | ------------- | --- | --- | ------------- |
Observation2.Moreconcretely,inthefirststep,theattackertrains
|     |     |     |     |     |     | shared page-0  | on the shared  |     |     | uncached |
| --- | --- | --- | --- | --- | --- | -------------- | -------------- | --- | --- | -------- |
theXPTprefetcherusingthefirst32cachelinesofthesharedpage. from the TLB page-0 data block
Thevictimthenrunsonadifferentcore,andtheattackerrepeatedly
teststheXPTprefetcherstatus.IftheXPTprefetcheristriggered, Figure8:Attackflowofflush-basedcovert-channel.
theattackerobservesanoptimizedcachemiss,whichmeansthe
keyboardisnotactivatedbythevictim,andviceversa.Theattacker
willre-traintheXPTprefetcheronceitsstatusisreset.Wediscuss Flush-based Eviction-based
|     |     |     |     |     |     | SecretPattern | Throughput | ErrorRate | Throughput | ErrorRate |
| --- | --- | --- | --- | --- | --- | ------------- | ---------- | --------- | ---------- | --------- |
theresultsofourattackinSection8.2.
|     |     |     |     |     |     | (111...111)2 | 1.7MiB/s | 0%   | 1.7MiB/s | 0%  |
| --- | --- | --- | --- | --- | --- | ------------ | -------- | ---- | -------- | --- |
|     |     |     |     |     |     | (101...010)2 | 21KiB/s  | 0.5% | 49KiB/s  | 8%  |
Table5:Covert-channelattackexperimentsusingIntelXPT
6.2 MonitoringNetworkTraffic
prefetcher.
Networktrafficanalysisattacks[6,23,39]representaseriousthreat
toonlinesecurity.Althoughanattackercannotgetthecontentof
themessagefromtheinterceptedmessage,anattackercandeter- 7 CROSS-CORECOVERT-CHANNELATTACKS
minethelocationofbothsidesofthecommunication,deanonymize
Attackassumptions.Similartopreviouscovert-channelattacks[11,
communicatingparties,anddeducesensitiveinformationbyob- 20,47],thesenderandreceiversharedataviasharedpageswith
servingthepatternsofdatagrams(e.g.,packettransmissiontiming eachother.Inaddition,thesenderandreceivershouldagreeonpre-
interval,numberoftransmittedpackets,etc.).
definedprotocols,includingsynchronization,encoding,anderror
Weobservethatwheneverapacketissentorreceivedbythe
correctionup-front.
serverorclient,theXPTprefetcherstatusisreset,whichiscaused
byTLBflush(Observation2).Hence,anattackeriscapableof 7.1 Flush-basedCross-CoreCovert-Channel
| trackingthetimingofpackettransmissionviatheXPTprefetcher |     |     |     |     |     | Attack |     |     |     |     |
| -------------------------------------------------------- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- |
status.
Listing7showsthecodesegmentoftheTCPclientasourvictim. Figure8depictsourflush-basedcovert-channelcommunication
Thekeypartsarelines6and8.Line6establishestheconnection flow.Ifthesenderwantstotransmitb’1,itfirsttrainstheXPTon
thefirsthalfpage(first32cachelines)onCore-0.Thetraining
withtheclient,whileline8isusedtoreceivepacketsandwrite
functionisthesameasthetrain()usedinListing1.Ifthesender
thesepacketsintoasharedbuffer.ThevictimonCore-1always
stillwantstotransmitb’1inthenextround,itdoesnotneedto
triestoreceivepackets.Similartoourkeystrokeattack,theattacker
onCore-0alsorepeatedlycheckstheXPTprefetcher’sstatus.In re-traintheXPTastheXPTprefetcherstatuswillberesetonlyif
caseofreceivingapacketbythevictim,theprefetcherresetsand pagesareflushedfromTLBorevictedfromtheprefetcher.Thus,the
theattackerinfersthetimingiftheprefetcherdoesnottrigger.The senderneedstoresettheXPTprefetcherstatusonlyifhe/shewants
totransmitb’0inthenextround.Afterthesendertrained/resetthe
attackerwillre-traintheXPTprefetcherforfollowingroundsof
XPTprefetcher,thereceiver,runningonCore-1,randomlyaccesses
detection.
acachelineonthelasthalfpageofthesharedpagetoobserveifthe
cachemissisanoptimizedLLCmissoranormalLLCmiss(XPTis
Use-Case2.WeexploitObservation2tomonitorvictimac-
nottriggered).Anoptimizedcachemissindicatesthatthesender
tivitiesthatinvolveflushingtheTLBbasedonaspecificand
didnotresettheXPT,sendingb’1,andviceversa.
sensitiveactivity(e.g.,keystrokes,networkpackettransmission,
Weconductedexperimentstoevaluatetheperformanceofour
etc.)andconstructaside-channelattack.
proposedcovert-channelattack,bymeasuringthechannelthrough-
putanderrorrateunderdifferentsecrets.Ourresultsshowthatin
8

Predefined
synchronization protocol MemoryAccessLatency(cycles)
Sender Receiver Scenario keyboard keyboard Distinguishable?
Core-0 Core-1 activated inactivated
Sending b’0 Sending b’1 Accessing an XPT 378 174 ✓
no-XPT 388 383 ✗
uncacheddata block
Table7:Averagememoryaccesslatencywhetherakeystroke
Calling an Training XPT on the
Doing nothing hasoccurredindifferentscenarios:XPTenabledordisabled.
eviction function shared page-0
Figure9:Attackflowofeviction-basedcovert-channel.
8.1 RSASide-ChannelAttacks
InourPoC,wemaptheSquare-and-MultiplyRSAandMontgomery-
ladderRSAcomputationstosharedmemoryandutilizethesyn-
AWSEC2Instance m6i.4xlarge chronizationandpageoffsetdetectiontechniquementionedinSec-
Processor Intel(R)Xeon(R)Platinum8375C tion5.1.TheMontgomery-LadderRSAattackresultisdemonstrated
Architecture IceLake(SunnyCove)
inFigure10(a).Weattemptedtoleaka1-byteprivateexponent
CPUcores 16
LastLevelCache Non-inclusive,54MiB
(0x93).Theresultsshowthatwesuccessfullyrevealedit.Similarto
OperatingSystem Ubuntu20.04 theMontgomery-ladderRSAimplementation,wesuccessfullyleak
ASLR/KASLR Enabled a1-byteprivateexponent(0xf0)fromtheSquare-and-Multiply
DRAM DDR4,64GiB RSA.TheexperimentalresultispresentedinFigure10(b).
Table6:Architectureandsystemconfigurations. ForbothRSAimplementations,weran10iterationstoleakone
bitoftheprivateexponent,whichconsumedapproximately2min-
utestoleak1byte.Aseverybit’scomputationisindependentinRSA
the best case where the sender always sends the same bit (i.e., decryption,itisreasonabletoestimatethatPrefetchXcanbreak
b’1111...1111, see the first row of Table 5), the throughput can a2048-bit(256-byte)RSAengineinapproximately512minutes(8.4
achieveupto1.7MiB/s.Evenintheworstcasewherethebinary hours),whichisapracticaltimewindowforanattacker.
patternalternatesbetween1sand0s,thethroughputisstillableto
achieve21KiB/s.Inbothcases,theerrorrateofthecovert-channel 8.2 KeystrokeSide-ChannelAttack
islessthan1%.Ourresultsdemonstratethefeasibilityandrobust-
Figure11presentstheresultsofourkeystrokeattack.Weclearlyob-
nessofourproposedcovert-channelexploitingObservation2.
servethatinvokingthekeyboardactivityfunctionresetsthestatus
oftheXPTprefetcher.Specifically,thesignificantdifferencebe-
7.2 Eviction-basedCovert-ChannelAttack
tweentheaccesslatencyofnormalLLCmissesandoptimizedLLC
Figure9illustratestheflowoftheeviction-basedcovert-channel missesallowsustoidentifytheprecisetimingofeachkeystroke.
attack.Incontrasttotheflush-basedcovert-channelattack,where Table7summarizestheaveragelatencyweobtainedunderdifferent
thesendertrainstheXPTprefetcherfirst,thereceivertrainsthe scenarios.TheresultsindicatethatwithouttheXPTprefetcher,we
XPTprefetcheronaspecificpage.Thereceiverthencontinually couldnotdetectkeyboardactivity.Thisfindingprovidesfurther
testswhetheritcanstilltriggertheprefetcher.Totransmitb’1,the evidencethattheXPTprefetcherplaysacrucialroleinourattacks.
senderinvokesanevictionfunctionthattrainsanunmappedshared
8.3 NetworkTrafficSide-ChannelAttack
pageandthusevictsthepagefromtheXPTprefetcherthatwas
trainedbythereceiver.Ontheotherhand,thesenderdoesnotneed OurPoCinvolvesaserver-clientsetupwheretheservercommuni-
toexecuteanyfunctiontotransmitb’0,otherthansynchronizing cateswiththevictimclient(seeListing7)viaport8888.Theserver
withthereceiver. randomlygeneratessomepacketsandthensendsthemtotheclient.
Inthebestcasescenario(i.e.,b’11...11),theachievedthroughput TheoutcomeofthisattackisshowninFigure12,whichclearly
isequivalenttothatofaflush-basedcovert-channel,whichcan showsthetimingofpacketreceptionbythevictim.Ourresults
reachupto1.7MiB/s(seeTable5).Importantly,intheworst-case demonstratethatbyobservingtheXPTprefetcherstatus,wecan
scenario(i.e.,b’101...010),theattainedthroughputis49KiB/s,which detectnetworktrafficpatternsthatrevealinformationaboutthe
istwicetherateoftheflush-basedcovert-channel.Itshouldbenoted victim’sbehavior.
thattheerrorrateinthisscenarioishigherthantheflush-based ResolutionAnalysis.Intherealworld,thefrequencyofnet-
covert-channel, as some unexpected page walks are performed workpackettransmissioncanbeveryhigh,andweneedtoensure
whenthereceiverthreadswitchestotheattackerthread(weassume thattrainingtheXPTprefetcherisshorterthanpackettransmis-
thatitcanbecausedbytheOSorthepageprefetcherintroduced sioninterval8.Toinvestigatethis,wesampledIPpacketsfromour
byIntel[3])andevictsthewell-trainedentry. networkinterfacecard(NIC)andrealizedthatthetiminginter-
valbetweensendingtwopacketsareconsistentlyaround26,000
8 EXPERIMENTALRESULTS nanoseconds(ns).TrainingtheXPTrequiresgenerating32LLC
misses,whichtypicallytake400cyclespermiss.Onourexperiment
Experimentalenvironment.WeperformProof-of-Concept(PoC)
side-channel experiments on an Ice Lake machine. The system
8Keystrokesrequiremillisecondstosecondstocomplete,whichtriviallytheXPT
detailsareshowninTable6. prefetchercanbetrainedinthisinterval.
9

Secret bit is 0 Secret bit is 1 Secret bit is 0 Secret bit is 1
 ycnetaL sseccA
|  ycnetaL sseccA | 700 |     |     |         |          | 700 |     |     |         |
| --------------- | --- | --- | --- | ------- | -------- | --- | --- | --- | ------- |
|                 | 600 |     |     |         |          | 600 |     |     |         |
|                 |     |     |     | LLCmiss | )selcyc( |     |     |     | LLCmiss |
| )selcyc(        | 500 |     |     |         |          | 500 |     |     |         |
|                 | 400 |     |     |         |          | 400 |     |     |         |
300
|     |     |       |     |             |     | 300 |       |       | XPT enabled |
| --- | --- | ----- | --- | ----------- | --- | --- | ----- | ----- | ----------- |
|     | 200 |       |     | XPT enabled |     | 200 |       |       |             |
|     | 100 |       |     |             |     | 100 |       |       |             |
|     |     |       |     | LLChit      |     |     |       |       | LLChit      |
|     | 0   |       |     |             |     | 0   |       |       |             |
|     | 0 1 | 2 3 4 | 5 6 | 7 8         |     | 0 1 | 2 3 4 | 5 6 7 | 8           |
Secret bits (ordered from MSB to LSB) Secret bits (ordered from MSB to LSB)
(a) Montgomery-Ladder RSA attack (secret = b'10010011) (b) Square-and-Multiply RSA attack (secret = b'11110000)
Figure10:RSAattackresults.(a)Montgomery-ladderRSA,and(b)Square-and-MultiplyRSA.Thex-axisordersthesecretbits
frommostsignificantbit(MSB)toleastsignificantbit(LSB).Note,thatincaseofb’0assecretbit,awell-trainedentryinXPTis
accessedwhichresultstooptimizedLLCmiss.
|                 | 600                |     |     |          |                 | 600                 |     |     |          |
| --------------- | ------------------ | --- | --- | -------- | --------------- | ------------------- | --- | --- | -------- |
|  ycnetaL sseccA | keyboard triggered |     |     |          |                 | packet transmission |     |     |          |
|                 | 500                |     |     | LLC miss |  ycnetaL sseccA | 500                 |     |     | LLC miss |
)selcyc(
|     | 400 |     |     |     |     | 400 |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
)selcyc(
|     | 300 |     |     | LLC miss      |     | 300 |     |     | LLC miss     |
| --- | --- | --- | --- | ------------- | --- | --- | --- | --- | ------------ |
|     | 200 |     |     | (XPT enabled) |     | 200 |     |     | (XPTenabled) |
|     | 100 |     |     | LLC hit       |     | 100 |     |     |              |
LLC hit
|     | 0    |          |          |     |     | 0    |             |          |     |
| --- | ---- | -------- | -------- | --- | --- | ---- | ----------- | -------- | --- |
|     | 0 10 | 20 30 40 | 50 60 70 | 80  |     | 0 10 | 20 30 40 50 | 60 70 80 | 90  |
Exection timing of the victim (ms)
Execution timing of the victim (ms)
Figure11:Side-channelattacktomonitoruserkeyboardac- Figure12:Side-channelattacktomonitorusernetworktraf-
| tivities. |     |     |     |     | fic. |     |     |     |     |
| --------- | --- | --- | --- | --- | ---- | --- | --- | --- | --- |
platform,1cycletakesabout0.3ns.Therefore,intheworstcase
ittoobservethevictim’saccesstimechanges.Flush+Reloadin-
wherethereisnomemory-levelparallelism(MLP)andout-of-order volvestheattackerflushingacachelineandthenwaitingforthe
(OoO)execution,werequire32×400×0.3ns(or3,840ns)totrain victimtoreloadit,therebyprovidinginsightintothevictim’smem-
theXPTprefetcher.ConsideringtheMLPandOoOexecution,in oryaccesspatterns.Cachetimingside-channelsareoftenreferred
ourreal-worldmeasurements,ittypicallytakesaround3,000ns. ascacheprimitivessincetheyserveasabuildingblockforother
Thesignificantgapbetweenthepackettransmissionintervaland
hardwareside-channels[40,43,44].
theXPTprefetchertrainingtimedemonstratesthatPrefetchX’s Prefetcherattacks.Side-channelattacksintroducedbyprefetch-
resolutionishighenoughtotracknetworkpacketsinreal-world inghavereceivedextensiveattentioninrecentyears.Fromthe
applications. software perspective, modern processors often provide specific
prefetchinstructionsthatprogrammerscanusetoimproveper-
formance.Someattacks[17,30]aimtobypassSupervisorMode
9 POTENTIALMITIGATION
AccessPrevention(SMAP)andKASLRonIntelandAMDproces-
While disabling the prefetcher blocks the leakages of the XPT sors.Theyexploitthetimingof PREFETCHinstructionstoleakthe
prefetcher,itmayintroduceunexpectedperformanceoverheads translationlevelofthevirtualaddressandinferthephysicalmap-
formemory-intensiveapplicationswithhighlyirregularmemory ping.LeakyWay[20]exploitsPREFETCHNTAinIntelprocessorsto
accesses.Amoreefficientmitigationistopartitiontheprefetcher. changethecachestatusandbuildacovert-channelattackthrough
Toachieveleakage-freepartitioning,thesystemneedstopartition conflictedcacheways.Anotherinstruction,PREFETCHW,canleak
theXPTprefetcherbasedontheASIDandtheTIDtagsthatalready cachecoherentstatusandallowcross-coreattacks[21].
existintheXPTstructure.Thisensurestoblockofinformation Onthecontrary,hardwareprefetchersinrealprocessorscannot
leakageacrossdifferentcoresanddifferentthreadsonthesame beexplicitlycontrolledandnormallyrequiremoreunderstanding
throughreverse-engineering.Augury[44]investigatedthedata
core.Note,thatclearingtheXPTprefetcheratcontextswitches
willnotbeeffectiveforcross-coreandhyperthreadingsetups. memory-dependentprefetcherintheAppleM1processortoper-
formout-of-boundsreads.AfterImage[10]studiedIntelIP-based
strideprefetcherthatenablestrackingloadinstructionsofthevic-
10 RELATEDWORK tim.Otherworks[11,41]areeitheralgorithmdependentorcan
onlybeusedasacovert-channel.However,allexistingprefetcher
Cachetimingattacks.Cachetimingside-channelsexploitthe
timingdifferencesbetweencachehitsandcachemisses,enabling side-channelsareinner-coreandtheyaremoreorlessdependenton
attackerstoinferthevictim’smemoryactivities.Therearetwopri- thecachehierarchy,whilePrefetchXcanlaunchcross-coreside-
marytypesofcachetimingside-channels:Prime+Probebased[13, channelandcovert-channelattacksthatdonotrelyonthecache
35,36,45]andFlush+Reloadbased[18,48].ForthePrime+Probe system.Table8showsasummaryofexistingprefetcher/prefetch-
basedside-channelandcovert-channelattacks.
type,theattackerprimesthecachewiththeirdataandthenprobes
10

|     |         |     |       |                 |       | Hardware |            | Requirements                      |                    |
| --- | ------- | --- | ----- | --------------- | ----- | -------- | ---------- | --------------------------------- | ------------------ |
|     | Trigger |     |       | Cross           | Side- |          |            |                                   |                    |
|     |         |     | Paper |                 |       | Leakage  |            |                                   |                    |
|     | From    |     |       | Core? Channel?∗ |       |          |            | fosserddA noitcurtsnI evitalucepS |                    |
|     |         |     |       |                 |       | Source   | sevitimirP | yromeM sserddA ‡ataDfo noitucexE  | mhtiroglA cfiicepS |
ehcaC derahS
|     |     | PrefetchAttack[17,30] |     | ✗   | ✓   | L1/L2/LLC |     |     |     |
| --- | --- | --------------------- | --- | --- | --- | --------- | --- | --- | --- |
erawfoS
|     |     | LeakyWay[20]            |     | ✓   | ✗   | LLC                 |     |     |     |
| --- | --- | ----------------------- | --- | --- | --- | ------------------- | --- | --- | --- |
|     |     | AdversarialPrefetch[21] |     | ✓   | ✓   | LLC                 |     |     |     |
|     |     | FetchingTale[11]        |     | ✗   | ✗   | IP-StridePrefetcher |     |     |     |
IP-StridePrefetcher+
|     |     | UnveilingPrefetcher[40] |     | ✗   | ✓   | L1/L2/LLC |     |     |     |
| --- | --- | ----------------------- | --- | --- | --- | --------- | --- | --- | --- |
erawdraH
Pointer-ChasingPrefetcher+
|     |     | Augury[44] |     | ✗   | ✓   |     |     |     |     |
| --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
L1/L2/LLC
|     |     | AfterImage[10]      |     | ✗   | ✓   | IP-StridePrefetcher |     |     |     |
| --- | --- | ------------------- | --- | --- | --- | ------------------- | --- | --- | --- |
|     |     | PrefetchX(thiswork) |     | ✓   | ✓   | XPTPrefetcher       |     |     |     |
∗✓:canbeusedasbothside-channelandcovert-channelattacks,✗:canonlybeusedasacovert-channelattack
‡ :noneedfortheaddressofthevictim’sdata, :needthepage-grainedaddress, :needthecache-line-grainedaddress
Table8:Summaryofprefetcher/prefetch-basedattacks.
11 CONCLUSION
[9] MohammadBakhshalipour,PejmanLotfi-Kamran,andHamidSarbazi-Azad.
2018.DominoTemporalDataPrefetcher.In2018IEEEInternationalSymposium
| In this work, | we  | uncover details | of the | XPT prefetcher | in Intel |                                                      |     |     |                     |
| ------------- | --- | --------------- | ------ | -------------- | -------- | ---------------------------------------------------- | --- | --- | ------------------- |
|               |     |                 |        |                |          | onHighPerformanceComputerArchitecture(HPCA).131–142. |     |     | https://doi.org/10. |
processors,whichcanbeintentionallytrainedandtriggeredacross 1109/HPCA.2018.00021
differentcoreswithinthesameprocessor.Capitalizingonthese [10] YunChen,LingfengPei,andTrevorECarlson.2023. AfterImage:Leaking
ControlFlowDataandTrackingLoadOperationsviatheHardwarePrefetcher.
features,weintroduceanovelattack,namedPrefetchX,capable InProceedingsofthe28thACMInternationalConferenceonArchitecturalSupport
ofleakingvictims’pageactivities.PrefetchXisacross-coreattack, forProgrammingLanguagesandOperatingSystems,Volume2.16–32.
[11] PatrickCroninandChengmoYang.2019.AFetchingTale:CovertCommuni-
independentofcacheprimitivesasthefoundationofmanyhard-
cationwiththeHardwarePrefetcher.InInternationalSymposiumonHardware
wareattacks.Toachievethis,weconductedanin-depthstudyofthe
|                                                          |     |     |     |     |     | OrientedSecurityandTrust(HOST).101–110. |     | https://doi.org/10.1109/HST.2019. |     |
| -------------------------------------------------------- | --- | --- | --- | --- | --- | --------------------------------------- | --- | --------------------------------- | --- |
| XPTprefetcher,revealingundocumenteddetails.Wedemonstrate |     |     |     |     |     | 8741033                                 |     |                                   |     |
[12] ShuwenDeng,BowenHuang,andJakubSzefer.2022.LeakyFrontends:Security
thatwecanextractsecretkeysinreal-worldSquare-and-Multiply VulnerabilitiesinProcessorFrontends.InInternationalSymposiumonHigh-
andMontgomery-LadderRSAapplications.Furthermore,weapply PerformanceComputerArchitecture(HPCA).53–66. https://doi.org/10.1109/
PrefetchXtoeffectivelyleakthevictim’sdriver-relatedevents. HPCA53966.2022.00013
|     |     |     |     |     |     | [13] Craig Disselkoen, | David | Kohlbrenner, Leo Porter, | and Dean Tullsen. 2017. |
| --- | --- | --- | --- | --- | --- | ---------------------- | ----- | ------------------------ | ----------------------- |
Additionally,weshowcasetheapplicabilityofPrefetchXascross-
Prime+Abort:ATimer-FreeHigh-PrecisionL3CacheAttackusingIntelTSX.
corecovert-channelattacks,achievinghighthroughputandlow In26thUSENIXSecuritySymposium(USENIXSecurity17).USENIXAssociation,
|     |     |     |     |     |     | Vancouver,BC,51–67. |     | https://www.usenix.org/conference/usenixsecurity17/ |     |
| --- | --- | --- | --- | --- | --- | ------------------- | --- | --------------------------------------------------- | --- |
errorrateswhentransmittingsecrets.Finally,weconcludethatthe
technical-sessions/presentation/disselkoen
processorsrequireeitherdisabletheXPTprefetcherorperforma [14] JackDoweck.2006. WhitepaperinsideIntel®Core™Microarchitectureand
thread-basedpartitioningtomitigatethePrefetchXattacks. SmartMemoryAccess.IntelCorporation(2006),72–87.
[15] HongyuFang,SaiSantoshDayapule,FanYao,MilošDoroslovački,andGuru
Venkataramani.2018.Prefetch-guard:Leveraginghardwareprefetchestodefend
againstcachetimingchannels.InSymposiumonHardwareOrientedSecurityand
| REFERENCES |     |     |     |     |     | Trust(HOST).187–190. |     |     |     |
| ---------- | --- | --- | --- | --- | --- | -------------------- | --- | --- | --- |
[16] StefanGast,JonasJuffinger,MartinSchwarzl,GururajSaileshwar,Andreas
[1] [n.d.].GNUPrivacyGuard.https://gnupg.org/. Kogler,SimoneFranza,MarkusKöstl,andDanielGruss.2022.SQUIP:Exploit-
| [2] [n.d.]. | HPCClusterTuningon3rdGenerationIntelXeonScalableProces- |     |     |     |     |     |     |     |     |
| ----------- | ------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
ingtheSchedulerQueueContentionSideChannel.In2023IEEESymposiumon
sors.https://www.intel.com/content/www/us/en/developer/articles/guide/hpc-
cluster-tuning-on-3rd-generation-xeon.html. SecurityandPrivacy(S&P).468–484.
[17] DanielGruss,ClémentineMaurice,AndersFogh,MoritzLipp,andStefanMan-
| [3] [n.d.]. | InconsistencyinTLBmisscounters. |     |     | https://software.intel.com/en- |     |     |     |     |     |
| ----------- | ------------------------------- | --- | --- | ------------------------------ | --- | --- | --- | --- | --- |
gard.2016.Prefetchside-channelattacks:BypassingSMAPandkernelASLR.In
us/forums/software-tuning-performance-optimization-platform-monitoring/ Proceedingsofthe2016ACMSIGSACconferenceoncomputerandcommunications
topic/593830.
security.368–379.
[4] [n.d.].OpenSSL,CryptographyandSSL/TLSToolkit.http://www.openssl.org. [18] DanielGruss,ClémentineMaurice,KlausWagner,andStefanMangard.2016.
| [5] 2021. | UnderstandingIntelSoftwareGuardExtensions(IntelSGX). |     |     |     | https: |     |     |     |     |
| --------- | ---------------------------------------------------- | --- | --- | --- | ------ | --- | --- | --- | --- |
Flush+Flush:AFastandStealthyCacheAttack.InConferenceonDetectionof
//www.intel.sg/content/www/xa/en/architecture-and-technology/software-
IntrusionsandMalware,andVulnerabilityAssessment(DIMVA).279–299.
guard-extensions-enhanced-data-protection.html. [19] DavidGullasch,EndreBangerter,andStephanKrenn.2011. Cachegames–
[6] A.Zinnen-M.HenzeJ.PennekampK.WehrleA.Panchenko,F.LanzeandT.
bringingaccess-basedcacheattacksonAEStopractice.In2011IEEESymposium
Engel.2016.Fingerprintingatinternetscale.In(ISOC)NetworkandDistributed onSecurityandPrivacy.IEEE,490–505.
SystemSecuritySymposium(NDSS).
[20] YananGuo,XinXin,YoutaoZhang,andJunYang.2022.LeakyWay:AConflict-
| [7] Jean-LoupBaerandTien-FuChen.1991. |     |     | AnEffectiveOn-ChipPreloading |     |     |     |     |     |     |
| ------------------------------------- | --- | --- | ---------------------------- | --- | --- | --- | --- | --- | --- |
BasedCacheCovertChannelBypassingSetAssociativity.In202255thIEEE/ACM
SchemetoReduceDataAccessPenalty.InProceedingsofthe1991ACM/IEEE InternationalSymposiumonMicroarchitecture(MICRO).IEEE,646–661.
ConferenceonSupercomputing(Albuquerque,NewMexico,USA)(Supercomputing
[21] YananGuo,AndrewZigerelli,YoutaoZhang,andJunYang.2022.Adversarial
’91).AssociationforComputingMachinery,NewYork,NY,USA,176–186. https: prefetch:Newcross-corecachesidechannelattacks.In2022IEEESymposiumon
//doi.org/10.1145/125826.125932
SecurityandPrivacy(SP).IEEE,1458–1473.
[8] MohammadBakhshalipour,PejmanLotfi-Kamran,andHamidSarbazi-Azad.2017.
AnEfficientTemporalDataPrefetcherforL1Caches.IEEEComputerArchitecture [22] DianaGuttman,MeenakshiArunachalam,VladCalina,andMahmutTaylan
Kandemir.2015.Chapter21-PrefetchTuningOptimizations.InHighPerformance
| Letters16,2(2017),99–102. |     | https://doi.org/10.1109/LCA.2017.2654347 |     |     |     |     |     |     |     |
| ------------------------- | --- | ---------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
11

ParallelismPearls,JamesReindersandJimJeffers(Eds.).401–419. https://doi. [49] K.ZhangandX.Wang.2009. PeepingTomintheneighborhood:Keystroke
org/10.1016/B978-0-12-803819-2.00018-5 eavesdroppingonmulti-usersystems..InUSENIXSecuritySymposium(USENIX
[23] A.Hintz.2002.FingerprintingWebsitesUsingTrafficAnalysis.InWorkshopon Security).
PrivacyEnhancingTechnologies. [50] ZiruiNeilZhao,AdamMorrison,ChristopherWFletcher,andJosepTorrellas.
[24] Intel.[n.d.]. DisclosureofH/WprefetchercontrolonsomeIntelprocessors. 2022.Binoculars:{Contention-Based}{Side-Channel}AttacksExploitingthe
https://radiable56.rssing.com/chan-25518398/article18.html. PageWalker.In31stUSENIXSecuritySymposium(USENIXSecurity22).699–716.
[25] Intel.2019. Intel®64andIA-32ArchitecturesSoftwareDeveloper’sManual.
IntelCorporation(2019).
[26] Intel.2023.Intel®64andIA-32ArchitecturesOptimizationReferenceManual.
IntelCorporation(2023).
[27] PaulKocher,JannHorn,AndersFogh,,DanielGenkin,DanielGruss,Werner
Haas,MikeHamburg,MoritzLipp,StefanMangard,ThomasPrescher,Michael
Schwarz,andYuvalYarom.2019.SpectreAttacks:ExploitingSpeculativeExecu-
tion.InIEEESymposiumonSecurityandPrivacy(S&P’19).
[28] AndrewKwong,DanielGenkin,DanielGruss,andYuvalYarom.2020.RAMBleed:
ReadingBitsinMemoryWithoutAccessingThem.In41stIEEESymposiumon
SecurityandPrivacy(S&P).
[29] LadderRSA2019.Montgomery-LadderRSA.https://github.com/merinjo/RSA-
Montgomery-Ladder-Implementation.
[30] MoritzLipp,DanielGruss,andMichaelSchwarz.2022.Amdprefetchattacks
throughpowerandtime.InUSENIXSecuritySymposium.
[31] MoritzLipp,MichaelSchwarz,DanielGruss,ThomasPrescher,WernerHaas,
AndersFogh,JannHorn,StefanMangard,PaulKocher,DanielGenkin,etal.
2018.Meltdown:Readingkernelmemoryfromuserspace.InUSENIXSecurity
Symposium(USENIXSecurity).973–990.
[32] D.Andriesse-C.GiuffridaH.BosM.Kurth,B.GrasandK.Razavi.2020.PNetCAT:
Practicalcacheattacksfromthenetwork.InIEEESymposiumonSecurityand
Privacy(S&P).
[33] ArmMbed.2019.MbedTLS:Anopensource,portable,easytouse,readableand
flexibleSSLlibrary.ARMHoldingsplc(2019).
[34] OpenSSL Ladder RSA 2011. OpenSSL 1.01 Montgomery-
Ladder RSA. https://github.com/openssl/openssl/blob/
46ebd9e3bb623d3c15ef2203038956f3f7213620/crypto/ec/ec2_mult.c.
[35] DagArneOsvik,AdiShamir,andEranTromer.2006.Cacheattacksandcoun-
termeasures:thecaseofAES.InCryptographers’trackattheRSAconference.
1–20.
[36] ColinPercival.2005.Cachemissingforfunandprofit.
[37] IvanPuddu,MoritzSchneider,MiroHaller,andSrdjanČapkun.2021.Frontal
attack:leakingcontrol-flowinSGXviatheCPUfrontend. USENIXSecurity
Symposium(USENIXSecurity),663–680.
[38] AdityaRohan,BiswabandanPanda,andPrakharAgarwal.2020.ReverseEngi-
neeringtheStreamPrefetcherforProfit.InEuropeanSymposiumonSecurityand
PrivacyWorkshops(EuroS&PW).682–687.
[39] N.Vallina-RodriguezS.Siby,M.JuárezandC.Troncoso.2018.Dnsprivacynot
soprivate:thetrafficanalysisperspective.InPrivacyEnhancingTechnologies
Symposium(PETS).
[40] YoungjooShin,HyungChanKim,DokeunKwon,JiHoonJeong,andJunbeom
Hur.2018. UnveilingHardware-BasedDataPrefetcher,aHiddenSourceof
InformationLeakage.InConferenceonComputerandCommunicationsSecurity
(CCS).131–145. https://doi.org/10.1145/3243734.3243736
[41] YoungjooShin,HyungChanKim,DokeunKwon,JiHoonJeong,andJunbeom
Hur.2018.Unveilinghardware-baseddataprefetcher,ahiddensourceofinfor-
mationleakage.InProceedingsofthe2018ACMSIGSACConferenceonComputer
andCommunicationsSecurity.131–145.
[42] StephenSomogyi,ThomasFWenisch,AnastassiaAilamaki,BabakFalsafi,and
AndreasMoshovos.2006.Spatialmemorystreaming.ACMSIGARCHComputer
ArchitectureNews34,2(2006),252–263.
[43] StephanvanSchaik,MarinaMinkin,AndrewKwong,DanielGenkin,andYuval
Yarom.2021.CacheOut:LeakingdataonIntelCPUsviacacheevictions.(2021),
339–354. https://doi.org/10.1109/SP40001.2021.00064
[44] JoseRodrigoSanchezVicarte,MichaelFlanders,RiccardoPaccagnella,Grant
Garrett-Grossman,AdamMorrison,ChristopherWFletcher,andDavidKohlbren-
ner.2022.Augury:Usingdatamemory-dependentprefetcherstoleakdataat
rest.In2022IEEESymposiumonSecurityandPrivacy(SP).IEEE,1491–1505.
[45] DaimengWang,ZhiyunQian,NaelAbu-Ghazaleh,andSrikanthVKrishna-
murthy.2019.Papp:Prefetcher-awareprimeandprobeside-channelattack.In
DesignAutomationConference(DAC).1–6.
[46] HaochengXiaoandSamAinsworth.2023.HackyRacers:ExploitingInstruction-
LevelParallelismtoGenerateStealthyFine-GrainedTimers.InProceedingsof
the28thACMInternationalConferenceonArchitecturalSupportforProgramming
LanguagesandOperatingSystems,Volume2.354–369.
[47] YoutaoZhang-JunYangYananGuo,AndrewZigerelli.2022.AdversarialPrefetch:
NewCross-CoreCacheSideChannelAttacks.InIEEESymposiumonSecurity
andPrivacy(S&P).
[48] YuvalYaromandKatrinaFalkner.2014.FLUSH+RELOAD:Ahighresolution,
lownoise,L3cacheside-channelattack.InUSENIXSecuritySymposium(USENIX
Security).719–732.
12
