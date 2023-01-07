# Chapter 5 Link Layer
## Goals of chapter
+ understand principles behind link layer services:
    + error detection, error correction
    + sharing a broadcast channel: multiple access
    + link layer addressing
    + local area network: Ethernet, VLANs
+ instantiation, implementation of various link layer technologies

# Link Layer, LAN: outline
5.1 Introduction, services
5.2 error detection, correction
5.3 multiple access protocols
5.4 LANs
    + addressing, ARP
    + Ethernet
    + Switches
    + VLANs
5.5 Link Virtualization: MPLS
5.6 Data Center Networking
5.7 a day in the life of a web request

## Link Layer: Introduction
**Terminology**
+ Hosts and routers: **nodes**
+ communication channels that connect adjacent nodes along communication path: **links**
    + wired links
    + wireless links
    + LANs
+ layer-2 packet: **frame**, encapsulates datagram

> **Data-Link Layer** has responsibility of transferring datagram from one node to **physically adjacent** node over a link

## Link Layer: Context
+ Datagram transferred by different link protocols over different links:
    + e.g., Ethernet on first link, frame relay on intermediate links, 802.11 on last link
+ each link protocol provides different services
    + e.g., may or may not provide rdt over link
    
### Transportation Analogy
+ trip from princeton to Lausanne
    + limo: princeton to JFK
    + plane: JFK to Geneva
    + train: Geneva to Lausanne
+ tourist = **datagram**
+ transport segment = **communication link**
+ transport mode = **link layer protocol**
+ travel agent = **routing algorithm**
    
## Link Layer Services 
+ **framing, link access:**
    + encapsulate datagram into frame, adding header, trailer
    + channel access if shared medium
    + "MAC" addresses used in frame headers to identify source, destination
        + different from IP address
+ **reliable delivery between adjacent nodes**
    + we learned how to do this already (Chapter 3)
    + seldom used on low bit-error link (fiber, some twisted pair)
    + wireless links: high error rates
    + **Question**: why both link-level and end-to-end reliability?
+ **Flow Control**:
    + pacing between adjacent sending and receiving nodes
+ **Error Detection**:
    + errors caused by signal attenuation, noise.
    + receiver detects presence of errors:
       + signals sender for retransmission or drop frame
+ **Error Correction**:
    + receiver identifies **and corrects** bit errors without resorting to retransmission
+ **half-duplex and full-duplex**
    + with half-duplex, nodes at both ends of link can transmit, but not at same time

## Where is the link layer implemented?
+ in each and every host
+ link layer implemented in "adaptor" (aka **network interface card** NIC) or on a chip
    + Ethernet card, 802.11 card; Ethernet chipset
    + implements link, physical layer
+ attaches into host's system buses
+ combination of hardware, software, firmware

## Adaptors communicating
+ Sending Side:
    + encapsulates datagram in frame
    + adds error checking bits, rdt, flow control, etc.
+ Receiving Side:
    + looks for errors, rdt, flow control, etc.
    + extracts datagram, passes to upper layer at receiving side

## Error Detection
EDC = Error Detection and Correction bits (redundancy)
D = Data protected by error checking, may include header fields

+ Error Detection not 100% reliable!
    + protocol may miss some errors, but rarely
    + larger EDC field yields better detection and correction
    
## Parity Checking
**Single Bit Parity**: Detect single bit errors
**Two-Dimensional Bit Parity**: detect and correct single bit errors

## Internet Checksum (review)
**GOAL:** detect "errors" (e.g.,flipped bits) in transmitted packet
**NOTE**: used at transport layer only.

**Sender:**
+ treat segment contents as sequence of 16-bit integers
+ checksum: addition of segment contents
+ sender puts checksum value into UPD checksum field
+ 
**Receiver:**
+ compute checksum of received segment
+ check if computed checksum equals checksum field value:
    + **NO** - Error Detected
    + **YES** - No Error Detected

