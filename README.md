# Core B1 + Open337 expansion board by Waveshare
This repo is aimed to help anyone who has recently bought the Waveshare Core B1 + Open4337-C expansion board and needs a little push/help to start programming/flashing code into it.

When I started from scratch with Cortex-M series Microcontrollers and particularly with this board (which hosts an NXP LPC4337-JBD144), the biggest issue I had at the beginning was how to comunicate with it since there isn't that much info provided by Waveshare on how to program the flash memory of the chip. The only article I've found uses Windows and a program called Flashmagic to programm the ".hex", so it was useless to me because I use Ubuntu and Mac.

**STOP!! -->** Before continuing, it is strongly suggested that the reader gets familiar Chapter 7 of the LPC4337's datasheet and have a clear picture of what the "Boot Source Bits" of the LPC4337 do. 
Keep in mind that the LPC4337 has 2 banks of flash A & B (Chapter 7.11 - On-chip flash memory). 

# About the board
![](https://github.com/snorkman88/Waveshare_Open337/blob/master/all_together.jpg). 
In the box there is nothing else besides the two boards and 2 usb cables.  

**IMPORTANT: there isn't an embedded JTAG debugger in this kit, you will have to buy one separately.**. 

## The Core B1 board
The core board hosts the microcontroller and the peripherals connected to the row of pin headers as well as 2 USB ports and a set of jumpers to configure the aforementioned "Boot Source Bits".
![coreboard1](https://github.com/snorkman88/Waveshare_Open337/blob/master/core_front.jpg)
![coreback](https://github.com/snorkman88/Waveshare_Open337/blob/master/core_back.jpg)
Core B1 schematic: https://www.waveshare.com/w/upload/1/16/Core4337-Schematic.pdf

## The Open4337-C expansion Board
Not too much to say about about this. Cool thing about the board is that it already has a PL-2303 in charge of adapting the USART0.  
![](https://github.com/snorkman88/Waveshare_Open337/blob/master/expansion_board.jpg)
![](https://github.com/snorkman88/Waveshare_Open337/blob/master/uart_on_expansion.jpg)
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

Although steps **a)** to **d)** are still valid for LPC43xx chips, step **e)** will always fail.  
![failedcopy](https://github.com/snorkman88/Waveshare_Open337/blob/master/failedcopy.png)
After spending almost two days going through the debug lines y started checking the "prepare P" and "copy C" commands sent to the device and realized that I missed a very important and well documented detail http://manpages.ubuntu.com/manpages/xenial/man1/lpc21isp.1.html. 

"This tool has been specifically designed for LPC1100/LPC1300/LPC1700/LPC2000 series ARM7/Cortex-M0/Cortex-M3 microcontrollers".  

![debug](https://github.com/snorkman88/Waveshare_Open337/blob/master/debug.png). 

So to sum up, seems like lpc21isp does not have the memory map of the LPC4337 in it and it's always trying to write from RAM to the address 0 of the Flash. However, since BankA of the flash starts at 0x1A000000 (decimal 436207616) the chip always returned "Destination address is not mapped in the memory map".  
Therefore, this tool does not seem to be appropriate for flashing this chip.  


### Alternative 2: mxli (Did not have time to test it YET). 

## ISP Programming via USB using "dfu-util"
The second possible method for writting the "hex" or "bin" into the flash memory of the chip, is making the LPC4337 boot from USB in DFU "device" mode. 

**IMPORTANT:** Once again, it is important to remark that this method is only possible if the target chip that you are trying to flash already has implemented on it the "device" side of the USB DFU protocol. If so, then your computer/notebook will adopt the role of "host" of the [USB DFU PROTOCOL] (https://www.usb.org/sites/default/files/DFU_1.1.pdf). 
http://manpages.ubuntu.com/manpages/xenial/man1/dfu-util.1.html.  
This will make possible the communication between PC (aka host) and the LPC chip (aka device). 

Before you interact with the board you must first install LPCSCRYPT which is a developed and maintined by NXP.  
https://www.nxp.com/design/microcontrollers-developer-resources/lpc-microcontroller-utilities/lpcscrypt-v2-1-1:LPCSCRYPT?&tab=Design_Tools_Tab7

**LPCScrypt for Linux is a 64-bit application, SO IT WILL NOT RUN on 32-bit systems**.  

**LINUX USERS:** when you download lpcscrypt for linux, the extension of it will be **.deb.bin** just run it with. 

`sudo sh lpcscrypt-2.1.1_15.x86_64.deb.bin`.  

To me the most difficult part of running lpcscrypt was finding it after the installation, so if you happen to be in the same situation as me try looking for it in **/usr/local/lpcscrypt/scripts**.  

Once dfu-util + lpscrypt + dependencies are installed and configured, your are ready to make the board boot into this mode. 

a) Configure the "Boot Source Bits" P1_2 and P2_8 to a logical state of "1".  
![](https://github.com/snorkman88/Waveshare_Open337/blob/master/boot_from_usb0.jpg).  

b) Plug the cable to USB1. 
c) Short P2_7 to GND and press the reset button in order to make it go into ISP mode.    
d) Verify that is already in DFU mode by running `lsusb` and you should see it listed.  

![](https://github.com/snorkman88/Waveshare_Open337/blob/master/lsusb.png).  

e) run lpcscrypt CLI tool `./LPCscrypt_CLI` and you are ready to flash your chip.  

f) Check partinfo, wipe and flash your file.  
![](https://github.com/snorkman88/Waveshare_Open337/blob/master/lpcscrypt_cli_program.png). 


DONE! :-D. 


If any other doubts about lpscrypt still remain, please refer to the user guide: https://www.nxp.com/docs/en/user-guide/LPCScrypt_User_Guide.pdf. 
