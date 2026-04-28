# Modern

OSPF is protocol 89.

* **LSA:** Link State Advertisement
* **LSDB:** Link-state Database
* **OSPF Process ID:** Just where the databases live. Not transmitted. Allows multiple OSPF processes.

| Type | Name | Purpose |
|------|------|---------|
| 1 | Hello                             | OSPF puts the neighbor ID into it's hello messages. |
| 2 | Database Description (DBD/DDP)    | Used to sync a new neighbor rapidly. Large update packet, to transfer the LSDB in bulk. Contains lots of LSAs. |
| 3 | Link-State Request (LSR)          | The router wants a specific LSA.   |
| 4 | Link-State Update (LSU)           | The neighbor sends a specific LSA. |
| 5 | Link-State Acknowledgment (LSAck) | To confirm a device got the intended LSAs, it transmits the exact same LSAs back to the receiver. |

These can be thought of as the five steps.

1. We say hello, using each others names, to confirm we can both hear one another.
2. We share state (like the weather).
3. I ask how something went.
4. You tell me how it went.
5. To make sure I really got it, I'll repeat it word-for-word.

# Hello Packets
These things must match for an adjacency to form
- Subnet
- Subnet mask
- Interface MTU
- Area
- Area flags (NSSA, Stub)
- Is DR/BDR enabled
- Authentication
- Hello time
- Dead time

These must not match
- Router ID


# OSPF State Machine

| State | Description |
|-------|-------------|
| Down        | OSPF is running, no hello packets received yet. |
| Attempt     | NBMA mode, the router has sent OSPF packets. |
| Init        | The router sees hello packets. |
| 2-Way       | The router sees it's own router-id in the hello packet. |
| ExStart     | Routers vote on who exchanges LSDB first. |
| Loading     | Router DB has been exchanged, router is requesting specific LSAs. |
| Full        | LSDBs for this area are identical on both sides. |

# DR and BDR

OSPF uses explicit acknowledgments (re-sending the LSAs), so as neighbors and adjacencies grow, the amount of OSPF traffic on a network increases.

A network with six ospf routers forming a full-mesh requires 30 adjacencies.

To mitigate the scaling problem, on broadcast segments OSPF elects a DR, and BDR, to maintain the LSDB.

## Network LSAs

These are sent by the DR to describe the routers on this segment.

```
R6# show ip ospf database network 

            OSPF Router with ID (6.6.6.6) (Process ID 1)

                Net Link States (Area 0)

  LS age: 184
  Options: (No TOS-capability, DC)
  LS Type: Network Links
  Link State ID: 10.0.0.1 (address of Designated Router)
  Advertising Router: 1.1.1.1
  LS Seq Number: 80000011
  Checksum: 0x2690
  Length: 48
  Network Mask: /24
        Attached Router: 1.1.1.1
        Attached Router: 2.2.2.2
        Attached Router: 3.3.3.3
        Attached Router: 4.4.4.4
        Attached Router: 5.5.5.5
        Attached Router: 6.6.6.6
```

### The DR

Forms full adjacencies.

```
R1# show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2          50   FULL/BDR        00:00:31    10.0.0.2        Ethernet0/0
3.3.3.3           1   FULL/DROTHER    00:00:37    10.0.0.3        Ethernet0/0
4.4.4.4           1   FULL/DROTHER    00:00:34    10.0.0.4        Ethernet0/0
5.5.5.5           1   FULL/DROTHER    00:00:32    10.0.0.5        Ethernet0/0
6.6.6.6           1   FULL/DROTHER    00:00:31    10.0.0.6        Ethernet0/0
```
 
### Drother

Only forms full adjacencies with the DR, and BDR. 
 
``` 
R1# show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2          50   FULL/BDR        00:00:31    10.0.0.2        Ethernet0/0
3.3.3.3           1   FULL/DROTHER    00:00:37    10.0.0.3        Ethernet0/0
4.4.4.4           1   FULL/DROTHER    00:00:34    10.0.0.4        Ethernet0/0
5.5.5.5           1   FULL/DROTHER    00:00:32    10.0.0.5        Ethernet0/0
6.6.6.6           1   FULL/DROTHER    00:00:31    10.0.0.6        Ethernet0/0
```

# Identical Databases

Each router can perform it's own SPT via [Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm).

LSAs are flooded throughout an area, all routers in the same area should have the same LSAs and same database.
```
R1# show ip ospf database database-summary  | s Area 0
Area 0 database summary
  LSA Type      Count    Delete   Maxage
  Router        5        0        0       
  Network       5        0        0       
  Summary Net   8        0        0       
  Summary ASBR  2        0        0       
  Type-7 Ext    0        0        0       
    Prefixes redistributed in Type-7  0
  Opaque Link   0        0        0       
  Opaque Area   0        0        0       
  Subtotal      20       0        0
```

