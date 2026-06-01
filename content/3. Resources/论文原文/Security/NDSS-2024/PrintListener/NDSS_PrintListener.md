---
publish: true
---

PrintListener: Uncovering the Vulnerability of Fingerprint Authentication via the
Finger Friction Sound

4
2
0
2

r
p
A
4
1

]

R
C
.
s
c
[

1
v
4
1
2
9
0
.
4
0
4
2
:
v
i
X
r
a

Man Zhou1†∗, Shuao Su1†, Qian Wang2, Qi Li3, Yuting Zhou1†, Xiaojing Ma1† and Zhengxiong Li4
1School of Cyber Science and Engineering, Huazhong University of Science and Technology, China
2School of Cyber Science and Engineering, Wuhan University, China
3Institute for Network Sciences and Cyberspace, Tsinghua University, China
4Department of Computer Science and Engineering, University of Colorado Denver, USA
Email: {zhouman, m202271728, m202271743, lindahust}@hust.edu.cn,
qianwang@whu.edu.cn, qli01@tsinghua.edu.cn, zhengxiong.li@ucdenver.edu

Abstract—Fingerprint authentication has been extensively
employed in contemporary identity verification systems owing
to its rapidity and cost-effectiveness. Due to its widespread
use, fingerprint leakage may cause sensitive information theft,
enormous economic and personnel losses, and even a potential
compromise of national security. As a fingerprint
that can
coincidentally match a specific proportion of the overall fin-
gerprint population, MasterPrint rings the alarm bells for the
security of fingerprint authentication. In this paper, we propose
a new side-channel attack on the minutiae-based Automatic
Fingerprint Identification System (AFIS), called PrintListener,
which leverages users’ fingertip swiping actions on the screen to
extract fingerprint pattern features (the first-level features) and
synthesizes a stronger targeted PatternMasterPrint with potential
second-level features. The attack scenario of PrintListener is
extensive and covert. It only needs to record users’ fingertip
friction sound and can be launched by leveraging a large number
of social media platforms. Extensive experimental results in real-
world scenarios show that Printlistener can significantly improve
the attack potency of MasterPrint.

I.

INTRODUCTION

The fingerprint is an impression left by the friction ridges of
a human finger. As an essential and well-recognized personal
identification (ID), the fingerprint has seamlessly integrated
into people’s daily life, e.g., phone screen unlock [1], fin-
gerprint online payments [2], national ID cards/electronic
passports [3], etc. Consequently,
it has gained widespread
adoption in applications such as user authentication, online
transaction platforms, access control, and government and law
enforcement agencies. It
the market size
of fingerprint authentication will touch USD 99.9 billion by
2032 [4].

is projected that

Preventing user fingerprint information leakage is a signifi-
cant challenge worldwide [5]. The breach of such information
can lead to the theft of sensitive data, substantial financial and

*Corresponding author: Man Zhou (zhouman@hust.edu.cn).
†Hubei Key Laboratory of Distributed System Security, Hubei Engineering
Research Center on Big Data Security, School of Cyber Science and Engi-
neering, Huazhong University of Science and Technology.

Network and Distributed System Security (NDSS) Symposium 2024
26 February - 1 March 2024, San Diego, CA, USA
ISBN 1-891562-93-2
https://dx.doi.org/10.14722/ndss.2024.24618
www.ndss-symposium.org

human losses, and even pose a threat to national security. The
current fingerprint attack explorations are mainly based on con-
tact scrutinization. It is possible to extract usable fingerprints
from the surface touched [6] or photos of exposed fingers
[7]. Therefore, one intuitive suggestion to enhance fingerprint
security is that people can keep their fingerprints away from
others’ sight, e.g., no adversary proximity/accessibility and no
access to their devices. However, is this ideal scenario truly
secure against attacks? As a fingerprint that can coincidentally
match a specific proportion of the overall fingerprint popu-
[9] sequences
lation, MasterPrint
conduct dictionary attacks on fingerprints without knowing
users’ fingerprint information. Luckily, their attack success
rates are very low in a high security-level setting. Specifically,
the attack success rates of MasterPrint and DeepMasterPrint
are only 1.88% and 1.11% with a 0.01% False Acceptance
Rate (F AR). But if the attacker has inferred some victim’s
fingerprint information contactlessly, is it possible to generate
stronger MasterPrint sequences for dictionary attacks?

[8] and DeepMasterPrint

Recently, some studies [10], [11], [12] have shown that the
uniqueness of finger-swiping friction sound is influenced by an
individual’s fingerprint biometric characteristics. Besides, such
finger-swiping friction sounds can be captured by attackers
online with a high possibility, which will be a catalyst to
enable such attacks. For instance, users engage in online
gaming through social applications such as Discord, where
players interact and cooperate through video and in-game
voice chat. In such cases, users frequently slide their fingers
across the device screens. Similarly, during audio and video
calls on mobile devices’ social platforms (e.g., Skype [13],
WeChat [14], and Apple FaceTime [15]), users unconsciously
perform swiping operations on the screen, such as searching
for other apps or scrolling through information. These finger-
sliding friction sounds will be transmitted to the other party
by social communication software as well as malware with
recording permission. Therefore, we will explore the possibil-
ity of an attacker remotely inferring the victim’s fingerprint
information based on the fingerprint friction sound.

In this work, we propose a new side-channel attack on
fingerprints, called PrintListener, which leverages users’ swip-
ing actions on the screen to extract fingerprint features and
synthesize a stronger MasterPrint sequence based on these
features to conduct dictionary attacks on users’ fingerprints,
as shown in Figure 1. To achieve this, PrintListener separates
weak frictional sounds that are buried in dynamic speech and
background noise interference and obtains the first-level fea-

monalities within the fingerprint dataset with the same patterns.
Furthermore, we employ pitch shift and time stretch techniques
to balance the diversity of velocity and pressure in the original
audio training set, including various sliding trajectories. 3)
After inferring the primary pattern features of fingerprints, the
potential search space for the secondary features corresponding
to fingerprints of the same pattern is vast. How to effectively
search and synthesize a PatternMasterPrint dictionary capable
of attacking the majority of fingerprints from the latent space?
To address this, we conduct a statistical analysis of the
intercorrelations between the primary and secondary features
and design a heuristic search algorithm specifically targeting
the detailed secondary features of fingerprints. Building upon
the traditional random restart hill-climbing mechanism, we
first identify the crucial region (CR) within the fingerprint
area, which is prone to high-frequency collisions of secondary
features. Within the CR, we adaptively narrow down the
potential search space based on the current search state while
simultaneously increasing the likelihood of collisions among
the secondary features. This approach allows us to synthesize
locally optimal patterns in the master fingerprint dictionary.

Our contributions can be summarized as follows:

• We uncover a new side-channel attack on fingerprint
and propose PrintListener, which leverages users’
swiping actions on the screen to identify the finger-
print pattern and synthesizes PatternMasterPrint se-
quences to conduct more powerful dictionary attacks.
To the best of our knowledge, this is the first work that
leverages swiping friction sounds to infer fingerprint
information.

• We design a series of algorithms for pre-processing
the raw audio signals, eliminating the interference of
redundant audio features, and providing a wide and
deep combination prediction for fingerprint patterns.
Specifically, PrintListener can automatically capture
the pattern features of fingerprints from a large number
of raw recordings and generate targeted synthetic
PatternMasterPrints.
Extensive experimental results in real-world scenarios
show that Printlistener has strong attack power on
fingerprint authentication. It can attack up to 27.9% of
partial fingerprints and 9.3% of complete fingerprints
the highest security F AR
within five attempts at
setting of 0.01%.

•

II. PRELIMINARIES

In this section, we first analyze whether PatternMasterprint
has a high probability of matching fingerprints with the same
pattern, then study the feasibility of fingerprint pattern predic-
tion via finger friction sound.

A. Is PatternMasterprint’s attack more powerful?

The friction ridge details of a fingerprint are generally
classified into 3 levels: Level 1 (pattern), Level 2 (minutiae
points), and Level 3 (pores and ridge shape). Currently, most
AFIS only use Level 1 and Level 2 features. The Level 1
features (fingerprint pattern) can usually be divided into four
distinct types: left loop, right loop, whorl, and arch [17]. Each
has unique variations depending on the shape and relationship

Fig. 1: Attack scenario of PrintListener.

ture (fingerprint pattern) of fingerprints through the wide and
deep combined prediction model. Further, PrintListener uses
the random restart hill-climbing algorithm to synthesize the
second-level feature (the position and direction of minutiae) of
fingerprints that correspond to the inferred first-level feature,
namely minutiae templates, which are the basis for finger-
print authentication. In addition, the synthesized fingerprint
minutiae templates can also be used to reconstruct fingerprint
images [16].

By using the swiping friction sound as a natural attack
entry point, our attack has two advantages: 1) Stealthiness:
PrintListener can be carried out by leveraging mainstream
social software with voice and video capabilities and does
not necessitate any supplementary hardware. It capitalizes
on the built-in microphones in electronic devices, such as
smartphones, to capture the faint friction sounds generated
by finger movements across electronic screens. Subsequently,
the user’s fingerprint patterns are inferred from these sounds.
2) Pervasiveness: PrintListener is based on the MasterPrint
sequences attack, which does not require large data training
on a specific person. These sequences can subsequently be
employed to launch more powerful dictionary attacks on all
victims’ fingerprints that conform to the specific pattern.

To accomplish this goal, we need to resolve three major
challenges: 1) The sound intensity of finger friction from users
is extremely weak, typically ranging from 0.2 to 0.8 seconds.
The original audio and video call information often contain
a significant amount of redundant information. To extract the
faint friction sound submerged in dynamic speech and back-
ground noise interference, we design a friction sound event
localization algorithm based on spectral analysis. By moving
time windows and examining the energy spectral density of
the starting
audio in different frequency bands, we detect
and ending points of friction sound events. By analyzing the
spectral peak-valley characteristics of the activity events, we
can eliminate the interference from multi-band and multi-type
activity noise, enabling precise localization of finger friction
events. 2) Friction sound characteristics are often influenced
by users’ physiological and behavioral features. In addition
to the primary feature patterns represented by the finger’s
surface morphology, there are also factors such as the pressure
and velocity, as well as the swiping trajectory. To address
this, we propose a joint prediction approach using a width
and depth classification module to capture both interpretable
audio features and deep representation features. We employ the
minimum redundancy maximum relevance (mRMR) feature
selection strategy and utilize conditional mutual information to
reduce feature redundancy. By deploying an adaptive weight-
ing strategy, we aim to balance the prediction results and
enhance the robustness of the fingerprint feature prediction
model to behavioral characteristics, thus capturing the com-

