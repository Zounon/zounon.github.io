+++
date = '2026-02-01'
draft = true
title = 'Hacking Firmware for Fun and Profit'
+++

My first phone had been jailbroken, ran multiple OS's and had a few hardware swaps; that phone was defintely *mine*.
My current phone came preinstalled with Facebook and YouTube and there is **no way to uninstall them**.
And my laptop asks if I want to update now or try again tomorrow; that's not *my* laptop anymore - microsoft owns it.
If I can't modify my computers, they are no longer mine. In this blog post I walk through how anyone can read and modify the firmware in their device to regain ownership.
[Read More](/experience/modifying-firmware-for-fun-and-profit/)

Between a computer's hardware and software sits its firmware. Firmware is like software, except as the name suggests it is not as flexible and runs closer to the hardware. You interact with the software layer by tapping the camera app. The app cannot access the camera directly; it asks the firmware to stream camera data its way, and the firmware talks to the camera.

These layers (software, firmware, hardware) abstract away the intricacies of each bit of hardware. Many OSes can run on your computer because they all just ask, "open the webcam please," and it is the firmware's job to handle the specifics. But firmware can also decide that an OS is "compromised" and refuse to boot. At first glance this sounds like a nice security feature. The problem is that companies are often more incentivized by money than customer safety.

Xbox and PlayStation systems will brick if they detect you tried to install your own software. Phones similarly will not run if you have gained root access. So, let's make these devices ours again.

Similar to the a.out file you get after compiling C code, firmware is compiled to run, not to be read. This post gives an overview of static and dynamic firmware analysis and the kinds of workflows used to understand and modify firmware safely.

## What are we working with?

Assume you have the firmware file in hand. For popular devices, firmware can often be downloaded from vendor servers. In other cases, it must be extracted from the target device.

Start by figuring out what kind of data you are dealing with. The `file` command reads the file header (or first 1024 bytes if it looks like ASCII) and tries to identify a format based on magic numbers in `/etc/magic`.

Example outputs (three different firmware files):

- `f.bin: firmware 1042 v1 TP-LINK Technologies ver. 1.0, version 3.15.7, 8258048 bytes or less, _ at 0x200 998400 bytes , _ at 0x100000 7077888 bytes`
- `s.bin: u-boot legacy uImage, SPI Flash Image, Linux/MIPS, Standalone Program (Not compressed), 111116 bytes, Load Address: 0X80200000, Entry Point: 0X80200000, Header CRC: 0X80EF7587, Data CRC: 0XCD95F789`
- `x.bin: data`

The first firmware claims to be for a TP-LINK router and gives a model, version, and layout. The second indicates a u-boot image plus a Linux/MIPS payload. The third just says "data." This tells us the `file` command is helpful but limited.

Magic numbers are just conventions. You can write whatever you want in the header and some tools will believe it. That is why we look deeper.

## Static analysis

Static analysis is safe but can be slow to understand. The goal is to identify the segments inside a firmware blob.

### Binwalk: peel the onion

`binwalk` works like `file` but scans the entire file for known signatures, not just the header.

Example output (truncated):

```
DECIMAL       HEXADECIMAL     DESCRIPTION
-------------------------------------------------------------------------------------------------------------------
900           0x384           Flattened device tree, size: 15620 bytes, version: 17
301887        0x49B3F         SHA256 hash constants, little endian
324813        0x4F4CD         LZO compressed data
412916        0x64CF4         uImage header, header size: 64 bytes, header CRC: 0x92BF52A5, created: 2021-01-05
                              01:11:01, image size: 2887456 bytes, Data Address: 0x8000, Entry Point: 0x8000,
                              data CRC: 0x334D567D, OS: Linux, CPU: ARM, image type: OS Kernel Image
412980        0x64D34         Linux kernel ARM boot executable zImage (little-endian)
3300436       0x325C54        Squashfs filesystem, little endian, version 4.0, compression:xz, size: 2951532
6253652       0x5F6C54        Squashfs filesystem, little endian, version 4.0, compression:xz, size: 22085754
```

Notice how `binwalk` gives you multiple embedded segments, with entry points and sizes. This is effectively many `file` checks rolled into one.

### Entropy scanning

Magic signatures can be missing or deceptive. Entropy scanning helps you identify sections that look like compressed data, code, or filler. `binwalk -E x.bin` does a rudimentary entropy scan. Tools like binvis.io are great for visualizing segment boundaries.

### Extracting segments with dd

Once you have offsets and sizes, you can extract segments directly. For example, if `binwalk` says a SquashFS filesystem starts at byte 3300436 and is 2951532 bytes long:

```
dd if=x.bin bs=1 skip=3300436 count=2951532 of=squash1.squashfs
```

Verify it:

```
file squash1.squashfs
```

Then unpack it:

```
unsquashfs squash1.squashfs
```

At that point you have a regular-looking filesystem and can start reading configs, scripts, and binaries. This is where `strings` and `grep` can reveal passwords, API keys, and boot logic.

## Modifying files

There are two common approaches:

- Edit directly in a disassembler like Ghidra, radare2, or IDA Pro.
- Extract a segment, decompress, edit, recompress, and then repack the firmware.

Patching firmware is delicate. Many images include CRCs or other checks that will prevent boot if anything changes. Some systems (famously phones) can hard-brick if they detect tampering.

## Dynamic analysis

Static analysis gives you structure. Dynamic analysis gives you behavior.

Most embedded firmware targets MIPS, ARM, or RISC-V, which do not run natively on x86. That is where emulation comes in.

### QEMU

QEMU is the standard emulator for this task. It offers two relevant modes:

- Full-system emulation: simulates the entire architecture (slow, but accurate).
- User-mode emulation: runs a single binary (fast, often enough).

If the binary depends on a full userland, user-mode can still work by emulating BusyBox. BusyBox is commonly used in embedded Linux because it compresses many utilities into a single binary.

If you can run BusyBox under QEMU user-mode, you effectively have a working environment for many tasks without full-system emulation.

## Conclusion

That is the high-level overview of static and dynamic firmware analysis. The tooling can sound intimidating, but the workflow is approachable once you break it into steps: identify, extract, verify, and then experiment. If you want to go deeper, I recommend starting with sample firmware images and practicing extraction and repacking workflows.

## References to explore

- HackTricks firmware analysis notes
- `file`, `binwalk`, `dd`, and `unsquashfs` man pages
- Ghidra, radare2, and IDA Pro
- QEMU and BusyBox
