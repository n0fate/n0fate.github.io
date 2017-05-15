---
layout: post
title: Introduction to OSX Auditor
date: 2013-08-04 16:43:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2013/08/introduction-to-osx-auditor.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/7264905959214328514
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1848'
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<h3>1. 소개</h3>
<p>최근에 Mac OS X 사용자가 많아지기도 했고, 윈도우 포렌식을 연구하는 사람이 많아져서인지 1~2년 사이에 OS X에 대한 포렌식 도구가 속속 등장하고 있다.</p>
<p>대략 2008년까지만 해도 Mac OS X 침해대응 시 라이브 정보를 수집하려면 유닉스에서 사용하던 쉘스크립트를 약간 수정하여 정보 수집을 해야했다. 뭐 사실 그 때만해도 메모리 포렌식 도구는 있지도 않았으니, 다른 고급 기술을 Mac OS X에서 사용한다는건 꿈같은 일이였다. 뭐 국내에선 크게 수요도 있지 않았지만 말이다.</p>
<p>하지만 아이폰의 영향인지 윈도우에서 해먹을 건 다 해먹어서인지 2010년부터 슬슬 Mac OS X에 대한 포렌식 분석 기술이 소개되기 시작했다. 내가 만든 volafox도 윈도우에선 할게없다보니 Mac OS X로 만들어본 메모리 포렌식 도구였고, 2011년초에 라이브 정보를 수집하는MacResponseLE도 등장하였다. 사실 내가 보았을 때 MacResponse는 잘나가는 오픈소스 프로젝트가 될 줄 알았는데, 사람들의 무관심으로 인해 라이언 이상은 제대로 지원을 못한다-_-;; 사실 개발회사에서 추진하는 오픈소스 프로젝트 치고는 정말 빠르게 버려진 프로젝트였다. (이 프로젝트와 관련된 글은 <a href="http://forensic.n0fate.com/2013/07/mac-response-forensicsmacresponsele.html?utm_source=BP_recent" target="_blank">여기</a>를 참조하기 바람)</p>
<p>최근에 몸이 안좋다보니, 공부를 멀리하고 유흥(?)을 즐기다가 이제 안되겠다 싶어서 인터넷을 좀 보다가 OSX Auditor라는 오픈소스 도구를 발견했다. 대충 보름 전에 최종 커밋을 한 것을 보아 나름 개발자가 애착을 가지고 진행하는 프로젝트인 것 같았다. <b>(제발 오래가길..)</b></p>
<p>일단 언어 자체는 파이썬으로 개발되어 있어서 모든 맥에서 별도의 라이브러리 설치없이 잘 작동하는 점은 매우 마음에 들었다. 사실 개인적으로 포렌식 도구가 대상 시스템에서 동작한다면, 최대한 대상 시스템에 아무것도 설치하지 않고 동작해야 한다고 생각하기 때문에 파이썬 언어를 사용하여 구현하는 것을 선호하는 편이다. 하지만 이 도구는 파이썬을 사용했음에도 pyobjc라는 파이썬 라이브러리를 설치해서 돌리라고 하고 있다.</p>
<p>뭐 사실 라이브 수집 자체가 시스템 무결성을 손상시키니 이정도 설치하는 것이 뭐가 문제가 있겠냐 싶겠지만, 추가적인 라이브러리를 설치하는 것과 그냥 돌리는 것은 생각보다 무결성 침해의 차이가 크다. <b>(이건 운영체제를 좀 아는 분들은 동의할 것이다. 왜냐고 댓글로 물어봐도 귀찮아서 안써줌.)</b></p>
<h3>2. 수집 정보</h3>
<p>어쨌든, 일단 도구가 나왔으니, 어떠한 정보를 수집하는지 확인해보기로 했다. GitHub에 나온 정보에 따르면, 다음과 같은 정보를 수집한다.</p>
<blockquote><p><i>OS X Auditor parses and hashes the following artifacts on the running system or a copy of a system you want to analyze:</i></p>
<ul>
<li><i>the kernel extensions</i></li>
</ul>
<ul>
<li><i>the system agents and daemons</i></li>
</ul>
<ul>
<li><i>the third party's agents and daemons</i></li>
</ul>
<ul>
<li><i>the old and deprecated system and third party's startup items</i></li>
</ul>
<ul>
<li><i>the users' agents</i></li>
</ul>
<ul>
<li><i>the users' downloaded files</i></li>
</ul>
<ul>
<li><i>the installed applications</i></li>
</ul>
<p><i>It extracts:</i></p>
<ul>
<li><i>the users' quarantined files</i></li>
</ul>
<ul>
<li><i>the users' Safari history, downloads, topsites, HTML5 databases and localstore</i></li>
</ul>
<ul>
<li><i>the users' Firefox cookies, downloads, formhistory, permissions, places and signons</i></li>
</ul>
<ul>
<li><i>the users' Chrome history and archives history, cookies, login data, top sites, web data, HTML5 databases and local storage</i></li>
</ul>
<ul>
<li><i>the users' social and email accounts</i></li>
</ul>
<ul>
<li><i>the WiFi access points the audited system has been connected to (and tries to geolocate them)</i></li>
</ul>
</blockquote>
<p>크게 "KEXT", "자동실행", "설치된 소프트웨어", 다운로드한 파일"을 분석한다고 한다. 그리고 다운로드하여 실행이 제한된 파일과, 웹브라우저 내역, 소셜 및 이메일 정보, 와이파이 연결 정보를 추출한다고 한다. 사실 다른 정보야 MacResponse에서도 잘 진행했던 부분이고, 이미 많이 알려진 정보를 수집하는거라 크게 감흥은 없었는데, 다음 두가지 기능이 매우 마음에 들었다.</p>
<ul>
<li>각 애플리케이션의 무결성을 입증하기 위해 설치한 애플리케이션의 md5 해시를 산출하고, 이 해시를 이용하여 VirusTotal의 링크를 제공.</li>
<li>현재까지 알려진 자동실행 내역을 수집하고, 해당 plist를 분석하여 자동실행 요소를 추출, 각 파일의 해시 값을 산출하여 VirusTotal의 링크를 제공.</li>
</ul>
<p>기존 도구는 그냥 단순히 데이터를 수집하고 목록을 보여주는데 그쳤는데, 이러한 기능은 분석가가 초동분석을 좀 더 쉽게 할 수 있도록 한다.</p>
<h3>3. 설치하기</h3>
<p>이 도구는 파이썬 기반으로 동작하기 때문에 맥 사용자는 손쉽게 사용할 수 있다. Mac OS X에서 사용한다면 다음과 같은 순서로 구동하면 된다.</p>
<p><b>1. GitHub에서 최신버전 클론 (git 명령어는 github for Mac 설치 시 사용 가능, MacPort도 됨.)</b></p>
<pre class="lang:default theme:twilight" title="Cloning OSXAuditor project">$ git clone https://github.com/jipegit/OSXAuditor.git</pre>
<p><b>2. pyobjc 설치 (easy_install로 설치함.)</b></p>
<pre class="lang:default theme:twilight" title="ObjectC for Python installation">$ sudo easy_install pyobjc

