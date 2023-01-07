# Chapter 4: Network Layer
+ 4.1 Introduction
+ 4.2 virtual circuit and datagram networks
+ 4.3 What's inside a router
+ 4.4 IP: Inter Protocol
+ 4.5 routing algorithm
+ 4.6 routing in the internet
+ 4.7 broadcast and multicast routing

Chapter Goals:
+ Understand principles behind network layer services:
+ network layer service models
+ forwarding versus routing
+ how a router works
+ routing(path selection)
+ broadcast, multicast
+ instantiation, implementation in the internet

## Network Layer
+ transport segment from sending to receiving host
+ on sending side encapsulates segments into diagrams
+ on receiving side, delivers segments to transport layer
+ network layer protocols in **every** host, router
+ router examines header fields in all IP datagrams passing through it

## Two key network-layer functions
1. Forwarding:
Move packets from router's input to appropriate router output

2. Routing:
Determine route taken by packets form source to destination.

## Interplay between routing and forwarding
routing algorithm determines end-end path through network

Forwarding table determines local forwarding at this router

## Connection Setup
3rd import function in some network architectures:
+ ATM, frame relay, X.25
+ before datagrams flow, two end hosts and intervening routers establish virtual connection
  + routers get involved
+ Network vs Transport layer connection service:
  + **Network**: between two hosts (may also involve intervening routers in case of VCs)
  + **Transport**: between two processes

## Network Service Model
**Question**: What **service model** for "channel" transporting datagrams from sender to receiver?

Example services for individual datagrams:
+ guaranteed delivery
+ guaranteed delivery with less than 40 msec delay

Example services for a flow of datagrams:
+ in-order datagram delivery
+ guaranteed minimum bandwidth to flow
+ restrictions on changes in inter-packet spacing

## 4.2 virtual circuit and datagram networks
## Connection, Connection-less service
+ **datagram** network provides network-layer **connectionless** service
+ **virtual-circuit** network provides network-layer **connection** service
+ analogous to TCP/UPD connection-oriented/connectionless transport-layer services, but:
  + **service**: host-to-host
  + **no choice**: network provides one or the other
  + **implementation**: in network core

## Virtual Circuits
"source-to-destination path behaves much like telephone circuit"
+ performance wise
+ Network actions along source-to-destination path

+ call setup, tear down for each call before data can flow
+ each packet carries VC identifier (not destination host address)
+ every router on source-destination path maintains "state" for each passing connection
+ link, router resources (bandwidth, buffers) may be allocated to VC (dedicated resources = predictable service)

## VC implementation
a VC consists of:
1. **path** from source to destination
2. **VC numbers**, one number for each link along path
3. **entries in forwarding tables** in routers along path

+ packet belonging to VC carries VC number (rather than destination address)
+ VC number can be changed on each link
  + new VC number comes from forwarding table

## Virtual circuits: singaling protocols
+ used to setup, maintain teardown VC
+ used in ATM, frame-relay, X.25
+ not used in today's internet

## Datagram Networks
+ no call setup at network layer
+ routers: no state about end-to-end connections
  + no network-level concept of "connection"
+ packets forwarded using destination host address

## Datagrams forwarding table
4 billion IP addresses, so rather list individual destination address list range of addresses(aggregate table entries)

## Longest prefix matching
when looking for forwarding table entry for given destination address, use **longest** address prefix that matches destination address.

## Datagram or VC network: Why?
**Internet(datagram)**
+ data exchange among computers
  + "elastic" service, no strict timing required.
+ many link types
  + different characteristics
  + uniform service difficult
+ "smart" end systems(computers)
  + can adopt, perform control, error recovery
  + **simple inside network, complexity at "edge"**

**ATM (VC)**
+ evolved from telephony
+ human conversation:
  + strict timing, reliability requirements
  + need for guaranteed service
+ "dumb" end systems
  + telephones
  + **complexity inside network**

