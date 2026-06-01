---
publish: true
---

Fuzztruction: Using Fault Injection-based Fuzzing
to Leverage Implicit Domain Knowledge
Nils Bars, Moritz Schloegel, Tobias Scharnowski, and Nico Schiller,
Ruhr-Universität Bochum; Thorsten Holz, CISPA Helmholtz Center
for Information Security
https://www.usenix.org/conference/usenixsecurity23/presentation/bars
This paper is included in the Proceedings of the
32nd USENIX Security Symposium.
August 9–11, 2023 • Anaheim, CA, USA
978-1-939133-37-3
Open access to the Proceedings of the
32nd USENIX Security Symposium
is sponsored by USENIX.

Fuzztruction: Using Fault Injection-based Fuzzing to
Leverage Implicit Domain Knowledge
NilsBars∗,MoritzSchloegel∗,TobiasScharnowski∗
NicoSchiller∗,ThorstenHolz‡
∗Ruhr-UniversitätBochum
‡CISPAHelmholtzCenterforInformationSecurity
Abstract to handle the communication: One application generates
Today’sdigitalcommunicationreliesoncomplexprotocols thedata(hereafterreferredtoasgenerator),whiletheother
andspecifications forexchanging structuredmessages and consumes it (called consumer). For example, various pro-
data. Communicationnaturallyinvolvestwoendpoints: One gramsgeneratePDFdocumentsasoutputandcorresponding
generatingdataandoneconsumingit.Traditionalfuzztesting PDF viewers display the result. Another example is cryp-
approachesreplaceoneendpoint,thegenerator,withafuzzer tographiclibraries,whichgenerateencryptedmessagesthat
andrapidlytestmanymutatedinputsonthetargetprogram theircorrespondingcounterpartscanprocess. Fromasecu-
undertest. Whilethisfullyautomatedapproachworkswell rityperspective,theconsumerplaysacrucialrole,asitpro-
forlooselystructuredformats,thisdoesnotholdforhighly cessespotentiallyuntrusteddataandisthusexposedtoattacks.
structuredformats,especiallythosethatgothroughcomplex Fuzztesting(fuzzingforshort),aformofrandomizedtesting,
transformationssuchascompressionorencryption. has proven helpful in efficiently finding software faults in
Inthiswork,weproposeanovelperspectiveongenerating theinputprocessingofconsumerprograms. Pastadvances
inputsinhighlycomplexformatswithoutrelyingonheavy- infuzzingmethodsfocusedonthroughput[1–3],effective-
weightprogramanalysistechniques,coarse-grainedgrammar ness[4–6],andapplicabilitytonewtargetdomains[7–12].
approximation, ora human domain expert. Instead of mu- Evenafteryearsofresearch,anunresolvedchallengeistheef-
tating the inputs fora targetprogram, we injectfaults into fectivegenerationofvalidinputsforcomplexformats,includ-
the data generation program so that this data is almost of ingcryptographicprimitives,compression,andotherkinds
theexpectedformat. Suchdatabypassestheinitialparsing oftrickytransformations.
stagesintheconsumerprogramandexercisesdeeperprogram Priorworksandindustrybestpracticesattemptedtwoap-
states,whereittriggersmoreinterestingprogrambehavior. proaches to tackle this challenge. First, heavy-weight pro-
To realize this concept, we propose a set of compile-time gram analysis techniques, such as symbolic execution and
andrun-timeanalysestomutatethegeneratorinatargeted taintanalysis,ormanualworkarounds,havebeensuggested
manner,sothatitremainsintactandproducessemi-validout- tosolveroadblocks(e.g.,checksumsorhashes)[5,6,13–20].
putsthatsatisfytheconstraintsofthecomplexformat. We Unfortunately, these methods do notscale to complexpro-
haveimplementedthisapproachinaprototypecalledFUZZ- grams. Second,grammar-basedfuzzing[21–29]hasbeenin-
TRUCTIONandshowthatitoutperformsthestate-of-the-art vestigatedtogenerateinputsofaspecificsyntacticalstructure.
fuzzers AFL++, SYMCC, and WEIZZ. FUZZTRUCTION However,theuseofgrammarsdoesnotaddressthecomplex-
findssignificantlymorecoveragethanexistingmethods,espe- ityofapplicationsusingcomplextypesoftransformations,
ciallyontargetsthatusecryptographicprimitives. Duringour especiallycryptographicprimitivesandcompression.
evaluation,FUZZTRUCTIONuncovered151uniquecrashes
Insteadofusingheavy-weighttechniques, grammars, or
(afterautomateddeduplication). Sofar,wemanuallytriaged
manually bypassing these fuzzing roadblocks, we propose
andreported27bugsand4CVEswereassigned.
anovelgenericapproachtogeneratehighlystructuredand
complexinputsforfuzzinginanautomatedway.Morespecif-
1 Introduction ically,weproposetotakeadvantageofthedomainknowledge
thatisalreadyencodedintheapplicationsthatgeneratedata:
Ourmodern digital infrastructure is based on well-defined Intraditionalfuzzingapproaches,thegeneratorisreplaced
messageanddataformats,includingstandardsandspecifica- byafuzzerthatpassesinputdirectlytotheconsumer(repre-
tionsfordataexchange. Thesesystemshaveatleasttwoend- sentingthesystemundertest). Onthecontrary,wedevisea
points,bothofwhichencodethedomainknowledgerequired mechanismtomutatethegeneratorandthenpassitsoutput
USENIX Association 32nd USENIX Security Symposium 1847

totheconsumer. Thecoreideaisthatthismutatedgenerator SYMCC, and WEIZZ. Our results show that our ap-
producesinputsthatmostlyadheretotherequireddataformat proachachievessignificantgainsintermsofcoverage
butintroducessubtledeviationsthatmaytriggerfaultsinthe andnumberofsoftwarefaultsfound.
consumer’s processing logic. For example, an application To foster further research on this topic, we release the
thatsignscryptographiccertificatesknowshowtogeneratea
|     |     |     |     | source code | andevaluation |     | artifacts | of  | FUZZTRUCTION |     | at  |
| --- | --- | --- | --- | ----------- | ------------- | --- | --------- | --- | ------------ | --- | --- |
validsignaturethatcanbeparsedandverifiedbyanyappli- https://github.com/fuzztruction/fuzztruction.
cationdesignedtoverifysuchsignatures(e.g.,browsersor
| cryptographiclibraries). |     | Toenablefuzzingofthesignature |     |     |     |     |     |     |     |     |     |
| ------------------------ | --- | ----------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
verificationlogic,weexploitthefactthatthegeneratorim-
2 FuzzingComplexInputFormats
| plicitlyknowshowtocomputevalidsignatures. |     |     | Weleverage |     |     |     |     |     |     |     |     |
| ----------------------------------------- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
thisknowledgebyslightlymutatingthegenerator’scodeso
|     |     |     |     | Generally | speaking, | fuzzing | means | that | we  | execute | a tar- |
| --- | --- | --- | --- | --------- | --------- | ------- | ----- | ---- | --- | ------- | ------ |
thattheproducedoutputmayviolatethespecificationbutis
getprogramwithnumerousmutatedinputstotriggerunex-
technicallyvalid(i.e.,avalidsignatureorencryption).
|     |     |     |     | pected behavior, |     | thus revealing |     | faults. | To reach | deep | into |
| --- | --- | --- | --- | ---------------- | --- | -------------- | --- | ------- | -------- | ---- | ---- |
Randomlyflippinginstructionbitsinthegeneratorwould
likelynotaffectitsoutput,and—evenworse—itwouldlead thestatespaceoftheprogram undertest, fuzzerstypically
|                                 |     |     |                     | needtogeneratewell-structuredinputs. |     |     |     |     | Inadditiontobeing |     |     |
| ------------------------------- | --- | --- | ------------------- | ------------------------------------ | --- | --- | --- | --- | ----------------- | --- | --- |
| thegeneratortocrashinmostcases. |     |     | Toavoidsuchundesir- |                                      |     |     |     |     |                   |     |     |
well-structured,theseinputsalsoneedtoaccountforcheck-
ablecases,wedevisedacompile-timeanalysistoidentifyop-
sums,compressionalgorithms,orcryptographicprimitives
erationsondataandfilteroutoperationsthatwouldcrashthe
|                              |     |                        |     | that guard | more | in-depth | processing. |     | In practice, |     | both the |
| ---------------------------- | --- | ---------------------- | --- | ---------- | ---- | -------- | ----------- | --- | ------------ | --- | -------- |
| generatorwhentheyaremutated. |     | Wealsoanalyzedata-flow |     |            |      |          |             |     |              |     |          |
efficientidentificationoflogicalunitswithinaninputformat
| dependenciestoavoidredundantmutations. |     |     | Then,weiden- |                                     |     |     |     |     |                   |     |     |
| -------------------------------------- | --- | --- | ------------ | ----------------------------------- | --- | --- | --- | --- | ----------------- | --- | --- |
|                                        |     |     |              | andthesubsequenteffectiveresolution |     |     |     |     | ofobstacleswithin |     |     |
tifywhichpartsofthegeneratoractuallyaffecttheoutputfor
|     |     |     |     | theseunitsposeamajorchallenge. |     |     |     | Fuzzerscurrentlyattempt |     |     |     |
| --- | --- | --- | --- | ------------------------------ | --- | --- | --- | ----------------------- | --- | --- | --- |
theremainingmutationcandidatesandfocusourmutations
onthemostpromisingcandidates. Basedontheseinsights, tosolvethesechallengesviadifferentapproaches,whichwe
outlineinthefollowing.
weinstrumentthegeneratorandjust-in-time(JIT)-compile
| both tracing | and mutation | mechanisms | into it, facilitating |                    |     |     |                                 |     |     |     |     |
| ------------ | ------------ | ---------- | --------------------- | ------------------ | --- | --- | ------------------------------- | --- | --- | --- | --- |
|              |              |            |                       | ExecutionFeedback. |     |     | Gatheringfeedbackontheexecution |     |     |     |     |
efficientfuzzing.
ofthetargetprogramforagiveninputisawell-established
| To demonstrate                               | the | practical feasibility | of the proposed |                                    |     |     |     |     |                  |     |     |
| -------------------------------------------- | --- | --------------------- | --------------- | ---------------------------------- | --- | --- | --- | --- | ---------------- | --- | --- |
|                                              |     |                       |                 | techniqueinmodernfuzzers[1,31,32]. |     |     |     |     | Thisfeedbackpro- |     |     |
| approach,weimplementatoolcalledFUZZTRUCTION. |     |                       | Our             |                                    |     |     |     |     |                  |     |     |
videsameasureofinputqualityandallowsafuzzertorecog-
methodcanbeusedasastand-alonetooloraugmentexisting
|          |                    |             |                  | nizeinputsthatexplorenewprogrambehavior. |          |       |              |     |                 | Bykeeping |     |
| -------- | ------------------ | ----------- | ---------------- | ---------------------------------------- | -------- | ----- | ------------ | --- | --------------- | --------- | --- |
| fuzzers. | In a comprehensive | evaluation, | we show that our |                                          |          |       |              |     |                 |           |     |
|          |                    |             |                  | and further                              | mutating | these | increasingly |     | well-structured |           | in- |
approachnotonlyavoidstheshortcomingsoftraditionalap-
puts,afuzzerderivesinputsthatmatchtheexpectedformats
proachesfortargetsthatconsumecomplexinputformats,but
|     |     |     |     | toagreaterdegreeovertime. |     |     | Gatheringexecutionfeedback |     |     |     |     |
| --- | --- | --- | --- | ------------------------- | --- | --- | -------------------------- | --- | --- | --- | --- |
alsooutperformsfuzzerssuchasSYMCC[6],WEIZZ[30],
|           |                                       |     |     | (and most         | prominently, |            | coverage | feedback)  |     | is attractive | as  |
| --------- | ------------------------------------- | --- | --- | ----------------- | ------------ | ---------- | -------- | ---------- | --- | ------------- | --- |
| and AFL++ | [1]: Onaverage,wefind21%morecoverage, |     |     |                   |              |            |          |            |     |               |     |
|           |                                       |     |     | it is generically |              | applicable | and      | introduces | a   | low run-time  |     |
bothfortargetsthatheavilyusecryptographicprimitives(up
|                                       |                |                                  |                 | overhead.                                | Infact,afterthiscoverage-guidedtechniquewas |                    |         |        |      |            |         |
| ------------------------------------- | -------------- | -------------------------------- | --------------- | ---------------------------------------- | ------------------------------------------- | ------------------ | ------- | ------ | ---- | ---------- | ------- |
| to 70%                                | more coverage) | and for extensively              | tested targets  |                                          |                                             |                    |         |        |      |            |         |
|                                       |                |                                  |                 | introducedin                             | AFL                                         | [31],              | fuzzers | have   | been | able to    | explore |
| (upto23%morecoverageonobjdump).       |                |                                  | Beyondcoverage, |                                          |                                             |                    |         |        |      |            |         |
|                                       |                |                                  |                 | common,                                  | more                                        | loosely-structured |         | binary | file | formats    | and     |
| we findmore                           | than five      | times the numberof(deduplicated) |                 |                                          |                                             |                    |         |        |      |            |         |
|                                       |                |                                  |                 | identifyalargenumberofbugsasaresult[33]. |                                             |                    |         |        |      | Eventhough |         |
| crashinginputscomparedtothesefuzzers. |                |                                  | Duringoureval-  |                                          |                                             |                    |         |        |      |            |         |
itisnotdirectlyavailabletothefuzzer,theexecutionfeedback
| uation, we | uncovered151 | unique crashes | (afterautomated |     |     |     |     |     |     |     |     |
| ---------- | ------------ | -------------- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
providesasidechanneltothedomainknowledgeencodedin
| deduplication). | Wemanuallytriagedandreported27bugs |     |     |     |     |     |     |     |     |     |     |
| --------------- | ---------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
theprogramundertest.
| inacoordinatedwaytothedevelopers. |     |     | Uptonow,4CVEs |     |     |     |     |     |     |     |     |
| --------------------------------- | --- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- |
wereassigned.
|     |     |     |     | Grammars. | Insteadoftryingtoextractdomainknowledge |     |     |     |     |     |     |
| --- | --- | --- | --- | --------- | --------------------------------------- | --- | --- | --- | --- | --- | --- |
Insummary,wemakethefollowingmaincontributions:
|     |     |     |     | from a | target program, |     | a human | expert | can | provide | pre- |
| --- | --- | --- | --- | ------ | --------------- | --- | ------- | ------ | --- | ------- | ---- |
• Weproposeanovelfuzzingmethodthatautomatically existingdomainknowledgetothefuzzer.Intheseapproaches,
leveragesthedomainknowledgeingeneratorapplica- thefuzzeriseitherprovidedwithinformationaboutthein-
tionstoimprovefuzzingwithoutrelyingonadvanced put structure of the target program (e.g., a grammar) or a
andexpensiveprogramanalysistechniques. structure-awarelogicforinputgenerationisintegratedinto
• Wedemonstratethegenericcapabilitiesoftheapproach thefuzzer[21–29]. Thispreciseknowledgeaboutthetarget
byfuzzingevencomplexcryptographicprocedures,such format allows the fuzzer to generate inputs that fulfill the
as the parsing and validation of encrypted RSA keys, target’s requirements about the input structure. The main
automaticallyandwithoutcustom-craftedseeds. drawbackoftheseapproachesliesinthefactthatwhilethey
• Weimplementandevaluateourprototype,calledFUZZ- allowgeneratinghigh-qualityinputs,theyrequirepre-existing
TRUCTION,againstthestate-of-the-artfuzzersAFL++, knowledgeabouttheprogramundertest.
| 1848    32nd USENIX Security Symposium |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

