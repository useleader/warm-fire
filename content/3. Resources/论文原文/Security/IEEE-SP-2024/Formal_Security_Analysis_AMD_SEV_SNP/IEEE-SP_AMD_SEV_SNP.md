---
publish: true
---

1
| Formal |     | Security |     |          | Analysis |     | of        | the AMD |     |     | SEV-SNP |     |     |
| ------ | --- | -------- | --- | -------- | -------- | --- | --------- | ------- | --- | --- | ------- | --- | --- |
|        |     |          |     | Software |          |     | Interface |         |     |     |         |     |     |
Petar Paradžik, University of Zagreb Faculty of Electrical Engineering and Computing, petar.paradzik@fer.hr
|      |        | University | of  | Zagreb | Faculty | of Electrical | Engineering | and | Computing, |     |                   |     |     |
| ---- | ------ | ---------- | --- | ------ | ------- | ------------- | ----------- | --- | ---------- | --- | ----------------- | --- | --- |
| Ante | Derek, |            |     |        |         |               |             |     |            |     | ante.derek@fer.hr |     |     |
Marko Horvat, University of Zagreb Faculty of Science, marko.horvat@math.hr
Abstract—AMD Secure Encrypted Virtualization technologies device and Confidential Compute Architecture (CCA) for
enable confidential computing by protecting virtual machines enhancing security in cloud and edge computing environ-
5202 naJ 01  ]RC.sc[  5v69201.3042:viXra
| from highly | privileged | software | such | as hypervisors. |     | In this |     |     |     |     |     |     |     |
| ----------- | ---------- | -------- | ---- | --------------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
ments [4].
work,wedevelopthefirst,comprehensivesymbolicmodelofthe
|          |           |               |               |     |        |            | Historically, | the | first of | the hardware-based |     |     | TEE solutions |
| -------- | --------- | ------------- | ------------- | --- | ------ | ---------- | ------------- | --- | -------- | ------------------ | --- | --- | ------------- |
| software | interface | of the latest | SEV iteration |     | called | SEV Secure |               |     |          |                    |     |     |               |
Nested Paging (SEV-SNP). Our model covers remote attestation, fromAMDwasnamedSecureVirtualization(SEV)[3].Devel-
key derivation, page swap and live migration. We analyze the opedin2016,SEVwasmadeavailableforthefirstgeneration
security of the software interface of SEV-SNP and formally oftheAMDEPYCbrandofx86-64microprocessors,basedon
| prove that | most | critical secrecy, | authentication, |     | attestation | and |     |     |     |     |     |     |     |
| ---------- | ---- | ----------------- | --------------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
theZen1microarchitecturecodenamedNaples.SEVenhanced
| freshness | properties | do indeed | hold in | the model. | Furthermore, |     |             |         |     |        |            |     |                |
| --------- | ---------- | --------- | ------- | ---------- | ------------ | --- | ----------- | ------- | --- | ------ | ---------- | --- | -------------- |
|           |            |           |         |            |              |     | VM security | through | VM  | memory | encryption |     | and isolation. |
wefindthattheplatform-agnosticnatureofmessagesexchanged
between SNP guests and the AMD Secure Processor firmware It also offered live VM migration and launch attestation
presents a potential weakness in the design. We show how this features.Thelatterrepresentedaratherlimitedformofremote
weaknessleadstoformalattacksonmultiplesecurityproperties, attestation,allowingonlytheguestownertoattesttheintegrity
| including   | the partial | compromise | of               | attestation | report | integrity, |                |            |            |           |               |            |             |
| ----------- | ----------- | ---------- | ---------------- | ----------- | ------ | ---------- | -------------- | ---------- | ---------- | --------- | ------------- | ---------- | ----------- |
|             |             |            |                  |             |        |            | of their guest | VMs        | during     | the guest | launch        | procedure. |             |
| and discuss | possible    | impacts    | and mitigations. |             |        |            |                |            |            |           |               |            |             |
|             |             |            |                  |             |        |            | A year         | later, AMD | introduced |           | SEV Encrypted |            | State (SEV- |
IndexTerms—trustedexecutionenvironments,formalsecurity ES)[5],makingitavailableforthesecondgenerationofAMD
| analysis, | SEV-SNP, | system verification |     |     |     |     |                      |              |                  |          |         |                     |               |
| --------- | -------- | ------------------- | --- | --- | --- | --- | -------------------- | ------------ | ---------------- | -------- | ------- | ------------------- | ------------- |
|           |          |                     |     |     |     |     | EPYC microprocessors |              | based            | on       | the Zen | 2 microarchitecture |               |
|           |          |                     |     |     |     |     | codenamed            | Rome. SEV-ES |                  | encrypts | and     | protects            | the integrity |
|           |          | I. INTRODUCTION     |     |     |     |     |                      |              |                  |          |         |                     |               |
|           |          |                     |     |     |     |     | of CPU registers     | that         | storeinformation |          | about   | the                 | VM runtime    |
An increasing number of cloud providers are beginning to beforehandingovercontroltoahypervisor,therebypreventing
|                 |     |                        |     |        |     |            | the hypervisor | from | reading | sensitive | information |     | of the guest |
| --------------- | --- | ---------------------- | --- | ------ | --- | ---------- | -------------- | ---- | ------- | --------- | ----------- | --- | ------------ |
| rely on trusted |     | execution environments |     | (TEEs) | to  | ensure the |                |      |         |           |             |     |              |
safety of user data while it is being processed by applications as well as tampering with the control flow of the guest.
and services on foreign platforms. TEEs are tamper-resistant In this paper, we focus on AMD SEV Secure Nested
environments intended to provide the confidentiality and in- Paging(SEV-SNP),whichcameoutin2020andrepresentsthe
tegrity of user code and data in use, and isolate them from latest iteration of the AMD SEV technologies. It is currently
untrusted, yet highly privileged software such as operating availableforthethirdandfourthgenerationoftheAMDEPYC
systemkernelsandhypervisors.ThismakesTEEssuitablefor microprocessors,basedonZen3andZen4microarchitectures
processing secrets on remote devices and platforms such as codenamed Milan and Genoa, respectively. With the added
publicclouds.ModernTEEsalsoprovideusefulfeaturessuch precaution of treating the hypervisor as fully untrusted, AMD
as remote attestation, a mechanism by which a remote party SEV-SNP introduces memory integrity protection as well as
canobtainevidencethataparticularprocessorvirtualmachine other security and usability enhancements to the existing
(VM) is running correctly with the expected configuration. SEVandSEV-ESfunctionalities,includingmoreflexibilityof
Hardware-based TEE solutions typically involve a security essential features such as live migration and enabling remote
kernel executed on a main processor with security extensions, attestationforthirdparties.AMDSEV-SNPisusedbypopular
oronacoprocessor.Thesecuritykernelisapieceofsoftware cloud providers such as Microsoft Azure [6], AWS [7], and
hardwired on a chip (ARM TrustZone [1]), or it comes in Google Cloud [8] to help provide Infrastructure as a Service.
the form of microcode (Intel SGX [2]) or firmware (AMD TherehasbeenasubstantialamountofanalysisoftheSEV
SEV [3]) that resides in an isolated area within the processor, technologyfrombothacademiaandindustrythat,forthemost
|     |     |     |     |     |     |     | part, focuses | on its | implementation. |     | However, |     | to the best of |
| --- | --- | --- | --- | --- | --- | --- | ------------- | ------ | --------------- | --- | -------- | --- | -------------- |
whichisnotdirectlyaccessibleormodifiablebyexternalcode.
Thesoftwareimplementsaninterfaceasaninstructionsetthat our knowledge, there is no work that formally analyzes the
can be used to establish a secure environment. software interface of SEV-SNP.
Multiple hardware vendors today offer competing TEE AMD SEV-SNP is a complex system which supports a
architectures with varying levels of granularity. Intel provides large number of guest policies and features, with most having
Software Guard Extensions (SGX) to isolate specific parts of multiple variants, and it uses an intricate key schedule. This
applications (enclaves) and Trust Domain Extensions (TDX) makesitverydifficulttoascertainwhetheranadversarycould
to protect the entire software stack of a VM. Similarly, ARM launch an interaction attack by using the interface in an
offers TrustZone for isolating sensitive operations within a unexpected way. The main research question that we would

like to answer in this work is the following: SEV technologies by enhancing memory integrity protection,
|     |              |     |     |         |         |          |     | attestation, | and | virtualization |         | capabilities. | In      | particular, | it  |
| --- | ------------ | --- | --- | ------- | ------- | -------- | --- | ------------ | --- | -------------- | ------- | ------------- | ------- | ----------- | --- |
|     |              |     |     |         |         |          |     | tracks       | the | owner          | of each | page of       | memory; |             |     |
| Can | an adversary |     | use | the AMD | SEV-SNP | software |     | •            |     |                |         |               |         |             |     |
interface to force the system into an undesirable state? • utilizes Trusted Computing Base (TCB) versioning to
|     |     |     |     |     |     |     |     | guarantee |             | that guests | run    | under       | up-to-date | firmware; |         |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ----------- | ----------- | ------ | ----------- | ---------- | --------- | ------- |
|     |     |     |     |     |     |     |     | adds      | a versatile |             | remote | attestation | mechanism  |           | where a |
•
Inthiswork,weemploytheTAMARINPROVER(TAMARIN)
|              |              |      |                 |          |        |           |         | third      | party      | may establish |       | trust in | a guest  | during | runtime   |
| ------------ | ------------ | ---- | --------------- | -------- | ------ | --------- | ------- | ---------- | ---------- | ------------- | ----- | -------- | -------- | ------ | --------- |
| protocol     | verification | tool | to meticulously |          | model  | and       | analyze |            |            |               |       |          |          |        |           |
|              |              |      |                 |          |        |           |         | of the     | guest;     |               |       |          |          |        |           |
| the software | interface    | of   | AMD             | SEV-SNP. | The    | model     | encom-  |            |            |               |       |          |          |        |           |
|              |              |      |                 |          |        |           |         | • supports | generating |               | guest | key      | material | from   | different |
| passes all   | key features |      | and captures    | many     | subtle | behaviors | of      |            |            |               |       |          |          |        |           |
sources;
SEV-SNP.Wespecifyandanalyzenearlyahundredproperties enhances the flexibility of live virtual machine migration
•
| and find | several | formal | attacks. | Specifically, | our | contributions |     |                |     |     |        |        |             |        |     |
| -------- | ------- | ------ | -------- | ------------- | --- | ------------- | --- | -------------- | --- | --- | ------ | ------ | ----------- | ------ | --- |
|          |         |        |          |               |     |               |     | by introducing |     | an  | entity | called | a Migration | Agent; |     |
are as follows:
providesseveraladditionalfeaturessuchassecurenested
•
| We  | develop | a formal | model | of  | the AMD |     | SEV-SNP |                 |     |     |     |     |     |     |     |
| --- | ------- | -------- | ----- | --- | ------- | --- | ------- | --------------- | --- | --- | --- | --- | --- | --- | --- |
| •   |         |          |       |     |         |     |         | virtualization. |     |     |     |     |     |     |     |
software interface. The model is close to being fully SEV-SNP prevents a hypervisor from replacing VM mem-
comprehensive; it covers protected guest launch, remote (replay attack)
|     |     |     |     |     |     |     |     | ory with | an old | copy |     |     | or mapping | two | different |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------ | ---- | --- | --- | ---------- | --- | --------- |
attestation, key derivation, page swap and live migration. guestphysicaladdressestoasinglesystemphysicaladdressor
Tothebestofourknowledge,thereexistsnopriorformal DRAM page (memory aliasing attack). It does so by utilizing
| model | of the | SEV-SNP | software |     | interface. |     |     |                  |     |        |         |     |             |     |        |
| ----- | ------ | ------- | -------- | --- | ---------- | --- | --- | ---------------- | --- | ------ | ------- | --- | ----------- | --- | ------ |
|       |        |         |          |     |            |     |     | a data structure |     | called | Reverse | Map | Table (RMP) | and | a page |
• We give formal definitions of, and automated proofs for, validation mechanism to track page ownership and enforce
critical secrecy, authentication, attestation, and freshness proper page access control.
| properties. |     | We show | that | for the | most part, | AMD | SEV- |     |     |     |     |     |     |     |     |
| ----------- | --- | ------- | ---- | ------- | ---------- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
InpreviousSEVinstances,ahypervisorwasassumednotto
SNP software interface does indeed meet the desired bemalicious,butpotentiallybuggy.Likebefore,fromtheper-
| security | goals. | This | includes | the | proof of | correct | stream |     |     |     |     |     |     |     |     |
| -------- | ------ | ---- | -------- | --- | -------- | ------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
spectiveofasingleSNP-protectedguest,otherVMs—whether
cipher usage. non-SNP (legacy) or SNP-protected—are also regarded as
We show that the platform-agnostic nature of SNP- untrustedentities.However,unlikebefore,SEV-SNPtreatsthe
•
| protected |     | guest messages |     | leads | to formal | attacks | on  |            |          |            |     |         |              |     |           |
| --------- | --- | -------------- | --- | ----- | --------- | ------- | --- | ---------- | -------- | ---------- | --- | ------- | ------------ | --- | --------- |
|           |     |                |     |       |           |         |     | hypervisor | as fully | untrusted, |     | capable | of tampering |     | with page |
severalauthenticationandattestationproperties,including tables, injecting arbitrary events, and providing false system
| the | compromise | of  | attestation | report | integrity. |     |     |     |     |     |     |     |     |     |     |
| --- | ---------- | --- | ----------- | ------ | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
information.
• We discuss the challenges of fully mitigating the dis- TheTCBofSEV-SNPcomprisestheAMDSystemonChip,
covered weaknesses while preserving the seamless guest which includes the AMD Security Processor (AMD-SP), and
| VM  | migration | feature. | Additionally, |     | we  | show | that the |              |         |     |        |              |           |     |           |
| --- | --------- | -------- | ------------- | --- | --- | ---- | -------- | ------------ | ------- | --- | ------ | ------------ | --------- | --- | --------- |
|     |           |          |               |     |     |      |          | the software | running |     | on top | of it, which | currently |     | comprises |
vaguespecificationofmigrationagentsisconsistentwith the microcode, bootloader, operating system, and the SNP
ascenariowhereacloudprovidermaytrickathirdparty firmwarewhichimplementstheSNPABI.EachTCBsoftware
into incorrectly believing that an SNP-protected guest is component can be upgraded, and its security version numbers
running on a platform with secure, up-to-date firmware, are included in the TCB_VERSION structure that is associated
| and | suggest | possible | mitigations. |     |     |     |     |           |     |           |     |     |     |     |     |
| --- | ------- | -------- | ------------ | --- | --- | --- | --- | --------- | --- | --------- | --- | --- | --- | --- | --- |
|     |         |          |              |     |     |     |     | with each | SNP | VM image. |     |     |     |     |     |
Paper Outline. We first give the necessary background related SEV-SNP introduces an interface that the SNP-protected
guestmayutilizetorequestservicesfromthefirmwareduring
| toAMDSEV-SNPandthe |     |     | TAMARIN |     | toolinSectionII.Next, |     |     |     |     |     |     |     |     |     |     |
| ------------------ | --- | --- | ------- | --- | --------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
we explain the minutiae of our formal model of AMD SEV- runtime. This enables the guest to obtain—via so-called guest
SNP in Section III. We provide the details of our analysis messages—attestation reports and derived keys, for instance.
|            |        |            |     |         |      |              |     | These messages |     | are encrypted |     | and integrity |     | protected | by the |
| ---------- | ------ | ---------- | --- | ------- | ---- | ------------ | --- | -------------- | --- | ------------- | --- | ------------- | --- | --------- | ------ |
| in Section | IV and | thoroughly |     | discuss | both | the positive | and |                |     |               |     |               |     |           |        |
negative results in Section V; we also suggest possible coun- Virtual Machine Platform Communication Key (vmpck). The
|              |          |     |      |             |     |             |      | firmware | generates | and | installs | this | key into | both | the guest |
| ------------ | -------- | --- | ---- | ----------- | --- | ----------- | ---- | -------- | --------- | --- | -------- | ---- | -------- | ---- | --------- |
| termeasures. | Finally, | we  | give | an overview | of  | the related | work |          |           |     |          |      |          |      |           |
in Section VI and conclude in Section VII. context that it maintains and the guest memory pages during
|     |     |     |     |     |     |     |     | the guest | launch. |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ------- | --- | --- | --- | --- | --- | --- |
InSEVandSEV-ES,theattestationprocedurewasconfined
|     |     | II. | BACKGROUND |     |     |     |     |              |         |     |          |         |      |          |         |
| --- | --- | --- | ---------- | --- | --- | --- | --- | ------------ | ------- | --- | -------- | ------- | ---- | -------- | ------- |
|     |     |     |            |     |     |     |     | to the guest | launch, |     | limiting | its use | to a | singular | entity— |
The core ideas behind SEV-SNP were outlined in a white the guest owner. SEV-SNP introduces a more versatile ap-
| paper published |     | in January | 2020 | [9], and | initially | specified |     | in  |     |     |     |     |     |     |     |
| --------------- | --- | ---------- | ---- | -------- | --------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
proachbyreplacinglaunchattestationwithremoteattestation,
April 2020. In June 2022, Linux 5.19-rc1 was released with enabling any third party to acquire an attestation report and
partial SEV-SNP support [10], and in August 2023, AMD establish trust in a guest at any given moment.
made the SEV-SNP Genoa firmware source code publicly The attestation report comprises various information about
available on GitHub [11]. The current revision of the spec- the guest, such as the guest policy, image digest (measure-
ification was released in September 2023 [12]. ment), the platform (chip identifier) it operates on, and TCB
SEV-SNP provides a novel application binary interface version. It is signed using a private ECDSA P-384 key that is
(ABI) intended for a hypervisor to bootstrap and manage eitheraVersionedChipEndorsementKey(VCEK)—amachine-
virtual machines within a secure environment. It extends specific key derived from chip-unique secrets and a digest
2

of TCB_VERSION—or a Versioned Loaded Endorsement Key While there is no bound on the number of times a fact can be
(VLEK), which is a cloud provider-specific key derived from a produced,linearfactsmodellimitedresourcesthatcanonlybe
seed maintained by the AMD Key Derivation Service (KDS). consumed as many times as they are produced, and persistent
TheinclusionoftheTCBversionallowsathirdpartytoreject facts model unlimited resources, which (once produced) can
the signature if it originated from an unpatched AMD-SP. be consumed any number of times.
In order to further demonstrate the authenticity of the Fresh variables, denoted by the prefix “~” (or suffix
attestation report signature, the VCEK/VLEK is verified against “:fresh”),indicatefreshlygeneratednames.Theyaresuitable
the AMD Signing Key (ASK), which is further verified against for modelling randomly generated values such as keys and
Fr(·)
the AMD Root Key (ARK), both of which are 4096-bit RSA thread identifiers. The built-in fact can be utilized to
keys. generatesuchnames.Publicvariables,identifiedbytheprefix
SEV-SNP introduces a robust key derivation mechanism. It “$” (or suffix “:pub”), are used to represent publicly known
enables an SNP-protected guest to instruct the firmware to namessuchasagentidentitiesandgroupgenerators.Addition-
derive a key rooted in either VCEK, VLEK, or a Virtual Machine ally, temporal variables, prefixed with “#”, signify timepoints.
RootKey(VMRK).Moreover,theguesthastheoptiontorequest The TAMARIN builtins include equational theories for
addingfurtherdata,suchasitslaunchdigestandTCBversion, Diffie-Hellman operations, (a)symmetric encryption, digital
intothekeyderivationprocess.Suchkeysmaybeusedfordata signatures, and hashing. The theory natural-numbers can be
sealing and other purposes. used to model monotonic counters. Additionally, TAMARIN
Similar to previous SEV technologies, SEV-SNP offers an supports user-defined function symbols and equational theo-
interfaceforsecureswappingwhereintheguestmaybesaved ries.Forexample,inourmodel,twoternaryfunctionsymbols,
todiskandlaterresumed.Thefirmwareensuresconfidentiality wrap and unwrap are defined, along with the equation
| and authenticity   | of the guest | memory |          | pages by | utilizing | an      |                    |             |              |        |         |     |                |
| ------------------ | ------------ | ------ | -------- | -------- | --------- | ------- | ------------------ | ----------- | ------------ | ------ | ------- | --- | -------------- |
|                    |              |        |          |          |           |         | 1 unwrap(wrap(msg, |             | nonce, key), | nonce, | key)    | =   | msg            |
| Offline Encryption | Key (OEK).   | The    | swapping | plays    | a         | crucial |                    |             |              |        |         |     |                |
| role in live       | migration.   |        |          |          |           |         |                    |             |              |        |         |     |                |
|                    |              |        |          |          |           |         | which models       | a symmetric | cipher       | with   | a nonce |     | as an initial- |
Live migration is a mechanism by which guests may be ization vector; incrementing the nonce is modelled by using
seamlesslyandsecurelytransferredtoanotherphysicalsystem,
|                |                     |           |          |            |             |      | the built-in | natural-numbers |              | theory. |               |     |          |
| -------------- | ------------------- | --------- | -------- | ---------- | ----------- | ---- | ------------ | --------------- | ------------ | ------- | ------------- | --- | -------- |
| without having | to shut themselves  |           | down     | first.     | Whereas     | in   |              |                 |              |         |               |     |          |
|                |                     |           |          |            |             |      | Consider     | the following   | rule         | (taken  | from          | our | SEV-SNP  |
| SEV and        | SEV-ES the firmware |           | on the   | source     | machine     | was  |              |                 |              |         |               |     |          |
|                |                     |           |          |            |             |      | model and    | simplified)     | that enables | an      | SNP-protected |     | guest to |
| responsible    | for authenticating  | the       | firmware | on the     | destination |      |              |                 |              |         |               |     |          |
|                |                     |           |          |            |             |      | request an   | attestation     | report.      |         |               |     |          |
| machine prior  | to guest context    | transfer, |          | in SEV-SNP | this        | task |              |                 |              |         |               |     |          |
is facilitated by an entity called a Migration Agent. 1 [ StateGvm(’RUNNING’, $image, ~key, %nonce, ~ptr)
AMigrationAgentisanSNP-protectedguestthatisrespon- 2 , In(rd), Fr(~newPtr) ]
|     |     |     |     |     |     |     | 3 −[ ReportRequest($image, |     | rd) | ]→  |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------------- | --- | --- | --- | --- | --- | --- |
sible for enforcing guest migration policies and providing the Out(wrap(< ’MSG_REPORT_REQ’, rd >, %nonce, key))
4 [
firmware with guest-unique secrets (VMRK). Whereas a single , StateGvm(’WAIT’, $image, ~key, %nonce, ~newPtr)
|                  |                    |     |            |               |            |     | 5        |          |      |         |     |     | ]           |
| ---------------- | ------------------ | --- | ---------- | ------------- | ---------- | --- | -------- | -------- | ---- | ------- | --- | --- | ----------- |
| guest may        | be associated with | at  | most       | one migration | agent,     | a   |          |          |      |         |     |     |             |
|                  |                    |     |            |               |            |     | The rule | requires | that | a guest | is  | in  | the RUNNING |
| single migration | agent may          | be  | associated | with          | and manage |     |          |          |      |         |     |     |             |
multiple guests concurrently. This association is indicated in state before it can be executed: it consumes the linear
|     |     |     |     |     |     |     | fact StateGvm(’RUNNING’,$image, |     |     | ~key, | %nonce, |     | ~ptr) from |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------- | --- | --- | ----- | ------- | --- | ---------- |
eachattestationreport.SEV-SNPdoesnotspecifythebehavior
ofmigrationagentsnorthemannerbywhichtheguestcontext the global state, receives an arbitrary message rd (which will
is securely transferred over the network. be included in the attestation report) from the network using
theIn(·)fact,andgeneratesanewpointernewPtrtoitsupdated
|     |     |     |     |     |     |     | state using | the Fr(·) | fact. |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --------- | ----- | --- | --- | --- | --- |
Tamarin Prover
|     |     |     |     |     |     |     | Subsequently, |     | the guest | constructs | a   | plaintext, | which |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --------- | ---------- | --- | ---------- | ----- |
TAMARIN PROVER (TAMARIN) [13] is an automated sym- is an ordered pair <’MSG_REPORT_REQ’, rd > comprising the
bolic verification tool for security protocols. It is based on message tag and received data, encrypts it with a sym-
multiset rewriting; more precisely, its semantics comprises metric key using a %nonce and obtains the ciphertext
a labeled transition system whose states are multisets of wrap(<’MSG_REPORT_REQ’, rd >, key, %nonce), sends the ci-
facts, and the transitions between them are specified by rules phertext to the network using the Out(·) fact, and a new
which prescribe the behavior of protocol participants and the linearfactStateGvm(’WAIT’,$image, ~key, %nonce, ~newptr)is
adversary. Each rule has a left-hand side (facts that must be produced. This fact can now be consumed by a rule wherein
available in the current global state for the rule to execute), the guest receives the attestation report.
actions (labels which are logged in the trace and used to TAMARIN assumes a Dolev-Yao adversary who relays all
express the desired security properties), and a right-hand side messages between protocol participants. More precisely, the
(facts that will be added to the state). KU and KD facts represent the adversary knowledge and its
Theleft-handandright-handsidecontainmultisetsoffacts, ability to receive messages from the network and send mes-
each fact being either linear or persistent (the latter are sages to the network, respectively, by using Out(x)−[]→KD(x)
prefixed with an exclamation point). Facts can be produced and KU(x)−[ K(x) ]→In(x) communication rules (as with all
(when executing a rule with the fact on the right-hand side) actions, the K action is part of the TAMARIN property spec-
and consumed (if it is both linear and on the left-hand side). ification language and can be used to express that the ad-
3

versary necessarily knows the term it sends to the network). properly model the behavior of SNP-protected guests, as they
The adversary can try to deconstruct messages it received can be executed indefinitely, launched with different policies
(constructing any keys needed) in order to gain knowledge and suspended temporarily due to swapping.
of additional terms (e.g. decrypt a ciphertext with the help To facilitate development, we use the general-purpose
m4
of the rule KD(wrap(x,y)), KU(y)−[ ]→KD(x)) and then switch macroprocessor.Thisallowsforefficientprototyping(e.g.,by
viaKD(x)−[ ]→KU(x)toconstructingnewmessagesbyapplying disabling certain precomputations) and provides the flexibility
cryptographic operations on known messages (e.g. applying a to extract multiple variants of the model. Some variants, such
hash function can be done with the rule KU(x)−[ ]→KU(h(x))). as the one obtained with the flag −DIGNORE_ROOT_MD_ENTRY, are
The trace properties of a protocol encoded as a set of intentionally not aligned with the specification; we use them
multiset-rewriting rules are specified as temporal first-order sanity checks, i.e., to confirm the necessity of certain integrity
| formulas | (lemmas) | over | the | rule labels. | For | example, | the | checks. |     |     |     |     |     |     |     |
| -------- | -------- | ---- | --- | ------------ | --- | -------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
following secrecy property states that, for all SNP firmware Our model is based on the specification SEV Secure Nested
threads (fwId), guest VM contexts (gvmCtx), VM Platform Paging Firmware ABI Specification, revision 1.55, published
Communication Keys (vmpck), and timepoints #i and #j, if in September 2023 [12]. AMD upstreamed SEV-SNP guest
a thread with the identifier fwId generates vmpck and installs Linux support [15] and recently published the source code of
it in gvmCtx at some timepoint #i. and if the adversary knows the SEV-SNP Genoa firmware on GitHub [11]. In scenarios
vmpck at #j, then it necessarily holds that there exists some where the specification was not clear enough, we referred to
timepoint #k such thatthe adversary corrupted vmpck at #k and the available implementations.
#k precedes #j: Although AMD has published in the SEV-SNP white pa-
per[9]asetofsecuritythreats(properties)thatitaddresses,we
lemma SecMaVmpckIsSecret:
1 do not consider them in our analysis. This is because they ac-
| 2 ∀ fwId | gvmCtx | vmpck #i | #j. |     |     |     |     |     |     |     |     |     |     |     |     |
| -------- | ------ | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
3 Install(’MA’, fwId, gvmCtx, vmpck)@i ∧ KU(vmpck)@j count for low-level behavior, such as memory aliasing, rather
4 ⇒ ∃ #k. Corrupt(vmpck)@k ∧ (#k < #j) then the specific manner in which the interface is utilized.
|          |        |            |            |               |                |          |          | Since no   | security | properties | are      | defined     | in      | the specification |         |
| -------- | ------ | ---------- | ---------- | ------------- | -------------- | -------- | -------- | ---------- | -------- | ---------- | -------- | ----------- | ------- | ----------------- | ------- |
| TAMARIN  | proves | trace      | properties | by            | falsification. |          | In order |            |          |            |          |             |         |                   |         |
|          |        |            |            |               |                |          |          | either, we | defined  | our own    | set of   | properties. |         |                   |         |
| to prove | that   | a property | is         | true, TAMARIN |                | tries to | find     | a          |          |            |          |             |         |                   |         |
|          |        |            |            |               |                |          |          | Although   | we       | believe    | that the | formal      | attacks |                   | we have |
counterexample execution—an alternating sequence of states identified are consistent with both the specification and its
| and transitions                            |            | that satisfies |       | the negation | of  | the property |       | in              |           |         |                |          |         |            |           |
| ------------------------------------------ | ---------- | -------------- | ----- | ------------ | --- | ------------ | ----- | --------------- | --------- | ------- | -------------- | -------- | ------- | ---------- | --------- |
|                                            |            |                |       |              |     |              |       | implementation, |           | we have | not yet        | been     | able    | to execute | them      |
| question.                                  | If TAMARIN |                | halts | the analysis | and | succeeds,    | then  |                 |           |         |                |          |         |            |           |
|                                            |            |                |       |              |     |              |       | on actual       | hardware. | This is | due to         | the fact | that,   | as of      | the time  |
| theresultingexecutionrepresentsanattack;if |            |                |       |              |     | TAMARIN      | halts |                 |           |         |                |          |         |            |           |
|                                            |            |                |       |              |     |              |       | of writing,     | the       | SEV-SNP | live migration |          | feature | is         | not fully |
theanalysisandfails,thenitprovidesaproofofthepropertyin
|     |     |     |     |     |     |     |     | supported | in open-source |     | tools like | QEMU/KVM. |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | -------------- | --- | ---------- | --------- | --- | --- | --- |
theformofatree.Lastly,TAMARINmaynotterminateasver-
|     |     |     |     |     |     |     |     | Our model, |     | complete | with proofs |     | and documentation, |     | is  |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | -------- | ----------- | --- | ------------------ | --- | --- |
ifyingsecuritypropertiesisingeneralundecidable[14].There
|              |         |          |                 |         |         |                |         | available   | on Gitlab | [16]. |     |     |     |     |     |
| ------------ | ------- | -------- | --------------- | ------- | ------- | -------------- | ------- | ----------- | --------- | ----- | --- | --- | --- | --- | --- |
| are several  | ways    | to avoid | non-termination |         |         | and to         | improve |             |           |       |     |     |     |     |     |
| verification | time;   | we       | customize       | the     | ranking | of proof       | goals   |             |           |       |     |     |     |     |     |
|              |         |          |                 |         |         |                |         | A. Entities |           |       |     |     |     |     |     |
| by writing   | scripts | called   | (proof)         | oracles | and     | use supporting |         |             |           |       |     |     |     |     |     |
lemmas to prove other lemmas. The model can be roughly described in terms of five,
Finally, restrictions can be used to filter out traces that somewhat simplified, interacting state machines. In particular,
| need not   | be considered |                  | during | the             | security | analysis      | or    | to we have: |         |                |             |        |           |        |        |
| ---------- | ------------- | ---------------- | ------ | --------------- | -------- | ------------- | ----- | ----------- | ------- | -------------- | ----------- | ------ | --------- | ------ | ------ |
| enforce    | certain       | behavior         | such   | as verification |          | of signatures | or    |             |         |                |             |        |           |        |        |
|            |               |                  |        |                 |          |               |       | • a state   | machine | that           | describes   | the    | behavior  | of the | SNP-   |
| branching. | One           | such restriction |        | in our          | model    | states that   | every |             |         |                |             |        |           |        |        |
|            |               |                  |        |                 |          |               |       | protected   | Guest   | (GVM)          | as depicted |        | in Figure | 2a;    |        |
| memory     | pointer       | can only         | be     | read before     | it is    | released.     |       |             |         |                |             |        |           |        |        |
|            |               |                  |        |                 |          |               |       | • a state   | machine | that           | describes   | the    | behavior  | of the | Guest  |
| ptr #i     | #j.           | Read(ptr)@i      |        | Free(ptr)@j     |          | #i < #j       |       |             |         |                |             |        |           |        |        |
| 1 ∀        |               |                  | ∧      |                 | ⇒        |               |       | Owner       | (GO)    | as depicted    | in          | Figure | 2b;       |        |        |
|            |               |                  |        |                 |          |               |       | • a state   | machine | that describes |             | the    | behavior  | of the | Migra- |
III. FORMALMODELOFAMDSEV-SNP
|          |     |          |             |     |          |           |     | tion  | Agent    | (MA) as depicted |          | in Figure | 4;       |     |         |
| -------- | --- | -------- | ----------- | --- | -------- | --------- | --- | ----- | -------- | ---------------- | -------- | --------- | -------- | --- | ------- |
| Our goal | is  | to model | and analyze | the | software | interface | of  |       |          |                  |          |           |          |     |         |
|          |     |          |             |     |          |           |     | state | machines | that             | describe | the       | behavior | of  | the SNP |
•
AMDSEV-SNP.Weaimtocaptureallofitsprincipalfeatures,
|               |            |              |          |                 |           |               |      | Firmware |        | (FW): one        | that launches |     | GVMs,    | and | one that |
| ------------- | ---------- | ------------ | -------- | --------------- | --------- | ------------- | ---- | -------- | ------ | ---------------- | ------------- | --- | -------- | --- | -------- |
| including     | remote     | attestation, |          | key derivation, |           | page swapping |      |          |        |                  |               |     |          |     |          |
|               |            |              |          |                 |           |               |      | responds | to     | GVM requests,    |               | as  | depicted | in  | Figure 5 |
| and live      | migration, | while        | ignoring | the             | low-level | details       | such |          |        |                  |               |     |          |     |          |
|               |            |              |          |                 |           |               |      | and      | Figure | 6, respectively. |               |     |          |     |          |
| as the memory |            | encryption   | and      | RMP structure.  |           | We consider   |      | a        |        |                  |               |     |          |     |          |
powerful Dolev-Yao adversary that has full control over the In our model, multiple entities can interact indefinitely, in
communication network in an idealized cryptography setting. an arbitrary order, with the SEV-SNP software interface. To
Before we delve into the intricacies of our model, we briefly support this interaction, we allow an unbounded execution of
explain the methodology we use. each thread, regardless of the program it executes (i.e., the
WeselectedTAMARINPROVERaswidelyacceptedverifica- entity it belongs to). For example, a single MA thread can
tion tool for security protocols which supports loops, branch- manage, i.e. be associated with, an unbounded number of
ing and mutable global state. These features are essential to GVMs. If a thread is capable of executing multiple tasks, it
4

Theper-chipuniquenessofprivVcekisenforcedbyapplying
1 rule ChipCreate[color=colorKDS]:
| let fwVcek  | = kdf(’VCEK’, |              | cek, | $fwTcbVersion) |     |     |     |                 |     |         |     |     |     |     |     |
| ----------- | ------------- | ------------ | ---- | -------------- | --- | --- | --- | --------------- | --- | ------- | --- | --- | --- | --- | --- |
| 2           |               |              |      |                |     |     |     | the restriction |     | Unique. |     |     |     |     |     |
| 3 fwPubVcek |               | = pk(fwVcek) |      |                |     |     |     |                 |     |         |     |     |     |     |     |
4 data = <’VCEK’, askId, $chipId, fwPubVcek> restriction Unique:
1
5 cert = <data, sign(data, privAsk)> x #i #j. Uniq(x)@i Uniq(x)@j #i = #j
| in  |     |     |     |     |     |     |     | 2 ∀ |     |     | ∧   |     | ⇒   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
6
| !LTK(’ASK’, | askId, |     | privAsk), | Fr(cek) |     |     |     |     |     |     |     |     |     |     |     |
| ----------- | ------ | --- | --------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
7 [ ] It requires that the Uniq action shown above be injective, so
| 8 −[ Uniq(<’VCEK’, |     | $chipId>) |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ------------------ | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
9 , ChipGenerateCek($chipId, cek) the restriction ensures that every instance of the PLCreate rule
10 , FwDeriveVcek($chipId, fwVcek) ]→ yields a distinct $chipId.
| !LTK(’VCEK’, |     | $chipId, | fwVcek), |     | CEK($chipId, |     | cek) |                     |     |     |                               |     |     |     |     |
| ------------ | --- | -------- | -------- | --- | ------------ | --- | ---- | ------------------- | --- | --- | ----------------------------- | --- | --- | --- | --- |
| 11 [         |     |          |          |     |              |     |      | Thefact!LTK(’VCEK’, |     |     | ...)maybeusedtostartanynumber |     |     |     |     |
, Out(cert)
| 12  | ]   |         |          |                |     |     |     |                |        |                |     |             |       |     |              |
| --- | --- | ------- | -------- | -------------- | --- | --- | --- | -------------- | ------ | -------------- | --- | ----------- | ----- | --- | ------------ |
|     |     |         |          |                |     |     |     | of FW threads, |        | all running    | on  | the same    | chip; | the | details will |
|     |     |         |          |                |     |     |     | be given       | in the | Initialization |     | subsection. |       |     |              |
|     |     | Fig. 1: | Platform | initialization |     |     |     |                |        |                |     |             |       |     |              |
D. Measurement
canexecutetheminanyorder.Forexample,aGVMthreadcan
|         |              |     |                |     |        |         |            | A guest     | image | measurement |            | (digest) |     | is computed | by the |
| ------- | ------------ | --- | -------------- | --- | ------ | ------- | ---------- | ----------- | ----- | ----------- | ---------- | -------- | --- | ----------- | ------ |
| execute | any sequence |     | of attestation |     | report | and key | derivation |             |       |             |            |          |     |             |        |
|         |              |     |                |     |        |         |            | guest owner | prior | to image    | deployment |          | and | afterwards  | by the |
requests. In some cases, the order in which calls are made firmware during guest launch. In the latter case, the firmware
| may result | in substantially |     | different |     | traces | as they | may update |            |     |             |     |                 |     |        |            |
| ---------- | ---------------- | --- | --------- | --- | ------ | ------- | ---------- | ---------- | --- | ----------- | --- | --------------- | --- | ------ | ---------- |
|            |                  |     |           |     |        |         |            | constructs | the | measurement |     | by initializing |     | a load | digest via |
a shared state. For example, if the GVM gets swapped out an SNP_LAUNCH_START call and subsequently updating it with
| immediately  | after        | it receives |      | a derived | key, | its encrypted | page |                   |                   |         |      |              |         |          |             |
| ------------ | ------------ | ----------- | ---- | --------- | ---- | ------------- | ---- | ----------------- | ----------------- | ------- | ---- | ------------ | ------- | -------- | ----------- |
|              |              |             |      |           |      |               |      | SNP_LAUNCH_UPDATE |                   | calls.  | Each | update       | saves   | a        | hash of the |
| (state) will | contain      | the         | key. |           |      |               |      |                   |                   |         |      |              |         |          |             |
|              |              |             |      |           |      |               |      | PAGE_INFO         | structure,        | which   |      | transitively | binds   | the      | contents of |
|              |              |             |      |           |      |               |      | each individual   |                   | page to | the  | digest.      |         |          |             |
| B. Key       | Distribution | Service     |      |           |      |               |      |                   |                   |         |      |              |         |          |             |
|              |              |             |      |           |      |               |      | When              | SNP_LAUNCH_FINISH |         |      | is           | called, | assuming | that        |
The rules KDSCreateARK and KDSCreateASK model a part of ID_BLOCK_EN is set, the firmware checks whether the
the AMD Key Distribution Service (KDS). This includes the computed measurement matches the one specified by the
generationoftheAMDRootKey,denotedasprivArk,andthe guest owner; if it does not, the firmware refuses to launch a
|             |      |         |     |             |     |                |     | guest and | returns | BAD_MEASUREMENT. |     |     |     |     |     |
| ----------- | ---- | ------- | --- | ----------- | --- | -------------- | --- | --------- | ------- | ---------------- | --- | --- | --- | --- | --- |
| AMD Signing | Key, | denoted |     | as privAsk. |     | Each long-term | key |           |         |                  |     |     |     |     |     |
(including privVcek which is discussed next) is available via Note the dual nature of a guest image as both a program,
the fact !LTK and certified by the next key in the hierarchy, which is intended to be executed on a virtual machine, and
|            |          |       |            |     |     |         |           | data that | can be | measured. |     | In order | for us | to directly | model |
| ---------- | -------- | ----- | ---------- | --- | --- | ------- | --------- | --------- | ------ | --------- | --- | -------- | ------ | ----------- | ----- |
| except for | privArk, | which | represents |     | the | root of | trust and | is        |        |           |     |          |        |             |       |
self-signed.Forthepurposeofproducingandverifyingdigital boththedynamicandstaticnatureoftheimage,themodelling
|             |     |        |     |          |         |        |          | language | would | need | to support | metaprogramming |     |     | features |
| ----------- | --- | ------ | --- | -------- | ------- | ------ | -------- | -------- | ----- | ---- | ---------- | --------------- | --- | --- | -------- |
| signatures, | we  | employ | the | built-in | message | theory | signing, |          |       |      |            |                 |     |     |          |
whichexportsfunctionsymbolssign,verify,pk,true;theyare thatwouldallowustotreatprogramsasdata.Tothebestofour
related by the equation verify(sign(m,sk),m,pk(sk)) = true. knowledge, none of the current security protocol verification
|            |     |                 |     |         |           |     |        | tools offers | such | support. |     |     |     |     |     |
| ---------- | --- | --------------- | --- | ------- | --------- | --- | ------ | ------------ | ---- | -------- | --- | --- | --- | --- | --- |
| We publish |     | the certificate |     | of each | long-term |     | key to | the          |      |          |     |     |     |     |     |
networkusingOutfacts,andmaketherootcertificateavailable We employ the following approach: to model an image
!Cert(...) as data, we use a public variable $image, and constants
| to a guest | owner | (GO) | via |     | fact. | Furthermore, |     | we  |     |     |     |     |     |     |     |
| ---------- | ----- | ---- | --- | --- | ----- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
allow the adversary to compromise the KDS and extract the ’5XPYKIAXFS06O’ and ’3A9B8C7D1E2F’ to represent specific im-
keys using the RevealARK and RevealASK rules. ages assigned to a guest (GVM) and a migration agent (MA),
respectively.Tomodelanimageasaprogram,weuseasetof
C. Platform multiset-rewriting rules and prefix each rule belonging to the
|     |     |     |     |     |     |     |     | respective | images | with | GVM | and MA. | By  | doing so | we assign |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | ------ | ---- | --- | ------- | --- | -------- | --------- |
Wemodelthecreation,initialization(SNP_INIT)andconfig-
|                      |               |     |             |          |            |         |            | a predefined    | load    | digest          | to  | each        | image  | we consider | (e.g.,       |
| -------------------- | ------------- | --- | ----------- | -------- | ---------- | ------- | ---------- | --------------- | ------- | --------------- | --- | ----------- | ------ | ----------- | ------------ |
| uration (SNP_CONFIG) |               | of  | the         | platform | by using   | the     | ChipCreate |                 |         |                 |     |             |        |             |              |
|                      |               |     |             |          |            |         |            | h(< ’VM_IMAGE’, |         | ’5XPYKIAXFS06O’ |     | >)).        |        |             |              |
| and PlCreate         | rules.        |     | This former |          | rule       | (Figure | 1) binds   | a               |         |                 |     |             |        |             |              |
|                      |               |     |             |          |            |         |            | Whenever        | a       | launches        |     | a           | or MA, | the         | behavior of  |
|                      |               |     |             |          |            |         |            |                 |         | FW              |     | GVM         |        |             |              |
| firmware             | to a specific |     | chip,       | which    | is denoted | by      | chipId:    | it              |         |                 |     |             |        |             |              |
|                      |               |     |             |          |            |         |            | the launched    | virtual | machine         |     | is governed |        | by a set    | of multiset- |
usesafreshlygenerated,chip-unique,long-termsecretcek,the
|          |          |          |               |     |     |          |         | rewriting | rules  | which  | is fixed   | in advance. |          | While | tampering    |
| -------- | -------- | -------- | ------------- | --- | --- | -------- | ------- | --------- | ------ | ------ | ---------- | ----------- | -------- | ----- | ------------ |
| value of | a public | variable | $fwTcbVersion |     |     | (the TCB | version | of        |        |        |            |             |          |       |              |
|          |          |          |               |     |     |          |         | with the  | images | is not | supported, |             | we allow | the   | adversary to |
thefirmwareimage)andakeyderivationfunctionkdftoderive
|                |                        |     |                |      |          |        |              | compromise | the | secrets | contained | in  | certain | images. |     |
| -------------- | ---------------------- | --- | -------------- | ---- | -------- | ------ | ------------ | ---------- | --- | ------- | --------- | --- | ------- | ------- | --- |
| an attestation | signing                |     | key, privVcek. |      | We       | model  | side-channel |            |     |         |           |     |         |         |     |
| attacks        | on its confidentiality |     |                | with | the help | of the | ExtractCek   |            |     |         |           |     |         |         |     |
E. Initialization
rule.
The latter rule creates a pair of associated migration agent Uponstartupandbeforeinitiatinganyprimaryguestlaunch,
images, and bind each to a particular platform. FW spawns a Migration Agent (MA) background thread ca-
Note that firmware updates (DOWNLOAD_FIRMWARE) are not pable of managing guests associated with it. The MA thread
supported; we represent all of CurrentTcb, and should be able to migrate only those guests associated with it
ReportedTcb
LaunchTcb by $fwTcbVersion, whose value (once initialized) during launch, where the association with a particular guest
persists throughout the lifetime of a chip, and consequently depends on the migration policy of that guest. However,
the lifetime of privVcek. because it is constantly running in the background, the MA
5

GO
GVM
INITIALIZE
SetmigrationpolicygvmMigPolicy
|     |     |     |     |                      | IDLE |     |     |     |     |     | Calculatelaunchdigest             |     |     |     |     |
| --- | --- | --- | --- | -------------------- | ---- | --- | --- | --- | --- | --- | --------------------------------- | --- | --- | --- | --- |
|     |     |     |     | Deletekey(ifderived) |      |     |     |     |     |     | Prepareimage,signidBlock,setidKey |     |     |     |     |
SendDEPLOY_REQ
ReceiveREPORT_DATA(GO)
|     |     | RUNNING |     |     |     |     | RUNNING |     |     |     |     |     |     |     |     |
| --- | --- | ------- | --- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
REPORT_REQUEST
|     | Selectrootvmrk         |     |     |     |     | SetreportData=nonce |     |     |     |     |                 |     |     |     |     |
| --- | ---------------------- | --- | --- | --- | --- | ------------------- | --- | --- | --- | --- | --------------- | --- | --- | --- | --- |
|     | Mixvmpl+hostData+idKey |     |     |     |     | WriteMSG_REPORT_REQ |     |     |     |     | Generatenonce   |     |     |     |     |
|     | WriteMSG_KEY_REQ       |     |     |     |     |                     |     |     |     |     | SendREPORT_DATA |     |     |     |     |
ReadMSG_REPORT_RSP
ReadMSG_KEY_RSP
|     |                 |             |     |     |     |                 | REPORT_REQUEST |     |     | ReceiveREPORT_CERT |                           |               |     |     |     |
| --- | --------------- | ----------- | --- | --- | --- | --------------- | -------------- | --- | --- | ------------------ | ------------------------- | ------------- | --- | --- | --- |
|     |                 | KEY_REQUEST |     |     |     | SendREPORT_CERT |                |     |     |                    |                           | REPORT_VERIFY |     |     |     |
|     | Savekeyinmemory |             |     |     |     |                 |                |     |     |                    | VerifycertificatewithVCEK |               |     |     |     |
Fig. 2a: SNP-protected Guest State Machine Fig. 2b: Guest Owner State Machine
| thread       | may attempt | to          | initiate | migration   |          | of any    | guest     | at  | 1 rule FwLaunchMa: |                |     |     |     |     |     |
| ------------ | ----------- | ----------- | -------- | ----------- | -------- | --------- | --------- | --- | ------------------ | -------------- | --- | --- | --- | --- | --- |
| any time     | and         | potentially | violate  | attestation |          | report    | integrity |     | 2 let              |                |     |     |     |     |     |
|              |             |             |          |             |          |           |           |     | nonce              | = %1           |     |     |     |     |     |
| if the guest | migration   |             | policy   | is not      | properly | enforced. |           | We  | 3                  |                |     |     |     |     |     |
|              |             |             |          |             |          |           |           |     | ma =               | <image, ~maId> |     |     |     |     |     |
4
prove that such violations are not possible in our model 5 image = ’3A9B8C7D1E2F’
| (AttReportIntegrityMd). |     |          |     |        |                 |     |        |     | 6 ld = | h(<’MA_IMAGE’, | image>) |            |         |     |      |
| ----------------------- | --- | -------- | --- | ------ | --------------- | --- | ------ | --- | ------ | -------------- | ------- | ---------- | ------- | --- | ---- |
|                         |     |          |     |        |                 |     |        |     | maCtx  | = <ld, ~vmpck, | %nonce, | ~reportId, | assocPl |     | ...> |
| By allowing             |     | a single |     | thread | to concurrently |     | manage |     | 7      |                |         |            |         |     |      |
MA
8 ...
| multipleguests,weaddressscenarioswheretheadversarymay |     |     |     |     |     |     |     |     | 9 in |     |     |     |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
replay the thread messages to multiple guests associated10 [ Platform(plNum, chipId, fwVcek, ma, assocPl)
MA
|          |         |               |     |               |     |       |         | to11 | , Fr(~vmpck), | Fr(~reportId), |     | Fr(~fwId), | Fr(~maStPtr) |     | ]   |
| -------- | ------- | ------------- | --- | ------------- | --- | ----- | ------- | ---- | ------------- | -------------- | --- | ---------- | ------------ | --- | --- |
| with the | thread. | For instance, |     | the adversary |     | might | be able |      |               |                |     |            |              |     |     |
|          |         |               |     |               |     |       |         |      | 12 −[...]→    |                |     |            |              |     |     |
reuse the MSG_VMRK_REQ message and install the same Virtual !StateFwMa(’RUNNING’, ~maStPtr, maCtx,...)
|     |     |     |     |     |     |     |     |     | 13 [ |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
Machine Root Key (vmrk) in more than one guest context,14 , StateMa(~fwId, ’IDLE’, ~vmpck, nonce, maId,...)
|             |               |            |            |         |              |         |              | key15 | , VMPCK(~fwId, | ~maId,       | ~vmpck)... |                | ]   |        |     |
| ----------- | ------------- | ---------- | ---------- | ------- | ------------ | ------- | ------------ | ----- | -------------- | ------------ | ---------- | -------------- | --- | ------ | --- |
| thus making | each          | of         | the guests | later   | derive       | the     | same         |       |                |              |            |                |     |        |     |
| rooted in   | vmrk;         | protecting | against    | such    | behavior     |         | is important |       |                |              |            |                |     |        |     |
|             |               |            |            |         |              |         |              |       | Fig.           | 3: Migration | Agent      | initialization | and | launch |     |
| because     | the derived   | key        | may        | be used | e.g.         | for key | sealing.     | We    |                |              |            |                |     |        |     |
| prove that  | the described |            | behavior   | is      | not possible |         | in our       | model |                |              |            |                |     |        |     |
(FreshKeyDerFromFwVmrkIsGvmUnique).
|     |     |     |     |     |     |     |     |     | the sequence | of rules | FwProvisionGvm, |     | FwAssociateGvmMa |     | and |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------ | -------- | --------------- | --- | ---------------- | --- | --- |
TheFwLaunchMarule,asshowninFigure3,isanabstraction
of all the SNP_LAUNCH commands. It models the launch of an FwLaunchGvmAssocMa, or the sequence of rules FwProvisionGvm
thread by FW. The thread context maCtx is initialized and FwLaunchGvmNoAssocMa, depending on whether migration is
| MA  |     |     | MA  |     |     |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
allowedornot(gvmMigPolicy).Thelaunchprocedurehassimi-
| with several | fresh | values: | a   | firmware | thread | identifier |     | fwId, |     |     |     |     |     |     |     |
| ------------ | ----- | ------- | --- | -------- | ------ | ---------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
which uniquely identifies a thread; an attestation report laritiestothatof MA;hereweonlyemphasizethedifferences.
FW
|            |           |       |       |     |             |     |        |      |     | reads an image | ($image), | an  | ID Block | (idBlock), | and |
| ---------- | --------- | ----- | ----- | --- | ----------- | --- | ------ | ---- | --- | -------------- | --------- | --- | -------- | ---------- | --- |
| identifier | reportId, | which | binds | an  | attestation |     | report | to a | FW  |                |           |     |          |            |     |
specific guest instance; a secret Virtual Machine Platform an ID Authentication Information Structure (idAuth) from the
Communication Key vmpck and the corresponding message guest owner (GO) using the FwProvisionGvm rule. An idBlock
includesthemigrationpolicy(gvmMigPolicy)fortheguest.An
| counter | nonce, which | are | used | to establish |     | a secure | communi- |     |     |     |     |     |     |     |     |
| ------- | ------------ | --- | ---- | ------------ | --- | -------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
cation channel between and FW; and a pointer maStPtr idAuth pair consists of an ID block signature (idBlockSig)
GVM
|          |       |            |           |     |      |            |         |     | and the | public part | of a | GO-provided | identity | key | (pubIdk), |
| -------- | ----- | ---------- | --------- | --- | ---- | ---------- | ------- | --- | ------- | ----------- | ---- | ----------- | -------- | --- | --------- |
| to maCtx | which | is freshly | generated |     | upon | each maCtx | update. |     |         |             |      |             |          |     |           |
The rule produces the state facts that establish a secure which is used to produce the signature. The FW validates
communication channel between and MA. More precisely, the launch digest (as described in Section III-D) and ensures
FW
signatureverificationusingtheEq(...)actionfactandEquality
| the persistent | fact      | !StateFwMa(...) |          |           | represents | the     | part      | of the |              |     |     |     |     |     |     |
| -------------- | --------- | --------------- | -------- | --------- | ---------- | ------- | --------- | ------ | ------------ | --- | --- | --- | --- | --- | --- |
| FW thread      | state     | relevant        | to       | launching | guests     | and     | enforcing |        | restriction. |     |     |     |     |     |     |
| migration      | policies. | It              | contains | the       | thread     | context |           | maCtx, |              |     |     |     |     |     |     |
MA
|     |     |     |     |     |     |     |     |     | 1 restriction | Equality: |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --------- | --- | --- | --- | --- | --- |
whichisreadandupdatedduringexecution(e.g.,toincrement
|             |           |     |      |      |          |     |        |      | 2 ∀ x y | #i. Eq(x, y)@i | ⇒   | x = y |     |     |     |
| ----------- | --------- | --- | ---- | ---- | -------- | --- | ------ | ---- | ------- | -------------- | --- | ----- | --- | --- | --- |
| the message | counter). |     | Both | that | fact and | the | linear | fact |         |                |     |       |     |     |     |
StateMa(...)includeastatename(e.g.,’IDLE’)asaparameter TheFWinstallsinaguestcontextgvmCtxseveral(fresh)val-
that is updated as the state machine is executed (cf. Figure 4). ues, including: an Offline Encryption Key oek, which is used
| The rule | also | outputs | a VMPCK(...) |     | fact, | which | enables | the |     |     |     |     |     |     |     |
| -------- | ---- | ------- | ------------ | --- | ----- | ----- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
toencryptcontentsofguestpagesthathavebeenswapped-out;
adversary to corrupt and extract the communication key. aVirtualMachineRootKeyvmrk,whichthederivedkeysmay
berootedin;apointergvmStPtrtoagueststatewhichisfreshly
F. Guest Launch
|     |     |     |     |     |     |     |     |     | generated | for each | guest state | machine | transition; | a   | platform- |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --------- | -------- | ----------- | ------- | ----------- | --- | --------- |
A simplified guest launch procedure is depicted in Fig- launch identifier pid which persists throughout the lifetime of
ure 5. An SNP-protected guest (GVM) is launched using a guest on a particular platform; and a session identifier sid
6

whichisfreshlygenerateduponeachguestswap.Additionally, which involves reading the state, such as FwProvisionGvm, the
we use the variable gvmIsMig as a flag to indicate whether the same fact must appear on both sides of the rule (if the rule
guest has been migrated or not, and the variable authTag to needed to update the state instead, the fact on the right-hand
store the authentication tag of the swapped-out guest state. side would contain the updated values). This results in non-
The specification mandates that the firmware rejects the termination when inductively proving some true statements,
pages donated by a hypervisor via an SNP_GCTX_CREATE call such as SupNoKeyNonceReuseMA, where TAMARIN perpetually
if they are not in the Firmware state. Given that a single loops over the FwProvisionGvm rule in an unsuccessful effort
MA thread may manage multiple GVMs concurrently, each to apply the induction hypothesis.
MA message includes a guest context address to distinguish However, using a persistent fact makes it harder to update
individual guests, as can be seen in the Figure 4. We link guest management states (although a new version of the fact
a guest context with an address $gctxAddr and ensure that it can be added to the global state, the old version can never be
is not already assigned on the platform (chipId) by applying removed). Updates are required, for example, to increment a
the Unique restriction; otherwise, the adversary can replay the message counter when FW services the MSG_VMRK_REQ request
MSG_VMRK_REQguestmessage,prompting FW toinstallthesame via the FwLaunchGvmAssocMa rule. We can fortunately employ a
vmrk in multiple contexts. trickusedforthispurposebyCremersetal.[17]:wegenerate
The produced state facts StateGvm(...) and StateFwGvm(...) a fresh value that serves as a pointer to the persistent fact, i.e.
allow GVM to request remote attestation and key derivation the memory represented by the fact, which can then be read
services via a secure communication channel, and FW to via Read(...) actions and released via Free(...) actions. We
provide these services (note that FW uses two separate state can then simply allocate new memory by generating a new
facts for execution: StateFwGvm(...) and !StateFwMa(...)). The persistent fact and a fresh pointer to it.
two states are bound by FW, who simply installs the reportId To ensure that memory is always read before it is released
of MA into the guest context as maReportId. Whenever FW and that it cannot be released more than once, we use the
receives a request from MA to manage a guest, FW checks respective two restrictions.
whetherthereportIdintheMAcontextmatchesthemaReportId
in the GVM context and refuses to service the request other- 1 restriction FreedMemoryCannotBeRead:
2 ∀ ptr #i #j. Read(ptr)@i ∧ Free(ptr)@j ⇒ #i < #j
wise. The check is performed by the rule FwLaunchGvmAssocMa,
3
wherein FW receives the vmrk request from the MA and 4 restriction MemoryCanBeFreedOnlyOnce:
finalizes the guest launch procedure. 5 ∀ ptr #i #j. Free(ptr)@i ∧ Free(ptr)@j ⇒ #i = #j
G. Guest messages
I. Remote Attestation
SNP-protected guests can utilize the SNP_GUEST_REQUEST
Each GVM may send any number of attestation re-
command to securely communicate with the AMD-SP
port requests (MSG_REPORT_REQ messages) to FW via the
firmware. All exchanged messages are tagged, encrypted and
GVMRequestsReport rule, regardless of whether the guest is
integrity-protected using AES-256-GCM with the Virtual Ma-
migrated or not. Each request includes the reportData field—
chine Platform Communication Key (vmpck) that is injected
an arbitrary value provided by the GVM and uninterpreted by
intoguestpagesthroughtheSNP_LAUNCH_UPDATEcall.Guests,
theFW.TheGOutilizesittoensureattestationreportfreshness.
whether primary or migration agents, employ messages for
This is shown in Figure 2a and Figure 2b.
tasks like obtaining attestation reports, deriving keys, and
Whilethespecificationaccommodatesguestswithanoption
managing migration.
to select either VCEK or VLEK as a report signing key, we only
We use the built-in natural−numbers theory in TAMARIN to
ever use VCEK for that purpose (i.e., privVcek in the model).
represent nonces. The theory features the constant 1:nat and
FWassemblesanattestationreport,asillustratedinFigure6,
an associative-commutative union operator %+, which suffice
by utilizing the FWGeneratesReport rule. The report comprises
to model nonces via monotonically increasing counters.
various information from the guest context including a chip
GVM and MA each maintain their own message counter
identifier (chipId), a migration policy (gvmMigPolicy), and a
(msgCount), while FW maintains two separate message
report identifier (reportId). An attestation report is always
counters—one for GVM and MA each. They are initialized to
signed with privVcek and forwarded by GVM to the public
1:nat and incremented upon each successful message receipt
network, where it is available for inspection by GO.
(as in, e.g., FwLaunchGvmAssocMa).
H. Guest Context J. Key Derivation
FW threads use !StateFwMa(...) state facts to store con- GVM may also ask FW to derive and provide a key by
texts of MA threads for the purpose of guest management. sending the MSG_KEY_REQ guest message, as depicted in Fig-
In addition, each FW thread may spawn any number of ure 2a, using the GVMRequestsKey rule. The keys are derived
StateFwGvm(...) state facts used to store the contexts of GVM by applying the ternary function symbol kdf to a root key and
threads, which enables the provision of services to guests. additional data. This is also shown in Figure 6.
Note that we represent each guest management state by a TherootkeyisthevmrkofMAorFW,dependingonwhether
persistentfact;otherwise,ifweusealinearfact,theninarule GVM is associated with MA or not.
7

MA
IDLE
ReadgvmCtxfromMAand
gctxAddrfromHV
RUNNING
RUNNING RUNNING
W Inc ri l t u e de MS g G c _ t E x X A P d O d R r T_REQ G W In e c r n i l t u e e d ra e M t S e g G c v _ t m V x r M A k R d K d _ r REQ W Inc ri l t u e de MS d G o _ n I at M e P d O g R c T t _ x R A E d Q dr
ReadMSG_EXPORT_RSP
ReadMSG_VMRK_RSP
ReadMSG_IMPORT_RSP
EXPORT_REQUEST IMPORT_REQUEST
VMRK_REQUEST
MigrategvmCtxtoassociatedMA
viaestablishedsecurechannel
Fig. 4: Migration Agent State Machine
FW
IDLE
ReceiveDEPLOY_REQ ReadMSG_IMPORT_REQ
RUNNING RUNNING
Installvmpck,vmrk,oek,reportIdtogvmCtx GenerateandinstallreportIdtogvmCtx
VerifysigIdBlock InstallgvmCtxtogueststate
IncludeidBlockandidAuthtogvmCtx WriteMSG_IMPORT_RSP
gvmMigPolicy = 0 gvmMigPolicy = 1 ReadSNP_PAGE_SWAP_IN
INITIALIZE INITIALIZE IMPORT
ExecuteGVM A In s s s t o al c l ia M t A e ’s GV r M ep w o i r th tI M d A togvmCtx V st er a i t fy e a = ut s h d t e a c g (e =? nc g S v n m a C p t s x h . o a t u , t o h e T k a ) g
ReadMSG_VMRK_REQ ExecuteGVM
ASSOCIATE
VerifyreportId=? gvmCtx.maReportId
ReplacevmrkingvmCtxwithMA’s
WriteMSG_VMRK_RSP
ExecuteGVM
Fig. 5: AMD Security Processor Firmware State Machine (guest launch)
The acquired key is deleted by GVM before the subsequent binary function symbol mac for the purpose of producing
request. However, the adversary can gain knowledge of the authentication tags.
oek-encrypted key if it swaps out GVM while it is still in the
During the swap-out process, FW encrypts the contents of
IDLEstate.Infact,theconfidentialityofaderivedkeydepends
the GVM state with oek to obtain the encrypted snapshot
ontheconfidentialityoffivedifferentkeysaswewillseelater.
(gvmEncSnapshot) of guest’s memory, which it subsequently
uses to calculate the authentication tag authTag. While both
K. Page Swap
gvmEncSnapshot and authTag are then transmitted over the
The model incorporates a swapping mechanism where an network, authTag is saved in the GVM context. Additionally,
adversary may use SNP_PAGE_SWAP_OUT and SNP_PAGE_SWAP_IN StateGvm(...)andStateFwGvm(...)statefactsareparameterized
commandstoswapoutandswapinGVManynumberoftimes. with a fresh session identifier ~sid to facilitate backward
The corresponding state machine is visualized in Figure 6. search.
We do not explicitly model memory pages of guests or VM
Conversely, during the swap-in process, FW first receives
Encryption Keys (VEK) used to encrypt them. Instead, we
gvmEncSnapshotandauthTagfromthenetwork.Itthendecrypts
represent GVM memory through StateGvm(...) state facts, and
the GVM state contents with oek and verifies authTag by com-
we model decryption and encryption with VEK through the
paringitagainstthesavedoneusingtheEqualityrestriction.If
consumption and production of the StateGvm(...) state facts.
thechecksucceeds,theGVMthreadmaycontinueitsexecution
FW utilizesfourrulesFwSwapOutGvm(BM)andFwSwapInGvm(BM)to
afterwards.Wecandemonstrate(asdescribedinSectionIV-A)
swap out and swap in GVM; employing two almost identical
that ignoring this check leads to a rollback attack where the
pairsofrulesforeachpurpose(BMstandsforBeforeMigration)
same key and nonce get reused.
facilitates our modelling of the migration procedure.
We abstract away from the details of stream cipher en- Forterminationreasons,weprohibitthe FW fromswapping
cryption with the OEK. Instead, we use the built-in the- out the same GVM state more than once. Each GVM transition
ory symmetric−encryption which defines two binary func- generatesafreshpointer,denotedasgvmStPtr,whichuniquely
tion symbols senc and sdec related by the equation identifiesaparticularstate.Toensurethatthestateisswapped
sdec(senc(m,k),k)=m. Moreover, we introduce an irreducible out at most once, we enforce the following restriction:
8

1 restriction MemoryCanBeSwappedOnlyOnce: the utilizes the FwImportGvm rule to add the guest con-
FW
| 2 ∀ k1 | k2 guest1 | guest2 | state1 | state2 | pointer | #i  | #j. |     |     |     |     |     |     |     |     |
| ------ | --------- | ------ | ------ | ------ | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Swap(k1, guest1, state1, pointer)@i text to the StateFwGvm(...) state fact and update it with a
| 3   |     |     |     |     |     | ∧   |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Swap(k2, guest2, state2, pointer)@j ⇒ #i = #j freshly generated reportId. Additionally, the FW establishes
4
|     |        |          |      |       |              |        |        | the association    |     | between |                                  | and | by incorporating |     | the |
| --- | ------ | -------- | ---- | ----- | ------------ | ------ | ------ | ------------------ | --- | ------- | -------------------------------- | --- | ---------------- | --- | --- |
|     |        |          |      |       |              |        |        |                    |     |         | MA                               | GVM |                  |     |     |
| We  | do not | consider | this | to be | a limitation | since, | in our |                    |     |         |                                  |     |                  |     |     |
|     |        |          |      |       |              |        |        | reportIdvalueofthe |     | MA      | intotheguestcontextasmaReportId. |     |                  |     |     |
model,theadversarygainsnoadvantagebyobtainingthesame Following this, is swapped in using the FwSwapInGvmAM
GVM
| ciphertext    | multiple     | times.                 |     |         |          |              |     |                    |          |            |               |           |            |              |     |
| ------------- | ------------ | ---------------------- | --- | ------- | -------- | ------------ | --- | ------------------ | -------- | ---------- | ------------- | --------- | ---------- | ------------ | --- |
|               |              |                        |     |         |          |              |     | rule and           | launched | afterward. |               |           |            |              |     |
| For           | performance  | reasons,               |     | we also | prohibit | swapping     | out |                    |          |            |               |           |            |              |     |
| GVM           | while        | it is in               | the | RUNNING | state    | by employing | the |                    |          |            |               |           |            |              |     |
| following     | restriction: |                        |     |         |          |              |     | M. Secure          | Channel  |            |               |           |            |              |     |
|               |              |                        |     |         |          |              |     | Migration          | agents   | employ     | ComChannelOut |           | and        | ComChannelIn |     |
| 1 restriction |              | GuestStateDuringASwap: |     |         |          |              |     |                    |          |            |               |           |            |              |     |
|               |              |                        |     |         |          |              |     | rules, illustrated |          | in Figure  | 7,            | to enable | the secure | transfer     | of  |
| 2 ∀ kind      | guest        | state pointer          |     | #i.     |          |              |     |                    |          |            |               |           |            |              |     |
3 Swap(kind, guest, state, pointer)@i ⇒ state = ’IDLE’ guestcontextsbetweenplatforms.Thisprocessinvolvesusing
|     | state | = ’KEY_REQ’ |     | state | = ’REPORT_REQ’ |     |     |     |     |     |     |     |     |     |     |
| --- | ----- | ----------- | --- | ----- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
4 ∨ ∨ ComChan__(...) state facts to link OutChan(...) and InChan(...)
|                                          |              |                 |           |         |              |           | IDLE      | state facts, | for          | the purpose | of          | sending   | and receiving |         | a guest   |
| ---------------------------------------- | ------------ | --------------- | --------- | ------- | ------------ | --------- | --------- | ------------ | ------------ | ----------- | ----------- | --------- | ------------- | ------- | --------- |
| Wealsodonotregardthisasalimitationsincea |              |                 |           |         |              |           | GVM       |              |              |             |             |           |               |         |           |
|                                          |              |                 |           |         |              |           |           | context.     | This         | use of      | state facts | ensures   | that          | the     | adversary |
| state, which                             | may          | be swapped      |           | out,    | differs from | a RUNNING | state     |              |              |             |             |           |               |         |           |
|                                          |              |                 |           |         |              |           |           | can neither  | modify       | nor         | learn       | messages  | that are      | sent    | over the  |
| only in                                  | the deletion | of              | a derived | key     | (modelled    | by        | replacing |              |              |             |             |           |               |         |           |
|                                          |              |                 |           |         |              |           |           | channel.     | Furthermore, |             | the use     | of linear | facts         | ensures | that the  |
| the key                                  | with         | the kdf(’NULL’, |           | ’NULL’, | ’NULL’)      | fact).    |           |              |              |             |             |           |               |         |           |
|                                          |              |                 |           |         |              |           |           | messages     | sent         | cannot be   | replayed    | at a      | later point   | in      | time.     |
The−DENABLE_REPLAY_OVER_COMMflagselectsamodelvariant
L. Live Migration
|     |     |     |     |     |     |     |     | that permits | message | replay | over | the | channel. | In this | model, |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | ------- | ------ | ---- | --- | -------- | ------- | ------ |
SEV-SNP offers several methods for migrating SNP- a persistent fact !ComChan__(...) is used instead of a linear
protected guests, depending on whether the assistance of a fact ComChan__(...). We demonstrate how the adversary may
migrationagentoraguestisutilized.Inourmodel,wepermit exploitthistoitsadvantageandcompromisevmpckbyutilizing
at most two migrations and do not consider guest-assisted the ExeAttackReplayOverCommChan executable lemma.
migration.
| The    | state     | machine | is        | depicted | in          | Figure | 4. The     |                |     |       |     |     |     |     |     |
| ------ | --------- | ------- | --------- | -------- | ----------- | ------ | ---------- | -------------- | --- | ----- | --- | --- | --- | --- | --- |
|        | MA        |         |           |          |             |        | MA         |                |     |       |     |     |     |     |     |
|        |           |         |           |          |             |        |            | N. Adversarial |     | Model |     |     |     |     |     |
| thread | is tasked | with    | providing | a        | vmrk during | the    | launch and |                |     |       |     |     |     |     |     |
migrating guests to other platforms. Each migratable GVM, as In accordance with the AMD SEV-SNP threat model, we
|           |     |                  |     |       |             |      |           | assume | that the | adversary | has | control | over a | hypervisor | and |
| --------- | --- | ---------------- | --- | ----- | ----------- | ---- | --------- | ------ | -------- | --------- | --- | ------- | ------ | ---------- | --- |
| indicated | by  | the gvmMigPolicy |     | flag, | is assigned | to a | single MA |        |          |           |     |         |        |            |     |
on a particular platform. Conversely, a single MA thread can a cloud provider and is able to launch an arbitrary number
manage an arbitrary number of primary guests. itself is of SNP-protected VMs, issue ABI commands in any order,
MA
not migratable. spy on the communication outputs, and tamper with the
Asmandatedbythespecification,GVMisswappedoutprior communication inputs. Moreover, the adversary may corrupt
|              |     |           |                |     |       |           |         | both the | SNP | firmware | and | guests, | and extract | keys | from |
| ------------ | --- | --------- | -------------- | --- | ----- | --------- | ------- | -------- | --- | -------- | --- | ------- | ----------- | ---- | ---- |
| to migration |     | using the | FwSwapOutGvmBM |     | rule. | Migration | is then |          |     |          |     |         |             |      |      |
initiated by MA, which sends the MSG_EXPORT_REQ guest mes- repeated keystreams. The Dolev-Yao adversary of TAMARIN
sage to FW. Upon receipt of the message via the FwExportGvm has most of these functionalities built in by default; we only
rule, the FW verifies whether the MA thread is assigned had to manually enable the last two behaviors.
to the specific guest by comparing reportId in the context We introduce various Extract (such as the one in Figure 8)
maReportId and Reveal rules that disclose, respectively, secrets generated
| of the | MA thread | with |     |     | in the | context | of GVM. |     |     |     |     |     |     |     |     |
| ------ | --------- | ---- | --- | --- | ------ | ------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
Assuming they are equal, FW responds by transmitting the during launch and long-term keys to the adversary. For in-
guestcontextviaMSG_EXPORT_RSPguestmessage.Theexported stance,theadversarymayusetheExtractVmpckruletocorrupt
context encompasses all GVM data except for maReportId, either GVM or MA and extract vmpck.
which will be replaced with the reportId value of MA on the We allow the compromise of any key generated during
destination machine. launch, excluding a specific finite set of keys bound to a
The specification indicates that the context is sent to a particular thread identifier (e.g., gvmId). Due to the swapping
migration agent on the destination machine through a secure and migration mechanisms, the confidentiality of a given key
channel. However, unlike in previous SEV instances, the may rely on that of other keys, thereby introducing additional
specific mechanism by which this transmission is achieved potential attack vectors.
is outside the scope of the specification. Similarly, the adversary may employ the ExtractCek rule to
Inourmodel,weassumethateach MA threadonthesource extract cek from FW, which it might be able to do in practice
machine(sourceMA),denotedbymaId,iscapableofestablish- byusingside-channelattacks.ItcanalsorevealtheAMDRoot
ingasecurecommunicationchannelwithanMAthreadonthe Key privArk and the AMD Signing Key privAsk.
destinationmachine(target MA).Thiscommunicationchannel The Galois/Counter mode stream cipher is extensively uti-
is modeled via two rules, ComChannelOut and ComChannelIn. lized within AMD SEV-SNP. However, considering the fact
Upon receiving the GVM context from the source MA, that the guest state may be swapped and the guest context
the target initiates the import procedure by sending the migrated,itisnotclearwhetherthiscipherisalwayscorrectly
MA
MSG_IMPORT_REQ guest message to the target FW. Subsequently, employed. We allow the adversary to recover the plaintext
9

FW
IDLE
|     | ReadMSG_KEY_REQ |     |     |     |     |     |     |     |     |     |     |     | ReadMSG_REPORT_REQ |     |     |
| --- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |
ReadSNP_PAGE_SWAP_OUT
|     |     |     | RUNNING |     |     |     |     | RUNNING |     |     |     |     |     |     |     |
| --- | --- | --- | ------- | --- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
RUNNING
|     | root=vmrk |     |     |     |     | encSnapshot=senc(state,oek) |     |     |     |     |     |     |     |     |     |
| --- | --------- | --- | --- | --- | --- | --------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
CreatereportfromgvmCtx
|     | params=<vmpl,hostData,pubIdk> |     |     |     |     | authTag=mac(encSnapshot,oek) |     |     |     |     |     |     |     |     |     |
| --- | ----------------------------- | --- | --- | --- | --- | ---------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ke y = k d f ( ’ V M R K ’ ,root,params) a u t h t a g g v m C t x sig=sign(report,VCEK)
|                 |                |             |               |                             |                      | Sa             | v e            | t o                              |                    |                   | WriteMSG_REPORT_RSP |          |     |     |     |
| --------------- | -------------- | ----------- | ------------- | --------------------------- | -------------------- | -------------- | -------------- | -------------------------------- | ------------------ | ----------------- | ------------------- | -------- | --- | --- | --- |
|                 | W r            | ite M S G _ | K E Y _ R S P |                             |                      | W              | r i te e n c S | n a p s h o t a n d authTag      |                    |                   |                     |          |     |     |     |
|                 |                |             |               |                             | ReadSNP_PAGE_SWAP_IN |                |                |                                  | ReadMSG_EXPORT_REQ |                   |                     |          |     |     |     |
|                 |                |             |               |                             |                      | SWAP_OUT       |                |                                  |                    | SWAP_OUT          |                     |          |     |     |     |
|                 |                |             |               | Verifyauthtag=?             |                      | gvmCtx.authTag |                | VerifyreportId=?                 |                    | gvmCtx.maReportId |                     |          |     |     |     |
|                 |                |             |               | state=sdec(encSnapshot,oek) |                      |                |                | encGctx=wrap(gvmCtx,nonce,vmpck) |                    |                   |                     |          |     |     |     |
|                 |                |             |               | ExecuteGVM                  |                      |                |                | WriteMSG_EXPORT_RSP              |                    |                   |                     |          |     |     |     |
|                 |                |             | Fig.          | 6: AMD                      | Security             | Processor      |                | Firmware State                   | Machine            | (guest            | management)         |          |     |     |     |
| 1 rule          | ComChannelOut: |             |               | 1                           | rule ComChannelIn:   |                |                |                                  |                    |                   | IV.                 | ANALYSIS |     |     |     |
| 2 [ OutChan(~A, |                | ~B,         | m) ]          | 2                           | [ ComChan__(~A,      |                | ~B, m)         | ]                                |                    |                   |                     |          |     |     |     |
ComChanOut(~A, ~B, m) ComChanIn(~A, ~B, m) We classify the specified properties, also referred to as
| 3 −[ |     |     |     | ]→ 3 | −[  |     |     | ]→  |     |     |     |     |     |     |     |
| ---- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
[ ComChan__(~A, ~B, m) ] [ InChan(~A, ~B, m) ] lemmas, into seven distinct groups: source, executability,
| 4   |     |     |     | 4   |     |     |     |             |     |          |                 |     |              |     |               |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | -------- | --------------- | --- | ------------ | --- | ------------- |
|     |     |     |     |     |     |     |     | supporting, |     | secrecy, | authentication, |     | attestation, |     | and freshness |
lemmas.Thecorrespondingcountsoflemmasandtheanalysis
Fig. 7: Secure communication channel establishment time for each group are outlined in Table I.
| 1 rule      | ExtractVmpck: |        |      |     |     |     |     | A. Executability |     | Lemmas |     |     |     |     |     |
| ----------- | ------------- | ------ | ---- | --- | --- | --- | --- | ---------------- | --- | ------ | --- | --- | --- | --- | --- |
| VMPCK(fwId, |               | gvmId, | key) |     |     |     |     |                  |     |        |     |     |     |     |     |
| 2 [         |               |        |      | ]   |     |     |     |                  |     |        |     |     |     |     |     |
−[ CorruptVmpck(key),... ]→ Executability lemmas claim the existence of traces of a
3
| 4 [ Out(key) |     | ]   |     |     |     |     |     |                                      |     |          |     |            |      |                    |        |
| ------------ | --- | --- | --- | --- | --- | --- | --- | ------------------------------------ | --- | -------- | --- | ---------- | ---- | ------------------ | ------ |
|              |     |     |     |     |     |     |     | particular                           |     | form and | are | identified | with | the “exists−trace” |        |
|              |     |     |     |     |     |     |     | keyword.Theuseofthiskeywordinstructs |     |          |     |            |      | TAMARIN            | tomark |
Fig. 8: VM Platform Communication Key compromise the lemma as true if there exists at least one execution which
|     |     |     |     |     |     |     |     | satisfies | the | underlying | formula. |     | This search | for | a particular |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | ---------- | -------- | --- | ----------- | --- | ------------ |
traceisincontrastwiththesecuritylemmas,whichmusthold
from one of two distinct guest messages, both encrypted with for all possible executions.
the same key and the same nonce. We use executability lemmas in two ways. First, we estab-
lishthefunctionalcorrectnessofourmodelbyverifyingthem.
| Note | that | we exclusively |     | model | the stream | cipher | for | guest |     |     |     |     |     |     |     |
| ---- | ---- | -------------- | --- | ----- | ---------- | ------ | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
messages, even though it is also employed for guest pages in Namely, these lemmas encompass a range of potential behav-
SEV-SNP. iors within the model. For instance, the ExeMAManagesTwoGVMs
lemmaspecifiesthatasingleMAthreadcanbeassociatedwith
|     |     |     |     |     |     |     |     | two       | GVMs. | In each  | such | lemma          | we limit | the number | of rule       |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ----- | -------- | ---- | -------------- | -------- | ---------- | ------------- |
|     |     |     |     |     |     |     |     | instances | and   | prohibit | any  | key compromise |          | to         | reduce search |
O. Summary
time.
Ourmodelincludes43rewriterulesspecifiedin1,100lines Second, we prove the existence of attacks in seem-
| of code | (LoC) | without | comments. |     | It has | several | constraints: |       |            |       |           |     |     |           |             |
| ------- | ----- | ------- | --------- | --- | ------ | ------- | ------------ | ----- | ---------- | ----- | --------- | --- | --- | --------- | ----------- |
|         |       |         |           |     |        |         |              | ingly | vulnerable | model | variants. |     | The | rationale | behind this |
two migrations per guest are permitted; is to verify whether certain checks and assumptions are
•
|     |            |        |         |          |     |           |           | indeed | necessary |     | for security |     | properties | to hold. | Specifi- |
| --- | ---------- | ------ | ------- | -------- | --- | --------- | --------- | ------ | --------- | --- | ------------ | --- | ---------- | -------- | -------- |
| •   | the stream | cipher | is only | employed |     | for guest | messages; |        |           |     |              |     |            |          |          |
• firmware updates are not supported; cally, we employ the m4 flags −DIGNORE_ROOT_MD_ENTRY and
key derivation always utilizes the VM Root Key; −DENABLE_REPLAY_OVER_COMM, respectively, to disable authTag
•
• wedonotmodeltheVersionedLoadedEndorsementKey. verification during swap in and to enable replaying messages
|     |     |     |     |     |     |     |     | via | CommChan | rules. | Both lead | to  | attacks | where | the same key |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------ | --------- | --- | ------- | ----- | ------------ |
andnoncegetreused.WeleveragetheExeAttacklemmas,such
1 rule MsgRevealFromKeyNonceReuse: as ExeAttackSwapInRollbackBM, to demonstrate this.
| 2 let | encM1 | = wrap(m1, | nonce, | key) |     |     |     |               |     |        |     |     |     |     |     |
| ----- | ----- | ---------- | ------ | ---- | --- | --- | --- | ------------- | --- | ------ | --- | --- | --- | --- | --- |
| 3     | encM2 | = wrap(m2, | nonce, | key) |     |     |     |               |     |        |     |     |     |     |     |
| in    |       |            |        |      |     |     |     | B. Supporting |     | Lemmas |     |     |     |     |     |
4
| In(< | encM1, | encM2 | >)  |     |     |     |     |     |     |     |     |     |     |     |     |
| ---- | ------ | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 5 [  |        |       | ]   |     |     |     |     |     |     |     |     |     |     |     |     |
6 −[ Neq(m1, m2) Due to the complexity of our model, including loops which
7 , ReuseNonceKey(nonce, key) ]→ tend to make unbounded verification challenging, proving
8 [ Out(m1) ] most of the security properties directly is not feasible. There-
|      |               |     |          |      |          |            |     | fore, | we employ | supporting |        | lemmas     | to  |     |     |
| ---- | ------------- | --- | -------- | ---- | -------- | ---------- | --- | ----- | --------- | ---------- | ------ | ---------- | --- | --- | --- |
| Fig. | 9: Extracting |     | messages | from | repeated | keystreams |     |       |           |            |        |            |     |     |     |
|      |               |     |          |      |          |            |     | •     | provide   | entry      | points | for loops; |     |     |     |
10

enforce termination; SecGvmOekIsSecretMd. These lemmas correspond to scenarios
•
• improve verification time. where the association with an MA is allowed or disallowed,
We allow loops to execute an unbounded number of times; respectively. The lemmas differ in that the latter allows the
|             |              |     |     |           |           |     |        | adversary | to corrupt | even | the | thread | that | is running | in the |
| ----------- | ------------ | --- | --- | --------- | --------- | --- | ------ | --------- | ---------- | ---- | --- | ------ | ---- | ---------- | ------ |
| this models | for instance |     | the | perpetual | servicing |     | of GVM |           |            |      |     | MA     |      |            |        |
requests by and may lead to non-termination of backward background on the same platform.
FW
|            |            |      |     |         |           |      |        | Key secrecy |     | is usually | contingent |     | on several | other | keys. |
| ---------- | ---------- | ---- | --- | ------- | --------- | ---- | ------ | ----------- | --- | ---------- | ---------- | --- | ---------- | ----- | ----- |
| search. We | can remedy | this | by  | proving | that each | loop | has an |             |     |            |            |     |            |       |       |
entrypoint,andwedosothroughinductivereasoning.Seefor Take, for instance, Lemma 1a, which specifies the secrecy of
instance the supporting lemma SupFWInitializesGCTX. the key derived from the vmrk of MA. With this property, we
|     |     |     |     |     |     |     |     | consider | the scenario |     | wherein | the | vmrk is | generated | by the |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------------ | --- | ------- | --- | ------- | --------- | ------ |
Wealsoleveragesupportinglemmastoproveothersupport-
ing and secrecy lemmas. For instance, we may prove certain MA thread maId and installed by the FW thread fwId within
|     |     |     |     |     |     |     |     | the context | of the |     | thread | gvmId. | If we | assume | that the |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ------ | --- | ------ | ------ | ----- | ------ | -------- |
invariants that hold for every loop iteration. An example of GVM
suchapropertyisthelemmaSupGVMMessageCounterMustBeEven, adversary knows the key derived from vmrk, then it must
whichstatesthattheGVMmessagecounteralwayshasaneven necessarily be the case that either vmrk, vmpck or oek of gvmId
|           |               |         |     |          |             |     |       | is corrupted, | or  | one of | the associated |     | MAs | is corrupted. |     |
| --------- | ------------- | ------- | --- | -------- | ----------- | --- | ----- | ------------- | --- | ------ | -------------- | --- | --- | ------------- | --- |
| value. We | utilize proof | oracles |     | to guide | the TAMARIN |     | proof |               |     |        |                |     |     |               |     |
procedure in a direction where such lemmas can be applied. Here,thesecrecyofthekeycriticallydependsonthesecrecy
Most of the supporting lemmas are used to prove the of five other keys. To illustrate this, consider the various way
following lemma which states that honest agents will never in which key could potentially be compromised.
reuse a nonce with the same encryption key (i.e., vmpck): First, if the adversary corrupts the vmrk, it can clearly
|     |     |     |     |     |     |     |     | construct | the key | independently. |     | Second, |     | the adversary | can |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ------- | -------------- | --- | ------- | --- | ------------- | --- |
1 lemma SupNoMsgRevealFromNonceReuse:
|           |                              |       |           |     |           |     |     | obtain the     | key by | compromising |         | the   | vmpck        | of the GVM | thread     |
| --------- | ---------------------------- | ----- | --------- | --- | --------- | --- | --- | -------------- | ------ | ------------ | ------- | ----- | ------------ | ---------- | ---------- |
| 2 ∀ nonce | key #i. ReuseNonceKey(nonce, |       |           |     | key)@i    |     |     |                |        |              |         |       |              |            |            |
|           |                              |       |           |     |           |     |     | and decrypting |        | the guest    | message |       | MSG_KEY_RSP. |            | Third, the |
| 3         | ⇒                            | ∃ #j. | KU(key)@j |     | ∧ #j < #i |     |     |                |        |              |         |       |              |            |            |
|           |                              |       |           |     |           |     |     | adversary      | can    | swap out     | the     | state | of the       | GVM thread | and        |
The difficulty arises as TAMARIN considers all possible acquire either the vmpck or key directly (assuming it has not
traces wherein two distinct messages are encrypted with the been deleted and the adversary possesses the corresponding
samevmpckusingthesamenonce.Moreover,theabilitytoswap
|     |     |     |     |     |     |     |     | oek). Finally, | corrupting |     | the | MA thread | on  | either | the source |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | ---------- | --- | --- | --------- | --- | ------ | ---------- |
a guest any number of times further complicates matters. The or destination platform lets the adversary obtain vmrk from a
SupNoMsgRevealFromNonceReuse lemma is necessary to prove MSG_EXPORT_REQorMSG_IMPORT_REQguestmessage,respectively.
most of the secrecy lemmas. Note that most of the specified secrecy properties can
The fresh variable pid is used to enforce that StateFwGvm additionally be viewed as supporting lemmas, because they
| and StateGvm | fact | symbols | have | injective | instances | [18]. | Our |            |                  |     |                    |     |     |              |        |
| ------------ | ---- | ------- | ---- | --------- | --------- | ----- | --- | ---------- | ---------------- | --- | ------------------ | --- | --- | ------------ | ------ |
|              |      |         |      |           |           |       |     | facilitate | the verification |     | of authentication, |     |     | attestation, | fresh- |
analysisreliesonthecapabilityof TAMARIN tounderapproxi- ness, and other secrecy lemmas.
mateasetofsuchsymbols.Forinstance,thefollowinglemma
StateGvm
can be proven only if we assume that has injective D. Authentication Lemmas
instances:
|     |     |     |     |     |     |     |     | We utilize | authentication |     |     | lemmas | to verify | whether | the |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | -------------- | --- | --- | ------ | --------- | ------- | --- |
1 lemma SupNoGVMHandlingAfterSwapOut: firmware and guest, with the latter being either GVM or
| (∃ pid | #i #j #k | .... |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ------ | -------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
2 ¬ MA, agree on the messages exchanged. Specifically, we are
FwActivateGuest(pid)@i
3
4 ∧ FwDeactivateGuest(pid, ...)@j interested in whether the messages received by the guest,
5 ∧ FwHandleGvm(..., pid, ...)@k running on a particular platform, indeed originate from the
6 ∧ (#i < #j) firmware on that same platform, and vice-versa.
| (#j | < #k)) |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
7 ∧
|            |        |      |     |               |          |     |         | Take for  | instance | Lemma           | 1b, | which | specifies | agreement | on     |
| ---------- | ------ | ---- | --- | ------------- | -------- | --- | ------- | --------- | -------- | --------------- | --- | ----- | --------- | --------- | ------ |
|            |        |      |     |               |          |     |         | the guest | message  | MSG_REPORT_REQ. |     | It    | considers | a GVM     | thread |
| This lemma | states | that | no  | guest request | handling |     | is pos- |           |          |                 |     |       |           |           |        |
gvmId,
sible after the guest has been swapped out, prior to export. with the thread identifier launched by a FW thread
Most of the subsequent supporting lemmas, including the with the thread identifier fwId, under a policy that permits
|            |           |                               |     |     |     |     |        | migration. | Here, | we want | to  | verify | whether | the | attestation |
| ---------- | --------- | ----------------------------- | --- | --- | --- | --- | ------ | ---------- | ----- | ------- | --- | ------ | ------- | --- | ----------- |
| previously | mentioned | SupNoMsgRevealFromNonceReuse, |     |     |     |     | depend |            |       |         |     |        |         |     |             |
on this property. report request the FW obtains was indeed issued by the GVM
afterlaunch,assumingbothhonestagentsareuncompromised.
| C. Secrecy | Lemmas     |         |           |         |            |            |          |                 |        |        |         |              |     |     |           |
| ---------- | ---------- | ------- | --------- | ------- | ---------- | ---------- | -------- | --------------- | ------ | ------ | ------- | ------------ | --- | --- | --------- |
|            |            |         |           |         |            |            |          | E. Attestation  | Lemmas |        |         |              |     |     |           |
| We verify  | the        | perfect | forward   | secrecy | of         | each       | key that |                 |        |        |         |              |     |     |           |
|            |            |         |           |         |            |            |          | The attestation |        | lemmas | specify | authenticity |     | and | integrity |
| we use     | within the | model,  | including |         | long-term, | generated, |          |                 |        |        |         |              |     |     |           |
and derived keys. In all security properties, we permit the properties of attestation reports. Both kinds of property con-
adversary to corrupt dishonest agents, and to take advantage siderthescenariowhereGOverifiesanattestationreportusing
of key and nonce reuse of any agent. the signing key that apparently belongs to a particular chip.
We specify security properties depending on whether the The authenticity properties then affirm the existence of a FW
guest is migratable or not. This approach enables us to thread, operating on that chip, which previously generated the
verify certain properties in the presence of a more pow- samereport.Incontrast,theintegritypropertiesareconcerned
erful adversary. For instance, when analyzing the secrecy with the state of the guest bound to that report, asserting that
of oek, we consider two lemmas: SecGvmOekIsSecret and the guest is indeed running on the designated platform, with
11

1 lemma SecKeyDerFromMaVmrkIsSecret: 1 lemma AuthFwGvmAgreeMsgAttest:
2 ∀ gctxAddr gvmId key info params fwId vmpck maId1 maId2 2 ∀ isMig gvmId chipId fwId vmpck
3 vmrk #i #j #k #l. 3 maId1 maId2 msg #i #j #k.
4 AssociateMigrationAgent(maId1, maId2)@i 4 AssociateMigrationAgent(maId1, maId2)@i
5 ∧ MaGenerateVmrk(maId, fwId, gctxAddr, vmrk)@j 5 ∧ FwAssociateMaGvm(gvmId, maId1)@j
6 ∧ FwInstallMaVmrk(fwId, vmpck, maId, gvmId, 6 ∧ FwReceiveGvmRequest(’FW_GENERATE_REPORT’, isMig,
7 gctxAddr, vmrk)@k 7 chipId, fwId, gvmId, vmpck, msg)@k
8 ∧ key = kdf(info, vmrk, params) 8 ⇒ ( ∃ #l. (#l < #k)
9 ∧ KU(key)@l 9 ∧ GvmIssueRequest(’GVM_REPORT_REQUEST’, isMig,
10 ⇒ (∃ #m. (#m < #l) ∧ CorruptVmrk(vmrk)@m) 10 gvmId, chipId, fwId, msg)@l )
11 ∨ (∃ #m. (#m < #l) ∧ CorruptVmpck(vmpck)@m) 11 ∨ (∃ #l. (#l < #k) ∧ CorruptVmpck(vmpck)@l)
12 ∨ (∃ #m. (#m < #l) ∧ CorruptImageOek(gvmId)@m) 12 ∨ (∃ #l. (#l < #k) ∧ CorruptImageOek(gvmId)@l)
13 ∨ (∃ #m. (#m < #l) ∧ CorruptImageVmpck(maId1)@m) 13 ∨ (∃ #l. (#l < #k) ∧ CorruptImageVmpck(maId1)@l)
14 ∨ (∃ #m. (#m < #l) ∧ CorruptImageVmpck(maId2)@m) 14 ∨ (∃ #l. (#l < #k) ∧ CorruptImageVmpck(maId2)@l)
Lemma 1a: Secrecy of a vmrk-derived key Lemma 1b: Agreement on the MSG_REPORT_REQ guest message
the correct policy and configuration. Lemma 2 is an example 1 lemma AttReportStrongIntegrity:
2 ∀ goId goImage privIdk report image policy reportData
of such a property.
3 ld digestIdk pubIdk reportId maReportId chipId fwId
4 hostData maId1 maId2 vmpck imageId gvmId fwLaunchTcb
5 fwCurrentTcb askId arkId ltks #i #j #k #l.
F. Freshness Lemmas 6 AssociateMigrationAgent(maId1, maId2)@i
7 ∧ FwInstallVmpck(’GVM’, fwId, gvmId, vmpck)@j
Thefreshnesslemmascompriseseveraluniquenesslemmas. 8 ∧ FwAssociateMaGvm(gvmId, maId1)@o
In particular, these lemmas allow us to determine whether 9 ∧ FwBindReportIdGvm(vmpck, reportId)@p
two separate GVM threads can acquire the same derived key,10 ∧ GoReportVerify(goImage, goId, privIdk, report, ltks)@l
11 ∧ ltks = <arkId, askId>
whether GO can verify outdated certificates, or if the launch 12 ∧ report = <imageId, policy, reportData, ld,
procedure can be manipulated to enforce FW to install the13 digestIdk, reportId, maReportId, chipId, gvmId,
same vmrk into multiple guests. 14 fwLaunchTcb, fwCurrentTcb>
15 ∧ policy = ’ENABLE_MIGRATION’
As we mentioned previously, guests have the option, 16 ∧ digestIdk = h(<’ID_KEY’, pubIDK>)
via MSG_KEY_REQ, to choose the root key—VCEK, VLEK, or17 ∧ pubIdk = pk(privIdk)
vmrk—and provide additional data for key derivation. Opt-18 ∧ ld = h(< ’VM_IMAGE’, image >)
19 ⇒ ∃ isMig vmpck gvmId gvmMsgCount #m.
ing for vmrk as the root key should ensure that each 20 (#m < #l)
guest instance will obtain a different key; see lemma21 ∧ GvmReportReq(chipId, fwId, imageId, gvmId, vmpck,
FreshKeyDerFromFwVmrkIsGvmUnique. We note here, however,22 gvmMsgCount:nat, reportData, isMig)@m
23 ∨ (∃ #m. (#m < #l) ∧ CorruptLtk(’ARK’, arkId)@m )
thatitisnotpossibleforaguesttoprovidearandomsequence 24 ∨ (∃ #m. (#m < #l) ∧ CorruptLtk(’ASK’, askId)@m )
of bytes to be included in SEV-SNP key derivation (so unlike25 ∨ (∃ #m. (#m < #l) ∧ CorruptLtk(’CEK’, chipId)@m )
in Intel SGX, key wear-out protection is not supported). 26 ∨ (∃ #m. (#m < #l) ∧ CorruptVmpck(vmpck)@m)
27 ∨ (∃ #m. (#m < #l) ∧ CorruptImageOek(gvmId)@m)
28 ∨ (∃ #m. (#m < #l) ∧ CorruptImageVmpck(maId1)@m)
29 ∨ (∃ #m. (#m < #l) ∧ CorruptImageVmpck(maId2)@m)
V. RESULTSANDDISCUSSION
Weanalyzedatotalof114properties,including42security
Lemma 2: Strong integrity of attestation report
properties.ThesummaryoftheresultsispresentedinTableI.
All but five security properties were successfully verified.
The analysis of the model was conducted using Debian
desired security guarantees, while exhibiting some potential
11, running on an AMD EPYC 7713 processor, equipped
drawbacks discussed in the following subsections.
with 16 cores and 32 threads, with a 2.0GHz base clock.
The positive results hold in a very general setting with an
The system was complemented by 256GB of memory. We
unbounded number of platforms, VMs, guest owners and ses-
employed TAMARIN version 1.10.0 (commit hash cb62c305)
sions. Furthermore, they hold assuming an adversary capable
for the analysis.
ofcorruptingarbitraryagents.Theverificationwasenabledby
All of the specified properties were automatically analyzed
supporting lemmas and proof strategies based on the oracle
in about 6 hours, as detailed in Table I. In this section, we
rankings. Both of them were utilized to either enforce ter-
discuss the results in detail, including the potential impact
mination or decrease verification time. Finally, many security
andmitigationsforthediscoveredweaknesses.Thesefindings
properties were specified in a very granular manner, which
were disclosed to AMD.
was instrumental in proving other security properties.
A substantial amount of work went into proving the lemma
A. Positive Results
FreshNoMsgRevealFromNonceReuse; the majority of the support-
Onapositivenote,allsecrecyandfreshnessproperties,and ing lemmas were specifically leveraged for this purpose. This
the majority of authentication and attestation properties, have is mainly because we allow the GVM state to be swapped
been successfully proven. Hence, our results show that the out and swapped in an unbounded number of times. Con-
SEV-SNPsoftwareinterfacedoesindeedprovidealmostallthe sequently, we had to prove that the FW and GVM message
12

| PROPERTIES |                                   | DESCRIPTION |     | # LEMMA |     |     | MODEL | STEPS RUNTIME(MIN) |     | #LOC |
| ---------- | --------------------------------- | ----------- | --- | ------- | --- | --- | ----- | ------------------ | --- | ---- |
| Source     | Mitigatingpartialdeconstructions. |             |     | 1 Types |     |     | ✓     | 938                | 1   | 40   |
✓
| Executability | Functionalcorrectness. |     |     | 13 Exe* |     |     |     | -   | 18  | 1360 |
| ------------- | ---------------------- | --- | --- | ------- | --- | --- | --- | --- | --- | ---- |
Supporting Enforceterminationandimproveverificationtime 58 Sup* ✓ - 304 2600
|     |            |                |                   | SecMaVmpckIsSecret    |     |     | ✓   | 6427 |     |     |
| --- | ---------- | -------------- | ----------------- | --------------------- | --- | --- | --- | ---- | --- | --- |
|     |            |                |                   | SecGvmOekIsSecretMd   |     |     | ✓   | 1621 |     |     |
|     |            |                |                   | SecGvmOekIsSecret     |     |     | ✓   | 3455 |     |     |
|     |            |                |                   | SecGvmVmpckIsSecretMd |     |     | ✓   | 1625 |     |     |
|     |            |                |                   | SecGvmVmpckIsSecret   |     |     | ✓   | 4436 |     |     |
|     | Secrecy of | long-term keys | and session keys. |                       |     |     | ✓   |      |     |     |
|     |            |                |                   | SecMaVmrkIsSecret     |     |     |     | 3418 |     |     |
F o r e x a m pl e , t h e s e c re c y of g ue st V M P C K , ✓
Secrecy w it h t h e m i g rat i o n-e n a b le d (M A_ E N =1 ) p o li cy 13 SecFwVmrkIsSecret 1619 107 1000
✓
|     | (SecGvmVmpckIsSecret). |     |     | SecKeyDerFromMaVmrkIsSecret |     |     |     | 12833 |     |     |
| --- | ---------------------- | --- | --- | --------------------------- | --- | --- | --- | ----- | --- | --- |
✓
|     |     |     |     | SecKeyDerFromFwVmrkIsSecret |     |     |     | 691  |     |     |
| --- | --- | --- | --- | --------------------------- | --- | --- | --- | ---- | --- | --- |
|     |     |     |     | SecArkIsSecret              |     |     | ✓   | 2155 |     |     |
|     |     |     |     | SecAskIsSecret              |     |     | ✓   | 2164 |     |     |
|     |     |     |     | SecCekIsSecret              |     |     | ✓   | 2156 |     |     |
|     |     |     |     | SecVcekIsSecret             |     |     | ✓   | 302  |     |     |
|     |     |     |     | AuthFwGvmAgreeMsgAttestMd   |     |     | ✓   | 52   |     |     |
|     |     |     |     | AuthFwGvmWeakAgreeMsgAttest |     |     | ✓   | 282  |     |     |
|     |     |     |     | AuthFwGvmAgreeMsgAttest     |     |     | ✗   | 27   |     |     |
|     |     |     |     | AuthFwGvmAgreeMsgKeyDerMd   |     |     | ✓   | 47   |     |     |
✓
|     |     |     |     | AuthFwGvmWeakAgreeMsgKeyDer |     |     |     | 282 |     |     |
| --- | --- | --- | --- | --------------------------- | --- | --- | --- | --- | --- | --- |
✗
|     |     |     |     | AuthFwGvmAgreeMsgKeyDer |     |     |     | 27  |     |     |
| --- | --- | --- | --- | ----------------------- | --- | --- | --- | --- | --- | --- |
✓
Authenticity of guest messages, exchanged be- AuthGvmFwAgreeMsgAttestMd 129
|     |                     |                 |           | AuthGvmFwWeakAgreeMsgAttest |     |     | ✓   | 56  |     |     |
| --- | ------------------- | --------------- | --------- | --------------------------- | --- | --- | --- | --- | --- | --- |
|     | tween SNP-protected | guest/migration | agent and |                             |     |     |     |     |     |     |
|     |                     |                 |           | AuthGvmFwAgreeMsgAttest     |     |     | ✗   | 29  |     |     |
Authentication SNP firmware. These, for instance, specify that 18 22 1600
a message received by the firmware necessarily AuthGvmFwAgreeMsgKeyDerMd ✓ 129
originatedattheassociatedguestthatwas,atthe AuthGvmFwWeakAgreeMsgKeyDer ✓ 56
time,executingonthesameplatform.
|     |     |     |     | AuthGvmFwAgreeMsgKeyDer |     |     | ✗   | 29  |     |     |
| --- | --- | --- | --- | ----------------------- | --- | --- | --- | --- | --- | --- |
|     |     |     |     | AuthFwMaAgreeMsgVmrk    |     |     | ✓   | 7   |     |     |
|     |     |     |     | AuthFwMaAgreeMsgExport  |     |     | ✓   | 5   |     |     |
|     |     |     |     | AuthFwMaAgreeMsgImport  |     |     | ✓   | 5   |     |     |
✓
|     |     |     |     | AuthMaFwAgreeMsgVmrk |     |     |     | 6   |     |     |
| --- | --- | --- | --- | -------------------- | --- | --- | --- | --- | --- | --- |
✓
|     |     |     |     | AuthMaFwAgreeMsgExport |     |     |     | 10  |     |     |
| --- | --- | --- | --- | ---------------------- | --- | --- | --- | --- | --- | --- |
✓
|     |     |     |     | AuthMaFwAgreeMsgImport |     |     |     | 6   |     |     |
| --- | --- | --- | --- | ---------------------- | --- | --- | --- | --- | --- | --- |
✓
|     |            |                    |               | FreshNoMsgRevealFromNonceReuse |     |     |     | 4096 |     |     |
| --- | ---------- | ------------------ | ------------- | ------------------------------ | --- | --- | --- | ---- | --- | --- |
|     |            |                    |               | FreshMaVmrkInstallIsUnique     |     |     | ✓   | 20   |     |     |
|     | Uniqueness | of certain events. | For instance, |                                |     |     |     |      |     |     |
Freshness whethereachmessageofguestemployesafresh 138 FreshAttReportFreshness ✓ 10 138 480
counter(FreshNoMsgRevealFromNonceReuse). FreshKeyDerFromFwVmrkIsGvmUnique ✓ 177
|     |     |     |     | FreshKeyDerFromMaVmrkIsGvmUnique |     |     | ✓   | 17825 |     |     |
| --- | --- | --- | --- | -------------------------------- | --- | --- | --- | ----- | --- | --- |
|     |     |     |     | AttReportAuthenticityMd          |     |     | ✓   | 505   |     |     |
Attestation report integrity and authenticity. For AttReportAuthenticity ✓ 16
instance,ifanattestationreportstatesthataguest
|     |     |     |     | AttReportIntegrityMd |     |     | ✓   | 554 |     |     |
| --- | --- | --- | --- | -------------------- | --- | --- | --- | --- | --- | --- |
Attestation with a certain configuration is executing on a 6 47 900
particularplatform,thenthismustindeedbetrue AttReportWeakIntegrity ✓ 2028
✗
|     | forthatguest. |     |     | AttReportStrongIntegrity |     |     |     | 52  |     |     |
| --- | ------------- | --- | --- | ------------------------ | --- | --- | --- | --- | --- | --- |
✗
|     |     |     |     | AttBackAndForthMig |     |     |     | 71  |     |      |
| --- | --- | --- | --- | ------------------ | --- | --- | --- | --- | --- | ---- |
|     |     |     | Σ   | 114                |     |     |     |     | 365 | 7845 |
TABLE I: A summary of the analyzed properties. All properties are automatically verified (✓) or falsified (✗). We leverage
oracle rankings based on the "smart" heuristic, with supporting lemmas, for proof guidance. The suffix “Md” indicates
that a guest was launched with the migration-disabled policy. STEPS denote proof length. The final lemma holds if the
| −DENFORCE_MIGRATION_POLICY |     | flag is | enabled. |     |     |     |     |     |     |     |
| -------------------------- | --- | ------- | -------- | --- | --- | --- | --- | --- | --- | --- |
counters are synchronised with each swap operation, i.e., The core issue is that guest messages lack binding to a
they are either equal or the FW counter exceeds the GVM specific platform. We believe that this is a design decision
counter by two. The corresponding lemmas are prefixed with made to facilitate the seamless migration of guests. However,
SupFWAndGVMMsgCountersAreInSync. this allows guest messages sent from one platform to be
|     |     |     |     |     | received | on another, although | it seems | reasonable | to  | expect, |
| --- | --- | --- | --- | --- | -------- | -------------------- | -------- | ---------- | --- | ------- |
e.g.,thatguestsonlyacceptmessagessentbythefirmwareon
| B. Platform | Confusion | Attacks |     |     | the same | platform. |     |     |     |     |
| ----------- | --------- | ------- | --- | --- | -------- | --------- | --- | --- | --- | --- |
We identified five formal attacks in our model; four on We would like to emphasize that we have not attempted
authenticationpropertiesandoneonanattestationproperty.In to launch these attacks in practice—this would require the
alltheattacks,theplatform-agnosticnatureofguestmessages, development of a malicious hypervisor and a kernel driver
along with the migration feature, is used by the adversary to for SNP-protected guests, both of which support AMD SEV-
guide the system into a state we believe is undesirable. SNPlivemigration.Notethatnosuchopen-sourcehypervisor
13

exists (e.g., AMD has still not added support for migration of Inspiteofthat,weshowthatthescopeofdiscoveredformal
SNP-protected guests to their QEMU fork [19]). attacksislimitedbystatingweakversionsofallfailinglemmas
Similarly, we do not claim that these attacks have a direct and proving that they all hold. Using the attestation integrity
impact on security of the deployed systems, but we do property as an example, the weak variant says that if a
GO
discuss potential impact as well as mitigation strategies in the verifiestheattestationreport,thenitmusthavebeenrequested
following section. Finally, we note that we did not anticipate by a running on some platform—the chipId value in
GVM
thesepropertiestofail—theywerediscoveredpurelybyformal the attestation report does not have to match the chipId value
analysis. of the platform where the GVM was running at the time of
In the four authentication attacks, a message originating on the request. Weaker variants of authentication properties are
| one platform | is          | accepted | on    | a different |          | platform.    | The four | similarly | defined. |     |     |     |     |     |     |
| ------------ | ----------- | -------- | ----- | ----------- | -------- | ------------ | -------- | --------- | -------- | --- | --- | --- | --- | --- | --- |
| attacks      | are brought |          | about | by all      | possible | combinations | of       |           |          |     |     |     |     |     |     |
the following two binary parameters: (1) where the message C. Back-and-forth migration attack
| originated  | (at | FW and | is      | intended | for | GVM, at | GVM and    |         |        |            |     |                  |         |             |         |
| ----------- | --- | ------ | ------- | -------- | --- | ------- | ---------- | ------- | ------ | ---------- | --- | ---------------- | ------- | ----------- | ------- |
|             |     |        |         |          |     |         |            | The     | formal | attacks    | in  | the previous     | section | show        | that if |
| is intended | for | FW)    | and (2) | the type | of  | request | being sent |         |        |            |     |                  |         |             |         |
|             |     |        |         |          |     |         |            | a guest | is     | configured | to  | allow migration, |         | the CHIP_ID | and     |
or responded to (attestation report request, key derivation values in the verified attestation report do
COMMITTED_TCB
request).Notethatminorvariantsexist;forexample,the
|     |     |     |     |     |     |     | GVM | not need | to  | correspond | to  | the platform | where | the | guest was |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | ---------- | --- | ------------ | ----- | --- | --------- |
can be migrated before or after the FW processes the request. running,atornearthetimeitrequestedtheattestation.Hence,
| The steps | to  | reproduce | the | attacks | wherein | the | GVM sends |             |     |       |          |             |     |           |         |
| --------- | --- | --------- | --- | ------- | ------- | --- | --------- | ----------- | --- | ----- | -------- | ----------- | --- | --------- | ------- |
|           |     |           |     |         |         |     |           | a malicious |     | cloud | provider | may execute | the | described | attacks |
arequestononeplatform,andtheadversaryforwardsittothe
|     |     |     |     |     |     |     |     | and | successfully | deceive |     | a guest | owner into | believing | that |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------ | ------- | --- | ------- | ---------- | --------- | ---- |
FW on another platform can be seen in Figure 10. The FW on a guest is executing critical functionality on an up-to-date
| thedestinationplatformincorrectlybelievesthatthe |     |     |     |     |     |     | GVM sent |          |       |     |          |            |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | --- | -------- | -------- | ----- | --- | -------- | ---------- | --- | --- | --- |
|                                                  |     |     |     |     |     |     |          | platform | when, | in  | reality, | it is not. |     |     |     |
themessagewhileitwasexecutingonthedestinationplatform, Asamatteroffact,theissuejustdescribedpersistsindepen-
whereas in reality the GVM was executing on the source dently of the discovered platform confusion attacks. Even if
| platform | at the | time. Because |     | the request |     | could have | been for |     |     |     |     |     |     |     |     |
| -------- | ------ | ------------- | --- | ----------- | --- | ---------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
weaddressedtheattackssomehow,amaliciouscloudprovider
either an attestation report or for key derivation, the described might still be able to migrate a guest just before the guest
| attack shows | that    | neither                 | of  | the AuthFwGvmAgreeMsgAttest |     |            | (see      |          |     |             |         |               |           |           |            |
| ------------ | ------- | ----------------------- | --- | --------------------------- | --- | ---------- | --------- | -------- | --- | ----------- | ------- | ------------- | --------- | --------- | ---------- |
|              |         |                         |     |                             |     |            |           | requests | an  | attestation | report. | Specifically, |           | the cloud | provider   |
| Lemma        | 1b) and | AuthFwGvmAgreeMsgKeyDer |     |                             |     | properties | holds. In |          |     |             |         |               |           |           |            |
|              |         |                         |     |                             |     |            |           | could    | run | the guest   | on      | a vulnerable  | platform, |           | but obtain |
both cases there is no agreement on the value of chipId. As a attestation reports by briefly migrating the guest to the secure
| technical | sidenote, | to  | find | an attack | trace | faster | in the final |     |     |     |     |     |     |     |     |
| --------- | --------- | --- | ---- | --------- | ----- | ------ | ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
platform,thuscreatinganillusionthattheguestisconsistently
model, we added several preconditions to the lemmas that managed by secure, up-to-date firmware.
| restricted | the set | of considered |     | traces. |     |     |     |     |     |     |     |     |     |     |     |
| ---------- | ------- | ------------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Thepropertythatisviolated,AttNoBackAndForthAttack,con-
| In the | remaining | two | authentication |     | attacks, | the | sends | a      |             |           |     |          |           |          |          |
| ------ | --------- | --- | -------------- | --- | -------- | --- | ----- | ------ | ----------- | --------- | --- | -------- | --------- | -------- | -------- |
|        |           |     |                |     |          |     | FW    | siders | the guest’s | execution |     | history. | It states | that not | only was |
responsetothecorrespondingguestrequest,andtheadversary theguestexecutingonaplatformwhosefirmwareTCBversion
| migrates | the | to  | another | platform |     | (where | the | is  |     |     |     |     |     |     |     |
| -------- | --- | --- | ------- | -------- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
GVM GVM matchestheversionintheattestationreport,butalsothatevery
resumed and waits for a response) before forwarding the previous guest launch occurred on firmware with the same
| response | to the | GVM. |     |     |     |     |     |     |     |     |     |     |     |     |     |
| -------- | ------ | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
TCBversion.Iftheversionintheattestationreportreflectsan
| However, | if  | the adversary |     | now forwards |     | the response | from |            |          |     |        |               |     |              |      |
| -------- | --- | ------------- | --- | ------------ | --- | ------------ | ---- | ---------- | -------- | --- | ------ | ------------- | --- | ------------ | ---- |
|          |     |               |     |              |     |              |      | up-to-date | firmware |     | image, | a third party | can | be confident | that |
theFWonthesourceplatformtotheGVM,theGVMwillaccept the guest has consistently operated on a secure platform.
| it. The | described | attack | falsifies | the | AuthGvmFwAgreeMsgAttest |     |     |     |     |     |     |     |     |     |     |
| ------- | --------- | ------ | --------- | --- | ----------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
AlthoughinpracticewewanttoensurethattheTCBversion
and AuthGvmFwAgreeMsgKeyDer properties. never "decreases" across platforms, here for simplicity we
Similarly, in the attack on the violated attestation property, focusonmaintainingversionequality.Toautomaticallyfindan
| theattestationreportthatthe |     |     |     | verifieswasnotgeneratedon |     |     |     |     |     |     |     |     |     |     |     |
| --------------------------- | --- | --- | --- | ------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
GO attack trace, we utilize the AttBackAndForthMigAttack lemma.
the platform where the GVM requested the attestation report. The most direct mitigation for the above issues, includ-
| Instead, | it was | generated | on  | the platform |     | that the | GVM was |         |      |           |                |     |           |     |            |
| -------- | ------ | --------- | --- | ------------ | --- | -------- | ------- | ------- | ---- | --------- | -------------- | --- | --------- | --- | ---------- |
|          |        |           |     |              |     |          |         | ing the | just | described | back-and-forth |     | migration |     | attack, is |
migratedto.Asaconsequence,AttReportStrongIntegrity(see
|     |     |     |     |     |     |     |     | to enforce |     | a TCB | policy | and other | migration |     | policies in |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ----- | ------ | --------- | --------- | --- | ----------- |
Lemma2)isfalsifiedbecausechipIdcandifferfromtheantic- the migration agent. In fact, the SEV-SNP firmware ABI
| ipated value. | The         | corresponding   |         | trace        | is shown | in        | Figure 10.   |               |        |                |         |               |            |                 |        |
| ------------- | ----------- | --------------- | ------- | ------------ | -------- | --------- | ------------ | ------------- | ------ | -------------- | ------- | ------------- | ---------- | --------------- | ------ |
|               |             |                 |         |              |          |           |              | specification |        | [12]           | already | states the    | following: |                 |        |
| When          | considering |                 | changes | to the       | model    | that      | would com-   |               |        |                |         |               |            |                 |        |
| pletely       | prevent     | the platform    |         | confusion    |          | attacks,  | we decided   |               |        |                |         |               |            |                 |        |
|               |             |                 |         |              |          |           |              | The           | MA     | is responsible |         | for supplying | the        | VMRK            | during |
| against       | changes     | that appear     |         | to translate | poorly   | to        | practice. In |               |        |                |         |               |            |                 |        |
|               |             |                 |         |              |          |           |              | the           | launch | process        | and     | for enforcing | the        | guest migration |        |
| particular,   | we          | have considered |         | to           | make     | the guest | messages     |               |        |                |         |               |            |                 |        |
policy.
| platform-specific |       | by   | employing | a      | fresh | VMPCK | after every |     |     |     |     |     |     |     |     |
| ----------------- | ----- | ---- | --------- | ------ | ----- | ----- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
| migration.        | While | this | change    | to the | model | would | address the |     |     |     |     |     |     |     |     |
failing formal properties, we are reluctant to recommend it as However, it does not give any details on what the migration
a practical solution. Namely, it would require modifications policy might be. Hence, we propose that the SEV-SNP ABI
to guest pages upon each guest import, and it is unclear to specification be updated to:
us how to do this while preserving seamless migration, i.e., make explicit the migration policy that a compliant mi-
•
without having to reboot the guest. gration agent should enforce, and
14

clarify the (lack of) guarantees on the and of components and proofs would be immensely practical for
| •   |     |     |     |     |     | CHIP_ID |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
COMMITTED_TCB values in the verified attestation report the analysis of similar systems. For example, a significant
when migration is enabled, and their dependence on the effort was dedicated to verifying the freshness of nonces used
migration agent behavior. in the GCM encryption, where the supporting scaffolding
The −DENFORCE_MIGRATION_POLICY flag enables the policy in substantially increased the overall complexity of the analysis.
|            |         |          |          |       |     |        |          | We envision | acompositional |     |     | approach | that wouldallow |     | for |
| ---------- | ------- | -------- | -------- | ----- | --- | ------ | -------- | ----------- | -------------- | --- | --- | -------- | --------------- | --- | --- |
| our model. | It adds | a simple | equality | check | to  | ensure | that the |             |                |     |     |          |                 |     |     |
firmware images on both the source and destination platforms the analysis of a simplified version of the system, once nonce
|                  |          |          |     |         |         |     |     | reuse has        | been ruled | out       | definitively. | However,    |                   | no automated |      |
| ---------------- | -------- | -------- | --- | ------- | ------- | --- | --- | ---------------- | ---------- | --------- | ------------- | ----------- | ----------------- | ------------ | ---- |
| have the         | same TCB | version. |     |         |         |     |     |                  |            |           |               |             |                   |              |      |
|                  |          |          |     |         |         |     |     | tools support    | such       | reasoning | at            | the moment. |                   |              |      |
|                  |          |          |     |         |         |     |     | The novel        | software   | interface |               | differs     | too significantly |              | from |
| D. Post-analysis |          | insights | and | lessons | learned |     |     |                  |            |           |               |             |                   |              |      |
|                  |          |          |     |         |         |     |     | that of SEV(-ES) | to         | allow     | the results   | to          | be carried        | over.        |      |
After performing the formal analysis of SEV-SNP, several As a matter of fact, even reusing parts of our model to an-
key insights and lessons emerged. alyze thesecurity of theSEV and SEV-ESsoftware interfaces
There is a tension between the simplicity of the SEV-SNP is currently infeasible. Namely, the described differences are
software interface and the security guarantees of the system simply too vast; for instance, the guest messages, which our
whose critical components are underspecified. analysis focuses on, were not supported previously. However,
WhileSEV-SNPisricherintermsofbothusabilityfeatures we believe our model is sufficiently modular to be easily
and security guarantees compared to both previous iterations, extended and used to analyze additional aspects of the SEV-
namely SEV and SEV-ES, the software interface of SEV- SNP software interface, even within an unbounded number of
SNP is also more streamlined in some ways. First, the launch migrations. In addition, it provides a useful reference for any
procedurehasbeensimplifiedasthelaunchprotocolnolonger similar modelling and verification endeavor, especially when
exists.Second,themechanismofmigrationisnowoutsourced the tools used are based on multiset rewriting.
| to a new | entity | called | a Migration |     | Agent. | Third, | the key |     |     |     |     |     |     |     |     |
| -------- | ------ | ------ | ----------- | --- | ------ | ------ | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
schedule is simpler as there are fewer key dependencies. This VI. RELATEDWORK
includes the absence of, for instance, the TIK, TEK, KEK and There is a vast body of literature devoted to AMD SEV
KIK.Fourth,noDiffie-Hellmanoperations,whicharetypically technologies in general. However, to the best of our knowl-
costly to analyze, are necessitated by the specification. edge, there has been no attempt to build a symbolic model
Although these changes make the SEV-SNP software inter- and analyze the security of SEV-SNP, including its ABI.
facemoreamenabletoformalanalysis,theaddedsimplicityis Apart from a work by Antonino et al. [20] that considers
adouble-edgedsword.Criticalpartsofthesystem,suchasthe SGX and pre-SNP SEV software interface to propose and
migration agent, are left underspecified. Our formal analysis formally verify a flexible remote attestation protocol, most
demonstrates that it is essential to have clear guidelines for of the existing SEV research focuses on its implementation.
building migration agents, or a canonical migration agent Although we find this line of work orthogonal to ours, here
implementation, so as to ensure critical security properties. we provide a short overview.
| There is | a tension | between |     | the ability | to easily | and | trans- |         |        |     |        |         |        |      |     |
| -------- | --------- | ------- | --- | ----------- | --------- | --- | ------ | ------- | ------ | --- | ------ | ------- | ------ | ---- | --- |
|          |           |         |     |             |           |     |        | Attacks | on SEV | and | SEV-ES | exploit | nested | page | ta- |
parently migrate an SNP-secured guest and the guarantees bles [21], external interfaces such as virtio devices [22]
provided by the attestation protocol to the relying party. and direct memory access [23], the signature verification
Many cloud workloads of today support live migration mechanism of the AMD-SP OS [24], the lack of memory
without perceived downtime. However, our formal analysis integrityprotection[25],TLBentries[26],blockpermutation-
shows that the platform-identifying data in the attestation agnostic measurements of VM images [27], guest address
reports cannot be trusted in all circumstances. It would be space identifiers [28], and the power-reporting interface [29].
beneficial if the attestation report included information about AttacksonSEV-SNP target#VC [30]and 0x80 interrupts[31],
a set of platforms on which the guest can execute. The guest usesoftware-basedfaultinjection[32,33],andrelyoncipher-
owner does not need to know or care on which particular textsidechannels[34,35].Additionally,Googlehasidentified
machineitiscurrentlyexecutingbecausethatinformationmay multiple issues with SEV-SNP firmware [36].
become deprecated soon (e.g., as soon as the guest migrates Other works have used formal techniques to reason about
to another machine). theinterfacesofothertrustedexecutionenvironments[37,38,
Multiset rewriting and the Tamarin prover are suitable 39,40,41,42]andtrustedplatformmodules(TPMs)[43,44].
tools for the formal analysis of complex software interfaces; Notably, attestation schemes have received a lot of atten-
however, SEV-SNP pushes the boundaries of these methods, tion: Intel SGX Data Center Attestation Primitives [45] and
requiring further theoretical advancements to enable routine, TDX [46], ARM CCA [47], and ECC-based [48] and TPM
full-fidelity analysis of such systems. 2.0-based[48]DirectAnonymousAttestationschemes.More-
While the final, fully automated formal analysis can be over, CCA is the first VM-based TEE architecture with a
conducted in eight hours, this is the result of much effort to formally verified firmware [4].
| specify the | supporting  |     | lemmas,   | implement | the         | proof-guiding |           |     |     |      |            |     |     |     |     |
| ----------- | ----------- | --- | --------- | --------- | ----------- | ------------- | --------- | --- | --- | ---- | ---------- | --- | --- | --- | --- |
|             |             |     |           |           |             |               |           |     |     | VII. | CONCLUSION |     |     |     |     |
| oracles and | to simplify |     | the model | without   | sacrificing |               | fidelity. |     |     |      |            |     |     |     |     |
We believe theoretical advancements in the area of automated In this paper, we developed the first, comprehensive formal
verification that would enable the compositionality and reuse model of the AMD SEV-SNP software interface. The model
15

Fig. 10: An attack trace for the AttReportStrongIntegrity lemma: deploys a guest image and policy; launches
1 GO 2 FW
guest on the first platform; 3 GVM requests an attestation report (MSG_REPORT_REQ); 4 FW immediately swaps out the guest;
5 exports the guest context; 6 transfers the guest context to MA’ on the second platform; 7 FW’ imports the guest
| FW  |     | MA  |     |     |
| --- | --- | --- | --- | --- |
context; 8 FW’ swaps in the guest; 9 FW’ receives MSG_REPORT_REQ from 3 , and issues an attestation report; 10 GO verifies
the report. In the end, GO erroneously believes that the attestation report was requested on the second platform (where it was
| generated), whereas | the GVM | was executing | on the first platform | at that time. |
| ------------------- | ------- | ------------- | --------------------- | ------------- |
16

| covers the | guest | launch, | remote | attestation, | key derivation, |     |     |     |     |     |     |     |     |     |
| ---------- | ----- | ------- | ------ | ------------ | --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Verification:25thInternationalConference,CAV2013,SaintPetersburg,
|     |     |     |     |     |     |     | Russia,July13-19,2013.Proceedings25. |     |     |     | Springer,2013,pp.696–701. |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------ | --- | --- | --- | ------------------------- | --- | --- | --- |
pageswapandlivemigrationfeatures.Weproducedautomated
|               |     |              |             |           |               |          | [14] J. Mitchell,  | A.       | Scedrov,       | N. Durgin,  | and P. | Lincoln,  | “Undecidability |     |
| ------------- | --- | ------------ | ----------- | --------- | ------------- | -------- | ------------------ | -------- | -------------- | ----------- | ------ | --------- | --------------- | --- |
| formal proofs |     | for the most | important   | secrecy,  | authenticity, |          |                    |          |                |             |        |           |                 |     |
|               |     |              |             |           |               |          | of bounded         | security | protocols,”    | in Workshop |        | on formal | methods         | and |
| attestation,  | and | freshness    | properties, | including | the           | proof of |                    |          |                |             |        |           |                 |     |
|               |     |              |             |           |               |          | securityprotocols. |          | Citeseer,1999. |             |        |           |                 |     |
correct stream cipher usage. [15] AMDESE,“sev-guest,”https://github.com/AMDESE/linux,2024.
|     |     |     |     |     |     |     | [16] P. Paradzik, | A.  | Derek, | and M. Horvat, | “Formal | Security | Analysis | of  |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ------ | -------------- | ------- | -------- | -------- | --- |
Whilemostoftherelevantsecuritypropertiesweresuccess-
theAMDSEV-SNPSoftwareInterface,”https://gitlab.com/sev-snp-abi
fully verified, our analysis shows that several authentication -security-analysis/sev-snp-abi-security-analysis,2024.
and attestation properties do not hold when SNP-protected [17] C. Cremers, B. Kiesl, and N. Medinger, “A formal analysis of
guests are launched with migration-enabled policies due to a IEEE 802.11’s WPA2: Countering the kracks caused by cracking the
counters,”in29thUSENIXSecuritySymposium(USENIXSecurity20).
lackofplatformbindinginguestmessages.Despiteidentifying USENIXAssociation,Aug.2020,pp.1–17.
these formal attacks, there is still work to be done; most [18] Tamarin-Prover Manual, https://tamarin-prover.com/manual/develop/te
x/tamarin-manual.pdf,TheTamarinTeam.
| pressing | are a | practical confirmation |     | of the attacks, |     | establish- |            |        |        |                                        |     |     |     |     |
| -------- | ----- | ---------------------- | --- | --------------- | --- | ---------- | ---------- | ------ | ------ | -------------------------------------- | --- | --- | --- | --- |
|          |       |                        |     |                 |     |            | [19] “QEMU | source | code,” | https://github.com/qemu/qemu/blob/7e3b |     |     |     |     |
ing the severity of their consequences and, if there are any, 6d8063f245d27eecce5aabe624b5785f2a77/target/i386/sev.c#L1121,
| designingandimplementingmitigationsthatpreserveseamless |     |     |     |     |     |     | QEMUteam.         |     |        |           |           |           |     |        |
| ------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ------ | --------- | --------- | --------- | --- | ------ |
|                                                         |     |     |     |     |     |     | [20] P. Antonino, | A.  | Derek, | and W. A. | Woloszyn, | “Flexible |     | Remote |
migration.
AttestationofPre-SNPSEVVMsUsingSGXEnclaves,”IEEEAccess,
vol.11,pp.90839–90856,Aug.2023.
|     |     |     |     |     |     |     | [21] M. Morbitzer, |     | M. Huber, | J. Horsch, | and | S. Wessel, | “Severed: |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --------- | ---------- | --- | ---------- | --------- | --- |
ACKNOWLEDGMENTS
|     |     |     |     |     |     |     | Subverting | amd’s | virtual | machine encryption,” |     | in Proceedings |     | of the |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ----- | ------- | -------------------- | --- | -------------- | --- | ------ |
This work has been supported by the European Union 11th European Workshop on Systems Security, ser. EuroSec’18. New
through the European Regional Development Fund, under York,NY,USA:AssociationforComputingMachinery,2018.
|     |     |     |     |     |     |     | [22] M. Radev | and | M.  | Morbitzer, “Exploiting |     | interfaces | of  | secure |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --- | ---------------------- | --- | ---------- | --- | ------ |
the grant KK.01.1.1.01.0009 (DATACROSS), and the project encrypted virtual machines,” in Reversing and Offensive-Oriented
AutoDataLog, a cooperation between the Faculty of Science TrendsSymposium,ser.ROOTS’20. NewYork,NY,USA:Association
and AVL-AST d.o.o. Croatia. We would also like to thank forComputingMachinery,2021,p.1–12.
|     |     |     |     |     |     |     | [23] M. Li, | Y. Zhang, | Z.  | Lin, and Y. | Solihin, | “Exploiting | unprotected |     |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --------- | --- | ----------- | -------- | ----------- | ----------- | --- |
UniversityComputingCenterofZagreb(SRCE)forproviding I/O operations in AMD’s secure encrypted virtualization,” in 28th
us with the computing resources. USENIXSecuritySymposium(USENIXSecurity19). SantaClara,CA:
USENIXAssociation,Aug.2019,pp.1257–1272.
|     |     |     |     |     |     |     | [24] R.Buhren,C.Werling,andJ.-P.Seifert,“Insecureuntilprovenupdated: |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
REFERENCES
|     |     |     |     |     |     |     | Analyzing | amd | sev’s remote | attestation,” | in  | Proceedings | of the | 2019 |
| --- | --- | --- | --- | --- | --- | --- | --------- | --- | ------------ | ------------- | --- | ----------- | ------ | ---- |
[1] “Building a Secure System using TrustZone Technology,” white ACMSIGSACConferenceonComputerandCommunicationsSecurity,
paper, https://documentation-service.arm.com/static/5f212796500e883a ser. CCS ’19. New York, NY, USA: Association for Computing
| b8e74531,ARMLimited,2005. |     |     |     |     |     |     | Machinery,2019,p.1087–1099. |     |     |     |     |     |     |     |
| ------------------------- | --- | --- | --- | --- | --- | --- | --------------------------- | --- | --- | --- | --- | --- | --- | --- |
[2] “Overview on Signing and Whitelisting for Intel® Software [25] L. Wilke, J. Wichelmann, M. Morbitzer, and T. Eisenbarth, “Sevurity:
Guard Extension (Intel® SGX) Enclaves,” white paper, No security without integrity : Breaking integrity-free memory
https://www.intel.com/content/dam/develop/external/us/en/documents/ encryption with minimal assumptions,” 2020 IEEE Symposium on
overview-signing-whitelisting-intel-sgx-enclaves.pdf,IntelCorporation, SecurityandPrivacy(SP),pp.1483–1496,2020.
2018. [26] M. Li, Y. Zhang, H. Wang, K. Li, and Y. Cheng, “TLB Poisoning
[3] “Secure Encrypted Virtualization API Version 0.24 (Revision 3.24),” AttacksonAMDSecureEncryptedVirtualization,”inAnnualComputer
technical preview, https://www.amd.com/content/dam/amd/en/docume Security Applications Conference, ser. ACSAC ’21. New York, NY,
nts/epyc-technical-docs/programmer-references/55766_SEV-KM_API USA:AssociationforComputingMachinery,Dec.2021,p.609–619.
_Specification.pdf,AdvancedMicroDevices,Inc.,2020. [27] L. Wilke, J. Wichelmann, F. Sieck, and T. Eisenbarth, “undeserved
[4] X.Li,X.Li,C.Dall,R.Gu,J.Nieh,Y.Sait,andG.Stockwell,“Design trust:Exploitingpermutation-agnosticremoteattestation,”in2021IEEE
and verification of the arm confidential compute architecture,” in 16th SecurityandPrivacyWorkshops(SPW),2021,pp.456–466.
USENIXSymposiumonOperatingSystemsDesignandImplementation
|     |     |     |     |     |     |     | [28] M.Li,Y.Zhang,andZ.Lin,“CrossLine:Breaking"Security-by-Crash" |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
(OSDI 22). Carlsbad, CA: USENIX Association, Jul. 2022, pp. Based Memory Isolation in AMD SEV,” in Proceedings of the 2021
465–484. ACMSIGSACConferenceonComputerandCommunicationsSecurity,
[5] D.Kaplan,“ProtectingVMRegisterStateWithSEV-ES,”whitepaper, ser. CCS ’21. New York, NY, USA: Association for Computing
https://www.amd.com/content/dam/amd/en/documents/epyc-business- Machinery,Nov.2021,p.2937–2950.
docs/white-papers/Protecting-VM-Register-State-with-SEV-ES.pdf, [29] W. Wang, M. Li, Y. Zhang, and Z. Lin, “Pwrleak: Exploiting
2017. power reporting interface for side-channel attacks on amd sev,” in
[6] “AzureConfidentialVMoptions,”https://learn.microsoft.com/en-us/azu Detection of Intrusions and Malware, and Vulnerability Assessment:
re/confidential-computing/virtual-machine-solutions, Microsoft Corpo- 20thInternationalConference,DIMVA2023,Hamburg,Germany,July
ration. 12–14,2023,Proceedings. Berlin,Heidelberg:Springer-Verlag,2023,
| [7] “UserguideforLinuxInstances:AMDSEV-SNP,”https://docs.aws.ama |     |     |     |     |     |     | p.46–66. |     |     |     |     |     |     |     |
| ---------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
zon.com/AWSEC2/latest/UserGuide/sev-snp.html,Amazon.com,Inc. [30] B. Schlüter, S. Sridhara, A. Bertschi, and S. Shinde, “WESEE: Using
[8] “Oh SNP! VMs get even more confidential,” https://cloud.google.com Malicious #VC Interrupts to Break AMD SEV-SNP,” in 2024 IEEE
/blog/products/identity-security/rsa-snp-vm-more-confidential, Google Symposium on Security and Privacy (SP). Los Alamitos, CA, USA:
| LLC. |     |     |     |     |     |     | IEEEComputerSociety,may2024,pp.238–238. |     |     |     |     |     |     |     |
| ---- | --- | --- | --- | --- | --- | --- | --------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
[9] “Strengthening VM isolation with integrity protection and more,” [31] B.Schlüter,S.Sridhara,M.Kuhne,A.Bertschi,andS.Shinde,“Heckler:
white paper, https://www.amd.com/system/files/TechDocs/56860.pdf, Breakingconfidentialvmswithmaliciousinterrupts,”in33rdUSENIX
AdvancedMicroDevices,Inc.,2020. SecuritySymposium(USENIXSecurity24),2024.
[10] Linus Torvalds, “Linux 5.19-rc1,” https://lore.kernel.org/lkml/CAHk-= [32] R. Buhren, H.-N. Jacob, T. Krachenfels, and J.-P. Seifert, “One glitch
wgZt-YDSKfdyES2p6A_KJoG8DwQ0mb9CeS8jZYp+0Y2Rw@mail.g torulethemall:Faultinjectionattacksagainstamd’ssecureencrypted
mail.com/T/#u,2022. virtualization,” in Proceedings of the 2021 ACM SIGSAC Conference
[11] “AMD-ASPFW,”https://github.com/amd/AMD-ASPFW,2023. on Computer and Communications Security, ser. CCS ’21. New
[12] “SEV Secure Nested Paging Firmware ABI Specification (Revi- York,NY,USA:AssociationforComputingMachinery,Nov.2021,p.
| sion | 1.55),” | https://www.amd.com/en/support/tech-docs/sev-secure-nes |     |     |     |     | 2875–2889. |     |     |     |     |     |     |     |
| ---- | ------- | ------------------------------------------------------- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
ted-paging-firmware-abi-specification, Advanced Micro Devices, Inc., [33] R. Zhang, L. Gerlach, D. Weber, L. Hetterich, Y. Lü, A. Kogler, and
Sep.2023. M.Schwarz,“CacheWarp:Software-basedfaultinjectionusingselective
[13] S.Meier,B.Schmidt,C.Cremers,andD.Basin,“TheTAMARINprover statereset,”in33rdUSENIXSecuritySymposium(USENIXSecurity24),
| for the | symbolic | analysis | of security | protocols,” | in Computer | Aided | 2024. |     |     |     |     |     |     |     |
| ------- | -------- | -------- | ----------- | ----------- | ----------- | ----- | ----- | --- | --- | --- | --- | --- | --- | --- |
17

[34] M. Li, Y. Zhang, H. Wang, K. Li, and Y. Cheng, “CIPHERLEAKS: Petar Paradžik received an MSc in Computer
BreakingConstant-timeCryptographyonAMDSEVviatheCiphertext Science and Mathematics from the Department of
SideChannel,”in30thUSENIXSecuritySymposium(USENIXSecurity Mathematics,FacultyofScience,UniversityofZa-
21). USENIXAssociation,Aug.2021,pp.717–732. greb. He is currently a PhD student and teaching
[35] M. Li, L. Wilke, J. Wichelmann, T. Eisenbarth, R. Teodorescu, and assistantattheFacultyofElectricalEngineeringand
Y. Zhang, “A Systematic Look at Ciphertext Side Channels on AMD ComputinginZagreb.Hisresearchinterestsinclude
SEV-SNP,”in2022IEEESymposiumonSecurityandPrivacy(SP),Jul. formalmethods,automatedverification,andapplied
| 2022,pp.337–351. |     |     |     |     |     |     |     | cryptography. |     |     |     |
| ---------------- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --- | --- |
[36] “AMDSecureProcessorforConfidentialComputing:SecurityReview,”
technicalReport,https://googleprojectzero.blogspot.com/2022/05/releas
| e-of-technical-report-into-amd.html, |     |     |     | Google | Project | Zero and | Google |     |     |     |     |
| ------------------------------------ | --- | --- | --- | ------ | ------- | -------- | ------ | --- | --- | --- | --- |
CloudSecurity,2022.
[37] Xu,ShiweiandZhao,YizhiandRen,ZhengweiandWu,Lingjuanand
Tong,YanandZhang,Huanguo,“ASymbolicModelforSystematically
AnalyzingTEE-BasedProtocols,”inInformationandCommunications Ante Derek (Member, IEEE) is currently an As-
Security,Meng,WeizhiandGollmann,DieterandJensen,ChristianD. sociate Professor with the Faculty of Electrical
andZhou,Jianying,Ed. Cham:SpringerInternationalPublishing,2020, Engineering and Computing, University of Zagreb.
| pp.126–144. |     |     |     |     |     |     |     | He participates | in a number | of national | and EU- |
| ----------- | --- | --- | --- | --- | --- | --- | --- | --------------- | ----------- | ----------- | ------- |
[38] R. Sinha, S. Rajamani, S. Seshia, and K. Vaswani, “Moat: Verifying fundedprojectsintheareaofcomputersecurity.His
confidentiality of enclave programs,” in Proceedings of the 22nd researchinterestsincludetheareaofapplyingformal
ACMSIGSACConferenceonComputerandCommunicationsSecurity,
|     |     |     |     |     |     |     |     | methods to | problems in computer | security, | privacy, |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | -------------------- | --------- | -------- |
ser. CCS ’15. New York, NY, USA: Association for Computing andcryptography.
Machinery,2015,p.1169–1184.
| [39] M. K. | Jangid, | G. Chen, | Y. Zhang, | and | Z. Lin, | “Towards | formal |     |     |     |     |
| ---------- | ------- | -------- | --------- | --- | ------- | -------- | ------ | --- | --- | --- | --- |
verificationofstatecontinuityforenclaveprograms,”in30thUSENIX
| Security | Symposium | (USENIX | Security | 21). | USENIX | Association, |     |     |     |     |     |
| -------- | --------- | ------- | -------- | ---- | ------ | ------------ | --- | --- | --- | --- | --- |
Aug.2021,pp.573–590.
[40] C.Jacomme,S.Kremer,andG.Scerri,“Symbolicmodelsforisolated
executionenvironments,”in2017IEEEEuropeanSymposiumonSecu- Marko Horvat received a DPhil in Computer
rityandPrivacy(EuroS&P),Jul.2017,pp.530–545. Science from the University of Oxford, UK. He
[41] P. Subramanyan, R. Sinha, I. Lebedev, S. Devadas, and S. A. Seshia, is currently working as Assistant Professor at the
“A formal foundation for secure remote execution of enclaves,” in Department of Mathematics, Faculty of Science,
Proceedings of the 2017 ACM SIGSAC Conference on Computer UniversityofZagreb,Croatia.Hisresearchinterests
and Communications Security, ser. CCS ’17. New York, NY, USA: rangefromformalverificationofsecurityprotocols
AssociationforComputingMachinery,2017,p.2435–2450. tocomputableanalysisandtopology.
| [42] E. Lanckriet,                         |           | M. Busi, | and D.     | Devriese,     | “πra: | A π-calculus    | for  |     |     |     |     |
| ------------------------------------------ | --------- | -------- | ---------- | ------------- | ----- | --------------- | ---- | --- | --- | --- | --- |
| verifying                                  | protocols | that     | use remote | attestation,” | in    | 2023 IEEE       | 36th |     |     |     |     |
| ComputerSecurityFoundationsSymposium(CSF). |           |          |            |               |       | LosAlamitos,CA, |      |     |     |     |     |
USA:IEEEComputerSociety,jul2023,pp.537–551.
| [43] S. Delaune, | S.  | Kremer, M. | D. Ryan, | and G. | Steel, | “A formal | analysis |     |     |     |     |
| ---------------- | --- | ---------- | -------- | ------ | ------ | --------- | -------- | --- | --- | --- | --- |
ofauthenticationinthetpm,”inFormalAspectsofSecurityandTrust,
| P.Degano,S.Etalle,andJ.Guttman,Eds. |     |     |     | Berlin,Heidelberg:Springer |     |     |     |     |     |     |     |
| ----------------------------------- | --- | --- | --- | -------------------------- | --- | --- | --- | --- | --- | --- | --- |
BerlinHeidelberg,2011,pp.111–125.
[44] J.Shao,Y.Qin,D.Feng,andW.Wang,“Formalanalysisofenhanced
| authorization   |     | in the tpm                             | 2.0,”    | in Proceedings | of             | the 10th | ACM       |     |     |     |     |
| --------------- | --- | -------------------------------------- | -------- | -------------- | -------------- | -------- | --------- | --- | --- | --- | --- |
| Symposium       | on  | Information,                           | Computer | and            | Communications |          | Security, |     |     |     |     |
| ser.ASIACCS’15. |     | NewYork,NY,USA:AssociationforComputing |          |                |                |          |           |     |     |     |     |
Machinery,2015,p.273–284.
| [45] M. U.                                          | Sardar, | R. Faqeh,   | and C.       | Fetzer, “Formal | Foundations |         | for Intel |     |     |     |     |
| --------------------------------------------------- | ------- | ----------- | ------------ | --------------- | ----------- | ------- | --------- | --- | --- | --- | --- |
|                                                     |         |             |              |                 | Formal      | Methods | and       |     |     |     |     |
| SGX Data                                            | Center  | Attestation | Primitives,” |                 | in          |         |           |     |     |     |     |
| SoftwareEngineering,S.-W.Lin,Z.Hou,andB.Mahony,Eds. |         |             |              |                 |             |         | Cham:     |     |     |     |     |
SpringerInternationalPublishing,2020,pp.268–283.
| [46] M. U.  | Sardar, | S. Musaev, | and C. | Fetzer, “Demystifying |     | attestation | in      |     |     |     |     |
| ----------- | ------- | ---------- | ------ | --------------------- | --- | ----------- | ------- | --- | --- | --- | --- |
| intel trust | domain  | extensions | via    | formal verification,” |     | IEEE        | Access, |     |     |     |     |
vol.9,pp.83067–83079,2021.
[47] M.U.Sardar,T.Fossati,S.Frost,andS.Xiong,“Formalspecification
andverificationofarchitecturally-definedattestationmechanismsinarm
ccaandinteltdx,”IEEEAccess,vol.12,pp.361–381,2024.
| [48] S. Wesemeyer, |     | C. J. Newton, | H.  | Treharne, | L. Chen, | R. Sasse, | and |     |     |     |     |
| ------------------ | --- | ------------- | --- | --------- | -------- | --------- | --- | --- | --- | --- | --- |
J.Whitefield,“Formalanalysisandimplementationofatpm2.0-based
| direct | anonymous | attestation | scheme,” | in  | Proceedings | of  | the 15th |     |     |     |     |
| ------ | --------- | ----------- | -------- | --- | ----------- | --- | -------- | --- | --- | --- | --- |
ACMAsiaConferenceonComputerandCommunicationsSecurity,ser.
| ASIA | CCS ’20. | New | York, NY, | USA: Association |     | for Computing |     |     |     |     |     |
| ---- | -------- | --- | --------- | ---------------- | --- | ------------- | --- | --- | --- | --- | --- |
Machinery,2020,p.784–798.
| [49] J. Whitefield, |     | L. Chen,    | R. Sasse, | S. Schneider, |     | H. Treharne,     | and |     |     |     |     |
| ------------------- | --- | ----------- | --------- | ------------- | --- | ---------------- | --- | --- | --- | --- | --- |
| S. Wesemeyer,       |     | “A symbolic | analysis  | of ecc-based  |     | direct anonymous |     |     |     |     |     |
attestation,”in2019IEEEEuropeanSymposiumonSecurityandPrivacy
(EuroS&P),2019,pp.127–141.
18
