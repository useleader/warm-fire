---
publish: true
---

ABACuS: All-Bank Activation Counters for Scalable
and Low Overhead RowHammer Mitigation
Ataberk Olgun, Yahya Can Tugrul, Nisa Bostanci, Ismail Emir Yuksel,
Haocong Luo, Steve Rhyner, Abdullah Giray Yaglikci, Geraldo F. Oliveira,
and Onur Mutlu, ETH Zurich
https://www.usenix.org/conference/usenixsecurity24/presentation/olgun
This paper is included in the Proceedings of the
33rd USENIX Security Symposium.
August 14–16, 2024 • Philadelphia, PA, USA
978-1-939133-44-1
Open access to the Proceedings of the
33rd USENIX Security Symposium
is sponsored by USENIX.

|     |     |          | ABACuS: |     |     | All-Bank |     | Activation |     | Counters |            |     |     |     |
| --- | --- | -------- | ------- | --- | --- | -------- | --- | ---------- | --- | -------- | ---------- | --- | --- | --- |
|     | for | Scalable |         | and | Low | Overhead |     | RowHammer  |     |          | Mitigation |     |     |     |
AtaberkOlgun YahyaCanTugrul NisaBostanci IsmailEmirYuksel HaocongLuo
SteveRhyner AbdullahGirayYaglikci GeraldoF.Oliveira OnurMutlu
ETHZurich
|     |     |     | Abstract |     |     |     |     | 1 Introduction |     |     |     |     |     |     |
| --- | --- | --- | -------- | --- | --- | --- | --- | -------------- | --- | --- | --- | --- | --- | --- |
ModernDRAMchipsarevulnerabletoRowHammer[1–
13],whererepeatedlyopeningandclosing(i.e.,activatingand
precharging,orsimplyhammering)aDRAMrow(aggressor
WeintroduceABACuS,anewlow-costhardware-counter-
basedRowHammermitigationtechniquethatperformance-, row) at a high enough rate can cause bitflips in physically
energy-,andarea-efficientlyscaleswithworseningRowHam- nearby rows (victim rows). DRAM chips become more
mer vulnerability. We observe that both benign workloads vulnerable to RowHammer as DRAM storage density in-
andRowHammerattackstendtoaccessDRAMrowswith creases across DRAM generations [2,4,14–19]. The mini-
mumnumberofrowactivationsneededtoinduceaRowHam-
thesamerowaddressinmultipleDRAMbanksataroundthe
sametime.Basedonthisobservation,ABACuS’skeyidea mer bitflip, i.e., the RowHammer threshold (N ), has re-
RH
istouseasinglesharedrowactivationcountertotrackacti- duced by more than an order of magnitude in less than a
decade[14].1Asmanypriorworksdemonstrateonrealsys-
vationstotherowswiththesamerowaddressinallDRAM
banks.Unlikestate-of-the-artRowHammermitigationmech- tems[1,2,4,15,20–83],RowHammerbitflipscanleadtosecu-
rityexploitsthat1)takeoverasystem,2)leaksecurity-critical
anismsthatimplementaseparaterowactivationcounterfor
eachDRAMbank,ABACuSimplementsfewercounters(e.g., orprivatedata,and3)manipulatesafety-criticalapplications’
onlyone)totrackanequalnumberofaggressorrows. behavior in undesirable ways. As a result,a large body of
work[1,15,19,38,44,55,84–88,88–135]proposesmitigation
| Our | comprehensive |     | evaluations |     | show that | ABACuS | se- |     |     |     |     |     |     |     |
| --- | ------------- | --- | ----------- | --- | --------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
mechanismstopreventRowHammerbitflips.
curelypreventsRowHammerbitflipsatlowperformance/en-
KeyProblem.Manypriorworks(e.g.,[1,98,102,106,107,
| ergy overhead |     | and low | area | cost. | We compare | ABACuS |     |     |     |     |     |     |     |     |
| ------------- | --- | ------- | ---- | ----- | ---------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
110,112,116,117,125,134,135])proposeusingasetofcoun-
| to four | state-of-the-art |     | mitigation | mechanisms. |     | At  | a near- |     |     |     |     |     |     |     |
| ------- | ---------------- | --- | ---------- | ----------- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
terstotracktheactivationcountsofpotentialaggressorrows
futureRowHammerthresholdof1000,ABACuSincursonly
|     |     |     |     |     |     |     |     | (counter-based |     | mechanisms). |     | Using | counters to determine |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | ------------ | --- | ----- | --------------------- | --- |
0.58%(0.77%)performanceand1.66%(2.12%)DRAMen-
rowsthatreachclosetoRowHammerthresholdsandtaking
ergyoverheads,averagedacross62single-core(8-core)work- mitigatingactionsaccordinglycanpreventRowHammerbit-
| loads,requiring |     | only | 9.47 KiBofstorageperDRAMrank. |     |     |     |     |     |     |     |     |     |     |     |
| --------------- | --- | ---- | ----------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
flipsatlowperformanceandenergyoverheads.Unfortunately,
| At the | RowHammer |     | threshold | of 1000,the |     | best prior | low- |     |     |     |     |     |     |     |
| ------ | --------- | --- | --------- | ----------- | --- | ---------- | ---- | --- | --- | --- | --- | --- | --- | --- |
mitigationmechanismsthatrelyoncountersfacetwoscalabil-
| area-cost | mitigation |     | mechanism | incurs | 1.80% | higher | aver- |     |     |     |     |     |     |     |
| --------- | ---------- | --- | --------- | ------ | ----- | ------ | ----- | --- | --- | --- | --- | --- | --- | --- |
itychallenges.First,theyneedtoimplementanincreasingly
| age performance |     | overhead | than | ABACuS,while |     | ABACuS |     |     |     |     |     |     |     |     |
| --------------- | --- | -------- | ---- | ------------ | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
largenumberofcounterstotrackallpotentialaggressorrows
requires2.50×smallerchipareatoimplement.Atafuture
|     |     |     |     |     |     |     |     | asN reduces.Thisisbecauseanattackercanconcurrently |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- | --- | --- | --- |
RH
RowHammerthresholdof125,ABACuSperformsverysimi-
|                                                    |     |     |     |     |     |     |     | hammer   | more DRAM | rows     | when       | N   | is smaller. | Second,   |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | -------- | --------- | -------- | ---------- | --- | ----------- | --------- |
| larlyto(within0.38%oftheperformanceof)thebestprior |     |     |     |     |     |     |     |          |           |          |            |     | RH          |           |
|                                                    |     |     |     |     |     |     |     | the area | overhead  | of these | mechanisms |     | linearly    | increases |
performance-andenergy-efficientRowHammermitigation
|           |       |           |     |        |         |            |     | with the    | numberof | DRAM | banks    | in    | the system,and | mod-    |
| --------- | ----- | --------- | --- | ------ | ------- | ---------- | --- | ----------- | -------- | ---- | -------- | ----- | -------------- | ------- |
| mechanism | while | requiring |     | 22.72× | smaller | chip area. | We  |             |          |      |          |       |                |         |
|           |       |           |     |        |         |            |     | ern systems | continue | to   | use more | banks | to scale       | up both |
showthatABACuS’sperformancescaleswellwiththenum-
|     |     |     |     |     |     |     |     | DRAM | capacity | andbandwidth |     | [136–152]. | A smallsetof |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | -------- | ------------ | --- | ---------- | ------------ | --- |
berofDRAMbanks.AttheRowHammerthresholdof125,
priorworks(e.g.,[1,97,100,106,116,133])aimtomitigate
ABACuSincurs1.58%,1.50%,and2.60%performanceover-
1Forexample,RowHammerthreshold(NRH)isonly4.8Kand10Kfor
headsfor16-,32-,and64-banksystemsacrossallsingle-core
somenewerLPDDR4andDDR4DRAMchips(manufacturedin2019–
workloads,respectively.ABACuSisfreelyandopenlyavail-
2020),whichis14.4×and6.9×lowerthantheNRHof69.2Kforsomeolder
ableathttps://github.com/CMU-SAFARI/ABACuS.
DRAMchips(manufacturedin2010–2013)[14].
| USENIX Association |     |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1579 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

RowHammeratlowareaoverhead.Unfortunately,toachieve and 62 8-core multi-programmed workloads from SPEC
low area overhead,these works cause (prohibitively) large CPU2006,SPECCPU2017,TPC,MediaBench,andYCSB
performanceoverheadsasDRAMchipsbecomemorevulner- benchmarksuitesandmemory-intensivemicrobenchmarks,
abletoRowHammer[2,4,14–19].Therefore,itisimportant andii)areaoverheadusingCACTI[157].WemodelABA-
toprovideascalableRowHammersolutionwhoseareaover- CuS’s hardware design (RTL) in Verilog and evaluate its
headandperformanceoverheadremainlowasDRAMchips circuitareaandlatencyoverheadsusingmodernASICdesign
becomemorevulnerabletoRowHammer. tools.WecompareABACuStofourstate-of-the-artRowHam-
Our goal is to prevent RowHammer bitflips at low per- mer mitigation mechanisms. We make four key observa-
formance,energy,andareaoverheadsinmodernandfuture tions from ourevaluation. First,ata near-future RowHam-
DRAM-basedsystemswithhighRowHammervulnerability. merthresholdof1K,ABACuSincursonly1)0.58%average
Tothisend,weproposeanewlow-costandscalablecounter- (32.00%maximum)performanceand1.66%average(2.02×
basedRowHammermitigationmechanism,All-BankActiva- maximum)DRAMenergyoverheadsacross62single-core
tionCountersforScalableandlowoverheadRowHammer workloads,and2)0.77%average(32.97%maximum)perfor-
mitigation(ABACuS).ABACuSleveragesourkeyobserva- manceand2.12%average(2.17×maximum)DRAMenergy
tiononbenignworkloads’andRowHammerattacks’mem- overheadsacross628-coreworkloadmixescomparedtoasys-
oryaccesspatterns.Manyworkloads(bothbenignworkloads temwithoutanyRowHammerprotection,whilerequiringonly
andRowHammerattacks)tendtoaccessDRAMrowswith 18.93KiBofstorage.Second,ABACuSscaleswellintothefu-
the same row address in multiple DRAM banks at around tureforDRAMchipswithextremelylowRowHammerthresh-
thesametimebecausei)modernmemoryaddressmapping olds:e.g.,ataRowHammerthresholdofonly125,ABACuS’s
schemesinterleaveconsecutivecacheblocksacrossdifferent performanceandenergyoverheadsare1.45%and1.27%,re-
banks(§2.1)andii)workloadstendtoaccesscacheblocks spectively,onaverageforsingle-coreworkloads,whilerequir-
in close proximityaroundthe same time due to the spatial ing151.41KiBofstorage.Third,attheN RH of125,ABACuS
localityintheirmemoryaccesses(§3). performsverysimilarlytothebestpriorperformance-and
Leveraging this observation, ABACuS’s key idea is to energy-efficientRowHammermitigationmechanismwhile
use a single shared activation counter to track activations requiring22.72×smallerchiparea.Fourth,ABACuSscales
to the rows with the same row ID (i.e.,same row address) wellwiththenumberofDRAMbanks.AttheN RH of125,
in all DRAM banks. By doing so, ABACuS i) retains the ABACuSincurs1.58%,1.50%,and2.60%performanceover-
performance-andenergy-efficiencybenefitsofcounter-based headsfor16-,32-,and64-banksystemsacrossall62single-
RowHammermitigationmechanisms(§9),andii)incurslow coreworkloads,respectively.OurevaluationofABACuS’s
areacost,asitrequiresonlyasmallnumberofcountersto circuitlatencyshowsthatABACuScouldbeimplementedoff
keeptrackofmanyaggressorrows (e.g.,2720 counters in- thecriticalpathinthememorycontroller.ABACuS’slatency
steadof43520[102]atanN of1000). (1.22ns)iseasilyoverlappedwiththelatency(2.5ns[158])
RH
ofissuingtwosuccessiveDRAMrowactivationcommands
KeyMechanism.Atahighlevel,ABACuSmapsDRAM
(tRRD). We open source our simulation infrastructure and
rows that have the same row ID in different banks (which
alldatasetsathttps://github.com/CMU-SAFARI/ABACuS
wecallsiblingrows)tothesameABACuScounter.Ineach
toenablereproducibilityandhelpfutureresearch.Detailed
ABACuS counter,we store i) the sibling activation vector
analyses and data,including memory intensity characteris-
thatcontainsasmanybitsasthenumberofbanks(e.g.,16
ticsofeachevaluatedworkload,per-workloadperformance
bitsifthereare16banks),andii)therowactivationcount.
and energy results, key configuration parameters of evalu-
ABACuS tracks only the maximum (i.e.,the worst) activa-
atedstate-of-the-artRowHammermitigations,andsecurity
tioncountoftheactivationcountsofsiblingrows.Beforethe
analysisofsomeoftheexistingin-DRAMRowHammermiti-
activation countvalue reaches N ,ABACuS preventively
RH
gations,areinanextendedversionofthispaper[159].
refreshes allpotentialvictim rows ofeachsibling row and
Thisworkmakesthefollowingkeycontributions:
thuspreventsanypotentialRowHammerbitflips.ABACuS
tracksthemaximumactivationcountofsiblingrowswithout
• Weshowthatitispossibletoleveragebenignworkload
unnecessarilyincrementingtherowactivationcountforeach
accesspatternstopreventRowHammerbitflipsatlow
siblingrow.Thisway,ABACuSreducesthenumberofunnec-
overheadintermsofperformance,energy,andarea,even
essarypreventiverefreshoperations,loweringitsperformance
forDRAMchipswithveryhighRowHammervulnera-
andenergyoverheads.ABACuSiscompletelyimplemented
bility.
insidethememorycontrollerandthereforedoesnotrequire
anymodificationstoexistingDRAMchipsorsoftware. • We develop ABACuS, a new low-cost and scalable
Key Results. We rigorously evaluate ABACuS’s i) im- RowHammer mitigation mechanism. ABACuS pre-
pactonsystemperformanceandenergyconsumptionusing vents RowHammerbitflips with small average perfor-
cycle-accurate memory system simulations (with Ramula- manceandenergyoverheadswhileusingsignificantly
tor [153–156]), executing a diverse set of 62 single-core smallerin-processor-chipstoragecomparedtostate-of-
1580 33rd USENIX Security Symposium USENIX Association

the-artRowHammermitigationmechanismsatverylow (e.g., [162–164]). Upon receiving a REF command, the
RowHammerthresholds(i.e.,1Kto125). DRAMchipinternallyrefreshesmultipleDRAMrowsfora
|     |     |     |     |     |     | refreshlatency(t | RFC | )amountoftime. |     |     |     |
| --- | --- | --- | --- | --- | --- | ---------------- | --- | -------------- | --- | --- | --- |
• Weevaluatetheperformance,energy,andareaoverheads
DRAMTimingParameters.Thememorycontrollersched-
offourstate-of-the-artRowHammermitigationmecha-
ulesDRAMcommandsaccordingtocertaintimingparame-
nisms.WeshowthatABACuSperformsverysimilarlyto
terstoguaranteecorrectoperation[136,143,148,158,160,
thebest-performingmechanismatamuchsmaller(e.g., 162–166].Inadditiontot ,t ,andt ,twoothertim-
|     |     |     |     |     |     |     |     | REFW | REFI RFC |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | -------- | --- | --- |
22.72×)chipareaoverhead.WemodelABACuS’shard-
ingparametersarerelevantforthiswork:i)theminimumtime
waredesign(RTL)inVerilogandevaluateitshardware
betweentwoconsecutiverowactivationstargetingthesame
| complexityusingmodernASICdesigntools. |     |     |     |     |     | bank(t |     |     |     |     |     |
| ------------------------------------- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- |
RC )andii)theminimumtimebetweentwoconsecutive
|              |     |     |     |     |     | rowactivationstargetingthesamerank(t |     |     |     | )(Fig.1b). |     |
| ------------ | --- | --- | --- | --- | --- | ------------------------------------ | --- | --- | --- | ---------- | --- |
| 2 Background |     |     |     |     |     |                                      |     |     | RRD |            |     |
Bank-LevelParallelism.Mainmemoryaccessesthattarget
2.1 DRAMOrganizationandOperation differentbankscanproceedconcurrently[136].Modernad-
dressmappingschemes(e.g.,[167–171])aimtointerleave
DRAMOrganization(Fig.1a).Amemorychannelconnects
consequentlyaddressedcacheblocksacrossdifferentbanks
theprocessor(CPU)toaDRAMrank,asetofDRAMchips
toexploitbank-levelparallelism[136,138].
| workinginlockstep. |     | ADRAMchiphasmultiplebankseach |     |     |     |     |     |     |     |     |     |
| ------------------ | --- | ----------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ofwhichcontaininga2DDRAMcellarrayintheformof 2.2 RowHammerMitigationTechniques
| rows and    | columns. | The DRAM          | cell | stores       | one bit of data |            |           |          |             |     |           |
| ----------- | -------- | ----------------- | ---- | ------------ | --------------- | ---------- | --------- | -------- | ----------- | --- | --------- |
|             |          |                   |      |              |                 | To prevent | RowHammer | bitflips | and protect |     | computing |
| in the form | of       | electrical charge | in   | a capacitor. | The access      |            |           |          |             |     |           |
transistor,controlledbythewordline,connectsthecelltothe systemsagainstRowHammerattacks,priorworkspropose
differentRowHammermitigationmechanisms[1,15,19,38,
bitlinewhichisconnectedtotherowbuffer.
|     |     |     |     |     |     | 44,55,84–88,88–135]. |     | These | works trigger | their | counter- |
| --- | --- | --- | --- | --- | --- | -------------------- | --- | ----- | ------------- | ----- | -------- |
measure(e.g.,refreshingpotentialvictimrowsorthrottling
accessestopotentialaggressorrows)basedoneitheri)there-
sultofaprobabilisticprocedureorii)trackingthenumberof
timesDRAMrowsareactivated(i.e.,rowactivationcounts).
|     |     |     |     |     |     | While probabilistic |      | procedures          | can be implemented |             | atlow |
| --- | --- | --- | --- | --- | --- | ------------------- | ---- | ------------------- | ------------------ | ----------- | ----- |
|     |     |     |     |     |     | chip area cost,     | they | incur prohibitively | large              | performance |       |
overheadswhenconfiguredforsub-1KRowHammerthresh-
Figure1:DRAMorganization(a),timingparameters(b),and old(N RH )values[14,104,116].Priorworksproposeseveral
RowHammer(c) differentrowactivationtrackingmechanismsthatdetectthe
frequently-activatedsetofrows.Unfortunately,whileprovid-
| Operation. | ThememorycontrollerissuesDRAMcommands, |     |     |     |     |                        |     |          |               |             |     |
| ---------- | -------------------------------------- | --- | --- | --- | --- | ---------------------- | --- | -------- | ------------- | ----------- | --- |
|            |                                        |     |     |     |     | ing better performance |     | than the | probabilistic | mechanisms, |     |
e.g.,rowactivation(ACT),bankprecharge(PRE),dataread thechipareaoverheadoftheserow-activation-count-tracking
(RD),datawrite(WR),andrefresh(REF)toservememory
mechanismssignificantlyincreasesasDRAMchipsbecome
requests.Todoso,thememorycontrollerfirstissuesanACT
morevulnerabletoRowHammer[13,106].2
commandwiththebankaddress(i.e.,bankID)androwad-
FrequentItemCounting.Anaïve,area-inefficienttracking
dress(i.e.,rowID)correspondingtothememoryrequest’s
methodtodetectpossibleaggressorrowsistostoretheacti-
address,activatingtherow.Whenarowisactivated,itsdata vationcountofeachDRAMrowinadedicatedcounter.How-
iscopiedtotherowbuffer.Thememorycontrollercanread-
ever,thismethodleadstoimpracticalon-chipareaoverheads
/writedataatcacheblock(512bits)granularityfrom/tothe
whenusedtoprotectmodern,high-densityDRAMmodules.
rowbufferusingasequenceofRD/WRcommands.Asubse-
Forexample,8-bitcountersforamodernDDR4rankwith
quentaccesstothesamerow(i.e.,arowhit)canbeserved
221 rows[158]wouldrequire2MiBon-chipstorageanda
quicklywithoutissuinganotherACT.Toaccessanotherrow
neweranddenserDDR5rankwith223rows[163]wouldre-
(i.e.,toservearowconflict),thememorycontrollermustissue
quireanevenlarger8MiBon-chipstorage.Fortunately,the
aprechargecommandandclosetheopenrow.
problemoftrackingthefrequentlyactivatedDRAMrowscan
| Periodic | Refresh. | DRAM | cells are | inherently | leaky and |     |     |     |     |     |     |
| -------- | -------- | ---- | --------- | ---------- | --------- | --- | --- | --- | --- | --- | --- |
beinterpretedasafrequentitemcountingproblemandcanbe
| thus lose | the stored | electrical | charge | over | time. To main- |     |     |     |     |     |     |
| --------- | ---------- | ---------- | ------ | ---- | -------------- | --- | --- | --- | --- | --- | --- |
solvedusingmorearea-efficientalgorithms.Forexample,the
| tain data | integrity, | a DRAM | cell is | periodically | refreshed |     |     |     |     |     |     |
| --------- | ---------- | ------ | ------- | ------------ | --------- | --- | --- | --- | --- | --- | --- |
Misra-Griesalgorithm[172]canbeimplementedinhardware
| every refresh |     | window (t | ), which | is  | typically 64ms |     |     |     |     |     |     |
| ------------- | --- | --------- | -------- | --- | -------------- | --- | --- | --- | --- | --- | --- |
REFW to accurately track aggressorrows using a relatively small
(e.g.,[158,160,161])or32ms(e.g.,[162–164]).Totimely
2Hydra[106]isanexceptioninthisclassificationasitincursalowchip
| refresh | all cells,the | memory | controller | periodically | issues |     |     |     |     |     |     |
| ------- | ------------- | ------ | ---------- | ------------ | ------ | --- | --- | --- | --- | --- | --- |
areaoverheadwhiletrackingrowactivationcounts.Hydraachievesthisby
| a refresh | (REF)     | command | every refresh         |     | interval (t ), |                                                           |     |     |     |     |     |
| --------- | --------- | ------- | --------------------- | --- | -------------- | --------------------------------------------------------- | --- | --- | --- | --- | --- |
|           |           |         |                       |     | REFI           | storingthecountersintheDRAMarrayandcachingtheminthememory |     |     |     |     |     |
| which is  | typically | 7.8µs   | (e.g., [158,160,161]) |     | or 3.9µs       |                                                           |     |     |     |     |     |
controller.§3discussesHydra’sscalabilitylimitations.
| USENIX Association |     |     |     |     |     |     | 33rd USENIX Security Symposium    1581 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