```  
R2# show ip ospf database database-summary | s Area 0
Area 0 database summary
  LSA Type      Count    Delete   Maxage
  Router        5        0        0       
  Network       5        0        0       
  Summary Net   8        0        0       
  Summary ASBR  2        0        0       
  Type-7 Ext    0        0        0       
    Prefixes redistributed in Type-7  0
  Opaque Link   0        0        0       
  Opaque Area   0        0        0       
  Subtotal      20       0        0
```

# Default Route

OSPF has two ways of originating a default route.

`default-information originate` if a default route is present.

`default-information originate always` do it anyway.

# Cost

Default OSPF is all links above 100Mbps are the same cost.

```
auto-cost reference-bandwidth 40,000
```

## Network Types
 
[OSPF Representation of routers and networks](https://www.rfc-editor.org/rfc/rfc2328#page-13)

| CLI                                   |      Network Types      | Multicast or Unicast	| LSA Type 1 or 2	|   Use-case                                                                            |
| ------------------------------------- | ----------------------- | ----------------------- | ----------------- | ------------------------------------------------------------------------------------- |
| `ip ospf network broadcast`           | Broadcast		          | Multicast		        | 2 - DR Election	|   Ethernet, Token Ring, FDDI                                                          |
| `ip ospf network non-broadcast`       | NBMA[^NBMA]			  | **Unicast**[^unicast]   | 2 - DR Election 	|   X.25, frame-relay, ATM (poorly, the hub needs to be DR)                             |
| `ip ospf network point-to-point`      | point-to-point		  | Multicast		        | 1 - No DR		    |   T1, T3, Serial, HDLC, PPP (Full Adjacency)                                          |
| `ip ospf network point-to-multipoint` | point-to-multipoint	  | Multicast		        | 1 - No DR		    |   Frame-relay with a Hub router. Uses more resources then NBMA, more fault tolerant  |

[^NBMA]: RFC compliant (??) implementation. I recommend using `ip ospf network point-to-multipoint`.
[^unicast]: The DR (which should be the HUB or bad things happen) needs to have static neighbor statements.

```
Moy                         Standards Track                    [Page 13]

RFC 2328                     OSPF Version 2                   April 1998

                                                  **FROM**

                                           *      |RT1|RT2|
                +---+Ia    +---+           *   ------------
                |RT1|------|RT2|           T   RT1|   | X |
                +---+    Ib+---+           O   RT2| X |   |
                                           *    Ia|   | X |
                                           *    Ib| X |   |

                     Physical point-to-point networks


                                                  **FROM**
                      +---+                *
                      |RT7|                *      |RT7| N3|
                      +---+                T   ------------
                        |                  O   RT7|   |   |
            +----------------------+       *    N3| X |   |
                       N3                  *

                              Stub networks

                                                  **FROM**
                +---+      +---+
                |RT3|      |RT4|              |RT3|RT4|RT5|RT6|N2 |
                +---+      +---+        *  ------------------------
                  |    N2    |          *  RT3|   |   |   |   | X |
            +----------------------+    T  RT4|   |   |   |   | X |
                  |          |          O  RT5|   |   |   |   | X |
                +---+      +---+        *  RT6|   |   |   |   | X |
                |RT5|      |RT6|        *   N2| X | X | X | X |   |
                +---+      +---+

                          Broadcast or NBMA networks
```


### Sham Link

**The Problem**

A customer with L3VPN service via OSPF-BGP-VPNv4 decides to connect two sites together via OSPF backdoor, a direct connection they manage themselves.

When they turn on their private OSPF peering, all the traffic between these two sites now prefers the new link, vs the L3VPN cloud.

**The Solution: Sham Links**

Sham links are needed because the routes provided by an L3VPN are `O IA`. When the OSPF backdoor link comes up it will be preferred for two reasons:
* OSPF has a lower AD than BGP.
* `O` routes are prefered over `O IA`

A sham link makes two PE routers at different sites in the same customer VRF form an intra-area connection. 

From [OSPF Sham-Link Support for MPLS VPN - Cisco](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_ospf/configuration/15-sy/iro-15-sy-book/iro-sham-link.html#GUID-B0CBC9E8-D423-4AEF-BAB4-15FED3EA486C).

> Before you create a sham-link between PE routers in an MPLS VPN, you must:
>
> * Configure a new interface with a /32 address on the remote PE so that OSPF packets can be sent over the VPN backbone to the remote end of the sham-link. The /32 address must meet the following criteria:
>   * Belong to a VRF
>   * Not be advertised by OSPF
>   * Be advertised by BGP
>   * You can use the /32 address for other sham-links

# References

https://datatracker.ietf.org/doc/html/rfc2328