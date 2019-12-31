---
layout: post
title: 'APT: OSX/Dockster analysis (Part III)'
date: 2014-02-03 10:22:35.000000000 +09:00
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
<p>&nbsp;</p>
<p>연재:</p>
<p><a title="APT: OSX/Dockster analysis (Part I)" href="http://forensic.n0fate.com/?p=742" target="_blank">APT: OSX/Dockster analysis (Part I)</a></p>
<p><a title="Edit “APT: OSX/Dockster analysis (Part II)”" href="http://forensic.n0fate.com/wp-admin/post.php?post=751&amp;action=edit" target="_blank">APT: OSX/Dockster analysis (Part II)</a></p>
<p>&nbsp;</p>
<p>오늘 알아볼 기능은 Dockster의 마지막 기능으로 C2(Command and Control)에 대해 알아보겠다.</p>
<p>C2 기능은 메인함수(main)에서 맨 마지막에 호출하는 mainD() 함수를 스레드로 실행하며, Part I에서 설정한 시간 값까지만 소켓 통신을 수행하도록 구성되어 있다. 통신을 위한 정보는 Part I에서 복호화된 도메인명(it.eicp.net)과 포트(8088)를 사용하며, 이를 사용자가 정의한 createworkendport 함수의 인자로 전달한다. createworkendport()는 inet_ntoa API로 도메인명을 주소로 변환한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/b56a52fa166153095780908527c32b89.png"><img class="aligncenter size-full wp-image-773" alt="createworkendport()" src="{{ site.baseurl }}/assets/b56a52fa166153095780908527c32b89.png" width="563" height="147" /></a></p>
<p>그 다음 사용자가 정의한 CreateSocketAndConnect 함수를 통해 소켓을 생성하고 서버에 접속을 시도한다. 서버 접속에 성공하면, 서버에 g_groupmd5 값을 전달하는데이는 Part I에서 획득한 MAC address를 md5로 해시한 것이다. 마지막으로 사용자가 정의한 WorkThreadEnter를 실행하며 본격적인 C2 기능을 수행한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/17f68b9195f9b2655cebe936a6121d0b.png"><img class="aligncenter size-full wp-image-774" alt="WorkThreadEnter()" src="{{ site.baseurl }}/assets/17f68b9195f9b2655cebe936a6121d0b.png" width="828" height="381" /></a></p>
<p>감염된 시스템 제어 기능의 핵심 코드를 담고 있는 이 함수는 소켓 통신을 수행하고 VerifyLoop 함수로 공격자와의 인증을 거친다. 이 과정에서 암호화 통신을 위한 RSA 키를 생성하고 모든 인증 과정을 암호화한다. 또한 내부적으로 AccountVerify 함수를 수행하는데 여기에서는 계정 인증에 사용되는 데이터를 AES로 암호화하여 통신한다. 암호화 하는 데이터는 총 2개이며, configuration file에 저장되어 있다. 암호화 키는 RSA로 암호화하여 받는다.</p>
<pre class="lang:default decode:true" title="AES로 암호화된 데이터">(lldb) x/1xs 0x160a0
0x000160a0: "default" // 첫 번째 암호화 데이터
(lldb) x/1xs 0x160a0
"00:0c:29:5b:4d:1b" // 두 번째 암호화 데이터
(lldb)</pre>
<p>VerifyLoop 함수를 통과하면 MainLoopEnter 함수에 진입한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/d757bbdef0b7e9e1555b5da66c14b41d.png"><img class="aligncenter size-full wp-image-775" alt="MainLoopEnter" src="{{ site.baseurl }}/assets/d757bbdef0b7e9e1555b5da66c14b41d.png" width="761" height="440" /></a></p>
<p>위 코드를 보면 쉽게 유추할 수 있겠지만, 사용자에게 명령을 받아서(recv) 받은 숫자에 맞는 명령을 수행한다. 공격자가 볼 수 있는 기본 메뉴에는 '파일 시스템 제어(FileSystemEnter())', '원격 쉘(RShellEnter())', '자기자신 제거(unlink)'가 있다.</p>
<h3>파일 시스템 제어</h3>
<p><!--?xml version="1.0" encoding="UTF-8" standalone="no"?--></p>
<p>파일 시스템 제어는 시스템 정보를 수집하거나 파일을 업로드/다운로드 하는 등의 기능을 수행한다. 모든 수집 결과는 compress_ex와 aes_encypt_ex로 압축 및 암호화하여 전송한다. 각 함수의 기능('함수 명', '명령 번호'로 표시)은 다음과 같다.</p>
<ul>
<li>CMD_getprocesslist(47) : sysctl 명령인 GetBSDProcessList를 호출하여 proc 구조체에서 ppid와 process name을 추출하여 암호화 전송한다. 재밌는 점은 구조체 오프셋 상 PID가 아니라 PPID를 가져온다는 점이다.  기능 상의 오류인지는 확인하지 못하였다.</li>
<li>CMD_killprocess(46) : 공격자에게 종료할 프로세스의 PID를 받아서 kill 명령으로 프로세스를 종료한다.</li>
<li>CMD_getservice(45) : launchctl list 명령을 RunOrder로 수행하고 결과를 암호화하여 전송한다.</li>
<li>CMD_uploadfile(17) : 파일을 업로드 한다. 만약에 업로드할 파일이 해당 디렉터리에 있으면, 삭제하고 업로드한다. 파일 전송에서 사용되는 모든 패킷은 암호화 된다.</li>
<li>CMD_downloadfile(18) : threaddownloadfile을 스레드로 실행하여 파일을 다운로드 한다.</li>
<li>CMD_list (29) : 공격자가 인자로 준 경로 정보를 해석하여 listdir()을 호출한다. listdir()은 opendir와 readdir API를 이용하여 디렉터리 내에 있는 파일 또는 디렉터리 목록을 전송한다.</li>
<li>CMD_filemove(20) : rename API를 이용하여 파일을 이동하거나 파일 이름을 변경한다.</li>
<li>CMD_filecopy(30) : open-&gt;creat-&gt;read and write call 로 파일을 복사한다.</li>
<li>CMD_filedelete(21) : unlink API로 해당 파일을 제거한다.</li>
<li>CMD_dircreate(24) : mkdir API로 디렉터리를 생성한다.</li>
<li>CMD_dirmove(25) : rename API로 디렉터리를 이동시킨다.</li>
<li>CMD_dircopy(31) : copytree API로 특정 디렉터리의 모든 데이터를 복사한다.</li>
<li>CMD_dirdelete(26) : rmdirall API로 특정 디렉터리의 모든 데이터를 제거한다.</li>
<li>CMD_fileattrib(32) : stat API의 결과 중 파일의 디바이스 타입(st_rdev), 생성시간(st_ctimespec), 최종 수정시간(st_mtimespec), 최종 접근시간(st_atimespec) 정보를 제공한다.</li>
<li>CMD_keylogger(34) : 키로거가 생성하는 파일인 $HOME/.Trash/.tmp 파일을 읽어서 공격자에게 전송한다.</li>
<li>종료 (-1) : 이전으로 돌아간다.</li>
</ul>
<p>&nbsp;</p>
<p><!--?xml version="1.0" encoding="UTF-8" standalone="no"?--></p>
<h3>원격 쉘</h3>
<p><!--?xml version="1.0" encoding="UTF-8" standalone="no"?--> CMD_RShell로 실행되며, newshell에 g_ip, g_rport, “/bin/bash” 를 인자로 전달한다. newshell()은 내부적으로 dup2() API를 이용하여 stdin, stdout, stderr를 소켓으로 입/출력하게 한 후, execlp() API로 bash를 실행한다. 이렇게 함으로 공격자와 bash 간에 소켓 통신을 수행할 수 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/3b6701a4324b41e9cda2d0f3886cf192.png"><img class="aligncenter size-full wp-image-776" alt="RShell()" src="{{ site.baseurl }}/assets/3b6701a4324b41e9cda2d0f3886cf192.png" width="981" height="291" /></a></p>
<p>&nbsp;</p>
<h3>자기 자신 삭제 및 종료</h3>
<p><!--?xml version="1.0" encoding="UTF-8" standalone="no"?--> 삭제 명령(237)을 받으면, unlink 명령으로 자동 실행 정보와 .Dockset 파일을 제거한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/f090f41a5135718ce6e2e427630bd8b2.png"><img class="aligncenter size-full wp-image-777" alt="unlink and exit" src="{{ site.baseurl }}/assets/f090f41a5135718ce6e2e427630bd8b2.png" width="807" height="242" /></a></p>
<p>&nbsp;</p>
<h2>결론</h2>
<p>Dockster는 APT 공격이라 불리기 딱 좋은 구성을 하고 있다.  첫 번째로 모든 통신을 암호화하고 설정 파일을 인코딩하여 보관하고 있다. 두 번째로 C2 기능을 통해 지속성을 확보했으며, 세 번째로 티벳의 dalai lama를 명확한 공격 대상으로 하였다. 각 기능에는 오류가 발생할 소지를 최소화하기 위해 다양한 검증 함수를 내장하고 있다. Mac OS X를 대상으로 하는 악성코드는 그 개체 수는 적지만 각 개체의 기능이 다양하게 구성되어 있으며, APT 공격에 많이 활용되므로 항상 기업 및 기관의 보안 울타리에서 제외시킴으로 공격의 시발점이 되는 구멍(hole)을 만들지 말아야 할 것이다.</p>
