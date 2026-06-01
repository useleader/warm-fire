---
publish: true
---

| Mayhem: | Targeted                      |              | Corruption |     |     | of Register | and                           | Stack | Variables |     |
| ------- | ----------------------------- | ------------ | ---------- | --- | --- | ----------- | ----------------------------- | ----- | --------- | --- |
|         | Andrew                        | J. Adiletta∗ |            |     |     |             | M. CanerTol∗                  |       |           |     |
|         | ajadiletta@wpi.edu            |              |            |     |     |             | mtol@wpi.edu                  |       |           |     |
|         | WorcesterPolytechnicInstitute |              |            |     |     |             | WorcesterPolytechnicInstitute |       |           |     |
|         | Worcester,MA,USA              |              |            |     |     |             | Worcester,MA,USA              |       |           |     |
|         | Yarkın                        | Doröz        |            |     |     |             | BerkSunar                     |       |           |     |
|         | ydoroz@wpi.edu                |              |            |     |     |             | sunar@wpi.edu                 |       |           |     |
4202 rpA 21  ]RC.sc[  2v54520.9032:viXra
|          | WorcesterPolytechnicInstitute |     |     |     |     |                     | WorcesterPolytechnicInstitute |     |     |     |
| -------- | ----------------------------- | --- | --- | --- | --- | ------------------- | ----------------------------- | --- | --- | --- |
|          | Worcester,MA,USA              |     |     |     |     |                     | Worcester,MA,USA              |     |     |     |
| ABSTRACT |                               |     |     |     |     | ACMReferenceFormat: |                               |     |     |     |
AndrewJ.Adiletta,M.CanerTol,YarkınDoröz,andBerkSunar.2024.May-
| In the past | decade, many | vulnerabilities | were | discovered | in mi- |     |     |     |     |     |
| ----------- | ------------ | --------------- | ---- | ---------- | ------ | --- | --- | --- | --- | --- |
hem:TargetedCorruptionofRegisterandStackVariables.InProceedings
| croarchitectures | whichyielded | attackvectorsand |     | motivatedthe |     |     |     |     |     |     |
| ---------------- | ------------ | ---------------- | --- | ------------ | --- | --- | --- | --- | --- | --- |
of 19thACMASIAConferenceonComputerandCommunicationsSecurity
studyofcountermeasures.Further,architecturalandphysicalim-
(ASIACCS2024).ACM,NewYork,NY,USA,16pages.https://doi.org/X
perfectionsinDRAMsledtothediscoveryofRowhammerattacks
whichgiveanadversarypowertointroducebitflipsinavictim’s
memoryspace.NumerousstudiesanalyzedRowhammerandpro- 1 INTRODUCTION
posedtechniquestopreventitaltogetherortomitigateitseffects.
TheemergenceofattackssuchasMeltdown[28]andSpectre[24]
Inthiswork,wepushtheboundaryandshowhowRowhammer
can be further exploited to inject faults into stack variables and exposedintrinsic vulnerabilitiesinourcomputinginfrastructure.
evenregistervaluesinavictim’sprocess.Weachievethisbytar- Thesemicroarchitecturalleakageswerefurtherdevelopedandex-
ploitedinanumberofstudies,e.g.,[5,13,21,44,46].
getingtheregistervaluethatisstoredintheprocess’sstack,which
subsequently is flushed out into the memory, where it becomes RowhammerFaultInjectionWhilethesevulnerabilitiesfocused
vulnerabletoRowhammer.Whenthefaultyvalueisrestoredinto onpassive leakages, Rowhammer emerged as a realistic toolfor
theregister,itwillendupusedinsubsequentiterations.Theregis-
anattackertoactivelyinjectfaultsinDRAMs[23,37].Rowham-
tervaluecanbestoredinthestackvialatentfunctioncallsinthe
|     |     |     |     |     |     | mer exploits | the fact that | if a row in | DRAM | is accessed repeat- |
| --- | --- | --- | --- | --- | --- | ------------ | ------------- | ----------- | ---- | ------------------- |
sourceorbyactivelytriggeringsignalhandlers.Wedemonstrate edly,thismayleadtobitflipsinneighboringrows.Rowhammer
the power of the findings by applying the techniques to bypass hasproveneffectiveinreal-lifeattackscenarios.Forinstance,[15]
SUDO and SSH authentication. We further outline how MySQL showed that it is possible to gain root access with opcode flip-
andothercryptographiclibrariescanbetargetedwiththenewat- pinginthesudobinary,[36]demonstratedanend-to-end
attack
tackvector.Thereareanumberofchallengesthisworkovercomes
|     |     |     |     |     |     | breaking OpenSSH | public-key | authentication, |     | [48] demonstrated |
| --- | --- | --- | --- | --- | --- | ---------------- | ---------- | --------------- | --- | ----------------- |
withextensiveexperimentationbeforecomingtogethertoyieldan a Bellcore attack on a CRT-based RSA implementation in Wolf-
end-to-endattackonanOpenSSLdigitalsignature:achievingco- SSLtorecover secret keys. Furtherpushingtheboundaries,[16]
location with stack and register variables, with synchronization and [11] have shown that Rowhammer can be applied even re-
providedviaablockingwindow.Weshowthatstackandregisters
motelythroughJavaScript.Similarly,[41]and[29]demonstrated
arenolongersafefromtheRowhammerattack.
thatRowhammercanbeexecutedoverthenetwork.Rowhammer
isalsoapplicableincloudenvironments[7,49]andheterogeneous
CCSCONCEPTS
|     |     |     |     |     |     | FPGA-CPU platforms | [48]. | Beyond DRAMs, |     | [4] has shown that |
| --- | --- | --- | --- | --- | --- | ------------------ | ----- | ------------- | --- | ------------------ |
•StackRowhammerAttack→UsingRowhammertoinject flashmemoriesarealsopronetoRowhammer-likecell-to-cellin-
faultsintostackandregistervariables. terference,whichthen,whenusedtotargetfile-systempages,can
resultinprivilegeescalation[26].
KEYWORDS
RowhammerCountermeasures.Theseverityofthethreatmoti-
Rowhammer,Stack,RegisterFlipping
vatednumerousRowhammercountermeasures,e.g.,fordetection
∗Bothauthorscontributedequallytothisresearch. [1,6,9,17,19,20,34,52]andmitigation[3,16,45].Unfortunately,
|     |     |     |     |     |     | [15]hasshownthat | allofthesecountermeasures |     |     | areineffective. |
| --- | --- | --- | --- | --- | --- | ---------------- | ------------------------- | --- | --- | --------------- |
Permissiontomakedigitalorhardcopiesofallorpartofthisworkforpersonalor
|     |     |     |     |     |     | Further, [8] | showed that | ECC, a hardware-enabled |     | error check- |
| --- | --- | --- | --- | --- | --- | ------------ | ----------- | ----------------------- | --- | ------------ |
classroomuseisgrantedwithoutfeeprovidedthatcopiesarenotmadeordistributed
forprofitorcommercialadvantageandthatcopiesbearthisnoticeandthefullcita- ing built into many memory devices, can also be bypassed. An-
tiononthefirstpage.Copyrightsforcomponentsofthisworkownedbyothersthan other hardware countermeasure Target Row Refresh (TRR), has
ACMmustbehonored.Abstractingwithcreditispermitted.Tocopyotherwise,orre- alsobeenrecentlydefeated[12].Thisworkwasextendedin[11],
publish,topostonserversortoredistributetolists,requirespriorspecificpermission
and/orafee.Requestpermissionsfrompermissions@acm.org. claimingthatmorethan80%oftheDRAMchipsinthemarketare
ASIACCS2024,July01–05,2024,Singapore
stillvulnerabletoRowhammer.Quiterecently,hammeringbeyond
©2024AssociationforComputingMachinery.
adjacentlocations,i.e.,HalfDouble[25]hammering,wasshownto
ACMISBN978-1-4503-XXXX-X/18/06...$15.00
| https://doi.org/X |     |     |     |     |     | beeffectiveincircumventingTRRmitigations. |     |     |     |     |
| ----------------- | --- | --- | --- | --- | --- | ----------------------------------------- | --- | --- | --- | --- |

ASIACCS2024,July01–05,2024,Singapore Adilettaetal.
FaultingCPUInternals.Withsignificanteffortsputintoadvanc- OutlineofthePaper
ing Rowhammer attacksand countermeasures, oneconstant has Therestofthepaperisorganizedasfollows.Section2givesback-
beentheassumptionthatCPUinternalsisimpervioustosoftware- ground on ourattack. In Section 3, we explain the threat model
based fault injection attacks. Specifically, SRAM-based registers ofourattack.InSection4,weexplaintheofflineandonlinestages
andcachesareassumedtobefreefromfaultinjection(exceptvia oftheattackwhichincludegettingphysicallycontinuousmemory
direct physical manipulations such as in laser fault injection at- andprofilingbaitpages.InSection5,weexplaininjectingfaultsin
tacks).Ontheotherhand,CPU-externaldevicessuchasDRAMs stackmemoryandexplainhowtoflipCPUregistervaluesusing
aregreatlyvulnerabletophysicaltampering.Thisviewhasbeen Rowhammer.InSection6,wegivetheexperimentalevaluation.In
aroundsincetheearlytimesofTrustedComputingandwasmoti- Section7,weexplainourfindingsandresultsonOpenSSL, sudo,
vatedfurtherbytheintroductionofcold-bootattacks[18]. andOpenSSH.InSection8,wegiveouranalysisonRSABellcore
Inthiswork,wedemonstratethatCPUinternalssuchasregis- AttacksonOpenSSLandMySQL.Insection9wedemonstrateafull
tervaluesarealsovulnerable.Untilnow,Rowhammerattackswere end-to-endattackonanOpenSSLclient/serversignatureverifica-
generallytargetedatcorruptingdynamicallyallocatedmemory[31] tion. In Section 10, we proposeseveral countermeasures against
orbinariesstoredondiskloadedintomemory[37].Otherworks ourattack.
[51]havementionedthattherearekeyregisterslikeEIPandESP
that if corrupted can affect the control flow of an x86 program,
2 BACKGROUND
butcannotbecorruptediftheregistervaluesarestoredinapro-
cessorcore.Hereweshowthatregistervaluescanbeforcedbyan 2.1 RowhammerAttacks
attackertobesavedtothestackandflushedouttomemory,where
With increasing DRAM densities the chance for bit flips and re-
theybecomevulnerabletoRowhammerfaultinjection.Uponreload,
liabilityfailuresisincreasing.Hence,toretaindataeveryDRAM
thefaultyvaluesarereloadedinto theregistersbeforeresuming
rowhastobecontinuouslyrefreshedusuallywith64msecinter-
execution.
vals.Althoughrefreshing therowsperiodicallyhelpspreventing
TargetingtheStack.Besidesflushedregistervalues,vulnerable thedatacorruption,Kimet al.[23] showed that frequent access
pieces of code exist within the stack of programs, e.g. security to the neighbor rows causes faster charge leakage, which effec-
checksandauthenticationstates.Whenthesesensitive variables tivelycausesbitflipsbeforethenextrefresh.Thisisknownasthe
are corrupted, this may result in privilege escalation. Crypto li- Rowhammereffect[23].Seabornetal.[37]introducedthedouble-
braries,forinstance,minimizeoreliminatedynamicmemoryand sidedRowhammerflippingthevictimcellsevenfaster.
stackuseeithertosupportexecutioninconstrainedenvironments, Grussetal.[15]introducedone-locationhammeringandachieved
orforsafety-criticalsystemssuchasembeddedorRTOSsystems rootaccesswithopcodeflippinginsudobinaryin2018.Grusset
ortominimizeexposureofpotentiallyvulnerableinternalsecrets. al. [16] and Ridder et al. [11] have shown that Rowhammer can
ThewolfSSLlibrary,forinstance,supportscompilationoptionsto beappliedthroughJavaScriptremotely.Tataretal.[41]andLipp
avoiddynamicmemoryuse.TheNaCLlibrary,incontrast,avoids etal.[29]haveproved thatitcanbeexecutedover thenetwork.
dynamicmemoryandvariable-sizestackallocationaltogether.Crypto Rowhammerisalsoapplicableincloudenvironments[7,49]and
libraryimplementations,therefore,heavilyrelyonregisters,and heterogeneous FPGA-CPU platforms [48]. In 2020, Kwong et al.
stackvariables.Hereweshowthatsuchvariablesarenotsecure [27]demonstratedthatRowhammerisnotjustanintegrityprob-
againstfaultinjection.HencetheattacksurfaceofRowhammeris lembutalsoaconfidentialityproblem.
greaterthanpreviouslyassumed. TherehavebeenmanyeffortsonRowhammerdetections[1,6,
9,17,19,20,34,52]andneutralization[3,16,45].Grussetal.[15]
OurContribution haveshownthatallofthesecountermeasuresareineffective.Co-
Inthispaper,wesystematicallyanalyzethethreatimposedbyRowham- jocar et al. [8] in 2019 showed that the ECC countermeasure is
merfaultinjectionstostackvariablesandregistervaluesthatwere notsecureeither.AnotherhardwarecountermeasureTargetRow
previouslyconsideredsecureagainstRowhammer.Specifically,we Refresh(TRR)hasalsobeenrecentlybypassedbyFrigoetal.[12].
• Introduceanovelattacktoinjectfaultsintoregistervaluesthrough ThisworkwasextendedbyRidderetal.[11]toattackTRR-enabled
thestackmemory; DDR4chipsfromJavaScriptandclaimthatmorethan80%ofthe
• Showhowstaticcode/dataallocationcanbemanipulatedwith DRAMchipsinthemarketarestillvulnerabletoRowhammer.Quite
baitpagestoachieveco-locationwiththevictim’sstack; recently,hammeringbeyondadjacentlocationswasshown[25]to
• Introduce new synchronization techniques to enable practical beeffectiveincircumventingTRRmitigations.
meanstotargetstackandregisterviaRowhammer;
• DemonstrateattacksonSUDO,OpenSSH,andMySQL; 2.2 Countermeasures inCryptoLibraries
• HighlightnewRSABellcorevulnerabilitiesenabledbytheattack
Physicalfaultinjectionattacksarewellknownamongcryptoprac-
vectorsdiscoveredinOpenSSL;
titioners[2].Cryptolibraries,especiallyonesdesignedforembed-
• DemonstrateafullattackoncodeusingOpenSSLforsignature
dedplatforms,haveimplementedcountermeasuressincetheearly
verificationwithattacksonbothstackandregistervariables1
2000s.Forinstance,OpenSSLimplementserrorchecksinCRT-based
• Outlinemitigativecodingstylestominimizetheattacksurface
exponentiationtothwartBellcoreattacks[2].Still,faultinjection
againstthenewlyintroducedattackvectors.
hasproveneffectivein[39]tocorruptEllipticCurveParameters
1Thesourcecodeisavailableonhttps://github.com/vernamlab/mayhem. intheOpenSSLlibrary.Further,[30]demonstratedaRowhammer

Mayhem:TargetedCorruptionofRegisterandStackVariables ASIACCS2024,July01–05,2024,Singapore
fault injection vulnerability in WolfSSL that resulted in ECDSA complete and thus would appear as peaks on the graph. For ev-
key disclosure. The fault was injected during the signing opera- erysystem,thethresholdvaluesofSPOILERneedtobeadjusted.
tionwithprivateECCkeys,whichoccurduringaTLShandshake Thesethresholdvaluesincludethetimingrequiredtocallamem-
betweenclientandserver.WolfSSLaddressedthisvulnerabilityby oryreadanoutlierinthedataset(atimingmeasurementabovea
implementingaseriesofchecksduringeachstageofthesigning certainvalueisprobablytheresultofasysteminterruptorsome
processtodetectifdatahasbeentamperedwith,andWOLFSSL_- othereventratherthanphysicalcontinuity),andatimingthresh-
CHECK_SIG_FAULTSwasreleasedasasecuritymeasure[33].Impor- oldvaluetoqualifyavalueasapeakandthuspartofthecontinu-
tantly,thesechecksthatprotectdynamicmemoryoperateonthe ousmemorybuffer.
ideathatvariablesinthestackaresafefromRowhammer,which Inourexperiments,wegenerallylookedforabout3-5%ofour
thispaperwilldemonstrateisnotthecase. memoryallocatedto SPOILER tobephysically continuous.This
meansthatifweallocated1024MBytesofmemorytoourbuffer,
3 THREATMODELFORMEMORYMAYHEM wewouldexpecttofindaround32-64bytesofcontinuousmem-
ory.Thisvariesdependingontheexperimentandthemachinethe
Wewillexplaintheattackscenariosindetailforeachattacktarget
experimentisrunningon.
inSection7.InlinewiththepreviousRowhammerattacks[7,15,
16,23,49],weassumeattacker-victimco-locationinthesamesys- FindingRowsintheSameBank.Inadditiontofindingmemory
tem.Co-locationisacommonassumptionformanymicro-architectural thatisphysicallycontinuous,thememorymustalsobeinthesame
side-channel attacks[5,24,28,44,46].Weassumetheoperating bankoftheDRAMforRowhammertowork.Weusetherowcon-
systemworksasintendedwithoutanycompromiseinitsintegrity flictsidechanneltoleakDRAMinformationwhich,likeSPOILER,
andtheattackerhasuserprivilegesthroughoutthepaper.Weas- doesnotrequirerootaccess.Rowconflictreadsfromthefirstad-
sumetheattackerdoesnothaveaccesstoanyservicethatreveals dress in the physically continuous memory buffer, then it reads
thephysicaladdressorDRAMaddressinginformation.Ourattack fromaddress𝑛 (where𝑛 is1 through thelength of thememory
doesnotrequirehuge-pageconfigurationandworkswithstandard- buffer)andcalculatesthetimedifferencebetweenreads.Alarger
sizepages. time difference indicates that the row buffer within the DRAM
bankneededtobeclearedandthuscausedaspikeintiming.Just
likeSPOILER,rowconflictneedsthresholdvaluestobeexperimen-
4 FLIPPINGBITSINTHESTACKAND
tallydeterminedanddefinedforeachmachine.
REGISTERVARIABLES
FortheRowhammerattackonDDR4memory,weperformamulti- 4.2 ProfilingforBaitPages
sidedattacktocircumventTRRprotection.Wefoundthatamulti-
Inordertoflipavariableinthestackofaprogram,thepagethat
sidedattackwith11rowswasmosteffectiveatgettingflipsonour thevariableislocatedinneedstobeplacedinapagethathasa
systemandweusedmfencetopreventout-of-orderexecution.It
flippablebitatthecorrectpageoffset.Thereareanumberofpages
ispossiblethatwithoutmfenceCPUoptimizationswoulddisrupt
used by the victim process that are irrelevant to our attack and
thecriticalorderthatrowsareaccessedforthemultisidedattack wouldfillourflippypagebeforethepagewithourtargetvariable.
whichwouldprevent theattackfromworking.Wefound1Mac- Thus,wemustreleaseunusedfillerpageswecallbaitpages,which
cesses ofalltherowswere optimalingettingflipsand reducing
arefilledwiththevictim’sdatathatisirrelevanttotheattackfirst.
profiling/online time. Wealso foundthatdoing 100iterationsof To flip a variable that is stored in a register and pushed to and
1Maccessesalongwith100Knopsinbetweenalsoimprovedthe
poppedfromstack,asimilarprocessisapplied,butmorecomplex
efficacyoftheattackingettingflips. profilingisnecessarybymanuallylookingatthememoryspacein
theLinuxkernelfortheregistervalue.
4.1 OfflineMemoryProfiling
BaitPageProfilingForStackAttacks.Thenumberofbaitpages
Rowhammer requires that rows in DRAM are adjacent to each thatneedtobereleaseddependsontheprocessandifASLRisen-
otherphysically.WeachievethisthroughtheuseoftheSPOILER abled.Thepseudo-codeforreleasingbaitpagesisgiveninListing1.
and RowConflictattacks.WeuseSPOILER [21]becauseitleaks Wecanseethattheprofilingprocessfirstallocatespagesintoits
virtualtophysicaladdresstranslationwithouttheneedtoreadthe ownprocessspacetobereleasedasbaitforthevictimprocess.It
pagemapfile,whichwouldrequirerootaccess.SPOILERtakesad- thenunmapstheflippypage(thishappensintheonlinestage)and
vantageofamicroarchitectureoptimizationsspeculativeloadsand unmapsthebaitpagessotheygetfilledfirst.
storeforwarding.Forfindingaddressesthatarewithinthesame During the offline stage, we determine the proper number of
bank,weuserowconflicts[35],whichisanothertimingsidechan- baitpagestoreleasebyfirstreleasing alargenumberbaitpages
nelthatweexploittocolocatememoryforRowhammer. (500or more) and recording all the physical addresses of there-
leasedpagesintoatextfile.Then,welaunchthevictimprocessand
Profiling for Contiguous Memory. SPOILER first allocates a translatethevirtualaddressofthetargetvariableintoaphysical
large buffer in the memory of the attacker program. The mem- address,whichwethensearchedforinthetextfilewithreleased
oryfromthisbufferisdistributedthroughouttheDRAMrandomly. addresses.Theindexofthephysicaladdressinthetextfiledeter-
Withinawindowofthememorybuffer,SPOILERwriteszerosto minedthenumberofbaitpagesweneededtorelease. Although
alltheaddresses,thentimeshowlongittakestoloadthefirstentry thereiscertainvariabilityinhowmanyofthebaitpagesarecon-
inthearray.Physicalmemorydependencyrequiresmorecyclesto sumedbythevictimprocessbeforeitallocatesthetargetvariable

| ASIACCS2024,July01–05,2024,Singapore |     |     |     |     |     |     |     | Adilettaetal. |
| ------------------------------------ | --- | --- | --- | --- | --- | --- | --- | ------------- |
to a page, experimentally we found the victim process will con- Normal Runtime
AttackWindow
sumethesamenumberofbaitpages30%ofthetimeasstatedin
section6.3.
|     |     |     |     |     |     | Victim Start | Victim End |     |
| --- | --- | --- | --- | --- | --- | ------------ | ---------- | --- |
BaitPageProfilingForRegisterAttacks.Registersalsofallvic-
timtothesamebaitpagesattack,butprofilingismoredifficultbe-
Blocking
causetheydonothaveavirtualaddressthatcanbetranslatedinto
Blocking
aphysicaladdresswhichcanbefoundinthebaitpagesreleased. Window
|                                   |     |                          |     |     |     | Victim Start |     | Victim End |
| --------------------------------- | --- | ------------------------ | --- | --- | --- | ------------ | --- | ---------- |
| Instead, duringtheprofilingstage, |     | weeditthevictimprocessto |     |     |     |              |     |            |
givetheregisterauniquevalue(like0xDEADBEEF),thenlookinto
Figure1:Diagramshowingtheruntimeofaprogramwitha
theprocessesmemorywith\proc\PID\memandlookoftheunique
blockingwindowallowingtheattackertoattackattheright
value.Thisistheeffectivevirtualaddressoftheregisterwhenit
time
| gets pushed | to stack, and | will get put back | into the stack | when |     |     |     |     |
| ----------- | ------------- | ----------------- | -------------- | ---- | --- | --- | --- | --- |
itspoppedoff.Wecanusethesamemethodofconvertingthevir-
tualaddresstoaphysicaladdressusingthePIDandthepagemap AttackingProcessesDuringaBlockingWindow.Themostop-
filefortheprocess.Importantly,editingthesourcecodetoadda timalscenarioforanattackisforvulnerablecodetohaveablock-
unique value to the register is only necessary during the offline ingwindowwheretheprocessiswaitingforaneventthatmaybe
stage.Duringtheonlinestage,thesourcecodeforthevictimre- triggeredbytheattacker.ForSUDO,thiscouldbetheperiodwhere
mainsuntouched,andthenumberofbaitpagesconsumedbefore theprocessiswaitingfortheattackertoenterapassword.Thepro-
theregistersarepushedtostackremainthesame. cesssavesstatedatatostackwhilewaitingfortheusertosubmit
apassword.High-levelexamplesofsynchronizingblockingcodes
arePasswordInput,IPSocketConnections,SignalInterrupts,Me-
Listing1:Pseudocodeshowinghowpagescanbeforcedinto diaUploads,OtherUserInput.
aspecificareainmemoryusingamapping-unmappingtech-
|     |     |     |     |     | Looking | at Figure 1, we | can see that there | is a blocking pe- |
| --- | --- | --- | --- | --- | ------- | --------------- | ------------------ | ----------------- |
nique
|     |     |     |     |     | riod during | the attack window. | This allows | for synchronization |
| --- | --- | --- | --- | --- | ----------- | ------------------ | ----------- | ------------------- |
betweenthevictimandattackersothereisnolongeraquestion
1 buffer = mmap(baitPages * PAGESIZE) ofprobabilityoftheattackerwilllaunchtheRowhammerattackat
| 2 munmap(flippyPageAddr, |     | PAGESIZE) |     |     |     |     |     |     |
| ------------------------ | --- | --------- | --- | --- | --- | --- | --- | --- |
therighttime.Practically,wecanseeanexampleofthisonSection
| 3 for(i                       | = 0; i < bait_pages; | i++)      |     |     |     |     |     |     |
| ----------------------------- | -------------------- | --------- | --- | --- | --- | --- | --- | --- |
| 4 munmap(&buffer[i*PAGESIZE], |                      | PAGESIZE) |     |     |     |     |     |     |
9.1ofsyncronizationonareal-worldTLShandshake.
|     |     |     |     |     | 4.4 MultisidedRowhammer |     |     |     |
| --- | --- | --- | --- | --- | ----------------------- | --- | --- | --- |
Wereleasethebaitpagesbeforetheflippypage,asseeninList-
TherehavebeenmanyeffortstodetectRowhammerbytracking
| ing 1. Thisis | becausetheLinux | BuddyAllocatoralgorithmthat |     |     |     |     |     |     |
| ------------- | --------------- | --------------------------- | --- | --- | --- | --- | --- | --- |
consecutivereadstoadjacentrowsinaSerialPresenceDetect(SPD)
| isusedtoallocatememorytodifferent |     | processeseffectivelyacts |     |     |     |     |     |     |
| --------------------------------- | --- | ------------------------ | --- | --- | --- | --- | --- | --- |
chipIntelCPUsdeployamitigationknownaspseudo-TRRorpTRR,
| like a last-out-first-insystem, |     | where thelatest | pages released | to  |     |     |     |     |
| ------------------------------- | --- | --------------- | -------------- | --- | --- | --- | --- | --- |
whichreadstheMaximumActivationCount,orMACvaluefrom
memoryareusedfirst.
theSPDandifreadstoconsecutiverowsreachestheMACvalue,
| 4.3 OnlineAttackPhase |     |     |     |     | theIntelCPUrefreshestherow. |     |     |     |
| --------------------- | --- | --- | --- | --- | --------------------------- | --- | --- | --- |
AmultisidedattackworksevenwithTRRenabled,withastrat-
ReleasingTheFlippyPageandBaitPages.Thefirststageof egybasedontheTrrespassmultisidedattack[12].
theattackisreleasingtheflippypagefoundduringtheofflinepro-
|     |     |     |     |     | Flipping | Bits using Rowhammer. | The final | step in the work- |
| --- | --- | --- | --- | --- | -------- | --------------------- | --------- | ----------------- |
filingstage,thenreleasing thecorrectnumberofbaitpagesalso flowafterthetargetvariableisloadedintomemoryistoactually
found during the profiling stage. It is important to immediately flipthebitsinthevariable.Whiletheprofilingstepallowedusto
launchthevictimprocessoncethebaitpagesarereleasedandto evaluatewhichbitswereflippedinarow,becausewedonotcon-
| start theprocessin | thesame | way thatit was | startedduringthe |     |     |     |     |     |
| ------------------ | ------- | -------------- | ---------------- | --- | --- | --- | --- | --- |
troltheareaofmemorybeingflipped,wecannotseewhichbits
profilingstage.
specificallywereflipped.However,generally,thesuccessofanat-
tackcanbedeterminedbycheckingthenewstateoftheprocess.
AttackingProcessesAfterSendingSIGSTOP.Inpractice,the
Forexample,iftheattackobjectivewasachievedtheattackermay
victimprocesscannotbealteredtosendasignalwhenitisready
bypasspasswordauthentication.
| to beattacked | and wait for | asignal that the | attackhas finished. |     |     |     |     |     |
| ------------- | ------------ | ---------------- | ------------------- | --- | --- | --- | --- | --- |
Instead,wecanusetheSIGSTOPsignaltostoptheprogram’sexe-
|     |     |     |     |     | 5 FLIPPINGBITSINCPUREGISTERS |     |     |     |
| --- | --- | --- | --- | --- | ---------------------------- | --- | --- | --- |
cutionandcreateaprobabilisticmodeltodetermineiftheprocess
Thestackisamemorysectionthatsoftwareprocessesusetostore
hasstoppedinthecorrectplaceintheprocessexecutiontoattack
thevariables.Afterthevariableshavebeenattacked,theSIGCONT valuestemporarily.Weexperimentallyfoundthecontextswitch-
signalcanbesenttocontinueitsexecution.Foranattackertohave ingduringmoderateprocessactivityandtheattackerprocessrun-
permission to send a signal, it must belong to the same session. ningwasenoughtoevictthevictimdatafromthecachetobecor-
ThisisspecialtotheSIGCONTsignal2. ruptedin DRAM without the attacker explicitly flushing it. The
locationofthelastvariableinsertedintothestackissavedinthe
2https://www.sudo.ws/docs/man/1.8.10/sudo.man/

Mayhem:TargetedCorruptionofRegisterandStackVariables ASIACCS2024,July01–05,2024,Singapore
stackpointerregisters.Inassemblylanguage,thestackcanbeused Thisexpandsthescopeoftheattackbeyondjustvariablesstored
| freelytostorevariables.However,higher-levellanguagesandcom- |     |     |     | onthestack. |     |     |     |     |
| ----------------------------------------------------------- | --- | --- | --- | ----------- | --- | --- | --- | --- |
pilersuseaconventionthatisbasedonthearchitectureandtheop- Below,wewilldiscusstwomethodstoattackthestackvariables.
eratingsystem.TheseconventionssetrulesforconvertingCcode
Passive.Thefirstmethodexploitsanaturaloccurrence.Whena
into anassemblycode,suchasSystemVi386,SystemVx86_64,
|     |     |     |     | compiler | uses the | C convention | to create | executable code, it oc- |
| --- | --- | --- | --- | -------- | -------- | ------------ | --------- | ----------------------- |
Microsoftx64,andARM.Eachconventionusesdifferentregisters
casionallystoresregistervaluesinthestackforsafekeeping,e.g.,
forfunctioninputsandreturnvariables,andincertaincasesthe pushinstructionisusedatthebeginningofthefunctioncalls.It
conventionalsousesthestacktostoretemporaryvariables.This ishardtomitigatesinceitisnotvisibleinthesourcecode,which
makesthevariablesvulnerabletofaultattacksbyusingRowham-
makes mostofthelibrariespotentiallyvulnerabledepending on
meronthestack.Nowwewillsummarizetheconventiontoshow
thecompileroptimizations.Higherlevelsofoptimizationsettings
whichsituationscausethecompilertousestackforvariablestor-
incompilersaremoreaggressiveinusingregisters.
age.SinceoursetupisfocusedonLinux,wewillfocusonSystem
|          |     |     |     |           |            |      | There         | are a number of com- |
| -------- | --- | --- | --- | --------- | ---------- | ---- | ------------- | -------------------- |
| Vx86_64. |     |     |     |           |            |      | mon functions | that push regis-     |
|          |     |     |     | Listing   | 2: Example | of   |               |                      |
|          |     |     |     | push from | LibC       | recv | ter values    | to stack by default, |
5.1 ForcingRegisterEvictiontoStack
|     |     |     |     | function |     |     | asshowninFigure2.Forexam- |     |
| --- | --- | --- | --- | -------- | --- | --- | ------------------------- | --- |
Intel-UbuntuCconvention.Thearchitectureuses1664-bitreg- ple, the ebx register is pushed
toasrax, rbx, rcx, rdx, rbp, rsp, rsi, <__recv@@GLIBC_PRIVATE>: to stack by both the sleep()
| isterswhichare | referred |     |     |     |     |     |                             |     |
| -------------- | -------- | --- | --- | --- | --- | --- | --------------------------- | --- |
|                |          |     |     | ... |     |     | function[22]andthegetchar() |     |
rdi,r8-15.Someoftheregistersarespecialpurposeregisters,e.g.
push %ebx
rsp:registerstackpointerandothersaregeneric/scratchpadregis- ... function[14].Listing2fromthe
ters.WhenaCcodeiscompiledandconvertedintoassemblycode %ebx,%eax glibc library shows the recv
p o p %ebx
thefollowingconventionisusedforfunctions: functionpushestheebxregister
% e si
• raxholdsthereturnvalueofthefunction ret tostack,andpopstheebxregis-
terafterthefunctioncompletes.
• rdi,rsi,rdx,rcx,r8,r9holdstheinputparametersofthefunc-
tion.Iftherearemorethan6inputparameters,restiswritten Thisisacommonconventionbe-
intothestack. cause registers are a fast but limited resource, and values are
• rax,rdi,rsi,rdx,rcx,r8-11areusedasscratchregisters. pushedandpoppedfromstacktooptimizetheirusage.Oncesuch
• rax, rdi, rsi, rdx, rcx, r8-11are acodepatternofstoringsecurityvariablesinregistersandthen
|     |     | caller-saved | registers. This |     |     |     |     |     |
| --- | --- | ------------ | --------------- | --- | --- | --- | --- | --- |
pushingthemtostackisfound,thesevaluescanbeattackedvia
meansthatifaroutinecallsasubroutine,itistheresponsibil-
Rowhammer.
| ityof themain | routine | topreserve thevalues | ofany relevant |     |     |     |     |     |
| ------------- | ------- | -------------------- | -------------- | --- | --- | --- | --- | --- |
registers, asthesubroutineisfree tomodifythem.To dothis, Active.Wecanactivelyforceregistersintothekernelstackbytrig-
thecallingfunctioncansavethesevaluesinotherregistersthat
geringasignalhandlerfunctionthatpushestheregistersintothe
willnotbechangedduringthesubroutinecallorsavethemon
kernelstack.Thisisabuilt-inpartoftheLinuxkerneltooptimize
thestack. registerusage.Thisenablesanewtypeofactiveattack,wherewe
• rbx,rsp,rbp,r12-15arecallee-savedregisters.Whenaroutine cantargetvariablesthatarestoredinregisters.AsseeninFigure
makesasubroutinecall,itistheresponsibilityofthesubroutine 2,eventhoughthevariablesmaynotbestoredinthestackduring
| to ensure | that the values | of the relevant registers | remain un- |     |     |     |     |     |
| --------- | --------------- | ------------------------- | ---------- | --- | --- | --- | --- | --- |
thecompilationconvention(asdiscussedpreviously),wecansend
changedafterthesubroutinecalliscompleted.Toachievethis,
|     |     |     |     | asignal | tothevictimprocessorcreateacontention |     |     | byrunning |
| --- | --- | --- | --- | ------- | ------------------------------------- | --- | --- | --------- |
thesubroutinepushesthecontentsofthese registersontothe anotherprocessormakingasystemcall.Thiswillresultinthevic-
stackandthenrestorestheoriginalvalueswhenithasfinished timprocessstoringitsCPUregistersinthekernelstack,making
executingbypoppingthemfromthestack. thevariablesvulnerabletoaRowhammerattack.Wefoundexper-
• Whenafunctioncallhasalargenumberofvariabledeclarations,
imentallythatsignalhandlersimplementedintheCprogramsby
| compilersaimto | utilizeasmany | registers as possibletostore |     |              |           |           |                   |                |
| -------------- | ------------- | ---------------------------- | --- | ------------ | --------- | --------- | ----------------- | -------------- |
|                |               |                              |     | default push | registers | to stack, | so any vulnerable | data stored in |
thesevaluesinordertooptimizeperformance.However,when thoseregisterswouldbecandidatesforaRowhammerattack.
thenumberofavailableregisters isinsufficient toholdallthe Since SIGSTOP cannot behandled by user programs, using it
variables, the compilers will resort to using the stack to store alonewillnotflushregisterstotheuserstack.However,ifapro-
theexcessvariables. gramhasacustomsignalhandler3,theLinuxkernelsavesthereg-
WhenthecompilersusetheIntel-UbuntuCconvention,theex-
|     |     |     |     | ister content | to the | user stack | while the signal | handler is being |
| --- | --- | --- | --- | ------------- | ------ | ---------- | ---------------- | ---------------- |
cessvariablesarestoredonthestackifafunctionhasmanyvari- executed.Thismechanismdoesnotrelyontheuser’sapplication
ables. This makes the variables vulnerable to stack attacks. Our logic.WhenwesendSIGSTOPrightaftertheprevioussignal,the
inspectionofdisassembledcodeofcommonlibrariesshowsthat user’ssignalhandleralsostops,andtheregistercontentoftheuser
thesecasesarelesscommonascompilersaimtoreducestackus- programstaysinmemoryuntilreceivingaSIGCONT,givingthe
age,butthereisstillapossibility.Ofcourse,theattackcanonlybe
attackertimetoexecuteRowhammer.
| executedifthetargetedvariableiswrittentothestack.Toenable |     |     |     | 3PoC     |          |       |                                            |     |
| --------------------------------------------------------- | --- | --- | --- | -------- | -------- | ----- | ------------------------------------------ | --- |
|                                                           |     |     |     | register | spilling | using | SIGINT: https://gist.github.com/anonymous- |     |
thestackattack,wecanforceprocessestotemporarilystoreregis- 60819/d1d6137e17c34f761f7b33d60e922c9d
tercontentsonthestackduringtheexecutionofanotherprocess.

ASIACCS2024,July01–05,2024,Singapore Adilettaetal.
Forcing Register Eviction to DRAM
Processor
Registers
Explicit Push or
SignalHandleror
Context Switch
Cache
Contentionor
clflush()
Main Memory (DRAM) rekcattA
Victim Variable
rekcattA
(a)DDR3 (b)DDR4
Figure 2: We can evict registers to stack by switching con-
texts, which pushes the registers to cache, and then with Figure3:ThecomparisonofheatmapsofbitflipsinDDR3
contention,wecanevictthemtoDRAMwheredatacanbe and DDR4 DRAM chips. Darker color illustrates the loca-
flippedwithRowhammer. tions of more reproducible bit flips. The bit flips seen in
DDR4arelessreproduciblethanDDR3.
Thisisdifferentthancontextswitchingwhichmayforceregis-
tersintokernelstack.Inthecontext-switchingcase,theOSsched- sizegetsclosertoapagesize,everybitflipfoundispotentiallyuse-
uleseachprocessforaspecificamountoftime,switchingbetween fulsinceitwilllandonthetarget.However,asthetargetrequires
themasneeded.TheOSsavesthecontentsandstateoftheCPU moreprecision,itishardertofindalignedbitflips;therefore,itis
(including registers) to a stack in order to allow a processto re- criticaltoattemptonlywhenwefindhighlyreproduciblebitflips.
sumefromwhere it left offwhenit isreloaded.Thecontents of Thisway,wecanputtheburdenontheofflinememoryprofiling
some registers are saved to the kernel stack associated withthe phaseandkeeptheonlinetimeasshortandaccurateaspossible.
process,whichcanstillbeflippedasshownin[42]. To test the reproducibility of bit flips, we select a 64 MB physi-
callycontiguousmemorybuffer.InDDR3,weapplydouble-sided
6 EXPERIMENTALEVALUATION Rowhammer and slide the Attacker-Victim-Attacker window by
oneateverystep.Oncewefinishthebuffer,westorethebitflip
6.1 ExperimentSetup
locationsandstartthesameprocessfromthebeginning.Weham-
TheexperimentsareconductedonasystemwithUbuntu20.04.01 merthesamememorybufferfor100timesandcountthenumber
LTSwith5.15.0-58-genericLinuxkernelinstalled.Thesystemuses offlipsforeachbitlocationthathasflippedatleastonce.Wefound
anIntelCorei9-9900KCPUwithaCoffeeLakemicroarchitecture. 1667uniqueflippybitlocationsintotal.Figure3illustratesthefre-
Weusedadynamicclockfrequencyratherthanastaticclockfre- quencyofbitflipsinaheatmap.Weobservethatonlyalimited
quency to improve the practicality of the attack. End-to-end at- portionoffoundbitflipsareactuallyreproducible,whilemostof
tackexperimentsaredoneonasingleDIMMCorsairDDR4DRAM themarenotreproducibleatallin100trials.
chipwithpartnumberCMU64GX4M4C3200C16and16GBcapac-
ity.DRAMrowrefreshperiodiskeptas64mswhichisthedefault 6.3 SuccessRateofBaitingMethod
valueinmostsystems.Fortheexperimentsonsudo,weusever-
During our experiments, we found that with ASLR enabled, we
sion1.9.12p14,whichisthelatestsudoversionatthetimeofthis
couldsuccessfullylocatethetargetpageintotheflippyrowabout
work.WeusetheportableOpenSSHlibrarywithversion9.1p15for
30%ofthetime.Withfurtherengineeringefforts,thisnumbercan
SSHexperiments.Tobetteraccommodatetheserverenvironment
bebroughtupto80%[27].InSection7,werefertotheabilityto
andreducethenoisecausedbydesktopapplications,weusethe
locateourtargetpageintotheflippyrowasourbait-pagesuccess
OS in console mode. For Rowhammer to successfully attack the
rate,aswedeallocateasetnumberofbaitpagesforthesystemin
stackofaprogram,thevariablesbeingattackedneedtobeloaded
thehopesitwillforceourtargetvariableintoavulnerableplace
intomemoryattherighttime.ForexperimentalpurposesinSUDO,
inmemory.WealsoobservedthatASLR-randomizedpageoffsets
OpenSSH, OpenSSL andMySQL, weusedsignalstomakesurethat
canpartiallyleakedthroughthenumberofPageFaultswhichcan
theprogramsweresynchronized.
furtheroptimizeourattackasseeninFigure4.Furtheranalysisis
giveninAppendixB.
6.2 ReproducibilityofBitFlips
Untilthiswork,thereproducibilityofbitflipsinducedbyRowham- 6.4 EvaluationonDifferentDRAMChips
merwasnotanalyzedindetail.Therefore,itwasnotknownwhether
Bothofflineandonlinephasesofourattackrequirefindingbitlo-
eachflippylocationhasequallyreproducibleornot.Asthetarget
cations that are vulnerable to Rowhammer attack. Since the bit
4sudogitcommitnumber3396267291328eccfcbc7bfb1729c77f30216513 flipfrequencydependsheavilyonhowflippyaDRAMchipis,we
5OpenSSHgitcommitnumber0ffb46f2ee2ffcc4daf45ee679e484da8fcf338c evaluateourattackondifferentDRAMchipsfrombothDDR3and
DDR4memoryprofiles.Wehavetaken14DDR3memoryprofiles

Mayhem:TargetedCorruptionofRegisterandStackVariables ASIACCS2024,July01–05,2024,Singapore
Listing3:ReturnsAUTH_SUCCESSifpasswordiscorrectAUTH_-
FAILUREotherwise.
1 // Gadget
|     |     |     |     |     | 2 int auth   | = 0;       |     |     |     |     |
| --- | --- | --- | --- | --- | ------------ | ---------- | --- | --- | --- | --- |
|     |     |     |     |     | 3 //password | check code |     |     |     |     |
|     |     |     |     |     | if(auth      | != 0)      |     |     |     |     |
4
|     |     |     |     |     | 5   | return AUTH_SUCCESS; |     |     |     |     |
| --- | --- | --- | --- | --- | --- | -------------------- | --- | --- | --- | --- |
6 else
|     |     |     |     |     | 7   | return AUTH_FAILURE; |     |     |     |     |
| --- | --- | --- | --- | --- | --- | -------------------- | --- | --- | --- | --- |
7 ATTACKS–INJECTINGFAULTSINTO
Figure4:PageFaultSideChannelAnalysisDemonstrating
PROGRAMS
ARelationshipBetweenMinorPageFaultsandPageOffset
OurattacksrequirefindingvulnerabilitiesinthecodewecallRowham-
mergadgets.Rowhammergadgetsarepiecesofcodewithsecurity-
𝑝𝑓𝑎𝑢𝑙𝑡
Brand SerialNumber Size criticallogicthatcanbecorruptedandbypassedbyaRowhammer
|     |     |     |     | [GB] [%] |     |     |     |     |     |     |
| --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- | --- |
attack.Itgenerallyconsistsofastackvariablebeingsettoanini-
|     | Corsair | CMD16GX3M2A1600C9 |     | 16 99.99 |     |     |     |     |     |     |
| --- | ------- | ----------------- | --- | -------- | --- | --- | --- | --- | --- | --- |
tialvalue,thenchangeddependingontheprogramflow,andbeing
|     | Corsair | CML16GX3M2C1600C9 |     | 16 99.99 |     |     |     |     |     |     |
| --- | ------- | ----------------- | --- | -------- | --- | --- | --- | --- | --- | --- |
Corsair CML8GX3M2A1600C9W 8 99.99 evaluatedasbeingnotequaltoacertainvalueasillustratedinList-
Corsair CMY8GX3M2C1600C9R 8 97.26 ing3.Wecandefineanintegerstackvariableauthasequaltozero
|     | Crucial | BLS2C4G3D1609ES2LX0CEU |     | 8 72.34 |     |     |     |     |     |     |
| --- | ------- | ---------------------- | --- | ------- | --- | --- | --- | --- | --- | --- |
initially,thenafterapasswordcheck(whichwouldsetauthto1
|     | Geil | GPB38GB1866C9DC |     | 8 99.95 |     |     |     |     |     |     |
| --- | ---- | --------------- | --- | ------- | --- | --- | --- | --- | --- | --- |
3RDD
Goodram GR1333D364L9/8GDC 8 57.47 ifenteredcorrectly),checkifthevariableisnotequaltozero.We
|     | GSkill | F3-14900CL8D-8GBXM |     | 8 90.44 |     |     |     |     |     |     |
| --- | ------ | ------------------ | --- | ------- | --- | --- | --- | --- | --- | --- |
wouldconsiderthisexampleaRowhammergadgetbecauseanybit
|     | GSkill | F3-19200C10-8GBZHD |     | 8 99.99 |     |     |     |     |     |     |
| --- | ------ | ------------------ | --- | ------- | --- | --- | --- | --- | --- | --- |
GSkill F3-14900CL9D-8GBSR 8 88.76 flipintheauthvariablewouldresultinitbeingnotequalto0,thus
Hynix HMT351U6CFR8C-H9 8 99.77 passingtheauthentication.Itwouldbebetterforsecurity-related
V7 V73T8GNAJKI 8 45.17 codetorequirethatcodebeequaltoacertainvalueratherthan
|     | PNY | MD8GK2D31600NHS-Z |     | 6 92.58 |     |     |     |     |     |     |
| --- | --- | ----------------- | --- | ------- | --- | --- | --- | --- | --- | --- |
Integral IN3T4GNZBIX 4 79.19 checkifitisnotequaltoacertainvalue.
|     | Samsung | M378B5173QH0       |     | 4 69.67  |                                 |     |     |     |     |     |
| --- | ------- | ------------------ | --- | -------- | ------------------------------- | --- | --- | --- | --- | --- |
|     | Samsung | M378B5773DH0       |     | 2 99.69  | 7.1 BypassingSUDOAuthentication |     |     |     |     |     |
|     | Corsair | CMU64GX4M4C3200C16 |     | 64 99.99 |                                 |     |     |     |     |     |
sudoisaprocessinLinux-basedoperatingsystemsthatstandsfor
|     | 4RDD Corsair | CMK32GX4M2B3200C16 |     | 32 99.98 |     |     |     |     |     |     |
| --- | ------------ | ------------------ | --- | -------- | --- | --- | --- | --- | --- | --- |
GSkill F4-3600C16D-16GVKC 16 99.99 SuperUser Do. It allowsauser to obtainrootaccess to reading,
|     | Crucial | CT8G4DFD824A.C16FF |     | 8 90.47 |     |     |     |     |     |     |
| --- | ------- | ------------------ | --- | ------- | --- | --- | --- | --- | --- | --- |
writing,andexecutingprotectedfilesgiventheyenterthecorrect
Table1:Theprobabilityofflippingatleastonebitina32-bit password.Breakingthefunctionalityof sudoisatextbookprivi-
integercalculatedon16differentDDR3chipsand4DDR4 legeescalationattackandcanbedevastatingtosystemsthathide
chips perprofile (128or 256MBs).Inour setup,ittakes95 crucial infrastructure behind the root password. The system ad-
| minutes | to profile | a 128 MB | on DDR3 | and 480 minutesto |     |     |     |     |     |     |
| ------- | ---------- | -------- | ------- | ----------------- | --- | --- | --- | --- | --- | --- |
ministratorsetsarootpasswordthatisstoredandhashedonthe
profile256MBonDDR4chips.
system,andwhenauserentersapassword,thehashesofthetwo
passwordsarecompared,andiftheymatch,rootaccessisgranted
totheuser.ThisisseeninthecodesamplegiveninListing4.
Afaultinjectionattackhasbeenproposedonthesudoprogram
from[40],andwegeneratedtheremaining6memoryprofileson
beforeusingadifferenttechnique[15]thatrequiresaspecificbit
ourDRAMchips.Intotal,wehaveanalyzed20DRAMchips.The
|     |     |     |     |     | flip. Theresearchers | foundareas |     | in thesudo | binary where | abit |
| --- | --- | --- | --- | --- | -------------------- | ---------- | --- | ---------- | ------------ | ---- |
resultsaresummarizedinTable1.Inthelastcolumn,wecansee
flipcouldresultinanopcodechangewhichcouldresultinprivi-
theprobabilityoffindingatleastoneflipina32-bitintegerafter
legeescalation.Theresearchersfoundatotalof29bitsthatcould
profiling0.1%ofthetotalmemory.Tocalculatethisprobability,we
beflipped,resultinginprivilegeescalation.Anopcodefliprequires
firstfind𝑛𝑎𝑣𝑔,theaveragenumberofbitflipsthatlandona32-bit
highprecision;onceapagewithflippybitsisfoundthroughthe
variablefor256possiblepageoffsets.Then,wecalculatetheprob-
|                 |     |                                          |     |     | Rowhammer | profiling stage, | a flippy | bit needs | to be located | in  |
| --------------- | --- | ---------------------------------------- | --- | --- | --------- | ---------------- | -------- | --------- | ------------- | --- |
| abilityofhaving |     | asuccessfulattackwithasingleflippypageby |     |     |           |                  |          |           |               |     |
thecorrectpositionwithinthepage.Flipabitthatisasinglebit-
dividingtheaverageflipcount,𝑛𝑎𝑣𝑔bythetotalnumberofflippy
distanceawayfromthetargetwillresultinabrokensudoprogram
| pages,𝑛 | .Finally,forastealthyattack,weassumeweonlyuse |     |     |     |                                                       |     |     |     |     |     |
| ------- | --------------------------------------------- | --- | --- | --- | ----------------------------------------------------- | --- | --- | --- | --- | --- |
|         | flippy                                        |     |     |     | andmayrequireuptoasystemreboot.Incontrast,ourattackon |     |     |     |     |     |
0.1%ofthetotalmemorysize,𝑁pages.Thefinalfaultprobabilityis
theRowhammergadgetcodeworksifanybitinthematchedvari-
|               |       | =(1−(1−𝑛𝑎𝑣𝑔/𝑛 |        | )𝑁pages/1000)×100.While |     |     |     |     |     |     |
| ------------- | ----- | ------------- | ------ | ----------------------- | --- | --- | --- | --- | --- | --- |
| calculatedas𝑝 | fault |               | flippy |                         |     |     |     |     |     |     |
ableisflipped,consistingof4bytesor32bitsforthematchedvari-
probabilitiesareover90%formostDRAMchips,itisimportantto
ablealone.
notethatotherfactorsaffecttheprobabilityofseeingaflipinthe
|     |     |     |     |     | Afterrunning | thesudo | experiment | for10hrs34minutes,we |     |     |
| --- | --- | --- | --- | --- | ------------ | ------- | ---------- | -------------------- | --- | --- |
targetvariableofanactualprocess,includingtheprobabilitythat
sawatotalof11successfulattackswherewegainedrootaccess.
theprocessgetsloadedintotheflippypageinthefirstplace. Thisamountstoanaverageofaboutanhourofprofiling,asseen

| ASIACCS2024,July01–05,2024,Singapore |     |     |     |     |     | Adilettaetal. |
| ------------------------------------ | --- | --- | --- | --- | --- | ------------- |
Listing4:Passwordauthenticationfunctioninsudo.Returns Listing5:PasswordauthenticationfunctioninOpenSSH.Re-
AUTH_SUCCESSifpasswordiscorrectAUTH_FAILUREotherwise. turns1ifthepasswordiscorrectand0otherwise.
1 int sudo_passwd_verify(...) { 1 int mm_answer_authpassword(...){
| 2 char des_pass[9], | *epass;       |     | 2 char *passwd;         |     |     |     |
| ------------------- | ------------- | --- | ----------------------- | --- | --- | --- |
| 3 char *pw_epasswd  | = auth->data; |     | 3 int r, authenticated; |     |     |     |
| size_t              | pw_len;       |     | ...                     |     |     |     |
| 4                   |               |     | 4                       |     |     |     |
5 int matched = 0; 5 authenticated=options.password_authentication
| 6 ...   |                        |              | 6 && auth_password(ssh, | passwd); |     |     |
| ------- | ---------------------- | ------------ | ----------------------- | -------- | --- | --- |
| 7 epass | = (char *) crypt(pass, | pw_epasswd); | 7 ...                   |          |     |     |
8 if (epass != NULL) { 8 if ((r=sshbuf_put_u32(m, authenticated)) != 0)
9 if (HAS_AGEINFO(pw_epasswd, pw_len) 9 fatal_fr(r, "assemble");
| 10  | && strlen(epass) | == DESLEN) | 10 ... |     |     |     |
| --- | ---------------- | ---------- | ------ | --- | --- | --- |
11 matched = !strncmp(pw_epasswd, epass, DESLEN); 11 return (authenticated);
| 12 else    |                       |         | 12 } |     |     |     |
| ---------- | --------------------- | ------- | ---- | --- | --- | --- |
| 13 matched | = !strcmp(pw_epasswd, | epass); |      |     |     |     |
14 }
15
16 explicit_bzero(des_pass, sizeof(des_pass)); Listing 6: Password authentication function in OpenSSH.
17
Triestoauthenticatetheuserusingpassword.Returnstrue
| 18 debug_return_int(matched |     | ? AUTH_SUCCESS   |                           |     |     |     |
| --------------------------- | --- | ---------------- | ------------------------- | --- | --- | --- |
| 19                          |     | : AUTH_FAILURE); | ifauthenticationsucceeds. |     |     |     |
20 }
1 int auth_password(...){
|     |     |     | 2 Authctxt *authctxt | = ssh->authctxt; |     |     |
| --- | --- | --- | -------------------- | ---------------- | --- | --- |
inTable2,toseeasuccessfulattack.Additionally,weseethatthe 3 int result, ok = authctxt->valid;
4 ...
totalonlinetimeislessthananhourtoseethe11flips,soatotalof 5 if (*password == '\0' && options.permit_empty_passwd == 0)
0;
| 5minutesofhammeringonaverageonthesudoprogramitselfto |     |     | 6 return |     |     |     |
| ---------------------------------------------------- | --- | --- | -------- | --- | --- | --- |
7 ...
seeaflip.Thetimebetweensuccessfulattacksoccasionallyvaried
|     |     |     | 8 result = sys_auth_passwd(ssh, | password); |     |     |
| --- | --- | --- | ------------------------------- | ---------- | --- | --- |
-sometimeswewouldsee2-3attacksina15-20minutewindowof 9 if (authctxt->force_pwchange)
profiling.Othertimesitmaytakeuptoafewhours.Wespeculate 10 auth_restrict_session(ssh);
|     |     |     | return (result | && ok); |     |     |
| --- | --- | --- | -------------- | ------- | --- | --- |
11
| thistobeduetowheretheprocessisbeingplacedinmemory,as |     |     | 12 } |     |     |     |
| ---------------------------------------------------- | --- | --- | ---- | --- | --- | --- |
someareasoftheDRAMbanksmaybemoreflippythanothersdue
tomanufacturingdefects.Wealsonotedthatofthe5334attacks,
wesaw1989attackswherethetargetvariablewasplacedcorrectly isachievedinaninfinitewhileloop.Whentheservergetsacon-
intheflippypage.Thisisabait-pagesuccessrateofabout37%.
nectionrequestfromaclientwithagivenusernameandpassword,
WewereinitiallyconcernedthattheRowhammerwouldfliptoo achainoffunctionsiscalledtocheckiftheprovidedpasswordis
manybitsinthestackoftheprocessthatitwouldbeunabletofin- correct.Here,wemention thetwomostimportantonesthatwe
ishexecution.Whilewedidfindthatitwasflippingbitsinother canuseforourattack.Thefirstfunctionismm_answer_authpass-
variablesotherthanmatchedunintentionally,theprogramstillex- word,andthesecondfunctionisauth_password
whichiscalled
ecutedsuccessfully,andwhenmatchedwasflippedwegainedroot
bythefirstone.Weshowthetruncatedversionsofthesefunctions
access.Fortunately,stabilitydidnotbecomeanissueinourexper-
inListing5and6.Withinthesetwofunctions,therearetwodiffer-
iments. The results of the experiment demonstrate the novel at- entlocalvariablesthatcarrytheinformationregardingiftheuser
tackonstackcanenableprivilegeescalationbyflippingbitsinthe willbeauthenticatedlateron.
stack. Infunctionmm_answer_authpassword, authenticated flagis
|     |     |     | set if the auth_password | returns | 1 in line 5 and | then returned. |
| --- | --- | --- | ------------------------ | ------- | --------------- | -------------- |
7.2 BypassingOpenSSHAuthentication
|     |     |     | After being returned, | the return value | is checked | if it equals 1. |
| --- | --- | --- | --------------------- | ---------------- | ---------- | --------------- |
Todemonstratetheextent ofthenew attacksurfacethatourat- The client is authenticated, and if the condition is met and the
tackworkenables,weimplementtheattackonSSHprotocols.SSH SSHsessionstarts.Otherwise,theclientisaskedtoenterthepass-
(SecureShellProtocol)isanapplicationlayerprotocolthatallows wordagain.Ifthecorrectpasswordisnotgiveninthreetrials,the
secureremoteuserlogin,commandexecution,andotherremote clienthastosendtheconnectionrequestagain.Here,theauthen-
networkoperationssuchasTCPportforwarding,tunneling,and ticated flag is stored in thestack memoryofthe programand,
filetransfer.SSHprotocolworksinaclient-servermodel.Public- therefore, in DRAM, and a potential target for our attack. If we
keyencryptionisusedforauthenticatingtheclientandtheserver fliptheleastsignificantbitofthis32-bitintegervalueafterline5,
toeachother.Aftertheauthenticationphase,thetransferreddata weseethattheclientisauthenticatedregardlessofthepassword
issecuredusingsymmetrickeyencryptionschemes,suchasAES. value,andremoteshellaccessisgiven.However,flippingotherbits
SeverallibrariesimplementSSHprotocol.OpenSSHisoneofthe otherthantheleastsignificantbitresultsinauthenticationfailure,
mostpopularimplementationsofSSHprotocol.Severalattackson evenifthepasswordiscorrect.
OpenSSHhavebeenshowntostealRSAsessionkeys[27]. Theothertargetforourattackistheresultflaginauth_pass-
Whentheserverprogramstarts,itconstantlymonitorsthein- wordfunction.Itisinitializedto0inline3andsetto1inline5
ifthepasswordiscorrect.Notethattheresultflagisgiventoa
comingconnectionsrequesttoport22bydefault.Thismonitoring

Mayhem:TargetedCorruptionofRegisterandStackVariables ASIACCS2024,July01–05,2024,Singapore
Category SUDO OpenSSH OpenSSL Listing7:ErrorcheckinginOpenSSLModExp
| TotalTime   |     | 1hr9mins |     | 45mins | 1hr45mins |       |        |                             |     |     |            |            |          |
| ----------- | --- | -------- | --- | ------ | --------- | ----- | ------ | --------------------------- | --- | --- | ---------- | ---------- | -------- |
|             |     |          |     |        |           |       | static | int rsa_ossl_mod_exp(BIGNUM |     |     | *r0, const | BIGNUM *I, | RSA *rsa |
| OnlineTime  |     | 5mins    |     | 6mins  |           | 7mins | 1      |                             |     |     |            |            |          |
|             |     |          |     |        |           |       |        | , BN_CTX *ctx){             |     |     |            |            |          |
| FlippyPages |     |          | 485 | 513    |           | 686   |        |                             |     |     |            |            |          |
2 ...
| CorrectBaiting |     |     | 181 | 206 |     | 139 | 3 if (rsa->e | && rsa->n)                | {   |     |                     |     |     |
| -------------- | --- | --- | --- | --- | --- | --- | ------------ | ------------------------- | --- | --- | ------------------- | --- | --- |
|                |     |     |     |     |     |     | 4            | if (rsa->meth->bn_mod_exp |     |     | == BN_mod_exp_mont) | {   |     |
Table2:ResultsfromtheSUDO,OpenSSHandOpenSSLexperi- if (!BN_mod_exp_mont(vrfy, r0, rsa->e, rsa->n, ctx,
5
mentsshowingofflinetimeandonlinetime,andthenum- 6 rsa->_method_mod_n))
| berofflippypagesfound,aswellasthenumberofattacks |     |     |     |     |     |     | 7   | goto     | err; |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | -------- | ---- | --- | --- | --- | --- |
|                                                  |     |     |     |     |     |     | 8   | } else { |      |     |     |     |     |
withthecorrectnumberofbaitpagesreleased 9 bn_correct_top(r0);
|     |     |     |     |     |     |     | 10  | if (!rsa->meth->bn_mod_exp(vrfy, |     |     |     | r0, rsa->e, | rsa->n, |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------- | --- | --- | --- | ----------- | ------- |
ctx,
|            |                                              |     |     |     |     |     | 11  |                   |      |       | rsa->_method_mod_n)) |     |     |
| ---------- | -------------------------------------------- | --- | --- | --- | --- | --- | --- | ----------------- | ---- | ----- | -------------------- | --- | --- |
|            |                                              |     |     |     |     |     | 12  | goto              | err; |       |                      |     |     |
| logicaland | operationwithokflag.okflagissetto1ifusername |     |     |     |     |     | 13  | }...              |      |       |                      |     |     |
|            |                                              |     |     |     |     |     | 14  | if (!BN_sub(vrfy, |      | vrfy, | I))                  |     |     |
isvalid.Therefore,aslongastheresultisanonzerovalue,the
|                                        |     |     |     |     |            |       | 15  | goto err;             |     |     |     |     |     |
| -------------------------------------- | --- | --- | --- | --- | ---------- | ----- | --- | --------------------- | --- | --- | --- | --- | --- |
| returnvaluewouldbe1.Thislogicincreases |     |     |     |     | thechanges | ofour |     |                       |     |     |     |     |     |
|                                        |     |     |     |     |            |       | 16  | if (BN_is_zero(vrfy)) |     | {   |     |     |     |
attacksinceaslongasweflipanybitofthe32bitsoftheresult 17 bn_correct_top(r0);
variable,wecansuccessfullybypassthepasswordauthentication. 18 ret = 1;
|                                                      |      |             |      |          |          |           | 19   | goto err; | /* not | actually | error | */  |     |
| ---------------------------------------------------- | ---- | ----------- | ---- | -------- | -------- | --------- | ---- | --------- | ------ | -------- | ----- | --- | --- |
| Table2showstheaveragesofasuccessfulattackontheresult |      |             |      |          |          |           | 20   | } ...     |        |          |       |     |     |
| variable in                                          | SSH. | We observed | that | over the | courseof | about one | 21 } |           |        |          |       |     |     |
andahalfhours,wesawtwototalsuccessfulloginsintotheSSH
serverwithoutthecorrectpassword,whichwouldbeanaverageof
45minutes,asseeninTable2.Thisrequiredatotalof11minutesof releasedin2001andisshowninListing76ThecurrentOpenSSL
implementationperformsacheckoperationtofindifanerroroc-
onlinetime,foranaverageofabout6minutesofhammeringSSH
curredafterthefastCRT-basedRSAexponentiation.Ifanerroris
| per successful | attack. | In order | to  | complete | the attack, | we found |     |     |     |     |     |     |     |
| -------------- | ------- | -------- | --- | -------- | ----------- | -------- | --- | --- | --- | --- | --- | --- | --- |
1025 memory pages in the system with flippy bits. We also saw detected,thecoderunsaslower(non-CRTbased)exponentiation
thatoftheattacks,412outof1025releasedthecorrectnumberof tocomputethesignature,thuspreventingthepossibilityofinitiat-
baitpagessuchthatthetargetvariableofSSHwasplacedcorrectly ingtheBellcoreattack.Thecheckmechanisminvolvesrecomput-
ingthemessageusingthesignatureandpublickey.Recomputed
intheflippyrow.Thisisabait-pagesuccessrateofabout40%.
messageislatersubtractedfromtheoriginalmessagetocheckif
| 7.3 AttackonOpenSSL |     |     | SecurityChecksStored |     |     |     |     |     |     |     |     |     |     |
| ------------------- | --- | --- | -------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
botharethesamemessage.Iftheresultiszero,itmeansthemes-
inStack sages match and the exponentiation is computed correctly. The
zerocheckfunctioncanbeseeninline17inListing7.
| We experiment | with | a simple | OpenSSL | process | where | we target |     |     |     |     |     |     |     |
| ------------- | ---- | -------- | ------- | ------- | ----- | --------- | --- | --- | --- | --- | --- | --- | --- |
Forasuccessfulattack,thefirststepistocreateafaultinone
| a security | check | variable. | At the | end of the | ECDSA | sign setup |              |              |     |                    |     |         |       |
| ---------- | ----- | --------- | ------ | ---------- | ----- | ---------- | ------------ | ------------ | --- | ------------------ | --- | ------- | ----- |
|            |       |           |        |            |       |            | ofthepartial | CRT-basedRSA |     | computations.Then, |     | another | fault |
method,asecuritycheckdeterminesifavariablecalledretisnot
isintroducedinthecheckmechanismtotrickthecodeintothink-
equaltozero.Ifthevariableisequaltozero,itmeansthatasecu-
|     |     |     |     |     |     |     | ing the CRT-based |     | RSA exponentiation |     | has | been calculated | cor- |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ------------------ | --- | --- | --------------- | ---- |
ritycheckfailedand ajumpoccurredpastwhere thevariableis rectly7.Thisisachievedbylaunchingastackattackonthefunc-
| set to1,indicating |     | allsecuritycheckspassed.A |     |     | successfulsecu- |     |     |     |     |     |     |     |     |
| ------------------ | --- | ------------------------- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
tion𝐵𝑁_𝑖𝑠_𝑧𝑒𝑟𝑜.Whenline17callsfor𝐵𝑁_𝑖𝑠_𝑧𝑒𝑟𝑜function,the
ritybypasswouldhammerthesecurityvariableinthestackand
resultofthezerocheckisreturnedusingtheEAXregister.Wecan
forceittobe1regardlessofifitmadeajumpornot.Thiscould
|     |     |     |     |     |     |     | force the | process to | halt and | put | the result | on the stack. | By us- |
| --- | --- | --- | --- | --- | --- | --- | --------- | ---------- | -------- | --- | ---------- | ------------- | ------ |
potentiallybeusedinconjunctionwithaRowhammerattackthat
ingRowhammer,wecanmanipulatethevariableonceitisonthe
targetsdynamicmemory.
|     |     |     |     |     |     |     | stack. When | thereturn | valueis |     | anything | other than zero, | theif |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --------- | ------- | --- | -------- | ---------------- | ----- |
FromTable2wecanseethatthereisanaverageofflinetimeof1
casewillbeexecuted,givingtheappearancethattheCRT-based
hr45mins,andanaverageonlineof7minutes.Thisrequiredonly
exponentiationwascomputedcorrectly.
14minutesofhammeringonOpenSSLitself.Duringprofiling,1372
pageswerefoundtobeflippy,and277ofthemwerecorrectlyuti- 8.2 BypassingMySQLAuthentication
lizedbyhavingthetargetsecurityvariableplacedinthemduring
MySQListhemostpopularopen-sourcedatabasemanagementsys-
theattackstage,aresultingbait-pagesuccessrateofabout20%.
|     |     |     |     |     |     |     | tem [38], | which is | commonly | used | by many | organizations | and |
| --- | --- | --- | --- | --- | --- | --- | --------- | -------- | -------- | ---- | ------- | ------------- | --- |
8 VULNERABILITY ANALYSIS websites in all industries from Defense & Government to Social
NetworksincludingUSNavy,NASA,Twitter,Facebook,LinkedIn,
8.1 RSABellcoreAttacks
andBankofAmerica[32].
TheearlyworkbyBonehetal.[2]popularlyreferredtoasBellcore
6https://github.com/openssl/openssl/commit/1777e3fd5eac0e491bb16a0bb37f4b0f298e6486
| attacks, demonstrated |     | the | importance | of checking |     | for errors in |     |     |     |     |     |     |     |
| --------------------- | --- | --- | ---------- | ----------- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- |
7Theprobabilityofbothfaultsgoingthroughwillbelow,howeverBellcorerequires
cryptographicimplementationsinaCRT-basedRSAimplementa-
onlyonefaultysampletosucceed.
tion.ThefirstmitigationagainstBellcoreattaclsonOpenSSLwas

| ASIACCS2024,July01–05,2024,Singapore |     |     |     |     |     |     |     |     | Adilettaetal. |
| ------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | ------------- |
theserver,andisabletoauthenticatetheserver.Importantly,the
Listing8:MySQLpasswordauthentication.Triestoauthenti-
catetheclientusingauthorization_idandscramble.Theau- client is vulnerable to the Rowhammer attack during the phase
thenticationsucceedsiffast_auth_result.first isfalse. whileitiswaitingforaresponsefromtheserver.Thisconnection
phasecantaketime(intheorderofmilliseconds)andisultimately
controlledbytheserver,andtherefore,theattackercanhammer
| 1 static | int caching_sha2_password_authenticate(...){ |     |     |     |                                   |     |     |     |     |
| -------- | -------------------------------------------- | --- | --- | --- | --------------------------------- | --- | --- | --- | --- |
| ...      |                                              |     |     |     | theclient’smemoryduringthisphase. |     |     |     |     |
2
| 3 std::pair<bool, | bool> | fast_auth_result | =   |     |     |     |     |     |     |
| ----------------- | ----- | ---------------- | --- | --- | --- | --- | --- | --- | --- |
4 g_caching_sha2_password->fast_authenticate(
5 authorization_id, reinterpret_cast<unsigned char *>( Client Server
scramble),
| 6   | SCRAMBLE_LENGTH, | pkt, |     |     |     |     |     |     |     |
| --- | ---------------- | ---- | --- | --- | --- | --- | --- | --- | --- |
7 info->additional_auth_string_length ? true : false); ClientHello
| 8 if (fast_auth_result.first) |     | {   |     |     |     |     |     |     |     |
| ----------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
9 if (vio->write_packet(vio, (uchar *)& ServerHello (includes pubkey + signed messaged)
|           | perform_full_authentication, |                          | 1))                   |       |     |     |                               |     |     |
| --------- | ---------------------------- | ------------------------ | --------------------- | ----- | --- | --- | ----------------------------- | --- | --- |
| 10 return | CR_AUTH_HANDSHAKE;           |                          |                       |       |     |     |                               |     |     |
| 11 } else | {                            |                          |                       |       |     |     | Verify signature using pubkey |     |     |
| 12 if     | (vio->write_packet(vio,      | (uchar                   | *)&fast_auth_success, | 1))   |     |     |                               |     |     |
| 13 return | CR_AUTH_HANDSHAKE;           |                          |                       |       |     |     | Client sends sensitive        |     |     |
| 14 if     | (fast_auth_result.second)    | {                        |                       |       |     |     | info if sig verification      |     |     |
| 15 const  | char *username               | =                        |                       |       |     |     | passes                        |     |     |
| 16        | *info->authenticated_as      | ? info->authenticated_as |                       | : ""; |     |     |                               |     |     |
Sends sensitive info (signature valid)
17 }
| 18 return | CR_OK; |     |     |     |     |        |     |     |        |
| --------- | ------ | --- | --- | --- | --- | ------ | --- | --- | ------ |
| 19 }      |        |     |     |     |     | Client |     |     | Server |
Figure5:Typicalscenariowheretheclientconnectstothe
WefoundaRowhammergadget8giveninListing8inthesource
server,sendsamessageandreceivesthemessagesignedby
codeofMySQLserverthatisusedforauthenticatingaclientwitha
theserverandisabletoauthenticatetheserver.
password.Thepasswordcheckhappensbetweenline3and7and
theresultisstoredinfast_auth_result.Whenwesimulatea0to
|                               |     |                          |     |     | InFigure6,wecanseetheattackscenario. |     |     |     | Theattackercapi- |
| ----------------------------- | --- | ------------------------ | --- | --- | ------------------------------------ | --- | --- | --- | ---------------- |
| 1fliponfast_auth_result.first |     | inline8,weobservethatthe |     |     |                                      |     |     |     |                  |
clientisauthenticatedevenwithanincorrectpassword.Notethat, talizesonthefactthattheclientisvulnerabletoRowhammerdur-
unlikethepreviousattacks,thetargetvariablerequiressingle-bit ingtheconnectionphase.Theattackeractsasboththeserverand
iscolocatedwiththeclient.Theattackerrespondstotheclients
precision;hence,theattackishardertoachieveusingRowhammer.
ClientHellowithaServerHellomessage,whichincludestheat-
9 END-TO-ENDATTACKEXAMPLE tacker’spublickey and asignature ofthehandshake. Theclient
|                   |         |                       |     |             | will | then verify | the signature | using the attacker’s | public key. If |
| ----------------- | ------- | --------------------- | --- | ----------- | ---- | ----------- | ------------- | -------------------- | -------------- |
| The effectiveness | of this | attackby demonstrated | by  | deploying a |      |             |               |                      |                |
theattackercanflipabitinthesignatureverificationprocess,the
client/serversignatureverificationviaOpenSSL.Notethatthisex-
|     |     |     |     |     | clientwillthinkthesignatureisvalidandwillsend |     |     |     | sensitive in- |
| --- | --- | --- | --- | --- | --------------------------------------------- | --- | --- | --- | ------------- |
ampledoesnotusesignalsorsignalhandlersforsynchronizingthe
formationtotheattacker.Theoretically,Theattackercanthenfor-
attackerandthevictimbutratherusestheconceptofablocking
wardthemessagetotherealserverandreceivetheresponse.The
windowinherenttotheclient/serversignatureverificationprocess
attackercanthenforwardtheresponsetotheclient,andtheclient
toensuretheattackerhammersattherighttime.Theattackerisas-
willthinkitiscommunicatingwiththerealserver,otherwiseknown
sumedtobecolocatedwiththeclientandwillhammerthevictim
asaman-in-the-middleattack.
client’shigh-levelsignatureverificationprocessforcingittointer-
Thisfullattackscenarioconsistsof3steps:
pretafaultysignatureasvalid.Thisisinthecontextofaman-in-
the-middleattack,whereanattackeristryingtotrickaclientinto
| thinkingaserveristheauthentictargettheyaretryingtoconnect |     |     |     |     |     | Hammering  |        |     |             |
| --------------------------------------------------------- | --- | --- | --- | --- | --- | ---------- | ------ | --- | ----------- |
|                                                           |     |     |     |     |     | Attacker   | Client |     | Fake Server |
to.
ClientHello
| Inthetypicalscenario,theclientwillattempttoconnecttothe |     |     |     |     |     | Rowhammer |     |     |     |
| ------------------------------------------------------- | --- | --- | --- | --- | --- | --------- | --- | --- | --- |
server and will send a ClientHello message to the server. The Attack ServerHello
(includespubkey+ signedmessaged)
serverwillrespondwithaServerHellomessage,whichincludes
theserver’spublickeyandasignatureofthehandshake.Theclient Verify signature using pubkey
willthenverifythesignatureusingtheserver’spublickey.Ifthe Attacker causes bit flip
bypassing signature
signatureisvalid,theclientwillassumethatitissafetosendsen-
verification process
sitiveinformationtotheserver.Iftheattackercanflipabitinthe
Sends sensitive info
signatureverificationprocess,theclientwillthinkthesignatureis (believing signature valid)
Hammering
validandwillsendsensitiveinformationtotheattacker. Attacker Client Fake Server
InFigure5,weseethetypicalscenariowheretheclientconnects
Figure6:Attackscenariowheretheattackeractsasboththe
totheserver,sendsamessageandreceivesthemessagesignedby
fakeserverandcolocatedwiththeclient.
8https://github.com/mysql/mysql-server

Mayhem:TargetedCorruptionofRegisterandStackVariables ASIACCS2024,July01–05,2024,Singapore
Listing9:Clientcodethatconnectstotheserverandsends Listing10:Clientcodethatusesthepass variabletodeter-
amessagetobesigned.ItisvulnerabletotheRowhammer mineifthesignatureisvalid.
attackduringtheconnectionphase.
|               |                   |              |     |     | 1 // remove                  | sensitive data from memory |     |
| ------------- | ----------------- | ------------ | --- | --- | ---------------------------- | -------------------------- | --- |
| 1 int pass=0; |                   |              |     |     | 2 EC_KEY_free(ec_key);       |                            |     |
| 2 // Create   | client socket     |              |     |     | 3 ECDSA_SIG_free(signature); |                            |     |
| 3 client_fd   | = socket(AF_INET, | SOCK_STREAM, | 0); |     | ...                          |                            |     |
4
| 4 ...                |           |          |                 |         | 5 if (pass        | != 0) {                    |     |
| -------------------- | --------- | -------- | --------------- | ------- | ----------------- | -------------------------- | --- |
| 5 // Connect         | to server |          |                 |         |                   |                            |     |
|                      |           |          |                 |         | 6 fprintf(stdout, | "Server Authenticated\n"); |     |
| 6 connect(client_fd, | (struct   | sockaddr | *)&server_addr, | sizeof( | 7 }               |                            |     |
server_addr));
| 7 // Send  | a message to the | server       |     |     |     |     |     |
| ---------- | ---------------- | ------------ | --- | --- | --- | --- | --- |
| 8 unsigned | char message[32] | = "message"; |     |     |     |     |     |
9 send(client_fd, message, sizeof(message), 0); Category Stack Register
10 ...
while ((bytes_received = recv(client_fd, buffer, sizeof(buffer) TotalTime 27mins 36mins
11
|                   | , 0)) > 0){        |         |                  |     |                                   |     |               |
| ----------------- | ------------------ | ------- | ---------------- | --- | --------------------------------- | --- | ------------- |
|                   |                    |         |                  |     | OnlineTime                        |     | 20mins 31mins |
| 12 memcpy(sig_buf | + sig_len,         | buffer, | bytes_received); |     |                                   |     |               |
| 13 sig_len        | += bytes_received; |         |                  |     | TotalFlippyPages                  |     | 447 402       |
| 14 }              |                    |         |                  |     | TotalAttacksw/Correct#ofBaitpages |     | 104 105       |
| // Deserialize    | the signature      |         |                  |     |                                   |     |               |
15
16 const unsigned char *pp = sig_buf; Table 3: Resultsfrom the end-to-endattackon codeusing
17 ECDSA_SIG *signature = d2i_ECDSA_SIG(NULL, &pp, sig_len); OpenSSLclient/serversignatureverification
18 ...
| 19 // Verify                   | the signature |                  |     |            |     |     |     |
| ------------------------------ | ------------- | ---------------- | --- | ---------- | --- | --- | --- |
| 20 if (verify_message(message, |               | sizeof(message), |     | signature, |     |     |     |
ec_key)==SUCCESS){
21 pass = 1; withaCoffeeLakemicroarchitecture.Weusedadynamicclockfre-
22 }
quencyratherthanastaticclockfrequencytoimprovethepracti-
calityoftheattack.
• Step1:TheclientconnectstotheattackerandsendsaClien- 9.2 FlippingaRegisterValuePushedtoStack
tHellomessage.
ThehighlevelsourcecodefortheOpenSSLsignatureverification
• Step2:TheattackersendsaServerHellomessagetotheclient,
whichincludestheattacker’spublickeyandasignatureofthe canbemodifiedtoseeminglymakeitmoredifficulttoattackwith
handshake. Rowhammer.Wecanforcepasstogotoaregisterwiththefollow-
ingCcodefromListing11.
| • Step 3: | The client will | then verify | the signature | using the at- |     |     |     |
| --------- | --------------- | ----------- | ------------- | ------------- | --- | --- | --- |
tacker’spublickey.Iftheattackercanflipabitinthesignature
verificationprocess,theclientwillthinkthesignatureisvalid Listing11:Thepasssecurityvariableisstoredinaregister
| andwillsendsensitiveinformationtotheattacker. |     |     |     |     | insteadofstack |     |     |
| --------------------------------------------- | --- | --- | --- | --- | -------------- | --- | --- |
9.1 TakingadvantageofIPSocketsfor register int pass asm("rbx") = 0;
Synchronization
Thisattackdoesnotrequireanydegradationorothersynchroniza- Itisacommonpracticebycompilersthatregisterspaceisused
tiontechniquestotimethebit-flipattackontheclient.Thisisbe- bydefaultwhenpossibletoincreaseperformance,buttheCcode
causetheattackeriscontrollingthetimethattheverificationpro- inListing11makesitexplicit.Afterassigningpasstoregisterrbx,
cesstakes,andthuscansimplywaitforthebitfliptooccurbefore thecodebehavesthesame,butduringtheblockingwindowwhen
rbx
sendingtheresponsetotheclient. OpenSSL iswaiting toreceive datafromtheserver, register
InListing9weseethattheclienthastheabilitytoverifyasig- ispushedtostackwhereitcanbeattacked.Thisisalsocommon
naturebasedonthepublickey.Itkeepsthestateoftheverification practicetomaximizetheutilizationofregisterswhicharealimited
processinthevariablepass.Thepassvariableissetto1ifthesig- resource.Whenthedataispoppedoffthestackafterreceivingdata
natureisvalid,and0otherwise.Duringtheconnectionphase,the fromtheserver,ifithasbeencorruptedbyRowhammmerandthat
Rowhammerattackercanattackthepassvariableandflipabitto
corrupteddataisthenputintotheregister.
maketheclientthinkthesignatureisvalid.
9.3 ResultsfromEnd-to-EndAttack
InListing10wecanseethatpassisusedtoverifyiftheserver
isauthenticated.If passisnot0,thentheserverisauthenticated. We were able to successfully force the client to misauthenticate
Theattackercanflipabitinthepassvariabletomaketheclient
thedigitalsignaturesentbytheserver.Table3summarizesthere-
| thinktheserver | isauthenticatedregardlessoftheTLSsignature |     |     |     |     |     |     |
| -------------- | ------------------------------------------ | --- | --- | --- | --- | --- | --- |
sults.Itisnotablethattheresultsforattackingthevariableinthe
verificationprocessexecutedpreviously. stack,andattackingitwhenitispushedfromaregisterarecom-
Justaswiththepreviousexperiments,thisfullattackwascon- parablefromapracticalstandpoint.Alsonotethatafterfindinga
ductedonasystemwithUbuntu20.04.01LTSwith5.15.0-58-generic flippylocationinmemory,thestackvariableorregistervariable
Linuxkernelinstalled.ThesystemusesanIntelCorei9-9900KCPU

| ASIACCS2024,July01–05,2024,Singapore |          |         |             |     |           |          |         |     |       |         |     |       | Adilettaetal. |
| ------------------------------------ | -------- | ------- | ----------- | --- | --------- | -------- | ------- | --- | ----- | ------- | --- | ----- | ------------- |
| wasloaded                            | into the | correct | address 23% | and | 26% ofthe | time re- |         |     |       |         |     |       |               |
|                                      |          |         |             |     |           |          | Listing | 12: | Loose | Listing | 13: | Tight | Logic         |
spectively.Basedonthesefindings,wecanconcludethatRegister Logic Suspectable to Less Suspectable to
variablesarenolongersafeagainstRowhammer. RowhammerAttack RowhammerAttack
10 COUNTERMEASURES
|                                      |     |     |     |     |     |     | 1 if(matched | !=  | 0)    |     | if(matched  | == 1) |       |
| ------------------------------------ | --- | --- | --- | --- | --- | --- | ------------ | --- | ----- | --- | ----------- | ----- | ----- |
|                                      |     |     |     |     |     |     | //passwords  |     | match |     | //passwords |       | match |
| 10.1 SystemChangestoPreventRowhammer |     |     |     |     |     |     | 2            |     |       |     |             |       |       |
|                                      |     |     |     |     |     |     | 3 else       |     |       |     | else        |       |       |
Oneofthemostcommoncountermeasurescitedisincreasingthe 4 //passwords don't //passwords don't
|     |     |     |     |     |     |     |     | match |     |     |     | match |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | ----- | --- |
DRAMrefreshrate.Afasterrefreshratewillresultinworseper-
formanceandmorepowerconsumptionandisnotanidealsolu-
Incontrast,Listing13issaferbecauseRowhammerisrequired
tion.Althoughvariousrowrefreshmethodshavebeenproposed
tofliponlytheleastsignificantbit;otherwise,thepasswordsstill
toreducetheoverheadsuchasparallel[50],andprobabilisticrow willnotmatch.Requiringsecurity-sensitivevariablestobestored
refresh[47],theyarenotyetavailableforuseinconsumersystems. inregistersoverstackisnotaneffectivecountermeasurebecause,
| One possibility |     | is Hidden | Row Activation |     | (HiRA). HiRA | is a |                               |     |     |              |     |               |     |
| --------------- | --- | --------- | -------------- | --- | ------------ | ---- | ----------------------------- | --- | --- | ------------ | --- | ------------- | --- |
|                 |     |           |                |     |              |      | asseen inSection5.1,registers |     |     | canbeflushed |     | tomemoryusing |     |
noveltechniqueproposedin[50]whichparallelizesrowrefreshes
signalinterruptsandcanstillbeflipped.Thecodewefoundinsudo
forDRAM.Itallowsarowrefreshoperationtobehiddeninthe
andSSHhavevulnerablecodethatissusceptibletoRowhammer
background while a row in the DRAM is being accessed or re- bychanginglogicforanyflipinthe32-bitvariable,whileMySQL
freshedinthesamebank.Ittakesadvantageofthefactthatdiffer- requiresaleastsignificantbitflip.Additionally,itcanbebeneficial
entrowsinthesamebankmaybeconnectedtodifferentcharge
tousebooleanvariablesoverintegerswhenpossibletoreducethe
restorationcircuitry,allowingforconcurrentrefreshes.Bymaking
targetsize.
anefforttoreducethelatencyofrefreshoperations,HiRAcanre-
Listing14:Specificpatterninthematchedvariableincreases
ducethetimewindowforRowhammerattacks.HiRAclaimstobe
thenumberofbitsthatneedtobeflippedresultinginafault-
abletoconcurrentlyrefresh32%ofrowsinaDRAMconcurrently
| on56%ofoff-the-shelf        |     | DRAMchips.However,despitestridesin |           |                     |     |     | resistantlogic. |     |     |     |     |     |     |
| --------------------------- | --- | ---------------------------------- | --------- | ------------------- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- |
| thedirectionofmoreefficient |     |                                    | refreshes | asarowhammermitiga- |     |     |                 |     |     |     |     |     |     |
tion,HiRAisstillinitsinfancyandisnotyetavailableforusein if(matched == 0x69d61fc8)
|                  |     |     |     |     |     |     | //passwords |     | match |     |     |     |     |
| ---------------- | --- | --- | --- | --- | --- | --- | ----------- | --- | ----- | --- | --- | --- | --- |
| consumersystems. |     |     |     |     |     |     | else        |     |       |     |     |     |     |
Additionally, [47] proposes a novel and efficient Rowhammer //passwords don't match
mitigationbybuildingonexistingProbabilisticAdjacentRowAc-
|     |     |     |     |     |     |     | Additionally,thestackRowhammer |     |     |     | attackcanfurtherbepre- |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------ | --- | --- | --- | ---------------------- | --- | --- |
tivation(PARA)RowhammerdefencesbybuildingDiscreet-PARA.
ventedbyrequiringspecificpatternsforsecuritysensitvechecks
Discreet-PARAcombinesdisturbancebincounting,amechanism
soasinglebitflipwillnotresultinasecurityfailure.Forourexam-
| for managing | refresh | operations | on rows | likely | to be corrupted |     |     |     |     |     |     |     |     |
| ------------ | ------- | ---------- | ------- | ------ | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
plewiththematchedvariable,wecouldrequirethatthevariable
byRowhammer,andPARA-cache,whichisacachethatstoresthe be set to a random set of bits that are not all zeros. This would
| most recentlyaccessed |     | rows.By      | tracking | accesses | and refreshes |     |           |          |            |        |              |     |                 |
| --------------------- | --- | ------------ | -------- | -------- | ------------- | --- | --------- | -------- | ---------- | ------ | ------------ | --- | --------------- |
|                       |     |              |          |          |               |     | requirean | attacker | toflip all | bitsin | the variable |     | tothat specific |
| torows, Discreet-PARA |     | candetectand |          | mitigate | Rowhammer     | at- |           |          |            |        |              |     |                 |
patternforauthenticationwhichisfarmoredifficultthanflipping
tacks.Researcherswereabletooptimizetheserefreshandaccess
anysinglebit.Ittakesadvantageofthefactthatrowhammerisa
trackingmechanismtoreducetheperformanceoverheadfromav-
blunttoolthatisofteninprecise.
eragesfrom10.5%-6.6%to5.3%-2.6%.Still,thismitigationresultsin Considerlisting14.Thiscodeisfarmoresecurethantheprevi-
overheadthatmaynotbeidealforconsumersystems.
ousexamplesbecauseitrequiresaspecificpatterninthematched
ItwasinitiallythoughtthatErrorCorrectingCode(ECC)would
variable.Thispatternisarandomsetofbitsthatarenotallzeros.
beanamplecountermeasuretoRowhammer.However,ECCisnot
Inthiscase,theattackerwouldneedtoflipthematchedvariable
asufficientcountermeasurebecauseitcanbedefeatedwithtriple
to0110100111...,whichincludes17bitflipsinpreciselocations
bitflips[8].ECCisacommonfeatureinservers,butitisgenerally alongthevariable.Thisismoredifficultthanflippinganysingle
notavailableinconsumerDRAMs.
bitinthevariable.
10.2 Tighter,MorePreciseLogic
|                |                   |     |     |               |         |     | 10.3 DetectingRowhammerGadgets |     |     |     |     |     |     |
| -------------- | ----------------- | --- | --- | ------------- | ------- | --- | ------------------------------ | --- | --- | --- | --- | --- | --- |
| We proposeaset | ofcountermeasures |     |     | thatcanbeused | against | a   |                                |     |     |     |     |     |     |
WebelievethatRowhammergadgetsmaybeanexcellentdomain
Rowhammerattackonthestackofaprocess.Theeasiestwayto
|     |     |     |     |     |     |     | for a machine | learning | algorithm |     | to find | and detect | vulnerable |
| --- | --- | --- | --- | --- | --- | --- | ------------- | -------- | --------- | --- | ------- | ---------- | ---------- |
makeanattackmoredifficultistotightenthelogicofthecodeand
piecesofcodeusingnaturallanguageprocessing.Similarworkhas
avoidusingif-not-zeroconditionals.
beendoneusingmachinelearningtodetectSpectregadgets[43].
InthefirstexampleseeninListing 12,ifanysingle bitinthe AdatasetofRowhammergadgetscouldbederivedfromexisting
matchedvariableisflipped,thefirststatementbecomestrue.The
codebysimulatingRowhammerflipsinstackvariablesandcheck-
| Rowhammerattackisnotalwaysprecise,socheckingif |     |     |     |     | matched |     |     |     |     |     |     |     |     |
| ---------------------------------------------- | --- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
ingiftheprocessexperiencesasecurityfailure.
isnotequaltozeroallowsanattackertoflipanyofthe32bitsthat
makeupmatched,andthepasswordswillseemtomatch.Thisis
averysimilargadgettotheonewefoundinthesudoprogram.

Mayhem:TargetedCorruptionofRegisterandStackVariables ASIACCS2024,July01–05,2024,Singapore
10.4 ResponsibleDisclosure ondetectionofintrusionsandmalware,andvulnerabilityassessment,pages300–
321.Springer,2016.
Weinformedthelibraryauthorsregardingthevulnerabilitieswe
[17] DanielGruss,ClémentineMaurice,KlausWagner,andStefanMangard.Flush+
identified.SUDOauthorscommittedaseriesofpatches9tomake Flush:afastandstealthycacheattack.InInternationalConferenceonDetectionof
the library more resistant against Rowhammer by using mitiga- IntrusionsandMalware,andVulnerabilityAssessment,pages279–299.Springer,
2016.
tiondescribedinSection10.2.WehaveissuedCVE-2023-42465for [18] J.AlexHalderman,SethD.Schoen,NadiaHeninger,WilliamClarkson,William
SUDO. Paul,JosephA.Calandrino,ArielJ.Feldman,JacobAppelbaum,andEdwardW.
Felten.Lestweremember:cold-bootattacksonencryptionkeys.InCACM,2008.
[19] NishadHerathandAndersFogh. ThesearenotyourgrandDaddyscpuper-
11 ACKNOWLEDGEMENTS
formancecounters–cpuhardwareperformancecountersforsecurity.BlackHat
Briefings,2015.
Wethankouranonymousreviewersfortheirinsightfulfeedback
[20] GorkaIrazoqui,ThomasEisenbarth,andBerkSunar. MASCAT:Stoppingmi-
andSalehK.MonfaredforusefuldiscussionsonLinux.Thiswork croarchitecturalattacksbeforeexecution.IACRCryptol.ePrintArch.,2016:1196,
wassupportedbytheNationalScienceFoundationgrantCNS-2026913 2016.
[21] SaadIslam,AhmadMoghimi,IdaBruhns,MoritzKrebbel,BerkGulmezoglu,
andinpartbyagrantfromtheQatarNationalResearchFund.
ThomasEisenbarth,andBerkSunar. SPOILER:Speculativeloadhazardsboost
rowhammerandcacheattacks. In28thUSENIXSecuritySymposium(USENIX
REFERENCES Security19),pages621–637,SantaClara,CA,August2019.USENIXAssociation.
[22] MichaelKerrisk. sleep(3)—Linuxmanualpage. man7.org,2023. Linuxman-
[1] ZelalemBirhanuAweke,SalessawiFeredeYitbarek,RuiQiao,ReetuparnaDas, pages6.04.
MatthewHicks,YossiOren,andToddAustin.ANVIL:Software-basedprotection [23] YoonguKim,RossDaly,JeremieKim,ChrisFallin,JiHyeLee,DonghyukLee,
againstnext-generationrowhammerattacks.ACMSIGPLANNotices,51(4):743– ChrisWilkerson,KonradLai,andOnurMutlu. Flippingbitsinmemorywith-
755,2016. outaccessingthem:Anexperimentalstudyofdramdisturbanceerrors. ACM
[2] DanBoneh,RichardA.DeMillo,andRichardJ.Lipton. Ontheimportance SIGARCHComputerArchitectureNews,42(3):361–372,2014.
ofeliminatingerrorsincryptographiccomputations. JournalofCryptology, [24] PaulKocher,JannHorn,AndersFogh,,DanielGenkin,DanielGruss,Werner
14:101–119,2015. Haas,MikeHamburg,MoritzLipp,StefanMangard,ThomasPrescher,Michael
[3] FerdinandBrasser,LucasDavi,DavidGens,ChristopherLiebchen,andAhmad- Schwarz,andYuvalYarom.Spectreattacks:Exploitingspeculativeexecution.In
RezaSadeghi. CAn’ttouchthis:Software-onlymitigationagainstrowhammer 40thIEEESymposiumonSecurityandPrivacy(S&P’19),2019.
attackstargetingkernelmemory.In26thUSENIXSecuritySymposium(USENIX [25] AndreasKogler,JonasJuffinger,SalmanQazi,YoonguKim,MoritzLipp,Nicolas
Security17),pages117–130,Vancouver,BC,August2017.USENIXAssociation. Boichat,EricShiu,MattiasNissler,andDanielGruss.Half-double:Hammering
[4] YuCai,SaugataGhose,YixinLuo,KenMai,OnurMutlu,andErichF.Haratsch. fromthenextrowover.In31stUSENIXSecuritySymposium:USENIXSecurity’22,
Vulnerabilitiesinmlcnandflashmemoryprogramming:Experimentalanalysis, 2022.
exploits,andmitigationtechniques.2017IEEEInternationalSymposiumonHigh [26] AnilKurmus,NikolasIoannou,NikolaosPapandreou,andThomasParnell.From
PerformanceComputerArchitecture(HPCA),pages49–60,2017. randomblockcorruptiontoprivilegeescalation:Afilesystemattackvectorfor
[5] ClaudioCanella,DanielGenkin,LukasGiner,DanielGruss,MoritzLipp,Ma- rowhammer-likeattacks.InWorkshoponOffensiveTechnologies,2017.
rinaMinkin,DanielMoghimi,FrankPiessens,MichaelSchwarz,BerkSunar, [27] AndrewKwong,DanielGenkin,DanielGruss,andYuvalYarom. RAMBleed:
JoVanBulck,andYuvalYarom. Fallout:Leakingdataonmeltdown-resistant Readingbitsinmemorywithoutaccessingthem. In2020IEEESymposiumon
cpus.InProceedingsofthe2019ACMSIGSACConferenceonComputerandCom- SecurityandPrivacy(SP),pages695–711.IEEE,2020.
municationsSecurity,CCS’19,page769–784,NewYork,NY,USA,2019.Associ- [28] MoritzLipp,MichaelSchwarz,DanielGruss,ThomasPrescher,WernerHaas,
ationforComputingMachinery. AndersFogh,JannHorn,StefanMangard,PaulKocher,DanielGenkin,Yuval
[6] MarcoChiappetta,ErkaySavas,andCemalYilmaz.Realtimedetectionofcache- Yarom,andMikeHamburg.Meltdown:Readingkernelmemoryfromuserspace.
basedside-channelattacksusinghardwareperformancecounters.AppliedSoft In27thUSENIXSecuritySymposium(USENIXSecurity18),2018.
Computing,49:1162–1174,2016. [29] MoritzLipp,MichaelSchwarz,LukasRaab,LukasLamster,MisikerTadesse
[7] LucianCojocar,JeremieKim,MineshPatel,LillianTsai,StefanSaroiu,Alec Aga,ClémentineMaurice,andDanielGruss.Nethammer:Inducingrowhammer
Wolman,andOnurMutlu. Arewesusceptibletorowhammer?anend-to-end faultsthroughnetworkrequests.In2020IEEEEuropeanSymposiumonSecurity
methodologyforcloudproviders. In2020IEEESymposiumonSecurityandPri- andPrivacyWorkshops(EuroS&PW),pages710–719.IEEE,2020.
vacy(SP),pages712–728.IEEE,2020. [30] KoksalMus,YarkınDoröz,MCanerTol,KristiRahman,andBerkSunar. Jolt:
[8] LucianCojocar,KavehRazavi,CristianoGiuffrida,andHerbertBos.Exploiting Recoveringtlssigningkeysviarowhammerfaults.In2023IEEESymposiumon
correctingcodes:OntheeffectivenessofECCmemoryagainstrowhammerat- SecurityandPrivacy(SP),pages1719–1736.IEEE,2023.
tacks.In2019IEEESymposiumonSecurityandPrivacy(SP),pages55–71.IEEE, [31] OnurMutluandJeremieSKim.Rowhammer:Aretrospective.IEEETransactions
2019. onComputer-AidedDesignofIntegratedCircuitsandSystems,39(8):1555–1571,
[9] JonathanCorbet. DefendingagainstRowhammerinthekernel,October2016. 2019.
https://lwn.net/Articles/704920/. [32] MySQL. Mysql customers, 2023. Accessed on 7 February 2023.
[10] LucasDavi,ChristopherLiebchen,Ahmad-RezaSadeghi,KevinZSnow,and https://www.mysql.com/customers/.
Fabian Monrose. Isomeron: Code randomization resilient to (just-in-time) [33] NIST.Cve-2022-42961detail.Oct2022.
return-orientedprogramming.InNDSS,2015. [34] MathiasPayer.HexPADS:aplatformtodetect“stealth”attacks.InInternational
[11] FinndeRidder,PietroFrigo,EmanueleVannacci,HerbertBos,CristianoGiuf- SymposiumonEngineeringSecureSoftwareandSystems,pages138–154.Springer,
frida,andKavehRazavi.SMASH:Synchronizedmany-sidedrowhammerattacks 2016.
fromJavaScript.In30thUSENIXSecuritySymposium(USENIXSecurity21),pages [35] PeterPessl,DanielGruss,ClémentineMaurice,MichaelSchwarz,andStefan
1001–1018.USENIXAssociation,August2021. Mangard. DRAMA:ExploitingDRAMaddressingforCross-CPUattacks. In
[12] PietroFrigo,EmanueleVannacc,HasanHassan,VictorVanDerVeen,Onur 25thUSENIXSecuritySymposium(USENIXSecurity16),pages565–581,Austin,
Mutlu,CristianoGiuffrida,HerbertBos,andKavehRazavi.TRRespass:Exploit- TX,August2016.USENIXAssociation.
ingthemanysidesoftargetrowrefresh. In2020IEEESymposiumonSecurity [36] KavehRazavi,BenGras,ErikBosman,BartPreneel,CristianoGiuffrida,and
andPrivacy(SP),pages747–762.IEEE,2020. HerbertBos. Flipfengshui:Hammeringaneedleinthesoftwarestack. In
[13] BenGras,KavehRazavi,ErikBosman,HerbertBos,andCristianoGiuffrida.Aslr 25thUSENIXSecuritySymposium(USENIXSecurity16),pages1–18,Austin,TX,
ontheline:Practicalcacheattacksonthemmu. InNDSS,volume17,page26, August2016.USENIXAssociation.
2017. [37] MarkSeabornandThomasDullien. Exploitingthedramrowhammerbugto
[14] IEEE/TheOpenGroup.getchar(3p)—Linuxmanualpage.man7.org,2017.POSIX gainkernelprivileges.BlackHat,15:71,2015.
Programmer’sManual. [38] ITSolid.Db-enginesrankingofrelationaldbms,2023.Accessedon7February
[15] DanielGruss,MoritzLipp,MichaelSchwarz,DanielGenkin,JonasJuffinger, 2023.https://db-engines.com/en/ranking.
SioliO’Connell,WolfgangSchoechl,andYuvalYarom.Anotherflipinthewall [39] AkiraTakahashiandMehdiTibouchi.Degeneratefaultattacksonellipticcurve
ofrowhammerdefenses. In2018IEEESymposiumonSecurityandPrivacy(SP), parametersinopenssl. InIEEEEuropeanSymposiumonSecurityandPrivacy,
pages245–261.IEEE,2018. EuroS&P2019,Stockholm,Sweden,June17-19,2019,pages371–386.IEEE,2019.
[16] DanielGruss,ClémentineMaurice,andStefanMangard. Rowhammer.js:A [40] AndreiTatar,CristianoGiuffrida,HerbertBos,andKavehRazavi.Defeatingsoft-
remotesoftware-inducedfaultattackinjavascript. InInternationalconference waremitigationsagainstrowhammer:Asurgicalprecisionhammer.InMichael
9https://github.com/sudo-project/sudo/commit/7873f8334c8d31031f8cfa83bd97ac6029309e4f Bailey,ThorstenHolz,ManolisStamatogiannakis,andSotirisIoannidis,editors,
ResearchinAttacks,Intrusions,andDefenses,pages47–66,Cham,2018.Springer

| ASIACCS2024,July01–05,2024,Singapore |     |     |     |     |     |     |     |     | Adilettaetal. |     |
| ------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- |
InternationalPublishing.
[41] AndreiTatar,RadheshKrishnanKonoth,EliasAthanasopoulos,CristianoGiuf-
frida,HerbertBos,andKavehRazavi.Throwhammer:Rowhammerattacksover
thenetworkanddefenses.In2018USENIXAnnualTechnicalConference(USENIX
ATC18),pages213–226,Boston,MA,July2018.USENIXAssociation.
[42] YoussefTobah,AndrewKwong,IngabKang,DanielGenkin,andKangGShin.
Spechammer:Combiningspectreandrowhammerfornewspeculativeattacks.
In2022IEEESymposiumonSecurityandPrivacy(SP),pages681–698.IEEE,2022.
| [43] M.CanerTol,BerkGulmezoglu,KorayYurtseven,andBerkSunar. |     |     |     | FastSpec: |     |     |     |     |     |     |
| ----------------------------------------------------------- | --- | --- | --- | --------- | --- | --- | --- | --- | --- | --- |
ScalableGenerationandDetectionofSpectreGadgetsUsingNeuralEmbeddings.
In2021IEEEEuropeanSymposiumonSecurityandPrivacy(EuroS&P),pages616–
632.IEEE,2021.
[44] JoVanBulck,DanielMoghimi,MichaelSchwarz,MoritzLipp,MarinaMinkin,
DanielGenkin,YaromYuval,BerkSunar,DanielGruss,andFrankPiessens.LVI:
HijackingTransientExecutionthroughMicroarchitecturalLoadValueInjection.
In41thIEEESymposiumonSecurityandPrivacy(S&P’20),2020. Figure8:Histogramofpageoffsetofastackvariableinstack
[45] VictorVanDerVeen,YanickFratantonio,MartinaLindorfer,DanielGruss,Clé-
memoryoutof100Ktrials.
mentineMaurice,GiovanniVigna,HerbertBos,KavehRazavi,andCristiano
Giuffrida.Drammer:Deterministicrowhammerattacksonmobileplatforms.In
Proceedingsofthe2016ACMSIGSACconferenceoncomputerandcommunications
|     |     |     |     |     | B BYPASSINGSTACKASLR |     |     |     |     |     |
| --- | --- | --- | --- | --- | -------------------- | --- | --- | --- | --- | --- |
security,pages1675–1689,2016.
[46] StephanvanSchaik,AlyssaMilburn,SebastianÖsterlund,PietroFrigo,Giorgi
|                                                          |     |     |            |     | B.1 ASLRBackground |     |     |     |     |     |
| -------------------------------------------------------- | --- | --- | ---------- | --- | ------------------ | --- | --- | --- | --- | --- |
| Maisuradze,KavehRazavi,HerbertBos,andCristianoGiuffrida. |     |     | RIDL:Rogue |     |                    |     |     |     |     |     |
in-flightdataload.InS&P,May2019.
Address-spacelayoutrandomization(ASLR)isoftenusedasapri-
| [47] Z.Wang,W.Liu,andY.Wang. |     | Discreet-para:Rowhammerdefensewithlow |     |     |     |     |     |     |     |     |
| ---------------------------- | --- | ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
costandhighefficiency.In2021IEEE39thInternationalConferenceonComputer marydefenseagainstmemorycorruptionattacks.ASLRarranges
Design(ICCD),pages1–8.IEEE,2021.
theaddressspaceofaprocessrandomlytopreventauserfromtar-
| [48] Zane Weissman, | Thore Tiemann, | Daniel Moghimi, | Evan Custodio, | Thomas |     |     |     |     |     |     |
| ------------------- | -------------- | --------------- | -------------- | ------ | --- | --- | --- | --- | --- | --- |
Eisenbarth,andBerkSunar. Jackhammer:Efficientrowhammeronheteroge- getingaspecificareaofcode.Itissupposedtorearrangethestack,
neousfpga-cpuplatforms. IACRTransactionsonCryptographicHardwareand heap,andlibrariesofanexecutableinanon-deterministicway.In
EmbeddedSystems,2020(3):169–195,Jun.2020.
theory,ifanattackerfindsawaytocorruptthememoryitshould
| [49] YuanXiao,XiaokuanZhang,YinqianZhang,andRaduTeodorescu. |     |     |     | Onebit |     |     |     |     |     |     |
| ----------------------------------------------------------- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- |
flips,onecloudflops:Cross-VMrowhammerattacksandprivilegeescalation. nothave accessto,it shouldnotbeabletotarget anyparticular
In25thUSENIXSecuritySymposium(USENIXSecurity16),pages19–35,Austin,
areaintheprocess.
TX,August2016.USENIXAssociation.
[50] AGirayYağlikçi,AtaberkOlgun,MineshPatel,HaocongLuo,HasanHassan, ASLR has been shown to be vulnerable in the past, typically
LoisOrosa,OğuzErgin,andOnurMutlu.Hira:hiddenrowactivationforreduc- throughtheuseofsoftware-sideweakpointssuchasmemorydis-
ingrefreshlatencyofoff-the-shelfdramchips.In202255thIEEE/ACMInterna- closurevulnerabilities that reveal run-time addresses [10]. More
tionalSymposiumonMicroarchitecture(MICRO),pages815–834.IEEE,2022.
[51] KeunSooYim.Therowhammerattackinjectionmethodology.In2016IEEE35th recentattackshavealsoshownthatASLRcanbebrokenthrough
symposiumonreliabledistributedsystems(SRDS),pages1–10.IEEE,2016.
|                                             |     |                               |                            |     | the use of   | EVICT+TIME     | cache | attacks that   | can derandomize | ad- |
| ------------------------------------------- | --- | ----------------------------- | -------------------------- | --- | ------------ | -------------- | ----- | -------------- | --------------- | --- |
| [52] TianweiZhang,YinqianZhang,andRubyBLee. |     |                               | Cloudradar:Areal-timeside- |     |              |                |       |                |                 |     |
|                                             |     |                               |                            |     | dress spaces | by correlating | cache | line addresses | with page-table |     |
| channelattackdetectionsysteminclouds.       |     | InInternationalSymposiumonRe- |                            |     |              |                |       |                |                 |     |
searchinAttacks,Intrusions,andDefenses,pages118–140.Springer,2016. entries[13].Importantly,theseattacksonASLRdonotcircumvent
stackASLR,whichisimplementedintheLinuxkernelasshownin
A SPOILERTIMINGS
Listing15.StackASLRshouldrandomizethebaseaddressofthe
|     |     |     |     |     | stack resulting                                          | in random | variable | offsets as | seen in Figure | 8 to |
| --- | --- | --- | --- | --- | -------------------------------------------------------- | --------- | -------- | ---------- | -------------- | ---- |
| 900 |     |     |     |     | thepointabruteforceattackrandomlyflippingbitsinthesystem |           |          |            |                |      |
| 800 |     |     |     |     | wouldbeineffective.                                      |           |          |            |                |      |
700
Listing15:PageoffsetrandomizationforStackmemoryin
600
| elcyC kcolC |     |     |     |     | LinuxKernel |     |     |     |     |     |
| ----------- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- |
500
400
| 300 |     |     |     |     | 1 unsigned                   | long arch_align_stack(unsigned |     | long                 | sp){ |     |
| --- | --- | --- | --- | --- | ---------------------------- | ------------------------------ | --- | -------------------- | ---- | --- |
|     |     |     |     |     | 2 if (!(current->personality |                                |     | & ADDR_NO_RANDOMIZE) | &&   |     |
200
randomize_va_space)
| 100     |       |             |           |     | 3 sp     | -= get_random_int() |     | % 8192; |     |     |
| ------- | ----- | ----------- | --------- | --- | -------- | ------------------- | --- | ------- | --- | --- |
|         |       |             |           |     | 4 return | sp & ~0xf;          |     |         |     |     |
| 0 0 0.5 | 1 1.5 | 2           | 2.5 3 3.5 | 4   |          |                     |     |         |     |     |
|         |       |             |           |     | 5 }      |                     |     |         |     |     |
|         |       | Page Number |           | 104 |          |                     |     |         |     |     |
Initially,itwouldseemthatASLRmakesrunningastackattack
Figure 7: Timing peaks found by SPOILER. Equidistant difficult.However,profilingaprocesstodeterminethenumberof
peaksindicatephysicalcontinuityinmemory. baitpagesrequiredtobereleasedtothesystemreducestheentropy
significantly.
|     |     |     |     |     | Thephysical | address | issplit | into two parts;thepage |     | number |
| --- | --- | --- | --- | --- | ----------- | ------- | ------- | ---------------------- | --- | ------ |
Thetimingsthatareshownin7demonstratehowphysicalad-
dress dependencies result in more cycles because of speculative andthepageoffset.Thenumberoftotalbitsinaphysicaladdress
|                                 |     |     |     |     | iscalculatedaslog | (𝑝)where𝑝isthetotalsizeofthemainmem- |     |     |     |     |
| ------------------------------- | --- | --- | --- | --- | ----------------- | ------------------------------------ | --- | --- | --- | --- |
| loading,asdescribedinsection4.1 |     |     |     |     |                   | 2                                    |     |     |     |     |
ory.Forasystemwith8GBofmainmemory,thephysicaladdress
|     |     |     |     |     | islog (8𝐺𝐵)or33bits.Ouroperatingsystemwasalsofragment- |     |     |     |     |     |
| --- | --- | --- | --- | --- | ------------------------------------------------------ | --- | --- | --- | --- | --- |
2
ingmemoryintoindivisible4KB-sizedpages,whichcanberep-
|     |     |     |     |     | resented as | 12bits.Thismeans |     | thatin thephysical | address | ofa |
| --- | --- | --- | --- | --- | ----------- | ---------------- | --- | ------------------ | ------- | --- |

Mayhem:TargetedCorruptionofRegisterandStackVariables ASIACCS2024,July01–05,2024,Singapore
200
180
160
140
120
100
80
60
40
20
0 500 1000 1500 2000 2500 3000 3500 4000
Page Offset
segaP
tiaB
#
Tounderstandtherootcauseofthisunusualbehavior,weinves-
tigatethefollowingmethodsthatleakinformationaboutthepage
offset.
B.2 ControllingthePageOffsetwithASLR
disabled
Weinvestigatethedependencybetweenthenumberofbaitpages
andthepageoffsetofastackvariableinamorecontrolledenvi-
ronment.Wecreatethefollowingfunctionwhereabufferwitha
predefinedBUFFER_SIZEbeforeintegervariablevar.
Figure9:Therelationbetweenthenumberofbaitpagesvs.
void main(){
pageoffsetofastackvariable.Whenthepageoffsetislarge, char buffer[BUFFER_SIZE] = {0};
thenumberofbaitpagesissignificantlyhigher (shownin int var = 0;
}
red).
Notethatboththebufferandthevariablearestoredinthestack.
systemwith8GBmainmemoryand4KBpages,thefirst21bits
WedisableASLRinthesystemtomakesurewehavefullcontrolon
representthepageofthephysicaladdress,andthelast12bitsrep-
thepageoffsetofthevariable.InFigure10,wevarytheBUFFER_-
resenttheoffsetwithinthepage.
SIZEvariablefrom0to4K.Increasingthesizeofthebufferpushes
Withourbait pages attack,wecan effectively remove theen-
thevariablebackinthestackandlinearlydecreasesthepageoff-
tropyofthefirst21bitsofrandomizationbyforcingthebasead-
set. We control the page offset by varying the size of the buffer.
dress tobeplaced onaknown pagearound 45%ofthe time, ac-
Wealsoobservethenumberofrequiredbaitpageshasasudden
cordingtoourfindings.Thisleavesthelast12bitsofrandomiza-
changetogetherwiththepageoffsetofthevariable.Wespeculate
tiontodealwith.Throughexperimentation, wenoticedthatthe
thisbehavioriscausedbycrossingthepageboundarieswhilein-
entropyinthelast12bitscanbefurtherreduced.Wefoundthat
creasingtheBUFFER_SIZEandresultsinanincreaseinthenumber
thelast4bitsoftheaddressalwaysstayedthesame.Whenattack-
oftotalpagesconsumedbytheprogram.Next,weinvestigatethe
ingOpenSSL,forexample,wenoticedthatthelast4bitsalwayshad
samedependencywithASLRenabled.
avalueof0x8.Ifthevariableweareattackingisa4-bytevariable,
thenthereareonlyfourpossibilitiesforthelast4bitsofanaddress
B.3 PageFaultSideChannel
tobepotentiallyvulnerable;𝑛,𝑛+1,𝑛+2,and𝑛+3,where𝑛isthe
startingaddressofthevariable.ThisfurtherreducesourASLRen- Wefoundthatmonitoringforpagefaultsgivesusasidechannelto
tropytoamere8bits,whichcanbeeasilyexhausted.Wefounda determinetheoffsetsetbyASLR.Apagefaultwillhappenwhena
relationshipbetweenthenumberofbaitpagesrequiredtobere- processrequestsdatafromapageinmemorythatisnotcurrently
leasedbyanattackerprogramtolocateavariableintheRowham- loadedinDRAM.Whenthepagefaultoccurs,thepageneedsto
merpagecorrectlyandtheoffsetthatvariableappearswithinthat bemovedfromtheswapspaceinthestoragetoDRAM.Thereare
page.Webelievethistobeanoveldiscoverybecausepartofthe twotypesofpagefaults;majorfaultsandminorfaults.Majorfaults
intentionofASLRistorandomizethepageoffset. occurwhenapageisrequestedthatdoesnotexistinmemoryand
Wefoundthisrelationshipbyunmappingpagesinourattacker needstobebroughtbackfromtheswap space. A minor faultis
programandrecordingtheirphysicaladdressinalist,theninour
victimprogram,determiningwhereourtargetvariableappearsin 50
thelist,aswellasthepageoffsetaddressofthetargetvariable(the 48
last12bits).Whilethisrelationshipwasdifferentforeachprogram, 46
itwasclearthattherewasalwaysasmallersetofdatapointswhere 44
the number ofbait pages clearlylimited the number of possible 42
page offsets. We created agraph ofthis relationship in Figure 9. 40
We can see from the graph that if 180 bait pages were required 38
tobereleasedtomountthevictimvariableinvulnerablememory 36
34
correctly,thenthepageoffsetofthesaidvariablewouldbearound
32
4000.Likewise,ifthenumberofbaitpagesis40,itcanbeassumed 0 500 1000 1500 2000 2500 3000 3500 4000
Buffer Size [bytes]
thatthepageoffsetisgoingtobesomewherebetween0-2500.It
shouldnotbepossibletofindpatternsinpageoffsetbecauseASLR
intendstheoffsettobebasedonrandomnumbergenerationasthe
offsetismaskedbyarandomlygeneratedvalue,asseeninListing
15.
segaP
tiaB
#
4000
3500
3000
2500
2000
1500
1000
500
0
tesffO
egaP
Figure 10: The dependency between the number of bait
pages (black) and page offset (red) when ASLR is disabled.
The page offset of the variable is manually controlled by
changingthesizeofthebuffer.Thejumpinthe#baitpages
andpageoffsetoccursatthesamepoint.

ASIACCS2024,July01–05,2024,Singapore Adilettaetal.
lessperformancedegradingandoccurswhenthepageiscurrently Relaunching worksbecauseASLR willputthevariable into a
inmemoryandneedstobeswappedbackouttothedisk(usually new locationin the page the next time it runs. This means that
tofreeupspaceinDRAMforotherpages). ratherthanlookingforanewflippybitthatmightcolocatewhere
LookingatFigure4,wecanseethatiftheprocessreceives275 a flip is needed in the victim process, the victim can simply be
pagefaults(markedinred),wecanguaranteethatthelocationof relaunchedandASLRreshufflesthevariablesomewhereelse,po-
theoffsetinDRAMisgoingtobesomewherebetween200and800, tentiallyintothelocationwhereitcanbeflippedbytheflippybits
which reduces the search space and randomization of the ASLR inthepage.
offsetbitsbymorethanafactorof6.Additionally,if286pagefaults
aredetected,weknowthattheoffsetwillgenerallynotbebetween
200-800,whichalsoreducesthesearchspace.
Wearenotsurewhythissidechannelexists,butwespeculate
thattherandomizedpageoffsetthrowspagefaultswhichwecan
monitorusingperformancemonitoringbytheattacker.Itisimpor-
tanttonotethatthisperformancemonitoring,e.g.,the/usr/bin/-
timecommand,donotrequirespecialpermissiontorunandthus
arepracticaltouseinarealattack.
B.4 RemappingPagesSideChannel
Onetechniqueweusedwaspageremapping,wherewewouldun-
map𝑛 pages ofourattacker program,launchourvictimprocess,
thenremapnpagesbacktoourattackprogram.Ifweunmapped
500pages,launchedourvictimprocess,thenremapped300pages
backtoourattackerprogram,wewouldassumethatthenumber
ofbaitpagesourvictimrequiredwas200pages.Wefoundaslight
correlationinthedata,butultimately,itwastoonoisytobeuseful.
We speculatethisis because remapping pages pullsfrom unpre-
dictablepoolsofmemory,sothenumberofpagesisnotzero-sum.
B.5 ExploitingOffsetRandomization
AlthoughASLRisbuiltasasecuritymeasuretopreventmem-
ory attacks, it can be exploited to make the Rowhammer attack
morepowerful.Weproposeatechniquenamedrelaunchingtoex-
ploitASLRforRowhammer.
Theattackerfirstprofilesthememorytofindaflippybitloca-
tion in memory. In some DRAMs, these flippy locationsmay be
rare.ForsomeRowhammerenabledattacks,thatrequireaspecific
bitinthepagetobeflippy,theattackwillbecomelessviable.Inour
attack,instead,wefirstfindaflippybitinmemory,thenperform
thefollowingsteps:
(1) Afterfindingaflippybitlocation,theattackerfreesmemory
tothesystemcontainingthenumberofbaitpagesfollowed
bytheflippypage;
(2) Theattackerlaunchesthevictimprocesswhichfillsthere-
centlydeallocatedpages;
(3) TheattackerperformstheRowhammerattackonthevictim
process(not knowing ifthe flippy bit aligns withthe bits
requiredtobeflippedinthevictimprocess);
(4) Thevictimprocessendsandtheattackerprocessremapsthe
memoryusedbythevictimprocessbacktoitselfandrepeats
theattackwiththesameflippyrow.
Withthisapproach,theoretically,theattackeronlyneedstofind
asingleflippybitinthewholesystemfortheattacktowork.This
isadramaticimprovementoverotherRowhammerattackswhere
extensiveprofilingisrequired,andoftenthousandsofflipsarere-
quiredbeforeasuccessfulattack.
