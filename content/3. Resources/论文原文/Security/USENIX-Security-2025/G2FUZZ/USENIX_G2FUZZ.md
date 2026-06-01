---
publish: true
---

Low-Cost and Comprehensive Non-textual Input Fuzzing with LLM-Synthesized
Input Generators∗
(cid:66) (cid:66)
KunpengZhang1 ZongjieLi1 DaoyuanWu1 ShuaiWang1 XinXia2
1TheHongKongUniversityofScienceandTechnology
2ZhejiangUniversity
Abstract 1 Introduction
Modernsoftwareoftenacceptsinputswithhighlycomplex
grammars.Toconductgreyboxfuzzinganduncoversecurity Modernsoftwareoftenacceptsinputswithhighlycomplex
bugsinsuchsoftware,itisessentialtogenerateinputsthat grammars,suchasimages,configurationfiles,andnetwork
conform to the software input grammar. However, this is packets. Fuzzing such software is well-known to be chal-
a well-known challenging task because it requires a deep lenging [14,19,23,27,33,47] because it requires a deep
understandingofthegrammar,whichisoftennotavailable understandingofsoftwaregrammartofullyexploretheinput
andhardtoinfer.Recentadvancesinlargelanguagemodels space. Often,one needs to prepare sample inputs thatcon-
(LLMs)haveshownthattheycanbeusedtosynthesizehigh- formtothegrammaroftheinputformatandalsoexhibita
qualitynaturallanguagetextandcodethatconformstothe varietyofcharacteristicsbeforeconductingeffectivegreybox
grammar of a given input format. Nevertheless,LLMs are fuzzing[6,15,28,35,36,48,56]anduncoversecuritybugs.
oftenincapableortoocostlytogeneratenon-textualoutputs, Recentstructure-awarefuzzershaveexploredsolutionsto
suchasimages,videos,andPDFfiles.Thislimitationhinders alleviatetheabovechallengeswithinference-basedfuzzing
theapplicationofLLMsingrammar-awarefuzzing. andgrammar-aware fuzzing,yettheystillhave limitations.
Wepresentanovelapproachtoenablinggrammar-aware Inference-based fuzzers, such as ProFuzzer [55], FuzzIn-
fuzzingovernon-textualinputs.WeemployLLMstosynthe- Mem[32],andWEIZZ[15],caninferinputgrammarsand
sizeandalsomutateinputgenerators,intheformofPython generateinputson-the-fly.However,theyoftensufferfrom
scripts,thatgenerate data conforming to the grammarofa lowaccuracyandweakscalability,byinferringsimpleinput
giveninputformat.Then,non-textualdatayieldedbytheinput fieldsandstrugglingtomutatefilestructures.ProFuzzerand
generatorsarefurthermutatedbytraditionalfuzzers(AFL++) WEIZZ are time-consuming forlong inputs,while FuzzIn-
toexplorethesoftwareinputspaceeffectively.Ourapproach, Memrequiresprogramshavingprinterfunctionsthatconvert
namelyG2FUZZ,featuresahybridstrategythatcombinesa in-memorydatastructurestofiles.Grammar-awaregreybox
“holisticsearch”drivenbyLLMsanda“localsearch”driven fuzzers[5,8,21,39,51]oftenrequirepre-knowledgeofthe
by industrial quality fuzzers. Two key advantages are: (1) inputgrammar(e.g.,providedbytheusers).Suchinforma-
LLMsaregoodatsynthesizingandmutatinginputgenera- tionisoftennotavailableorincomplete,makingitobscureto
torsandenablingjumpingoutoflocaloptima,thusachiev- comprehensivelyunderstandinputfieldsandtheirrelations.
ingasynergisticeffectwhencombinedwithmutation-based Moreover,currentfuzzingapproachesprimarilyvalidateba-
fuzzers;(2)LLMsarelessfrequentlyinvokedunlessreally sicstructuralfieldslikesizeandchecksumsinfileformats
needed,thussignificantlyreducingthecostofLLMusage. butneglectcomplexfeatures.Thesefeaturescantriggermore
WehaveevaluatedG2FUZZonavarietyofinputformats,in- complexlogic,revealingdeeperbugs.Complexfeaturesof-
cludingTIFFimages,MP4audios,andPDFfiles.Theresults tenincludeintricatechunksorconstraintsbetweenchunks,
showthatG2FUZZoutperformsSOTAtoolssuchasAFL++, posingchallengestotraditionalfuzzingmethods.Fuzztruc-
Fuzztruction,andFormatFuzzerintermsofcodecoverage tion[7]mitigatesthischallengefromadifferentperspective
andbug finding across mostprograms testedon three plat- byinjectingfaultsintogeneratorapplicationstoproducein-
forms:UNIFUZZ,FuzzBench,andMAGMA.G2FUZZalso puts with highly complex formats. However, Fuzztruction
discovers10uniquebugsinthelatestreal-worldsoftware,of reliesontheavailabilityofasuitablegeneratorapplication,
which3areconfirmedbyCVE. whichstillrequiresexperiencedresearcherstoidentify.
∗ThispaperhasbeenacceptedbyUSENIXSecurity2025. Largelanguagemodels(LLMs)aretransformer-basedneu-
(cid:66)Correspondingauthors. ralnetworksthathaveachievedstate-of-the-art(SOTA)per-
5202
naJ
13
]ES.sc[
1v28291.1052:viXra

formanceinnaturallanguageandcodeprocessingtasks.Thus, baselines, in terms of code coverage and bug finding. We
onemayexpectthatLLMscouldgenerateinputsampleswith evaluateditonthreethird-partybenchmarks:UNIFUZZ[30],
variousvalidgrammars,thusdrivinggrammar-awarefuzzing FuzzBench[38],andMAGMA[22].Ourresultsshowthat
on its own. Indeed, we have seen some recent works that G2FUZZachievesthebestperformanceincodecoverageand
leverageLLMstogenerateinputsforfuzzing[9,10,37,53]. bugfindingacrossallthreeplatforms.Wefindthatwiththe
Nevertheless,weclarifythatalthoughLLMsarecapableof helpofLLMs,G2FUZZisabletodiscovermanyuniqueedge/-
generatingtextualoutputs,suchasnaturallanguagetextand functioncoveragethatotherfuzzerscannotfind.Moreover,
code,we find that LLMs are often incapable or too costly weshowthatG2FUZZincursaverylowcostofLLMusage;
ofgeneratingnon-textualdatasamplesasrequiredbymany fuzzingatargetsoftwarewithGPT-3.5for24hoursonlycosts
software.WepresentadetailedanalysisinSec.3. lessthan0.2$inLLMusage.WehaveusedG2FUZZtofind
Instead of instructing LLMs to directly generate non- 10 unique bugs in the latest real-world software,of which
textualfuzzinginputs,thispaperexploresanotherperspec- 3 are confirmed by CVE. In sum,our contributions are as
tivetoaugmentmutation-basedfuzzingwithLLMs.Thekey follows:
idea is to leverage LLMs to automatically synthesize and • We introduce a novel approach to augmenting mutation-
furthermutateinputgenerators(oftenintheformofPython basedfuzzingusingLLMs.Thecoreideaistocombinethe
scripts) customized to the specific features and structures strengthsofLLMsinsynthesizingandmutatingdiversein-
ofthetargetfileformat.Byexecutingthesegenerators,we putgeneratorsandthestrengthsofmutation-basedfuzzers
canproduceinputsthatexhibitawiderangeoffeaturesand inperformingfine-grainedmutationsovernon-textualdata.
structures,potentiallytriggeringdifferentprogramlogicand Thisapproachleveragesasynergisticeffecttodelivereffec-
exploringpreviouslyuntestedcoderegions.Moreover,these tivefuzzingatamoderatecost.
generatednon-textualinputscanberapidlymutatedbytradi- • We design G2FUZZ that concretizes the above idea.
tionalmutation-basedfuzzers,suchasAFL++,toeffectively G2FUZZ properly and periodically invokes LLMs and
exploretheinputspace.Holistically,thisapproachoffersa mutation-based fuzzers to benefit from their respective
newanduniquehybridviewtoaugmentfuzzingwithLLMs: strengths.G2FUZZfeaturesasetofdesignprinciplesand
LLMsareparticularlygoodatsynthesizingdistinctinputgen- optimizationstomakeithighlyefficientandpractical.
eratorsandenablingtheescapefrom“localoptima,”whereas • Our results show that G2FUZZ consistently outperforms
mutation-basedfuzzersexcelatconductingfine-grained,lo- SOTAmutation-basedfuzzersandseveralotherfuzzerbase-
calsearchesintheinputspaceefficiently.Weshowthatour lines in terms of code coverage and bug finding across
novelcombinationofLLMsandmutation-basedfuzzerscan variousinputformatsandtestingplatforms.G2FUZZhas
achieveasynergisticeffect,leadingtoasignificantimprove- discovered10uniquebugsinthelatestreal-worldsoftware.
mentincodecoverageandbugfinding.Moreover,sincewe
onlyinvokeLLMswhennecessarytosynthesizenewinput
2 Preliminaries
generators,wesubstantiallyreducethecostofLLMusage.
Weimplementtheaboveapproachinanovelfuzzingframe-
LargeLanguageModels(LLMs).LLMs,transformer-based
work,namelyG2FUZZ.2Whenusersspecifyaninputformat
neuralnetworks,havereachedSOTAperformanceinvarious
name(e.g.,“TIFF”),G2FUZZemploysdefactoLLMs,such
NLPtasks,includingtranslationandsummarization.Autore-
asGPT-3.5andllama-3-70b-instruct,toautomaticallysyn-
gressive (e.g., GPT) and masked language modeling (e.g.,
thesizeinputgeneratorsinPythonscriptsthatgenerateTIFF
BERT) are essential for textual output, while models like
images.G2FUZZfacilitatesseveralstrategiestofurthermu-
CLIP[29]andDALL-E[44]handlenon-textualdata,enhanc-
tatethesynthesizedgenerators.Then,G2FUZZexecutesgen-
ingtheirrangeofapplications.Thecommunityhasnotedthat
erators to produce a diverse set of non-textual inputs,and
LLMshavethepotentialtoaugmentsoftwarefuzzing[12].
alsoemploysAFL++tomutatethesynthesizedinputs.When
Greybox Fuzzing. Greybox fuzzing, a technique for find-
theemployedfuzzersfailtouncovernewcodecoverageto
ingsoftwaresecuritybugs,reliesonlightweightinstrumen-
acertainextent,G2FUZZ invokesLLMstosynthesizenew
tation for execution feedback to mutate inputs more effec-
and distinct input generators,and then further mutates the
tively.FuzzerslikeAFL[57],AFL++[16],andHonggfuzz[2]
generatednon-textualinputsusingAFL++.Thisprocesscon-
have advanced this field. AFL++,with optimizations such
tinuesuntilthetargetsoftwareisfullycoveredoracertain
asRedqueen,isrecognizedasthedefactostandardfuzzer,
timebudgetisreached.
widelyusedbythesecuritycommunitytodetectbugs.
WeevaluateG2FUZZon34inputformats,includingJPEG
Grammar-AwareFuzzing.Grammar-awarefuzzing,aform
images,TIFFimages,andMP4videos.Ourresultsshowthat
ofgreyboxfuzzing,producesinputsbasedonprecisegram-
G2FUZZcanconsistentlyoutperformSOTAmutation-based
marrules,effectivelyidentifyingvulnerabilitiesinsoftware
fuzzers,suchasAFL++,andseveralstructure-awarefuzzer
that handle complex input structures. Tools like Format-
2G2FUZZstandsfor“grammar-awarefuzzingwithLLM-synthesized Fuzzer[13],Gramatron[49],andSuperion[51]leveragepro-
inputgenerators”. videdgrammarstouncoversecuritybugsinreal-worldsoft-

ware.Togenerateinputsinhighlycomplexformats,Fuzztruc- specificationsorhumanefforttomodifycode,asseeninFor-
tion[7]deliberatelyinjectsfaultsintogeneratorapplications. matFuzzer[13].FormatFuzzerobtainsformattemplatesfrom
Inference-BasedFuzzing.Inference-basedfuzzing,suchas the010Editorrepositoryandusesthemforparsing.However,
ProFuzzer[55],GreyOne[17],andWEIZZ[15],leverages manual coding is still required for the generation process.
inferredrelationshipsbetweeninputbytesandpathconstraints Furthermore,modifyingcomplexformatslikeMP4cantake
togeneratetargetedtestinputs.Thismethodanalyzesinternal “over a week” (per [13]), due to its multiple chunk types,
logicanddataformatstocreaterelevanttestcases,enhancing many ofwhichare notfully detailedin the originalbinary
coverageandreducingnoiseinresults. template.Additionally,Fuzztructioncangeneratediversefiles
byinjectingfaultsintogeneratorapplications.Yet,itrelieson
experiencedresearcherstomanuallyidentifyandinstrument
3 Motivation
suitablegeneratorapplications,andfindingappropriategener-
atorsforlesscommonformatsisoftenchallenging.Here,we
3.1 RelatedWorkandLimitations
searchGitHubforgeneratorapplicationsforall34formats
listedinTable5.Thesearchusesthekeywords"FORMAT
Existingmethodscanbecategorizedintotwotypesbasedon
converter/transformer/generatorlanguage:C++stars:>5",
theinputtheyhandle:(1)Text-formatfuzzing,and(2)Binary-
where“"FORMAT”servesaplaceholderforspecificformats.
formatfuzzing.Text-formatfuzzingprimarilytestsprograms
Forexample,forJPGfiles,oneofthesearchqueriesis"JPG
using text inputs,such as Superion [51],Nautilus [5],and
converterlanguage:C++stars:>5".Generatorswerefound
Grimoire[8].Thesemethodsgenerateavarietyofvalidtext
for21formats(usabilityuntested),whilenogeneratorswere
inputs based on provided specifications,including formats
availablefortheremaining13formats.
likeXML,Ruby,SQL,andSMT.Ontheotherhand,binary-
formatfuzzingtestsprogramswithbinaryinputs,suchasFor- ChallengeIII:SimultaneouslyProcessMultipleFormats.
matFuzzer[13],FuzzInMem[32],WEIZZ[15],andAFLS- Manysoftwarecanprocessmultipleinputformats.However,
mart[41].Thesemethodssplittheinputintomultiplechunks existing grammar-aware fuzzers typically generate files of
andperformmutationsonthesechunkstocreatediversein- a single formatduring the fuzzing process. This limitation
puts,suchasJPEG,PDF,TIFF,MP3,andMP4. hampers their effectiveness in thoroughly testing software
G2FUZZbelongstothelattercategory,constructingbinary that accept diverse input formats,potentially missing bugs
formatfileswithcomplexfeaturesforexploringdeeperinto relatedtothehandlingofspecificfiletypes.
thecode.Despitethesignificantadvancementsmadebyex- Onesolutionmaybelaunchingmultiplefuzzersinparallel,
istingmethods,theystillfacethefollowingthreechallenges: eachfocusingononeinputformat,andthenaggregatingthe
ChallengeI:GeneratingFileswithComplexFeatures.The individualfuzzingresultsattheend.However,programsoften
currentapproachesfocusprimarilyonthebasicstructureof include routine code for preprocessing and error handling
targetfileformats,suchasgeneratingvalidbasicstructural thatareindependentofspecificfileformats.Parallelismcan
fieldslikesizefields,checksums,andbitfields.However,a resultinrepetitiveeffortsinthesecommonroutines,notonly
targetbinaryfileformatoftenincorporatesvariouscomplex consumingtimebutalsowastingresources.
features,andperourobservation(seeSec.5.1.1),fileswith Insight. We view that the aforementioned limitations can
complex features often have the potential to trigger more be addressed by cleverly leveraging LLMs. Our insights
complex program logic, thereby likely uncovering deeper- are as follows: for generating files with complex features
seatedbugs.Comparedtothebasicstructures,thesecomplex (ChallengeI),numerouslibrariesforfilegenerationareal-
features differ mainly in two aspects: 1) complex features readyavailableonline.TheselibrariesofferAPIstodirectly
likelyintroduceextracomplexchunksinthebinaryfile,and constructcomplexfeaturesofthetargetformat.SinceLLMs
2)varyingconstraints(e.g.,numericalconstraintsraisedby havebeentrainedonvastdatasetsthatpresumablyinclude
checksum)maybeintroducedamongbinaryfilechunks.Ad- theseonlinecodebases,theyshallbeabletoyieldbinaryfile
ditionally,certaindependenciesexistamongdifferent(basic/- generationscripts(codeinPython)tailoredtotherequired
complex)features,whereonefeaturedependsonanotherto filefeatures.Byrunningthisgenerator,wecanproducefiles
beimplemented.Forinstance,toenableJPEGcompressionin thatexhibitthedesiredfeatures.Forexample,toimplement
aTIFFfile,thefilemustfirstsupportthe“YCbCr/RGBcolor LZWcompressionforTIFF(Fig.10b),wecanemployLLMs
space”feature.Allthesescenariosposemajorchallengesfor toconstructthecorrespondinggeneratorwith3linesofcode,
existingbinary-formatfuzzers.Forexample,currentfuzzers asinFig.11.WithLLMs,commoncomplexfilestructures
failtogenerateTIFFfileswithLZWdataduetoinaccuratein- canbegeneratedwithamoderateamountofcode.
ference(e.g.,WEIZZ)orincompletegrammarslackingLZW UsingLLMstogenerategeneratorsevidentlyeliminates
syntax(e.g.,FormatFuzzer,AFLSmart),limitingtheirparsing humaneffortsortheneedforpreparingformatspecifications.
andmutationcapabilities.SeeAppendixAformoredetails. Thisenablesafullyautomatedtestingprocess,whereasexist-
Challenge II: Require Format Specifications and Man- ingmethodsrequiremanualcodingandformatpreparation
ualCoding.Previousworksoftenrelyonprovidedformat (Challenge II). Moreover, our fuzzing pipeline maintains

