---
layout: post
title: 'DFIRCON 2014 : Memory Forensics (Part IV) – irykmmww.sys'
date: 2014-02-11 16:22:27.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- Memory Forensics
tags:
- malware analysis
- memory forensics
- rootkit
- windows
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  slide_template: default
  pyre_video: ''
  pyre_full_width: 'no'
  pyre_sidebar_position: default
  pyre_fimg_width: ''
  pyre_fimg_height: ''
  pyre_image_rollover_icons: linkzoom
  pyre_link_icon_url: ''
  pyre_related_posts: 'yes'
  pyre_slider_type: 'no'
  pyre_slider: '0'
  pyre_wooslider: '0'
  pyre_flexslider: '0'
  pyre_revslider: '0'
  pyre_elasticslider: '0'
  pyre_fallback: ''
  pyre_page_bg_layout: default
  pyre_page_bg: ''
  pyre_page_bg_color: ''
  pyre_page_bg_full: 'no'
  pyre_page_bg_repeat: repeat
  pyre_wide_page_bg: ''
  pyre_wide_page_bg_color: ''
  pyre_wide_page_bg_full: 'no'
  pyre_wide_page_bg_repeat: repeat
  pyre_header_bg: ''
  pyre_header_bg_color: ''
  pyre_header_bg_full: 'no'
  pyre_header_bg_repeat: repeat
  pyre_page_title: 'no'
  pyre_page_title_text: 'no'
  pyre_page_title_custom_text: ''
  pyre_page_title_custom_subheader: ''
  pyre_page_title_height: ''
  pyre_page_title_bar_bg: ''
  pyre_page_title_bar_bg_retina: ''
  pyre_page_title_bar_bg_full: default
  pyre_page_title_bar_bg_color: ''
  pyre_page_title_bg_parallax: default
  sbg_selected_sidebar: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_replacement: a:1:{i:0;s:1:"0";}
  avada_post_views_count: '2056'
  fusion_builder_content_backup: "Part IV에서는 루트킷인 irykmmww.sys에 대한 내용을 다룬다.\n<h2>1.
    기능 분류</h2>\n디바이스 드라이버인 irykmmww.sys는 DLL이나 D1L의 행위를 사용자에게 노출되지 않도록 은닉하는 임무를 담당한다.
    참고로 디바이스 드라이버를 메모리에서 덤프한 경우에는 Native API가 전부 순서(Ordinal number)만 남기 때문에 함수의 인자나
    구성에 기반하여 함수 명을 추정해야 한다. sys 하나만 분석하는 경우에는 함수를 정의할만한 단서를 찾기가 쉽지 않기 때문에 루트킷과 통신하는
    프로세스의 코드를 함께 분석하는 것이 좋다. 이 글에서 보이는 함수 명은 악성코드를 분석하여 직접 함수 명을 맞춰 준 것이기 때문에 실제 파일을
    분석할 때는 함수 명을 식별할 수 없을 것이다.\n\n메시지에 따라 다음 기능을 수행한다.\n<ul>\n\t<li>0x25610008 : SSDT
    후킹</li>\n\t<li>0x2561000C : SSDT 후킹 해제</li>\n\t<li>0x25610010 : D1L이 전달한 PID 저장</li>\n\t<li>0x25610014
    : D1L의 파일 명 저장</li>\n\t<li>0x25610018 : D1L이 전달한 IP주소 저장</li>\n\t<li>0x25610020
    : 특정 서비스(dmserver or rpcss)의 경로를 악성 DLL로 변경(추정)</li>\n\t<li>0x25610024 : 레지스트리
    키 롤백(추정)</li>\n</ul>\n여기에서는 SSDT 후킹과 특정 서비스의 경로를 악성 DLL로 변경하는 부분만 알아보도록 하겠다.\n<h2>2.
    SSDT 후킹</h2>\nDriveEntry에서 ntoskrnl에 SSDT의 Base Address를 저장하고 있는 KeServiceDescriptorTable
    값을 저장하고, 이 주소를 기준으로 원본 함수의 주소를 가져온다. 주소를 획득한 후에는 IRQL과 CR0 레지스터의 Write-Protection
    Flag를 Disable하고 SSDT의 주소를 루트킷 정의 함수로 변경한다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/ssdthook.png\"><img
    class=\"aligncenter size-full wp-image-821\" alt=\"ssdthook\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/ssdthook.png\"
    width=\"1186\" height=\"500\" /></a>\n\n루트킷 변경하는 함수는 \"NtOpenKey, NtQueryValueKey,
    NtEnumerateKey, NtEnumerateValueKey, DeviceIoControlFile, NtQuerySystemInformation,
    NtQueryDirectoryFile)이다. 각 후킹함수는 다음 역할을 수행한다.\n<ul>\n\t<li>NtOpenKey : 일단 정상적인
    NtOpenKey를 호출한다. 리턴받은 핸들을 토대로 다음 경로의 하위 키 중 \"irykmmww\" 또는 \"LEGACY_IRYKMMWW\"가
    있는지 확인하고 있다면, '레지스트리 경로'와 '키의 Index' 값을 저장한다.\n<ul>\n\t<li>REGISTRYMACHINESYSTEMCURRENTCONTROLSETSERVICES
    -&gt; irykmmww 확인</li>\n\t<li>REGISTRYMACHINESYSTEMCURRENTCONTROLSETENUMROOT -&gt;
    LEGACY_IRYKMMWW 확인</li>\n\t<li>REGISTRYMACHINESYSTEMCONTROLSET001SERVICES -&gt;
    irykmmww 확인</li>\n\t<li>REGISTRYMACHINESYSTEMCONTROLSET001ENUMROOT -&gt; LEGACY_IRYKMMWW
    확인</li>\n\t<li>REGISTRYMACHINESYSTEMCONTROLSET002SERVICES -&gt; irykmmww 확인</li>\n\t<li>REGISTRYMACHINESYSTEMCONTROLSET002ENUMROOT -&gt;
    LEGACY_IRYKMMWW 확인</li>\n</ul>\n</li>\n\t<li>NtQueryValueKey : 정상적인 NtQueryValueKey를
    호출하고 결과를 받는다. 만약,  호출하는 레지스트리 키가 dmserver 또는 rpcss 서비스의 파라미터이고, 그 중 ServiceDll
    값(value)의 데이터를 찾는다면, 결과 값을 \"%SystemRoot%System32dmserver.dll\" 또는 rpcss.dll로
    변경하여 리턴한다. 이로인해 분석가가 dmserver나 rpcss 서비스를 라이브 상태에서 분석하더라도 변경된 내용을 식별할 수 없다. 체크하는
    레지스트리 키는 dmserver 기준으로 다음과 같다.\n<ul>\n\t<li>REGISTRYMACHINESYSTEMCURRENTCONTROLSETSERVICESDMSERVERPARAMETERS</li>\n\t<li>REGISTRYMACHINESYSTEMCONTROLSET001SERVICESDMSERVERPARAMETERS</li>\n\t<li>REGISTRYMACHINESYSTEMCONTROLSET002SERVICESDMSERVERPARAMETERS</li>\n</ul>\n</li>\n\t<li>NtEnumerateKey
    : 호출 시, NtOpenKey에서  저장한 레지스트리 경로와 키의 인덱스 값을 토대로 해당 레지스트리 경로일 경우, irykmmww 서비스
    또는 드라이버의 레지스트리 키 인덱스 값을 0으로 변경하여 원본 NtEnumerateKey 함수를 호출한다. 이를 통해 루트킷은 자신이 설치됨으로
    인해 생성되는 레지스트리 값을 은닉할 수 있다.</li>\n\t<li>NtEnumerateValueKey : NtQueryValueKey와
    마찬가지로 원본 함수를 실행하고 dmserver나 rpcss 서비스의 ServiceDll 값을 바꿔치기하여 리턴한다.</li>\n\t<li>NtQueryDirectoryFile
    : 루트킷은 디렉터리 내의 파일 및 폴더 목록을 제공하는 API인 이 함수를 호출하고, 결과에서 irykmmww 문자열이 들어가는 모든 파일을
    제거한다. 만약에 D1L이 로드된 프로세스의 PID라면, 정상적인 결과를 제공한다. 이 방법으로 악성코드는 악성코드가 생성하는 모든 파일(모든
    파일엔 irykmmww가 들어가 있음)을 숨길 수 있다.</li>\n\t<li>NtQuerySystemInformation : 정상적인 NtQuerySystemInformation을
    호출한다. 호출 인자 중 SystemClass가 <a href=\"http://anncc.tistory.com/m/post/view/id/37\"
    target=\"_blank\">SystemProcessesAndThreadsInformation(5)</a>라면 결과 값을 추적하여, 결과
    값에 있는 프로세스 정보와 D1L이 로드된 프로세스의 PID를 비교한다. 만약 일치한다면 프로세스 링크 정보를 수정하고 해당 프로세스의 정보를
    제거한다. 이 방법으로 공격자는 프로세스 목록에서 자기자신을 숨길 수 있다.</li>\n\t<li>DeviceIoControlFile : 루트킷은
    DeviceIoControlFile을 호출한다. 호출한 IOCTL이 0x120003 (IOCTL_TCP_QUERY_INFORMATION_EX)라면,
    TCP 세션 구조체를 추적하여 D1L이 전송한 IP 주소(218.85.133.23)이 있을 경우 해당 정보를 제거한다. 이를 통해 악성코드는
    자신의 세션을 은닉할 수 있다.</li>\n</ul>\n<h2>3. 악성 DLL로 변경</h2>\n루트킷은 악성 D1L에게 받은 레지스트리키와
    악성코드 이름을 토대로 다음 경로에 있는 ServiceDll 값을 \"%SystemRoot%system32irykmmww.d1l\"로 변조한다.
    (XP 기준)\n<ul>\n\t<li>REGISTRYMACHINESYSTEMCURRENTCONTROLSETSERVICESDMSERVERPARAMETERS</li>\n\t<li>REGISTRYMACHINESYSTEMCONTROLSET001SERVICESDMSERVERPARAMETERS</li>\n\t<li>REGISTRYMACHINESYSTEMCONTROLSET002SERVICESDMSERVERPARAMETERS</li>\n</ul>\n변조된
    내용은 SSDT 후킹을 통해 라이브 상태에서는 변조되지 않은 것처럼 보이게 만든다.\n<h2>4. 추정되는 감염 시나리오</h2>\nPart
    I부터 Part IV까지의 시나리오를 토대로 가능한 시나리오를 만들어 보았다. 분석 결과를 보면 알겠지만, 메모리 분석만으로는 악성코드의 감염
    시나리오를 완벽하게 짤순 없었다. 이에 크게 2가지 시나리오를 생각해볼 수 있다.\n<h3>첫 번째 시나리오</h3>\n<ol>\n\t<li>공격자는
    explorer.exe에 DLL 인젝션을 수행하고, 루트킷을 서비스 형태로 설치한다.</li>\n\t<li>시스템을 재부팅하면, 루트킷이 변조한
    ServiceDll 값을 토대로 irykmmww.d1l을 svchost.exe가 로드한다.</li>\n\t<li>svchost.exe에 인젝션된
    d1l은 iexplore.exe를 실행하고 내부에 d1l을 인젝션한다.</li>\n\t<li>iexplore.exe는 공격자 서버에 접속한다.</li>\n\t<li>iexplore.exe가
    서비스를 다시 설치하고 실행한다. 그리고 SSDT 후킹 명령을 서비스에게 전송한다.</li>\n\t<li>DLL에 있는 익스포트 함수를 이용하여
    전역 키보드 후킹을 수행한다.</li>\n</ol>\n위 시나리오는 Rob Lee가 설명한 풀이 방법에 가장 근접한 시나리오이다. 단 이 시나리오의
    문제는 1번을 증명하기 쉽지 않다는 점이다. 일단 irykmmww.d1l에 루트킷을 서비스 형태로 설치하고 실행하는 모든 과정이 담겨있기 때문에
    저런 과정을 거칠 필요가 없다. 그리고 프로세스가 생성된 시간을 보면, iexplore.exe, cmd.exe, MIRAgent.exe만 한달
    뒤에 실행된 것을 볼 수 있는데, 이 점만 보더라도 시스템이 재부팅된 것으로 보긴 힘들다.\n<h3>두 번째 시나리오</h3>\n시간 정보를
    보면 일단 가상머신을 2009년 4월로 셋팅하고 아무 작업도 하지 않다가, 5월달에 문제를 내기위해 악성코드를 감염시키기 시작한 것으로 보인다.
    시스템이 재부팅되지 않았다는 가정하에 시나리오를 짜면 다음과 같다.\n<ol>\n\t<li>악성코드는 explorer.exe 및 그 하위 프로세스와
    svchost.exe에 각각 DLL, D1L을 인젝션한다.</li>\n\t<li>explorer.exe 및 하위프로세스는 메신저 기록 등 사용자
    주요 정보를 탈취한다.</li>\n\t<li>svchost.exe에 인젝션된 D1L은 iexplore.exe를 실행하고 D1L 을 인젝션한다.</li>\n\t<li><span
    style=\"line-height: 1.5em;\">iexplore.exe는 공격자 서버에 접속한다.</span></li>\n\t<li>iexplore.exe는
    루트킷 서비스를 생성(irykmmww)하고, 루트킷 서비스를 실행하며 SSDT 후킹 명령을 내린다.</li>\n\t<li>DLL에 있는 익스포트
    함수를 이용하여 전역 키보드 후킹을 수행한다.</li>\n</ol>\n이 시나리오의 경우에는 1번 항목을 검증할 순 없지만, 나머지가 악성코드의
    역할에 맞게 순차적으로 수행된다. Rob Lee가 도출한 답과는 다르지만, 가장 적합한 시나리오라 판단된다.\n<h2>5. 결론</h2>\n본
    메모리 분석 문제는 SANS에서 냈다고하기에는 너무 퀄리티가 떨어지는 문제였다. 일단 문제의 전제 자체가 매우 부정확하며, 분석 보고서를 보면
    메모리 분석만으로 결론을 도출하고 있다. 디지털 포렌식이라는 분야는 추정을 최소화해야하는데, 본 문제는 많은 추정을 요구하고 있다. 하물며
    분석 결과만으로도 말이다.. ;-? 내년에 내는 SANS 문제에는 더 나은 결과를 보여주길 바란다."
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>Part IV에서는 루트킷인 irykmmww.sys에 대한 내용을 다룬다.</p>
<h2>1. 기능 분류</h2>
<p>디바이스 드라이버인 irykmmww.sys는 DLL이나 D1L의 행위를 사용자에게 노출되지 않도록 은닉하는 임무를 담당한다. 참고로 디바이스 드라이버를 메모리에서 덤프한 경우에는 Native API가 전부 순서(Ordinal number)만 남기 때문에 함수의 인자나 구성에 기반하여 함수 명을 추정해야 한다. sys 하나만 분석하는 경우에는 함수를 정의할만한 단서를 찾기가 쉽지 않기 때문에 루트킷과 통신하는 프로세스의 코드를 함께 분석하는 것이 좋다. 이 글에서 보이는 함수 명은 악성코드를 분석하여 직접 함수 명을 맞춰 준 것이기 때문에 실제 파일을 분석할 때는 함수 명을 식별할 수 없을 것이다.</p>
<p>메시지에 따라 다음 기능을 수행한다.</p>
<ul>
<li>0x25610008 : SSDT 후킹</li>
<li>0x2561000C : SSDT 후킹 해제</li>
<li>0x25610010 : D1L이 전달한 PID 저장</li>
<li>0x25610014 : D1L의 파일 명 저장</li>
<li>0x25610018 : D1L이 전달한 IP주소 저장</li>
<li>0x25610020 : 특정 서비스(dmserver or rpcss)의 경로를 악성 DLL로 변경(추정)</li>
<li>0x25610024 : 레지스트리 키 롤백(추정)</li>
</ul>
<p>여기에서는 SSDT 후킹과 특정 서비스의 경로를 악성 DLL로 변경하는 부분만 알아보도록 하겠다.</p>
<h2>2. SSDT 후킹</h2>
<p>DriveEntry에서 ntoskrnl에 SSDT의 Base Address를 저장하고 있는 KeServiceDescriptorTable 값을 저장하고, 이 주소를 기준으로 원본 함수의 주소를 가져온다. 주소를 획득한 후에는 IRQL과 CR0 레지스터의 Write-Protection Flag를 Disable하고 SSDT의 주소를 루트킷 정의 함수로 변경한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/ssdthook.png"><img class="aligncenter size-full wp-image-821" alt="ssdthook" src="{{ site.baseurl }}/assets/ssdthook.png" width="1186" height="500" /></a></p>
<p>루트킷 변경하는 함수는 "NtOpenKey, NtQueryValueKey, NtEnumerateKey, NtEnumerateValueKey, DeviceIoControlFile, NtQuerySystemInformation, NtQueryDirectoryFile)이다. 각 후킹함수는 다음 역할을 수행한다.</p>
<ul>
<li>NtOpenKey : 일단 정상적인 NtOpenKey를 호출한다. 리턴받은 핸들을 토대로 다음 경로의 하위 키 중 "irykmmww" 또는 "LEGACY_IRYKMMWW"가 있는지 확인하고 있다면, '레지스트리 경로'와 '키의 Index' 값을 저장한다.
<ul>
<li>REGISTRYMACHINESYSTEMCURRENTCONTROLSETSERVICES -&gt; irykmmww 확인</li>
<li>REGISTRYMACHINESYSTEMCURRENTCONTROLSETENUMROOT -&gt; LEGACY_IRYKMMWW 확인</li>
<li>REGISTRYMACHINESYSTEMCONTROLSET001SERVICES -&gt; irykmmww 확인</li>
<li>REGISTRYMACHINESYSTEMCONTROLSET001ENUMROOT -&gt; LEGACY_IRYKMMWW 확인</li>
<li>REGISTRYMACHINESYSTEMCONTROLSET002SERVICES -&gt; irykmmww 확인</li>
<li>REGISTRYMACHINESYSTEMCONTROLSET002ENUMROOT -&gt; LEGACY_IRYKMMWW 확인</li>
</ul>
</li>
<li>NtQueryValueKey : 정상적인 NtQueryValueKey를 호출하고 결과를 받는다. 만약,  호출하는 레지스트리 키가 dmserver 또는 rpcss 서비스의 파라미터이고, 그 중 ServiceDll 값(value)의 데이터를 찾는다면, 결과 값을 "%SystemRoot%System32dmserver.dll" 또는 rpcss.dll로 변경하여 리턴한다. 이로인해 분석가가 dmserver나 rpcss 서비스를 라이브 상태에서 분석하더라도 변경된 내용을 식별할 수 없다. 체크하는 레지스트리 키는 dmserver 기준으로 다음과 같다.
<ul>
<li>REGISTRYMACHINESYSTEMCURRENTCONTROLSETSERVICESDMSERVERPARAMETERS</li>
<li>REGISTRYMACHINESYSTEMCONTROLSET001SERVICESDMSERVERPARAMETERS</li>
<li>REGISTRYMACHINESYSTEMCONTROLSET002SERVICESDMSERVERPARAMETERS</li>
</ul>
</li>
<li>NtEnumerateKey : 호출 시, NtOpenKey에서  저장한 레지스트리 경로와 키의 인덱스 값을 토대로 해당 레지스트리 경로일 경우, irykmmww 서비스 또는 드라이버의 레지스트리 키 인덱스 값을 0으로 변경하여 원본 NtEnumerateKey 함수를 호출한다. 이를 통해 루트킷은 자신이 설치됨으로 인해 생성되는 레지스트리 값을 은닉할 수 있다.</li>
<li>NtEnumerateValueKey : NtQueryValueKey와 마찬가지로 원본 함수를 실행하고 dmserver나 rpcss 서비스의 ServiceDll 값을 바꿔치기하여 리턴한다.</li>
<li>NtQueryDirectoryFile : 루트킷은 디렉터리 내의 파일 및 폴더 목록을 제공하는 API인 이 함수를 호출하고, 결과에서 irykmmww 문자열이 들어가는 모든 파일을 제거한다. 만약에 D1L이 로드된 프로세스의 PID라면, 정상적인 결과를 제공한다. 이 방법으로 악성코드는 악성코드가 생성하는 모든 파일(모든 파일엔 irykmmww가 들어가 있음)을 숨길 수 있다.</li>
<li>NtQuerySystemInformation : 정상적인 NtQuerySystemInformation을 호출한다. 호출 인자 중 SystemClass가 <a href="http://anncc.tistory.com/m/post/view/id/37" target="_blank">SystemProcessesAndThreadsInformation(5)</a>라면 결과 값을 추적하여, 결과 값에 있는 프로세스 정보와 D1L이 로드된 프로세스의 PID를 비교한다. 만약 일치한다면 프로세스 링크 정보를 수정하고 해당 프로세스의 정보를 제거한다. 이 방법으로 공격자는 프로세스 목록에서 자기자신을 숨길 수 있다.</li>
<li>DeviceIoControlFile : 루트킷은 DeviceIoControlFile을 호출한다. 호출한 IOCTL이 0x120003 (IOCTL_TCP_QUERY_INFORMATION_EX)라면, TCP 세션 구조체를 추적하여 D1L이 전송한 IP 주소(218.85.133.23)이 있을 경우 해당 정보를 제거한다. 이를 통해 악성코드는 자신의 세션을 은닉할 수 있다.</li>
</ul>
<h2>3. 악성 DLL로 변경</h2>
<p>루트킷은 악성 D1L에게 받은 레지스트리키와 악성코드 이름을 토대로 다음 경로에 있는 ServiceDll 값을 "%SystemRoot%system32irykmmww.d1l"로 변조한다. (XP 기준)</p>
<ul>
<li>REGISTRYMACHINESYSTEMCURRENTCONTROLSETSERVICESDMSERVERPARAMETERS</li>
<li>REGISTRYMACHINESYSTEMCONTROLSET001SERVICESDMSERVERPARAMETERS</li>
<li>REGISTRYMACHINESYSTEMCONTROLSET002SERVICESDMSERVERPARAMETERS</li>
</ul>
<p>변조된 내용은 SSDT 후킹을 통해 라이브 상태에서는 변조되지 않은 것처럼 보이게 만든다.</p>
<h2>4. 추정되는 감염 시나리오</h2>
<p>Part I부터 Part IV까지의 시나리오를 토대로 가능한 시나리오를 만들어 보았다. 분석 결과를 보면 알겠지만, 메모리 분석만으로는 악성코드의 감염 시나리오를 완벽하게 짤순 없었다. 이에 크게 2가지 시나리오를 생각해볼 수 있다.</p>
<h3>첫 번째 시나리오</h3>
<ol>
<li>공격자는 explorer.exe에 DLL 인젝션을 수행하고, 루트킷을 서비스 형태로 설치한다.</li>
<li>시스템을 재부팅하면, 루트킷이 변조한 ServiceDll 값을 토대로 irykmmww.d1l을 svchost.exe가 로드한다.</li>
<li>svchost.exe에 인젝션된 d1l은 iexplore.exe를 실행하고 내부에 d1l을 인젝션한다.</li>
<li>iexplore.exe는 공격자 서버에 접속한다.</li>
<li>iexplore.exe가 서비스를 다시 설치하고 실행한다. 그리고 SSDT 후킹 명령을 서비스에게 전송한다.</li>
<li>DLL에 있는 익스포트 함수를 이용하여 전역 키보드 후킹을 수행한다.</li>
</ol>
<p>위 시나리오는 Rob Lee가 설명한 풀이 방법에 가장 근접한 시나리오이다. 단 이 시나리오의 문제는 1번을 증명하기 쉽지 않다는 점이다. 일단 irykmmww.d1l에 루트킷을 서비스 형태로 설치하고 실행하는 모든 과정이 담겨있기 때문에 저런 과정을 거칠 필요가 없다. 그리고 프로세스가 생성된 시간을 보면, iexplore.exe, cmd.exe, MIRAgent.exe만 한달 뒤에 실행된 것을 볼 수 있는데, 이 점만 보더라도 시스템이 재부팅된 것으로 보긴 힘들다.</p>
<h3>두 번째 시나리오</h3>
<p>시간 정보를 보면 일단 가상머신을 2009년 4월로 셋팅하고 아무 작업도 하지 않다가, 5월달에 문제를 내기위해 악성코드를 감염시키기 시작한 것으로 보인다. 시스템이 재부팅되지 않았다는 가정하에 시나리오를 짜면 다음과 같다.</p>
<ol>
<li>악성코드는 explorer.exe 및 그 하위 프로세스와 svchost.exe에 각각 DLL, D1L을 인젝션한다.</li>
<li>explorer.exe 및 하위프로세스는 메신저 기록 등 사용자 주요 정보를 탈취한다.</li>
<li>svchost.exe에 인젝션된 D1L은 iexplore.exe를 실행하고 D1L 을 인젝션한다.</li>
<li><span style="line-height: 1.5em;">iexplore.exe는 공격자 서버에 접속한다.</span></li>
<li>iexplore.exe는 루트킷 서비스를 생성(irykmmww)하고, 루트킷 서비스를 실행하며 SSDT 후킹 명령을 내린다.</li>
<li>DLL에 있는 익스포트 함수를 이용하여 전역 키보드 후킹을 수행한다.</li>
</ol>
<p>이 시나리오의 경우에는 1번 항목을 검증할 순 없지만, 나머지가 악성코드의 역할에 맞게 순차적으로 수행된다. Rob Lee가 도출한 답과는 다르지만, 가장 적합한 시나리오라 판단된다.</p>
<h2>5. 결론</h2>
<p>본 메모리 분석 문제는 SANS에서 냈다고하기에는 너무 퀄리티가 떨어지는 문제였다. 일단 문제의 전제 자체가 매우 부정확하며, 분석 보고서를 보면 메모리 분석만으로 결론을 도출하고 있다. 디지털 포렌식이라는 분야는 추정을 최소화해야하는데, 본 문제는 많은 추정을 요구하고 있다. 하물며 분석 결과만으로도 말이다.. ;-? 내년에 내는 SANS 문제에는 더 나은 결과를 보여주길 바란다.</p>
