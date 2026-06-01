---
publish: true
---

SoK: Prudent Evaluation Practices for Fuzzing
To appear at IEEE S&P 2024. Author version.
Moritz Schloegel1, Nils Bars1, Nico Schiller1, Lukas Bernhard1, Tobias Scharnowski1
Addison Crump1, Arash Ale-Ebrahim1, Nicolai Bissantz2, Marius Muench3, Thorsten Holz1
1CISPA Helmholtz Center for Information Security, {first.lastname}@cispa.de
2Ruhr University Bochum, nicolai.bissantz@ruhr-uni-bochum.de
3University of Birmingham, m.muench@bham.ac.uk
Abstract—Fuzzinghasproventobeahighlyeffectiveapproach 1. Introduction
touncoversoftwarebugsoverthepastdecade.AfterAFLpop-
ularized the groundbreaking concept of lightweight coverage Fuzzing, a portmanteau of “fuzz testing”, has gained
feedback, the field of fuzzing has seen a vast amount of scien- much attention in recent years, and the method has proven
tificworkproposingnewtechniques,improvingmethodological to be highly successful in uncovering many types of faults
aspects of existing strategies, or porting existing methods to in software systems. Companies such as Meta, Google, and
new domains. All such work must demonstrate its merit by Oraclehaveinvestedsignificantresourcesinthistechnology
showing its applicability to a problem, measuring its perfor- anduseittotesttheirproducts.Largesoftwareprojectssuch
as web browsers or the Linux kernel incorporate fuzzing
mance,andoftenshowingitssuperiorityoverexistingworksin
into their development cycle, and Google is running an
athorough,empiricalevaluation.Yet,fuzzingishighlysensitive
extensive and continuous fuzzing campaign for more than
toitstarget,environment,andcircumstances,e.g.,randomness
1,200 open-source projects via OSS-Fuzz [62]. Beyond the
in the testing process. After all, relying on randomness is one
wideacceptanceintheindustry,alargenumberofacademic
of the core principles of fuzzing, governing many aspects of
papers have proposed numerous improvements and novel
a fuzzer’s behavior. Combined with the often highly difficult
techniquestoenhancefuzzingfurther.Morespecifically,we
to control environment, the reproducibility of experiments is a
foundthat,overthepastsixyears,morethan280paperson
crucial concern and requires a prudent evaluation setup. To
fuzzing have been published in the top computer security
address these threats to validity, several works, most notably
and software engineering venues.
Evaluating Fuzz Testing by Klees et al., have outlined how
A cornerstone of fuzzing research, and science in gen-
a carefully designed evaluation setup should be implemented,
eral, is that other researchers can critically assess the cor-
butitremainsunknowntowhatextenttheirrecommendations
rectnessofscientificresults.Tothisend,theresearchresults
have been adopted in practice.
mustbereproducible,meaningthatanothergroupshouldbe
able to obtain the same results using the same experimental
In this work, we systematically analyze the evaluation
setup, often by using a research artifact provided by the au-
of 150 fuzzing papers published at the top venues between
thors[8].Reproducibilityisparamountforotherresearchers
2018 and 2023. We study how existing guidelines are imple-
to understand, trust, and build on the research results.
mented and observe potential shortcomings and pitfalls. We
To enable high-quality research and provide a common
findasurprisingdisregardoftheexistingguidelinesregarding
foundation for evaluating fuzzing methods, several works
statistical tests and systematic errors in fuzzing evaluations.
describehownewlyproposedfuzzingapproachesshouldbe
For example, when investigating reported bugs, we find that
evaluated. In 2018, the first and most influential paper de-
the search for vulnerabilities in real-world software leads to
scribing a reproducible evaluation design was published by
authorsrequestingandreceivingCVEsofquestionablequality.
Kleesetal.[88].Itdescribesguidelinestoadviseresearchers
Extending our literature analysis to the practical domain, we
on how fuzzing research should evaluate their respective
attempt to reproduce claims of eight fuzzing papers. These
contributions. For example, a crucial insight introduced by
case studies allow us to assess the practical reproducibility
Klees et al. is the repetition of experiments to account for
of fuzzing research and identify archetypal pitfalls in the the inherent randomness of the fuzzing process. Although
evaluationdesign.Unfortunately,ourreproducedresultsreveal Klees et al. recommend “a sufficient number of trials” and
severaldeficienciesinthestudiedpapers,andweareunableto use 30 trials in their own experiments, we found that in
fullysupportandreproducetherespectiveclaims.Tohelpthe practice, this recommendation is interpreted as anything
fieldoffuzzingmovetowardascientificallyreproducibleeval- between three and 20 repetitions. Another guideline is to
uation strategy, we propose updated guidelines for conducting confirm the fuzzers’ performance statistically; however, this
a fuzzing evaluation that future work should follow. makes little sense with few repetitions and is often skipped.
4202
yaM
61
]ES.sc[
1v02201.5042:viXra

In this work, we systematically review how the recom- 2. Fuzzing Evaluation Guidelines
mendationsforevaluatingfuzzingmethodsareimplemented
in practice and critically evaluate the reproducibility of We first provide a brief overview of fuzzing before
fuzzing research. We propose revised best practices for describing several generally accepted best practices that
evaluating fuzzing methods and point out pitfalls that we guide a typical fuzzing evaluation.
| have observed | in  | practice. | In  | other | fields, | such | work has |     |     |     |     |     |     |
| ------------- | --- | --------- | --- | ----- | ------- | ---- | -------- | --- | --- | --- | --- | --- | --- |
had a significant impact on improving research from a 2.1. Background on Fuzzing
| methodological |     | point of | view | [1], [4], | [46], | [155]. |     |     |     |     |     |     |     |
| -------------- | --- | -------- | ---- | --------- | ----- | ------ | --- | --- | --- | --- | --- | --- | --- |
We conduct a thorough literature review of 150 fuzzing Fuzzing, also referred to as fuzz testing, is a dynamic
|                  |     |                |     | A∗  |           |        |     | testing | technique | with | the goal of uncovering |     | bugs in |
| ---------------- | --- | -------------- | --- | --- | --------- | ------ | --- | ------- | --------- | ---- | ---------------------- | --- | ------- |
| papers published |     | in prestigious |     |     | venues—as | ranked | by  |         |           |      |                        |     |         |
CORE2023 [128]—between 2018 and 2023. While we pri- systems. This typically happens by mutating some initial
marily focus on computer security venues, namely IEEE input(s)tothesystemorbyderivinginputsfrominputspec-
SymposiumonSecurityandPrivacy(S&P),USENIXSecu- ifications such as grammars. While processing the provided
ritySymposium(USENIX),ACMConferenceonComputer input, the system under test is monitored for interesting
and Communications Security (CCS), and ISOC Network behavior. Beyond easily observable faults, such as program
and Distributed System Security (NDSS) Symposium, we crashes, fuzzers can use more sophisticated bug oracles,
alsoexaminethreesoftwareengineeringvenues:IEEE/ACM such as sanitizers or differential testing. Moreover, modern
International Conference on Automated Software Engineer- fuzzers often use lightweight instrumentation to receive
ing (ASE), ACM Joint European Software Engineering coverage feedback, allowing them to track inputs that ex-
Conference and Symposium on the Foundations of Soft- ecutedpreviouslyunseenedges.Acomprehensiveoverview
ware Engineering (ESEC/FSE), and International Confer- of various fuzzing techniques can be found in the Fuzzing
ence on Software Engineering (ICSE). For all papers, we: Book [178], and several surveys present a comprehensive
|                    |     |         |     |             |     |               |     | overview | of this | topic [112], | [193] or open | challenges | in  |
| ------------------ | --- | ------- | --- | ----------- | --- | ------------- | --- | -------- | ------- | ------------ | ------------- | ---------- | --- |
| (i) systematically |     | analyze | how | evaluations |     | are conducted |     |          |         |              |               |            |     |
(in terms of metrics, targets, baselines, reported bugs, etc.), this domain [14]. Most fuzzing research proposes an im-
(ii) check whether common fuzzing guidelines (as outlined provement by way of new techniques, new components, or
byKleesetal.[88]orembodiedinimplicitcommunitywis- entirelynewfuzzers—fewworksfocusonthetheorybehind
|     |     |     |     |     |     |     |     | fuzzing | [20], [21], | [23], [107]. |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ----------- | ------------ | --- | --- | --- |
dom,e.g.,“donotuseartificialbugdatasets”)arefollowed,
and (iii) investigate potential flaws threatening the validity A fundamental principle of all fuzzers is the inherent
of the respective evaluation. inclusion of randomness into the testing process. Starting
Following our literature analysis, we present eight case from the scheduling order of the process, through the input
|            |         |        |        |           |        |     |         | and the | mutations | applied | to it, to the fuzzing | environment |     |
| ---------- | ------- | ------ | ------ | --------- | ------ | --- | ------- | ------- | --------- | ------- | --------------------- | ----------- | --- |
| studies of | fuzzing | papers | across | different | fields | and | attempt |         |           |         |                       |             |     |
|            |         |        |        |           |        |     |         |         |           |         | getpid, time,         | rand,       |     |
toreproduce(partsof)theirevaluation.Foreachcasestudy, (including functions such as or or
we discuss any shortcomings we have identified because sharedresourcessuchasthefilesystem),therearenumerous
they illustrate potential pitfalls of which researchers should sources of randomness that make deterministic and repro-
|           |         |              |            |                |     |       |          | ducible | execution | challenging. |     |     |     |
| --------- | ------- | ------------ | ---------- | -------------- | --- | ----- | -------- | ------- | --------- | ------------ | --- | --- | --- |
| be aware. | Note    | that         | these case | studies        | are | not   | intended |         |           |              |     |     |     |
| to point  | fingers | or criticize |            | any particular |     | work. | Instead, |         |           |              |     |     |     |
we aim to highlight potential challenges that can affect the 2.2. Guidelines of Evaluating Fuzz Testing
| outcome | of a research |     | paper and | explore | what | aspects | need |     |     |     |     |     |     |
| ------- | ------------- | --- | --------- | ------- | ---- | ------- | ---- | --- | --- | --- | --- | --- | --- |
Therandomizednatureoffuzzingneedstobetakeninto
tobeconsideredwhendesigningtheevaluationofafuzzing
method. Based on the findings of our literature review and account during the evaluation, which leads to challenges
casestudies,weproposebestpracticesforevaluatingfuture with reproducibility of research results in practice. Hence,
fuzzing methods to enable reproducible research. the seminal paper by Klees et al. [88] outlined several
|             |     |      |               |     |                    |     |     | guidelines | on  | how a proper   | fuzzing evaluation | should      | be   |
| ----------- | --- | ---- | ------------- | --- | ------------------ | --- | --- | ---------- | --- | -------------- | ------------------ | ----------- | ---- |
| In summary, | we  | make | the following |     | key contributions: |     |     |            |     |                |                    |             |      |
|             |     |      |               |     |                    |     |     | conducted. | For | a reproducible | and fair           | evaluation, | they |
Weconductasystematicliteraturesurveyof150papers
| •         |     |          |     |       |        |        |           | propose         | the following | recommendations: |              |      |        |
| --------- | --- | -------- | --- | ----- | ------ | ------ | --------- | --------------- | ------------- | ---------------- | ------------ | ---- | ------ |
| published | in  | the past | six | years | at top | venues | to assess |                 |               |                  |              |      |        |
|           |     |          |     |       |        |        |           | Recommendation1 |               | – Baseline:      | A comparison | with | a rel- |
how fuzzing methods are typically evaluated. evant and reasonable baseline is imperative to show what
| • We      | attempt | to reproduce |         | eight        | papers | to assess | the |             |     |            |                  |     |     |
| --------- | ------- | ------------ | ------- | ------------ | ------ | --------- | --- | ----------- | --- | ---------- | ---------------- | --- | --- |
|           |         |              |         |              |        |           |     | improvement | a   | particular | fuzzer provides. |     |     |
| practical | aspect  | of           | fuzzing | evaluations. |        | In doing  | so, |             |     |            |                  |     |     |
Recommendation2–Targets:Arelevantsampleoftargets
| we  | identify | several | obstacles | that | illustrate | (sometimes |     |            |         |               |               |           |     |
| --- | -------- | ------- | --------- | ---- | ---------- | ---------- | --- | ---------- | ------- | ------------- | ------------- | --------- | --- |
|     |          |         |           |      |            |            |     | to compare | against | is necessary. | This includes | benchmark |     |
subtle) shortcomings of evaluating fuzzing methods. programs with known bugs that can be used as a ground
| • Based      | on our | lessons | learned, | we        | provide | revised | rec-    |          |         |               |               |     |     |
| ------------ | ------ | ------- | -------- | --------- | ------- | ------- | ------- | -------- | ------- | ------------- | ------------- | --- | --- |
|              |        |         |          |           |         |         |         | truth to | measure | bug detection | capabilities. |     |     |
| ommendations |        | and     | best     | practices | for     | future  | fuzzing |          |         |               |               |     |     |
Recommendation3–Setup&Parameters:Duetothein-
evaluations.
herentrandomnessoffuzzing,individualrunswiththesame
Supplementarymaterialforthisworkisavailableonline configuration can yield significantly different outcomes. To
at https://github.com/fuzz-evaluator/, including our repro- address this problem, Klees et al. propose repeating the
duction artifacts and recommended best practices for future experiment multiple times. Similarly, fuzzing performance
work (see https://github.com/fuzz-evaluator/guidelines). may vary within a single run, so short runtimes are not

appropriate for extrapolating the behavior of a fuzzer over confirm evaluation results, authors must measure signifi-
longer times. They propose 24 hours as a reasonable fuzzer cance and effect size using established techniques. They
runtimeandrecommendplottingtheperformanceovertime. should discuss threats to the validity of their evaluation and
Seed sets must be well documented and carefully selected; how they mitigated them. Finally, authors should carefully
ideally, various sets, including the empty or uninformed document their setup and publish evaluation artifacts on
| seed, are      | tested. |                                     |     |     |     |     |     | long-term | stable | platforms | such | as Zenodo. |     |     |     |
| -------------- | ------- | ----------------------------------- | --- | --- | --- | --- | --- | --------- | ------ | --------- | ---- | ---------- | --- | --- | --- |
| Recommendation |         | 4–EvaluationMetrics:Ideally,fuzzing |     |     |     |     |     |           |        |           |      |            |     |     |     |
evaluations should not be based on proxy metrics such as 2.5. Fuzzing Benchmarks
| code coverage |      | alone,    | but on | a fuzzer’s    | ability | to             | find bugs, |      |            |         |              |     |     |            |     |
| ------------- | ---- | --------- | ------ | ------------- | ------- | -------------- | ---------- | ---- | ---------- | ------- | ------------ | --- | --- | ---------- | --- |
| i.e., the     | goal | for which | it     | was designed. |         | In particular, | an         |      |            |         |              |     |     |            |     |
|               |      |           |        |               |         |                |            | Over | the years, | several | standardized |     |     | benchmarks | and |
evaluationshouldnotrelyonheuristicssuchasAFL’scover- platforms to conduct fair and comparable fuzzing eval-
ageprofilesorstackhashing.Complementingtheevaluation uations have been proposed, e.g., Google’s Fuzzer-Test-
on bug detection, Klees et al. recommend code coverage in Suite [63] (2016; superseded by FuzzBench), LAVA-
| terms of | basic | blocks | or edges | as secondary |     | metric. |     |        |         |     |              |     |       |      |         |
| -------- | ----- | ------ | -------- | ------------ | --- | ------- | --- | ------ | ------- | --- | ------------ | --- | ----- | ---- | ------- |
|          |       |        |          |              |     |         |     | M [51] | (2016), | CGC | [45] (2018), |     | Magma | [70] | (2020), |
Recommendation 5 – Statistical Evaluation: Finally, the FuzzBench [118] (2020), Unibench [99] (2021), Pro-
fuzzing evaluation should undergo statistical evaluation to FuzzBench [123] (2021), and RevBugBench [183] (2022).
rule out that the observed behavior is by mere chance. This These benchmark platforms aim to measure the per-
requiresasufficientnumberoftrials(Kleesetal.themselves
|     |     |     |     |     |     |     |     | formance | of general-purpose |     |     | fuzzing, | except |     | for Pro- |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------------------ | --- | --- | -------- | ------ | --- | -------- |
use 30); then, a statistical test such as the Mann-Whitney FuzzBench, which focuses on stateful protocol fuzzing.
U-testorbootstrap-basedmethodsshouldbeusedtotestthe Overall, we can distinguish between benchmarks focusing
null hypothesis that the new method exhibits no difference on the comparison of achieved coverage (Google’s Fuzzer-
| compared | to a | reasonable | baseline. |     |     |     |     |             |           |     |            |     |               |     |     |
| -------- | ---- | ---------- | --------- | --- | --- | --- | --- | ----------- | --------- | --- | ---------- | --- | ------------- | --- | --- |
|          |      |            |           |     |     |     |     | Test-Suite, | Unibench, |     | FuzzBench, | and | ProFuzzBench) |     | and |
thosefocusingonthebug-findingcapabilitiesofthefuzzing
2.3. Guidelines of FuzzBench technique (LAVA-M, CGC, Magma, and RevBugBench).
|     |     |     |     |     |     |     |     | In the latter | category, |     | some utilize |     | artificial | bug | injection |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --------- | --- | ------------ | --- | ---------- | --- | --------- |
FuzzBench [118], a benchmarking suite for general- (LAVA-MandCGC),makeeffortstoportactualvulnerabil-
itiestothelatestversionofaprogram(Magma),ortorevert
| purpose | fuzzer | evaluation | developed |     | by  | Google, | provides |     |     |     |     |     |     |     |     |
| ------- | ------ | ---------- | --------- | --- | --- | ------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
fixes(RevBugBench).Artificialbuginjectionmethodsoften
| several | target | programs | and | aims | to provide | a   | standard- |     |     |     |     |     |     |     |     |
| ------- | ------ | -------- | --- | ---- | ---------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
introduceshallowbugsthatareamenabletofuzzers,andare
| ized setup | for | fair comparison |     | of  | fuzzers. | FuzzBench | is  |     |     |     |     |     |     |     |     |
| ---------- | --- | --------------- | --- | --- | -------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
the successor to the Google Fuzzer Test Suite (FTS) [63]. generally no longer recommended for an evaluation [18],
|     |     |     |     |     |     |     |     | [118], [162], | [183]. |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | ------ | --- | --- | --- | --- | --- | --- |
Duringtheirextensiveevaluation,theauthorsmadetwokey
| observations      | that   | can             | serve as     | a guideline |            | for future | fuzzing |               |                  |          |     |            |     |       |        |
| ----------------- | ------ | --------------- | ------------ | ----------- | ---------- | ---------- | ------- | ------------- | ---------------- | -------- | --- | ---------- | --- | ----- | ------ |
|                   |        |                 |              |             |            |            |         | 3. Literature |                  | Analysis |     |            |     |       |        |
| research.         | First, | the performance |              | of          | a fuzzer   | varies     | signif- |               |                  |          |     |            |     |       |        |
| icantly depending |        | on              | the number   |             | of initial | seeds;     | running |               |                  |          |     |            |     |       |        |
|                   |        |                 |              |             |            |            |         | With          | these guidelines |          | and | benchmarks | in  | mind, | we now |
| without           | seeds  | allows          | for studying | the         | difference | when       | only    |               |                  |          |     |            |     |       |        |
studytheiradoptiontobetterunderstandwhatbestpractices
| a particular | fuzzer | can | solve | some | comparisons/branches. |     |     |     |     |     |     |     |     |     |     |
| ------------ | ------ | --- | ----- | ---- | --------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Second, using a saturated corpus for fuzzing is not rec- are used in fuzzing research. To this end, we perform a
ommended, as fuzzers are barely capable of augmenting it. comprehensive literature survey of recent fuzzing papers.
Eventhoughthisiscommoninpractice,itisnotwellsuited
| to discern      | or measure |     | the performance |             | of  | fuzzers.    |     | 3.1. Method |          |             |          |             |           |             |         |
| --------------- | ---------- | --- | --------------- | ----------- | --- | ----------- | --- | ----------- | -------- | ----------- | -------- | ----------- | --------- | ----------- | ------- |
|                 |            |     |                 |             |     |             |     | We examine  |          | all fuzzing |          | papers      | published | at          | the top |
| 2.4. Guidelines |            | of  | On the          | Reliability |     | of Coverage |     |             |          |             |          |             |           |             |         |
|                 |            |     |                 |             |     |             |     | computer    | security | and         | software | engineering |           | conferences |         |
between2018and20231.Weincludeapaperinouranalysis
| More | recently, | Böhme |     | et al. [23] | made | a number | of  |              |       |          |       |             |     |            |     |
| ---- | --------- | ----- | --- | ----------- | ---- | -------- | --- | ------------ | ----- | -------- | ----- | ----------- | --- | ---------- | --- |
|      |           |       |     |             |      |          |     | if its focus | is on | fuzzing, | e.g., | it proposes | a   | new method | or  |
recommendationsbasedontheirevaluationofthereliability
|     |     |     |     |     |     |     |     | extensively | evaluates | existing |     | ones. In | contrast, | we  | exclude |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | --------- | -------- | --- | -------- | --------- | --- | ------- |
ofcoverage.Inparticular,theyrecommendtouseatleastten
|                |       |              |            |       |          |           |           | papers using | fuzzers | as  | a means  | to   | support | their   | primary |
| -------------- | ----- | ------------ | ---------- | ----- | -------- | --------- | --------- | ------------ | ------- | --- | -------- | ---- | ------- | ------- | ------- |
| representative |       | programs,    | eachtested |       | at least | ten times | for at    |              |         |     |          |      |         |         |         |
|                |       |              |            |       |          |           |           | focus, e.g., | solely  | to  | generate | some | diverse | inputs. | We      |
| least 12       | hours | (preferably, | each       | value | should   | be        | doubled). |              |         |     |          |      |         |         |         |
identify289candidatepapersforwhichwecollectmetadata
| The selected     | programs |        | should   | be real-world |               | programs,     | and      |           |              |            |              |         |             |            |         |
| ---------------- | -------- | ------ | -------- | ------------- | ------------- | ------------- | -------- | --------- | ------------ | ---------- | ------------ | ------- | ----------- | ---------- | ------- |
|                  |          |        |          |               |               |               |          | about the | underlying   | evaluation |              | method, | including   |            | whether |
| a bug evaluation |          | should | be       | done          | on real-world |               | bugs. In |           |              |            |              |         |             |            |         |
|                  |          |        |          |               |               |               |          | the paper | successfully |            | participated | in      | an artifact | evaluation |         |
| addition         | to bugs, | code   | coverage | should        | also          | be evaluated— |          |           |              |            |              |         |             |            |         |
process.Wethenrandomlyselect52%(150)fromthese289
| both using | established |     | metrics. | In particular, |     | fuzzer-specific |     |     |     |     |     |     |     |     |     |
| ---------- | ----------- | --- | -------- | -------------- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
papersandmanuallyreviewthem,i.e.,studythedesignand
| measures        | such       | as AFL’s | unique | paths    | should | be            | avoided.  |             |         |      |            |       |         |     |          |
| --------------- | ---------- | -------- | ------ | -------- | ------ | ------------- | --------- | ----------- | ------- | ---- | ---------- | ----- | ------- | --- | -------- |
|                 |            |          |        |          |        |               |           | evaluation  | of the  | work | in detail. | Table | 1 shows | an  | overview |
| For comparison, |            | authors  | should | choose   | a      | suitable      | baseline, |             |         |      |            |       |         |     |          |
|                 |            |          |        |          |        |               |           | of analyzed | papers. |      |            |       |         |     |          |
| such as         | the fuzzer | on       | top    | of which | the    | new technique | is        |             |         |      |            |       |         |     |          |
implemented.Authorsshouldconsidersplittingbenchmarks 1.For2023,ASEandFSEhavenotpublishedthepapersatthetimeof
into a training and validation set to avoid overfitting. To writing.Wethereforeworkwithavailablepreprints.

TABLE1.OVERVIEWOFANALYZEDPAPERS. the code of their technique, while 23% (66) do not share
their code. Some do not contribute new code, upstreamed
Year Venue Papers Studied theircode,orhavenotyetreleasedthecode(appliestoFSE,
ASE∗ [159],[76],[105] 3/7 whichwilltakeplaceaftertimeofwriting).Regardingother
FSE∗ [166] 1/6
data (excluding code), we find that 11% (31) share data,
ICSE [82],[80],[165],[67],[92] 5/11
2023 CCS [182],[47],[32],[116] 4/9 20 of which publish data as a substitute because they do
NDSS [78],[65],[17] 3/4 not share their code or have no code to share. All software
S&P [108],[19],[104] 3/9
[149],[109],[186],[143],[41],[111], engineering conferences (ASE, FSE, and ICSE), USENIX
USENIX 12/29
[153],[2],[10],[161],[134],[96] Security, and CCS (since 2023) offer an artifact evaluation
ASE [58],[174] 2/4 process where independent reviewers assess the published
FSE [66],[189] 2/6
research artifact (for 2023, ASE and FSE have not yet
ICSE [97],[124],[89],[163],[64],[115],[151],[52] 8/17
2022 CCS [83],[12],[57],[144],[37],[29],[191] 7/8 published this data). Our analysis found that 36% (103) of
NDSS [84],[169],[180] 3/6 thepapersdidnothaveaccesstosuchanartifactevaluation;
S&P [74],[147],[102],[28],[100] 5/9
[148],[190],[183],[140],[120], 37% (107) had access but opted to not participate or failed
USENIX 9/19
[43],[185],[9],[27] to receive any badge. Only 23% (66) of the papers have
ASE [106],[81] 2/6 one or more badges. Of these, 64 are considered available
FSE [118],[181] 2/4 and 63 functional or reusable, a crucial requirement for
ICSE [16],[157],[131] 3/6
2021 CCS [61],[192],[122],[54],[72],[33] 6/13 reproduction.USENIXSecurityandCCSoffertoreproduce
NDSS [49],[136],[86] 3/6 the results of a paper, which only 16 out of 57 eligible
S&P [117],[40] 2/7
USENIX [91],[142],[55],[139] 4/13 papers achieved. We emphasize that artifact evaluation has
been introduced only in recent years, but participation is
ASE [125],[188] 2/4
FSE [22],[152],[145] 3/7 rising. CCS offered artifact evaluation for the first time in
ICSE [113],[164],[167],[158] 4/6 2023, further supporting this trend.
2020 CCS [171] 1/2
NDSS [87],[141],[162] 3/4
S&P [132],[6],[168],[75],[48],[34] 6/7 With 74%, a majority of works releases their code.
USENIX [150],[175],[137],[77],[59],[42], 11/19 Despitebeingrelativelynew,60%ofthepapersalready
[24],[93],[53],[194],[135]
hadaccesstoartifactevaluation,withadoptionlagging
ASE – 0/0
behind at 23% of papers that obtained a badge.
FSE [98] 1/4
ICSE [36],[126],[39],[172],[160] 5/7
2019 CCS [38],[31] 2/3
NDSS [69],[5],[7] 3/4 3.2.2. Targets under Test. To showcase the strengths of
S&P [121],[170],[146],[173],[79] 5/5
an approach, a suitable set of targets is required. Looking
USENIX [68],[35],[187],[85],[13],[110] 6/6
at the distribution of used targets (excluding datasets) in
ASE [95] 1/2
FSE – 0/0 Table 2, we find that they are strongly biased towards byte-
ICSE – 0/0 orientedfileformats,especiallybinutils.Onaverage,fuzzing
2018 CCS [25] 1/2
NDSS [26],[119] 2/2 papers evaluate on 8.9 targets. In summary, we found 753
S&P [60],[133] 2/3 differenttargetsusedacrossallstudiedpapers;ofthese,76%
USENIX [154],[129],[176] 3/3
(576) were evaluated in only one paper. In addition to real-
total#papersanalyzed 150/289
world targets, a common way of reproducibly measuring
∗limitedtoavailablepreprints
fuzzer performance is using benchmarks. Figure 1 shows
how benchmarks have been adopted in the past years. In
Weinvestigatewhetherthefuzzingevaluationguidelines
total, 61% (91) of the papers use no benchmark, 17% (26)
outlined in Section 2 are followed or whether an evaluation
useLAVA-M[51],10%(15)useFuzzBench[118],8%(12)
deviates from them. We want to stress that there may
useGoogle’sFuzzerTestSuite(FTS)[63],5%(8)DARPA’s
be good reasons to deviate from these guidelines, making
CGC binaries (CGC) [45], 4% (6) rely on Magma [70],
a manual review and judgment on a case-by-case basis
and 1% (2) build on Unibench [99] for benchmarking pur-
mandatory.Wealsostudywhethertheevaluationsperformed
poses.Despiteitssuccess,LAVA-Misnowadaysconsidered
expose flaws that future fuzzing papers could avoid.
flawed because it artificially injects vulnerabilities into a
given target program that are easy for a fuzzer to find but
3.2. Results
do not correspond to real bugs [18], [118], [162], [183].
More recent works using LAVA-M often do so only for
We study the papers regarding their reproducibility, tar-
comparability reasons [78], [82]. Similar to LAVA-M, CGC
gets, fuzzers, evaluation setup in terms of resources, com-
is widely considered outdated and inadequate.
mon metrics, and statistical evaluation.
Real-world targets are often limited to binary input-
3.2.1. Reproducibility. A crucial aspect of verifying and affineprograms,whilebenchmarksarenotusedbythe
advancing science is the ability to reproduce existing re- majority of papers. Benchmarks with artificial vulner-
search results. When examining the metadata we collected abilities are still used.
for all 289 fuzzing papers, we find that 74% (214) publish

TABLE2.TARGETSFUZZEDINFIVEORMOREANALYZEDPAPERS 25
(EXCLUDINGBENCHMARKS).SOMEPAPERSREPORTGENERICALLYTO
EVALUATEONBINUTILS,WHILEOTHERSSPECIFYEXACTTARGETS, 20
SUCHTHATNUMBERSINPRACTICEMAYDIFFERSLIGHTLY.
15
#Uses Target
25 objdump,readelf
10
20 nm,tcpdump
19 libpng
17 libtiff 5
13 cxxfilt,jhead,libjpeg
12 libxml2
0
11 nasm 2018(of9) 2019(of22) 2020(of30) 2021(of22) 2022(of36) 2023(of31)
10 jasper,libming,openssl,size
9 file,ImageMagick,mjs,tiff2pdf
8 djpeg,exiv2,JavaScriptCore,libarchive,SQLite,v8,xmllint
7 ChakraCore,ffmpeg,harfbuzz
6 binutils,lcms,lrzip,mupdf,OpenJPEG,SpiderMonkey
bento,bsdtar,catdoc,cflow,curl,freetype2,GraphicMagick,
5 json,pcre2,proj4,strip,tiff2ps,yara,zlib
3.2.3. Evaluation against State of the Art. Comparison
with a strong set of existing work helps to demonstrate
that a new method is particularly suited to solve a specific
problem. Yet, only a few techniques published in the past
few years have been broadly incorporated in follow-up
work. Instead, the most famous fuzzers extended with new
techniques are AFL [177] with 30% (45), AFL++ [56] with
6%(9),libFuzzer[101]with5%(7),andsyzkaller[50]with
4% (6). Interestingly, all of these tools are non-academic
works; only for AFL++ a peer-reviewed paper has been
published [56]. Contrasting this number, 33% (49) of the
proposed tools are not based on any existing tool.
When looking at the fuzzers chosen as baselines for
comparison, we find that AFL is compared against by 35%
(53) of studies, followed by QSym [176] with 15% (23),
AFLFast [15] with 14% (21), Angora [30] with 13% (20),
FairFuzz [95] with 8% (12), and AFL++ with 9% (14).
From the 150 papers we analyzed, only QSym (2018),
FairFuzz (2018), and MOpt [110] (2019) have been chosen
by more than five follow-up works for comparison. More
recently, only Fuzzilli [65] (published 2023, open-sourced
early2019)wasusedbymultipleworksfortheirevaluation,
even before the paper was published. This does not account
for techniques replicated in AFL++ or LibAFL [57], which
reimplementmanysuccessfultechniquesproposed[7],[15],
[90], [110]. On average, a fuzzing paper evaluates against
3.2 other fuzzers.
Analyzing whether papers omit comparing against a
relevant fuzzer in their evaluation, we find that 20% (30) of
theworksignoreatleastonerelevantstate-of-the-artmethod
and3%(4)evenomitcomparingagainsttheirbaseline,i.e.,
the tool on which they base their own fuzzer.
45%offuzzingresearchbuildsontopofnon-academic
fuzzers, 33% build a new tool. 23% percent of fuzzing
evaluations fail to compare against relevant state-of-
the-art fuzzers or their own baseline.
desusemit
24
CGC Magma Fuzzbench
LAVA-M FTS none 21
18
16
10
8
7 7
5
4 4 4
3 3 33
2 2 2 22 2
1 1 1 1 1 1
0 0 0 0 0 0 0 0
Figure 1. Benchmark usage over the years. The numbers in brackets
representthenumberofpapersanalyzedfortherespectiveyear.Notethat
somepapersusemultiplebenchmarks,hencethenumbersdonotaddup.
3.2.4. Evaluation Setup. With respect to the evaluation
setup, we analyze the runtime, the number of CPU cores
assigned, whether all resources were allocated fairly, and
the seeds used for the experiments.
Runtime. Reviewing the experiment setup used
across fuzzing evaluations, we find that the majority of
papers uses a runtime of 24h, more precisely 56% (84) of
the papers run at least one experiment for 24 hours. As
Figure2outlines,only27%(40)oftheworksusearuntime
of less than 23 hours, while 29% (44) use an even higher
runtime.5%(8)donotspecifytheirruntimeorhavenoown
experiments measuring time.
CPU cores. In terms of CPU cores assigned to
fuzzers, we find an inconsistent picture, with a significantly
varying number of CPU cores used. The most common
result was that 25% (38) of the papers did not specify how
many CPU cores they used, 27% (40) used one core, and
8% (12) used two cores.
Faircomputingresources.Whencheckingwhether
the available computing resources were allocated fairly
(e.g., the same number of cores were allocated to each
fuzzer and they were run for the same amount of time),
we find that this is the case for 74% (111) of the works.
For 15% (23), we could not infer this information from the
description in the paper, and 5% (8) did not evaluate other
fuzzers or did not conduct any experiments where this was
anissue.Crucially,5%(8)unfairlyallocateresources,giving
srepaP# 14
12
10
8
6
4
2
0
FDC
86 100%
84
82 80%
18
16 60%
40%
20%
0%
<1 1 2 3 4 5 6 8 1012232433344850546072>72
Runtime[hours]
Figure 2. Distribution of runtimes used in practice and cumulative distri-
butionfunction(CDF),whichshowsthat27%ofpapersusearuntimeof
lessthan23hours.26papersusemultiple,differentruntimes;weinclude
allinthesecases.

onefuzzeranadvantageoveranother.Forthese8,wefound the paper considers a path. Differences exist, for example,
one benign case in which an existing method was given between actual program paths and AFL’s path metric, re-
moreresources,onecaseinwhichthenumberofexecutions quiring any paper to specify what they consider a path for
wasfairlydistributedratherthantheruntime(therebygiving their work. Beyond the type of coverage, the process of
slow fuzzers an advantage), two cases in which a different measuringcoverageisalsopronetoerrors,andtheconcrete
number of cores was used (in one case, giving the new choiceofmeasurementisoftennotdocumented.Intotal,we
fuzzertwicethecoresthanothers),andfourcaseswherethe find that 45% (67) of the works lack a clear definition or
new approach was allowed some preprocessing time, e.g., explanation of how they measure coverage, whereas 32%
forsomestaticanalysispassorseedpreprocessing,beforeit (48) document this (the remaining papers do not measure
wasthenallottedthesametimeasallothertools,effectively coverage). For example, measuring coverage using a binary
giving it more computation time. Unfortunately, the authors withinstrumentationthatnotallfuzzershadaccesstoduring
rarely explain their motivation for doing so, nor do they thefuzzingcampaigngivessomefuzzersanadvantage.Sim-
considerconsequencesfortheevaluation.Also,ouranalysis ilarly,whenmeasuringcoverageonabitmapwithcollisions,
does not address manual work, which may be distributed thereportedcoverageisupto9%smaller[103]thanthetrue
unfairly between different fuzzers, for example, giving one one. This may cause problems when a different bitmap size
fuzzer a fine-tuned configuration that performs better. was used during fuzzing, as the inputs saved by a fuzzer
Initial seeds. Another crucial factor determining a may no longer trigger the new coverage on the bitmap with
fuzzer’sperformanceisthesetofinitialseeds[73],[88].We collisions. A further pitfall affects emulation-based fuzzing,
studied if the type of seeds is specified and if information especiallywhenusingQEMU[11].Weobservedthatpapers
on concrete seed files is available. Out of the 150 papers, often provide no clear distinction between translated blocks
11% (16) require no seeds, 25% (38) use uninformed or aspresentedbytheemulatorandactualbasicblocksforthe
empty seeds, 20% (30) use informed seeds, 16% (24) use target binary. We found that in at least one case this led
seeds provided by the project as test cases or those that are to overcounting the reached coverage, as translated blocks
shipped with a benchmark, and 3% (5) use multiple types were mistaken for basic blocks.
of seed sets, while 25% (37) do not specify at all what Known Bugs. As research from Klees et al. [88]
type of seeds are used, making a reproduction challenging. as well as Böhme et al. [23] points out, coverage may not
Regarding concrete details, we find that 50% (75) of the be an accurate proxy for bug finding, even though a strong
papers fail to disclose what seeds they use, compared to correlationexists.Ultimately,afuzzer’sgoalisfindingbugs,
39%(59)thatoutlinetheirseeds.Afurtherpitfallpotentially making the evaluation of whether it can find known or
threateninganevaluation’svalidityisthefairdistributionof unknown vulnerabilities an excellent experiment. Known
the same seeds to all fuzzers. While this is the case in 46% bugs are a good way of measuring a fuzzer’s performance,
(69) of the studied papers, in 30% (45) of the works this yet it is difficult to find suitable bugs outside well-designed
does not become clear, and 5% (8) even use diverging seed benchmarks, such as Magma [70] or RevBugBench [183].
sets. Three of these cases arise due to the fuzzer design or New Bugs / CVEs. Another commonly used ap-
other fuzzers lacking the capability to process a particular proachisthecapabilityoffindingpreviouslyunknownbugs.
typeofinput.Westressthatthismaybevalid,forexample, Ethicalhandlingrequiresresearcherstoresponsiblydisclose
when a fuzzer used for comparison needs a larger seed set these bugs to the vendors or maintainers. Both sides can
than the proposed fuzzer, yet giving a fuzzer a different set additionallyrequestaCVEthatservesasauniqueidentifier
of seeds requires special attention and documentation. for the found vulnerability. In practice, CVEs have become
a commonly used metric to assess whether a fuzzer can
We find that 5% of the papers allocate computing
find bugs in real-world software, presumably showing its
resources unfairly, and 5% use different seed sets.
impact. Of the 150 analyzed papers, 59 claim one or more
CVEs (9.7 on average, 662 in total). Given the implicit
3.2.5. Evaluation Metrics. While many different metrics expectation of submissions to have a real-world impact, the
exist, often specific to the particular technique introduced, authors often try to obtain as many CVEs as possible. We
a small number of metrics has found widespread adoption: randomly selected 35 of these papers [9], [19], [33], [35],
77% (115) of the papers use some sort of code coverage, [36], [40], [41], [47], [49], [52], [72], [75], [77], [82], [93],
and 71% (107) use the (re-)discovery of bugs as a metric to [96], [97], [105], [106], [109], [110], [115], [120], [129],
compare fuzzers. The third most widespread metric, Time- [130], [139], [146], [160], [164], [174], [175], [184]–[186],
To-Exposure (TTE), is used by 13% (20) of the papers, [189] and analyze the 339 CVEs they claim (51% of all
mainly from the directed fuzzing domain. CVEs claimed across the 59 papers).
Code Coverage. Code coverage comes in different As Figure 3 shows, surprisingly, only 43% (145) of the
forms; the most popular are the following: 19% (29) of CVEs are valid (i.e., neither formally disputed, reserved,
the papers use branch coverage, 17% (25) employ edge norignoredorrejectedbytheprojectmaintainers)andhave
coverage, 13% (19) rely on basic block coverage, and 5% been fixed (or at least acknowledged). 26% (88) of the
(8)uselinecoverageonthesourcecodelevel.Furthermore, CVEs were still marked as RESERVED, preventing us from
11%(17)usesomenotionofpathstomeasurecoverage.We viewing and analyzing them (all of them were assigned be-
stress this metric is unreliable without a definition of what fore 2023). For such CVEs and depending on the assigning

|     |     |     |     |     |     |     | In summary, |     | the need | to  | show a fuzzer’s |     | real-world | im- |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --- | -------- | --- | --------------- | --- | ---------- | --- |
pactresultsinalargenumberofunwarrantedCVEs,leading
Fixed:143
Acknowledged:145
|     |     |     |     |     |     |     | to a situation |       | where only | 42%  | (143)  | of the | 339  | assigned |
| --- | --- | --- | --- | --- | --- | --- | -------------- | ----- | ---------- | ---- | ------ | ------ | ---- | -------- |
|     |     |     |     |     |     |     | CVEs are       | valid | and have   | been | fixed, | while  | many | are what |
Unfixed:2
onemaintainerreferredtoas“fuzzerfakeCVEs”[114].Cre-
Reported:339
|     |     |             |     |             |     |     | ating such | invalid | vulnerabilities |     | causes | multiple | problems: |     |
| --- | --- | ----------- | --- | ----------- | --- | --- | ---------- | ------- | --------------- | --- | ------ | -------- | --------- | --- |
|     |     | Reserved:88 |     | Reserved:88 |     |     |            |         |                 |     |        |          |           |     |
Itunnecessarilyalertspeople,reducesmaintaineracceptance
offuzzerfindings,andraisestheexpectationsforsubsequent
|     |     | Ignored:69 |     | Other/Unknown:55 |     |     |           |      |           |        |                     |     |     |     |
| --- | --- | ---------- | --- | ---------------- | --- | --- | --------- | ---- | --------- | ------ | ------------------- | --- | --- | --- |
|     |     |            |     |                  |     |     | papers to | find | a similar | number | of vulnerabilities. |     |     |     |
Projectinactive:14
Invalid:37
|     |     |     |     | Bugrejected:17 |     |     | 20%      | of the | CVEs         | have | been ignored |           | and     | remain |
| --- | --- | --- | --- | -------------- | --- | --- | -------- | ------ | ------------ | ---- | ------------ | --------- | ------- | ------ |
|     |     |     |     |                |     |     | unfixed, | 11%    | are invalid. |      | 26% are      | reserved, | eluding |        |
Duplicate:18
|     |     |     |     | Disputed:1 |     |     | analysis. |     |     |     |     |     |     |     |
| --- | --- | --- | --- | ---------- | --- | --- | --------- | --- | --- | --- | --- | --- | --- | --- |
WrongCVEID:1
|     |     |     |     |     |     |     | 3.2.6. Statistical |     | Evaluation. |     | To confirm |     | the results | ob- |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | ----------- | --- | ---------- | --- | ----------- | --- |
Figure3.Outcomeof339CVEsthatwerereportedacross35papers.Only
|            |                |              |     |                 |     |         | tained in   | the evaluation, |             | a   | statistical | evaluation |     | is highly |
| ---------- | -------------- | ------------ | --- | --------------- | --- | ------- | ----------- | --------------- | ----------- | --- | ----------- | ---------- | --- | --------- |
| 43% of the | CVEs have been | acknowledged | by  | the developers. |     | Pending |             |                 |             |     |             |            |     |           |
|            |                |              |     |                 |     |         | recommended |                 | [88], [127] | to  | detect      | whether    | the | observed  |
publicdisclosure,informationonCVEsintheReservedstateiswithhold.
|           |                |          |          |                 |           |         | difference  | is significant |              | or by     | chance. | In practice, |            | the most |
| --------- | -------------- | -------- | -------- | --------------- | --------- | ------- | ----------- | -------------- | ------------ | --------- | ------- | ------------ | ---------- | -------- |
|           |                |          |          |                 |           |         | common      | approach       | is to        | compare   | the     | final        | coverage   | values   |
|           |                |          |          |                 |           |         | achieved    | by different   | fuzzers      |           | across  | multiple     | runs.      |          |
| authority | (called CNA),  | authors  | usually  | have            | to follow | up      |             |                |              |           |         |              |            |          |
|           |                |          |          |                 |           |         | In general, |                | a frequently | used      | test    | for the      | comparison | of       |
| with the  | CNA to unblind | them     | once the | vulnerabilities |           | are     |             |                |              |           |         |              |            |          |
|           |                |          |          |                 |           |         | the means   | of two         | sample       | sets—such |         | as the       | coverage   | values   |
| publicly  | disclosed. Our | analysis | found    | 11% (37)        | of        | invalid |             |                |              |           |         |              |            |          |
CVEs, including both CVEs that were formally disputed of two fuzzers operating on the same target—is the t-test.
|                                     |               |     |               |      |              |         | While powerful      |     | for the | detection   | of  | differences, | it   | requires |
| ----------------------------------- | ------------- | --- | ------------- | ---- | ------------ | ------- | ------------------- | --- | ------- | ----------- | --- | ------------ | ---- | -------- |
| or rejected                         | as duplicates | by  | the assigning | CNA, |              | such as |                     |     |         |             |     |              |      |          |
|                                     |               |     |               |      |              |         | strong assumptions. |     | In      | particular, | the | samples      | have | to be    |
| MITRE,andsuchCVEswherethemaintainer |               |     |               |      | oftheproject |         |                     |     |         |             |     |              |      |          |
approximatelynormallydistributed.Thisisparticularlytrue
consideredthereporttobeinvalidornotabug.Inonecase,
the CVE ID specified in the paper did not match the target, for small sample sizes, such as n ≈ 10. To avoid these
strongassumptions,theMann-WhitneyorthesimilarU-test
| leading us | to believe | the authors | mistakenly |     | reported | the |     |     |     |     |     |     |     |     |
| ---------- | ---------- | ----------- | ---------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
(calledMann-WhitneyU-testtoemphasizetheirequivalence
wrongnumber.ThreeCVEswereclaimedbymorethanone
subsequently[138])isoftenused.Here,thetwosamplesare
| paper, raising | questions | about | who identified |     | and reported |     |     |     |     |     |     |     |     |     |
| -------------- | --------- | ----- | -------------- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
theminitially.Alargernumber,20%(69)oftheCVEs,have assumed to have the same unknown distribution except for
been ignored by the maintainers of the respective projects. a potential shift. The test statistics for the Mann-Whitney
|                |          |       |            |         |     |          | U-test is | mainly | based         | on  | the sum      | of ranks | of     | the two |
| -------------- | -------- | ----- | ---------- | ------- | --- | -------- | --------- | ------ | ------------- | --- | ------------ | -------- | ------ | ------- |
| Investigating  | this, we | found | that in 14 | cases,  | the | projects |           |        |               |     |              |          |        |         |
|                |          |       |            |         |     |          | samples   | in the | joint sample. |     | This results | in       | a test | for the |
| were abandoned | several  | years | before     | the bug | was | found,   |           |        |               |     |              |          |        |         |
or the projects had not found widespread adoption (with a difference of distribution medians, which is rather robust
single digit number of stars and forks on GitHub). In these w.r.t. assumptions that do not hold. For a more detailed
|     |     |     |     |     |     |     | discussion | of such | tests, | we  | refer to | Sachs’ | work | [138]. |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ------- | ------ | --- | -------- | ------ | ---- | ------ |
cases,theperceivedneedtoreportmanyvulnerabilitiesina
However,theMann-WhitneyUtestcanhavelowpower,
| paper appears | to be the | driving | factor | in requesting |     | a CVE |     |     |     |     |     |     |     |     |
| ------------- | --------- | ------- | ------ | ------------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
for such bugs. especially for small sample sizes. Suppose, for example,
|               |                |          |               |             |           |         | that we             | have two  | samples | of  | three               | runs that | achieved | the |
| ------------- | -------------- | -------- | ------------- | ----------- | --------- | ------- | ------------------- | --------- | ------- | --- | ------------------- | --------- | -------- | --- |
| Studying      | why some       | bug      | reports       | were        | ignored   | while   |                     |           |         |     |                     |           |          |     |
|               |                |          |               |             |           |         | following           | coverage: |         |     |                     |           |          |     |
| other bugs    | were fixed,    | we found | that          | maintainers |           | tend to |                     |           |         |     |                     |           |          |     |
| ignore issues | such as memory |          | leaks in      | client-side | software, |         |                     |           |         |     |                     |           |          |     |
|               |                |          |               |             |           |         | x=(1000,1002,1001), |           |         |     | y =(1208,1207,1205) |           |          |     |
| for example,  | an assembler.  |          | The reasoning |             | appears   | to be   |                     |           |         |     |                     |           |          |     |
that the program does not run continuously and is not As is easy to see, these samples are strongly separated, and
exposed to external attackers. Many of the ignored CVEs it is hard to explain these results assuming the similarity
were segmentation faults in mjs or yasm. The bug tracker of the samples’ distributions. Yet, the Mann-Whitney U
mjs
of appears to be flooded with similar fuzzer-generated test will not reject the hypothesis of no difference for a
bug reports, while the project has not received an update significance level α=5%. Even worse, it will never reject
fortwoyears.Similarly,themaintainerofyasmhasmoved samples of this size, since it only uses the ordering of
to other projects, only occasionally merging pull requests. the observations, and the probability of two samples of
As security researchers usually only drop the bug details size 3 generated from the same distribution to show this
without proposing a patch, these issues remain unfixed. pattern of full separation on the real line has a probability
Whilestudyingpapers,wenoticedthatseveralpapersclaim >5%. In other words, we cannot use the Mann-Whitney U
a specific number of CVEs credited to their work but do test to statistically confirm that the difference between two
not specify any identifier, making it difficult to track them. fuzzersissignificantifonlythreetrialshavebeenconducted.
Interestingly, 18 of the 35 papers report only CVEs that all Such situations frequently arise if sample sizes are small
have been fixed, accounting for 67 of the CVEs. or observations cannot be approximately described by a

|     |     |     |     |     |     | 100% | 4.  | Artifact | Evaluation |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ---- | --- | -------- | ---------- | --- | --- | --- | --- | --- | --- |
40
| 35  |     |     |     |     |     | 80% |     |        |          |     |            |          |     |               |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | -------- | --- | ---------- | -------- | --- | ------------- | --- |
|     |     |     |     |     |     |     |     | Beyond | studying | the | evaluation | outlined |     | and described |     |
30
srepaP#
25 60% FDC inthepapers,weselecteightpapersandstudytheirartifacts.
| 20  |     |     |     |     |     |     | This | allows | us  | to assess | the | practical | reproducibility |     | of  |
| --- | --- | --- | --- | --- | --- | --- | ---- | ------ | --- | --------- | --- | --------- | --------------- | --- | --- |
15 40% fuzzing research and provide recommendations grounded in
10 practice. As selection criteria, we pick four recent papers
| 5   |     |     |     |     |     | 20% |     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
from2023andfocusonsecurityvenuesfeaturinganartifact
0
1 3 4 5 6 8 10 12 15 16 20 24 30 40 evaluation. In our experience, papers undergoing an artifact
Repetitions evaluation process provide enhanced documentation and
|     |     |     |     |     |     |     | significantly |     | ease | the process | of  | setting | up a particular |     | tool. |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ---- | ----------- | --- | ------- | --------------- | --- | ----- |
Figure4.Distributionoftrialsusedinpracticeandcumulativedistribution
function (CDF). 8 papers use a different number of trials for different However, we test papers that have not undergone artifact
experiments;weincludeallnumbersinthiscase.Further21papersfailto evaluation as well to gain a more complete picture. Note
specifythenumberoftrials. that all papers we chose as case studies had attracted our
|     |     |     |     |     |     |     | attention | during |     | the initial | reading | for | the literature |     | survey |
| --- | --- | --- | --- | --- | --- | --- | --------- | ------ | --- | ----------- | ------- | --- | -------------- | --- | ------ |
parametricdistributionthatdependsonlyonfewparameters, in terms of evaluation setup or execution.
|             |        |               |            |     |        |              |        | In the  | following, | we        | discuss | our    | lessons | learned, | pit- |
| ----------- | ------ | ------------- | ---------- | --- | ------ | ------------ | ------ | ------- | ---------- | --------- | ------- | ------ | ------- | -------- | ---- |
| such as a   | normal | distribution. |            |     |        |              |        |         |            |           |         |        |         |          |      |
|             |        |               |            |     |        |              | falls, | and how | fuzzing    | artifacts |         | can be | further | improved | to   |
| In summary, |        | a statistical | evaluation |     | should | use a suffi- |        |         |            |           |         |        |         |          |      |
cient number of trials, ideally 10 or more, and use a robust enhancetheirreproducibility.Again,weemphasizethatitis
test. Studying the trials used in the 150 analyzed papers, notourintentiontopointfingersatspecificworksbutrather
we find that 1, 3, 5, 10, or 20 trials are the most common to highlight potential pitfalls that researchers in this area
|     |     |     |     |     |     |     | should | be  | aware | of. More | information |     | on all | case studies | is  |
| --- | --- | --- | --- | --- | --- | --- | ------ | --- | ----- | -------- | ----------- | --- | ------ | ------------ | --- |
repetitionschosen.Figure4providesadetaileddistribution.
|              |      |        |        |           |      |              | available | in  | dedicated | reproduction |     | repositories |     | on GitHub: |     |
| ------------ | ---- | ------ | ------ | --------- | ---- | ------------ | --------- | --- | --------- | ------------ | --- | ------------ | --- | ---------- | --- |
| Overall, 55% | (83) | of the | papers | use fewer | than | 10 trials in |           |     |           |              |     |              |     |            |     |
at least one experiment (8 papers use a different number of https://github.com/fuzz-evaluator/. Despite our best efforts,
trialsthroughouttheirpaper).Evenworse,63%(94)conduct ourreproductionmaycontainerrors.Ifwebecomeawareof
|                |      |         |          |      |        |            | any, | we will | update | the respective |     | reproduction |     | repositories |     |
| -------------- | ---- | ------- | -------- | ---- | ------ | ---------- | ---- | ------- | ------ | -------------- | --- | ------------ | --- | ------------ | --- |
| no statistical | test | at all. | Only 37% | (55) | of the | papers run |      |         |        |                |     |              |     |              |     |
on GitHub.
| a Mann-Whitney |     | U test | to measure | statistical | significance, |     |     |     |     |     |     |     |     |     |     |
| -------------- | --- | ------ | ---------- | ----------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
which—pairedwithfewtrials—risksthatitmayneverreject Author Contact. We have anonymously contacted
the hypothesis. We find that 15% (22) of the analyzed the authors of all case studies and brought up our findings
fordiscussionwiththem,askingfortheirhelp,confirmation,
| papers conduct | a   | Mann-Whitney |     | U-test | while | having five |     |     |     |     |     |     |     |     |     |
| -------------- | --- | ------------ | --- | ------ | ----- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
or less trials. One work reports p-values without specifying or clarification. Five groups have responded to our mails.
how they have been derived. Interestingly, we found no Where desired by the authors, we publish a statement of
othertests,suchasbootstrap-basedones,beingused,despite them alongside our reproduction artifact.
beingrecommendedbyKleesetal.[88].Beyondmeasuring Setup. All our experiments were performed on two
statistical significance, it is recommended to quantify the servers running Ubuntu 22.04 with 196 GB RAM, one with
effect size, for example, using Vargha and Delaney’s Aˆ an Intel Xeon Gold 6230R CPU with 52 cores at 2.10GHz,
12
test[156].Yet,wefindthatonly10%(15)ofstudiesconduct and the other with an Intel Xeon Gold 6230 CPU with 40
this test; 2% (3) rely on other means to specify the effect cores at 2.10GHz (for consistency, a case study was fully
size,leavinguswith88%(132)notusinganytesttomeasure run on one type of server or the other). We use the settings
the effect size. provided by the original papers where sensible, otherwise
Beyond the use of statistical tests, we find that 73% we run 10 trials for 24 hours each, restricting each fuzzer
| (109) of     | the papers | provide    | no             | measure   | of             | uncertainty, | to   | a single  | core.   |            |            |         |             |           |     |
| ------------ | ---------- | ---------- | -------------- | --------- | -------------- | ------------ | ---- | --------- | ------- | ---------- | ---------- | ------- | ----------- | --------- | --- |
| for example, | intervals  | in         | coverage       | plots     | or the         | standard     |      |           |         |            |            |         |             |           |     |
| deviation.   | This makes | it         | difficult      | to assess | the            | robustness   |      |           |         |            |            |         |             |           |     |
|              |            |            |                |           |                |              | 4.1. | Case      | Study:  | Artificial |            | Runtime | Environment |           |     |
| of reported  | results,   | especially | considering    |           | the inherent   | ran-         |      |           |         |            |            |         |             |           |     |
|              |            |            |                |           |                |              | and  | Unique    | Crashes |            |            |         |             |           |     |
| domness      | in fuzzing | runs.      |                |           |                |              |      |           |         |            |            |         |             |           |     |
| 63% of       | the works  | use        | no statistical |           | test to assess | their        |      |           |         |            |            |         |             |           |     |
|              |            |            |                |           |                |              |      | Our first | case    | study      | is MemLock |         | [164],      | published | at  |
results, and 15% use too few trials to achieve robust ICSE’20,whichproposestousememoryusageasadditional
| outcomes.      | 73% | provide   | no measure |     | of uncertainty. |       |            |          |           |         |         |             |             |          |          |
| -------------- | --- | --------- | ---------- | --- | --------------- | ----- | ---------- | -------- | --------- | ------- | ------- | ----------- | ----------- | -------- | -------- |
|                |     |           |            |     |                 |       | feedback.  |          | This way, | the     | paper   | aims        | to identify | resource |          |
|                |     |           |            |     |                 |       | exhaustion |          | bugs,     | such as | stack   | exhaustion. |             |          |          |
|                |     |           |            |     |                 |       |            | Artifact |           | status. | MemLock | has         | undergone   |          | artifact |
| 3.2.7. Threats | to  | Validity. | Scientific |     | works often     | use a |            |          |           |         |         |             |             |          |          |
dedicatedsectiononthreatstovaliditytoenumerate,reflect, evaluation and received the available and reusable badges.
and address any issue that could potentially render their Our additional experiments can be found at https://github.
evaluation invalid. However, when studying how many of com/fuzz-evaluator/MemLock-Fuzz-eval.
the 150 analyzed papers provide such a section, we find Observations. After studying the paper and artifact,
that only a minority of 20% (30) of the papers does so. we observe the following:

1) According to the artifact but not documented in the 4.2. Case Study: Exaggerated Vulnerabilities
paper, the authors artificially alter the runtime envi-
ronment of one target and lower the maximum stack Forthenextcasestudy,weselectedSoFi[72],published
size. Manually limiting the stack size makes it easier at ACM CCS’21. This work aims to use a reflection-based
totriggerstackoverflowbugs,oneofthedeclaredgoals analysis to create a syntactically and semantically valid but
of the presented technique. diverse set of seeds for fuzzing JavaScript engines.
2) MemLock,similartomanyotherfuzzingpapers,relies Artifactstatus.Artifactevaluationwasnotavailable
on unique crashes as reported by AFL to draw con- for SoFi, but the authors released the source code via an
clusions on the fuzzer’s performance. This metric is independent web page [71]. While trying to set up the
generally unreliable since a unique crash depends on artifact, we noticed that crucial parts of the source code
thesetofexercisededges;itdoesnotreflectthenumber were missing. The authors stated they would release the
of actual bugs. Here, MemLock’s use of the call stack missing pieces once the code is polished [71], but did not
depth as additional feedback may lead to an inflated react to our e-mails asking for access to the code. Without
number of “unique” crashes per root cause. a chance to reproduce the artifact, we solely studied the
3) To demonstrate practical impact, MemLock reports 26 paper, in particular the reported vulnerabilities summarized
CVEs. We found multiple cases among them where up in Table 2 of their paper, entitled “Summary of discovered
to five CVEs were requested and assigned for a single vulnerabilities” [72].
bugreport,towhichnoneofthemaintainersresponded. Observations. We find that all seven vulnerabilities
4) MemLock’s artifact is based on PerfFuzz [94] (itself claimed in the actively used modern browser engines (i.e.,
an AFL-derivative), but the paper suggests it is based v8,SpiderMonkey,andJavaScriptCore)areinvalidandhave
on AFL. been rejected by the respective developers, six out of seven
evenbeforetheconferencesubmissiondeadline.WhileSoFi
We design three experiments to analyze and reproduce managestofindconfirmedvulnerabilitiesinotherprograms,
MemLock’s performance. For full details, we refer the in- webelieveitisimportanttonotoversellresultsbyclaiming
terested reader to our reproduction artifact. to have found vulnerabilities in browser engines, when in
Experiment 1: Artificial Runtime Limits. We first facttheywerenotabugatall.Weassumethatthebugreport
study the impact of artificially lowering the stack size for IDs were blinded, as is common practice for submission,
the target flex, which was not documented in the paper. such that the reviewers could not verify the validity of the
Afterrecreatingthesetupandrunningthefuzzingcampaign presumed vulnerabilities.
with and without the artificial limit, we observe that Mem-
Lessons learned: We highly discourage marketing in-
Lock finds the claimed crashes only with the artificially
valid bug reports as a vulnerability. Feedback from the
lowered limit. While memory corruption bugs may warrant
developers must be taken into account (especially if
discussing artificial scenarios, we believe memory exhaus-
bug reports are rejected by the developers). Pledges to
tion created through artificial limits cannot be considered
release the source code should be kept.
realistic. In any case, we recommend documenting such
limits in the paper.
Experiment 2: Unique Crashes. We investigate 4.3. Case Study: Missing Baselines
whether superiority claimed due to unique crashes persists
DARWIN [78] was published at NDSS’23 and honored
whenexaminingtheunderlyingbugsandrootcauses.Using
with a Distinguished Paper Award. The paper focuses on
a developer patch and manual triaging, we identify the
improving mutation scheduling. More specifically, the au-
underlying bugs for three evaluation targets and find that
thors propose to use an evolution strategy and dynamically
AFL finds four bugs, while MemLock locates only three,
adapt the mutation selection to the target under test.
even though it finds significantly more unique crashes.
Artifactstatus.Artifactevaluationwasnotavailable
Experiment3:ReportedCVEs.Whenstudyingthe
to DARWIN, but the authors publicly released an artifact.
reported vulnerabilities, we noticed that six CVEs, CVE-
Our reproduction artifact is available at https://github.com/
2020-36370 to CVE-2020-36375, refer to the same bug in
fuzz-evaluator/DARWIN-eval.
mjs. This bug was never acknowledged by the maintainers Observations. Analyzing the paper and artifact, we
of mjs. This pattern repeats for other groups of CVEs.
found a number of issues:
1) Coverage differences between DARWIN and tested
Lessons learned: Unique crashes are not a reliable
baselines on FuzzBench are not statistically significant
metric; instead, we suggest using (known) bugs. We
nor consistent with the paper’s FuzzBench results.
recommend not using artificial runtime environments
2) TheresultsonMOpt[110]listedintheDARWINpaper
without good reason and, if done, documenting such
indicate that the port implemented for MOpt may have
limits. We strongly recommend against the practice
erroneously restricted the number of usable mutations.
of obtaining as many CVEs as possible. Real-world
We find that this strongly influences the results.
impact should not be measured based on the number
3) The artifact appears to be based on Git tag 2.55b of
of assigned CVEs.
Google’sAFLforkandnot2.54b,aslistedinthepaper.

