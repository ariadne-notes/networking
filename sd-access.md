* **Cisco Catalyst Center:** Formerly Cisco DNA center. Speaks NETCONF, SNMP, SSH southbound, REST/HTTPS Northbound.

* **Campus Fabric:** Equipment managed without Catalyst Center, can be CLI or NETCONF/RESTCONF.

* **ISE:** Identity Services Engine. Cisco's modern AAA server.

* **SD-Access:** Campus Fabric managed with Cisco Catalyst Center and Cisco ISE.

* **SGT:** Scalable Group tags, formally called Security Group Tags. These are managed by ISE.

* **SGT Policy:** Instead of identifying traffic based on IP or MAC, traffic can be identified by SGT.

* **Overlay:** LISP, VXLAN and CTS (Cisco TrustSec, carries SGTs inside of VXLAN-GPO.

* **VXLAN-GPO:** Cisco extended the VXLAN header to include SGTs (Now called Scalable Group Tags)

* **Underlay:** Usually IS-IS, since it's IPv4 and IPv6 agnostic. Even the underlay can be automatically deployed.

* **Control Plane Node:** Contains the LISP MS/MR databases Endpoint-to-location, or EID-to-RLOC. Each node contains the full database.

* **Fabric Border Node:** Connects other L3 networks to SDA fabric.

* **Fabric WLC:** Connects APs and the WLC to the SDA fabric.

* **Fabric Intermediate Node:** Only does underlay services, like IS-IS or IP transport.

* **Fabric Edge Node:** Connects campus host devices to the SDA fabric, usually an access layer or distribution layer device. Is a LISP xTR, with an anycast gateway, with overlay host protocols, (like DHCP).

## Fabric Edge Onboardin
* (Method 1) Open Auth or MAB, user connects to a port -> host pool.
* (Method 2) 802.1x authenticates the device -> host pool.
* Host pool has a SGT, SVI and VRF instance.
* SVI is the anycast gateway (same IP address and MAC for that SVI & VRF) on all edge nodes.
* Host address is now an EID (MAC, /32 IPv4, /128 IPv6), that can be registered with the control plane node.
* Control plane signaling is LISP, dataplane is managed via VXLAN-GPO.

## Fabric Border Nodes Types
* **Internal Border:** WLC, Firewall, Data center

* **Default Border:** Internet.

* **Internal + Default:** Both.

## Wireless
If the WLC can participate in the fabric, it's a fabric aware WLC. It performs PxTR (proxy lisp encap/de-encap) for hosts connected to fabric APs, and registers their EIDs with the control nodes.

Control plane traffic is CAPWAP inside of VXLAN-GPO. Dataplane traffic can just ride VXLAN-GPO

## LISP
* The LISP instance ID is the VRF.

## Cisco Catalyst Center
* **NCP:**Network Control Platform. This module is connect via API to the GUI, and is what talks to the network gear via NETCONF, SNMP, or SSH. Does all the underlay automation.

* **NDA:** Network Data Platform. Data collection and analytics. Netflow, Syslog, ERSPAN, etc.

* **ISE:** Is required. 802.1x, Mac Authentication Bypass (MAB), or Web Authentication (WebAuth). Can talk to AWS or Active Directory. ISE is *tightly* integrated via API calls to CatC.