2

Fake fingerprints User finger swiping on the screenSocial media appsOnline meetingSpywareAccess controlPayment appsPrivacy informationFingertip friction soundPredict fingerprint featuresFig. 2: Variations of F AR with threshold settings in D0
(mixed patterns), D1 (whorl), D2 (left loop), and D3 (right
loop) Datasets.

of the ridges. To determine whether PatternMasterPrint (the
MasterPrint generated by fingerprints with a specific pattern)
has a stronger attack power for fingerprints with the same
pattern, we initially analyze the similarities between finger-
prints that share the same pattern and evaluate the F AR
for fingerprints in datasets containing the same pattern, as
well as mixed patterns. Subsequently, we investigate statistical
evidence to support our intuitive hypothesis that a higher F AR
of fingerprints in datasets with the same pattern will increase
the likelihood of detecting a PatternMasterPrint (referred to as
PMP).

Hypothesis. The probability of finding PMP in the same
pattern fingerprint dataset Ds is P (P M P ⊂ Ds), and the prob-
ability of finding MasterPrint (MP) in the mixed fingerprint
dataset Dm is P (M P ⊂ Dm). Then our alternative hypothesis
is

H1 : P (P M P ⊂ Ds) >= P (M P ⊂ Dm).

(1)

Hypothesis Analysis Setup. We select the livedet2011 It-
aldataLive dataset [18], which comprises a total of 400 fingers,
each with 5 complete fingerprint images, resulting in a total
of 2000 images. It consists of 680 whorl fingerprints (dataset
D1), 565 left loop fingerprints (dataset D2), 580 right loop
fingerprints (dataset D3), and 175 arch fingerprints (dataset
D4). Since the population of arch fingerprints is less than 5%
[19], we exclude dataset D4 from our statistical analysis‡.

Thus,

the mixed dataset D0 amalgamates three finger
patterns (D0 = D1 ∪ D2 ∪ D3). To test our hypothesis H1,
the single-pattern dataset Ds can be D1, D2, or D3, and the
mixed-pattern dataset Dm is D0.

Hypothesis Statistics. Figure 2 depicts the variation of
F AR concerning threshold values from datasets D0 to D3.
With the same threshold, the F AR of the single-pattern fin-
gerprint dataset is considerably higher than that of the dataset
containing a mixture of patterns. This disparity arises from
the fact that fingerprints belonging to the same pattern tend
to exhibit similar locations of singular points and coarse flow
directions of ridges.

We define the potential MPs/PMPs as those that incorrectly
match with at least 4% of the fingerprint population. In the

Fig. 3: Propagation paths of frictional sound waves.

mixed fingerprint dataset D0 of 1825 fingerprints, there are
146 MPs, which accounts for an 8.00% (146/1825) proportion
of MPs. We also conducted separate statistical analyses on the
D1-D3 datasets to determine the presence of potential PMPs.
The proportions of PMPs in the D1-D3 datasets are as follows:
17.65% (102/578), 25.28% (114/451), and 13.73% (70/510).
Notably, all these proportions are higher than that of the D0
dataset, indicating a higher prevalence of PMPs in single-type
fingerprint datasets. These findings provide support for the
alternative hypothesis H1.

B. Acoustic principle of fingerprint pattern prediction

1) How are the finger frictional sounds created? : When
two solid surfaces come into contact and slide against each
other, they generate vibrations and waves that result in fric-
tional noise. Frictional noise can be divided into two categories
depending on the contact pressure (high or low) between the
frictional pairs [20], [21]. Frictional noise under high contact
pressure is typically transient and caused by mechanical insta-
bility between surfaces, such as door squeaks. Low-pressure
frictional noise is relatively stable and is commonly referred
to as roughness noise, such as the rustling sound produced by
rubbing two sheets of sandpaper.

When the finger swipes on a screen, the weak coupling
between the finger pad and the screen will generate a rough-
ness noise. The production of roughness noise involves three
essential factors: friction (the elastic deformation between the
fingertips and the smartphone screen amplifies the vibrations),
dynamics (the vibrations and waves propagate between the
finger and the screen), and acoustics (audible roughness sound
radiates from the finger to the surface of the phone and
propagates through the air and solid medium to the phone’s
microphone [22]), as shown in Figure 3.

2) Influential

factors of

friction sound characteristics:

When the finger slides across a screen, the surface of the finger
and the screen are subjected to a light load, resulting in only
slight nonlinear deformation of the skin on the fingertip. The
sound pressure level Lp (dB) of the roughness noise can be
expressed as:

nV m,

(2)

‡Arch fingerprints will not be considered in this paper.

Lp = 20 log10 Ra

3

           7 K U H V K R O G      ) D O V H Accept  5 D W H     '    0 L [ H G  S D W W H U Q V  '    : K R U O '    / H I W  O R R S  '    5 L J K W  O R R SMicOther reflection pathStructure-bronepathAir-bronepathFriction soundFingertip morphological featuresFig. 4: 2D optical graph and corresponding 3D topographic
map (including ridge and valley lines) of the left loop, whorl,
and right loop. (The 3D topographic areas of greater pressure
are marked as red. The dashed lines indicate the overall
direction of ridges lines, the triangles represent the singular
points of different patterns).

where Ra denotes the arithmetic mean value of the sliding
surface roughness, V represents the sliding speed, and n
and m are two independent exponents. Ra depends on the
morphological features of the finger and the phone screen
surface. Specifically, the surface morphology of the finger is
primarily determined by the pattern of the fingerprint. Figure 4
illustrates the different fingertip morphologies, including left
loop, right loop, and whorl patterns. The ridges of fingerprints
reduce the contact area between the finger pad and the phone
screen, resulting in variations in frictional radiation of air,
solid vibration, and wave propagation modes. Intuitively, this
is reflected in the characteristics of frictional sound waves [23].
In addition to the roughness of the finger pad and the phone
screen surface and the sliding speed of the finger, the charac-
teristics of the sound waves are also influenced by the pressure
applied by the finger, the humidity of the finger, and the sliding
posture of users [10].

3) A Feasibility Study: Given that each user possesses
unique surface morphology features on their fingerprints, we
assume that the generated frictional sound waves can disclose
the distinctive fingerprint pattern for certain types of mobile
phone screens.

Proof-of-concept Setup. We record friction sounds pro-
duced by 9 participants as they rub their fingers against the
screen of the Google Pixel 4, which is covered with a matte
screen protector. In the first experiment, participants gradually
increase the pressure and speed of their finger rubbing while
swiping their fingers across the smartphone screen 15 times.
In the second experiment, participants wrap their fingers with
transparent tape and repeat the same rubbing motion as in the
first experiment. Our objective is to identify specific acoustic
features in the friction sounds that correspond to different
fingerprint ridge patterns.

Result. To compare the characteristics of friction sounds,
Figure 5 presents the two-dimensional mapping of frictional
sound characteristics of Mel frequency cepstral coefficients
(MFCC), Mel spectrograms, and other frequency-domain and
time-domain features. Notably, each data point of the friction
sound formed a cluster corresponding to a unique fingerprint
pattern. As shown in Figure 5a, under the influence of pressure

4

(a)

(b)

Fig. 5: A proof of concept for two-dimensional mapping
of frictional sound characteristics of (a) different fingerprint
patterns; (b) blurred fingers.

TABLE I: Guessing framework of PrintListener.

Guessing path
Instant audio/video
Online meetings
Live streaming
Others

Example
Skype / Wecom / Facetime
Zoom / Google Meet
Twitch / Youtube Live / Discord
Malware / Swipe keyboard

and velocity, different fingerprint patterns can still be roughly
distinguished with overlap. However, the corresponding fea-
tures cannot be distinguished in Figure 5b when the fingers are
blurred. Thus, the fingerprint pattern depends on the fingerprint
itself rather than the shape of the fingertip set or the contact
area.

Based on the friction sounds, features such as MFCC can
be utilized to distinguish corresponding fingerprint patterns
within a small range. Hereafter, we aim to enhance the
accuracy of fingerprint feature prediction.

III. THREAT MODEL

Goal. The attacker aims to leverage PrintListener to deduce
extensive fingerprint information of users by analyzing the
friction sound produced when users swipe on their phone
screens while engaged in audio and video calls on social media
platforms. For instance, users commonly swipe their fingers
to scroll through the browse messages or news during audio
and video calls, or frequently perform swipe operations when
engaging in online gaming through social applications. In this
paper, we focus on generating targeted synthetic PatternMas-
terPrints with the following assumptions.

Capabilities and Knowledge. We assume the attacker can
access the victim’s finger friction sound. For example, the
attacker has the victim’s contact information, which could
include the victim’s phone number or account details for pop-
ular audio and video social software platforms (e.g., Google
Meet, Skype, Discord, etc). Then, the attacker may proceed to
initiate a voice or video call with the victim, connect to voice
collaboration software for playing games, or infiltrate the same
online voice or video conference as the victim. Malware with
recording permissions can also silently record the swiping fric-
tion sound. This assumption is close to the practical scenario.
The attacker’s guessing framework is illustrated in Table I.

Right loopRidgesValleysLeft loopWhorlSingular PointsRidge Direction3D Topographic Map2D Optical GraphWhorlRight loopLeft loopRight loopLeft loopWhorlFig. 6: The workflow of PrintListener.

No adversary proximity/accessibility. We assume the user
is cautious about the surrounding environment and protects
the fingerprint carefully. The user will not actively disclose
the fingerprint to others nearby or casually leave fingerprint
impressions on personal or public devices. Thus, attackers are
unable to obtain the fingerprint pattern information through
observation or proximity to the devices used by the user.

No access to personal devices. The attacker cannot obtain
any private devices (such as mobile phones, tablets, and other
electronic devices with interactive screens) touched by the user.
Once the PatternMasterPrints for the victim are created, their
efficacy can be tested on public devices that use the user’s
fingerprint, such as fingerprint-operated public devices used
for time attendance and access control. Besides, the attackers
can also directly sell generated candidate fingerprints to other
malicious parties, which will also cause a significant loss to
users.

No additional user action. Attackers cannot require any
additional actions from the user, such as asking the user to
rub their finger pads on the screen repeatedly. Throughout the
entire attack process, the attacker should ensure that the user is
unaware and only collects the sounds stealthily during regular
audio and video calls.

IV. DESIGN OF PRINTLISTENER

A. Overview