| AutomatedGrammarApproximation. |           |     |          | Insteadofrequir- |     | 3 Design |     |     |     |
| ------------------------------ | --------- | --- | -------- | ---------------- | --- | -------- | --- | --- | --- |
| ing domain                     | knowledge | in  | the form | of a grammar     | a   | priori,  |     |     |     |
Inthiswork,wetakeanorthogonalapproachbychangingthe
| there are | techniques | to automate | the | process | ofgenerating |     |     |     |     |
| --------- | ---------- | ----------- | --- | ------- | ------------ | --- | --- | --- | --- |
approximationsofthegrammarembeddedintothetarget.Dif- perspectiveoffuzzing: Weproposetofocusonthegenera-
ferentapproachesexistwhichtargettext-based[34]orbinary torapplicationsthatproducetheinputtothetargetprogram
|         |          |          |           |               |     | under test. | Our approach uses | a simple | yet powerful idea: |
| ------- | -------- | -------- | --------- | ------------- | --- | ----------- | ----------------- | -------- | ------------------ |
| formats | [30,35]. | Bothhave | in common | thattheytryto |     | in-         |                   |          |                    |
fer the structure of a provided input by identifying logical instead of directly mutating the input to create a new test
case,wemutatethegeneratorapplicationanduseitsoutput
unitswithinthedataformat,e.g.,chunks,tokens,orfields.
Thisenablesstructure-awaremutationsthatallowthefuzzer asafuzzingtestcaseforthetargetundertest. Thisway,we
tomodify,insert,remove,orreplacesuchlogicalunits. Al- can(implicitly)leveragedomainknowledgeandovercome
complexconstraintswithoutsufferingfromtheshortcomings
thoughthesetechniquessidesteptherequirementforcrafting
thegrammarmanuallyandsuccessfullylocatelogicalunits, ofheavyweighttechniquesormanualapproaches.
Ourobservationisthatgeneratorprogramsgenerallypro-
theprocessofapproximatingisinherentlycoarse-grained.
ducewell-formedoutputs,whichisindispensabletoensure
|     |     |     |     |     |     | interoperability. | Byselectivelyinjectingfaultsintothesegen- |     |     |
| --- | --- | --- | --- | --- | --- | ----------------- | ----------------------------------------- | --- | --- |
HeavyweightFeedbackandAnalyses. Beyondtheefforts eratorprograms,theyproducealmostwell-formedoutputs,
described above, which improve the fuzzer’s capability to i.e.,theymayviolatethespecificationsinsubtleways. This
generate highly structuredinputs, an orthogonal line ofre- allowsustoproducehigh-qualitytestcasesfortherespective
search has focused on solving typical fuzzing roadblocks, consumerprogram.Forexample,supposethatweinjectfaults
suchaschecksumsorcryptographicprimitives,withoutre- intoinstructionsthatmanipulate(i.e.,readorwrite)partially
quiring a grammar. As a more direct way of extracting processeddata,whichwillsubsequentlybecryptographically
the domain knowledge encoded in the target program, re- signed. Inthiscase,wecanproducemutatedbutvalidinputs
centapproachesemploysophisticatedprogramanalysesand withrespecttotheenvelopingsignature. Crucially,thesein-
| moreheavyweightfeedbacktypes. |     |     |     | Theseapproachestackle |     |     |     |     |     |
| ----------------------------- | --- | --- | --- | --------------------- | --- | --- | --- | --- | --- |
putsarenotdiscardedearlyduringsignaturevalidation,but
thedeficitsof(lightweight)feedback-drivenfuzzersbyaid- reachdeeperprogramlogicintheconsumer.
ingthefuzzerin solvingconstraintswithin datastructures, Figure1presentsahigh-leveloverviewofourapproachand
e.g., via taint tracking [17–19] or concolic/symbolic exe- showshowtheindividualcomponentsofourdesigninteract.
cution [5,6,13–15]. Using taint tracking, the fuzzer can Onahighlevel,wewantthegeneratorapplicationtoproduce
backtrace which parts of an input affect a specific branch diversetestcasesthatwecansupplytoouractualfuzzingtar-
condition. Giventhissemanticinsight,fuzzerscanconcen- get,theconsumer. Tothisend,thefuzzingschedulerselects
tratetheireffortsonmutatinginputpartsthatarerelevantto (cid:202)oneormultiplemutation(s)forthegeneratorapplication
overcomespecificconstraints. Usingsymbolicorconcolicex- (e.g., changingavaluestoredbyamovinstruction)and, if
ecution,fuzzerscancomputevaluesthatarerequiredtosolve needed,aseedfileprocessedbythegenerator. Next,(cid:203)the
conditionsorintegritychecksor, moregenerallyspeaking, selected mutation(s) are applied to the generator. The mu-
tatedgeneratornowprocessestheinput(cid:204)andproduces(cid:205)a
| exercise | all paths | in a given | program. | While | heavyweight |     |     |     |     |
| -------- | --------- | ---------- | -------- | ----- | ----------- | --- | --- | --- | --- |
feedback-drivenfuzzershaveproveneffectiveingenerating slightlymutatedoutput. Finally,theschedulerpasses(cid:206)the
well-structuredinputs,theysufferfromnewchallengesand generatedoutputtotheconsumer,andcoveragefeedbackis
limitations. The main limitation is that they are relatively collected(cid:207). Ifthetestcasetriggersinterestingbehavior(new
slow,failtoscaletolargetargetprograms,andrequirerun- coverage),thepairofmutatedgeneratorprogramandinput
timeenvironmentsthatspecify, e.g., sideeffectsoflibrary fileisenqueuedforfurthermutationinsubsequentrounds.
| functions[4]. | Furthermore,complexconstraintsimposedby |     |     |     |     |     |     |     |     |
| ------------- | --------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
cryptographicprimitivessuchashashfunctionsorsignatures 3.1 Generator
cannotbesolvedduetotheircomputationalcomplexity.
Inessence,thegeneratorcanbeconsideredaseedgenerator
|     |     |     |     |     |     | forproducinginputstailoredtothefuzzingtarget,     |     |     | thecon- |
| --- | --- | --- | --- | --- | --- | ------------------------------------------------- | --- | --- | ------- |
|     |     |     |     |     |     | sumer. Whilecommonfuzzingapproachesmutateinputson |     |     |         |
Insummary,wefindthatcurrentapproachestogenerating
theflythroughbit-levelmutations,wemutateinputsindirectly
| complex, | highly | structured | input rely | eitheron | a grammar |     |     |     |     |
| -------- | ------ | ---------- | ---------- | -------- | --------- | --- | --- | --- | --- |
byinjectingfaultsintothegeneratorprogram.Moreprecisely,
| provided | by a human | expert, | which | is effective | but | costly, |     |     |     |
| -------- | ---------- | ------- | ----- | ------------ | --- | ------- | --- | --- | --- |
weidentifyandmutatedataoperationsthegeneratorusesto
orapproximatethegrammar(lesscostlybutlesseffective).
produceitsoutput.
Notethatneitherofthemcanhandlemutationsofcomplex
data. Other methods use heavyweight techniques that do GeneratorRequirements. Tofacilitateourapproach,we
not require a grammar but are inefficient and do not scale. requireaprogramthatgeneratesoutputsthatmatchtheinput
Noneofthestate-of-the-artapproachescanleverageexisting formatthefuzzingtargetexpects.Mostgeneratorapplications,
domainknowledgeinanautomatedandeffectivemanner. suchasimageconverters,requirefilesthatcanbeconverted
| USENIX Association |     |     |     |     |     |     | 32nd USENIX Security Symposium    1849 |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- |

Ê
morethanoncesincetheinitialmodificationcanalready
... ... produceanypossiblevalue;intheworstcase,thesecond
changecancancelthechangesofthefirst. Thisdoesnot
PNG PNG
Seeds Mutations 5 Consumer applyforpartialdependencies.
Toaddressthesechallenges,wedesignourcompilerpass
Ï
Ë 0110 0 0 10 01 1 0 0101 0 0 1 1 1 10 00 10 1 1 00 01 0 01 00 1 11 00 1 001 0 11 1 10 0 00 00 1 1 1 0 1 01 101 0 00 0 10 11 01 0 111 0101 00 00 00 11 01 00 00 0 10 011 01 0 00 10 1 1 1 011 1111 1 1110 1 1 1 000 010 1 1 01 1 1100 1 0 101 11 0 01 01 0 11 011 11 00 00 1 101 00 00 1 1 00 00010 10 11 01 100 11 000 100 0 111 000 0 110 10 00 11 1 1 1 11 1 1 001111 t
a
o
nd
on
a
l
v
y
o
i
i
n
d
st
p
r
o
u
i
m
nt
e
e
n
r
t
t
i
y
n
p
st
e
r
s
u
.
c
A
tio
d
n
d
s
it
l
i
o
o
a
n
d
a
i
l
n
ly
g
,
o
w
r
e
st
u
o
s
r
e
in
t
g
he
va
d
l
a
u
t
e
a-
ty
fl
p
o
e
w
s
Î
Ì3 4 informationavailabletothecompilertoinstrumentonlythe
Generator Í Scheduler firstinstanceofadatavaluebeingmodified. Unfortunately,
staticanalysisduringcompile-timefallsshortindetermining
Figure1:High-leveloverviewofourapproachforageneratoranda whetheraparticularinstructionwillhaveasignificantimpact
consumerprogram.Noteworthy,thegeneratormayproduceoutputs (ifatall)ontheapplication’soutput. Consequently,before
((cid:205))thatslightlyviolatethespecification.
startingtheactualfuzzingprocess,ourdesignbudgetsfora
lightweightidentificationandpruningphaseduringruntime,
aprocesswedescribeinthefollowing.
tothetargetformat. Thus,incontrasttotraditionalfuzzing
approaches,theinitialseedfilesareinputsforthegenerator Instrumentation Site Pruning & Impact Analysis. We
insteadoftheconsumer. Basedonthetypeofgenerator,seed haveobservedthatallowingthefuzzertoinjectfaultsatall
inputs may not be required. This applies, for instance, to potentialinstrumentationsitesleadstomanyineffectivemuta-
generatorsofcryptographickeys. tions. Forexample,agiveninputfileexercisesonlyaspecific
pathinthe(potentiallymutated)generator’scode,whichcom-
Dataoperations. Onecorechallengeformutatingthegen-
monly only includes a fraction of all instrumentation sites.
eratoristoidentifyandmutatedataoperations. Weconsider
Wecallthesesitesdeadw.r.t. aspecificinputand(mutated)
anyinstructionreadingfromorwritingtomemoryasadata
version ofthegenerator. Even alive(i.e., notdead)instru-
operation. The underlying insight is that output generated
mentationsitesmayhavenoactualimpact. Thismeansthat,
by a program is stored in memory at one point or another
foragivencombinationofinputandgeneratorversion,inject-
inalmostallcases,especiallyforthetypeofcomplexdata
ingafaultintothisinstrumentationsitewillnotleadtoany
formats that we target in this work. For a particular input
coveragechangeintheconsumer.
totheprogram,wesayitcoversthedataoperationwhenit
Intuitively, tomaximizethenumberofeffectivefaultin-
executestherespectiveoperation(anessentialrequirement
jections, we want to avoid mutating dead instrumentation
for a mutation to have any effect). Moreover, we say that
sites andones withouta visible impact. Consequently, the
the data operation is mutated when the underlying data is
goal of our pruning and impact analysis phase is twofold:
modifiedwithrespecttotheoriginalprogram.
Foreachcombinationofseedinputand(potentially)mutated
Toreliablyidentifyalldataoperationswithinaprogram,
generator,wefirstaimtoidentifyandremovedeadinstrumen-
we design a compiler pass that allows us to instrument all
tationsitesandthenanalyzetheimpactoftheremainingones.
loadandstoreoperations. Theseoperationsareubiquitousin
Tothisend,wetracetheexecutionandkeeponlythealive
programs,resultinginmanypotentiallyinterestingoperations.
instrumentation sites. Additionally, to analyze the impact
Consequently,wecannotinstrumenteveryinstance. Instead,
ofaliveinstrumentationsites,weobservewhetherinjecting
we must identify a subset of data operations that gives us
afaultintotheunderlyingdataoperationyieldsadifferent
maximumcontrolovertheprocesseddatawhileminimizing
code coverage in the consumer. This stage results in a list
adverseruntimeeffects,suchascrashes. Besidestheirsheer
ofaliveandimpactfulinstrumentationsitesperseedfileand
number,severalfactorsdetermineaninstruction’srelevance:
(potentially)mutatedgeneratorapplication. Furthermore,we
• Impact: Doesthisparticularinstructionmodifyrelevant
recordadditionalinformation,suchasthenumberoftimes
data, i.e., data that has an observable impact on the
aninstrumentationsiteisexercised,forlateruse,e.g.,when
output? Modifyinginstructionsunrelatedtothegener-
generatingmutations.
ateddatahasnobenefittowardsthegoalofproducing
interestinginputfilesforourfuzzingtarget. Mutations. Togenerateslightlycorruptinputs,wemutate
• Type: Doesthemodifieddatarepresentavaluethatis thegenerator: Werandomlyselectaninputforthegenerator
likelytocauseacrashofthegeneratorifitismutated, and one or more instrumentation sites. We then apply bit-
e.g.,becauseitisafunctionpointer?Modifyingpointers levelmutationstothedatavaluesprocessedattherespective
insteadofactualvaluesispronetocrashtheapplication instrumentationsites. Asthesesitescanbevisitedmultiple
insteadofproducinginterestingvalues. timesduringprogramexecution,e.g.,inloops,wecaneither
• Data-flowDependency:Doesthedatadependonavalue alwaysapplythesamemutationoroptformorefine-granular
thathasalreadybeenmodifiedbyearlierinstructions? controlintheformofindependentmutationsthatdifferfor
Intuitively, it is undesirable to modify the same data eachvisit. Forthelatter,weneedaprioriknowledgeofhow
1850 32nd USENIX Security Symposium USENIX Association

oftenaparticularinputwillvisitaninstrumentationsite. This 2. AddPhase: Wepickmultipleinstrumentationsitesand
information is conveniently available from our pruning & apply mutations. We prefersites thathave previously
impactanalysispass. Thisallowsustospecifyalistofmuta- successfullyyieldednewcodecoverageintheconsumer.
tionsforeachinstrumentationsite. Eachvisitofthemutation Thisphaseaddsnewmutationstothequeueentry,ex-
siteconsumesoneofitsmutations,untilallmutationsfora tendingtheinstancesofmutateddataoperations.
| giveninstrumentationsiteareconsumed. |     |     | Intherarecasea |                 |                                |     |     |     |
| ------------------------------------ | --- | --- | -------------- | --------------- | ------------------------------ | --- | --- | --- |
|                                      |     |     |                | 3. MutatePhase: | Weapplyafixednumberofmutations |     |     |     |
mutationmodifiescontrolflow,suchthataninstrumentation
toallmutateddataoperationsoftheselectedqueueentry.
| site is visited | more often | than observed | during the impact |     |     |     |     |     |
| --------------- | ---------- | ------------- | ----------------- | --- | --- | --- | --- | --- |
Incontrasttotheadd-phase,thisphasedoesnotaddany
analysis,wecannotapplyanyfurthermutationbutreturnthe
unmodifieddatavalue. newdataoperationsbutmutatesexistingones.
|     |     |     |     | 4. CombinePhase: | Foreachmutateddataoperation,we |     |     |     |
| --- | --- | --- | --- | ---------------- | ------------------------------ | --- | --- | --- |
inspectwhetherotherqueueentrieshavemutateditas
3.2 Consumer
|     |     |     |     | well. If | so, we try | their mutations. | This | is similar to |
| --- | --- | --- | --- | -------- | ---------- | ---------------- | ---- | ------------- |
Thegenerator’scounterpartistheconsumer: Asthe target splicing,knownfromtraditionalfuzzing,andallowsto
ofourfuzzingcampaign, itconsumestheinputsgenerated benefitfrommutationsthathavealreadyprovedtoaffect
| bythegenerator. | Similartotypicalfuzzingtargets, |     | weim- | coverage. |     |     |     |     |
| --------------- | ------------------------------- | --- | ----- | --------- | --- | --- | --- | --- |
posenospecificrestrictionsorlimitationsontheconsumer.
Ifweobservenewcodecoverageduringthemainfuzzing
Sinceweusecoveragefeedbacktoguideourmutationsinthe
|     |     |     |     | loop, we needto | execute | the calibration | phase | forthe new |
| --- | --- | --- | --- | --------------- | ------- | --------------- | ----- | ---------- |
generator,theconsumermustprovideaninterfacetoretrieve
queueentry,whichiscreatedtorepresentthecombinationof
coverageinformation(viainstrumentingthesourcecodeor
inputandmutationsthatyieldedthenewcoverage.
fromemulation).
Ourqueueentryselectionalgorithmissimilartotheone
|     |     |     |     | usedbyAFL. | Bothusetheconceptofnoveltysearch,i.e., |     |     |     |
| --- | --- | --- | --- | ---------- | -------------------------------------- | --- | --- | --- |
3.3 Scheduler wekeepinputsbasedonwhethertheyyieldednewcodecov-
erage. Furthermore,weapplyasimilarfavoringschemethat
Thelastcomponentisthescheduler,whichorchestratesthe
prioritizesaminimalsetofinputscoveringamaximumof
interactionofthegeneratorandtheconsumer. Itgovernsthe the code in the consumer. The main difference to AFL is
fuzzingcampaign,anditsmaintaskistoorganizethefuzzing thatwepreferqueueentriescontainingrarelyobserveddata
loop. Theschedulercontainsthefollowingcomponents:
operations;AFLhasnosimilarconceptgiventhatthemethod
doesnotobservedataoperationsatall.
| Queue. | Theschedulermaintainsaqueuecontainingqueue |     |     |     |     |     |     |     |
| ------ | ------------------------------------------ | --- | --- | --- | --- | --- | --- | --- |
entries. Eachentryconsistsoftheseedinputpassedtothe
generator(ifany)andallmutationswhichhavebeenapplied 3.4 CombinedFuzzing
| tothegenerator. | Eachsuchqueueentryrepresentsasingle |     |     |     |     |     |     |     |
| --------------- | ----------------------------------- | --- | --- | --- | --- | --- | --- | --- |
Comparedtotraditionalfuzzerdesigns,ourinputgeneration
| test case. | In traditional | fuzzing, such a | test case would be |                  |                   |     |         |               |
| ---------- | -------------- | --------------- | ------------------ | ---------------- | ----------------- | --- | ------- | ------------- |
|            |                |                 |                    | methodis slower: | insteadofflipping |     | a byte, | the generator |
representedasasinglefile.
programismutatedandexecutedtoproduceaninput.Further-
Phases. Themainfuzzingloopissplitintomultiplephases, more,AFL-basedfuzzersarecapableofsplicingorsplitting
inputs—operationsthatageneratortypicallydoesnotexpose.
| see Algorithm | 1. Depending | on the phase | type, the steps |     |     |     |     |     |
| ------------- | ------------ | ------------ | --------------- | --- | --- | --- | --- | --- |
withinaphaseareperformedexactlyonceforeachnewqueue Tocompensateforthesemissingoperationsandtheperfor-
(calibration phase) mance impact, ourapproachcan be usedin tandem witha
| entry |     | or several times | during fuzzing. |                               |     |     |                       |     |
| ----- | --- | ---------------- | --------------- | ----------------------------- | --- | --- | --------------------- | --- |
|       |     |                  |                 | traditionalfuzzersuchasAFL++. |     |     | Thisapproachissimilar |     |
Uponlaunchingafuzzingcampaign,allseedfilesareadded
tothequeueandthuscalibrated. tofuzzerssuchasQSYM[5],SYMCC[6],orDRILLER[14],
whichuseAFLforregularfuzzingandaugmentitbyprovid-
1. CalibrationPhase:Wepasstheinputtothe(potentially ingnewinputsthatsolvefuzzingroadblockswhichcommon
|     |     |     |     | mutationscannotaddress. |     | Inthesamevein,weproposean |     |     |
| --- | --- | --- | --- | ----------------------- | --- | ------------------------- | --- | --- |
mutated)generatorandrecordtheinstrumentationsites
approachfocusingonageneratorapplicationtoproducein-
visitedduringexecution(instrumentationsitepruning).
Foreachinstrumentation site ofthe target, we further terestinginputsthatunlocknew, deeperstatespaceforthe
traditionalfuzzer.
assessitsimpactonthecoverageintheconsumer(im-
pactanalysis).Thisinformation,alongsidetheinputand
mutations,isstoredwithinthequeueentry. Importantly, 4 Implementation
thisphaseispartoftheregularfuzzingiteration.
|     |     |     |     | We implement | our design | in a | prototype | called FUZZ- |
| --- | --- | --- | --- | ------------ | ---------- | ---- | --------- | ------------ |
Duringthemainfuzzingloop,wethenrepeatedlypickone TRUCTIONthatconsistsofabout14,000linesofRustcode.
queueentryandselectoneofthefollowingphases: Nextwediscussimplementationaspects.
| USENIX Association |     |     |     |     | 32nd USENIX Security Symposium    1851 |     |     |     |
| ------------------ | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

