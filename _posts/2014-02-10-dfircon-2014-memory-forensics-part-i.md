---
layout: post
title: 'DFIRCON 2014 : Memory Forensics (Part I)'
date: 2014-02-10 19:27:16.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- Memory Forensics
tags:
- malware
- memory forensics
- rootkit
- windows
meta:
  _edit_last: '1'
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
  avada_post_views_count: '3046'
  _wpas_done_all: '1'
  sbg_selected_sidebar: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_replacement: a:1:{i:0;s:1:"0";}
  fusion_builder_content_backup: "본 글은 2013년 12월부터 2014년 1월까지 진행한 DFIRCON에 대한 글이다.\n\nSANS는
    매 년말에 이벤트성 행사를 하는데, 요번에는 SANS의 Simulcast(온라인 강의) 티켓을 한장 걸고 행사를 진행했다. 사실 온라인 대회에
    티켓 한 장은 인간적으로 너무 적은 보상이긴 했지만 나름 재미있을 것 같아 참여하였다. 결과적으로는 Rob Lee가 말한 결과와 다른 분석
    결과가 나왔지만, 답이 나온 아직까지도 왜 그게 정확한 답인지 잘 모르겠다. 문제를 낸 사람은 악성코드를 직접 돌렸으니 알 수 있겠지만, 메모리
    분석 및 덤프한 바이너리 분석까지 해본 나로써는 해당 답을 도출하기에는 여러가지 제약사항이 있었다. 이에 본 메모리 이미지 분석을 통해 내
    자신의 의견을 게재하고 다양한 관점의 이야기를 해보고자 한다.\n\n<a href=\"https://www.surveymonkey.com/s/JQ9QFHP\"
    target=\"_blank\">DFIRCON APT Memory Analysis Challenge</a>는 메모리 이미지를 분석하여 총 5개의
    문제에 대한 답을 도출하는 것으로 문제에 대한 특별한 룰은 존재하지 않는다. 상황도 없다. 그냥 분석해서 답을 채우기만 하면 되는 문제이다.
    포렌식 문제임에도 아무런 상황을 제공하지 않은건 나름 포렌식 교육의 대표 기관 중 하나인 SANS에서 낸 문제치고는 아쉬움이 많았다. 아무래도
    상황이 없이 디지털 증거 중 일부 데이터만으로 분석을 할 경우, 포렌식에서 가장 중요한 사건 '추리'를 전혀할 수 없으며 이 부분에 대해 추정을
    해야하는 문제가 생긴다. 다음 문제에는 좀 더 포렌식 다운 문제를 제공하였으면 한다. 이러한 상황을 고려하여 본 연재에서는 일단 메모리 이미지
    분석만으로 유추해볼 수 있는 여러 시나리오를 작성해보도록 하겠다.\n<h2>1. 정보 수집(메모리 분석)</h2>\n제공 받은 메모리 이미지는
    'APT.img'로 MD5는 14AC99E2E6980D6A962F0AC2A1CB65C8이다. 메모리 이미지를 분석하기 위해 volatility
    framework를 사용하였으며, imageinfo 결과 Windows XP SP3 x86 으로 확인되었다. 일단 프로세스 목록을 확인하기
    위해 pstree 플러그인을 수행하였다.\n<pre class=\"lang:sh decode:true\" title=\"pstree on APT.img\">$
    python vol.py -f APT.img --profile=WinXPSP3x86 pstree\nVolatility Foundation Volatility
    Framework 2.3.1\nName                                                  Pid   PPid
    \  Thds   Hnds Time\n-------------------------------------------------- ------
    ------ ------ ------ ----\n 0x823c8830:System                                      4
    \     0     55    254 1970-01-01 00:00:00 UTC+0000\n. 0x8230aad8:smss.exe                                 564
    \     4      3     19 2009-04-16 16:10:01 UTC+0000\n.. 0x81f63020:winlogon.exe
    \                           660    564     16    502 2009-04-16 16:10:06 UTC+0000\n...
    0x81f22020:services.exe                           704    660     15    254 2009-04-16
    16:10:06 UTC+0000\n.... 0x81f739b0:svchost.exe                          1088    704
    \    70   1445 2009-04-16 16:10:07 UTC+0000\n..... 0x81f96220:wscntfy.exe                         1260
    \  1088      1     39 2009-04-16 16:10:22 UTC+0000\n.... 0x81da4590:svchost.exe
    \                          968    704     10    241 2009-04-16 16:10:07 UTC+0000\n....
    0x81dc2570:VMwareService.e                      1032    704      3    175 2009-04-16
    16:10:16 UTC+0000\n.... 0x8231eda0:msiexec.exe                          1464    704
    \     6    294 2009-04-16 16:11:02 UTC+0000\n.... 0x81e54da0:svchost.exe                           884
    \   704     17    208 2009-04-16 16:10:07 UTC+0000\n..... 0x81dbdda0:iexplore.exe
    \                        796    884      8    152 2009-05-05 19:28:28 UTC+0000\n....
    0x81e91da0:svchost.exe                          1212    704     14    208 2009-04-16
    16:10:09 UTC+0000\n.... 0x81d33628:alg.exe                               464    704
    \     6    105 2009-04-16 16:10:21 UTC+0000\n.... 0x8219b630:spoolsv.exe                          1512
    \   704     10    129 2009-04-16 16:10:10 UTC+0000\n.... 0x822cb458:vmacthlp.exe
    \                         872    704      1     25 2009-04-16 16:10:07 UTC+0000\n....
    0x8232c020:svchost.exe                          1140    704      5     60 2009-04-16
    16:10:08 UTC+0000\n... 0x82164da0:lsass.exe                              716    660
    \    21    342 2009-04-16 16:10:06 UTC+0000\n.. 0x822ca2c0:csrss.exe                               636
    \   564     10    356 2009-04-16 16:10:06 UTC+0000\n 0x81da71a8:explorer.exe                             1672
    \  1624     15    586 2009-04-16 16:10:10 UTC+0000\n. 0x81f1c7e8:VMwareTray.exe
    \                         1984   1672      1     37 2009-04-16 16:10:11 UTC+0000\n.
    0x81e4d648:cmd.exe                                  840   1672      1     33 2009-05-05
    15:56:24 UTC+0000\n.. 0x82161558:MIRAgent.exe                            456    840
    \     1     77 2009-05-05 19:28:40 UTC+0000\n. 0x81dc1a78:VMwareUser.exe                          2004
    \  1672      8    228 2009-04-16 16:10:11 UTC+0000\n. 0x81f1a650:ctfmon.exe                              2020
    \  1672      1     71 2009-04-16 16:10:11 UTC+0000</pre>\n여기서 중요하게 봐야하는 점은 '프로세스
    간의 부모-자식 관계', '프로세스 생성 시간', '프로세스 명'을 들 수 있다. 일단 884번 프로세스인 svchost.exe 아래에 796번
    프로세스인 iexplore.exe가 있는 것을 볼 수 있다. iexplore.exe는 인터넷 익스플로러(Internet Explorer)로
    보통 explorer.exe의 자식 프로세스여야 맞는데 마치 서비스 형태로 동작한 것처럼 보인다. iexplore.exe의 사용자 권한을 보면
    문제가 있음이 좀 더 명확해진다.\n<pre class=\"lang:sh decode:true\" title=\"getsids on internet
    explorer\">$ python vol.py -f APT.img --profile=WinXPSP3x86 getsids -p 796\niexplore.exe
    (796): S-1-5-18 (Local System)\niexplore.exe (796): S-1-5-32-544 (Administrators)\niexplore.exe
    (796): S-1-1-0 (Everyone)\niexplore.exe (796): S-1-5-11 (Authenticated Users)</pre>\n분석
    결과를 보면 로컬 시스템 권한을 가지고 있다. 프로세스 분석에서 볼 수 있는 또하나 재미있는 사실은 시간 정보이다. 분석 결과를 보면, MIRAgent.exe,
    cmd.exe, iexplore.exe는 2009년 5월 5일에 동작했으며, 나머지 프로세스는 2009년 4월 16일에 실행되었다. 구동 중인
    환경이 가상 머신(VMware)인 것을 고려해볼 때, 문제 제작자가 2009년 4월 16일로 설정된 가상머신을 구동을 시키고 사용하다가 출제를
    위해 5월 5일에 관련된 프로세스를 실행했거나, 고의적으로 시간 값을 돌렸을 가능성이 있다. MIRAgent.exe는 Mandiant의 Incident
    Response Agent로 악성코드와는 무관하다.\n\n혹시 자기자신을 숨기는 프로세스가 있을 수도 있기 떄문에 psxview로 분석을 수행하였으나
    은닉된 프로세스는 발견되지 않았다.\n<pre class=\"lang:sh decode:true\" title=\"psxview\">$ python
    vol.py -f APT.img --profile=WinXPSP3x86 psxview\nVolatility Foundation Volatility
    Framework 2.3.1\nOffset(P)  Name                    PID pslist psscan thrdproc
    pspcid csrss session deskthrd\n---------- -------------------- ------ ------ ------
    -------- ------ ----- ------- --------\n0x02163020 winlogon.exe            660
    True   True   True     True   True  True   True\n0x02122020 services.exe            704
    True   True   True     True   True  True   True\n0x0211a650 ctfmon.exe             2020
    True   True   True     True   True  True   True\n0x01fa71a8 explorer.exe           1672
    True   True   True     True   True  True   True\n0x0252c020 svchost.exe            1140
    True   True   True     True   True  True   True\n0x0204d648 cmd.exe                 840
    True   True   True     True   True  True   True\n0x01fc1a78 VMwareUser.exe         2004
    True   True   True     True   True  True   True\n0x02054da0 svchost.exe             884
    True   True   True     True   True  True   True\n0x02196220 wscntfy.exe            1260
    True   True   True     True   True  True   True\n0x021739b0 svchost.exe            1088
    True   True   True     True   True  True   True\n0x01fa4590 svchost.exe             968
    True   True   True     True   True  True   True\n0x02361558 MIRAgent.exe            456
    True   True   True     True   True  True   True\n0x02364da0 lsass.exe               716
    True   True   True     True   True  True   False\n0x0211c7e8 VMwareTray.exe         1984
    True   True   True     True   True  True   True\n0x02091da0 svchost.exe            1212
    True   True   True     True   True  True   True\n0x01fbdda0 iexplore.exe            796
    True   True   True     True   True  True   True\n0x024cb458 vmacthlp.exe            872
    True   True   True     True   True  True   True\n0x0239b630 spoolsv.exe            1512
    True   True   True     True   True  True   True\n0x0251eda0 msiexec.exe            1464
    True   True   True     True   True  True   True\n0x01f33628 alg.exe                 464
    True   True   True     True   True  True   True\n0x01fc2570 VMwareService.e        1032
    True   True   True     True   True  True   True\n0x0250aad8 smss.exe                564
    True   True   True     True   False False   False\n0x025c8830 System                    4
    True   True   True     True   False False   False\n0x024ca2c0 csrss.exe               636
    True   True   True     True   False True   True\n0x03178220 wscntfy.exe            1260
    False  True   False    False  False False   False\n0x0c605020 svchost.exe            1140
    False  True   False    False  False False   False\n0x0ad69da0 iexplore.exe            796
    False  True   False    False  False False   False\n0x0edd0628 alg.exe                 464
    False  True   False    False  False False   False\n0x032b3da0 svchost.exe             884
    False  True   False    False  False False   False\n0x0eed3628 alg.exe                 464
    False  True   False    False  False False   False\n0x10b54628 alg.exe                 464
    False  True   False    False  False False   False\n0x15934830 System                    4
    False  True   False    False  False False   False\n0x1b217da0 iexplore.exe            796
    False  True   False    False  False False   False\n0x04097020 svchost.exe            1140
    False  True   False    False  False False   False\n0x035c1590 svchost.exe             968
    False  True   False    False  False False   False\n0x07b1ada0 iexplore.exe            796
    False  True   False    False  False False   False\n0x0edd59b0 svchost.exe            1088
    False  True   False    False  False False   False\n0x12f3dda0 svchost.exe             884
    False  True   False    False  False False   False</pre>\n네트워크 연결 정보는 다음과 같다.\n<pre
    class=\"lang:sh decode:true\" title=\"network connection\">$ python vol.py -f
    APT.img --profile=WinXPSP3x86 connections\nVolatility Foundation Volatility Framework
    2.3.1\nOffset(V)  Local Address             Remote Address            Pid\n----------
    ------------------------- ------------------------- ---\n0x81e611f8 192.168.157.10:1053
    \      218.85.133.23:89          796\n$\n$ python vol.py -f APT.img --profile=WinXPSP3x86
    connscan\nVolatility Foundation Volatility Framework 2.3.1\nOffset(P)  Local Address
    \            Remote Address            Pid\n---------- -------------------------
    ------------------------- ---\n0x0205ece0 192.168.157.10:1050       222.128.1.2:443
    \          1672\n0x020611f8 192.168.157.10:1053       218.85.133.23:89          796\n0x032c01f8
    192.168.157.10:1053       218.85.133.23:89          796\n0x0337dce0 192.168.157.10:1050
    \      222.128.1.2:443           1672\n0x08a4ace0 192.168.157.10:1050       222.128.1.2:443
    \          1672\n0x18200ce0 192.168.157.10:1050       222.128.1.2:443           1672</pre>\nexplorer.exe(1672)는
    HTTPS 포트인 443으로 통신하고 있었으며, iexplore.exe(796)는 89번 포트로 통신을 하고 있었다. 두 IP 다 China
    Telecom에서 관리하고 있으며, 중국 내의 서버로 확인할 수 있다.\n<blockquote>실제 바이너리 분석을 해보면 222.128.1.2에
    연결한 흔적은 찾을 수 없었다. 또한 explorer.exe에 인젝션된 DLL은 별도의 소켓 통신을 하진 않는 것으로 보인다.</blockquote>\n서비스를
    확인해보니, 252번째 순서의 서비스(irykmmww)가 매우 의심스러운 서비스 명을 하고 있었다. 서비스 명을 키워드로 드라이버 목록인 driverscan으로
    확인하면 정상적으로 메모리에 로드되어 있는 것을 확인할 수 있다.\n<pre class=\"lang:sh decode:true\" title=\"svcscan
    and driverscan\">$ python vol.py -f APT.img --profile=WinXPSP3x86 svcscan\n.....\nOffset:
    0x38ab98\nOrder: 252\nProcess ID: -\nService Name: irykmmww\nDisplay Name: irykmmww\nService
    Type: SERVICE_KERNEL_DRIVER\nService State: SERVICE_RUNNING\nBinary Path: Driverirykmmww\n\n$
    python vol.py -f APT.img --profile=WinXPSP3x86 driverscan | findstr irykmmww\nVolatility
    Foundation Volatility Framework 2.3.1\n0x024c33b8    3    0 0xf836f000     0x3900
    irykmmww             irykmmww     Driverirykmmww</pre>\n서비스 등록은 윈도우에서 커널 드라이버를
    로드하는 방법 중 가장 안정적인 방법이다. 드라이버를 로드하기 위한 다양한 비정상적인 방법이 있지만 보통 시스템의 불안정성을 최소화하면서 손쉽게
    로드/언로드가 가능한 이점이 있다보니 루트킷 설치에도 서비스 등록이 많이 사용된다. volatility에서는 루트킷이 후킹한 데이터를 탐지하는
    여러가지 방법이 있으며, 이 중 ssdt(System Service Descriptor Table)에서 루트킷이 후킹한 함수를 확인할 수 있다.\n<pre
    class=\"lang:sh decode:true\" title=\"SSDT\">$ python vol.py -f APT.img --profile=WinXPSP3x86
    ssdt | grep irykmmww\nVolatility Foundation Volatility Framework 2.3.1\n  Entry
    0x0042: 0xf836fe9c (NtDeviceIoControlFile) owned by irykmmww.sys\n  Entry 0x0047:
    0xf83706dc (NtEnumerateKey) owned by irykmmww.sys\n  Entry 0x0049: 0xf837075e
    (NtEnumerateValueKey) owned by irykmmww.sys\n  Entry 0x0077: 0xf837028f (NtOpenKey)
    owned by irykmmww.sys\n  Entry 0x0091: 0xf8370a8c (NtQueryDirectoryFile) owned
    by irykmmww.sys\n  Entry 0x00ad: 0xf836fe3e (NtQuerySystemInformation) owned by
    irykmmww.sys\n  Entry 0x00b1: 0xf837091a (NtQueryValueKey) owned by irykmmww.sys</pre>\n장치와
    애플리케이션간 통신에 사용되는 DeviceIoControlFile과 레지스트리 관리 및 파일 관리를 위한 Native 함수를 후킹한 것을 확인하였다.
    그리고 후킹에 사용된 드라이버가 'irykmmww.sys'인 것도 확인할 수 있었다. 루트킷은 보통 드라이버 단독적으로 활동하지 않으며 프로세스나
    DLL과 같은 사용자 영역(User Space)의 악성코드의 제어에 따라 임무를 수행하는 구조로 되어있다. 위에서 프로세스 정보 확인 시에는
    명시적으로 악성코드 명처럼 보이거나 은닉된 프로세스가 없었기 때문에 DLL 인젝션일 수 있겠다는 생각에 ldrmodules 플러그인을 이용하여
    프로세스에 로딩된 모든 DLL을 추출하였다. 이 중 DLL에서 루트킷 드라이버와 동일한 파일 명(irykmmww)의 DLL이 존재함을 확인하였다.\n<pre
    class=\"lang:sh decode:true\" title=\"ldrmodules\">$ python vol.py -f APT.img
    --profile=WinXPSP3x86 ldrmodules | findstr irykmmww\nVolatility Foundation Volatility
    Framework 2.3.1\n     884 svchost.exe          0x10000000 True   True   True  WINDOWSsystem32irykmmww.d1l\n
    \   1672 explorer.exe         0x00970000 True   True   True  WINDOWSsystem32irykmmww.dll\n
    \   1984 VMwareTray.exe       0x00a40000 True   True   True  WINDOWSsystem32irykmmww.dll\n
    \   2004 VMwareUser.exe       0x00fb0000 True   True   True  WINDOWSsystem32irykmmww.dll\n
    \   2020 ctfmon.exe           0x10000000 True   True   True  WINDOWSsystem32irykmmww.dll\n
    \    796 iexplore.exe         0x00150000 True   True   True  WINDOWSsystem32irykmmww.dll\n
    \    796 iexplore.exe         0x10000000 True   True   True  WINDOWSsystem32irykmmww.d1l</pre>\n&nbsp;\n<h2>2.
    악성코드로 의심되는 파일 추출</h2>\n위에서 메모리를 분석한 결과 의심되는 파일은 크게 'irykmmww.sys', 'irykmmww.dll',
    'irykmmww.d1l'이였다. 일단 메모리에 있는 DLL을 덤프하기 위해 dlldump 플러그인을 사용하였다.\n<pre class=\"lang:sh
    decode:true\" title=\"dlldump\">$ python vol.py -f APT.img --profile=WinXPSP3x86
    dlldump -r irykmmww --dump-dir dumped\nVolatility Foundation Volatility Framework
    2.3.1\nProcess(V) Name                 Module Base Module Name          Result\n----------
    -------------------- ----------- -------------------- ------\n0x81e54da0 svchost.exe
    \         0x010000000 irykmmww.d1l         OK: module.884.2054da0.10000000.dll\n0x81da71a8
    explorer.exe         0x000970000 irykmmww.dll         OK: module.1672.1fa71a8.970000.dll\n0x81f1c7e8
    VMwareTray.exe       0x000a40000 irykmmww.dll         OK: module.1984.211c7e8.a40000.dll\n0x81dc1a78
    VMwareUser.exe       0x000fb0000 irykmmww.dll         OK: module.2004.1fc1a78.fb0000.dll\n0x81f1a650
    ctfmon.exe           0x010000000 irykmmww.dll         OK: module.2020.211a650.10000000.dll\n0x81dbdda0
    iexplore.exe         0x010000000 irykmmww.d1l         OK: module.796.1fbdda0.10000000.dll\n0x81dbdda0
    iexplore.exe         0x000150000 irykmmww.dll         OK: module.796.1fbdda0.150000.dll</pre>\n추출한
    파일의 해시 정보(프로세스 명, PID, 모듈 명, 덤프된 모듈 명, MD5)는 다음과 같다.\n<ul>\n\t<li>svchost.exe,
    884, irykmmww.d1l, module.884.2054da0.10000000.dll, f4ec994a7e06fae31fb6fcd9444f6910</li>\n\t<li>explorer.exe, 796, irykmmww.dll, module.1672.1fa71a8.970000.dll, 319bff282b3046e6c85bbe0e67338c72</li>\n\t<li>VMwareTray.exe, 1984, irykmmww.dll, module.1984.211c7e8.a40000.dll, decc38255c9b68d75ac7b9f6066891af</li>\n\t<li>VMwareUser.exe, 2004, irykmmww.dll, module.2004.1fc1a78.fb0000.dll, 830e8e4c71899b5756c41fb48767dd76</li>\n\t<li>ctfmon.exe, 2020, irykmmww.dll, module.2020.211a650.10000000.dll, 0f5edfce522450459b76357b3773b449</li>\n\t<li>iexplore.exe, 796, irykmmww.d1l, module.796.1fbdda0.10000000.dll, 5d00d18dc0126aa48ba51e910aa067bb</li>\n\t<li>iexplore.exe, 796,
    irykmmww.dll, module.796.1fbdda0.150000.dll, 650fd54ba6800ec59a48dd7ca5bd1103</li>\n</ul>\nvolatiliy의
    moddump 플러그인으로 드라이버를 덤프하였다.\n<pre class=\"lang:sh decode:true\" title=\"moddump\">$
    python vol.py -f APT.img --profile=WinXPSP3x86 moddump -r irykmmww --dump-dir
    dumped\nVolatility Foundation Volatility Framework 2.3.1\nModule Base Module Name
    \         Result\n----------- -------------------- ------\n0x0f836f000 irykmmww.sys
    \        OK: driver.f836f000.sys</pre>\n덤프한 파일의 정보(서비스 명, 순서, 모듈 명, 덤프된 모듈 명,
    MD5)는 다음과 같다.\n<ul>\n\t<li>IRYKMMWW, 252, irykmmww.sys, driver.f836f000.sys, f7400a2b01aeb16dc7c78af0f112f23f</li>\n</ul>\n본
    글에서는 메모리 포렌식을 통해 의심되는 서비스를 식별하고 이를 토대로 로드된 디바이스 드라이버와 인젝셔된 악성 DLL을 추출하는 것을 다루었다.
    메모리 분석을 하면 악성 연결도 메모리 분석에서 확인할 수 있으나, 이는 추 후 덤프한 바이너리 분석 부에서 네트워크 연결과 관련된 코드 분석
    시 언급하도록 하겠다. 다음 글부터 추출한 파일에 대한 본격적인 악성코드 분석을 진행하겠다."
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>본 글은 2013년 12월부터 2014년 1월까지 진행한 DFIRCON에 대한 글이다.</p>
<p>SANS는 매 년말에 이벤트성 행사를 하는데, 요번에는 SANS의 Simulcast(온라인 강의) 티켓을 한장 걸고 행사를 진행했다. 사실 온라인 대회에 티켓 한 장은 인간적으로 너무 적은 보상이긴 했지만 나름 재미있을 것 같아 참여하였다. 결과적으로는 Rob Lee가 말한 결과와 다른 분석 결과가 나왔지만, 답이 나온 아직까지도 왜 그게 정확한 답인지 잘 모르겠다. 문제를 낸 사람은 악성코드를 직접 돌렸으니 알 수 있겠지만, 메모리 분석 및 덤프한 바이너리 분석까지 해본 나로써는 해당 답을 도출하기에는 여러가지 제약사항이 있었다. 이에 본 메모리 이미지 분석을 통해 내 자신의 의견을 게재하고 다양한 관점의 이야기를 해보고자 한다.</p>
<p><a href="https://www.surveymonkey.com/s/JQ9QFHP" target="_blank">DFIRCON APT Memory Analysis Challenge</a>는 메모리 이미지를 분석하여 총 5개의 문제에 대한 답을 도출하는 것으로 문제에 대한 특별한 룰은 존재하지 않는다. 상황도 없다. 그냥 분석해서 답을 채우기만 하면 되는 문제이다. 포렌식 문제임에도 아무런 상황을 제공하지 않은건 나름 포렌식 교육의 대표 기관 중 하나인 SANS에서 낸 문제치고는 아쉬움이 많았다. 아무래도 상황이 없이 디지털 증거 중 일부 데이터만으로 분석을 할 경우, 포렌식에서 가장 중요한 사건 '추리'를 전혀할 수 없으며 이 부분에 대해 추정을 해야하는 문제가 생긴다. 다음 문제에는 좀 더 포렌식 다운 문제를 제공하였으면 한다. 이러한 상황을 고려하여 본 연재에서는 일단 메모리 이미지 분석만으로 유추해볼 수 있는 여러 시나리오를 작성해보도록 하겠다.</p>
<h2>1. 정보 수집(메모리 분석)</h2>
<p>제공 받은 메모리 이미지는 'APT.img'로 MD5는 14AC99E2E6980D6A962F0AC2A1CB65C8이다. 메모리 이미지를 분석하기 위해 volatility framework를 사용하였으며, imageinfo 결과 Windows XP SP3 x86 으로 확인되었다. 일단 프로세스 목록을 확인하기 위해 pstree 플러그인을 수행하였다.</p>
<pre class="lang:sh decode:true" title="pstree on APT.img">$ python vol.py -f APT.img --profile=WinXPSP3x86 pstree
Volatility Foundation Volatility Framework 2.3.1
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0x823c8830:System                                      4      0     55    254 1970-01-01 00:00:00 UTC+0000
. 0x8230aad8:smss.exe                                 564      4      3     19 2009-04-16 16:10:01 UTC+0000
.. 0x81f63020:winlogon.exe                            660    564     16    502 2009-04-16 16:10:06 UTC+0000
... 0x81f22020:services.exe                           704    660     15    254 2009-04-16 16:10:06 UTC+0000
.... 0x81f739b0:svchost.exe                          1088    704     70   1445 2009-04-16 16:10:07 UTC+0000
..... 0x81f96220:wscntfy.exe                         1260   1088      1     39 2009-04-16 16:10:22 UTC+0000
.... 0x81da4590:svchost.exe                           968    704     10    241 2009-04-16 16:10:07 UTC+0000
.... 0x81dc2570:VMwareService.e                      1032    704      3    175 2009-04-16 16:10:16 UTC+0000
.... 0x8231eda0:msiexec.exe                          1464    704      6    294 2009-04-16 16:11:02 UTC+0000
.... 0x81e54da0:svchost.exe                           884    704     17    208 2009-04-16 16:10:07 UTC+0000
..... 0x81dbdda0:iexplore.exe                         796    884      8    152 2009-05-05 19:28:28 UTC+0000
.... 0x81e91da0:svchost.exe                          1212    704     14    208 2009-04-16 16:10:09 UTC+0000
.... 0x81d33628:alg.exe                               464    704      6    105 2009-04-16 16:10:21 UTC+0000
.... 0x8219b630:spoolsv.exe                          1512    704     10    129 2009-04-16 16:10:10 UTC+0000
.... 0x822cb458:vmacthlp.exe                          872    704      1     25 2009-04-16 16:10:07 UTC+0000
.... 0x8232c020:svchost.exe                          1140    704      5     60 2009-04-16 16:10:08 UTC+0000
... 0x82164da0:lsass.exe                              716    660     21    342 2009-04-16 16:10:06 UTC+0000
.. 0x822ca2c0:csrss.exe                               636    564     10    356 2009-04-16 16:10:06 UTC+0000
 0x81da71a8:explorer.exe                             1672   1624     15    586 2009-04-16 16:10:10 UTC+0000