## Cyclic Redundancy Check
+  more powerful error-detection coding
+ view data bits, **D** as a binary number
+ choose r+1 bit pattern (generator), **G**
+ goal: choose r CRC bits, **R**, such that
    + <D,R> exactly divisible by G (modulo 2)
    + receiver knows G, divides <D,R> by G. If non-zero remainder: Error Detected!
    + can detect all burst errors less than r+1 bits
+ widely used in practice (Ethernet, 802.11 WiFi, ATM)

## CRC Example
**Want**: D.2^r XOR R = nG
**Equivalently**: D.2^r = nG XOR R
**Equivalently**: if we divide D.2^r by G, want remainder R to satisfy:
> R = remainder [(D.2^r)/G]

## Multiple Access links, protocols
Two types of "links":
+ Point-to-Point:
    + PPP for dial-up access
    + point-to-point link between Ethernet switch, host

+ **broadcast (shared wire or medium)**
    + old-fashioned Ethernet
    + upstream HFC
    + 802.11 wireless LAN

## Multiple Access Protocols
+ Single shared broadcast channel
+ two or more simultaneous transmissions by nodes:
interference
    + **collision** if node receives two or more signals at the same time

**Multiple Access Protocol**
+ distributed algorithm that determines how nodes share channel, i.e., determine when node can transmit
+ communication about channel sharing must use channel itself!
    + no out-of-band channel for coordination
    
## An Ideal Multiple Access Protocol
**Given**: broadcast channel of rate R bps
**Desired**:
1. when one node wants to transmit, it can send at rate R
2. when M nodes want to transmit, each can send at average rate R/M
3. fully decentralized:
    + no special node to coordinate transmissions
    + no synchronization of clocks, slots
4. simple

## MAC protocols: taxonomy
Three broad classes:
+ **Channel Partitioning**
    + divide channel into smaller "pieces" (time slots, frequency, code)
    + allocate piece to node for exclusive use
+ **Random Access**
    + channel not divided, allow collisions
    + "recover" from collisions
+ **"Taking Turns"**
    + nodes take turns, but nodes with more to send can take longer turns

## Channel partitioning MAC protocols: TDMA
**TDMA: Time Division Multiple Access**
+ Access to channel in "rounds"
+ each station gets fixed length slot (length=pkt trans time) in each round
+ unused slots go idle
+ example: 6-station LAN, 1,3,4 have pkt, slots 2,5,6 idle

## Channel partitioning MAC protocols: FDMA
**FDMA: Frequency Division Multiple Access**
+ channel spectrum divided into frequency bands
+ each station assigned fixed frequency band
+ unused transmission time in frequency bands go idle
+ example: 6-station LAN, 1,3,4 have pkt, frequency bands 2,5,6 idle
 
## Random Access Protocols
+ When node has packet to send
    + transmit at full channel data rate R.
    + no a-prior coordination among nodes
+ two or more transmitting nodes -> "collision",
+ **Random Access MAC Protocol** specifies:
    + how to detect collisions
    + how to recover from collisions (e.g., via delayed retransmissions)
+ examples of random access MAC protocols:
    + slotted ALOHA
    + ALOHA
    + CSMA, CSMA/CD, CSMA/CA
    
## Slotted ALOHA
**Assumptions:**
+ all frames same size
+ time divided into equal size slots (time to transmit 1 frame)
+ nodes start to transmit only slot beginning
+ nodes are synchronized
+ if 2 or more nodes transmit in slot, all nodes detect collision

**Operation:**
+ When node obtains fresh frame, transmits in next slot
    + if **no collision**: node can send new frame in next slot
    + if **collision**: node retransmits frame in each subsequent slot with probabiliy p until success

**Pros:**
+ single active node can continuously transmit at full rate of channel
+ highly decentralized: only slots in nodes need to be in sync
+ simple

**Cons:**
+ collisions, wasting slots
+ idle slots
+ nodes may be able to detect collision in less than time to transmit packet
+ clock synchronization

## Slotted ALOHA: Efficiency
**Efficiency**: long-run fraction of successful slots(many nodes, all with many frames to send)
+ suppose: N nodes with many frames to send, each transmits in slot with probability p
+ probability that given node has success in a slot = p(1-p)^N-1
+ probability that any node has a success = Np(1-p)^N-1

