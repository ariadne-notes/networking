# Terms
* **DIA:** Direct Internet Access. What we usually have has residential customers. No real guarantee of service, but tends to be fast.
* **SLA:** Service Level Agreement. Business Internet, especially, to connect sites together tends to have a SLA.
* **MPLS:** A kind of VPN service provided by an ISP, to connect business sites together. Comes with a SLA. More expensive than DIA.
* **BFD:** Bidirectional Forwarding Detection

# Devices

* **Manager:** AKA vManage, AKA, the NMS. What a human interacts with, the GUI
* **Validator:** AKA vBond. Initial Authentication and provisioning, (Cisco calls this orchestration) Responsible for NAT traversal.
* **Controller:** AKA vSmart. Holds the current state of the network, (routes and data policy) maintains active connections to the edges and programs them.
* **WAN Edge:** AKA vEdge. What gets programmed. Provides data-plane between sites, via circuits like DIA, or MPLs.
* **vEDGE:** Old hardware-based Viptela gear, pre-Cisco acquisition. Unfavored.

# Marketing Terms
* **Cisco SD-WAN Cloud OnRamp:** AKA, CoR. Edges can perform analytics to SaaS or IaaS offerings to select the best path, via jitter.

## Validator
Should be give a FQDN, so WAN edges have no problems finding it on connection to a DIA.

FQDNs also mean we aren't putting a static IP into a config.

Initial authentication is done with PKI, and RSA encryption.

Can not be placed behind NAT, unless the NAT device does a 1:1 static translation.

This device does the load balancing if multiple controllers are being used.

The Validator has a permanent dTLS tunnel to all the controllers.

## Controllers
* Keeps all the routes between sites, that are managed via the OMP protocol (like BGP, but proprietary)
* Logical tunnel topologies (such as hub and spoke, regional, and partial mesh)
* Service Chaining
* Traffic Engineering
* Segmentation per VPN 

## WAN Edge
* Dataplane for a site
* Has OMP, BGP, OSPF, EIGRP, ACLs, ARP, HA, and QoS.
* Connects via dTLS to the controllers.
* Connects via dTLS to other edges.

## SD-WAN Policy
Policies are further classified as

* **Local Policy:** Programed on the edges. ACLs, QoS, routing, and AAA.
* **Centralized Policy:** Route policy, before being sent to the edges, (Topology, VPN Membership, Application Aware Routing)

## Application Aware Routing
* If two edges connect to each other over dTLS, BFD is run over the tunnel.
* For AAR, or CoR, the edge will send HTTP probes and measure the jitter and/or loss.
* The score for an app is the vQoS (Viptela Quality of Experience) from 0 to 10, 10 being best.

# VPNs

**VPN0:** Underlay Signaling, transport WAN. Typically public addresses or SRC-NAT Public addresses.

**VPN512:** OOB Management

**VPNn:** Any number from 1 to 65527. Not 0. Not 512. Used for service-side (also known as LAN-side) traffic.



# sd-wan commands

`show sdwan control local-properties`

# DTLS Tunnels to SDWAN Manager and SDWAN Controllers
`show sdwan control connections`

`show sdwan control connection-history`

# OMP
`show sdwan omp peers`

`show sdwan omp routes`

`show sdwan omp tlocs`

`show sdwan omp services`

`show sdwan omp multicast-routes`

# Validator Only
`show orchestrator connections`

# Initial Bringup

### Pasting in the bootstrap
```
tclsh
puts [open "bootflash:name-of-bootstrap-file.cfg" w+] {
<list of certs goes here>
<must be done via an actual terminal>
<like SecureCRT>
<with character and line send delay>
}
```

### Copy via HTTP using Python

1. Get the current IP

`python -c "import socket; s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM); s.connect(('8.8.8.8', 80)); print(s.getsockname()[0]); s.close()"`

1. Start the server with above IP

`python -m http.server 8000 --bind 10.0.0.1`

1. Copy into cisco box

copy tftp://10.0.0.1:8000/<boot-strap>.cfg bootflash:/<bootstrap>.cfg

**controller-mode enable**


