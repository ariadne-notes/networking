# Terms

* **1 second**, is 1000 ms.
* **1 millisecond:** Network latency is measured in ms, or 1 thousandth of a second 0.001.
* **1 microsecond:** 1 μs (a millionth) of a second. 0.000 001. 1000 μs is 1 ms.
* **1 nanosecond:** 1 ns (a billionth) of a second. 0.000 000 001. 1000 ns is 1 μs.
* **NTP:** An older time standard. Can sync time between 10 to 1 ms.
* **PTP:** Modern time standard. Can sync time between 10 to 1 ns.
* **PTPv1:** - Defined in IEEE 1588-2002
* **PTPv2:** - Defined in IEEE 1588-2008, not backwards compatible.
* **PTPv2.1:** - Defined in IEEE 1588-2019, is backward compatible.
* **1588 Clock:** A clock in the PTP time domain. Clocks have ports.
* **Terminating Clock:** A clock with one port.
* **Ordinary Clock:** a clock in a terminating device.
* **Boundary Clock::** a clock in a transmitting device, like an ethernet switch. Connects PTP domains.
* **Transparent Clock:** a boundary clock that can correct for delay and modifies the PTP event message.
* **Grandmaster:** All clocks sync to this one clock.
* **Master:** All clocks in a subdomain sync to the master. The master sync's to the grand master.

## Time Terms
* **Epoch:** The start of time.
* **Offset:** The estimated time between a master clock sending time, and a slave clock receiving it.


# Uses
- Robotics, synchronizing movements.
- Mobile Phone networks, telemetry, billing, logging
- Financial Networks, trade settling fairness.
- Power Networks, to sync to the 60hz grid.
- Science network, seismic data

# Process
After PTP has time from something like a GPS device, it can pass that time along, so long as the devices in the path can mark and read the timestamps

![PTP Delay and Offset Calculations](./images/ptp-algo-riedel-communications.jpg)

# General Messages

* **Announce:** Used to determine which Grand Master is selected Best Master

* **Follow_Up:** Used to convey a captured timestamp of a transmitted SYNC message

* **Delay_Response:** Used to measure delay between IEEE 1588 devices

* **Pdelay_Response_Follow_Up:** Used between IEEE 1588 devices to measure the delay on an incoming link

* **Management:** Used between management devices and clocks

* **Signaling:** Used by clocks to deliver how messages are sent

# Event Messages
* **Sync:** Used to convey time

* **Delay_Request:** Used to measure delay from downstream devices

* **Pdelay_Request:** Used to initiate and measure delay

* **Pdelay_Response:** Used to respond and measure delay