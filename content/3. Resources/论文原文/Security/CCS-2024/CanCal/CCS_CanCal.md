---
publish: true
---

4
2
0
2

g
u
A
9
2

]

R
C
.
s
c
[

1
v
5
1
5
6
1
.
8
0
4
2
:
v
i
X
r
a

CanCal: Towards Real-time and Lightweight Ransomware
Detection and Response in Industrial Environments

Shenao Wang∗†
shenaowang@hust.edu.cn
Huazhong University of Science and
Technology
Wuhan, China

Feng Dong∗†
dongfeng@hust.edu.cn
Huazhong University of Science and
Technology
Wuhan, China

Hangfeng Yang
yanghangfeng@sangfor.com.cn
Sangfor Technologies Inc.
Shenzhen, China

Jingheng Xu‡
xjh@sangfor.com.cn
Sangfor Technologies Inc.
Shenzhen, China

Haoyu Wang†‡
haoyuwang@hust.edu.cn
Huazhong University of Science and
Technology
Wuhan, China

ABSTRACT

Ransomware attacks have emerged as one of the most significant
cybersecurity threats. Despite numerous methods proposed for de-
tecting and defending against ransomware, existing approaches face
two fundamental limitations in large-scale industrial applications:
(1) Behavior-based detection engines suffer from the enormous over-
head of monitoring all processes and resource constraints for model
inference, failing to meet the requirements for real-time detection;
(2) Decoy-based detection engines generate an overwhelming num-
ber of false positives in large-scale industrial clusters, leading to
intolerable disruptions to critical processes and excessive inspec-
tion efforts from security analysts. To address these challenges, we
propose CanCal, a real-time and lightweight ransomware detec-
tion system. Specifically, instead of indiscriminately analyzing all
processes, CanCal selectively filters suspicious processes by the
monitoring layers and then performs in-depth behavioral analysis
to isolate ransomware activities from benign operations, minimiz-
ing alert fatigue while ensuring lightweight computational and
storage overhead. The experimental results on a large-scale indus-
trial environment (1,761 ransomware, ~ 3 million events, continuous
test over 5 months) indicate that CanCal achieves a remarkable
99.65% true positive rate on 555,678 unknown ransomware behav-
ior events, with near-zero false positives. CanCal is as effective as
state-of-the-art techniques while enabling rapid inference within
30ms and real-time response within a maximum of 3 seconds. Can-
Cal dramatically reduces average CPU utilization by 91.04% (from

∗Both authors contributed equally to this research.
†Hubei Key Laboratory of Distributed System Security, Hubei Engineering Research
Center on Big Data Security, School of Cyber Science and Engineering, Huazhong
University of Science and Technology.
‡Co-corresponding authors.

Permission to make digital or hard copies of all or part of this work for personal or
classroom use is granted without fee provided that copies are not made or distributed
for profit or commercial advantage and that copies bear this notice and the full citation
on the first page. Copyrights for components of this work owned by others than the
author(s) must be honored. Abstracting with credit is permitted. To copy otherwise, or
republish, to post on servers or to redistribute to lists, requires prior specific permission
and/or a fee. Request permissions from permissions@acm.org.
CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.
© 2024 Copyright held by the owner/author(s). Publication rights licensed to ACM.
ACM ISBN 979-8-4007-0636-3/24/10
https://doi.org/10.1145/3658644.3690269

6.7% to 0.6%) and peak CPU utilization by 76.69% (from 26.6% to
6.2%), while avoiding 76.50% (from 3,192 to 750) of the inspection
efforts from security analysts. By the time of this writing, CanCal
has been integrated into a commercial product and successfully
deployed on 3.32 million endpoints for over a year. From March
2023 to April 2024, CanCal successfully detected and thwarted
61 ransomware attacks. A detailed manual forensic analysis of
27 ransomware attacks from March to June 2023 (including 13 n-
day exploits and 5 high-risk zero-day attacks) demonstrates the
effectiveness of CanCal in combating sophisticated and unknown
ransomware threats in real-world scenarios.

CCS CONCEPTS
• Security and privacy → Malware and its mitigation.

KEYWORDS

Ransomware detection, malware behavior analysis, EDR

ACM Reference Format:
Shenao Wang, Feng Dong, Hangfeng Yang, Jingheng Xu, and Haoyu Wang.
2024. CanCal: Towards Real-time and Lightweight Ransomware Detection
and Response in Industrial Environments. In Proceedings of the 2024 ACM
SIGSAC Conference on Computer and Communications Security (CCS ’24),
October 14–18, 2024, Salt Lake City, UT, USA. ACM, New York, NY, USA,
19 pages. https://doi.org/10.1145/3658644.3690269

1 INTRODUCTION

Recently, ransomware attacks have become one of the biggest
threats in the field of network security. With the rise of Ransomware
as a Service (RaaS), cybercriminals can launch attacks against indi-
vidual users, enterprises and governments [36, 38]. These attack-
ers usually adopt sophisticated tactics and techniques (e.g., 0-day
exploitation [3], file-less attacks [4], and process injection [28]),
making the ransomware more targeted and covert [30]. Tradi-
tional signature-based Endpoint Detection and Response (EDR)
systems [24, 35, 43] rely on identifying known patterns within bi-
nary files, which is prone to be evaded by adversarial techniques,
such as obfuscation and polymorphism, and ineffective against
evolving ransomware variants [20]. Modern ransomware detec-
tion solutions have shifted towards dynamic behavioral analysis,

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

which includes real-time monitoring of suspicious file operations
patterns [9], network traffic [29], API calls [25], and registry ac-
tivities [17]. By combining machine learning and deep learning
techniques, these methods [2, 12, 23, 35] offer a more robust mech-
anism against evasion techniques. However, their effectiveness in
industrial practices is still tampered by several intrinsic limitations
in time constraints [39] and heavy system overhead [7, 25]. Recently,
deception-based detection [14, 15, 21, 25] has been considered a
promising way to identify and mitigate ransomware attacks in real-
time, offering a proactive defense mechanism to lure, detect, and
analyze ransomware activities. These decoys are designed to mimic
real system assets (e.g., project codes or sensitive data), attracting
attackers while minimizing interaction with actual resources.
Research Gap. Despite these promising results, two substantial
gaps remain in translating these advancements into practical solu-
tions in industrial environments: intolerable system overheads and
notorious alert fatigue.

On one hand, advanced behavior-based detection methods neces-
sitate intensive monitoring and continuous analysis. While theoreti-
cally effective for batch processing, these methods require extensive
computing resources and result in intolerable delays in industrial-
scale streaming data processing. For example, ShieldFS [9] col-
lects runtime data over 60 minutes, resulting in approximately
500MB and nearly 7 million I/O Request Packet (IRP) logs (Note
that the scale of data encountered in actual industrial settings typ-
ically ranges from 10 to 100 times greater than this experimental
setup [10]), and then analyzes in batches for model prediction. How-
ever, this batch-processing approach does not align well with the
requirements of real-time detection needed in industrial settings.
Different from the controlled and post-mortem analysis in aca-
demic studies, industrial applications necessitate the streaming of
IRP log data directly into detection models to ensure timely threat
identification and response. According to the latest research from
MarauderMap [17], behavior pattern monitoring approaches have
an average response time ranging from 22.27 to 55.91 seconds. Even
with the most advanced computational resources such as NVIDIA
BlueField DPUs (Data Processing Units) and GPUs (Graphics Pro-
cessing Units), the processing time still needs approximately 12
seconds [32], which means that up to 20% of user files could be
encrypted by ransomware [10, 32], highlighting a significant gap in
response time that could lead to considerable data loss and system
compromise. Moreover, the continuous operation of logging IRP
data results in an average user-perceive overhead of 26%, let alone
intensive running inference models.

On the other hand, the issue of alert fatigue becomes particu-
larly acute in industrial practices. The deployment of decoy files
inevitably leads to a high volume of alerts, many of which may
be false positives (FPs). In industrial contexts, the effectiveness of
a detection system is significantly compromised by the inability
to efficiently manage and prioritize alerts. For example, accord-
ing to continuous monitoring by RTrap [14] over four weeks, the
ratio of user files to decoy files to FPs was 510:15:1. This implies
that if an industrial environment has a cluster of 1,000 hosts to
manage, each with 100,000 user files (about 54GB according to
Splunk Report [10]), there would be over 196,000 alerts monthly,
an overwhelming burden for security analysts [31]. Faced with
this challenge, EDR solutions in practice often resort to directly

terminating suspicious processes without manual checks, poten-
tially disrupting critical business or system operations. To avoid
such disruptions, industrial practitioners have to whitelist criti-
cal business or system processes. This coarse-grained approach,
however, renders the EDR system ineffective against ransomware
variants that employ whitelist-process injection techniques (about
39.1% [17]). Striking the right balance between false positives and
false negatives remains an ongoing research gap in the industrial
implementation of ransomware detection systems.

To bridge the gap between academic research and industrial
implementation of ransomware detection, we propose CanCal1, a
novel Monitoring-Detection-Response (MDR) framework that
combines lightweight monitoring, selective detection, and real-time
response to enable effective and efficient protection against evolving
ransomware threats. Specifically, CanCal employs a two-pronged
canary monitoring strategy that synergistically integrates decoy
files and ransom note monitoring. Instead of indiscriminately log-
ging IRP for all processes, CanCal focuses on processes that trigger
the monitoring points. By performing an in-depth analysis of the
IRP logs generated by the suspicious processes reported at the mon-
itoring layers, CanCal can detect unknown threats almost in real
time and combat alert fatigue caused by decoy files. We next de-
scribe the technical challenges and key insights in bridging these
research gaps.
Technical Challenges. Implementing such a lightweight and real-
time detection system presents three technical challenges:
① How to achieve real-time ransomware detection with lightweight
behavioral monitoring and model inference while minimizing sys-
tem overhead? The volume and velocity of data generated in
industrial-scale environments pose significant challenges for
real-time detection and response. Traditional batch processing
approaches [9, 18, 19, 33], which collect runtime data over ex-
tended periods and analyze them offline, are not suitable for the
timely detection and mitigation of ransomware attacks. The de-
lays introduced by batch processing can allow ransomware to
encrypt a significant portion of user files before response actions
are triggered, leading to substantial data loss.

② How to ensure monitoring points capture all potential ransomware
processes with the highest possible recall rate? Ransomware em-
ploys sophisticated evasion techniques to bypass detection, mak-
ing it challenging to capture all potential ransomware processes.
For example, ransomware may selectively encrypt files based on
certain criteria (e.g., file type, size, or location) to avoid trigger-
ing decoy file [11, 14, 39], resulting in false negatives. Improving
the recall rate of monitoring points is a daunting challenge that
requires a multi-faceted approach.

③ How to effectively identify and isolate the ransomware process,
akin to finding a needle in a haystack of massive alerts triggered
by monitoring points? The vast number of alerts generated by
monitoring points can overwhelm security analysts, leading to
alert fatigue and missed detections. Reducing false positives and
accurately identifying the ransomware process among the sea of
alerts is crucial for effective response and mitigation.

1The name is derived from the metaphorical phrase “Canary Calls”, signifying its role
as an early warning system against ransomware threats, akin to the canary’s distress
calls in coal mines.

CanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

Key Insights. To address these challenges, CanCal incorporates
several key insights and techniques:
• Lightweight Process Filtering. To achieve real-time detection
with minimal performance overhead, CanCal employs a light-
weight process filtering mechanism. Instead of indiscriminately
monitoring all processes, CanCal focuses on suspicious pro-
cesses that exhibit behaviors indicative of ransomware activity.
By selectively capturing critical system events and behaviors
associated with these suspicious processes, CanCal significantly
minimizes storage requirements and computational overhead,
allowing for real-time detection and response.

• Integrated Canary Monitoring. CanCal addresses the chal-
lenge of capturing all potential ransomware processes by em-
ploying an integrated canary monitoring approach. Unlike tra-
ditional decoy-based strategies that rely solely on decoy files,
CanCal combines multiple monitoring methods, including de-
coy file access monitoring and ransomware note monitoring. The
integration of diverse monitoring methods provides a more com-
prehensive and resilient detection mechanism, improving the
recall rate of capturing ransomware processes.

• Fine-grained Detection. To effectively identify and isolate ran-
somware processes among the vast number of alerts triggered
by monitoring points, CanCal employs fine-grained detection
techniques. CanCal leverages the gradient-boosting decision
tree to analyze the behavioral patterns and characteristics of
suspicious processes, enabling accurate differentiation between
benign and malicious activities and minimizing alert fatigue.
Contributions. Based on these insights, CanCal develops three
key components to address these challenges:
(1) Lightweight Decoy File Monitor. We introduce a novel approach
that transitions from passive damage acceptance to active ran-
somware enticement. By strategically placing lightweight decoy
files as traps, CanCal lures ransomware into exposing itself during
the early stages of infiltration. This enables real-time interception
and minimizes the impact on system performance, effectively ad-
dressing the challenge of early detection.
(2) Semantic-based Ransom Note Monitor. We leverage the distinct
textual patterns present in ransom notes to identify ransomware
infections. By performing semantic analysis on newly created or
modified files, CanCal swiftly detects ransom notes and alerts the
victim. This approach overcomes the potentially bypassable limita-
tions of decoy-based monitoring and allows behavioral detection
of specific processes to be triggered.
(3) Multi-granularity Behavior Detector. We develop an advanced de-
cision engine that combines automatically constructed embedding
with manually collected expert features to capture ransomware’s
complex and evolving behavior. By representing processes’ be-
havior as bipartite graphs and encoding patterns into compact
embeddings, the detector learns intricate behavioral characteris-
tics. The integration of expert knowledge further enhances the
decision-making process. This multi-granularity approach effec-
tively addresses the challenge of detecting sophisticated and evolv-
ing ransomware variants.
Evaluation. We construct an automated simulation system and
collect a large-scale sample library of behavior logs, encompassing
3 million event sequences from 1,335 known ransomware, 1,768

Figure 1: Overall architecture of CanCal.

benign software, and 426 unknown samples capable of evading
anti-virus engine detection. The experimental results indicate that
CanCal achieves a true positive rate of 99.65% on a large dataset
comprising 555,678 unknown ransomware behaviors, with near-
zero false positives. Not only does CanCal demonstrate excep-
tional detection capabilities, but it also exhibits efficient real-time
response and a lightweight design. Our ablation study reveals that,
compared to traditional approaches relying solely on decoy moni-
toring or file behavior monitoring, CanCal’s multi-layered detec-
tion mechanism significantly reduces resource consumption and
false positive alerts. For instance, the suspicious process filtering
based on monitoring points reduced the average CPU utilization
by 91.04% (6.7% → 0.6%) and the peak CPU utilization by 76.69%
(26.6% → 6.2%), while achieving rapid model inference within 30ms
and real-time response within 3 seconds. Furthermore, continuous
tests over 9 months demonstrated that the file behavior detector
effectively mitigates the overwhelming number of alerts triggered
by monitoring points (3, 192 → 750), avoiding 76.50% of the unex-
pected process interruption and required inspection efforts from
security analysts.

By the time of this writing, CanCal has been integrated into
a commercial product and successfully deployed on 3.32 million
endpoints for over a year. From March 2023 to April 2024, Can-
Cal has successfully detected and thwarted 61 ransomware attack
incidents spanning various industries, including manufacturing,
technology, services, and transportation. CanCal can effectively
detect both well-known and distinctive variants of ransomware, in-
cluding Mallox, Lockbit 3.0, and Tellyouthepass. Notably, according
to a manual forensic of 27 ransomware detected from March to June
2023, CanCal has defended against 13 n-day attacks that exploited
vulnerabilities documented in the CVE database and prevented 5
high-risk zero-day attacks, further demonstrating its ability to com-
bat unknown and sophisticated ransomware threats. Overall, Can-
Cal is highly effective in defending against real-world ransomware
attacks on user endpoints and can provide reliable protection for
various industries.

2 DESIGN DETAILS OF CANCAL

This section introduces the framework of “Monitoring-Detection-
Response” and the layered architecture of CanCal. The full work-
flow is shown in Figure 1.

Response StrategyDecoy FileDeploymentIRPLogs ofSuspiciousProcesstriggerReal-timeBehavior DetectionCanaryFileMonitorCanCalgenerateRansom NoteReleaseSimilarity Thresholdlow-riskanalysisdeliveryDecoyFileOperationswritedeleteencrypthigh-riskCCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

Monitoring: This process involves two components: (1) Decoy
File Monitor deploys decoy files and closely monitors suspicious
activities like deletion, writing, or encryption. If detected, further
analysis and detection are performed. (2) Ransom Note Monitor de-
tects the execution of ransomware. When the ransomware executes,
it releases a ransom note. CanCal analyzes the similarity of ransom
notes and assigns a threat level accordingly.
Detection: The detection component aims to identify ransomware
processes from the vast number of low-risk level threat alerts re-
ported by monitoring points. This engine collects IRP logs of suspi-
cious processes and combines expert-based features with automatic
embedding to capture the ransomware behavior pattern.
Response: For low-risk threat levels, CanCal relies on real-time
behavior detection to track the suspicious process. While for high-
risk threat levels, CanCal takes appropriate response measures to
mitigate the impact. This may involve terminating the process and
isolating the endpoint.

2.1 Lightweight Decoy File Monitor

CanCal proposes a real-time dynamic detection method based on
decoy files, which serves as the detection trigger layer to enable
the transition from passive acceptance of ransomware damage to
active enticing. By deploying carefully crafted decoy traps, CanCal
continuously monitors system activities and implements counter-
interception during the ransomware invasion phase.

2.1.1 Decoy File Design. In the context of ransomware detection,
the primary objective of decoy file design is to enhance their credi-
bility and access priority, thereby ensuring their interaction with
ransomware. A suboptimal design may lead to two undesirable out-
comes: either the ransomware identifies and circumvents the decoy,
compromising detection accuracy, or it prioritizes the encryption
of legitimate files before accessing the decoy, thus impairing the
timeliness of detection. To address these challenges, we have de-
veloped a holistic approach to decoy file design, encompassing file
names, types, and content.

• Decoy File Name and Type: To effectively deceive ransomware,
the naming convention for decoy files must be meticulously crafted.
Our approach aims to mimic the nomenclature of legitimate files as
closely as possible, minimizing the risk of detection by ransomware.
We prioritize information-rich file types for our decoys, such as
documents and images, which are typically high-value targets for
ransomware attacks.

• Decoy File Content: To bolster the credibility of our decoys,
we ensure that their content is contextually appropriate for their
purported file type. For instance, document-type decoy files are
populated with plausible, randomly generated text that aligns with
the expected content of such files.

2.1.2 Decoy File Deployment. CanCal is engineered to provide
real-time protection for end users by leveraging a simulated environ-
ment that closely mimics genuine hardware and software systems.
This approach is designed to provoke potential malicious activ-
ities and enhance interaction with ransomware threats, thereby
significantly reducing the likelihood of evasion. The system em-
ploys two distinct deployment strategies: automatic deployment
and user-defined placement.

Figure 2: An example of the ransom note.

• Automatic Deployment: This method involves a thorough anal-
ysis of typical ransomware file traversal patterns, with particular
attention to the initial directories accessed during these traversals.
Common targets, such as C:\Users\username\AppData, are iden-
tified and utilized as predefined locations for decoy file placement.
These strategically positioned decoys are then subject to real-time
monitoring. This approach enables the endpoint to swiftly detect
any modification attempts and implement immediate countermea-
sures to prevent ransomware propagation. By ensuring early access
to decoy files, this method substantially enhances overall system
security.

• User-Defined Placement: This technique empowers users to
customize the location of decoy files, thereby offering enhanced
protection for critical data. Users simply specify the desired number
of decoys and their intended locations within the file system. This
method strikes an optimal balance between the security benefits
of manual decoy placement and the efficiency of automated sys-
tems, significantly reducing the time required for internal system
maintenance.

2.1.3 Decoy File Monitoring. Upon any modification to a decoy
file, the system’s callback function initiates a filtering interception
logic. This triggers an alert within the system’s trigger layer, sig-
naling a potential security breach. To mitigate false positives and
ensure accuracy, the system doesn’t immediately classify this as a
ransomware attack. Instead, it prompts an in-depth ransomware
screening process, activating targeted process monitoring and com-
prehensive behavior collection for the suspect processes. The col-
lected data is then analyzed by CanCal’s sophisticated dynamic
behavioral engine for definitive threat discrimination.

2.2 Semantic-based Ransom Note Monitor

Once files are encrypted or devices are disabled, ransomware at-
tackers typically alert victims to the infection. The usual practice
is to use a pop-up notification or release a file (txt/html/exe, etc.)
deposited in a file directory or on the computer desktop. Figure 2
illustrates a typical ransom note issued by ransomware, wherein vic-
tims are instructed to remit payment in cryptocurrency to recover
their files. These ransom notes generally contain comprehensive

CanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

information about the ransomware attack, including encryption
status, ransom amount, and payment methods, often presented in
a consistent textual pattern. Leveraging this consistency, seman-
tic analysis of ransom notes can effectively detect the behavioral
characteristics of ransomware in real-time, thereby enhancing pre-
ventive measures against such attacks.

2.2.1 Ransom Note Content. Ransom notes typically delineate the
crypto-extortion scenario and outline the payment process, usually
demanding cryptocurrency or similarly untraceable methods in ex-
change for data recovery and system restoration. The fundamental
components of these notes generally include:

• Headline: Usually an eye-catching headline like “Your files are
encrypted” or “Your computer is locked” is employed to attract the
victim’s attention.

• Threat: The ransom note usually details the compromised state
of the victim’s computer or files, emphasizing their encrypted or
locked status. It often warns that failure to pay the ransom will re-
sult in permanent inaccessibility or corruption of files. Additionally,
threats of exposing the victim’s personal information and data are
common tactics.

• Payment Information: Attackers typically include a payment
link or cryptocurrency address, along with detailed instructions for
submitting the ransom.

• Deceptive Offers: To enhance their perceived credibility and
manipulate victims, attackers may present ostensibly benevolent
options such as “free decryption” or “file recovery” services.

• Contact Information: To further their deception, attackers often
provide contact details, such as an email address, under the guise
of offering support or facilitating ransom payment.

2.2.2 Ransom Note Analyzer. According to the basic composition
of ransom notes, it can be found that most of the expressions re-
volve around the specific behavior of ransomware attacks, such as
“encryption”, “decryption”, “file loss”, etc. Using natural language
processing technology, it is possible to automate the recognition
and matching of repetitive language structures and vocabulary. This
technology plays an important role in the real-time monitoring of
ransomware, enabling the immediate identification of potential
threats at the moment of ransom note release. Specifically, this
module can be divided into three steps: parsing of ransom notes,
gene pool creation, and similarity discrimination.
Step 1: Ransomware Parser. To capture the context of encryp-
tion and ransom, this module is designed with a parsing step. This
parsing can divide the long ransom text into small unit fragments
for analysis. Specifically, the word-level 𝑛 − 𝑔𝑟𝑎𝑚-based method
is used to construct a local continuous sequence. Given a ran-
som note of length 𝑘, 𝑊 = (𝑤1, 𝑤2, 𝑤3, · · · , 𝑤𝑘 ) , where 𝑤𝑖 de-
notes the 𝑖-th word in the text segment 𝑊 . A window of size 𝑛 is
slid over this text segment, and the 𝑛 words in the current win-
dow are selected as segments at each move. In this way, the set
of all 𝑛 − 𝑔𝑟𝑎𝑚 sequences are extracted, consisting of 𝑊𝑛𝑔𝑟𝑎𝑚 =
{(𝑤1, 𝑤2, · · · , 𝑤𝑛), · · · , (𝑤𝑘 −𝑛+1, 𝑤𝑘 −𝑛+2, · · · , 𝑤𝑘 )}, where the set
size is 𝑘 − 𝑛 + 1, each sequence (𝑤𝑖, 𝑤𝑖+1, · · · , 𝑤𝑖+𝑛−1) consists of
the adjacent 𝑛 words in the text segment 𝑊 .

In the n-gram method, the larger the selected value of 𝑛, the
richer the contextual information that the model can capture, but it

Figure 3: Sliding window of trigram method. It extracts three
consecutive text words in the ransom note.

also leads to the problems of subsequent matching errors. After an-
alyzing a set of real ransom note samples, we have determined that
the optimal 𝑛 value for our system is 3 (See Appendix A). It strikes
a balance between capturing contextual information and avoiding
subsequent matching errors. While larger 𝑛 values can capture
richer contextual information, they also may be more sensitive to
minor variations in phrasing, which could result in missed matches
for semantically similar but syntactically different expressions. Con-
versely, smaller 𝑛 values such as unigrams or bigrams may not
provide sufficient context for accurate recognition of ransomware-
specific language patterns. Specifically, Figure 3 shows an example,
which involves extracting sequences of three consecutive words.
When considering three adjacent words, the consecutive sequences
“all your files” and “have been encrypted” are extracted. They ap-
pear frequently as important linguistic features of ransom notes
and can express more complete ransom semantics. Therefore, we
choose 𝑛 = 3 as the optimal value to improve the ransom note
classification accuracy.
Step 2: Create a Ransomware Family Gene Pool. To analyze the
textual features of ransom notes, we conduct statistical analysis on
each parsed text sequence (𝑤𝑖, 𝑤𝑖+1, 𝑤𝑖+2). Specifically, we quan-
tify the frequency of occurrence 𝑐𝑖 for each sequence within the
entire set, as frequently appearing sequences likely represent key
information in the ransom note. We then normalize these frequency
counts to scores 𝑓𝑖 , which are arranged in descending order. This
process allows us to compare the relative importance of different
word sequences and identify the most prominent linguistic patterns.
This normalization process is mathematically expressed as:

𝑓𝑖 =

𝑐𝑖

(cid:205)𝑘 −𝑛+1
𝑚=1

𝑐𝑚

(1)

Based on these key fragments, we further construct a gene pool
of ransomware families for storing and managing the information in
ransom notes. This gene library contains all common key fragments
and their normalized values in different ransom notes. This curated
collection of linguistic features enhances our ability to efficiently
identify and detect potential ransom notes.
Step 3: Calculate Ransom Note Similarity. Once the ransomware
gene fragments are constructed, we can determine whether a given
file 𝑑 exhibits characteristics of ransom note by comparing its simi-
larity to these established gene fragments. Specifically, the sequence
fragments are extracted from the file using the same parsing method.
Then, we calculate the similarity score by summing the normalized

AllyourfileshavebeenencryptedAllyourfileshavebeenencryptedAllyourfileshavebeenencryptedAllyourfileshavebeenencryptedSource ransom textText segment(All, your, files)(your, files, have)(have, been, encrypted)(files, have, been)CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

Figure 4: An overview of the behavioral engine with multi-granularity features. The engine includes five main modules: (1)
Behavior log collection module: records the process activities; (2) Behavior graph construction module: converts the behavioral
operations into a two-part "instruction-parameter" behavior graph; (3) Behavior pattern encoding module: encodes each
behavioral operation into an embedded representation; (4) Expert knowledge feature module: extracts features based on expert
experience; (5) Classifier module: performs ransomware classification using the decision tree algorithm.

values 𝑓𝑖 of the 𝑡 gene fragments that successfully match:

𝑠𝑖𝑚(𝑑) =

𝑡
∑︁

𝑖=1

𝑓𝑖

(2)

To determine whether the current sample is a ransom note, we
compare its similarity score to a threshold 𝜏𝑠𝑖𝑚. This threshold is es-
tablished by analyzing the similarity of known samples to the gene
pool of ransomware families. A detailed analysis of the similarity
threshold and its determination can be found in Appendix A.

2.3 Multi-granularity Behavior Detector

The behavioral engine is an advanced AI-powered tool designed to
enhance the analysis and classification of ransomware. By lever-
aging both automatically collected logs and manually constructed
features, this engine significantly improves the performance of the
ransomware classifier. The overall architecture is shown in Figure 4,
and the specific implementation consists of the following steps:
Step 1: Collect Behavior Logs. CanCal automatically collects
process behavior logs, which include the sequence of system events
during execution and their corresponding parameters. These logs
provide detailed information about the behavior of each process.
Step 2: Construct Behavior Graphs. CanCal converts the be-
havioral operations from the logs into bipartite behavior graphs.
These graphs contain two types of vertices: event operations and
parameters. By drawing edges between each behavioral operation
and its parameters, the graph structure enables better extraction
of specific behavioral patterns (e.g., network communication, file
encryption modifications, etc.).
Step 3: Encode Behavior Patterns. Each behavior pattern is
compressed into a sparse one-hot vector, which indicates whether

a specific operation or parameter is present in each process. A
neural network then converts this vector into a compact pattern
embedding, facilitating the learning of more complex feature rep-
resentations and enhancing the performance of the ransomware
classifier.
Step 4: Combine Expert Knowledge. CanCal combines the au-
tomatically collected behavior embeddings with multi-granularity
behavior features derived from the expert system. These combined
features serve as the final input to the decision tree classifier.
Step 5: Train a Decision Tree Model. Using the collected data and
integrated features, CanCal trains a decision tree classifier to de-
termine whether a process is ransomware or not. The decision tree
classifier autonomously selects the optimal path for classification
based on the input feature vector.

Note that the expert knowledge features were derived from the
experience of an industrial ransomware emergency response team,
comprising 6 members with an average of 6-8 years of experience.
The selection process followed a rigorous approach, involving in-
dependent proposals, structured discussions, and consensus voting.
The following section will delve into the detailed design and ratio-
nale behind these expert-derived features.

2.3.1 Ransomware Encryption Mode. To reach the goal of hiding
itself, encrypting victim host files, and demanding ransom, ran-
somware usually performs frequent operations on files and presents
significant anomalous features. To address this issue, we focus on
file encryption behavior, and according to different I/O access poli-
cies, existing file encryption methods can be divided into two main
categories [18, 42]: (1) Overwrite: The ransomware reads the file
content and encrypts it, then writes it back to the original file

Automatically-collected feature embeddingManually-collected feature embeddingCanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

Figure 5: There are six distinct encryption modes, each characterized by different I/O access modes and encryption suffix
names: (1) The attacker creates an encrypted file with the same suffix name and overwrites the original file. (2) This mode is
distinguished from Mode#1 by creating encrypted files with random suffixes. (3) The attacker creates an encrypted file with the
same suffix name and deletes the original file. (4) This mode is distinguished from Mode#3 by creating encrypted files with
random suffix names. (5) The attacker creates an encrypted file with the same suffix name and smashes the original file. (6)
This mode is distinguished from Mode#5 by creating encrypted files with random suffix names.

directly; (2) Create and Delete: The ransomware creates a new ci-
phertext and then deletes the original text; (3) Create and Smash:
The ransomware creates a new ciphertext and then smashes the
original text with random data.

2.3.2 Expert Knowledge Feature Engineering. The essence of feature
engineering is to extract the maximum number of features for use in
the model. Taking the encryption of important files as an example,
different encryption strategies use different file operations, which
can be characterized by analyzing the involved file operations for
behavioral characterization.

The process is the basic unit of ransomware scheduling opera-
tions. When the suspicious process 𝑝 triggers the monitoring point
𝑚, the file operations are recorded during the duration period Δ𝑡.
Specifically, the operation of each process consists of elements such
as timestamp, process number, and file name, in the form of a five-
tuple, <Time,Pid,Pid_name,Operation,File_name, File_type>.
Based on this information, the number of file changes in the mon-
itored time period is counted, and the characteristics of different
levels are fused to build a feature set of file operations. This feature
set can be specifically divided into three categories:
Coarse-grained Features Based on File Operations. This type
of feature is mainly based on the operation behavior of file creation,
file deletion, file renaming, etc. We count the number of file changes
in process-based units. Specifically, by monitoring the behavior of
processes in the system, the number of changes over a period of
time is recorded and counted. The following are the definitions of
this type of feature:

Def 1. Number of files created 𝑛𝑐𝑟𝑒𝑎𝑡𝑒 . Ransomware creates a

large number of files to store ransom notes or encrypted.

Def 2. Number of deleted files 𝑛𝑑𝑒𝑙𝑒𝑡𝑒 . Ransomware deletes the
original files during encryption to prevent users from recovering them.

Def 3. Number of renamed files 𝑛𝑟𝑒𝑛𝑎𝑚𝑒𝑑 . The ransomware re-

names or modifies the extension of files during encryption.

Fine-grained Features Based on Different Encryption Modes.
During the encryption process, attackers often add or modify file
suffixes, resulting in changes to the file type that make it difficult
to access. In ransomware-initiated encryption, suffix modifications
can occur in two scenarios: first, by adding the same file suffix to
all target files, such as the name of the ransomware family; and
second, by randomly replacing the file suffix with a different string.
To account for the different characteristics of I/O modes and suffix
names, this study divides encryption operations into six modes,
which are defined as follows. Figure 5 illustrates the six modes of
encryption operations.

• Mode#1: In this mode, the original file is overwritten with the
encrypted file, and the same suffix name is added uniformly to all
affected files.

• Mode#2: This mode involves the encrypted file overwriting
the original file, with different suffix names added randomly during
the encryption process.

• Mode#3: This mode involves deleting the original file and
creating a new encrypted file, with the same suffix name added
uniformly to all affected files.

• Mode#4: This mode also involves deleting the original file and
creating a new encrypted file, but with different suffix names added
randomly during the encryption process.

• Mode#5: This mode involves smashing the original file and
creating a new encrypted file, with the same suffix name added
uniformly to all affected files.

Mode#3: Create with the same suffixthen deleteransom.exedeleteFileransom.exeFile.lockedreadcreate and writeEncrypt dataransom.exereadencryptFileEncrypt dataMode#4: Create with the random suffixthen deleteransom.exedeleteFileransom.exeFile.randomreadcreate and writeEncrypt dataransom.exereadencryptFileEncrypt dataMode#1: Overwrite with the same suffixransom.exerenameFileransom.exeFile.lockedreadoverwriteEncrypt dataransom.exereadencryptFileEncrypt dataFileMode#2: Overwrite with the random suffixransom.exerenameFileransom.exeFile.randomreadoverwriteEncrypt dataransom.exereadencryptFileEncrypt dataFileMode#5: Create with the same suffixthen smashransom.exesmashFileransom.exeFile.lockedreadcreate and writeEncrypt dataransom.exereadencryptFileEncrypt dataMode#6: Create with the random suffixthen smashransom.exesmashFileransom.exeFile.randomreadcreate and writeEncrypt dataransom.exereadencryptFileEncrypt dataCCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

• Mode#6: This mode also involves smashing the original file
and creating a new encrypted file, but with different suffix names
added randomly during the encryption process.

When the process 𝑝𝑖 triggers any monitoring point, the number
of file types 𝑛𝑡𝑦𝑝𝑒, 𝑛𝑡𝑦𝑝𝑒′ before and after the monitoring time
period Δ𝑡 are recorded respectively. To further analyze and identify
different encryption behavior features, we count the increase or
decrease in the number of file types. Although six encryption modes
are described, from the perspective of 𝑛𝑡𝑦𝑝𝑒𝑐ℎ𝑎𝑛𝑔𝑒 , these modes
actually result in four distinct features, which can be calculated as:
𝑛𝑡𝑦𝑝𝑒𝑐ℎ𝑎𝑛𝑔𝑒 = 𝑛𝑡𝑦𝑝𝑒′ − 𝑛𝑡𝑦𝑝𝑒
(3)
Fusion-based Features. We employ a multi-feature fusion ap-
proach to leverage the complementary advantages of diverse fea-
tures, which enhances the accuracy and reliability of our feature
set, thereby improving ransomware classification capabilities.

Def 4. Ratio of file types (𝑟𝑡𝑦𝑝𝑒): This feature represents the rela-
tive change in file types before and after the monitoring period. Let
𝑛𝑡𝑦𝑝𝑒 and 𝑛𝑡𝑦𝑝𝑒′ denote the number of file types before and after the
monitoring period, respectively.

𝑟𝑡𝑦𝑝𝑒 =

𝑛𝑡𝑦𝑝𝑒′
𝑛𝑡𝑦𝑝𝑒

(4)

Def 5. Ratio of deleted to created file types (𝑟𝑡𝑦𝑝𝑒𝑐ℎ𝑎𝑛𝑔𝑒 ): This
feature aids in identifying ransomware with different encryption
modes by analyzing file type changes.

During the monitoring period Δ𝑡, we record 𝑛𝑡𝑦𝑝𝑒𝑑𝑒𝑙 (the num-
ber of deleted file types) and 𝑛𝑡𝑦𝑝𝑒𝑐𝑟𝑒𝑎𝑡𝑒 (the number of created
file types). The ratio is then computed as:

𝑟𝑡𝑦𝑝𝑒𝑐ℎ𝑎𝑛𝑔𝑒 =

𝑛𝑡𝑦𝑝𝑒𝑑𝑒𝑙
𝑛𝑡𝑦𝑝𝑒𝑐𝑟𝑒𝑎𝑡𝑒

(5)

Def 6. Maximum number of created files with the same name
(𝑚𝑎𝑥 (𝑛𝑓 𝑖𝑙𝑒 )): This feature helps identify ransomware behavior, as
these malicious programs often distribute multiple ransom notes with
identical filenames.

During Δ𝑡, we count the occurrences 𝑛𝑓 𝑖𝑙𝑒 of each unique file-

name and determine the maximum value 𝑚𝑎𝑥 (𝑛𝑓 𝑖𝑙𝑒 ).

Def 7. Number of folders containing the most files with the same
name (𝑛𝑓 𝑜𝑙𝑑𝑒𝑟 ): This feature distinguishes ransomware behavior from
normal file operations.

For files whose creation count reaches 𝑚𝑎𝑥 (𝑛𝑓 𝑖𝑙𝑒 ), we analyze
their distribution across different folders during Δ𝑡. While benign
software typically creates files with the same default filename in
a single directory, ransomware often distributes identical ransom
notes across multiple directories.

Def 8. Ratio of file creation with the same name (𝑟 𝑓 𝑖𝑙𝑒 ): This
feature combines the above two metrics to provide a comprehensive
view of file creation patterns.

𝑟 𝑓 𝑖𝑙𝑒 =

𝑚𝑎𝑥 (𝑛𝑓 𝑖𝑙𝑒 )
𝑛𝑓 𝑜𝑙𝑑𝑒𝑟

(6)

Further insights into the design of these expert knowledge fea-

tures are provided in Appendix B.

Algorithm 1: Decision Tree Generation
Input: Training dataset 𝐷 = (𝑥1, 𝑦1 ), (𝑥2, 𝑦2 ), . . . , (𝑥𝑚, 𝑦𝑚 ),

Feature set 𝐴 = 𝑎1, 𝑎2, . . . , 𝑎𝑛

Output: Decision tree 𝑓 (𝑥 )
1 Function TreeGenerate(𝐷, 𝐴):
𝑛𝑜𝑑𝑒 ← new Node()
2
if ∀𝑖 ∈ {1, 2, . . . , 𝑚} : 𝑦𝑖 = 𝑐 then

3

𝑛𝑜𝑑𝑒.𝑐𝑙𝑎𝑠𝑠 ← 𝑐
return 𝑛𝑜𝑑𝑒

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

end
if 𝐴 = ∅ ∨ ∀𝑖 ∈ {1, 2, . . . , 𝑚} : 𝑥𝑖 = 𝑇 then

𝑛𝑜𝑑𝑒.𝑡 𝑦𝑝𝑒 ← leaf
𝑛𝑜𝑑𝑒.𝑐𝑙𝑎𝑠𝑠 ← arg max𝑐 ∈𝐶 (cid:205)𝑛
return 𝑛𝑜𝑑𝑒

𝑖=1 [𝑦𝑖 = 𝑐 ]

end
𝑎∗ ← arg max𝑎∈𝐴 InformationGain(𝐷, 𝑎)
foreach 𝑎𝑣
∗ ∈ Domain(𝑎∗ ) do

𝑐ℎ𝑖𝑙𝑑 ← new Node()
𝐷𝑣 ← { (𝑥, 𝑦) ∈ 𝐷 : 𝑥𝑎∗ = 𝑎𝑣
∗ }
if 𝐷𝑣 = ∅ then

𝑐ℎ𝑖𝑙𝑑.𝑡 𝑦𝑝𝑒 ← leaf
𝑐ℎ𝑖𝑙𝑑.𝑐𝑙𝑎𝑠𝑠 ← arg max𝑐 ∈𝐶 (cid:205)𝑛

𝑖=1 [𝑦𝑖 = 𝑐 ]

else

𝑐ℎ𝑖𝑙𝑑 ← TreeGenerate(𝐷𝑣, 𝐴 \ {𝑎∗ })

end
𝑛𝑜𝑑𝑒.addChild(𝑎𝑣

∗, 𝑐ℎ𝑖𝑙𝑑 )

end
return 𝑛𝑜𝑑𝑒

24
25 end

2.3.3 Attack Detection Classifier. In large-scale industrial environ-
ments, we exclude computationally demanding deep learning ap-
proaches due to performance constraints and limited GPU resources.
Adopting a traditional machine learning model was a pragmatic
decision. In practice, CanCal leverages the Gradient Boosting De-
cision Tree (GBDT) algorithm [13] for the attack detection process,
specifically implemented using XGBoost [8] (the comparative ex-
periment details are provided in Appendix C). By combining the
advantages of both decision tree [6] and gradient boosting mod-
els [37], a classifier for ransomware is built. Following the concept
of ensemble learning, a group of weak learners are integrated into a
strong learner. When classifying benign software and ransomware,
the decision tree comprises nodes and directed edges, where in-
ternal nodes represent features and leaf nodes represent software
categories. The main principle of the GBDT approach is to use
gradient descent to approximate each decision tree. Specifically,
each iteration minimizes training errors along the negative gradient
direction.

Consider a training set 𝐷 = {(𝑥1, 𝑦1) , (𝑥2, 𝑦2) , · · · , (𝑥𝑚, 𝑦𝑚)}
comprising 𝑚 samples, where 𝑥𝑖 =
represents
the 𝑛-dimensional feature vector of the 𝑖-th sample, and 𝑦𝑖 ∈ {0, 1}
denotes its corresponding category label. The decision tree genera-
tion process is outlined in Algorithm 1. This algorithm partitions
the input space into disjoint regions. Specifically, it selects the 𝑗-
th dimensional feature 𝑎∗ from the 𝑛-dimensional feature space

𝑖 . · · · , 𝑥𝑛
𝑖

(cid:16)
𝑖 , 𝑥 2
𝑥 1

(cid:17)T

CanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

and determines a criterion value 𝑎𝑣
allocated to either the left subtree 𝑅1 (𝑎∗, 𝑎𝑣
𝑅2 (𝑎∗, 𝑎𝑣
is less than or equal to 𝑎𝑣
optimal division feature 𝑎∗ and its corresponding threshold 𝑎𝑣
minimize the prediction error of both subtrees:

∗ for division. Samples are then
∗) or the right subtree
∗), based on whether their 𝑗-th dimensional feature value
∗. The training objective is to identify the
∗ that

min
𝑎∗,𝑎𝑣
∗

min

∑︁

𝑥𝑖 ∈𝑅1








(cid:0)𝑦𝑖 − 𝑦

1

(cid:1)2 + min

∑︁

(cid:0)𝑦𝑖 − 𝑦

(cid:1)2

2

𝑥𝑖 ∈𝑅2
(cid:110)
𝑥 | 𝑥 (𝑎∗ ) ≤ 𝑎𝑣
∗

(cid:111)








(7)

(cid:111)

and

∗) =

