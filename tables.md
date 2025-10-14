#### IP Protocol Numbers

When IP encapsulates another protoctol it labels the `protoctol` field with a number to define the next layer. 

| IP Protocol Number | Description |
| ------------------ | ----------- |
| 1                  | ICMP        |
| 2                  | IGMP        |
| 6                  | TCP         |
| 17                 | UDP         |
| 46                 | RSVP        |
| 47                 | GRE         |
| 51                 | ESP (IPSec) |
| 51                 | AH (IPSec)  |
| 69                 | TFTP        |
| 88                 | EIGRP       |
| 89                 | OSPF        |
| 103                | PIM         |
| 112                | VRRP        |
| 115                | L2TP        |
| 161                | SNMP        |
| 162                | TRAPS       |

[Protocol Numbers - IANA](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)

#### Cisco Administrative Distance

| Protocol                        | Administrative Distance |
| ----------------                | ----------------------- |
| Connected                       | 0                       | 
| Static                          | 1                       |
| EIGRP Summary                   | 5                       | 
| eBGP                            | 20                      | 
| EIGRP Internal                  | 90                      | 
| OSPF                            | 110                     | 
| IS-IS                           | 115                     | 
| RIP                             | 120                     | 
| ODR                             | 160                     | 
| EIGRP External                  | 170                     | 
| iBGP                            | 200                     | 
| Unknown/Infinite[^infinite]     | 255                     | 

[Troubleshooting TechNotes - What is Administrative Distance? - Cisco](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/15986-admin-distance.html#topic2)

[^infinite]:Can use to do route-filtering.