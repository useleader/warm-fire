---
publish: true
---

Adversarial Prefetch: New Cross-Core Cache Side
Channel Attacks
Yanan Guo1, Andrew Zigerelli, Youtao Zhang2, and Jun Yang1
1Electrical and Computer Engineering Department, University of Pittsburgh
2Department of Computer Science, University of Pittsburgh
yag45@pitt.edu, zhangyt@cs.pitt.edu, juy9@pitt.edu
Abstract—Modernx86processorshavemanyprefetchinstruc- variances to stealthily transfer data (in the covert channel case)
tions that can be used by programmers to boost performance. or infer some secrets from a victim (in the side channel case)
However, these instructions may also cause security problems.
such as cryptographic keys.
In particular, we found that on Intel processors, there are
A critical step in most cache attacks is evicting the victim’s
two security flaws in the implementation of PREFETCHW, an
instruction for accelerating future writes. First, this instruction data from a cache level. Based on how the attacker evicts the
can execute on data with read-only permission. Second, the victim’s data, most cache attacks can be classified into flush-
execution time of this instruction leaks the current coherence based attacks [1], [2], [7], [37] and conflict-based attacks [4],
state of the target data.
[8], [38]. Flush-based attacks usually assume data sharing
Based on these two design issues, we build two cross-core
between the attacker and victim. Thus, the attacker directly
private cache attacks that work with both inclusive and non-
performs CLFLUSH on the victim’s data to evict it from all
inclusive LLCs, named Prefetch+Reload and Prefetch+Prefetch.
We demonstrate the significance of our attacks in different cache levels. For conflict-based attacks, the attacker instead
scenarios. First, in the covert channel case, Prefetch+Reload achieves the eviction by constructing set conflicts, i.e., the
and Prefetch+Prefetch achieve 782 KB/s and 822 KB/s channel attackerfillsthecacheset(thatthevictim’sdataoccupies)with
capacities, when using only one shared cache line between the
his own data. Many secure cache designs have been proposed
sender and receiver, the largest-to-date single-line capacities for
to defend cache attacks. For example, flush-based attacks can
CPU cache covert channels. Further, in the side channel case,
our attacks can monitor the access pattern of the victim on be prevented by modifying CLFLUSH (to make it a privileged
the same processor, with almost zero error rate. We show that instruction),assuggestedinpriorwork[1],[39]–[41].Conflict-
they can be used to leak private information of real-world based attacks can be defended by stopping/limiting attackers
applicationssuchascryptographickeys.Finally,ourattackscan
from discovering congruent addresses [39], [40], [42]. Thus,
be used in transient execution attacks in order to leak more
in this work we present new cache eviction methods to enable
secrets within the transient window than prior work. From the
experimental results, our attacks allow leaking about 2 times as practical cache attacks.
many secret bytes, compared to Flush+Reload, which is widely PREFETCHW is an x86 prefetch instruction introduced in
used in transient execution attacks. 2000. It is now available on all Intel Xeon Scalable processors
Index Terms—cache security, side channel attacks
and recent Core processors (since Broadwell). According to
the technology manual [43], the function of this instruction
I. INTRODUCTION
is to prepare data for future writes. It is different from other
Modern processors often feature many microarchitectural prefetch instructions (e.g., PREFETCHT0) which only move
structures that are shared among applications. Although such the target cache line closer to the CPU core (i.e., to a higher
resource sharing enables significant performance benefits, it cache level) to get ready for future accesses. PREFETCHW
also gives adversaries the potential to build powerful covert moves the cache line to the requesting core’s L1 data cache
channel and side channel attacks. When an application runs (L1D cache), as well as sets the coherence state of the cache
on such hardware, its execution may cause various state line to be Modified. This can accelerate future write operations
changes to these shared microarchitectural structures, which from this requesting core, because a cache line in Modified
can be observed by an attacker on the same platform. Through state indicates that 1) the current private cache has exclusive
repeated observations, the attacker can derive the application’s ownership of this cache line, meaning a write operation on
private information related to the state changes, bypassing this cache line can be directly served by the private cache,
sandboxes and traditional privilege boundaries. Cache timing and 2) this cache line is already marked as dirty, so the flag
covert channel and side channel attacks, or cache attacks (i.e., the dirty bit) does not need to be changed when serving
for short, are extremely potent [1]–[25]. They are especially a write operation. For correctness, setting the coherence state
powerful primitives used in the more recently discovered of a cache line to Modified causes all copies of this cache line
transientexecutionattacks[26]–[36].Differentcachebehaviors, in other cores’ private caches to be invalidated [44], [45].
such as hits and misses create significant timing differences to In this work, we make two important observations regarding
the execution of an instruction. Attackers can use these timing PREFETCHW on Intel processors. First, although its purpose
2202
guA
71
]RC.sc[
3v04321.0112:viXra

CCCooorrreee   000 CCCooorrreee   111 CCCooorrreee   222 CCCooorrreee   000 CCCooorrreee   111 CCCooorrreee   222 CCCooorrreee   000 CCCooorrreee   111 CCCooorrreee   222 CCCooorrreee   000 CCCooorrreee   111 CCCooorrreee   222
|     | RR//WW |     |     |     | RR  |     | RR  | RR//WW |     |     |     |     |     |
| --- | ------ | --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- |
Private Private Private Private Private Private Private Private Private Private Private Private
Cache Cache Cache Cache Cache Cache Cache Cache Cache Cache Cache Cache
|     | (M)odified |           |           |     | (S)hared | (S)hared |           | (E)xclusive | (I)nvalid |           | (I)nvalid |           |           |
| --- | ---------- | --------- | --------- | --- | -------- | -------- | --------- | ----------- | --------- | --------- | --------- | --------- | --------- |
|     |            | (I)nvalid | (I)nvalid |     |          |          | (I)nvalid |             |           | (I)nvalid |           | (I)nvalid | (I)nvalid |
SSSSShhhhhaaaaarrrrreeeeeddddd     LLLLLaaaaasssssttttt-----LLLLLeeeeevvvvveeeeelllll     CCCCCaaaaaccccchhhhheeeee     (((((LLLLLLLLLLCCCCC))))) SSSSShhhhhaaaaarrrrreeeeeddddd     LLLLLaaaaasssssttttt-----LLLLLeeeeevvvvveeeeelllll     CCCCCaaaaaccccchhhhheeeee     (((((LLLLLLLLLLCCCCC))))) SSSSShhhhhaaaaarrrrreeeeeddddd     LLLLLaaaaasssssttttt-----LLLLLeeeeevvvvveeeeelllll     CCCCCaaaaaccccchhhhheeeee     (((((LLLLLLLLLLCCCCC)))))
SSSSShhhhhaaaaarrrrreeeeeddddd     LLLLLaaaaasssssttttt-----LLLLLeeeeevvvvveeeeelllll     CCCCCaaaaaccccchhhhheeeee     (((((LLLLLLLLLLCCCCC)))))
SSttaallee  ddaattaa VVaalliidd  ddaattaa VVaalliidd  ddaattaa VVaalliidd  ddaattaa
|     |     | (a) M |     |     |     | (b) S |     |     | (c) E  |     |     | (d) I |     |
| --- | --- | ----- | --- | --- | --- | ----- | --- | --- | ------ | --- | --- | ----- | --- |
Fig. 1: The four possible states of a private cache line, when using the MESI protocol.
is to accelerate future writes, PREFETCHW works on data cache (LLC) hit. Then, the attacker can determine the victim’s
with read-only permission. Second, the execution time of behavior using timing information: a remote private cache hit
PREFETCHW is related to the current coherence state of the and an LLC hit take different amounts of time to finish. We
targetdata.Withthefirstobservation,anattackeronadifferent show that our attacks can be deployed on Intel processors to
core than the victim can use PREFETCHW on the shared data leak secrets from real-world applications, and that they can be
between the attacker and the victim (which is usually read- used in transient execution attacks, making those attacks faster
only [1]), to evict this data from the victim’s private cache. In (and more potent) than before. To the best of our knowledge,
addition, the second observation means that the attacker can our prefetch-basedattacks arethe first cross-core private cache
timetheexecutionofPREFETCHWontheshareddatabetween side channel attacks that can work with both inclusive and
the attacker and victim to learn the coherence state changes of non-inclusive LLCs: in our attacks, the victim’s data is only
thisdata,whichcouldberelatedtothevictim’scacheaccesses. evicted from the private cache but never the LLC.
Based on these two observations, we first propose two In this paper, we make the following contributions:
covert channel attacks: Prefetch+Load and Prefetch+Prefetch. We discover two severe security vulnerabilities in the
•
In Prefetch+Load, the sender transmits a bit by prefetching implementation of PREFETCHW.
(with PREFETCHW) the shared data between the sender and • We present a new cache eviction method, as well as two
receiver (for “1”) or not prefetching (for “0”). The receiver (on new cross-core cache covert channels and side channels,
adifferentcore)receivesthebitbyloadingthisdataandtiming using PREFETCHW.
the load to determine if it is a local private cache hit (for “0”) We evaluate the proposed prefetch-based covert chan-
•
or a remote private cache hit (for “1”). In Prefetch+Prefetch, nels and side channels on multiple desktop and server
the sender transmits a bit by loading (or not) the shared data, processors. The experimental results show that 1) our
|     |              |     |          |            |             |     |              | covert | channels | are | faster than | most existing | cache |
| --- | ------------ | --- | -------- | ---------- | ----------- | --- | ------------ | ------ | -------- | --- | ----------- | ------------- | ----- |
| and | the receiver |     | receives | the bit by | prefetching |     | the data and |        |          |     |             |               |       |
timing the prefetch instruction to determine whether the sender covert channels and 2) our side channel attacks can leak
loaded. We show that our prefetch-based channels have very information from daily applications with high temporal
| high | capacities: |     | on our Kaby | Lake | processor, | when | only using | resolution. |     |     |     |     |     |
| ---- | ----------- | --- | ----------- | ---- | ---------- | ---- | ---------- | ----------- | --- | --- | --- | --- | --- |
one shared cache line between the sender and receiver, the We have disclosed the security vulnerabilities we found in
capacities are 840KB/s for Prefetch+Load, and 822KB/s for thispapertoIntel.Thesourcecodeofourattackscanbefound
Prefetch+Prefetch, which are the highest single-line capacities at https://github.com/PittECEArch/AdversarialPrefetch.
| among           | all     | existing | CPU                   | cache covert | channels. |              |               |        |                    |     |               |          |     |
| --------------- | ------- | -------- | --------------------- | ------------ | --------- | ------------ | ------------- | ------ | ------------------ | --- | ------------- | -------- | --- |
|                 |         |          |                       |              |           |              |               |        |                    | II. | BACKGROUND    |          |     |
|                 | We then | modify   | the covert            | channel      | attacks   |              | and build the |        |                    |     |               |          |     |
|                 |         |          |                       |              |           |              |               | A. CPU | Cache Architecture |     | and Coherence | Protocol |     |
| Prefetch+Reload |         |          | and Prefetch+Prefetch |              |           | side channel | attacks,      |        |                    |     |               |          |     |
which can be used to leak the victim’s access patterns, similar Cache architecture.MostCPUcachesonmodernx86proces-
to previous cache attacks (e.g., [1]–[7]). Prefetch+Prefetch can sors are divided into L1, L2, and L3. The L1 and L2 caches
bedirectlyusedasasidechannelattackbylettingthevictimbe are very fast but relatively small. Typically, they are organized
the sender, and the attacker be the receiver, since in this attack separately for each CPU core, and are thus often referred to
the sender transmits signals by accessing (or not) the shared as private caches. In contrast, the L3 cache, also known as
data. However, in Prefetch+Load, the sender is sending signals last-level cache (LLC), is a larger but slower cache, shared
by prefetching (or not), which is unlikely a side channel. Thus, among CPU cores.
we modify it and build Prefetch+Reload, where the attacker Caches operate on fixed-size (e.g., 64 bytes) data blocks
owns two threads running on different cores. The attacker first called cache lines. Additionally, caches are usually set-
uses one thread to prefetch and waits for the victim’s possible associative:acacheisorganizedintomultiplecachesets.Every
access, and then reloads using the other thread. When the cache set consists of multiple equivalent cache ways, and each
attacker reloads, he will get a remote private cache hit if the of them can store one cache line. The address bits of a cache
victim accessed this data; otherwise he will get a last level line determine which cache set that this line is mapped to.

