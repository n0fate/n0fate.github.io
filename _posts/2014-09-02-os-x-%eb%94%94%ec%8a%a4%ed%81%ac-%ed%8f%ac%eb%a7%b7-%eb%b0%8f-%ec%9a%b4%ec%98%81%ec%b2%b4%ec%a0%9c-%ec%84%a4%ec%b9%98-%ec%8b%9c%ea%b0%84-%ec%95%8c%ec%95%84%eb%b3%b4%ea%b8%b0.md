---
layout: post
title: HFS+ 파일 시스템 포맷 및 OS X 설치 시간 알아보기
date: 2014-09-02 18:33:51.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags:
- HFS Format Date
- OS Installation Date
author: "n0fate"
---
<p>사용자가 고의로 OS X 파일 시스템인 HFS+ 디스크를 포맷하거나, OS X를 설치한 시간을 알아봐야할 경우가 있다. 두 이벤트는 동시에 일어나거나 별도로 일어날 수 있는데 다음과 같은 행동을 했을 경우로 볼 수 있다.</p>
<ul>
<li>기존 포맷된 시스템에 운영체제 재설치 : 사용자가 디스크를 HFS+로 포맷과 운영체제 설치 프로세스를 별도로 진행하는 경우이다. 예를 들어 사용자가 디스크를 교체하기 위해 외장 디스크를 HFS+로 포맷하고 디스크를 교체한 후 운영체제를 설치할 수 있다. 이러한 경우에는 디스크 포맷 시점과 운영체제 설치 시점에 차이가 발생할 수 있다.</li>
<li>기존 시스템 또는 새로운 맥에 운영체제 설치 : 디스크가 파티셔닝이 되어있지 않거나 기존에 사용하던 맥에 새로운 운영체제를 설치할 경우에는 운영체제 설치 전에 디스크 관리자(Disk Utility)를 이용하여 포맷한 후 설치가 이루어질 수 있다. 이런 경우에는 운영체제 포맷시간과 설치 시간이 거의 동일하게 된다.</li>
</ul>
<p>이에 두 가지 상황을 분리하여 분석한다면, 사용자가 별도로 진행했는지 하나의 프로세스로 진행했는지 파악할 수 있다.</p>
<h2>디스크 포맷 시간</h2>
<p>디스크 포맷 시간을 판단하기 가장 좋은 요소는 HFS+ 파일 시스템의 볼륨 헤더에 있다. 볼륨 헤더는 파일 시스템 구성 시점에 파일 시스템 전체를 관리하기 위한 메타데이터를 저장하고 있는 공간으로 이 중, dateCreated 라는 요소는 볼륨의 생성 시간을 담당한다. 우선 볼륨을 확인하면 다음과 같다.</p>
<pre class="lang:sh decode:true">$ diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *512.1 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:          Apple_CoreStorage                         511.3 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS Mac OS X               *510.9 GB   disk1</pre>
<p>본 시스템은 FileVault2로 암호화되어 있기 때문에 실제 HFS+ 볼륨 정보는 /dev/disk1에 저장되어 있다. 해당 정보를 확인하면 다음과 같다.</p>
<pre class="lang:sh decode:true">/dev$ sudo xxd /dev/rdisk1 | more
...
0000400: 482b 0004 8000 2000 4846 534a 0000 0ee6  H+.... .HFSJ....
0000410: ce1b 1f3a d02b 3c6d 0000 0000 ce1b 81aa  ...:.+</pre>
<p>타임스탬프 정보는 0x400을 기준으로 0x10에 위치하는 0xce1b1f3a(빅엔디안)가 된다.</p>
<pre class="lang:sh decode:true">$ python
Python 2.7.5 (default, Mar  9 2014, 22:15:05) [GCC 4.2.1 Compatible Apple LLVM 5.0 (clang-500.0.68)] on darwinType "help", "copyright", "credits" or "license" for more information.
&gt;&gt;&gt; import time
&gt;&gt;&gt; time.gmtime(0xce1b1f3a) // Create Time
time.struct_time(tm_year=2079, tm_mon=7, tm_mday=29, tm_hour=19, tm_min=19, tm_sec=22, tm_wday=5, tm_yday=210, tm_isdst=0) // 2013년 7월 30일 4시 19일 22초
&gt;&gt;&gt; time.gmtime(0xd02b3c6d) // Modified Time
time.struct_time(tm_year=2080, tm_mon=9, tm_mday=2, tm_hour=9, tm_min=20, tm_sec=45, tm_wday=0, tm_yday=246, tm_isdst=0) // 2014년 9월 2일 오후 6시 20분 45초
&gt;&gt;&gt; time.gmtime(0xce1b81aa) // Disk Checking Time
time.struct_time(tm_year=2079, tm_mon=7, tm_mday=30, tm_hour=2, tm_min=19, tm_sec=22, tm_wday=6, tm_yday=211, tm_isdst=0) // 2013년 7월 30일 11시 19분 22초
&gt;&gt;&gt;</pre>
<p>HFS Plus Dates가 1904년 1월 1일 00시 기준이므로 유닉스 타임인 1970년 1월1일 00시와의 차이를 구하면 결과를 알 수 있다.</p>
<p>&gt;&gt;&gt; 2013년 7월 29일 19시 19분 22초 UTC+0</p>
<p>사용자가 포맷을 수행하면, 디스크의 파티션 테이블을 재정의하고, 각 파티션의 파일 시스템을 초기화하는 작업을 수행하는데, 이 과정에서 HFS+ 파일 시스템의 볼륨이 새롭게 생성되므로, 볼륨 헤더의 dateCreated 요소는 사용자가 시스템을 포맷한 시간으로 볼 수 있다.<br />
보통 안티포렌식에서 파일 시스템의 필드 정보를 수정하는 것은 리스크가 큰 작업인 만큼 많이 수행하지 않으므로, 이 과정으로 추출한 결과는 높은 신뢰성을 보장할 수 있다.</p>
<h2>운영체제 설치 시간</h2>
<p>운영체제 설치 시간을 확인할 수 있는 요소는 크게 2개가 있다. 첫 번째는 “/private/var/db/.AppleSetupDone”이라는 파일로 이 파일은 운영체제 설치 과정에서 Setup Assistant 프로세스 즉, 운영체제에 파일을 옮긴 후, 사용자 설정 시점에 생성된다. 즉, 파일의 생성 시간을 통해 운영체제 설치 시점을 알 수 있다. 운영체제 업그레이드 시에도 파일을 새롭게 생성하므로, 최초 운영체제 설치 및 업그레이드 시점을 파악할 수 있다.</p>
<pre class="lang:sh decode:true">/var/db$ stat -x .AppleSetupDone
   File: ".AppleSetupDone"  Size: 0
   FileType: Regular File  Mode: (0204/--w----r--)
   Uid: (    0/    root)  Gid: (    0/   wheel)Device: 1,4   Inode: 2064663    Links: 1
