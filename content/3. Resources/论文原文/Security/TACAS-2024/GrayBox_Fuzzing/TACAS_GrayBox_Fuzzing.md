---
publish: true
---

Could not get FontBBox from font descriptor because None cannot be parsed as 4 floats
Gray-Box Fuzzing via Gradient Descent and
Boolean Expression Coverage
Technical Report
Martin Jonáš1 , Jan Strejček1 , Marek Trtík2,3 , and Lukáš Urban3
Faculty of Informatics„ Masaryk University, Czechia
{martin.jonas,strejcek,trtikm,492717}@mail.muni.cz
Abstract. We present a novel gray-box fuzzing algorithm monitoring
executions of instructions converting numerical values to Boolean ones.
Animportantclassofsuchinstructionsevaluatepredicates,e.g.,*cmpin
LLVM. That alone allows us to infer the input dependency (c.f. the taint
analysis)duringthefuzzingon-the-flywithreasonableaccuracy,whichin
turnenablesaneffectiveuseofthegradientdescentontheseinstructions
(to invert the result of their evaluation). Although the fuzzing attempts
tomaximizethecoverageoftheinstructions,thereisaninterestingcor-
relationwiththestandardbranchcoverage,whichweareabletoachieve
indirectly. The evaluation on Test-Comp 2023 benchmarks shows that
our approach, despite being a pure gray-box fuzzing, is able to compete
with the leading tools in the competition, which combine fuzzing with
other powerful techniques like model checking, symbolic execution, or
abstract interpretation.
Keywords: gray-box · fuzzing · taint analysis · gradient descent
1 Architecture
tClien Sererv
er'sFizz
�ats c
arieslibr
erttrumenIns
32/64-bit
Fig.1. FIzzer’s modules
Ournovelgray-boxfuzzingalgorithmisimplementedinatoolcalledFIzzer.
It consists of Server, Client, and Instrumenter 64-bit executables, and a col-
lection of static Libraries, each provided in 32 and 64-bit version (see Fig.11).
1 There is also a Python script providing a user friendly interface to the whole tool.

| 2 Martin |     | Jonáš , Jan | Strejček | , Marek Trtík, | and | Lukáš Urban |     |     |
| -------- | --- | ----------- | -------- | -------------- | --- | ----------- | --- | --- |
The Server is responsible for generation of inputs for the analyzed program,
which we denote as the Target. It must first be built from an input C file into a
32 or 64-bit executable file 2 , as depicted in Fig.2. The Instrumenter and the
static Libraries play an important role in the process. Details are discussed in
the next section.
Instrumenta�on
| C   |     | Instrumenter |     |     |     | LLVM |     |     |
| --- | --- | ------------ | --- | --- | --- | ---- | --- | --- |
Fizzer's
|     | Compiling |     |     |     |     | Linking |     |     |
| --- | --------- | --- | --- | --- | --- | ------- | --- | --- |
sta�c
|       |     |     |           |     |     | Clang++ |     | Target |
| ----- | --- | --- | --------- | --- | --- | ------- | --- | ------ |
| Clang |     |     | libraries |     |     |         |     |        |
LLVM
32/64-bit
|     |     |     |        | Building the |        |     |     |     |
| --- | --- | --- | ------ | ------------ | ------ | --- | --- | --- |
|     |     |     | Fig.2. |              | Target |     |     |     |
TheClientexecutablemediatescommunicationbetweenServerandTarget
vianetwork.Thatisanalternativetypeofthecommunication.Theprimaryone
is the shared memory. Therefore, FIzzer can run without Client binary. We
| discuss details |     | of both kinds | of  | communication | in  | Sec.3. |     |     |
| --------------- | --- | ------------- | --- | ------------- | --- | ------ | --- | --- |
2 Instrumentation
TheInstrumenterisresponsibleforinsertion(instrumentation)ofamonitoring
code into the Target executable. This code, when executed, collects valuable
data about Target’s execution. The data are essential for an effective input
| generation | in the       | Server.  |     |            |        |                   |     |          |
| ---------- | ------------ | -------- | --- | ---------- | ------ | ----------------- | --- | -------- |
| The        | Instrumenter | proceeds |     | in several | steps. | First, it applies | the | standard |
LLVM pass replacing switch instructions by equivalent sequences of branchings.
3
Then, it renames each function in the LLVM module such that it adds a prefix
__fizzer_rename_prefix__.Thissteppreventsaccidentalnamecollisionswith
| those in | the standard | C library |     | or in FIzzer’s | Libraries. |     |     |     |
| -------- | ------------ | --------- | --- | -------------- | ---------- | --- | --- | --- |
4
| Next, | it surrounds | each | function | call instruction |     | by calls | to FIzzer’s | mon- |
| ----- | ------------ | ---- | -------- | ---------------- | --- | -------- | ----------- | ---- |
itoring functions
| void | __sbt_fizzer_process_call_begin(uint32_t |     |     |     |     | id); |     |     |
| ---- | ---------------------------------------- | --- | --- | --- | --- | ---- | --- | --- |
| void | __sbt_fizzer_process_call_end(uint32_t   |     |     |     |     | id); |     |     |
2 TheTargetmustbebuildforanarchitecturewiththesameendianastheoneused
| for building | of  | the Server. |     |     |     |     |     |     |
| ------------ | --- | ----------- | --- | --- | --- | --- | --- | --- |
3 Next should follow a replacement of calls via pointer by sequences of branchings,
| but that | is not | implemented | yet. |     |     |     |     |     |
| -------- | ------ | ----------- | ---- | --- | --- | --- | --- | --- |
4 We ignore special functions prefixed by __sbt_fizzer_ and __VERIFIER_nondet_.

| Gray-Box | Fuzzing | via | Gradient | Descent | and Boolean | Expression | Coverage | 3   |
| -------- | ------- | --- | -------- | ------- | ----------- | ---------- | -------- | --- |
both accepting the same unique ID of that call instruction. Tracking function
calls allows Server to include calling context into the input generation process.
| Lastly, | it inserts | monitoring |     | code after | each | instruction | converting | one or |
| ------- | ---------- | ---------- | --- | ---------- | ---- | ----------- | ---------- | ------ |
more numeric values to a Boolean one. We call these instructions as Boolean
instructions. The comparison *cmp instructions are Boolean instructions of the
highest importance. However, truncation instructions and calls to functions re-
turning Boolean type are also of the kind. The instrumented monitoring code
is supposed to collect maximum information from the conversion. Namely, con-
version is quantified by a value of the type double. For truncation and Boolean
function call instructions the value is always 1. But for a comparison instruc-
tion the value is inferred from its predicate, having a general form l ▷◁ r,
where l and r are some LLVM registers of a numeric type and ▷◁ is a com-
parator from {=,̸=,<,≤,>,≥}. The instrumented code computes the value
5
(double)l −(double)r. This value is passed as the third argument to the
| monitoring | function                                |               |     |        |     |        |            |     |
| ---------- | --------------------------------------- | ------------- | --- | ------ | --- | ------ | ---------- | --- |
| void       | __sbt_fizzer_process_condition(uint32_t |               |     |        |     |        | id,        |     |
|            | bool                                    | instr_result, |     | double |     | value, | bool xor); |     |
together with the unique ID of the comparison instruction (1st argument), the
resulting Boolean value of the comparison instruction (2nd argument), and
Boolean value determining whether there appears a xor instruction anywhere
| before the   | comparison | instruction    |              | in the    | same      | basic block | or not.     |     |
| ------------ | ---------- | -------------- | ------------ | --------- | --------- | ----------- | ----------- | --- |
| For example, |            | a C expression |              |           |           |             |             |     |
| x <          | 123456789  |                |              |           |           |             |             |     |
| where x is   | of the     | int type,      | is expressed |           | by LLVM’s | Boolean     | instruction |     |
| %4 =         | icmp       | slt i32        | %3,          | 123456789 |           |             |             |     |
where %3 is the register holding the value of x. The Instrumenter inserts the
| following | code after | the                                 | instruction |            |     |            |                      |     |
| --------- | ---------- | ----------------------------------- | ----------- | ---------- | --- | ---------- | -------------------- | --- |
| %5 =      | sext       | i32 %3                              | to i64      |            |     |            |                      |     |
| %6 =      | sub        | i64 %5,                             | 123456789   |            |     |            |                      |     |
| %7 =      | sitofp     | i64                                 | %6 to       | double     |     |            |                      |     |
| call      | void       | @__sbt_fizzer_process_condition(i32 |             |            |     |            | 1, i1 %4,            |     |
|           |            | double                              | %7,         | i1 false)  |     |            |                      |     |
| At this   | point      | it may                              | not be      | clear, why | we  | instrument | Boolean instructions |     |
rather than branching br instructions. The reason for that is to be able to com-
pute the double values with the maximal precision. For example, if we instru-
mented branching instructions, then we would get almost zero precision from
| any C code | of this | pattern |     |     |     |     |     |     |
| ---------- | ------- | ------- | --- | --- | --- | --- | --- | --- |
5 If the size of the type of l or r is greater than or equal to the size of double, then
wemaynotinfactgetmaximuminformationduetopossibleoverfloworunderflow.

4 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
int foo(int b) { ... if (b) ... }
... foo(x < 123456789) ...
Observe that foo accepts int, which can only be either zero or one. Therefore,
the value (double)b - (double)0 6 computed at the branching if (b) inside
foo can also only be either zero or one. In contrast, the value (double)x -
(double)123456789computedfromtheBooleaninstructionatthecallsitecan
be arbitrary.
The same effect can also be observed for another frequently used pattern
struct ListItem { ... bool flag; ... };
... item->flag = x < 123456789; ...
... if (item->flag) ....
In this code the result of the evaluation of x < 123456789 is stored in a list
item and it is used later in a program branching.
Unfortunately,instrumentationof Booleaninstructionshasalsoadrawback,
related to measuring coverage.
– A Boolean instruction is covered, iff it was evaluated for at least one test
generated by the Server to true and also for at least one test to false.
– A branching br instruction is covered, iff it was evaluated for at least one
test generated by the Server such that the execution continued to the true
branch and also for at least one test the execution continued to the false
branch.
Now, consider the following C program
int x,y;
... // Read input to variables x and y.
bool b1 = (x == 1);
bool b2 = (y == 1);
if (b1)
if (b2) return 1; else return 2;
else
if (b2) return 3; else return 4;
If the Server generates two inputs
x <- 0, y <- 0 and x <- 1, y <- 1
then both Boolean instructions x == 1 and y == 1 are covered, while only
branching instruction corresponding to if (b1) is covered and the other two
are not.
Although FIzzer’s primary goal is to generate inputs maximizing coverage
of branching instructions, the goal is approached indirectly through maximizing
coverageof Booleaninstructions.Thereasonsforthatisthefact,thateffectivity
6 if (b) is only an abbreviation of if (b != 0).

Gray-Box Fuzzing via Gradient Descent and Boolean Expression Coverage 5
of Server in input generation fundamentally depends on information captured
in the computed double values.
The secondary information contributing to the efficiency of input generation
is the count of input bytes read from the start of the Target up to each call
to this monitoring function. The count is not passed to the function as the
parameter, because the information is available from functions providing input
to the program (they are discussed below). Therefore, the count of the input
bytes read is recorded together with the information passed via parameters.
The Libraries linked to the Target provide the main function of the ex-
ecutable (the original one is renamed and called from the library one), and
definitions of functions called from the instrumented monitoring code, namely:
void __sbt_fizzer_process_condition(uint32_t id,
bool instr_result, double value, bool xor);
void __sbt_fizzer_process_call_begin(uint32_t id);
void __sbt_fizzer_process_call_end(uint32_t id);
There are also definitions of functions providing input to the program. Cur-
rently, this is limited to the concept used in the Test-Comp competition, i.e., to
functions with the prototype
T __VERIFIER_nondet_T();
where T stands for any basic type, like int, char, float, etc.
3 Fuzzing loop
(1)Config
Shared
Server Target
memory
(2)Results
(1)Config (2)Config
Client
Client Shared
Server Network Client Target
memory
(4)Results (3)Results
Fig.3. Fuzzing loop via shared memory (top) and via network (bottom).
The analysis in FIzzer is performed within a top level loop, called fuzzing
loop. In each iteration the Server generates an input (which is the subject of

6 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
the next section), executes the Target with it, and processes data produced by
the executed monitoring code (see previous section).
Detailsofaniteration,withthefocusondataflow,areshowninFig.3(top).
The Server and the Target are separate processes, because the Server may
generate an input for which the Target crashes. If that crashed Server too, the
analysiswillbeover.Theprocessesexchangedataviasharedmemory,sincethat
is the fastest way of inter-process communication.
We can further see that Server first sends a Config to the Target. It com-
prizes of the following data:
– Maximum length of the execution trace. The length is the number of exe-
cutions of the monitoring code of the Boolean expressions. The reason for
this limit is simple. Long execution trace consumes a lot of memory and its
processing by the Server decreases an overall performance of the analysis.
– Maximum stack size. Since our analysis is context sensitive, we also restrict
size of the stack to manageable size.
– Maximum number of input bytes the Target may read. The Server uses
inputs from previous iterations of the loop for input generation in later iter-
ations. We thus need to keep the size of inputs in reasonable bounds so that
server can effectively process them.
– The name of a model of the input device. There are several types of input
devicestheTargetprogrammayuse,likestdin,commandlineoptions,disk,
network. FIzzer does not work with physical devices. A model of a device
must always be provided (implemented). There can be more models for one
device.Butcurrently,thereisonlyonemodelforstdindeviceimplemented
in FIzzer. This model is initialized with a sequence of bytes, i.e., with the
input generated by the Server. Reading from stdin 7 consumes bytes from
the sequence. When there is not enough bytes in the sequence to be read,
then the sequence is automatically extended by bytes of a predefined value,
which can be either 0 or 85 (there is nothing fundamental behind choice
of the values). The name of the model thus currently primarily determines
which of the value should be used.
– Asequenceofbytestobeusedfortheinitializationofthemodeloftheinput
device. That is the input generated by the Server.
Next, the Target reads the Config from the shared memory, creates the
model of the device, initializes it with the sequence of input bytes, clears the
shared memory, and calls the original main function (see Sec.2). Whenever a
monitoring code is executed, it tries to append the collected data to the shared
memory.TheexecutionoftheTargetalwaysterminates,whichhappensinthese
situations:
– The Target returns from the original main function. That is the normal
termination, which the Target records in the shared memory by setting the
termination flag to NORMAL.
7 Currently, reading from stdin can only be done via calls to __VERIFIER_nondet_
functions (see Sec.2).

