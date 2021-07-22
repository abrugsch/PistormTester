# PiStorm Tester
![image](https://github.com/abrugsch/PistormTester/raw/main/pics/zz9-top-render.jpg)

### What is?
PiStorm Tester is a PCB that exposes all the IO's of the 68000 DIP64 socket that [PiStorm](https://github.com/captain-amygdala/pistorm/) uses. There are status LED's on all the control/Address/Data Bus lines to easily see if a single line is stuck high or low. each IO is also broken out to a header for easy connection to an external microcontroller such as an arduino for more accurate analysis of the outputs, or for pulling inputs high or low as required. where the line is either an input or bi-directional (such as the Data bus) the headers have a handy VCC and GND rail adjacent for easy jumpering. 
Combined with testing programs on the pi such as buptest, it becomes easy to narrw down individual faults such as unsoldered or broken individual flip-flops.

### Make Them
You can build them by hand. submit the gerbers (Rev B gerbers have not yet been tested. use at your own risk. roll back to Rev A if you want a confirmed working board) to a PCB fab of your choice (AllPCB has a free promotion ATM) but then you have to place and solder 60 0603 LED's and 15 4x0402 resistor arrays by hand. That's pretty tedious so I've also included the JLCPCB production files to get them assembled. All the JLC parts are basic parts so it costs barely more to get them assembled than to just order the bare boards. (10x costs around $50 including UK delivery by courier)

The big caveat is that they(JLCPCB assembly) don't have a 5v 7-ish MHz oscillator in their assembly catalog so that part has an LCSC part number ([C387338](https://lcsc.com/product-detail/Oscillators_Shenzhen-SCTF-Elec-S3D8-000000A20F30T_C387338.html)) but must be sourced and placed seperately.

### How to
The code folder of this repo has some simple tests for checking that the address and data busses are behaving correctly. here are the instructions for using them on PiStorm. For more advanced tests, see the note at the bottom of this readme.    
First copy the files in the [code folder of this repo](https://github.com/abrugsch/PistormTester/tree/main/code) to your pistorm folder on your pi. (put the files in directly, not in a subfolder.)  
Run  
> ```./build_zz9tests.sh```  

to build the test program just like buptest.  
Then run  
> ```sudo ./zz9fulltest```  

* Test 1:  
Watch the LED's on the address bus. they should all light in turn from A1 to A23. AS should dim slightly as it goes from being held high to being pulsed. All data bits should appear to be on for the duration of this test.
* Test 2:  
Watch the Data bus LED's. They should light in turn from D0 through to D15.  
* Test 3:  
**THIS TEST ONLY WORKS FOR NON-A VARIANT FLIP-FLOPS** On A-Variant flip-flops (LVC16373**A** or LVC16374**A**) the data bits won't even appear to light up as they are only on for such a brief period of time  
The zz9fulltest program will then walk the databus lines individually and read the state back so if there is a discrepency between what is written and what is read then it will be displayed as a data mismatch error.
If you want to force an error, you can go ahead and set one of the data lines high or low with the breakout area by bridging the centre row with an adjacent vcc or gnd. this is probably best done with a ~1K resistor, but not entirely necessary.

Any discrepencies at this point can be referenced to the schematic to find the appropriate pin that might need attention.

For a more thorough test of the data reads there is also zz9readloop which will loop for 2 minutes reading the data bus and displaying the results on screen in Hex and Binary.

simply run  
> ```sudo ./zz9readloop```  

and then modify data bits using the tweak headers and see what comes on the screen. A-variant flip-flops (LVC16373**A** or LVC16374**A**) will hold the data after you set it allowing you to set multiple bits at a time while only using one wire/resistor to do it.

The program will read at half second intervals for 2 minutes.  
ctrl-c will terminate early

### Typical problems: 
* Address or data bus LED's don't come on when supposed to  
This is usually means the output pin of the flip-flop connected to the CPU pin isn't connected. This can either be a incorrectly soldered CPU pin or flip-flop pin. Knowing which Data or Address pin is at fault can then be directly traced through the schematic in the code folder. It can also mean a connection from the CPLD to the flip-flops is broken. This can manifest as bot an address pin AND a data pin being out. e.g. A3 and D3, due to the way the internal data path is shared.  
* Address or data LED's stuck on  
can either be a short to VCC or sometimes open circuit on the flip-flops input as the inputs can float high.
* LED's show correct but there's a read mismatch  
Run buptest and while it's doing a read loop, put each data pin high or low and see that buptest shows the read data correctly (the state of the lights should be what it reads on the read loop)  
Example: if you make D3 high (connect to vcc) buptest should say garbege data mismatch at $[some address] read 0x0008. should be: 0x[some random number]  
The hex value read should correspond to what you set (remember how to convert data bits to hex... D0 = 0x0001, D1 = 0x0002, D2 = 0x0004, D3 = 0x0008 - repeat up to D15 = 0x8000)

If you need to address any of these issues, here's where to start:  
Data **read** is handled by U3 on the PiStorm  
Data **write** is handled by U5

Address bus (write only) A1-A15 is handled by U2  
Address bus A16-A23 is handled by U6

Reference the [Pistorm schematic](https://github.com/abrugsch/PistormTester/blob/main/code/Pistorm_Rev_B_schematic.pdf) for further details of specific pins to target 

## Advanced tests
Currently the included tests only cover the address and data busses. There are 18 other signal lines that are handled by this tester and at the moment nothing has been written to assess these other lines.  
When viewing the board, in normal operation the following should be lit (dimly as they are being strobed)  
* IPL0
* IPL1
* IPL2
* BG
* VMA
* E
* UDS
* LDS
* RW

This only covers the output of the 384 data buffers, and therefore there could also be hidden disconnections between the 384's and the CPLD.  
Example: a disconnected pin on the 384 on the CPLD side of the RST line meant the CPLD I/O was floating. the test board appeared correct (light off) but in use the emulator wouldn't start because RST was toggling on and off due to floating. Re-flowing the suspect 384 resolved the problem.

Testing of the supporting signals will probably have to be done with a logic analyzer or a complementing microcontroller dev board such as an arduino (one that's 5v tolerant) to read the pins and log what states they take. this board was originally intended to have a socket for a mini/micro arduino but for the sake of flexibility was left off.

As Advanced tests are created by myself or the PiStorm community, they will be documented here. (please submit issues or pull requests for any suitable tests you need or have come up with)
