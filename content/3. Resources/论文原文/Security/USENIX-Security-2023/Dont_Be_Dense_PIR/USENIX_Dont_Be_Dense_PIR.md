---
publish: true
---

|         | Don’t    |           | be Dense:    | Efficient   |               | Keyword       | PIR                                                 | for Sparse |           | Databases         |     |         |
| ------- | -------- | --------- | ------------ | ----------- | ------------- | ------------- | --------------------------------------------------- | ---------- | --------- | ----------------- | --- | ------- |
|         |          |           | SarvarPatel∗ |             |               | JoonYoungSeo† |                                                     |            | KevinYeo‡ |                   |     |         |
|         |          |           | Abstract     |             |               |               | multi-serversettingare,typically,moreefficient.How- |            |           |                   |     |         |
|         |          |           |              |             |               |               | ever, they                                          | rely on    | stronger  | trust assumptions |     | between |
| In this | paper,we | introduce |              | SparsePIR,a | single-server |               |                                                     |            |           |                   |     |         |
keywordprivateinformationretrieval(PIR)construction differentorganizationsthatmaybedifficulttomaterial-
ize.Inourwork,wewillfocusonsingle-serverPIR.
thatenablesqueryingoversparsedatabases.Atitscore,
SparsePIRisbasedonanovelencodingalgorithmthat The standard PIR problem considers databases that
|     |     |     |     |     |     |     | are n-entry | arrays. | The client’s | goal | is to | retrieve the |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ------- | ------------ | ---- | ----- | ------------ |
encodessparsedatabaseentriesaslinearcombinations
|     |     |     |     |     |     |     | i-th |     |     |     |     | i   |
| --- | --- | --- | --- | --- | --- | --- | ---- | --- | --- | --- | --- | --- |
whilebeingcompatiblewithimportantPIRoptimizations entry in the array without revealing index to the
includingrecursion.SparsePIRachievesresponseover- serverholdingthepublicdatabase.Unfortunately,inmost
practicalapplications,databasesmorecloselyresemble
headthatishalfofstate-of-theartkeywordPIRschemes
withoutrequiringlong-termclientstorageoflinear-sized key-value pairs where users wantto retrieve the value
mappings.Wealsointroducetwovariants,SparsePIRg associatedtoacertain key. Forexample,wecan think
andSparsePIRc,thatfurtherreducesthesizeoftheserv- ofdata sets like contactlists,videos,websites,etc. To
ingdatabaseatthecostofincreasedencodingtimeand address this,keyword PIR [20] was introduced where
databasesconsistofkey-valuepairsandaclientwishes
smalladditionalclientstorage,respectively.Ourframe-
worksenableperformingkeywordPIRwith,essentially, toretrievethevalueassociatedwithacertainkey.
thesamecostsasstandardPIR.Finally,wealsoshowthat AnaivesolutionforkeywordPIRistoreplicatemap-
SparsePIRmaybeusedtobuildbatchkeywordPIRwith pingsfromkeystoarrayindicesthatneedtobestored
halvedresponseoverheadwithoutanyclientmappings. byallclients.Toquery,aclientusesthemappingtode-
|     |     |     |     |     |     |     | termine the                           | arrayindex | storing | the | entryassociatedto |        |
| --- | --- | --- | --- | --- | --- | --- | ------------------------------------- | ---------- | ------- | --- | ----------------- | ------ |
|     |     |     |     |     |     |     | thequerykeyandusesastandardPIRscheme. |            |         |     |                   | Unfor- |
1 Introduction
tunately,thisrequirestheclienttostoremappingsthat
arelinearinthedatabasesize.Choretal.[20]presented
Privateinformationretrieval(PIR)[21]isanimportant
|     |     |     |     |     |     |     | a multiple | round | solution introducing |     | additional | over- |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ----- | -------------------- | --- | ---------- | ----- |
cryptographicprimitivethatallowsaclienttoretrieveen-
head.AnothersolutiontokeywordPIRutilizescuckoo
triesfromapublicdatabasewithoutrevealingtheclient’s
hashing[8]inasingleround,butresultsin2xresponse
entryofinterest.Duetoitsstrongprivacyguarantees,PIR
overheadcomparedtostandardPIR.Arecentwork[50]
isacriticalbuildingblockformanypracticalapplications
|     |     |     |     |     |     |     | builds one-round |     | keyword | PIR using | constant-weight |     |
| --- | --- | --- | --- | --- | --- | --- | ---------------- | --- | ------- | --------- | --------------- | --- |
includingadvertisements[36],anonymouscommunica-
equalityoperators.However,thisschemerequiressignif-
| tion [7,10,46,52], |     | contact |     | discovery | [12,27], | device |     |     |     |     |     |     |
| ------------------ | --- | ------- | --- | --------- | -------- | ------ | --- | --- | --- | --- | --- | --- |
icantlymorecommunicationandcomputationthanre-
enrollment[4],mediaconsumption[38],passwordleak
centPIRschemes.Inthecurrentstateofaffairs,moving
checks[8]androutenavigation[65].
fromstandardPIRtokeywordPIRrequiresincreasing
| PIR | has been | studied | in  | two settings | with | a single |     |     |     |     |     |     |
| --- | -------- | ------- | --- | ------------ | ---- | -------- | --- | --- | --- | --- | --- | --- |
roundtrips,doublingtheresponsesizeorlargelong-term
server[17,25,34,37,45,47,55,56,63]ormultiple,non-
clientstorage.Noneofthesechoicesareappealing.This
colludingservers[13,21,29,30,35].PIRschemesinthe
paperaddressesthisinefficiency.
∗Google,sarvar@google.com.
|     |     |     |     |     |     |     | Our Contributions. |     | We present |     | SparsePIR | that is a |
| --- | --- | --- | --- | --- | --- | --- | ------------------ | --- | ---------- | --- | --------- | --------- |
†Google,jyseo@google.com.
|     |     |     |     |     |     |     | framework | for building | keyword | PIR | from | a standard |
| --- | --- | --- | --- | --- | --- | --- | --------- | ------------ | ------- | --- | ---- | ---------- |
‡GoogleandColumbiaUniversity,kwlyeo@google.com.

aandbin{0,1}n,wedenotethedotproductoperator
|     |     |     | Client |     | Encoding | Response |     |     |     |     |     |     |     |
| --- | --- | --- | ------ | --- | -------- | -------- | --- | --- | --- | --- | --- | --- | --- |
Storage Size Overhead asa·b=∑n a[i]·b[i].ForamatrixM=[M ... M ],
|                  |     |     |      |     |        |     |           | i=1      |             |          |              |     | 1 m            |
| ---------------- | --- | --- | ---- | --- | ------ | --- | --------- | -------- | ----------- | -------- | ------------ | --- | -------------- |
| ClientMapping    |     |     | n    |     | n      | 1x  |           |          |             |          |              |     |                |
|                  |     |     |      |     |        |     | we denote | the      | dot product | a·M=[a·M |              |     | 1 ... a·M n ]. |
| CuckooHashing[8] |     |     | O(1) |     | (2+ε)n | 2x  |           |          |             |          |              |     |                |
|                  |     |     |      |     |        |     | For any   | database | consisting  |          | of key-value |     | pairs D =      |
Constant-Weight[50] O(1) n 2-9x {(k ,v ),...,(k ,v )},wedenoteD[k]tobethestandard
|                |     |     |      |     |        |     | 1 1       |           | n n |       |            |     |               |
| -------------- | --- | --- | ---- | --- | ------ | --- | --------- | --------- | --- | ----- | ---------- | --- | ------------- |
| Ours:SparsePIR |     |     | O(1) |     | (1+ε)n | 1x  |           |           |     |       |            |     |               |
|                |     |     |      |     |        |     | operation | to access | the | value | associated |     | with k in the |
Ours:SparsePIRg
|                 |     |     | O(1)    |     | (1+ε)n  | 1x  | databaseD.Ifk=k |     | forsomei∈[n],thenD[k]=v.If |     |     |     |     |
| --------------- | --- | --- | ------- | --- | ------- | --- | --------------- | --- | -------------------------- | --- | --- | --- | --- |
| Ours:SparsePIRc |     |     |         |     |         |     |                 |     | i                          |     |     |     | i   |
|                 |     |     | <11.2KB |     | (1.03)n | 1x  |                 |     |                            |     |     |     |     |
k̸=k i foralli∈[n],thenD[k]=⊥.
|     |     |     |     |     |     |     | Next, | we present | definitions |     | of  | PIR and | batch PIR. |
| --- | --- | --- | --- | --- | --- | --- | ----- | ---------- | ----------- | --- | --- | ------- | ---------- |
Figure1:One-roundkeywordPIRcomparison.Response
Formaldefinitionsmaybefoundinthefullversion.
| overhead | is compared |     | to state-of-the-art |     | standard | PIR, |         |      |          |           |     |                |      |
| -------- | ----------- | --- | ------------------- | --- | -------- | ---- | ------- | ---- | -------- | --------- | --- | -------------- | ---- |
|          |             |     |                     |     |          |      | PIR. We | will | restrict | ourselves | to  | single-server, | one- |
Spiral[51],thatqueryoverdensen-entryarrays.
roundschemes.AkeywordPIRschemeconsistsofalgo-
PIR.ThecoretechniquebehindSparsePIRistheusage rithmsΠ=(Init,Encode,Query,Answer,Decrypt).Init
| of encoding | algorithms |     | that     | aim to   | encode  | key-value |                |     |              |     | Encode |         |           |
| ----------- | ---------- | --- | -------- | -------- | ------- | --------- | -------------- | --- | ------------ | --- | ------ | ------- | --------- |
|             |            |     |          |          |         |           | initializes    | all | crypto keys  | and |        | enables | prepar-   |
| pairs as    | a function | of  | multiple | database | entries | as op-    |                |     |              |     |        |         |           |
|             |            |     |          |          |         |           | ing a database |     | for queries. | The | client | runs    | Query and |
posedtoallocatingeachkey-valuepairintoasingleentry Decrypttocreatearequestandcomputeananswerfrom
asdonebypriorhashingschemes.
theserver’sresponse.TheserverrunsAnswertocreatea
SparsePIRensuresthatrequestandresponsesizesare responsefromtheclient’srequest.Forcorrectness,the
identicaltotheunderlyingstandardPIRschemeswith- clientshouldretrievethecorrectvalueassociatedwith
outanyadditionalclientstorage.Asaresult,SparsePIR
thequeriedkey.Intermsofprivacy,theservershouldnot
halvestheresponsesizecomparedtothecuckoohashing learninformationabouttheclient’squeriedkey.
| keyword       | PIR approach |           | [8].     | The only | slight     | drawback |             |         |               |          |         |              |            |
| ------------- | ------------ | --------- | -------- | -------- | ---------- | -------- | ----------- | ------- | ------------- | -------- | ------- | ------------ | ---------- |
|               |              |           |          |          |            |          | Batch PIR.  | In      | this setting, |          | clients | query        | a batch of |
| of SparsePIR  | is           | a small   | increase | in       | the server | compu-   |             |         |               |          |         |              |            |
|               |              |           |          |          |            |          | ℓ ≥ 1 query | keys    | to            | retrieve | all     | ℓ associated | values.    |
| tation costs. | For          | databases | with     | one      | million    | 256-byte |             |         |               |          |         |              |            |
|               |              |           |          |          |            |          | A batch     | keyword | PIR           | consists | of      | the same     | four algo- |
entries,SparsePIRbuiltonSpiral[51]halvesresponse
rithmsΠ=(Init,Encode,Query,Answer,Decrypt).For
sizesfrom42KBto21KBcomparedtothecuckoohash-
|     |     |     |     |     |     |     | correctness,the |     | scheme | should | return | all | ℓ associated |
| --- | --- | --- | --- | --- | --- | --- | --------------- | --- | ------ | ------ | ------ | --- | ------------ |
ingapproach[8]usingSpiral.Inexchange,SparsePIR
|                                  |     |     |     |     |            |     | valuescorrectly. |     | Intermsofprivacy,theservershould |     |     |     |     |
| -------------------------------- | --- | --- | --- | --- | ---------- | --- | ---------------- | --- | -------------------------------- | --- | --- | --- | --- |
| usesonly2%moreservercomputation. |     |     |     |     | Wealsoshow |     |                  |     |                                  |     |     |     |     |
notlearnanyinformationabouttheclient’ssetofqueried
thatSparsePIRuses2-12xsmallercommunicationand
keys.
atleast10xlesscomputationthanconstant-weightkey-
wordPIR[50].WealsointroduceSparsePIRg
thathas
identical request and response overhead as SparsePIR. 3 KeywordPIRoverSparseDatabases
SparsePIRg
| The main | benefit | of  |     | is  | that we | can further |     |     |     |     |     |     |     |
| -------- | ------- | --- | --- | --- | ------- | ----------- | --- | --- | --- | --- | --- | --- | --- |
decreasetheencodingsizeand,thus,computationtime 3.1 State-of-the-ArtPIRSchemes
byupto20%inexchangeformoretimetoencodethe
database.Finally,weintroduceSparsePIRcthatachieves We start by revisiting the current state-of-the-art PIR
schemes[6–9,33,51,54,57]thatutilizeleveledfullyho-
nearoptimalencodingsizewithsmalladditionalclient
momorphicencryption(FHE)builtontheRingLearning
| storage of | < 11.2 | KB. | In summary,we |     | show | that key- |     |     |     |     |     |     |     |
| ---------- | ------ | --- | ------------- | --- | ---- | --------- | --- | --- | --- | --- | --- | --- | --- |
withErrors(RLWE)assumption[49].
wordPIRrequiresnoadditionalcommunicationanda
veryslightincreaseincomputationcomparedtostandard Fully Homomorphic Encryption. Most leveled FHE
schemesrelyingonRLWE(suchas[15,16,31])share
PIR.Furthermore,webelieveanyfutureimprovements
tostandardlattice-basedPIRshouldalsoenablefaster similarmathematicalstructures.TheseFHEschemesare
keywordPIRschemesusingourframeworks.Figure1 basedonacyclotomicringR=Z[x]/(xN+1)whereN
|     |     |     |     |     |     |     | is the degree | of  | the polynomials. |     | N   | is also | commonly |
| --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ---------------- | --- | --- | ------- | -------- |
presentsacomparisonofkeywordPIRs.Finally,weshow
thatwecanbuildbatchkeywordPIRschemesusingany referred to as the ring dimension. Plaintext values are
|                                                 |     |     |     |     |     |     | encodedaspolynomialsintheringR              |         |             |                 |        | =R/αRwhere |               |
| ----------------------------------------------- | --- | --- | --- | --- | --- | --- | ------------------------------------------- | ------- | ----------- | --------------- | ------ | ---------- | ------------- |
| ofourSparsePIRalgorithmswithsmallresponsesizes. |     |     |     |     |     |     |                                             |         |             |                 |        | α          |               |
|                                                 |     |     |     |     |     |     | the integer                                 | α       | is referred | to              | as the | plaintext  | modulus.      |
|                                                 |     |     |     |     |     |     | Ciphertexts                                 | consist | of          | two polynomials |        |            | from the ring |
| 2 Preliminaries                                 |     |     |     |     |     |     | R =R/βRforsomeintegerβ.Thesecretkeysinthese |         |             |                 |        |            |               |
β
FHEschemeswillbepolynomialsinRwithverysmall
⊺
Wedenotevectorsvascolumnvectorsandv asrowvec- coefficients.Anencryptionofaplaintextpolynomialpt
tors.Wedenotethei-thentryofvbyv[i].Fortwovectors will look like (r,r·s+e+pt) where r is a uniformly

