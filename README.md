# PiStorm Tester
![image](https://github.com/abrugsch/PistormTester/raw/main/pics/zz9-top-render.jpg)

## Assistance:
If you've gone through the instructions below and watched the video and you need more help decoding what the board is telling you (some things are not so obvious if you're not ultra familiar with the PiStorm schematic) then feel free to hit me up. the best way is usually through Discord via the [PiStorm server](https://discord.gg/mrmyGmvGE5) and I'm @ABrugsch on there too.

## Donate:  
**If you build these, consider [Buying me a coffee](https://ko-fi.com/abrugsch)**
I have spent a considerable amount of time refining the board and the tests that go with them, and will often help people understand what their ZZ9 is telling them. A little coffee goes a long way to helping me keep my sanity 🙂  
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/V7V13CVG6)

### What is?
PiStorm Tester is a PCB that exposes all the IO's of the 68000 DIP64 socket that [PiStorm](https://github.com/captain-amygdala/pistorm/) uses. There are status LED's on all the control/Address/Data Bus lines to easily see if a single line is stuck high or low. Each IO is also broken out to a header for easy connection to an external microcontroller such as an arduino for more accurate analysis of the outputs, or for pulling inputs high or low as required. where the line is either an input or bi-directional (such as the Data bus) the headers have a handy VCC and GND rail adjacent for easy jumpering. 
Combined with testing programs on the pi such as buptest, it becomes easy to narrow down individual faults such as unsoldered or broken individual flip-flops.

### Make Them
* Latest Version gerbers: [RevC.1](https://github.com/abrugsch/PistormTester/tree/main/production/gerbers/PiStormTesterRevC.1.zip)

You can build them by hand. submit the gerbers (Rev C gerbers have adjustments made for ZIF sockets with bigger pins. If you definitely don't want to use a ZIF, a Rev-B board might fit better but RevC should be universal) to a PCB fab of your choice (AllPCB has a free promotion ATM) but then you have to place and solder 60 0603 LED's and 15 4x0402 resistor arrays by hand. That's pretty tedious so I've also included the JLCPCB [production files](https://github.com/abrugsch/PistormTester/tree/main/production) to get them assembled. All the JLC parts are basic parts so it costs barely more to get them assembled than to just order the bare boards. (10x costs around $50 including UK delivery by courier)  
* PistormTester-top-pos-JLC.csv - JLC position file
* PistormTester-BOM-jlc.csv - JLC BOM file
* gerbers/PiStormTesterRevC.1.zip - latest gerber files

~~The big caveat is that they(JLCPCB assembly) don't have a 5v 7-ish MHz oscillator in their assembly catalog so that part has an LCSC part number ([C387338](https://lcsc.com/product-detail/Oscillators_Shenzhen-SCTF-Elec-S3D8-000000A20F30T_C387338.html)) but must be sourced and placed separately.~~ As of RevC.1 there is now a 3.3v oscillator and regulator so that all parts can be assembled by JLC with the included BOM. There are even alternative 3.3v Oscillators if the one in the BOM is out of stock. If you use a RevC or older gerber set, then the BOM and position files will be incorrect so please roll them back to earlier versions.  

__IMPORTANT__ There is now a jumper 0 Ohm resistor in the BOM and position file. If you use the 3.3v regulator and oscillator you ___MUST___ exclude R1 from the build. If you use a 5v Oscillator you ___MUST___ exclude U1, C3 and C4 from the build and keep R1.  

If you do a JLC run of 10 boards, they should come in at around $30 (plus taxes and delivery - Total cost to the UK using an SMD voucher and their cheapest (about 2 weeks) delivery was $37 last time I did a batch, £35 for all red LEDs.)  

The biggest costs in producing these boards are the LED's and the oscillator. If you don't care about colo(u)r of LEDs the cheapest option is to use all red ones as they are significantly cheaper than any other colo(u)r so there is an alternative BOM file with all the LED's using the cheaper RED part. (Although now that the BOM LCSC part numbers have been tweaked to put more expensive ones on places where the lowest quantity are, the difference isn't so great: about $2 between All-Red and different colours.)  
* PistormTester-all-red-BOM-jlc.csv  

### How to
The code folder of this repo has some simple tests for checking that the address and data busses are behaving correctly. Here are the instructions for using them on PiStorm. For more advanced tests, see the note at the bottom of this readme.    

If you have a recent pull of the [main PiStorm repo](https://github.com/captain-amygdala/pistorm/) then you do not need to copy any files as the test files are now included in the PiStorm root folder, though you will still need to follow the build instructions below, but the code files are probably already in your PiStorm folder.  
If not, then copy the files in the [code folder of this repo](https://github.com/abrugsch/PistormTester/tree/main/code) to your pistorm folder on your pi. (put the files in directly, not in a subfolder.)  
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
The zz9fulltest program will then walk the databus lines individually and read the state back so if there is a discrepancy between what is written and what is read then it will be displayed as a data mismatch error.
If you want to force an error, you can go ahead and set one of the data lines high or low with the breakout area by bridging the centre row with an adjacent vcc or gnd. This is probably best done with a ~1K resistor, but not entirely necessary.

Any discrepancies at this point can be referenced to the schematic to find the appropriate pin that might need attention.

For a more thorough test of the data reads there is also zz9readloop which will loop for 2 minutes reading the data bus and displaying the results on screen in Hex and Binary.

simply run  
> ```sudo ./zz9readloop```  

and then modify data bits using the tweak headers and see what comes on the screen. A-variant flip-flops (LVC16373**A** or LVC16374**A**) will hold the data after you set it, allowing you to set multiple bits at a time while only using one wire/resistor to do it.

The program will read at half second intervals for 2 minutes.  
ctrl-c will terminate early  
There is now a video demonstrating the use of the ZZ9 board:  
[![PiStorm Tester Howto!](https://user-images.githubusercontent.com/1519975/128720736-20eae241-e480-43d3-9899-2ef6e7a40ef6.jpg)](https://www.youtube.com/watch?v=HWeGSCD97hg)  
 
### Typical problems: 
* Address or data bus LED's don't come on when they are supposed to:  
This usually means the output pin of the flip-flop that should be connected to the CPU pin isn't actually connected. This can either be an incorrectly soldered CPU pin or flip-flop pin. Knowing which Data or Address pin is at fault can then be directly traced through the schematic in the code folder. It can also mean a connection from the CPLD to the flip-flops is broken. This can manifest as both an address pin AND a data pin being out. e.g. A3 and D3, due to the way the internal data path is shared.  
* Address or data LED's stuck on  
can either be a short to VCC or sometimes open circuit on the flip-flops input as the inputs can float high.
* LED's show correct but there's a buptest read mismatch  
Run zz9readloop and while it's doing the loop, put each data pin high or low and see that the program shows the read data correctly (the state of the lights is displayed directly on the read loop in both Hex and Binary (bit) format)  
Example: if you make D3 high (connect to vcc) zz9readloop should say `READ16: read data: 0x0008(hex) 0000000000001000 (binary)`
* Whole 8-bit blocks of any of the address/data busses are not working:  
each 8-bit bank (A1-7, A8-15, A16-24, D0-7, D8-15) is enabled by an output enable pin on the respective latch chip and is controlled by the CPLD. They are active low and if left floating can have unpredictable results, but usually simply fail to operate the entire bank. The OE pins to check for each bank are as follows:  

| bank | chip | pin |  
:-: | :-: | :-:  
| A1-7 | U2 | 1 |  
| A8-15 | U2 | 24 |  
| A16-23 | U6 | 1 |  
| D0-7 | U5 | 1 |  
| D8-15 | U5 | 24 |  

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
Example: a disconnected pin on the 384 on the CPLD side of the RST line meant the CPLD I/O was floating. The test board appeared correct (light off) but in use the emulator wouldn't start because RST was toggling on and off due to floating. Re-flowing the suspect 384 resolved the problem.

Testing of the supporting signals will probably have to be done with a logic analyzer or a complementing microcontroller dev board such as an arduino (one that's 5v tolerant) to read the pins and log what states they take. This board was originally intended to have a socket for a mini/micro arduino but for the sake of flexibility was left off.

As Advanced tests are created by myself or the PiStorm community, they will be documented here. (please submit issues or pull requests for any suitable tests you need or have come up with)