| Gray-Box | Fuzzing | via Gradient |     | Descent | and | Boolean Expression |     | Coverage |     | 7   |
| -------- | ------- | ------------ | --- | ------- | --- | ------------------ | --- | -------- | --- | --- |
– The Target executable crashes. This situation is recognized as follows. The
Targetsetstheterminationflagtoaninvalidvaluebeforecallingtheoriginal
| main | function | and to | NORMAL | once | the execution |     | returns | from the | call. | The |
| ---- | -------- | ------ | ------ | ---- | ------------- | --- | ------- | -------- | ----- | --- |
ServeralwaysgetstheexitcodefromtheTargetprocess.Ifthetermination
| flag is | invalid, | then the | Server | sets | the | termination | flag | based | on the | exit |
| ------- | -------- | -------- | ------ | ---- | --- | ----------- | ---- | ----- | ------ | ---- |
8
| code | to either | CRASH or | NORMAL |     | .   |     |     |     |     |     |
| ---- | --------- | -------- | ------ | --- | --- | --- | --- | --- | --- | --- |
– The time reserved by the Server for the execution of the Target was ex-
ceeded.InthatcasetheServerforcefullyterminates(kills)theTarget,and
| sets the | termination | flag | to  | TIMEOUT. |     |     |     |     |     |     |
| -------- | ----------- | ---- | --- | -------- | --- | --- | --- | --- | --- | --- |
– Any of the limits passed to the Target in the Config was exceeded. Then
theexecutionsoftheTargetisforcefullyterminatedfromwithintheTarget
| by exit(0)                       |     | right after | setting | the        | termination | flag               | in  | the shared    | memory    |     |
| -------------------------------- | --- | ----------- | ------- | ---------- | ----------- | ------------------ | --- | ------------- | --------- | --- |
| to BOUNDARY_CONDITION_VIOLATION. |     |             |         |            | We          | do not distinguish |     | what          | condition |     |
| was actually                     |     | violated.   |         |            |             |                    |     |               |           |     |
| The termination                  |     | flag        | sits at | a reserved | location    | in                 | the | shared memory |           | and |
represents an important information of the Results passed from the Target to
the Server via the shared memory, see Fig.3 (top). Besides the termination flag
| the following | data | are in the | Results |     | (in the | shared | memory): |     |     |     |
| ------------- | ---- | ---------- | ------- | --- | ------- | ------ | -------- | --- | --- | --- |
– A sequence of bytes read by the target during the execution. The sequence
| always      | starts | by the input | bytes | passed |           | from the | Server | to the         | Target | via |
| ----------- | ------ | ------------ | ----- | ------ | --------- | -------- | ------ | -------------- | ------ | --- |
| the Config, |        | but it can   | be of | any    | length up | to the   | limit  | in the Config. |        |     |
– Asequenceoftypesassignedtorangesofbytesinthesequenceabove.Atype
| can be    | one               | of the following |     | BOOLEAN,                             | UINTN, | SINTN,       | FLOATM, |     | UNTYPEDN, |     |
| --------- | ----------------- | ---------------- | --- | ------------------------------------ | ------ | ------------ | ------- | --- | --------- | --- |
| whereN    | ∈{8,16,32,64}andM |                  |     | ∈{32,64}.Forexample,ifduringTarget’s |        |              |         |     |           |     |
| execution | there             | were called      |     | functions                            | (in    | that order): |         |     |           |     |
__VERIFIER_nondet_char();
__VERIFIER_nondet_float();
__VERIFIER_nondet_short();
| then          | there      | will be seven | bytes    | in          | the input      | bytes     | sequence. | The       | first        | byte |
| ------------- | ---------- | ------------- | -------- | ----------- | -------------- | --------- | --------- | --------- | ------------ | ---- |
| will be       | associated | with          | the      | type SINT8, | the            | range     | of the    | next four | bytes        | will |
| be associated |            | with FLOAT32, |          | and         | the last       | two bytes | with      | SINT16.   |              |      |
| Remark        | 1.         | The types     | UNTYPEDN |             | are introduced | for       | cases     | when      | type assign- |      |
mentisnotasstraightforwardaswiththeuseofthefunctions__VERIFIER_nondet_,
| i.e., when | the | assignment | becomes |     | unknown. |     |     |     |     |     |
| ---------- | --- | ---------- | ------- | --- | -------- | --- | --- | --- | --- | --- |
– AsequenceofrecordscapturinginformationaboutevaluationofallBoolean
| instructions |     | along the | executed | path | in  | the Target. | The | order | of records |     |
| ------------ | --- | --------- | -------- | ---- | --- | ----------- | --- | ----- | ---------- | --- |
matchestheorderofthecorrespondingBooleaninstructionsexecutedalong
thepath.Wedenotethesequenceastheexecutiontrace.Eachrecordinthe
| trace | consist | of the following |             | information: |              |     |     |     |     |     |
| ----- | ------- | ---------------- | ----------- | ------------ | ------------ | --- | --- | --- | --- | --- |
| • The | unique  | ID of            | the Boolean |              | instruction. |     |     |     |     |     |
8 That is for treating forceful termination by calling exit(0) as NORMAL termination.

8 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
• A hash of the calling context. We use context sensitivity in order to
reducethenumberofcaseswheretheServerwronglyconcludesthatall
reachable Boolean instructions were already considered in the analysis.
Forexample,letussupposeweignorethecallingcontextandweanalyze
the following program:
void foo(int x) { if (x < 0) abort(); }
bool x,y;
... // Read input to variables x and y.
foo(x);
foo(y);
... // A lot of code is here.
If the Server generated, for instance, an input x <- 1, y <- -1, then
theBooleaninstructioninfoowillbecovered.Sincenootherinstruction
wasdiscovered,theServerconcludesthereisnootherreachableBoolean
instruction in the program to cover. In contrast, the context sensitivity
allows us to distinguish the Boolean instruction in each of the two calls
of foo, leaving the Boolean instruction in the second call uncovered.
We in fact do not need to know exactly what functions are on the call
stack.WeonlywanttodistinguishBooleaninstructionsbythecontexts.
So,wejustcomputea32-bithashfromIDs(seecallsiteinstrumentation
in Sec.2) of functions on the stack. 9
We denote the unique ID of the Boolean instruction with the context
hash as an execution ID.
• The result of the evaluation of the Boolean instruction, denoted as di-
rection. 10
• The double value, denoted as a value of the branching function, com-
puted by the monitoring code from the syntactical structure of the
Boolean instruction. For example, for *cmp instructions the branching
function is (double)l−(double)r, where l and r are registers of a nu-
meric type appearing as arguments in a predicate (for details see Sec.2).
• Booleanvaluedeterminingwhetherthereappearsaxorinstructionany-
where before the comparison instruction in the same basic block or not.
• Thecountofinputbytesreadfromthestartofthetrace(beforethefirst
record) up to this record.
The elements of the sequences forming the Results are in fact interleaved
in the shared memory. They appear there in the order as the monitoring code
in the Target wrote them to the shared memory. Individual sequences are thus
constructed in the Server during a sequential scan of the elements the shared
memory.
9 Duetorecursivefunctionswerestrictedcomputationofcallingcontexthashonlyup
to a predefined call stack size. For larger context the hash thus remains the same.
10 WewillseeinSec.4thatweconstructnodesofabinarytreefromtracerecordsand
the direction identifies the true or false successor node in the tree corresponding
to the successor record in the trace. I.e., it is the “direction” to the successor.

| Gray-Box | Fuzzing    | via | Gradient       | Descent | and     | Boolean | Expression | Coverage |       | 9      |
| -------- | ---------- | --- | -------------- | ------- | ------- | ------- | ---------- | -------- | ----- | ------ |
| FIzzer   | implements |     | an alternative |         | version | of the  | fuzzing    | loop     | which | is de- |
picted at Fig.3 (bottom). We see that, in contrast to the original version of the
fuzzing loop, the Server is replaced by a Client binary. The Client indeed
implementsexactlythesameprocedureofcommunicationwiththeTarget.The
Target is thus unable to tell whether it communicates with the Server or the
Client. From the Server’s point of view, the Client behaves like the Target.
Only the communication medium is different. In summary, the alternative ver-
sionofthefuzzingloopisaslowerimplementationoftheoriginalversion,because
| the data flow   | through |         | two media, | namely | the | network | and        | shared | memory.    |     |
| --------------- | ------- | ------- | ---------- | ------ | --- | ------- | ---------- | ------ | ---------- | --- |
| The alternative |         | version | however    | can    | be  | used    | in a setup | which  | can poten- |     |
tially improve the overall performance. Observe in the Fig.3 (bottom) that the
Server can simultaneously instruct multiple Clients on multiple computers 11
toexecutetheirTargets.Althoughthesimultaneousexecutionscouldbeimple-
mented also in the original version of the fuzzing loop, its practical applicability
| is considerably |            | reduced | due to | limited | resources | of  | a single | computer. |     |     |
| --------------- | ---------- | ------- | ------ | ------- | --------- | --- | -------- | --------- | --- | --- |
| 4 Input         | generation |         |        |         |           |     |          |           |     |     |
The goal of input generation is to produce a shortest sequence of inputs for the
Targetwhoseexecutionscumulativelycoversthemaximumof Booleaninstruc-
| tions in the    | Target. |            |            |     |           |        |           |        |            |     |
| --------------- | ------- | ---------- | ---------- | --- | --------- | ------ | --------- | ------ | ---------- | --- |
| The Server      |         | initially  | generates  | the | empty     | input. | All other | inputs | are gener- |     |
| ated by exactly |         | four input | generation |     | analyses: |        |           |        |            |     |
– Sensitivity:identifiesasubsetofinputbits,calledsensitivebits,tobefocused
| on by | other | analyses. |     |     |     |     |     |     |     |     |
| ----- | ----- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
– Bitshare: reuse of sensitive bits in previously generated inputs in the con-
| struction | of  | new inputs. |     |     |     |     |     |     |     |     |
| --------- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
– Typed minimization: a gradient descent on sensitive bits forming variables
| of a known |     | numeric | type. |     |     |     |     |     |     |     |
| ---------- | --- | ------- | ----- | --- | --- | --- | --- | --- | --- | --- |
– Minimization: a gradient descent on sensitive bits forming variables whose
| numeric | type | is not | known. |     |     |     |     |     |     |     |
| ------- | ---- | ------ | ------ | --- | --- | --- | --- | --- | --- | --- |
They are described in details later in Sec.5. Exactly one of them is active at
a time. Only the active analysis generates inputs. Other analyses wait for their
activation.Onceananalysisisactivated,itstaysactiveuntiliteitherdeactivates
itselforitisforcefullydeactivated.Ananalysisdeactivatesitself,whenitsinput
generation strategy is finished. An analysis is deactivated forcefully, when its
| goal was achieved |     | before | the input | generation |     | strategy | is finished. |     |     |     |
| ----------------- | --- | ------ | --------- | ---------- | --- | -------- | ------------ | --- | --- | --- |
Thegoalofallanalyses,exceptthesensitivity,alwaysistoinverttheevalua-
tion result of a particular Boolean instruction corresponding to a certain record
in an execution trace. The sensitivity analysis has a different goal - to compute
11 The alternative version of the fuzzing loop is currently only in a prototype stage
| where the | loop | works | only on | a localhost | with | one | Client. |     |     |     |
| --------- | ---- | ----- | ------- | ----------- | ---- | --- | ------- | --- | --- | --- |

| 10  | Martin | Jonáš , | Jan Strejček | , Marek | Trtík, | and Lukáš | Urban |     |
| --- | ------ | ------- | ------------ | ------- | ------ | --------- | ----- | --- |
sensitive bits. These bits are essential for all other analyses. Therefore, we al-
wayswanttocompleteitsinputgenerationstrategy,i.e.,thesensitivityanalysis
| is never | forcefully  | deactivated. |                 |              |     |         |          |               |
| -------- | ----------- | ------------ | --------------- | ------------ | --- | ------- | -------- | ------------- |
| Once     | an analysis |              | is (forcefully) | deactivated, |     | another | one must | be activated. |
Thatisaresponsibilityofananalysisselectionstrategy.Thegoalofthisstrategy
is to maximize coverage of Boolean instructions. It approaches the problem
such that it builds a short-term goals for the four input generator analyses
and activates the analyses for these goals. The ultimate long-term goal with
the maximal coverage is thus achieved indirectly - it is approached by solving
a sequence of short term goals. In the heart of the building short-term goals
there is a maintenance of and a search in core data structures constructed from
the data accepted by the Server from the Target after each its execution. We
| discuss | the details | of  | the selection | strategy | later       | in Sec.6.  |          |           |
| ------- | ----------- | --- | ------------- | -------- | ----------- | ---------- | -------- | --------- |
| In each | iteration   | of  | the fuzzing   | loop     | (see Sec.3) | the active | analysis | generates |
exactly one input for the Target. The Server then accepts back an input x
(which is the generated input, possibly extended or truncated), the sequence of
types t (logically splitting x into sequences of bits and assigning them types),
and an execution trace T. These data are used for construction of core data
structuresessentialforallanalyses.Itisthusfirstnecessarytounderstandthese
core data structures and how they are built from the accepted data. That is the
| subject | of the | following | subsections. |     |     |     |     |     |
| ------- | ------ | --------- | ------------ | --- | --- | --- | --- | --- |
Notation: If S is a sequence, then |S| denotes the number of elements in the
sequence and S[i] denotes the i-th element. We also use Python-like syntax for
denoting subsequences, e.g., S[k : l], S[: l], S[k :], denote sequences of elements
fromSatindicesk,...,l−1,0,...,l−1,k,...,|S|−1,respectively.Iftheelement
hassomestructure,thenweuse“dot” notationtoaccessthefields.Forinstance,
if T is an execution trace and 0 ≤ i < |T| is an index to T, then following are
| all field | of yjr | record | T[i]: |     |     |     |     |     |
| --------- | ------ | ------ | ----- | --- | --- | --- | --- | --- |
– T[i].id is the execution ID of the Boolean instruction corresponding to T[i],
| – T[i].f         | is the | double | value  | of the branching  |     | function,      |              |     |
| ---------------- | ------ | ------ | ------ | ----------------- | --- | -------------- | ------------ | --- |
| – T[i].direction |        | is the | result | of the evaluation |     | of the Boolean | instruction, |     |
– T[i].xor indicates whether a xor instruction appears before the Boolean
| instruction |     | in the | same basic | block. |     |     |     |     |
| ----------- | --- | ------ | ---------- | ------ | --- | --- | --- | --- |
– T[i].nbytes is the number of input bytes read from the begin of the trace
| (before | T[0]) | up to | T[i], |     |     |     |     |     |
| ------- | ----- | ----- | ----- | --- | --- | --- | --- | --- |
Further, fields can be nested, for which we also use the same notation, e.g.,
T[i].id.uid and T[i].id.ctx are the unique ID of a Boolean instruction and the
context hash, respectively (see Sec.3). Finally, if we speak about a field of some
record in general, we omit the record, e.g., we just write id.uid when we speak
| about the     | unique | ID   | of a Boolean | instruction. |     |     |     |     |
| ------------- | ------ | ---- | ------------ | ------------ | --- | --- | --- | --- |
| 4.1 Execution |        | tree |              |              |     |     |     |     |
Attheheartoftheinputgenerationthereisabinaryrootedtree,calledexecution
tree.Initiallyitisempty.EachnodeN inthetreecorrespondstoanexecutionof

Gray-Box Fuzzing via Gradient Descent and Boolean Expression Coverage 11
a Boolean instruction alongsome programpath, for whichthe Server accepted
an execution trace. Since a Boolean instruction can be evaluated to two values
true or false, the node may have two successors, called true-successor and
false-successor. Since the nodes are connected via edges, the node may also
have two edges, called true-edge and false-edge. The edges carry labels. We
discuss their purpose later.
→−
Notation: LetN beanodeoftheexecutiontree.Then N isasequenceofnodes
in the tree from the root node to N (including N). The depth of N in the tree
→−
is the count of edges between nodes in N and we denote it as dN. Clearly,
→− →− →−
dN = |N|−1, N[0] is the root node, and N[dN] is N. When b is a Boolean
value, i.e., true or false, then N.successor[b] is the b-successor node of N and
N.label[b] is the label of the b-edge of N. And N.parent is the parent node of N
in the tree. The parent of the root node is null.
In the end of each iteration of the fuzzing loop the tree is updated according
to data accepted by the Server (see Sec.3), which is the termination flag, an
input x, types t, and a trace T. During this process the tree may be extended
(new nodes are created) and some existing nodes may be updated (their fields).
Updating tree’s shape A trace T accepted by the Server is mapped to the
nodes of the execution tree such that T[0] is mapped to the root node R, then
T[1] is mapped to R.successor[T[0].direction], and so on. When T[i] is mapped
to a node N, i+1 < |T|, and N.successor[T[i].direction] = null, then the
missing successor node is created and inserted to the tree.
LabelsofedgesinN.labeldescribethetransitiontosuccessornodes,including
the case the successors are missing. Let b is a Boolean value. If N.label[b] is
– NOT_VISITED, then N.successor[b] = null. This indicates that there is no
trace among all traces accepted by the Server so far, which has a record
T[i] mapped to N such that T[i].direction=b.
– END_EXCEPTIONAL, then N.successor[b] = null. This indicates that there
was at least one trace T accepted by the Server, which has the last record
T[dN]mappedtoN,T[dN].direction=b,andalsotheterminationflag(see
Sec.3) of the execution was set to CRASH. Further, there is no trace among
all traces accepted by the Server, for which N.label[b] would be set to any
of the values listed below.
– END_NORMAL, then N.successor[b]=null. The indication is the same as for
the previous label, except the termination flag has the value NORMAL.
– VISITED, then N.successor[b] points to a valid node. That indicates there
was at least one trace T accepted by the Server such that |T| > dN +1,
T[dN] is mapped to N, and T[dN].direction=b.
The values of labels are ordered from top down, i.e., NOT_VISITED < ··· <
VISITED. It favours longer execution paths and also normal paths over crushes.
That maximizes the potential to cover Boolean instructions deeper in the code.