numberofcounterstodetectpotentialaggressorrows,andits Hydra’smechanismhastwokeydrawbacks.First,Hydra
variantsareadoptedbyseveralpriorRowHammermitigation overestimatestheactivationcountsofDRAMrows,causing
mechanisms[102,107,110,112,117]. alargenumberofunnecessaryrefreshoperations.According
3 Motivation tooursystem-levelsimulations(in§9),approximatelyhalfof
Hydra’spreventiverefreshoperationsareunnecessaryfor62
| Repeatedly | activating | and precharging | (hammering) | a   |     |     |     |     |
| ---------- | ---------- | --------------- | ----------- | --- | --- | --- | --- | --- |
single-coreworkloadsataverylowRowHammerthresholdof
| DRAMrowatleastN | RH  | timesinarefreshwindowinduces |     |     |     |     |     |     |
| --------------- | --- | ---------------------------- | --- | --- | --- | --- | --- | --- |
125.HydraoverestimatesactivationcountsofDRAMrows
oneormultiplebitflipsinthatrow.AsDRAMchipsbecome
morevulnerabletoRowHammer(i.e.,thechip’srowshave because modern memory-intensive workloads can rapidly
increasethegroupcountervaluetothegroupcountthreshold
| smallerN | RH values),fewerhammerscaninducebitflips.Even |     |     |     |     |     |     |     |
| -------- | --------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
valuewithonlyafewactivationstoeachDRAMrowinthe
thoughthenumberofactivateandprechargecommandsthat
group.Suchworkloadscancausetheactivationcountersto
thememorycontrollercanissueinarefreshwindowremains
|           |               |                 |          | overestimate | the actualactivation | countofeachrowin |     | the |
| --------- | ------------- | --------------- | -------- | ------------ | -------------------- | ---------------- | --- | --- |
| the same, | more rows can | be concurrently | hammered | N            |                      |                  |     |     |
RH
times at a smaller N . As RowHammer vulnerability in- groupbyupto396(inHydra’sdefaultconfigurationforan
RH
|                                                       |     |     |     | N of1K).Therefore,Hydraoftenmistakenlyrefreshesthe |     |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- | --- |
| creases,state-of-the-artcounter-basedRowHammermitiga- |     |     |     | RH                                                 |     |     |     |     |
neighborsofmanyrowsthatwillnotbeactivatedasmanyas
tionmechanismsneedtotrackmoreDRAMrowsandimple-
|     |     |     |     | N RH times.Second,supposethecounterofanaccessedrow |     |     |     |     |
| --- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- | --- |
mentmoreactivationcounters.
isnotcachedinthememorycontroller.Inthatcase,Hydra
Acommonmethodofincreasingmemorybandwidthand
|     |     |     |     | needs to | fetch the counter | from the main | memory, | which |
| --- | --- | --- | --- | -------- | ----------------- | ------------- | ------- | ----- |
capacityistoincreasethenumberofDRAMbanks[136–141].
incursadditionalmemorylatencyforwritingbacktheevicted
| However,as | the numberofbanks | increases,counter-based |     |     |     |     |     |     |
| ---------- | ----------------- | ----------------------- | --- | --- | --- | --- | --- | --- |
counterandfetchingthenewcounter.Bothofthesedrawbacks
mechanismsincurincreasingchipareaoverhead.
incursignificantperformanceandenergyoverheadsasHydra
LimitationsofPriorWork.Severalpriorworks[1,98,100,
increasesthememorylatency(i.e.,thetimeittakestoserve
105,106,126,173]aimtomitigateRowHammeratlowarea
|          |                 |           |                       | amemorydemandrequest)by23.67%onaverageatN |     |     |     | =   |
| -------- | --------------- | --------- | --------------------- | ----------------------------------------- | --- | --- | --- | --- |
| overhead | by implementing | a limited | set of row activation |                                           |     |     |     | RH  |
125(asweshowindetailin§9).
counters(i.e.,fewercountersthantherearerowsintheDRAM
module)atthecostofreducedtrackingaccuracy.However,a
|     |     |     |     | 3.1 MotivationalAnalysisforABACuS |     |     |     |     |
| --- | --- | --- | --- | --------------------------------- | --- | --- | --- | --- |
RowHammermitigationcountermeasure(e.g.,preventively
refreshingpotentialvictimrowsorthrottlingaccessestopo- Weinvestigatememoryaccesspatternsofmodernmemory-
tential aggressor rows) fundamentally consumes memory intensiveworkloadsandexistingRowHammerattacks.We
bandwidth and reduced tracking accuracy exacerbates the observethattheyactivateDRAMrowswiththesamerowad-
numberofcountermeasuresdeployedbythemitigationmech- dressinmultipleDRAMbanks(i.e.,siblingrows)ataround
anism.Thus,thesemechanismsoccupyasignificantportion the same time. This observation motivates us to design a
ofmain memory bandwidthandincurlarge system perfor- performance-,energy,andarea-efficientDRAMrowactiva-
mance and DRAM energy overheads at small N values. tioncounttrackingmechanismbyimplementingoneshared
RH
|     |     |     |     | activation | counter |     |     |     |
| --- | --- | --- | --- | ---------- | ------- | --- | --- | --- |
Toprovidemoreinsightintothisproblem,wedescribethe for all sibling rows. Implementing one
key drawback of one such state-of-the-art mechanism,Hy- sharedactivationcounterreducesthenumberofcountersre-
dra[106],asaconcreteexample.Hydra[106]maintainsthe quiredtotrackaggressorrows(andthustheareacost)bya
activationcountofeachDRAMrowinaphysicallocation factorofthenumberofbanks(e.g.,16inDDR4[158])com-
inmainmemory(i.e.,intheDRAMchips).Tominimizethe paredtotheaggressorrowtrackingmechanismsusedbythe
state-of-the-artperformance-andenergy-efficientRowHam-
overheadsoffetchingthecountersfromthemainmemory,
Hydraimplementsafilteringandcachinglogic.Thefiltering mermitigations(e.g.,Graphene[102],Panopticon[134],and
logicgroupsanumberof(e.g.,125)DRAMrowsintorow PRHT[135]).However,thesharedcountermaynotaccurately
groups and assigns a counterto each row group called the representtheactivationcountsofmultiplesiblingrowsbe-
groupcounter.DRAMrowactivationsupdateonlythecorre- causethesharedcountercanstoreonlyoneactivationcount.
|     |     |     |     | Misrepresenting | the activation | counts | of sibling rows | may |
| --- | --- | --- | --- | --------------- | -------------- | ------ | --------------- | --- |
spondinggroupcountersatthebeginningofarefreshwindow.
Whenagroupcounterexceedsapredeterminedgroupcount induceperformanceandenergyoverheadsduetounnecessary
threshold(e.g.,400),thegroupcounter’svalueiscopiedto victimrowrefreshoperations.Intheremainderofthissec-
theactivationcountersofrowsinthatgroup,suchthatHydra tion,weshowthat1)siblingrowsareactivatedataroundthe
can track each row’s activation count individually and de- sametime,and2)asharedactivationcountercanaccurately
ployitscountermeasure(preventivelyrefreshingvictimrows) representtheactivationcountsofmultiplesiblingrows.
moreaccurately(e.g.,insteadofpreventivelyrefreshingall We simulate 34 memory-intensive workloads,each hav-
125rowsinagroup,HydracanrefreshoneorseveralDRAM ing more than two row buffer misses per kilo instructions
rowsthatareactivatedfrequentlyinthegroupof125rows, (RBMPKI),andthreevariationsofthedouble-sided(ds)[14,
depending on workload memory access patterns), and the 17,28,31,32,39,41]andmany-sided(ms)[15,54–56,58,62]
groupcounterisnolongerqueried. ona32-banksystemusingthesimulationmethodologythat
| 1582    33rd USENIX Security Symposium |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

weexplainin§8.Wecarefullycreatememorytraces(load whilesomeactivateupto25siblingrows(outof31),before
andstorerequeststhatarriveatmainmemory)ofdouble-and activatinganysiblingrowagain.Second,theaveragesibling
many-sidedattacks1)withoutprefetching,2)withasimple rowactivationcountdoesnotsignificantlycorrelatewiththe
prefetcherthatprefetchesthenextcacheline(p1)[174,175], memoryintensityoftheworkload.Third,theRowHammer
3)thenexteightcachelines(p8),4)andthenext32cache attackscanactivateupto31siblingrowsbeforeactivatingany
lines(p32)foreveryloadrequest.WenameaRowHammer siblingrowagainduetotheprefetchrequestsgeneratedbythe
attacktraceastheconcatenationofitstype(dsorms)andthe simplenextcachelineprefetcher.Fromtheseobservations,
prefetcherconfigurationusedincreatingthetrace(p1,p8,or weconcludethatafteraccessingarowwiththeaddressRina
p32).Forexample,ds-p32isthedouble-sidedRowHammer bank,therowsataddressRinotherbanks(i.e.,siblingrows)
attackwiththe next32 cache line prefetcher. Fig. 2 shows arealsolikelytobeactivated.Wehypothesizethatthisaccess
howmanysiblingrowsgetactivatedbeforeoneofthesibling patternoccursduetotwoproperties:i)thememoryaddress
rows is activated again,on average across all DRAM row mapping schemes that aim to increase memory bank-level
|             |          |         |           |          | (x-axis).3 | parallelismtoimprovesystemperformance(§2.1)5andii)the |     |     |     |     |     |     |     |     |
| ----------- | -------- | ------- | --------- | -------- | ---------- | ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
| activations | (y-axis) | foreach | simulated | workload |            |                                                       |     |     |     |     |     |     |     |     |
Benignworkloadsareorderedfromlefttorightinincreas- intrinsicspatiallocalityinworkloads’mainmemoryaccesses.
ingmemoryintensity(intermsofrowbuffermissesperkilo
|     |     |     |     |     |     | To  | strengthen |     | our motivation |     | for sharing | an  | activation |     |
| --- | --- | --- | --- | --- | --- | --- | ---------- | --- | -------------- | --- | ----------- | --- | ---------- | --- |
instructions)inthefigure.Wehighlightthehighestpossible counterbetweensiblingrows,weplotthedistributionofthe
y-axisvalue(31)witharedlineontheplot.Aworkloadwith activationcountofeachsiblingrowwhenatleastonesibling
abarclosertothislineindicatesthattheworkloadaccesses
|     |     |     |     |     |     | row | gets | activatedN | RH  | times. | In otherwords,one |     | pointin |     |
| --- | --- | --- | --- | --- | --- | --- | ---- | ---------- | --- | ------ | ----------------- | --- | ------- | --- |
allsiblingrowsataroundthesametime. thedistributionisanactivationcountofarow. Oneofthis
|     |     |     |     |     |     | row’ssiblingshasbeenactivatedN |     |     |     |     | RH times.Fig.3showshow |     |     |     |
| --- | --- | --- | --- | --- | --- | ------------------------------ | --- | --- | --- | --- | ---------------------- | --- | --- | --- |
manytimesasiblingrowgetsactivated(y-axis)beforeone
|     |     |     |     |     |     | ofthesiblingrowsgetsactivatedN |     |                |     |           |        | timesforN | =500,     |     |
| --- | --- | --- | --- | --- | --- | ------------------------------ | --- | -------------- | --- | --------- | ------ | --------- | --------- | --- |
|     |     |     |     |     |     |                                |     |                |     |           | RH     |           | RH        |     |
|     |     |     |     |     |     | 250,and                        |     | 125 (different |     | subplots) | across | benign    | workloads |     |
andRowHammerattacks(x-axis).Wehighlightthehighest
possibley-axisvaluesforeachN
RH value.
RowHammer Threshold (NRH) = 500
500
384
| Figure2:Numberofsiblingrowsactivatedbeforeonesibling |     |     |     |     |     |     | tnuoc noitavitca wor gnilbiS 256 |     |     |     |     |     |     |     |
| ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | -------------------------------- | --- | --- | --- | --- | --- | --- | --- |
128
rowisactivatedagain
|                                                  |     |     |     |     |     |     | 0   |     | RowHammer Threshold (NRH) = 250 |     |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------- | --- | --- | --- | --- | --- |
| Intheupperextremecase(aty=31)4,theworkloadalways |     |     |     |     |     |     | 250 |     |                                 |     |     |     |     |     |
192
| activatesallsiblingrowsoncebeforeactivatinganysibling |     |     |     |     |     |     | 128 |     |     |     |     |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
64
rowforthesecondtime.Thispropertyoftheworkloadmakes
0
RowHammer Threshold (NRH) = 125
itagoodfitforusingasingleactivationcountersharedbe-
125
| tweensiblingrows.Forustoknowwhenanaggressorsibling |     |     |     |     |     |     | 96  |     |     |     |     |     |     |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
64
| rowhasbeenactivatedN |     |     | RH times,thesharedcounterstores |     |     |     | 32  |     |     |     |     |     |     |     |
| -------------------- | --- | --- | ------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
0
thehighestactivationcountacrossallsiblingrows.Because
|     |     |     |     |     |     |     |     | revresc_bscy tserap.015 revresb_bscy revrese_bscy 46ccpt | revresa_bscy zx.755 3xnihps.284 edoced_2pj fcm.505 3448_cw 0pam_cw | MDAsutcac.634 pptenmo.174 ratsa.374 edocne_2pj 71hcpt kmbcnalax.384 | mutnauqbil.264 2hcpt clim.334 pptenmo.025 d3eilsel.734 xelpos.054 DTDFsmeG.954 d3kinotof.945 pmsuez.434 | mbl.915 mbl.074 fcm.924 spug edoced_462h yn_sfb 3002mc_sfb plbd_sfb | sd 1p-sd 8p-sd 23p-sd sm 1p-sm | 8p-sm 23p-sm |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ------------------------------ | ------------ |
theworkloadactivatesallsiblingrowsoncebeforeactivating
anyotherforthesecondtime,thesharedcounteraccurately RowHammer
Attacks
| represents | the activation |     | countofevery | sibling | row. In the |     |     |     |     |     |     |     |     |     |
| ---------- | -------------- | --- | ------------ | ------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Workload
lowerextremecase(aty=0),theworkloadneveractivates
|            |         |       |         |                       |     | Figure  | 3:  | The distribution |        | of  | the number | of activations |           | a   |
| ---------- | ------- | ----- | ------- | --------------------- | --- | ------- | --- | ---------------- | ------ | --- | ---------- | -------------- | --------- | --- |
| two ormore | sibling | rows. | Forthis | type ofa workload,the |     |         |     |                  |        |     |            |                |           |     |
|            |         |       |         |                       |     | sibling | row | receives         | before | any | sibling    | row gets       | activated |     |
singlesharedactivationcounter’svaluemisrepresentsalmost
N RH times
allofthesiblingrows’activationcounts(whichare0).
|         |       |     |              |           |              | We  | make | two | observations |     | from Fig. | 3. First, | a sibling |     |
| ------- | ----- | --- | ------------ | --------- | ------------ | --- | ---- | --- | ------------ | --- | --------- | --------- | --------- | --- |
| We make | three | key | observations | from Fig. | 2. First, on |     |      |     |              |     |           |           |           |     |
averageacrossallworkloads,12.8siblingrowsgetactivated row gets activated 99, 194, and 370 times for N RH values
before any sibling receives anotheractivation. We observe of125,250,and500,onaverageacrossallworkloads.This
|                                                       |     |     |     |     |     | indicatesthatwhenanysiblingrowgetsactivatedN |            |        |              |     |      |            |       | times, |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | -------------------------------------------- | ---------- | ------ | ------------ | --- | ---- | ---------- | ----- | ------ |
| thatsomeworkloadsactivateatleastthreesiblingrowsonce, |     |     |     |     |     |                                              |            |        |              |     |      |            | RH    |        |
|                                                       |     |     |     |     |     | the                                          | activation | counts | ofallsibling |     | rows | are (very) | close | to     |
3Theboxislower-boundedbythefirstquartile(i.e.,themedianofthe
|     |     |     |     |     |     | N   | .Second,asN |     | reduces,thegapbetweentheaverage |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ------------------------------- | --- | --- | --- | --- | --- |
firsthalfoftheorderedsetofdatapoints)andupper-boundedbythethird RH RH
quartile(i.e.,themedianofthesecondhalfoftheorderedsetofdatapoints). activation count of a sibling row and the sibling row with
Theinter-quartilerange(IQR)isthedistancebetweenthefirstandthird thehighestactivationcountbecomessmallerinproportion.
quartiles(i.e.,boxsize).Whiskersshowthecentral90thpercentileofthe Forexample,thebfs_cm203workload,onaverage,activates
distribution.
4ADRAMrowhas31siblingrowsinasystemwith32banks. 5Ourextendedversion[159]describestheseschemesinmoredetail.
| USENIX Association |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1583 |     |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- | --- |

