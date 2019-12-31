---
layout: post
title: REGIN의 은닉 기법 및 대응 방법
date: 2015-01-01 22:48:12.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- OS Artifacts
tags:
- incident response
- malware
- regin
author: "n0fate"
---
<h2 id="침해-대응-관점의-regin-분석">I. 서론<br />
<a href="#침해-대응-관점의-regin-분석" name="침해-대응-관점의-regin-분석"></a></h2>
<p>2014년 말에는 차세대 Stuxnet이라 불리는 Regin가 전세계 정보 보안 분야를 핫하게 만들었다. (이 글을 쓰는 시점에 소니 픽처스와 대한민국의 한수원 사태 덕에 묻혀버렸지만..) Regin은 사이버 첩보(Cyber espioinage) 임무를 수행하는 악성코드로 기존 사이버전 악성코드와 다르게 프레임워크 구조를 가지고 있었다. 프레임워크 구조라는 것은 다른게 아니라, 각 기능을 파일(플러그인 모듈)단위로 쪼개두고 이를 자동 또는 수동으로 동적 로딩할 수 있는 형태를 말한다.</p>
<p>이러한 프레임워크 기반의 악성코드는 여러 모듈이 시스템에 흔적으로 남겨질 수 있다보니, 다양한 방법으로 모듈을 은닉하여 시스템 상태 모니터링 도구를 우회하거나, 포렌식을 통한 역추적을 최소화하도록 한다. 본 포스팅에서는 Regin의 다양한 은닉 기법을 알아보고 침해 대응 관점에서 어떻게 이를 탐지해야하는지 알아보도록 한다. 여기서 설명하는 Regin의 대응 방법은 명확하지 않으며, '추 후 침해사고 시스템을 분석 시 이러이러한 경우이렇게 한번 해봐라' 정도의 대안만 제시할 수 있음을 염두해 두길 바란다. (사실 명확하게 나올 수가 없다.)</p>
<p>또한, 본 글은 Regin에 대한 상세한 내용을 기술하진 않았으므로, 기본적인 행위에 대한 이해가 이루어진 후에 읽어보길 추천한다. 이미 <a href="http://www.symantec.com/content/en/us/enterprise/media/security_response/whitepapers/regin-analysis.pdf" target="_blank">시만텍</a>과 <a href="http://securelist.com/files/2014/11/Kaspersky_Lab_whitepaper_Regin_platform_eng.pdf" target="_blank">카스퍼스키</a>엔 훌륭한 보고서가 나와있다.</p>
<h2 id="은닉-기능"><a href="#은닉-기능" name="은닉-기능"></a>II. 은닉 기능</h2>
<h3 id="기존-스테이지의-악성-행위-제거"><a href="#기존-스테이지의-악성-행위-제거" name="기존-스테이지의-악성-행위-제거"></a>1. 기존 스테이지의 악성 행위 제거</h3>
<p>Regin은 매 단계를 수행할 때마다 이전 단계에 남겨둔 흔적을 제거하는 기능을 가지고 있다. 이러한 기능으로 분석가가 악성코드를 분석하더라도, 악성코드의 기능 파악을 쉽게할 수 없게 된다. 이 경우 분석가는 디스크 이미징을 수행하고, 이미징에서 파일 카빙을 통해 악성코드의 후보군을 추출, 관련있는 악성코드를 추려내는 작업이 필요하다. 이려한 일련의 작업은 분석가 입장에서는 많은 노동 시간으로 낮은 효율을 보여주지만 공격자 입장에선 최고의 효율을 보여주는 방법이라 볼 수 있다. 안티포렌식을 고려한 악성코드는 기존 스테이지에서 남긴 악성 행위를 제거하는 방법과 다른 은닉 기법을 같이 결합하여 사용하기도 하며, 데이터를 와이핑(Wiping)하여 복구 자체를 불가능하게 만드는 방법도 사용된다.</p>
<h4 id="regin의-악성-행위-제거"><a href="#regin의-악성-행위-제거" name="regin의-악성-행위-제거"></a>A. Regin의 악성 행위 제거</h4>
<p>Regin은 바이너리 내에 저장된 인코딩된 설정 파일을 디코딩하여, 관련 정보를 추출하고 행위를 제거한다. 예를 들어 스테이지1의 설정 정보에 대한 디코딩 스크립트는 다음과 같다.</p>
<pre><code>xorkey = [0x33, 0x10, 0x15, 0xEA, 0x26, 0x1D, 0x38, 0xA7]
SourceString = [0x6F, 0x43, 0x52, 0xae, 0x6b, 0x4b, 0x6a, 0xf2, 0x62, 0x45, 0x52, 0x80, 0x49, 0x78, 0x5F, 0xC6] # 0x15009, Example
decoded = []

count = 0

while count &lt; 750:
    decoded = (SourceString[count] ^ xorkey[count%8]) ^ count
    count += 1

    print '%c'%decoded