| 12  | Martin | Jonáš , | Jan Strejček | , Marek | Trtík, | and Lukáš | Urban |     |     |
| --- | ------ | ------- | ------------ | ------- | ------ | --------- | ----- | --- | --- |
AtreenodeiscreatedwithNOT_VISITEDforbothlabels.Thelabelsmaychange
duringtheanalysis,namelytoincreaseinthatorder.Forexample,ifN.label[b]=
END_EXCEPTIONAL and the Server accepts a trace T such that |T| > dN +1,
T[dN]ismappedtoN,andT[dN].direction=b,thenN.label[b]willbechanged
| to VISITED | and | N.successor[b] |     | will point | to  | a newly created | node. |     |     |
| ---------- | --- | -------------- | --- | ---------- | --- | --------------- | ----- | --- | --- |
Purpose of branching functions Let us consider the following C program
| char | x =     | __VERIFIER_nondet_char(); |      |     |     |     |     |     |     |
| ---- | ------- | ------------------------- | ---- | --- | --- | --- | --- | --- | --- |
| ...  | // some | code                      |      |     |     |     |     |     |     |
| bool | bi      | = x >                     | 254; |     |     |     |     |     |     |
ThebranchingfunctionoftheBooleaninstructioncorrespondingtothevariable
biisf(x)=(double)x−254.IfwewanttocovertheBooleaninstruction,then
we should attempt to find some inputs u and v such that f(u) and f(v) have
| opposite | signs. | We should | first | realize | the following: |     |     |     |     |
| -------- | ------ | --------- | ----- | ------- | -------------- | --- | --- | --- | --- |
– f maynotbelinear,becausexmaynotbeanindependentvariable.Indeed,
| the           | code abbreviated |        | by  | “...” could   | modify | x arbitrarily. | It        | means | f is in  |
| ------------- | ---------------- | ------ | --- | ------------- | ------ | -------------- | --------- | ----- | -------- |
| fact          | unknown          | to us. | So, | the best      | thing  | we can do is   | to sample | the   | function |
| by generating |                  | inputs | x   | and observing | f(x).  |                |           |       |          |
– Randomsamplingoftheinputdomainmayeasilybeineffectiveforobtaining
theinputsuandv.Thatcanbeseeninourexampleevenifthecodein“...”
| does | not affect | x.  | Indeed, | there is | only | one input evaluating |     | f to a | positive |
| ---- | ---------- | --- | ------- | -------- | ---- | -------------------- | --- | ------ | -------- |
number.
– Since we search for u and v producing f(u) and f(v) of opposite signs,
| random  | sampling |                 | of the | input domain         | in       | a neighborhood | around       | the         | global  |
| ------- | -------- | --------------- | ------ | -------------------- | -------- | -------------- | ------------ | ----------- | ------- |
| minimum |          | of the function |        | |f(x)| may           | actually | be effective.  | That         | can         | be seen |
| in our  | example, | when            | the    | code in              | “...”    | does not       | affect x. If | we randomly |         |
| sample  | the      | inputs          | from   | a small neighborhood |          | around         | the global   | minimum     |         |
254,thenourchancesofgeneratingthedesiredinputsuandvquicklywillbe
| considerably |     | higher | (depending | on  | the | size of the neighborhood |     | we  | sample |
| ------------ | --- | ------ | ---------- | --- | --- | ------------------------ | --- | --- | ------ |
from).
The purpose of a branching function f(x) is thus to allow us quickly converge
to a neighborhood around the global minimum of the function |f(x)|, where we
can then effectively obtain the desired inputs via random sampling form the
neighborhood. We use the gradient descent as the convergence method, where
we compute partial derivatives numerically, since the function f is unknown.
Instead of detecting whether we already are in a neighborhood for an effective
randomsamplingornotwerathertakemultiplesamplesineachgradientdescent
step.Thiswaywealsotakeseveralsamplesfromtheneighborhoodintheendof
the descent, in a price of taking samples outside the neighborhood. We discuss
| details later | in  | Sec.5.3. |     |     |     |     |     |     |     |
| ------------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
Updatingcontentofnodes LetusconsideranodeN.Duringtheanalysisthe
Server may accept several inputs x ,x ,...,x , types t ,t ,...,t , and traces
|     |     |     |     | 1   | 2   | n   | 1 2 | n   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

| Gray-Box | Fuzzing | via | Gradient | Descent | and Boolean | Expression | Coverage |     | 13  |
| -------- | ------- | --- | -------- | ------- | ----------- | ---------- | -------- | --- | --- |
T 1 ,T 2 ,...,T n , where the records at the index dN are all mapped to N. The
values f in these records may be different. Which of the triples (x ,t ,T ) we
|     |     |     |     |     |     |     |     | j j | j   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
needforaneffectivecoverageofN.id?Sincewewanttoapproachaneighborhood
around the global minimum of |f(x)|, only one tuple seems to be sufficient - the
onewiththesmallest|T [dN].f|.However,itisquitecommonthatsamebit(s)in
j
theinputx j affectvaluesf inmultiplerecordsinT j .Wemustthereforeconsider
| all predecessors |     | of N. So, | we use | a triple | with the | smallest | value |     |     |
| ---------------- | --- | --------- | ------ | -------- | -------- | -------- | ----- | --- | --- |
dN
(cid:88)
|     |     |     | w   | (T )= | T [i].f2 |     |     |     |     |
| --- | --- | --- | --- | ----- | -------- | --- | --- | --- | --- |
|     |     |     |     | N j   | j        |     |     |     |     |
i=0
Thesquaresofvaluesinthesumincrease(emphasize)theimpactoflargervalues
| and they | also handle | negative |     | values. |     |     |     |     |     |
| -------- | ----------- | -------- | --- | ------- | --- | --- | --- | --- | --- |
Notation: Each node N has also fields N.x, N.t, N.T used for storing values
x , t , T , which give the smallest value of w . We further abbreviate accesses
| j j | j   |     |     |     | N   |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
to fields of N.T[dN] such that we omit “.T[dN]”, e.g., instead of N.T[dN].f we
write just N.f. And finally, we say that an execution trace T is mapped to N
→−
(or N), if |T| > dN and for each index 0 ≤ i ≤ dN and 0 ≤ j < dN we have
→−
T[i].id = N.T[i].id and T[i].direction = N.T[i].direction (or T[i].id = N[i].id,
|                |            |             |       | →−         | →−                               |     |        |          |      |
| -------------- | ---------- | ----------- | ----- | ---------- | -------------------------------- | --- | ------ | -------- | ---- |
| T[0] is mapped |            | to the root | node, | N[j+1]=    | N[j].successor[T[j].direction]). |     |        |          |      |
| Since          | the Server | generates   |       | the inputs | sequentially                     |     | (we do | not have | them |
all at once), the field x, t, T may be changed during the analysis. Namely, if a
new triple (x ,t ,T ) is accepted by the Server such that T [dN] is
|     | n+1 | n+1 | n+1 |     |     |     |     | n+1 |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
mapped to the node N, and w (T ) < w (N.T), then we set all fields N.x,
|          |           |       |         | N n+1     | N             |     |     |     |     |
| -------- | --------- | ----- | ------- | --------- | ------------- | --- | --- | --- | --- |
| N.t, N.T | to values | x n+1 | , t n+1 | , T n+1 , | respectively. |     |     |     |     |
Therearemoreinformationstoredineachnode.However,thesefieldsarere-
latedtoindividualinputgenerationanalysesandtheanalysisselectionstrategy.
| So, we introduce |            | these | fields later. |     |     |     |     |     |     |
| ---------------- | ---------- | ----- | ------------- | --- | --- | --- | --- | --- | --- |
| 5 Input          | generation |       | analyzes      |     |     |     |     |     |     |
Wealreadyknowthereareexactlyfouranalysesresponsibleforinputgeneration
(sensitivity, bitshare, and two minimization analyzes); exactly one of them is
active at time; an analysis may stay active over several iterations of the fuzzing
loop; the active analysis generates a single input in each iteration of the fuzzing
| loop and | also processes |          | the corresponding |      | trace       | in the | same iteration. |          |     |
| -------- | -------------- | -------- | ----------------- | ---- | ----------- | ------ | --------------- | -------- | --- |
| The      | sensitivity    | analysis | differs           | from | other three | in the | sense that      | its goal | for |
any node N in the execution tree is to identify a subsets of bits in the input
N.x, called sensitive bits. Other analyses then focus only on the sensitive bits,
which considerably improves the performance of these analyses. In other words,
the goal of sensitivity analysis is to boost effectivity of other analyses rather
thanaimingtoimprovingthecoverageof Booleaninstructions.Thatisalsothe
reason why we always start sensitivity analysis on N before any other analysis.

| 14 Martin | Jonáš , Jan  | Strejček | , Marek | Trtík,         | and Lukáš | Urban           |
| --------- | ------------ | -------- | ------- | -------------- | --------- | --------------- |
| The goal  | of all other | analyses | is to   | find a missing | successor | node of a given |
node in the execution tree. More precisely, given a node N in the execution tree
| and a Boolean | value b | such that |     |     |     |     |
| ------------- | ------- | --------- | --- | --- | --- | --- |
– thesetofsensitivebitsofN detectedbythesensitivityanalysisisnotempty,
| – N.label[b]=NOT_VISITED, |     |     | i.e., the | b-successor | of N is | not in the tree yet, |
| ------------------------- | --- | --- | --------- | ----------- | ------- | -------------------- |
the goal of all other analyses is to find an input x so that the obtained trace T
→−
| is mapped | to N, |T|>dN | +1    | and T[dN].direction=b. |        |              |              |
| --------- | ------------ | ----- | ---------------------- | ------ | ------------ | ------------ |
| Observe   | that neither | these | three analyses         | aiming | to improving | the coverage |
of Boolean instructions. Indeed, the analysis can be asked to find b-successor of
a node N, whose corresponding instruction with ID N.id was already
Boolean
covered. The only analysis aiming at the coverage of Boolean instructions is
the analysis selections strategy, whose task is to choose a node N in the tree
and start one of our four analyses on it, whenever the previously active analysis
| becomes inactive. | We discuss |     | details of | the selection | strategy | later in Sec.6. |
| ----------------- | ---------- | --- | ---------- | ------------- | -------- | --------------- |
Notation Observe that each analysis is activated with a certain node N in the
execution tree. We will see later (namely in Sec.6) that we need to track the
information what analysis was already applied to what node and when. So, we
introducetoeachnodeN Booleanfields(flags)N.sa,N.ba,andN.maindicating
whether the sensitivity analysis, bitshare analysis, and minimization analysis
respectively were already applied to the node or not. Also notice that we do
notdistinguishbetweenthetwominimizationanalyses.Thatisbecauseatmost
one of them can be run on a given node. In order to keep track of when the
analyses were applied we introduce integer fields N.sn, N.bn, and N.mn which
we set to the number of the fuzzing loop iteration. So, whenever an analysis
y ∈ {s,b,m} is (forcefully) deactivated, then the filed ya is set to true and yn
is set to the current fuzzing loop iteration number. Notice that we record the
last iteration number, in which the analysis was active (which is typically after
tens or hundreds of subsequent iterations). In general, beside the node N, the
fields are set in all nodes in the tree which were changed by the analysis since
itsactivation.Thesensitivityanalysisoftencomputes(updates)sensitivebitsof
several nodes in the tree along the path from the root node to N. So, fields of
all these nodes are thus set. All other analyses modify only the node N, so only
| fields of N | are updated. |     |     |     |     |     |
| ----------- | ------------ | --- | --- | --- | --- | --- |
Notation We further use the field N.fn to store the number of the fuzzing loop
| iteration, | when the field | N.f(x) | was set | for the | last time. |     |
| ---------- | -------------- | ------ | ------- | ------- | ---------- | --- |
Fuzzing loop integration: In this paper we present the analyses from the algo-
rithmic point of view. In our implementation the algorithms have a different
structure. The actual computation is of course the same. The reason for the
difference is the integration of the algorithms to the fuzzing loop (see Sec.3). In
| each iteration | of the fuzzing | loop | two method |     | of the analysis | are called: |
| -------------- | -------------- | ---- | ---------- | --- | --------------- | ----------- |
– generate_input: The analysis is supposed to return an input for which the
| Target | will be executed. |     |     |     |     |     |
| ------ | ----------------- | --- | --- | --- | --- | --- |

| Gray-Box | Fuzzing | via | Gradient | Descent | and | Boolean | Expression | Coverage | 15  |
| -------- | ------- | --- | -------- | ------- | --- | ------- | ---------- | -------- | --- |
– process_results: The analysis is supposed to process the obtained execu-
| tion | trace | T.  |     |     |     |     |     |     |     |
| ---- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
The algorithms thus contain auxiliary variables providing a bookkeeping of of
its the current state so that they can proceed further within calls to the two
| functions | above. |     |     |     |     |     |     |     |     |
| --------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
Fast execution cache: Input generation algorithms of some analyses discussed
belowmayoccasionallygenerateaninputalreadygeneratedbefore.Ratherthan
complicating the implementation we introduced a cache to these analyses. The
| cache work | as  | a map | from 64-bit | hashes | of  | all generated | inputs | to  | the |
| ---------- | --- | ----- | ----------- | ------ | --- | ------------- | ------ | --- | --- |
double
valuesoftheconsideredbranchingfunction.Anygeneratedinputsisfirstlooked
| up in the       | cache | and it   | is executed | by  | the Target | only | on cache | miss. |     |
| --------------- | ----- | -------- | ----------- | --- | ---------- | ---- | -------- | ----- | --- |
| 5.1 Sensitivity |       | analysis |             |     |            |      |          |       |     |
The purpose of this analysis is to boost effectivity of other three analyses.
Namely, given a tree node N, its goal is to compute a set of indices of those
bits in N.x having an impact on N.f. We call these bits as sensitive bits. The
other analyses may thus focus only on the sensitive bits, i.e., safely ignore all
others.
| Since    | the | formal definition         |          | of sensitive | bits    | is not    | intuitive, | we start | with an |
| -------- | --- | ------------------------- | -------- | ------------ | ------- | --------- | ---------- | -------- | ------- |
| example. | Let | us consider               | this     | C program    |         |           |            |          |         |
| char     | c = | __VERIFIER_nondet_char(); |          |              |         | //        | read 8     | bits     |         |
| c =      | c & | 7; //                     | Set bits | at           | indices | 0,1,2,3,4 |            | to 0.    |         |
bool bi0 = ((c ^ 7) * (c ^ 1)) != 0; // Boolean instruction; ID=0
| if   | (bi0) | return; | //    | Return  | if c is      | neither | 7    | nor 1. |     |
| ---- | ----- | ------- | ----- | ------- | ------------ | ------- | ---- | ------ | --- |
| bool | bi1   | = c >   | 2; // | Boolean | instruction; |         | ID=0 |        |     |
From the second line we can immediately conclude that input bits at indices
0,1,2,3, and 4 may not be sensitive (no impact on branching functions), because
they are cleared after read. There are two Boolean instructions in the program;
they correspond to the variables bi0 and bi1. They both operate on inputs, all
| with the | size      | m=8 bits. |     |       |        |           |     |                |         |
| -------- | --------- | --------- | --- | ----- | ------ | --------- | --- | -------------- | ------- |
| Let      | us decide | whether   | the | input | bit at | the index | s = | 7 is sensitive | for the |
first Boolean instruction or not. For m = 8 we have exactly 256 possible in-
puts x = 0,...,x = 255 for which the execution reaches and evaluates the
| 0   |     | 255 |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Boolean instruction. The evaluation is captured in the record at index d=0 in
allexecutiontracesT ,...,T correspondingtotheinputs.Wecansplitallpairs
|     |     |     | 0 255 |     |     |     |     |     |     |
| --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- |
(x i ,T i ) into a disjoint sets X f according to the equality of the values T i [d].f,
i.e.,twopairs(x ,T )and(x ,T )areinthesameset,iffT [d].f =T [d].f.Since
|     |     | i i |     | j j |     |     | i   | j   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
the branching function can evaluate only to four values 0, 7, 8, and 15, there
will be four corresponding sets of the pairs. Intuitively, a bit at the index s
should be sensitive, if there exist pairs (x ,T ) and (x ,T ) from different sets
|     |     |     |     |     | i   | i   | j j |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
suchthatx i [s]̸=s j [s].Forinstance,inputs(x 0 ,T 0 )∈X 7 and(x 1 ,T 1 )∈X 0 and
x [s] = 0 ̸= 1 = x [s]. So, the bit at the index s should be sensitive. Although
| 0   |     | 1   |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