siblingrows272(54.4%oftheN of500)and108(86.4% lingrowisalreadyset(i.e.,thesiblingrowwasactivatedonce
RH
oftheN of125)timesforN =500and125,respectively.6 sincethesharedcounterwaslastincremented).
|     | RH  |     |     | RH  |     |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Fromouranalysis,weconcludethatasinglesharedactiva-
|     |     |     |     |     |     |     |     |     | 4.1 ABACuS’sHardwareDesign |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------- | --- | --- | --- | --- | --- | --- |
tioncounter,whichstoresthehighestactivationcountamong
Fig.4presentsABACuS’skeycomponents.ABACuSis
theactivationcountsofallsiblingrows,canreasonablyaccu-
ratelyrepresenttheactivationcountofallsiblingrows.This placedinsidethememorycontroller.TheABACuScounter
propertyofthesharedactivationcounterbecomesstrongeras tablecontains(❶)multipleABACuScounters,eachmapped
N RH reducesfrom500to125. toarowID.ThereareexactlyN entries ABACuScountersin
theABACuScountertable.AnABACuScounter(❷)consists
4 Mechanism of a row activation counter (RAC) of size S bits and a
RAC
|                                                 |     |     |     |     |     |     |     |     | sibling activation |     | (bit) vector    | (SAV) of | size | S bits.    | The |
| ----------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --------------- | -------- | ---- | ---------- | --- |
| Overview.ABACuSisdesignedtopreventRowHammerbit- |     |     |     |     |     |     |     |     |                    |     |                 |          |      | SAV        |     |
|                                                 |     |     |     |     |     |     |     |     | ABACuS controller  |     | (❸) dynamically |          | maps | (not shown | in  |
flipsatlowperformance,energy,andareaoverhead.Achiev-
thefigure)arowIDtoacounterduringruntimeandusesa
| ing | low performance |     | and | energy | overheads | requires |     | accu- |     |     |     |     |     |     |     |
| --- | --------------- | --- | --- | ------ | --------- | -------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
spillovercounter(❹)totrackthemaximumactivationcount
| rately | identifying |     | aggressor | rowMsaaxnidmpurmev aecnttiivvaetliyonr ecforuesnht - |     |     |     |     |     |     |     |     |     |     |     |
| ------ | ----------- | --- | --------- | ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Row ID Activation Count of all DRAM rows that do not have an ABACuS counter
ingvictimrowsonlywhennecessoaf rRy.oTwo Xth aiscreonsds, aAllB bAaCnkusS
assigned(§4.2).Weexplainhowwedeterminethesizesof
adoptstheMisra-Griesalgorithm[172](§2)totrackaggressor
|     |     |     |       |     |      |     | X   | 42  | eachkeycomponent(S |     | ,S      | ,andN | )in§4.3. |     |     |
| --- | --- | --- | ----- | --- | ---- | --- | --- | --- | ------------------ | --- | ------- | ----- | -------- | --- | --- |
|     | X   | 72  | X 123 |     | X 31 |     |     |     |                    |     | RAC SAV |       | entries  |     |     |
rows,similartopriorwork[102,107,110,112,117].However,
|     | Bank 0 |     | Bank1 |     | Bank2 |     | Bank N |     |     |     | 1 ABACuS Counter Table |     |     |     |     |
| --- | ------ | --- | ----- | --- | ----- | --- | ------ | --- | --- | --- | ---------------------- | --- | --- | --- | --- |
Misra-GriesalonecannotpreventRowHammerbitflipsatlow
areacost(§7.1).Thus,ABACuSperformsMisra-Griestrack- rellortnoC SuCABA 2 ABACuSCounter
|     |     |     |     |     |     |     |     |     |     |     | Row Activation Counter |     | Sibling Activation Vector |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---------------------- | --- | ------------------------- | --- | --- |
ingusingsharedactivationcounterstosignificantlyreduce Row ID (RAC) (SAV) seirtne
theareaoverheadofimplementingmanycounters.
|     |     |     |     |     |     |     |     |     |     |     | S RAC  bits |     | S SAV  | bits | N   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ------ | ---- | --- |
ABACuS’skeyideaistosharearowactivationcounter
ABACuS
amongrowsthathavethesamerowIRDowac IDrossallbanks(which
Controller
wecallsiblingrows).Thesharedrowactivationcountertracks
4 Spillover Counter
| themaximumactivationcountamongthesiblingrows.ABA- |     |     |     |     |     |     |     |     | 3   |     |     |           |     |     |     |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | --- | --- |
|                                                   |     |     |     |     |     |     |     |     |     |     | S   | RAC  bits |     |     |     |
CuSpreventivelyrefreshesalltheneighboringrowsofallthe
Figure4:KeycomponentsofABACuS
siblingrowstrackedbythesamerowactivationcounterjust
Therowactivationcounter(RAC)inanABACuScounter
enoughbeforetherowactivationcounter’svaluereachesN
|          |     |                                        |     |     |     |     |     | RH  | (❷)storesthemaximumactivationcountacrossallsibling |     |     |     |     |     |     |
| -------- | --- | -------------------------------------- | --- | --- | --- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- | --- | --- | --- |
| withinat |     | topreventRowHammerbitflips(i.e.,noneof |     |     |     |     |     |     |                                                    |     |     |     |     |     |     |
REFW
rows’activationcounts.Thesiblingactivationvector(SAV)
| thesiblingrows’activationcountreachesN |              |     |        |     |         | withint    |      | ).    |                                                    |     |     |     |     |     |     |
| -------------------------------------- | ------------ | --- | ------ | --- | ------- | ---------- | ---- | ----- | -------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|                                        |              |     |        |     |         | RH         | REFW |       | storesthesiblingactivationbitsusedbyABACuStoincre- |     |     |     |     |     |     |
|                                        | While ABACuS |     | tracks | the | maximum | activation |      | count |                                                    |     |     |     |     |     |     |
menttheRAConlywhennecessary(see§4).TheABACuS
amongsiblingrows,itdoesnotunnecessarilyincrementthe
|     |     |     |     |     |     |     |     |     | controller updates |     | RAC and SAV | to  | ensure | the maximum |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | ----------- | --- | ------ | ----------- | --- |
sharedrowactivationcounterwitheachsiblingrowactivation.
activationcountamongallsiblingrowsistrackedintheRAC.
Forexample,whenaworkloadactivatesmultiplesiblingrows
onlyonce(whichisacommonbehaviorweobservein§3), 4.2 OperationofABACuS
| itis | sufficientforABACuS | Row Activation Counter |       | to  | incrementthe Bank Activation Vector | sharedactiva- |     |     |     |     |     |     |     |     |     |
| ---- | ------------------- | ---------------------- | ----- | --- | ----------------------------------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|      | Row ID              |                        | (RAC) |     |                                     | (BAV)         |     |     |     |     |     |     |     |     |     |
WedescribeABACuS’soperationinfivekeysteps.
tioncounterbyonlyone.Afterasiblingrowisactivatedand
(1)InitializationandReset.Initially(atsystempoweron)
ABACuSincrementsthesharedrowactivationcounter,other
andafterperiodicABACuScountertablereset(Step5),no
siblingrowscanbeactivatedatmostoncebeforetheshared
|                                                     |     |     |     |     |     |     |     |     | DRAMrowisactivatedforthelastt |     |     |      | .Thus,rowactiva- |     |     |
| --------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------- | --- | --- | ---- | ---------------- | --- | --- |
| counterisincrementedagain.Toallowformultiplesibling |     |     |     |     |     |     |     |     |                               |     |     | REFW |                  |     |     |
tioncounters(RACs)andsiblingactivationvectors(SAVs)
rowstobeactivatedwithoutunnecessarilyincrementingthe
Bank ID 3 Bank ID 0 inallABACuScountersandthespillovercounterallstore0.
sharedrowactivationcounter,ABACuSmaintainsabitvector
(2)ABACuSCounterTableSearch.Thememorycontroller
forthecounter,siblingactivation Row ID *RAC *SAV vector,thatstoreswhich Row ID RAC SAV Row ID RAC SAV Row ID RAC SAV
|     |     |     |     |     | ACT |     |     |     | issuesanACT ACT |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- |
siblingrowswereactivatedsincethesharedcounterwaslast etatS laitinI commandtoarowIDinabank.Consequently, ACT
13 27 0001 Row ID: 13 13 27 0 0 11 Row ID: 13 13 28 0 0 1 0 Row ID: 20 13 28 0 0 1 0
|     |     |     |     |     | Bank ID: 1 |     |     |     | theABACuScontrollersearchesallrowIDmappingstofind Bank ID: 1 |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | ---------- | --- | --- | --- | ------------------------------------------------------------ | --- | --- | --- | --- | --- | --- |
incremented.ABACuSincrementsthesharedrowactivation 9 12 0101 9 12 0 1 0 1 9 12 0 1 0 1 Bank ID: 2 20 13 0 1 0 0
iftheactivatedrowisalreadytrackedbyanABACuScounter.
counteronlywhenthebitcorrespondingtotheactivatedsib-
1 14 1000 1 Update 1 14 1 0 0 0 2 Update 1 14 1 0 0 0 3 Replace 1 14 1 0 0 0
Norowistrackedimmediatelyafterinitializationandreset.
6Thegupsworkloadcannotactivateanysamerow125timesbecausea
12 (Spillover Counter) 12 (Spillover Counter) Thus,theABACuScontrollerneedstomaparowIDtoan 12 (Spillover Counter) 12 (Spillover Counter)
workloadislimitedinthenumberofDRAMrowactivationsitcanissuetoa
*RAC: Row Activation Counter, SAV: Sibling Activation Vector ABACuScounter.Tofindthecounterthatwillbemappedto
DRAMchipbyDRAMtimingparameters(e.g.,fourrowactivationwindow,
tFAW [158])andgups’rowactivationsarerandomlyandevenlydistributedto theactivatedrow’sID,theABACuScontrollerlooksforan
all128KDRAMrows.AworkloadthatfullyexercisestheavailableDRAM ABACuScounterwhoseRACstoresthesamevalueasthe
rowactivationbandwidthcanonlyissue12’190’476activatecommands
spillovercounter’svalue.IfaRACandthespillovercounter
| inarefreshwindow(64ms)duetotFAW |     |     |     |     | (21ns)timingconstraint[158]. |     |     |     |               |        |           |     |       |          |      |
| ------------------------------- | --- | --- | --- | --- | ---------------------------- | --- | --- | --- | ------------- | ------ | --------- | --- | ----- | -------- | ---- |
|                                 |     |     |     |     |                              |     |     |     | have the same | value, | the RAC’s | row | ID is | replaced | with |
Therefore,randomlyandevenlydistributing12’190’476activatecommands
|     |     |     |     |     |     |     |     |     | theactivatedrow’sID(Step3). |     |     | Incontrast,ifanABACuS |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------------------- | --- | --- | --------------------- | --- | --- | --- |
to128Krowswouldactivatearowatmost94times.
|     |     |     |     |     |     |     |     | Row ID | RAC SAV       |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------------- | --- | --- | --- | --- | --- | --- |
|     |     |     |     |     |     |     |     |        | 13 27 0 0 0 1 |     |     |     |     |     |     |
1584    33rd USENIX Security Symposium 9 12 0 1 0 1 USENIX Association
|     |     |     |     |     |     |     |     |     | 1 14 1 0 0 0 |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- |
12 (Spillover Counter)

| Bank ID 3   | Bank ID 0  |       |                |       |       |          |       |     |            |           |     |              |         |
| ----------- | ---------- | ----- | -------------- | ----- | ----- | -------- | ----- | --- | ---------- | --------- | --- | ------------ | ------- |
| Ro w  ID *R | A C *S A V |       | Ro w  ID R A C | S A V |       | Ro w  ID | R A C |     |            |           |     |              |         |
|             |            | A C T |                |       | A C T |          | S A V |     | Ro w  ID R | A C S A V |     | Ro w  ID R A | C S A V |
etatS laitinI 1 3 2 7 0 0 0 1 Row   ID : 13 1 3 2 7 0  0  1 1 Row   ID : 13 1 3 2 8 0  0  1  0 A C T 1 3 2 8 0  0  1  0 A C T 1 3 2 8 0  0  1  0
|     |     |     |     |     |     |     |     | Row   ID : 20 |     |     | Ro w  ID : 7 |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --- | ------------ | --- | --- |
9 12 0101 Bank ID: 1 9 12 0 1 0 1 Bank ID: 1 9 12 0 1 0 1 Bank ID: 2 20 13 0 1 0 0 Bank ID: 1 20 13 0 1 0 0
1 14 1000 1 Update 1 14 1 0 0 0 2 Update 1 14 1 0 0 0 3 Replace 1 14 1 0 0 0 Spillover 1 14 1 0 0 0
4
12 (Spillover Counter) 12 (Spillover Counter) 12 (Spillover Counter) 12 (Spillover Counter) Update 13(Spillover Counter)
*RAC: Row Activation Counter, SAV: Sibling Activation Vector
Figure5:ABACuSworkflowusingfouractivate(ACT)commands.Wehighlightstatechangesusingblackboxesandredtext.
counteralready tracks the row ID,the ABACuS controller preventaggressorrowsfrombeingactivatedN /2timesina
RH
updatesthematchingcounter(Step4).IncasenoRACvalue resetperiod.ABACuSisconfiguredinthiswaybecauseABA-
equalsthespillovercounter’svalue(i.e.,theactivatedrow’s CuSdoesnotpreciselyknowwheneachrowisperiodically
IDcannotbemappedtoanABACuScounter)andthereisno refreshed:Apotentialaggressorrowmightbehammeredfor
ABACuScounterthatalreadytrackstheactivatedrow’sID, PRT−1timesbeforeitsneighborsarerefreshedandABA-
theABACuScontrollerincrementsthespillovercountervalue. CuSisreset.AfterABACuSisreset,anattackercanhammer
Whenthespillovercounterreachesapredefinedvalueofthe the same aggressorrow for2∗PRT times,accumulating a
refreshcyclethreshold(RCT),ABACuSissuest REFW /t REFI total activation count of 2∗PRT−1 on the aggressor row.
refresh(REF)commandstorefreshallDRAMrowsinthe Thus,wesetPRTtoN /2.
RH
DRAMrankandresetsallcounters.Wecallthetimewhen
|     |     |     |     |     |     |     | 4.3 ConfiguringABACuS |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------------- | --- | --- | --- | --- | --- | --- |
thememorycontrollerrefreshesallDRAMrowsduetothe
spillovercounterreachingtheRCTarefreshcycle.
Dependingonthesystem’sRowHammervulnerability(typ-
| (3) ABACuS | Counter | Mapping | and | Replacement. |     | The |                      |     |     |                               |     |     |     |
| ---------- | ------- | ------- | --- | ------------ | --- | --- | -------------------- | --- | --- | ----------------------------- | --- | --- | --- |
|            |         |         |     |              |     |     | icallymeasuredusingN |     |     | RH ),ABACuShasthreekeyparame- |     |     |     |
ABACuScontrollermapsthenewly-activatedrowIDtothe
tersthatareconfiguredatdesigntime.First,thenumberof
matchingABACuScounter.Tocorrectlytrackthemaximum entries(N )intheABACuScountertable.Second,the
entries
activationcount,theABACuScontrolleri)initializesRAC
|     |     |     |     |     |     |     | sizeofeachrowactivationcounter(S |     |     |     | RAC | ).Third,thesizeof |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------------------- | --- | --- | --- | --- | ----------------- | --- |
withspillovercountervalue+1,andii)setsthebitintheSAV eachsiblingactivationvector(S ).
SAV
thatcorrespondstothebankIDoftheactivatedrow.
|     |     |     |     |     |     |     | Configuring | the | Number | of Entries |     | (N entries ). | We deter- |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ------ | ---------- | --- | ------------- | --------- |
(4) ABACuS Counter Update. The ABACuS controller mine the number of entries based on how many rows can
checkstheSAVbitvaluecorrespondingtotheactivatedrow’s
behammeredinoneDRAMbankduringoneABACuSre-
| bankID. | Ifthe SAV | bitis | not set(i.e.,stores |     | logic-0),i.e., |     |                 |     |       |                   |     |                  |     |
| ------- | --------- | ----- | ------------------- | --- | -------------- | --- | --------------- | --- | ----- | ----------------- | --- | ---------------- | --- |
|         |           |       |                     |     |                |     | setperiod(64ms) |     | given | i) the preventive |     | refreshthreshold |     |
theactivatedrowisactivatedforthefirsttimesincetheRAC (PRT),ii)t ,andiii)t asdescribedinapriorworkthat
|     |     |     |     |     |     |     |     | RC  |     | RFC |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
waslastincremented,theABACuScontrollersetstheSAV
|     |     |     |     |     |     |     | adopts Misra-Gries |     | tracking | [102]. | We  | first calculate | how |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | -------- | ------ | --- | --------------- | --- |
bit.IftheSAVbitisset(i.e.,storeslogic-1),i.e.,theactivated many ACT commands can be issued by the memory con-
rowisactivatedagainsincetheRACwaslastincremented, trollerinaresetperiodwhenthebankisnotunavailabledue
theABACuScontrollerincrementstheRACby1.Afterin-
|     |     |     |     |     |     |     | to periodic | refresh | (N  | ) as t | ∗(1−t | /t  | )/t .   |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ------- | --- | ------ | ----- | --- | ------- |
|     |     |     |     |     |     |     |             |         | ACT | REFW   |       | RFC | REFI RC |
crementingtheRAC,theABACuScontrollersetstheSAV Thus,atmostN /PRT rowscanbeactivatedPRTtimes
ACT
| bit corresponding |     | to the bank | ID of the | activated | row | and |     |     |     |     |     |     |     |
| ----------------- | --- | ----------- | --------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
inanABACuSresetperiodinabank.Settingthenumberof
resetsallotherbits.Thisway,asetbitcorrespondingtothe entriestoN /PRT issufficienttoensurethatallhammered
ACT
bankIDoftheactivatedrowintheSAVstillindicatesthat rowsinanABACuSresetperiodinonebankaretrackedin
thecorrespondingrowwasactivatedoncesincetheRACwas
oneofthecounters.
lastincremented.IftheRAC’svalueisamultipleofthepre- ConfiguringtheSizeofRowActivationCounters(S ).
RAC
ventiverefreshthreshold(PRT),i.e.,oneofthesiblingrows
|     |     |     |     |     |     |     | The number | of  | activations | that | the memory | controller | can |
| --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ----------- | ---- | ---------- | ---------- | --- |
tracked by the RAC was activated PRT times since all the issue in a t determines the row activation counter’s
REFW
| siblingrows’victimswerepreventivelyrefreshed,ABACuS |     |     |     |     |     |     |             |     |                |         |     |           | S = |
| --------------------------------------------------- | --- | --- | --- | --- | --- | --- | ----------- | --- | -------------- | ------- | --- | --------- | --- |
|                                                     |     |     |     |     |     |     | size. Thus, | a   | row activation | counter |     | should be | RAC |
performspreventiverefreshoperationstoallthevictimrows
|     |     |     |     |     |     |     | ⌈log ((N | ∗t  | )/t    | )⌉ bits | large. However, | we  | can re- |
| --- | --- | --- | --- | --- | --- | --- | -------- | --- | ------ | ------- | --------------- | --- | ------- |
|     |     |     |     |     |     |     | 2        | ACT | RC RRD |         |                 |     |         |
(theneighborsoftheactivatedrow)inallbanks. duceS to⌈log (PRT)⌉byaddinganoverflowbit[102]to
|           |         |                  |            |     |          |     | RAC |     | 2   |     |     |     |     |
| --------- | ------- | ---------------- | ---------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
| (5) Reset | Period. | It is sufficient | for ABACuS |     | to track | the |     |     |     |     |     |     |     |
eachABACuScounter.Usingtheoverflowbit,wemakesure
activationcountswithinat REFW ,afterwhichallDRAMrows thatABACuSpreventivelyrefreshesrowsaccordingtothe
arerefreshed.Therefore,weresettheABACuScountersand Misra-Griestrackingalgorithm[102]:WhenaRACreaches
the spillovercounteraftereveryt REFW (i.e.,ABACuS’s re- PRT,ABACuSsetstheoverflowbitoftheRACandresets
set period is t ). After periodic reset,ABACuS’s state the RAC’s value. The set overflow bit in a RAC indicates
REFW
becomesasdescribedinStep1.
thatABACuSshouldnotreplacetherowIDtrackedbythis
Determining the Preventive Refresh Threshold (PRT). RAC,eveniftheRAC’svalueequalsthevalueofthespillover
ABACuS’spreventiverefreshthresholdissetaccordinglyto counter.Theoverflowbitisresetaftereachresetperiod.
| USENIX Association |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1585 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

