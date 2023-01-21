# Chapter 3: Transport Layer
Chapter 3 Outline
3.1 Transport-Layer Services
3.2 multiplexing and demultiplexing
3.3 connectionless transport: UDP
3.4 principles of reliable data transfer
3.5 connection-oriented transport: TCP
3.6 principles of congestion control
3.7 TCP congestion control

Understanding principles behind transport layer:
+ multiplexing, demultiplexing
+ reliable data transfer
+ flow control
+ congestion control

learn about internet transport layer protocols:
+ UDP: connectionless transport
+ TCP: connection-oriented reliable transport
  + TCP congestion control

+ Transport services and protocols provide ***logical*** communication between app processes running on different hosts
+ Transport protocols run in end systems
  + send side: breaks app messages into segments, passes to network layer
  + rcv  side: reassembles segments into messages, passes to app layer
+ More than one transport protocol available to apps
  + Internet: TCP and UDP

## Difference between Transport and Network Layer
Network   Layer: logical communication between hosts
Transport Layer: logical communication between processes
Relies on, enhances, network layer services

## Internet transport-layer protocols
+ **TCP**: reliable, in-order delivery
  + congestion control
  + flow control
  + connection setup

+ **UDP**: unreliable, unordered delivery
  + no-frills extension of "best-effort" IP

+ services not available: 
1. Delay Guarantees
2. Bandwidth Guarantees

## Multiplexing/Demultiplexing

**Multiplexing** at sender: 
handle data from multiple sockets, add transport header(later used for demultiplexing)

**Demultiplexing** at receiver:
use header info to deliver received segments to correct socket

## How demultiplexing works
+ Host receives IP datagrams
  + each datagram has **source IP address**, **destination IP address**
  + each datagram carries **one transport-layer segment**
  + each segment has **source, destination port number**
+ Host uses **IP** address & port numbers to direct segment to appropriate socket

**NOTE**: Segment in datagram, as well as addresses.

**NOTE**: There are 2 types of Demultiplexing:
1. connectionless
2. connection-oriented

## Connection-Oriented Demux
+ TCP socket identified by 4-tuple:
  + source IP address
  + source port number
  + dest IP address
  + dest port number

+ Demux: receiver uses all four values to direct segment to appropriate socket

+ Server host may support many simultaneous TCP sockets:
  + each socket identified by its own 4-tuple

+ web servers have different sockets for each connection client
  + non-persistent HTTP will have different socket for each request

## Connectionless Demultiplexing
+ Recall: created socket hast host-local port number
+ when host receives UDP segment:
  + checks destination port number in segment => (IP datagrams with same destination port number, but different source IP addresses and/or source port numbers will be directed to same socket at destination)
  + directs UDP segment to socket with that port number
+ Recall: when creating datagram to send into UDP socket, must specify
  + destination IP address
  + destination port number

## UDP: User Datagram Protocl [RFC 768]
+ "no frills", "bare bones" internet transport protocol
+ "best effort" service, UDP segments may be:
  + lost 
  + delivered out-of-oreder to app
+ connectionless:
  + no handshaking between UDP sender and receiver
  + each UDP segment handled independently of others
+ UDP use:
  + streaming multimedia
  + DNS
  + SNMP
+ Reliable transfer over UDP:
  + add reliablity at application layer
  + application-specific error recovery

**NOTE**: Why is there a UDP?
+ no connection establishment (which can add delay)
+ simple: no connection state at sender/receiver
+ small header size -> 64bits or 8 bytes
+ no congestion control: UDP can blast away as fast as desired

## UDP checksum
**GOAL:** detect "errors"(e.g. flipped bits) in transmitted segment

