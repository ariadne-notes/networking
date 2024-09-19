# Terms

* **STDM** - Synchronous Time-Division Multiplexing
* **DS0** - Level 0. One timeslot. A timeslot carries 8 bits. Frame rate is 8000 hz. 8 * 8000 = 64Kbps.
* **B8ZS** - Binary Eight Zero Substitution. A special way to encode `0000 0000` for DS0 lines.
* **T1 Frame** - T-Carrier, Level 1. Aggregates 24 DS0 frames, or 192 bits. The T1 gets an extra bit, for framing so 193. 193 * 8000 is 1.544 Mbps.
* **Super Frame** - 12 T1 frames.
* **Framing Search** - Each T1 frame uses the extra bit to encode part of the superframe bit pattern `0101 1101 0001` or (5, 13, 1).
* **APS** - Automatic Protection Switching. The device engaging in APS sends the data on *both* links, the working link and the protected link. The recieving device devices which to use.
* **DS1** - Data Stream, Level 1.
* **T1** - T-Carrier, Level 1, Carries 24 DS0 frames, or 192 bits. The T1 gets an extra bit, for framing so 193. 193 * 8000 is 1.544 Mbps.
* **ACR** - Access Circuit Redundancy

The common STDM system in the US is T-Carrier.

# Cisco CEM Terms

* **ACR** - Adaptive Clock Recovery, A technique to recovery the clock based on the fill level of the jitter buffer.

## Access Circuit Redundancy

# References

[T-Carrier and SONET](https://pld.cs.luc.edu/telecom/mnotes/tcarrier_sonet.html)

[All you Wanted to Know about T1 But Were afraid to Ask](https://www.dcbnet.com/notes/9611t1.html)

[OCx CEM Interface Module Config Guide IOS-XE 17 ASR 900 Series](https://www.cisco.com/c/en/us/td/docs/routers/asr903/software/guide/cem/17-1-1/b-cem-ocx-xe-asr900/m-configuring-aps-cem-ocx.html)