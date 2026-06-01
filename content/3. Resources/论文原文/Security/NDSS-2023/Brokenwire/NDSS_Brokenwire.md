---
publish: true
---

BROKENWIRE : Wireless Disruption of CCS Electric Vehicle Charging

Sebastian K¨ohler†
University of Oxford

Richard Baker†
University of Oxford

Martin Strohmeier
Armasuisse S+T

Ivan Martinovic
University of Oxford

Abstract—We present a novel attack against the Combined
Charging System, one of the most widely used DC rapid charging
technologies for electric vehicles (EVs). Our attack, BROKEN-
WIRE, interrupts necessary control communication between the
vehicle and charger, causing charging sessions to abort. The
attack requires only temporary physical proximity and can be
conducted wirelessly from a distance, allowing individual vehicles
or entire ﬂeets to be disrupted stealthily and simultaneously. In
addition, it can be mounted with off-the-shelf radio hardware and
minimal technical knowledge. By exploiting CSMA/CA behavior,
only a very weak signal needs to be induced into the victim to
disrupt communication — exceeding the effectiveness of broad-
band noise jamming by three orders of magnitude. The exploited
behavior is a required part of the HomePlug Green PHY, DIN
70121 & ISO 15118 standards and all known implementations
exhibit it.

We ﬁrst study the attack in a controlled testbed and then
demonstrate it against eight vehicles and 20 chargers in real
deployments. We ﬁnd the attack to be successful in the real world,
at ranges up to 47 m, for a power budget of less than 1 W. We
further show that the attack can work between the ﬂoors of a
building (e.g., multi-story parking), through perimeter fences, and
from ‘drive-by’ attacks. We present a heuristic model to estimate
the number of vehicles that can be attacked simultaneously for
a given output power.

BROKENWIRE has immediate implications for a substantial
proportion of the around 12 million battery EVs on the roads
worldwide — and profound effects on the new wave of electriﬁ-
cation for vehicle ﬂeets, both for private enterprise and crucial
public services, as well as electric buses, trucks and small ships.
As such, we conducted a disclosure to the industry and discussed
a range of mitigation techniques that could be deployed to limit
the impact.

I.

INTRODUCTION

Replacing petrol and diesel vehicles with electric vehicles
(EVs) has been one of the main approaches to cut down
the global greenhouse-gas emission. As a result, many coun-
tries have committed to completely ban the sale of vehicles
with combustion engines within the next decade [30], [36],
[41]. The US government announced the transition of their
645,000 vehicles to a fully electric ﬂeet [19]. The National
Health Service in the UK plans to purchase fully electric
ambulances [29] and the Ministry of Defence announced
the introduction of fully electric battleﬁeld vehicles [71]. In
addition to governmental institutions, delivery companies are
moving towards EVs, too. Amazon started switching to electric

† Both authors contributed equally to this paper.

Network and Distributed System Security (NDSS) Symposium 2023
27 February - 3 March 2023, San Diego, CA, USA
ISBN 1-891562-83-5
https://dx.doi.org/10.14722/ndss.2023.23251
www.ndss-symposium.org

Fig. 1: Europe’s largest high-power charging hub with 26 CCS
charging stations that allow a total of 52 vehicles to be charged
concurrently [22].

delivery vans [1] and electric trucks [28]. The United States
Postal Service (USPS), Royal Mail, United Parcel Service
(UPS), and DPD announced the transition to fully electric
delivery vans [69], [23], [62], [20].

Nevertheless, one disadvantage of EVs is that they are
slower to refuel than fossil-fuel vehicles. Therefore, the suc-
cessful transition to all-electric vehicles requires a compre-
hensive and harmonized charging infrastructure that enables
a vehicle to be charged in a short time [67]. This is achieved
by increasing the charging capabilities of the charging stations,
also known as Electric Vehicle Supply Equipment (EVSE), and
building so-called charging hubs that enable the charging of
multiple cars simultaneously. Figure 1 shows such a charging
hub, the largest currently in operation in Europe.

Direct Current (DC) charging has become the de-facto
standard to enable rapid charging. For safety and efﬁciency
reasons, the high-power DC charging stations rely on commu-
nication with the vehicle to exchange vital information, such
as maximum voltage, required current, and the State of Charge
(SoC). This means the availability of the communication link
is crucial for the charging session and any disruption of the
communication, intentional or unintentional, will result in the
charging process being aborted for safety reasons [40].

The fastest-growing DC charging standard, already dom-
inant across North America and Europe, is the Combined
Charging System (CCS) [37], [26]. CCS provides a high-
bandwidth IP link via power-line communication (PLC) for the
communication between the EV and EVSE. This brings bene-
ﬁts in terms of reusing commodity technologies and affording
capacity for additional services, such as automatic billing and
demand-response charging, in addition to the crucial charging
session control [38].

However, it is known that PLC, as used in CCS charging,
unintentionally leaks communication signals via the charging

(a) Heavy-duty truck using CCS [14].

(b) Bus depot with 46 CCS plugs [64].

(c) Ferry utilizing 28 CCS feeds [42].

Fig. 2: Examples of current electric vehicles now following the Combined Charging System standard and implementing the
high-level communication using PLC, which makes them vulnerable to the BROKENWIRE attack.

cable [4] and it has been shown in other settings that PLC is
susceptible to electromagnetic interference [49].

In this paper, we present BROKENWIRE, an attack that
exploits the combination of the susceptibility of PLC to
the use of
intentional electromagnetic interference (IEMI),
unshielded charging cables, and the application of a colli-
sion avoidance mechanism in the low-level communication
protocol. We demonstrate that the charging communication
required by CCS can be disrupted wirelessly — causing the
charging session to abort and leaving the vehicle and the
EVSE in an error state, from which it will not automatically
recover, even if the attack ceases. We show that only temporary
physical proximity is required and that
to
execute the attack for only around two seconds — just long
enough to cause a timeout and for the charging to abort.
Therefore, the attack can reach beyond physical barriers and
enables wardriving, which makes it stealthy, easily-deployed
and challenging to detect.

is sufﬁcient

it

Since CCS is becoming increasingly popular as a charging
standard for a wide range of EVs — beyond solely passenger
vehicles — the BROKENWIRE attack has immediate impli-
cations to a variety of other applications as well, such as
emergency vehicles [29], buses [64], heavy-duty trucks [15],
aircraft pushback tractors [25], private boats [2], public fer-
ries [52] and even airplanes [27]. A handful of examples of
vehicles that use CCS and are vulnerable to the BROKENWIRE
attack are shown in Figure 2. These illustrate both the range
of affected vehicles and also the density with which chargers
are arranged for ﬂeet purposes and therefore their vulnerability
to a ranged, wireless attack. Moreover, CCS is also poised to
play a decisive role in the future of the power grid [10]. As the
energy generated from renewable resources increases, the need
for electricity storage has become more important than ever.
Bi-directional charging in combination with Vehicle-to-Grid
(V2G) communication will enable vehicles to act as energy
storage to buffer excess energy and feed it back into the grid
to meet peak demand [55]. First trials of this approach have re-
cently started in different countries, for example, Germany [8],
Switzerland [32] and the UK [7]. As such, EVs with CCS will
also soon play a signiﬁcant role in the stability of the power
grid, intertwining them even further into critical infrastructure.
Our work highlights a severe design ﬂaw in the use of PLC
for charging communication that leaves millions of vehicles,
some of which are in constant use within critical infrastructure
sectors, vulnerable.

2

Contributions

•

•

•

•

We identify a vulnerability in the most widely adopted
DC rapid charging standard in Europe and North
America leaving millions of EVs vulnerable.

We demonstrate the BROKENWIRE attack in an ex-
tensive evaluation, with both controlled laboratory and
real-world environments.

We analyze the effects of the BROKENWIRE attack
and its real-world implications.

We propose different countermeasures,
including a
cheap and easy-to-deploy hardware approach that
makes the attack orders of magnitude harder to con-
duct.

II. BACKGROUND

This section describes the underlying technical concepts
and terminologies of today’s charging standards, which help
better understand the BROKENWIRE attack.

A. Electric Vehicle Charging

Charging an electric vehicle can be done with either
Alternating Current (AC) or Direct Current (DC). While for
AC charging the car has to be equipped with a rectiﬁer
that converts the alternating current to direct current, for DC
charging this process is outsourced to the charging station.
To reduce the additional weight from the rectiﬁer, its capacity
is limited, which caps the maximum charging power for AC
charging to around 22 kW. In contrast, DC charging enables
high-power charging, often referred to as rapid charging, with
up to 350 kW. Therefore, for recharging a vehicle in a short
time frame, DC chargers are the ﬁrst choice.

In addition to the Combined Charging System, three com-
peting DC charging standards currently exist — CHAdeMO,
Tesla’s supercharger, and GB/T 20234. While Tesla’s super-
charger is a proprietary technology, CHAdeMO was developed
by car manufacturers from Japan and is an alternative to CCS
mainly in the Asian market [16]. Similarly, GB/T 20234 was
designed for the Chinese market. The main difference between
these standards and CCS is the use of CAN for the charging
communication rather than PLC. However, more and more
non-European car manufacturers previously using CHAdeMO,
such as Kia, Nissan, and Hyundai, moved to CCS for markets

standard requires the CCS charging session to halt immediately
and cease power transfer, upon which the EV and EVSE
choose whether or not to attempt to start a new session from
the beginning (or, in limited cases, continue charging using the
basic pulse–width modulation protocol) [39], [40]2.

In addition to the main charging control loop, CCS also
provides a range of additional capabilities using the IP link.
One feature, known as ‘Plug & Charge’, enables automatic
billing without the user authorizing a transaction manually [38]
— and is currently appearing in the newest EVs and EVSE
installations. Trials of ‘Vehicle-to-Grid’ capabilities, in which
vehicles can change their charging proﬁle in response to power
availability and even provide bidirectional power transfer, are
at an advanced stage and these capabilities are expected to be
publicly deployed shortly [72].

The Combined Charging System was initially intended only
for light vehicles [38]. However, the lack of alternatives that
provide additional beneﬁts, such as higher charging capacity,
led to the use in some much larger vehicles [52], [9] Hence,
derivative standards that are better suited to these uses have
since been developed and are expected to be adopted in the
future [11]. The SAE J3105 standard describes systems for
charging electric buses using a range of connector options un-
derpinned by ISO 15118 [33]. Meanwhile, the new Megawatt
Charging System (MCS) standard has been recently completed
by the same standardization body as CCS, speciﬁcally de-
signed to accommodate the needs of heavy vehicles, such as
freight trucks [11]. At the time of writing, trials have been
run with 3.75 MW of charging power [11]. We describe in
Section VII how our work is relevant for these standards, too.