4) The artifact does not provide the AFL 2.55b port for seed mutation scheduling on seven targets: we found that
MOptortheirbaselineAFL-S,preventingreproduction disabling the per-seed mutations slightly improved perfor-
or analysis. mance overall, leading to higher median coverage in some
WedesignthreeexperimentstoanalyzeDARWIN.More targets,butnotstatisticallysignificantlysoforanytargetby
experiments and details are available in our artifact. Mann-Whitney U. We have used the author-recommended
Experiment 1: Coverage. We use FuzzBench to configuration (no -p flag) for Experiments 1 and 2.
reproduceDARWIN’scoveragemeasurements(inparticular,
Lessonslearned:Abaselinesuitedtotesttheproposed
TableIIIoftheirpaper).Runningalltargetsfor24hours,we
techniqueisnecessarytodetectdifferencesthatcanbe
compareitagainstAFL2.55bandMOpt,whichisbasedon
attributedtotheproposedtechniqueratherthanthenew
AFL2.52b.Notably,wedonotuseDARWINasconfigured
fuzzer implementation as a whole. We further recom-
inFuzzBenchbutfollowtheauthor’srecommendedconfigu-
mend publishing all evaluation artifacts, also including
ration (see Experiment 3). In our FuzzBench results, MOpt
benchmarking reports and raw data.
does not show the major performance degradation shown
in the paper results. Overall, FuzzBench ranks DARWIN
aboveMOptandAFL,bothbyscoreandrank.Inindividual
targets,DARWINisthebestperformerinnineofthetargets, 4.4. Case Study: Non-reproducible Measurements
but only with statistical significance in four. Our results
show the difference between DARWIN and its baselines to ArecentpaperpublishedatUSENIX’23,FuzzJIT[161],
be less than reported in Table III of their paper. Where they aims to detect bugs in JIT compilers, including those used
find DARWIN’s median relative coverage to be the highest in modern browsers.
for 15 out of 19 targets, we find this to be the case for Artifact Status. FuzzJIT underwent artifact evalua-
4 out of 18 targets2 (DARWIN is worse than at least one tion and was awarded the available and functional badges.
baseline in two cases and tied with at least one baseline in Our reproduction artifact can be found at: https://github.
the other cases). Note that the original paper evaluates over com/fuzz-evaluator/fuzzjit-eval.
a six hour period instead of the 24 hours recommended by Observations. After studying the paper and testing
Kleesetal.[88].Whileweprovidethestatisticaldataforthe the artifact, we observe several shortcomings:
24hourdatahere,weemphasizethattheresultsreportedin 1) Coverage does not reproduce as outlined in the paper;
thepaperforthesixhourmarkaresimilarlynotreproducible in our experiments, FuzzJIT performed worse than
and invite the reader to view our full evaluation report data Fuzzilli on all targets.
available on GitHub. 2) Reported improvements of the semantic correctness
Insummary,ourresultsshowasimilartendencytotheir rate did not materialize in our experiments.
paper,butthedifferenceobservedbetweenDARWINandits 3) It is not possible to study the bugs found because the
baselines is smaller. Notably, DARWIN reports a coverage time frame, engine versions, and resources spent were
improvement of only 1.73% over AFL, making it difficult not specified in the paper, hindering fair reproduction.
to judge the difference between these fuzzers meaningfully.
We design two experiments to analyze the claims of
Experiment 2: New Baseline. We propose a sec-
FuzzJIT in more detail.
ond baseline to test DARWIN’s contribution of a dynami-
Experiment 1: Code Coverage. When trying to
cally adapting mutation selection: we replaced its proposed
reproduce code coverage, we find significantly different re-
weighting with a random selection (that is reweighted at
sults.AsshowninTable3,FuzzJITreportsacodecoverage
a constant interval). This implementation, DARWIN ,
RAND improvement of up to 33% over Fuzzilli. In stark contrast,
provides a new baseline that allows to better judge DAR-
WIN’s contribution, as any improvement can be directly
attributed to DARWIN’s evolutionary algorithm rather than TABLE3.COMPARINGTHECODECOVERAGEREPORTEDBYFUZZJIT
other fuzzer implementation details, such as dynamically TOOURMEASUREMENTS.
adapting mutation selection. We find in our FuzzBench re-
sults no statistical significant difference between DARWIN Reported Measured
Engine Fuzzilli FuzzJIT Rel.Increase Rel.Increase
and DARWIN , meaning we were unable to demon-
RAND
strate that the evolutionary aspects of DARWIN’s approach JSC 16.47% 21.90% 33% -2%
V8 13.82% 16.67% 21% -3%
significantly contributed to the improvement compared to
SM 15.53% 17.97% 16% -12%
randomly changing mutation selection over time.
Experiment 3: Per-Seed Mutation Scheduling. TABLE4.COMPARINGTHESEMANTICCORRECTNESSRATEREPORTED
After contacting the authors, they noted that the per-seed BYFUZZJITTOOURMEASUREMENTS.
mutation scheduling (-p flag) set by FuzzBench should be
disabled for the evaluation because it worsens performance FuzzJIT Fuzzilli
Engine Reported Measured Reported Measured
and was not intended as part of the paper. To confirm this,
we separately evaluated DARWIN with and without per- JSC 90.33% 65.88% 62.80% 66.56%
V8 97.04% 63.67% 64.34% 66.74%
SM 93.28% 63.93% 64.13% 67.47%
2.FuzzBenchhasmeanwhileremovedthetargetphp.

