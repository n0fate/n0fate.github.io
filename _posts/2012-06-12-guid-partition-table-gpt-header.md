---
layout: post
title: 'Guid Partition Table : GPT Header'
date: 2012-06-12 17:06:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Boot Strap
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/06/guid-partition-table-gpt-header.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/2700289432752043047
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1718'
  fusion_builder_content_backup: '이번 포스팅에선 GPT 중 GPT Header에 대해 알아볼 것이다. GPT Header는
    LBA 1에 위치한 데이터 구조체로 Primary Partition Table 영역 중 맨 앞에 위치한다.<br /><br /><div><img
    alt="gpt structure.png" border="0" height="213" src="http://lh6.ggpht.com/-K4aSTP3533s/T9V2rGEYRzI/AAAAAAAAAR8/pqf1P71P3_M/gpt%252520structure.png?imgmax=800"
    title="gpt structure.png" width="600"></div><div><span><gpt Structure></span></div><br
    />Primary Partition Table에는 GPT Header 뒤에 128개의 테이블 엔트리가 위치하고 있다. 이전에 설명한 것과 같이
    LBA는 Sector Size와 동일한 512바이트를 가지며 각각 다음과 같은 정보를 가진다.<br /><ul><li>Signature(8bytes)
    - 해당 영역을 설명하는 시그너처</li><li>Revision(4) - 리비전 넘버</li><li>Header Size(4) - GPT Header의
    크기(보통 512바이트)</li><li>CRC32 of Header(4) - 해당 4바이트를 0으로 설정했을 때, GPT Header의 CRC32
    값</li><li>Reserved(4) - 예약 영역</li><li>Current LBA(8) - 현재 LBA, 즉 Primary GPT Header의
    LBA</li><li>Backup LBA(8) - 백업 LBA, 즉 Secondary GPT Header의 LBA</li><li>First
    usable LBA for partitions(8) - GPT로 정의된 디스크 영역에서 사용할 LBA 시작 위치</li><li>Last usable
    LBA for partitions(8)- GPT로 정의된 디스크 영역에서 사용할 LBA 끝 위치</li><li>Disk GUID(16) -
    Disk GUID로 유닉스의 리틀-엔디안 UUID 포맷을 따름</li><li>Partition entries starting LBA(8) -
    파티션 엔트리가 시작하는 LBA 위치</li><li>Number of partition entries(4) - 파티션 엔트리의 갯수</li><li>Size
    of partition entry(4) - 파티션 엔트리의 크기</li><li>CRC32 of partition array(4) - 파티션
    엔트리 그룹의 CRC32 값으로 파티션 엔트리의 변조 여부를 확인 가능</li></ul>각 정보를 토대로 실제 디스크에서 확인해보겠다.<br
    /><div><img alt="Gpt header" border="0" height="498" src="http://lh5.ggpht.com/-6l1bEL7xhBU/T9b2B6kXESI/AAAAAAAAASQ/r5TjghLpQ-w/gpt%252520header.jpg?imgmax=800"
    title="gpt header.jpg" width="600"></div><div><span><gpt Header Area></span></div><br
    />GPT는 EFI 표준과 함께 등장한 개념이다보니 인텔이 제정한 표준을 따른다. 그로 인해 스펙 문서에 정의된 구조체가 디스크에 쓰여질 땐
    항상 리틀-엔디안으로 작성한다.<br />이제 각 항목을 매치하여 값을 확인해보겠다. 이 과정에선 본 저자가 만든 도구를 활용하였다.<br
    /><br /><span>gpt parser - <a href="https://github.com/n0fate/raw/blob/master/gpt_parser.py">https://github.com/n0fate/raw/blob/master/gpt_parser.py</a></span><br
    /><br /><div><img alt="Gpt parser1" border="0" height="274" src="http://lh5.ggpht.com/-s1h1Nj1yQQE/T9b2FqFcw9I/AAAAAAAAASY/0DArJR-BHOo/gpt%252520parser1.jpg?imgmax=800"
    title="gpt parser1.jpg" width="534"></div><div><span><gpt Parser></span></div><br
    />각 요소는 다음과 같이 해석된다.<br /><br /><ul><li>일단 시그너처가 ''EFI PART(ITION)''이다. 시그너처를
    특별한 의미를 가지지 않지만, 이름으로써 바이오스가 아닌 EFI 기반으로 동작하는 시스템에서 만든 GPT 테이블이란걸 생각할 수 있다.</li><li>리비전
    번호는 특별한 의미를 지니진 않지만, UEFI로 넘어오면서부터는 기존 EFI 1.10에서 사용했던 리비전 번호(65536)을 그대로 사용한다고
    한다.</li><li>헤더 사이즈는 LBA 1 영역인 512바이트 중에 실제 GPT 헤더 영역을 의미하는 것으로 92바이트를 사용한다.</li><li>헤더의
    CRC32 값은 ''CRC32 of header'' 영역 4바이트를 0x00000000으로 설정하고, 헤더 사이즈(예제에선 92)만큼을 잘라내어
    산출한 값이다. EFI는 GPT Header의 CRC32 값이 틀린 경우, Secondary GPT Header의 정보를 이용하여 복구를 수행한다.</li><li>Current
    LBA와 Backup LBA는 각각 Primary Partition Table과 Backup Partition Table을 의미하는 것으로
    실제 디스크 이미지에서 해당 영역에 접근할 경우엔 LBA 값에 Logical Block Size(512바이트)를 곱한 위치를 찾아가야 한다.
    그 다음에 나타나는 파티션에서 사용할 수 있는 첫 번째 LBA와 마지막 LBA는 디스크 상에서 GPT 헤더와 같은 관리 구조체를 제외하고 실제
    데이터를 쓸 수 있는 영역을 표현한다.</li><li>Disk GUID는 해당 Disk에 대한 고유 식별자로 URN(Uniform Resource
    Name) 네임스페이스(namespace)를 따르고 있으며, RFC 4122에 명시되어 있는 규약이다. 입력 값이 리틀-엔디안이므로 해석할
    때도 리틀-엔디안 방식으로 해석하여 표현해야 한다. (link: http://docs.python.org/library/uuid.html)</li><li>파티션
    엔트리 시작 LBA는 2로 GPT 헤더 바로 뒤에 이어짐을 알 수 있다. 이 LBA를 시작으로 파티션 엔트리 사이즈인 0x80(128바이트)씩
    파티션 엔트리 수(128개)만큼 사용하게 된다. 즉, LBA하나당 4개의 파티션 엔트리가 올 수 있으며, 총 128개의 엔트리가 할당되어 있으므로,
    32개의 LBA가 파티션 엔트리에 사용된다. Protective MBR과 GPT Header가 가지는 각 LBA를 합치면 총 34개(0x22)로
    파티션 엔트리 뒤에 파티션에서 사용할 수 있는 첫 번째 LBA가 위치함을 확인할 수 있다.</li></ul><br />위 예제와 같이 GPT
    헤더를 분석하면, 헤더의 CRC32 체크를 통해 무결성을 확인(CRC32 자체가 완벽할 순 없지만)할 수 있으며, 데이터 작성이 가능한 공간을
    확인하고, 백업 파티션 테이블의 위치와 디스크 GUID, 파티션 엔트리의 시작 위치를 파악할 수 있다.<br /><br />파티션 위치를 파악했으므로,
    다음 포스팅에선 파티션 엔트리 분석을 통해 실제 각 파티션 영역의 정보를 파악해보겠다. :)<br /><br /><br /><br /><br
    /><br /><div>n0fate''s Forensic Space :)</div>'
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>이번 포스팅에선 GPT 중 GPT Header에 대해 알아볼 것이다. GPT Header는 LBA 1에 위치한 데이터 구조체로 Primary Partition Table 영역 중 맨 앞에 위치한다.</p>
<div><img alt="gpt structure.png" border="0" height="213" src="{{ site.baseurl }}/assets/gpt%252520structure.png?imgmax=800" title="gpt structure.png" width="600" /></div>
<div><span><gpt structure /></span></div>
<p>Primary Partition Table에는 GPT Header 뒤에 128개의 테이블 엔트리가 위치하고 있다. 이전에 설명한 것과 같이 LBA는 Sector Size와 동일한 512바이트를 가지며 각각 다음과 같은 정보를 가진다.
<ul>
<li>Signature(8bytes) - 해당 영역을 설명하는 시그너처</li>
<li>Revision(4) - 리비전 넘버</li>
<li>Header Size(4) - GPT Header의 크기(보통 512바이트)</li>
<li>CRC32 of Header(4) - 해당 4바이트를 0으로 설정했을 때, GPT Header의 CRC32 값</li>
<li>Reserved(4) - 예약 영역</li>
<li>Current LBA(8) - 현재 LBA, 즉 Primary GPT Header의 LBA</li>
<li>Backup LBA(8) - 백업 LBA, 즉 Secondary GPT Header의 LBA</li>
<li>First usable LBA for partitions(8) - GPT로 정의된 디스크 영역에서 사용할 LBA 시작 위치</li>
<li>Last usable LBA for partitions(8)- GPT로 정의된 디스크 영역에서 사용할 LBA 끝 위치</li>
<li>Disk GUID(16) - Disk GUID로 유닉스의 리틀-엔디안 UUID 포맷을 따름</li>
<li>Partition entries starting LBA(8) - 파티션 엔트리가 시작하는 LBA 위치</li>
<li>Number of partition entries(4) - 파티션 엔트리의 갯수</li>
<li>Size of partition entry(4) - 파티션 엔트리의 크기</li>
<li>CRC32 of partition array(4) - 파티션 엔트리 그룹의 CRC32 값으로 파티션 엔트리의 변조 여부를 확인 가능</li>
</ul>
<p>각 정보를 토대로 실제 디스크에서 확인해보겠다.
<div><img alt="Gpt header" border="0" height="498" src="{{ site.baseurl }}/assets/gpt%252520header.jpg?imgmax=800" title="gpt header.jpg" width="600" /></div>
<div><span><gpt header area /></span></div>
<p>GPT는 EFI 표준과 함께 등장한 개념이다보니 인텔이 제정한 표준을 따른다. 그로 인해 스펙 문서에 정의된 구조체가 디스크에 쓰여질 땐 항상 리틀-엔디안으로 작성한다.<br />이제 각 항목을 매치하여 값을 확인해보겠다. 이 과정에선 본 저자가 만든 도구를 활용하였다.</p>
<p><span>gpt parser - <a href="https://github.com/n0fate/raw/blob/master/gpt_parser.py">https://github.com/n0fate/raw/blob/master/gpt_parser.py</a></span></p>
<div><img alt="Gpt parser1" border="0" height="274" src="{{ site.baseurl }}/assets/gpt%252520parser1.jpg?imgmax=800" title="gpt parser1.jpg" width="534" /></div>
<div><span><gpt parser /></span></div>
<p>각 요소는 다음과 같이 해석된다.</p>
<ul>
<li>일단 시그너처가 'EFI PART(ITION)'이다. 시그너처를 특별한 의미를 가지지 않지만, 이름으로써 바이오스가 아닌 EFI 기반으로 동작하는 시스템에서 만든 GPT 테이블이란걸 생각할 수 있다.</li>
<li>리비전 번호는 특별한 의미를 지니진 않지만, UEFI로 넘어오면서부터는 기존 EFI 1.10에서 사용했던 리비전 번호(65536)을 그대로 사용한다고 한다.</li>
<li>헤더 사이즈는 LBA 1 영역인 512바이트 중에 실제 GPT 헤더 영역을 의미하는 것으로 92바이트를 사용한다.</li>
<li>헤더의 CRC32 값은 'CRC32 of header' 영역 4바이트를 0x00000000으로 설정하고, 헤더 사이즈(예제에선 92)만큼을 잘라내어 산출한 값이다. EFI는 GPT Header의 CRC32 값이 틀린 경우, Secondary GPT Header의 정보를 이용하여 복구를 수행한다.</li>
<li>Current LBA와 Backup LBA는 각각 Primary Partition Table과 Backup Partition Table을 의미하는 것으로 실제 디스크 이미지에서 해당 영역에 접근할 경우엔 LBA 값에 Logical Block Size(512바이트)를 곱한 위치를 찾아가야 한다. 그 다음에 나타나는 파티션에서 사용할 수 있는 첫 번째 LBA와 마지막 LBA는 디스크 상에서 GPT 헤더와 같은 관리 구조체를 제외하고 실제 데이터를 쓸 수 있는 영역을 표현한다.</li>
<li>Disk GUID는 해당 Disk에 대한 고유 식별자로 URN(Uniform Resource Name) 네임스페이스(namespace)를 따르고 있으며, RFC 4122에 명시되어 있는 규약이다. 입력 값이 리틀-엔디안이므로 해석할 때도 리틀-엔디안 방식으로 해석하여 표현해야 한다. (link: http://docs.python.org/library/uuid.html)</li>
<li>파티션 엔트리 시작 LBA는 2로 GPT 헤더 바로 뒤에 이어짐을 알 수 있다. 이 LBA를 시작으로 파티션 엔트리 사이즈인 0x80(128바이트)씩 파티션 엔트리 수(128개)만큼 사용하게 된다. 즉, LBA하나당 4개의 파티션 엔트리가 올 수 있으며, 총 128개의 엔트리가 할당되어 있으므로, 32개의 LBA가 파티션 엔트리에 사용된다. Protective MBR과 GPT Header가 가지는 각 LBA를 합치면 총 34개(0x22)로 파티션 엔트리 뒤에 파티션에서 사용할 수 있는 첫 번째 LBA가 위치함을 확인할 수 있다.</li>
</ul>
<p>위 예제와 같이 GPT 헤더를 분석하면, 헤더의 CRC32 체크를 통해 무결성을 확인(CRC32 자체가 완벽할 순 없지만)할 수 있으며, 데이터 작성이 가능한 공간을 확인하고, 백업 파티션 테이블의 위치와 디스크 GUID, 파티션 엔트리의 시작 위치를 파악할 수 있다.</p>
<p>파티션 위치를 파악했으므로, 다음 포스팅에선 파티션 엔트리 분석을 통해 실제 각 파티션 영역의 정보를 파악해보겠다. :)</p>
<div>n0fate's Forensic Space :)</div>
