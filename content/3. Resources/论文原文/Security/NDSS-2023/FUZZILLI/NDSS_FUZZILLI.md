---
publish: true
---

FUZZILLI: Fuzzing for JavaScript JIT Compiler Vulnerabilities
Samuel Groß Simon Koch Lukas Bernhard
Google TU Braunschweig Ruhr University Bochum
saleo@google.com simon.koch@tu-braunschweig.de lukas.bernhard@rub.de
Thorsten Holz Martin Johns
CISPA Helmholtz Center for Information Security TU Braunschweig
holz@cispa.de m.johns@tu-braunschweig.de
Abstract—JavaScript has become an essential part of the on a single vulnerability. An attacker can chain a successful
Internet infrastructure, and today’s interactive web applications attack with an escape from the browser sandbox, gaining
would be inconceivable without this programming language.
unauthorized privileges by luring a victim to a malicious
On the downside, this interactivity implies that web appli-
website.
cations rely on an ever-increasing amount of computationally
intensive JavaScript code, which burdens the JavaScript engine Byitsverynature,JavaScriptasaprogramminglanguageis
responsible for efficiently executing the code. To meet these flexibleanddynamic.Becauseofthisflexibility,JIToptimiza-
rising performance demands, modern JavaScript engines ship tions require assumptions about the global state of the engine,
with sophisticated just-in-time (JIT) compilers. However, JIT
groups of related objects, or even a single object involved in
compilers are a complex technology and, consequently, provide
the optimized code segment. Such assumptions must either
a broad attack surface for potential faults that might even be
security-critical.Previousworkondiscoveringsoftwarefaultsin be proven true or protected by complex runtime mechanisms
JavaScript engines found many vulnerabilities, often using fuzz that notify the engine when a previously made assumption
testing.Unfortunately,thesefuzzingapproachesarenotdesigned is violated. Any assumption that turns out to be false but
to generate source code that actually triggers JIT semantics.
remains undetected during execution represents a significant
Consequently, JIT vulnerabilities are unlikely to be discovered
vulnerability,suchasinCVE-2018-4233(seeSectionIII-Bfor
by existing methods.
In this paper, we close this gap and present the first fuzzer details), a bug in the JIT compiler of WebKit. Consequently,
that focuses on JIT vulnerabilities. More specifically, we present JIT compilation bugs should be a focus of software testing.
thedesignandimplementationofanintermediaterepresentation A popular method for finding bugs in complex software
(IR)thatfocusesondiscoveringJITcompilervulnerabilities.We systems such as JavaScript engines is fuzz testing (fuzzing for
implementedacompleteprototypeoftheproposedapproachand
short). Fuzzing involves testing software with many different
evaluated our fuzzer over a period of six months. In total, we
discovered17confirmedsecurityvulnerabilities.Ourresultsshow inputs and evaluating how the software responds to those
thattargetedJITfuzzingispossibleandadangerouslyneglected inputs. The underlying hope is to find corner cases in the
gap in fuzzing coverage for JavaScript engines. software that lead to non-trivial crashes. An analyst can then
further investigate these crashes to create a proof-of-concept
I. INTRODUCTION
for an exploit that may break out of the JavaScript sandbox.
The modern Web is unimaginable without JavaScript (JS). In the past, fuzzing was mainly used to find vulnerabilities
Driven by powerful JavaScript frameworks such as Angu- in JavaScript engines, and several critical problems with
larJS [1], React [8], or jQuery [5], modern web content is JavaScriptengineswerefound[28],[28],[36],[43].However,
typically created entirely on the client side, rather than being previousfuzzingapproachestargetedJavaScriptengineswith-
delivered in the form of HTML [10], [40]. This evolution outfocusingonspecificcomponents[28],[31],[43]orfocused
caused increasing performance issues for existing JS engines onlyontheruntimeAPI[30].Suchapproachescanfindawide
that relied on simply interpreting JS code. As a result and to range of vulnerabilities, but more complex vulnerabilities that
enableadynamicwebexperience,modernwebbrowsershave require the concurrence of multiple preconditions have rarely
aggressivelymovedtowardsjust-in-time(JIT)compilationand been discovered. In particular, JIT compilation vulnerabilities
optimization of JS code. While JIT engines provide desirable are precisely such a type of vulnerability.
performance improvements, they make the execution of JS For JIT optimization to occur at all, certain conditions
codesignificantlymorecomplexandinherentlyexposealarge must be met: The engine must frequently execute the code
attack surface. Software vulnerabilities based on JIT compiler in question, and the code must behave predictably during
faultsareattractivetoattackersbecausetheyprovidepowerful the observation because only then the JIT compilation starts.
exploit primitives and typically allow code execution based These conditions imply that not only must the JS code be
structuredinaspecialwaytoemphasizethefaults,butitmust
alsobeexecutednumeroustimesinasimilarmannerandthen
Network and Distributed Systems Security (NDSS) Symposium 2023
change its behavior in an unpredictable way in order for the
27 February - 3 March 2023, San Diego, CA, USA
ISBN 1-891562-83-5 JS engine to encounter an error. This behavior is difficult to
https://dx.doi.org/10.14722/ndss.2023.24290
www.ndss-symposium.org

reproduce for a fuzzer that generates code snippets more or fuzzing [44]. For a comprehensive listing of recent fuzzing
less randomly via mutations. publications, we refer the reader to an online repository that
ClassicJSfuzzers[4]generateJSconstructsandwrapevery maintains a list of papers published in this area [9].
statementintry-catchblocksbecausetheycannotguaran-
|              |              |     |                |     |       |          |        | A. Fuzzing | Overview |     |     |     |     |     |     |
| ------------ | ------------ | --- | -------------- | --- | ----- | -------- | ------ | ---------- | -------- | --- | --- | --- | --- | --- | --- |
| tee semantic | correctness. |     | Unfortunately, |     | a JIT | compiler | treats |            |          |     |     |     |     |     |     |
codewrappedintry-catchdifferentlyfromcodethatisnot Fuzzing can be divided at a high level into several different
|          |         |            |       |       |             |     |       | approaches, | which | we  | briefly | explain | below. | These | general |
| -------- | ------- | ---------- | ----- | ----- | ----------- | --- | ----- | ----------- | ----- | --- | ------- | ------- | ------ | ----- | ------- |
| wrapped, | so many | JIT errors | elude | these | approaches. |     | Other |             |       |     |         |         |        |       |         |
work [28], [36], [43] generates test cases from existing JS approaches provide a rough classification, and in practice,
corpora.Relyingonpre-existingcorporarequiresasufficiently manyhybridsareused,sothataclearseparationisnotalways
| diverse set | of vulnerabilities |     | of  | a specific | type | to extrapolate |     | possible. |     |     |     |     |     |     |     |
| ----------- | ------------------ | --- | --- | ---------- | ---- | -------------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- |
similar ones. This requirement limits the ability to uncover a) GenerativeFuzzing: Generative-basedapproaches[7]
vulnerabilities dissimilar to existing test cases. In conclusion, generate each input from scratch, using generator functions
developing a targeted fuzzing approach to detect novel faults that output the relevant data. The main advantage of the gen-
inJITcompilationisachallengethathasnotyetbeentackled. erative approach is that the produced inputs are syntactically
In this paper, we address this research gap in fuzzing correct by design, since the generator functions respect the
coverageandproposethefirstfuzzerthatusesanintermediate underlying syntax expected by the program under test.
representation (IR) that focuses on discovering just-in-time b) Mutation Based Fuzzing: Mutation-based approaches
compiler vulnerabilities in JavaScript engines. Our IR allows use seed files and manipulate them according to certain rules,
us to generate new JavaScript programs without initial input and then continue with the slightly modified files as new seed
corpora,targetingtheJITcompiler.Furthermore,ourIRallows files. Mutations can be random and arbitrary, such as bit/byte
theimplementationofsemanticallymeaningfulmutationoper- flipsorrandomizedaddition/deletionofmessageparts,ormore
ations,suchassplicingmultipleinputprogramswhilerewiring targeted, such as replacing integers or strings with data points
instructionoperands,afeaturemissingincommonAST-based known to have had problems in the past (e.g., magic values
| fuzzing approaches. |     |     |          |          |     |               |     | such as | MAX INT  | or  | MIN    | INT).   |       |             |      |
| ------------------- | --- | --- | -------- | -------- | --- | ------------- | --- | ------- | -------- | --- | ------ | ------- | ----- | ----------- | ---- |
| We implemented      |     | the | proposed | approach |     | and performed |     |         |          |     |        |         |       |             |      |
|                     |     |     |          |          |     |               |     | Guided  | Fuzzing: |     | Guided | fuzzing | [17], | [18], [26], | [45] |
a comprehensive evaluation of our prototype on the major extends the approach used in mutation-based fuzzing and
JS engines Apple JavaScriptCore, Google V8, and Mozilla prunesthemutatedfilesbasedonrelevanceaccordingtosome
SpiderMonkey. We find that our fuzzer compares well with metric (e.g., coverage-guided fuzzing). A popular metric for
Superion, a state-of-the-art open-source fuzzer [43], on all pruning in language fuzzing is branch coverage. A fuzzer
| engines. | Furthermore, | we  | show | that Superion |     | cannot achieve |     |              |          |          |     |            |     |          |        |
| -------- | ------------ | --- | ---- | ------------- | --- | -------------- | --- | ------------ | -------- | -------- | --- | ---------- | --- | -------- | ------ |
|          |              |     |      |               |     |                |     | using branch | coverage | collects |     | data about | the | branches | of the |
significant code coverage gains when provided with a com- executedtargetprogramanddeletes/ignoresnewmutatedfiles
prehensive input corpus. In contrast, our approach performs for further consideration if they have not discovered any new
wellindifferentkindsofsetupsandweidentified17security- branchesduringtheirexecution.Thisapproachensuresthatthe
critical vulnerabilities. fuzzing process retains some momentum and does not reach
| Contributions. |         | In summary, | our | main           | contributions | are:  |     | a dead end. |       |          |     |           |     |                |     |
| -------------- | ------- | ----------- | --- | -------------- | ------------- | ----- | --- | ----------- | ----- | -------- | --- | --------- | --- | -------------- | --- |
|                |         |             |     |                |               |       |     | Structure   | Aware | Fuzzing: |     | Depending | on  | the underlying |     |
| • We           | present | the design  | and | implementation |               | of an | IR- |             |       |          |     |           |     |                |     |
programtobefuzzed,especiallyifitrequiresaspecificsyntax
| based | fuzzing | approach | targeting |     | JIT vulnerabilities |     | in  |     |     |     |     |     |     |     |     |
| ----- | ------- | -------- | --------- | --- | ------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
JS engines of modern web browsers. as input, e.g., an interpreter, itis challenging to generate valid
|        |               |     |             |     |          |              |     | mutations | of the | input | files. | In particular, | highly | structured |     |
| ------ | ------------- | --- | ----------- | --- | -------- | ------------ | --- | --------- | ------ | ----- | ------ | -------------- | ------ | ---------- | --- |
| • In a | comprehensive |     | evaluation, | we  | discover | 17 security- |     |           |        |       |        |                |        |            |     |
criticalvulnerabilitieswithourprototypeimplementation. input data [16], [21], [25], [31], [38] such as source code can
bechallenging,asrandomchangesarelikelytoresultininput
| A more | detailed | analysis | of  | the identified |     | vulnerabilities |     |           |                |     |           |            |          |     |          |
| ------ | -------- | -------- | --- | -------------- | --- | --------------- | --- | --------- | -------------- | --- | --------- | ---------- | -------- | --- | -------- |
|        |          |          |     |                |     |                 |     | data that | is immediately |     | rejected, | preventing | in-depth |     | testing. |
confirmedthatmostofthefaultsareindeedrelatedtothe
JIT compiler. To counter this problem, the fuzzer can be made aware of the
|     |     |     |     |     |     |     |     | required | input structure. |     | In this | paper, | we explore | the | use of |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ---------------- | --- | ------- | ------ | ---------- | --- | ------ |
• Weperformacomprehensivecomparisonagainstmodern
browser fuzzers and find that our approach outperforms an intermediate language for this purpose.
| the | state-of-the-art             | method |     | called | Superion. |     |     |               |          |              |     |       |         |         |     |
| --- | ---------------------------- | ------ | --- | ------ | --------- | --- | --- | ------------- | -------- | ------------ | --- | ----- | ------- | ------- | --- |
|     |                              |        |     |        |           |     |     | B. JavaScript | Fuzzing  |              |     |       |         |         |     |
|     | II. BACKGROUNDANDRELATEDWORK |        |     |        |           |     |     |               |          |              |     |       |         |         |     |
|     |                              |        |     |        |           |     |     | Several       | previous | publications |     | cover | general | fuzzing | of  |
Fuzzing is a popular research area that has received much browser engines for JavaScript or stand-alone JavaScript en-
attention in the past years. In the following, we briefly intro- gines, but so far, there has been no publication that focused
duce this area and discuss work that is closely related to ours. on vulnerabilities in JIT compilers. Consequently, there is
Given the enormous scope of this area, we cannot provide a a gap in JavaScript fuzzer coverage that we fill with our
comprehensive overview of all related work and thus focus approach. Note that previous JavaScript fuzzing work dealt
mainly on related work that improves JavaScript fuzzing. For with using intermediate representations for fuzzing JavaScript
an introduction and overview of the fuzzing research area, or semantically correct fuzzing of JavaScript, two properties
we refer the reader to surveys on fuzzing [34] and greybox that our fuzzer has (and needs) for its success in fuzzing JIT
2

