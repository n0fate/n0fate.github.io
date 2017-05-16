---
layout: post
title: Check kernel state for OS X/BSD rootkit analysis
date: 2015-09-09 10:49:47.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- Memory Forensics
tags:
- Mac OS X
- memory forensics
- rootkit
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p id="os-x-/-bsd-커널-상태(kernel-state)-정보로-루트킷-식별하기">프로세스가 커널 모듈(e. 윈도는 디바이스 드라이버)와 통신할 때, IO Control / Kernel Event API를 이용하여 약속된 메시지를 전달/이벤트를 설정하고, 드라이버는 그 메시지를 핸들러에서 받아 그에 맞는 임무를 수행하는 구조를 가진다. 범용 운영체제로 알려진 시스템은 대부분 이러한 구조로 프로세스와 커널 모듈이 통신한다.<br />
<a href="#os-x-/-bsd-커널-상태(kernel-state)-정보로-루트킷-식별하기" name="os-x-/-bsd-커널-상태(kernel-state)-정보로-루트킷-식별하기"></a></p>
<p>기존 BSD는 커널 모듈에서 커널의 특정 값을 가져오고 싶거나 커널 설정을 변경하여 커널 내의 서비스 설정을 변경/참조하거나, 커널 튜닝을 할 때, 기존 모듈을 수정하여 그에 맞는 인터페이스를 만들어야하며, 커널 튜닝의 경우 커널을 리빌드해야 할 수도 있다. (물론 다른 방법이 있을 수도 있다.) 이는 운영체제의 확장성, 유연성, 가용성을 낮추는 문제로 이어진다. 이러한 문제를 해결하기 위해 BSD는 커널 상태를 관리하는 구조를 만들어서 사용자가 필요한 변수를 확인하여 해당 변수를 커널/커널 모듈이 참조할 수 있게 하였다.</p>
<p>커널 상태는 리눅스와 BSD에 있으며, 커널 상태가 변경되면, 각 커널 상태 변수와 맵핑된 핸들러로 제어를 넘겨 동적으로 변경된 사항을 처리할 수 있도록 구성되어 있다.</p>
<p>커널 상태 정보를 확인하거나 설정하기 위한 도구로 <code>sysctl(system control)</code>이 있다. 이 도구는 몇몇 유닉스 스타일 운영체제에서 인자 값을 동적으로 변경하고 가져오기 위한 인터페이스 역할을 한다. 여기에서는 OS X의 커널 상태 관리 구조를 알아보고 OS X 루트킷이 이를 어떻게 활용했으며, 메모리에서 추적하는 방법에 대해 알아본다.</p>
<h2 id="os-x에서의-커널-상태-관리-구조"><a href="#os-x에서의-커널-상태-관리-구조" name="os-x에서의-커널-상태-관리-구조"></a>OS X에서의 커널 상태 관리 구조</h2>
<p>커널 상태 정보에는 다양한 정보가 있을 수 있기 때문에 이 정보를 카테고리화 하여 관리한다. 예를 들어, 커널 관련된 상태 정보는 <code>kern</code> 카테고리에 변수로 저장한다.</p>
<pre><code>$ sysctl -a | grep "kern."
kern.ostype: Darwin
kern.osrelease: 14.5.0
kern.osrevision: 199506
kern.version: Darwin Kernel Version 14.5.0: Wed Jul 29 02:26:53 PDT 2015; root:xnu-2782.40.9~1/RELEASE_X86_64
kern.maxvnodes: 132096
kern.maxproc: 1064
kern.maxfiles: 12288
kern.argmax: 262144
kern.securelevel: 0
kern.hostname: Macbook
kern.hostid: 0
</code></pre>
<p>주 카테고리는 다음과 같이 나눠진다.</p>
<table>
<thead>
<tr>
<th>카테고리</th>
<th>설명</th>
</tr>
</thead>
<tbody>
<tr>
<td>kern</td>
<td>일반적인 커널 인자</td>
</tr>
<tr>
<td>vm</td>
<td>가상 메모리 옵션</td>
</tr>
<tr>
<td>fs</td>
<td>파일 시스템 옵션</td>
</tr>
<tr>
<td>machdep</td>
<td>기기 독립적인 설정</td>
</tr>
<tr>
<td>net</td>
<td>네트워크 관련 설정</td>
</tr>
<tr>
<td>debug</td>
<td>디버깅 설정</td>
</tr>
<tr>
<td>hw</td>
<td>하드웨어 정보(보통 읽기 전용)</td>
</tr>
<tr>
<td>user</td>
<td>사용자 프로그램에 영향을 미치는 인자</td>
</tr>
<tr>
<td>ddb</td>
<td>커널 디버거</td>
</tr>
</tbody>
</table>
<p>필요 시엔 서브 카테고리를 여러개 둘 수도 있다. 단순하게 생각하면 여러 자식을 가질 수 있는 트리 구조라고 생각하면 된다. 이 커널 상태 정보는 KEXT에 의해 추가될 수 있으며, 프로세스는 시스템 제어(system control, sysctl) API를 통하여 해당 값을 읽고 쓸 수 있다. KEXT에는 각 커널 상태 변경 값을 <code>SYSCTL_PROC</code> 매크로로 등록할 수 있다.</p>
<pre><code>SYSCTL_PROC(parent, nbr, name, access, ptr, arg, handler, fmt, descr);
</code></pre>
<p>각 커널 상태마다 접근권한과 MIB, 이름, 상태를 처리할 핸들러를 등록할 수 있다. 접근 권한은 “읽기 전용, 읽기/쓰기 가능, 시스템 튜닝에 의해 설정 가능”을 각각 R,W,L으로 표현한다. 예를 들어 하드웨어 정보는 시스템이 부팅되어 있는 동안에 변경되긴 힘들기 때문에 대부분 읽기 전용으로 설정된다.</p>
<p>MIB(Management Information Base)는 숫자로 커널 상태를 관리하는 코드라고 보면된다. 이 값은 <code>sysctl</code>에서는 나타나지 않으며, 내부적으로 관리 목적으로 활용된다. 보통 자동(OID_AUTO)으로 설정한다.</p>
<h2 id="os-x-루트킷의-커널-상태-활용"><a href="#os-x-루트킷의-커널-상태-활용" name="os-x-루트킷의-커널-상태-활용"></a>OS X 루트킷의 커널 상태 활용</h2>
<p>커널 상태에는 INT, FLOAT, STRING과 같은 다양한 정보를 저장할 수 있으므로, 프로세스가 KEXT에 정보를 전달할 수 있다. 루트킷도 이 방법으로 프로세스 명이나 사용자 명과 같은 정보를 전달할 수 있다. 예를 들어 특정 프로세스를 은닉하고 싶다면, 커널 상태에 프로세스의 ID만 설정하면 커널이 변경된 커널 상태를 확인하여 루트킷의 핸들러로 제어를 넘긴다다. 루트킷은 핸들러를 통해 커널 상태 변수를 확인하여 프로세스 ID에 해당하는 프로세스를 은닉하고 커널 상태 값을 초기화한다.</p>
<h2 id="라이브-분석의-한계"><a href="#라이브-분석의-한계" name="라이브-분석의-한계"></a>라이브 분석의 한계</h2>
<p>이 정보를 라이브로 분석하려면 <code>sysctl</code> 도구에 의존해야 한다. <code>sysctl</code>에서는 각 커널 상태 문자열(이름)과 설정 값을 보여준다. 즉, 라이브로 분석하려면 명령어 결과를 개별적으로 확인하여 의심가는 설정 정보를 확인해야 한다. 분석은 가능하나 효과적인 방법은 아니다. 또한 루트킷은 시스템 콜 후킹을 통해 명령어 결과 값을 간단히 변조할 수 있는 문제점도 가지고 있다.</p>
<h2 id="메모리-분석을-이용한-루트킷-탐지"><a href="#메모리-분석을-이용한-루트킷-탐지" name="메모리-분석을-이용한-루트킷-탐지"></a>메모리 분석을 이용한 루트킷 탐지</h2>
<p>메모리 포렌식에서는 라이브 포렌식보다 더 많은 정보를 획득할 수 있다. 커널 변수 중 <code>sysctl__children</code> 은 각 커널 상태 리스트인 <code>oid_list</code> 구조체의 시작 주소를 가리키고 있으므로 이를 통해 커널 상태 구조체인 <code>sytsctl_oid</code> 를 분석할 수 있다. <code>volafox</code>를 이용해 요세미티 10.3 메모리 이미지의 커널 상태 정보를 분석한 결과는 다음과 같다.</p>
<pre><code>$ python vol.py -i ~/Desktop/external/dumped.bin -o sysctl
Name                                                                   MIB PERMISSION                                                         Handler Value
sysctl.debug                                                           0.0        R-L                                    __kernel__(ffffff80045e0a70) None
sysctl.name2oid                                                        0.3        RWL                                    __kernel__(ffffff80045e0500) None
sysctl.proc_native                                                   0.101        R-L                                    __kernel__(ffffff80045dccd0) None
sysctl.proc_cputype                                                  0.102        R-L                                    __kernel__(ffffff80045dcc20) None
kern.ostype                                                            1.1        R-L                                    __kernel__(ffffff80045df370) Darwin
...
</code></pre>
<p>구조체를 분석하면 커널 상태 명, MIB, 권한 등의 정보 뿐만 아니라 커널 상태를 처리하는 핸들러의 주소를 획득할 수 있다. 이 핸들러와 KEXT 주소를 비교하면 어떤 KEXT가 커널 상태를 제어하는지 알 수 있다. KEXT 로 필터링하여 루트킷을 식별할 수 있는 것이다.</p>
<h2 id="케이스-스터디"><a href="#케이스-스터디" name="케이스-스터디"></a>케이스 스터디</h2>
<p>케이스는 이전 Virus Bulletin 2013에 투고한 논문의 샘플 루트킷인 rubilyn으로 하였다. 루트킷에 대한 자세한 정보는 <a href="https://forensic.n0fate.com/wp-content/uploads/2012/07/Hunting-Mac-OS-X-rootkit-with-Memory-Forensics.pdf">Hunting Mac OS X rootkit with memory forensics</a>를 참조하기 바란다. 이 루트킷은 디바이스에 명령을 내리고 명령에 필요한 정보를 커널 상태로 전달하도록 구성되어 있다. 예를 들어 KEXT의 권한 상승처리는 다음과 같다.</p>
<pre><code>static int getroot(int pid)
{
    struct proc *rootpid;
    kauth_cred_t creds;
    rootpid = proc_find(pid);
    if(!rootpid)
        return 0;
    lck_mtx_lock((lck_mtx_t*)&amp;rootpid-&gt;p_mlock);
    creds = rootpid-&gt;p_ucred;
    creds = my_kauth_cred_setuidgid(rootpid-&gt;p_ucred,0,0);
    rootpid-&gt;p_ucred = creds;
    lck_mtx_unlock((lck_mtx_t*)&amp;rootpid-&gt;p_mlock);
    return 0;
}
...[SNIP]..
/* prototypes for read/write handling functions for our sysctl nodes. */
static int sysctl_rubilyn_pid SYSCTL_HANDLER_ARGS;
...[SNIP]..
SYSCTL_PROC(_debug_rubilyn,OID_AUTO,pid,
            (CTLTYPE_INT|CTLFLAG_RW|CTLFLAG_ANYBODY),
            &amp;k_pid,0,sysctl_rubilyn_pid,"IU","");