| Algorithm | 1:  |     |     |     |     | (2)themutationofdataoperationsthemselves. |     |     |     |     |     |
| --------- | --- | --- | --- | --- | --- | ----------------------------------------- | --- | --- | --- | --- | --- |
Simplifiedalgorithmrepresentingourap-
proach.Beforethemainfuzzingloopstarts,allseedinputs
|     |     |     |     |     |     | InstrumentationSitePruningandImpactAnalysis. |     |     |     |     | Us- |
| --- | --- | --- | --- | --- | --- | -------------------------------------------- | --- | --- | --- | --- | --- |
arecalibrated.Thisincludestheinstrumentationsiteprun-
ingtheJITcompiler,wefacilitatetheanalysisphasebyin-
ingandanalysisofthedataoperations’impactw.r.t. the jecting a callback to a custom logging function into each
| target’s (i.e.,consumer’s) |     |     | coverage. | Finally, | the fuzzing |                      |                                   |     |     |     |     |
| -------------------------- | --- | --- | --------- | -------- | ----------- | -------------------- | --------------------------------- | --- | --- | --- | --- |
|                            |     |     |           |          |             | instrumentationsite. | Basedonthesecallbacks,wedetermine |     |     |     |     |
loopisexecuteduntilitisstopped.
|     |     |     |     |     |     | whichinstrumentationsitesarealive. |     |     | Asanadditionalpiece |     |     |
| --- | --- | --- | --- | --- | --- | ---------------------------------- | --- | --- | ------------------- | --- | --- |
Input:Seeds
| 1   |     |     |     |     |     | ofmetadata,wealsocounthowoftentheyarevisited(i.e., |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | -------------------------------------------------- | --- | --- | --- | --- | --- |
2 Q←create_queue() executed)foraspecificinput. Theseexecutioncountsallow
forallseed∈Seedsdo thefuzzingloopoftheschedulertodeterminehowmanydata
3
Q←Q∪calibration_phase(seed)
| 4             |     |     |     |     |     | operationscanbemutatedateachinstrumentationsite. |     |     |     |     |     |
| ------------- | --- | --- | --- | --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- |
| 5 whileTruedo |     |     |     |     |     | Mutations.                                       |     |     |     |     |     |
ThesecondtypeofJITstubimplementstheap-
entry←select_next(Q)
| 6       |        |        |     |     |     | plicationofmutationstodataoperations.         |     |     |     | InFUZZTRUCTION, |     |
| ------- | ------ | ------ | --- | --- | --- | --------------------------------------------- | --- | --- | --- | --------------- | --- |
| choice∈ |        | {1..3} |     |     |     |                                               |     |     |     |                 |     |
| 7       | Random |        |     |     |     | weimplementamutationonadataoperationbyXORinga |     |     |     |                 |     |
8 switchchoicedo
|     |     |     |     |     |     | bitmaskintoitsdataoperand. |     | Forloadoperationstheinstru- |     |     |     |
| --- | --- | --- | --- | --- | --- | -------------------------- | --- | --------------------------- | --- | --- | --- |
case1do
| 9   |     |     |     |     |     | mentationsiteisplacedaftertheoperation,whileforstores |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ----------------------------------------------------- | --- | --- | --- | --- | --- |
10 result←add_phase(entry) thesiteisplacedbeforethedataoperation. Figure2shows
11 case2do how a store operation is mutated based on a provided list
result←mutate_phase(entry) ofbitmasks(mutationmasks). BeyondXORing,wecould
12
|     | case3do |     |     |     |     | implementotheroperationstomutatethedata,e.g.,setting |     |     |     |     |     |
| --- | ------- | --- | --- | --- | --- | ---------------------------------------------------- | --- | --- | --- | --- | --- |
13
|     |                               |     |     |     |     | ittospecificvalueorincrementing/decrementingit. |     |     |     |     | Thisis   |
| --- | ----------------------------- | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- | -------- |
| 14  | result←combine_phase(entry,Q) |     |     |     |     |                                                 |     |     |     |     |          |
|     |                               |     |     |     |     | analogoustomutationsusedintraditionalfuzzing.   |     |     |     |     | However, |
15 ifcrash_or_new_coverage(Q,result)then thesemutatorsempiricallyoftenyieldonlyfewcoverage,thus
|     | Q←Q∪calibration_phase(result) |     |     |     |     | wedonotimplementthem. |     |     |     |     |     |
| --- | ----------------------------- | --- | --- | --- | --- | --------------------- | --- | --- | --- | --- | --- |
16
Weusethenumberofobservedexecutionsofeachinstru-
mentationsitewecollectedduringtracingasahinttowards
howmanymutations(i.e.,bitmasks)canbeinsertedforeach
| Generator. | Toinstrumentthegenerator,wedevelopacom- |     |     |     |     |                      |                                  |     |     |     |     |
| ---------- | --------------------------------------- | --- | --- | --- | --- | -------------------- | -------------------------------- | --- | --- | --- | --- |
|            |                                         |     |     |     |     | instrumentationsite. | Wealsokeeptrackoftheinstrumenta- |     |     |     |     |
pilerpassforLLVMwhichidentifiesalldataoperationsand
tionsitesforwhichmutationsyieldednewcodecoveragein
| preparesthemformutation. |     |     | WeuseanexperimentalLLVM |     |     |     |     |     |     |     |     |
| ------------------------ | --- | --- | ----------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
theconsumer,prioritizingthemwhilesubsequentlypicking
featurecalledstackmapstocreateaninstrumentationsitefor
instrumentationsitestomutate.
| eachdataoperation. |     | Inessence,thismeansthatthecompiler |     |     |     |     |     |     |     |     |     |
| ------------------ | --- | ---------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
recordsthelocationofinstructionargumentsduringruntime, Generator Stability. Given thatwe mutate the generator
e.g., in which registeran argument is passed to a store in- application,anddespitetheanalysispassesdescribedinSec-
| struction | (stack map | record). | In  | conjunction | with another |            |               |                 |     |       |          |
| --------- | ---------- | -------- | --- | ----------- | ------------ | ---------- | ------------- | --------------- | --- | ----- | -------- |
|           |            |          |     |             |              | tion 3, we | riskmodifying | the generatorin |     | sucha | way that |
LLVMfeaturecalledpatchpoints,whichplacespadding(in input causes it to crash instead of producing an output. If
theformofnopinstructions)atthepositionofastackmap wedetectsuchacase,weremovetheoffendinginstrumen-
record, this allows us to inject arbitrary code that mutates tationsitefromthesetofsitesthatarepickedformutation.
theoperandsofeachdataoperation. WeJust-In-Time(JIT) Whilethismayappearconservative(astherecanexistother
| compile | dynamically | generated |     | JIT stubs | into the padding |     |     |     |     |     |     |
| ------- | ----------- | --------- | --- | --------- | ---------------- | --- | --- | --- | --- | --- | --- |
mutationsthatwouldnotcrash),weempiricallyfounditis
providedbypatchpoints. Tokeeptherequiredpaddingsize very unlikely that a crashing instrumentation site recovers.
smallandpredictable,weoptfortrampolinesthatcallinto Similarly,wedetectstallsinthegeneratorbysettingatimeout
codethatweallocateinaseparate,executablesection.
ofafewmillisecondsandhandletheminthesamewayas
Asstackmapsandpatchpointsareanexperimentalfeature generatorcrashes. Furthermore,topreventgeneratorsfrom
ofLLVM,theydonotcorrectlyhandleallcornercases(such negatively affecting the host filesystem, we jail them such
asvectoroperations).WedevelopedpatchesforLLVM11.0.1 thattheycannotmodifyfilesbeyondtheiroutput.
andLLVM12.0.1tointroducethemissingsupport1.Beyond
|     |     |     |     |     |     | Consumer. | To collectcode | coverage |     | from | the consumer, |
| --- | --- | --- | --- | --- | --- | --------- | -------------- | -------- | --- | ---- | ------------- |
implementingtheinstrumentationpass,weinsertaruntime
|     |     |     |     |     |     | weusetheAFL-compatibleforkserverinterface. |     |     |     |     | Ifsource |
| --- | --- | --- | --- | --- | --- | ------------------------------------------ | --- | --- | --- | --- | -------- |
componentthatimplementsaforkservertoquicklyexecute
|     |     |     |     |     |     | code is available, | we  | apply AFL’s | compile-time |     | coverage |
| --- | --- | --- | --- | --- | --- | ------------------ | --- | ----------- | ------------ | --- | -------- |
multipleinputsanddeveloptheJITcompilertoapply,remove,
|              |     |        |        |             |              | instrumentation. | Otherwise,wecanfallbacktoQEMUuser |     |     |     |     |
| ------------ | --- | ------ | ------ | ----------- | ------------ | ---------------- | --------------------------------- | --- | --- | --- | --- |
| and generate | JIT | stubs. | We use | these stubs | to implement |                  |                                   |     |     |     |     |
modeinstrumentationforbinary-onlyprograms.
| twotypesoffunctionality: |     |     | (1)Thetracingrequiredforthe |     |     |     |     |     |     |     |     |
| ------------------------ | --- | --- | --------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
instrumentationsitepruningandimpactanalysisphase,and Toavoidunnecessaryexecutionsoftheconsumer,wehash
theinputs,i.e.,thedataproducedbythegenerator,andonly
1Wereleasethepatchesalongsideourcode. executethosenottestedbefore.
| 1852    32nd USENIX Security Symposium |     |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

...
Target Data Operation prototype implementation of FUZZTRUCTION against two
othermethods,SYMCC[6]andWEIZZ[30],representingthe
Instrumentation
stateoftheartregardingheavyweightprogramanalysisand
mov eax, [ebx] approximationofinputstructure. Noneofthefuzzersrequire
| nop |     |     | anyprecomputation. |     |     |     |
| --- | --- | --- | ------------------ | --- | --- | --- |
...
| nop | 0x0 0x4       | 0x8 0xC       |                       |     |                        |     |
| --- | ------------- | ------------- | --------------------- | --- | ---------------------- | --- |
|     |               |               | 1)FUZZTRUCTION-NOAFL. |     | Thisstand-alonevariant |     |
| nop | Offset Length | Mutation Mask |                       |     |                        |     |
... ofFUZZTRUCTIONisnotpairedwithAFL++,butinsteadre-
liessolelyontheinputsproducedbythegeneratorapplication.
| push ... |     |     | Asaconsequence,ithasnoaccesstotraditionalmutations, |     |     |     |
| -------- | --- | --- | --------------------------------------------------- | --- | --- | --- |
mov ebx, mutation_base
call stub_n especiallythefreedomofsplicingandsplittinginputs.
mov eax, [ebx]; offset into msk
pop ...
mov eax, [ebx+eax+8]; load mask 2)AFL++. ThesecondbaselineisAFL++[1](version
| mov [ecx], edx | xor edx, eax ; apply mask |     |     |     |     |     |
| -------------- | ------------------------- | --- | --- | --- | --- | --- |
4.00c): Anapproachrepresentingthetraditionalbyte-level
add [ebx], 4
|     | ret |     | mutation-orientedfuzzers. |     | Beingconstantlydevelopedand |     |
| --- | --- | --- | ------------------------- | --- | --------------------------- | --- |
improved,AFL++representsstate-of-the-artgreyboxfuzzers
...
andisusedbymanyspecializedfuzzers,suchasSYMCC.
Figure2: TechnicalperspectiveofhowJITstubsareusedtoper- 3) FUZZTRUCTION. This method represents our ap-
formamutationondifferenttypesofdataoperations.First,aload proachpresentedinthispaper. WeaugmentAFL++withthe
operationisleftunmodified.Inthesecondinstrumentationsite,an ideaofmutatinggeneratorprograms,basicallycombiningthe
operandofastoreoperationismodifiedbeforeitiscommitedto
twotechniquesmentionedabove.
memory.Theexampleassemblycodeissimplifiedinthatitassumes
a32-bitprocessandomitsboundschecks.Thestoreddatavalueis 4)SYMCC. WechooseSYMCC[6](commit07c8895)
asarepresentativemethodforheavyweightprogramanalysis-
modifiedbyXORingadifferentbitmaskintotheedxregistereach
timebeforethemovinstructionisexecuted. basedapproaches, here symbolicexecution. SYMCC uses
compiler-basedinstrumentationtoleverageLLVMoptimiza-
|            |                                          |     | tionpassestomakeconstraintextractionmorefeasible. |     |     | Simi- |
| ---------- | ---------------------------------------- | --- | ------------------------------------------------- | --- | --- | ----- |
| Scheduler. | Inadditiontothecompilerpassesinstrument- |     |                                                   |     |     |       |
lartoFUZZTRUCTION,itispairedwithAFL(inourexperi-
inggeneratorandconsumer,weimplementaschedulerthat mentswithAFL++)andusedtosolveconstraintsthatAFL
orchestratesthewholefuzzingprocessandcommunication
|     |     |     | cannotsolve. | Tothebestofourknowledge,SYMCCisthe |     |     |
| --- | --- | --- | ------------ | ---------------------------------- | --- | --- |
betweengeneratorandconsumer. Furthermore,weintegrated onlystate-of-the-artconcolicfuzzerthatdoesnotrequireany
supportforusingAFL++toapplysimplebitlevelmutations sortofwarm-upto,e.g.,collectconstraints,andthereforeis
| toinputsfoundbyFUZZTRUCTION. |     | Consequently,wecan |                          |     |                          |     |
| ---------------------------- | --- | ------------------ | ------------------------ | --- | ------------------------ | --- |
|                              |     |                    | comparabletoourapproach. |     | AsSYMCCrequiresadescrip- |     |
spendmoretimeonunlockingnewprogramcompartments tionoflibraryfunctions,suchthatconstraintscanbecarried
whileleavingthetaskofdiscoveringthesetoafuzzerthatis acrossthelibrarycalls,e.g.,libc,PoeplauandFrancillon
suitedtoachievehightestcasethroughput. haveannotatedseveralfunctions. Unfortunately,others,such
|     |     |     | as open64 | or pread64 | are missing. Without | descriptions, |
| --- | --- | --- | --------- | ---------- | -------------------- | ------------- |
5 Evaluation SYMCCwillnotworkforatargetcallingthese. Wemanually
|     |     |     | add missing | annotations, | such that SYMCC | works for the |
| --- | --- | --- | ----------- | ------------ | --------------- | ------------- |
Inthissection,weevaluateourprototypeFUZZTRUCTIONto targetsinourevaluation.
gaindeeperinsightsintowhereourapproachappliesandhow
|     |     |     | 5) WEIZZ. | WEIZZ | [30] is a fuzzer | that approximates, |
| --- | --- | --- | --------- | ----- | ---------------- | ------------------ |
ourprototypeperformscomparedtostate-of-the-artfuzzers. duringrun-time,theinputstructurebasedoninput-to-state
|           |     |     | correspondence.                                    | ItsharesFUZZTRUCTION’sideaofcreating |     |     |
| --------- | --- | --- | -------------------------------------------------- | ------------------------------------ | --- | --- |
| 5.1 Setup |     |     | complexstructuredinputsandusesaREDQUEEN-like[4]ap- |                                      |     |     |
proachtoprofitfrominput-to-statecorrespondence,whichhas
| We firstdescribe | ourexperimentalsetupforthe | evaluation |     |     |     |     |
| ---------------- | -------------------------- | ---------- | --- | --- | --- | --- |
beensuccessfullyusedtoovercomeroadblockssuchascheck-
includingthehardwareenvironment,thefuzzersweareeval- sums[4]. WeuseWEIZZ(commitc9cbeef)asprovidedfor
uatingagainst,andthetargetprograms. ourevaluation: Contraryto SYMCC and FUZZTRUCTION,
WEIZZistightlycoupledtoAFLandabinary-onlyapproach
HardwareEnvironment. Weusethesamehardwarecon-
|     |     |     | using AFL’s | QEMU | mode instead of source | instrumenta- |
| --- | --- | --- | ----------- | ---- | ---------------------- | ------------ |
figurationforallexperiments: AnIntelXeonGold5320CPU
tion. Thisgivesitaslightdisadvantagecomparedtotheother
@2.20GHz(52physicalcores),256GBofRAM,andSSD
fuzzers,whichhaveaccesstothefastercompile-timeinstru-
memoryasbackingstorage.
|     |     |     | mentation. | However,westillincludeitinourevaluationasit |     |     |
| --- | --- | --- | ---------- | ------------------------------------------- | --- | --- |
Fuzzers. Weevaluatethefollowingfivefuzzingmethods. is,tothebestofourknowledge,themostpowerfulapproach
Asabaseline,weuseFUZZTRUCTION-NOAFLandAFL++, inthedomainofgrammarinferencepairedwithtechniques
thetwocomponentsusedbyourapproach. Weevaluateour toovercomeroadblocks.
| USENIX Association |     |     |     | 32nd USENIX Security Symposium    1853 |     |     |
| ------------------ | --- | --- | --- | -------------------------------------- | --- | --- |

