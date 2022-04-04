# Warning
This folder in particular contains works in progress that definitely do not work right now.

It's here as a useful resource and to be able to link from blog posts, but don't trust anything here.


# Linux boostrapping

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