---
layout: post
title: 'Volafox Project: volafox 0.7 alpha 1 release'
date: 2011-12-19 20:26:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2011/12/volafox-project-volafox-07-alpha-1.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/430750505337743654
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '2242'
  fusion_builder_content_backup: <span>정말 오랜만에 볼라폭스 새버전이 릴리즈 되었습니다. 이번 릴리즈에는 SVN을
    통해 개발된 다양한 기능이 추가되었습니다. 이 글에는 0.6에서 추가된 기능을 적어보려고 합니다.</span><br /><span><br /></span><div><a
    href="http://4.bp.blogspot.com/-kZ-11mfkYGg/Tu8hijxufSI/AAAAAAAAAOk/vBPLx6lN488/s1600/volafox.png"
    imageanchor="1"><img border="0" src="http://4.bp.blogspot.com/-kZ-11mfkYGg/Tu8hijxufSI/AAAAAAAAAOk/vBPLx6lN488/s1600/volafox.png"></a></div><span><br
    /></span><span><br /></span><strong><span>1. 64bit Kernel Support</span></strong><br
    /><span>볼라폭스에서 64비트 커널 기반의 메모리 이미지 분석을 지원합니다. 스노우 레오파드까지만 해도 32비트 부팅이 기본으로 되어있었기
    때문에 32비트 커널에 올라온 64비트 프로세스를 분석하지 않는 이상 특별히 64비트 분석 기능을 추가할 필요가 없었습니다만, 라이언부턴 CPU가
    64비트를 지원할 경우, 64비트 커널 부팅이 기본으로 설정되어 있습니다. 이에 라이언 운영체제 지원을 위해 64비트 메모리 이미지 분석 기능을
    추가하였습니다.</span><br /><span><br /></span><strong><span>2. Mac OS X Lion support</span></strong><br
    /><span>Mac OS X Lion에서 수집한 메모리 이미지 분석을 지원합니다. 이 기능은 32비트 64비트 상관없이 가능하며, 현재까지
    나온 모든 라이언 운영체제(10.7.0 ~ 10.7.2)를 지원합니다.</span><br /><span><br /></span><strong><span>3.
    Automated architecture, kernel version detection</span></strong><br /><span>기존엔
    해당 메모리 이미지와 맞는 커널 이미지를 함께 입력으로 받아 분석을 진행하였습니다만, 물리 메모리에 특정 오프셋에 존재하는 구조체 정보를 통해
    해당 정보를 수집할 수 있습니다. 그리하여 메모리 분석을 위해 입력해주는 인자가 (메모리 이미지 명, 분석할 정보)로 줄어들게 되었습니다.</span><br
    /><a href="http://lh5.ggpht.com/-UiSGBYLhwvY/Tu8fUkrvzQI/AAAAAAAAAOE/N5e3dkN6m5g/s1600-h/image%25255B3%25255D.png"><span><img
    alt="image" border="0" height="332" src="http://lh4.ggpht.com/-V_3wMugJ5ag/Tu8fVkPyYTI/AAAAAAAAAOM/Sw0J8rOzZtE/image_thumb%25255B1%25255D.png?imgmax=800"
    title="image" width="528"></span></a><br /><span><br /></span><strong><span>4.
    Generating Symbol List Database</span></strong><br /><span>기존엔 심볼 정보를 Mac OS X의
    커널 이미지인 mach_kernel에서 실행 시 추출해서 분석하였습니다. 볼라폭스 0.7에선 overlay data로 DB화하여 관리합니다.
    거의 대부분의 커널 심볼 정보를 저장하고 있으며, 이를 통해 볼라폭스는 메모리 이미지만 입력으로 받아도 분석을 진행할 수 있습니다. 만약 심볼
    정보를 가지지 않은 경우엔 커널 이미지를 추출하여 ‘overlay_generator.py’를 이용하여 overlay data를 생성할 수 있습니다.
    현재 64비트 심볼 데이터베이스는 많이 구성되지 않은 상태입니다.</span><br /><span><br /></span><strong><span>5.
    Network Information</span></strong><br /><span>이제 볼라폭스는 네트워크 정보를 분석합니다. 단 아직 완벽한
    상태는 아닙니다.  실제 전체 네트워크 상태보다 20%적은 정보를 보여줍니다. 하지만 실제 외부와 연결된 네트워크 정보는 모두 표현해 줍니다.
    이 기능은 계속 테스트를 진행 중이기 때문에 실험적 기능이라 할 수 있습니다.</span><br /><a href="http://lh3.ggpht.com/-xp3QYNvOgtY/Tu8fWiphhXI/AAAAAAAAAOU/UEsGTI6sCLM/s1600-h/image%25255B8%25255D.png"><span><img
    alt="image" border="0" height="313" src="http://lh3.ggpht.com/-ChgguOLvyb4/Tu8fXXPeOEI/AAAAAAAAAOc/ZQ2lRGebxng/image_thumb%25255B4%25255D.png?imgmax=800"
    title="image" width="518"></span></a><br /><span><br /></span><strong><span>6.
    MMR memory format to Linear memoty format</span></strong><br /><span>Mac Memory
    Reader를 이용하여 생성한 메모리 이미지는 Mach-O 포맷을 가지고 있습니다. 볼라폭스에는 이를 Linear Format으로 바꿔주는
    flatten.py 가 내장되어 있습니다. 단, 지금은 32비트 메모리 이미지만 정상적으로 변환됩니다. 이 도구는 Mac Memory Reader
    개발자인 hajime Inoue 가 직접 개발하였습니다.</span><br /><span><br /></span><strong><span>7.
    Windows Binary Version Release</span></strong><br /><span>py2exe로 생성한 volafox.exe
    파일을 함께 배포합니다. 이에 윈도우에 아무것도 설치되어 있지 않더라도, 볼라폭스를 동작시킬 수 있습니다. 이는 사용에 어려움을 겪는 분들을
    위해 추가한 것입니다. 단 바이너리엔 overlay를 생성하는 overlay_generator.py 의 기능이나, Mac Memory Reader에서
    생성한 이미지를 Linear Format으로 바꿔주는 flatten.py 의 기능은 포함되어 있지 않습니다.</span><br /><span><br
    /></span><b><span>8. Process Virtual Memory Map </span><span>Permission</span></b><br
    /><span>프로세스 가상 메모리 맵에 권한 기능을 추가하였습니다. 이 것 뿐만 아니라 프로세스와 관련된 여러 정보를 뿌려줄 예정입니다.</span><br
    /><div><a href="http://3.bp.blogspot.com/-SRIWG413vDo/Tu8ijCjCDpI/AAAAAAAAAOs/s9TNH1O8h9A/s1600/memory+map.PNG"
    imageanchor="1"><img border="0" height="298" src="http://3.bp.blogspot.com/-SRIWG413vDo/Tu8ijCjCDpI/AAAAAAAAAOs/s9TNH1O8h9A/s400/memory+map.PNG"
    width="400"></a></div><span><br /></span><span><br /></span><span><br /></span><span>SVN을
    통해 개발한 기간이 길다보니 여러모로 추가된 기능이 많습니다. 사실 alpha 딱지 띄어도 될 것 같긴 하지만, 아직 안정화 단계를 거치지
    않았기 때문에 넣어두었습니다.  이제 2012년이 다가오네요. 볼라폭스에서 앞으로 할 일은 크게 두 가지 입니다.</span><br /><span><br
    /></span><strong><span>1. Open File Listing/Dump</span></strong><br /><span>프로세스가
    오픈한 파일의 목록을 출력하고, 해당 파일의 메모리 영역만 덤프하는 기능을 구현할 계획입니다. 본 기능은 구조체도 어느정도 찾아놓았고 조만간
    구현할 예정이였는데, 금일 메일로 같이하는 member가 자신이 1월 초부터 해당 작업을 진행하겠다고 하여, 그때부터 구현이 이루어질 것 같습니다.
    (전 이거 말고도 할게 많기에-_-;)</span><br /><span><br /></span><strong><span>2. Inline Function
    Hooking Detection</span></strong><br /><span>이전에도 포스팅한 적이 있습니다만, 인라인 함수 후킹 탐지
    기능을 추가할 계획입니다. 방법 다 찾고 구현만 하면 되는데, 차일피일 미루다가 여기까지 와버렸네요-_-; distorm library를 사용할
    계획입니다.</span><br /><span><br /></span><strong><span>3. Hibernation Image Forensics</span></strong><br
    /><span>이건 forensicinsight에서 진행하고 있습니다. 자세한 내용은 이전 포스팅을 참조하세요 :)</span><br /><span><br
    /></span><span>이 두 기능만 완료되면 볼라폭스 1.0 으로 릴리즈할 생각을 하고 있습니다. 지금까진 더 이상 기능 추가할만한게
    없거든요 :) 앞으로도 할게 많네요. 다들 즐거운 저녁 되시기 바랍니다.!!</span><div>n0fate's Forensic Space
    :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p><span>정말 오랜만에 볼라폭스 새버전이 릴리즈 되었습니다. 이번 릴리즈에는 SVN을 통해 개발된 다양한 기능이 추가되었습니다. 이 글에는 0.6에서 추가된 기능을 적어보려고 합니다.</span><br /><span><br /></span>