TargetApplications. Weselecttendifferentapplications Furthermore,since SYMCC isincompatiblewiththede-
basedonseveralfactorstodeterminetheeffectivenessand fault,collision-freeinstrumentationschemausedbyAFL++,
wesetAFL_LLVM_INSTRUMENTtoCLASSICforSYMCC,such
| efficiencyofourapproach. | Wemarkapplicationsusingcryp- |     |     |     |     |     |
| ------------------------ | ---------------------------- | --- | --- | --- | --- | --- |
tographicprimitiveswithalocksymbol,(cid:181). Thesymbol((cid:181)) thattheinjectedinstrumentationremainsbackwardcompati-
indicatesthataprogramusestheseprimitivesonlyoptionally. blewithplainAFL. WeusetheSYMCCcompilerwrapper
Overall,ourgoalwastoselectdifferentgroupsofapplications toinjecttheconstraintrecordinglogic. ForWEIZZ,weuse
thatprocessthreedifferenttypesofinputformats: the same uninstrumented binary as for the coverage com-
|                      |         |           |           | putation. Finally, | sinceweneedgeneratorapplicationsfor |     |
| -------------------- | ------- | --------- | --------- | ------------------ | ----------------------------------- | --- |
| • Loosely Structured | Formats | (objdump, | readelf): |                    |                                     |     |
FUZZTRUCTIONtomutate,wecompileforeachtarget(con-
Theseformatsdonotemploycomplexconstraintsand sumer)applicationageneratorapplicationwithourcustom
| arechunk-based. | Thetargetsofthisgrouparealready |     |     |     |     |     |
| --------------- | ------------------------------- | --- | --- | --- | --- | --- |
LLVMcompilerpass.
well-testedbytraditionalfuzzerswhichemploybit-level
mutations (AFL++) orinference forchunkbasedfor- Seeds. Tocreateseedsforeachtarget,weuseexistingseed
mats(WEIZZ). setsforthegeneratorofeachrespectiveconsumerorcreate
|     |     |     |     | basiconesfromscratch. | Adescriptionofeachseedinputcan |     |
| --- | --- | --- | --- | --------------------- | ------------------------------ | --- |
7zip((cid:181)),
• Complex Formats (pngtopng, unzip, and befoundinAppendixC. Insomecases,suchasgenrsa,we
pdftotext((cid:181))): Such programs usually exhibit an in- donotprovideanyseedcorpustothegeneratorsincetheap-
putstructurethatischallengingtofuzz,e.g.,becauseof plicationdoesnotconsumeanyinput. Toensurefairness,the
transformations,suchascompressionorchecksums. As seedfilesetsfortheconsumeraregeneratedbyexecutingthe
aresult,moresophisticatedapproachesthantraditional generatoronallgeneratorseedfilesandusingtheresulting
bytelevel-orientedmutations,e.g.,symbolicexecution filesasseedfilesetsforAFL++,SYMCC,andWEIZZ. Addi-
orstructure inference pairedwithinput-to-state corre- tionally,incasethegeneratorandconsumerprocessthesame
spondence(WEIZZ),arenecessarytoachievehighcov- kindofformat, we also provide the unprocessedgenerator
erage. Weincludetargetsusingcryptographyoptionally, inputstotheotherfuzzers.
| as fuzzers     | can create inputs   | exercising | deep program |                            |                           |                         |
| -------------- | ------------------- | ---------- | ------------ | -------------------------- | ------------------------- | ----------------------- |
|                |                     |            |              | Coverage computation.      | We use                    | coverage as a metric to |
| states without | using cryptography, | e.g.,      | because only |                            |                           |                         |
|                |                     |            |              | evaluatefuzzerperformance. | Forthis,weusethedrcovsub- |                         |
somechunksoftheinputsareencrypted.
moduleoftheDYNAMORIO[38]tracingframework. This
• Cryptographic Formats (OpenSSL’s dsa(cid:181) and rsa(cid:181), allowsustoretrieveexecutiontraces(i.e.,startaddressesof
allexecutedbasicblocks)foragiveninputonanuninstru-
andMozillaNSS’vfychain(cid:181)):Theseapplicationselude
state-of-the-artfuzzers, primarilyduetousingcrypto- mentedbinary. Runningtheinputsfromallfuzzerson the
sameuninstrumentedbinariesensuresthatcoveragenumbers
graphicprimitives.
reportedinthispaperareconsistentandcomparable. Tore-
Allapplicationsusedduringourevaluationaredescribedin ducenoisegeneratedduringtracing,weexcludedthefollow-
moredetailinAppendixB.Ascommonlydoneinfuzzereval- ingstandardlibrariesfrombeingtraced:libgcc,libstdc++,
uations,weusecodecoverageasaproxyofhowwellafuzzer libc,libpthread,libm,libdl,andld-linux.
performs. Todiscusswhetherthisreflectsthefuzzers’ability
tofindbugs,werefertheinterestedreadertoBöhmeetal.[36].
Table1:Fuzzingroadblocksinourtargetsalongsidetheirgenerators.
Weusenosanitizersastheydonotexhibitnewcodecoverage
Sometargetsheavilyrelyonchecksumsorcrypto.Severaltargets,
(totriagebugs,weuseValgrindasdescribedinSection5.3).
suchas7zip,canworkwithunencryptedandencrypteddata;((cid:51))
Noteworthy,SYMCCrunsintosegmentationfaultsfor7zip
indicatesthatthesetargetsusecryptographicasanopt-in. More
andfailstobuildvfychain(cid:181)becauseitdoesnotsupportsome
detailsontheapplicationscanbefoundinAppendix5.
vectorizedinstructions(i.e.,anassertistriggeredduringcom-
pilation). Weexcludeitfromthesetargetsasaconsequence. Roadblocks
|     |     |     |     | Target |     | GeneratorforFT |
| --- | --- | --- | --- | ------ | --- | -------------- |
Checksums Crypto
| TargetPreparation. | Alltargetsundertestwereprepared |     |     |     |                   |     |
| ------------------ | ------------------------------- | --- | --- | --- | ----------------- | --- |
|                    |                                 |     |     |     | (cid:51) (cid:51) |     |
according to the respective fuzzers’ needs. Since AFL++, rsa(cid:181) genrsa(cid:181)
|     |     |     |     |     | (cid:51) (cid:51) |     |
| --- | --- | --- | --- | --- | ----------------- | --- |
FUZZTRUCTION, and SYMCC rely on the AFL++’s com- dsa(cid:181) gendsa(cid:181)
|     |     |     |     | vfychain(cid:181) | (cid:51) (cid:51) | sign(cid:181) |
| --- | --- | --- | --- | ----------------- | ----------------- | ------------- |
pilerpassinstrumentation,alltargets(forFUZZTRUCTION,
|     |     |     |     | 7zip((cid:181)) | (cid:51) ((cid:51)) | 7zip,7zip(cid:181) |
| --- | --- | --- | --- | --------------- | ------------------- | ------------------ |
onlytheconsumers)werecompiledviaafl-clang-fastin
|     |     |     |     | pdftotext((cid:181)) | (cid:51) ((cid:51)) | pdfseparate,qpdf(cid:181) |
| --- | --- | --- | --- | -------------------- | ------------------- | ------------------------- |
version4.00c. Inadditiontothedefaultsettings,wealsoset (cid:51) ((cid:51))
|     |     |     |     | unzip((cid:181)) |     | zip |
| --- | --- | --- | --- | ---------------- | --- | --- |
thefollowingflags[1,37]:
|     |     |     |     | pngtopng | (cid:51) (cid:56) | pngtopng |
| --- | --- | --- | --- | -------- | ----------------- | -------- |
|     |     |     |     | e2fsck   | (cid:51) (cid:56) | mke2fs   |
• AFL_LLVM_LAF_SPLIT_SWITCHES=1
|     |     |     |     | readelf | (cid:56) (cid:56) | objcopy |
| --- | --- | --- | --- | ------- | ----------------- | ------- |
• AFL_LLVM_LAF_SPLIT_COMPARES=1
|     |     |     |     | objdump | (cid:56) (cid:56) | objcopy |
| --- | --- | --- | --- | ------- | ----------------- | ------- |
• AFL_LLVM_LAF_TRANSFORM_COMPARES=1
| 1854    32nd USENIX Security Symposium |     |     |     |     |     | USENIX Association |
| -------------------------------------- | --- | --- | --- | --- | --- | ------------------ |

|     | rsa |     |     | dsa |     |     |     |     | vfychain |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- |
20000
| 12000 |     |     | 10000 |     |     |     |     |     |     |     |
| ----- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
10000
|      |     |     | 8000 |     |     |     | 15000 |     |     |     |
| ---- | --- | --- | ---- | --- | --- | --- | ----- | --- | --- | --- |
| 8000 |     |     | 6000 |     |     |     |       |     |     |     |
10000
6000
|      |     | Fuzztruction-NoAFL | 4000 |     | Fuzztruction-NoAFL |     |      |     |     |                    |
| ---- | --- | ------------------ | ---- | --- | ------------------ | --- | ---- | --- | --- | ------------------ |
| 4000 |     | AFL++              |      |     | AFL++              |     |      |     |     | Fuzztruction-NoAFL |
|      |     | Fuzztruction       |      |     | Fuzztruction       |     | 5000 |     |     | AFL++              |
2000
| 2000 |          | SymCC |     |               | SymCC |     |     |     |          | Fuzztruction |
| ---- | -------- | ----- | --- | ------------- | ----- | --- | --- | --- | -------- | ------------ |
|      |          | Weizz |     |               | Weizz |     |     |     |          | Weizz        |
| 0    |          |       | 0   |               |       |     | 0   |     |          |              |
| 0 4  | 8 12     | 16 20 | 0 4 | 8 12          | 16    | 20  |     | 0 4 | 8        | 12 16 20     |
|      | 7zip-enc |       |     | pdftotext-enc |       |     |     |     | pngtopng |              |
5000
50000
40000
| 40000 |     |     |     |     |     |     | 4000 |     |     |     |
| ----- | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- |
30000
| 30000 |     |     |       |     |                    |     | 3000 |     |     |                    |
| ----- | --- | --- | ----- | --- | ------------------ | --- | ---- | --- | --- | ------------------ |
|       |     |     | 20000 |     | Fuzztruction-NoAFL |     | 2000 |     |     | Fuzztruction-NoAFL |
20000
|       |     | Fuzztruction-NoAFL |       |     | AFL++        |     |      |     |     | AFL++        |
| ----- | --- | ------------------ | ----- | --- | ------------ | --- | ---- | --- | --- | ------------ |
|       |     | AFL++              | 10000 |     | Fuzztruction |     | 1000 |     |     | Fuzztruction |
| 10000 |     | Fuzztruction       |       |     | SymCC        |     |      |     |     | SymCC        |
skcolBcisaBderevoC#
|     |      | Weizz |     |           | Weizz |     |     |     |        | Weizz    |
| --- | ---- | ----- | --- | --------- | ----- | --- | --- | --- | ------ | -------- |
| 0   |      |       | 0   |           |       |     | 0   |     |        |          |
| 0 4 | 8 12 | 16 20 | 0 4 | 8 12      | 16    | 20  |     | 0 4 | 8      | 12 16 20 |
|     | 7zip |       |     | pdftotext |       |     |     |     | e2fsck |          |
50000
40000
15000
| 40000 |     |     |     |     |     |     | 12500 |     |     |     |
| ----- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- |
30000
10000
30000
|       |     |                    | 20000 |     |                    |     | 7500 |     |     |                    |
| ----- | --- | ------------------ | ----- | --- | ------------------ | --- | ---- | --- | --- | ------------------ |
| 20000 |     |                    |       |     | Fuzztruction-NoAFL |     |      |     |     | Fuzztruction-NoAFL |
|       |     | Fuzztruction-NoAFL |       |     | AFL++              |     | 5000 |     |     | AFL++              |
10000
| 10000 |         | AFL++        |     |         | Fuzztruction |     |      |     |       | Fuzztruction |
| ----- | ------- | ------------ | --- | ------- | ------------ | --- | ---- | --- | ----- | ------------ |
|       |         | Fuzztruction |     |         | SymCC        |     | 2500 |     |       | SymCC        |
|       |         | Weizz        |     |         | Weizz        |     |      |     |       | Weizz        |
| 0     |         |              | 0   |         |              |     | 0    |     |       |              |
| 0 4   | 8 12    | 16 20        | 0 4 | 8 12    | 16           | 20  |      | 0 4 | 8     | 12 16 20     |
|       | objdump |              |     | readelf |              |     |      |     | unzip |              |
15000
|       |     |     | 15000 |     |     |     | 3000 |     |     |     |
| ----- | --- | --- | ----- | --- | --- | --- | ---- | --- | --- | --- |
| 12500 |     |     | 12500 |     |     |     |      |     |     |     |
2500
10000
|      |     |     | 10000 |     |     |     | 2000 |     |     |     |
| ---- | --- | --- | ----- | --- | --- | --- | ---- | --- | --- | --- |
| 7500 |     |     | 7500  |     |     |     |      |     |     |     |
Fuzztruction-NoAFL Fuzztruction-NoAFL 1500 Fuzztruction-NoAFL
5000
|      |      | AFL++        | 5000 |      | AFL++        |     | 1000 |     |     | AFL++        |
| ---- | ---- | ------------ | ---- | ---- | ------------ | --- | ---- | --- | --- | ------------ |
|      |      | Fuzztruction |      |      | Fuzztruction |     |      |     |     | Fuzztruction |
| 2500 |      |              | 2500 |      |              |     | 500  |     |     |              |
|      |      | SymCC        |      |      | SymCC        |     |      |     |     | SymCC        |
|      |      | Weizz        |      |      | Weizz        |     |      |     |     | Weizz        |
| 0    |      |              | 0    |      |              |     | 0    |     |     |              |
| 0 4  | 8 12 | 16 20        | 0 4  | 8 12 | 16           | 20  |      | 0 4 | 8   | 12 16 20     |
Time[h]
Figure3:Thecoverage(inbasicblocks)producedbyvarioustoolsoverfive24hrunsondifferenttargets.Displayedarethemedianandthe
60%intervals. SYMCCcrasheson7zipandvfychain(cid:181);thuswehaveexcludeditfromthesetargets.
5.2 CoverageExperiments
|     |     |     |     | readelf |     | and unzip, | most | fuzzers | perform | equally, while |
| --- | --- | --- | --- | ------- | --- | ---------- | ---- | ------- | ------- | -------------- |
FUZZTRUCTIONisstillonparwiththeothercandidates.Only
objdump,
Toevaluatetheeffectivenessofourapproach, wecompare in a single case, for FUZZTRUCTION is outper-
FUZZTRUCTIONwithAFL++,SYMCC,andWEIZZusing formedbyWEIZZbyasmallmargin.
| thetenapplication(pairs)displayedinTable1. |     |     | Foreachof |     |     |     |     |     |     |     |
| ------------------------------------------ | --- | --- | --------- | --- | --- | --- | --- | --- | --- | --- |
thetentargets,werepeatedlyruneachfuzzerfivetimes,for FUZZTRUCTIONoutperformsstate-of-the-artmethods
intermsofoverallcodecoverage.
24hourson52cores(ifafuzzerhasmultiplebaselines,e.g.,
FUZZTRUCTION,wefairlysplitthecores,givingeach26).
The results of these experiments are shown in Figure 3. In the following, we analyze the results in more detail
At first, we take a look at the overall results and how our for each of the application groups described in Target Ap-
approachperforms in general. In the majority ofallcases, plications in Section 5.1. In particular, we consider two
FUZZTRUCTIONcoversthemostbasicblocks. Intwocases, dimensions: First,weinspectFUZZTRUCTION’sindividual
| USENIX Association |     |     |     |     |     |     | 32nd USENIX Security Symposium    1855 |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

baselines,AFL++andFUZZTRUCTION-NOAFL,andana- thecombinationofthetwo,performsbetterthanitsrespec-
lyze if andhow they synergize. Second, we compare how tivebaselines,wecanseethatFUZZTRUCTION-NOAFLcon-
FUZZTRUCTION performsrelativeto SYMCC andWEIZZ, tributesevenmorehigh-qualityinputsforcomplexformats
whichrepresentthestate-of-the-artofmoretraditionalfuzzing thanforlooselystructuredformats.
| approaches. |     |     |     |     |     |     | Notably,FUZZTRUCTION-NOAFLperformsworsethan |     |     |     |     |     |
| ----------- | --- | --- | --- | --- | --- | --- | ------------------------------------------- | --- | --- | --- | --- | --- |
AFL++ontargetswhentheydonotusecryptographicprim-
| LooselyStructuredFormats. |              |         |            | Thisgroup,representedby |     |           |               |                |       |                |               |            |
| ------------------------- | ------------ | ------- | ---------- | ----------------------- | --- | --------- | ------------- | -------------- | ----- | -------------- | ------------- | ---------- |
|                           |              |         |            |                         |     |           | itives (here, | password-based |       | encryption     | of input      | files). In |
| readelf                   | and objdump, |         | represents | the baseline            | of  | targets   |               |                |       |                |               |            |
|                           |              |         |            |                         |     |           | contrast,     | if the input   | files | are encrypted, | FUZZTRUCTION- |            |
| which general             |              | purpose | fuzzers    | are commonly            |     | evaluated |               |                |       |                |               |            |
NOAFLcanshowitsstrengthsandperformsaboutaswellas
against. FUZZTRUCTION-NOAFL,asanisolatedbaselineof AFL++. Thisimpliesthatourapproachiscapableofgener-
FUZZTRUCTION,faresworsethanitsotherbaselineAFL++.
|          |             |              |     |             |         |      | atinginterestingencryptedinputfiles. |     |     |     | Toverifythatnotion, |     |
| -------- | ----------- | ------------ | --- | ----------- | ------- | ---- | ------------------------------------ | --- | --- | --- | ------------------- | --- |
| This can | intuitively | be expected: |     | Traditional | fuzzers | such |                                      |     |     |     |                     |     |
weinspecttheuniquelycoveredfunctionsforallfuzzersand
| as AFL++ | use | simple bitmutations |     | whichare | effective | in  |     |     |     |     |     |     |
| -------- | --- | ------------------- | --- | -------- | --------- | --- | --- | --- | --- | --- | --- | --- |
findthatFUZZTRUCTIONistheonlyfuzzertocovervarious
exploringcommon,chunk-basedbinaryfileformatsasthey
|                                               |     |     |     |     |     |          | encryption-relatedfunctions. |     |     | Theseinputs, | inturn, | unlock |
| --------------------------------------------- | --- | --- | --- | --- | --- | -------- | ---------------------------- | --- | --- | ------------ | ------- | ------ |
| featureahighthroughputintheirinputgeneration. |     |     |     |     |     | Thisisin |                              |     |     |              |         |        |
AFL++’sbyte-levelmutationswithinFUZZTRUCTIONtofind
contrasttothefaultinjection-basedmutationintroducedin
|     |     |     |     |     |     |     | newcoverage,showcasingtheirsynergyeffects. |     |     |     | Interestingly, |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------------ | --- | --- | --- | -------------- | --- |
thiswork,wheregeneratinganinputrequiresthegenerator
FUZZTRUCTION’scoverageintervalsfor7zip(cid:181)significantly
| applicationtoberuntoproduceanewinput. |     |     |     |     | Measuringthe |     |                                  |     |     |     |                    |     |
| ------------------------------------- | --- | --- | --- | --- | ------------ | --- | -------------------------------- | --- | --- | --- | ------------------ | --- |
|                                       |     |     |     |     |              |     | differfromthemedianafter24hours, |     |     |     | indicatingithasnot |     |
actualexecutionspersecondforalltargets,FUZZTRUCTION-
|                |     |      |        |               |      |        | yetconverged. | Anecdotally,whenrunningFUZZTRUCTION |     |     |     |     |
| -------------- | --- | ---- | ------ | ------------- | ---- | ------ | ------------- | ----------------------------------- | --- | --- | --- | --- |
| NOAFL performs |     | by a | factor | of 3.2 slower | than | AFL++. |               |                                     |     |     |     |     |
for72hourson7zip(cid:181),weindeedfinditstilluncoverednew
| Beyondthroughput,    |     | there                      | is a secondreason |     | forthe | differ- |                       |     |                                   |                    |     |      |
| -------------------- | --- | -------------------------- | ----------------- | --- | ------ | ------- | --------------------- | --- | --------------------------------- | ------------------ | --- | ---- |
|                      |     |                            |                   |     |        |         | coverageafter24hours. |     | Anotherinterestingtargetise2fsck, |                    |     |      |
| enceincoveragefound: |     | FUZZTRUCTION-NOAFLhasnono- |                   |     |        |         |                       |     |                                   |                    |     |      |
|                      |     |                            |                   |     |        |         | where FUZZTRUCTION    |     | and                               | FUZZTRUCTION-NOAFL |     | per- |
tionofsplicing,whichisparticularlyusefulforchunk-based formnearlyequally,indicatingthatAFL++isunabletocon-
formats—suchasELFfilesprocessedbyobjdump.
|     |     |     |     |     |     |     | tributeanymeaningfultestcases. |     |     |     | Lookingatthetestcases |     |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------ | --- | --- | --- | --------------------- | --- |
Apart from the rather low individual performance of producedbythegenerator,mke2fs,wefindthatonereason
FUZZTRUCTION-NOAFL,wefindthatcombiningthetwo
|           |                |     |        |           |     |          | for our approaches’ |     | good | performance | is the fact | that our |
| --------- | -------------- | --- | ------ | --------- | --- | -------- | ------------------- | --- | ---- | ----------- | ----------- | -------- |
| baselines | (FUZZTRUCTION) |     | yields | synergies | for | objdump: |                     |     |      |             |             |          |
mutationsmanagedtogeneratedifferentfilesystemversions,
FUZZTRUCTION-NOAFLuniquelycoversfunctionsthathan- suchasext2,ext3orext4.
dlecomplexformatpartssuchascompressedELFsections
|     |     |     |     |     |     |     | Comparing | the | coverage | produced | by FUZZTRUCTION |     |
| --- | --- | --- | --- | --- | --- | --- | --------- | --- | -------- | -------- | --------------- | --- |
oronesinformatsthatareentirelydifferentfromtheformat
|     |     |     |     |     |     |     | to WEIZZ | and | SYMCC, | the synergies | between | FUZZ- |
| --- | --- | --- | --- | --- | --- | --- | -------- | --- | ------ | ------------- | ------- | ----- |
oftheprovidedseedfiles(e.g.,CommonObjectFileFormat
TRUCTION’scomponentsalsobecomemoreclearlyvisible.
(COFF)). Thisway,FUZZTRUCTION-NOAFLprovideshigh-
FUZZTRUCTIONoutperformsWEIZZandSYMCCbyamore
value inputs to AFL++, such that FUZZTRUCTION profits significantmarginthanforthelastsetoftargets.
fromboth.
Compared to WEIZZ and SYMCC, we observe a simi- The synergy effects of combining FUZZTRUCTION-
larlyintuitiveoverallpicture: Astheotherfuzzersarealso NOAFLandAFL++areclearlyvisiblefortargetsthat
| optimizedfortestingchunk-basedbinarytargets, |     |     |     |     |     | theother |     |     |     |     |     |     |
| -------------------------------------------- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- | --- | --- |
imposecomplextransformations,suchascompression
fuzzers perform comparativelywell, where WEIZZ outper- orchunk-wiseencryptionontotheirinput.
formsFUZZTRUCTIONforobjcopy,whereasallfuzzersper-
formsimilarlyforreadelf.
|                |     |                                  |     |     |     |     | CryptographicFormats.                                            |     |     | Thelastgroupentailsthetargets |     |           |
| -------------- | --- | -------------------------------- | --- | --- | --- | --- | ---------------------------------------------------------------- | --- | --- | ----------------------------- | --- | --------- |
|                |     |                                  |     |     |     |     | rsa(cid:181),dsa(cid:181),andvfychain(cid:181)(toprowinFigure3). |     |     |                               |     | Thesetar- |
| FUZZTRUCTION’s |     | faultinjection-basedinputgenera- |     |     |     |     |                                                                  |     |     |                               |     |           |
getsarecharacterizedbythefactthattheyimplementcomplex
| tion falls | shorton | traditionalfuzzing |     | targets, | butpro- |     |     |     |     |     |     |     |
| ---------- | ------- | ------------------ | --- | -------- | ------- | --- | --- | --- | --- | --- | --- | --- |
cryptographicprimitivessuchasasymmetriccryptographyor
ducesinputsunlockingnewcoverageinseveralcases.
operations,suchassigning,whichistypicallyappliedtocer-
tificates. Thesevaluesarecomplexsincetheyhaveaninner
Complex Formats. The next group contains pngtopng, structuredefinedbytheunderlyingmathematicalprimitives,
|     |     | 7zip((cid:181)), | pdftotext((cid:181)) |     |     |     |     |     |     |     |     |     |
| --- | --- | ---------------- | -------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
unzip, e2fsck, and in the middle whichislikelyvoidedifmutated.
rows of Figure 3. These programs feature more sophisti- Foralltargetsinthisgroup,FUZZTRUCTION-NOAFLand
catedchallenges forfuzzers, namely checksums andtrans- FUZZTRUCTION perform equallywell. One interesting in-
formationssuchas(chunk-wise)encryptionorcompression. sight is that AFL++ does not meaningfully contribute to
Regarding the interplay between the baselines of FUZZ- FUZZTRUCTION’scoverageandalsodoesnotbenefitfrom
TRUCTION, we see that in comparison to the previous set theseedsgeneratedbyourapproach. Thisisbecauseitinstan-
ofapplications,FUZZTRUCTION-NOAFLiscloserinindivid- taneouslybreakstheinnerstructureoftheseinputsbyapply-
ualperformancetoAFL++thanbefore. AsFUZZTRUCTION, ingitsbit-orientedmutations. Bymorecloselyinspectingthe
| 1856    32nd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

