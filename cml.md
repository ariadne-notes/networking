# Setting IP Routes
```
hostname pc-20
ip link set dev eth0 up
ip address add 10.0.20.20/24 dev eth0
ip route add default via 10.0.20.1
```

# iperf
**Server**
`iperf --port 2000 --server`

**Client**
`iperf --port 2000 --client 10.0.0.1 --num 10k --reverse --udp`
