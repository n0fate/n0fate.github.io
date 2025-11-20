---
layout: post
title: macOS 세콰이어에서 커널 페이지 테이블 주소를 찾는 방법(x86_64)
date: 2025-11-20 11:16:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- macOS Forensics
tags: []
author: "n0fate"
comments: true
---
# 서론

커널 권한은 운영체제를 통제할 수 있는 가장 강력한 권한이다. 공격자는 이 권한을 얻기 위하여 애플리케이션의 취약점을 이용하여 사용자 권한을 획득한 후, 권한 상승을 수행하여 전체 시스템을 통제하고자 노력한다.

권한을 획득한 공격자는 커널 메모리 상의 코드를 패치하여 공격자가 원하는 코드를 주입할 수 있다. 애플리케이션에서 처리되는 데이터는 입/출력 단계에서 커널을 거치기 때문에 커널로 들어오는 데이터를 획득하는 것은 공격자에게 중요한 행동 중 하나가 된다.

이러한 공격을 방어하기 위하여 애플은 macOS 세콰이어에 KTRR, SKVA과 같은 기술을 적용하였다. 그리고 이러한 기술의 적용으로 인하여 기존의 macOS 커널 페이지 테이블 탐색 방법이 더이상 적용되지 않는 문제가 발견되었다. 이번 글에서는 이러한 문제가 발생한 원인과 이에 대한 해결 방안을 연구한 결과를 작성하였다.

# Background

## KTRR(Kernel Text Read-Only Region)

기존 macOS는 EFI의 부트로더가 커널에 ASLR을 적용하여 물리 메모리에 맵핑하고 맵핑된 커널의 페이지 테이블을 구성하여 CR3 레지스터가 이를 가리키도록 하였다.

KTRR는 애플의 A10칩부터 사용가능한 기능으로 macOS에 M1칩 적용 이후 적용된 보안 기능이다. 실제 코드에 적용된 시점은 macOS 10.14 이후로 확인된다.

커널의 코드 영역을 읽기 전용으로 만들기 위한 방법이다. EFI에 있는 부트로더(`boot.efi`)는 커널을 로딩하기 전에 커널의 페이지 테이블을 구성하고 커널을 로드한다. 커널 이미지는 물리 메모리에 시퀀셜(sequential)하게 로드 되고, CR3 레지스터에는 부트로더가 구성한 커널의 페이지 테이블의 주소를 가리키도록 한다.

이 방식으로 커널 이미지의 TEXT 영역은 쓰기 권한이 제거된 상태가된다. 권한을 강제로 수정하는 경우 페이지 폴트(page fault)가 발생한다.

## SKVA (Static Kernel Virtual Addresses)

최근 많은 코드가 힙을 이용하여 동적 할당/해제하여 구현되고 있다. 공격자는 취약점을 이용하여 힙 메모리를 제어하면서 정적 데이터의 함수 포인터나 전역 변수를 덮어씌울 수 있는 문제점이 발생할 수 있다.

정적 커널 가상 메모리(SKVA)는 macOS Seuoia에서 적용된 기술로 정적 데이터와 동적 힙 영역을 분리하고 그 사이에 가드 페이지를 삽입하는 방법이다. 정적 커널 가상 메모리 영역의 앞 뒤에 맵핑되지 않은 가드 페이지를 추가하여 힙 메모리를 덮어쓰는 과정에서 SKVA에 접근 시 페이지 폴트가 발생하도록 한다.

추가적으로 KASLR을 정적 커널 가상 메모리 영역과 힙 메모리에 따로 지정하여 힙 메모리의 KASLR을 유추하더라도 SKVA의 KASLR을 알아낼 수 없게 한다.

# 기존 커널 페이지 테이블 생성 절차

기존 커널 페이지는 IdlePML4가 커널 페이지 테이블을 가리키도록 되어 있었다. 메모리 분석 도구는 이 포인터를 통해 커널 데이터의 물리 메모리 주소를 산출할 수 있었다. 문제는 IdlePML4 또한 다른 페이지 테이블의 영향을 받는 가상 메모리 페이지였기에 해당 포인터의 올바른 물리 메모리 주소를 산출하기 위한 과정이 필요하였다.

컴퓨터 부팅을 하면 부트로더는 커널 캐시(Kernel Cache)를 메모리에 로드한다. 커널 캐시는 말그대로 성능을 높이기 위한 목적으로 활용되므로 페이지 테이블을 거치지 않고 물리 메모리에 1:1 맵핑을 하는 형태로 구성한다.

커널 진입점 코드인 \_start 기계어가 시작되는 시점에 커널 이미지 내의 데이터 섹션 내에 이미 구성되어 있는 테이블에 CR3 레지스터가 가리키도록 구성한다. 그리고 이 주소를 BootPML4 심볼 포인터가 가리킨다. 이러한 페이지 테이블을 구성하는 이유는 커널의 가상 메모리 주소는 높은 주소 (0x7ffff...)을 가지는 반면에 실제 로드되는 주소는 낮은 주소(0x00b2...)을 가지기 때문이다.

페이지 테이블이 구성 되었으므로 커널은 본격적인 하드웨어 초기화 작업을 수행하고 PMAP(Physical Map)을 메모리 페이지를 할당하는 과정에서 새로운 페이지 테이블이 구성된다. 커널의 IdlePML4 심볼 포인터 주소에 페이지 테이블을 복사하고 프로세스 동작을 위한 힙/스택 메모리 등의 영역을 새로운 페이지 테이블에 구성한다. 그리고 이 주소를 CR3 레지스터에 할당한다.