Access: Thu Jul 10 13:30:37 2014
Modify: Thu Oct 24 16:25:37 2013
Change: Thu Oct 24 16:25:37 2013</pre>
<p>두 번째 요소는 “/private/var/log/install.log” 파일로 이 파일의 시간 정보를 통해 시스템의 설치 시간을 파악할 수 있다. 단, OS X가 업그레이드 된 경우에는 이전 기록이 삭제되기 때문에 운영체제 클린 설치 시점보다는 운영체제 최종 버전 설치 및 업그레이드 시점을 파악하는데 사용하는 것이 좋다.</p>
<pre class="lang:sh decode:true ">/var/log$ cat install.log | more
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: @(#)PROGRAM:Install  PROJECT:Install-735
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: @(#)PROGRAM:IA  PROJECT:InstallAssistant-467.1
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Hardware: MacBookPro8,2 @ 2.20 GHz (x 8), 8192 MB RAM
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Running OS Build: Mac OS X 10.8.4 (12E55)
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: PATH=/usr/bin:/bin:/usr/sbin:/sbin
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: TMPDIR=/var/folders/zr/strlzp7s6mb_f8396y2mrmz40000gn/T/
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: SHELL=/bin/bash
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: HOME=/Users/chainbreaker
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: USER=chainbreakerOct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: LOGNAME=chainbreaker
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: SSH_AUTH_SOCK=/tmp/launch-Z5cK4j/Listeners
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: Apple_Ubiquity_Message=/tmp/launch-SkFla1/Apple_Ubiquity_Message
Oct 24 15:57:20 testmachine.local Install OS X Mavericks[11897]: Env: Apple_PubSub_Socket_Render=/tmp/launch-WwE5ia/Render</pre>
<p>로그 파일에는 운영체제 설치 과정이 저장되어 있으므로 최초 설치 시점을 파악할 수 있다. 단, 위에 설명한대로 운영체제를 업그레이드하는 경우에 새로운 파일을 만드는 것으로 확인된만큼 다른 여러 정보와 조합하여 분석해야할 것이다.</p>
