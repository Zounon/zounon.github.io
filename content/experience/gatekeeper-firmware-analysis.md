+++
title = "Firmware Analysis & QA Tooling @ Gatekeeper"
date = 2022-12-01
start = "05-2022"
end = "12-2022"
description = """
When manual testing is the norm I saw many opportunties to improve the efficeny and quality of QA's work. I built tools (which are still in use today) which 'just work'. Allowing any QA member to be more productive; instead of learning how to work with yet another tool.

I also analysed firmeware and reverse engineered netowrk protocols to better understand - and therefore better test - the embedded devices we were working on. 
"""
+++

Role: Software Systems Engineer  
Location: Abbotsford, BC  
Team: Gatekeeper Systems Inc.

## Overview
Built tooling to eliminate manual testing bottlenecks while also hardening the QA workflow with firmware analysis and anomaly detection.

## What I shipped
- Automated 150 manual tests by emulating HTTPS device traffic using scapy.
- Reconstructed firmware images with binwalk and analyzed behavior with radare2 and Ghidra.
- Prototyped user-space emulation with QEMU to validate findings.
- Delivered a concurrent network/log anomaly tool now standard in the QA toolkit.

## Impact
- Removed the dependency on physical hardware for core regression tests.
- Improved visibility into device behavior and potential security issues.

## Stack
Python, scapy, Wireshark, tcpdump, binwalk, radare2, Ghidra, QEMU
