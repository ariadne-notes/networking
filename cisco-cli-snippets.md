### Supressing logging

> %CDP-4-DUPLEX_MISMATCH: duplex mismatch discovered on FastEthernet0/0 (not half duplex), with eve-R24 FastEthernet0/0 (half duplex).

```
configure terminal
logging discriminator duplex_msg facility includes CDP msg-body includes "duplex mismatch"
logging buffered discriminator duplex_msg
```
