# wch32v307
Collection of notes and files related to the wch32v307 chips


# EVT - Eval Board

So I have most things working on the Linux toolchain.   MounRiver Studio works just fine to build and upload the examples.   You plug in to the WCHLink (which is the tab on the top of the eval board) and then you have the ability to program and debug (simultaneously-- the WCHLink attaches a UART to the USB, so you can get debugging messages out that way), it uses SWD to acutally do the file transfer.

To make the eval board with actual USB C connections (not just USB C cables), you need to solder 5.1k 0603 resistors on the back of the board, these signal to the other side of the link that you want power.   The 5.1k value is not super critical.   I have used two 10ks in parallel, but I have even used 4.7k resistors to no ill effect.

However, the ETH examples didn't work.

## Variable RAM vs Flash

So I have no idea how this works from an electronic perspective, but while the ch32v307 is rated as 256K flash and 64K RAM, you can "reallocate" that back and forth.    This seems to be the options available:

| RAM  | FLASH |
|------|-------|
| 32K  | 288K  |
| 64K  | 256K  |
| 96K  | 224K  |
| 128K | 192K  |

So all of the ETH examples are written with 128K RAM and 192K FLASH.   The only documented way to change this is using the wchisptool, which is a windows program.  You can [download it here](http://www.wch-ic.com/downloads/WCHISPTool_Setup_exe.html).    

Setting BOOT0 = 1, puts the device in bootloader mode, and WCHISPTool connects to the device USB port (NOT THE WCHLink port), you cannot adjust the RAM/FLASH split without also uploading a firmware file, so have a .hex file ready to go.   (I did not have one on the windows virtual machine I was running, but I had a random atmel hex, so I uploaded that)

After you change the ram/flash split, you can flip BOOT0 back to 0 and then go back to using WCHLink / MRS to handle your uploads.

One unknown: I have no idea how to tell MRS how much RAM I want to use.   To be updated. 

# Linux Commandline Toolchain

## Linux OpenOCD

There is a separate version of OpenOCD that supports the WCH-Link as a debugger.   (They are in theory required by the GPL to release the source for this, they have not yet to my knowledge).

If you go to the MounRiver download page: http://mounriver.com/download -- click on the MRS Toolchain, it's included in there.

Go to the beforeinstall directory and run the start.sh, which will install the libraries.  

Then go to OpenOCD/bin and run `sudo ./openocd -f wch-riscv.cfg`

I got this:

    Open On-Chip Debugger 0.11.0+dev-02215-gcc0ecfb6d-dirty (2022-03-30-18:53)
    Licensed under GNU GPL v2
    For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
    Info : only one transport option; autoselect 'jtag'
    Ready for Remote Connections
    Info : Listening on port 6666 for tcl connections
    Info : Listening on port 4444 for telnet connections
    Info : WCH-Link version 2.2 
    Info : wlink_init ok
    Info : This adapter doesn't support configurable speed
    Info : JTAG tap: riscv.cpu tap/device found: 0x00000001 (mfg: 0x000 (<invalid>), part: 0x0000, ver: 0x0)
    Warn : Bypassing JTAG setup events due to errors
    Info : [riscv.cpu.0] datacount=2 progbufsize=8
    Info : Examined RISC-V core; found 1 harts
    Info :  hart 0: XLEN=32, misa=0x40901125
    [riscv.cpu.0] Target successfully examined.
    Info : starting gdb server for riscv.cpu.0 on 3333
    Info : Listening on port 3333 for gdb connections

So that seems cool.

## GCC toolchain

So MRS is just using the GCC RISCV toolchain.    [This one](https://xpack.github.io/riscv-none-embed-gcc/install/) in particular.   Included in the toolchain download is a copy of xpack's riscv-non-emebed-gcc toolchain, which is *probably* the stock system, but I don't know for certain.

Where I got a bit stuck was in trying to put together a Makefile based compiler using this toolchain AND the WCH SDK.   I need to do some digging into it, I'm a bit spoiled by having toolchains just ready to go.

I know how to do most of it, but this initial bootstrapping is not something I've done.

## Back to WCHISPTool

So it seems totally reasonable to be able to get to the point of having a .hex file and then being able to flash it on the device via WCHLink.  This is a fabulous development system because you can keep the debug UART running, update flash, get new debugging logs, connect gdb and do all the cool on chip debugging you want.

However, the functionality of WCHISPTool to change the RAM/Flash split is something that will likely need some reverse engineering to figure out what it's doing and how to do that.    

Regardless, if you have a reverse-engineered verison of WCHISPTool that can run on the UART interface (also bootloader there), it should be fairly simple to toggle BOOT0 and nRST to get it into bootloader mode, set the fuses and blow the firmware on there.

## Updates on this front

32.4.6 Selection word register (FLASH_OBR) is the register that controls what happens in terms of the RAM/FLASH split

So it seems the bootloader has a method by which you can modify these special registers.    So it's definitely doable, just need to find out how the bootloader works.