Configuringthesizeofsiblingactivationvectors(S ).A andotherstate-of-the-artmechanismswithablastradiusof
SAV
SAVcontainsasmanybitsastherearebanks.ThusS = oneinourperformanceandenergyevaluation(§8).
SAV
| N banks bits. | Forexample,there | are 32 | banks in a | dual-rank |     |     |     |     |     |
| ------------- | ---------------- | ------ | ---------- | --------- | --- | --- | --- | --- | --- |
7 HardwareImplementation
| DDR4-basedsystem[158],makingS |     |     | 32-bitlarge. |     |     |     |     |     |     |
| ----------------------------- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- |
SAV
4.4 ExampleABACuSWorkflow ABACuSisimplementedinthememorycontroller.Itdoes
notrequireanymodificationstoexistingDRAMchips.
| Fig. 5 shows | an example | ABACuS | workflow withthree |     |     |     |     |     |     |
| ------------ | ---------- | ------ | ------------------ | --- | --- | --- | --- | --- | --- |
KeyComponents.ABACuS’shardwareimplementationcon-
ABACuScountersandfoursiblingrows.Weshowhowfour sistsoftwocomponents:i)theABACuScountertable,and
ACT commands cause state changes in the three ABACuS ii)thespillovercounter.TheABACuScountertablecontains:
| counters. | The first ACT | command (❶) | updates the | sibling |     |     |     |     |     |
| --------- | ------------- | ----------- | ----------- | ------- | --- | --- | --- | --- | --- |
i)theRowIDTable(RIT),ii)theRACTable(RACT),and
activationvector(SAV)ofitsABACuScounterasthebank iii)theSAVTable(SAVT).Toefficientlytrackthenumberof
| ID corresponds | to a zero | bit in the SAV. | The second | ACT |     |     |     |     |     |
| -------------- | --------- | --------------- | ---------- | --- | --- | --- | --- | --- | --- |
activations,ABACuSsearchesandupdatestheRITandthe
(❷)
command increments the row activation counterof its RACT.Thus,weimplementRITandRACTusingcontent-
ABACuScounterasthebankIDcorrespondstoanon-zerobit addressablememory(CAM)arrays.WeimplementSAVTas
| intheSAV.ThethirdACT |     | command(❸)replacesthesecond |     |     |     |     |     |     |     |
| -------------------- | --- | --------------------------- | --- | --- | --- | --- | --- | --- | --- |
anSRAMarraysinceABACuSdoesnotsearchSAVTentries.
ABACuS counter with row ID 20 because i) row ID 20 is Aregisterstoresthespillovercounter’svalue.
nottrackedbyanycounter,andii)thespillovercountervalue
|     |     |     |     |     | Performing | Preventive | Refresh. | Since the standard | REF |
| --- | --- | --- | --- | --- | ---------- | ---------- | -------- | ------------------ | --- |
isequaltothesecondABACuScounter’sRACvalue. The commandisrow-address-agnosticinDRAMstandards[147,
| fourthACT | command(❹)incrementsthespillovercounter |     |     |     |     |     |     |     |     |
| --------- | --------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
158,163],ABACuScannotusestandardrefreshcommands
asi)rowID7isnottrackedbyanyABACuScounter,andii)
torefreshdetectedvictimrows.Toremaincompatiblewith
thespillovercountervalueissmallerthanallRACvalues. existingDRAMchipsandinterfacestandards,ABACuSper-
formsapreventiverefreshoperationbyaccessingavictim
5 SecurityAnalysis
|     |     |     |     |     | rowonceusingACT |     | andPRE | commands.Whenatracked |     |
| --- | --- | --- | --- | --- | --------------- | --- | ------ | --------------------- | --- |
row’sRACvaluereachesPRT,ABACuSperformspreventive
ABACuSpreventivelyrefreshesthevictimrowsofapoten-
tialaggressorrowbeforetheaggressorrowisactivatedN refreshoperationstovictimrowsinallbanks.ABACuSpri-
RH
oritizespreventiverefreshesoverothermemoryrequests:the
| timesinat | REFW .AssumingABACuSaccuratelystoresthe |     |     |     |     |     |     |     |     |
| --------- | --------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
maximumactivationcountacrossallsiblingrows,theMisra- memorycontrollerdoesnotserveanymemoryrequesttothe
samebankuntilthevictimrowsarepreventivelyrefreshed.
Gries-basedtrackingtechniqueguaranteesthatnoaggressor
rowisactivatedmorethanthepreventiverefreshthresholdin
7.1 AreaOverhead
at [102].ABACuSaccuratelymaintainsthemaximum
REFW
activationcountintherowactivationcountersbecausethe WeevaluateABACuS’andfourotherstate-of-the-artmiti-
| rowactivation | counter’svalueisincremented1)when |     |     | any |     |     |     |     |     |
| ------------- | --------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
gationmechanisms’[1,102,106,177](theirconfigurationde-
siblingrowisactivatedforthefirsttime,and2)whenasibling tailsareexplainedin§8)chiparea,staticpower,andmemory
rowwhosesiblingactivationvectorbitisset(i.e.,thesibling
arrayaccessenergyusingCACTI[157].Table1summarizes
rowwasactivatedaftertherowactivationcounter’svaluewas
|     |     |     |     |     | ourresults.WeperformthisanalysisatN |     |     | valuesof1000 |     |
| --- | --- | --- | --- | --- | ----------------------------------- | --- | --- | ------------ | --- |
RH
lastincremented)isactivated.AppendixAformallyanalyzes and125.Table2showsABACuS’keyparameters.
andshowsthattherowactivationcounteralwaysstoresthe
|     |     |     |     |     | All three | ABACuS | hardware | structures (Row | ID Table, |
| --- | --- | --- | --- | --- | --------- | ------ | -------- | --------------- | --------- |
maximumactivationcount.
|     |     |     |     |     | RACT,andSAVT)containN |     |     | entries.Atanearfuture |     |
| --- | --- | --- | --- | --- | --------------------- | --- | --- | --------------------- | --- |
entries
6 AccountingforRowHammerBlastRadius N RH of1000,weestimateABACuS’soverallareaoverhead
|     |     |     |     |     | to be 0.04mm2 | perDRAM | channelfora | dual-ranksystem. |     |
| --- | --- | --- | --- | --- | ------------- | ------- | ----------- | ---------------- | --- |
Anaggressorrowcancausebitflipsinvictimrowsthatare ABACuS consumes approximately 0.02% of the chip area
notphysicallyadjacent[1,14].TheimpactofRowHammer ofahigh-endIntelXeonprocessorwithfourmemorychan-
|                                                   |     |     |     |     | nels[178].AtalowN |     | of125,ABACuS’sestimatedchip |     |     |
| ------------------------------------------------- | --- | --- | --- | --- | ----------------- | --- | --------------------------- | --- | --- |
| onavictimrowdecreasesandeventuallydisappearsasthe |     |     |     |     |                   |     | RH                          |     |     |
areacostincreasesto0.25mm2,takinguponlyapproximately
physicaldistancebetweenthevictimandtheaggressorrows
increases.Toaccountforthischaracteristic,priorworksdefine 0.11%ofthesameprocessor’sarea.
blastradiusasthedistancebetweenanaggressorrowandits AreaComparison.AtanN RH of1000,ABACuStakesup
furthestvictimrow[1,14,15,17,50,54–56,58,102–104,123, 20.25×and2.50×smallerchipareathanGraphene[102]and
126,176].ArecentRowHammerattack,HalfDouble[58],ex- Hydra[106],respectively.Graphene’sareaoverheadislarger
ploitsblastradiustoinducebitflipswithasignificantlylower thanothermechanismsbecauseitimplementsalargenumber
activationcount.ToaccountforblastradiusandaddressHalf ofcounters(e.g.,2720perbank,87040intotalforadual-rank
Double,ABACuS1)preventivelyrefreshesallpotentialvic- DDR4system).REGA[177]takes2.06%DRAMchiparea
timrowswithintheblastradiusand2)countseachpreventive toimplement. ComparedtoABACuS’smemorycontroller
refreshas an additionalactivation. We configure ABACuS chiparearequirement,REGArequiresalargerDRAMchip
| 1586    33rd USENIX Security Symposium |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

Table1:Area,energy,powerofABACuSvs.state-of-the-artRowHammermitigationmechanismsfora2-rankmemorysystem
|     |     |      |     |     | NRH=1000 |     |     |          |     | NRH=125 |     |     |     |
| --- | --- | ---- | --- | --- | -------- | --- | --- | -------- | --- | ------- | --- | --- | --- |
|     |     | SRAM | CAM |     | Area     |     |     | SRAM CAM |     | Area    |     |     |     |
MitigationMechanism AccessEnergy StaticPower AccessEnergy StaticPower
KB KB mm2 %CPU %DRAM (pJ) (mW) KB KB mm2 %CPU %DRAM (pJ) (mW)
ABACuS 10.63 8.30 0.04 0.02 - 24.36 12.19 85.00 66.41 0.25 0.11 - 36.87 50.39
RowIDTable - 5.64 0.01 <0.01 - 11.23 6.59 - 45.16 0.12 0.05 - 20.64 27.42
RowActivationCounterTable - 2.66 0.02 <0.01 - 11.13 4.66 - 21.25 0.06 0.03 - 11.66 15.53
<0.01
SiblingActivationVector 10.63 - 0.01 - 1.99 0.95 85.00 - 0.07 0.03 - 4.57 7.44
| PARA[1] |     | -   | -   | -   | <0.01 - | -   | -   | - - | -   | <0.01 | -   | -   | -   |
| ------- | --- | --- | --- | --- | ------- | --- | --- | --- | --- | ----- | --- | --- | --- |
Graphene[102] - 286.51 0.81 0.35 - 876.85 188.64 - 2037.09 5.68 2.43 - 1042.49 1385.52
Hydra[106] 61.56 - 0.10 0.04 - 43.07 24.23 56.5 - 0.07 0.03 - 40.25 23.14
| REGA[177] |                         | -   | -   | -   | - 2.06 | -   | -   | - -                   | -   | -   | 2.06 | -   | -   |
| --------- | ----------------------- | --- | --- | --- | ------ | --- | --- | --------------------- | --- | --- | ---- | --- | --- |
|           | Table2:ABACuSParameters |     |     |     |        |     | 8   | EvaluationMethodology |     |     |      |     |     |
| Term      | Definition              |     |     |     | Value  |     |     |                       |     |     |      |     |     |
WeevaluateABACuS’sperformanceandenergyconsump-
NRH RowHammerthreshold 1000 500 250 125 tionwithRamulator[153,154],acycle-accurateDRAMsim-
| PRT | Preventiverefreshthreshold |     |     | 500 | 250 125 | 62  |     |     |     |     |     |     |     |
| --- | -------------------------- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
RCT Refreshcyclethreshold 498 248 123 60 ulator,and DRAMPower [180]. We specify our simulated
Nentries Numberoftableentries 2720 5440 10880 21760 system’sconfigurationinTable3.
| SSAV | Bit-lengthofaSAVentry |     |     | 32  | 32 32 | 32  |     |     |     |     |     |     |     |
| ---- | --------------------- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
Table3:SimulatedSystemConfiguration
| SRID | Bit-lengthofaRowIDentry |     |     | 17  | 17 17 | 17  |     |     |     |     |     |     |     |
| ---- | ----------------------- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
SRAC Bit-lengthofaRACentry 10 9 8 7 1or8cores,3.6GHzclockfrequency,
Processor
4-wideissue,128-entryinstructionwindow
area. PARA [1] does notmaintain any state,thus ithas no DDR4,1channel,2rank/channel,4bankgroups,
DRAM
significantareaoverhead. 4banks/bankgroup,128Krows/bank,3200MT/s
64-entryreadandwriterequestsqueues,
| We repeat | our | area overhead |     | analysis | for future | DRAM |     |     |     |     |     |     |     |
| --------- | --- | ------------- | --- | -------- | ---------- | ---- | --- | --- | --- | --- | --- | --- | --- |
Schedulingpolicy:FR-FCFS[181,182]
chipsbyscalingtheRowHammerthresholddownto125.Al- withacolumncapof16[183],
MemoryCtrl.
thoughABACuS’sareaoverheadincreasesasitimplementsa Addressmapping:MOP[167,169]
45nstRC,7.9µstREFI,64mstREFW
| largernumberofABACuScountersatthelowerN |     |     |     |     |     | ,ABA- |     |     |                       |     |     |     |     |
| --------------------------------------- | --- | --- | --- | --- | --- | ----- | --- | --- | --------------------- | --- | --- | --- | --- |
|                                         |     |     |     |     |     | RH    |     |     | 64msABACuSresetperiod |     |     |     |     |
CuSstillconsumesarelativelysmall0.11%processorchip Last-LevelCache 2MiBpercore
| areaatanN | of125. | ABACuSrequires22.72×lesschip |     |     |     |     |         |          |      |           |     |         |         |
| --------- | ------ | ---------------------------- | --- | --- | --- | --- | ------- | -------- | ---- | --------- | --- | ------- | ------- |
|           | RH     |                              |     |     |     |     | Address | Mapping. | Fig. | 6 depicts | our | address | mapping |
areatoimplementthanGraphene.ABACuS’sareaoverhead
scheme.Weuseanaddressmappingschemethatinterleaves
| atthisverylowN |     | is3.57×thatofHydra’s,however,Hydra |     |     |     |     |             |       |        |        |                 |     |           |
| -------------- | --- | ---------------------------------- | --- | --- | --- | --- | ----------- | ----- | ------ | ------ | --------------- | --- | --------- |
|                | RH  |                                    |     |     |     |     | consecutive | cache | blocks | in the | physicaladdress |     | space be- |
incursuptoaverylarge85.42%performanceoverheadfor8-
tweendifferentDRAMbanks.
| corememory-intensiveworkloadsatthesameN       |     |     |     |     | RH  | (see§9.1). |     |     |     |     |     |     |     |
| --------------------------------------------- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
| Hydra’schipareaoverheadreduceswithdecreasingN |     |     |     |     |     | as         |     |     |     |     |     |     |     |
RH
itrequirescounterswithfewerbitsofstorageeach.Wecon-
cludethatABACuS’schiparearequirementscalesbetterthan
Graphene’sandthatABACuS’sarearequirementatlowN
RH
isclosertothemostarea-efficientstate-of-the-artmitigation
mechanism,Hydra.
Figure6:Simulatedaddressmapping
| Energy | and Static | Power | Comparison. |     | For N | = 1000, |     |     |     |     |     |     |     |
| ------ | ---------- | ----- | ----------- | --- | ----- | ------- | --- | --- | --- | --- | --- | --- | --- |
RH
ComparisonPoints.WecompareABACuStoabaselinesys-
| ABACuS | has 36.00× |     | and 1.77× | smaller | access | energy |     |     |     |     |     |     |     |
| ------ | ---------- | --- | --------- | ------- | ------ | ------ | --- | --- | --- | --- | --- | --- | --- |
temwithnoRowHammermitigationandtofourstate-of-the-
thanGrapheneandHydra,respectively.ABACuSconsumes
12.19mW ofstaticpower,whichis15.47×and1.99×smaller artRowHammermitigationmechanisms:(1)Graphene[102]
implementsperbankcounterstotrackthepossibleaggressor
thanGrapheneandHydra’sstaticpowerconsumptions.As
|     |     |     |     |     |     |     | rows | using Misra-Gries |     | algorithm | [172]. | When | a counter |
| --- | --- | --- | --- | --- | --- | --- | ---- | ----------------- | --- | --------- | ------ | ---- | --------- |
N reducesto125,ABACuS’sstaticpowerandaccessen-
RH
valueexceedsathresholdvalue,Grapheneissuespreventive
ergyscalemoreefficiently(similarlytoHydra)comparedto
|     |     |     |     |     |     |     | refreshes | to the | victim | rows. (2) | Hydra | [106] | implements |
| --- | --- | --- | --- | --- | --- | --- | --------- | ------ | ------ | --------- | ----- | ----- | ---------- |
Graphene,whereGraphenehas28.27×and27.50×theaccess
energyandstaticpowerofABACuS,respectively. agroupcounttabletotrackactivationsforagroupofrows
andarowcounttabletotrackperrowactivations.Weconfig-
ureHydrasuchthatallrowsinarowgrouphavetheirrow
7.2 LatencyAnalysis
counttableentriesresideintwoconsecutivecacheblocks(64
byteseach).TherowcounttableisstoredintheDRAMand
We implement ABACuS in Verilog and use Synopsys cachedinthememorycontroller.Hydraperformspreventive
DC [179] to evaluate ABACuS’s latency impact on mem- refreshoperationswhenacounterexceedsathresholdvalue.
oryaccesses.ABACuSneeds1.22nstoupdatetheABACuS (3)REGA[177]augmentsDRAMdesignsuchthatoneor
counter of an activated DRAM row. This latency overlaps more victim rows can be refreshed when a DRAM row is
withthelatencyofregularmemorycontrolleroperationsasit activated.REGAtunesitsprotectionguaranteesbychanging
issmallerthant (e.g.,2.5nsinDDR4[158,161]). thedefaultt value.Asmallert allowsREGAtorefresh
|                    | RRD |     |     |     |     |     |     | RC  |                                        | RC  |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |
| USENIX Association |     |     |     |     |     |     |     |     | 33rd USENIX Security Symposium    1587 |     |     |     |     |

