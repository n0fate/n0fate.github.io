---
layout: post
title: DTrace based Rootkit Detection (System Call Hooking)
date: 2013-06-02 12:46:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- Memory Forensics
tags: []
author: "n0fate"
---
<p>DTrace는 Sun Microsystems에서 개발한 동적 트레이싱 프레임워크이다. [fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][<a href="http://en.wikipedia.org/wiki/DTrace" target="_blank">link</a>] DTrace는 동작 중인 시스템 환경에서 애플리케이션이나 커널의 트러블 슈팅을 위해 개발되었다. 이 프레임워크는 생산성을 높인 D 언어를 사용하며, 코드 몇 줄만으로도 시스템 정보 수집이나 프로세스를 디버깅할 수 있게해주는 강력한 기능을 제공한다. 개발자 입장에서 자신이 작성한 프로그램을 바이너리 레벨에서 디버깅하기에는 아주 훌륭한 프레임워크라 할 수 있지만 조금만 관점을 바꿔보면, 도구의 강력한 기능을 해킹 기술에도 사용할 수 있다.</p>
<p>Mac OS X도 DTrace 프레임워크를 제공하고 있다. 올해 4월, Immunity에서 주최하는 InfiltrateCon 2013에서는 DTrace를 이용하여 다양한 루트킷 기법을 스크립트 형태로 작성하는 방법을 소개하였다 [<a href="http://felinemenace.org/~nemo/dtrace-infiltrate.pdf" target="_blank">pdf</a>]. 본 포스팅에서는 이 중 시스템 콜 후킹을 수행하는 특정 디렉터리 내의 파일 엔트리를 은닉하는 방법과 이를 탐지하는 방법을 설명하고자 한다. 코드의 동작 순서를 살펴보면 다음과 같다.</p>
<div></div>
<div><a href="http://4.bp.blogspot.com/-IRYgxXz-NI4/Uaq9-2geAJI/AAAAAAAAA1A/IXeMyyeuoEk/s1600/posting_dtrace_rootkit.jpg"><img alt="" src="{{ site.baseurl }}/assets/posting_dtrace_rootkit.jpg" width="302" height="400" border="0" /></a></div>
<p>DTrace에서 제공하는 시스템 콜 후킹 기능을 사용하기 때문에, 메모리 분석도구인 volafox[<a href="http://code.google.com/p/volafox" target="_blank">tool</a>]로 탐지할 수 있으나, 실제 테스트 결과 기존 후킹 탐지 방법과는 약간 다른 결과를 보였다. 우선 메모리 분석 도구를 이용하여 시스템 콜 변조를 탐지하기 위해 해당 도구를 Mountain Lion이 설치된 VMware에서 실행하고, 메모리 스냅샷을 분석하였다.</p>
<pre class="lang:default theme:twilight">chainbreakers-MacBook-Pro:~ chainbreaker$ uname -a
Darwin chainbreakers-MacBook-Pro.local 12.0.0 Darwin Kernel Version 12.0.0: Sun Jun 24 23:00:16 PDT 2012; root:xnu-2050.7.9~1/RELEASE_X86_64 x86_64
</pre>
<p>커널 버전은 12.0.0인 Moutain Lion 초기버전이고 64비트 커널이 로드되어 있다.</p>
<pre class="lang:default theme:twilight">chainbreakers-MacBook-Pro:~ chainbreaker$ file dirhide.d
dirhide.d: a /usr/sbin/dtrace -s script text executable
</pre>
<p>스크립트 파일임을 확인하고 현재 디렉터리 상태를 확인했다.</p>
<pre class="lang:default theme:twilight">chainbreakers-MacBook-Pro:~ chainbreaker$ ls -al /private/tmp
total 0
drwxrwxrwt  9 root             wheel  306  6  2 11:44 .
drwxr-xr-x@ 6 root             wheel  204  3 18 15:05 ..
drwx------  3 chainbreaker     wheel  102  6  2 11:42 launch-2W8n1B
drwx------  3 chainbreaker     wheel  102  4 20 13:38 launch-YK6Mx4
drwx------  3 chainbreaker     wheel  102  4 20 13:38 launch-rDjEkQ
drwx------  3 _spotlight       wheel  102  4 20 13:12 launchd-129.TJC0L3
drwx------  3 chainbreaker     wheel  102  4 20 13:12 launchd-134.iboIMx
drwx------  3 _windowserver    wheel  102  4 20 13:38 launchd-191.YPTPFj
drwx------  3 _softwareupdate  wheel  102  6  2 11:44 launchd-473.dEMigI
</pre>
<p>dtrace의 로그를 생성하기 위해 -w 옵션을 주어서 스크립트를 실행한다.</p>
<pre class="lang:default theme:twilight">chainbreakers-MacBook-Pro:~ chainbreaker$ sudo dtrace -w -s dirhide.d
</pre>
<p>스크립트를 실행하고 난 후, 다시 'ls' 명령을 실행하였다.</p>
<pre class="lang:default theme:twilight">chainbreakers-MacBook-Pro:~ chainbreaker$ ls -al /private/tmp/
total 0
drwxrwxrwt  9 root             wheel  306  6  2 11:44 .
drwxr-xr-x@ 6 root             wheel  204  3 18 15:05 ..
drwx------  3 chainbreaker     wheel  102  4 20 13:38 launch-YK6Mx4
drwx------  3 chainbreaker     wheel  102  4 20 13:38 launch-rDjEkQ
drwx------  3 _spotlight       wheel  102  4 20 13:12 launchd-129.TJC0L3
drwx------  3 chainbreaker     wheel  102  4 20 13:12 launchd-134.iboIMx
drwx------  3 _windowserver    wheel  102  4 20 13:38 launchd-191.YPTPFj
drwx------  3 _softwareupdate  wheel  102  6  2 11:44 launchd-473.dEMigI
chainbreakers-MacBook-Pro:~ chainbreaker$
</pre>
<p>확인 결과 3번째 엔트리인 'launch-2W8n1B' 가 숨겨진 걸 확인할 수 있다.</p>
<p>이제 현재 메모리 상태를 스냅샷(snapshot)을 volafox로 분석하였다. 도구는 SVN Repository의 최신 버전을 이용하였다. [<a href="https://code.google.com/p/volafox/source/detail?r=102" target="_blank">Revision 102</a>]</p>
<pre class="lang:default theme:twilight">n0fate@n0fate-ui-MacBook-Pro: ~/volafox$ python vol.py -i ~/Documents/Parallels/Mac OS X.pvm/{8ffc8dfd-9a5b-481a-b88d-5653bee07b6b}.mem -o system_profiler
[+] Mac OS X Basic Information
 [-] Darwin kernel Build Number: 12A269
 [-] Darwin Kernel Major Version: 12
 [-] Darwin Kernel Minor Version: 0
 [-] Number of Physical CPUs: 1
 [-] Size of memory in bytes: 2147483648 bytes
 [-] Size of physical memory: 2147483648 bytes
 [-] Number of physical CPUs now available: 1
 [-] Max number of physical CPUs now possible: 1
 [-] Number of logical CPUs now available: 1
 [-] Max number of logical CPUs now possible: 1
 [-] Last Hibernated Sleep Time: Thu Jan 01 00:00:00 1970 (GMT +0)
 [-] Last Hibernated Wake Time: Thu Jan 01 00:00:00 1970 (GMT +0)