After eavesdropping on the user’s finger friction sound
through a social network, PrintListener generates a special-
ized PatternMasterprint sequence specifically designed for the
user’s fingerprint. As shown in Figure 6, the workflow consists
of three modules: Data Pre-processing, Fingerprint Pattern
Mapping, and PatternMasterPrint Synthesizing.

1) Data Pre-processing: This module separates weak fric-
tion sounds that are buried in dynamic speech and background
noise interference. High-pass filters are utilized to eliminate
low-frequency noise, and a spectrum density analysis method
is employed to obtain precise segments of the friction sound.
Finally, waveform resampling is applied to extend the frictional
segments and optimize the original dataset.

2) Fingerprint Pattern Mapping: This module employs
width and depth combination models to extract interpretable
audio and deep representation features from the augmented
friction sound segments. An adaptive weighting strategy is

employed to balance the predicted results of the primary
feature pattern.

3) PatternMasterPrint Generating: This module first sam-
ples independent and sequential PatternMasterPrints (PMPs).
Based on the sequential PMPs, a random restart hill-climbing
algorithm based on a crucial region synthesizes a PMP dic-
tionary in the secondary feature latent space. The synthetic
PMP dictionary is capable of high-frequency collision with
fingerprints of the same pattern.

B. Data Pre-processing

Once the raw audio and video call voice are obtained, Print-
Listener removes noise and compensates for the target fric-
tional sound signal. Subsequently, it obtains precise segments
of the frictional sound based on the frequency spectrum and
finally performs data augmentation on the acquired frictional
sound segments.

1) Background Noise Isolation: The intensity of finger
friction sound is generally low, and it is frequently subject
to interference from either steady background white noise or
non-stationary additive noise (e.g., dynamic speech or device
electronic noise). When we use mobile phones for audio and
video calls in indoor environments, such as family rooms and
offices, the noise energy is typically concentrated below 4 kHz.
Environmental noise can be disregarded when the frequency
range exceeds 8 kHz [24]. To eliminate low-frequency noise
while preserving the fingerprint information, we select a finite
impulse response (FIR) high-pass filter [25] with a 4 kHz
passband to erase the background noise.

2) Noise Compensation: The method based on short-time
amplitude and phase spectrum noise compensation [26] is
widely used to enhance the target signal degraded by additive
noise without introducing any distortion. The speech signal
is analyzed frame by frame, and the noisy speech is seg-
mented into overlapping frames by sliding windows. Under
the condition that the noise speech phase remains unchanged,
the corrected complex spectrum is obtained by aggregating the
corrected amplitude spectrum.

(cid:12)Z t [ωk](cid:12)
(cid:12)

(cid:12) = (cid:12)

(cid:12)GGA [ωk] · Y t [ωk](cid:12)
(cid:12) ,

(3)

where GGA [ωk] represents the gain function of the conven-
tional geometric approach. Y t[ωk] is the Fast Fourier Trans-
form (FFT) representation of it. Enhanced speech frames ˆxτ [n]

5

Friction soundNoise CompensationFriction Event SegmentationData Pre-processingBackground IsolationFingerprint Pattern MappingPatternMasterPrint Generating Independent PatternMasterPrint GeneratingData AugmentationWeighted Joint PredictionPredicted Finger Pattern Sequential PatternMasterPrint GeneratingCrucial Region (CR) Area DetectionmRMRFeatures SelectionSynthetic PatternMasterPrint GeneratingWide ModelDeep ModelVGGish-based CNN EncoderLeft loopRight loopWhorlMinutiae Random Restart Hill ClimbingFingerprint Minutia TemplatesMel Spectrogram OverlappingSpectrogram AnalysisFig. 7: Data pre-processing before friction event segmentation.

are synthesized by performing an Inverse Fast Fourier Trans-
form (IFFT) on the corrected cepstral coefficients Z τ [ωk].

ˆxτ [n] = Re (IFFT {|Z τ [ωk]| f (Z τ [ωk] + φτ [ωk])}) ,

(4)

where Re(·) denotes the real part of a complex number, τ
represents the frame number, Z τ [ωk] is the modified phase of
Z t[ωk], and f (Z τ [ωk]+φτ [ωk]) represents the corrected com-
plex spectrum obtained by adding the corrected phase spectrum
Z τ [ωk] and the corrected amplitude spectrum φτ [ωk]. The
final enhanced speech signal is synthesized using the standard
overlap and add method.

3) Friction Event Segmentation: To accurately extract spe-
cific friction sound segments from recordings and exclude
redundant
information about friction segments, we design
a friction sound event localization algorithm based on the
frequency spectrogram of audio signals. By analyzing the
energy spectrum density changes across the entire frequency
range, we perform friction sound event localization. Firstly,
we obtain the spectrogram of the original audio signal by
applying the Fourier transform. Then, for each time window,
we calculate the squared amplitude of the frequency-domain
signal of the audio frame, to obtain the power spectral density
of the original audio at different frequencies. Next, we conduct
a three-step detection on the entire activity signal:

Step 1: Silent regions exclusion. When friction sound
or activity noise (such as continuous human speech) occurs
or ends, the difference in energy density between adjacent
time windows increases, which is reflected in the spectrogram
as a sudden brightening or darkening of color. During silent
periods, the spectral energy is weaker, and the difference in
spectral energy between adjacent time windows is lower.

The audio energy spectral density P (n, k) of the k-th
frequency component at the n-th sample point in the time
window T is calculated as:

P (n, k) =

(cid:12)
(cid:12)
(cid:12)
(cid:12)
(cid:12)

N −1
(cid:88)

m=0

w(m) · x(n + m) · e−j2πkm/N

2

(cid:12)
(cid:12)
(cid:12)
(cid:12)
(cid:12)

,

(5)

where N denotes the number of samples in each time
window, w(m) is the window function, and x(n + m) denotes
the (n+m)-th sample point in the time window. This equation
uses the window function to smooth the data within the time
window and then performs a short-time Fourier transform to
obtain the energy spectral density of the different frequency
components. By moving the time window and comparing the

(a) rate = 0.8

(b) rate = 1.2

(c) n steps = 2

(d) n steps = -2

Fig. 8: Data augmentation

energy differences between adjacent time windows, we first
exclude the silent regions.

Step 2: Full-frequency energy verification. Through
extensive experimentation, we have observed that the spectral
energy of friction sounds uniformly increases across the entire
frequency range of 0 ∼ 22 kHz. However, common interfering
sounds, such as human speech are usually distributed in low-
frequency bands. Therefore, we set a threshold TDif for the
difference in spectral energy density between adjacent time
windows and a threshold TV ar for the variance of spectral
energy density across 0 ∼ 22 kHz of the same time window,
to analyze the potential friction events. When the difference
in spectral energy density between adjacent time windows is
greater than TDif , and the variance is smaller than TV ar, it
is considered a potential starting or ending point of friction
sound segments.

Step 3: Duration verification. After obtaining the poten-
tial friction events, we verify the duration of these activity
segments. Only when the duration of the activity noise exceeds
TDur, it is considered as the friction sound. Through extensive
testing, we adjust the values of TDif , TV ar, and TDur to
accurately locate friction sound events. As shown in Figure 7,
most friction sound fragments can be automatically segmented
out.

4) Data Augmentation: Waveform resampling is a common
data augmentation technique. In our study, we use time stretch-
ing and pitch shifting [27] to generate a range of similar yet
distinct waveforms. This expands our datasets and improves
the physical reality of samples (e.g. different swiping pressures

6

Frequency ( kHz)Sticky SoundSpeech NoiseAnthropogenic Noise01510520Time(s)21340.51.52.53.5(b) Spectrogram of original friction sound signal-150-130-140-110-120-100-90-80Power / Frequency (dB / Hz)Target Friction Signal(c) Spectrogram of Noise-compensated friction sound signalFrequency (kHz)1510520Time(s)21340.51.52.53.5Time(s)21340.51.52.53.5Amplitude (dB)-1-∞2-6-12-6(a) Oscillogram of original friction sound signalTarget Friction SignalSpeech NoiseOriginal audioAftertimestretchAfterpitchshift-0.08-0.040.040.080Amplitude36912015Time (seconds)036912Time (seconds)-0.08-0.040.040.080Amplitude369Time (seconds)012369Time (seconds)012TABLE II: Selected interpretable audio features.

Domain

y
c
n
e
u
q
e
r
F

l
a
r
t
s
p
e
C

Feature
LSF
Chroma
Spectral Kurtosis
Spectral Skewness
Spectral Contrast
Spectral Centroid
MFCC
LPCC
RASTA-PLP

Feature vector
f 1 − f 12
f 13 − f 24
f 25
f 26
f 27 − f 33
f 34
f 35 − f 73
f 74 − f 86
f 87 − f 99

and speeds).

Time Stretch. The user’s swiping speed can interfere with
the features of acoustic signals. PrintListener employs the
waveform similarity overlap-and-add (WSOLA) algorithm [28]
to modify the temporal scale of the signals. The original wave-
forms are accelerated by a factor of 0.8 and 1.2 respectively
(As shown in Figure 8a and 8b).

Pitch Shift. In the actual attack scenario, the configuration
and orientation of the microphone, the touchscreen material,
and the state of the user’s finger (dry or wet) can vary.
Therefore, even if the finger slides along the phone screen
in the same direction, the collected friction sound will have
a different pitch trajectory. To enhance the training data,
PrintListener utilizes pitch shifting to slightly alter the pitch
of the frictional sound waveforms, increasing or decreasing it
by 2 semitones (As shown in Figure 8c and 8d).

C. Fingerprint Pattern Mapping

After pre-processing the raw friction sound, PrintListener
utilizes wide models to extract interpretable audio features and
VGG-based deep models to extract deep representation fea-
tures, then employs an adaptive weighting strategy to balance
the predicted results.

1) Interpretable Audio Features Extraction: We use a two-
stage features selection strategy and analyze the interpretable
audio features comprising six frequency domain features and
three cepstral domain features.

Spectrogram Analysis. Since the important perceptual
characteristics of the audio signal are in the power spectrum,
we need to perform a frequency domain analysis. Firstly, the
friction sound is processed with framing and windowing to
prevent the spectral leakage of the signal. Next, the discrete
Fourier transform (DFT) is employed to convert the time-
domain signal to the frequency-domain signal. We obtain the
cepstrum coefficients for each frame of the frequency-domain
signal by performing a DCT transform on the logarithmic
amplitude of the DFT.

For each frame of the frequency-domain signal, we obtain
the cepstrum coefficients by performing a DCT transform on
the logarithmic amplitude of the DFT.