compiler vulnerabilities. Hence we discuss in the following semantic correctness to achieve the intended test area. This
how our approach relates to previous work in this area. approach is similar to the challenges we faced in fuzzing
One of the first works on this topic was presented by the JIT-related code parts, which also requires high semantic
Holler et al., who proposed to use an abstract syntax tree and syntactic correctness. However, the targeted aspects of
(AST) as an intermediate representation [31]. During fuzzing, JavaScriptenginesarenotcomparabletoours,aswefocuson
the subnodes of the AST are taken and replaced by nodes software faults in JIT compilers.
from other programs (or evennewly generated code) and then To improve the semantic correctness of fuzzed inputs,
translated back into the actual fuzzed language. Note that this Dewey et al. studied how Constraint Logic Programming
processgeneratescodethatadherestothesyntaxofthefuzzed (CLP) [23] can be used in this area. The authors use CLP
language. However, no focus was placed on any particular to generate semantically valid code, which is a similar focus
bug class or semantic validity of the generated code. In our to ours, but they do not focus on JIT compiler vulnerabilities
work, we do not use the AST as an intermediate representa- giventhatthisclassofsoftwarefaultsisparticularlychalleng-
tion. Instead, we develop our own intermediate language that ing to handle.
represents a subset of JavaScript. Most importantly, it enables Wang et al. proposed Skyfire, a seed generation tool for
afocusonJITerrorsandthesemanticvalidityofthegenerated fuzzing that requires a corpus of inputs and a grammar [42].
code. Based on this input, Skyfire learns a probabilistic context-
Most recently, He et al. presented SoFi [29], a semantic- sensitive grammar and uses this grammar to generate seed
awarefuzzingapproach.Toensurethevalidityofthegenerated inputs. The authors show that their approach works well for
test cases, the authors propose to use a fine-grained program highly structured languages, such as XML. However, they
analysis to identify variables and derive the types of these onlyprovidepreliminaryresultsonJavaScriptfuzzing,leaving
variables for mutation. In addition, SoFi uses an automatic future work to extend their approach “to better support more
repair strategy to fix syntactic and semantic errors in invalid complex languages such as JavaScript and SQL.” [42].
test cases. Unfortunately, the full source code of SoFi is not Han et al. presented CodeAlchemist [28], a generative
publiclyavailable,andhencewewerenotabletodirectlycom- fuzzer for JavaScript. Park et al. [36] proposed DIE, a novel
pare our approach to SoFi. Furthermore, we have reservations method for exploiting hidden information, which they termed
that the bugs discovered by SoFi are indeed security-critical aspects, in input corpora. This method enables a fuzzer to
(see Section VII-B2 for details). generate more complex, and consequently more profound,
Saxena et al. developed an intermediate language to canon- test programs. They analyze the given input seed files and
icalizedifferentJavaScriptinstructions(e.g.,splittingastring) extract not only code snippets but also aspects of the code
into a single action to improve fuzzing [39]. In this way, snippets, such as structure and runtime types. The proposed
the small details of an implementation are abstracted into a fuzzing method then uses this information to generate new
higher-level, easier to handle representation. Their focus is code snippets containing the extracted aspects. Although the
ondetectingclient-sidevalidationvulnerabilitiesinJavaScript work features a type system similar to ours, the type infor-
applications,ratherthanthebrowser’sJavaScriptengineitself. mation is applied to the AST layer instead of an IR. While,
Consequently, their fuzzing was based on the inputs of a generallyspeaking,ASTsarecapableofrepresentinganyvalid
JavaScript program rather than the inputs of a JavaScript JavaScript program, this abstraction layer is not ideal for im-
engine(i.e.,JavaScriptcode)itself.Inasimilarspirit,Hodova´n plementingmutations.Analogoustocodetransformationsused
et al. created a graph-based representation of the JavaScript by modern compilers [32], [35], we apply our mutations on
engine API and used this graph to generate input data for an IR layer. This design decision enables the implementation
fuzzing [30]. However, they focus on the API provided by of semantically meaningful mutations that produce a high
JavaScript engines and do not generate code beyond it. Con- diversity of generated JavaScript programs, a crucial aspect
sequently, the resulting code focuses on semantic correctness for fuzzing.
but is unlikely to detect JIT compiler bugs. In an orthogonal approach, Aschermann et al. proposed
Montage, a fuzzer based on a neural network language Nautilus [14], a multi-language fuzzer that combines an
model (NNLM), was proposed by Lee et al. [33]. They input grammar with code coverage. Mutations are applied
transform an AST into AST subtrees that can be used directly at the AST layer, hence suffering from the aforementioned
to train an NNLM. Using Montage, the authors found 37 limitations. A more recent work on multi-language fuzzing
bugs, including 3 CVEs. Although they found a JIT-related called Polyglot [21] improves Nautilus by translating a seed
vulnerability, the overall approach is orthogonal to ours as corpustoalanguage-agnosticIR.Unfortunately,themutations
they use machine learning on an AST. In contrast, we use applicable to the IR are quite limited, e.g., not even basic
predefined mutations on an IR. Moreover, they do not target language features such as variable definitions are amenable to
JIT but perform extensive fuzzing of JavaScript engines in mutation. In contrast, our specialization allows us to include
general. highly specialized mutators and generators that specifically
Recently, Ta Dinh et al. presented Favocado [24], a fuzzer target code to trigger JIT routines. As a result, we find
specializingonfuzzingbindinglayersinJavaScriptcode.They significantly more security-critical software vulnerabilities in
report that fuzzing such bindings requires both syntactic and real-world JIT engines used by major web browsers.
3

However,duringexecution,usagepatternsbecomeapparent.
| 1 // addition |     | function | in  | C   |     |     |     |     |     |     |     |     |     |     |     |
| ------------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
2 int add(int a, int b) { E.g., let us assume that the add operation is only observed
| return | a   | + b; |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ------ | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
3 being called with integers. Based on this observation, specu-
}
4 lative optimization can be performed: The compiler compiles
| // addition |     | function | in  | javascript |     |     |     |     |     |     |     |     |     |     |     |
| ----------- | --- | -------- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
5
function add(a, b) { the JavaScript code specifically for the inferred type profile.
6
return a + b; Finally, type guards are added that represent the optimization
7
8 }
|           |                    |          |     |          |          |                 |         | type          | assumptions.  | The       | guards | check      | that         | the given | values     |
| --------- | ------------------ | -------- | --- | -------- | -------- | --------------- | ------- | ------------- | ------------- | --------- | ------ | ---------- | ------------ | --------- | ---------- |
|           |                    |          |     |          |          |                 |         | are really    | of the        | assumed   | type   | (in        | our example, |           | integers). |
| Figure    | 1: A simple        | addition |     | function | in C     | and JavaScript. |         |               |               |           |        |            |              |           |            |
|           |                    |          |     |          |          |                 |         | As long       | as the        | guard     | holds, | the code   | proceeds     |           | with the   |
|           |                    |          |     |          |          |                 |         | now optimized |               | function. | If the | guard      | fails,       | the code  | ’bails     |
|           |                    |          |     |          |          |                 |         | out’ and      | the execution |           | of the | JavaScript | code         | returns   | to the     |
| Regarding | JavaScript-related |          |     | security | research |                 | without | a             |               |           |        |            |              |           |            |
specific focus on fuzzing, we refer to two recent survey interpreter,whichexecutesthenon-optimized,slowerfunction.
|              |       |     |     |     |     |     |     | The resulting |     | abstract | assembler | code       | would | be            | similar to |
| ------------ | ----- | --- | --- | --- | --- | --- | --- | ------------- | --- | -------- | --------- | ---------- | ----- | ------------- | ---------- |
| papers [13], | [41]. |     |     |     |     |     |     |               |     |          |           |            |       |               |            |
|              |       |     |     |     |     |     |     | the compiled  | C   | code     | with the  | difference |       | of containing | the        |
III. JUSTINTIMECOMPILERVULNERABILITIES type guards and, as we are talking about integer addition, an
|            |                     |          |        |            |           |             |            | overflow | check.        |               |         |               |     |          |           |
| ---------- | ------------------- | -------- | ------ | ---------- | --------- | ----------- | ---------- | -------- | ------------- | ------------- | ------- | ------------- | --- | -------- | --------- |
| In this    | section,            | we first | give   | a brief    | overview  | of          | JIT compi- |          |               |               |         |               |     |          |           |
|            |                     |          |        |            |           |             |            | We       | can summarize |               | such an | optimization  |     | into the | following |
| lation for | JavaScript          | in       | modern | browsers,  | followed  |             | by a case  |          |               |               |         |               |     |          |           |
|            |                     |          |        |            |           |             |            | steps    | resulting     | in a compiled |         | and optimized |     | version  | of the    |
| study of   | a JIT vulnerability |          | to     | illustrate | technical | challenges. |            |          |               |               |         |               |     |          |           |
functionunderscrutiny:(1)collectusagepatterndata,(2)infer
typepatterns,(3)optimizethecodeforthosetypes,(4)deploy
| A. Just | in Time | Compilation |     |     |     |     |     |             |     |          |               |     |       |     |     |
| ------- | ------- | ----------- | --- | --- | --- | --- | --- | ----------- | --- | -------- | ------------- | --- | ----- | --- | --- |
|         |         |             |     |     |     |     |     | type guards | in  | front of | the optimized |     | code. |     |     |
Weuseasmall,intuitiveexampletogiveabriefintroduction The reason why an engine does not immediately JIT com-
to the current approach to designing and implementing an pile the JavaScript code of a web page is twofold: First, the
efficient JS JIT compiler. More specifically, we explain the profiler has to collect execution information for JIT optimiza-
| basic concept | of  | a mixed-mode |     | JIT | compiler | architecture, |     |         |       |         |                  |     |     |                |     |
| ------------- | --- | ------------ | --- | --- | -------- | ------------- | --- | ------- | ----- | ------- | ---------------- | --- | --- | -------------- | --- |
|               |     |              |     |     |          |               |     | tion to | work. | Second, | JIT optimization |     | is  | time-consuming |     |
i.e., an interpreter as a baseline, followed by a sequence of becauseoptimizingeveryaspectofagivenJavaScriptprogram
successively higher optimizing JIT compilers. For a more might consume more time than is ever conserved by gains in
| detailed | and comprehensive |     | explanation |     | of different |     | JIT com- | execution | speed. |     |     |     |     |     |     |
| -------- | ----------------- | --- | ----------- | --- | ------------ | --- | -------- | --------- | ------ | --- | --- | --- | --- | --- | --- |
pilation approaches, including template- and trace-based JIT To gather the required information before triggering JIT
| compilation, | we  | refer | the reader | to  | common | compiler | and |              |     |           |       |         |        |         |          |
| ------------ | --- | ----- | ---------- | --- | ------ | -------- | --- | ------------ | --- | --------- | ----- | ------- | ------ | ------- | -------- |
|              |     |       |            |     |        |          |     | compilation, | a   | profiler, | which | is part | of the | engine, | collects |
interpreter literature (e.g., [11], [15], [19], [20], [22], [27]). execution information of the executed code. After reaching
JavaScript engines used in browsers contain a parser, a an internally specified threshold, the engine schedules the
bytecode compiler, an interpreter, and usually a JIT compiler. code for JIT compilation. Later executions directly call the
When JavaScript code is first encountered, the parser of the optimizedcodeinsteadofexecutingthefunctionviabytecode
| engine constructs |     | the corresponding |     | AST, | which | is  | compiled |     |     |     |     |     |     |     |     |
| ----------------- | --- | ----------------- | --- | ---- | ----- | --- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
interpretation.
to engine specific bytecode by the bytecode compiler [11]. Consequently, a typical modern JIT compiler pipeline con-
This bytecode is consumed by an interpreter. sists of the steps visualized in Figure 2. (a) The engine
However, bytecode interpretation is slow due to the dis- translates the source code into an AST. (b) The engine
patching overhead as well as the numerous, often redundant, compilestheASTintobytecodeforacustomVMandexecutes
| type checks | being | performed |     | by each | bytecode |     | handler. | If            |       |     |             |          |     |           |        |
| ----------- | ----- | --------- | --- | ------- | -------- | --- | -------- | ------------- | ----- | --- | ----------- | -------- | --- | --------- | ------ |
|             |       |           |     |         |          |     |          | this bytecode | using | the | interpreter | a number |     | of times, | during |
code is executed frequently, it is desirable to optimize the which type information is collected. (c) The engine passes
execution by compiling the JavaScript code to machine code the bytecode to the JIT compiler, which translates it into
and optimizing it on the way. An intuitive example of the a compiler-specific IR. While the bytecode is designed for
challenges faced by a JavaScript JIT compiler in comparison executionbytheinterpreter,theJITIRisdesignedtofacilitate
| with a | classic ahead-of-time |     |     | compiler | (e.g., | clang) | can be |                    |     |     |         |         |                |     |         |
| ------ | --------------------- | --- | --- | -------- | ------ | ------ | ------ | ------------------ | --- | --- | ------- | ------- | -------------- | --- | ------- |
|        |                       |     |     |          |        |        |        | the implementation |     | of  | various | program | optimizations. |     | (d) The |
seen in Figure 1: The given C code can directly be compiled JIT compiler optimizes the IR and adds type guards, which
into assembler code, as all required information is present. essentially add type information to the IR. (e) Finally, the JIT
Contrary to C, JavaScript is dynamically typed and not all compiler lowers the IR to machine code, which is directly
required information for machine code generation is present. executed on the host CPU.
| Consequently, |      | a JIT        | compiler | cannot  |             | trivially | compile  |        |               |      |       |     |     |     |     |
| ------------- | ---- | ------------ | -------- | ------- | ----------- | --------- | -------- | ------ | ------------- | ---- | ----- | --- | --- | --- | --- |
|               |      |              |          |         |             |           |          | B. JIT | Vulnerability | Case | Study |     |     |     |     |
| JavaScript    | code | into machine |          | code if | performance |           | is a re- |        |               |      |       |     |     |     |     |
quirement. The execution must first confirm the types used CVE-2018-4233 was one of the first vulnerabilities we
and then proceed according to the type at hand. Those types discovered during the initial exploration of JIT compiler
can range from primitive integers to highly complex objects, vulnerabilities. The JIT compiler tries to merge multiple type
all exhibiting different behavior when used with the same guards and fails to recognize that the type of the checked
| functionality. |     |     |     |     |     |     |     | variable | can change | in-between. |     |     |     |     |     |
| -------------- | --- | --- | --- | --- | --- | --- | --- | -------- | ---------- | ----------- | --- | --- | --- | --- | --- |
4