. 0x81f1c7e8:VMwareTray.exe                          1984   1672      1     37 2009-04-16 16:10:11 UTC+0000
. 0x81e4d648:cmd.exe                                  840   1672      1     33 2009-05-05 15:56:24 UTC+0000
.. 0x82161558:MIRAgent.exe                            456    840      1     77 2009-05-05 19:28:40 UTC+0000
. 0x81dc1a78:VMwareUser.exe                          2004   1672      8    228 2009-04-16 16:10:11 UTC+0000
. 0x81f1a650:ctfmon.exe                              2020   1672      1     71 2009-04-16 16:10:11 UTC+0000</pre>
<p>여기서 중요하게 봐야하는 점은 '프로세스 간의 부모-자식 관계', '프로세스 생성 시간', '프로세스 명'을 들 수 있다. 일단 884번 프로세스인 svchost.exe 아래에 796번 프로세스인 iexplore.exe가 있는 것을 볼 수 있다. iexplore.exe는 인터넷 익스플로러(Internet Explorer)로 보통 explorer.exe의 자식 프로세스여야 맞는데 마치 서비스 형태로 동작한 것처럼 보인다. iexplore.exe의 사용자 권한을 보면 문제가 있음이 좀 더 명확해진다.</p>
<pre class="lang:sh decode:true" title="getsids on internet explorer">$ python vol.py -f APT.img --profile=WinXPSP3x86 getsids -p 796
iexplore.exe (796): S-1-5-18 (Local System)
iexplore.exe (796): S-1-5-32-544 (Administrators)
iexplore.exe (796): S-1-1-0 (Everyone)
iexplore.exe (796): S-1-5-11 (Authenticated Users)</pre>
<p>분석 결과를 보면 로컬 시스템 권한을 가지고 있다. 프로세스 분석에서 볼 수 있는 또하나 재미있는 사실은 시간 정보이다. 분석 결과를 보면, MIRAgent.exe, cmd.exe, iexplore.exe는 2009년 5월 5일에 동작했으며, 나머지 프로세스는 2009년 4월 16일에 실행되었다. 구동 중인 환경이 가상 머신(VMware)인 것을 고려해볼 때, 문제 제작자가 2009년 4월 16일로 설정된 가상머신을 구동을 시키고 사용하다가 출제를 위해 5월 5일에 관련된 프로세스를 실행했거나, 고의적으로 시간 값을 돌렸을 가능성이 있다. MIRAgent.exe는 Mandiant의 Incident Response Agent로 악성코드와는 무관하다.</p>
<p>혹시 자기자신을 숨기는 프로세스가 있을 수도 있기 떄문에 psxview로 분석을 수행하였으나 은닉된 프로세스는 발견되지 않았다.</p>
<pre class="lang:sh decode:true" title="psxview">$ python vol.py -f APT.img --profile=WinXPSP3x86 psxview
Volatility Foundation Volatility Framework 2.3.1
Offset(P)  Name                    PID pslist psscan thrdproc pspcid csrss session deskthrd
---------- -------------------- ------ ------ ------ -------- ------ ----- ------- --------
0x02163020 winlogon.exe            660 True   True   True     True   True  True   True
0x02122020 services.exe            704 True   True   True     True   True  True   True
0x0211a650 ctfmon.exe             2020 True   True   True     True   True  True   True
0x01fa71a8 explorer.exe           1672 True   True   True     True   True  True   True
0x0252c020 svchost.exe            1140 True   True   True     True   True  True   True
0x0204d648 cmd.exe                 840 True   True   True     True   True  True   True
0x01fc1a78 VMwareUser.exe         2004 True   True   True     True   True  True   True
0x02054da0 svchost.exe             884 True   True   True     True   True  True   True
0x02196220 wscntfy.exe            1260 True   True   True     True   True  True   True
0x021739b0 svchost.exe            1088 True   True   True     True   True  True   True
0x01fa4590 svchost.exe             968 True   True   True     True   True  True   True
0x02361558 MIRAgent.exe            456 True   True   True     True   True  True   True
0x02364da0 lsass.exe               716 True   True   True     True   True  True   False
0x0211c7e8 VMwareTray.exe         1984 True   True   True     True   True  True   True
0x02091da0 svchost.exe            1212 True   True   True     True   True  True   True
0x01fbdda0 iexplore.exe            796 True   True   True     True   True  True   True
0x024cb458 vmacthlp.exe            872 True   True   True     True   True  True   True
0x0239b630 spoolsv.exe            1512 True   True   True     True   True  True   True
0x0251eda0 msiexec.exe            1464 True   True   True     True   True  True   True
0x01f33628 alg.exe                 464 True   True   True     True   True  True   True
0x01fc2570 VMwareService.e        1032 True   True   True     True   True  True   True
0x0250aad8 smss.exe                564 True   True   True     True   False False   False
0x025c8830 System                    4 True   True   True     True   False False   False
0x024ca2c0 csrss.exe               636 True   True   True     True   False True   True
0x03178220 wscntfy.exe            1260 False  True   False    False  False False   False
0x0c605020 svchost.exe            1140 False  True   False    False  False False   False
0x0ad69da0 iexplore.exe            796 False  True   False    False  False False   False
0x0edd0628 alg.exe                 464 False  True   False    False  False False   False
0x032b3da0 svchost.exe             884 False  True   False    False  False False   False
0x0eed3628 alg.exe                 464 False  True   False    False  False False   False
0x10b54628 alg.exe                 464 False  True   False    False  False False   False
0x15934830 System                    4 False  True   False    False  False False   False
0x1b217da0 iexplore.exe            796 False  True   False    False  False False   False
0x04097020 svchost.exe            1140 False  True   False    False  False False   False
0x035c1590 svchost.exe             968 False  True   False    False  False False   False
0x07b1ada0 iexplore.exe            796 False  True   False    False  False False   False
0x0edd59b0 svchost.exe            1088 False  True   False    False  False False   False
0x12f3dda0 svchost.exe             884 False  True   False    False  False False   False</pre>
<p>네트워크 연결 정보는 다음과 같다.</p>
<pre class="lang:sh decode:true" title="network connection">$ python vol.py -f APT.img --profile=WinXPSP3x86 connections
Volatility Foundation Volatility Framework 2.3.1
Offset(V)  Local Address             Remote Address            Pid
---------- ------------------------- ------------------------- ---
0x81e611f8 192.168.157.10:1053       218.85.133.23:89          796
$
$ python vol.py -f APT.img --profile=WinXPSP3x86 connscan
Volatility Foundation Volatility Framework 2.3.1
Offset(P)  Local Address             Remote Address            Pid
---------- ------------------------- ------------------------- ---
0x0205ece0 192.168.157.10:1050       222.128.1.2:443           1672
0x020611f8 192.168.157.10:1053       218.85.133.23:89          796
0x032c01f8 192.168.157.10:1053       218.85.133.23:89          796
0x0337dce0 192.168.157.10:1050       222.128.1.2:443           1672
0x08a4ace0 192.168.157.10:1050       222.128.1.2:443           1672
0x18200ce0 192.168.157.10:1050       222.128.1.2:443           1672</pre>
<p>explorer.exe(1672)는 HTTPS 포트인 443으로 통신하고 있었으며, iexplore.exe(796)는 89번 포트로 통신을 하고 있었다. 두 IP 다 China Telecom에서 관리하고 있으며, 중국 내의 서버로 확인할 수 있다.</p>
<blockquote><p>실제 바이너리 분석을 해보면 222.128.1.2에 연결한 흔적은 찾을 수 없었다. 또한 explorer.exe에 인젝션된 DLL은 별도의 소켓 통신을 하진 않는 것으로 보인다.</p></blockquote>
<p>서비스를 확인해보니, 252번째 순서의 서비스(irykmmww)가 매우 의심스러운 서비스 명을 하고 있었다. 서비스 명을 키워드로 드라이버 목록인 driverscan으로 확인하면 정상적으로 메모리에 로드되어 있는 것을 확인할 수 있다.</p>
<pre class="lang:sh decode:true" title="svcscan and driverscan">$ python vol.py -f APT.img --profile=WinXPSP3x86 svcscan
.....
Offset: 0x38ab98
Order: 252
Process ID: -
Service Name: irykmmww
Display Name: irykmmww
Service Type: SERVICE_KERNEL_DRIVER
Service State: SERVICE_RUNNING
Binary Path: Driverirykmmww