Table2:Confirmatorydataanalysisofourexperiment.Wecompare
1650014773
|     |     |      |     |     |     |     | FT  | the coverage | producedby | FUZZTRUCTION |     | againstthe |     | strongest |
| --- | --- | ---- | --- | --- | --- | --- | --- | ------------ | ---------- | ------------ | --- | ---------- | --- | --------- |
| 104 |     | 8964 |     |     |     |     |     |              |            |              |     |            |     |           |
6978 Weizz competitor. We reportboththe p-values producedbythe Mann-
4781
3093 3258 SymCC Whitney-UtestaswellastheeffectsizefromVargha-Delaney’sAˆ .
|     |     |     |     |     | 1935 |     |     |     |     |     |     |     |     | 12  |
| --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
103 1435 1395 ThelabelsS, M, andLrefertoasmall, medium, orlargeeffect
| skcolBcisaB# |     |     | 725 | 820 |     |     |     |     |     |     |     |     |     |     |
| ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
619 669 size,respectively. Aˆ >0.5meansapositiveeffectsize,i.e.,an
12
improvementoverthebaseline,Aˆ
|     |     |     |     | 140 | 141 | 176 |     |     |     |     | 12 <0.5anegativeeffectsize. |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------------------- | --- | --- | --- |
102
|     |     |     |     |     | 87  | 75 63 | 67  |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
58
Aˆ
|     |     | 19  |     |     |     |     | 1919 | Target |     | BestCompetitor | p-value |     | 12 effectsize |     |
| --- | --- | --- | --- | --- | --- | --- | ---- | ------ | --- | -------------- | ------- | --- | ------------- | --- |
18
| 101 |     |     |     |     |     |     | 1011 | rsa(cid:181) |     |       |       |     |          |     |
| --- | --- | --- | --- | --- | --- | --- | ---- | ------------ | --- | ----- | ----- | --- | -------- | --- |
|     |     | 6   |     |     | 7   |     |      |              |     | AFL++ | <0.05 |     | +L(1.00) |     |
5
|     |     |     |     |     |     |     |     | dsa(cid:181) |     | AFL++ | <0.05 |     | +L(1.00) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | ----- | ----- | --- | -------- | --- |
2 2
|     |     |     |     |     |     |     |     | vfychain(cid:181) |     | AFL++ | <0.05 |     | +L(1.00) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | ----- | ----- | --- | -------- | --- |
a a n nc p c xt g k p delf p 7zip(cid:181) AFL++ <0.05 +L(1.00)
|     | rs  | ds ai | p-e 7zi | xt-e n ote | p n 2fsc | m     | nzi |                    |     |       |       |     |          |     |
| --- | --- | ----- | ------- | ---------- | -------- | ----- | --- | ------------------ | --- | ----- | ----- | --- | -------- | --- |
|     |     | yc h  |         |            | o        | d u   | a u | pdftotext(cid:181) |     | AFL++ | <0.05 |     | +L(1.00) |     |
|     |     |       | 7zi     | ote dft    | gt e     | bj re |     |                    |     |       |       |     |          |     |
|     |     | vf    |         | p          | n        | o     |     | pngtopng           |     | SYMCC | <0.05 |     | +L(1.00) |     |
|     |     |       |         | dft        | p        |       |     |                    |     |       |       |     |          |     |
|     |     |       |         |            |          |       |     | 7zip               |     | AFL++ | <0.05 |     | +L(1.00) |     |
p
|        |                |     |              |     |              |     |              | pdftotext |     | AFL++ | <0.05 |     | +L(1.00) |     |
| ------ | -------------- | --- | ------------ | --- | ------------ | --- | ------------ | --------- | --- | ----- | ----- | --- | -------- | --- |
|        |                |     |              |     |              |     |              | e2fsck    |     | WEIZZ | <0.05 |     | +L(1.00) |     |
| Figure | 4: Logarithmic |     | plot showing |     | the numberof |     | basic blocks |           |     |       |       |     |          |     |
objdump
exclusivelyfoundby FUZZTRUCTION (FT), WEIZZ, orSYMCC. WEIZZ 1.0000 (0.48)
SYMCCdoesnotsupportvfychain(cid:181)and7zip((cid:181)). readelf SYMCC 0.8413 (0.44)
|     |     |     |     |     |     |     |     | unzip |     | SYMCC | 0.6905 |     | -S(0.40) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | ----- | ------ | --- | -------- | --- |
uniquefunctionscoveredbyFUZZTRUCTIONincaseofrsa(cid:181),
|     |     |     |     |     |     |     |     | Statisticalsignificance. |     |     | FollowingKleesetal.’[39]aswell |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------ | --- | --- | ------------------------------ | --- | --- | --- |
wefindthatFUZZTRUCTIONuniquelycovers298functions
asArcuri’sandBriand’s[40,41]recommendations,wever-
relatedtoimplementationsofdifferentencryptionandhash-
ifywhethertheobserveddifferencesarestatisticallysignifi-
ingalgorithms,likesha256,sha512,AES,andIDEA;iteven
cant. Todoso,weusethetwo-sidednon-parametricMann-
triggerstheuseofdifferentciphermodes. Thisisparticularly Whitney-Utest[42]. Additionally,wemeasureeffectsizes
noteworthyasthegeneratordoesnotconsumeanyseedinput
|                |     |                                           |     |     |     |     |     | to quantify                      | the | improvement. | To this | end,         | we conduct | the    |
| -------------- | --- | ----------------------------------------- | --- | --- | --- | --- | --- | -------------------------------- | --- | ------------ | ------- | ------------ | ---------- | ------ |
| forthistarget. |     | Consequently,thevarietyinoutputsresulting |     |     |     |     |     |                                  |     |              |         |              |            |        |
|                |     |                                           |     |     |     |     |     | non-parametricVarghaandDelaneyAˆ |     |              |         | test[43,44]. |            | Itmea- |
12
fromourapproachsucceedsingeneratinghigh-qualityinputs
|     |     |     |     |     |     |     |     | sures the | probability | that | running | FUZZTRUCTION |     | yields |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | ----------- | ---- | ------- | ------------ | --- | ------ |
forcomplexstructures,evenwithoutseedinputs.
highercoveragevaluesthanitsbestperformingcompetitor
Otherfuzzersstrugglewiththistypeoftargets: Sincethe (AFL++,SYMCC,orWEIZZ): Ifbothalgorithmsareequiv-
mathematicsunderpinningpublickeycryptographyarede- alent,Aˆ =0.5;if,forinstance,Aˆ
|     |     |     |     |     |     |     |     |     | 12  |     | 12  | =0.9,90%ofthetime, |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |
signedasone-wayoperations,itisimpossibleforsymbolic FUZZTRUCTION achieves better coverage results than the
executiontogeneratevalidkeypairsoutofthinair. AFL++ fuzzerwecompareditto. Basedon VarghaandDelaney’s
guidelines[43],weconsideraneffectsizeAˆ
and WEIZZ both void the cryptographic primitives due to >0.56assmall,
12
thebit-levelmutationsandWEIZZdoesnotbenefitfromits >0.64asmedium,and>0.71aslarge. Forthesakeofsim-
approachsinceitcannotinfervalidsignaturesorencryption.
plicity,weonlyreportthedifferencetothebest-performing
Asaresult,FUZZTRUCTIONsignificantlyoutperformsother competitor(chosenbymediancoverage).
fuzzersforcryptographicapplications. The results in Table 2 show that FUZZTRUCTION is sig-
nificantlybetterforallbutthreetargets,i.e.,thedifference
|     |          |     |           |     |         |               |     | in the number | of  | found basic | blocks | is statistically |     | signifi- |
| --- | -------- | --- | --------- | --- | ------- | ------------- | --- | ------------- | --- | ----------- | ------ | ---------------- | --- | -------- |
| Our | approach | is  | excellent | for | fuzzing | cryptographic |     |               |     |             |        |                  |     |          |
applicationsandrepresentstheonlywayoffuzzingsuch cant(indicatedbyp-value< 0.05)andtheeffectsizeislarge
applicationswithoutrelyingonmanualharnessing. (Aˆ > 0.71).Unsurprisingly,thisdoesnotholdforobjdump
12
andreadelf,targetswheretraditionalfuzzersalreadyper-
|     |     |     |     |     |     |     |     | formwell. | Here,WEIZZandSYMCCareslightlybetter(in |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | -------------------------------------- | --- | --- | --- | --- | --- |
ExclusivelyCoveredBasicBlocks. Tofurtherquantifythe themediancoverage),however,accordingtoMann-Whitney-
U,thereisnostatisticaldifferenceforobjdumpandreadelf.
| difference | between |     | the state-of-the-art |     | techniques |     | tackling |     |     |     |     |     |     |     |
| ---------- | ------- | --- | -------------------- | --- | ---------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
the creation of complex input formats, we analyze the ba- Additionally,inbothcases,theeffectsizedoesnotmeetthe
sicblocksfoundexclusivelybyasinglefuzzerfrom FUZZ- barsuggestedbyVarghaandDelaney[43]. Onlyforthethird
TRUCTION, WEIZZ, and SYMCC. As visible in Figure 4, target,unzip,asmallnegativeeffectsizeisvisible,however,
FUZZTRUCTIONfindssignificantlymorebasicblocks(that itisstatisticallyinsignificant(p-value> 0.05).
arenotfoundbyanyotherfuzzer)foralltargetsbutreadelf Overall, these results support our intuition that FUZZ-
andunzip. ThisindicatesthatFUZZTRUCTIONcoverscode TRUCTIONfindssignificantlymorecoverageonmosttargets.
theothertwofuzzersfailedtoexplore. Fortheothertargets,wecouldnotfindasignificantdifference
| USENIX Association |     |     |     |     |     |     |     |     |     | 32nd USENIX Security Symposium    1857 |     |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | --- |

Table3:Uniquecrashesfoundbydifferentfuzzers.Allcrasheshave supporting our argument that it is effective in uncovering
beenbucketedbyhashingthelastthreefunctionsofValgrind’s bugsinprogramsusingcryptographicprimitives. Still, for
backtrace(orofallbacktracesifValgrindreportsmultiple,e.g.,for
|     |     |     |     |     |     | severaltargets,nofuzzerfindsanycrash: |     |     | Thisisunsurpris- |     |
| --- | --- | --- | --- | --- | --- | ------------------------------------- | --- | --- | ---------------- | --- |
adjacentallocationsofthememorylocationaccessedbyaninvalid
|     |     |     |     |     |     | ing,astargetssuchasOpenSSL’srsa(cid:181) |     |     | anddsa(cid:181) havebeen |     |
| --- | --- | --- | --- | --- | --- | ---------------------------------------- | --- | --- | ------------------------ | --- |
write).Wemarktargetsforwhichafuzzerfailedtorunwithadash.
auditedoftenandexhibitacomparablysmallattacksurface.
Bestresultismarkedinbold.
|        |       |             |          |     |       | Others, such | as pngtopng | (i.e., libpng)    | have been            | well- |
| ------ | ----- | ----------- | -------- | --- | ----- | ------------ | ----------- | ----------------- | -------------------- | ----- |
|        |       |             |          |     |       | tested by    | previous    | fuzzing campaigns | [33]. Interestingly, |       |
| Target | SYMCC | WEIZZ AFL++ | FT-NOAFL | FT  | Total |              |             |                   |                      |       |
SYMCCperformsaswellasAFL++despiteitssubparcov-
| rsa(cid:181) | 0   | 0 0 | 0   | 0   | 0   |     |     |     |     |     |
| ------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
dsa(cid:181) erage. Underlining the synergy effects of combining our
|     | 0   | 0 0 | 0   | 0   | 0   |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
vfychain(cid:181) - 0 0 3 3 3 approachwithAFL++,FUZZTRUCTIONfindssignificantly
| 7zip(cid:181)      | -   | 2 5 | 4   | 86  | 90  |                                         |              |         |                         |     |
| ------------------ | --- | --- | --- | --- | --- | --------------------------------------- | ------------ | ------- | ----------------------- | --- |
|                    |     |     |     |     |     | morecrashesthanFUZZTRUCTION-NOAFLalone. |              |         | Weman-                  |     |
| pdftotext(cid:181) | 0   | 0 0 | 0   | 1   | 1   |                                         |              |         |                         |     |
|                    |     |     |     |     |     | ually triaged                           | and reported | 27 bugs | so far in a coordinated |     |
| pngtopng           | 0   | 0 0 | 0   | 0   | 0   |                                         |              |         |                         |     |
7zip
- 4 2 1 54 56 way to the developers. At the time of writing, 19 of these
| pdftotext | 0   | 0 0 | 0   | 0   | 0   |     |     |     |     |     |
| --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
reportswereacknowledgedasvalidbythedevelopers.There-
| e2fsck | 1   | 1 1 | 6   | 7   | 10  |     |     |     |     |     |
| ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
portedbugsbelongtothefollowingtargets(#Acknowledged,
| objdump | 0   | 0 0 | 0   | 8   | 8   |     |     |     |     |     |
| ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
readelf 1 1 0 0 2 2 # Reported): unzip (3/3), readelf (1/1), objdump (4/4),
| unzip | 23  | 31 25 | 4   | 23  | 38  |     |     |     |     |     |
| ----- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
pdftotext(1/1),7zip(cid:181)(0/8),e2fsck(7/7),andvfychain(cid:181)
| Sum1 | 25  | 37 31 | 17  | 151 | 261 | (3/3). |     |     |     |     |
| ---- | --- | ----- | --- | --- | --- | ------ | --- | --- | --- | --- |
1)For7zip(cid:181)and7zip,afuzzermayinadvertentlyfindthesamecrashtwice(oncefor
7zip(cid:181)andoncefor7zip).Wecountsuchoverlappingbugsonlyonceforthesum.
FUZZTRUCTIONsignificantlyoutperformscurrentstate-
of-the-artfuzzersbothw.r.t.tocoverageandthenumber
betweenFUZZTRUCTIONanditsbestcompetitor. offoundsecurity-relevantcrashes.
5.3 FoundBugs
|     |     |     |     |     |     | 6 Discussion |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- |
Duringourevaluation,wefoundmultiplecrashesinthepro-
| gramsundertest. | Sincedetectingcrashes,andthereforebugs, |     |     |     |     |                     |     |           |                      |     |
| --------------- | --------------------------------------- | --- | --- | --- | --- | ------------------- | --- | --------- | -------------------- | --- |
|                 |                                         |     |     |     |     | The novelapproachwe |     | presentin | this workis suitable | for |
istheoverarchinggoalofafuzzer,thesefindingsprovidean
successfullyleveragingdomainknowledgeingeneratorappli-
| additional | proxy forthe | effectiveness | of ourapproach. |     | To  |     |     |     |     |     |
| ---------- | ------------ | ------------- | --------------- | --- | --- | --- | --- | --- | --- | --- |
cationstogenerateinputsforcomplexformatsthatareboth
| reduce noise | in the huge | number | of produced | crashes, | we  |     |     |     |     |     |
| ------------ | ----------- | ------ | ----------- | -------- | --- | --- | --- | --- | --- | --- |
structurallycorrectandadheretoconstraintswithinthestruc-
firstranValgrindonallcrashinginputstocategorizethem
|                          |             |                   |                 |            |     | ture, suchascryptographicprimitivesorcompression. |     |     |              | We  |
| ------------------------ | ----------- | ----------------- | --------------- | ---------- | --- | ------------------------------------------------- | --- | --- | ------------ | --- |
| according                | to the type | of the underlying | memory          | violation, |     |                                                   |     |     |              |     |
|                          |             |                   |                 |            |     | provideacriticaldiscussionoftheresults,           |     |     | limitations, | and |
| e.g., segmentationfault, |             | invalidread,      | orinvalidwrite. |            | For |                                                   |     |     |              |     |
possiblefutureworkinthefollowing.
vfychain(cid:181),wealsoconsideruninitializedreads,whichare
consideredsecurityrelevantbyupstream. Alongsidethiscat- Threats to Validity. It is critical to assert the validity of
egory,Valgrindalsoprovidesastacktraceofthefunction conclusionsdrawnfromexperiments,especiallyiftheyare
causingthecrashandcontextinformation,suchasstacktraces empirical. Weidentifythreeparticularlyrelevantdimensions
ofneighboringallocations. Webucketthecrashesbasedon forourresearchandoutline ourassumptions andthe steps
| thelastthreefunctionsinthestacktrace(s). |     |     |     | Thismassively |     |     |     |     |     |     |
| ---------------------------------------- | --- | --- | --- | ------------- | --- | --- | --- | --- | --- | --- |
takentoensuretheexperimentsarevalid.
deduplicatesthenumberofcrashinginputs(aseachbucket ExternalValidity.Amajorriskiswhetheranyconclusions
containshundredsifnotthousandsofcrashinginputs),how- basedonthesetoftargetprogramstestedcanbeappliedto
ever, itstillrepresents no exactmapping to the underlying a broader, more general category of targets. Even though
bugs. Identifyingandtriagingbugsrequiressignificantman- assessinguntestedsoftwareischallenging,wehavecarefully
ualeffortandoftentheexpertiseofadomainexpertfamiliar
selectedadiversesetoftargetscoveringdifferentcategories
withtheprogram. ofprograms. Inparticular,wehavenotonlyselectedtargets
OurresultsarereportedinTable3. Ascanbeseen,FUZZ- employingcryptography,forwhichourapproachisdesigned
TRUCTION found the most crashes, with a majority of all and thus likely to succeed, but also targets that have been
crashesoccurringin7zip. Aswefuzztwoconfigurationsof testedregularlybyotherfuzzers,suchasobjdumporlibpng
7zip((cid:181)),oneusingencryptionandonewithout,thefuzzers (usedbypngtopng). Beyondourevaluation,weopen-source
caninadvertentlyfindthesamecrashforbothofthem. In- ourimplementationtoallowanyonetoevaluateourapproach.
vestigating how many bugs overlap this way, we find this InternalValidity.Beyondgenerality,itiscrucialtomini-
occursforallfuzzers(WEIZZ: 2overlappingbugs,AFL++: mizesystematicerrorsintheevaluationprocessitself. We
2, FUZZTRUCTION-NOAFL: 1, FUZZTRUCTION: 33). In- repeatourexperimentsforalltargetsfivetimestoavoidany
terestingly,FUZZTRUCTIONfinds53bugsuniqueto7zip(cid:181), sucherrors. Furthermore,weevaluatetheachievedcoverage
| 1858    32nd USENIX Security Symposium |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

