---
layout: post
title: Mac Response Forensics(MacResponseLE)의 메모리 이미징 모듈
date: 2013-07-21 15:10:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
author: "n0fate"
---
<p><!--?xml version="1.0" encoding="UTF-8" standalone="no"?--> 2010년 경에 <a href="http://macresponseforensics.com/">Mac Response Forensics</a> 는 프레임워크가 공개되었다. 이 프레임워크는 제목에서 기술한 메모리 이미징 기능 뿐만 아니라 라이브 데이터 수집 기능까지 내장하고 있는 오픈 소스 프로젝트이다. 이 프레임워크 자체가 대충 만든 것이 아니라 플러그인 추가가 쉽게 구성되어 있고, 구글 코드와 함께 많은 오픈소스 개발자가 프로젝트를 진행하는 <a href="https://github.com/assuredinformationsecurity/MacResponse-Forensics">GitHub에 등록</a>된 것을 보고 필자는 다양한 플러그인이 추가되며 지속적인 업데이트를 수행할 줄 알았다. 하지만 프로젝트를 최초 커밋한 그룹도 더이상 개발에 관여하지 않으면서, 사실상 죽은 프로젝트가 된 안타까운 맥용 프레임워크이다.</p>
<div></div>
<div><a href="http://2.bp.blogspot.com/-sllfWoCkwuI/Uet5yTmNfpI/AAAAAAAABDM/UmK1XVtjysY/s1600/macre_header.png"><img class="aligncenter" alt="" src="{{ site.baseurl }}/assets/macre_header.png" width="400" height="167" border="0" /></a></div>
<div></div>
<div></div>
<div>필자가 이 프레임워크를 확인했을 때 가장 흥미로운 부분은 물리 메모리 이미징 부분이였다. 사실 이때만해도 소프트웨어적인 물리 메모리 이미징 방법은 Hajime Inoue의 MacMemoryReader 뿐이였다. 이 도구는 물리 메모리 이미징을 훌륭하게 수행하지만, 코드가 공개되어 있지 않고, Mac OS X의 커널 디버그 킷을 이용하기 때문에 새로운 Mac OS X 버전이 나올 때마다 개발자가 도구를 업데이트 하기 전까지 물리 메모리를 이미징 할 방법이 없었다. 하지만, MacResponse Forensics는 오픈소스로 코드가 공개되어 있어 새로운 버전에 맞게 리빌드만 해주면 바로 사용할 수 있다.</div>
<div></div>
<div>뭐, 프로젝트 중단으로 인해 Snow Leopard까지만 지원하지만 말이다. 사실 해당 코드를 기반으로 개발된 OSXPMem이 이 코드를 수정하여 마운틴 라이언까지 지원되는 별도의 도구로 런칭되었으니 나름 긍정적 효과를 가져오긴 한 것 같다. ;-) 이 도구에 대해서는 다음 포스팅에 반영하도록 하겠다.</div>
<div></div>
<div>여튼 본 포스팅에서는 MacResponse Forensics의 메모리 이미징 모듈에 대해 정리해보고자 한다. MacResponse Forensics의 메모리 이미징 모듈은 크게 다음 순서로 이미징을 수행한다.&nbsp;</p>
</div>
<div>
<ol>
<li>DTrace 명령을 이용하여 bootArgs 구조체 정보를 추출</li>
<li>bootArgs 구조체를 이용하여 efiMemoryMap을 구성</li>
<li>메모리 맵 정보를 기반으로 물리 메모리를 kernel task의 가상 메모리에 맵핑하여 복사</li>
</ol>
<p>&nbsp;</p>
</div>
<div><b>1. DTrace 명령을 이용하여 bootArgs 구조체 정보를 추출</b></div>
<div>bootArgs는 Mac OS X 커널이 부팅하는 시점에 부팅과 관련된 정보를 PE_state 구조체에 저장한다. <a href="http://code.google.com/p/volafox" target="_blank">volafox</a>에서도 efi 루트킷 분석을 위해 이 구조체를 해석하는 기능을 내장하고 있다.</div>
<pre class="lang:objc theme:twilight" title="pe_state structure in XNU kernel source">typedef struct PE_state {
	boolean_t	initialized;
	PE_Video	video;
	void		*deviceTreeHead;
	void		*bootArgs;
} PE_state_t;</pre>
<pre class="lang:objc theme:twilight" title="boot_args structure in XNU kernel source">typedef struct boot_args {
    uint16_t    Revision;	/* Revision of boot_args structure */
    uint16_t    Version;	/* Version of boot_args structure */

    uint8_t     efiMode;    /* 32 = 32-bit, 64 = 64-bit */
    uint8_t     debugMode;  /* Bit field with behavior changes */
    uint16_t    flags;

    char        CommandLine[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][BOOT_LINE_LENGTH];	/* Passed in command line */

    uint32_t    MemoryMap;  /* Physical address of memory map */
    uint32_t    MemoryMapSize;
    uint32_t    MemoryMapDescriptorSize;
    uint32_t    MemoryMapDescriptorVersion;

    Boot_Video	Video;		/* Video Information */

    uint32_t    deviceTreeP;	  /* Physical address of flattened device tree */
    uint32_t    deviceTreeLength; /* Length of flattened tree */

    uint32_t    kaddr;            /* Physical address of beginning of kernel text */
    uint32_t    ksize;            /* Size of combined kernel text+data+efi */

    uint32_t    efiRuntimeServicesPageStart; /* physical address of defragmented runtime pages */
    uint32_t    efiRuntimeServicesPageCount;
    uint64_t    efiRuntimeServicesVirtualPageStart; /* virtual address of defragmented runtime pages */

    uint32_t    efiSystemTable;   /* physical address of system table in runtime area */
    uint32_t    kslide;

    uint32_t    performanceDataStart; /* physical address of log */
    uint32_t    performanceDataSize;

    uint32_t    keyStoreDataStart; /* physical address of key store data */
    uint32_t    keyStoreDataSize;
    uint64_t	bootMemStart;
    uint64_t	bootMemSize;
    uint64_t    PhysicalMemorySize;
    uint64_t    FSBFrequency;
    uint64_t    pciConfigSpaceBaseAddress;
    uint32_t    pciConfigSpaceStartBusNumber;
    uint32_t    pciConfigSpaceEndBusNumber;
    uint32_t    __reserved4[730];

} boot_args;</pre>
<div>MacResponse Forensics는 이 구조체 정보를 얻기 위해, 내부적으로 DTrace 명령을 실행하고 그 결과를 받는 구조로 되어있다.</div>
<pre class="lang:objc theme:twilight" title="Dtrace Script for getting bootArgs structure">BEGIN