</code></pre>
<p>위 스크립트를 통해 데이터 영역을 디코딩하면, 파일이나 레지스트리 경로 정보를 볼 수 있다.</p>
<p><img class="aligncenter size-full wp-image-1382" src="{{ site.baseurl }}/assets/decodestr.png" alt="decodestr" width="872" height="107" /></p>
<p>스테이지1은 위 스크립트에 표시된 디렉터리 경로나 레지스트리 경로에서 스테이지2를 덤프하여 실행한다. 그리고 스테이지2가 실행되면, 스테이지1가 생성한 스테이지2의 데이터를 제거함으로 탐지를 최소화하고 역추적을 방해한다.</p>
<p>&nbsp;</p>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>이전 악성 행위를 대응하려면, 앞서 잠깐 언급한 디스크 카빙 기법이 필요하게 된다.<br />
만약 사내에 여러 시스템이 감염되어 있다면, 몇몇 시스템에서는 알수 없는 이유로 이전 스테이지의 악성 코드가 남아있을 수 있으므로 해당 자원을 체크하는 것도 도움이 된다.</p>
<p>&nbsp;</p>
<h3 id="ntfs-확장-속성(extended-attributes)에-악성코드-저장"><a href="#ntfs-확장-속성(extended-attributes)에-악성코드-저장" name="ntfs-확장-속성(extended-attributes)에-악성코드-저장"></a>2. NTFS 확장 속성(Extended Attributes)에 악성코드 저장</h3>
<p>스트림은 바이트 배열로, NTFS 파일 시스템에서는 파일에 대한 더 많은 정보를 제공하기 위한 속성(attributes)와 정보(properties)가 저장된다. 예를 들면, 키워드 검색이나 파일을 생성한 사용자 계정 식별자를 스트림으로 만들 수 있다.<br />
NTFS에는 여러개의 스트림 타입이 있는데, Regin이 사용한 ‘$EA’ 는 확장 속성 데이터를 정의하는 영역이다. 이 스트림은 HPFS(<a href="http://en.wikipedia.org/wiki/High_Performance_File_System">High Performance File System</a>)의 확장 속성을 따서 만든 것이며, 스트림의 크기를 조정할 수 있기 떄문에 non-resident 속성을 가진다. 다음은 스트림 구조체를 표현한 것이다 (<a href="http://0cch.net/ntfsdoc/attributes/ea.html">NTFS $EA Format</a>).</p>
<table style="border: 1px solid black; align: center;" border="1">
<thead>
<tr>
<th>위치</th>
<th>크기</th>
<th>설명</th>
</tr>
</thead>
<tbody>
<tr>
<td>0x00</td>
<td>4</td>
<td>다음 확장 속성의 위치</td>
</tr>
<tr>
<td>0x04</td>
<td>1</td>
<td>플래그</td>
</tr>
<tr>
<td>0x05</td>
<td>1</td>
<td>이름 길이(N)</td>
</tr>
<tr>
<td>0x06</td>
<td>2</td>
<td>값 길이(V)</td>
</tr>
<tr>
<td>0x08</td>
<td>N</td>
<td>이름</td>
</tr>
<tr>
<td>N+0x08</td>
<td>V</td>
<td>값</td>
</tr>
</tbody>
</table>
<p>$EA는 위에서 설명한 것처럼 non-resident 속성을 가지며, 구조체를 단일 연결 리스트(Single Linked List)로 관리한다. 각 구조체의 Name 정보를 다르게하여 다른 속성 정보를 여러개 생성할 수 있다. 이 방법은 ZeroAccess 악성코드에서 최초 발견된 방법(<a href="http://www.hexacorn.com/blog/2013/01/25/detecting-extended-attributes-zeroaccess-and-other-frankensteins-monsters-with-hmft/" target="_blank">Detecting Extended Attributes and other Frankenstein's Monters with HMFT</a>)이며, Regin 또한 동일한 방법을 사용하였다.</p>
<h4 id="regin의-은닉기법"><a href="#regin의-은닉기법" name="regin의-은닉기법"></a>A. Regin의 은닉기법</h4>
<p>Regin은 이 방법을 이용하여 악성코드를 은닉하였다. $EA 영역에 드롭퍼가 다음 스테이지의 인코딩된 바이너리를 저장하고, 실행 시점에 ntoskrnl.exe의 Undocumented API 인 NtQueryEAFile를 이용하여 해당 파일 스트림을 추출한다.</p>
<p><img class="aligncenter size-full wp-image-1347" src="{{ site.baseurl }}/assets/regin_006.png" alt="regin_006" width="557" height="177" /></p>
<p>대상 경로는 다음과 같다.</p>
<ul>
<li>WINDOWS</li>
<li>WINDOWSfonts</li>
<li>WINDOWSCursors</li>
</ul>
<p>Undocumented API 사용으로 인한 탐지를 피하기 위해 API 이름은 XOR로 인코딩하여 보관하며, 호출이 필요한 시점에 동적으로 로드하도록 구성하였다. 32비트인 경우 대략적인 코드는 다음과 같다.</p>
<pre><code>Stream = ''    # Stream is Next Encoded Stage (Binary)
Handle = ZwCreateFile(FILEPATH, FILE_READ_EA|FILE_READ_DATA, ...);
DecodeStr();    # Decode a byte array for extracting extended attributes
EncNtQueryEaFile = GetNtQueryEaFileAddr('ntoskrnl.exe');
EncNtQueryEaFile(Handle, Buf, IoStatusBlock, NULL, NULL, NULL, TRUE); # EncNtQueryEaFile is decoded by DecodeStr()
do:
    NextPtr = Buf-&gt;Offset;
    NameLen = Buf-&gt;NameLength;
    if NameLen != 1:
        return
    if Buf-&gt;Name[0] != '_':
        return
    ValueLength = Buf-&gt;ValueLen
    Stream += Buf-&gt;Value[:ValueLength]
while NexPtr != NULL
</code></pre>
<p>Decodestr()의 수행결과는 다음과 같다.</p>
<pre><code>$ python decodestr.py
ntoskrnl.exe
KeServiceDescriptorTable
NtQueryEaFile
NtSetEaFile
</code></pre>
<p>Regin은 앞서 설명한 디렉터리를 ZwCreateFile로 오픈하고, 심볼 이름을 기반으로 ntoskrnl.exe 바이너리 내에 있는 NtQueryEaFile API의 시작 주소를 찾는다. 만약 윈도우 2000(NT 4.0)이라면, KeServiceDescriptorTable 주소를 기준으로 해당 함수의 주소를 가져온다. 커널과 드라이버가 동일한 메모리 영역에 있기 때문에 이러한 행위가 가능하다.</p>
<p><img class="aligncenter size-full wp-image-1351" src="{{ site.baseurl }}/assets/regin_0071.png" alt="regin_007" width="1671" height="844" /></p>
<p>획득한 함수 주소를 이용하여 해당 디렉터리에 저장된 EA를 추출한다. 추출한 EA 구조체의 이름 필드 값이 ‘<em>‘가 아니라면, 프로그램을 종료한다. ‘</em>‘ 이라면, 파일 스트림을 추출하고 추출한 스트림을 디코딩하여 메모리에 로드한다.</p>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>과거 악성코드는 NTFS의 파일 스트림 중 하나인 ADS(Alternate Data Stream)을 이용하여 자신을 은닉하였다. 이러한 기법이 잘 알려져있다보니 분석가들이 디스크 이미지를 분석할 때 ADS를 꼭 확인하고 있으며 포렌식 도구에서도 추가 데이터 스트림을 별도의 파일로 표현해주고 있다. $EA도 동일한 방법으로 분석이 가능하다. 실제로 윈도우는 $EA를 사용하지 않기 때문에 이 스트림이 발견된다면 악성코드 후보군으로 삼아도 될 정도이다. 단, 포렌식 도구에서 이 영역을 따로 보여주지 않으므로, 분석 시 이 점을 고려해야한다.</p>
<p>&nbsp;</p>
<h3 id="암호화된-레지스트리-데이터로-악성코드-저장"><a href="#암호화된-레지스트리-데이터로-악성코드-저장" name="암호화된-레지스트리-데이터로-악성코드-저장"></a>3. 암호화된 레지스트리 데이터로 악성코드 저장</h3>
<p>레지스트리는 윈도우 관리를 위한 핵심 요소로, 파일 연결, 서비스 등의 정보를 관리한다. 트리 구조로 이루어져있으며, 여러 정보를 관리할 수 있도록 다양한 포맷의 데이터를 저장(문자열, 정수, 바이너리)할 수 있다. 레지스트리에 악성코드를 저장하는 방법은 TrendMicro에 작성한 <a href="http://blog.trendmicro.com/trendlabs-security-intelligence/poweliks-malware-hides-in-windows-registry/">POWELIKS: Malware Hides In Windows Registry</a> 를 통해 널리 알려졌다.</p>
<h4 id="regin의-기법"><a href="#regin의-기법" name="regin의-기법"></a>A. Regin의 기법</h4>
<p>Regin 32비트 스테이지1은 파일시스템이 NTFS라면, EA에서 스테이지2를 추출하고 아닐 경우, 레지스트리에서 스테이지2를 추출한다. 스테이지2는 레지스트리의 특정 값(Class)에 저장된다.</p>
<ul>
<li>REGISTRYMachineSystemCurrentControlSetControlClass{3939744-44FC-AD65-474B-E4DDF8C7FB91} &lt;- 스테이지2에서 로드</li>
<li>REGISTRYMachineSystemCurrentControlSetControlClass{3F90B1B4-58E2-251E-6FFE-4D38C5631A04} &lt;- 스테이지2에서 로드</li>
<li>REGISTRYMachineSystemCurrentControlSetControlClass{4F20E605-9452-4787-B793-D0204917CA58} &lt;- 스테이지1에서 로드</li>
</ul>
<p>이 경로에는 각 클래스에 대한 설명과 관련 파일, 버전 정보를 기록한다. 악성코드는 여기에 스테이지2의 인코딩된 바이너리터를 저장한다. 인코딩 알고리즘은 EA와 같이 Customized XOR을 수행한다.</p>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>Stuxnet이나 Duqu가 레지스트리를 설정 정보를 저장하는 용도로 사용한 것처럼 Regin도 보통 포렌식 분석가가 접근하지 않는 Class 키에 악성코드 바이너리를 저장하여 탐지를 우회하고 있다. 레지스트리에 저장된 바이너리를 찾는 일은 파일시스템에서 악성행위를 하는 파일을 샘플링하는 것과 같은 난이도를 가질 수 있다. 하지만 다행이도 레지스트리에는 바이너리 형태로 데이터를 저장하는 경우가 많지 않기 때문에, 레지스트리 값에서 바이너리 타입인 ‘REG_BINARY’만 추출하면 후보군을 많이 줄일 수 있다.</p>
<p>또한, Regin이 저장한 값인 Class는 클래스 이름을 정의하는 값이기 때문에, 문자열인 ‘REG_SZ’ 타입으로 저장된다. 분석가는 Class값을 해석하여 ‘REG_BINARY’ 인 값의 데이터만 추출하는 방법을 적용해볼 수도 있다.</p>
<p>&nbsp;</p>
<h3 id="하드디스크-비할당-영역에-악성파일-보관"><a href="#하드디스크-비할당-영역에-악성파일-보관" name="하드디스크-비할당-영역에-악성파일-보관"></a>4. 하드디스크 비할당 영역에 악성파일 보관</h3>
<p>하드디스크의 마지막 섹터 영역은 보통 파티션이 사용하지 않는 영역이 있다. 이 영역을 보통 비사용 영역(Unused Area) 또는 비할당 영역(Unallocated Area)로 부르고 있다. 윈도우7의 경우에는 파티션을 생성하면서 디스크의 끝부분의 일정 섹터 크기를 파티션 변경을 원할히 하기 위해 남겨 두는데, 이 영역이 된다.</p>
<h4 id="regin의-은닉-기법"><a href="#regin의-은닉-기법" name="regin의-은닉-기법"></a>A. Regin의 은닉 기법</h4>
<p>Regin은 악성파일이 파일 시스템에서 드러나지 않도록 핵심 기능을 가진 플러그인을 비할당 영역에 악성코드를 위치하여 필요 시 동적으로 로드하는 방법을 사용한다. 이 방법은 과거에 TDL3나 TDL4에서 사용했던 방법으로 Regin은 이에 더해 비할당 영역에 있는 바이너리를 커스터마이징된 RC5 알고리즘으로 암호화하여 필요할 때 메모리에 맵핑한 후, 복호화하는 방법을 사용하였다.</p>
<p><img class="aligncenter size-full wp-image-1346" src="{{ site.baseurl }}/assets/regin_005.png" alt="regin_005" width="620" height="370" /></p>
<p>이 방법의 가장 큰 장점은 라이브포렌식으로는 악성 파일의 핵심 기능을 추적할 수 없다는 것이다. 운영체제에서 접근 가능한 윈도우 파티션에는 아무런 파일이 존재하지 않으므로, 악성코드를 직접 분석해서 EVFS의 존재를 알아야만 분석이 가능하다.</p>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>이전에 이 기법을 사용한 TDL 패밀리의 경우에는 MBR 루트킷 기법을 이용하여 MBR 코드 실행 시점에 가상 파티션의 코드를 로드하도록 조치하다보니 안티바이러스 제품들이 MBR의 코드 영역의 변조 여부를 판단하여 조기에 차단할 수 있었다. 하지만 Regin의 경우에는 스테이지 로딩 시점에 복호화되어 로드하므로 악성코드가 먼저 발견되어야 차단이 가능하게 된다. 악성코드에서 비할당 영역 접근 여부를 확인하고, 복호화 알고리즘을 해석하여 암호화된 악성코드를 복호화하는 과정이 필요하다.</p>
<p>&nbsp;</p>
<h3 id="암호화된-가상-파일시스템-구성"><a href="#암호화된-가상-파일시스템-구성" name="암호화된-가상-파일시스템-구성"></a>5. 암호화된 가상 파일시스템 구성</h3>
<p>가상 파일 시스템 구성은 필자가 TDL에서 처음으로 확인한 기법으로 프레임워크 구조를 가지는 악성코드가 다수의 플러그인을 효과적으로 은닉하기 위한 방법이다. 피해자 시스템에서 사용되지 않는 디스크 영역에 공격자가 정의한 파일시스템을 구성하여 이 곳에 파일을 저장한다. 그리고 악성코드는 C&amp;C 서버의 명령에 따라 파일시스템을 해석하여 관련 파일을 메모리에 로드한다.</p>
<h4 id="regin의-evfs(encrypted-virtual-file-system)"><a href="#regin의-evfs(encrypted-virtual-file-system)" name="regin의-evfs(encrypted-virtual-file-system)"></a>A. Regin의 EVFS(Encrypted Virtual File System)</h4>
<p>Regin은 스테이지3,4의 커널/사용자 프레임워크에서 사용할 플러그인을 EVFS에 저장한다. Regin은 앞서 설명한 하드디스크 비할당 영역에 악성코드를 은닉했기 때문에 EVFS를 로컬 디스크 영역에 파일 형태로 생성한다. 예를 들어 스테이지3인 커널 프레임워크는 다음 EVFS에 접근한다.</p>
<ul>
<li>%System32%configSecurityAudit.evt</li>
<li>%System32%configSystemAudit.evt</li>
</ul>
<p>각 파일이 파일시스템 구조를 가지고 있으며, 내부에는 여러 파일을 컨테이너(Container)라는 이름으로 보관한다. 각 파일은 RC5 암호화 알고리즘 + NV2e 압축 알고리즘으로 저장되며, 각각의 파일의 메타 정보는 FAT 파일시스템의 FAT 테이블과 유사한 형태로 관리한다.</p>
<p><img class="aligncenter size-full wp-image-1345" src="{{ site.baseurl }}/assets/regin_004.png" alt="regin_004" width="661" height="433" /></p>
<p>또한 각 컨테이너는 별도의 파일 이름을 가지고 있지 않으며, 메이저/마이너 ID를 이용하여 파일을 식별한다. 메이저 ID는 각 파일의 카테고리 형태로 나누고 있으며, 마이너 ID 가 각 세부 기능을 정의한다.</p>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>Regin의 EVFS는 일반적인 파일 포맷의 구조와 상이하기 때문에, 탐지 자체는 용이하다. 특히 Regin은 evt라는 이벤트로그 확장자를 사용하기 때문에, Encase와 같은 도구를 이용한다면, 확장자와 파일포맷이 상이한 파일로 추출할 수 있다. 단, EVFS는 말 그대로 파일 시스템의 각 컨테이너가 커널 프레임워크 바이너리 내의 키로 암호화되어 있고 커스터마이징된 암호화 알고리즘을 사용할 수 있으므로, 바이너리 분석 과정이 추가적으로 필요하다.</p>
<p>&nbsp;</p>
<h3 id="메모리에-로드한-바이너리의-pe-헤더-제거"><a href="#메모리에-로드한-바이너리의-pe-헤더-제거" name="메모리에-로드한-바이너리의-pe-헤더-제거"></a>6. 메모리에 로드한 바이너리의 PE 헤더 제거</h3>
<p>최근에 고도화된 악성코드는 시스템에 자신의 흔적을 남기지 않기 위해 메모리에 악성코드를 로드하고 파일시스템에 있는 데이터를 제거하였다. 악성코드 메모리 상주 기법은 포렌식 분석을 까다롭게 만드는 훌륭한 안티 포렌식 기법이다.</p>
<h4 id="regin의-메모리-상주-기법"><a href="#regin의-메모리-상주-기법" name="regin의-메모리-상주-기법"></a>A. Regin의 메모리 상주 기법</h4>
<p>Regin은 NTFS의 EA 영역이나 레지스티리의 특정 값에 인코딩된 악성코드를 보관하고 있다가, 메모리에서 코드를 로드하고 디코딩한 후, 파일시스템에 있는 악성코드를 삭제한다. 그것만하면 최근 메모리 상주 악성코드와 동일한 방법이다. 하지만 분석가가 메모리에서 MZ 시그니처와 같이 PE 파일 헤더 패턴을 이용하여 악성코드를 탐지하는 방법이 존재했다. Regin은 이러한 탐지를 피하기 위해 꼭 필요한 정보만 메모리에 보관하고 로드한 PE파일의 헤더 전체를 제거하였다. 재미있는 점은 자기자신도 이미 로드한 악성코드를 찾을 수 없기 때문에, 파일의 맨 처음 4바이트를 특정 시그니처(0xfedcfedd)를 기록하였다.</p>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>Regin과 같은 메모리 상주 악성코드를 탐지하기 위한 가장 좋은 방법은 메모리 포렌식이다. 메모리 포렌식 도구는 운영체제가 로더를 통해 로드하지 않은 PE 파일을 시그니처 기반으로 찾는 기능을 제공한다. 하지만 Regin을 개발한 사람도 이 기능을 알고있다보니, 탐지 우회를 위해 PE파일의 헤더를 NULL로 제거한다.이러한 기능으로 시그니처 기반으로 PE파일을 찾는 포렌식 도구를 우회할 수 있다.</p>
<p><img class="aligncenter size-full wp-image-1343" src="{{ site.baseurl }}/assets/regin_002.png" alt="regin_002" width="833" height="438" /><br />
이 악성코드를 추적하려면, 정상적으로 로드된 PE 파일의 코드섹션을 제외한 메모리 페이지의 권한을 확인해야한다. 그 후, 제외한 메모리 페이지의 권한이 R*X(PAGE_READ/WRITE_EXECUTE)라면, 메모리 페이지를 의심영역으로 식별하여 추출한다.</p>
<p><img class="aligncenter size-full wp-image-1344" src="{{ site.baseurl }}/assets/regin_003.png" alt="regin_003" width="880" height="414" /></p>
<p>추출한 메모리 영역을 디스어셈블하여 함수 프롤로그가 나오는지 확인한다면, 오류를 줄일 수 있다. Volatility에서는 이렇게 의심되는 메모리 페이지를 코드 인젝션 탐지 플러그인(<a href="https://code.google.com/p/volatility/wiki/CommandReferenceMal22#malfind">malfind</a>)으로 추출할 수 있다. 단, 이 플러그인은 오탐이 존재할 수 있으므로 덤프한 영역의 악성 여부는 분석가의 판단이 요구된다.</p>
<p>&nbsp;</p>
<h3 id="코드-보호(암호화,-인코딩,-압축)"><a href="#코드-보호(암호화,-인코딩,-압축)" name="코드-보호(암호화,-인코딩,-압축)"></a>7. 코드 보호(암호화, 인코딩, 압축)</h3>
<p>사이버전으로 분류된 악성코드의 가장 눈에 띄는 특징은 상용 패커나 프로텍터를 사용하지 않는다는 점이다. 악성 샘플로 발견된 시점에는 언젠가는 분석이 가능하기도하고, 공격자 입장에서는 프로텍터로 인한 탐지의 위험도를 올릴 필요가 없다는 이유에서 사이버전 악성코드는 프로텍터를 잘 사용하지 않는다. 패커도 유사한 이유로 잘 사용되지 읺는데, 상용 패커는 아무리 잘 만들더라도 시그니처가 생성될 수 밖에 없기 때문이다. 그러더라도 악성코드가 자기자신을 대놓고 드러내는 것도 매우 위험한 행위가 될 수 있다.</p>
<h4>A. Regin의 코드 압축 기법</h4>
<p>Regin은 일반적인 프로그램도 사용하는 정상적인 기법으로 자기자신을 은폐한다. 크게 다음과 같다.</p>
<ul>
<li>암호화 : 알려진 암호화 기법인 RC5를 이용하여 암호화하였다.</li>
<li>인코딩 : Regin의 경우에는 커스텀 XOR 인코딩으로 의심을 받을 수 있는 문자열을 인코딩하였다. 정적 분석을 까다롭게 만드는 요소 중 하나이다.</li>
<li>압축 : Regin의 경우에는 <a href="http://os4depot.net/index.php?function=showfile&amp;file=development/library/misc/libucl.lha" target="_blank">UCL 라이브러리의 NRV2e 압축 알고리즘</a>을 사용하였다. 악의적이지 않은 압축 목적의 라이브러리를 사용함으로 탐지되지 않는다.</li>
</ul>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>사실 이러한 기법을 적용한 악성코드는 일반적인(General) 실행 파일과 큰 차이를 보이지 않기 때문에, 탐지 시그니처나 패턴을 찾기가 쉽지 않다. 보통은 임포트 함수를 추출하거나, 문자열 추출 과정에서 추출된 정보가 너무 적은 경우 문자열 암호화/인코딩을 의심하여 접근한다. 각 섹션의 엔트로피를 계산하여 일정 수치 이상이 나타나거나, 코드 섹션의 크기가 유난히 작은 경우엔 암호화/압축되었음을 의심해볼 수 있다.</p>
<p>&nbsp;</p>
<h3 id="다중-채널을-이용한-통신"><a href="#다중-채널을-이용한-통신" name="다중-채널을-이용한-통신"></a>7. 다중 채널을 이용한 통신</h3>
<p>악성코드에게 통신은 공격자와 연결하기 위한 가장 중요한 기능이다. 보통 악성코드는 단일 프로토콜을 이용하여 공격자와 통신하거나 악성코드간의 통신을 수행하는데, 이러다보니 분석가가 특정 프로토콜을 시그니처로 잡아 네트워크 보안장비에 추가하면 손쉽게(?) 네트워크 레벨의 통신을 차단할 수 있다. 이에 최근 악성코드는 알려진 프로토콜과 알려진 포트를 이용하여 마치 정상적인 것처럼 보이는 네트워크 통신을 사용하기 시작하였다. Regin은 이러한 악성코드보다 좀 더 진보된 다중 채널로 정상적인 것처럼 보이는 통신을 수행하였다.</p>
<h4 id="regin의-통신-방법"><a href="#regin의-통신-방법" name="regin의-통신-방법"></a>A. Regin의 통신 방법</h4>
<p>Regin은 탐지를 피하기 위해 중앙 서버를 통한 제어와 시스템간의 P2P 통신을 둘 다 지원하며, 여러 통신 프로토콜을 이용하였다. 이러한 통신은 분석가의 네트워크 분석을 더 까다롭게 만들었다. 통신 프로토콜도 일반적으로 많이 사용하는 HTTP, HTTPS, SMB, NamedPipe, ICMP를 이용하였는데, ICMP의 경우에는 핑 패킷의 뒷부분에 자신이 전송할 데이터를 추가하여 전송하는 은닉 채널(Covert Channel) 기법을 이용하였으며, HTTP나 HTTPS의 경우에는 잘 알려진 포트(80, 443)으로 특정 값(SMSWRAP 등)을 생성하여 데이터를 베이스64로 인코딩하여 전송하는 방법을 사용하였다.</p>
<p><img class="aligncenter size-full wp-image-1364" src="{{ site.baseurl }}/assets/Screen-Shot-2014-12-28-at-10.52.34-PM.png" alt="Screen Shot 2014-12-28 at 10.52.34 PM" width="908" height="575" /></p>
<p>또한, 위 그림과 같이 기관 단위 통신은 유관기관끼리 연결되도록 구성함으로 네트워크 보안 장비로부터 의심을 최소화하도록 노력한 흔적을 확인할 수 있다.</p>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>이건 사실 정해진 방법이 없다. 가장 좋은 방법은 악성코드 분석을 통해 사건 시간에 생성된 패킷만 덤프한 후, 정상 상태일 때의 패킷 덩어리와 덤프한 패킷 덩어리를 비교하여 비정상 상태의 패킷만 따로 추출 후 분석하는 것 정도를 생각해볼 수 있다. 하지만 이 방법은 기업 내에서 정상 패킷 데이터를 제대로 보관안한다면 적용할 수 없다. 사실 이전에 로깅한 패킷이 정상 패킷이라는 보장도 없다.</p>
<p>&nbsp;</p>
<h3 id="디지털-서명"><a href="#디지털-서명" name="디지털-서명"></a>8. 디지털 서명</h3>
<p>디지털 서명은 바이너리에 대한 신뢰성을 입증하는 매체로 PE 파일의 디지털 서명은 PKCS7 공개키 암호화 표준과  X.509 인증서를 기반으로 한다. 해당 서명은 exe, dll, sys 파일에 적용할 수 있으며, 최상위 서명자인 마이크로소프트가 서명한 기관만 유효한 디지털 서명을 할 수 있다. 디지털 서명은 마이크로소프트가 인증한 기업에서 배포한 바이너리임을 입증하는 하나의 수단이기 때문에 처음에는 유효한 디지털 서명을 가진 바이너리는 제외하고 분석하는 절차를 가지고 있었다. 하지만 Stuxnet나 Duqu와 같은 악성코드가 가짜 기업을 통해 유효한 디지털 서명을 하게되면서, 더이상 디지털 서명된 바이너리도 신뢰할 수 없게 되었다. MS가 비스타 이상의 윈도우 64비트 운영체제부터는 디지털 서명된 드라이버만 로드되도록 KMCS라는 기능을 추가하다보니, 이러한 초정밀 악성코드는 KMCS의 취약점을 찾거나, 어떻게든 유효한 디지털 서명으로 임무를 안전(?)하게 수행하려 할 것이다.</p>
<h4 id="regin의-디지털-서명"><a href="#regin의-디지털-서명" name="regin의-디지털-서명"></a>A. Regin의 디지털 서명</h4>
<p>Regin은 감염 운영체제가 64비트 환경일 경우, 로더(Loader)로 디지털 서명이 적용된 DLL을 사용한다. 하지만, 해당 DLL을 확인해보면, Issuer가 올바르지 않다는 메시지를 확인할 수 있다. 즉, 마이크로소프트에서 공식적으로 서명된 인증서가 아니라는 의미이다. 실제로 Issuer를 확인하면, 'Windows Root Authority'라는 정상적인 명칭을 가지고 있으나, 비정상적인 KeyID를 가지고 있었다. 이러한 부분으로 오히려 악성 파일로 의심을 살 수 있음에도 Regin은 왜 유효하지 않은 인증서로 위장하려고 했던 것인가?</p>
<p>카스퍼스키의 보고서에 따르면 Regin의 목적은 단지 자신이 따라하고자하는 바이너리와 동일한 것처럼 보이게 하기 위한 것이였다. Regin은 네트워크 통신을 수행하는 윈도의 정상적인 DLL 파일인 것처럼 보이도록 파일의 속성정보를 그대로 가져왔을 뿐만 아니라 디지털 서명처리도 해두어서, 디지털 서명의 존재여부만 확인한다면 외형적으론 문제없는 DLL인 것처럼 구성하였다. 이게 Regin이 유효하지 않은 서명을 이용해서라도 디지털 서명을 한 이유이다.</p>
<h4 id="탐지-기법"><a href="#탐지-기법" name="탐지-기법"></a>B. 대응 기법</h4>
<p>위에 Regin의 기법에 써있다시피, 각 PE 파일 내 디지털 서명의 유효성(validation)만 확인하여 validation하지 않은 파일을 추출한다면, 샘플 파일을 찾아낼 수 있다. 단, Regin은 로더(Loader)에만 이러한 행위를 수행하다보니 다음 스테이지가 이전 스테이지를 제거하는 특징으로 인해 삭제된 파일을 복구하여 디지털 서명을 검증해야하므로, 분석이 좀 더 까다롭게 된다.</p>
<p>&nbsp;</p>
<h3 id="악성코드-동적-분석-회피"><a href="#악성코드-동적-분석-회피" name="악성코드-동적-분석-회피"></a>9. 악성코드 동적 분석 회피</h3>
<p>안티 바이러스의 주 기능 중 하나인 시그니처 기반의 악성코드 탐지 기법은 시그니처만 우회하면 손쉽게 공격을 파악할 수 있다는 문제점을 가지고 있다. 이러한 문제를 해결하고자, 악성코드를 샌드박스 환경에서 동작시키고 그 행동을 모니터링하는 동적 분석 시스템이 등장하기 시작하였다. 이러한 동적 분석 기술은 악성코드의 패커 및 안티 가상화를 제외한 프로텍터 기능을 무력화시킬 수 있게 되었다. 현재의 동적 분석 기술은 자동화로 발전되었으며, 동적 분석의 여러 단점을 해소하기 위해 스코어링 기법과 안티-안티 가상머신 기술을 적용하고 있다. Regin은 이러한 기법이 적용된 솔루션을 우회하는 기술을 내장하고 있다.</p>
<h4 id="regin의-동적-분석-회피"><a href="#regin의-동적-분석-회피" name="regin의-동적-분석-회피"></a>A. Regin의 동적 분석 회피</h4>
<p>동적 분석기가 악성코드를 모니터링하려면, 실행 프로세스가 어떠한 함수를 호출하며, 호출한 함수의 리턴 값의 정보를 확인할 수 있어야 한다. 이를 위해 주요 커널 함수를 호출하는 시점을 미리 후킹해두고, 함수가 호출될 때 리턴 주소를 확인하여, 호출한 프로세스를 파악한다. 다음은 Regin의 우회 방법을 그림으로 표현한 것이다.</p>
<p><img class="aligncenter size-full wp-image-1342" src="{{ site.baseurl }}/assets/regin_001.png" alt="regin_001" width="775" height="364" /></p>
<p>Regin은 자동화된 동적 분석을 방해하기 위해 시스템 함수의 리턴 주소를 변경하는 방법을 사용하였다. 악성코드의 IAT(Import Address Table)을 후킹하여, Pre API 단계로 이동(JMP)하도록 수정한다. Pre API는 CALL로 인해 스택에 추가된 악성코드의 다음 명령어 주소를 신뢰성을 가진 안전한 모듈의 JMP 코드의 주소로 변경한다. Regin은 신뢰도를 가진 몇 개의 모듈(ntoskrnl.exe, disk.sys, HAL.dll, atapi.sys)에서 다음과 같은 방법으로 JMP 코드를 스캐닝한다.</p>
<pre><code>for offset in lenofbin:
    if bin[offset] == 0xFF:
        if bin[offset+1] == 0x26:    # jmp dword ptr [esi]
            return true    # find
        if bin[offset+1] == 0x27:    # jmp dword ptr [edi]
            return true    # find!
        if bin[offset+1] == 0x66:    # jmp dword ptr [esi+word]
            return true    # find!
        if bin[offset+1] == 0x67:    # jmp dword ptr [edi+word]
            return true    # find!
        if bin[offset+1] == 0xA6:    # jmp dword ptr [esi+dword]
            return true    # find!
        if bin[offset+1] == 0xA7:    # jmp dword ptr [edi+dword]
            return true    # find!
        if bin[offset+1] == 0xE6:    # jmp esi
            return true    # find!
        if bin[offset+1] == 0xE7:    # jmp edi
            return true    # find!

return false
</code></pre>
<p>JMP 코드로 리턴 주소를 변경하면, 시스템 함수를 실행한다. 시스템 함수가 코드를 실행하고 리턴 명령어가 실행되면, 앞서 설명한 JMP 코드의 주소로 복귀하게 되고, 이 JMP 코드는 바로 Post API로 제어를 이전한다. 이런식의 코드 수행으로 시스템 함수의 리턴 주소를 검증하더라도, 신뢰도를 가진 모듈이 호출자로 식별되어 동적 분석 시스템이 악성 행위로 판단하지 않는다.</p>
<h4 id="탐지-방법"><a href="#탐지-방법" name="탐지-방법"></a>B. 대응 방법</h4>
<p>현재 이 방법을 탐지하려면, 악성 실행 파일의 호출 흐름을 실행파일에서부터 추적해야한다. 실행 파일이 로드되는 시점에 해당 PID에서 호출하는 흐름을 추적하는 방법을 사용할 수 있다. 단, 사용자 레벨 후킹을 위해 IAT를 수정했다면, Regin이 덮어씌우지 않도록 해당 영역을 보호해야한다. 단, 이 방법은 시스템 성능이 매우 저하될 수 있기 때문에, 단일 샘플에 대한 분석을 수행할 경우 적용해볼 수 있다.(사실 이 방법도 명확한 해결 방법이라곤 볼 수 없다..)<br />
또 다른 방법으로는 주요 시스템 모듈의 호출 흐름을 데이터베이스화 해두고, 이와 다른 흐름이 발생할 경우를 추적해볼 수 있다. 단, 이 방법은 운영체제가 업데이트 될 때마다 데이터베이스를 갱신해두어야하며, 백신과 같이 보호 목적으로 흐름 제어를 변경하는 경우에 오탐이 발생하는 문제를 가지고 있다.</p>
<h2 id="결론"><a href="#결론" name="결론"></a>결론</h2>
<p>Regin의 은닉 기능은 일반적인 탐지 방법을 적용하기가 쉽지 않다. 공격자 입장에서는 다른 악성코드에서 몇 가지 요소만 수정하여 기법을 적용하면, 악성코드 탐지가 힘들도록 만들 수 있다. 기술적인 부분으로 탐지가 여의치 않다면, 정책적인 부분으로 사전에 차단할 수 있어야 한다. 기업 내 보안 인력에 대한 지속적인 보안 의식 고취와 물리적인 망 분리 뿐만이 아니라, 직원들의 예상치 못한 행동으로 발생하는 피해도 논리적인 망 분리도 완벽하게 이루어져야 한다. 또한 침해 사고 대응을 위한 분석 증거를 안전한 장소에 보관하여 역추적이 용이한 사후 분석 프로세스를 정립해야 할 것이다.</p>
<p>이렇게 새로운 기법이 적용된 악성코드는 지속적으로 등장할 것이며, 보안 의식이 낮은 기업에서는 몇겹의 보안 솔루션이 진을 치고 있더라도 무조건적인 피해를 당하게 될 것임을 알아두어야 할 것이다.</p>
<p>&nbsp;</p>
<p><em> 사족) 앞서 설명한 침해 대응 방법을 보면 명확한 방법을 제시한 것은 몇 개 없이 대부분 추상적인 내용으로 작성되어 있는 것을 볼 수 있을 것이다. 침해사고대응 업무를 하는 사람들은 항상 이렇게 많은 변수를 안고 일을 시작할 수 밖에 없게 된다. 이러한 변수를 최소화하는 방법 중 하나가 공격자의 관점에서 사건을 바라보는 것이다.</em></p>
<p><em>하지만 포렌식 업무를 하는 사람들 중엔 공격자가 사용하는 기술은 크게 관심없고, 알려진 포렌식 분석 기법만 알아도 충분하다고 생각하는 사람이 가끔있다. 공격자의 의도를 파악하지 못하면 분석 시간의 지연으로 이어질 수 있으며, 분석된 증거 조각간의 관계 파악도 힘들기 때문에 공격자의 숨겨진 의도를 파악하지 못할 수 있다. 손자병법에도 나와있는 지피지기 백전백승은 괜히 있는 말이 아니다. 그래서 필자는 포렌식 공부를 하는 학생들에게 포렌식 뿐만 아니라 해킹 기술에 대한 내용도 알아두길 권고하고 있다.</em></p>
