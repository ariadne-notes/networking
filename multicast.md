## Theory

Multicast is always TO a group not FROM a group. 

A multicast address can only ever be a destination.

Addresses      | Description
-------------- | --------------
`224.0.0.0/4`  | Multicast Supernet
`224.0.0.1`    | All Systems on this Subnet
`224.0.0.2`    | All Routers on this Subnet
`232.0.0.0/8`  | Source-Specific Multicast (SSM)
`239.0.0.0/8`  | Organization-Local Scope

## Common Addresses

Protocol           | Multicast Address
--------------     | --------------
RIP                | 255.255.255.0
OSPF-hello         | 224.0.0.5
OSPF-DR            | 224.0.0.6
RIPv2              | 224.0.0.9
EIGRP              | 224.0.0.10
mDNS               | 224.0.0.251
cisco-rp-announce  | 224.0.1.39
cisco-rp-discovery | 224.0.1.40

[IANA Assignments](https://www.iana.org/assignments/multicast-addresses/multicast-addresses.xhtml)

PIM forms adjacencies in only one direction

The multicast source is the root of the tree. Packets flow downstream from the source. Control plane traffic like PIM joins flow upstream to the RP, or to the reciever.

# PIM
PIM has five modes:
* PIM Dense (PIM-DM) - Any Source
* PIM Sparse (PIM-SM) - Any Source
* PIM Sparse Dense
* PIM Source Specific (PIM-SSM)
* PIM Bidirectional (Bidir-PIM)

## PIM Dense Mode
Based on RFC 3973 Protocol Independent Multicast Dense Mode (PIM-DM)
- Push or Implicit Join
  - Flood and Prune
  - Doesn't care about Designated Routers
  - Routers with no Recievers prune
  - `224.0.0.13` to find neighbors
  - Send traffic out all interfaces running dense
  - Recievers prune back
  - Router attached to LAN listens for multicast control plane.
     - Recieves source traffic
       - Insert (*,G) and (S,G) into mrib
       - Incomming traffic is attached to IIL
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
