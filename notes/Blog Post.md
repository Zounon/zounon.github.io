# Modifying Firmware for Fun and Profit 


Something about me, I find it pretty annoying when my devices tell me what I must do.
Isn't it annoying .. phone demands you update within the next 48hours, 


From my highschool days I owned a calculator. It's obvious that since it's my calculator I can use it how I please. Somehow companies have make us think differently about our phones, gaming consoles, and cars.

All these devices are mine but so many phones come pre-installed with facebook / youtube / google and **theres no way to uninstall it!** 
I find this behaviour frankly disgusting. It's one thing to say I'm banned from using your service, but when did I lose my right to not delete youtube from my phone?
Between any computers hardware and software is it's firmware.
Firmware is like software, except -as the name suggests- it's not as flexible, runs closer to the hardware. 

You, the user, interacts with the software layer by tapping the camera app on your phone. The software isn't able to access the camera directly, it must ask the firmware to stream camera data it's way. Then the firmware talks directly with the camera.

These layers, Software, Firmware, Hardware, are necessary for abstracting away the intricacies of each bit of hardware. Many OS will run on your computer because they all just ask "open the webcam please", and it's the firware's job to ...



well I say any OS. The firmware can have written in it "don't start if the OS is compromised" 
At first glance this is a nice security feature, if someone happened to hack your OS, it can detect this and choose to not load.
The problem is that companies are far more incetivized by money rather than customer saftey. 

Xobx and playstations will 'brick' if they detect you've tried to install your own software.
Phones similarly won't run if you've gained 'root' access on them. 



Lets make these devices ours again.


Similar to the a.out file you get after compiling C code, firmware  compiled - made to run, not to be read. 

This post will give an overview of static and dynamic firmware analysis,
The process of 


We'll start by assuming you have the firmware file in hand. For popular devices, firmware can be simply downloaded from host servers, in other cases it must be extracted from the target device.

#### - Step 1 - Static Analysis - what are we working with
Firmware is the glue between the ignorant software and the very particular hardware.
...
Firmware is very specific to each device

###### - file / strings / hexdump / fdisk
###### Binwalk!

###### SquashFS and dd


#### Dissecting files with Ghidra, Radare2 or IDA Pro

reading binaries without symbols
what it means to not have symbols

how can these tools make sense of anything?


## Dynamic analysis

Static analysis is safe  but hard to understand
lets make it run!

#### QEMU 
Emulating MIPS / ARM hardware


#### user 
#### user-static
#### full system emulation







                          Embeded System Organization
                          ~~~~~~~~~~~~~~~~~~~~~~~~~~~

                               What Is Firmware Good For?
                               ~~~~~~~~~~~~~~~~---------

                               Static Firmware Analysis
                               ~~~~~~~~~~~~~~~~--

                               Dynamic Firmware Analysis
                               ~~~~~~~~~~~~~~~~--

                               Modifying for Fun and Profit
                               ~~~~~~~~~~~~~~

very specific to particular use case, 
firmware is packaged in an optimized way.
anding an extra byte will offset the rest of the firmware by one byte

complex firmware often include these security features ---
- here are conman attack around them

as simple as changing one constant value, (game engine)
as complex as ? 



Cite links to tools mentioned 
particularly influential 
shout out to Will for introducing and teaching me about the topic

Presented here is a very high overview of what firmware analysis often entails. Hopefully  



Here i give 

                                 Introduction
                                 ~~~~~~~~~~~~


