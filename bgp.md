## Theory
- BGP works on the premise that if a router sees its own AS path, it must be a loop.
- The default timer is 60 seconds with 180 seconds for hold time. This means worst-case is 3 minutes to fail-over.
- BGP `aggregate-address` only works if there is a subnet inside the aggregate range in BGP.

#### Working with BGP
 - Only consider traffic in one direction at a time
 - Accepting a route will affect outgoing traffic
 - Advertising a route will affect incomming traffic
 - Filter out everything except the routes needed
 - BGP DOES NOT LOAD BALANCE

On Cisco IOS `bgp soft-reconfig-backup` tells the router "if you must, save a entire table" otherwise rely on [RFC2918](https://www.rfc-editor.org/rfc/rfc2918), which are *dynamic updates.*

Soft reconfig is ancient, pre-RFC.

#### Soft Reconfig via Route Refresh (trusting the other device)!

`clear ip bgp <neighbor_ip> soft in`[^clear-soft]

[^clear-soft]: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_bgp/configuration/xe-16/irg-xe-16-book/bgp-4-soft-configuration.html

#### Example of a BGP AS Path

These read left to right like a book. This prefix was most recently from AS `7018`.

```
7018 701 15 i
            ^ this means IGP, and AS 15 has an IGP route for it like OSPF or EIGRP
```

### BGP Best Path Selection
```
- Higher Weigth                                       
- Higher Local Preference                            
- Locally Originated                                 (Network or Aggregate Command)
- Shortest AS-PATH
- Lowest Origin Type                                 (IGP > EGP > Incomplete)
- Lowest MED                                         (Neighbor ASes must be the same)
- Prefer eBGP > Confederated eBGP > iBGP
- Prefer path with lowest IGP metric to next hop
- Determine if bestpath is enabled
  - Prefer external path which is oldest
  - Prefer path from router with lower ID
  - Prefer path with shorter cluster length
  - Prefer path from lowest neighbor address
```
[Cisco - Select BGP Best Path Algorithm](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/13753-25.html)

#### BGP Path Attributes
[RFC 4271 - BGP-4](https://www.rfc-editor.org/rfc/rfc4271#section-5) 

- Well-known mandatory
- Well-known discretionary
- Optional transitive
- Optional nontransitive

| Path Attribute | Category |
| -- | -- | 
| Origin             | Mandatory                 |
| AS_PATH            | Mandatory                 |
| NEXT_HOP           | Mandatory                 |
| LOCAL_PREF         | Discretionary             |
| ATOMIC_AGGREGATE   | Discretionary             |
| AGGREGATOR         | Optional Transitive       |
| COMMUNITY          | Optional Transitive       |
| MULTI_EXIT_DISC    | Optional Non-Transitive   |
| ORIGINATOR_ID      | Optional Non-Transitive   |
| CLUSTER_LIST       | Optional Non-Transitive   |

### Origin

IGP > EGP > Incomplete
- IGP means it came from an IGP. This is the highest preference.
- Incomplete means its likely a redistributed route

### Next Hop
1. eBGP, routers in different AS, destination outside AS.  The Next hop will be the advertising router.
1. iBGP, routers in same AS,      destination inside AS.   The Next hop will be the advertising router.
1. iBGP, routers in same AS,      destination outside AS.  The Next hop is the external peer who advertised the address.

... When the third option happens ...
- Advertise into the IGP the external links to the BGP peers.
- Tell the AS border router to change the next hop to its own IP address. [next-hop-self]

### LOCAL_PREF
- Controls traffic Outgoing traffic.
- Only shared between iBGP peers, used to determine the exit. Higher is better.

### MULTI_EXIT_DISC
- Controls incoming traffic.
- Lower is better

### ATOMIC_AGGREGATE
BGP can aggregate smaller prefixes into larger ones even if a smaller prefix comes from a different AS.

A router in AS 105 gets these prefixes from its peers.

```
192.168.0.0/24 (123 204)
192.168.1.0/24 (123 205)
```

If the administrator chooses, they can aggregate this, but lose path information.
```
192.168.0.0/23 (105) ATOMIC_AGGREGATE. 
```
Downstream peers can not remove this tag

### AGGREGATOR
AS and Router ID of the BGP router that did the atomic aggregation.

### COMMUNITY
Usually used to tag routes from a specific customer.

Tag | Purpose
-------------|--------------------------
INTERNET     | Default community.
NO_EXPORT    | Do not share with other ASes
NO_ADVERTISE | Do not share with other routers
LOCAL_AS     | ????

### ORIGINATOR_ID 
For route reflectors
The origaning router puts its `Router_ID` here. If it sees this, it knows a loop as occured.

### CLUSTER_LIST
- For route reflectors
- The sequence of `Router_IDs` through which the route has passed. If a router seeis its Router_ID a loop has occured.

### WEIGHT
- Cisco specific & this router only
- Routes learned are 0
- Locally generated routes are 32768

#### Route Reflectors
	
A RR will not change any attributes of a route.
- If a route is learned from a non-client iBGP peer, reflect to clients
- If a route is learned from a client, reflect to everyone
- If a route is learned from a eBGP peer, reflect to everyone

Only the route reflector is aware of the reflecting. The clients are dumb

If you configure route reflectors as a cluster you must manually configure the `cluster_ID`

##### BGP by default will summarize. 
Use `no auto-summary`.

Using redistribute under BGP will make the resulting route show up with an orign code of `incomplete`.

##### Sending a default route
`neighbor A.B.C.D default-originate` 

To get iBGP routers to update the next-hop to be themselves when advertising to other iBGP routers use

`neighbor A.B.C.D next-hop-self` 

This makes it so other iBGP routers don't need reachability information for the physical link to the next AS.

#### BGP Finite State Machine
* **Idle** -  check the config
* **Connect** -  TCP is probably broken
* **Active** -  Listening for TCP
* **OpenSent**
* **OpenConfirm**
* **Established**

#### Fixing next-hop issues

Just because the route shows up in `show ip bgp` doesn't mean it will install. BGP needs to be able to reach the next-hop.

1. Add the transit routes the IGP.
2. Use next-hop self in BGP.
3. Use a route-map to set the next hops.

#### Route Reflection
##### Terms
* **Cluster List** - Router ID of the route Reflector. Used to prevent loops between RRs.
* **Originator** - Route reflector peer.  Used to prevent loops between clients.

##### Three rules for route reflectors
  - If the route is recieved from a non-client peer, reflect to clients only.
  - If the route is recieved from a client peer, reflect to non-client peers, and client peers.
  - If the route is recieved from an EBGP peer, reflect to all client and non-client peers.

##### Notes
- Route reflectors can be clients of each other. This causes extra overhead.
- If multiple route reflectors server the same cluster they should have the same `Cluster_ID`.

##### BGP Route Reflectors Loop Prevention
- If a BGP router that receives a route from an iBGP neighbor in the incoming update detects the presence of its own Router-ID in the Originator-ID attribute it will reject the update.
- If a BGP router that receives a route from an iBGP neighbor is configured to operate as a route reflector and in the incoming update detects the presence of its own `Cluster-ID` in the
  `Cluster-list` attribute it will reject the update.

##### Confederations
	
`NEXT_HOP` is preserved throughout the confederation.

`MED` is preserved for routes advertised into the confederation
	
`LOCAL_PREF` is preserved throughout the confederation
	
`AS_PATH` for privates ASes is used within the confederation
 
###### Force interior confederation MEDs to be considered:
`bgp deterministic-med`

Route Reflectors are generally preferred.

IF you want to add two BGP speakers to the same router reflector cluster, specify the cluster ID.
- clients can not detect inter-cluster loops. They don't have the attributes in the BGP table.

### BGP redistribution into anything

  
