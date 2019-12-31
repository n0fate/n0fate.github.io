---
layout: post
title: OSXPMem의 물리 메모리 이미징 방법
date: 2013-07-21 15:52:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
author: "n0fate"
---
<p><!--?xml version="1.0" encoding="UTF-8" standalone="no"?--> 2012년까지만해도 OSX이 메모리 이미징 방법에 Closed-Source 프로젝트인 MacMemoryReader에 의존하다보니, 개발자가 업데이트하지 않는 이상 최신 버전의 OS X의 메모리 이미징을 하지 못하는 문제가 발생하였다. <a href="http://code.google.com/p/pmem/" target="_blank">OSXPMem</a>는 2012년 말에 volatility framework를 개발 중인 구글 코드 프로젝트에서 오픈소스로 개발된 OSX용 물리 메모리 이미징 도구이다. 이 도구는 Mac OS X Leopard부터 최신 버전인 Mountain Lion까지 지원했기 때문에 새로운 기술을 이용한 메모리 이미징 기법을 사용했을 거라 생각했다. 하지만, 실제 코드 분석 결과 물리 메모리를 읽는 루틴이 MacResponseLE와 동일하였다. 이 포스팅에서는 OSXPMem의 메모리 이미징 절차를 알아보겠다. 이 도구도 크게 세 절차를 통해 물리 메모리를 이미징하며, 크게 보면 MacResponseLE와 동일한 방법을 사용한다.</p>
<div>
<div></div>
<div></div>
<div><b>1. Platform Export를 이용한 PE_state 구조체 정보 획득</b></div>
<div>OSXPMem은 MacResponseLE와 다르게 DTrace를 이용하지 않는다. 대신에 KEXT의 platform export를 이용한다. Platform export는 운영체제 플랫폼인 커널에서 익스포트한 구조체나 함수를 의미하는 것으로, 필요 시 커널 구조체의 정보에 원할하게 접근할 수 있게 하기 위해 존재한다. KEXT는 로드될 때, start 핸들러인 'pmem_start'가 실행되며, 이 함수는 PE_state의 bootArgs 구조체에 접근한다.</div>
<pre class="lang:obj-c theme:twilight" title="access to bootArgs structure">// Access the boot arguments through the platform export,
  // and parse the systems physical memory configuration.
  boot_args * ba = reinterpret_cast<boot_args *="">;(PE_state.bootArgs);
  pmem_physmem_size = ba->PhysicalMemorySize;
  pmem_mmap = reinterpret_cast(ba->MemoryMap + pmem_kernel_voffset);
  pmem_mmap_desc_size = ba->MemoryMapDescriptorSize;
  pmem_mmap_size = ba->MemoryMapSize;
