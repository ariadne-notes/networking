## Terms

| Term                                         | Definition                                                                      |
| -------------------------------------------- | --------------------------------------------------------------------------------|
| MR-APS                                       | inter-chassis APS.                                                              |
| APS                                          |  Automatic Protection Switching for POS                                         |
| UNI                                          |  User Network Interface                                                         |
| NNI                                          |  Network Node Interface                                                         |
| Interworking                                 |  Getting L2 information from Ethernet to work over Sonet or frame relay.        |
| STE                                          | Section Terminating Equipment                                                   |
| LTE                                          | Line terminating equipment                                                      |
| PTE                                          | Path terminating equipment                                                      |
| POH                                          | Path overhead - This layer represents end-to-end status.                        |
| LOH                                          | Line overhead - Typically major nodes in SONET like ADMs                        |
| SOH                                          | Section overhead - Optical regenators                                           |
| SPE                                          | Synchronous payload envelope                                                    |
| BIP                                          | Bit Interleaved Parity                                                          |
| FEBE                                         | Far End Block Error                                                             |

## Sonet

Path Payloads must match. Check Scrambling.

Network elements are expected to terminate and understand their layer, and layer overhead

If a SONET reciever at the Line level counts a BIP, it returns it to sender. The sender increments the line FEBE

It's been a while, the below might be wrong.

```
+-------------------------------------------------- PATH -------------------------------------------------+
|                                                                                                         |
|                                                                                                         |
|   +--------------- LINE --------------------+            +------------------ LINE-------------------+   |
|   |                                         |            |                                          |   |
v   v                                         v            v                                          v   v

+---+      +------------+       +-----+       +------------+      +-----+       +------------+        +---+
|CPE|------|Terminal    |-------|Regen|-------|Add/Drop    |------|Regen|-------|Terminal    |--------|CPE|
+---+ DS-n | Multiplexer| OC-N  +-----+ OC-N  | Multiplexer| OC-N +-----+ OC-N  | Multiplexer|  DS-n  +---+
           +------------+                     +------------+                    +------------+

    ^      ^            ^       ^     ^       ^            ^      ^     ^       ^            ^        ^
    |      |            |       |     |       |            |      |     |       |            |        |
    +------+            +-------+     +-------+            +------+     +-------+            +--------+
    SECTION              SECTION       SECTION             SECTION       SECTION              SECTION
```

### C2 Byte

C2 Defines the SONET payload

An old note, probably from a standard document.

    The SONET standard defines the C2 byte as the path signal label. The purpose of this byte 
    is to communicate the payload type that the SONET Framing OverHead (FOH) encapsulates. 
    The C2 byte functions similar to Ethertype and Logical Link Control (LLC)/Subnetwork 
    Access Protocol (SNAP) header fields on an Ethernet network. The C2 byte allows a single
    interface to transport multiple payload types simultaneously.

This table lists common values for the C2 byte:

| Hex Value  | SONET Payload Contents                                          |
| ---------- | --------------------------------------------------------------- |
| 00         | Unequipped.                                                     |
| 01         | Equipped - non-specific payload.                                |
| 02         | Virtual Tributaries (VTs) inside (default).                     |
| 03         | VTs in locked mode (no longer supported).                       |
| 04         | Asynchronous DS3 mapping.                                       |
| 12         | Asynchronous DS-4NA mapping.                                    |
| 13         | Asynchronous Transfer Mode (ATM) cell mapping.                  |
| 14         | Distributed Queue Dual Bus (DQDB) cell mapping.                 |
| 15         | Asynchronous Fiber Distributed Data Interface (FDDI) mapping.   |
| 16         | IP inside Point-to-Point Protocol (PPP) with scrambling.        |
| CF         | IP inside PPP without scrambling.                               |
| E1- FC     | Payload Defect Indicator (PDI).                                 |
| FE         | Test signal mapping (see ITU Rec. G.707).                       |
| FF         | Alarm Indication Signal (AIS).                                  |

An Example:
```
Framing: SONET
SPE Scrambling: Enabled
C2 State: Stable   C2_rx = 0xCF (207)   C2_tx = 0x16 (22) / Scrambling Derived
S1S0(tx): 0x0  S1S0(rx): 0x2 / Framing Derived
```
    
Monitoring at each Network Element is usually helpful
    
POS - Spawned interface from SONET controller.

`controller SONET0/2/0/0`

`clock source internal`


Sonet YELLOW is RDI (Remote Defect indication)