forallfuzzersonthesameuninstrumentedbinarytoenable asinput: Whenfuzzed,AFL++willmakemanymutations
comparableandconsistentcoveragemeasurements. Finally, destroyingthePDF’sformat,suchthatpdftotextisunable
to avoid selection bias, we use the same set of seeds files to produce interesting outputs thatcan be fedinto the con-
forallfuzzers(orevenlargersetsforcompetitorsofourap- sumer, pdftotext. Ultimately, this shifts the problem of
proach),asoutlinedinSection5.1. generating valid input for the consumer to the problem of
ConstructValidity.Afinalthreattovalidityiswhetherthe generatingvalidinputforthegenerator. FUZZTRUCTION,on
evaluationmeasureswhatitissupposedtomeasure. Ingen- theotherhand,modifiesthegeneratoritselftoproducevalid
eral,itisdifficulttocomparetheapproachimplementedby butslightlyincorrectdata.
aparticularfuzzerwithanother,sinceitishighlydependent
|     |     |     |     |     |     |     | Seeding. | Inpractice,formingawell-roundedseedcorpus |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------- | ----------------------------------------- | --- | --- | --- |
onengineeringfactorsthatareorthogonaltothefuzzer’sap-
iscrucialforsettingupasuccessfulfuzzingcampaign. Prior
| proach itself. | To  | avoid | such discrepancies, |     | we ensure | that |     |     |     |     |     |
| -------------- | --- | ----- | ------------------- | --- | --------- | ---- | --- | --- | --- | --- | --- |
researchhasalsoshownthatselectingseedshasanimpacton
| fuzzer configurations |      | use      | the same | baseline, | i.e., we | com- |                        |     |                                 |     |     |
| --------------------- | ---- | -------- | -------- | --------- | -------- | ---- | ---------------------- | --- | ------------------------------- | --- | --- |
|                       |      |          |          |           |          |      | fuzzingresults[45,46]. |     | Forafairevaluation,weusesimple, |     |     |
| bine them             | with | the same | version  | of AFL++. | Thus,    | when |                        |     |                                 |     |     |
uninformedseedfilesduringourexperiments(seeTable6in
| observing | changes | in coverage |     | relative | to the performance |     |     |     |     |     |     |
| --------- | ------- | ----------- | --- | -------- | ------------------ | --- | --- | --- | --- | --- | --- |
AppendixC).
| of AFL++, | we  | can attribute | this | change | to the paired | ap- |     |     |     |     |     |
| --------- | --- | ------------- | ---- | ------ | ------------- | --- | --- | --- | --- | --- | --- |
proach,FUZZTRUCTIONorSYMCCrespectively. Thisdoes InteractiveGeneratorConsumerExecution. Whilethis
| not hold | for WEIZZ, | which | cannot | be configured | this | way. |     |     |     |     |     |
| -------- | ---------- | ----- | ------ | ------------- | ---- | ---- | --- | --- | --- | --- | --- |
isnotalimitationinherenttoourapproach,ourprototypeim-
| Note that | WEIZZ | and AFL++ |     | share some | concepts, | e.g., |     |     |     |     |     |
| --------- | ----- | --------- | --- | ---------- | --------- | ----- | --- | --- | --- | --- | --- |
plementationdoesnotsupportbidirectionalcommunications,
AFL++’s cmpcov feature is inspired by WEIZZ [1]. Ad- e.g.,client-serverapplications. Althoughitisgenerallypossi-
| ditionally, | WEIZZ | outperforms |     | AFL++ | onalmostalltheir |     |     |     |     |     |     |
| ----------- | ----- | ----------- | --- | ----- | ---------------- | --- | --- | --- | --- | --- | --- |
ble,supportingsuchscenariosintroducesnewchallengesthat
testedbenchmarks[30]. Hence,webelievethecomparisonto demandconsideration. Forexample,itrequiresaneworacle
beacceptablew.r.t. toconstructvalidity. todeterminewhetherafuzzingiterationisoverorifoneof
theparticipantsisstillprocessingdataandabouttosendan
| Requirement |     | for a Generator |     | Application. | Intuitively, |     |          |                 |           |       |                |
| ----------- | --- | --------------- | --- | ------------ | ------------ | --- | -------- | --------------- | --------- | ----- | -------------- |
|             |     |                 |     |              |              |     | answerto | the otherparty. | Apartfrom | that, | we believe our |
requiringageneratorapplicationforourapproachseemslike
a restriction. However, if an application parses a specific approachcouldbewellsuitedtofuzzcomplexprotocolssuch
asTLSthatcannotbefuzzedbycurrentapproaches[47,48],
dataformatorprotocol,thereistypicallyacomplementing
mainlyduetothecryptographicprimitivesused.
| programcapableofproducingsuchdata. |     |     |     |     | Havingadatafor- |     |     |     |     |     |     |
| ---------------------------------- | --- | --- | --- | --- | --------------- | --- | --- | --- | --- | --- | --- |
matwithoutprogramsproducingsuchdatawouldrenderthe
| existenceoftheformatitselffutile. |     |     |     |                      |     |     | 7 RelatedWork |     |     |     |     |
| --------------------------------- | --- | --- | --- | -------------------- | --- | --- | ------------- | --- | --- | --- | --- |
| MultipleGeneratorApplications.    |     |     |     | Havingmultiplegener- |     |     |               |     |     |     |     |
Ourapproachopensanewavenuetowardsovercomingthe
atorapplicationsthatimplementdifferentsetoffeaturesw.r.t.
tothetargetdataformatcanbebeneficialtocovermorecode problemofsolvingcomplexconstraints. Previousresearch
hasproposedseveraldifferentapproachestotackledifferent
| withinthetargetprogram. |     |     | Intuitively,thisiscomparableto |     |     |     |     |     |     |     |     |
| ----------------------- | --- | --- | ------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- |
havingamorediverseseedsetfortraditionalfuzzing,where aspectsofthesameproblem.
theseedscoverdifferentcodelocationsandserveasastarting
|                         |     |     |                                 |     |     |     | Hybrid Fuzzers. | To  | address | the shortcomings | of blind |
| ----------------------- | --- | --- | ------------------------------- | --- | --- | --- | --------------- | --- | ------- | ---------------- | -------- |
| pointforlatermutations. |     |     | Similarly,havingmultiplegenera- |     |     |     |                 |     |         |                  |          |
fuzzersandfeedback-drivenfuzzersinsolvingcomplexcon-
torsimplementingthesamefeaturesmayhelpfuzzing,since
|     |     |     |     |     |     |     | straints,severalhybridfuzzershavebeendeveloped. |     |     |     | These |
| --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------- | --- | --- | --- | ----- |
differentimplementationsmayproducedifferentoutputs.
fuzzerscommonlyemployadvancedprogramanalysistech-
Using an Unmodified Generator. Approximating FUZZ- niques to assist the fuzzer. They identify difficult-to-solve
TRUCTION’sapproach,wecoulduseAFL++toprovidemu- constraintsandcomputeaninputthatpassesthisconstraint.
tated input to the generator and feed the outputs produced Intheory,thisunlocksthefuzzerbyshowingithowtobypass
by the generator into the consumer. Other than for FUZZ- thisparticularroadblock. Frequentlyusedprogramanalysis
techniquesaretainttracking[17–19]andconcolic/symbolic
| TRUCTION,thegeneratoritselfishereleftuntouched. |     |     |     |     |     | This |     |     |     |     |     |
| ----------------------------------------------- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- |
hasanumberofdrawbacks: First,anumberofapplications execution[5,6,14,16]. Whilethesetechniquesworkforcon-
doesnotuseanyinput(genrsa(cid:181),gendsa(cid:181),mke2fs,orsign(cid:181)). straintsimposedbychecksumsandsimilarconstructs,they
Second,otherapplicationssuchas7zip((cid:181))orziponlywrap
failformorecomplexconstraintsasimposedbycryptographic
theinput,hereinancompressedcontainer. FUZZTRUCTION’s primitives,asalsobecomesvisibleinourevaluation.
approachcanmodifythewrapperitself,i.e.,produceslightly Moreover,symbolicexecutionfailstoscaletolargepro-
corruptzip-files,whichAFL++mutatedinputdataisunable grams due to the path explosion problem and requires de-
toachieve. Third,forapplicationsnotaffectedbythesedraw- scriptionsoftheexecutionenvironment. Ourapproach,how-
backs, e.g., qpdf(cid:181) orpngtopng, using AFL++ toproduce ever,fulfillsthesameroleasthesetechniques,butwithout
inputstillsuffersfromAFL++’sinabilitytoovercomemore theirshortcomings: Insteadoftryingtoextractthedomain
complexconstraints. Forexample,qpdf(cid:181)expectsaPDFfile knowledgeneededtosolveaparticularconstraintfromthe
| USENIX Association |     |     |     |     |     |     |     | 32nd USENIX Security Symposium    1859 |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

fuzzingtarget(usingheavy-weightanalysistechniques),we 8 Conclusion
| useasecond | program |     | togeneratesuchdata. |     |     | Whilenotdi- |     |     |     |     |     |     |     |     |
| ---------- | ------- | --- | ------------------- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
Inthispaper,wepresentFUZZTRUCTION,anovelapproach
rectedtowardssolvingoneparticularconstraint,ourapproach
generatesinputsthat(almost)fulfillthespecificationthese to software fault injection-based fuzzing. Based on the in-
programsusefordataexchange,therebyimplicitlypassing sightthatprogramsthatconsumeaninputhaveone(ormore)
theseconstraints. counterpartprograms generating this input, we propose to
|     |     |     |     |     |     |     |     | instrumentandmutatethisgenerator. |     |     |     | Byinjectingsubliminal |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --------------------------------- | --- | --- | --- | --------------------- | --- | --- |
Grammar-based Fuzzing. Like our approach, grammar- softwarefaults,wecanharnesstheimplicitdomainknowl-
basedfuzzersuseaspecificationtogeneratevalidinputsfor
edgeencodedintheapplicationandgenerateinputsthatal-
| programs, | thus | exercising | deeper | state | space | [21–29]. | As  |                            |     |     |                             |     |     |     |
| --------- | ---- | ---------- | ------ | ----- | ----- | -------- | --- | -------------------------- | --- | --- | --------------------------- | --- | --- | --- |
|           |      |            |        |       |       |          |     | mostmatchthespecification. |     |     | Usingtheseinputsforfuzzing, |     |     |     |
opposedtoourapproach,typicallythesegrammarsmustbe
FUZZTRUCTIONproduceshigh-qualityinputsthatproduce
| manuallygenerated. |     | Asubsetoffuzzersattemptstoapprox- |     |     |     |     |     |                                               |     |     |     |     |     |     |
| ------------------ | --- | --------------------------------- | --- | --- | --- | --- | --- | --------------------------------------------- | --- | --- | --- | --- | --- | --- |
|                    |     |                                   |     |     |     |     |     | highcodecoverageandexercisedeepprogramstates. |     |     |     |     |     | Our |
imategrammarswithoutpriordomainknowledgeaboutthe approachislightweightanddoesnotrequirecostlyanalyses
| target[30,34,35]. |          | Whilesuchapproximationscanidentify |         |           |        |              |     |                                          |     |     |     |     |              |     |
| ----------------- | -------- | ---------------------------------- | ------- | --------- | ------ | ------------ | --- | ---------------------------------------- | --- | --- | --- | --- | ------------ | --- |
|                   |          |                                    |         |           |        |              |     | ormanuallypreparedexecutionenvironments. |     |     |     |     | Intheevalua- |     |
| logicalunits      | (chunks, |                                    | tokens, | orfields) | within | the targeted |     |                                          |     |     |     |     |              |     |
tion,wefindthatourapproachshowsitsstrengthbygenerally
dataformat,theycannotsolvecomplexconstraintsimposed outperforming state-of-the-art fuzzing methods, especially
| ontheselogicalunits. |     | Thiscanalsobeseeninourevaluation, |     |     |     |     |     |     |     |     |     |     |     |     |
| -------------------- | --- | --------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ontargetswithcomplexconstraints,usuallyintheformof
whereFUZZTRUCTIONoutperformsWEIZZfortargetsthat cryptographicprimitivesorcompressionappliedtotheinput.
usecryptography.
DomainExpertise. Givenahumanexpert,theycanbypass 9 Acknowledgements
| many of | the challenges |     | addressed | by  | our work. | A   | human |     |     |     |     |     |     |     |
| ------- | -------------- | --- | --------- | --- | --------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
expertcanmanuallyharnessthetarget,removechecksums,
|     |     |     |     |     |     |     |     | We wouldlike |     | to thankouranonymous |     | reviewers | fortheir |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | --- | -------------------- | --- | --------- | -------- | --- |
provideagrammartothefuzzer,orexplicitlyannotatethetar-
|     |     |     |     |     |     |     |     | valuablecommentsandsuggestions. |     |     |     | WealsothankMarcel |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------- | --- | --- | --- | ----------------- | --- | --- |
gettoguidethefuzzer[20].However,havingadomainexpert
|     |     |     |     |     |     |     |     | Böhme, | Merlin | Chlosta, | Thorsten | Eisenhofer, | Joel | Frank, |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------ | -------- | -------- | ----------- | ---- | ------ |
iscostlyandnotfeasibleinallapplications,suchaslegacy KenoHassler,DanielKlischies,LeaSchönherr,andSimon
| software. | Our approach |     | approximates |     | the human | expert’s |     |        |           |           |           |     |        |        |
| --------- | ------------ | --- | ------------ | --- | --------- | -------- | --- | ------ | --------- | --------- | --------- | --- | ------ | ------ |
|           |              |     |              |     |           |          |     | Wörner | for their | feedback. | This work | was | funded | by the |
knowledgebyharnessingthedomainknowledgeencodedby
DeutscheForschungsgemeinschaft(DFG,GermanResearch
theprogrammerinthegeneratorapplication.
|     |     |     |     |     |     |     |     | Foundation) | under | Germany’s | Excellence | Strategy |     | – EXC |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | ----- | --------- | ---------- | -------- | --- | ----- |
2092CASA–390781972andbytheGermanFederalMin-
| Differential                                              | Fuzzing. |      | Other    | approaches | [27,49–56] |              | sim- |              |           |              |        |         |       |     |
| --------------------------------------------------------- | -------- | ---- | -------- | ---------- | ---------- | ------------ | ---- | ------------ | --------- | ------------ | ------ | ------- | ----- | --- |
|                                                           |          |      |          |            |            |              |      | istry of     | Education | and Research | (BMBF, | project | CPSec | –   |
| ilarly exploit                                            | the      | fact | that two | programs   | share      | a specifica- |      |              |           |              |        |         |       |     |
| tion: Differentialfuzzersor,moregeneral,differentialtest- |          |      |          |            |            |              |      | 16KIS1564K). |           |              |        |         |       |     |
ing. However,theirunderlyingideaistocomparetwocon-
sumersagainsteachother,providingafine-granularoracle
References
| that detects | miscomputations |     |     | beyond | memory | safety | bugs. |     |     |     |     |     |     |     |
| ------------ | --------------- | --- | --- | ------ | ------ | ------ | ----- | --- | --- | --- | --- | --- | --- | --- |
Ourapproachfocusesonthetwoendpoints,ageneratorand
[1] AndreaFioraldi,DominikMaier,HeikoEißfeldt,and
| a consumer, | using | a shared | data | format, | making | these | ap- |            |     |        |                             |     |     |     |
| ----------- | ----- | -------- | ---- | ------- | ------ | ----- | --- | ---------- | --- | ------ | --------------------------- | --- | --- | --- |
|             |       |          |      |         |        |       |     | MarcHeuse. |     | AFL++: | CombiningIncrementalStepsof |     |     |     |
proachesorthogonal. Ourapproachcouldbeusedtogenerate FuzzingResearch. InUSENIXWorkshoponOffensive
seedsthatarethentestedinadifferentialfuzzingsetupoftwo
|     |     |     |     |     |     |     |     | Technologies(WOOT),2020. |     |     | (Citedon1,2,7,8,13) |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------ | --- | --- | ------------------- | --- | --- | --- |
consumers.
[2] RenDing,YonghaeKim,FanSang,WenXu,Gururaj
| MutationTesting.                           |     | Similartoourapproach,mutationtest- |     |     |     |           |     |                                        |     |            |      |          |         |     |
| ------------------------------------------ | --- | ---------------------------------- | --- | --- | --- | --------- | --- | -------------------------------------- | --- | ---------- | ---- | -------- | ------- | --- |
|                                            |     |                                    |     |     |     |           |     | Saileshwar,                            |     | and Taesoo | Kim. | Hardware | Support | to  |
| ing[57–59]insertsfaultsintoatargetprogram. |     |                                    |     |     |     | Thegoalis |     |                                        |     |            |      |          |         |     |
|                                            |     |                                    |     |     |     |           |     | ImproveFuzzingPerformanceandPrecision. |     |            |      |          | InACM   |     |
usuallytosimulatecommonprogrammingbugstoassessthe
ConferenceonComputerandCommunicationsSecurity
| qualityofa | testsuite |     | orgenerate | one. | Ourworkis |     | similar |             |     |            |     |     |     |     |
| ---------- | --------- | --- | ---------- | ---- | --------- | --- | ------- | ----------- | --- | ---------- | --- | --- | --- | --- |
|            |           |     |            |      |           |     |         | (CCS),2021. |     | (Citedon1) |     |     |     |     |
inthatweinjectfaultsintoaprogram;however,wedonot
attempttogeneratemutationsthatthetestsuitesdonotcover,
[3] WenXu,SanidhyaKashyap,ChangwooMin,andTae-
but instead inject arbitrary faults. Instead of assessing the sooKim. DesigningNewOperatingPrimitivestoIm-
| quality  | of an existing |         | set of test | cases,    | we use   | the | outputs |       |         |              |     |     |            |     |
| -------- | -------------- | ------- | ----------- | --------- | -------- | --- | ------- | ----- | ------- | ------------ | --- | --- | ---------- | --- |
|          |                |         |             |           |          |     |         | prove | Fuzzing | Performance. | In  | ACM | Conference | on  |
| produced | by the         | mutant, | i.e.,       | the buggy | program, |     | to fuzz |       |         |              |     |     |            |     |
ComputerandCommunicationsSecurity(CCS),2017.
| anotherapplication. |     | Wethenusecoveragefeedbackwithin |     |     |     |     |     | (Citedon1) |     |     |     |     |     |     |
| ------------------- | --- | ------------------------------- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- | --- | --- |
thefuzzingtarget,amongstotherinformation,todetermine
thequalityofamutation. Insummary,mutationtestingand [4] CorneliusAschermann,SergejSchumilo,TimBlazytko,
ourapproachsharetheideaofinjectingfaults;however,their RobertGawlik,andThorstenHolz. RedQueen:Fuzzing
goalsandthusdesignsarefundamentallydifferent. withInput-to-StateCorrespondence. InSymposiumon
| 1860    32nd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | --- |