| morerowsconcurrentlywithaDRAMrowactivation,atthe |     |     |     |     | CPI dezilamroN | 1.1 |     |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | -------------- | --- | --- | --- | --- | --- | --- |
Baseline IPC
| costofincreasedaccesslatency.TosimulateREGA,wemod- |     |     |     |     |     | noitubirtsid 1.0 |     |     |     |     |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- |
ifyt RC asdescribedin[177].(4)PARA[1]protectsagainst 0.9 NRH=1000 (leftmost box)
|                                                      |     |     |     |     |     |     | NRH=500 |     |     | gups |     |
| ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | ------- | --- | --- | ---- | --- |
| RowHammerbyperformingprobabilisticadjacentrowactiva- |     |     |     |     |     | 0.8 | NRH=250 |     |     |      |     |
tion.Whenarowisclosed(i.e.,whenthememorycontroller 0.7 NRH=125 (rightmost box)
0.6
issuesaprecharge(PRE)),PARAissuespreventiverefreshes
|     |     |     |     |     |     |     | low (<2) |     | medium (<10) | high (>10) |     |
| --- | --- | --- | --- | --- | --- | --- | -------- | --- | ------------ | ---------- | --- |
to the adjacent rows based on a probability threshold. We Row buffer misses per kilo instructions (RBMPKI)
tune the probability thresholdofPARA fora targetfailure Figure 7: Performance of single-core applications for four
probabilityof10−15withina64msasinpriorwork[104].
differentRowHammerthresholds(higherisbetter)
| Workloads. | We evaluate | 62 single-core | and 62 | homoge- |     |     |     |     |     |     |     |
| ---------- | ----------- | -------------- | ------ | ------- | --- | --- | --- | --- | --- | --- | --- |
WemaketwomajorobservationsfromFig.7.First,ABA-
neousmulti-programmed8-coreworkloadsfromfivebench-
marksuites:SPECCPU2006[184],SPECCPU2017[185], CuSinducesminorsystemperformanceoverheadforalleval-
|                                               |     |     |     |     | uatedsingle-coreworkloadsatanear-futureN |     |     |     |     |     | of1000.At |
| --------------------------------------------- | --- | --- | --- | --- | ---------------------------------------- | --- | --- | --- | --- | --- | --------- |
| TPC[186],MediaBench[187],andYCSB[188].Basedon |     |     |     |     |                                          |     |     |     |     | RH  |           |
suchN ,ABACuSincursonly0.58%(32.00%)slowdown
| the row | buffer misses-per-kilo-instruction |     | (RBMPKI), | we  |     | RH  |     |     |     |     |     |
| ------- | ---------------------------------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
grouptheapplicationsintothreecategories,whichTable4 on average (at maximum) across all workloads. ABACuS
|     |     |     |     |     | increases |     | the average | memory | latency experienced |     | by ap- |
| --- | --- | --- | --- | --- | --------- | --- | ----------- | ------ | ------------------- | --- | ------ |
describes:(1)L(lowmemory-intensity,RBMPKI∈[0,2)),
(2) M (medium memory-intensity,RBMPKI ∈[2,10)),(3) plication memoryrequests by1.87% on average across all
H(highmemory-intensity,RBMPKI∈[10+)).Todoso,we workloads(notshown)duetopreventiverefreshes.Second,
ABACuScanefficientlypreventRowHammerbitflipseven
obtaintheRBMPKIvaluesoftheapplicationsbyanalyzing
eachapplication’sSimPoint[189]traces(200Minstructions). atverylowN .AtsuchafutureN of125,ABACuSin-
|     |     |     |     |     |     |     | RH  |     | RH  |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
duces1.45%(12.37%)slowdownonaverage(atmaximum)
Allofthesetracesareopen-sourced[190].
Table4:Evaluatedsingle-coreworkloads across all workloads. At this N RH ,the average memory la-
tencyincreasesby2.72%acrossallworkloadsonaveragedue
| RBMPKI |     | Workloads |     |     |     |     |     |     |     |     |     |
| ------ | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
topreventiverefreshoperations.Weattributetheincreasing
519.lbm,459.GemsFDTD,450.soplex,h264_decode,
trendinABACuS’saverageslowdowntothelargernumberof
| [10+) | 520.omnetpp,433.milc,434.zeusmp,bfs_dblp, |     |     |     |     |     |     |     |     |     |     |
| ----- | ----------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
preventiverefreshoperationsperformedbyABACuSatlower
| (High) | 429.mcf,549.fotonik3d,470.lbm,bfs_ny, |     |     |     |     |     |     |     |     |     |     |
| ------ | ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
bfs_cm2003,437.leslie3d,gups N RH values.Moreworkloadshammermorerowsmoretimes
510.parest,462.libquantum,tpch2,wc_8443, inarefreshwindowasN reduces,whichleadstobothi)
RH
ycsb_aserver,473.astar,jp2_decode,436.cactusADM, ABACuSrowactivationcounters(RACs)incrementingfaster
[2,10)
557.xz,ycsb_cserver,ycsb_eserver,471.omnetpp,
(Medium) 483.xalancbmk,505.mcf,wc_map0,jp2_encode, andii)moreABACuSRACsreachingthepreventiverefresh
tpch17,ycsb_bserver,tpcc64,482.sphinx3 thresholdearlier(andABACuSperformingcostlypreventive
502.gcc,544.nab,h264_encode,507.cactuBSSN,
|     |     |     |     |     | refresh | operations). |     | In contrast | to the trend | in the | average |
| --- | --- | --- | --- | --- | ------- | ------------ | --- | ----------- | ------------ | ------ | ------- |
525.x264,ycsb_dserver,531.deepsjeng,526.blender,
slowdown,weobservethatABACuSinducesasmallermax-
435.gromacs,523.xalancbmk,447.dealII,508.namd,
| [0,2) |                                             |     |     |     | imumslowdownasN |     |     | reduces.ThisisbecauseABACuS |     |     |     |
| ----- | ------------------------------------------- | --- | --- | --- | --------------- | --- | --- | --------------------------- | --- | --- | --- |
|       | 538.imagick,445.gobmk,444.namd,464.h264ref, |     |     |     |                 |     |     | RH                          |     |     |     |
(Low)
ycsb_abgsave,458.sjeng,541.leela,tpch6, implementsmorerowactivationcountersatlowerN RH val-
511.povray,456.hmmer,481.wrf,grep_map0, uesandaverymemory-intensiverandomaccessworkload
500.perlbench,403.gcc,401.bzip2 (e.g.,gups)cannotquicklyexhaustallABACuSRACsandin-
creasesthespillovercountervaluetotherefreshcyclethresh-
9 Evaluation
|     |     |     |     |     | old | slower(than | at higherN |     | values) such | that | ABACuS |
| --- | --- | --- | --- | --- | --- | ----------- | ---------- | --- | ------------ | ---- | ------ |
RH
We1)analyzeABACuS’ssystemperformanceandDRAM lessfrequentlyperformsrefreshcycles(§4.2).
DRAMEnergyOverhead.Fig.8presentstheDRAMenergy
| energy overheads | and | compare ABACuS’s | system | perfor- |     |     |     |     |     |     |     |
| ---------------- | --- | ---------------- | ------ | ------- | --- | --- | --- | --- | --- | --- | --- |
manceandDRAMenergyoverheadstostate-of-the-artmiti- consumptionforallsingle-coreworkloadsforfourdifferent
RowHammerthresholdswhenexecutedonasystemthatuses
gationmechanisms(§9.1),2)showtheeffectofthenumber
ABACuS,normalizedtoabaselinesystemthatdoesnotem-
| of banks | in the system | on ABACuS’s | performance,and | 3)  |     |     |     |     |     |     |     |
| -------- | ------------- | ----------- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
analyzeABACuS’sperformanceunderRowHammerattacks. ployanyRowHammermitigationmechanism.
ygrene MARD 2.0
9.1 SystemPerformanceandDRAMEnergy dezilamroN noitubirtsid NRH=1000 (leftmost box)
NRH=500
NRH=250
System Performance Overhead. Fig. 7 presents the per- 1.5 NRH=125 (rightmost box) gups
formance(ininstructionspercycle)ofallsingle-corework-
Baseline DRAM energy
| loads (grouped | into three | categories | and sorted | based on |     | 1.0 |     |     |     |     |     |
| -------------- | ---------- | ---------- | ---------- | -------- | --- | --- | --- | --- | --- | --- | --- |
RBMPKI;seeTable4)forfourdifferentnear-futureandvery low (<2) medium (<10) high (>10)
Row buffer misses per kilo instructions (RBMPKI)
lowRowHammerthresholdswhenexecutedonasystemthat
Figure8:DRAMenergyforsingle-coreapplicationsforfour
usesABACuS,normalizedtoabaselinesystemthatdoesnot
employanyRowHammermitigationmechanism. differentRowHammerthresholds(lowerisbetter)
| 1588    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

WemaketwokeyobservationsfromFig.8.First,ABACuS
inducesminorDRAMenergyoverheadatN =1000.ABA-
RH
CuS increases DRAM energy consumption by only 1.66%
(2.02×)onaverage(atmaximum)acrossallevaluatedwork-
loads.Second,ABACuSincreasesDRAMenergyconsump-
tion by 1.27% (30.46%) on average (maximum) across all
workloadsatN =125.WeattributetheDRAMenergyover-
RH
headstoi)increasedDRAMactivation,precharge,andcom-
mandbusenergyinducedbythepreventiverefreshoperations,
andii)increasedDRAMbackground(standby)energycon-
sumptionduetoincreasedexecutiontimeforapplications.
PerformanceComparison.Fig.9presentstheperformance
impactofABACuSandfourstate-of-the-artmechanismsona
single-coresystemforfourdifferentRowHammerthresholds,
normalizedtothebaselinesystem.
1.1
1.0
0.9
0.8
0.7
0.6
0.5
0.4
0.3
0.2
1000 500 250 125
RowHammer Threshold (NRH)
noitubirtsiD
CPI
dezilamroN
when a counter in its row count cache (RCC) needs to be
evictedtotherowcounttable(RCT)inDRAM,andii)an
ACT andaread(RD)commandwhenacounterneedstobe
retrievedfromtheRCTandplacedintheRCC.Theseoper-
ationsincuradditionalperformanceoverheadsduetoi)row
buffermissesthatinterferewithapplicationmemoryrequests,
andii)DRAMbanksbeingunavailableduringRCCeviction
andRCTaccessoperations,ontopoftheoverheadscaused
bypreventiverefreshoperations.Forexample,atanN of
RH
125,the Hydra-based system has i) a row buffer miss rate
6.22%largerthanthatofABACuSandii)anaveragememory
latencyexperiencedbyapplicationmemoryrequests20.94%
higherthanthatofABACuSonaverageacrossallworkloads.
Sixth,Graphene[102]incursslightlyhigherperformance
overheadthanABACuSonaverageacrossallworkloadsatan
N of125.EventhoughABACuS,comparedtoGraphene,
ABACuS Graphene Hydra REGA PARA RH
Baseline IPC performs2.06×morepreventiverefreshoperationsasABA-
CuS’ssharedactivationcountersreachthepreventiverefresh
thresholdfaster,theamountoftimewhereatleastoneDRAM
bankisunavailable(forservingapplicationmemoryrequests)
becauseofpreventiverefreshisanestimated7.73×higherin
GraphenecomparedtoABACuS.OnceanABACuSactiva-
tioncounterreachesN ,ABACuSperforms64preventive
RH
Figure9:PerformancecomparisonofABACuSvs.state-of- refreshoperations(toall64victimrowsin32banksofthe
the-artmitigationtechniquesforsingle-coreworkloadsatfour rank)inquicksuccession.Thememorycontrollertakesap-
differentRowHammerthresholds proximately170ns7toissueallactivateandprechargecom-
WemakesixkeyobservationsbasedonFig.9.First,ABA- mandsthatmakeupapreventiverefreshoperation,leveraging
CuSoutperformsHydra,REGA,andPARAatRowHammer bank-levelparallelism.Incontrast,issuingtwopreventivere-
thresholdsbelow1000andperformssimilarlytoGraphene freshoperationstoasinglebanktakesapproximately90ns,an
at all tested RowHammer thresholds on average across all alreadylargefractionof170ns.KeepingatleastoneDRAM
workloads.Second,ABACuSoutperformsHydraandPARA bankunavailableforalongertotaltimeincreasesthecritical
atN =1000.Third,REGA[177]atN =1000doesnot pathforapplicationmemoryrequestsinGraphenebyalarger
RH RH
incuranyperformanceoverhead.AtN =1000,REGAcan amount than in ABACuS. As such,the amount of time in
RH
hidethelatencyofapreventiverefreshbehindthelatencyof whichtheprocessorcannot executeinstructionsduetothe
aDRAMrowaccess(i.e.,preventiverefreshhappensconcur- re-order buffer being full is higher by 1.87% in Graphene
rentlywithaDRAMrowaccessatthenominalt definedin comparedtoABACuSonaverageacrossallworkloads.
RC
theDDR4standard[158]).However,REGAincursincreas- Fig. 10 shows ABACuS and four state-of-the-art
inglyhigheroverheadsasN RH reducesbecauseREGAneeds mechanisms’ performance impact in terms of weighted
toperformmultiplepreventiverefreshesoneachDRAMrow speedup[191–193]forfourdifferentN valuesonaneight-
RH
access. To perform 8 preventive refreshes on each DRAM coresystem,normalizedtothebaselinesystem.
rowaccessatanN of125,REGAincreasest fromthe
RH RC
nominalvalueof45.0ns[158]to167.5ns,whereREGAin-
1.0
duces16.65%performanceoverheadonaverageacrossall
0.8
workloadsastheaveragememoryaccesslatencyincreases.
0.6
Fourth,PARA[1]performstheworstamongallevaluated
0.4
mechanisms.PARAincurs5.47%and31.08%performance
0.2
overheads on average across all workloads at N = 1000
RH 0.0
1000 500 250 125
and125,respectively,becauseitperformsmanyunnecessary RowHammer Threshold (NRH)
refreshoperations[14,104].
Fifth,Hydra[106]incurs1.80%,3.33%,5.70%,and9.75%
higherperformanceoverheadsthanABACuSforN of1000,
RH
500,250,and125,respectively,onaverageacrossallwork-
loads.Inadditiontoperformingpreventiverefreshoperations,
Hydraalsoperformsi)anACT andawrite(WR)command
pudeepS
dethgieW
dezilamroN
ABACuS Graphene Hydra REGA PARA
Baseline weighted speedup
Figure10:Performancecomparisonformulti-programmed
(8core)workloadsatfourdifferentRowHammerthresholds
7Calculatedasthetimeittakestoactivateandprechargetworowsinthe
samebank(2∗tRC)plusthenumberofbanksmultipliedbytheminimum
timebetweentwosuccessiveACT commandstodifferentbanksinthesame
rank(32∗tRRD).
USENIX Association 33rd USENIX Security Symposium 1589