Most LLCs in Intel processors are inclusive, meaning that data it (Figure 1(d)).
present in private caches are necessarily also present in the With MESI, a memory request from a CPU core will
LLC(andconversely,datanotintheLLCarenotintheprivate sometimes 1) change the coherence state of the target cache
| caches). | However, | recent | Intel | server | processors |     | (e.g., Intel |           |         |           |         |     |         |         |           |
| -------- | -------- | ------ | ----- | ------ | ---------- | --- | ------------ | --------- | ------- | --------- | ------- | --- | ------- | ------- | --------- |
|          |          |        |       |        |            |     |              | line, and | 2) take | different | amounts | of  | time to | finish, | depending |
Xeon Scalable processors [46], [47]) use non-inclusive LLCs, on the coherence state of the target cache line.
| i.e., cache | lines | in private | caches | may | not | be present | in the |                    |     |       |     |      |           |           |       |
| ----------- | ----- | ---------- | ------ | --- | --- | ---------- | ------ | ------------------ | --- | ----- | --- | ---- | --------- | --------- | ----- |
|             |       |            |        |     |     |            |        | State transitions. |     | There | are | many | different | coherence | state |
LLC. For non-inclusive LLCs, a separate directory structure transitions, we only discuss the two scenarios related to our
is required for tracking the cache lines that are in the private attacks. First, as shown in Figure 2(a), when a CPU core (core
| caches but | not | in the LLC. |     |     |     |     |     |               |     |       |           |            |     |         |         |
| ---------- | --- | ----------- | --- | --- | --- | --- | --- | ------------- | --- | ----- | --------- | ---------- | --- | ------- | ------- |
|            |     |             |     |     |     |     |     | 1) is reading | a   | cache | line that | is present | in  | the LLC | and the |
When a CPU core performs a memory access request, it private cache of another core (core 0) in M state, this read
| first checks | whether | the | target | cache | line is | present | in its L1 |         |            |      |                |     |           |      |            |
| ------------ | ------- | --- | ------ | ----- | ------- | ------- | --------- | ------- | ---------- | ---- | -------------- | --- | --------- | ---- | ---------- |
|              |         |     |        |       |         |         |           | request | will first | miss | in its private |     | cache and | then | search the |
or L2 cache. If present, the request results in a private cache LLC. Although this target cache line can be found in the LLC,
hit; if not, it is a private cache miss and the core must further its content is potentially stale. Thus, the LLC will fetch the
| check the | LLC | (and the | directory | for | a non-inclusive |     | LLC). |           |           |         |       |     |         |               |     |
| --------- | --- | -------- | --------- | --- | --------------- | --- | ----- | --------- | --------- | ------- | ----- | --- | ------- | ------------- | --- |
|           |     |          |           |     |                 |     |       | data from | the owner | private | cache | (in | core 0) | that contains | the |
If the cache line is found, the request finishes and the data is up-to-date data of this cache line, change the coherence state
sent to the CPU. If not, the cache forwards the request to the of this cache line in that private cache (in core 0) to S, update
memory controller, which can read data from DRAM. the content of this cache line in the LLC, and then return the
Cache coherence. In multi-core systems, a cache line can updated cache line to the requesting core (core 1) as well as
| be present | in multiple | private |     | caches, | due | to data | sharing. | A                |     |            |        |     |            |      |             |
| ---------- | ----------- | ------- | --- | ------- | --- | ------- | -------- | ---------------- | --- | ---------- | ------ | --- | ---------- | ---- | ----------- |
|            |             |         |     |         |     |         |          | fill its private |     | cache with | a copy | of  | this cache | line | in S state. |
cache coherence protocol1 is required for maintaining data Thus, after serving this read request, the target cache line is
consistency among the copies of a cache line in different present in two private caches, and is in S state in both caches,
private caches: each private cache line is assigned a coherence as shown in Figure 2(b). This case is usually referred to as
state, and the LLC needs to track this state to prevent the remote private cache hit.
| use of stale  | data. | For inclusive |          | LLCs, | the coherence |         | states       | of     |     |        |     |     |        |     |        |
| ------------- | ----- | ------------- | -------- | ----- | ------------- | ------- | ------------ | ------ | --- | ------ | --- | --- | ------ | --- | ------ |
| private cache | lines | are stored    | together |       | with          | the tag | array in the |        |     |        |     |     |        |     |        |
|               |       |               |          |       |               |         |              | Core 0 |     | Core 1 |     |     | Core 0 |     | Core 1 |
| LLC since     | all   | the private   | cache    | lines | are also      | in the  | LLC. For     |        |     |        |     |     |        |     |        |
non-inclusive LLCs, the directory structure mentioned earlier Private Private Private Private
|                |             |               |         |        |          |       |          | Cache      |     | Cache     |                  |     | Cache    |         | Cache    |
| -------------- | ----------- | ------------- | ------- | ------ | -------- | ----- | -------- | ---------- | --- | --------- | ---------------- | --- | -------- | ------- | -------- |
| is used        | for storing | the coherence |         | states | of cache | lines | that are |            |     |           |                  |     |          |         |          |
|                |             |               |         |        |          |       |          | (M)odified |     | (I)nvalid |                  |     | (S)hared |         | (S)hared |
| in the private |             | caches but    | not the | LLC.   |          |       |          |            |     |           | Read from Core 1 |     |          |         |          |
|                |             |               |         |        |          |       |          |            | 1C  | o r e  1  |                  |     | 1C o     | r e   0 |          |
Most modern x86 processors use variants of the MESI re a d s Write from Core 0 w ri t e s
| coherence     | protocol | [44],        | [45].    | In the         | rest      | of this    | section, we |                         |                                                    |         |          |           |                         |                                                    |          |
| ------------- | -------- | ------------ | -------- | -------------- | --------- | ---------- | ----------- | ----------------------- | -------------------------------------------------- | ------- | -------- | --------- | ----------------------- | -------------------------------------------------- | -------- |
|               |          |              |          |                |           |            |             |                         | SSSSShhhhhaaaaarrrrreeeeeddddd     LLLLLLLLLLCCCCC |         |          |           |                         | SSSSShhhhhaaaaarrrrreeeeeddddd     LLLLLLLLLLCCCCC |          |
|               |          |              |          |                |           |            |             | 2LLC forwards the read  |                                                    |         |          |           | 2LLC sends invalidation |                                                    |          |
| use inclusive | cache    | as an        | example  | to             | introduce | MESI.      | For non-    |                         |                                                    |         |          |           |                         |                                                    |          |
|               |          |              |          |                |           |            |             |                         | SSttaallee  ddaattaa                               |         |          |           |                         | VVaalliidd  ddaattaa                               |          |
| inclusive     | caches,  | the protocol |          | is essentially |           | the same,  | except      |                         |                                                    |         |          |           |                         |                                                    |          |
|               |          |              |          |                |           |            |             |                         | (a)                                                |         |          |           |                         | (b)                                                |          |
| that a cache  | line     | in a private | cache    | might          | not       | be present | in the      |                         |                                                    |         |          |           |                         |                                                    |          |
|               |          |              |          |                |           |            |             | Fig. 2: The             | illustration                                       |         | of cache | coherence | state                   | changes.                                           | The      |
| LLC. With     | MESI,    | there        | are four | possible       |           | states of  | a private   |                         |                                                    |         |          |           |                         |                                                    |          |
|               |          |              |          |                |           |            |             | state of                | a line                                             | changes | from     | M (shown  | in                      | (a)) to                                            | S (shown |
cache line:
|          |     |               |     |           |      |         |         | in (b)) when | a    | CPU core | is loading |     | it; conversely, |     | the state  |
| -------- | --- | ------------- | --- | --------- | ---- | ------- | ------- | ------------ | ---- | -------- | ---------- | --- | --------------- | --- | ---------- |
| Modified |     | (M), in which |     | the cache | line | is only | present |              |      |          |            |     |                 |     |            |
| •        |     |               |     |           |      |         |         | changes      | from | S to M   | when a     | CPU | core is writing |     | it. Dashed |
in one private cache and is dirty, i.e., the copy of this lines shows the request path of the read/write operation.
| cache         | line  | in the LLC | contains       |       | stale      | data (Figure | 1(a)).     |        |     |        |     |     |        |     |        |
| ------------- | ----- | ---------- | -------------- | ----- | ---------- | ------------ | ---------- | ------ | --- | ------ | --- | --- | ------ | --- | ------ |
| Additionally, |       | when       | a private      | cache | line       | is in M      | state, the |        |     |        |     |     |        |     |        |
| current       | owner | core       | has read/write |       | permission |              | for it.    |        |     |        |     |     |        |     |        |
|               |       |            |                |       |            |              |            | Core 0 |     | Core 1 |     |     | Core 0 |     | Core 1 |
| • Shared      | (S),  | in which   | the            | cache | line is    | present      | in one     | or     |     |        |     |     |        |     |        |
more private caches and is clean, i.e., the data of this Private Private Private Private
|        |      |           |           |         |       |          |           | Cache      |     | Cache     |     |     | Cache    |     | Cache     |
| ------ | ---- | --------- | --------- | ------- | ----- | -------- | --------- | ---------- | --- | --------- | --- | --- | -------- | --- | --------- |
| cache  | line | matches   | all other | copies  | (both | in other | private   |            |     |           |     |     |          |     |           |
|        |      |           |           |         |       |          |           | (M)odified |     | (I)nvalid |     |     | (S)hared |     | (I)nvalid |
| caches | and  | the LLC). | The       | current | core  | can only | read this |            |     |           |     |     |          |     |           |
Slower than
| cache       | line        | (Figure        | 1(b)). |           |          |            |            |             |                                                    | Read  |           |        |      |                                                    | Read  |
| ----------- | ----------- | -------------- | ------ | --------- | -------- | ---------- | ---------- | ----------- | -------------------------------------------------- | ----- | --------- | ------ | ---- | -------------------------------------------------- | ----- |
| • Exclusive |             | (E), in which  |        | the cache | line     | is only    | present    |             |                                                    |       |           |        |      |                                                    |       |
|             |             |                |        |           |          |            |            |             | SSSSShhhhhaaaaarrrrreeeeeddddd     LLLLLLLLLLCCCCC |       |           |        |      | SSSSShhhhhaaaaarrrrreeeeeddddd     LLLLLLLLLLCCCCC |       |
| in          | one private | cache,         | and    | is clean  | (Figure  |            | 1(c)). The |             |                                                    |       |           |        |      |                                                    |       |
|             |             |                |        |           |          |            |            |             | SSttaallee  ddaattaa                               |       |           |        |      | VVaalliidd  ddaattaa                               |       |
| current     | core        | can read/write |        | this      | cache    | line;      | however,   | a           |                                                    |       |           |        |      |                                                    |       |
|             |             |                |        |           |          |            |            |             | (a)                                                |       |           |        |      | (b)                                                |       |
| write       | operation   | will           | change | the       | state of | this cache | line       | to          |                                                    |       |           |        |      |                                                    |       |
|             |             |                |        |           |          |            |            | Fig. 3: The | illustration                                       |       | of an LLC | access | with | the target                                         | cache |
M.
|           |         |          |         |       |           |            |          | line in M | state    | (a), and | S state | (b).  |            |      |          |
| --------- | ------- | -------- | ------- | ----- | --------- | ---------- | -------- | --------- | -------- | -------- | ------- | ----- | ---------- | ---- | -------- |
| • Invalid | (I),    | in which | the     | cache | line is   | invalid,   | and thus |           |          |          |         |       |            |      |          |
| the       | current | core has | neither | read  | nor write | permission | for      |           |          |          |         |       |            |      |          |
|           |         |          |         |       |           |            |          | Second,   | as shown | in       | Figure  | 2(b), | when a CPU | core | (core 0) |
istryingtowriteacachelinethatisinSstateinitsownprivate
1Inthispaper,weonlyfocusonthecachecoherenceinsideaprocessor;this
shouldnotbeconfusedwiththecoherenceamongsockets(processors)[37]. cache, this private cache (in core 0) needs to send request to

the LLC to acquire write permission before it can serve this attacks.Theothertypeutilizescachestates:theattackeractively
write operation. As a result, the LLC will send invalidation brings the cache line/cache set to a certain state, then lets
signal(s) to the other private cache(s) that the cache line is the victim execute (which potentially changes the state), and
present in (in core 1), and then change the state of the cache later checks the state again to infer the victim’s behavior. Such
lineintheprivatecacheoftherequestingcore(core0)toMso attacksareoftenreferredtoasevictionbasedattacksorstateful
that the requesting core can write this cache line in its private attacks. In this overview, we focus on stateful attacks because
cache. Thus, after this write operation, the target cache line is they are more numerous, and our proposed attacks are stateful.
only present in the requesting core’s private cache, and is in Wefurtherdividestatefulattacksintoprivatecacheattacksand
M state, as shown in Figure 2(a). LLC attacks, based on whether the attacker evicts the victim’s
Timing difference. As one can observe in Figure 3, if a CPU data from the LLC during the attack.
core is reading a cache line that is not present in its private Private cache attacks. In private cache attacks, the attacker
cache but is present in the LLC, the total latency it takes to learns the victim’s cache behavior by monitoring the state
finish this read request can be different when this cache line of the victim’s data in the private cache. For example, in L1
has different coherence states: a remote private cache hit is Evict+Reload, the attacker evicts the victim’s data (which is
much slower than an LLC hit. When another core has a copy shared with the attacker) from the L1 cache to the L2 cache
of this cache line in M state in its private cache, this request by building set conflicts, and waits for the victim’s execution.
results in a remote private cache hit. As explained earlier, Later the attacker accesses this data and times the access to
serving this request will require fetching data from the owner determineitisintheL1orL2cache:ifitisintheL1cache,it
private cache. In contrast, when all the private cache copies means the victim accessed the data and brought it back to the
of this cache line are in S state, the data of this cache line L1 cache, otherwise the victim did not access. Private cache
in the LLC is up-to-date. This means the LLC can serve this attacks could have high-bandwidth since they do not create
read request directly, resulting in an LLC hit. Due to these slow DRAM accesses. However, most private cache attacks
different execution paths, an LLC hit is much faster than a require the attacker to be on the same physical core with the
remote private cache hit. This has been observed by previous victim (e.g., [5], [11]), and many of them further require SMT.
work [7] and has been verified in our experiments. On an Intel This significantly limits the attacks, as cloud providers may
Core i7-6700 processor, an LLC hit takes less than 60 cycles allocate users to different cores and may disable SMT for
to finish and a remote private cache hit takes about 90 cycles. security [51]–[53].
The directory Prime+Probe attack [47] and its optimization,
B. Prefetch
the directory Prime+Scope attack [38], are an exception: they
Prefetch is a technique to boost performance by fetching
arecross-coreprivatecacheattacks.Onaprocessorwithanon-
data and placing them closer to the CPU core (e.g., from
inclusive LLC, the attacker can “remotely” evict the victim’s
the LLC to L1 cache) before they are needed. Prefetch can
data from the victim’s private cache to the LLC (but not to
be performed in two ways: 1) hardware prefetch, which is
DRAM) by building conflicts in the directory.
implemented in cache hardware and is transparent to users
LLC attacks. In LLC attacks (e.g., [1], [4], [54]), the attacker
(e.g., the adjacent cache line prefetcher); 2) software prefetch,
monitors the state of the victim’s data in the LLC. The LLC is
which needs to be explicitly done by the programmer/com-
usually shared among CPU cores. Thus, different than private
piler. Recent x86 CPUs offer many different instructions
cache attacks, LLC attacks do not require the attacker to be on
for software prefetch, such as PREFETCHT0, PREFETCHT1,
the same core as the victim. These attacks are considered more
PREFETCHT2, PREFETCHNTA, and PREFETCHW.2 These
practical. However, DRAM accesses are usually involved in
instructions are used to hint the processor that a memory
LLC attacks. To monitor the victim’s access on the LLC data,
location is very likely to be accessed soon [43], then the
the attacker needs to first evict the victim’s data from the LLC
processor will prefetch the corresponding data into certain
to memory. For example, in Flush+Reload [1], the attacker
level of cache, thereby accelerating future accesses to this data.
uses CLFLUSH instruction to flush the victim’s data from the
Software prefetch is an important way to improve performance.
LLC (and also the private caches), and later reloads this data
For example, compilers sometimes inject prefetch instructions
and times this operation to determine whether the victim has
to accelerate for loops.
brought this data back to the LLC. Therefore, the bandwidths
C. Cache Side Channel Attacks of LLC attacks are bottlenecked by DRAM latencies.
Therearetypicallytwotypesofcache attacks.Thefirsttype
utilizes the contention on certain cache hardware (e.g., the ring
III. CHARACTERIZINGDATAPREFETCHING
interconnect [48] or L1 cache ports [49], [50]): the attacker
passively monitors the latency of accessing this hardware
Among the prefetch instructions discussed in Section II-B,
resource to infer the victim’s usage of it. Such attacks are
PREFETCHW (or PREFETCHWT1 on some CPU models)
usually referred to as contention based attacks or stateless
works slightly differently than the others. It not only brings
the data close to the CPU core, but also changes the coherence
2Some CPU models (e.g., Intel Xeon Phi Processor 7200) use
PREFETCHWT1insteadofPREFETCHW. state of the data: PREFETCHW places the target data cache