C. HomePlug Green PHY

The high-bandwidth IP link used for communication in
ISO 15118 is provided by the HomePlug Green PHY (HPGP)
power-line communication technology [40]. Power-line com-
munication is a technology in which communication sig-
nals share cabling principally used for power transfer; with
low-power high-frequency communication signals superposed
alongside high-power direct- or alternating-current energy
transmission [49]. The HomePlug family of standards describe
implementations of PLC intended for LAN communication,
with HPGP as the derivation intended for embedded, industrial
use-cases. The HPGP standard is publicly available at no
cost [31], and is discussed in detail in [4] and [5].

In brief, HPGP uses an Orthogonal Frequency-Division
Multiplexing (OFDM) physical-layer operating in the 2 – 28
MHz band and modulating data onto (typically) 917 unmasked
subcarriers spaced 24.414 kHz apart. The technology provides
a set of communication modes that
trade throughput vs.
reliability with maximum speeds of 3.77, 4.92 (default), and
9.84 Mbps. The network is random access, with a Carrier-
Sense Multiple Access scheme, with Collision Avoidance
(CSMA/CA) to control medium access both within a network
and when co-existing with other nearby HPGP networks.
Although vehicle charging involves only two nodes in the
HPGP network (EV and EVSE), the random-access nature of
the technology still allows a situation in which both nodes

(a) CCS Combo 1 (US).

(b) CCS Combo 2 (EU).

Fig. 3: The two different plug layouts used by CCS in North
America (Combo 1) and Europe (Combo 2) respectively.

outside of Asia [46], [13], [54]. Moreover, Tesla started using
CCS with the introduction of the European version of their
Model 3 in 2018 [47] and recently announced an updated
version of their Model S and Model X for the European
market, which will be equipped with a CCS socket by default
and no longer require an adapter [48]. The recent developments
suggest that CCS will play a crucial role as a globally adapted
standard in the future. As such, our work focuses on CCS
only and will discuss the relevant details about its underlying
technical details in the subsequent section.

B. Combined Charging System

The charging technology standardized as the Combined
Charging System — the name presented to a vehicle user — is,
in fact, a collection of multiple technical standards, assembled
by the Charging Interface Initiative e.V. (CharIN e.V.), a
non-proﬁt association founded by a consortium of different
stakeholders, such as automobile manufacturers, component
suppliers and charging station vendors. In this paper, we focus
on communication between the EV and the EVSE, which
is governed by the ISO 15118 series of standards1 [38].
Depending on the geographical region, CCS uses different plug
types, which are illustrated in Figure 3. In North America
Combo 1 and in Europe Combo 2 is used. While the con-
nector arrangement differs slightly between the two types, the
underlying technology is the same. As the name suggests, PLC
usually uses the main power lines as a transmission medium.
However, in the case of CCS, the two separate wires, the
Control Pilot (CP) and Protective Earth (PE) are used. The
actual direct current is delivered by the two large conductors
at the bottom of the plug.

For backward compatibility, a basic communication
scheme using a pulse-width modulation (PWM) protocol de-
ﬁned in the IEC 61851 standard [34] is used when initializing
the charging process. Upon successful initialization of this
communication, a high-bandwidth IP link is established and
control is handed to it. The vehicle and charger then engage
in an application-layer protocol
to control charging. This
communication must be maintained throughout the charging
session for a variety of reasons, the most important of which
is fault detection. If the communication is lost, the ISO 15118

1Previously by a simpliﬁed subset in DIN 70121, which provides limited

capabilities but is functionally identical for underlying communication.

2This depends on various factors including the payment method used for
the charging. We observed no instances of this fallback occurring in practice.

3

transmit simultaneously. Furthermore, cross-talk from nearby
charging stations could cause interference [4]. To ensure that
the communication is not affected by collisions, CSMA/CA is
required.

The work in [4] also describes a notable challenge with
HPGP PLC:
its propensity to leak communication signals
from the charging cable. It has also been noted in [49]
that the related HomePlug AV technologies are vulnerable to
interference emitted in the same frequency range by broadcast-
ing and wireless communication systems. This vulnerability
was carried across to the HPGP technology and presents a
risk of cross-talk between different vehicle-charger pairs in
adjacent parking bays. It is possible that a vehicle would begin
communicating, not with the charger that it is physically con-
nected to, but with a nearby charger that happened to respond
ﬁrst, but is actually communicating via leaked signals. The
HPGP speciﬁcation implements the Signal Level Attenuation
Characterisation (SLAC) protocol to mitigate this, in which the
vehicle and charger exchange sounding messages to determine
which are connected directly (thereby experiencing the least
attenuation of their sounding messages) and which are being
inadvertently overheard due to cross-talk [31].

III. THREAT MODEL

A. Goals

The overarching goal of our considered attacker is the
disruption of charging sessions for one or more EVs. We group
possible intentions into three categories:

Single Vehicle: In this case, a speciﬁc vehicle is targeted. This
may be done as an attack on the owner, to make it difﬁcult
for them to travel, either at home or a remote location, thus
exposing them to inconvenience or making them vulnerable
to further physical attack. Alternatively, it may be intended as
an attack on the vehicle; immobilizing it at a remote location,
from which it could be stolen if the driver leaves to obtain
assistance. Even in less sinister scenarios, disrupting the charge
sessions of others would allow an attacker to take the newly-
unlocked charge cable for use in charging their vehicle, or
achieve faster charging by preventing other cars from sharing
a load-balanced electricity supply.

Fleet Denial: In this case, a speciﬁc organization is targeted
and their vehicles immobilized en masse. This may be a deliv-
ery or transport business, in order to cause ﬁnancial loss, harm
supply chains or blackmail the operator for monetarization.
Alternatively, the organization may represent a public service,
such as buses, taxis, a police force, or ambulances, with the
attack impacting local citizens.

Widespread Disruption: In this case, as many vehicles as
possible are attacked, without regard to who owns or operates
them. There may be an alternative attack rationale, such as
harming the business of a charging service provider or inﬂu-
encing the local power grid through manipulation of the high
power consumption of EVs and their proposed future use as bi-
directional storage batteries. Given the high capacity of today’s
charging parks, which is expected to increase substantially in
the next few years, the attack is an easy and effective way to
control multiple megawatts of load. It has been shown that ma-
liciously controlled high-power devices can cause instabilities

Fig. 4: Attack illustration. The injected preamble is not distin-
guishable from background noise. Only applying a correlation
function on the captured data revealed the preamble position.
Nevertheless, it caused a packet loss of around 80%.

to the grid and are of great concern [65], [63]. Alternatively,
the motivation may simply be spite — blocking or vandalizing
vehicle chargers due to a strong dislike of EV technologies has
been documented in countries worldwide.

Overall, we consider an attacker who seeks to conduct their
disruption with stealth, speed, and scalability. The simplest
ways to interrupt charging are to operate an emergency cutout
switch or to damage the vehicle, cable, or charger. However,
such approaches require direct access, which is risky for per-
petrators and mitigated with physical barriers or surveillance.
Furthermore,
the same process must be repeated for each
targeted vehicle or charger, with commensurate repeated risks.
In contrast, BROKENWIRE enables an adversary to disrupt
charging sessions simultaneously at scale and from a safe, rea-
sonable distance without interacting directly with the target(s).
The attack can be performed from beyond the line of sight of
potential CCTV cameras, concealing the attacker’s presence.
In addition, physically-secured charging parks located behind a
fence or wall are vulnerable. The adversary can either execute
the attack in person from a nearby location or deploy a device
at the target site and control it remotely.

B. Capabilities

Since the entry barrier for carrying out the attack is low,
our threat model considers a malicious actor with only access
to off-the-shelf hardware that can easily be purchased online.
At a minimum, this constitutes a software-deﬁned radio and
an antenna, with additional power ampliﬁers potentially being
used to increase transmission power and, hence, range. We
consider an attacker who can generate or capture the required
attack signal — which they may subsequently distribute to oth-
ers to further reduce the barrier to entry. Thus, even someone
with little to no background in digital signal processing could
retrieve an attack signal online and conduct the attack.

IV. THE BROKENWIRE ATTACK

In this section, we give an overview of the technical details
of the BROKENWIRE attack. The attack involves repeatedly
triggering the CSMA/CA mechanism that is required to be
present in any standard-compliant implementation, such that
neither vehicle (EV) nor charger (EVSE) ever have an op-
portunity to transmit. While this action alone could prevent
communication indeﬁnitely, it only needs to be applied for

4

a few seconds in order to trigger a timeout in the higher
layers of the communication protocol [39], [40]. At this point
the entire charging process is aborted and the attacker can
stop broadcasting, making it only necessary to have temporary
physical proximity to the victim.

Exploiting CSMA/CA: As mentioned in Section II,
the
Combined Charging System uses HomePlug Green PHY PLC
for communication during the charging session. The public
HPGP standard deﬁnes Carrier-Sense Multiple Access with
Collision Avoidance (CSMA/CA) as a channel access method.
If a node wishes to transmit a message, it will check for other
nodes already transmitting. In case an ongoing transmission
is detected, the node will wait for a short, random period of
time before attempting to transmit again [31]. This process
is repeated indeﬁnitely, until the transmission medium is idle,
and the message can be transmitted.

The BROKENWIRE attack exploits this channel access
mechanism to force the PLC modems at both nodes to back off
and stop communicating. The attacker continuously transmits
a recognizable signal, in this case a preamble, convincing any
listening nodes that the channel is busy. The transmission is
repeated indeﬁnitely, such that both modems continue to wait
and cannot transfer any data.

Per the ISO 15118 standard, the communication session
between the EV and EVSE is terminated if a short message
timeout, the average in the standard being 1.5 seconds, is
exceeded [39], [40]. Once communication is lost, the charging
session must cease immediately and a new session has to be
initiated manually [40]. This means that even after the attack
ends, and the nodes can continue to communicate, the charging
process will not automatically restart.

Generating an Attack Signal: In HPGP, the preamble is
used to mark the beginning of a frame, to synchronize the
receiver’s clock with the transmitter and to permit channel state
estimation. All frames begin with a standard HPGP preamble,
which is sufﬁcient to trigger the CSMA/CA mechanism in
nodes that receive it. As such, BROKENWIRE uses a preamble
waveform as an attack signal.

The preamble is deﬁned in the standard as a concatenation

of repeated preamble symbols, each generated as follows:

S[t] =

20

10 3
√384 ·

(cid:88)

c∈C

(cid:18) 2

cos

π
c
·
384