Password:
Searching for pyobjc
Reading http://pypi.python.org/simple/pyobjc/
Best match: pyobjc 2.5.1
Downloading https://pypi.python.org/packages/source/p/pyobjc/pyobjc-2.5.1.tar.gz#md5=f242cff4a25ce397bb381c21a35db885
Processing pyobjc-2.5.1.tar.gz
Running pyobjc-2.5.1/setup.py -q bdist_egg --dist-dir /tmp/easy_install-Qopl7q/pyobjc-2.5.1/egg-dist-tmp-7JX7TK
warning: install_lib: 'build/lib' does not exist -- no Python modules to install

Adding pyobjc 2.5.1 to easy-install.pth file

Installed /Library/Python/2.7/site-packages/pyobjc-2.5.1-py2.7.egg
Processing dependencies for pyobjc
Searching for pyobjc-framework-ServiceManagement==2.5.1
Reading http://pypi.python.org/simple/pyobjc-framework-ServiceManagement/
Best match: pyobjc-framework-ServiceManagement 2.5.1
Downloading https://pypi.python.org/packages/source/p/pyobjc-framework-ServiceManagement/pyobjc-framework-ServiceManagement-2.5.1.tar.gz#md5=0ec67fb8fae22104643d423a2b66ca17
Processing pyobjc-framework-ServiceManagement-2.5.1.tar.gz
Running pyobjc-framework-ServiceManagement-2.5.1/setup.py -q bdist_egg --dist-dir /tmp/easy_install-xz0b4q/pyobjc-framework-ServiceManagement-2.5.1/egg-dist-tmp-UvaErB
error: Installed distribution pyobjc-core 2.3.2a0 conflicts with requirement pyobjc-core&gt;=2.5.1</pre>
<p>이렇게만 하면 구동할 준비가 완료된다.</p>
<h3>4. 돌려보기</h3>
<p>이제 대충 기능은 알았으니, 테스트 겸 도구를 돌려보았다. 도구는 위에서 설명한 pyobjc만 easy_install로 설치해놓으면 잘 돌아간다... <b>는 훼이크고</b> 다음과 같은 에러를 발생한다.</p>
<table cellspacing="0" cellpadding="0" align="center">
<tbody>
<tr>
<td><a href="http://2.bp.blogspot.com/-jUPORXnAp3A/Uf4ATCte7lI/AAAAAAAABDk/MLBysg4rO3Y/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2013-08-04+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+4.18.29.png"><img alt="" src="{{ site.baseurl }}/assets/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2013-08-04+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+4.18.29.png" border="0" /></a></td>
</tr>
<tr>
<td><b>??</b></td>
</tr>
</tbody>
</table>
<p>이 에러는 외국 친구가 개발할 때 많이 실수하는 부분으로 동아시아권의 URL정보가 있는 경우, 유니코드 인코딩을 제대로 해석 못해서 발생하는 문제이다. 전에 volafox 개발자 중 한명이 lsof를 플러그인을 init commit했을 때도 간헐적으로 저 에러를 목격한 적이 있었다. 에러를 알려주긴 해야하는데.. 일단은 영어로 쓰기가 귀찮아서 안알려주고 있다. 나중에 생각나면 보내야겠다.<br />
<b><br />
</b><b>(여담이지만 예전에 웹크롤러를 만들 때도 저놈의 인코딩 때문에 짜증나 죽는줄 알았음.)</b></p>
<h3>5. 가상머신에서 돌려보기</h3>
<p>여튼 이 문제 때문에 도구 결과를 제대로 볼 수도 없어서 가상머신인 라이언으로 돌려봤다. 가상머신에는 들은게 없기 때문에 에러가 날리가 없어서 기쁜 마음으로 도구를 돌려보았다.</p>
<table cellspacing="0" cellpadding="0" align="center">
<tbody>
<tr>
<td><a href="http://4.bp.blogspot.com/-NaPU5id-Mh4/Uf4Bm-MA-8I/AAAAAAAABD0/vzkrhyppOlM/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2013-08-04+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+4.24.30.png"><img alt="" src="{{ site.baseurl }}/assets/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2013-08-04+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+4.24.30.png" width="640" height="464" border="0" /></a></td>
</tr>
<tr>
<td><b>자동 실행 내역</b></td>
</tr>
</tbody>
</table>
<p>HTML로 결과를 뽑아봤는데, 생각보다 깔끔한 UI를 제공한다. 최근에 volafox도 모든 정보를 한번에 분석해서 HTML로 뽑아주는걸 생각 중인데, 요런 output으로 출력하는 것도 나쁘진 않은 것 같다.</p>
<p>위 결과는 자동실행(Startup) 항목에 대한 분석 결과이다. 자동실행의 각 항목에 대해서는 다음 포스팅으로 다룰 예정이니 지금은 그냥 윈도우 자동 실행과 똑같이 생각하면 된다. 사실 포렌식 분석 보고서에도 '자동 실행' 정도도 과한 정보이기 때문에 이정도 정보만 제공해도 충분하다고 생각한다. <b>나처럼 덕스럽게 파고들 것 아니면 말이다.</b></p>
<p>여하튼 자동 실행이 가능한 10군데에 있는 항목을 추출하고, 각 항목에서 로드하는 파일의 경로와 해시값을 기반으로 하는 Virus Total 링크 정보, 파일 생성 및 수정시간을 제공한다. 글을 쓰다보니, 저걸 테이블로 보여줘서 시간 순이나 파일 명 순 정렬을 해주면 더 좋을 것 같다는 생각이 들었으나 <b>내가하긴 귀찮으니 생략</b>한다.</p>
<h3>5. 결론</h3>
<div>이 도구는 가뜩이나 척박한 Mac OS X 포렌식 시장에서 한줄기 단비같은 도구인줄 알았으나, 아직은 갈길이 멀은 포렌식 도구이다. 사실 <b>최근에는 메모리 분석 도구의 기능이 막강해져서 대부분의 라이브 데이터 분석이 가능</b>하다보니 이러한 도구의 의미가 많이 퇴색되었다. 하지만, 디스크 이미지를 분석하기엔 시간이 부족한 상황에서 빠르게 알려진 악성코드가 존재했는지를 확인하기에는 여러모로 유용한 도구로 판단된다.</div>
<div>디지털 포렌식 분야 또는 악성코드 분석 분야에 종사자라면 한번 정도 고려할만한 포렌식 도구라 생각한다 :-)</div>
