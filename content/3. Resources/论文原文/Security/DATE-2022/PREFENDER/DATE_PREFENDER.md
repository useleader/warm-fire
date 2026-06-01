---
publish: true
---

1
P : A Prefetching Defender against Cache
REFENDER
Side Channel Attacks as A Pretender
Luyi Li, Jiayi Huang, Member, IEEE, Lang Feng∗, Member, IEEE, Zhongfeng Wang∗, Fellow, IEEE
Abstract—Cache side channel attacks are increasingly alarm- prefetching [12], [13], etc. However, these countermeasures
inginmodernprocessorsduetotherecentemergenceofSpectre either incur performance overhead, or have limited scope of
and Meltdown attacks. A typical attack performs intentional
security, such as only defending against the attacks conducted
cache access and manipulates cache states to leak secrets by
cross-core, so they failed to benefit both security and perfor-
observing the victim’s cache access patterns. Different counter-
measures have been proposed to defend against both general mance.
andtransientexecutionbasedattacks.Despitetheireffectiveness, Inthispaper,weproposeanapproachtodefeatingthecache
theymostlytradesomelevelofperformanceforsecurity,orhave
side channel attacks while maintaining or even improving the
restricted security scope. In this paper, we seek an approach to
performance. During the attack, the attacker obtains the cache
enforcing security while maintaining performance. We leverage
the insight that attackers need to access cache in order to state changes made by the victim by accessing the cache.
manipulate and observe cache state changes for information If the access patterns of both the attacker and the victim
leakage.Specifically,weproposePREFENDER,asecureprefetcher can be learned, the processor can prefetch the data that can
that learns and predicts attack-related accesses for prefetching
furtherchangethecachestatetoconfusetheattacker.Besides,
the cachelines to simultaneously help security and performance.
effective prefetching can help performance if the prefetcher is
Our results show that PREFENDER is effective against several
cache side channel attacks while maintaining or even improving able to predict the access patterns of the benign programs.
performance for SPEC CPU 2006 and 2017 benchmarks. We propose PREFENDER, a prefetching defender to defeat
Index Terms—Security, Cache Side Channel Attacks, cache side channel attacks while preserving performance ben-
Prefetcher. efits for benign programs. Specifically, three low-cost designs
are proposed, which are called Scale Tracker (ST), Access
Tracker(AT),andRecordProtector(RP).ScaleTrackerisable
I. INTRODUCTION
toprefetchthedatathatthevictimmayaccess,bytrackingthe
Over the last few decades, continuing optimization of mi-
target address calculation history of the memory instructions.
croarchitecturehasledtoadramaticincreaseinitscomplexity,
Access Tracker can learn the cache access patterns of the
whichmightunfortunatelybeaccompaniedbymanypotential
attackersandprefetchdataforconfusion,eveniftheattackers
security vulnerabilities. As a result, the cache side channel
perform intentional random accesses. Record Protector can
attacks [1], [2] become serious threats to modern processors.
linkScaleTrackerandAccessTrackertopreventnoisyinstruc-
For example, it is possible for Spectre [3] and Meltdown [4]
tions and accesses from affecting PREFENDER, and further
attacks to steal almost any data in the memory, by leveraging
enhance the robustness of PREFENDER. Furthermore, effec-
vulnerabilities of the out-of-order execution and the specula-
tive prefetching of PREFENDER also maintains or improves
tive execution. More seriously, these two attacks can threaten
performance. The contributions of this work are as follows:
most of the modern commercial processors from Intel, AMD,
and ARM.Lots ofvariants of cacheside channelattacks have • PREFENDER is proposed, where a novel address predic-
also been found in recent years [5], so the defense methods tion and a noise preventing approaches for prefetching are
are urgently needed to enforce the security of the processors. proposed. PREFENDER can prevent wide range of general
Cache side channel attacks exploit the cache state changes access-based cache timing side channel attacks including
for information leakage [6]. For example, the attacker can both single-core and cross-core attacks, while maintaining
infer the cache footprint of the victim program by the time the performance.
differences between cache hits and cache misses when ac- • A new approach to analyzing cache access patterns is pro-
cessing the data [3], [4]. Different countermeasures have been posed. Scale Tacker and Access Tacker are designed to
proposedforeithergeneralortransientexecutionbasedattacks realize the runtime analysis for effective prefetching.
through isolation [7], conditional speculation [8], stateless • Anapproachisproposedtoprotect PREFENDER frombeing
mis-speculative cache accesses [9], noise injection [10], [11], affected by the noisy memory instructions and accesses. To
realize this, Record Protector is designed to link the scale
LuyiLi,LangFengandZhongfengWangarewiththeSchoolofElectronic tracker and the access tracker to help identify the cache
ScienceandEngineering,NanjingUniversity,Nanjing,Jiangsu210023,China.
accesses from the attackers.
E-mail:luyli@smail.nju.edu.cn,{flang,zfwang}@nju.edu.cn.JiayiHuangis
withtheDepartmentofElectricalandComputerEngineering,UCSantaBar- • The detailed experiments show the effectiveness and the
bara,SantaBarbara,California93106-9010USA.E-mail:jyhuang@ucsb.edu. robustnessfordefeatingcachesidechannelattacks.Besides,
∗The corresponding authors. †This work was partially supported by Na-
tional Natural Science Foundation of China (Grant No. 62204111) and
PREFENDER alsobringstheperformanceimprovement,and
ShuangchuangProgramofJiangsuProvince(GrantNo.JSSCBS20210003). is highly compatible with other prefetchers.
3202
luJ
31
]RA.sc[
1v65760.7032:viXra

2
For the following sections, Section II introduces the back- this cacheline. For example, assuming the cacheline size is 64
ground and the threat model. The related work is discussed bytes,ifthevictimloadsasecret-dependentdataarray[s×64]
in Section III, and the details of PREFENDER are proposed in in phase 2, where s is the secret. During phase 3, array[768]
Section IV. Then the experiments are described in Section V. will be accessed with a cache hit; the attacker can infer the
Finally, Section VI concludes the paper. secret is s=768/64=12.
|     |     |     |     |     |     |     |     | Compared | with | Flush+Reload, |     | Evict+Reload |     | mainly | differs |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ---- | ------------- | --- | ------------ | --- | ------ | ------- |
II. BACKGROUNDANDTHREATMODEL in the way of phase 1. In Evict+Reload, the attacker loads
|     |       |              |         |     |     |     |     | some irrelative |     | data to   | evict           | the cachelines |     | instead      | of flush |
| --- | ----- | ------------ | ------- | --- | --- | --- | --- | --------------- | --- | --------- | --------------- | -------------- | --- | ------------ | -------- |
| A.  | Cache | Side Channel | Attacks |     |     |     |     |                 |     |           |                 |                |     |              |          |
|     |       |              |         |     |     |     |     | instructions.   | In  | contrast, | in Prime+Probe, |                |     | the attacker | and the  |
Cache side channel attacks are to detect the cache state victim do not share memory pages. Therefore, the attacker
changes caused by the victim’s memory accesses and fur- has its own data which maps to the same cache sets with the
ther infer the sensitive information of the victim from these victim’s data. In phase 1, the attacker evicts the cachelines
changes. In a cache side channel attack, a round of attack is by loading its own data. In phase 2, the victim accesses its
|           |      |             |         |        |     |                  |     | data and | evicts | the attacker’s |     | data. | In phase | 3,  | the attacker |
| --------- | ---- | ----------- | ------- | ------ | --- | ---------------- | --- | -------- | ------ | -------------- | --- | ----- | -------- | --- | ------------ |
| typically | made | up of three | phases. | During |     | the first phase, | the |          |        |                |     |       |          |     |              |
attacker initializes the cache states. For example, the attacker re-accesses its data and detects if there is a high latency, i.e.,
usually uses flush instructions to invalidate the cachelines or a cache miss. This cache miss can reveal the victim’s secret.
loads irrelative data to evict the original cachelines. Then, Thethreeattackssharethesamekeyidea,whichistoleverage
in the second phase, the attacker does nothing but wait for the access latency to identify the secrets.
| the      | victim | to be executed.   | During | the     | execution, | the       | victim    |                |     |     |     |     |     |     |     |
| -------- | ------ | ----------------- | ------ | ------- | ---------- | --------- | --------- | -------------- | --- | --- | --- | --- | --- | --- | --- |
| accesses | its    | data and causes   |        | changes | in the     | cache. In | the last  |                |     |     |     |     |     |     |     |
|          |        |                   |        |         |            |           |           | B. Prefetching |     |     |     |     |     |     |     |
| phase,   | the    | attacker measures |        | which   | cache      | state is  | different |                |     |     |     |     |     |     |     |
Itiswidelyknownthatthememorywallisoneofthemajor
| from | the initialized | state | and | therefore | deduces | what | data the |     |     |     |     |     |     |     |     |
| ---- | --------------- | ----- | --- | --------- | ------- | ---- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
victim has accessed. bottlenecks of modern processors. One approach to reducing
|     |     |                |     |     |              |     |     | the memory   | access  | latency |      | is prefetching, |       | which       | refers to |
| --- | --- | -------------- | --- | --- | ------------ | --- | --- | ------------ | ------- | ------- | ---- | --------------- | ----- | ----------- | --------- |
|     |     |                |     |     |              |     |     | predictively | loading | data    | into | the             | cache | in advance. | If the    |
|     |     | Attacker Data  |     |     | Victim Data  |     |     |              |         |         |      |                 |       |             |           |
Flush Attacker Data Access   Victim Data Access processor requests the data later, it will encounter a cache
Flush+Reload hit and the access latency is reduced. This technique is usu-
Attacker Victim Attacker ally implemented by the hardware module named prefetcher.
|     |     |     |     |     |     |     |     | Such typical | examples |          | include  | Tagged | Prefetcher |       | [15], Stride |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | -------- | -------- | -------- | ------ | ---------- | ----- | ------------ |
|     |     |     |     |     |     |     |     | Prefetcher   | [16],    | Feedback | Directed |        | Prefetcher | [17], | Address      |
Cachelines Low Latency Correlation Based Prefetcher [18], [19], etc.
Evict+Reload
|     | Attacker |     |     | Victim |     | Attacker |     |           |         |      |               |     |     |          |           |
| --- | -------- | --- | --- | ------ | --- | -------- | --- | --------- | ------- | ---- | ------------- | --- | --- | -------- | --------- |
|     |          |     |     |        |     |          |     | C. Threat | Model   |      |               |     |     |          |           |
|     |          |     |     |        |     |          |     | We refer  | to work | [20] | to categorize |     | the | attacks. | The cache |
Cachelines Low Latency timing side channel attacks that are access-based (types 2 and
Prime+Probe
|     |          |     |     |        |     |          |     | 4 [20])     | are included | in      | our        | threat    | model,   | which    | contains all |
| --- | -------- | --- | --- | ------ | --- | -------- | --- | ----------- | ------------ | ------- | ---------- | --------- | -------- | -------- | ------------ |
|     | Attacker |     |     | Victim |     | Attacker |     |             |              |         |            |           |          |          |              |
|     |          |     |     |        |     |          |     | the attacks | described    |         | in Section | II-A.     | Besides, |          | both single- |
|     |          |     |     |        |     |          |     | core and    | cross-core   | attacks | are        | included. |          | In these | attacks, the |
attackerisabletomodifythestatesofanycachelines(usually
Cachelines High Latency the eviction cachelines) and measure their access latencies.
|     | Phase 1 |     | Phase 2 |     |     | Phase 3 |     |     |     |     |     |     |     |     |     |
| --- | ------- | --- | ------- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Thedataattheevictioncachelinesareeithersharedorconflict
Fig.1. TheexamplesofFlush+Reload,Evict+Reload,andPrime+Probe.The
|     |     |     |     |     |     |     |     | between | the victim | and | the | attacker. | Besides, |     | the attacker |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ---------- | --- | --- | --------- | -------- | --- | ------------ |
secretcanberevealedbytheonlylow(orhigh)latencyevictioncacheline.
|              |            |               |            |            |         |               |           | needs and | is able    | to access  |          | multiple | eviction | cachelines | and       |
| ------------ | ---------- | ------------- | ---------- | ---------- | ------- | ------------- | --------- | --------- | ---------- | ---------- | -------- | -------- | -------- | ---------- | --------- |
|              |            |               |            |            |         |               |           | leverages | the timing | difference |          | between  | their    | access     | latencies |
| One          | kind       | of widely     | used       | cache side | channel | attacks       | is the    |           |            |            |          |          |          |            |           |
|              |            |               |            |            |         |               |           | to infer  | the secret | of the     | victim1. |          |          |            |           |
| timing-based |            | attack, where | the        | cache      | states  | (hit or       | miss) can |           |            |            |          |          |          |            |           |
| be           | identified | by access     | latencies. | Figure     |         | 1 illustrates | three     |           |            |            |          |          |          |            |           |
attacks, including Flush+Reload [2], Evict+Reload [14], and III. RELATEDWORK
| Prime+Probe |     | [6]. Take    | Flush+Reload |         | as an      | example, | which   | is       |              |         |         |        |     |          |          |
| ----------- | --- | ------------ | ------------ | ------- | ---------- | -------- | ------- | -------- | ------------ | ------- | ------- | ------ | --- | -------- | -------- |
|             |     |              |              |         |            |          |         | A. Cache | Side         | Channel | Attacks |        |     |          |          |
| based       | on  | page sharing | between      | the     | attacker   | and the  | victim. |          |              |         |         |        |     |          |          |
|             |     |              |              |         |            |          |         | Cache    | side channel |         | attack  | is one | of  | the most | powerful |
| In phase    | 1,  | the attacker | flushes      | all the | cachelines | that     | may be  |          |              |         |         |        |     |          |          |
accessed by the victim. Each cacheline is called an eviction micro-architectural side channel attacks, where the attacker
|            |       |                  |          |             |        |          |        | can directly | detect | the       | cache | states   | and obtain | accurate    | timing |
| ---------- | ----- | ---------------- | -------- | ----------- | ------ | -------- | ------ | ------------ | ------ | --------- | ----- | -------- | ---------- | ----------- | ------ |
| cacheline, |       | and they compose |          | an eviction | set.   | In phase | 2, the |              |        |           |       |          |            |             |        |
|            |       |                  |          |             |        |          |        | information  | for    | inferring | the   | secrets. | Many       | researchers | have   |
| victim     | loads | the data         | that are | related     | to the | secrets, | which  | is           |        |           |       |          |            |             |        |
also called secret-dependent data. In phase 3, the attacker ac- studied various types of effective attack methods.
cessestheevictionsetandmeasurestheaccesslatencyofeach
1Aseachcachelineisnotnecessarilyaccessedmultipletimesinourthreat
| eviction | cacheline. | If  | the attacker | detects | a   | low latency, | i.e., |     |     |     |     |     |     |     |     |
| -------- | ---------- | --- | ------------ | ------- | --- | ------------ | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
model,theattackinworksPrefetch-guard[10],PrODACT[21],andReuse-
a cache hit, the secret might be inferred from the address of trap[11]isoutofthescope.

3
TABLEI
COMPARISONSWITHRELATEDWORKINTHREATMODEL,APPROACH,ANDPERFORMANCEOVERHEAD.
Conditional NDA SpecShield InvisiSpec SafeSpec MuonTrap SpecPref Catalyst StealthMem DAWG CEASER RPcache SHARP
Speculation[8] [22] [23] [9] [24] [25] [26] [27] [28] [7] [29] [30] [31] PREFENDER
ThreatModels SpeculativeExecutionAttacks CacheTimingSideChannelAttacks
Approaches SpeculationRestriction ShadowStructurestoHoldSpeculativeData CachePartition NewCacheReplacementPolicy Prefetch
| Performance |     |     |     |     |     |     |     |     |     |     |     |     |     | -1.69% |     |
| ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------ | --- |
Overhead 13%-54% 11%-125% 10%-73% 21%-72% -3% 4% 1.17% 0.70% 5.90% 15% 1% 0.30% 0% /-6.28%
Kosher [32] firstly mentioned that the timing difference may lead to high overhead if they fail to accurately detect
in the cache can be exploited to extract cryptographic se- thedangerousloads.Anothercategory,suchasInvisiSpec[9],
crets.Osvik[6]proposedEvict+TimeandPrime+Probemeth- SafeSpec[24]andMuonTrap[25],designsashadowstructure
ods to attack the AES algorithm [33]. In 2014, Yarom [2] to temporarily hold the data brought by speculative loads dur-
proposed a more powerful and more fine-grained method, ing transient execution, but they require many modifications
calledFlush+Reload.Thismethodutilizestheflushinstruction to the existing hardware systems. Although SafeSpec [26]
supported by some architectures, for example, clflush in achieves a 3% performance improvement by avoiding cache
x86. Moreover, since it is based on shared memory, it has pollution, its threat model is attacks on transient speculative
much lower noise and finer granularity, e.g., a single cache- execution,whicharedifferentfromourthreatmodeloncache
line. The Flush+Reload has more variants, one of which is timing side channel attacks. SpecPref [26] also aims at specu-
Evict+Reload [14]. The Evict+Reload is applicable to devices lative execution vulnerabilities and prefetchers, but the role of
that do not support a flush instruction because it replaces the the prefetchers in SpecPref is the source of the data leakage
flush behavior with the cache eviction. instead of the way of defense, which is a different threat
| Research | on  | cache side | channel | attacks |     | continues | to spring | model. |     |     |     |     |     |     |     |
| -------- | --- | ---------- | ------- | ------- | --- | --------- | --------- | ------ | --- | --- | --- | --- | --- | --- | --- |
up,especiallyafterSpectre[3]andMeltdown[4]attackswere
|     |     |     |     |     |     |     |     | The above | defenses |     | only | prevent data | leakage | caused | by  |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | -------- | --- | ---- | ------------ | ------- | ------ | --- |
reported. These attacks exploit one of the most important transient execution. They are ineffective in defending against
| microarchitectural |     | optimizations, |     | speculation, |     | to get | sensitive |     |     |     |     |     |     |     |     |
| ------------------ | --- | -------------- | --- | ------------ | --- | ------ | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
othertraditionalcachesidechannelattacks.Forthetraditional
data. They and their variants show that many critical mi- ones, some new cache policies were introduced. Catalyst [27]
| croarchitectural |        | components, |        | including | Branch | Target   | Buffer |                |         |      |            |            |               |           |     |
| ---------------- | ------ | ----------- | ------ | --------- | ------ | -------- | ------ | -------------- | ------- | ---- | ---------- | ---------- | ------------- | --------- | --- |
|                  |        |             |        |           |        |          |        | and StealthMem |         | [28] | partition  | the cache  | into          | different | re- |
| (BTB) [3],       | Return | Stack       | Buffer | (RSB)     | [34],  | Floating | Point  |                |         |      |            |            |               |           |     |
|                  |        |             |        |           |        |          |        | gions for      | private | data | and shared | resources, | respectively. |           | For |
Unit(FPU)[35],Page-tableEntry[5],IntelSGXenclave[36], Catalyst [27], software modifications are needed. DAWG [7]
mayinadvertentlyleaktheirinternalstates,includingpotential
|     |     |     |     |     |     |     |     | achieves | a higher | granularity, |     | which | dynamically | partitions |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | -------- | ------------ | --- | ----- | ----------- | ---------- | --- |
secretswhilerunning.However,eveniftheseattacksarebased
|     |     |     |     |     |     |     |     | cache ways | to  | avoid | cache | sharing | among | different | secu- |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ----- | ----- | ------- | ----- | --------- | ----- |
ondifferenthardwarecomponents,mostofthemstillleavethe rity domains. However, these methods require programmers
secretsinthecacheandusecachesidechannelattackssuchas
|     |     |     |     |     |     |     |     | to rewrite | the | source | codes | to flag | the sensitive |     | data. In |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ------ | ----- | ------- | ------------- | --- | -------- |
Flush+Reload,Evict+Reload,Prime+Probe,etc.,asmentioned contrast, the key idea of CEASER [29] and RPcache [30]
| before,toextracttheinformation.Therefore, |         |       |         |            |     | PREFENDER | has         |                 |     |     |       |         |           |          |     |
| ----------------------------------------- | ------- | ----- | ------- | ---------- | --- | --------- | ----------- | --------------- | --- | --- | ----- | ------- | --------- | -------- | --- |
|                                           |         |       |         |            |     |           |             | is to randomize |     | the | cache | mapping | algorithm | in order | to  |
| a broad                                   | defense | scale | because | it is able | to  | defend    | against the |                 |     |     |       |         |           |          |     |
preventtheattackerfromevictingthecache.SHARP[31]also
cachesidechannelattacksthatexploitthetimingdifferenceof designsanewcachereplacementpolicytopreventtheeviction
| cache access | latency, |     | including | both | traditional | and | transient |           |      |         |     |           |             |     |          |
| ------------ | -------- | --- | --------- | ---- | ----------- | --- | --------- | --------- | ---- | ------- | --- | --------- | ----------- | --- | -------- |
|              |          |     |           |      |             |     |           | and flush | from | forcing | out | dedicated | cachelines. | It  | requires |
execution based ones. operating system support to handle interrupts generated by
alarmcountersanddoesnotdefendagainstsingle-coreattacks
B. Microarchitectural Defenses in the private cache. Indicated in Table I, almost all the
|      |                 |     |      |      |          |     |           | approaches | for | cache | timing | side channel | attacks | pay | some |
| ---- | --------------- | --- | ---- | ---- | -------- | --- | --------- | ---------- | --- | ----- | ------ | ------------ | ------- | --- | ---- |
| Many | countermeasures |     | have | been | proposed |     | to defend |            |     |       |        |              |         |     |      |
against cache side channel attacks, including software and level of performance for the security strength, or are not able
|          |             |     |          |          |     |      |            | to defeat | general | cache | side | channel attacks. | To  | sum | up, it is |
| -------- | ----------- | --- | -------- | -------- | --- | ---- | ---------- | --------- | ------- | ----- | ---- | ---------------- | --- | --- | --------- |
| hardware | approaches. |     | Software | defenses | are | more | compatible |           |         |       |      |                  |     |     |           |
withcurrentplatforms,buttheymaynotfundamentallydefeat alwaysachallengingtasktodesignbothefficientandeffective
|              |     |      |           |      |             |     |           | defenses | for both | security | and | performance. |     |     |     |
| ------------ | --- | ---- | --------- | ---- | ----------- | --- | --------- | -------- | -------- | -------- | --- | ------------ | --- | --- | --- |
| the attacks, | and | they | can incur | high | performance |     | overhead. |          |          |          |     |              |     |     |     |
Therefore, microarchitectural defenses are further proposed in Besides the above studies with different approaches from
many studies. PREFENDER, there are also multiple studies using prefetchers
The comparisons between PREFENDER and related work for defense, as summarized in Table II. Prefetch-guard [10],
are shown in Tables I and II. Cache side channel attacks PrODACT [21] and Reuse-trap [11] propose several methods
can be combined with transient speculative execution for data to detect the spy and leverage prefetching to obfuscate the
leakage, such as Spectre [3] attacks. To mitigate cache side spy based on previously recorded information, sharing the
channel attacks caused by transient execution, some of the same idea with PREFENDER. However, their threat model is
prior work restricts speculation by constraining the execution different from ours. They focus on covert channel attacks.
of speculative loads, such as Conditional Speculation [8], One key feature their defenses are based on is that the at-
NDA [22] and SpecShield [23]. They seek to identify the tacker needs to access one cacheline multiple times. As this
dangerous load instructions that can be potentially exploited assumption is not included in our threat model, they cannot
by attackers and then delay their execution until all the past defeat the targeting attacks of this paper. In addition, the
instructions are guaranteed to be safe. However, this method attackerinourthreatmodelmightaccessthecachesrandomly

4
TABLEII
COMPARISONSWITHRELATEDWORKUSINGPREFETCHING.(“-”STANDSFORTHATTHEINFORMATIONISNOTMENTIONEDINTHECORRESPONDING
WORK.)
Prefetch-guard[10] PrODACT[21] Reuse-trap[11] Disruptive BITP[13] PREFENDER
Prefetching[12]
Access-BasedCacheAttacks(Types2and4[20])
Flush+√Reload
|              |                  |     |     | √   |     |     | √   |               |     |     |     |     |
| ------------ | ---------------- | --- | --- | --- | --- | --- | --- | ------------- | --- | --- | --- | --- |
|              | Single-Cacheline |     |     |     |     |     |     |               |     | -   | -   | √×  |
|              | Multi-Cacheline  |     |     | ×   |     |     | ×   |               | ×   | -   | -   |     |
|              |                  |     |     | √   |     |     |     | Evict+√Reload |     |     | √   |     |
| sledoMtaerhT | Single-Cacheline |     |     |     |     |     | -   |               |     | -   | √   | √×  |
|              | Multi-Cacheline  |     |     | ×   |     |     | -   |               | ×   | -   |     |     |
|              |                  |     |     | √   |     |     | √   | Prime√+Probe  |     |     | √   | √   |
|              | Single-Cacheset  |     |     |     |     |     |     |               |     | √-  |     |     |
|              |                  |     |     |     |     |     |     |               |     |     | √   | √   |
|              | Multi-Cacheset   |     |     | ×   |     |     | ×   |               | ×   |     |     |     |
Timing-BasedCacheAttacks(Types1and3[20])
√
|     | Evict+Time |     |     | ×   |     |     | ×   |     | ×   | ×   |     | ×   |
| --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
CacheCollisionAttack
|     |             |     |     | √   |     |     | √ Single/Cross-C√oreAttacks |     |     | √   |     | √   |
| --- | ----------- | --- | --- | --- | --- | --- | --------------------------- | --- | --- | --- | --- | --- |
|     | Single-Core |     |     |     |     |     |                             |     |     |     | √×  |     |
|     |             |     |     | √   |     |     | √                           |     | √   | √   |     | √   |
Cross-Core
Considering
|     |     |     |     | ×   |     |     | ×   |     | √   | ×   | ×   | √   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
RandomAccess
Pattern
| seuqinhceT | Defense |     |     |           |     |     |           |           |     |          |           |           |
| ---------- | ------- | --- | --- | --------- | --- | --- | --------- | --------- | --- | -------- | --------- | --------- |
|            |         |     |     | Cacheline |     |     | Cacheline | Cacheline |     | Cacheset | Cacheline | Cacheline |
Granularity
|     | HandlingBenign |     |     |     |     |     |     |     |     |     | √   | √   |
| --- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|     |                |     |     | ×   |     |     | ×   |     | ×   | ×   |     |     |
NoiseAccesses
|     | NoSoftware |     |     |     |     |     |     |     |     | √   | √   | √   |
| --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|     |            |     |     | ×   |     |     | ×   |     | ×   |     |     |     |
Modification
|     |     |     |     | High |     |     | High |     | High | Low | Low | Low |
| --- | --- | --- | --- | ---- | --- | --- | ---- | --- | ---- | --- | --- | --- |
ecnamrofreP erawdraH& Hardware Oneconflictmisstracker Oneconflict Onereuse Onemarkedbitpercache BACK-INV ST+AT+RP,
andoneflushinstruction misstrackerper distancecounter set,randomizationand command detailedin
Overhead
trackerpercacheset. cacheset. percacheset. set-balancerlogic. tracker. SectionV-E.
Performance - - - 0%(SPEC2006) 1.10%(SPEC2006) 1.69%(SPEC2006)
|     | Improvement |     |     |     |     |     |     |     |     |     |     | 6.28%(SPEC2017) |
| --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------- |
to mislead the prefetchers, and this is not handled by the enhancement better than prior work through accurate runtime
studies[10],[21],[11].Moreover,notechniquesareproposed analyses and well-designed hardware prefetching strategies.
| in these | studies | to handle | the | noise | from the | benign | memory |     |     |     |     |     |
| -------- | ------- | --------- | --- | ----- | -------- | ------ | ------ | --- | --- | --- | --- | --- |
accesses. These studies need software modifications and can IV. PREFENDERDESIGN
| be intrusive. | For | hardware | consumption, |     | since | they | need | one |                  |              |                 |           |
| ------------- | --- | -------- | ------------ | --- | ----- | ---- | ---- | --- | ---------------- | ------------ | --------------- | --------- |
|               |     |          |              |     |       |      |      |     | In this section, | the overview | of the proposed | PREFENDER |
trackerforeachcacheset,thehardwareoverheadcanbehighly shown in Figure 2 is first introduced, and the details of Scale
| increased | with | the growth | of  | the cache | size. | Besides, | Reuse- |     |     |     |     |     |
| --------- | ---- | ---------- | --- | --------- | ----- | -------- | ------ | --- | --- | --- | --- | --- |
Tracker(ST),AccessTracker(AT),andRecordProtector(RP)
| trap needs | to know      | the   | victim’s | process |     | ID in | advance  | to  |                      |     |     |     |
| ---------- | ------------ | ----- | -------- | ------- | --- | ----- | -------- | --- | -------------------- | --- | --- | --- |
|            |              |       |          |         |     |       |          |     | are then elaborated. |     |     |     |
| record     | the victim’s | cache | misses,  | which   | may | cause | software |     |                      |     |     |     |
modifications. Finally, they still trade some level of perfor- Memory Stage Execute Stage
| mance | and fail | in gaining | performance |     | improvement |     | that | can |     |     |     |     |
| ----- | -------- | ---------- | ----------- | --- | ----------- | --- | ---- | --- | --- | --- | --- | --- |
Data
rreegg00::  ffvvaa,,  sscc
beachievedwithprefetching.DisruptivePrefetching[12]also rreegg11::  ffvvaa,,  sscc
......
| modifies   | the prefetchers |             | to defeat     | cache | side        | channel | attacks.     |     | Controller |                    |       |          |
| ---------- | --------------- | ----------- | ------------- | ----- | ----------- | ------- | ------------ | --- | ---------- | ------------------ | ----- | -------- |
|            |                 |             |               |       |             |         |              |     |            | Calculation Buffer | Core0 | Core_n-1 |
| But it     | manipulates     | in          | a granularity |       | of cacheset |         | instead      | of  |            |                    |       |          |
|            |                 |             |               |       |             |         |              |     |            | SSTT Inst0 Inst1   |       |          |
| cacheline, | and only        | Prime+Probe |               | is    | discussed,  | so      | the security |     | Prefender  |                    |       |          |
AATT
is restricted. Meanwhile, it may cause cache pollution due to ... ... ... L1D L1I ... L1D L1I
|     |     |     |     |     |     |     |     |     | Basic Pref. | RRPP |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ---- | --- | --- |
Prot. P r o te c ted
its random prefetching policy. BITP [13] prefetches the data Acc e s s   B uffer
whenidentifyingcross-coreback-invalidation-hitsinmulticore L2
sscc00,,  BBllkkAAddddrr00
sscc11,,  BBllkkAAddddrr11
| systems. | So, it | targets | cross-core | attacks |     | but not | single-core |     |     |     |     |     |
| -------- | ------ | ------- | ---------- | ------- | --- | ------- | ----------- | --- | --- | --- | --- | --- |
......
| attacks.Incontrast,PREFENDERcanalsobeappliedtosingle- |     |     |     |     |     |     |     |     |     |     | Memory |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------ | --- |
Scale Buffer
| core attacks | as  | it is able | to filter | the | benign | memory | accesses |     |     |     |     |     |
| ------------ | --- | ---------- | --------- | --- | ------ | ------ | -------- | --- | --- | --- | --- | --- |
in the single-core executions. Both BITP and PREFENDER Fig.2. Theoveralldesignarchitectureofoursystem.
| improve | performance, |     | and PREFENDER |     | achieves |     | higher | im- |     |     |     |     |
| ------- | ------------ | --- | ------------- | --- | -------- | --- | ------ | --- | --- | --- | --- | --- |
A. Overview
provement.
Compared with related work, PREFENDER is a completely According to Section II-A, three phases need to be per-
hardware-based and resource-efficient method without modi- formed by a cache side channel attack so the attack can be
fying any policy of speculative execution or cache in mod- defeatedbyinterferingwithoneofthephases. PREFENDER is
ern processors. It can effectively defend against the multi- designed in each L1Dcache for interfering with the attacks by
cacheline (cacheset) access-based cache attacks, as well as prefetching the eviction cachelines. Specifically, PREFENDER
single-coreandcross-coreattacks.Italsoconsiderstherandom includes Scale Tacker (ST) and Access Tacker (AT) to inter-
accesses from the attacks and the noise from the benign fere with the second and third phases, respectively. Record
accesses,andthedefensegranularityiseachcacheline.Onthe Protector (RP) can further protect PREFENDER from being
premiseofensuringsecurity,itfurtherachievesaperformance interferedwithbythenoisymemoryinstructionsandaccesses,

5
and enhance the robustness. A basic prefetcher (Basic Pref. noise,andtheprefetchingisguidedbytherecordsinthescale
| in Figure | 2)  | is also | supported, | such | as the | Tagged | or Stride | buffer. |     |     |     |     |
| --------- | --- | ------- | ---------- | ---- | ------ | ------ | --------- | ------- | --- | --- | --- | --- |
prefetcher. The scale tracker, the access tracker, and the basic Note that ST and AT also work for cross-core attacks. An
prefetcherareabletoprefetchdata,whiletherecordprotector exampleisshowninFigure4.Inthisexample,theprogramsof
canincreasetheaccuracyofpredictingtheevictioncachelines. theattackerandthevictimareondifferentcoreswithdifferent
Notethatthebasicprefetchercanonlyhelpwithperformance, L1D caches, but they share the same last level cache (LLC).
while the scale tracker, the access tracker, and the record For ST, after the attacker flush the eviction cachelines, when
protector can enforce security and also improve performance the victim accesses the data on another core, ST will prefetch
to some extent. the additional eviction cacheline similar as Figure 3, both in
The scale tracker aims at predicting the eviction cachelines victim’s L1D cache and LLC. For phase 3, the cross-core
that might be accessed by the victim program during phase 2. attack originally can identify the only LLC hit to infer the
The prediction is based on the arithmetic calculation histories sensitiveinformation,butwithST,therearetwoLLChitsand
of the victim instructions, which are stored in the Calculation the attacker is not able to distinguish which one is accessed
Buffer. The scale tracker will predict and prefetch additional bythevictim’sprogram.ForAT,similarasthecaseofsingle-
eviction cachelines after a victim instruction loads the data core attack, AT can directly prefetch the eviction cachelines
into an eviction cacheline in phase 2. The prefetched eviction into both attacker’s L1D cache and LLC in phase 3. As the
cachelinescanmisleadtheattackersincetheattackerisunable attacker keeps accessing the eviction cachelines, AT will keep
to distinguish them from the cacheline loaded by the victim prefetching,whichcanprefetchthecachelineinLLCaccessed
instruction. An example is shown at the top of Figure 3. by the victim to L1D cache, and can directly mislead the
| The      | access | tracker      | aims       | at predicting | the       | attacker’s | access     | attacker. |     |     |     |     |
| -------- | ------ | ------------ | ---------- | ------------- | --------- | ---------- | ---------- | --------- | --- | --- | --- | --- |
| patterns | of     | the eviction | cachelines | for           | measuring |            | the access |           |     |     |     |     |
latencyduringphase3.Theaccesstrackerleveragestheinsight Victim Data  Invalid Victim Data
|     |     |     |     |     |     |     |     | Flush | Victim Data Access |     | Prefender Prefetching |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | ------------------ | --- | --------------------- | --- |
that a few load instructions are intensively used for the attack Defense against Cross-Core Flush+Reload by ST
and stores the attacker’s access patterns in the Access Buffer. Attacker’s L1DVictim’s L1D Attacker’s L1DVictim’s L1D Attacker’s L1DVictim’s L1D
|     |         |          |     |            |           |     |           | Attacker |     | Prefender | Victim Attacker |     |
| --- | ------- | -------- | --- | ---------- | --------- | --- | --------- | -------- | --- | --------- | --------------- | --- |
| An  | example | is shown | at  | the bottom | of Figure | 3,  | where the |          |     |           |                 |     |
access tracker prefetches the eviction cacheline before the Low Latency
(LLC Latency)
| attacker | accesses | it            | and measures | the | access | latency. | This can |     |     |     |     |     |
| -------- | -------- | ------------- | ------------ | --- | ------ | -------- | -------- | --- | --- | --- | --- | --- |
| also     | mislead  | the attacker. |              |     |        |          |          | LLC | LLC |     | LLC |     |
Defense against Cross-Core Flush+Reload by AT
|     |              |       |     |                    |     |                       |     | Attacker’s L1DVictim’s L1D | Attacker’s L1DVictim’s L1D |     | Attacker’s L1DVictim’s L1D   |     |
| --- | ------------ | ----- | --- | ------------------ | --- | --------------------- | --- | -------------------------- | -------------------------- | --- | ---------------------------- | --- |
|     | Victim Data  | Flush |     | Victim Data Access |     | Prefender Prefetching |     |                            |                            |     |                              |     |
|     |              |       |     |                    |     |                       |     | Attacker                   |                            |     | Victim 7 8 At 9 tac 1 k 0 er | 11  |
Defense against Flush+Reload by ST
7 4 5 611
|     | Attacker |     | Prefender | Victim |     | Attacker |     |     |     |     | 4 ,5 , 6      |     |
| --- | -------- | --- | --------- | ------ | --- | -------- | --- | --- | --- | --- | ------------- | --- |
|     |          |     |           |        |     |          |     |     |     |     | Pre fe n d er |     |
Low Latency
(L1D Latency)
|     |     |     |     |     |     |     |     | LLC | LLC |     | LLC |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Cachelines Low Latency Fig.4. Theexampleofthedefensesagainstcross-coreFlush+Reloadattacks
Defense against Flush+Reload by AT (Thenumbernearanarrowrepresentstheaccesstime,andthenumberinside
|     |          |     |     |        |     | Attacker |        | each rectangle represents | the first       | time when      | the corresponding | cacheline is |
| --- | -------- | --- | --- | ------ | --- | -------- | ------ | ------------------------- | --------------- | -------------- | ----------------- | ------------ |
|     | Attacker |     |     | Victim | 5   | 6 7      | 8 9 10 | accessed).                |                 |                |                   |              |
|     |          |     |     |        |     | 5 4 7    | 8 9 10 |                           |                 |                |                   |              |
|     |          |     |     |        |     |          |        | Since the                 | key idea of the | scale tracker, | the access        | tracker,     |
4
Cachelines Prefender Low Latency andtherecordprotectoristocorrectlylearncacheaccesspat-
Fig. 3. The example of the defenses against Flush+Reload attacks (The ternsforprefetching,effectiveprefetchingonbenignloadscan
number near an arrow represents the access time, and the number inside also improve performance while enforcing security. However,
each rectangle represents the first time when the corresponding cacheline is there are four major challenges for effective prefetching for
accessed).
PREFENDER.
Although the access tracker can interfere with phase 3 to C1. During phase 2, the victim may only access one eviction
mislead the attacker, since phase 3 is much longer than phase cacheline.Eventhoughthereareotherevictioncachelines
2, there is more noise during the phase, which may affect the that may also be accessed, they may not be simply
predictionoftheaccesstracker.Becausephase2isperformed contiguous. How to effectively predict the access pattern
bythevictim,thevictim’saccesspatternslearnedbythescale givenlimitedaccesses(evensingleaccess)ischallenging,
tracker are regarded as trusted patterns, and can help correct which we overcome with the scale tracker.
the prediction of the access tracker. The record protector is C2. During phase 3, the eviction cachelines might be ran-
designed to link the scale tracker and the access tracker to domly accessed by the attacker. This can bypass some
preventthenoisefromaffectingtheaccesstracker.Therecord prefetchers such as Stride prefetcher. Predicting the evic-
protector can record the victim’s cache access prediction of tion cachelines based on a random access pattern is
the scale tracker into the Scale Buffer. If the attacker’s access challenging, which is tackled by the access tracker.
patternintheaccessbuffermatchesapredictedvictim’scache C3. Duringphase3,theremightbenoisymemoryinstructions
accessinthescalebuffer,thecorrespondinginformationinthe executed, so that the records of the attacker’s access
access buffer is protected from being interfered with by the patterns in the access buffer are overwritten by the noisy

6
instructions. In this case, the access tracker might be makes the result to be 652, there may be another pair (e.g.,
bypassed. We tackle this problem by using the record i increments 1) to make the result as 652+128. The 128 can
protector. be sc in this calculation. Similarly, 32 and any multiples of
r
C4. During phase 3, if some non-eviction cachelines are also them like 256, 512, etc., can also be sc . Note that an access
r
accessed by the same attacking instruction, the prefetch- pattern that involves multiplications of several variables (such
ing of the access tracker can be affected by these noisy as(128i i i +32j ×16j )×(48k +imm))canalsobehandled
|     |     |     |     |     |     |     |     | 0 1 2 | 0   | 1   | 0   |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- |
accesses. We tackle this by using the record protector. by propagating the scales and the fixed values during the
| Note       | that the | challenge        | C3 is      | related to overwriting |               | the | calculations.            |     |     |                                           |     |     |     |     |
| ---------- | -------- | ---------------- | ---------- | ---------------------- | ------------- | --- | ------------------------ | --- | --- | ----------------------------------------- | --- | --- | --- | --- |
| attacker’s | recorded | access           | behaviors, | while                  | the challenge |     |                          |     |     |                                           |     |     |     |     |
| C4 is      | about    | extra misleading | behaviors. |                        |               |     |                          |     |     | TABLEIII                                  |     |     |     |     |
|            |          |                  |            |                        |               |     | Therulestocalculatefvard |     |     | andscrd.(rdisthedestinationregister;“-”is |     |     |     |     |
notapplicable.†Theruleisalsoforsubtractionwhen+isreplacedby−.
‡Theruleisalsoforshiftingwhen×isreplacedby>>or<<.)
B. Scale Tacker
|     |     |     |     |     |     |     |     |     | Conditions |     |     |     | Results |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | ------- | --- |
The prediction of the Scale Tracker (ST) is based on the Instruction Arg.a Arg.b fvars0 fvars1 fvard scrd
|     |     |     |     |     |     |     | loadrda | im m | 0 - | -   | -   | im  | m 0 | 1   |
| --- | --- | --- | --- | --- | --- | --- | ------- | ---- | --- | --- | --- | --- | --- | --- |
target address calculation of the victim load. For example, if im m (r s0) - - - N A 1
the load’s target address is calculated by 128×i+192, where i rs0 imm0 NA - NA scrs0
|     |     |     |     |     |     |     |     | rs0 | imm0 | Valid | -   | fvars0+imm0 |     | 1   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- | ----- | --- | ----------- | --- | --- |
is an integer variable, the target address can only be 192, 320, addrdab† r s 0 r s 1 V al id V a l i d fvars0 + fvars1 N A
|     |     |     |     |     |     |     |     | r s | 0 r s | 1 N A | V a l i d | N   | A   | s cr s0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | ----- | --------- | --- | --- | ------- |
|     |     |     |     |     |     |     |     | rs0 | rs1   | Valid | NA        | NA  |     | scrs1   |
448, etc. After the virtual address is translated to the physical rs0 rs1 NA NA NA min(scrs0,scrs1)
a d d r e s s, i f p a d d r i s th e t a r g e t p h y s i c a l a d d r e ss f o r t h i s ti m e , i t r s 0 i m m 0 N A - N A s c r s × i m m 0
|     |     |     |     |     |     |     |     | r s | i m m | V a l i d   | -         | f v a       | × im m   | 0 1 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | ----------- | --------- | ----------- | -------- | --- |
|     |     |     |     |     |     |     |     | r s | 0 r s | 0 V a l i d | V a l i d | f v a r s 0 | × f va 0 | N A |
c a n b e d e d u c e d th a t p a d d r - 1 2 8 , p a d d r , p a d d r+ 1 2 8 , e tc ., m a y mulrdab‡ 0 1 r s 0 rs 1
|     |     |     |     |     |     |     |     | r s | 0 r s | 1 N A | V a l i d | N   | A   | s c r s 0 × f v a r s 1 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | ----- | --------- | --- | --- | ----------------------- |
a l s o b e a c c e s s e d b y t h i s l o a d i f t h e y a r e i n t h e s a m e p a g e . r s 0 r s 1 V a l i d N A N A f v a r s 0 × s c r s 1
|     |     |     |     |     |     |     |     | r s | 0 r s | 1 N A | N A | N   | A   | s c r s × s c r s |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | ----- | --- | --- | --- | ----------------- |
In this way, we can predict the access pattern of the victim Otherwise - - - - 0 1 1
NA
| instruction | in phase | 2. The | main goal | is to learn | the 128 | as in |     |     |     |     |     |     |     |     |
| ----------- | -------- | ------ | --------- | ----------- | ------- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
the example, which is called the scale in our work. The proposed rules for calculating sc (and fva , which
|                                                       |     |     |     |     |     |     |          |           |     |       |             | r   |       | r           |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | -------- | --------- | --- | ----- | ----------- | --- | ----- | ----------- |
|                                                       |     |     |     |     |     |     | can help | calculate | sc  | ) are | illustrated | in  | Table | III. When a |
| Thetargetaddressofaloadisusuallystoredintheregisters, |     |     |     |     |     |     |          |           |     | r     |             |     |       |             |
so the scale tracker needs to track how the register values are program is started, the fixed and scale values are initialized to
calculated.Thiscanberealizedbyrecordingallthecalculation NA and 1, respectively. During the execution of the program,
|     |     |     |     |     |     |     | the fixed | value | and scale | of  | the | destination | register | rd are |
| --- | --- | --- | --- | --- | --- | --- | --------- | ----- | --------- | --- | --- | ----------- | -------- | ------ |
historyofeachregister,butitcanincurunacceptablehardware
consumption. Therefore, only addition and multiplication (in- calculatedaccordingtotheoperandandthepropagatedvalues
cluding subtraction and shifting) are considered, as they are of the source registers.
widely used in the calculation, and their calculation history For data movement instructions, if an immediate number is
can be tracked by using only two values for each register. loaded to rd, fva is set to the number. If a value is loaded
rd
|     |     |     |     |     |     |     | frommemorytord,fva |     |     | andsc |     | arereinitializedsincewe |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- | ----- | --- | ----------------------- | --- | --- |
We use two values to track the history for each register r: rd rd
a fixed value fva and a scale sc , which are stored in the conservativelyregardtheloadedvalueasanunknownvariable.
|     |     | r   | r   |     |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
calculationbuffers.Thefva isneededtohelptrackthescale, For addition, when fva is calculated by one immediate
|     |     |     | r   |     |     |     |     |     |     | rd  |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
and it records the calculation result if all the calculations of number and one register rs , if rs ’s fva is NA, sc is
|     |     |     |     |     |     |     |     |     |     | 0   |     | 0   | rs0 | rd  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
register r only depend on constant values (immediate num- the same as sc since adding the immediate number as the
rs0
bers). If the value of r depends on some variables such as the offsethasnoeffectonthescale.Iffva rs0 isvalid,fva rd isthe
loaded memory values, fva is not applicable (NA). addition of fva and the immediate number since both are
|           |        |         | r         |        |       |         |               |      | rs0    |     |            |     |         |             |
| --------- | ------ | ------- | --------- | ------ | ----- | ------- | ------------- | ---- | ------ | --- | ---------- | --- | ------- | ----------- |
|           |        |         |           |        |       |         | fixed values. | When | adding | two | registers, |     | if only | one of them |
| The cache | access | pattern | predicted | by the | scale | tracker |               |      |        |     |            |     |         |             |
mainlydependsonthescalesc .Usually,arrayaccessaddress has a valid fixed value, the scale of the destination register is
r
in a loop is calculated as base+scale×i (e.g., base+128×i), the same as the scale of the source register without a valid
wherebaseisthebaseaddressandiisanintegervariable.The fixed value. If neither of the source registers has a valid fixed
above calculation will be propagated through some registers, value,thescaleofthedestinationregistercanbetheminimum
and the final calculation result is stored in a register and scale of the two registers. The reason is that when the values
used as the target address of a load to access the array. One oftwo registersare added, bothscales can be usedas thenew
task of the scale tracker is to track the scale by propagating scale. Using the minimum one can reduce the possibility of
|            |              |      |           |              |        |     | making the | scale | larger | than | a page. |     |     |     |
| ---------- | ------------ | ---- | --------- | ------------ | ------ | --- | ---------- | ----- | ------ | ---- | ------- | --- | --- | --- |
| scales and | fixed values | from | registers | to registers | during | the |            |       |        |      |         |     |     |     |
calculations. Assuming the target address addr is stored in For multiplication, the calculations of fva and sc are
|     |     |     |     |     |     |     |     |     |     |     |     |     | rd  | rd  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
register r, we can obtain the scale sc related to r. When one similartothoseofaddition,excepttheconsiderationofmulti-
r
load is executed even for a single time, the scale tracker can plicativefactorsduetomultiplication.Ifanyothercalculations
predict that the nearby cachelines (addr±sc ) may also be areinvolved,tobeconservative,thedestinationregisterofthe
r
accessed by the same load. This is the access pattern tracked calculation is reinitialized.
by the scale tracker. When an instruction load rd imm(rs) or the equivalent
The scale tracker can also support more complicated ac- instruction is executed, assuming the target address for this
cess patterns, such as 128×i+32×j+imm, where i and j are time is addr′, then addr′±sc are the candidate prefetching
rs
variables as the indices and imm is an immediate number. addresses. Once sc is larger than the cacheline size and
rs
In this example, given an imm, if there is a pair of i and j smallerthanthepagesize,thecandidateaddressesthatarenot

7
currently in the L1Dcache are prefetched. We conservatively the attacker’s memory accesses in phase 3 are only associated
assume that all the load instructions might be the victim’s withafewloadinstructions.Thiscanhelpthelearningofthe
instructions that are vulnerable. Therefore, the scale tracker access patterns by recording the access history of each load
is applied to all the instructions. Although all loads instruction separately.
load
| are considered, |     | the defense | is performed |     | when | the target |     |     |     |     |     |     |     |
| --------------- | --- | ----------- | ------------ | --- | ---- | ---------- | --- | --- | --- | --- | --- | --- | --- |
addresses are calculated by addition and multiplication and Instruction Address 0x8008 Instruction Address 0x8018
|     |     |     |     |     |     |     |     |     | Load  0 x 1 000 |     | Load  0 x 1 500 |     | ① Buffer Allocation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --------------- | --- | ------------------- |
the scales are larger than the cacheline size. This implies that ② Entry Updating
|     |     |     |     |     |     |     |     |     | ·· ·· · · |     | ·· ·· · · |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- | --------- | --- | --- |
the prefetching is performed when the loads are likely from Load 0x2800 ······ ③ DiffMin Updating
phase 2 of the attacks instead of arbitrary loads, and this Load 0x1C00 ······ ④ Data Prefetching
canmitigatethepotentialcachepollution.Forimplementation, Valid Buffer[0] Buffer[1] Buffer[n - 1]
|     |     |     |     |     |     |     |     |     |     | ①   |     | ①   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
since the scale tracker prefetches data in the same page, the InstAddr 1 0x8008 1 0x8010 0x8018 1 ···
③
|                                     |     |     |     |         |     |            |     | DiffMin | 1 0x600 0x300 |     | 1 0 0x100 |     | 1 ··· |
| ----------------------------------- | --- | --- | --- | ------- | --- | ---------- | --- | ------- | ------------- | --- | --------- | --- | ----- |
| bitwidthforstoringandcalculatingfva |     |     |     | r andsc | r   | canbesmall |     |         |               |     |           |     |       |
②
| (Section          | V-E).            |     |     |     |     |     |     |               | 1 0x1000 |     | 1 0xA000 0x1500 |         | 1 ··· |
| ----------------- | ---------------- | --- | --- | --- | --- | --- | --- | ------------- | -------- | --- | --------------- | ------- | ----- |
|                   |                  |     |     |     |     |     |     |               | 1 0x1F00 |     | 1 0 0xA100      |         | 1 ··· |
|                   |                  |     |     |     |     |     |     |               | 1 0x1600 |     | 1 0 0xA200      | ······1 | ···   |
| 1   l o a d   r 0 | ,   4 ( s p )    |     |     |     | sc  | fva |     | B u f f e r   |          |     |                 |         |       |
r 0 = s e c r e t’s address 1 0 x 2 80 0 1   0 0 x A 3 0 0 1 · · ·
| 2   l o a d   r 1 | ,   0 ( r 0 ) |       |             | r0  | 1   | NA  |     | E n t r i e s |             | ②   |                 |     |         |
| ----------------- | ------------- | ----- | ----------- | --- | --- | --- | --- | ------------- | ----------- | --- | --------------- | --- | ------- |
|                   |               | r 1 = | s e c r e t |     |     |     |     |               | 1 0 x 1 C 0 | 0   | 1   0 0 x A 4 0 | 0   | 1 · · · |
|                   |               |       |             | r1  | 1   | NA  |     |               |             |     |                 |     |         |
3 load r2, arr_addr r2=arr_addr (imm) ··· ··· ··· ··· ··· ···
|                  |     |                 |     | r2  | 1     | arr_addr |            |                             |          |     |          |     |        |
| ---------------- | --- | --------------- | --- | --- | ----- | -------- | ---------- | --------------------------- | -------- | --- | -------- | --- | ------ |
| 4 load r3, 0x200 |     | r3=0x200 (imm)  |     |     |       |          |            |                             | 0        |     | 1 0 · ·· |     | 1 · ·· |
|                  |     |                 |     | r3  | 1     | 0x200    |            |                             |          |     |          |     |        |
| 5 mul r4, r1, r3 |     | r4=secret×0x200 |     |     |       |          |            |                             |          |     |          |     | ···    |
|                  |     |                 |     | r4  | 0x200 | NA       | Cachelines |                             |          |     |          |     |        |
| 6 add r5, r2, r4 |     | r5=arr_addr+r4  |     |     |       |          |            |                             |          |     |          |     |        |
|                  |     |                 |     |     |       | NA       |            |                             | ④ 0x1C00 |     |          |     |        |
| 7 load r6, 0(r5) |     | r6=array[r4]    |     | r5  | 0x200 |          |            | 0x1000                      |          |     |          |     |        |
|                  |     | (a)             |     |     |       | (b)      | Fig.6.     | Anexampleoftheaccessbuffer. |          |     |          |     |        |
Fig.5. (a)Apseudocodeexampleforaccessingarray[secret×0x200],
|          |                                                      |     |     |     |     |     | For | the | access tracker, | there | is a set | of access | buffers, each |
| -------- | ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --------------- | ----- | -------- | --------- | ------------- |
| wherearr | addrisanimmediatenumberthatrepresentstheaddressofthe |     |     |     |     |     |     |     |                 |       |          |           |               |
firstelementinarray. of which is associated with a instruction and records the
load
(b)Thescales(sc)andthefixedvalues(fva)intheaccessbuffer,whereeach
|              |           |        |                  |          |       |                | target | block   | addresses | accessed | by the         | associated | load. The  |
| ------------ | --------- | ------ | ---------------- | -------- | ----- | -------------- | ------ | ------- | --------- | -------- | -------------- | ---------- | ---------- |
| value is set | according | to the | instruction with | the same | color | and the values |        |         |           |          |                |            |            |
|              |           |        |                  |          |       |                | access | buffers | can help  | the      | access tracker | learn      | the access |
indicatedbythearrows.
patternsoftheassociatedloadinstructions.Foreachload,the
An example is shown in Figure 5. The pseudo code in access pattern is estimated as a stride access—an arithmetic
sequencewithaconstantdifference,whichisestimatedasthe
| Figure 5(a) | accesses | array[secret×0x200] |     |     | at  | line 7. For |     |     |     |     |     |     |     |
| ----------- | -------- | ------------------- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- |
Lines 1-2, the instructions load the secret’s address and the minimumdifferencebetweenblockaddressesintheassociated
| secret from | the | memory | to r0 and r1, | respectively. |     | Therefore, | buffer. |     |     |     |     |     |     |
| ----------- | --- | ------ | ------------- | ------------- | --- | ---------- | ------- | --- | --- | --- | --- | --- | --- |
the values of r0 and r1 are regarded as variables and fva of The microarchitecture of the access buffer is shown in
them are NA. Lines 3-4 load the immediate numbers to r2 Figure 6. Each buffer maintains a register for storing the
|               |       |     |           |        |        |          | instruction |     | address InstAddr |     | of the associated |     | load. For each |
| ------------- | ----- | --- | --------- | ------ | ------ | -------- | ----------- | --- | ---------------- | --- | ----------------- | --- | -------------- |
| and r3, which | makes | the | fva of r2 | and r3 | be arr | addr and |             |     |                  |     |                   |     |                |
0x200, respectively. Next, line 5 multiplies r1 (secret) and entry of a buffer, the block address BlkAddr accessed by the
r3 (0x200) and stores the result to r4. According to Table III, associated load is recorded. There is also a register in each
since r1’s fva is NA and r3’s fva is 0x200, the sc of r4 is buffer,whichstorestheminimumdifferenceDiffMinbetween
0x200×1, and fva of r4 is NA. For line 6, the r2 and r4 are two block addresses among all the entries. Each register or
added to r5, which makes sc of r4 directly propagated to r5 entry of an access buffer has a valid bit for indicating if the
since r2 has a valid fva. Finally, when the load of line 7 is data is valid or not. All valid bits are set to 0 upon the reset
executed, the scale tracker will prefetch the data at (target of the buffer. Note that we discuss the conceptual idea in
|             |      |       |                                  |     |     |     | this | section. | For implementation, |     | we do | not need | to store a |
| ----------- | ---- | ----- | -------------------------------- | --- | --- | --- | ---- | -------- | ------------------- | --- | ----- | -------- | ---------- |
| address)±sc | r5 , | which | are arr addr+secret×0x200±0x200. |     |     |     |      |          |                     |     |       |          |            |
Inthiscase,assumesecretis12atthistime,thereareatleast complete block address in each entry (Section V-E).
2 more eviction cachelines in the cache, which can mislead Four stages are involved in the flow of the access tracker:
the attacker to get the wrong secret value 11 or 13. 1 Buffer Allocation: When a load accesses the cache
|     |     |     |     |     |     |     | each | time, | its instruction | address | (the | value in | the program |
| --- | --- | --- | --- | --- | --- | --- | ---- | ----- | --------------- | ------- | ---- | -------- | ----------- |
counter)iscomparedwiththeInstAddrstofindtheassociated
| C. Access | Tracker |     |     |     |     |     |     |     |     |     |     |     |     |
| --------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
accessbuffer,whichisthenactivated.Ifthereisnoassociated
Forphase3,theattackerneedstotimealltheevictioncache- buffer, an empty buffer is allocated to this load. If there is
linestogettheaccesslatencies.Therefore,AccessTacker(AT)
|     |     |     |     |     |     |     | no  | empty | nor associated | buffer, | one buffer | is selected | by the |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | -------------- | ------- | ---------- | ----------- | ------ |
is proposed to interfere with phase 3 to further mislead the least recently used (LRU) replacement policy for allocation.
attacker.Thegoaloftheaccesstrackeristolearntheattacker’s For example, in Figure 6, when the load with InstAddr
access pattern in phase 3, and prefetch the eviction cachelines 0x8008 accesses the cache, associated Buffer[0] is activated.
before the attacker times them. In contrast, when the load with InstAddr 0x8018 accesses
However,accordingtochallengeC2,attackersmaytimethe the cache, no buffer is associated, so Buffer[1] is selected to
eviction cachelines in a random order to bypass prefetchers allocate this load by LRU policy.
such as Stride prefetcher. This increases the difficulty of 2 Entry Updating: In the activated buffer, if the BlkAddr
learningtheaccesspatterns.Itisfoundthatincommoncases, of the accessed data is not recorded, a new entry is selected

8
to store this BlkAddr. If all the entries are occupied, LRU is attacker’s load can be occupied by a noisy instruction. Ac-
applied to find the entry for this BlkAddr. cording to the access tracker’s policy, this noisy instruction
3 DiffMin Updating: The access tracker calculates willinitializethebufferandevicttheattacker’sinformation.
DiffMin of a buffer when the buffer is activated and the In this case, the access tracker may fail to prefetch the
number of valid entries of this buffer surpasses a threshold eviction cachelines to defeat the attack.
(such as 4). The number of entries of each buffer is set • Challenge C4: In phase 3, if the attacker accesses the non-
to be small (such as 8) to reduce the hardware complexity. eviction cachelines, the access tracker will calculate wrong
DiffMin can be used to estimate the difference between each DiffMin. These accessed cachelines are also noise for the
two addresses to be accessed by the attacker in phase 3. access tracker. For example, the BlkAddrs stored in the
4 Data Prefetching: After the number of valid entries access buffers are 0x8000, 0x8200, 0x8400, and 0x8600,
in a buffer surpasses a threshold (such as 4), each time which are all the eviction cachelines. The DiffMin is 0x200
this buffer is activated, candidate prefetching addresses are in this case. However, once a non-eviction cacheline with
calculated.IfBlkAddr′ istheblockaddressofthecurrentload, BlkAddr=0x8100isaccessedbythesameload,DiffMinwill
the candidate prefetching addresses are BlkAddr′± DiffMin. be changed to 0x100. This can mislead the access tracker to
Then, the access tracker checks if these addresses exist in the prefetch the cachelines that are not the eviction cachelines,
activated buffer, and prefetches one of them that is not in the and the attacks can bypass the access tracker’s defense.
activated buffer nor in the L1DCache. For example, assuming To tackle the above two challenges, Record Protector (RP)
thecachelinesizeis256bytes,inFigure6,thecoloredcache- is proposed, which can link the scale tracker and the access
lines’blockaddressesarerecordedinthebufferentries,where trackertoincreasetherobustnessof PREFENDER,asshownin
the cachelines and their corresponding block addresses have Fig 2. When a victim load accesses the cache, assuming reg-
the same color. When load with InstAddr 0x8008 accesses ister r stores the target address, the scale tracker will prefetch
the cache, Buffer[0] is activated. The latest block address the data according to sc . Meanwhile, the record protector
r
0x1C00isstoredinthebuffer,andDiffMinisupdatedto0x300 will store sc and the block address BlkAddr of this access’s
r r
as it is calculated by |0x1F00-0x1C00|. At this moment, the targetaddresstothescalebuffer.Eachtimewhentheattacker’s
access tracker predicts that the eviction cachelines are 0x1C00 load accesses the cachelines for the timing measurement in
+ 0x300×k,wherekisaninteger.TheredmarginsinFigure6 phase 3, the block address BlkAddr’ of this access is checked
indicatetheevictioncachelinesthatarenotcurrentlyaccessed. with all the sc and BlkAddr pairs in the scale buffer, where
i i
In this case, the candidate addresses are 0x1C00±0x300. As i is the index of the entry. If (BlkAddr’-BlkAddr )%sc =0,
i i
0x1C00+0x300 is already in the buffer, 0x1C00-0x300 is finally it is estimated that this access is the access to the eviction
prefetched by the access tracker (indicated by the arrow near cachelines.Therefore,theassociatedaccessbufferisprotected
4). so that it cannot be directly replaced by LRU, and this tackles
In this way, the access tracker can learn the access pat- challenge C3. Meanwhile, upon protection, the prefetching is
terns of the actively executed load, and prefetch the data guidedbysc
i
butnotDiffMin,whichcanprotectPREFENDER
accordinglytomisleadtheattacker.Weconservativelyassume from being affected by the non-eviction cacheline records in
that all the loads might be leveraged by the attacker, so the the access buffer, and challenge C4 is tackled.
access tracker is applied to all of them. The possibility of An example of the flow of the record protector is shown
associatingthebufferwiththeattacker’sloadcanbeincreased in Figure 7, and the detailed policy of the record protector is
by increasing the number of the buffers. Note that the access elaborated as follows, where 3 stages are involved.
tracker (or the scale tracker) only prefetch one cacheline for 1 ScaleRecording:Whenthevictimaccessestheeviction
each load execution in order to reduce the risk of incurring cacheline,thescaletrackerusesthescalesc′ inthecalculation
performance overhead and avoid additional hardware com- buffer for prefetching (Section IV-B). At the same time, the
plexity. Although all loads are considered, the access tracker record protector records sc′ and the block address BlkAddr’
finallyprefetcheswhenaloadisfrequentlyexecutedinatime of the target address to the scale buffer, as shown in step 1
interval, which is the access pattern of the attack’s phase 3. of Figure 7(a). The records in the scale buffer represent the
Therefore,prefetchinghappenswhentheloadsarelikelyfrom pattern of the possible eviction cachelines. They can guide
the attacks instead of arbitrary loads, and the potential cache the access tracker to avoid being affected by noisy accesses,
pollution is mitigated. which is discussed in the later stages.
However, one pattern might be a subset of another pattern.
If so, to reduce the redundancy, only the pattern with the
D. Record Protector
larger scale is recorded. For example, in the step 1 of
The access tracker can defeat the side channel attacks by Figure7(a),r1iscalculatedby0x400×i+0x200,andthetarget
prefetching the data that are predicted to be accessed by address is 0x1000 for this time. In this case, sc′ =0x400 and
the attackers in phase 3. However, in practice, two scenarios BlkAddr’=0x1000, so the pattern is S′ ={... 0x0c00, 0x1000,
(challenges C3 and C4) might bypass the access tracker. 0x1400, ...}. For Entry 1 of the scale buffer, sc =0x100 and
1
• Challenge C3: In phase 3, between two eviction cacheline BlkAddr 1 =0x2000, so the pattern is S 1 ={... 0x1f00, 0x2000,
accessesoftheattacker’sload,theremightbeotherbenign 0x2100, ...}. Since S′ ⊂ S (which means sc′ > sc ), all the
1 1
memory access instructions executed, which are noise for possible eviction cachelines in S′ are also in S . In this case,
1
the access tracker. The access buffer associated with the onlyS′ needstobekeptforreducingredundancy,andEntry1

9
Attacker Instruction Attacker Instruction policyinSectionIV-C.So,thenoisyaccessesoftheattacker’s
| ① Scale Recording  |     |     |     | ······ |     |     | ······ |     |     |     |     |     |     |     |     |
| ------------------ | --- | --- | --- | ------ | --- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
② Protection Status Updating  Load 0x2600 Load 0x2200 load have much lower effects on the defense.
|     |     |     | Load 0x2400 |     |     | Load 0x2c00 |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | ----------- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
③ Protected Prefetching Anexampleisthestep 3 ofFigure7(a).Theloadaccesses
|     |     |     |     | ······ |     |     | ······ |         |         |          |     |     |                   |     |           |
| --- | --- | --- | --- | ------ | --- | --- | ------ | ------- | ------- | -------- | --- | --- | ----------------- | --- | --------- |
|     |     |     |     |        |     |     |        | address | 0x2400. | Although |     |     | in the associated |     | buffer is |
Victim Instruction ②  Valid Buffer[0] Valid Buffer[0] DiffMin
| Lo a | d   ( r1) |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ---- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
InstAddr 1 0x8008 InstAddr 1 0x8008 0x200, the prefetching is performed based on the hit scale
| · · | ·· · · |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
DiffMin 1 0x800 0x200 DiffMin 1 0x200 0x400 in the scale buffer. As a result, one of the candidate
  r1=0x400×i+0x200
|   =0x1000 |     |     |     | 0x400 |     |     | 0x400 |     |     |     |     |     |     |     |     |
| --------- | --- | --- | --- | ----- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
Protected  1 Protected  1 addresses0x2400±0x400notintheaccessbufferisprefetched.
| sc       | fva | Scale |     | 0x1000 | Scale |     | 0x1000 |        |           |        |       |             |     |          |          |
| -------- | --- | ----- | --- | ------ | ----- | --- | ------ | ------ | --------- | ------ | ----- | ----------- | --- | -------- | -------- |
|          |     |       |     |        |       |     |        | If the | hit scale | buffer | entry | is replaced |     | later so | that the |
| r0 0x200 | NA  |       |     |        |       |     |        |        |           |        |       |             |     |          |          |
1 ··· 1 ··· BlkAddr’nolongerhitsthescalebuffer,theassociatedaccess
| r1 0x400 | NA  |     |     |        |     |     |        |     |     |     |     |     |     |     |     |
| -------- | --- | --- | --- | ------ | --- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
| ··· ···  | ··· |     | 1   | 0x2600 |     | 1   | 0x2600 |     |     |     |     |     |     |     |     |
1 0x2400 1 0x2400 buffer’sprotectedscalewillbecheckedinstead.Ifthereisahit
| Ca lculation B | uffer | Buffer  |     |     | Buffer  |     |     |     |     |     |     |     |     |     |     |
| -------------- | ----- | ------- | --- | --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Entries 0 0 Entries 1 0x2200 like the case in Figure 7(b), the prefetching is still performed
| Valid sc | BlkAddr① |     | 0   | 0   |     | 1   | 0x2c00 |     |     |     |     |     |     |     |     |
| -------- | -------- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
accordingtothehitscale0x400.Foraprotectedaccessbuffer,
| 0 1 0x160 | 0x1300 |     | ··· | ··· |     | ··· | ··· |     |     |     |     |     |     |     |     |
| --------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
    A c c e s s   B u f fer A c c e s s   B u f fer o n c e t h e n um b e r o f th e pr e fe t ch i n g u s in gt h e hi t sc al e e x c e e d s
| 1 1 0 x 10 0 | 0 x 20 0 0 |     |     |     | ②   |     | ②   |     |     |     |     |     |     |     |     |
| ------------ | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
0 x 4 0 0 0 x 1 0 0 0 Buffer Pro te c t e d   F l a g 0 1 Buffer Pro te c t e d   F l a g 1 0 a t h re s h o ld or t h e b u ff er s ta y s u n t ou c h ed f o r a t im e t hr e s h o l d ,
| ··· ··· ··· | ··· |            |     |     |            |     |     |     |     |     |     |     |     |     |     |
| ----------- | --- | ---------- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|             |     | Cachelines |     |     | Cachelines |     |     |     |     |     |     |     |     |     |     |
n -1 0 the access buffer is set back to unprotected status, as shown
③
| Scale Buffer |         |        | 0x2400  | 0x2600     |            | 0x2200 | 0x2c00 ③   | in Figure | 7(b).       |     |        |           |     |      |            |
| ------------ | ------- | ------ | ------- | ---------- | ---------- | ------ | ---------- | --------- | ----------- | --- | ------ | --------- | --- | ---- | ---------- |
|              |         | (a)    |         |            |            |        | (b)        |           |             |     |        |           |     |      |            |
|              |         |        |         |            |            |        |            | In        | conclusion, | the | record | protector | can | help | the access |
| Fig. 7. An   | example | of the | flow of | the record | protector. | (The   | underlined |           |             |     |        |           |     |      |            |
instructionsaccesstheevictioncachelines;Eachredmarginisatthecandidate tracker tackle challenges C3 and C4 by protecting the access
address by the access tracker’s policy; Each red block is at the candidate buffersandperformingprefetchingbasedonthescaletracker’s
addressbytherecordprotector’spolicy.)
|               |         |     |           |            |         |          |             | information,        |       | respectively. | We        | still conservatively |            | assume         | that      |
| ------------- | ------- | --- | --------- | ---------- | ------- | -------- | ----------- | ------------------- | ----- | ------------- | --------- | -------------------- | ---------- | -------------- | --------- |
|               |         |     |           |            |         |          |             | all the             | loads | might         | be the    | victim’s             | and        | the attacker’s | in-       |
|               |         |     |           |            |         |          |             | structions,         | so    | the record    | protector |                      | is applied | to all         | of them.  |
| is replaced   | by sc′  | and | BlkAddr’. | In detail, |         | assuming | the scale   |                     |       |               |           |                      |            |                |           |
|               |         |     |           |            |         |          |             | For implementation, |       |               | since the | access               | buffer     | stores         | the block |
| and the block | address |     | related   | to the     | current | load     | are sc′ and |                     |       |               |           |                      |            |                |           |
|               |         |     |           |            |         |          |             | addresses,          | the   | bitwidth      | for       | the modulus          |            | calculation    | can be    |
)%min(sc′,sc
| BlkAddr’,when(BlkAddr’−BlkAddr |           |         |      |        |      |         | i )=0for  |       |        |                 |     |          |       |     |     |
| ------------------------------ | --------- | ------- | ---- | ------ | ---- | ------- | --------- | ----- | ------ | --------------- | --- | -------- | ----- | --- | --- |
|                                |           |         |      |        | i    |         |           | small | enough | to be practical |     | (Section | V-E). |     |     |
| Entry i of                     | the scale | buffer, | only | if sc′ | > sc | , Entry | i will be |       |        |                 |     |          |       |     |     |
i
sc′
| updated by |     | and BlkAddr’. |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ---------- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
V. EVALUATION
| 2 Protection |            | Status | Updating: |     | In phase | 3,       | each time |                 |     |       |     |     |     |     |     |
| ------------ | ---------- | ------ | --------- | --- | -------- | -------- | --------- | --------------- | --- | ----- | --- | --- | --- | --- | --- |
|              |            |        |           |     |          |          |           | A. Experimental |     | Setup |     |     |     |     |     |
| when the     | attacker’s | load   | accesses  | the | cache,   | BlkAddr’ | of this   |                 |     |       |     |     |     |     |     |
load’s target address is checked with all the records (sc and In our experiments, gem5 simulator [37] is used, where
i
BlkAddr ) in the scale buffer. If BlkAddr’ matches one of the the baseline configuration contains a 2GHz x86 out-of-order
i
|          |           |       |       |                   |     |     |          | CPU | with a | 32KB L1Icache, |     | a 64KB | L1Dcache, | and | a 2MB |
| -------- | --------- | ----- | ----- | ----------------- | --- | --- | -------- | --- | ------ | -------------- | --- | ------ | --------- | --- | ----- |
| recorded | patterns, | which | means | (BlkAddr’-BlkAddr |     |     | )%sc =0, |     |        |                |     |        |           |     |       |
|          |           |       |       |                   |     |     | i i      |     |        |                |     |        |           |     |       |
wesayBlkAddr’hitsthescalebuffer.Whenacacheaccesshits L2cache.Thereare4miss-statushandlingregisters(MSHRs),
eachofwhichcanmergeatmost20requeststothesameline.
| the scale      | buffer, | it is estimated |     | that the   | load | of this | access       | is           |     |           |         |           |         |         |       |
| -------------- | ------- | --------------- | --- | ---------- | ---- | ------- | ------------ | ------------ | --- | --------- | ------- | --------- | ------- | ------- | ----- |
|                |         |                 |     |            |      |         |              | For security |     | analysis, | we test | different | Spectre | attacks | using |
| the attacker’s | load    | in phase        | 3.  | Therefore, | upon | the     | hit, the hit |              |     |           |         |           |         |         |       |
sc and BlkAddr are copied to the protected scale registers in Flush+Reload, Evict+Reload and Prime+Probe. Challenges
| i   |     | i   |     |     |     |     |     |       |              |       |     |       |          |                 |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | ------------ | ----- | --- | ----- | -------- | --------------- | --- |
|     |     |     |     |     |     |     |     | C1-C4 | are involved | based | on  | these | attacks. | For performance |     |
theassociatedaccessbuffer,andthisassociatedaccessbufferis
markedasprotected.Withtherecordprotector,theLRUpolicy analysis, SPEC CPU 2006 and 2017 benchmark suites [38],
|               |                 |     |            |          |             |       |           | [39]                                          | are evaluated. | Based |     | upon the | baseline, | PREFENDER |     |
| ------------- | --------------- | --- | ---------- | -------- | ----------- | ----- | --------- | --------------------------------------------- | -------------- | ----- | --- | -------- | --------- | --------- | --- |
| in the access | tracker         |     | for access | buffer   | replacement |       | is only   |                                               |                |       |     |          |           |           |     |
|               |                 |     |            |          |             |       |           | canincludedifferentbasicprefetchers,including |                |       |     |          |           | PREFENDER |     |
| applied to    | the unprotected |     | access     | buffers. | By          | using | the scale |                                               |                |       |     |          |           |           |     |
buffertopredictwhichloadisfromtheattacker,andprotectits only, PREFENDER with a Tagged prefetcher [15], and with a
|     |     |     |     |     |     |     |     | Stride | prefetcher | [40]. | Note | that the | priority | of PREFENDER’s |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ---------- | ----- | ---- | -------- | -------- | -------------- | --- |
associatedaccessbuffer,theaccessbufferwillnotbereplaced
by noisy loads. In this way, challenge C3 is tackled. prefetchingishigherthanbasicprefetchersfortimelydefense.
| For example, |     | in the | step 2 | of Figure | 7(a), | load | accesses |     |     |     |     |     |     |     |     |
| ------------ | --- | ------ | ------ | --------- | ----- | ---- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
address 0x2400, which corresponds to an eviction cacheline. B. Security Evaluation
Since it hits the scale buffer, the associated access buffer is Differentsidechannelattacksareusedtoevaluatethesecu-
marked as protected by setting the “Buffer Protected Flag” as rityeffectivenessof PREFENDER,andtheresultsareshownin
| 1. Scale | 0x400 | and block | address | 0x1000 |     | are also | copied | to     |       |                |         |       |        |              |     |
| -------- | ----- | --------- | ------- | ------ | --- | -------- | ------ | ------ | ----- | -------------- | ------- | ----- | ------ | ------------ | --- |
|          |       |           |         |        |     |          |        | Figure | 8. We | first evaluate | without | noisy | memory | instructions |     |
the protected scale registers. and noisy accesses (i.e., without challenges C3 and C4), and
3 Protected Prefetching: Besides tackling challenge C3 the results are shown in Figure 8(a)-(c). For Flush+Reload,
by protecting the access buffers, challenge C4 can be tackled withoutapplyingPREFENDER,theattackercaninferthesecret
by prefetching data according to the scales in the scale buffer. valuebyobtainingtheonlycachehitwhenaccessingtheevic-
Each time a load’s target block address BlkAddr’ is stored tioncachelinesofthearrayinphase3.WhentheScaleTracker
into the access tracker, if it hits the scale buffer or the (ST)isapplied,thescaletrackerisabletointroduceadditional
protected scale, the access tracker will use the hit scale sc misleading cache hits on eviction cachelines, according to
hit
to prefetch data, i.e., the access tracker’s candidate prefetch- the calculation history. Besides, by learning the attacker’s
ing addresses are BlkAddr’±sc . Otherwise, the candidate access pattern, the Access Tracker (AT) successfully predicts
hit
prefetching addresses are calculated by the access tracker’s the accesses of the phase 3, and confuses the attacker by

10
Hit Threshold Base Prefender-ST Prefender-AT Prefender-ST+AT Prefender-AT+RP Prefender
| )elcyC( ycnetaL | 400 |        |     |     | 400 |     |        |     | 400 |     |        |     |     |     |
| --------------- | --- | ------ | --- | --- | --- | --- | ------ | --- | --- | --- | ------ | --- | --- | --- |
|                 |     | Secret |     |     |     |     | Secret |     |     |     | Secret |     |     |     |
Cache
|     | 200 |     |     |     | 200 |     |     |     | 200 |     |     |     |     | Miss |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
Cache
|     |     |     |     |     | 0   |     |     |     | 0   |     |     |     |     | Hit |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
0
|     | 50  | 6570 | 90  | 110 | 50  | 6570 | 90  | 110 |     | 50  | 6570 | 90  | 110 |     |
| --- | --- | ---- | --- | --- | --- | ---- | --- | --- | --- | --- | ---- | --- | --- | --- |
Array Index
|     |     |     |     |     |     |     | Array Index |     |     |     | Array Index |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | --- | --- | ----------- | --- | --- | --- |
(a) Flush + Reload (C1 + C2) (b) Evict + Reload (C1 + C2) (c) Prime + Probe (C1 + C2)
| )elcyC( ycnetaL | 400 |        |     |     | 400 |     |        |     | 400 |     |        |     |     |     |
| --------------- | --- | ------ | --- | --- | --- | --- | ------ | --- | --- | --- | ------ | --- | --- | --- |
|                 |     | Secret |     |     |     |     | Secret |     |     |     | Secret |     |     |     |
Cache
|     | 200 |     |     |     | 200 |     |     |     | 200 |     |     |     |     | Miss |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
Cache
|     | 0   |             |     |     | 0   |      |             |     | 0   |     |             |     |     | Hit |
| --- | --- | ----------- | --- | --- | --- | ---- | ----------- | --- | --- | --- | ----------- | --- | --- | --- |
|     | 50  | 6570        | 90  | 110 | 50  | 6570 | 90          | 110 |     | 50  | 6570        | 90  |     | 110 |
|     |     | Array Index |     |     |     |      | Array Index |     |     |     | Array Index |     |     |     |
(d) Flush + Reload (C1 + C2 + C3) (e) Evict + Reload (C1 + C2 + C3) (f) Prime + Probe (C1 + C2 + C3)
|     | 400 |     |     |     | 400 |     |     |     | 400 |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
)elcyC( ycnetaL
|     |     | Secret |     |     |     |     | Secret |     |     |     |     |     |     |     |
| --- | --- | ------ | --- | --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
Secret
Cache
|     | 200 |     |     |     | 200 |     |     |     | 200 |     |     |     |     | Miss |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
Cache
|     | 0   |             |     |     | 0   |      |             |     | 0   |     |             |     |     | Hit |
| --- | --- | ----------- | --- | --- | --- | ---- | ----------- | --- | --- | --- | ----------- | --- | --- | --- |
|     | 50  | 6570        | 90  | 110 | 50  | 6570 | 90          | 110 |     | 50  | 6570        | 90  | 110 |     |
|     |     | Array Index |     |     |     |      | Array Index |     |     |     | Array Index |     |     |     |
(g) Flush + Reload  (C1 + C2 + C4) (h) Evict + Reload (C1 + C2 + C4) (i) Prime + Probe (C1 + C2 + C4)
| )elcyC( ycnetaL | 400 |        |     |     | 400 |     |        |     | 400 |     |        |     |     |     |
| --------------- | --- | ------ | --- | --- | --- | --- | ------ | --- | --- | --- | ------ | --- | --- | --- |
|                 |     | Secret |     |     |     |     | Secret |     |     |     | Secret |     |     |     |
Cache
|     | 200 |     |     |     | 200 |     |     |     | 200 |     |     |     |     | Miss |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
Cache
|     | 0   |             |     |     | 0   |      |             |     | 0   |     |             |     |     | Hit |
| --- | --- | ----------- | --- | --- | --- | ---- | ----------- | --- | --- | --- | ----------- | --- | --- | --- |
|     | 50  | 6570        | 90  | 110 | 50  | 6570 | 90          | 110 |     | 50  | 6570        | 90  | 110 |     |
|     |     | Array Index |     |     |     |      | Array Index |     |     |     | Array Index |     |     |     |
(j) Flush + Reload (C1 + C2 + C3 + C4) (k) Evict + Reload (C1 + C2 + C3 + C4) (l) Prime + Probe (C1 + C2 + C3 + C4)
Fig. 8. The results of different attack methods with different challenges. (“PREFENDER” means that the scale tracker, the access tracker, and the record
protectorareallapplied.NotethatforPREFENDER-ST,thelatencyresultsofarrayindices64-66arethesamein(a)-(c).)
|                 |     |           |     |     |     | ST   | AT        | RP   |     |     |           |     |     |     |
| --------------- | --- | --------- | --- | --- | --- | ---- | --------- | ---- | --- | --- | --------- | --- | --- | --- |
| sehcteferP fo # | 120 |           |     |     | 120 |      |           |      | 120 |     |           |     |     |     |
|                 | 80  |           |     |     | 80  |      |           |      | 80  |     |           |     |     |     |
|                 | 40  |           |     |     | 40  |      |           |      | 40  |     |           |     |     |     |
|                 | 0   |           |     |     | 0   |      |           |      | 0   |     |           |     |     |     |
|                 | 540 |           | 560 | 580 |     | 7640 | 7660      | 7680 |     | 320 |           | 380 | 440 |     |
|                 |     | Time (µs) |     |     |     |      | Time (µs) |      |     |     | Time (µs) |     |     |     |
(a) Flush + Reload (C1 + C2) (b) Evict + Reload (C1 + C2) (c) Prime + Probe (C1 + C2)
| sehcteferP fo # | 60  |     |     |     | 60  |     |     |     | 60  |     |     |     |     |     |
| --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|                 | 40  |     |     |     | 40  |     |     |     | 40  |     |     |     |     |     |
|                 | 20  |     |     |     | 20  |     |     |     | 20  |     |     |     |     |     |
0
|     | 0   |     |     |     | 0   |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
500 520 540 560 580 600 10700 10740 10780 10820 200 260 320 380 440 500 560
|     |     | Time (µs) |     |     |     |     | Time (µs) |     |     |     | Time (µs) |     |     |     |
| --- | --- | --------- | --- | --- | --- | --- | --------- | --- | --- | --- | --------- | --- | --- | --- |
(d) Flush + Reload (C1 + C2 + C3 + C4) (e) Evict + Reload (C1 + C2 + C3 + C4) (f) Prime + Probe (C1 + C2 + C3 + C4)
Fig. 9. The number of the prefetches performed under different attack methods with different challenges. (PREFENDER-ST+AT is applied in (a)-(c), and
PREFENDER withthescaletracker,theaccesstracker,andtherecordprotectorisappliedin(d)-(f).Notethattheprefetchesoftherecordprotectorreferto
thoseoftheaccesstrackerguidedbytherecordprotector.)

11
introducingthecachehits.Whenboththescaletrackerandthe C. Performance Evaluation
accesstrackerareimplemented,theireffectsoncachelinesare While enforcing security, PREFENDER can also maintain
overlapped.Similarresultsarealsoobtainedwhenperforming
|              |         |     |              |     |     |          |            | or even | improve | performance. |     | When | the record |     | protector is |
| ------------ | ------- | --- | ------------ | --- | --- | -------- | ---------- | ------- | ------- | ------------ | --- | ---- | ---------- | --- | ------------ |
| Evict+Reload | attack. | For | Prime+Probe, |     | the | attacker | infers the |         |         |              |     |      |            |     |              |
notimplemented,theperformanceresultsofSPECCPU2006
| secret by | the  | only cache | miss.      | When | the        | scale | tracker | is          |            |       |          |      |     |          |          |
| --------- | ---- | ---------- | ---------- | ---- | ---------- | ----- | ------- | ----------- | ---------- | ----- | -------- | ---- | --- | -------- | -------- |
|           |      |            |            |      |            |       |         | benchmarks  | are        | shown | in Table | IV.  | The | results  | show the |
| applied,  | more | eviction   | cachelines | are  | prefetched | in    | phase   | 2,          |            |       |          |      |     |          |          |
|           |      |            |            |      |            |       |         | improvement | percentile |       | compared | with | the | baseline | that has |
which incurs more cache misses. When the access tracker no prefetchers. The main results are Columns 2, 6 and 10,
| is applied, | all | eviction | cachelines | are | prefetched | so  | that the |     |     |     |     |     |     |     |     |
| ----------- | --- | -------- | ---------- | --- | ---------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
where32accessbuffersareimplemented.WhenPREFENDER-
| attacker | can only | obtain | cache | hits when | accessing |     | the array. |     |     |     |     |     |     |     |     |
| -------- | -------- | ------ | ----- | --------- | --------- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
ST+ATisimplemented(Column2),theperformanceimprove-
| This also | misleads | the | attacker. | When | both | the scale | tracker |         |          |     |          |          |          |              |     |
| --------- | -------- | --- | --------- | ---- | ---- | --------- | ------- | ------- | -------- | --- | -------- | -------- | -------- | ------------ | --- |
|           |          |     |           |      |      |           |         | ment is | about 2% | on  | average, | with the | security | enforcement. |     |
andtheaccesstrackerareapplied,onlytheeffectoftheaccess
|     |     |     |     |     |     |     |     | For Columns | 6   | and 10 | where the | conventional |     | prefetchers | are |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | ------ | --------- | ------------ | --- | ----------- | --- |
tracker remains since the access tracker prefetches (phase 3) applied, PREFENDER-ST+AT can further improve the perfor-
| after the | scale | tracker (phase | 2). |     |     |     |     |     |     |     |     |     |     |     |     |
| --------- | ----- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
mancecomparedwithColumns4and8wherenoPREFENDER
When there are noisy memory instructions during phase 3 is implemented, respectively. This shows PREFENDER’s capa-
| (challenge | C3), | the access | buffers | of  | the access | tracker | can |            |             |     |         |           |     |              |     |
| ---------- | ---- | ---------- | ------- | --- | ---------- | ------- | --- | ---------- | ----------- | --- | ------- | --------- | --- | ------------ | --- |
|            |      |            |         |     |            |         |     | bility for | maintaining |     | or even | improving | the | performance. |     |
be occupied by these accesses of the noisy instructions, and Whentherecordprotectorisimplemented,theperformance
applying the access tracker only may not defeat the attack, as resultsofSPECCPU2006benchmarksareshowninTableV,
showninFigure8(d)-(f).However,whentheRecordProtector where the performance distributions are similar as Table IV.
(RP) is implemented, the access buffer associated with the Withtherecordprotector, PREFENDER alsoimprovestheper-
attacker’s load is successfully identified and protected, so the formance on average, no matter if there are basic prefetchers
access tracker is able to prefetch the eviction cachelines and or not. At the same time, not only is the security enforced,
mislead the attacker again. Similarly, without the record pro- butalsotherobustnessof PREFENDER isgreatlyimprovedby
tector, when there are noisy accesses by the attacker’s load in the record protector.
phase 3 (challenge C4), the value of DiffMin can be affected, While the performance is improved by PREFENDER on
andtheaccesstrackermaynotbeabletoprefetchtheeviction average, the impacts on different benchmarks vary. For ex-
cachelines, as shown in Figure 8(g)-(i). In contrast, when the ample, 401.bzip2, 429.mcf and 462.libquantum have the most
record protector is applied, the prefetching is guided by the speedupwith PREFENDER.Incontrast,thereisalmostnoper-
scalebufferthatcontainsthepossibleevictioncachelinesfrom
|     |     |     |     |     |     |     |     | formance | impact | on 999.specrand. |     | For | 445.gobmk, |     | 458.sjeng |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------ | ---------------- | --- | --- | ---------- | --- | --------- |
the victim, so the access tracker can again correctly prefetch and 471.omnetpp, their performance has a slight drop with
the eviction cachelines to mislead the attacker. PREFENDER. The effect of the number of the access buffers
Combining all the challenges and all the designs, the se- is also evaluated and shown in Tables IV and V. The results
indicatethatmoreaccessbuffersusuallyhelptheperformance.
| curity can | be illustrated |     | in Figure | 8(j)-(l). |     | Without | applying |     |     |     |     |     |     |     |     |
| ---------- | -------------- | --- | --------- | --------- | --- | ------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
PREFENDER, the attacker can infer the secret with the only Besides, if the buffers are more than 32, marginal improve-
one cache hit (or miss). With PREFENDER, even though all ments are obtained.
|                |     |               |          |     |       |          |         | Besides, | the | results | of the cases | newly | presented |     | in SPEC |
| -------------- | --- | ------------- | -------- | --- | ----- | -------- | ------- | -------- | --- | ------- | ------------ | ----- | --------- | --- | ------- |
| the challenges |     | are involved, | multiple |     | cache | hits (or | misses) |          |     |         |              |       |           |     |         |
are introduced, and the attack is defeated. CPU 2017 benchmarks are shown in Table VI. Similar to
SPECCPU2006,PREFENDERalsohasperformanceimprove-
| We further |                     | analyze | the insights |            | of the     | defense, | which      |                 |              |                                       |          |             |            |            |            |
| ---------- | ------------------- | ------- | ------------ | ---------- | ---------- | -------- | ---------- | --------------- | ------------ | ------------------------------------- | -------- | ----------- | ---------- | ---------- | ---------- |
|            |                     |         |              |            |            |          |            | ment, both      | with         | and without                           | the      | record      | protector. | At         | the same   |
| are shown  | in Figure           | 9,      | where        | the x-axis | represents |          | the ex-    |                 |              |                                       |          |             |            |            |            |
|            |                     |         |              |            |            |          |            | time, PREFENDER |              | canfurtherincreasetheperformancebased |          |             |            |            |            |
| ecution    | time.               | We only | show         | the part   | where      | the      | attack     | is              |              |                                       |          |             |            |            |            |
|            |                     |         |              |            |            |          |            | on the basic    | prefetchers. |                                       | Note     | that for    | some       | benchmarks | such       |
| performed. | For                 | Figure  | 9(a)-(c),    | challenges |            | C1 and   | C2 are     |                 |              |                                       |          |             |            |            |            |
|            |                     |         |              |            |            |          |            | as 510.parest   | r,           | the performance                       |          | improvement |            | is         | relatively |
| involved,  | and PREFENDER-ST+AT |         |              | is         | applied.   | One      | can notice |                 |              |                                       |          |             |            |            |            |
|            |                     |         |              |            |            |          |            | large. This     | is because   |                                       | the data | prefetched  | by         | PREFENDER  | can        |
thatthescaletrackerprefetchesasmallamountofdatashown
greatlyhelpreducethecachemissrate.Forexample,thecache
| in Figure        | 9(a)-(c), | which          | corresponds |            | to     | the data   | at array  |                                                     |      |       |              |              |         |               |            |
| ---------------- | --------- | -------------- | ----------- | ---------- | ------ | ---------- | --------- | --------------------------------------------------- | ---- | ----- | ------------ | ------------ | ------- | ------------- | ---------- |
|                  |           |                |             |            |        |            |           | missrateandthecachemisses’accesslatencyof510.parest |      |       |              |              |         |               | r          |
| indices          | 64 and    | 66 of          | the green   | curves     | in     | Figure     | 8(a)-(c). |                                                     |      |       |              |              |         |               |            |
|                  |           |                |             |            |        |            |           | in Column                                           | 2 of | Table | VI (in       | the revision | letter) |               | are 50.26% |
| After this,      | the       | access tracker |             | prefetches | more   | data       | shown     | in                                                  |      |       |              |              |         |               |            |
|                  |           |                |             |            |        |            |           | and 55.99%                                          | less | than  | that without | PREFENDER,   |         | respectively. |            |
| Figure 9(a)-(c), |           | which          | is also     | shown      | by the | orange     | curves    |                                                     |      |       |              |              |         |               |            |
| in Figure        | 8(a)-(c). | For            | Figure      | 9(d)-(f),  | all    | challenges | are       |                                                     |      |       |              |              |         |               |            |
involved, and PREFENDER with all three designs is applied. D. Analysis of Cache Miss and Defense
It is indicated that the scale tracker still prefetches several Prefetching can impact the cache miss rate and latency. We
| data. After | this, | with the | guidance | of  | the record | protector, | the |           |           |         |        |       |        |         |        |
| ----------- | ----- | -------- | -------- | --- | ---------- | ---------- | --- | --------- | --------- | ------- | ------ | ----- | ------ | ------- | ------ |
|             |       |          |          |     |            |            |     | evaluated | the total | latency | of all | cache | misses | of each | bench- |
access tracker successfully prefetches the data even with the mark, which is shown in Figure 10. Each result is normalized
| noisy instructions |     | and accesses. |     | The | corresponding |     | results are |                  |     |           |                       |     |     |     |          |
| ------------------ | --- | ------------- | --- | --- | ------------- | --- | ----------- | ---------------- | --- | --------- | --------------------- | --- | --- | --- | -------- |
|                    |     |               |     |     |               |     |             | to the baseline. |     | In Figure | 10, “PREFENDER-ST+AT” |     |     |     | have the |
showninFigure8(j)-(l).Thisfurthershowsthemechanismof
|     |     |     |     |     |     |     |     | same configuration |     | as  | Columns | 2, 6, | 10 in | Table | IV, where |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- | ------- | ----- | ----- | ----- | --------- |
the defense. the record protector is not applied. “PREFENDER” has the
In summary, by successfully defeating the attacks in the same configuration as Columns 2, 6, 10 in Table V with the
threatmodel,PREFENDERcanenforcethesecurityasthesame scale tracker, the access tracker, and the record protector. It is
as the previous work [8], [9], [23], [24]. indicatedthatthetotallatenciesofcachemissesarereducedon

12
TABLEIV
PerformanceimprovementofSPECCPU2006benchmarkswithouttherecordprotector.(†Thebasicprefetcher.)
ColumnID 1 2 3 4 5 6 7 8 9 10 11
Prefetcher PREFENDER-ST+AT Tagged PREFENDER-ST+AT(†Tagged) Stride PREFENDER-ST+AT(†Stride)
#ofAcc.Tra.Buf. 16 32 64 - 16 32 64 - 16 32 64
kramhcneB
400.perlbench 0.677% 0.679% 1.110% 0.241% 0.427% 0.588% 0.324% 0.389% 1.117% 1.065% 1.536%
401.bzip2 3.314% 3.346% 3.407% 4.428% 5.717% 5.728% 5.732% 1.777% 3.922% 3.959% 4.052%
429.mcf 6.421% 8.562% 8.585% 8.636% 12.069% 12.228% 12.237% 13.233% 14.803% 17.684% 17.653%
445.gobmk -0.025% -0.106% -0.122% 1.318% 1.164% 1.103% 1.102% 0.363% 0.433% 0.379% 0.347%
456.hmmer 0.830% 0.862% 0.891% 10.115% 10.128% 10.152% 10.158% 7.119% 6.417% 6.474% 6.512%
458.sjeng -0.354% -0.355% -0.366% -0.437% -0.613% -0.615% -0.609% -0.016% -0.300% -0.303% -0.322%
462.libquantum 6.533% 6.533% 6.532% 4.852% 6.501% 6.501% 6.501% 7.555% 9.768% 9.770% 9.773%
464.h264ref 0.269% 0.256% 0.408% 1.762% 1.707% 1.521% 1.804% 0.934% 0.724% 0.993% 0.793%
471.omnetpp -0.006% -0.006% -0.011% 0.112% 0.109% 0.109% 0.109% 0.229% 0.213% 0.213% 0.211%
473.astar 0.033% 0.398% -0.132% 0.183% 0.212% 0.415% -0.176% 0.032% 0.059% 0.474% -0.021%
483.xalancbmk 0.702% 2.840% 3.941% 11.576% 11.577% 11.952% 10.592% 2.137% 2.771% 4.952% 5.683%
999.specrand 0.000% 0.000% 0.000% 0.001% 0.001% 0.001% 0.001% 0.000% 0.000% 0.000% 0.000%
Avg. 1.533% 1.918% 2.020% 3.566% 4.083% 4.140% 3.981% 2.813% 3.327% 3.805% 3.851%
TABLEV
PerformanceimprovementofSPECCPU2006benchmarkswiththerecordprotector.(†Thebasicprefetcher.)
ColumnID 1 2 3 4 5 6 7 8 9 10 11
Prefetcher PREFENDER Tagged PREFENDER(†Tagged) Stride PREFENDER(†Stride)
#ofAcc.Tra.Buf. 16 32 64 - 16 32 64 - 16 32 64
kramhcneB
400.perlbench 0.584% 0.562% 0.585% 0.241% 0.001% 0.524% 0.545% 0.389% 1.115% 1.118% 1.116%
401.bzip2 3.129% 3.192% 3.251% 4.428% 5.621% 5.646% 5.667% 1.777% 3.828% 3.916% 3.958%
429.mcf 4.347% 5.494% 5.497% 8.636% 9.335% 9.557% 9.540% 13.233% 12.114% 12.755% 12.755%
445.gobmk -0.030% -0.066% -0.084% 1.318% 1.189% 1.171% 1.163% 0.363% 0.386% 0.347% 0.335%
456.hmmer 0.830% 0.861% 0.891% 10.115% 10.128% 10.149% 10.162% 7.119% 6.431% 6.467% 6.529%
458.sjeng -0.411% -0.428% -0.422% -0.437% -0.649% -0.660% -0.687% -0.016% -0.324% -0.337% -0.373%
462.libquantum 6.516% 6.518% 6.521% 4.852% 6.502% 6.502% 6.502% 7.555% 9.781% 9.782% 9.782%
464.h264ref 0.346% 0.300% 0.346% 1.762% 1.739% 1.806% 1.800% 0.934% 0.899% 0.812% 0.843%
471.omnetpp 0.025% 0.047% 0.058% 0.112% 0.104% 0.112% 0.106% 0.229% 0.231% 0.225% 0.230%
473.astar 0.029% 0.308% -0.139% 0.183% 0.208% 0.355% -0.182% 0.032% 0.054% 0.385% -0.027%
483.xalancbmk 0.860% 2.372% 3.822% 11.576% 11.533% 11.704% 10.624% 2.137% 3.123% 4.644% 5.628%
999.specrand 0.000% 0.000% 0.000% 0.001% 0.001% 0.001% 0.001% 0.000% 0.000% 0.000% 0.000%
Avg. 1.352% 1.597% 1.694% 3.566% 3.809% 3.905% 3.770% 2.813% 3.136% 3.343% 3.398%
TABLEVI
PerformanceimprovementofSPECCPU2017benchmarks.(†Thebasicprefetcher.)
Column ID 1 (ST+AT) 2 3 4 (ST+AT) 5 6 7 (ST+AT) 8
Prefetcher PREFENDER Tagged PREFENDER (†Tagged) Stride PREFENDER (†Stride)
# of Acc. Tra. Buf. 32 32 - 32 32 - 32 32
kramhcneB
507.cactuBSSN r 0.917% 0.874% 12.256% 12.752% 12.711% 10.707% 11.672% 11.557%
526.blender r 0.015% 0.015% 0.356% 0.302% 0.302% 0.120% 0.133% 0.133%
531.deepsjeng r -0.396% -0.379% -0.121% -0.525% -0.513% 0.000% -0.380% -0.369%
538.imagick r 5.664% 5.664% 4.240% 6.389% 6.389% 0.561% 6.292% 6.292%
541.leela r -0.072% -0.249% 0.164% 0.257% 0.120% 0.145% 0.187% 0.073%
557.xz r 0.243% 0.332% 4.015% 4.107% 4.104% 1.637% 1.873% 1.892%
510.parest r 39.738% 50.291% 44.043% 49.822% 54.617% 0.700% 35.586% 46.775%
548.exchange2 r 0.000% -0.006% 0.000% 0.000% 0.000% 0.011% -0.004% 0.015%
554.roms r 0.000% 0.000% 30.898% 30.898% 30.898% 15.797% 15.797% 15.797%
Avg. 5.123% 6.282% 10.650% 11.556% 12.070% 3.298% 7.906% 9.129%
1 .2
1 .0
0 .8
0 .6
0 .4
4 0 0 .p erlb en ch 4 0 1 .b zip 2 4 2 9 .m cf 4 4 5 .gob m k 4 5 6 .h m m er 4 5 8 .sjen g 4 6 2 .lib q u an tu m 4 6 4 .h 2 6 4 ref 4 7 1 .om n etp p 4 7 3 .astar 4 8 3 .x alan cb m k 9 9 9 .sp ecran d A vg.
yc
neta
L
ssi
M
dezila
mr
o
N
B aselin e T agged S trid e
P refen d er-S T + A T P refen d er-S T + A T (T agged ) P refen d er-S T + A T (S trid e)
P refen d er P refen d er (T agged ) P refen d er (S trid e)
Fig.10. ThenormalizedtotallatencyofallcachemissesofL1Dcache.
S T (T agged ) S T (S trid e)
A T (T agged ) A T (S trid e)
1 0 7 R P (T agged ) R P (S trid e)
1 0 5
1 0 3
1 0 1
4 0 0 .p erlb en ch 4 0 1 .b zip 2 4 2 9 .m cf 4 4 5 .gob m k 4 5 6 .h m m er 4 5 8 .sjen g 4 6 2 .lib q u an tu m 4 6 4 .h 2 6 4 ref 4 7 1 .om n etp p 4 7 3 .astar 4 8 3 .x alan cb m k 9 9 9 .sp ecran d A vg.
)
0 1
g
o
L(
se
hctefer
P
f
o
#
S T
A T
R P
Fig.11. Thenumberoftheprefetches.(Theprefetchesoftherecordprotectorrefertothoseoftheaccesstrackerguidedbytherecordprotector.)

13
average when PREFENDER is implemented. For a few cases, the datapath of the access tracker, since the access tracker
thelatencybecomeshigherthanthebaseline,whichleadstoa predicts and prefetches the eviction cachelines, 20 bits are
slight performance drop, such as 458.sjeng. Some cases have enough for calculating the DiffMin even when L1Dcache is
similar miss latencies before and after applying PREFENDER, aslargeas1MB.Several20-bitcomparatorsand20-bitadders
but the performance is still improved, such as 400.perlbench are used for each access buffer. The hardware consumption is
| and 429.mcf. |                |       |      |           |     |               |     | also reasonable. |      |            |                  |        |            |      |       |
| ------------ | -------------- | ----- | ---- | --------- | --- | ------------- | --- | ---------------- | ---- | ---------- | ---------------- | ------ | ---------- | ---- | ----- |
|              |                |       |      |           |     |               |     | For the          | SRAM | size       | of the           | record | protector, | the  | scale |
|              |                |       |      |           |     |               |     | buffer has       | 8    | entries in | the experiments, |        | with       | each | entry |
|              |  4 0 0 .p erlb | en ch |  4 5 | 8 .sjen g |     |  4 5 6 .h m m | er, |                  |      |            |                  |        |            |      |       |
 4 0 1 .b zip 2  4 6 4 .h 2 6 4 ref           4 6 2 .lib q u an tu m , 16(sc)+64(BlkAddr)= 80 bits. For each access buffer, the
|     |  4 2 9 .m | cf  |  4 7 | 1 .o m n etp p |           4 | 7 3 .astar, |     |     |     |     |     |     |     |     |     |
| --- | --------- | --- | ---- | -------------- | ----------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
3 5           9 9 9 .sp ecran d record protector requires another 80-bit register for the scale
| sreff |  4 4 5 .g o | b m k |  4 8 | 3 .x alan cb m | k   |     |     |     |     |     |     |     |     |     |     |
| ----- | ----------- | ----- | ---- | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
3 0 history. Therefore, 400 bytes are needed. For the datapath of
u
B  2 5
detcet the record protector, a 2-way associative L1Dcache is 64KB,
2 0
|     |     |     |     |     |     |     |     | with each | cacheline | of  | 64 bytes, | so 9 | bits are | used | for the |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | --------- | --- | --------- | ---- | -------- | ---- | ------- |
or 1 5
P e 1 0 set index of the cache. Since the target of the prefetching
h 5
T f is the cachelines, we only use the set index (the address
o  0
# 0 % 2 5 % 5 0 % 7 5 % 1 0 0 % of the cache entries) to calculate the modules, and several
|     |     |     | E x ecu | tio n  P ro g ress |     |     |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | ------- | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Fig. 12. The number of the protected buffers during the execution. (The hardware modules of 9-bit modulus are needed. According
configurationsarethesameasthatofColumn2inTableV.) to the synthesis results from Synopsys Design Compiler with
|           |                   |          |            |         |           |             |         | ASAP 7nm       | library | [41],            | the modulus           | only      | needs         | 2 cycles | for     |
| --------- | ----------------- | -------- | ---------- | ------- | --------- | ----------- | ------- | -------------- | ------- | ---------------- | --------------------- | --------- | ------------- | -------- | ------- |
| We        | further evaluated |          | the        | number  | of the    | prefetches  | per-    |                |         |                  |                       |           |               |          |         |
|           |                   |          |            |         |           |             |         | calculation    | with    | 9-bit bandwidth, |                       | which     | is much       | quicker  | than    |
| formed    | by the scale      | tracker, | the        | access  | tracker,  | and the     | record  |                |         |                  |                       |           |               |          |         |
|           |                   |          |            |         |           |             |         | memory         | access. | Since the        | record                | protector | only          | works    | upon    |
| protector | of PREFENDER.     |          | The        | results | are shown | in Figure   | 11.     |                |         |                  |                       |           |               |          |         |
|           |                   |          |            |         |           |             |         | the memory     | access  | of a             | load, the             | modulus   | calculation   |          | latency |
| Note that | the access        | tracker  | prefetches |         | the most  | data,       | and the |                |         |                  |                       |           |               |          |         |
|           |                   |          |            |         |           |             |         | can be ignored |         | through          | parallel calculation. |           |               |          |         |
| record    | protector         | guides   | the access | tracker |           | to prefetch | more    |                |         |                  |                       |           |               |          |         |
|           |                   |          |            |         |           |             |         | In summary,    |         | the hardware     | consumption           |           | is reasonable |          | when    |
datathanthescaletracker.Thereasonisthatthescaletracker
|            |              |           |                     |          |         |                 |        | PREFENDER | is  | implemented | in a       | modern | 64-bit | processor. |     |
| ---------- | ------------ | --------- | ------------------- | -------- | ------- | --------------- | ------ | --------- | --- | ----------- | ---------- | ------ | ------ | ---------- | --- |
| performs   | one prefetch |           | when                | a target | address | of              | a load | is        |     |             |            |        |        |            |     |
| calculated | by addition  |           | and multiplication, |          |         | and the         | scale  | is        |     |             |            |        |        |            |     |
|            |              |           |                     |          |         |                 |        |           |     | VI.         | CONCLUSION |        |        |            |     |
| larger     | than the     | cacheline | size.               | This     | happens | less frequently |        |           |     |             |            |        |        |            |     |
than triggering the record protector, which helps the access In this work, a secure prefetcher named PREFENDER is
tracker prefetch each time a scale from the scale tracker is proposed, which can defeat cache side channel attacks while
recordedandaload’stargetaddresshitsthescalehistory.For maintaining or even improving performance. In PREFENDER,
theaccesstracker,therequirementforprefetchingistheeasiest
|     |     |     |     |     |     |     |     | Scale Tracker |     | (ST), Access | Tracker | (AT), | and | Record | Pro- |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ------------ | ------- | ----- | --- | ------ | ---- |
to be satisfied since it only needs a load to be frequently tector (RP) are designed to predict the eviction cachelines
executed. according to the victim’s memory access during phase 2,
| Finally, | the number |     | of the | protected | access | buffers | dur- |         |                |        |          |     |        |       |        |
| -------- | ---------- | --- | ------ | --------- | ------ | ------- | ---- | ------- | -------------- | ------ | -------- | --- | ------ | ----- | ------ |
|          |            |     |        |           |        |         |      | predict | the attacker’s | access | patterns |     | during | phase | 3, and |
ing the execution is tested in Figure 12, which indicates increasethe robustness,respectively.The security isincreased
| that different | benchmarks |     | have | different | patterns | on  | the pro- |                |     |              |            |     |          |         |     |
| -------------- | ---------- | --- | ---- | --------- | -------- | --- | -------- | -------------- | --- | ------------ | ---------- | --- | -------- | ------- | --- |
|                |            |     |      |           |          |     |          | by prefetching |     | the eviction | cachelines |     | that can | confuse | the |
tected buffer numbers. For 400.perlbench, 458.sjeng, and attacker. Experiments on Flush+Reload, Evict+Reload, and
464.h264ref, most of the buffers are protected during the Prime+Probe prove the effectiveness and robustness of our
| execution. | In contrast, |     | 456.hmmer, |     | 462.libquantum, |     | 473.as- |          |          |             |             |     |     |                |     |
| ---------- | ------------ | --- | ---------- | --- | --------------- | --- | ------- | -------- | -------- | ----------- | ----------- | --- | --- | -------------- | --- |
|            |              |     |            |     |                 |     |         | defense. | Besides, | the average | performance |     | is  | also increased |     |
tar, and 999.specrand have no protected buffer. For other by the accurate prediction, according to the evaluations on
benchmarks,thenumberoftheprotectedbuffersvaries.These SPEC CPU 2006 and 2017 benchmarks.
| results    | indicate  | that different | functionality |            | of  | the program | can |     |     |     |     |     |     |     |     |
| ---------- | --------- | -------------- | ------------- | ---------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| affect the | behaviors | of             | the record    | protector. |     |             |     |     |     |     |     |     |     |     |     |
REFERENCES
|             |          |     |             |     |          |     |     | [1] D. Page, | “Theoretical | use | of cache | memory | as a | cryptanalytic | side- |
| ----------- | -------- | --- | ----------- | --- | -------- | --- | --- | ------------ | ------------ | --- | -------- | ------ | ---- | ------------- | ----- |
| E. Hardware | Resource |     | Consumption |     | Analysis |     |     |              |              |     |          |        |      |               |       |
channel.”IACRCryptologyePrintArchive,vol.2002,no.169,pp.1–23,
| Webrieflyanalyzetheupperboundofthehardwareresource |     |     |      |         |     |                |     | 2002.                                                             |     |     |     |     |     |     |     |
| -------------------------------------------------- | --- | --- | ---- | ------- | --- | -------------- | --- | ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
|                                                    |     |     |      |         |     |                |     | [2] Y.YaromandK.Falkner,“Flush+reload:Ahighresolution,lownoise,l3 |     |     |     |     |     |     |     |
| consumption.                                       | For | the | SRAM | size of | the | scale tracker, | the |                                                                   |     |     |     |     |     |     |     |
cacheside-channelattack,”USENIXSecuritySymposium,pp.719–732,
| prefetching | is performed |     | within | one | page, | so 16 | bits are | 2014. |     |     |     |     |     |     |     |
| ----------- | ------------ | --- | ------ | --- | ----- | ----- | -------- | ----- | --- | --- | --- | --- | --- | --- | --- |
enough for the values in the calculation buffers even with a [3] P.Kocher,J.Horn,A.Fogh,D.Genkin,D.Gruss,W.Haas,M.Ham-
burg,M.Lipp,S.Mangard,T.Prescheretal.,“Spectreattacks:Exploit-
| page size | of 64KB. | For | each | register, | there | are two | values |     |             |             |                |     |             |     |          |
| --------- | -------- | --- | ---- | --------- | ----- | ------- | ------ | --- | ----------- | ----------- | -------------- | --- | ----------- | --- | -------- |
|           |          |     |      |           |       |         |        | ing | speculative | execution,” | IEEE Symposium |     | on Security | and | Privacy, |
associated,sothescaletrackerneedshundredsofbytesintotal
pp.1–19,2019.
|            |               |     |         |            |     |             |          | [4] M.Lipp,M.Schwarz,D.Gruss,T.Prescher,W.Haas,A.Fogh,J.Horn, |     |            |           |                    |     |         |        |
| ---------- | ------------- | --- | ------- | ---------- | --- | ----------- | -------- | ------------------------------------------------------------- | --- | ---------- | --------- | ------------------ | --- | ------- | ------ |
| for dozens | of registers. |     | For the | datapath   | of  | the scale   | tracker, |                                                               |     |            |           |                    |     |         |        |
|            |               |     |         |            |     |             |          | S. Mangard,                                                   |     | P. Kocher, | D. Genkin | et al., “Meltdown: |     | Reading | kernel |
| an adder,  | a multiplier  |     | and a   | comparator | are | used, which | are      |                                                               |     |            |           |                    |     |         |        |
memoryfromuserspace,”USENIXSecuritySymposium,pp.973–990,
also 16-bit.
2018.
FortheSRAMsizeoftheaccesstracker,thereare32access [5] C.Canella,J.VanBulck,M.Schwarz,M.Lipp,B.VonBerg,P.Ortner,
|          |         |       |       |          |      |         |          | F. Piessens, | D.  | Evtyushkin, | and D. | Gruss, “A | systematic | evaluation | of  |
| -------- | ------- | ----- | ----- | -------- | ---- | ------- | -------- | ------------ | --- | ----------- | ------ | --------- | ---------- | ---------- | --- |
| buffers, | each of | which | has 8 | entries. | Even | if each | value of |              |     |             |        |           |            |            |     |
transientexecutionattacksanddefenses,”USENIXSecuritySymposium,
the buffer is 64-bit, only <3KB SRAMs are required. For pp.249–266,2019.

14
[6] D.A.Osvik,A.Shamir,andE.Tromer,“Cacheattacksandcountermea- [29] M. K. Qureshi, “Ceaser: Mitigating conflict-based cache attacks via
sures: the case of aes,” Cryptographers’ track at the RSA conference, encrypted-addressandremapping,”IEEE/ACMInternationalSymposium
| pp.1–20,2006. |     |     |     |     |     |     |     | onMicroarchitecture,pp.775–787,2018. |
| ------------- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------ |
[7] V. Kiriansky, I. Lebedev, S. Amarasinghe, S. Devadas, and J. Emer, [30] Z. Wang and R. B. Lee, “New cache designs for thwarting software
“Dawg:Adefenseagainstcachetimingattacksinspeculativeexecution cache-basedsidechannelattacks,”ACM/IEEEInternationalsymposium
processors,”IEEE/ACMInternationalSymposiumonMicroarchitecture, onComputerarchitecture,pp.494–505,2007.
pp.974–987,2018. [31] M. Yan, B. Gopireddy, T. Shull, and J. Torrellas, “Secure hierarchy-
[8] P.Li,L.Zhao,R.Hou,L.Zhang,andD.Meng,“Conditionalspecula- awarecachereplacementpolicy(sharp):Defendingagainstcache-based
tion:Aneffectiveapproachtosafeguardout-of-orderexecutionagainst sidechannelattacks,”ACM/IEEEInternationalSymposiumonComputer
spectre attacks,” IEEE International Symposium on High Performance Architecture,pp.347–360,2017.
ComputerArchitecture,pp.264–276,2019. [32] P.C.Kocher,“Timingattacksonimplementationsofdiffie-hellman,rsa,
[9] M.Yan,J.Choi,D.Skarlatos,A.Morrison,C.Fletcher,andJ.Torrel- dss,andothersystems,”InternationalCryptologyConference,pp.104–
| las, “InvisiSpec: |     | Making | Speculative | Execution |     | Invisible | in the Cache | 113,1996. |
| ----------------- | --- | ------ | ----------- | --------- | --- | --------- | ------------ | --------- |
Hierarchy,”IEEE/ACMInternationalSymposiumonMicroarchitecture, [33] J.DaemenandV.Rijmen,“Aesproposal:Rijndael,”1999.
pp.428–441,2018. [34] E. M. Koruyeh, K. N. Khasawneh, C. Song, and N. Abu-Ghazaleh,
[10] H. Fang, S. S. Dayapule, F. Yao, M. Doroslovacˇki, and G. Venkatara- “SpectreReturns!SpeculationAttacksusingtheReturnStackBuffer,”
mani, “Defeating cache timing channels with hardware prefetchers,” USENIXWorkshoponOffensiveTechnologies,2018.
IEEEDesign&Test,vol.38,no.3,pp.7–14,2021. [35] J.StecklinaandT.Prescher,“LazyFP:LeakingFPURegisterStateusing
[11] H. Fang, M. Doroslovacˇki, and G. Venkataramani, “Reuse-trap: re- MicroarchitecturalSide-Channels,”arXivpreprint,2018.
purposingcachereusedistancetodefendagainstsidechannelleakage,” [36] J.V.Bulck,M.Minkin,O.Weisse,D.Genkin,B.Kasikci,F.Piessens,
ACM/IEEEDesignAutomationConference,pp.1–6,2020.
M.Silberstein,T.F.Wenisch,Y.Yarom,andR.Strackx,“Foreshadow:
[12] A.FuchsandR.B.Lee,“Disruptiveprefetching:impactonside-channel ExtractingtheKeystotheIntelSGXKingdomwithTransientOut-of-
attacks and cache designs,” ACM International Systems and Storage OrderExecution,”USENIXSecuritySymposium,pp.991–1008,2018.
Conference,pp.1–12,2015. [37] Thegem5Simulator,http://www.gem5.org/Main Page.
[13] B.Panda,“FoolingtheSenseofCross-CoreLast-LevelCacheEviction [38] SPECCPU2006Benchmark,https://www.spec.org/cpu2006/.
Based Attacker by Prefetching Common Sense,” International Confer- [39] SPECCPU2017Benchmark,https://www.spec.org/cpu2017/.
ence on Parallel Architectures and Compilation Techniques, pp. 138– [40] J.-L.BaerandT.-F.Chen,“Aneffectiveon-chippreloadingschemeto
150,2019. reducedataaccesspenalty,”ACM/IEEEConferenceonSupercomputing,
[14] D. Gruss, R. Spreitzer, and S. Mangard, “Cache template attacks: pp.176–186,1991.
Automating attacks on inclusive last-level caches,” USENIX Security [41] ASAP7nmPredictivePDK,http://asap.asu.edu/asap/,2016.
Symposium,pp.897–912,2015.
| [15] A. Smith, | “Sequential |     | Program | Prefetching | in  | Memory | Hierarchies,” |     |
| -------------- | ----------- | --- | ------- | ----------- | --- | ------ | ------------- | --- |
Computer,vol.11,no.12,pp.7–21,1978.
[16] J.-L.BaerandT.-F.Chen,“Aneffectiveon-chippreloadingschemeto
reducedataaccesspenalty,”ACM/IEEEconferenceonSupercomputing,
pp.176–186,1991.
| [17] S. Srinath, | O.  | Mutlu,    | H. Kim,         | and Y. | N. Patt,                 | “Feedback | directed |     |
| ---------------- | --- | --------- | --------------- | ------ | ------------------------ | --------- | -------- | --- |
| prefetching:     |     | Improving | the performance |        | and bandwidth-efficiency |           | of       |     |
hardwareprefetchers,”IEEEInternationalSymposiumonHighPerfor-
manceComputerArchitecture,pp.63–74,2007.
| [18] D. Joseph | and           | D. Grunwald, |           | “Prefetching | using    | markov        | predictors,” |     |
| -------------- | ------------- | ------------ | --------- | ------------ | -------- | ------------- | ------------ | --- |
| ACM/IEEE       | International |              | Symposium | on           | Computer | Architecture, | pp.          |     |
252–263,1997.
| [19] A.-C. | Lai, C. | Fide, | and B. Falsafi, | “Dead-block |     | prediction | & dead- |     |
| ---------- | ------- | ----- | --------------- | ----------- | --- | ---------- | ------- | --- |
blockcorrelatingprefetchers,”ACM/IEEEInternationalSymposiumon
ComputerArchitecture,pp.144–154,2001.
[20] Z.HeandR.B.Lee,“HowSecureisYourCacheagainstSide-Channel
Attacks?”IEEE/ACMInternationalSymposiumonMicroarchitecture,p.
341–353,2017.
[21] H.Fang,S.S.Dayapule,F.Yao,M.Doroslovaa˘?Ki,andG.Venkatara-
mani,“PrODACT:Prefetch-ObfuscatortoDefendAgainstCacheTiming
| Channels,” | International |     | Journal | of Parallel | Programming, |     | vol. 47, |     |
| ---------- | ------------- | --- | ------- | ----------- | ------------ | --- | -------- | --- |
no.4,p.571–594,2019.
[22] O.Weisse,I.Neal,K.Loughlin,T.F.Wenisch,andB.Kasikci,“Nda:
| Preventing | speculative |     | execution | attacks | at their | source,” | IEEE/ACM |     |
| ---------- | ----------- | --- | --------- | ------- | -------- | -------- | -------- | --- |
InternationalSymposiumonMicroarchitecture,pp.572–586,2019.
| [23] K. Barber, | A.  | Bacha, | L. Zhou, | Y. Zhang, | and R. | Teodorescu, | “Spec- |     |
| --------------- | --- | ------ | -------- | --------- | ------ | ----------- | ------ | --- |
shield:Shieldingspeculativedatafrommicroarchitecturalcovertchan-
nels,”InternationalConferenceonParallelArchitecturesandCompila-
tionTechniques,pp.151–164,2019.
| [24] K. N. | Khasawneh, | E.            | M. Koruyeh,   | C. Song,   | D.        | Evtyushkin, | D. Pono-     |     |
| ---------- | ---------- | ------------- | ------------- | ---------- | --------- | ----------- | ------------ | --- |
| marev,     | and N.     | Abu-Ghazaleh, |               | “Safespec: | Banishing | the         | spectre of a |     |
| meltdown   | with       | leakage-free  | speculation,” |            | ACM/IEEE  | Design      | Automa-      |     |
tionConference,pp.1–6,2019.
| [25] S. Ainsworth |         | and T. | M. Jones, | “Muontrap:  | Preventing |          | cross-domain |     |
| ----------------- | ------- | ------ | --------- | ----------- | ---------- | -------- | ------------ | --- |
| spectre-like      | attacks | by     | capturing | speculative | state,”    | ACM/IEEE | Inter-       |     |
nationalSymposiumonComputerArchitecture,pp.132–144,2020.
| [26] T. Solanki | and       | B. Panda, | “SpecPref:    | High | Performing         | Speculative | At-       |     |
| --------------- | --------- | --------- | ------------- | ---- | ------------------ | ----------- | --------- | --- |
| tacks           | Resilient | Hardware  | Prefetchers,” |      | IEEE International |             | Symposium |     |
onHardwareOrientedSecurityandTrust(HOST),pp.57–60,2022.
[27] F.Liu,Q.Ge,Y.Yarom,F.Mckeen,C.Rozas,G.Heiser,andR.B.Lee,
“Catalyst:Defeatinglast-levelcachesidechannelattacksincloudcom-
puting,”IEEEinternationalsymposiumonhighperformancecomputer
architecture(HPCA),pp.406–418,2016.
[28] T.Kim,M.Peinado,andG.Mainar-Ruiz,“{STEALTHMEM}:{System-
| Level} | protection | against | {Cache-Based} |     | side | channel | attacks in the |     |
| ------ | ---------- | ------- | ------------- | --- | ---- | ------- | -------------- | --- |
cloud,”USENIXSecuritySymposium,pp.189–204,2012.