Sender:
+ treat segment contents, including header fields, as sequence of 16-bit integers
+ checksum: addition(one's complement sum) of segment contents
+ sender puts checksum value into UDP checksum field

Receiver:
+ compute checksum of received segment
+ check if computed checksum equals checksum field value: 
  + **NO**  - error detected
  + **YES** - no error detected

**NOTE**: when adding numbers, a carryout from the most significant bit needs to be added to the result.

## principles of reliable data transfer

+ Important in application, transport, link layers
+ characteristics of unreliable channel will determine complexity of **reliable data transfer** protocol (rdt)

Reliable Data Transfer: Getting Started
we'll:
+ incrementally develop sender, receiver sides of Reliable Data Transfer protocol (rdt)
+ consider only unidirectional data transfer
  + but control info will flow on both directions
+ Use finite state machines(FSM) to specify sender, receiver


## rdt1.0: reliable transfer over a reliable channel
+ Underlying channel perfectly reliable
  + no bit errors
  + no loss of packets

+ seperate FSMs for sender, receiver:
  + sender sends data into underlying channel
  + receiver reads data from underlying channel

## rdt2.0: channel with bit errors
+ underlying channel may flip bits in packet
  + checksum to detect bit errors

+ the question: how to recover from errors: How do humans recover from "errors" during conversation?
  + acknowledgements(ACKs): receiver explicitly tells sender that pkt received OK
  + negative acknowledgements(NAKs): receiver explicitly tells sender that pkt had errors
  + sender retransmits pkt on receipt of NAK

+ new mechanism in rdt2.0:
  + error detection
  + feedback: control msgs(ACK, NAK) from receiver to sender


## rdt2.0 has a fatal flaw
What happens if ACK/NAK corrupted?
+ Sender doesn't know what happened at receiver!
+ can't just retransmit: possible duplicate

Handling duplicates:
+ sender retransmits current pkt if ACK/NAK corrupted
+ Sender adds sequence number to each pkt
+ receiver discards (doesn't deliver up) duplicate pkt

Stop and Wait:
+ Sender sends one packet, then waits for receiver response.

## rdt2.1: sender, handles garbled ACK/NAK
**Sender:**
+ seq number added to pkt
+ two sequence numbers (0,1) will suffice. Why?
+ must check if received ACK/NAK corrupted
+ twice as many states
  + state must "remember" whether "expected" pkt should have sequence number of 0 or 1

**Receiver:**
+ must check if received packet is duplicate
  + state indicates whether 0 or 1 is expected pkt sequence number

**NOTE**: receiver can not know if its last ACK/NAK received OK at sender

## rdt2.2: a NAK-free protocol
+ same functionality as rdt2.1, using ACKs only
+ instead of NAK, receiver sends ACK for last pkt received OK
  + receiver must explicitly include sequence number of pkt being ACKed
+ duplicate ACK at sender results in same action as NAK: retransmit current pkt

## rdt3.0 
+ rdt3.0 is correct, but performance stinks
+ e.g.: 1 Gbps link, 15 ms prop.delay, 8000 bit packet:
  D-trans = L/R = 8000 bits/ 10^9 bits/sec = 8 micro seconds

+ U-sender: utilization - fraction of time sender busy sending
  U-sender = (L/R) / (RTT + L / R) = .008/30.008 = 0.00027

+ if RTT=30 msec, 1KB pkt every 30 msec: 33kB/sec thruput over 1Gbps link
+ network protocol limits use of physical resources!

**selective repeat:** imagine we have sent 100 pkts and we have received a NAK for 40th, here we will retransmit only the 40th pkt.

## Pipelined prtocols
**pipelining:** Sender allows multiple, "in-flight", yet-to-be-acknowledged pkts
+ range of sequence numbers must be increased
+ buffering at sender and/or receiver

**TWO GENERIC FORMS OF PIPELINED PROTOCOLS:** 
1) Go-Back-N
2) Selective Repeat

### Go-Back-N (GBN)
+ sender can have up to N unpacked packets in pipeline
+ receiver only sends **cumulative ack**
  + doesn't ack packet if there's a gap
+ sender has time for oldest unpacked packet
  + when timer expires, retransmit all unacked packets

### Selective Repeat
+ sender can have up to N unacked packets in pipeline
+ receiver sends **individual ack** for each packet
+ sender maintains timer for each unacked packet
  + when timer expires, retransmit only that unacked packet


## Selective Repeat
+ receiver individually acknowledges all correctly received pkts
  + buffers pkts, as needed, for eventual in-order delivery to upper layer
+ sender only resends pkts for which ACK not received
  + sender timer for each unACKed pkt
+ sender window
  + N consecutive sequence numbers
  + limits sequence numbers of sent, unACKed pkts


## Selective Repeat-Sender/Receiver
> sender
data from above:
+ if next available sequence number in window, send pkt

timeout(n):
+ resend pkt n, restart timer

ACK(n):
+ mark pkt n as received
+ if n smallest unACKed pkt, advance window base to next unACKed sequence number


