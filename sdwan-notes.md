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


