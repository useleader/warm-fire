---
publish: true
---

Exploiting Sequence Number Leakage: TCP
Hijacking in NAT-Enabled Wi-Fi Networks
Yuxiang Yang∗, Xuewei Feng∗, Qi Li†§, Kun Sun‡, Ziqiang Wang¶, and Ke Xu∗§(cid:12)
∗Department of Computer Science and Technology & BNRist, Tsinghua University
†Institute for Network Sciences and Cyberspace & BNRist, Tsinghua University, §Zhongguancun Lab
‡Department of Information Sciences and Technology & CSIS, George Mason University
¶School of Cyber Science and Engineering, Southeast University
{yangyx22@mails, qli01@, xuke@}tsinghua.edu.cn, fengxw06@126.com, ksun3@gmu.edu, ziqiangwang@seu.edu.cn
Abstract—In this paper, we uncover a new side-channel tunnels of wireless routers [51], creating a rogue clone (i.e.,
vulnerability in the widely used NAT port preservation strategy Evil-Twin)ofthenetwork[2],orjustabusingtheclassicARP
and an insufficient reverse path validation strategy of Wi-Fi poisoning attack [19] to hijack the communication between
routers,whichallowsanoff-pathattackertoinferifthereisone victim clients and servers, thus disrupting normal user usage,
victim client in the same network communicating with another
stealing confidential information, and potentially causing fi-
host on the Internet using TCP. After detecting the presence of
nanciallosses.Fortunately,mostofthepriorattackshavebeen
TCP connections between the victim client and the server, the
repairedormitigatedandtargeteddefensemeasureshavebeen
attacker can evict the original NAT mapping and reconstruct a
proposed as well [54], [35], [53], [57]. Nowadays, with the
new mapping at the router by sending fake TCP packets due
to the routers’ vulnerability of disabling TCP window tracking widespreaddeploymentofwirelesssecuritymechanisms(e.g.,
strategy, which has been faithfully implemented in most of the WPA2 and WPA3) and the adoption of protection strategies
routers for years. In this way, the attacker can intercept TCP (e.g.,APisolation,ARPprevention,andRogueAPdetection),
packets from the server and obtain the current sequence and it is increasingly difficult for an off-path attacker (i.e., with
acknowledgment numbers, which in turn allows the attacker to no control over the router) to obtain the communication
forcibly close the connection, poison the traffic in plain text, or information between other clients in the same Wi-Fi network
reroute the server’s incoming packets to the attacker.
and outside servers.
Wetest67widelyusedroutersfrom30vendorsanddiscover
In public Wi-Fi networks, network address translation
that 52 of them are affected by this attack. Also, we conduct an
extensive measurement study on 93 real-world Wi-Fi networks. (NAT) is widely used to save IPv4 address space and protect
The experimental results show that 75 of these evaluated Wi-Fi internal clients from being identified by external attackers.
networks(81%)arefullyvulnerabletoourattack.Ourcasestudy After attaching to the same Wi-Fi network enabling NAT,
showsthatittakesabout17.5,19.4,and54.5secondsonaverage clients share the external IP address to access the Internet.
toterminateanSSHconnection,downloadprivatefilesfromFTP When it takes the upper protocols (e.g., TCP and UDP)
servers,andinjectfakeHTTPresponsepacketswithsuccessrates into consideration, the router will create NAT mappings to
of87.4%,82.6%,and76.1%.Weresponsiblydisclosethevulner-
keep track of the connections, which record the IP addresses,
ability and suggest mitigation strategies to all affected vendors
upper-level information such as protocol, ports, timeout, and
andhavereceivedpositivefeedback,includingacknowledgments,
reply status, etc. In most cases, the router tries to keep the
CVEs, rewards, and adoption of our suggestions.
layer-4 information the same as the originators, such as the
TCP source port, which is the so-called port preservation
I. INTRODUCTION
strategy[14].However,casesarethatsomeclientsintheLAN
Wi-Fihasemergedasoneofthemostpopulartechnologies may communicate with the same remote server with the same
for providing Internet access, being widely used in restau- source port at the same time as they have no idea about each
rants, offices, coffee shops, airports, and other public places. other. Although with very little probability, the router has to
However, Wi-Fi networks are often exploited by malicious deal with these cases and it will assign a new TCP source
attackers to launch various attacks. In addition to exploiting port, change the IP address, and port at the same time when
vulnerabilitiestobreaktheprotectionofencryption[55],[56], TCP packets pass through it. Besides, due to reasons such as
[57], a lot of prior works have already been conducted on performanceconsiderations,therouterwillnotrecordallofthe
session hijacking in Wi-Fi networks [47], [53], [51], [2], [19], sessioninformationintheNATmappings,suchastrackingthe
e.g., injecting forged wireless frames via vulnerabilities in currentTCPwindow.Thus,itwillnotcheckthesequenceand
WPA2 implementations [47], [53], eavesdropping on wireless acknowledgment numbers strictly when TCP packets arrive.
channels [37], intercepting packets via side channels in VPN
In this paper, we uncover a new off-path TCP hijacking
attack in Wi-Fi networks that exploits vulnerabilities in the
NAT mapping strategies of routers. The attack includes three
NetworkandDistributedSystemSecurity(NDSS)Symposium2024 steps.First,theattackerprobestherouter’sexternalIPaddress,
26February-1March2024,SanDiego,CA,USA
identifies whether AP isolation is enabled and scans to find
ISBN1-891562-93-2
potential victims in the same network when it is disabled.
https://dx.doi.org/10.14722/ndss.2024.23419
www.ndss-symposium.org Second, the attacker infers the presence of TCP connections
4202
rpA
6
]RC.sc[
1v10640.4042:viXra

between any client and a remote server by sending fake TCP source port, as there is no port collision anymore. And the
SYN and SYN/ACK packets. Third, the attacker evicts the packet will match the victim connection from the perspective
original NAT mapping of the victim connection with forged of the server, which will return a TCP ACK packet carrying
RST packets and replaces it with a new mapping at the theexactsequenceandacknowledgmentnumbersofthevictim
router by sending a TCP data packet to the server. After connection upon seeing the packet with wrong numbers [42].
that, it can intercept the ACK packet from the server that is When arriving at the router, this ACK packet will be routed
meant to send to the victim and thus obtain the sequence to the attacker as the NAT mapping has been falsified, and
and acknowledgment numbers within it so as to completely thus it steals the sequence and acknowledgment numbers of
hijacktheTCPconnection.Theattackeronlyneedstoconnect the victim connection easily, i.e., without traversing the 32-bit
to the same network as the victim client, and it does not space to infer these numbers as previous methods [6], [10],
need any assistance of malicious puppets, i.e., unprivileged [7]. It should be noted that the NAT mapping timeout will
applications or sandboxed scripts deployed on victim clients. be refreshed if there are related packets traveling through the
Compared with prior attacks, our work sheds light on the router, which may interfere with the attack. We will analyze
vulnerabilities existing in the abusing peculiarities of NAT the detailed influence in Section VI-A.
strategies and behaviors of routers instead of flaws in TCP
Once the sequence and acknowledgment numbers are ob-
specifications,andourattackisnotlimitedtospecificscenarios
tained by the attacker, it can choose to launch three types of
orapplications(e.g.,WPA2/WPA3,orVPNs).Besides,theOS
attacks: (i) TCP Denial-of-Service (DoS) attack to terminate
types or versions of the clients and servers are unrestricted
victim TCP connections directly by sending RST packets.
in our attack in contrast to previous TCP hijacking attacks
(ii) TCP hijacking attack to take over the NAT mapping
that can only target servers or clients with specific operating
and replace the victim by itself since the router will con-
systems[17], [43], [44].
tinue forwarding packets (intended for the victim client) to
In our investigations, most Wi-Fi routers (e.g., Asus, Net- the attacker instead. (iii) TCP injection attack to poison
gear, Linksys, TP-Link, Huawei, and Xiaomi) adopt the port the victim TCP traffic by sending crafted data packets after
preservation strategy when creating new NAT mappings for restoringthemappingforthevictimclientviaissuingspoofed
TCP connections initiated by internal clients [22], [18]. The TCP RST and ACK packets. Note that traffic encryption (e.g.,
attacker can intentionally initiate a connection, i.e., sending a HTTPS) may disturb the attacker’s poisoning. However, about
SYN packet, to the target server with a guessed client’s port 20% of websites still transmit traffic in plaintext according
anddistinguishtheguessbyobservingwhethertheportwillbe to the reports on HTTPS adoption1. AP isolation may also
changed at the router as a collision will happen if it is a right influence the TCP injection attack due to the requirement of
guess. Theattacker cansend aspoofed SYN/ACK packet with reconstructing the client’s original NAT mapping. However,
a source address of the remote server, a destination address the other two attacks (i.e., TCP DoS and hijacking attacks)
of the external IP of the router, and a destination port of the are not affected, which will be illustrated in Section VI-A.
guessed port as a response to verify if the port is changed.
We conduct a large-scale empirical study to demonstrate
If the router disobeys the RFC recommendation to enable the
that the attack can be performed to cause potential damage
reverse path validation with a strict mode [48], [4], the forged
in the real world. First, we investigate the default settings of
SYN/ACK packet cannot be detected and will not be dropped
routers on the market and have tested 67 widely used router
bytherouter,whichisoftenthecaseinmostrouterswetested.
models from 30 vendors and find that 52 of them from 24
If there is any connection from the LAN to a target remote
vendors are vulnerable to the attack. Moreover, our empirical
server with the guessed source port, the router will choose
measurement results show that the attacks can be successfully
another source port to initiate the connection, and then the
performed in various real-world Wi-Fi networks. We evaluate
SYN/ACK packet will be forwarded to the victim. Yet if there
93Wi-Finetworksinsixmonths,includingmostofthepopular
is no connection with this source port from the LAN to the
Wi-Fi scenarios (e.g., Wi-Fi networks in coffee shops, hotels,
server, the router will keep the port to initiate the connection,
bookstores, and enterprises). The experimental results show
and then the SYN/ACK packet will be forwarded back to the
that 75 (81%) out of these evaluated Wi-Fi networks are fully
attacker. In this way, the attacker can infer whether any client
vulnerable to our attacks. We implement a PoC and perform
is communicating with the server and the source port of the
case studies on applications like SSH, FTP, and HTTP to
client if there is such a TCP connection.
validate the effectiveness of the attack. In our experiments, an
After identifying a target TCP connection, the attacker can off-path attacker can detect and terminate an SSH connection
directly get the sequence and acknowledgment numbers of in 17.5 seconds with a success rate of 87.4%, download
the connection by exploiting a new vulnerability arising in private files from an FTP server within 19.4 seconds with a
the disabled TCP window tracking strategy of Wi-Fi routers. success rate of 82.6%, and manipulate web traffic within 54.5
As routers pursue higher performance, they choose to disable secondswithasuccessrateof76.1%,onaverage.Theseresults
TCP window tracking by default, i.e., they will not check demonstratethatthisattackisfeasibleandmaythrowpotential
the sequence and acknowledgment numbers strictly in TCP threats to normal Wi-Fi users.
packets. So the attacker can send forged TCP reset packets
Finally, we identify the root cause and suggest mitigation
to clean the NAT mapping of the victim connection. After
tovoidthisattackwiththeintuitiveideaofbreakingthecondi-
waiting for the timeout of the NAT mapping (i.e., 1 second or
tions of the attack. Besides, we have responsibly disclosed the
10seconds),theattackercansendaTCPdatapacketusingits
vulnerability to the affected router vendors and the OpenWrt
private IP address and the same source port to the server with
arbitrary sequence and acknowledgment numbers. The router 1See https://w3techs.com/technologies/details/ce-httpsdefault for daily
will only translate the IP address of the packet except for the statisticsonHTTPSadoption.
2

community with affirmative feedback. At the time of writing, port (e.g., another random unused port). (2) random selection,
researchers from the OpenWrt community and 7 of these where the NAT device translates the source port to another
vendors have confirmed the vulnerability and are repairing it random port from a pool of available ports. (3) sequential
in their products according to our suggestions. In addition, 10 selection, where the NAT device selects a random port for
CVE numbers have been assigned for this vulnerability from thefirstconnectiontoeachdestinationandtranslatestheports
different vendors (i.e., from CVE-2023-30305 to CVE-2023- of subsequent packets to that destination consecutively based
30314).Therestvendorsarestillintheprocessofinvestigating on the first port. (4) port overloading, where the NAT device
the vulnerability. always uses port preservation even in the case of collision.
In this case, new connections will take over the original
Contributions. Our main contributions are the following:
mapping, and the old connection will be disturbed, which is
• We uncover a new side channel vulnerability of the NAT not recommended in RFC 5382 [14].
behaviors in Wi-Fi networks that can be exploited to
Aswithanystatefulmiddledevice,routershavetomanage
attack TCP connections by off-path malicious insiders.
the state of mappings and track active flows. Generally, the
• We perform a large-scale measurement and reveal a
routers often rely on both the states of connections and
number of routers vulnerable to the attack. Our extensive
timeouts of mappings to prune unnecessary NAT mappings.
evaluationsagainst67widelyusedroutermodelsandcase
RFC 5382 recommends that the minimum timeout for the
studiesin93variousWi-Finetworksshowthatourattacks
ESTABLISHED state is 2 hours and 4 minutes, which is
can cause potential damage in the real world.
faithfully implemented in most routers [14], and the routers
• We suggest three countermeasures by eliminating the
also set timeouts for other states (e.g., 1 second or 10 seconds
conditions to fight back the attack, and some of them
for the CLOSE state which the mapping will turn into upon
have been adopted by the affected manufacturers.
seeing corresponding RST packets).
II. BACKGROUND
B. TCP Window Tracking in Routers
A. NAT and Port Allocation Strategies
Asamiddledevicebetweentheclientandserver,therouter
Network Address Translation (NAT) is a technology de- has to record the connection information of the related hosts
veloped to solve the shortage of IPv4 addresses and hide the for subsequent packet delivery. However, as the TCP protocol
network topology from an external entity [5], which is widely was originally designed for end-to-end communication and
used by routers in Wi-Fi networks. When packets traverse did not take the middle devices into consideration, the router
through the router, it has to translate the IP addresses of the cannot and will not record all of the information due to
packets between internal and external addresses2 and record manyreasons(e.g.,performanceconsiderations).Forinstance,
the other necessary information of the related connection. the router will choose not to track the current TCP window
The router maintains a NAT mapping table to keep track of the connection, and thus it will not check the sequence
of the internal IP addresses and ports associated with each and acknowledgment numbers of TCP packets strictly. The
corresponding external IP address and port, which allows in- open-sourced router operating systems, i.e., OpenWrt and
comingpacketstobedirectedtothecorrecthostontheprivate AsusWrt, both have related options to reduce CPU overhead,
network.SinceourworkfocusesontheTCPprotocol,wewill i.e.,thenf_conntrack_tcp_no_window_checkoption
illustrate the NAT behavior of TCP mappings henceforth. in OpenWrt and the ip_conntrack_tcp_be_liberal
optioninAsusWrt.Theseoptionsaresettotruebydefault,and
When an internal host initiates a connection to an external
once they are set, Netfilter [25] will not perform TCP window
server, i.e., sending a SYN packet, the router will create a
trackingincontrasttotheoriginalLinuxkernel.Thedifference
new mapping in the table, which is called a binding in NAT
between the two systems is that OpenWrt does not check the
terminology [20]. Besides, we find that not only SYN packets
sequence number of the packet at all, while AsusWrt only
but also packets with PUSH, or ACK flags can incur new
checks if thesequence number is beyond thecurrent sequence
NAT mappings at the router. The mapping will record the
numberina2Gspace.Besides,wefoundmostoftheroutersin
source IP addresses and ports translated before and after, the
the market also disable the TCP window tracking strategy by
destination IP address and port, protocol, session state, and
default and have similar behaviors to the two systems above.
corresponding mapping timeout. After the replies from the
external host arrive, the router forwards the packets to the We will show that routers disabling TCP window tracking
internal host according to the mapping and updates its state can be abused by an off-path attacker to clean the NAT map-
simultaneously. Since the related RFCs have not proposed a pings of other clients with forged RST packets. For OpenWrt-
fixedstrategyforthetranslationbehaviorofsourceports,itcan based routers and those with similar settings, the attacker can
bedifferentdependingontheimplementationofNATdevices, useoneforgedRSTpacketwithanyarbitrarysequencenumber
which includes the following strategies [18]: tocleanthemapping,andforAsusWrt-basedroutersandthose
with similar settings, the attacker can forge two RST packets
(1) port preservation, where the NAT device attempts to
specified with two sequence numbers in the gap of 2G to
preservethesourceportifpossible.Whenacollisionhappens,
bypass the range check easily and effectively.
i.e., different internal hosts choose the same source port to
communicate with the same external host of the same port,
theNATdeviceshouldresolvethecollisionbyselectinganew C. Reverse Path Validation
2SinceourworkconsidersmultiplelevelsofNAT,therouter’sexternalIP To prevent IP spoofing attacks and promote the process of
addressmaynotbeapublicIPaddress. source address validation, RFC 2827 and RFC 3704 propose
3

the concept of reverse path validation, which verifies the A router acts as the gateway of clients in the LAN to provide
authenticity of inbound traffic by checking whether the source Internet services for the Wi-Fi network.
IP address can be routed back via the interface on which
packets are received against the routing table, to ensure they Existing studies [2], [39] demonstrate that a malicious
come from an authorized sender [48], [4]. With this strategy insider can create an evil twin of the network and trick the
enabled, only if the packets can be routable back from the victims into connecting to it by broadcasting the same SSID
incoming interface will they be processed by the kernel and in the open (with no encryption) or home mode (accessed
routed to their destinations. Otherwise, they will be dropped. through pre-shared key) Wi-Fi networks, thus hijacking the
Most Linux-based systems control the strategy through the traffic in the network. However, these attacks can be throttled
rp filter kernel variable, which offers three options [26]: by existing defenses, e.g., Rogue AP detection [21], [24]. It is
widelybelievedthatonlyAPisolationenabledenterprisemode
• 0:Inthismode,thesourceaddressvalidationisdisabled. Wi-Finetworkscaneffectivelyprotectclientsfromeachother,
• 1: Strict Mode as defined in RFC3704. In this mode, the whereasopenandhomemodeWi-Finetworksfacechallenges
device should compare the source address of incoming in preventing insider threats. In this work, we propose a novel
packets to the Forwarding Information Base (FIB). If the attackthatcanevadealldefensesaboveinWi-Finetworks.As
incoming interface is not the best reverse path, packets a result, our attack holds particular significance for enterprise
will be dropped. mode Wi-Fi networks, differentiating it from the rogue clone
• 2: Loose Mode as defined in RFC3704. In this mode, the attacks in open and home mode Wi-Fi networks. Moreover,
device compares the source address of incoming packets our attack can serve as an alternative method to compromise
against the FIB, and only if the packets are not reachable openandhomemodeWi-Finetworks.Inourattack,weassume
via any interface will they be dropped. thatwiththedeploymentofsecuritymechanisms(e.g.,WPAs)
and the usage of security protection strategies (e.g., ARP
RFC3704recommendsusingthestrictmodetopreventIP prevention, AP isolation, and Rogue AP detection), an off-
spoofingattacks.Theloosemodeisrecommendedifthedevice path attacker would not be able to discern if any client is
uses asymmetric routing (e.g., a mobile phone with a Wi-Fi communicating with a specific remote server. Furthermore,
interface and multiple interfaces for receiving packets from the attacker would not be able to ascertain the source port
cell towers) or other complicated routing strategies. Previous of the TCP connection, if it exists, and the sequence and
research [51] has shown that in the VPN scenarios, the lack acknowledgment numbers.
of reverse path validation on client devices allows a blind in-
path attacker (e.g., a router controlled by an attacker) to spoof
packets to learn the virtual IP used by the tun0 interface for
the VPN connection and infer the necessary fields to hijack
the active connection. By contrast, we find that most routers
also do not obey the recommendation, and they will not drop
Attacker
packets with spoofed source addresses matching a connection Internet
in the NAT mappings and will accept them on any interface.
Router Server
We will show that an off-path attacker in the LAN can
abuse routers without reserve path validation to forward
Victim
spoofed SYN/ACK packets with the server’s IP address as
the source and the router’s external IP address as destination,
Fig.1. ThreatmodelofTCPhijackingattacksinWi-Finetworks.
which can be leveraged to infer source ports of connections
used by other clients through observing the whereabouts of
these SYN/ACK packets. Additionally, the attacker can also
send forged RST packets to the router’s external IP address. To successfully launch our attacks, there are some require-
Though the source address specified in the packets is the ments to be fulfilled. First, the attacker should be able to
server, the router without reverse path validation will process probe the external IP address of the router. We will illustrate
them in the kernel mistakenly and thus change the state of the our methods in Section IV-B. Then the attacker tests whether
NAT mappings to CLOSE, leading to our attack. AP isolation is enabled in the network3 [32]. The attacker
can successfully carry out the TCP DoS and TCP hijacking
attacks regardless of whether AP isolation is enabled. When
III. THREATMODEL
AP isolation is enabled, the attacker will not be able to probe
Figure 1 illustrates the threat model of our off-path TCP potential victim clients within the network using scanning
attacks in Wi-Fi networks. The model consists of three hosts tools (e.g., Nmap [36]). Thus, the TCP injection attack will
and one router, namely, a remote server, a victim client, an be thwarted with AP isolation. We will discuss the impacts of
off-path attacker, and a vulnerable router. The remote server APisolationonourattacksinSectionIV-DandSectionVI-A.
may be a web application, an SSH or FTP server in different Besides, the attacker has to target the remote server that the
attack scenarios. The victim client (e.g., a mobile phone or a client is communicating with or will connect to, which can be
laptop)isconnectedtoawirelessaccesspointtocommunicate set as those providing popular services as previous works [6],
withtheremoteserverontheInternet,i.e.,visitingwebpages, [10], e.g., famous servers, web search engines, or social sites.
downloading files through FTP, or using the SSH service to
controlremotehosts.Theoff-pathattackerisamaliciousclient 3Previous research has revealed that nearly 89% of the public Wi-Fi
who can access the same Wi-Fi network as the victim client. networksallowclientstocommunicatewitheachother[9].
4

Second, the router adopts the port preservation strategy. given client is communicating with the server [6], [10] or
Our investigations show that most routers adopt this strategy redirecting the victim’s traffic to the attacker [9].
except for an enterprise router model from Huawei and the
Secondly, the attacker probes the router’s external IP ad-
open-sourced routing firmware of pfSense [40]. Also, the
dress. With the widely deployed carrier-grade NAT [45], the
routerdisablesthereversepathvalidationstrategy.Wefindthat
Wi-Fi networks in the real world may consist of multiple
routers from 24 out of 30 vendors will forward forged packets
levels of NAT [15], which means that the router’s external IP
except for Asus, Aruba, Cisco Meraki, Netgear, pfSense,
address is not always a public IP address that can be obtained
and ZTE. Besides, some models from TP-Link, Mercury and
easily by querying its own public IP. We adopt the following
Huaweialsoenablethisstrategy.Moreover,therouterdisables
methods to deal with this problem. (i) First, the attacker gets
the TCP window tracking strategy. In our measurement, most
the gateways along the way to any outside host (e.g., 8.8.8.8)
routershavedisableditbydefault,withtheexceptionofCisco
through Traceroute [52]. Second, the attacker issues the ping
Meraki.
command to the second gateway with the RECORD ROUTE
Third, the victim client does not communicate with the option, which will record the passed routes [41], and then
server frequently. The state of the NAT mapping will transfer all the IP addresses of the passed interfaces will be returned.
from ESTABLISHED to CLOSE state after receiving corre- The result snapshot of the method is provided in Appendix A
spondingTCPRSTpackets,andthemappingwillberemoved (refertoFigure7).(ii)Incertainscenarios,theaforementioned
completely after its timeout (1 second or 10 seconds in our method may encounter failure, as the passed routes might
test). It should be noted that if the client’s communication not be returned when pinging the second gateway. In such
continuesduringthisperiod,itmayinterferewiththeattackas cases, the attacker can opt to scan the subnet of the second
the mapping will be refreshed. As there are many long-lived gateway to identify live hosts’ IP addresses. Subsequently, it
TCP connections that clients periodically retrieve new data can proceed to ping these IPs using the RECORD ROUTE
fromtheserverinminutesand42%ofthetestedrouterssetthe option. When the ping reaches the external IP of the router,
timeout to only 1 second, the attacker has been provided with the previously passed routes will be returned. However, when
enough time to finish its attack. We will analyze its influence pinging other IPs, the routes will not be returned. Besides,
in detail in Section VI-A. the attacker can access these IPs via a web browser. When
accessingtheexternalIPoftherouter,therouter’ssettingpage
IV. ATTACKPROCEDURE (Web GUI) will be displayed, whereas accessing the other IPs
will lead to different pages.
A. Attack Overview
To perform our attacks, the attacker has to carry out the
C. Phase 2: Making Inferences about Active Connections
following three steps:
Assuming that the attacker has connected to a Wi-Fi
1. Probe the router’s external IP address and identify network, in which one of the normal users has established
whether AP isolation is enabled, thus finding potential a TCP connection with a remote server from source port m.
victim clients. The router has a corresponding NAT mapping to keep track
2. Make inferences about whether there is any active con- of the connection. The attacker intends to infer which source
nection from the LAN to the server. port is used by the victim client of the connection.
3. Evict and construct NAT mappings at the router and
theninterceptthesequenceandacknowledgmentnumbers Figure 2 illustrates the side-channel vulnerability that
from the replies to unsolicited packets from the server. leverages the NAT port preservation strategy and insufficient
reversepathvalidationoftherouter.First,theattackersendsa
After the above steps, the attacker can terminate the SYN packet targeting the server with its own IP address and a
connection directly or hijack the connection by replacing the guessed port number as the source. If the source port number
victim client. Besides, when AP isolation is disabled, the (e.g., n) specified in the SYN packet does not equal m, the
attacker can restore the original NAT mapping of the victim router will create a new NAT mapping with source port n
clientattherouterandsendfakeresponsepacketstotheclient. thatrecordsthisnewTCPconnection.Notethatwiththewide
deploymentofNIDSattheserverside[29],alargeamountof
B. Phase 1: Probing the Network SYN packets arriving at the server may be detected, and the
server may find it attacked and take corresponding fightback.
Inthisstep,theattackerpreparestheattackintwoaspects,
The attacker can set the TTL of the SYN packet to a small
namely, identifying the status of AP isolation in the network
number(e.g.,2inourtest),andthusthepacketwillbedropped
and probing the external IP address of the router. Firstly, the
quickly at the intermediate routers. In most cases, the routers
attacker detects whether AP isolation is enabled via network
do not deploy detection systems typically.
scanning tools (e.g., Nmap [36], MacStealer [32]). If it is
disabled, the attacker records the scanning results of potential Then, the attacker impersonates the server and sends a
victimclientsforthefuturalTCPinjectionattack.Notethatthe forged SYN/ACK packet whose destination is the router’s
attacker does not need to know the specific private IP address external IP address and whose destination port is the guessed
of the victim client (i.e., which IP is the victim), as we will source port n. According to RFC 3704 recommendation, the
show that it only needs to send related packets to the router, reverse path of packets received should be strictly checked so
theserver,oralloftheclientsinthesubsequentattackphases, astopreventIPspoofingattacks.Inthiscase,astheSYN/ACK
which is different from previous works that they will choose packet is received from the router’s internal interface while its
a target victim client beforehand, i.e., identifying whether a source address is a public IP address and it actually cannot
5

|     |        |                             |     |              |     |        | attempt        | to obtain       | the            | current   | sequence | number      | SEQ           | and the |
| --- | ------ | --------------------------- | --- | ------------ | --- | ------ | -------------- | --------------- | -------------- | --------- | -------- | ----------- | ------------- | ------- |
|     |        |                             |     |              |     |        | acknowledgment |                 | number         | ACK       | of       | the server. | Note          | that as |
|     |        |                             |     |              |     |        | TCP is         | a bidirectional |                | symmetric |          | full-duplex | protocol,     | the     |
|     | Victim | Attacker                    |     | Router       |     | Server |                |                 |                |           |          |             |               |         |
|     |        |                             |     |              |     |        | sequence       | and             | acknowledgment |           | numbers  |             | of the victim | client  |
|     |        | TCP connection src.port= m  |     | NAT mappings |     |        |                |                 |                |           |          |             |               |         |
TCP are symmetrical to the server. Previous works mostly infer
Traffic victim:m    wan_ip:m these values by exploring the entire possible 4G space via
src.ip=attacker
……
src.port=m leveraging some side channels [10], [6], [7], [51]), which are
|     |     | ttl=2 |     | Collide and create a new mapping |     |     |     |     |     |     |     |     |     |     |
| --- | --- | ----- | --- | -------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
SYN rather time-consuming and largely impact the success rate
src=server victim:m    wan_ip:m of hijacking the short-period TCP connections. However, in
C o rr e c t
d s t .i p = w a n_ip attacker:m wan_ip:m’ this work, we demonstrate a new method to obtain these two
| So  | u r c e   | d s t. p o rt = m | SYN/ACK |     |     |     |     |     |     |     |     |     |     |     |
| --- | --------- | ----------------- | ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Port values directly and precisely, which abuses vulnerable routers
DNAT and
src=server forward it to the victim
dst.ip=victim without TCP window tracking. We assume that there will be
SYN/ACK dst.port=m some intervals in the communication between the client and
|     |     |     |     |     |     |     | the server. | Depending |     | on the | scenarios, | the        | client | periodically |
| --- | --- | --- | --- | --- | --- | --- | ----------- | --------- | --- | ------ | ---------- | ---------- | ------ | ------------ |
|     |     |     |     |     |     |     | initiates   | a request | and | waits  | for        | responses, | or     | the server   |
src.ip=attacker
|     |     |     |     |     |     |     | proactively | pushes | notification |     | messages, |     | which are | often the |
| --- | --- | --- | --- | --- | --- | --- | ----------- | ------ | ------------ | --- | --------- | --- | --------- | --------- |
src.port=n
|     |     | ttl=2      | SYN | Directly create a new mapping |          |     | cases in | real-world | services. |     |     |     |     |     |
| --- | --- | ---------- | --- | ----------------------------- | -------- | --- | -------- | ---------- | --------- | --- | --- | --- | --- | --- |
|     |     | src=server |     | victim:m                      | wan_ip:m |     |          |            |           |     |     |     |     |     |
I n c o r r e ct  d s t .i p = w a n_ip Fi g u re 3 s h ow s ou r m e t ho d f o r t h e a t ta c k er to h i ja c k t h e
| S o u | r c e   |     |     | attacker:n | wan_ip:n |     |     |     |     |     |     |     |     |     |
| ----- | ------- | --- | --- | ---------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
d s t. p o rt = n SYN/ACK TCP c o n ne cti o n b etw e en t h e v ic t im c l ie n t a n d se rve r . F i rst l y ,
P o r t
DNAT and
src=server forward it to the attacker the attacker cleans the router’s NAT mapping of the victim
dst.ip=victim
connectionbysendingspoofedTCPRSTpacketswhosesource
|     |     | SSYYNN//AACCKK | dst.port=n |     |     |     |               |     |             |     |           |          |          |         |
| --- | --- | -------------- | ---------- | --- | --- | --- | ------------- | --- | ----------- | --- | --------- | -------- | -------- | ------- |
|     |     |                |            |     |     |     | is the server | and | destination |     | IP is the | router’s | external | IP, and |
destinationportisthepreviouslyinferredportm.Thesequence
Fig.2. InferringthesourceportofthevictimTCPconnection numbers specified in these packets are crafted for various
|     |     |     |     |     |     |     | brands and | types   | of routers |     | due to their | different |            | behaviors of |
| --- | --- | --- | --- | --- | --- | --- | ---------- | ------- | ---------- | --- | ------------ | --------- | ---------- | ------------ |
|     |     |     |     |     |     |     | disabling  | the TCP | window     |     | tracking     | strategy. | Generally, | there        |
be routed from this incoming interface for responding to the are two popular behaviors as stated in Section II-B that the
packet,thenitshouldbedroppedbytherouter.However,many first type of router does not check the sequence number at
routersintherealworlddonotadopttheRFCrecommendation all, and the second type of router will check if the sequence
and will not check the reverse path of packets received. So number is beyond the exact sequence number in a 2G space.
| when | the forged | packet | arrives, | it will | be processed | by the |     |     |     |     |     |     |     |     |
| ---- | ---------- | ------ | -------- | ------- | ------------ | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
Forthefirsttypeofrouter,theattackercanspecifyanarbitrary
router’s kernel and forwarded according to NAT mappings. sequence number in one crafted RST packet to clean the TCP
Since there is a NAT mapping that translates the router’s NAT mapping at the router directly. And for the second type
external IP to the attacker’s private IP when the source is the of router the attacker can send two RST packets, one with
remote server and the destination port is n, the forged packet sequence number x and the other with (x + 2G) % 4G, which
| will | match | this mapping | and be | forwarded | to the | attacker. In |         |          |         |      |             |     |              |        |
| ---- | ----- | ------------ | ------ | --------- | ------ | ------------ | ------- | -------- | ------- | ---- | ----------- | --- | ------------ | ------ |
|      |       |              |        |           |        |              | ensures | that one | of them | will | fall within |     | the required | range. |
thisway,theattackercanreceivetheforgedpacketthatissent
| from | itself          | again if the guessed |         | source | port n is not | equal to |     |     |     |     |     |     |     |     |
| ---- | --------------- | -------------------- | ------- | ------ | ------------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
| the  | victim client’s | source               | port m. |        |               |          |     |     |     |     |     |     |     |     |
…
|     | If the attacker | guesses | the right | source | port, i.e., | m, when |          |          |     |          |     |        |     |        |
| --- | --------------- | ------- | --------- | ------ | ----------- | ------- | -------- | -------- | --- | -------- | --- | ------ | --- | ------ |
|     |                 |         |           |        |             |         | Client_n | Client_1 |     | Attacker |     | Router |     | Server |
theSYNpacketarrivesattherouter,itwilltranslatethesource
|     |     |     |     |     |     |     | TCP | A victim constructs a TCP connection with src.port= m  |     |     |     |     | NAT mappings |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------ | --- | --- | --- | --- | ------------ | --- |
port of the new mapping to another port due to the collision. Traffic src=server victim:m    wan_ip:m
|     |     |     |     |     |     |     |     |     |     | dst.ip=wan_ip |     |     | …... |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | --- | ---- | --- |
Let us say that the changed source port is m’. In the second dst.port=m
step,whentheforgedSYN/ACKarrivesattherouter,however, seq=rand RST Check and delete the mapping
1
itwillbeforwardedtothevictimaccordingtotheclient’sNAT s r c . ip = a t tacker
|     |     |     |     |     |     |     |     |     |     | s r c | .p o rt = m |     | …... |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ----- | ----------- | --- | ---- | --- |
mapping as the port specified in it is m instead of m’. Thus, Obtain PUSH/ACK Create a new mapping
seq=arbitrary
from the view of the attacker, it cannot receive the forged SEQ/ 2
|     |     |     |     |     |     |     | ACK |     |     |     |     |     | attacker:m   wan_ip:m |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------------- | --- |
…...
SYN/ACK packet again if the port it guesses is right, i.e., with the exact
ACK
previously occupied. In this way, the attacker can infer that SEQ and ACK
there is a connection from some local host to the target server src.ip=server
dst.ip=wan_ip
| with | the source | port m. |     |     |     |     |     |     |     | dst.port=m |     |     |     |     |
| ---- | ---------- | ------- | --- | --- | --- | --- | --- | --- | --- | ---------- | --- | --- | --- | --- |
RST
|     |     |     |     |     |     |     |     |     |     | seq=rand |     |     | Check and delete the mapping |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | --- | ---------------------------- | --- |
3
The attacker repeats the above procedure, i.e., changing ACK … src=server
|     |     |     |     |     |     |     |     |     | dst.ip=client_i |     |     |     | …... |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --------------- | --- | --- | --- | ---- | --- |
the guessed source port number specified in the forged SYN Restore ACK dst.port=m
|     |         |         |          |           |           |         | Mapping |     |     | 4   |     |     |     |     |
| --- | ------- | ------- | -------- | --------- | --------- | ------- | ------- | --- | --- | --- | --- | --- | --- | --- |
| and | SYN/ACK | packets | and then | observing | if it can | receive |         |     |     |     |     |     |     |     |
src.ip=victim
SYN/ACK back until the correct port m is identified, which src.port=m ACK Restore the original mapping
SYN/ACK
will be used for the subsequent attacks. victim:m    wan_ip:m
…...
D. Phase 3: Hijacking Active Connections Fig.3. Hijackingactiveconnections
OncetheattackerhasdeterminedanactiveTCPconnection
to a given remote server with the source port m, it will To simplify the description, we will only elaborate on the
6