f } f } u o a r n r d e c t d ( t ( u i i r i o , n = n i 0 a a + ; d 1 d + ) i ( ; a b ! , ; = b 1 ) 00 { 00;i++){ P R F 1 2 3 3 C H a e r 2 5 4 8 o a r g a n n a i m E S E S s d m s e > > > > t l e t a e t e s 0 0 0 0 n r e r i x x x x t r z e e e e T c e 0 0 0 0 p a c o 1 1 1 1 o b o u 0 f f f f o l u n a a a a l e n t d d d d t 5 5 5 5 ( ( 0 3 3 3 3 s s 3 7 7 8 8 i i e f 1 4 z z e e @ @ @ @ = = 0 1 3 6 0 0 ) ) / v C v C v C r / 0 h 1 h 3 h e e e e t c = c = c = c u h k k k r e L I L I v I n c o f o f 0 f k a I a I I v d n d n + n 3 i A t A t t f r e r e v e g g g g 1 g p u e u e e a m r m r r r e e O a n v n v v m t 0 t 1 er i 0 1 f s lo i w n H t a e p g p e e r ned R R j R R c i . E E z E E a n . X X X X l t . . . 0 . . l 3 . W W x W W l 7 r l c b m m 1 e m d o o 0 a p 3 v v q q 8 q q d r r 0 r r b b 2 d 1 x x e x 0 , , 4 , , [ r 4 0 0 r c x x i x < 3 1 p + 6 0 + 0 0 f 0 x 0 1 x 2 0 1 f 4 0 1 f > 0 0 f 0 c f 0 0 f 0 f ( f A 9 b ] ort)
(a) (b) (c) (d) (e)
Figure 2: High-level overview of the JIT compiler pipeline: (a) The engine gets the code and (b) translates it into an AST.
(c) The AST is translated to bytecode that is executed by the compiler. (d) The JIT compiler translates bytecode into a JIT
intermediate representation and optimizes it. (e) Finally, the JIT compiler lowers the IR to machine code.
Figure 3 shows a proof-of-concept to trigger this behavior.
1 function Constructor(a, v) {
2 a[0] = v; TheJITcompilerassumesthattheconstructorfunctionalways
3 } receives an array with doubles as the first argument. It guards
4 this assumption with a type check at the beginning of the
5 var trigger = false;
6 var arg = null; emitted machine code. However, the CreateThis operation
7 var handler = { is executed after the type check of the argument object
8 get(target, propname) { and invokes JavaScript through a Proxy callback when the
9 if(trigger) {
10 arg[0] = {}; prototype property of the constructor is retrieved. Changing
11 } thetypeoftheargumentoftheargumentarrayinthecallback
12 return target[propname]; then causes a type confusion when the constructor function
13 },
14 }; resumes and accesses the array. As a result of executing the
15 var EvilProxy = new Proxy(Constructor, handler); proof-of-concept code the double value 3.54484805889626e-
16 310, which is stored as 0x414141414141, is wrongly used as
17 for(var i = 0; i < 100000; i++) {
18 new EvilProxy([1.1, 2.2, 3.3], 13.37); a pointer, resulting in an attacker controlled crash due to an
19 } access violation when dereferencing the address.
20
21 trigger = true; IV. METHOD
22 arg = [1.1, 2.2, 3.3];
23 new EvilProxy(arg, 3.54484805889626e-310); Fuzzing for JIT compilation vulnerabilities is an area that
24 arg[0]; has not yet been explored in detail (see Section II) and re-
quires special considerations concerning semantic correctness
Figure 3: Proof-of-concept code to trigger CVE-2018-4233.
(see Section III). In this section, we describe our approach to
fillthecurrentgapinfuzzingJITcompilersforvulnerabilities.
Westartbydefiningasetofrequirementsthatwedeemneces-
1) Guard Redundancy Removal: The JIT code deploys
sary to successfully fuzz JIT compiler engines (Section IV-A)
guards (Section III-A) to ensure that all type assumptions
and then show how a fuzzer based on mutations of a custom
made during compilation indeed hold at runtime. Missing any
IR can satisfy them (Section IV-B).
violated assumption may have severe consequences, ranging
from crashes to exploitable vulnerabilities. Depending on the A. Requirements
code, however, guards can be redundant. The JIT compiler
1) SyntacticCorrectness: TheparsersofJavaScriptengines
canremovearedundantchecktofurtheroptimizethecode.To
are simple and easy to understand compared with the rest
ensurethatguardsareredundant,theJITcompileranalyzesthe
of the code base, and are not of interest to us. Additionally,
code between guards for potential side effects. This analysis
the parser does not influence the JIT compiler. Consequently,
can be faulty, as was the case for CVE-2018-4233. A call to
our fuzzing approach needs to target the components behind
function deemed side-effect free could cause a user-defined
the parser. Aiming at these components requires the syntactic
JavaScript callback to be invoked, which in turn could change
correctness of emitted programs. Since the parsing phase
the type of a variable.
rejects syntactically invalid examples, we ensure syntactically
2) The Concrete Vulnerability: The compiler assumed that correct programs.
the CreateThis operation, responsible for creating a new 2) Guided Fuzzing: JIT compilers are deeply embedded
object in a constructor, would not result in any side effects. within a JavaScript engine. The first element of the engine to
However, this assumption is violated by wrapping the con- getintocontactwiththecodeistheparser,thentheinterpreter,
structor in a Proxy. and only when the code is executed with the correct pattern
By being able to change the type of an argument object, in is the JIT compiler triggered. To reach that deep into the
this case from an array of floating-point numbers to an array engine, we require feedback to generate increasingly complex
of JavaScript values, it is possible to achieve a type confusion inputs stressing different features, eventually reaching the JIT
in the emitted machine code. compiler.
5

| 3) Semantic |     | Correctness: | As  | explained | in  | Section | III, JIT |              |     |      |     |     |     |     |
| ----------- | --- | ------------ | --- | --------- | --- | ------- | -------- | ------------ | --- | ---- | --- | --- | --- | --- |
|             |     |              |     |           |     |         |          | 1 // verbose |     | code |     |     |     |     |
compiler optimization is only triggered in cases of repeatedly 2 function foo(b) {
|               |          |             |     |             |            |       |            | const  | a   | = 42; |     |     |     |     |
| ------------- | -------- | ----------- | --- | ----------- | ---------- | ----- | ---------- | ------ | --- | ----- | --- | --- | --- | --- |
| and reliably  | executed | code.       | For | such        | executions | to    | occur, we  | 3      |     |       |     |     |     |     |
|               |          |             |     |             |            |       |            | const  | c   | = a + | b;  |     |     |     |
| need semantic |          | correctness | of  | the emitted |            | code. | Exceptions | 4      |     |       |     |     |     |     |
|               |          |             |     |             |            |       |            | return | c;  |       |     |     |     |     |
5
| prevent | the execution | of  | subsequent | code | and | JIT compilation |     | 6 } |     |     |     |     |     |     |
| ------- | ------------- | --- | ---------- | ---- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
altogether, because the engine does not execute the code 7 // concise code
|     |     |     |     |     |     |     |     | function | foo(b) |     | {   |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------ | --- | --- | --- | --- | --- |
8
often enough. Commonly, fuzzers work around this issue by return b + 42;
9
| wrapping | every | generated | statement |     | with a | try-catch | block. | }   |     |     |     |     |     |     |
| -------- | ----- | --------- | --------- | --- | ------ | --------- | ------ | --- | --- | --- | --- | --- | --- | --- |
10
| This approach |     | does succeed |     | in ensuring | execution |     | of sub- |     |     |     |     |     |     |     |
| ------------- | --- | ------------ | --- | ----------- | --------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
Figure4:Exampleofasourcecodesnippetshowingaverbose
| sequent | code | after the | execution | encountered |     | an  | exception. |     |     |     |     |     |     |     |
| ------- | ---- | --------- | --------- | ----------- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
Unfortunately, this substantially alters the program’s seman- andaconciseversionperformingthesamesemanticoperation.
| tics, as        | an additional |            | control   | flow is | introduced. |             | Therefore,  |       |         |     |     |     |     |     |
| --------------- | ------------- | ---------- | --------- | ------- | ----------- | ----------- | ----------- | ----- | ------- | --- | --- | --- | --- | --- |
| a JIT compiler  |               | treats the | generated | example |             | differently | than        |       |         |     |     |     |     |     |
| if no try-catch |               | statements | had       | been    | inserted.   | In          | fact, a JIT |       |         |     |     |     |     |     |
|                 |               |            |           |         |             |             |             | v0 <- | LoadInt | 0   |     |     |     |     |
1
compilercannotperformmanyoptimizationswhenthecontrol 2 v1 <- LoadInt 10
|               |     |                 |          |               |            |           |            | v2 <-    | LoadInt            | 1      |       |           |     |     |
| ------------- | --- | --------------- | -------- | ------------- | ---------- | --------- | ---------- | -------- | ------------------ | ------ | ----- | --------- | --- | --- |
| flow graph    | is  | fragmented,     | as       | with inserted |            | try-catch | blocks.    | 3        |                    |        |       |           |     |     |
|               |     |                 |          |               |            |           |            | 4 v3 <-  | Phi v0             |        |       |           |     |     |
| We confirmed  |     | this assumption |          | by adding     | try-catch  |           | constructs | BeginFor |                    |        |       |           |     |     |
|               |     |                 |          |               |            |           |            | 5        | v0,                | <, v1, | +, v2 | -> v4     |     |     |
|               |     |                 |          |               |            |           |            | v6       | <- BinaryOperation |        |       | v3, +, v4 |     |     |
| into programs |     | found during    | fuzzing, | which         | afterward  |           | stopped    | 6        |                    |        |       |           |     |     |
|               |     |                 |          |               |            |           |            | 7 Copy   | v3,                | v6     |       |           |     |     |
| triggering    | the | defect          | in most  | cases.        | Therefore, | it        | becomes    | EndFor   |                    |        |       |           |     |     |
8
|     |     |     |     |     |     |     |     | 9 v7 <- | LoadString | 'Result: |     | '   |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | ---------- | -------- | --- | --- | --- | --- |
clear that a central requirement for successful fuzzing of JIT v8 <- BinaryOperation v7, +, v3
10
compilers is the ability to produce semantically correct code 11 v9 <- LoadGlobal 'console'
| with a high | likelihood. |                 |     |               |     |      |         | 12 v10 <- | CallMethod | v9, | 'log', | [v8] |     |     |
| ----------- | ----------- | --------------- | --- | ------------- | --- | ---- | ------- | --------- | ---------- | --- | ------ | ---- | --- | --- |
| 4) Semantic |             | Code Mutations: |     | We determined |     | that | we want |           |            |     |        |      |     |     |
to use feedback and require semantic correctness for a suc- Figure 5: An example of an IR program with a highlighted
|         |         |            |     |           |           |     |          | slice of the | program. |     |     |     |     |     |
| ------- | ------- | ---------- | --- | --------- | --------- | --- | -------- | ------------ | -------- | --- | --- | --- | --- | --- |
| cessful | fuzzing | framework. | One | essential | component |     | is still |              |          |     |     |     |     |     |
missing:theunderlyingsemanticsofthecode.AJITcompiler
| deals only | with | the semantic |      | properties | of  | the code, | such     |                    |     |                |     |          |             |     |
| ---------- | ---- | ------------ | ---- | ---------- | --- | --------- | -------- | ------------------ | --- | -------------- | --- | -------- | ----------- | --- |
|            |      |              |      |            |     |           |          | B. An Intermediate |     | Representation |     | Designed | for Fuzzing |     |
| as control | and  | data flow.   | This | is due     | to  | the JIT   | compiler |                    |     |                |     |          |             |     |
commonlyoperatingonitsownIRofthebytecodewithoutany
|     |     |     |     |     |     |     |     | As explained |     | in the | previous | section, | our fuzzing approach |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | ------ | -------- | -------- | -------------------- | --- |
knowledgeoftheinitialASTandthussyntax.Consequently,it
|     |     |     |     |     |     |     |     | uses its | own IR. | Thus, | we  | centered | our fuzzer’s | design |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------- | ----- | --- | -------- | ------------ | ------ |
isdesirabletoperformmutationsonthatlevelandincorporate
|               |       |           |              |            |             |          |           | around the     | idea    | of mutating |             | code in a | custom intermediate   |     |
| ------------- | ----- | --------- | ------------ | ---------- | ----------- | -------- | --------- | -------------- | ------- | ----------- | ----------- | --------- | --------------------- | --- |
| the feedback  | given | from      | guided       | fuzzing.   |             |          |           |                |         |             |             |           |                       |     |
|               |       |           |              |            |             |          |           | representation | (IR),   | then        | translating | the       | IR code to JavaScript |     |
| The simplest  |       | way to    | use feedback |            | is by using | a        | mutation- |                |         |             |             |           |                       |     |
|               |       |           |              |            |             |          |           | for execution. |         | We designed |             | our IR to | our requirements      | as  |
| based fuzzing |       | approach. | With         | this       | process,    | a fuzzer | can       |                |         |             |             |           |                       |     |
|               |       |           |              |            |             |          |           | stated in      | Section | IV-A:       |             |           |                       |     |
| dynamically   | add   | samples   | to           | the corpus | that        | result   | in new    |                |         |             |             |           |                       |     |
IR Design:
coverage, and mutate them further in the future. In our IR, a program consists of a list of
Existing mutation-based interpreter fuzzers, such as Lang- instructions, each in turn consisting of an operation together
|              |     |           |          |        |       |       |           | with a list | of input | and | output | variables. | Figure 5 shows | an  |
| ------------ | --- | --------- | -------- | ------ | ----- | ----- | --------- | ----------- | -------- | --- | ------ | ---------- | -------------- | --- |
| Fuzz, mutate |     | syntactic | elements | of the | code, | using | represen- |             |          |     |        |            |                |     |
tationssuchastheAST[31].However,syntacticelementsare example program which computes the sum of the numbers
irrelevant to the component targeted by our approach, the JIT from zero to nine. Note that IR operations can be paramet-
|           |          |     |         |               |     |               |     | ric. Parameters |     | include | constants | in operations, | property | and |
| --------- | -------- | --- | ------- | ------------- | --- | ------------- | --- | --------------- | --- | ------- | --------- | -------------- | -------- | --- |
| compiler. | Further, | the | AST can | be ambiguous. |     | Consequently, |     |                 |     |         |           |                |          |     |
solely semantic mutations are more challenging to imple- method names, the operators of binary and unary operations,
|         |           |           |     |              |        |     |           | as well as | comparisons. |     |     |     |     |     |
| ------- | --------- | --------- | --- | ------------ | ------ | --- | --------- | ---------- | ------------ | --- | --- | --- | --- | --- |
| ment as | immediate | mutations |     | could result | merely | in  | syntactic |            |              |     |     |     |     |     |
changes to the program and not semantic ones. We implemented the control flow using special block in-
Figure 4 shows an example where two code snippets structions, for which at least a starting and an ending block
with different AST express the same computation. An AST- exist. Our IR uses static single-assignment (SSA) form [12],
based mutation could simply be the transformation between [37], meaning that any variable has exactly one assignment.
those two code snippets. To counter this issue, we opted SSA form facilitates the implementation of a define-use anal-
to use an intermediate representation that is close to the ysis that we employ later on. It also increases the reliability
representation used by the compiler. Mutations on such an oftypeinferenceandsimplifiescodegenerationbecause,e.g.,
IR avoid semantically meaningless mutations and increase output values will always be assigned to a new SSA variable.
fuzzing effectiveness. By performing a different set of mu- Reassignments of JavaScript variables are possible through a
Phi
tations on an IR, we can detect different defects faster. We operation that produces an output reassignable through a
note that an AST mutation could be restricted to counter Copy instruction. We give a complete list of implemented IR
meaningless mutations but would then effectively become an operations in Appendix C, together with a description of the
| IR on its | own. |     |     |     |     |     |     | JavaScript | language | features | that | they cover. |     |     |
| --------- | ---- | --- | --- | --- | --- | --- | --- | ---------- | -------- | -------- | ---- | ----------- | --- | --- |
6