## Pure(unlotted) ALOHA
+ unslotted Aloha: simpler, no synchronization
+ when frame first arrives
    + transmit immediately
+ collision probability increases:
    + frame sent at t0 collides with other frames sent in [t0-11, t0+1]
    
## Pure ALOHA: Efficiency
Even worse than slotted ALOHA!

## CSMA (Carrier Sense Multiple Access)
**CSMA**: Listen before transmit:
**if channel sensed idle**: transmit entire frame
+ ** if channel sensed busy**, defer transmission
+ human analogy: don't interrupt others!

## CSMA collisions
+ **collisions can still occur**: propagation delay means two nodes may not hear each other's transmission
+ **collision**: entire packet transmission time wasted
    + distance & propagation delay play role in determining collision probability

## CSMA/CD (Collision Detection)
+ carrier sensing, deferral as in CSMA
    + collisions detected within short time
    + colliding transmissions aborted, reducing channel wastage
+ Collision Detection:
    + easy in wired LANs: measure signal strengths, compare transmitted, received signals
    + difficult in wireless LANs: received signal strength overwhelmed by local transmission strength
+ Human Analogy: The polite conversationalist

## Ethernet CSMA/CD algorithm
1. NIC receives datagram from network layer, creates frame
2. If NIC senses channel idle, start frame transmission. If NIC senses channel busy, waits until channel idle, then transmits
3. If NIC transmits entire frame without detecting another transmission, NIC is done with frame.
4. If NIC detects another transmission while transmitting, aborts and sends jam signal
5. After aborting, NIC enters **binary (exponential backoff)**:
    + after m'th collision, NIC chooses K at random from {0,1,2,3,...,2^m-1}. NIC waits k.512 bit times, returns to step 2.
    + longer backoff interval with more collisions
    
## CSMA/CD efficiency
+ T-prop = max prop delay between 2 nodes in LAN
+ T-trans = time to transmit max-size frame
efficiency = 1/((1+5t-prop)/(t-trans))
+ efficiency goes to 1
    + as t-prop goes to 0
    + as t-trans goes to infinity
+ better performance than ALOHA: and simple, cheap, decentralized

## "Taking Turns" MAC protocols
**Channel partitioning MAC protocols:**
+ share channel efficiency and fairly at high load
+ inefficient at low load: delay in channel access, 1/N bandwidth allocated even if only 1 active node

**random access MAC protocols**
+ efficient at low load: single node can fully utilize channel
+ high load: collision overhead

**"taking turns" protocols**
look for best of both worlds!

**polling**:
+ master node "invites" slave nodes to transmit in turn
+ typically used with "dumb" slave devices
+ concerns:
    + polling overhead
    + latency
    + single point of failure (master)
    
**token passing**:
+ control **token** passed from one node to next sequentially
+ token message
+ concerns:
    + token overhead
    + latency
    + single point of failure (token)
    
## "Taking Turns" MAC protocols
+ **multiple** 40Mbps downstream (broadcast) channels
    + single CMTS transmits into channels
+ **multiple** 30Mbps upstream channels
    + **multiple access**: all users contend for certain upstream channel time slots (others assigned)

**DOCSIS**: data over cable service interface spec
+ FDM over upstream, downstream frequency channels
+ TDM upstream: some slots assigned upstream slots
    + downstream MAP frame; assigns upstream slots
    + request for upstream slots (and data) transmitted random access(binary backoff) in selected slots

## Summary of MAC protocols
+ **channel partitioning**, by time, frequency or code
    + Time Division, Frequency Division
+ **random access**(dynamic),
    + ALOHA, S-ALOHA, CSMA, CSMA/CD
    + carrier sensing: easy in some technologies(wire), hard in others(wireless)
    + CSMA/CD used in Ethernet
    + CSMA/CA used in 802.11
+ **taking turns**
    + polling from central site, token passing
    + bluetooth, FDDI, token ring

