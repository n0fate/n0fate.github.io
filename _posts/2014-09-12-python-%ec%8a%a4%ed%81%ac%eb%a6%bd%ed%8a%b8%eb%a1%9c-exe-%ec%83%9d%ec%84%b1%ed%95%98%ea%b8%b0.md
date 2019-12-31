---
layout: post
title: python 스크립트를 exe로 빌드하기
date: 2014-09-12 17:09:03.000000000 +09:00
type: post
published: true
status: publish
categories:
- Development
tags:
- pyinstaller
- pywin32
- Qt
author: "n0fate"
---
<p class="">파이썬으로 프로그램을 만들다보면 효율적인 사용을 위해 바이너리 형태로 빌드해야하는 경우가 생긴다. 보통 윈도우의 경우에는 py2exe를 사용하고 맥은 py2app을 사용하곤 한다. 요번에 Qt 4기반의 chainbreaker 도구를 개발하면서 바이너리로 빌드하면서 py2exe를 사용했는데 에러가 발생하여 pyinstaller를 사용해보았다.</p>
<p class="">pyinstaller의 경우 현재 안정 버전은 2.1이다. 단, 이 버전은 단일 파일을 생성해주는 --onefile 옵션을 줄 경우, 프로그램 실행 시점에 에러가 발생하는 버그가 있으므로, 개발자 버전을 이용하도록 한다. 홈페이지에 가보면, Development 란에 있는 코드를 다운로드 한다.<br />
<img class="aligncenter full" title="" src="{{ site.baseurl }}/assets/1410502785_thumb.png" alt="" align="middle" /><br />
프로그램은 간단하게 ‘python setup.py install’로 설치할 수 있다. pyinstaller를 사용하려면 하나의 플러그인이 더 필요한데 ‘pywin32’이다. 이 프로그램은 "<a href="http://sourceforge.net/projects/pywin32/files/pywin32/" target="_blank">http://sourceforge.net/projects/pywin32/files/pywin32/</a>“에서 파이썬 버전에 맞게 다운로드할 수 있다.<br />
pywin32를 설치하면, 이제 프로그램 빌드 준비가 완료된 것이다. Pyinstaller는 경로에 한글이 있을 경우 에러가 발생하므로, 영문 디렉터리나 ‘C:’와 같은 루트 디렉터리에서 작업하는 것을 권장한다. Chainbreaker의 경우에는 다음과 같은 명령어로 실행하였다.</p>
<p class="">python pyinstaller.py --onefile --noconsole --icon=C:chainbreakerimagesicon.ico C:chainbreakerchainbreakergui.py</p>
<p class="">빌드하면 pyinstaller 디렉터리에 파이썬 파일명(위의 경우 chainbreakergui)로 디렉터리에 생성된다. 해당 디렉터리에 내의 ‘dist’폴더를 가면 실행파일을 확인할 수 있다.</p>
<p><img class="aligncenter full" title="" src="{{ site.baseurl }}/assets/1410503058_thumb.png" alt="" align="middle" /></p>
<p class="">이제 exe 파일을 실행하면 된다 :)</p>