<div><a href="http://4.bp.blogspot.com/-kZ-11mfkYGg/Tu8hijxufSI/AAAAAAAAAOk/vBPLx6lN488/s1600/volafox.png" imageanchor="1"><img border="0" src="{{ site.baseurl }}/assets/volafox.png" /></a></div>
<p><span><br /></span><span><br /></span><strong><span>1. 64bit Kernel Support</span></strong><br /><span>볼라폭스에서 64비트 커널 기반의 메모리 이미지 분석을 지원합니다. 스노우 레오파드까지만 해도 32비트 부팅이 기본으로 되어있었기 때문에 32비트 커널에 올라온 64비트 프로세스를 분석하지 않는 이상 특별히 64비트 분석 기능을 추가할 필요가 없었습니다만, 라이언부턴 CPU가 64비트를 지원할 경우, 64비트 커널 부팅이 기본으로 설정되어 있습니다. 이에 라이언 운영체제 지원을 위해 64비트 메모리 이미지 분석 기능을 추가하였습니다.</span><br /><span><br /></span><strong><span>2. Mac OS X Lion support</span></strong><br /><span>Mac OS X Lion에서 수집한 메모리 이미지 분석을 지원합니다. 이 기능은 32비트 64비트 상관없이 가능하며, 현재까지 나온 모든 라이언 운영체제(10.7.0 ~ 10.7.2)를 지원합니다.</span><br /><span><br /></span><strong><span>3. Automated architecture, kernel version detection</span></strong><br /><span>기존엔 해당 메모리 이미지와 맞는 커널 이미지를 함께 입력으로 받아 분석을 진행하였습니다만, 물리 메모리에 특정 오프셋에 존재하는 구조체 정보를 통해 해당 정보를 수집할 수 있습니다. 그리하여 메모리 분석을 위해 입력해주는 인자가 (메모리 이미지 명, 분석할 정보)로 줄어들게 되었습니다.</span><br /><a href="http://lh5.ggpht.com/-UiSGBYLhwvY/Tu8fUkrvzQI/AAAAAAAAAOE/N5e3dkN6m5g/s1600-h/image%25255B3%25255D.png"><span><img alt="image" border="0" height="332" src="{{ site.baseurl }}/assets/image_thumb%25255B1%25255D.png?imgmax=800" title="image" width="528" /></span></a><br /><span><br /></span><strong><span>4. Generating Symbol List Database</span></strong><br /><span>기존엔 심볼 정보를 Mac OS X의 커널 이미지인 mach_kernel에서 실행 시 추출해서 분석하였습니다. 볼라폭스 0.7에선 overlay data로 DB화하여 관리합니다. 거의 대부분의 커널 심볼 정보를 저장하고 있으며, 이를 통해 볼라폭스는 메모리 이미지만 입력으로 받아도 분석을 진행할 수 있습니다. 만약 심볼 정보를 가지지 않은 경우엔 커널 이미지를 추출하여 ‘overlay_generator.py’를 이용하여 overlay data를 생성할 수 있습니다. 현재 64비트 심볼 데이터베이스는 많이 구성되지 않은 상태입니다.</span><br /><span><br /></span><strong><span>5. Network Information</span></strong><br /><span>이제 볼라폭스는 네트워크 정보를 분석합니다. 단 아직 완벽한 상태는 아닙니다.  실제 전체 네트워크 상태보다 20%적은 정보를 보여줍니다. 하지만 실제 외부와 연결된 네트워크 정보는 모두 표현해 줍니다. 이 기능은 계속 테스트를 진행 중이기 때문에 실험적 기능이라 할 수 있습니다.</span><br /><a href="http://lh3.ggpht.com/-xp3QYNvOgtY/Tu8fWiphhXI/AAAAAAAAAOU/UEsGTI6sCLM/s1600-h/image%25255B8%25255D.png"><span><img alt="image" border="0" height="313" src="{{ site.baseurl }}/assets/image_thumb%25255B4%25255D.png?imgmax=800" title="image" width="518" /></span></a><br /><span><br /></span><strong><span>6. MMR memory format to Linear memoty format</span></strong><br /><span>Mac Memory Reader를 이용하여 생성한 메모리 이미지는 Mach-O 포맷을 가지고 있습니다. 볼라폭스에는 이를 Linear Format으로 바꿔주는 flatten.py 가 내장되어 있습니다. 단, 지금은 32비트 메모리 이미지만 정상적으로 변환됩니다. 이 도구는 Mac Memory Reader 개발자인 hajime Inoue 가 직접 개발하였습니다.</span><br /><span><br /></span><strong><span>7. Windows Binary Version Release</span></strong><br /><span>py2exe로 생성한 volafox.exe 파일을 함께 배포합니다. 이에 윈도우에 아무것도 설치되어 있지 않더라도, 볼라폭스를 동작시킬 수 있습니다. 이는 사용에 어려움을 겪는 분들을 위해 추가한 것입니다. 단 바이너리엔 overlay를 생성하는 overlay_generator.py 의 기능이나, Mac Memory Reader에서 생성한 이미지를 Linear Format으로 바꿔주는 flatten.py 의 기능은 포함되어 있지 않습니다.</span><br /><span><br /></span><b><span>8. Process Virtual Memory Map </span><span>Permission</span></b><br /><span>프로세스 가상 메모리 맵에 권한 기능을 추가하였습니다. 이 것 뿐만 아니라 프로세스와 관련된 여러 정보를 뿌려줄 예정입니다.</span>
<div><a href="http://3.bp.blogspot.com/-SRIWG413vDo/Tu8ijCjCDpI/AAAAAAAAAOs/s9TNH1O8h9A/s1600/memory+map.PNG" imageanchor="1"><img border="0" height="298" src="{{ site.baseurl }}/assets/memory+map.PNG" width="400" /></a></div>
<p><span><br /></span><span><br /></span><span><br /></span><span>SVN을 통해 개발한 기간이 길다보니 여러모로 추가된 기능이 많습니다. 사실 alpha 딱지 띄어도 될 것 같긴 하지만, 아직 안정화 단계를 거치지 않았기 때문에 넣어두었습니다.  이제 2012년이 다가오네요. 볼라폭스에서 앞으로 할 일은 크게 두 가지 입니다.</span><br /><span><br /></span><strong><span>1. Open File Listing/Dump</span></strong><br /><span>프로세스가 오픈한 파일의 목록을 출력하고, 해당 파일의 메모리 영역만 덤프하는 기능을 구현할 계획입니다. 본 기능은 구조체도 어느정도 찾아놓았고 조만간 구현할 예정이였는데, 금일 메일로 같이하는 member가 자신이 1월 초부터 해당 작업을 진행하겠다고 하여, 그때부터 구현이 이루어질 것 같습니다. (전 이거 말고도 할게 많기에-_-;)</span><br /><span><br /></span><strong><span>2. Inline Function Hooking Detection</span></strong><br /><span>이전에도 포스팅한 적이 있습니다만, 인라인 함수 후킹 탐지 기능을 추가할 계획입니다. 방법 다 찾고 구현만 하면 되는데, 차일피일 미루다가 여기까지 와버렸네요-_-; distorm library를 사용할 계획입니다.</span><br /><span><br /></span><strong><span>3. Hibernation Image Forensics</span></strong><br /><span>이건 forensicinsight에서 진행하고 있습니다. 자세한 내용은 이전 포스팅을 참조하세요 :)</span><br /><span><br /></span><span>이 두 기능만 완료되면 볼라폭스 1.0 으로 릴리즈할 생각을 하고 있습니다. 지금까진 더 이상 기능 추가할만한게 없거든요 :) 앞으로도 할게 많네요. 다들 즐거운 저녁 되시기 바랍니다.!!</span>
<div>n0fate's Forensic Space :)</div>
