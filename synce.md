SyncE synchronizes clock frequency over an Ethernet port. It does not synchronize time-of-day, that's done by PTP, IEEE 1588.

Setting as oscillator to a frequency is syntonization.

## References

**ITU-T Rec. G.8261** - Architecture and the wander performance of SyncE networks

**ITU-T Rec. G.8262** - Synchronous Ethernet clocks for SyncE

**ITU-T Rec. G.8264** - Ethernet Synchronization Messaging Channel (ESMC)

### Config Options

##### ITU-T G.813 Option 1 clock (QL-SEC)
`EEC-option 1`


##### ITU-T G.812 type IV clock (QL-ST3)		      
`EEC-option 2`

## Terms

Synchronous Ethernet and IEEE 1588 in Telecoms

* **Time Interval:** Distance between two events, (measured in seconds), milliseconds, microseconds, nanoseconds, picoseconds

* **Frequency:** Rate of a repetitive event. Measured in cyles per second. A device that produces frequency is an oscilator.

* **T0:** System Clock (line interface output)
	
* **T1:** Timing Reference signal derived from STM-N (STS-N/SyncE) input.
	 
* **T2:** Timing Reference signal derived from 2048/1544 kbit input [input from PDH]
	
* **T3:** Timing reference signal derived from 2048 or 2048 1544 with SSM.
	 
* **T4:** Clock-interface output.

* **OSC:** Internal ST3 oscillator

* **SSM:**  Synchronization Status Message

* **ESMC:** Ethernet Synchronization Message Channel

* **MTIE:** Maximum time interval error is a measure of the worst case phase variation of a signal with respect to a perfect signal over a given period of time.

* **TDEV:** Time deviation is a statistical analysis of the phase stability of a signal over a given period of time.




