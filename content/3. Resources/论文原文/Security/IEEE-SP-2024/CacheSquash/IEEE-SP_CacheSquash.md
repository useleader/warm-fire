---
publish: true
---

CacheSquash:
Making caches speculation-aware
Hossam ElAtali N. Asokan
University of Waterloo University of Waterloo
hossam.elatali@uwaterloo.ca asokan@acm.org
Abstract—Speculation is key to achieving high CPU perfor- cachestate).Notably,thetransmitstep,whichinvolvessending
mance, yet it enables risks like Spectre attacks which remain a request to the cache hierarchy, does not need to complete
a significant challenge to mitigate without incurring substantial
within the speculation window for the attack to succeed. The
performance overheads. These attacks typically unfold in three
core executes loads and issues the read requests. The cache
stages: access, transmit, and receive. Typically, they exploit a
cachetimingsidechannelduringthetransmitandreceivephases: hierarchy receivesrequests, and processes themto completion
speculatively accessing sensitive data (access), altering cache (potentially modifying cache state), irrespective of if/when
state (transmit), and then utilizing a cache timing attack (e.g., mis-speculation is detected. Any changes to cache state are
Flush+Reload)toextractthesecret(receive).Ourkeyobservation
persistent; attackers can probe them to leak information.
is that Spectre attacks only require the transmit instruction to
Despite efforts to mitigate it, Spectre remains a relevant
executeanddispatcharequesttothecachehierarchy.Itneednot
complete before a misprediction is detected (and mis-speculated threat, challenging the balance between security and perfor-
instructions squashed) because responses from memory that mance [2], [3]. Existing solutions [4], [5] aim for strong
arrive at the cache after squashing still alter cache state. security guarantees, but require significant performance (and
We propose a novel mitigation, CacheSquash, that cancels
area) overheads. In systems where performance is critical and
mis-speculated memory accesses. Immediately upon squashing,
attacker capabilities limited, such as network equipment [6],
a cancellation is sent to the cache hierarchy, propagating down-
stream and preventing any changes to caches that have not [7], the tradeoff is unacceptable, and a lightweight hardening
yet received a response. This minimizes cache state changes, mechanism with little to no performance impact is preferred.
thereby reducing the likelihood of Spectre attacks succeeding.
Inthispaper,weproposeCacheSquash,atechniquetomake
We implement CacheSquash on gem5 and show that it thwarts
the cache hierarchy speculation-aware by sending a cancella-
practical Spectre attacks, with near-zero performance overheads.
tion to the cache hierarchy as soon as an outstanding load
I. INTRODUCTION is squashed. The cancellation propagates down the hierarchy,
preventing cache state changes that have not yet occurred,
Speculationisafundamentaltechniqueemployedinmodern
thereby significantly reducing the likelihood of a successful
central processing units (CPUs) to optimize performance by
Spectre attack. Unlike prior work, we make a trade-off in
predicting and executing instructions ahead of time. Correct
favorofperformance.CacheSquashhardensprotectionagainst
predictionseliminatestallsintheprocessorpipeline,providing
Spectre-style attacks at near-zero overheads. Furthermore,
significantperformancegains.Anincorrectprediction,ormis-
CacheSquash only requires minimal control-logic changes
speculation, causes the offending instructions to be squashed,
to the CPU’s load-store unit and the caches’ miss-handling
their results discarded and the pipeline flushed to restart
circuitry,withnoadditionalstatestorage,unlikepriorfiltering
execution at the correct location.
or “cache-undo” approaches [5], [8]. It is applicable to any
While speculation is tightly integrated into CPU cores, the instruction set architecture (ISA) and requires no changes to
cachehierarchyinmodernCPUsisstillnotspeculation-aware. software or external hardware interfaces.
This means that loads executed speculatively will always be
We implement CacheSquash in gem5 and evaluate its per-
processed to completion by the cache hierarchy, even if the
formance under different configurations on the SPEC CPU
load is found to be a mis-speculation and squashed before
2017 and PARSEC benchmarks. We also evaluate its efficacy
theprocessingiscomplete.Cachestatechangescausedbythe
against various Spectre proofs-of-concept (PoCs) and provide
processing persist even after mis-speculation is detected. This
an analysis on its efficacy against real-world attacks.
results in a low attack barrier for Spectre attacks [1].
Our contributions are:
Spectre attacks exploit speculative execution to leak sensi-
tive information, such as cryptographic keys. They train the 1) CacheSquash,anISA-agnosticmechanismforcancelling
CPU’s branch prediction mechanism and then use it to tran- read requests upon squashing, requiring no changes to
siently access architecturally-inaccessible secrets in memory. software or external hardware interfaces(SectionIV),
The attack consists of three steps: accessing the secret, trans- 2) its implementation in gem5 (Section V)1,
mitting it through a side channel (e.g., changing cache state),
andreceiving/extractingitfromthesidechannel(e.g.,probing 1Wewillopen-sourcetheimplementation.
5202
yaM
8
]RC.sc[
2v01121.6042:viXra

3) performance evaluations showing a negligible geometric instructions that access sensitive data. Even though these
mean overhead (0.48%) on SPEC CPU 2017, and a instructions are eventually discarded, they can leave traces in
geometric mean speedup of 2.06% and overhead of the cache that can be exploited to infer the sensitive data. In
0.37% on the medium and large PARSEC benchmarks, other words, speculatively executed memory instructions can
respectively, (Section VII), and cause persistent changes to cache state.
4) case studies showing that CacheSquash is effective
III. PROBLEMDESCRIPTION
against several Spectre PoCs (Section VI).
A. Goals & Objectives
II. BACKGROUND
Ideally, desiderata for speculation-aware caches are:
CPUcaches.Cachesaresmall,high-speedmemorystructures
R1—Performance: no negative run-time performance im-
situated closer to the CPU cores compared to main memory.
pact on realistic workloads.
They are designed to store frequently-accessed data and in-
structions, thus reducing the time the CPU needs to fetch this
R2—Software Compatibility: require no changes to soft-
information from the slower main memory.
wareandbefullycompatiblewithexistingprogrambinaries.
Dataisstoredincachesinchunkscalled“lines”or“blocks”,
usually 64 bytes in size. Each cache line stores the data itself R3—HardwareCompatibility: requirenochangestoexter-
as well as a tag that identifies which address the data belongs nal interfaces (e.g., DRAM) or hardware components, other
to. When a cache receives a read or write request, it searches than the CPU.
its tags for one matching the request. If a match is found,
this is called a “cache hit”, and the cache can respond with R4—ISA Compatibility: be applicable to any ISA.
the data. Otherwise, a “cache miss” has occurred and must be
handled by a miss status holding register (MSHR). MSHRs R5—Effectiveness: reduce leakage of secret data through
are in charge of keeping track of outstanding misses, issuing cache state changes.
requestsdownstream,andservicingthemissesoncearesponse
is received. Each active MSHR is in charge of a single tag WedefineacachechangemetricCC foreffectiveness(R5):
(i.e., 64-byte aligned address). Multiple misses with the same
tag are added as “targets” to the same MSHR. Concretely, if CC =
(cid:80)K
i N i ×(K−i+1) (1)
there is already an outstanding MSHR with the same tag, the N × (cid:80)Ki
total i
miss is added to it as an additional target. Once a response is
K isthenumberofcachelevelsinthesystem.N isthe
received for this MSHR, all its targets are serviced. If there is total
total number of squashed access and transmit instructions in
nomatchingoutstandingMSHR,anemptyMSHRisallocated
a program. N is the number of squashed access and transmit
to the miss. Caches have a fixed number of MSHRs and a i
instructions that cause a change in the ith cache. We assign
maximumnumberoftargetsperMSHR;iftherearenoempty
more weight to changes to caches closer to the CPU because
MSHRs to handle a new miss or the matching MSHR has
they are easier to exploit via cache timing [11]. CC ∈ [0,1].
reached its maximum number of targets, the cache must stall.
Any non-zero value implies that Spectre attacks may succeed.
Speculation.Modernprocessorsemployspeculativeexecution
to improve performance by predicting and executing instruc- B. Threat model
tions ahead of time. This allows the processor to continue We consider the strongest Spectre threat model, where the
processing instructions even when there is a branch instruc- attacker and the victim execute within the same process,
tion whose outcome is uncertain. The speculation window sharing the same address space and having the same process
refers to the period during which instructions are executed context. The attacker is unable to directly access the victim’s
speculatively. The larger the speculation window, the more secret (e.g., due to in-process isolation mechanisms such as
instructions that can be executed before a squash occurs. sandboxing), but can train the branch predictor to access the
Cache timing attacks. Cache timing attacks, such as secret speculatively. This corresponds to “same-address in-
Flush+Reload [9] and Prime+Probe [10], exploit variations in place” (SA-IP) as defined by Canella et al [12]. A defense
thetimeittakesforaCPUtoaccesscachedvs.uncacheddata. thatworksunderthisstrongthreatmodel,isalsosecureunder
A cache hit takes less time to complete than a cache miss. weaker threat models such as where the attacker and victim
An attacker can compare the time it takes to access a certain are in different processes. We only consider cache-timing
address to determine whether the data at this address was channels; other channels, e.g., contention-based channels, are
cached. If a process uses secret-dependent memory addresses, out of scope.
this can leak information about the secret to the attacker.
Spectre. Spectre [1] attacks are a class of side-channel at-
IV. CACHESQUASH:DESIGN
tacks that exploit speculative execution. They use speculative The idea behind CacheSquash is to minimize cache state
loads to leak sensitive information across security boundaries. changes by issuing cancellations to outstanding speculative
By manipulating the CPU’s branch prediction mechanism, readrequests assoon astheyare squashed.Whenever acache
an attacker can force the execution of speculatively loaded receives a cancellation for an outstanding request, it drops
2