| 16  | Martin | Jonáš | ,   | Jan Strejček | , Marek | Trtík, and | Lukáš Urban |     |     |
| --- | ------ | ----- | --- | ------------ | ------- | ---------- | ----------- | --- | --- |
this is the result we want, the condition we formulated is too weak, because the
bit at the index 4 would be sensitive too (c.f., (x ,T ) ∈ X and (x ,T ) ∈ X
|     |     |     |     |     |     | 0   | 0   | 7   | 9 9 0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- |
andx [4]=0̸=1=x [4]).Therefore,wemustrestrictoursearchtothe“closest”
|        | 0    |           |     | 9         |              |             |           |     |     |
| ------ | ---- | --------- | --- | --------- | ------------ | ----------- | --------- | --- | --- |
| inputs | from | different |     | sets. For | that can use | the Hamming | distance: |     |     |
Letusconsidertwoinputsuandvsuchthat|u|=|v|.TheHammingdistance
| H(u,v) |     | is the number |     | of all indices | 0≤i<|u| | where | u[i]̸=v[i]. |     |     |
| ------ | --- | ------------- | --- | -------------- | ------- | ----- | ----------- | --- | --- |
ObservethatH(x ,x )=1whileH(x ,x )=2.Usingboth,theintuitivecondi-
|     |     |     | 0 1 |     | 0   | 9   |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
tionandtheHammingdistance,wecanfurtherdecidethatbitsatindices5and
6 are also sensitive (c.f., (x ,T ),(x ,T )∈X and H(x ,x )=H(x ,x )=1)
|       |        |       |          | 4              | 4 2 2   | 15          | 0              | 4   | 0 2      |
| ----- | ------ | ----- | -------- | -------------- | ------- | ----------- | -------------- | --- | -------- |
| while | all    | other | bits are | not sensitive. |         |             |                |     |          |
|       | Let us | now   | focus on | the second     | Boolean | instruction | (corresponding |     | to bi1). |
This instruction executed only for 64 of all 256 inputs above. In all of the corre-
sponding traces the instruction corresponds to records at the index d = 1. For
32 inputs x ,x ,x ,...,x the instruction is evaluated to false and for all
|     |     | 1   | 9 17 | 249 |     |     |     |     |     |
| --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- |
i we have T i [d].f = −1. And for 32 inputs x 7 ,x 15 ,x 23 ,...,x 255 the instruction
is evaluated to true and for all i we have T [d].f = 5. So, we have two sets
i
X and X . Observe, that for any (x ,T ) ∈ X and (x ,T ) ∈ X we have
| −1  |     | 5   |     |     | i   | i −1 |     | j j | 5   |
| --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- |
H(x ,x ) ≥ 2. Also, only bits at indices 5 and 6 satisfy both conditions, i.e.,
i j
they are sensitive (c.f., (x ,T ) ∈ X and (x ,T ) ∈ X and x [5] ̸= x [5] and
|     |     |     |     | 1 1 | −1  | 7 7 | 5   | 1   | 7   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
x 1 [6] ̸= x 7 [6])). Observe, the minimal Hamming distance between sets defines
also the minimal number of bits considered as sensitive simultaneously. We are
| ready | to                       | define | sensitive | bits formally. |        |                                |     |     |     |
| ----- | ------------------------ | ------ | --------- | -------------- | ------ | ------------------------------ | --- | --- | --- |
|       | Let0<mand0≤dbeintegers,x |        |           |                | ,...,x | befinitesequencesofallpossible |     |     |     |
|       |                          |        |           |                | 1      | n                              |     |     |     |
inputssuchthat|x |=mandT ,...,T befinitesequencesofthecorresponding
|     |     |     | i   |     | 1 n |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
traces such that for all integers 0<i,j <m, 0≤k ≤d, and 0≤l <d we have
|T i |≥d,T i [k].id=T j [k].id,andT i [l].direction=T j [l].direction.Inotherwords,
foreachinputx theTargetexecutesexactlythesamesequenceofd+1Boolean
i
instructions (we ignore the suffixes of the traces T [d+1:]). The bit at an index
i
0≤s<missensitiveatthetraceindexd,iffthereexisttwopairs(x ,T )∈X
|     |     |     |     |     |     |     |     |     | i i fi |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------ |
and (x ,T )∈X such that f ̸=f , x [s]̸=s [s], and H(x ,x ) is equal to the
|         | j   | j       | fj       |         | i j i | j          |     | i j |     |
| ------- | --- | ------- | -------- | ------- | ----- | ---------- | --- | --- | --- |
| minimal |     | Hamming | distance | between | X fi  | and X fj . |     |     |     |
Precisecomputationofsensitivebitscanbeexpensiveinpractice.Thenum-
ber of possible inputs to generate grows exponentially with m. We, of course,
consider only inputs for which the execution proceeds along the same program
pathuptotherecordattheindexdinthetraces.However,enumerationofonly
such inputs is a hard problem. Further, we do not know the minimal Hamming
distancebetweenthesetsX f inadvance(itmaydecreasewithanyinputwetry).
The goal of the sensitivity analysis is thus to compute only an approximation of
the sensitive bits. The set of detected bits may thus contain some non-sensitive
bits(causingadecreaseofeffectivityofotheranalyses)and/orsometrulysensi-
tivebitmaybemissingtheset(causingpossibledecreaseintheoverallcoverage
of Boolean instructions, because other analyses may be then unable to invert
their evaluation).

| Gray-Box        | Fuzzing | via      | Gradient | Descent |     | and Boolean   | Expression |              | Coverage | 17      |
| --------------- | ------- | -------- | -------- | ------- | --- | ------------- | ---------- | ------------ | -------- | ------- |
| The sensitivity |         | analysis | computes |         | the | approximation |            | of sensitive | bits     | as fol- |
lows.LetusconsideranodeN intheexecutiontree.So,wehavem=8·N.nbytes
and d = dN. We also have one pair (N.x[: m],N.T[: d+1]) ∈ X . Instead of
N.f
considering all possible pairs from all possible sets X we fix the first pair to
f
(N.x[: m],N.T[: d+1]) and we generate a sequence of other pairs (x ,T ) from
i i
other sets. Since we do not know the minimal Hamming distance from X N.f to
other sets, we generate inputs x by gradually increasing H(N.x[: m],x ) as we
|     |     |     |     | i   |     |     |     |     | i   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
generate more inputs. Namely, we first generate all 1-bit mutations of N.x[:m]
(cid:0)m(cid:1)
(i.e., first generated inputs), then all 2-bit mutations of N.x[:m] (i.e., next
| (cid:0)m(cid:1) | 1   |          |        |     |     |     |     |     |     |     |
| --------------- | --- | -------- | ------ | --- | --- | --- | --- | --- | --- | --- |
| generated       |     | inputs), | and so | on. |     |     |     |     |     |     |
2
| Unfortunately, |     | it turns | out | from | our | evaluation | that | performing | more | than |
| -------------- | --- | -------- | --- | ---- | --- | ---------- | ---- | ---------- | ---- | ---- |
1-bit mutations has negative impact on the overall performance of the FIzzer.
In fact, even 1-bit mutations already represent a considerable portions of all
inputsproducedbythetoolduringthewholeanalysis.Inordertodealwiththe
| situation | we implemented |     | the | following |     | two approaches: |     |     |     |     |
| --------- | -------------- | --- | --- | --------- | --- | --------------- | --- | --- | --- | --- |
– Theevaluationalsoreviledthat1-bitmutationsunder-approximatethetrue
| set of        | sensitive | bits          | a lot. | Since      | we        | cannot | generate | higher      | bit mutations, |       |
| ------------- | --------- | ------------- | ------ | ---------- | --------- | ------ | -------- | ----------- | -------------- | ----- |
| we extended   |           | the detection |        | of         | sensitive | bits   | to byte  | boundaries, | i.e.,          | when- |
| ever          | a bit     | is detected   | as     | sensitive, | then      | all    | bits in  | the same    | input byte     | are   |
| automatically |           | marked        | as     | sensitive  | as        | well.  |          |             |                |       |
– Although the approach above increased the precision considerably, we also
| generate | sequences |     | of “extreme” |     | bits | - those | with | high Hamming | distance |     |
| -------- | --------- | --- | ------------ | --- | ---- | ------- | ---- | ------------ | -------- | --- |
fromarandomlygeneratedbits.Forthisweusetheinformationabouttypes
in N.t:
• Bitscorrespondingtointegertypeswesettoallzerosandalsoalltoone.
| • Bits       | corresponding |           | to        | floating  | point  | types          | we   | set -1, 1,      | and to | special |
| ------------ | ------------- | --------- | --------- | --------- | ------ | -------------- | ---- | --------------- | ------ | ------- |
| values,      |               | like INF, | NAN,      | EPSILON.  |        |                |      |                 |        |         |
| We also      | observed      |           | these     | “extreme” | values | provide        |      | a considerable  | chance | to      |
| accidentally |               | uncover   | “special” |           | paths  | in the Target. |      |                 |        |         |
| Since        | we detect     | sensitive |           | bits      | w.r.t. | the fixed      | pair | (N.x[: m],N.T[: | d+1]), |         |
→−
we can detect sensitive bits simultaneously for multiple nodes in N. Indeed, for
each 0 ≤ k ≤ d we know N.T[k].f and we also know the number of bits we
should consider, namely 8·N.T[k].nbytes. Therefore, for each generated input
x, obtained from X.x[: m] either by 1-bit mutation or by the “extreme” values
mutation, we obtain the corresponding trace T, which we then map to nodes
of the execution tree. Namely, if K is the greatest index such that for all 0 ≤
k ≤ K and 0 ≤ l < K we have T[k].id = N.T[k].id and T[k].direction =
→−
N.T[k].direction, then we extend the mapping of each T[k] to N[k] by the
| sensitive | bit(s) | check: |     |     |     |     |     |     |     |     |
| --------- | ------ | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
– 1-bit mutation: If s is the index of the mutated bit, s < 8·N.T[k].nbytes,
| and | T[k].f | ̸= N.T[k].f, |     | then the | bit | at the | index | s is sensitive | in the | node |
| --- | ------ | ------------ | --- | -------- | --- | ------ | ----- | -------------- | ------ | ---- |
→−
| N[k] | (and | also all | other | bits in | the same | byte). |     |     |     |     |
| ---- | ---- | -------- | ----- | ------- | -------- | ------ | --- | --- | --- | --- |
– “extreme” value mutation: The same procedure as above repeated for each
| bit index | into | the | mutated | value. |     |     |     |     |     |     |
| --------- | ---- | --- | ------- | ------ | --- | --- | --- | --- | --- | --- |

| 18  | Martin | Jonáš | , Jan | Strejček | , Marek | Trtík, and Lukáš | Urban |     |
| --- | ------ | ----- | ----- | -------- | ------- | ---------------- | ----- | --- |
Notation For each tree node we store the set N.sbits of indices of all sensitive
| bits detected |     | by the   | sensitivity | analysis. |     |     |     |     |
| ------------- | --- | -------- | ----------- | --------- | --- | --- | --- | --- |
| 5.2 Bitshare  |     | analysis |             |           |     |     |     |     |
Let us consider a node N in the execution tree such that N.sbits̸=∅ and also a
Boolean value b such that N.label[b] = NOT_VISITED. The goal of the bitshare
→−
analysis is to find an input x so that the obtained trace T is mapped to N,
| |T|>dN                       | +1  | and T[dN].direction=b. |     |     |                                     |     |     |     |
| ---------------------------- | --- | ---------------------- | --- | --- | ----------------------------------- | --- | --- | --- |
| TheanalysislooksforeachnodeM |     |                        |     |     | inthetreesuchthatM.id.uid=N.id.uid, |     |     |     |
M.sbits ̸= ∅ and M.label[b] ̸= NOT_VISITED. Observe that we intentionally ig-
|     |     |     |     |     |     | →−  | −→  |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
nore the calling context M.id.ctx. Although the N and M represent different
sequencesof Booleaninstructions,theybothpassthroughtheinstructionunder
question (possibly even more than once). Since M.x evaluated the instruction
to b, then we could try to somehow compose N.x and M.x so that the resulting
| input x | would       | produce | a trace | T   | as described | above.        |               |              |
| ------- | ----------- | ------- | ------- | --- | ------------ | ------------- | ------------- | ------------ |
| The     | composition |         | of N.x  | and | M.x to       | x is based on | the sensitive | bits N.sbits |
and M.sbits. First we initialize x to be equal to N.x. Then we build sorted 12
sequences I and J of indices in N.sbits and M.sbits, respectively. Now for each
| 0≤i<min{|I|,|J|} |       |     | we set | x[I[i]]=M.x[J[i]]. |        |                   |         |             |
| ---------------- | ----- | --- | ------ | ------------------ | ------ | ----------------- | ------- | ----------- |
| Clearly,         | there | are | more   | ways               | how to | use the sequences | I and J | for mapping |
the sensitive bits of M to to x. But we do not have information telling us which
| is better. | So,       | we use    | the most | straightforward |      | approach.     |              |         |
| ---------- | --------- | --------- | -------- | --------------- | ---- | ------------- | ------------ | ------- |
| The        | described | approach, |          | of course,      | does | not guarantee | the obtained | trace T |
→−
for x will be mapped to N. But if it does, then there is reasonable chance the
| instruction | evaluates |     | to b | (see the | evaluation | results). |     |     |
| ----------- | --------- | --- | ---- | -------- | ---------- | --------- | --- | --- |
NOTE: Sincetheexecutiontreecanbelarge,theanalysisinfactdoesnotsearch
the tree for all such nodes M. Instead, whenever any of the two minimization
analyses,startedonsomenodeM,isforceterminated,i.e.,theBooleaninstruc-
tion was evaluated to the desired value b, then the bitshare analysis is informed
aboutthat,meaningthatitupdatesitsmapfrominstructionuniqueIDs(id.uid
fields) and evaluation results (direction fields) to values of sensitive bits of M.
When the bitshare analysis is started, then it uses input bits stored in its map.
| 5.3 Typed |     | minimization |     | analysis |     |     |     |     |
| --------- | --- | ------------ | --- | -------- | --- | --- | --- | --- |
The goal of the analysis is the same as of bitshare analysis (see the first article
in Sec.5.2). However, the analysis can be started for the node N, only if each
sensitive input bit N.x[s], where s ∈ N.sbits, belongs to a range of bits in N.x
associated with a type in N.t such that the type is none of UNTYPED* types
(see Sec.3). The reason for this requirement is that the analysis works on typed
| numerical | variables.   |     |           |     |         |              |     |     |
| --------- | ------------ | --- | --------- | --- | ------- | ------------ | --- | --- |
| 12 Using  | the standard |     | “<” order | on  | the set | of integers. |     |     |