</pre>
<p>버전 정보를 확인하고 systab 명령어로 시스템 콜 테이블 정보 중 후킹된 엔트리를 확인한다.</p>
<pre class="lang:default theme:twilight">n0fate@n0fate-ui-MacBook-Pro: ~/volafox$ python vol.py -i ~/Documents/Parallels/Mac OS X.pvm/{8ffc8dfd-9a5b-481a-b88d-5653bee07b6b}.mem -o systab | grep hooked
427         4    0     0              0xFFFFFF8000304CA0 0xFFFFFF8000304CA0 0xFFFFFF80005E3640      0x00000000        6        20 Maybe hooked
n0fate@n0fate-ui-MacBook-Pro: ~/volafox$
</pre>
<p>코드를 확인해보면, volafox에서 버그로 나타나는 427번 시스템 콜을 제외하곤 후킹을 판단되지 않는다.</p>
<p>DTrace로 인해 후킹된 시스템 콜은 해당 스크립트의 메모리 영역을 점프하는 것이 아니라 커널의 'dtrace_systrace_syscall' 함수로 점프한 후, 사용자가 만든 trace 코드를 실행한다. [<a href="http://www.opensource.apple.com/source/xnu/xnu-1228.5.20/bsd/dev/dtrace/systrace.c" target="_blank">ref</a>] OS가 지원하는 정상적인 호출 흐름이기 때문에, 메모리 분석 도구에서 악성 후킹으로 판단하지 않는다. 이러한 후킹을 판단하려면, grep 인자로 'dtrace'를 주어야 한다.</p>
<pre class="lang:default theme:twilight">n0fate@n0fate-ui-MacBook-Pro: ~/volafox$ python vol.py -i ~/Documents/Parallels/Mac OS X.pvm/{8ffc8dfd-9a5b-481a-b88d-5653bee07b6b}.mem -o systab | grep dtrace
344         4    0     0        _dtrace_systrace_syscall 0xFFFFFF80005DC5E0 0xFFFFFF80005E34B0      0x00000000        6        16 True
n0fate@n0fate-ui-MacBook-Pro: ~/volafox$
</pre>
<p>344번 시스템 콜인 'getdirentries64' 함수가 변경된 것을 알 수 있다. 스크립트 실행 정보는 'ps' 명령어로 확인할 수 있다.</p>
<pre class="lang:default theme:twilight">n0fate@n0fate-ui-MacBook-Pro: ~/volafox$ python vol.py -i ~/Documents/Parallels/Mac OS X.pvm/{8ffc8dfd-9a5b-481a-b88d-5653bee07b6b}.mem -o ps
[+] Process List
OFFSET(P)  PID PPID PRIORITY NICE     PROCESS_NAME        USERNAME(UID,GID)  CRED(UID,GID)      CREATE_TIME (UTC+0)
0x008DA2E0   0    0        0    0      kernel_task                    (0,0)          (0,0) Sat Apr 20 04:10:56 2013
0x020B9A60   1    0      128    0          launchd     _softwareupdate(0,0)          (0,0) Sat Apr 20 04:10:56 2013
0x020B91A0  11    1      128    0   UserEventAgent                root(0,0)          (0,0) Sat Apr 20 04:11:00 2013
0x020BA8E0  12    1      128    0            kextd                root(0,0)          (0,0) Sat Apr 20 04:11:00 2013
...[SNIP]...
0x3F471760 464    1      128    0         installd     _softwareupdate(0,0)          (0,0) Sun Jun 02 02:43:25 2013
0x1D9D3BC0 473    1      128    0          launchd _softwareupdate(200,200)      (200,200) Sun Jun 02 02:44:24 2013
0x363258C0 475  473      128    0         cfprefsd _softwareupdate(200,200)      (200,200) Sun Jun 02 02:44:24 2013
0x01C31D40 492  448      128    0             sudo       chainbreaker(0,20)         (0,20) Sun Jun 02 02:45:41 2013
0x4026FA60 494  492      128    0           dtrace        chainbreaker(0,0)          (0,0) Sun Jun 02 02:45:45 2013
0x3F8F0480 495    1      128    0 coresymbolicatio     _softwareupdate(0,0)          (0,0) Sun Jun 02 02:45:47 2013
0x446E8300 497    1      128    0             sshd        chainbreaker(0,0)          (0,0) Sun Jun 02 02:46:23 2013
0x43CA7D40 500  497      128    0             sshd     chainbreaker(501,20)       (501,20) Sun Jun 02 02:46:27 2013
0x3ED72EA0 501  500      128    0             bash     chainbreaker(501,20)       (501,20) Sun Jun 02 02:46:27 2013
0x35664180 507  464      128    0 update_dyld_shar     _softwareupdate(0,0)          (0,0) Sun Jun 02 02:51:57 2013
</pre>
<p>그간 루트킷은 KEXT 기반으로 동작하였기 때문에, 메모리 분석 도구의 후킹 탐지 로직으로 함수 테이블 변조 여부를 확인할 수 있었다. dtrace 기반 루트킷은 기존 KEXT 기반의 루트킷과 다르게 OS에서 제공하는 후킹이기 때문에 기존 메모리 분석 도구의 후킹 탐지 로직을 우회할 수 있다.</p>
<p>KEXT 기반의 루트킷은 커널 메모리를 비공식적으로 조작하는 방법을 사용하기 때문에, 조금의 실수가 발생하더라도 커널 패닉을 유발할 수 있는 문제점을 가지고 있다. 또한, 많은 포렌식 조사관이 'kextstat' 명령을 이용하여 루트킷을 식별하기 때문에 탐지 위험이 높았다. dtrace를 이용한 트릭은 코드 작성이 간단하면서 손쉽게 함수 테이블을 후킹할 수 있는 장점을 가지고 있다.</p>
<p>이러한 상황을 고려하여 앞으로 Mac OS X 시스템을 분석할 경우에는 dtrace를 고려한 메모리 분석을 진행해야 할 것이다.</p>