1 void* thread0 (void* addr d0, int expt idx){ 1 void* thread0 (void* addr d0, int expt idx){
2 for(int i = 0; i < 1000000; i++){ 2 for(int i = 0; i < 1000000; i++){
3 /* check the experiment index*/ 3 /* check the experiment index*/
4 if(expt idx == 0){ 4 if(expt idx == 0){
5 /* execute prefetchw on d0*/ 5 read(addr d0);}
6 prefetchw(addr d0);} 6 /*let thread1 execute 1 iteration*/
7 /*let thread1 execute 1 iteration*/ 7 wait for thread1()
8 wait for thread1(); 8 }}
9 }} 9
10 10 void* thread1 (void* addr d0){
11 void* thread1 (void* addr d0){ 11 for(int i = 0; i < 1000000; i++){
12 for(int i = 0; i < 1000000; i++){ 12 /*let thread0 execute 1 iteration*/
13 /*let thread0 execute 1 iteration*/ 13 wait for thread0();
14 wait for thread0(); 14 int t1 = rdtscp(); /* read time stamp*/
15 int result = read and time(addr d0); 15 prefetchw(addr d0);
16 }} 16 int result = rdtscp()−t1;
17 17 }}
18 18
19 int main() { 19 int main() {
20 /* open and map a file as read−only*/ 20 /* open and map a file as read−only*/
21 int fd = open(FILE NAME, ORDONLY); 21 int fd = open(FILE NAME, ORDONLY);
22 int* addr d0 = mmap(fd, PROT READ, ...); 22 int* addr d0 = mmap(fd, PROT READ, ...);
23 23
24 /*pin thread0 on core0 and start thread0*/ 24 /*pin thread0 on core0 and start thread0*/
25 /*pin thread1 on core1 and start thread1*/ 25 /*pin thread1 on core1 and start thread1*/
26 ... 26 ...
Listing 1: The code snippet for verifying Observation 1. Listing 2: The code snippet for verifying Observation 2.
line into the L1D cache3 and sets the coherence state of this iteration, they both move to the next iteration and repeat this
cache line to M. According to the technology manual [43], procedure again. We use pthread mutex locking [57] to ensure
[55], the role of PREFETCHW is to accelerate future writes that in each iteration thread 0 and thread 1 execute sequentially
on the target cache line. As explained in Section II-A, the (the implementation details of locking is omitted in Listing 1).
CPU core can directly write a cache line in its local L1 cache We run the code in Listing 1 twice: in the first experiment
iff the state of this cache line is E/M. Thus, PREFETCHW (i.e., expt idx = 0 in line 3), in each iteration of the for loop,
pre-sets the coherence state of the target cache line to M so thread 0 performs PREFETCHW on d 0 , and then thread 1 loads
that future writes on this cache line will likely have an L1 d 0 as well as times the load. In the second experiment (i.e.,
hit. PREFETCHW sets the cache line state to M instead of E expt idx = 1 in line 3), in each iterationthread 0 stays idle and
because writing a cache line in E state results in changing the then thread 1 still loads d 0 and times the load.
state to M, and thus has higher latency than writing a cache
line that is already in M state.
100
MostoftherecentInteldesktopandserverprocessors(since
80
Broadwell) support PREFETCHW. When used appropriately, it
can significantly improve performance. However, we make two 60
observations about PREFETCHW on Intel processors, which
40
can be leveraged to create security vulnerabilities.
20
Observation 1 PREFETCHW successfully executes on data 0
1000 1020 1040 1060 1080 1100
with read-only permission.
We observe this by monitoring the coherence state changes
of the data, using timing information. Specifically, as shown
in Listing 1, we run a program with two threads (thread
0
and thread , both in one process), and pin them on different
1
physical cores. We use mmap [56] to map part of a system file
(e.g., glibc) as a read-only data block (in cache line size) in
this program and name it d : both threads can only read d . If
0 0
any thread tries to write d , it will trigger a segmentation fault.
0
thread and thread both consist of a for loop with the same
0 1
amount of iterations. In each iteration, thread first executes,
0
then waits for thread to execute. After thread finishes this
1 1
3PREFETCHW can only be used on data but not instructions [43], [55].
Thus,thecachelinewillbebroughtintoL1Dcache.
)selcyc(
ycnetaL
160
120
Experiment 0
Experiment 1
80
40
Listing 1
0
1000 1020 1040 1060 1080 1100
Iteration ID
)selcyc(
ycnetaL
Experiment 0
Experiment 1
Listing 2
Iteration ID
Fig. 4: The timing measurement results inthread of Listing 1
1
and Listing 2.
Figure4showsasegmentofthetimingresultsfromthread
1
(in line 15) in both of the above experiments on an Intel Core
i7-6700processor.Notethatweobservesimilarresultsonother
Intel processors that support PREFETCHW. In experiment 0,
thread prefetchesd ineachiteration,whichcausesthread to
0 0 1
take around 90 cycles to load d after the prefetch. In contrast,
0
in experiment 1, thread stays idle, which causes thread to
0 1
take only around 30 cycles to load d . This timing difference
0
infersthatd isindifferentstatesintheabovetwoexperiments.
0
In experiment 0, every time when thread prefetches, it will 0

load d to its private cache and set the coherence state of it M state. Thus, in this case PREFETCHW does not cause any
0
to be M. According to MESI, explained in Section II-A, this state change and finishes much faster.
will invalidate the copy of d in the private cache of thread Affected processors. We have tested these two observations
|     |     |     | 0   |     |     |     |     | 1   |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
(if it exists). Therefore, when thread later loads d , it will on many Intel processors including the available 1st/2nd/3rd
|     |     |     |     |     | 1   | 0   |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
have a remote private cache hit (see Figure 2). This load also Generation Intel Xeon Salable Processors on AWS EC2, and
changesthestateofd fromMtoSandfillsacopyofitinthe five Intel desktop/server processors we own. As shown in
0
private cache of thread 1 . Thus, the same cache behavior (i.e., Table I, Observation 1 is valid on all the tested processors, and
invalidatingthecopyofd intheprivatecacheofthread )will Observation 2 is valid on most, excluding the Intel Xeon
|     |     |     | 0   |     |     |     | 1   |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
happenwhenthread prefetchesin thenextiteration. However, Platinum 8375C processor. On this processor, there is no
0
inexperiment1,sincethread 0 isnotprefetching,whenthread 1 difference on the execution time of PREFETCHW when the
loadsd ,itwillverylikelyhavealocalprivatecachehit,which target data is different coherence states: PREFETCHW always
0
cycles4
is much faster than a remote private cache hit (30 vs. takes 70 to 80 cycles to finish, even when the target data is
| 90 cycles  | on the     | tested | processor). |       |       |         |          | not already | in M state. |      |             |     |             |          |
| ---------- | ---------- | ------ | ----------- | ----- | ----- | ------- | -------- | ----------- | ----------- | ---- | ----------- | --- | ----------- | -------- |
|            |            |        |             |       |       |         |          | In general, | we believe  | that | Observation |     | 1 should be | valid on |
| Rationale. | We observe |        | reliable    | cache | state | changes | on read- |             |             |      |             |     |             |          |
only data when executing PREFETCHW, with a F-score of 1.0 allIntelprocessorsthatsupportPREFETCHW,andObservation
(n=1000000).ThisindicatesthatIntelprocessorsveryunlikely 2 should be valid on most of them. Note that all 1st/2nd/3rd
|     |     |     |     |     |     |     |     | Generation | Intel Xeon | Scalable |     | processors | and most | Intel |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | ---------- | -------- | --- | ---------- | -------- | ----- |
performawritepermissioncheckwhenexecutingPREFETCHW.
This does not cause any error in the architecture level, because Core i7/i9 processors (other than the early generations before
PREFETCHW only has microarchitectural effects: although it Broadwell) support PREFETCHW.
cangetacachelinereadyforfuturewrites,iflatertheprogram
| without   | write permission |               | for | this cache | line       | actually  | tries | to    |                  |            |     |         |                   |     |
| --------- | ---------------- | ------------- | --- | ---------- | ---------- | --------- | ----- | ----- | ---------------- | ---------- | --- | ------- | ----------------- | --- |
|           |                  |               |     |            |            |           |       | TABLE | I: The evaluated | processors |     | for the | two observations. |     |
| write it, | it will          | still trigger | a   | fault      | and likely | terminate | the   |       |                  |            |     |         |                   |     |
process. However, later we will show that allowing coherence- Processor Microarch. LLCType Observ.1 Observ.2
based cache invalidation (which should only happen upon IntelCorei7-6700 Skylake Inclusive (cid:51) (cid:51)
|                 |           |       |         |             |           |                    |     | IntelCorei7-6800K  |     |             | Skylake  | Inclusive |     | (cid:51) (cid:51) |
| --------------- | --------- | ----- | ------- | ----------- | --------- | ------------------ | --- | ------------------ | --- | ----------- | -------- | --------- | --- | ----------------- |
| writes) on      | read-only | data  | cause   | significant |           | security problems. |     |                    |     |             |          |           |     |                   |
|                 |           |       |         |             |           |                    |     | IntelCorei7-7700K  |     |             | KabyLake | Inclusive |     | (cid:51) (cid:51) |
| This is because | in        | cache | attacks | based       | on shared | memory,            | the |                    |     |             |          |           |     |                   |
|                 |           |       |         |             |           |                    |     | IntelCorei9-10900X |     | CascadeLake |          | Non-incl. |     | (cid:51) (cid:51) |
attacker can manipulate the coherence state of the shared data (cid:51) (cid:51)
|     |     |     |     |     |     |     |     | IntelXeonSilver4114 |     |     | Skylake-SP | Non-incl. |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------- | --- | --- | ---------- | --------- | --- | --- |
(which is usually read-only) between the victim and attacker (cid:51) (cid:51)
|            |              |     |        |         |       |     |     | IntelXeonPlatinum8151  |     |     | Skylake-SP | Non-incl. |     |                   |
| ---------- | ------------ | --- | ------ | ------- | ----- | --- | --- | ---------------------- | --- | --- | ---------- | --------- | --- | ----------------- |
| to monitor | the victim’s |     | access | to this | data. |     |     |                        |     |     |            |           |     |                   |
|            |              |     |        |         |       |     |     | IntelXeonPlatinum8124M |     |     | Skylake-SP | Non-incl. |     | (cid:51) (cid:51) |
|            |              |     |        |         |       |     |     | IntelXeonPlatinum8175M |     |     | Skylake-SP | Non-incl. |     | (cid:51) (cid:51) |
Observation 2 The execution time of PREFETCHW is related IntelXeonPlatinum8259CL Skylake-SP Non-incl. (cid:51) (cid:51)
to the coherence state of the target cache line. IntelXeonPlatinum8275CL Skylake-SP Non-incl. (cid:51) (cid:51)
|            |             |        |             |           |          |            |       |                        |     |     |         |           |     | (cid:51) (cid:55) |
| ---------- | ----------- | ------ | ----------- | --------- | -------- | ---------- | ----- | ---------------------- | --- | --- | ------- | --------- | --- | ----------------- |
|            |             |        |             |           |          |            |       | IntelXeonPlatinum8375C |     |     | IceLake | Non-incl. |     |                   |
| We observe | this        | with   | the program |           | shown    | in Listing | 2. We |                        |     |     |         |           |     |                   |
| still use  | two threads | pinned | on          | different | physical | cores,     | and   |                        |     |     |         |           |     |                   |
IV. PREFETCH-BASEDCOVERTCHANNELATTACKS
| let them | execute | sequentially |     | in each | iteration | of the | for loop. |     |     |     |     |     |     |     |
| -------- | ------- | ------------ | --- | ------- | --------- | ------ | --------- | --- | --- | --- | --- | --- | --- | --- |
Again, we run the program twice: in experiment 0 (expt idx BasedontheobservationsinSectionIII,webuildtwocache
= 0 in line 3), in each iteration, thread 0 loads d 0 , and then covert channel attacks: Prefetch+Load and Prefetch+Prefetch.
thread performs PREFETCHW on d and times the prefetch. Inthissection,wefirstintroducethethreatmodel,thendiscuss
| 1             |              |     |         |                | 0   |                  |     |             |                 |     |     |     |     |     |
| ------------- | ------------ | --- | ------- | -------------- | --- | ---------------- | --- | ----------- | --------------- | --- | --- | --- | --- | --- |
| In experiment | 1,thread     |     | stays   | idle andthread |     | still prefetches |     |             |                 |     |     |     |     |     |
|               |              |     | 0       |                |     | 1                |     | the details | of each attack. |     |     |     |     |     |
| and times     | the prefetch |     | in each | iteration.     |     |                  |     |             |                 |     |     |     |     |     |
|               |              |     |         |                |     |                  |     | A. Threat   | Model           |     |     |     |     |     |
Figure4showstheexecutiontimeofPREFETCHWobserved
|           |          |     |        |            |         |           |     | We assume | that | the two | essential | parties | in the | attack, |
| --------- | -------- | --- | ------ | ---------- | ------- | --------- | --- | --------- | ---- | ------- | --------- | ------- | ------ | ------- |
| by thread | (in line | 16) | on our | Intel Core | i7-6700 | processor |     | in        |      |         |           |         |        |         |
1
bothexperiments.Inthefirstexperiment,italwaystakesaround the sender and receiver, are two unprivileged processes that
|            |               |       |        |         |          |         |         | are running | on the same  | processor |        | with       | multiple CPU | cores.    |
| ---------- | ------------- | ----- | ------ | ------- | -------- | ------- | ------- | ----------- | ------------ | --------- | ------ | ---------- | ------------ | --------- |
| 130 cycles | for PREFETCHW |       | to     | finish; | however, | in the  | second  |             |              |           |        |            |              |           |
|            |               |       |        |         |          |         |         | The sender  | and receiver | can       | launch | themselves | on           | different |
| experiment | it only       | takes | around | 70      | cycles.  | This is | because |             |              |           |        |            |              |           |
in the first experiment, after thread loads d , the state of d physical cores (e.g., using taskset [58]). We also assume
|         |        |        |      | 0         |      | 0           |       | 0        |            |          |     |       |                |     |
| ------- | ------ | ------ | ---- | --------- | ---- | ----------- | ----- | -------- | ---------- | -------- | --- | ----- | -------------- | --- |
|         |        |        |      |           |      |             |       | that the | sender and | receiver | can | share | data; however, | the |
| becomes | S, and | a copy | of d | is filled | into | the private | cache |          |            |          |     |       |                |     |
0
of thread (see Figure 2). Then when thread prefetches, it shared data can be read-only (e.g., via shared library or page
|           | 0       |               |      |      |          | 1        |         |                  |            |     |          |         |           |          |
| --------- | ------- | ------------- | ---- | ---- | -------- | -------- | ------- | ---------------- | ---------- | --- | -------- | ------- | --------- | -------- |
|           |         |               |      |      |          |          |         | deduplication),5 | similar    | to  | previous | attacks | [1], [2], | [5]–[7]. |
| needs to  | change  | the state     | from | S to | M, which | means    | it has  |                  |            |     |          |         |           |          |
|           |         |               |      |      |          |          |         | In addition,     | the sender | and | receiver | should  | agree     | on pre-  |
| to inform | the LLC | to invalidate |      | the  | copy of  | d in the | private |                  |            |     |          |         |           |          |
0
cache of thread . However, in the second experiment, since defined channel protocols, including the synchronization, core
0
|          |             |      |        |               |     |                |     | allocation, | data encoding, |     | and error | correction | protocols. | We  |
| -------- | ----------- | ---- | ------ | ------------- | --- | -------------- | --- | ----------- | -------------- | --- | --------- | ---------- | ---------- | --- |
| thread 0 | stays idle, | when | thread | 1 prefetches, |     | d 0 is already |     | in          |                |     |           |            |            |     |
5Pagededuplication(a.k.akernelsame-pagemerging[59])wasoriginally
4Duetothegranularityoftimestampcounters,thismeasuredlatencyisin createdforvirtualenvironmentsbutisnowincludedinOSs.Althoughmany
factlongerthantherealprivatecacheaccesslatency. cloudprovidersnolongeruseit,itisusuallystillavailableinOSs[6].

Algorithm 1: Prefetch+Load Covert Channel Algorithm 2: Prefetch+Prefetch Covert Channel
line0:thesharedcachelinebetweenthesenderandreceiver line0:thesharedcachelinebetweenthesenderandreceiver
message[n]:then-bitlongmessagetotransferonthechannel message[n]:then-bitlongmessagetotransferonthechannel
Th0:thetimingthresholdfordistinguishinglocalandremoteprivatecachehit Th0:thetimingthresholdonPREFETCHWtodistinguishMandSstates
| ————————————————————————————– |                |     |     |     |     |     |     | ————————————————————————————– |            |     |     |     |     |     |     |
| ----------------------------- | -------------- | --- | --- | --- | --- | --- | --- | ----------------------------- | ---------- | --- | --- | --- | --- | --- | --- |
| SenderAlgorithm               |                |     |     |     |     |     |     | SenderAlgorithm               |            |     |     |     |     |     |     |
| ————————————————————————————– |                |     |     |     |     |     |     | ————————————————————————————– |            |     |     |     |     |     |     |
| //Send1bitineachiteration.    |                |     |     |     |     |     |     | //Send1bitineachiteration.    |            |     |     |     |     |     |     |
| fori=0;i<n;i++do              |                |     |     |     |     |     |     | fori=0;i<n;i++do              |            |     |     |     |     |     |     |
| sync_with_receiver();         |                |     |     |     |     |     |     | sync_with_receiver();         |            |     |     |     |     |     |     |
| ifmessage[i]==1then           |                |     |     |     |     |     |     | ifmessage[i]==1then           |            |     |     |     |     |     |     |
|                               | Prefetchline0; |     |     |     |     |     |     |                               | Loadline0; |     |     |     |     |     |     |
| else                          |                |     |     |     |     |     |     | else                          |            |     |     |     |     |     |     |
|                               | Donotprefetch; |     |     |     |     |     |     |                               | Donotload; |     |     |     |     |     |     |
| ————————————————————————————– |                |     |     |     |     |     |     | ————————————————————————————– |            |     |     |     |     |     |     |
ReceiverAlgorithm
ReceiverAlgorithm
| ————————————————————————————– |     |     |     |     |     |     |     | ————————————————————————————– |     |     |     |     |     |     |     |
| ----------------------------- | --- | --- | --- | --- | --- | --- | --- | ----------------------------- | --- | --- | --- | --- | --- | --- | --- |
| //Detect1bitineachiteration.  |     |     |     |     |     |     |     | //Detect1bitineachiteration.  |     |     |     |     |     |     |     |
| fori=0;i<n;i++do              |     |     |     |     |     |     |     | fori=0;i<n;i++do              |     |     |     |     |     |     |     |
| sync_with_sender();           |     |     |     |     |     |     |     | sync_with_sender();           |     |     |     |     |     |     |     |
Accessline0andtimetheaccess; Prefetchline0andtimetheprefetch;
|          |                  |     |     |     |     |     |     | ifprefetch |                  | time>Th0then |     |     |     |     |     |
| -------- | ---------------- | --- | --- | --- | --- | --- | --- | ---------- | ---------------- | ------------ | --- | --- | --- | --- | --- |
| ifaccess | time>Th0then     |     |     |     |     |     |     |            |                  |              |     |     |     |     |     |
|          | Receivedabit“1”; |     |     |     |     |     |     |            | Receivedabit“1”; |              |     |     |     |     |     |
else
else
|     | Receivedabit“0”; |     |     |     |     |     |     |     | Receivedabit“0”; |     |     |     |     |     |     |
| --- | ---------------- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- |
do not have requirement on the LLC inclusivity; our attacks Thus, this attack can be directly applied as a side channel
work with both inclusive and non-inclusive LLCs. We also do attack to leak a victim’s access pattern on the shared data:
not require SMT; SMT can be turned off for security. the victim is the sender, and the attacker is the receiver. This
|                  |     |        |     |     |     |     |     | leakage  | (the victim’s |         | access | pattern)  | is same    | as the | one in |
| ---------------- | --- | ------ | --- | --- | --- | --- | --- | -------- | ------------- | ------- | ------ | --------- | ---------- | ------ | ------ |
| B. Prefetch+Load |     | Attack |     |     |     |     |     |          |               |         |        |           |            |        |        |
|                  |     |        |     |     |     |     |     | previous | cache         | attacks | (e.g., | [1], [2], | [7], [8]). |        |        |
We build the first covert channel attack, Prefetch+Load, However, Prefetch+Load cannot be directly used as a side
| following | Observation | 1. Algorithm |     | 1   | shows | the details | of it. |          |         |     |        |                 |     |            |     |
| --------- | ----------- | ------------ | --- | --- | ----- | ----------- | ------ | -------- | ------- | --- | ------ | --------------- | --- | ---------- | --- |
|           |             |              |     |     |       |             |        | channel, | because | the | sender | is transmitting |     | the signal | by  |
In this attack, the sender and receiver first agree on the shared “prefetching (or not) the shared data”. In other words, the
cache line used to transmit information. Then in each iteration attacker (receiver) can only detect the victim’s (sender’s)
of the attack, the sender transmits a bit “1” by performing prefetch patterns on the shared data. Since software prefetch is
PREFETCHW on the shared cache line, or a bit “0” by idling. not as common as memory accesses in real-world applications,
| The receiver | loads | the same | cache | line | and times | the load | to  |            |               |     |              |          |     |     |        |
| ------------ | ----- | -------- | ----- | ---- | --------- | -------- | --- | ---------- | ------------- | --- | ------------ | -------- | --- | --- | ------ |
|              |       |          |       |      |           |          |     | the attack | opportunities |     | are limited. | However, | we  | can | modify |
determine if it is a remote private cache hit or local private the attack slightly to make it work more generally.
cache hit: the receiver receives a bit “1” when having a remote We term the new attack Prefetch+Reload. The attacker
| private cache | hit, and | otherwise |     | receives | a bit | “0”. |     |            |     |        |         |         |               |        |     |
| ------------- | -------- | --------- | --- | -------- | ----- | ---- | --- | ---------- | --- | ------ | ------- | ------- | ------------- | ------ | --- |
|               |          |           |     |          |       |      |     | prefetches | the | shared | data to | pre-set | the coherence | state, | and |
Note that different than the experiments in Section III, the then waits for the victim to possibly access this data. Later the
sender and receiver cannot synchronize using pthread mutex attackerreloadsthedata(usingadifferentthreadonadifferent
locking, since they do not belong to the same process. Thus, core, explained later) and uses the timing information to learn
we let the sender and receiver synchronize the transmission the current coherence state of the data, which leaks whether
| using time | stamp | counters | (TSCs), | as  | done | in prior | covert |            |     |        |           |       |          |               |     |
| ---------- | ----- | -------- | ------- | --- | ---- | -------- | ------ | ---------- | --- | ------ | --------- | ----- | -------- | ------------- | --- |
|            |       |          |         |     |      |          |        | the victim | has | loaded | this data | (thus | changing | the coherence |     |
channel attacks (e.g., [1], [2], [5], [7], [38]). state). Different than Prefetch+Load, in Prefetch+Reload, the
|                      |     |        |     |     |     |     |     | attacker | needs  | to have   | two threads | running | on different |     | cores.  |
| -------------------- | --- | ------ | --- | --- | --- | --- | --- | -------- | ------ | --------- | ----------- | ------- | ------------ | --- | ------- |
| C. Prefetch+Prefetch |     | Attack |     |     |     |     |     |          |        |           |             |         |              |     |         |
|                      |     |        |     |     |     |     |     | Threat   | model. | We assume | a           | similar | threat model | as  | the one |
Our second attack, Prefetch+Prefetch, is based on Obser- for the covert channels. First, the attacker is an unprivileged
| vation 2.   | As shown | in Algorithm |     | 2, in      | each | iteration | of the |         |          |        |        |                |      |     |        |
| ----------- | -------- | ------------ | --- | ---------- | ---- | --------- | ------ | ------- | -------- | ------ | ------ | -------------- | ---- | --- | ------ |
|             |          |              |     |            |      |           |        | process | that can | 1) run | on the | same processor | with | the | victim |
| attack, the | sender   | transmits    | “1” | by loading | the  | shared    | cache  |         |          |        |        |                |      |     |        |
and2)sharedatawiththevictim(e.g.,throughasharedlibrary).
line,ortransmits“0”byidling.Afterthis,thereceiverperforms The attacker aims at leaking the victim’s access pattern on a
| PREFETCHW         | on the   | shared   | cache    | line            | and times  | the prefetch |        |                      |              |          |              |                   |                        |           |         |
| ----------------- | -------- | -------- | -------- | --------------- | ---------- | ------------ | ------ | -------------------- | ------------ | -------- | ------------ | ----------------- | ---------------------- | --------- | ------- |
|                   |          |          |          |                 |            |              |        | shared data          | block,       | as       | in [1],      | [2]. In addition, | the                    | attacker  | can     |
| to decode         | the bit: | when the | sender   | sends           | “1”,       | it takes     | longer |                      |              |          |              |                   |                        |           |         |
|                   |          |          |          |                 |            |              |        | launch his           | thread(s)    | on       | different    | core(s)           | than the               | victim’s. |         |
| for the receiver  | to       | prefetch | than     | when            | the sender | sends        | “0”.   |                      |              |          |              |                   |                        |           |         |
|                   |          |          |          |                 |            |              |        | For Prefetch+Reload, |              |          | the attacker | needs             | to have                | two       | threads |
| Prefetch+Prefetch |          | follows  | the same | synchronization |            | method       |        |                      |              |          |              |                   |                        |           |         |
|                   |          |          |          |                 |            |              |        | running              | on different | physical |              | cores; but        | for Prefetch+Prefetch, |           |         |
with Prefetch+Load. there is still only one thread required in the attacker’s process,
V. PREFETCH-BASEDSIDECHANNELATTACKS which is the same setup as the covert channel attacks.
| A. Basic | Idea and | Assumptions |     |     |     |     |     | B. Prefetch+Reload |     | Attack |     |     |     |     |     |
| -------- | -------- | ----------- | --- | --- | --- | --- | --- | ------------------ | --- | ------ | --- | --- | --- | --- | --- |
In the Prefetch+Prefetch covert channel attack, the sender In this attack, we assume that the attacker controls two
is sending the signal by “accessing (or not) the shared data”. threads named Trojan and Spy. Trojan and Spy should be