| Gray-Box |     | Fuzzing | via Gradient | Descent | and | Boolean | Expression | Coverage |     | 19  |
| -------- | --- | ------- | ------------ | ------- | --- | ------- | ---------- | -------- | --- | --- |
300
250
200
150
100
50
0
|                                                   | 1     | 9 71 52 | 33 14 94 75 56 37 | 18 98 79 | 501 311 121 921 731 | 541 351 161 961 | 771 581 391 102 | 902 712 522    | 332 142 942 |     |
| ------------------------------------------------- | ----- | ------- | ----------------- | -------- | ------------------- | --------------- | --------------- | -------------- | ----------- | --- |
|                                                   |       | XOR 50  | XOR 64            | XOR 83   | XOR 90              |                 | XOR 111         | XOR 213        |             |     |
|                                                   | Plots | of      | function          | for      | 8-bit variable      |                 | and few         | fixed          | constants   | D.  |
| Fig.4.                                            |       | x       | xor D             |          |                     |                 | x               |                |             |     |
| Anothersituationwhenthisanalysisisnotused,ifN.xor |       |         |                   |          |                     |                 |                 | istrue.Whenxor |             |     |
instructionisusedinabranchingfunction,itthenoftenhasalotoflocalminima
which are difficult to escape from (see Fig.4). Although the gradient descent is
noteffectiveforbranchingfunctionwithxoringeneral,theversionpresentedin
Sec.5.4 performs slightly better in more cases. Therefore, we leave the analysis
| of nodes | with | N.xor | being true | to  | the other | algorithm. |     |     |     |     |
| -------- | ---- | ----- | ---------- | --- | --------- | ---------- | --- | --- | --- | --- |
Theanalysisthusstartsbyidentifyingtypednumericalvariablesv =(v 1 ,...,v m )
in N.x with types t=(t ,...,t ) in N.t using N.sbits. An example of this pro-
|     |     |     | 1   | m   |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
cess is depicted in Fig.5. There we identify two variables, since bit indices in
N.sbits points only to two regions associated with types in N.t. Observe that
not all bits of the variable v are sensitive. That is all right, because they are
1
| ignored | in the  | construction             | of       | inputs.   |           |        |          |       |        |     |
| ------- | ------- | ------------------------ | -------- | --------- | --------- | ------ | -------- | ----- | ------ | --- |
|         | 𝑁.𝑠𝑏𝑖𝑡𝑠 | { 16, …, 23, 64, …, 95 } |          |           |           |        |          |       |        |     |
|         | 𝑁.𝑥     |                          | 𝑣 1      |           |           |        | 𝑣        | 2     |        |     |
|         |         | 0                        | 8 24     | 32        |           | 64     |          |       | 96 104 |     |
|         | 𝑁.𝑡     | SINT8                    | UINT16   |           | SINT8     | SINT32 | FLOAT32  | UINT8 |        |     |
|         |         |                          | 0 1      |           | 2         | 3      | 4        |       | 5      |     |
|         |         |                          |          | 𝑡         |           |        | 𝑡        |       |        |     |
|         |         |                          |          | 1         |           |        | 2        |       |        |     |
|         | An      | example                  | of typed | numerical | variables |        | and with | types | and    | .   |
| Fig.5.  |         |                          |          |           |           | v 1    | v 2      |       | t 1    | t 2 |

20 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
Algorithm 1 Typed gradient descent
1: loop
2: v:= generate next seed
3: f(v):=ExecuteTarget(v)
4: if f(v) is not finite then continue
5: loop
6: for all i=1,...,m do
7: Compute the smallest ∆v >0 s.t. v +∆v ̸=v .
i i i i
8: ∇ f(v):= |ExecuteTarget(v1,...,vi+∆vi,...,vm)|−|f(v)|
9: if i ∇ f(v) is finite then loc ∆ k v [ i i]:=false
i
10: else ∇ f(v):=0, lock[i]:=true
i
11: success:=false
12: while ||∇f(v)||2 is finite and for some i we have lock[i]=false do
13: λ:=|f(v)|/||∇f(v)||2
14: if λ is zero or not finite then break
15: V′ =∅
16: for all e=0,−1,1,−2,2,−3,3 do
17: v′ :=v−10eλ∇f(v)
18: f(v′):=ExecuteTarget(v′)
19: V′ :=V′∪{(v′,f(v′))}
20: Let (v′,f(v′))∈V′ be the pair with the smallest |f(v′)|
21: if |f(v′)|<|f(v)| then
22: v:=v′, f(v)=f(v′), success:=true
23: break
24: else
25: L:={1/∇ f(v)2 | i=1,...,m and lock[i]=false}
i
26: l:=min(L)+0.6∗(max(L)−min(L))
27: for all i=1,...,m s.t. lock[i]=false do
28: if ∇ f(v)2 =0 or 1/∇ f(v)2 <l or not finite then
i i
29: ∇ f(v):=0, lock[i]:=true
i
30: if no direction was locked in the loop above then break
31: if success=false then break
Next follows the gradient descent of the unknown branching function f(v)
associated with the evaluation of the Boolean instruction, which corresponds
to the node N. The process is depicted in Alg.1. We see that the computation
happens in a seemingly infinite loop (see line 1). The algorithm terminates,
when the number of calls to ExecuteTarget exceed a certain limit 13 , say
K. 14 The check against the limit happens inside ExecuteTarget. If the limit
is exceeded the whole analysis is deactivated (meaning the search strategy is
finished). The function ExecuteTarget emulates the part of the fuzzing loop,
where the Target is executed for the passed input v and the obtained trace
T is mapped to the node N. If the T does actually not map to N, then the
function return ∞, representing a failure. Otherwise, the function returns the
13 The algorithm can also be force terminated any time from outside.
14 In our implementation we use an empirically adjusted number 100·|N.sbits|.

Gray-Box Fuzzing via Gradient Descent and Boolean Expression Coverage 21
valueT[dN].f.Thegradientdescentalgorithmcannotworkwithinfinitevalues.
Therefore, if T[dN].f is ∞, then it is also considered as a failure.
In each iteration of the outer loop we first try to compute a seed input v for
which we want to get a valid (finite) f(v). Once we succeed we enter the inner
loop at line 5 where we perform the gradient descent.
The process of seed generation depends on types in t and also on the actual
number of calls to ExecuteTarget. If the number of bits of a type t is smaller
i
than 16, then we uniformly sample from the entire domain the variable v (i.e.,
i
from all possible values of the type t ). For t with the size 16 bits or more, we
i i
uniformly sample from a certain interval of values of the domain. The bounds
of the interval are functions of the number of already performed calls, say k,
to ExecuteTarget. Namely, for signed integer type with |t | bits the interval
i
is [−p,p], where p = 27+(|ti|−9)k/K. For unsigned integer type the interval is
[0,p], where p = 27+(|ti|−8)k/K. And for floating type the interval is [−p,p],
where p=27+(q−8)k/K and q is 119 for float and 115 for double. All numeric
constants were adjusted empirically. The general idea behind the process is to
expand the sampled interval more and more as we approach closer and closer to
the limit K of Target executions.
Intheinnerloopatline5weperformthegradientdescentfromtheseedinput
storedinv.Eachiterationofthelooprepresentsasinglesuccessfuldescentstep,
i.e., we have found a new v such that |f(v)| decreased.
Intheloopatline6wenumericallycomputecoordinates∇ f(v),oneforeach
i
variablev ,ofthegradientvector∇f(v).Observethecoordinatesarecomputed
i
using right differences, since ∆v > 0. The computation of ∆v for t being an
i i i
integer type is simple. We always choose ∆v = 1. For the floating point type
i
t we must take into account the value of v . For example, if v = 1020 and we
i i i
choose ∆v =1, then we get v +∆v =v , which is something we do not want.
i i i i
For each coordinate ∇ f(v) we also maintain Boolean flag lock[i] which
i
can temporarily lock, i.e., disable, the coordinate from the descent. We lock the
coordinateifthevalue∇ f(v)isnotfiniteorifitconsiderablyreducesthespeed
i
of the descent.
Intheloopatline12weusethegradientvector∇f(v)forfindinganewinput
v′ andthecorrespondingvaluef(v′)suchthat|f(v′)|<|f(v)|.Weperformthe
search till there is at least one gradient coordinate available for use (i.e., not
locked), and the magnitude of the gradient vector is finite.
The computation of the parameter λ at line 13 represents the core of the
descent, because we use it for computation of new input vectors v′ (see line 17).
Wecomputeλunderanassumptionthatthebranchingfunctionislineararound
v so that we can get to zero in single step. More precisely we want to compute
the new input v′ as the intersection of the line
(cid:18) (cid:19) (cid:18) (cid:19)
v ∇f(v)
−λ
0 0

22 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
and a (hyper)plane
(cid:18) v (cid:19) (cid:18) e1 (cid:19) (cid:18) em (cid:19)
+t m +...+t m
|f(v)| 1 ∇ f(v) m ∇ f(v)
1 m
where ei is the vector of the i-th coordinate axis in the m-dimensional vector
m
space. So, we solve for λ
(cid:18) v (cid:19) (cid:18) ∇f(v) (cid:19) (cid:18) v (cid:19) (cid:18) e1 (cid:19) (cid:18) em (cid:19)
−λ = +t m +...+t m
0 0 |f(v)| 1 ∇ f(v) m ∇ f(v)
1 m
−λ∇ f(v)=t
1 1
.
.
.
−λ∇ f(v)=t
m m
0=|f(v)|+t ∇ f(v)+···+t ∇ f(v)
1 1 m m
We can substitute variables t to the last equation
i
0=|f(v)|+(−λ∇ f(v))∇ f(v)+···+(−λ∇ f(v))∇ f(v)
1 1 m m
0=|f(v)|−λ(∇ f(v)2+···+∇ f(v)2)
1 m
0=|f(v)|−λ||∇f(v)||2
λ=|f(v)|/||∇f(v)||2
In practice the branching function is not linear around v, so we generate
several inputs from v in the opposite direction of ∇f(v). That is done in the
loop at line 16. Observe that we generate inputs such that the parameters 10e
rangeoversevenordersofmagnitude.So,weperformsmallerstepsthanλ(upto
3ordersofmagnitude)andlargerstepsthanλ(alsoupto3ordersofmagnitude).
This approach tackles two important problems:
– When the gradient descent converges the a neighborhood close to global
minimumofthebranchingfunctionthegeneratedinputssamplethatneigh-
borhood.
– The gradient descent is more robust, meaning the generated input samples
increase change of escaping from a local minima.
The code in the “else” branch (below the at line 24) further improves and
robustness and also effectivity of the descent. If some ∇ f(v) is extremely large
i
compare to other coordinates, then the vector −λ∇f(v) tends to change those
other coordinates only negligibly. By locking the coordinate with the extreme
value we allow a descent in a new direction.
5.4 Minimization analysis
The goal of this minimization analysis is the same as of the typed minimization
(seeSec.5.3).Bothanalysesinfactapplythesamekindofalgorithm–thegradi-
entdescent.Thekeydifferenceisthatthisanalysisdoesnotusetheinformation

| Gray-Box | Fuzzing | via Gradient | Descent |     | and Boolean | Expression | Coverage | 23  |
| -------- | ------- | ------------ | ------- | --- | ----------- | ---------- | -------- | --- |
about types of bits in the input. So, the analysis can be started for the node
N, if some sensitive input bit N.x[s], where s ∈ N.sbits, belongs to a range
of bits in N.x associated with a type in N.t being some of UNTYPED* types (see
Sec.3).Thisanalysisisalsoused,ifN.xoristrue.Thatisforreasonswealready
discussed in Sec.5.3. In summary, this analysis is applied for nodes, for which
the typed minimization either cannot work (missing information about types),
| or when | this analysis | is expected |     | to perform | better | (xor instructions). |     |     |
| ------- | ------------- | ----------- | --- | ---------- | ------ | ------------------- | --- | --- |
Theminimizationanalysisappliesthegradientdescentalgorithm.Incontrast
to the gradient descent of the typed minimization analysis (see Sec.5.3), here
we consider each sensitive bit N.x[s], where s ∈ N.sbits, as an independent
variable of the Boolean type. So, we assume we have m = |N.sbits| variables
v = (v 1 ,...,v m ), where each v i can either be 0 or 1. This has, of course, an
impact on the structure and functionality of the algorithm. It is depicted at
Alg.2.
| Algorithm   | 2 Binary   | gradient | descent     |        |     |     |     |     |
| ----------- | ---------- | -------- | ----------- | ------ | --- | --- | --- | --- |
| 1: Generate | a sequence |          | of all seed | inputs |     |     |     |     |
S
| 2:      |        | in                        |     |     |     |     |     |     |
| ------- | ------ | ------------------------- | --- | --- | --- | --- | --- | --- |
| for all | v seed | S do                      |     |     |     |     |     |     |
| 3: v:=v |        | ,f(v):=|ExecuteTarget(v)| |     |     |     |     |     |     |
seed
| 4: mag:=[0,...,0] |     | s.t. | |mag|=m | //  | where | m=|N.sbits| |     |     |
| ----------------- | --- | ---- | ------- | --- | ----- | ----------- | --- | --- |
5: loop
6:
|     | for all | i=1,...,m | do  |     |     |     |     |     |
| --- | ------- | --------- | --- | --- | --- | --- | --- | --- |
7:
|     | f i (v):=|ExecuteTarget(v |     |     | 1 ,...,¬v | i ,...,v   | m )| |     |     |
| --- | ------------------------- | --- | --- | --------- | ---------- | ---- | --- | --- |
| 8:  | mag[i−1]:=max{mag[i−1],|f |     |     |           | (v)−f(v)|} |      |     |     |
i
9:
|     | k=argmin{f | i (v) | | i=1,...,m} |     |     |     |     |     |
| --- | ---------- | ----- | ------------ | --- | --- | --- | --- | --- |
i
| 10: | if f (v)<f(v) |     | then |     |     |     |     |     |
| --- | ------------- | --- | ---- | --- | --- | --- | --- | --- |
k
| 11: |     | ,              |     |     |     |     |     |     |
| --- | --- | -------------- | --- | --- | --- | --- | --- | --- |
|     | v k | :=¬v k f(v):=f | k   | (v) |     |     |     |     |
12:
continue
13: LetIbeapermutationof0,...,m−1s.t.∀i,j.i≤j →mag[I[j]]≤mag[I[i]]
| 14: | for all | i=1,...,m | do            |      |                |           |        |     |
| --- | ------- | --------- | ------------- | ---- | -------------- | --------- | ------ | --- |
| 15: | Let     | v be v    | with inverted | bits | at all indices | I[j] s.t. | j ≥i−1 |     |
i
16:
|     | f i (v     | i ):=|ExecuteTarget(v |                | i   | )|  |     |     |     |
| --- | ---------- | --------------------- | -------------- | --- | --- | --- | --- | --- |
| 17: | k=argmin{f | (v                    | ) | i=1,...,m} |     |     |     |     |     |
i i
i
| 18: | if f (v | )<f(v) | then |     |     |     |     |     |
| --- | ------- | ------ | ---- | --- | --- | --- | --- | --- |
k k
| 19: | v:=v     | , f(v):=f | (v′) |     |     |     |     |     |
| --- | -------- | --------- | ---- | --- | --- | --- | --- | --- |
|     |          | k         | k    |     |     |     |     |     |
| 20: | continue |           |      |     |     |     |     |     |
21:
break
Thealgorithmstartsbygeneratingallseedsthealgorithmmaypossiblyuse.
We define the count as a function of sensitive bits. Namely, we want to generate
aboutm+1seeds.Thiscountwasestablishedempirically.Thegoalistosample
thesetofall2m
possiblem-bitinputsuniformly.Wecanpartitionallinputsinto
m+1 classes C ,...,C according to their Hamming distance from the input
|     |     | 0 m |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

