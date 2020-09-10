HairyDairyMaid WRT54G Debrick Utility
=====================================

This is a mirror of the original HairyDairyMaid JTAG Debrick Utility v4.8.

This program is for De-Bricking the WRT54G/GS routers.

This software was found on the [download.modem-help.co.uk][1] website, along with other helpful JTAG tools.

I have made this source code freely available according to the terms of the GPLv2 
for purposes of gifting it back to the community, and to spur the sharing of further improvements and 
forks of this great software that has brought so many routers back to life :smile_cat:.
Given that it's a significant piece of Open Source software enabling JTAG recovery of so many routers, it should be included in the [GitHub Arctic Code Archive][12].

Forks & Modifications
=====================

I am not currently interested in maintaining this software, but am completely open to anyone improving it,
forking and taking it in new directions.

Many forks and modifications are available for this utility including but not limited to:

 - [zjtag][4] - An EJTAG Debrick Utility based on BrJTAG
 - [brjtag][5] - Broadcom EJTAG Debrick Utility - by hugebird
 - [jpt][6] - JTag programming tools (looks like a small wrapper script project over brjtag to modify a [CFE (Common Firmware Environment)][7] and modify the stored MAC address for a router so routers with broken CFE can be repaired and programmed with their original MAC)
 - [bpjtag][8] - HairyDairyMaid's debrick tool ported to use Bus Pirate OpenOCD mode
 - [Various JTAG Software][9] - Modem-Help.org's listing of JTAG Tools (Please consider making a donation to help support their community)
 - [tjtag-arduino][10] - Fork of tjtag with Arduino support! Use the arduino as a JTAG adaptor with Tjtag.
 - [tjtag-pi][11] - Fork of tjtag with Raspberry Pi support! 

If you have found a fork that you want to put on this list, feel free to submit a Pull Request to have it added to the README.

License
=======

This software is the original work of "HairyDairyMaid (a.k.a. Lightbulb)".  The original source code for `wrt54g.h` and `wrt54g.c`
contain the [GPLv2][2] license notice, and are:

Copyright © 2004 HairyDairyMaid (a.k.a. Lightbulb)

[Copyleft][3] 🄯 2004 HairyDairyMaid (a.k.a. Lightbulb)

The license text has been also included in the file `LICENSE` in this repo.

[1]: http://download.modem-help.co.uk/utilities/JTAG/Software/
[2]: http://choosealicense.com/licenses/gpl-2.0/
[3]: http://en.wikipedia.org/wiki/Copyleft
[4]: https://github.com/zoobab/zjtag "EJTAG Debrick Utility based on BrJTAG"
[5]: http://www.codeforge.com/article/211104
[6]: https://github.com/beoran/jpt
[7]: http://www.tiaowiki.com/w/Debrick_Routers_Using_JTAG_Cable#Common_Firmware_Environment_.28CFE.29
[8]: https://github.com/notch/bpjtag
[9]: http://download.modem-help.co.uk/utilities/JTAG/Software/
[10]: https://github.com/zoobab/tjtag-arduino
[11]: https://github.com/oxplot/tjtag-pi
[12]: https://archiveprogram.github.com/


It wasn't working out of the box and I had to tweak the code a little bit. You can get my changes from the git repository

$ git clone http://git.stefanchrist.eu/HairyDairyMaid_WRT54G_Debrick_Utility_v48.git

After compiling HairyDairyMaid you can try it with

$ ./wrt54g  -probeonly /noemw

====================================
WRT54G/GS EJTAG Debrick Utility v4.8
====================================

Probing bus ... Done

Instruction Length set to 8

CPU Chip ID: 00000101001101010010000101111111 (0535217F)
*** Found a Broadcom BCM5352 Rev 1 CPU chip ***

    - EJTAG IMPCODE ....... : 00000000100000000000100100000100 (00800904)
    - EJTAG Version ....... : 1 or 2.0
    - EJTAG DMA Support ... : Yes

Issuing Processor / Peripheral Reset ... Done
Enabling Memory Writes ... Skipped
Halting Processor ... <Processor Entered Debug Mode!> ... Done
Clearing Watchdog ... Done

Probing Flash at (Flash Window: 0x1fc00000) ... Done

Flash Vendor ID: 00000000000000000000000011000010 (000000C2)
Flash Device ID: 00000000000000000010001010101000 (000022A8)
*** Found a MX29LV320B 2Mx16 BotB      (4MB) Flash Chip ***

    - Flash Chip Window Start .... : 1fc00000
    - Flash Chip Window Length ... : 00400000
    - Selected Area Start ........ : 00000000
    - Selected Area Length ....... : 00000000



 *** REQUESTED OPERATION IS COMPLETE ***

