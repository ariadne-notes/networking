## EIGRP Terminology

* **Successor route:** The current best path, with the smallest metric. The "successful" route.

* **Successor:** The first next-hop router for the successor route.

* **Feasible distance (FD):** Lowest metric to reach a subnet. The sum of the RD + local cost.

* **Reported distance (RD):** The metric inside a route update from another router. The sending router included it's FD, which becomes out RD.

* **Feasibility condition:** If another path is *actually a backup*, the RD will be less than the current FD.

* **Feasible successor:** A route that satisfies the feasibility condition and is maintained as a backup route.

* **Split Horizon:** Never advertise a network, out the same interface it was learned on.

* **Poison Reverse:** If you must advertise a network out the same interface it was received on, advertise the delay as infinity.
                                                                      
## Example.

R2 sends an update
 - 10.0.0.0/24 - RD is 2000

R3 Sends an update
 - 10.0.0.0/24 - RD is 2050

R1 calculates total path metric.
 - R2 is 2000 + 1000 = 3000.
 - R3 is 2050 +   50 = 2100.  < - Successor route.
 
R1 sees it has an reported distance less than the current distance, so installs that route as the feasible successor.

```                                                                      
+--------+            1000             +--------+    10.0.0.0/24      
|   R1   +-----------------------------+   R2   +---------------------                           
+-----+--+                             +-+------+      2000                                                                               
      |            +--------+            |                            
      +------------+   R3   +------------+                                                                                                        
         50        +--------+      50
```                   

# Example with the EIGRP topology table

```
R1# show ip eigrp topology 10.0.0.0/24
EIGRP-IPv4 Topology Entry for AS(1)/ID(1.1.1.1) for 10.0.0.0/24
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 2100
P 10.0.0.0/24, 1 successors, FD is 2100                <--- Feasible Distance
        via 10.0.13.3 (2100/2050), GigabitEthernet0/3  <--- Successor Route
        via 10.0.12.2 (3000/2000), GigabitEthernet0/2  <--- Feasible Successor
                       |     |
                       |     +-- Reported Distance 
                       +-------- Path Metric
        
                                                             (RD 2000 < FD 2100)
```

# Metric calculation

metric = ([K1 * bandwidth + (K2 * bandwidth) / (256 - load) + K3 * delay] * [K5 / (reliability + K4)]) * 256

K1, set to 1
K3, set to 1

Wide metrics allow for faster links.

# Unequal Cost Multi Path
EIGRP can load balance over the successor and feasible successor routes with a variance command.

# Timers
- Hello packets are every 5 seconds, on 60 seconds on T1 links.
  - The deadtime is 3x the hold timer.



# Initial Bringup
- Send Hello packets, to 224.0.0.10
  - Doesnt' require multicast to be on
  - Unicast Init from neighbor, set Seq, Set Ack to 0
    - Neighbor Sends back Ack as prior sequence number.
    - Update Messages
    
# Stuck in Active

- The router is too busy to answer the query (generally due to high CPU utilization).
- The router has memory problems and cannot allocate the memory to process the query or build the reply packet.
- The circuit between the two routers is not good; there are not enough packets that get through to keep the neighbor relationship up, but some queries or replies are lost between the routers.
- unidirectional links (a link on which traffic can only flow in one direction because of a failure)


# Update Message
- AS number
- Prefixes
- End-of-table Flag

# Prefixes
- Type (internal, etc)
- Reliability
- Load
- MTU
- Hop Count
- Delay
- Bandwidth
- Flags
  - Source Withdrawn
  - Candidate Default
  - Route is Active
  - Route is Replicated
- Next-hop
- Prefix Length

# Network

* The CLI parser is converting the IP into binary, then comparing it to the wild mask.
* The CLI parser will only save the matched bits of the IP.
* The CLI parser will not save the zeroth network, anything starting with 0.
* The CLI parser will only save the matched bits of an IP if if finds bits that are "on"
* Using the "all" mask of 255.255.255.255 creates this statement 'network 0.0.0.0' and matches everything.
* Using the "unique-ip" mask of 0.0.0.0 means "match this single address"
* The wildcard mask only accepts contiguous numbers "Discontiguous mask is not supported."

192.0.2.5 127.255.255.255 - becomes 128.0.0.0, the rest of the bits get dropped.

# References
https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/16406-eigrp-toc.html