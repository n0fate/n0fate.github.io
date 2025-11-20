---
layout: post
title: Methods for Retrieving Kernel Page Table Addresses in macOS Sequoia (x86_64) (English)
date: 2025-11-20 13:39:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- macOS Forensics
tags: []
author: "n0fate"
comments: true
---
# Introduction

Kernel privileges represent the ultimate level of control over an operating system. Attackers strive to obtain these privileges by first exploiting application vulnerabilities to gain user access, and subsequently performing privilege escalation to seize control of the entire system.

Once an attacker gains these privileges, they can patch code in kernel memory to inject malicious payloads. Since data processed by applications passes through the kernel during I/O operations, intercepting data entering the kernel is a critical objective for attackers.

In response to these threats, Apple integrated advanced security features, including **KTRR** and **SKVA**, into macOS Sequoia. These enhancements, however, have rendered legacy techniques for kernel page table traversal obsolete. This article explores the underlying causes of this disruption and proposes a verified solution based on recent research.

# Background

## KTRR (Kernel Text Read-Only Region)

In previous versions of macOS, the EFI bootloader applied ASLR to the kernel, mapped it to physical memory, configured the kernel's page tables, and set the CR3 register to point to them.

KTRR is a security feature available on Apple's A10 chips and later, and it was introduced to macOS following the transition to M1 chips. In terms of actual code implementation, it appears to have been applied since macOS 10.14.

The goal of KTRR is to make the kernel's code region read-only. The bootloader in EFI (`boot.efi`) configures the kernel's page tables *before* loading the kernel itself. The kernel image is loaded sequentially into physical memory, and the CR3 register is set to point to the page table address configured by the bootloader.

Through this method, the TEXT region of the kernel image is rendered read-only. Any attempt to forcibly modify these permissions results in a page fault.

## SKVA (Static Kernel Virtual Addresses)

Modern code often relies heavily on the heap for dynamic allocation and deallocation. Attackers can exploit vulnerabilities to manipulate heap memory, potentially overwriting function pointers or global variables located in static data regions.

Static Kernel Virtual Addresses (SKVA) is a technology applied in macOS Sequoia that separates static data from dynamic heap areas and inserts guard pages between them. By adding unmapped guard pages before and after the static kernel virtual memory region, any attempt to overflow from the heap into SKVA triggers a page fault.

Additionally, KASLR is applied separately to the static kernel virtual memory region and the heap memory. This ensures that even if an attacker infers the KASLR layout of the heap, they cannot determine the KASLR layout of the SKVA.

# Previous Kernel Page Table Creation Procedure

Previously, kernel pages were set up such that `IdlePML4` pointed to the kernel page table. Memory analysis tools could calculate the physical memory address of kernel data via this pointer. However, since `IdlePML4` itself was a virtual memory page influenced by other page tables, a process was required to calculate the correct physical memory address for this pointer.

When a computer boots, the bootloader loads the Kernel Cache into memory. Since the Kernel Cache is intended to improve performance, it is configured as a 1:1 mapping to physical memory without passing through page tables.

At the point where the kernel entry point code, `_start`, begins execution, the CR3 register is configured to point to a table already set up within the Data Section of the kernel image. The `BootPML4` symbol pointer points to this address. The reason for constructing such a page table is that while the kernel's virtual memory addresses are high (0x7ffff...), the actual load addresses are low (0x00b2...).

Once the page tables are configured, the kernel performs full hardware initialization. During the process of allocating memory pages via PMAP (Physical Map), new page tables are constructed. The page table is copied to the address of the kernel's `IdlePML4` symbol pointer, and areas such as heap/stack memory for process operations are configured in this new page table. Finally, this address is assigned to the CR3 register.

> **Important:** Since the symbol pointer for `IdlePML4` is also mapped to a physical memory address via the `BootPML4` page table, one must trace back from `BootPML4` to correctly reconstruct the page tables.

Once the kernel's `pmap_init` function completes, the `kernel_task` process (PID 0) is configured to run its first thread under the `IdlePML4` environment.

# Changed Kernel Page Table Creation Procedure

In macOS Sequoia, the creation of page tables has changed due to KTRR, as described earlier. If you attempt to construct the page table by tracing `IdlePML4` as before, it will point to a completely incorrect address. Fundamentally, In macOS Sequoia, the page at the `BootPML4` address appears to be NULL, making it look unused. Why does this happen?