1 Trojan prefetches the left path of Figure 5). However, Spy is able to distinguish
whether the victim accessed this cache line. Trojan’s original
PREFETCHWinvalidatedthecopyinSpy’sprivatecache.Thus,
Trojan Spy Victim if Spy now accesses this cache line, he will get an LLC hit if
Private Private Private
the victim has accessed this cache line after Trojan’s prefetch
Cache Cache Cache
(M)odified (I)nvalid (I)nvalid (Step 3 in the left path of Figure 5); otherwise, he will get a
remote private cache hit (Step 3 in the right path of Figure 5).
We recall that Spy can distinguish these two situations by
SShhaarreedd LLLLCC
timing the access (the difference is over 30 cycles on our
Stale data
desktop processor). Based on this, we build Prefetch+Reload.
Similar to previous cache attacks, each iteration in this attack
2 Yes 2 No (No state changes)
Victim contains three steps, as shown in Figure 5:
accesses
Step 1: Trojan performs PREFETCHW on the target cache line
and becomes the exclusive owner of this cache line.
Trojan Spy Victim Trojan Spy Victim Step 2: The attacker waits for the victim’s behavior: if the
Private Private Private Private Private Private victim accesses this cache line, its coherence state will
Cache Cache Cache Cache Cache Cache
(S)hared (I)nvalid (S)hared (M)odified (I)nvalid (I)nvalid become S, meaning the copy in the LLC is now valid.
Step 3: Spy accesses this cache line and times the access to
determine it was a remote private cache hit or an LLC
SShhaarreedd LLLLCC SShhaarreedd LLLLCC hit.Ifitwasaremoteprivatecachehit,thenthevictim
Valid data Stale data did not access this cache line; otherwise the victim
3 Spy accesses: 3 Spy accesses: did access.
LLC hit Remote private cache hit
LLCpresence.Prefetch+Reloadrequiresthatthetargetshared
Trojan Spy Victim Trojan Spy Victim cache line is present in the LLC, so that Spy can get an LLC
Private Private Private Private Private Private hit in Step 3, if the victim has accessed this cache line. This is
Cache Cache Cache Cache Cache Cache
naturally true for inclusive LLCs, since all the cache lines in
(S)hared (S)hared (S)hared (S)hared (S)hared (I)nvalid
theprivatecachearealsopresentintheLLC.However,itisnot
guaranteed for non-inclusive LLCs. In those caches, a cache
SShhaarreedd LLLLCC SShhaarreedd LLLLCC lineisdirectlybroughtintotheprivatecachewhenloadedfrom
Valid data Valid data DRAM, bypassing the LLC; it usually goes to the LLC when
evicted from the private cache due to cache replacement [46],
Fig. 5: The details of the three steps in Prefetch+Reload. [47]. Thus, strictly speaking, it is the attacker’s responsibility
to ensure the presence of this cache line in the LLC, if it is
non-inclusive.Forexample,beforetheattackerstartstheattack
located on different cores, which are also both different than loop,hecanfirstbuildsetconflictsinhisprivatecachetoevict
the victim’s core, i.e., Trojan, Spy, and the victim all run on this cache line to the LLC.
different cores. As mentioned in Section II-A, the execution In fact, empirically we found that in Step 1, when
timesofaremoteprivatecachehitandanLLChitaredifferent. PREFETCHW invalidates the copies of the target cache line
The Prefetch+Reload attacker uses this timing difference to in Spy and the victim’s private caches, this cache line will
observe cache state changes caused by the victim’s accesses. be placed in the LLC if it does not already exist. Therefore,
Specifically, before the victim accesses the target shared cache in practice the attacker does not need to explicitly place this
line, Trojan executes PREFETCHW on this cache line, which cache line in the LLC.
invalidates the copies of this cache line in the victim’s and
Spy’s private caches (if they exist), and places a copy of this C. Prefetch+Prefetch Attack
cache line (in M state) in Trojan’s private cache, as shown
in Step 1 of Figure 5. Then, if the victim accesses this cache Following the Prefetch+Prefetch covert channel attack, we
line, according to MESI, the coherence state changes from M also build the Prefetch+Prefetch side channel attack. The
to S, and the copy of this cache line in the LLC is updated attacker learns if the victim accessed the shared cache line by
(although the content did not change, see Section II-A) and is timing PREFETCHW. In contrast to the Prefetch+Reload side
now valid (Step 2 in the left path of Figure 5). channel attack, each iteration in this attack only has two steps:
Unfortunately,Trojancannotobservethisstatechangecaused Step 1: The attacker prefetches the target shared cache line
by the victim’s access: if Trojan accesses (reloads) this cache using PREFETCHW, and times this operation to learn
line, he will get a private cache hit, no matter if the victim whether the victim accessed this cache line in the last
accessedthislineornot.Thisisbecausethevictim’sreaddoes iteration.
not invalidate the copy in Trojan’s private cache (Step 2 in Step 2: The attacker waits for the victim’s behavior.

1 0 0 0
P re fe tc h + R e lo a d c a p a .
P re fe tc h + L o a d c a p a .
P re fe tc h + P re fe tc h c a p a .
P re fe tc h + R e lo a d e rro r ra te
8 0 0 P re fe tc h + L o a d e rro r ra te
P re fe tc h + P re fe tc h e rro r ra te
6 0 0
4 0 0
2 0 0
2 0 0 4 0 0 6 0 0 8 0 0 1 0 0 0
R a w T r a n s m i s s i o n R a t e ( K B / s )
)
s
/
B
K (
y
t i
c
a
p
a
C l
e
n
n
a
h
C
P re fe tc h + R e lo a d c a p a . P re fe tc h + L o a d c a p a . P re fe tc h + P re fe tc h c a p a . P re fe tc h + R e lo a d e rro r ra te P re fe tc h + L o a d e rro r ra te P re fe tc h + P re fe tc h e rro r ra te
3 0
2 0
1 0
0
)
% (
e
t
a
R
r
o
r
r
E t
i
B
Prefetch+R eload capa. Prefetch+R eload error rate
Prefetch+Prefetch capa. Prefetch+Prefetch error rate
Prefetch+Load capa. Prefetch+Load error rate
1 0 0 0
8 0 0
6 0 0
4 0 0
2 0 0
2 0 0 4 0 0 6 0 0 8 0 0 1 0 0 0
R a w T ra n s m is s io n R a te (K B /s )
)
s/
B K(
yti
c
a p a
C l e n
n a
h
C
3 0
2 0
1 0
0
) %(
et
a
R r orr
E ti
B
Prefetch+R eload capa. Prefetch+R eload error rate
Prefetch+Prefetch capa. Prefetch+Prefetch error rate
Prefetch+Load capa. Prefetch+Load error rate
1 0 0 0
8 0 0
6 0 0
4 0 0
2 0 0
2 0 0 4 0 0 6 0 0 8 0 0 1 0 0 0
R a w T ra n s m is s io n R a te (K B /s )
(a) IntelCorei7-6700
)
s/
B K(
yti
c
a p a
C l e n
n a
h
C
3 0
2 0
1 0
0
) %(
et
a
R r orr
E ti
B
Prefetch+R eload capa. Prefetch+R eload error rate
Prefetch+Prefetch capa. Prefetch+Prefetch error rate
Prefetch+Load capa. Prefetch+Load error rate
1 0 0 0
8 0 0
6 0 0
4 0 0
2 0 0
2 0 0 4 0 0 6 0 0 8 0 0 1 0 0 0
R a w T ra n s m is s io n R a te (K B /s )
(b) IntelCorei7-7700K
)
s/
B K(
yti
c
a p a
C l e n
n a
h
C
3 0
2 0
1 0
0
) %(
et
a
R r orr
E ti
B
Prefetch+R eload capa. Prefetch+R eload error rate
Prefetch+Prefetch capa. Prefetch+Prefetch error rate
Prefetch+Load capa. Prefetch+Load error rate
1 0 0 0
8 0 0
6 0 0
4 0 0
2 0 0
2 0 0 4 0 0 6 0 0 8 0 0 1 0 0 0
R a w T ra n s m is s io n R a te (K B /s )
(c) IntelXeonPlatinum8124M
)
s/
B K(
yti
c
a p a
C l e n
n a
h
C
3 0
2 0
1 0
0
) %(
et
a
R r orr
E ti
B
(d) IntelXeonPlatinum8151
Fig. 6: The capacities and bit-error-rates of the prefetch-based channels on various Intel processors.
As explained earlier in Section III, in Step 1 above, if the them since we aim at showing the conservative results (i.e.,
victim accessed this cache line, PREFETCHW executes slower; the lower bounds).
if the victim did not access, PREFETCHW executes faster. We measure the channel capacity and bit error rate of each
In contrast to most previous cross-core cache attacks, which attack, under different transmission intervals. Although the raw
can only work on the LLC and require repeatedly evicting the transmission rate increases when decreasing the transmission
target cache line to DRAM (e.g., Flush+Reload), the proposed interval, the bit error rate may also increase, especially when
prefetch-based attacks work on the private cache. Thus, the the interval is too short. To find the best transmission rate,
target cache line is always kept in the on-chip cache hierarchy. we use the channel capacity metric (as in [48], [61]). This
Compared to cross-core LLC attacks, cross-core private cache metric is computed by multiplying the raw transmission rate
attacks have two benefits. First, higher bandwidth, since cache with 1−H(e), where e is the bit error rate and H is the
accesses are fast and are usually much faster than DRAM binary entropy function. The results are shown in Figure 6.
accesses.Thisisespeciallyimportantwhentheattacksareused The bit error rates of all three attacks stay low (lower than
ascovertchannels.Second,stealthier,sincetherearelesscache 0.6%) and are almost constant, when the raw transmission rate
misses, especially LLC misses involved in the attacks [60]. To is under a threshold (e.g., 660 KB/s for Prefetch+Reload in
the best of our knowledge, the proposed prefetch-based attacks Figure6(a)).Thus,thechannelcapacityincreasesproportionally
are the first cross-core private cache side channel attacks that to the raw transmission rate. It reaches the peak when the raw
can work regardless of the LLC inclusivity. transmissionrateisaroundthisthreshold.Beyondthisthreshold,
the increasing error rate causes a decrease in the channel
VI. EVALUATION capacity. The peak channel capacities of the three attacks
are summarized in Table III. Prefetch+Reload always has
We evaluate the proposed covert channel and side channel
lower capacity than the other two attacks because more cache
attacks on modern Intel processors. For covert channel attacks,
operations are involved in each iteration of Prefetch+Reload.
weevaluatethechannelcapacities,comparingthemtoprevious
cache covert channels on CPU. For side channel attacks, we
TABLE II: The specifications of the tested processors.
demonstrate how they can be used to leak information from
common applications. In addition, we also show how our
Desktopprocessors Serverprocessors
attacks strengthen transient execution attacks. Platform Core Core XeonPlatinum XeonPlatinum
i7-6700 i7-7700K 8124M 8151
A. Evaluation of Prefetch-Based Covert Channel Attacks Microarchitecture Skylake KabyLake Skylake-SP Skylake-SP
Numofcores 4 4 N/A6 N/A
We implement Prefetch+Load, Prefetch+Prefetch, and
Frequency 3.4GHz 4.2GHz 3.0GHz 3.4GHz
Prefetch+Reload on four Intel processors, including two desk- LLCtype Inclusive Inclusive Non-inclusive Non-inclusive
top processors and two server processors. Note that although
Prefetch+Reload is introduced as a side channel attack in
Ourprefetch-basedattacksarefasterthanalmostallexisting
SectionV,itcanbeacovertchannelattackaswell.TableIIlists
cacheattacksonx86CPUs.First,forattackstestedondesktop
thespecificationsofthefourtestedprocessors.Thetwodesktop
processors,theringinterconnectcontentionbasedattack[48]is
processorshaveinclusiveLLCs,andtheserverprocessorshave
reported with a very high capacity which is 518 KB/s on a 4.0
non-inclusive LLCs.
GHz desktop processor. Flush+Reload and Flush+Flush have
Weuseonesharedcachelinebetweenthesenderandreceiver
capacities of 298 KB/s and 496 KB/s on a 3.6 GHz desktop
to transmit secrets. Although using more shared cache lines
or using channel coding techniques (e.g., [2]) may further
6WeuseIntelXeonScalableprocessorsonAmazonAWSEC2platform,
improve the channel capacity [2], [7]; here we do not include andweleasedfourphysicalcoresonthetestedprocessorsforourexperiments.

TABLE III: The maximum capacities of the prefetch-based lines containing those instructions are brought to the attacker’s
PREFETCHW
| channels. |     |                   |          |     |                  |              |     | L1D cache.  | Thus,            | although |        |              |          | can only | prefetch |
| --------- | --- | ----------------- | -------- | --- | ---------------- | ------------ | --- | ----------- | ---------------- | -------- | ------ | ------------ | -------- | -------- | -------- |
|           |     |                   |          |     |                  |              |     | cache lines | into             | L1D      | cache, | it can still | leak the | victim’s | access   |
|           |     | Desktopprocessors |          |     | Serverprocessors |              |     | patterns    | to instructions. |          |        |              |          |          |          |
| Platform  |     | Core              | Core     |     | XeonPlatinum     | XeonPlatinum |     |             |                  |          |        |              |          |          |          |
|           |     | i7-6700           | i7-7700K |     | 8124M            | 8151         |     |             |                  |          |        |              |          |          |          |
300
|                 |     | (3.4GHz) | (4.2GHz) |     | (3.0GHz) | (3.4GHz) |     | )selcyc( ycnetaL hcteferP |     |     |     |     |     |     |     |
| --------------- | --- | -------- | -------- | --- | -------- | -------- | --- | ------------------------- | --- | --- | --- | --- | --- | --- | --- |
| Prefetch+Reload |     | 631KB/s  | 782KB/s  |     | 394KB/s  | 476KB/s  |     |                           |     |     |     |     |     |     |     |
| Prefetch+Load   |     | 709KB/s  | 840KB/s  |     | 586KB/s  | 680KB/s  |     |                           |     |     |     |     |     |     |     |
200
| Prefetch+Prefetch |     | 721KB/s | 822KB/s |     | 556KB/s | 605KB/s |     |     |     |     |     |     |     |     |     |
| ----------------- | --- | ------- | ------- | --- | ------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
100
| processor | [2], | respectively. | Prime+Scope |     |     | [38], the | optimized |     |     |     |         |         |     |     |         |
| --------- | ---- | ------------- | ----------- | --- | --- | --------- | --------- | --- | --- | --- | ------- | ------- | --- | --- | ------- |
|           |      |               |             |     |     |           |           |     |     |     | Bit "0" | Bit "1" |     |     |  Square |
 Multiply
| attack for | Prime+Probe, |     | achieves |     | 438 KB/s | on  | a 3.5 GHz |     |     |     |     |     |     |     |     |
| ---------- | ------------ | --- | -------- | --- | -------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
0
desktop processor. Second, most of the attacks that were tested 1000 1050 1100 1150 1200
Sample ID
| on server | processors, |     | including | the | L1 LRU | attack | [5], the |     |     |     |     |     |     |     |     |
| --------- | ----------- | --- | --------- | --- | ------ | ------ | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
directory Prime+Probe attack [47], and the Flush+Coherence Fig. 7: A segment of the prefetch latencies measured in
| attack [7] | have | capacities | of less | than | 200 | KB/s. The | directory |                   |     |       |           |     |        |      |            |
| ---------- | ---- | ---------- | ------- | ---- | --- | --------- | --------- | ----------------- | --- | ----- | --------- | --- | ------ | ---- | ---------- |
|            |      |            |         |      |     |           |           | Prefetch+Prefetch |     | while | attacking |     | GnuPG; | part | of the the |
version of Prime+Scope achieves 387 KB/s. exponent e shown here is “111001011001”.
| To the | best | of our | knowledge, | our | attacks | are | only slower |     |     |     |     |     |     |     |     |
| ------ | ---- | ------ | ---------- | --- | ------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
than Streamline [62]. This attack claims to achieve a capacity Results. For simplicity, we only show the attack results of
of 1801 KB/s. However, it has such a high channel capacity Prefetch+Prefetch on the Intel Xeon Platinum 8151 processor.
| because | the sender | and | receiver | use | 64  | MB shared | data | to       |     |                |     |             |     |       |            |
| ------- | ---------- | --- | -------- | --- | --- | --------- | ---- | -------- | --- | -------------- | --- | ----------- | --- | ----- | ---------- |
|         |            |     |          |     |     |           |      | However, | we  | have performed |     | this attack | on  | other | processors |
transmit secrets; our results are based on one shared cache line listed in Table II too, using both Prefetch+Prefetch and
(64 B). Prefetch+Reload. Here we use a waiting latency of 500 cycles
|               |         |                   |        |                  |         |         |            | in each            | iteration | of         | Prefetch+Prefetch. |              | Figure   | 7         | shows the |
| ------------- | ------- | ----------------- | ------ | ---------------- | ------- | ------- | ---------- | ------------------ | --------- | ---------- | ------------------ | ------------ | -------- | --------- | --------- |
| B. Evaluation |         | of Prefetch-Based |        | Side             | Channel | Attacks |            |                    |           |            |                    |              |          |           |           |
|               |         |                   |        |                  |         |         |            | timing measurement |           |            | results            | from the     | attacker | for 200   | samples:  |
| 1) Side       | Channel |                   | Attack | on Cryptographic |         |         | Code: Our  |                    |           |            |                    |              |          |           |           |
|               |         |                   |        |                  |         |         |            | a lower            | prefetch  | latency    | (less              | than 100     | cycles)  | indicates | that      |
| first attack  | targets | cryptographic     |        | libraries,       |         | where   | the access |                    |           |            |                    |              |          |           |           |
|               |         |                   |        |                  |         |         |            | the victim         | did       | not access | the                | target cache | line     | during    | the last  |
patterns to some instructions are related to the value of the iteration; a higher prefetch latency (around 200 cycles) means
| cryptographic |           | key. More | specifically, |       | we      | target   | the square- |            |       |         |              |               |        |           |         |
| ------------- | --------- | --------- | ------------- | ----- | ------- | -------- | ----------- | ---------- | ----- | ------- | ------------ | ------------- | ------ | --------- | ------- |
|               |           |           |               |       |         |          |             | the victim | did   | access. | As explained |               | above, | an access | to sqr  |
| and-multiply  | algorithm |           | [63]          | which | is used | in GnuPG | 1.4.13      |            |       |         |              |               |        |           |         |
|               |           |           |               |       |         |          |             | followed   | by an | access  | to           | mul indicates | a      | bit “1”,  | and two |
for ciphers such as RSA [64] and ElGamal [65]: leaking the sqr
|          |      |                |     |       |             |      |          | consecutive | accesses |                 | to  | (one from | the       | current | iteration,  |
| -------- | ---- | -------------- | --- | ----- | ----------- | ---- | -------- | ----------- | -------- | --------------- | --- | --------- | --------- | ------- | ----------- |
| exponent | e of | this algorithm |     | leaks | the private | key. | As shown |             |          |                 |     |           |           |         |             |
|          |      |                |     |       |             |      |          | one from    | the      | next iteration) |     | indicate  | a bit “0” | (in     | the current |
in Algorithm 3, in each loop iteration, it first executes a sqr iteration). Thus, part of the exponent e shown in Figure 7 is
| and a mod   | instruction. |               | Then,    | if the        | exponent | bit is    | “1”, a mul   |                 |         |            |         |              |          |               |           |
| ----------- | ------------ | ------------- | -------- | ------------- | -------- | --------- | ------------ | --------------- | ------- | ---------- | ------- | ------------ | -------- | ------------- | --------- |
|             |              |               |          |               |          |           |              | “111001011001”. |         | The        | average | attack       | accuracy | is            | 96.2%.    |
| and another | mod          | instruction   |          | are executed; |          | otherwise | they are     |                 |         |            |         |              |          |               |           |
|             |              |               |          |               |          |           |              | 2) Side         | Channel | Attack     |         | on Keystroke | Timing:  |               | Oursecond |
| skipped.    | Thus,        | by monitoring |          | the access    | pattern  |           | to the cache |                 |         |            |         |              |          |               |           |
|             |              |               |          |               |          |           |              | attack focuses  |         | on leaking |         | the precise  | timing   | information   | of        |
| lines that  | contain      | sqr           | and mul, | the           | attacker | is        | able to leak |                 |         |            |         |              |          |               |           |
|             |              |               |          |               |          |           |              | keystrokes,     | i.e.,   | detecting  | when    | a keyboard   |          | input occurs. | This      |
each bit of the exponent e and therefore the decryption key. leakage is important since it can assist reconstructing typed
|     |     |     |     |     |     |     |     | words from | users | [66]–[68]. |     | Previous | work | shows | that certain |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | ----- | ---------- | --- | -------- | ---- | ----- | ------------ |
Algorithm 3: Square-and-multiply Exponentiation functions in graphics libraries are called when a keystroke
Input:baseb,modulom,exponente=(en1...e0)2 happens (e.g., [2], [69]). Thus, we can monitor the accesses to
Output:bemodm
|     |     |     |     |     |     |     |     | the cache       | lines | containing | these  | functions  | to  | detect     | keystrokes. |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------- | ----- | ---------- | ------ | ---------- | --- | ---------- | ----------- |
| r←1 |     |     |     |     |     |     |     | Implementation. |       | We         | attack | an address | in  | the shared | GDK         |
fori=n−1;i>=0;i−−do
r←r2 modn library which is invoked when processing keystrokes. Specifi-
ifei==1then cally, we launch gedit as the victim, and input keystrokes in it.
r←r∗b modn
|     |     |     |     |     |     |     |     | At the same | time,  | we      | run the  | prefetch-based |     | attacks  | to monitor |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ------ | ------- | -------- | -------------- | --- | -------- | ---------- |
|     |     |     |     |     |     |     |     | accesses    | to the | address | selected | in the         | GDK | library, | and record |
Implementation. As done in the Flush+Reload attack on the timing measurement results. The attacker process raises an
GnuPG [1], we use mmap to map the pages that contain sqr alarm when a keystroke is detected.
and mul into the attacker’s address space. Note that during the Results. Figure 8 shows the timing trace collected by
execution of the victim (GnuPG), the cache lines containing Prefetch+Reload when the user is typing “abcdefg1234” in
those instructions are brought into the victim’s L1 instruction gedit, on our Intel Core i7-6700 processor. Again, the attack
cache(L1Icache).However,sincewemaptheinstructionpages has been done on the other desktop processor too (but not
as data blocks in the attacker’s address space, the same cache on the server processors since EC2 instances do not come

with GUI). As one can observe, when a keystroke occurs, the
reload operation (in Step 3 of Prefetch+Reload) takes around
50 cycles to finish; it takes over 80 cycles to reload when
there is no keystroke. This significant timing difference makes
keystrokes very detectable. During the attack, we observe zero
false positives and zero false negatives.
120
100
80
60
40
20
0.0E+0 2.0E+9 4.0E+9 6.0E+9 8.0E+9
)selcyc(
ycnetaL
daoleR
1 .0
0 .9
0 .8
0 .7
0 .6
0 .5
2 0 0 4 0 0 1 0 0 0 1 0 0 0 0
a b c d e f g 1 2 3 4
Time (cycles)
Fig. 8: The access latencies measured in Step 3 of
Prefetch+Reload when a user types “abcdefg1234” in gedit;
we monitor address 0x7b980 of libgdk.so.7
3) Windowless Prefetch+Prefetch: Using the terminology
in prior work [38], PREFETCHW has two important properties:
1) PREFETCHW is preserving, meaning the measurement
(prefetching and timing the prefetch) does not change the
state in the absence of the victim’s event; 2) PREFETCHW is
also concurrent, meaning it detects the events that temporally
overlap with it. With these two features, Prefetch+Prefetch can
be used in a windowless way (no waiting window between
two consecutive prefetches is necessary). We verify this using
the following experiment.
We use two processes, the victim and attacker. The victim
process first waits a random amount of time, and then triggers
an event (accessing the target shared cache line). This process
terminates after triggering the event. The attacker process
runs Prefetch+Prefetch with a waiting window in each attack
iteration to detect the victim’s event. The attacker process
terminates after detecting the event or after the victim process
terminates.Werunthisexperimentwithdifferentwindowsizes
and repeat the experiment for 1000 times for each window
size. Figure 9 shows the attacker’s detection accuracy on our
Intel Core i7-6700 processor. Note that the results on other
processors in Table II are similar. For comparison, we also
show the accuracy of Flush+Reload on the same processor.
For Prefetch+Prefetch, the attacker’s detection accuracy does
not change when the window size varies; the attacker always
has a very high detection accuracy which is around 1. This
indicates that Prefetch+Prefetch, unlike prior attacks such as
Flush+Reload, can always be used as a windowless attack.
Such a windowless attack has much higher temporal resolution
than a windowed attack since the latter’s resolution is bounded
by the window size. For example, to reach 95% detection
7We found the appropriate library and address to monitor following the
methodinpriorwork[8].
y c
ar
u
c
c
A
P re fe tc h + P re fe tc h
F lu s h + R e lo a d
W in d o w S iz e (c y c le s )
Fig. 9: The accuracy of Prefetch+Prefetch and Flush+Reload
on our Intel Core i7-6700 processor, with different waiting
window sizes.
accuracy, Flush+Reload needs a waiting window with over
4000 cycles.
C. Prefetch-Based Channels in Transient Execution Attacks
Transient execution attacks such as Spectre [26] and Melt-
down [27] usually require a microarchitectural covert channel
to transfer the secrets to the attacker. Currently, most transient
execution attacks (e.g., [26]–[29], [70]) use the Flush+Reload
channel because it is simple, reliable, and ubiquitous. Here we
demonstrate that prefetch-based channels can also work with
transient execution attacks to leak secrets, and may even work
better than Flush+Reload. We use Spectre v1 as an example to
show the details and benefits of using prefetch-based channels
in transient execution attacks.
Higher bandwidth. When using Flush+Reload, the sender
operation in Spectre is a memory access where the ad-
dress depends on the secret value. Since Prefetch+Reload
and Prefetch+Prefetch use the same sender function as
Flush+Reload, a victim program vulnerable to Spectre
with Flush+Reload is also vulnerable to Spectre with
Prefetch+Reload and Prefetch+Prefetch. We have verified
this using the Spectre v1 PoC code [71]. We modify it
accordingly such that Prefetch+Reload or Prefetch+Prefetch
is used in the attacker code; the victim remains the same. In
addition, as observed in prior work [27], the leakage rate
of a transient execution attack is significantly affected by
the capacity of the covert channel used in the attack. Since
Prefetch+Reload and Prefetch+Prefetch have much higher
capacities than Flush+Reload, Spectre works faster with these
twochannels.Forexample,onourIntelCorei7-6700processor,
when leaking an 8-bit secret in each transmission, the leakage
rate of Spectre is 3.02 times and 1.61 times as fast when
using Prefetch+Prefetch and Prefetch+Reload, respectively, as
compared to Flush+Reload. The results on other processors
are shown in Appendix A.
More leakage in the transient window. When using Spectre
with Flush+Reload, the data access for sending (encoding) the
secret in the transient window is a slow DRAM access, since
this data was flushed by the attacker. In contrast, the data
access for secret encoding is a remote private cache hit when