| 24 Martin | Jonáš | , Jan | Strejček , | Marek           | Trtík, and | Lukáš | Urban |     |     |
| --------- | ----- | ----- | ---------- | --------------- | ---------- | ----- | ----- | --- | --- |
|           |       |       |            | (cid:0)m(cid:1) | 15         |       |       |     |     |
0 = [0,...,0]. Observe that |C i | = . For an uniform sampling we should
i
take more samples from larger classes. Fortunately, the number of samples we
wanttogeneratecorrelateswiththenumberofclasses.So,wetakeonerandomly
chosen input from each class as a seed (for C we flip i randomly chosen (yet
i
| different) | bits in 0). |           |          |      |        |     |          |            |     |
| ---------- | ----------- | --------- | -------- | ---- | ------ | --- | -------- | ---------- | --- |
| The rest   | of the      | algorithm | operates | with | inputs | of  | the size | m although | the |
actual size of the input is |N.x|≥m. This is possible, because any m-bit input
v passed to ExecuteTarget is used together with N.x and N.sbits to build the
actual input x for the Target. Namely, x is first initialized to N.x and then, if
we assume the indices in N.sbits are ordered by the standard “<”, then for each
i = 1,...,m we set x[N.sbits[i]] to v[i]. Another important assumption about
the ExecuteTarget function always returns a finite floating point value. More
precisely, when the accepts a trace T from the (executed on the
|     |     | Server |     |     |     |     | Target |     |     |
| --- | --- | ------ | --- | --- | --- | --- | ------ | --- | --- |
input x), then it returns T[dN].f if the trace is mapped to N and T[dN].f is
finite. Otherwise, the maximal double value is returned. Since the goal of the
algorithm is to approach the global minimum of the branching function, the
| maximal double   | value | represents | of    | the       | worst possible |       | outcome.     |     |        |
| ---------------- | ----- | ---------- | ----- | --------- | -------------- | ----- | ------------ | --- | ------ |
| The minimization |       | algorithm  | takes | generated |                | seeds | sequentially | one | by one |
and for each it tries to apply the binary gradient descent to approach the global
minimuminahopeofinvertingtheevaluationresultoftheBooleaninstruction
associated with the node N along the way. The descent is inside the loop at line
5. There we first sample the branching function around v for 1-bit mutations,
for each bit index i one mutation, in order to obtain the absolute values of the
corresponding branching function values |f (v)|. That is done in the loop at line
i
6. Observe that we actually do not compute partial derivatives of the branching
function. That is because we perform the step only in one of m coordinates of
the gradient, i.e., in the coordinate k such that f (v) is the smallest. 16 This
k
index k is computed at line 9. If we further have f k (v) < f(v), then we move
in the direction of the coordinate k (see line 11) and we continue to the next
gradient step. The decision for modifying the standard version of the gradient
descent so that we step only in direction (coordinate) is based on our practical
experience with the algorithm – the single coordinate version is more robust,
i.e., it has a higher success rate of escaping from a local minimum, in a price of
decreased effectivity.Since weuse thisalgorithm mostlyfor branching functions
with lots of local minima (like xor function, see Fig.4), the robustness is more
| valuable than | convergence |      | speed.         |      |         |     |            |          |      |
| ------------- | ----------- | ---- | -------------- | ---- | ------- | --- | ---------- | -------- | ---- |
| Observe       | that the    | code | at lines 14–19 | look | similar | to  | the binary | gradient | step |
describedabove(lines6–11).Thereisonekeydifferencethough.Intheconstruc-
tion of the mutated inputs v at line 15, in contrast to line 7, more than one bit
i
canbemutated.Thesemulti-bitmutationsaretargetedtosituationswhensome
sensitive bits N.sbits collectively behave as an integer. It is easy to show that
15 SizesoftheclassesthusformtherowmofthePascal’striangle(rowsbeingindexed
from 0).
16 Thecomputationof∇ f(v)=(f (v)−f(v))/1wouldbeuseless,becausethef(v):=
|          |        | k    | k   |     |     |     |     |     |     |
| -------- | ------ | ---- | --- | --- | --- | --- | --- | --- | --- |
| f(v)+1·∇ | f(v)=f | (v). |     |     |     |     |     |     |     |
k k

| Gray-Box | Fuzzing | via Gradient | Descent |     | and Boolean | Expression | Coverage |     | 25  |
| -------- | ------- | ------------ | ------- | --- | ----------- | ---------- | -------- | --- | --- |
convergence from one integer to another using only single bit mutations can get
| stuck in a | local minimum.               |       | Let us | consider | this program |     |     |     |     |
| ---------- | ---------------------------- | ----- | ------ | -------- | ------------ | --- | --- | --- | --- |
| char       | x = __VERIFIER_nondet_char() |       |        |          | & 15;        |     |     |     |     |
| bool       | bi = x                       | == 4; |        |          |              |     |     |     |     |
Weclearlyhavefoursensitivebits N.sbits={4,5,6,7}andourbranching func-
tion is f(x) = x−4. Observe that for v = (0,0,1,1) we have |f(v)| = 1 and
there is no single-bit mutation v′ of v for which |f(v′)<|f(v)|. So, v is a local
minimum.However,wecanescapefromitbymutatingthelast3bitssimultane-
ously,i.e.,wegetv′ =(0,1,0,0)andf(v′)=0.Observealso,thatwecanobtain
(0,1,0,0), if we trait v as an integer and we added 1 to it. So, the idea behind
ourmulti-bitmutationsistoincrementv by1inahopetoescapethedescribed
local minimum, if we happened to get stuck there. However, we need to know
the importance of the sensitive bits in the integer value for the implementation
of the incrementation. Let us insert the following line in between the two lines
of code above
| x = ((x | & 1) | << 3) | | (x | & 6) | | (x & 8) | >> 3; |     |     |     |
| ------- | ---- | ----- | ---- | ------ | ------- | ----- | --- | --- | --- |
This line swaps the bits at indices 4 and 7. Clearly, the desired input we seek is
now v′ =(0,1,0,0), which means that we need to mutate a different 3 bits then
previously.
Wethusalwaysneedtodetecttheimportanceofbits.Wedosointhebinary
gradient descent, at line 8, by computing elements of the sequence mag. An
element mag[i] stores the maximum difference between values f (v) and f(v)
i
computed during all gradient steps from a given seed. The higher the value
mag[i] the higher importance of the the sensitive bit i. The permutation I then
| represents | the order | of sensitive | bits       | in the | decreasing | importance. |         |     |        |
| ---------- | --------- | ------------ | ---------- | ------ | ---------- | ----------- | ------- | --- | ------ |
| In the     | example   | above        | we guessed | which  | 3 bits     | should be   | mutated | to  | escape |
from the local minimum. Unfortunately, in general we do not know what sub-
sequence of m sensitive bit should be mutated. Therefore we try all m of them
| (see the loop | at line | 14).  |           |     |     |     |     |     |     |
| ------------- | ------- | ----- | --------- | --- | --- | --- | --- | --- | --- |
| 6 Selection   | of      | input | generator |     |     |     |     |     |     |
Whenever none of the four input generation analysis is active there must be
some of them selected and activated. Each of these analyses operates with some
node N of the execution. The node is passed to the analysis as an argument of
the activation. Therefore, the selection process of an analysis to activate starts
with a search for a node N in the tree. The analysis is then selected based on
node’s properties.
| Only nodes | corresponding |     | to  | Boolean | instructions | which | has | not been | cov- |
| ---------- | ------------- | --- | --- | ------- | ------------ | ----- | --- | -------- | ---- |
ered yet are considered in the search. We update the information about cover-
age of Boolean instructions (and corresponding nodes) in each iteration of the
fuzzing loop; that happens during the process of mapping accepted execution
| traces to the | execution | tree.   |       |      |               |            |     |           |     |
| ------------- | --------- | ------- | ----- | ---- | ------------- | ---------- | --- | --------- | --- |
| The node      | selection | process | works | with | the following | properties |     | of nodes: |     |

| 26 Martin | Jonáš | , Jan      | Strejček | , Marek   | Trtík, | and Lukáš            | Urban           |     |     |
| --------- | ----- | ---------- | -------- | --------- | ------ | -------------------- | --------------- | --- | --- |
| – A node  | N is  | directly   | input    | dependent | (DID), | iff N.sa∧N.sbits̸=∅. |                 |     |     |
| – A node  | N is  | indirectly | input    | dependent |        | (IID), iff           | N.sa∧N.sbits=∅. |     |     |
– AnodeN isopeniffthereisaBooleanvaluebs.t.N.labels[b]=NOT_VISITED
and ¬N.sa∨(N.sbits̸=∅∧(¬N.ba∨¬N.ma))
– A node N is closed, iff it is not open and for both Boolean values b either
| N.labels[b]̸=VISITED |             |     | or      | N.succ[b]   | is closed. |               |               |     |     |
| -------------------- | ----------- | --- | ------- | ----------- | ---------- | ------------- | ------------- | --- | --- |
| Before               | we continue |     | further | let us look | at         | the following | observations: |     |     |
– Thecheckforanodebeingcloseddependsonsuccessornode(s)beingclosed
(ifthereissome).Itmeansthatfirstclosednodesareleavesofthetree,then
| their parents, |        | and    | so on up | to the   | root node. |     |          |                 |     |
| -------------- | ------ | ------ | -------- | -------- | ---------- | --- | -------- | --------------- | --- |
| Note: We       | update | closed | state    | of nodes | whenever   | an  | analysis | is deactivated. |     |
Theupdatestartsfromthenodetheanalysiswasstartedwithandcontinues
| towards | the | root of | the execution | tree. |     |     |     |     |     |
| ------- | --- | ------- | ------------- | ----- | --- | --- | --- | --- | --- |
– A node cannot be DID and IID in the same time, but it can be neither DID
| nor IID. | The | same | we can | say also | for open | and closed | properties. |     |     |
| -------- | --- | ---- | ------ | -------- | -------- | ---------- | ----------- | --- | --- |
Wefirstsearchforthenodeamongstprimarycoveragetargets(discussedlater
in Sec.6.1). If the search fails, then we continue by a Monte Carlo search from
some IID pivot (also discussed later in Sec.6.2). If this search fails as well, then
the analysis cannot make any further progress and the fuzzing loop terminates.
17 However, if some of the two searches succeeds, then we obtain the winning
node N and we proceed with it to the selection of the analysis to be activated.
This process is depicted in Alg.3. But before we look at it we introduce two
notations.
Notation Weallowtodefineasequenceina“set” style,i.e.,[f(v)|v =v ,...,v ]
|                 |      |             |     |       |     |     |     |     | 1 n |
| --------------- | ---- | ----------- | --- | ----- | --- | --- | --- | --- | --- |
| is the sequence | [f(v | 1 ),...,f(v |     | n )]. |     |     |     |     |     |
Notation For each node N in the execution tree we introduce an integer field
N.height which is updated for every execution trace T mapped to N to a value
max{N.height,|T|}. When the node N is created and inserted to the tree,
then the field is initialized to |T| (recall that a node can be inserted to the
tree only when some trace is mapped to the tree). Observe the field actually
stores the maximal from depths of all nodes in its sub-tree(s), i.e., the value
−→
| max{dM | | N ∈M}. |            |        |          |     |          |        |        |           |
| -------- | ------ | ---------- | ------ | -------- | --- | -------- | ------ | ------ | --------- |
| We are   | ready  | to discuss | Alg.3. | In first | two | lines we | try to | find a | node N in |
the tree to be used for the selection of the analysis (to be then activated with
the node). Details are discussed in Sec.6.1 and Sec.6.2. If the selection of the
node N fails, then we terminate the fuzzing loop, i.e., we terminate the entire
| fuzzing process. |     | Otherwise, | we  | may proceed |     | to the analysis | selection. |     |     |
| ---------------- | --- | ---------- | --- | ----------- | --- | --------------- | ---------- | --- | --- |
17 There are circumstances under which the termination of the fuzzing loop can be
| resumed,    | meaning | that    | we are | able to    | make        | some nodes | in the | tree to | be primary |
| ----------- | ------- | ------- | ------ | ---------- | ----------- | ---------- | ------ | ------- | ---------- |
| targets. We | discuss | details | of     | this later | in Sec.6.3. |            |        |         |            |

| Gray-Box    | Fuzzing   | via            | Gradient | Descent        | and             | Boolean      | Expression | Coverage      | 27  |
| ----------- | --------- | -------------- | -------- | -------------- | --------------- | ------------ | ---------- | ------------- | --- |
| Algorithm   | 3 Select  | analysis       |          | for activation |                 |              |            |               |     |
| 1: Selected | a primary | coverage       |          | target         | N (see Sec.6.1) |              |            |               |     |
| 2: if N     | =null     | then Selected  |          | N by           | the Monte       | Carlo method |            | (see Sec.6.2) |     |
| 3: if N     | =null     | then Terminate |          | the            | fuzzing loop.   |              |            |               |     |
4:
| if ¬N.sa | then |     |     |     |     |     |     |     |     |
| -------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
5:
loop
| 6:  | succ:=[N.succ[b]                            |     | |    | b=false,true] |     |     |          |     |     |
| --- | ------------------------------------------- | --- | ---- | ------------- | --- | --- | -------- | --- | --- |
| 7:  | dir:=[succ[i]̸=null∧succ[i].nbytes=N.nbytes |     |      |               |     |     | | i=0,1] |     |     |
| 8:  | if dir[0]∧dir[1]                            |     | then |               |     |     |          |     |     |
9:
|     | N   | :=succ[succ[0].height≥succ[1].height |     |     |     | ?   | 0 : 1] |     |     |
| --- | --- | ------------------------------------ | --- | --- | --- | --- | ------ | --- | --- |
10:
|     | else if | dir[0]    | then |     |     |     |     |     |     |
| --- | ------- | --------- | ---- | --- | --- | --- | --- | --- | --- |
| 11: | N       | :=succ[0] |      |     |     |     |     |     |     |
| 12: | else if | dir[1]    | then |     |     |     |     |     |     |
| 13: | N       | :=succ[1] |      |     |     |     |     |     |     |
14:
else
15:
break
| 16: Select | the      | sensitivity | analysis | with | the node | N.  |     |     |     |
| ---------- | -------- | ----------- | -------- | ---- | -------- | --- | --- | --- | --- |
| 17: else   | if ¬N.ba | then        |          |      |          |     |     |     |     |
| 18: Select | the      | bitshare    | analysis | with | the node | N.  |     |     |     |
19: ¬N.xorandN.sbitscorrespondtonumericalvariablesofknowntypesthen
elseif
| 20: Select | the | typed | minimization |     | analysis with | the node | N.  |     |     |
| ---------- | --- | ----- | ------------ | --- | ------------- | -------- | --- | --- | --- |
21: else
| 22: Select | the         | minimization |     | analysis | with the     | node N. |        |                |       |
| ---------- | ----------- | ------------ | --- | -------- | ------------ | ------- | ------ | -------------- | ----- |
| If the     | sensitivity | analysis     |     | has not  | been applied | to      | N yet, | it is selected | (with |
possibly a node in N’s subtree; we discuss details below), because it will com-
pute the sensitive bits necessary for other analyses. Otherwise, we attempt to
select the bitshare analysis, because it is fast (it basically retrieves inputs from
thecache)andalsoeffective.Iftheprevioustwoanalysesarenotavailable,then
we select one of the two minimization analyses. If the conditions for activation
of the typed minimization are satisfied, then we select it, because it performs
better under these conditions. Otherwise, the conditions are such that the min-
imization analysis is expected to perform better, and so it is selected. Observe
that although we do not check for N.ma when choosing between the two min-
imization analyses, we are sure that N.ma is false, because the selected node
N is open.
| It remains | to  | explain | the | purpose | of the | loop in | the algorithm. | Recall | that |
| ---------- | --- | ------- | --- | ------- | ------ | ------- | -------------- | ------ | ---- |
givenanodeN,thesensitivityanalysismaycomputeorupdatethesensitivebits
→−
N′s
of any node in N. If we activate the analysis with some node in subtree (if
thereisany),thenN.sbitswillstillbecomputedandtheanalysiswillinaddition
compute N.sbits of more nodes. So, it looks like we achieve the best effectivity,
ifwestarttheanalysisinaleafnodeinN′ssubtreeatthehighestdepth.There
is a catch however. Nodes below N may correspond to more input bytes, i.e.,
their field nbytes can be greater than N.nbytes. If such node is chosen, then
the analysis will check for sensitivity of bits at indices ≥ 8·N.nbytes, which

