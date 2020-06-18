This project is an attempt to reverse engineer the RS 485 protocol of Valence U1-12RT LiFePo4 batteries. There are some
projects and tools for the Valence XP series BMS, but these don't work with the RT-12 batteries. At least not with the
batteries that I own. So I looked at the communication between the 12RT batteries and the genuine SOC display and tried to make
some sense out of it. This is work in progress. There is probably more information in the data provided by the battery
 and some of the identified data points could be wrong. 

The provided Python script will try to get a data message from every battery on the bus and display the information that
is contains. As far as it has been identified. It also outputs all messages on the bus.

This is not intended to be an end user tool or library, but it should be easy to modify the output to your needs or
extract/port the protocol for your own needs.     

###Usage
The script needs Python 3, it accepts one parameter that is used as COM port (eg COM9). Without it a default port will 
be used, and probably not the right one. Example:

`python ValRTRX.py COM9`

**Important:** The RS 485 interface of the RT-12 battery is only active if the battery is charged or discharged. At least
with the two batteries I have. This is probably a feature to disable the genuine external display to save power, but not good
for other uses. Connecting a 5V supply voltage to the interface doesn't activate it. Connecting a 100 Ohm 2W resistor to
the battery reliably activates it, but also drains a lot of power (almost 2W). Probably a higher resistance will also 
work, but I haven't tested that.

### Physical connection
This is for the 5 pin AMP-Superseal connector. Pin 3 is RS485 A and pin 4 RS485 B on the connector (pin 1 can be identified by the key in the connector edge between 1 and 2). 

2 is ground and 5 the supply voltage. But there is no need to connect this.      

### Protocol description
The baudrate is 115200, no parity, one stopbit, little endian encoding. The messages end with a 2 byte modbus CRC.

The communication is started with the message "ff4306064246ff4306064246". Then there is a polling loop for battery IDs
 with the messages "ff50060" + X + CRC and X from 0 to 5. The batteries answer with a message like "ff701143424231313238313033313850b9" that 
contains
some kind of battery ID. Then a bus ID is assigned to the battery ID with a message like "ff4306064246ff4306064246".
 There is no response to this assignments.

After 2-3 battery ID polling loops the display sends a data poll request for battery every battery ID it has assigned
  before. This requests look like "02030029001bd43a". This is responded by message like "020336171515150f000004a8005004151715181815800084000000840000000b0d090d2834000028340000eeee001004000a0d090d0a0d0b0deae4".
  
These data messages seem to have a fixed format and contain information at fixed positions. For details please check the code.    

### Licence
MIT

Use it, improve it and make the world a better place.

### Todo and remarks
* Why are there 6 numbered voltages? I don't know. These seem to be voltages of the four serial cell groups. The 
numbers fit. But then there should only be four. In fact there are even more bytes with numbers that could be voltages, 
but they seem to be duplicates of these six. Possible U5 is the same as U4 and U6 is U1. They are very close, but not 
always the same, so maybe they measure the same group, but use separate ADC readings. Hard to tell without opening the battery.
* I2: This seems to be the battery current as I1, but while charging it is sometimes lower than I1 and the actual charging current. This might be caused by the balancer (temps should go up then, but I haven't checked).    
* Undiscovered data points: At least the fault codes are missing. If anyone can identify them or anything else in the 
data message I will gladly add it. 
* When the battery is idle the RX led on my RS485 connector flashes roughly every 16 seconds when it is not connected to 
USB. I haven't found out why yet.     