using Prefetch+Reload or Prefetch+Prefetch, which is usually Other transient execution attacks. All of the three prefetch-
fasterthanaDRAMaccess.Thisindicatesthatwithinthesame basedchannelscanbeusedintransientexecutionattackswhen
transient window, more encoding operations can be performed the attacker has full control of the gadget (e.g., Meltdown). As
using the two prefetch-based channels than Flush+Reload, and shownabove,Prefetch+ReloadandPrefetch+Prefetchhasfaster
thus more secrets may be leaked. An example Spectre v1 encodingoperationsthanFlush+Reload,enablingmoreleakage
gadget that can benefit from this is shown in Listing 3. There in a transient window. The same is true for Prefetch+Load,
arenoperationsinthebranch,whereeachoperationaccessesa since a remote private cache hit for PREFETCHW is usually
secretandencodesittoacacheindex.Thesecretsarearray1[x] faster than a DRAM access. In a Meltdown PoC with the three
to array1[x+n] (when x is out of bounds); each of the secrets prefetch-based channels, we can reliably leak 8 bytes in the
is encoded to an index of a sub-array in array2. The more of transient window on Our Intel Core i7-6700 processor; we can
these n operations we can perform in the transient window, the only leak 6.1 bytes on average when using Flush+Reload. An
more secrets we can leak out at once. example gadget to achieve is shown in Appendix B.
if (x+n < array1 size)
{ 16
y0 = array2[0][array1[x] * 4096]; 14
y2 = array2[1][array1[x+1] * 4096]; 12
... 10
yn = array2[2][array1[x+n] * 4096];
8
}
6
4
Listing 3: The Spectre v1 code example when a bounds 2
0.0 0.2 0.4 0.6
check is followed by multiple secret accessing and encoding Flush+Reload
operations.Thiscodeisessentiallyaforloopinaconditional
branch; we show the unrolled version for clarity.
This gadget might be found in a victim; it is essentially
the original Spectre v1 gadget with multiple secrets accessed
and encoded in the branch (instead of one). Additionally, in
the scenario where the attacker has control over the gadget
(e.g., spectre-type-meltdown),8 the attacker can build such a
gadgettoleakmultiplesecretsinonetransientwindowandthus
acceleratetheattack.WestillprovethiswiththeSpectrev1PoC
code and modify the attacker code to use Prefetch+Reload or
Prefetch+Prefetch. We also modify the victim code to simulate
the gadget in Listing 3 where n secrets are accessed and
encodedinthebranch.Werunthiscodeandcollecttheamount
of these n secrets the victim can transmit within one transient
window, and draw the histograms in Figure 10. We omit the
results when leaking by Prefetch+Reload since its encoding
stage is same as the one of Prefetch+Prefetch.
On the desktop processors, the victim can transmit up to
17 8-bit secrets speculatively when using Prefetch+Reload
or Prefetch+Prefetch, while the victim can transmit at most
8 secrets when using Flush+Reload. However, on server
processors, the amount of transmitted secrets when using
prefetch-based channels is only slightly larger than the one
when using Flush+Reload. This is because on these processors,
the latency of a remote private cache hit is much longer,
compared to desktop processors (160 cycles vs. 90 cycles).
Notethatalthoughsame-coreprivatecacheattacks,suchasthe
L1 LRU attack [5], can also achieve more secret encodings in
a transient window than Flush+Reload, these attacks are less
practical, because they are limited by the number of private
cache sets. In these attacks, secret values are encoded into the
cache set index instead of cache line index.
8SpectrecanbeusedforexceptionsuppressioninMeltdown.
setyB
dekaeL
fo muN
18
16
14
12
10
8
6
4
2
0.0 0.2 0.4 0.6 0.0 0.2 0.4 0.6
Prefetch+Prefetch Flush+Reload
(a) IntelCorei7-6700
setyB
dekaeL
fo muN
0.0 0.2 0.4 0.6
Prefetch+Prefetch
(b) IntelCorei7-7700K
16
14
12
10
8
6
4
2
0.0 0.2 0.4 0.6
Flush+Reload
setyB
dekaeL
fo
muN
16
14
12
10
8
6
4
2
0.0 0.2 0.4 0.6 0.0 0.2 0.4 0.6
Prefetch+Prefetch Flush+Reload
(c) IntelXeonPlatinum8124M
setyB
dekaeL
fo
muN
0.69 0.91
0.0 0.2 0.4 0.6
Prefetch+Prefetch
(d) IntelXeonPlatinum8151
Fig. 10: The distributions of the amount of secret bytes that
can be accessed and encoded in a transient window, when
leaking by Flush+Reload and Prefetch+Prefetch, respectively.
VII. DISCUSSION
A. Attack Reliability
According to Intel [43], a prefetch instruction will not fetch
any data when the request buffer between the L1 and L2 cache
is full. This may reduce the performance of the prefetch-based
attacks, when SMT is available and a memory-intensive thread
is located on the same core as the attacker thread. We verified
thisbyrunningstress -m 1inaco-locatedthread(i.e.,the
hyperthread sibling) of the attacker thread: this causes many
prefetch instructions from the attacker to be ignored, which
significantly reduces the attack performance. For example,
on our Intel Core i7-6700 processor, the channel capacity
of Prefetch+Prefetch is reduced to 56 KB/s. However, SMT
enables many security vulnerabilities (e.g., [72]) and thus is
often suggested to be disabled. In fact, if SMT is available,
the attacker can always launch same-core private cache attacks
instead.Ourcross-coreprivatecacheattackstargetthescenarios
where same-core attacks are impractical or impossible.

