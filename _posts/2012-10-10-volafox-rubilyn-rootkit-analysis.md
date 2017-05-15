---
layout: post
title: 'volafox : rubilyn Rootkit Analysis'
date: 2012-10-10 08:55:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/10/volafox-rubilyn-rootkit-analysis.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/5018343123744516551
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '856'
  fusion_builder_content_backup: "<h3>1. 소개</h3>nullsecurity에서 rubilyn이라는 루트킷을 공개하였다.
    해당 루트킷은 바이너리 뿐만아니라 소스코드가 함께 있기 때문에, 새로운 Mac OS X 에 맞게 적절하게 리빌드하여 사용할 수 있다.<br
    /><br />코드 : <a href=\"http://www.nullsecurity.net/backdoor.html\">http://www.nullsecurity.net/backdoor.html</a><br
    /><br />해당 코드의 description 내용은 다음과 같다.<br /><br /><blockquote><span>64bit Mac
    OS-X kernel rootkit that uses no hardcoded address to hook the BSD subsystem in
    all OS-X Lion & below. It uses a combination of syscall hooking and DKOM to hide
    activity on a host. String resolution of symbols no longer works on Mountain Lion
    as symtab is destroyed during load, this code is portable on all Lion & below
    but requires re-working for hooking under Mountain Lion.</span></blockquote><br
    />이 포스팅에선 rubilyn 루트킷이 설치된 시스템을 분석하고자 할때, 메모리 이미지 분석을 통해 루트킷을 효과적으로 파악하고 분석할 수
    있는지에 대해 알아보겠다.<br /><br />이 분석에서 필자가 원하는 부분과 그 대상은 다음과 같다.<br /><br /><ol><li>루트킷
    기법에 대해 메모리 분석 툴의 효용성 입증 - 볼라폭스 개발팀 그리고 독자</li><li>현재 메모리 분석 도구의 한계점 도출 - 볼라폭스
    개발팀</li><li>실제 상황에 존재할 수 있는 루트킷을 분석을 통해 메모리 분석의 중요성을 일깨워주기 - 이 글을 읽는 독자</li></ol><br
    /><br />일단 설정한 상황을 미리 제시하고 각각의 은닉 및 변조 기법을 메모리에서 어떻게 탐지하는지를 확인하는 것이 좀 더 재밌을 것이라
    생각되어, 셋팅한 내용을 다음과 같이 기록한다.<br />(<u>물론 아래의 분석 단계에선 셋팅한 내용을 모르는 상태에서 행동한 모든 내용을
    알아내도록 기술할 것이다.</u>)<br /><br />루트킷 설정은 다음과 같다.<br /><br /><ul><li>kextstat, netstat,
    grep, w, who 명령으로부터 자기자신을 은닉</li><li>PID 258 프로세스(bash)에 은닉 기법 적용</li><li>22번
    포트로 접근하는 모든 세션 숨김</li><li>루트킷 관련 파일이 Finder(파일 탐색기)에서 보이지 않도록 함.</li></ul>루트킷이
    설치된 시스템 환경은 다음과 같다.<br /><br /><ul><li>OS: Mac OS X Lion</li><li>Architecture
    : Intel x86_64bit</li></ul><br /><br /><h3>2. 라이브 정보 수집하기</h3>보통 사고 대응을 나가면 가장
    먼저 해야할 일이 라이브 정보를 수집하는 일이다. 아무래도 네트워크 정보나 프로세스 정보는 실시간으로 변화가 일어나기 때문에, 간발의 차이로
    루트킷에 대한 흔적을 찾지 못할 수 있기 때문이다.<br /><br />우선 라이브 상태에서는 위의 4가지 정보가 완벽하게 숨겨져 있었다.
    즉 스크립트 또는 명령어 입력으로 수집한 라이브 데이터는 루트킷을 찾아내는데 전혀 효용성이 없다고 볼 수 있다.<br /><br />그렇다고해서,
    라이브 정보 수집을 제외해야 할 것인가?하면 그건 또 아니다. 쓸모 없는 정보를 괜히 수집할 필요가 있을까 싶지만, 라이브 정보는 추 후 진행할
    메모리 분석 결과와 비교하는데 큰 도움이 된다.<br /><br />예를 들어, 시스템에 프로세스가 400개가 구동 중이고 이 중 하나의 프로세스만
    악성코드인 상황에서 모든 프로세스를 찾아낼 수 있는 메모리 분석 도구로 메모리 분석을 진행한다면 이 400개를 모두 의심하는 상태에서 분석을
    진행해야 할 것이다. 하지만, 변조된 결과를 받은 라이브 데이터와 메모리 분석 결과를 비교해보면 메모리 분석 결과에만 나타나는 프로세스를 발견할
    수 있을 것이며, 이 프로세스는 루트킷과 관련된 프로세스일 가능성이 매우 높게 된다(물론 상황에 따라 메모리 이미징 시점에 새로운 프로세스가
    생성될 수도 있긴하지만 다양한 방법으로 추가 생성된 프로세스는 판단할 수 있으므로, 이는 논외로 생각하자). 이러한 판단은 프로세스 뿐만 아니라
    드라이버 및 네트워크 정보에서도 비교해볼 수 있기 때문에 가능하다면 라이브 정보는 꼭 수집하는 것을 권장한다.<br /><br /><b><i>*
    단 이번 포스팅에서는 라이브 데이터와 일일히 비교하는 것을 포스팅하면 해당 데이터가 지면을 너무 많이 할애할 것이기 때문에, 메모리 분석 정보만으로
    루트킷을 찾도록 하겠다.</i></b><br /><br /><h3>3. 메모리 이미징 수행</h3>Mac OS X 10.7.2 이상부터는 FW를
    이용하여 메모리 이미징하는 행위를 방지한다고 알려져있다.(맥이 한대로 아직 확인을 못해봤지만, 내가아는 애플이라면 충분히 가능하다.) 그래서
    최근에는 윈도우와 같이 소프트웨어 기반의 메모리 이미징을 추천하고 있다.<br /><br />메모리 이미징의 가장 대표적인 도구는 MacMemoryReader로
    커널 코어 덤프(풀 덤프)를 수행해주는 툴이다. 이 도구로 이미징한 <u>코어 덤프 파일은 volafox에 있는 flatten.py 를 이용하여
    volafox에서 분석 가능한 포맷으로 변경</u>할 수 있다.<br /><br />만약에 지금 상황과 같이 악성코드나 루트킷 샘플을 분석하는
    것이라면, vmware의 메모리 이미지를 바로 복사해서 사용할 수 있다.vmware를 suspend 상태로 두고 해당 vmem 파일을 복사해오면
    volafox에서 바로 분석이 가능하다. 본 분석에는 vmware의 메모리 이미지를 이용하여 분석을 진행하였다. 물론 코어 덤프를 이용해도
    동일한 분석이 가능하다. :)<br /><br /><h3>4. 기본 정보 확인</h3>메모리 이미지를 가져오면 volafox에서 바로 분석을
    진행할 수 있다.<br /><br /><br /><span>James-ui-MacBook-Pro:volafox n0fate$ python vol.py</span><br
    /><span><br /></span><span>volafox: Mac OS X Memory Analysis Toolkit</span><br
    /><span>project: http://code.google.com/p/volafox</span><br /><span>support: 10.6-8;
    32/64-bit kernel</span><br /><span>  input: *.vmem (VMWare memory file), *.mmr
    (Mac Memory Reader, flattened x86, IA-32e)</span><br /><span>  usage: python vol.py
    -i IMAGE [-o COMMAND [-vp PID][-x PID][-x KEXT_ID][-x TASKID]]</span><br /><span><br
    /></span><span>Options:</span><br /><span>-o CMD            : Print kernel information
    for CMD (below)</span><br /><span>-p PID            : List open files for PID
    (where CMD is \"lsof\")</span><br /><span>-v                : Print all files,
    including unsupported types (where CMD is \"lsof\")</span><br /><span>-x PID/KID/TASKID
    : Dump process/task/kernel extension address space for PID/KID/Task ID (where
    CMD is \"ps\"/\"kextstat\"/\"tasks\")</span><br /><span><br /></span><span>COMMANDS:</span><br
    /><span>system_profiler : Kernel version, CPU, and memory spec, Boot/Sleep/Wakeup
    time</span><br /><span>mount           : Mounted filesystems</span><br /><span>kextstat
           : KEXT (Kernel Extensions) listing</span><br /><span>ps              :
    Process listing</span><br /><span>tasks           : Task listing (& Matching Process
    List)</span><br /><span>systab          : Syscall table (Hooking Detection)</span><br
    /><span>mtt             : Mach trap table (Hooking Detection)</span><br /><span>netstat
            : Network socket listing (Hash table)</span><br /><span>lsof          
     : Open files listing by process (research, osxmem@gmail.com)</span><br /><span>pestate
            : Show Boot information (experiment)</span><br /><span>efiinfo        
    : EFI System Table, EFI Runtime Services(experiment)</span><br /><span>keychaindump
       : Dump master key candidates for decrypting keychain(Lion, ML)</span><br /><span>James-ui-MacBook-Pro:volafox
    n0fate$ </span><br /><br /><br />보통은 시스템의 기본 정보를 확인하기 위해 system_profiler 명령을 우선
    사\n용한다.<br /><br /><br /><span>James-ui-MacBook-Pro:volafox n0fate$ python vol.py
    -i ~/Desktop/rubilyn_evidence.vmem -o system_profiler</span><br\n><span>[+] Mac
    OS X Basic Information</span><br /><span> [-] Darwin kernel Build Number: <strong><span>11E53</span></strong></span><br
    /><span> [-] Darwin Kernel Major Version: 11</span><br /><span> [-] Darwin Kernel
    Minor Version: 4</span><br /><span> [-] Number of Physical CPUs: 1</span><br /><span> [-]
    Size of memory in bytes: 2147483648 bytes</span><br /><span> [-] Size of physical
    memory: 2147483648 bytes</span><br /><span> [-] Number of physical CPUs now available:
    1</span><br /><span> [-] Max number of physical CPUs now possible: 1</span><br
    /><span> [-] Number of logical CPUs now available: 1</span><br /><span> [-] Max
    number of logical CPUs now possible: 1</span><br /><span> [-] Last Hibernated
    Sleep Time: Thu Jan 01 00:00:00 1970 (GMT +0)</span><br /><span> [-] Last Hibernated
    Wake Time: Thu Jan 01 00:00:00 1970 (GMT +0)</span><br /><span>James-ui-MacBook-Pro:volafox
    n0fate$ </span><br /><br /><br />다윈 커널 버전인 11.4는 Mac OS X Lion(11)을 의미한다. 이외에
    메모리 용량 같은 부가적인 정보를 확인할 수 있다.<br /><br /><h3>5. 시스템 콜 분석</h3>필자가 보통 루트킷이 감염된 메모리
    이미지를 분석할 때는 커널 레벨로 접근할 수 있는 방법을 제공하는 KEXT와 시스템 콜/Mach Trap 테이블 분석을 우선적으로 진행한다.
    일단, 시스템 콜 목록과 Mach Trap 테이블을 확인하겠다. 전체 리스트를 출력하면 코드가 길어지므로 후킹으로 판단되는 콜 정보만 추출하겠다.<br
    /><br /><br /><span>James-ui-MacBook-Pro:volafox n0fate$ python vol.py -i ~/Desktop/rubilyn_evidence.vmem
    -o mtt | grep HOOK</span><br /><span>NUM  ARG_COUNT                        CALL_NAME
              CALL_PTR HOOK_FINDER</span><br /><span>James-ui-MacBook-Pro:volafox
    n0fate$ python vol.py -i ~/Desktop/rubilyn_evidence.vmem -o systab | grep HOOK</span><br
    /><span>NUM  ARG_COUNT RESV FLAGS                        CALL_PTR    ARG_MUNGE32_PTR
    ARG_MUNGE64_PTR RET_TYPE ARG_BYTES HOOK_FINDER</span><br /><span>222          8
       0     0              0xFFFFFF7F8079241D 0xFFFFFF80005CEEA0      0x00000000
           1        32 SYSCALL HOOKING</span><br /><span>344          4    0     0
                 0xFFFFFF7F807922EE 0xFFFFFF80005CEE60      0x00000000        6  
         16 SYSCALL HOOKING</span><br /><span>397          3    0     0          
       0xFFFFFF7F80792A7E 0xFFFFFF80005CEE50      0x00000000        6        12 SYSCALL
    HOOKING</span><br /><span>427          4    0     0              0xFFFFFF8000302E20
    0xFFFFFF80005CEFD0      0x00000000        6        20 SYSCALL HOOKING</span><br
    /><span>James-ui-MacBook-Pro:volafox n0fate$ </span><br /><br /><br />분석 결과를 보면
    mtt 커맨드인 Mach Trap Table에는 이상이 발견되지 않았지만, 시스템 콜 테이블에는 4개의 시스템 콜의 후킹 가능성이 있음을 알려주고
    있다. 이 중 427번은 volafox에서 항상 나타나는 오진이므로 제외(클린 시스템에서도 427번은 항상 후킹되었다고 나타난다. 운영체제
    자체에서 실제 심볼 주소와 다른 주소로 변경을 수행하는 것으로 예상된다.)하면, 총 3개의 시스템 콜이 후킹된 것이다. 후킹된 함수의 정보는
    '/usr/include/sys/syscall.h'의 시스템 콜 목록에서 확인할 수 있다.<br /><br /><br /><span>James-ui-MacBook-Pro:sys
    n0fate$ cat syscall.h | grep 222</span><br /><span>#define<span> </span><span><strong>SYS_getdirentriesattr</strong></span>
    222</span><br /><span>James-ui-MacBook-Pro:sys n0fate$ cat syscall.h | grep 344</span><br
    /><span>#define<span> </span><span><strong>SYS_getdirentries64</strong></span>
    344</span><br /><span>James-ui-MacBook-Pro:sys n0fate$ cat syscall.h | grep 397</span><br
    /><span>#define<span> </span><span><strong>SYS_write_nocancel</strong></span>
    397</span><br /><span>James-ui-MacBook-Pro:sys n0fate$</span><br /><br /><br />위
    두 시스템 콜은 Finder(파일 탐색기)에서 특정 파일을 숨기기 위해 후킹하는 함수이고, write_nocancel은 특정 프로세스의 명령
    수행 결과를 조작할 때 후킹하는 함수이다. <b>여기에서 우리는 다음을 예측할 수 있다.</b><br /><br /><ul><li>루트킷이
    특정한 파일을 Finder(파일 탐색기)에서 보이지 않도록 하고, 특정 명령에 대해 수행 결과를 조작할 것이다.</li><li>세 개의 후킹된
    함수의 포인터 주소가 상당히 근접해 있으므로 하나의 루트킷 드라이버에서 3개의 함수를 후킹했을 것이다.</li></ul><br />이제 어떤
    루트킷이 숨겨져 있는지 확인하기 위해 KEXT의 목록을 살펴보기로 한다.<br /><br /><h3>6. 루트킷의 KEXT 찾기</h3>일단
    루트킷을 찾기 위해 메모리에서 KEXT 목록을 추출한다.<br /><br /><br /><span>James-ui-MacBook-Pro:volafox
    n0fate$ python vol.py -i ~/Desktop/rubilyn_evidence.vmem -o kextstat</span><br
    /><span>[+] Kernel Extention List</span><br /><span>OFFSET(P)  INFO KID      
                                                KEXT_NAME    VERSION REFER_COUNT  
          REFER_LIST            ADDRESS     SIZE HDRSIZE          START_PTR STOP_PTR</span><br
    /><span>0x16B877C8    1  90                                 com.hackerfantastic.rubilyn
             1           0 0xFFFFFF8007324AC0 0xFFFFFF7F80791000    20480       0
    0xFFFFFF7F80792E6A 0xFFFFFF7F80792EB6</span><br /><span>0x310F9080    1  88  
                                       com.vmware.kext.vmhgfs 0079.97.03          
    0 0xFFFFFF800736B4C0 0xFFFFFF7F815EC000    40960       0 0xFFFFFF7F815F30D0 0xFFFFFF7F815F309C</span><br
    /><span>0x1F7D6060    1  87                                    com.vmware.kext.vmmemctl
    0079.97.03           0 0xFFFFFF80072C0780 0xFFFFFF7F815E7000    20480       0
    0xFFFFFF7F815EA60A 0xFFFFFF7F815EA656</span><br /><span>0x28A084B0    1  86  
                                 com.apple.filesystems.autofs        3.0          
    0 0xFFFFFF80072C5C00 0xFFFFFF7F815DE000    36864       0 0xFFFFFF7F815E4FE2 0xFFFFFF7F815E502E</span><br
    /><span>0x1F4CB020    1  85                                     com.apple.kext.triggers
           1.0           1 0xFFFFFF80072C2200 0xFFFFFF7F80CBF000    20480       0
    0xFFFFFF7F80CC1BF6 0xFFFFFF7F80CC1C42</span><br /><span>0x28165550    1  84  
                                   com.apple.driver.AudioAUUC       1.59          
    0 0xFFFFFF80072CF100 0xFFFFFF7F815D9000    20480       0 0xFFFFFF7F815DBF6C 0xFFFFFF7F815DBFB8</span><br
    /><span><b>... (80여개의 KEXT 생략) ...</b></span><br /><span>0x05EB7900    1   6  
                                        com.apple.kpi.private     11.4.0          28
            0x00000000         0x00000000        0       0         0x00000000 0x00000000</span><br
    /><span>0x05EB7A00    1   5                                          com.apple.kpi.mach
        11.4.0          68         0x00000000         0x00000000        0       0
            0x00000000 0x00000000</span><br /><span>0x05EB7B00    1   4          
                                com.apple.kpi.libkern     11.4.0          79      
      0x00000000         0x00000000        0       0         0x00000000 0x00000000</span><br
    /><span>0x05EB7C00    1   3                                         com.apple.kpi.iokit
        11.4.0          74         0x00000000         0x00000000        0       0
            0x00000000 0x00000000</span><br /><span>0x05EB7D00    1   2          
                                   com.apple.kpi.dsep     11.4.0           6      
      0x00000000         0x00000000        0       0         0x00000000 0x00000000</span><br
    /><s\npan>0x05EB7E00    1   1                                           com.apple.kpi.bsd
        11.4.0         "
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<h3>1. 소개</h3>
<p>nullsecurity에서 rubilyn이라는 루트킷을 공개하였다. 해당 루트킷은 바이너리 뿐만아니라 소스코드가 함께 있기 때문에, 새로운 Mac OS X 에 맞게 적절하게 리빌드하여 사용할 수 있다.</p>
<p>코드 : <a href="http://www.nullsecurity.net/backdoor.html">http://www.nullsecurity.net/backdoor.html</a></p>
<p>해당 코드의 description 내용은 다음과 같다.</p>
<blockquote><p><span>64bit Mac OS-X kernel rootkit that uses no hardcoded address to hook the BSD subsystem in all OS-X Lion & below. It uses a combination of syscall hooking and DKOM to hide activity on a host. String resolution of symbols no longer works on Mountain Lion as symtab is destroyed during load, this code is portable on all Lion & below but requires re-working for hooking under Mountain Lion.</span></p></blockquote>
<p>이 포스팅에선 rubilyn 루트킷이 설치된 시스템을 분석하고자 할때, 메모리 이미지 분석을 통해 루트킷을 효과적으로 파악하고 분석할 수 있는지에 대해 알아보겠다.</p>
<p>이 분석에서 필자가 원하는 부분과 그 대상은 다음과 같다.</p>
<ol>
<li>루트킷 기법에 대해 메모리 분석 툴의 효용성 입증 - 볼라폭스 개발팀 그리고 독자</li>
<li>현재 메모리 분석 도구의 한계점 도출 - 볼라폭스 개발팀</li>
<li>실제 상황에 존재할 수 있는 루트킷을 분석을 통해 메모리 분석의 중요성을 일깨워주기 - 이 글을 읽는 독자</li>
</ol>
<p>일단 설정한 상황을 미리 제시하고 각각의 은닉 및 변조 기법을 메모리에서 어떻게 탐지하는지를 확인하는 것이 좀 더 재밌을 것이라 생각되어, 셋팅한 내용을 다음과 같이 기록한다.<br />(<u>물론 아래의 분석 단계에선 셋팅한 내용을 모르는 상태에서 행동한 모든 내용을 알아내도록 기술할 것이다.</u>)</p>
<p>루트킷 설정은 다음과 같다.</p>
<ul>
<li>kextstat, netstat, grep, w, who 명령으로부터 자기자신을 은닉</li>
<li>PID 258 프로세스(bash)에 은닉 기법 적용</li>
<li>22번 포트로 접근하는 모든 세션 숨김</li>
<li>루트킷 관련 파일이 Finder(파일 탐색기)에서 보이지 않도록 함.</li>
</ul>
<p>루트킷이 설치된 시스템 환경은 다음과 같다.</p>
<ul>
<li>OS: Mac OS X Lion</li>
<li>Architecture : Intel x86_64bit</li>
</ul>
<h3>2. 라이브 정보 수집하기</h3>
<p>보통 사고 대응을 나가면 가장 먼저 해야할 일이 라이브 정보를 수집하는 일이다. 아무래도 네트워크 정보나 프로세스 정보는 실시간으로 변화가 일어나기 때문에, 간발의 차이로 루트킷에 대한 흔적을 찾지 못할 수 있기 때문이다.</p>
<p>우선 라이브 상태에서는 위의 4가지 정보가 완벽하게 숨겨져 있었다. 즉 스크립트 또는 명령어 입력으로 수집한 라이브 데이터는 루트킷을 찾아내는데 전혀 효용성이 없다고 볼 수 있다.</p>
<p>그렇다고해서, 라이브 정보 수집을 제외해야 할 것인가?하면 그건 또 아니다. 쓸모 없는 정보를 괜히 수집할 필요가 있을까 싶지만, 라이브 정보는 추 후 진행할 메모리 분석 결과와 비교하는데 큰 도움이 된다.</p>
<p>예를 들어, 시스템에 프로세스가 400개가 구동 중이고 이 중 하나의 프로세스만 악성코드인 상황에서 모든 프로세스를 찾아낼 수 있는 메모리 분석 도구로 메모리 분석을 진행한다면 이 400개를 모두 의심하는 상태에서 분석을 진행해야 할 것이다. 하지만, 변조된 결과를 받은 라이브 데이터와 메모리 분석 결과를 비교해보면 메모리 분석 결과에만 나타나는 프로세스를 발견할 수 있을 것이며, 이 프로세스는 루트킷과 관련된 프로세스일 가능성이 매우 높게 된다(물론 상황에 따라 메모리 이미징 시점에 새로운 프로세스가 생성될 수도 있긴하지만 다양한 방법으로 추가 생성된 프로세스는 판단할 수 있으므로, 이는 논외로 생각하자). 이러한 판단은 프로세스 뿐만 아니라 드라이버 및 네트워크 정보에서도 비교해볼 수 있기 때문에 가능하다면 라이브 정보는 꼭 수집하는 것을 권장한다.</p>
<p><b><i>* 단 이번 포스팅에서는 라이브 데이터와 일일히 비교하는 것을 포스팅하면 해당 데이터가 지면을 너무 많이 할애할 것이기 때문에, 메모리 분석 정보만으로 루트킷을 찾도록 하겠다.</i></b></p>
<h3>3. 메모리 이미징 수행</h3>
<p>Mac OS X 10.7.2 이상부터는 FW를 이용하여 메모리 이미징하는 행위를 방지한다고 알려져있다.(맥이 한대로 아직 확인을 못해봤지만, 내가아는 애플이라면 충분히 가능하다.) 그래서 최근에는 윈도우와 같이 소프트웨어 기반의 메모리 이미징을 추천하고 있다.</p>
<p>메모리 이미징의 가장 대표적인 도구는 MacMemoryReader로 커널 코어 덤프(풀 덤프)를 수행해주는 툴이다. 이 도구로 이미징한 <u>코어 덤프 파일은 volafox에 있는 flatten.py 를 이용하여 volafox에서 분석 가능한 포맷으로 변경</u>할 수 있다.</p>
<p>만약에 지금 상황과 같이 악성코드나 루트킷 샘플을 분석하는 것이라면, vmware의 메모리 이미지를 바로 복사해서 사용할 수 있다.vmware를 suspend 상태로 두고 해당 vmem 파일을 복사해오면 volafox에서 바로 분석이 가능하다. 본 분석에는 vmware의 메모리 이미지를 이용하여 분석을 진행하였다. 물론 코어 덤프를 이용해도 동일한 분석이 가능하다. :)</p>
<h3>4. 기본 정보 확인</h3>
<p>메모리 이미지를 가져오면 volafox에서 바로 분석을 진행할 수 있다.</p>
<p><span>James-ui-MacBook-Pro:volafox n0fate$ python vol.py</span><br /><span><br /></span><span>volafox: Mac OS X Memory Analysis Toolkit</span><br /><span>project: http://code.google.com/p/volafox</span><br /><span>support: 10.6-8; 32/64-bit kernel</span><br /><span>  input: *.vmem (VMWare memory file), *.mmr (Mac Memory Reader, flattened x86, IA-32e)</span><br /><span>  usage: python vol.py -i IMAGE [fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][-o COMMAND [-vp PID][-x PID][-x KEXT_ID][-x TASKID]]</span><br /><span><br /></span><span>Options:</span><br /><span>-o CMD            : Print kernel information for CMD (below)</span><br /><span>-p PID            : List open files for PID (where CMD is "lsof")</span><br /><span>-v                : Print all files, including unsupported types (where CMD is "lsof")</span><br /><span>-x PID/KID/TASKID : Dump process/task/kernel extension address space for PID/KID/Task ID (where CMD is "ps"/"kextstat"/"tasks")</span><br /><span><br /></span><span>COMMANDS:</span><br /><span>system_profiler : Kernel version, CPU, and memory spec, Boot/Sleep/Wakeup time</span><br /><span>mount           : Mounted filesystems</span><br /><span>kextstat        : KEXT (Kernel Extensions) listing</span><br /><span>ps              : Process listing</span><br /><span>tasks           : Task listing (& Matching Process List)</span><br /><span>systab          : Syscall table (Hooking Detection)</span><br /><span>mtt             : Mach trap table (Hooking Detection)</span><br /><span>netstat         : Network socket listing (Hash table)</span><br /><span>lsof            : Open files listing by process (research, osxmem@gmail.com)</span><br /><span>pestate         : Show Boot information (experiment)</span><br /><span>efiinfo         : EFI System Table, EFI Runtime Services(experiment)</span><br /><span>keychaindump    : Dump master key candidates for decrypting keychain(Lion, ML)</span><br /><span>James-ui-MacBook-Pro:volafox n0fate$ </span></p>
<p>보통은 시스템의 기본 정보를 확인하기 위해 system_profiler 명령을 우선 사<br />
용한다.</p>
<p><span>James-ui-MacBook-Pro:volafox n0fate$ python vol.py -i ~/Desktop/rubilyn_evidence.vmem -o system_profiler</span><br /><span>[+] Mac OS X Basic Information</span><br /><span> [-] Darwin kernel Build Number: <strong><span>11E53</span></strong></span><br /><span> [-] Darwin Kernel Major Version: 11</span><br /><span> [-] Darwin Kernel Minor Version: 4</span><br /><span> [-] Number of Physical CPUs: 1</span><br /><span> [-] Size of memory in bytes: 2147483648 bytes</span><br /><span> [-] Size of physical memory: 2147483648 bytes</span><br /><span> [-] Number of physical CPUs now available: 1</span><br /><span> [-] Max number of physical CPUs now possible: 1</span><br /><span> [-] Number of logical CPUs now available: 1</span><br /><span> [-] Max number of logical CPUs now possible: 1</span><br /><span> [-] Last Hibernated Sleep Time: Thu Jan 01 00:00:00 1970 (GMT +0)</span><br /><span> [-] Last Hibernated Wake Time: Thu Jan 01 00:00:00 1970 (GMT +0)</span><br /><span>James-ui-MacBook-Pro:volafox n0fate$ </span></p>
<p>다윈 커널 버전인 11.4는 Mac OS X Lion(11)을 의미한다. 이외에 메모리 용량 같은 부가적인 정보를 확인할 수 있다.</p>
<h3>5. 시스템 콜 분석</h3>
<p>필자가 보통 루트킷이 감염된 메모리 이미지를 분석할 때는 커널 레벨로 접근할 수 있는 방법을 제공하는 KEXT와 시스템 콜/Mach Trap 테이블 분석을 우선적으로 진행한다. 일단, 시스템 콜 목록과 Mach Trap 테이블을 확인하겠다. 전체 리스트를 출력하면 코드가 길어지므로 후킹으로 판단되는 콜 정보만 추출하겠다.</p>
<p><span>James-ui-MacBook-Pro:volafox n0fate$ python vol.py -i ~/Desktop/rubilyn_evidence.vmem -o mtt | grep HOOK</span><br /><span>NUM  ARG_COUNT                        CALL_NAME           CALL_PTR HOOK_FINDER</span><br /><span>James-ui-MacBook-Pro:volafox n0fate$ python vol.py -i ~/Desktop/rubilyn_evidence.vmem -o systab | grep HOOK</span><br /><span>NUM  ARG_COUNT RESV FLAGS                        CALL_PTR    ARG_MUNGE32_PTR ARG_MUNGE64_PTR RET_TYPE ARG_BYTES HOOK_FINDER</span><br /><span>222          8    0     0              0xFFFFFF7F8079241D 0xFFFFFF80005CEEA0      0x00000000        1        32 SYSCALL HOOKING</span><br /><span>344          4    0     0              0xFFFFFF7F807922EE 0xFFFFFF80005CEE60      0x00000000        6        16 SYSCALL HOOKING</span><br /><span>397          3    0     0              0xFFFFFF7F80792A7E 0xFFFFFF80005CEE50      0x00000000        6        12 SYSCALL HOOKING</span><br /><span>427          4    0     0              0xFFFFFF8000302E20 0xFFFFFF80005CEFD0      0x00000000        6        20 SYSCALL HOOKING</span><br /><span>James-ui-MacBook-Pro:volafox n0fate$ </span></p>
<p>분석 결과를 보면 mtt 커맨드인 Mach Trap Table에는 이상이 발견되지 않았지만, 시스템 콜 테이블에는 4개의 시스템 콜의 후킹 가능성이 있음을 알려주고 있다. 이 중 427번은 volafox에서 항상 나타나는 오진이므로 제외(클린 시스템에서도 427번은 항상 후킹되었다고 나타난다. 운영체제 자체에서 실제 심볼 주소와 다른 주소로 변경을 수행하는 것으로 예상된다.)하면, 총 3개의 시스템 콜이 후킹된 것이다. 후킹된 함수의 정보는 '/usr/include/sys/syscall.h'의 시스템 콜 목록에서 확인할 수 있다.</p>
<p><span>James-ui-MacBook-Pro:sys n0fate$ cat syscall.h | grep 222</span><br /><span>#define<span> </span><span><strong>SYS_getdirentriesattr</strong></span> 222</span><br /><span>James-ui-MacBook-Pro:sys n0fate$ cat syscall.h | grep 344</span><br /><span>#define<span> </span><span><strong>SYS_getdirentries64</strong></span> 344</span><br /><span>James-ui-MacBook-Pro:sys n0fate$ cat syscall.h | grep 397</span><br /><span>#define<span> </span><span><strong>SYS_write_nocancel</strong></span> 397</span><br /><span>James-ui-MacBook-Pro:sys n0fate$</span></p>
<p>위 두 시스템 콜은 Finder(파일 탐색기)에서 특정 파일을 숨기기 위해 후킹하는 함수이고, write_nocancel은 특정 프로세스의 명령 수행 결과를 조작할 때 후킹하는 함수이다. <b>여기에서 우리는 다음을 예측할 수 있다.</b></p>
<ul>
<li>루트킷이 특정한 파일을 Finder(파일 탐색기)에서 보이지 않도록 하고, 특정 명령에 대해 수행 결과를 조작할 것이다.</li>
<li>세 개의 후킹된 함수의 포인터 주소가 상당히 근접해 있으므로 하나의 루트킷 드라이버에서 3개의 함수를 후킹했을 것이다.</li>
</ul>
<p>이제 어떤 루트킷이 숨겨져 있는지 확인하기 위해 KEXT의 목록을 살펴보기로 한다.</p>
<h3>6. 루트킷의 KEXT 찾기</h3>
<p>일단 루트킷을 찾기 위해 메모리에서 KEXT 목록을 추출한다.</p>
<p><span>James-ui-MacBook-Pro:volafox n0fate$ python vol.py -i ~/Desktop/rubilyn_evidence.vmem -o kextstat</span><br /><span>[+] Kernel Extention List</span><br /><span>OFFSET(P)  INFO KID                                                   KEXT_NAME    VERSION REFER_COUNT         REFER_LIST            ADDRESS     SIZE HDRSIZE          START_PTR STOP_PTR</span><br /><span>0x16B877C8    1  90                                 com.hackerfantastic.rubilyn          1           0 0xFFFFFF8007324AC0 0xFFFFFF7F80791000    20480       0 0xFFFFFF7F80792E6A 0xFFFFFF7F80792EB6</span><br /><span>0x310F9080    1  88                                      com.vmware.kext.vmhgfs 0079.97.03           0 0xFFFFFF800736B4C0 0xFFFFFF7F815EC000    40960       0 0xFFFFFF7F815F30D0 0xFFFFFF7F815F309C</span><br /><span>0x1F7D6060    1  87                                    com.vmware.kext.vmmemctl 0079.97.03           0 0xFFFFFF80072C0780 0xFFFFFF7F815E7000    20480       0 0xFFFFFF7F815EA60A 0xFFFFFF7F815EA656</span><br /><span>0x28A084B0    1  86                                com.apple.filesystems.autofs        3.0           0 0xFFFFFF80072C5C00 0xFFFFFF7F815DE000    36864       0 0xFFFFFF7F815E4FE2 0xFFFFFF7F815E502E</span><br /><span>0x1F4CB020    1  85                                     com.apple.kext.triggers        1.0           1 0xFFFFFF80072C2200 0xFFFFFF7F80CBF000    20480       0 0xFFFFFF7F80CC1BF6 0xFFFFFF7F80CC1C42</span><br /><span>0x28165550    1  84                                  com.apple.driver.AudioAUUC       1.59           0 0xFFFFFF80072CF100 0xFFFFFF7F815D9000    20480       0 0xFFFFFF7F815DBF6C 0xFFFFFF7F815DBFB8</span><br /><span><b>... (80여개의 KEXT 생략) ...</b></span><br /><span>0x05EB7900    1   6                                       com.apple.kpi.private     11.4.0          28         0x00000000         0x00000000        0       0         0x00000000 0x00000000</span><br /><span>0x05EB7A00    1   5                                          com.apple.kpi.mach     11.4.0          68         0x00000000         0x00000000        0       0         0x00000000 0x00000000</span><br /><span>0x05EB7B00    1   4                                       com.apple.kpi.libkern     11.4.0          79         0x00000000         0x00000000        0       0         0x00000000 0x00000000</span><br /><span>0x05EB7C00    1   3                                         com.apple.kpi.iokit     11.4.0          74         0x00000000         0x00000000        0       0         0x00000000 0x00000000</span><br /><span>0x05EB7D00    1   2                                          com.apple.kpi.dsep     11.4.0           6         0x00000000         0x00000000        0       0         0x00000000 0x00000000</span><br /><s pan>0x05EB7E00    1   1                                           com.apple.kpi.bsd     11.4.0         [/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</s></p>