generatorsofdifferentbinaryformatsunifiedly(Sec.4.1),al- LLM-EnabledOpportunity.Weobservethatmostbinary
lowingsimultaneouslyprocessingmultipleformatsyetlargely files can be produced using Python libraries. Forexample,
reducingrepetitiveefforts(ChallengeIII).Thisallowsusto wecanuse PIL togenerateJPEGfileswithdifferentstruc-
concentrateresourcesandeffortsonmorein-depthtestingof tures and CV2 to generate PNG files. Since documents for
codeareasthatarecloselyrelatedtodifferentfileformats. theselibrariesarelikelyincludedintheLLMtrainingdata,
it is reasonable to expect that LLMs can synthesize gener-
ators for binary files based on these libraries. In this step,
3.2 ThePilotStudyofLLMs weconductedexperimentsontheavailableformatsinUNI-
FUZZ,FuzzBench,andMAGMAtotestwhetherLLMscan
LLMs have been extensively trained using large-scale generateageneratorforeachformat.Formoredetails,refer
datasets,enabling them to learn complexpatterns andgen- toSec.5.1.5.Inshort,allthesenon-textualdatacanbegener-
erate high-quality outputs. This extensive pre-training en- atedbyPythonscripts.Consequently,wecanuseLLMsto
ables LLMs to excel in various open-ended, structured synthesizevariousgenerators,producingdifferentstructured
data generation tasks,such as code completion and gener- inputsandexploringdeepercoderegions.However,wehave
ation[3,11,11,20,31,40,42,50,54,58],texttoimagetrans- toaddressthefollowingtwochallenges.
lation [4,18,24,34,43], and QA tasks for customer sup-
TechnicalChallengeI:Diversity.WefindthatLLMoutputs
port[26,45,46].Itisbelievedthatthevastamountoftraining
areoftenlessdiverseanduneasytocontrol;thisisundesirable
datahelpsLLMscapturethenuancesoflanguageandproduce
infuzzing,whichexpectsalargenumberofgeneratorsthat
accurateandcontextuallyappropriateresults.
arediverseandcoverasmuchoftheinputspaceaspossible.
LimitationofDeFactoLLMs.WearepositivethatLLMs Overall,ourtentativeexplorationshowsthatLLMoutputsare
canbeusedtoaugmentmutation-basedfuzzing,giventhat oftenpredictable,meaningthatthesoftwareunderfuzzing
LLMsmaypossesscomplexgrammaticalknowledgetofa- mayprocessmanysimilarinputsthatdonoteffectivelycover
cilitate continuous testing of software with complex input theinputspace.
formats.However,wefindthatLLMsareoftenlesscapable
During ourpreliminary study,we attempted to calibrate
orevenincapableofgeneratingnon-textualoutputs.Inpartic-
the diversityofsynthesizedgenerators withseveraltactics,
ular,whilemodernfuzztestingfrequentlytargetsnon-textual
suchastemperaturecontrolandtop-k sampling,butthere-
inputslikeimageprocessinglibraries,general-purposeLLMs
sultswerenotsatisfying.Forexample,whileonemayinstruct
arenotdesignedtogeneratesuchnon-textualdata.Moreover,
LLMsto“generate100JPEGimagegeneratorsthatareas
whilecutting-edgeLLMssuchasDALL-E[1]cangenerate
diverseaspossible”,wefindthatmanyofthegeneratedim-
images,ourtentativeexplorationshowsthattheimageformat
agesaresimplyrepeated,and“100”isalreadytoolargefor
islimitedtoasmallsetofpredefinedformats.DALL-Eonly
theLLMtoprocessinonego.Recentworkspointoutthat
supportsgeneratingimagesincommonimageformats,such
usingonlyLLMstogeneratediversesamplesisinherently
asPNGandGIF,evenifwespecifyrequiringotherformats
challenging[25,52];thisisoftenreferredtoasthe“tailphe-
(TIFF,RAW,BMP,etc.)intheprompts.Moreover,DALL-E
nomena”,wheretheLLMstendtogeneratealargenumberof
usage costs $40.00 per 1,000 images,making it expensive
similarsamplesandonlyasmallnumberofdiversesamples.
forlarge-scalefuzzing.Generatinganinputsamplecantake
severalseconds,significantlyimpactingfuzzingthroughput. TechnicalChallengeII:Overhead.Real-worldfuzzingcam-
Besidesimages,othernon-textualfiletypes,suchasMP4for paignsrequiremanyinputsamplestobegeneratedandtested,
video,MP3foraudio,PDFfordocuments,andBinaryLarge andthe suggestedfuzzing duration is often in the orderof
Object(BLOB),areoftennotsupportedbyexistingLLMs. daysorweeks.Thisraisesasevereconcernonoverhead.For
FindingallcustomizedLLMscanbechallenging. example,the costofusing GPT-4 Turbo is estimatedto be
$10.00per1Mtokens,andthecostofusingDALL-Eises-
WeanalyzetheinputformatdistributionsofFuzzBench
timatedtobe$40.00per1,000images.Giventhatasingle
programs.FuzzBench[38]isonemostwidely-usedbench-
fuzzingcampaignmayrequiremillionsoftokensorimages,
markingplatformdevelopedbyGoogletoevaluatefuzzing.
thecostofusingLLMscanbeprohibitivelyhigh.
Itincludesmanywidelyusedopen-sourceprojectsthatpro-
cess a variety ofinputformats. We believe the analysis re- ThetimecostofusingLLMsisalsohigh,asthegeneration
sultswillbegeneralizableduetothesizeanddiversityofthe ofasingleinputsamplemaytakeseveralsecondsorupto
benchmarks.Wefindthat73%oftheprogramsonlyaccept twenty seconds,depending on the complexity of the input
non-textualinputs.Programsthatacceptnon-textualinputs formatandthequalityofthegeneratedsample.Thisisnot
aremorecommonthanthoseacceptingtextualinputsintra- practicalforfuzzing,asthefuzzerisexpectedtogenerateand
ditionalfuzzing.Fortheseprograms,generalLLMscannot testalargenumberofinputsamplesinashorttime.Suppose
directlygeneratefuzzinginputs(reasonsdiscussedbefore). eachJPEGimagetakes10secondstogenerate,andthefuzzer
Moreover,whilesomecutting-edgeLLMscangeneratePNG needstogenerate1,000,000images.Thiswilltake115days
andJPEGinputs,otherformatsstilllacksupport. tocomplete,whichisnotpracticalinreal-worldfuzzing.

4 Design byte-levelmutationsandexploringtheinputspacesystem-
aticallyatlowcost.However,conventionalmutation-based
InlinewithchallengesnotedinSec.3,wepresentG2FUZZ, fuzzers often lack the grammatical knowledge to generate
a novel and efficient approach to augment mutation-based high-qualityinputsamples,andtheyoftenlackthe“bigpic-
fuzzingwithLLMs.Fig.1illustratesthehigh-leveldesignof ture”toprogressivelyexploretheinputspace.Agoodsynergy
G2FUZZ.G2FUZZfeaturesahybridstrategythatcombinesa betweenLLMsandmutation-basedfuzzerscanbeachieved,
“holisticsearch”drivenbyLLMsanda“localsearch”driven whereLLMsexcelatsynthesizinginputgeneratorsanden-
byindustrial-qualitymutation-basedfuzzers.Thekeyideais ablingescapefromlocaloptima,andmutation-basedfuzzers
toleverageLLMstoautomaticallysynthesizeinputgenera- excelatdeeplyexploringthelocalinputspace.Thisallevi-
torsthatarecustomizedtothespecificfeatures,structures,and atesthelimitationsofbothLLMsandconventionalmutation-
grammarofthetargetfileformat.Wefurthermutatethesyn- based fuzzers,and achieves better performance than using
thesizedgeneratorstoenhancetheirdiversity(seeSec.4.1). eitherofthemalone;seeevaluationsinSec.5.
Byrunningthesegenerators,G2FUZZcanobtainseedswith Addressing Technical Challenge I. Challenge I concerns
diversestructuresandfeatures.Then,thosegeneratedinputs thelackofdiversityinLLM-generatedoutputs.Asaforemen-
canbefurthermutatedbytraditionalmutation-basedfuzzers tioned,LLMsareinherentlypronetothe“tailphenomena”
(AFL++) to explore the input space more effectively (see andoftengenerateoutputsthatarerepeatedorverysimilar
Sec.4.2).WhenG2FUZZcannotidentifyanewpathduring toeachother.RatherthandirectlyaskingLLMstoproduce
the local search, it switches back to the holistic search to “diverse”generators,G2FUZZfirstanalyzesthepossiblefea-
generatenewinputgenerators. turesofatargetfileformat,andthenusesLLMstosynthesize
G2FUZZcomprisestwocorecomponents:inputgenerator inputgeneratorstailoredtospecificfeatures/structuresofthe
synthesis and input generatormutation. In input generator targetfileformat.Wealsoproposeasetofstrategiestoextend
synthesis,G2FUZZfirstanalyzesthefilefeaturesforthetarget andmutatethesynthesizedgenerators.
formatandsynthesizesaninputgeneratorforeachfeature.At AddressingTechnicalChallengeII.ChallengeIIconcerns
thisstage,weobtainsomeinitial,rathersimplegenerators. thehighcostofLLMusage.Weaddressthischallengeina
In the generator mutation stage, G2FUZZ aims to produce principledmanner,whereweonlyinvokeLLMstogenerate
generators customizedto multiple features orstructures si- newinputgeneratorswhenneeded.Holistically,LLMsare
multaneously,whichcanyieldmorecomplexfuzzinginputs onlyinvokedwhenthelocalsearch(conductedbyAFL++)
and enhance the generator diversity. We also evaluate the cannotidentifynewedges.Thislargelyreducesthecostof
performanceofeachgeneratorbasedonmutationfeedback LLMusage,from15.16$(ourLLM-baselinesetting;seecom-
duringfuzzingandextractusefulknowledgefromthesuccess- parisonsinSec.5)to0.124$,thusmakingG2FUZZpractical
fulgeneratormutationstoguidefuturemutationdirections. andcost-friendlyinreal-worldfuzzingcampaigns.
Exceptforspecifyingthetargetfileformat(whichrequiresthe Application Scope. G2FUZZ is designed to be general-
usertoprovide),thewholeprocessofG2FUZZisautomated.
purposeandapplicabletoawiderangeofinputformats.We
The bottom flowchart in Fig. 1 specifies the pipeline of haveevaluatedG2FUZZonavarietyofinputformats,includ-
standardmutation-basedfuzzing.Westartwithinitialseeds, ingJPEGimages,TIFFimages,MP4videos,and31other
addthemtotheseedqueue,andthenselectaseedtomutate. formats.ThedesignofG2FUZZisnotspecifictoanypartic-
Finally,wemutatethetargetseedunderthepredefinedmu- ular input format,and it can be easily extended to support
tationstrategyandcheckifthemutatedinputstriggerbugs. newinputformats.WithmodernLLMsincreasinglycapable
G2FUZZaugmentsthestandardpipelinebyincorporatingthe
ofgainingcomplexgrammaticalknowledgeandcopingwith
above two LLM-basedcomponents. Before seedselection, advanceddatatypes(e.g.,videosandaudio),wearepositive
weobtainthefuzzingstatebasedonthefuzzingperformance. thatG2FUZZcanbeusedtoaugmentmutation-basedfuzzing
Ifitisthefirstcycleoffuzzing,weperformbothinputgen- forawiderangeofinputformats.Weleavetheexploration
eratorsynthesisandinputgeneratormutationtoiteratebasic ofthoseadvanceddatatypestofuturework.
featuresandenrichourinitialseedcorpus.Ifthefuzzingpro-
cesscannotfindanewpathforalongtime,G2FUZZdirectly
mutatesinputgenerators,creatingmorecomplexgenerators 4.1 InputGeneratorSynthesis
throughfeaturecombinationsandpresumablyenablingthe
fuzzingcampaigntoescapefrom“localoptima.” When to Use. Before entering the formal fuzzing loop,
SynergyEffect.Wehighlightthat G2FUZZ featuresasyn- G2FUZZfirstobtainsthetargetinputfileformatfromtheuser
ergisticeffectwhencombinedwithmutation-basedfuzzers. (e.g.,“TIFF”;thisistheonlyinformationrequired),extracts
LLMsareknowledgeableaboutthegrammaticalinformation its features, and synthesizes the corresponding generators.
ofvariousinputformats,yettheyarelesscapableofgener- Then,itrunsthesegeneratorspreviously-synthesizedbythe
ating those non-textual inputs directly. On the other hand, LLMtoproducenewdiverseseedsandaddsthemtotheseed
mutation-basedfuzzersaregoodatperformingfine-grained, queue.Notethatifasoftwareacceptsmultipleinputfilefor-

|     | Input Generator Synthesis |     |     |     | F1  |     |     |     |     |     |     |     |
| --- | ------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Generator
|     |             |        |     |     | F2  | F o r  E a c | h       |        |     |     |        |     |
| --- | ----------- | ------ | --- | --- | --- | ------------ | ------- | ------ | --- | --- | ------ | --- |
|     | TIFF        |        |     |     | ... | F e a tu r e | Fx      |        |     | Gx  |        |     |
|     |             |        |     |     |     |              | Feature | Prompt |     |     | Seed X |     |
|     | File Format | Prompt |     | LLM | FN  |              |         |        | LLM |     |        |     |
Features
Generator database
|     | tceleS | Feature database |     |     |     |     |     |     |     |     |     |     |
| --- | ------ | ---------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
LLM
|     |     |     |     |     |     | OR  |     |     |     |     | Seed M |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------ | --- |
Feature-Structure Havoc Mutation
Rare-Feature Directed Mutation
|     | Generator |     |     |     |     |     |     |     |     | Mutated Generator |     |     |
| --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | ----------------- | --- | --- |
Generator Mutation
|     |     | Useful mutation database |            |     | Pattern-Based Mutation |                          |      |          |     |      |     |     |
| --- | --- | ------------------------ | ---------- | --- | ---------------------- | ------------------------ | ---- | -------- | --- | ---- | --- | --- |
|     |     |                          |            |     |                        | Add new seeds into queue |      | Feedback |     |      |     |     |
|     |     |                          |            |     | Stall / Init           |                          | Seed |          |     |      |     |     |
|     |     | Seeds                    | Seed Queue |     | Check State            | Seed Selection           |      | Mutation |     | Bugs |     |     |
Figure1:TheworkflowofG2FUZZ.
mats,G2FUZZ
analyzeseachformatindividuallytoobtain Algorithm1:GeneratorProcedure
thecorrespondinggenerators.
Input:Thetargetfileformat,target_format.
DesignConsideration:Featuresvs.Structures.Todescribe Output:Asetof(generator,featuredescription),G.
|                                                      |     |     |     |     |     |     | prompt←construct_prompt(target_format) |     |     |     | // Based on | Fig. 2 |
| ---------------------------------------------------- | --- | --- | --- | --- | --- | --- | -------------------------------------- | --- | --- | --- | ----------- | ------ |
| afile,twomainaspectscanbeconsidered:featureandstruc- |     |     |     |     |     |     | 1                                      |     |     |     |             |        |
2 features←LLM(prompt)
forfinfeaturesdo
| ture.“Filefeature”referstoattributesorcharacteristicsthat |     |     |     |     |     |     | 3                                                      |     |     |     |     |     |
| --------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------ | --- | --- | --- | --- | --- |
|                                                           |     |     |     |     |     |     | 4 g←generator_generation(target_format,target_feature) |     |     |     |     |     |
can provide external details about the file. “File structure” // Based on Alg. 2
|     |     |     |     |     |     |     | 5 ifg̸=nonethen |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --- | --- | --- | --- |
referstothewaydatawithinafileisorganizedandformatted,
|     |     |     |     |     |     |     | 6 seeds←run(g) |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --- | --- | --- | --- |
providinginternaldetailsabouthowdataisarrangedwithin add_to_queue(seeds)
7
| thefile.Usingfilestructuretodescribeadesiredfileinputis |     |     |     |     |     |     | 8 G←(g,f) |     |     |     |     |     |
| ------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --------- | --- | --- | --- | --- | --- |
amorestraightforwardapproach.However,thereisagapbe-
tweenspecifyingstructureandpreparingthegeneratorPython
code.ThedocumentofPythonfilelibrariesoftenlacksde- features(lines1-2).Foreachfeature,G2FUZZleveragesan
LLMtocreateageneratorforit(lines3-4).Then,G2FUZZ
tailsonhowtowritecodetoyieldaspecificfilestructure.In
Python,constructingafilewithaspecificstructureisnotlike runsthegeneratorstoobtainseedswithvariousfeaturesand
addstheseseedstotheseedqueueinfuzzing(lines6-8).
buildingwithblocks,whereonechunkcanbeaddedatatime;
instead,thefileisoftenconstructedfromamoreholisticper- FeatureAnalysis.Astherearemanyfeaturedescriptionsin
spective.ThismakesitdifficultforanLLMtounderstandand thedocumentofPythonfilelibraries,theLLMcansynthesize
ageneratorproducingafilewiththespecificfeature.Toobtain
usethelibrariestoachievecertainstructures.Also,relations
amongchunkscanbecomplexandhaveintricatedependen- thefeaturesforagivenfileformat,weinstructtheLLMto
|     |     |     |     |     |     |     | summarize | the possible | features. | The | prompt is | shown in |
| --- | --- | --- | --- | --- | --- | --- | --------- | ------------ | --------- | --- | --------- | -------- |
cies.Ourtentativestudyshowsthatcreatinggeneratorsbased
onstructureshasahighfailurerate,consumingmuchtime Fig.2.Forexample,whenapplyingthisprompttoextractthe
andnegativelyaffectingfuzzingthroughput. featuresofTIFFfiles,theoutputmightinclude:1.Lossless
compression:TIFFfilessupport...2.Multiplelayers:....We
| We find | that relevant | Python | file libraries | often | provide |     |     |     |     |     |     |     |
| ------- | ------------- | ------ | -------------- | ----- | ------- | --- | --- | --- | --- | --- | --- | --- |
donotlimitthenumberoffeatures,asdifferentfileformats
APIstoimplementfeaturesforspecificfileformats,suchas
havevaryingrangesoffeatures.Weaimtocapturecommon
| the compression | flag | in libtiff | forstoring | TIFF | files. | In  |     |     |     |     |     |     |
| --------------- | ---- | ---------- | ---------- | ---- | ------ | --- | --- | --- | --- | --- | --- | --- |
thesecases,featuresandcodehaveadirectmap,asthedocu- featuresatthisstep,andexploreunusualfeaturesinSec.4.2.
mentoftheselibrariesincludescorrespondingdescriptions.
LLMscanlearnfromthisinformation,makingfilefeatures What features can '<TARGET>' files have? Output the
information in the following format:
easyforthemtounderstand.Consequently,thetransformation
1. <feature 1>: <feature description>
fromfeaturestoaPythongeneratorcodeisstraightforward.
2. <feature 2>: <feature description>
| Indeed,sincefilefeaturesalsoencompassstructuraldescrip- |     |     |     |     |     |     | . . . . . .                                |     |     |     |     |     |
| ------------------------------------------------------- | --- | --- | --- | --- | --- | --- | ------------------------------------------ | --- | --- | --- | --- | --- |
|                                                         |     |     |     |     |     |     | N .  < f e ature N>: <feature description> |     |     |     |     |     |
tionsandcomplexconstraints,generatinginputwithaspecific
featuremustadheretothegrammarandstructuralconstraints. Figure2:Thepromptusedtoanalyzethefeaturesofaspecific
program.
Therefore,weusefilefeaturestosynthesizegenerators.
Overview.Inputgeneratorsynthesishastwosteps:givena GeneratorSynthesis.Afterweobtainthefeaturesforaspe-
fileformat,wefirstperform featureanalysistoidentifyall cificfileformat,wesynthesizeageneratorforeachfeature.
possible features. Then,for each feature,we ask LLMs to Forthegenerator,wehavetworequirements:(i)itshouldbe
synthesizeageneratorthatproducesaninputwiththetarget writteninPython,and(ii)itshouldbeexecutable.Obtaininga
feature. As shown in Alg. 1,given a file format, G2FUZZ PythongeneratorisstraightforwardforLLMs.However,there
constructsapromptandaskstheLLMforthecorresponding areseveralchallengesforgeneratorstorunsmoothly.First,