#### To play along
This post goes over the general process of firmware analysis, each firmware is unique and requires it's own tricks and tools to modify. If you'd like to play along there are some resources to start
[0](https://book.hacktricks.xyz/hardware-physical-access/firmware-analysis#vulnerable-firmware-to-practice) 

### What are we working with? 

Using the file command 
`file firmware.bin`


`f.bin: firmware 1042 v1 TP-LINK Technologies ver. 1.0, version 3.15.7, 8258048 bytes or less, _ at 0x200 998400 bytes , _ at 0x100000 7077888 bytes 

`s.bin: u-boot legacy uImage, SPI Flash Image, Linux/MIPS, Standalone Program (Not compressed), 111116 bytes, Load Address: 0X80200000, Entry Point: 0X80200000, Header CRC: 0X80EF7587, Data CRC: 0XCD95F789`

`x.bin: data`


The output of `file` on three firmware files already demonstrates wild diversity between different firmware.  
The first firmware,  `f.bin` (claims) to be firmware for a TP-LINK router. We nicely given the firmware's version, the router model and version.

`s.bin` shows a more complex firmware. This one has a u-boot image, SPI falsh image, and even Linux built for MIPS arch! 

Before we dive into what `_ at 0x200 998400 bytes` and `Entry Point: 0X80200000, Header CRC: 0X80EF7587` mean. Let's answer why `x.bin` isn't telling us anything about itself.


To get all of this information the `file` command reads the the first 1024 bytes of the file if it appears to be ASCII, or if the target is binary it'll consult `/etc/magic/` ,
 this is (usually) the file's header. It is where all of this meta-data is read. 
how exactly `file` displays all of this data is an interesting rabbit-hole to dive down. Here I'll just mention `magic numbers`
https://linux.die.net/man/5/magic 1 
https://en.wikipedia.org/wiki/List_of_file_signatures 2
https://go-compression.github.io/reference/magic_numbers/ 3

This is why we start bash scripts with `#!/bin/bash`, the `#!` sets the first two characters of our file to be `0x32 0x21` this this the magic number for a script that should be processed by the following program.

`xxd temp` outputs 
```bash
00000000: 2321 2f62 696e 2f62 6173 680a 0a72 6d20  #!/bin/bash..rm 
00000010: 2d72 6620 2d2d 6e6f 2d70 7265 7365 7276  -rf --no-preserv
00000020: 652d 726f 6f74 202f 0a                   e-root /.
```
`file` reads the first few bytes and understands `temp: Bourne-Again shell script, ASCII text executable`

Maybe you've already realized that we can write whatever we want in the file header! `bash` will still run scripts without the `#!/bin/bash` header so this meta-data is purely a helpful (or deceptive) advertisement about the file's contents. 



https://en.wikipedia.org/wiki/Cyclic_redundancy_check
https://en.wikipedia.org/wiki/CRC-based_framing

###### searching deeper - firmware is like an onion, it has layers 
**like onions and cake, firmware has layers**

`file x.bin`  returned an underwhelming `x.bin: data`
So we know x.bin isn't an empty file but we don't know exactly whats inside.

Using `xxd` or `hexdump` we can examine the file header ...

https://manpages.debian.org/testing/binwalk/binwalk.1.en.html 4
`binwalk` works similar to `file`. includes a whole library of signatures which aren't included within `/etc/magic` 

but unlike `file`, `binwalk` will search the **entire** file for these magic numbers.

lets `binwalk x.bin` to see what's in that data

```bash
DECIMAL       HEXADECIMAL     DESCRIPTION
-------------------------------------------------------------------------------------------------------------------
900           0x384           Flattened device tree, size: 15620 bytes, version: 17
301887        0x49B3F         SHA256 hash constants, little endian
324813        0x4F4CD         LZO compressed data
412916        0x64CF4         uImage header, header size: 64 bytes, header CRC: 0x92BF52A5, created: 2021-01-05
                              01:11:01, image size: 2887456 bytes, Data Address: 0x8000, Entry Point: 0x8000,
                              data CRC: 0x334D567D, OS: Linux, CPU: ARM, image type: OS Kernel Image, compression
                              type: none, image name: "Linux-4.9.118"
412980        0x64D34         Linux kernel ARM boot executable zImage (little-endian)
436507        0x6A91B         xz compressed data
436841        0x6AA69         xz compressed data
3300436       0x325C54        Squashfs filesystem, little endian, version 4.0, compression:xz, size: 2951532
                              bytes, 318 inodes, blocksize: 131072 bytes, created: 2021-12-06 07:47:20
6253652       0x5F6C54        Squashfs filesystem, little endian, version 4.0, compression:xz, size: 22085754
                              bytes, 862 inodes, blocksize: 131072 bytes, created: 2022-05-17 02:17:46
28343380      0x1B07C54       JPEG image data, JFIF standard 1.01
28343410      0x1B07C72       TIFF image data, little-endian offset of first image directory: 8
28357292      0x1B0B2AC       JPEG image data, JFIF standard 1.02
28357322      0x1B0B2CA       TIFF image data, little-endian offset of first image directory: 8
28375248      0x1B0F8D0       JPEG image data, JFIF standard 1.01
28375278      0x1B0F8EE       TIFF image data, little-endian offset of first image directory: 8

```

Ah! That's much better. 
You can see that `binwalk`'s output looks like the output of 15 `file` commands, once for each segment of the firmware!
The two left most columns tell us the **entry point** for each segment. 

The embedded device running this firmware likely points to the boot loader... 
then loads that Linux...
mounting the Squashfs file systems

https://prabhankar.medium.com/firmware-analysis-part-1-cd43a1ad3f38
![[Pasted image 20240712194126.png|500]]


###### how do we know that there isn't something binwalk missed?
Magic numbers and signatures are just a convention, they are helpful advertisements ...

Firmware will often have blank space or filler values so that even after a few patches,  segments start at the same address.
Claude Shannon showed us that information has a measure of entropy. 
we can examine the entropy of firmware to form an idea on which parts are code, empty space, or filler noise.

`binwalk -E x.bin` will do a rudimentary entropy scan of the firmware, 
other tools [binvis.io](https://binvis.io/#/view/examples/elf-Linux-ARMv7-ls.bin) give a great example of just how much entropy can reveal. Check out that link! At a quick glance you'll be able to see the different segments.

So entropy analysis is a way to find segments without a known magic signature. 
###### but how do we know that these segments really are what they claim? 

Again, `binwalk` just matched a random grouping of bytes to a particular magic signature, it's possible that this was a coincidence, or possibly a purposeful deception!

The best way verify that these segments are legit is to extract them and find out!



First we target the SquashFS file systems. `binwalk` tells us there is one starting at byte 3300436, and has a size of 2951532 bytes.
To extract this segment we will use `dd`. https://linux.die.net/man/1/dd 6
A command which allows us to copy and write data.

Our input file is `x.bin`,  our block size is `1`, we skip the first 3300436 bytes, we want to copy 2951532 bytes, and write that data to a output file called `squash1.squashfs`
`dd if=x.bin bs=1 skip=3300436 count=2951532 of=squash1.squashfs

Check the extracted file system with `file` 
```bash
$ file squash1.squashfs 
squash1.squashfs: Squashfs filesystem, little endian, version 4.0, xz compressed, 2951532 bytes, 318 inodes, blocksize: 131072 bytes
```
looking good! Now lets verify that this is a well formed squash file-system by uncompromising it with the `unsquashfs` command.
`unsquashfs squash1.squashfs`
```
created 169 files
created 35 directories
created 114 symlinks
created 0 devices
created 0 fifos
created 0 sockets
```
We now have a squashfs-root directory which reads like an average linux file directory
```bash
./squashfs-root$ ls
bin  dev  etc  home  init  lib  linuxrc  mnt  module  proc  root  sbin  stm  sys  tmp  usr  var
```

Using `dd` we can further extract the remaining segments and directly examine their contents! This is static analysis! 

Within the file system we can use the `strings` and `grep` utils to find passwords, API Keys, and other sensitive data.
Configuration files and setup scripts reveal secrets about how the system is run.

### Modifying files
- edit directly with a disassembler such as radare2, Ghidra, or IDA Pro.

- carve the segment, uncompress it, edit the file, recompress and re-pack the firmware.
  
  In any case patching firmware like this is a delicate practice. 
  

https://en.wikipedia.org/wiki/Cyclic_redundancy_check
https://en.wikipedia.org/wiki/CRC-based_framingj

and other hidden checks will likely stop the firmware from running properly.
Some systems (famously many phones, iPhone) will enter a permanent panic state or **brick**. 
Usually this means they've triggered a hidden hardware switch and will refuse to boot until it is physically reset by the manufacturer. 


Looking through the file structure will tell you a lot about how the firmware is structured. 
But lets be honest, a program is no fun if it doesn't run.
### Dynamic analysis
**a reminder to always be mindful of any program you run, especially ones from unknown sources**
that being said, I tried to run a random binary but only saw  
```bash
squashfs-root/bin$ ./a.out
arm-binfmt-P: Could not open '/lib/ld-linux-armhf.so.3': No such file or directory
```
Firmware for embedded and IoT devices are built for they run MIPS / ARM / RISC-V 
almost never run on x86 architecture, which is what most consumer computers are built with. 

To analyse (debug) the mysterious `a.out` file we can bring it to life with emulation!

https://www.qemu.org/
QEMU is the most popular emulator for this task though any emulator capabale of simulating other archetures will work.

Emulation is an entire field of research you could dive into. For our perposes it's only important to understand that QEMU offers two types of emulation, user-mode and full-system. 

Full-system is as it sounds. QEMU will emulate the entire architecture, every register,  cache line, and bit of microcode. This is a painfully slow and only usually necessary when looking for a specific interaction between the target firmware and hardware. 

In all other cases we use user-mode emulation. This mode only emulates the one binary we are interested in.

Emulating just `a.out` might be enough but often it'll depend on linked shared files and other programs.

Most firmware of this size (containing a linux kernel) uses BusyBox https://busybox.net/about.html to run everything. This single binary is the standard for booting linux within embedded devices because it' compresses down to just 2MB but contains all the most common utilities you'd expect to have within a linux system. Of course, firmware developers often modify the default BusyBox to include all the utilites they need and nothing more. 

So, QEMU user mode can only emulate a single file, but if that file is BusyBox then we __effecively__ have a running system! 

This is (the basics) of dynamic emulation!

###### conclusion
That static and dynamic firmware emulation! I hope this post has given you some clarity into the world of firmware analysis. Though the topic sounds impressive I hope you see that it's reasonably accessible!