WemakethreekeyobservationsfromFig.10.First,ABA- 1000,500,250,and125,respectively.AtaverylowN =
RH
CuSinducessmallsystemperformanceoverheadacrossall 125,ABACuS’sDRAMenergyoverheadis19.64%,70.41%,
evaluatedworkloadsandRowHammerthresholds.ABACuS and33.99%smallerthanHydra,REGA,andPARA,onaver-
has0.77%,1.19%,2.29%,and4.48%performanceoverhead ageacrossallevaluatedworkloads.Grapheneinduces3.95%
onaverageacrossallworkloadsforN of1000,500,250, lowerDRAMenergyoverheadthanABACuSatN =125.
|     |     |     |     | RH  |     |     |     |     |     |     |     |     | RH  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
and 125,respectively. Second,Hydra incurs 2.56% higher Summary.WeconcludethatABACuSinducessmallsystem
performanceoverheadthanABACuSatanN of1K.Third, performanceandDRAMenergyoverheadsonaverageacross
RH
ABACuS outperforms Hydra,REGA,and PARA at an ex- all tested single-core and multi-core workloads for N RH =
tremeRowHammerthresholdof125.AtsuchN ,ABACuS 1000,500,250,and125.ABACuS’sperformanceandDRAM
RH
incurs only 4.48% performance overhead, whereas Hydra, energyoverheadsareclosertothemost-performance-efficient
REGA,andPARAincur16.49%,29.31%,and40.16%per- state-of-the-artmechanism ([102]). ABACuS outperforms
formanceoverheadonaverageacrossallworkloads. andconsumeslessDRAMenergythanthemost-area-efficient
Energy Comparison. Fig. 11 presents the DRAM energy state-of-the-art(counter-based)mechanism([106]).
| consumption |     | of ABACuS | and | four state-of-the-art |     | mecha- |     |     |     |     |     |     |     |
| ----------- | --- | --------- | --- | --------------------- | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
9.2 SensitivitytoNumberofBanks
nismsonasingle-coresystemforfourdifferentRowHammer
thresholds,normalizedtothebaselinesystem. Werun16-,32-,and64-bank(1-,2-,and4-rank)simula-
noitubirtsiD ygrenE dezilamroN tions using ABACuS andthe baseline system. We observe
|     |     | ABACuS | Graphene | Hydra REGA | PARA |     |                                                   |     |     |     |     |     |     |
| --- | --- | ------ | -------- | ---------- | ---- | --- | ------------------------------------------------- | --- | --- | --- | --- | --- | --- |
| 2.5 |     |        |          |            |      |     | thatABACuScanpreventRowHammerbitflipswithlowover- |     |     |     |     |     |     |
headinsystemsthatusememorymoduleswithdifferentnum-
2.0
|     |     |     |     |     |     |     | bersofbanks(ranks).AtN                           |     |     | RH =125,ABACuSincurs1.58%, |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------ | --- | --- | -------------------------- | --- | --- | --- |
| 1.5 |     |     |     |     |     |     | 1.50%,and2.60%performanceoverheadsfor16-,32-,and |     |     |                            |     |     |     |
64-bankconfigurations,respectively,onaverage(geometric
1.0
Baseline DRAM energy mean)acrossallevaluatedsingle-coreworkloads.
|     | 1000 |     | 500 | 250 |     | 125 |     |     |     |     |     |     |     |
| --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
RowHammer Threshold (NRH)
9.3 AdversarialWorkloads
Figure11:DRAMenergycomparisonforsingle-corework-
loadsatfourdifferentRowHammerthresholds ABACuS securely prevents bitflips under RowHammer
|     |     |     |     |     |     |     | attacks | (§5). We | demonstrate |     | that, in a | dual-core | system, |
| --- | --- | --- | --- | --- | --- | --- | ------- | -------- | ----------- | --- | ---------- | --------- | ------- |
FromFig.11,wemaketwoobservations.First,ABACuS
ABACuSincurssmallerperformanceoverheadsthanHydra,
| induces | smaller | DRAM | energy | overhead than | other | evalu- |     |     |     |     |     |     |     |
| ------- | ------- | ---- | ------ | ------------- | ----- | ------ | --- | --- | --- | --- | --- | --- | --- |
atedmitigationmechanisms(exceptGraphene)onaverage REGA,andPARAfortheevaluatedsingle-coreworkloadson
average,whileonecoreinthesystemexecutesatraditional
| acrossallworkloadsforN |     |     | RH  | <1000. Second,atanN |     | RH = |                 |     |         |                   |     |     |         |
| ---------------------- | --- | --- | --- | ------------------- | --- | ---- | --------------- | --- | ------- | ----------------- | --- | --- | ------- |
|                        |     |     |     |                     |     |      | RowHammeraccess |     | pattern | (RowHammerAttack) |     |     | thatre- |
1000,ABACuSinduces1.34%and1.36%smallerDRAMen-
peatedlyactivates32rowsineachbankinabank-interleaved
ergyoverheadsthanREGAandPARA,respectively,because
manner.WealsodeveloptwospecializedRowHammeraccess
| REGA | preventively |     | refreshes | one row with | every | DRAM |     |     |     |     |     |     |     |
| ---- | ------------ | --- | --------- | ------------ | ----- | ---- | --- | --- | --- | --- | --- | --- | --- |
row activation (at N = 1000) and PARA performs many patterns(whichareopensource[190]):Hydra-Adversarial
RH
andABACuS-Adversarial.1)Hydra-Adversarialexacerbates
unnecessaryrefreshoperations.ABACuSinduces1.66%av-
Hydra’sRowCountCacheevictionratetosignificantlyin-
| erage(2.02×maximum)DRAMenergyoverheadatthisN |     |     |     |     |     | ,   |     |     |     |     |     |     |     |
| -------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
RH
whichisclosetoHydra’s0.73%average(1.11×maximum) crease the throughput of Hydra’s DRAM read and write
|     |     |     |     |     |     |     | requests. | 2) ABACuS-Adversarial |     |     | rapidly | increments | the |
| --- | --- | --- | --- | --- | --- | --- | --------- | --------------------- | --- | --- | ------- | ---------- | --- |
DRAMenergyoverhead.
Fig.12showstheDRAMenergyconsumptionofABACuS spillover counter value to cause frequent refresh cycles
(§4.2).Allthreeaccesspatterns(RowHammerAttack,Hydra-
andfourstate-of-the-artmechanismsforfourdifferentN
|     |     |     |     |     |     | RH  | Adversarial,andABACuS-Adversarial)incurthesame,sub- |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------------------------------------------- | --- | --- | --- | --- | --- | --- |
onaneight-coresystem,normalizedtothebaselinesystem.
|     |     |     |     |     |     |     | stantially | highrate | ofACT | commands | in  | the memory | con- |
| --- | --- | --- | --- | --- | --- | --- | ---------- | -------- | ----- | -------- | --- | ---------- | ---- |
noitubirtsiD ygrenE dezilamroN
ABACuS Graphene Hydra REGA PARA troller.ThememorycontrollerissuesanACT commandevery
4
|     |     |     |     |     |     |     | 20ns while                                         | executing | each | access | pattern. | Fig. | 13 shows |
| --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------- | --------- | ---- | ------ | -------- | ---- | -------- |
| 3   |     |     |     |     |     |     | theperformanceimpactofABACuSandthestate-of-the-art |           |      |        |          |      |          |
mechanismsonallevaluatedsingle-coreworkloadsinadual-
2
coresystemwhenthesecondcoreexecutesoneofthethree
| 1   |     |     |     |     |     |     | RowHammeraccesspatterns. |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------ | --- | --- | --- | --- | --- | --- |
Baseline DRAM energy
|     |      |     |     |     |     |     | We make | three | major | observations. |     | First, ABACuS | in- |
| --- | ---- | --- | --- | --- | --- | --- | ------- | ----- | ----- | ------------- | --- | ------------- | --- |
|     | 1000 |     | 500 | 250 |     | 125 |         |       |       |               |     |               |     |
RowHammer Threshold (NRH)
ducesonly0.88%performanceoverheadonaverageacross
Figure12:DRAMenergycomparisonformulti-programmed
allworkloadswhenonecoreexecutestheRowHammerat-
(8core)workloadsatfourdifferentRowHammerthresholds
tack,whereasGraphene,Hydra,REGA,andPARAinduce
From Fig. 12,we observe thatABACuS induces 2.12%, 0.61%,3.03%,4.43%,and14.62%,respectively.Second,Hy-
2.44%,3.25%,and4.76%DRAMenergyoverheadatN = drainducesalarge73.96%averageslowdownwhenonecore
RH
| 1590    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

1.0
0.8
0.6
0.4
0.2
0.0
RowHammer Attack Hydra-Adversarial ABACuS-Adversarial
noitubirtsiD
CPI
dezilamroN
ABACuS Graphene Hydra REGA PARA dual-coresystemwiththethreeRowHammeraccesspatterns
Baseline IPC describedearlierinthissection.
1.0
0.8
RowHammer Attack Hydra-Adversarial ABACuS-Adversarial
Figure13:Performancecomparisonforsingle-coreworkloads
withthreedifferentRowHammeraccesspatterns(N =500) RH
executestheHydra-Adversarialaccesspattern.Weattribute
this overhead to the high rate of Row Count Cache (RCC)
evictionstheHydra-Adversarialaccesspatternincurs.Hydra
evicts1.13RCCentriesperlastlevelcachemissonaverage
acrossallworkloads.ThememorycontrollerservesanRCC
evictionbyissuinghigh-priorityWRandRDDRAMrequests
(i.e.,WRandRDrequestscausedbyRCCevictionsareon
thecriticalpathofworkloadmainmemoryrequests).Forthe
sameaccesspattern,ABACuSincursonly0.79%onaverage
acrossallworkloads.Third,ABACuSinduces9.20%perfor-
manceoverhead,onaverageacrossallworkloadswhenone
coreexecutestheABACuS-Adversarialaccesspattern.This
isbecausetheABACuS-Adversarialaccesspatterntriggers
multipleABACuSrefreshcycles,duringwhichnomemory
requestcanbeserviced,whilethesingle-coreworkloadexe-
cutes.Thesameaccesspatternincurs48.08%performance
overheadonaverageacrossallworkloadsforHydra.
We conclude that ABACuS incurs almost-negligible ad-
ditionalperformanceoverheadforbenignworkloadswhen
anothercoreexecutesatraditionalRowHammerattack.Spe-
cializedadversarialaccesspatternscanexacerbatesuchover-
headsbyfrequentlytriggeringABACuSrefreshcycles.
9.3.1 ImprovingABACuS’sPerformance
Aworkloadmay,intentionally(e.g.,ABACuS-Adversarial)
orunintentionally(e.g.,gups),rapidlyincrementthespillover
counter’s value,frequently triggering refresh cycles where
ABACuSissuesarefreshcommandtoeachDRAMrowID
in a rank,and cause substantial performance overheads in
an ABACuS-based system. To prevent such overheads, a
less-area-constrained version of ABACuS can remove the
spillovercounterandimplementonesharedactivationcounter
(ABACuScounter)perDRAMrowID(i.e.,ABACuS-Big’s
N (Table2)isequaltothenumberofrowsinaDRAM
entries
bank).WedesignandevaluateABACuS-Big,whichimple-
mentsoneABACuScounterperDRAMrowID.TheABA-
CuScounterinABACuS-Bigisupdatedinthesamewayas
in ABACuS. ABACuS-Big implements as many ABACuS
countersastherearerowsinabank(i.e.,thereisa1-1map-
ping between ABACuS counters and DRAM row IDs) to
keep precise track of every sibling row’s maximum activa-
tioncountandABACuS-Bigdoesnotneedtouseaspillover
counter.Fig.14showstheperformanceimpactofABACuS
andABACuS-Bigonallevaluatedsingle-coreworkloadsina
CPI
dezilamroN
noitubirtsiD Baseline IPC ABACuS ABACuS-Big
Figure14:PerformanceofABACuS&-Big(N =500) RH
We observe that ABACuS-Big incurs only 0.28% per-
formance overhead on average across all workloads for
theABACuS-Adversarialpattern,whereasABACuSincurs
9.20%becauseABACuS-Bigdoesnotperformanyrefresh
cycleswherethememorycontrollerisbusyrapidlyissuing
REF commands.WeevaluateABACuS-Big’schipareausing
themethodologydescribedin§7.1. ABACuS-Bigrequires
40bits(8bitsforrowactivationcounter,32bitsforsibling
activationvector)ofstorageperDRAMrow,amountingto
640KiBon-chipstorageatanN of500for128KDRAM
RH
rows.ABACuS-Bigtakesup0.48mm2chiparea(0.20%of
ahigh-endIntelXeonprocessor’sarea[178]).Weconclude
thatABACuS-Bigisahigh-performanceimplementationof
theABACuSdesign,whichsystemdesignersthathavefewer
chipareaconstraintscanchoosetoimplement,thatimproves
systemperformanceunderadversarialworkloadsandsome
benignworkloads(e.g.,gups)comparedtoABACuS.
10 RelatedWork
Toourknowledge,ABACuSisthefirstworkthatmitigates
RowHammerefficientlyandscalablyatverylowRowHam-
merthresholds(e.g.,125)withoutincurringlargearea,perfor-
mance,orenergyoverheads.Sections7and9alreadyqual-
itatively and quantitatively compare ABACuS to the most
relevantstate-of-the-artmechanisms[1,102,106,177].This
sectiondiscussesotherRowHammermitigationmechanisms.
Hardware-based Mitigation Mechanisms. Many prior
works [1,87,88,97,98,100–102,104–107,110–112,114,
116,117,122,123,126,131,133,134,173,194–196] pro-
pose hardware-based mitigation mechanisms to prevent
RowHammerbitflips.Weclassifytheseintothreemaincat-
egories. 1) Probabilistic preventive refresh (PPR) mecha-
nisms[1,97,100,114,116,122,133,194]preventivelyrefresh
victimrowsbasedonaprobability.PPRmechanismsincur
impractical performance overheads at very low RowHam-
mer thresholds as they perform many unnecessary preven-
tiverefreshoperations.Arecentwork[195]proposesanew
methodologyforconfiguringPPRs.2)Deterministicpreven-
tive refresh (DPR) mechanisms [88,98,101,102,105,107,
110–112,117,123,126,131,134,173,196] trackactivation
counts of aggressor rows and preventively refresh victim
rows. DPR mechanisms incur less performance overhead
thanPPRmechanisms(fromfewerunnecessarypreventive
refreshoperations)atthecostoflargerchipareaoverheadto
storeaggressorrowactivationcounters.and3)Deterministic
aggressorrowaccessthrottling(DAT)mechanisms[1,87,104]
track activation counts of aggressor rows and preventively
USENIX Association 33rd USENIX Security Symposium 1591

blockmemoryaccessestoaggressorrows.DATmechanisms techniqueincurssignificantlysmallerarea,performance,and
incuraveragesystemperformanceandtotalchipareaover- DRAMenergyoverheadsformodernandfutureDRAMchips.
heads similar to DPR mechanisms [104]. However, exist- Ourtechniqueachievesthisbysharingactivationcountersof
ing DAT mechanisms can incur delays in the order of mi- rowsthathasthesamerowIDindifferentbanks.WhileABA-
crosecondsonmemorydemandrequests(e.g.,loadinstruc- CuSefficientlyandsecurelypreventsRowHammerbitflips,it
tions)[104,106,117]. alsoscaleswellwithworseningRowHammervulnerability
Software-basedMitigationMechanisms.Manyworks[38, downtoRowHammerthreshold(N )=125.
RH
44,93,108,113,118,132]proposesoftware-basedmitigation Acknowledgments
mechanismstoavoidhardwaremodifications.Unfortunately, Wethankourshepherdandtheanonymousreviewersof
itisnotpossibleforthesemechanismstomonitorallmemory USENIX Security 2023. We thank the SAFARI Research
requests,and thus most of these mechanisms have already Groupmembersforfeedbackandthestimulatingintellectual
beendefeatedbyrecentattacks[30,37,42,46,49,52,197]. environment.Weacknowledgethegenerousgiftfundingpro-
Integrity-based Mitigation Mechanisms. Several videdbyourindustrialpartners(especiallyGoogle,Huawei,
works [115,119,127,198] propose integrity check mech- Intel,Microsoft,VMware),whichhasbeeninstrumentalin
anisms to detect and correct bitflips that may have been enabling our decade-long research on read disturbance in
induced by RowHammer. Unfortunately, it is either not DRAM and memory systems. This work was in part sup-
possible,difficult,or prohibitively expensive to correct all portedbyaGoogleSecurityandPrivacyResearchAwardand
possible RowHammer bitflips using these mechanisms. theMicrosoftSwissJointResearchCenter.
However,thesemechanismscanbecombinedwithABACuS
References
toimproveoverallsystemreliabilityandfutureworkcould
demonstratethebenefitsofcombiningthemwithABACuS.
[1] Y.Kimetal.FlippingBitsinMemoryWithoutAccessingThem:An
RowHammerMitigationinCommodityChips. DRAM ExperimentalStudyofDRAMDisturbanceErrors.InISCA,2014.
manufacturersemployRowHammermitigationmechanisms, [2] OnurMutlu. TheRowHammerProblemandOtherIssuesWeMay
commonlyreferredtoastargetrowrefresh(TRR),incom- FaceasMemoryBecomesDenser.InDATE,2017.
modityDRAMchips[158,163]withoutpubliclydocumenting [3] ThomasYangetal.Trap-AssistedDRAMRowHammerEffect.EDL,
theirdetaileddesigns.Thesemechanismstypicallydonotin- 2019.
duceanyperformanceoverheadbecausetheytakeaction(e.g., [4] OnurMutluetal.RowHammer:ARetrospective.TCAD,2019.
refreshavictimrow)whentheDRAMchipisbusyperform- [5] KyungbaeParketal. StatisticalDistributionsofRow-Hammering
InducedFailuresinDDR3Components.MicroelectronicsReliability,
ingaperiodicrefreshoperation(i.e.,theirvictimrowrefresh
2016.
latency is hidden by the latency of performing a periodic
[6] Kyungbae Parket al. Experiments andRootCauseAnalysis for
refresh operation). However,recent studies experimentally
Active-PrechargeHammeringFaultinDDR3SDRAMunder3xnm
demonstratethatspecializedadversarialaccesspatternscan Technology.MicroelectronicsReliability,2016.
defeatsomeofthesemechanisms[15,29,54–56,128].Are- [7] AndrewJ.Walkeretal.OnDRAMRowHammerandthePhysicson
centwork[121]developsatoolthatcanautomaticallyinfer Insecurity.IEEETED,2021.
parametersofTRRmechanisms.Morerecentworksfromin- [8] Seong-Wan Ryu etal. OvercomingtheReliabilityLimitation in
dustrydesignnewin-DRAMRowHammermitigationmecha- theUltimatelyScaledDRAMusingSiliconMigrationTechniqueby
HydrogenAnnealing.InIEDM,2017.
nisms[135,199].Unfortunately,thesemechanismscannotor
[9] Chia Yanget al. Suppression ofRowHammerEffectbyDoping
arenotproventodeterministicallypreventallRowHammer
Profile Modification in Saddle-Fin ArrayDevices forSub-30-nm
bitflips. DRAMTechnology.TDMR,2016.
Device-level Mechanisms for Mitigating RowHammer.
[10] Chia-MingYangetal.ScanningSpreadingResistanceMicroscopy
Severalpriorworks[8,9,130,200]designnewDRAMcellsor forDopingProfileinSaddle-FinDevices. IEEETransactionson
arrayswithimprovedRowHammerresilience.Unfortunately, Nanotechnology,2017.
theseworksalonecannotcompletelypreventRowHammer [11] SKGautametal.RowHammeringMitigationUsingMetalNanowire
inSaddleFinDRAM.IEEETED,2019.
bitflips(butcouldeffectivelyincreasetheRowHammerthresh-
[12] YichenJiangetal.QuantifyingRowHammerVulnerabilityforDRAM
olds).Theycanbeusedwithotherhardware-/software-based
Security.InDAC,2021.
mitigationtechniquestomitigateRowHammer.
[13] Onur Mutlu et al. Fundamentally Understanding and Solving
11 Conclusion RowHammer.InASP-DAC,2023.
WeintroducedanewRowHammermitigationmechanism [14] JeremieS.Kimetal. RevisitingRowHammer:AnExperimental
thatpreventsRowHammerbitflipsatlowarea,performance, AnalysisofModernDevicesandMitigationTechniques. InISCA,
2020.
andenergyoverheadsformodernandfutureDRAMchipsthat
[15] PietroFrigoetal.TRRespass:ExploitingtheManySidesofTarget
areveryvulnerabletoRowHammer(e.g.,withRowHammer
RowRefresh.InS&P,2020.
thresholdsaslowas125).ComparedtoexistingRowHam-
[16] A.GirayYag˘lıkçıetal.UnderstandingRowHammerUnderReduced
mermitigationmechanisms,ourtechnique,all-bankactiva-
WordlineVoltage:AnExperimentalStudyUsingRealDRAMDe-
tioncountersforscalableRowHammermitigation(ABACuS) vices.InDSN,2022.
1592 33rd USENIX Security Symposium USENIX Association