Algorithm2:GeneratorSynthesisAlgorithm uptoDEBUG_MAX timesstillfailstoproduceavalidgener-
ator,thealgorithmattemptstogenerateanewinitialgenerator
Input:Thetargetfileformat,target_format.Thetargetfilefeature,
target_feature. (backtoline3).Thisiscrucialbecause,duetothestochastic
Output:Avalidgeneratorthatcangenerateafilewithspecificfeatures,g.
|     |     |     |     |     | nature of LLMs,the | same prompt | can yield generators | of  |
| --- | --- | --- | --- | --- | ------------------ | ----------- | -------------------- | --- |
1 init_cnt←0
varyingquality,helpingtoavoidgettingstuckin“localmin-
2 whileinit_cnt<INIT_MAXdo
3 dialogue←[]
ima”(lines9-12andlines26-30).Basedonourpreliminary
4 prompt←construct_prompt(target_format,target_feature)
// Based on Fig. 3 exploration,wesetINIT_MAX to2andDEBUG_MAX to3
5 dialogue.append(prompt)
g←LLM(dialogue) tobalancethetrade-offbetweenthequalityofthegenerated
6
| 7 status,msg←exec(g) |     |     |     |     | generatorandthetimecost. |     |     |     |
| -------------------- | --- | --- | --- | --- | ------------------------ | --- | --- | --- |
8 debug_cnt←0
whiledebug_cnt<DEBUG_MAXdo
9
| 10  | ifstatus==SUCCESSthen |     |     |     |     |     |     |     |
| --- | --------------------- | --- | --- | --- | --- | --- | --- | --- |
ruturng Generate ‘<TARGET>’ files containing the following features
11
12 error_info←get_msg(msg) using Python without any input files, and save the
| 13  | whileTRUEdo |     |     |     | generated files into ‘./tmp/’. |     |     |     |
| --- | ----------- | --- | --- | --- | ------------------------------ | --- | --- | --- |
if”ModuleNotFoundError”notinerror_infothen
| 14  |                                             |     |     |     | ```               |     |     |     |
| --- | ------------------------------------------- | --- | --- | --- | ----------------- | --- | --- | --- |
| 15  | break                                       |     |     |     |                   |     |     |     |
|     | library_prompt←construct_prompt(error_info) |     |     |     | <TARGET_FEATURES> |     |     |     |
16
|     |     |     | // Based | on Fig. 4 | ``` |     |     |     |
| --- | --- | --- | -------- | --------- | --- | --- | --- | --- |
17 relied_library←LLM(library_prompt) Please use Markdown syntax to represent code blocks. Please
flag,g=automatic_installation(relied_library)
18 ensure that there is only one code block. You don't need to
| 19  | ifflag==0then |           |                |         |                                               |     |     |     |
| --- | ------------- | --------- | -------------- | ------- | --------------------------------------------- | --- | --- | --- |
|     |               | // Failed | to install the | library | tell me which libraries need to be installed. |     |     |     |
| 20  | ruturnNone    |           |                |         |                                               |     |     |     |
21 status,msg←exec(g) Figure3:Thepromptfordevelopingageneratorfromaspe-
ifstatus==SUCCESSthen
22
| 23  | ruturng |     |     |     | cificfeature. |     |     |     |
| --- | ------- | --- | --- | --- | ------------- | --- | --- | --- |
| 24  | else    |     |     |     |               |     |     |     |
error_info←get_msg(msg)
25
| 26  | dialogue.append(g) |     |     |     |     |     |     |     |
| --- | ------------------ | --- | --- | --- | --- | --- | --- | --- |
```
| 27  | dialogue.append(error_info+“Regenerate”) |     |     |     |       |     |     |     |
| --- | ---------------------------------------- | --- | --- | --- | ----- | --- | --- | --- |
| 28  | G←LLM(dialogue)                          |     |     |     | <MSG> |     |     |     |
| 29  | status,msg←exec(g)                       |     |     |     | ```   |     |     |     |
debug_cnt←debug_cnt+1
30 Please use Markdown syntax to represent the command. Please
31 init_cnt←init_cnt+1
ensure that there is only one command. To solve the above
issue using Python's package manager pip, you should run the
following command in the command-line interface:
as seed construction may rely on certain Python libraries, Figure4:Thepromptforextractingtherequiredlibrary.
| it is common | to encounter | the ModuleNotFound |            | problem   |     |     |     |     |
| ------------ | ------------ | ------------------ | ---------- | --------- | --- | --- | --- | --- |
| (Challenge   | I). Thus,we  | use the LLM        | to analyze | the error |     |     |     |     |
4.2 GeneratorMutation
| information | toautomaticallyidentifytherequiredlibraries |     |     |     |     |     |     |     |
| ----------- | ------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
andinstallthem.Second,asLLMscannotensurethevalidity
ThegeneratorsobtainedfromSec.4.1generateaseedwith
ofgeneratedcodes,thecodegeneratedbyanLLMmaycon-
asinglespecificfeature,whichoftencoversasmallpartof
tainsomebugs(ChallengeII).Weproposeanalgorithmto
thefeaturespace.Toeffectivelyutilizethemutationfeedback
automaticallydebugthegeneratedcode.
informationfromfuzzingandcoveralargerfeaturespace,we
| As in Alg. | 2,the synthesis | algorithm | first constructs | an  |     |     |     |     |
| ---------- | --------------- | --------- | ---------------- | --- | --- | --- | --- | --- |
takeintoaccountmorecomplexfeaturesusingthefollowing
initialgeneratorbasedonthetargetfileformatandthedesired
threemutationstrategies:Rare-FeatureDirectedMutation:
feature,andrunsittoobtaintheexecutionstatus(lines3-7).
Weincorporatehistoricalinformation—specifically,thefea-
TheprompttemplateusedisshowninFig.3.Iftheexecution turesthathavealreadybeencoveredbythegenerators—into
fails,thespecificerrormessageisextracted(line12).
theprompttoguidetheLLMinextractingunanalyzedfea-
Ifthe errorinvolves a missing module,the algorithm at- tures,focusingspecificallyonaddingtheserarefeaturesto
temptstoinstalltherequiredlibrary.Thisprocessinvolves theexistinggenerators.Feature-StructureHavocMutation:
constructingapromptbasedontheerrorinformation,using Weaddarandomfeature/structuretotheexistinggenerators,
theLLMtoidentifythemissinglibrary,andthenattempting aimingtounleashthepotentialupperboundcapabilityofthe
anautomatedinstallation(lines14-18).Theprompttemplate
LLM.Pattern-BasedMutation:Asdifferentfeaturesmay
is in Fig. 4. If the installation is successful, the generator exertvaryinginfluencesonthetargetprogram,weleverage
isre-executed,andiftheexecutionstatusisSUCCESS,the historical information to extract useful features and retain
validgeneratorisreturned(lines22-23).Otherwise,theerror thembycombiningthemwithotherfeatures.Thus,weuse
handlingloopcontinues. thefeedbackinformationfromthefuzzingprocesstoguide
Afterresolvingthelibrarydependencyissue,theerrorin- thegeneratormutations.
formationisfedbackintotheLLMtoregenerateaprogram WhentoUse.Inthefuzzingprocess,generatormutationisex-
thatcanresolvethecurrenterror(lines26-28).Ifdebugging ecutedintwospecificsituations.First,whenG2FUZZinitially

Algorithm3:GeneratorMutationAlgorithm providearoundtenfeaturesatatimebutoftenneglectrare
featuresandcannotgeneratethemdirectly.
Input:Thetargetfileformat,format.
Output:Agenerator,gm. To achieve rare feature mutation, we maintain a fea-
1 state←get_fuzz_state() turedatabasethatcollectsanalyzedfeaturesasdescribedin
2 ifstate==initthen
3 prompt←construct_prompt(target_format) // Based on Fig. 5 Sec.4.1. Onceafeaturehasbeen analyzedtosynthesizea
4 features←LLM(prompt) generator,itsname,andcorrespondingdescriptionareadded
5 forfinfeaturesdo
6 g←generator_select() tothefeaturedatabase.Wethenincorporatetheseanalyzed
7 prompt←construct_prompt(format,g,f) // Based on features into a prompt and ask the LLM to identify other
Fig. 6
8 gm←LLM(prompt) unexploredfeatures,asillustratedinFig.5.
9 gm←self_debug(gm) // Reuse the code lines 9 - 30 in
Alg. 2 Atthesametime,westoreallthesynthesizedgenerators
10 seeds←run(gm) inthegeneratordatabase,andwerandomlyselectagenerator
11 add_to_queue(seeds)
fromthisdatabase.Afterward,weasktheLLMtomutatethe
12 ifstate==stallthen
13 g←generator_select() selectedgeneratortoproduceafilethatincludesanadditional
14 mutator←mutator_choose()
rarefeaturealongsidetheexistingones.Thepromptforthis
15 ifmutator==featureormutator==structurethen
16 prompt←construct_prompt(format,g,mutator) // Based on stepisshowninFig.6.Finally,werunthemutatedgenera-
Fig. 13
tor,obtainnewseedswithmultiple(newly-added)features,
17 elseifmutator==patternthen
18 example←pre_mutation_select() andaddthisseedtotheseedqueue.Giventhatthismethod
19 prompt←construct_prompt(g,example)// Based on Fig. 7 requiresputtingallpreviouslyanalyzedfeaturedescriptions
20 gm←LLM(prompt)
21 gm←self_debug(gm) // Reuse the code lines 9 - 30 in intoaprompt,weonlyusethisstrategythefirsttimefuzzing
Alg. 2
entersthelooptoreducetokenoverhead.
22 seeds←run(gm)
23 add_to_queue(seeds)
Analyzed features:
```
1. <feature 1>: <feature description>
entersthefuzzingloop,weuserare-featuredirectedmutation
2. ... : ...
toenrichthevarietyoffeaturesintheinitialseeds.Second, ```
whenfuzzingfailstofindnewpathswithinasettimelimit Apart from the above features, what other features can '<TARGET>'
files have? Output the information in the following format:
(likelytrappedinalocaloptimum),weusefeature-structure 1. <feature 1>: <feature description>
2. <feature 2>: <feature description>
havocmutationandpattern-basedmutationtoconstructseeds ......
N. <feature N>: <feature description>
withdifferentfeaturesorstructures,whichcanhelpfuzzing
exploreothercoderegions.
Figure5:Thepromptforrarefeatureextraction.
Overview.Whenfuzzingstallsorinitializes,G2FUZZuses
generatormutationtogeneratemorecomplexinputs.Theal- ```
gorithmisinAlg.3.G2FUZZobtainsthecurrentfuzzingstate <TARGET_GENERATOR>
(line1).Ifitisinitialization,G2FUZZperformsrare-feature ```
The code above is used to generate <FROMAT> files. Now, we
directedmutation.Todoso,G2FUZZasksLLMstoextract
need to extend this code to generate a new <FROMAT> file
theunanalyzedfeaturesbasedonhistoricalinformation(lines that includes an additional `<NEW_FEATURE>` feature besides
3-4).Foreachunanalyzedfeature,G2FUZZrandomlyselects the existing features. The description of the
`<NEW_FEATURE>` feature is as follows:
a generator and asks LLMs to incorporate the unanalyzed
```
featureintoittocreatenewinputs(lines5-11). <FEATURE_DES>
Ifitis a stall, G2FUZZ performs feature-structure havoc ```
mutationorpattern-basedmutation. G2FUZZ randomlyse- Figure6:Thepromptforrarefeaturemutation.
lectsageneratorfromadatabasecontainingallexecutable Feature-StructureHavocMutation.AlthoughtheLLMis
generators(line13).Itthenrandomlychoosesamutatorto powerful,itsoutputcansometimesbeunstable.Toexplore
applytothisselectedgenerator(line14).Next,G2FUZZcon-
thefullpotentialoftheLLM,weaskittorandomlymutate
structsapromptbasedonthechosengeneratorandmutator the current generator to produce a file that includes an ad-
(lines15-19).Finally,G2FUZZretrievesamutatedgenerator
ditionalfeatureorstructurealongsidetheexistingfeatures.
fromtheLLM,runsittoobtainanewseed,andaddsthisseed ThepromptisshowninFig.13.Sinceageneratorcanbemu-
tothequeueforfurthermutation(lines20-23). tatedmultipletimes,itispossibletogenerateafilewithmany
Rare-FeatureDirectedMutation.Toimprovethecompre- featuresorstructures.TherandomnessoftheLLMmayintro-
hensivenessofourtesting,itisessentialtocoverrarefeatures ducerarefeaturesthatcannotbediscoveredthroughdirected
thatthegeneratorsfromSec.4.1mayhaveoverlooked.Our rare-featuremutation.Whilerare-featuredirectedmutation
tentativestudyshowsLLMscannotoftenidentifyallrelevant typicallygeneratesafilewithtwofeatures,feature-structure
featuresofafileformatinasinglerequest. Typically,they havocmutationcanproduceafilewithmorethantwofeatures.

Thisallowsfortheconstructionofmorecomplexgenerators, perform SOTA in terms of code coverage and the number
enablingustoexploreadeeperfeaturespace. ofuniquebugs?RQ2:CanG2FUZZ’sperformancesurpass
Pattern-BasedMutation.Giventhatdifferentfeaturesmay thatofstructure-awarefuzzers?RQ3:Howmanytokenswill
exertvaryinginfluencesonthetargetprogram,wepropose beconsumedwhenfuzzingaprogramfor24hours? RQ4:
pattern-basedmutation.Thisapproachuseshistoricalinfor- WhichpartofG2FUZZcontributesthemost?
mationtoextractusefulfeatures,whicharethenaccentuated
byintegratingthemwithotherfeatures.Feedbackfromthe
5.1 CodeCoverageandUniqueBugs
fuzzingprocesseffectivelyhighlightswhichmutationsresult
inmoreusefulgenerators(i.e.,thosecapableofdiscovering Code coverage and unique bugs are common metrics for
newedges).Byanalyzingthisfeedback,wecanaskLLMsto evaluating fuzzers. To ensure fairness and reproducibility,
learnthesemutationstrategies,thusguidingandoptimizing we conductourexperiments on UNIFUZZ,MAGMA,and
futuremutationdirections. FuzzBench.Allruntimesettings,includingtheinitialseeds,
The feature space of a file format is often vast,and not followthedefaultconfiguration.
everyuniquefeaturetriggersdistinctprocessinglogicinthe
targetprogram.Iteratingthroughallpossibilitiesisinefficient;
5.1.1 ExperimentsonUNIFUZZ
instead,wefocusonthefeaturesthatthetargetprogramis
interestedin,namely“program-relevant”features.Seedswith UNIFUZZ is an open-source and metrics-driven platform
program-relevantfeatureswillbeprocesseddifferentlybythe designedfortheholisticandfairevaluationoffuzzers.
targetprogram.Ifaseedobtainedfromamutatedgenerator
Compared Fuzzers. AFL++’s performance demonstrates
discoversanewpath,weinferthatthisseedcontainsprogram-
thattheimplementationsignificantlyaffectstestingefficiency.
relevantfeatures. Consequently,weconsiderthe<original
Toavoidtheinfluenceofimplementation,wehavedeveloped
generator,mutatedgenerator>tupletocontainusefulinfor- G2FUZZ basedon AFL++,integrating itas a mode within
mationandincorporateitintoourusefulpatterndatabase.
AFL++. Since AFL++ already incorporates many SOTA
Toreuseeffectivemutationstrategies,weemployLLMsto
fuzzers,this allows foreasyandfaircomparisons between
learnthemutationpatternsfromthemutationgeneratortuples G2FUZZandotherfuzzersimplementedinAFL++.Asthe
andapplythesepatternstoothergenerators.Thepromptis
rangeoffeaturesincorporatedintoAFL++wouldexceedthe
showninFig.7.Bydoingso,weaimtogenerateseedswith scopeofthispaper,wecomparedG2FUZZwiththefourmost
a richervarietyofprogram-relevantfeatures. In ourimple-
widelyusedconfigurationsofAFL++.AFL++(cmplog):en-
mentation,wetracetheperformanceofeachgeneratedseed
ablesREDQUEENmutator.AFL++(mopt):enablesMOPT
duringfuzzing.Ifausefulseed(i.e.,onethatdiscoversnew
mutator. AFL++(fast): enables AFLFast seed scheduling.
paths)isproducedthroughgeneratormutation,weconsider
AFL++(rare): prioritizes seeds that are rarely covered by
thismutationusefulforthisprogram.Therefore,weaddboth otherseeds.ForG2FUZZ,weenablecmplogmodetofacili-
themutatedandoriginalgeneratorstothepromptinFig.7
tateefficientlow-levelmutation.
andapplythismutationstrategytoothergeneratormutations. ProgramsSelection.SinceG2FUZZisdesignedfortesting
programswithnon-textualinputs,weselectedprogramsthat
The original code: ```<ORI>``` meetthiscriterion.Thetargetprograms,listedinTable17,
The mutated code: ```<MUT>```
include10programswithover20differentinputformattypes.
Imitate the mutation of 'The original generator -> The
mutated generator' above and apply it to the following Experiment Results. Table 1 shows the edge coverage
target code: achievedbyeightfuzzers.G2FUZZ(GPT-4)discoversatotal
```
of59,642edges,whichis15,437morethanthebestbaseline
<TARGET_CODE>
```
fuzzer,AFL++(cmplog).G2FUZZ(GPT-4)andG2FUZZ(GPT-
3.5)achievethehighestperformanceon9outof10programs,
Figure7:Thepromptforpattern-basedmutation. whileAFL++(cmplog)excelsononeprogram.Furthermore,
G2FUZZ(GPT-4)isabletodiscovermoreuniquebugsthan
5 Evaluation theotherbaselinefuzzers.Specifically,G2FUZZ(GPT-4)finds
143uniquebugs,whichis32morethanthebest-performing
G2FUZZ is built upon AFL++,enabling integration of our comparisonfuzzer,AFL++(cmplog).
methodwithotherexistingtechniques.Toensuretheaccu- Moreover, we also calculated the pairwise unique code
racyandfairnessofourresults,weconductedexperimentson coverage. We summed up the unique code coverage for
threetestingplatforms:UNIFUZZ,MAGMA,andFuzzBench. eachpairoffuzzersacrossallprograms,resultinginFig.8.
Theexperimentswerecarriedoutonthreesystemsrunning G2FUZZ(GPT-4) and G2FUZZ(GPT-3.5) are quite similar,
Ubuntu22.04,eachequippedwith64cores(Intel(R)Xeon(R) andbothareabletoidentifymoreuniquecodecoveragethan
Gold 6444Y CPU) and 256GB memory. We study the fol- theremainingfourfuzzers.Thisresultdemonstratesthatwith
lowing research questions (RQs): RQ1: Can the tool out- theassistanceofLLMs,theinputswegenerated,whichpos-