28 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
definitely cannot be sensitive bits of N. So, we thus actually could decrease
effectivity, especially if nbytes of the leaf node is much larger than N.nbytes.
Also,ifthenodeselectionalgorithmwantedtoapplythesensitivityforalonger
input,thenitwouldselectedsomenodebelowN inthefirstplace.Therefore,in
order to be sure we maximize the effectivity, we must consider only those nodes
below N with the same value in the field nbytes. But from all these nodes we
may still choose any with the maximal depth in the tree. The search for such
node is implemented in the loop in Alg.3. Observe that we use the field height
to navigate towards a leaf at the highest depth.
6.1 Searching in primary coverage targets
A primary target is a node appearing in any of the following
– Loopheads:AsetHofnodes.EachitsnodeN correspondtoanexecutionof
aBooleaninstructionrepresentingtheheadofsomeloopalonganexecution
trace mapped either to the node or some node in its subtree. Since these
nodesrepresentbordersbetweeniterationsofloops,executiontracesmapped
the yet not visited successor of N may improve coverage of many nodes. Of
course, a loop can be iterated many time, so we must computed which of
alliterationsareactuallyimportantfortheoveralleffectivityoftheFIzzer.
We discuss details below.
– Sensitive:AnorderedsetS ofopennodesN suchthatN wasonlyprocessed
by the sensitivity analysis, i.e., it was prepared for other input generation
analyses (see Sec.5), but none of those has been activated with N yet. The
order of nodes has an impact on effectivity of the overall performance. We
discuss it later.
– Untouched: An ordered set U of open nodes N such that N.sa is false and
further N.id is not the location of any IID pivot (see Sec.6.2). It means that
N can be processed by the four input generation analyses (see Sec.5), but it
has not been “touched” by any of them yet. The order of nodes is given by
the same relation as the one on the set of sensitive targets.
– IIDtwins:AsequenceT ofopennodesN suchthatN.saisfalseandthere
isanIIDpivotM (seeSec.6.2)suchthatN.id=M.id∧|N.f(x)|<|M.f(x)|.
ItmeansthatN canbebuthasnotbeenprocessedbyanyofthefourinput
generation analyses (see Sec.5) yet. It also represents the same uncovered
Boolean instruction as the node M. So, N is a “twin” of M. Moreover,
from the comparison of values of the branching function f(x) the node N
is closer to the global minimum and so it has higher potential for covering
the instruction than M. Therefore, even if an activation of the sensitivity
analysiswithN wouldresultinN.sbits=∅(whichisquitelikelytohappen),
the node could still be valuable for the search discussed in Sec.6.2.
We insert nodes into S,U,T during the process of mapping of each accepted
executiontraceT totheexecutiontree.Namely,foreachnodeN insertedtothe
tree we consider the insertion of N to each of them. The insertion of nodes into

| Gray-Box | Fuzzing | via | Gradient | Descent | and | Boolean Expression |     | Coverage | 29  |
| -------- | ------- | --- | -------- | ------- | --- | ------------------ | --- | -------- | --- |
H is more complicated. It happens during the actual node selection process. We
discussitlater.WefurtherprunecontentsofallH,S,U,T everytimetheactive
input generation analysis becomes deactivated. We erase all those nodes which
| do not satisfy | the      | criteria | we      | defined | above. |     |     |     |     |
| -------------- | -------- | -------- | ------- | ------- | ------ | --- | --- | --- | --- |
| Algorithm      | 4 Select | a        | primary | target  | node   |     |     |     |     |
1:
| if H̸=∅    | then |      |      |     |        |     |     |     |     |
| ---------- | ---- | ---- | ---- | --- | ------ | --- | --- | --- | --- |
| 2: Extract | any  | node | from | and | return | it. |     |     |     |
H
| 3: else | if S ≠ ∅ | then         |     |         |             |        |     |     |     |
| ------- | --------- | ------------ | --- | ------- | ----------- | ------ | --- | --- | --- |
| 4: Let  | N be      | the smallest |     | node of | the ordered | set S. |     |     |     |
−→
| 5: if | Loop heads | has        | not   | been detected |     | along N then |     |     |     |
| ----- | ---------- | ---------- | ----- | ------------- | --- | ------------ | --- | --- | --- |
| 6:    | Detect     | loop heads | along | −→            |     |              |     |     |     |
N
| 7:  | goto line | 1.  |     |     |     |     |     |     |     |
| --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
8: else
| 9:  | Extract | N from | S   | and return | it. |     |     |     |     |
| --- | ------- | ------ | --- | ---------- | --- | --- | --- | --- | --- |
10:
| else    | if U ̸=∅ | then         |     |         |             |        |     |     |     |
| ------- | -------- | ------------ | --- | ------- | ----------- | ------ | --- | --- | --- |
| 11: Let | be       | the smallest |     | node of | the ordered | set U. |     |     |     |
N
−→
| 12: if | Loop heads | has | not | been detected |     | along N then |     |     |     |
| ------ | ---------- | --- | --- | ------------- | --- | ------------ | --- | --- | --- |
−→
| 13: | Detect | loop heads | along | N   |     |     |     |     |     |
| --- | ------ | ---------- | ----- | --- | --- | --- | --- | --- | --- |
| 14: | line   | 1.         |       |     |     |     |     |     |     |
goto
15:
else
| 16:      | Extract  | N from | U   | and return | it. |     |     |     |     |
| -------- | -------- | ------ | --- | ---------- | --- | --- | --- | --- | --- |
| 17: else | if |T|>0 | then   |     |            |     |     |     |     |     |
| 18: N    | :=T[0]   |        |     |            |     |     |     |     |     |
−→
| 19: if | Loop heads | has | not | been detected |     | along N then |     |     |     |
| ------ | ---------- | --- | --- | ------------- | --- | ------------ | --- | --- | --- |
−→
| 20: | Detect | loop heads | along | N   |     |     |     |     |     |
| --- | ------ | ---------- | ----- | --- | --- | --- | --- | --- | --- |
| 21: | line   | 1.         |       |     |     |     |     |     |     |
goto
22:
else
| 23: | T :=T[1:], | return |     | N   |     |     |     |     |     |
| --- | ---------- | ------ | --- | --- | --- | --- | --- | --- | --- |
24:
| else | return  | null         |     |           |        |      |             |           |     |
| ---- | ------- | ------------ | --- | --------- | ------ | ---- | ----------- | --------- | --- |
| The  | process | of selection |     | a primary | target | node | is depicted | in Alg.4. | The |
procedureisstraightforward.WetakeH,S,U,T inthisexactorderandwelook
for the first one not being empty. If all are empty, no primary target can be
selected and we return null. Otherwise, we extract some node from the non-
18
empty set and we return it. From H,T we choose the node randomly. From
| the ordered | sets | S,U | we use    | the order | to   | take the smallest | node.  |              |        |
| ----------- | ---- | --- | --------- | --------- | ---- | ----------------- | ------ | ------------ | ------ |
| Observe     | that | the | selection | of node   | from | S,U,T             | can be | interrupted, | if the |
chosen node N has not been considered in the loop detection yet. In which case
we perform the detection (which can make H non-empty) and then basically
restart the selection process (we return to the first line). There is the following
→−
reason for this implementation. The count of paths N in the tree, for any node
18 AlthoughweinfactalwaystakethefirstelementofT,theorderinwhichthenodes
arrive,andwhichwealwayspushtotheendofT,israndom(notimportantforus).

| 30  | Martin | Jonáš , | Jan Strejček | ,   | Marek Trtík, | and | Lukáš Urban |     |     |
| --- | ------ | ------- | ------------ | --- | ------------ | --- | ----------- | --- | --- |
N, can be large. We thus cannot perform the computation for all nodes, since
that would have a serious negative impact on the overall performance of the
Server. But we know that we surely want to process the node selected from
| S,U,T, |     | i.e., we surely | want | to consider | loop | heads | on the path | to that node. |     |
| ------ | --- | --------------- | ---- | ----------- | ---- | ----- | ----------- | ------------- | --- |
Detection of loop heads The Server has no information about control flow
structures, like branchings or loops, in the Target. Therefore, when we speak
about loops and loop heads, we actually consider only repetitions of Boolean
instructions along paths in the execution tree. 19 We compute loop heads, to
be inserted to H, in three steps. First we must select a node N in the tree. We
already know that it is a node selected from any of S,U,T. In the second step
→−
we detect all loop heads along the path N using Alg.5. The algorithm is in fact
moregeneral,becauseitisalsousedinSec.6.2.So,wewilldescribeitcompletely.
| Algorithm |     | 5 Detect | loops |     |     |     |     |     |     |
| --------- | --- | -------- | ----- | --- | --- | --- | --- | --- | --- |
1: loops:=[],
heads2bodies:=∅
2: stack:=[],
lookup:=∅
−→
| 3:  | for all | i=|N|−1,...,0 |     | do  |     |     |     |     |     |
| --- | ------- | ------------- | --- | --- | --- | --- | --- | --- | --- |
| 4:  |         |               | −→  |     |     |     |     |     |     |
j :=min{i+1,|N|−1}
−→
| 5:  | if  | N[i].id̸∈Dom(lookup) |     | then |     |     |     |     |     |
| --- | --- | -------------------- | --- | ---- | --- | --- | --- | --- | --- |
−→
| 6:  |     | lookup[N[i].id]:=|stack| |     |     |     |     |     |     |     |
| --- | --- | ------------------------ | --- | --- | --- | --- | --- | --- | --- |
−→ −→
| 7:  |     | stack:=stack+[(N[i],N[j],0)] |     |     |     |     |     |     |     |
| --- | --- | ---------------------------- | --- | --- | --- | --- | --- | --- | --- |
8:
else
| 9:  |     |     | −→  |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
k:=lookup[N[i].id]
| 10: |     | if stack[k].index=0     |     | then |     |     |     |     |     |
| --- | --- | ----------------------- | --- | ---- | --- | --- | --- | --- | --- |
| 11: |     | stack[k].index:=|loops| |     |      |     |     |     |     |     |
| 12: |     |                         |     | −→   |     |     |     |     |     |
loops:=loops+[(N[i],stack[k].X,stack[k].S)]
| 13: |     | else |     |     |     |     |     |     |     |
| --- | --- | ---- | --- | --- | --- | --- | --- | --- | --- |
−→
| 14: |     | loops[stack[k].index].E |     |     | :=N[i] |     |     |     |     |
| --- | --- | ----------------------- | --- | --- | ------ | --- | --- | --- | --- |
15:
|     |         | while |stack|>k+1     |              | do   |                             |     |     |     |     |
| --- | ------- | --------------------- | ------------ | ---- | --------------------------- | --- | --- | --- | --- |
| 16: |         | Insert stack[−1].X.id |              | to   | heads2bodies[stack[k].X.id] |     |     |     |     |
| 17: |         | Erase stack[−1].X.id  |              | from | lookup                      |     |     |     |     |
| 18: |         | Erase the             | last element | from | stack                       |     |     |     |     |
| 19: |         | triples in            |              |      |                             |     |     |     |     |
|     | for all | L                     | loops        | do   |                             |     |     |     |     |
20:
|     | while                 | L.E.parent       | ̸=  | null ∧ | (L.E.parent.id |     | = L[1].id | ∨ L.E.parent.id | ∈   |
| --- | --------------------- | ---------------- | --- | ------ | -------------- | --- | --------- | --------------- | --- |
|     | heads2bodies[L.X.id]) |                  | do  |        |                |     |           |                 |     |
| 21: |                       | L.E :=L.E.parent |     |        |                |     |           |                 |     |
22:
|     | return | loops,heads2bodies |     |     |     |     |     |     |     |
| --- | ------ | ------------------ | --- | --- | --- | --- | --- | --- | --- |
Loopdetection Thealgorithmcomputesasequenceloopsandamapheads2bodies.
An elements of loops is a triple (E,X,S), called a loop boundary, where E is
19 Nevertheless,arepetitionofaBooleaninstructionoftenmeansthatitinfactappears
| inside |     | an actual loop | inside | the Target. |     |     |     |     |     |
| ------ | --- | -------------- | ------ | ----------- | --- | --- | --- | --- | --- |

| Gray-Box | Fuzzing | via | Gradient | Descent | and | Boolean | Expression | Coverage |     | 31  |
| -------- | ------- | --- | -------- | ------- | --- | ------- | ---------- | -------- | --- | --- |
thetreenodefromwhichweentertotheloop,X isthenodefromwhichweexit
→−
from the loop, and S is the successor of X in N. The map heads2bodies maps
IDs of Boolean instruction detected as loop heads to a set of IDs of all Boolean
| instructions | representing |     | the | body | of the loop. |     |     |     |     |     |
| ------------ | ------------ | --- | --- | ---- | ------------ | --- | --- | --- | --- | --- |
→−
| WecomputebothresultsbyprocessingthepathN |     |     |     |     |     |     | backwards.20Duringthis |     |     |     |
| ---------------------------------------- | --- | --- | --- | --- | --- | --- | ---------------------- | --- | --- | --- |
traversalwebuildastackstack,wherewestackthefirstoccurrencesof Boolean
instructions (their ids). An element of the stack is a triple (X,S,index), where
X is the node from which we exit from the loop, S is the successor of X in
→−
N, and index is the index of the corresponding element in the sequence loops.
We use a map lookup for mapping the first occurrences of Boolean instructions
(their ids) to indices the of the corresponding records in stack. Observe that
we check for the first occurrences at line 5. In the case of the first occurrence
we extend both stack and lookup map. Otherwise, we query the lookup map
to get the index k of the triple in stack representing the first occurrence of
→−
N[i].id. The case when stack[k].index = 0 identifies the first repetition of the
|     |     |     |     | →−  |     | →−  |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Boolean instruction with ID N[i].id along N (by going backwards). Therefore,
this is the first evidence that we are in a loop, and so we record the loop in the
sequence loops. Otherwise, this is some other iteration of the loop. So we only
→−
move the entry to the loop to the current node N[i]. The loop at line 15 erases
everything from stack and lookup what was recorded since the first occurrence
of the instruction, which is the record at index k in stack. Note that erased
records represent Boolean instructions forming the body of the loop. Therefore,
we insert all IDs of Boolean instructions corresponding to all erased records to
map heads2bodies.
| The | loop at | line 19 | performs | a   | postprocessing |     | of loop entries | of  | all recorded |     |
| --- | ------- | ------- | -------- | --- | -------------- | --- | --------------- | --- | ------------ | --- |
loops. We basically do not want the entry and exit nodes (instructions) be the
same and we also do not want the entry to be in the loop body. We resolve such
| situations     | by moving     | the           | entry | towards | the     | root node                 | of the | execution | tree. |     |
| -------------- | ------------- | ------------- | ----- | ------- | ------- | ------------------------- | ------ | --------- | ----- | --- |
| Algorithm      | 6 Detect      | loop          | heads |         |         |                           |        |           |       |     |
| 1: W :={(2i,∅) |               | | i=0,...,10} |       |         |         |                           |        |           |       |     |
|                |               | −→            |       | −→      |         | −→                        |        |           |       |     |
| 2:             |               |               | s.t.  |         | is open | and                       |        |           |       |     |
| for all        | i=0,...,|N|−1 |               |       | N[i]    |         | N[i].id∈Dom(heads2bodies) |        |           |       | do  |
|                |               |               |       |         |         | −→                        |        | −→        |       |     |
3: Letw∈Dom(W)bes.t.∀w′ ∈Dom(W).|N[i].nbytes−w|≤|N[i].nbytes−w′|
−→
| 4: Insert |     | to the | ordered | set | W[w]. |     |     |     |     |     |
| --------- | --- | ------ | ------- | --- | ----- | --- | --- | --- | --- | --- |
N[i]
| 5: for all | H ∈Rng(W) |          | do            |        |         |         |             |     |         |      |
| ---------- | --------- | -------- | ------------- | ------ | ------- | ------- | ----------- | --- | ------- | ---- |
| 6: Insert  | the       | smallest | element       | (node) | in H    | into H. |             |     |         |      |
| Now        | we are    | back at  | the detection |        | of loop | head    | for the set | H.  | We only | need |
the domain of the map heads2bodies obtained from Alg.5. The computation of
20 Wecanexitfromalooponlyfromtheloop-headBooleaninstructions,buttheloop
doesnothavetostartwithit.Backwardtraversalthusallowsforeasierdetectionof
| the loop | heads. |     |     |     |     |     |     |     |     |     |
| -------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |

32 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
theloopheadusingDom(heads2bodies)isdepictedinAlg.6.Inthefirstloopwe
→−
collectallopenloopheadnodesalong N.However,insteadofinsertingthemall
directly to H, we actually group them, according to the number of input bytes
read along the path up to them.
The reason for that comes from evaluations, where we observed that overall
performance of the tool is highly sensitive to the selection of loop heads. Any of
the following two serious performance issues may occur, if we do not filter the
loop heads (e.g., as we do in Alg.6):
– Eachloopheadcorrespondstoacertainiterationofsomeloop.Thenumber
of loop iterations can be large. So, our analysis can easily get ineffective
because of processing of just a lots of loop heads.
– Moreinputbytesmaybereadorprocessedwiththeincreasingcountofloop
iterations. Effectivity of all four input generation analyses depend on input
size. So, a lot of effort can be spent just on the detection of sensitive bits by
the sensitivity analysis, leading to a serious performance decrease.
Our approach to the issues is to keep both the count of loop heads and also the
number of processed input bytes in reasonable bounds. Therefore, we group all
loop heads into just 11 classes based on the number of input bytes. We use the
map W for the grouping. Dom(W) define classes of input size and Rng(W) are
ordered sets of open loop head nodes N[i]. The exponential function 2i allows
for more refine grouping for small input size and coarse grouping for large input
size. For example, it allows distinguishing between input sizes 4 and 8, while
ignoring the difference between the sizes 1000 and 1004.
OncethemapW isfilledin,thenweinsertonlyonerepresentativenodefrom
each group into H, namely the smallest representative. Given H ∈ Rng(W),
thennodesinH isorderedusingthefollowingstrictorder:LetP,Q∈H.Then,
P <Q, iff
P.nbytes<Q.nbytes∨(P.nbytes=Q.nbytes∧dP <dQ)
.
Orderonthesetsofsensitiveanduntouchedtargets Theoveralleffectiv-
ity of the analysis not only depends on what nodes are selected, but also when.
Basedonresultofourevaluations,weestablishedthefollowingstrictweakorder
thesetsS,U:LetP,QbenodesineitherS orU andmax_bytesbethegreatest
valueofthefieldnbytesofallnodesintheexecutiontree.Then,P <Q,iffAlg.7
returns true.
The purpose of the lines 9 and 10 is to prefer nodes whose field nbytes is
close to the half of the maximal number input bytes read so far.
6.2 Monte Carlo search from IID pivot
TheultimategoalhereistocoverthoseBooleaninstructionswhosecorrespond-
ing nodes in the execution tree all have empty set of sensitive bits. We denote

| Gray-Box         | Fuzzing | via    | Gradient      | Descent |        | and Boolean | Expression | Coverage | 33  |
| ---------------- | ------- | ------ | ------------- | ------- | ------ | ----------- | ---------- | -------- | --- |
| Algorithm        | 7       | Strict | weak ordering |         | of S,U |             |            |          |     |
| 1: if P.sa∧¬Q.sa |         | then   | return        | true    |        |             |            |          |     |
2:
| if ¬P.sa∧Q.sa             |     | then | return | false  |       |     |     |     |     |
| ------------------------- | --- | ---- | ------ | ------ | ----- | --- | --- | --- | --- |
| 3: if |P.sbits|<|Q.sbits| |     |      | then   | return | true  |     |     |     |     |
| 4: if |P.sbits|>|Q.sbits| |     |      | then   | return | false |     |     |     |     |
5: :=[20,...,210]
W
6:
| p:=argmin{|P.nbytes−W[i]| |     |     |     | |   | i=0,...,|W|−1} |     |     |     |     |
| ------------------------- | --- | --- | --- | --- | -------------- | --- | --- | --- | --- |
i
7:
| q:=argmin{|Q.nbytes−W[i]| |     |     |     | |   | i=0,...,|W|−1} |     |     |     |     |
| ------------------------- | --- | --- | --- | --- | -------------- | --- | --- | --- | --- |
i
| 8: m:=argmin{|max_bytes/2−W[i]| |     |     |     |     | | i=0,...,|W|−1} |     |     |     |     |
| ------------------------------- | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- |
i
| 9: if |W[m]−W[p]|<|W[m]−W[q]|  |     |     |      |        | then | return | true  |     |     |
| ------------------------------ | --- | --- | ---- | ------ | ---- | ------ | ----- | --- | --- |
| 10: if |W[m]−W[p]|>|W[m]−W[q]| |     |     |      |        | then | return | false |     |     |
| 11: if P.nbytes<Q.nbytes       |     |     | then | return |      |        |       |     |     |
true
| 12: if P.nbytes>Q.nbytes |     |     | then | return | false |     |     |     |     |
| ------------------------ | --- | --- | ---- | ------ | ----- | --- | --- | --- | --- |
13:
| if dP     | <dQ | then | return | true  |     |     |     |     |     |
| --------- | --- | ---- | ------ | ----- | --- | --- | --- | --- | --- |
| 14: if dP | >dQ | then | return | false |     |     |     |     |     |
15:
| return | P.height>Q.height |     |     |     |     |     |     |     |     |
| ------ | ----------------- | --- | --- | --- | --- | --- | --- | --- | --- |
these nodes as IID nodes. Given an IID node N, we cannot activate any of the
four input generation analyses (see Sec.5). The actual goal here is to search for
such node M in the tree, with which some of the input generation analyses can
beactivatedandthesuccessoftheanalysis(i.e.,wegettheoppositeresultfrom
theevaluationoftheBooleaninstruction)wouldgetusclosertothecoverageof
N (i.e., of the instructions corresponding to N). By “get us closer” we
Boolean
actually mean that once the analysis is done (deactivated), there may appear a
| node N′ | in a subtree |     | of M   | such that | N′.id=N.id∧|N′.f(x)|<|N.f(x)|. |        |             |              |      |
| ------- | ------------ | --- | ------ | --------- | ------------------------------ | ------ | ----------- | ------------ | ---- |
| Before  | we explain,  |     | how we | search    | for                            | a node | M, we first | need to know | what |
IID nodes should actually be covered. We call them IID pivots and we detect
them in the execution tree whenever the sensitivity analysis is (forcefully) de-
activated. Each node changed by the analysis which is also uncovered IID node
(see Sec.6.1) is a new IID pivot. Recall the sensitivity analysis of a node N may
→−
actually change sensitive bits of any node in N. A node stays as an IID pivot
| until it | is covered. |        |              |     |        |       |          |               |        |
| -------- | ----------- | ------ | ------------ | --- | ------ | ----- | -------- | ------------- | ------ |
| We       | start our   | search | by selecting |     | an IID | pivot | which we | would like to | cover. |
This is done in two steps. First we partition the set of all pivots by the field
id. That makes sense, because there can be several IID pivots in the tree corre-
sponding to the same Boolean instruction. In this step we just want to decide,
whichoftheBooleaninstructionwefocuson.Sincewearenotawareofamean-
ingful information for ordering the instructions, we choose a partition class C
of IID pivots randomly, using the uniform distribution. In the second step we
select a representative pivot from C. In contrast to the previous step we have
an information (inferred from our evaluations) to build a strict weak order on
C. It is depicted in Alg.8, where P,Q ∈ C. Surprisingly, our evaluation shows
that instead of always choosing the smallest pivot in C, it is often more effec-

34 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
tive to actually choose the representative pivot randomly, using a distribution
biased towards smaller elements in C. Namely, if we consider C as an ordered
sequence of pivots, then the probability of choosing C[i], where 0 ≤ i < |C|, is
p = 3(1−p ),p = 3. The values p decay exponentially with increasing i.
i 4 i−1 0 4 i
Algorithm 8 Strict weak ordering of a partition class of IID nodes
1: if |P.f(x)|<|Q.f(x)| then return true
2: if |P.f(x)|>|Q.f(x)| then return false
3: W :=[20,...,210]
4: p:=argmin{|P.nbytes−W[i]| | i=0,...,|W|−1}
i
5: q:=argmin{|Q.nbytes−W[i]| | i=0,...,|W|−1}
i
6: m:=argmin{|max_bytes/2−W[i]| | i=0,...,|W|−1}
i
7: if |W[m]−W[p]|<|W[m]−W[q]| then return true
8: if |W[m]−W[p]|>|W[m]−W[q]| then return false
9: if P.nbytes<Q.nbytes then return true
10: if P.nbytes>Q.nbytes then return false
11: return dP <dQ
Once we have the representative pivot, say P, selected, then we may focus
on searching for a node M (as explained above). First we should realize the
following facts which basically justify the approach we take.
– There is no input x to the Target such that the corresponding trace T will
bemappedtoP andthemissingsuccessorofP willbecreated.21 Therefore,
→− →−
if we want to cover P, then we have to escape from P at some node P[k],
where 0≤k <dP −1.
→−
– Although we want to escape from P at the index k, we still want to get
backtothesameinstructionP.id.AlthoughP.idcanbereached,ingeneral,
several completely different way, considering the information we have, it is
→−
reasonable to restrict our search for those paths which are similar to P.
There can be many of such similar paths. They may differ in the numbers
of iterations in loops along the path. There may also be differences in what
pathistakenineachiterationofeachloop.Andobservethatweindeedhave
valuable information about theses paths. Namely, for each IID pivot in the
class C we know the corresponding path. And for each of that path we also
know the loops (entries, exits and bodies) along it.
→−
So, our algorithm is as follows. We first need to go backwards along P (to-
→−
wards the root node) to find the index k where to leave P. Then, we walk
→−
forward in the execution tree from the node P[k] along a path similar to those
21 That is, of course, only true under the assumption the sensitivity analysis did not
under-approximate the sensitive bits of P.

Gray-Box Fuzzing via Gradient Descent and Boolean Expression Coverage 35
in C. We will see, this forward walk is inspired in the Monte Carlo walk used in
games theory. Once we reach an open node M with unexplored success, which
oursimilarpathcontinuesto,thenwestopandM isthenodewewantaninput
generation analysis to be activated with.
Letusnofocusonthecomputationoftheindexk.Itisbasedonthefollowing
observation in our evaluations:
– The value of the branching function of an IID pivot typically depends on
those loops (their iteration counts and interleaving of its path) which are
close to the pivot. Higher the distance from the pivot, lower the chance of
affecting the branching function.
→−
So, we thus choose k as an index of a loop entry along P. And we should prefer
→−
thoseloopentrieswhichareclosetoP.Wecollectallloopentiresalong P using
theAlg.5.Formally,ifloopsistheoutputfromthealgorithm,thenwebuildthe
sequence E = [L.E | E ∈ loops] of loop entries. Then we sort nodes in E by
their depth in the tree in the decreasing order (because we want nodes closer to
P earlier in the sequence). Next, we choose an index i into E randomly using
the same probabilities p assigned to indices, which we used for selection of P
i →−
from C. Lastly, our index k is then dE[i]. Note though, that if P[k] is closed,
→−
then we keep incrementing k until P[k] is not closed. In case the root node of
the tree is closed, we cannot select any node in the tree, and so we return null.
It remains to discuss how we describe a path similar to those in C (used in
theMonteCarlowalk).ThepathisrepresentedbytwomapsF andG,bothfrom
−→
unique IDsof Boolean instructions, i.e., F =G ={N.id.id | N ∈N′∧N′ ∈C}.
GivenanodeN,thevalueF(N.id.id)istheprobability(in[0,1])withwhich
we should move to N.succ[false] (the probability to move to N.succ[true] is
1−F(N.id.id)). The effectivity of the Monte Carlo walk thus highly depends
on these values. We compute them from pivots in C. But not from all. We
consider only this sequence C′ = [N | N ∈ C ∧N.nbytes = P.nbytes]. And
we sort it by the absolute value of the branching function, i.e., by |f(x)|. This
restriction to pivots to those operating on inputs of the same size is important,
because paths to them in the execution tree tend to be highly similar, which
leads to more accurate values in F (than if all pivots were considered). We
ordered C′, because |f(x)| also affects the final probability. Namely, for each
−→
id.id and 0 ≤ i < |C′|, let n (id.id,C′[i]) be the count of nodes along C′[i]
f
with this id.id and the the path continues from them to the false successor.
Similarly, let n (id.id,C′[i]) be the count to true successors. And finally, let
t
F(id.id,C′[i]) = n (id.id,C′[i])/(n (id.id,C′[i])+n (id.id,C′[i])). We then set
f f t
F(id.id) to the average of all these values
{F(id.id,C′[i])+t(F(id.id,C′[0])−F(id.id,C′[i])) | 0≤i<|C′|}
where t = −|C′[i].f(x)|/(|C′[0].f(x)|−|C′[i].f(x)|), only with the nonzero de-
nominator, of course. If this set is empty, ten we put there F(id.id,C′[0]) and
if id.id was further detected to lie inside a loop body, then we also include the

36 Martin Jonáš , Jan Strejček , Marek Trtík, and Lukáš Urban
value0.5.Expressioninsidetheset,togetherwiththeexpressionfort,represent
the solution on the following system
(cid:18) 0 (cid:19) (cid:18) |C′[i].f(x)| (cid:19) (cid:18) |C′[0].f(x)|−|C′[i].f(x)| (cid:19)
= +t
p F(id.id,C′[i]) F(id.id,C′[0])−F(id.id,C′[i])
where the only unknown p represents the element of the set. Due to ordering of
C′, the pivot C′[0] is the one with the smallest |f(x)| and it is thus the closest
node to the coverage of the corresponding Boolean instruction. The processed
pivot C′[i] can be worse, so we interpolate the count F(id.id,C′[i]) along the
line in the right-hand side of the system. The purpose for the addition of 0.5 to
the set in the case id.id being inside loop body is that we actually want to add
variabilityinsideloopbodiesinordertoexplorediversepathsinloopiterations.
The range Rng(G) consists of three random generators:
– G : Generates numbers in [0,1] using the uniform distribution.
uni
– G :ThisgeneratorisinitializedwiththeprobabilityF(id.id)andthecount
1,0
K = (cid:80)|C′|−1n (id.id,C′[i])+n (id.id,C′[i]). The generator then repeats
i=0 f t
the sequence of KF(id.id) numbers 1 and then K −KF(id.id) numbers 0,
forever.
– G : Differs from G such that it first generates the sequence of zeros.
0,1 f,t
The purpose of the last two generators is that the top one has difficulties to
generate the “extreme” sequences of the other two in reasonable time. However,
the extreme sequences are in fact quite common when speaking of paths in a
loop. The mapping of Dom(G) to these generators is straightforward. If id.id
represent a Boolean instruction inside a loop body, then we choose randomly
between all three generators. Othervise, we set G(id.id) to G .
uni
With the maps F and G in hand the Monte Carlo walk proceeds as follows.
LetN bethecurrentnodeinthewalk.ThentheBooleanvalueb=F(N.id.id)<
G(N.id.id)()22representsthedirectioninwhichwewanttocontinueinthewalk.
However, we can move in that direction only if N.succ[b] is a valid non-closed
node.IfwecannotmoveinthedirectionbandN isopen,thenwestop,because
N is the node M we have been searching for. Otherwise, we move into a non-
closed successor node (there has to be one), and we process it the same way.
6.3 Recovery from early termination
Wheneverthe(typed)minimizationfailstoinverttheresultofevaluationofthe
Boolean instruction corresponding to the processed node N, then record the
node together with the current value n of the fuzzing loop’s iteration counter.
Later, when the fuzzing loop is being terminated with the reason that no
node in the tree can be selected for an input generation analysis, then we try to
make the recorded nodes available for the analysis.
So, let us consider our recorded node N. If N.id was covered since it was
recorder, then we do nothing. We also do nothing, if n ≥ N.fn. Otherwise, we
22 If N.id.id̸∈Dom(F), then F(N.id.id) means 0.5 and G(N.id.id) means G .
uni

| Gray-Box | Fuzzing via | Gradient Descent | and Boolean Expression | Coverage | 37  |
| -------- | ----------- | ---------------- | ---------------------- | -------- | --- |
try to make the node available, because the (typed) minimization analysis may
succeednow.ThatisbecausetheinputN.xhaschangedsincetheminimization
run with the node. Although it failed for the previous input it may succeed for
| the current | one.                                                   |     |     |     |     |
| ----------- | ------------------------------------------------------ | --- | --- | --- | --- |
| TomakeN     | available,wesetallfieldssa,ba,andmatofalse.Wealsoclear |     |     |     |     |
its closed state (if marked as such) and lastly we try to insert it into the sets of
| the primary | targets | (see Sec.6.1). |     |     |     |
| ----------- | ------- | -------------- | --- | --- | --- |
7 Optimizer
After the termination of the fuzzing loop we look into generated tests. For each
| test, for | which the termination | result was |     |     | we  |
| --------- | --------------------- | ---------- | --- | --- | --- |
BOUNDARY_CONDITION_VIOLATION
run the Target for the corresponding input again, but this time with all limits
highly extended. If the accepted trace improves the coverage of Boolean in-
struction,thentheacceptedinputisincludedtothefinaltestsuite,ifitactually
| differs from | the input | passed to the Target. |     |     |     |
| ------------ | --------- | --------------------- | --- | --- | --- |
