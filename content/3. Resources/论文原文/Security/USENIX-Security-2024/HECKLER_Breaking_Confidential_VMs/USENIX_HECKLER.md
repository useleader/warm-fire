---
publish: true
---

HECKLER: Breaking Confidential VMs with Malicious Interrupts
BenedictSchlüter SuprajaSridhara MarkKuhne AndrinBertschi ShwetaShinde
ETHZurich
Abstract HardwareisolationandmemoryencryptioninTEEsensure
theconfidentialityandintegrityofCVMs.However,despite
Hardware-basedTrustedexecutionenvironments(TEEs)offer
being untrusted, the privileged software components such
anisolationgranularityofvirtualmachineabstraction.They
asthehypervisorremainresponsibleforresourceallocation
provideconfidentialVMs(CVMs)thathostsecurity-sensitive
and virtualization management. As a result, it’s crucial to
codeanddata.AMDSEV-SNPandIntelTDXenableCVMs
reconsiderhowtheseuntrustedcomponentsinteractwiththe
andarenowavailableon popularcloudplatforms. Theun-
CVMs.Weexamineonesuchclassofinterfaces,namelythe
trusted hypervisor in these settings is in control of several
interruptmanagementthatisunderthehypervisor’scontrol.
resourcemanagementandconfigurationtasks,includingin-
InthispaperwepresentHECKLER,anewsoftware-based
terrupts. We present HECKLER, a new attack wherein the
attackthatbreakstheconfidentialityandintegrityofCVMs
hypervisorinjectsmaliciousnon-timerinterruptstobreakthe
onAMDSEV-SNPandIntelTDX.HECKLERleveragesthe
confidentialityandintegrityofCVMs.Ourinsightistouse
untrustedhypervisor’sabilitytoinjectcontrolledinterrupts
theinterrupthandlersthathaveglobaleffects,suchthatwe
intothevictimCVMatpointsofitschoice.SincetheCVMs
can manipulate a CVM’s registerstates to change the data
runafull-fledgedtrustedoperatingsystem,ithasvalidhan-
andcontrolflow.WithAMDSEV-SNPandIntelTDX,we
dlersforseveralinterrupts.Thus,unbeknownsttoitself,the
demonstrateHECKLERonOpenSSHandsudotobypassau-
victim CVM starts executing the interrupt handlers corre-
thentication.OnAMDSEV-SNPwebreakexecutionintegrity
spondingtotheinterruptinjectedbythehypervisor.However,
ofC,Java,andJuliaapplicationsthatperformstatisticaland
unliketimerinterruptsthatarewidelyusedforside-channel
text analysis. We explain the gaps in current defenses and
attacks [54–56,62] because of their effects on cache and
outlineguidelinesforfuturedefenses.
micro-architecturalstates,theCVMhashandlerschangereg-
istersandglobalstatethusimpactingthesubsequentexecu-
1 Introduction tion. Thus bysimplyinjecting interrupts,the hypervisoris
abletochangethevictimVM’sdataandcontrolflow.
Hardware-basedtrustedexecutionenvironments(TEEs)flip HECKLER is part of a largerfamily of attacks where an
the conventional trust mode. They designate the cloud ser- privilegedattackersendsmaliciousnotificationstothevictim
viceproviderandprivilegedsoftwaresuchasthehypervisor runninginaTEE.WecointhetermAhoitorefertothisclass
as untrusted entities. Recent TEEs lean towards a virtual ofattacks.1Previousstudiesthatexploittimerinterruptsand
machineabstractionforisolationgranularitytoprovidecon- page faults fall within the category of Ahoi attacks—they
fidentialVMs(CVMs)thathostsecurity-sensitivecodeand producemaliciousinterruptstoallowtheattackertomonitor
data.AMDSecureEncryptedVirtualization-SecureNested side-effectslikecacheandtiming.Unlikethesepriorinstances
Paging(SEV-SNP)andIntelTrustDomainExtensions(TDX) ofAhoi,HECKLERgeneratesinterruptsthatgobeyondside-
arethetwomainextensionsofferedcurrentlyfromhardware effects;ittargetsexpliciteffecthandlerexecutionthatdirectly
providers[2,35],whileArmConfidentialComputingArchi- modifiesregistersi.e.,theCVM’sglobalstate.
tecture (CCA) is anticipated to be in production in the fu- Findings. Weanalyzethehypervisor’sinterruptinjectionbe-
ture[7].CVMshavereceivedwide-scaleadoptionascloud havioronAMDSEV-SNPandIntelTDX.Wefindthatbothof
confidentialcomputinghostedbymajorcloudproviderssuch themforwardsome,ifnotall,interruptstothevictimCVMs.
asGoogleCloud,MicrosoftAzure,AlibabaCloud,andIBM Notably,both of them allow the attackerto inject int0x80
Cloud[1,9,28,29,33].
1Ahoiisasignalwordtocallashiporboat.Itisalsoananagramof
0Thisistheauthor’sversionoftheUSENIXSecurity2024paper. Iago[17]witheditdistanceofone.
1
4202
rpA
4
]RC.sc[
1v78330.4042:viXra

oncoresexecutingCVMs.Asaneffect,theCVMexecutes inprevalentservicesandworkloadstypicallyhostedin
thecorrespondinghandleronbehalfofauser-spaceprocess cloud-basedCVMs.Weinvokeandchainthesegadgets
(e.g.,statisticalanalysis,userauthentication,daemons)thatis usingcustomorchestrationtechniques.
currentlyexecutingonthecore.Worseyet,aspertheseman-
• Proof-of-conceptExploits.WeshowthatourAMDSEV-
ticsofint0x80,thehandlertreatsthecurrentregisterstate
SNPandIntelTDXexploitscanbypassOpenSSHand
setupbytheprocessassyscallnumber(rax)andinputargs
sudo;ourAMDSEV-SNPexploitscanbreakstatistical
(rbx,rcx,rdx)forthesystemcall.Theguestkernelinthe
andtextanalysisforAMDSEV-SNP.Thisdemonstrates
CVM,completely unaware thatthe hypervisorandnotthe thatHECKLERbreakstheintegrityandconfidentiality
| process invoked |     | this handler,executes |     |     | the system |     | call and |     |     |     |     |     |     |
| --------------- | --- | --------------------- | --- | --- | ---------- | --- | -------- | --- | --- | --- | --- | --- | --- |
guaranteesofferedbythesestate-of-the-artTEEs.
| returns the | resultofthe |     | system | callbackto |     | the process | by  |     |     |     |     |     |     |
| ----------- | ----------- | --- | ------ | ---------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- |
updatingitsrax.HECKLERabusesthisbehaviortooperate Disclosure. WeinformedIntelandAMDaboutint0x80on
asagadgetthatchangesthevictimprograms’rax.Further, 27and28September2023respectively.WeupdatedAMDon
AMDSEV-SNPallowstheattackertoinjectotherinterrupts 14October2023aboutourfindingsforotherinterruptsand
suchasint0x0andmanymore.Someoftheseinterruptsare ouranalysisoftheirdefenses.HECKLERistrackedundertwo
| presented | as signals | to  | the user | program. | We  | find | that the |     |     |     |     |     |     |
| --------- | ---------- | --- | -------- | -------- | --- | ---- | -------- | --- | --- | --- | --- | --- | --- |
CVEs:CVE-2024-25744forint0x80wasmitigatedwitha
application-specifichandlerforthesesignalscanhaveglobal kernelpatchforSEV-SNPandTDX[51].CVE-2024-25743
sideeffects.Forexample,scientificcalculationshavehandlers forotherinterruptsremainsunmitigatedforAMDon6March
toconverttheoperandsoffaultinginstructions(e.g.,thede-
2024atthetimeofthewriting.
nominatorinadivzissettoaNaN)tocapturespecificnotions HECKLERtoolingandPoCexploitsareopen-sourceat:
(e.g.,∞,-∞).HECKLERchangesthisbehaviorintoagadget https://ahoi-attacks.github.io/heckler
| to convert | particular | program |     | variables | (e.g.,to | NaN) | and |     |     |     |     |     |     |
| ---------- | ---------- | ------- | --- | --------- | -------- | ---- | --- | --- | --- | --- | --- | --- | --- |
continueexecution.Lastly,wecanchaingadgetsbyinjecting
|     |     |     |     |     |     |     |     | 2 Overview |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- |
multipleinterruptsatselectivelocationsofvictim’sexecution
tochangemorethanonedataandcontrolflow.
Hardware-basedtrustedexecutionenvironmentsprovidean
| Orchestrating |     | HECKLER. | End-to-end |     | exploits | built | with |     |     |     |     |     |     |
| ------------- | --- | -------- | ---------- | --- | -------- | ----- | ---- | --- | --- | --- | --- | --- | --- |
abstractiontoexecutecodeanddata,suchthatitsconfiden-
HECKLERrequireinjectinginterruptsattargetedexecution
tialityandintegrityispreservedeveninthepresenceofprivi-
| points in | the victim | programs |     | toinduce | effects | broughton |     |     |     |     |     |     |     |
| --------- | ---------- | -------- | --- | -------- | ------- | --------- | --- | --- | --- | --- | --- | --- | --- |
legedsoftware.AMDSecureEncryptedVirtualization-Secure
byourgadgets.Specifically,weneedtoknowtheexactcore
NestedPaging(AMDSEV-SNP),AMDSecureEncrypted
| on which | the user | program | executes |     | inside | the CVM, | the |     |     |     |     |     |     |
| -------- | -------- | ------- | -------- | --- | ------ | -------- | --- | --- | --- | --- | --- | --- | --- |
Virtualization-EncryptedState(SEV-ES),andIntelTrustDo-
guestphysicaladdressofthepointofgadgetinjection,and
mainExtensions(IntelTDX)provideaVM-levelabstraction
themomentwhentheprogramreachesthepointofinterestin
calledconfidentialVMs(CVMs).FortheseTEEabstractions,
itsexecution.ForAMDSEV-SNP,weuseseveralheuristics
theuntrustedprivilegedhypervisorprovisionstheexecution
particulartoourtargetprogramsbasedontheinformationwe
resources(e.g.,CPUandmemory)forVMs.Thehardwareen-
cangleanaboutitsexecution(e.g.,pagefaults).Wemaximize
suresexecutionandmemoryisolationsuchthattheuntrusted
thisbyleveragingauxiliaryinformationleakedbyobservable
softwarecannotcompromisetheCVM.
behaviordespiteencryptionofCVMstate(e.g.,orderofpage
Notably,theuntrustedhypervisorprovidesvirtualization
accesses,executioninsharedlibraries)[45,60].
|               |                                      |     |     |     |     |     |     | abstractions | such | as interrupt | routing | to  | CVMs. Thus,the |
| ------------- | ------------------------------------ | --- | --- | --- | --- | --- | --- | ------------ | ---- | ------------ | ------- | --- | -------------- |
| Implications. | WeusetheHECKLERgadgetstoalterthedata |     |     |     |     |     |     |              |      |              |         |     |                |
attackercanabusethisinterfacetoinjectnon-genuine(e.g.,
andcontrolflowoffivecase-studiestobreakconfidentiality
wronginterruptnumber)andunexpectedinterrupts(e.g.,at
andintegrityofCVMs.First,onAMDSEV-SNPandIntel
thewronginstruction),i.e.,maliciousinterruptsintothetarget.
| TDX,we | bypass | the authentication |     | in  | OpenSSH | andsudo, |     |     |     |     |     |     |     |
| ------ | ------ | ------------------ | --- | --- | ------- | -------- | --- | --- | --- | --- | --- | --- | --- |
Physicaltimers,themostwidely-studiedinterrupt,havebeen
thusallowingthehypervisortogaincompleterootaccessto
|     |     |     |     |     |     |     |     | shown to | break | the confidentiality |     | of TEEs | by amplifying |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ----- | ------------------- | --- | ------- | ------------- |
theCVM.Next,webreakexecutionintegrityofAMDSEV-
side-channelattacks[54].However,otherinterruptshavere-
| SNP by | altering | the results | of  | statistical | and | text | analysis |     |     |     |     |     |     |
| ------ | -------- | ----------- | --- | ----------- | --- | ---- | -------- | --- | --- | --- | --- | --- | --- |
ceivedlittletonoattention,becausetheyareassumedtonever
inC,Java,andJulia.Lastly,wediscusstheeffectivenessof
|     |     |     |     |     |     |     |     | explicitly | affectthe | victim’s | execution | beyondside-effects |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --------- | -------- | --------- | ------------------ | --- |
existingdefensesofferedbyAMDSEV-SNPandshowthat
thatcanbegleanedviaside-channels.
theyareinsufficient.Wedevelopkernel-patchesforIntelTDX
tostopgaptheeffectsofourint0x80gadget.
|                |         |                                       |           |          |     |       |        | 2.1 InterruptDeliverytoCVMs |              |     |        |          |                  |
| -------------- | ------- | ------------------------------------- | --------- | -------- | --- | ----- | ------ | --------------------------- | ------------ | --- | ------ | -------- | ---------------- |
| Contributions. |         | Wemakethefollowingnovelcontributions: |           |          |     |       |        |                             |              |     |        |          |                  |
| • Novel        | Attack. | We                                    | introduce | HECKLER, |     | a new | attack |                             |              |     |        |          |                  |
|                |         |                                       |           |          |     |       |        | The guest                   | OS executing |     | inside | the CVMs | relies on inter- |
whereinahypervisorinjectsmaliciousinterruptstotrig- ruptsforitsoperation(e.g.,theLinuxkernelrequirestimer
| ger | handlers | that change |     | the data | and control |     | flow of |            |                  |     |                   |     |                |
| --- | -------- | ----------- | --- | -------- | ----------- | --- | ------- | ---------- | ---------------- | --- | ----------------- | --- | -------------- |
|     |          |             |     |          |             |     |         | interrupts | for scheduling). |     | Therefore,similar |     | to traditional |
victimCVMs.
virtualizationinnon-confidentialexecution,thehypervisor
• Gadgets&Chaining.Weidentifyseveralcrucialgadgets hastovirtualizetheinterruptmanagementanddeliverytothe
2

|     |     |     |     | Reg. state: | Userspace: |     | Kernel space: |
| --- | --- | --- | --- | ----------- | ---------- | --- | ------------- |
Guest:
3
|     | handle_interrupt_#(): |     |     |     | 1. # returns 0 if auth fails |     |     |
| --- | --------------------- | --- | --- | --- | ---------------------------- | --- | --- |
|     |   ...                 |     |     |     | 2. <mm_answer_authpassword>: |     |     |
|     |   do_ack()            |     |     |     | 3.   ...                     |     |     |
4.   call <auth_password>
|     |   ... | 4   |     | eax =0 |     |     | handle_int0x80: |
| --- | ----- | --- | --- | ------ | --- | --- | --------------- |
  call syscall_0()
|     |     |     |     | eax!=0 | 5.   test eax, eax |     |     |
| --- | --- | --- | --- | ------ | ------------------ | --- | --- |
vCPU
|     |     |     | IC  |     | 6.   setne bpl |     |     |
| --- | --- | --- | --- | --- | -------------- | --- | --- |
7.   mov r14,ebp
8.   ...
|     | Hypervisor: |     |     | Hypervisor | 9.   mov eax,r14 |     |     |
| --- | ----------- | --- | --- | ---------- | ---------------- | --- | --- |
|     |             | 1   |     | injects    | 10.   ret        |     |     |
int0x80
do_interrupt():
  ...
  fwd_virt_interrupt() Figure 2: Inject int 0x80 for OpenSSH authentication.
|     |   ... | 2   |     |     |     |     |     |
| --- | ----- | --- | --- | --- | --- | --- | --- |
mm_answer_authpasswordisinvokedduringsshauthentica-
tion.Itreturns0whenauthenticationfails.Amaliciousint
0x80triggersacalltothesyscall0handlerwhichsetseaxto
Figure1:VirtualizedinterruptforCVMs.Solidarrows(⃝1, anon-zerovaluewhenauth_passwordreturns,resultingin
⃝3):assertedinterruptlines;dottedarrows(⃝2,⃝4):memory- asuccessfulauthentication.
mappedwrite.Theinterruptcontroller(IC)deliversaphysical
| interrupt | to the hypervisor | ⃝1. The hypervisor | writes to a |     |     |     |     |
| --------- | ----------------- | ------------------ | ----------- | --- | --- | --- | --- |
memory-mappedregionofmemory⃝2 thatemulatesavirtual
InterruptController(vIC)forthevCPUtoforwardthevirtual
interrupt⃝3.TheOSwritestoamemory-mappedregisterin
thevICtoacknowledgetheinterrupt⃝4.
|     |     |     |     | CVM mayhostthis | process | to allowtrustedusers | to login |
| --- | --- | --- | --- | --------------- | ------- | -------------------- | -------- |
CVMs.Todoso,thehypervisorhooksonallphysicalinter- andmanagetheconfidentialservices. TheSSHauthentica-
ruptsintheinterruptcontroller. Fig.1showsthismechanism tionroutineinsshdinvokesthemm_answer_authpassword
|     |     |     |     | function | to check the user’s | credentials. | If authentication |
| --- | --- | --- | --- | -------- | ------------------- | ------------ | ----------------- |
atahigh-level.Foreveryinterrupt,thehypervisordetermines
which VM the interrupt should be routed to,based on the fails,the function returns 0. The disassembly of this func-
CPU-to-vCPU mapping it maintains. Then,the hypervisor tionshowsthatthereturnvalueofauth_passwordisstored
forwards the virtual interrupt to the vCPU. The guest OS in the eax register (see Lines 5-10 in Fig. 2). Further,
oftheCVMservicesthevirtualinterrupt.Finally,theguest the caller of mm_answer_authpassword checks if the re-
|     |     |     |     | turn value | is non-zero, | and if so, allows | the user to login. |
| --- | --- | --- | --- | ---------- | ------------ | ----------------- | ------------------ |
OSacknowledgestheinterruptintheinterruptservicerou-
tine (ISR). The SEV and TDX hardware implementations Consider the case where the attacker is trying to log into
andhardenedguestLinuximages(calledenlightenedguest the CVM. Since it does not have the correct user creden-
tials,thereturnvalueofauth_passwordandconsequently
| OS) attempt | to limit the | interfaces that a CVM | exposes to |     |     |     |     |
| ----------- | ------------ | --------------------- | ---------- | --- | --- | --- | --- |
theuntrustedhypervisor.However,ouranalysisshowsthat mm_answer_authpasswordwillalwaysbe0.However,ifthe
the hypervisor is still able to inject certain or all types of attackercanchangeeaxfromzerotoanon-zerovalue,then
interrupts(seeSec.3.2forresults). thecallerofmm_answer_authpasswordwilllettheattacker
login,despiteusingwrongcredentials.Fromamalicioushy-
2.2 HECKLER Attack pervisor’sperspective,ifitinjectsanint0x80rightafterthe
returnofauth_password,itcanindeedchangethevalueof
ThehypervisorcanarbitrarilyinjectinterruptstotheCVMs. eax before itis usedbymm_answer_authpassword. Then,
Such interrupts cause the guest OS to execute its interrupt mm_answer_authpasswordreturnsanon-zerovaluetothe
serviceroutines(ISRs)whichcanhaveside-effectsthatan caller.Theonlythingthatremainsistotriggerint0x80such
thatitreturnssomeothernon-zerovalueineax.Ifwetake
attackercanexploit.Forexample,Linuxusesinterruptnum-
ber 0x80 for legacy 32-bit system calls on x86. Asserting acloserlookatthepointofinterruptinjection,eaxissetto
interrupt0x80triggersthecorrespondingISR.TheISRreads 0 by the function auth_password. If a malicious hypervi-
registereaxandexecutesthesystemcall. Further,itstores sorinjectsanint0x80atthispoint,ittriggerstheexecution
the result of the system call in the eax register. Note that ofthehandleronbehalfofthesshdprocess.Thisresultsin
thissystemcallinterfaceonlyupdatestheeaxregister.All executing system call number 0. In the Linux kernel, this
otherregistersarerestoredbythekernelbeforereturningto correspondstotherestartsystemcallwhichshouldalways
theuser-space.Therefore,amalicioushypervisorcaninject beinvokedfromwithinthekernel.Sinceweinvokeitfrom
interrupt0x80andchangethevaluestoredineax. theuser-space,thekernelreturnsanEINTRerror(−4,i.e.,a
AttackingOpenSSH. WeconsidertheOpenSSHapplication non-zerovalue)ineax.Insummary,thehypervisorusesthe
executing in user-space that runs a server process sshd. A interruptinjectionprimitivetogainaccesstotheCVM.
3

| 3 MaliciousInterrupts |     |     |     |     | T Benign |     | S   | S   |     | S S   |     |
| --------------------- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | ----- | --- |
|                       |     |     |     |     |          |     | 0   | i   |     | j     | n   |
|                       |     |     |     |     | T        |     | S   | S   |     | S' S' |     |
HECKLERleveragestheeffectsofinterrupthandlersonuser- Malicious 0 i j n
SIGFPE
levelapplications,suchthattheattackercanaltertheirbenign
S'
| behaviortodoitsbidding.Apartfromtheint0x80handlerwe |     |     |     |     |     |     |     |     |     | i   |     |
| --------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
usedinourmotivatingexample,wesystematicallyanalyze Figure3:T andT representtracesforbenignand
|     |     |     |     |     |     | Benign | Malicious |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ------ | --------- | --- | --- | --- | --- |
otherinterruptsandtheirpotentialuseinHECKLER. maliciousexecutionofPunderinputI.Thisleadstotraces
,S,S′,S′,...,S′
ThreatModel. Weoperateinthestandardthreatmodelof S ,S,S i j ,...,S n andS i toproduceoutputsO
|     |     |     |     | 0   |     |     | 0   | i j | n   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
confidentialVMsprovidedbyIntelTDXandAMDSEV-SNP. andO′respectively.Theattackerinjectsint0x0whenPisin
TheuntrustedhypervisorloadstheCVMimageinmemory stateS.ThisinducesastateS′:S[mem|mem[a](cid:55)→1],where
|     |     |     |     |     | i   |     |     |     | i   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
i
and controls the initial configurations. Remote attestation thememorythatholdsvariablea(i.e.,mem[a])issetto1.
measurestheCVM’sinitialmemorybeforeinitiatingtheboot
up.ThesoftwareexecutinginsidetheCVM(guestOS,user
applications,trusted modules for TEEs) is included in the Explicit Effect Handlers. If a program P incurs a fault,
TCB.Asforconfigurations,thespecificationsforTDXand interrupt, exception, or signal during its benign execution,
SEV-SNPoutlinecertaininitialstatethatthehypervisorhas thenthesystemexecutesahandlereitherintheguestkernel
tosetup(e.g.,numberofvCPUs,supportedhardwarefeatures, oruserspaceviaanapplication-registeredhandler.Thetrace
memorysize).Thehardwarechecksthisandonlyentersthe T capturesitgracefully.Forkernelhandlers,theydonotaffect
CVMifthesetupiscorrect.Thehardwarezeroesoutcertain theprogramandhencearenotaccountedforinthetrace.If
values(e.g.,certaingeneral-purposeregisters)beforeexiting theprogramexecutesahandlertoterminatetheprogram,that
iscapturedbythestatewiththelaststatebeingprogramexit.
theCVM.Duringexecution,SEV-SNPandTDXencryptand
integrity protectthe VM pages. Further,they protectregis- Moreimportantly,handlersthatupdatetheprogramstateand
terstateandchecksomecontrolandcommunicationspages continueexecutionarealsocapturedbythenotionofstates.
(e.g.,VirtualMachineControlBlock)thataresharedwiththe Forexample,consideraprogramwithacustomfloatingpoint
hypervisor. The hypervisor is still expected to manage the errorhandlerthatroundsoffthevaluetothenearestinteger,
|     |     |     |     | say | 1. When | the | program | executing | on  | input I is | in state |
| --- | --- | --- | --- | --- | ------- | --- | ------- | --------- | --- | ---------- | -------- |
CVMsbyallocatingphysicalpagesandschedulingvCPUs.
Thisincludesinjectinginterruptsthroughdifferentinterfaces S i ,itreceivesaSIGFPEforanoperationonvariableathat
such that the CVM can continue to perform its tasks (e.g., overflows.Theprogramexecutesthehandlerthatconvertsthe
problematicvariablefromatoa′,thuschangingthestatefrom
virtioupdates)andtonotifytheCVMaboutcriticalinterrupts
(e.g.,virtualtimers).Wenotethatthespecificprotectionsof S toS a′.Werefertosuchhandlers,thateffectastatechange,
a
asexpliciteffecthandlers.But,iftheprogramreceivesatimer
statesharedbetweenthehypervisorandtheCVMvaryfor
AMDSEV-ES,AMDSEV-SNP,andIntelTDX. interruptthentheprogramstatesstayunaffected.
Scope. IthasbeenshownthatattackingAMDSEV-SNPis Inducing Malicious State Transitions. The attacker has
morechallengingthanattackingAMDSEV-ES[2].Thisis thecapabilitytoinjectarbitraryinterruptsintotheCVMto
mainly because SEV-ES does not provide integrity protec- invokethecorrespondinghandlers.Forexample,considera
|            |                  |               |            | benignexecutionofprogramP.Attimet |     |     |     |     |     | ,itisinstateS | and |
| ---------- | ---------------- | ------------- | ---------- | --------------------------------- | --- | --- | --- | --- | --- | ------------- | --- |
| tion [59]. | We leave attacks | on AMD SEV-ES | outofscope |                                   |     |     |     |     |     | i             | i   |
forthispaperandinsteadfocusonAMDSEV-SNP,withthe changestoS att .However,underamaliciousexecution,
|     |     |     |     |     |     | j   | i+1 |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
expectationthatiftheattacksworkonSEV-SNP,theywill attimet ,the attackersends an int0x0 to the VM’s vCPU
i
workonSEV-ESaswell. thatisexecutingPwhoreceivesaSIGFPE.P’shandlerwill
|     |     |     |     | executeatt |     | ,thusinducingamaliciousstatetransitionfrom |     |     |     |     |     |
| --- | --- | --- | --- | ---------- | --- | ------------------------------------------ | --- | --- | --- | --- | --- |
i+1
|     |     |     |     | S   | to S′. | If we consider |     | our above | described | handler | that |
| --- | --- | --- | --- | --- | ------ | -------------- | --- | --------- | --------- | ------- | ---- |
i j
3.1 Trace-basedReasoning
setsvariableato1onSIGFPE,theattackerhassuccessfully
|                                                       |     |     |     | managed |     | to achieve | a state | transition | from | S to | S where |
| ----------------------------------------------------- | --- | --- | --- | ------- | --- | ---------- | ------- | ---------- | ---- | ---- | ------- |
| Ourgoalistoidentifyinterrupthandlersthat,whenexecuted |     |     |     |         |     |            |         |            |      | i    | j′      |
atarbitrarypointsduringavictimprogramexecution,induce mem[a](cid:55)→1.Worseyet,sincethehandlerresumesexecution
oftheprogram,theattackercantimetheinterruptsuchthatthe
changesthatimpacttheapplication.Tocapturethissystemat-
subsequentprogramlogicusesthemodifiedstatevariables,
ically,weintroducethenotionoftracesasdefinedbelow.
ainourexample,thusleadingtoadifferentdataorcontrol
Trace. ConsideragivenprogramPandaninputIthatpro-
flowandtrace(seeFig.3).
| ducesoutputO.ThenprogramtraceT |     |     | (I,O)isasequence |     |     |     |     |     |     |     |     |
| ------------------------------ | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- | --- | --- |
P
| ofstatesS | ,...,S ,whereS | istheprogramstatethatcaptures |     |     |     |     |     |     |     |     |     |
| --------- | -------------- | ----------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|           | 1 n            | i                             |     |     |     |     |     |     |     |     |     |
registersandvirtualmemoryattimet .Wecaptureexplicit 3.2 DetectedExplicitEffectHandlers
i
| inputs as | well as environment | variables | in I,and our state |     |     |     |     |     |     |     |     |
| --------- | ------------------- | --------- | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- |
captures the register states and virtual memory of the pro- Wefirstanalyzethehypervisor’sabilitytoinjectinterrupts
cess.NotethatforagivenP,I,O,itstraceT (I,O)isalways intotheCVM,bothonIntelTDXandAMDSEV-SNP.For
P
| deterministic. |     |     |     | this,weconductasimpletestonAMDSEV-SNPandIntel |     |     |     |     |     |     |     |
| -------------- | --- | --- | --- | --------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
4

| TDX machines | (see | Sec. 8 | for CPU and software | details). |     | User |     |     |
| ------------ | ---- | ------ | -------------------- | --------- | --- | ---- | --- | --- |
Application
| Weenumeratetheinterruptsfrom0-255,thevalidrangeof |     |     |     |     |     | space |     |     |
| ------------------------------------------------- | --- | --- | --- | --- | --- | ----- | --- | --- |
interruptsthataVMcanreceive.Weinjecttheminourvictim
|             |           |        |                 |             |     | eax = #n |                | eax = |
| ----------- | --------- | ------ | --------------- | ----------- | --- | -------- | -------------- | ----- |
| application | executing | inside | the CVM via the | hypervisor- |     |          |                |       |
|             |           |        |                 |             |     | int 0x80 | syscall result |       |
providedinterface.Then,weuse2mainobservationsregard-
ingthex86architecturetodetectexpliciteventhandlersfor handle_int0x80:
interrupts:(a)ithasanexplicitinstructionthatusestheinter- Kernel   /* read eax for #n */
ruptnumber128(i.e.,int 0x80)toperformlegacysystem space   call syscall_#n()
  /* store result in eax */
calls,and(b)theLinuxkernelmapsinterruptstosignalsthat
aredeliveredtouser-spaceapplications.First,wetestifint
0x80isdeliveredtotheCVMonbothAMDSEV-SNPand Figure4: Forint0x80,theLinuxkernelexecutesasystem
IntelTDXmachineswheninjectedfromthehypervisor.We
callcorrespondingtothenumber(#n)storedineaxbythe
seethattheLinuxkernel’sint0x80handleralwaysreturns application. When returning to the application, the kernel
theresultofthelegacysystemcallintheeaxregister.Further, storestheresultofthesystemcallintheeaxregister.
thedifferentsystemcallhandlersconditionallyreadebx,ecx,
edx,esi,andediregisters.
Next,todetectifinterruptsfromthehypervisoraredeliv- basedonthevalueineaxanattackercanusethisinterfaceto
eredassignalstotheuserapplication,wewriteaCapplication executearbitrarysystemcallstoattackthevictimCVM(e.g.,
thatregistershandlersforallsignalsandwaitsinabusyloop. changepagepermissions,copymemory).2
With this setup,we inject all interrupts to the CVM. Fora Example. Consider an application that stores a secret on
given interrupt,if the CVM has a valid handler registered the stack (ebp-4) and accesses shared memory in the non-
wecanobserveitsimpact,ifany,ontheapplication.Wesee secure region (e.g., for communication with a non-secure
that,formostinterrupts,theLinuxkernelusesadefaulthan- VM).Anattackercanusetheint0x80toleakthissecretby
dlerthatacknowledgestheinterruptinthekernelandhasno triggeringthewritesystemcall.TheLinuxkernelexecutes
explicit effecton the application. Next,we summarize our thewritesystemcallintheint0x80handler(seeFig.4)when
specificfindingsforinterruptsthatimpactedtheapplications. eaxissetto4.Then,withtherightparameters,suchacall
SEV. Ourexperiments showthatallinterrupts were deliv- writesthesecrettothehypervisoraccessiblesharedmemory.
eredtotheCVMandhandledbytheguestLinuxkernel.We Specifically,thewritesystemcalltakes3parameters;(fd)a
observe that int 0x80 is delivered to the CVM and always file descriptor to write to in ebx,(buf) the address to read
noticeably impacts the user application. Further,the guest frominecx,and(count)thenumberofbytestoreadinedx.
Linux kernel delivers 11 interrupts as a signal to the user- Therefore,weneedanapplicationthathasagadgetasshown
spaceapplication.Therefore,these12interruptshaveexplicit inthecodesnippetbelow:
effectsontheapplication.
|     |     |     |     |     | 1 mov | eax , 4 | %   |     |
| --- | --- | --- | --- | --- | ----- | ------- | --- | --- |
TDX. Allinterruptsbelow31weredroppedbythehardware
|     |     |     |     |     | 2 mov | ebx ... | %   |     |
| --- | --- | --- | --- | --- | ----- | ------- | --- | --- |
andneverevendeliveredtotheguestVM.Theonlyinterrupt
|     |     |     |     |     | mov | ecx, [ebp - 4] | %   |     |
| --- | --- | --- | --- | --- | --- | -------------- | --- | --- |
3
thatwasselectivelyallowedinthisrangewasanNMI.For mov edx, 8 %
4
| interruptsabove31thatreachedtheguestVM,onlyint0x80 |     |     |     |     | ... |     |     |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
5
noticeablyimpactedtheapplication.
Now,ifthehypervisorinjectsint0x80online5,thekernel
intheCVMwillexecutethewritesystemcallandleakthe
| 4 HECKLER | Gadgets |     |     |     |     |     |     |     |
| --------- | ------- | --- | --- | --- | --- | --- | --- | --- |
secretinecxtothesharedmemoryregioninebx.Notethat,
|     |     |     |     |     | this program | never executes | the int | 0x80 instruction. So, |
| --- | --- | --- | --- | --- | ------------ | -------------- | ------- | --------------------- |
Next,wedetailparticularexpliciteffecthandlerswedetected
|     |     |     |     |     | the attacker’s | int0x80 | injection introduces | a new state S a′, |
| --- | --- | --- | --- | --- | -------------- | ------- | -------------------- | ----------------- |
andtheirexacteffects.WerefertohandlercodeasaHECK-
wherea′capturestheresultofexecutingtheint0x80handler.
LERgadget,inspiredbymemorycorruptionattacks[32,49].
|     |     |     |     |     | ScopeofSyscalls&Registers.               |     | Theattackerhasachoice |           |
| --- | --- | --- | --- | --- | ---------------------------------------- | --- | --------------------- | --------- |
|     |     |     |     |     | ofinvokingallsyscallsbyinjectingint0x80. |     |                       | Asshownin |
4.1 SyscallsfromUserspace thewritesyscallexample,theattackerneedstohaveprecise
argumentsingeneralpurposeregisters:eaxshouldholdthe
Linuxusesint0x80forlegacysystemcallsasshowninFig.4.
|     |     |     |     |     | correct | syscall number | and ebx,ecx,and | edx should hold |
| --- | --- | --- | --- | --- | ------- | -------------- | --------------- | --------------- |
Assertingint0x80triggersthecorrespondingISRintheker-
|           |             |     |                       |         | the correctsyscallarguments. |     | Then,depending | on register |
| --------- | ----------- | --- | --------------------- | ------- | ---------------------------- | --- | -------------- | ----------- |
| nel space | of the CVM. | The | ISR reads registereax | and ex- |                              |     |                |             |
states,anattackercanchangeeaxandmemory(arguments
| ecutes the | corresponding | system | call. Further,it | stores the |     |     |     |     |
| ---------- | ------------- | ------ | ---------------- | ---------- | --- | --- | --- | --- |
passedbyreference)withsyscalls.Identifyingcodelocations
resultofthesystemcallintheeaxregister.Therefore,ama-
inapplicationsthatsatisfythisrequirement,ifnotimpossi-
licioushypervisorcaninjectint0x80andarbitrarilychange
the value stored in eax at any time (see Sec. 2.2). Further, 2int0x80instructioncanbeexecutedin64and32-bitbinaries.
5

ble,ischallenging.Toreducethesearchspace,welimitour (a) OpenSSH (b) sudo
analysistosyscallsthatonlydependoneax.Weanalyze328
syscallsandfind40syscallsonlytakeeaxasanargumentand // returns !0 on auth success int pam_authenticate():
int mm_answer_authpassword():
|                |             |             |                      |                        |                  |     . . .                     |     |       |     |                 | 1 c a                 | ll        |
| -------------- | ----------- | ----------- | -------------------- | ---------------------- | ---------------- | ----------------------------- | --- | ----- | --- | --------------- | --------------------- | --------- |
| r e t u rn e a | x i .e ., i | n d e p e n | d e nt o f o t h e r | r eg i s te r s ( e .g | ., g e t p i d , |                               |     |       |     |                 |                       |           |
|                |             |             |                      |                        |                  |    r e t urn auth_password(); |     | P ssh |     | / /   r e t u r | n s   ! 0   o n   s u | c c e s s |
g e t m as k , a n d o t h e r g e t te r fu n c t i o n s) . s i g r e t u r n u s e s t h e 1
|     |     |     |     |     |     |     |     |     |     | i n t   p a m _ | s m _ a u t h e n t i | c a t e ( ): |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------- | --------------------- | ------------ |
|     |     |     |     |     |     | 1   | 2   |     |     |   ...           |                       |              |
currentuserstacktorestoretheprocessstackandcanbeused call return Ppam
1
forcodereuseattacks.Similarly,setsidcreatesanewses-
// returns !0 on success
|     |     |     |     |     |     |     |     |     |     | 2   | 3   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
sionandprocessgroupandcanbeusedtomodifythevalue int auth_password(): call return
  ...
ofeax.Next,weassesswhichofthesesyscallinvocationsare   return 0; ssh pam
|     |     |     |     |     |     |     |     | P 2 |     | int _unix_blankpasswd(): |     | P 2 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------------ | --- | --- |
ofinteresttoanattacker.Itisunlikelythatataninteresting
pointduringaprogram’sexecutioneaxwillholdthevalueof
|                                                         |     |     |     |     |     | Figure 5:   | (a) Pssh | andPssh: | gadgetpages |        | in the | OpenSSH    |
| ------------------------------------------------------- | --- | --- | --- | --- | --- | ----------- | -------- | -------- | ----------- | ------ | ------ | ---------- |
| oneofthesesyscalls.eaxusuallystoresthereturnvalueof     |     |     |     |     |     |             | 1        | 2        |             |        |        |            |
|                                                         |     |     |     |     |     |             | Psudo    |          | Psudo:      |        |        |            |
|                                                         |     |     |     |     |     | binary. (b) | sudo     | and      |             | gadget | pages  | in the pam |
| functions,soitoftencontainspointersanderrorvalues.While |     |     |     |     |     |             |          | 1        | 2           |        |        |            |
sharedlibraryusedbythesudobinary.
wecannotmeaningfullychangepointervaluesbyinvoking
syscalls,weobservethatwecanchangereturnederrorcodes
asshowninSec.2.2.However,itraisesthequestion:issuch
Forexample,inthecodesnippetabove,theapplicationregis-
aprimitivetooweaktobringaboutanymaliciouseffects?
tersaSIGFPEhandleronline7.Ifthecomputationonline8
| Altering | eax to | non-zero | value. | Often guard conditions |     |     |     |     |     |     |     |     |
| -------- | ------ | -------- | ------ | ---------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
causesaSIGFPE,thehandlerisexecuted.Then,theexecution
checkfornon-zerovalues,whichifmaliciouslyaltered,can
continuesonline5.Anattackercaninjectthedivide-by-zero
inducedataandcontrolflowchanges,asshowninRowham-
interruptonline9.Thisforcestheapplicationtoalwaysex-
mer[31]andnon-control-dataattacks[19].Thus,wemake
|     |     |     |     |     |     | ecute the | handler | changing | its | execution. | As a | result, the |
| --- | --- | --- | --- | --- | --- | --------- | ------- | -------- | --- | ---------- | ---- | ----------- |
theconsciouschoicetorestrictourselvestoonlyusetheint
functionalwayscomputesanon-weightedaveragecompro-
0x80gadgetwitheaxequaltozero(e.g.,changethereturn
misingitsintegrity.Therefore,byinjectingint0x0anattacker
| valuefrom | 0to-4). | Ourcasestudiesin |     | Sec.5.1showthat |     |                        |     |     |                                 |     |     |     |
| --------- | ------- | ---------------- | --- | --------------- | --- | ---------------------- | --- | --- | ------------------------------- | --- | --- | --- |
|           |         |                  |     |                 |     | canintroduceanewstateS |     |     | a′ intheprogram’sexecutionstate |     |     |     |
thisisapowerfulprimitiveinitself.
(see Sec.3.1).
Notethat,unliketheattackusingint0x80gadgetwhich
4.2 SignalstoUserspace always invokes a syscall, the gadgets for FPE rely on
application-specifichandlersinuser-space.Further,iftheap-
x86architecturemapsfloatingpointexceptions(e.g.,divide
plicationdoesnotregisterahandler,thekernelusesadefault
byzero,overflow)tointerrupts.Whentheseinterruptsoccur,
handlerthatterminatestheprocess.
theLinuxkernelhandlesthemandraisesasignal(SIGFPE)
|     |     |     |     |     |     | OtherSignals. | HECKLERcaninjectinterruptsthatgenerate |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ------------- | -------------------------------------- | --- | --- | --- | --- | --- |
totheuser-spaceapplication.Applicationscanregisteruser-
SIGTRAP(1),SIGILL(6),SIGSEGV(4,5,10),andSIGBUS
spacehandlersforthesesignalswhichareexecutedwhenthe
(11,12,17,29)signalstouserspaceapplications.However,we
kernelraisesthesignal.Wesurveyedopen-sourceapplications
didnotfindapplicationsthatregisteredexpliciteffecthandlers
thatregisterexpliciteffecthandlersforthesesignals.
|             |     |          |                  |         |     | for these | four signals. | In  | the absence |     | of handlers, | POSIX |
| ----------- | --- | -------- | ---------------- | ------- | --- | --------- | ------------- | --- | ----------- | --- | ------------ | ----- |
| int 0,9,and | 16: | Floating | Point Exceptions | (FPEs). | We  |           |               |     |             |     |              |       |
standardstatesthatuserspaceapplicationmustbeterminated.
| found that | of all | the signals | that the | kernel raises | because |     |     |     |     |     |     |     |
| ---------- | ------ | ----------- | -------- | ------------- | ------- | --- | --- | --- | --- | --- | --- | --- |
Thus,thesefoursignalsareuninterestingforHECKLER.
| ofinterrupts,SIGFPEisthemostinteresting. |     |     |     | Handlersfor |     |          |             |     |           |            |     |           |
| ---------------------------------------- | --- | --- | --- | ----------- | --- | -------- | ----------- | --- | --------- | ---------- | --- | --------- |
|                                          |     |     |     |             |     | Chaining | Interrupts. | A   | malicious | hypervisor |     | can chain |
SIGFPEperformoperationslikesettingvariablestocertain
|     |     |     |     |     |     | multiple | gadgets by | injecting | interrupts |     | at different | points |
| --- | --- | --- | --- | --- | --- | -------- | ---------- | --------- | ---------- | --- | ------------ | ------ |
values(e.g.,setthedenominatortoanon-zerovaluetohandle
duringanapplication’sexecution.Forexample,consideran
adivide-by-zero),orskippingsomeoperations(e.g.,ignore
applicationthatperformsmultipleauthenticationchecksand
faultingdatathatcauseoverflows).Therefore,amalicioushy-
registersaSIGFPEhandler.Tosuccessfullyauthenticate,the
pervisorcanchangethecontrolanddataflowofapplications
attackershouldcompromisethedataflowonlines7and9.
bytriggeringinterruptsthatraiseSIGFPE.
First,theattackerusesint0x80tobypassthecheckonline7.
/* Example: SIGFPE handling */ Then,afterline8,theattackertriggersSIGFPEtochangethe
1
2 double arr[] = {...} valueofnto0.Thischangestheexecutiononline9passing
| 3 double | weights[] | =   | {...} |     |     | thesecondcheck. |     |     |     |     |     |     |
| -------- | --------- | --- | ----- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- |
| 4 double | avg =     | 0   |       |     |     |                 |     |     |     |     |     |     |
5 void handler() { /* compute non-weighted avg */ } /* Example: Chaining interrupts */
1
| 6 int compute_weighted() |     |     | {   |     |     | int n | = 1 |     |     |     |     |     |
| ------------------------ | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- |
2
| 7 register(SIGFPE, |     |     | handler) |     |     | void handler() |     | { n = 0 | }   |     |     |     |
| ------------------ | --- | --- | -------- | --- | --- | -------------- | --- | ------- | --- | --- | --- | --- |
3
| 8 avg | = ... | /*  | compute weighted | avg */ |     |              |          |     |     |     |     |     |
| ----- | ----- | --- | ---------------- | ------ | --- | ------------ | -------- | --- | --- | --- | --- | --- |
|       |       |     |                  |        |     | 4 int auth() | { return | 0   | }   |     |     |     |
9 ...
|        |     |     |     |     |     | 5 void grant_access() |     | {        |     |     |     |     |
| ------ | --- | --- | --- | --- | --- | --------------------- | --- | -------- | --- | --- | --- | --- |
| return | avg |     |     |     |     |                       |     |          |     |     |     |     |
| 10     |     |     |     |     |     | 6 register(SIGFPE,    |     | handler) |     |     |     |     |
}
| 11  |     |     |     |     |     | 7 if (!auth()) |     | { ... } | /* deny | access |     | */  |
| --- | --- | --- | --- | --- | --- | -------------- | --- | ------- | ------- | ------ | --- | --- |
6

|     |     |        | ssh |        | ssh |        |     | P a = (P a,1) |     |     |     |     | P c = (P c,1) |     |
| --- | --- | ------ | --- | ------ | --- | ------ | --- | ------------- | --- | --- | --- | --- | ------------- | --- |
|     |     | Benign |     | Benign |     | Attack |     |               |     |     |     |     |               |     |
s sh
|     |     | CVM |     | CVM |     | CVM |     | 1.void auth(): |     |     |     |     | call 1.int check(): |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --- | --- | --- | ------------------- | --- |
[p w d]
|     |     |     |     |     |     |     |     | 2.  res = check() |     |     |     |     | 2.  ...  |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | --- | --- | --- | -------- | --- |
sshd
non-root non-root root 3.  if (!res): /*success*/ ret 3.  return 0
|     |            |     | shell   |     | shell   | shell |     |      |     |     |     |     |         |         |
| --- | ---------- | --- | ------- | --- | ------- | ----- | --- | ---- | --- | --- | --- | --- | ------- | ------- |
|     | Hypervisor |     |         |     |         |       |     |      | t 0 | t 1 | t 2 | t 3 | t 4 t 5 | t 6 ... |
|     |            |     | sudo su |     | sudo su |       |     |      |     |     |     |     |         | eax!=0  |
|     |            |     | [pwd]   |     | [pwd]   |       |     | exec |     |     |     |     |         |         |
Attack
| ssh   |            |      |            |          |            |          |     |     | Pfa    | P a | Pfc    | P c | P c Pfa  | P a  |
| ----- | ---------- | ---- | ---------- | -------- | ---------- | -------- | --- | --- | ------ | --- | ------ | --- | -------- | ---- |
|       |            | CVM  |            |          |            |          |     |     |        | 1   |        | 1   | 3        | 3    |
| [pwd] |            |      |            |          |            |          |     |     | (call) |     | (call) |     | (ret)    |      |
|       |            | sshd |            | ssuuddoo |            | ssuuddoo |     |     | Pa     |     | Pa,Pc  |     | Pa,Pc,Pa |      |
|       |            |      |            |          |            |          | PT  | app |        | ... |        | ... |          | Aseq |
|       | Hypervisor |      | Hypervisor |          | Hypervisor |          |     |     |        |     |        |     |          |      |
Figure7:Attackerbypassesauthenticationcheckbyinjecting
|     |             |     |     |     |          |     | interrupt |     | at timet | when | detecting |     | A . Superscript | forP: |
| --- | ----------- | --- | --- | --- | -------- | --- | --------- | --- | -------- | ---- | --------- | --- | --------------- | ----- |
|     | (a) OpenSSH |     |     |     | (b) sudo |     |           |     |          | 5    |           |     | seq             |       |
pageid,subscriptforP:linenumberinpage,Pfa:pagefault
Figure6:Red:attacker-controlled,lightning:int0x80injec- inpagewithida.Foreverypagefault(blue),theGPAofthe
tion,(a):AttackonOpenSSH,amalicioushypervisorsuccess- pageisaddedtothePT app .
fullyauthenticatessshonCVMwithwrongpwd.(b):Attack
onsudo,amalicioushypervisorwithnon-rootshellonCVM
|     |     |     |     |     |     |     | user. | We  | identify | a   | gadget | in the | PAM module | with the |
| --- | --- | --- | --- | --- | --- | --- | ----- | --- | -------- | --- | ------ | ------ | ---------- | -------- |
escalatesprivilegetorootshell.
pam_sm_authenticatefunctionasshowninFig.5(b).This
functionfirstchecksiftheuserhasablankpasswordbycall-
ingthe_unix_blankpasswdfunction.Ifthischecksucceeds,
| 8   | n = second_auth() |     | /* !0 | if auth | fails | */  |     |     |     |     |     |     |     |     |
| --- | ----------------- | --- | ----- | ------- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
9 if (!n) { ... } /* auth. success */ the PAM module does notpromptthe userfora password.
| 10  | }   |     |     |     |     |     | Instead,itconsiderstheusertobecorrectlyauthenticatedand |     |          |              |     |     |              |           |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------- | --- | -------- | ------------ | --- | --- | ------------ | --------- |
|     |     |     |     |     |     |     | returns                                                 |     | to sudo. | Therefore,we |     | can | use int 0x80 | to change |
thereturnvalueof_unix_blankpasswdtoanon-zerovalue
5 CaseStudies leadingtosuccessfulauthentication.Applicationsthatusethe
samePAMlibrarytoauthenticateauser(e.g.,doas[24])are
Wechooseopen-sourceapplicationstodemonstratethefea- alsosusceptibletoHECKLERinprinciple.
sibilityandimpactofHECKLER.Then,weidentifygadgets ChainingOpenSSHandSudo. OpenSSHcanbeconfig-
thatallowamalicioushypervisortomountHECKLER. ured to prevent login as the root user. Similarly,sudo can
beconfigured(usingthesudoersfile)tolimittheuserswho
|     |     |     |     |     |     |     | can | execute | it. | With | this setup,ourattack |     | using | OpenSSH |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | --- | ---- | -------------------- | --- | ----- | ------- |
5.1 int0x80
|     |     |     |     |     |     |     | can | only | get a | non-root | shell | and our | attack | using sudo is |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ----- | -------- | ----- | ------- | ------ | ------------- |
OpenSSH. Itallowsauthenticateduserstoobtainasecure notpossible.However,wecanchainthetwoattackstoget
shell,usesubsystems(e.g.,sftp)forfiletransfers,andexecute pasttheseissues.Specifically,weattackOpenSSHtogeta
commandsonremoteservers.Inourthreatmodel,bypassing non-rootshellofauserinthesudoerslist.Thisensuresthat
OpenSSH’sauthenticationimpartstheattackerswithpower- thenon-rootusercanexecutesudo.Then,weusetheattack
fulcapabilitiestocompromisetheexecutionofaCVM.To onsudotoescalatethenon-rootshelltorootprivilegeasex-
plainedabove.Notethat,tosuccessfullychaintheattacks,the
| this | end,we | demonstrate | an attack | on  | an OpenSSH | server |     |     |     |     |     |     |     |     |
| ---- | ------ | ----------- | --------- | --- | ---------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
on the CVM using int 0x80 as shown in Fig. 6(a). We as- malicioushypervisorinjectsint0x80twotimes.
sumeamalicioushypervisorthatdoesnothavethecorrect
root password to authenticate a secure-shell on the CVM. 5.2 ApplicationswithSIGFPE
AsshowninFig.5(a),weidentifyagadgetwherechanging
Wefirstsurveyedlanguagesupportforsignalhandlersand
| the | return | value to a | non-zero | number | leads to | successful |     |     |     |     |     |     |     |     |
| --- | ------ | ---------- | -------- | ------ | -------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
authentication.Specifically,ourattacksetsthereturnvalueof then looked for existing applications that register SIGFPE
auth_passwordtoanon-zerovalueusingint0x80. handlerswithexpliciteffects.
Sudo. Using sudo, an authorized non-root user can esca- JavaStatisticalAnalysisTool. InJava,theruntime(Java
late privileges to a rootuser. We demonstrate an attackon virtualmachineorJVM)registersahandlerforSIGFPEin
sudo where an adversary with access to a non-root shell theuser-space.WhenitreceivesSIGFPEfromthekernel,the
on the CVM can gain root access (see Fig. 6(b)). Specifi- JVM translates itto a language-levelArithmeticException.
cally,themalicioushypervisorusesint0x80tobypasssudo’s TheArithmeticExceptionisthencaughtandhandledinthe
authentication mechanisms. Bydefault,sudoisconfigured application.Weanalyzeopen-sourceJavaapplicationsthat
to use Privileged Access Management (PAM). With PAM catchtheArithmeticException.Wefindaninterestinggadget
enabled, sudo invokes a PAM module to authenticate the intheJavaStatisticalAnalysisTool(JSAT)[47]:afunction
7

thatisusedtoaddnewdatatoadistributionthatrecalculates (1) Offline Phase (2) Online Phase, (3) Injection Phase
themeanandcovarianceasshownbelow.
|               |         |      |             |     |     |     |     |      CVM   |     |     |      CVM   |     |
| ------------- | ------- | ---- | ----------- | --- | --- | --- | --- | ---------- | --- | --- | ---------- | --- |
|               |         |      |             |     |     |     |     | pagefaults |     |     | pagefaults |     |
| 1 /* Example: | Disrupt | Java | with SIGFPE | */  |     |     |     |            |     |     |            |     |
2 try {
| 3 Vec       | newMean =  | ...;   | /*  | new mean       | */  |     |     |     |     |     |     |     |
| ----------- | ---------- | ------ | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
| 4 Matrix    | covariance | = ...; | /*  | new covariance |     | */  |     |     |     |     |     |     |
| 5 this.mean | = newMean; |        |     |                |     |     |     |     |     |     |     |     |
6 setCovariance(covariance);
| 7 } catch(ArithmeticException |              |     | ex)          |         |     |           |     |                |     |     |                                   |     |
| ----------------------------- | ------------ | --- | ------------ | ------- | --- | --------- | --- | -------------- | --- | --- | --------------------------------- | --- |
| { this.mean                   | = origMean;  |     | }            |         |     |           |     |                |     |     |                                   |     |
| 8                             |              |     |              |         |     |           |     | learn function |     |     | apply function + inject interrupt |     |
| During normal                 | execution,if |     | the function | catches |     | an Arith- |     |                |     |     |                                   |     |
meticExceptionitusestheoriginalmean,effectivelyignoring Figure8:Overviewofprofiling.Duringofflinephase(1)we
learnafunction(f
thefaultingdata.Online3,amalicioushypervisorcaninject app )thatmapspagefaultpatternstoHECK-
|     |     |     |     |     |     |     | LERgadgets.WerepeatedlycreateS |     |     |     | ,S  |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------ | --- | --- | --- | --- | --- |
aninterruptthatraisesSIGFPE(e.g.,int0x0fordivide-by- andPT .Dur-
|     |     |     |     |     |     |     |     |     |     |     | boot | app app |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- | ------- |
ingonlinephase(2),weapplythefunctiontomonitorwhen
| zero) andconsequently |     | the | ArithmeticException |     |     | to the ap- |     |     |     |     |     |     |
| --------------------- | --- | --- | ------------------- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- |
plication.Thiswillensurethatthefunctionalwaysignores theCVMreachesapointofinterestinitsexecution,inthein-
anynewdataadded.Thisgadgetisusedtoaddnewdatatoa jectionphase(3)weinjecttheinterrupt.{P ,P ,P }:physical
|     |     |     |     |     |     |     |             |         |     |                 |     | 1 2 3           |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ------- | --- | --------------- | --- | --------------- |
|     |     |     |     |     |     |     | addressesof | HECKLER |     | gadgetpagesinPT |     | ,{P ′,P ′,P ′}: |
multivariatenormaldistribution.Therefore,ourattackcanbe app 1 2 3
usedtobiasthedistributiontoneveracceptnewdata. predictedHECKLERgadgetpagesinPT .
app
| TextAnalysis.jl | in        | Julia. Like | Java,the                     | Julia | runtime | for- |         |             |        |     |         |                   |
| --------------- | --------- | ----------- | ---------------------------- | ----- | ------- | ---- | ------- | ----------- | ------ | --- | ------- | ----------------- |
| wards signals   | forSIGFPE | to          | a language-levelDivideError. |       |         |      |         |             |        |     |         |                   |
|                 |           |             |                              |       |         |      | We then | maliciously | invoke | the | handler | to bias the model |
WefindaninterestinggadgetinanestablishedJuliapackage
|     |     |     |     |     |     |     | trainedbytheMLP. |     | Specifically,on |     | everycalltothetanh |     |
| --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | --------------- | --- | ------------------ | --- |
fortextanalysis(TextAnalysis.jl)[20]:anevaluationfunction
function,weinjecttheinterrupttotriggerSIGFPE(line8in
tocalculateaperformancemetricbasedonprecisionandre-
thecodesnippetbelow).Thisensuresthatthetanhfunction
callscores(F-Score).IfthefunctioncatchesaDivideError,it
alwaysreturns1.Thisallowsustobiasthefinalconfusion
reportstheworstperformance,indicatingthatapairoftext
matrixforourtestdataset.
(e.g.machineandhuman-produced)arenotsimilar.
| # Example: | Disrupt            | Julia | with SIGFPE |         |     |     |                                 |     |     |     |     |     |
| ---------- | ------------------ | ----- | ----------- | ------- | --- | --- | ------------------------------- | --- | --- | --- | --- | --- |
| 1          |                    |       |             |         |     |     | 6 When&WheretoInjectInterrupts? |     |     |     |     |     |
| function   | fmeasure_lcs(RLCS, |       | PLCS,       | beta=1) |     |     |                                 |     |     |     |     |     |
2
try
3
|          |             |     |        |       |     |     | For our | attacks | to succeed, | it is | crucial | that we inject the |
| -------- | ----------- | --- | ------ | ----- | --- | --- | ------- | ------- | ----------- | ----- | ------- | ------------------ |
| 4 return | ((1+beta^2) | *   | RLCS * | PLCS) | /   |     |         |         |             |       |         |                    |
interruptsatspecificpointsduringtheapplication’sexecution.
| 5   | (RLCS | + (beta^2) | * PLCS) |     |     |     |     |     |     |     |     |     |
| --- | ----- | ---------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
6 catch ex Forexample,toattackOpenSSH(seeSec.5)weshouldinject
7 if ex isa DivideError theinterruptbeforethemm_answer_authpasswordusesthe
| 8   | return 0 |     |     |     |     |     | valuereturnedbyauth_passwordasshowninFig.5(a).If |     |     |     |     |     |
| --- | -------- | --- | --- | --- | --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- |
9 ...
weinjecttheinterruptatotherpointsduringtheapplication’s
execution,theinjectionmightnothavethedesiredside-effect
WeleveragethisbymaliciouslyraisingSIGFPEandconse-
(e.g.,changingeaxbeforeitisused),orcrashtheapplication.
quentlyDivideErrortoreporttheworstperformance.
Next,iftheCVMhasmultipleVMcores,weshouldensure
| Hand-codedMulti-layerPerceptron(MLP)inC. |     |     |     |     |     | Wetake |     |     |     |     |     |     |
| ---------------------------------------- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- |
thatourinterruptinjectionistargetedtotherightcorethat
| an MLP implementation |     | written | in  | C [46] | that uses | tanh |     |     |     |     |     |     |
| --------------------- | --- | ------- | --- | ------ | --------- | ---- | --- | --- | --- | --- | --- | --- |
fromthemathlibraryasanactivationfunctionasshownin executestheapplicationlogicwithourgadget.
|     |     |     |     |     |     |     | Overview. | ForSEV-SNP,themainchallengeforasuccessful |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------- | ----------------------------------------- | --- | --- | --- | --- |
thecodesnippetbelow.WemanuallyaddaSIGFPEhandler,
thatrecoversfromoverflowsbysettingthereturnvalueto1 attackisidentifyingthephysicalpagesofthefunctionsofin-
terest(i.e.,mm_answer_authpasswordandauth_password
asshowninthecodesnippetbelow.
forOpenSSH).Bymarkingthestage-2pagetablesasnon-
1 /* Example: Disrupt MLP with SIGFPE */ executablewecantracethetransitionfromauth_password
2 void tan_h_classify(...) { tomm_answer_authpassword.Thisispossiblebecauseour
3 output[0] = 1 /* bias term */ twotargetfunctionsareontwodifferentphysicalpages.If
| 4 for | (i = 0; i | < n; i++) |     |     |     |     |     |     |     |     |     |     |
| ----- | --------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
thisisnotthecase,i.e.,boththefunctionsareonthesame
| 5 if | (sigsetjmp(buf, | 1)) | /* on | SIGFPE | */  |     |         |           |           |                    |     |              |
| ---- | --------------- | --- | ----- | ------ | --- | --- | ------- | --------- | --------- | ------------------ | --- | ------------ |
|      |                 |     |       |        |     |     | page,we | will have | to resort | to single-stepping |     | this part of |
| 6    | output[i+1]     | = 1 |       |        |     |     |         |           |           |                    |     |              |
7 else /* no overflow */ theexecution[60]. However,forourbuildsofthetargetli-
output[i+1] = tanh(input[i]) braries,thefunctionsareindeedondifferentpages.Therefore,
8
| }   |     |     |     |     |     |     | onceweobserveapagefaultonauth_passwordfollowed |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ---------------------------------------------- | --- | --- | --- | --- | --- |
9
8

byapagefaultonmm_answer_authpassword,weinjectint setofpagesexecutedwhenthevictimapplicationexecutes
0x80.Specifically,everystage-2pagefaultcausesaVMEXIT onCVM(S ).Notethat,thisonlyrequires1pagefaultper
vm
pagethatisexecutedontheCVM.S
transparenttotheCVM.ThisallowsHECKLERtoinjectan vm containssomepages
interruptwhentheVMresumes,rightbeforeitexecutesthe executedbythekernelthatneedtoberemovedwhilecreating
nextguestinstruction.Insummary,theVMusestheattacker PT . Toidentifythekernel’spages,wecapturethesetof
app
pagesaccessedduringkernelboottoformS
| alteredstateonresumptionfromthepagefault. |     |     |     |     |     | .Byremov- |
| ----------------------------------------- | --- | --- | --- | --- | --- | --------- |
boot
AttackPhases. HECKLERattackrequiresthreephases:(a) ingallpagesinS fromS wegetS i.e.,S =S
|     |     |     |     | boot | vm user | user vm |
| --- | --- | --- | --- | ---- | ------- | ------- |
\S .Now,S
anofflineanalysistolearnafunction(f )thatmapspage boot user containsalluser-spacepagesexecutedin
app
fault patterns to HECKLER gadgets; (b) an online analysis theCVM.Toeliminatepagesthatdonotbelongtoourvictim
application(e.g.,OpenSSH,sudo)weexecutetheapplication
tomonitorwhentheCVMreachesapointofinterestinits
multipletimes(n)andcomputeS
execution;and(c)injectingtheinterrupt(seeFig.8).Inthe foreveryiteration(i).
useri
offlinephase,weassumethatthemalicioushypervisorcan The set intersection of all S gives us S i.e., S =
|                                                 |     |     |                                                      |     | useri app | app |
| ----------------------------------------------- | --- | --- | ---------------------------------------------------- | --- | --------- | --- |
|                                                 |     |     | (cid:84)n S .Byincreasingthevalueofn,wecanensurethat |     |           |     |
| createandrunCVMsidenticaltothevictimCVMmultiple |     |     | i=1 useri                                            |     |           |     |
S
timestoprofilethebehaviorofthevictimapplications[63]. onlycontainspagesexecutedbyourapplication.
app
Inthisphase,theattackercontrolsboththemalicioushyper- Oncewehavecorrectlyidentifiedtheapplication’spages,
wecancapturethepagesinS
visorandtheCVM. Intheonlinephase,whentheattacker app everytimetheyareexecuted
injectstheinterrupt,theattackeronlycontrolsthemalicious toformPT .Theguestphysicaladdressesoftheapplica-
app
tion’spageschangewhentheVMisrebooted.Therefore,to
| hypervisorbutcan | observe the CVM. | Next,we detailhow |     |     |     |     |
| ---------------- | ---------------- | ----------------- | --- | --- | --- | --- |
HECKLERusesthedifferentphasestolearnthe f function. reliablyfindourgadgetpages(PaandPc)weshouldaccount
app
|             |                                        |     | forthechangingGPAs.Tocapturethis,wecollectPT |     |     | over |
| ----------- | -------------------------------------- | --- | -------------------------------------------- | --- | --- | ---- |
| PageTraces. | Totimeandtargetourinterruptstotheright |     |                                              |     |     | app  |
core,werelyonthefactthatourgadgetssequentiallyexecute multipleVMboots.Then,weanalyzeallPT app tofindafunc-
functionsondifferentpagesasshowninFigs.5and7.Let tion f togetthegadgetpagesPaandPcinFig.7.Finally,
app
|                                                    |     |     | wecanusethegadgetpagestoidentifyA |     | tocorrectlytime |     |
| -------------------------------------------------- | --- | --- | --------------------------------- | --- | --------------- | --- |
| usassumethatthehypervisorcancaptureallpageswiththe |     |     |                                   |     | seq             |     |
cores they were used on during a CVM’s execution (e.g., andtargettheinterruptinjection.
usingpagefaults).Specifically,thehypervisorcapturesalist
(PT ) of tuples with the guest physical addresses (GPAs) 7 ImplementationforAMDSEV-SNP
vm
ofpagesandtheircorrespondingcores[(pid,core)].Using
PT ,thehypervisorcreatesapplication-specificPT shown Wedescribeourmethodtoidentifytheguestphysicaladdress
vm app
inFig.7withallpagesexecutedbytheappinuser-space.
ofthepagethathousesthegadgetsofourinterest.
ThecodesnippetsinFig.7areanalogoustothegadgetswe
detailforourcasestudiesinSec.5.Here,theauthfunction
7.1 GeneratingPageTraces
onpagePacallscheckonPcandusesitsreturnvalue.There-
fore,theapplication’spagetrace(PT )alwayscontainsthe To generate the page trace for the application (PT ), we
|           |                                         | app |                                                |     |     | app |
| --------- | --------------------------------------- | --- | ---------------------------------------------- | --- | --- | --- |
| sequenceA | =[Pa,Pc,Pa].Totimetheinterruptandtarget |     |                                                |     |     |     |
|           | seq                                     |     | needtoinducepagefaultseverytimeapageintheCVMis |     |     |     |
therightcore,thehypervisorobservestheapplication’sac- executed.InSEV-SNPthehypervisorcanforcepagefaults
cesstothesepagesandwaitstodetectthesequenceofpages. in the CVM [45,60]. SEV-Step implements a mechanism
| WhenthehypervisordetectsthesequenceA |     | itinjectsthe |                                                    |     |     |     |
| ------------------------------------ | --- | ------------ | -------------------------------------------------- | --- | --- | --- |
|                                      |     | seq          | thatcanbeconfiguredtoinducepagefaultsonallpages,or |     |     |     |
interrupt(e.g.,int0x80tochangethereturnvalueofcheck) only on 1 page. We use the formerconfiguration to create
beforeexecutionresumesonline3onPa(PainFig.7).Note
|     |     | 3   | the unorderedsets | describedin | Sec. 6. Specifically,before |     |
| --- | --- | --- | ----------------- | ----------- | --------------------------- | --- |
thatPT app issufficienttotargettheinterrupttotherightcore booting the VM, we mark all pages as not-executable by
asitcontainsinformationaboutthecoreonwhichthepage settingthenxbit.Everytimeapagefaultoccurs,wenotethe
wasaccessedbytheapplication.
page’sGPAandcore.BeforetheCVMresumesexecution,
ApplicationTrace(PT ). TocapturePT ,weassumethat KVMclearsthenxbit. Thisensuresthatonly1pagefault
|     | app | vm  |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- |
thehypervisorcaninducepagefaultsforallpageaccessesin orderedlist(PT
|     |     |     | is triggeredperpage. | To create | the | app ) we |
| --- | --- | --- | -------------------- | --------- | --- | -------- |
theCVM.CreatingPT fromPT isnotstraightforward. usethemechanismfromSEV-Steptomarksinglepagesas
|     | app | vm  |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- |
First,theGPAsfortheapplication’spagesaredifferentfor not-executable. We startbysetting allpages in S as not
app
everyexecution.Next,PT
vm containspagesusedbythekernel executable.Then,oneverypagefault,wenotetheGPAand
andalluser-spaceapplications.Further,theorderinwhichthe core.Next,wesetthenxbitofthepagethatgeneratedthe
pagesareaccessedintheCVMisaffectedbythescheduling previouspagefault.Thismechanismensuresthateveryaccess
decisions in the Linux kernel. Given these challenges,we totheapplication’spagesgeneratesafault.
detailamethodtoreliablycreatePT andidentifyA . Toimplementthismechanism,weusethemodifiedKVM
app seq
CapturingeverypageaccessforaCVM’sexecution(PT vm ) fromSEV-Stepwhichexposesioctlstotheuser-space[60].
isexpensive(manypagefaultsforthesamepage)andgener- Theseioctlsallowuser-spaceapplicationstoregisterandwait
atesanintractabletrace.Instead,itissufficienttostartwitha forevents(e.g.,pagefaults).WecreateCPython(409LoC)
9

| andPythonprograms(2291LoC)tointerfacewithKVMto |     |     |     |     |     |     | jmp1: |     |     |     |     |     |
| ---------------------------------------------- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- |
13
registerandhandleeventsforpagefaults. asm volatile("mfence":: :"memory");
14
|                   |             |                                       |                                |        |                   |     | asm volatile("push |     | %   |     |     |     |
| ----------------- | ----------- | ------------------------------------- | ------------------------------ | ------ | ----------------- | --- | ------------------ | --- | --- | --- | --- | --- |
| Optimization.     |             | Ifweenablepagefaultsforallapplication |                                |        |                   |     | 15                 |     |     |     |     |     |
|                   |             |                                       |                                |        |                   |     | asm volatile("jmp  |     | *%  |     |     |     |
| pages,thesizeofPT |             |                                       | grows.Weknowthatourgadgetpages |        |                   |     | 16                 |     |     |     |     |     |
|                   |             | app                                   |                                |        |                   |     | jmp2:              |     |     |     |     |     |
| will only         | be accessed |                                       | a few times                    | during | the application’s |     | 17                 |     |     |     |     |     |
18 }
execution.Therefore,wedefineanupperlimitonthenumber
ofoccurrencesofaparticularpageinourtracing.Thisreduces
|     |     |     |     |     |     |     | WeexecutethisCprogram |     | severaltimeson |     | theCVMand |     |
| --- | --- | --- | --- | --- | --- | --- | --------------------- | --- | -------------- | --- | --------- | --- |
PT app sizeandoptimizestheapplicationexecutiontime. capturethepagesthatareexecutedtocreateS .Notethat,
app
theaddressesforpam_unixarefixed,sowedonotneedto
executesudoapplicationduringthisphasetoformS
| 7.2 BootSet(S |     |      | )andApplicationSet(S |     |     | )   |     |     |     |     |     | .   |
| ------------- | --- | ---- | -------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|               |     | boot |                      |     |     | app |     |     |     |     |     | app |
MLP. Weuseanopen-sourceimplementationofMLPwrit-
Inboththeofflineandonlinephasesoftheattack,tocreate teninC[46]andaddaSIGFPEhandlertoitstanhactivation
theapplicationpagetrace(PT ),wefirstneedtoformthe functionimplementation.Everycalltothisactivationfunc-
app
bootsetandapplicationsetsforeachcasestudy. tionresultsinmultiplecallstothetanhfunctioninthemath
Bootset. Weusethebootsettoeliminateallpagesexecuted sharedlibraryasshowninSec.5.2.Weimplementaninter-
bythekernelfromPT .Tocreatethisset,wemarkallpages faceintheCVMtoallowuserstostartandstopthetraining
app
asnot-executablebeforebootingtheCVM. Wecaptureall oftheMLP.Thetrainingprocesstakesalongtime.Therefore,
pagesthatgeneratepage-faultswhiletheLinuxkernelboots capturingallpagesexecutedduringthetrainingresultsina
ontheCVMandaddthemtothebootset.Westopthecapture very large unusable set. So,we capture the pages multiple
oncetheCVMbootcompletes.Thisensuresthatonlykernel timesduringthetraininginsmallwindowsof1second.To
pagesarecapturedinthebootset.Next,weexplainhowwe formS wecomputeanintersectionoverallpagesfrom
useri
|                                                     |                                          |     |     |     |     |     | thewindowstocreateS |     | (seeSec.6). |     |     |     |
| --------------------------------------------------- | ---------------------------------------- | --- | --- | --- | --- | --- | ------------------- | --- | ----------- | --- | --- | --- |
| createtheapplicationsetforourend-to-endcasestudies. |                                          |     |     |     |     |     |                     |     | app         |     |     |     |
| OpenSSH.                                            | Forpasswordauthentication,OpenSSHprompts |     |     |     |     |     |                     |     |             |     |     |     |
theuserforapassword.Iftheauthenticationfails,itprompts
7.3 FindingaFunction
| the user | again. | The | code gadgets |     | we are interested | in  |     |     |     |     |     |     |
| -------- | ------ | --- | ------------ | --- | ----------------- | --- | --- | --- | --- | --- | --- | --- |
(seeSec.5)areexecutedbetweenthesesuccessiveprompts. Intheofflinephase,weusetheapplicationpagetrace(PT )
app
Therefore,toformtheapplicationsetforOpenSSH,itissuf- todefineafunction(f )topredictthephysicaladdressesof
app
ficienttocapturethepagesthatareexecutedinthispassword ourgadgetpages.Next,weexplainourhowtocreate f for
app
promptwindow.Todothis,weimplementaGoprogramasan
eachoftheend-to-endcasestudies.
sshclientwith70LoC.Forfine-grainedcontroloverthepass- OpenSSH. Weanalyzethepagetraces(PT )frommultiple
app
wordauthentication process,we modifyGo’s crypto/ssh CVMboots.Wefirstcreate2setswithpotentialcandidates
standardlibrary.Weexecutethesshclientfromtheuntrusted forgadgetpages(PsshandPssh).Usingthepagetracesacross
|     |     |     |     |     |     |     |     | 1   | 2   |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
hostmultipletimesandcapturethepagesthatareexecutedto multipleCVMbootsweprofiletheOpenSSHbehaviordur-
| formS | andsubsequentlyS |     |     | asexplainedinSec.6. |     |     |     |     |     |     |     |     |
| ----- | ---------------- | --- | --- | ------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
useri app ingpasswordauthenticationanddefineafrequencyinterval
Sudo. ItusesPAMtoperformpasswordauthenticationby [9,11].WedefineallpagesthatappearinPT withfrequen-
app
callingthepam_unixsharedlibrarywhichhasourcodegad- ciesinthisintervalascandidatepagesforPssh.Similarly,we
1
getfrom Sec.5.TheLinuxkernelexecutessharedlibraries defineafrequencyinterval[5,7]tofindthecandidatepages
fromthesamephysicaladdresses.Thus,forallexecutionsof forPssh.Notethat,fortheseVMboots,theattackeralsocon-
2
thesharedlibrary,theGPAsremainconstant.Weusethisfact
trolstheCVM.Duringtheattack,wefirstformthecandidate
tocreateourapplicationsetforthesudobinary.Specifically, sets using the values forthe frequency intervals we define
we write a C program to repeatedly access the pages with above.Then,tofurthereliminatepagesfromthecandidate
ourcodegadgeti.e.,Ppam andPpam ofthepam_unixlibrary setsandformpagetuples(Pssh,Pssh),weusethefactthat
|     |     |     | 1   | 2   |     |     |     |     | 1   | 2   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
showninFig.5(b)asshownbelow. thegadgetpagesmustappearinaparticularsequence(A )
seq
inallpagetraces.
| 1 /* Profiling |       | shared                             | libraries | */       |         |     |                 |           |             |                 |        |          |
| -------------- | ----- | ---------------------------------- | --------- | -------- | ------- | --- | --------------- | --------- | ----------- | --------------- | ------ | -------- |
|                |       |                                    |           |          |         |     | Sudo. Unlike    | OpenSSH   | identifying | the             | gadget | pages is |
| 2 char*        | lib = | "/usr/lib64/security/pam_unix.so"; |           |          |         |     |                 |           |             |                 |        |          |
|                |       |                                    |           |          |         |     | straightforward | for sudo. | First,      | in this setting | the    | attacker |
| unsigned       | long  | gad1,                              | gad2;     | char* a; | int fd; |     |                 |           |             |                 |        |          |
3
fd = open(lib, O_RDONLY); alreadycontrolsanon-rootshellontheCVM.Then,ourgad-
4
a = mmap(0, 0x4000000, (PROT_READ | PROT_EXEC), getpageslieinthepam_unixsharedlibrarywhoseGPAsdo
5
MAP_SHARED, fd, 0); notchangeacrossmultipleruns.TheCprogram’sloopuses
6
| gad1    | = a + 0xCAFEBABE;   |     | /*  | ret gadget  | 1 */ |     |                                                       |        |              |                        |              |       |
| ------- | ------------------- | --- | --- | ----------- | ---- | --- | ----------------------------------------------------- | ------ | ------------ | ---------------------- | ------------ | ----- |
| 7       |                     |     |     |             |      |     | thevirtualaddressesofthegadgetpagestorepeatedlyaccess |        |              |                        |              |       |
| gad2    | = a + 0xCAFED00D;   |     | /*  | ret gadget  | 2 */ |     |                                                       |        |              |                        |              |       |
| 8       |                     |     |     |             |      |     | them(seeSec.7.2).TodeterminetheGPAsofthesegadget      |        |              |                        |              |       |
| 9 while | (1) {               |     |     |             |      |     |                                                       |        |              |                        |              |       |
|         |                     |     |     |             |      |     | pages our function                                    | (f app | ) just picks | the                    | 2 pages that | occur |
| 10 asm  | volatile("mfence":: |     |     | :"memory"); |      |     |                                                       |        |              |                        |              |       |
|         |                     |     |     |             |      |     | themostnumberoftimesinPT                              |        |              | .Then,itusestheorderof |              |       |
| 11 asm  | volatile("push      |     | %   |             |      |     |                                                       |        | app          |                        |              |       |
|         |                     |     |     |             |      |     | accessestodeterminetheGPAsforthetuple(P               |        |              |                        | pam,P        | pam). |
| 12 asm  | volatile("jmp       |     | *%  |             |      |     |                                                       |        |              |                        | 1            | 2     |
10

Weidentify3gadgetpagesforMLP:Pmlp
| MLP. |     |     |     | thepage | 8.1 InjectingInterrupts |     |     |     |     |
| ---- | --- | --- | --- | ------- | ----------------------- | --- | --- | --- | --- |
1
thatcontainsthecallingfunctionoftanh,Pmlpthetanhshared
2
| library, | and P mlp a page | in the shared | library | executed by |                                                  |     |     |     |     |
| -------- | ---------------- | ------------- | ------- | ----------- | ------------------------------------------------ | --- | --- | --- | --- |
|          | 3                |               |         |             | WhilebothSEV-SNPandTDXallowthehypervisortoinject |     |     |     |     |
thetanhfunction.Whilethefirstpageisbackedbydifferent interruptstotheCVMs,themethodtoinjecttheinterruptis
GPAs for each application execution,the second and third differentforeachofthem.Below,weoutlinethemechanisms
pageinthesharedlibraryremainconstant.Oninvestigating
weusetoinjectinterruptsforHECKLER.
| theapplicationtracePT |     | ,weidentifyasequenceoflength |     |     |          |                                          |     |     |     |
| --------------------- | --- | ---------------------------- | --- | --- | -------- | ---------------------------------------- | --- | --- | --- |
|                       |     | app                          |     |     | SEV-SNP. | AMDvirtualmachineextensionsexposevarious |     |     |     |
9withthegadgetpagesthatoccurwithhighfrequency.We
usethistodefinethefunction(f interfacesthatahypervisorcanusetoinjectinterruptsinto
app )thefindscandidatesfor
|     |     |     |     |     | a VM. | In our implementation, | we  | use the | event injection |
| --- | --- | --- | --- | --- | ----- | ---------------------- | --- | ------- | --------------- |
tuplesofgadgetpages(Pmlp,Pmlp,Pmlp).
|                                |     | 1 2 | 3             |     |                    |               |                 |          |                 |
| ------------------------------ | --- | --- | ------------- | --- | ------------------ | ------------- | --------------- | -------- | --------------- |
|                                |     |     |               |     | interface          | to inject int | 0x80 and int    | 0x0 (see | Appx. A for     |
| EffectofImperfectPageAnalysis. |     |     | OurAMDSEV-SNP |     |                    |               |                 |          |                 |
|                                |     |     |               |     | other interfaces). | For           | this,we use the | event    | injection field |
analysisisintentionallyspecifictoourobservationsperap- (VMCB.EventInj)thatisaccessibletothehypervisorinthe
| plication. | Itis notdesignedforothergadgets |     |     | thatmay not |     |     |     |     |     |
| ---------- | ------------------------------- | --- | --- | ----------- | --- | --- | --- | --- | --- |
VirtualMachineControlBlock(VMCB)oftheSEVVM.We
conformtosuchbehaviour,anddependingonthegadget,may
implementakernelmodulewith150LoC,whichinterfaces
needinstructionsingle-stepping[60,63].Injectingint0x80 with KVM to write the interrupt number to be injected in
onthewrongpageeitherhasnoobservableeffectorcrashes
|     |     |     |     |     | the respective | VMCB | field. When | the SEV | VM resumes |
| --- | --- | --- | --- | --- | -------------- | ---- | ----------- | ------- | ---------- |
OpenSSHwhichisrestartedbythedaemon.
|                   |     |                            |     |     | execution,this | method | ensures that | the interrupt | is always |
| ----------------- | --- | -------------------------- | --- | --- | -------------- | ------ | ------------ | ------------- | --------- |
| RemarkonIntelTDX. |     | Weneedaprimitivetoknowwhen |     |     |                |        |              |               |           |
raisedbeforethenextinstructionisexecuted[4].Thismakes
theA occursduringtheapplication’sexecution.Sinceour ourinjectiondeterministic,thusensuringHECKLERdoesnot
seq
goalisnottobuildsingle-steppingandanalysistechniques needtotimetheinterruptinjectionbetweenawindowofa
demonstratedforAMDSEV[60],wedonotinvestigateusing fewCPU cycles as alreadyexplainedin Sec. 6. The KVM
pagefaults,cacheside-channels,ortimerinterrupts,toachieve implementation expects acknowledgments from the guest
thisprimitive.WehadlimitedaccesstotheTDXmachineto
kernelintheVMformostexternalinterrupts.Duringnormal
fullyexperiment.Tomakethebestuseofourlimitedaccess operation,forallexternalinterrupts,theguestLinuxkernel
andtodemonstrateourattack,weuseabusyloopinfunctions writestheacknowledgmentstoaregisterinthisvirtualAPIC
mm_answer_authpasswordandpam_sm_authenticatefor
page.Weobservethattheint0x80handlerintheguestLinux
OpenSSHandsudorespectively.Futureworkscanaddress kernel does not acknowledge the interrupt because it does
thisusingadvancesinTDX-step[36].
notexpecttheseinterruptstobeinjectedexternally.Without
suchacknowledgment,KVMwillnotinjectcertaininterrupts
whichcanleadtounexpectedbehavior(e.g.,frozenterminal
|     |     |     |     |     | because | of tty interrupts). | To remedy | this,we | perform the |
| --- | --- | --- | --- | --- | ------- | ------------------- | --------- | ------- | ----------- |
8 Proof-of-conceptExploits
virtualAPICpageregisterwritefromthehost.
TDX. Weimplementakernelmodulein150LoCtoinject
TodemonstrateHECKLERonSEV-SNPandIntelTDX,we
|     |     |     |     |     | interrupts | into the TDX | VM. Ourhostmodule |     | uses kernel |
| --- | --- | --- | --- | --- | ---------- | ------------ | ----------------- | --- | ----------- |
usethelatestproductionsystemsandsetupsrecommended
|     |     |     |     |     | hooks to | calla function | in KVM thatis | usedto | deliverint |
| --- | --- | --- | --- | --- | -------- | -------------- | ------------- | ------ | ---------- |
byAMDandIntelrespectively.
0x80interruptstoTDXVMs.UnlikeSEV-SNP,TDXdoes
SEV-SNP. WedemonstrateourattacksonanEPYC9124 notexposetheVirtualMachineControlStructure(VMCS)or
withZen4SEV-SNPenabledworkstationwith16coresand
thevirtualAPICpagestotheuntrustedhypervisor.Instead,
192GBRAM.WebootthehostLinuxkernelwithpatches
itexpectsthehypervisortowriteintoaPostedInterruptRe-
from SEV-Step that introduce the page-fault interfaces in quest(PIR)buffer.Thisbufferisusedbyhardwaretoinject
KVM[60].ThiskernelalsocontainsthepatchesforKVM
interruptsintoTDXVMsthroughthevirtualAPIC[34].We
tolaunchandmanageSEV-SNPVMs. Further,weusethe inject two interrupts into two different cores of the CVM
sameQEMUversion6.1.50andSEV-SNPVMLinuxkernel
|     |     |     |     |     | with this | mechanism,one | to gain login | into | the TDX VM |
| --- | --- | --- | --- | --- | --------- | ------------- | ------------- | ---- | ---------- |
v5.19.0toperformourexperiments.
withOpenSSHandanothertogetrootaccesswithsudo.Dur-
TDX. WehadearlyaccesstoTDXinSeptember2023.We ingthesetwoinjects,theguestkerneldoesnotacknowledge
confirmourattacksonapre-productionIntelXeonPlatinum the interrupts. While this does notstopourattacks,itdoes
processorwithTDXsupportwith112coresand256GiBof leavetheAPICwithanelevatedTask-Priority-Register(TPR),
RAM.WefollowtheofficialInteldocumentationandboota blockingalllower-priorityinterruptsontheaffectedvCPU.
patchedLinuxkernelv5.19.17onboththeguestandthehost. ThismaybreakCVMfunctionalitythatisnoticeablebythe
Further,weusemodifiedQEMUv7.0.50providedbyIntelto user.Toevadesuchdetection,weimplementaguestkernel
createTDXVMs.InMarch2024,wetestedint0x80injection module(kern_ack)thatresetstheAPICstate.Weinjectthis
onaproductionIntelXeonGold6526YprocessorwithTDX kernelmoduleintotheTDXVMasthelastpartofourattack
supportandconfirmedthatitisvulnerabletoHECKLER. aftergainingrootaccess.
11

Table1:Cardinalityofthesets(S ,S ,S )andtraces Table 2: Number of times (in % and absolute) the gadget
|     |     |     |     | boot | user | app |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | ---- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
(PT app )tofindgadgetpages.VMb:VMboot,maxcaptures: pagesforthedifferentapplicationsappearintheapplication’s
maximumnumberoftimeswecaptureapageinPT where page trace (PT ). Page trace size (|PT |) as detailed in
|     |     |     |     |     |     | app |     |     |     | app |     |     |     | app |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
0indicatesthatwealwayscapture. Tab.1.ThegadgetpageP isnotapplicabletoOpenSSHand
3
sudoastheyonlyhave2gadgetpages.
|         |     | traces |         |      |      | max      |     |     |     |         |      |      |         |        |     |
| ------- | --- | ------ | ------- | ---- | ---- | -------- | --- | --- | --- | ------- | ---- | ---- | ------- | ------ | --- |
|         |     |        | |S |    | |S   | | |S | | |PT |  |     |     |     |         |      |      |         |        |     |
|         | VMb |        | boot    | user | app  | app      |     |     |     |         |      |      |         |        |     |
|         |     | perVMb |         |      |      | captures |     |     |     | OpenSSH |      |      | Sudo    | MLP    |     |
|         |     |        |         |      |      |          |     |     |     | %       | abs. | %    | abs.    | % abs. |     |
| Openssh | 392 | 10     | 82433   | 666  | 236  | 22440    | 200 |     |     |         |      |      |         |        |     |
| Sudo    |     | 9      | 1 82431 | 259  | 6    | 199013   | 0   |     |     |         |      |      |         |        |     |
|         |     |        |         |      |      |          |     |     | P 1 | 0.044   | 9.8  | 25.3 | 50348.6 | 0.6    | 200 |
| MLP     |     | 6 20   | 82378   | 718  | 255  | 32832    | 200 |     |     |         |      |      |         |        |     |
|         |     |        |         |      |      |          |     |     | P   | 0.026   | 5.9  | 24.6 | 49051.0 | 0.6    | 200 |
2
|     |     |     |     |     |     |     |     |     | P 3 | -   | -   | -   | -   | 0.4 | 133 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
8.2 OpenSSH
|     |     |     |     |     |     |     |     | Table3:Overheadsforboottrace,applicationset(S |     |     |     |     |     |     | ),and |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------------------------------------- | --- | --- | --- | --- | --- | --- | ----- |
app
WedoourattackonanOpenSSHbinaryv9.4.P1+withPAM
|     |     |     |     |     |     |     |     | pagetrace(PT |     | app | )w.r.t.executionwithoutpagefaultsin%. |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | ------------------------------------- | --- | --- | --- | --- |
disabled.WerunansshclientonthesamehostastheCVM.
SEV-SNP. In theofflinephase,weprofilethebehaviorof App boot S PT
|     |     |     |     |     |     |     |     |     |     |     |     |     | app | app |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
OpenSSHover392VMboots.ForeveryVMboot,wecollect
10usersets(S ).Usingthese,wecreatetheapplicationset OpenSSH 14 131 5332
user
| (S  |                   |     |     |                       |     |     |     |     |     | Sudo |     | 37  | 3   | 602 |     |
| --- | ----------------- | --- | --- | --------------------- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- |
|     | )andpagetraces(PT |     |     | )ofsizesshowninTab.1. |     |     |     |     |     |      |     |     |     |     |     |
| app |                   |     | app |                       |     |     |     |     |     |      |     |     |     |     |     |
|     |                   |     |     |                       |     |     |     |     |     | MLP  |     | 38  | 3   | 81  |     |
Intheonlinephase,toprofileandattacktheapplication,we
setuptheVMtogeneratepagefaultsduringbootandduring
applicationexecution.Withthepagefaultmechanismenabled,
|     |     |     |     |     |     |     |     | SEV-SNP. |     | Weperformourofflineprofilingover9VMboots |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | ---------------------------------------- | --- | --- | --- | --- | --- |
weobserveanoverheadof11.41secondstobootascompared
andcreatetheapplicationset(S
to10.01secondswithoutthepagefaults(+14%).Creatingthe app ).Tocreateanapplication
|                                                    |     |     |     |     |     |     |     | trace(PT |     | ),weexecutetheloopthatrepeatedlyaccesses |     |     |     |     |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | ---------------------------------------- | --- | --- | --- | --- | --- |
| applicationsetinthepasswordpromptwindowtakes32.9ms |     |     |     |     |     |     |     |          |     | app                                      |     |     |     |     |     |
thesharedlibrarypagesasexplainedinSec.7.2.Withthis,
toexecutecomparedto14.2mswithoutpagefaults(+131%).
weseethatourgadgetpagesareinS
CreatingapagetracePT forthepasswordpromptwindow andupto49.9%of
|                                                   |     |     | app |     |     |     |     |                  |     |                                    |                                 |     | app |     |     |
| ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | ---------------------------------- | ------------------------------- | --- | --- | --- | --- |
|                                                   |     |     |     |     |     |     |     | thefinaltrace(PT |     |                                    | )asshowninTab.2.Fortheattack,we |     |     |     |     |
| takes773.9mstoexecute(+5332%).InTab.5,wesummarize |     |     |     |     |     |     |     |                  |     |                                    | app                             |     |     |     |     |
|                                                   |     |     |     |     |     |     |     | executesudo      |     | sufromthenon-rootshellontheCVM.Our |                                 |     |     |     |     |
pagefaultoverheadsforallthecasestudies.ForOpenSSH
andMLP,wecapthenumberoftimeseachpageiscaptured loopingtechniquetoaccessthepagesofthepam_unixshared
libraryensuresthatwereliablyfindtheGPAsofthegadget
to200(seeSec.7.1).Tab.2showsthenumberoftimesour
gadgetpagesappearonaverageinPT .Fromourprofiling, pagesandourattackalwayssucceedswith1injection.
app
|           |     |                 |     |      |        |               |     | TDX. | To  | perform | the | sudo attack,we |     | implement | a busy- |
| --------- | --- | --------------- | --- | ---- | ------ | ------------- | --- | ---- | --- | ------- | --- | -------------- | --- | --------- | ------- |
| we report |     | that on average | the | size | of our | candidate set | for |      |     |         |     |                |     |           |         |
loopinthepam_sm_authenticatefunctionthatwaitsforint
| Pssh | is4.81,andPssh |     | is8.52beforeconsideringtheattack |     |     |     |     |     |     |     |     |     |     |     |     |
| ---- | -------------- | --- | -------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1    |                | 2   |                                  |     |     |     |     |     |     |     |     |     |     |     |     |
sequence.Finally,whenweaccountforthesequence(A ) 0x80.Therefore,ourattackalwayssucceedsandweescalate
seq
toarootshellontheTDXVM.Toacknowledgetheinterrupt,
| in PT | ,on | average | we get | 2.24 (Pssh,Pssh) |     | tuples. | With |     |     |     |     |     |     |     |     |
| ----- | --- | ------- | ------ | ---------------- | --- | ------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
|       | app |         |        |                  | 1   | 2       |      |     |     |     |     |     |     |     |     |
this,wegetanaverageprobabilityofsuccessof44.71%with weinsertthekernelmoduleaswiththeOpenSSHattack.
|     |     |     |     |     |     |     |     | Chaining |     | OpenSSH | and | Sudo. | We chain | our attacks | on  |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | ------- | --- | ----- | -------- | ----------- | --- |
1interruptinjection.
TDX. AsexplainedinSec.7,weimplementbusyloopsinour OpenSSH and sudo to get around the problems discussed
inSec.5.1byinjectingint0x80twotimes.
| gadget | page | with the | function | mm_answer_authpassword. |     |     |     |     |     |     |     |     |     |     |     |
| ------ | ---- | -------- | -------- | ----------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Thiseliminatestheneedtotimeourinterruptinjection.We
| use | our kernel | module | in the | host | to inject | int 0x80. | The |     |     |     |     |     |     |     |     |
| --- | ---------- | ------ | ------ | ---- | --------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|     |            |        |        |      |           |           |     | 8.4 | FPE |     |     |     |     |     |     |
interrupt-deliveringfunctiontakes1835cyclesforeveryin-
jection.Further,oncetheattacksucceeds,weinsertakernel WeusethreedifferentapplicationstodemonstrateHECKLER
withinterruptsthatraiseSIGFPE.
moduleintheTDXVMtoresettheAPIC.Theresettakes
about3092cyclesonaverage.Withthissetup,wereportthat MLP. ToprofiletheMLPapplicationoffline,werecordall
ourattackalwayssucceeds. pagesthatareexecutedinonesecondwindows.Wecapture
|     |     |     |     |     |     |     |     | thepagesover6VMbootsandcollect10usersets(S |     |     |     |     |     |     | )per |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------ | --- | --- | --- | --- | --- | --- | ---- |
user
boot.Weobserveaverageapplicationset(S
)sizesof255
app
| 8.3 | Sudo |     |     |     |     |     |     | pages.Wecreate20traces(PT |     |     |     |     | )perVMboot.Ourgadget |     |     |
| --- | ---- | --- | --- | --- | --- | --- | --- | ------------------------- | --- | --- | --- | --- | -------------------- | --- | --- |
app
|     |     |     |     |     |     |     |     | pages | P 1 | , P 2 , P | 3 occur | 1.6% | in PT app . | Using our | function |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --------- | ------- | ---- | ----------- | --------- | -------- |
WeuseanunmodifiedsudobinaryintheUbuntu23.10distri- fromSec.7.3onaveragewefind20.5tuplesforthegadget
butionwithdefaultconfigurations. pages.Weseeanaverageprobabilityofsuccessof41.6%.
12

JSATandTextAnalysis.jl. OurmethodinSec.7requires 9 IneffectivenessofCurrentDefensesonAMD
moreengineeringtoJavaandJuliaapplicationswithruntimes
(e.g.,OpenJDKandJuliaRuntime).Asopposedtoahead-of- AMDSEV-SNPoutlinestwooptionalmodescalledRestricted
timecompiledprograms,findingthegadgetpagesforinter- and Alternate injections. They are designed to restrict the
pretedprogramsrequiresprofilingthedynamicbehaviorof hypervisor’s interruptandexception interface to the CVM.
theruntime’scodecacheandhotpaths.Forsimplicity,werun We explain the changes brought by these modes and then
ourprogramswithabusyloopinthegadgetfunctioninstead analyzetheireffectivenessagainstHECKLER.
of profiling it. We run the JVM in interpretermode where AMDSEV-SNPRestrictedInjection. Thehypervisorsets
SIGFPEistranslatedtoalanguage-levelArithmeticExcep- bit3intheSEV_FEATURESregisterpervCPUoftheCVMto
tion. enableordisablethismode.Whendisabled,thehypervisor
For JSAT, we run the LVQLLC test to create a multivari- continuestousethelegacyinterfacestoinjectallinterrupts.
Whenenabled,thehypervisorisstillabletopartiallyusethe
atenormaldistributionfromtheJSATrepository[47].With
legacyinterface(seeFig.9(b)).Specifically,itcaninjectonly
ourattack,weneedtoinject240interruptswhiletheappli-
#HVinterrupt—anewinterruptwithnumber28introduced
cation executes to change all return values of our gadget
forthismode.Further,thehypervisorcannotusethevirtual
function (Sec. 5.2). Similarly, for TextAnalysis.jl, we run
theEvaluation MetricstestsuitefromtheTextAnalysis.jl interruptqueuing.Instead,thehypervisorandtheCVMsetup
asharedmemoryregiontohousetheeventqueue.Thehyper-
repository[20]andneedtoinject2interrupts.
visorusesthe#HVasadoorbelltoinformtheCVMabout
anewinterruptinthequeue.The#HVhandlerintheCVM
thenaccessesthequeue,retrievestheactualinterruptnumber
(e.g.,int0x80)andthenhandlesthequeued-upinterrupt.
8.5 End-to-EndAttackCost
AMDSEV-SNPAlternateInjection. Therestrictedmode
describedaboveintroducesanewinterfaceforthehypervisor.
HECKLER is performed in 3 different phases as shown Moreimportantly,itbreakscompatibilitywithexistingguest
in Fig. 8. To understand the end-to-end cost of our attack, OSimplementations,requiresenlighteningtheguestOS,and
weexplaintheoverheadsforeachofthesephases. hinderslift-and-shift.Tolimitthiseffect,thealternateinjec-
tionmodeoffersthetraditionalinterruptinterface,butwitha
Offline Phase. During the offline phase,we get multiple
caveat.First,oneofthevCPUsintheCVMrunsataspecial
tracesassummarizedinTab.1.Inthiscase,theoverheads
privilegedlevelcalledVMPL0whiletherestofthevCPUs
of tracing slowdown the function generation described in
executeatnon-privilegedlevelsVMPL1-VMPL3.Second,all
Sec.8.1.Whilethiscanbefurtheroptimized,wedidnotput
thevCPUsthatexecutetheguestOSruninVMPL1-3and
effortsinsuchoptimizationssincethisisapreparatorystep
enablealternatemode.Withthiscombination,theycontinue
beforethevictimrunsitsVM.
to see a traditional interrupt interface both for configuring
OnlinePhase. HECKLERalsoenablespagefaulttracingin and receiving interrupts. Third,the vCPU that executes in
theonlinephase,i.e.whenthevictimstartsinteractingwith VMPL0 acts as a trusted bridge between VMPL1-3 CPUs
theVM.Tab.5showsatiminganalysistogenerateboottrace, andthehypervisor.Italsoperformssecurityandvirtualiza-
applicationset(S app ),andpagetrace(PT app )duringonline tiontaskswithintheCVM.Sincethisisanewpieceofcode
phase,whencomparedtotheexecutionoftheCVMwithout thatisintroduced,itcanverywellbeinchargeofpresenting
pagefaulttracing.HECKLERcausessomeslowdownbutit legacyinterruptinterfacesfortheCVM.Thisiswhy,itrunsin
doesnotimpactthevictim’susabilityorresultindetection. restrictedmode,createsasharedpage,handles#HV,converts
Thisisbecausewecanpotentiallyperformthetracingandthe themtovirtualinterrupts,anddeliversthemtotheguestOS.
injectionafterattestation,butduringtheCVMprovisioning Fig.9(b)showsthesetupwhereboththemodesareenabled
whichcantakeseveralminuteseveninabenignsetting.Thus, on CVM cores. Note that both of these modes change the
HECKLERattackhappensbeforetheusergetsaccesstothe deliverymechanismandinterfacesthatthehypervisorneeds
VM,soitwillnotnoticethelag.Incaseswherethisisnot to use to deliverthe interrupts to the guest OS,it does not
possible,wecanfurthercapthenumberofpagefaultsforany fundamentallyintroduceanyfilteringordroppingrules.
givenpage(maxcaptureinTab.1)toreducethelag,aswell
HardwareAvailability. Ourmachinesupportsbothofthese
asrepeatthesetintersectionforS todecreasethesizeof
app modesinthehardwareandwewereabletotestthatthenewly
theresultingtrace(PT ).
app introducedMSRsareoperational.
Interrupt Injection. As already layed out in Sec. 6 and ImpactonHECKLER. Themaingoalofthesenewmodes
8.1,weuseSEV’seventinjectioninterface(VMCB.EventInj) istoallowtheCVMstocontinuewiththeirassumedbehav-
toinjectinterrupts. Thismethodensuresthatthehardware iorabouttheinterruptinterfaceprovidedbythehypervisor
raisestheinterrupttotheguestkernelbeforetheVMexecutes forcompatibility.TheAMDdocumentationalludesthatthis
subsequentinstructions. mode can address potentialmisbehaviorby the hypervisor
13

maliciousinterruptisunclearbecauseitwouldrequireana-
| (a) No Protection |     | (b) Restricted |     | Alternate |     |     |     |     |     |     |     |     |     |
| ----------------- | --- | -------------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
lyzingandinterpretingtheexecuteduserspacecode.
VMPL* VMPL0 VMPL1-3 Disabling Interrupt Handlers. Another approach is to
|     |       |     |       |     |       | disable vulnerable |               | interrupts |         | by not | registering       |     | handlers |
| --- | ----- | --- | ----- | --- | ----- | ------------------ | ------------- | ---------- | ------- | ------ | ----------------- | --- | -------- |
|     | vCPU1 |     | vCPU1 |     | vCPU2 |                    |               |            |         |        |                   |     |          |
|     |       |     |       |     |       | for them           | in the        | guest      | OS.     | This   | works for         | int | 0x80 if  |
| #HV | other | #HV | other | #HV | other |                    |               |            |         |        |                   |     |          |
|     |       |     |       |     |       | the kernel         | is recompiled |            | without |        | the configuration |     | flag     |
Hypervisor Hypervisor Hypervisor CONFIG_IA32_EMULATION,whichdisablesIA32emulation.
|     |     |     |     |     |     | However,again,this |     | does | not | generalize | beyond | int | 0x80; |
| --- | --- | --- | --- | --- | --- | ------------------ | --- | ---- | --- | ---------- | ------ | --- | ----- |
andeventhenmaybreakcompatibilitywithlegacycodethat
Figure9:(a):Withoutanydefenseenabled,thehypervisorcan
reliesonint0x80behavior.Wesurvey5flavorsofGCP-and
injectallinterruptsintotheSEVVM.(b)Restrictedmode:
Azure-recommendedCVMimages(Redhat,Fedora,CentOS,
enabledonvCPUthatrunsVMPL0andthehypervisorcan
Ubuntu),standaloneDebian-rolling,andArchLinux.Allof
onlyinject#HV.Alternatemode:enabledonallnon-VMPL0
themhavekernelswith32-bitsupportcompiledinatthetime
coresandthehypervisorcannotinjectanyinterrupts.
ofwriting.Itisrequiredtoensuremaximalcompatibilityand
guaranteelegacysupport.Linux6.6.onwardsitispossibleto
thatbreakstheOSassumptions(e.g.,injectinterruptswhile dynamicallydisableCONFIG_IA32_EMULATIONatboottime.
TPRiselevated).But,itdoesnotdiscussanymandatoryse- TDXImplementation. ForIntelTDX,weimplementedboth
curity checks or filtering rules. The pseudo-code provided software-baseddefenses.First,wecompiledtheLinuxkernel
withtheCONFIG_IA32_EMULATIONflagdisabledinthecon-
byAMDdoesnotdoanysecuritychecks.Moreimportantly,
the software support for restricted mode does not perform figuration.Second,detectingifanint0x80cameexternally
anychecksorfilters[5].Thus,evenwithrestrictedmodeand requiredapatchof14LoCwherewecheckedtheAPICpage
bit.Sincethisistheonlywaytoinjectexternalinterruptson
#HV,HECKLERattacksarepossible.Themainreasonisthat
thenewmodechangesthedeliverymechanismbutdoesnot TDX,thispatchwassufficient.Whenrunningauserapplica-
stoporfilterthedeliveryofinterrupts.Asforalternatemode, tionintheguestOSthatdidagenuineint0x80,servicingit
thecurrenthypervisorandguestOSimplementationsdonot onourpatchedkernelresultedinanoverheadof460 cycles
supportalternatemode.Whenimplemented,itremainstosee whencomparedtoavanillakernel.WetestedHECKLERon
boththesepatchedversionsonIntelTDXandconfirmedthat
| if it filters   | any interrupts,even |     | though | such filtering | is not |                            |     |      |       |           |        |            |     |
| --------------- | ------------------- | --- | ------ | -------------- | ------ | -------------------------- | --- | ---- | ----- | --------- | ------ | ---------- | --- |
| specifiedbyAMD. |                     |     |        |                |        | theattackdoesnotgothrough. |     |      |       |           |        |            |     |
|                 |                     |     |        |                |        | We co-operated             |     | with | Intel | and Linux | kernel | developers |     |
toapplythesecondapproachthatdetectsexternalinterrupts
10 PotentialDefenses
|     |     |     |     |     |     | to protect | TDX | VMs against |     | HECKLER. | By  | default,TDX |     |
| --- | --- | --- | --- | --- | --- | ---------- | --- | ----------- | --- | -------- | --- | ----------- | --- |
VMsexecutewiththeIA32emulationenabledandapatch
| Given that | existing | mechanisms | for interrupt | security | are |     |     |     |     |     |     |     |     |
| ---------- | -------- | ---------- | ------------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
totheguestkernelcheckstheAPICpagebittostopexternal
insufficient,wedevelopsoftwaremethods(wherepossible) injectionsofint0x80[26].
andproposehardwaremechanismstomitigateHECKLER.
|     |     |     |     |     |     | SEV-SNP | Implementation. |     |     | We attempted |     | to implement |     |
| --- | --- | --- | --- | --- | --- | ------- | --------------- | --- | --- | ------------ | --- | ------------ | --- |
thedefenseofdetectinginterruptsbyexaminingtheAPIC
10.1 SoftwareMitigations page.OnAMD,thehypervisorcaninjectexternalinterrupts
asynchronouslyviatheAPICpage(sameasTDX).Similarto
| The main | ingredient | for HECKLER | is the | ability | of the hy- |     |     |     |     |     |     |     |     |
| -------- | ---------- | ----------- | ------ | ------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
TDX,weimplementedthevirtualAPICcheckforAMDSEV-
pervisortoexternallyinjectmaliciousinterruptsintoavCPU SNPwith14LoC.Weobservedanoverheadof10182cycles
executingtheCVM. comparedtotheoriginalunpatchedexecutionofabinarythat
DetectingExternalInterrupts. Oneseeminglystraightfor- genuinelyperformsint0x80.However,thisisinsufficienton
wardfixistoaddressthesymptomofexternalinterruptsin AMDbecausethehypervisorcanalsoinjectinterruptsviathe
software.Forexample,theguestkernelcanbepatchedtode- VMCBregisterswhicharehandledwhentheVMresumes
tectandselectivelyallowexternalinterrupts.Interruptssuch execution(Sec.8.1).Tostopthisattacksurfaceweusedthe
as int 0x80,should perhaps never arise externally and can defensestrategyofdetectingexternalinterruptsbyexamining
bedropped.However,wedidfinduse-caseswherethisisa thelastinstructionthattheuser-spaceapplicationexecuted.
desired behavior [10,12],after all,it is part of the x86-64 Thisrequiresexaminingthememoryreferencedbytherip
ISAstandard.Todisableexternaldeliveryofint0x80inthe onthesavedcontextstacktocheckiftheprogramexecuted
guest,thekernel’shandlercancheckiftheinstructioncame an intinstruction with0x80asaparameter. Thisrequires
fromtheuser-spaceorfromanexternalsourcebyexamining disassemblingtheripinreversefor2-bytes(sinceint0x80
thepreviouslyexecutedinstructionreferencedbytheRIPon results in a 2-byte opcode), where we inevitably run into
the contextstackorbychecking the APIC page. Forother classicproblemsstemmingfromvariablelengthinstructions.
interruptssuchasint0x0,determiningifitisagenuineora Determiningiftheuser-codeindeedperformedint0x80or
14

someotherstreamofinstructionsandparametersthatresult between0-31withexpliciteffecthandlersareforwardedby
inthesameopcodesisundecidable.Thusourpatchesprovide the TD module. We recommend thatTDX should treat int
incompleteprotection. 0x80 the same as 0-31 and filter it. This will break legacy
TodefendagainstHECKLER,theLinuxkernelintroduced codethatmayexternallyinjectint0x80[10,12].
a patch that disables IA32 emulation by default for SEV SEV-SNP. WerecommendthatSEVshouldemploysimilar
VMs[51]. Whilethissoftwarepatchstops HECKLER’sint filtering of all externally injected interrupts that may have
0x80attacks,itisineffectiveagainstattacksfrominterrupts expliciteffecthandlers.Doingsuchfilteringinmicrocodecan
(e.g.,int0x0)whichareconvertedtosignals.Detectingex- providecomprehensiveprotectionagainstHECKLER.While
ternalinjectionsoftheseinterruptsusingtheripisnotfea- thesameeffectcanperhapsbeachievedwiththerestricted
sible.Todecideifaninterruptislegitimate,theguestkernel andalternatemodes,wehavetworeservations.Thisrequires
would need to parse the whole instruction (opcode and all correctlypatchingseveralcodebasesforhypervisors,guest
arguments),andinsomecasesemulatetheinstruction.For OSes,andVMPL0implementations.Sincewewerenotable
example,tocheckiftheapplicationlegitimatelycausedan totestthecompleteandfunctionalimplementationsofthese
overflowresultinginanint0x10,theguestkernelwouldneed modes,itisuncleariftheyarecompletelyrobustagainsthy-
toemulatethefullarithmeticoperationtoreliablydetermine pervisors.Specifically,oneneedstoensurethatthehypervisor
overflowconditions.Protectingagainsttheseinterruptswould hasnowayto:(i)injecttheseinterruptsviatheAPICorthe
requirehardware-basedfilteringtechniquesinSec.10.2. synchronous interface; (ii) disable the restricted and alter-
UsingRestricted&AlternateInjection. Weattemptedto nate modes at any point during the CVM’s execution; (iii)
leverage the restricted and alternate mode to implement a re-enterthehandlerstoexploitrace-conditionsorbreakatom-
softwaredefensethataddsthemissingchecksatleastforint icityandnestedinterruptassumptions[30]. Theupcoming
0x0andint0x80.However,duetolackofsoftwaresupport secure AVIC proposal from AMD is a good candidate to
forthesemodesinthehypervisorandtheguestOS,wewere achievehardware-levelfiltering,wheretheCVMcanspecify
unabletoprototypethesechecks.Onecanimplementstand- ahardwareinterruptfilterwithoutsoftwareintervention[57].
alone restricted injection directly in the host Linux kernel.
However,therearenoopen-sourceimplementationsthatwe
11 RelatedWork
cantest. Further,aninitialpatchsetproposedbyMicrosoft
receivedstrongpushbackbytheLinuxcommunity[38].The
Previous works attack SEV’s memory protection to inject
main criticism for rejecting the patches was that a nested
arbitrary code to the CVM. [42,59] CrossLine attacks use
#HV might corrupt the stack and hardware cannot protect
hypervisor-controlled address space identifiers (ASIDs) to
againstthisracecondition.Onecanalsoimplementrestricted
compromiseSEVVMsjustbeforetheycrash[41].Further,
andalternatemodeincombination,whichnecessitatesnested
there have been numerous exploits that compromise SEV
virtualizationtotakeadvantageofVMPLs.Priorworksthat
VMsusingside-channels[40,43,45,58].Buhrenetal.[14]
implementsuchnestedvirtualizationforAMDSEVreport
compromiseSEV’sremoteattestationmechanismstoextract
highperformancecost—throughputdropsbetween57%and
platformkeysandperformarbitrarycodeinjectioninSEV
85%forMySQL,memcached,andNginx[25].Weanticipate
VMs.Zhangetal.architecturallyrevertmodifiedcachelines
furtherslowdownforinterruptfilteringsince,foreachinter-
tobreakSEV[63].Buhrenetal.mountfaultinjectionattacks
ruptinjectionthehosthastoschedulethevCPUrunningin
againstSEV-SNPVMsbyextractingendorsementkeysusing
VMPL0followedbythevCPUinahigherVMPLrunning
voltageglitching[13].SEV-EShasbeenshowntooffermuch
thenestedguestLinuxOS.
weaker security than SEV-SNP [2]. However, HECKLER
breaksSEV-SNPguaranteeswithoutrelyingonanymicro-
10.2 Hardware-basedSelectiveFiltering architectural,architectural,power,orglitchingside-channels.
GoogleperformedasecurityreviewofIntelTDXandSEV
Insteadofrelyingonkernelpatchesthatmaybreakcompati- SNPandreportedseveralissues[27,30].Notably,onTDX
bility,hardware-levelfilteringoffersacleanerdefense.One they founda vulnerability thatalloweduntrustedfirmware
extremesolutionistofilterallexternalinterruptsfortheCVM, to induce software exceptions during the earlybootstages.
butthisbreakscriticalfunctionalitysuchastimers.Instead, Usingthis,theygaincontrolovertheinstructionpointerdur-
weproposeselectivefilteringofinterruptsthattypicallyhave ingtrustedfirmwareexecution,thusachievingarbitrarycode
expliciteffecthandlers. execution. To the best of our knowledge, HECKLER is the
TDX. Intel already blocks interrupts 0-31 from APIC by firstattackonTDXfromuntrustedhypervisor.Further,we
default.Ifthehypervisorneedstoinjectnecessaryinterrupts donotcontroltheinstructionpointer,insteadwere-usethe
between0-31(e.g.,NMI),itneedstousetheTDXinterface. handlersinthetrustedsoftware(guestOSanduserapplica-
Thetrustdomainsmodule(TDmodule),whichisintheTCB, tions). AMD emphasizes that the hypervisor must respect
providesthisinterfaceanddetermineswhethertoforwardit RFLAGS.IF to preserve guest kernel functionality [2],but
totheCVM.AswereportedinSec.3.2,noneoftheinterrupts HECKLERdoesnotviolatethisflag.Futureworkscanexplore
15

thecombinationofHECKLERwiththismechanismtoexploit virtualinterruptinjectioninterfacetoalterthegueststateto
thekernel[39]. break the execution integrity of CVMs. WeSee [48] is our
Tooling. SEV-stepandSGX-stepusetimerinterruptstobuild follow-up work on HECKLER. It expands our analysis but
single-steppingprimitivesforSEV-SNPVMsandSGXen- focusesononeparticularinterruptvector29,VMMcommu-
clavesrespectively[54,60,63].HECKLERdoesnotrequire nication exception (#VC), which was introduced in AMD
thefull-fledgedsuiteofprimitivesofferedbythesetoolsand SEV-SNP.RefertoAppendixEforfurtherdetails.
theydonotapplyout-of-boxforourattack.However,when InterruptProtection. WojtczukandRutkowskashowedthat
webuildourtooling,were-usevaluableinsightsandimple- inamutuallyuntrustedco-tenantVMsetting,attackerscan
mentationdetailsfromthesetools. useroguedevicestoperforminterruptinjectionattacks[61].
LiftandShift. PortinglegacyapplicationstoTEEplatforms Next,wediscusspriorworksthatfocusonTEEsettings.Iso-
withzerodevelopereffortsisreferredtoaslift-and-shift.Port- latedcomputationonlow-endmicro-controllerscanbemade
ing applications to IntelSGX entails maintaining compati- resistanttointerrupt/exceptionattacks(e.g.,timerinterrupts
bility[8,11,16,50]andperformance[8,50].CVMs,dueto forside-channels)withprogrammingmechanisms[15,22,23].
theirVMabstraction,reducetheoverheadsofportinglegacy TrustZone’s secure interrupts can isolate interrupts of the
applications.However,usingAMDSEV-SNPandIntelTDX secure-world from the untrusted normal world [6]. AEX-
stillrequiresenlighteningtheguestOStoensurethatlegacy Notify makes SGX enclaves aware of timer interrupt [21]
codewrittenwiththeassumptionofatrustedhypervisoris using an ISA extension. Specifically,enclaves can register
protected in the TEE threat model. Further, the untrusted interrupthandlerstothwartsingle-steppingattacksstemming
hypervisoralsoneedstosupportthecreationofCVMsfor fromtimerinterrupts.
differentTEEbackends.Tothisend,Intel,AMD,andseveral ArmCCA. Unlikex86,Armusesdifferentinterruptarchi-
hypervisorsolutionssuchasKVMandHyper-Vareworking
tectureandnomenclature.TheArmdefines4classesofex-
towardspatchingthehypervisorsandguestOSes.Otherap-
ceptions(synchronousexception,IRQ,FIQ,andSError).We
proachesintroduceatrustedmanagerinsidetheCVMthat
studytheArmCCAsupportforcreatingCVMsandreport
actsasabridgebetweenthehypervisorandtheguestOS,re-
thatitonlyallowsinjectionofIRQsandFIQsintoArmCCA
movingtheneedtopatchexistingguestOSes.Recentworks
CVMs.TherestarefilteredbythetrustedRealmManagement
haveshownthatonecanleverageAMDSEV-SNP’sVMPL
Monitor(RMM).WetestedalltheIRQsandFIQswithRMM
modestoachievethisgoal[25].Alloftheseworksemphasize
v0.3.0anddidnotobserveexpliciteffecthandlers.Armdoes
andaimtoprotectagainstthethreatsofuntrustedprivileged
nothaveaconceptofasyscallinterruptlikex86.
software.However,theirreasoningaboutmaliciousinterrupts,
especiallyforCVMs,iseithermissingorincomplete.
Interface Security. Previous works thatattackIntelSGX
12 Conclusion
enclavesshowtheimportanceofcorrectlysecuringuntrusted
interfaces(e.g.,systemcalls)[17,37].Severalworksexploit
interfaces of various TEEs to leak secret keys and enable HECKLER presents a new attack on Intel TDX and AMD
remotecodereuse[18,44,52,53].Inasimilarvein,HECK- SEV-SNPthatofferVM abstractions. Itusesthe untrusted
LERabusestheinterruptinterfacecontrolledbytheuntrusted hypervisor’sinterruptmanagementanddeliveryinterfaceto
hypervisorbutforCVMswhichofferadifferentabstraction. injectmaliciousinterruptsintoCVMs.HECKLER’sgadgets
Physicalvs.VirtualInterrupts. Physicalinterrupts,includ- use the explicitandglobaleffects ofthe interrupthandlers
ing timers and page faults,are transparent—the victim ap- tochangethedataandcontrolflowofvictimprograms.By
plication/CVM does not recognize it was interrupted and injectingparticularmaliciousinterruptsattherighttimeinthe
resumed.Thisallowstheattackertoobserveside-effectsof rightcore,HECKLERbreakstheintegrityandsubsequently
saidinterruption[54,60].DefensessuchasAEX-Notifymake confidentialityofCVM.Ourcase-studiesshowtheseverity
thevictimawareofphysicalinterrupts,suchthatitcantake ofHECKLERandhighlighttheneedforrobustdefenses.
preventiveactions[21].HECKLERobservesthatvirtualinter-
ruptsarenottransparenttotheCVM,theydonotcauseaVM
exitbutinsteadtheCVMactivelyreactstothem asifthey
Acknowledgement
werebenigninterrupts.Oneeffectofsuchunexpectedvirtual
interruptsisthatthevictimVMcrashes(e.g.,invalidopcode
inkernelmode)orresumesexecution(e.g.,timers).Thiscan We thank our shepherd, the anonymous reviewers, and
perhapsbeusedtoamplifyside-channels,asisthecasewith MélisandeZonta-Roudesfortheirconstructivefeedbackfor
physicalinterrupts.Moreimportantly,HECKLERshowsthat improvingthepaper.ThankstoIntel,AMD,andLinuxfor
certainvirtualinterrupts,wheninjectedattherighttimeand themitigationdiscussionsandfordevelopingthepatches.We
location,haveexpliciteffectsthataltertheregisterstateof thankBennyFuhryandMonaVijfromIntelforgrantingus
thevictimCVM.HECKLERisthefirstworkthatabusesthe early-accesstoTDXpre-productionmachines.
16

References [17] StephenCheckowayandHovavShacham. Iagoattacks:
whythesystemcallAPIisabaduntrustedRPCinterface.
| [1] Alibaba. | BuildaTDXconfidentialcomputingenviron- |     |     |     |     |     |     |     |     |     |     |
| ------------ | -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
InASPLOS13.
ment,2024.
|     |     |     |     |     |     | [18] SanchuanChen,ZhiqiangLin,andYinqianZhang. |     |     |     |     | Con- |
| --- | --- | --- | --- | --- | --- | ---------------------------------------------- | --- | --- | --- | --- | ---- |
[2] AMD. AMDSEV-SNP:StrengtheningVMIsolation trolledDataRacesinEnclaves:AttacksandDetection.
| withIntegrityprotectionandmore,2020. |     |     |     |     |     | USENIXSecurity,2023. |     |     |     |     |     |
| ------------------------------------ | --- | --- | --- | --- | --- | -------------------- | --- | --- | --- | --- | --- |
[3] AMD. AMDSEV-SNPHostTree,2023. [19] ShuoChen,JunXu,andEmreC.Sezer. Non-Control-
DataAttacksAreRealisticThreats.InUSENIXSecurity,
| [4] AMD. | AMD64 | Architecture | Programmer’s |     | Manual |     |     |     |     |     |     |
| -------- | ----- | ------------ | ------------ | --- | ------ | --- | --- | --- | --- | --- | --- |
2005.
Volumes1–5,Rev.4.07,2023.
|          |       |              |     |         |          | [20] Julia | Community. | TextAnalysis.jl,Julia |     |     | package for |
| -------- | ----- | ------------ | --- | ------- | -------- | ---------- | ---------- | --------------------- | --- | --- | ----------- |
| [5] AMD. | Linux | SVSM (Secure | VM  | Service | Module), |            |            |                       |     |     |             |
TextAnalysis,accessed2023-10-15.
accessed2023-10-15.
[21] ScottConstable,JoVanBulck,XiangCheng,YuanXiao,
| [6] ARM.    | LearntheArchitecture:TrustZoneforAArch64, |     |     |     |     |                                      |           |                      |     |     |             |
| ----------- | ----------------------------------------- | --- | --- | --- | --- | ------------------------------------ | --------- | -------------------- | --- | --- | ----------- |
|             |                                           |     |     |     |     | Cedric                               | Xing,Ilya | Alexandrovich,Taesoo |     |     | Kim,Frank   |
| v.1.1,2021. |                                           |     |     |     |     | Piessens,MonaVij,andMarkSilberstein. |           |                      |     |     | AEX-Notify: |
ThwartingPreciseSingle-SteppingAttacksthroughIn-
| [7] ARM. | ArmConfidentialComputeArchitecture(ARM- |     |     |     |     |                                      |     |     |     |     |          |
| -------- | --------------------------------------- | --- | --- | --- | --- | ------------------------------------ | --- | --- | --- | --- | -------- |
|          |                                         |     |     |     |     | terruptAwarenessforIntelSGXEnclaves. |     |     |     |     | InUSENIX |
CCA),accessed2023-10-15.
Security,2023.
[8] SergeiArnautov,BohdanTrach,FranzGregor,Thomas
[22] CarlosToméCortiñas,MarcoVassena,andAlejandro
Knauth,AndreMartin,ChristianPriebe,JoshuaLind,
InIEEE
|     |     |     |     |     |     | Russo. | SecuringAsynchronousExceptions. |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ------ | ------------------------------- | --- | --- | --- | --- |
DivyaMuthukumaran,DanO’Keeffe,MarkL.Stillwell,
CSF,2020.
DavidGoltzsche,DaveEyers,RüdigerKapitza,Peter
Pietzuch,andChristofFetzer. SCONE:SecureLinux [23] RuandeClercq,FrankPiessens,DriesSchellekens,and
ContainerswithIntelSGX. InUSENIXOSDI,2016. IngridVerbauwhede. Secureinterruptsonlow-endmi-
|                     |     |                                  |     |     |     | crocontrollers. |       | InIEEEASAP,2014. |             |     |              |
| ------------------- | --- | -------------------------------- | --- | --- | --- | --------------- | ----- | ---------------- | ----------- | --- | ------------ |
| [9] MicrosoftAzure. |     | AzureConfidentialVMoptions,2024. |     |     |     |                 |       |                  |             |     |              |
|                     |     |                                  |     |     |     | [24] Debian.    | Doas: | minimalist       | replacement |     | for the more |
[10] PaulBarham,BorisDragovic,KeirFraser,StevenHand,
popularsudo,accessed2023-10-15.
TimHarris,AlexHo,RolfNeugebauer,IanPratt,and
|                 |                |                               |             |     |             | [25] XinyangGe,Hsuan-ChiKuo,andWeidongCui.    |     |                |     |     | Hecate: |
| --------------- | -------------- | ----------------------------- | ----------- | --- | ----------- | --------------------------------------------- | --- | -------------- | --- | --- | ------- |
| AndrewWarfield. |                | XenandtheArtofVirtualization. |             |     | In          |                                               |     |                |     |     |         |
| SOSP,2003.      |                |                               |             |     |             | LiftingandShiftingOn-PremisesWorkloadstoanUn- |     |                |     |     |         |
|                 |                |                               |             |     |             | trustedCloud.                                 |     | InACMCCS,2022. |     |     |         |
| [11] Andrew     | Baumann,Marcus |                               | Peinado,and |     | Galen Hunt. |                                               |     |                |     |     |         |
ShieldingApplicationsfromanUntrustedCloudwith [26] Thomas Gleixner. x86/entry: Do not allow external
0x80interrupts,accessed2023-12-10.
| Haven.                             | InUSENIXOSDI,2014. |     |     |                     |     |              |                                       |     |     |     |     |
| ---------------------------------- | ------------------ | --- | --- | ------------------- | --- | ------------ | ------------------------------------- | --- | --- | --- | --- |
|                                    |                    |     |     |                     |     | [27] Google. | AMDSecureProcessorforConfidentialCom- |     |     |     |     |
| [12] FredericBeckandOlivierFestor. |                    |     |     | SyscallInterception |     |              |                                       |     |     |     |     |
puting,2022.
| inXenHypervisor. |     | 2009. |     |     |     |              |                 |     |               |     |         |
| ---------------- | --- | ----- | --- | --- | --- | ------------ | --------------- | --- | ------------- | --- | ------- |
|                  |     |       |     |     |     | [28] Google. | ConfidentialVMs |     | on IntelCPUs: |     | Yournew |
[13] RobertBuhren,Hans-NiklasJacob,ThiloKrachenfels,
intelligentdefense,2023.
| and | Jean-Pierre | Seifert. | One | Glitch to | Rule Them |     |     |     |     |     |     |
| --- | ----------- | -------- | --- | --------- | --------- | --- | --- | --- | --- | --- | --- |
All:FaultInjectionAttacksAgainstAMD’sSecureEn- [29] Google. Oh SNP! VMs get even more confidential,
| cryptedVirtualization. |     | InACMCCS,2021. |     |     |     |     |     |     |     |     |     |
| ---------------------- | --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
2023.
[14] Robert Buhren, Christian Werling, and Jean-Pierre [30] Google. IntelTrustDomainExtensions(TDX)Security
| Seifert. | Insecure | Until | Proven | Updated: | Analyzing |     |     |     |     |     |     |
| -------- | -------- | ----- | ------ | -------- | --------- | --- | --- | --- | --- | --- | --- |
Review,2023.
| AMDSEV’sRemoteAttestation. |     |     |     | InACMCCS,2019. |     |             |              |     |              |                |     |
| -------------------------- | --- | --- | --- | -------------- | --- | ----------- | ------------ | --- | ------------ | -------------- | --- |
|                            |     |     |     |                |     | [31] Daniel | Gruss,Moritz |     | Lipp,Michael | Schwarz,Daniel |     |
[15] MatteoBusi,JobNoorman,JoVanBulck,LetterioGal- Genkin, Jonas Juffinger, Sioli O’Connell, Wolfgang
letta,PierpaoloDegano,JanTobiasMühlberg,andFrank Schoechl,andYuvalYarom. AnotherFlipintheWall
Piessens. Securinginterruptibleenclavedexecutionon InIEEESP,2018.
ofRowhammerDefenses.
| smallmicroprocessors. |     |     | ACMTOPLAS,2021. |     |     |           |     |        |         |          |         |
| --------------------- | --- | --- | --------------- | --- | --- | --------- | --- | ------ | ------- | -------- | ------- |
|                       |     |     |                 |     |     | [32] Hong | Hu, | Shweta | Shinde, | Sendroiu | Adrian, |
[16] Chia che Tsai, Donald E. Porter, and Mona Vij. ZhengLeongChua,PrateekSaxena,andZhenkaiLiang.
Graphene-SGX:ApracticallibraryOSforunmodified Data-OrientedProgramming:OntheExpressivenessof
applicationsonSGX. InUSENIXATC,2017. Non-controlDataAttacks. InIEEES&P,2016.
17

[33] IBM. Confidentialcomputing fortotalprivacy assur- [48] BenedictSchlüter,Supraja Sridhara,Andrin Bertschi,
ance,accessed2023-10-15. and Shweta Shinde. WeSee: Using Malicious #VC
|     |     |     |     | InterruptstoBreakAMDSEV-SNP. |     | InIEES&P,2024. |
| --- | --- | --- | --- | ---------------------------- | --- | -------------- |
[34] Intel. Intel®64andIA-32ArchitecturesSoftwareDe-
veloper’sManualCombinedVolumes: 1,2A,2B,2C, [49] HovavShacham. TheGeometryofInnocentFleshon
2D,3A,3B,3C,3D,and4,2023. theBone:Return-into-libcwithoutFunctionCalls(on
thex86). InACMCCS,2007.
[35] Intel. IntelTrustDomainExtensions(IntelTDX),ac-
cessed2023-10-15. [50] Youren Shen,Hongliang Tian,Yu Chen,Kang Chen,
RunjiWang,YiXu,YubinXia,andShoumengYan. Oc-
[36] Intel. IntelTrustDomainExtensionResearchandAs-
clum:SecureandEfficientMultitaskingInsideaSingle
surance,accessed2023-10-15. EnclaveofIntelSGX. InASPLOS,2020.
[37] MustakimurRahmanKhandaker,YueqiangCheng,Zhi [51] KirillShutemov. x86/coco:Disable32-bitemulationby
| Wang,andTaoWei. | COINAttacks:OnInsecurityof |     |     |     |     |     |
| --------------- | -------------------------- | --- | --- | --- | --- | --- |
defaultonTDXandSEV,accessed2023-12-10.
InASPLOS,2020.
EnclaveUntrustedInterfacesinSGX.
|     |     |     | [52] | DariusSuciu,StephenMcLaughlin,LaurentSimon,and |     |     |
| --- | --- | --- | ---- | ---------------------------------------------- | --- | --- |
[38] TianyuLan. x86/sev:AddCheckof#HVeventinpath, RaduSion. HorizontalPrivilegeEscalationinTrusted
| accessed2023-10-15. |     |     |     | Applications. | InUSENIXSecurity,2020. |     |
| ------------------- | --- | --- | --- | ------------- | ---------------------- | --- |
[39] YoochanLee,ChangwooMin,andByoungyoungLee. [53] JoVanBulck,DavidOswald,EduardMarin,Abdulla
ExpRace:ExploitingKernelRacesthroughRaisingIn-
Aldoseri,FlavioD.Garcia,andFrankPiessens. ATale
terrupts. InUSENIXSecurity,2021. ofTwoWorlds:AssessingtheVulnerabilityofEnclave
|     |     |     |     | ShieldingRuntimes. | InACMCCS,2019. |     |
| --- | --- | --- | --- | ------------------ | -------------- | --- |
[40] MengyuanLi,LucaWilke,JanWichelmann,Thomas
Eisenbarth,RaduTeodorescu,andYinqianZhang. A [54] JoVanBulck,FrankPiessens,andRaoulStrackx. SGX-
SystematicLookatCiphertextSideChannelsonAMD
Step:APracticalAttackFrameworkforPreciseEnclave
| SEV-SNP. | InIEEES&P,2022. |     |     | ExecutionControl. | InSysTEX,2017. |     |
| -------- | --------------- | --- | --- | ----------------- | -------------- | --- |
[41] Mengyuan Li, Yinqian Zhang, and Zhiqiang Lin. [55] JoVanBulck,FrankPiessens,andRaoulStrackx.Neme-
CrossLine:Breaking"Security-by-Crash"BasedMem-
sis:StudyingMicroarchitecturalTimingLeaksinRudi-
oryIsolationinAMDSEV. InACMCCS,2021. mentaryCPUInterruptLogic. InACMCCS,2018.
[42] MengyuanLi,YinqianZhang,ZhiqiangLin,andYan [56] JoVanBulck,NicoWeichbrodt,RüdigerKapitza,Frank
Solihin. Exploiting Unprotected I/O Operations in Piessens,andRaoulStrackx. TellingYourSecretswith-
| AMD’sSecureEncryptedVirtualization. |     | InUSENIX |     |     |     |     |
| ----------------------------------- | --- | -------- | --- | --- | --- | --- |
outPageFaults:StealthyPageTable-BasedAttackson
| Security,2019. |     |     |     | EnclavedExecution. | InUSENIXSecurity,2017. |     |
| -------------- | --- | --- | --- | ------------------ | ---------------------- | --- |
[43] MengyuanLi,YinqianZhang,HuiboWang,KangLi, [57] Kishon Vijay Abraham I,Suravee Suthikulpanit,and
and Yueqiang Cheng. CIPHERLEAKS: Breaking AMD. SecureAVIC:SecuringInterruptInjectionfrom
Constant-timeCryptographyonAMDSEVviatheCi- a’malicious’Hypervisor. InLPC,2023.
phertextSideChannel. InUSENIXSecurity,2021.
|     |     |     | [58] | Jan Werner, | Joshua Mason, | Manos Antonakakis, |
| --- | --- | --- | ---- | ----------- | ------------- | ------------------ |
[44] Aravind Machiry, Eric Gustafson, Chad Spensky, Michalis Polychronakis, and Fabian Monrose. The
ChristopherSalls,NickStephens,RuoyuWang,Anto-
SEVerEStOfThemAll:InferenceAttacksAgainstSe-
nioBianchi,YungRynChoe,ChristopherKruegel,and cureVirtualEnclaves. InACMAsiaCCS,2019.
| Giovanni   | Vigna. BOOMERANG:    | Exploiting    | the Se- |                                              |           |                     |
| ---------- | -------------------- | ------------- | ------- | -------------------------------------------- | --------- | ------------------- |
|            |                      |               | [59]    | LucaWilke,JanWichelmann,MathiasMorbitzer,and |           |                     |
| mantic Gap | in Trusted Execution | Environments. | In      |                                              |           |                     |
|            |                      |               |         | Thomas Eisenbarth.                           | SEVurity: | No Security Without |
NDSS,2017.
Integrity:BreakingIntegrity-FreeMemoryEncryption
[45] MathiasMorbitzer,ManuelHuber,JulianHorsch,and withMinimalAssumptions. InIEEES&P,2020.
| Sascha Wessel.     | SEVered: Subverting | AMD’s | Virtual |                                              |                                    |     |
| ------------------ | ------------------- | ----- | ------- | -------------------------------------------- | ---------------------------------- | --- |
|                    |                     |       | [60]    | LucaWilke,JanWichelmann,AnjaRabich,andThomas |                                    |     |
| MachineEncryption. | InEuroSec,2018.     |       |         |                                              |                                    |     |
|                    |                     |       |         | Eisenbarth.                                  | SEV-Step:ASingle-SteppingFramework |     |
[46] ManoharMukku. ImplementationofMultiLayerPer- forAMD-SEV,2023.
ceptroninC,2021.
|     |     |     | [61] | RafalWojtczukandJoannaRutkowska. |     | Followingthe |
| --- | --- | --- | ---- | -------------------------------- | --- | ------------ |
[47] Edward Raff. Java Statistical Analysis Tool,a Java WhiteRabbit:SoftwareattacksagainstIntel(R)VT-d
| libraryforMachineLearning,2017. |     |     |     | technology,2011. |     |     |
| ------------------------------- | --- | --- | --- | ---------------- | --- | --- |
18

| [62] Yuanzhong | Xu, Weidong | Cui, and Marcus | Peinado. |     |     |     |     |
| -------------- | ----------- | --------------- | -------- | --- | --- | --- | --- |
Table4:SEV-SNPinterfacesforinterruptinjectiontoVM[4]
Controlled-ChannelAttacks:DeterministicSideChan-
nelsforUntrustedOperatingSystems. InIEEES&P, AMDVMinterface InjectionPoint Effect
| 2015. |     |     |     | VMCB |     | raiseinterruptbeforethefirstguest |     |
| ----- | --- | --- | --- | ---- | --- | --------------------------------- | --- |
synchronous
|     |     |     |     | EventInjection |              | instructionisexecuted         |     |
| --- | --- | --- | --- | -------------- | ------------ | ----------------------------- | --- |
|     |     |     |     | virtualAPIC    | asynchronous | raiseinterruptwhenevertheAVIC |     |
[63] Ruiyi Zhang, Lukas Gerlach, Daniel Weber, Lorenz registersachangeintheIRRregister
Hetterich,YouhengLü,AndreasKogler,andMichael VMCB VMRUNloadstheintr.information
synchronous
Schwarz. CacheWarp:Software-basedFaultInjection V_IRQ intotherespectiveon-chipregisters
|     |     |     |     | PhysicalIRQ | asynchronous | Physicalinterruptsareinterpretedas |     |
| --- | --- | --- | --- | ----------- | ------------ | ---------------------------------- | --- |
usingSelectiveStateReset. InUSENIXSecurity,2024. Intercept virtualinterruptsanddonoexittheVM
| A InterruptInjectionFlow |     |     |     | B InterruptInjectionInterfaces |           |                                |     |
| ------------------------ | --- | --- | --- | ------------------------------ | --------- | ------------------------------ | --- |
|                          |     |     |     | AMD has four                   | different | interrupt injection interfaces | im- |
<mm_anwser_auth_password>
| 1   |     |     |     | plementedintheirvirtualizationextensionsassummarized |     |     |     |
| --- | --- | --- | --- | ---------------------------------------------------- | --- | --- | --- |
...
| 2   |     |     |     | inTab.4.Asdescribedinthemaintextweexclusivelyused |     |     |     |
| --- | --- | --- | --- | ------------------------------------------------- | --- | --- | --- |
call auth_password
| 3   |     |     |     | VMCB.EventInjinourproof-of-conceptimplementation.It |     |     |     |
| --- | --- | --- | --- | --------------------------------------------------- | --- | --- | --- |
test eax,eax
4
| ... |     |     |     | allowstoserveinterruptswhenaCVMisresumedwiththe |     |     |     |
| --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- |
5
|     |     |     |     | VMRUN instruction. | Next,we | have the virtual APIC | (AVIC |
| --- | --- | --- | --- | ------------------ | ------- | --------------------- | ----- |
Listing1:OpenSSHmm_anwser_auth_passwordfunction AdvancedVirtualInterruptControllerinAMDsnomencla-
ture).Thisisusedforasynchronousinterruptinjection,i.e.,
thevCPUdoesnotneedtoexitandenteragaintoreceivean
1. Guestexecutesretinauth_password interrupt. WedidnotusetheV_IRQbutthedocumentation
|     |     |     |     | indicatesitdoesthesameastheVMCB.EventInj. |     |     | Mostin- |
| --- | --- | --- | --- | ----------------------------------------- | --- | --- | ------- |
CPUfetchespageofmm_answer_authpasswordtocon- terestinglyisthelastinterface,thePhysicalIRQinterception
2.
tinueexecutionatline3inmm_answer_authpassword. bit. We can unset this bit on VMRUN and cause all physical
interruptstobereceivedbytheCVMratherthancausinga
3. CPUthrowsastage-2pagefault,sincethepagewhich VMEXIT.However,thisbitisignoredwhenRestricted/Alter-
containsmm_answer_authpasswordismarkedasnon- nateInjectionisenabled.Thusthisinterfacecannotbeused
executable. tocircumventthosemodes.TDXhasasimilarinterfacebut
isnotsusceptible.Theinterruptinterceptionbitiscontrolled
4. CPUexitsVMmodeandtransferscontrolbacktothe bytheTDmodulethatispartoftheTCB.
hypervisor.
C BusyWaitLoop
| 5. Hypervisor | clears NX | (non-executable) | bit of |     |     |     |     |
| ------------- | --------- | ---------------- | ------ | --- | --- | --- | --- |
mm_answer_authpasswordstage-2entry
Thecodebusyloopsuntilauthenticatedbecomesequalto
6. HypervisormodifiesVMCB.EventInjfieldtoinjectin- one.ThisisusedinOpenSSHandsudofortheTDXproof-
| terruptintoVM |     |     |     | of-conceptexploits. |               |      |     |
| ------------- | --- | --- | --- | ------------------- | ------------- | ---- | --- |
|               |     |     |     | 1 // assume         | authenticated | == 0 |     |
7. HypervisorexecutesVMRUN
|     |     |     |     | 2 __asm__ | __volatile__("1:;\n" |     |     |
| --- | --- | --- | --- | --------- | -------------------- | --- | --- |
"nop\n"
3
8. VMRUN evaluates VMCB.EventInj field and signals an "test %
4
| Interruptbeforeguestexecutionisresumed. |     |     |     |     | "je 1b \n" |     |     |
| --------------------------------------- | --- | --- | --- | --- | ---------- | --- | --- |
5
:"+a"(authenticated):);
6
|                                         |     |     |     | return authenticated; |     |     |     |
| --------------------------------------- | --- | --- | --- | --------------------- | --- | --- | --- |
| 9. GuestKernelhandlesinterrupt(int0x80) |     |     |     | 7                     |     |     |     |
10. Guestcontinuesexecutiononline3withcorruptedstate
D TracingOverhead
Technicallythiscorrespondstoawindowofsingleinstruction
fromthevictimCVMpointofview.However,ourpagefault Table5showsthetimetakentogenerateboottrace,applica-
mechanismexitstheVM.Thenonthehypervisor,theattacker tionset(S ),andpagetrace(PT ),whencomparedtothe
|     |     |     |     | app |     | app |     |
| --- | --- | --- | --- | --- | --- | --- | --- |
cantakeasmuchtimeasitwantstosettheinterruptvector executionoftheCVMwithoutpagefaulttracing.Duringthe
andresumetheVM.Onresumption,subsequentstepshappen offlinephase,wegetmultipletracesassummarizedinTable1.
outofthebox. Inthiscase,theoverheadsoftracingslowdownthefunction
19

| generationdescribedinSection8.1.Whilethiscanbefurther |     |     |     |     |     |     |     |     | }   |     |     |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
30
| optimized,wedidnotputeffortsinsuchoptimizationssince |     |     |     |     |     |     |     | }   |     |     |     |     |     |     |
| ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
31
|                                                 |     |     |     |     |     |     |     | return | 0;  |     |     |     |     |     |
| ----------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- |
| thisisapreparatorystepbeforethevictimrunsitsVM. |     |     |     |     |     |     |     | 32     |     |     |     |     |     |     |
}
| HECKLER    |     | also | enables    | page   | fault tracing | in the | online  | 33  |     |     |     |     |     |     |
| ---------- | --- | ---- | ---------- | ------ | ------------- | ------ | ------- | --- | --- | --- | --- | --- | --- | --- |
| phase,i.e. |     | when | the victim | starts | interacting   | with   | the VM. |     |     |     |     |     |     |     |
Listing2:User-spaceapplicationwithgeneralsignalhandler
HECKLERcausessomeslowdownbutitdoesnotimpactthe
victim’susabilityorresultindetection.Thisisbecausefor Setup. Similartotheend-to-endattacksinSec.8,weusea
|     |     |     |     |     |     |     |     | workstation | with | an EPYC | 9124 | CPU | with 16 cores | and |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ---- | ------- | ---- | --- | ------------- | --- |
OpenSSHandsudo,wecanpotentiallyperformthetracing
andtheinjectionafterattestation,butduringtheCVMpro- 192GB RAM. We boot a Linux kernel v6.5.0-rc2 from
visioning whichcan take severalminutes even in a benign AMD [3] as the host operating system,which is modified
tosupportourinjectionhooks.TostarttheVMsandperform
setting.Thus,HECKLERattackhappensbeforetheusergets
accesstotheVM,soitwillnotnoticethelag.Incaseswhere ourexperiments,weuseQEMUversion8.0.0.FortheSEV-
SNPVMweusethesameLinuxkernelasforthehost,but
thisisnotpossible(e.g.MLP),wecanfurthercapthenumber
ofpagefaultsforanygivenpage(maxcaptureinTab.1)to withoutthepatches.Totestiftheguestkernelgeneratesasig-
reducethelag. nalbasedontheinjectedinterrupt,weimplementauserspace
applicationthatiscapableofcatchingthesesignals.Theap-
|     |     |     |     |     |     |     |     | plication | (sources | in Lst. | 2) registers |     | dummy handlers | for |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | -------- | ------- | ------------ | --- | -------------- | --- |
E InterruptClassification
signals0-31withthekernelandbusy-waitsfortheirdelivery.
Furthermore,itcheckscontinuouslyifregisters,suchaseax,
| CustomInterruptInjections. |     |     |     | Tofindinterruptsofinterest, |     |     |     |     |     |     |     |     |     |     |
| -------------------------- | --- | --- | --- | --------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
weremodified,forexamplebythekernel.Toensurethatwe
weanalyzetheimpactofmaliciousinjectionsonaguestVM areactuallyinjectingthetheinterruptwhileourapplication
kernelanduserspaceapplication.Forthis,wesurveyall256
isexecuting,werunandpinitoneachavailablevCPU(8in
| interrupt |     | vectors | that are | supported | by current | AMD | SEV- |     |     |     |     |     |     |     |
| --------- | --- | ------- | -------- | --------- | ---------- | --- | ---- | --- | --- | --- | --- | --- | --- | --- |
ourcase)usingtaskset.Thislowerstheriskoftheinterrupt
| SNP-enabledCPUs. |     |     | Togetstarted,weexaminethelegacy |     |     |     |     |     |     |     |     |     |     |     |
| ---------------- | --- | --- | ------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
beingdeliveredduringkernelexecution,bypreventingitfrom
| interrupt |     | injection | functions | in  | KVM that | are used | by the |     |     |     |     |     |     |     |
| --------- | --- | --------- | --------- | --- | -------- | -------- | ------ | --- | --- | --- | --- | --- | --- | --- |
idling.
hypervisortoinjectbenigninterrupts.Onthehostkernel,we Evaluation. To classify the interrupts, we first manually
hooktheseKVMfunctionstobuildakernelmoduleinterface.
analyzedtheinterrupthandleroftheguestkernelforthean-
Usingacustomkernelmodule,wecannowselectivelyinject
ticipatedbehavior.Thisgivesustheabilitytocomparethe
interruptsintoVMsasexplainedinSec.8.1.
|     |     |     |     |     |     |     |     | actual behavior |     | of the OS | to the | injected | interrupt | with an |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --------- | ------ | -------- | --------- | ------- |
expectednormalexecutionflow.Tothebestofourabilities,
| 1   | jmp_buf | jumper; |     |     |     |     |     |     |     |     |     |     |     |     |
| --- | ------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
weclassifythepredictionsintothefollowingcategoriesusing
| 2   | void handler(int |     | signum){         |     |     |     |     |                 |     |     |     |     |     |     |
| --- | ---------------- | --- | ---------------- | --- | --- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- |
| 3   | printf("in       |     | sig handler\n"); |     |     |     |     | manualanalysis: |     |     |     |     |     |     |
| 4   | longjmp(jumper,  |     | 1);              |     |     |     |     |                 |     |     |     |     |     |     |
1. Signaltousermode:Thekernelisexpectedtogenerate
5 }
|     |     |     |     |     |     |     |     | and | send a | signal to | the user | space | application | when |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------ | --------- | -------- | ----- | ----------- | ---- |
int main()
6
|     | {   |     |     |     |     |     |     | theinterruptarriveswhilethecoreisexecutinginuser |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- | --- |
7
|     | volatile |     | int i, j; |     |     |     |     | space. |     |     |     |     |     |     |
| --- | -------- | --- | --------- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- |
8
|     | struct | sigaction |     | act; |     |     |     |     |     |     |     |     |     |     |
| --- | ------ | --------- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
9
struct sigaction oldact; 2. SpecifickernelhandlerThekernelhasreservedhan-
10
memset(&act, 0, sizeof(act)); dlersthatareexecutedinthekernelwhenaninterrupt
11
12 act.sa_handler = handler; ofthistypeisdelivered.However,itdoesnotsendany
| 13  | act.sa_flags |      | = SA_NODEFER |           | | SA_NOMASK; |     |     | signaltouserspace. |     |     |     |     |     |     |
| --- | ------------ | ---- | ------------ | --------- | ------------ | --- | --- | ------------------ | --- | --- | --- | --- | --- | --- |
| 14  | for          | (i = | 0; i <=      | 32; i++){ |              |     |     |                    |     |     |     |     |     |     |
15 if (sigaction(i, &act, NULL)){ 3. DefaultkernelhandlerThekernelhasnotregistered
16 printf("Sig. install err\n");}} anyspecialhandlingforthistypeofinterrupt.Instead,
| 17  | while | (1){   |     |     |     |     |     |         |         |             |     |      |                   |     |
| --- | ----- | ------ | --- | --- | --- | --- | --- | ------- | ------- | ----------- | --- | ---- | ----------------- | --- |
|     |       |        |     |     |     |     |     | a basic | handler | is executed |     | that | logs the delivery | and |
| 18  |       | int x; |     |     |     |     |     |         |         |             |     |      |                   |     |
resumestheexecution.
| 19  |     | asm("label2:"); |     |         |              |     |     |                                                     |     |     |     |     |     |     |
| --- | --- | --------------- | --- | ------- | ------------ | --- | --- | --------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|     |     | printf("Reached |     | outside | of loop\n"); |     |     |                                                     |     |     |     |     |     |     |
| 20  |     |                 |     |         |              |     |     | 4. KernelPanicWhenaninterruptofthistypeisdelivered, |     |     |     |     |     |     |
x = setjmp(jumper);
| 21  |     |       |        |     |     |     |     | thekernelreactswithanunrecoverablepanicandhalts |     |     |     |     |     |     |
| --- | --- | ----- | ------ | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- | --- | --- |
|     |     | if (x | == 0){ |     |     |     |     |                                                 |     |     |     |     |     |     |
| 22  |     |       |        |     |     |     |     | theexecutiononallvCPUs.                         |     |     |     |     |     |     |
loop1:
23
|     |     | while  | (1){      |           |     |     |     |                                                     |     |        |            |     |                 |      |
| --- | --- | ------ | --------- | --------- | --- | --- | --- | --------------------------------------------------- | --- | ------ | ---------- | --- | --------------- | ---- |
| 24  |     |        |           |           |     |     |     | 5. Syscall                                          | We  | expect | the kernel | to  | handle a system | call |
|     |     |        | asm("cmpl | $0,       | %   |     |     |                                                     |     |        |            |     |                 |      |
| 25  |     |        |           |           |     |     |     | originatedbytheuserspaceapplication.                |     |        |            |     |                 |      |
| 26  |     |        | "jne      | label2"); |     |     |     |                                                     |     |        |            |     |                 |      |
| 27  |     | }      |           |           |     |     |     | 6. UnclearTheexpectedbehaviorofthisinterrupthandler |     |        |            |     |                 |      |
| 28  |     | } else | {         |           |     |     |     | isinconclusivefromourmanualanalysis.                |     |        |            |     |                 |      |
| 29  |     | goto   | loop1;    |           |     |     |     |                                                     |     |        |            |     |                 |      |
20

Table5:Overheadstogenerateboottrace,applicationset(S
|     |     |       |       |     |       | ),andpagetrace(PT |       | )w.r.t.executionwithoutpagefaults. |     |
| --- | --- | ----- | ----- | --- | ----- | ----------------- | ----- | ---------------------------------- | --- |
|     |     |       |       |     |       | app               | app   |                                    |     |
|     | App |       | boot  |     |       | S                 |       | PT                                 |     |
|     |     |       |       |     |       | app               |       | app                                |     |
|     |     | trace | trace |     | trace | trace             | trace | trace                              |     |
|     |     |       |       | +%  |       |                   | +%    |                                    | +%  |
|     |     | off   | on    |     | off   | on                | off   | on                                 |     |
OpenSSH 10.01s 11.41s 14% 14.2ms 32.9ms 131% 14.2ms 773.9ms 5332%
|     | Sudo | 10.01s | 13.68s | 37% | 1.00s | 1.03s | 3% 1.00s | 7.02s | 602% |
| --- | ---- | ------ | ------ | --- | ----- | ----- | -------- | ----- | ---- |
|     | MLP  | 8.71s  | 12.01s | 38% | 1.08s | 1.11s | 3% 1.08  | 1.95s | 81%  |
Table6:SummaryofobservationsforinjectinginterruptsintoaguestVMonAMDSEV-SNP(pervector).
Int.(Dec) Int.(Hex) Description ExpectedbehavioringuestVM Observedbehavior
| 0     | 00    | Divideby0                    |     |     |     | Signaltousermode      |     | Signaltousermode(SIGFPE)  |     |
| ----- | ----- | ---------------------------- | --- | --- | --- | --------------------- | --- | ------------------------- | --- |
| 1     | 01    | Debug                        |     |     |     | Signaltousermode      |     | Signaltousermode(SIGTRAP) |     |
| 2     | 02    | NMI                          |     |     |     | Specifickernelhandler |     | VM_EXITfail               |     |
| 3     | 03    | Breakpoint                   |     |     |     | Specifickernelhandler |     | VM_EXITfail               |     |
| 4     | 04    | Overflow                     |     |     |     | Signaltousermode      |     | VM_EXITfail               |     |
| 5     | 05    | BoundRangeExceeded           |     |     |     | Signaltousermode      |     | VM_EXITfail               |     |
| 6     | 06    | InvalidOpcode                |     |     |     | Signaltousermode      |     | Signaltousermode(SIGILL)  |     |
| 7     | 07    | Devicenotavailable           |     |     |     | Specifickernelhandler |     | Appcrash                  |     |
| 8     | 08    | DoubleFault                  |     |     |     | KernelPanic           |     | Kernelpanic               |     |
| 9     | 09    | Co-ProcessorSegmentoverrun   |     |     |     | Signaltousermode      |     | VM_EXITfail               |     |
| 10-13 | 0A-0D | TSS/Segment/ProtectionFaults |     |     |     | Unclear               |     | Appcrash                  |     |
| 14    | 0E    | PageFault                    |     |     |     | Specifickernelhandler |     | Kernelpanic               |     |
| 15    | 0F    | SpuriousInterrupt            |     |     |     | Unclear               |     | VM_EXITfail               |     |
| 16    | 10    | x87FloatingPointException    |     |     |     | Unclear               |     | Noeffect                  |     |
| 17    | 11    | AlignmentCheck               |     |     |     | Specifickernelhandler |     | Appcrash                  |     |
| 18    | 12    | MachineCheck                 |     |     |     | Specifickernelhandler |     | Noeffect                  |     |
| 19    | 13    | SIMDFloatingPointException   |     |     |     | Unclear               |     | Noeffect                  |     |
20 14 VirtualizationException Specifickernelhandler VM_EXITfail
21 15 ControlProtectionException Specifickernelhandler Appcrash
| 22-28 | 16-1C | Undefined |     |     |     | Unclear |     | VM_EXITfail |     |
| ----- | ----- | --------- | --- | --- | --- | ------- | --- | ----------- | --- |
29 1D VMMCommunicationException Specifickernelhandler VM_EXITfail
| 30  | 1E  | Undefined     |     |     |     | Unclear               |     | Kernelpanic            |     |
| --- | --- | ------------- | --- | --- | --- | --------------------- | --- | ---------------------- | --- |
| 31  | 1F  | Undefined     |     |     |     | Unclear               |     | VM_EXITfail            |     |
| 32  | 20  | IRETException |     |     |     | Specifickernelhandler |     | Noeffect               |     |
| 33  | 21  | Undefined     |     |     |     | Defaultkernelhandler  |     | Defaulthandlerexecuted |     |
| 34  | 22  | Undefined     |     |     |     | Defaultkernelhandler  |     | Noeffect               |     |
35-47 23-2F Undefined Defaultkernelhandler Defaulthandlerexecuted
| 48  | 30  | ISAIRQ |     |     |     | Specifickernelhandler |     | Noeffect               |     |
| --- | --- | ------ | --- | --- | --- | --------------------- | --- | ---------------------- | --- |
| 49  | 31  | ISAIRQ |     |     |     | Specifickernelhandler |     | Defaulthandlerexecuted |     |
| 50  | 32  | ISAIRQ |     |     |     | Specifickernelhandler |     | Systemunresponsive     |     |
51-63 33-3F ISAIRQ Specifickernelhandler Defaulthandlerexecuted
64-127 40-7F Undefined Defaultkernelhandler Defaulthandlerexecuted
| 128 | 80  | Syscall |     |     |     | Syscall |     | Appcontrolflowchanged |     |
| --- | --- | ------- | --- | --- | --- | ------- | --- | --------------------- | --- |
129-235 81-EB Undefined Defaultkernelhandler Defaulthandlerexecuted
236-243 EC-F3 LocalTimerandHypervisorInt. Specifickernelhandler Noeffect
244 F4 DeferredError Specifickernelhandler Specifichandlerexecuted
245-247 F5-F7 IRQWork+x86IPIInterrupts Specifickernelhandler Noeffect
248 F8 RebootInterrupt Specifickernelhandler Systemunresponsive
249-250 F9-FA Threshold+ThermalAPICInt. Specifickernelhandler Specifichandlerexecuted
251-254 FB-FE FunctionCall,Resched.,ErrorInt. Specifickernelhandler Noeffect
255 FF SpuriousAPICInterrupt Specifickernelhandler Systemunresponsive
21

We classify the observable behavior into the following
categories:
1. Signaltouserspace(TYPE)WeobservedthataTYPE
signalwasreceivedintheuserspaceapplication
2. Specific handlerexecuted The guestVM executeda
dedicatedinterrupthandlerforthisvectorinthekernel.
3. DefaulthandlerexecutedTheguestVMexecutedthe
basicplaceholderinterrupthandlerwithoutanyimplica-
tions.
4. NoeffectTheinvocationoftheinterruptdidnothavean
observableeffectontheguestVM.
5. System unresponsive The status of the guest VM is
inconclusive; no information exchange withthe guest
Kernelispossibleaftertheinterruptinjection.
6. AppcrashTheexecutionofouruserspaceapplication
wasterminated,buttheOSwasabletocontinueoperat-
ingnormally.
7. AppcontrolflowchangedWewereabletoseeanim-
pactonthecontrolflowoftheexecuteduserspaceappli-
cation.
8. VM_EXITfailAfterinjectingtheinterrupt,thehyper-
visorwasnotabletocontinuetheexecutionoftheVM
astheCPUrefusedtoentertheguestsuccessfully.
WepresentasummaryofourfindingsinTab.6.Weusedthe
svm_deliver_interruptfunctionwithintheLinuxkernel
toinjectalltheinterrupts.ThisAPIdoesnotallowustoset
anerrorcodenordoesitalwayssettherighthardwareflags
forallinterrupts(i.e.,NMIsaresupposedtobeinjectedus-
ingadifferentAPI).Thus,forallinterruptswherewereport
VM_EXITfail,onecanstillinjecttheseinterruptsintheVM
bydirectlymodifyingtherespectivefieldsintheVMCB.Fur-
thermore,someinterruptsmightpanicthekernelonlyunder
certaincircumstancesandhavedifferenteffectsiftheinterrupt
israisedattherighttime(e.g.,int3onadebuginstruction).
Asaconcreteexample,weconsiderinterrupt29whichwas
flaggedas“VM_EXITfail”inouranalysis.Onfurtherman-
ualinvestigation,weidentifiedthatbydirectlymanipulating
VMCB.EventIn,wecaninjecttheinterruptintotheVMand
causeatermination.Byfurthermodifyingtheerror_code
wecancircumventthecrashandcausedifferentglobalstate
changeeffectswhentheCVMresumesexecution.Weinvesti-
gatetheimpactofthisbehaviorindetailinWeSee[48].For
thescopeofHECKLER,wedonotdoanextensiveanalysis
of:(a)allpotentialhardwarefeaturesandinterfacestoinject
interruptsand(b)allinjectionpointsduringthevictimexecu-
tion.Weleavethisanalysisaswellaseffectsofcombining
(a)and(b)todetectotherinstancesofAhoiattackstofuture
work.
22
