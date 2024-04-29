# Modulation
Modulation can only do a few things to a carrier signal to include data
* Frequency (a tiny bit faster or slower)
* Phase (changing the start and stop intervals)
* Amplitude (signal strength)

# Wireless Terms

* **BSS** - Basic Service Set. A wireless network segment, a star network with the AP as the center. The hosts cannot talk directly to each other.

* **Association Request** - Joining an AP.

* **Reassociation Requests** - Roaming from one AP to another.

* **dBi** - An *isotropic* antenna. Basically, a single point in space, mathematically perfect.

* **EIRP** - effective isotropic radiated power. Also used by regulatory agencies.

* **Link Budget** - How much power is needed in total to go from sending a message to being understood.

* **Free Space Loss** - Energy lost to signal propagation through the air. Faster frequencies are more effected.

* **RSSI** - Received Signal Strength Indicator. It's an 802.11 standard, an internal 1 byte value from 0 to 255, 255 being the strongest. Not super useful, it's not standardized.

* **Noise Floor** - Average signal strength of the noise.

* **SNR** - Signal to Noise Floor. The difference between a signal and the noise.

* **Carrier Signal** - The thing that must be present for a signal to be present.

* **Modulation** - What is done to the carrier signal to put data into it.

* **Demodulation** - How a carrier signal is read to see data.

* **Narrowband** - Modulation doesn't take a lot of space. Like CW, or PSK.

* **Spread Spectrum** - There is a lot of modulation.

* **DSSS** - Direct Sequence Spread Spectrum. Used in the 2.4 band. Complex phase modulation.

* **OFDM** - Orthogonal Frequency Division Multiplexing. 2.4 and 5 ghz band. A 20mhz channel is sending data in parallel over multiple frequencies, with subcarriers. Phase and Amplitude are modulated, in a method called QAM.

* **QAM** - Quadrature Amplitude Modulation.

* **802.11** - The wireless standard from 1997.

* **802.11n** -- 2009, it brought the HT.

* **HT** -- High Throughput

* **802.11ac** -- 2013, VHT. Lots to configure, about 320 different data-rates.

* **VHT** - Very High Throughput.

* **802.11ax** -- WiFi 6. High-effiency wireless. Multiple devices can transmit at the same time. Uses OFDMA.

* **SISO** - Single In, Single Out. One radio.

* **MIMO** - Multiple In, Multiple Out. Multiple Radios.

* **Radio Chain** - A pathway to either send or recieve a signal. A MIMO device has multiple radio chains.

2x2, 2 Tx, 2 Rx

2x3, 2 Tx, 3 Rx

* **Spatial Multiplexing** - Uses digital signal processing to demodulate a signal recieved on multiple radio chains. Each radio chain will hear a different signal and the known phase difference can be demodulated.

3x3:2, 3 Tx, 3 Rx, 2 Spatial Streams

* **Transmit Beamforming** - Knowing a signal will arrive out-of-phase if sent on multiple antennas to the same reciever, a MIMO device can intentionally send a signal on multiple radio chains, out of phase, so when it arrives the composite signal is construcitvely in-phase.

* **TxBF** - Transmit Beamforming. The feedback sent from the reciever to the transmitter on how to better phase-shift the transmission.

* **MRC** - Maximal Ratio Combining - Combining copies of signals on multiple antennas to get better SNR.

* **DRS** - Dynamic Rate Shifting. The both stations agree ahead of time on how to speed up or slow down their datarate based on the changing RF environment. Also known as link adaptation, adaptive modulation and coding, rate adaptation.

* **AMC** - Adaptive Modulation and Coding, the same as DRS.



# Table 17-4 from the ENCOR 350-401 text

| Standard | 2.4Ghz?  | 5ghz?  | Data Rates Supported                                | Channel Width Supported       |
|----------|----------|--------|-----------------------------------------------------|-------------------------------|
| 802.11b  | Yes      | No     | 1, 2, 5.5, and 11 Mbps                              | 22 MHz                        |
| 802.11g  | Yes      | No     | 6, 9, 12, 18, 24, 36, 48, and 54 Mbps               | 22 MHz                        |
| 802.11a  | No       | Yes    | 6, 9, 12, 18, 24, 36, 48, and 54 Mbps               | 20 MHz                        |
| 802.11n  | Yes      | Yes    | Up to 150 Mbps* per spatial stream, up to 4 spatial streams | 20 or 40 MHz          |
| 802.11ac | No       | Yes    | Up to 866 Mbps per spatial stream, up to 4 spatial streams | 20, 40, 80, or 160 MHz |
| 802.11ax | Yes*     | Yes*   | Up to 1.2 Gbps per spatial stream, up to 8 spatial streams | 20, 40, 80, or 160 MHz |

`*` 802.11ax is designed to work on any band from 1 to 7 GHz, provided that the band is approved for use.

# AP States

**AP boot:** The AP starts a bootloader OS. It starts DHCP, both to get it's own address and the address of the controller to download the config and OS image. 

**WLC discovery:** The AP looks for a WLC.

**CAPWAP tunnel:** The AP builds a CAPWAP tunnel to a WLC after exchanging certificates. Traffic is DTLS (Datagram Transport Layer Security).

**WLC join:** The AP selects a WLC from a list of candidates and then sends a CAPWAP Join Request message to it. The WLC replies with a CAPWAP Join Response message. The next section explains how an AP selects a WLC to join.

**Download image:** The WLC informs the AP of its software release. If the AP’s own software is a different release, the AP downloads a matching image from the controller, reboots to apply the new image, and then returns to step 1. If the two are running identical releases, no download is needed.

