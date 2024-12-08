---
layout: post
title: "Tools"
permalink: /tools/
---
This page introduces the open-source tools that have been developed. Detailed information can be accessed by clicking the links next to each tool.

# volafox - [link](https://github.com/n0fate/volafox)
The `volafox` is a memory analysis tool designed to extract user or malicious behavior from physical memory images of macOS systems. This tool takes only a memory image as input, automatically identifies the operating system version, and performs analysis at a faster speed compared to other tools with similar functionality. It extracts volatile information from the operating system, including details related to malicious activities that manipulate memory information, as well as critical data required for digital forensic investigations, such as encryption keys. The detailed features of the tool are as follows:
* volatile information
	* host name(`hostname`)
	* debug message at boot (`dmesg`)
	* list of process(`ps`)
	* list of network sessions (`netstat`)
	* list of open files (`lsof`)
	* mounted filesystem(`mount`)
	* system call table
	* mach trap table
	* TrustedBSD Mac Framework
	* bash history(`history`)
	* sysctl
	* EFI information
* encryption keys
	* master key candidates for keychain decryption with `chainbreaker`
* hooking detection
	* Kdebug hooking detection
	* Kauth hooking detection
	* BSM hooking detection
	* FBT hooking detection
	* System call hooking detection
	* Mach trap table hooking detection
	* I/O Function hooking detection
* Resource hiding detection
	* KEXT Scanning
	* Process hiding detection(tasks)

# chainbreaker - [link](https://github.com/n0fate/chainbreaker)
The `chainbreaker` is a tool that decrypts and extracts macOS Keychain information. It can perform decryption by using the master key extracted through `volafox` as input. With only the Keychain file and an encryption key (or password), it decrypts confidential information stored in the Keychain and outputs it to the user.

# Tools for Apple Continuity Analysis
This section summarizes small-scale tools developed in relation to Apple Continuity features.

## ichainbreaker - [link](https://github.com/n0fate/ichainbreaker)
The `ichainbreaker` is a tool designed to decrypt information shared via iCloud Keychain, enabling the retrieval of Keychain data shared through an iPhone. By using this tool, confidential information stored on the iPhone and shared through iCloud Keychain can be decrypted and retrieved.

## The Apple Call History Decryptor Tool - [link](https://github.com/n0fate/OS-X-Continuity/tree/master/Call%20History%20Decryptor)
The `Apple Call History Decryption Tool` is designed to decrypt encrypted phone number information from call history synchronized via the cloud between iPhone and Mac. Using this tool, call history information from an iPhone can be extracted on macOS. To retrieve meaningful information, the user must be logged into both the iPhone and Mac with the same Apple ID