...[SNIP]..
static int sysctl_rubilyn_pid SYSCTL_HANDLER_ARGS
{
    int ret = sysctl_handle_int(oidp, oidp-&gt;oid_arg1, oidp-&gt;oid_arg2, req);
    getroot(k_pid);
    k_pid = 0;
    return ret;
}
</code></pre>
<p>루트킷이 로드된 메모리를 덤프하여 <code>volafox</code>의 플러그인인 <code>sysctl</code>을 구동하면 다음과 같은 결과를 볼 수 있다.</p>
<pre><code>$ python vol.py -i ~/Desktop/rubilyn.mem -o sysctl
Name                                                          MIB PERMISSION                                                         Handler Value
sysctl.debug                                                  0.0        R-L                                    __kernel__(ffffff8000556650) None
sysctl.name2oid                                               0.3        RWL                                    __kernel__(ffffff8000556120) None
..[SNIP]..
debug.SerialATAPI                                           5.118        RW-             com.apple.iokit.IOAHCISerialATAPI(ffffff7f809eef0f) None
debug.USB                                                   5.119        RW-                   com.apple.iokit.IOUSBFamily(ffffff7f809fa7cb) None
debug.rubilyn.pid                                       5.120.101        RW-                   com.hackerfantastic.rubilyn(ffffff7f80bc70dd) 0
debug.rubilyn.pid2                                      5.120.102        RW-                   com.hackerfantastic.rubilyn(ffffff7f80bc714b) 0
debug.rubilyn.pid3                                      5.120.103        RW-                   com.hackerfantastic.rubilyn(ffffff7f80bc71ed) 0
debug.rubilyn.dir                                       5.120.104        RW-                   com.hackerfantastic.rubilyn(ffffff7f80bc72aa)
debug.rubilyn.cmd                                       5.120.105        RW-                   com.hackerfantastic.rubilyn(ffffff7f80bc72bb) /tmp
debug.rubilyn.user                                      5.120.106        RW-                   com.hackerfantastic.rubilyn(ffffff7f80bc72cc) victim
debug.rubilyn.port                                      5.120.107        RW-                   com.hackerfantastic.rubilyn(ffffff7f80bc72dd) 88
..[SNIP]..
</code></pre>
<p>앞서 설명한 것처럼 메모리를 분석하면 각 커널 상태의 핸들러를 등록한 KEXT와 어떤 함수가 처리하는지 확인할 수 있다. 예를 들어 <code>debug.rubilyn.pid</code>(핸들러 0xffffff7f80bc70dd) 핸들러를 분석한다면 다음과 같이 코드를 볼 수 있다. KEXT는 로드되는 베이스 주소가 매번 달라지기 때문에 메모리에서 추출해서 비교하는 것이 함수 식별이 좀 더 빠르다.<br />
또한 메모리에서 추출한 KEXT를 분석할 때는 KASLR을 고려해야하기 때문에 <code>volafox</code>에서 <code>dumpsym</code>으로 현재 KASLR에 맞는 심볼 정보를 추출한 후, IDAPython 스크립트인 <a href="https://github.com/n0fate/idapython">makecommsyscallref</a>로 변경된 커널 함수를 매핑할 수 있다.</p>
<pre><code>$ python vol.py -i ~/Desktop/rubilyn.mem -o kextstat | grep rubilyn
0x37E387C8    1  85                                 com.hackerfantastic.rubilyn                1           0 0xFFFFFF8004969BE0 0xFFFFFF7F80BC6000   20480       0 0xFFFFFF7F80BC7E6A 0xFFFFFF7F80BC7EB6
$ python vol.py -i ~/Desktop/rubilyn.mem -o kextstat -x 85
[+] Find KEXT: com.hackerfantastic.rubilyn, Virtual Address : 0xFFFFFF7F80BC6000, Size: 20480
[DUMP] FILENAME: com.hackerfantastic.rubilyn-ffffff7f80bc6000-ffffff7f80bcb000
[DUMP] Complete.