[17] LoisOrosaetal. ADeeperLookintoRowHammer’sSensitivities: [43] MoritzLippetal.Nethammer:InducingRowhammerFaultsThrough
ExperimentalAnalysisofRealDRAMChipsandImplicationson NetworkRequests.arXiv:1805.04956[cs.CR],2018.
FutureAttacksandDefenses.InMICRO,2021.
[44] VictorvanderVeenetal.GuardION:PracticalMitigationofDMA-
BasedRowhammerAttacksonARM.InDIMVA,2018.
| [18] OnurMutlu. | RowHammer. | TopPicksinHardwareandEmbedded |     |     |     |     |
| --------------- | ---------- | ----------------------------- | --- | --- | --- | --- |
Security,2018. [45] PietroFrigoetal.GrandPwningUnit:AcceleratingMicroarchitec-
[19] Onur Mutlu et al. Fundamentally Understanding and Solving turalAttackswiththeGPU.InS&P,2018.
RowHammer.arXiv:2211.07613[cs.CR],2022.
|     |     |     |     | [46] LucianCojocaretal. | ExploitingCorrectingCodes:OntheEffec- |     |
| --- | --- | --- | --- | ----------------------- | ------------------------------------- | --- |
InS&P,
[20] ApostolosPFournarisetal.ExploitingHardwareVulnerabilitiesto tivenessofECCMemoryAgainstRowhammerAttacks.
2019.
AttackEmbeddedSystemDevices:ASurveyofPotentMicroarchi-
tecturalAttacks.Electronics,2017. [47] SangwooJietal.PinpointRowhammer:SuppressingUnwantedBit
[21] DamianPoddebniaketal.AttackingDeterministicSignatureSchemes FlipsonRowhammerAttacks.InASIACCS,2019.
usingFaultAttacks.InEuroS&P,2018.
[48] SanghyunHongetal.TerminalBrainDamage:ExposingtheGrace-
lessDegradationinDeepNeuralNetworksUnderHardwareFault
| [22] AndreiTataretal. | Throwhammer:RowhammerAttacksOverthe |     |     |     |     |     |
| --------------------- | ----------------------------------- | --- | --- | --- | --- | --- |
Attacks.InUSENIXSecurity,2019.
NetworkandDefenses.InUSENIXATC,2018.
[23] SebastienCarreetal. OpenSSLBellcore’sProtectionHelpsFault [49] AndrewKwongetal.RAMBleed:ReadingBitsinMemoryWithout
| Attack.InDSD,2018. |     |     |     | AccessingThem.InS&P,2020. |     |     |
| ------------------ | --- | --- | --- | ------------------------- | --- | --- |
[50] LucianCojocaretal.AreWeSusceptibletoRowhammer?AnEnd-
| [24] AlessandroBarenghietal. |     | Software-OnlyReverseEngineeringof |     |     |     |     |
| ---------------------------- | --- | --------------------------------- | --- | --- | --- | --- |
to-EndMethodologyforCloudProviders.InS&P,2020.
PhysicalDRAMMappingsforRowhammerAttacks.InIVSW,2018.
[51] ZaneWeissmanetal.JackHammer:EfficientRowhammeronHetero-
| [25] ZhenkaiZhangetal. | TriggeringRowhammerHardwareFaultson |     |     |     |     |     |
| ---------------------- | ----------------------------------- | --- | --- | --- | --- | --- |
ARM:ARevisit.InASHES,2018. geneousFPGA–CPUPlatforms.arXiv:1912.11523[cs.CR],2020.
[52] ZhiZhangetal.PTHammer:Cross-User-Kernel-BoundaryRowham-
[26] SaraniBhattacharyaetal.AdvancedFaultAttacksinSoftware:Ex-
merThroughImplicitAccesses.InMICRO,2020.
ploitingtheRowhammerBug.FaultTolerantArchitecturesforCryp-
tographyandHardwareSecurity,2018. [53] FanYaoetal. Deephammer: DepletingtheIntelligenceofDeep
NeuralNetworksThroughTargetedChainofBitFlips.InUSENIX
| [27] SAFARI                              | Research Group. | RowHammer | — GitHub Repository. |                |     |     |
| ---------------------------------------- | --------------- | --------- | -------------------- | -------------- | --- | --- |
| https://github.com/CMU-SAFARI/rowhammer. |                 |           |                      | Security,2020. |     |     |
[54] FinndeRidderetal.SMASH:SynchronizedMany-SidedRowham-
| [28] MarkSeaborn | etal. ExploitingtheDRAMRowhammerBugto |     |     |     |     |     |
| ---------------- | ------------------------------------- | --- | --- | --- | --- | --- |
merAttacksfromJavaScript.InUSENIXSecurity,2021.
| GainKernelPrivileges. |     | http://googleprojectzero.blogspot. |     |     |     |     |
| --------------------- | --- | ---------------------------------- | --- | --- | --- | --- |
com.tr/2015/03/exploiting-dram-rowhammer-bug-to-gain. [55] HasanHassanetal.UncoveringIn-DRAMRowHammerProtection
html,2015. Mechanisms:ANewMethodology,CustomRowHammerPatterns,
[29] VictorvanderVeenetal. Drammer: DeterministicRowhammer andImplications.InMICRO,2021.
AttacksonMobilePlatforms.InCCS,2016. [56] PatrickJattkeetal. Blacksmith: ScalableRowhammeringin the
FrequencyDomain.InSP,2022.
[30] DanielGrussetal.Rowhammer.js:ARemoteSoftware-InducedFault
AttackinJavascript.arXiv:1507.06955[cs.CR],2016. [57] MCanerToletal.TowardRealisticBackdoorInjectionAttackson
DNNsusingRowHammer.arXiv:2110.07683v2[cs.LG],2022.
| [31] KavehRazavietal. | FlipFengShui:HammeringaNeedleinthe |     |     |     |     |     |
| --------------------- | ---------------------------------- | --- | --- | --- | --- | --- |
[58] AndreasKogleretal.Half-Double:HammeringFromtheNextRow
SoftwareStack.InUSENIXSecurity,2016.
Over.InUSENIXSecurity,2022.
[32] PeterPessletal.DRAMA:ExploitingDRAMAddressingforCross-
CPUAttacks.InUSENIXSecurity,2016. [59] LoisOrosaetal.SpyHammer:UsingRowHammertoRemotelySpy
onTemperature.arXiv:2210.04084[cs.CR],2022.
| [33] YuanXiaoetal. | OneBitFlips,OneCloudFlops:Cross-VMRow |     |     |                |                 |                                  |
| ------------------ | ------------------------------------- | --- | --- | -------------- | --------------- | -------------------------------- |
|                    |                                       |     |     | [60] Zhi Zhang | et al. Implicit | Hammer: Cross-Privilege-Boundary |
HammerAttacksandPrivilegeEscalation.InUSENIXSecurity,2016.
RowhammerthroughImplicitAccesses.IEEETransactionsonDe-
| [34] ErikBosmanetal. | DedupEstMachina:MemoryDeduplicationas |     |     |     |     |     |
| -------------------- | ------------------------------------- | --- | --- | --- | --- | --- |
pendableandSecureComputing,2022.
AnAdvancedExploitationVector.InS&P,2016.
[61] LiangLiu,YananGuo,YueqiangCheng,YoutaoZhang,andJunYang.
[35] SaraniBhattacharyaetal. CuriousCaseofRowhammer:Flipping GeneratingRobustDNNwithResistancetoBit-FlipbasedAdversarial
SecretExponentBitsUsingTimingAnalysis.InCHES,2016. WeightAttack.IEEETransactionsonComputers,2022.
[36] WayneBurlesonetal.Invited:WhoistheMajorThreattoTomorrow’s [62] YaakovCohenetal.HammerScope:ObservingDRAMPowerCon-
Security?You,theHardwareDesigner.InDAC,2016.
sumptionUsingRowhammer.InCCS,2022.
[37] RuiQiaoetal.ANewApproachforRowHammerAttacks.InHOST, [63] MengxinZhengetal.TrojViT:TrojanInsertioninVisionTransform-
| 2016. |     |     |     | ers.arXiv:2208.13049[cs.LG],2022. |     |     |
| ----- | --- | --- | --- | --------------------------------- | --- | --- |
[38] FerdinandBrasseretal.Can’tTouchThis:Software-OnlyMitigation [64] MichaelFahrJretal.WhenFrodoFlips:End-to-EndKeyRecovery
AgainstRowhammerAttacksTargetingKernelMemory.InUSENIX onFrodoKEMviaRowhammer.CCS,2022.
Security,2017.
[65] YoussefTobahetal.SpecHammer:CombiningSpectreandRowham-
[39] YeongjinJangetal. SGX-Bomb:LockingDowntheProcessorvia merforNewSpeculativeAttacks.InSP,2022.
RowhammerAttack.InSOSP,2017.
|     |     |     |     | [66] AdnanSirajRakinetal. |     | DeepSteal:AdvancedModelExtractions |
| --- | --- | --- | --- | ------------------------- | --- | ---------------------------------- |
[40] MisikerAgaetal.WhenGoodProtectionsGoBad:ExploitingAnti- LeveragingEfficientWeightStealinginMemories.InSP,2022.
DoSMeasurestoAccelerateRowhammerAttacks.InHOST,2017. [67] HakanAydinetal. CyberSecurityinIndustrialControlSystems
[41] AndreiTataretal.DefeatingSoftwareMitigationsAgainstRowham- (ICS):ASurveyofRowHammerVulnerability. AppliedComputer
| mer:ASurgicalPrecisionHammer.InRAID,2018. |     |     |     | Science,2022. |     |     |
| ----------------------------------------- | --- | --- | --- | ------------- | --- | --- |
[42] DanielGrussetal.AnotherFlipintheWallofRowhammerDefenses. [68] KoksalMusetal.Jolt:RecoveringTLSSigningKeysviaRowhammer
| InS&P,2018.        |     |     |     | Faults.CryptologyePrintArchive,2022. |                                        |     |
| ------------------ | --- | --- | --- | ------------------------------------ | -------------------------------------- | --- |
| USENIX Association |     |     |     |                                      | 33rd USENIX Security Symposium    1593 |     |

[69] JianxinWangetal. ResearchandImplementationofRowhammer [95] KuljitSBainsetal.RowHammerMonitoringBasedonStoredRow
AttackMethodbasedonDomesticNeoKylinOperatingSystem.In HammerThresholdValue,2016.U.S.Patent9,384,821.
ICFTIC,2022.
[96] KuljitSBainsetal.DistributedRowHammerTracking,2016.U.S.
[70] SamLefforge. ReverseEngineeringPost-QuantumCryptography Patent9,299,400.
SchemestoFindRowhammerExploits.Bachelor’sThesis,University [97] MungyuSonetal.MakingDRAMStrongerAgainstRowHammering.
ofArkansas,2023.
InDAC,2017.
| [71] MichaelJacobFahr. | TheEffectsofSide-ChannelAttacksonPost- |     |     |     |     |
| ---------------------- | -------------------------------------- | --- | --- | --- | --- |
[98] S.M.Seyedzadehetal.MitigatingWordlineCrosstalkUsingAdap-
QuantumCryptography:InfluencingFrodoKEMKeyGenerationUs-
tiveTreesofCounters.InISCA,2018.
ingtheRowhammerExploit.Master’sthesis,UniversityofArkansas,
2022. [99] GorkaIrazoquietal.MASCAT:StoppingMicroarchitecturalAttacks
BeforeExecution.IACRCryptology,2016.
[72] AnandpreetKauretal. Work-in-Progress:DRAM-MaUT:DRAM
AddressMappingUnveilingToolforARMDevices.InCASES,2022. [100] JungMinYouetal.MRLoc:MitigatingRow-HammeringBasedon
MemoryLocality.InDAC,2019.
[73] KunbeiCaietal.OntheFeasibilityofTraining-timeTrojanAttacks
throughHardware-basedFaultsinMemory.InHOST,2022. [101] EojinLeeetal.TWiCe:PreventingRow-HammeringbyExploiting
TimeWindowCounters.InISCA,2019.
| [74] DaweiLietal. | CyberRadar:APUF-basedDetectingandMapping |     |     |     |     |
| ----------------- | ---------------------------------------- | --- | --- | --- | --- |
FrameworkforPhysicalDevices.arXiv:2201.07597,2022. [102] YeonhongParketal.Graphene:StrongyetLightweightRowHammer
Protection.InMICRO,2020.
| [75] ArmanRoohietal. | EfficientTargetedBit-FlipAttackAgainstthe |     |     |     |     |
| -------------------- | ----------------------------------------- | --- | --- | --- | --- |
LocalBinaryPatternNetwork.InHOST,2022. [103] A.GirayYag˘lıkçıetal.SecurityAnalysisoftheSilverBulletTech-
niqueforRowHammerPrevention.arXiv:2106.07084[cs.CR],2021.
[76] FelixStaudigletal.NeuroHammer:InducingBit-FlipsinMemristive
CrossbarMemories.InDATE,2022. [104] A.GirayYag˘lıkçıetal.BlockHammer:PreventingRowHammerat
LowCostbyBlacklistingRapidly-AccessedDRAMRows.InHPCA,
[77] Li-HsingYangetal.Socially-AwareCollaborativeDefenseSystem
2021.
againstBit-FlipAttackinSocialInternetofThingsandItsOnline
AssignmentOptimization.InICCCN,2022. [105] IngabKangetal. CAT-TWO:Counter-BasedAdaptiveTree,Time
|     |     |     | Window OptimizedforDRAM | Row-HammerPrevention. | IEEE |
| --- | --- | --- | ----------------------- | --------------------- | ---- |
[78] SaadIslametal.SignatureCorrectionAttackonDilithiumSignature
Access,2020.
Scheme.InEuroS&P,2022.
[79] AnandpreetKauretal. FlippingBitsLikeaPro:PreciseRowham- [106] MoinuddinQureshietal.Hydra:EnablingLow-OverheadMitigation
meringonEmbeddedDevices. IEEEEmbeddedSystemsLetters, ofRow-HammeratUltra-LowThresholdsviaHybridTracking. In
ISCA,2022.
2023.
|     |     | [107] | GururajSaileshwaretal.RandomizedRow-Swap:MitigatingRow |     |     |
| --- | --- | ----- | ------------------------------------------------------ | --- | --- |
[80] AndrewJ.Adilettaetal.Mayhem:TargetedCorruptionofRegister
HammerbyBreakingSpatialCorrelationBetweenAggressorand
andStackVariables.arXiv:2309.02545,2023.
VictimRows.InASPLOS,2022.
| [81] JianshuoDongetal. | One-bitFlipisAllYouNeed:WhenBit-flip |     |     |     |     |
| ---------------------- | ------------------------------------ | --- | --- | --- | --- |
AttackMeetsModelTraining.InICCV,2023. [108] RadheshKrishnanKonothetal.ZebRAM:ComprehensiveandCom-
patibleSoftwareProtectionAgainstRowhammerAttacks.InOSDI,
| [82] FangxinLiuetal.HyperAttack:AnEfficientAttackFrameworkfor |     |     | 2018. |     |     |
| ------------------------------------------------------------- | --- | --- | ----- | --- | --- |
HyperDimensionalComputing.InDAC,2023.
|     |     | [109] | SaruVigetal. RapidDetectionofRowhammerAttacksUsingDy- |     |     |
| --- | --- | ----- | ----------------------------------------------------- | --- | --- |
[83] M.CanerToletal. Don’tKnock!RowhammerattheBackdoorof namicSkewedHashTree.InHASP,2018.
DNNModels.InDSN,2023.
|     |     | [110] | MichaelJaeminKimetal.Mithril:CooperativeRowHammerProtec- |     |     |
| --- | --- | ----- | -------------------------------------------------------- | --- | --- |
[84] AppleInc.AbouttheSecurityContentofMacEFISecurityUpdate
tiononCommodityDRAMLeveragingManagedRefresh.InHPCA,
2015-001.https://support.apple.com/en-us/HT204934.June
2022.
2015.
|     |     | [111] | Gyu-Hyeon Lee et al. CryoGuard: | A NearRefresh-Free | Robust |
| --- | --- | ----- | ------------------------------- | ------------------ | ------ |
[85] Hewlett-PackardEnterprise.HPMoonshotComponentPackVersion DRAMDesignforCryogenicComputing.InISCA,2021.
2015.05.0,2015.
|     |     | [112] | MicheleMarazzietal. ProTRR:PrincipledyetOptimalIn-DRAM |     |     |
| --- | --- | ----- | ------------------------------------------------------ | --- | --- |
[86] LenovoGroupLtd.RowHammerPrivilegeEscalation,2015.
TargetRowRefresh.InIEEES&P,2022.
[87] ZvikaGreenfieldetal.ThrottlingSupportforRow-HammerCounters, [113] ZhiZhangetal.SoftTRR:ProtectPageTablesagainstRowhammer
2016.U.S.Patent9,251,885. AttacksusingSoftware-OnlyTargetRowRefresh.InUSENIXATC,
| [88] Dae-Hyun | Kim et al. ArchitecturalSupportforMitigating | Row | 2022. |     |     |
| ------------- | -------------------------------------------- | --- | ----- | --- | --- |
HammeringinDRAMMemories.CAL,2015.
|     |     | [114] | BireshKumarJoardaretal. | LearningtoMitigateRowHammerAt- |     |
| --- | --- | ----- | ----------------------- | ------------------------------ | --- |
[89] K.S.BainsandJ.B.Halbert.DistributedRowHammerTracking.US tacks.InDATE,2022.
PatentApp.13/631,781,April32014.
|     |     | [115] | JonasJuffingeretal.CSI:Rowhammer-CryptographicSecurityand |     |     |
| --- | --- | ----- | --------------------------------------------------------- | --- | --- |
IntegrityagainstRowhammer.InSP,2023.
| [90] K.S.Bainsetal. | Method,ApparatusandSystemforProvidinga |     |     |     |     |
| ------------------- | -------------------------------------- | --- | --- | --- | --- |
MemoryRefresh.USPatentApp.13/625,741,March272014.
|     |     | [116] | A.GirayYag˘likçietal.HiRA:HiddenRowActivationforReducing |     |     |
| --- | --- | ----- | -------------------------------------------------------- | --- | --- |
[91] K.S. Bains et al. Row HammerRefreshCommand. US Patent RefreshLatencyofOff-the-ShelfDRAMChips.InMICRO,2022.
App.13/539,415,January22014.
|     |     | [117] | AnishSaxenaetal. AQUA:ScalableRowhammerMitigationby |     |     |
| --- | --- | ----- | --------------------------------------------------- | --- | --- |
[92] K. Bains et al. Row Hammer Refresh Command. US Patent QuarantiningAggressorRowsatRuntime.InMICRO,2022.
App.14/068,677,February272014. [118] ShuheiEnomotoetal.EfficientProtectionMechanismforCPUCache
[93] ZelalemBirhanuAwekeetal. ANVIL:Software-BasedProtection FlushInstructionBasedAttacks.IEICETransactionsonInformation
| AgainstNext-GenerationRowhammerAttacks.InASPLOS,2016. |     |     | andSystems,2022. |     |     |
| ----------------------------------------------------- | --- | --- | ---------------- | --- | --- |
[94] KuljitBainsetal.RowHammerRefreshCommand,2015.U.S.Patent [119] EvgenyManzhosovetal. RevisitingResidueCodesforModern
| 9,117,544.                             |     |     | Memories.InMICRO,2022. |                    |     |
| -------------------------------------- | --- | --- | ---------------------- | ------------------ | --- |
| 1594    33rd USENIX Security Symposium |     |     |                        | USENIX Association |     |