Core L1 L2 DRAM Core L1 L2 DRAM Core L1 L2 DRAM
L1 req (t L ra 1 n r s e m q it) (t L ra 1 n r s e m q it)
(transmit)
L2 req L2 req
L2 req
Mem req Mem req
Mem req
L1 cancel
L2 resp L2 resp
L2 cancel
L2 resp L1 cancel
L1 resp L1 resp
LSU resp
discards
response discards response
L1 unchanged L1,L2 already
L1,L2 unchanged L2 modified modified
(a) Best-case scenario (b) Intermediate (c) Worst-case
Fig. 1: The solid black activation bar (for core) represents the speculation window. The hollow activation bars (for caches)
representlifetimesoftherequests’MSHRs.Thehollowactivationbar(forDRAM)representsthememory-onlyaccesslatency.
the request from its MSHRs (and ignores any responses it
Cancellation Response
receives for it in the future), and, if appropriate, forwards the
cancellation to caches downstream.
Match discard No Check Yes Handle as
The final effect of the cancellation depends on the state of MSHR? No MSHR? usual
the read request/response within the cache hierarchy at the
No
Yes
time the cancellation is sent. We present all possible cases in
Figures 1a to 1c. In the best case, Figure 1a, the cancellation Remove MSHR Yes Forward
request from empty? cancellation
reaches the last-level cache (LLC), L2, before it receives a MSHR
responsefrommemory.Thispreventsanychangestothecache
Fig. 2: CacheSquash flow chart.
hierarchy as any response received by the LLC from memory
is ignored; subsequently, the LLC does not provide further
responses to caches upstream (which have already cancelled
the request and the corresponding evictions2 themselves). corresponding MSHR (MatchMSHR), and if so, remove the
Intheworstcase,Figure1c,thecancellationiseithernever cancelled request from it. If the MSHR then becomes empty
made(becausetheresponseisreceivedbytheCPUcorebefore (i.e.,nootherrequestsarewaitingforthiscacheblock),wecan
the speculation window ends), or it reaches L1 after it has sendacancellationtothelowercachelevel.Foraresponse,if
receivedaresponse.IftheCPUhasmorethanonecachelevel, amatchingMSHRisnotfound,itisdiscarded(CheckMSHR).
other intermediate cases between Figure 1a and Figure 1c can
1) MatchMSHR: MatchMSHR searches the MSHR queue
occur: Figure 1b shows a cancellation reaching L1, but not
for a match, i.e., one that has the same cache block address.
L2,beforetheresponse;onlyL2ismodifiedbytherequest.If
The circuitry to perform this search already exists in modern
thisrequestisfromaSpectretransmitinstruction,onlyattacks
caches. It is required to check if incoming misses match an
targeting the LLC can succeed; those targeting L1 will not.
outstanding MSHR, and coalesce requests to the same block.
ItispossibleforcancellationstofindnomatchingMSHRdue
A. Cache flow chart
to simultaneous transfers of responses and cancellations.
Figure 2 shows the CacheSquash flowchart. It works with
Simultaneous response and cancellation. Cache buses can
anycachecoherenceprotocol.Besidetheadditionofhandling
have several channels, allowing simultaneous bidirectional
cancellations, which is only relevant when a cache has an
communication, e.g., TileLink [13] or Arm AMBA CHI [14].
outstanding request to the level below it, the rest of the
This means that a cancellation can be received while a
coherence protocol remains unchanged. Outstanding requests
response is being sent. This can occur in two cases: 1) if
can either be reads (if the block is currently in the invalid
the original request hits in this cache, or 2) the original
state) or upgrades (if the block is valid, but does not have the
request misses, but, before the cancellation arrives, the cache
required permissions, e.g., writable). As soon as we receive
receives a response and the MSHR is serviced and freed. The
a cancellation, we only need to check whether there is a
cancellation in both cases will have no corresponding MSHR
and therefore must be discarded. This requires an MSHR
2Evictionsdonotrequirespecialhandlingastheyareonlycompletedwhen
theresponsearrives,ratherthanwhentheMSHRisfirstallocated. search to detect.
3