## 4.3 What's inside a router
## Router Architecture Overview
Two key router functions:
+ run routing algorithms/protocols (RIP, OSPF, BGP)
+ forwarding datagrams from incoming to outgoing link

## Input port functions
**physical layer**: bit-level perception
**data link layer**: Ethernet
**Decentralized Switching**:
+ given datagram destination, lookup output port using forwarding table in input port memory
+ goal: complete input port processing at "line speed"
+ queuing: if datagrams arrive faster than forwarding rate into switch fabric

## Switching Fabrics
+ transfer packet from input buffer to appropriate output buffer
+ Switching Rate: rate at which packets can be transferred from inputs to outputs
  + often measured as multiple of input/output line rate
  + N inputs: Switching rate N times line rate desirable
+ Three types of switching fabrics

## Switching via memory
**first generation routers:**
+ traditional computers with switching under direct control of CPU
+ packet copied to system's memory
+ CPU extracts destination address from packet's header, looks up output port in forwarding table, copies to output port
+ speed limited by memory bandwidth (2 bus crossings per datagram)
+ one packet at a time

## Switching via a bus
+ datagram from input port memory to output port memory via a shared bus
+ **bus contention**: switching speed limited by bus bandwidth
+ one packet a time
+ 32 Gbps bus, Cisco 5600: sufficient speed for access and enterprise routers

## Switching via interconnection network
+ forwards multiple packets in parallel
+ banyan networks, crossbar, other interconnection nets initially developed to connect processors in multiprocessor
+ when packet from port A needs to forwarded to port Y, controller closes cross point at intersection of two buses
+ advanced desgin: fragmenting datagram into fixed length cells, switch cells through the fabric

## Output ports
+ **buffering**: required when datagrams arrive from fabric faster than the transmission rate
+ ** scheduling discipline**: chooses among queued datagrams for transmission

## Output port queueing
+ suppose R-switch is N times faster than R-line
+ still have output buffering when multiple inputs send to same output
+ **queueing (delay) and loss due to output port buffer overflow!**


## How much buffering?
+ RFC 3439 rule of thumb:
average buffering equal to "typical" RTT (say 250 msec) times link capacity C
  + e.g., C=10Gbps link: 2.5 Gbit buffer
+ recent recommendation: with N flows, buffering equal to: (RTT.C)/sqrt(N)

## Input port queueing
+ fabric slower than input ports combined => queueing may occur at input queues
  + **queueing delay and loss due to input buffer overflow!**
+ **Head-of-the-Line(HOL) blocking**: queued datagram at front of queue prevents others in queue from moving forward


## 4.4 IP: Internet Protocol
## The Internet Network Layer
host, router network layer functions:

## IP fragmentation, reassembly
+ network links have MTU(max.transfer size) - largest possible link-level frame
  + different link types, different MTUs
+ large IP datagram divided ("fragmented") within net
  + one datagram becomes several datagrams
  + "reassembled" only at final destination
  + IP header bits used to identify, order related fragments


## IP addressing: introduction
+ **IP Address**: 32-bit identifier for host, router interface
+ **interface**: connection between host/router and physical link
  + routers typically have multiple interfaces
  + host typically has one active interface (wired Ethernet, wireless 802.11)
+ **one IP address associated with each interface**
**Question**: How are interfaces actually connected?
**Answer**: we'll learn about that in chapter 5 and 6
**For Now**: don't need to worry about how one interface is connected to another (with no intervening router)

## Subnets
+ **IP address**
  + subnet part - high order bits
  + host part - low order bits
+ **what's a subnet?**
  + device interfaces with same subnet part of IP address
  + can physically reach each other **without intervening router**

**Recipe**
+ to determine the subnets, detach each interface from its host or router, creating islands of isolated networks
+ each isolated network is called a **subnet**