NetworkandDistributedSystemSecurity(NDSS),2019. SymbolicExecution. InSymposiumonNetworkand
(Citedon1,3,7) DistributedSystemSecurity(NDSS),2016. (Citedon1,
3,5,13)
[5] InsuYun,SanghoLee,MengXu,YeongjinJang,and
Taesoo Kim. QSYM: A Practical Concolic Execu- [15] CristianCadar,DanielDunbar,DawsonREngler,etal.
tionEngineTailoredforHybridFuzzing. InUSENIX KLEE: UnassistedandAutomaticGenerationofHigh-
| SecuritySymposium,2018.                    |     | (Citedon1,3,5,13)       |          |     |                       |       |                      |              |             |            |
| ------------------------------------------ | --- | ----------------------- | -------- | --- | --------------------- | ----- | -------------------- | ------------ | ----------- | ---------- |
|                                            |     |                         |          |     | Coverage              |       | Tests for Complex    | Systems      | Programs.   | In         |
|                                            |     |                         |          |     | Symposium             |       | on Operating         | Systems      | Design      | and Imple- |
| [6] SebastianPoeplauandAurélienFrancillon. |     |                         | Symbolic |     |                       |       |                      |              |             |            |
|                                            |     |                         |          |     | mentation(OSDI),2008. |       |                      | (Citedon1,3) |             |            |
| ExecutionwithSymCC:                        |     | Don’tInterpret,Compile! |          | In  |                       |       |                      |              |             |            |
| USENIXSecuritySymposium,2020.              |     | (Citedon1,2,3,          |          |     |                       |       |                      |              |             |            |
|                                            |     |                         |          |     | [16] Hui              | Peng, | Yan Shoshitaishvili, |              | and Mathias | Payer.     |
5,7,13)
|     |     |     |     |     | T-Fuzz: |     | Fuzzing by Program | Transformation. |     | In  |
| --- | --- | --- | --- | --- | ------- | --- | ------------------ | --------------- | --- | --- |
[7] SergejSchumilo,CorneliusAschermann,RobertGaw- IEEESymposiumonSecurityandPrivacy(S&P),2018.
(Citedon1,13)
| lik, Sebastian | Schinzel, | and Thorsten | Holz. | kAFL: |     |     |     |     |     |     |
| -------------- | --------- | ------------ | ----- | ----- | --- | --- | --- | --- | --- | --- |
Hardware-AssistedFeedbackFuzzingforOSKernels.
InUSENIXSecuritySymposium,2017. (Citedon1) [17] Vijay Ganesh, Tim Leek, and Martin Rinard. Taint-
|     |     |     |     |     | basedDirectedWhiteboxFuzzing. |     |     | InInternationalCon- |     |     |
| --- | --- | --- | --- | --- | ----------------------------- | --- | --- | ------------------- | --- | --- |
[8] TobiasScharnowski,NilsBars,MoritzSchloegel,Eric ferenceonSoftwareEngineering(ICSE),2009. (Cited
| Gustafson,MariusMuench,GiovanniVigna,Christo- |     |     |           |     | on1,3,13)   |       |          |        |         |          |
| --------------------------------------------- | --- | --- | --------- | --- | ----------- | ----- | -------- | ------ | ------- | -------- |
| pherKruegel,ThorstenHolz,andAliAbbasi.        |     |     | Fuzzware: |     |             |       |          |        |         |          |
|                                               |     |     |           |     | [18] Tielei | Wang, | Tao Wei, | Guofei | Gu, and | Wei Zou. |
UsingPreciseMMIOModelingforEffectiveFirmware
|                                          |     |     |     |        | TaintScope: |           | AChecksum-awareDirectedFuzzingTool |     |            |     |
| ---------------------------------------- | --- | --- | --- | ------ | ----------- | --------- | ---------------------------------- | --- | ---------- | --- |
| Fuzzing. InUSENIXSecuritySymposium,2022. |     |     |     | (Cited |             |           |                                    |     |            |     |
| on1)                                     |     |     |     |        | for         | Automatic | Software Vulnerability             |     | Detection. | In  |
IEEESymposiumonSecurityandPrivacy(S&P),2010.
| [9] BoFeng,AlejandroMera,andLongLu. |     |     | P2IM: | Scal- |     |     |     |     |     |     |
| ----------------------------------- | --- | --- | ----- | ----- | --- | --- | --- | --- | --- | --- |
(Citedon1,3,13)
ableandHardware-independentFirmwareTestingvia
AutomaticPeripheralInterfaceModeling. InUSENIX [19] PengChenandHaoChen. Angora: EfficientFuzzing
SecuritySymposium,2020. (Citedon1) byPrincipledSearch. InIEEESymposiumonSecurity
|     |     |     |     |     | andPrivacy(S&P),2018. |     |     | (Citedon1,3,13) |     |     |
| --- | --- | --- | --- | --- | --------------------- | --- | --- | --------------- | --- | --- |
[10] EricGustafson,MariusMuench,ChadSpensky,Nilo
Redini, Aravind Machiry, Yanick Fratantonio, Da- [20] CorneliusAschermann, SergejSchumilo, AliAbbasi,
| videBalzarotti, | AurélienFrancillon,YungRynChoe, |     |     |     |                  |     |                               |     |     |     |
| --------------- | ------------------------------- | --- | --- | --- | ---------------- | --- | ----------------------------- | --- | --- | --- |
|                 |                                 |     |     |     | andThorstenHolz. |     | IJON:ExploringDeepStateSpaces |     |     |     |
ChristopherKruegel,etal. TowardtheAnalysisofEm- via Fuzzing. In IEEE Symposium on Security and
beddedFirmwarethroughAutomatedRe-hosting. In Privacy(S&P),2020. (Citedon1,14)
SymposiumonRecentAdvancesinIntrusionDetection
(RAID),2019. (Citedon1) [21] Christian Holler, Kim Herzig, and Andreas Zeller.
|                                                 |     |                          |     |      | Fuzzing                                        | with | Code Fragments. | In  | USENIX | Security |
| ----------------------------------------------- | --- | ------------------------ | --- | ---- | ---------------------------------------------- | ---- | --------------- | --- | ------ | -------- |
| [11] KarlKoscher,TadayoshiKohno,andDavidMolnar. |     |                          |     | Sur- |                                                |      |                 |     |        |          |
|                                                 |     |                          |     |      | Symposium,2012.                                |      | (Citedon1,2,14) |     |        |          |
| rogates: EnablingNear-Real-TimeDynamicAnalyses  |     |                          |     |      |                                                |      |                 |     |        |          |
| ofEmbeddedSystems.                              |     | InUSENIXWorkshoponOffen- |     |      |                                                |      |                 |     |        |          |
|                                                 |     |                          |     |      | [22] Van-ThuanPham,MarcelBöhme,AndrewESantosa, |      |                 |     |        |          |
siveTechnologies(WOOT),2015. (Citedon1) Alexandru Ra˘zvan Ca˘ciulescu, andAbhikRoychoud-
|                      |                      |                |              |     | hury.                                     | SmartGreyboxFuzzing. |     | IEEETransactionson |     |        |
| -------------------- | -------------------- | -------------- | ------------ | --- | ----------------------------------------- | -------------------- | --- | ------------------ | --- | ------ |
| [12] SergejSchumilo, | CorneliusAschermann, |                | AliAbbasi,   |     |                                           |                      |     |                    |     |        |
|                      |                      |                |              |     | SoftwareEngineering,47(9):1980–1997,2019. |                      |     |                    |     | (Cited |
| Simon Wörner,        | and                  | Thorsten Holz. | Nyx: Greybox |     |                                           |                      |     |                    |     |        |
on1,2,14)
| Hypervisor                             | Fuzzing | using Fast Snapshots | and | Affine |                                                    |     |     |     |     |     |
| -------------------------------------- | ------- | -------------------- | --- | ------ | -------------------------------------------------- | --- | --- | --- | --- | --- |
| Types. InUSENIXSecuritySymposium,2021. |         |                      |     | (Cited |                                                    |     |     |     |     |     |
|                                        |         |                      |     |        | [23] CorneliusAschermann,TommasoFrassetto,Thorsten |     |     |     |     |     |
on1)
|     |     |     |     |     | Holz, | Patrick | Jauernig, | Ahmad-Reza | Sadeghi, | and |
| --- | --- | --- | --- | --- | ----- | ------- | --------- | ---------- | -------- | --- |
[13] Sang Kil Cha, Maverick Woo, and David Brumley. Daniel Teuchert. Nautilus: Fishing for Deep Bugs
|                                     |     |     |              |     | withGrammars.                      |     | In Symposium | on  | NetworkandDis- |     |
| ----------------------------------- | --- | --- | ------------ | --- | ---------------------------------- | --- | ------------ | --- | -------------- | --- |
| Program-adaptiveMutationalFuzzing.  |     |     | InIEEESympo- |     |                                    |     |              |     |                |     |
|                                     |     |     |              |     | tributedSystemSecurity(NDSS),2019. |     |              |     | (Citedon1,2,   |     |
| siumonSecurityandPrivacy(S&P),2015. |     |     | (Citedon1,   |     |                                    |     |              |     |                |     |
| 3)                                  |     |     |              |     | 14)                                |     |              |     |                |     |
[14] Nick Stephens, John Grosen, Christopher Salls, An- [24] RohanPadhye,CarolineLemieux,KoushikSen,Mike
drew Dutcher, Ruoyu Wang, Jacopo Corbetta, Yan Papadakis,andYvesLeTraon. SemanticFuzzingwith
Shoshitaishvili,ChristopherKruegel,andGiovanniVi- Zest. InInternationalSymposiumonSoftwareTesting
gna. Driller: AugmentingFuzzingThroughSelective andAnalysis(ISSTA),2019. (Citedon1,2,14)
| USENIX Association |     |     |     |     |     |     | 32nd USENIX Security Symposium    1861 |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

