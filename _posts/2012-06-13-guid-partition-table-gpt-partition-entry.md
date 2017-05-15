---
layout: post
title: 'Guid Partition Table : GPT Partition Entry'
date: 2012-06-13 10:54:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Boot Strap
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/06/guid-partition-table-gpt-partition.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/4164174175650893997
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1652'
  fusion_builder_content_backup: GPT에 대한 마지막 포스팅이다. 이번 과정에는 GPT 헤더에서 가져온 정보를 기반으로
    GPT 파티션 엔트리의 정보를 확인하고자 한다.<br /><br />우선 이전 포스팅에서 설명한 각 영역을 통해 디스크 내에 어떠한 정보를
    가져올 수 있는지를 간단하게 정리하겠다.<br /><br /><br /><b>1. Protective MBR</b><br /><br />Protective
    MBR은 EFI를 공식적을 지원하는 운영체제의 경우엔 접근하지 않는 부분이며, 비 지원 운영체제(ie. Windows XP)의 경우 EFI에서
    바이오스 에뮬레이션을 하여 접근하는 부분이다. Protective MBR은 GPT 파티션 영역 전체의 시작 주소와 끝주소를 16바이트 주 파티션
    엔트리 0번에 저장하고 있다.<br /><br /><img alt="protective MBR.png" border="0" height="169"
    src="http://lh4.ggpht.com/-Lsf9KGVeisc/T9fxhpVKpqI/AAAAAAAAASo/xbk-n6g_LKI/protective%252520MBR.png?imgmax=800"
    title="protective MBR.png" width="600"><br /><div><span><protective MBR 분석 결과></span></div><div><br
    /></div><div><br /></div><div><b>2. (Primary) GPT Header</b></div><div><br /></div><div>GPT
    Area의 맨 처음 주소이며, LBA(Logical Block Area) 1번(0x200)에 위치하는 헤더로, GPT에 대한 전반적인 설정
    정보를 담고 있다. GPT 헤더는 보통 92바이트의 크기를 가지며, 그 외의 영역(Logical Block Size - GPT Header
    Size)은 NULL로 작성되어 있다.</div><div><br /></div><div>GPT 헤더에는 헤더의 무결성 체크를 위한 CRC32
    값(CRC32 of Header)과 각 파티션 엔트리의 시작 위치(Partition entries starting LBA), 엔트리의 크기(Size
    of partition entry), 엔트리의 갯수(Number of partition entries), 파티셔닝 할 수 있는 영역(First/Last
    usable LBA for partitions), 보조 GPT 헤더의 위치(Backup LBA) 정보를 담고 있다. 이 정보를 기반으로 디스크
    내에 확인할 수 있는 정보는 다음과 같다.</div><div><br /></div><div><br /></div><div><img alt="gpt
    header.png" border="0" height="245" src="http://lh4.ggpht.com/-IDTx4CuY4a0/T9fxj5wxA3I/AAAAAAAAASw/1EtynxCf5l8/gpt%252520header.png?imgmax=800"
    title="gpt header.png" width="600"></div><div><span><gpt header 분석 결과></span></div><div><br
    /></div><br /><b>3. (Primary) GPT Partition Entry</b><br /><br />각 파티션의 이름과 위치
    정보와 같은 파티션 관련 정보를 가지는 엔트리로 이번 포스팅에서 다룰 주제이다.<br /><br />일단 GPT 파티션 엔트리가 가지는 정보를
    알아보겠다.<br /><ul><li>Partition type GUID(16bytes) - 파티션 타입을 표현하는 고유 정보이다. 이 정보를
    토대로 각 파티션의 파일 시스템과 사용하는 운영체제 및 서비스 정보를 확인할 수 있다.</li><li>Unique partition GUID(16)
    - 각 파티션마다 할당하는 고유 값을 저장한다.</li><li>First LBA(8) - 해당 파티션이 시작하는 LBA 위치를 저장한다.</li><li>Last
    LBA(8) - 해당 파티션이 끝나는 LBA 위치를 저장한다.</li><li>Attribute flags(8) - 파티션이 시스템 파티션(System
    Partition)인지, 구 바이오스 부팅 방식(Legacy BIOS Bootable)인지, 읽기 전용(Read-Only)인지, 숨김(Hidden)인지와
    같은 정보를 가진다.</li><li>Partition Name(72) - 파티션 명을 저장한다. 파티션 명은 시스템이 할당하거나 사용자가 지정할
    수 있다.</li></ul><br />GPT 파티션 엔트리는 앞선 헤더 정보에 비하면 간단한 구조를 가지고 있다. 이제 실제 디스크 이미지
    상에서 파티션 영역을 추적해보겠다. 디스크 이미지가 Mac OS X Snow Leopard라서 앞선 포스팅에서 <gpt 구조> 상에 표현된
    파티션 구조와는 약간 다를 수 있다.<br /><br /><div><img alt="Gpt entry" border="0" height="263"
    src="http://lh6.ggpht.com/-ZMW4cNtVZ5c/T9fxsOEuPtI/AAAAAAAAAS4/-zd2J4FeBnE/gpt%252520entry.jpg?imgmax=800"
    title="gpt entry.jpg" width="600"></div><div><span><gpt Partition Entry></span></div><br
    /><br />내부 데이터를 표현하기 위해 Parittion GUID를 제외한 영역엔 별도 정보를 표기하지 않았다. 그럼 첫 번째 GPT Partition
    Entry의 항목을 해석하겠다.<br /><ul><li>Partition type GUID - GPT 헤더에 나타난 디스크 UUID와 동일한
    포맷으로 파티션의 타입 정보를 가진다. 파티션 타입 GUID는 <a href="http://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs">위키피디아:GUID
    Partition Table</a>에서 확인할 수 있다. 위 예에선 파티션 엔트리 0 의 Type GUID가 C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    이므로 별도의 운영체제가 명시되지 않은 EFI System Partition 임을 확인할 수 있다.</li><li>Partition GUID
    - 앞선 GUID와 동일한 형식으로 표현된다. 생략.</li><li>First LBA - 위 예에선 LBA 0x28 을 가리키고 있다. 이는
    디스크 오프셋으로 0x5000을 의미한다.</li><li>Last LBA - 위 예에선 LBA 0x064027을 가리키고 있다. 이는 디스크
    오프셋으로 0xC804E00을 의미한다. 즉, EFI System Partition 타입의 파티션은 0x5000 ~ 0xC804E00의 크기를
    가진다.</li><li>Attribute flags - 위 예제에선 0x00이며 이 값은 System Partition을 의미한다. (위키피디아
    또는 인텔 스펙 문서 참조)</li><li>Partition Name - 유니코드 형식으로 표현되며 'EFI System Partition'이라고
    명명되어 있다.</li></ul><br />위 정보를 토대로 실제 시작 LBA 주소를 추적하면, FAT32 파일시스템의 부트 섹터를 확인할
    수 있다.<br /><br /><div><img alt="fat32.png" border="0" height="279" src="http://lh6.ggpht.com/-Op3DWx3eMrg/T9fx4SUhUAI/AAAAAAAAATA/l2zSttJgD8M/fat32.png?imgmax=800"
    title="fat32.png" width="600"></div><div><span><efi System Partition 시작 주소></span></div><br
    /><br />파티션 엔트리 1도 동일한 방식으로 해석할 수 있으며, 이를 gpt parser로 분석한 화면은 다음과 같다.<br /><br
    /><span>gpt parser - </span><a href="https://github.com/n0fate/raw/blob/master/gpt_parser.py">https://github.com/n0fate/raw/blob/master/gpt_parser.py</a><br
    /><br /><div><img alt="Gpt parser partition entry" border="0" height="301" src="http://lh3.ggpht.com/-UGAq3I0B3nU/T9fx94e8CaI/AAAAAAAAATI/T6_RAgFmBd8/gpt%252520parser%252520partition%252520entry.jpg?imgmax=800"
    title="gpt parser partition entry.jpg" width="600"></div><div><span><gpt parser를
    통한 분석></span></div><br /><br />분석 결과를 그림으로 표현하면 다음과 같다.<br /><br /><div><img alt="gpt
    entry pic.png" border="0" height="213" src="http://lh5.ggpht.com/-fo1wd9-jSI0/T9fyCoBDBcI/AAAAAAAAATQ/vr40J6eztG0/gpt%252520entry%252520pic.png?imgmax=800"
    title="gpt entry pic.png" width="600"></div><div><span><gpt Partition Entry 분석
    결과></span></div><br /><br />GPT 파티션 엔트리까지 GPT 영역에 대한 모든 분석을 완료하였다. GPT는 MBR과 다르게
    부트 섹터가 존재하지 않으며, 보든 코드 실행을 EFI에서 수행하기 때문에 기존 루트킷의 코드 수정을 통한 은닉(MBR Rootkit)은 통하지
    않는다. 하지만, EFI는 누구나 개발할 수 있도록 SDK를 제공하기 때문에 EFI를 교체하거나 BootService 종료 함수를 후킹하는
    등 다양한 방법을 통한 은닉을 수행할 수 있다.<br /><br />GPT에 대해 더 흥미가 있다면, EFI에 대한 연구를 진행하면 다양한
    연구 주제를 도출할 수 있을 것이다 :)<br /><div><br /></div><div>n0fate's Forensic Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>GPT에 대한 마지막 포스팅이다. 이번 과정에는 GPT 헤더에서 가져온 정보를 기반으로 GPT 파티션 엔트리의 정보를 확인하고자 한다.</p>