## 5.4 LANs
## MAC addresses and ARP
+ 32-bit IP address:
    + network-layer address for interface
    + used for layer 3 (network layer) forwarding
+ MAC (or LAN or physical or Ethernet) address:
    + function: used 'locally' to get frame from one interface to another physically-connected interface (same network, in IP-addressing sense)
    + 48 bit MAC address (for most LANs) burned in NIC ROM, also sometimes software settable
    + e.g.: 1A-2F-BB-76-09-AD

## LAN addresses and ARP
each adapter on LAN has unique **LAN** address
+ MAC address allocation administered by IEEE
+ manufacturer buys portion of MAC address space (to assume uniqueness)
+ analogy:
    + MAC address: like Social Security Number
    + IP address: Like postal address
+ MAC flat address -> portability
    + can move LAN card from one LAN to another
+ IP hierarchical address not portable
    + address depends on IP subnet to which node is attached

## ARP: address resolution protocol
**Question:** How to determine interface's MAC address, knowing its IP address?
**ARP table**: each IP node (host, router) on LAN has table
    + IP/MACC address mappings for some LAN nodes: <IP address; MAC address; TTL>
    + TTL(TimeToLive): time after which address mapping will be forgotten(typically 20 minutes)


## ARP protocol: same LAN
+ A wants to send datagram to B
  + B's MAC address not in A's ARP table
+ A **broadcasts** ARP query packet, containing B's IP address
  + destination MAC address = FF-FF-FF-FF-FF-FF
  + all nodes on LAN receive ARP query
