# Terms

* **Multicast:** A one-to-many service using UDP packets destined to group IP address. Hosts subscribe to the group, routers replicate for the group.
* **IGMP:** Internet Group Management Protocol. A host uses IGMP to request a multicast stream. Switches see it (for snooping), and the FHR uses this to build the MDT.
* **PIM:** Protocol Independent Multicast. Multicast capable routers communicate to each over via PIM.
* **IIL:** Incoming Interface List, part of the MDT.
* **OIL:** Outgoing Interface List, part of the MDT.
* **MDT:** Multicast Distribution Tree. The full set of links participating in multicast, via PIM, IGMP, including IILs, and OILs.
* **RP:** Rendezvous Point. A router designated as the root of a shared tree.
* **(*,G):** Star comma Gee. AKA, a shared tree. These require a RP. Called Star comma Gee, because typing "show ip mroute" ... this is what shows up.
* **(S,G):** Ess comma Gee. AKA a source tree. These do not require a RP.
* **Source Tree:** AKA, SPT, or shortest path tree. SPT is best tree.
* **RPT:** Rendezvous Point Tree, this is a *,G that points towards the RP.
* **ASM:** Any Source Multicast. The host only knows the group it wants to receive (239.10.10.10).
* **SSM:** Source Specific multicast. The host already knows the source, and group address (10.0.0.1, 232.10.10.10).
* **Upstream:** Towards the source.
* **Downstream:** Towards group members.
* **FHR:** First hop router. This router receives a multicast stream.
* **LHR:** Last Hop router receives IGMP messages from receivers, which are translated into PIM join messages.
* **MRIB:** The multicast routing table. Shows RPTs, SPTs, RPFIs, OILs, and IILs.
* **MFIB:** The forwarding table. This is used for programming the hardware.
* **RIB:** Routing Information Base
* **DF:** Designated Forwarder. Used in PIDIR-PIM.

# Harder Terms

### RPF - Reverse Path Forwarding

PIM is protocol independent, in the sense, that if a stream turns on, it must have a source, so it takes the form (10.0.0.1, 239.1.1.1), a (S,G).

If we do `show ip route 10.0.0.1`, we'll see the interface the router intends to send any traffic towards that source address. This is the "upstream" interface.

As multicast traffic flows from 10.0.0.1, it should flow into the upstream interface, and out of any downstream interfaces (the OIL).

Tracing the traffic back to the source this way is called "reverse path forwarding" and the interface along this path is the RPF.

The PIM neighbor on the RPF is called the RPF neighbor.

Any multi-cast traffic from any given source, not received on the RPF is discarded. This prevents loops.


## Shared Trees
(*,G) entries in the mroute table require fewer resources, since multiple sources can use the same tree.

(*,G) entries in the mroute table represent a security risk, because any source can send to this shared tree.

## Theory (in v4)

Multicast is always TO a group, a destination, or a set of destinations.

Multicast comes from an older time. Unlike Unicast addresses, you can tell via bits if a v4 address is multicast.

A multicast address always start with `1110`

Address Scopes      | Description
------------------- | --------------
`224.0.0.0/4`       | Multicast Supernet
`224.0.0.0/24`      | Local Control (TTL=1)
`224.0.1.0/24`      | Internetwork Control (an example is NTP, Cisco RP-Announce, Cisco RP-Discovery)
`232.0.0.0/8`       | Source-Specific Multicast (SSM). Via an extension PIM can build (S,G) MDTs.
`233.0.0.0/8`       | GLOP! Companies with a 16-bit ASN can have globally static multicast. 233.X.Y.0/8
`239.0.0.0/8`       | Organization-Local Scope. Exactly like RFC1918, but for multicast.


## Common L3 Addresses

**Same Broadcast Domain**

Protocol           | Multicast Address
--------------     | --------------
all-hosts          | 224.0.0.1
all-routers        | 224.0.0.2
OSPF-hello         | 224.0.0.5
OSPF-DR            | 224.0.0.6
RIPv2              | 224.0.0.9
EIGRP              | 224.0.0.10
PIM                | 224.0.0.13
mDNS               | 224.0.0.251