Required Invariants: In addition, we require that the fol- variablesintheinsertedprogramtoavoidcollisionsofvariable
lowing five invariants must hold for every program in our IR, names, which is, however, trivially possible.
and thus for the JavaScript programs generated from it: The more complex version of the mutation only inserts a
• Allinputsarevariables:Allinputvaluestoaninstruction partofanexistingprogramintothesecondone.Themutation
must be variables. There are no immediate values or selects a random instruction and, recursively, all instructions
|        |              |     |      |         |                      |     |     | whose outputs |     | are also | used | as inputs. | We  | then copy | the |
| ------ | ------------ | --- | ---- | ------- | -------------------- | --- | --- | ------------- | --- | -------- | ---- | ---------- | --- | --------- | --- |
| nested | expressions. |     | This | enables | more straightforward |     |     |               |     |          |      |            |     |           |     |
reasoningaboutthedataflowofaprogramandfacilitates resulting slice into another program. Figure 5 shows an
mutations to it. example slice of the program. This slice could then simply
|             |     |         |        |      |           |          |     | be copied | into | a different | program | as  | it is self-contained. |     |     |
| ----------- | --- | ------- | ------ | ---- | --------- | -------- | --- | --------- | ---- | ----------- | ------- | --- | --------------------- | --- | --- |
| • Variables | are | defined | before | use: | To reduce | possible |     |           |      |             |         |     |                       |     |     |
semanticerrors,allvariablesmustbedefinedbeforebeing However,thismutationdoesnotalteranyexistingdataflow,
astheSSAvariablesofthetwoinputprogramsarenotmixed.
| used, | either | in the | current | block or | an enclosing | one. |     |     |     |     |     |     |     |     |     |
| ----- | ------ | ------ | ------- | -------- | ------------ | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
• No open semantic blocks: A block beginning must even- To merge the data flows of the two input programs, an input
tually be followed either by the corresponding closing mutation would need to happen afterwards.
instruction or by an intermediate block instruction, such 4) Generative Mutation: The generative mutation simply
as a BeginElse for which the same holds true. This is inserts newly generated code, which makes use of existing
necessary to guarantee syntactic correctness. values, at random positions into an existing program. For this
• Inputs of blocks defined outside: All inputs to block purpose,weimplementedseveralcodegeneratorfunctionsthat
instructions must be defined in an outer block, reflecting emit short code snippets. Overal, we implemented one simple
the variable definition rules of JavaScript code generator for every language feature of the IR as well
Usage of Phi: To preserve SSA semantics, the first input as a small number of special code generators to either trigger
•
| Copy |     |     |     |     |     |     | Phi |     |     |     |     |     |     |     |     |
| ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
to a instruction must be the output of a JIT compilation or stress historically error-prone features.
instruction. Note that this mutator also ultimately makes it possible to
commencefromaninitiallyemptycorpus,asthemutatorgen-
| LiftingtheIRtoJavaScript: |     |     |     | WeliftaprogramfromourIR |     |     |     |     |     |     |     |     |     |     |     |
| ------------------------- | --- | --- | --- | ----------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
to JavaScript by first translating each instruction in isolation. erates new code and programs increasing the code coverage,
|                     |     |           |             |        |          |             |        | which our      | fuzzer | then            | adds to | the corpus  | dynamically. |                 |     |
| ------------------- | --- | --------- | ----------- | ------ | -------- | ----------- | ------ | -------------- | ------ | --------------- | ------- | ----------- | ------------ | --------------- | --- |
| As a next step,     | we  | inline    | expressions | when   | possible | to          | create |                |        |                 |         |             |              |                 |     |
| more human-readable |     | code.     |             |        |          |             |        |                |        |                 |         |             |              |                 |     |
|                     |     |           |             |        |          |             |        | D. Achieving   | a      | High Likelihood |         | of Semantic |              | Correctness     |     |
| C. Mutating         | the | IR        |             |        |          |             |        |                |        |                 |         |             |              |                 |     |
|                     |     |           |             |        |          |             |        | The invariants |        | we impose       | on      | the IR—each |              | being preserved |     |
| We designed         | the | mutations | in          | such a | way that | they modify |        |                |        |                 |         |             |              |                 |     |
byeverymutation—avoidsometrivialsemanticerrors,suchas
the central aspects of a program expressed in our IR. In par- theuseofavariablebeforeitisdefined.Thoserestrictionsare,
| ticular, we | achieve | the following |     | four goals | by our | mutations: |     |                |     |                |     |           |          |             |     |
| ----------- | ------- | ------------- | --- | ---------- | ------ | ---------- | --- | -------------- | --- | -------------- | --- | --------- | -------- | ----------- | --- |
|             |         |               |     |            |        |            |     | by themselves, |     | not sufficient |     | to ensure | semantic | correctness |     |
• Mutation of the data flow between instructions (Input overthegeneratedcorpus.Weaddedthreeadditionalmeasures
| Mutation, | Generative |     | Mutations) |     |     |     |     |            |     |          |              |     |     |     |     |
| --------- | ---------- | --- | ---------- | --- | --- | --- | --- | ---------- | --- | -------- | ------------ | --- | --- | --- | --- |
|           |            |     |            |     |     |     |     | to improve | the | semantic | correctness. |     |     |     |     |
• Mutation of the computation performed by instructions 1) Allowing only a valid corpus: We achieve an additional
| (Operation | Mutation) |     |     |     |     |     |     |           |          |             |     |             |      |               |     |
| ---------- | --------- | --- | --- | --- | --- | --- | --- | --------- | -------- | ----------- | --- | ----------- | ---- | ------------- | --- |
|            |           |     |     |     |     |     |     | degree of | semantic | correctness |     | by ensuring | that | only semanti- |     |
• Mutation of the control flow of the program (Combine callyvalidsamplesareaddedtothecorpusatruntime.Inorder
Mutation, Generative Mutation) to achieve that, it is necessary to not only record coverage
| Combination |     | of aspects |     | from two | different | programs |     |             |        |       |            |     |          |         |     |
| ----------- | --- | ---------- | --- | -------- | --------- | -------- | --- | ----------- | ------ | ----- | ---------- | --- | -------- | ------- | --- |
| •           |     |            |     |          |           |          |     | information | during | every | execution, |     | but also | whether | the |
(Combine Mutation) program terminated abnormally due to an uncaught runtime
|                   |     |     |          |     |       |           |     | exception. | In all | supported | engines, |     | this is possible |     | through |
| ----------------- | --- | --- | -------- | --- | ----- | --------- | --- | ---------- | ------ | --------- | -------- | --- | ---------------- | --- | ------- |
| In the following, |     | we  | describe | how | these | goals can | be  |            |        |           |          |     |                  |     |         |
achieved via different kinds of mutations. the exit code, which will generally be zero if no uncaught
1) InputMutation: Theinputmutationisasimplemutation exception was raised, and nonzero otherwise.
to the data flow of a program. We replace one SSA input to 2) Only performing small changes: A pivotal insight to
an instruction with a different one. This causes the instruction further improve the likelihood of semantic correctness is that
to operate on another value at runtime, potentially yielding each mutation only has a small probability of turning a valid
different results. (inthesemanticsense)programintoaninvalidone.Thisisdue
2) OperationMutation: to the fact that each mutation is either inherently semantically
Theoperationmutationconsistsof
selectingarandomparameterizedinstructionandchangingone correct(combinationmutations)oronlyaffectstheprogramin
of its parameters. For example, we change constant values, a minor way (input mutation, operation mutation, generative
| cause the program |     | to use | different | methods | or  | properties, | or  | mutations). |     |     |     |     |     |     |     |
| ----------------- | --- | ------ | --------- | ------- | --- | ----------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- |
replace binary or unary operations. 3) Alightweighttypesystem: Ourfinalsteptoincreasethe
3) Combine Mutation: The combine mutation combines semantic correctness of the generative component is a custom
parts of different programs into a new one: In the simple typesystem,astypeerrorsareasignificantsourceofsemantic
versionofthemutation,weinsertaprograminfullatarandom errors. For this, we implemented a lightweight abstract type
position in the second program. This requires renaming of inference engine. The inference engine, which can statically
7

approximatetheruntimetypesofanSSAvariable.Thisinfor- Algorithm 1: Scheduling and mutating samples in
| mation      | is then | used to   | avoid | generating | trivially   |     | invalid     | code FUZZILLI. |     |     |     |     |     |     |
| ----------- | ------- | --------- | ----- | ---------- | ----------- | --- | ----------- | -------------- | --- | --- | --- | --- | --- | --- |
| constructs, | such    | as method |       | calls on   | non-objects |     | or function | Corpus←[];     |     |     |     |     |     |     |
1
callsonnon-callableobjects.Inordertonotlimitthediversity
2 Corpus.add(genSeedProgram());
| ofcodethatthefuzzercanproduce,othermutationsgenerally |     |     |     |     |     |     |     | while | True | do  |     |     |     |     |
| ----------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | ----- | ---- | --- | --- | --- | --- | --- |
3
| ignore type | information. |          |        |     |       |        |           |       |                          |     |     |     |     |     |
| ----------- | ------------ | -------- | ------ | --- | ----- | ------ | --------- | ----- | ------------------------ | --- | --- | --- | --- | --- |
|             |              |          |        |     |       |        |           | 4 P   | ←Corpus.randomElement(); |     |     |     |     |     |
| We designed |              | the type | system | to  | be as | simple | as possi- | 5 for | N do                     |     |     |     |     |     |
ble, yet powerful enough to enable inference of the pos- P ←mutate(P);
|                  |     |      |        |           |     |        |       | 6   | m                   |     |     |       |     |     |
| ---------------- | --- | ---- | ------ | --------- | --- | ------ | ----- | --- | ------------------- | --- | --- | ----- | --- | --- |
| sible operations |     | that | can be | performed |     | on the | value | at  |                     |     |     |       |     |     |
|                  |     |      |        |           |     |        |       | 7   | exec←execute(lift(P |     |     | m )); |     |     |
runtime. The basic types supported are T integer , T float , if exec.returnStatus==crash then
8
| T ,T |     | ,T  |     |     | ,T  |     |     | ,   | saveToDisk(P |     |     | );  |     |     |
| ---- | --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- |
string boolean function(signature) constructor(signature) 9 m
T object([properties],[methods]) , T undefined , and T unknown 10 else if exec.returnStatus==normal then
| These             | types | can be | combined | using  | the | union    | operator, |     | P   | ←P ;              |     |      |     |     |
| ----------------- | ----- | ------ | -------- | ------ | --- | -------- | --------- | --- | --- | ----------------- | --- | ---- | --- | --- |
|                   |       |        |          |        |     |          |           | 11  |     | m                 |     |      |     |     |
| t1|t2, expressing |       | that   | a value  | is one | of  | multiple | types.    | For |     |                   |     |      |     |     |
|                   |       |        |          |        |     |          |           | 12  | if  | newCoverage(exec) |     | then |     |     |
example, the outcome of the addition operator in JavaScript P ←minimize(P);
|     |     |     |     |     |     |     |     | 13  |     | min |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
can generally be a number or a string and would as such be Corpus.add(P );
|             |         |                |          |             |           |            |            | 14            |        |              |             | min        |            |             |
| ----------- | ------- | -------------- | -------- | ----------- | --------- | ---------- | ---------- | ------------- | ------ | ------------ | ----------- | ---------- | ---------- | ----------- |
|             |         | T              | |T       | |T          |           |            |            |               |        |              |             |            |            |             |
| represented | as      | integer        | float    | string      | .         |            |            |               |        |              |             |            |            |             |
| Further,    | two     | types can      | also     | be combined |           | using      | the merge  |               |        |              |             |            |            |             |
| operator,   | t1+t2.  | Such           | a merged | type        | expresses |            | that a     | value         |        |              |             |            |            |             |
| is two or   | more    | types          | at the   | same time.  | An        | example    | for        | this          |        |              |             |            |            |             |
|             |         |                |          |             |           |            |            | P before      | adding | it to        | the corpus. | Otherwise, |            | the size of |
| would be    | strings | in JavaScript, |          | as they     | expose    | properties |            | and m         |        |              |             |            |            |             |
|             |         |                |          |             |           |            |            | the programs  | in     | the corpus   | would       | keep       | increasing | and slow    |
| methods     | to the  | user. The      | type     | system      | thus      | represents |            | them          |        |              |             |            |            |             |
|             |         |                |          |             |           |            |            | down fuzzing. |        | Minimization | is          | naively    | possible   | through a   |
| as T        | +T      | . Finally,     |          | the type    | system    | can        | also model |               |        |              |             |            |            |             |
| string      | object  |                |          |             |           |            |            |               |        |              |             |            |            |             |
the list of properties and methods of an object as well as the fixpointiterationthatsuccessivelyattemptstoremoveinstruc-
|              |              |                |                   |        |     |        |       | tions while | ensuring | that      | the resulting |     | program  | still exhibits |
| ------------ | ------------ | -------------- | ----------------- | ------ | --- | ------ | ----- | ----------- | -------- | --------- | ------------- | --- | -------- | -------------- |
| signatures   | of functions |                | and constructors. |        |     |        |       |             |          |           |               |     |          |                |
|              |              |                |                   |        |     |        |       | the same    | coverage | increase. | As            | the | distance | between two    |
| The abstract |              | type inference |                   | engine | has | simple | rules | that        |          |           |               |     |          |                |
determine the output types of every operation and operates interesting programs is often larger than a single mutation
canbridge,wemutateaprogrammultipletimesconsecutively.
| on a static | model | of the | runtime | environment, |     | containing |     | type |     |     |     |     |     |     |
| ----------- | ----- | ------ | ------- | ------------ | --- | ---------- | --- | ---- | --- | --- | --- | --- | --- | --- |
information for every built-in object. Whenever two or more However, to prevent the unnecessary investment of resources,
thelastmutationisrevertedifitproducedaninvalidprogram.
| alternative | control-flow |     | paths | merge, | we combine |     | the variable |            |     |                |     |         |           |          |
| ----------- | ------------ | --- | ----- | ------ | ---------- | --- | ------------ | ---------- | --- | -------------- | --- | ------- | --------- | -------- |
|             |              |     |       |        |            |     |              | Pseudocode | for | the high-level |     | fuzzing | algorithm | is given |
states,usingtheunionoperator.Theexecutionsemanticsdiffer
| slightly | between | JavaScript | and | our | inference | engine, | e.g., | the in Algorithm | 1.  |     |     |     |     |     |
| -------- | ------- | ---------- | --- | --- | --------- | ------- | ----- | ---------------- | --- | --- | --- | --- | --- | --- |
inferenceenginedoesnothaveaconceptofprototypesasthey V. EXPERIMENT
existinJavaScript.Whilesimplifyingtheimplementation,this
|               |        |         |                |                   |        |      |             | We implemented |              | the            | fuzzer design    | outlined       |            | in the previous |
| ------------- | ------ | ------- | -------------- | ----------------- | ------ | ---- | ----------- | -------------- | ------------ | -------------- | ---------------- | -------------- | ---------- | --------------- |
| leads to      | errors | in the  | static type    | approximation.    |        |      | However,    | in             |              |                |                  |                |            |                 |
|               |        |         |                |                   |        |      |             | section        | in the Swift | programming    |                  | language       | in         | a tool called   |
| practice,     | these  | turned  | out to         | be unproblematic, |        |      | as the      | type           |              |                |                  |                |            |                 |
|               |        |         |                |                   |        |      |             | Fuzzilli.      | We used      | this prototype |                  | implementation |            | for the eval-   |
| approximation |        | is used | conservatively |                   | by the | code | generators. |                |              |                |                  |                |            |                 |
|               |        |         |                |                   |        |      |             | uation and     | ran          | it against     | the instrumented |                | JavaScript | engine          |
Asthestatictypeinferencesystemismerelyaperformance
|               |               |        |             |           |           |          |      | code of               | three state-of-the-art |     | JavaScript |               | engines: | Google V8, |
| ------------- | ------------- | ------ | ----------- | --------- | --------- | -------- | ---- | --------------------- | ---------------------- | --- | ---------- | ------------- | -------- | ---------- |
| optimization, | it            | can be | disabled    | entirely, |           | in which | case | the                   |                        |     |            |               |          |            |
|               |               |        |             |           |           |          |      | Apple JavaScriptCore, |                        | and | Mozilla    | SpiderMonkey. |          |            |
| types of      | all variables |        | will become |           | T unknown | and      | code | gen-                  |                        |     |            |               |          |            |
erators will generate truly random operations. In practice, we A. Fuzzing Time Frame
| found that | the correctness |     | rate | varied | between | 50% | and | 75%. |     |     |     |     |     |     |
| ---------- | --------------- | --- | ---- | ------ | ------- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
Thevulnerabilitiesreportedinthissectionaretheresultsof
|            |     |                 |     |      |                |     |     | a consecutive | series | of fuzzing |     | sessions | spanning | six months. |
| ---------- | --- | --------------- | --- | ---- | -------------- | --- | --- | ------------- | ------ | ---------- | --- | -------- | -------- | ----------- |
| E. Fuzzing | on  | an Intermediate |     | Code | Representation |     |     |               |        |            |     |          |          |             |
Eachsessionlastedforaroundoneweekandusedaround500
Ourfuzzingapproachgenerallyfollowsthestandarddesign
|     |     |     |     |     |     |     |     | CPU cores. | For | each session, |     | either the | most | recent source |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | ------------- | --- | ---------- | ---- | ------------- |
ofamutation-basedfuzzer.Ineveryiteration,thefuzzerselects codeversionatthatpointor,ifavailable,thesourcecodeofthe
| a program | P from | the | existing | corpus | (seeded | with | a single- |              |         |     |       |            |         |           |
| --------- | ------ | --- | -------- | ------ | ------- | ---- | --------- | ------------ | ------- | --- | ----- | ---------- | ------- | --------- |
|           |        |     |          |        |         |      |           | current beta | release | was | used. | We ran the | fuzzing | on Google |
lineJavaScriptprogram)andmutatesitrandomlytoproducea ComputeEngine(GCE)andpredominantlyusedmultipleN1-
| newprogramP |              | .ThefuzzerthenliftsP |          |     |            | toJavaScriptcode, |        |                                                 |     |     |     |     |     |     |
| ----------- | ------------ | -------------------- | -------- | --- | ---------- | ----------------- | ------ | ----------------------------------------------- | --- | --- | --- | --- | --- | --- |
|             |              | m                    |          |     | m          |                   |        | standard-4machinetypemachines(4CPUs,15GBofRAM). |     |     |     |     |     |     |
| which is    | subsequently |                      | executed | on  | the target |                   | engine | while                                           |     |     |     |     |     |     |
Further,wechosetousepreemptibleinstancestodecreasethe
| gathering | coverage | statistics, |     | e.g., through |     | Clang’s | sanitizer- | costs. |     |     |     |     |     |     |
| --------- | -------- | ----------- | --- | ------------- | --- | ------- | ---------- | ------ | --- | --- | --- | --- | --- | --- |
coverage feature.
B. Setup
| If execution |     | of P m | increases | the | coverage |     | of the target |     |     |     |     |     |     |     |
| ------------ | --- | ------ | --------- | --- | -------- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- |
program, P is regarded as interesting and kept for future We compiled the target engines as standalone binaries,
m
mutations. However, as our mutations can only increase a without the web browser bindings. Additionally, we modi-
programinsizebutnevershrinkit,itisnecessarytominimize fied the engines to support the target interface that requires
8