mRMR Feature Selection. After extracting common
frequency-domain and cepstral-domain features, we adopt the

7

minimum redundancy maximum relevance (mRMR) [29] cri-
terion and use mutual information to select features. Firstly,
we perform a first-order incremental search to maximize the
mutual
information between the selected features and the
class variable, thereby identifying a candidate feature set in
the feature space. Since mRMR only considers local optima
further to weigh the correlation and redundancy between
different features, we perform a second feature selection in
the way of Wrapper until the classification accuracy and fea-
ture dimensionality reduction are not improved. This process
allows for the extraction of six frequency-domain features and
three cepstral-domain features: linear spectral frequency (LSF),
chroma, spectral kurtosis, spectral skewness, spectral contrast,
spectral centroid, Mel-frequency cepstral coefficients (MFCC),
linear prediction cepstral coefficients (LPCC), and RASTA-
PLP. The dimensionalities of feature vectors are represented
in Table II.

2) Deep Representation Features Extraction: In this paper,
we leverage a VGGish-based model for audio analysis, which
has a powerful audio feature learning capability. We employ
the feature-based transfer learning method to fine-tune the
pre-trained VGGish-base model. Our VGG-like network is
trained to learn feature mappings from the high-dimensional
space of raw friction sound to the corresponding fingerprint
patterns. The implementation details of the VGGish-based
CNN Encoder are explained in APPENDIX A.

Mel Spectrogram Overlapping. We process the friction
segment in accordance with the input requirements of the
model network. Firstly, we calculate the one-sided short-time
Fourier transform (STFT) of the friction sound segment with
a 20 ms periodic Hann window and a 10 ms non-overlapping
shift. The spectral density is computed by Formula 5. Secondly,
we map the resulting spectrogram onto a 64-band Mel filter
bank to calculate the Mel spectrogram, which is buffered into
96 overlapping spectrograms. Thus, a 96 × 64 × 1 matrix
represents each friction sound segment.

VGGish-based CNN Encoder. We use a VGG-like model
trained on AudioSet, with 96×64×1 of friction sound segment
as input and a semantically meaningful 1 × 128 dimensional
embedding as output, which is sent to the downstream classi-
fication model. The deep network consists of multiple layers
of filters that capture different levels of features in the sound,
including 8 convolutional layers, 5 pooling layers, and 2 fully
connected layers.

3) Weighted Joint Prediction: To minimize overfitting and
accurately assess classification performance, we divide the
dataset into training, validation, and testing sets in a 6:2:2 ratio
and employ 10-fold cross-validation. We assessed the classi-
fication performance of four widely-used multi-classifiers: K-
Nearest Neighbors algorithm (KNN), Decision Tree, Random
Forest, and Adaboost. Meanwhile, due to traditional machine
learning models’ high interpretability but relatively low clas-
sification accuracy, PrintListener combines a CNN network to
calculate the joint distribution of features.

We employ three segments of friction sound to predict the
fingerprint pattern for each finger. The labels i can be 1, 2, or
3 and correspond to the left loop, right loop, or whorl pattern,
respectively. The predicted scorei for each pattern is calculated
as follows:

scorei =

3
(cid:88)

j=1

wwide · prewji + wdeep · predji,

(6)

where j represents the jth segment of friction sound used
for prediction, wwide / wdeep denote the weights assigned to
two modules, and prewji / predji indicates whether the jth
segment of friction sound is predicted as the ith fingerprint
pattern. If the jth segment is predicted as the ith pattern,
prewji/predji = 1, and otherwise, prewji/predji = 0. Cor-
responding weights are set for the two classification modules
through voting. From empirical analysis, the combination of
the KNN classifier and VGGish-like network is found to be
optimal. The final predicted fingerprint pattern corresponds to
the label with the highest value among score1, score2, and
score3.

D. PatternMasterPrint Synthesizing

This section introduces three methods for generating Pat-
ternMasterPrint: independent PatternMasterPrints generating,
sequential PatternMasterPrints generating, and synthesized
PatternMasterPrints generating. The first two methods involve
direct sampling from existing fingerprint datasets, while the
synthesized PatternMasterprints are generated by using a ran-
dom restart hill-climbing algorithm applied to a large dataset of
fingerprints with diverse patterns. This approach enables them
to adapt to various image qualities and features. These three
types of PatternMasterprints are designed explicitly for finger-
print authentication systems that rely on minutiae templates.
Given that conventional smartphone fingerprint authentication
systems often permit up to 5 attempts [30], PatternMasterPrints
are composed of five fingerprints intended to be used consec-
utively to compromise the authentication system.

1) Independent PatternMasterprint Generating: For a
dataset containing n left-loop fingerprint images, we set it
as dataset A and perform exhaustive matching on all finger-
prints in the dataset to identify the corresponding independent
PatternMasterPrint sequence. For each fingerprint image xi in
dataset A, we perform pairwise matching between xi and all
fingerprint images xj in dataset A to obtain the matching score
Mi, which is represented as follows:

Mi =

n
(cid:88)

j=1

p(xi, xj)/n,

p(xi, xj) =

(cid:26)1,
0,

score(xi, xj) > Λ,
otherwise,

(7)

(8)

where Λ represents the matching threshold, n is the number

of fingerprint images in dataset A.

We sort Mi

in descending order and select the finger-
print images corresponding to the top-five Mi as independent
left-loop PatternMasterPrints. Sampling independent Pattern-
MasterPrints from a fingerprint dataset offers the advantage
of computational simplicity and speed. However, there can
be some overlap in the set of victim fingerprints targeted
by different independent PatternMasterPrints. To address this

Fig. 9: Variation of attack successful rate with iterations of
hill-climbing algorithm.

issue, we introduce the second method to obtain sequential
PatternMasterPrints. The process is the same for the right
loop and whorl. In the following, we also take left-loop
PatternMasterPrints generating as an example.

2) Sequential PatternMasterPrint Generating: The sequen-
tial PatternMasterPrints are generated similarly to the indepen-
dent PatternMasterPrints but require 5 rounds of computation.
First, for each fingerprint image xi in dataset A, the matching
score Mi is calculated by Equations 7 and 8. The fingerprint
image corresponding to the highest Mi value is selected as the
first sequential left-loop PatternMasterPrint. This fingerprint
image is then removed from dataset A, and the previous steps
are repeated for each remaining fingerprint image to obtain the
second sequential PatternMasterPrint. This process is repeated
five times to yield five sequential PatternMasterPrints.

As both independent PatternMasterPrints and sequential
PatternMasterPrints are derived from real user fingerprint im-
ages, the randomness of features (e.g. the size of the fingerprint
area, the position or direction of minutiae) is low. This will
reduce their matching ability when attacking other fingerprint
images. Moreover, finding PatternMasterPrints with a high
impersonation rate in selected fingerprint image datasets may
not always be possible. To address these issues, we propose
a random restart Hill-climbing algorithm for synthesizing Pat-
ternMasterPrints artificially and improving their generalization
ability.

3) Synthetic PatternMasterPrint Generating: In this sec-
tion, we iterate on the previously generated sequential Pattern-
MasterPrints using a random-restart hill-climbing algorithm to
obtain synthetic PatternMasterPrints. To improve the efficiency
of the iteration process, we detect the central region (CR) area
in the fingerprints.

Crucial Region Area Detection. The detailed template
of a fingerprint is determined by the details and orientation
of each secondary feature of the fingerprint, such as ridge
endings and bifurcation points. The seed fingerprint is divided
into multiple units. To improve the efficiency of the iteration,
we analyze the central region (CR) of fingerprints, which
is the area with a high probability of fingerprint minutiae
collision in the fingerprint image and is usually located in
the middle region. We first define a middle area of 200 x
250 pixels as a crucial region and then conduct the random
restart hill-climbing algorithm iteration within the CR area. To

8

 ϭ Ϭ Ϭ Ϯ Ϭ Ϭ ϯ Ϭ Ϭ ϰ Ϭ Ϭ ϱ Ϭ Ϭ / ƚ Ğ ƌ Ă ƚ ŝ Ž Ŷ Ɛ Ϭ ϭ Ϭ Ϯ Ϭ ϯ Ϭ ϰ Ϭ ϱ Ϭ  ƚ ƚ Ă Đ Ŭ  ^ Ƶ Đ Đ Ğ Ɛ Ɛ Ĩ Ƶ ů  Z Ă ƚ Ğ  ; й Ϳ E Ž ͘ ϭ  ^ z D W E Ž ͘ Ϯ  ^ z D W E Ž ͘ ϯ  ^ z D W E Ž ͘ ϰ  ^ z D W E Ž ͘ ϱ  ^ z D WTABLE III: Thresholds of three fingerprint datasets at different false acceptance rates (F AR).

Datasets
F AR (%)
Threshold

FingerPassDB7
0.1
19

0.01
28

1
12

Livedet2011 ItalData
0.01
0.1
1
31
25
18

PatternFinger
0.1
31

0.01
41

1
23

prevent unnecessary iterations, each unit covers a ridge spacing
on the fingerprint (e.g., 9 pixels on a 500 dpi fingerprint
image). Meanwhile, PrintListener quantizes the range of the
fine direction into 16 equidistant intervals, each representing a
change of 22.5 degrees.

Minutiae Random Restart Hill Climbing. The specific
iterative algorithm aims to obtain the synthetic PatternMaster-
Print template by searching within the training set and maxi-
mizing the Attack Success Rate (ASR). Due to the difficulty
of searching the entire space, this paper adopts a local search
approach to synthesize the PatternMasterPrint. Specifically, a
random-restart hill-climbing algorithm is employed to synthe-
size the PatternMasterPrint. The initial seed for each round is
the sequential PatternMasterPrint obtained in Section IV-D2.
In each round, the details of the template are incrementally or
degressively modified to create a new PatternMasterPrint and
iterative modifications are made based on the output scores
of the matcher. Throughout the iterative process, the best-
performing detail template serves as the stored state. If the
new iteration of hill-climbing produces a template f px that
is better than any previous generation, it replaces the existing
stored state. The iteration stops when f px meets the given
Attack Success Rate or the hill-climbing iterations reach the
predefined maximum value. The details of the random restart
hill-climbing algorithm 1 are described in APPENDIX.