random element from R , s is the secret key and e is other words, one can view our constructions as trans-
β
thenoisepolynomialwhereeachcoefficientistypically formingstandardPIRschemesintokeywordPIRsthat
small and drawn from a truncated Gaussian distribu- canhandlesparsedatabases.Wewillassumeastandard
tion.Todecryptaciphertext(ct ,ct ),onewillcompute PIRschemeΠ =(Init,Query,Answer,Decrypt)with
|     |     |     |     | 1 2 |     |     |     |     | PIR |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ct −(ct ·s)=e+pt.Aslongasthenoisepolynomiale thefollowingproperties:
2 1
remainssmall,theplaintextptcanberetrievedbyround-
|     |     |     |     |     |     |     | •   | The underlying |     | PIR | scheme | will represent | any m- |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | --- | ------ | -------------- | ------ |
ingtoremovee.
|     |     |     |     |     |     |     |     | element | database | as  | a hypercube | with | dimensions |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | -------- | --- | ----------- | ---- | ---------- |
FHEschemesallowhomomorphicoperationsoverci-
|                                                       |                                      |     |     |           |     |             |     | d 1 ×...×d        | z .   | Our framework |                           | will be parameterized |            |
| ----------------------------------------------------- | ------------------------------------ | --- | --- | --------- | --- | ----------- | --- | ----------------- | ----- | ------------- | ------------------------- | --------------------- | ---------- |
| phertexts.                                            | PIRschemesgenerallyrelyuponthreecore |     |     |           |     |             |     |                   |       |               |                           |                       |            |
|                                                       |                                      |     |     |           |     |             |     | bythedimensions(d |       |               | ,...,d )thatmaydependonn. |                       |            |
| operations:                                           | ciphertext-ciphertext                |     |     | addition, |     | ciphertext- |     |                   |       |               | 1 z                       |                       |            |
|                                                       |                                      |     |     |           |     |             | •   | The Π             | .Init | algorithm     | produces                  | client                | key ck and |
| plaintextmultiplicationandciphertext-ciphertextmulti- |                                      |     |     |           |     |             |     | PIR               |       |               |                           |                       |            |
serverkeysk(i.e.,query-independentparameters).
plication.Thecostsandnoisegrowthoftheseoperations
|                                                        |      |                                |     |     |     |           | •   | TheΠ     | .Queryalgorithmreceiveszvectorsoflength |     |     |     |     |
| ------------------------------------------------------ | ---- | ------------------------------ | --- | --- | --- | --------- | --- | -------- | --------------------------------------- | --- | --- | --- | --- |
| arequitedifferent.Typically,ciphertext-ciphertextaddi- |      |                                |     |     |     |           |     | PIR      |                                         |     |     |     |     |
|                                                        |      |                                |     |     |     |           |     | d ,...,d | thatitwillhomomorphicallyencryptandup-  |     |     |     |     |
| tion is the                                            | most | efficient,ciphertext-plaintext |     |     |     | multipli- |     | 1        | z                                       |     |     |     |     |
cationisabitmoreexpensiveandciphertext-ciphertext loadtotheserver.ForstandardPIR,thesezvectorsare
indicatorvectorsrepresentingthequeryindexineach
multiplicationistheworstofthethree.
dimension.Ascompressionandobliviousexpansion
Query-IndependentPIRParameters.Followingprior
canhandlearbitraryHammingweightvectors(seethe
| works [8,9,51,54],we |     |     | willconsiderthe |     | modelwhere |     |     |                                          |     |     |     |     |     |
| -------------------- | --- | --- | --------------- | --- | ---------- | --- | --- | ---------------------------------------- | --- | --- | --- | --- | --- |
|                      |     |     |                 |     |            |     |     | fullversionformoredetails),weassumethatΠ |     |     |     |     | can |
PIR
theclientuploadsquery-independentparameterstothe
|               |            |     |        |        |     |          |     | receivearbitraryvectorsforthefirstd |     |     |     | 1 -lengthvector. |     |
| ------------- | ---------- | --- | ------ | ------ | --- | -------- | --- | ----------------------------------- | --- | --- | --- | ---------------- | --- |
| server. These | parameters |     | willbe | usedby | the | serverto |     |                                     |     |     |     |                  |     |
processallfuturerequestssentbytheclient.Atahigh • The Π PIR .Answer algorithm receives an encoding E,
|                                                 |        |            |     |        |            |        |     | a two-dimensional |     | matrix      | of  | size d ×⌈m/d | ⌉, and       |
| ----------------------------------------------- | ------ | ---------- | --- | ------ | ---------- | ------ | --- | ----------------- | --- | ----------- | --- | ------------ | ------------ |
| level,these                                     | public | parameters |     | enable | the server | to op- |     |                   |     |             |     | 1            | 1            |
|                                                 |        |            |     |        |            |        |     | homomorphic       |     | encryptions | of  | vectors v    | ,...,v . The |
| erateoverFHEciphertextsefficiently.Weassumethat |        |            |     |        |            |        |     |                   |     |             |     |              | 1 z          |
Π PIR .AnsweralgorithmwillperformthestandardPIR
theseparametersaresentbytheclienttotheserverinan
|                                             |     |     |     |     |     |     |     | algorithmofapplyingv       |     |     | toEtoobtaina⌈m/d |        | ⌉vec-        |
| ------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | -------------------------- | --- | --- | ---------------- | ------ | ------------ |
| “offline”phasebeforeanyqueriesareperformed. |     |     |     |     |     |     |     |                            |     |     | 1                |        | 1            |
|                                             |     |     |     |     |     |     |     | tor,arrangethevectorintoad |     |     |                  | ×⌈m/(d | d            |
|                                             |     |     |     |     |     |     |     |                            |     |     |                  | 2      | 1 2 )⌉matrix |
Recursion.AcommontechniqueinstandardPIRisto
|     |     |     |     |     |     |     |     | andapplyv | toobtainavectorofsize⌈m/(d |     |     |     | d )⌉,and |
| --- | --- | --- | --- | --- | --- | --- | --- | --------- | -------------------------- | --- | --- | --- | -------- |
utilize recursion introduced in [45] that represents the 2 1 2
repeatthisprocessforallzdimensions.
| n-entry     | array | database                       | as a | hypercube | of dimensions |     |     |      |                                         |     |     |     |     |
| ----------- | ----- | ------------------------------ | ---- | --------- | ------------- | --- | --- | ---- | --------------------------------------- | --- | --- | --- | --- |
|             |       |                                |      |           |               |     | •   | TheΠ | .Decryptalgorithmreceivestheserver’sre- |     |     |     |     |
| d ×d ×...×d |       | wheretheproductofthedimensions |      |           |               |     |     | PIR  |                                         |     |     |     |     |
| 1 2         |       | z                              |      |           |               |     |     |      |                                         |     |     |     |     |
sponseandproducesadecryptedanswer.Wewillas-
| is atleastn,d                  | 1          | ···d z ≥n. | To                           | query | an entry,the   | client |     |                                          |     |           |         |             |     |
| ------------------------------ | ---------- | ---------- | ---------------------------- | ----- | -------------- | ------ | --- | ---------------------------------------- | --- | --------- | ------- | ----------- | --- |
|                                |            |            |                              |       |                |        |     | sume thatthe                             |     | answerwas | already | decodedfrom | the |
| willgeneratezindicatorvectorsv |            |            |                              |       | ∈{0,1}d1,...,v | ∈      |     |                                          |     |           |         |             |     |
|                                |            |            |                              | 1     |                | z      |     | decryptedplaintextpolynomialintoastring. |     |           |         |             |     |
| {0,1}dz                        | whereeachv |            | haszeroeseverywhereexceptfor |       |                |        |     |                                          |     |           |         |             |     |
i
| a one indicating |     | the location |     | of the | entry with | respect |     |     |     |     |     |     |     |
| ---------------- | --- | ------------ | --- | ------ | ---------- | ------- | --- | --- | --- | --- | --- | --- | --- |
to the dimension d. The benefit of this representation 3.2 Warm-Up: Keyword PIR without Re-
i
| is that querying               |     | for an | entry | only | requires | uploading |     | cursion |     |     |     |     |     |
| ------------------------------ | --- | ------ | ----- | ---- | -------- | --------- | --- | ------- | --- | --- | --- | --- | --- |
| an encryptedbitvectoroflengthd |     |        |       |      | +...+d   | thatcan   |     |         |     |     |     |     |     |
|                                |     |        |       | 1    |          | z         |     |         |     |     |     |     |     |
besubstantiallysmallerthann. Forexample,ifweset Tostart,weconsiderasimplifiedsettingwherewewill
| √   |     | √   |     |     | √   |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
d 1 = nandd 2 = n,thetotalbitlength2 n.Anunfor- utilizeanunderlyingPIRschemeΠ PIR thatdoesnotuse
tunatesidenoteisthatrecursionintroducestheneedfor recursion.Inotherwords,wewillrepresenttheencoded
ciphertext-ciphertextmultiplication.ThemajorityofPIR databaseEasavectoroflengthd =m.Forthissetting,
1
works aim to achieve recursion while obtaining better we do not care if the query vector can be represented
trade-offsbetweencommunicationandcomputation.In succinctlyandwewillbecontentwithuploadinganen-
ourwork,wewillpresentkeywordPIRschemeswhose cryptedversionofalinearnumberofvalues.Infuture
constructionswillutilizerecursiontoreducecommuni- sections,wewillfixthisissuetoenablerecursion.How-
cation.WenotethereareotherimportantPIRtechniques ever,we choose to startfrom this simplifiedsetting as
suchascompressionandobliviousexpansion(seethefull itilluminatessomeoftheideasthatwewilluseinour
version).However,unlikerecursion,theyendupbeing moreefficientconstructions.
easiertofitintoourframework. RepresentingKey-ValuePairs.Foranypair(k,v)ofa
Building on a PIR Scheme. Our constructions will keyandavalue,wewillrepresenttheminthefollowing
be frameworks that can be built upon FHE based PIR way. For some hash key K, we denote rep(K,k,v) as
schemes, which are the most common ones today. In the hash evaluation of k concatenated with v. In more

oftheencodeddatabaseE∈Fm.Therefore,wewould
detail,rep(K,k,v)=F(K,k)||vwherewedenote||as
concatenation. Weassumetherepresentationlengthis like to pick m in such a way that M has full row rank
fixedforall(k,v).Forconvenience,wewillassumethat withhighprobabilitywhileminimizingthesizeofthe
| rep(K,k,v)issmallenoughtobeuniquelyrepresentedin |     |     |     |     |     |     |     | encoding. |     |     |     |     |     |
| ------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --------- | --- | --- | --- | --- | --- |
Z whereαistheplaintextmodulusandeachrep(K,k,v)
α
willbestoredintooneciphertext.Wewilldiscusslater DecodinganEntry.Fromtheaboveencodingscheme,
howtohandlethesettingwhenlargervaluesandpacking
wenotethatdecodinganentryisprettystraightforward.
multiplevaluesintoaciphertext. Foranyquerykeyk,theclientwillrandomlygenerate
Usingourrepresentation,wenotethatonecandistin- thevectorv .Theclientalsocomputesthehashevalua-
k
| guish | representations |     | of  | different | key-value | pairs. | For |         |                                          |     |     |     |     |
| ----- | --------------- | --- | --- | --------- | --------- | ------ | --- | ------- | ---------------------------------------- | --- | --- | --- | --- |
|       |                 |     |     |           |           |        |     | tionF(K | r ,k).UsingourPIRschemewithoutrecursion, |     |     |     |     |
differentkeysk 1 ̸=k 2 ,theresultingrepresentationswill theclientuploadsanencryptedversionofv .ThePIR
k
bedifferentrep(K,k ,v)̸=rep(K,k ,v)exceptwithneg- schemecontinueswithoutchangeandtheclientwilleven-
|     |     |     | 1   |     | 2   |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ligible probability as the representations are the same tuallyreceivethedotproductv ·Einplaintext.Theclient
k
onlywhenF(K,k )=F(K,k ).Ifthehashoutputissuf- willparsethisresultasa||b. Ifa=F(K ,k),thenthe
|     |     | 1   |     | 2   |     |     |     |     |     |     |     |     | r   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
ficientlylong,thesecollisionsdonotoccurinpractice.
clientreturnsbastheretrievedvalue.Otherwise,when
EncodingtheDatabase.LetK bethehashkeyusedto a̸=F(K ,k),thentheclientreturns⊥.
|     |     |     |     | r   |     |     |     |     | r   |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
generaterepresentationsofkey-valuepairs.Throughout Ifk=k forsome(k,v)∈D,thenweknowthatv ·E
|     |     |     |     |     |     |     |     |     | i   | i i |     |     | k   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
thissection,wewillassumethatalloperationsaredone
|       |         |            |                       |           |            |              |        | willbeexactlyF(K                               |                                              | ,k)||v | enablingretrievalofv.On |     |     |
| ----- | ------- | ---------- | --------------------- | --------- | ---------- | ------------ | ------ | ---------------------------------------------- | -------------------------------------------- | ------ | ----------------------- | --- | --- |
| in Z  | where   | α is       | the plaintextmodulus. |           |            | We willasso- |        |                                                |                                              | r i    | i                       |     | i   |
|       | α       |            |                       |           |            |              |        | theotherhand,supposethatk∈/D.Then,weknowthat   |                                              |        |                         |     |     |
| ciate | eachkey | k∈K        | witha                 | uniformly |            | random       | subset |                                                |                                              |        |                         |     |     |
|       |         |            |                       |           |            |              |        | a̸=F(K                                         | r ,k)meaningtheclientreturnstherightresponse |        |                         |     |     |
| ofS   | ⊆[m]    | thatwillbe | randomly              |           | generated. | The          | goal   |                                                |                                              |        |                         |     |     |
|       | k       |            |                       |           |            |              |        | (exceptwithnegligibleprobabilityofcollisions). |                                              |        |                         |     |     |
istoarrangetheencodingEsuchthatforanydatabase
| D={(k | ,v  | ),...,(k | ,v )}andforanyintegeri∈[n], |     |     |     |     |       |            |      |       |             |        |
| ----- | --- | -------- | --------------------------- | --- | --- | --- | --- | ----- | ---------- | ---- | ----- | ----------- | ------ |
|       | 1   | 1        | n n                         |     |     |     |     | Quick | Comparison | with | Prior | Approaches. | In the |
∑ E[x]=rep(K ,k,v) above, we only needed to retrieve a single ciphertext,
r i i
whichwasthedotproductresponse.Ontheotherhand,
x∈Ski
cuckoohashingrequiredretrievingtwoentries.Further-
where S ki is the random subset associated with key k. i more,theaboveapproachensuresthatthenumberofen-
Forconvenience,wecandenotethesetS usingavector triesisveryclosetolinear.Wehavealsoensuredminimal
k
| v suchthatv |     | [x]=1onlywhenx∈S |     |     | .Wecanre-write |     |     |                                                |     |     |     |     |     |
| ----------- | --- | ---------------- | --- | --- | -------------- | --- | --- | ---------------------------------------------- | --- | --- | --- | --- | --- |
| k           |     | k                |     |     | k              |     |     | errorgrowthasnoadditionalhomomorphicoperations |     |     |     |     |     |
theaboveequationasthedotproduct: areperformed.Thisshowsusthatweareheadinginthe
rightdirection,althoughtherearestillseveralproblems
|     |     | v ·E= | ∑ E[x]=rep(K |     | ,k  | ,v ). |     |                       |     |     |     |     |     |
| --- | --- | ----- | ------------ | --- | --- | ----- | --- | --------------------- | --- | --- | --- | --- | --- |
|     |     | ki    |              |     | r   | i i   |     | thatneedtoberesolved. |     |     |     |     |     |
x∈Ski
ToconstructtheencodeddatabaseE,wenotethatthe Problems with this Approach. While the above ap-
proachisverypromising,ithastwosignificantproblems
aboverepresentsnsystemsofequationsthatneedtobe
⊺
satisfied. We can view eachv to be the i-throw of a thatweoutlinebelowandsolveinlatersections:
ki
n×mmatrixM.Wealsodenotethevectorysuchthatthe
|                                    |     |     |     |     |                  |     |     | 1. Asdiscussedearlier,weconsideredasimplifiedset- |     |     |     |     |     |
| ---------------------------------- | --- | --- | --- | --- | ---------------- | --- | --- | ------------------------------------------------- | --- | --- | --- | --- | --- |
| i-thentryisthei-thvalue,y[i]=rep(K |     |     |     |     | ,k,v).Therefore, |     |     |                                                   |     |     |     |     |     |
|                                    |     |     |     |     | r                | i i |     | tingofPIRthatdoesnotutilizerecursion.Unfortu-     |     |     |     |     |     |
wemustfindavectorE∈Fmsatisfying:
nately,PIRwithoutrecursionendsuprequiringlinear
|    | ⊺   |    |     |    |       |       |    | communication. |     | Moststate-of-the-artPIR |     |     | schemes |
| --- | --- | --- | --- | --- | ----- | ----- | --- | -------------- | --- | ----------------------- | --- | --- | ------- |
|     | v   |     |     |     | rep(K | ,k ,v | )   |                |     |                         |     |     |         |
k ⊺1 r 1 1 utilize recursion to represent the database using a
|     | v   |              |     |     | rep(K | ,k ,v | )      |           |                                          |                 |     |        |           |
| --- | --- | ------------ | --- | --- | ----- | ----- | ------ | --------- | ---------------------------------------- | --------------- | --- | ------ | --------- |
|    | k2  |  ·E=M·E=y= |     |    |       | r 2   | 2  . |           |                                          |                 |     |        |           |
|    |     |              |     |    |       |       |        | hypercube |                                          | with dimensions | d   | ×...×d | such that |
|    | ... |             |     |    |       | ...   |       |           |                                          |                 | 1   |        | z         |
|     | ⊺   |              |     |     |       |       |        | d         | ···d ≥n.Inourscheme,wechoseeachrowvector |                 |     |        |           |
|     | v   |              |     |     | rep(K | ,k ,v | )      | 1         | z                                        |                 |     |        |           |
|     | kn  |              |     |     |       | r n   | n      |           |                                          |                 |     |        |           |
|     |     |              |     |     |       |       |        | v k       | to b e uniformly                         | random,which    |     | cannot | be repre- |
Inotherwords,wecangenerateacorrectencodingEof sentedinanysuccinctmannerusingrecursion.
thedatabaseD={(k ,v ),...,(k ,v )}aslongM·E= 2. Asecond,moresubtle,issueisthatcomputingasolu-
|     |     |     | 1 1 |     | n n |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
y has a solution, i.e. M has full row rank. Intuitively, tiontothesystemoflinearequationsM·E=yisvery
increasingthenumberofcolumnsinM,denotedbym, expensive.Thebestpossiblealgorithmforsolvinga
willalsoincreasetheprobabilitythatithasfullrowrank. generallinearsystemwouldalreadybequadraticin
However,thishastheaffectofalsoincreasingthesize thesizeofthedatabasen.