Starting with macOS Sequoia, consistent with the KTRR description, the bootloader configures the `BootPML4` page table and uses it to configure `IdlePML4`. To elevate the Intel boot process to a security level on Apple Silicon, Bootloader eanbles the `CR0.WP`, and the kernel page table is set to read-only(**Write XOR Execute**). **Subsequently, the `BootPML4` page table is removed.** This is why the traditional method based on `BootPML4` is no longer possible.

# Methods for Locating Kernel Memory Page Tables

As explained, the changes in kernel page table configuration made it impossible to trace page tables based on `BootPML4`. Based on an analysis of the XNU kernel source code (`xnu-11417.140.69`) and GPT assistance, I identified three methods to locate the page tables.

## 1. Finding via `KDP_JTAG_COREDUMP_T`

Among kernel structures, `kdp_jtag_coredump_t` contains information for providing core dumps via JTAG. The `kernel_pmap_pml4` field within this structure holds the page table information after booting.

```c
/* data required for JTAG extraction of coredump */
typedef struct _kdp_jtag_coredump_t {
    uint64_t signature;
    uint64_t version;
    uint64_t kernel_map_start;
    uint64_t kernel_map_end;
    uint64_t kernel_pmap_pml4;
    uint64_t pmap_memory_regions;
    uint64_t pmap_memory_region_count;
    uint64_t pmap_memory_region_t_size;
    uint64_t physmap_base;
} kdp_jtag_coredump_t;
```

The address of this structure is stored in the `lgKdpJtagCoredumpAddr` field of the lowGlo structure.

```c
/*
 * Don't change these structures unless you change the corresponding assembly code
 * which is in lowmem_vectors.s
 */

#pragma pack(8)         /* Make sure the structure stays as we defined it */
typedef struct lowglo {
    unsigned char   lgVerCode[8];           /* 0xffffff8000002000 System verification code */
    uint64_t        lgZero;                 /* 0xffffff8000002008 Double constant 0 */
    uint64_t        lgStext;                /* 0xffffff8000002010 Start of kernel text */
    uint64_t        lgLayoutMajorVersion;   /* 0xffffff8000002018 Low globals layout major version */
    uint64_t        lgLayoutMinorVersion;   /* 0xffffff8000002020 Low globals layout minor version */
    uint64_t        lgRsv028;               /* 0xffffff8000002028 Reserved */
    uint64_t        lgVersion;              /* 0xffffff8000002030 Pointer to kernel version string */
    uint64_t        lgCompressorBufferAddr; /* 0xffffff8000002038 Pointer to compressor buffer */
    uint64_t        lgCompressorSizeAddr;   /* 0xffffff8000002040 Pointer to size of compressor buffer */
    uint64_t        lgRsv038[278];          /* 0xffffff8000002048 Reserved */
    uint64_t        lgKmodptr;              /* 0xffffff80000028f8 Pointer to kmod, debugging aid */
    uint64_t        lgTransOff;             /* 0xffffff8000002900 Pointer to kdp_trans_off, debugging aid */
    uint64_t        lgReadIO;               /* 0xffffff8000002908 Pointer to kdp_read_io, debugging aid */
    uint64_t        lgDevSlot1;             /* 0xffffff8000002910 For developer use */
    uint64_t        lgDevSlot2;             /* 0xffffff8000002918 For developer use */
    uint64_t        lgOSVersion;            /* 0xffffff8000002920 Pointer to OS version string */
    uint64_t        lgRebootFlag;           /* 0xffffff8000002928 Pointer to debugger reboot trigger */
    uint64_t        lgManualPktAddr;        /* 0xffffff8000002930 Pointer to manual packet structure */
    uint64_t        lgKdpJtagCoredumpAddr;  /* 0xffffff8000002938 Pointer to kdp_jtag_coredump_t structure */

    uint64_t        lgRsv940[216];          /* 0xffffff8000002940 Reserved - push to 1 page */
} lowglo;
```

&nbsp;

o obtain the value of kernel_pmap_pml4, follow these steps:

1. Locate the `lowGlo` structure using the existing Catfish signature.
2. Calculate the kernel ASLR offset by finding the difference between the lowGlo kernel symbol address (& `LOW_4GB_MASK`) and the physical memory address where the signature is located.
3. Add the calculated kernel ASLR offset to `lgKdpJtagCoredumpAddr` to find the location of `kdp_jtag_coredump_t`.
4. Verify if the structure signature matches `PMUDEROC` (reverse of COREDUMP).
5. Construct memory pages based on the `kernel_pmap_pml4` value to analyze kernel data structures.