our experiments show a code coverage decrease of -2% AFL AFLFast.new
to -12%. Despite searching for the cause, we find none nm objdump
explaining this difference. We speculate that the negative
outcome of the comparison experiment is a consequence
of benchmarking with different versions of Fuzzilli. This
is based on the observation that the state-of-the-art fuzzers
compared to in the evaluation are taken from UniFuzz [99],
which usesan outdated versionof Fuzzilli; FuzzJITitself is
based on a more recent version of Fuzzilli. Unfortunately,
the authors have not responded to our request for help.
Experiment 2: Semantic Correctness Rate. Be-
sides code coverage, FuzzJIT also evaluates the semantic
correctness rate of generated samples, i.e., the number of
samplesthatdonotraiseanuncaughtexceptionduringexe-
nm
cution in the JS engine. As shown in Table 4, we could not
measure any improvement of the semantic correctness rate,
contrasting the paper’s claim of a significant improvement.
Lessons learned: Relying on outdated baseline ver-
sionscancreateadistortedpictureofafuzzer’sperfor-
mance. Authors should ensure that they use the latest
version of all tools for comparison.
4.5. Case Study: Uncommon Metrics
Published at USENIX’20, EcoFuzz [175] proposes to
replace AFL’s seed scheduling algorithm with a version
relying on the adversarial multi-armed bandit model. This
way, EcoFuzz finds more paths while generating less seeds.
Artifact status. EcoFuzz has undergone artifact
evaluation and was awarded the passed badge, indicating
that the artifact is available and ready to be reproduced.
Our independent reproduction repository is located online
at https://github.com/fuzz-evaluator/EcoFuzz-eval.
Observations.Whenstudyingthepaperandartifact,
we noticed that the evaluation deviates from typical fuzzing
evaluations: The work does not report achieved code cover-
ageovertime.Instead,thepapervisualizesthetotalnumber
of paths discovered over executions. This aligns with the
paper’s goal of finding more path (bandits in EcoFuzz’s
multi-armed bandit model) with fewer executions (trials in
the model). The presented results may lead readers to infer
that a higher number of total paths equates to higher code
coverage, which is not necessarily true.
Experiment: Code Coverage. We design an exper-
iment in FuzzBench where we compare EcoFuzz against its
best-performingcompetitor,AFLFast,anditsbaseline,AFL.
We test these fuzzers on three targets, nm, libpng, and
objdump, where the original evaluation3 found EcoFuzz
to be the fuzzer to find the most paths. Our results, shown
in Figure 5, demonstrate that EcoFuzz achieves less code
coverage than the other fuzzers in all scenarios, except for
a statistically insignificant one, where it performs similar
to AFLFast on libpng. This underlines that finding more
paths does not necessarily translate to achieving a higher
3.Theevaluationusedreadpng,whichinternallyuseslibpng,while
weuselibpng_read_fuzzerasbundledwithFuzzBench.
shtaplatotforebmuN shtaplatotforebmuN
FidgetyAFL FairFuzz AFLFast EcoFuzz
9000 8000
8000 7000
7000 6000
6000
5000
5000
4000
4000
3000
3000
2000 2000
1000 1000
0 0
NumberofTotalExecutions(∗107) NumberofTotalExecutions(∗107)
AFL2.52b AFLFast ecofuzz
objdump
26000
24000
22000
20000
18000
0 4 8 12 16 20 24 0 4 8 12 16 20 24
Time[hours] Time[hours]
egarevoChcnarB egarevoChcnarB
0 1 2 3 4 5 6 7 8 0 1 2 3 4 5 6 7 8
10000 28000
9000
8000
7000
6000
5000
4000
Figure 5. The upper two graphs published in the EcoFuzz paper [175]
show a strong advantage over all competitors on the non-standard metric
numberoftotalspathsoverthenumberoftotalexecutions.Thetwoplots
at the bottom compare EcoFuzz on the standard metric branch coverage
overtime.Onthestandardmetric,EcoFuzzperformssignificantlyworse.
coverage. The full results and the generated FuzzBench
reports can be found in our reproduction repository.
Corresponding with the authors, they state they have
been following fuzzing evaluations at the time that focused
on path coverage, and they have confirmed that EcoFuzz
may cover fewer branches on some binaries, stating that its
goal is to optimize for paths over executions rather than
branches over time.
Lessons learned: A fuzzer may excel at one metric
but not on another; hence, selecting a suitable set
of evaluation metrics is crucial to provide a reader
with the full picture. Evaluating on established metrics
is required, as new metrics may imply a completely
different picture.
4.6. Case Study: Unclear Documentation
Another paper published at USENIX’23, Polyfuzz [96],
targets programs containing code in different languages,
such as interpreter languages calling into native bindings.
Artifact status. PolyFuzz has been awarded the
available badge. Our reproduction artifact is available at
https://github.com/fuzz-evaluator/PolyFuzz-eval.
Observations. While studying the artifact, we no-
ticedirregularitiesregardingtheseedsetsusedbyPolyFuzz
comparedtotheotherfuzzers.Anexampleofsuchacaseis
the image_load harness for the Python image processing