> 여기서 중요한 것은 IdlePML4의 심볼 포인터도 BootPML4의 페이지 테이블을 통해 물리 메모리 주소가 맵핑되어 있으므로 BootPML4부터 추적하여야 올바른 페이지 테이블 구성이 가능하다.

커널의 `pmap_init` 함수 호출이 종료되면 PID 0을 가지는 `kernel_task` 프로세스가 첫 번째 스레드를 IdlePML4 환경하에 돌아가도록 구성한다.

# 변경된 커널 페이지 테이블 생성 절차

세콰이어 운영체제는 앞 서 설명한 KTRR로 인해 페이지 테이블의 생성이 달라졌다. 기존과 동일하게 IdlePML4의 값을 추적해서 페이지 테이블을 구성하려고하면, 전혀 엉뚱한 주소를 가리키게 된다. 근본적으로 세콰이어는 BootPML4를 사용하지 않는다. 그러면 어떻게 구성할까?

macOS 세콰이어 버전부터는 KTRR의 설명과 같이 부트로더가 BootPML4의 페이지 테이블을 구성하고 이 테이블을 통해 IdlePML4를 구성한다. 그리고 인텔의 부팅 과정을 애플 실리콘과 유사한 수준으로 끌어올리기 위하여 CPU의 CR0.WP(Write-Protection)을 활성화하고 IdlePML4 페이지 테이블 내 페이지 정보를 읽기 전용으로 구성한다. 그리고 **BootPML4의 페이지 테이블을 제거한다.** 이로 인해 기존의 BootPML4 기반의 페이지 테이블 구성이 불가했던 것이다.

# 커널 메모리 페이지 테이블 탐색 방법

앞서 설명한 것과 같이 변경된 커널 페이지 테이블 구성으로 인하여 더이상 BootPML4에 기반하여 페이지 테이블을 탐색하는 것이 불가능해져서 이에 대한 방안이 필요하였다. XNU 커널 소스코드(11417.140.69)를 기반으로 GPT와 분석을 통해 3가지 페이지 테이블 식별 방안을 찾았다.

## KDP_JTAG_COREDUMP_T 를 이용한 페이지 테이블 찾기

커널 구조체 중 kdp_jtag_coredump_t 는 JTAG을 통한 코어 덤프를 제공하기 위한 정보를 담고 있다. 이 중 kernel_pmap_pml4 필드는 부팅 후 페이지 테이블 정보를 담고 있다.

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

이 구조체는 lowGlo 구조체 중 `lgKdpJtagCoredumpAddr` 필드에 주소가 저장되어 있다.

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

즉, 우리가 `kernel_pmap_pml4`의 값을 획득하기 위해서는 다음과 같은 순서를 따른다.

1.  기존의 Catfish 시그니처를 이용하여 lowGlo 구조체를 찾는다.
2.  lowGlo의 커널 심볼 주소(와 `LOW_4GB_MASK`를 AND 연산한 값)와 시그니처가 위치한 물리 메모리 주소의 차이를 계산하여 kernel aslr offset을 산출한다.
3.  산출된 kernel aslr offset과 `lgKdpJtagCoredumpAddr`를 더한 값의 위치가 `kdp_jtag_coredump_t` 의 위치가 된다.
4.  구조체의 시그니처 `PMUDEROP` (COREDUMP의 역순)인지 확인한다.
5.  `kernel_pmap_pml4` 값을 토대로 메모리 페이지를 구성하여 커널 자료형을 분석한다.

## CPU_DATA_T 구조체에서 획득

cpu_data_t 구조체는 시피유 정보를 담고 있는 구조체이다. 이 구조체 내의 cpu_kernel_cr3 값을 읽어서 페이지 테이블을 구성하는 방법이다. cpu_data_ptr을 찾아가면 코어 갯수만큼 포인터가 위치하고, 이 중 하나를 추적하여 0xE8에 위치한 cpu_task_cr3 또는 그 다음의 cpu_kernel_cr3, 또는 cpu_ucr3  값의 하위 4바이트를 절대 값으로 하여 페이지를 구성할 수 있다.

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

## KERNEL_PMAP_STORE 구조체에서 획득

커널 PMAP과 관련된 정보를 가지는 구조체를 해석하여 획득하는 방법이다. 구조체를 보면 아래와 같이 구성된다.

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

해당 구조체의 4번째 값을 추적해야 한다. 실제 메모리 상으로는 해당 포인터의 0x40의 4개의 포인터가 해당 값을 가진다. (느낌상 pm_cr3, pm_ucr3, pm_pml4(가상 주소라 하위 4바이트만 사용), pm_umpl4(가상 주소라 하위 4바이트만 사용) 으로 보임). 즉, lck_rw_t 구조체의 크기가 0x40 바이트라는건데, 이건 확인이 필요함.

# 실험 결과

기존 코드가 이미 Catfish를 통해 lowGlo 구조체를 찾아내므로 해당 구조체 기반의 페이지 테이블 탐색 방법인KDP_JTAG_COREDUMP_T 구조체의 정보를 기반으로 코드 반영하였다.

이렇게 macOS 세콰이어의 커널 페이지 테이블을 구성하고 system_profiler 플러그인을 수행해봤더니 잘 나오는 것을 볼 수 있다.

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

이제, 메모리 분석을 수행하는 기본 준비가 된 것이다. (이제부터 시작임..)


# 참조

1.  https://github.com/googleprojectzero/ktrw
2.  https://googleprojectzero.blogspot.com/2019/10/ktrw-journey-to-build-debuggable-iphone.html
3.  XNU Kernel 11417.140.69 (https://opensource.apple.com/releases)
