# Modulation
Modulation can only do a few things to a carrier signal to include data
* Frequency (a tiny bit faster or slower)
* Phase (changing the start and stop intervals)
* Amplitude (signal strength)

# Wireless Terms

* **dBi** - An *isotropic* antenna. Basically, a single point in space, mathematically perfect.

* **EIRP** - effective isotropic radiated power.

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




 