details of the attack on the first type of router. After receiving in the network probing phase (see Section IV-B) when AP
| RST |     |     |     |     |     |     |     |     |     |     |     |     | ACK |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
the packet, the router will falsely process the packet as isolation is disabled. The source of the packets is the
it does not perform reverse path validation, and the state of server, and the destination port is m. After arriving at the
therecordedTCPmappingwilltransferfromESTABLISHED irrelevanthostswithnocorrespondingconnections,thepackets
to CLOSE. The CLOSE state will last for a timeout according will be dropped. Yet the victim will send an ACK packet back
to a kernel variable (e.g., ip conntrack tcp timeout close in to the server, which restores the NAT mapping of the victim
OpenWrt-based systems). In our empirical investigation, 41% whenittravelsthroughtherouter.Thentheattackercaninject
of the tested routers set this value to as short as 1 second, forged responses to the client by sending them to the external
and the other vulnerable routers set it to 10 seconds. After the IPoftherouterviaabusingthedisabledreversepathvalidation
countdown of the timeout, the original TCP NAT mapping at and NAT mappings (the same as it did when inferring the
the router will be completely cleared. sourceport).However,theattackercannotrestorethemapping
ifAPisolationisenabledwithinthenetwork,asitcannotsend
| Secondly, |              | after the | original | NAT        | mapping | has             | been fully |         |                   |     |      |             |              |     |     |
| --------- | ------------ | --------- | -------- | ---------- | ------- | --------------- | ---------- | ------- | ----------------- | --- | ---- | ----------- | ------------ | --- | --- |
|           |              |           |          |            |         |                 |            | packets | to other clients, | and | thus | this attack | is thwarted. |     |     |
| evicted,  | the attacker | replaces  |          | the victim | client  | by constructing |            |         |                   |     |      |             |              |     |     |
a new mapping at the router via sending a forged data packet Note that in the above attack phases, there are timeouts in
PUSH/ACK m which the NAT mappings are in the state of CLOSE. We have
| with |     | flags | to the | remote | server, | using | the port |     |     |     |     |     |     |     |     |
| ---- | --- | ----- | ------ | ------ | ------- | ----- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
as the source port and its private IP as the source IP address. foundthatifrelatedpacketsaretravelingthroughtherouterat
The sequence and acknowledgment numbers specified in the this period, the countdown will be refreshed, which may lead
packet can be arbitrary as the router does not verify them. to interference with the attack. We will discuss the influence
With the translation of IP addresses at the router, the packet in detail in Section VI-A.
willberoutedtotheserver.Fromtheperspectiveoftheremote
server, this packet is from the public IP address of the client V. EMPIRICALSTUDY
| with source | port | m and | it will | match | the | victim connection. |     |     |     |     |     |     |     |     |     |
| ----------- | ---- | ----- | ------- | ----- | --- | ------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Since the sequence and acknowledgment numbers specified in Weconductextensivereal-worldevaluationstomeasurethe
the packet are wrong, an ACK packet with the server’s exact impacts of the attack. We first investigate the default settings
sequenceandacknowledgmentnumberswillbetriggeredback of routers on the market. Next, we conduct case studies to
|                                                      |     |     |        |         |        |         |            | evaluate        | the effectiveness |     | of the | attack in | various | real-world |     |
| ---------------------------------------------------- | --- | --- | ------ | ------- | ------ | ------- | ---------- | --------------- | ----------------- | --- | ------ | --------- | ------- | ---------- | --- |
| [42]. When                                           | the | ACK | packet | arrives | at the | router, | it will be |                 |                   |     |        |           |         |            |     |
| translatedandroutedtotheattackeraccordingtothenewNAT |     |     |        |         |        |         |            | Wi-Fi networks. |                   |     |        |           |         |            |     |
mappingcreatedjustnow.Thenitcanobtainthesequenceand
|                |           |         |        |              |            |            |           | Ethical | Considerations. |           | As it       | is essential | to            | respect    | the   |
| -------------- | --------- | ------- | ------ | ------------ | ---------- | ---------- | --------- | ------- | --------------- | --------- | ----------- | ------------ | ------------- | ---------- | ----- |
| acknowledgment |           | numbers | of     | the victim   | connection |            | directly. |         |                 |           |             |              |               |            |       |
|                |           |         |        |              |            |            |           | privacy | and security    | of others | when        | engaging     | in            | authorized |       |
|                |           |         |        |              |            |            |           | hacking | experiments,    | our       | experiments |              | in real-world |            | Wi-Fi |
| After          | the above | two     | steps, | the attacker |            | can decide | on the    |         |                 |           |             |              |               |            |       |
follow-up procedures according to the purpose of the attack. networks require careful consideration of ethical issues. We
In this work, we will illustrate three types of possible attacks. addressed the ethical issues of our real-world experiments
(i) TCP DoS attack. If the attacker intends to forcibly close from the following perspectives. First, we provided the Wi-Fi
|                |     |        |          |              |     |         |            | network | administrators | with | detailed | explanations |     | of our | ex- |
| -------------- | --- | ------ | -------- | ------------ | --- | ------- | ---------- | ------- | -------------- | ---- | -------- | ------------ | --- | ------ | --- |
| the connection |     | in the | scenario | of encrypted |     | tunnels | (e.g., SSH |         |                |      |          |              |     |        |     |
perimentalplansandobtainedtheirapprovalbeforeconducting
| or HTTPS), | it  | can just | send | forged | TCP | RST packets | to the |     |     |     |     |     |     |     |     |
| ---------- | --- | -------- | ---- | ------ | --- | ----------- | ------ | --- | --- | --- | --- | --- | --- | --- | --- |
server with the information obtained before, thus causing the the experiments. Second, with the help of the administrator,
connection terminated at the server side. After that, the client we ensured that no other users were accessing the Wi-Fi
will not receive any response when it sends requests to the network during our experiments, thus avoiding potential risks
|           |         |          |              |         |         |           |             | or side effects | for             | other users. | We       | then deployed |        | our machine |     |
| --------- | ------- | -------- | ------------ | ------- | ------- | --------- | ----------- | --------------- | --------------- | ------------ | -------- | ------------- | ------ | ----------- | --- |
| server,   | which   | leads to | a denial     | of      | service | attack.   | (ii) TCP    |                 |                 |              |          |               |        |             |     |
|           |         |          |              |         |         |           |             | (a laptop       | or a cellphone) |              | as the   | victim        | client | in the      | Wi- |
| hijacking | attack. | If       | the attacker | intends |         | to hijack | the traffic |                 |                 |              |          |               |        |             |     |
|           |         |          |              |         |         |           |             | Fi network,     | thus ensuring   |              | that all | the machines  |        | involved    | in  |
| from the  | server  | to the   | victim       | client, | it      | can take  | over the    |                 |                 |              |          |               |        |             |     |
NAT mapping and impersonate the client again with the exact our experiments were under our control and would not affect
sequence and acknowledgment numbers to launch requests to other machines. Finally, after completing the experiments,
|     |     |     |     |     |     |     |     | we provided | feedback | on  | the results | to  | the administrator. |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ----------- | -------- | --- | ----------- | --- | ------------------ | --- | --- |
theserverandwaitforresponsesfromtheserver.Forinstance,
|                   |     |     |        |               |     |              |        | Moreover, | we recommended |            | that | they restart | the       | Wi-Fi  | router |
| ----------------- | --- | --- | ------ | ------------- | --- | ------------ | ------ | --------- | -------------- | ---------- | ---- | ------------ | --------- | ------ | ------ |
| at the beginning, |     | the | victim | client logins |     | into the FTP | server |           |                |            |      |              |           |        |        |
|                   |     |     |        |               |     |              |        | and clear | the cache      | to restore | the  | network      | to a safe | state. |        |
andrequestspersonalfilesfromtheserver.Aftertheattack,the
| attacker | can bypass | the | initial | verification |     | stage by | replacing |     |     |     |     |     |     |     |     |
| -------- | ---------- | --- | ------- | ------------ | --- | -------- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
thevictimclienttosendrequeststotheserver,whichmaylead A. Analysis of Routers
| to permission |        | bypass   | and privacy | leakage. |        | (iii) TCP | injection |          |                  |                |            |               |     |     |         |
| ------------- | ------ | -------- | ----------- | -------- | ------ | --------- | --------- | -------- | ---------------- | -------------- | ---------- | ------------- | --- | --- | ------- |
|               |        |          |             |          |        |           |           | The      | attack leverages | the            | strategies | adopted       | by  | the | router, |
| attack.       | If the | attacker | intends     | to send  | forged | responses | by        |          |                  |                |            |               |     |     |         |
|               |        |          |             |          |        |           |           | and only | if all of        | the conditions |            | are fulfilled | can | our | attack  |
impersonatingtheserverwhenthevictimclientinitiatesanew
succeed.Inordertoexplorethecoverageofvulnerablerouters,
| request | later, it | needs | to restore | the | original | NAT mapping | of  |     |     |     |     |     |     |     |     |
| ------- | --------- | ----- | ---------- | --- | -------- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
the victim client at the router so as not to interfere with the we perform tests on real router models from lots of vendors,
|          |        |                |     |     |           |              |     | including | 360, Aruba, | ASUS, | Amazon, |     | Cisco Meraki, |     | China |
| -------- | ------ | -------------- | --- | --- | --------- | ------------ | --- | --------- | ----------- | ----- | ------- | --- | ------------- | --- | ----- |
| client’s | normal | communication. |     | We  | are going | to elaborate | on  |           |             |       |         |     |               |     |       |
Mobile,Comfast,D-Link,GL.iNet,Google,H3C,Huawei,IP-
| this case | as shown | in  | the last | two steps | in  | Figure 3. |     |             |          |          |     |          |          |          |     |
| --------- | -------- | --- | -------- | --------- | --- | --------- | --- | ----------- | -------- | -------- | --- | -------- | -------- | -------- | --- |
|           |          |     |          |           |     |           |     | COM, iKuai, | JdCloud, | Linksys, |     | Mercury, | Netgear, | Netcore, |     |
The attacker repeats the first step to clean the mapping of Ruijie, Skyworth, Tenda, TP-Link, Ubiquiti, Volans, Wavlink,
CLOSE
itself and waits for another NAT mapping timeout of WiMaster, Xiaomi, and ZTE. To our best knowledge, the
state. To restore the original mapping of the victim, as the operatingsystemsofmostrouterswetestedarebasedonLinux
attacker does not know the victim client’s exact private IP, it with custom modifications, except for some router models
can send forged ACK packets to all of the local hosts probed from TP-Link and Mercury, which are based on VxWorks.
7