2) CheckMSHR: WithoutCacheSquash,MSHRsarelocked allow configurable cache coherence protocols. However, they
to a single downstream request until a response is received. currentlydonotsupportcachemaintenanceoperations,suchas
The downstream request contains an MSHR index which is flushing, and therefore cannot be used with the Flush+Reload
copiedintotheresponsebythedownstreamcache.Thismakes cache timing attack. As this is the attack used by the majority
it easy to determine the exact MSHR corresponding to a of the publicly available Spectre PoCs, we choose to imple-
response.WithCacheSquash,thisassumptionnolongerholds. ment CacheSquash on the classic caches. The classic caches
The cache must double-check the designated MSHR when have a fixed snooping-based cache coherence protocol, so we
receiving a response to ensure that it has not been freed and implementcancellationsnoopingasdescribedinSectionIV-B.
possibly reassigned to another block. This can happen either MatchMSHR & CheckMSHR. The logic for MatchMSHR
due to simultaneous response and cancellation transfer (as already exists for handling cache misses. We use the same
described above), or due to responses from memory. latency of searching the MSHR queue for cancellations.
| Responses | from | memory. |     | CacheSquash |     | does | not require |     |     |     |     |     |     |     |     |
| --------- | ---- | ------- | --- | ----------- | --- | ---- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
CheckMSHRaddsnewfunctionality.However,sincenosearch
modifications to DRAM or memory controllers (satisfying isrequired,onlyasimplecheck,weassumeitiscombinational
R3).Cancellationsarenotsupportedonthememorybus;LLCs logic and can be performed in the same cycle. CheckMSHR
must therefore not forward cancellations to memory. Thus, a thus incurs no additional latency.
request from the LLC to memory will eventually receive a O3 CPU. In addition to modifying the caches, we add
response, even if it is cancelled in all caches. LLCs need to CacheSquash support to gem5’s O3 model. Whenever an
detect whether the MSHR in the response still corresponds to instruction is squashed, the load-store unit checks if there are
the original request, hence the need for CheckMSHR. anyoutstandingmemoryrequestsfortheinstruction,andifso,
sendsacancellationtothecachehierarchy.Notethatwedonot
| B. Forwarding |     | cancellations | upstream |     |     |     |     |      |             |     |              |     |         |             |     |
| ------------- | --- | ------------- | -------- | --- | --- | --- | --- | ---- | ----------- | --- | ------------ | --- | ------- | ----------- | --- |
|               |     |               |          |     |     |     |     | make | any changes | to  | ISA-specific | CPU | models; | CacheSquash |     |
Cache coherence protocols ensure that requests for a cache is ISA-agnostic, satisfying R4.
| line get | an up-to-date |     | response. | The | protocol | can | probe up- |     |     |     |     |     |     |     |     |
| -------- | ------------- | --- | --------- | --- | -------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
stream caches for dirty cache lines, causing them to allocate VI. SECURITYEVALUATION
| an MSHR      | while | they       | probe         | caches | further | upstream. | With     | A. Case | studies |      |         |          |         |          |        |
| ------------ | ----- | ---------- | ------------- | ------ | ------- | --------- | -------- | ------- | ------- | ---- | ------- | -------- | ------- | -------- | ------ |
| CacheSquash, |       | forwarding | cancellations |        | is      | thus      | required | to      |         |      |         |          |         |          |        |
|              |       |            |               |        |         |           |          | We      | present | case | studies | with two | Spectre | variants | to em- |
ensuretheseMSHRsarefreed.Cachecoherenceprotocolsare pirically show the effectiveness of CacheSquash. We analyze
| mainly       | split into | two           | groups:  | snooping-based |            | and       | directory-  |             |          |           |        |                         |              |            |              |
| ------------ | ---------- | ------------- | -------- | -------------- | ---------- | --------- | ----------- | ----------- | -------- | --------- | ------ | ----------------------- | ------------ | ---------- | ------------ |
|              |            |               |          |                |            |           |             | the first   | (Section | VI-A1)    |        | to show the             | CPU          | events     | occuring     |
| based. In    | snooping,  | caches        | “snoop”  |                | the bus    | to detect | requests    |             |          |           |        |                         |              |            |              |
|              |            |               |          |                |            |           |             | throughout  |          | a Spectre | attack | and how                 | CacheSquash  |            | affects      |
| that require | their      | intervention. |          | This           | is usually |           | done using  |             |          |           |        |                         |              |            |              |
|              |            |               |          |                |            |           |             | them.       | For the  | second,   | we     | only report             | our findings |            | for brevity; |
| additional   | circuitry  | and           | can span | multiple       |            | cache     | levels. For |             |          |           |        |                         |              |            |              |
|              |            |               |          |                |            |           |             | the attacks |          | use the   | same   | access-transmit-receive |              | mechanism. |              |
CacheSquash, this circuitry can also be used to snoop on can- Note that since cancellations can be used for any squashed
| cellations.    | In directory-based |             |      | protocols, | a directory     |              | tracks the    |             |           |            |            |                  |             |           |             |
| -------------- | ------------------ | ----------- | ---- | ---------- | --------------- | ------------ | ------------- | ----------- | --------- | ---------- | ---------- | ---------------- | ----------- | --------- | ----------- |
|                |                    |             |      |            |                 |              |               | memory      | request,  |            | regardless | of the           | speculation |           | condition,  |
| caches holding |                    | each block. | A    | cache      | receiving       | a            | cancellations |             |           |            |            |                  |             |           |             |
|                |                    |             |      |            |                 |              |               | CacheSquash |           | is equally | applicable | to               | all Spectre | variants. |             |
| must consult   | its                | directory   | and  | forward    | a               | cancellation | to all        |             |           |            |            |                  |             |           |             |
|                |                    |             |      |            |                 |              |               | All         | PoCs      | define     | a secret   | string as        | the target  | of        | the attack. |
| caches with    | an                 | outstanding | MSHR | of         | a corresponding |              | target.       |             |           |            |            |                  |             |           |             |
|                |                    |             |      |            |                 |              |               | For each    | character |            | of the     | string, all PoCs | continue    |           | attempting  |
C. TLBs & I-Caches to leak the character until the extracted value matches certain
|              |     |            |           |     |         |     |              | criteria, | e.g., | value | is a valid | English | ASCII | character. | This |
| ------------ | --- | ---------- | --------- | --- | ------- | --- | ------------ | --------- | ----- | ----- | ---------- | ------- | ----- | ---------- | ---- |
| An important |     | cache-like | structure |     | in CPUs | is  | the transla- |           |       |       |            |         |       |            |      |
meansthatwhenattacksaresuccessful,theprogramterminates
| tion lookaside | buffer | (TLB), | which |     | caches | virtual-to-physical |     |     |     |     |     |     |     |     |     |
| -------------- | ------ | ------ | ----- | --- | ------ | ------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
address translations. Prior work has also shown that TLBs quickly. On the other hand, if attacks do not succeed, the
programrunsuntilittimesout.Thedefaulttimeoutperiodsfor
| can be         | used to | leak information |     | in   | a way        | similar | to cache |          |     |            |      |          |          |     |           |
| -------------- | ------- | ---------------- | --- | ---- | ------------ | ------- | -------- | -------- | --- | ---------- | ---- | -------- | -------- | --- | --------- |
|                |         |                  |     |      |              |         |          | the PoCs | are | infeasibly | long | when run | on gem5. | We  | therefore |
| timing attacks |         | [15]. Therefore, |     | with | CacheSquash, |         | we also  |          |     |            |      |          |          |     |           |
send cancellations for mis-speculated address translations that shorten all timeouts to allow the simulation to complete in a
|                 |            |              |            |             |         |                   |             | reasonable |            | amount   | of time.      | To ensure    | a fair   | comparison, | we        |
| --------------- | ---------- | ------------ | ---------- | ----------- | ------- | ----------------- | ----------- | ---------- | ---------- | -------- | ------------- | ------------ | -------- | ----------- | --------- |
| result in       | page-table | walks,       | reducing   |             | changes | to TLB            | state.      |            |            |          |               |              |          |             |           |
|                 |            |              |            |             |         |                   |             | verify     | that,      | without  | CacheSquash,  | both         | PoCs     | are still   | able to   |
| The instruction |            | cache        | is also    | affected    | by      | (mis-)speculation |             |            |            |          |               |              |          |             |           |
|                 |            |              |            |             |         |                   |             | leak       | the secret | using    | the shortened | timeouts.    |          |             |           |
| since the       | branch     | predictor    | speculates |             | on      | which             | path of in- |            |            |          |               |              |          |             |           |
|                 |            |              |            |             |         |                   |             | 1)         | Google     | SafeSide | –             | Spectre PHT: | SafeSide |             | [17] is a |
| structions      | will       | be executed. |            | To minimize |         | instruction       | cache       |            |            |          |               |              |          |             |           |
pollution, we also send cancellations to the instruction cache Google code repository containing several Spectre and Melt-
|         |             |       |                    |     |     |     |     | down  | [18]      | PoCs. We | use     | the spectre_v1_pht_sa |       |       | PoC,     |
| ------- | ----------- | ----- | ------------------ | --- | --- | --- | --- | ----- | --------- | -------- | ------- | --------------------- | ----- | ----- | -------- |
| when an | instruction | fetch | is mis-speculated. |     |     |     |     |       |           |          |         |                       |       |       |          |
|         |             |       |                    |     |     |     |     | which | mistrains | the      | pattern | history               | table | (PHT) | and then |
V. CACHESQUASH:IMPLEMENTATION
|     |     |     |     |     |     |     |     | exploits | it  | to achieve | a   | bounds check | bypass | (BCB). | The |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | ---------- | --- | ------------ | ------ | ------ | --- |
WeimplementCacheSquashongem5[16],acycle-accurate PHT is a component of the CPU’s branch predictor in charge
computer system simulator. gem5 includes a speculative out- of guessing whether a branch will be taken. The PoC uses
of-order CPU model, O3, and supports arbitrary cache con- Flush+Reload to transmit and receive the secret.
figurations. It provides two separate cache hierarchy imple- Listing 1 shows the x86 disassembly of the speculatively
mentations: classic and ruby. Ruby caches are newer and executed instructions. Lines 3-15 are executed speculatively
4

