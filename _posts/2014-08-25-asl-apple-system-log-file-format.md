---
layout: post
title: ASL (Apple System Log) File Format
date: 2014-08-25 16:58:32.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags:
- ASL
- FileFormat
author: "n0fate"
---
<h2>소개</h2>
<p>ASL은 Mac OS X 10.4 이상에 있는 바이너리 형태의 시스템 로그이다. ASL에는 다음과 같은 정보가 저장된다.</p>
<ul>
<li>방화벽</li>
<li>로그인 정보</li>
<li>프로그램 에러</li>
<li>네트워크 정보</li>
<li>설치 정보</li>
<li>시스템 부팅/재부팅/종료</li>
<li>권한 상승 정보</li>
</ul>
<h2></h2>
<h2>위치</h2>
<p>ASL은 “/var/log/asl” 디렉터리에 저장되어 있다. 쓰기는 루트 권한만 가지고 있으며, 파일을 읽는 것은 사용자도 가능하다. 파일명에는 특별한 규칙이 있는데 다음과 같다.</p>
<ul>
<li>현재 로깅 데이터 : ‘YYYY.MM.DD.[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][UID]/[GID].asl’ 형태로 UID 및 GID 별로 분류하여 로깅한다.</li>
<li>백업 데이터 : ‘BB.YYYY.MM.DD.[GID].asl’ 형식을 가지며, 보통 매달 말일 날짜로 지정된다. 한달에 한번 생성된다.</li>
<li>에러 메시지 : ‘AUX.XXXX.MM.DD’ 형식을 가지며, 에러 발생 당시 스택 상태 등 에러 프로세스의 주요 정보를 저장한다.</li>
</ul>
<h2></h2>
<h2>파일 포맷</h2>
<p>ASL 포맷은 파일 헤더와 로그 레코드 셋으로 이루어져 있다. 헤더의 구조는 다음과 같다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/1408948601_full.png" target="_blank"><img id="blogsy-1408960372105.0828" class="aligncenter full" src="{{ site.baseurl }}/assets/1408948601_thumb.png" alt="" /></a></p>
<p>&nbsp;</p>
<p>헤더에는 ASL DB 시그너처와, 버전 정보, 첫 번째 레코드의 위치, 시간 정보, 마지막 레코드의 위치 등의 정보를 가진다. 헤더의 모든 데이터는 빅엔디안으로 저장된다. 실제 데이터베이스 파일로 비교해보면 다음과 같다.</p>
<p>&nbsp;</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/1408949187_full.png" target="_blank"><img id="blogsy-1408960372117.759" class="full aligncenter" src="{{ site.baseurl }}/assets/1408949187_thumb.png" alt="" width="640" height="116" /></a></p>
<p>버전 : 2 (OS X SL부터 버전2이다)<br />
첫 번째 레코드 위치 : 0xD1<br />
타임스탬프 : 0x53F29B55 -&gt; 2014.08.19 00시 33분 25초 (UTC+0)<br />
문자열 캐시 크기 = 0x100<br />
마지막 레코드 위치 = 0x01D674</p>
<p>첫 번째 레코드 위치와 마지막 레코드 위치를 알았으므로, 파일 시작에서 0xD1로 추적하여 정보를 추출한다. 레코드는 첫 번째 2바이트에 0x00이 저장되고 뒤에 4바이트에 레코드 길이를 저장한다. 마지막으로 그 다음 8바이트에 다음 레코드의 위치가 저장된다. 앞에 2바이트 null 값을 제외하고 재구성하면 다음과 같은 구조체를 가지고 있다.</p>
<p>&nbsp;</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/1408950259_full.png" target="_blank"><img id="blogsy-1408960372105.9136" class="aligncenter full" src="{{ site.baseurl }}/assets/1408950259_thumb.png" alt="" /></a></p>
<p>&nbsp;</p>
<p>ASL의 레코드에는 레코드 고유번호(ID), 레코드 기록 시간(초 + 나노초/1000000000), 메시지의 등급(level), 메시지 기록 프로세스, 권한(uid, gid, ruid, rgid), 키-값 셋(kv_count) 등 레코드 기록에 필요한 모든 정보가 저장되어 있다. kv_count 같은 경우에는 해당 값의 절반 갯수만큼의 키-값 쌍이 레코드 구조체 뒤에 각각 16바이트(키: 8바이트, 값: 8바이트)로 연결되어 있다. 메시지 등급은 8개로 나누어져 있으며 각각 ‘Emergency, Alert, Critical, Error, Warning, Notice, Info, Debug'로 0~7까지 값을 가진다.<br />
맨 뒤에 Reference라는 명칭이 들어간 변수는 변수 값의 맨 첫 번째 비트가 ‘1’이라면 다음 바이트에 문자열을 저장하고 ‘0’이라면 그 뒤에 7바이트를 주소(포인터) 값으로 하여 실제 문자열을 추적해야한다. 추적한 데이터의 구조는 다음과 같다.</p>
<p>&nbsp;</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/1408952221_full.png" target="_blank"><img id="blogsy-1408960372093.0261" class="aligncenter" src="{{ site.baseurl }}/assets/1408952221_thumb.png" alt="" width="649" height="106" /></a></p>
<p>&nbsp;</p>
<p>0x0001 이라는 바이트로 올바르게 추적했는지 확인한 후, 뒤에 있는 4바이트로 문자열의 길이를 구한다. 길이는 문자열에 끝인 0x00을 포함한 길이이다.</p>
<p>예제의 데이터베이스를 해석하면 다음과 같다.</p>
<p>&nbsp;</p>
<p><img id="blogsy-1408960372105.9294" class="aligncenter" src="{{ site.baseurl }}/assets/1408952005_thumb.png" alt="" width="625" height="275" /></p>
<p>&nbsp;</p>
<p>레코드 길이 : 0x84<br />
다음 레코드 오프셋 : 0x1CC<br />
ID : 0x0DACDC -&gt; 896220<br />
timestamp (seconds) : 0x53F29B52 -&gt; 2014년 08월 19일 00시 33분 22초(UTC+0)<br />
timestamp (nano) : 0x0ACBDAE0 -&gt; 0<br />
메시지 등급 : 0x05 -&gt; Notice<br />
flags : 0x02<br />
pid : 0x48 -&gt; 80<br />
uid : 0x00<br />
gid : 0x00<br />
ruid : 0xFFFFFFFF -&gt; -1<br />
rgid : 0x50 -&gt; 80<br />
reference pid : 0x00<br />
kv_count : 0x02 / 2 -&gt; 1개의 키-값 쌍 존재(메시지에 대한 추가 정보 기록)<br />
host reference : 0x77 -&gt; 0x0001(unknown), 0x21(길이 4바이트), 문자열(MultitouchHID: device bootloaded)<br />
sender reference : ‘hidd’ -&gt; 첫번째 바이트가 8(1000b)이므로 바로 뒤의 문자열을 해석<br />
facility reference : ‘user’ -&gt; 첫번째 바이트가 8(1000b)이므로 바로 뒤의 문자열을 해석<br />
message reference : 0x50 -&gt; 0x01(unknown), 0x12(길이 4바이트), 문자열(testmachine.local)<br />
referenced process reference : 0x00<br />
session reference : 0x00<br />
Key : 0x8F -&gt; Sender_Mach_UUID, Value : 0xA6 -&gt; 9D77ED51-7B0F-3C18-9783-34AD5B298243</p>
<p>이러한 방법으로 단일 연결리스트를 추적하면서 레코드를 해석하면 ASL을 분석할 수 있다.</p>
<h2>분석</h2>
<p>앞서 설명한 것과 같이 ASL은 바이너리 형태이기 때문에 포맷을 해석해서 보여줘야 한다. 파일 포맷을 분석해주는 도구는 크게 2개가 있으며, 하나는 OS X에 내장된 시스템 로그 분석 도구인 ‘syslog’이고, 다른 하나는 CCL Forensics에서 개발한 파이썬 기반 오픈소스 도구인 ccl-asl이다.</p>
<h3>syslog</h3>
<p>애플 시스템 로그 도구로 OS X에 기본적으로 내장된 프로그램이다. 로그 분석 도구인 만큼 상당히 많은 기능을 제공하며, 다른 도구를 통해 추가적인 분석이 가능하도록 다양한 출력 타입을 지원한다. 기본적인 사용 방법은 -f 옵션으로 로그 파일을 지정해주면 된다.</p>
<pre class="lang:sh decode:true ">$ syslog -f BB.2015.07.31.G80.asl Jun 30 16:53:27 172.20.nate.com login[19780] : USER_PROCESS: 19780 ttys001Jul 10 10:27:35 testmachine.local login[31670] : USER_PROCESS: 31670 ttys002Jul 10 10:28:00 testmachine.local login[31670] : DEAD_PROCESS: 31670 ttys002Jul 16 13:07:41 testmachine.local login[19780] : DEAD_PROCESS: 19780 ttys001</pre>
<p>옵션 설정을 통해 이벤트 유형별 필터링을 한다거나, 정규 표현식으로 특정 로그 정보(특정 프로세스, 특정 시간, 특정 이벤트)만 뽑아낼 수 있다. 출력 타입은 파일 포맷만 해석해 보여주는 raw, 표준 포맷 형태로 재구성해주는 standard, BSD 포맷 형태인 bsd, XML 형태인 xml을 지원한다. 스플렁크와 같은 도구에는 standard로 추출하여 밀어넣으면 타임라인 분석에 도움이 될 것이다. 참고로 별다른 옵션을 주지 않으면 standard로 출력한다.</p>
<h3>ccl-asl</h3>
<p>파이선 기반의 오픈소스 도구로 ASL 파일을 해석하여 결과를 보여준다. 간단하게 출력할 위치랑 입력 파일만 지정해주면 파일을 분석하여 tsv 파일로 생성한다.</p>
<pre class="lang:sh decode:true ">$ python ccl_asldb.py   -o ~/rapfer-ccl-asl/report.txt ~/2014.08.19.G80.asl 2014-08-25T16:25:39.000498 Processing: /Users/fate/2014.08.19.G80.asl$ cat report.txt Timestamp Host Sender PID Reference Process Reference PID Facility Level Message Other details2014-08-19T00:33:22 testmachine.local hidd 72 0 user Notice MultitouchHID: device bootloaded Sender_Mach_UUID='9D77ED51-7B0F-3C18-9783-34AD5B298243'2014-08-19T00:33:23 testmachine.local loginwindow 67 auth Notice magsafeStateChanged state changed old 2 new 1 Sender_Mach_UUID='D4C17042-701C-3BB2-B24B-807086205F53'2014-08-19T00:33:26 testmachine.local WindowServer 98 user Error _CGXHWCaptureWindowList: No capable active display found. Sender_Mach_UUID='61FE4BA9-1E5D-37F6-9126-24ACD7C7559D'</pre>
<p>운영체제를 가리지 않으므로, 디스크 이미지에서 타임라인 생성 시에 유용하게 사용할 수 있다.</p>
<h2>분석 시 유용한 키워드</h2>
<p>로그 데이터를 분석할 때는 어떤 키워드로 필터링할지가 가장 중요하다. ASL의 경우에는 다음과 같은 키워드가 중요하게 사용될 수 있다.</p>
<ul>
<li>'Allow sshd’ - 방화벽을 통해 SSH 연결을 했을 경우 기록</li>
<li>‘Installed’ - 애플리케이션 설치시 기록</li>
<li>‘su‘ 또는 ‘sudo‘ - su 또는 sudo 명령어를 실행했을 경우 기록</li>
<li>'MAC AUTH’ - 무선 AP에 연결했을 때 남기는 MAC 기록</li>
</ul>
<p>특히 ‘su’, ‘sudo’, ‘ssh’는 공격자가 침투 후 자주 사용하는만큼 해당 로그에서 꼭 확인해봐야할 것이다.</p>
<h2>결론</h2>
<p>OS X에는 여러 커널을 조합하여 만들어진만큼 해당 커널에서 제공하는 많은 로그가 존재한다. 또한, 애플에서 소프트웨어 충돌이나 커널 패닉과 같은 상황을 대비하여 애플에서 정의한 파일포맷으로 만들어진 유용한 아티팩트가 많이 있다. 이러한 아티팩트를 미리미리 알아둔다면, 생각지도 못한 보석을 찾을 수 있을 것이라 생각한다.</p>
<h2>참고자료</h2>
<p>The Apple System Log - Part I, http://crucialsecurityblog.harris.com/2011/06/22/the-apple-system-log-–-part-1/, Crucial Security Forensics Blog<br />
The Apple System Log - Part II, http://crucialsecurityblog.harris.com/2011/08/24/the-apple-system-log-–-part-2-–-console-app/, Crucial Security Forensics Blog<br />
Python Module for parsing ASL files, http://code.google.com/p/ccl-asl, ccl Forensics</p>
<p>&nbsp;