**Can be forwarded**

Protocol           | Multicast Address
--------------     | --------------
ntp                | 224.0.1.1
cisco-rp-announce  | 224.0.1.39
cisco-rp-discovery | 224.0.1.40


Protocol           | Multicast Address | Notes
-------------------|-------------------|------------------------------------------
ntp                | 224.0.1.1         |
cisco-rp-announce  | 224.0.1.39        | Candidate RPs announce every 60s. Highest IP wins.
cisco-rp-discovery | 224.0.1.40        | Mapping agent floods RP-to-group mappings.

[IANA Assignments](https://www.iana.org/assignments/multicast-addresses/multicast-addresses.xhtml)

PIM forms adjacencies in only one direction

The multicast source is the root of the tree. Packets flow downstream from the source. Control plane traffic like PIM joins flow upstream to the RP, or to the reciever.


Protocol           | Multicast Address
--------------     | --------------
all-hosts          | 224.0.0.1
all-routers        | 224.0.0.2
OSPF-hello         | 224.0.0.5
OSPF-DR            | 224.0.0.6
RIPv2              | 224.0.0.9
EIGRP              | 224.0.0.10
PIM                | 224.0.0.13
mDNS               | 224.0.0.251


# PIM

PIM Mode             | Full Name             | How it works
---------------------|-----------------------|------------------------------------------------------
PIM-DM               | Dense Mode            | No RP. Floods everywhere, routers send prune messages to un-join. Assumes everyone wants the traffic.
PIM-SM               | Sparse Mode           | Complex. Requires a RP, RP Discovery, and phases. Uses register messages, and both tree types.
PIM Sparse-Dense     | Sparse-Dense Mode     | Runs sparse for groups with a known RP, dense for groups without. Legacy transitional mode.
Bidir-PIM            | Bidirectional         | Shared tree only, traffic flows both toward and away from RP. No SPT switchover. Good for many-to-many applications.
PIM-SSM              | Source Specific       | No RP. Receiver specifies both source and group (S,G).


## PIM Message Types
 Type | Message Type               | Destination                       | Purpose
------|----------------------------|-----------------------------------|------------------------------------------------------
 0    | Hello                      | 224.0.0.13 (all PIM routers)      | Establish adjacency, negotiate parameters.
 1    | Register                   | RP address (unicast)              | First-hop router notifies RP of new source, encapsulates multicast data until SPT is built.
 2    | Register stop              | First-hop router (unicast)        | RP tells first-hop router to stop sending Register messages.
 3    | Join/prune                 | 224.0.0.13 (all PIM routers)      | Join or prune a multicast tree, either (*,G) toward RP or (S,G) toward source.
 4    | Bootstrap                  | 224.0.0.13 (all PIM routers)      | BSR floods RP-set information throughout the domain so all routers know candidate RPs.
 5    | Assert                     | 224.0.0.13 (all PIM routers)      | Elect a single forwarder on a multi-access segment when duplicate traffic is detected.
 8    | Candidate RP advertisement | Bootstrap router (BSR) (unicast)  | Candidate RPs advertise themselves to the BSR.
 9    | State refresh              | 224.0.0.13 (all PIM routers)      | PIM-DM only. Prevents prune state from timing out and triggering a re-flood.
 10   | DF election                | 224.0.0.13 (all PIM routers)      | Bidir-PIM only. Elects a Designated Forwarder per link to forward traffic toward the RP.
 
 
## Auto RP

Cisco devices can announce their willingness to be an RP, via `cisco-rp-announce`

A different service, a mapping agent, will read these messages, pick a winner, then advertise that out via `cisco-rp-discovery`

* 5.5.5.5, Candidate RP.
* 4.4.4.4, mapping agent.

```
R4# show ip pim autorp 
AutoRP Information: 
  AutoRP is enabled.
  RP Discovery packet MTU is 1500.
  224.0.1.40 is joined on Loopback0.
  AutoRP groups over sparse mode interface is enabled

PIM AutoRP Statistics: Sent/Received
  RP Announce: 0/16, RP Discovery: 64/42
```

These packets are slow.

```
R4#debug ip pim auto-rp 
PIM Auto-RP debugging is on
R4#
!
! Sent to cisco-rp-discovery
!
*Apr 25 19:57:08.940: Auto-RP(0): Build RP-Discovery packet
*Apr 25 19:57:08.941: Auto-RP(0):  Build mapping (224.0.0.0/4, RP:5.5.5.5), PIMv2 v1,
*Apr 25 19:57:08.942: Auto-RP(0): Send RP-discovery packet of length 48 on GigabitEthernet0/3 (1 RP entries)
*Apr 25 19:57:08.943: Auto-RP(0): Send RP-discovery packet of length 48 on GigabitEthernet0/4 (1 RP entries)
*Apr 25 19:57:08.945: Auto-RP(0): Send RP-discovery packet of length 48 on GigabitEthernet0/0 (1 RP entries)
*Apr 25 19:57:08.948: Auto-RP(0): Send RP-discovery packet of length 48 on Loopback0(*) (1 RP entries)
*Apr 25 19:57:12.008: Auto-RP(0): Received RP-discovery packet of length 48, from 10.0.45.5, ignored
!
! Received by cisco-rp-announce
!
*Apr 25 19:58:30.159: Auto-RP(0): Received RP-announce packet of length 48, from 5.5.5.5, RP_cnt 1, ht 181
*Apr 25 19:58:30.159: (0): pim_add_prm:: 224.0.0.0/240.0.0.0, rp=5.5.5.5, repl = 0, ver =3, is_neg =0, bidir = 0, crp = 0
*Apr 25 19:58:30.160: Auto-RP(0): Update
*Apr 25 19:58:30.160:  prm_rp->bidir_mode = 0 vs bidir = 0 (224.0.0.0/4, RP:5.5.5.5), PIMv2 v1
R4# undebug all
All possible debugging has been turned off
```

## Dense
Based on RFC 3973 Protocol Independent Multicast Dense Mode (PIM-DM)
- Push Model
  - Good for when every subnet probably wants this traffic
- No PIM DR
  - All FHR forward multicast traffic
    - Multicast traffic is flooded out every interface that isn't the RPF.
- Eventually builds a SPT after prunes
- IGMP joins turn into graft messages
- Prunes last 3 minutes
  - Flood and Prune
  - Routers with no Receivers or duplicate S,G traffic prune.
  - `224.0.0.13` to find neighbors
  - Receivers prune back
  - Router attached to LAN listens for multicast control plane.
     - Receives source traffic
       - Insert (*,G) and (S,G) into mrib
       - Incoming traffic is attached to IIL
       - OIL is all other interfaces
       - Flood to OIL
       - PIM dense always uses SPT.
- Prune occurs
  - Traffic flows stop, but (S,G) remains in table
  - Multicast fails RPF
  - No downstream neighbor or reciever
  - Downstream sent prune
  - LAN Prune override exception
- After pruning 
  - Flood again, prune back, flood again, prune back
  
## PIM Sparse
Based on [RFC4601](https://www.rfc-editor.org/rfc/rfc4601) - Protocol Independent Multicast Sparse Mode (PIM-SM)
- Explicit joins everywhere. No flooding.
- LHR, sends a PIM-Join towards the RP, building a (*,G).
- Phased
  - 1. [The RPT tree](https://www.rfc-editor.org/rfc/rfc4601#section-3.1)
    - Receivers sending their (*,G) messages towards the RP.
    - FHR encapsulates the multicast traffic directly towards the RP.
    - PIM-Register
    - RP de-encapsulates the traffic, sending it down the RPT.
  - 2. [Register Stop](https://www.rfc-editor.org/rfc/rfc4601#section-3.2)
    - The RP sends a (S,G) towards the source.
    - When multicast packets start showing up, without encapsulation, the RP sends a Register-Stop.
  - 3. [SPT tree](https://www.rfc-editor.org/rfc/rfc4601#section-3.3)
    - LHR requests a (S,G) entry towards it's upstream, until it's joined to the (S,G) tree.
    - When the LHR starts getting two copies of the traffic, it sends a (S,G,rpt) prune message. (A prune specific to the RPT)
- If two LHRs exist, and duplicate traffic is detected a PIM elections happens.
  - These Asserts are every 3 minutes.
  - RPTbit, 0 is preferred and means "has (S,G) tree"
    - Metric Preference (Administrative Distance)
      - Metric
        - IP address of subnet interface.
- Specify the tunnel, for the pim-register messages on Cisco via `ip pim register-source loopback 0`
- The tunnel interface encapsulates the entire multicast packet, which adds 28 bytes of overhead. Packets close to the MTU will be silently dropped on IOS-XE.

[PIM-SM-register-register-stop-prune.pcap](https://github.com/user/repo/raw/main/captures/multicast/PIM-SM-register-register-stop-prune.pcap)
  
  

a DR is elected by highest priority, or highest IP in the subnet.
  - DR sends the PIM join upstream.

The RP always gets the stream, even if it has no receivers to forward it to.

## BIDIR-PIM 
Based on RFC 4601 - Bidirectional Protocol Independent Multicast (BIDIR-PIM)
- Superset of PIM-SM
- No (S,G) entries
- Traffic can flow up and down the same tree.
- Still needs RPs
  - RP must be dedicated to BIDIR-PIM.
- Each bidirectional link has a DF election.
  - Ingress packets on any PIM interface can be forwarded downstream onto DF links.
    - No DF links, no forwarding.
  - Ingress packets to a DF can be forwarded upstream via the RPF towards the RPA.
  
## MSDP
- RPs register to each other, in different multicast domains.
- RP sends a SA (source active) message.
- Still needs PIM running for the S,G.
- TCP port 639.
- Has keepalives.

`show ip msdp peer`
`show ip msdp sa-cache`


#### Shared-Tree (*,G)
- Shared trees are essential for multiple senders to the same group
- A single tree is built for each group, regardless of source
  - 3 sources, 1 tree
- Selects a router as the root of the tree
- If a receiver is on the same subnet as the sending host, it will need to revert to PIM Dense for that segment

- This isn't always better. Shared trees will typically take suboptimal paths through a network
- Source trees are better distributed, hence they are more robust
- RP Selection is a hassle

#### Source Based Multicast (S,G)
- PIM dense uses a separate tree for each multicast source and destination group.
- Groups do not share trees.
  - 3 Sources 3 trees.

## Commands

`show pim rpf hash`

`show pim range-list`

`show pim topology`

`show mrib route`

`show ip mroute`

What interface should I receive this host traffic from?

`show ip rpf 10.0.0.`

`show ip mfib`

See if multicast even works

`show ip pim stats`

See if PIM adjacency traffic even arrives.

`show ip pim interface detail`

See results of DF election

`show ip pim interface df`



```
FLAGS
 A - Accepting. This interface is accepting data
 F - Forwarding. Where to send multicast traffic
```

#### Nexus 7K
`show forwarding multicast route group <>`

## L2 Addresses
MAC addresses are 48 bits.

The first 25 bits are always.

```
0000 0001 . 0000 0000 . 0101 1110 . 0??? ????
       01 :        00 :        5E :
        ^                           ^
        |                           └─  Multicast requires this bit be 0.
        |
        └─ Individual/Group. Multicast requires this bit be 1.
```

So the first six bytes are `01:00:5E`

The last 23 bits come from the IP address.

**A Multicast IP**

Mapping 232.10.10.10 → 01:00:5E:0A:0A:0A

Copy the low order 23 bits directly from the v4 address.

```

  232.10.10.10/8
  (in binary)
  1110 1000 . 0000 1010 . 0000 1010 . 0000 1010
               \______________________________/
               Remember these 23 bits.
   
```

### Building the L2 Address.

Ethernet Multicast MAC Address

```
          1 :         0 :        5E :        0A :       0A  :        0A
  0000 0001 . 0000 0000 . 0101 1110 . 0000 1010 . 0000 1010 . 0000 1010
  \__________________________________/|\______________________________/
        Assigned first 25 bits        |   Same bits as above.
        (always 01:00:5E)             |  (24 bits → 23 bits, 1 bit dropped)
                                      |
                                      |
                                      └─  Multicast requires this bit be 0
```        

### Quirks and Tech Debt.

Because we copied only 23 bits, vs 28 bits, we have 5 bits of overlap.

v4 is 32 bits, minus those four bits that can never change `1110` to get 28 bits.

All these IPs share the same multicast L2 address.

```
All 32 IPv4 addresses mapping to 01:00:5E:0A:0A:0A
══════════════════════════════════════════════════════════════════════════════
Address           Octet 1    Octet 2    Octet 3    Octet 4
──────────────────────────────────────────────────────────────────────────────
224. 10.10.10     1110 0000  0000 1010  0000 1010  0000 1010
224.138.10.10     1110 0000  1000 1010  0000 1010  0000 1010
225. 10.10.10     1110 0001  0000 1010  0000 1010  0000 1010
225.138.10.10     1110 0001  1000 1010  0000 1010  0000 1010
226 .10.10.10     1110 0010  0000 1010  0000 1010  0000 1010
226.138.10.10     1110 0010  1000 1010  0000 1010  0000 1010
227 .10.10.10     1110 0011  0000 1010  0000 1010  0000 1010
227.138.10.10     1110 0011  1000 1010  0000 1010  0000 1010
228 .10.10.10     1110 0100  0000 1010  0000 1010  0000 1010
228.138.10.10     1110 0100  1000 1010  0000 1010  0000 1010
229 .10.10.10     1110 0101  0000 1010  0000 1010  0000 1010
229.138.10.10     1110 0101  1000 1010  0000 1010  0000 1010
230 .10.10.10     1110 0110  0000 1010  0000 1010  0000 1010
230.138.10.10     1110 0110  1000 1010  0000 1010  0000 1010
231 .10.10.10     1110 0111  0000 1010  0000 1010  0000 1010
231.138.10.10     1110 0111  1000 1010  0000 1010  0000 1010
232 .10.10.10     1110 1000  0000 1010  0000 1010  0000 1010  < --- This is our SSM address.
232.138.10.10     1110 1000  1000 1010  0000 1010  0000 1010
233 .10.10.10     1110 1001  0000 1010  0000 1010  0000 1010  < --- An address in the GLOP block.
233.138.10.10     1110 1001  1000 1010  0000 1010  0000 1010
234 .10.10.10     1110 1010  0000 1010  0000 1010  0000 1010
234.138.10.10     1110 1010  1000 1010  0000 1010  0000 1010
235 .10.10.10     1110 1011  0000 1010  0000 1010  0000 1010
235.138.10.10     1110 1011  1000 1010  0000 1010  0000 1010
236 .10.10.10     1110 1100  0000 1010  0000 1010  0000 1010
236.138.10.10     1110 1100  1000 1010  0000 1010  0000 1010
237 .10.10.10     1110 1101  0000 1010  0000 1010  0000 1010
237.138.10.10     1110 1101  1000 1010  0000 1010  0000 1010
238 .10.10.10     1110 1110  0000 1010  0000 1010  0000 1010
238.138.10.10     1110 1110  1000 1010  0000 1010  0000 1010
239 .10.10.10     1110 1111  0000 1010  0000 1010  0000 1010
239.138.10.10     1110 1111  1000 1010  0000 1010  0000 1010  < --- an Organizational scope address.
══════════════════════════════════════════════════════════════════════════════
                       ^^^^  ^
                       ||||  | 
                       └└└└──└─ I incremented these five bits to show the pattern.
```


# Lab Stuff.

BPF - Capture all PIM, but not PIM hello messages.

`ip proto 103 and not ether[34] == 0x20`

### Sending Multicast

```
iperf --client 239.10.10.10 --udp --time 3600 --interval 1 --bandwidth 1pps --ttl 15 --len 1000
```

### Receiving Multicast
```
iperf --server --udp --bind 239.10.10.10 --interval 1
```