Table1:AveragecodecoverageandthetotaluniquecrashesfoundbyG2FUZZwithGPT-3.5/GPT-4and6comparedfuzzers.
G2FUZZ(GPT-3.5)1 G2FUZZ(GPT-4)2 AFL++(cmplog) AFL++(fast) AFL++(mopt) AFL++(rare)
Programs
|     |         | Cov.      | #Bug | Cov.     | #Bug | Cov.   | #Bug | Cov.   | #Bug | Cov.   | #Bug | Cov.   | #Bug |
| --- | ------- | --------- | ---- | -------- | ---- | ------ | ---- | ------ | ---- | ------ | ---- | ------ | ---- |
|     | exiv2   | 5,099     |      | 31 5,171 | 28   | 4,965  | 26   | 3,776  | 5    | 3,758  | 12   | 3,851  | 11   |
|     | ffmpeg  | 31,706    |      | 0 34,218 | 1    | 22,099 | 0    | 17,380 | 0    | 15,566 | 0    | 14,613 | 0    |
|     | flvmeta | 228       |      | 4 228    | 4    | 228    | 4    | 228    | 4    | 228    | 4    | 228    | 4    |
|     |         | gdk 2,958 |      | 6 2,327  | 2    | 2,172  | 5    | 2,093  | 4    | 2,082  | 2    | 1,991  | 4    |
|     | imginfo | 2,839     |      | 0 3,825  | 0    | 2,189  | 0    | 1,998  | 0    | 2,004  | 0    | 1,976  | 0    |
|     | jhead   | 315       |      | 13 491   | 23   | 445    | 21   | 195    | 3    | 195    | 3    | 195    | 4    |
|     | mp3gain | 921       |      | 11 921   | 12   | 923    | 11   | 900    | 10   | 899    | 8    | 891    | 10   |
|     | mp42aac | 2,067     |      | 17 2,700 | 14   | 2,091  | 13   | 1,178  | 6    | 1,157  | 3    | 1,135  | 2    |
pdftotext 8,265 43 7,921 49 7,434 25 6,483 25 6,376 28 11,257 28
|     | tiffsplit | 1,817 |     | 7 1,840 | 10  | 1,659 | 6   | 1,619 | 9   | 1,644 | 9   | 1,626 | 7   |
| --- | --------- | ----- | --- | ------- | --- | ----- | --- | ----- | --- | ----- | --- | ----- | --- |
1G2FUZZ(GPT-3.5):G2FUZZbasedonGPT-3.5.
2G2FUZZ(GPT-4):G2FUZZbasedonGPT-4.
sesscomplexfeatures,arecapableoftriggeringmoreintricate
programlogic.Consequently,thisleadstothediscoveryofa
higheramountofuniquecodecoverage.
G2FUZZincorporatesadditionalstepsintoAFL++,such
| as generatorsynthesis |     | andexecution. |     | To assess | the | impact |     |     |     |     |     |     |     |
| --------------------- | --- | ------------- | --- | --------- | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
ofthesestepsonfuzzingthroughput(i.e.,executionspeed),
wemeasurethetotalnumberofexecutionsforeachfuzzer.
| Fig. 14 presents |     | the total | numberof | executions | performed |     |     |     |     |     |     |     |     |
| ---------------- | --- | --------- | -------- | ---------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
byallprogramsthatthefuzzersrunafter24hours.There-
sultsindicatethatG2FUZZ’sthroughputdoesnotsignificantly
decreasecomparedtootherfuzzers.Wealsofuzzedcertain
programsfor48hourstoobserveG2FUZZ’sperformancein
thelaterstagesoffuzzing.Duringthe24to48-hourperiod,
G2FUZZdiscovered32,5,1,33,and89newedgesinimginfo,
jhead,mp3gain,mp42aac,andtiffsplit,respectively.
| Furthermore,we |     | evaluated | the | token costs | of LLMs | for |     |     |     |     |     |     |     |
| -------------- | --- | --------- | --- | ----------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |
fuzzing.G2FUZZminimizestokenusagebyreducingreliance Figure8:Pairwiseuniquecodecoverageacrossallprograms.
on LLMs for mutation. GPT-3.5 incurs costs of less than Eachcellrepresentsthenumberofcodebranchescoveredby
0.2$,andGPT-4lessthan13$for24hoursoffuzzing.More thefuzzerofthecolumnbutnotbythefuzzeroftherow.
detailscanbefoundinAppendixB.Additionally,theablation
obtainthreesetsofseedsfromtheLLM-generationalgorithm.
studyshowsthatbothcomponentsofG2FUZZareeffective,
Therefore,weneedtoperformthreesetsofexperiments.
| with generator |     | synthesis | and mutation | contributing |     | 82,001 |     |     |     |     |     |     |     |
| -------------- | --- | --------- | ------------ | ------------ | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
ProgramsSelection.SinceG2FUZZissuitableonlyfornon-
and141,340newpaths,respectively.LLM-onlyapproaches
textualinputs,weexcludedprogramswithtextualinputs,leav-
struggle,highlightingthenecessityofintegration.Formore
ing11programsfortestingG2FUZZ,aslistedinTable17.
details,pleaserefertoAppendixC.
Metric.Wechoosetheaveragerankoffuzzersasourevalua-
tionmetrictoassesseachfuzzer’sperformanceacrossmulti-
5.1.2 ExperimentsonFuzzBench
plebenchmarks.Byrankingthefuzzersoneachbenchmark
Toconductmorecomprehensiveexperiments,wealsoevalu- accordingtotheirmedianreachedcodecoverage,withlower
ateG2FUZZusingFuzzBench. valuesindicatingbetterperformance,wecanderiveanoverall
ExperimentSetup.FuzzBenchbuildsaDockerimagefor understandingoftheireffectiveness.
|                       |     |      |        |                  |     |      | Experiment |     | Results. |     | As shown | in  | Table 2, the perfor- |
| --------------------- | --- | ---- | ------ | ---------------- | --- | ---- | ---------- | --- | -------- | --- | -------- | --- | -------------------- |
| each fuzzer-benchmark |     | pair | to run | the experiments. |     | How- |            |     |          |     |          |     |                      |
G2FUZZ
ever,theresultingcontainerdoesnothaveaninternetconnec- mance of remains stable across different experi-
tion. Therefore,we modify the condition to run the LLM- mentalgroups.G2FUZZachievesthebestranksinallthree
groups,whichare2.09,2.18,and2.18.Thesecond-bestfuzzer,
| generation | algorithm | from | ‘stall/init’ | to ‘init’ | to create | a   |     |     |     |     |     |     |     |
| ---------- | --------- | ---- | ------------ | --------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
variantofG2FUZZ.Specifically,weruntheinputgenerator AFL++,achievesranksof2.73,2.73,and2.91intherespec-
synthesisandgeneratormutationwhenthefuzzingprocess tivegroups.InFigure9,weadditionallyprovidethecodecov-
firstentersthefuzzingloop.Thisallowsustorunthealgo- eragedistributionsforallfuzzersacrossallprogramswithin
rithm in advance and upload the results into the container oneoftheexperimentalgroups.3 G2FUZZachievesthehigh-
builtbyFuzzBench.Consequently,wecanconducttheexper-
3Thereportisavailableat:https://storage.googleapis.com/www.
imentswithoutneedinganinternetconnection.Tomitigate
fuzzbench.com/reports/experimental/2024-05-16-formatfuzz/
randomnessinLLMgeneration,weconductthreeepochsto
index.html.

estperformanceonfiveoutof11programs,whileLibAFL
Table3:ThetotalCVEsdiscovered(onMAGMA).G2FUZZ
andAFL++eachexcelontwoprograms,andLibFuzzerand performsthebestindiscoveringrealCVEs.
FairFuzzexcelononeprogrameach.
eff T ec h t e iv L e L ne M ss -g o e n ne c r e a r t t i a o i n n a p l r g o o g r r i a th m m s, d s e u m ch on a s s tr v a o t r e b s i r s e _ m de a c rk o a d b e l _ e - Program G 2 F U Z Z A F L + + M O P T A F L F ast Li b F
uzzer
E ntr o pic
fuzzer. We illustrate the average code coverage evolution
libpng_read_fuzzer 3 3 2 1 3 0
overtimeforvorbis_decode_fuzzerinFig.15.Notably,we
tiff_read_rgba_fuzzer 5 5 5 4 2 3
observethatG2FUZZachieveshighercoverageat15minutes
tiffcp 7 6 4 4 0 0
compared to all other compared fuzzers at 23 hours. This pdf_fuzzer 5 5 4 2 2 2
highlightsthecapabilityofG2FUZZtogeneratediverseand pdftoppm 6 5 6 2 0 0
pdfimages 6 5 5 3 0 0
complexstructures,enablingthediscoveryofcoderegions
thatarechallengingforconventionalfuzzerstouncover. Table4:Therealbugsdiscoveredbyeachfuzzer.
T
o
to
a
f
b
t
f
h
u
le
e
z
i
2
z
r
e
:
m
r
F
s
u
e
,
d
z
a
z
i
f
a
t
b
e
n
e
r
n
r
w
e
c
a
h
e
c
f
r
h
u
a
e
z
n
d
z
k
e
c
r
t
o
h
r
d
e
a
e
m
n
-
k
c
o
i
o
n
n
v
g
e
e
.
r
a
I
a
t
c
g
h
r
e
e
b
p
(
e
o
lo
n
r
w
t
c
s
h
e
t
m
r
he
i
a
s
r
a
k
b
v
e
e
a
t
r
c
t
a
e
c
g
r
o
e
)
r
.
d
ra
in
n
g
k
F U Z
Z
L +
+(c m
pl o g)
L +
+( m o
pt)
L +
+(f
ast)
L +
+(r
are)
2 F F F F
Program G A A A A
Fuzzers GroupI GroupII GroupIII Average
mp3gain 1 1 - 1 1
G2FUZZ 2.09 2.18 2.18 2.15 pdftotext 1 1 1 1 1
AFL++ 2.73 2.73 2.91 2.79 mp42aac 3 3 1 - -
LibAFL 4.55 4.55 4.64 4.58 mp42avc 3 - - - 1
LibFuzzer 4.73 4.64 4.64 4.67 mp42hevc 2 - - - -
HonggFuzz 6.45 6.45 6.36 6.42 Total 10 5 2 2 3
AFLSmart 6.73 6.73 6.73 6.73
AFL 7.27 7.27 7.27 7.27
MOPT 7.27 7.27 7.27 7.27
extensionsintheinitialseeds.Foropenssl,asMAGMA’sini-
Eclipser 7.36 7.36 7.27 7.3
FairFuzz 8.82 8.82 8.82 8.82 tialseedslackextensions,weexcludeitfromconsideration.
AFLFast 9.09 9.09 9.09 9.09 Toavoidrandomness,werepeattheexperiments5times.
Centipede 9.18 9.18 9.18 9.18
We analyze the number of CVEs found by each fuzzer,
AdditionalAnalysisTime.SinceweruntheLLM-generation whose results are in Table 3. We found that G2FUZZ per-
algorithmbeforetheexperiments,G2FUZZhasmorefuzzing formsthebestontheMAGMAbenchmark,uncoveringthe
time compared to other fuzzers. To assess its impact, we mostbugsinallprograms.Specifically,G2FUZZperformsthe
analyze additional analysis time perprogram,as shown in best on libpng_read_fuzzer, tiff_read_rgba_fuzzer,
Table 18. The program with the highest additional time is tiffcp,pdf_fuzzer,pdftoppmandpdfimages,exposing
bloaty_fuzz_targetat925seconds(1.06%of23hoursfuzzing 3,5,7,5,6and6bugs,respectively.
time),whilezlib_zlib_uncompress_fuzzerrequirestheleastat
163seconds(0.19%).Eightoutof11programsneedunder
500seconds(0.6%).Wealsofindthattheextra15minutes
requiredforLLM-generationhadnoeffectonmediancode 5.1.4 FindingBugsinLatestProgramVersions
coverageafter23hours,asshowninTable19.
ToevaluateG2FUZZ’sabilitytodiscoverrealbugs,wetest
the latest versions of projects from UNIFUZZ,along with
5.1.3 ExperimentsonMAGMA
allotherexecutableprogramsintheseprojectssuitablefor
Codecoverageanduniquebugsarekeymetrics,butdiscover- fuzzing.Eachfuzzer-programpairrunsfor24hoursandisre-
ingrealCVEsdirectlyshowsafuzzer’sabilitytofindsecurity peated5times.FollowingUNIFUZZ’ssuggestion,weusethe
vulnerabilitieswithsignificantreal-worldimpact.Toavoid topthreefunctionsfromtheASANoutputtode-duplicateun-
bias,weconductexperimentsonMAGMA,aground-truth coveredbugs.TheresultsareshownintheTable4.G2FUZZ
fuzzingbenchmarkwithreal-worldbugsforaccurateperfor- discoversatotalof10bugs,whilethebestcomparativefuzzer,
manceevaluation.WeintegrateG2FUZZintoMAGMAand AFL++(cmplog),identifies5.Notably,4bugsareexclusively
compareitwithfivefuzzers(i.e.,AFL++,MOPT,AFLFast, discoveredby G2FUZZ andremainundetectedbyallother
LibFuzzer,andEntropic). AllAFL++-relatedfuzzers used comparative fuzzers. By the time of writing, we have re-
inthisexperimentwerebasedonAFL++(commit1d17210) ported these bugs to the developers, and 3 of them have
andenabledtheRedQueenmutator.Weexcludedprograms been confirmed by CVE: CVE-2024-57509 (in mp42avc),
withtextualinputs.Table17showsthetargetprogramsused CVE-2024-57510 (in mp42avc), and CVE-2024-57513 (in
inMAGMA.Inputformattypesaredeterminedbyfilename mp42hevc).