Figure 9 displays the results of finger-level matching on
the dataset using synthetic PatternMasterPrints. By applying
a hill-climbing method on the top five ranked synthetic Pat-
ternMasterPrints, they are regenerated from the training set. If
the attack success rate remains unchanged within a predefined
number of iterations, it indicates that the performance of the
synthetic PatternMasterPrints is stabilizing. The hill-climbing
process is terminated if the success rate does not improve
within 100 iterations. From Figure 9, it can be observed that
after 300 iterations, the improvement in the success rate of
the iteratively generated PatternMasterPrint becomes smaller.
Therefore, we set the maximum iteration count, denoted as
imax, to 500.

V. EVALUATION

In this section, we present the experimental results based

on our PrintListener prototype.

A. Experimental Settings

Data Collection. After obtaining IRB approval, we re-
cruited 65 participants (24 females and 41 males, aged 18
to 30) to participate in our experiments. Some participants
exhibit mild perspiration on their hands as we do not require
participants’ fingers to be completely dry. Thus, our datasets
match the finger characteristics of a larger and more diverse
population. Before data collection, the participants were pro-
vided with a detailed explanation of the study’s purpose and

signed an informed consent form. We observed the fingerprint
patterns of all ten fingers of the participants and selected 1-
6 fingers for each participant to collect fingerprint friction
sounds and corresponding fingerprint images. As imbalanced
datasets could introduce bias in the feature weighting for
the three fingerprint patterns and lead to skewed predictive
results, we chose a different number of fingers for different
participants to balance the ratio of the whorl, left loop, and
right loop fingerprints at 1:1:1. We collect friction sounds
of 180 fingers (60 each of left loop, right loop, and whorl
patterns) and use the ZKLive20R optical fingerprint scanner to
get corresponding optical fingerprint images. For each chosen
finger, the participant swiped on the screen 25 times based on
their regular usage habits. The details of compiled datasets are
explained in APPENDIX B.

Default Setting. The sampling rates of instant messaging
software are usually set from 16 kHz to 44.1 kHz. For
consistency and comparability, we set the basic sampling rate
at 44.1kHz. Our experiments are conducted in three different
environments: a conference room (quiet), an office (slightly
noisy), and a playground (noisy). We collected the finger-
swiping friction sound on three commercial smartphones: the
Google Pixel 4, iPhone 13, and Samsung A20S. By default, we
used Google Pixel 4 as the evaluation platform in a conference
room. All smartphones were coated with matte screen protec-
tors. We utilized the open-source NBIS fingerprint matcher
[31] to conduct exhaustive matching of fingerprint images in
the FingerPassDB7, Livdet2011 ItalData, and FingerPattern
datasets.

B. Metrics

We employ accuracy, weighted-average precision (wP ),
weighted-average recall (wR), and F1 score as evaluation
metrics to assess the performance of our fingerprint pattern pre-
diction model. Accuracy measures the proportion of correctly
classified samples among all samples. T P (True Positive) is the
number of positive samples correctly predicted as positive, F P
(False Positive) is the number of negative samples incorrectly
predicted as positive, and F N (False Negative) is the number
of positive samples incorrectly predicted as negative. For each
fingerprint type, we calculate its precision (T P/(T P + F P ))
and recall (T P/(T P + F N )) separately. The precision and
recall of the left loop, right loop, and whorl are weighted
according to the dataset proportions to obtain wP and wR. To
comprehensively evaluate the precision and recall performance
of our friction sound classification model, we also select F1
score (2 · wP · wR/(wP + wR)) to balance the two metrics.

The False Acceptance Rate (F AR) is the proportion of
unauthorized user data mistakenly accepted as genuine users.
A lower F AR value signifies increased security, albeit with
diminished usability for the authentication system. To evaluate
the attack capability of PrintListener within various security

9

TABLE IV: Results by using different classifiers (P: Precision, R: Recall).

Module

YAMnet+KNN
YAMnet+Decision Tree
YAMnet+Ramdom Forest
YAMnet+Adaboost
VGGish-like+KNN
VGGish-like+Decision Tree
VGGish-like+Ramdom Forest
VGGish-like+Adaboost
Resnet34+KNN
Resnet34+Decision Tree
Resnet34+Ramdom Forest
Resnet34+Adaboost

Accuracy
0.709
0.820
0.731
0.776
0.884
0.766
0.739
0.774
0.746
0.686
0.753
0.686

P (left loop/right loop/whorl) R (left loop/right loop/whorl) F1 score

0.760 / 0.712 / 0.657
0.828 / 0.758 / 0.881
0.718 / 0.767 / 0.705
0.778 / 0.751 / 0.797
0.939 / 0.865 / 0.857
0.777 / 0.791 / 0.736
0.711 / 0.779 / 0.725
0.831 / 0.737 / 0.764
0.739 / 0.706 / 0.815
0.746 / 0.673 / 0.652
0.795 / 0.735 / 0.735
0.647 / 0.900 / 0.658

0.747 / 0.694 / 0.685
0.840 / 0.804 / 0.816
0.614 / 0.825 / 0.755
0.762 / 0.715 / 0.851
0.915 / 0.895 / 0.887
0.800 / 0.672 / 0.825
0.752 / 0.835 / 0.632
0.696 / 0.736 / 0.889
0.763 / 0.841 / 0.633
0.644 / 0.697 / 0.718
0.712 / 0.681 / 0.865
0.771 / 0.612 / 0.675

0.709
0.821
0.731
0.775
0.886
0.767
0.739
0.776
0.750
0.688
0.754
0.710

Fig. 10: Confusion matrix of different models with/without data augmentation.

settings, we establish three distinct security gradients for the
fingerprint matcher (F AR = 1%, F AR = 0.1%, F AR =
0.01%). We utilize the open-source NBIS fingerprint matcher
[31] to conduct exhaustive matching of fingerprint images in
Dataset-4 FingerPassDB7, Dataset-5 Livedet2011 ItalData,
and Dataset-6 PatternFinger. Subsequently, we determine the
corresponding threshold values for the three F AR settings.
The ultimate threshold values for the three fingerprint datasets
under different F AR settings are presented in Table III.

We use a weighted attack success rate (wASR) to evaluate
the effectiveness of PatternMasterPrint. For the i-th type of fin-
gerprints, the wASR of generated PatternMasterPrint wASRi
is calculated as:

wASRi =

3
(cid:88)

j=1

Rij · ASRji,

(9)

where i and j take the values 1, 2, or 3, representing the
left-loop, right-loop, and whorl patterns, respectively. Rij
represents the probability of predicting a fingerprint with i-
th type features as the j-th class fingerprint. ASRji denotes
the attack success rate of the j-th PatternMasterPrint against
the i-th type of fingerprint.

C. Fingerprint Pattern Prediction

In this section, we compare the performance of different
wide & deep modules on friction sound classification and then
select the best one (our wav2pattern model) for the subsequent
evaluation. After determining the chosen combined network,

we assess the effects of data augmentation, sampling rate,
environmental noise, and friction sound collection equipment.

1) Impact of Combined Network: We evaluate the im-
pact of various joint models on Dataset-1. We compare our
VGGish-like model with a sound-based convolutional neu-
ral network YAMnet [32], and an image-based CNN model
ResNet34 [33]. By utilizing combination and permutation,
we can obtain 12 distinct wide and deep joint classification
networks. The results of our experiments are presented in
Table IV, which displays the classification accuracy, precision,
recall, and F1 scores of each joint network. The results show
that our wav2pattern network outperforms the others with an
accuracy of 88.4%. Moreover, the classifier does not show
a significant bias towards any particular type of fingerprint.
Figure 10a shows the confusion matrix of wav2pattern. We
the wide
compared the classification performance without
module (KNN) as shown in Figure 10b. When predicting with-
out the joint use of the wide module, the classification accuracy
is 73.09%, which decreased by approximately 15%. Since our
wav2pattern combined network achieves the best performance,
we use wav2pattern for the subsequent evaluation.

2) Impact of Data Augmentation: To evaluate the impact of
data augmentation on PrintListener, we compare the accuracy
of wav2pattern on Dataset-1 before data augmentation (a total
of 9,000 friction segments), as shown in Figure 10a and 10c.
Wav2pattern achieves an average classification accuracy of
83.09% without data augmentation, while data augmentation
increased the accuracy by 5%. We attribute this improvement
to the variation in users’ sliding speed and pressure. Our data

10

0.7780.0830.0560.0940.0590.8650.0750.1400.8500.6780.0830.1060.1440.0590.7650.1750.2400.7500.8420.0680.0300.0750.0240.9150.0610.0890.895Ture ClassPredicted Class(a) Wide& Deep module + Dataaugmentation (c) Deep module + Dataaugmentation (b) Wide & Deep module Left loopRight loopWhorlLeft loopRight loopWhorl0.90.80.70.60.50.40.30.20.1Left loopRight loopWhorlLeft loopRight loopWhorl(a)  Wide & deep model + data augmentation(b)Deep model +data augmentation (c)Wide & deep models(a) PatternFinger

(b) FingerPassDB7

(c) Livedet2011 ItalData

Fig. 11: The wASR of three methods for generating PatternMasterPrint on different datasets with the F AR set at 0.1%.

TABLE V: Impact of environments.

Environment
Office
Laboratory
Corridor

wP
0.887
0.880
0.875

wR
0.884
0.878
0.874

F1 score
0.886
0.879
0.874

is insensitive to low ambient noise since the user’s fingerprint
features are widely distributed in different frequency bands.

5) Generalization: We conduct generalization experiments
to evaluate the robustness of wav2pattern to hardware finger-
print and quality. We select 4,500 friction sound segments
(Dataset-3) collected from a Samsung A20s phone with a long-
term used microphone. Then, we examine whether the joint
network trained on Pixel 4 and iPhone 13 can accurately clas-
sify them. The result demonstrates a certain degree of robust-
ness to microphone hardware fingerprint and quality, achieving
an accuracy of 80.32%. The accuracy on the Samsung A20s
only decreases by approximately 8% compared to that on
Pixel 4 and iPhone 13, indicating that hardware fingerprint
and quality slightly impact the friction sound features.

D. PatternMasterprint Attack

1) Impact of Fingerprint integrity: Figure 11 illustrates
the weighted attack success rates (wASR) of three meth-
ods
for generating PatternMasterPrint on partial finger-
prints (Dataset-5 FingerPassDB7) and complete fingerprints
(Dataset-4 PatternFinger and Dataset-6 Livedet2011 ItalData
), with the F AR set at 0.1%. It can be observed that the wASR
of partial fingerprints is significantly higher than that of com-
plete fingerprints. This is because partial fingerprints are more
prone to erroneous matching due to the loss of information
entropy. Moreover, the wASR of synthetic PatternMasterPrints
is generally higher than that of sequential PatternMasterPrints
and independent PatternMasterPrints. Therefore, we use syn-
thetic PatternMasterPrints as an example to discuss the impact
factors of the attack success rate of PrintListener.