TableI:SummaryofthefoundsecurityvulnerabilitiesforGoogleV8,AppleJavaScriptCore(JSC),andMozillaSpiderMonkey
(SM). The table also presents more information about each vulnerability along two taxonomies (effect of the vulnerability and
the time a vulnerability may occur). Finally, we list the age of the vulnerability in months.
| Engine | Issue |     | Type |     | Runtime |     | Age | Description |     |     |     |     |     |     |     |
| ------ | ----- | --- | ---- | --- | ------- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- |
SM CVE-2019-9813 Type Safety ✓ 18 Incorrect update to inferred property types
SM CVE-2019-9791 Type Safety ✓ >36 Incorrect type inference for constructors entered via OSR
SM CVE-2019-11707 Type Safety ✓ 20 Incorrect return type inference of Array.prototype.pop
V8 Issue 944062 Type Safety ✓ 1 ReduceArrayIndexOfIncludes fails to add Map checks
V8 Issue 944865 Type Safety ✓ 8 Invalid value representation in V8
JSC CVE-2019-8671 Type Safety ✓ 9 LICM leaves object property access unguarded
JSC CVE-2019-8765 Type Safety ✘ > 6 GetterSetter type confusion during DFG compilation
V8 Issue 939316 Spatial Safety ✘ 4 Optimizing Reflect.construct causes Map pointer OOB
JSC CVE-2019-8518 Spatial Safety ✓ 9 LICM moves array access before bounds check causing OOB
JSC CVE-2019-8622 Temporal Safety ✓ >36 doesGC() incorrectly models behavior of StringObjects
JSC CVE-2019-8672 Temporal Safety ✓ 30 JSValue use-after-free in ValueProfiles
JSC CVE-2019-8558 Temporal Safety ✘ >30 CodeBlock use-after-free due to dangling Watchpoints
JSC CVE-2019-8623 Uninit Data ✓ 24 LICM leaves stack variable uninitialized
JSC CVE-2019-8611 Uninit Data ✓ 4 Optimization incorrectly removes assignment to register
SM CVE-2019-9792 Misc ✓ 4 Leaks JS_OPTIMIZED_OUT magic value to script
SM CVE-2019-9816 Misc ✓ >36 Unexpected ObjectGroup in ObjectGroupDispatch operation
V8 Issue 958717 Misc ✓ 4 Incorrect interaction between DCE and inlining
communication between our fuzzer and the JavaScript engine or crash condition, others first required substantial analysis
over a set of communication pipes. Further, we lowered the to determine exploitability. All identified bugs were reported
JIT compilation thresholds to trigger JIT compilation earlier, to the developers in a coordinated way. We consider all
thusspeedingupthefuzzing.Ingeneral,wesetthethresholds defects listed to be exploitable and either received a CVE or
such that roughly 100 executions of a function would cause Chrome Internal Issue number. Consequently, the issues were
it to be compiled. This threshold allows a sufficient number subsequently fixed by the developers.
of iterations for the engine to collect type information, while 1) Type Classification: Table I shows a comprehensive
speedingupthefuzzing.Modifyingthethresholdisacommon summary of all 17 vulnerabilities found which resulted in
technique deployed by previous fuzzing solutions [3]. Finally, a CVE or internal issue number being assigned to us. All
wecompiledtheenginesinacustomdebugconfigurationwith
|     |     |     |     |     |     |     |     | the identified | vulnerabilities |     |     | were | related | to JIT compilation |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --------------- | --- | --- | ---- | ------- | ------------------ | --- |
optimizations enabled for performance reasons. The debug and span across issues such as invalid bounds check removal,
configuration includes a number of internal assertions, which incorrect type inference, and register misallocation issues.
| are removed | in release | builds | for | performance | reasons. |     |     |        |                |     |     |      |            |         |        |
| ----------- | ---------- | ------ | --- | ----------- | -------- | --- | --- | ------ | -------------- | --- | --- | ---- | ---------- | ------- | ------ |
|             |            |        |     |             |          |     |     | 2) Age | Determination: |     | We  | also | determined | the age | of the |
Weenableassertions,astheyhelpwithdetectingexploitable vulnerability by compiling old versions of the software and
| defects that | do not | immediately |     | materialize | as memory |     | safety |           |         |     |       |      |               |          |     |
| ------------ | ------ | ----------- | --- | ----------- | --------- | --- | ------ | --------- | ------- | --- | ----- | ---- | ------------- | -------- | --- |
|              |        |             |     |             |           |     |        | verifying | whether | the | found | test | case triggers | a crash. | For |
violations. As an example, CVE-2019-8622 was discovered JavaScriptCore and V8, we used git’s bisecting feature to find
| through | an assertion | failure. | The | JIT compiler | assumed |     | that a |            |           |     |          |     |      |              |          |
| ------- | ------------ | -------- | --- | ------------ | ------- | --- | ------ | ---------- | --------- | --- | -------- | --- | ---- | ------------ | -------- |
|         |              |          |     |              |         |     |        | the oldest | revision. | For | Firefox, | we  | used | old official | releases |
specificoperationcouldnevercauseagarbagecollection(GC)
|     |     |     |     |     |     |     |     | and tested | on those. |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | --------- | --- | --- | --- | --- | --- | --- |
to happen. However, during execution, the lowered operation As the test case might not trigger on a different version
| did invoke | APIs | that can, | under | some | circumstances, | trigger |     |               |     |          |          |     |              |         |      |
| ---------- | ---- | --------- | ----- | ---- | -------------- | ------- | --- | ------------- | --- | -------- | -------- | --- | ------------ | ------- | ---- |
|            |      |           |       |      |                |         |     | for unrelated |     | reasons, | or might |     | even trigger | another | bug, |
GC. This situation can then be exploited by first triggering a the age results could contain inaccuracies, but are generally
GCataspecificallychosentimeandthendeliberatelycrafting
|            |      |         |       |       |          |              |     | conservative. | In          | cases | where | we could  | not    | determine   | the age |
| ---------- | ---- | ------- | ----- | ----- | -------- | ------------ | --- | ------------- | ----------- | ----- | ----- | --------- | ------ | ----------- | ------- |
| JavaScript | code | so that | a now | freed | JSObject | is afterward |     |               |             |       |       |           |        |             |         |
|            |      |         |       |       |          |              |     | of an issue   | dynamically |       | as    | described | above, | we resorted | to      |
accessed in the JIT compiled code. manual source code analysis to try to determine when the
| Both of | these | steps are | necessary | to  | cause memory |     | safety |            |      |     |             |     |      |               |       |
| ------- | ----- | --------- | --------- | --- | ------------ | --- | ------ | ---------- | ---- | --- | ----------- | --- | ---- | ------------- | ----- |
|         |       |           |           |     |              |     |        | vulnerable | code | was | introduced. | As  | this | is more error | prone |
violation.Asbothstepsrequireanon-trivialamountofspecif- than compiling and testing the code, the results here are less
| ically crafted | code, | they | are unlikely |     | to be directly |     | found |     |     |     |     |     |     |     |     |
| -------------- | ----- | ---- | ------------ | --- | -------------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
certain.
| through  | fuzzing,    | but the | indicators | for | such a | violation | are |                                             |     |     |     |     |     |     |     |
| -------- | ----------- | ------- | ---------- | --- | ------ | --------- | --- | ------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
| detected | by fuzzing. |         |            |     |        |           |     |                                             |     |     |     |     |     |     |     |
|          |             |         |            |     |        |           |     | VI. CLASSIFICATIONOFTHEFOUNDVULNERABILITIES |     |     |     |     |     |     |     |
C. Results
|     |     |     |     |     |     |     |     | To systematize |     | the | found | vulnerabilities, |     | we first | describe |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --- | ----- | ---------------- | --- | -------- | -------- |
Many of the vulnerabilities found first materialized as therootcausesthatleadtoeachvulnerability,followedbytwo
failed assertions or null pointer dereferences. Afterward, we taxonomies.Thefirstclassifiesthevulnerabilitiesaccordingto
performed manual triaging of the crashes to determine if they the effect and the second by the time the vulnerability occurs.
were security-critical and exploitable. While some observed A tabular overview of classified vulnerabilities is shown in
| crashes | were clearly | exploitable |     | given | the failed | assertion |     | Table I. |     |     |     |     |     |     |     |
| ------- | ------------ | ----------- | --- | ----- | ---------- | --------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
9

