# Core B1 + Waveshare_Open337 expansion board
This repo is aimed to help anyone who has recently bought the Waveshare Core B1 + Open4337-C expansion board and need a little push/help to start programming/flashing code into it.

When I started from scratch with Cortex-M series Microcontrollers and particularly with this board (which hosts an NXP LPC4337-JBD144), the biggest issue I had at the beginning was how to comunicate with it since there isn't that much info provided by Waveshare on how to program flash memory of the chip. The only article I've found uses Windows and a program called Flashmagic to programm the ".hex", so it was useless to me.

STOP!! --> Before continuing, it is strongly suggested that the reader gets familiar Chapter 7 of the LPC4337's datasheet and have a clear picture of what the "Boot Source Bits" of the LPC4337 do.
Talk about flash banks

# About the board

In the box there is nothing else besides the two boards and 2 usb cables.

## The Core B1 board
The core board hosts the microcontroller and the peripherals connected to the row of pin headers as well as 2 USB ports and a set of jumpers to configure the aforementioned "Boot Source Bits".

Core B1 schematic: https://www.waveshare.com/w/upload/1/16/Core4337-Schematic.pdf

## The Open4337-C expansion Board
Not too much to say about about this. Cool thing about the board is that it already has a PL-2303 in charge of adapting the USART0.
Expansion board schematic: https://www.waveshare.com/w/upload/9/96/Open4337-C-Schematic.pdf

# How to flash the chip?
I will present two possible ways of flashing bin or hex files into the chip.

## ISP Programming via USART0
If you start looking for possible tools to flash the most of them will lead you to the same one Flashmagic. Sadly this app only works on Windows.
### Alternative 1: lpc21isp (NOT SUITABLE FOR flashing LPC43XX, April 2020). 
I've found lpc21isp as a possible alternative to flash NXP's cortex microcontrollers.  
a) Boot source bits must be configured all to 0 (USART0).  
b) Plug the USB cable to the UART0 of the Open4337 expansion board.  
c) Short P2_7 to GND and press the reset button in order to make it go into ISP mode.  
d) Run lpc21isp to detect your chip. Pay attention to the parameters that you need to pass i.e: baudrate, specify externsion (bin or hex), Osc frequency **in KHz**.


![lpcmanual](https://github.com/snorkman88/Waveshare_Open337/blob/master/lpcmanual.png).  

and you should receive something like this.   

![detectonly](https://github.com/snorkman88/Waveshare_Open337/blob/master/detectonly.png).  


e) Write your hex to your chip. 

Although steps a) to d) are still valid for LPC43xx chips, step e) will always fail.
After spending almost two days going through the debug lines y started checking the "prepare P" and "copy C" commands sent to the device and realized that I missed a very important and well documented detail http://manpages.ubuntu.com/manpages/xenial/man1/lpc21isp.1.html
"This tool has been specifically designed for LPC1100/LPC1300/LPC1700/LPC2000 series ARM7/Cortex-M0/Cortex-M3 microcontrollers."  
So to sum up, seems like lpc21isp does not have the memory map of the LPC4337 in it and it's always trying to write from RAM to the address 0 of the Flash. However, since BankA of the flash starts at 0x1A000000 (decimal 436207616) the chip always returned "Destination address is not mapped in the memory map".  
Therefore, this tool does not seem to be appropriate for flashing this chip.  


### Alternative 2: mxli. 