| # bounds | check |     |     |     |     |     |     |     |     |     |     |     |     |
| -------- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
1
| 404bec: | jae 404bbf | <main+0xca> |     |     |     |     |     |     |     |     |     |     |     |
| ------- | ---------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
2
| 3 404bee: | movsbq 0x0(%r13,%rax,1),%rax |     |     |     |     |     |     |     |     |     |     |     |     |
| --------- | ---------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 404bf4:   | imul $0x71,%rax,%rax         |     |     |     |     |     |     |     |     |     |     |     |     |
4
| 404bf8: | add $0x64,%rax |     |     |     |     |     |     |     |     |     |     |     |     |
| ------- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
5
| 6 # access | secret: data[local_offset] |     |     |     |     |     |     |     |     |     |     |     |     |
| ---------- | -------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 404bfc:    | movzbl %al,%eax            |     |     |     |     |     |     |     |     |     |     |     |     |
7
| 8 404bff: | add $0x1,%rax |     |     |     |     |     |     |     |     |     |     |     |     |
| --------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 404c03:   | mov %rax,%rsi |     |     |     |     |     |     |     |     |     |     |     |     |
9
| 404c06: | shl $0x6,%rsi |     |     |     |     |     |     |     |     |     |     |     |     |
| ------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
10
| 11 404c0a: | add %rsi,%rax |     |     |     |     |     |     |     |     |     |     |     |     |
| ---------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 404c0d:    | shl $0x6,%rax |     |     |     |     |     |     |     |     |     |     |     |     |
12
| 13 404c11:  | add 0x28(%rsp),%rax  |     |     |     |     |     |     |     |     |     |     |     |     |
| ----------- | -------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| # transmit: | timing_array[secret] |     |     |     |     |     |     |     |     |     |     |     |     |
14
| 404c16: | movzbl (%rax),%eax |     |     |     |     |     |     |     |     |     |     |     |     |
| ------- | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
15
| 16 404c19: | jmp 404bbf           | <main+0xca> |        |             |          |             |        |     |          |              |     |        |         |
| ---------- | -------------------- | ----------- | ------ | ----------- | -------- | ----------- | ------ | --- | -------- | ------------ | --- | ------ | ------- |
| Listing    | 1: spectre_v1_pht_sa |             | x86    | assembly    | extract. |             |        |     |          |              |     |        |         |
| until the  | failed bounds        | check on    | line 2 | is detected |          | and the     |        |     |          |              |     |        |         |
|            |                      |             |        |             |          | Fig. 3: CPU | events | for | transmit | instructions |     | of all | Spectre |
instructionsaresquashed.Beforethesquashoccurs,thesecret
|              |                  |                        |          |             |          | attack instances |          | from Experiment |         | 1.         | Each     | row represents | a         |
| ------------ | ---------------- | ---------------------- | -------- | ----------- | -------- | ---------------- | -------- | --------------- | ------- | ---------- | -------- | -------------- | --------- |
| is accessed  | (line 7)         | and transmitted        | across   | a           | side     | channel          |          |                 |         |            |          |                |           |
|              |                  |                        |          |             |          | single attack,   | the      | y-axis          | showing | the attack | number.  |                | All event |
| by modifying | the cache        | state (line            | 15).     | Extracting  | the      | secret           |          |                 |         |            |          |                |           |
|              |                  |                        |          |             |          | times are        | relative | to speculation  |         | start:     | for each | attack,        | events    |
| from the     | cache state is   | done non-speculatively |          | afterwards. |          | We               |          |                 |         |            |          |                |           |
|              |                  |                        |          |             |          | are shifted      | to make  | speculation     |         | start at   | t=0.     |                |           |
| now describe | four experiments | to                     | evaluate | the         | PoC with | and              |          |                 |         |            |          |                |           |
without CacheSquash.
|     |           |     |     |     |     | cached and | the          | access instruction |     | does        | not complete |                | in time |
| --- | --------- | --- | --- | --- | --- | ---------- | ------------ | ------------------ | --- | ----------- | ------------ | -------------- | ------- |
|     | Parameter |     | C1  |     | C2  |            |              |                    |     |             |              |                |         |
|     |           |     |     |     |     | to allow   | the transmit | instruction        |     | to execute. |              | But subsequent |         |
Corecount 2 2 attacks (e.g., attack 3) can access the now-cached secret
|     | Corefrequency(GHz) |     |     | 3   | 0.1 |     |     |     |     |     |     |     |     |
| --- | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
PrivateL1I/Dsize(kB) 32 32 quickly and thus have enough time to execute the transmit.
SharedL2size(kB) 512 512 In attack 3, the access instruction misses in L1 but hits in L2.
|     | L1I/L1D/L2associativity   |     | 8/8/16 | 8/8/16   |     |                 |     |                    |     |        |      |          |         |
| --- | ------------------------- | --- | ------ | -------- | --- | --------------- | --- | ------------------ | --- | ------ | ---- | -------- | ------- |
|     |                           |     |        |          |     | Experiment      | 2   | – C1, CacheSquash. |     | We     | run  | the PoC  | with C1 |
|     | L1I/L1D/L2latency(cycles) |     | 4/4/14 | 80/80/80 |     |                 |     |                    |     |        |      |          |         |
|     |                           |     |        |          |     | and CacheSquash |     | enabled,           | and | indeed | find | that the | program |
TABLE I: Configurations used in case study experiments. For times out without leaking any secrets. Our analysis leads to
C1, realistic values are used for L1 and L2 based on Intel aninterestingdiscovery:notransmitinstructionsareexecuted.
IceLake [19]. C2 is modified from C1 to intentionally make Upon further investigation, we find that this occurs because
|             |              |          |         |     |     | CacheSquash | cancels | the | access | instructions, |     | preventing | even |
| ----------- | ------------ | -------- | ------- | --- | --- | ----------- | ------- | --- | ------ | ------------- | --- | ---------- | ---- |
| CacheSquash | ineffective. | For main | memory, | we  | use | 3GB of      |         |     |        |               |     |            |      |
dual channel DDR4-2400. the first step of the attack. In its entire run, the PoC only
|     |     |     |     |     |     | finds the | secret | in the L1D | cache | twice, | and | in both | cases, |
| --- | --- | --- | --- | --- | --- | --------- | ------ | ---------- | ----- | ------ | --- | ------- | ------ |
Experiment 1 – C1, Baseline. We first run the PoC in gem5 speculation ends before the transmit is executed. In all other
using the O3 model and the C1 gem5 configuration shown in cases,theattackhasthesameresultasattack1inExperiment
|         |                      |     |     |         |            | 1 (Figure | 3). This | is shown | in  | Figure | 4, where | we  | plot the |
| ------- | -------------------- | --- | --- | ------- | ---------- | --------- | -------- | -------- | --- | ------ | -------- | --- | -------- |
| Table I | without CacheSquash. | The | PoC | is able | to extract | the       |          |          |     |        |          |     |          |
entire secret. We dump all load-store unit and cache events access instructions instead, and see that they hit in L1D only
from gem5, and identify and extract events related to pairs twice. Note that because Experiment 2 times out, the total
of the access and transmit instructions in Listing 1. Each number of attacks (102) is much larger than in Experiment 1
pair represents a single Spectre attack. Figure 3 plots relevant (31). However, for clarity, we only plot attacks #1–31.
eventsforthetransmitinstructionofeachattack,showingthat Experiment 3 – C1, CacheSquash, secret cached. Here,
for all attacks where the transmit instruction is executed, the we want to test the effectiveness of CacheSquash even when
squash occurs before any response is returned to the LLC. the secret is cached and the access instruction completes. We
Further, in almost all of those cases, there is a significant modifythePoCtoaccessthesecretnon-speculativelyonevery
delay between the squash occurring, and the LLC receiving a iteration. Running the PoC again results in a timeout with no
|           |               |                |     |                    |     | secrets leaked. |     | Our analysis | confirms |     | that the | access | instruc- |
| --------- | ------------- | -------------- | --- | ------------------ | --- | --------------- | --- | ------------ | -------- | --- | -------- | ------ | -------- |
| response. | This provides | an opportunity |     | for cancellations, |     | and             |     |              |          |     |          |        |          |
hintsthatCacheSquashmightpreventthisattack(seeournext tions now hit in L1D, and the transmit instructions execute.
experiment). Exceptions to this are the cases highlighted with We show all relevant events for the transmit instructions in
black boxes. These can pose a problem because cancellations Figure 5. Despite the execution of the transmit, no Spectre
might not reach L2 in timebefore the response from memory. attacksucceeds.Thisisbecausethecancellationsalwaysreach
|                 |     |                                   |     |     |     | the caches | before | the response, |     | even | for the | last-level | L2  |
| --------------- | --- | --------------------------------- | --- | --- | --- | ---------- | ------ | ------------- | --- | ---- | ------- | ---------- | --- |
| Wedonotfindthis | in  | ourfollowingexperimentswithC1,but |     |     |     |            |        |               |     |      |         |            |     |
we force this situation to occur in Experiment 4. cache, hence preventing any changes to cache state.
In attack 1 (y = 1 in Figure 3), the transmit instruction Experiment 4 – C2, CacheSquash, secret cached. Here, we
is never executed. This occurs because the secret is not yet intentionally change the system configuration to C2 to reduce
5

