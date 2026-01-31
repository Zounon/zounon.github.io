+++
title = "Firmware Analysis & QA Tooling @ Gatekeeper"
date = 2022-12-01
start = "05-2022"
end = "12-2022"
description = "Replaced manual tests with protocol emulation and added firmware analysis workflows."
tags = ["firmware", "security", "qa"]
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