libraryPillow.Inthisparticularcase,thefuzzerAtherisgets configurations. Unfortunately, we could only partially re-
39 seed files, while PolyFuzz’s seed directory has 58 files. produce the claims as presented in the Firm-AFL paper and
Experiment: Fair seed allocation. We intended to observed one case where the baseline performed better than
run both fuzzers with their respective seed sets to measure Firm-AFL.Thefullresultsofourexperimentscanbefound
the impact of these different seed sets on the coverage. Un- in our reproduction repository.
fortunately,theauthors’extensionofAtheris(calledAtheris-
CextinthePolyFuzzpaper),whichwouldallowtocompute Lessons learned: While it is unreasonable to expect
combined coverage for both Python and the native code, each academic artifact to be of production quality, we
was not released alongside their artifact. Hence, as proxy recommendtostriveforareasonablelevelofreadabil-
measurement, we compute the initial coverage achieved by ityanddocumentationthatallowsotherstounderstand
PolyFuzzonbothseedsets.FortheseedsetgiventoAtheris, and use the code, thus promoting reproducibility.
PolyFuzz covers 218 edges, while for its own seed set, it
covers814edges.Evidently,oneseedsetprovidesmorethan
three times as much coverage as the other, giving PolyFuzz 4.8. Case Study: Unfair Coverage Measurements
a headstart during the evaluation.
When contacted, the authors clarified that they did not
keeptheseedsetsfromtheirevaluation,buttheyassuredus The final case study analyzes FishFuzz [186], published
thattheyusedtheseedsfromthecorrespondingbenchmarks at USENIX’23. The paper proposes an input prioritization
for all fuzzers. strategy based on a multi-distance metric that allows for
optimizing the fuzzing efforts towards thousands of targets
Lessons learned: Seeds have an impact on fuzzer
(e.g., sanitizer labels) in the sense of direct fuzzing.
performance. We recommend to give all fuzzers the
Artifact status. FishFuzz has received the available
same set of seeds and to publish the seeds used.
andfunctionalbadges.Ouradditionalexperimentsareavail-
able at https://github.com/fuzz-evaluator/FishFuzz-eval.
4.7. Case Study: Incomplete Artifact Observations. When studying the artifact in de-
tail, we notice that FishFuzz’s way of measuring coverage
Firm-AFL [187], published at USENIX Security’19,
may erroneously give FishFuzz an unfair edge. From all
aims to fuzz Linux-based IoT firmware via augmented pro-
evaluated fuzzers, FishFuzz was the only fuzzer to place
cess emulation. To do so, the core fuzzing loop targets a
coverage instrumentation not only within the actual target
single binary under user-mode emulation, while selectively
but also in the added ASAN instrumentation. Consequently,
forwarding system calls to a full-system emulator.
FishFuzz also stored inputs that exercised new coverage in
Artifact status. Artifact evaluation was not avail-
the instrumentation; other fuzzers discarded these inputs,
able to Firm-AFL, but different versions of its source
as no new coverage was observed. This became a problem
code are publicly available across multiple repositories.
when the binary instrumented by FishFuzz was used for
Our reproduction artifact is available at https://github.com/
coverage measurements for all fuzzers during evaluation
fuzz-evaluator/firmafl-eval/.
since—by design—only FishFuzz would keep inputs exer-
Observations. During our analysis of the artifact,
cising coverage in the ASAN instrumentation.
we noticed that the repository lacks documentation. Crucial
Experiment: Fair coverage measurement. To
stepsaremissing,likecorrectbuildinstructionsfordifferent
demonstrate the impact of measuring coverage in instru-
configurations, making it hard for researchers to reuse the
mentation code, we measure the coverage for a binary both
artifact and set up the fuzzer and its environment correctly.
with and without FishFuzz instrumentation. The result is
Furthermore, when setting up the experiments, we noticed
depictedinFigure6.IftheFishFuzzcoveragebinaryisused
thatsomeoftheexperimentconfigurationfilesweremissing
for coverage computation, FishFuzz covers 8.44% more
and target harnessing is heavily inlined with core emulation
edges on average over all runs. When using a binary with
logic. Not only do these issues hinder extensibility, but
standard AFL instrumentation (i.e., where coverage is not
they also prevented us from getting all targets working to
measured in the additional instrumentation), the observed
reproduce the Firm-AFL experiments. The fuzzer binaries
coverage increase is reduced to 1.69%. Furthermore, the
are shipped in a pre-compiled binary version and fail to
total number of edges is considerably smaller, showing that
buildfromtheprovidedsourcecode.Moreover,theprovided
edge counts between different binaries do not translate.
baseline uses an older version of AFL (2.06b), while the
Note that both coverage binaries rely on colliding bitmaps
augmented mode uses AFL v2.52b.
since the artifact tooling of FishFuzz expects standard AFL
Experiment: Crash Triggers. Being the only ex-
bitmaps to be used. We recommend to not use colliding
periment with enough documentation to reproduce, we aim
bitmaps for coverage measurements.
to measure the number of crashes produced by both the
augmentedandfull-systememulatorversions.Wewereable
Lessons learned: Unintended side effects may skew
to run fuzzing campaigns for 9 out of 11 targets, where
coverage measurements; we recommend using stan-
one of them only ran for the baseline and not Firm-AFL.
dardized methods of measuring coverage.
The remaining two targets lack the required target-specific