Therefore, we also build a soft routing environment with a B. Attack Evaluation
FreeBSD-based firmware, i.e., pfSense 2.7.0 [40]. In total,
To evaluate the impacts of the attack in the real world, we
we perform tests on 67 mainstream router models (acting as
also conduct thorough experiments of the attack in 93 various
the gateway to provide Internet services) from 30 vendors.
Wi-Fi networks. We investigate whether the conditions of the
For each router model, we test if it fits all attack conditions
attackarefulfilledineachnetworkbytakingthreecasestudies
proposed in Section III. Here we list the detailed test results
ofattacksonSSH,FTP,andHTTPapplicationsandmeasuring
of 33 representative routers from these 30 vendors in Table I.
the time cost and success rate of each attack.
First, the router has to take the port preservation strategy. Experimental Setup. Our experiments consist of four types
In our test, most of the routers adopt this strategy by default. of devices, i.e., a router, a victim client, a remote server, and
Only the enterprise wired router model “AR6140E-9G-2AC” an attacker.
producedbyHuaweiandthesoftroutingmachinewithpfSense
which co-work with a wireless AP to provide Wi-Fi service, • Router. The router in Wi-Fi networks works as the
take the random selection strategy that prevents the attacker gateway to provide Internet access and forward packets
frominferringthesourceportofclients’connections.Besides, between local clients and outside servers.
there is no router model which takes the sequential selection
or port overloading strategy as stated in Section II-A • Remote Server. For the DoS attack, we set up an
SSH server equipped with Ubuntu 22.04 (kernel ver-
Second, we investigate the deployment of the reverse path sion 5.15.0), OpenSSH 8.9, and OpenSSL 3.0.2. For
validationstrategyintherouters.Amongtheserouters,Netgear the hijacking attack, we set an FTP server equipped
andAsussetthekernelvariablerp filterto1bydefault,which with Ubuntu 22.04 (kernel version 5.15.0) and vsftpd
means they are secure to check the packet received strictly. version 3.0.3. And for the injection attack, we pick
And some of the old-styled models of TP-Link and Mercury a well-known finance website www.ANONYMOUS.com
(i.e., VxWorks-based) will also validate the received packets. (anonymizedforethicalconsideration)inwhichtheclient
However,theirnewestmodels(e.g.,designedforWi-Fi6)and initiates a long-lived TCP connection that periodically
some enterprise routers will not validate them anymore. In retrieves data updates every minute.
addition, ZTE routers, the router model “AR6140E-9G-2AC” • Victim Client. Though the OS type or version of the
from Huawei, and the soft routing machine with pfSense will client is unrestricted in our attack, we still deploy victim
alsonotforwardtheforgedpackets.Theroutersfromtheother clients equipped with five typical OSes (i.e., Windows,
vendorsalldisablereversepathvalidation,whichresultsinthe Linux, Mac, iOS, and Android). Each victim client has
vulnerability of inferring active connections. connected to the Wi-Fi network. In the case of DoS and
hijackingattacks,thevictimclientwillcommunicatewith
Third, the router has to disable the TCP window tracking the remote server through SSH and FTP. And in the case
strategy. In our test, only one enterprise wired router model of the injection attack, the victim client will access the
“Meraki 64” produced by Cisco Meraki will check the se- website above to get the newest future index of stocks
quence number strictly and all of the other routers disable (e.g., HSI, HSCEI).
TCP window tracking while the processing logic is slightly • Attacker. An attack machine is equipped with Linux
different. Asus, Google, Netgear, Tenda, Wimaster, and ZTE 5.15.0, which is capable of crafting packets. The attacker
routers will check if the sequence number is in the 2G space aims to forcibly close the victim client’s SSH connection
beyond the exact sequence number. And the other routers do with the remote server, steal private files from the FTP
not check the sequence number at all. server, or inject fake HTTP responses to the client by
performing the attack.
Fourth,thetimeoutofTCPCLOSEstateofNATmappings
Attack Procedure. The attacker first tries to get the router’s
may influence the time cost and success rate of our attack.
external IP address and test whether AP isolation is enabled
The shorter the timeout is, the easier the attack can succeed.
andotherhostscanbedetectedintheWi-Finetwork.Next,the
Among the 66 router models without TCP window tracking,
attack can be constructed in the following steps: (1) detecting
28 of them will clean the NAT mapping in only 1 second, 37
whether there is any TCP connection from the LAN to the
of them will be tricked to clean the mapping in 10 seconds,
given server, i.e., identifying the correct source port, (2) evict-
and the default setting of pfSense is 90 seconds.
ingtherouter’soriginalNATmappingoftheclientwithforged
RST packets and constructing a new one by sending a TCP
Duetothelimitedspace,thedetailedinformationofthe67
data packet to the server, which in turn incurs an ACK packet
tested routers is listed in Appendix B (see Table IV). We take
from it, (3) terminating the SSH connection or requesting an
the first row as an instance to analyze the results. The Linux-
FTP file download, (4) restoring the original NAT mapping
based router model “TL-XDR6020” produced by TP-Link,
of the client and answering the client’s HTTP requests with
provides the latest generation of Wi-Fi 6 for network services.
forged segments specified with the inferred values.
As for the four metrics mentioned above, this model takes the
port preservation strategy, does not validate the reverse path For the SSH DoS attack, we define the result that the
of received packets, disables the TCP window tracking, and client or the server cannot receive messages from each other
sets the TCP CLOSE timeout to 10 seconds by default. In this as a successful attack, which includes two cases. First, after
way, it is vulnerable to our attacks. In conclusion, 52 of the the attacker receives the ACK packet from the server, it can
67 tested routers are vulnerable, and 15 models are immune send a TCP RST packet with the exact sequence number to
to the attack as they do not fulfill all of the conditions. the server, resulting in the connection terminated at the server
8