{
     self->boot_args = ((unsigned char *)(`PE_state).bootArgs);
     self->i = 0;
     self->inited = 1;
}

fbt:::entry /self->inited && self->i < sizeof(struct boot_args)/
{
     this->;byte = *(self->boot_args + self->i);
     printf("%c", this->byte);
     self->i++;
}

fbt:::return /self->inited && self->i >= sizeof(struct boot_args)/
{
     exit(0);
}</pre>
<div><b>2. bootArgs 구조체를 이용하여 efiMemoryMap을 구성</b></div>
<div>bootArgs 구조체에는 물리 메모리 맵에 대한 정보를 가지고 있다. 이 맵은 운영체제가 물리 메모리를 효과적으로 관리하기 위한 용도로 사용한다. 도구는 bootArgs 중, MemoryMapSize, MemoryMapDescriptorSize, MemoryMap 정보를 가져와서 efiMemoryMap이라는 힙영역에 데이터를 저장한다. 메모리 맵핑을 수행한다.</div>
<pre class="lang:objc theme:twilight" title="getEfiMemoryMap">- (Boolean) getEfiMemoryMap

{
    uint32_t memoryMapSize;

    memoryMapSize = bootArgs.MemoryMapSize;

    LogDebugObjc(@"Obtained boot arguments...n");

               // some quick sanity checks

    if (bootArgs.efiMode != 32 && bootArgs.efiMode != 64)
    {
        LogDebugObjc(@"Invalid bootArgs.efiMode: %d (0x%x)n", bootArgs.efiMode, bootArgs.efiMode);
        return FALSE;
    }

    // since only version 1 has ever seen the light of day...
    if (bootArgs.Version != 1)
    {
        LogDebugObjc(@"Invalid bootArgs.Version: %d (0x%x)n", bootArgs.Version, bootArgs.Version);
        return FALSE;
    }

    LogDebugObjc(@"Validated boot arguments...n");

    LogDebugObjc(@"Allocating %d bytes for MemoryMap...n", memoryMapSize);

    efiMemoryMap = malloc(memoryMapSize);

    if (efiMemoryMap == NULL)
    {
        LogDebugObjc(@"Failed to malloc efiMemoryRangeList...n");
        return FALSE;
    }

    memset(efiMemoryMap, 0, sizeof(efiMemoryMap));

    // now fill in memory map
    void *sourceMemoryMap = [self mapMemoryRegion:bootArgs.MemoryMap withLength:memoryMapSize];

    if (sourceMemoryMap)
    {
        LogDebugObjc(@"Obtained memory map successfully...n");
        memcpy(efiMemoryMap, sourceMemoryMap, memoryMapSize);
        [self printRange:efiMemoryMap withLength:memoryMapSize];
        return TRUE;
    }

    LogDebugObjc(@"Failed to obtain memory map...n");
    return FALSE;
}</pre>
<div><b>3. 메모리 맵 정보를 기반으로 물리 메모리를 kernel task의 가상 메모리에 맵핑하여 복사</b></div>
<div>애플리케이션에서 메모리 맵을 구성하고 나면, 로드한 KEXT에 물리 메모리 주소와 크기 정보를 전달한다. 두 정보는 메모리 맵에 있다. 물리 메모리 주소와 크기를 전달하면, KEXT는 다음 메서드를 호출한다.</div>
<pre class="lang:objc theme:twilight" title="com_ainfosec_driver_memoryaccessiokit">UInt64 com_ainfosec_driver_MemoryAccessIOKit::mapMemoryIntoUserTask(task_t userTask, UInt64 requestedAddress, UInt64 requestedSize)

{
    // ensure the previous descriptors have been released
    freeMemoryDescriptors();

    lastMemoryDescriptor = IOMemoryDescriptor::withPhysicalAddress((IOPhysicalAddress)requestedAddress, (IOByteCount)requestedSize, kIODirectionIn );

    if (!lastMemoryDescriptor)
    {
        IOLog("mapMemoryIntoUserTask: unable to allocate memDesc!n");
        return 0;
    }

    lastMemoryMap = lastMemoryDescriptor->createMappingInTask(userTask, 0, kIOMapAnywhere | kIOMapDefaultCache, 0, 0);

    if (!lastMemoryMap)
    {
        IOLog("mapMemoryIntoUserTask: unable to map to user task!n");
        freeMemoryDescriptors();
        return 0;
    }

    return (UInt64) lastMemoryMap->getAddress();
}</pre>
<div>물리 메모리는 직접 접근이 불가능하기 때문에, Mac OS X의 ABI를 이용하여 물리 메모리 페이지를 task의 메모리 영역에 맵핑해야 한다. KEXT가 로드되면, 커널의 가상 메모리 영역에 위치하므로, 여기에선 kernel_task의 메모리 영역을 이용한다. 이 일련의 과정은 <a href="https://developer.apple.com/library/mac/#documentation/Kernel/Reference/IOMemoryDescriptor_reference/translated_content/IOMemoryDescriptor.html">IOMemoryDescriptor Class</a>를 이용한다.</div>
<ul>
<li>IOMemoryDescriptor::withPhysicalAddress로 특정 물리 메모리 주소 영역을 IOMemoryDescriptor에 맵핑한다.</li>
<li>createMappingInTask로 'kerneltask' task에 해당 IOMemoryDescriptor를 맵핑하고, IOMemoryMap을 돌려준다. 이 객체에는 맵핑된 task 내부 가상 주소 정보를 가지고 있다.</li>
</ul>
<div>Mac Response Forensics가 사용하는 방법은 KEXT에서 커널 메모리 주소로 접근하여 덤프하는 방법으로 물리 메모리 이미지를 획득한다. 단, 이 방법은 Mac OS X Lion 또는, 64비트 환경에서는 정상적인 동작이 불가능한 것으로 알려져있다. 그리고 최근에 이 문제를 해결한 OSXPMem이 공개되었다. ;-) OSXPMem이 이 문제를 어떻게 해결했는지에 대해서는 다음 포스팅으로 작성하겠다.</div>