[IN IDA PRO]
__text:FFFFFF7F80BC70DD sub_FFFFFF7F80BC70DD proc near
__text:FFFFFF7F80BC70DD                 push    rbp
__text:FFFFFF7F80BC70DE                 mov     rbp, rsp
__text:FFFFFF7F80BC70E1                 push    r15
__text:FFFFFF7F80BC70E3                 push    r14
__text:FFFFFF7F80BC70E5                 push    rbx
__text:FFFFFF7F80BC70E6                 push    rax
__text:FFFFFF7F80BC70E7                 mov     edx, [rdi+20h]
__text:FFFFFF7F80BC70EA                 mov     rsi, [rdi+18h]
__text:FFFFFF7F80BC70EE                 call    near ptr 0FFFFFF8000555D10h ; sysctl_handle_int
__text:FFFFFF7F80BC70F3                 mov     ebx, eax
__text:FFFFFF7F80BC70F5                 mov     edi, cs:g_pid
__text:FFFFFF7F80BC70FB                 call    near ptr 0FFFFFF8000546DC0h ; proc_find
__text:FFFFFF7F80BC7100                 mov     r14, rax
__text:FFFFFF7F80BC7103                 test    r14, r14
__text:FFFFFF7F80BC7106                 jz      short loc_FFFFFF7F80BC7134
__text:FFFFFF7F80BC7108                 lea     r15, [r14+50h]
__text:FFFFFF7F80BC710C                 mov     rdi, r15
</code></pre>
<h2 id="결론"><a href="#결론" name="결론"></a>결론</h2>
<p>메모리 분석 시 커널 상태 정보를 확인(<code>sysctl</code> 플러그인)하면, 루트킷 분석 속도를 높일 수 있는 다양한 정보를 얻을 수 있다.<br />
여러 컨퍼런스에서 자주 언급하지만 메모리 포렌식은 고도의 악성코드/루트킷 분석에 소요되는 많은 시간을 절약해주는 훌륭한 기술이다. 메모리 분석의 특성상 분석을 통해 악성 행위임을 유추한 결과를 전달하다보니 분석가 자신도 커널에 대한 이해가 많이 필요하다. 그로인해 진입 장벽이 높아 시도하지 않는 분들이 종종 있다. 한번 익혀두면 업그레이드된 자기자신을 볼 수 있을 것이다.</p>
<h2 id="참조"><a href="#참조" name="참조"></a>참조</h2>
<p>[1] sysctl, wikipedia, <a href="https://en.wikipedia.org/wiki/Sysctl">https://en.wikipedia.org/wiki/Sysctl</a>.<br />
[2] sysctl(9) - The FreeBSD Project, <a href="https://www.freebsd.org/cgi/man.cgi?query=sysctl&amp;sektion=9">https://www.freebsd.org/cgi/man.cgi?query=sysctl&amp;sektion=9</a>.<br />
[3] BSD sysctl API, Kernel Programming Guide, <a href="http://docs.huihoo.com/darwin/kernel-programming-guide/boundaries/chapter_14_section_7.html">http://docs.huihoo.com/darwin/kernel-programming-guide/boundaries/chapter_14_section_7.html</a>.</p>