30
25
20
15
10
5
0
0 5 90 100 110 120 130 140
Relative Time (1e3 ticks)
#
kcattA
Spec start Mem req Squash
L1 req L1 resp L1 cancel
L2 req L2 resp L2 cancel
Fig. 4: CPU events for access instructions of Spectre attacks
#1–31 from Experiment 2. Attacks #32-102 are not shown.
Note the break in the x-axis. Cancellations have the same
latencyasregularrequests.Typically,cancellationsreachboth
L1 and L2 caches before a response is received. The secret is
thus almost never cached in a Spectre attack.
30
25
20
15
10
5
0
0 5 90 100 110 120 130 140
Relative Time (1e3 ticks)
#
kcattA
Fig. 6: CPU events for transmit instructions of all attacks
fromExperiment4.Inallbutthefirstattack,thecancellation
reaches L2 only after the response from memory has arrived.
step of the Spectre attack to succeed. This is due to the large
difference in hit access latencies for L1 and L2 (80 vs. 160,
Spec start Mem req Squash respectively) caused by the large latencies used for C2.
L1 req L1 resp L1 cancel Here, the PoC requires fewer attacks than in Experiment 1
L2 req L2 resp L2 cancel to extract the secret because we use the modified PoC where
we intentionally cache the secret before each Spectre attack.
Effectiveness. Table II shows the cache change metric, CC
(Equation(1))forallexperiments.Experiment1givesavalue
of 1 because CacheSquash is not enabled and any transmit
instruction executed results in changes to all caches. For
experiments 2 and 3, we get a value of 0 because no cache
changes occurred. Experiment 4 shows that when transmit
instructions manage to cause a partial change to cache state,
the value of CC is between 0 and 1.
Experiment N1 N2 N total CC
1 29 29 29 1
2 0 0 104 0
3 0 0 204 0
Fig. 5: CPU events for transmit instructions of attacks #1–
4 0 15 32 0.234
31 from Experiment 3. In all cases, the cancellations reach
L1 and L2 caches before a response is received. The transmit TABLE II: The values of CC for all experiments.
instruction thus never succeeds in changing cache state.
2) Google SafeSide – ret2spec: As before, we evaluate
CacheSquash on the ret2spec_sa PoC and verify that it
the effectiveness of CacheSquash. We drastically increase the thwarts the attack: no part of the secret is leaked. Ret2spec,
L1 and L2 latencies, and reduce the CPU clock frequency also called Spectre-RSB [12], [20], targets the return stack
to 100MHz, shown in red in Table I. Reducing the clock buffer (RSB), which predicts the addresses of return instruc-
frequencyeffectivelyincreasesthespeculationwindowrelative tions. It exploits the fact that the RSB has a limited number
to main memory latency (as main memory uses a separate of entries, and must remove addresses once it is filled due to
clock), making responses from memory more likely to arrive deeply nested functions, causing mispredictions of the return
within the speculation window. addresses.Thesemispredictionsleadtospeculativelyexecuted
RunningthemodifiedSpectrePoC(withsecretcaching),we instructions that are abused by ret2spec to leak the secret.
find that the entire secret is indeed extracted. Figure 6 shows
B. Security limitations
the relevant events. The key change, highlighted with a black
box, compared to Figure 5 is that the response reaches L2 1) Cache hits for transmit instructions: Flush+Reload uses
before the cancellation. This is not the case for L1; however, flushing to prepare caches before transmit instructions are
the cache state change in L2 alone is sufficient for the extract executed, preventing the cache line from being present at any
6