|     |     |     |     |     | TABLEI. | PARTIALTESTEDROUTERSFROM30VENDORS |     |     |              |           |     |          |     |
| --- | --- | --- | --- | --- | ------- | --------------------------------- | --- | --- | ------------ | --------- | --- | -------- | --- |
|     |     |     |     |     |         |                                   |     |     | Reverse-path | TCPWindow |     | TCPClose |     |
Port
No. RouterModel Vendor OS Generation Validation Tracking Timeout Vulnerable
Preservation
|     |                   |     |             |     |                       |     |        |     | Disabled | Disabled |     | (second) |     |
| --- | ----------------- | --- | ----------- | --- | --------------------- | --- | ------ | --- | -------- | -------- | --- | -------- | --- |
|     | 1 TL-XDR6020      |     | TP-Link     |     | Linux-based           |     | Wi-Fi6 | ✔   | ✔        |          | ✔   | 1        | ✔   |
|     | 2 TL-WDR7620      |     | TP-Link     |     | Vxworks-based         |     | Wi-Fi5 | ✔   | ✘        |          | ✔   | 1        | ✘   |
|     | 3 AX3Pro          |     | Huawei      |     | EMUI(Linux-based)     |     | Wi-Fi6 | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     | AR6140E-9G-2AC∗   |     |             |     |                       |     |        | ✘   | ✘        |          | ✔   |          | ✘   |
|     | 4                 |     | Huawei      |     | VRP(Linux-based)      |     | -      |     |          |          |     | 10       |     |
|     |                   |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 5 V6G             |     | 360         |     | 360OS(Linux-based)    |     | Wi-Fi6 |     |          |          |     | 1        |     |
|     | 6 MagicR365       |     | H3C         |     | Comware(Linux-based)  |     | Wi-Fi5 | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     | 7 W30E            |     | Tenda       |     | Linux-based           |     | Wi-Fi6 | ✔   | ✔        |          | ✔   | 1        | ✔   |
|     | 8 RAX1800Z        |     | ChinaMobile |     | AOS(Linux-based)      |     | Wi-Fi6 | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     |                   |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 9 X32Pro          |     | Ruijie      |     | RGOS(Linux-based)     |     | Wi-Fi6 |     |          |          |     | 1        |     |
|     |                   |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 10 RedmiRA81      |     | Xiaomi      |     | MiWiFi(Linux-based)   |     | Wi-Fi6 |     |          |          |     | 1        |     |
|     | 11 MW300R         |     | Mercury     |     | Vxworks-based         |     | Wi-Fi4 | ✔   | ✘        |          | ✔   | 1        | ✘   |
|     | 12 X30G           |     | Mercury     |     | Linux-based           |     | Wi-Fi6 | ✔   | ✔        |          | ✔   | 1        | ✔   |
|     | 13 RAX50          |     | Netgear     |     | DumaOS(Linux-based)   |     | Wi-Fi6 | ✔   | ✘        |          | ✔   | 10       | ✘   |
|     | 14 RT-AX89X       |     | ASUS        |     | AsusWrt(Linux-based)  |     | Wi-Fi6 | ✔   | ✘        |          | ✔   | 10       | ✘   |
|     |                   |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 15 E9450          |     | Linksys     |     | Linux-based           |     | Wi-Fi6 |     |          |          |     | 10       |     |
|     | 16 QUANTUMD2G     |     | Wavlink     |     | Linux-based           |     | Wi-Fi5 | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     | 17 CF-616AC       |     | Comfast     |     | OrangeOS(Linux-based) |     | Wi-Fi5 | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     | 18 DI-7003GV2∗    |     | D-Link      |     | Linux-based           |     | -      | ✔   | ✔        |          | ✔   | 1        | ✔   |
|     | 19 AX3000         |     | ZTE         |     | ZXR10ROS(Linux-based) |     | Wi-Fi6 | ✔   | ✘        |          | ✔   | 10       | ✘   |
|     | M80∗              |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 20                |     | IP-COM      |     | Linux-based           |     | -      |     |          |          |     | 1        |     |
|     |                   |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 21 SK-WR6640X     |     | Skyworth    |     | Linux-based           |     | Wi-Fi6 |     |          |          |     | 10       |     |
|     | 22 VE5200G∗       |     | Volans      |     | Linux-based           |     | -      | ✔   | ✔        |          | ✔   | 1        | ✔   |
|     | 23 NBR1009GPE     |     | Netcore     |     | NOS(Linux-based)      |     | -      | ✔   | ✔        |          | ✔   | 1        | ✔   |
|     | 24 Wimaster∗      |     | Wimaster    |     | Linux-based           |     | -      | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     |                   |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 25 IK-Enterprise∗ |     | iKuai       |     | iKuaiOS(Linux-based)  |     | -      |     |          |          |     | 10       |     |
|     |                   |     |             |     |                       |     |        | ✔   | ✘        |          | ✔   |          | ✘   |
|     | 26 InstantOnAP22  |     | Aruba       |     | ArubaOS(Linux-based)  |     | Wi-Fi6 |     |          |          |     | 10       |     |
|     | 27 EdgeRouterX∗   |     | Ubiquiti    |     | Linux-based           |     | -      | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     | 28 AX1800         |     | JdCloud     |     | Linux-based           |     | Wi-Fi6 | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     | 29 CiscoMeraki64∗ |     | CiscoMeraki |     | Linux-based           |     | -      | ✔   | ✘        |          | ✘   | -        | ✘   |
|     | 30 eeropro        |     | Amazon      |     | Linux-based           |     | Wi-Fi5 | ✔   | ✔        |          | ✔   | 10       | ✔   |
|     |                   |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 31 GoogleWi-Fi    |     | Google      |     | ChromeOS(Linux-based) |     | Wi-Fi5 |     |          |          |     | 10       |     |
|     |                   |     |             |     |                       |     |        | ✔   | ✔        |          | ✔   |          | ✔   |
|     | 32 GL-MT3000      |     | GL.iNet     |     | Linux-based           |     | Wi-Fi6 |     |          |          |     | 10       |     |
|     | 33 pfSense2.7.0∗  |     | pfSense     |     | FreeBSD-based         |     | -      | ✘   | ✘        |          | ✔   | 90       | ✘   |
✔meansthattherouterissatisfiedwiththecondition,and✘meansthattherouterisdissatisfiedwiththecondition.
✔meansthattherouterisvulnerabletoourattack,and✘meansthattherouterisimmunetoourattack.
*meansthatthemodelisanenterpriserouterwhichdoesnotsupportWi-Fibyitselfandneedstoworktogetherwithwirelessaccesspoints.
side. Second, as the attacker has replaced the NAT mapping on the client’s web page in 5 minutes. As mentioned before,
at the router, the source port of the packet will be translated it takes time (mostly 1 second or 10 seconds) for the mapping
if it happens that the client sends a packet to the server at this to disappear completely. The countdown will be refreshed
stage.Whenthepacketarrivesattheserver,itwillincuraRST if the client sends packets during this period, which may
packet as there is no corresponding connection, which will be interfere with the time cost and success rate of our attack.
routed to the client, resulting in the connection terminated at To simulate real-world situations, we require the tested client
the client side. In the context of the FTP hijacking attack, the to send requests to the server for random times, and we set
attackcanbedeemedsuccessfulwhentheattackermanagesto the interval between two requests as a random number from
download files from the FTP server that belong to the victim 5 to 30 seconds during the 5 minutes of an experiment. We
client. And for the HTTP injection attack, we define the result willfurtherinvestigatetheimpactsofcommunicationintervals
that the client receives forged packets, and the falsified data is betweentheclientandserverandthetimeoutofNATmappings
displayed on the web page as a successful attack. Compared in Section VI-A.
| with | SSH       | DoS and  | FTP hijacking | attacks, | the conditions |      | are |     |     |     |     |     |     |
| ---- | --------- | -------- | ------------- | -------- | -------------- | ---- | --- | --- | --- | --- | --- | --- | --- |
| more | difficult | to meet. | As the        | attacker | only knows     | that | the |     |     |     |     |     |     |
ExperimentalResults.Weevaluateourattackagainst93real-
requestintervalis60secondsforthewww.ANONYMOUS.com
|     |     |     |     |     |     |     | world | Wi-Fi | networks | to cover | the most | typical | Wi-Fi sce- |
| --- | --- | --- | --- | --- | --- | --- | ----- | ----- | -------- | -------- | -------- | ------- | ---------- |
website while it does not know when the client will request narios, e.g., Wi-Fi networks in coffee shops, hotels, shopping
an update. The client may request new data during the attack, malls,campuses,andofficebuildings.Astheattackercansniff
| which | results | in the | connection | being | terminated, | and | we  |     |     |     |     |     |     |
| ----- | ------- | ------ | ---------- | ----- | ----------- | --- | --- | --- | --- | --- | --- | --- | --- |
thenon-encryptedpacketsontheairinopennetworksdirectly,
| strictly | take | this case | as a failure. |     |     |     |     |               |      |               |       |                |           |
| -------- | ---- | --------- | ------------- | --- | --- | --- | --- | ------------- | ---- | ------------- | ----- | -------------- | --------- |
|          |      |           |               |     |     |     | we  | mainly launch | our  | experiments   | under | networks       | protected |
|          |      |           |               |     |     |     | by  | WPAs from     | home | mode networks |       | and enterprise | mode      |
We repeat the experiments 20 times in each tested Wi-Fi networks,e.g.,45withWPA2-Personalenabled(homemode),
network. Each experiment is conducted independently with a 22 with WPA2-Enterprise enabled (enterprise mode), and 26
renewed connection between the client and server. In order withWPA3-Personalenabled(homemode)andwedonotfind
to limit the time of experiments, we take an experiment anynetworkwithWPA3-Enterpriseenabled.Theexperimental
as a failure if the attacker cannot terminate the connection, results illustrate that more than 81% of the real-world Wi-Fi
download private files, or the forged data does not show up networks (i.e., 75 out of the 93 evaluated networks) are fully
9

vulnerablethattheysatisfyalloftheconditionsofourattacks.
For the other 18 Wi-Fi networks, 9 of them have AP isolation
enabled, which prevents the detection of potential victims and
thwarts the HTTP injection attack. However, the SSH DoS
and FTP hijacking attacks remain unaffected. Our attack fails
in7networksastheydonotusethevulnerablerouters,andwe
cannotgettherouter’sexternalIPasdescribedinSectionIV-B
in the rest two networks. We successfully acquire the external
IP addresses of routers using the route-recording method in
80 networks, involving router models from 22 vendors (i.e.,
Ubiquiti, Amazon, Google, Tenda, ASUS, Netgear, Huawei,
Linksys, Xiaomi, Ruijie, ZTE, H3C, Wavlink, Comfast, IP-
COM,Skyworth,Netcore,iKuai,WiMaster,GL.iNet,JdCloud,
and China Mobile). Additionally, we employ the scanning Fig.4. Snapshotsofwebpoisoning
methodin11networks,utilizingroutermodelsfrom6vendors
(i.e., D-Link, Volans, pfSense, and some models of 360,
Mercury, and TP-Link). In the remaining two networks using
and acknowledgment numbers, and the whole attack to inject
the router models from Cisco Meraki and Aruba, we fail to
forgeddatatothevictim’swebpageare9.4seconds,15.2sec-
gettheexternalIPaddressunlesswelogintothecontrolpage
onds, and 54.5 seconds, respectively, with an average success
of the router with the help of the network administrators, and
rateof76.1%.ComparedwiththeSSHDoSandFTPhijacking
we take these networks as failures.
attacks, it is more time-consuming as the attacker has to wait
TABLEII. EXPERIMENTALRESULTSINOURTESTS(ONAVERAGE). for more time to inject fake responses until the next request,
and it has a lower success rate as the connection may be
Attack Inferring Getting Finishing Total BW Success terminatediftheclientsendsarequestwhentheNATmapping
Type Port(s) SEQ/ACK(s)Attacking(s)Time(s)(pkts) Rate has been occupied by the attacker. Another scenario of failure
SSHDoS 8.1 8.4 1.0 17.5 4000 87.4% occurs when the attacker fails to win the condition race of
FTPHijacking 9.1 9.2 1.1 19.4 4000 82.6% returning responses with the server, i.e., the client accepts the
HTTPInjection 9.4 15.2 29.9 54.5 4000 76.1%
right data from the server. Figure 4 shows the snapshot of our
HTTP injection attack against www.ANONYMOUS.com. The
Next,weelaborateonourexperimentalresultsintheWi-Fi
original website shows that the HSI number is 19,319, and it
networks as shown in Table II. In the case of the SSH DoS
hasreducedby605withadroprateof3.04%.Aftertheattack,
attack, the average time cost of identifying the client’s source
the victim will find that the HSI number is 20,000 and it has
port is 8.1 seconds with a bandwidth of 4000 packets per
increasedby5000withagrowthrateof20%.Thesameistrue
second,whichismuchshorterthanpreviousmethods[10],[7],
for the data of HSCEI. The attack may lead to wrong stock
[6] as we only need to transfer packets in the same LAN and
purchase or sale, affecting the financial status of the victim.
we are not restricted by rate limits. And the average time cost
ofobtainingtheexactsequenceandacknowledgmentnumbers The experimental results of 30 Wi-Fi networks in our
is8.4seconds,asthisstepmainlyreliesonthedefaultsettings investigationsarelistedinTableIII.Asshowninthefirstrow,
of the timeout of CLOSE state in NAT mappings. Besides, an enterprise mode Wi-Fi network with the SSID of “Campus
the communication between the client and the server may 14” is located in a campus. The router is produced by the
also influence the time cost. Finally, the average time cost of vendor of Huawei, and it supports the generation of Wi-Fi 6.
totallyterminatinganSSHconnectionis17.5seconds,andthe We take the experiment of SSH DoS attack in this network,
average success rate is 87.4%. The failure cases in the tests and it takes the attacker 15.43 seconds to terminate an SSH
are due to continuous communications between the client and connection with a success rate of 90%.
the server (e.g., the client requests a file download and related
packetswillalwaysrefreshtheNATmapping).Aftertheattack VI. DISCUSSION
succeeds, the client’s SSH terminal will be stuck for a period
Inthissection,wediscussthefactorsthataffecttheattack’s
of time, which greatly affects the user experience.
effectiveness.Wealsocompareourattackwithexistingattacks
For the FTP hijacking attack, the average time costs of in WLANs. Besides, we extend our attack model to launch a
identifying the client’s source port and getting the sequence remote TCP DoS attack from an attacker on the Internet.
and acknowledgment numbers are 9.1 and 9.2 seconds, re-
spectively, which results in a time cost of 19.4 seconds for
A. Factors Impacting the Attack
the entire attack to get a private file from the server with a
success rate of 82.6% on average. The failure cases in the Impacts of Traffic Load. We analyze the impact of traffic
tests are due to two reasons. The first is the same as the cases load on the attacks from two aspects: 1) the bandwidth
in the SSH DoS attack, i.e., continuous communications. The between the client and the server, and 2) the communication
second is that the attacker happens to begin the attack when interval between the client and the server. First, we extend
the victim connection has been constructed while the victim the experiments of the FTP hijacking attack with varied
has not logged in. bandwidths (i.e., 10KBps, 100KBps, 1000KBps) and set the
As for the HTTP injection attack, the average time costs 4We anonymized the real SSIDs of the Wi-Fi networks due to ethical
of identifying the client’s source port, obtaining the sequence considerations.
10

