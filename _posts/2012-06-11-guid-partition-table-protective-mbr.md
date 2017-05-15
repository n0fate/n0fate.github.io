---
layout: post
title: 'Guid Partition Table : Protective MBR'
date: 2012-06-11 13:40:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Boot Strap
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/06/guid-partition-table-protective-mbr.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/4027640696434079210
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1589'
  fusion_builder_content_backup: Mac OS X 포렌식에 관심이 있다면, 바이오스가 아닌 EFI를 활용한다는 것은 알고
    있을 것이다. 윈도우 운영체제도 비스타 SP1부터 EFI-GPT 를 지원하면서 최근 출시되는 많은 노트북에서 바이오스가 사라지고 있다. 기존
    바이오스와 비교해서 새로운 부트 플랫폼 구조는 다음과 같이 매치 된다.<br /><br />BIOS(Bootable Input/Output
    Service) -> EFI(Extensible Firmware Interface)<br />MBR(Master Boot Record) ->
    GPT(Guid Partition Table)<br /><br />이번 포스팅을 시작으로 GPT의 구조에 대해 알아보려고 한다.  우선 GPT의
    기본적인 구조는 다음과 같다.<br /><br /><div><img alt="gpt structure.png" border="0" height="213"
    src="http://lh6.ggpht.com/-K4aSTP3533s/T9V2rGEYRzI/AAAAAAAAAR8/pqf1P71P3_M/gpt%252520structure.png?imgmax=800"
    title="gpt structure.png" width="600"></div><div><span><b><guid Partition Table
    - Mac OS X Lion></b></span></div><br />보통 GPT를 설명하는 문서에는 LBA라는 표현을 사용한다. LBA는
    논리 블록 주소(Logical Block Address)로 특정 바이트 사이즈를 하나의 블록으로 잡아 표현한다. GPT의 경우엔 512바이트를
    하나의 블록으로 설정하며, 이에 Protective MBR 영역은 LBA 0 이라고 표현한다. Mac OS X는 이 Protective MBR
    영역에 하나의 주 파티션 테이블 정보를 가지고 있다.<br /><br /><div><img alt="mbr.png" border="0" height="495"
    src="http://lh4.ggpht.com/-ipgyMui6YMg/T9V2tg6cNzI/AAAAAAAAASE/8Q3OORrwBY0/mbr.png?imgmax=800"
    title="mbr.png" width="600"></div><div><span><b><protective MBR Area></b></span></div><br
    />MBR의 코드, 디스크 시그너처, 영역이 전부 NULL 상태이며, 0x1BE 부터 시작하는 주 파티션 테이블에만 정보가 존재한다. 주 파티션
    테이블엔 보통 4개의 파티션 엔트리가 위치할 수 있는데 본 예제의 경우엔 하나의 파티션 정보를 가지고 있다.<br /><ul><li>0x1BE
    - 0x00 - Non Bootable</li><li><span><strong>0x1BF ~ 0x1C1 - 0xFE, 0xFF, 0xFF -
    Starting CHS address - 사용되지 않음</strong></span></li><li><strong>0x1C2 - 0xEE -
    EFI GPT Partition Type (Hybrid MBR)</strong></li><li><span><strong>0x1C3 ~ 0x1C5 -
    0xFE, 0xFF, 0xFF - Ending CHS address - 사용되지 않음</strong></span></li><li>0x1C6
    ~ 0x1C9 - 0x01 0x00 0x00 0x00 - Starting LBA Address(0x01)</li><li>0x1CA ~ 0x1CD
    - 0xAF, 0xC2, 0xE7, 0x0E - Total sectors(0xEE7C2AF * 0x200bytes(sector size) -->
    128,035,675,648bytes --> 120GB)</li></ul>각 요소는 다음과 같이 해석된다.<br /><ul><li>0x1BE
    - Non Bootable - 부팅되지 않는 파티션이란 의미로, GPT 헤더를 가리킬 경우엔 보통 0x00을 사용한다.</li><li><b><span>0x1BF
    ~ 0x1C1 - 사용되지 않음</span></b></li><li>0x1C2 - EFI GPT Partition Type은 해당 주 파티션이
    GPT 형식으로 관리를 하고 있다는 의미로, 하이브리드 MBR로 운용할 경우 설정하는 정보이다.(http://www.rodsbooks.com/gdisk/hybrid.html).
    Mac OS X만 설치된 경우엔 GPT 기반으로 동작하므로 GPT 영역 정보를 가지는 하나의 엔트리만 존재하게 된다.</li><li><span><b>0x1C3
    ~ 0x1C5 - 사용되지 않음</b></span></li><li>0x1C6 ~ 0x1C9 - Starting LBA Address가 0x01로
    LBA 1부터 GPT 영역임을 의미한다. GPT 파티션 관리 구조에서는 보통 LBA 1에 주 GPT 헤더가 위치한다.</li><li>0x1CA
    ~ 0x1CD - Total Sector Size는 총 몇 개의 섹터가 해당 파티션에 할당되어 있는지를 말하는 것으로 예제의 경우엔 전체 디스크
    사이즈만큼을 할당하여 사용한다.</li></ul>기존 MBR을 공부한 사람이라면, 다른 부분은 이해할 수 있지만 하이브리드 MBR은 상당히
    생소할 것이다. 사실 EFI가 지원되는 맥 시스템(최근 맥 시스템은 모두 지원함)에서 Mac OS X만 구동된다면, MBR 영역은 필요 없다.
    Mac OS X는 운영체제가 EFI를 지원하기 때문에 처음부터 LBA1인 GPT 헤더에 접근하기 때문이다.<br /><br />맥은 Mac
    OS X의 애플리케이션만 사용함으로 인해 생기는 불편함을 해소하기 위해 윈도우 운영체제를 네이티브로 구동할 수 있는 기술을 지원하며, 이를
    부트캠프(Boot Camp)라고 부른다. 부트캠프에는 윈도우 운영체제를 설치할 수 있는데, 이 윈도우 운영체제의 경우엔 윈도우 비스타 SP1
    이상부터 EFI를 지원하고 있으며, xp를 설치할 경우엔 GPT를 인식 못하는 문제가 발생할 수 있다.<br /><br />그래서 EFI-GPT
    구조에선 이러한 GPT 비지원 운영체제를 위해 LBA 0에 MBR 영역을 보존하고 있으며, 이 영역을 통해 비지원 운영체제의 부팅을 수행한다.
    이런 식으로 GPT와 기존 MBR 둘 다 지원할 수 있는 구조를 하이브리드 MBR이라고 부른다. 맥에선 gptsync 프로그램으로 어떤 파티션을
    MBR로 동기화할지를 지정할 수 있다.<br /><br />또한 위 MBR 영역에서는 부트코드 영역이 NULL로 작성되어 있는데, 이 부분은
    EFI에서 BIOS 에뮬레이션을 통해 에뮬레이트 된 바이오스가 내장한 부트 코드를 수행하도록 설계되어 있는 것으로 알려져 있다.<br /><br
    />위 예제의 경우엔 별도의 부트캠프가 설정되지 않은 시스템으로 GPT 헤더의 위치를 가리키고 있으며, 부트 캠프가 설치된다면, 저 정보는
    변경될 것이다.<br /><br />MBR 영역을 분석하는 도구는 여러가지가 있는데, 최근에 포렌식적으로 유용한 정보를 제공해주는 파이썬 코드가
    하나 올라온 것이 있다. 다음 링크의 내용을 참조하기 바란다.<br /><br /><span>Posting : <a href="http://gleeda.blogspot.kr/2012/04/mbr-parser.html">http://gleeda.blogspot.kr/2012/04/mbr-parser.html</a></span><br
    /><span>Script Link: https://raw.github.com/gleeda/misc-scripts/master/misc_python/mbr_parser.py</span><br
    /><br />다음 포스팅에선 GPT Header에 대해 알아보겠다. :-)<br /><br /><br /><div>n0fate's Forensic
    Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>Mac OS X 포렌식에 관심이 있다면, 바이오스가 아닌 EFI를 활용한다는 것은 알고 있을 것이다. 윈도우 운영체제도 비스타 SP1부터 EFI-GPT 를 지원하면서 최근 출시되는 많은 노트북에서 바이오스가 사라지고 있다. 기존 바이오스와 비교해서 새로운 부트 플랫폼 구조는 다음과 같이 매치 된다.</p>
