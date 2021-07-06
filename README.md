# PiStorm Tester
![image](https://github.com/abrugsch/PistormTester/raw/main/pics/zz9-top-render.jpg)

### What is?
PiStorm Tester is a PCB that exposes all the IO's of the 68000 DIP64 socket that PiStorm uses. There are status LED's on all the control/Address/Data Bus lines to easily see if a single line is stuck high or low. each IO is also broken out to a header for easy connection to an external microcontroller such as an arduino for more accurate analysis of the outputs, or for pulling inputs high or low as required. where the line is either an input or bi-directional (such as the Data bus) the headers have a handy VCC and GND rail adjacent for easy jumpering. 
Combined with testing programs on the pi such as buptest, it becomes easy to narrw down individual faults such as unsoldered or broken individual flip-flops.
### How to
First copy the files in the code folder to your pistorm folder on your pi.  
Run ./build_zz9.sh to build the test program just like buptest.  
Then run sudo ./zz9fulltest.
* Test 1:  
Watch the LED's on the address bus. they should all light in turn from A1 to A23. AS should dim slightly as it goes from being held high to being pulsed. All data bits should appear to be on for the duration of this test.
* Test 2:
Watch the Data bus LED's. They should light in turn from D0 through to D15. the zz9fulltest program will also read the state back so if there is a discrepency between what is written and what is read then it will be displayed as a data mismatch error.
if you want to force an error, you can goahead and set one of the data lines high or low with the breakout area by bridging the centre row with an adjacent vcc or gnd. this is probably best done with a 1K resistor, but not entirely necessary.

Any discrepencies at this point can be referenced to the schematic to find the appropriate pin that might need attention.

### Typical problems: 
* Address or data bus LED's don't come on when supposed to  
This is usually means the output pin of the flip-flop connected to the CPU pin isn't connected. This can either be a incorrectly soldered CPU pin or flip-flop pin. Knowing which Data or Address pin is at fault can then be directly traced through the schematic in the code folder. It can also mean a connection from the CPLD to the flip-flops is broken. This can manifest as bot an address pin AND a data pin being out. e.g. A3 and D3, due to the way the internal data path is shared.  
* Address or data LED's stuck on  
can either be a short to VCC or sometimes open circuit on the flip-flops input as the inputs can float high.
* LED's show correct but there's a read mismatch  
Run buptest and while it's doing a read loop, put each data pin high or low and see that buptest shows the read data correctly (the state of the lights should be what it reads on the read loop)  
Example: if you make D3 high (connect to vcc) buptest should say garbege data mismatch at $[some address] read 0x0008. should be: 0x[some random number]  
The hex value read should correspond to what you set (remember how to convert data bits to hex... D0 = 0x0001, D1 = 0x0002, D2 = 0x0004, D3 = 0x0008 - repeat up to D15 = 0x8000)

TODO: the tester program needs a read loop to display what's been set on the data bus in real time until the program is quit. For now use the read portion of buptest.
