I learned this protocol using IOS-XR.

**Async, no echo** - Please respond to this packet with the control plane of the far device.

### BFD Async without Echo
```
          Peer-A to Peer-B, lets agree to use BFD.
          
          Peer-A, I see your control packets.
          
          Peer-B, I also see your control packets.
          
          
          L3 SRC A
          L3 DST B
          
         +------------------------------->
+-------+                                 +-------+
|Peer-A |                                 |Peer-B |
+-------+                                 +-------+
         <-------------------------------+
```

**Async, with echo** - Just loop the BFD packets back onto the link, please.

### BFD Async with Echo
The packets never leave the data plane, and never touches the control plane of Peer-A or Peer-B.
```

           L3 SRC A
           L3 DST A

!
! Peer A tests it's return path
!
+-------+                                   +-------+
|       | +-------------------------------+ |       |
|Peer-A |                                 | |Peer-B |
|       | <-------------------------------+ |       |
+-------+                                   +-------+


           L3 SRC A
           L3 DST A
!
! Peer B also tests it's return path
!
+-------+                                   +-------+
|       | +-------------------------------+ |       |
|Peer-A | |                                 |Peer-B |
|       | +-------------------------------> |       |
+-------+                                   +-------+
```

## Ports

BFD is UDP, to an application on the network device

BFD Control is sent as SRC UDP 49512 --> Destination 3784

BFD Payload is sent as SRC UDP 3785  --> Destination 3785


#### BFD State Machine
Courtesy of the RFC

```
RFC 5880           Bidirectional Forwarding Detection          June 2010

(removed) 

The following diagram provides an overview of the state machine.
Transitions involving AdminDown state are deleted for clarity (but
are fully specified in sections 6.8.6 and 6.8.16).  The notation on
each arc represents the state of the remote system (as received in
the State field in the BFD Control packet) or indicates the
expiration of the Detection Timer.

                             +--+
                             |  | UP, ADMIN DOWN, TIMER
                             |  V
                     DOWN  +------+  INIT
              +------------|      |------------+
              |            | DOWN |            |
              |  +-------->|      |<--------+  |
              |  |         +------+         |  |
              |  |                          |  |
              |  |               ADMIN DOWN,|  |
              |  |ADMIN DOWN,          DOWN,|  |
              |  |TIMER                TIMER|  |
              V  |                          |  V
            +------+                      +------+
       +----|      |                      |      |----+
   DOWN|    | INIT |--------------------->|  UP  |    |INIT, UP
       +--->|      | INIT, UP             |      |<---+
            +------+                      +------+
```

* **Async** - If the other side doesn't recieve the packets, it's declared down.

* **BOB** - BFD over Bundle

* **BLB** - BFD over Logical Bundle - (VLANS, Sub-interfaces). This requires multipath to be enabled. Multipath doesn't inject BFD packets into the HP queue.



### IOS-XR Commands
```
multipath include location 0/1/CPU0
bundle coexistence bob-blb logical
show tech-support routing bfd file
```

## IOS-XR Examples

#####  Take the session down if latency grows to 150ms for a single echo packet.
```
bfd fast detect 
bfd multiplier 50
echo latency detect
```

##### Take the session down if latency grows to 300ms for a single echo packet.
```
bfd fast detect 
bfd multiplier 50
bfd echo latency detect percentage 200
```

##### Take the session down if the latency grows to 150ms for 3 consequitive echo packets
```
bfd fast detect
bfd multiplier 50
bfd echo latency detect percentage 100 count 3
```

#### Disable echo mode
```
bfd 
interface g0/0/0/0
 echo disable
```

#### Protecting the BFD data-plane packets from QoS
`192.168.100.1 <-> 192.168.100.2`

```
!
! Config for 192.168.100.1
!
ipv4 access-list BFD-TRAFFIC
 5 permit udp host 192.168.100.1 any range 3784 3785
 10 permit udp host 192.168.100.2 any range 3784 3785
!
class-map match-any BFD-CLASS
 match access-group ipv4 BFD-TRAFFIC
!
policy-map OUT
class BFD-CLASS
 priority level 1
 police rate 10 kbps
!
interface TenGig <>
 service-policy output OUT
 bfd address-family ipv4 multiplier 3
 bfd address-family ipv4 destination 192.168.100.1
 bfd address-family ipv4 fast-detect
 bfd address-family ipv4 minimum-interval 100
!
```

## Enabling BFD on RSVP (IOS)

###### A Config
```
ip rsvp signalling bfd hello
!
! this very dangerous because CPU load will affect processing of BFD control packets
!
int f0/0.45
 ip rsvp signalling hello bfd
 bfd interval 50 min_rx 50 multiplier 3
```

###### Verification 
`show ip rsvp hello bfd nbr`