level. However, an attacker can instead use evictions [10] VII. PERFORMANCEEVALUATION
| to remove   | the       | data        | from        | one level   | but     | keep     | it in lower    |                     |      |               |          |             |          |                   |         |
| ----------- | --------- | ----------- | ----------- | ----------- | ------- | -------- | -------------- | ------------------- | ---- | ------------- | -------- | ----------- | -------- | ----------------- | ------- |
|             |           |             |             |             |         |          |                | We evaluate         |      | CacheSquash’s |          | performance |          | on 1-             | and 4-  |
| levels. For | example,  |             | an attacker | can         | evict   | the data | from L1        |                     |      |               |          |             |          |                   |         |
|             |           |             |             |             |         |          |                | core configurations |      | using         | similar  | parameters  |          | to Invisispec     | []      |
| but not     | L2 before | launching   |             | the         | Spectre | attack,  | and later      |                     |      |               |          |             |          |                   |         |
|             |           |             |             |             |         |          |                | and CleanupSpec     |      | [5].          | We first | run         | the SPEC | CPU               | 2017    |
| use the     | timing    | difference  | between     |             | an L1   | access   | and an L2      |                     |      |               |          |             |          |                   |         |
|             |           |             |             |             |         |          |                | benchmarks          | [33] | with          | the ref  | input       | size,    | using a           | warm-up |
| access to   | leak      | the secret. | While       | CacheSquash |         | is       | less effective |                     |      |               |          |             |          |                   |         |
|             |           |             |             |             |         |          |                | period of           | 10B  | instructions  | and      | measuring   |          | the instructions- |         |
againstsuchattacks,theyarealsomoredifficulttolaunchdue per-cycle (IPC) for the next 1B instructions. This follows
| to the smaller |         | timing           | difference.  | Further, |            | for architecturally |           |                       |           |             |              |            |            |                |         |
| -------------- | ------- | ---------------- | ------------ | -------- | ---------- | ------------------- | --------- | --------------------- | --------- | ----------- | ------------ | ---------- | ---------- | -------------- | ------- |
|                |         |                  |              |          |            |                     |           | standard              | procedure | from        | prior        | work       | [4], [5],  | [34].          | We ex-  |
| inaccessible   | secrets | that             | are          | flushed  | out on     | context             | switches, |                       |           |             |              |            |            |                |         |
|                |         |                  |              |          |            |                     |           | clude 507.cactuBSSN_r |           |             | as           | it crashes | on         | the baseline   | and     |
| the attacker   | must    | first            | successfully | cache    | the        | secret,             | which     | is                    |           |             |              |            |            |                |         |
|                |         |                  |              |          |            |                     |           | CacheSquash.          |           | The results | are          | shown      | in Figures | 7a             | and 7b. |
| made difficult |         | with CacheSquash |              | (see     | Experiment |                     | 2).       |                       |           |             |              |            |            |                |         |
|                |         |                  |              |          |            |                     |           | The geometric         |           | mean        | IPC overhead |            | across     | all benchmarks |         |
| 2) Windowing   |         | gadgets:         |              |          |            |                     |           |                       |           |             |              |            |            |                |         |
We demonstrated the effectiveness and configurations is −0.48%. However, the results show a
| of CacheSquash  |               | against      | Spectre         | PoCs,      | which         | were     | originally   |                    |            |                    |           |            |              |                 |           |
| --------------- | ------------- | ------------ | --------------- | ---------- | ------------- | -------- | ------------ | ------------------ | ---------- | ------------------ | --------- | ---------- | ------------ | --------------- | --------- |
|                 |               |              |                 |            |               |          |              | high geometric     |            | standard           | deviation | of         | 1.10. As     | our simulations |           |
| created         | to simply     | show         | the feasibility |            | of the        | attack.  | Attackers    |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              | are deterministic, |            | running            | the       | benchmarks |              | more            | than once |
| can use         | several       | techniques   |                 | to make    | attacks       |          | more robust. |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              | produces           | identical  | results.           | We        | therefore  | evaluate     | further         | using     |
| One is          | to increase   | the          | speculation     | window     |               | [21].    | Gigerl [22], |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              | the PARSEC         | benchmarks |                    | [35].     | We         | run them     | to completion   |           |
| Mambretti       | et al.        | [23]         | and Xiao        | et al.     | [24]          | identify | empirical    |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              | with the           | medium     | and                | large     | input      | sizes and    | measure         | the       |
| limits on       | speculation   |              | window          | sizes      | achievable    |          | on different |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              | number             | of gem5    | ticks              | (Figures  | 7c to      | 7f) yielding | a               | geometric |
| platforms       | via different |              | instructions    | for        | speculation   |          | conditions.  |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              | mean speedup       |            | of 2.06%           | and       | overhead   | of 0.37%     | with            | lower     |
| While           | increasing    | speculation  |                 | window     | size          | can      | reduce the   |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              | geometric          | standard   | deviations         |           | of 1.07    | and 1.008,   | respectively.   |           |
| effectiveness   | of            | CacheSquash, |                 | attackers  | must          | also     | overcome     |                    |            |                    |           |            |              |                 |           |
| other practical |               | challenges   | that            | maintain   | CacheSquash’s |          | effec-       |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              |                    |            | Parameter          |           |            | Value        |                 |           |
| tiveness.       | In the        | PoCs,        | both            | the victim | and           | attacker | code are     |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              |                    |            | Corecount          |           |            |              | 1,4             |           |
| under our       | control.      | In           | practice,       | attackers  | are           | forced   | to rely on   |                    |            |                    |           |            |              |                 |           |
|                 |               |              |                 |            |               |          |              |                    |            | Corefrequency(GHz) |           |            |              | 3               |           |
speculative gadgets [2], [3], [25], [26] present in the victim’s PrivateL1I/L1Dsize(kB) 32/64
code, in a manner similar to return-oriented programming SharedL2sizepercore(MB) 2
|               |     |          |               |         |               |          |          |     |     | L1/L2associativity        |     |     |        | 2/8 |     |
| ------------- | --- | -------- | ------------- | ------- | ------------- | -------- | -------- | --- | --- | ------------------------- | --- | --- | ------ | --- | --- |
| (ROP) gadgets |     | [27].    | This includes |         | disclosure    | gadgets, | used     |     |     |                           |     |     |        |     |     |
|               |     |          |               |         |               |          |          |     |     | L1I/L1D/L2latency(cycles) |     |     | 1/2/20 |     |     |
| to access     | and | transmit | the           | secret, | and windowing |          | gadgets, |     |     |                           |     |     |        |     |     |
neededtoincreasethespeculationwindowasdescribedabove. TABLE III: Parameters used in performance evaluation.
Significantpriorworkhasbeendonetoinvestigateandreduce
| the availability |     | of speculative |     | gadgets | in  | critical | software |     |     |     |     |     |     |     |     |
| ---------------- | --- | -------------- | --- | ------- | --- | -------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
VIII. DISCUSSION&FUTUREWORK
| targets such | as  | the Linux | Kernel | [2], | [28]. | As a | result, 1) at- |     |     |     |     |     |     |     |     |
| ------------ | --- | --------- | ------ | ---- | ----- | ---- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- |
tackersareincreasinglyforcedtouseless-than-idealdisclosure Meltdown. CacheSquash provides support for read request
gadgets that can have many redundant instructions (reducing cancellations regardless of the reason for cancellation. While
|     |     |     |     |     |     |     |     | we tackle | speculative |     | execution | in this | paper, | CacheSquash |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ----------- | --- | --------- | ------- | ------ | ----------- | --- |
theeffectiveavailablespeculationwindow),and2)fewerwin-
dowing gadgets are available, making it harder to circumvent is also applicable to fault-based transient attacks such as
CacheSquash. CacheSquash serves as complementary work to Meltdown [18]. Any transmit instruction executed transiently
reduce the effective attack surface of critical software targets. during the Meltdown attack can be cancelled once the fault is
3) Speculative-interference attacks: A crucial requirement detected, thereby reducing cache state changes and reducing
forCacheSquashisthatthetransmitinstructionisspeculative, the attack’s chance of success.
and is therefore squashed when the speculation window ends. Cancellation broadcasts. Dedicated circuitry, similar to that
In speculative interference attacks [29], however, this is not used for snooping, can be added to the CPU die to broadcast
the case. Instead, speculative execution is used to affect the cancellations to all caches, even if a snooping protocol is
order of non-speculative loads/stores, resulting in a cache not used. This can drastically improve the security provided
state difference that can later be measured to leak the secret. by CacheSquash, by eliminating the dependency on cache
As non-speculative memory requests cannot be cancelled, forwarding latency. However, this adds complexity to can-
CacheSquash cannot thwart such attacks. We consider them cellation handling because lower-level caches would now get
out-of-scopeandrelyonorthogonaldefenses,e.g.,fullpipelin- cancellations even if the corresponding upstream MSHR is
ing to prevent speculative instructions from affecting older not empty. A mechanism must therefore be added to allow
ones, as suggested by Behnia et al [29]. lower-level caches to track upstream MSHRs and only act on
4) Non-cache-basedsidechannels: CacheSquashworksby cancellations once the upstream MSHR is deallocated.
reducing persistent secret-dependent changes to cache state. Cancellation of memory bus transactions. In Section III-A,
As a result, CacheSquash only covers cache-based side chan- weexplicitlyavoidchangestoexternalmodulesandinterfaces
nels.Othersidechannels,e.g.contention-basedchannels[30], such as main memory (R3) to enable backward compat-
[31], [32], are out-of-scope, as in many invisible speculation ibility of CacheSquash. However, introducing cancellations
schemes (Section IX). to memory buses can be an opportunity to improve system
7