cxxfilt desirable to include targets that have been tested in other
8,000 works. Actions such as patches applied to targets should
|     |       |     |     |     |     |     | be explained.                                       |            | If a fuzzer |           | has certain | restrictions |        | (such as |
| --- | ----- | --- | --- | --- | --- | --- | --------------------------------------------------- | ---------- | ----------- | --------- | ----------- | ------------ | ------ | -------- |
|     | 7,000 |     |     |     |     |     | symbolicexecution-basedtechniquesnotbeingableofmod- |            |             |           |             |              |        |          |
|     |       |     |     |     |     |     | eling all                                           | syscalls), | we          | recommend |             | outlining    | those. | We also  |
|     | 6,000 |     |     |     |     |     | highlyrecommendusingwell-establishedbenchmarks,such |            |             |           |             |              |        |          |
segdE derevoC# as FuzzBench, to facilitate easy reproducibility.
5,000
|     |     |     |     |     |     |     | 5.3. Comparison |     |     | to Other | Fuzzers |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --- | -------- | ------- | --- | --- | --- |
4,000
3,000 It is crucial to compare against the state of the art in
Fuzzer
|     |     |     |     |     |     |     | the respective |     | field and | the | baseline | (if any) | on  | which the |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --------- | --- | -------- | -------- | --- | --------- |
AFL++
|     |       |     |     |     |            |     | new technique |     | is implemented. |     |     | This also | includes | well- |
| --- | ----- | --- | --- | --- | ---------- | --- | ------------- | --- | --------------- | --- | --- | --------- | -------- | ----- |
|     | 2,000 |     |     |     | FishFuzz++ |     |               |     |                 |     |     |           |          |       |
establishedandactivelymaintainedfuzzers,suchasAFL++.
Coverage Binary
|     | 1,000 |     |     |     | AFL      |     | IncludingthenewfuzzerinbenchmarkssuchasFuzzBench |           |           |         |        |           |             |          |
| --- | ----- | --- | --- | --- | -------- | --- | ------------------------------------------------ | --------- | --------- | ------- | ------ | --------- | ----------- | -------- |
|     |       |     |     |     | FishFuzz |     | allows for                                       | comparing |           | against | a wide | range     | of fuzzers. | If       |
|     |       | 0   |     |     |          |     | presenting                                       | a new     | technique |         | with   | separable | design      | choices, |
00:00 06:00 12:00 18:00 review them individually via ablation studies, for example,
Time [hh:mm]
|     |     |     |     |     |     |     | by designing | baselines |     | that | successively |     | enable | or disable |
| --- | --- | --- | --- | --- | --- | --- | ------------ | --------- | --- | ---- | ------------ | --- | ------ | ---------- |
Figure6.Mediancoverageovertimeforcxxfilt:Inonecase,wemea- individual components.
surecoverageviaastandardAFLbinaryand,intheotherweuseFishFuzz’s
binarythatcontainsadditionalcoverageinstrumentation.Foreachfuzzer,
|     |     |     |     |     |     |     | 5.4. Evaluation |     | Setup |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | ----- | --- | --- | --- | --- | --- |
thetargetwasrun10timesfor24heach.Thedisplayedintervalsencloseall
tenrunsoftherespectivefuzzer.Ifthecoverageismeasuredonthebiased
binary with FishFuzz instrumentation ( ), FishFuzz++ finds on average Thechosenevaluationsetupshouldbewelldocumented.
| 8.44%  | more | edges than AFL++.   | Measuring       | coverage | on         | a standard AFL |               |         |        |              |     |                |               |         |
| ------ | ---- | ------------------- | --------------- | -------- | ---------- | -------------- | ------------- | ------- | ------ | ------------ | --- | -------------- | ------------- | ------- |
|        |      |                     |                 |          |            |                | This entails  | details |        | regarding    | the | used hardware, |               | experi- |
| binary | ( )  | (without additional | instrumentation |          | introduced | by FishFuzz),  |               |         |        |              |     |                |               |         |
|        |      |                     |                 |          |            |                | ment runtime, |         | number | of allocated |     | cores,         | and processes | per     |
thecoveragedeltaisonly1.69%.
|     |         |                |     |     |            |     | fuzzer. The | conducted    |            | experiments  |     | and how  | to  | reproduce |
| --- | ------- | -------------- | --- | --- | ---------- | --- | ----------- | ------------ | ---------- | ------------ | --- | -------- | --- | --------- |
| 5.  | Revised | Best Practices |     | for | Evaluation |     |             |              |            |              |     |          |     |           |
|     |         |                |     |     |            |     | them should | be           | explained. |              |     |          |     |           |
|     |         |                |     |     |            |     | For         | the runtime, |            | we recommend |     | choosing | at  | least 24  |
Based on our literature analysis and the case studies, hours. Longer runtimes may be appropriate if the evalu-
we now provide recommendations on ensuring a fair and ated fuzzers do not flatline at the end of the experiment.
| reproducible |     | fuzzing evaluation. |     | A comprehensive |     | check- |           |     |        |          |     |          |      |         |
| ------------ | --- | ------------------- | --- | --------------- | --- | ------ | --------- | --- | ------ | -------- | --- | -------- | ---- | ------- |
|              |     |                     |     |                 |     |        | Regarding | CPU | cores, | choosing |     | a single | core | may not |
list that summarizes these recommendations is available in be representative of modern systems. Special care must
our GitHub repository at https://github.com/fuzz-evaluator/ be taken to avoid congestion in the kernel when running
guidelines. Overall, we recommend that authors thoroughly multiple fuzzers in parallel on one system; even if using
| review | the | threats to validity | for | their | respective | works to |         |            |     |        |     |              |     |           |
| ------ | --- | ------------------- | --- | ----- | ---------- | -------- | ------- | ---------- | --- | ------ | --- | ------------ | --- | --------- |
|        |     |                     |     |       |            |          | Docker, | the kernel | may | become |     | a bottleneck | in  | resolving |
reflect potential issues that could invalidate their evaluation. certainsyscalls,unfairlyslowingdownonefuzzingprocess.
|     |     |     |     |     |     |     | Individual | fuzzer | instances |     | can be | encapsulated | in  | separate |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ------ | --------- | --- | ------ | ------------ | --- | -------- |
5.1. Reproducible Artifact virtual machine instances to avoid such situations.
|     |     |     |     |     |     |     | Regarding |     | seeds, | we  | recommend | running | with | unin- |
| --- | --- | --- | --- | --- | --- | --- | --------- | --- | ------ | --- | --------- | ------- | ---- | ----- |
Forreproducibility,itiscrucialtoopen-sourcethesource formedseedsormultipleseedsets.Seedsmustbedescribed
code including documentation. We highly recommend par- and accessible (in the case of informed seeds) to allow
ticipating in an artifact evaluation if available. Furthermore, for reproducibility. All fuzzers should have fair access to
| it  | is essential | to (i) specify | the | exact | versions | of targets |            |          |          |     |        |              |     |          |
| --- | ------------ | -------------- | --- | ----- | -------- | ---------- | ---------- | -------- | -------- | --- | ------ | ------------ | --- | -------- |
|     |              |                |     |       |          |            | all seeds. | If using | informed |     | seeds, | we recommend |     | plotting |
(and harnesses) and fuzzers used for comparison, (ii) use or analyzing the coverage achieved by the initial seed set.
runtime environment abstractions, such as Docker (where This avoids attributing a high coverage achieved to fuzzer
feasible),(iii)namethebaselineonwhichthenewtechnique performance instead of the initial seeds.
| is  | implemented | upon (if | any) | as well | as its | version, and |     |     |     |     |     |     |     |     |
| --- | ----------- | -------- | ---- | ------- | ------ | ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
avoid squashing commits of this baseline. In the long term, 5.5. Evaluation Metrics
| a mandatory |     | artifact evaluation |     | as part | of the | submission |     |     |     |     |     |     |     |     |
| ----------- | --- | ------------------- | --- | ------- | ------ | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
process could improve the quality and reproducibility of A fuzzer comparison should use standardized, well-
research artifacts. established metrics (at least as a complementary metric if a
|     |     |     |     |     |     |     | technique | requires | the | introduction |     | of a | new metric); | this |
| --- | --- | --- | --- | --- | --- | --- | --------- | -------- | --- | ------------ | --- | ---- | ------------ | ---- |
5.2. Targets under Test includes both coverage and found bugs. Optimally, both
|     |     |     |     |     |     |     | code coverage |     | and bug-finding |     | capability |     | are evaluated, | as  |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --------------- | --- | ---------- | --- | -------------- | --- |
Selected evaluation targets should form a representative both suffer from individual drawbacks [23], [88], [179]. We
set that shows strengths of the proposed approach and recommendusingmodernbenchmarksthataidinsettingup
allows for comparability with previous work. It is therefore the experiment and ensure a fair, bias-free execution.

It is necessary to specify details such as how coverage Asolutiontothisproblemisthebootstrapversionofthe
is collected, for example, whether it is measured on a non- ANOVA method. If the ANOVA rejects the null hypothesis,
instrumented binary, translated blocks from an emulator, or it shows at level α that there is at least one pair of fuzzing
using established means such as lcov. Ideally, coverage methods that perform significantly different for the target
is not measured using bitmaps with collisions, but using considered. In a second step, a so-called Posthoc-test is
a collision-free encoding or other means. Additionally, the performed to determine which pairwise comparisons are
evaluation must ensure that the same notion of coverage is significant, given that the ANOVA has already shown that
used for each of the compared fuzzers. therearesignificantdifferencesatall.PossiblePosthoc-tests
When searching for bugs in new targets to show real- are, for example, the Tukey-Kramer method if all pairwise
world impact, it is crucial to select reasonable targets, comparisons among all samples are of interest or the Dun-
i.e., projects that are not insecure by design, have been nett method if only the comparisons to a reference method,
inactive for years, or are unsuitable for other reasons. We such as the newly developed fuzzer, are of interest [138].
also recommend running other state-of-the-art fuzzers to For a bootstrap version of these algorithms, we propose as
see whether they find the bugs as well, thereby addressing a simple solution two-sample t-tests with critical values for
concerns regarding fuzzing previously untested software. rejection based on a bootstrap resampling with replacement
Crashes identified by the fuzzer should be deduplicated oftheteststatistics.Here,foreachsimulation,themaximum
before opening a report, and the triaging process should value of the test statistics is used for all pairwise compar-
be clearly described. When testing crashes, we recommend isons of interest. We provide more details, algorithms, and
reproducing them on a binary without fuzzer or coverage scripts implementing examples for these tests in our artifact
instrumentation to avoid reproducibility issues. at https://github.com/fuzz-evaluator/statistics. Additionally,
Ideally, only maintainers should request CVEs. If they evaluations should measure effect size, e.g., using Vargha
do not request one, researchers can still link to the bug and Delaney’s Aˆ test [156], and quantify uncertainty, for
12
report instead. Requesting multiple CVEs for a single bug example, by using intervals in plots.
or requesting CVEs without coordinating or informing the
maintainers must be avoided. If possible, reporting bugs or 6. Conclusion
CVEsanonymouslyallowsforprovidingthereviewerswith
access during submission, such that they can inspect the
Reproducibilityisacornerstoneofscienceandthebasis
CVEs or bug reports and assess their validity (as opposed
for research. In this work, we have systematically studied
to the current practice of blinding CVEs and bug reports
how 150 fuzzing papers published in the past six years at
during submission, preventing any analysis by reviewers).
leading conferences design their evaluation. Furthermore,
That said, we do not believe that having CVEs should be
we have performed an in-depth analysis of the artifacts
required to show the practical impact of a fuzzer.
of eight papers and attempted to reproduce their results.
Based on the insights gained, we outlined several potential
5.6. Statistical Evaluation pitfalls and shortcomings threatening the validity of fuzzing
evaluations. Ultimately, we provided revised recommenda-
Any evaluation should be backed by statistical tests.
tions and best practices to improve future evaluation of
To enable these tests, we recommend running at least ten
fuzzing research. We published a concise set of guidelines
trials. Alternatively, the number of trials can be calculated
athttps://github.com/fuzz-evaluator/guidelinesandwelcome
via an a-priori power analysis to ensure a sufficient sample
community contributions.
size leading to statistically significant results [44]. This is
particularlyimportantifthefuzzerunderconsiderationonly
Acknowledgment
slightly outperforms the state of the art, where n ≫ 10
may be required. To avoid the problems mentioned in
We thank our anonymous shepherd and reviewers for
Section 3.2.6, we recommend an alternative to the widely
their valuable feedback. Further, we thank Dominik Maier,
usedMann-Whitney-Utest;permutationtestsorresampling
Johannes Willbold, Daniel Klischies, Merlin Chlosta, and
testssuchasbootstrapmethods.Thesemethodsavoidstrong
Marcel Böhme (in no particular order) for their helpful
assumptions regarding a normal distribution.
comments on a draft of this work. We also thank the
If more than two fuzzers have been compared for a
countlessresearcherswithwhomwehavediscussedfuzzing
target, the (bootstrap-based) two-sample t-test is not a good
research and how to evaluate it, ultimately paving the
choice, since we would have to perform more than one
way for this work. This work was funded by the Eu-
pairwise comparison to test the null hypotheses of no dif-
ropean Research Council (ERC) under the consolidator
ference between any of the expected means for the fuzzing
grant RS3 (101045669) and the German Federal Ministry
methods.Thisresultsinthemultipletestingproblem,which
of Education and Research under the grants KMU-Fuzz
is the observation that the probability of at least one false
(16KIS1898) and CPSec (16KIS1899). Additionally, this
positive result in the set of comparisons performed for a
researchwaspartiallysupportedbytheUKEngineeringand
targetexceedsthesingletestlevelαsubstantially.Thesame
Physical Sciences Research Council (EPSRC) under grant
argument holds for other strategies based on two-sample
EP/V000454/1. The results feed into DsbDtech.
comparisons such as the Mann-Whitney-U test [3].

