# Terms

* **Multicast:** A one-to-many service using UDP packets destined to group IP address. Hosts subscribe to the group, routers replicate for the group.
* **IGMP:** Internet Group Management Protocol. A host uses IGMP to request a multicast stream. Switches see it (for snooping), and the FHR uses this to build the MDT.
* **PIM:** Protocol Independent Multicast. Multicast capable routers communicate to each over via PIM.
* **FHR:** First hop router. This router gets the IGMP messages.
* **IIL:** Incoming Interface List, part of the MDT.
* **OIL:** Outgoing Interface List, part of the MDT.
* **MDT:** Multicast Distribution Tree. The full set of links participating in multicast, via PIM, IGMP, including IILs, and OILs.
* **RP:** Rendezvous Point. A router designated as the root of a shared tree.
* **(*,G):** Star comma Gee. AKA, a shared tree. These require a RP. Called Star comma Gee, because typing "show ip mroute" ... this is what shows up.
* **(S,G):** Ess comma Gee. AKA a source tree. These do not require a RP.
* **Source Tree:** AKA, SPT, or shortest path tree. SPT is best tree.
* **ASM:** Any Source Multicast. The host only knows the group it wants to receive (239.10.10.10).
* **SSM:** Source Specific multicast. The host already knows the source, and group address (10.0.0.1, 232.10.10.10).


## Theory (in v4)

Multicast is always TO a group not FROM a group. 

A multicast group is a destination, or a set of destinations.

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
PIM-DM               | Dense Mode            | No RP. Floods everywhere, routers send prune messages to un-join.
PIM-SM               | Sparse Mode           | Needs a RP.
PIM Sparse-Dense     | Sparse-Dense Mode     | Runs sparse for groups with a known RP, dense for groups without. Legacy transitional mode.
Bidir-PIM            | Bidirectional         | Shared tree only, traffic flows both toward and away from RP. No SPT switchover. Good for many-to-many applications.
PIM-SSM              | Source Specific       | No RP. Receiver specifies both source and group (S,G).

## Dense
Based on RFC 3973 Protocol Independent Multicast Dense Mode (PIM-DM)
- Useful when you know every subnet needs this multicast group
- Push or Implicit Join
  - Flood and Prune
  - Doesn't care about Designated Routers
  - Routers with no Receivers prune
  - `224.0.0.13` to find neighbors
  - Send traffic out all interfaces running dense
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
- Discover PIM neighbors
- Discover the RP
- Tell RP about the source
- Tell RP about recievers 
- Build shortest path tree from sender to recievers through RP
- Join shortest path tree
- Leave shared tree
- Perform mrib maintainance

#### Shared-Tree (*,G)
Based on RFC4601 - Protocol Independent Multicast Sparse Mode (PIM-SM)
- PIM Sparse
  - A single tree is built for each group, regardless of source
    - 3 sources, 1 tree
  - Selects a router as the root of the tree [Rendezvous Point]
  - Multicast sources not on the shared tree encapsulate their data to the RP, which de-encapsulates it, then flows down the tree
  - If a reciever is on the same subnet as the sending host, it will need to revert to PIM Dense for that segment
  - Shared trees are essential for multiple senders to the same group
  - This isn't always better. Shared trees will typically take suboptimal paths through a network
  - Source trees are better distributed, hence they are more robust
  - RP Selection is a hassle

#### Source Based Multicast (S,G)
- PIM dense uses a seperate tree for each multicast source and destination group.
- Groups do not share trees.
  - 3 Sources 3 trees.

## Commands

#### IOS-XR
`show pim rpf hash`

`show pim range-list`

`show pim topology`

`show ip mroute`

`show mrib route summary`

#### IOS
`show ip rpf`

`show ip mfib`
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