|     | (a)bloaty |     |     | (b)freetype2 |     |     | (c)libpcap |     |     |     | (d)vorbis |     |     |
| --- | --------- | --- | --- | ------------ | --- | --- | ---------- | --- | --- | --- | --------- | --- | --- |
Figure9:CodecoveragedistributionsachievedinaFuzzBenchexperiment.Duetospaceconstraints,weonlypresenttheresults
onfourprograms.Forotherexperimentalresults,refertoFig.12.
Table5:ClassificationofformatshandledbyG2FUZZ.
5.1.5 ClassificationofHandledFileFormats
|        |                   |        |       |            |        | Category |     | Level | Formats |     |     | RelatedLibraries |     |
| ------ | ----------------- | ------ | ----- | ---------- | ------ | -------- | --- | ----- | ------- | --- | --- | ---------------- | --- |
| In the | above experiments | across | three | platforms, | we use |          |     |       |         |     |     |                  |     |
|        |                   |        |       |            |        |          |     |       | JPG     |     |     | PIL/piexif       |     |
G2FUZZtoevaluateitseffectivenessinhandlingvariousfile GIF PIL
|     |     |     | G2FUZZ |     |     |     |     |     | BMP |     |     | PIL |     |
| --- | --- | --- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
formats. As shown in Table 5, successfully pro- PNG PIL/matplotlib/cv2
|     |     |     |     |     |     |     |     | L11 | ICO |     |     | PIL |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
cessesavarietyofimageformatsincludingJPG,GIF,BMP,
|     |     |     |     |     |     |              |     |     | XMP |     |     | lxml/xml.dom.minidom |     |
| --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | -------------------- | --- |
|     |     |     |     |     |     | ImageFormats |     |     | TGA |     |     | PIL                  |     |
andPNG,aswellasseveralaudioandvideoformatssuchas
|                     |     |     |               |        |        |     |     |     | TIFF |     |     | PIL/tifffile |     |
| ------------------- | --- | --- | ------------- | ------ | ------ | --- | --- | --- | ---- | --- | --- | ------------ | --- |
| MP3,WAV,MP4,andFLV. |     |     | Additionally, | G2FUZZ | demon- |     |     |     | ANI  |     |     | PIL          |     |
|                     |     |     |               |        |        |     |     | L22 | RAS  |     |     | PIL          |     |
stratescapabilityinhandlingPDFdocumentsandvariousfont
|     |     |     |     |     |     |     |     |     | PGX     |     |     | PIL |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | --- | --- | --- | --- |
|     |     |     |     |     |     |     |     | L33 | PNM/RAW |     |     | -   |     |
formatslikeTTFandOTF.Intermsoffileformats,itsupports
|     |     |     |     |     |     |     |     |     | OGG |     |     | soundfile |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- |
processingformatssuchasELF,Mach_O,andWebAssembly.
|     |     |     |     |     |     |     |     |     | MP3 |     |     | pydub/mutagen |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- |
|     |     |     |     |     |     |     |     | L1  | WAV |     |     | wave/scipy    |     |
Allprogramsassociatedwiththe34formatswereevaluatedin AIFF soundfile/pydub/wave
AudioFormats
thepreviousexperiments.ThesefindingshighlightG2FUZZ’s AIFC aifc
|     |     |     |     |     |     |     |     | L3  | AU/CAF |     |     | -   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------ | --- | --- | --- | --- |
strongperformanceinfuzztestingacrossdiversefiletypes.
|     |     |     |     |     |     |              |     |     | FLV |     |     | moviepy |     |
| --- | --- | --- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | ------- | --- |
|     |     |     |     |     |     | VideoFormats |     | L1  |     |     |     |         |     |
Theconditionsofconstructingdifferentfileformatsvary: MP4 cv2/moviepy/mutagen
someformatsaresupportedbyspecificlibraries,whileothers DocumentFormats L1 PDF fpdf/PyPDF2/reportlab
are not. We classify these conditions into three levels. (1) FontFormats L1 TTF/OTF/WOFF/TTC fontTools
|     |     |     |     |     |     |     |     |     | Zlibcompressed |     |     | zlib |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --- | ---- | --- |
L1:Thetargetformathasspecificlibrariesthatcandirectly L1 PCAP scapy
FileFormats
generatefiles.(2)L2:Somecomponentsoftheformatcanbe DERcertificate cryptography
|     |     |     |     |     |     |     |     | L3  | ELF/MachO/WebAssembly/ICCprofile |     |     | -   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | -------------------------------- | --- | --- | --- | --- |
generatedusingexistinglibraries,andthesecomponentsare 1L1:Usespecializedlibrariestocreatefilesinthetargetformat.
2L2:Constructpartsofthefilewithspecificlibrariesandorganizethemaccordingtothetarget
thenorganizedaccordingtothetargetformat’ssyntaxrules.
format’ssyntaxrules.
(3)L3:Filesaregeneratedentirelyfromscratch,basedsolely 3L3:Buildthefilefromscratchbasedonthetargetformat’srules,directlywritingbinarydataor
usingstructtowritethedata.
onthetargetformat’ssyntaxrules.Amongthe34testedfor-
| mats,23 | fall into L1,3 | into | L2,and | 8 into L3. Formats | in  |     |     |     |     |     |     |     |     |
| ------- | -------------- | ---- | ------ | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
haveonlybeenassessedontext-basedgrammarinputformats.
L1typicallyleadtohigher-qualitygeneratorsthesupporting
|     |     |     |     |     |     | Moreover,we |     | do  | not include | FuzzInMem,ProFuzzer,and |     |     |     |
| --- | --- | --- | --- | --- | --- | ----------- | --- | --- | ----------- | ----------------------- | --- | --- | --- |
libraries–oftenaccompaniedbydocumentation,samplecode,
GreyOnebecausetheyhavenotbeenmadeopensource.
| andotherresources–are |                   | includedin  |     | LLM training      | data,in- |     |     |     |     |     |     |     |     |
| --------------------- | ----------------- | ----------- | --- | ----------------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
| creasing              | boththe diversity | andaccuracy |     | ofgeneratedfiles. |          |     |     |     |     |     |     |     |     |
Table6:TheaveragelinecoveragediscoveredbyG2FUZZ,
Nevertheless,wealsoobservedecent-qualitygeneratorsfor
FormatFuzzer,andWEIZZ.
formatsinL2andL3,whichdemonstratestherobustnessof
| G2FUZZinhandlingvariousformats. |     |     |     |     |     |          |     | G2FUZZ |          |              |          |      |          |
| ------------------------------- | --- | --- | --- | --- | --- | -------- | --- | ------ | -------- | ------------ | -------- | ---- | -------- |
|                                 |     |     |     |     |     | Programs |     |        |          | FormatFuzzer |          |      | WEIZZ    |
|                                 |     |     |     |     |     |          |     | line   | function | line         | function | line | function |
5.2 ComparedwithStructure-AwareFuzzers exiv2 5,984 1488 1,138 369 3,732 1025
|     |     |     |     |     |     |     | ffmpeg   | 53,664 | 3028 | 23,114 | 1554 | 26,789 | 1795 |
| --- | --- | --- | --- | --- | --- | --- | -------- | ------ | ---- | ------ | ---- | ------ | ---- |
|     |     |     |     |     |     | fl  | v m e ta | 6 2 3  | 5 9  | - 1    | -    | 6 3 2  | 6 0  |
WecompareG2FUZZwiththeSOTAgrammar-awarefuzzer
|     |     |     |     |     |     | i   | m g in f o | 5 ,0 0 3 | 3 6 4 | 2,1 2 8 | 19 3 | 3 ,4 8 1 | 2 7 5 |
| --- | --- | --- | --- | --- | --- | --- | ---------- | -------- | ----- | ------- | ---- | -------- | ----- |
FormatFuzzerandtheSOTAinference-basedfuzzerWEIZZ jhead 431 21 239 16 300 18
#2
using the UNIFUZZ benchmark. We do not compare mp3gain 2,168 58 # 2,103 56
|     |     |     |     |     |     | mp42aac |     | 3,378 | 811 | #   | #   | 2,041 | 504 |
| --- | --- | --- | --- | --- | --- | ------- | --- | ----- | --- | --- | --- | ----- | --- |
againstAFLSmartbecauseithasalreadybeencomparedin
|     |     |     |     |     |     | pdftotext |     | 13,733 | 1182 | -   | -   | 9,133 | 914 |
| --- | --- | --- | --- | --- | --- | --------- | --- | ------ | ---- | --- | --- | ----- | --- |
FuzzBench,whereG2FUZZsignificantlyoutperformsAFLS- tiffsplit 3,176 194 - - 3,019 185
=3
mart. G2FUZZ’saveragecodecoveragerankis2.15,while gdk 4,856 315 2,287 192 =
1-:FormatFuzzerdoesnotsupportPDF,TIFF,andFLVformats.
AFLSmart’sis6.73.Duetotheconsiderablegap,wedonot
2#:WeencounteredissueswhilerunningFormatFuzz.
conduct additional experiments here. We do not compare 3=:Weareunabletocompilegdk-pixbufbyWEIZZ.
| against | Superion,Nautilus,and |     | Grimoire,as | these | fuzzers |     |     |     |     |     |     |     |     |
| ------- | --------------------- | --- | ----------- | ----- | ------- | --- | --- | --- | --- | --- | --- | --- | --- |

Table7:Functionsexclusivelydiscoveredbyeachfuzzer.
testedbyFuzztruction.ThemeancoverageisshowninTa-
|     | G2FUZZ  |              |       |     | ble8.Overall,G2FUZZoutperformsFuzztructiononseven |     |     |     |     |     |     |
| --- | ------- | ------------ | ----- | --- | ------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|     | Program | FormatFuzzer | WEIZZ |     |                                                   |     |     |     |     |     |     |
outofnineprograms.G2FUZZsemanticallyconstructsfiles
|     | gdk | 126 0 | -   |     |     |     |     |     |     |     |     |
| --- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
exiv2 424 0 0 withdifferentstructuresfromscratch,whileFuzztruction’s
ffmpeg 1951 6 14 generator–primarilyaconverter–stillrequiresstructuredini-
|     | flvmeta | 0 - | 1   |     |     |     |     |     |     |     |     |
| --- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
imginfo 148 0 0 tialseeds,anditsbit-levelmutationonlymakessubtleadjust-
jhead 8 0 0 ments.However,Fuzztructionperformsbetteronprograms
mp3gain 1 - 0 usingzipfiles.WebelieveG2FUZZfindsfewerbugsdueto
|     | mp42aac | 338 - | 0   |     |     |     |     |     |     |     |     |
| --- | ------- | ----- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
pdftotext 334 - 0 thelimitedfunctionalityofPythonlibrariesforconstructing
tiffsplit 10 - 0 zipfiles,restrictingcoverageoffilecharacteristics.
WecomparetheseedqualityofG2FUZZandFuzztruction
AsG2FUZZ,FormatFuzzer,andWEIZZusedifferentin- bymeasuringfeaturecoverage.Inthisexperiment,G2FUZZ
strumentationmethods,theymayachievevaryingedgecover-
generatesseedsonlyduringtheinitialstage,whileFuzztruc-
agelevelswithidenticalinputs.Toaccuratelymeasureline tion continuously generates seeds. To ensure fairness, we
coverage,weutilizeafl-cov.TheresultsarepresentedinTa- comparethefeaturecoverageofG2FUZZandFuzztruction
ble 6. To clarify,we encounteredissues running some pro- usingthesamenumberofgeneratedseeds,withthenumber
gramswithFormatFuzzerandWEIZZ.Specifically,Format- ofseedsgeneratedbyG2FUZZservingasthebaseline.For
Fuzzergeneratedanexcessivenumberofcore.*fileswhile
|     |     |     |     |     | program | selection,we | target | all | programs | that | take image |
| --- | --- | --- | --- | --- | ------- | ------------ | ------ | --- | -------- | ---- | ---------- |
testingmp42aac,consumingover500GBofmemorywithin ordocumentinputs,includingpngtopng,pdftotext,andqpdf.
10 hours. Additionally,we face errors when building MP3 In Fuzztruction,PDFs generatedbythe qpdfgeneratorare
generatorsformp3gainaccordingtoFormatFuzzer’sinstruc-
excludedduetounparseablepasswordoptions.Theresults
tions. FormatFuzzerisalsounsuitablefortestingpdftotext, areshownintheTable9,revealingthatG2FUZZ discovers
tiffsplit,andflvmeta,asitdoesnotsupportPDF,TIFF,andFLV
|     |     |     |     |     | more | unique features | than | Fuzztruction. |     | In terms | of valid- |
| --- | --- | --- | --- | --- | ---- | --------------- | ---- | ------------- | --- | -------- | --------- |
formats.AsforWEIZZ,weareunabletocompilegdk-pixbuf. ityratio,G2FUZZachievesahighervalidityratioforPNGs
Fornineoutof10programs,G2FUZZachieveshigherline andPDFscomparedtoFuzztruction.Additionally,G2FUZZ
coveragethanbothFormatFuzzerandWEIZZ.Forexample, discoversmorerarefeatures,suchasProperties-DigitalSig-
G2FUZZ achieves more than twice as much line coverage natureandProperties-png:PLTE.number_colorsinPNGfiles,
asFormatFuzzerinexiv2,ffmpeg,imginfo,gdk. Unlikethe whichFuzztructioncannotcover.
grammar-basedfuzzerFormatFuzzer,G2FUZZisscalableto
|           |                   |             |           |          | Table | 8: The average | coverage |     | (in basic | blocks) | and bugs |
| --------- | ----------------- | ----------- | --------- | -------- | ----- | -------------- | -------- | --- | --------- | ------- | -------- |
| a broader | range of programs | that accept | different | formats. |       |                |          |     |           |         |          |
discoveredbyFuzztructionandG2FUZZ.
| Moreover, | it is common | for a program | to accept | multiple |     |     |     |     |     |     |     |
| --------- | ------------ | ------------- | --------- | -------- | --- | --- | --- | --- | --- | --- | --- |
inputformats,butFormatFuzzercanhandleonlyoneformat Fuzztruction G2FUZZ
|     |     |     |     |     |     | InputFormat | Program |     |     |     |     |
| --- | --- | --- | --- | --- | --- | ----------- | ------- | --- | --- | --- | --- |
atatime,whichcanreducethediversityofgeneratedinputs. Cov Bug Cov Bug
TofurthervalidatethatthefilesgeneratedbyG2FUZZwith pdf pdftotext-enc 36853.8 0 38866.0 0
complexfeatureshelpuncovermoreintricateprogramlogic, pdf pdftotext 35108.4 0 39011.4 0
|     |     |     |     |     |     | elf objdump |     | 12468.8 |     | 0 12851.8 | 0   |
| --- | --- | --- | --- | --- | --- | ----------- | --- | ------- | --- | --------- | --- |
wemeasurethenumberoffunctionsexclusivelydiscovered
|     |     |     |     |     |     | elf | readelf | 12347.8 |     | 0 13328.2 | 0   |
| --- | --- | --- | --- | --- | --- | --- | ------- | ------- | --- | --------- | --- |
byeachfuzzer.Here,“exclusive”referstofunctionsthatare
|              |              |           |             |            |     | png pngtopng |     | 4414.6  |     | 0 4566.2  | 0   |
| ------------ | ------------ | --------- | ----------- | ---------- | --- | ------------ | --- | ------- | --- | --------- | --- |
| not detected | by any other | fuzzer. A | fuzzer that | finds more |     |              |     |         |     |           |     |
|              |              |           |             |            |     | der vfychain |     | 14937.4 |     | 0 11600.4 | 0   |
exclusivefunctionsillustratesitsabilitytotriggermoresubtle 7z 7zip-enc 28887.2 8 28909.0 6
programlogics.TheresultsareshownintheTable7.Fornine zip 7zip 34585.4 8 31691.4 6
| outoftenprograms,G2FUZZ |     |     |     |     |     | zip | unzip | 2788.2 |     | 1 3104.8 | 1   |
| ----------------------- | --- | --- | --- | --- | --- | --- | ----- | ------ | --- | -------- | --- |
identifiesthelargestnumber
ofexclusivefunctions,confirmingtheeffectivenessofusing
LLMstogeneratecomplexbinaryinputs. Table9:Functionsexclusivelydiscoveredbyeachfuzzer.
|     |     |     |     |     |     | PNG(pngtopng) |                     |     |            | PDF(pdftotext)      |     |
| --- | --- | --- | --- | --- | --- | ------------- | ------------------- | --- | ---------- | ------------------- | --- |
|     |     |     |     |     |     | FeatureCov    | ValidNum/InvalidNum |     | FeatureCov | ValidNum/InvalidNum |     |
5.3 ComparedwithFuzztruction
|     |     |     |     |     | G2FUZZ       | 457.0 |          | 36/0 | 531.0 |     | 50/12     |
| --- | --- | --- | --- | --- | ------------ | ----- | -------- | ---- | ----- | --- | --------- |
|     |     |     |     |     | Fuzztruction | 427.4 | 27.8/8.2 |      | 152.4 |     | 48.8/13.2 |
G2FUZZ
| To compare  | with Fuzztruction, | we          | use         | to gener- |     |     |     |     |     |     |     |
| ----------- | ------------------ | ----------- | ----------- | --------- | --- | --- | --- | --- | --- | --- | --- |
| ate a batch | of initial seeds   | and conduct | experiments | in the    |     |     |     |     |     |     |     |
DockerenvironmentprovidedbyFuzztruction,ensuringthat
5.4 FeatureCoverage
allexperimentalparametersremainconsistent.Wetestnine
programsusedbyFuzztruction;however,threelackclearin- ToverifywhetherG2FUZZcangeneratefilefeaturesthatother
putfileextensionsandarethusincompatiblewithG2FUZZ.
fuzzerscannotcover,wecomparethefeaturecoverageofeach
Thetestsrun for6hoursandarerepeated5times,andthe fuzzer.Thecalculationoffeaturecoverageischallengingdue
versionsofalltestprogramsareconsistentwiththeversions tothelackofaunifiedquantificationmethod.Therefore,we