3.3 Partition-BasedKeywordPIR Partition
Atahighlevel,bothproblemsfromthepriorconstruction
resultedfromthefactthattherandommatrixMwasvery
densewithmanynon-zeroentries.Theexpectednumber
ofnon-zeroentrieswasnm/2,soitisnotsurprisingthat
findingasolutionwouldrequireatleastquadratictime.
Furthermore,as eachrow vectorhas,on average,m/2
non-zeroentries,itisnotsurprisingthatanyrepresenta-
EncodeP ,P ,P ,P onlyP shownhere
tionoftherowvectormustalsobelarge.Wepresentour 1 2 3 4 3
constructionSparsePIRthatdealswiththeissuesfrom  −1 
F(K ,k ||1) ... F(K ,k ||5) F(K ,k )||v
theprevioussectiontoobtainanefficientkeywordPIR. 2 4 2 4 r 4 4
High-LevelIdeaofPartitioning.Tosolvethisproblem e 3 =    F F ( ( K K 2 2 , , k k 6 8 | | | | 1 1 ) ) . . . . . . F F ( ( K K 2 2 , , k k 6 8 | | | | 5 5 ) )      F F ( ( K K r r , , k k 8 6 ) ) | | | | v v 8 6  
thatrowvectorshavehighdensity,wewillusetheidea F(K ,k ||1) ... F(K ,k ||5) F(K ,k )||v
2 9 2 9 r 9 9
ofpartitioningwhereweaimtobreakdowntheproblem
into smaller sub-problems. To do this,we will aim to FinalEncoding
embedtherandomvectorintoonlythefirstdimension
(cid:2) (cid:3)
usedin recursion. We willassume thatthe underlying E= e 1 e 2 e 3 e 4
PIRschemeusesrecursionandrepresentsthedatabase
asd 1 ×...×d z hypercube. Figure2: EncodingalgorithmforSparsePIRwithn=12,
At a high level,we will create b=(1+ε)n/d 1 par- b=4and(d 1 =5,d 2 =2,d 3 =2).
titionswhereeachpartwillbesized .Wewillchoose
1
thevalueofεlaterthroughexperimentation. On input Algorithm1SparsePIR.Initalgorithm
ofdatabaseD={(k ,v ),...,(k ,v )}thatwewishto
1 1 n n Input: 1λ:securityparameter.
encode,wewillassignthenkey-valuepairstothebparti-
Output: (ck,sk):clientandserverkey.
tionsuniformlyatrandom.Therefore,theexpectednum-
(ck,sk)←Π .Init(1λ)
berofkey-valuepairsineachpartisn/b=d /(1+ε)< PIR
1 return(ck,sk)
d . As a result,we have now reduced the problem to
1
efficientlyencodingdatabasesofsizeO(d ).
1
Within each part P, we will essentially repeat the
i
retrievedisv ·e.Therefore,thisreducestoperforming
construction from Section 3.2. Consider the i-th part 1 i
astandardPIRqueryoverab-entryarraywithrecursion
P ={(k ,v ),...,(k ,v )}.WewillgenerateM that
i 1 1 |Pi| |Pi| i
dimensionsd ×...×d .Weformallypresenttheabove
isa|P|×d matrixwhereeachentryisuniformlycho- 2 z
i 1
ideasinourSparsePIRconstructionbelow.
sen from {0,1}. In particular, the i-th row is gener-
⊺
atedrandomlyusingK andkeyk. Wecomputey = Encoding the Database. The encoding algorithm
2 i i
[rep(K ,k ,v) ... rep(K ,k ,v )].Finally,wesolve SparsePIRisformallypresentedinAlgorithm2thatuti-
r 1 i r |Pi| |Pi|
thelinearsystemM ·e =y andobtaintheencodinge lizesthepartitioningideasthatwerediscussedpreviously.
i i i i
forpartP. Our algorithms are also portrayed pictorially in Fig-
i
TobuildthefinalencodeddatabaseE,weputeachof ure2.RecallthatthealgorithmreceivesadatabaseD=
e 1 ,...,e b as column vectors in E to constructa d 1 ×b {(k 1 ,v 1 ),...,(k n ,v n )}asinputandmustoutputparame-
matrix.Note,thecosttosolvethelinearsystemineach tersfordecodingandanencodingofthedatabase.The
partisO(d
1
3)andthetotaltimeacrossallb=O(n/d
1
) encodingisparameterizedbyεwhereb=(1+ε)n/d
1
is
parts is O(n·d2). Forsmall values of d ,this is much thenumberofpartsinthepartitioning.
1 1
moreefficientthanapproachinSection3.2. First,thealgorithmsamplesthreehashkeys:K ,K
1 2
To query for a key k, the client will compute i = and K . The first key K will be used to generate the
r 1
F (K ,k) to determine the partition associated with k. randompartitioningofthedatabaseintobbinssuchthat
1 1
Next,theclientrandomlygeneratestheassociatedran- F (K ,·)∈{0,...,b−1}. The second key K will be
1 1 2
domvectorusinghashkeyK andkthatwedenotev . used to generate the random row vectors within each
2 1
Thiswillbethevectoruploadedforthefirstdimensionof partitionwhereF (K ,·)∈{0,1}.K isusedtogenerate
2 2 r
PIR.Supposethattheserverappliedv tothefirstdimen- representationsrep(K ,k,v)ofkey-valuepairs(k,v).
1 r
sionofE.Note,theresultis[v ·e ... v ·e ].Askis ForanyinputdatabaseD={(k ,v ),...,(k ,v )},the
1 1 1 b 1 1 n n
assignedtothei-thpartition,theonlyentryneededtobe nextstepistopartitionthenkey-valuepairsintotheb

Algorithm2SparsePIR.Encodealgorithm Algorithm4GenerateEncodealgorithm
Input: D={(k ,v ),...,(k ,v )}:database Input: (K ,K ,P):hashkeysandapart.
1 1 n n 2 r
Output: (prms,E):parametersandencoding Output: e:anencodingofthepart.
SamplehashkeysK ,K andK M←[]asemptyarray
1 2 r
b←(1+ε)n/d y←[]asemptyarray
1
m←(1+ε)n for(k,v)∈Pdo
P ←0/,...,P ←0/ AppendRandVector(K ,k)⊺ toMasrow.
1 b 2
fori=1,...,ndo ▷Partitiondatabase Appendrep(K ,k,v)toy.
r
j←F 1 (K 1 ,k i ) e←SolveLinearSystem(M,y)
P j+1 ←P j+1 ∪{(k i ,v i )} returne
for j=1,...,bdo
e ←GenerateEncode(K ,K ,P)
i 2 r j Algorithm5SparsePIR.Queryalgorithm
prms←(K ,K ,K )
1 2 r Input: (prms,ck,k):parametersandthequerykey.
E←[e ,...,e ]
1 b Output: (st,req):temporarystateandrequest.
return(prms,E)
b←(1+ε)n/d
1
Parseprms=(K ,K ,K )
1 2 r
Algorithm3RandVectoralgorithm v ←RandVector(K ,k)
1 2
Input: (K 2 ,k):thehashkeyanddatabasekey. j←F 1 (K 1 ,k)
Output: v :arandomlygeneratedvector. fori=2,...,zdo ▷Generateencodingof j
k
v k ←[0]d1 j i ←⌊j/⌈b/d i ⌉⌋
fori=1,...,d 1 do v i ←[0]di
v k [i]←F 2 (K 2 ,k||i) v i [j i +1]←1
b←⌈b/d⌉
returnv i
k
j← j modb
(st ,req)←Π .Query(ck,v ,...,v )
PIR PIR 1 z
return((st ,k),req)
entriesuniformlyatrandom.Inparticular,thekey-value PIR
pair(k,v)isplacedintotheF (K ,k)-thpartition.Let
i i 1 1 i
P ,...,P bethebpartsofthepartitioningsuchthateach
1 b
(k i ,v i )isassignedtoexactlyonepart.EachpartP i will and7.Wealsopresentadiagramofthedecodingprocess
produce an encoding e i and the final encoding E will in Figure 3. As input,the decoding algorithm will re-
combineallofe 1 ,...,e b together. ceiveparametersprms=(K 1 ,K 2 ,K r )aswellasaquery
Foreachparti∈[b],wewillusepartP i toconstruct key k∈K. Recall that the goal is to output z vectors
amatrixM i ,avectory i aswellasanencodinge i . We v 1 ,...,v z suchthatapplyingstandardPIRwithrecursion
iteratethroughthepartP i .Foreach(k,v)∈P i ,weappend schemesalongwithourencodeddatabaseEwoulden-
thebitvectorgeneratedby(F 2 (K 2 ,k||1),...,F 2 (K 2 ,k|| abledecodingthevalueassociatedwithquerykeyk(ifit
d 1 )) as a row vector in M i where each F 2 (K 2 ,k||·)∈ exists).
{0,1}.Additionally,weappendrep(K ,k,v)intoy.At
r i ThefirststepistocomputeF (K ,k)tocomputethe
1 1
theend,wenotethatM isavectorwith|P|rowsand
i i partitionassociatedwithk.Supposethatkwasassociated
d columns.Furthermore,y isavectorwith|P|entries.
1 i i withthei-thpart.ThenextstepistocomputeF (K ,k||
2 2
Next,wecomputee thatsatisfiesthefollowingequality
i x)forallx∈[d ]toobtaintherandombitvectoroflength
1
Me =y.Ifasolutiondoesn’texistforanyofthepart
i i i d . The above bitvectorwillbe v . Next,suppose we
1 1
i∈[b],theencodingalgorithmoutputs⊥andterminates.
appliedthefirstlayeroftheserver’sresponsealgorithmin
Afterdoingthisforallpartsi∈[b],weobtainthebpar-
standardPIRschemesthatutilizerecursion.Note,thatthe
tialencodingse ,...,e .Toobtainthefinalencoding,we
1 b resultis[v ·e ... v ·e ].Furthermore,asweknowthat
1 1 1 b
constructthetwo-dimensionalmatrixasE=[e ... e ]
1 b kwasassignedtothei-thpart,weonlyneedtoretrieve
where each e is a column vector. Note, this matrix
i thei-thcolumncontainingtheentryv ·e.Inotherwords,
1 i
has dimension d ×b. The output of the encoding is
1 the restofthe problem becomes a standardPIR query
prms=(K ,K ,K )andthematrixE.
1 2 r ofretrievingthei-thentryfromanarraywithbentries.
Decoding an Entry. Our decoding process of Query, Todothis,weencodetheindexiusingthestandardPIR
Answer and Decrypt are outlined in Algorithms 5, 6 queryoveradatabaseofdimensionsd ×...×d .
2 z

Algorithm6SparsePIR.Answeralgorithm Queryfork 6
Input: (prms,sk,E,req): parameters, server key, en-  
codeddatabasesandtherequest. F(K 2 ,k 6 ||1) (cid:20) 0 (cid:21) (cid:20) 1 (cid:21)
Output: resp:theresponsetotherequest.
v
k6
= ... ,
1
,
0
F(K ,k ||5)
2 6
resp←Π .Answer(sk,E,req)
PIR
returnresp FirstLevelofRecursionford =5
1
(cid:2) (cid:3)
v ·E→ v ·e ... v ·e
Algorithm7SparsePIR.Decryptalgorithm k6 k6 1 k6 4
Input: (prms,ck,st,resp): parameters,clientkey,tem-
SecondLevelofRecursionford =2
2
porarystateandresponse.
(cid:20) (cid:21) (cid:20) (cid:21)
Output: v:outputvalue 0 · v k6 ·e 1 v k6 ·e 2 → (cid:2) v ·e v ·e (cid:3)
Parseprmsas(K 1 ,K 2 ,K r ) 1 v k6 ·e 3 v k6 ·e 4 k6 3 k6 4
Parsestas(st ,k)
PIR
x←Π PIR .Decrypt(ck,st PIR ,resp) ThirdLevelofRecursionford 3 =2
Parsexas(id,v)
(cid:20) (cid:21) (cid:20) (cid:21)
1 v ·e
ifid=F(K r ,k)then 0 · v k6 ·e 3 →v k6 ·e 3 =F(K 2 ,k 6 )||v 6
returnv k6 4
else
Figure3: DecodingalgorithmforSparsePIRwithn=
return⊥
12,b=4anddimensions(d =5,d =2,d =2).The
1 2 3
serverapplicationispresentedinplaintextforclarity,but
therealconstructionwouldperformtheseoperationsover
Packing. Throughout this section, we assumed that
FHEciphertextshomomorphically.
rep(K ,k,v) fit into Z where α is the plaintext mod-
r α
ulus and each representation was stored in a single ci- goodK isfound,wefixit.Afterwards,weonlykeepre-
1
phertext. We show how to handle arbitrary length val- samplingK untilallrandomlygeneratedmatriceshave
2
ues and efficiency improvements using packing. We fullrowrank.
will represent rep(K,k,v) as a base-α string with L=
HandingDatabaseUpdates.Inmostapplications,the
⌈log (|rep(K,k,v)|)⌉ characters in Z . We repeat the
α α databaseDwillperiodicallychange.Ourencodingalgo-
above encoding algorithm foreach part P to obtain L
i rithmdoesnotenableincrementalupdatestoadd/remove
encodingse1,...,eL usingthesamematrixM.There-
i i i entries. Ifthe database changes,the encoding mustbe
sultingencodingis:E=[e1 ... eL ... e1 ... eL].
1 1 b b run again. To handle database updates, new encoded
If L>N,we encode each row of E using b·⌈L/N⌉
databases may be generated. As we will show in Sec-
plaintextpolynomialswhereeachtuple(e1,...,eL)uses
i i tion7,theencodingisconcretelyefficienttoenablecre-
⌈L/N⌉polynomials.WhenL≤N,wecanencode⌊N/L⌋
atingencodeddatabaseseveryfewminutes.
valuesintoasinglepolynomialmeaningeachrowofE
isencodedusing⌈b/⌊N/L⌋⌉polynomials.
3.4 EfficientlySolvableMatrices
Re-SamplingandOptimizations.Theencodingalgo-
rithm of SparsePIR fails (outputs ⊥) when any of the ForSparsePIR,wehavebeengeneratingrandommatri-
smallerlinearsystemsdoesn’thaveasolution.Inprac- cessuchthateachentryischosentobeuniformlyrandom
tice,ratherthanterminating,theencodingwouldsimply from {0,1}. The state-of-the-artalgorithm forsolving
re-samplenewhashkeysK andK andtrytoencode generallinearsystemsforpracticalsizesofnrequireat
1 2
thedatabaseagain.Inourimplementation,thiswillalso leastquadratic(and,typically,near-cubic)time.Thisis
bethecase.However,weshowthatonemayoptimize unsurprisingastheexpectednumberofnon-zeroentries
the re-sampling step. The naive approachwouldbe to inthesematricesisn2/2.
re-samplebothK andK ,whichisunnecessary. Tocircumventthisissue,SparsePIRutilizedpartition-
1 2
Recall that K partitions the n key-value pairs into ingtoguaranteethatthegeneratedlinearsystemswere
1
b parts. If any single part P receives too many items, verysmallinsize.RecallthateachoftheO(n/d )parts
i 1
theassociatedlinearsystemM ·e =y willnothavea involves solving a linear system of size O(d2),which
i i i 1
solution.Inthiscase,wedonotneedtoevengenerate takesO(d3)timeusingGaussianeliminationalgorithm.
1
M. Therefore,we try to sample K untilthe resulting As long as we chose d to be small, then the overall
i 1 1
partitiondoesnotover-assignitemstoanypart.Oncea encodingalgorithmrequiredO(n·d2)timethatwasrel-
1

