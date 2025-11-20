---
layout: post
title: "Tools"
permalink: /tools/
---
This page showcases the open-source tools developed. For detailed information, please visit the links provided for each tool.

# Volafox - [link](https://github.com/n0fate/volafox)
**Volafox** is a memory forensics tool designed to extract evidence of user activity and malicious behavior from physical memory images of macOS systems. Requiring only a raw memory dump as input, it automatically detects the operating system version and delivers superior analysis speed compared to similar utilities.

Volafox retrieves critical volatile data from the OS, including artifacts of memory-manipulating malware and essential forensic evidence such as encryption keys.

* Key Features
  * Volatile System Information
    * Hostname identification (`hostname`)
    * Boot debug messages (`dmesg`)
    * Process listing (`ps`)
    * Network session listing (`netstat`)
    * Open file listing (`lsof`)
    * Mounted file systems (`mount`)
    * System call & Mach trap tables
    * TrustedBSD MAC Framework status
    * Bash history extraction
    * sysctl information
    * EFI information
  * Credential Extraction
    * Keychain master key candidates (compatible with `chainbreaker` for decryption)
  * Rootkit & Hooking Detection
    * Kdebug, Kauth, BSM, and FBT hooking detection
    * System call & Mach trap table hooking detection
    * I/O function hooking detection
  * Anomaly & Stealth Detection
    * KEXT scanning (Hidden kernel extension detection)
    * Hidden process detection (Task analysis)


# chainbreaker - [link](https://github.com/n0fate/chainbreaker)
**Chainbreaker** is a specialized utility for decrypting and parsing macOS Keychain files. It is capable of utilizing master keys extracted via `volafox` to perform offline decryption. By requiring only the target Keychain file and a valid credential (encryption key or password), Chainbreaker efficiently recovers stored secrets and presents them to the analyst.

# Apple Continuity Analysis Tools
This collection features specialized tools developed to investigate artifacts related to Apple Continuity.

## iChainbreaker - [link](https://github.com/n0fate/ichainbreaker)
**iChainbreaker** focuses on decrypting iCloud Keychain data, facilitating the retrieval of credentials synchronized from iOS devices. This tool allows forensic examiners to access and decrypt confidential information that originated on an iPhone and was propagated through the iCloud Keychain service.

## Apple Call History Decryptor Tool - [link](https://github.com/n0fate/OS-X-Continuity/tree/master/Call%20History%20Decryptor)
The **Apple Call History Decryptor** is engineered to decrypt call logs, including obscured phone numbers, that are synchronized across the cloud between iOS and macOS devices. It enables the forensic extraction of iPhone call history artifacts found on macOS. _Please note that successful decryption requires both devices to be associated with the same Apple ID._