Table10:Thefeaturecoveragecoveredbyeachfuzzer. tionduringtheinitializationstage.ForGPT-3.5andGPT-4,
wereusetheinitialseedsgeneratedfromthefirstepochofex-
Program
G2 F U Z Z
A
F L + +(c
m
plog)
A
F L + +(
mo
pt)
A
F L +
+(fast)
A
F L +
+(rare)
w
p
on
e
i
r
t e h
im
x t i h
e
v
n
2 e
t
, t
s
t a i
f
r f
o
f g s
r
e p t
J
li
P
f t o ,
G
r m m
,
p
T
a 3
I
t g
F
s a
F
u i
,
f n
M
fi , x m
P
a p
3
r 4
,
e 2
M
c a o
P
a n c
4
, s
,
i a
a
d n
n
e d
d
re p
P
d d
D
, ft a
F
o s t
w
e t x h
i
t e
t
.
h
O g
G
e n n
2
ly e
F
r
U
fi at l
Z
o e
Z
s r
mayproducefilesinotherformats.
TIFF(tiffsplit) 4719 2303 2169 2369 2178
JPG(exiv2) 4096 1910 1910 1913 1909 The results are shown in the Table 11. GPT-4 achieves
MP4(mp42aac) 1459 0 0 0 0 the highest feature coverage for JPG and PDF, while the
PDF(pdftotext) 37797 33451 40730 44704 36203
open-source models llama-3-8b-instruct and llama-3-70b-
instruct achieve the highestfeature coverage forTIFF and
use ImageMagick to extract each seed’s attributes,such as
MP4,respectively.Notably,llama-3-70b-instructoutperforms
compressiontype,treatingeachattributeasafeature.Wethen
GPT-3.5acrossallfourformats.Theseresultsdemonstrate
manuallyremoveirrelevantattributesthatvaryacrossmost
G2FUZZ’sscalabilityanditsabilitytogeneratehigh-quality
files,suchasthefilename.
generatorsusingopen-sourcemodels.
Weselectfourformats–TIFF,JPG,MP4,andPDF–from
Wealsoevaluatetheeffectivenessofthepromptusedby
Sec.5.1.1foranalysis,coveringimagefiles,videofiles,and
G2FUZZwith10differentformats(includingimages,videos,
complexdocuments.TheresultsareshownintheTable10.
and documents) and find that GPT-4 performs well across
G2FUZZ (usingGPT-4)achievesthehighestfeaturecover-
mostformats.MoredetailscanbefoundinAppendixD.Ad-
ageforTIFF,JPG,andMP4.AFL++focusesmoreonlow-
ditionally,weanalyzetheimpactofdifferentlibrariesonthe
levelmutations,whichstruggletomodifyhigh-levelfeatures.
generatorqualityandfindthatcollaborationbetweenmulti-
Changinghigh-levelfeaturesrequireshandlingtheconstraints
plelibrariesisthemostefficientapproach.Formoredetails,
acrossmultiplechunkssimultaneously,whichishighlychal-
pleaserefertoAppendixE.
lenging for byte-level mutation. In contrast, G2FUZZ can
semanticallymutatethegeneratororgenerateseedswiththe
Table11:Functionsexclusivelydiscoveredbyeachfuzzer.
target features from scratch,enabling broader coverage of
high-levelfilefeatures.NotethatImageMagickcanonlyparse Format GPT-3.5 GPT-4 llama-3-8b-instruct llama-3-70b-instruct
validinputs,whilemostseedsgeneratedbyAFL++mutations JPG 259 984 211 636
areinvalid,resultinginlowerfeaturecoveragebeingrecorded MP4 -1 290 245 517
PDF 374 559 555 504
forAFL++.Forexample,themutationsofAFL++failtopro-
TIFF 388 387 591 516
ducevalidMP4files,resultinginafeaturecoverageof0.In
1-:GPT-3.5cannotgeneratevalidMP4filesduringthefirstrounddue
thePDFformat(pdftotext),AFL++(rare)andAFL++(fast)
torandomness.
covermorefeaturesintermsofskewness,kurtosis,andstan-
darddeviationintheBlue/Green/Redchannels.Seedswith
such features receive higher weights in AFL++ (rare) and
6 Discussion
AFL++(fast),leadingtomorefrequentmutationsand,con-
sequently,highercoverageofthesefeatures.However,from
G2FUZZ supportsonlyfileformatsthathaveaccompanied
theperspectiveoffinalcodecoverage,allocatingexcessive
generator libraries available. Nevertheless,it can integrate
energytoexploresuchfeaturesisinefficient.
withuser-writtenfileformatspecifications(byusingprompts
G2FUZZ can construct some rare features, such as
suchas“generatePythongeneratorcodebasedonthepro-
Properties-tiff:timestamp, Properties-tiff:copyright, and
videdformatspecifications.”).Thus,supportingcornercases
Properties-Contact in TIFF files. Furthermore, for
ornewfileformatsrequiresonlyextraengineeringandman-
Chromaticity-Compression, G2FUZZ can cover all four
ual effort. More importantly,we see an encouraging trend
compressionmethods–Zip,RLE,JPEG,andLZW–whereas
ofemerging Python libraries forfile generation (searching
otherfuzzerscanonlycoverRLEandJPEG.Weobservethat
forJPEGandMP4onGitHubreveals21and26filegener-
covering rare features can better trigger specific program
ation/editing libraries,respectively,created within the past
logicinthetargetprogram,therebyimprovingcodecoverage.
threeyears);thisillustratesthehighextensibilityofG2FUZZ
toadapttonewformatswithoutcodechanges.Overall,lever-
5.5 GeneralizabilityAcrossLLMs
agingexistingandemerginglibraries,G2FUZZcansupport
more file formats, thus improving fuzzing efficiency in a
Todemonstrate G2FUZZ’sgeneralizabilityacrossdifferent broaderrangeofscenariosandcontinuousmanner.Inaddi-
LLMs,weselecttheopen-sourcemodelsllama-3-8b-instruct tion,G2FUZZiscurrentlyunabletohandlecustomformats.
andllama-3-70b-instructforourexperiments.Underthesame Thislimitationcouldbealleviatedbyaddingdocumentpars-
setup,thesemodelsgenerateinitialseedsforfivefileformats, ingcapabilities,allowingLLMstolearnandadapttocustom
completingInputGeneratorSynthesisandGeneratorMuta- syntax;weleavethisasfuturework.

7 Conclusion temadministratorsorusers,thiscouldraiseethicalconcerns,
particularlyifitresultsinharmoralossoftrust.
Inthispaper,wepresentG2FUZZ,anovelandhighlyefficient We avoid using deception in our research. If deception
approachthataugmentsmutation-basedfuzzingwithLLMs. is necessary forthe validity ofthe study,itis ethically jus-
We identifya unique opportunityto combine the strengths tifiedandfollowedbyathoroughdebriefingtoexplainthe
ofLLMsandmutation-basedfuzzerstoachieveasynergis- research’spurposeandmethodstoallaffectedparties.This
tic effect. The evaluation shows that G2FUZZ consistently approachmaintainstransparencyandtrust.
outperformsSOTAmutation-basedfuzzersandseveralother WellbeingforTeamMembers.Ourworkmayexposeteam
fuzzerbaselines. memberstostressfulordisturbingcontent,especiallywhen
analyzingmalicioussoftware,whichcouldimpacttheirpsy-
chologicalwellbeing.
8 Acknowledgment
We prioritize our team’s wellbeing by supporting those
exposed to stressful content, setting clear boundaries, and
Wesincerelythanktheanonymousreviewersandourshep-
maintainingasafe,supportiveworkenvironment.
herdfortheirvaluablefeedbackandguidance.TheHKUST
InnovationswithBothPositiveandNegativePotentialOut-
authorsaresupportedinpartbyaRGCGRFgrantunderthe
comes.Thetoolsandtechniqueswedevelophavepotential
contract16214723.
formisusebyadversaries.Whileourintentionistoimprove
softwaresecurity,thereisariskthatotherscoulduseourtool
9 EthicsConsiderations tofindandexploitvulnerabilitiesformaliciouspurposes.
Werecognizethedual-usenatureofourtoolandhaveim-
Vulnerability Disclosure. Our fuzzing tool is designed to plementedsafeguardstopreventmisuse.Accessisrestricted,
uncoversecurity vulnerabilities in software. If we identify andweworkwithethicalreviewboardstoassessrisks.We
suchvulnerabilities,failingtodisclosethemresponsiblycould also engage with the security community to ensure ourre-
leadtoserioussecurityrisks,suchasunauthorizedaccessor searchisusedpositively.
exploitationbymaliciousactors. Retroactively Identifying Negative Outcomes. If our re-
We have established a responsible disclosure process. searchunintentionallycausesnegativeoutcomes,likeservice
Whenwefindvulnerabilities,wepromptlynotifytheaffected disruptionsorexploitationofvulnerabilities,failingtoaddress
software vendors,giving them sufficient time to patch the themcouldharmusersanddamageourcredibility.
issuesbeforemakinganypublicdisclosures.Thisensuresour Wewillmonitorforanyissuesandtakeresponsibilityif
researchcontributespositivelytosecuritywithoutexposing theyarise,workingtoremediateanyharm.Thisproactiveap-
userstounnecessaryrisks. proachensuresourresearchremainsethicalandresponsible.
ExperimentswithLiveSystemsWithoutInformedCon- TheLaw.Ourfuzzingactivitiesmustcomplywithcyberse-
sent.Ifweapplyourfuzzingtooltolivesystemsorreal-world curitylawsandregulations.Anyinadvertentviolationscould
softwarewithoutobtainingconsentfromtheownersoroper- leadtolegalconsequencesforusandourinstitution.
ators,thiscoulddisruptservicesornegativelyimpactusers Weconsultlegalexpertstoensurecomplianceandobtain
whorelyonthosesystems. necessaryapprovalsbeforeengaginginriskyactivities,mini-
Weavoidtestinglivesystemswithoutexplicitconsentfrom mizinglegalrisksandensuringproperconduct.
the system owners. When testing on live systems is neces-
sary,wefirstobtaininformedconsentanddesignourtesting
10 OpenScience
methodstominimizeanypotentialharmordisruption.This
approachrespectstherightsandinterestsofthosewhorely
Toensurecompliancewithopenscienceprinciples,wecom-
onthesystemswestudy.
mit to making our research data, code, and materials pub-
TermsofService.Ourtoolcouldpotentiallyviolatetheterms
liclyaccessiblethroughpubliclyavailablerepositories.This
ofserviceofthesoftwarewearetesting,particularlyifthe
includes providing access to ourtoolcode andexperiment
software explicitly prohibits automated testing or fuzzing.
dataathttps://github.com/G2FUZZ/G2FUZZ4andhttps:
Thiscouldleadtolegalissuesorharmourreputationinthe
//github.com/G2FUZZ/G2FUZZ-DATA, allowing others to
researchcommunity.
review,utilize,andadaptourimplementation.
Beforeconductinganyfuzzing,wethoroughlyreviewthe
Wealsodocumentourresearchmethods,experiments,and
termsofserviceofthesoftwarebeingtested.Ifouractivities
resultsindetailtoenablereproducibility.Allrelevantinfor-
mightviolatetheseterms,weseekpermissionfromthesoft-
mation willbe sharedopenly to allow otherresearchers to
wareprovideroradjustourmethodstoavoidviolations.This
replicateandbuilduponourwork.
ensuresourresearchisbothethicalandlegallycompliant.
Deception. If our testing involves any form of deception, 4Ourtool code is also available at https://zenodo.org/records/
suchas obscuring the true nature ofthe tests from the sys- 14728879.

References [16] AndreaFioraldi,DominikMaier,HeikoEißfeldt,and
|                |                                     |     |                            |     |     |     | MarcHeuse.         |     | {AFL++}:Combiningincrementalsteps |     |     |     |     |
| -------------- | ----------------------------------- | --- | -------------------------- | --- | --- | --- | ------------------ | --- | --------------------------------- | --- | --- | --- | --- |
| [1] Dall-e-3.  | https://openai.com/index/dall-e-3/. |     |                            |     |     |     |                    |     |                                   |     |     |     |     |
|                |                                     |     |                            |     |     |     | offuzzingresearch. |     | InWOOT,2020.                      |     |     |     |     |
| [2] honggfuzz. |                                     |     | https://github.com/google/ |     |     |     |                    |     |                                   |     |     |     |     |
[17] ShuitaoGan,ChaoZhang,PengChen,BodongZhao,Xi-
honggfuzz.
aojunQin,DongWu,andZuoningChen.{GREYONE}:
[3] BaleeghAhmad,ShailjaThakur,BenjaminTan,Ramesh
|                                          |     |     |     |                       |     |       | Dataflowsensitivefuzzing. |                                       |       | InUSENIXSecurity,2020. |     |            |     |
| ---------------------------------------- | --- | --- | --- | --------------------- | --- | ----- | ------------------------- | ------------------------------------- | ----- | ---------------------- | --- | ---------- | --- |
| Karri,andHammondPearce.                  |     |     |     | Onhardwaresecuritybug |     |       |                           |                                       |       |                        |     |            |     |
|                                          |     |     |     |                       |     |       | [18] Hanan                | Gani,ShariqFarooqBhat,MuzammalNaseer, |       |                        |     |            |     |
| codefixesbypromptinglargelanguagemodels. |     |     |     |                       |     | TIFS, |                           |                                       |       |                        |     |            |     |
|                                          |     |     |     |                       |     |       | Salman                    | Khan,and                              | Peter | Wonka.                 | Llm | blueprint: | En- |
2024.
ablingtext-to-imagegenerationwithcomplexandde-
| [4] Maria                | Alabdulrahman, |     | Renad | Khayyat,                | Kawthar | Al- |                |     |             |     |     |     |     |
| ------------------------ | -------------- | --- | ----- | ----------------------- | ------- | --- | -------------- | --- | ----------- | --- | --- | --- | --- |
|                          |                |     |       |                         |         |     | tailedprompts. |     | arXiv,2023. |     |     |     |     |
| mowallad,andZahraAlharz. |                |     |       | Sarid:Arabicstoryteller |         |     |                |     |             |     |     |     |     |
[19] PatriceGodefroid,AdamKiezun,andMichaelY.Levin.
| usingafine-tunedllmandtext-to-imagegeneration. |     |     |     |     |     | In  |     |     |     |     |     |     |     |
| ---------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
InProceedingsof
| ICCAE,2024. |     |     |     |     |     |     | Grammar-basedwhiteboxfuzzing. |     |     |     |     |     |     |
| ----------- | --- | --- | --- | --- | --- | --- | ----------------------------- | --- | --- | --- | --- | --- | --- |
the29thACMSIGPLANConferenceonProgramming
[5] CorneliusAschermann,TommasoFrassetto,Thorsten
LanguageDesignandImplementation,PLDI’08,pages
| Holz, | Patrick | Jauernig, |     | Ahmad-Reza | Sadeghi, | and |     |     |     |     |     |     |     |
| ----- | ------- | --------- | --- | ---------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
DanielTeuchert. Nautilus:Fishingfordeepbugswith 206–215.ACM,2008.
grammars. InNDSS,2019. [20] Daya Guo,Canwen Xu,Nan Duan,Jian Yin,and Ju-
[6] CorneliusAschermann,SergejSchumilo,TimBlazytko, lian McAuley. Longcoder: A long-range pre-trained
RobertGawlik,andThorstenHolz. Redqueen:Fuzzing languagemodelforcodecompletion. InICML,2023.
| withinput-to-statecorrespondence. |     |     |     |     | InNDSS,2019. |     |          |            |        |     |       |           |      |
| --------------------------------- | --- | --- | --- | --- | ------------ | --- | -------- | ---------- | ------ | --- | ----- | --------- | ---- |
|                                   |     |     |     |     |              |     | [21] Tao | Guo, Puhan | Zhang, | Xin | Wang, | and Qiang | Wei. |
[7] NilsBars,MoritzSchloegel,TobiasScharnowski,Nico Gramfuzz:Fuzzingtestingofwebbrowsersbasedon
| Schiller,andThorstenHolz. |     |                        |     | Fuzztruction:Usingfault |          |        |         |          |     |            |           |     |          |
| ------------------------- | --- | ---------------------- | --- | ----------------------- | -------- | ------ | ------- | -------- | --- | ---------- | --------- | --- | -------- |
|                           |     |                        |     |                         |          |        | grammar | analysis | and | structural | mutation. |     | In ICIA, |
| injection-based           |     | fuzzing                | to  | leverage                | implicit | domain | 2013.   |          |     |            |           |     |          |
| knowledge.                |     | InUSENIXSecurity,2023. |     |                         |          |        |         |          |     |            |           |     |          |
[22] AhmadHazimeh,AdrianHerrera,andMathiasPayer.
| [8] Tim | Blazytko, | Matt | Bishop, | Cornelius | Aschermann, |     |                                      |     |     |     |     |     |         |
| ------- | --------- | ---- | ------- | --------- | ----------- | --- | ------------------------------------ | --- | --- | --- | --- | --- | ------- |
|         |           |      |         |           |             |     | Magma:Aground-truthfuzzingbenchmark. |     |     |     |     |     | POMACS, |
JustinCappos,MoritzSchlögel,NadiaKorshun,AliAb-
2020.
basi,MarcoSchweighauser,SebastianSchinzel,Sergej
|                |     |                                  |     |     |     |     | [23] Renáta | Hodován, | Ákos | Kiss, | and | Tibor | Gyimóthy. |
| -------------- | --- | -------------------------------- | --- | --- | --- | --- | ----------- | -------- | ---- | ----- | --- | ----- | --------- |
| Schumilo,etal. |     | {GRIMOIRE}:Synthesizingstructure |     |     |     |     |             |          |      |       |     |       |           |
whilefuzzing. InUSENIXSecurity,2019. Grammarinator:agrammar-basedopensourcefuzzer.
InA-TEST,2018.
| [9] Yinlin | Deng, | Chunqiu |     | Steven | Xia, Haoran | Peng, |     |     |     |     |     |     |     |
| ---------- | ----- | ------- | --- | ------ | ----------- | ----- | --- | --- | --- | --- | --- | --- | --- |
ChenyuanYang,andLingmingZhang. Largelanguage [24] HuiHuang,ShuangzhiWu,XinnianLiang,BingWang,
models are zero-shot fuzzers: Fuzzing deep-learning YanruiShi,PeihaoWu,MuyunYang,andTiejunZhao.
librariesvialargelanguagemodels. InISSTA,2023. Towardsmakingthemostofllmfortranslationquality
|             |       |         |         |      |           |          | estimation. |          | InNLPCC,2023. |       |      |          |      |
| ----------- | ----- | ------- | ------- | ---- | --------- | -------- | ----------- | -------- | ------------- | ----- | ---- | -------- | ---- |
| [10] Yinlin | Deng, | Chunqiu | Steven  | Xia, | Chenyuan  | Yang,    |             |          |               |       |      |          |      |
| Shizhuo     | Dylan | Zhang,  | Shujing |      | Yang, and | Lingming |             |          |               |       |      |          |      |
|             |       |         |         |      |           |          | [25] Nikhil | Kandpal, | Haikang       | Deng, | Adam | Roberts, | Eric |
Zhang. Largelanguagemodelsareedge-casegenera- Wallace,andColinRaffel.Largelanguagemodelsstrug-
tors:Craftingunusualprogramsforfuzzingdeeplearn- gletolearnlong-tailknowledge. InICML,2023.
| inglibraries. |     | InICSE,2024. |     |     |     |     |               |      |           |     |      |            |       |
| ------------- | --- | ------------ | --- | --- | --- | --- | ------------- | ---- | --------- | --- | ---- | ---------- | ----- |
|               |     |              |     |     |     |     | [26] Jaehyung | Kim, | Dongyoung |     | Kim, | and Yiming | Yang. |
[11] TuanDinh,JinmanZhao,SamsonTan,RenatoNegrinho,
|     |     |     |     |     |     |     | Learning | to  | correct for | qa reasoning |     | with | black-box |
| --- | --- | --- | --- | --- | --- | --- | -------- | --- | ----------- | ------------ | --- | ---- | --------- |
LeonardLausen,ShengZha,andGeorgeKarypis.Large
|     |     |     |     |     |     |     | llms. | arXiv,2024. |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | ----- | ----------- | --- | --- | --- | --- | --- |
languagemodelsofcodefailatcompletingcodewith
potentialbugs. NeurIPS,2024. [27] Xuan-Bach D Le, Corina Pasareanu, Rohan Padhye,
|              |               |     |     |         |            |          | David | Lo,Willem | Visser,and |     | Koushik | Sen. | Saffron: |
| ------------ | ------------- | --- | --- | ------- | ---------- | -------- | ----- | --------- | ---------- | --- | ------- | ---- | -------- |
| [12] Brendan | Dolan-Gavitt. |     |     | Is “ai” | useful for | fuzzing? |       |           |            |     |         |      |          |
Adaptivegrammar-basedfuzzingforworst-caseanaly-
| (keynote). |     | InFUZZINGWorkshop,2024. |     |     |     |     |     |     |     |     |     |     |     |
| ---------- | --- | ----------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
sis. SEN,2021.
| [13] RafaelDutra,RahulGopinath,andAndreasZeller. |     |     |     |     |     | For- |     |     |     |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- | --- | --- |
matfuzzer: Effective fuzzing of binary file formats. [28] Caroline Lemieux and Koushik Sen. Fairfuzz: A tar-
getedmutationstrategyforincreasinggreyboxfuzztest-
TOSEM,2023.
[14] MartinEberlein,YannicNoller,ThomasVogel,andLars ingcoverage. InASE,2018.
Grunske. Evolutionary grammar-based fuzzing. In [29] Manling Li, Ruochen Xu, Shuohang Wang, Luowei
| SSBSE,2020. |     |     |     |     |     |     | Zhou, | Xudong | Lin, | Chenguang | Zhu, | Michael | Zeng, |
| ----------- | --- | --- | --- | --- | --- | --- | ----- | ------ | ---- | --------- | ---- | ------- | ----- |
[15] Andrea Fioraldi, Daniele Cono D’Elia, and Emilio HengJi,andShih-FuChang. Clip-event:Connecting
Coppa. Weizz:Automaticgrey-boxfuzzingforstruc- textandimageswitheventstructures. InCVPR,pages
| turedbinaryformats. |     |     | InISSTA,2020. |     |     |     | 16420–16429,2022. |     |     |     |     |     |     |
| ------------------- | --- | --- | ------------- | --- | --- | --- | ----------------- | --- | --- | --- | --- | --- | --- |