References
|     |     |     |     |     |     |     | [21] M. | Böhme, D. | Liyanage, | and V. Wüstholz, |     | “Estimating | Residual |
| --- | --- | --- | --- | --- | --- | --- | ------- | --------- | --------- | ---------------- | --- | ----------- | -------- |
RiskinGreyboxFuzzing,”inACMJointEuropeanSoftwareEngi-
neeringConferenceandSymposiumontheFoundationsofSoftware
[1] M.AbadiandR.Needham,“PrudentEngineeringPracticeforCryp- Engineering(ESEC/FSE),2021.
tographic Protocols,” IEEE Transactions on Software Engineering, [22] M. Böhme, V. J. M. Manès, and S. K. Cha, “Boosting Fuzzer
vol.22,no.1,pp.6–15,1996. Efficiency: An Information Theoretic Perspective,” in ACM Joint
[2] I. Angelakopoulos, G. Stringhini, and M. Egele, “FirmSolo: En- EuropeanSoftwareEngineeringConferenceandSymposiumonthe
ablingDynamic Analysis of BinaryLinux-based IoTKernelMod- FoundationsofSoftwareEngineering(ESEC/FSE),2020.
ules,”inUSENIXSecuritySymposium,2023. [23] M. Böhme, L. Szekeres, and J. Metzman, “On the Reliability of
[3] A. Arcuri and L. Briand, “A Practical Guide for Using Statistical Coverage-BasedFuzzerBenchmarking,”inIEEE/ACMInternational
TeststoAssessRandomizedAlgorithmsinSoftwareEngineering,” ConferenceonAutomatedSoftwareEngineering(ASE),2022.
inInternationalConferenceonSoftwareEngineering(ICSE),2011. [24] H. Chen, S. Guo, Y. Xue, Y. Sui, C. Zhang, Y. Li, H. Wang, and
[4] D. Arp, E. Quiring, F. Pendlebury, A. Warnecke, F. Pierazzi, Y.Liu,“MUZZ:Thread-awareGrey-boxFuzzingforEffectiveBug
C. Wressnegger, L. Cavallaro, and K. Rieck, “Dos and don’ts Hunting in Multithreaded Programs,” in USENIX Security Sympo-
sium,2020.
| of machine |     | learning | in computer | security,” | in  | USENIX Security |     |     |     |     |     |     |     |
| ---------- | --- | -------- | ----------- | ---------- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- |
Symposium,2022.
|     |     |     |     |     |     |     | [25] H. | Chen, Y. | Xue, Y. | Li, B. Chen, | X. Xie, | X. Wu, | and Y. Liu, |
| --- | --- | --- | --- | --- | --- | --- | ------- | -------- | ------- | ------------ | ------- | ------ | ----------- |
[5] C. Aschermann, T. Frassetto, T. Holz, P. Jauernig, A.-R. Sadeghi, “Hawkeye:TowardsaDesiredDirectedGrey-boxFuzzer,”inACM
|     |     |     |     |     |     |     | Conference | on  | Computer | and Communications |     | Security | (CCS), |
| --- | --- | --- | --- | --- | --- | --- | ---------- | --- | -------- | ------------------ | --- | -------- | ------ |
andD.Teuchert,“NAUTILUS:FishingforDeepBugswithGram-
| mars,” | in Symposium |     | on Network | and Distributed |     | System Security | 2018. |     |     |     |     |     |     |
| ------ | ------------ | --- | ---------- | --------------- | --- | --------------- | ----- | --- | --- | --- | --- | --- | --- |
(NDSS),2019.
|     |     |     |     |     |     |     | [26] J. Chen, | W. Diao, | Q.  | Zhao, C. Zuo, | Z. Lin, | X. Wang, | W. C. Lau, |
| --- | --- | --- | --- | --- | --- | --- | ------------- | -------- | --- | ------------- | ------- | -------- | ---------- |
M.Sun,R.Yang,andK.Zhang,“IoTFuzzer:DiscoveringMemory
| [6] C. Aschermann, |     | S. Schumilo, |     | A. Abbasi, | and T. | Holz, “Ijon: Ex- |     |     |     |     |     |     |     |
| ------------------ | --- | ------------ | --- | ---------- | ------ | ---------------- | --- | --- | --- | --- | --- | --- | --- |
ploring Deep State Spaces via Fuzzing,” in IEEE Symposium on CorruptionsinIoTThroughApp-basedFuzzing,”inSymposiumon
SecurityandPrivacy(S&P),2020. NetworkandDistributedSystemSecurity(NDSS),2018.
|     |     |     |     |     |     |     | [27] J. Chen, | W. Han, | M.  | Yin, H. Zeng, | C. Song, | B. Lee, | H. Yin, and |
| --- | --- | --- | --- | --- | --- | --- | ------------- | ------- | --- | ------------- | -------- | ------- | ----------- |
[7] C.Aschermann,S.Schumilo,T.Blazytko,R.Gawlik,andT.Holz,
“REDQUEEN: Fuzzing with Input-to-State Correspondence,” in I. Shin, “SYMSAN: Time and Space Efficient Concolic Execution
Symposium on Network and Distributed System Security (NDSS), viaDynamicData-flowAnalysis,”inUSENIXSecuritySymposium,
| 2019. |     |     |     |     |     |     | 2022. |     |     |     |     |     |     |
| ----- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- |
[8] Association for Computing Machinery, “Artifact Review and [28] J. Chen, J. Wang, C. Song, and H. Yin, “JIGSAW: Efficient and
Badging Version 1.1,” 2020. [Online]. Available: https://www.acm. ScalablePathConstraintsFuzzing,”inIEEESymposiumonSecurity
org/publications/policies/artifact-review-and-badging-current andPrivacy(S&P),2022.
[9] J.Ba,M.Böhme,Z.Mirzamomen,andA.Roychoudhury,“Stateful [29] L.Chen,Q.Cai,Z.Ma,Y.Wang,H.Hu,M.Shen,Y.Liu,S.Guo,
GreyboxFuzzing,”inUSENIXSecuritySymposium,2022. H. Duan, K. Jiang, and Z. Xue, “SFuzz: Slice-based Fuzzing for
[10] N. Bars, M. Schloegel, T. Scharnowski, N. Schiller, and T. Holz, Real-Time Operating Systems,” in ACM Conference on Computer
andCommunicationsSecurity(CCS),2022.
“Fuzztruction:UsingFaultInjection-basedFuzzingtoLeverageIm-
plicitDomainKnowledge,”inUSENIXSecuritySymposium,2023. [30] P. Chen and H. Chen, “Angora: Efficient Fuzzing by Principled
[11] F. Bellard, “QEMU, a Fast and Portable Dynamic Translator,” in Search,”inIEEESymposiumonSecurityandPrivacy(S&P),2018.
USENIXAnnualTechnicalConference(ATC),2005. [31] P.Chen,J.Liu,andH.Chen,“Matryoshka:FuzzingDeeplyNested
[12] L. Bernhard, T. Scharnowski, M. Schloegel, T. Blazytko, and Branches,”inACMConferenceonComputerandCommunications
| T.Holz,“JIT-Picking:DifferentialFuzzingofJavaScriptEngines,” |     |     |     |     |     |     | Security(CCS),2019. |     |     |     |     |     |     |
| ------------------------------------------------------------ | --- | --- | --- | --- | --- | --- | ------------------- | --- | --- | --- | --- | --- | --- |
in ACM Conference on Computer and Communications Security [32] P.Chen,Y.Xie,Y.Lyu,Y.Wang,andH.Chen,“HOPPER:Inter-
(CCS),2022. pretative Fuzzing for Libraries,” in ACM Conference on Computer
[13] T.Blazytko,C.Aschermann,M.Schloegel,A.Abbasi,S.Schumilo, andCommunicationsSecurity(CCS),2023.
S.Wörner,andT.Holz,“GRIMOIRE:SynthesizingStructurewhile [33] W. Chen, Y. Wang, Z. Zhang, and Z. Qian, “SyzGen: Auto-
Fuzzing,”inUSENIXSecuritySymposium,2019. matedGenerationofSyscallSpecificationofClosed-SourcemacOS
|                                                            |     |     |     |     |     |     |           | ACM | Conference | on Computer |     | and Communications |     |
| ---------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --------- | --- | ---------- | ----------- | --- | ------------------ | --- |
| [14] M.Böhme,C.Cadar,andA.Roychoudhury,“Fuzzing:Challenges |     |     |     |     |     |     | Drivers,” | in  |            |             |     |                    |     |
Security(CCS),2021.
andReflections,”IEEESoftw.,vol.38,no.3,pp.79–86,2021.
[15] M. Böhme, V.-T. Pham, and A. Roychoudhury, “Coverage-based [34] Y. Chen, P. Li, J. Xu, S. Guo, R. Zhou, Y. Zhang, T. Wei, and
|     |     |     |     |     |     |     | L.  | Lu, “SAVIOR: | Towards | Bug-Driven | Hybrid | Testing,” | in IEEE |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | ------- | ---------- | ------ | --------- | ------- |
GreyboxFuzzingasMarkovChain,”IEEETransactionsonSoftware
SymposiumonSecurityandPrivacy(S&P),2020.
Engineering,vol.45,no.5,pp.489–506,2017.
[16] L.Borzacchiello,E.Coppa,andC.Demetrescu,“FuzzingSymbolic [35] Y. Chen, Y. Jiang, F. Ma, J. Liang, M. Wang, C. Zhou, X. Jiao,
andZ.Su,“EnFuzz:EnsembleFuzzingwithSeedSynchronization
Expressions,”inIEEE/ACMInternationalConferenceonAutomated
amongDiverseFuzzers,”inUSENIXSecuritySymposium,2019.
SoftwareEngineering(ASE),2021.
[17] A.Bulekov,B.Das,S.Hajnoczi,andM.Egele,“NoGrammar,No [36] Y.Chen,T.Su,andZ.Su,“DeepDifferentialTestingofJVMImple-
mentations,”inIEEE/ACMInternationalConferenceonAutomated
| Problem: | Towards | Fuzzing | the | Linux Kernel | without | System-Call |     |     |     |     |     |     |     |
| -------- | ------- | ------- | --- | ------------ | ------- | ----------- | --- | --- | --- | --- | --- | --- | --- |
SoftwareEngineering(ASE),2019.
| Descriptions,” |     | in Symposium |     | on Network | and Distributed | System |     |     |     |     |     |     |     |
| -------------- | --- | ------------ | --- | ---------- | --------------- | ------ | --- | --- | --- | --- | --- | --- | --- |
Security(NDSS),2023. [37] Z.Chen,S.L.Thomas,andF.D.Garcia,“MetaEmu:AnArchitec-
tureAgnosticRehostingFrameworkforAutomotiveFirmware,”in
| [18] J. Bundt, | A.  | Fasano, | B. Dolan-Gavitt, | W.  | Robertson, | and T. Leek, |     |     |     |     |     |     |     |
| -------------- | --- | ------- | ---------------- | --- | ---------- | ------------ | --- | --- | --- | --- | --- | --- | --- |
ACMConferenceonComputerandCommunicationsSecurity(CCS),
| “Evaluating |     | Synthetic | Bugs,” | in ACM Symposium |     | on Information, |     |     |     |     |     |     |     |
| ----------- | --- | --------- | ------ | ---------------- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- |
2022.
ComputerandCommunicationsSecurity(ASIACCS),2021.
|         |        |             |     |             |        |                 | [38] M. | Cho, S. Kim, | and | T. Kwon, “Intriguer: |     | Field-Level | Constraint |
| ------- | ------ | ----------- | --- | ----------- | ------ | --------------- | ------- | ------------ | --- | -------------------- | --- | ----------- | ---------- |
| [19] M. | Busch, | A. Machiry, | C.  | Spensky, G. | Vigna, | C. Kruegel, and |         |              |     |                      |     |             |            |
SolvingforHybridFuzzing,”inACMConferenceonComputerand
M.Payer,“TEEzz:FuzzingTrustedApplicationsonCOTSAndroid
Devices,”inIEEESymposiumonSecurityandPrivacy(S&P),2023. CommunicationsSecurity(CCS),2019.
|               |       |             |                 |           |                 |                | [39] J. Choi, | J. Jang,  | C.     | Han, and S. | K. Cha,       | “Grey-box | Concolic   |
| ------------- | ----- | ----------- | --------------- | --------- | --------------- | -------------- | ------------- | --------- | ------ | ----------- | ------------- | --------- | ---------- |
| [20] M.       | Böhme | and B.      | Falk, “Fuzzing: | On        | the Exponential | Cost of        |               |           |        |             |               |           |            |
|               |       |             |                 |           |                 |                | Testing       | on Binary | Code,” | in IEEE/ACM | International |           | Conference |
| Vulnerability |       | Discovery,” | in              | ACM Joint | European        | Software Engi- |               |           |        |             |               |           |            |
neeringConferenceandSymposiumontheFoundationsofSoftware onAutomatedSoftwareEngineering(ASE),2019.
Engineering(ESEC/FSE),2020. [40] J.Choi,K.Kim,D.Lee,andS.K.Cha,“NtFuzz:EnablingType-

AwareKernelFuzzingonWindowswithStaticBinaryAnalysis,”in CPUs,” in ACM Conference on Computer and Communications
IEEESymposiumonSecurityandPrivacy(S&P),2021. Security(CCS),2021.
[41] N. Christou, D. Jin, V. Atlidakis, B. Ray, and V. P. Kemerlis, [62] Google, “OSS-Fuzz: Continuous Fuzzing for Open Source
“IvySyn: Automated Vulnerability Discovery in Deep Learning Software.”[Online].Available:https://github.com/google/oss-fuzz
Frameworks,”inUSENIXSecuritySymposium,2023.
[63] ——, “Fuzzer-Test-Suite,” 2016. [Online]. Available: https:
[42] A.A.Clements,E.Gustafson,T.Scharnowski,P.Grosen,D.Fritz, //github.com/google/fuzzer-test-suite
C. Kruegel, G. Vigna, S. Bagchi, and M. Payer, “HALucinator:
[64] H. Green and T. Avgerinos, “GraphFuzz: Library API Fuzzing
Firmware Re-hosting Through Abstraction Layer Emulation,” in
withLifetime-awareDataflowGraphs,”inIEEE/ACMInternational
USENIXSecuritySymposium,2020.
ConferenceonAutomatedSoftwareEngineering(ASE),2022.
[43] T. Cloosters, J. Willbold, T. Holz, and L. Davi, “SGXFuzz: Effi-
[65] S.Groß,S.Koch,L.Bernhard,T.Holz,andM.Johns,“FUZZILLI:
ciently Synthesizing Nested Structures for SGX Enclave Fuzzing,”
FuzzingforJavaScriptJITCompilerVulnerabilities,”inSymposium
inUSENIXSecuritySymposium,2022.
onNetworkandDistributedSystemSecurity(NDSS),2023.
[44] J. Cohen, Statistical Power Analysis for the Behavioral Sciences.
[66] T. Gu, X. Li, S. Lu, J. Tian, Y. Nie, X. Kuang, Z. Lin, C. Liu,
Academicpress,2013.
J.Liang,andY.Jiang,“Group-basedCorpusSchedulingforParallel
[45] DARPA, “DARPA Cyber Grand Challenge,” 2018. [Online]. Fuzzing,” in ACM Joint European Software Engineering Confer-
Available:https://github.com/CyberGrandChallenge ence and Symposium on the Foundations of Software Engineering
[46] N.Demir,M.Große-Kampmann,T.Urban,C.Wressnegger,T.Holz, (ESEC/FSE),2022.
andN.Pohlmann,“ReproducibilityandReplicabilityofWebMea-
[67] S. Guo, X. Wan, W. You, B. Liang, W. Shi, Y. Zhang, J. Huang,
surementStudies,”inACMWebConference2022,2022.
andJ.Zhang,“Operand-Variation-OrientedDifferentialAnalysisfor
[47] P. Deng, Z. Yang, L. Zhang, G. Yang, W. Hong, Y. Zhang, and FuzzingBindingCallsinPDFReaders,”inIEEE/ACMInternational
M.Yang,“NestFuzz:EnhancingFuzzingwithComprehensiveUn- ConferenceonAutomatedSoftwareEngineering(ASE),2023.
derstanding of Input Processing Logic,” in ACM Conference on
[68] E.Güler,C.Aschermann,A.Abbasi,andT.Holz,“AntiFuzz:Im-
ComputerandCommunicationsSecurity(CCS),2023.
pedingFuzzingAuditsofBinaryExecutables,”inUSENIXSecurity
[48] S.Dinesh,N.Burow,D.Xu,andM.Payer,“RetroWrite:Statically Symposium,2019.
InstrumentingCOTSBinariesforFuzzingandSanitization,”inIEEE
[69] H.Han,D.Oh,andS.K.Cha,“CodeAlchemist:Semantics-Aware
SymposiumonSecurityandPrivacy(S&P),2020.
Code Generation to Find Vulnerabilities in JavaScript Engines,” in
[49] S. T. Dinh, H. Cho, K. Martin, A. Oest, K. Zeng, A. Kapravelos, Symposium on Network and Distributed System Security (NDSS),
G.-J. Ahn, T. Bao, R. Wang, A. Doupé, and Y. Shoshitaishvili, 2019.
“Favocado:FuzzingtheBindingCodeofJavaScriptEnginesUsing
[70] A.Hazimeh,A.Herrera,andM.Payer,“Magma:AGround-Truth
Semantically Correct Test Cases,” in Symposium on Network and
FuzzingBenchmark,”ACMonMeasurementandAnalysisofCom-
DistributedSystemSecurity(NDSS),2021.
putingSystems(POMACS),vol.4,no.3,pp.49:1–49:29,2020.
[50] Dmitry Vyukov and Google, “Syzkaller – Kernel Fuzzer,” 2015.
[71] X.He,X.Xie,Y.Li,J.Sun,F.Li,W.Zou,Y.Liu,L.Yu,J.Zhou,
[Online].Available:https://github.com/google/syzkaller
W. Shi, and W. Huo, “SoFi Artifact,” 2021. [Online]. Available:
[51] B. Dolan-Gavitt, P. Hulin, E. Kirda, T. Leek, A. Mambretti,
https://sites.google.com/view/sofi4js/souce-and-data
W. Robertson, F. Ulrich, and R. Whelan, “Lava: Large-scale Au-
tomated Vulnerability Addition,” in IEEE Symposium on Security [72] ——, “SoFi: Reflection-Augmented Fuzzing for JavaScript En-
andPrivacy(S&P),2016. gines,” in ACM Conference on Computer and Communications
Security(CCS),2021.
[52] Z.Du,Y.Li,Y.Liu,andB.Mao,“Windranger:ADirectedGreybox
Fuzzer driven by Deviation Basic Blocks,” in IEEE/ACM Interna- [73] A.Herrera,H.Gunadi,S.Magrath,M.Norrish,M.Payer,andA.L.
tionalConferenceonAutomatedSoftwareEngineering(ASE),2022. Hosking,“SeedSelectionforSuccessfulFuzzing,”inInternational
SymposiumonSoftwareTestingandAnalysis(ISSTA),2021.
[53] B. Feng, A. Mera, and L. Lu, “P2IM: Scalable and Hardware-
independent Firmware Testing via Automatic Peripheral Interface [74] H.Huang,Y.Guo,Q.Shi,P.Yao,R.Wu,andC.Zhang,“BEACON:
Modeling,”inUSENIXSecuritySymposium,2020. Directed Grey-Box Fuzzing with Provable Path Pruning,” in IEEE
SymposiumonSecurityandPrivacy(S&P),2022.
[54] X. Feng, R. Sun, X. Zhu, M. Xue, S. Wen, D. Liu, S. Nepal,
and Y. Xiang, “Snipuzz: Black-box Fuzzing of IoT Firmware via [75] H.Huang,P.Yao,R.Wu,Q.Shi,andC.Zhang,“Pangolin:Incre-
MessageSnippetInference,”inACMConferenceonComputerand mentalHybridFuzzingwithPolyhedralPathAbstraction,”inIEEE
CommunicationsSecurity(CCS),2021. SymposiumonSecurityandPrivacy(S&P),2020.
[55] A.Fioraldi,D.C.D’Elia,andD.Balzarotti,“TheUseofLikelyIn- [76] A. Humayun, Y. Wu, M. Kim, and M. A. Gulzar, “NaturalFuzz:
variantsasFeedbackforFuzzers,”inUSENIXSecuritySymposium, NaturalInputGenerationforBigDataAnalytics,”inIEEE/ACMIn-
2021. ternationalConferenceonAutomatedSoftwareEngineering(ASE),
[56] A.Fioraldi,D.Maier,H.Eißfeldt,andM.Heuse,“AFL++:Combin- 2023.
ingIncrementalStepsofFuzzingResearch,”inUSENIXWorkshop [77] K. K. Ispoglou, D. Austin, V. Mohan, and M. Payer, “FuzzGen:
onOffensiveTechnologies(WOOT),2020. Automatic Fuzzer Generation,” in USENIX Security Symposium,
[57] A.Fioraldi,D.C.Maier,D.Zhang,andD.Balzarotti,“LibAFL:A 2020.
FrameworktoBuildModularandReusableFuzzers,”inACMCon- [78] P. Jauernig, D. Jakobovic, S. Picek, E. Stapf, and A.-R. Sadeghi,
ferenceonComputerandCommunicationsSecurity(CCS),2022. “DARWIN:SurvivaloftheFittestFuzzingMutators,”inSymposium
[58] J.Fu,J.Liang,Z.Wu,M.Wang,andY.Jiang,“Griffin:Grammar- onNetworkandDistributedSystemSecurity(NDSS),2023.
Free DBMS Fuzzing,” in IEEE/ACM International Conference on [79] D.R.Jeong,K.Kim,B.Shivakumar,B.Lee,andI.Shin,“Razzer:
AutomatedSoftwareEngineering(ASE),2022. Finding Kernel Race Bugs through Fuzzing,” in IEEE Symposium
[59] S.Gan,C.Zhang,P.Chen,B.Zhao,X.Qin,D.Wu,andZ.Chen, onSecurityandPrivacy(S&P),2019.
“GREYONE: Data Flow Sensitive Fuzzing,” in USENIX Security [80] H. Jia, M. Wen, Z. Xie, X. Guo, R. Wu, M. Sun, K. Chen, and
Symposium,2020. H. Jin, “Detecting JVM JIT Compiler Bugs via Exploring Two-
[60] S. Gan, C. Zhang, X. Qin, X. Tu, K. Li, Z. Pei, and Z. Chen, DimensionalInputSpaces,”inIEEE/ACMInternationalConference
“CollAFL:PathSensitiveFuzzing,”inIEEESymposiumonSecurity onAutomatedSoftwareEngineering(ASE),2023.
andPrivacy(S&P),2018. [81] J. Jiang, H. Xu, and Y. Zhou, “RULF: Rust Library Fuzzing via
[61] X. Ge, B. Niu, R. Brotzman, Y. Chen, H. Han, P. Godefroid, API Dependency Graph Traversal,” in IEEE/ACM International
andW.Cui,“HyperFuzzer:AnEfficientHybridFuzzerforVirtual ConferenceonAutomatedSoftwareEngineering(ASE),2021.