2) Impact of FAR security setting: The impact of the F AR
value on wASR of PatternMasterPrint attacks in Dataset-
4 PatternFinger is demonstrated in Figure 13a. We can see
the wASR decreases at a lower F AR value setting
that

Fig. 12:

Impact of sampling frequencies.

augmentation technique, which involves applying pitch shifting
and time stretching, helps to alleviate the impact of sliding
speed and pressure on the recorded sounds, thereby improving
the accuracy of our results.

3) Impact of Sampling Rate: We evaluate the performance
of PrintListener under four common sampling rates used
in current audio and video social networking software. The
friction segments at different sampling rates are obtained by
downsampling 45,000 data points from Dataset-1. As shown
in Figure 12, the recall in classifying the fingerprint pattern
gradually decreases as the sampling rate decreases. This is
because the effective information contained in the audio signal
inevitably reduces as the cutoff frequency decreases. Specif-
ically, high-frequency sound segments contain more detailed
information than low-frequency sound segments. Notably, the
classification accuracy of sound at 44.1 kHz and 32 kHz
sampling rates does not exhibit a significant distinction. 32
kHz is a commonly used sampling rate in most audio and
video social networking software.

4) Impact of Environments: We conduct a comparison of
two noise environments: (a laboratory, Dataset-2A) and a corri-
dor (Dataset-2B) with a quiet environment (an office, Dataset-
1). As shown in Table V, the weighted-average precision (wP ),
weighted-average recall (wR), and F1 score slightly decreased
when friction sounds are affected by low noise. Even in noisy
corridors with speech or crowd-walking sounds, the wP is
87.5%, a decrease of only 1.2%. This indicates that our model

11

Synthetic PMP, whorl Sequential PMP, whorl Independent PMP, whorl  Synthetic PMP, right loop         Sequential PMP, right loop  Independent PMP, right loop Synthetic PMP, left loop         Sequential PMP, left loop  Independent PMP, left loop  ϭ Ϯ ϯ ϰ ϱ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϭ ϭ ϱ ϯ Ϭ ϰ ϱ ϲ Ϭ ǁ  ^ Z  ; й Ϳ ϭ Ϯ ϯ ϰ ϱ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϯ ϱ ϯ ϱ ϰ ϱ ϱ ϱ ϲ ϱ ǁ  ^ Z  ; й Ϳ ϭ Ϯ ϯ ϰ ϱ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϭ ϭ Ϭ Ϯ Ϭ ϯ Ϭ ϰ Ϭ ǁ  ^ Z  ; й Ϳ ϭ ϲ Ϯ ϰ ϯ Ϯ ϰ ϰ ͘ ϭ d Ś Ğ  Ɛ Ă ŵ Ɖ ů ŝ Ŷ Ő  Ĩ ƌ Ğ Ƌ Ƶ Ğ Ŷ Đ Ǉ  ; Ŭ , ǌ Ϳ ϱ ϱ ϲ ϱ ϳ ϱ ϴ ϱ ϵ ϱ Z Ğ Đ Ă ů ů  ; й Ϳ > Ğ Ĩ ƚ  ů Ž Ž Ɖ Z ŝ Ő Ś ƚ  ů Ž Ž Ɖ t Ś Ž ƌ ůFig. 13: The wASR of synthetic PatternMasterPrint and baseline on different datasets.

(higher security), while more test subjects can be successfully
attacked at a higher F AR value setting (lower security).
When the F AR is set to 0.1% (a value that balances security
and usability), PrintListener can successfully attack 52% of
users with the left loop fingerprints, 48.8% of users with the
right loop fingerprints, and 53.7% of users with the whorl
fingerprints within 5 attempts. When the F AR is set to 0.01%,
the wASR is 9.8% for users with the left loop fingerprints,
7.9% for users with the right loop fingerprints, and 7.8% for
users with the whorl fingerprints within 5 attempts. Therefore,
PrintListener will indeed pose a huge threat to real fingerprint
authentication systems.

E. Baseline Comparisons

Finally, we compare the performance of PrintListener and
the related fingerprint attack method, i.e., MasterPrint [8]. Due
to the lack of open sourcing of DeepMasterPrint [9] and the
unavailability of critical information regarding its training pro-
cess, it is inaccessible to set DeepMasterPrint as a baseline for

comparison with PrintListener. Besides, DeepMasterPrint only
conducted attack experiments on partial fingerprint images, not
applicable to this work.

Partial Fingerprints. Firstly, we compare the wASR
for attacking Dataset-
of Printlistener and MasterPrint
5 FingerPassDB7 at three F AR settings. As shown in Fig-
ure 13b, the MasterPrint sequences selected through pattern
prediction generally have higher attack success rates than those
without pattern prediction. At a F AR of 0.1%, Printlistener
demonstrates an average improvement of 37.0% in wASR
compared to the attack success rate of Masterprint in only
one attempt. At the highest security F AR setting of 0.01%,
PrintListener achieves the average wASR of 27.9% within
5 attempts which is 1.8 times the attack success rate of
Masterprint.

Complete Fingerprints. Next, we compare the wASR
for attacking Dataset-
of Printlistener and MasterPrint
6 Livedet2011 ItalData at
three F AR settings. As shown
in Figure 13c, when the F AR is 0.1%, Printlistener can

12

 Ϯ ϰ ϲ ϴ ϭ Ϭ ϭ ϱ Ϯ ϯ ϰ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϭ ϲ ϭ Ϯ ϭ ϴ Ϯ ϰ ϯ Ϭ ǁ  ^ Z  ; й Ϳ(b)FingerPassDB7 ϭ ϱ Ϯ ϯ ϰ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϯ Ϭ ϯ Ϭ ϰ Ϭ ϱ Ϭ ϲ Ϭ ϳ Ϭ ǁ  ^ Z  ; й Ϳ(c)Livedet2011 ItalData ϭ Ϯ ϱ d Ś Ğ  ϯ ϰ Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ ϱ ϱ ϲ ϱ ϳ ϱ ϴ ϱ ϵ ϱ ǁ  ^ Z  ; й Ϳ(a)PatternFinger ϭ ϱ Ϯ ϯ ϰ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϭ ϯ ϲ ϵ ϭ Ϯ ǁ  ^ Z  ; й Ϳ ϭ ϱ Ϯ ϯ ϰ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϭ ϭ Ϭ Ϯ Ϭ ϯ Ϭ ϰ Ϭ ǁ  ^ Z  ; й Ϳ ϭ ϱ Ϯ ϯ ϰ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ ϯ Ϭ ϰ Ϭ ϱ Ϭ ϲ Ϭ ϳ Ϭ ϴ Ϭ ǁ  ^ Z  ; й Ϳ ϭ Ϯ ϯ ϰ ϱ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϭ Ϯ ϰ ϲ ϴ ϭ Ϭ ǁ  ^ Z  ; й Ϳ ϭ ϱ Ϯ ϯ ϰ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϭ ϭ ϱ ϯ Ϭ ϰ ϱ ϲ Ϭ ǁ  ^ Z  ; й Ϳ ϭ ϱ Ϯ ϯ ϰ d Ś Ğ  Ŷ Ƶ ŵ ď Ğ ƌ  Ž Ĩ  Ă ƚ ƚ Ğ ŵ Ɖ ƚ Ɛ Ϭ Ϯ Ϭ ϰ Ϭ ϲ Ϭ ϴ Ϭ ǁ  ^ Z  ; й ͿFAR = 0.01%FAR = 0.1%FAR = 1%Fig. 14: Frictional sound on different apps.

TABLE VI: Sampling rate statistics for Social networking
applications.

Apps
Skype
FaceTime
Google Meet
Microsoft Teams
Wecom

Sampling rates (kHz)
8 / 12 / 16 / 24
8 / 12 / 16 / 24
24 / 32
16 / 32
16 / 24

Fig. 15: Frictional sound on different films.

successfully attack 37.7% of left loop fingerprints, 34.9% of
right loop fingerprints, and 33.0% of whorl fingerprints within
5 attempts, representing a significant improvement of 93.3%,
79.0%, and 69.2%, respectively, compared to the performance
of MasterPrint. At the highest security F AR setting of 0.01%,
PrintListener achieves an average wASR of 9.3% within
5 attempts which is 2.6 times the attack success rate of
Masterprint.

sample rates. As shown in Figure 12, the recall in classifying
the fingerprint pattern gradually decreases as the sampling
rate decreases. Limiting the audio sampling rate can reduce
the leakage of fingerprint
information to a certain extent.
Additionally, audio/video social apps can destroy finger fric-
tional sound features with automatic speech noise reduction,
or implement pop-up reminders to caution users to be careful
when performing swiping operations while the microphone is
in use.

VI. DISCUSSION

C. Limitations

A. Attack Feasibility via Social Networking Apps

The prevalent sampling rates used by popular audio and
video social networking Apps are shown in table VI. It can be
observed that the majority of software applications maintain
sampling rates above 16 kHz, which is the minimum sampling
rate in our evaluation. As stated in the evaluation, when the
sampling rate is sufficiently high, audio compression does not
interfere with the extraction of friction sound signal features.
When an attacker covertly records the transmitted audio signal
in the background, the spectrogram retains distinct features
that can be utilized. This indicates that our inference of
friction sound features can be extended to other audio and
video software applications. We collect swipe sounds of users
on several social software. The spectrograms are shown in
Figure 14.

B. Defense

A simple countermeasure to prevent the leakage of finger
friction sound containing fingerprint features is to correct some
users’ habits. For example, users try not to swipe their fingers
on the phone screen when making audio and video calls
on social media platforms. However, it is difficult to avoid
not performing the swiping operation in some scenarios, e.g.,
engaging in online gaming on mobile phones or tablets through
social applications. In addition, we suggest that audio/video
social and communication apps be limited to lower audio

Different screen protectors on mobile devices may af-
fect the sliding friction sound characteristics, weakening our
model’s classification accuracy and robustness. Screen protec-
tors can be divided into matte film and glossy film according
to the processing technology. When users interact with mobile
devices, their fingers will come into contact with the screen
surfaces, leaving behind fingerprint marks in the form of grease
or sweat stains. These marks can be exploited by attackers
to create fake fingerprints based on the imprints [34]. As a
solution, matte films have gained popularity recently due to
their ability to prevent fingerprint leakage and reduce glare
[35], [36], [37]. Our experiments evaluated the finger friction
sound produced on the matte film. At the same time, we also
tested the friction effects on the glossy film, the spectral energy
of frictional sound is a little weak (Figure 15). However, with
longer use, the surface of the glossy film will become rough,
and the friction sound will be more pronounced.