$ python vol.py -f APT.img --profile=WinXPSP3x86 driverscan | findstr irykmmww
Volatility Foundation Volatility Framework 2.3.1
0x024c33b8    3    0 0xf836f000     0x3900 irykmmww             irykmmww     Driverirykmmww</pre>
<p>서비스 등록은 윈도우에서 커널 드라이버를 로드하는 방법 중 가장 안정적인 방법이다. 드라이버를 로드하기 위한 다양한 비정상적인 방법이 있지만 보통 시스템의 불안정성을 최소화하면서 손쉽게 로드/언로드가 가능한 이점이 있다보니 루트킷 설치에도 서비스 등록이 많이 사용된다. volatility에서는 루트킷이 후킹한 데이터를 탐지하는 여러가지 방법이 있으며, 이 중 ssdt(System Service Descriptor Table)에서 루트킷이 후킹한 함수를 확인할 수 있다.</p>
<pre class="lang:sh decode:true" title="SSDT">$ python vol.py -f APT.img --profile=WinXPSP3x86 ssdt | grep irykmmww
Volatility Foundation Volatility Framework 2.3.1
  Entry 0x0042: 0xf836fe9c (NtDeviceIoControlFile) owned by irykmmww.sys
  Entry 0x0047: 0xf83706dc (NtEnumerateKey) owned by irykmmww.sys
  Entry 0x0049: 0xf837075e (NtEnumerateValueKey) owned by irykmmww.sys
  Entry 0x0077: 0xf837028f (NtOpenKey) owned by irykmmww.sys
  Entry 0x0091: 0xf8370a8c (NtQueryDirectoryFile) owned by irykmmww.sys
  Entry 0x00ad: 0xf836fe3e (NtQuerySystemInformation) owned by irykmmww.sys
  Entry 0x00b1: 0xf837091a (NtQueryValueKey) owned by irykmmww.sys</pre>