[82] L.Jiang,H.Yuan,M.Wu,L.Zhang,andY.Zhang,“Evaluatingand [102] Z.Lin,Y.Chen,Y.Wu,D.Mu,C.Yu,X.Xing,andK.Li,“GREBE:
ImprovingHybridFuzzing,”inIEEE/ACMInternationalConference Unveiling Exploitation Potential for Linux Kernel Bugs,” in IEEE
onAutomatedSoftwareEngineering(ASE),2023. SymposiumonSecurityandPrivacy(S&P),2022.
[103] S.Lipp,D.Elsner,T.Hutzelmann,S.Banescu,A.Pretschner,and
| [83] Z. Jiang, | S. Gan, | A. Herrera, | F.  | Toffalini, | L. Romerio, | C. Tang, |     |     |     |     |     |     |     |     |
| -------------- | ------- | ----------- | --- | ---------- | ----------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
M. Egele, C. Zhang, and M. Payer, “Evocatio: Conjuring Bug M.Böhme,“FuzzTastic:AFine-grained,Fuzzer-agnosticCoverage
CapabilitiesfromaSinglePoC,”inACMConferenceonComputer Analyzer,” in International Conference on Software Engineering
| andCommunicationsSecurity(CCS),2022. |     |     |     |     |     |     | (ICSE),2022. |     |     |     |     |     |     |     |
| ------------------------------------ | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
[84] Z.-M. Jiang, J.-J. Bai, K. Lu, and S.-M. Hu, “Context-Sensitive [104] Q. Liu, F. Toffalini, Y. Zhou, and M. Payer, “VIDEZZO:
andDirectionalConcurrencyFuzzingforData-RaceDetection,”in Dependency-aware Virtual Device Fuzzing,” in IEEE Symposium
Symposium on Network and Distributed System Security (NDSS), onSecurityandPrivacy(S&P),2023.
2022. [105] Y. Liu, S. Chen, Y. Xie, Y. Wang, L. Chen, B. Wang, Y. Zeng,
[85] J. Jung, H. Hu, D. Solodukhin, D. Pagan, K. H. Lee, and T. Kim, Z. Xue, and P. Su, “VD-Guard: DMA Guided Fuzzing for Hyper-
“Fuzzification:Anti-FuzzingTechniques,”inUSENIXSecuritySym- visor Virtual Device,” in IEEE/ACM International Conference on
AutomatedSoftwareEngineering(ASE),2023.
posium,2019.
[86] J. Jung, S. Tong, H. Hu, J. Lim, Y. Jin, and T. Kim, “WINNIE: [106] Y. Liu, Y. Wang, P. Su, Y. Yu,and X. Jia, “InstruGuard: Find and
|         |         |              |     |              |           |          | Fix         | Instrumentation | Errors        | for | Coverage-based |              | Greybox | Fuzzing,” |
| ------- | ------- | ------------ | --- | ------------ | --------- | -------- | ----------- | --------------- | ------------- | --- | -------------- | ------------ | ------- | --------- |
| Fuzzing | Windows | Applications |     | with Harness | Synthesis | and Fast |             |                 |               |     |                |              |         |           |
|         |         |              |     |              |           |          | in IEEE/ACM |                 | International |     | Conference     | on Automated |         | Software  |
Cloning,”inSymposiumonNetworkandDistributedSystemSecurity
| (NDSS),2021. |     |     |     |     |     |     | Engineering(ASE),2021. |     |     |     |     |     |     |     |
| ------------ | --- | --- | --- | --- | --- | --- | ---------------------- | --- | --- | --- | --- | --- | --- | --- |
[107] D.Liyanage,M.Böhme,C.Tantithamthavorn,andS.Lipp,“Reach-
[87] K.Kim,D.R.Jeong,C.H.Kim,Y.Jang,I.Shin,andB.Lee,“HFL:
|     |     |     |     |     |     |     | able | Coverage: | Estimating | Saturation | in  | Fuzzing,” | in International |     |
| --- | --- | --- | --- | --- | --- | --- | ---- | --------- | ---------- | ---------- | --- | --------- | ---------------- | --- |
HybridFuzzingontheLinuxKernel,”inSymposiumonNetworkand
DistributedSystemSecurity(NDSS),2020. ConferenceonSoftwareEngineering(ICSE),2023.
[108] C.Luo,W.Meng,andP.Li,“SelectFuzz:EfficientDirectedFuzzing
| [88] G. Klees, | A.  | Ruef, B. Cooper, | S.  | Wei, and | M. Hicks, | “Evaluating |     |     |     |     |     |     |     |     |
| -------------- | --- | ---------------- | --- | -------- | --------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
FuzzTesting,”inACMConferenceonComputerandCommunica- with Selective Path Exploration,” in IEEE Symposium on Security
| tionsSecurity(CCS),2018. |     |     |     |     |     |     | andPrivacy(S&P),2023. |     |     |     |     |     |     |     |
| ------------------------ | --- | --- | --- | --- | --- | --- | --------------------- | --- | --- | --- | --- | --- | --- | --- |
[109] Z.Luo,J.Yu,F.Zuo,J.Liu,Y.Jiang,T.Chen,A.Roychoudhury,
[89] J.Kukucka,L.Pina,P.Ammann,andJ.Bell,“CONFETTI:Ampli-
fyingConcolicGuidanceforFuzzers,”inIEEE/ACMInternational andJ.Sun,“Bleem:PacketSequenceOrientedFuzzingforProtocol
ConferenceonAutomatedSoftwareEngineering(ASE),2022. Implementations,”inUSENIXSecuritySymposium,2023.
[110] C.Lyu,S.Ji,C.Zhang,Y.Li,W.-H.Lee,Y.Song,andR.Beyah,
| [90] lafintel, | “laf-intel | -   | Circumventing |     | Fuzzing | Roadblocks |     |     |     |     |     |     |     |     |
| -------------- | ---------- | --- | ------------- | --- | ------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
with Compiler Transformations.” [Online]. Available: https: “MOPT: OptimizedMutationSchedulingforFuzzers,”in USENIX
SecuritySymposium,2019.
//lafintel.wordpress.com
[111] C.Lyu,J.Xu,S.Ji,X.Zhang,Q.Wang,B.Zhao,G.Pan,W.Cao,
[91] G.Lee,W.Shim,andB.Lee,“Constraint-guidedDirectedGreybox
Fuzzing,”inUSENIXSecuritySymposium,2021. P.Chen,andR.Beyah,“MINER:AHybridData-DrivenApproach
forRESTAPIFuzzing,”inUSENIXSecuritySymposium,2023.
| [92] M. | Lee, S. Cha, | and H. | Oh, “Learning |     | Seed-Adaptive | Mutation |             |           |     |         |         |         |           |       |
| ------- | ------------ | ------ | ------------- | --- | ------------- | -------- | ----------- | --------- | --- | ------- | ------- | ------- | --------- | ----- |
|         |              |        |               |     |               |          | [112] V. J. | M. Manès, | H.  | Han, C. | Han, S. | K. Cha, | M. Egele, | E. J. |
StrategiesforGreyboxFuzzing,”inIEEE/ACMInternationalCon- Schwartz, and M. Woo, “The Art, Science, and Engineering of
ferenceonAutomatedSoftwareEngineering(ASE),2023.
|     |     |     |     |     |     |     | Fuzzing: | A Survey,” | IEEE | Transactions |     | on Software | Engineering, |     |
| --- | --- | --- | --- | --- | --- | --- | -------- | ---------- | ---- | ------------ | --- | ----------- | ------------ | --- |
[93] S.Lee,H.Han,S.K.Cha,andS.Son,“Montage:ANeuralNetwork vol.47,no.11,pp.2312–2331,2021.
Language Model-Guided JavaScript Engine Fuzzer,” in USENIX [113] V.J.M.Manès,S.Kim,andS.K.Cha,“Ankou:GuidingGrey-box
SecuritySymposium,2020.
FuzzingtowardsCombinatorialDifference,”inIEEE/ACMInterna-
[94] C.Lemieux,R.Padhye,K.Sen,andD.Song,“PerfFuzz:Automat- tionalConferenceonAutomatedSoftwareEngineering(ASE),2020.
icallyGeneratingPathologicalInputs,”inInternationalSymposium
|     |     |     |     |     |     |     | [114] M. | Matz, | “Comment | 1,” | 2018. | [Online]. |     | Available: |
| --- | --- | --- | --- | --- | --- | --- | -------- | ----- | -------- | --- | ----- | --------- | --- | ---------- |
onSoftwareTestingandAnalysis(ISSTA),2018.
https://gcc.gnu.org/bugzilla/show_bug.cgi?id=87675#c1
[95] C. Lemieux and K. Sen, “FairFuzz: A Targeted Mutation Strategy [115] R. Meng, Z. Dong, J. Li, I. Beschastnikh, and A. Roychoud-
| forIncreasingGreyboxFuzz |     |     | TestingCoverage,”in |     |     | IEEE/ACMIn- |       |              |          |     |              |         |           |     |
| ------------------------ | --- | --- | ------------------- | --- | --- | ----------- | ----- | ------------ | -------- | --- | ------------ | ------- | --------- | --- |
|                          |     |     |                     |     |     |             | hury, | “Linear-time | Temporal |     | Logic guided | Greybox | Fuzzing,” | in  |
ternationalConferenceonAutomatedSoftwareEngineering(ASE),
IEEE/ACMInternationalConferenceonAutomatedSoftwareEngi-
2018.
neering(ASE),2022.
| [96] W. | Li, J. Ruan, | G. Yi, L. | Cheng, | X. Luo, | and H. | Cai, “PolyFuzz: |                |     |         |                  |     |        |         |          |
| ------- | ------------ | --------- | ------ | ------- | ------ | --------------- | -------------- | --- | ------- | ---------------- | --- | ------ | ------- | -------- |
|         |              |           |        |         |        |                 | [116] R. Meng, | G.  | Pirlea, | A. Roychoudhury, |     | and I. | Sergey, | “Greybox |
HolisticGreyboxFuzzingofMulti-LanguageSystems,”inUSENIX
FuzzingofDistributedSystems,”inACMConferenceonComputer
| SecuritySymposium,2023. |     |     |     |     |     |     | andCommunicationsSecurity(CCS),2023. |     |     |     |     |     |     |     |
| ----------------------- | --- | --- | --- | --- | --- | --- | ------------------------------------ | --- | --- | --- | --- | --- | --- | --- |
[97] W.Li,J.Shi,F.Li,J.Lin,W.Wang,andL.Guan,“µAFL:Non-
[117] A.Mera,B.Feng,L.Lu,andE.Kirda,“DICE:AutomaticEmulation
| intrusive | Feedback-driven |     | Fuzzing | for Microcontroller |     | Firmware,” |     |     |     |     |     |     |     |     |
| --------- | --------------- | --- | ------- | ------------------- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
ofDMAInputChannelsforDynamicFirmwareAnalysis,”inIEEE
in IEEE/ACM International Conference on Automated Software SymposiumonSecurityandPrivacy(S&P),2021.
Engineering(ASE),2022.
|     |     |     |     |     |     |     | [118] J. Metzman, |     | L. Szekeres, | L.  | Simon, | R. Sprabery, | and | A. Arya, |
| --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ------------ | --- | ------ | ------------ | --- | -------- |
[98] Y. Li, Y. Xue, H. Chen, X. Wu, C. Zhang, X. Xie, H. Wang, and “FuzzBench:AnOpenFuzzerBenchmarkingPlatformandService,”
Y. Liu, “Cerebro: Context-aware Adaptive Fuzzing for Effective inACMJointEuropeanSoftwareEngineeringConferenceandSym-
Vulnerability Detection,” in ACM Joint European Software Engi- posium on the Foundations of Software Engineering (ESEC/FSE),
| neeringConferenceandSymposiumontheFoundationsofSoftware |     |     |     |     |     |     | 2021. |     |     |     |     |     |     |     |
| ------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
Engineering(ESEC/FSE),2019.
[119] M.Muench,J.Stijohann,F.Kargl,A.Francillon,andD.Balzarotti,
[99] Y.Li,S.Ji,Y.Chen,S.Liang,W.-H.Lee,Y.Chen,C.Lyu,C.Wu, “WhatYouCorruptIsNotWhatYouCrash:ChallengesinFuzzing
R. Beyah, P. Cheng, K. Lu, and T. Wang, “UNIFUZZ: A Holistic Embedded Devices,” in Symposium on Network and Distributed
andPragmaticMetrics-DrivenPlatformforEvaluatingFuzzers,”in SystemSecurity(NDSS),2018.
USENIXSecuritySymposium,2021.
|                 |     |          |          |               |     |                  | [120] C. Myung, | G.          | Lee, and | B. Lee, | “MundoFuzz: |         | Hypervisor  | Fuzzing |
| --------------- | --- | -------- | -------- | ------------- | --- | ---------------- | --------------- | ----------- | -------- | ------- | ----------- | ------- | ----------- | ------- |
| [100] J. Liang, | M.  | Wang, C. | Zhou, Z. | Wu, Y. Jiang, | J.  | Liu, Z. Liu, and |                 |             |          |         |             |         |             |         |
|                 |     |          |          |               |     |                  | with            | Statistical | Coverage | Testing | and         | Grammar | Inference,” | in      |
J.Sun,“PATA:FuzzingwithPathAwareTaintAnalysis,”inIEEE USENIXSecuritySymposium,2022.
SymposiumonSecurityandPrivacy(S&P),2022.
|     |     |     |     |     |     |     | [121] S. Nagy | and | M. Hicks, | “Full-Speed |     | Fuzzing: | Reducing | Fuzzing |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --------- | ----------- | --- | -------- | -------- | ------- |
[101] “LibFuzzer - A Library for Coverage-guided Wuzz Testing.” Overhead through Coverage-Guided Tracing,” in IEEE Symposium
[Online].Available:https://llvm.org/docs/LibFuzzer.html onSecurityandPrivacy(S&P),2019.

[122] S. Nagy, A. Nguyen-Tuong, J. D. Hiser, J. W. Davidson, and [142] ——,“Nyx:GreyboxHypervisorFuzzingusingFastSnapshotsand
M. Hicks, “Same Coverage, Less Bloat: Accelerating Binary-only AffineTypes,”inUSENIXSecuritySymposium,2021.
| Fuzzing | with | Coverage-preserving |     | Coverage-guided |     | Tracing,” | in  |          |         |           |     |            |          |                 |     |
| ------- | ---- | ------------------- | --- | --------------- | --- | --------- | --- | -------- | ------- | --------- | --- | ---------- | -------- | --------------- | --- |
|         |      |                     |     |                 |     |           |     | [143] L. | Seidel, | D. Maier, | and | M. Muench, | “Forming | Faster Firmware |     |
ACMConferenceonComputerandCommunicationsSecurity(CCS), Fuzzers,”inUSENIXSecuritySymposium,2023.
2021.
|     |     |     |     |     |     |     |     | [144] A. | Shah, D. | She, | S. Sadhu, | K. Singal, | P. Coffman, | and | S. Jana, |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | -------- | ---- | --------- | ---------- | ----------- | --- | -------- |
[123] R. Natella and V.-T. Pham, “ProFuzzBench: A Benchmark for “MC2:RigorousandEfficientDirectedGreyboxFuzzing,”inACM
StatefulProtocolFuzzing,”inInternationalSymposiumonSoftware Conference on Computer and Communications Security (CCS),
| TestingandAnalysis(ISSTA),2021. |     |     |     |     |     |     |     | 2022. |     |     |     |     |     |     |     |
| ------------------------------- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
[124] H. L. Nguyen and L. Grunske, “BEDIVFUZZ: Integrating Behav- [145] D.She,R.Krishna,L.Yan,S.Jana,andB.Ray,“MTFuzz:Fuzzing
ioral Diversity into Generator-based Fuzzing,” in IEEE/ACM In- withaMulti-taskNeuralNetwork,”inACMJointEuropeanSoftware
ternationalConferenceonAutomatedSoftwareEngineering(ASE), Engineering Conference and Symposium on the Foundations of
2022.
SoftwareEngineering(ESEC/FSE),2020.
[125] H.L.Nguyen,N.Nassar,T.Kehrer,andL.Grunske,“MoFuzz:A [146] D.She,K.Pei,D.Epstein,J.Yang,B.Ray,andS.Jana,“NEUZZ:
FuzzerSuiteforTestingModel-DrivenSoftwareEngineeringTools,” Efficient Fuzzing with Neural Program Smoothing,” in IEEE Sym-
| in  | IEEE/ACM | International | Conference |     | on Automated | Software |     |     |     |     |     |     |     |     |     |
| --- | -------- | ------------- | ---------- | --- | ------------ | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
posiumonSecurityandPrivacy(S&P),2019.
Engineering(ASE),2020.
|     |     |     |     |     |     |     |     | [147] D. | She, A. | Shah, | and S. | Jana, “Effective |     | Seed Scheduling | for |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------- | ----- | ------ | ---------------- | --- | --------------- | --- |
[126] S.Nilizadeh,Y.Noller,andC.S.Pasareanu,“DifFuzz:Differential Fuzzing with Graph Centrality Analysis,” in IEEE Symposium on
Fuzzing for Side-channel Analysis,” in IEEE/ACM International SecurityandPrivacy(S&P),2022.
ConferenceonAutomatedSoftwareEngineering(ASE),2019.
|     |     |     |     |     |     |     |     | [148] Z. | Shen, R. | Roongta, | and | B. Dolan-Gavitt, |     | “Drifuzz: Harvesting |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | -------- | -------- | --- | ---------------- | --- | -------------------- | --- |
[127] D. Paaßen, S. Surminski, M. Rodler, and L. Davi, “My Fuzzer Bugs in Device Drivers from Golden Seeds,” in USENIX Security
| BeatsThemAll!DevelopingaFrameworkforFairEvaluationand |     |              |             |     |           |             |     | Symposium,2022.                                            |     |     |     |     |     |     |     |
| ----------------------------------------------------- | --- | ------------ | ----------- | --- | --------- | ----------- | --- | ---------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| Comparison                                            |     | of Fuzzers,” | in European |     | Symposium | on Research | in  |                                                            |     |     |     |     |     |     |     |
|                                                       |     |              |             |     |           |             |     | [149] J.Shi,Z.Wang,Z.Feng,Y.Lan,S.Qin,W.You,W.Zou,M.Payer, |     |     |     |     |     |     |     |
ComputerSecurity(ESORICS),2021. and C. Zhang, “AIFORE: Smart Fuzzing Based on Automatic In-
[128] L. Padgham, Y. Lee, S. Sadiq, M. Winikoff, A. Fekete, putFormatReverseEngineering,”inUSENIXSecuritySymposium,
| S.  | MacDonell, | D. Kaafar, | and | S. Zollmann, |     | “CORE Rankings.” |     | 2023. |     |     |     |     |     |     |     |
| --- | ---------- | ---------- | --- | ------------ | --- | ---------------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
[Online].Available:https://www.core.edu.au/conference-portal [150] D.Song,F.Hetzelt,J.Kim,B.B.Kang,J.-P.Seifert,andM.Franz,
[129] S. Pailoor, A. Aday, and S. Jana, “MoonShine: Optimizing OS “Agamotto: Accelerating Kernel Driver Fuzzing with Lightweight
FuzzerSeedSelectionwithTraceDistillation,”inUSENIXSecurity Virtual Machine Checkpoints,” in USENIX Security Symposium,
| Symposium,2018. |     |     |     |     |     |     |     | 2020. |     |     |     |     |     |     |     |
| --------------- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
[130] G. Pan, X.Lin, X. Zhang, Y. Jia,S. Ji, C. Wu, X.Ying, J. Wang, [151] S.Song,J.Hur,S.Kim,P.Rogers,andB.Lee,“R2Z2:Detecting
and Y. Wu, “V-Shuttle: Scalable and Semantics-Aware Hypervisor RenderingRegressionsinWebBrowsersthroughDifferentialFuzz
Virtual Device Fuzzing,” in ACM Conference on Computer and Testing,” in IEEE/ACM International Conference on Automated
SoftwareEngineering(ASE),2022.
CommunicationsSecurity(CCS),2021.
[131] J.Park,S.An,D.Youn,G.Kim,andS.Ryu,“JEST:N+1-version [152] S. Song, C. Song, Y. Jang, and B. Lee, “CrFuzz: Fuzzing Multi-
purposeProgramsthroughInputValidation,”inACMJointEuropean
DifferentialTestingofBothJavaScriptEnginesandSpecification,”
|     |          |               |            |     |              |          |     | Software | Engineering |     | Conference | and | Symposium | on the | Founda- |
| --- | -------- | ------------- | ---------- | --- | ------------ | -------- | --- | -------- | ----------- | --- | ---------- | --- | --------- | ------ | ------- |
| in  | IEEE/ACM | International | Conference |     | on Automated | Software |     |          |             |     |            |     |           |        |         |
Engineering(ASE),2021. tionsofSoftwareEngineering(ESEC/FSE),2020.
|          |          |             |          |     |                  |            |     | [153] L. | Stone, | R. Ranjan, | S.      | Nagy, and | M. Hicks, | “No Linux,  | No      |
| -------- | -------- | ----------- | -------- | --- | ---------------- | ---------- | --- | -------- | ------ | ---------- | ------- | --------- | --------- | ----------- | ------- |
| [132] S. | Park, W. | Xu, I. Yun, | D. Jang, | and | T. Kim, “Fuzzing | JavaScript |     |          |        |            |         |           |           |             |         |
|          |          |             |          |     |                  |            |     | Problem: | Fast   | and        | Correct | Windows   | Binary    | Fuzzing via | Target- |
EngineswithAspect-preservingMutation,”inIEEESymposiumon
SecurityandPrivacy(S&P),2020. embeddedSnapshotting,”inUSENIXSecuritySymposium,2023.
|          |          |                  |     |     |                 |         |     | [154] S.M.S.Talebi,H.Tavakoli,H.Zhang,Z.Zhang,A.A.Sani,and |     |     |     |     |     |     |     |
| -------- | -------- | ---------------- | --- | --- | --------------- | ------- | --- | ---------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| [133] H. | Peng, Y. | Shoshitaishvili, | and | M.  | Payer, “T-Fuzz: | Fuzzing | by  |                                                            |     |     |     |     |     |     |     |
Z.Qian,“Charm:FacilitatingDynamicAnalysisofDeviceDrivers
| Program | Transformation,” |     | in  | IEEE Symposium |     | on Security | and |     |     |     |     |     |     |     |     |
| ------- | ---------------- | --- | --- | -------------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Privacy(S&P),2018. ofMobileSystems,”inUSENIXSecuritySymposium,2018.
|     |     |     |     |     |     |     |     | [155] E.vanderKouwe,G.Heiser,D.Andriesse,H.Bos,andC.Giuffrida, |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
[134] H.Peng,Z.Yao,A.A.Sani,D.J.Tian,andM.Payer,“GLeeFuzz:
“SoK:BenchmarkingFlawsinSystemsSecurity,”inIEEEEuropean
| Fuzzing | WebGL | Through | Error | Message | Guided | Mutation,” | in  |     |     |     |     |     |     |     |     |
| ------- | ----- | ------- | ----- | ------- | ------ | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
USENIXSecuritySymposium,2023. SymposiumonSecurityandPrivacy(EuroS&P),2019.
|          |             |                |     |           |           |      |        | [156] A. | Vargha | and H. | D. Delaney, | “A  | Critique | and Improvement | of  |
| -------- | ----------- | -------------- | --- | --------- | --------- | ---- | ------ | -------- | ------ | ------ | ----------- | --- | -------- | --------------- | --- |
| [135] S. | Poeplau and | A. Francillon, |     | “Symbolic | execution | with | SymCC: |          |        |        |             |     |          |                 |     |
Don’tinterpret,compile!”inUSENIXSecuritySymposium,2020. the CL Common Language Effect Size Statistics of McGraw and
|     |     |     |     |     |     |     |     |        | Journal | of  | Educational | and | Behavioral | Statistics, |          |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------- | --- | ----------- | --- | ---------- | ----------- | -------- |
|     |     |     |     |     |     |     |     | Wong,” |         |     |             |     |            |             | vol. 25, |
[136] ——,“SymQEMU:Compilation-basedSymbolicExecutionforBi- no.2,pp.101–132,2000.
naries,”inSymposiumonNetworkandDistributedSystemSecurity
(NDSS),2021. [157] V. Vikram, R. Padhye, and K. Sen, “Growing A Test Corpus
|     |     |     |     |     |     |     |     | with | Bonsai | Fuzzing,” | in  | IEEE/ACM | International | Conference | on  |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ------ | --------- | --- | -------- | ------------- | ---------- | --- |
[137] J. Ruge, J. Classen, F. Gringoli, and M. Hollick, “Frankenstein: AutomatedSoftwareEngineering(ASE),2021.
Advanced Wireless Fuzzing to Exploit New Bluetooth Escalation [158] H. Wang, X. Xie, Y. Li, C. Wen, Y. Li, Y. Liu, S. Qin, H. Chen,
Targets,”inUSENIXSecuritySymposium,2020.
|     |     |     |     |     |     |     |     | and | Y. Sui, | “Typestate-guided |     | Fuzzer | for Discovering | Use-after- |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | ----------------- | --- | ------ | --------------- | ---------- | --- |
[138] L. Sachs, Applied Statistics: A Handbook of Techniques, 2nd ed., free Vulnerabilities,” in IEEE/ACM International Conference on
ser. Springer Series in Statistics. New York, NY: Springer New AutomatedSoftwareEngineering(ASE),2020.
York,1984.
|     |     |     |     |     |     |     |     | [159] H.Wang,J.Chen,C.Xie,S.Liu,Z.Wang,Q.Shen,andY.Zhao, |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
[139] C. Salls, C. Jindal, J. Corina, C. Kruegel, and G. Vigna, “Token- “MLIRSmith: Random Program Generation for Fuzzing MLIR
LevelFuzzing,”inUSENIXSecuritySymposium,2021.
CompilerInfrastructure,”inIEEE/ACMInternationalConferenceon
AutomatedSoftwareEngineering(ASE),2023.
[140] T.Scharnowski,N.Bars,M.Schloegel,E.Gustafson,M.Muench,
G. Vigna, C. Kruegel, T. Holz, and A. Abbasi, “Fuzzware: Us- [160] J. Wang, B. Chen, L. Wei, and Y. Liu, “Superion: Grammar-
| ing | Precise MMIO | Modeling |     | for Effective | Firmware | Fuzzing,” | in  |       |         |           |     |             |               |            |     |
| --- | ------------ | -------- | --- | ------------- | -------- | --------- | --- | ----- | ------- | --------- | --- | ----------- | ------------- | ---------- | --- |
|     |              |          |     |               |          |           |     | aware | Greybox | Fuzzing,” |     | in IEEE/ACM | International | Conference |     |
USENIXSecuritySymposium,2022. onAutomatedSoftwareEngineering(ASE),2019.
[141] S. Schumilo, C. Aschermann, A. Abbasi, S. Wörner, and T. Holz, [161] J.Wang,Z.Zhang,S.Liu,X.Du,andJ.Chen,“FuzzJIT:Oracle-
“HYPER-CUBE: High-Dimensional Hypervisor Fuzzing,” in Sym- EnhancedFuzzingforJavaScriptEngineJITCompiler,”inUSENIX
posiumonNetworkandDistributedSystemSecurity(NDSS),2020. SecuritySymposium,2023.

