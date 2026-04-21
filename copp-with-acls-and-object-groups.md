# CoPP Configuration.

This was performed on an C8000v, running 17.13.1a

1. A simple ACL that matches based on ICMP.

```
ip access-list extended ACL_ICMP_UNKNOWN
 permit icmp any any
```

1. Make class-map to use the ACL.
```
class-map CLASS_MAP_ICMP_UNKNOWN
 match access-group name ACL_ICMP_UNKNOWN
```

1. Make a policy map that uses the above class-maps

```
policy-map POLICY_MAP_COPP
 class CLASS_MAP_ICMP_UNKNOWN
  police cir 10000 conform-action transmit  exceed-action drop
 class class-default
```

1. Apply it to the control plane.
```
control-plane
 service-policy input COPP-POLICY-MAP
```

1. Validate

```
router# show policy-map control-plane input 
 Control Plane 

  Service-policy input: POLICY_MAP_COPP

    Class-map: CLASS_MAP_RFC1918 (match-all)  
      0 packets, 0 bytes
      5 minute offered rate 0000 bps
      Match: access-group name ACL_RFC1918

    Class-map: CLASS_MAP_ICMP_UNKNOWN (match-all)  
      0 packets, 0 bytes
      5 minute offered rate 0000 bps, drop rate 0000 bps
      Match: access-group name ACL_ICMP_UNKNOWN
      police:
          cir 1000000 bps, bc 31250 bytes
        conformed 0 packets, 0 bytes; actions:
          transmit 
        exceeded 0 packets, 0 bytes; actions:
          drop 
        conformed 0000 bps, exceeded 0000 bps

    Class-map: class-default (match-any)  
      0 packets, 0 bytes
      5 minute offered rate 0000 bps, drop rate 0000 bps
      Match: any
```

# Test Setup
This uses python3, scapy, and sendpfast, to send icmp packets with random sources.

1. Install sendpfast
```
sudo apt install tcpreplay
```

1. Start a python virtual environment.
```
python3 -m venv venv
source venv/bin/activate
```

1. Install scapy inside it.
```
pip install scapy
```

1. Modify then paste in the following python script.

`dst`
`iface`

```
cat > flood.py << 'EOF'
from scapy.all import *
import random

def random_public_ip():
    while True:
        ip = f"{random.randint(1,223)}.{random.randint(0,255)}.{random.randint(0,255)}.{random.randint(1,254)}"
        if not (ip.startswith("10.") or 
                ip.startswith("192.168.") or 
                ip.startswith("172.") and 16 <= int(ip.split(".")[1]) <= 31):
            return ip

pkts = [Ether()/IP(src=random_public_ip(), dst="192.168.52.198")/ICMP() for _ in range(1000)]
sendpfast(pkts, pps=10000, loop=100, iface="ens18")
EOF
```

1. In a different terminal run something like this to see the packets leaving the interface.
```
sudo tcpdump -i ens18 icmp -n
```

1. This requires raw sockets to run.
```
sudo venv/bin/python3 flood.py
```