<p>장치와 애플리케이션간 통신에 사용되는 DeviceIoControlFile과 레지스트리 관리 및 파일 관리를 위한 Native 함수를 후킹한 것을 확인하였다. 그리고 후킹에 사용된 드라이버가 'irykmmww.sys'인 것도 확인할 수 있었다. 루트킷은 보통 드라이버 단독적으로 활동하지 않으며 프로세스나 DLL과 같은 사용자 영역(User Space)의 악성코드의 제어에 따라 임무를 수행하는 구조로 되어있다. 위에서 프로세스 정보 확인 시에는 명시적으로 악성코드 명처럼 보이거나 은닉된 프로세스가 없었기 때문에 DLL 인젝션일 수 있겠다는 생각에 ldrmodules 플러그인을 이용하여 프로세스에 로딩된 모든 DLL을 추출하였다. 이 중 DLL에서 루트킷 드라이버와 동일한 파일 명(irykmmww)의 DLL이 존재함을 확인하였다.</p>
<pre class="lang:sh decode:true" title="ldrmodules">$ python vol.py -f APT.img --profile=WinXPSP3x86 ldrmodules | findstr irykmmww
Volatility Foundation Volatility Framework 2.3.1
     884 svchost.exe          0x10000000 True   True   True  WINDOWSsystem32irykmmww.d1l
    1672 explorer.exe         0x00970000 True   True   True  WINDOWSsystem32irykmmww.dll
    1984 VMwareTray.exe       0x00a40000 True   True   True  WINDOWSsystem32irykmmww.dll
    2004 VMwareUser.exe       0x00fb0000 True   True   True  WINDOWSsystem32irykmmww.dll
    2020 ctfmon.exe           0x10000000 True   True   True  WINDOWSsystem32irykmmww.dll
     796 iexplore.exe         0x00150000 True   True   True  WINDOWSsystem32irykmmww.dll
     796 iexplore.exe         0x10000000 True   True   True  WINDOWSsystem32irykmmww.d1l</pre>