[162] Y. Wang, X. Jia, Y. Liu, K. Zeng, T. Bao, D. Wu, and P. Su, [179] A. Zeller, S. Just, and K. Greshake, “When Re-
“NotAllCoverageMeasurementsAreEqual:FuzzingbyCoverage sults Are All That Matters: Consequences,” 2019.
AccountingforInputPrioritization,”inSymposiumonNetworkand [Online]. Available: https://andreas-zeller.blogspot.com/2019/10/
DistributedSystemSecurity(NDSS),2020. when-results-are-all-that-matters.html
[163] A. Wei, Y. Deng, C. Yang, and L. Zhang, “Free Lunch for [180] G. Zhang, P. Wang, T. Yue, X. Kong, S. Huang, X. Zhou, and
Testing: Fuzzing Deep-Learning Libraries from Open Source,” in K.Lu,“MobFuzz:AdaptiveMulti-objectiveOptimizationinGray-
IEEE/ACMInternationalConferenceonAutomatedSoftwareEngi- box Fuzzing,” in Symposium on Network and Distributed System
| neering(ASE),2022. |     |     |     |     |     |     |     | Security(NDSS),2022. |     |     |     |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | -------------------- | --- | --- | --- | --- | --- | --- | --- |
[164] C. Wen, H. Wang, Y. Li, S. Qin, Y. Liu, Z. Xu, H. Chen, X. Xie, [181] Q. Zhang, J. Wang, and M. Kim, “HeteroFuzz: Fuzz Testing to
G. Pu, and T. Liu, “MemLock: Memory Usage Guided Fuzzing,” Detect Platform Dependent Divergence for Heterogeneous Appli-
|     |          |               |     |            |     |           |          | cations,” | in  | ACM Joint | European | Software | Engineering |     | Confer- |
| --- | -------- | ------------- | --- | ---------- | --- | --------- | -------- | --------- | --- | --------- | -------- | -------- | ----------- | --- | ------- |
| in  | IEEE/ACM | International |     | Conference | on  | Automated | Software |           |     |           |          |          |             |     |         |
Engineering(ASE),2020. ence and Symposium on the Foundations of Software Engineering
(ESEC/FSE),2021.
[165] M.Wu,M.Lu,H.Cui,J.Chen,Y.Zhang,andL.Zhang,“JITfuzz:
Coverage-Guided Fuzzing for JVM Just-in-Time Compilers,” in [182] Y. Zhang, C. Pang, S. Nagy, X. Chen, and J. Xu, “Profile-guided
IEEE/ACM International Conference on Automated Software En- System Optimizations for Accelerated Greybox Fuzzing,” in ACM
gineering(ASE),2023. Conference on Computer and Communications Security (CCS),
| [166] M.Wu,Y.Ouyang,M.Lu,J.Chen,Y.Zhao,H.Cui,G.Yang,and |     |     |     |     |     |     |     | 2023. |     |     |     |     |     |     |     |
| ------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
Y.Zhang,“SJFuzz:Seed&MutatorSchedulingforJVMFuzzing,” [183] Z. Zhang, Z. Patterson, M. Hicks, and S. Wei, “FIXREVERTER:
inACMJointEuropeanSoftwareEngineeringConferenceandSym- A Realistic Bug Injection Methodology for Benchmarking Fuzz
Testing,”inUSENIXSecuritySymposium,2022.
| posium | on the | Foundations |     | of Software | Engineering | (ESEC/FSE), |     |     |     |     |     |     |     |     |     |
| ------ | ------ | ----------- | --- | ----------- | ----------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
2023. [184] Z. Zhang, W. You, G. Tao, Y. Aafer, X. Liu, and X. Zhang,
“StochFuzz:SoundandCost-effectiveFuzzingofStrippedBinaries
| [167] V. | Wüstholz | and M. | Christakis, | “Targeted | Greybox | Fuzzing | with |     |             |     |            |             |         |           |     |
| -------- | -------- | ------ | ----------- | --------- | ------- | ------- | ---- | --- | ----------- | --- | ---------- | ----------- | ------- | --------- | --- |
|          |          |        |             |           |         |         |      | by  | Incremental | and | Stochastic | Rewriting,” | in IEEE | Symposium | on  |
StaticLookaheadAnalysis,”inIEEE/ACMInternationalConference
onAutomatedSoftwareEngineering(ASE),2020. SecurityandPrivacy(S&P),2021.
|          |        |          |          |     |         |         |           | [185] B. | Zhao, Z. | Li, S. | Qin, Z. | Ma, M. | Yuan, W. | Zhu, Z. | Tian, and |
| -------- | ------ | -------- | -------- | --- | ------- | ------- | --------- | -------- | -------- | ------ | ------- | ------ | -------- | ------- | --------- |
| [168] M. | Xu, S. | Kashyap, | H. Zhao, | and | T. Kim, | “Krace: | Data Race |          |          |        |         |        |          |         |           |
C.Zhang,“StateFuzz:SystemCall-BasedState-AwareLinuxDriver
FuzzingforKernelFileSystems,”inIEEESymposiumonSecurity
andPrivacy(S&P),2020. Fuzzing,”inUSENIXSecuritySymposium,2022.
|     |     |     |     |     |     |     |     | [186] H.Zheng,J.Zhang,Y.Huang,Z.Ren,H.Wang,C.Cao,Y.Zhang, |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
[169] P.Xu,Y.Wang,H.Hu,andP.Su,“COOPER:TestingtheBinding
CodeofScriptingLanguageswithCooperativeMutation,”inSym- F. Toffalini, and M. Payer, “FISHFUZZ: Catch Deeper Bugs by
posiumonNetworkandDistributedSystemSecurity(NDSS),2022. ThrowingLargerNets,”inUSENIXSecuritySymposium,2023.
|          |              |     |          |       |            |         |          | [187] Y. | Zheng, | A. Davanian, | H.  | Yin, C. | Song, H. | Zhu, and | L. Sun, |
| -------- | ------------ | --- | -------- | ----- | ---------- | ------- | -------- | -------- | ------ | ------------ | --- | ------- | -------- | -------- | ------- |
| [170] W. | Xu, H. Moon, | S.  | Kashyap, | P.-N. | Tseng, and | T. Kim, | “Fuzzing |          |        |              |     |         |          |          |         |
File Systems via Two-Dimensional Input Space Exploration,” in “FIRM-AFL: High-Throughput Greybox Fuzzing of IoT Firmware
|     |     |     |     |     |     |     |     | via | Augmented | Process | Emulation,” |     | in USENIX | Security | Sympo- |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --------- | ------- | ----------- | --- | --------- | -------- | ------ |
IEEESymposiumonSecurityandPrivacy(S&P),2019.
sium,2019.
| [171] W.   | Xu, S. | Park, and | T. Kim, | “FREEDOM:      | Engineering |             | a State- |          |          |                         |           |         |               |                  |       |
| ---------- | ------ | --------- | ------- | -------------- | ----------- | ----------- | -------- | -------- | -------- | ----------------------- | --------- | ------- | ------------- | ---------------- | ----- |
|            |        |           |         |                |             |             |          | [188] C. | Zhou, M. | Wang,                   | J. Liang, | Z. Liu, | and Y. Jiang, | “Zeror:          | Speed |
| of-the-Art | DOM    | Fuzzer,”  | in      | ACM Conference |             | on Computer | and      |          |          |                         |           |         |               |                  |       |
|            |        |           |         |                |             |             |          | Up       | Fuzzing  | with Coverage-sensitive |           |         | Tracing       | and Scheduling,” | in    |
CommunicationsSecurity(CCS),2020.
IEEE/ACMInternationalConferenceonAutomatedSoftwareEngi-
[172] W.You,X.Liu,S.Ma,D.M.Perry,X.Zhang,andB.Liang,“SLF:
neering(ASE),2020.
| Fuzzing | without | Valid | Seed | Inputs,” | in IEEE/ACM | International |     |                                                          |     |     |     |     |     |     |     |
| ------- | ------- | ----- | ---- | -------- | ----------- | ------------- | --- | -------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
|         |         |       |      |          |             |               |     | [189] C.Zhou,Q.Zhang,M.Wang,L.Guo,J.Liang,Z.Liu,M.Payer, |     |     |     |     |     |     |     |
ConferenceonAutomatedSoftwareEngineering(ASE),2019.
andY.Jiang,“Minerva:BrowserAPIFuzzingwithDynamicmod-
| [173] W. | You, X. | Wang, | S. Ma, | J. Huang, | X. Zhang, | X.  | Wang, and |     |            |        |       |          |          |             |      |
| -------- | ------- | ----- | ------ | --------- | --------- | --- | --------- | --- | ---------- | ------ | ----- | -------- | -------- | ----------- | ---- |
|          |         |       |        |           |           |     |           | ref | Analysis,” | in ACM | Joint | European | Software | Engineering | Con- |
B. Liang, “ProFuzzer: On-the-fly Input Type Probing for Better ferenceandSymposiumontheFoundationsofSoftwareEngineering
| Zero-DayVulnerabilityDiscovery,”inIEEESymposiumonSecurity |     |     |     |     |     |     |     | (ESEC/FSE),2022. |     |     |     |     |     |     |     |
| --------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- | --- |
andPrivacy(S&P),2019. [190] S. Zhou, Z. Yang, D. Qiao, P. Liu, M. Yang, Z. Wang, and
[174] Y. Yu, X. Jia, Y. Liu, Y. Wang, Q. Sang, C. Zhang, and C.Wu,“Ferry:State-AwareSymbolicExecutionforExploringState-
P. Su, “HTFuzz: Heap Operation Sequence Sensitive Fuzzing,” in DependentProgramPaths,”inUSENIXSecuritySymposium,2022.
IEEE/ACMInternationalConferenceonAutomatedSoftwareEngi-
|     |     |     |     |     |     |     |     | [191] W. | Zhou, | L. Zhang, | L. Guan, | P.  | Liu, and | Y. Zhang, | “What |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ----- | --------- | -------- | --- | -------- | --------- | ----- |
neering(ASE),2022.
|     |     |     |     |     |     |     |     | Your | Firmware | Tells | You Is | Not How | You Should | Emulate | It: A |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | -------- | ----- | ------ | ------- | ---------- | ------- | ----- |
[175] T. Yue, P. Wang, Y. Tang, E. Wang, B. Yu, K. Lu, and X. Zhou, Specification-Guided Approach for Firmware Emulation,” in ACM
| “EcoFuzz:                                               | Adaptive |     | Energy-Saving |     | Greybox Fuzzing | as  | a Variant |            |     |             |     |                |     |          |        |
| ------------------------------------------------------- | -------- | --- | ------------- | --- | --------------- | --- | --------- | ---------- | --- | ----------- | --- | -------------- | --- | -------- | ------ |
|                                                         |          |     |               |     |                 |     |           | Conference |     | on Computer | and | Communications |     | Security | (CCS), |
| oftheAdversarialMulti-ArmedBandit,”inUSENIXSecuritySym- |          |     |               |     |                 |     |           | 2022.      |     |             |     |                |     |          |        |
posium,2020.
|     |     |     |     |     |     |     |     | [192] X. | Zhu and | M. Böhme, | “Regression |     | Greybox | Fuzzing,” | in ACM |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------- | --------- | ----------- | --- | ------- | --------- | ------ |
[176] I. Yun, S. Lee, M. Xu, Y. Jang, and T. Kim, “QSYM: A Prac- Conference on Computer and Communications Security (CCS),
| tical | Concolic | Execution | Engine | Tailored | for | Hybrid Fuzzing,” | in  | 2021. |     |     |     |     |     |     |     |
| ----- | -------- | --------- | ------ | -------- | --- | ---------------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
USENIXSecuritySymposium,2018.
|     |     |     |     |     |     |     |     | [193] X.Zhu,S.Wen,S.Camtepe,andY.Xiang,“Fuzzing:ASurveyfor |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
[177] M. Zalewski, “American Fuzzy Lop.” [Online]. Available: Roadmap,”ACMComputingSurveys(CSUR),vol.54,no.11s,pp.
| http://lcamtuf.coredump.cx/afl/ |     |     |     |     |     |     |     | 1–36,2022. |     |     |     |     |     |     |     |
| ------------------------------- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
[178] A. Zeller, R. Gopinath, M. Böhme, G. Fraser, and [194] S. Österlund, K. Razavi, H. Bos, and C. Giuffrida, “ParmeSan:
C. Holler, “The Fuzzing Book,” 2019. [Online]. Available: Sanitizer-guided Greybox Fuzzing,” in USENIX Security Sympo-
| https://www.fuzzingbook.org/ |     |     |     |     |     |     |     | sium,2020. |     |     |     |     |     |     |     |
| ---------------------------- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |

| Appendix | A.  |     |     |     |     |     |
| -------- | --- | --- | --- | --- | --- | --- |
Meta-Review
Thefollowingmeta-reviewwaspreparedbytheprogram
| committee     | for the 2024 | IEEE   | Symposium |         | on Security | and         |
| ------------- | ------------ | ------ | --------- | ------- | ----------- | ----------- |
| Privacy (S&P) | as part      | of the | review    | process | as          | detailed in |
| the call for  | papers.      |        |           |         |             |             |
A.1. Summary
| This SoK       | submission      |           | selects 150  | papers       | from     | 2018-      |
| -------------- | --------------- | --------- | ------------ | ------------ | -------- | ---------- |
| 2023 published | in top-tier     |           | security     | and software |          | engineer-  |
| ing venues     | for fuzzing     | research. | It           | then         | performs | a meta-    |
| evaluation     | of each paper’s |           | evaluation   | in           | terms    | of experi- |
| mental design  | and adherence   |           | to generally |              | accepted | fuzzing    |
guidelinesusingKleesetal.asabaseline.Inaddition,eight
| papers are      | subject to     | artifact | evaluation.     |           | The conclusions |          |
| --------------- | -------------- | -------- | --------------- | --------- | --------------- | -------- |
| are stark:      | fuzzing papers | continue |                 | to fall   | short           | of known |
| best practices  | in conducting  |          | rigorous        | fuzzing   | research.       | An       |
| updated set     | of guidelines  | is       | then presented. |           |                 |          |
| A.2. Scientific | Contributions  |          |                 |           |                 |          |
| Independent     | Confirmation   |          | of              | Important | Results         | with     |
•
| Limited     | Prior Research |      |         |     |                |     |
| ----------- | -------------- | ---- | ------- | --- | -------------- | --- |
| • Addresses | a Long-Known   |      | Issue   |     |                |     |
| • Provides  | a Valuable     | Step | Forward | in  | an Established |     |
Field
| Other | (Reproducibility |     | Study) |     |     |     |
| ----- | ---------------- | --- | ------ | --- | --- | --- |
•
| A.3. Reasons   | for             | Acceptance  |                 |            |                 |          |
| -------------- | --------------- | ----------- | --------------- | ---------- | --------------- | -------- |
| 1) Fuzzing     | is an important |             | research        | area,      | and understand- |          |
| ing whether    | fuzzing         | papers      | hew             | to         | best practices  | in-      |
| tended         | to maximize     | the         | validity        | and        | reproducibility | of       |
| the results    | is important    |             |                 |            |                 |          |
| 2) The paper   | uses            | an overall  | strong          | review     | methodology     |          |
| 3) The         | paper examines  | a           | wide range      | of         | fuzzing         | papers   |
| over           | time and across | conferences |                 |            |                 |          |
| 4) The         | paper includes  | an          | artifact        | evaluation | on              | a subset |
| of the         | reviewed        | fuzzing     | papers          |            |                 |          |
| 5) The paper’s | observations    |             | are significant |            |                 |          |