[120] Samira Ajorpazet al. EVAX: Towards a Practical,Pro-active & [144] Yoongu Kim et al. ATLAS: A Scalable and High-Performance
AdaptiveArchitectureforHighPerformance&Security.InMICRO, SchedulingAlgorithmforMultipleMemoryControllers.InHPCA,
| 2022. |     |     |     |     | 2010. |     |     |     |
| ----- | --- | --- | --- | --- | ----- | --- | --- | --- |
[121] AmirNaseredinietal. ALARM:ActiveLeArningofRowhammer [145] YoonguKimetal.ThreadClusterMemoryScheduling:Exploiting
Mitigations. https://users.sussex.ac.uk/~mfb21/rh-draft. DifferencesinMemoryAccessBehavior.InMICRO,2010.
pdf,2022.
[146] EimanEbrahimietal.ParallelApplicationMemoryScheduling.In
| [122] BireshKumarJoardaretal.MachineLearning-BasedRowhammer |     |     |     |     | MICRO,2011. |     |     |     |
| ----------------------------------------------------------- | --- | --- | --- | --- | ----------- | --- | --- | --- |
Mitigation.TCAD,2022.
[147] JamieLiuetal.RAIDR:Retention-AwareIntelligentDRAMRefresh.
| [123] HasanHassanetal.ACaseforSelf-ManagingDRAMChips:Improv- |     |     |     |     | InISCA,2012. |     |     |     |
| ------------------------------------------------------------ | --- | --- | --- | --- | ------------ | --- | --- | --- |
ingPerformance,Efficiency,Reliability,andSecurityviaAutonomous [148] DonghyukLeeetal. Tiered-LatencyDRAM:ALowLatencyand
in-DRAMMaintenanceOperations.arXiv:2207.13358[cs.AR],2022.
LowCostDRAMArchitecture.InHPCA,2013.
| [124] ZhenkaiZhangetal. | LeveragingEMSide-ChannelInformationto |     |     |     |     |     |     |     |
| ----------------------- | ------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
[149] OnurMutlu.MemoryScaling:ASystemsArchitecturePerspective.
DetectRowhammerAttacks.InSP,2020.
InIMW,2013.
[125] KevinLoughlinetal.Stop!HammerTime:RethinkingOurApproach
toRowhammerMitigations.InHotOS,2021. [150] IBM. IBM PowerS1014,S1022s,S1022,and S1024 Technical
|     |     |     |     |     | Overview | and Introduction. | https://www.redbooks.ibm.com/ |     |
| --- | --- | --- | --- | --- | -------- | ----------------- | ----------------------------- | --- |
[126] FabriceDevauxetal. MethodandCircuitforProtectingaDRAM abstracts/redp5675.html,2022.
| MemoryDevicefromtheRowHammerEffect,2021. |     |     |     | U.S.Patent |           |              |                    |           |
| ---------------------------------------- | --- | --- | --- | ---------- | --------- | ------------ | ------------------ | --------- |
|                                          |     |     |     |            | [151] AMD | Inc. 4TH GEN | AMD EPYC PROCESSOR | ARCHITEC- |
10,885,966.
|     |     |     |     |     | TURE. | https://www.amd.com/system/files/documents/ |     |     |
| --- | --- | --- | --- | --- | ----- | ------------------------------------------- | --- | --- |
[127] AliFakhrzadehganetal.SafeGuard:ReducingtheSecurityRiskfrom 4th-gen-epyc-processor-architecture-white-paper.pdf,
Row-HammerviaLow-CostIntegrityProtection.InHPCA,2022.
2023.
[128] StefanSaroiuetal.ThePriceofSecrecy:HowHidingInternalDRAM [152] IntelInc.Intel®Xeon®W-3400&Intel®Xeon®W-2400Processors
TopologiesHurtsRowhammerDefenses.InIRPS,2022. andtheIntel®W790ChipsetWorkstationPlatformBrief,2023.
[129] KevinLoughlinetal.MOESI-Prime:PreventingCoherence-Induced [153] YoonguKimetal.Ramulator:AFastandExtensibleDRAMSimula-
| HammeringinCommodityWorkloads.InISCA,2022. |     |     |     |     | tor.CAL,2016. |     |     |     |
| ------------------------------------------ | --- | --- | --- | --- | ------------- | --- | --- | --- |
[130] JinHanetal. SurroundGateTransistorWithEpitaxiallyGrownSi SAFARIResearchGroup.Ramulator—GitHubRepository.https:
[154]
PillarandSimulationStudyonSoftErrorandRowhammerTolerance //github.com/CMU-SAFARI/ramulator,2021.
forDRAM.TED,2021.
[155] HaocongLuoetal.Ramulator2.0:AModern,Modular,andExtensi-
[131] Jeonghyun Woo et al. Scalable and Secure Row-Swap: Ef- bleDRAMSimulator.arXiv:2308.11030[cs.AR],2023.
| ficient | and Safe Row | Hammer Mitigation | in Memory | Systems. |                            |     |                                |     |
| ------- | ------------ | ----------------- | --------- | -------- | -------------------------- | --- | ------------------------------ | --- |
|         |              |                   |           |          | [156] SAFARIResearchGroup. |     | Ramulator2.0—GitHubRepository. |     |
arXiv:2212.12613,2022.
https://github.com/CMU-SAFARI/ramulator2,2023.
| [132] CarstenBocketal. | RIP-RH:PreventingRowhammer-BasedInter- |     |     |     |     |     |     |     |
| ---------------------- | -------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
[157] RajeevBalasubramonianetal.CACTI7:NewToolsforInterconnect
ProcessAttacks.InASIACCS,2019.
ExplorationinInnovativeOff-ChipMemories.ACMTACO,2017.
[133] YichengWangetal.Discreet-PARA:RowhammerDefensewithLow
[158] JEDEC.JESD79-4C:DDR4SDRAMStandard,2020.
CostandHighEfficiency.InICCD,2021.
[134] TanjBennettetal.Panopticon:ACompleteIn-DRAMRowhammer [159] AtaberkOlgunetal. ABACuS:All-BankActivationCountersfor
Mitigation.InWorkshoponDRAMSecurity(DRAMSec),2021. ScalableandLowOverheadRowHammerMitigation.arXiv,2023.
[160] JEDEC.JESD79-3:DDR3SDRAMStandard,2012.
[135] WoongraeKimetal.A1.1V16GbDDR5DRAMwithProbabilistic-
AggressorTracking,Refresh-ManagementFunctionality,Per-Row [161] MicronInc. SDRAM,4Gb:x4,x8,x16DDR4SDRAMFeatures,
| HammerTracking,aMulti-StepPrecharge,andCore-BiasModulation |     |     |     |     | 2014. |     |     |     |
| ---------------------------------------------------------- | --- | --- | --- | --- | ----- | --- | --- | --- |
forSecurityandReliabilityEnhancement.InISSCC,2023. [162] JEDEC. JESD209-4B:LowPowerDoubleDataRate4(LPDDR4)
[136] YoonguKimetal.ACaseforExploitingSubarray-LevelParallelism Standard,2017.
(SALP)inDRAM.InISCA,2012.
[163] JEDEC.JESD79-5:DDR5SDRAMStandard,2020.
[137] KevinKChangetal.ImprovingDRAMPerformancebyParallelizing
[164] JEDEC.JESD209-5A:LPDDR5SDRAMStandard,2020.
RefresheswithAccesses.InHPCA,2014.
[165] JEDEC.JESD235C:HighBandwidthMemory(HBM)DRAM,2020.
| [138] OnurMutlu | and Thomas | Moscibroda. | Parallelism-Aware | Batch |     |     |     |     |
| --------------- | ---------- | ----------- | ----------------- | ----- | --- | --- | --- | --- |
Scheduling:EnhancingBothPerformanceandFairnessofShared [166] JEDEC. JESD79F:DoubleDataRate(DDR)SDRAMStandard,
| DRAMSystems.InISCA,2008. |     |     |     |     | 2008. |     |     |     |
| ------------------------ | --- | --- | --- | --- | ----- | --- | --- | --- |
[139] LavanyaSubramanianetal.BLISS:BalancingPerformance,Fairness [167] ZhaoZhangetal.APermutation-BasedPageInterleavingSchemeto
andComplexityinMemoryAccessScheduling.TPDS,2016. ReduceRow-BufferConflictsandExploitDataLocality.InMICRO,
| [140] LavanyaSubramanianetal. |     | TheBlacklistingMemoryScheduler: |     |     | 2000. |     |     |     |
| ----------------------------- | --- | ------------------------------- | --- | --- | ----- | --- | --- | --- |
AchievingHighPerformanceandFairnessatLowCost. InICCD, [168] JaeYoungHuretal. AdaptiveLinearAddressMapforBankInter-
| 2014. |     |     |     |     | leavinginDRAMs.IEEEAccess,2019. |     |     |     |
| ----- | --- | --- | --- | --- | ------------------------------- | --- | --- | --- |
[141] ChangJooLeeetal.ImprovingMemoryBank-LevelParallelismin [169] DimitrisKaseridisetal.MinimalistOpen-Page:ADRAMPage-Mode
thePresenceofPrefetching.InMICRO,2009. SchedulingPolicyfortheMany-CoreEra.InMICRO,2011.
[142] EimanEbrahimietal.FairnessviaSourceThrottling:AConfigurable [170] Y.Liuetal.GetOutoftheValley:Power-EfficientAddressMapping
andHighPerformanceFairnessSubstrateforMultiCoreMemory forGPUs.InISCA,2018.
Systems.InASPLOS,2010.
|     |     |     |     |     | [171] Mohsen | Ghasempouret | al. DReAM: DynamicRe-Arrangement |     |
| --- | --- | --- | --- | --- | ------------ | ------------ | -------------------------------- | --- |
[143] EnginIpeketal.Self-OptimizingMemoryControllers:AReinforce- ofAddressMappingtoImprovethePerformanceofDRAMs. In
| mentLearningApproach.InISCA,2008. |     |     |     |     | MEMSYS,2016. |                                        |     |     |
| --------------------------------- | --- | --- | --- | --- | ------------ | -------------------------------------- | --- | --- |
| USENIX Association                |     |     |     |     |              | 33rd USENIX Security Symposium    1595 |     |     |

| [172] JayadevMisraetal.FindingRepeatedElements.ScienceofComputer |     |     |     |     |     |     | Appendix |     |     |     |     |     |
| ---------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- |
Programming,1982.
[173] SeyedMohammadSeyedzadehetal.Counter-BasedTreeStructure A ABACuSSecurityAnalysis
forRowHammeringMitigationinDRAM.CAL,2017.
WeexplainhowABACuSmaintainsthemaximumactiva-
| [174] N.P.Jouppi. |     | ImprovingDirect-MappedCachePerformancebythe |     |     |     |     |     |     |     |     |     |     |
| ----------------- | --- | ------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
tioncountamongallsiblingrows(§4)inRACsbyshowing
AdditionofaSmallFully-AssociativeCacheandPrefetchBuffers.In
thatInvariant1holdsafteraRACupdateuponaDRAMrow
ISCA,1990.
activation.
[175] AlanJaySmith.CacheMemories.ACMCompututingSurveys,1982.
AnalysisOverview.Invariant1formallydefinestheprop-
| [176] MineshPateletal. |     | ACaseforTransparentReliabilityinDRAM |     |     |     |     |     |     |     |     |     |     |
| ---------------------- | --- | ------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Systems.arXiv:2204.10378[cs.AR],2022. ertyofthevaluestoredinarowactivationcounterintermsof
theactualactivationcountofsiblingDRAMrows(i.e.,each
| [177] M.Marazzietal. |     | REGA: | ScalableRowhammerMitigationwith |     |     |     |     |     |     |     |     |     |
| -------------------- | --- | ----- | ------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Refresh-GeneratingActivations.InIEEES&P,2023. siblingrow’strueactivationcount,regardlessoftheRAC’s
[178] WikiChip. CascadeLakeSP-Intel. https://en.wikichip.org/ value)inarefreshwindow(t ).Thespillovercounter’s
REFW
wiki/intel/cores/cascade_lake_sp.
|     |     |     |     |     |     |     | value is already | greaterthan |     | orequal | to a row’s | activation |
| --- | --- | --- | --- | --- | --- | --- | ---------------- | ----------- | --- | ------- | ---------- | ---------- |
[179] Synopsys, Inc. Synopsys Design Compiler. https: countforDRAMrowsthatarenottrackedbyanyABACuS
//www.synopsys.com/support/training/rtl-synthesis/
counter[102].8
design-compiler-rtl-synthesis.html.
| [180] KarthikChandrasekaretal. |     |     | DRAMPower: | Open-SourceDRAM |     |     |     |     |     |     |     |     |
| ------------------------------ | --- | --- | ---------- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
Invariant1
Power&EnergyEstimationTool.http://www.drampower.info/.
[181] ScottRixneretal.MemoryAccessScheduling.InISCA,2000.
LetACT_COUNT(bankID,rowID)denotetheactualactivation
[182] WilliamKZuravleffetal. ControllerforaSynchronousDRAM countofarowwithrowIDinabank.Ifarowistrackedbyan
ABACuScounter,therowactivationcounter(RAC)corresponding
| ThatMaximizes |     | ThroughputbyAllowing |     | MemoryRequests |     | and |     |     |     |     |     |     |
| ------------- | --- | -------------------- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
CommandstoBeIssuedOutofOrder,1997.U.S.Patent5,630,096. tothisrowisalwaysgreaterthanorequaltotheactualactivation
countoftherowwiththesamerowIDinanyofthebanks.Thatis,
| [183] OnurMutluetal. |     | Stall-TimeFairMemoryAccessSchedulingfor |     |     |     |     |     |     |     |     |     |     |
| -------------------- | --- | --------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
∀(b,r)∈(Banks,SiblingRows);RAC(r)≥ACT_COUNT(b,r)
ChipMultiprocessors.InMICRO,2007.
| [184] StandardPerformanceEvaluationCorp. |     |     |     | SPECCPU2006. |     | http: |     |     |     |     |     |     |
| ---------------------------------------- | --- | --- | --- | ------------ | --- | ----- | --- | --- | --- | --- | --- | --- |
//www.spec.org/cpu2006/,2006. Proof:Byinductionontheactualactivationcountofrowr
inbankb,trackedbyRAC(r).
| [185] StandardPerformanceEvaluationCorp. |     |     |     | SPECCPU2017. |     | http: |     |     |     |     |     |     |
| ---------------------------------------- | --- | --- | --- | ------------ | --- | ----- | --- | --- | --- | --- | --- | --- |
//www.spec.org/cpu2017,2017.
BaseCase:WhentheRACstartstrackingarowrinbankb
[186] Transaction Processing Performance Council. TPC Benchmarks. forthefirsttime,thefollowingholds:
http://tpc.org/.
| [187] Jason | E. Fritts | et al. MediaBenchII |     | Video: Expediting |     | the next |     |     |     |     |     |     |
| ----------- | --------- | ------------------- | --- | ----------------- | --- | -------- | --- | --- | --- | --- | --- | --- |
• RAC(r)≥ACT_COUNT(b,r)
| GenerationofVideoSystemsResearch. |     |     |     | Microprocess.Microsyst., |     |     |     |     |     |     |     |     |
| --------------------------------- | --- | --- | --- | ------------------------ | --- | --- | --- | --- | --- | --- | --- | --- |
2009.
|             |          |                  |     |              |         |      | • SAV(b,r)isset.OtherSAV |     |     | bitsarezero. |     |     |
| ----------- | -------- | ---------------- | --- | ------------ | ------- | ---- | ------------------------ | --- | --- | ------------ | --- | --- |
| [188] Brian | Cooperet | al. Benchmarking |     | CloudServing | Systems | with |                          |     |     |              |     |     |
YCSB.InSoCC,2010.
InductionHypothesis:Assumethatinvariantholdsforany
[189] GregHamerlyetal.SimPoint3.0:FasterandMoreFlexibleProgram rowr′inbankb′whicharetrackedbyanABACuScounter.
PhaseAnalysis.JournalofInstruction-LevelParallelism,2005. StepCase:Letr′beanarbitrarysiblingofrinbankb′.Note
[190] SAFARIResearchGroup.ABACuS—GitHubRepository.https:
|     |     |     |     |     |     |     | that RAC(r′)=RAC(r) |     | by definition |     | of RAC. Assume | that |
| --- | --- | --- | --- | --- | --- | --- | ------------------- | --- | ------------- | --- | -------------- | ---- |
//github.com/CMU-SAFARI/ABACuS,2023.
suchanr′isactivated.Wedistinguishbetweentwocases.
[191] AllanSnavelyetal.SymbioticJobSchedulingforASimultaneous Case1:SSSAAAVVV(((bbb′′′,,,rrr′′′)))isnotset.Inthissubcase,beforeacti-
MultithreadedProcessor.InASPLOS,2000.
vatingr′,wehaveRAC(r′)>ACT_COUNT(b′,r′)because
[192] StijnEyermanetal.System-LevelPerformanceMetricsforMultipro-
|                               |     |     |     |     |     |     | SAV(b′,r′)isnot | set.Therefore,afteractivatingr′ |     |     |     |        |
| ----------------------------- | --- | --- | --- | --- | --- | --- | --------------- | ------------------------------- | --- | --- | --- | ------ |
| gramWorkloads.IEEEMicro,2008. |     |     |     |     |     |     |                 |                                 |     |     |     | wehave |
[193] PierreMichaud.DemystifyingMulticoreThroughputMetrics.CAL, RAC(r′)≥ACT_COUNT(b′,r′)andSAV(b′,r′)isset.
| 2012. |     |     |     |     |     |     | SSSAAAVVV(((bbb′′′,,,rrr′′′))) |     |         |         |                |          |
| ----- | --- | --- | --- | --- | --- | --- | ------------------------------ | --- | ------- | ------- | -------------- | -------- |
|       |     |     |     |     |     |     | Case 2:                        |     | is set. | In this | subcase,before | activat- |
[194] R.Zhouetal.LT-PIM:AnLUT-BasedProcessing-in-DRAMArchi- ingr′,wehaveRAC(r′)≥ACT_COUNT(b′,r′).Hence,af-
tectureWithRowHammerSelf-Tracking.IEEECAL,2022. ter activating r′,the actual activation count of r′ increases
[195] StefanSaroiuetal.HowtoConfigureRow-Sampling-BasedRowham- RAC(r′) SAV(b′,r′)
|     |     |     |     |     |     |     | by one, and |     | is incremented. |     | The | re- |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | --------------- | --- | --- | --- |
merDefenses.DRAMSec,2022.
|     |     |     |     |     |     |     | mains set,while | otherSAV |     | bits are | reset. Thus,RAC(r′)≥ |     |
| --- | --- | --- | --- | --- | --- | --- | --------------- | -------- | --- | -------- | -------------------- | --- |
[196] H.Hassanetal.CROW:ALow-CostSubstrateforImprovingDRAM
ACT_COUNT(b′,r′)stillholds,satisfyingtheinvariant.
Performance,EnergyEfficiency,andReliability.InISCA,2019.
BasedontheproofofthecorrectnessofInvariant1,wecon-
[197] ZhiZhangetal.TeleHammer:AStealthyCross-BoundaryRowham-
cludethatABACuSaccuratelystoresthemaximumactivation
merTechnique.arXiv:1912.03076[cs.CR],2019.
[198] MoinuddinQureshi. RethinkingECCintheEraofRow-Hammer. countacrossallsiblingrows.
DRAMSec,2021.
| [199] Seungki | Hong | et al. DSAC: | Low-Cost | RowhammerMitigation |     |     |     |     |     |     |     |     |
| ------------- | ---- | ------------ | -------- | ------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
UsingIn-DRAMStochasticandApproximateCountingAlgorithm.
8Ourextendedversion[159]providesasimpleexplanationforwhythe
arXiv:2302.03591,2023.
|          |         |          |                           |     |     |       | spillovercounter’svalueisgreaterthan |     |     | orequaltoa | non-trackedrow’s |     |
| -------- | ------- | -------- | ------------------------- | --- | --- | ----- | ------------------------------------ | --- | --- | ---------- | ---------------- | --- |
| [200] H. | Gomezet | al. DRAM | Row-HammerAttackReduction |     |     | Using | activationcount.                     |     |     |            |                  |     |
DummyCells.InNORCAS,2016.
| 1596    33rd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |
