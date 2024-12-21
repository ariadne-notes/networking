# Lightweight Modes

## Client-Serving AP Modes

    Local: This is the default mode. A local mode AP tunnels all client traffic, for all WLANs, in CAPWAP, to the controller. In this mode, the AP’s radios are operational only when the AP is connected to its controller. Local mode APs do not support mesh operation. All AP models support Local mode.

    FlexConnect: In this mode, client traffic can either be tunneled in CAPWAP to the controller, or egress at the AP’s LAN port, depending on the WLAN configuration. FlexConnect mode APs do not support mesh operation. All models support FlexConnect mode.

    Bridge and Flex+Bridge: These modes are used in mesh deployments, where wireless rather than wired backhaul is used for CAPWAP connectivity. Not all AP models support these modes; see the relevant mesh documentation for information about support for mesh operation.

## Network Management AP Modes

    Monitor: In this mode, the AP radios are dedicated to monitoring the Wi-Fi channel for RRM and rogue detection. All AP models support this mode.

    Rogue Detector: In this mode, the AP radios are disabled; the AP monitors the LAN to detect on-wire rogue activity. This mode is not supported on Cisco Wave 2 or 802.11ax APs and is deprecated.

    Sniffer: In this mode, the AP radio operates in promiscuous mode and captures all Wi-Fi traffic on a channel. These packets are tunneled in CAPWAP to the controller, which forwards them to a machine running OmniPeek or Wireshark for storage and analysis.

    SE-Connect: In this mode, the AP provides a dedicated connection to CleanAir for spectrum analysis by software such as Spectrum Expert or Chanalyzer. SE-Connect mode is supported only on SE models with CleanAir.
    
[Source](https://www.cisco.com/c/en/us/td/docs/wireless/controller/8-10/config-guide/b_cg810/managing_aps.html)