·

t

·

(cid:19)

+ ψ(c)

where t is the preamble sample time step (for 0

≤
384
1), C is the set of unmasked subcarriers, c is the
subcarrier index and ψ is a function mapping subcarriers to
speciﬁc phase offsets deﬁned in the standard [31].

≤

−

t

The concatenated symbols are then post-processed to ﬁt
one of two preamble formats, depending on compatibility
mode. These modes differ only in the number of symbols
used. In either case, the trailing symbols are inverted to assist
with time synchronization and the entire preamble windowed
to limit spectral leakage. The repeating structure, inversion of
trailing symbols and windowing are visible in Figure 4.

As the preamble format is entirely documented, an adver-
sary can generate a preamble from only public information.

5

Fig. 5: An overview of the experimental setup used to evaluate
the attack under controlled lab conditions.

However, even this is not strictly required. Our empirical tests
found that it was also highly effective to simply capture a
small snippet of the communication between an EV and EVSE,
extract the preamble and replay it. As such, the attack can be
executed with little to no digital signal processing knowledge.

Injecting the Attack Signal: The attack signal can be
trivially injected with physical access. However, as shown
by [4], the charging cable acts as an unintentional antenna
that leads to electromagnetic emanation. At the same time,
this phenomenon makes the charging cable susceptible to
electromagnetic interference. Since the cable is unshielded,
electromagnetic waves can easily couple onto the wires within
it. While the PLC uses differential signaling over two wires,
any asymmetries in the two pathways lead to some signal
still being retained. By transmitting the attack signal over-the-
air, an attacker can therefore cause sufﬁcient coupling on the
charging cable of a victim EVSE for it to correctly detect the
injected preambles.

This makes the use of a CSMA/CA attack particularly
powerful. While devices are actively designed to resist noise,
they are designed to detect faint preambles efﬁciently. We
found that broadcasting a preamble over-the-air, in the same
frequency band in which PLC is operating, couples well onto
the cable and is picked up by the PLC modems. In accordance
with the standard, if the signal-to-noise (SNR) ratio of the
2 dB, the preamble is correctly recognized and
preamble is
interpreted as the start of a packet which indicates an ongoing
communication [31].

≥

V. LAB EVALUATION

In this section, we present the results of our experiments
with the attack in a controlled laboratory environment. Later,
in Section VI, we describe real-world testing.

A. Experimental Setup

We evaluated the attack under controlled laboratory con-
ditions, using a testbed composed of two PLC HomePlug
Green PHY evaluation boards from Devolo (dLAN Green

PHY eval board II) [18]. The boards were equipped with
the same Qualcomm QCA7000 chips, as they are used for
communication in EVs and DC high-power charging sta-
tions [76]. The two HPGP evaluation boards were connected
via a 4 m long copper wire with two ﬂex cores (2
×
1.5 mm2), one for the Control Pilot and one for Protective
Earth, the same cable structure found in CCS charging cables.
The cable length corresponded roughly to the same length
found at most DC rapid chargers. Each PLC modem was
connected to a RaspberryPi via Ethernet, which mimicked
the Electric Vehicle Communication Controller (EVCC) and
Supply Equipment Communication Controller (SECC). The
communication controllers are responsible for the higher-level
communication (HLC). In our case, they ran IPerf, as described
in the subsequent section. As such, any device that can handle
the HLC would be suitable for this task.

On the attacker side, we used a LimeSDR as the transmitter
connected to a 1 W ampliﬁer (Mini-Circuits ZX60-100VH+).
We used GNURadio to operate the LimeSDR and emit the
malicious attack signal. Our antenna was a low-cost solution
composed of a balun (1:9) with a dipole made from two simple
24AWG copper wires. Our antenna was electrically-short (7 m)
for the frequency band, so we expect the attack to be even
more effective with a dipole optimized for maximum gain ( 5
4 λ,
where λ is the wavelength) [43]. The entire experimental setup
is depicted in Figure 5.

B. Method

We evaluated the effectiveness of the attack on network
throughput using IPerf, an open-source software tool for per-
formance testing. We set up a UDP communication between
the two RaspberryPis, whereby the IPerf server was running
on the EVCC and the IPerf client was executed on the SECC.
Under normal operation, we observed no packet losses and a
maximum transmission rate of 833 packets per second for a
throughput of 5 Mbps. We considered the communication link
as ofﬂine and the attack as successful, once IPerf reported an
unsuccessful establishment of a connection or a packet loss of
100%.

In order to determine the minimum required transmission
power for a successful attack for a given distance d, we iterated
over the gain settings of the LimeSDR (14 – 64) with a step
size of 2. For each gain setting, we broadcast a preamble for 20
seconds and measured the packet loss during this period. The
center frequency of the LimeSDR was set to 17 MHz and the
sample rate to 25 MSPS. While the sample rate of 25 MSPS
was not enough to cover the entire spectrum used by HPGP,
it was sufﬁcient for our attack.

To emphasize the simplicity and effectiveness of our ap-
proach, we replayed a captured preamble rather than using
one generated from scratch. We argue that this yields conser-
vative results as the captured preamble will have experienced
additional distortion and should therefore be less likely to be
recognized by the PLC modems. Moreover, it highlights the
fact that the adversary does not have to know how to generate
the preamble making the attack even easier to execute.

We want to note that although the experiments were as con-
trolled as possible, some parameters were out of our control.
For example, the noise that coupled onto the PLC lines from

other devices in the building varied widely. To compensate for
these variations, we conducted multiple runs and calculated
the mean power required for the attack to work. Finally, we
repeated the experiment for different distances d between the
charging cable and the antenna. To ensure reproducibility, we
automated our testing by using a Python script that iterates
over the different output powers3.

Measuring Output Power: The output power of the LimeSDR
was not calibrated and the gain was only given as a unitless
number. In order to report accurate values for the required
transmission power, we measured the actual output power of
the LimeSDR for each gain setting. We directly connected the
LimeSDR to an RF power meter and transmitted the preamble
used in our evaluation.

C. Results

The efﬁciency of the BROKENWIRE attack becomes ap-
parent when examining the results presented in Figure 6.
The graphs show the minimum required transmission power
for various distances to cause a packet
loss of 100% or
IPerf to throw a “No route to host.” exception. As previously
mentioned, the signal-to-noise ratio of the injected preamble
2 dB to be correctly interpreted as the
only needs to be
start of a packet. Unanimously with this requirement, it is
less surprising to ﬁnd that even an extremely low output
power is sufﬁcient to induce a strong enough preamble into
the wire to interfere and disrupt communications. Although
the required transmission power increased substantially with
greater distance, 10 mW was still sufﬁcient to disrupt the
communication of the testbed from 10 m away.

≥

D. Effectiveness across Multiple Stories

The advantage of using electromagnetic emanation as an
attack vector is that no physical access as well as line-of-sight
to the target is necessary. To demonstrate the capabilities of our
attack, we conducted it in a limestone building with thick walls
and ﬂoors, and a ceiling height of about 3.5 m. We positioned
the attacker on the ground ﬂoor of the building and placed the
charging testbed one story above. The components were not
directly aligned above each other, but slightly offset, resulting
in a distance of approximately 6 m between the antenna and the
PLC modems, including a ceiling approximately 20 cm thick.
In line with the other lab experiments, we ran a UDP IPerf
session between the RaspberryPis that were connected via the
PLC modems. We then increased the gain of the LimeSDR
with a step size of two until IPerf reported a packet loss
of 100% or the unsuccessful establishment of a connection.
We repeated the experiment multiple times and averaged the
required output power. We found that an output power of
around 100 mW was sufﬁcient to disrupt the communication.
While precise ranges and power budgets will vary between
environments, we believe that this result amply demonstrates
that the attack can be conducted beyond a physical barrier and,
given the nature of the test building, that it approximates even
a multi-story parking lot.

3The evaluation source code is available at: https://github.com/ssloxford/

brokenwire.

6

(a) Results in dBm.

(b) Results in mW.

Fig. 6: Results of the distance experiments in lab conditions. Minimum power required to cause a packet loss of 100%.

E. Comparison to Broadband Noise Jamming

HomePlug Green PHY was designed to withstand noisy
environments [31]. Using the ROBO mode, which transmits
the data redundantly on multiple subcarriers, and the usage
of QPSK as a modulation scheme, make it robust even under
severe communication channel conditions. To emphasize the
efﬁciency of our attack and demonstrate the robustness of
HPGP against noise, we compared the required power of
the preamble injection to the signal required for successful
broadband noise jamming. As the name suggests, noise is
emitted simultaneously in the entire spectrum in which PLC is
operating. We used the same experimental setup and method
as described in Section V-A with a ﬁxed distance between
the target and the attacker of 1 m. However,
instead of
broadcasting a preamble, we just emitted Gaussian White
Noise. With the maximum output power of the Mini-Circuits
ZX60-100VH+ (1 W), we could not observe any degradation
of the connection quality whatsoever. Hence, we replaced the
1 W ampliﬁer with a Kalmus 161C ampliﬁer (max. 100 W)
and repeated the experiment for transmission powers up to
20 W. We still observed no effect on the connection.

The results are in line with our expectations and validate
our hypothesis in Section IV that a preamble injection attack
substantially outperforms noise as a disruption technique. Not
even a signal with three orders of magnitude more power was
enough to disrupt the communication.

F. Predicting Attack Range

A range of mathematical methods exist for estimating
received power over a wireless link. However, the majority of
methods employ simplifying assumptions that may not always
hold valid. The Friis path-loss equation is often used to esti-
mate expected power levels in wireless security contexts [24].
However, the equation is only well-deﬁned for signals in the
far-ﬁeld of transmission and is frequency dependent. At the
comparatively low frequencies used by HPGP PLC signals,
the far-ﬁeld begins at least 10.7 m away from the transmitter
and the difference in predicted power at a receiver changes
substantially when considering the lowest or highest parts of
the signal bandwidth.

Given these concerns, we investigated whether our ob-
served results bore any similarity to those predicted by the Friis
equation. To do so, we measured the ambient noise level on the

Fig. 7: Comparison of observed results in our testbed against
2 dB greater
predictions from the Friis equation, for signal
than noise of 2.45 mV. Shaded region represents the range of
predictions across the 2 – 28 MHz frequency range. Dashed
vertical line represents transition between near-ﬁeld regions.
The far-ﬁeld region is beyond the range of the plot.

≥