VII. RELATED WORK

A. Fingerprint Template Attack

An Automated fingerprint

identification system (AFIS)
recognizes individuals by comparing the spatial distribution
of ridge structures and detail points [38], [39]. Although
AFIS is considered to be efficient and practical,
they are
also vulnerable to presentation attack. This attack usually
uses some commonly available materials (e.g., gelatin, wax,

13

Frequency ( kHz)0Time(s)21340.51.52.53.5(b) Microsoft Teams sound signal-150-130-140-110-120-100-90-80Power / Frequency (dB / Hz)(c) Wecomsound signalFrequency (kHz)Time(s)21340.51.52.53.5Time(s)(a) Google Meet sound signal012841621340.51.52.53.5Frequency ( kHz)12841601284Frequency(kHz)-150-130-140-110-120-100-90-80Power/Frequency (dB/Hz)15105202130.51.52.5Time(s)(a) Matte film2130.51.52.5Time(s)(b) Glossy filmliquid latex, etc) to create fake synthetic fingerprints, such as
2D fingerprint images and 3D fake fingers. Cao et al. [40]
successfully spoofed the Samsung Galaxy S6 and Huawei
Honor 7 using 2D fingerprint images printed on special paper.
The research of CISCO [41] chose fabric glue to create 3D
fake fingers, achieving an average success rate of 80% in
bypassing AFIS. However, in practical scenarios, collecting
one clear fingerprint image of the victim is challenging. To
avoid this problem, Roy et al. [8] generated Masterprint based
on a large set of fingerprint data that could match one or more
fingerprint templates of a vast number of users by chance.
Bontrager et al. [9] leveraged a GAN network to generate
image-level MasterPrint, called DeepMasterPrint. However, the
attack success rate of Masterprint and DeepMasterPrint is very
low at a high-security level. Chen et al. [42] discovered a
design flaw in the smartphone fingerprint authentication system
that allows unlimited attempts at fingerprint authentication,
thereby enabling brute-force attacks without requiring any
prior knowledge of the victim. Some simple measures can
make up for this design flaw, e.g., checking whether a can-
cellation happens during each fingerprint authentication.

B. User Information Inferring via Acoustic

The acoustic signals related to the user contain some impor-
tant user information. There have been some studies on stealing
users’ private information through acoustic signals. Keyboard
recognition based on acoustic emanation has been studied
in [43], [44], [45], [46], [47], [48]. These approaches utilized
slightly different keystroke sounds between different keys,
or the difference in arrival time of the keystroke sounds of
different keys to infer the user’s keystroke information. Among
them, [46], [47], [48] leveraged a malicious APP on a mobile
device to identify keystrokes on a nearby keyboard, thereby
eavesdropping on the user’s keyboard input. Arp et al. [49]
explored the functionality, prevalence, and technical limitations
of ultrasonic beacons embedded in audio and tracked users
using mobile device microphones. PatternListener [50] and
UltraPIN [51] utilized the user’s smartphone to emit ultrasonic
signals to track the user’s finger movement and infer the unlock
pattern and PIN respectively. PrintListener is the first work to
infer users’ fingerprint information using acoustic signals.

VIII. CONCLUSION

In this paper, we uncover a new side-channel attack on
fingerprint authentication and propose PrintListener, which
utilizes users’ fingertip-swiping actions on the screen to extract
fingerprint features and synthesizes powerful PatternMaster-
Print sequences for fingerprint dictionary attacks. Extensive
experimental results in real-world scenarios demonstrate that
Printlistener can attack up to 26.5% of partial fingerprints
and 9.3% of complete fingerprints within five attempts at the
highest security FAR setting of 0.01%, which far exceeds the
attack potency of MasterPrint.

ACKNOWLEDGMENTS

We would like to thank the anonymous reviewers for their
helpful feedback. Man Zhou’s work was partially supported
by the NSFC under Grants 62202180. Qian Wang’s work was
partially supported by the NSFC under Grants U20B2049 and
U21B2018. Qi Li’s work was partially supported by the NSFC

under Grant 62132011. Xiaojing Ma’s work was partially
supported by the NSFC under Grants 62272175.

REFERENCES

[1] Most

Used

Methods

Screens,
2022,
screen-lock-methods-global-users/.

to

Phone
https://www.statista.com/statistics/1305993/

Mobile

Secure

[2] Fingerprint Payment Market Research, 2031, 2022, https://www.

alliedmarketresearch.com/fingerprint-payment-market-A12966.

[3] Safety & Security of U.S. Borders: Biometrics, 2023, https://travel.state.

gov/content/travel/en/us-visas/other-visa-categories/safety.html.

[4] Automated Fingerprint

Identification Market Size Growing at
23.1% CAGR Set to touch USD 99.9 Billion By 2032, 2023, https:
//www.globenewswire.com/news-release/2023/04/03/2639837/0/en/
Automated-Fingerprint-Identification-Market-Size-Growing-at-23-1-CAGR-Set-to-touch-USD-99-9-Billion-By-2032.
html.

[5] Y. Zhang, Z. Chen, H. Xue, and T. Wei, “Fingerprints on mobile devices:

Abusing and leaking,” in Black Hat Conference, 2015.

[6] B. A. Fisher and D. R. Fisher, Techniques of crime scene investigation.

crc Press, 2012.

[7]

I. Echizen and T. Ogane, “Biometricjammer: method to prevent ac-
quisition of biometric information by surreptitious photography on
fingerprints,” IEICE TRANSACTIONS on Information and Systems, vol.
101, no. 1, pp. 2–12, 2018.

[8] A. Roy, N. Memon, and A. Ross, “Masterprint: Exploring the vul-
nerability of partial fingerprint-based authentication systems,” IEEE
Transactions on Information Forensics and Security, vol. 12, no. 9,
pp. 2013–2025, 2017.

[9] P. Bontrager, A. Roy, J. Togelius, N. Memon, and A. Ross, “Deep-
masterprints: Generating masterprints for dictionary attacks via latent
variable evolution,” in Proc. of IEEE BTAS, 2018, pp. 1–9.

[10] A. S. Rathore, W. Zhu, A. Daiyan, C. Xu, K. Wang, F. Lin, K. Ren,
and W. Xu, “Sonicprint: a generally adoptable and secure fingerprint
biometrics in smart devices,” in Proc. of ACM MobiSys, 2020, pp. 121–
134.

[11] Z. Shu, Z. Wang, G. Yang, C. Zang, Z. Ma, F. Lin, and K. Ren,
“Fingersound: A low-cost and deployable authentication system with
fingertip sliding sound,” in Proc. of IEEE ICPADS, 2023, pp. 33–40.

[12] M. Zhou, Y. Zhou, S. Su, Q. Wang, Q. Li, S. Hu, C. Yu, and Z. Li,
“Fingerpattern: Securing pattern lock via fingerprint-dependent friction
sound,” IEEE Transactions on Mobile Computing, vol. PP, pp. 1–1,
DOI: 10.1109/TMC.2023.3 338 148, 2023.

[13] Skype — Stay connected with free video calls worldwide, 2023, https:

//www.skype.com/en/.

[14] WeChat, 2023, www.wechat.com.
[15] Use FaceTime with your iPhone or iPad, 2022, https://support.apple.

com/en-us/HT204380.

[16] R. Cappelli, D. Maio, A. Lumini, and D. Maltoni, “Fingerprint image
reconstruction from standard templates,” IEEE Transactions on Pattern
Analysis and Machine Intelligence, vol. 29, no. 9, pp. 1489–1503, 2007.
[17] K. Karu and A. K. Jain, “Fingerprint classification,” Pattern Recogni-

tion, vol. 29, no. 3, pp. 389–404, 1996.

[18] D. Yambay, L. Ghiani, P. Denti, G. L. Marcialis, F. Roli, and S. Schuck-
ers, “Livdet 2011—fingerprint liveness detection competition 2011,” in
Proc. of IEEE ICB, 2012, pp. 208–215.

[19] D. Maltoni, D. Maio, A. K. Jain, S. Prabhakar et al., Handbook of

fingerprint recognition. Springer, 2009, vol. 2.

[20] A. Akay, “Acoustics of friction,” The Journal of the Acoustical Society

of America, vol. 111, no. 4, pp. 1525–1548, 2002.