Algorithm8RandVectoralgorithm Algorithm9SolveLinearSystemalgorithm
Input: (K 2 ,k):thehashkeyanddatabasekey. Input: (M,y):bandmatrixandvaluestosolvefor.
Output: v :arandomlygeneratedvector. Output: e:solutiontothelinearsystemM·e=y.
k
/*F outputselementsin{0,...,d−w}.*/ SortrowsofMandyaccordingtofirstnon-zeroentry
3
| p←F 3 (K 2 ,k||“position”) |     |     | ofM                              |     |     |     |     |     |     |
| -------------------------- | --- | --- | -------------------------------- | --- | --- | --- | --- | --- | --- |
| v ←[0]d1                   |     |     | ExecuteGaussianeliminationtogete |     |     |     |     |     |     |
k
fori=1,...,wdo
returne
| v [p+i+1]←F | (K ,k||i) |     |     |     |     |     |     |     |     |
| ----------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
| k           | 2 2       |     |     |     |     |     |     |     |     |
returnv
| k   |     |     | theefficiencyofourencodingalgorithmforlargerval- |                  |     |         |      |           |     |
| --- | --- | --- | ------------------------------------------------ | ---------------- | --- | ------- | ---- | --------- | --- |
|     |     |     | ues of d                                         | . In particular, |     | we only | need | to modify | the |
1
|                                 |     |                  | RandVector | and | SolveLinearSystem |     | sub-routines. |     | The |
| ------------------------------- | --- | ---------------- | ---------- | --- | ----------------- | --- | ------------- | --- | --- |
| ativelyefficient.Forexample,ifd |     | =O(logn),thenthe |            |     |                   |     |               |     |     |
1
overallencodingalgorithmwouldtakeonlyO(nlog2n) new sub-routines may be found in Algorithm 8 and 9.
Therestoftheencodingalgorithmremainsunchanged.
whichseemsreasonable.However,wecriticallyrequire
thatd mustbesmallastheencodingtimewouldgrow Decoding an Entry. We note that the decoding algo-
1
|     |     |     | rithm remains | identical |     | except | that we | utilize | the new |
| --- | --- | --- | ------------- | --------- | --- | ------ | ------- | ------- | ------- |
otherwise.
| Takingacloserlook,SparsePIRneverrequiredthatthe |     |     | RandVectoralgorithm. |     |     |     |     |     |     |
| ----------------------------------------------- | --- | --- | -------------------- | --- | --- | --- | --- | --- | --- |
randommatricesweregeneratedsuchthateachentrywas
|     |     |     | Encoding | Running | Time | Analysis. |     | Finally,we | show |
| --- | --- | --- | -------- | ------- | ---- | --------- | --- | ---------- | ---- |
chosenuniformlyatrandomfrom{0,1}.Infact,itonly thattheusageofrandombandmatricesimprovestherun-
requiredthelinearsystemassociatedwiththegenerated
ningtimeoftheencodingalgorithmandenablesusing
randommatrixtohaveasolutionwithhighprobability. large values of d . Previously,solving a linearsystem
1
We show thatthere are ways to generate suchrandom for each of the O(n/d ) parts took O(d3) using Gaus-
|     |     |     |     |     | 1   |     |     | 1   |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
matricesthatadmitefficientlysolvablelinearsystems.
sianelimination.Utilizingtherandombandmatrixcon-
SparseRandomMatrices. Itturnsoutthatcorealgo- structions,wecanreducethisdowntoO(d )(ignoring
1
constantfactorsofεfornow),whichmakesthetotalen-
rithmicproblemofgeneratingrandommatricessuchthat
theassociatedlinearsystemcanbesolvedefficientlyisa codingtimeonlyO(n).Asaresult,wecannowconsider
well-studiedarea(see[22,53,60,61,64]andreferences arbitraryvaluesofd astheencodingalgorithmwillnot
1
thereinassomeexamples).Forourwork,wewillutilize growsignificantlyasd 1 increasesinsize.
therandombandmatrixconstructionsofDietzfelbinger
andWalzer[28]thatsatisfytherequirementsneededby 4 ExtendingPartitionBasedKeywordPIR
ourkeywordPIRschemes.
Randombandmatricesareconstructedsuchthateach
|     |     |     | In the partition | basedkeywordPIR,eachofthe |     |     |     |     | n key- |
| --- | --- | --- | ---------------- | ------------------------- | --- | --- | --- | --- | ------ |
rowvectorwillconsistofashortbandoflengthw.The wordswasrandomlyassignedtooneofb=(1+ε)n/d
1
shortbandwillbeauniformlyrandomw-bitvector.All
|     |     |     | partitionsofsized |     | . Thisallowedustotreateachpar- |     |     |     |     |
| --- | --- | --- | ----------------- | --- | ------------------------------ | --- | --- | --- | --- |
1
entries outside of the band will be zero. The location titionasasmallindependentlinearsystemandsolveit
oftheshortbandischosenuniformlyatrandom.Diet-
separately,makingtheschemehighlyefficient.
zfelbingerandWalzer[28]showedthatformatricesof
|     |     |     | However,partition |     | basedkeywordPIR |     |     | suffers | from |
| --- | --- | --- | ----------------- | --- | --------------- | --- | --- | ------- | ---- |
dimension(1−γ)n×nforsomeconstant0<γ<1,if amajorproblem:atmostd keywordscanbeassigned
1
| the band parameter | is chosen | as w=O(logn/γ),then |             |            |      |       |      |       |           |
| ------------------ | --------- | ------------------- | ----------- | ---------- | ---- | ----- | ---- | ----- | --------- |
|                    |           |                     | to a single | partition. | This | means | that | ε has | to be big |
theresultingrandommatrixhasfullrowrankwithhigh enoughforthistohappenwithhighprobability.Section7
probability. To solve the associatedlinearsystem,one showsthatforn=220andd
1 =128,εhastobeatleast
willfirstsorttherowsbythestartinglocationofthenon-
0.38.Throughouttherestofthissectionandthenextone,
zerobandoflengthw.Next,onecanemployGaussian we will consider the example of d =128 as all prior
1
| eliminationinaveryefficientmanner. |     | Atahighlevel, |     |     |     |     |     |     |     |
| ---------------------------------- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- |
concretelyefficientPIRschemes(including[8,9,51,54])
Gaussian elimination only needs to consider columns used ≥128. Itturnsoutthatsmallervaluesofd are
|     |     |     | 1   |     |     |     |     |     | 1   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
withinthew-lengthbandforeachrow.Therefore,Gaus-
|     |     |     | more difficult | (see | Section | 7). | Therefore, | considering |     |
| --- | --- | --- | -------------- | ---- | ------- | --- | ---------- | ----------- | --- |
sianeliminationrequiresonlyO(n/γ2)time.InFigure4,
d =128iseffectivelythemostdifficultsetting.
1
wepresentadiagramdepictingrandombandmatrices Lookingback,thetwomotivationsbehindthepartition
andtheGaussianeliminationalgorithm.
|     |     |     | based keyword | PIR | were | the | inefficiency | of  | solving a |
| --- | --- | --- | ------------- | --- | ---- | --- | ------------ | --- | --------- |
Encoding the Database. We modifyourconstruction largerandomlinearsystemandrecursionincompatibility
SparsePIRtoutilizerandombandmatricestoimprove ofthegeneratedrandomvectors.Thefirstissuecanbe

Figure 4: The left matrix is a random band matrix with 4 rows and band width w=5. The middle matrix is the
intermediatestepoftheGaussianeliminationthatsortsrowsbybandlocation.ThefinalmatrixshowsthatGaussian
eliminationonlyconsidersasmallsubsetofentries.
Figure 6: Modified construction from Figure 5. Vari-
ablesinpartition1arereversedandarecoloredpurple.
TheredbandinFigure5isrecursioncompatibleafter
transformingtothegreenband.
recursionPIRprotocol.
Figure5: Atwodimensionalrepresentationofadatabase
RecallthatpartitionbasedkeywordPIRallowedusto
of9elementsandanexampleLHSofthecombinedlinear
vieweachpartitionasanindependentlinearsystem.An-
system.Eachcolumnofthedatabasecorrespondstoa
otherequivalentwaytoformulatethisistocombinethese
partition.Thegraybandsallliestrictlyinsideapartition
independentlinearsystemsandformasinglesystemas
andarerecursioncompatible,whiletheredbandspans
inFigure5.Pictorially,eachpartitioncorrespondstoa
partition1andpartition2andisrecursionincompatible.
contiguousrangeofd variables(andcolumnsinthema-
1
solvedbyutilizingtherandombandmatrixconstructions
trix).InFigure5,noticethatthegraybandsallliestrictly
ofDietzfelbingerandWalzer[28],whichallowustocon- inside a partition and do not span multiple partitions -
structlargelinearsystemsthatcanbesolvedefficiently. thisisthecrucialpropertythatallowedtherowvectorsto
Thus,if we can somehow construct random band ma- berecursioncompatibleinthepartitionbasedkeyword
tricesthatarealsorecursioncompatible,weshouldbe
PIR. Ontheotherhand,theredbandspanspartition1
abletoreducetheεinthepartitionbasedkeywordPIR andpartition2andisnotrecursioncompatible.However,
whilestillkeepingtheschemeefficient.Inthissection, abandspanningmultiplepartitionsisnotreallyfunda-
we present our extended construction SparsePIRg that mentallyproblematic;itisjustthatsuchrandombandis
achievesthisgoal. morelikelytoberecursionincompatiblethannot.The
Warmup: Two DimensionalRecursion. To motivate mainideabehindthenewconstructionistoartificially
ourconstruction,wewillstartoffwithconstructingrecur- modifythesespanningbandstoberecursioncompatible,
sioncompatiblebandmatricesforthetwodimensional withthehopethatthenewbandmatricesremain"random

ExtendingtoHigherDimensionsWewillstartbyfo-
|     |     |     |     | cusing | on  | hypercube | representation |     | in  | the form | d × |
| --- | --- | --- | --- | ------ | --- | --------- | -------------- | --- | --- | -------- | --- |
1
d×···×d,i.e.allsubsequentdimensionsafterthefirst
(cid:124) (cid:123)(cid:122) (cid:125)
z
areequal.
InthecontextofPIR,oneusefulwayofvisualizing
|     |     |     |     | a   | hypercube | ofthis | form is | as a | d ×dz | matrix,where |     |
| --- | --- | --- | --- | --- | --------- | ------ | ------- | ---- | ----- | ------------ | --- |
1
eachcolumncorrespondstoafirstdimensionslice(of
|     |     |     |     | size | d ) | and the column | indices |     | correspond |     | to base d |
| --- | --- | --- | --- | ---- | --- | -------------- | ------- | --- | ---------- | --- | --------- |
1
|     |     |     |     | representations |        | ofthe  | "coordinates" |        | ofthe   | firstdimen- |           |
| --- | --- | --- | --- | --------------- | ------ | ------ | ------------- | ------ | ------- | ----------- | --------- |
|     |     |     |     | sion            | slices | in the | dz space.     | In the | context | of          | partition |
basedkeywordPIR,eachcolumncanbeviewedasapar-
tition(numberedfrom0todz−1).Selectingpartitioni
isequivalenttotransformingitoitsbasedrepresentation
|                                                 |     |     |     | andnaturallymappingthe |     |     |                 | jthdigittothecorresponding |     |     |     |
| ----------------------------------------------- | --- | --- | --- | ---------------------- | --- | --- | --------------- | -------------------------- | --- | --- | --- |
| Figure7: Modifiedbandconstructionsforahypercube |     |     |     |                        |     |     |                 |                            |     |     |     |
|                                                 |     |     |     | indicatorvectorforthe  |     |     | j+1thdimension. |                            |     |     |     |
representationof3×2×2afterpermutingthepartitions
Inthetwodimensionalrecursioncase,forbandsthat
| accordingtotheGraycodeof01         |     | ,00 ,10 ,and11 | .The |                                                   |     |     |     |     |     |     |     |
| ---------------------------------- | --- | -------------- | ---- | ------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
|                                    |     | 2 2 2          | 2    | spannedtwopartitions,wehadtoconstructtheseconddi- |     |     |     |     |     |     |     |
| greenbandisnowrecursioncompatible. |     | Notethatwe     |      |                                                   |     |     |     |     |     |     |     |
mensionvectortoindicatethatwewantedtoselectthose
| mustreversetheorderofthevariablesinpartition00 |     |     | 2   |     |     |     |     |     |     |     |     |
| ---------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
twopartitions.Thiscouldbedoneinastraightforward
| (andnotinpartition01 | )asthisisnowtheodd-indexed |     |     |                                                 |     |     |     |     |     |     |     |
| -------------------- | -------------------------- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- | --- | --- | --- |
|                      | 2                          |     |     | mannerbyturningonbitsatpositionscorrespondingto |     |     |     |     |     |     |     |
partition.
thepartitionnumbersinthequeryvector.
However,suchnaiveapproachnolongerworksifwe
enough"andstillhasasolutionwithhighprobability. moveontohigherdimensions.Forexample,supposewe
embedthedatabaseinFigure5andFigure6ontoahyper-
Beforeproceedingfurther,wewillfirstpermutethe
cubeofdimensions3×2×2.Thepartitions(columns)
variablesinthelinearsystembyreversingtheorderofthe
|                                                   |     |     |     | are | numbered | 00  | , 01 , 10 | , 11 | in base | two | (the last |
| ------------------------------------------------- | --- | --- | --- | --- | -------- | --- | --------- | ---- | ------- | --- | --------- |
| variablesthatcorrespondtooddindexedpartitionsasin |     |     |     |     |          | 2   | 2         | 2 2  |         |     |           |
partitionwillcorrespondtoemptyentriesinthiscase).
Figure6.Themotivationbehindthischangewillbecome
|     |     |     |     | Thegreenbandspanspartition01 |     |     |     |     | 2 andpartition10 |     | 2 ,but |
| --- | --- | --- | --- | ---------------------------- | --- | --- | --- | --- | ---------------- | --- | ------ |
apparentlateron.Now,insteadofchoosingapartition
wecannolongerconstructrecursioncompatiblequery
andsamplingthebandwithinthechosenpartition,we
vectors(unlikethetwodimensionalcase).
insteadsamplethebandacrosstheentirerow,allowing
Togetaroundthis,wemakethefollowingobservation:
thebandtospantwopartitions(westillkeeptheband
widthw<d ,whichmeansthatabandcanspanatmost wecanconstructrecursioncompatiblequeryvectorsfor
1
selectingtwopartitionsaslongastheHammingdistance
twopartitions).Ifthisbandhappenstoliestrictlyinside
|     |     |     |     | between |     | the base | d representation |     | ofthe | two | partition |
| --- | --- | --- | --- | ------- | --- | -------- | ---------------- | --- | ----- | --- | --------- |
apartition,wearedone.Ontheotherhand,supposethat
|                                   |                                    |                    |     | numbers                       |         | is 1. Indeed,we |     | can observe |       | that there  | is no   |
| --------------------------------- | ---------------------------------- | ------------------ | --- | ----------------------------- | ------- | --------------- | --- | ----------- | ----- | ----------- | ------- |
| thebandspanstwopartitions.Denotev |                                    | 1 ||v 2 asthespan- |     |                               |         |                 |     |             |       |             |         |
|                                   |                                    |                    |     | problemofselectingpartition00 |         |                 |     |             | and01 | -simplysend |         |
| ningbandwherev                    | liesstrictlyinthefirstpartitionand |                    |     |                               |         |                 |     | 2           |       | 2           |         |
|                                   | 1                                  |                    |     |                               | (cid:2) | (cid:3)         |     |             |       | (cid:2)     | (cid:3) |
v lies strictly in the second partition. We modify the vector 10 fortheseconddimensionand 11 forthe
2
thirddimension.Similarly,ifwewanttoselectpartition
| bandbydiscardingv                                | 2 andappendingthereverseofv |     | 1 : |              |     |         |                       |        |                 |         |        |
| ------------------------------------------------ | --------------------------- | --- | --- | ------------ | --- | ------- | --------------------- | ------ | --------------- | ------- | ------ |
|                                                  |                             |     |     | 10           | and | 11 , we | can send              | vector | (cid:2) (cid:3) | for the | second |
| v ||rev(v )whererevdenotesthereversefunction.Ob- |                             |     |     |              | 2   | 2       |                       |        | 01              |         |        |
| 1 1                                              |                             |     |     |              |     | (cid:2) | (cid:3)               |        |                 |         |        |
|                                                  |                             |     |     | dimensionand |     | 11      | forthethirddimension. |        |                 |         |        |
servethatwiththistransformation,therowvectornow
becomesrecursioncompatibleasillustratedbythegreen Infact,wecanseethatthisobservationisapplicableto
bandinFigure6.Toqueryforthegreenband,theclient anarbitraryhypercubestructureofdimensionsd ×d ×
|     |     |     |     |     |     |     |     |     |     |     | 1 2 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
(cid:2) (cid:3) ···×d .Ifwerepresentthepartitionnumbersinmixed
| cansendvector | 010 forthefirstdimensionandvector |     |     |     | z   |     |     |     |     |     |     |
| ------------- | --------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
(cid:2) (cid:3)
011 fortheseconddimension.Thewidthofthenew bases(d 2 ,···,d z ),thenwecanselectthetwopartitionsas
bandisupperboundedby2w,sotheefficiencyofsolving longastheHammingdistancebetweenthetwopartition
| thelinearsystemremainsunchangedasymptotically.Fi- |     |     |     | numbersis1. |     |     |     |     |     |     |     |
| ------------------------------------------------- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- | --- | --- |
nally,becausethenewbandvectorgenerationalgorithm Thismotivatesthefollowingconstruction:inthelin-
onlydependsonthekeywordandthedatabaseparame- earsystem,permute the orderof the variables in such
ters,theclientcanreplicatethisproceduretoconstruct a way that adjacent partition numbers have Hamming
PIRqueryforanarbitrarykeyword. distances1.Thisway,allbandsthatspantwoadjacent

