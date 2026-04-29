# From the DR
```
R1# show ip ospf database router 1.1.1.1

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

  LS age: 32
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 1.1.1.1
  Advertising Router: 1.1.1.1
  LS Seq Number: 8000007B
  Checksum: 0x1A77
  Length: 36
  Number of Links: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 10.0.0.1
     (Link Data) Router Interface address: 10.0.0.1
      Number of MTID metrics: 0
       TOS 0 Metrics: 10
```

# From a DROther

```
R4#show ip ospf database router 4.4.4.4

            OSPF Router with ID (4.4.4.4) (Process ID 1)

                Router Link States (Area 0)

  LS age: 135
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 4.4.4.4
  Advertising Router: 4.4.4.4
  LS Seq Number: 8000007C
  Checksum: 0x5D18
  Length: 36
  Number of Links: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 10.0.0.1
     (Link Data) Router Interface address: 10.0.0.4
      Number of MTID metrics: 0
       TOS 0 Metrics: 10
```

The 