| A. Root | Causes |     |     |     |     |     |     | VII. | EFFICACYOFOURMETHOD |     |     |     |     |     |
| ------- | ------ | --- | --- | --- | --- | --- | --- | ---- | ------------------- | --- | --- | --- | --- | --- |
We identified three common root causes: Optimizations Duetothenon-deterministicnatureoffuzzing,theobjective
that move code, incorrect modeling of runtime execution properties of different fuzzing methods are difficult to com-
semantics, and faulty deletion of type checks. Root causes pare.Anotherobstacletoperformingameaningfulcomparison
|             |                     |     |             |     |                |     | is the different |     | goals | and design | principles |     | of fuzzers. | We  |
| ----------- | ------------------- | --- | ----------- | --- | -------------- | --- | ---------------- | --- | ----- | ---------- | ---------- | --- | ----------- | --- |
| not fitting | this categorization |     | are grouped | as  | miscellaneous. |     |                  |     |       |            |            |     |             |     |
1) CodeMotion: Acommonoptimizationconsistsofmov- designed our approach called Fuzzilli specifically for finding
ing code fragments in the program (e.g., loop-invariant code JIT vulnerabilities and specialized accordingly. Moreover, our
motion). However, if this is done incorrectly, previously safe methoddoesnotrequireaninputcorpusandcangeneratenew
code fragments become unsafe (3 vulnerabilities). JavaScript code on its own. Other JavaScript fuzzers, such as
|              |           |        |     |            |        |      | Superion | [43] | or SoFi | [29], | have a more | general | design | goal |
| ------------ | --------- | ------ | --- | ---------- | ------ | ---- | -------- | ---- | ------- | ----- | ----------- | ------- | ------ | ---- |
| 2) Incorrect | Modeling: | Issues | may | arise from | faulty | mod- |          |      |         |       |             |         |        |      |
eling of runtime execution semantics, such as whether an and require an input corpus. Consequently, the context of the
operation has side effects or could trigger garbage collection design must be considered when comparing them.
(2 vulnerabilities). To evaluate the efficacy of our method, we perform both a
3) Incorrect Type Inference: A central optimization of JIT descriptiveandanempiricalanalysis.Webeginourdescriptive
|           |                  |     |         |                   |     |       | evaluation | by  | analyzing | both | the objective |     | generality | and |
| --------- | ---------------- | --- | ------- | ----------------- | --- | ----- | ---------- | --- | --------- | ---- | ------------- | --- | ---------- | --- |
| compilers | is the inference | of  | runtime | type information, |     | which |            |     |           |      |               |     |            |     |
allows type checks to be omitted. Whenever an incompatible quality of our approach. Then we present an empirical study
value is stored in a property with associated type information, inwhichweinvestigatetheimpactofourdifferentgenerators,
thattypeinformationhastobeupdatedbecauseJITcompilers measure the code coverage of our method, and compare it to
rely on it to omit runtime type checks (4 vulnerabilities). Superion [43]. To allow future replication, we make our code
|            |                |                       |            |                 |            |           | and artifacts  | openly | available    |            | online.    |     |          |      |
| ---------- | -------------- | --------------------- | ---------- | --------------- | ---------- | --------- | -------------- | ------ | ------------ | ---------- | ---------- | --- | -------- | ---- |
| 4) Misc:   | Not all        | found vulnerabilities |            | shared          | a          | common    |                |        |              |            |            |     |          |      |
| underlying | issue. This    | could                 | be because | they            | are        | “one-off” |                |        |              |            |            |     |          |      |
|            |                |                       |            |                 |            |           | A. Descriptive |        | Efficacy     | Evaluation |            |     |          |      |
| bugs or    | simply because | no other              | similar    | vulnerabilities |            | were      |                |        |              |            |            |     |          |      |
|            |                |                       |            |                 |            |           | During         | our    | experiments, | we         | discovered | and | reported | mul- |
| found with | which they     | could                 | have       | formed          | a category | (9        |                |        |              |            |            |     |          |      |
vulnerabilities). tiple previously unknown vulnerabilities in all three major
|     |     |     |     |     |     |     | JavaScript | engines | and | were | assigned | corresponding |     | CVE or |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ------- | --- | ---- | -------- | ------------- | --- | ------ |
Furthermore, the discovered vulnerabilities can be differen- Issue numbers (see Table I). These findings show that our
| tiated according | to their | effect | and time | of impact. |     | Next, we |          |          |            |     |          |                   |     |          |
| ---------------- | -------- | ------ | -------- | ---------- | --- | -------- | -------- | -------- | ---------- | --- | -------- | ----------------- | --- | -------- |
|                  |          |        |          |            |     |          | approach | achieves | generality |     | in terms | of applicability, |     | i.e., we |
briefly explain these two categories. didnotoptimizeforaspecificengineorbenchmarkandhada
positivesecurityimpactondifferenthighlyrelevantJavaScript
| B. Classification | by  | Effect |     |     |     |     |     |     |     |     |     |     |     |     |
| ----------------- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
engines.Furthermore,allthreeengineshavebeencontinuously
We determined that there were four common clusters (and tested by vendor-specific fuzzing infrastructures and by third-
| a misc | cluster) of | different | types | of effects | most | found |     |     |     |     |     |     |     |     |
| ------ | ----------- | --------- | ----- | ---------- | ---- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
partyfuzzers.Duetothesepriorefforts,JavaScriptenginesare
| vulnerabilities | could | have: |     |     |     |     |           |            |     |             |           |               |     |     |
| --------------- | ----- | ----- | --- | --- | --- | --- | --------- | ---------- | --- | ----------- | --------- | ------------- | --- | --- |
|                 |       |       |     |     |     |     | generally | considered |     | well-tested | software. | Nevertheless, |     | we  |
1) Type Safety Violations: All vulnerabilities that cause found 17 vulnerabilities that eluded competing fuzzers, with
| some kind | of type confusion |     | (7 vulnerabilities). |     |     |     |         |                     |     |            |     |      |            |       |
| --------- | ----------------- | --- | -------------------- | --- | --- | --- | ------- | ------------------- | --- | ---------- | --- | ---- | ---------- | ----- |
|           |                   |     |                      |     |     |     | some of | the vulnerabilities |     | introduced |     | more | than three | years |
2) Spatial Memory Safety Violations: All vulnerabilities ago. This shows that our fuzzer is a significant qualitative
| that cause  | spatial memory    | corruption, |             | such | as out-of-bounds  |     |              |          |            |            |          |     |     |     |
| ----------- | ----------------- | ----------- | ----------- | ---- | ----------------- | --- | ------------ | -------- | ---------- | ---------- | -------- | --- | --- | --- |
|             |                   |             |             |      |                   |     | improvement  | for      | JavaScript | JIT        | fuzzing. |     |     |     |
| accesses    | to heap allocated | memory      | blocks      | (2   | vulnerabilities). |     |              |          |            |            |          |     |     |     |
|             |                   |             |             |      |                   |     | B. Empirical | Efficacy |            | Evaluation |          |     |     |     |
| 3) Temporal | Memory            | Safety      | Violations: | All  | vulnerabilities   |     |              |          |            |            |          |     |     |     |
that cause temporal memory safety violations, e.g., due to Our empirical efficacy evaluation is two-fold. First, we
usage of previously freed memory (3 vulnerabilities). analyze what effect each of our different generators had and
4) UsageofUninitializedData: Allvulnerabilitiesthatuse how many mutations we are able to achieve across time for
uninitialized data, such as reading a pointer value from an the three major JavaScript engines.
uninitialized location in the stack (2 vulnerabilities). Second, to show that our fuzzer is competitive in the
5) Misc: All vulnerabilities that do not fit into any of the contextofoverallJavaScriptfuzzing,weconductanempirical
preceding categories (3 vulnerabilities). evaluation against Superion [43]. This fuzzer represents the
|                   |                   |         |                 |     |     |        | current state | of   | the art        | in JavaScript |       | fuzzing    | and the | authors |
| ----------------- | ----------------- | ------- | --------------- | --- | --- | ------ | ------------- | ---- | -------------- | ------------- | ----- | ---------- | ------- | ------- |
| C. Classification | by                | Time of | Impact          |     |     |        |               |      |                |               |       |            |         |         |
|                   |                   |         |                 |     |     |        | have shown    | that | it outperforms |               | other | approaches | in      | several |
| There             | are two different | times   | a vulnerability |     | may | occur, | dimensions.   |      |                |               |       |            |         |         |
either during runtime or during compile time. 1) Generator Effect Analysis: We ran our fuzzer five times
1) Runtime: All vulnerabilities in this category are logical for 24 hours against the three JavaScript engines SpiderMon-
compilerflaws,possiblyleadingtomemorycorruptiontrigger- key, JavaScriptCore, and V8. During these runs, we logged
ingintheemittedmachinecodeatruntime(14vulnerabilities). the mutations resulting in code coverage increases. Figure 6
2) CompileTime: Thiscategoryincludes“classic”memory shows the results for JavaScriptCore and SpiderMonkey, the
corruptionbugs aswell ascompiler-specificones, which,lead plot for V8 is shown in Appendix A due to page restric-
to memory corruption during compilation (3 vulnerabilities). tions. We can observe that generative and input mutations
10

Figure 6: Temporal analysis of the proportion of our different mutation strategies for JSC and SpiderMonkey, respectively.
Measured at 10 minute intervals.
were the most significant contributors when generating new age branch coverage as our metric. As our fuzzer specializes
samples. Combine and operation mutations are the next two in JIT fuzzing, we also compare the JIT-specific coverage.
influential mutations. Interestingly, explicitly stressing the JIT As noted above, our targeted engines are JavaScriptCore, V8,
was the smallest contributor during our empirical analysis. and SpiderMonkey, given that they are used in modern web
These results show that when fuzzing for JIT vulnerabilities, browsers. The exact command line flags for each fuzzer JS
there is no need to focus entirely on JIT-related mutations. A engine are shown in Appendix B.
relatively small but constant effort suffices in practice. An in- Setup: We ran each fuzzer five times for 24 hours on 100
depth explanation of the different mutation strategies can be cores of a Xeon Gold 5320 CPU with 256GB RAM using
found in Section IV-C. Ubuntu 22.04. Fuzzilli and Superion instances were deployed
There is also no noticeable difference across the different in virtual machines with 2GB of RAM for each of the 100
engines. The distribution of the successful mutators stays instances. For both fuzzers, corpus sharing was enabled.
within the same proportions. This observation also holds for Used Corpus: Our fuzzer does not use an input corpus,
thenumberofmutationperminutethatresultinnovelsamples, whereas Superion does require a corpus. This is a caveat
whichconvergetowardszeroforbothJSCandSpiderMonkey. when trying to perform a fair and objective comparison, as
However,wefindthattheinitialdeclineisslowerforJSCthan the corpus might determine the quality and final coverage
for SpiderMonkey. of the fuzzing results and progress. A further hindrance is
2) Comparison with SoFi: We would have preferred to that Superion does not publish its corpus. We opted to use
include a direct comparison with the recently published the publicly available DIE corpus [2] as input for Superion.
SoFi [29] approach. Unfortunately, the authors neither pub- To measure the impact of the start corpus on the success
lished the full source code nor provided the code when con- of Superion, we additionally evaluated on randomly chosen
tacted.WeanalyzedtheresultsreportedinTable2ofthepaper sub-corpora of DIE. We generated these sub-corpora by suc-
and found that the discovered “bugs” in the three relevant cessively adding random samples until reaching 17% branch
JavaScript engines SpiderMonkey, JavaScriptCore, and V8 do coverage, which is roughly half the coverage yielded by the
not seem to represent actual security-critical vulnerabilities. entire DIE corpus. Each sub-corpus was used for a separate
For example, the first four bugs reported for SpiderMonkey evaluation. We acknowledge that not using the original input
are marked as invalid by the developers and do not represent corpus might result in worse results than previously reported.
a vulnerability at all. The fifth report is a duplicate. Similarly, However, the fact that a fuzzer works well without a specific
the “bug” reported for JavaScriptCore is marked as invalid body is a characteristic that a fuzzer must have to be generic.
by the developers, too. We were puzzled by this analysis Evaluation: To evaluate the code coverage, we split the
of the reported results and the unavailability of the source collected sample files into sets for each minute of fuzzing.
code. Unfortunately, the concerns could not be resolved in a Eachsetwasevaluatedagainstthecorrespondingllvm-cov[6]
directexchangewiththeauthors,sonodirectcomparisonwas instrumented engine. We merged the resulting coverage data
possible. for each set across time, starting with the time-wise first set
3) Comparison with Superion: A widely-used metric to forFuzzilliandthecoverageoftheinputcorpusforSuperion.
comparefuzzersiscodecoverage,asitshowshowmuchofan The total coverage encompasses the coverage of the whole
engine is reached and consequently tested. We opted to lever- engine denoted by the llvm-cov report file as “TOTAL”. To
11

Table II: Final mean branch coverage results. DIE Coverage
An interesting observation is that Wang et al. [43] reported
denotes the coverage already reached by the provided DIE
a line coverage increase of Superion for WebKit/JSC from
corpus prior to fuzzing. The first number represents the full
52.4% to 78.0%, an additional 25.6% of coverage. However,
corpus the second, the randomly selected sub-corpora.
using the DIE corpus with an initial line coverage of 52.01%
Engine DIECoverage SuperionCoverage OurCoverage only lead to a final line coverage of 53.45%, an addition of
JSC 44.56%/17% 46.71%/28.14% 43.72%
merely 1.44% (ref. Figure 8). Our assumption is that this is
JSCJIT 56.78%/34.98% 57.67%/49.35% 59.22%
V8 35.17%/17% 36.15%/23.13% 30.64% duetothedifferenceininputcorporaandshouldbeconsidered
V8JIT 53.33%/38.82% 54.22%/47.93% 53.47% a demonstration of the effect different input corpora can have
SM 36.49%/17% 38.96%/23.74% 30.53% on the final coverage.
SMJIT 59.28%/43.82% 60.50%/52.69% 56.27%
We strictly outperformed Superion when it was only pro-
vided with the reduced corpus. Our fuzzer reached an addi-
tional coverage of 15%, 7%, and 7% for JavaScriptCore, V8,
gain the JIT specific coverage, we extracted and averaged the
and SpiderMonkey. In contrast to the large corpus, Superion
reported coverage for each file potentially involved in JIT
could noticeably improve on the coverage again, showing the
compilation, using regular expressions on the individual file
paths1. impact an initial corpus can make on the results. However,
those improvements are of questionable benefit, as they are
Results: We evaluated the branch coverage and averaged
a subset of the already existing larger DIE corpus, which
it across our five runs. Superion improves the initial overall
largely consists of browser vendor test cases. Thus, no overall
coverage of the full DIE input corpus by 2.15%, 0.98%,
improvement in testing of JavaScript engines has been done.
and 2.47% for JavaScriptCore, V8, and SpiderMonkey, re-
Concerning JIT specific coverage, we outperformed Su-
spectively. For the 17% coverage sub-corpus, Superion im-
perion on a full DIE corpus for JSC by 1.6% and got
proves the coverage by 11.14%, 6.13%, and 6.74%. Our
outperformed by 0.8% and 3.8% for V8 and SpiderMonkey.
fuzzer reached a final coverage of 43.72%, 30.64%, and
But again, the initial coverage by the start corpus itself was
30.53%.ConcerningJITspecificcoverage,Superionimproves
alreadyhigherthanourfinalcoverageandtheaddedcoverage
by 0.89%, 0.89%, and 1.22%. For the 17% coverage sub-
by Superion was marginal. However, this demonstrates that
corpus the improvements are 14.37%, 9.11%, and 8.87%.
we are able to outperform or compete with Superion in terms
Our fuzzer reached a final coverage of 59.22%, 53.47%, and
of JIT focused fuzzing even if Superion is supplied with a
56.27%.
comprehensive start corpus. The reduced start corpus lead to
We also evaluated the line coverage specifically for
similar results for JIT coverage as it did for full coverage.
JavaScriptCore,asthisistheintersectionofevaluatedengines
by us and Wang et al., as well as their reported metric [43].
Using the full DIE Corpus with an initial line coverage of
C. Lessons Learned
52.01%, Superion improves by 1.44%. For the partial corpus,
Superion improves by 11.49%. Our fuzzer reaches a line cov- We were able to compete with and, for JSC, even outper-
erageof49.51%.ForJIT-specificcoverage,Superionimproves formSuperionwhenitcomestoJITcoverageevenwhenpro-
the full DIE corpus with an initial coverage of 64.58% by viding Superion with the full DIE corpus. Surprisingly, when
0.53%. For the partial corpus, with an initial coverage of looking at the overall coverage, Superion barely improved
44.10%, Superion improves by 14.13%. Our fuzzer reaches on the full DIE corpus. Concerning a reduced initial corpus,
a JIT specific line coverage of 65.60%. we strictly outperformed Superion in both JIT and general
The plots showing branch coverage over time are given coverage.
in Figure 7. A tabular overview of the raw branch coverage Finally, we were unable to reproduce the reported 25.6%
results can be found in Table II. Figure 8 visualizes the improvement of line coverage for JSC even though we also
comparison along line coverage. started at an initial line coverage of 52%. This is perplexing
Discussion: Line and branch coverage are similar for to us, as this suggests that the corpus used by Wang et al. had
JavaScriptCore, leading to the assumption that either metric significantandreachablecoveragegapsthatcouldbefilledbut
is interchangeable. Overall, Superion outperformed our fuzzer that do not exist in the DIE corpus. As the original Superion
whenprovidedwiththefullDIEcorpus.Thefinalcoverageof corpushasnotbeenpublished,wewereunfortunatelynotable
Superion was 3%, 6%, and 8% larger than our final coverage. to try and reproduce their original results. As a consequence,
However, the coverage that was achieved in addition to the we emphasize that future fuzzing research has to provide
start corpus was marginal, and the DIE corpus itself already any initial corpus to enable reproduction, and more emphasis
reached a higher coverage than the final coverage of our shouldbeputonreproductionofpreviousfuzzingresultswith
fuzzer. Consequently, the better coverage cannot be attributed different,possiblynovel,corporatoestimatehowwellafuzzer
to Superion. generalizes across different corpora.
Concerning the influence of different mutation strategies,
we showed that a small but constant effort to stress the JIT
1JSC:’. ∗ /(b3|ftl|dfg|assembler|jit)/.∗’,V8:’. ∗ src/compiler/.∗’,
SpiderMonkey:’.∗js/src/jit/.∗’ suffices to be able to focus on JIT vulnerabilities.
12

