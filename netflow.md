Netflow
v5 - v4 flows only
v9 - template based
IPFIX

## Flexible Netflow

Netflow needs four things to work:

* Records
* Exporters
* Monitors
* Interfaces

## Record

flow record FLOW_RECORD_IPV4
 match ipv4 protocol
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 match interface input
 collect interface output
 collect counter bytes long
 collect counter packets long
 collect timestamp sys-uptime first
 collect timestamp sys-uptime last

## Exporter

flow exporter FLOW_EXPORTER
 !
 ! IPFix is standards based netflow.
 !
 export-protocol ipfix
 destination 10.0.52.100
 source GigabitEthernet2
 transport udp 2055
 template data timeout 60
!

## Monitor

flow monitor FLOW_MONITOR_IPV4
 exporter FLOW_EXPORTER
 cache timeout active 60
 record FLOW_RECORD_IPV4


## Interfaces
 
interface GigabitEthernet1
 ip flow monitor FLOW_MONITOR_IPV4 input
 ip flow monitor FLOW_MONITOR_IPV4 output
 

### Lab validations
```
R1# show flow monitor FLOW_MONITOR_IPV4 statistics 
  Cache type:                               Normal (Platform cache)
  Cache size:                               200000
  Current entries:                               4
  High Watermark:                                4

  Flows added:                                   8
  Flows aged:                                    4
    - Active timeout      (    60 secs)          4


R1# show flow monitor FLOW_MONITOR_IPV4 cache sort highest counter bytes long top 10 format table
Processed 3 flows
Aggregated to 3 flows
Showing the top 3 flows

IPV4 SRC ADDR    IPV4 DST ADDR    TRNS SRC PORT  TRNS DST PORT  INTF INPUT            IP PROT  intf output                     bytes long             pkts long    time first     time last
===============  ===============  =============  =============  ====================  =======  ====================  ====================  ====================  ============  ============
10.0.10.101      10.0.20.101              48640           5000  Gi4                        17  Gi1                                 334100                   325  20:37:12.210  20:37:44.424
10.0.12.2        224.0.0.5                    0              0  Gi1                        89  Null                                   600                     6  20:36:54.026  20:37:41.568
10.0.12.1        224.0.0.5                    0              0  Null                       89  Gi1                                    600                     6  20:36:52.808  20:37:38.836
```

