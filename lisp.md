A very basic setup, that assumes a working underlay. I implemented this on my home lab of c7200s in GNS3 running `15.2(4)S7`. My underlay was IS-IS to router loopbacks.

```
Site 1 EIDs - 192.168.100.0/24
Site 2 EIDs - 192.168.101.0/24

xTR for Site 1 - Lo0 16.16.16.16
xTR for Site 2 - Lo0 19.19.19.19
```
 
#### Site 1 - xTR
##### config
```
R18# show run | s lisp
router lisp
 database-mapping 192.168.100.0/24 18.18.18.18 priority 1 weight 50
 ipv4 itr map-resolver 16.16.16.16
 ipv4 itr
 ipv4 etr map-server 16.16.16.16 key cisco
 ipv4 etr
 exit
```
##### verify 
```
R18# show ip lisp map-cache 
LISP IPv4 Mapping Cache for EID-table default (IID 0), 2 entries

0.0.0.0/0, uptime: 00:19:42, expires: never, via static send map-request
  Negative cache entry, action: send-map-request
192.168.101.0/24, uptime: 00:10:08, expires: 23:49:44, via map-reply, complete
  Locator      Uptime    State      Pri/Wgt
19.19.19.19  00:10:08  up           1/50 
```

#### Site 2 - xTR
##### config
```
R19# show run | s lisp
router lisp
 database-mapping 192.168.101.0/24 19.19.19.19 priority 1 weight 50
 ipv4 itr map-resolver 16.16.16.16
 ipv4 itr
 ipv4 etr map-server 16.16.16.16 key cisco
 ipv4 etr
 exit
```
##### verify
```
R19#show ip lisp map-cache 
LISP IPv4 Mapping Cache for EID-table default (IID 0), 2 entries

0.0.0.0/0, uptime: 00:11:50, expires: never, via static send map-request
  Negative cache entry, action: send-map-request
192.168.100.0/24, uptime: 00:11:29, expires: 23:48:23, via map-reply, complete
  Locator      Uptime    State      Pri/Wgt
  18.18.18.18  00:11:29  up           1/50
```
##### MS/MR
###### config
```
R16# show run | s lisp
router lisp
 site 1
  authentication-key cisco
  eid-prefix 192.168.100.0/24
  exit
 !
 site 2
  authentication-key cisco
  eid-prefix 192.168.101.0/24
  exit
 !
 ipv4 map-server
 ipv4 map-resolver
 exit
```
##### verify
```
R16# show lisp site name 1
Site name: 1
Allowed configured locators: any
Allowed EID-prefixes:
  EID-prefix: 192.168.100.0/24 
    First registered:     00:25:12
    Routing table tag:    0
    Origin:               Configuration
    Merge active:         No
    Proxy reply:          No
    TTL:                  1d00h
    State:                complete
    Registration errors:  
      Authentication failures:   0
      Allowed locators mismatch: 0
    ETR 10.0.0.23, last registered 00:00:28, no proxy-reply, no map-notify
                   TTL 1d00h, no merge, nonce 0x3E715231-0x150380FC
                   state complete
      Locator      Local  State      Pri/Wgt
      18.18.18.18  yes    up           1/50 

R16# show lisp site name 2
Site name: 2
Allowed configured locators: any
Allowed EID-prefixes:
  EID-prefix: 192.168.101.0/24 
    First registered:     00:25:24
    Routing table tag:    0
    Origin:               Configuration
    Merge active:         No
    Proxy reply:          No
    TTL:                  1d00h
    State:                complete
    Registration errors:  
      Authentication failures:   0
      Allowed locators mismatch: 0
    ETR 10.0.0.26, last registered 00:00:37, no proxy-reply, no map-notify
                   TTL 1d00h, no merge, nonce 0x2F281A3C-0x0760FD58
                   state complete
      Locator      Local  State      Pri/Wgt
      19.19.19.19  yes    up           1/50 
```

#### A Packet (an ICMP Request)

[Capture is here](https://github.com/ariadne-notes/networking/blob/main/packet-captures/lisp/lisp-icmp-ping.pcapng)

```
Frame 4156: 134 bytes on wire (1072 bits), 134 bytes captured (1072 bits) on interface -, id 0
Ethernet II, Src: ca:17:30:54:00:08 (ca:17:30:54:00:08), Dst: ca:1a:39:b0:00:08 (ca:1a:39:b0:00:08)
Internet Protocol Version 4, Src: 10.0.0.24, Dst: 19.19.19.19
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
    Total Length: 120
    Identification: 0x0096 (150)
    010. .... = Flags: 0x2, Don't fragment
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 63
    Protocol: UDP (17)
    Header Checksum: 0x0aa2 [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 10.0.0.24
    Destination Address: 19.19.19.19
User Datagram Protocol, Src Port: 1024, Dst Port: 4341
    Source Port: 1024
    Destination Port: 4341
    Length: 100
    Checksum: 0x0000 [zero-value ignored]
    [Stream index: 2]
    [Timestamps]
    UDP payload (92 bytes)
Locator/ID Separation Protocol (Data)
    Flags: 0xc0
    Nonce: 939002 (0x0e53fa)
    0000 0000 0000 0000 0000 0000 0000 0001 = Locator-Status-Bits: 0x00000001
Internet Protocol Version 4, Src: 192.168.100.100, Dst: 192.168.101.100
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
    Total Length: 84
    Identification: 0xc736 (50998)
    010. .... = Flags: 0x2, Don't fragment
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 63
    Protocol: ICMP (1)
    Header Checksum: 0x2959 [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 192.168.100.100
    Destination Address: 192.168.101.100
Internet Control Message Protocol
    Type: 8 (Echo (ping) request)
    Code: 0
    Checksum: 0xc078 [correct]
    [Checksum Status: Good]
    Identifier (BE): 82 (0x0052)
    Identifier (LE): 20992 (0x5200)
    Sequence Number (BE): 1 (0x0001)
    Sequence Number (LE): 256 (0x0100)
    [Response frame: 4157]
    Timestamp from icmp data: Jul 20, 2023 18:00:03.000000000 Eastern Daylight Time
    [Timestamp from icmp data (relative): 0.551525000 seconds]
    Data (48 bytes)

0000  53 4e 08 00 00 00 00 00 10 11 12 13 14 15 16 17   SN..............
0010  18 19 1a 1b 1c 1d 1e 1f 20 21 22 23 24 25 26 27   ........ !"#$%&'
0020  28 29 2a 2b 2c 2d 2e 2f 30 31 32 33 34 35 36 37   ()*+,-./01234567
```

### Sources

[LISP Fundamentals and Troubleshooting Basics - Cisco](https://community.cisco.com/t5/networking-knowledge-base/lisp-fundamentals-and-troubleshooting-basics/ta-p/3155439)