B. Prefetch-Based Attacks on AMD attacker learns the victim’s behavior by distinguishing between
Modern AMD processors also support PREFETCHW; this remote private cache hits and DRAM accesses. A variant of
|               |                |     |           |     |      |             |     | Flush+Reload | attack            | on  | x86 processors |        | was proposed | in [7].    |
| ------------- | -------------- | --- | --------- | --- | ---- | ----------- | --- | ------------ | ----------------- | --- | -------------- | ------ | ------------ | ---------- |
| instruction   | was originally |     | invented  | by  | AMD  | [55], and   | was |              |                   |     |                |        |              |            |
|               |                |     |           |     |      |             |     | It works     | by distinguishing |     | between        | remote | private      | cache hits |
| later adopted | by Intel.      | We  | performed | the | same | experiments | as  |              |                   |     |                |        |              |            |
the ones in Section III on AMD desktop and server processors. and LLC hits. These attacks are more general than the ones
|          |          |              |     |           |     |          |       | discussed | earlier, | but they | all | suffer | from low bandwidth | as  |
| -------- | -------- | ------------ | --- | --------- | --- | -------- | ----- | --------- | -------- | -------- | --- | ------ | ------------------ | --- |
| However, | from our | experiments, |     | PREFETCHW |     | does not | cause |           |          |          |     |        |                    |     |
anycoherencestatechangesondatawithread-onlypermission; DRAM accesses are involved in the attacks.
| it only works | on data    | with     | write | permission. |            | Thus, we | believe |                |     |     |     |     |     |     |
| ------------- | ---------- | -------- | ----- | ----------- | ---------- | -------- | ------- | -------------- | --- | --- | --- | --- | --- | --- |
| that AMD      | processors | actually |       | have        | permission | checks   | for     | D. Mitigations |     |     |     |     |     |     |
PREFETCHW.
|     |     |     |     |     |     |     |     | Our | attacks | can be | prevented | through | modifications | on  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | ------ | --------- | ------- | ------------- | --- |
C. Related Work the microarchitecture behavior of PREFETCHW. The complete
protectionistwo-fold.First,PREFETCHWshouldperformwrite
| Prefetch-based | attacks. |     | Gruss | et al. | [73] made | two | obser- |     |     |     |     |     |     |     |
| -------------- | -------- | --- | ----- | ------ | --------- | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
vations about prefetch instructions on Intel processors. They permission checks, just as a regular memory write instruction,
|            |               |     |         |            |     |              |      | and trigger | a fault | or ignore | this | instruction | if the target | data is |
| ---------- | ------------- | --- | ------- | ---------- | --- | ------------ | ---- | ----------- | ------- | --------- | ---- | ----------- | ------------- | ------- |
| found that | the execution |     | time of | a prefetch |     | instruction, | such |             |         |           |      |             |               |         |
as PREFETCHT0, leaks the translation levels of inaccessible not writable. Second, PREFETCHW should execute in constant
|                   |       |        |               |       |     |           |       | time. These | modifications |             | may         | introduce | some performance |       |
| ----------------- | ----- | ------ | ------------- | ----- | --- | --------- | ----- | ----------- | ------------- | ----------- | ----------- | --------- | ---------------- | ----- |
| kernel addresses. |       | Using  | this, they    | built | an  | attack to | break |             |               |             |             |           |                  |       |
|                   |       |        |               |       |     |           |       | overhead.   | We do         | not suggest | eliminating |           | PREFETCHW        | since |
| Kernel Address    | Space | Layout | Randomization |       |     | (KASLR).  | They  |             |               |             |             |           |                  |       |
also observed that prefetch instructions change the cache state it is important for improving write performance.
|                 |        |         |     |            |      |      |        | Similar | to prior | cache | attacks, | our | attacks also | work by |
| --------------- | ------ | ------- | --- | ---------- | ---- | ---- | ------ | ------- | -------- | ----- | -------- | --- | ------------ | ------- |
| of inaccessible | kernel | memory, |     | but recent | work | [74] | proved |         |          |       |          |     |              |         |
thisincorrect.Infact,theirobservationistheresultoftransient manipulating and monitoring cache states. Thus, defenses
|           |        |              |     |        |        |         |         | against | prior cache | attacks | (e.g., | [84]–[88]) | may also | work |
| --------- | ------ | ------------ | --- | ------ | ------ | ------- | ------- | ------- | ----------- | ------- | ------ | ---------- | -------- | ---- |
| execution | caused | by a Spectre |     | gadget | in the | kernel, | not the |         |             |         |        |            |          |      |
prefetch instruction. on our attacks. For example, DAWG [85] allows replicated
Very recently, Lipp et al. [75] observed that on AMD cache copies of the data shared across security domains: each
|             |            |      |       |              |     |               |     | domain | gets their | own | copy of | this data | in cache. | Thus, the |
| ----------- | ---------- | ---- | ----- | ------------ | --- | ------------- | --- | ------ | ---------- | --- | ------- | --------- | --------- | --------- |
| processors, | the timing | (and | power | consumption) |     | of a prefetch |     |        |            |     |         |           |           |           |
instruction on an inaccessible kernel address leaks the trans- attacker cannot monitor the coherence state changes from a
lation level and TLB state of this address. They used this to victimwhoisinanotherdomain,whichwouldstopourattacks.
breakKASLRandleakkernelmemory(withSpectre)onAMD
| processors.   | These | two prefetch    |     | attacks   | are orthogonal |             | to our |     |     |       |            |     |     |     |
| ------------- | ----- | --------------- | --- | --------- | -------------- | ----------- | ------ | --- | --- | ----- | ---------- | --- | --- | --- |
|               |       |                 |     |           |                |             |        |     |     | VIII. | CONCLUSION |     |     |     |
| attacks. They | focus | on specifically |     | attacking |                | the kernel; | we     |     |     |       |            |     |     |     |
instead focus on building general cache timing attacks. In this paper, we proposed a new cache eviction method
Regarding attacks based on hardware prefetchers, Shin et as well as two new two cross-core cache side channel attacks
al.[76]attackedOpenSSL,leakingtheprivatekeybyleveraging that work with both inclusive and non-inclusive LLCs. One
the Intel stride prefetcher. Rohan et al. [77] reverse-engineered of the prefetch instructions on x86 processors, PREFETCHW,
|            |            |     |                   |     |       |       |         | prepares | the data | for future | writes | by  | modifying the | coherence |
| ---------- | ---------- | --- | ----------------- | --- | ----- | ----- | ------- | -------- | -------- | ---------- | ------ | --- | ------------- | --------- |
| the stream | prefetcher | on  | Intel processors, |     | using | it to | build a |          |          |            |        |     |               |           |
covert channel. state of the data. In this work, we made two important
Cache coherence vulnerabilities. Although we are the first microarchitecturalobservationsonPREFETCHW.First,itworks
to propose cross-core private cache side channel attacks lever- on data with read-only permission. Second, its execution
aging cache coherence protocol invalidations, cache coherence time is related to the coherence state of the target data.
protocolshavebeenexploitedinmanydifferentattacks.Trippel Given these observations, the coherence state modifications by
et al. [78] discovered that a transient write may change the PREFETCHW enable significant security vulnerabilities. Using
coherencestateofthetargetdata,whichcanbeusedasacovert PREFETCHW, we first built two covert channel attacks which
channel in transient execution attacks. In addition, previous have very high capacities. We also demonstrated that these
studies [26], [79], [80] mention that “bouncing” cache lines high-capacity covert channels enable more powerful transient
between private caches may be used as a replacement for executionattacks.Wethenslightlymodifiedthecovertchannel
CLFLUSH or set conflicts in Spectre and Rowhammer attacks. attacks to build two side channel attacks and showed that these
However, in this method, coherence states are manipulated by attacks leaked information from real-world applications.
| write operations. |            | This means | it  | requires | that     | at least | part of |     |     |     |     |     |     |     |
| ----------------- | ---------- | ---------- | --- | -------- | -------- | -------- | ------- | --- | --- | --- | --- | --- | --- | --- |
| the target        | cache line | happens    | to  | contain  | writable | data     | (unless |     |     |     |     |     |     |     |
IX. ACKNOWLEDGEMENT
| Meltdown-RW | [81], | [82] | can be | exploited). | Unfortunately, |     | as  |     |     |     |     |     |     |     |
| ----------- | ----- | ---- | ------ | ----------- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
discussed in [80], this requirement is impractical for general WethanktheanonymousIEEES&P2022reviewersfortheir
side channel attacks. insightful feedback. We would also like to thank Daniel Weber
Prior work [37], [83] built cross-core attacks on AMD and forthehelpontheMeltdownimplementation,andDanielGruss
ARM processors, respectively, based on cache coherence. An for his comments on the preprint. This work is supported in
Evict+Reload attack on Intel processors with non-inclusive part by US National Science Foundation #1422331, #1535755,
LLCs was proposed in [47]. In these three attacks, the #1617071, #1718080, #1725657, #1910413, and #2011146.