Figure7:Thefirstrowshowsthecomparisonofoverallcoverage,thesecondrowshowstheJITspecificcoverage.Theengines
are shown in the order JavaScriptCore, V8, and SpiderMonkey. The main line displays the mean across all runs, whereas the
shaded area denotes the confidence band ranging from minimum value to maximum value.
|     |     |     |        |     | VIII.  | CONCLUSION |        |     |        |         |
| --- | --- | --- | ------ | --- | ------ | ---------- | ------ | --- | ------ | ------- |
|     |     |     | During | the | course | of this    | paper, | we  | showed | how JIT |
compilationcanleadtoseriousvulnerabilitiesandwhycurrent
|     |     |     | fuzzing           | approaches    | are        | insufficient    |              | to detect | such             | vulnera-   |
| --- | --- | --- | ----------------- | ------------- | ---------- | --------------- | ------------ | --------- | ---------------- | ---------- |
|     |     |     | bilities.         | We proposed   |            | filling         | this fuzzing |           | gap by           | our novel  |
|     |     |     | approach          | of generating |            | semantically    |              | correct   | code,            | leveraging |
|     |     |     | an IR with        | a proportion  |            | of JIT          | focused      | mutation  | strategies.      |            |
|     |     |     | We implemented    |               | our        | approach        | in           | the swift | programming      |            |
|     |     |     | language          | and ran       | a 6-months |                 | experiment   | on        | 500 cores        | against    |
|     |     |     | V8, SpiderMonkey, |               | and        | JavaScriptCore. |              | During    | this             | test time  |
|     |     |     | frame we          | discovered    | 17         | previously      |              | unknown   | vulnerabilities. |            |
16
|     |     |     | Those vulnerabilities |                | were | on              | average | at  | least       | months |
| --- | --- | --- | --------------------- | -------------- | ---- | --------------- | ------- | --- | ----------- | ------ |
|     |     |     | old and               | were therefore |      | also overlooked |         | by  | the fuzzers | of the |
|     |     |     | respective            | vendors        | and  | researchers.    |         |     |             |        |
TofosterresearchandstrengthenthesecurityofJSengines,
|     |     |     | we will     | open source     | our             | code.       |                  |               |           |             |
| --- | --- | --- | ----------- | --------------- | --------------- | ----------- | ---------------- | ------------- | --------- | ----------- |
|     |     |     | We also     | performed       | a               | descriptive |                  | and empirical |           | analysis of |
|     |     |     | our fuzzer. | In              | our descriptive |             | analysis         | we            | showcased | how         |
|     |     |     | our fuzzer  | generalizes     |                 | well across | different        |               | engines   | and was     |
|     |     |     | able to     | find previously |                 | unknown     | vulnerabilities, |               | showing   | its         |
|     |     |     | qualitative | improvements    |                 | of the      | state            | of the        | art. Our  | empirical   |
analysisshowedthataconstant,butlimited,mutationfocusfor
|     |     |     | the JIT     | is deployed | by      | our fuzzer. | Furthermore, |            | the      | empirical |
| --- | --- | --- | ----------- | ----------- | ------- | ----------- | ------------ | ---------- | -------- | --------- |
|     |     |     | analysis    | showed      | that we | are         | able to      | outperform | or       | compete   |
|     |     |     | with the    | state of    | the art | fuzzer      | Superion     |            | when it  | comes to  |
|     |     |     | JIT focused | fuzzing     | even    | when        | providing    |            | Superion | with a    |
Figure8:Linecoverageevaluationfortheoverallengine(top)
|                |              |                 | comprehensive |             | start corpus. | When          | reducing |             | the start | corpus,    |
| -------------- | ------------ | --------------- | ------------- | ----------- | ------------- | ------------- | -------- | ----------- | --------- | ---------- |
| and concerning | JIT specific | files (bottom). |               |             |               |               |          |             |           |            |
|                |              |                 | Superion      | was unable  |               | to outperform |          | us, neither | for       | general    |
|                |              |                 | nor for       | JIT focused | coverage      |               | across   | all three   | engines.  | Those      |
|                |              |                 | results       | underline   | the           | importance    | to       | test        | fuzzer    | leveraging |
13

input corpora across multiple different corpora to judge the [15] John Aycock. A brief history of just-in-time. ACM Comput. Surv.,
generality. We also call for all future fuzzing research to 35(2):97–113,jun2003.
[16] Tim Blazytko, Cornelius Aschermann, Moritz Schloegel, Ali Abbasi,
be required to publish not only source code but also used
SergejSchumilo,SimonWo¨rner,andThorstenHolz. Grimoire:Synthe-
evaluation corpora. sizingStructurewhileFuzzing. InUSENIXSecuritySymposium,2019.
However,ourfuzzingmethodologyisnotcompleteasthere [17] Marcel Bo¨hme, Van-Thuan Pham, Manh-Dung Nguyen, and Abhik
Roychoudhury. Directed greybox fuzzing. In ACM Conference on
is still room to improve type information, e.g., by instrument-
ComputerandCommunicationsSecurity(CCS),2017.
ing the emitted code, to be more exact, which would allow
[18] MarcelBo¨hme,Van-ThuanPham,andAbhikRoychoudhury.Coverage-
for even more targeted code generation. Furthermore, our set basedgreyboxfuzzingasmarkovchain.IEEETransactionsonSoftware
of mutations is limited. Increasing the set of mutations to Engineering,45(5):489–506,2017.
[19] Carl Friedrich Bolz, Antonio Cuni, Maciej Fijalkowski, and Armin
include focused mutations for control flow could yield even
Rigo. Tracingthemeta-level:Pypy’stracingjitcompiler. InWorkshop
more deeply hidden vulnerabilities. Also the set of special on the Implementation, Compilation, Optimization of Object-Oriented
features can still be increased, as JavaScript is a complex and LanguagesandProgrammingSystems(ICOOOLPS),2009.
[20] Stefan Brunthaler. Inline caching meets quickening. In European
feature rich language and less popular features might contain
ConferenceonObject-OrientedProgramming(ECOOP),2010.
yet undiscovered issues. [21] Y. Chen, R. Zhong, H. Hu, H. Zhang, Y. Yang, D. Wu, and W. Lee.
OneEnginetoFuzz’emAll:GenericLanguageProcessorTestingwith
AVAILABILITY SemanticValidation.InIEEESymposiumonSecurityandPrivacy,2021.
[22] Anshuman Dasgupta. Tailoring traditional optimizations for runtime
All of our research artifacts are available online at https://
compilation. RiceUniversity,2007.
github.com/evaluating-fuzzilli-for-js-jit-fuzzingandthefuzzer
[23] KyleDewey,JaredRoesch,andBenHardekopf.Languagefuzzingusing
itself is available at https://github.com/googleprojectzero/ constraint logic programming. In ACM International Conference on
fuzzilli.
AutomatedSoftwareEngineering(ASE),2014.
[24] Sung Ta Dinh, Haehyun Cho, Kyle Martin, Adam Oest, Kyle Zeng,
ACKNOWLEDGEMENTS Alexandros Kapravelos, Gail-Joon Ahn, Tiffany Bao, Ruoyu Wang,
AdamDoupe´,andYanShoshitaishvili. Favocado:Fuzzingthebinding
Funded by the Deutsche Forschungsgemeinschaft (DFG, code of javascriptengines using semantically correct test cases. In
German Research Foundation) under Germany’s Excellence SymposiumonNetworkandDistributedSystemSecurity(NDSS),2021.
[25] Andrea Fioraldi, Daniele Cono D’Elia, and Emilio Coppa. Weizz:
Strategy - EXC 2092 CASA - 390781972, by the German
Automatic grey-box fuzzing for structured binary formats. In ACM
Federal Ministry of Education and Research (BMBF, project SIGSOFT International Symposium on Software Testing and Analysis,
KMU-Fuzz – 16KIS1523), and by the European Union’s 2020.
[26] Andrea Fioraldi, Dominik Maier, Heiko Eißfeldt, and Marc Heuse.
Horizon 2020 research and innovation program under grant
AFL++:CombiningIncrementalStepsofFuzzingResearch.InUSENIX
agreement No 101019206. WorkshoponOffensiveTechnologies(WOOT),2020.
[27] Andreas Gal, Brendan Eich, Mike Shaver, David Anderson, David
REFERENCES Mandelin, Mohammad R. Haghighat, Blake Kaplan, Graydon Hoare,
Boris Zbarsky, Jason Orendorff, Jesse Ruderman, Edwin W. Smith,
[1] AngularJS - Superheroic JavaScript MVW Framework. [software],
Rick Reitmaier, Michael Bebenita, Mason Chang, and Michael Franz.
https://www.google.com/search?client=safari&rls=en&q=angular+js&
Trace-basedjust-in-timetypespecializationfordynamiclanguages. In
ie=UTF-8&oe=UTF-8. visited2022-07-29.
ACM SIGPLAN Conference on Programming Language Design and
[2] DIE corpus. https://github.com/sslab-gatech/DIE-corpus.git. visited
Implementation(PLDI),2009.
2022-04-09.
[28] HyungSeokHan,DongHyeonOh,andSangKilCha. CodeAlchemist:
[3] GitHub:funfuzzshellflags.https://github.com/MozillaSecurity/funfuzz/
Semantics-AwarecodeGenerationtoFindVulnerabilitiesinJavaScript
blob/master/src/funfuzz/js/shell flags.py#L99. visited2022-07-29.
Engines. In Symposium on Network and Distributed System Security
[4] Introducing jsfunfuzz. http://www.squarefree.com/2007/08/02/
(NDSS),2019.
introducing-jsfunfuzz/. visited2022-07-29.
[29] Xiaoyu He, Xiaofei Xie, Yuekang Li, Jianwen Sun, Feng Li, Wei
[5] jQuery:TheWriteLess,DoMore,JavaScriptLibrary.[software],https:
Zou, Yang Liu, Lei Yu, Jianhua Zhou, Wenchang Shi, and Wei Huo.
//jquery.com. visited2022-07-29.
SoFi: Reflection-Augmented Fuzzing for JavaScript Engines. In ACM
[6] llvm-cov - emit coverage information – llvm 16.0.0git documentation.
ConferenceonComputerandCommunicationsSecurity(CCS),2021.
https://llvm.org/docs/CommandGuide/llvm-cov.html. visited 2022-07-
[30] Hodova´, Rena´ta, and A´kos Kiss. Fuzzing javascript engine apis. In
27.
InternationalConferenceonIntegratedFormalMethods-Volume9681,
[7] Peachfuzzer. https://peachtech.gitlab.io/peach-fuzzer-community/. vis-
2016.
ited2022-07-30.
[8] Reactjs. https://reactjs.org/. visited2022-06-07. [31] Christian Holler, Kim Herzig, and Andreas Zeller. Fuzzing with code
[9] Recent papers related to fuzzing. https://wcventure.github.io/ fragments. InUSENIXSecuritySymposium,2012.
FuzzingPaper/. visited2022-07-29. [32] Chris Lattner and Vikram Adve. Llvm: A compilation framework for
[10] Web technology for developersjavascript. https://developer.mozilla.org/ lifelongprogramanalysis&transformation.InInternationalSymposium
en-US/docs/Web/JavaScript. visited2022-07-29. onCodeGenerationandOptimization,2004.CGO2004.,pages75–86.
[11] Alfred Aho and Jeffrey D. Ullman. Principles of Compiler Design. IEEE,2004.
Addison-WesleyLongmanPublishingCo.,Inc.,1977. [33] SuyoungLee,HyungSeokHan,SangKilCha,andSooelSon.Montage:
[12] B.Alpern,M.N.Wegman,andF.K.Zadeck.Detectingequalityofvari- A neural network language model-guided javascript engine fuzzer. In
ablesinprograms. InACMSymposiumonPrinciplesofProgramming USENIXSecuritySymposium,2020.
Languages(POPL),1988. [34] V. Manes, H. Han, C. Han, s. cha, M. Egele, E. J. Schwartz, and
[13] EsbenAndreasen,LiangGong,AndersMøller,MichaelPradel,Marija M.Woo. Theart,science,andengineeringoffuzzing:Asurvey. IEEE
Selakovic, Koushik Sen, and Cristian-Alexandru Staicu. A survey of TransactionsonSoftwareEngineering,2019.
dynamicanalysisandtestgenerationforjavascript.ACMComput.Surv., [35] JasonMerrill. Genericandgimple:Anewtreerepresentationforentire
50(5):66:1–66:36,September2017. functions. InProceedingsofthe2003GCCDevelopers’Summit,pages
[14] Cornelius Aschermann, Tommaso Frassetto, Thorsten Holz, Patrick 171–179.Citeseer,2003.
Jauernig,Ahmad-RezaSadeghi,andDanielTeuchert. Nautilus:Fishing [36] Soyeon Park, Wen Xu, Insu Yun, Daehee Jang, and Taesoo Kim.
fordeepbugswithgrammars.InSymposiumonNetworkandDistributed FuzzingJavaScriptEngineswithAspect-preservingMutation. InIEEE
SystemSecurity(NDSS),2019. SymposiumonSecurityandPrivacy,2020.
14