core count = 1 core count = 4 core count = 1 core count = 1 core count = 4 core count = 4
|     |     |     |     |     | baseline    |     |     | baseline    | baseline    |     |     |
| --- | --- | --- | --- | --- | ----------- | --- | --- | ----------- | ----------- | --- | --- |
|     |     |     | 2.5 |     |             | 2.5 |     | 1013        |             |     |     |
|     |     |     |     |     | CacheSquash |     |     | CacheSquash | CacheSquash |     |     |
1012
|     |     |     | 2.0 |     |     | 2.0 |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
1012
|     |     |     | 1.5 |     |     | 1012 1.5  |     |             |     |       |     |
| --- | --- | --- | --- | --- | --- | --------- | --- | ----------- | --- | ----- | --- |
|     |     |     | CPI |     |     | skcit CPI |     | skcit skcit |     | skcit |     |
|     |     |     | 1.0 |     |     | 1.0       |     |             |     |       |     |
1012 1011
|     |     |     | 0.5                                                                      |                                                                                                      |                                                                                         | 0.5                                                                      |                                                                                                      |                                                                                         |     |      |     |
| --- | --- | --- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --- | ---- | --- |
|     |     |     |                                                                          |                                                                                                      |                                                                                         | 1011                                                                     |                                                                                                      |                                                                                         |     | 1011 |     |
|     |     |     | 0.0                                                                      |                                                                                                      |                                                                                         | 0.0                                                                      |                                                                                                      |                                                                                         |     |      |     |
|     |     |     | r_hcneblrep.005 r_ccg.205 r_sevawb.305 r_fcm.505 r_dman.805 r_tserap.015 | r_yarvop.115 r_mbl.915 r_pptenmo.025 r_frw.125 r_kmbcnalax.325 r_462x.525 r_4mac.725 r_gnejspeed.135 | r_kcigami.835 r_aleel.145 r_ban.445 r_2egnahcxe.845 r_d3kinotof.945 r_smor.455 r_zx.755 | r_hcneblrep.005 r_ccg.205 r_sevawb.305 r_fcm.505 r_dman.805 r_tserap.015 | r_yarvop.115 r_mbl.915 r_pptenmo.025 r_frw.125 r_kmbcnalax.325 r_462x.525 r_4mac.725 r_gnejspeed.135 | r_kcigami.835 r_aleel.145 r_ban.445 r_2egnahcxe.845 r_d3kinotof.945 r_smor.455 r_zx.755 |     |      |     |
selohcskcalb kcartydob laennac puded misecaf terref etaminadiulf enimqerf ecartyar retsulcmaerts snoitpaws spiv selohcskcalb selohcskcalb kcartydob kcartydob laennac laennac puded puded misecaf misecaf terref terref etaminadiulf etaminadiulf enimqerf enimqerf ecartyar ecartyar retsulcmaerts retsulcmaerts snoitpaws snoitpaws spiv spiv selohcskcalb kcartydob laennac puded misecaf terref etaminadiulf enimqerf ecartyar retsulcmaerts snoitpaws spiv
|     |     |     |     | benchmark |     |     | benchmark benchmark |     | benchmark benchmark |     | benchmark |
| --- | --- | --- | --- | --------- | --- | --- | ------------------- | --- | ------------------- | --- | --------- |
(a) SPEC CPU 2017 1-core (c) PARSEC 1-core medium input size (e) PARSEC 1-core large input size
core count = 1 core count = 4 core count = 1 core count = 1 core count = 4 core count = 4
| 2.5 |     | baseline    | 2.5 |     | baseline    |      | baseline    |     |     |     |     |
| --- | --- | ----------- | --- | --- | ----------- | ---- | ----------- | --- | --- | --- | --- |
|     |     | CacheSquash |     |     | CacheSquash | 1013 | CacheSquash |     |     |     |     |
1012
| 2.0 |     |     | 2.0 |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
1012
| CPI 1.5 |     |     | CPI 1012 1.5 |     |     |             |     |       |     |     |     |
| ------- | --- | --- | ------------ | --- | --- | ----------- | --- | ----- | --- | --- | --- |
|         |     |     | skcit        |     |     | skcit skcit |     | skcit |     |     |     |
| 1.0     |     |     | 1.0          |     |     |             |     |       |     |     |     |
1012 1011
| 0.5 |     |     | 0.5 |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
1011 1011
| 0.0                                                                      |                                                                                                      |                                                                                         | 0.0                                                                      |                                                                                                      |                                                                                         |     |     |     |     |     |     |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
| r_hcneblrep.005 r_ccg.205 r_sevawb.305 r_fcm.505 r_dman.805 r_tserap.015 | r_yarvop.115 r_mbl.915 r_pptenmo.025 r_frw.125 r_kmbcnalax.325 r_462x.525 r_4mac.725 r_gnejspeed.135 | r_kcigami.835 r_aleel.145 r_ban.445 r_2egnahcxe.845 r_d3kinotof.945 r_smor.455 r_zx.755 | r_hcneblrep.005 r_ccg.205 r_sevawb.305 r_fcm.505 r_dman.805 r_tserap.015 | r_yarvop.115 r_mbl.915 r_pptenmo.025 r_frw.125 r_kmbcnalax.325 r_462x.525 r_4mac.725 r_gnejspeed.135 | r_kcigami.835 r_aleel.145 r_ban.445 r_2egnahcxe.845 r_d3kinotof.945 r_smor.455 r_zx.755 |     |     |     |     |     |     |
selohcskcalb kcartydob laennac puded misecaf terref etaminadiulf enimqerf ecartyar retsulcmaerts snoitpaws spiv selohcskcalb selohcskcalb kcartydob kcartydob laennac laennac puded puded misecaf misecaf terref terref etaminadiulf etaminadiulf enimqerf enimqerf ecartyar ecartyar retsulcmaerts retsulcmaerts snoitpaws snoitpaws spiv spiv selohcskcalb kcartydob laennac puded misecaf terref etaminadiulf enimqerf ecartyar retsulcmaerts snoitpaws spiv
|     | benchmark |     |     | benchmark benchmark |     |     | benchmark benchmark |     | benchmark |     |     |
| --- | --------- | --- | --- | ------------------- | --- | --- | ------------------- | --- | --------- | --- | --- |
(b) SPEC CPU 2017 4-core (d) PARSEC 4-core medium input size (f) PARSEC 4-core large input size
Fig. 7: Results for SPEC CPU 2017 (IPC) with ref input size and PARSEC (ticks) with medium and large input sizes on 1-
|     |     | and | 4-core configurations | of  | baseline and | CacheSquash. |     |     |     |     |     |
| --- | --- | --- | --------------------- | --- | ------------ | ------------ | --- | --- | --- | --- | --- |
performance. By aborting transactions that are no longer non-speculativerequest,theyhavenoneedforaSpectreattack
needed,wecanfreeupthememorybusforothertransactions. and can simply use a non-transient cache timing attack.
Furthermore,formemorieswithintegratedon-chipcaches,this
IX. RELATEDWORK
|     |     | can | improve security | by cancelling | changes | to the on-chip |     |     |     |     |     |
| --- | --- | --- | ---------------- | ------------- | ------- | -------------- | --- | --- | --- | --- | --- |
Invisiblespeculationmechanisms[4],[8],[36],[37],[38]at-
|     |     | cache. | We leave | such research | for future work. |     |     |     |     |     |     |
| --- | --- | ------ | -------- | ------------- | ---------------- | --- | --- | --- | --- | --- | --- |
tempttohidespeculativesideeffectsuntiltheyaredetermined
Overlapping cancelled and uncancelled requests. If there to be non-speculative. While hidden, speculative changes are
are n requests waiting for the same cache block, canceling stored in shadow structures which are invisible to the rest of
up to (n−1) of them will have no effect on the cache as thenon-speculativesystem.CleanupSpec[5]takesan“undo”,
the MSHR must still be serviced. Information leakage can rather than a hiding, approach, allowing speculative changes
occur if the first request to allocate the MSHR is cancelled, to be seen by the system, but undoing them on squashes.
but the MSHR cannot be deallocated due to the existence However, prior schemes suffer from significant overheads,
of other non-speculative targets that arrived later. There is a (e.g., 21−72% for InvisiSpec) and/or require the addition of
timing difference between when the response arrives in this expensive on-chip storage to track speculative changes (e.g.,
case, and when the response would have arrived had there not L0 in MuonTrap, buffers in CleanupSpec). In comparison,
been the first request (i.e., the MSHR was instead allocated CacheSquash does not require any structures to track spec-
by the non-speculative second request). While this timing ulative changes and has negligible overheads.
difference can theoretically be used to leak information, our Speculative Taint Tracking (STT) [34] is another Spectre
threat model assumes that this second request is not under mitigation technique that taints data loaded by Spectre access
the attacker’s control, and they cannot determine the time at instructions and delays any instructions that use it until the
which it occurs (and therefore cannot accurately measure the access instruction becomes non-speculative. While STT is not
timing difference). Note that if the attacker can control this limited to protecting only cache-based side channels, it can
8