<p>BIOS(Bootable Input/Output Service) -> EFI(Extensible Firmware Interface)<br />MBR(Master Boot Record) -> GPT(Guid Partition Table)</p>
<p>이번 포스팅을 시작으로 GPT의 구조에 대해 알아보려고 한다.  우선 GPT의 기본적인 구조는 다음과 같다.</p>
<div><img alt="gpt structure.png" border="0" height="213" src="{{ site.baseurl }}/assets/gpt%252520structure.png?imgmax=800" title="gpt structure.png" width="600" /></div>
<div><span><b><guid partition table - mac os x lion /></b></span></div>
<p>보통 GPT를 설명하는 문서에는 LBA라는 표현을 사용한다. LBA는 논리 블록 주소(Logical Block Address)로 특정 바이트 사이즈를 하나의 블록으로 잡아 표현한다. GPT의 경우엔 512바이트를 하나의 블록으로 설정하며, 이에 Protective MBR 영역은 LBA 0 이라고 표현한다. Mac OS X는 이 Protective MBR 영역에 하나의 주 파티션 테이블 정보를 가지고 있다.</p>
<div><img alt="mbr.png" border="0" height="495" src="{{ site.baseurl }}/assets/mbr.png?imgmax=800" title="mbr.png" width="600" /></div>
<div><span><b>
<protective mbr area /></b></span></div>
<p>MBR의 코드, 디스크 시그너처, 영역이 전부 NULL 상태이며, 0x1BE 부터 시작하는 주 파티션 테이블에만 정보가 존재한다. 주 파티션 테이블엔 보통 4개의 파티션 엔트리가 위치할 수 있는데 본 예제의 경우엔 하나의 파티션 정보를 가지고 있다.
<ul>
<li>0x1BE - 0x00 - Non Bootable</li>
<li><span><strong>0x1BF ~ 0x1C1 - 0xFE, 0xFF, 0xFF - Starting CHS address - 사용되지 않음</strong></span></li>
<li><strong>0x1C2 - 0xEE - EFI GPT Partition Type (Hybrid MBR)</strong></li>
<li><span><strong>0x1C3 ~ 0x1C5 - 0xFE, 0xFF, 0xFF - Ending CHS address - 사용되지 않음</strong></span></li>
<li>0x1C6 ~ 0x1C9 - 0x01 0x00 0x00 0x00 - Starting LBA Address(0x01)</li>
<li>0x1CA ~ 0x1CD - 0xAF, 0xC2, 0xE7, 0x0E - Total sectors(0xEE7C2AF * 0x200bytes(sector size) --> 128,035,675,648bytes --> 120GB)</li>
</ul>
<p>각 요소는 다음과 같이 해석된다.
<ul>
<li>0x1BE - Non Bootable - 부팅되지 않는 파티션이란 의미로, GPT 헤더를 가리킬 경우엔 보통 0x00을 사용한다.</li>
<li><b><span>0x1BF ~ 0x1C1 - 사용되지 않음</span></b></li>
<li>0x1C2 - EFI GPT Partition Type은 해당 주 파티션이 GPT 형식으로 관리를 하고 있다는 의미로, 하이브리드 MBR로 운용할 경우 설정하는 정보이다.(http://www.rodsbooks.com/gdisk/hybrid.html). Mac OS X만 설치된 경우엔 GPT 기반으로 동작하므로 GPT 영역 정보를 가지는 하나의 엔트리만 존재하게 된다.</li>
<li><span><b>0x1C3 ~ 0x1C5 - 사용되지 않음</b></span></li>
<li>0x1C6 ~ 0x1C9 - Starting LBA Address가 0x01로 LBA 1부터 GPT 영역임을 의미한다. GPT 파티션 관리 구조에서는 보통 LBA 1에 주 GPT 헤더가 위치한다.</li>
<li>0x1CA ~ 0x1CD - Total Sector Size는 총 몇 개의 섹터가 해당 파티션에 할당되어 있는지를 말하는 것으로 예제의 경우엔 전체 디스크 사이즈만큼을 할당하여 사용한다.</li>
</ul>
<p>기존 MBR을 공부한 사람이라면, 다른 부분은 이해할 수 있지만 하이브리드 MBR은 상당히 생소할 것이다. 사실 EFI가 지원되는 맥 시스템(최근 맥 시스템은 모두 지원함)에서 Mac OS X만 구동된다면, MBR 영역은 필요 없다. Mac OS X는 운영체제가 EFI를 지원하기 때문에 처음부터 LBA1인 GPT 헤더에 접근하기 때문이다.</p>
<p>맥은 Mac OS X의 애플리케이션만 사용함으로 인해 생기는 불편함을 해소하기 위해 윈도우 운영체제를 네이티브로 구동할 수 있는 기술을 지원하며, 이를 부트캠프(Boot Camp)라고 부른다. 부트캠프에는 윈도우 운영체제를 설치할 수 있는데, 이 윈도우 운영체제의 경우엔 윈도우 비스타 SP1 이상부터 EFI를 지원하고 있으며, xp를 설치할 경우엔 GPT를 인식 못하는 문제가 발생할 수 있다.</p>
<p>그래서 EFI-GPT 구조에선 이러한 GPT 비지원 운영체제를 위해 LBA 0에 MBR 영역을 보존하고 있으며, 이 영역을 통해 비지원 운영체제의 부팅을 수행한다. 이런 식으로 GPT와 기존 MBR 둘 다 지원할 수 있는 구조를 하이브리드 MBR이라고 부른다. 맥에선 gptsync 프로그램으로 어떤 파티션을 MBR로 동기화할지를 지정할 수 있다.</p>
<p>또한 위 MBR 영역에서는 부트코드 영역이 NULL로 작성되어 있는데, 이 부분은 EFI에서 BIOS 에뮬레이션을 통해 에뮬레이트 된 바이오스가 내장한 부트 코드를 수행하도록 설계되어 있는 것으로 알려져 있다.</p>
<p>위 예제의 경우엔 별도의 부트캠프가 설정되지 않은 시스템으로 GPT 헤더의 위치를 가리키고 있으며, 부트 캠프가 설치된다면, 저 정보는 변경될 것이다.</p>
<p>MBR 영역을 분석하는 도구는 여러가지가 있는데, 최근에 포렌식적으로 유용한 정보를 제공해주는 파이썬 코드가 하나 올라온 것이 있다. 다음 링크의 내용을 참조하기 바란다.</p>
<p><span>Posting : <a href="http://gleeda.blogspot.kr/2012/04/mbr-parser.html">http://gleeda.blogspot.kr/2012/04/mbr-parser.html</a></span><br /><span>Script Link: https://raw.github.com/gleeda/misc-scripts/master/misc_python/mbr_parser.py</span></p>
<p>다음 포스팅에선 GPT Header에 대해 알아보겠다. :-)</p>
<p>
<div>n0fate's Forensic Space :)</div>