<p>&nbsp;</p>
<h2>2. 악성코드로 의심되는 파일 추출</h2>
<p>위에서 메모리를 분석한 결과 의심되는 파일은 크게 'irykmmww.sys', 'irykmmww.dll', 'irykmmww.d1l'이였다. 일단 메모리에 있는 DLL을 덤프하기 위해 dlldump 플러그인을 사용하였다.</p>
<pre class="lang:sh decode:true" title="dlldump">$ python vol.py -f APT.img --profile=WinXPSP3x86 dlldump -r irykmmww --dump-dir dumped
Volatility Foundation Volatility Framework 2.3.1
Process(V) Name                 Module Base Module Name          Result
---------- -------------------- ----------- -------------------- ------
0x81e54da0 svchost.exe          0x010000000 irykmmww.d1l         OK: module.884.2054da0.10000000.dll
0x81da71a8 explorer.exe         0x000970000 irykmmww.dll         OK: module.1672.1fa71a8.970000.dll
0x81f1c7e8 VMwareTray.exe       0x000a40000 irykmmww.dll         OK: module.1984.211c7e8.a40000.dll
0x81dc1a78 VMwareUser.exe       0x000fb0000 irykmmww.dll         OK: module.2004.1fc1a78.fb0000.dll
0x81f1a650 ctfmon.exe           0x010000000 irykmmww.dll         OK: module.2020.211a650.10000000.dll
0x81dbdda0 iexplore.exe         0x010000000 irykmmww.d1l         OK: module.796.1fbdda0.10000000.dll
0x81dbdda0 iexplore.exe         0x000150000 irykmmww.dll         OK: module.796.1fbdda0.150000.dll</pre>
<p>추출한 파일의 해시 정보(프로세스 명, PID, 모듈 명, 덤프된 모듈 명, MD5)는 다음과 같다.</p>
<ul>
<li>svchost.exe, 884, irykmmww.d1l, module.884.2054da0.10000000.dll, f4ec994a7e06fae31fb6fcd9444f6910</li>
<li>explorer.exe, 796, irykmmww.dll, module.1672.1fa71a8.970000.dll, 319bff282b3046e6c85bbe0e67338c72</li>
<li>VMwareTray.exe, 1984, irykmmww.dll, module.1984.211c7e8.a40000.dll, decc38255c9b68d75ac7b9f6066891af</li>
<li>VMwareUser.exe, 2004, irykmmww.dll, module.2004.1fc1a78.fb0000.dll, 830e8e4c71899b5756c41fb48767dd76</li>
<li>ctfmon.exe, 2020, irykmmww.dll, module.2020.211a650.10000000.dll, 0f5edfce522450459b76357b3773b449</li>
<li>iexplore.exe, 796, irykmmww.d1l, module.796.1fbdda0.10000000.dll, 5d00d18dc0126aa48ba51e910aa067bb</li>
<li>iexplore.exe, 796, irykmmww.dll, module.796.1fbdda0.150000.dll, 650fd54ba6800ec59a48dd7ca5bd1103</li>
</ul>
<p>volatiliy의 moddump 플러그인으로 드라이버를 덤프하였다.</p>
<pre class="lang:sh decode:true" title="moddump">$ python vol.py -f APT.img --profile=WinXPSP3x86 moddump -r irykmmww --dump-dir dumped
Volatility Foundation Volatility Framework 2.3.1
Module Base Module Name          Result
----------- -------------------- ------
0x0f836f000 irykmmww.sys         OK: driver.f836f000.sys</pre>
<p>덤프한 파일의 정보(서비스 명, 순서, 모듈 명, 덤프된 모듈 명, MD5)는 다음과 같다.</p>
<ul>
<li>IRYKMMWW, 252, irykmmww.sys, driver.f836f000.sys, f7400a2b01aeb16dc7c78af0f112f23f</li>
</ul>
<p>본 글에서는 메모리 포렌식을 통해 의심되는 서비스를 식별하고 이를 토대로 로드된 디바이스 드라이버와 인젝셔된 악성 DLL을 추출하는 것을 다루었다. 메모리 분석을 하면 악성 연결도 메모리 분석에서 확인할 수 있으나, 이는 추 후 덤프한 바이너리 분석 부에서 네트워크 연결과 관련된 코드 분석 시 언급하도록 하겠다. 다음 글부터 추출한 파일에 대한 본격적인 악성코드 분석을 진행하겠다.</p>