## Finding via `CPU_DATA_T` Structure

The cpu_data_t structure contains CPU information. This method involves reading the `cpu_kernel_cr3` value within this structure to configure the page tables. By following `cpu_data_ptr`, you can locate pointers corresponding to the number of cores. By tracing one of these, you can construct the pages using the lower 4 bytes of `cpu_task_cr3` (at offset `0xE8`), `cpu_kernel_cr3` (next), or `cpu_ucr3` as an absolute value.

```c
struct cpu_data_t {
    ...
    volatile addr64_t       cpu_task_cr3; // 0xe8
    addr64_t                cpu_kernel_cr3; // 0xf0
    volatile addr64_t       cpu_ucr3;	// 0xf8
    ...
}
```

&nbsp;

## Finding via `KERNEL_PMAP_STORE` Structure
This method involves interpreting the structure that holds information related to the Kernel PMAP(Physical Map). The structure is defined as follows:

```C
// osfmk/i386/pmap.h
struct pmap {
    lck_rw_t        pmap_rwl __attribute((aligned(64)));
    pmap_paddr_t    pm_cr3 __attribute((aligned(64))); /* Kernel+user shared PML4 physical*/
    pmap_paddr_t    pm_ucr3;        /* Mirrored user PML4 physical */
    pml4_entry_t    *pm_pml4;       /* VKA of top level */
    pml4_entry_t    *pm_upml4;      /* Shadow VKA of top level */
    pmap_paddr_t    pm_eptp;        /* EPTP */
    ...
}
```

You need to track the 4th value of this structure. In actual memory, four pointers starting at offset 0x40 contain these values (likely `pm_cr3`, `pm_ucr3`, `pm_pml4` (& `LOW_4GB_MASK`), and pm_umpl4 (&`LOW_4GB_MASK`) ). This implies the size of the lck_rw_t structure is 0x40 bytes, though this requires verification.

# Experimental Result
Since the existing code already locates the lowGlo structure via Catfish, I updated the code to reflect the page table discovery method based on the `KDP_JTAG_COREDUMP_T` structure.

After configuring the macOS Sequoia kernel page tables this way and running the `system_profiler` plugin, the results are successfully displayed:

```shell
n0fate@nMacBook-Pro14 volafox % python vol.py -i sequoia_sample/macOS\ 10.15-1.vmem -o system_profiler -v
[+] Memory Image: sequoia_sample/macOS 10.15-1.vmem
[+] Command: system_profiler
[+] Get Memory Image Information
 [-] Difference(Catfish Signature): 0
 [-] Valid Mac Linear File Format
 [-] 64-bit memory image
 [-] Build Version in Memory : 24F74
[+] Open overlay file 'overlays/24F74x64.overlay'
[+] Finding Kernel Base Address (KASLR)
 [-] lowGlo Symbol Address : 0xc68000
[+] It's Sequoia!!
 [-] Kernel Base Address : 0xdce4000
 [-] VM Slide Value : 0xdf1c000
 [+] BootPDPT symbol address : 0xe022000
 [+] BootPML4 symbol address : 0xe021000
 [-] Is a Linear Format
 Kernel PMAP: 0xffffff800ebb9340
[+] Loading Intel IA-32e(PAE Enabled) Paging Table
[+] Mac OS X Basic Information
 [-] Darwin kernel Build Number: 24F74
 [-] Darwin Kernel Major Version: 24
 [-] Darwin Kernel Minor Version: 5
 [-] Number of Physical CPUs: 8
 [-] Size of memory in bytes: 2147483648 bytes
 [-] Size of physical memory: 4294967296 bytes
 [-] Number of physical CPUs now available: 8
 [-] Max number of physical CPUs now possible: 8
 [-] Number of logical CPUs now available: 8
 [-] Max number of logical CPUs now possible: 8
 [-] Last Hibernated Sleep Time: Thu Jan 01 00:00:00 1970 (GMT +0)
 [-] Last Hibernated Wake Time: Thu Jan 01 00:00:00 1970 (GMT +0)
```

We are now ready to perform memory analysis. (This is just the beginning...)


# References

1.  https://github.com/googleprojectzero/ktrw
2.  https://googleprojectzero.blogspot.com/2019/10/ktrw-journey-to-build-debuggable-iphone.html
3.  XNU Kernel 11417.140.69 (https://opensource.apple.com/releases)

Translated by Gemini 3 ;)