> receiver
pkt n in:
+ send ACK(n)
+ out-of-order: buffer
+ in-order: deliver (also deliver buffered, in-order pkts), advance window to next not-yet-received pkt

pkt n in
+ ACK(n)

otherwise:
+ ignore

## 3.5 connection-oriented transport: TCP
+ segment structure
+ reliable data transfer
+ flow control
+ connection management

## TCP: Overview
+ point-to-point:
  + one sender, one receiver

+ reliable, in-order byte stream:
  + no "message boundaries"

+ pipelined:
  + TCP congestion and flow control set window size

+ full duplex data:
  + bi-directional data flow in same connection
  + MSS: maximum segment size

+ connection-oriented:
  + handshaking inits sender, receiver state before data exchange

+ flow controlled:
  + sender will not overwhelm receiver

## TCP: seq.numbers, ACKs
sequence numbers:
  + byte stream "number" of first byte in segment's data

acknowledgements:
  + sequence number of next byte expected from other side
  + cumulative ACK

**Question:**How receiver handles out-of-order segments
**Answer:** TCP spec doesn't say, up to implementor


## TCP: round trip time, timeout
**Question:**How to set TCP timeout value?
+ longer than RTT but RTT varies
+ too short: premature timeout, unnecessary retransmissions
+ too long: slow reaction to segment loss

**Question:**How to estimate RTT?
+ SampleRTT: measured time from segment transmission until ACK receipt
  + ignore retransmissions
+ SampleRTT will vary, want estimated RTT "smoother"
  + average several recent measurements, not just current SampleRTT

## TCP: reliable data transfer
+ TCP creates rdt service on top of IP's unreliable service
  + pipelined segments
  + cumulative acks
  + single retransmission timer

+ retransmissions triggered by:
  + timeout events
  + duplicate acks

+ let's initially consider simplified TCP sender:
  + ignore duplicate acks
  + ignore flow control, congestion control


## TCP: sender events
data received from app:
  + create segment with sequence number
  + sequence number is byte-stream number of first data byte in segment
  + start timer if not already running
    + think of timer as for oldest unacked segment
    + expiration interval: TimeOutInterval

timeout:
  + retransmit segment that caused timeout
  + restart timer

ack received:
  + if ack acknowledges previously unacked segments
    + update what is known to be ACKed
    + start timer if there are still unacked segments


## TCP: fast retransmit
If sender receives 3 ACKs for same data ("triple duplicate ACKs"), resend unacked segment with smallest sequence number
  + likely that unacked segment lost, so don't wait for timeout

+ time-out period often relatively long:
  + long delay before resending lost packet

+ detect lost segments via duplicate ACKs.
  + sender often sends many segments back-to-back
  + if segment is lost, there will likely be many duplicate ACKs.


## TCP: flow control
Flow Control: 
Receiver controls sender, so sender won't overflow receiver's buffer by transmitting too much, too fast

+ receiver "advertises" free buffer space by including **rwnd** value in TCP header of receiver-to-sender segments
  + **RcvBuffer** size set via socket options (typical default is 4096 bytes)
  + many operating systems autoadjust **RcvBuffer**

+ sender limits amount of unacked ("in-flight") data to receiver's **rwnd** value
+ guarantees receive buffer will not overflow


## TCP: Connection Management
before exchanging data, sender/receiver "handshake":
+ agree to establish connection (each knowing the other willing to establish connection)
+ agree on connection parameters

## TCP: Closing a connection
+ client, server each close their side of connection
  + send TCP segment with FIN bit = 1
+ respond to received FIN with ACK
  + on receiving FIN, ACK can be combined with own FIN
+ simultaneous FIN exchanges can be handled

## Principles of Congestion Control
Congestion:
+ informally: "too many sources sending too much data too fast for **network** to handle"
+ different from flow control
+ manifestations:
  + lost packets(buffer overflow at routers)
  + long delays(queueing in router buffers)
+ a top-10 problem

## Causes/Costs of congestion: scenario 2
+ one router, **finite** buffers
+ sender retransmission of timed-out packet
  + application-layer input = application-layer output
  + transport-layer input includes retransmissions

idealization: perfect knowledge
+ sender sends only when router buffers available

**Idealization:** known loss
packets can be lost, dropped at router due to full buffers
+ sender only resends if packet known to be lost