<p>우선 이전 포스팅에서 설명한 각 영역을 통해 디스크 내에 어떠한 정보를 가져올 수 있는지를 간단하게 정리하겠다.</p>
<p><b>1. Protective MBR</b></p>
<p>Protective MBR은 EFI를 공식적을 지원하는 운영체제의 경우엔 접근하지 않는 부분이며, 비 지원 운영체제(ie. Windows XP)의 경우 EFI에서 바이오스 에뮬레이션을 하여 접근하는 부분이다. Protective MBR은 GPT 파티션 영역 전체의 시작 주소와 끝주소를 16바이트 주 파티션 엔트리 0번에 저장하고 있다.</p>
<p><img alt="protective MBR.png" border="0" height="169" src="{{ site.baseurl }}/assets/protective%252520MBR.png?imgmax=800" title="protective MBR.png" width="600" />
<div><span>
<protective MBR 분석 결과></span></div>
<div></div>
<div></div>
<div><b>2. (Primary) GPT Header</b></div>
<div></div>
<div>GPT Area의 맨 처음 주소이며, LBA(Logical Block Area) 1번(0x200)에 위치하는 헤더로, GPT에 대한 전반적인 설정 정보를 담고 있다. GPT 헤더는 보통 92바이트의 크기를 가지며, 그 외의 영역(Logical Block Size - GPT Header Size)은 NULL로 작성되어 있다.</div>
<div></div>
<div>GPT 헤더에는 헤더의 무결성 체크를 위한 CRC32 값(CRC32 of Header)과 각 파티션 엔트리의 시작 위치(Partition entries starting LBA), 엔트리의 크기(Size of partition entry), 엔트리의 갯수(Number of partition entries), 파티셔닝 할 수 있는 영역(First/Last usable LBA for partitions), 보조 GPT 헤더의 위치(Backup LBA) 정보를 담고 있다. 이 정보를 기반으로 디스크 내에 확인할 수 있는 정보는 다음과 같다.</div>
<div></div>
<div></div>
<div><img alt="gpt header.png" border="0" height="245" src="{{ site.baseurl }}/assets/gpt%252520header.png?imgmax=800" title="gpt header.png" width="600" /></div>
<div><span><gpt header 분석 결과></span></div>
<div></div>
<p><b>3. (Primary) GPT Partition Entry</b></p>
<p>각 파티션의 이름과 위치 정보와 같은 파티션 관련 정보를 가지는 엔트리로 이번 포스팅에서 다룰 주제이다.</p>
<p>일단 GPT 파티션 엔트리가 가지는 정보를 알아보겠다.
<ul>
<li>Partition type GUID(16bytes) - 파티션 타입을 표현하는 고유 정보이다. 이 정보를 토대로 각 파티션의 파일 시스템과 사용하는 운영체제 및 서비스 정보를 확인할 수 있다.</li>
<li>Unique partition GUID(16) - 각 파티션마다 할당하는 고유 값을 저장한다.</li>
<li>First LBA(8) - 해당 파티션이 시작하는 LBA 위치를 저장한다.</li>
<li>Last LBA(8) - 해당 파티션이 끝나는 LBA 위치를 저장한다.</li>
<li>Attribute flags(8) - 파티션이 시스템 파티션(System Partition)인지, 구 바이오스 부팅 방식(Legacy BIOS Bootable)인지, 읽기 전용(Read-Only)인지, 숨김(Hidden)인지와 같은 정보를 가진다.</li>
<li>Partition Name(72) - 파티션 명을 저장한다. 파티션 명은 시스템이 할당하거나 사용자가 지정할 수 있다.</li>
</ul>
<p>GPT 파티션 엔트리는 앞선 헤더 정보에 비하면 간단한 구조를 가지고 있다. 이제 실제 디스크 이미지 상에서 파티션 영역을 추적해보겠다. 디스크 이미지가 Mac OS X Snow Leopard라서 앞선 포스팅에서 <gpt 구조> 상에 표현된 파티션 구조와는 약간 다를 수 있다.</p>
<div><img alt="Gpt entry" border="0" height="263" src="{{ site.baseurl }}/assets/gpt%252520entry.jpg?imgmax=800" title="gpt entry.jpg" width="600" /></div>
<div><span><gpt partition entry /></span></div>
<p>내부 데이터를 표현하기 위해 Parittion GUID를 제외한 영역엔 별도 정보를 표기하지 않았다. 그럼 첫 번째 GPT Partition Entry의 항목을 해석하겠다.
<ul>
<li>Partition type GUID - GPT 헤더에 나타난 디스크 UUID와 동일한 포맷으로 파티션의 타입 정보를 가진다. 파티션 타입 GUID는 <a href="http://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs">위키피디아:GUID Partition Table</a>에서 확인할 수 있다. 위 예에선 파티션 엔트리 0 의 Type GUID가 C12A7328-F81F-11D2-BA4B-00A0C93EC93B 이므로 별도의 운영체제가 명시되지 않은 EFI System Partition 임을 확인할 수 있다.</li>
<li>Partition GUID - 앞선 GUID와 동일한 형식으로 표현된다. 생략.</li>
<li>First LBA - 위 예에선 LBA 0x28 을 가리키고 있다. 이는 디스크 오프셋으로 0x5000을 의미한다.</li>
<li>Last LBA - 위 예에선 LBA 0x064027을 가리키고 있다. 이는 디스크 오프셋으로 0xC804E00을 의미한다. 즉, EFI System Partition 타입의 파티션은 0x5000 ~ 0xC804E00의 크기를 가진다.</li>
<li>Attribute flags - 위 예제에선 0x00이며 이 값은 System Partition을 의미한다. (위키피디아 또는 인텔 스펙 문서 참조)</li>
<li>Partition Name - 유니코드 형식으로 표현되며 'EFI System Partition'이라고 명명되어 있다.</li>
</ul>
<p>위 정보를 토대로 실제 시작 LBA 주소를 추적하면, FAT32 파일시스템의 부트 섹터를 확인할 수 있다.</p>
<div><img alt="fat32.png" border="0" height="279" src="{{ site.baseurl }}/assets/fat32.png?imgmax=800" title="fat32.png" width="600" /></div>
<div><span><efi System Partition 시작 주소></span></div>
<p>파티션 엔트리 1도 동일한 방식으로 해석할 수 있으며, 이를 gpt parser로 분석한 화면은 다음과 같다.</p>
<p><span>gpt parser - </span><a href="https://github.com/n0fate/raw/blob/master/gpt_parser.py">https://github.com/n0fate/raw/blob/master/gpt_parser.py</a></p>
<div><img alt="Gpt parser partition entry" border="0" height="301" src="{{ site.baseurl }}/assets/gpt%252520parser%252520partition%252520entry.jpg?imgmax=800" title="gpt parser partition entry.jpg" width="600" /></div>
<div><span><gpt parser를 통한 분석></span></div>
<p>분석 결과를 그림으로 표현하면 다음과 같다.</p>
<div><img alt="gpt entry pic.png" border="0" height="213" src="{{ site.baseurl }}/assets/gpt%252520entry%252520pic.png?imgmax=800" title="gpt entry pic.png" width="600" /></div>
<div><span><gpt Partition Entry 분석 결과></span></div>
<p>GPT 파티션 엔트리까지 GPT 영역에 대한 모든 분석을 완료하였다. GPT는 MBR과 다르게 부트 섹터가 존재하지 않으며, 보든 코드 실행을 EFI에서 수행하기 때문에 기존 루트킷의 코드 수정을 통한 은닉(MBR Rootkit)은 통하지 않는다. 하지만, EFI는 누구나 개발할 수 있도록 SDK를 제공하기 때문에 EFI를 교체하거나 BootService 종료 함수를 후킹하는 등 다양한 방법을 통한 은닉을 수행할 수 있다.</p>
<p>GPT에 대해 더 흥미가 있다면, EFI에 대한 연구를 진행하면 다양한 연구 주제를 도출할 수 있을 것이다 :)
<div></div>
<div>n0fate's Forensic Space :)</div>