[25] HyungSeokHan, DongHyeon Oh, andSang KilCha. [36] MarcelBöhme, László Szekeres, andJonathan Metz-
CodeAlchemist: Semantics-AwareCodeGenerationto man. On the Reliability of Coverage-Based Fuzzer
Find Vulnerabilities in JavaScript Engines. In Sym- Benchmarking. In InternationalConference on Soft-
posium on Network and Distributed System Security wareEngineering(ICSE),2022. (Citedon8)
| (NDSS),2019. |       | (Citedon1,2,14) |          |      |        |           |                              |                                          |     |     |                   |     |     |
| ------------ | ----- | --------------- | -------- | ---- | ------ | --------- | ---------------------------- | ---------------------------------------- | --- | --- | ----------------- | --- | --- |
|              |       |                 |          |      |        |           | [37] lafintel.               | laf-intel-CircumventingFuzzingRoadblocks |     |     |                   |     |     |
|              |       |                 |          |      |        |           | withCompilerTransformations. |                                          |     |     | https://lafintel. |     |     |
| [26] Soyeon  | Park, | Wen             | Xu, Insu | Yun, | Daehee | Jang, and |                              |                                          |     |     |                   |     |     |
TaesooKim. FuzzingJavaScriptEngineswithAspect- wordpress.com. (Citedon8)
| preservingMutation.   |     |     | InIEEESymposiumonSecurity |     |     |     |                 |           |     |            |            |     |          |
| --------------------- | --- | --- | ------------------------- | --- | --- | --- | --------------- | --------- | --- | ---------- | ---------- | --- | -------- |
|                       |     |     |                           |     |     |     | [38] The        | DynamoRIO |     | Team.      | DynamoRIO. |     | https:// |
| andPrivacy(S&P),2020. |     |     | (Citedon1,2,14)           |     |     |     |                 |           |     |            |            |     |          |
|                       |     |     |                           |     |     |     | dynamorio.org/. |           |     | (Citedon8) |            |     |          |
[27] JihyeokPark,SeungminAn,DongjunYoun,Gyeong-
[39] GeorgeKlees,AndrewRuef,BenjiCooper,ShiyiWei,
| won          | Kim, | and Sukyoung | Ryu.    | JEST:      | N+1-Version |     |                  |     |     |                        |     |     |       |
| ------------ | ---- | ------------ | ------- | ---------- | ----------- | --- | ---------------- | --- | --- | ---------------------- | --- | --- | ----- |
|              |      |              |         |            |             |     | andMichaelHicks. |     |     | EvaluatingFuzzTesting. |     |     | InACM |
| Differential |      | Testing      | of Both | JavaScript | Engines     | and |                  |     |     |                        |     |     |       |
ConferenceonComputerandCommunicationsSecurity
InInternationalConferenceonSoftware
| Specification.          |         |       |                 |     |             |      | (CCS),2018.                       |     | (Citedon11) |     |     |                 |     |
| ----------------------- | ------- | ----- | --------------- | --- | ----------- | ---- | --------------------------------- | --- | ----------- | --- | --- | --------------- | --- |
| Engineering(ICSE),2021. |         |       | (Citedon1,2,14) |     |             |      |                                   |     |             |     |     |                 |     |
|                         |         |       |                 |     |             |      | [40] AndreaArcuriandLionelBriand. |     |             |     |     | APracticalGuide |     |
| [28] Vasudev            | Vikram, | Rohan | Padhye,         |     | and Koushik | Sen. |                                   |     |             |     |     |                 |     |
forUsingStatisticalTeststoAssessRandomizedAlgo-
GrowingATestCorpuswithBonsaiFuzzing. InInter- rithmsinSoftwareEngineering. InInternationalCon-
nationalConferenceonSoftwareEngineering(ICSE), ferenceonSoftwareEngineering(ICSE),2011. (Cited
| 2021. | (Citedon1,2,14) |     |     |     |     |     |     |     |     |     |     |     |     |
| ----- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
on11)
[29] Y.Chen,R.Zhong,H.Hu,H.Zhang,Y.Yang,D.Wu, [41] Andrea Arcuri and Lionel Briand. A Hitchhiker’s
and W. Lee. One Engine to Fuzz ’em All: Generic Guide to Statistical Tests for Assessing Randomized
LanguageProcessorTestingwithSemanticValidation. Algorithms in Software Engineering. Software Test-
In IEEE Symposium on Security and Privacy (S&P), ing,VerificationandReliability,24(3):219–250,2014.
| 2021.       | (Citedon1,2,14) |         |      |         |     |            | (Citedon11)                       |     |     |     |     |     |           |
| ----------- | --------------- | ------- | ---- | ------- | --- | ---------- | --------------------------------- | --- | --- | --- | --- | --- | --------- |
|             |                 |         |      |         |     |            | [42] HenryBMannandDonaldRWhitney. |     |     |     |     |     | OnaTestof |
| [30] Andrea | Fioraldi,       | Daniele | Cono | D’Elia, |     | and Emilio |                                   |     |     |     |     |     |           |
Coppa. WEIZZ: Automatic Grey-box Fuzzing for whether One of Two Random Variables is Stochasti-
|                          |     |     |                          |     |     |     | callyLargerthantheOther. |     |     |     | AnnalsofMathematical |     |     |
| ------------------------ | --- | --- | ------------------------ | --- | --- | --- | ------------------------ | --- | --- | --- | -------------------- | --- | --- |
| StructuredBinaryFormats. |     |     | InInternationalSymposium |     |     |     |                          |     |     |     |                      |     |     |
onSoftwareTestingandAnalysis(ISSTA),2020. (Cited Statistics,18(1):50–60,1947. (Citedon11)
on2,3,7,13,14)
|     |     |     |     |     |     |     | [43] AndrásVarghaandHaroldDDelaney. |     |     |     |     | ACritiqueand |     |
| --- | --- | --- | --- | --- | --- | --- | ----------------------------------- | --- | --- | --- | --- | ------------ | --- |
ImprovementoftheCLCommonLanguageEffectSize
| [31] Michał               | Zalewski. |     | American | Fuzzy         | Lop. | http:// |                                             |     |     |     |                      |     |        |
| ------------------------- | --------- | --- | -------- | ------------- | ---- | ------- | ------------------------------------------- | --- | --- | --- | -------------------- | --- | ------ |
|                           |           |     |          |               |      |         | StatisticsofMcGrawandWong.                  |     |     |     | JournalofEducational |     |        |
| lcamtuf.coredump.cx/afl/. |           |     |          | (Citedon2,18) |      |         |                                             |     |     |     |                      |     |        |
|                           |           |     |          |               |      |         | andBehavioralStatistics,25(2):101–132,2000. |     |     |     |                      |     | (Cited |
on11)
[32] MarcelBöhme,Van-ThuanPham,andAbhikRoychoud-
| hury.               | Coverage-based |              | Greybox    | Fuzzing  |              | as Markov |                                   |     |         |           |           |             |           |
| ------------------- | -------------- | ------------ | ---------- | -------- | ------------ | --------- | --------------------------------- | --- | ------- | --------- | --------- | ----------- | --------- |
|                     |                |              |            |          |              |           | [44] Robert                       | J   | Grissom | and John  | J Kim.    | Effect      | Sizes for |
| Chain.              | IEEE           | Transactions | on         | Software | Engineering, |           |                                   |     |         |           |           |             |           |
|                     |                |              |            |          |              |           | Research:                         |     | A Broad | Practical | Approach. |             | Lawrence  |
| 45(5):489–506,2017. |                |              | (Citedon2) |          |              |           |                                   |     |         |           |           |             |           |
|                     |                |              |            |          |              |           | ErlbaumAssociatesPublishers,2005. |     |         |           |           | (Citedon11) |           |
| [33] OSS-Fuzz       |                | - Continuous | Fuzzing    |          | for Open     | Source    |                                   |     |         |           |           |             |           |
[45] AlexandreRebert,SangKilCha,ThanassisAvgerinos,
https://github.com/google/oss-fuzz.
| Software. |     |     |     |     |     |     | Jonathan |     | Foote, | David Warren, | Gustavo |     | Grieco, and |
| --------- | --- | --- | --- | --- | --- | --- | -------- | --- | ------ | ------------- | ------- | --- | ----------- |
(Citedon2,12)
|          |           |           |     |             |     |        | DavidBrumley.                   |     |     | OptimizingSeedSelectionforFuzzing. |     |             |     |
| -------- | --------- | --------- | --- | ----------- | --- | ------ | ------------------------------- | --- | --- | ---------------------------------- | --- | ----------- | --- |
|          |           |           |     |             |     |        | InUSENIXSecuritySymposium,2014. |     |     |                                    |     | (Citedon13) |     |
| [34] Tim | Blazytko, | Cornelius |     | Aschermann, |     | Moritz |                                 |     |     |                                    |     |             |     |
Schloegel,AliAbbasi,SergejSchumilo,SimonWörner, [46] DongdongShe,AbhishekShah,andSumanJana. Effec-
andThorstenHolz. Grimoire: SynthesizingStructure tiveSeedSchedulingforFuzzingwithGraphCentrality
| whileFuzzing. |     | InUSENIXSecuritySymposium,2019. |     |     |     |     |             |     |                                     |     |     |     |     |
| ------------- | --- | ------------------------------- | --- | --- | --- | --- | ----------- | --- | ----------------------------------- | --- | --- | --- | --- |
|               |     |                                 |     |     |     |     | Analysis.   |     | InIEEESymposiumonSecurityandPrivacy |     |     |     |     |
| (Citedon3,14) |     |                                 |     |     |     |     | (S&P),2022. |     | (Citedon13)                         |     |     |     |     |
[35] WeiYou,XuweiLiu,ShiqingMa,DavidMitchelPerry, [47] Van-ThuanPham,MarcelBöhme,andAbhikRoychoud-
XiangyuZhang,andBinLiang. SLF: Fuzzingwithout hury. AFLNet: AGreyboxFuzzerforNetworkProto-
Valid Seed Inputs. In International Conference on cols. InInternationalConferenceonSoftwareTesting,
SoftwareEngineering(ICSE),2019. (Citedon3,14) ValidationandVerification(ICST),2020. (Citedon13)
| 1862    32nd USENIX Security Symposium |     |     |     |     |     |     |     |     |     |     |     | USENIX Association |     |
| -------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------------ | --- |

[48] SergejSchumilo,CorneliusAschermann,AndreaJem- [58] GoranPetrovic´andMarkoIvankovic´. StateofMutation
mett, AliAbbasi, andThorstenHolz. Nyx-Net: Net- TestingatGoogle. InInternationalConferenceonSoft-
work Fuzzing with Incremental Snapshots. In Euro- wareEngineering: SoftwareEngineeringinPractice,
peanConferenceonComputerSystems(EuroSys),2022. 2018. (Citedon14)
(Citedon13)
|     |     |     |     |     |     |     | [59] Mike                  | Papadakis, | Marinos | Kintis, | Jie Zhang,      | Yue Jia, |
| --- | --- | --- | --- | --- | --- | --- | -------------------------- | ---------- | ------- | ------- | --------------- | -------- |
|     |     |     |     |     |     |     | YvesLeTraon,andMarkHarman. |            |         |         | MutationTesting |          |
[49] XuejunYang,YangChen,EricEide,andJohnRegehr.
FindingandUnderstandingBugsinCCompilers. In Advances: an Analysis and Survey. In Advances in
ACMSIGPLANConferenceonProgrammingLanguage Computers,volume112,pages275–378. Elsevier,2019.
| DesignandImplementation(PLDI),2011. |     |     |     |     |     | (Citedon | (Citedon14) |     |     |     |     |     |
| ----------------------------------- | --- | --- | --- | --- | --- | -------- | ----------- | --- | --- | --- | --- | --- |
14)
|                              |           |       |     |       |                      |              | [60] Samples | of  | binary with                        | different | formats     | and ar-  |
| ---------------------------- | --------- | ----- | --- | ----- | -------------------- | ------------ | ------------ | --- | ---------------------------------- | --------- | ----------- | -------- |
|                              |           |       |     |       |                      |              | chitectures. |     | A test suite                       | for       | your binary | analysis |
| [50] Chad                    | Brubaker, | Suman |     | Jana, | Baishakhi            | Ray, Sarfraz |              |     |                                    |           |             |          |
|                              |           |       |     |       |                      |              | tools.       |     | https://github.com/JonathanSalwan/ |           |             |          |
| Khurshid,andVitalyShmatikov. |           |       |     |       | UsingFrankencertsfor |              |              |     |                                    |           |             |          |
AutomatedAdversarialTestingofCertificateValidation binary-samples. (Citedon18)
| inSSL/TLSImplementations.     |     |     |     |     | InIEEESymposiumon |     |                |     |     |     |     |     |
| ----------------------------- | --- | --- | --- | --- | ----------------- | --- | -------------- | --- | --- | --- | --- | --- |
| SecurityandPrivacy(S&P),2014. |     |     |     |     | (Citedon14)       |     | A AssignedCVEs |     |     |     |     |     |
[51] Vu Le, Mehrdad Afshari, and Zhendong Su. Com- In Table 4, we list the CVEs assigned to bugs found by
pilerValidationviaEquivalenceModuloInputs. ACM FUZZTRUCTION-NOAFL.
| SIGPLANNotices,49(6):216–226,2014. |     |     |     |     |     | (Citedon14) |                                               |     |     |     |     |     |
| ---------------------------------- | --- | --- | --- | --- | --- | ----------- | --------------------------------------------- | --- | --- | --- | --- | --- |
|                                    |     |     |     |     |     |             | Table4: TableofCVEsfoundbyFUZZTRUCTION-NOAFL. |     |     |     |     | We  |
[52] ChangwooMin,SanidhyaKashyap,ByoungyoungLee, usedFUZZTRUCTION-NOAFLratherthanFUZZTRUCTIONtoen-
ChengyuSong,andTaesooKim. Cross-checkingSe- surethefoundvulnerabilitiescannotbeattributedtoAFL++’smu-
manticCorrectness: TheCaseofFindingFileSystem tationsbutarearesultofournovelapproach.
| Bugs.        | InSymposiumonOperatingSystemsPrinciples |             |     |     |     |     |               |     |        |                       |     |     |
| ------------ | --------------------------------------- | ----------- | --- | --- | --- | --- | ------------- | --- | ------ | --------------------- | --- | --- |
| (SOSP),2015. |                                         | (Citedon14) |     |     |     |     | CVEidentifier |     | Target | BugDescription        |     |     |
|              |                                         |             |     |     |     |     | CVE-2021-4217 |     | unzip  | Out-of-boundsreadinfn |     |     |
[53] YutingChen,TingSu,ChengnianSun,ZhendongSu, unzip Out-of-boundsreadinfn
CVE-2022-0530
| andJianjunZhao.          |     |     | Coverage-directedDifferentialTest- |     |                  |     |               |     |        |                        |     |     |
| ------------------------ | --- | --- | ---------------------------------- | --- | ---------------- | --- | ------------- | --- | ------ | ---------------------- | --- | --- |
|                          |     |     |                                    |     |                  |     | CVE-2022-0529 |     | unzip  | Out-of-boundswriteinfn |     |     |
| ingofJVMImplementations. |     |     |                                    |     | InACMSIGPLANCon- |     |               |     |        |                        |     |     |
|                          |     |     |                                    |     |                  |     | CVE-2022-1304 |     | e2fsck | Out-of-boundswriteinfn |     |     |
ferenceonProgrammingLanguageDesignandImple-
| mentation(PLDI),2016. |     |     |     | (Citedon14) |     |     |     |     |     |     |     |     |
| --------------------- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
[54] TheofilosPetsios,AdrianTang,SalvatoreStolfo,Ange- B TargetDescription
| losD.                                  | Keromytis,andSumanJana. |     |     |     |     | Nezha: Efficient |     |     |     |     |     |     |
| -------------------------------------- | ----------------------- | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- |
| Domain-independentDifferentialTesting. |                         |     |     |     |     | InIEEESym-       |     |     |     |     |     |     |
Table5describestheprogramswehavetestedforourevalua-
| posiumonSecurityandPrivacy(S&P),2017. |     |     |     |     |     | (Cited | tion. |     |     |     |     |     |
| ------------------------------------- | --- | --- | --- | --- | --- | ------ | ----- | --- | --- | --- | --- | --- |
on14)
[55] Jaewon Hur, Suhwan Song, Dongup Kwon, Eunjin C SeedDescription
| Baek, | Jangwoo | Kim, | and | Byoungyoung |     | Lee. Difuz- |     |     |     |     |     |     |
| ----- | ------- | ---- | --- | ----------- | --- | ----------- | --- | --- | --- | --- | --- | --- |
Table6showsthedifferentseedssetsweusedasinputforthe
| zRTL: | DifferentialFuzzTestingtoFindCPUBugs. |     |     |     |     | In  |     |     |     |     |     |     |
| ----- | ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
generatorsandconsumersduringourevaluation.
IEEESymposiumonSecurityandPrivacy(S&P),2021.
(Citedon14)
| [56] Igor                          | Lima,              | Jefferson | Silva,      | Breno    | Miranda, | Gustavo          |     |     |     |     |     |     |
| ---------------------------------- | ------------------ | --------- | ----------- | -------- | -------- | ---------------- | --- | --- | --- | --- | --- | --- |
| Pinto,                             | and                | Marcelo   | d’Amorim.   |          |          | Exposing Bugs    |     |     |     |     |     |     |
| in JavaScript                      |                    | Engines   |             | through  | Test     | Transplantation  |     |     |     |     |     |     |
|                                    |                    |           |             | Software |          | Quality Journal, |     |     |     |     |     |     |
| and                                | Differential       | Testing.  |             |          |          |                  |     |     |     |     |     |     |
| 29(1):129–158,2021.                |                    |           | (Citedon14) |          |          |                  |     |     |     |     |     |     |
| [57] Yue                           | Jia andMarkHarman. |           |             | An       | Analysis | andSurvey        |     |     |     |     |     |     |
| oftheDevelopmentofMutationTesting. |                    |           |             |          |          | IEEETrans-       |     |     |     |     |     |     |
actionsonSoftwareEngineering,37(5):649–678,2010.
(Citedon14)
| USENIX Association |     |     |     |     |     |     |     |     | 32nd USENIX Security Symposium    1863 |     |     |     |
| ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- |

Table5:Thistableprovidesanoverviewofthedifferentapplicationsusedduringtheevaluation.WetestedthelatestversionsoftheUbuntu
20.04packagesavailableatthetimeofourevaluation.ForOpenSSL,vfychain(cid:181),andpngtopng,wetestedthelatestversionsfromupstream.
| Name | Version |     | Description |     |     |     |
| ---- | ------- | --- | ----------- | --- | --- | --- |
gendsa(cid:181) openssl1.1.1l AnOpenSSLsubcommandusedtogenerate(possiblyencrypted)DSAkeys
dsa(cid:181) openssl1.1.1l SubcommandofOpenSSLforparsing(possiblyencrypted)DSAkeys
genrsa(cid:181) openssl1.1.1l ASubcommandofOpenSSLtogenerate(possiblyencrypted)RSAkeys
rsa(cid:181) openssl1.1.1l Parsingof(possiblyencrypted)RSAkeysviathersasubcommandofOpenSSL
sign(cid:181) openssl1.1.1l ThereqsubcommandofOpenSSLforproducingaself-signedkeyscertificate
vfychain(cid:181) vfychain3.79 A utility that is part of the Network Security Services (NSS) suite of Mozilla
Firefoxandisusedtovalidatecertificate(chains).
7zip((cid:181)) p7zip16.02 Application used to compress or decompress data. Optionally encrypted and
protectedviapassword.
qpdf(cid:181)((cid:181)) qpdf9.1.1 ToolforencryptingandmanipulatingPDFfiles
pdftotext((cid:181))
poppler-utils0.86.1 Utilitythatispartofthepopplersoftwaresuiteandisusedtoconvert(potentially
encrypted)PDFstotext
zip((cid:181))
|     | zip3.0 |     | Decompressingutilitywithoptionalsupportforencryption |     |     |     |
| --- | ------ | --- | ---------------------------------------------------- | --- | --- | --- |
unzip((cid:181)) unzip6.0 Decompressingutilitywithoptionalsupportforencryption
pngtopng libpng1.6.37 Utilityoflibpngthatsimplyparsesapngintomemoryandwritesitbacktohard
diskafterwards
| e2fsck | e2fsprogs1.45.5 |     |     |     |     |     |
| ------ | --------------- | --- | --- | --- | --- | --- |
Aapplicationforcheckingharddiskimagesforerrorsandinconsistency
objcopy binutils2.34 Utilityfortransformingdifferenttypesoffiles,likeELFs.Usedfor,e.g.,stripping
symbolsorremovingsections.
| readelf | binutils2.34 |     | ToolfordumpinginformationofELFfiles |     |     |     |
| ------- | ------------ | --- | ----------------------------------- | --- | --- | --- |
objdump binutils2.34 ToolfordisassemblingELFfilesanddumpingadditionalinformation
Table6:Tableofthedifferentseedsetsusedduringfuzzing.ThecolumnsGeneratorandConsumerlisttheapplicationsthatusedthedescribed
seedsetintheroleofaconsumerorgenerator,respectively.Incaseofthegenerators,objcopywasusedtogenerateinputsfortwodifferent
consumers(readelf,objdump)whichcausesthenumberofgenerators(11)tobeunequaltothenumberofconsumers(12).
|                                           | Seedsetusedfor |          |     | Description                            | SetName   |     |
| ----------------------------------------- | -------------- | -------- | --- | -------------------------------------- | --------- | --- |
|                                           | Generator      | Consumer |     |                                        |           |     |
| {rsa(cid:181),dsa(cid:181),sign(cid:181)} |                |          |     |                                        | empty     |     |
|                                           |                | {}       |     | Anemptyseedset                         |           |     |
|                                           | {mke2fs}       | {}       |     | Asinglefileofsize256KiBcontainingzeros | empty_256 |     |
{zip,7zip,7zip(cid:181)} {} Asingletextfilewiththecontentaaaaa text
{pdfseparate,qpdf(cid:181)} {pdftotext} SixsimplePDFdocumentscontainingforms,annotations, pdf
orbasicgeometricshapes
{} {pdftotext(cid:181)} Thesamefilesasinthepdfset,butpasswordprotected -
{} {vfychain(cid:181)} AselfsignedcertificategeneratedusingOpenSSL -
{} {rsa(cid:181)} ApasswordprotectedRSAkeypairgeneratedbyopenssl -
{} {dsa(cid:181)} ApasswordprotectedDSAkeypairgeneratedbyopenssl -
|     | {}  | {7zip} |     | Thetextseedsetcompressedusing7zip |     | -   |
| --- | --- | ------ | --- | --------------------------------- | --- | --- |
{} {7zip(cid:181)} Thetextseedsetcompressedandencryptedusing7zip -
|            | {}  | {unzip}    |     | Thetextseedsetcompressedusingzip               |     | -   |
| ---------- | --- | ---------- | --- | ---------------------------------------------- | --- | --- |
| {pngtopng} |     | {pngtopng} |     | ThepngseedfromAFL[31]                          |     | -   |
|            |     | {e2fsck}   |     | Thefileoftheempty_256setconvertedtoanext4image |     | -   |
{}
viamke2fs
{objcopy} {objdump,readelf} Filesfrom[60]ofsizesmallerthan1MiB(arequirement -
enforcedbyAFL++)whichobjcopyisabletoprocess
| 1864    32nd USENIX Security Symposium |     |     |     | USENIX Association |     |     |
| -------------------------------------- | --- | --- | --- | ------------------ | --- | --- |