[30] YuweiLi,ShoulingJi,YuanChen,SizhuangLiang,Wei- [44] Aditya Ramesh, Mikhail Pavlov, Gabriel Goh, Scott
HanLee,YueyaoChen,ChenyangLyu,ChunmingWu, Gray,ChelseaVoss,AlecRadford,MarkChen,andIlya
Raheem Beyah,Peng Cheng,et al. {UNIFUZZ}: A Sutskever.Zero-shottext-to-imagegeneration.InICML,
| holisticandpragmatic{Metrics-Driven}platformfor |                        |     |     | 2021.      |          |                    |               |
| ----------------------------------------------- | ---------------------- | --- | --- | ---------- | -------- | ------------------ | ------------- |
| evaluatingfuzzers.                              | InUSENIXSecurity,2021. |     |     |            |          |                    |               |
|                                                 |                        |     |     | [45] Aryan | Rangapur | and Aman Rangapur. | The battle of |
[31] FangLiu,GeLi,YunfeiZhao,andZhiJin. Multi-task llms: A comparative study in conversationalqa tasks.
| learningbasedpre-trainedlanguagemodelforcodecom- |             |     |     | arXiv,2024. |     |     |     |
| ------------------------------------------------ | ----------- | --- | --- | ----------- | --- | --- | --- |
| pletion.                                         | InASE,2020. |     |     |             |     |     |     |
[46] KuniakiSaito,KihyukSohn,Chen-YuLee,andYoshi-
[32] XuweiLiu,WeiYou,YapengYe,ZhuoZhang,Jianjun takaUshiku. Unsupervisedllmadaptationforquestion
arXiv,2024.
| Huang,andXiangyuZhang. |     | Fuzzinmem:Fuzzingpro- |     | answering. |     |     |     |
| ---------------------- | --- | --------------------- | --- | ---------- | --- | --- | --- |
gramsviain-memorystructures. InICSE,2024. [47] Sevak Sargsyan, Shamil Kurmangaleev, Matevos
[33] YuweiLiu,SiqiChen,YuchongXie,YanhaoWang,Libo Mehrabyan,MaksimMishechkin,TsolakGhukasyan,
Chen,BinWang,YingmingZeng,ZhiXue,andPuruiSu. and Sergey Asryan. Grammar-based fuzzing. In
| Vd-guard: | Dmaguidedfuzzingforhypervisorvirtual |     |     | IVMEM,2018. |     |     |     |
| --------- | ------------------------------------ | --- | --- | ----------- | --- | --- | --- |
| device.   | InASE,2023.                          |     |     |             |     |     |     |
[48] DongdongShe,AdamStorek,YuchongXie,Seoyoung
[34] YujieLu,XianjunYang,XiujunLi,XinEricWang,and Kweon, Prashast Srivastava, and Suman Jana. Fox:
Coverage-guidedfuzzingasonlinestochasticcontrol.
| WilliamYangWang.                                  |               | Llmscore:Unveilingthepowerof |     |                                         |     |     |               |
| ------------------------------------------------- | ------------- | ---------------------------- | --- | --------------------------------------- | --- | --- | ------------- |
| largelanguagemodelsintext-to-imagesynthesisevalu- |               |                              |     | InCCS,2024.                             |     |     |               |
| ation.                                            | NeurIPS,2024. |                              |     |                                         |     |     |               |
|                                                   |               |                              |     | [49] PrashastSrivastavaandMathiasPayer. |     |     | Gramatron:Ef- |
[35] ChenyangLyu,ShoulingJi,ChaoZhang,YuweiLi,Wei- fectivegrammar-awarefuzzing. InISSTA,2021.
HanLee,YuSong,andRaheemBeyah. {MOPT}:Op- [50] LiyanTang,ZhaoyiSun,BetinaIdnay,JordanGNestor,
timizedmutation scheduling forfuzzers. In USENIX AliSoroush,PierreAElias,ZiyangXu,YingDing,Greg
Security,2019.
|     |     |     |     | Durrett,JustinFRousseau,etal. |     | Evaluatinglargelan- |     |
| --- | --- | --- | --- | ----------------------------- | --- | ------------------- | --- |
guagemodelsonmedicalevidencesummarization. npj
[36] ChenyangLyu,ShoulingJi,XuhongZhang,HongLiang,
Binbin Zhao,Kangjie Lu,and Raheem Beyah. Ems: DigitalMedicine,2023.
| History-drivenmutationforcoverage-basedfuzzing. |     |     | In  |     |     |     |     |
| ----------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
[51] JunjieWang,BihuanChen,LeiWei,andYangLiu.Supe-
| NDSS,2022. |     |     |     | rion:Grammar-awaregreyboxfuzzing. |     |     | InICSE,2019. |
| ---------- | --- | --- | --- | --------------------------------- | --- | --- | ------------ |
[37] RuijieMeng,MartinMirchev,MarcelBöhme,andAb- [52] YizhongWang,YeganehKordi,SwaroopMishra,Alisa
hikRoychoudhury. Largelanguagemodelguidedproto- Liu,Noah A Smith,Daniel Khashabi,and Hannaneh
InNDSS,2024.
colfuzzing. Hajishirzi. Self-instruct:Aligninglanguagemodelwith
|               |          |                  |                | selfgeneratedinstructions. |     | arXiv,2022. |     |
| ------------- | -------- | ---------------- | -------------- | -------------------------- | --- | ----------- | --- |
| [38] Jonathan | Metzman, | László Szekeres, | Laurent Simon, |                            |     |             |     |
Read Sprabery, and Abhishek Arya. Fuzzbench: an [53] Chunqiu Steven Xia, Matteo Paltenghi, Jia Le Tian,
| open | fuzzer benchmarking | platform | and service. In |                                 |     |     |               |
| ---- | ------------------- | -------- | --------------- | ------------------------------- | --- | --- | ------------- |
|      |                     |          |                 | MichaelPradel,andLingmingZhang. |     |     | Fuzz4all:Uni- |
FSE,2021. versal fuzzing with large language models. In ICSE,
2024.
| [39] Soyeon | Park, Wen | Xu, Insu Yun, Daehee | Jang, and |     |     |     |     |
| ----------- | --------- | -------------------- | --------- | --- | --- | --- | --- |
Taesoo Kim. Fuzzing javascript engines with aspect- [54] FrankFXu,UriAlon,GrahamNeubig,andVincentJo-
| preservingmutation. |     | InSP,2020. |     |                 |     |                                  |     |
| ------------------- | --- | ---------- | --- | --------------- | --- | -------------------------------- | --- |
|                     |     |            |     | suaHellendoorn. |     | Asystematicevaluationoflargelan- |     |
[40] Hammond Pearce, Benjamin Tan, Baleegh Ahmad, guagemodelsofcode. InMAPS,2022.
RameshKarri,andBrendanDolan-Gavitt. Examining [55] WeiYou,XueqiangWang,ShiqingMa,JianjunHuang,
zero-shotvulnerabilityrepairwithlargelanguagemod- XiangyuZhang,XiaoFengWang,andBinLiang. Pro-
els. InSP,2023. fuzzer:On-the-flyinputtypeprobingforbetterzero-day
|     |     |     |     | vulnerabilitydiscovery. |     | InSP,2019. |     |
| --- | --- | --- | --- | ----------------------- | --- | ---------- | --- |
[41] Van-ThuanPham,MarcelBöhme,AndrewESantosa,
Alexandru Ra˘zvan Ca˘ciulescu,and Abhik Roychoud- [56] TaiYue,PengfeiWang,YongTang,EnzeWang,BoYu,
| hury. | Smartgreyboxfuzzing. | TSE,2019. |     |                  |     |                            |     |
| ----- | -------------------- | --------- | --- | ---------------- | --- | -------------------------- | --- |
|       |                      |           |     | KaiLu,andXuZhou. |     | {EcoFuzz}:Adaptive{Energy- |     |
[42] JonathanPilault,RaymondLi,SandeepSubramanian, Saving}greyboxfuzzingasavariantoftheadversarial
|                    |     |                                |     | {Multi-Armed}bandit. |     | InUSENIXSecurity,2020. |     |
| ------------------ | --- | ------------------------------ | --- | -------------------- | --- | ---------------------- | --- |
| andChristopherPal. |     | Onextractiveandabstractiveneu- |     |                      |     |                        |     |
raldocumentsummarizationwithtransformerlanguage [57] MichalZalewski. Americanfuzzylop,2017.
models. InEMNLP,2020. [58] Tianyi Zhang, Faisal Ladhak, Esin Durmus, Percy
[43] Leigang Qu, Haochuan Li, Tan Wang, Wenjie Wang, Liang,KathleenMcKeown,andTatsunoriBHashimoto.
Yongqi Li,Liqiang Nie,andTat-Seng Chua. Unified Benchmarkinglargelanguagemodelsfornewssumma-
text-to-imagegenerationandretrieval. arXiv,2024. rization. TACL,2024.

A ChallengesinGeneratingComplexFiles:A Based on our exploration,current binary-format fuzzers
TIFFCaseStudy cannotgenerateTIFFfilescontainingLZWdata.Thesemeth-
|     |     |     |     | ods can be categorized | into | two types: 1. inference-based |     |
| --- | --- | --- | --- | ---------------------- | ---- | ----------------------------- | --- |
fuzzing,suchasWEIZZ.WEIZZinfersinputfieldsandanap-
proximatestructureofthechunkson-the-flywhilemutating.
Theinferenceresultscanbeinaccurate,failingtoprecisely
capturerelationsbetweenchunksandmakingitunsuitablefor
constructingfileswithcomplexfeatures.2.grammar-aware
fuzzing,suchasFormatFuzzerandAFLSmart.Theyrelyon
user-providedgrammarstoparseandmutateinputs.However,
(a)OriginalTIFFfile.
thestandardTIFFspecificationcanbeinsufficientfrequently,
|     |     |     |     | and specifications | of other | formats are needed. | In particu- |
| --- | --- | --- | --- | ------------------ | -------- | ------------------- | ----------- |
lar,duetotheabsenceofLZWsyntaxinthegrammarfiles
shippedbyFormatFuzzerandAFLSmart,theycannotgener-
ateTIFFfilesthatincludeLZWdata.Thus,evenifaninitial
TIFFseedcontainscompressiondata,existingmethodsstill
cannotparseandmutateit.
Overall,complexfeaturesareverycommonacrossvarious
fileformatsindifferentdomains,suchasthecomplexExif
(b)TIFFfilewithLZWcompressionenabled.
datainJPEGfiles,transparencycapabilitiesinPNGs,anden-
Figure10:ComparingtwoTIFFfileswithLZWcompression
enabledornot,bothcontaininganidenticalimagedata.The cryptionandDRMprotectioninMP4files.It’sworthnoting
thatthesecomplexfeaturesofteninvolvemoreintricatelogic
newlyaddedLZW-relatedchunksinFig.10bfrom(0x000,
|     |     |     |     | and state management,which |     | may likely result | in security |
| --- | --- | --- | --- | -------------------------- | --- | ----------------- | ----------- |
0x08)to(0x100,0x1D)cannotbeparsedwithoutspecifica-
vulnerabilities.Therefore,constructingtestinputfileswith
tions.
variouscomplexfeaturesiscrucialforenhancingfuzzing.
| 1   | from PIL import Image                |     |     |     |     |     |     |
| --- | ------------------------------------ | --- | --- | --- | --- | --- | --- |
| 2   | image = Image.new('RGB', (100, 100)) |     |     |     |     |     |     |
Table12:Tokenconsumptionandcostanalysisfor24hours
| 3   | image.save('./tmp/tiff.tiff’,  |     |     |                     |     |     |     |
| --- | ------------------------------ | --- | --- | ------------------- | --- | --- | --- |
|     |   compression='tiff_lzw')      |     |     | offuzzinginUNIFUZZ. |     |     |     |
Figure11:PythongeneratorforcreatingtheTIFFfilewith
|                              |     |     |     |          | G2FUZZ(GPT-3.5) | G2FUZZ(GPT-4)      |         |
| ---------------------------- | --- | --- | --- | -------- | --------------- | ------------------ | ------- |
| LZWcompressiondatainFig.10b. |     |     |     | Programs |                 |                    |         |
|                              |     |     |     |          | TokenCount      | Cost($) TokenCount | Cost($) |
InSec.3,wearguethatgeneratingfileswithcomplexfea-
|     |     |     |     | ffmpeg | 57,870.0 | 0.10 112,763.8 | 3.97 |
| --- | --- | --- | --- | ------ | -------- | -------------- | ---- |
turesischallengingforcurrentfuzzers.Toillustratethis,we gdk 80,720.4 0.14 186,810.8 6.50
|                   |                                |     |     | jhead   | 68,816.2 | 0.12 158,385.6 | 5.68 |
| ----------------- | ------------------------------ | --- | --- | ------- | -------- | -------------- | ---- |
| provideanexample. | TIFF,whichstandsforTaggedImage |     |     |         |          |                |      |
|                   |                                |     |     | mp42aac | 64,593.6 | 0.12 170,129.4 | 6.03 |
FileFormat,isaflexibleandadaptablefileformatforstoring
|     |     |     |     | tiffsplit | 75,616.6 | 0.14 156,247.8 | 5.58 |
| --- | --- | --- | --- | --------- | -------- | -------------- | ---- |
images.NotethatTIFFfilessupportvariouscompressional-
|     |     |     |     | exiv2   | 41,958.4 | 0.08 100,312.0 | 3.58  |
| --- | --- | --- | --- | ------- | -------- | -------------- | ----- |
|     |     |     |     | flvmeta | 52,277.4 | 0.09 350,607.6 | 12.71 |
gorithms.Here,weanalyzetheuseofLZWcompresseddata
|     |     |     |     | imginfo | 66,237.4 | 0.12 138,265.4 | 4.78 |
| --- | --- | --- | --- | ------- | -------- | -------------- | ---- |
withinTIFFfilestoclarifywhygeneratingfileswithcomplex
|     |     |     |     | mp3gain | 108,270.2 | 0.20 152,863.2 | 5.50 |
| --- | --- | --- | --- | ------- | --------- | -------------- | ---- |
features is hard for existing fuzzers. Fig. 10 illustrates the pdftotext 69,787.6 0.13 131,047.2 4.64
differencesbetweentwoTIFFfileswithanidenticalimage
data:Fig.10ashowstheoriginalTIFFfile,whereasFig.10b
| shows the | file with LZW compression | enabled. Two | main |     |     |     |     |
| --------- | ------------------------- | ------------ | ---- | --- | --- | --- | --- |
differencesexist:1.introducingmany(unparsed)datablocks.
InFig.10b,alargedatablockisintroduced.Notethatitis B TokenandCostAnalysis
“unparsed”becausetheLZWspecificationismissinginthe
010EditortemplateusedbyFormatFuzzer,whichprevents
parsingandfurthermutating.2.changesindatavaluesand WefurtherevaluatethetokencostofLLMsforfuzzing.Over-
newconstraints:ManydatavaluesinFig.10bhavechanged, all,aswedonotrelyonLLMstoperformmutationengines,
withthesechangesintroducednewconstraints(e.g.,offsets G2FUZZdoesnotneedtoomanytokens.Wecollecttheto-
andsizes)thatneedtobemet.Forexample,whenaddingExif kenconsumptionofGPT-3.5andGPT-4intheexperiments
featurestoaTIFFfile,anExifIFDPointertagisaddedtothe ofUNIFUZZ.TheresultsareshowninTable12.Inallpro-
grams,G2FUZZ(GPT-3.5)costslessthan0.2$fora24hour
primaryIFDtorefertheExifdata.TheExifdata,likecolor
space,mustbealignedwiththoseinTIFFdata,introducing fuzzingprocess,whileG2FUZZ(GPT-4)costslessthan13$.
newconstraintsbetweentheExifdataandtheprimaryIFD. Weinterpretthatthetokencostisacceptableforfuzzing.

