<!--- Copyright (c) 2014 Spence Konde. See the file LICENSE for copying permission. -->
MCP4xxx I2C digital potentiometers
======================

* KEYWORDS: Module,I2C,digipot,digital potentiometer,ADC,rheostat

Overview
------------------

A digital potentiometer is a potentiometer that can be controlled via digital means (typically I2C or SPI). They consist of a "resistor ladder" of many identical value resistors between two pins (the ends of the potentiometer), while a third pin (the wiper) can be connected between any of those resistors. The number of taps (points between resistors that the wiper can be connected to) varies, but is typically a power of two, or 1 more than a power of two. Some digital potentiometers have non-volatile memory and use that to initialize the wiper position on startup. Digital pots are available in the same resistance ranges as analog pots. Digital rheostats are also available. Note that in most digital potentiometers, the voltage on any of the potentiometer terminals must always be between ground and supply voltage (sometimes with allowance for voltages slightly beyond that), though there do exist digital potentiometers capable of handling much higher voltages on the analog pins. Digital potentiometers generally cannot handle very much current - see the datasheet for specifics. 

Supported Parts
------------------

This module [[MCP4xxxI2C.js]] interfaces with a wide variety of I2C digital potentiometers and rheostats made by Microchip. Supported devices have either 129 or 257 taps, 1, 2, or 4 potentiometers (or rheostats), and may or may not have non-volitile memory.  Many of these devices have multiple pins available to set the I2C address. This module does not work with MCP401x devices. 

Supported parts will have part numbers of the form MCP4abc
a - Indicates number of pots. 4 -> 4 pots, 5 -> 1 pot, 6 -> 2 pots. 
b - Indicates number of taps and whether it has non-volitile memory. 3 -> 129 taps volatile, 4-> 129 taps non-volatile. 5 -> 257 taps volatile, 6-> 257 taps non-volatile. 
c - 1 indicates a potentiometer, 2 indicates a rheostat. 

Wiring
------------------

Unfortunately, the parts supported by this module are available only in inconvenient packages: QFN (which is impractical to use at home) and TSSOP, which is a finer pitch than the Espruino SMD prototyping area. Generic breakout boards for TSSOP packages are available from Adafruit and many other hobby electronics suppliers. Soldering using the "drag soldering" technique is recommended. 

Wiring is straightforward:  
GND goes to GND on the Espruino
VCC goes to 3.3v on the Espruino (VCC can be 5v on these parts, which means you could use VBat instead *if* VBat isn't above 5v). 
Connect SDA and SCL to the corresponding pins on the Espruino. 
Connect a 10k pullup resistor between SDA and VCC, and between SCL and VCC. 
Connect a 0.1uf ceramic capacitor between VCC and GND. 
Connect the address pins to GND (for 0) or VCC (for 1). If not using the alternate address pins, connect them all to GND. 

Usage
-----------------

Setup I2C, and then call

```
var digipot=require("MCP4xxxI2C").connect(i2c,pots,taps,addr);
```

`i2c` is the I2C interface it is connected to
`pots` is the number of pots (1, 2, or 4)
`taps` is the number of taps on each pot (129 or 257)
`addr` is the value of the address pins (if omitted, 0 is assumed)

Control it using:

```
digipot.setValue(pot,value,nv)
digipot.getValue(pot,nv)
digipot.getStatus()
digipot.incr(pot) //increment value by 1
digipot.decr(pot) //decrement value by 1
```

`pot` is the number of the pot being acted on (0~3)
`value` is the value to be set
`nv` sets whether to refer to the non-volatile registers (if available) (If omitted, volatile memory is assumed)

`digipot.setTCON(pot,sdwn,a,w,b)` 
The TCON (Terminal CONnect) register controls whether pins on the specified pot are connected to the resistor ladder or not. `sdwn` controls shutdown - if set to 0, the all terminals will be disconnected, 1 for normal operation. `a`, `w`, and `b` control whether the corresponding pins are connected to the resistor ladder.

`digipot.getStatus()` returns an object with up to 5 properties. The pot0-pot3 properties contain the tcon settings for each pot. This is returned as a number from 0-15 - the MSB is shutdown status, then a, w, and b in that order. The `status` property contains the value of the status register, used to indicate successful write to non-volatile memory on devices with such functionality (otherwise, it may be disregarded). 

Applications
----------------


* Extra analog outputs (connect one side to VCC and the other side to ground, and you have a simple DAC)
* Replacing resistors used to set behavior of parts - for example, many constant current source chips (LED drivers, et al) have a resistor to set the current (and hence brightness). 
* Replacing analog pots on external devices, allowing them to be digitally controlled
* Controlling voltage or current adjustments on a power supply, creating a digitally controlled power supply. Be aware of the limits on the analog voltages applied across the potentiometer. A high voltage potentiometer may be better for this purpose.