charging cable in our testbed, ﬁnding it to be 2.45 mV. From
this, we computed the minimum preamble detection threshold
2 dB compared to noise). We
stated in the HPGP standard (
≥
then applied a rearranged Friis equation to determine the trans-
mission power required to achieve this received power level.
Full details of the calculations are provided in Appendix A.

Figure 7 shows a comparison of the results observed in
our testbed with predictions made using the Friis equation.
We noted that, while the Friis estimates provide a wide range
of powers, depending on the selected frequency, the estimates
made using the upper end of the band were close to our
observed results. This is consistent with the observations in [4],
showing that the higher part of the band appeared most prone
to radiating from the charging cable (and thus, we expect most
prone to ingress as well). The predicted values were furthest
from the observed ones in the reactive near-ﬁeld region, but
became closer in the radiative near-ﬁeld. We tentatively suggest
that the predictions would continue to be accurate beyond our
furthest measured distance.

VI. REAL-WORLD TESTING

Following our experiments in a controlled lab environment,
we subsequently examined the effects of the BROKENWIRE

7

1.02.03.05.07.510.0Distanced(m)−50510OutputPowerPout(dBm)1.02.03.05.07.510.0Distanced(m)0.02.55.07.510.0OutputPowerPout(mW)1.02.03.05.07.510.0Distanced(m)−30−20−10010OutputPowerPout(dBm)ReactiveRadiativeFriis(17MHz)Observed(a) Scenario 1.

(b) Scenario 2.

(c) Scenario 3.

(d) Scenario 4.

(e) Scenario 5.

Fig. 8: Example scenarios that we tested during our real-world evaluation. In all of the scenarios, the victim is represented by
the car(s) connected to the charging station and with the battery symbol. The vehicle that is not charging is the adversary.

A. Method

We evaluated the BROKENWIRE attack in various scenarios
that replicate how it could be carried out in a real-world
attack. Figure 8 illustrates ﬁve of the different scenarios that
we examined.

Scenario 1: In this scenario, we simulated a drive-by attack,
in which the adversary aims to disrupt the charging session of
vehicles that are parked close to a public road. To achieve
the maximum possible coverage, we mounted the antenna
onto the roof of the attacking car on the site closest to the
target vehicle. We then drove slowly (approx. 10 mph/15 kph)
past the charging vehicle. A drive-by attack could be used by
an adversary to fulﬁl the goal of ﬂeet denial or widespread
disruption as described in Section III. Since only temporary
physical proximity is required, so-called wardriving would be
possible. This makes the attack stealthy and challenging to
detect.

Scenario 2: This scenario mimicked a situation that can often
be found on public car parks, for example, at supermarkets.
We parked the attacking car, with the antenna coiled in the
trunk, in a parking bay on the opposite side of the victim.

Scenario 3: Similar to Scenario 2, we emulated a public car
park. However, this time, we parked the attacking vehicle
parallel to the victim. In line with the previous scenario, we
kept the antenna coiled in the trunk.

Scenario 4: Scenario 4 had the same settings as Scenario 3.
However, we parked another vehicle between the attacker and
the victim. This enabled us to evaluate the attack for a setting
where there was no direct line-of-sight between the attacker
and the victim. At the same time, it allowed us to test the
attack on multiple cars at once. This scenario, along with the
previous two, particularly reﬂects an attacker seeking to access
an in-use charging cable or avoid sharing the available current
with others.

Scenario 5: In Scenario 5, we simulated an attack from a
distance to hide the physical presence of the adversary. The
victim was charging close to a large intersection and the
adversary was located on the opposite side of the intersection.
This setting can, for example, be used to target an EV ﬂeet at
a commercial site/depot with CCTV surveillance.

Fig. 9: The equipment we used during the real-world experi-
ments. The hardware, including the antennas, was installed in
the back of the trunk and powered via a UPS.

TABLE I: Overview of the eight tested vehicles.

Class
Subcompact
Compact SUV
Shooting Brake
Subcompact

Vehicle
Vehicle A
Vehicle B
Vehicle C
Vehicle D
Vehicle E Mid-size Sedan
Vehicle F Mid-size SUV
Vehicle G
Vehicle H

Compact
Compact

Price ($)
50,000
85,000
150,000
20,000
50,000
70,000
45,000
32,000

Charging Capacity
50 kW
150 kW
270 kW
50 kW
120 kW
150 kW
125 kW
50 kW

attack on real charging sessions. We tested the attack on a
broad range of passenger cars from different manufacturers,
spanning across different classes, price ranges, and charging
capacities. All eight cars were equipped with a CCS charging
port, either Combo 1 (US) or Combo 2 (EU), and followed
the ISO 15118 or the DIN 70121 standard. Table I gives an
overview of the cars we tested. In addition, we evaluated a
total of 20 DC charging stations from various providers and
manufacturers. We did not exhaustively test every combination
of vehicles and charging stations, but tested multiple vehicles
with most chargers. In one case, we successfully tested with
three vehicles charging simultaneously on identical charging
stations.

8

EV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYPEV ONLYFig. 10: Distance measurement for Scenario 5 via zoom.earth.
The target charges at the green parking bay, while the attacker
is located on the other side of the intersection.

The real-world evaluation was performed with the same
equipment used for the laboratory experiments. However, the
ampliﬁer was powered from an Uninterruptible Power Supply
(UPS) and a bench power source. The setup used is depicted in
Figure 9. This research prototype cost less than $1,000 and is
easily contained in the vehicle. However, we later achieved a
smaller, lighter setup by exchanging the UPS and bench power
source with a small USB power bank and a step-up converter.
The capacity of the battery (10 Ah) was sufﬁcient to run the
attacking hardware for more than 10 hours and could easily be
carried in a rucksack or mounted on a drone. We are conﬁdent
that further optimizations could be applied for size and cost
that would allow an adversary to plant the device in a remote
location and continuously disrupt an entire area. Depending on
the tested scenario, the antenna was either placed within the
attacking vehicle, mounted on the bodywork, or positioned on
the ground immediately next to it.

B. Observations

The BROKENWIRE attack was successful

in disrupting
charging in every observed case — for any combination of
EV and EVSE we tested, and independently of the CCS plug
used (Combo 1 and Combo 2). Substantial variations were ob-
served in effective range and power requirements in each case.
However, once a suitable power level had been selected for the
environment, charging sessions were terminated immediately.

Required Output Power: In contrast to the lab experiments,
the conditions during the real-world experiments were largely
uncontrolled, which led to less predictable results. In particular,
we noticed that in some cases, a much higher transmission
power was required to successfully terminate the charging
session. For example, while we successfully disrupted the
testbed from 10 m away using less than 10 mW of transmission
power (see Figure 6b), disrupting Vehicles B and C from a
distance of 10 m in a setting similar to Scenario 2 required
around 600 mW. In contrast, at another charging site, we
stopped vehicle D from charging using only the LimeSDR
without the ampliﬁer from 1.5 m away, which according to
our measurements, corresponded to 0.3 mW of output power.
At another EVSE, we replicated Scenario 5 and successfully
disrupted an ongoing charging session of Vehicle D from 47 m
away with just under 700 mW. The exact arrangement of
this scenario is depicted in Figure 10. We believe that the

9

Fig. 11: The error message displayed on a tested charging
station shortly after starting the broadcast of the preamble.

large variations in the power requirements are due to several
parameters that cannot be controlled by the adversary, such as
electromagnetic interference from other devices, differences
in signal propagation and noise generated by the charger or
the vehicle itself. Nevertheless,
the results from our real-
world evaluation are largely in agreement with the results and
observations from our laboratory experiments, showing that
the attack is effective and easy to execute.

Disrupting Multiple Vehicles: Since large charging hubs,
where multiple vehicles can be charged simultaneously, are
becoming more and more common, we also evaluated the
effects of the BROKENWIRE attack in such a setting. The
arrangement of our test was similar to Scenario 4. However,
we tried to interrupt the charging sessions of three cars at once.
We connected the vehicles to two separate charging stations
and successfully disrupted all charging sessions at about the
same time.

To further demonstrate the scalability of the attack and
examine how the signal propagates when the line-of-sight is
occluded by other vehicles, we conducted the attack again in a
large car park with three rows of cars between the attacker and
the victim. While the vehicles that occluded the line-of-sight
were parked on normal parking bays without chargers and only
the victim cars were charged and disrupted, the layout of the
car park mimicked a real, multi-unit charging hub, illustrating
how the attack would affect other vehicles in the vicinity. In
this speciﬁc scenario, a total of 20 vehicles were parked in the
range of the attack signal. Therefore, we are positive that any
EV within that range would have been affected by the attack.
The results also prove that the attack works against occluded
vehicles.

Antenna Alignment: We noticed that the alignment of the
antenna is crucial for the effectiveness of the attack. The
attack worked reliably and within moderate distance when
the antenna was coiled in the closed trunk, as shown in
Figure 9. However, for the same output power, the maximum
possible attack range increased signiﬁcantly when the antenna
the signal
was stretched out. Moreover, we observed that
is attenuated when the trunk is closed. Moving the antenna
outside increased the attack range even further. Nevertheless,
our ﬁndings indicate that even a coiled antenna is adequate to
conduct a successful attack. This means the antenna can easily
be hidden in, for example, a backpack.

Error State: In all of our tests and independent of the vehicle
and charging station, we observed that after the successful
disruption of the charging communication, the car and the

charger switched into an error state. An example error message
displayed on one of the tested chargers is depicted in Figure 11.
Even though the ISO 15118 standard provides the option of
continuing the charging using the basic PWM communica-
tion [40], it appears that none of the tested vehicles imple-
mented this feature. To continue charging, it was necessary to
manually start an entirely new charging session from scratch.
This means we had to unplug the charging cable from the
vehicle and plug it back into the charging station, wait for a
short period, plug the cable into the car again, authenticate
the charging via the preferred authentication method and wait
for the charging to start. While we have not observed the
behavior for Plug & Charge, we are conﬁdent that the car will
not automatically continue charging once the communication
link is re-established. We consider this to be a safety feature
since the communication is crucial for operating the equipment
safely. To our surprise, one of the tested charging stations did
not show an error at all. Instead, the charger stated that the
charging had ended successfully.

Prevention of New Charging Sessions: In addition to dis-
rupting ongoing charging sessions, we also tested the effects
of the attack if it commences before a charging session
has begun. We started broadcasting the preamble before the
vehicle was connected to the charging station. We then tried
to start a charging session as described above. In line with our
expectations, the charging did not start and the EVSE displayed
an error message. In contrast to the error shown in Figure 11,
no state-of-charge value was displayed in this case — as the
EVSE had never learned it. An adversary could use this variant
of the attack to cause a denial-of-service of a certain charging
hub for a longer period of time, for example, to blackmail the
operator.

Liquid Cooled Cables: We tested the attack on different high-
power DC charging stations with varying maximum charging
capabilities ranging from 22 kW up to 350 kW. Due to the
high heat dissipation during the charging process, the charging
stations must comply with strict regulations to guarantee a safe
operation. More speciﬁcally, for parts of the charging cable
that can be grasped during the charging session, IEC 62196-1
deﬁnes a maximum temperature of 50◦ C for metal and 60◦ C
for non-metal parts [35]. To prevent burn injuries or damage
to the cable, it is crucial to operate the equipment within these
speciﬁed limits. However, as charging power increases, air
cooling becomes inadequate and the use of liquid cooled cables
has become necessary [51]. We expected that the presence of
a liquid jacket in the cable would attenuate the attack signal
substantially and make the attack more difﬁcult. However,
in contrast to our expectations, we could not observe any
difﬁculties interrupting the charging session when the charger
was equipped with liquid-cooled charging cables. We argue
that this is due to the coolant only running in between the
current-carrying DC wires rather than wrapping around the
entire cable, thus excluding the wires (CP and PE) used for
the communication.

VII.

IMPACT

We consider the impact of BROKENWIRE to be signiﬁcant.
In the sections above, we have observed a variety of ways in
which the attack could be successfully deployed and found it
to be reliable, once suitably conﬁgured for the environment.

Each category of attack within our threat model appears valid.
For an attacker with a ‘single vehicle’ objective, the attack
is achievable at a substantial range and almost irrespective
of the target vehicle and charger. The only limitation is
that the attack could affect other nearby vehicles, so it may
be challenging to target only a speciﬁc vehicle at range,
while leaving others unaffected. Directional transmission at
these frequencies requires a large antenna arrangement, which
may limit the attacker if they wish to selectively affect one
vehicle from afar. However, we believe that the ability to
miniaturize the equipment and conceal it still provides ample
opportunity to mount this attack in a different way. Having
demonstrated the disruption of multiple vehicles at once, we
are conﬁdent that ‘ﬂeet denial’ and ‘widespread disruption’
attacks are achievable, scaled principally by the effective range
of the attack. Indeed, in our multiple-vehicle test, we directly
emulated the actions of an attacker attempting to render a
charging site temporarily useless. We consider these as the
more serious versions of the attack and so consider further
details of their scalability in Section VII-C below.

We also recall that the implications of the attack extend far
beyond passenger cars. We identiﬁed a wide range of vehicles
that make use of CCS charging directly, along with derivative
standards based upon it. The Megawatt Charging System
(MCS) and SAE J3105 follow the ISO 15118 standard and use
PLC for the charging communication [11], [33]. While we have
not directly tested either, we see no fundamental reason why
the attack would not transfer directly to these. In both cases,
these derived standards target heavy vehicles with substantial
power draws and consider the needs of large, tightly-packed
charging parks for ﬂeet use. These factors would make the
impact of BROKENWIRE particularly severe in these cases.

A. Transferability

Besides the Qualcomm QCA7000 chips that were used
by the evaluation boards of our testbed, various other HPGP-
compliant modems from other manufacturers exist. Since we
could not access the HPGP modems in our real-world study,
we cannot conﬁrm whether any other modem implementations
were used in the vehicles and chargers we tested. Nevertheless,
as the exploited behavior is part of the HPGP standard, we
expect that any compliant implementation will be vulnerable,
but consider that differences between manufacturers may affect
the level of vulnerability on each.

B. Next-Generation Home Charging

Our evaluation was heavily focused on DC rapid-charging
via the Combined Charging System and Combo 1 and
Combo 2 plugs. However, the new release of ISO 15118-20
standardizes V2G communication and bi-directional charging,
and an increasing number of domestic AC chargers with Type-
2 plugs are being marketed that offer the same PLC commu-
nication link. We therefore tested one such new AC charger
for domestic use that enables bi-directional charging for the
home via ISO 15118. In accordance with our expectations, the
tested charger was also susceptible to the attack and affected
in the same way as DC rapid-chargers. This indicates that
the impact of BROKENWIRE will
increase substantially in
the near future as home chargers incorporate ISO 15118 and
become susceptible. Domestic users typically lack the security

10

e.V., a non-proﬁt association founded by car manufacturers
and suppliers that
leads the standardization of CCS, and
Automotive Information Sharing and Analysis Center (Auto-
ISAC), as well as to different government entities. Although
our source code is now publicly available, we have embargoed
the release of our code during the disclosure period to provide
manufacturers with sufﬁcient time to address the vulnerability
while making it more difﬁcult for a potential attacker to exploit
the vulnerability. Since the disclosure, the attack has been
independently veriﬁed and acknowledged by the industry, and
CVE-2022-0878 has been published as a result.

IX. COUNTERMEASURES

Preventing electromagnetic interference completely is chal-
lenging. In the following, we discuss potential countermea-
sures that cannot fully prevent the attack but raise the bar
high enough to make the attack too resource-intensive for
a would-be attacker. If the required attack budget increases
substantially, the attack becomes potentially unattractive even
for more sophisticated adversaries.

A. Electromagnetic Hardening

Due to the nature of the vulnerability, hardware adjustments
are a good ﬁrst step to improve the security of the affected sys-
tem. In particular, the two single, unbalanced wires currently
used for communication in CCS could either be replaced by
a more EMI-resistant alternative (a balanced twisted-pair or
coaxial cable) or additionally shielded.

Similar to the protection of any other electronic device,
the most straightforward approach to reduce the susceptibility
to electromagnetic waves is electromagnetic shielding. At the
same time, wrapping a conductive layer around the CP and
PE lines would mitigate data leakage through unintentional
electromagnetic emanation [4]. While shielding makes the
attack more difﬁcult,
it. Instead,
it only attenuates the injected signal with the effectiveness
largely depending on the thickness of the shielding [70]. Since
a preamble just above the noise level is sufﬁcient, the adversary
can utilize more powerful equipment or reduce the distance to
ensure the electromagnetic waves can penetrate the shielding
and couple onto the cable. Therefore, shielding can be seen as
an arms race between defender and attacker.

it does not fully prevent

As an alternative to shielding, the cable type can improve
the robustness of a communication against external interfer-
ence. Instead of using two single, unbalanced wires as used
by CCS, a balanced twisted pair cable should harden the
communication against injections.

However hardening is applied, the inclusion of extra wiring
or insulation also has practical issues. Charging cables are
already heavy, stiff, and sometimes difﬁcult to handle and
the addition of additional material would further increase this
problem — making it less usable for drivers while simul-
taneously more costly for manufacturers. This issue is less
pronounced when using twisted pair cabling as it is more
ﬂexible. Finally, any modiﬁcation to the wiring cannot easily
be retroﬁtted onto old cables, so it is necessary to replace the
entire charging cable. This is both costly (circa $1,200 per
cable) and time-consuming. However, for future deployments

Fig. 12: Area of parking bays covered by disruption with a
given effective range, for three different regulation bay sizes.

resources of commercial charge-point operators, in order to
defend against attacks.

C. Estimating Affected Parking Bays

Throughout our experiments in lab settings and real-world
environments, we have considered the attack in terms of the
distance, or effective range. However, we also demonstrated the
capability for the attack to affect multiple vehicles at once and
to work with occluding objects (e.g., cars, barriers) between
the attacker and the victim. As such, we put our results into
greater context by estimating the number of parking bays that
could be affected by an attack of a given range. We stress
that effective ranges are subject to variation and therefore so
are counts of affected parking bays. However, we believe it
is helpful to understand the scalability of the attack and that
our analysis helps to illustrate other important factors, such as
parking lot design.

We built a simulation tool to calculate the coverage of an
attack for arbitrary charging park layouts. The full details are
described in Appendix A. Figure 12 shows the results of this
comparison. Bays begin to be covered by the disruption signal
above 4 m of range. For the tightest arrangement of parking
bays (small cars, perpendicular parking, double-sided rows)
an area equivalent to 80 parking bays would be covered by a
25 m range (with 64 fully overlapped). The same range would
allow coverage of an area equivalent to 22 coach bays in a
perpendicular arrangement (with 16 fully overlapped). Stations
with overhead charging, such as shown in Figure 2b are even
denser, permitting yet more vehicles to be disrupted.

VIII. ETHICAL & LEGAL CONSIDERATIONS

Safety Measures during the Experiments: Given the nature
of the infrastructure under investigation, we collaborated with
a national government department and a local charge-point
operator for our evaluation. We further took precautions to
limit any risk of unintentional effects from our testing. We
selected only test sites for which no other charging parks were
within a reasonable range. We only executed the attack when
no other vehicles were charging and could immediately abort
the experiments if the conditions became uncontrolled. Outside
our closed laboratory sites, we were limited to a maximum
output power of 1 W to ensure our attack signal was compliant
with all national transmission regulations.

Responsible Disclosure: The ﬁndings of this paper have been
disclosed to different standardization bodies, such as CharIN

11

0510152025EﬀectiveRange(m)020406080BaysAreaCoveredEU/UKStandardCarUSLargeCarEU/UKCoachon the cable,
the results also support our hypothesis that
the signal couples mainly onto the charging cable. However,
we would like to note that it is still possible for the signal
to couple onto other components, such as the PCB directly
or the power lines and transformers to which the charging
station is connected.

B. Software Solutions

Hardware countermeasures are often challenging to deploy.
In particular, the high cost of retroﬁtting or replacing vul-
nerable hardware and the limited scalability of the approach
make them unattractive. Therefore, a software-based approach,
which can preferably be rolled out via an over-the-air software
update, would be optimal.

Optimizing the Channel Access Mechanism: BROKENWIRE
is inherent to the channel access mechanism implemented in
the HPGP standard — CSMA/CA. As such, a straightforward
approach to ﬁx the vulnerability would be the deactivation
of this mechanism. However, this decision could cause in-
terference and degrade the performance and reliability of the
communication. Therefore, in contrast to entirely deactivating
CSMA/CA, a more gentle solution would be to adjust the
SNR for which a preamble is detected, seen as valid, and the
communication link considered as busy. Similar to shielding,
this approach does not fully prevent the attack; it only requires
the adversary to increase their maximum power budget for the
attack to be successful.

Enabling Re-Authentication: The communication is only dis-
rupted as long as the adversary continues to emit the malicious
signal. Once the broadcasting stops and the attacker leaves, the
communication link is re-established and the EV and EVSE
could continue to communicate. However, as discussed in
Section VI, none of the tested vehicles automatically continued
the charging process once the session was interrupted. Instead,
the car and charger switched into an error state, forcing the user
to repeat the entire authentication process. We argue that the
attack would be less harmful if the vehicle would automatically
re-establish the charging session once the communication link
is available again. With the help of the Proximity Pilot (PP),
which is part of the CCS Combo 1 and Combo 2 plug and
used to detect if the charging cable has been plugged in, it
would be possible to monitor if the charging cable has been
unplugged. If the circuit has not been interrupted, the charger is
still connected to the same vehicle, which has previously been
authenticated successfully by the user, making a manual re-
authentication by the user unnecessary. With the introduction
of Plug & Charge, the re-establishment of the charging session
is even easier. The car has all the necessary information to
start charging without the need for the user to authenticate.
While this countermeasure does not solve the vulnerability, it
substantially reduces the impact of one-off attacks, such as the
drive-by attack.

Internal Counter: Performance and error metrics are typically
tracked by modems internally and sometimes exposed to
higher layers in the host. These could be used to detect the
abnormal situation that BROKENWIRE creates. For example,
during an attack, the number of new packet detections in-
creases to an abnormally high level — beyond the maximum
that could ever occur in normal operation. The number of

(a) Results in dBm.

(b) Results in mW.

Fig. 13: Minimum required transmission power to cause a
packet loss of 100% for different cable types.

changing the cable can add an additional layer that increases
attack difﬁculty.

Testing Effectiveness: To test how the required transmission
power changes for different cables, we replaced the normal
unshielded power cable with a shielded CAT 7. Ethernet cable
(SF/FTP), an unshielded CAT 5. Ethernet cable (UTP), and
a shielded coaxial cable. The length of the cables matched
exactly the length of the other cable used in previous tests.
The Ethernet cables consisted of four twisted pair sets, but
only one was used for the CP and PE. The twisted pair sets
of the CAT 7. cable were individually shielded, in addition
to an outer foil and braided shielding. The CAT 5. cable was
not shielded at all. In contrast, the coaxial cable was shielded
using a woven copper shield. For each cable type, we repeated
the experiments as described in Section V for distances 1, 2,
and 3 m.

≥

The results of the experiments are depicted in Figure 13.
To our surprise,
the unshielded power cable outperformed
the UTP and coaxial cables for close distance. We argue
that this is due to the higher noise resistance of the cables.
As such, not only the attacking signal is coupling less well
onto the wire, but also the environmental noise, making it
2 dB. However,
easier to achieve the required SNR of
with increasing distance,
the advantage of better cabling
becomes more apparent. For distances above 2 m, all tested
alternatives demonstrated higher resistance against the attack,
with the coaxial cable performing particularly well. While the
testbed with the unshielded power cable was disrupted from
3 m away with around 1.4 mW, the coaxial cable increased
the required transmission power to 8.5 mW. The results
indicate that a different cable can substantially increase the
power required for a successful attack. At the same time,
the results emphasize that electromagnetic shielding and
improved cables on their own are not sufﬁcient to prevent the
BROKENWIRE attack. Nevertheless, we believe that changing
the communication cable in future deployments can easily
be implemented and adds an additional layer of defense by
raising the bar for the attacker.

Since the effectiveness of

the attack varied depending

12

1.02.03.0Distanced(m)°10°50510OutputPowerPout(dBm)SF/FTPUTPCoaxialUnshielded1.02.03.0Distanced(m)−10−50510OutputPowerPout(dBm)1.02.03.0Distanced(m)0.02.55.07.510.0OutputPowerPout(mW)invalid packets also increases, as only the preamble is transmit-
ted and no valid data follow. Detecting an impossible packet
detection rate and a packet error rate approaching 1.0 would
indicate the presence of the attack. This approach does not
prevent the attack, but could be use to drive other, reactive
countermeasures or reported to the driver and charge-point
operator for further action.

C. Increasing the Noise Floor

While the aforementioned solution would be ideal, it might
not be possible to update the ﬁrmware of the PLC modem.
Therefore, we propose a simple and easy-to-deploy defense
technique that is highly effective and only costs a fraction
of what it would cost to replace the entire charging cable.
As mentioned in Section V-E, we observed that
the PLC
communication is surprisingly robust to Gaussian white noise.
Even a very powerful noise signal was not sufﬁcient to degrade
the communication link quality. Our countermeasure exploits
this fact to increase the difﬁculty for the adversary to execute
a successful attack. As a reminder, the BROKENWIRE attack
relies on the injection of a preamble with an SNR > 2 dB.
This means that if the background noise on the wire increases,
the adversary requires more power to keep the SNR of the
injected preamble above this threshold. We propose to add a
small noise source to the PLC modem to intentionally increase
the noise on the communication lines. Usually, noise needs to
be prevented to guarantee a robust communication, however,
in this scenario, noise contributes to the defense against
the BROKENWIRE attack. We admit that this countermeasure
might not be optimal, yet, we argue that the advantages, cheap
and easy to deploy, outweigh the disadvantages.

Method: To test the effectiveness of the countermeasure, we
repeated the experiments as described in Section V. We only
modiﬁed the experimental setup slightly to enable the injection
of noise. More precisely, we connected a Picoscope 5244D as
a noise source directly via the SMA connector provided by the
PLC evaluation boards to the communication lines. We then
started a UDP IPerf session and re-ran the attack for different
distances while injecting Gaussian white noise with varying
amplitude onto the wires.

Results: We found the countermeasure to be effective even
for close attack distances. In Figure 14, the minimum required
transmission power to completely disrupt the communication
for different distances under various noise levels is depicted.
The results show that even for a distance of 1 m, noise with
an amplitude of only 25 mV was adequate to increase the
required transmission power for a successful attack by a factor
of 50. For a packet loss of 100%, a transmission power of
20.6 mW was required compared to 0.37 mW without the
countermeasure in place. Given the maximum output power
of around 1 W of our attack setup, noise with an amplitude of
25 mV was sufﬁcient to prevent the attack from a distance of
10 m. As such, for a ﬁxed power budget, the maximum attack
range is considerably reduced, making wardriving or attacks
at larger charging parks far more challenging, expensive and
potentially unattractive. Surprisingly, the power required to
disrupt the testbed from 2 and 3 m away was almost identical.
As mentioned earlier, we consider this to be a side-effect of
the uncontrollable background noise in our lab environment.
In general, the results underline our hypothesis that a higher

(a) Results in dBm.

(b) Results in W.

Fig. 14: Minimum required transmission power to cause a
packet loss of 100%. With increasing background noise, the
attack becomes progressively more difﬁcult.

noise ﬂoor makes the attack inevitably more difﬁcult. This is
also in agreement with our assumptions drawn from our real-
world evaluation discussed in Section VI, where we observed
that the required output power varies between charging station
locations.

X. RELATED WORK

Disrupting wireless communication has been well stud-
ied [53], [6], [60], [74], [73]. Nevertheless, simple jamming
that aims to decrease the signal-to-noise ratio by emitting noise
in the spectrum of the communication channel is ineffective
and resource-intensive. As such, more intelligent
jamming
strategies that exploit design weaknesses in the targeted pro-
tocol, for example, IEEE 802.11, LTE, or 5G, have been
demonstrated [12], [61], [45], [3]. In addition, denial-of-
service attacks utilizing vulnerabilities at the Medium Access
Control (MAC) layer have proven effective [57]. Exploiting
the CSMA/CA mechanism in IEEE 802.11 networks has been
discussed by different papers [68], [75], [44], [59], with [75]
being the closest to our work. The authors demonstrated that
the throughput of a WiFi channel can be substantially reduced
by injecting fake preambles that cause the target node to back
off and stop communicating.

It is well known that PLC is susceptible to electromagnetic
interference (EMI) [56], [66], [49]. At the same time, PLC
tends to cause electromagnetic emanation, even so strong that
it can interfere with amateur radio [49]. The work by [4]
demonstrated that this also applies to the Combined Charging
System. A similar attack, although wired rather than wireless,
has been demonstrated in [21]. However, to the best of our
knowledge, no research has been conducted that evaluates the
real impact of EMI against the CCS charging communication.
The authors of [5] mentioned only brieﬂy the possibility of
a denial-of-service attack against the charging communication
by emitting noise in its spectrum. And the researchers in [17]
focused on the electromagnetic susceptibility of the voltage
and current sensors in charging stations.

13

1.02.03.0Distanced(m)1015202530OutputPowerPout(dBm)25mV50mV100mV1.02.03.0Distanced(m)1015202530OutputPowerPout(dBm)1.02.03.0Distanced(m)0.00.20.40.60.8OutputPowerPout(W)XI. CONCLUSIONS

In this paper, we presented BROKENWIRE, a wireless
attack against the Combined Charging System (CCS), the most
widely used DC rapid charging standard for electric vehicles in
North America and Europe. We investigated the effects of the
BROKENWIRE attack in a controlled laboratory environment
and an extensive real-world evaluation, including eight EVs
and 20 charging stations. We demonstrated that the attack
can be conducted with only off-the-shelf equipment and with
little knowledge, making the entry barrier for an attacker low.
Based on the insights that we gained from our evaluation,
we proposed, examined, and compared different mitigation
strategies. Our results suggest that the use of PLC for charging
communication is a serious design ﬂaw that leaves millions
of vehicles, some of which belong to critical infrastructure,
vulnerable.

ACKNOWLEDGMENTS

We are grateful for the support from Armasuisse S+T and
EWZ (Elektrizit¨atswerk der Stadt Z¨urich). We would also like
to thank Daniel and Peter K¨ohler for providing access to their
vehicles and supporting us during some of the experiments.
Sebastian was supported by the Hans-B¨ockler Foundation
and the Engineering and Physical Sciences Research Council
(EPSRC).

AVAILABILITY

Our evaluation source code is available at https://github.
com/ssloxford/brokenwire. To facilitate deployment and make
it easier for the community to reproduce our results, the entire
project is dockerized.

REFERENCES

[1] Amazon.com, “Amazon’s custom electric delivery vehicles are starting

to hit the road,” 2021,
https://www.aboutamazon.com/news/transportation/amazons-custom-
electric-delivery-vehicles-are-starting-to-hit-the-road.

[2] Aqua superPower, “Aqua superPower - The Global Marine Charging

Network,” 2022,
https://aqua-superpower.com.

[3] Y. Arjoune and S. Faruque, “Smart jamming attacks in 5g new radio:
A review,” in 2020 10th Annual Computing and Communication
Workshop and Conference (CCWC).

IEEE, 2020, pp. 1010–1015.

[4] R. Baker and I. Martinovic, “Losing the Car Keys: Wireless PHY-Layer
Insecurity in EV Charging,” in 28th USENIX Security Symposium
(USENIX Security 19). Santa Clara, CA: USENIX Association, 2019.
[5] K. Bao, H. Valev, M. Wagner, and H. Schmeck, “A threat analy-
sis of the vehicle-to-grid charging protocol ISO 15118,” Computer
Science-Research and Development, vol. 33, no. 1-2, pp. 3–12, 2018.
[6] E. Bayraktaroglu, C. King, X. Liu, G. Noubir, R. Rajaraman, and
B. Thapa, “Performance of IEEE 802.11 under jamming,” Mobile
Networks and Applications, vol. 18, no. 5, pp. 678–696, 2013.
[7] BBC.co.uk, “London bus garage to become world’s largest ’trial power

station’,” 2020,
https://www.bbc.co.uk/news/uk-england-london-53762711.

[8] BMW Group, “Bidirectional Charging Management

(BCM) pilot
project enters key phase: Customer test vehicles with the ability to
give back green energy.” 2021,
https://www.press.bmwgroup.com/global/article/detail/T0338036EN/
bidirectional-charging-management-bcm-pilot-project-enters-key-
phase:-customer-test-vehicles-with-the-ability-to-give-back-green-
energy.

[9] BYD, “BYD Charging Solutions,” 2022,
https://en.byd.com/charging-solutions.

[10] CharIN e.V., “Grid Integration Levels - Version 5.2,” 2020,

https://www.charin.global/media/pages/technology/knowledge-base/
60d37b89e2-1615552583/charin levels grid integration v5.2.pdf.

[11] CharIN e.V., “Megawatt Charging System (MCS),” 2022,

https://www.charin.global/technology/mcs/.

[12] T. C. Clancy, “Efﬁcient ofdm denial: Pilot jamming and pilot nulling,” in
2011 IEEE International Conference on Communications (ICC).
IEEE,
2011, pp. 1–5.

[13]

J. Crosse, “Kona to Edinburgh: 700 miles in Hyundai’s new EV for
the masses,” 2019,
https://www.autocar.co.uk/car-news/features/kona-edinburgh-700-
miles-hyundais-new-ev-masses.

[14] Daimler Truck AG, “Mercedes-Benz eTruck,” 2020,

https://www.mercedes-benz-trucks.com/fr BL/brand/news/press-
releases/daimler-trucks-e-mobility-group-lance-une-initiative-globale-
en-faveur-de-infrastructure-de-recharge-des-camions-electriques.html.

[15] Daimler Trucks North America LLC, “eCascadia,” 2022,

https://freightliner.com/trucks/ecascadia.

[16] H. Das, M. Rahman, S. Li, and C. Tan, “Electric vehicles standards,
charging infrastructure, and impact on grid integration: A technological
review,” Renewable and Sustainable Energy Reviews, vol. 120, p.
109618, 2020.

[17] G. Y. Dayanikli, R. R. Hatch, R. M. Gerdes, H. Wang, and R. Zane,
“Electromagnetic sensor and actuator attacks on power converters for
electric vehicles,” in 2020 IEEE Security and Privacy Workshops
(SPW).

IEEE, 2020, pp. 98–103.

[18] devolo AG, “dLAN Green PHY eval board II,” 2022,

https://www.devolo.co.uk/dlan-green-phy-eval-board-ii.

[19]

J. Dow, “President Biden will make entire 645k federal vehicle ﬂeet
electric,” 2021,
https://electrek.co/2021/01/25/president-biden-will-make-entire-645k-
vehicle-federal-ﬂeet-electric.

[20] DPD Group, “DPD doubles EV ﬂeet to almost 1,500 with UK’s ﬁrst

MAXUS e Deliver electric van order,” 2021,
https://www.dpd.co.uk/content/about dpd/press centre/dpd-doubles-
ev-ﬂeet-to-almost-1500-with-uks-ﬁrst-maxus-edeliver-electric-van-
order.jsp.

[21] S. Dudek, J.-C. Delaunay, and V. Fargues, “V2G Injector: Whispering

to cars and charging units through the Power-Line,” in SSTIC, 2019.

[22] EnBW, “Verkehrsreiches Autobahnkreuz bei Kamen wird Drehkreuz

f¨ur Elektromobilit¨at im Nordwesten,” 2021,
https://www.enbw.com/unternehmen/presse/groesster-enbw-
schnellladepark-eroeffnet.html.

[23] D. Ferris, “Postal Service: Here’s the price tag for 100% EVs,” 2022,
https://www.eenews.net/articles/postal-service-heres-the-price-tag-for-
100-evs.

[24] H. T. Friis, “A note on a simple transmission formula,” proc. IRE,

vol. 34, no. 5, pp. 254–256, 1946.

[25] Goldhofer AG, “PHOENIX AST-2E,” 2022,

https://www.goldhofer.com/en/towbarless-tractors/ast-2e-phoenix.

[26] Grandview Research Inc., “Electric Vehicle Charging Infrastructure
Market Size, Share & Trends Analysis Report By Charger Type (Slow,
Fast), By Connector (CHAdeMO, CCS), By Application (Commercial,
Residential), And Segment Forecasts, 2021 - 2028,” 2021,
https://www.grandviewresearch.com/industry-analysis/electric-vehicle-
charger-and-charging-station-market.

[27] Green Motion SA, “SKYCHARGER - Setting the standard of cutting-

edge electric plane charging,” 2022,
https://greenmotion.ch/en/products/skycharge.

[28] C. Hampel, “Huge order for Lion Electric trucks from Amazon,” 2021,
https://www.electrive.com/2021/01/11/huge-order-for-lion-electric-
trucks-from-amazon.

[29] C. Hampel, “NHS develops electric ambulances with Ford,” 2021,

https://www.electrive.com/2021/08/09/nhs-procures-ﬂeet-of-electric-
ambulances.

14

[30] R. Harrabin, “Ban on new petrol and diesel cars in UK from 2030 under

PM’s green plan,” 2020,
https://www.bbc.co.uk/news/science-environment-54981425.

[31] HomePlug Powerline Alliance, “Homeplug Green PHY Speciﬁcation,”

2013,
https://web.archive.org/web/20180825120357/https://www.
homeplug.org/media/ﬁler public/74/40/7440ccd5-8c66-49ed-a2ce-
5ef661932c27/homeplug gp speciﬁcation v111 ﬁnal public.pdf.

[32] Honda Motor Europe Ltd., “Honda and V2X Suisse Consortium to
advance Vehicle-to-Grid Charging Technology in Switzerland,” 2022,
https://hondanews.eu/eu/en/cars/media/pressreleases/362145/honda-
and-v2x-suisse-consortium-to-advance-vehicle-to-grid-charging-
technology-in-switzerland.

[33] Hybrid - EV Committee, Electric Vehicle Power Transfer System Using

Conductive Automated Connection Devices, 2020,
https://doi.org/10.4271/J3105 202001.

[34]

[35]

[36]

IEC 61581, Electric vehicle conductive charging system.
Schweiz: IEC, 2017.

Genf,

IEC 62196-1, Plugs, socket-outlets, vehicle connectors and vehicle
inlets. Conductive charging of electric vehicles. General requirements.
Genf, Schweiz: IEC, 2014.

Independent.co.uk, “Germany pushes to ban petrol-fuelled cars within
next 20 years,” 2016,
https://www.independent.co.uk/news/world/europe/germany-petrol-car-
ban-no-combustion-diesel-vehicles-2030-a7354281.html.

[37] R. Irle, “Global EV Sales for 2021 H1,” 2022,

https://www.ev-volumes.com.

[38]

[39]

[40]

ISO 15118-1, Road vehicles – Vehicle to grid communication interface
– Part 1: General information and use-case deﬁnition. Genf, Schweiz:
ISO, 2014.

ISO 15118-2, Road vehicles – Vehicle to grid communication interface
– Network and application protocol requirements. Genf, Schweiz: ISO,
2014.

ISO 15118-3, Road vehicles - Vehicle to grid communication interface
– Part 3: Physical and data link layer requirements. Genf, Schweiz:
ISO, 2015.

[41] K. Joshua, “List of Countries Banning Internal Combustion Engines in

the Near Future,” 2021,
https://www.ecomparemo.com/info/list-of-countries-banning-internal-
combustion-engines-in-the-near-future.

[42] M. Kane, “Incredible Electric Ferry Fast Charges Using 26 Plugs

Simultaneously, But Why?” 2021,
https://insideevs.com/news/466633/electric-ferry-26-plugs-dc-fast-
charging.

[43]

J. Kraus, Antennas for all applications. New York: McGraw-Hill, 2002.

[44] P. Kyasanur and N. H. Vaidya, “Selﬁsh mac layer misbehavior in
wireless networks,” IEEE transactions on mobile computing, vol. 4,
no. 5, pp. 502–516, 2005.

[45] M. J. La Pan, T. C. Clancy, and R. W. McGwier, “Physical layer orthog-
onal frequency-division multiplexing acquisition and timing synchro-
nization security,” Wireless Communications and Mobile Computing,
vol. 16, no. 2, pp. 177–191, 2016.

[46] F. Lambert, “Kia unveils 2020 Soul EV with 201HP, 64kWh, 200+ mile

battery and 100kW CCS charging,” 2018,
https://electrek.co/2018/11/29/kia-2020-soul-ev-battery-pack-range.

[47] F. Lambert, “Tesla conﬁrms Model 3 is getting a CCS plug in Europe,

adapter coming for Model S and Model X,” 2018,
https://electrek.co/2018/11/14/tesla-model-3-ccs-2-plug-europe-
adapter-model-s-model-x.

[48] F. Lambert, “Tesla unveils updated Model S with new headlights,

taillights, and CCS charge ports,” 2022,
https://electrek.co/2022/01/11/tesla-unveils-updated-model-s-new-
headlights-taillights-ccs-charge-ports.

[49] L. Lampe, A. M. Tonello,

Communications: Principles, Standards
Multimedia to Smart Grid, 2nd ed. Wiley Publishing, 2016.

and T. G. Swart, Power Line
from

and Applications

[51] C. J. Michelbacher, S. Ahmed, I. Bloom, A. Burnham, B. Carlson,
F. Dias, E. J. Dufek, A. N. Jansen, M. Keyser, D. Howell et al.,
“Enabling fast charging: A technology gap assessment,” Idaho National
Lab.(INL), Idaho Falls, ID (United States), Tech. Rep., 2017.

[52] Mine Smart Ferry, “Mine Smart Ferry - Innovation,” 2022,

https://minesmartferry.com/innovation.

[53] R. H. Mitch, R. C. Dougherty, M. L. Psiaki, S. P. Powell, and
B. W. O’Hanlon, “Signal characteristics of civil gps jammers,” in
Radionavigation Laboratory Conference Proceedings, 2011.

[54] C. Morris, “The war is over: Nissan to switch from CHAdeMO to

CCS in US and Europe,” 2020,
https://chargedevs.com/newswire/the-war-is-over-nissan-to-switch-
from-chademo-to-ccs-in-us-and-europe.

[55] F. Mwasilu, J. J. Justo, E.-K. Kim, T. D. Do, and J.-W. Jung, “Electric
vehicles and smart grid interaction: A review on vehicle to grid
and renewable energy sources integration,” Renewable and sustainable
energy reviews, vol. 34, pp. 501–516, 2014.

[56] A. Nateghi, M. Schaarschmidt, S. Fisahn, and H. Garbe, “Susceptibility
of Power Line Communication (PLC) Channel to DS, AM and Jam-
ming Intentional Electromagnetic Interferences,” in 2021 Asia-Paciﬁc
International Symposium on Electromagnetic Compatibility (APEMC).
IEEE, 2021, pp. 1–4.

[57] R. Negi and A. Perrig, “Jamming analysis of mac protocols,” Technical

report, Carnegie Mellon University, Tech. Rep., 2003.

[58] T. I. of Structural Engineers, “Design recommendations for multi-storey

and underground car parks,” 2002,
https://masseguridadvial.com/FILES/Underground Carparks EN.pdf.

[59] K. Pelechrinis, M. Iliofotou, and S. V. Krishnamurthy, “Denial of
service attacks in wireless networks: The case of jammers,” IEEE
Communications surveys & tutorials, vol. 13, no. 2, pp. 245–257, 2010.

[60] R. A. Poisel, Modern Communications Jamming Principles and
Techniques, 2nd ed. Norwood, MA, USA: Artech House, Inc., 2011.

[61] H. Rahbari, M. Krunz, and L. Lazos, “Swift

jamming attack on
frequency offset estimation: The Achilles’ heel of OFDM systems,”
IEEE Transactions on Mobile Computing, vol. 15, no. 5, pp. 1264–
1278, 2015.

[62] Royal Mail Group Limited, “Royal Mail switches on all-electric

company car schemes,” 2022,
https://www.royalmailgroup.com/en/press-centre/press-releases/royal-
mail-group/royal-mail-switches-on-all-electric-company-car-schemes.

[63] M. A. Sayed, R. Atallah, C. Assi, and M. Debbabi, “Electric vehicle at-
tack impact on power grid operation,” International Journal of Electrical
Power & Energy Systems, vol. 137, p. 107784, 2022.

[64] Siemens AG, “Siemens technology becoming part of state-of-the-art

bus depot in Hamburg,” 2020,
https://press.siemens.com/global/en/pressrelease/siemens-technology-
becoming-part-state-art-bus-depot-hamburg.

[65] S. Soltan, P. Mittal, and H. V. Poor, “BlackIoT:IoT Botnet of High
the Power Grid,” in 27th USENIX

Wattage Devices Can Disrupt
Security Symposium (USENIX Security 18), 2018, pp. 15–32.

[66] M. M. Soriano and A. W. Mitchell, “Feasibility of powerline com-
munications (PLC) on future spacecraft EMI/EMC test results on
cots PLC technology,” in 2017 IEEE International Symposium on
Electromagnetic Compatibility & Signal/Power Integrity (EMCSI).
IEEE, 2017, pp. 38–43.

[67] M. Sp¨ottle, M. Staats, K. J¨orling, M. Schimmel, J. Gartner, L. Jerram,
W. Drier, and L. Grizzel, “Research for TRAN Committee – Charging
infrastructure for electric road vehicles,” Brussels, 2018,
https://www.europarl.europa.eu/RegData/etudes/STUD/2018/617470/
IPOL STU(2018)617470 EN.pdf.

[68] A. L. Toledo and X. Wang, “Robust detection of MAC layer denial-of-
service attacks in CSMA/CA wireless networks,” IEEE Transactions on
Information Forensics and Security, vol. 3, no. 3, pp. 347–358, 2008.

[69] V. Tomlinson, “UPS invests in Arrival and orders 10,000 Generation 2

Electric Vehicles,” 2020,
https://arrival.com/uk/en/news/ups-invests-in-arrival-and-orders-
10000-generation-2-electric-vehicles.

[50]

J. R. . S. Ltd., “Car Parking Bays,” 2022,
https://multi-storey-car-parks.com/car parking bays.html.

[70] X. C. Tong, Advanced materials and design for electromagnetic

interference shielding. CRC press, 2016.

15

[71] UK Ministry of Defence, “Army announces battleﬁeld vehicle

electriﬁcation plans,” 2021,
https://www.army.mod.uk/news-and-events/news/2021/09/army-
announces-battleﬁeld-vehicle-electriﬁcation-plans.

[72] Volkswagen AG, “Convenient, networked and sustainable: new

solutions for charging electric Volkswagen models,” 2022,
https://www.volkswagen-newsroom.com/en/press-releases/convenient-
networked-and-sustainable-new-solutions-for-charging-electric-
volkswagen-models-7695.

[73] M. Wilhelm, I. Martinovic, J. B. Schmitt, and V. Lenders, “Short paper:
Reactive jamming in wireless networks: How realistic is the threat?”
in Proceedings of the fourth ACM conference on Wireless network
security, 2011, pp. 47–52.

[74] W. Xu, W. Trappe, Y. Zhang, and T. Wood, “The feasibility of launching
and detecting jamming attacks in wireless networks,” in Proceedings of
the 6th ACM international symposium on Mobile ad hoc networking
and computing, 2005, pp. 46–57.

[75] Z. Zhang and M. Krunz, “Preamble injection and spooﬁng attacks in
wi-ﬁ networks,” in 2021 IEEE Global Communications Conference
(GLOBECOM).
J. Zyren, “EV Combined Charging System Featuring HomePlug Green
PHY,” 2015,
https://www.qualcomm.com/sites/ember/ﬁles/uploads/ev combined
charging qualcommautotechconf april 2015.pdf.

IEEE, 2021, pp. 1–6.

[76]

APPENDIX

Comparison to Friis Equation:

In order to make estimates of transmission power re-
quirements, we ﬁrst measured the ambient noise level on the
cable, ﬁnding it to be 2.45 mV. The HPGP standard states
a conformance threshold for detection of preambles (Section
3.8.4.2 [31]):

The receiver shall be able to detect the presence
of Preamble Symbols within a Slot Time with a
miss rate not exceeding 1% using the standard North
American mask [...]
When the desired Preamble Symbol waveform
present at the receiver has a signal power of -35
dBm and is corrupted by Gaussian noise producing
a total SNR of 2 dB at the receiver terminal.

providing an equation as:

SN RdB = 10

log10(

·

mean(x(t)2)
mean(n(t)2) ·

BWN oise
BWSignal

)

(1)

with x(t) as the time-domain signal, n(t) as the noise
signal and BWN oise, BWSignal as the bandwidths of noise
and signal measured in subcarriers.

Based on this, we re-arrange as follows, simplifying the
mean() of time-domain signals as our values are already
averages:

ˆx2 = 10

SN RdB
10

(ˆn2

·

·

BWSignal)

1
BWN oise

·

(2)

and divide for speciﬁed cable impedance of 100 Ω [18].

Setting the value of n to 2.45 mV and taking SN RdB as
2, per the standard, we can compute the minimum power of

16

Fig. 15: Layout of the charging hub shown in Figure 1
demonstrating that with 15 m attack range, an adversary can
disrupt the charging of roughly 22 vehicles simultaneously.

a preamble signal that must be detected in order to comply.
Taking a common formulation of the Friis equation [24]:

we rearrange as:

PR =

PT GT GRλ2
(4πR)2

PT =

PR(4πR)2
GT GRλ2

(3)

(4)

We model the antennas as dipole and monopole and estimate
their gains from typical values (GT = 2 dBi, GR = 4 dBi)
and apply this equation to compute the expected transmission
power PT for a signal, assuming free-space path loss and a
distance of R. The results are shown in Figure 7 in the main
text.

Parking Bay Coverage Simulation:

In order to help assess the scalability of BROKENWIRE,
we created a simulation tool to help estimate how the effective
range of an attack covers bays in parking lots.

In Figure 15, we illustrate the layout of the charging
hub pictured in Figure 1 and show how a ‘drive-by’ attack
with an effective range of 15 m could be expected to affect
approximately 22 vehicles at once.

However, the design of parking lots varies substantially, not
only in terms of regulations on sizing and angle, but also in
layout, which is subject to site constraints and design choices
by the land developers. In Figure 15 there is a central, double-
sided row (i.e., bays on both sides of a sidewalk) and another
single-sided row, offset in the top-left.

Given this variation, our simulation accepts as input an
arbitrary description of the layout of a parking lot, along with
an attacker location and effective range. The simulation then
estimates the number of parking bays that could be affected
by an attack of that size. Bays can be deﬁned with any size,
shape and orientation, allowing the simulation to consider
bays suitable for any type of vehicle. An attacker can be
situated at any point within the layout — including outside
the parking lot. They are modeled as a point-source with a

EV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLYEV ONLY15 muniform, circular radiation pattern. The simulation computes
the total area of parking bays covered by an attack of a given
range. Although we have shown effectiveness of the attack
between ﬂoors, we implement only the 2-dimensional case
here. We make the simulation tool available as part of our
open source code.

For the analysis in the main text, we consider an example
parking lot arrangement. This does not follow a speciﬁc real-
world case, as in Figure 15, but a generally common de-
sign. While there are site-speciﬁc variations, parking bays are
typically arranged in regular patterns, broken only by access
roads and site boundaries. We use a double-sided arrangement
in repeated rows, with chargers between rows, perpendicular
parking and a range of bay sizes and spacings from regulations
around the world [58], [50]. EU/UK ‘normal’ bays are 2.4 m
4.8 m with access road width of 6 m. US ‘large’ bays are
6.5 m with 9.2 m roads. EU/UK coach bays are 3.5 m

×
3.2 m

×

14 m with 13 m roads.

×

The parking lot is scaled to be sufﬁciently large for any
effective range selected. Our selected layout represents the
most dense arrangement used in real parking lots — such as
in Figure 15. As such our results give an upper bound on the
scale of the attack, yet still a realistic one.

We selected an attacker position that maximizes the cover-
age of parking bays: centrally-positioned in the parking lot on
an access road. We simulate for distances up to 25 m, noting
that while we achieved success at nearly double this range,
charging parks with more than 50 fast-charging bays are all
but non-existent at time of writing.

17