REFERENCES
|     |     |     |     |     |     |     |     | [24] C.Disselkoen,D.Kohlbrenner,L.Porter,andD.Tullsen,“Prime+Abort: |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
Atimer-freehigh-precisionL3cacheattackusingIntelTSX,”inUSENIX
SecuritySymposium,2017.
[1] Y.YaromandK.Falkner,“Flush+Reload:Ahighresolution,lownoise,
|     |     |     |     |     |     |     |     | [25] T.Hornby,“Side-channelattacksoneverydayapplications:Distinguishing |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
L3cacheside-channelattack,”inUSENIXSecuritySymposium,2014.
inputswithFlush+Reload,”BlackHatUSA,2016.
[2] D.Gruss,C.Maurice,K.Wagner,andS.Mangard,“Flush+Flush:afast
|     |     |     |     |     |     |     |     | [26] P.Kocher,J.Horn,A.Fogh,,D.Genkin,D.Gruss,W.Haas,M.Hamburg, |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
andstealthycacheattack,”inInternationalConferenceonDetectionof M.Lipp,S.Mangard,T.Prescher,M.Schwarz,andY.Yarom,“Spectre
IntrusionsandMalware,andVulnerabilityAssessment(DIMVA),2016.
|           |         |             |     |          |             |     |           | attacks: | Exploiting | speculative | execution,” | in IEEE | Symposium | on  |
| --------- | ------- | ----------- | --- | -------- | ----------- | --- | --------- | -------- | ---------- | ----------- | ----------- | ------- | --------- | --- |
| [3] Y. A. | Younis, | K. Kifayat, | Q.  | Shi, and | B. Askwith, | “A  | new Prime |          |            |             |             |         |           |     |
SecurityandPrivacy(S&P),2019.
| and Probe     | cache      | side-channel |             | attack for | cloud           | computing,” | in IEEE     |               |     |          |              |           |             |       |
| ------------- | ---------- | ------------ | ----------- | ---------- | --------------- | ----------- | ----------- | ------------- | --- | -------- | ------------ | --------- | ----------- | ----- |
|               |            |              |             |            |                 |             |             | [27] M. Lipp, | M.  | Schwarz, | D. Gruss, T. | Prescher, | W. Haas, A. | Fogh, |
| International | Conference |              | on Computer |            | and Information |             | Technology; |               |     |          |              |           |             |       |
J.Horn,S.Mangard,P.Kocher,D.Genkin,Y.Yarom,andM.Hamburg,
UbiquitousComputingandCommunications;Dependable,Autonomic “Meltdown:Readingkernelmemoryfromuserspace,”inUSENIXSecurity
andSecureComputing;PervasiveIntelligenceandComputing,2015.
Symposium,2018.
[4] F.Liu,Y.Yarom,Q.Ge,G.Heiser,andR.B.Lee,“Last-levelcache
|     |     |     |     |     |     |     |     | [28] M.Schwarz,M.Lipp,D.Moghimi,J.VanBulck,J.Stecklina,T.Prescher, |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------------------ | --- | --- | --- | --- | --- | --- |
side-channelattacksarepractical,”inIEEESymposiumonSecurityand
andD.Gruss,“ZombieLoad:Cross-privilege-boundarydatasampling,”
Privacy(S&P),2015.
inACMSIGSACConferenceonComputerandCommunicationsSecurity
| [5] W. Xiong | and | J. Szefer, | “Leaking | information |     | through | cache LRU | (CCS),2019. |     |     |     |     |     |     |
| ------------ | --- | ---------- | -------- | ----------- | --- | ------- | --------- | ----------- | --- | --- | --- | --- | --- | --- |
states,”inIEEEInternationalSymposiumonHighPerformanceComputer [29] S. van Schaik, A. Milburn, S. O¨sterlund, P. Frigo, G. Maisuradze,
Architecture(HPCA),2020. K.Razavi,H.Bos,andC.Giuffrida,“RIDL:Roguein-flightdataload,”
S.Briongos,P.Malago´n,J.M.Moya,andT.Eisenbarth,“Reload+Refresh:
| [6] |     |     |     |     |     |     |     | inIEEESymposiumonSecurityandPrivacy(S&P),May2019. |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------- | --- | --- | --- | --- | --- | --- |
Abusingcachereplacementpoliciestoperformstealthycacheattacks,”
|     |     |     |     |     |     |     |     | [30] D.Skarlatos,M.Yan,B.Gopireddy,R.Sprabery,J.Torrellas,andC.W. |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
inUSENIXSecuritySymposium,2020. Fletcher,“MicroScope:Enablingmicroarchitecturalreplayattacks,”in
[7] F. Yao, M. Doroslovacki, and G. Venkataramani, “Are coherence InternationalSymposiumonComputerArchitecture(ISCA),2019.
protocolstatesvulnerabletoinformationleakage?,”inIEEEInternational [31] J.VanBulck,D.Moghimi,M.Schwarz,M.Lipp,M.Minkin,D.Genkin,
SymposiumonHighPerformanceComputerArchitecture(HPCA),2018. Y.Yuval,B.Sunar,D.Gruss,andF.Piessens,“LVI:Hijackingtransient
[8] D. Gruss, R. Spreitzer, and S. Mangard, “Cache template attacks: execution through microarchitectural load value injection,” in IEEE
Automatingattacksoninclusivelast-levelcaches,”inUSENIXSecurity SymposiumonSecurityandPrivacy(S&P),2020.
Symposium,2015. [32] M.Schwarz,M.Schwarzl,M.Lipp,J.Masters,andD.Gruss,“NetSpec-
[9] D.Gullasch,E.Bangerter,andS.Krenn,“Cachegames–bringingaccess- tre:Readarbitrarymemoryovernetwork,”inEuropeanSymposiumon
basedcacheattacksonAEStopractice,”inIEEESymposiumonSecurity ResearchinComputerSecurity,2019.
andPrivacy(S&P),2011. [33] C. Trippel, D. Lustig, and M. Martonosi, “Checkmate: Automated
synthesisofhardwareexploitsandsecuritylitmustests,”inIEEE/ACM
[10] C.Percival,“Cachemissingforfunandprofit,”2005.
InternationalSymposiumonMicroarchitecture(MICRO),2018.
[11] D.A.Osvik,A.Shamir,andE.Tromer,“Cacheattacksandcountermea-
sures:thecaseofAES,”inCryptographers’trackattheRSAconference, [34] S. van Schaik, M. Minkin, A. Kwong, D. Genkin, and Y. Yarom,
2006. “CacheOut:LeakingdataonIntelCPUsviacacheevictions,”inIEEE
[12] Y. Oren, V. P. Kemerlis, S. Sethumadhavan, and A. D. Keromytis, SymposiumonSecurityandPrivacy(S&P),2021.
|      |            |          |           |       |         |               |     | [35] A. Bhattacharyya, |     | A. Sandulescu, | M.  | Neugschwandtner, | A. Sorniotti, |     |
| ---- | ---------- | -------- | --------- | ----- | ------- | ------------- | --- | ---------------------- | --- | -------------- | --- | ---------------- | ------------- | --- |
| “The | spy in the | sandbox: | Practical | cache | attacks | in JavaScript | and |                        |     |                |     |                  |               |     |
B.Falsafi,M.Payer,andA.Kurmus,“SMoTherSpectre:Exploitingspec-
| their | implications,” | in  | ACM SIGSAC |     | Conference | on Computer | and |     |     |     |     |     |     |     |
| ----- | -------------- | --- | ---------- | --- | ---------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
CommunicationsSecurity(CCS),2015. ulativeexecutionthroughportcontention,”inACMSIGSACConference
[13] G.Irazoqui,T.Eisenbarth,andB.Sunar,“S$A:Asharedcacheattack onComputerandCommunicationsSecurity(CCS),2019.
thatworksacrosscoresanddefiesVMsandboxing–anditsapplication [36] M. Behnia, P. Sahu, R. Paccagnella, J. Yu, Z. N. Zhao, X. Zou,
T.Unterluggauer,J.Torrellas,C.Rozas,A.Morrison,etal.,“Speculative
toAES,”inIEEESymposiumonSecurityandPrivacy(S&P),2015.
interferenceattacks:Breakinginvisiblespeculationschemes,”inACM
[14] T.Ristenpart,E.Tromer,H.Shacham,andS.Savage,“Hey,you,get
|     |     |     |     |     |     |     |     | International |     | Conference | on Architectural | Support | for Programming |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ---------- | ---------------- | ------- | --------------- | --- |
offofmycloud:Exploringinformationleakageinthird-partycompute
clouds,”inACMSIGSACConferenceonComputerandCommunications LanguagesandOperatingSystems(ASPLOS),2021.
Security(CCS),2009. [37] G.Irazoqui,T.Eisenbarth,andB.Sunar,“Crossprocessorcacheattacks,”
inACMonAsiaconferenceoncomputerandcommunicationssecurity
[15] Z.Wu,Z.Xu,andH.Wang,“Whispersinthehyper-space:High-speed
(AsiaCCS),2016.
covertchannelattacksinthecloud,”inUSENIXSecuritySymposium,
|     |     |     |     |     |     |     |     | [38] A.Purnal,F.Turan,andI.Verbauwhede,“Prime+Scope:Overcoming |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
2012.
theobservereffectforhigh-precisioncachecontentionattacks,”inACM
| [16] Y. Zhang, | A. Juels, | M.  | K. Reiter, | and | T. Ristenpart, | “Cross-VM | side |     |     |     |     |     |     |     |
| -------------- | --------- | --- | ---------- | --- | -------------- | --------- | ---- | --- | --- | --- | --- | --- | --- | --- |
SIGSACConferenceonComputerandCommunicationsSecurity(CCS),
| channels | and | their use | to extract | private | keys,” | in ACM | SIGSAC | 2021. |     |     |     |     |     |     |
| -------- | --- | --------- | ---------- | ------- | ------ | ------ | ------ | ----- | --- | --- | --- | --- | --- | --- |
ConferenceonComputerandCommunicationsSecurity(CCS),2012.
|                |     |           |            |     |                |     |               | [39] M.K.Qureshi,“Newattacksanddefenseforencrypted-addresscache,” |     |     |     |     |     |     |
| -------------- | --- | --------- | ---------- | --- | -------------- | --- | ------------- | ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
| [17] Y. Zhang, | A.  | Juels, M. | K. Reiter, | and | T. Ristenpart, |     | “Cross-tenant |                                                                   |     |     |     |     |     |     |
inACM/IEEEAnnualInternationalSymposiumonComputerArchitecture
side-channelattacksinPaaSclouds,”inACMSIGSACConferenceon
(ISCA),2019.
ComputerandCommunicationsSecurity(CCS),2014.
|     |     |     |     |     |     |     |     | [40] M. Werner, | T.  | Unterluggauer, | L. Giner, | M. Schwarz, | D. Gruss, | and |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------- | --- | -------------- | --------- | ----------- | --------- | --- |
[18] R.Hund,C.Willems,andT.Holz,“Practicaltimingsidechannelattacks S. Mangard, “ScatterCache: Thwarting cache attacks via cache set
againstkernelspaceASLR,”inIEEESymposiumonSecurityandPrivacy randomization,”inUSENIXSecuritySymposium,2019.
(S&P),2013.
|     |     |     |     |     |     |     |     | [41] M.Yan,B.Gopireddy,T.Shull,andJ.Torrellas,“Securehierarchy-aware |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
[19] O.Acıic¸mez,“Yetanothermicroarchitecturalattack:ExploitingI-cache,”
cachereplacementpolicy(SHARP):Defendingagainstcache-basedside
inACMWorkshoponComputerSecurityArchitecture,2007.
channelatacks,”inProceedingsoftheAnnualInternationalSymposium
[20] O.Acıic¸mezandW.Schindler,“AvulnerabilityinRSAimplementations onComputerArchitecture(ISCA),2017.
duetoinstructioncacheanalysisanditsdemonstrationonOpenSSL,”in [42] M. K. Qureshi, “Ceaser: Mitigating conflict-based cache attacks via
Cryptographers’TrackattheRSAConference,2008. encrypted-addressandremapping,”inAnnualIEEE/ACMInternational
[21] A. Shusterman, A. Agarwal, S. O’Connell, D. Genkin, Y. Oren, and SymposiumonMicroarchitecture(MICRO),2018.
| Y. Yarom, | “Prime+Probe |     | 1, JavaScript |     | 0: Overcoming |     | browser-based |                                                                        |     |     |     |     |     |     |
| --------- | ------------ | --- | ------------- | --- | ------------- | --- | ------------- | ---------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|           |              |     |               |     |               |     |               | [43] “Intel®64andIA-32architecturesoptimizationreferencemanual.”Avail- |     |     |     |     |     |     |
side-channeldefenses,”inUSENIXSecuritySymposium,2021. ableathttps://software.intel.com/content/www/us/en/develop/download/
[22] L. Groot Bruinderink, A. Hu¨lsing, T. Lange, and Y. Yarom, “Flush, intel-64-and-ia-32-architectures-optimization-reference-manual.html.
Gauss,andReload–acacheattackontheBLISSlattice-basedsignature [44] P.ConwayandB.Hughes,“TheAMDOpteronnorthbridgearchitecture,”
| scheme,”inInternationalConferenceonCryptographicHardwareand |     |     |     |     |     |     |     | IEEEMicro,2007. |     |     |     |     |     |     |
| ----------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- |
EmbeddedSystems,2016.
|     |     |     |     |     |     |     |     | [45] “IntelQuickPatharchitecture,”2012. |     |     | Availableathttp://www.intel.com/ |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------------------------------- | --- | --- | -------------------------------- | --- | --- | --- |
[23] A.Shusterman,L.Kang,Y.Haskal,Y.Meltser,P.Mittal,Y.Oren,and pressroom/archive/reference/whitepaperQuickPath.pdf.
Y.Yarom,“Robustwebsitefingerprintingthroughthecacheoccupancy [46] “Intel® Xeon® Scalable Processor: The foundation of data
channel,”inUSENIXSecuritySymposium,2019. centre innovation,” 2017. Available at https://simplecore-