∗) =
(cid:110)
𝑥 | 𝑥 (𝑎∗ ) > 𝑎𝑣
∗

where the divided left subtree is 𝑅1 (𝑎∗, 𝑎𝑣
the right subtree is 𝑅2 (𝑎∗, 𝑎𝑣
. The average of
∗)(cid:9),
the label values of the left subtree 𝑦
∗)(cid:9).
and the corresponding right subtree 𝑦
XGBoost extends this objective by incorporating a regularization
term Ω(𝑓 ) to control model complexity, where Ω(𝑓 ) = 𝛾𝑇 + 1
𝜆∥𝑤 ∥2,
2
with 𝑇 representing the number of leaves in the tree and 𝑤 denoting
the leaf weights.

1 = ave (cid:8)𝑦𝑖 | 𝑥𝑖 ∈ 𝑅1 (𝑎∗, 𝑎𝑣
2 = ave (cid:8)𝑦𝑖 | 𝑥𝑖 ∈ 𝑅2 (𝑎∗, 𝑎𝑣

The computation process for selecting optimal division criteria
can be further optimized using Equation 8, reducing the time com-
plexity to O (𝑛𝑚2). Notably, the feature dimension 𝑛 is 12, which is
significantly smaller than the number of samples 𝑚.

min
𝑎∗,𝑎𝑣
∗

min







∑︁

∑︁

𝑥𝑖 ∈𝑅1

(cid:0)𝑦𝑖 − 𝑦

1

(cid:1)2 + min

∑︁

𝑥𝑖 ∈𝑅2

(cid:0)𝑦𝑖 − 𝑦

(cid:1)2

2








(8)

=

𝑥𝑖 ∈𝑅

𝑦2
𝑖 − min
𝑎∗,𝑎𝑣
∗

∑︁

(cid:169)
(cid:173)
𝑥𝑖 ∈𝑅1
(cid:171)

1
𝑚1

𝑦2
𝑖 +

∑︁

𝑥𝑖 ∈𝑅2

1
𝑚2

𝑦2
𝑖 (cid:170)
(cid:174)
(cid:172)

This optimization approach employs functional gradient descent,
applying the steepest descent step to the minimization problem.
The objective is to locate a local minimum of the loss function
by iterating on the previous function value, denoted as 𝑓𝑡 −1 (𝑥).
Consequently, the local maximum-descent direction of the loss
function is represented by its negative gradient.

3 EVALUATION

To comprehensively evaluate CanCal on ransomware detection, we
conducted a series of experiments to answer the following research
questions:

• RQ1 (Effectiveness): How do the quantitative results of the

training process of the CanCal?

• RQ2 (Comparison): How does the CanCal perform in de-
tecting threats on public datasets, and how does its performance
compare to other state-of-the-art solutions?

• RQ3 (Ablation Study): How does each component affect the

performance of CanCal separately?

• RQ4 (Industrial Deployment): How well does CanCal per-
form when it is deployed in real-world industrial environments to
defend against ransomware attacks?

3.1 Experiment Setup

To conduct the experiments of RQ1~3, we simulate the operating
system environment using virtual sandbox technology. The virtual
machine is based on the Windows operating system and is equipped

Table 1: Training & Testing dataset statistics.

Dataset Components
Known ransomware samples
Unknown ransomware samples
Normal samples
Known ransomware behavior data
Unknown ransomware behavior data
Normal behavior data

Quantity
1,335
426
1,768
1,602,775
555,678
1,885,490

with the necessary software and configuration environment. To
ensure the security and stability of the virtual machine, we apply the
snapshot and check mechanisms. In the event that a ransomware
sample attempts to launch an attack, the system could be rolled
back to the initial state by starting a snapshot. This setup allows us
to perform dynamic analysis of samples in a more secure manner.
For RQ4, we deployed the prototype of CanCal on 3.32 million
end users to detect ransomware over three months (from March to
June) to demonstrate its usefulness in a real industrial environment.

3.1.1 Datasets. To ensure high-quality evaluation, we carefully
curate two comprehensive datasets, one for training and testing,
including both ransomware and benign samples, and the other
for benchmarking performance comparison, containing only ran-
somware samples.
Training & Testing Dataset. To assess the effectiveness of the
training process, we curate a diverse dataset consisting of ran-
somware samples and benign files. We collect 1,335 ransomware
samples based on real cases from Bazaar2 and VirusTotal3, cover-
ing the period from January 2018 to December 2022. These sam-
ples are used to form the training dataset. To evaluate CanCal’s
generalization performance on previously unseen and unknown
ransomware variants, we curate a distinct test dataset comprising
426 actively exploited ransomware samples from the wild. These
samples, entirely separate from our training set, represent current
threats that are still evading detection by all engines on VirusTo-
tal. These elusive ransomware variants exemplify zero-day attacks,
where adversaries exploit vulnerabilities before they become pub-
licly known, making them particularly challenging to identify and
mitigate. For normal samples, we obtain 1,768 commonly used
client-side and industry-specific applications (office, medical, and
design software) and collect their behavioral characteristics through
a sandbox virtual execution engine and operating system environ-
ment simulation. We deliberately exclude benign samples such as
zip, encryption, and backup software to avoid behaviors similar to
ransomware. The specific statistics are shown in Table 1.
Monthly Public Dataset. To conduct a comparative study for
RQ2, we collect a monthly public dataset comprising real-world
ransomware samples reported by users and from open-source web-
sites. This dataset consists of 250 ransomware attacks targeting user
endpoints, spanning a period from February to June 2023. These
samples include some common ransomware families such as Pho-
bos, LockBit, and Conti, which have been widely documented and

2https://bazaar.abuse.ch
3https://www.virustotal.com

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

Table 2: Overall experimental performance.

Metric
Training time
Model size
Inference time
Maximum response time
Maximum file loss
TPs / TPR on training dataset
FNs / FNR on training dataset
TPs / TPR on test dataset
FNs / FNR on test dataset
FP / FPR

Result
10s
29.3KB
30ms
3s
1.21%
1,599,994 / 99.83%
2,781 / 0.17%
553,740 / 99.65%
1,938 / 0.35%
7 / 3.71𝑒 −6

observed in the wild. Additionally, the dataset incorporates twelve
classes of emerging ransomware families such as PayMe100USD.
These emerging families conceal their behavioral characteristics
through process injection, making it challenging for security tools
to differentiate between benign and malicious processes, and ulti-
mately hindering their ability to detect these stealthy attacks.

3.1.2 Evaluation Metric. To evaluate the performance, we use the
following effectiveness metrics: True Positives (TPs), False Posi-
tives (FPs), True Negatives (TNs), and False Negatives (FNs). The
specific metrics for evaluation are:

• True Positive Rate (TPR) represents the percentage of ran-

somware attacks that are correctly detected and identified.

• False Positive Rate (FPR) measures the percentage of benign

files that are incorrectly classified as ransomware.

• True Negative Rate (TNR) represents the percentage of benign

files that are correctly identified as non-ransomware.

• False Negative Rate (FNR), on the other hand, measures the
percentage of ransomware attacks that are incorrectly classified as
legitimate by the detection system.

3.2 RQ1: Effectiveness
Real-time Response. By enabling the protection of CanCal, we
execute samples in a sandbox environment and collect behavioral
data on suspicious processes that trigger monitoring points. The
data collection utilizes a sliding window of 1 second, allowing for
granular analysis of process behavior over time. As shown in Ta-
ble 2, CanCal demonstrates exceptional efficiency, with an average
model inference time of merely 30ms. This rapid response ensures
that any potential ransomware threat can be detected and isolated
within up to 3 seconds after initial execution. Even when confronted
with LockBit, currently the fastest ransomware [10], CanCal can
successfully prevent the vast majority of file encryption, limiting
the overall file loss to a mere 1.21%. Considering the average en-
cryption speed reported by Splunk [10], user file loss can even be
minimized to a mere 0.12%.
Lightweight Framework. We employ a lightweight, funnel-type
detection framework for CanCal, designed to strike a balance be-
tween effectiveness and efficiency. As shown in Table 2, the training
process for this framework is remarkably fast, taking only 10 sec-
onds to complete. Moreover, the resulting model has a compact size

of merely 29.3KB, making it highly portable and resource-efficient.
The low computational requirements and minimal model size of
our funnel-type detection framework make CanCal particularly
well-suited for deployment on endpoint devices, without imposing
significant overhead or compromising system performance.
High Precision for Zero-Day Ransomware Detection. We lever-
age a multi-layered approach, employing various behavioral en-
gines to filter and identify malicious threats efficiently. For known
ransomware samples in the training dataset, CanCal boasts an
impressive TPR of 99.83%, showcasing its ability to identify and
mitigate previously documented ransomware variants accurately.
Remarkably, even when faced with these unknown ransomware
or novel variants in the unseen test dataset, CanCal maintains
a high TPR of 99.65%. This outstanding performance underscores
the system’s exceptional threat disposal capability, highlighting
its ability to adapt and effectively identify emerging ransomware
threats that have not been encountered before.
Near-Zero False Positives. Existing anti-virus software systems
often generate a significant number of false ransomware alerts,
which lack credibility and reliability. These false positives not only
increase the workload of security analysts but also disrupt normal
business operations by flagging legitimate software as potential
threats. In contrast, CanCal’s ability to maintain a near-zero false
positive rate is particularly impressive, especially when considering
the vast amount of benign software behavior data it has to analyze.
As shown in Table 2, CanCal correctly classifies an astonishing
1,885,483 events of legitimate software, with only 7 FPs, resulting
in an extremely low FPR of 3.71𝑒 −6. By minimizing false positive
alerts, CanCal significantly reduces the noise and unnecessary
interruptions that often plague traditional anti-virus systems.

3.3 RQ2: Comparison with SOTA Techniques

To further demonstrate the superiority of CanCal, we conduct
a comparative study against state-of-the-art (SOTA) ransomware
detection tools from both academia and industry.
Comparison with Academic Solutions. We compare CanCal
with five SOTA academic solutions, i.e., Unveil [18], Redemption [19],
ShieldFS [9], RWGuard [25], and RTrap [14]. The comparison re-
sults are shown in Table 3. Note that as the SOTA research systems
are not publicly available, we could not evaluate them in a real
experimental environment. Therefore, we collect public evaluation
data of the research systems from their publications and then make
a comparison with CanCal. As shown in Table 3, we compare
various key performance metrics, including dataset size, real-time
capability, file loss, runtime overhead, effectiveness, and capabil-
ity to combat bypasses. Compared to solutions relying solely on
file behavior monitoring, such as ShieldFS, CanCal significantly
reduces the runtime overhead associated with comprehensive pro-
cess behavior monitoring while offering superior effectiveness,
particularly in terms of the FNR. Although Unveil and Redemption
reported near-zero FNR in their original studies, the reproduced
results from MarauderMap indicate their poor generalization capa-
bilities on the latest ransomware dataset. In contrast to the latest
advancements in bait-based ransomware detection, CanCal intro-
duces a novel approach that combines ransomware note semantic
detection and file behavior monitoring. This integrated approach

CanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

Table 3: Comparison with SOTA academic solutions. Note that DFM represents decoy file monitor; RNM represents ransom
note monitor; and FBD represents file behavior detection.
represents
denotes the SOTA techniques.
that the tool supports this feature, but there is a gap when compared to the SOTA techniques;

represents that the tool does not support this feature;

(cid:72)(cid:35)

(cid:35)

(cid:32)

Solutions

Dataset

Realtime

Size

Detection

File-Loss

Runtime

Effectiveness

Approach

Anti-

Overhead

FPR

FNR

DFM RNM FBD

Bypass

Unveil [18]

Redemption [19]

ShieldFS [9]

RWGuard [25]

RTrap [14]

CanCal

319

677

305

14

1,106

1,761

(cid:35)

(cid:35)

(cid:35)
(8.87s)

(cid:72)(cid:35)
(5.35s~11.00s)

/

/

/
2.79%♣

0.17%

/

2.80%~8.70%

30%~380%

1.90%

/

(cid:72)(cid:35)

(<3.00s)

<0.12%~1.21%

0.60%

0.00%

1.00%

0.00%

0.10%
77.34%★
0.17%

41%♣
69%♣

2.30%

0.00%

/

0.35%

(cid:35)

(cid:35)

(cid:35)

(cid:32)

(cid:32)

(cid:35)

(cid:35)

(cid:35)

(cid:35)

(cid:35)

(cid:32)

(cid:32)

(cid:32)

(cid:32)

(cid:35)

(cid:35)

(cid:35)

(cid:35)

(cid:72)(cid:35)

(cid:32)

♣ These data marked with ♣ denote results that have been reproduced by the latest studies, rather than the original reported results. The FNR of

(cid:32)

(cid:32)

(cid:32)

(cid:32)

(cid:32)

Unveil and Redemption have been reproduced by MarauderMap [17]; the file-loss of RWGuard has been reproduced by RTrap [14].

★ Due to the fact that RTrap is a solution solely based on DFM, the calculation method for its FPR is different from others. So we calculate the

FPR of decoy files based on a continuous deployment in an industrial environment for 9 months.

Table 4: Monthly performance test when compared with SOTA industrial solutions.

Month

Technique

Quantity

CanCal

Solution A

Solution B

Solution C Solution D

Feb-2023

Non-process Injection

33 (6 families)

Mar-2023

Non-process Injection

49 (5 families)

32 (96.97%)

48 (97.96%)

30 (90.91%)

20 (60.61%)

31 (93.94%)

31 (93.94%)

47 (95.92%)

29 (59.18%)

45 (91.84%)

43 (87.76%)

Apr-2023

May-2023

Jun-2023

Non-process Injection

25 (10 families)

25 (100.00%)

25 (100.00%)

18 (72.00%)

20 (80.00%)

22 (88.00%)

Process Injection

31 (5 families)

31 (100.00%)

31 (100.00%)

21 (67.74%)

0 (0.00%)

2 (6.45%)

Non-process Injection

36 (9 families)

36 (100.00%)

36 (100.00%)

36 (100.00%)

33 (91.67%)

35 (97.22%)

Process Injection

12 (6 families)

Non-process Injection

50 (5 families)

Process Injection

14 (6 families)

12 (100.00%)

50 (100.00%)

14 (100.00%)

10 (83.33%)

6 (50.00%)

4 (33.33%)

4 (33.33%)

48 (96.00%)

46 (92.00%)

42 (84.00%)

36 (72.00%)

12 (85.71%)

9 (64.29%)

5 (35.71%)

7 (50.00%)

1 To mitigate the impact of virus signature updates, we freeze the versions of all tested solutions as of February 2023.

effectively filters out false positives that may arise from bait trig-
gers in solutions like RTrap, while also addressing the vulnerability
of RWGuard to evasion techniques that exploit file access priority
policies. Furthermore, CanCal excels in real-time detection capa-
bilities, outperforming bait-based academic solutions in terms of
responsiveness.
Comparison with Industry Solutions. To further evaluate the
effectiveness of CanCal, we conduct monthly performance tests.
By comparing the system detection rates for different months, we
can track the effectiveness of our model over time and identify
any potential weaknesses or areas for improvement. We select
international cutting-edge endpoint security vendors Solution A;
well-known Chinese domestic vendors Solution B, Solution C, and
Solution D to compare the ransomware protection performance.
We calculate the system detection rate according to the month
of ransomware attack events. The statistical results are shown in
Table 4. Detailed test results of each family from February to June
2023 can be found in Appendix D. The results show that our model
demonstrates competitive performance with Solution A, the world’s

leading anti-virus software, in blocking the majority of threats in
non-process injection ransomware through scanning and detection.
In process injection ransomware, our model achieves a high-risk
virus detection rate of 100% with high accuracy, by using fine-
grained tracing of the parent process and its context, combined with
the ransomware decoy protection engine, to terminate abnormal
processes immediately. Overall, our continuous test on monthly
public datasets demonstrates the competitive performance of our
model in detecting both known and unknown ransomware threats.
By doing so, we can ensure that our defense systems remain up-to-
date and effective in the face of evolving threats.

3.4 RQ3: Ablation Study

To gain a deeper understanding of the individual contributions of
the key components in CanCal’s ransomware detection pipeline,
we conduct an ablation study. This systematic analysis involves
selectively disabling or removing specific modules or techniques
within the pipeline and evaluating the impact on overall perfor-
mance. Specifically, we evaluate the following configurations:

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

Table 5: Resource utilization comparison of Variant#1.

3.5 RQ4: Industrial Deployment

Resource

Variant#1 CanCal

Average CPU Utilization

Peak CPU Utilization

Disk Consumption

Running Time

6.70%

26.60%

78MB

30s

0.60%

6.20%

Negligible

30s

Figure 6: Performance of Variant#2 w/o FDB.

Variant#1 (w/o DFM and RNM): In this configuration, we disable
the decoy file monitor and ransom note monitor module. In the
absence of monitoring points, Variant#1 have to monitor the file
access behavior of all processes, resulting in significant overhead
and resource consumption. The provided data in Table 5 highlights
the stark contrast in resource utilization between scenarios with
and without monitoring points. When logging all file behavior
without monitoring points on a standard office computer (Win10,
2 CPU cores, 4GB RAM, 30GB HDD), the average CPU utilization
soars to 6.70%, with peak levels reaching a staggering 26.60%. This
substantial CPU usage can potentially impact the overall system per-
formance and responsiveness, especially on resource-constrained
office systems.
Variant#2 (w/o FBD): In this variant, we remove the file behavior
detector component, which tracks and analyzes changes to file
systems, such as encryption, deletion, or modification of user data.
As shown in Figure 6, Variant#2, which excludes the file behavior
detector component, exhibits a significant increase in false positives
across multiple months. The data provided indicates that from
January 2023 to September 2023, the false positive rate for Variant#2
remained consistently high, ranging from 75.88% to 89.72% for
most months. With the FBD component in place, CanCal avoids
76.50% of the inspection efforts required from security analysts,
reducing the number of alerts that need manual review from 3,192
to 750 (per endpoint). This dramatic reduction in false positives not
only enhances the accuracy of the detection system but also allows
security personnel to focus on genuine threats rather than being
inundated with false alarms.

By the time of this writing, CanCal has been successfully deployed
on 3.32 million end users and has been running steadily for over a
year. As shown in Appendix E , CanCal has successfully detected
61 ransomware attacks that occurred from March 2023 to April
2024, which are directed toward diverse industrial enterprises, en-
compassing manufacturing, technology, services, transportation,
and additional domains. Specifically, CanCal has defended against
both well-known and distinctive variations of ransomware, includ-
ing Mallox, Lockbit 3.0, and Tellyouthepass. For further analysis,
we conducted a manual forensic analysis of 27 ransomware attacks
detected from March to June 2023. These case studies showcase
that CanCal can identify and defend a diverse range of intrusive
techniques employed by ransomware, including the utilization of
disguised files, the exploitation of vulnerabilities, and process injec-
tion. Notably, it successfully thwarted 13 attacks leveraging n-day
vulnerabilities documented in the CVE database, such as XVE-2023-
3377 and QVD-2023-14179, as well as 5 high-risk attacks exploiting
zero-day vulnerabilities. This highlights the exceptional capabil-
ity of CanCal for addressing unknown and sophisticated threats.
Overall, these results demonstrate that CanCal is highly effective
in defending against real-world ransomware attacks on user end-
points and can provide reliable protection for various industries.

4 DISCUSSION
Potential Evasion Strategies In this study, CanCal relies on
fixed monitoring points to trigger the subsequent behavioral en-
gine. However, attackers may attempt to evade detection by the
system. One potential evasion strategy is to avoid triggering moni-
toring points. Attackers may choose not to release ransom notes,
modify decoy files, or perform file operations. In this particular
situation, CanCal is unable to detect effectively. We do recognize
that ransomware detection is a dynamic adversarial process with
no perfect solution. Indeed, this dynamic nature of the ransomware
threat landscape motivates CanCal’s modular, multi-layered de-
sign. While this paper focuses on decoy files and ransom notes, the
actual engineering deployment could include additional strategies
to counteract evasion tactics, such as monitoring magic number
changes in file headers or file entropy changes before and after
encryption. Although sophisticated evasion techniques may chal-
lenge individual monitoring methods, the multi-layered approach
of CanCal significantly raises the bar for successful attacks.
Response Measures for Ransomware Incidents During the
post-incident response phase, CanCal is capable of intercepting
and blocking the ransomware, which involves terminating the en-
cryption process and isolating the affected host from the network.
While this initial intervention provides immediate containment,
it does not fully mitigate the underlying threat. The attacker may
retain remote control over the compromised endpoint, creating a
persistent access vector that enables potential repeated execution
of the ransomware payload. One possible solution is to terminate
potential remote control services and enable two-factor authentica-
tion. This approach would significantly reduce the impact on user
endpoints, particularly when compared to the current strategy of
host isolation.

2023-012023-022023-032023-042023-052023-062023-072023-082023-090100200300400500600Number of Alerts30405060708090100False Positive Rate (%)True PositivesFalse PositivesFalse Positive RateCanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

5 RELATED WORK

Ransomware detection and response systems aim to identify and
mitigate ransomware attacks on endpoints. These systems typically
employ a combination of static analysis, behavior-based analysis,
and decoy-based techniques.
Static Analysis. The static detection method refers to analyzing
static attributes such as byte sequences and opcodes without run-
ning the sample, extracting features from them, and training de-
tection models. Medhat et al. [24] extract a series of static features
(including file keywords, API functions, cryptographic signatures,
file extensions, etc.). Sheen et al. [35] summarize the frequency
pattern of API file calls in malware. Yamany et al. [41] focus on
raw ransomware binaries. They extract static feature sets, such as
import address tables and strings, to identify common features and
cluster similar samples. Zhu et al. [43] use more fine-grained code
analysis to directly calculate the entropy features in ransomware
binaries. However, as technology evolves, attackers employ various
methods such as compression, encryption, shelling, and obfusca-
tion to prevent the extraction of static features. Aghakhani et al. [1]
demonstrate that machine learning classifiers based on static anal-
ysis features are limited in their ability to generalize to unseen
packers and are susceptible to adversarial examples, leading to false
positives and potential mislabeling of ground-truth data. While
these static analysis approaches provide valuable insights, they are
increasingly vulnerable to evasion techniques employed by modern
ransomware. In contrast, CanCal adopts a dynamic approach that
combines behavior-based and decoy-based techniques, offering a
more robust and adaptive solution.
Behavior-based Dynamic Analysis. Behavior-based dynamic
analysis monitors system activities during program execution to
detect ransomware. Scaife et al. [33] track file activity patterns
of read/write/delete operations to identify ransomware behavior.
Kharraz et al. [19] propose Redemption, a system that monitors I/O
request patterns on a per-process basis and maintains a transparent
buffer for all storage I/O, allowing for immediate termination of
suspicious processes and data restoration with minimal system mod-
ifications and performance overhead. Ma et al. [22] propose Ran-
somTag, a hypervisor-based approach that employs fine-grained
behavior analysis by introspecting OS-level context information,
including file-related operations, process structures, and file ob-
jects, to accurately detect ransomware activities and enable precise
data recovery. Similar approaches monitor call sequences of file
access specifically for ransomware detection [16, 34]. Berrueta et
al. [5] combine file behavior monitoring with network communica-
tion analysis to detect ransomware activities, including command
and control communications. However, these methods often suf-
fer from high false positive rates, as some benign programs can
exhibit similar file operation patterns to ransomware [42]. Existing
approaches [15, 25, 40] are very sensitive to file operations, poten-
tially misclassifying normal file-intensive programs as ransomware
threats. In contrast to these approaches, CanCal addresses the
limitations of existing behavior-based methods by selectively fil-
tering suspicious processes and performing in-depth behavioral
analysis. This approach significantly reduces computational over-
head and false positives, making it suitable for large-scale industrial
applications.

Decoy-based Dynamic Analysis. Decoy-based techniques deploy
honeypot files or folders to detect ransomware activities. Moore [26]
monitors file screens of honeypot folders to identify ransomware
encryption attempts. Moussailebe et al. [27] deploy decoy folders in
specific paths under the C drive to trap ransomware. Lee et al. [21]
propose an efficient method for creating decoy files to detect ran-
somware, based on a source code-level analysis of ransomware
behavior patterns, suggesting the strategic placement of two decoy
files in the root directory to exploit common file traversal patterns
observed in ransomware samples. Gómez-Hernández et al. [15] in-
troduce R-Locker, using named pipe decoy files, though its uniform
nature might limit effectiveness. Ganfure et al. [14] develop RTrap,
a machine learning-based framework that strategically places de-
ceptive decoy files across directories to lure and detect ransomware,
employing a lightweight decoy watcher for real-time monitoring
and rapid automated response, achieving effective detection with
minimal file loss. RWGuard [25] combines decoy techniques with
process and file system monitoring to provide real-time ransomware
detection with high accuracy and low overhead. These approaches
leverage the fact that ransomware typically attempts to encrypt all
accessible files, including decoys. However, existing decoy-based
methods often generate an overwhelming number of false positives
in large-scale industrial clusters [10, 14, 31], leading to intolerable
disruptions and excessive inspection efforts. Moreover, they typi-
cally rely solely on decoy file interactions, potentially missing other
ransomware indicators. In contrast, CanCal addresses these limita-
tions with a multi-faceted monitoring approach. By combining the
decoy-based monitor with the ransom note monitor, CanCal pro-
vides a more robust and reliable solution for ransomware detection
in large-scale industrial applications.

6 CONCLUSION

In this paper, we propose CanCal, a lightweight and real-time
ransomware detection system specifically designed for large-scale
industrial environments. By innovatively prioritizing the analysis
of suspicious processes through selective monitoring layers, Can-
Cal effectively minimizes computational load and reduces alert
fatigue without compromising detection capabilities. Furthermore,
the rapid response and low CPU overhead provided by CanCal
ensure its practicality for long-term, seamless deployment in indus-
trial environments. The successful deployment across millions of
endpoints and its proven track record in thwarting both known and
zero-day ransomware attacks further attest to its robustness and
reliability. In conclusion, CanCal offers promising prospects for
safeguarding sensitive industrial environments against the growing
menace of ransomware.

ACKNOWLEDGMENT

This work was partly supported by the National Key R&D Pro-
gram of China (2021YFB2701000), the Key R&D Program of Hubei
Province (2023BAB017, 2023BAB079), the National NSF of China
(grants No.62302181, 62072046), the Knowledge Innovation Program
of Wuhan-Basic Research, Huawei Research Fund, and HUSTCSE-
FiberHome Joint Research Center for Network Security.

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

REFERENCES
[1] Hojjat Aghakhani, Fabio Gritti, Francesco Mecca, Martina Lindorfer, Stefano
Ortolani, Davide Balzarotti, Giovanni Vigna, and Christopher Kruegel. [n. d.].
When Malware is Packin’ Heat; Limits of Machine Learning Classifiers Based
on Static Analysis Features. Network and Distributed Systems Security (NDSS)
Symposium 2020 ([n. d.]). https://doi.org/10.14722/ndss.2020.24310

[2] Yahye Abukar Ahmed, Baris Koçer, Md. Shamsul Huda, Bander Ali Saleh Al-
rimy, and Mohammad Mehedi Hassan. 2020. A system call refinement-based
enhanced Minimum Redundancy Maximum Relevance method for ransomware
early detection. J. Netw. Comput. Appl. 167 (2020), 102753. https://doi.org/10.
1016/j.jnca.2020.102753

[3] Steve Alder. 2023. Ransomware Gangs Increasingly Exploiting 0Day and 1Day
Vulnerabilities. https://www.hipaajournal.com/ransomware-gangs-increasingly-
exploiting-0day-and-1day-vulnerabilities/.

[4] Kurt Baker. 2023. Fileless Malware Explained. https://www.crowdstrike.com/

cybersecurity-101/malware/fileless-malware/.

[5] Eduardo Berrueta, Daniel Morató, Eduardo Magaña, and Mikel Izal. 2022. Crypto-
ransomware detection using machine learning models in file-sharing network
scenarios with encrypted traffic. Expert Syst. Appl. 209 (2022), 118299. https:
//doi.org/10.1016/j.eswa.2022.118299

[6] Leo Breiman, J. H. Friedman, Richard A. Olshen, and C. J. Stone. 1984. Classifica-

tion and Regression Trees. Wadsworth.

[7] Mingcan Cen, Frank Jiang, Xingsheng Qin, Qinghong Jiang, and Robin Doss.
2024. Ransomware early detection: A survey. Computer Networks 239 (2024),
110138.

[8] Tianqi Chen and Carlos Guestrin. 2016. XGBoost: A Scalable Tree Boosting
System. In Proceedings of the 22nd ACM SIGKDD International Conference on
Knowledge Discovery and Data Mining (San Francisco, California, USA) (KDD
’16). Association for Computing Machinery, New York, NY, USA, 785–794. https:
//doi.org/10.1145/2939672.2939785

[9] Andrea Continella, Alessandro Guagnelli, Giovanni Zingaro, Giulio De Pasquale,
Alessandro Barenghi, Stefano Zanero, and Federico Maggi. 2016. ShieldFS: a
self-healing, ransomware-aware filesystem. In Proceedings of the 32nd Annual
Conference on Computer Security Applications, ACSAC 2016, Los Angeles, CA, USA,
December 5-9, 2016, Stephen Schwab, William K. Robertson, and Davide Balzarotti
(Eds.). ACM, 336–347. http://dl.acm.org/citation.cfm?id=2991110

[10] Shannon Davis. 2022. An Empirically Comparative Analysis of Ransomware
Binaries. https://www.splunk.com/content/dam/splunk2/en_us/gated/white-
paper/an-empirically-comparative-analysis-of-ransomware-binaries.pdf.
[11] Byron Denham and Dale R. Thompson. 2023. Analysis of Decoy Strategies for
Detecting Ransomware. In IEEE Conference on Communications and Network
Security, CNS 2023, Orlando, FL, USA, October 2-5, 2023. IEEE, 1–6. https://doi.
org/10.1109/CNS59707.2023.10288691

[12] Damien Warren Fernando and Nikos Komninos. 2022. FeSA: Feature selection
architecture for ransomware detection under concept drift. Comput. Secur. 116
(2022), 102659. https://doi.org/10.1016/j.cose.2022.102659

[13] Jerome H. Friedman. 2001. Greedy function approximation: A gradient boosting

machine. Annals of Statistics 29 (2001), 1189–1232.

[14] Gaddisa Olani Ganfure, Chun-Feng Wu, Yuan-Hao Chang, and Wei-Kuan Shih.
2023. RTrap: Trapping and Containing Ransomware With Machine Learning.
IEEE Trans. Inf. Forensics Secur. 18 (2023), 1433–1448. https://doi.org/10.1109/
TIFS.2023.3240025

[15] José Antonio Gómez-Hernández, L. Álvarez-González, and Pedro García-Teodoro.
2018. R-Locker: Thwarting ransomware action through a honeyfile-based ap-
proach. Comput. Secur. 73 (2018), 389–398. https://doi.org/10.1016/j.cose.2017.11.
019

[16] Nikolai Hampton, Zubair A. Baig, and Sherali Zeadally. 2018. Ransomware
behavioural analysis on windows platforms. J. Inf. Secur. Appl. 40 (2018), 44–51.
https://doi.org/10.1016/j.jisa.2018.02.008

[17] Yiwei Hou, Lihua Guo, Chijin Zhou, Yiwen Xu, Zijing Yin, Shanshan Li, Cheng-
nian Sun, and Yu Jiang. 2024. An Empirical Study of Data Disruption by
Ransomware Attacks. In Proceedings of the 46th IEEE/ACM International Con-
ference on Software Engineering (ICSE’24) (Lisbon, Portugal). ACM.
https:
//doi.org/10.1145/3597503.3639090

[18] Amin Kharraz, Sajjad Arshad, Collin Mulliner, William K. Robertson, and Engin
Kirda. 2016. UNVEIL: A Large-Scale, Automated Approach to Detecting Ran-
somware. In 25th USENIX Security Symposium, USENIX Security 16, Austin, TX,
USA, August 10-12, 2016, Thorsten Holz and Stefan Savage (Eds.). USENIX Associ-
ation, 757–772. https://www.usenix.org/conference/usenixsecurity16/technical-
sessions/presentation/kharaz

[19] Amin Kharraz and Engin Kirda. 2017. Redemption: Real-Time Protection Against
Ransomware at End-Hosts. In Research in Attacks, Intrusions, and Defenses - 20th
International Symposium, RAID 2017, Atlanta, GA, USA, September 18-20, 2017,
Proceedings (Lecture Notes in Computer Science, Vol. 10453), Marc Dacier, Michael D.
Bailey, Michalis Polychronakis, and Manos Antonakakis (Eds.). Springer, 98–119.
https://doi.org/10.1007/978-3-319-66332-6_5

[20] Amin Kharraz, William K. Robertson, Davide Balzarotti, Leyla Bilge, and Engin
Kirda. 2015. Cutting the Gordian Knot: A Look Under the Hood of Ransomware

Attacks. In Detection of Intrusions and Malware, and Vulnerability Assessment -
12th International Conference, DIMVA 2015, Milan, Italy, July 9-10, 2015, Proceed-
ings (Lecture Notes in Computer Science, Vol. 9148), Magnus Almgren, Vincenzo
Gulisano, and Federico Maggi (Eds.). Springer, 3–24. https://doi.org/10.1007/978-
3-319-20550-2_1

[21] Jeonghwan Lee, Jinwoo Lee, and Jiman Hong. 2017. How to Make Efficient Decoy
Files for Ransomware Detection?. In Proceedings of the International Conference
on Research in Adaptive and Convergent Systems, RACS 2017, Krakow, Poland,
September 20-23, 2017. ACM, 208–212. https://doi.org/10.1145/3129676.3129713
[22] Boyang Ma, Yilin Yang, Jinku Li, Fengwei Zhang, Wenbo Shen, Yajin Zhou, and
Jianfeng Ma. 2023. Travelling the Hypervisor and SSD: A Tag-Based Approach
Against Crypto Ransomware with Fine-Grained Data Recovery. In Proceedings
of the 2023 ACM SIGSAC Conference on Computer and Communications Security
(Copenhagen, Denmark) (CCS ’23). Association for Computing Machinery, New
York, NY, USA, 341–355. https://doi.org/10.1145/3576915.3616665

[23] Mohammad Masum, Md. Jobair Hossain Faruk, Hossain Shahriar, Kai Qian,
Dan Lo, and Muhaiminul Islam Adnan. 2022. Ransomware Classification and
Detection With Machine Learning Algorithms. In 12th IEEE Annual Computing
and Communication Workshop and Conference, CCWC 2022, Las Vegas, NV, USA,
January 26-29, 2022. IEEE, 316–322. https://doi.org/10.1109/CCWC54503.2022.
9720869

[24] May Medhat, Samir Gaber, and Nashwa Abdelbaki. 2018. A New Static-Based
Framework for Ransomware Detection. In 2018 IEEE 16th Intl Conf on De-
pendable, Autonomic and Secure Computing, 16th Intl Conf on Pervasive In-
telligence and Computing, 4th Intl Conf on Big Data Intelligence and Comput-
ing and Cyber Science and Technology Congress, DASC/PiCom/DataCom/Cyber-
SciTech 2018, Athens, Greece, August 12-15, 2018. IEEE Computer Society, 710–715.
https://doi.org/10.1109/DASC/PiCom/DataCom/CyberSciTec.2018.00124
[25] Shagufta Mehnaz, Anand Mudgerikar, and Elisa Bertino. 2018. RWGuard: A
Real-Time Detection System Against Cryptographic Ransomware. In Research in
Attacks, Intrusions, and Defenses - 21st International Symposium, RAID 2018, Her-
aklion, Crete, Greece, September 10-12, 2018, Proceedings (Lecture Notes in Computer
Science, Vol. 11050), Michael Bailey, Thorsten Holz, Manolis Stamatogiannakis,
and Sotiris Ioannidis (Eds.). Springer, 114–136. https://doi.org/10.1007/978-3-
030-00470-5_6

[26] Chris Moore. 2016. Detecting Ransomware with Honeypot Techniques. In Cy-
bersecurity and Cyberforensics Conference, CCC 2016, Amman, Jordan, August 2-4,
2016. IEEE, 77–81. https://doi.org/10.1109/CCC.2016.14

[27] Routa Moussaileb, Benjamin Bouget, Aurélien Palisse, Hélène Le Bouder, Nora
Cuppens, and Jean-Louis Lanet. 2018. Ransomware’s Early Mitigation Mecha-
nisms. In Proceedings of the 13th International Conference on Availability, Reliability
and Security, ARES 2018, Hamburg, Germany, August 27-30, 2018, Sebastian Doerr,
Mathias Fischer, Sebastian Schrittwieser, and Dominik Herrmann (Eds.). ACM,
2:1–2:10. https://doi.org/10.1145/3230833.3234691

[28] Mosimilolu Odusanya. 2020. MITRE ATT&CK spotlight: Process injec-
https://www.infosecinstitute.com/resources/mitre-attck/mitre-attck-

tion.
spotlight-process-injection/.

[29] Harun Oz, Ahmet Aris, Albert Levi, and A. Selcuk Uluagac. 2022. A Survey on
Ransomware: Evolution, Taxonomy, and Defense Solutions. ACM Comput. Surv.
54, 11s (2022), 238:1–238:37. https://doi.org/10.1145/3514229

[30] Salwa Razaulla, Claude Fachkha, Christine Markarian, Amjad Gawanmeh, Wathiq
Mansoor, Benjamin C. M. Fung, and Chadi Assi. 2023. The Age of Ransomware:
A Survey on the Evolution, Taxonomy, and Research Directions. IEEE Access 11
(2023), 40698–40723. https://doi.org/10.1109/ACCESS.2023.3268535

[31] RiskOptics. 2021. Avoiding Cyber Security False Positives. https://reciprocity.

com/blog/avoiding-cyber-security-false-positives/.

[32] Nir Rosen, Haim Elisha, Ahmad Saleh, Vadim Gechman, and Sharon Mashhadi.
2023. Supercharge Ransomware Detection with AI-Enhanced Cybersecurity So-
lutions. https://developer.nvidia.com/blog/supercharge-ransomware-detection-
with-ai-enhanced-cybersecurity-solutions/.

[33] Nolen Scaife, Henry Carter, Patrick Traynor, and Kevin R. B. Butler. 2016.
CryptoLock (and Drop It): Stopping Ransomware Attacks on User Data. In
36th IEEE International Conference on Distributed Computing Systems, ICDCS
2016, Nara, Japan, June 27-30, 2016. IEEE Computer Society, 303–312. https:
//doi.org/10.1109/ICDCS.2016.46

[34] Daniele Sgandurra, Luis Muñoz-González, Rabih Mohsen, and Emil C. Lupu. 2016.
Automated Dynamic Analysis of Ransomware: Benefits, Limitations and use for
Detection. CoRR abs/1609.03020 (2016). arXiv:1609.03020 http://arxiv.org/abs/
1609.03020

[35] Shina Sheen and Ashwitha Yadav. 2018. Ransomware detection by mining API
call usage. In 2018 International Conference on Advances in Computing, Commu-
nications and Informatics, ICACCI 2018, Bangalore, India, September 19-22, 2018.
IEEE, 983–987. https://doi.org/10.1109/ICACCI.2018.8554938

[36] Sophos. 2023. The State of Ransomware 2023. https://assets.sophos.com/

X24WTUEQ/at/c949g7693gsnjh9rb9gr8/sophos-state-of-ransomware-2023-
wp.pdf.

[37] Dan Steinberg, Mikhail Golovnya, and Nicholas Scott Cardell. 2002. Stochastic
Gradient Boosting: An Introduction to TreeNet™. In The 15th Australian Joint

CanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

Conference on Artificial Intelligence 2002, Proceedings Australasian Data Mining
Workshop, Canberra, Australia, 3rd December 2002, Simeon J. Simoff, Graham J.
Williams, and Markus Hegland (Eds.). University of Technology Sydney, Australia,
1–12.

[38] Chainalysis Team. 2024. Ransomware Payments Exceed 1 Billion in 2023, Hitting
Record High After 2022 Decline. https://www.chainalysis.com/blog/ransomware-
2024/.

[39] Umara Urooj, Bander Ali Saleh Al-rimy, Anazida Zainal, Fuad A Ghaleb, and
Murad A Rassam. 2022. Ransomware detection using the dynamic analysis and
machine learning: A survey and research directions. Applied Sciences 12, 1 (2022),
172.

[40] Jonathan Voris, Yingbo Song, Malek Ben Salem, Shlomo Hershkop, and Salvatore J.
Stolfo. 2019. Active authentication using file system decoys and user behavior

modeling: results of a large scale study. Comput. Secur. 87 (2019). https://doi.org/
10.1016/j.cose.2018.07.021

[41] Bahaa Yamany, Mahmoud Said Elsayed, Anca D Jurcut, Nashwa Abdelbaki, and
Marianne A Azer. 2022. A New Scheme for Ransomware Classification and
Clustering Using Static Features. Electronics 11, 20 (2022), 3307.

[42] Chijin Zhou, Lihua Guo, Yiwei Hou, Zhenya Ma, Quan Zhang, Mingzhe Wang,
Zhe Liu, and Yu Jiang. 2023. Limits of I/O Based Ransomware Detection: An
Imitation Based Attack. In 2023 IEEE Symposium on Security and Privacy (SP).
IEEE Computer Society, 2584–2601.

[43] Jinting Zhu, Julian Jang-Jaccard, Amardeep Singh, Ian Welch, Harith Al-Sahaf,
and Seyit Camtepe. 2022. A few-shot meta-learning based siamese neural network
using entropy features for ransomware classification. Comput. Secur. 117 (2022),
102691. https://doi.org/10.1016/j.cose.2022.102691

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

A SENSITIVITY ANALYSIS

We conduct sensitivity analysis on two key parameters: the sliding
window size for ransom note gene fragments and the similarity
matching threshold for ransomware detection.

A.1 Sliding Window Size Sensitivity Analysis

To determine the optimal sliding window size for ransom note gene
fragments, we analyzed the system’s performance using 158 ransom
note samples and 100 benign samples. For each sliding window size,
we selected the 300 most frequently occurring combinations as
gene fragments. Table 6 presents the results of this analysis.

Table 6: Sliding Window Size Sensitivity Analysis Results

Ransomware Benign

Size Threshold Recall

rate as the threshold varies. Figure 7 illustrates how these perfor-
mance metrics change with different similarity matching thresholds.
The analysis reveals that the system’s performance is particularly
sensitive to the similarity-matching threshold. At 𝜏𝑠𝑖𝑚 = 0.21, we
achieve an optimal balance: a false alarm rate of 0% while detecting
over 87% of ransomware samples with 100% precision.

B BEHAVIORAL FEATURE DISTRIBUTION

To further explore the behavioral characteristics of the extortion
samples, we plot the 12-dimensional feature distribution of benign
and ransomware samples in Figure 8 and Figure 9. The horizontal
axis represents the feature values, and the vertical axis represents
the amount of benign/ransomware numbers at the corresponding
feature. The 12-dimensional features correspond to the definition
description of § 2.3.2.

158

100

1

2

3

4

5

6

131

55

4

1

0

0

6.32%

34.18%

84.17%

81.64%

68.35%

56.32%

For each window size, we determined the threshold as the num-
ber of gene fragments that resulted in zero false positives in the
benign samples. The analysis reveals that the system’s performance
is highly sensitive to the sliding window size, with the recall rate
peaking at 84.17% for a window size of 3, while maintaining zero
false positives.

A.2 Similarity Matching Threshold Sensitivity

Analysis

Figure 7: Sensitivity analysis of precision, recall, and false
alarm rate for the ransom note classifier. The optimal thresh-
old 𝜏𝑠𝑖𝑚 = 0.21 achieves a 0% FPR with 100% precision.

We also conducted a sensitivity analysis on the similarity match-
ing threshold 𝜏𝑠𝑖𝑚 to optimize ransomware detection. This analysis
focused on the trade-off between precision, recall, and false alarm

Figure 8: Distribution differences for Feature#1~Feature#6.

The analysis reveals that the ransomware samples show ex-
tremely strong distribution differences from the benign samples in
Feature#3, Feature#8, and Feature#11. Therefore, the feature engi-
neering in this paper can distinguish the benign samples from the
ransomware samples well.

C MODEL SELECTION

To determine the most effective machine learning model for our
ransomware detection system, we conducted a comprehensive eval-
uation of various algorithms. We used a total of 87,843 behavior
data samples, split into 70,274 (80%) for training and 17,569 (20%) for
validation. Our analysis employed two popular machine learning li-
braries: scikit-learn and XGBoost. All models were tested using their
default parameters to ensure a fair comparison. Table 7 presents
the performance metrics for each model tested.

Our analysis shows that ensemble methods, particularly tree-
based models, demonstrated superior performance across most
metrics. Among these, the XGBoost algorithm, an efficient imple-
mentation of Gradient Boosting Decision Trees (GBDT), achieved
the highest accuracy (99.38%), precision (98.66%), and F1 Score
(97.74%). Given its robust and balanced performance in ransomware
detection, we selected XGBoost as our primary model for the ran-
somware detection system.

0.00.20.40.60.81.0Similarity Threshold0.00.20.40.60.81.0FPR=0PR-FPR CurveFalse Positive RatePrecisionRecall02505007501000ransomware f-1 distribution020004000Count0250500750ransomware f-2 distribution0100020003000Count0200400600800ransomware f-3 distribution05001000Count050100150200benign f-1 distribution01000020000Count02505007501000benign f-2 distribution01000020000Count0255075100benign f-3 distribution01000020000Count0200400ransomware f-4 distribution020004000Count0102030ransomware f-5 distribution0100020003000Count0.02.55.07.510.0ransomware f-6 distribution020004000Count0510benign f-4 distribution0100002000030000Count050100150benign f-5 distribution01000020000Count0510benign f-6 distribution01000020000Count U D Q V R P Z D U H  D Q G  E H Q L J Q  I H D W X U H  G L V W U L E X W L R Q  F R P S D U L V R QCanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

Mallox, Lockbit 3.0, and Tellyouthepass. For further analysis, we
conducted a manual forensic analysis of 27 ransomware attacks
detected from March to June 2023. These cases in Table 10 showcase
CanCal’s remarkable ability to identify and defend against various
ransomware families and variants, regardless of whether the at-
tacks were conducted through disguise, vulnerability exploitation,
process injection, or other advanced techniques. Notably, Can-
Cal successfully prevented 13 attacks leveraging n-day vulnerabili-
ties documented in the CVE database, such as XVE-2023-3377 and
QVD-2023-14179, as well as 5 high-risk attacks exploiting zero-day
vulnerabilities. This further demonstrates CanCal’s outstanding
capability in combating unknown and complex ransomware threats.

Table 8: Ransomware attacks detected in each month from
March 2023 to April 2024. These ransomware are categorized
by their families.

Month

March 2023

May 2023

Family

Phobos

Mallox

Mallox

Tellyouthepass

Mallox

Count

1

4

5

5

1

June 2023

Tellyouthepass

10

Lockbit3.0

Mallox

July 2023

Tellyouthepass

August 2023

September 2023

October 2023

November 2023

December 2023

January 2024

March 2024

April 2024

Lockbit 3.0

Mallox

Tellyouthepass

Mallox

Mallox

Mallox

Tellyouthepass

Phobos

Mallox

Encrypted

Tellyouthepass

Mallox

1

2

9

1

4

3

3

2

1

3

2

1

1

1

1

Figure 9: Distribution differences for Feature#7~Feature#12.

Table 7: Comparison of Machine Learning Models for Ran-
somware Detection

Model
GradientBoosting
LogisticRegression
SVM
DecisionTree
XGBoost
RandomForest
ExtraTrees
NaiveBayes
AdaBoosting

Rec

Pre
Acc
96.49% 97.80%
99.23%
97.30%
90.26%
98.33%
94.99%
88.80%
97.84%
99.30%
97.06%
97.83%
99.38% 98.66% 96.84%
97.15%
98.20%
99.36%
97.19%
98.16%
99.36%
66.98%
78.65%
91.81%
96.93%
89.76%
98.22%

F1 Score
97.14%
93.65%
91.79%
97.44%
97.74%
97.67%
97.67%
72.34%
93.21%

D DETAILED MONTHLY TEST RESULTS

This appendix summarizes the continuous monthly test results
from February to June 2023. The results shown in Table 9 are ob-
tained from different antivirus software, namely CanCal, Solution
A, Solution B, Solution C, and Solution D. The detection results are
categorized into two groups: non-process injection categories and
process injection categories. Overall, the statistical results show
that CanCal has the highest detection rate for most ransomware
families, while Solution A has the highest detection rate for some
specific families. Solution B and Solution C have the lowest detec-
tion rates for most families.

E REALWORLD RANSOMWARE DETECTED

As statistics listed in Table 8, CanCal has successfully detected
and thwarted 61 real-world ransomware attacks from March 2023
to April 2024. The detected ransomware families include Phobos,

0200400600800ransomware f-7 distribution05001000Count010203040ransomware f-8 distribution05001000Count050100150200ransomware f-9 distribution020004000Count010203040benign f-7 distribution01000020000Count0246benign f-8 distribution01000020000Count0246benign f-9 distribution01000020000Count0200400600800ransomware f-10 distribution0100020003000Count0200400600800ransomware f-11 distribution0100020003000Count0.000.250.500.751.00ransomware f-12 distribution0100020003000Count0102030benign f-10 distribution0100002000030000Count0510benign f-11 distribution0100002000030000Count0.000.250.500.751.00benign f-12 distribution0100002000030000Count U D Q V R P Z D U H  D Q G  E H Q L J Q  I H D W X U H  G L V W U L E X W L R Q  F R P S D U L V R QCCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

S Wang, F Dong, H Yang, J Xu, and H Wang

Table 9: Detailed comparison results with four SOTA industrial solutions from February to June 2023.

Month

Injection Type

Family

Quantity

February 2023 Non-Process Injection

March 2023

Non-Process Injection

Non-Process Injection

April 2023

Process Injection

Non-Process Injection

May 2023

Process Injection

Non-Process Injection

June 2023

Process Injection

BianLian
LockbitGreen
Putin
Conti
Lockbit
Mimic
Netwalker
Ryuk
Phobos
Chaos
Stop/Djvu
Blackcat
Cylance
DarkPower
Hermes
KadavroVector
MoneyMessage
PayMe100USD
Play
Rcru64
RedEye
Ryuk
Waiting
Magniber
Netwalker
Egregor
Akira
Blackbit
Conti
Gazprom
Lockbit
Locky
Paradise
Play
Rancoz
Waiting
Lockbit
Netwalker
CrossLock
Magniber
Ryuk
Rhysida
Darkrace
WannaCry3.0
BigHead
BlackBasta
Mallox
NoEscape
Phobos
Egregor
Magniber
Netwalker

4
4
4
5
8
8
3
5
8
14
19
1
1
2
2
3
4
2
1
8
1
11
2
6
10
2
5
5
7
3
6
3
2
3
2
2
2
2
2
2
2
2
2
6
10
30
1
5
2
2
2
2

CanCal
4/4 (100.00%)
4/4 (100.00%)
4/4 (100.00%)
4/5 (80.00%)
8/8 (100.00%)
8/8 (100.00%)
2/3 (66.67%)
5/5 (100.00%)
8/8 (100.00%)
14/14 (100.00%)
19/19 (100.00%)
1/1 (100.00%)
1/1 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
3/3 (100.00%)
4/4 (100.00%)
2/2 (100.00%)
1/1 (100.00%)
8/8 (100.00%)
1/1 (100.00%)
11/11 (100.00%)
2/2 (100.00%)
6/6 (100.00%)
10/10 (100.00%)
2/2 (100.00%)
5/5 (100.00%)
5/5 (100.00%)
7/7 (100.00%)
3/3 (100.00%)
6/6 (100.00%)
3/3 (100.00%)
2/2 (100.00%)
3/3 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
6/6 (100.00%)
10/10 (100.00%)
30/30 (100.00%)
1/1 (100.00%)
5/5 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)
2/2 (100.00%)

Solution A Solution B Solution C Solution D

4/4
3/4
4/4
4/5
7/8
8/8
3/3
4/5
8/8
14/14
18/19
1/1
1/1
2/2
2/2
3/3
4/4
2/2
1/1
8/8
1/1
11/11
2/2
6/6
10/10
2/2
5/5
5/5
7/7
3/3
6/6
3/3
2/2
3/3
2/2
2/2
2/2
2/2
2/2
1/2
1/2
2/2
1/2
6/6
10/10
29/30
1/1
5/5
2/2
2/2
0/2
2/2

2/4
0/2
4/4
0/5
7/8
7/8
1/3
0/5
8/8
1/14
19/19
1/1
0/1
0/2
2/2
1/3
4/4
0/2
1/1
8/8
1/1
11/11
2/2
6/6
0/10
2/2
5/5
5/5
7/7
3/3
6/6
3/3
2/2
3/3
2/2
2/2
0/2
0/2
1/2
2/2
1/2
0/2
0/2
6/6
10/10
30/30
0/1
5/5
0/2
2/2
2/2
0/2

3/4
4/4
4/4
5/5
7/8
8/8
0/3
5/5
8/8
13/14
19/19
1/1
1/1
1/2
0/2
3/3
3/4
2/2
0/1
8/8
1/1
0/11
0/2
0/6
0/10
0/2
5/5
5/5
7/7
0/2
6/6
3/3
2/2
3/3
2/2
0/2
2/2
0/2
2/2
0/2
0/2
2/2
2/2
6/6
10/10
22/30
0/1
5/5
0/2
0/2
0/2
0/2

3/4
4/4
4/4
4/5
8/8
8/8
0/3
3/5
8/8
13/14
19/19
1/1
1/1
2/2
2/2
2/3
4/4
0/2
1/1
8/8
1/1
0/11
0/2
0/6
0/10
2/2
5/5
5/5
7/7
3/3
5/6
3/3
2/2
3/3
2/2
0/2
1/2
0/2
2/2
0/2
1/2
2/2
2/2
0/6
3/10
29/30
1/1
4/5
0/2
2/2
0/2
0/2

CanCal: Towards Real-time and Lightweight Ransomware Detection and Response in Industrial Environments

CCS ’24, October 14–18, 2024, Salt Lake City, UT, USA.

Table 10: The detailed manual forensic analysis of real-world ransomware attacks detected from March to June 2023.

Date

Industry

Attack Method4

Family

Security Advisories

2023.03.27

Chemical Industry

2023.04.11

2023.05.01

Business Services

Retail Industry

2023.05.02

Equipment Manufacturing

2023.05.03

Construction Engineering

A1

A1

A1

A1

A1

Phobos

Mallox

Mallox

Mallox

Mallox

-

-

-

-

-

2023.05.08

Electronics Manufacturing

A2, A3

Tellyouthepass

XVE-2023-3377

2023.05.10

Apparel Manufacturing

2023.05.15

General Services

2023.05.15

Electronics Manufacturing

2023.05.15

Pet Hospital

2023.05.15

Biomedical Technology

2023.05.15

Smart Manufacturing

2023.05.22

Wholesale & Retail

2023.05.23

Biomedical Technology

2023.06.09

Data Bureau

2023.06.10

Investment Management

2023.06.18

Coal Mining

2023.06.23

Scientific Research

2023.06.23

General Services

2023.06.23

Smart Transportation

2023.06.23

Equipment Manufacturing

2023.06.23

Home Furnishings

2023.06.25

Chemical Industry

2023.06.25

Construction Engineering

2023.06.29

Equipment Manufacturing

2023.06.29

Internet

2023.06.29

Scientific Research

A1

A2, A3

A2, A3

A2, A3

A2, A3

A2, A3

A1

A4

A2, A3

A2, A3

A3

A2, A3

A2, A3

A2, A3

A2, A3

A2, A3

A2, A3

A2, A3

A2, A3

A2, A3

A2, A3

Mallox

-

Tellyouthepass

0-day vulnerability

Tellyouthepass

0-day vulnerability

Tellyouthepass

0-day vulnerability

Tellyouthepass

0-day vulnerability

Tellyouthepass

0-day vulnerability

Mallox

Lockbit3.0

-

-

Tellyouthepass

XVE-2023-3377

Tellyouthepass

XVE-2023-3377

Mallox

-

Tellyouthepass

QVD-2023-14179

Tellyouthepass

QVD-2023-14179

Tellyouthepass

QVD-2023-14179

Tellyouthepass

QVD-2023-14179

Tellyouthepass

QVD-2023-14179

Tellyouthepass

XVE-2023-3377

Tellyouthepass

XVE-2023-3377

Tellyouthepass

QVD-2023-14179

Tellyouthepass

QVD-2023-14179

Tellyouthepass

QVD-2023-14179

7 We analyzed four main attack methods. A1: Disguise as normal file; A2: Vulnerability exploitation; A3:

Process injection; A4: Add malicious files to the trust zone.