**Realistic:** duplicates
+ packets can be lost, dropped at router dut to full buffers
+ sender times out prematurely, sending **two** copies, both of which are delivered

**"COSTS" of congestion:**
+ more work(retrans) for given "goodput"
+ unneeded retransmissions: linke carries multiple copies of pkt
  + decreasing goodput

**ANOTHER "COST" of congestion:**
+ when packet dropped, any "upstream transmiision capacity used for that packet was wasted!"


## Approaches towards congestion control
Two broad approaches towards congestion control:

**end-end congestion control:**
+ no explicit feedback from network
+ congestion inferred from end-system observed loss, delay
+ approach taken by TCP

**network-assisted congestion control:**
+ routers provide feedback to end systems
  + single bit indicating congestion(SNA, DECbit, TCP/IP ECN, ATM)
  + explicit rate for sender to send at


## Case Study: ATM ABR congestion control

**ABR**: available bit rate
+ "elastic service"
+ if sender's path "underloaded":
  + sender should use available bandwidth
+ if sender's path congested:
  + sender throttled to minimum guaranteed rate

**RM**: resource management cells
+ sent by sender, interpersed with data cells
+ bits in RM cell set by switches
  + NI bit: no increase in rate (mild congestion)
  + CI bit: congestion indication
+ RM cells returned to sender by receiver, with bits intact

**Hmmmmmmm**
+ two-byte ER (explicit rate) field in RM cell
  + congested switch may lower ER value in cell
  + sender's send rate thus max supportable rate on path
+ EFCI bit in data cells: set to I in congested switch
  + if data cell preceding RM cell has EFCI set, receiver sets CI bit in returned RM cell


## TCP congestion control: additive increase multiplicative decrease
+ **approach**: sender increases transmission rate (window size), probing for usable bandwidth, until loss occurs
  + **additive increase**: increases **cwnd** by 1 MSS every RTT until loss detected
  + **multiplicative decrease**: cut **cwnd** in half after loss

## TCP congestion control: details
+ sender limits transmission: LastByteSent-LastByteAcked <= cwnd
+ **cwnd** is dynamic, function of perceived network congestion

TCP sending rate:
+ roughly: send cwnd bytes, wait RTT for ACKS, then send more bytes
rate = (cwnd)/(RTT) bytes/sec

## TCP Slow Start
+ when connection begins, increase rate exponentially until first loss event:
  + initially **cwnd** = 1 MSS
  + double **cwnd** every RTT
  + done by incrementing **cwnd** for every ACK received

+ **Summary**: intial rate is slow but ramps up exponentially fast

## TCP: Detecting, reacting to loss
+ loss indicated by timeout:
  + **cwnd** set to 1 MSS;
  + window then grows exponentially (as in slow start) to threshold, then grows linearly
+ loss indicated by 3 duplicate ACKs: TCP RENO
  + duplicate ACKs indicate network capable of delivering some segments
  + **cwnd** is cut in half window then grows linearly
+ TCP Tahoe always sets **cwnd** to 1 (timeout or 3 duplicate ACKs)


## TCP: Switching from slow start to CA
**Question**: when should the exponential increase switch to linear?
**Answer**: when **cwnd** gets to 1/2 of its value before timeout.


**Implementation**: 
+ variable **ssthresh**
+ on loss event, **ssthresh** is set to 1/2 of **cwnd** just before loss event

## TCP: throughput
+ average TCP thruput as function of window size, RTT?
  + ignore slow start, assume always data to send
+ W: window size (measured in bytes) where loss occurs
  + average window size (# in-fligth bytes) is 3/4 W
  + average thruput is 3/4W per RTT

## TCP: Fairness
**fairness goal**: if K TCP sessions share same bottleneck link of bandwidth R, each should have average rate of R/K

**Question**:Why is TCP fair?
Two competing sessions:
+ additive increase gives slope of 1, as throughput increases
+ multiplicative decrease decreases throughput proportionally

**Fairness and UDP**
+ multimedia apps often do not use TCP
  + do not want rate throttled by congestion control
+ instead use UDP:
  + send audio/video at constant rate, tolerate packet loss

**Fairness, parallel TCP connections**
+ application can open multiple parallel connections between two hosts
+ web browsers do this
+ e.g., link of rate R with 9 existing connections:
  + new app asks for 1 TCP, gets rate R/10
  + new app asks for 11 TCPs, gets R/2