partitionsinthelinearsystemwillberecursioncompati- bitswhereU israngeofF(K ,k).Forn=220,d =128,
r i 1
ble.Inparticular,wecanusemixed-radixGraycode[43] |U|=264,thenthisrequires64KB.
toconstructthispermutation.Afterpermutingthevari- TruncationofBoundaries.Forournextimprovement,
ablesaccordingtotheGraycode,wehavetoreversethe wenotethatitisnotnecessarytostorethefullboundary
orderofoddindexedpartitionvariablesasinthetwodi- value. Instead,we can setmostofthe leastsignificant
mensionalcasetoensurethatthefirstdimensionremains bitsofboundaryvaluestozeroandtruncatethemsince
recursioncompatible.Figure7showsanexamplecon- we needB only to be precise enoughto distinguishit
i
struction.Notethatthelastpartition11 2 canbeignored from the neighboring keyword hash value to maintain
whilesolvingthelinearsystem,andwillcorrespondto exactpartitioning.ThiscanreducethelogU bitsusedto
emptyentriesinthedatabase.Wepointoutthatthiscom- representeachboundary.Forexample,whenn=220and
positionofpermutationsdoesnotrequireextrastorage d =128,weseethatapproximately25bitsarenecessary
1
ontheclientside,ascomputingthenthGraycodecanbe throughexperimentation(seethefullversionformore
doneefficientlyontheflywithoutrequiringextraspace. details).Fortheconcreteexampleofn=220 andd =
1
OurexperimentalevaluationshowsthatSparsePIRgis
128,thisreducesclientstorageby60%to25KB.
abletoreduceεandispracticallyefficient.Forexample,
CompressionforPersistentStorage.Furthermore,we
forn=220 andd =128,SparsePIRg isabletoreduce
1 showthatonecanfurthercompresstheboundaryvalues
εfrom0.38to0.1,asignificantimprovementovernaive
whenstoringinpersistentstorage.Inparticular,wecan
partitionbasedkeywordPIR.
storethedifferencesbetweenboundariesasopposedto
theboundariesthemselves.Additionally,wecanapply
5 KeywordPIRwithClientStorage standardcompressiontechniquesforvariablelengthen-
codings.Thiscanreducetheclientstoragetolessthan
InSparsePIR,ifd 1 +1itemswereassignedtoanysingle 11.2KBforn=220andd 1 =128.However,wenotethat
part then the associated linear system would not have thiscompressionmustbedecompressedtobeabletohan-
asolution. Inpractice,weneededslightlylessthand 1 dlequeries(sotheclientmustuse25KBoftemporary
itemstomakesurethesolutionexistswithhighproba- storagewhenperformingqueries).
bilitythusnotrequiringtoomuchresampling.Wewould
DiscussionaboutClientStorageSize.Whiletheclient
haveexactpartitioningifallthepartshaveslightlyless storageofSparsePIRcconsistsofO(n/d )boundaryval-
thand items.WepresentSparsePIRc,anenhancement 1
1 ues,thepracticalclientstorageisverysmall.Inthecon-
of SparsePIR with client-side storage,achieving exact creteexampleaboveofn=220andd =128,theresult-
1
partitioningandthusminimalencodeddatabasesizeat
ingclientstorageis≈11KB.Ifweconsiderapractical
thecostofstoringmoderateamountsofextrainformation
database size used in prior works (such as [8–10]) of
inthedecodingparameter.Weprovidemoredetailedal- n=220256-byteentries,wenotethattheclientstorage
gorithmsandjustificationforthechoiceofclientstorage
isequivalentto0.004%ofthetotaldatabasesize(or43
inthefullversionofthispaper.
entries).Furthermore,theclientstorageisindependentof
ExplicitExactPartitioning.Oneveryexpensiveway thesizeofeachentry.Forlargerentries,theclientstorage
of exactly partitioning is for the client to receive and isanevensmallerpercentageofthedatabase. Further-
explicitlystoretheassignmentofnitemstothen/d parts more,theclientstorageissmallerforlargervaluesofd .
1 1
requiringnlog(n/d )bits.Whenn=220 andd =128, Forexample,n=220 andd =1024 requires only1.8
1 1 1
thiswouldrequire1.6MB,whichisquitelarge. KBofclientstorageamountingto0.0006%ofthetotal
ExactPartitioningviaBoundaries.Ourmainideaisto database(or7entries).
firstsortthehashedoutputsofthenkeywordsandthen ComparisonwithSparsePIRgandSparsePIR.Wenote
evenlysplittheminton/d parts.Theclientstoresthe that SparsePIRc and SparsePIRg obtain near-optimal
1
n/d partition boundary values that separate the parts, database sizes in different ways. SparsePIRg requires
1
B ,B ,...,B ,B . All items in the i-th part are largeencodingtimestocreatethedatabase.Incontrast,
0 1 n/d1 n/d1+1
containedintheinterval[B ,B).Forconvenience,we SparsePIRcrequireslong-termclientstorage.Iflargeen-
i−1 i
willassumeB tobethesmallestpossiblehashoutputand codingtimesaretolerable(i.e.,databasesmaybecreated
0
B tobethelargesthashoutput.ThelistBbecomes in the background before serving),one should choose
n/d1+1
partofprmsatsetuptime.Duringquerytime,theclient SparsePIRg. If long-term client storage is reasonable,
hashesthekeykandfindstheinterval[B ,B)contain- thenonecanchooseSparsePIRc.Ifneitherslowencod-
i−1 i
ing the hashoutput. Then,the clientdetermines key k ingtimesorlong-termclientstorageareacceptable,one
belongstothei-thpart.ThisonlyrequiresO(n/d logU) can simply use SparsePIR instead. See Section 7 for
1