result in significant overheads (8.5 − 14.5%) compared to [18] M. Lipp, M. Schwarz, D. Gruss, T. Prescher, W. Haas, A. Fogh,
CacheSquash and does not cover the case where only the J.Horn,S.Mangard,P.Kocher,D.Genkin,Y.Yarom,andM.Hamburg,
“Meltdown:Readingkernelmemoryfromuserspace,”in27thUSENIX
transmit, but not the access, instruction is speculative.
SecuritySymposium. USENIXAssociation,2018.
[19] A. Fog, “The microarchitecture of intel, amd and via cpus: An
ACKNOWLEDGMENTS optimization guide for assembly programmers and compiler makers,”
2024. [Online]. Available: https://www.agner.org/optimize/microarchit
This work is supported in part by Natural Sciences and En-
ecture.pdf
gineeringResearchCouncilofCanada(grantnumberRGPIN- [20] E.M.Koruyeh,K.N.Khasawneh,C.Song,andN.B.Abu-Ghazaleh,
2020-04744). Views expressed in the paper are those of the “Spectre returns! speculation attacks using the return stack buffer,” in
12thUSENIXWorkshoponOffensiveTechnologies,WOOT. USENIX
authors and do not necessarily reflect the position of the
Association,2018.
funders. [21] Microsoft, “Mitigating speculative execution side channel hardware
vulnerabilities,”2018.[Online].Available:https://msrc.microsoft.com/b
REFERENCES log/2018/03/mitigating-speculative-execution-side-channel-hardware-v
ulnerabilities/
[1] P.Kocher,J.Horn,A.Fogh,D.Genkin,D.Gruss,W.Haas,M.Ham- [22] B. Gigerl, “Automated Analysis of Speculation Windows in Spectre
burg, M. Lipp, S. Mangard, T. Prescher, M. Schwarz, and Y. Yarom, Attacks,”Master’sthesis,GrazUniversityofTechnology,2019.
“Spectreattacks:Exploitingspeculativeexecution,”in2019IEEESym- [23] A. Mambretti, M. Neugschwandtner, A. Sorniotti, E. Kirda, W. K.
posiumonSecurityandPrivacy. IEEE,2019. Robertson, and A. Kurmus, “Speculator: a tool to analyze speculative
[2] B. Johannesmeyer, J. Koschel, K. Razavi, H. Bos, and C. Giuffrida, execution attacks and mitigations,” in Proceedings of the 35th Annual
“Kasper: Scanning for generalized transient execution gadgets in the ComputerSecurityApplicationsConference. ACM,2019.
linuxkernel,”in29thAnnualNetworkandDistributedSystemSecurity [24] Y.Xiao,Y.Zhang,andR.Teodorescu,“SPEECHMINER:Aframework
Symposium. TheInternetSociety,2022. forinvestigatingandmeasuringspeculativeexecutionvulnerabilities,”in
[3] S. Wiebing, A. de Faveri Tron, H. Bos, and C. Giuffrida, “InSpectre 27thAnnualNetworkandDistributedSystemSecuritySymposium. The
Gadget:InspectingtheResidualAttackSurfaceofCross-privilegeSpec- InternetSociety,2020.
tre v2,” in 33rd USENIX Security Symposium. USENIX Association, [25] Intel, “Refined speculative execution terminology,” 2022. [Online].
2024. Available: https://www.intel.com/content/www/us/en/developer/articles
[4] M. Yan, J. Choi, D. Skarlatos, A. Morrison, C. W. Fletcher, and /technical/software-security-guidance/best-practices/refined-speculative
J. Torrellas, “Invisispec: Making speculative execution invisible in the -execution-terminology.html
cache hierarchy,” in 51st Annual IEEE/ACM International Symposium [26] A. Bhattacharyya, A. Sa´nchez, E. M. Koruyeh, N. B. Abu-Ghazaleh,
onMicroarchitecture. IEEEComputerSociety,2018. C. Song, and M. Payer, “Specrop: Speculative exploitation of ROP
[5] G.SaileshwarandM.K.Qureshi,“Cleanupspec:An”undo”approach chains,” in 23rd International Symposium on Research in Attacks,
to safe speculation,” in Proceedings of the 52nd Annual IEEE/ACM IntrusionsandDefenses. USENIXAssociation,2020.
InternationalSymposiumonMicroarchitecture. ACM,2019. [27] H.Shacham,“Thegeometryofinnocentfleshonthebone:return-into-
[6] I. Cisco Systems, “Cpu side-channel information disclosure libc without function calls (on the x86),” in Proceedings of the 2007
vulnerabilities,” 2018. [Online]. Available: https://sec.cloudapps.ci ACMConferenceonComputerandCommunicationsSecurity. ACM,
sco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20180 2007.
104-cpusidechannel [28] Intel, “Intel research on disclosure gadgets at indirect branch
[7] I. Juniper Networks, “Cpu side-channel information disclosure targets in the linux kernel,” 2022. [Online]. Available: https:
vulnerabilities,”2018.[Online].Available:https://supportportal.juniper. //www.intel.com/content/www/us/en/developer/articles/news/update-t
net/s/article/2018-01-Out-of-Cycle-Security-Bulletin-Meltdown-Spect o-research-on-disclosure-gadgets-in-linux.html
re-CPU-Speculative-Execution-and-Indirect-Branch-Prediction-Side-C [29] M.Behnia,P.Sahu,R.Paccagnella,J.Yu,Z.N.Zhao,X.Zou,T.Un-
hannel-Analysis-Method terluggauer,J.Torrellas,C.V.Rozas,A.Morrison,F.McKeen,F.Liu,
[8] S. Ainsworth and T. M. Jones, “Muontrap: Preventing cross-domain R.Gabor,C.W.Fletcher,A.Basak,andA.R.Alameldeen,“Speculative
spectre-likeattacksbycapturingspeculativestate,”in47thACM/IEEE interferenceattacks:breakinginvisiblespeculationschemes,”inASPLOS
Annual International Symposium on Computer Architecture. IEEE, ’21: 26th ACM International Conference on Architectural Support for
2020. ProgrammingLanguagesandOperatingSystems. ACM,2021.
[9] Y.YaromandK.Falkner,“FLUSH+RELOAD:Ahighresolution,low [30] M. Taram, X. Ren, A. Venkat, and D. M. Tullsen, “Secsmt: Securing
noise,L3cacheside-channelattack,”inProceedingsofthe23rdUSENIX SMT processors against contention-based covert channels,” in 31st
SecuritySymposium. USENIXAssociation,2014. USENIXSecuritySymposium. USENIXAssociation,2022.
[10] D. A. Osvik, A. Shamir, and E. Tromer, “Cache attacks and counter- [31] Z.N.Zhao,A.Morrison,C.W.Fletcher,andJ.Torrellas,“Binoculars:
measures: The case of AES,” in Topics in Cryptology - CT-RSA 2006, Contention-based side-channel attacks exploiting the page walker,” in
TheCryptographers’TrackattheRSAConference. Springer,2006. 31stUSENIXSecuritySymposium. USENIXAssociation,2022.
[11] F. Liu, Y. Yarom, Q. Ge, G. Heiser, and R. B. Lee, “Last-level cache [32] R.Paccagnella,L.Luo,andC.W.Fletcher,“Lordofthering(s):Side
side-channelattacksarepractical,”in2015IEEESymposiumonSecurity channelattacksontheCPUon-chipringinterconnectarepractical,”in
andPrivacy. IEEEComputerSociety,2015. 30thUSENIXSecuritySymposium. USENIXAssociation,2021.
[12] C.Canella,J.V.Bulck,M.Schwarz,M.Lipp,B.vonBerg,P.Ortner, [33] Standard Performance Evaluation Corporation, “SPEC CPU 2017
F. Piessens, D. Evtyushkin, and D. Gruss, “A systematic evaluation benchmark,”2017.[Online].Available:https://www.spec.org/cpu2017/
of transient execution attacks and defenses,” in 28th USENIX Security [34] J. Yu, M. Yan, A. Khyzha, A. Morrison, J. Torrellas, and C. W.
Symposium. USENIXAssociation,2019. Fletcher,“Speculativetainttracking(STT):Acomprehensiveprotection
[13] SiFive, “SiFive TileLink specification,” 2017. [Online]. Available: for speculatively accessed data,” in Proceedings of the 52nd Annual
https://static.dev.sifive.com/docs/tilelink/tilelink-spec-1.7-draft.pdf IEEE/ACM International Symposium on Microarchitecture. ACM,
[14] Arm, “Amba chi architecture specification,” 2024. [Online]. Available: 2019.
https://developer.arm.com/documentation/ihi0050/latest/ [35] C.Bienia,S.Kumar,J.P.Singh,andK.Li,“ThePARSECbenchmark
[15] B. Gras, K. Razavi, H. Bos, and C. Giuffrida, “Translation leak-aside suite: characterization and architectural implications,” in Proceedings
buffer:Defeatingcacheside-channelprotectionswithTLBattacks,”in of the 17th international conference on Parallel architectures and
27thUSENIXSecuritySymposium. USENIXAssociation,2018. compilationtechniques. ACM,2008.
[16] N. L. Binkert, B. M. Beckmann, G. Black, S. K. Reinhardt, A. G. [36] K. N. Khasawneh, E. M. Koruyeh, C. Song, D. Evtyushkin, D. Pono-
Saidi,A.Basu,J.Hestness,D.Hower,T.Krishna,S.Sardashti,R.Sen, marev, and N. B. Abu-Ghazaleh, “Safespec: Banishing the spectre of
K.Sewell,M.S.B.Altaf,N.Vaish,M.D.Hill,andD.A.Wood,“The ameltdownwithleakage-freespeculation,”inProceedingsofthe56th
gem5simulator,”SIGARCHComput.Archit.News,2011. AnnualDesignAutomationConference2019. ACM,2019.
[17] Google,“Safeside,”2020.[Online].Available:https://github.com/googl [37] C. Sakalis, S. Kaxiras, A. Ros, A. Jimborean, and M. Sja¨lander,
e/safeside/tree/main “Efficient invisible speculative execution through selective delay and
9

|                         |              | Proceedings       | of the 46th International | Symposium |
| ----------------------- | ------------ | ----------------- | ------------------------- | --------- |
| value                   | prediction,” | in                |                           |           |
| onComputerArchitecture. |              | ACM,2019.         |                           |           |
| [38] P. Li,             | L. Zhao,     | R. Hou, L. Zhang, | and D. Meng, “Conditional | spec-     |
| ulation:                | An effective | approach          | to safeguard out-of-order | execution |
againstspectreattacks,”in25thIEEEInternationalSymposiumonHigh
| PerformanceComputerArchitecture. |     |     | IEEE,2019. |     |
| -------------------------------- | --- | --- | ---------- | --- |
10