### Packet Over Sonet
```
Document: Troubleshooting Bit Error on SONET Links
URL: http://www.cisco.com/en/US/tech/tk482/tk607/technologies_tech_note09186a0080094a79.shtml
Section: When Do Particular BIP Errors Occur?

In addition, you must understand that BIP errors have different error detection resolutions, which are explained here:

B1: B1 can detect up to eight parity errors per frame. This level of resolution is not acceptable at OC-192 rates. Even-numbered errors can elude the parity check on links with high error rates.

B2: B2 can detect a far higher number of errors per frame. The exact number increases as the number of STS-1s (or STM-1s) increases in the SONET frame. For example, an OC-192/STM-64 produces a 192 x 8 = 1536 bit-wide BIP field. In other words, B2 can count up to 1536 bit errors per frame. There is considerably less chance of an even-numbered error that eludes the B2 parity calculation. B2 offers superior resolution when compared to B1 or B3. Therefore, a SONET interface can report B2 errors only for a particular monitored segment.

B3: B3 can detect up to eight parity errors in the entire SPE. This number produces acceptable resolution for a channelized interface because, (for example) each STS-1 in an STS-3 has a path overhead and B3 byte. However, this number produces poor resolution over concatenated payloads in which a single set of path overhead must cover a relatively large payload frame. 
```

##### Packet over SONET commands

##### Displays information about the automatic protection switching feature
`show aps`

##### Displays information about the hardware
`show controller sonet slot/port-adapter/port`

##### Displays information about the interface
`show controllers pos`


## G709
G709 is an optical specification that is specifcially designed for FEC (Forward Error correction)
	It uses Reed-Solomon to produce redundant information that can be used to rebuild the frame.

* **OTU** - Optical channel Transport Unit

* **ODU** - Optical channel Data Unit

* **OPU** - OPtical channel Payload Unit

## SRP - Spatial Reuse protocol

This is used for fiber rings, its where the destination nodes pulls the info from the ring so it doesn't loop endlessly.

Like taken from a standards document someplace

```
Spatial Reuse Protocol (SRP) is a media-independent MAC layer protocol that operates over two counterrotating
fiber-optic rings. The dual rings provide survivability of data in case of a failed node or a break in 
connecting cables by rerouting the data path over the alternate ring. SRP provides a more efficient use of 
bandwidth by having packets traverse only the part of the ring necessary to get to the destination node. Once
the packet has reached the destination node, it is removed from the ring, allowing other parts of the ring
to reuse the bandwidth. Data packets travel on one ring, while associated control packets travel in the opposite
direction on the alternate ring, ensuring that the data takes the shortest path to its destination.
```

## RPR - Resilient Packet Ring

802.17

* **Steering** - Nodes are told the affected node is down and don't include it.
* **Wrapping** - The node closest to the break route the traffic on the other direction of the ring.

Side A Always connects to Side B.

Example of a working connection.

```
Node2# show controller srp 4/0
SRP4/0 - Side A (Outer RX, Inner TX)
SECTION
  LOF = 0          LOS    = 0                            BIP(B1) = 3
LINE
  AIS = 0          RDI    = 0          FEBE = 36599      BIP(B2) = 46
PATH
  AIS = 0          RDI    = 0          FEBE = 4440       BIP(B3) = 26
  LOP = 0          NEWPTR = 0          PSE  = 0          NSE     = 0

Active Defects: None
Active Alarms:  None
Alarm reporting enabled for: SLOS SLOF PLOP 

Framing           : SONET
Rx SONET/SDH bytes: (K1/K2) = 0/0        S1S0 = 0  C2 = 0x16
Tx SONET/SDH bytes: (K1/K2) = 0/0        S1S0 = 0  C2 = 0x16  J0 = 0x1 
Clock source      : Internal
Framer loopback   : None
Path trace buffer : Stable 
  Remote hostname : Node1
  Remote interface: SRP4/0
  Remote IP addr  : <removed>
  Remote side id  : B
BER thresholds:           SF = 10e-3  SD = 10e-6
IPS BER thresholds(B3):   SF = 10e-3  SD = 10e-6
TCA thresholds:           B1 = 10e-6  B2 = 10e-6  B3 = 10e-6

SRP4/0 - Side B (Inner RX, Outer TX)
SECTION
LOF = 0          LOS    = 0                            BIP(B1) = 65535
LINE
AIS = 0          RDI    = 0          FEBE = 65535      BIP(B2) = 65535
PATH
AIS = 0          RDI    = 0          FEBE = 65535      BIP(B3) = 65535
LOP = 0          NEWPTR = 3          PSE  = 0          NSE     = 0
Active Defects: None
Active Alarms:  None
Alarm reporting enabled for: SLOS SLOF PLOP 
Framing           : SONET
Rx SONET/SDH bytes: (K1/K2) = 0/0        S1S0 = 0  C2 = 0x16
Tx SONET/SDH bytes: (K1/K2) = 0/0        S1S0 = 0  C2 = 0x16  J0 = 0x1 
Clock source      : Internal
Framer loopback   : None
Path trace buffer : Stable 
Remote hostname : Node3
Remote interface: SRP4/0
Remote IP addr  : <removed>
Remote side id  : A
BER thresholds:           SF = 10e-3  SD = 10e-6
IPS BER thresholds(B3):   SF = 10e-3  SD = 10e-6
TCA thresholds:           B1 = 10e-6  B2 = 10e-6  B3 = 10e-6
```

##### References

[SONET Primer](https://www.cisco.com/c/en/us/td/docs/optical/15000r5_0/planning/guide/r50engpl/r50appb.pdf)