experimentalcomparisonsbetweenthethreeoptions. cannotbeallocatedaccordingtocuckoohashing,thenthe
client’squerywillfail.Angeletal.[9]suggestedpicking
|     |     |     |     |     |     |     |     | t=3andb | CH =1.5ℓusingtheexperimentalevaluation |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------- | -------------------------------------- | --- | --- |
6 BatchKeywordPIR
ofpriorworks[19,59]showingthattheconcretefailure
probabilityisverysmall.
Inthissection,weshowthatourtechniquescanbeex-
Thisapproachachievesmuchbettercomputationalef-
tendedtobatchsettingswhereaclientwishestoquery
ficiencycomparedtothetrivialapproachofperforming
abatchofℓ≥1queriesatonetime.Clearly,thereisa
|     |     |     |     |     |     |     |     | ℓkeywordPIRqueries. |     | Thetrivialapproachwouldre- |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------- | --- | -------------------------- | --- |
trivialapproachofperformingℓindependentPIRqueries,
quireperformingO(n·ℓ)overheadaseachkeywordPIR
butthisendsupbeingveryinefficient.Thegoalinthe
querywouldrequirelinearoverhead.Ontheotherhand,
batchsettingistodesigntechniquesthatenableperform-
thisframeworkrequiresonlyO(n)servercomputation
ingbatchqueriesmoreefficiently.
asonlyonekeywordPIRqueryisperformedperbucket
| Revisiting |     | State-of-the-Art |     | Batch | PIR. | To date, | the |     |     |     |     |
| ---------- | --- | ---------------- | --- | ----- | ---- | -------- | --- | --- | --- | --- | --- |
andthetotalbucketsizeis3n.
mostpracticallyefficientapproachtoconstructingbatch
PriorApproachestounderlyingKeywordPIR.The
PIRschemesarisesfromtheframeworkintroducedby
aboveframeworkessentiallyreducesabatchPIRquery
Angeletal.[9]thatutilizescuckoohashingtoencode
forℓkeysinto1.5ℓkeywordPIRqueriesinto1.5ℓbuck-
| both | the database | and | queries. |     | We note | that the | origi- |                        |     |                        |     |
| ---- | ------------ | --- | -------- | --- | ------- | -------- | ------ | ---------------------- | --- | ---------------------- | --- |
|      |              |     |          |     |         |          |        | etswhosetotalsizeis3n. |     | Wenotethattheframework |     |
nalframeworkwasdesignedfordoingbatchPIRover
requireskeywordPIRevenifdatabasewasanarrayand
ann-entryarray.Theframeworkdidnotconsiderbatch
|                                |     |     |     |     |                |     |     | thedatabase’skeyswerek |     | =1,...,k | =n.Eachofthe |
| ------------------------------ | --- | --- | --- | --- | -------------- | --- | --- | ---------------------- | --- | -------- | ------------ |
| keywordPIRoversparsedatabases. |     |     |     |     | However,wewill |     |     |                        |     | 1        | n            |
1.5ℓbucketsisadatabasewhosekeysareasubsetof[n].
presenttheframeworkwithrespecttothekeywordset-
Toretrievethevalueassociatedwithanykeyq∈[n]from
| ting | and point | out | where | the framework |     | is inefficient |     |     |     |     |     |
| ---- | --------- | --- | ----- | ------------- | --- | -------------- | --- | --- | --- | --- | --- |
abucket,onemustusekeywordPIR.
whenqueryingsparsedatabases.
keywordPIR,Angeletal.
At a high level, the servers utilizes t random To perform [9] proposed
severaldifferentoptions.Thesimplestapproachwasfor
| hash | functions   | H ,...,H | t                   | to encode | a   | database  | D = |            |             |          |                       |
| ---- | ----------- | -------- | ------------------- | --------- | --- | --------- | --- | ---------- | ----------- | -------- | --------------------- |
|      |             | 1        |                     |           |     |           |     | the client | to download | a direct | mapping from each ar- |
| {(k  | ,v ),...,(k | ,v       | )}.Weassumethatthet |           |     | hashfunc- |     |            |             |          |                       |
|      | 1 1         | n n      |                     |           |     |           |     |            |             |          |                       |
rayentrytothephysicalindexwithineachbucket(ora
tionsaresharedbetweenallclientsandtheserver.The
|     |     |     |     |     |     |     |     | compressed | version | using Bloom | filter or similar data |
| --- | --- | --- | --- | --- | --- | --- | --- | ---------- | ------- | ----------- | ---------------------- |
serverwillreplicateeachentryinthedatabasettimesand
structures).Theseapproacheswouldrequirelinearstor-
| assignthemintob |     |     | bucketsasfollows.Foreach(k,v), |     |     |     |     |     |     |     |     |
| --------------- | --- | --- | ------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- |
|                 |     | CH  |                                |     |     |     | i i |     |     |     |     |
ageontheclientsidethatmaybeexpensive.Furthermore,
| theserverassignsthe(k,v)tothet |     |     | i   | i   | bucketsaccording |     |     |     |     |     |     |
| ------------------------------ | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- |
fordatabaseswithsmallvalues,thesemapsmaybeal-
| tothet | hashfunctionevaluationsH |     |     |     | (k),...,H(k).The |     |     |     |     |     |     |
| ------ | ------------------------ | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- |
|        |                          |     |     |     | 1 i              | t i |     |     |     |     |     |
resultingencodingconsistsofatotalt·nkey-valuepairs mostaslargeasthedatabase. Instead,Angeletal.[9]
proposedaveryclevertrickthatenablesaclienttode-
| acrossb |     | buckets. |     |     |     |     |     |                                 |     |     |                   |
| ------- | --- | -------- | --- | --- | --- | --- | --- | ------------------------------- | --- | --- | ----------------- |
|         | CH  |          |     |     |     |     |     | rivethemappingusingtheoriginalt |     |     | hashfunctionsthat |
Forthequeryalgorithm,supposetheclientreceives
|         |     |             |     |             | )∈Kℓ. |            |     | aresharedbetweentheserverandtheclient.Theclient |     |     |     |
| ------- | --- | ----------- | --- | ----------- | ----- | ---------- | --- | ----------------------------------------------- | --- | --- | --- |
| a batch | of  | ℓ≥1 queries |     | (q 1 ,...,q | ℓ     | For conve- |     |                                                 |     |     |     |
cansimplyrepeattheserver’sallocationprocesstocom-
nience,itistypicallyassumedthatℓqueriesaredistinct.
putethemapping.Therefore,theclientcanre-createthe
| If not, | the | client may | de-duplicate |     | locally, | perform | a   |     |     |     |     |
| ------- | --- | ---------- | ------------ | --- | -------- | ------- | --- | --- | --- | --- | --- |
mappingwithoutlargelong-termstorage.
batchPIRqueryforadistinctsetofkeysandthenrepli-
catetheblocksasneededtoproducethecorrectanswer. Going backto batchkeywordPIR,we note thatthe
abovetrickoftheclientrepeatingtheserver’sallocation
| The | clientuses | thet | hashfunctions,H |     | 1   | ,...,H,to t | per- |     |     |     |     |
| --- | ---------- | ---- | --------------- | --- | --- | ----------- | ---- | --- | --- | --- | --- |
processcannotbeused.Thekeyobservationisthat,for
| formcuckoohashingthatallocateseachq |     |     |     |     |     | intoexactly |     |     |     |     |     |
| ----------------------------------- | --- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- |
i
oneoftheb bucketsspecifiedbythet hashfunctions standardPIR,theclientknowsthatthedatabase’skeys
CH
|                                                |                                |     |     |     |     |        |     | areexactlyk                                         | =1,...,k | =n.Therefore,theclientcanre- |     |
| ---------------------------------------------- | ------------------------------ | --- | --- | --- | --- | ------ | --- | --------------------------------------------------- | -------- | ---------------------------- | --- |
| H (q),...,H(q).Cuckoohashingguaranteesthateach |                                |     |     |     |     |        |     |                                                     | 1        | n                            |     |
| 1                                              | i                              | t i |     |     |     |        |     | peattheserver’sallocationprocessidentically.Whenthe |          |                              |     |
| oftheb                                         | bucketsisassignedatmostoneof{q |     |     |     |     | ,...,q | }.  |                                                     |          |                              |     |
|                                                | CH                             |     |     |     |     | 1      | ℓ   |                                                     |          |                              |     |
database’skeysarefromasparseuniverse,theclientno
| Finally,theclientperformsb |     |     |     | CH keywordPIRqueriesinto |     |     |     |     |     |     |     |
| -------------------------- | --- | --- | --- | ------------------------ | --- | --- | --- | --- | --- | --- | --- |
longerknowstheexactkeysthatappearinthedatabase.
| eachofthe |     | b buckets | to  | retrieve | the | assignedquery |     |     |     |     |     |
| --------- | --- | --------- | --- | -------- | --- | ------------- | --- | --- | --- | --- | --- |
CH
key(oranarbitraryindexifnoquerykeywasassigned). Therefore,theclientcannotcomputethemappingwith-
outknowingthekeyspresentinthedatabase.
WefurtherdiscussthenecessityofkeywordPIRlater.
The last step is to pick concrete parameters of the ReplacingtheKeywordPIR.Toconstructbatchkey-
cuckoo hashing scheme: t and b CH . This is an impor- wordPIRschemes,itseemslikewemustuseatruekey-
tantstepasbadparameterscancausealargeportionof wordPIRasthetrickofsuccinctlygenerateclientmaps
queriestofail.Note,ifasetofℓquerykeys{q ,...,q } cannolongerbeused.Priortoourwork,previouskey-
|     |     |     |     |     |     | 1   | ℓ   |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

|     |     | Client | Client | Resp. |     |     |     | Computationtimeonε,256Bperelement |     |     |     |     |     |
| --- | --- | ------ | ------ | ----- | --- | --- | --- | --------------------------------- | --- | --- | --- | --- | --- |
Keyword?
|     |     | Storage | Time | Size |     |     |     |     |     |     |     |     |     |
| --- | --- | ------- | ---- | ---- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
80
| 3-WayCH[9] |     | O(1) | O(n) | 1.5ℓ |     | ×   |     |                      |     |     |     |        |     |
| ---------- | --- | ---- | ---- | ---- | --- | --- | --- | -------------------- | --- | --- | --- | ------ | --- |
|            |     |      |      |      |     |     |     | sdnocesnievlosotemiT |     |     |     | d =128 |     |
| 3-WayCH[9] |     | O(n) | O(1) | 1.5ℓ |     | ✓   |     | 70                   |     |     |     | 1      |     |
d
|                                               |     |      |      |      |     | ✓   |     |     |     |     |     | 1 =512  |     |
| --------------------------------------------- | --- | ---- | ---- | ---- | --- | --- | --- | --- | --- | --- | --- | ------- | --- |
| 3-Way+BatchCH                                 |     | O(1) | O(1) | 3ℓ   |     |     |     | 60  |     |     |     |         |     |
|                                               |     |      |      |      |     |     |     |     |     |     |     | d =1024 |     |
| SparseBatchPIR                                |     | O(1) | O(1) | 1.5ℓ |     | ✓   |     | 50  |     |     |     | 1       |     |
| Figure8:Comparisonofbatch(keyword)PIRschemes. |     |      |      |      |     |     |     | 40  |     |     |     |         |     |
30
wordPIRschemesincurredsignificantadditionalclient
20
storage,doublingresponsesizesormultipleroundtrips
| comparedtostandardPIR.                     |        |               |      |           |       |      |     | 10  |       |     |     |     |     |
| ------------------------------------------ | ------ | ------------- | ---- | --------- | ----- | ---- | --- | --- | ----- | --- | --- | --- | --- |
| WeshowthatourSparsePIRfamiliesofkeywordPIR |        |               |      |           |       |      |     | 0   |       |     |     |     |     |
|                                            |        |               |      |           |       |      |     |     | 0 0.2 | 0.4 | 0.6 | 0.8 | 1   |
| schemes                                    | enable | significantly | more | efficient | batch | key- |     |     |       |     |     |     |     |
ε
wordPIRschemes.WeobtainSparseBatchPIRbyplug-
Computationtimeonε,16KBperelement
| ging ourkeywordPIR |               |     | schemes       | into the | frameworkof  |     |     |     |     |     |     |     |     |
| ------------------ | ------------- | --- | ------------- | -------- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
| Angel et           | al. [9],whose |     | communication |          | is identical | to  |     | 400 |     |     |     |     |     |
batchPIRschemesovern-entryarrays.Thisisa2xim- sdnocesnievlosotemiT d 1 =128
| provementoverusinganypriorkeywordPIRschemethat |     |     |     |     |     |     |     | 350 |     |     |     | d =512 |     |
| ---------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------ | --- |
1
| doesnotrequirelong-termclientstorageoflinear-sized |     |     |     |     |     |     |     |     |     |     |     | d 1 =1024 |     |
| -------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------- | --- |
300
mappings.Wepresentacomparisonofbatch(keyword)
PIRschemesinFigure8.InSection7,weexperimentally
250
evaluateournewbatchkeywordPIRschemesshowing
significantimprovements.
200
| 7 ExperimentalEvaluation |     |     |     |     |     |     |     | 150 |       |     |     |     |     |
| ------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | ----- | --- | --- | --- | --- |
|                          |     |     |     |     |     |     |     |     | 0 0.2 | 0.4 | 0.6 | 0.8 | 1   |
ε
7.1 Implementation
|                |     |            |     |            |     |     | Figure9: |        | ComputationtimetosolvetheSparsePIRlin- |              |     |               |       |
| -------------- | --- | ---------- | --- | ---------- | --- | --- | -------- | ------ | -------------------------------------- | ------------ | --- | ------------- | ----- |
| We implemented |     | SparsePIR, |     | SparsePIRg |     | and |          |        |                                        |              |     |               |       |
|                |     |            |     |            |     |     | ear      | system | vs ε on                                | 220 elements |     | for dimension | sizes |
SparsePIRc
in C++ on top of several open source d =128,512 and1024. The graphs suggestthatthe ε
1
PIRschemes:OnionPIR[3]andSpiral[5].Tocompare,
|     |     |     |     |     |     |     | lower | bounds | for d | =128,512 | and | 1024 | are roughly |
| --- | --- | --- | --- | --- | --- | --- | ----- | ------ | ----- | -------- | --- | ---- | ----------- |
1
we also implement the keyword PIR cuckoo hashing 0.38,0.18,and0.13respectively.Wechosebandparam-
| framework | of  | Ali et al. | [8] on | top of | the same | PIR |     |     |     |     |     |     |     |
| --------- | --- | ---------- | ------ | ------ | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
eterw∈[50,60].
schemes.WealsoimplementSparseBatchPIRusingthe
frameworkofAngeletal.[9].Intotal,ourimplementa-
| tions requiredaround2000 |     |     | lines | ofcodes. | We  | plan to |     |     |     |     |     |     |     |
| ------------------------ | --- | --- | ----- | -------- | --- | ------- | --- | --- | --- | --- | --- | --- | --- |
opensourceourimplementationsinthenearfuture. worksusestrictlylessthan2nentries.
FHEParameterSelection.OurSparsePIRfamiliesof
frameworkswereconstructedcarefullytoensuremini-
|          |         |      |           |     |          |         | Experimental |     | Setup.  | Our    | experiments |        | are conducted |
| -------- | ------- | ---- | --------- | --- | -------- | ------- | ------------ | --- | ------- | ------ | ----------- | ------ | ------------- |
| malnoise | growth. | As a | result,we | can | directly | use the |              |     |         |        |             |        |               |
|          |         |      |           |     |          |         | withmachines |     | thatare | Ubuntu | PCs         | with12 | cores,3.7     |
parametersoftheunderlyingstandardPIRscheme.The
|     |     |     |     |     |     |     | GHzIntelXeon |     | W-2135and64GBofRAM. |     |     |     | Weuse |
| --- | --- | --- | --- | --- | --- | --- | ------------ | --- | ------------------- | --- | --- | --- | ----- |
onlyexceptiontothisistheplaintextmodulus,whichwe
theAVX2andAVX-512instructionsetswithSIMDin-
rounddowntotheclosestprime. structionsenabled.Allourexperimentsusesingle-thread
Cuckoo Hashing Optimization. In our cuckoo hash- execution.Reportedresultsaretheaverageofatleast10
ingkeywordPIRimplementations,weusetheoptimiza- experimentaltrialswithstandarddeviationlessthan10%
tion that empty entries will be skipped. As empty en- ofthemeans.MonetarycostsarecomputedusingAma-
triesarerepresentedusing0,theycanbeskippedduring zonEC2pricingoft2.2xlargeinstances[2]of$0.09per
serverprocessing.Toourknowledge,thisoptimization GBofoutboundtrafficand$0.014perCPUhouratthe
wasneverpresentedelsewhere.Withoutthis,SparsePIR time.Wedonotreportclienttimesastheyareverysmall
wouldactuallyhavebettercomputationascuckoohash- (see[8,9,51]).Followingpriorworks,wedefinerateas
ingkeywordPIRneeds(2+ε)nentrieswhileourframe- theratiooftheretrievedrecordsizetotheresponsesize.

|                                   |     |     | d   |           |     | SparsePIRg | SparsePIRc |     |
| --------------------------------- | --- | --- | --- | --------- | --- | ---------- | ---------- | --- |
| Computationtimeonε,256Bperelement |     |     | 1   | SparsePIR |     |            |            |     |
128
400
| sdnocesnievlosotemiT | d =128 | ε             |     | 0.38 |     | 0.1 | <0.03  |     |
| -------------------- | ------ | ------------- | --- | ---- | --- | --- | ------ | --- |
| 350                  | 1      |               |     |      |     |     |        |     |
|                      | d      | ClientStorage |     | 0B   |     | 0B  | 11.1KB |     |
1 =512
| 300 |     | 512 |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
d =1024
|     | 1   | ε             |     | 0.18 |     | 0.06 | <0.03 |     |
| --- | --- | ------------- | --- | ---- | --- | ---- | ----- | --- |
| 250 |     | ClientStorage |     | 0B   |     | 0B   | 3.4KB |     |
| 200 |     | 1024          |     |      |     |      |       |     |
|     |     | ε             |     | 0.12 |     | 0.05 | <0.03 |     |
150
|     |     | ClientStorage |     | 0B  |     | 0B  | 1.8KB |     |
| --- | --- | ------------- | --- | --- | --- | --- | ----- | --- |
100
|     |     | Figure11: | ComparisonsofSparsePIR,SparsePIRgand |     |     |     |     |     |
| --- | --- | --------- | ------------------------------------ | --- | --- | --- | --- | --- |
50
4·10−2 8·10−2 0.12 0.16 0.2 SparsePIRcofachieveableεforn=220elements.
ε
7.3 Comparisons
Computationtimeonε,256Bperelement
| 500                  |        | CuckooHashingKeywordPIR.Figure12compares |        |         |         |             |              |     |
| -------------------- | ------ | ---------------------------------------- | ------ | ------- | ------- | ----------- | ------------ | --- |
| sdnocesnievlosotemiT | d =128 |                                          |        |         |         |             |              |     |
| 450                  | 1      | the                                      | cuckoo | hashing | [8] and | ourfamilies | of SparsePIR |     |
d =512
|     | 1   | frameworksappliedtovariousPIRprotocols.SparsePIR |     |     |     |     |     |     |
| --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- | --- |
400
d =1024
|     | 1   | exhibitsclearadvantageoverthecuckoohashingframe- |     |     |     |     |     |     |
| --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- | --- |
| 350 |     | workontheresponsesizeandtherate.Thisisexpected   |     |     |     |     |     |     |
becausecuckoohashingframeworkrequirestheserver
300
tosendtwociphertexts,whileSparsePIRsendsonlyone
250
ciphertext.Cuckoohashingframeworkhasaslightad-
| 200 |     | vantage  | in computation |          | cost        | with | the optimization | of  |
| --- | --- | -------- | -------------- | -------- | ----------- | ---- | ---------------- | --- |
|     |     | skipping | empty          | entries. | However,the |      | smalladditional  |     |
150
4·10−2 8·10−2 0.12 0.16 0.2 server computation time is a worthwhile trade-off for
| ε   |     | thesubstantialimprovementintheresponsesizeasevi- |     |     |     |     |     |     |
| --- | --- | ------------------------------------------------ | --- | --- | --- | --- | --- | --- |
dencedinthereducedservercosts.Duetopacking,we
Figure10: ComputationtimetosolvetheSparsePIRg
obtainsmallerεforsmallerentries(e.g.256bytes).
linearsystemvsεon220
elementsfordimensionsizes WenotethatbothSparsePIRgandSparsePIRccanre-
d =128,512and1024.Wechosebandparameterw∈
| 1   |     | ducetheservercomputationcomparedtoSparsePIRbe- |     |     |     |     |     |     |
| --- | --- | ---------------------------------------------- | --- | --- | --- | --- | --- | --- |
[85,120].
causeofthesmallεbothencodingschemescanachieve.
7.2 SparsePIRFamilyParameters Constant-WeightKeywordPIR.InFigure13,wecom-
pareSparsePIRwithkeywordPIRusingconstant-weight
Wechooseourparameterstoaccomodatetherecursion equality operators [50] using their open source imple-
dimensionsofpriorworks.RecallOnionPIR[54]used mentation [1]. We observe that SparsePIR framework
instantiatedontheSpiralprotocoloutperformsconstant-
| d 1 =128dimensionsandSpiral[51]usedd | 1 =512.Re- |     |     |     |     |     |     |     |
| ------------------------------------ | ---------- | --- | --- | --- | --- | --- | --- | --- |
callthatourframeworksencodeadatabaseofnkey-value weightkeywordPIRforbothcommunicationandcom-
pairsintoaservingdatabaseofsize(1+ε)n. putation,demonstratingthebenefitsofSparsePIR.Our
experimentsusesimilarsettingsas[50]with16-bitkey-
ForSparsePIRandSparsePIRg,seeFigure9andFig-
wordlengthsanddatabasesizesof210,212and214.
ure10forthetimetocreateanencodingofsize(1+ε)n.
ForSparsePIR,wechoosebandparameterw∈[50,60] BatchKeywordPIR.InFigure14,wepresentexperi-
forallourexperiments,whichseemtoprovidegoodtrade- mentsforbatchkeywordPIRusingthebatchPIRframe-
offsbetweenthecomputationtimeandthesolvabilityof work of Angel et al. [9]. As the underlying keyword
the random bandlinearsystems. Similarly,we choose PIR,weusethecuckoohashingframework[8]andour
| bandparameterw∈[85,120]forSparsePIRg.Fromthese |     | SparsePIR |            |     |     |            |                 |     |
| ---------------------------------------------- | --- | --------- | ---------- | --- | --- | ---------- | --------------- | --- |
|                                                |     |           | framework. |     | We  | use Spiral | as the blackbox |     |
empiricalresults,wecanchooseappropriatevaluesof PIR protocolforbothframeworks. We run ourexperi-
ε.InFigure11,wecompareSparsePIR,SparsePIRgand mentson220288-byteentries,whicharealsousedin[9].
SparsePIRcintermsofthetradeoffsbetweenεandthe
Weusedt=3hashfunctionsandthenumberofbuckets
client storage. For SparsePIRc, we used bzip2 for the to be 1.5ℓ where ℓ is the batch size. We observe that
compressionalgorithm. SparsePIRhalvesresponsesizeandreducesservercosts

|          | Onion  | Onion     |     | Onion      |     | Onion      |     | Spiral |     | Spiral    | Spiral     | Spiral     |
| -------- | ------ | --------- | --- | ---------- | --- | ---------- | --- | ------ | --- | --------- | ---------- | ---------- |
| Database |        |           |     | SparsePIRg |     | SparsePIRc |     |        |     |           | SparsePIRg | SparsePIRc |
|          | CH-PIR | SparsePIR |     |            |     |            |     | CH-Pir |     | SparsePir |            |            |
220×256B
| QuerySize    | 63KB  | 63KB  |     | 63KB  |     | 63KB  |     | 14KB  |     | 14KB  | 14KB  | 14KB  |
| ------------ | ----- | ----- | --- | ----- | --- | ----- | --- | ----- | --- | ----- | ----- | ----- |
| ResponseSize | 254KB | 127KB |     | 127KB |     | 127KB |     | 42KB  |     | 21KB  | 21KB  | 21KB  |
| Computation  | 3.03s | 3.04s |     | 3.10s |     | 3.05  |     | 1.41s |     | 1.44s | 1.45s | 1.42s |
| Rate         | 0.001 | 0.002 |     | 0.002 |     | 0.002 |     | 0.006 |     | 0.012 | 0.012 | 0.012 |
ServerCost $0.000034 $0.000027 $0.000029 $0.000028 $0.0000091 $0.0000074 $0.0000074 $0.0000073
217×30KB
| QuerySize    | 63KB  | 63KB  |     | 63KB  |     | 63KB  |     | 14KB  |     | 14KB | 14KB | 14KB |
| ------------ | ----- | ----- | --- | ----- | --- | ----- | --- | ----- | --- | ---- | ---- | ---- |
| ResponseSize | 254KB | 127KB |     | 127KB |     | 127KB |     | 172KB |     | 86KB | 86KB | 86KB |
Computation 32.25s 41.91s 32.24s 32.28s 10.02s 11.57s 10.21s 10.18s
| Rate | 0.118 | 0.236 |     | 0.236 |     | 0.236 |     | 0.174 |     | 0.349 | 0.349 | 0.349 |
| ---- | ----- | ----- | --- | ----- | --- | ----- | --- | ----- | --- | ----- | ----- | ----- |
ServerCost $0.00015 $0.00017 $0.00014 0.00014 $0.000054 $0.000052 $0.000047 $0.000047
214×100KB
| QuerySize | 63KB | 63KB |     | 63KB |     | 63KB |     | 14KB |     | 14KB | 14KB | 14KB |
| --------- | ---- | ---- | --- | ---- | --- | ---- | --- | ---- | --- | ---- | ---- | ---- |
ResponseSize 1016KB 508KB 508KB 508KB 484KB 242KB 242KB 242KB
Computation 14.43s 17.32s 15.14s 15.10s 4.93s 5.91s 5.11s 5.17s
| Rate | 0.098 | 0.197 |     | 0.197 |     | 0.197 |     | 0.207 |     | 0.413 | 0.413 | 0.413 |
| ---- | ----- | ----- | --- | ----- | --- | ----- | --- | ----- | --- | ----- | ----- | ----- |
ServerCost $0.00014 $0.00011 $0.00010 $0.00010 $0.000061 $0.000044 $0.000041 $0.000041
Comparisonofcuckoohashing(CH)andSparsePIRframeworksforvariousPIRprotocols.
Figure12:
inexchangeforsmallincreaseincomputation. Other PIR Variants. The notion of PIR with prepro-
cessingwasintroducedbyBeimeletal.[11]wherethe
serverholdsapublichintstring.Thiswasalsostudiedas
8 RelatedWork
doubly-efficientPIR[14,18]withsub-linearonlinetime
EarlyPIRSchemes.Single-serverPIRwasintroduced usingstrongobfuscationassumptions.
|     |     |     |     |     |     |     | Stateful |     | PIR introduced |     | by Patel et al. | [58] allows |
| --- | --- | --- | --- | --- | --- | --- | -------- | --- | -------------- | --- | --------------- | ----------- |
byKushilevitzandOstrovsky[45].Severalfollow-ups
builtPIRusingnumber-theoreticassumptions[25,47,56] clientstocomputequery-independenthintsinanoffline
phasetohelpspeeduponlinequeriesthatwasalsostud-
andthephi-hidingassumption[17,34].
iedin[54].ArecentlineofworkstartingwithCorrigan-
| Lattice-basedPIR. | Inrecentyears,manyworksstud- |     |     |     |     |     |     |     |     |     |     |     |
| ----------------- | ---------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
GibbsandKogan[24]presentedconstructionswithsub-
iedpracticallyefficientPIRschemesusinglattice-based
linearonlinetimes.Multipleworkshavefurtherstudied
homomorphicencryptionasfirstdonein[6]withmany
thetopic[23,26,41,44,62,66].
follow-upsincluding[7–10,33,51,54,57].
| KeywordPIR.                                     | The notion | ofkeywordPIR |     |     | was intro- |     |     |             |     |     |     |     |
| ----------------------------------------------- | ---------- | ------------ | --- | --- | ---------- | --- | --- | ----------- | --- | --- | --- | --- |
| ducedbyChoretal.[20]thatrequiredmultiplerounds. |            |              |     |     |            |     | 9   | Conclusions |     |     |     |     |
Freedmanetal.[32]alsoconsideredkeywordPIRwith
Inthispaper,weproposedSparsePIR,SparsePIRg
| database privacy | using | oblivious | PRFs. | Ali | et al. | [8] |     |     |     |     |     | and |
| ---------------- | ----- | --------- | ----- | --- | ------ | --- | --- | --- | --- | --- | --- | --- |
presented a keyword PIR using cuckoo hashing. Mah- SparsePIRc,frameworks that build keyword PIR from
daviandKerschbaum[50]presentedaconstructionfrom
lattice-basedstandardPIRschemes.Ourschemesreduce
constant-weightequalityoperators. theresponsesizebyatleast2xcomparedtopriorkey-
wordPIRconstructions.Inessence,ourframeworksshow
BatchPIR.Severalworks[11,37,40,42,48]havestudied
batchPIRwiththegoalofefficientlyqueryingmultiple thatkeywordPIRmaybebuiltwithidenticalcommuni-
recordsatonce.Recentworks[9,10]introducedproba- cationandcomputationcostsasstandardPIR.Wealso
bilisticbatchcodesthatarethemostefficientbatchPIR showourkeywordPIRframeworksmayalsobeusedto
frameworkstodate. alsohalvetheresponseoverheadofbatchkeywordPIR.
Multi-ServerPIR.Alonglineofworkhasalsostudied
| PIR with | multiple, non-colluding |     | servers. |     | Early work |     |     |     |     |     |     |     |
| -------- | ----------------------- | --- | -------- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- |
References
| considered | information-theoretic |     | security | [21,29,30]. |     |     |     |     |     |     |     |     |
| ---------- | --------------------- | --- | -------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
Recentworkshowedconcretelyefficientconstructions [1] Constant-weight PIR. https://github.com/
usingfunctionsecretsharing [13,35,39]. rasoulam/constant-weight-pir.

| Database |     | CWKeywordPIR |     | SpiralSparsePir |     |     |     |     | Spiral | Spiral |     |
| -------- | --- | ------------ | --- | --------------- | --- | --- | --- | --- | ------ | ------ | --- |
BatchSize(ℓ)
| 214×256B |     |     |     |     |     |     |     | CH-BatchPir |     | SparseBatchPir |     |
| -------- | --- | --- | --- | --- | --- | --- | --- | ----------- | --- | -------------- | --- |
16
| QuerySize    |              | 216KB    |           |            | 14KB      |              |            |            |            |                |             |
| ------------ | ------------ | -------- | --------- | ---------- | --------- | ------------ | ---------- | ---------- | ---------- | -------------- | ----------- |
| ResponseSize |              | 103KB    |           |            | 11KB      | QuerySize    |            |            | 24KB       | 24KB           |             |
| Computation  |              | 280.34s  |           |            | 0.75s     | ResponseSize |            |            | 48KB       | 24KB           |             |
| Rate         |              | 0.0025   |           |            | 0.023     | Computation  |            |            | 0.56s      | 0.64s          |             |
| ServerCost   |              | $0.0011  |           | $0.0000039 |           | Rate         |            | 0.0000051  |            | 0.000010       |             |
| 212×30KB     |              |          |           |            |           |              |            |            |            | $0.0000045     |             |
|              |              |          |           |            |           | ServerCost   |            | $0.0000063 |            |                |             |
| QuerySize    |              | 216KB    |           |            | 14KB      | 64           |            |            |            |                |             |
| ResponseSize |              | 206KB    |           |            | 82KB      | QuerySize    |            |            | 24KB       | 24KB           |             |
| Computation  |              | 73.68s   |           |            | 0.93s     | ResponseSize |            |            | 48KB       | 24KB           |             |
|              |              |          |           |            |           | Computation  |            |            | 0.36s      | 0.42s          |             |
| Rate         |              | 0.146    |           |            | 0.366     |              |            |            |            |                |             |
| ServerCost   |              | $0.0003  |           | $0.000011  |           | Rate         |            | 0.0000051  |            | 0.000010       |             |
| 210×100KB    |              |          |           |            |           | ServerCost   |            | $0.0000055 |            | $0.0000037     |             |
| QuerySize    |              | 216KB    |           |            | 14KB      | 256          |            |            |            |                |             |
|              |              |          |           |            |           | QuerySize    |            |            | 24KB       | 24KB           |             |
| ResponseSize |              | 515KB    |           | 236KB      |           |              |            |            |            |                |             |
| Computation  |              | 20.98s   |           |            | 0.89s     | ResponseSize |            |            | 38KB       | 19KB           |             |
| Rate         |              | 0.194    |           |            | 0.424     | Computation  |            |            | 0.29s      | 0.33s          |             |
| ServerCost   |              | $0.00013 |           | $0.000024  |           | Rate         |            | 0.0000064  |            | 0.000013       |             |
|              |              |          |           |            |           | ServerCost   |            | $0.0000044 |            | $0.0000029     |             |
| Figure 13:   | Comparison   | of       | SparsePIR | and        | constant- |              |            |            |            |                |             |
|              |              |          |           |            |           | Figure 14:   | Comparison |            | of         | cuckoo hashing | [8] and     |
| weight       | (CW) keyword | PIR      | with      | 16-bit     | keyword   |              |            |            |            |                |             |
|              |              |          |           |            |           | SparsePIR    | keyword    | PIR        | frameworks | with           | Spiral [51] |
length[50].
forbatchPIRqueries.Thedatabasesizeis220288-byte
entries.Thereportednumbersaretheamortizedcostof
| [2] EC2On-DemandPricing. |     |     | https://aws.amazon.com/ |     |     |     |     |     |     |     |     |
| ------------------------ | --- | --- | ----------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
retrievingasingleentry.
ec2/pricing/on-demand/.
[3] OnionPIR. https://github.com/mhmughees/ [10] SebastianAngelandSrinathSetty. Unobservablecom-
Onion-PIR. municationoverfullyuntrustedinfrastructure. InOSDI
16,pages551–569,2016.
[4] Protectingyourdeviceinformationwithprivatesetmem-
| bership. | https://security.googleblog.com/2021/ |     |     |     |     |           |         |       |        |                 |        |
| -------- | ------------------------------------- | --- | --- | --- | --- | --------- | ------- | ----- | ------ | --------------- | ------ |
|          |                                       |     |     |     |     | [11] Amos | Beimel, | Yuval | Ishai, | and Tal Malkin. | Reduc- |
10/protecting-your-device-information-with.
|       |     |     |     |     |     | ing      | the servers           | computation |     | in private information | re- |
| ----- | --- | --- | --- | --- | --- | -------- | --------------------- | ----------- | --- | ---------------------- | --- |
| html. |     |     |     |     |     | trieval: | PIRwithpreprocessing. |             |     | In MihirBellare,edi-   |     |
tor,CRYPTO2000,volume1880ofLNCS,pages55–73.
| [5] Spiral. | https://github.com/menonsamir/spiral. |     |     |     |     |     |     |     |     |     |     |
| ----------- | ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Springer,Heidelberg,August2000.
| [6] Carlos | AguilarMelchor,Joris |     | Barrier,LaurentFousse, |     |     |     |     |     |     |     |     |
| ---------- | -------------------- | --- | ---------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
andMarc-OlivierKillijian. XPIR:Privateinformation [12] NikitaBorisov,GeorgeDanezis,andIanGoldberg. DP5:
retrievalforeveryone. PoPETs,2016(2):155–174,April Aprivatepresenceservice. PoPETs,2015(2):4–24,April
2015.
2016.
[7] Ishtiyaque Ahmad,Yuntian Yang,Divyakant Agrawal, [13] EletteBoyle,NivGilboa,andYuvalIshai. Functionse-
AmrElAbbadi,andTrinabhGupta. Addra:Metadata- cretsharing:Improvementsandextensions. InEdgarR.
privatevoicecommunicationoverfullyuntrustedinfras- Weippl,StefanKatzenbeisser,ChristopherKruegel,An-
tructure. InOSDI21,2021. drewC.Myers,andShaiHalevi,editors,ACMCCS2016,
pages1292–1303.ACMPress,October2016.
| [8] Asra | Ali, Tancrède | Lepoint, | Sarvar | Patel, | Mariana |     |     |     |     |     |     |
| -------- | ------------- | -------- | ------ | ------ | ------- | --- | --- | --- | --- | --- | --- |
Raykova,PhillippSchoppmann,Karn Seth,andKevin [14] EletteBoyle,YuvalIshai,RafaelPass,andMaryWootters.
Yeo. Communication-computationtrade-offsinPIR. In Canweaccessadatabasebothlocallyandprivately? In
MichaelBaileyandRachelGreenstadt,editors,USENIX YaelKalaiandLeonidReyzin,editors,TCC2017,PartII,
Security2021,pages1811–1828.USENIXAssociation, volume10678ofLNCS,pages662–693.Springer,Hei-
| August2021. |     |     |     |     |     | delberg,November2017. |     |     |     |     |     |
| ----------- | --- | --- | --- | --- | --- | --------------------- | --- | --- | --- | --- | --- |
[9] SebastianAngel,HaoChen,KimLaine,andSrinathT.V. [15] ZvikaBrakerski. Fullyhomomorphicencryptionwithout
Setty. PIRwithcompressedqueriesandamortizedquery modulusswitchingfromclassicalGapSVP. InReihaneh
processing. In2018IEEESymposiumonSecurityand Safavi-NainiandRanCanetti,editors,CRYPTO2012,vol-
Privacy,pages962–979.IEEEComputerSocietyPress, ume7417ofLNCS,pages868–886.Springer,Heidelberg,
| May2018. |     |     |     |     |     | August2012. |     |     |     |     |     |
| -------- | --- | --- | --- | --- | --- | ----------- | --- | --- | --- | --- | --- |

[16] ZvikaBrakerski,CraigGentry,andVinodVaikuntanathan. [28] MartinDietzfelbingerandStefanWalzer. Efficientgauss
(leveled) fully homomorphic encryption without boot- elimination fornear-quadratic matrices withone short
strapping. ACMTransactionsonComputationTheory randomblockperrow,withapplications. InESA2019,
| (TOCT),6(3):1–36,2014. |     |     | 2019.                          |     |                     |
| ---------------------- | --- | --- | ------------------------------ | --- | ------------------- |
|                        |     |     | [29] ZeevDvirandSivakanthGopi. |     | 2-serverPIRwithsub- |
[17] ChristianCachin,SilvioMicali,andMarkusStadler.Com-
putationallyprivateinformationretrievalwithpolyloga- polynomialcommunication. InRoccoA.Servedioand
rithmic communication. In Jacques Stern,editor,EU- RonittRubinfeld,editors,47thACMSTOC,pages577–
ROCRYPT’99,volume 1592 of LNCS,pages 402–414. 584.ACMPress,June2015.
Springer,Heidelberg,May1999. [30] Klim Efremenko. 3-query locally decodable codes of
|     |     |     | subexponentiallength. | InMichaelMitzenmacher,editor, |     |
| --- | --- | --- | --------------------- | ----------------------------- | --- |
[18] RanCanetti,JustinHolmgren,andSilasRichelson. To-
41stACMSTOC,pages39–44.ACMPress,May/June
wardsdoublyefficientprivateinformationretrieval. In
2009.
YaelKalaiandLeonidReyzin,editors,TCC2017,PartII,
volume10678ofLNCS,pages694–726.Springer,Hei- [31] JunfengFanandFrederikVercauteren. Somewhatpracti-
delberg,November2017. calfullyhomomorphicencryption. CryptologyePrint
|                   |                       |             | Archive, Report | 2012/144, 2012. | https://eprint. |
| ----------------- | --------------------- | ----------- | --------------- | --------------- | --------------- |
| [19] Hao Chen,Kim | Laine,andPeterRindal. | Fastprivate |                 |                 |                 |
iacr.org/2012/144.
setintersectionfromhomomorphicencryption. InBha-
vaniM.Thuraisingham,DavidEvans,TalMalkin,and [32] Michael J. Freedman,Yuval Ishai,Benny Pinkas,and
DongyanXu,editors,ACMCCS2017,pages1243–1255. OmerReingold. Keywordsearchandobliviouspseudo-
|     |     |     | randomfunctions. | InJoeKilian,editor,TCC2005,vol- |     |
| --- | --- | --- | ---------------- | ------------------------------- | --- |
ACMPress,October/November2017.
ume3378ofLNCS,pages303–324.Springer,Heidelberg,
| [20] BennyChor,NivGilboa,andMoniNaor. |     | Privateinfor- |     |     |     |
| ------------------------------------- | --- | ------------- | --- | --- | --- |
February2005.
| mationretrievalbykeywords. | CryptologyePrintArchive, |     |                                |                     |     |
| -------------------------- | ------------------------ | --- | ------------------------------ | ------------------- | --- |
|                            |                          |     | [33] CraigGentryandShaiHalevi. | CompressibleFHEwith |     |
Report1998/003,1998. https://eprint.iacr.org/
| 1998/003. |     |     | applicationstoPIR. | InDennisHofheinzandAlonRosen, |     |
| --------- | --- | --- | ------------------ | ----------------------------- | --- |
editors,TCC2019,PartII,volume11892ofLNCS,pages
[21] Benny Chor, Eyal Kushilevitz, Oded Goldreich, and 438–464.Springer,Heidelberg,December2019.
| Madhu Sudan. | Private information | retrieval. Journal |                                    |     |                     |
| ------------ | ------------------- | ------------------ | ---------------------------------- | --- | ------------------- |
|              |                     |                    | [34] CraigGentryandZulfikarRamzan. |     | Single-databasepri- |
oftheACM(JACM),45(6):965–981,1998.
vateinformationretrievalwithconstantcommunication
|                   |                            | Random | rate. InLuísCaires,GiuseppeF.Italiano,LuísMonteiro, |     |     |
| ----------------- | -------------------------- | ------ | --------------------------------------------------- | --- | --- |
| [22] ColinCooper. | Ontherankofrandommatrices. |        |                                                     |     |     |
Yung,editors,ICALP
Structures&Algorithms,16(2):209–232,2000. Catuscia Palamidessi,and Moti
2005,volume3580ofLNCS,pages803–815.Springer,
[23] HenryCorrigan-Gibbs,AlexandraHenzinger,andDmitry
Heidelberg,July2005.
| Kogan. Single-serverprivateinformationretrievalwith |     |     |     |     |     |
| --------------------------------------------------- | --- | --- | --- | --- | --- |
sublinearamortizedtime. InOrrDunkelmanandStefan [35] NivGilboaandYuvalIshai. Distributedpointfunctions
|     |     |     | andtheirapplications. | InPhongQ.NguyenandElisa- |     |
| --- | --- | --- | --------------------- | ------------------------ | --- |
Dziembowski,editors,EUROCRYPT2022,PartII,vol-
bethOswald,editors,EUROCRYPT2014,volume8441of
ume13276ofLNCS,pages3–33.Springer,Heidelberg,
LNCS,pages640–658.Springer,Heidelberg,May2014.
May/June2022.
|                           |           |                    | [36] MatthewGreen,WatsonLadd,andIanMiers. |     | Aprotocol |
| ------------------------- | --------- | ------------------ | ----------------------------------------- | --- | --------- |
| [24] Henry Corrigan-Gibbs | andDmitry | Kogan. Private in- |                                           |     |           |
forprivatelyreportingadimpressionsatscale.InEdgarR.
| formationretrievalwithsublinearonlinetime. |     | InAnne |     |     |     |
| ------------------------------------------ | --- | ------ | --- | --- | --- |
Weippl,StefanKatzenbeisser,ChristopherKruegel,An-
| Canteaut | and Yuval Ishai,editors,EUROCRYPT | 2020, |     |     |     |
| -------- | --------------------------------- | ----- | --- | --- | --- |
drewC.Myers,andShaiHalevi,editors,ACMCCS2016,
PartI,volume12105ofLNCS,pages44–75.Springer,
pages1591–1601.ACMPress,October2016.
Heidelberg,May2020.
[37] JensGroth,AggelosKiayias,andHelgerLipmaa. Multi-
| [25] IvanDamgårdandMatsJurik. | Ageneralisation,asimpli- |     |     |     |     |
| ----------------------------- | ------------------------ | --- | --- | --- | --- |
querycomputationally-privateinformationretrievalwith
ficationandsomeapplicationsofPaillier’sprobabilistic constantcommunicationrate. InPhongQ.Nguyenand
public-keysystem. InKwangjoKim,editor,PKC2001, DavidPointcheval,editors,PKC2010,volume6056of
volume1992ofLNCS,pages119–136.Springer,Heidel-
LNCS,pages107–123.Springer,Heidelberg,May2010.
berg,February2001.
[38] TrinabhGupta,NatachaCrooks,WhitneyMulhern,Sri-
| [26] Alex Davidson, | Gonçalo Pestana, | and Sofía Celi. |     |     |     |
| ------------------- | ---------------- | --------------- | --- | --- | --- |
nathSetty,LorenzoAlvisi,andMichaelWalfish.Scalable
FrodoPIR:Simple,scalable,single-serverprivateinfor- andprivatemediaconsumptionwithpopcorn. InNSDI
| mationretrieval. | 2023. |     | 16,pages91–107,2016. |     |     |
| ---------------- | ----- | --- | -------------------- | --- | --- |
[27] Daniel Demmler, Peter Rindal, Mike Rosulek, and [39] SyedMahbubHafizandRyanHenry.Abitmorethanabit
Ni Trieu. PIR-PSI: Scaling private contact discov- ismorethanabitbetter:Faster(essentially)optimal-rate
ery. ProceedingsonPrivacyEnhancingTechnologies, many-serverPIR. PoPETs,2019(4):112–131,October
| 2018(4):159–178,2018. |     |     | 2019. |     |     |
| --------------------- | --- | --- | ----- | --- | --- |

[40] RyanHenry. PolynomialbatchcodesforefficientIT-PIR. [54] MuhammadHarisMughees,HaoChen,andLingRen.
PoPETs,2016(4):202–218,October2016. OnionPIR:Responseefficientsingle-serverPIR. InGio-
|     |     |     |     |     |     | vanni | Vigna | and Elaine | Shi,editors,ACM |     | CCS 2021, |
| --- | --- | --- | --- | --- | --- | ----- | ----- | ---------- | --------------- | --- | --------- |
[41] AlexandraHenzinger,MatthewM.Hong,HenryCorrigan-
Gibbs,SarahMeiklejohn,andVinodVaikuntanathan.One pages2292–2306.ACMPress,November2021.
serverforthepriceoftwo:Simpleandfastsingle-server [55] Rafail Ostrovsky and William E. Skeith III. A sur-
privateinformationretrieval. InUSENIXSecurity23(to vey of single database PIR: Techniques and applica-
appear),2023. tions. CryptologyePrintArchive,Paper2007/059,2007.
https://eprint.iacr.org/2007/059.
[42] YuvalIshai,EyalKushilevitz,RafailOstrovsky,andAmit
Sahai. Batch codes and theirapplications. In László [56] PascalPaillier. Public-keycryptosystemsbasedoncom-
Babai,editor,36thACMSTOC,pages 262–271. ACM positedegreeresiduosityclasses. InJacquesStern,editor,
| Press,June2004. |     |     |     |     |     | EUROCRYPT’99,volume1592ofLNCS,pages223–238. |     |     |     |     |     |
| --------------- | --- | --- | --- | --- | --- | ------------------------------------------- | --- | --- | --- | --- | --- |
[43] DonaldErvinKnuth. TheArtofComputerProgramming, Springer,Heidelberg,May1999.
volume4,fasicle2. 2005. [57] JeongeunParkandMehdiTibouchi. SHECS-PIR:Some-
whathomomorphicencryption-basedcompactandscal-
| [44] DmitryKoganandHenryCorrigan-Gibbs. |     |     |     |     | Privateblock- |     |     |     |     |     |     |
| --------------------------------------- | --- | --- | --- | --- | ------------- | --- | --- | --- | --- | --- | --- |
ableprivateinformationretrieval.InLiqunChen,Ninghui
| listlookupswithchecklist. |     |     | InMichaelBaileyandRachel |     |     |           |           |     |          |                       |     |
| ------------------------- | --- | --- | ------------------------ | --- | --- | --------- | --------- | --- | -------- | --------------------- | --- |
|                           |     |     |                          |     |     | Li,Kaitai | Liang,and |     | Steve A. | Schneider,editors,ES- |     |
Greenstadt,editors,USENIXSecurity2021,pages875–
ORICS2020,PartII,volume12309ofLNCS,pages86–
892.USENIXAssociation,August2021.
106.Springer,Heidelberg,September2020.
| [45] EyalKushilevitzandRafailOstrovsky. |     |     |     |     | Replication is |     |     |     |     |     |     |
| --------------------------------------- | --- | --- | --- | --- | -------------- | --- | --- | --- | --- | --- | --- |
NOTneeded:SINGLEdatabase,computationally-private [58] SarvarPatel,GiuseppePersiano,andKevinYeo. Private
information retrieval. In 38th FOCS,pages 364–373. statefulinformationretrieval. InDavidLie,Mohammad
Mannan,MichaelBackes,andXiaoFengWang,editors,
IEEEComputerSocietyPress,October1997.
ACMCCS2018,pages1002–1019.ACMPress,October
| [46] AlbertKwon,DavidLazar,SrinivasDevadas,andBryan |         |                                    |     |     |     | 2018. |     |     |     |     |     |
| --------------------------------------------------- | ------- | ---------------------------------- | --- | --- | --- | ----- | --- | --- | --- | --- | --- |
| Ford.                                               | Riffle: | Anefficientcommunicationsystemwith |     |     |     |       |     |     |     |     |     |
[59] BennyPinkas,ThomasSchneider,andMichaelZohner.
stronganonymity.PoPETs,2016(2):115–134,April2016.
|                    |     |                                     |     |     |     | Scalable | private | set | intersection | based | on OT exten- |
| ------------------ | --- | ----------------------------------- | --- | --- | --- | -------- | ------- | --- | ------------ | ----- | ------------ |
| [47] HelgerLipmaa. |     | Anoblivioustransferprotocolwithlog- |     |     |     |          |         |     |              |       |              |
sion. ACMTransactionsonPrivacyandSecurity(TOPS),
| squaredcommunication. |     |     | InInternationalConferenceon |     |     |     |     |     |     |     |     |
| --------------------- | --- | --- | --------------------------- | --- | --- | --- | --- | --- | --- | --- | --- |
21(2):1–35,2018.
InformationSecurity,pages314–328.Springer,2005.
|                                 |     |                     |     |                     |           | [60] Boris            | Pittel | and Gregory | B Sorkin.                 |     | The satisfiability |
| ------------------------------- | --- | ------------------- | --- | ------------------- | --------- | --------------------- | ------ | ----------- | ------------------------- | --- | ------------------ |
| [48] WouterLueksandIanGoldberg. |     |                     |     | Sublinearscalingfor |           |                       |        |             |                           |     |                    |
|                                 |     |                     |     |                     |           | thresholdfork-XORSAT. |        |             | Combinatorics,Probability |     |                    |
| multi-client                    |     | private information |     | retrieval.          | In Rainer |                       |        |             |                           |     |                    |
andComputing,25(2):236–268,2016.
BöhmeandTatsuakiOkamoto,editors,FC2015,volume
|     |     |     |     |     |     | [61] ElyPorat. | Anoptimalbloomfilterreplacementbased |     |     |     |     |
| --- | --- | --- | --- | --- | --- | -------------- | ------------------------------------ | --- | --- | --- | --- |
8975ofLNCS,pages168–186.Springer,Heidelberg,Jan-
InInternationalComputerScience
| uary2015. |     |     |     |     |     | onmatrixsolving. |     |     |     |     |     |
| --------- | --- | --- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- |
SymposiuminRussia,pages263–273.Springer,2009.
[49] VadimLyubashevsky,ChrisPeikert,andOdedRegev.On
ideal lattices and learning with errors over rings. In [62] ElaineShi,WaqarAqeel,BalakrishnanChandrasekaran,
HenriGilbert,editor,EUROCRYPT2010,volume6110 andBruceM.Maggs. Puncturablepseudorandomsets
ofLNCS,pages1–23.Springer,Heidelberg,May/June andprivateinformationretrievalwithnear-optimalonline
|     |     |     |     |     |     | bandwidthandtime. |     |     | In TalMalkin | andChrisPeikert, |     |
| --- | --- | --- | --- | --- | --- | ----------------- | --- | --- | ------------ | ---------------- | --- |
2010.
editors,CRYPTO2021,PartIV,volume12828ofLNCS,
| [50] Rasoul     | Akhavan | Mahdavi           | and | Florian | Kerschbaum. |       |                 |     |              |     |                 |
| --------------- | ------- | ----------------- | --- | ------- | ----------- | ----- | --------------- | --- | ------------ | --- | --------------- |
|                 |         |                   |     |         |             | pages | 641–669,Virtual |     | Event,August |     | 2021. Springer, |
| Constant-weight |         | PIR: Single-round |     | keyword | PIR via     |       |                 |     |              |     |                 |
Heidelberg.
| constant-weightequalityoperators. |     |     |     | InUSENIXSecurity |     |     |     |     |     |     |     |
| --------------------------------- | --- | --- | --- | ---------------- | --- | --- | --- | --- | --- | --- | --- |
22,pages1723–1740,Boston,MA,2022. [63] RaduSionandBogdanCarbunar. Onthepracticalityof
|                                   |     |     |     |                   |     | privateinformationretrieval. |     |     | InNDSS2007.TheInternet |     |     |
| --------------------------------- | --- | --- | --- | ----------------- | --- | ---------------------------- | --- | --- | ---------------------- | --- | --- |
| [51] SamirJordanMenonandDavidJWu. |     |     |     | Spiral:Fast,high- |     |                              |     |     |                        |     |     |
Society,February/March2007.
| rate | single-serverPIR | via | FHE | composition. | In 2022 |              |            |     |         |        |                 |
| ---- | ---------------- | --- | --- | ------------ | ------- | ------------ | ---------- | --- | ------- | ------ | --------------- |
|      |                  |     |     |              |         | [64] Douglas | Wiedemann. |     | Solving | sparse | linearequations |
IEEESymposiumonSecurityandPrivacy,2022.
|              |             |     |                  |     |           | overfinitefields. |     | IEEEtransactionsoninformationthe- |     |     |     |
| ------------ | ----------- | --- | ---------------- | --- | --------- | ----------------- | --- | --------------------------------- | --- | --- | --- |
| [52] Prateek | Mittal,Femi | G.  | Olumofin,Carmela |     | Troncoso, |                   |     |                                   |     |     |     |
ory,32(1):54–62,1986.
| Nikita | Borisov,and | Ian | Goldberg. | PIR-tor: | Scalable |     |     |     |     |     |     |
| ------ | ----------- | --- | --------- | -------- | -------- | --- | --- | --- | --- | --- | --- |
anonymouscommunicationusingprivateinformationre- [65] DavidJ.Wu,JoeZimmerman,JérémyPlanul,andJohnC.
trieval. InUSENIXSecurity2011.USENIXAssociation, Mitchell. Privacy-preservingshortestpathcomputation.
| August2011. |     |     |     |     |     | InNDSS2016.TheInternetSociety,February2016. |     |     |     |     |     |
| ----------- | --- | --- | --- | --- | --- | ------------------------------------------- | --- | --- | --- | --- | --- |
[53] Michael Molloy. Cores in random hypergraphs and [66] KevinYeo. Lowerboundsfor(batch)pirwithprivate
boolean formulas. Random Structures & Algorithms, preprocessing. InEurocrypt2023(toappear),2023.
27(1):124–135,2005.