|     |         | TABLEIII. |      | EXPERIMENTALRESULTSOFTCPATTACKSIN30WI-FINETWORKS. |        |       |        |        |     |      |         |     |
| --- | ------- | --------- | ---- | ------------------------------------------------- | ------ | ----- | ------ | ------ | --- | ---- | ------- | --- |
|     | Network |           |      |                                                   | Router | Wi-Fi | WPA2/3 | Attack |     | Time | Success |     |
| No. |         |           | SSID |                                                   |        |       |        |        |     |      |         |     |
Mode Vendor Generation Enterprise/Personal Result Cost(s) Rate
1 Enterprisemode Campus1 Huawei Wi-Fi6 WPA2-Enterprise SSHDoS 15.43 18/20
2 Enterprisemode Campus2 TP-Link Wi-Fi4 WPA2-Enterprise FTPHijacking 10.32 18/20
3 Enterprisemode Campus3 H3C Wi-Fi6 WPA2-Enterprise HTTPInjection 48.87 15/20
4 Enterprisemode Enterprise1 TP-Link Wi-Fi6 WPA2-Enterprise SSHDoS 11.56 16/20
5 Enterprisemode Enterprise2 TP-Link Wi-Fi5 WPA2-Enterprise FTPHijacking 11.43 18/20
6 Enterprisemode Enterprise3 Netcore Wi-Fi6 WPA2-Enterprise HTTPInjection 87.20 15/20
7 Enterprisemode Officebuilding1 TP-Link Wi-Fi5 WPA2-Enterprise SSHDoS 9.56 18/20
8 Enterprisemode Officebuilding2 iKuai Wi-Fi6 WPA2-Enterprise FTPHijacking 21.46 17/20
9 Enterprisemode Officebuilding3 Mercury Wi-Fi6 WPA2-Enterprise HTTPInjection 31.14 15/20
10 Enterprisemode Hotel1 Netcore Wi-Fi5 WPA2-Enterprise SSHDoS 15.75 18/20
11 Enterprisemode Hotel2 D-Link Wi-Fi6 WPA2-Enterprise FTPHijacking 9.45 19/20
12 Enterprisemode Hotel2 iKuai Wi-Fi6 WPA2-Enterprise HTTPInjection 71.32 16/20
13 Homemode Restaurant1 TP-Link Wi-Fi5 WPA2-Personal SSHDoS 8.95 17/20
14 Homemode Restaurant2 Comfast Wi-Fi5 WPA2-Personal FTPHijacking 21.56 18/20
15 Homemode Restaurant3 Skyworth Wi-Fi6 WPA2-Personal HTTPInjection 62.35 13/20
16 Homemode Coffeeshop1 Mercury Wi-Fi4 WPA2-Personal SSHDoS 8.98 17/20
17 Homemode Coffeeshop2 TP-Link Wi-Fi4 WPA2-Personal FTPHijacking 9.29 18/20
18 Homemode Coffeeshop3 Wavlink Wi-Fi5 WPA2-Personal HTTPInjection 45.22 13/20
19 Homemode Shoppingmall1 Tenda Wi-Fi6 WPA3-Personal SSHDoS 24.23 18/20
20 Homemode Shoppingmall2 TP-Link Wi-Fi4 WPA2-Personal FTPHijacking 11.44 19/20
21 Homemode Shoppingmall3 Huawei Wi-Fi6 WPA3-Personal HTTPInjection 78.44 15/20
22 Homemode Bookstore1 360 Wi-Fi5 WPA2-Personal SSHDoS 19.45 18/20
23 Homemode Bookstore2 Xiaomi Wi-Fi6 WPA3-Personal FTPHijacking 10.61 18/20
24 Homemode Bookstore3 H3C Wi-Fi6 WPA3-Personal HTTPInjection 56.12 14/20
25 Homemode Experiencestore1 Xiaomi Wi-Fi6 WPA3-Personal SSHDoS 16.97 17/20
26 Homemode Experiencestore2 Huawei Wi-Fi6 WPA3-Personal FTPHijacking 23.98 18/20
27 Homemode Experiencestore3 Xiaomi Wi-Fi5 WPA2-Personal HTTPInjection 52.14 16/20
28 Homemode Cinema1 Ruijie Wi-Fi5 WPA2-Personal SSHDoS 8.89 19/20
29 Homemode Cinema2 Mercury Wi-Fi6 WPA3-Personal FTPHijacking 11.31 18/20
30 Homemode Cinema2 Huawei Wi-Fi6 WPA3-Personal HTTPInjection 54.26 16/20
communication interval to 16 seconds with a NAT mapping shown in Figure 5(b)), if the communication interval is below
timeout of 10 seconds. We repeat the experiment 50 times 10 seconds, the attack will fail due to the same reason. When
for each bandwidth and record the time cost and the number the interval is above 10 seconds, the attack can succeed with
of successful attacks. The experimental results show that the a high success rate (97.5%). The average time cost shows a
average time costs (i.e., 29.46 seconds, 28.78 seconds, 29.21 downwardtrendwiththeincrementofcommunicationinterval,
seconds) and success rates (i.e., 96%, 94%, 94%) remain as the attacker can try fewer times and wait less time for the
largely unaffected since our attack mainly relies on the time NATmappingstobecleanedwhenthecommunicationinterval
interval left for the attacker to clean the NAT mappings. is longer in an attack.
Second, we evaluate the three attacks under various com- Distribution of Time Cost and Failures. Besides evaluating
munication intervals (e.g., 2 seconds, 4 seconds, 6 seconds, the FTP hijacking attack, we also measure the time costs and
etc.) between the client and server, with a bandwidth of failure reasons of the other two attacks, i.e., the SSH DoS
100KBps. We repeat the experiments 20 times for each com- attackandtheHTTPinjectionattack.FortheSSHDoSattack,
munication interval and record the time used in each attack as shown in Figure 5(a), when the NAT mapping timeout is 1
phase and count the successful attacks. The experimental second, if the communication interval is below 1 second, the
results are shown in Figure 5. attack will fail. However, when the communication interval
|          |                 |               |     |             |                | is larger | than 1 second,   | the  | attack   | can always | succeed.      | The |
| -------- | --------------- | ------------- | --- | ----------- | -------------- | --------- | ---------------- | ---- | -------- | ---------- | ------------- | --- |
| Here,    | we take the     | FTP hijacking |     | attack      | as an example. |           |                  |      |          |            |               |     |
|          |                 |               |     |             |                | average   | time cost is not | very | impacted | by the     | communication |     |
| As shown | in Figure 5(a), | when          | the | NAT mapping | timeout        |           |                  |      |          |            |               |     |
interval,anditshowsaslightfluctuatingtrendwithanaverage
| is set to 1   | second, if the | communication     |        | interval     | is below  | 1            |                  |          |      |             |         |           |
| ------------- | -------------- | ----------------- | ------ | ------------ | --------- | ------------ | ---------------- | -------- | ---- | ----------- | ------- | --------- |
|               |                |                   |        |              |           | value        | of 9.24 seconds. | The main | cost | is incurred | by      | inferring |
| second, the   | attack will    | fail due          | to the | continuously | refreshed |              |                  |          |      |             |         |           |
|               |                |                   |        |              |           | the client’s | source ports.    | Figure   | 5(b) | shows       | similar | results   |
| NAT mappings. | When           | the communication |        | interval     | is        | above        |                  |          |      |             |         |           |
|               |                |                   |        |              |           | when         | the NAT mapping  | timeout  | is   | 10 seconds. |         |           |
| 1 second,     | the attack can | succeed           | with   | a high       | success   | rate         |                  |          |      |             |         |           |
(97.67%), where the small partial failures are due to the As for the HTTP injection attack, as shown in Figure
attacks being launched during the login phase of the FTP 5(a), when the NAT mapping timeout is 1 second, if the
application. The average time cost is less affected by the communication interval is below 2 seconds, the attack will
communicationinterval,anditshowsafluctuatingtrend,which fail due to the continuously refreshed NAT mappings, or the
mainly depends on the time to infer the client’s source port. connection will be terminated during the reconstruction of the
Similarly, when the NAT mapping timeout is 10 seconds (as original NAT mapping as stated in Section V-B. However,
11

|     |                |     | Time of Inferring Source Port | Time of Fininshing Attacking |        |                       |     |     |
| --- | -------------- | --- | ----------------------------- | ---------------------------- | ------ | --------------------- | --- | --- |
|     |                |     | Time of Getting Seq/Ack       | Success Rate                 |        |                       |     |     |
|     | SSH DoS Attack |     |                               | FTP Hijacking Attack         |        | HTTP Injection Attack |     |     |
|     |                |     | 100 12                        |                              | 100 50 |                       |     | 100 |
10
|     |     |     | 80 10 |     | 80 40 |     |     | 80  |
| --- | --- | --- | ----- | --- | ----- | --- | --- | --- |
8
| )dnoces( emiT |     |     |     |     |     |     |     | )%( etaR sseccuS |
| ------------- | --- | --- | --- | --- | --- | --- | --- | ---------------- |
8
|     |     |     | 60  |     | 60 30 |     |     | 60  |
| --- | --- | --- | --- | --- | ----- | --- | --- | --- |
6
6
|     |     |     | 40  |     | 40 20 |     |     | 40  |
| --- | --- | --- | --- | --- | ----- | --- | --- | --- |
4
4
| 2   |     |     | 20 2 |     | 20 10 |     |     | 20  |
| --- | --- | --- | ---- | --- | ----- | --- | --- | --- |
| 0   |     |     | 0 0  |     | 0 0   |     |     | 0   |
2 4 6 8 1012141618202224262830 2 4 6 8 1012141618202224262830 2 4 6 8 1015202530354045505560
Communication Interval (second)
(a) ThetimecostsandsuccessratesindifferentcommunicationintervalswhenthetimeoutoftheNATmappingis1second.
|     | SSH DoS Attack |     |     | FTP Hijacking Attack |     | HTTP Injection Attack |     |     |
| --- | -------------- | --- | --- | -------------------- | --- | --------------------- | --- | --- |
|     |                |     | 100 |                      | 100 |                       |     | 100 |
60
| 40  |     |     | 40  |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|     |     |     | 80  |     | 80  |     |     | 80  |
50
)%( etaR sseccuS
| )dnoces( emiT |     |     | 30  |     |       |     |     |     |
| ------------- | --- | --- | --- | --- | ----- | --- | --- | --- |
| 30            |     |     | 60  |     | 60 40 |     |     | 60  |
30
| 20  |     |     | 20  |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|     |     |     | 40  |     | 40  |     |     | 40  |
20
| 10  |     |     | 20 10 |     | 20  |     |     | 20  |
| --- | --- | --- | ----- | --- | --- | --- | --- | --- |
10
| 0   |     |     | 0 0 |     | 0 0 |     |     | 0   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
2 4 6 8 1012141618202224262830 2 4 6 8 1012141618202224262830 2 4 6 8 101520212530354045505560
Communication Interval (second)
(b) ThetimecostsandsuccessratesindifferentcommunicationintervalswhenthetimeoutoftheNATmappingis10seconds.
Fig.5. ThetimecostsandsuccessratesindifferentcommunicationintervalsanddifferentNATmappingtimeouts.
whenthecommunicationintervalislargerthan2seconds,with probing overhead. For example, compared to the cases with a
the increase of the communication interval, the time cost and 1-second timeout, when the timeout is 10 seconds, the time
successratetendtoincrease.Thetimecostmainlydependson to obtain sequence and acknowledgment numbers takes up a
thewaitingperioduntiltheclientsendsarequest,asinjections largerproportionastheattackermustwaitforthemappingsto
before that will not be accepted. In this way, the longer the be completely cleared (i.e., at least 10 seconds). Under a long
communication interval is, the more waiting time will cost on timeout, the connection is more likely to be unintentionally
average. The failures mainly come from two aspects. First, terminated, and thus the success rate is reduced. In summary,
the original connection may also be terminated. Second, the the increase of NAT mapping timeouts will incur increased
attacker should inject data before the response of the server time costs and decreased success rates.
| when the | victim client sends | a new | request. Similar | results |     |     |     |     |
| -------- | ------------------- | ----- | ---------------- | ------- | --- | --- | --- | --- |
can be found in Figure 5(b) when the NAT mapping timeout Impacts of AP Isolation. AP isolation may influence some
| is 10 seconds.     | The difference | is that  | the time to get sequence |              |                  |            |                 |          |
| ------------------ | -------------- | -------- | ------------------------ | ------------ | ---------------- | ---------- | --------------- | -------- |
|                    |                |          |                          | phases of    | our attack. With | the policy | enabled, the    | attacker |
| and acknowledgment | numbers        | occupies | a larger proportion      |              |                  |            |                 |          |
|                    |                |          |                          | cannot probe | potential victim | clients in | the first phase | of the   |
compared with that of 1 second. attack, and it cannot reconstruct the original NAT mapping at
|     |     |     |     | the router | by sending spoofed | TCP ACK | packets to | all of the |
| --- | --- | --- | --- | ---------- | ------------------ | ------- | ---------- | ---------- |
Impacts of NAT Mapping Timeout. We analyze the impact potential victim clients when launching the HTTP injection
ofNATmappingtimeoutbycomparingtheexperimentalresult attack. However, the SSH DoS and FTP hijacking attacks are
of the HTTP injection attack when we set the communication not affected as the attacker does not need to send packets
intervalbetweenthevictimclientandtheserverto60seconds, directlytothevictimclient.Besides,onlylessthan10%(9out
reflecting an actual client-server communication scenario in of 93) of real-world Wi-Fi networks we observed enforce AP
the real world. When setting the NAT mapping timeout to 1 isolation. We also enabled AP isolation on three routers (i.e.,
second,wehaveatimecostof46.80secondsandasuccessrate TP-LinkTL-XDR6020,LinksysE5600,andXiaomiRA81)in
of85%.However,weobserveatimecostof64.96secondsand our laboratory and performed the two attacks. The experiment
a success rate of 70% when the NAT mapping timeout is 10 resultsshowthatthetimecostandsuccessratearenotaffected
| seconds. Thereasonisthatalargertimeoutvalueincurslonger |     |     |     | by AP isolation. |     |     |     |     |
| ------------------------------------------------------- | --- | --- | --- | ---------------- | --- | --- | --- | --- |
12

B. Comparison with Prior Attacks in Wi-Fi Networks isolation. Moreover, lots of strategies have been proposed to
|               |     |         |          |     |      |             |        | prevent | prior | attacks, while |     | our attack | is a | novel | one whose |
| ------------- | --- | ------- | -------- | --- | ---- | ----------- | ------ | ------- | ----- | -------------- | --- | ---------- | ---- | ----- | --------- |
| ARP Poisoning |     | Attack. | Compared |     | with | our attack, | a suc- |         |       |                |     |            |      |       |           |
vulnerabilityhasexistedinroutersforyears.Additionally,our
| cessful | ARP poisoning |     | attack | can | intercept | traffic | in both |               |     |            |               |     |     |           |          |
| ------- | ------------- | --- | ------ | --- | --------- | ------- | ------- | ------------- | --- | ---------- | ------------- | --- | --- | --------- | -------- |
|         |               |     |        |     |           |         |         | attack serves | as  | a valuable | supplementary |     |     | attack in | networks |
directions,i.e.,fromthevictimclientandtherouter,whileour
|     |     |     |     |     |     |     |     | equipped | with | defense | measures | against | existing | attacks. |     |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ---- | ------- | -------- | ------- | -------- | -------- | --- |
attackcanonlyinterceptTCPtrafficfromtherouter.However,
| ARP poisoning |     | attack | in wired | or  | wireless | LANs | has been |              |     |            |       |     |     |     |     |
| ------------- | --- | ------ | -------- | --- | -------- | ---- | -------- | ------------ | --- | ---------- | ----- | --- | --- | --- | --- |
|               |     |        |          |     |          |      |          | C. Extending |     | the Attack | Model |     |     |     |     |
well-researchedsinceitappeared.Userscaninstallsomeopen-
| sourced        | tools [8], | [49],       | [1] to      | prevent        | the           | attack.          | Besides,    |              |            |              |          |                |               |              |            |
| -------------- | ---------- | ----------- | ----------- | -------------- | ------------- | ---------------- | ----------- | ------------ | ---------- | ------------ | -------- | -------------- | ------------- | ------------ | ---------- |
|                |            |             |             |                |               |                  |             | In our       | extended   | model,       | we       | eliminate      | the           | requirement  | that       |
| some routers   | (e.g.,     | TP-Link)    |             | offer built-in |               | ARP protections. |             |              |            |              |          |                |               |              |            |
|                |            |             |             |                |               |                  |             | the attacker | and        | the victim   |          | client have    | to            | be located   | in the     |
| Moreover,      | AP         | isolation   | can         | also defend    |               | against          | the ARP     |              |            |              |          |                |               |              |            |
|                |            |             |             |                |               |                  |             | same Wi-Fi   | network.   |              | Instead, | we demonstrate |               | that         | a remote   |
| poisoning      | attack     | effectively | by          | preventing     | communication |                  | be-         |              |            |              |          |                |               |              |            |
|                |            |             |             |                |               |                  |             | attacker     | from       | the Internet | can      | launch         | a DoS         | attack       | on TCP     |
| tween clients, | while      | our         | attack      | is only        | partially     |                  | affected as |              |            |              |          |                |               |              |            |
|                |            |             |             |                |               |                  |             | connections  | between    | victim       | clients  | behind         | a             | vulnerable   | router     |
| stated in      | Section    | IV-D        | and Section | VI-A.          | We            | make             | a further   |              |            |              |          |                |               |              |            |
|                |            |             |             |                |               |                  |             | and an       | external   | server.      | We       | require        | that          | the attacker | can        |
| empirical      | study      | in 10       | real-world  | Wi-Fi          | networks.     |                  | Three of    |              |            |              |          |                |               |              |            |
|                |            |             |             |                |               |                  |             | send packets |            | with spoofed |          | source IP      | addresses,    |              | which is a |
| them have      | enabled    | AP          | isolation   | and            | can           | prevent          | the ARP     |              |            |              |          |                |               |              |            |
|                |            |             |             |                |               |                  |             | practical    | assumption | considering  |          | that           | approximately |              | a quarter  |
poisoningattack.OntheotherWi-Finetworks,ARPpoisoning
|     |     |     |     |     |     |     |     | of autonomous |     | systems | still | do not | employ | source | address |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------- | --- | ------- | ----- | ------ | ------ | ------ | ------- |
cansucceed.However,weobservethatitfailswhenweenable
|                 |            |              |            |             |        |        |            | validation | (SAV), | as reported |     | by the | Spoofer | project | [27]. |
| --------------- | ---------- | ------------ | ---------- | ----------- | ------ | ------ | ---------- | ---------- | ------ | ----------- | --- | ------ | ------- | ------- | ----- |
| protections     | on routers |              | and client | devices     | (e.g., | using  | the tool   |            |        |             |     |        |         |         |       |
| developed       | in [1]).   | In contrast, |            | our attack  | can    | still  | succeed in |            |        |             |     |        |         |         |       |
| these networks, |            | even when    | these      | protections |        | are in | place.     |            |        |             |     |        |         |         |       |
…
Eavesdropping Attack. Against WPA2-Personal mode Wi- Client_n Client_1 Router Attacker
Server
A victim constructs a TCP connection with src.port= m
| Fi networks, | a malicious |     | attacker | who | knows | the | pre-shared |     |     |     |     |     |     |     |     |
| ------------ | ----------- | --- | -------- | --- | ----- | --- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
NAT mappings
| key can  | sniff the | frames      | in  | the air  | of other | clients. | If it     |     |     |                      |      |     |     |     |     |
| -------- | --------- | ----------- | --- | -------- | -------- | -------- | --------- | --- | --- | -------------------- | ---- | --- | --- | --- | --- |
|          |           |             |     |          |          |          |           |     |     | victim:m    wan_ip:m | …... |     |     |     |     |
| wants to | decrypt   | the frames, |     | it needs | to       | capture  | the 4-way |     |     |                      |      |     |     |     |     |
handshake frames when other clients are connecting to the R…ST s r c = s e r v e r
network. Though the attacker can force the victim client to be d s t. p o r t = i   in range
|     |     |     |     |     |     |     |     |     |     | RST | seq=rand |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | --- | --- | --- |
1
detachedfromthecurrentAPbysendingfakedeauthentication s r c . ip = w a n _ i p PUS…H/ACK
Check and delete the mapping
frames [46] and wait for its re-connection, the attack is s r c .p o rt = i in   r ange
|     |     |     |     |     |     |     |     |     |     |     | …... |     | seq=arbitrary |     | PUSH/ACK |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---- | --- | ------------- | --- | -------- |
perceivable by the victim that its device will lose Internet 2
access. However, our attack is stealthier as the attacker only with the exact
|          |         |        |      |         |     |          |         |     |     |     |     |     | ACK | SEQ and ACK |     |
| -------- | ------- | ------ | ---- | ------- | --- | -------- | ------- | --- | --- | --- | --- | --- | --- | ----------- | --- |
| needs to | connect | to the | same | network | and | does not | need to |     |     |     |     |     |     |             |     |
No corresponding mapping
maketheclientdisconnectfromtheexistingnetworktolaunch
…...
the attack. Besides, it’s much harder to decrypt the frames src.ip=wan_ip
src.port=m
| encrypted | with | WPA2-Enterprise |     | mode | or  | WPA3-Personal |     |     |     |     |     | RST |     |     |     |
| --------- | ---- | --------------- | --- | ---- | --- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
seq=SEQ
| mode, while | our | attack | can also | influence | these | networks. |     |     |     |     |     |     |     |     |     |
| ----------- | --- | ------ | -------- | --------- | ----- | --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
terminate
the connection
| Rogue | AP Attack. | The | malicious |     | insider | who | knows the |     |     | ACK |     |     |     |     |     |
| ----- | ---------- | --- | --------- | --- | ------- | --- | --------- | --- | --- | --- | --- | --- | --- | --- | --- |
SYN/ACK
pre-shared key can also create a rogue clone (evil twin) Fig.6. RemoteTCPDoSattack
| of the      | network      | and entice | unsuspecting |         | victims |            | to connect |     |     |     |     |     |     |     |     |
| ----------- | ------------ | ---------- | ------------ | ------- | ------- | ---------- | ---------- | --- | --- | --- | --- | --- | --- | --- | --- |
| to it, thus | intercepting |            | all the      | traffic | [2],    | [39]. This | attack     |     |     |     |     |     |     |     |     |
Assumingthatthereisaliveconnectionbetweenanoutside
| requires       | broadcasting | the            | same | SSID,    | which   | can be | detected |                  |     |        |          |             |        |     |            |
| -------------- | ------------ | -------------- | ---- | -------- | ------- | ------ | -------- | ---------------- | --- | ------ | -------- | ----------- | ------ | --- | ---------- |
|                |              |                |      |          |         |        |          | server and       | the | victim | client   | who resides | behind | a   | vulnerable |
| by the network |              | administrator, |      | and some | routers | also   | provide  |                  |     |        |          |             |        |     |            |
|                |              |                |      |          |         |        |          | router. Compared |     | to the | original | attack      | model, | the | attacker   |
protectionstrategiessuchasRogueAPdetection[21],[24].In
|           |            |               |     |          |       |          |          | cannot    | infer the | source  | port     | of the | victim | client | anymore   |
| --------- | ---------- | ------------- | --- | -------- | ----- | -------- | -------- | --------- | --------- | ------- | -------- | ------ | ------ | ------ | --------- |
| contrast, | our attack | is stealthier |     | as there | is no | specific | strategy |           |           |         |          |        |        |        |           |
|           |            |               |     |          |       |          |          | using the | method    | before. | However, |        | as we  | show   | in Figure |
providedtodetectourattack.Inaddition,alightweightdevice
|             |            |              |          |          |         |          |          | 6, the attacker |        | can send    | forged  | TCP    | RST         | packets | covering  |
| ----------- | ---------- | ------------ | -------- | -------- | ------- | -------- | -------- | --------------- | ------ | ----------- | ------- | ------ | ----------- | ------- | --------- |
| compromised | by         | the attacker |          | remotely | may     | not have | enough   |                 |        |             |         |        |             |         |           |
|             |            |              |          |          |         |          |          | the entire      | space  | of possible |         | source | ports       | to the  | public IP |
| resources   | to provide | the          | services | as       | a rogue | AP. The  | attacker |                 |        |             |         |        |             |         |           |
|             |            |              |          |          |         |          |          | address         | of the | vulnerable  | router. | As     | the routers | do      | not check |
whoisphysicallyintheLANcansetitsowndeviceasarogue
|     |     |     |     |     |     |     |     | the sequence | number | specified |     | in TCP | packets | strictly, | these |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | ------ | --------- | --- | ------ | ------- | --------- | ----- |
AP,butithastoprovideastrongersignalthantheoriginalAP,
|             |               |         |              |            |            |          |            | RST packets     | can  | easily           | bypass   | routers’     | checks      | to          | clean the   |
| ----------- | ------------- | ------- | ------------ | ---------- | ---------- | -------- | ---------- | --------------- | ---- | ---------------- | -------- | ------------ | ----------- | ----------- | ----------- |
| and thus    | the influence |         | is limited   | to clients |            | in close | proximity. |                 |      |                  |          |              |             |             |             |
|             |               |         |              |            |            |          |            | possible        | NAT  | mapping          | of the   | victim       | connection. |             | After the   |
| Conversely, | our           | attack  | does not     | face       | the signal | race     | that any   |                 |      |                  |          |              |             |             |             |
|             |               |         |              |            |            |          |            | NAT mapping     |      | disappears,      | the      | attacker     | can         | send forged | TCP         |
| device in   | the same      | Wi-Fi   | network      | can        | launch     | the      | attack and |                 |      |                  |          |              |             |             |             |
|             |               |         |              |            |            |          |            | data packets    | to   | the server       | with     | a spoofed    |             | source      | IP address  |
| potentially | influence     | all     | clients.     | Besides,   | enterprise |          | mode Wi-   |                 |      |                  |          |              |             |             |             |
|             |               |         |              |            |            |          |            | of the router’s |      | public IP,       | covering | the          | entire      | space       | of possible |
| Fi networks | can           | protect | clients      | from       | the        | Rogue    | AP attack. |                 |      |                  |          |              |             |             |             |
|             |               |         |              |            |            |          |            | source ports,   | too. | The              | server   | will respond | to          | the matched | one         |
| However,    | our empirical |         | measurements |            | have       | revealed | that our   |                 |      |                  |          |              |             |             |             |
|             |               |         |              |            |            |          |            | with an         | ACK  | packet specified |          | with the     | current     | sequence    | and         |
attackcancompromisethetrafficofclientswithin22different
|            |      |       |           |     |     |     |     | acknowledgment   |     | numbers | to      | the router. | However, |        | as there is |
| ---------- | ---- | ----- | --------- | --- | --- | --- | --- | ---------------- | --- | ------- | ------- | ----------- | -------- | ------ | ----------- |
| enterprise | mode | Wi-Fi | networks. |     |     |     |     |                  |     |         |         |             |          |        |             |
|            |      |       |           |     |     |     |     | no corresponding |     | NAT     | mapping | of this     | ACK      | packet | anymore,    |
RST
Comparedwiththepriorworks,ourattackleveragesanew the router will just send a packet back to the server with
side channel vulnerability of the NAT behaviors in routers the sequence number received just now. Then the connection
that can be exploited to hijack TCP connections by off- will be terminated from the server side. If the client continues
path attackers, even in enterprise mode networks with AP communicating with the server, it will receive a TCP RST
13

packetbackfromtheserverafterward.Inthisway,theattacker TCP Window Tracking. As a middle device between the
can interfere with TCP communications between the victim internal clients and outside servers, the router has to keep
andserver,causingaDoSattackandaffectinguserexperience. the necessary information about connections. However, most
|     |     |     |     |     |     |     |     | routers have | disabled |     | TCP window |     | tracking | for performance |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------ | -------- | --- | ---------- | --- | -------- | --------------- | --- |
Theattackerneedstodetectsuchvictimclientswhoaccess
|     |     |     |     |     |     |     |     | reasons. | Nevertheless, |     | we find | that | a simple | TCP | RST packet |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | ------------- | --- | ------- | ---- | -------- | --- | ---------- |
the Internet through these vulnerable routers. We find that can be abused to clear the NAT mapping and be leveraged
| it is of        | great   | convenience  | for    | the         | attacker   | to identify | these   |                 |          |          |         |                |             |                 |              |
| --------------- | ------- | ------------ | ------ | ----------- | ---------- | ----------- | ------- | --------------- | -------- | -------- | ------- | -------------- | ----------- | --------------- | ------------ |
|                 |         |              |        |             |            |             |         | to launch       | our      | attack.  | In this | way,           | we believe  | it              | essential to |
| vulnerable      | routers | through      | open   | search      | engines    | [50],       | [13],   |                 |          |          |         |                |             |                 |              |
|                 |         |              |        |             |            |             |         | strictly check  | the      | sequence | and     | acknowledgment |             | numbers         | for          |
| which contain   |         | a large      | amount | of publicly | accessible |             | devices |                 |          |          |         |                |             |                 |              |
|                 |         |              |        |             |            |             |         | received        | packets. | The      | OpenWrt | community      |             | has implemented |              |
| (e.g., routers, |         | web servers, | and    | webcams).   | For        | example,    | mil-    |                 |          |          |         |                |             |                 |              |
|                 |         |              |        |             |            |             |         | this mitigation |          | as they  | believe | the            | performance | impact          | should       |
lions of TP-Link routers with public IP addresses can be not matter anymore on any currently supported hardware.
| found through |     | FOFA       | [13]. We | estimate | that | there     | are tens |     |     |     |     |     |     |     |     |
| ------------- | --- | ---------- | -------- | -------- | ---- | --------- | -------- | --- | --- | --- | --- | --- | --- | --- | --- |
| of millions   | of  | vulnerable | routers  | existing | in   | the world | which    |     |     |     |     |     |     |     |     |
may be influenced by the attack, and we believe this attack VIII. RELATEDWORK
| is promising | and   | practical   | and | may      | affect many    | more            | users. |              |           |           |              |                 |               |              |            |
| ------------ | ----- | ----------- | --- | -------- | -------------- | --------------- | ------ | ------------ | --------- | --------- | ------------ | --------------- | ------------- | ------------ | ---------- |
|              |       |             |     |          |                |                 |        | Traffic      | hijacking |           | has been     | widely          |               | studied, and | lots of    |
| We leave     | it as | future work | to  | validate | the real-world |                 | impact |              |           |           |              |                 |               |              |            |
|              |       |             |     |          |                |                 |        | attacks have | been      | proposed. |              | Vulnerabilities |               | that lead    | to traffic |
| in practice  | owing | to reasons  |     | such as  | ethical        | considerations. |        |              |           |           |              |                 |               |              |            |
|              |       |             |     |          |                |                 |        | hijacking    | may       | exist     | in protocols |                 | at all levels | of           | the TCP/IP |
protocolstack.Forinstance,inthesameLAN,anattackercan
VII. COUNTERMEASURES exploitthevulnerabilityoftheARPprotocoltohijacknetwork
|             |               |     |             |     |     |               |     | traffic by      | sending | fake | ARP    | packets | and    | compromising | the         |
| ----------- | ------------- | --- | ----------- | --- | --- | ------------- | --- | --------------- | ------- | ---- | ------ | ------- | ------ | ------------ | ----------- |
| Responsible | Vulnerability |     | Disclosure. |     | We  | have reported | the |                 |         |      |        |         |        |              |             |
|             |               |     |             |     |     |               |     | victim device’s |         | ARP  | cache, | which   | allows | the          | attacker to |
issuetotheaffectedmanufacturersbysubmittingvulnerability
|             |            |          |           |        |          |         |             | intercept, | modify,      | or  | even discard |            | the traffic | of victims, | thus |
| ----------- | ---------- | -------- | --------- | ------ | -------- | ------- | ----------- | ---------- | ------------ | --- | ------------ | ---------- | ----------- | ----------- | ---- |
| reports and | contacting |          | them via  | email. | At the   | time    | of writing, |            |              |     |              |            |             |             |      |
|             |            |          |           |        |          |         |             | hijacking  | the victims’ |     | traffic      | completely |             | [19].       |      |
| we have     | received   | positive | responses |        | from the | OpenWrt | com-        |            |              |     |              |            |             |             |      |
munity that confirms our findings and has released patches to At the IP layer, attackers may leverage the ICMP redirect
fix the vulnerability, and seven router vendors (i.e., TP-Link, mechanism to hijack victims’ traffic by placing themselves
Huawei, Xiaomi, 360, Mercury, Ubiquiti, and Linksys) that in the man-in-the-middle position [28], [3]. Recently, Feng
haveallacknowledgedourreportsandaretryingtorepairtheir et al. developed a new method to circumvent the ICMP
products. In addition, we have been assigned 10 CVE num- redirect legitimacy checks in Wi-Fi networks and presented
bers for the vulnerability in different vendors (i.e., TP-Link, an attack to evade the security mechanisms of WPAs [9].
Linksys, Mercury, Ruijie, D-Link, Comfast, H3C, OpenWrt, However, the attack targets out-of-date systems (e.g., iOS 1-8,
Wavlink, and 360). The other vendors are still investigating Android before 10.0) except for the latest versions of Linux
the vulnerability. We also provide them with countermeasure andFreeBSD.Besides,theyalsoshowedthatoff-pathattackers
suggestions to mitigate the identified attack, and some of from the Internet could trick public servers into redirecting
them have been adopted by the vendors. As mentioned in their traffic to neighboring hosts with forged ICMP redirect
Section III, we outline several conditions that characterize a messages, thus causing a DoS attack [12].
| vulnerable       | router | implementation. |     |        | Intuitively, | any | breach of |     |       |           |         |     |      |           |           |
| ---------------- | ------ | --------------- | --- | ------ | ------------ | --- | --------- | --- | ----- | --------- | ------- | --- | ---- | --------- | --------- |
|                  |        |                 |     |        |              |     |           | DNS | cache | poisoning | attacks | can | also | be abused | to hijack |
| these conditions |        | will render     | the | attack | ineffective. |     |           |     |       |           |         |     |      |           |           |
traffic.InthesameLAN,Herzbergetal.proposedthreemeth-
Random Port Allocation. The first solution is for the router ods to circumvent source port randomization, which leverages
to use the random selection strategy when creating new NAT theportallocationstrategiesusedbyNATdevices[18].Zheng
mappings. In detail, the router can choose a random port et al. developed an attack targeting DNS forwarders (e.g.,
from the available port pool and record the port translation home routers) by forcing fragmentation using attacker-owned
whenallocatingnewmappings.Withthisstrategy,theattacker authoritative name servers [59]. Man et al. proposed that a
cannotidentifywhethertheporthasbeenusedbyotherinternal purely off-path attacker from the Internet can exploit the side
hosts, and the attack will be foiled. It should be noted that channel in ICMP rate limit or the limited space for storing
some TCP punch-through schemes (e.g., TCP simultaneous the next hop exception cache to infer the source ports of DNS
open) may be influenced by random selection as they rely requests and poison DNS caches maliciously [30], [31].
| on port | prediction | [58]. | Alternatively, |     | clients | can utilize | some |     |     |     |     |     |     |     |     |
| ------- | ---------- | ----- | -------------- | --- | ------- | ----------- | ---- | --- | --- | --- | --- | --- | --- | --- | --- |
other common-used schemes (e.g., TURN relaying) for NAT To hijack TCP connections so as to inject forged TCP
|                |     |       |          |             |       |     |     | segments | into    | the target | connection   |     | or terminate     |     | it, attackers |
| -------------- | --- | ----- | -------- | ----------- | ----- | --- | --- | -------- | ------- | ---------- | ------------ | --- | ---------------- | --- | ------------- |
| punch-through, |     | which | will not | be affected | [16]. |     |     |          |         |            |              |     |                  |     |               |
|                |     |       |          |             |       |     |     | mainly   | rely on | various    | side-channel |     | vulnerabilities. |     | Cao et        |
Reverse Path Validation. Another effective measure to pre- al. demonstrated that a global shared variable used in the
vent the attack is to adopt the RFC 3704 recommendation, challenge ACK mechanism could be abused for an off-path
which suggests using the strict mode to filter out forged attacker to manipulate the victim TCP traffic [6]. Chen et
packets.Inourtest,routersfromASUS,Netgear,ZTE,Aruba, al. showed that a timing side channel that exists in half-
Cisco Meraki, and certain models of TP-LINK, Mercury, and duplex IEEE 802.11 or Wi-Fi technology [7] and Feng et al.
Huawei take this recommendation by default, thus defending discovered a side channel in the mixed IPID assignment [10],
against our attack. However, this strategy may introduce addi- [11],whichcanalsobeexploitedtomanipulateTCPtrafficby
tional performance overhead and potentially impact the relia- off-path attackers. Tolley et al. demonstrated that blind in/on-
bility of networking for certain applications (e.g., OpenVPN path attackers could learn the virtual IP of a host behind a
running on the router may be affected as the reverse path VPNandhijackTCPconnectionssupposedlyprotectedbythe
validation may interfere with packet delivery [38]). tunnel [51]. Besides, Schepers et al. discovered that modern
14

| operating | systems | fail | to manage | the | security | context | of their |                 |      |               |                                         |     |     |     |
| --------- | ------- | ---- | --------- | --- | -------- | ------- | -------- | --------------- | ---- | ------------- | --------------------------------------- | --- | --- | --- |
|           |         |      |           |     |          |         |          | [8] A. Chirila, | “Arp | antispoofer,” | https://www.softpedia.com/get/Security/ |     |     |     |
transmitqueuessecurely,therebyallowingamaliciousattacker Firewall/ARP-AntiSpoofer.shtml,AccessedJuly2023.
to intercept frames in Wi-Fi networks, thus hijacking TCP [9] X. Feng, Q. Li, K. Sun, Y. Yang, and K. Xu, “Man-in-the-middle
connections or intercepting client and web traffic [47]. attacks without rogue ap: When wpas meet icmp redirects,” in 2023
|              |     |      |        |                       |     |      |         | IEEESymposiumonSecurityandPrivacy(SP)(SP). |     |     |     |     |     | IEEEComputer |
| ------------ | --- | ---- | ------ | --------------------- | --- | ---- | ------- | ------------------------------------------ | --- | --- | --- | --- | --- | ------------ |
| Fortunately, |     | most | of the | prior vulnerabilities |     | have | already | Society,2023.                              |     |     |     |     |     |              |
been addressed [35], [6], [11], and the security community [10] X. Feng, C. Fu, Q. Li, K. Sun, and K. Xu, “Off-path tcp exploits
has developed corresponding defense measures against these of the mixed ipid assignment,” in Proceedings of the 2020 ACM
SIGSACConferenceonComputerandCommunicationsSecurity,2020,
| attacks | [33], [34], | [23]. | However, | we  | present | a new | type of |     |     |     |     |     |     |     |
| ------- | ----------- | ----- | -------- | --- | ------- | ----- | ------- | --- | --- | --- | --- | --- | --- | --- |
p.1323–1335.
| TCP traffic | hijacking |     | attack | leveraging | the | vulnerabilities | in  |     |     |     |     |     |     |     |
| ----------- | --------- | --- | ------ | ---------- | --- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
routers,whichcancircumventtraditionaldefensesagainstTCP [11] X. Feng, Q. Li, K. Sun, C. Fu, and K. Xu, “Off-path tcp hijacking
attacksviathesidechannelofdowngradedipid,”IEEE/ACMTransac-
traffic hijacking attacks and lead to new challenges for the tionsonNetworking,pp.409–422,2022.
security communities.
|     |     |     |            |     |     |     |     | [12] X.Feng,Q.Li,K.Sun,Z.Qian,G.Zhao,X.Kuang,C.Fu,andK.Xu, |         |         |              |                 |         |               |
| --- | --- | --- | ---------- | --- | --- | --- | --- | ---------------------------------------------------------- | ------- | ------- | ------------ | --------------- | ------- | ------------- |
|     |     |     |            |     |     |     |     | “Off-Path                                                  | network | traffic | manipulation | via revitalized |         | ICMP redirect |
|     |     |     |            |     |     |     |     | attacks,”                                                  | in 31st | USENIX  | Security     | Symposium       | (USENIX | Security 22), |
|     |     | IX. | CONCLUSION |     |     |     |     |                                                            |         |         |              |                 |         |               |
2022,pp.2619–2636.
In this paper, we uncover a new off-path TCP hijacking [13] FOFA, “Fofa search engine,” https://en.fofa.info/, Accessed March
2023.
| attack in | the Wi-Fi | networks |     | that leverages | vulnerable |     | routers. |     |     |     |     |     |     |     |
| --------- | --------- | -------- | --- | -------------- | ---------- | --- | -------- | --- | --- | --- | --- | --- | --- | --- |
We find that a malicious insider can abuse the NAT port [14] B. Ford, S. Guha, K. Biswas, S. Sivakumar, and P. Srisuresh, “NAT
BehavioralRequirementsforTCP,”RFC5382,Tech.Rep.5382,Oct.
| preservation | strategy |     | and insufficient |     | reverse | path validation |     |     |     |     |     |     |     |     |
| ------------ | -------- | --- | ---------------- | --- | ------- | --------------- | --- | --- | --- | --- | --- | --- | --- | --- |
2008.[Online].Available:https://www.rfc-editor.org/info/rfc5382
strategyoftheroutertoinfertheexistenceofTCPconnections
fromtheLANtoaremoteserverandthenobtainthesequence [15] B.FordandP.Srisuresh,“Unintendedconsequencesofnatdeployments
|     |     |     |     |     |     |     |     | with | overlapping | address | space,” | Internet Requests |     | for Comments, |
| --- | --- | --- | --- | --- | --- | --- | --- | ---- | ----------- | ------- | ------- | ----------------- | --- | ------------- |
and acknowledgment numbers by manipulating the state of InternetEngineeringTaskForce,RFC5684,February2010.[Online].
NATmappingswithforgedresetpacketsduetothevulnerable Available:http://www.rfc-editor.org/rfc/rfc5684.txt
| routers | disabling | TCP | window | tracking | strategy. | We  | confirm |                                                                     |     |     |     |     |     |     |
| ------- | --------- | --- | ------ | -------- | --------- | --- | ------- | ------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|         |           |     |        |          |           |     |         | [16] B.Ford,P.Srisuresh,andD.Kegel,“Peer-to-peercommunicationacross |     |     |     |     |     |     |
the vulnerability in a wide range of routers from different networkaddresstranslators,”inProceedingsoftheAnnualConference
manufacturers and evaluate the new attack in different scenar- on USENIX Annual Technical Conference, ser. ATEC ’05. USENIX
Association,2005,p.13.
| ios, such | as SSH | DoS,      | FTP      | hijacking, | and HTTP | injection        | in  |                                                                   |     |     |     |     |     |     |
| --------- | ------ | --------- | -------- | ---------- | -------- | ---------------- | --- | ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|           |        |           |          |            |          |                  |     | [17] Y.GiladandA.Herzberg,“Off-pathtcpinjectionattacks,”ACMTrans. |     |     |     |     |     |     |
| various   | Wi-Fi  | networks. | Finally, | we suggest |          | countermeasures, |     |                                                                   |     |     |     |     |     |     |
Inf.Syst.Secur.,2014.
| report the | vulnerabilities |     | to  | the affected | manufacturers, |     | and |     |     |     |     |     |     |     |
| ---------- | --------------- | --- | --- | ------------ | -------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
have received positive acknowledgments. [18] A.HerzbergandH.Shulman,“Securityofpatcheddns,”inComputer
|     |     |     |     |     |     |     |     | Security | – ESORICS | 2012. | Springer | Berlin | Heidelberg, | 2012, pp. |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --------- | ----- | -------- | ------ | ----------- | --------- |
271–288.
ACKNOWLEDGMENT
|     |     |     |     |     |     |     |     | [19] S. Hijazi | and | M. S. Obaidat, | “Address | resolution | protocol | spoofing |
| --- | --- | --- | --- | --- | --- | --- | --- | -------------- | --- | -------------- | -------- | ---------- | -------- | -------- |
attacksandsecurityapproaches:Asurvey,”SecurityandPrivacy,vol.2,
We thank our shepherd and anonymous reviewers for no.1,pp.1–9,2019.
| their thoughtful |     | comments. |     | This work | was | in part | sup- |                                                                |     |     |     |     |     |     |
| ---------------- | --- | --------- | --- | --------- | --- | ------- | ---- | -------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|                  |     |           |     |           |     |         |      | [20] M.HoldregeandP.Srisuresh,“IPNetworkAddressTranslator(NAT) |     |     |     |     |     |     |
ported by China National Funds for Distinguished Young Terminology and Considerations,” RFC 2663, Tech. Rep. 2663, Aug.
Scientists under No.61825204, NSFC under No.62132011, 1999.[Online].Available:https://www.rfc-editor.org/info/rfc2663
and Beijing Outstanding Young Scientist Program under [21] Huawei, “Rogue device detection,” https://support.
huawei.com/enterprise/en/doc/EDOC1100096321/3eb0a62e/
| No.BJJWZYJH01201910003011. |     |     |     | Ke Xu | is the | corresponding |     |     |     |     |     |     |     |     |
| -------------------------- | --- | --- | --- | ----- | ------ | ------------- | --- | --- | --- | --- | --- | --- | --- | --- |
example-for-configuring-rogue-device-detection-and-containment,
author.
AccessedJuly2023.
|     |     |     |     |     |     |     |     | [22] M.Kol,A.Klein,andY.Gilad,“Devicetrackingvialinux’snewTCP |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | ------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
REFERENCES sourceportselectionalgorithm,”in32ndUSENIXSecuritySymposium
(USENIXSecurity23),2023.
| [1] 360-ARP, | “360 | total | security: | Free antivirus | protection | for | home and |                                                                   |     |     |     |     |     |     |
| ------------ | ---- | ----- | --------- | -------------- | ---------- | --- | -------- | ----------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|              |      |       |           |                |            |     |          | [23] M.LepinskiandK.Sriram,“BGPsecProtocolSpecification,”Internet |     |     |     |     |     |     |
devices,”http://www.360totalsecurity.com/en/,AccessedJuly2023.
|     |     |     |     |     |     |     |     | Requests | for | Comments, | Internet Engineering |     | Task Force, | RFC 8205, |
| --- | --- | --- | --- | --- | --- | --- | --- | -------- | --- | --------- | -------------------- | --- | ----------- | --------- |
[2] A.M.Alsahlany,A.R.Almusawy,andZ.H.Alfatlawy,“Riskanalysis September 2017. [Online]. Available: http://www.rfc-editor.org/rfc/
| of  | a fake | access point | attack | against | wi-fi network,” | International |     |     |     |     |     |     |     |     |
| --- | ------ | ------------ | ------ | ------- | --------------- | ------------- | --- | --- | --- | --- | --- | --- | --- | --- |
rfc8205.txt
| Journal | of  | Scientific | & Engineering | Research, |     | vol. 9, pp. | 322–326, |                                                                 |     |     |     |     |     |     |
| ------- | --- | ---------- | ------------- | --------- | --- | ----------- | -------- | --------------------------------------------------------------- | --- | --- | --- | --- | --- | --- |
|         |     |            |               |           |     |             |          | [24] Linksys,“Howtoenablerogueapdetectiononyourlinksyswireless- |     |     |     |     |     |     |
2018.
|        |             |          |         |               |                        |     |     | ac access | point,” | https://www.linksys.com/support-article?articleNum= |     |     |     |     |
| ------ | ----------- | -------- | ------- | ------------- | ---------------------- | --- | --- | --------- | ------- | --------------------------------------------------- | --- | --- | --- | --- |
| [3] A. | Ayer, “Icmp | redirect | attacks | in the wild,” | https://www.agwa.name/ |     |     |           |         |                                                     |     |     |     |     |
135793,AccessedJuly2023.
| blog/post/icmp |       | redirect   | attacks    | in the wild,AccessedMarch2023. |                |           |            |               |            |                                          |       |             |                          |     |
| -------------- | ----- | ---------- | ---------- | ------------------------------ | -------------- | --------- | ---------- | ------------- | ---------- | ---------------------------------------- | ----- | ----------- | ------------------------ | --- |
|                |       |            |            |                                |                |           |            | [25] Linux,   | “Netfilter | conntrack                                | sysfs | variables,” | https://docs.kernel.org/ |     |
| [4] F. Baker   | and   | P. Savola, | “Ingress   | Filtering                      | for Multihomed |           | Networks,” |               |            |                                          |       |             |                          |     |
|                |       |            |            |                                |                |           |            | networking/nf |            | conntrack-sysctl.html,AccessedMarch2023. |       |             |                          |     |
| RFC            | 3704, | Tech.      | Rep. 3704, | Mar.                           | 2004.          | [Online]. | Available: |               |            |                                          |       |             |                          |     |
https://www.rfc-editor.org/info/rfc3704 [26] ——,“rp filter,”https://sysctl-explorer.net/net/ipv4/rp filter/,Accessed
March2023.
[5] A.Biggadike,D.Ferullo,G.Wilson,andA.Perrig,“NATBLASTER:
Establishing TCP connections between hosts behind NATs,” in Pro- [27] Q. Lone, A. Frik, M. Luckie, M. Korczyn´ski, M. van Eeten, and
ceedingsofACMSIGCOMMASIAWorkshop,2005. C. Gan˜a´n, “Deployment of source address validation by network
|        |         |          |       |            |                   |     |           | operators: | A   | randomized | control trial,” | in 2022 | IEEE | Symposium on |
| ------ | ------- | -------- | ----- | ---------- | ----------------- | --- | --------- | ---------- | --- | ---------- | --------------- | ------- | ---- | ------------ |
| [6] Y. | Cao, Z. | Qian, Z. | Wang, | T. Dao, S. | V. Krishnamurthy, |     | and L. M. |            |     |            |                 |         |      |              |
SecurityandPrivacy(SP),2022,pp.2361–2378.
Marvel,“Off-pathtcpexploits:Globalratelimitconsidereddangerous,”
in25thUSENIXSecuritySymposium(USENIXSecurity16),2016,pp. [28] C.Low,“Icmpattacksillustrated,”https://www.sans.org/reading-room/
whitepapers/threats/paper/477,AccessedMarch2023.
209–225.
[7] W.ChenandZ.Qian,“Off-pathtcpexploit:Howwirelessrouterscan [29] V. Mahajan and S. K. Peddoju, “Deployment of intrusion detection
jeopardizeyoursecrets,”in27thUSENIXSecuritySymposium(USENIX system in cloud: A performance-based study,” in 2017 IEEE Trust-
Security18),2018,pp.1581–1598. com/BigDataSE/ICESS,2017,pp.1103–1108.
15

[30] K. Man, Z. Qian, Z. Wang, X. Zheng, Y. Huang, and H. Duan, “Dns [53] M. Vanhoef, “Fragment and forge: Breaking Wi-Fi through frame
cache poisoning attack reloaded: Revolutions with side channels,” in aggregation and fragmentation,” in 30th USENIX Security Symposium
Proceedings of the 2021 ACM SIGSAC Conference on Computer and (USENIXSecurity21),2021,pp.161–178.
CommunicationsSecurity. ACM,2020,pp.1337–1350. [54] M. Vanhoef, P. Adhikari, and C. Po¨pper, “Protecting wi-fi beacons
[31] K. Man, X. Zhou, and Z. Qian, “Dns cache poisoning attack: Resur- from outsider forgeries,” in Proceedings of the 13th ACM Conference
rectionswithsidechannels,”inProceedingsofthe2021ACMSIGSAC on Security and Privacy in Wireless and Mobile Networks, 2020, p.
ConferenceonComputerandCommunicationsSecurity. ACM,2021, 155–160.
pp.3400–3414. [55] M.VanhoefandF.Piessens,“Keyreinstallationattacks:Forcingnonce
[32] D. S. Mathy Vanhoef, “Macstealer: Wi-fi client isolation by- reuseinwpa2,”inProceedingsofthe2017ACMSIGSACConference
pass,” https://github.com/vanhoefm/macstealer#id-test-isolation, Ac- onComputerandCommunicationsSecurity,2017,pp.1313–1328.
cessedJuly2023. [56] ——, “Release the kraken: new kracks in the 802.11 standard,” in
[33] J. McCann, S. Deering, and J. Mogul, “Path mtu discovery Proceedings of the 2018 ACM SIGSAC Conference on Computer and
for ip version 6,” Internet Requests for Comments, Internet CommunicationsSecurity,2018,pp.299–314.
EngineeringTaskForce,RFC1981,August1996.[Online].Available: [57] M. Vanhoef and E. Ronen, “Dragonblood: Analyzing the dragonfly
http://www.rfc-editor.org/rfc/rfc1981.txt handshakeofwpa3andeap-pwd,”in2020IEEESymposiumonSecurity
[34] J. Mogul and S. Deering, “Path mtu discovery,” Internet Requests for andPrivacy(SP),2020.
Comments, Internet Engineering Task Force, RFC 1191, November [58] Z.YongjunandZ.Shiquan,“Natholepunchingbasedonsimultaneous
1990.[Online].Available:http://www.rfc-editor.org/rfc/rfc1191.txt tcpopen,”inIEEEComputerSociety,USA,2013,p.235–238.
[35] S.Y.Nam,S.Jurayev,S.-S.Kim,K.Choi,andG.S.Choi,“Mitigating [59] X.Zheng,C.Lu,J.Peng,Q.Yang,D.Zhou,B.Liu,K.Man,S.Hao,
arppoisoning-basedman-in-the-middleattacksinwiredorwirelesslan,” H. Duan, and Z. Qian, “Poison over troubled forwarders: A cache
EURASIPJournalonWirelessCommunicationsandNetworking,pp.1– poisoning attack targeting DNS forwarding devices,” in 29th USENIX
17,2012. SecuritySymposium(USENIXSecurity20),2020,pp.577–593.
[36] Nmap,“Thenetworkmapper,”https://nmap.org/,AccessedMarch2023.
[37] T.OhigashiandM.Morii,“Apracticalmessagefalsificationattackon APPENDIX
wpa,”Proc.JWIS,vol.54,p.66,2009.
A. Experimental Results of Probing the Router’s External IP
[38] OpenVPN,“Concepts-policyrouting-linux,”https://community.openvpn.
Address
net/openvpn/wiki/Concepts-PolicyRouting-Linux,AccessedJuly2023.
[39] R. Orsi, “Understanding evil twin ap attacks and how to
prevent them,” https://www.darkreading.com/attacks-breaches/
understanding-evil-twin-ap-attacks-and-how-to-prevent-them,2018.
[40] pfSense, “pfsense- world’s most trusted open source firewall,” https:
//www.pfsense.org/,AccessedJuly2023.
[41] Ping, “Linux manual page,” https://man7.org/linux/man-pages/man8/
ping.8.html,AccessedMarch2023.
[42] J.Postel,“TransmissionControlProtocol,”RFC793,Tech.Rep.793,
Sep.1981.[Online].Available:https://www.rfc-editor.org/info/rfc793
[43] Z.QianandZ.M.Mao,“Off-pathtcpsequencenumberinferenceattack
-howfirewallmiddleboxesreducesecurity,”in2012IEEESymposium
onSecurityandPrivacy,2012,pp.347–361.
[44] Z.Qian,Z.M.Mao,andY.Xie,“Collaborativetcpsequencenumber
inference attack: How to crack sequence number under a second,” in
Proceedings of the 2012 ACM Conference on Computer and Commu-
nicationsSecurity,2012,p.593–604.
[45] P. Richter, F. Wohlfart, N. Vallina-Rodriguez, M. Allman, R. Bush,
A. Feldmann, C. Kreibich, N. Weaver, and V. Paxson, “A multi-
perspective analysis of carrier-grade nat deployment,” in Proceedings Fig.7. ThesnapshotoftheprobingtheexternalIPaddressoftherouter.
ofthe2016InternetMeasurementConference. NewYork,NY,USA:
AssociationforComputingMachinery,2016,p.215–229.
[46] D. Schepers, A. Ranganathan, and M. Vanhoef, “On the robustness Figure 7 show the snapshot of the method to probe the
of wi-fi deauthentication countermeasures,” in WiSec ’22: 15th ACM external IP address of the router. The attacker, using a laptop
ConferenceonSecurityandPrivacyinWirelessandMobileNetworks,
withUbuntu22.04,hasconnectedtotheWi-Finetworkwhose
SanAntonio,TX,USA,May16-19,2022. ACM,2022,pp.245–256.
routervendorisWimasterandwhosegatewayIPis10.254.0.1.
[47] ——, “Framing frames: Bypassing Wi-Fi encryption by manipulating
And it has been assigned with the private IP address of
transmit queues,” in 32nd USENIX Security Symposium (USENIX
Security23),2023. 10.254.205.199. Firstly, it gets the gateways along the way to
8.8.8.8throughTracerouteandfindsthatthesecondgateway’s
[48] D. Senie and P. Ferguson, “Network Ingress Filtering: Defeating
DenialofServiceAttackswhichemployIPSourceAddressSpoofing,” IP is 100.64.0.1, which is a carried-grade IP address [45].
RFC 2827, Tech. Rep. 2827, May 2000. [Online]. Available: Secondly, it pings the second gateway (100.64.0.1) with the
https://www.rfc-editor.org/info/rfc2827 RECORD ROUTE option.Theresultshowstheroutespassed,
[49] shARP,https://github.com/europa502/shARP,AccessedJuly2023. and then the attacker can get the external IP address of the
[50] SHODAN,“Searchenginefortheinternetofeverything,”https://www. router (i.e., 100.64.129.73).
shodan.io/,AccessedMarch2023.
[51] W. J. Tolley, B. Kujath, M. T. Khan, N. Vallina-Rodriguez, and J. R.
B. Full List of Tested Routers
Crandall, “Blind In/On-Path attacks and applications to VPNs,” in
30th USENIX Security Symposium (USENIX Security 21). USENIX The detailed information of the 67 tested routers is shown
Association,2021,pp.3129–3146.
in Table IV.
[52] Traceroute, “Linux manual page,” https://man7.org/linux/man-pages/
man8/traceroute.8.html,AccessedMarch2023.
16

|     |     | TABLEIV. | ALLTESTEDROUTERS |     |              |           |          |
| --- | --- | -------- | ---------------- | --- | ------------ | --------- | -------- |
|     |     |          |                  |     | Reverse-path | TCPWindow | TCPClose |
No. RouterModel Vendor OS Generation Port Validation Tracking Timeout Vulnerable
Preservation
|                   |             |                       |        |     | Disabled | Disabled | (second) |
| ----------------- | ----------- | --------------------- | ------ | --- | -------- | -------- | -------- |
|                   |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 1 TL-XDR6020      | TP-Link     | Linux-based           | Wi-Fi6 |     |          |          | 1        |
| 2 TL-R473GP-AC∗   | TP-Link     | Linux-based           | -      | ✔   | ✔        | ✔        | 10 ✔     |
| 3 TL-R4239GP∗     | TP-Link     | Linux-based           | -      | ✔   | ✔        | ✔        | 1 ✔      |
|                   |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 4 TL-WAR1200L     | TP-Link     | Linux-based           | Wi-Fi5 |     |          |          | 1        |
| 5 TL-R476G        | TP-Link     | Linux-based           | Wi-Fi5 | ✔   | ✔        | ✔        | 1 ✔      |
| 6 TL-WDR7620      | TP-Link     | Vxworks-based         | Wi-Fi5 | ✔   | ✘        | ✔        | 1 ✘      |
|                   |             |                       |        | ✔   | ✘        | ✔        | ✘        |
| 7 TL-WR886N       | TP-Link     | Vxworks-based         | Wi-Fi4 |     |          |          | 1        |
| 8 AX3Pro          | Huawei      | EMUI(Linux-based)     | Wi-Fi6 | ✔   | ✔        | ✔        | 10 ✔     |
| 9 AR6140E-9G-2AC∗ | Huawei      | VRP(Linux-based)      | -      | ✘   | ✘        | ✔        | 10 ✘     |
|                   |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 10 TC7102         | Huawei      | EMUI(Linux-based)     | Wi-Fi6 |     |          |          | 10       |
| 11 TC7001         | Huawei      | EMUI(Linux-based)     | Wi-Fi6 | ✔   | ✔        | ✔        | 10 ✔     |
| 12 Q2S            | Huawei      | EMUI(Linux-based)     | Wi-Fi5 | ✔   | ✔        | ✔        | 10 ✔     |
| 13 WS5200         | Huawei      | EMUI(Linux-based)     | Wi-Fi5 | ✔   | ✔        | ✔        | 10 ✔     |
| 14 T6M            | 360         | 360OS(Linux-based)    | Wi-Fi6 | ✔   | ✔        | ✔        | 10 ✔     |
| 15 V6G            | 360         | 360OS(Linux-based)    | Wi-Fi6 | ✔   | ✔        | ✔        | 1 ✔      |
| 16 T5G            | 360         | 360OS(Linux-based)    | Wi-Fi5 | ✔   | ✔        | ✔        | 10 ✔     |
| 17 P1             | 360         | 360OS(Linux-based)    | Wi-Fi4 | ✔   | ✔        | ✔        | 1 ✔      |
| 18 MagicR100      | H3C         | Comware(Linux-based)  | Wi-Fi5 | ✔   | ✔        | ✔        | 10 ✔     |
| 19 MagicR365      | H3C         | Comware(Linux-based)  | Wi-Fi5 | ✔   | ✔        | ✔        | 10 ✔     |
| 20 MagicR2+       | H3C         | Comware(Linux-based)  | Wi-Fi5 | ✔   | ✔        | ✔        | 10 ✔     |
| 21 W30E           | Tenda       | Linux-based           | Wi-Fi6 | ✔   | ✔        | ✔        | 1 ✔      |
| 22 EM12           | Tenda       | Linux-based           | Wi-Fi6 | ✔   | ✔        | ✔        | 1 ✔      |
| 23 RAX1800Z       | ChinaMobile | AOS(Linux-based)      | Wi-Fi6 | ✔   | ✔        | ✔        | 10 ✔     |
| 24 EG105G∗        | Ruijie      | RGOS(Linux-based)     | -      | ✔   | ✔        | ✔        | 1 ✔      |
| EG105G-V2∗        |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 25                | Ruijie      | RGOS(Linux-based)     | -      |     |          |          | 1        |
| 26 EG210G-P∗      | Ruijie      | RGOS(Linux-based)     | -      | ✔   | ✔        | ✔        | 1 ✔      |
| 27 NBR∗           | Ruijie      | RGOS(Linux-based)     | -      | ✔   | ✔        | ✔        | 1 ✔      |
| 28 X32Pro         | Ruijie      | RGOS(Linux-based)     | Wi-Fi6 | ✔   | ✔        | ✔        | 1 ✔      |
| 29 RedmiRA81      | Xiaomi      | MiWiFi(Linux-based)   | Wi-Fi6 | ✔   | ✔        | ✔        | 1 ✔      |
| 30 RedmiRA67      | Xiaomi      | MiWiFi(Linux-based)   | Wi-Fi6 | ✔   | ✔        | ✔        | 1 ✔      |
| 31 R3L            | Xiaomi      | MiWiFi(Linux-based)   | Wi-Fi6 | ✔   | ✔        | ✔        | 10 ✔     |
| 32 R3G            | Xiaomi      | MiWiFi(Linux-based)   | Wi-Fi5 | ✔   | ✔        | ✔        | 10 ✔     |
|                   |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 33 CR6609         | Xiaomi      | MiWiFi(Linux-based)   | Wi-Fi6 |     |          |          | 10       |
| 34 MW300R         | Mercury     | Vxworks-based         | Wi-Fi4 | ✔   | ✘        | ✔        | 1 ✘      |
| 35 X30G           | Mercury     | Linux-based           | Wi-Fi6 | ✔   | ✔        | ✔        | 1 ✔      |
|                   |             |                       |        | ✔   | ✘        | ✔        | ✘        |
| 36 D121G          | Mercury     | Vxworks-based         | Wi-Fi5 |     |          |          | 1        |
| 37 YR1900MG       | Mercury     | Vxworks-based         | Wi-Fi5 | ✔   | ✘        | ✔        | 1 ✘      |
| 38 YR1800XG       | Mercury     | Linux-based           | Wi-Fi6 | ✔   | ✔        | ✔        | 1 ✔      |
|                   |             |                       |        | ✔   | ✘        | ✔        | ✘        |
| 39 RAX20          | Netgear     | DumaOS(Linux-based)   | Wi-Fi6 |     |          |          | 10       |
| 40 RAX50          | Netgear     | DumaOS(Linux-based)   | Wi-Fi6 | ✔   | ✘        | ✔        | 10 ✘     |
| 41 RT-AX57        | ASUS        | AsusWrt(Linux-based)  | Wi-Fi6 | ✔   | ✘        | ✔        | 10 ✘     |
|                   |             |                       |        | ✔   | ✘        | ✔        | ✘        |
| 42 RT-AX89X       | ASUS        | AsusWrt(Linux-based)  | Wi-Fi6 |     |          |          | 10       |
| 43 E5600          | Linksys     | Linux-based           | Wi-Fi6 | ✔   | ✔        | ✔        | 10 ✔     |
| 44 E9450          | Linksys     | Linux-based           | Wi-Fi6 | ✔   | ✔        | ✔        | 10 ✔     |
|                   |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 45 QUANTUMD2G     | Wavlink     | Linux-based           | Wi-Fi5 |     |          |          | 10       |
| 46 CF-616AC       | Comfast     | OrangeOS(Linux-based) | Wi-Fi5 | ✔   | ✔        | ✔        | 10 ✔     |
| 47 DI-7003GV2∗    | D-Link      | Linux-based           | -      | ✔   | ✔        | ✔        | 1 ✔      |
| 48 E3630          | ZTE         | ZXR10ROS(Linux-based) | Wi-Fi6 | ✔   | ✘        | ✔        | 10 ✘     |
|                   |             |                       |        | ✔   | ✘        | ✔        | ✘        |
| 49 AX3000         | ZTE         | ZXR10ROS(Linux-based) | Wi-Fi6 |     |          |          | 10       |
| 50 M80∗           | IP-COM      | Linux-based           | -      | ✔   | ✔        | ✔        | 1 ✔      |
| 51 SK-WR6640X     | Skyworth    | Linux-based           | Wi-Fi6 | ✔   | ✔        | ✔        | 10 ✔     |
|                   |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 52 VX3000         | Volans      | Linux-based           | Wi-Fi6 |     |          |          | 1        |
| 53 VE5200G∗       | Volans      | Linux-based           | -      | ✔   | ✔        | ✔        | 1 ✔      |
| 54 MG1200AC       | Netcore     | NOS(Linux-based)      | Wi-Fi5 | ✔   | ✔        | ✔        | 1 ✔      |
|                   |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 55 NBR1009GPE     | Netcore     | NOS(Linux-based)      | -      |     |          |          | 1        |
| 56 Wimaster-mini∗ | Wimaster    | Linux-based           | -      | ✔   | ✔        | ✔        | 10 ✔     |
| 57 Wimaster∗      | Wimaster    | Linux-based           | -      | ✔   | ✔        | ✔        | 10 ✔     |
|                   |             |                       |        | ✔   | ✔        | ✔        | ✔        |
| 58 IK-Q90         | iKuai       | iKuaiOS(Linux-based)  | Wi-Fi6 |     |          |          | 10       |
| 59 IK-Enterprise∗ | iKuai       | iKuaiOS(Linux-based)  | -      | ✔   | ✔        | ✔        | 10 ✔     |
60 InstantOnAP22 Aruba ArubaOS(Linux-based) Wi-Fi6 ✔ ✘ ✔ 10 ✘
| EdgeRouterX∗      |             |                       |        | ✔   | ✔   | ✔   | ✔    |
| ----------------- | ----------- | --------------------- | ------ | --- | --- | --- | ---- |
| 61                | Ubiquiti    | Linux-based           | -      |     |     |     | 10   |
| 62 AX1800         | JdCloud     | Linux-based           | Wi-Fi6 | ✔   | ✔   | ✔   | 10 ✔ |
| 63 CiscoMeraki64∗ | CiscoMeraki | Linux-based           | -      | ✔   | ✘   | ✘   | - ✘  |
| 64 eeropro        | Amazon      | Linux-based           | Wi-Fi5 | ✔   | ✔   | ✔   | 10 ✔ |
|                   |             |                       |        | ✔   | ✔   | ✔   | ✔    |
| 65 GoogleWi-Fi    | Google      | ChromeOS(Linux-based) | Wi-Fi5 |     |     |     | 10   |
| 66 GL-MT3000      | GL.iNet     | Linux-based           | Wi-Fi6 | ✔   | ✔   | ✔   | 10 ✔ |
| 67 pfSense2.7.0∗  | pfSense     | FreeBSD-based         | -      | ✘   | ✘   | ✔   | 90 ✘ |
✔meansthattherouterissatisfiedwiththecondition,and✘meansthattherouterisdissatisfiedwiththecondition.
✔meansthattherouterisvulnerabletoourattack,and✘meansthattherouterisimmunetoourattack.
*meansthatthemodelisanenterpriserouterwhichdoesnotsupportWi-Fibyitselfandneedstoworktogetherwithwirelessaccesspoints.
17