You see, it's working nicely. I had to use /noemw otherwise my router stopped working.
4. Flash layout and backup

The WRT54GL contains 4MiB of flash memory. If your OpenWrt firmware is running, you can see the flash layout with the command cat /proc/mtd. The output is

dev:    size   erasesize  name
mtd0: 00040000 00010000   "cfe"
mtd1: 003b0000 00010000   "linux"
mtd2: 002c9800 00010000   "rootfs"
mtd3: 00170000 00010000   "rootfs_data"
mtd4: 00010000 00010000   "nvram"

From this you can guess the actual flash layout

from - to              size         name
0x000000 - 0x03FFFF    0x040000     CFE
0x040000 - 0x3EFFFF    0x3b0000     Firmware (here 'linux', also called 'kernel')
0x3F0000 - 0x400000    0x010000     nvram

For further details see The OpenWrt Flash Layout in the OpenWrt wiki. CFE is the bootloader of the device (You can get the source from the broadcom site Communications Processors Downloads. It shouldn't be possible to build your own CFE binary, because the hardware vendors may include there own setup code). The CFE will read configuration variables from the nvram (e.g. the above mentioned 'boot_wait' is such a configuration parameter). You can either use the nvram utility of OpenWrt or read it with your favorite unix tools, e.g. cat /dev/mtd4ro | strings. The Firmware is the actual operating system of your router. It maybe the original firmware or a free one like OpenWrt.

Before flashing a new firmware, I took a minute and made a backup of my flash memory. I could to this with

$ ./wrt54g -backup:kernel /noemw
$ ./wrt54g -backup:cfe /noemw
$ ./wrt54g -backup:nvram /noemw

These commands created three files in the current working directory:

'KERNEL.BIN.SAVED_20131214_235857', 'CFE.BIN.SAVED_20131215_004211' and 'NVRAM.BIN.SAVED_20131215_004553'

Based on the timestamps you can guess that this procedure took more than just a minute ;-). The JTAG interfaces is quite slow. Therefore the HairyDairyMaid utility is recommending:

***************************************************************************
* Flashing the KERNEL or WHOLEFLASH will take a very long time using JTAG *
* via this utility.  You are better off flashing the CFE & NVRAM files    *
* & then using the normal TFTP method to flash the KERNEL via ethernet.   *
***************************************************************************

In my case this approach wasn't possible, because I haven't made a backup of the CFE and the nvram before the accident. Additionally the TFTP server in the bootloader wasn't working.
5. Flashing a new firmware

The last step is flashing a new firmware. First I tried to use my OpenWrt firmware, but the router didn't come up. So I decided to use the original firmware from Linksys. You can find it on the product page Linkysys WRT54GL. Here is the direct link to the file: FW_WRT54GL_4.30.16.6_US_20130308_code.bin (sha1sum: 313a977ca262a8908eab18ad8e912e128407ee33). Running the Linksys firmware on the device I used the webinterface to flash my OpenWrt firmware again. Yeah the Linksys firmware is only useful as a sophisticated flash utility to install a real OS :-).
Later I noticed that this procedure is suggested in the DD-Wrt wiki - Recover from a bad flash:

    Linksys WRT54 GL:
    Linksys wrt54 GL users please note that if flashing with tftp using dd-wrt firmware gives no results, original Linksys firmware from www.linksys.com is worth trying. If that works, do a hard reset and you can continue to flash with dd-wrt.

Ok, the overall process is clear. How to use the HairyDairyMaid program to flash the Linksys firmware/kernel? First you have to convert the .bin firmware image to a trx-file. You have to remove the first 32 bytes, e.g. you can use the tool dd

$ dd bs=32 skip=1 if=FW_WRT54GL_4.30.16.6_US_20130308_code.bin of=FW_WRT54GL_4.30.16.6_US_20130308_code.trx

Then the trx-file must be flashed to the flash location 0x040000. On boot CFE will compare the checksum of the firmware. If it's correct, it will load and execute it. More information is in the OpenWrt wiki TRX vs. TRX2 vs. BIN.
To flash it you have to rename the trx-file to KERNEL.bin, because the program wrt54g uses this specific filename. There isn't a commandline parameter for that. Instead of renaming I used a symbolic link:

$ ln -s FW_WRT54GL_4.30.16.6_US_20130308_code.trx KERNEL.BIN
$ ./wrt54g -flash:kernel /noemw

Flashing with the JTAG adapter will take some time, but afterwords you have a happily running WRT54GL router again. That's it. 