… ...</pre>
<div> KEXT가 물리 메모리 분석에 필요한 기본적인 정보를 저장해두고, 애플리케이션은 ioctl() 함수를 이용하여 자신이 로드한 pmem 디바이스에 bootArgs 구조체에서 물리 메모리 맵 정보를 가져온다.</div>
<pre class="lang:obj-c theme:twilight" title="get_mmap()">// Send an ioctl to the driver to get the physical memory map.
// Will also retrieve the size of the map and its descriptors.
// This function will allocate memory for mmap, make sure you free it.
//
// args: mmap is a pointer to a pointer that will recieve the memory map.
//       mmap_size is a pointer that will recieve the size of the memory map.
//       mmap_desc_size is a pointer that will recieve the size of an individual
//       memory descriptor in the memory map.
//       device_file is an open file descriptor to the pmem device file.
//
// return: EXIT_SUCCESS and EXIT_FAILURE.
//
unsigned int get_mmap(uint8_t **mmap, unsigned int *mmap_size,
                      unsigned int *mmap_desc_size, int device_file) {
  int err;
  int status = EXIT_FAILURE;

  err = ioctl(device_file, PMEM_IOCTL_GET_MMAP_SIZE, mmap_size);
  if (err != 0) {
    PMEM_ERROR_LOG("Error getting size of memory map");
    goto error;
  }
…</pre>
<div>이 결과를 통해 물리 메모리 맵을 확보한다.</div>
<div></div>
<div></div>
<div><b>2. EfiMemoryRange 구조체에 메모리 맵을 저장</b></div>
<div>앞서 획득한 bootArgs 구조체의 정보를 이용하여 EfiMemoryRange라는 이름의 버퍼에 물리 메모리 맵 정보를 만든다. 물리 메모리 맵핑 정보는 세그먼트 형태로 나뉘어져있기 때문에, 익스포트할 파일 포맷에 맞게 맵핑 데이터를 재구성한다.</div>
<pre class="lang:obj-c theme:twilight" title="dump_memory_raw()">// Parse the mmap and dump each section into a raw file. Memory holes or
// unreadable sections like MMIO are zero padded in the dump file.
//
// args: mem_dev is an open filehandle to the pmem device file (/dev/pmem).
//       dump_file is an open filehandle to which the image will be written.
//
// return: EXIT_SUCCESS or EXIT_FAILURE.
//
unsigned int dump_memory_raw(int mem_dev, int dump_file) {
  unsigned int status = EXIT_FAILURE;
  uint64_t section = 0;
  uint64_t phys_as_size = 0;
  uint64_t bytes_imaged = 0;
  uint8_t *mmap = NULL;
  unsigned int mmap_size = 0;
  unsigned int mmap_desc_size = 0;

  if (get_mmap(&mmap, &mmap_size, &mmap_desc_size, mem_dev) == EXIT_FAILURE) {
    print_msg(STD, "Failed to obtain memory mapn");
    goto error_malloc;
  }
  // Iterate over each section in the physical memory map and write it to disk.
  for (section = 0; section < mmap_size / mmap_desc_size; section++) {     EfiMemoryRange *segment = (EfiMemoryRange *)(         mmap + (section * mmap_desc_size));     // dump the segment     uint64_t start = segment->PhysicalStart;
    uint64_t size = segment->NumberOfPages * PAGE_SIZE;
    print_msg(STD, "[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][%016llx - %016llx] %s ", start, start + size,
              physmem_type_tostring(segment->Type));
    if (segment_accessible(segment)) {
      if (write_segment(segment, mem_dev, dump_file, start) == EXIT_FAILURE) {
        print_msg(STD, "Failed to dump segment %dn", section);
        goto error;
      }
…</pre>
<div></div>
<div></div>
<div><b>3. IOMemoryDescriptor 클래스의 메서드를 이용한 물리 메모리 추출</b></div>
<div>물리 메모리 맵을 확보하면, MacReponseLE와 같은 방법으로 물리 메모리를 덤프한다. OSXPMem이 등록한 'pmem' KEXT에 read 메시지를 전달하면, 내부적으로 'pmem_read'를 호출하며 이 함수는 결과적으로 다음 함수를 호출한다.</div>
<pre class="lang:obj-c theme:twilight" title="pmem_partial_read()">// Copy the requested amount to userspace if it doesn't cross page boundaries

// or memory mapped io. If it does, stop at the boundary. Will copy zeroes
// if the given physical address is not backed by physical memory.
//
// args: uio is the userspace io request object
// return: number of bytes copied successfully
//
static uint64_t pmem_partial_read(struct uio *uio, addr64_t start_addr,
                                  addr64_t end_addr) {
  // Separate page and offset
  uint64_t page_offset = start_addr & PAGE_MASK;
  addr64_t page = trunc_page_64(start_addr);
  // don't copy across page boundaries
  uint32_t chunk_len = (uint32_t)MIN(PAGE_SIZE - page_offset,
                                     end_addr - start_addr);
  // Prepare the page for IOKit
  IOMemoryDescriptor *page_desc = (
      IOMemoryDescriptor::withPhysicalAddress(page, PAGE_SIZE, kIODirectionIn));
  if (page_desc == NULL) {
    pmem_error("Can't read from %#016llx, address not in physical memory range",
               start_addr);
    // Skip this range as it is not even in the physical address space
    return chunk_len;
  } else {
    // Map the page containing address into kernel address space.
    IOMemoryMap *page_map = (
        page_desc->createMappingInTask(kernel_task, 0, kIODirectionIn, 0, 0));
    // Check if the mapping succeded.
    if (!page_map) {
      pmem_error("page %#016llx could not be mapped into the kernel, "
               "zero padding return buffer", page);
      // Zero pad this chunk, as it is not inside a valid page frame.
      uiomove64((addr64_t)pmem_zero_page + page_offset,
                (uint32_t)chunk_len, uio);
    } else {
      // Successfully mapped page, copy contents...
      uiomove64(page_map->getAddress() + page_offset, (uint32_t)chunk_len, uio);
      page_map->release();
    }
    page_desc->release();
  }
  return chunk_len;
}</pre>
<div>이 함수는 인자로 받은 uio(User I/O) 구조체에 맵핑된 물리 메모리 정보를 복사한다. uio 구조체는 사용자가 애플리케이션의 버퍼이다. 단순 버퍼 복사로는 커널 메모리 영역의 데이터를 사용자 영역의 데이터로 전송할 수 없으므로, <a href="http://www.opensource.apple.com/source/xnu/xnu-517.9.5/bsd/kern/kern_subr.c" target="_blank">uiomove64()</a>를 이용한다. 이 함수는 BSD에 있는 시스템 콜(system call)중 하나로, 커널 영역 버퍼의 정보를 사용자 영역의 버퍼로 복사한다.</div>
<div></div>
<div>결론적으로 OSXPMem과 MacResponseLE는 내부적으로 동일한 루틴을 가지지만, 기존 프로젝트와 다르게 DTrace 호출하지 않고 KEXT가 커널의 데이터에 접근 가능하다는 점을 이용하여 platform export를 활용하였다. OSXPMem은 앞으로도 지속적인 업데이트를 수행할 예정이니만큼, 기대해봐도 좋을 것 같다. ;-)</div>
</div>