+ B receives ARP packet, replies to A with its(B's) MAC address
  + frame sent to A's MAC address(unicast)
+ A caches IP-to-MAC address pair in its ARP table until information becomes old(times out)
  + soft state: information that times out(goes away) unless refreshed
+ ARP is "Plug-and-Play":
  + nodes create their ARP tables without intervention form net administrator

## Addressing: routing to another LAN
walk through: **send datagram from A to B via R**
+ focus on addressing - at IP(datagram) and MAC layer(frame)
+ assume A knows B's IP address
+ assume A knows IP address of first hop router, R (how?)
+ assume A knows R's IP address (how?)

+ A creates IP datagram with IP source A, destination B
+ A creates link-layer frame with R's MAC address as destination, frame contains A-to-B IP datagram
+ frame sent from A to R
+ frame received at R, datagram removed, passed up to IP
+ R forwards datagram with IP source A, destination B
+ R creates link-layer frame with B's MAC address as destination, frame contains A-to-B IP datagram

## Ethernet
"dominant" wired LAN technology
+ cheap $20 for NIC
+ first widely used LAN technology
+ simpler, cheaper than token LANs and ATM
+ kept up with speed race: 10Mbps - 10Gbps

## Ethernet: physical topology
+ **bus**: popular through mid 90s
  + all nodes in same collision domain (can collide with each other)

+ **star**: prevails today
  + active **switch** in center
  + each "spoke" runs a (separate) Ethernet protocol (nodes do not collide with each other)

## Ethernet frame structure
Sending adapter encapsulates IP datagram (or other network layer protocol packet) in **Ethernet Frame**

**preamble**
+ 7 bytes with pattern 10101010 followed by one byte with pattern 10101011
+ used to synchronize receiver, sender clock rates
+ **address**: 6 byte source, destination MAC addresses
  + if adapter receives frame with matching destination address, or with broadcast address (e.g. ARP packet), it passes data in frame to network layer protocol
  + otherwise, adapter discards frame
+ **type**: indicates higher layer protocol (mostly IP but others possible, e.g. Novell IPX, AppleTalk)
+ **CRC**: cyclic redundancy check at receiver
  + error detected: frame is dropped

## Ethernet: unreliable, connectionless
+ **connectionless**: no handshaking between sending and receiving NICs
+ **unreliable**: receiving NIC doesn't send ACKs or NACKs to sending NIC
  + data in dropped frames recovered only if initial sender uses higher layer rdt(e.g. TCP), otherwise dropped data lost
+ Ethernet's MAC protocol: unslotted **CSMA/CD w'th binary backoff**

## 802.3 Ethernet standards: link & Physical layers
+ **many** different Ethernet standards
  + common MAC protocol and frame format
  + different speeds: 2Mbps, 10Mbps, 100Mbps, 1Gbps, 10Gbps
  + different physical layer media: fiber, cable

## Ethernet switch
+ **link-layer device: takes an active role**
  + store, forward Ethernet frames
  + examine incoming frame's MAC address, **selectively** forward frame to one-or-more outgoing links when frame is to be forwarded on segment, uses CSMA/CD to access segment
+ **transparent**
  + hosts are unaware of presence of switches
+ **plug-and-play, self-learning**
  + switches do NOT need to be configure

## Switch: multiple simultaneous transmissions
+ hosts have dedicated, direct connection to switch
+ switches buffer packets
+ Ethernet protocol used on each incoming link, but no collisions; full duplex
  + each link is its own collision domain
+ **switching**: A-to-A's and B-to-B can transmit simultaneously, without collisions

## Switch forwarding table
**Question**: How does switch know A's reachable via interface 4, B's reachable via interface 5?
**Answer**: each switch has a **switch table**, each entry:
  + (MAC address of host, interface to reach host, time stamp)
  + looks like a routing table

**Question**: How are entries created, maintained in switch table?
  + something like a routing protocol?

## Switch: self-learning
+ switch **learns** which hosts can be reached through which interfaces
  + when frame received, switch "learns" location of sender: incoming LAN segment
  + records sender/location pair in switch table

## Switch: frame filtering/forwarding
when frame received at switch:
1. record incoming link, MAC address of sending host
2. index switch table using MAC destination address
3. if entry found for destination then if destination on segment from which frame arrived then drop frame else forward frame on interface indicated by entry. else flood.

## Self-learning, forwarding: example
+ frame destination, A', location unknown: **flood**
+ destination A location known: **selectively send on just one link**

## Interconnecting switches
+ switches can connected together

## Switches vs Routers
both are store-and-forward:
+ **routers**: network-layer devices (examine network layer headers)
+ **switches**: link-layer devices (examine link layer headers)

both have forwarding tables:
+ **routers**: compute tables using routing algorithms, IP addresses
+ **switches**: learn forwarding table using flooding, learning, MAC addresses

## VLANs: motivation
**consider**:
+ CS user moves office to EE, but wants connect to CS switch?
+ single broadcast domain:
  + all layer-2 broadcast traffic (ARP, DHCP, unknown location of destination MAC address) must cross entire LAN
  + security/privacy, efficiency issues

## VLAN
**port-based VLAN**: switch ports grouped (by switch management software) so that **single** physical switch operates as **multiple** virtual switches.
**Virtual Local Area Network**: switch(es) supporting VLAN capabilities can be configured to define multiple **virtual** LANs over single physical LAN infrastructure.

## Port-based VLAN
+ **traffic isolation**: frames to/from ports 1-8 can only reach ports 1-8
  + can also define VLAN based on MAC addresses of endpoints, rather than switch port
+ **dynamic membership**: ports can be dynamically assigned among VLANs
+ **forwarding between VLANs**: done via routing (just as with separate switches)
  + in practice vendors sell combined switches plus routers

## VLANS spanning multiple switches
+ **trunk port**: carries frames between VLANs defined over multiple physical switches
  + frames forwarded within VLAN between switches can't be vanilla 802.1 frames (must carry VLAN ID info)
  + 802.1q protocol adds/removed additional header fields for frames forwarded between trunk ports

## Multiprotocol lable switching (MPLS)
+ **Initial goal**: high-speed IP forwarding using fixed length table (instead of IP address)
  + fast lookup using fixed length identifier (rather than shortest prefix matching)
  + borrowing ideas from Virtual Circuit (VC) approach
  + but IP datagram still keeps IP address!

## MPLS capable routers
+ a.k.a. label-switched router
+ forward packets to outgoing interface based only on label value (don't inspect IP address)
  + MPLS forwarding table distinct from IP forwarding tables
+ **flexibility**: MPLS forwarding decisions can differ from those of IP
  + use destination and source addresses to route flows to same destination differently (traffic engineering)
  + re-route flows quickly if link fails: pre-computed backup paths(useful for VoIP)

## MPLS vs IP paths
+ **IP routing**: path to destination determined by destination address alone
+ **MPLS routing**: path to destination can be based on source and destination address
  + **fast reroute**: precompute backup routes in case of link failure

## MPLS signaling
+ modify OSPF, IS-IS link-state flooding protocols to carry info used by MPLS routing
  + e.g., link bandwidth, amount of "reserved" link bandwidth
+ entry MPLS router uses RSVP-TE signaling protocol to set up MPLS forwarding at downstream routers

## Data Center Networks
+ 10's to 100's of thousands of hosts, often closely coupled, in close proximity:
  + e-business (e.g. Amazon)
  + content-servers (e.g. YouTube, Apple, Microsoft)
  + search engines, data mining(e.g. Google)

+ Challenges:
  + multiple applications, each serving massive numbers of clients
  + managing/balancing load, avoiding processing, networking, data bottlenecks

**load balancer: application-layer routing**
  + receives external client requests
  + directs workload within data center
  + returns results to external client (hiding data center internals from client)

+ rich interconnection among switches, racks:
  + increased throughput between racks (multiple routing paths possible)
  + increased reliability via redundancy

## Synthesis: a day in the life of a web request
+ journey down protocol stack complete!
  + application, transport, network, link
+ putting-it-all-together: Synthesis!

**GOAL**: identify, review, understand protocols (at all layers) involved in seemingly simple scenario: requesting www page
**Scenario**: student attaches laptop to campus network, requests/receives www.google.com
+ connecting laptop needs to get its own IP address, address of first-hop router, address of DNS server: use **DHCP**
+ DHCP requests **encapsulated** in **UDP** in **IP**, encapsulated in 802.3 Ethernet
+ Ethernet frame **broadcast** (destination: FFFFFFFFFFFF) on LAN, received at router running **DHCP** server
+ Ethernet **demuxed** to IP demuxed, UPD demuxed to DHCP
+ DHCP server formulates **DHCP, ACK** containing client's IP address, IP address of first-hop router for client, name & IP address of DNS server
+ encapsulation at DHCP server, frame forwarded(switch learning) through LAN, demultiplexing at client
+ DHCP client receives DHCP ACK reply
Client now has IP address, knows name & address of DNS server, IP address of its first-hop router
+ before sending **HTTP** request, need IP address of www.google.com: **DNS**
+ DNS query created, encapsulated in UDP, encapsulated in IP, encapsulated in Tth. To send frame to router, need MAC address of router interface: **ARP**
+ **ARP query** broadcast, received by router, which replies with **ARP reply** giving MAC address of router interface
+ client now knows MAC address of first hop router, so can now send frame containing DNS query
+ IP datagram containing DNS query forwarded via LAN switch from client to 1st hop router
+ IP datagram forwarded from campus network into comcast network, routed(tables created by RIP, OSPF,IS-IS and/or BGP routing protocols) to DNS server
+ dmeux'ed to DNS server
+ DNS server replies to client with IP address of www.google.com
+ to send HTTP request, client first opens **TCP socket** to web server
+ TCP **SYN segment** (step 1 in 3-way handshake) inter-domain routed to web server
+ web server responds with **TCP SYNACK** (step 2 in 3-way handshake)
+ TCP **connection established !**
+ web page **finally** displayed
+ **HTTP request** sent into TCP socket
+ IP datagram containing HTTP request routed to www.gooogle.com
+ web server responds with **HTTP reply** (containing web page)
+ IP datagram containing HTTP reply routed back to client

## Chapter 5: Summary
+ principles behind data link layer services:
  + error detection, correction
  + sharing a broadcast channel: multiple access
  + link layer addressing
+ instantiation and implementation of various link layer technologies
  + Ethernet
  + switched LANS, VLANS
  + virtualized networks as a link layer: MPLS
+ Synthesis: a day in a life of web request

Fin.