**Download config:** The AP pulls configuration parameters down from the WLC and can update existing values with those sent from the controller. Settings include RF, service set identifier (SSID), security, and quality of service (QoS) parameters.

**Run:** Once the AP is fully initialized, the WLC places it in the “run” state. The AP and WLC then begin providing a BSS and begin accepting wireless clients.

**Reset:** If an AP is reset by the WLC, it tears down existing client associations and any CAPWAP tunnels to WLCs. The AP then reboots and starts through the entire state machine again.

## WLC Discovery

#### (IOS Config)

When APs boot, they will send a request to the controller IP, or if they don't have one, will broadcast a request to port 5246. Here is a config to forward the request to one or more IPs.

```
router(config)# ip forward-protocol udp 5246
router(config)# interface vlan <number>
router(config-int)# ip helper-address WLC1-MGMT-ADDR
router(config-int)# ip helper-address WLC2-MGMT-ADDR
```

#### Priming
The APs themselves keep lists of known controllers

#### DHCP
Option 43 can contain a controller address.

#### DNScisco-capwap-controller
The AP does an A record lookup for
`cisco-capwap-controller.local-domain`
To get an address.

## WLC Joining
1. Previously configured
1. Discovery
1. Least loaded WLC (the controllers report on their load)

## AP Priority
A WLC can only support so many APs. APs have priority like low, medium, high, and critical. The default is low.
The WLC will disconnect a lower priority AP for a higher priority AP.

## WLC keepalive
APs send heartbeat messages every 30 seconds to WLCs to make sure they are alive. The WLC must respond. If the WLC doesn't respond the AP sends 4 more keepalives at 3 second intervals.

APs can detect WLC failures in about 35 seconds.

## High Availability (HA)
WLCs can be configured to do statefull switchover (SSO) with an active and hot-standby. There isn't a need to configure a secondary WLC then.
Failures are transparent with HA.

## Cisco AP Modes

### Cisco AP Operational Modes

**Local**: Operates in default lightweight mode with active BSSs on a designated channel. Scans other channels for noise, interference, rogue device detection, and IDS event matching when not transmitting.

**Monitor**: Functions as a dedicated sensor without transmitting. It enables receiver only to check IDS events, detect rogue APs, and support location-based services by determining station positions.

**FlexConnect**: Allows an AP at a remote site to locally switch traffic between an SSID and a VLAN during CAPWAP tunnel outages, given prior configuration.

**Sniffer**: Dedicates its radios to capture 802.11 traffic for analysis. Captured data is forwarded to network analysis software like LiveAction Omnipeek or Wireshark.

**Rogue Detector**: Focuses on identifying rogue devices by correlating MAC addresses observed on both wired and wireless networks.

**Bridge**: Serves as a dedicated bridge (either point-to-point or point-to-multipoint) to connect networks. Utilized for linking distant locations or forming mesh networks.

**Flex+Bridge**: Combines FlexConnect with bridge capabilities in a mesh AP configuration.

**SE-Connect**: The AP dedicates its radios to perform spectrum analysis across all wireless channels. Allows for remote connection of a PC running analysis software like MetaGeek Chanalyzer or Cisco Spectrum Expert. This setup is used to collect and analyze spectrum data to identify sources of interference.

## Radiation Patterns

**H Plane** - Horizontal, or XY, or Azimuth. The top-down view, like in a video game.

**E Plane** - Elevation, or XZ. The side view on a 2D video game.

**Omnidirectional** - +4dBi, a donut.

**Directional** - +12dBi, a spear, also called a lobe.

**Beamwidth** - Looking at the radiation pattern, how much it can spread before being halfed. A 30 degree beam is 0dBm on axis, and -3dBm 15% off axis.

## Polarization

A vertical antenna sends vertically polarized signals, with the electrical field going up and down. It works best when recieved on another vertically polarized antenna.

## Roaming

#### Autonomous
The APs keep track of what clients are connected.

#### Intracontroller
A client goes from AP-to-AP but those APs are on the same controller.


10ms - The controller handles the roaming process.

#### Client Authentication
If the controller needs to perform 802.1x on each client, this process should be streamlined to prevent long roam times.
* **CCKM** - Cisco Centralized Key Management. One controller maintains a database of clients and keys on behalf of APs and provides them to other clients and APs during client roams. Clients need to support CCX (Cisco Compatible Extensions)

* **Key Caching** - The client keeps keys for eight APs, and the destination AP must be in this list.

* **802.11r** - Fast roaming or fast BSS transition. The client caches a portion of the authentication server's key, presenting that to future APs. The client can also cache it's QoS parameters.

Each of these strategies requires the client to support it. (supplicant or driver software)

#### Intercontroller 

Layer2 - 20ms - Local-to-Local - Two controllers need to agree to also move the client's IP address from one controller to another. Fast, because both controllers are on the same vlan, and the client keeps it's address.

Layer3 - Local-to-Foreign - The client has an anchor controller (where it started) and a foreign controller (for the BSS it's actually on), if they aren't on the same subnet, the controllers build a CAPWAP tunnle between them, so the client's data appears on the correct subnet. Uses two CAPWAP tunnels.

#### Mobility Groups

Mobility groups speed up client roaming, by caching credentials. Clients can roam between mobility groups, but must re-authenticate.

## Locating Devices

* **RTLS** - Real Time Location Services

#### Probe Requests
Wifi devices don't need to authenticate to send probe requests. They just need to be turned on. Even RFID tags can send probe requests.