## IP addressing: CIDR
**CIDR**: Classless Inter Domain Routing
  + subnet portion of address of arbitrary length
  + address format: **a.b.c.d/x**, where x is number of bits in subnet portion of address

## IP addresses: How to get one?
**Question**: how does network get subnet part of IP address?
**Answer**: gets allocated portion of its provider ISP's address space

## Hierarchical addressing: route aggregation
Hierarchical addressing allows efficient advertisement of routing information

## IP addressing: How to get a block?
**Question**: How does an ISP get block of addresses?
**Answer**: **ICANN**: Internet Corporation for Assigned Names and Numbers [ICANN](http://www.icann.org)
  + allocates addresses
  + manages DNS
  + assigns domain names, resolves disputes

## IP addresses: How to get one?
**Question**: How does a host get IP address?
+ hard-coded by system admin in a file
  + windows: control-panel -> network -> configuration -> tcp/ip -> properties
  + UNIX: /etc/rc.config
+ **DHCP**: Dynamic Host configuration Protocol
dynamically gets address from as server
  + plug and play

## DHCP: Dynamic Host Configuration Protocol
**GOAL**: allow host to dynamically obtain its IP address from network server when it joins network
  + can renew its lease address in use
  + allows reuse of addresses (only hold address while connected/"ON")
  + support for mobile users who want to join network (more shortly)
**DHCP overview**:
  + host broadcasts "**DHCP discover**" message(optional)
  + DHCP server responds with "**DHCP offer**" message(optional)
  + host requests IP address: "**DHCP request**" message
  + DHCP server sends address: "**DHCP ack**" message

## DHCP: more than IP addressing
DHCP returns:
+ IP address
+ address of first-hop router for client
+ name and IP address of DNS server
+ network mask (indicating network versus host portion of address)

## DCHP: example
+ connecting laptop needs its IP address, address of first-hop router, address of DNS server: use DHCP
+ DHCP request encapsulated in UDP, encapsulated in IP, encapsulated in 802.3 Ethernet
+ Ethernet frame broadcast (destination: FFFFFFFFFFFF) on LAN, received at router running DHCP server
+ Ethernet demuxed to IP demuxed, UDP demuxed to DHCP
+ DHCP server formulates DHCP ACK containing client's IP address, IP address of first-hop router for client, name & IP address of DNS server
+ encapsulation of DHCP server, frame forwarded to client, demuxing up to DHCP at client
+ client now knows its IP address, name and IP address of DNS server, IP address of its first-hop router

## NAT: Network Address Translation
**Motivation**: local network uses just one IP address as far as outside world is concerned:
  + range of addresses not needed from ISP: just one IP address for all devices
  + can change addresses of devices in local network without notifying outside world
  + can change ISP without changing addresses of devices in local network
  + devices inside local network not explicitly addressable, visible by outside world(a security plus)

## NAT: Network Address Translation
**Implementation**: NAT router must:
+ **outgoing datagrams:** **replace** (source IP address, port Number) of every outgoing datagram to (NAT IP address, new port Number) ... remote clients/servers will respond using(NAT IP address, new port number) as destination address
+ **remember (in NAT translation table)** every (source IP address, port number) to (NAT IP address, new port number) translation pair
+ **incoming datagrams: replace** (NAT IP address, new port number) i destination fields of every incoming datagram with corresponding (source IP address, port number) stored in NAT table
+ 16-bit port number field:
  + 60000 simultaneous connections with a single LAN-side address!
+ NAT is controversial:
  + routers should only process up to layer 3
  + violates end-to-end argument
    + NAT possibility must be taken into account by app designers, e.g., P2P applications
  + address shortage should instead be solved by IPv6

## NAT traversal problem
+ client wants to connect to server with address 10.0.0.1
  + server address 10.0.0.1 local to LAN (client can't use it as destination address)
  + only one externally visible NATeed address: 138.76.29.7
+ **Solution 1**: statically configure NAT to forward incoming connection requests at given port to server
  + e.g., (123.76.29.7, port 25000) always forwarded to 10.0.0.1 port 25000
+ **Solution 2**: Universal Plug and Play (UPnP) Internet Gateway Device (IGP) Protocol. Allows NATed host to:
  + learn public IP address (138.76.29.7)
  + add/remove port mappings (with lease times)
+ **Solution 3**: relaying (used in Skype)
  + NATed client establishes connection to relay
  + external client connects to relay
  + relay bridges packets between to connections

## ICMP: Internet Control Message Protocol
+ used by hosts & routers to communicate network level information
  + error reporting: unreachable host, network, port, protocol_
  + echo request/reply (used by ping)
+ network-layer "above" IP:
  + ICMP messages carried in IP datagrams
+ **ICMP message**: type, code plus first 8 bytes of IP datagram causing error

## Traceroute and ICMP
+ source sends series of UDP segments to destination
  + first set has TTL = 1
  + second set has TTL = 2
  + unlikely port number
+ when n'th set of datagrams arrives to n'th router:
  + router discards datagrams
  + and sends source ICMP messages (type 11, code 0)
  + ICMP messages includes name of router & IP address
+ when ICMP messages arrives, source records RTTs
**Stopping Criteria**:
+ UDP segment eventually arrives at destination host
+ destination returns ICMP "port unreachable" message (type 3, code 3)
+ source stops

## IPv6: Motivation
+ **initial motivation**: 32-bit address space soon to be completely allocated.
+ additional motivation:
  + header format helps speed processing/forwarding
  + header changes to facilitate QoS

**IPv6 datagram format**:
+ fixed-length 40-byte header
+ no fragmentation allowed

## IPv6 datagram format
+ **priority**: identify priority among datagrams in flow
+ **flow label**: identify datagrams in same "flow".
+ **next header**: identify upper layer protocol for data

## Other changes from IPv4
+ **checksum**: removed entirely to reduce processing time at each hop
+ **options**: allowed, but outside of header, indicated by "Next Header" field
+ **ICMPv6**: new version of ICMP
  + additional message types, e.g. "Packet Too Big"
  + multicast group management functions

## Transition from IPv4 to IPv6
+ not all routers can be upgraded simultaneously
  + no "flag days"
  + how will network operate with mixed IPv4 and IPv6 routers?
+ **tunneling**: IPv6 datagram carried as payload in IPv4 datagram among IPv4 routers

## 4.5 Routing Algorithms
## Routing algorithm classification
**Question**: Global or decentralized information?

**Global**:
+ all routers have complete topology, link cost info
+ **"link state" algorithms**

**Decentralized**:
+ router knows physically-connected neighbors, link costs to neighbors
+ iterative process of computation, exchange of info with neighbors
+ **"distance vector" algorithms**

**Question**: Static or Dynamic?
**static**:
+ routes change slowly over time

**dynamic**:
+ Routes change more quickly
  + periodic update
  + in response to link cost change

## A link-state Routing Algorithm
### Djikstra's Algorithm
+ net topology, link costs known to all nodes
  + accomplished via "link state broadcast"
  + all nodes have same info
+ computes least cost paths from one node ('source') to all other nodes
  + gives **forwarding table** for that node
+ Iterative: after k iterations, know least cost path to k destinations

### Notation
+ **c(x,y)**: link cost from node x to y;
+ **D(v)**: current value of cost of path from source to destination
+ **p(v)**: predecessor node along path from source to v
+ **N'**: set of nodes whose least cost path definitively known

## Djikstra's Algorithm, discussion
**Algorithm complexity**: n nodes
+ each iteration: need to check all nodes, w, not in N
+ n(n+1)/2 comparisons: O(n^2)
+ more efficient implementations possible: O(nLog n)

**Oscillations Possible**:
+ e.g., support link cost equals amount of carried traffic

 
# up until slide 84 of 155.
# Believe rest of this chapter was disregarded.