|                                          |           |               |     |        |           |                    |     | ffx.js, and | v8.js.      | Note that Superion | did benefit          | from |
| ---------------------------------------- | --------- | ------------- | --- | ------ | --------- | ------------------ | --- | ----------- | ----------- | ------------------ | -------------------- | ---- |
| [37] B.K.Rosen,M.N.Wegman,andF.K.Zadeck. |           |               |     |        |           | Globalvaluenumbers |     |             |             |                    |                      |      |
| and                                      | redundant | computations. |     | In ACM | Symposium | on Principles      |     | of          |             |                    |                      |      |
|                                          |           |               |     |        |           |                    |     | changes in  | commandline | parameters,        | similar to Fuzzilli. | We   |
ProgrammingLanguages(POPL),1988.
madesurethatallfuzzersareevaluatedinafairandobjective
[38] ChristopherSalls,ChaniJindal,JakeCorina,ChristopherKruegel,and
way.
GiovanniVigna.Token-LevelFuzzing.InUSENIXSecuritySymposium,
| 2021.        |         |       |        |         |            |          |       | SpiderMonkey: |     |     |     |     |
| ------------ | ------- | ----- | ------ | ------- | ---------- | -------- | ----- | ------------- | --- | --- | --- | --- |
| [39] Prateek | Saxena, | Steve | Hanna, | Pongsin | Poosankam, | and Dawn | Song. |               |     |     |     |     |
a) Fuzzilli:
| Flax: | Systematic | discovery | of  | client-side | validation | vulnerabilities |     | in  |     |     |     |     |
| ----- | ---------- | --------- | --- | ----------- | ---------- | --------------- | --- | --- | --- | --- | --- | --- |
richwebapplications.InSymposiumonNetworkandDistributedSystem
−−fuzzing−safe
Security(NDSS),2010.
−−no−threads
| [40] Ben | Stock, | Martin Johns, | Marius | Steffens, | and | Michael Backes. | How |     |     |     |     |     |
| -------- | ------ | ------------- | ------ | --------- | --- | --------------- | --- | --- | --- | --- | --- | --- |
the Web Tangled Itself: Uncovering the History of Client-Side Web −−baseline−warmup−threshold=10
−−ion−warmup−threshold=100
| (In)Security. |     | InUSENIXSecuritySymposium,2017. |     |     |     |     |     |     |     |     |     |     |
| ------------- | --- | ------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
−−ion−check−range−analysis
[41] Kwangwon Sun and Sukyoung Ryu. Analysis of javascript programs: −−ion−extra−checks
| Challengesandresearchtrends. |       |            |              | ACMComput.Surv.,50(4):59:1–59:34, |                |               |          |              |     |     |     |     |
| ---------------------------- | ----- | ---------- | ------------ | --------------------------------- | -------------- | ------------- | -------- | ------------ | --- | --- | --- | --- |
| August2017.                  |       |            |              |                                   |                |               |          | b) Superion: |     |     |     |     |
| [42] Junjie                  | Wang, | Bihuan     | Chen, Lei    | Wei, and                          | Yang           | Liu. Skyfire: | Data-    |              |     |     |     |     |
| driven                       | seed  | generation | for fuzzing. | In                                | IEEE Symposium | on            | Security |              |     |     |     |     |
−−fuzzing−safe
andPrivacy,2017.
−−no−threads
| [43] JunjieWang,BihuanChen,LeiWei,andYangLiu.Superion:Grammar- |     |     |     |               |            |     |          | −−fast−warmup |     |     |     |     |
| -------------------------------------------------------------- | --- | --- | --- | ------------- | ---------- | --- | -------- | ------------- | --- | --- | --- | --- |
|                                                                |     |     |     | International | Conference | on  | Software |               |     |     |     |     |
Aware Greybox Fuzzing. In −f /chakra.js −f /ffx.js −f /jsc.js −f /v8.js
Engineering(ICSE),2019.
| [44] Pengfei | Wang | and Xu | Zhou. | Sok: The | progress, | challenges, | and |     |     |     |     |     |
| ------------ | ---- | ------ | ----- | -------- | --------- | ----------- | --- | --- | --- | --- | --- | --- |
JavaScriptCore:
| perspectivesofdirectedgreyboxfuzzing. |     |     |     |     | corr/arxive,2020. |     |     |     |     |     |     |     |
| ------------------------------------- | --- | --- | --- | --- | ----------------- | --- | --- | --- | --- | --- | --- | --- |
c) Fuzzilli:
| [45] MichalZalewski. |     | americanfuzzylop. |     | https://lcamtuf.coredump.cx/afl/. |     |     |     |     |     |     |     |     |
| -------------------- | --- | ----------------- | --- | --------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
−−validateOptions=true
APPENDIXA
−−thresholdForJITSoon=10
|     |     | V8GENERATOREFFECTPLOT |     |     |     |     |     | −−thresholdForJITAfterWarmUp=10 |     |     |     |     |
| --- | --- | --------------------- | --- | --- | --- | --- | --- | ------------------------------- | --- | --- | --- | --- |
−−thresholdForOptimizeAfterWarmUp=100
| Figure | 9 shows | the | fraction | of  | applied | mutations | over |     |     |     |     |     |
| ------ | ------- | --- | -------- | --- | ------- | --------- | ---- | --- | --- | --- | --- | --- |
−−thresholdForOptimizeAfterLongWarmUp=100
−−thresholdForOptimizeSoon=100
| time in | V8. We | observed | that | the distribution |     | is similar |     | to  |     |     |     |     |
| ------- | ------ | -------- | ---- | ---------------- | --- | ---------- | --- | --- | --- | --- | --- | --- |
−−thresholdForFTLOptimizeAfterWarmUp=1000
SpiderMonkey and JavaScriptCore (see Figure 6). Note that −−thresholdForFTLOptimizeSoon=1000
the total number of mutations is lower from the beginning, −−validateBCE=true $file
| but does | not | converge | to zero | towards | the | end, suggesting |     |     |     |     |     |     |
| -------- | --- | -------- | ------- | ------- | --- | --------------- | --- | --- | --- | --- | --- | --- |
d) Superion:
| that V8 | would           | benefit | from longer | fuzzing     |     | runs as it | does not |     |     |     |     |     |
| ------- | --------------- | ------- | ----------- | ----------- | --- | ---------- | -------- | --- | --- | --- | --- | --- |
| exhaust | its exploration |         | paths       | as quickly. |     |            |          |     |     |     |     |     |
−−useConcurrentJIT=false
−−useConcurrentGC=false
−−thresholdForJITSoon=10
−−thresholdForJITAfterWarmUp=10
−−thresholdForOptimizeAfterWarmUp=100
−−thresholdForOptimizeAfterLongWarmUp=100
−−thresholdForFTLOptimizeAfterWarmUp=1000
−−thresholdForFTLOptimizeSoon=1000
|     |     |     |     |     |     |     |     | −f /chakra.js | −f /ffx.js | −f /jsc.js | −f /v8.js |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | ---------- | ---------- | --------- | --- |
V8:
e) Fuzzilli:
−−expose−gc
−−fuzzing
−−allow−natives−syntax
−−interrupt−budget=1024
f) Superion:
−−expose−gc
−−fuzzing
−−predictable
|           |            |          |              |                |       |                   |           | −f /chakra.js | −f /ffx.js | −f /jsc.js | −f /v8.js |     |
| --------- | ---------- | -------- | ------------ | -------------- | ----- | ----------------- | --------- | ------------- | ---------- | ---------- | --------- | --- |
| Figure 9: | Temporal   | analysis | of           | the proportion |       | of our            | different |               |            |            |           |     |
| mutation  | strategies | for      | V8, measured |                | at 10 | minute intervals. |           |               |            |            |           |     |
APPENDIXB
JSENGINEFLAGS
| The following  |          | listings    | summarize |       | the command | line     | flags    |     |     |     |     |     |
| -------------- | -------- | ----------- | --------- | ----- | ----------- | -------- | -------- | --- | --- | --- | --- | --- |
| used during    | the      | experiments | to        | start | the code    | coverage | evalu-   |     |     |     |     |     |
| ation. As      | Superion | was         | fed with  | the   | DIE corpus, | we       | supplied |     |     |     |     |     |
| the evaluation |          | with the    | required  | setup | JavaScript  | files    | jsc.js,  |     |     |     |     |     |
15

APPENDIXC
OPERATIONSANDGENERATORS
TableIII:OverviewoftheoperationsimplementedinourIRandthecorrespondingJavaScriptlanguagefeaturethattheycover.
Operation CoveredJavaScriptLanguageFeature
Nop Emptystatement(doesnothing)
LoadInteger Anumberliteralcontaininganintegervalue
LoadFloat Anumberliteralcontainingafloatingpointvalue
LoadString Astringliteral
LoadBoolean Abooleanliteral
LoadUndefined Theundefinedvalue
LoadNull Thenullvalue
CreateObject Anobjectliteral
CreateArray Anarrayliteral
CreateObjectWithSpread Anobjectliteralpossiblyusingspreadsyntax
CreateArrayWithSpread Anarrayliteralpossiblyusingspreadsyntax
LoadBuiltin Variableaccess(builtinobjectsareaccessiblethroughglobalvariables)
LoadProperty Propertyaccessusingthedotnotation
StoreProperty Propertyaccessusingthedotnotation
DeleteProperty Propertyaccessusingthedotnotation
LoadElement Propertyaccessusingthebracketnotation(withaconstantintegeraspropertyname)
StoreElement Propertyaccessusingthebracketnotation(withaconstantintegeraspropertyname)
DeleteElement Propertyaccessusingthebracketnotation(withaconstantintegeraspropertyname)
LoadComputedProperty Propertyaccessusingthebracketnotation
StoreComputedProperty Propertyaccessusingthebracketnotation
DeleteComputedProperty Propertyaccessusingthebracketnotation
TypeOf Thetypeofoperator
InstanceOf Theinstanceofoperator
In Theinoperator
BeginFunctionDefinition Aplainfunction
Return Thereturnstatement
EndFunctionDefinition Aplainfunction
CallMethod Amethodcall
CallFunction Afunctioncall
Construct Aconstructorcall
CallFunctionWithSpread Afunctioncallpossiblyusingspreadsyntax
UnaryOperation Aunaryoperation
BinaryOperation Abinaryoperation
Phi Variabledefinition/assignment
Copy Variableassignment
Compare Acomparisonoperation
BeginWith Awithstatement
EndWith Awithstatement
LoadFromScope Variableaccess(inawithstatement,propertiesofthecontextobjectbecomelocalvariables)
StoreToScope Variableaccess(inawithstatement,propertiesofthecontextobjectbecomelocalvariables)
BeginIf Ifstatement
BeginElse Ifstatement
EndIf Ifstatement
BeginWhile Whileloop
EndWhile Whileloop
BeginDoWhile Do-Whileloop
EndDoWhile Do-Whileloop
BeginFor Forloop
EndFor Forloop
BeginForIn For-Inloop
EndForIn For-Inloop
BeginForOf For-Ofloop
EndForOf For-Ofloop
Break Breakstatement
Continue Continuestatement
BeginTry Try-Catchstatement
BeginCatch Try-Catchstatement
EndTryCatch Try-Catchstatement
ThrowException Throwoperation
16

Table IV: A complete list of all code generators used and a brief description. If a generator emits a block, that block is filled
| with the result                  | of another code | generator invocation.                        |
| -------------------------------- | --------------- | -------------------------------------------- |
| Name                             |                 | Description                                  |
| IntegerLiteralGenerator          |                 | Loadsarandominteger                          |
| FloatLiteralGenerator            |                 | Loadsarandomfloatingpointnumber              |
| StringLiteralGenerator           |                 | Loadsarandomstring                           |
| BooleanLiteralGenerator          |                 | Loadsarandomboolean                          |
| UndefinedValueGenerator          |                 | Loadstheundefinedvalue                       |
| NullValueGenerator               |                 | Loadsthenullvalue                            |
| BuiltinGenerator                 |                 | Loadsareferencetoarandombuiltin              |
| ObjectLiteralGenerator           |                 | Generatesanobjectliteral                     |
| ArrayLiteralGenerator            |                 | Generatesanarrayliteral                      |
| ObjectLiteralWithSpreadGenerator |                 | Generatesanobjectliteralusingspreadingsyntax |
| ArrayLiteralWithSpreadGenerator  |                 | Generatesanarrayliteralusingspreadingsyntax  |
| FunctionDefinitionGenerator      |                 | Definesanewfunction                          |
| FunctionReturnGenerator          |                 | Generatesareturnstatement                    |
| PropertyRetrievalGenerator       |                 | Loadsarandompropertyonanexistingvalue        |
PropertyAssignmentGenerator Storesanexistingvalueasrandompropertyonanexistingvalue
| PropertyRemovalGenerator  |     | Deletesarandompropertyofanexistingvalue     |
| ------------------------- | --- | ------------------------------------------- |
| ElementRetrievalGenerator |     | Loadsarandomindexedelementofanexistingvalue |
ElementAssignmentGenerator Storesarandomvalueasindexedelementonanexistingvalue
ElementRemovalGenerator Deletesarandomindexedelementfromanexistingvalue
TypeTestGenerator PerformstheJavaScripttypeofoperatoronanexistingvalue
InstanceOfGenerator PerformstheJavaScriptinstanceofoperatoronexistinginputs
| InGenerator                         |     | PerformstheJavaScriptinoperatoronexistinginputs |
| ----------------------------------- | --- | ----------------------------------------------- |
| ComputedPropertyRetrievalGenerator  |     | Loadsacomputedpropertyofanexistingvalue         |
| ComputedPropertyAssignmentGenerator |     | Storesacomputedpropertyofanexistingvalue        |
| ComputedPropertyRemovalGenerator    |     | Removesacomputedpropertyfromanexistingvalue     |
FunctionCallGenerator Callsanexistingfunctionwithexistingvaluesasarguments
FunctionCallWithSpreadGenerator CallsanexistingfunctionusingtheJavaScriptspreadingsyntax
| MethodCallGenerator      |     | Callsarandommethodonanexistingvalue          |
| ------------------------ | --- | -------------------------------------------- |
| ConstructorCallGenerator |     | Performsaconstructorcallonanexistingfunction |
UnaryOperationGenerator Performsarandomunaryoperationonanexistingvalue
BinaryOperationGenerator Performsarandombinaryoperationontwoexistingvalues
PhiGenerator Createsaphivariablewithanexistingvalueasinitialvalue
ReassignmentGenerator Reassignsanexistingphivariabletoadifferent,existingvalue
| WithStatementGenerator |     | GeneratesaJavaScriptwithstatement |
| ---------------------- | --- | --------------------------------- |
LoadFromScopeGenerator Insideawithstatement,loadapropertyofthecontextobject
StoreToScopeGenerator Insideawithstatement,storeapropertyofthecontextobject
ComparisonGenerator Generatesarandomcomparisonoperationoftwoexistingvalues
| IfStatementGenerator |     | Generatesanif-elsestatement |
| -------------------- | --- | --------------------------- |
| WhileLoopGenerator   |     | Generatesawhileloop         |
| DoWhileLoopGenerator |     | Generatesado-whileloop      |
| ForLoopGenerator     |     | Generatesaforloop           |
| ForInLoopGenerator   |     | Generatesafor-inloop        |
| ForOfLoopGenerator   |     | Generatesafor-ofloop        |
| BreakGenerator       |     | Generatesabreakstatement    |
| ContinueGenerator    |     | Generatesacontinuestatement |
TryCatchGenerator Generatesatrycatchstatementwiththeresultofanothergeneratorasbodies
| ThrowGenerator |     | Throwsanexistingvalueasexception |
| -------------- | --- | -------------------------------- |
WellKnownPropertyLoadGenerator Loadsoneofthewell-knownSymbolpropertiesofanexistingvalue
WellKnownPropertyStoreGenerator Storestooneofthewell-knownSymbolpropertiesofanexistingvalue
| TypedArrayGenerator |     | ConstructsaJavaScripttypedarray |
| ------------------- | --- | ------------------------------- |
FloatArrayGenerator ConstructsaregularJavaScriptarraycontainingonlyfloatingpointnumbers
IntArrayGenerator ConstructsaregularJavaScriptarraycontainingonlyintegers
| ObjectArrayGenerator     |     | ConstructsaregularJavaScriptarraycontainingobjects |
| ------------------------ | --- | -------------------------------------------------- |
| PrototypeAccessGenerator |     | Retrievestheprototypeofanexistingvalue             |
PrototypeOverwriteGenerator Changestheprototypeofanexistingvaluetoanotherexistingvalue
CallbackPropertyGenerator InstallsavalueOfortoStringcallbackonanexistingvalue
PropertyAccessorGenerator Generatesapropertygetterandsetteronanexistingvalue
| ProxyGenerator |     | GeneratesaJavaScriptproxyobject |
| -------------- | --- | ------------------------------- |
LengthChangeGenerator Storesanumericvalueas.lengthpropertyonanexistingvalue
ElementKindChangeGenerator Storesanobjectvalueasindexedelementintoanexistingvalue
17
