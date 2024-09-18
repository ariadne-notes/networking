* **SA** - Source Address

* **DA** - Destination Adress

```
                      INSIDE NETWORK                                   OUTSIDE NETWORK

           ┌────────────────────────────────────┐         ┌──────────────────────────────────────┐
           │                                    │         │                                      │
           │       ┌────────────┬─────────────┐ │         │       ┌─────────────┬──────────────┐ │
           │ ────► │    SA      │     DA      │ │ ──────► │ ────► │    SA       │     DA       │ │
  ┌──────┐ │       │Inside Local│Outside Local│ │         │       │Inside Global│Outside Global│ │ ┌───────┐
  │Inside│ │       └────────────┴─────────────┘ │  ┌───┐  │       └─────────────┴──────────────┘ │ │Outside│
  │ Host │ │                                    │  │NAT│  │                                      │ │ Host  │
  └──────┘ │ ┌────────────┬─────────────┐       │  └───┘  │ ┌─────────────┬──────────────┐       │ └───────┘
           │ │    SA      │     DA      │       │         │ │    SA       │     DA       │       │
           │ │Inside Local│Outside Local│ ◄──── │ ◄────── │ │Inside Global│Outside Global│ ◄──── │
           │ └────────────┴─────────────┘       │         │ └─────────────┴──────────────┘       │
           │                                    │         │                                      │
           └────────────────────────────────────┘         └──────────────────────────────────────┘
```

Based on a diagram [here.](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/13772-12.html)

# NAT Overload - Port Address Translation or PAT

This is Source NAT.[^source] 

Packets to R3 will appear to be from `10.0.0.2`

<pre>
          192.168.0.0/24             10.0.0.0/24        
┌────┐.1                 .2┌────┐.2             .1┌────┐
│ R1 │─────────────────────│ R2 │─────────────────│ R3 │
└────┘E0/0             E0/0└────┘E0/1         E0/1└────┘
                           ▲    ▲                       
                           │    │                       
           Inside ─────────┘    └─────── Outside        
</pre>           

### R1
<pre>
interface Ethernet0/0
 ip address 192.168.1.1 255.255.255.0

ip route 0.0.0.0 0.0.0.0 192.168.1.2
</pre>
### R2
<pre>
interface Ethernet0/0
 ip address 192.168.1.2 255.255.255.0
 ip nat inside

interface Ethernet0/1
 ip address 10.0.0.2 255.255.255.0
 ip nat outside

ip nat inside source list 1 interface Ethernet0/1 overload

ip access-list standard 1
 10 permit 192.168.1.0 0.0.0.255
</pre>
### R3
<pre>
interface Ethernet0/1
 ip address 10.0.0.3 255.255.255.0

ip route 0.0.0.0 0.0.0.0 10.0.0.2
</pre>
## R2 Debugs during NAT
Performed with the above configs via [CML](https://developer.cisco.com/modeling-labs/) IOL routers version 17.12.1.

<pre>
R2# debug ip nat 1
IP NAT debugging is on for access list 1

*Sep 16 21:32:21.386: NAT: Entry assigned id 4
*Sep 16 21:32:21.386: NAT*: ICMP id=5->1024
*Sep 16 21:32:21.386: NAT*: s=192.168.1.1->10.0.0.2, d=10.0.0.3 [17]
*Sep 16 21:32:21.387: NAT*: ICMP id=1024->5
*Sep 16 21:32:21.387: NAT*: s=10.0.0.3, d=10.0.0.2->192.168.1.1 [17]

R2# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.0.0.2:1024     192.168.1.1:5      10.0.0.3:5         10.0.0.3:1024
</pre>

[^source]: Source NAT, because the source address needs to be changed to access outside hosts. As packets move through the router, they will create entries for return packets.