ger.intel.com/swdevcon-uk/wp-content/uploads/sites/5/2017/10/UK- [72] J.VanBulck,M.Minkin,O.Weisse,D.Genkin,B.Kasikci,F.Piessens,
Dev-Con Toby-Smith-Track-A 1000.pdf. M.Silberstein,T.F.Wenisch,Y.Yarom,andR.Strackx,“Foreshadow:
[47] M. Yan, R. Sprabery, B. Gopireddy, C. Fletcher, R. Campbell, and ExtractingthekeystotheIntelSGXkingdomwithtransientout-of-order
J.Torrellas,“Attackdirectories,notcaches:Sidechannelattacksina execution,”inUSENIXSecuritySymposium,2018.
non-inclusiveworld,”inIEEESymposiumonSecurityandPrivacy(S&P), [73] D.Gruss,C.Maurice,A.Fogh,M.Lipp,andS.Mangard,“Prefetchside-
channelattacks:BypassingSMAPandkernelASLR,”inACMSIGSAC
2019.
[48] R.Paccagnella,L.Luo,andC.W.Fletcher,“LordoftheRing(s):Side ConferenceonComputerandCommunicationsSecurity(CCS),2016.
channelattacksontheCPUon-chipringinterconnectarepractical,”in [74] M. Schwarzl, T. Schuster, M. Schwarz, and D. Gruss, “Speculative
USENIXSecuritySymposium,2021. dereferencing:RevivingForeshadow,”inInternationalConferenceon
[49] A.Moghimi,J.Wichelmann,T.Eisenbarth,andB.Sunar,“Memjam:A FinancialCryptographyandDataSecurity,2021.
|     |     |     |     |     |     |     |     | [75] M.Lipp,D.Gruss,andM.Schwarz,“AMDprefetchattacksthrough |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------------- | --- | --- | --- | --- |
falsedependencyattackagainstconstant-timecryptoimplementations,”
powerandtime,”inUSENIXSecuritySymposium,2022.
InternationalJournalofParallelProgramming,2019.
[50] Y.Yarom,D.Genkin,andN.Heninger,“CacheBleed:Atimingattack [76] Y. Shin, H. C. Kim, D. Kwon, J. H. Jeong, and J. Hur, “Unveiling
onOpenSSLconstant-timeRSA,”JournalofCryptographicEngineering, hardware-baseddataprefetcher,ahiddensourceofinformationleakage,”
2017. inACMSIGSACConferenceonComputerandCommunicationsSecurity
(CCS),2018.
| [51] L. | Armasu,            | “OpenBSD | will     | disable     | Intel | Hyper-Threading |           |                                                                |     |     |     |     |
| ------- | ------------------ | -------- | -------- | ----------- | ----- | --------------- | --------- | -------------------------------------------------------------- | --- | --- | --- | --- |
|         |                    |          |          |             |       |                 |           | [77] A.Rohan,B.Panda,andP.Agarwal,“Reverseengineeringthestream |     |     |     |     |
| to      | avoid Spectre-like |          | exploits | (updated),” |       | 2018.           | Available | at                                                             |     |     |     |     |
https://www.tomshardware.com/news/openbsd-disables-intel-hyper- prefetcher for profit,” in IEEE European Symposium on Security and
threading-spectre,37332.html. PrivacyWorkshops(EuroS&PW),2020.
[52] T.Claburn,“RIPHyper-Threading?ChromeOSaxeskeyIntelCPUfea- [78] C. Trippel, D. Lustig, and M. Martonosi, “MeltdownPrime and Spec-
trePrime:Automatically-synthesizedattacksexploitinginvalidation-based
| tureoverdata-leakflaws–Microsoft,Applesuggestsnub,”2019. |     |     |     |     |     |     | Avail- |     |     |     |     |     |
| -------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- |
coherenceprotocols,”arXivpreprintarXiv:1802.03802,2018.
| ableathttps://www.theregister.co.uk/2019/05/14/intel |     |     |     |     |     | hyper | threading |                                                                        |     |     |     |     |
| ---------------------------------------------------- | --- | --- | --- | --- | --- | ----- | --------- | ---------------------------------------------------------------------- | --- | --- | --- | --- |
|                                                      |     |     |     |     |     |       |           | [79] J.Horn,“CPUsecuritybug:Informationleakusingspeculativeexecution,” |     |     |     |     |
mitigations/.
[53] A.Marshall,M.Howard,G.Bugher,B.Harden,C.Kaufman,M.Rues, 2017. Available at https://bugs.chromium.org/p/project-zero/issues/
andV.Bertocci,“SecuritybestpracticesfordevelopingWindowsAzure attachmentText?aid=287305.
|     |     |     |     |     |     |     |     | [80] A. Fogh, | “Row hammer, | java script | and MESI,” 2016. | Available |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | ------------ | ----------- | ---------------- | --------- |
applications,”MicrosoftCorp,2010.
athttps://dreamsofastone.blogspot.com/2016/02/row-hammer-java-script-
[54] M.Kayaalp,N.Abu-Ghazaleh,D.Ponomarev,andA.Jaleel,“Ahigh-
and-mesi.html.
| resolution | side-channel |     | attack | on last-level | cache,” | in Annual | Design |     |     |     |     |     |
| ---------- | ------------ | --- | ------ | ------------- | ------- | --------- | ------ | --- | --- | --- | --- | --- |
AutomationConference(DAC),2016. [81] V.KirianskyandC.Waldspurger,“Speculativebufferoverflows:Attacks
[55] “3DNow!technologymanual,”2000. Availableathttps://www.amd.com/ anddefenses,”arXivpreprintarXiv:1807.03757,2018.
|     |     |     |     |     |     |     |     | [82] C.Canella,J.VanBulck,M.Schwarz,M.Lipp,B.VonBerg,P.Ortner, |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------------------- | --- | --- | --- | --- |
system/files/TechDocs/21928.pdf.
F.Piessens,D.Evtyushkin,andD.Gruss,“Asystematicevaluationof
| [56] “mmap(2)—Linuxmanualpage.” |     |     |     | Availableathttps://man7.org/linux/ |     |     |     |     |     |     |     |     |
| ------------------------------- | --- | --- | --- | ---------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
transientexecutionattacksanddefenses,”inUSENIXSecuritySymposium,
man-pages/man2/mmap.2.html.
2019.
| [57] “pthread | mutex | lock(3p) | — Linux | manual | page.” | Available | at https: |                                                                  |     |     |     |     |
| ------------- | ----- | -------- | ------- | ------ | ------ | --------- | --------- | ---------------------------------------------------------------- | --- | --- | --- | --- |
|               |       |          |         |        |        |           |           | [83] M.Lipp,D.Gruss,R.Spreitzer,C.Maurice,andS.Mangard,“Armaged- |     |     |     |     |
//man7.org/linux/man-pages/man3/pthread mutex lock.3p.html. don:Cacheattacksonmobiledevices,”inUSENIXSecuritySymposium,
| [58] “taskset(1)—Linuxmanualpage.” |     |     |     | Availableathttps://man7.org/linux/ |     |     |     |     |     |     |     |     |
| ---------------------------------- | --- | --- | --- | ---------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
2016.
man-pages/man1/taskset.1.html.
|     |     |     |     |     |     |     |     | [84] G.Saileshwar,S.Kariyappa,andM.Qureshi,“Bespokecacheenclaves: |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------------------- | --- | --- | --- | --- |
[59] A.Arcangeli,I.Eidus,andC.Wright,“Increasingmemorydensityby
Fine-grainedandscalableisolationfromcacheside-channelsviaflexible
usingKSM,”inProceedingsofthelinuxsymposium,2009. set-partitioning,” in International Symposium on Secure and Private
[60] M.-M.Bazm,T.Sautereau,M.Lacoste,M.Sudholt,andJ.-M.Menaud, ExecutionEnvironmentDesign(SEED),2021.
“Cache-basedside-channelattacksdetectionthroughIntelcachemoni- [85] V. Kiriansky, I. Lebedev, S. Amarasinghe, S. Devadas, and J. Emer,
toringtechnologyandhardwareperformancecounters,”inInternational
“DAWG:Adefenseagainstcachetimingattacksinspeculativeexecu-
ConferenceonFogandMobileEdgeComputing(FMEC),2018.
|     |     |     |     |     |     |     |     | tion | processors,” in Annual | IEEE/ACM | International | Symposium on |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ---------------------- | -------- | ------------- | ------------ |
[61] P.Pessl,D.Gruss,C.Maurice,M.Schwarz,andS.Mangard,“DRAMA: Microarchitecture(MICRO),2018.
ExploitingDRAMaddressingforcross-CPUattacks,”inUSENIXSecurity [86] “Security best practices for side channel resistance.” Available at
Symposium,2016. https://www.intel.com/content/www/us/en/developer/articles/technical/
[62] G. Saileshwar, C. W. Fletcher, and M. Qureshi, “Streamline: a fast, software-security-guidance/best-practices/security-best-practices-side-
flushlesscachecovert-channelattackbyenablingasynchronouscollu-
channel-resistance.html.
sion,” in ACM International Conference on Architectural Support for [87] B.C.Vattikonda,S.Das,andH.Shacham,“Eliminatingfinegrained
ProgrammingLanguagesandOperatingSystems(ASPLOS),2021. timersinXen,”inACMworkshoponCloudcomputingsecurityworkshop,
| [63] D.M.Gordon,“Asurveyoffastexponentiationmethods,”J.Algorithms, |     |     |     |     |     |     |     | 2011. |     |     |     |     |
| ------------------------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- |
1998. [88] R.Martin,J.Demme,andS.Sethumadhavan,“Timewarp:Rethinking
[64] R.L.Rivest,A.Shamir,andL.Adleman,“Amethodforobtainingdigital timekeepingandperformancemonitoringmechanismstomitigateside-
signaturesandpublic-keycryptosystems,”Commun.ACM,1978. channel attacks,” in Annual International Symposium on Computer
[65] T.Elgamal,“Apublickeycryptosystemandasignatureschemebased Architecture(ISCA),2012.
ondiscretelogarithms,”IEEETrans.Inf.Theor.,2006.
[66] K.ZhangandX.Wang,“PeepingTomintheneighborhood:Keystroke
eavesdroppingonmulti-usersystems.,”inUSENIXSecuritySymposium,
2009.
[67] D.X.Song,D.A.Wagner,andX.Tian,“Timinganalysisofkeystrokes
andtimingattacksonssh.,”inUSENIXSecuritySymposium,2001.
[68] M.Kurth,B.Gras,D.Andriesse,C.Giuffrida,H.Bos,andK.Razavi,
“NetCAT:Practicalcacheattacksfromthenetwork,”inIEEESymposium
onSecurityandPrivacy(S&P),2020.
[69] D.Wang,A.Neupane,Z.Qian,N.B.Abu-Ghazaleh,S.V.Krishnamurthy,
| E. J. | Colbert, | and P. Yu, | “Unveiling | your | keystrokes: | A   | cache-based |     |     |     |     |     |
| ----- | -------- | ---------- | ---------- | ---- | ----------- | --- | ----------- | --- | --- | --- | --- | --- |
side-channelattackongraphicslibraries.,”inNetworkandDistributed
SystemSecuritySymposium(NDSS),2019.
| [70] C. Canella, | D.  | Genkin, | L. Giner, | D.  | Gruss, | M. Lipp, | M. Minkin, |     |     |     |     |     |
| ---------------- | --- | ------- | --------- | --- | ------ | -------- | ---------- | --- | --- | --- | --- | --- |
D.Moghimi,F.Piessens,M.Schwarz,B.Sunar,etal.,“Fallout:Leaking
| data | on Meltdown-resistant |     | CPUs,” | in  | ACM SIGSAC | Conference | on  |     |     |     |     |     |
| ---- | --------------------- | --- | ------ | --- | ---------- | ---------- | --- | --- | --- | --- | --- | --- |
ComputerandCommunicationsSecurity(CCS),2019.
| [71] “SpectrePoC.” |     | Availableathttps://github.com/crozone/SpectrePoC. |     |     |     |     |     |     |     |     |     |     |
| ------------------ | --- | ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

APPENDIX
A. The Leakage Rate of Spectre v1
TABLE I: The leakage rate of Spectre v1 when using
Prefetch+Reload and Prefetch+Prefetch, respectively, normal-
ized to Flush+Reload.
Desktopprocessors Serverprocessors
Model Core Core XeonPlatinum XeonPlatinum
i7-6700 i7-7700K 8124 8151
(3.4GHz) (4.2GHz) (3.0GHz) (3.4GHz)
Prefetch+Reload 1.61 2.40 1.57 1.64
Prefetch+Prefetch 3.02 3.94 2.01 2.08
B. The Meltdown Gadget
#define encode(x, b) ((((x) >> (b * 8)) & 0xff))
#define SPACING 4096
char mem[8][SPACING * 256];
uint64 t secret = *(uint64 t*)secret addr;
memaccess(mem[0] + encode(secret , 0) * SPACING);
memaccess(mem[1] + encode(secret , 1) * SPACING);
memaccess(mem[2] + encode(secret , 2) * SPACING);
memaccess(mem[3] + encode(secret , 3) * SPACING);
memaccess(mem[4] + encode(secret , 4) * SPACING);
memaccess(mem[5] + encode(secret , 5) * SPACING);
memaccess(mem[6] + encode(secret , 6) * SPACING);
memaccess(mem[7] + encode(secret , 7) * SPACING);
Listing 1: The example Meltdown gadget where an access to
the 64-secret is followed by eight secret encoding operations.
