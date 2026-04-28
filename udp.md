## UDP Packet Format

[User Datagram Protocol - RFC 768](https://www.rfc-editor.org/rfc/rfc768)

UDP does try to send error-free packets by including a checksum, the below via the RFC

> Checksum is the 16-bit one's complement of the one's complement sum of a
> pseudo header of information from the IP header, the UDP header, and the
> data,  padded  with zero octets  at the end (if  necessary)  to  make  a
> multiple of two octets.

...

> If the computed  checksum  is zero,  it is transmitted  as all ones (the
> equivalent  in one's complement  arithmetic).   An all zero  transmitted
> checksum  value means that the transmitter  generated  no checksum  (for
> debugging or for higher level protocols that don't care).

```
 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8
┌────────────────────────────────┬───────────────────────────────┐
│          Source Port           │       Destination Port        │
├────────────────────────────────┼───────────────────────────────┤
│            Length              │           Checksum            │
├────────────────────────────────┴───────────────────────────────┘
│          Data Octets
└────────────────────────────────►
```

##### TFTP Read Request

```
Frame 115: 69 bytes on wire (552 bits), 69 bytes captured (552 bits) on interface -, id 0
    Internet Protocol Version 4, Src: 10.0.10.22, Dst: 10.0.10.33
    User Datagram Protocol, Src Port: 52775, Dst Port: 69
        Source Port: 52775
        Destination Port: 69
        Length: 31
        Checksum: 0x4aed [correct]
        [Checksum Status: Good]
        [Stream index: 0]
        [Timestamps]
        UDP payload (23 bytes)
    Trivial File Transfer Protocol
        Opcode: Read Request (1)
        Source File: startup-config
        Type: octet
```
##### TFTP Data Packet

```
Frame 116: 562 bytes on wire (4496 bits), 562 bytes captured (4496 bits) on interface
    Internet Protocol Version 4, Src: 10.0.10.33, Dst: 10.0.10.22
    User Datagram Protocol, Src Port: 52590, Dst Port: 52775
        Source Port: 52590
        Destination Port: 52775
        Length: 524
        Checksum: 0xde83 [correct]
        [Checksum Status: Good]
        [Stream index: 1]
        [Timestamps]
        UDP payload (516 bytes)
    Trivial File Transfer Protocol
        Opcode: Data Packet (3)
        [Destination File: startup-config]
        [Read Request in frame 115]
        Block: 1
        [Full Block Number: 1]
    Data (512 bytes)
    
    0000  0a 21 0a 21 20 4c 61 73 74 20 63 6f 6e 66 69 67   .!.! Last config
    0010  75 72 61 74 69 6f 6e 20 63 68 61 6e 67 65 20 61   uration change a
    0020  74 20 30 35 3a 31 31 3a 31 35 20 55 54 43 20 53   t 05:11:15 UTC S
    0030  61 74 20 4a 75 6c 20 38 20 32 30 32 33 0a 21 0a   at Jul 8 2023.!.
    0040  76 65 72 73 69 6f 6e 20 31 35 2e 32 0a 73 65 72   version 15.2.ser
    0050  76 69 63 65 20 74 69 6d 65 73 74 61 6d 70 73 20   vice timestamps 
    0060  64 65 62 75 67 20 64 61 74 65 74 69 6d 65 20 6d   debug datetime m
    0070  73 65 63 0a 73 65 72 76 69 63 65 20 74 69 6d 65   sec.service time
    0080  73 74 61 6d 70 73 20 6c 6f 67 20 64 61 74 65 74   stamps log datet
    0090  69 6d 65 20 6d 73 65 63 0a 6e 6f 20 73 65 72 76   ime msec.no serv
    00a0  69 63 65 20 70 61 73 73 77 6f 72 64 2d 65 6e 63   ice password-enc
    00b0  72 79 70 74 69 6f 6e 0a 73 65 72 76 69 63 65 20   ryption.service 
    00c0  63 6f 6d 70 72 65 73 73 2d 63 6f 6e 66 69 67 0a   compress-config.
    00d0  21 0a 68 6f 73 74 6e 61 6d 65 20 53 57 33 0a 21   !.hostname SW3.!
    00e0  0a 62 6f 6f 74 2d 73 74 61 72 74 2d 6d 61 72 6b   .boot-start-mark
    00f0  65 72 0a 62 6f 6f 74 2d 65 6e 64 2d 6d 61 72 6b   er.boot-end-mark
    0100  65 72 0a 21 0a 21 0a 6c 6f 67 67 69 6e 67 20 64   er.!.!.logging d
    0110  69 73 63 72 69 6d 69 6e 61 74 6f 72 20 45 58 43   iscriminator EXC
    0120  45 53 53 20 73 65 76 65 72 69 74 79 20 64 72 6f   ESS severity dro
    0130  70 73 20 36 20 6d 73 67 2d 62 6f 64 79 20 64 72   ps 6 msg-body dr
    0140  6f 70 73 20 45 58 43 45 53 53 43 4f 4c 4c 20 0a   ops EXCESSCOLL .
    0150  6c 6f 67 67 69 6e 67 20 62 75 66 66 65 72 65 64   logging buffered
    0160  20 35 30 30 30 30 0a 6c 6f 67 67 69 6e 67 20 63    50000.logging c
    0170  6f 6e 73 6f 6c 65 20 64 69 73 63 72 69 6d 69 6e   onsole discrimin
    0180  61 74 6f 72 20 45 58 43 45 53 53 0a 21 0a 6e 6f   ator EXCESS.!.no
    0190  20 61 61 61 20 6e 65 77 2d 6d 6f 64 65 6c 0a 21    aaa new-model.!
    01a0  0a 21 0a 21 0a 21 0a 21 0a 6e 6f 20 69 70 20 69   .!.!.!.!.no ip i
    01b0  63 6d 70 20 72 61 74 65 2d 6c 69 6d 69 74 20 75   cmp rate-limit u
    01c0  6e 72 65 61 63 68 61 62 6c 65 0a 21 0a 21 0a 21   nreachable.!.!.!
    01d0  0a 6e 6f 20 69 70 20 64 6f 6d 61 69 6e 2d 6c 6f   .no ip domain-lo
    01e0  6f 6b 75 70 0a 69 70 20 63 65 66 0a 6e 6f 20 69   okup.ip cef.no i
    01f0  70 76 36 20 63 65 66 0a 21 0a 21 0a 21 0a 73 70   pv6 cef.!.!.!.sp
```