Table14:TheevaluationofG2FUZZ(LLM-Only).
C AblationStudy
| C.1 Thecontributionofeachcomponent |     |     |     |     |          | G2FUZZ |            |            |     |         |
| ---------------------------------- | --- | --- | --- | --- | -------- | ------ | ---------- | ---------- | --- | ------- |
|                                    |     |     |     |     | Programs |        | Throughput | TokenCount |     | Cost($) |
(LLM-Only)
AsG2FUZZcomprisestwomaincomponents:inputgenerator
|     |     |     |     |     | flvmeta | 150 | 5,758,008 | 16,253,249 |     | 15.72 |
| --- | --- | --- | --- | --- | ------- | --- | --------- | ---------- | --- | ----- |
synthesisandgeneratormutation,weanalyzethecontribution exiv2 2,126 7,643 20,831,178 17.80
|                  |           |           |                   |     | gdk     | 830 | 18,007 | 18,253,243 |     | 16.63 |
| ---------------- | --------- | --------- | ----------------- | --- | ------- | --- | ------ | ---------- | --- | ----- |
| ofeachcomponent. | Ourgoalis | to assess | the effectiveness |     |         |     |        |            |     |       |
|                  |           |           |                   |     | imginfo | 909 | 13,155 | 17,353,735 |     | 16.10 |
ofthe seeds generatedby these components. Ifmutating a jhead 245 150,634 18,523,648 17.00
seed results in the discovery of a new path,we considerit mp42aac 728 11,645 16,349,895 15.13
|     |     |     |     |     | tiffsplit | 938 | 6,729 | 20,539,769 |     | 17.75 |
| --- | --- | --- | --- | --- | --------- | --- | ----- | ---------- | --- | ----- |
useful.Therefore,wecountthenumberofnewpathsfound
|     |     |     |     |     | mp3gain | 691 | 9,337 | 18,547,650 |     | 16.31 |
| --- | --- | --- | --- | --- | ------- | --- | ----- | ---------- | --- | ----- |
bymutatingseedsfromeachcomponent.Thecomponentthat pdftotext 3,300 3,084 22,996,919 19.20
contributesmorenewpathsisdeemedmoreeffective.
The results are presented in Table 13. On average,both G2FUZZshouldbeabletoconstructvalidseeds.2)Proportion
theinputgeneratorsynthesisandthegeneratormutationhave
ofSeedswiththeTargetFeature(PSTF):Seedsthatcontain
proventobeeffective.Intotal,theinputgeneratorsynthesis
thenecessarycodetoproducethetargetfeaturearedeemedto
contributes82,001newpaths,whilethegeneratormutation possessit.3)ProportionofUnique,UsefulFeatures(PUUF).
contributes141,340.Specifically,injhead,theinputgenerator
WeanalyzeallgeneratorsfromSec.5.1.1forvalidityand
synthesis is responsible fordiscovering almost all the new manuallyreviewthegeneratorsproducedduringInputGen-
paths.Intiffsplit,ffmpeg,exiv2,andmp3gain,thegenerator
eratorSynthesisfortheothertwoattributes.Specifically,we
mutationcontributesthemosttodiscoveringnewpaths.
excludefileswhosesuffixmatchesthetargetformatanduse
ImageMagickforanalysis.NotethatImageMagickcanpro-
Table13:Thenumberofnewpathescontributedbythedif-
|     |     |     |     |     | cess various | image formats | as well | as PDF | and | MP4 (see |
| --- | --- | --- | --- | --- | ------------ | ------------- | ------- | ------ | --- | -------- |
ferentcomponentsofG2FUZZ.
|     |     |     |     |     | Table15). | Forexample,forTIFF,weexcludeseedswitha |     |     |     |     |
| --- | --- | --- | --- | --- | --------- | -------------------------------------- | --- | --- | --- | --- |
InputGenerator TIFFsuffixfromallprograms,thenparseeachonewithIm-
| Programs InitialSeeds |     |           | GeneratorMutation |     |                                                  |     |     |     |     |     |
| --------------------- | --- | --------- | ----------------- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- |
|                       |     | Synthesis |                   |     | ageMagick.Seedsthatcanbeparsedaredeemedvalid,and |     |     |     |     |     |
viceversa.TheresultsareshowninTable15.GPT-4achieves
| tiffsplit | 101 | 539   | 2,549 |     |     |     |     |     |     |     |
| --------- | --- | ----- | ----- | --- | --- | --- | --- | --- | --- | --- |
| jhead     | 2   | 1,046 | 0     |     |     |     |     |     |     |     |
avalidityrateexceeding80%acrossall10formats,withPSTF
| mp42aac | 6,859 | 522 | 6,298 |     |     |     |     |     |     |     |
| ------- | ----- | --- | ----- | --- | --- | --- | --- | --- | --- | --- |
over70%in5formatsandPUUFabove70%in8formats.
| gdk | 9,832 | 12,993 | 1   |     |     |     |     |     |     |     |
| --- | ----- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
G2FUZZ’s
ffmpeg 4,877 14,603 109,381 These findings demonstrate the effectiveness of
| exiv2 | 398 | 40,719 | 18,328 |     |     |     |     |     |     |     |
| ----- | --- | ------ | ------ | --- | --- | --- | --- | --- | --- | --- |
promptsinefficientlyaccomplishingthetargettasks.
| flvmeta | 1,133 | 1,066 | 29  |     |     |     |     |     |     |     |
| ------- | ----- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
Theunsuccessfuloutcomescanbeattributedtofourrea-
| imginfo | 21,236 | 593 | 0   |     |     |     |     |     |     |     |
| ------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
mp3gain 1,675 1,377 4,266 sons:1)LLMhallucinationsgeneratenon-existentfeatures.
| pdftotext | 7,735 | 8,543 | 488 |     |     |     |     |     |     |     |
| --------- | ----- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
2)Debugging(Alg.2)leadstheLLMtoremovecoderelated
| Total | 53,848 | 82,001 | 141,340 |     |               |             |                   |     |         |          |
| ----- | ------ | ------ | ------- | --- | ------------- | ----------- | ----------------- | --- | ------- | -------- |
|       |        |        |         |     | to the target | feature for | proper execution. |     | 3) Rare | features |
arehardertogenerate.4)Somefeaturesexistinallfilesof
agiventype,renderingthemuseless.Wefurtheranalyzethe
| C.2 ComparedwithLLM-Only |     |     | G2FUZZ |     |     |     |     |     |     |     |
| ------------------------ | --- | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
impactofLLMhallucinationsonG2FUZZ.Duringthefeature
InG2FUZZ,weleverageLLMsforgeneratingdiverseseeds generationphase,hallucinationsarerelativelyrare,withmost
|     |     |     |     |     | useless features | like | “defaultfeatures | describing |     | the target |
| --- | --- | --- | --- | --- | ---------------- | ---- | ---------------- | ---------- | --- | ---------- |
andperformingmutationsusingtraditionalbyte-leveltech-
niques.PreviousexperimentsconfirmtheLLM’seffective- format”or“redundantfeatures.”Hallucinationsprimarilyoc-
ness in seed generation. To assess the need for combining curduringgeneratorsynthesis,oftenreferencingnon-existent
LLMswithtraditionalmethods,wecreatedG2FUZZ(LLM-
functionsorattributesandtriggeringexceptionssuchasAt-
Only),whichsolelyreliesonLLMsforseedmutation.Test- tributeErrororNotImplementedError.However,duetoour
G2FUZZ(LLM-Only) debuggingstrategy(Alg.2),theseerrorscausedbyhalluci-
| ing on UNIFUZZ | shows | that |     | finds |     |     |     |     |     |     |
| -------------- | ----- | ---- | --- | ----- | --- | --- | --- | --- | --- | --- |
fewer edges and has lower throughput,often less than 1% nationsarepromptlydetectedwhenexecutingthegenerator.
ofG2FUZZ,asshowninTable14.Italsostruggleswithlow- TheLLMthenattemptstofixthem,effectivelymitigatingthe
levelmutationsandissignificantlymoreexpensive,making impactofhallucinations.
theintegrationofLLMsandtraditionalfuzzingbothneces-
saryandefficient.
E LibraryInfluence
D PromptEffectiveness Toevaluatetheimpactofdifferentlibrariesonthequalityof
generator,weconductexperimentsacrossfourtargetformats.
Toevaluatetheeffectivenessofthepromptsweused,weana- Specifically,weuseGPT-4toconstructgenerators,specifying
lyzethreeattributes.1)Validity:Thegeneratorproducedby thelibrarytobeusedintheprompt,suchas“Youmustusecv2

Table15:Analysisofprompteffectivenessfordifferentfor- Table 17: Benchmark programs selected from UNIFUZZ,
| mats. |     |     |     |     |     |     | FuzzBench,andMAGMA. |     |           |     |       |     |
| ----- | --- | --- | --- | --- | --- | --- | ------------------- | --- | --------- | --- | ----- | --- |
|       |     |     |     |     |     |     | UNIFUZZ             |     | FuzzBench |     | MAGMA |     |
ValidityRate
Format PSTF(GPT-4) PUUF(GPT-4) gdk-pixbuf-pixdata bloaty_fuzz_target libpng_read_fuzzer
|     | GPT-3.5 | GPT-4 |     |     |     |     |       |     |                |     |                  |     |
| --- | ------- | ----- | --- | --- | --- | --- | ----- | --- | -------------- | --- | ---------------- | --- |
|     |         |       |     |     |     |     | jhead |     | freetype2-2017 |     | read_rgba_fuzzer |     |
TIFF 91.90% 94.31% 90.00% 90.00% mp3gain harfbuzz-1.3.2 tiffcp
BMP 57.25% 97.26% 50.00% 60.00% ffmpeg lcms-2017-03-21 pdf_fuzzer
|     |        |        |        |     |        |     | tiffsplit |     | libjpeg-turbo-07-2017       |     | pdfimages      |     |
| --- | ------ | ------ | ------ | --- | ------ | --- | --------- | --- | --------------------------- | --- | -------------- | --- |
| JPG | 98.05% | 99.16% | 70.00% |     | 80.00% |     |           |     |                             |     |                |     |
|     |        |        |        |     |        |     | pdftotext |     | libpcap_fuzz_both           |     | pdftoppm       |     |
| PNG | 90.78% | 99.65% | 77.77% |     | 77.77% |     |           |     |                             |     |                |     |
|     |        |        |        |     |        |     | mp42aac   |     | libpng-1.2.56               |     | sndfile_fuzzer |     |
| GIF | 88.37% | 100%   | 75.00% |     | 75.00% |     | flvmeta   |     | openssl_x509                |     |                |     |
| ICO | 47.88% | 100%   | 57.14% |     | 85.71% |     | imginfo   |     | vorbis-2017-12-11           |     |                |     |
| TGA | 22.22% | 82.81% | 88.88% |     | 88.88% |     | exiv2     |     | woff2-2016-05-06            |     |                |     |
| PNM | 67.12% | 90.00% | 37.50% |     | 37.50% |     |           |     | zlib_zlib_uncompress_fuzzer |     |                |     |
| MP4 | 35.29% | 90.17% | 60.00% |     | 80.00% |     |           |     |                             |     |                |     |
| PDF | 98.80% | 95.71% | 60.00% |     | 86.66% |     |           |     |                             |     |                |     |
Table18:TheadditionalanalysistimeofLLMGenFuzz.The
unitisseconds.
tocreatethisPythongenerator.”Foreachformat,weselect
twolibrariescapableofgeneratingfilesinthecorresponding Average(ExtraPCT1)
|     |     |     |     |     |     |     |     | Programs | GroupI GroupII | GroupIII |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | -------------- | -------- | --- | --- |
format.
|     |     |     |     |     |     |     |     | lcms  | 131 299 | 158 | 196(0.22%) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----- | ------- | --- | ---------- | --- |
|     |     |     |     |     |     |     |     | woff2 | 431 434 | 220 | 361(0.42%) |     |
TheresultsarepresentedinTable16.Inmostcases,dif-
|     |     |     |     |     |     |     |     | vorbis | 552 329 | 194 | 358(0.41%) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------- | --- | ---------- | --- |
ferentlibraries exhibitlarge variations in feature coverage, freetype2 817 788 410 671(0.77%)
as observed in the cases of JPG, MP4, and PDF. Notably, libpcap 261 173 220 218(0.25%)
|     |     |     |     |     |     |     |     | bloaty | 743 990 | 1,043 | 925(1.06%) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------- | ----- | ---------- | --- |
combiningmultiplelibrariesleadstohigheroverallfeature harfbuzz 465 651 282 466(0.54%)
coveragebecausetheircomplementaryfunctionalitiesenable libjpeg-turbo 575 163 118 285(0.33%)
|     |     |     |     |     |     |     |     | libpng | 379 158 | 47  | 194(0.22%) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------ | ------- | --- | ---------- | --- |
theconstructionofmoresophisticatedgenerators. openssl 1,259 189 551 666(0.77%)
|     |     |     |     |     |     |     |     | zlib | 220 161 | 110 | 163(0.19%) |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ------- | --- | ---------- | --- |
1ExtraPCT:Thepercentageoftheadditionalanalysistimecomparedtothe
| Table | 16: Feature | coverage | achieved | by  | using different | li- |     |     |     |     |     |     |
| ----- | ----------- | -------- | -------- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- |
23hoursoffuzzingtime.
braries.
Table19:ThemediancodecoverageachievedbyG2FUZZat
|     | Format | Library | FeatureCov | ValidNum/InvalidNum |     |     |     |     |     |     |     |     |
| --- | ------ | ------- | ---------- | ------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
23hoursand23hoursand45mins.
|     |     | PIL       | 252 | 40/0  |     |     |     |     |     |     |     |     |
| --- | --- | --------- | --- | ----- | --- | --- | --- | --- | --- | --- | --- | --- |
|     | JPG | cv2       | 337 | 40/0  |     |     |     |     |     |     |     |     |
|     |     | Unlimited | 984 | 130/0 |     |     |     |     |     |     |     |     |
23hoursand
|     |     | cv2       | 471 | 23/11 |     |     |     | Programs                  |     | 23hours  | 45minutes | Diff |
| --- | --- | --------- | --- | ----- | --- | --- | --- | ------------------------- | --- | -------- | --------- | ---- |
|     | MP4 | moviepy   | -   |       | -   |     |     |                           |     |          |           |      |
|     |     |           |     |       |     |     |     | bloaty_fuzz_target        |     | 6,377.0  | 6,377.0   | 0    |
|     |     | Unlimited | 290 | 19/15 |     |     |     | freetype2_ftfuzzer        |     | 11,630.0 | 11,630.0  | 0    |
|     |     | fpdf      | 353 | 27/0  |     |     |     | harfbuzz_hb-shape-fuzzer  |     | 10,935.5 | 10,935.5  | 0    |
|     |     |           |     |       |     |     |     | lcms_cms_transform_fuzzer |     | 1,610.0  | 1,610.0   | 0    |
|     | PDF | PyPDF2    | 72  | 18/4  |     |     |     |                           |     |          |           |      |
Unlimited 559 50/1 libjpeg-turbo_libjpeg_turbo_fuzzer 2,551.0 2,551.0 0
|     |      |           |     |      |     |     |     | libpcap_fuzz_both             |     | 3,003.0 | 3,003.0 | 0   |
| --- | ---- | --------- | --- | ---- | --- | --- | --- | ----------------------------- | --- | ------- | ------- | --- |
|     |      | PIL       | 164 | 27/0 |     |     |     | libpng_libpng_read_fuzzer     |     | 2,006.0 | 2,006.0 | 0   |
|     | TIFF | tifffile  | 161 | 22/2 |     |     |     | openssl_x509                  |     | 5,833.0 | 5,833.0 | 0   |
|     |      | Unlimited | 387 | 48/8 |     |     |     | vorbis_decode_fuzzer          |     | 1,283.5 | 1,283.5 | 0   |
|     |      |           |     |      |     |     |     | woff2_convert_woff2ttf_fuzzer |     | 1,178.0 | 1,178.0 | 0   |
|     |      |           |     |      |     |     |     | zlib_zlib_uncompress_fuzzer   |     | 471.0   | 471.0   | 0   |

| (a)harfbuzz |     | (b)lcms | (c)libjpeg | (d)libpng |
| ----------- | --- | ------- | ---------- | --------- |
| a 叩         | ♦   |         |            |           |
o 响 泗
ea 」 妇
> 泗 g
O uuq 叩 螂
u 干+
eJ
| a q p | !   |     |     |     |
| ----- | --- | --- | --- | --- |
4 ·
| u   | ♦   |     |     |     |
| --- | --- | --- | --- | --- |
| e   | ♦   |     |     |     |
| a u |     |     |     |     |

Fuzzer (highest median coverage on the left)
|     | (e)openssl | (f)woff2 |     | (g)zlib |
| --- | ---------- | -------- | --- | ------- |
Figure12:CodecoveragedistributionsachievedinaFuzzBenchexperiment.
```
<TARGET_GENERATOR>
```
Based on the above code, provide me with a more complex code
that can generate <FROMAT> files with additional more
complex file <features/structures>.
Figure13:Thepromptforrandommutation.
Figure14:Thetotalthroughputofdifferentfuzzersrunning
for24hours.
Figure15:AverageCodeCoverageEvolutionOverTimefor
vorbis_decode_fuzzer.