[21] H. Zahouani, R. Vargiolu, G. Boyer, C. Pailler-Mattei, L. Laqui`eze, and
A. Mavon, “Friction noise of human skin in vivo,” Wear, vol. 267, no.
5-8, pp. 1274–1280, 2009.

[22] H. B. Abdelounis, A. Le Bot, J. Perret-Liaudet, and H. Zahouani, “An
experimental study on roughness noise of dry rough flat surfaces,” Wear,
vol. 268, no. 1-2, pp. 335–345, 2010.

14

[23] S. Derler, M. Preiswerk, G.-M. Rotaru, J.-P. Kaiser, and R. Rossi,
“Friction mechanisms and abrasion of the human finger pad in contact
with rough surfaces,” Tribology International, vol. 89, pp. 119–127,
2015.

[24] M. Zhou, Q. Wang, K. Ren, D. Koutsonikolas, L. Su, and Y. Chen,
“Dolphin: Real-time hidden acoustic signal capture with smartphones,”
IEEE Transactions on Mobile Computing, vol. 18, no. 3, pp. 560–573,
2019.

[25] Y. Neuvo, D. Cheng-Yu, and S. Mitra, “Interpolated finite impulse
response filters,” IEEE Transactions on Acoustics, Speech, and Signal
Processing, vol. 32, no. 3, pp. 563–570, 1984.

[26] M. T. Islam, U. Saha, K. Shahid, A. B. Hussain, and C. Shah-
naz, “Speech enhancement based on non-stationary noise-driven
geometric spectral subtraction and phase spectrum compensation,”
arXiv:1803.02870, 2018.

[27] L. Nanni, G. Maguolo, and M. Paci, “Data augmentation approaches for
improving animal audio classification,” Ecological Informatics, vol. 57,
p. 101084, 2020.

[28] W. Verhelst and M. Roelands, “An overlap-add technique based on
waveform similarity (wsola) for high quality time-scale modification
of speech,” in Proc. of IEEE ICASSP, vol. 2, 1993, pp. 554–557.
[29] H. Peng, F. Long, and C. Ding, “Feature selection based on mu-
tual information criteria of max-dependency, max-relevance, and min-
redundancy,” IEEE Transactions on Pattern Analysis and Machine
Intelligence, vol. 27, no. 8, pp. 1226–1238, 2005.

[30] About Touch ID advanced security technology., 2023, https://support.

apple.com/en-us/HT207537.

[31] NIST Biometric Image Software (NBIS), 2019, https://www.nist.gov/
services-resources/software/nist-biometric-image-software-nbis.
yamnet,
research/audioset/yamnet.

https://github.com/tensorflow/models/tree/master/

2020,

[32]

[33] K. He, X. Zhang, S. Ren, and J. Sun, “Deep residual learning for image

recognition,” in Proc. of IEEE CVPR, 2016, pp. 770–778.

[34] G. S. Bumbrah, R. M. Sharma, and O. P. Jasuja, “Emerging latent
fingerprint technologies: a review,” Research and Reports in Forensic
Medical Science, vol. 6, pp. 39–50, 2016.

[35] W. Wang, W. Gu, P. Liu, J. Liu, X. Wang, J. Liu, X. Yu, W. S.
Wong, and Y. Zhang, “Heterogeneously-wetting glass with enhanced
anti-fingerprint properties,” Chemical Engineering Journal, vol. 430, p.
132902, 2022.

vs.

[36] Matte
2019,
matte-vs-glossy-screen-protector.

Glossy

Screen

Cons,
https://saharacase.com/blogs/saharacase-tech-news/

Protector:

Pros

and

[37] Screen protector tempered glass vs matte, 2021, https://www.mietubl.

com/News/185.html.

[38] K. Cao and A. K. Jain, “Automated latent fingerprint recognition,” IEEE
Transactions on Pattern Analysis and Machine Intelligence, vol. 41,
no. 4, pp. 788–800, 2018.

[39] Z. Zhang, S. Liu, and M. Liu, “A multi-task fully deep convolutional
neural network for contactless fingerprint minutiae extraction,” Pattern
Recognition, vol. 120, p. 108189, 2021.

[40] K. Cao and A. K. Jain, “Hacking mobile phones using 2d printed
fingerprints,” Michigan State University, Tech. Rep. MSU-CSE-16-2,
2016.
[41] Fingerprint

cloning: Myth

https://blog.

reality?,

2020,

or

talosintelligence.com/fingerprint-research/.

[42] Y. Chen, Y. Yu, and L. Zhai, “Infinitygauntlet: Brute-force attack on
smartphone fingerprint authentication,” in Proc. of USENIX Security,
2023.

[43] D. Asonov and R. Agrawal, “Keyboard acoustic emanations,” in Proc.

of IEEE S&P, 2004, pp. 3–11.

[44] Y. Berger, A. Wool, and A. Yeredor, “Dictionary attacks using keyboard
acoustic emanations,” in Proc. of ACM CCS, 2006, pp. 245–254.
[45] L. Zhuang, F. Zhou, and J. D. Tygar, “Keyboard acoustic emanations
revisited,” ACM Transactions on Information and System Security,
vol. 13, no. 1, p. 3, 2009.

[46] T. Zhu, Q. Ma, S. Zhang, and Y. Liu, “Context-free attacks using
keyboard acoustic emanations,” in Proc. of ACM CCS, 2014, pp. 453–
464.

15

[47]

[48]

J. Wang, K. Zhao, X. Zhang, and C. Peng, “Ubiquitous keyboard
for small mobile devices: harnessing multipath fading for fine-grained
keystroke localization,” in Proc. of ACM MobiSys, 2014, pp. 14–27.
J. Liu, Y. Wang, G. Kar, Y. Chen, J. Yang, and M. Gruteser, “Snooping
keystrokes with mm-level audio ranging on a single phone,” in Proc.
of ACM MobiCom, 2015, pp. 142–154.

[49] D. Arp, E. Quiring, C. Wressnegger, and K. Rieck, “Privacy threats
through ultrasonic side channels on mobile devices,” in Proc. of IEEE
EuroS&P, 2017, pp. 35–47.

[50] M. Zhou, Q. Wang, J. Yang, Q. Li, F. Xiao, Z. Wang, and X. Chen,
“Patternlistener: Cracking android pattern lock using acoustic signals,”
in Proc. of ACM CCS, 2018, pp. 1775–1787.

[51] X. Liu, Y. Li, and R. H. Deng, “Ultrapin: Inferring pin entries via

ultrasound,” in Proc. of ACM AsiaCCS, 2021, pp. 944–957.

APPENDIX

TABLE VII: The parameters of our VGGish-based CNN
model.

Layer Type
Conv1 + BN +Relu1+ Pool1
Conv2 + BN +Relu2 + Pool2
Conv3 1 +Relu3
Conv3 2 +Relu4 + Pool3
Conv4 1 + Relu5
Conv4 2 + Relu6 + Pool4
Flatten Layer
Inner product (FC1+FC2+FC3)

Kernel Output Shape

64
128
256
256
256
256
/
/

48×32×64
24×16×128
24×16×256
12×8×256
12×8×512
16×4×512
12288
128

A. Implementation Details of VGGish-based CNN Encoder

Based on data extracted from 120 finger friction sound
segments collected from 65 users, we train our model with
a total of 10,800 data points. We utilize a computer with
the GPU of Tesla T4 and TensorFlow as the backend. The
network architecture and parameters are shown in Figure 16
and Table VII. The network optimization is performed using
mini-batch gradient descent (MBGD), with a fixed learning
rate of 0.001 and weight decay parameter of 0.0005. The batch
size is set to 128, and batch normalization and rectified linear
activation functions are applied after each convolutional layer.
The model is trained for 1000 iterations. Although the VGGish
model is trained on a limited dataset, experimental results
demonstrate that it could effectively transfer the discriminative
knowledge of audio to finger friction sound for classification
modeling. Additionally, we have considered the influence of
different environments and recorded the audio of users sliding
their mobile phone screens in a noisy office environment.

B. Datasets

The following datasets are used in our experiments, where
datasets 1-5 are produced from our data collection. Datasets 6
and 7 are typically open-source datasets as the baseline.

Dataset-1: We select Google Pixel 4 and iPhone 13 as the
devices for finger friction sound collection. Sixty-five users
(a total of 180 fingers) slide their fingers on each phone
screen 25 times. i) Dataset-1A: In this dataset, the friction
sounds of 180 fingers sliding on Pixel 4’s screen are collected.
After undergoing audio denoising, segmentation, and data

Fig. 16: The architecture of our wav2pattern model.

Algorithm 1: Random Restart Hill-Climbing.

input : sP M P =

(sP M P 1, sP M P 2, sP M P 3, sP M P 4, sP M P 5);
F p; outF p; jmax; S1, S2, S3, S4, S5 ;
T empS ;
template */
output: Top 5 Synthetic PMPs

/* Temporary storage

1 for i = 1 to 5 do
2

Calculate ASRmax and smpi;
j ← 0;
while j ≤ jmax do

3

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

j++;
a) Modify a random detail to T empS;
b) Add a randomly generated detail to
T empS;
c) Randomly replace details in existing detail

sets;

d) Randomly delete a detail from an existing

detail set;
Calculate tempasr = ASR(tempS);
if tempasr ≥ bestasri then

Si = tempS;
bestasri = tempasr;

end

end
if i == 5 then
Break;

end

18
19 end

augmentation, a total of 180 × 25 × 5 = 22, 500 finger friction
sound segments are obtained. ii) Dataset-1B: To increase the
robustness of the classification model, we also collect Dataset-
1B consisting of 22500 friction sound segments using iPhone
13. Due to microphone fingerprints, the sound collected by
different microphones may have slight differences. To address
this potential issue, we collect data using two different devices
in a quiet office, resulting in a total of 22, 500 × 2 = 45, 500
friction sound segments.

Dataset-2: To evaluate the impact of noise, we additionally
record the friction sound of 90 fingers on a Google Pixel 4 in
two different noise environments. The first environment is a

16

laboratory with mild noise, including the sounds of typing and
a printer, and the second environment 2 is a moderately noisy
corridor, including sounds of walking and light conversation.
These environments corresponded to dataset-2A and dataset-
2B, respectively. After data pre-processing, dataset-2 consists
of a total of 90 × 2 × 25 × 5 = 22, 500.

Dataset-3: To evaluate the generalization performance of
our model on friction sounds collected from different devices
and microphones, we collect a set of friction sounds using the
Samsung A20s. Specifically, we aimed to test the robustness
of our model to untrained device microphone fingerprints. 180
fingers were slid 25 times on the Samsung A20s, resulting
in dataset-3 of 4500 (180 × 25) friction sound segments. As
dataset-3 was only used for generalization testing, we did not
perform any data augmentation on these data points.

Dataset-4 PatternFinger: This is the complete fingerprint
dataset, where the data is collected by an optical sensor. There
are 2160 complete fingerprint images from 180 fingers with
12 complete fingerprint impressions per finger. Each complete
fingerprint image is 400 × 300 pixels in size and 500 dpi in
resolution. To evaluate the impact of fingerprint patterns on the
attack success rate, dataset-6 is further subdivided into dataset-
4A, dataset-4B, and dataset-4C, corresponding to whorl, left
loop, and right loop patterns, respectively. Each subset includes
720 (60 × 12) fingerprint images from 60 fingers.

Dataset-5 FingerPassDB7: In order to evaluate the impact
of fingerprint image integrity, we selected a subset of the
fingerPassDB7 fingerprint dataset. This is a natural partial fin-
gerprint dataset with data collected by the Authentic AES3400
capacitive sensor. There are 8640 partial fingerprints from 720
fingers. Each fingerprint image is 144×144 pixels and 500 dpi
in resolution. Dataset-5 can be further subdivided into dataset-
5A, dataset-5B, dataset-5C, and dataset-5D, corresponding to
whorl, left loop, right loop, and arch patterns, respectively.

Dataset-6 Livedet2011 ItalData: This is the complete fin-
gerprint dataset, where the data is collected by an optical
sensor. There are 2000 complete fingerprint images from 400
fingers with 5 complete fingerprints per finger, each complete
fingerprint image is 680 × 480 pixels in size and has the same
resolution of 500 dpi. Based on the whorl, left loop, right loop,
and arch patterns, dataset-6 can be divided into dataset-5A,
dataset-5B, dataset-5C, and dataset-5D.

Frequency2D conv layer BN layerPooling layerWide featuresDeep featuresPool4+FlattenFC layersDFTmRMRCepstralPrediction fusionOutputLeftloop or Rightloop or WhorlReLU
