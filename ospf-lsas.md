Type 1 and Type 2 describe what's inside an area.

Type 1 - Here are my links.

Type 2 - Here is my attached network.



# Type 1 - Router 

**DR**

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

**DROther**

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

**DR Describing the network


# Type 2 - Network

```
R4# show ip ospf database network 

            OSPF Router with ID (4.4.4.4) (Process ID 1)

                Net Link States (Area 0)

  LS age: 183
  Options: (No TOS-capability, DC)
  LS Type: Network Links
  Link State ID: 10.0.0.1 (address of Designated Router)
  Advertising Router: 1.1.1.1
  LS Seq Number: 80000002
  Checksum: 0x4481
  Length: 48
  Network Mask: /24
        Attached Router: 1.1.1.1
        Attached Router: 2.2.2.2
        Attached Router: 3.3.3.3
        Attached Router: 4.4.4.4
        Attached Router: 5.5.5.5
        Attached Router: 6.6.6.6
```