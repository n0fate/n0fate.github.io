---
layout: post
title: Dumping bash history in memory image
date: 2013-11-29 21:41:48.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags:
- bash
- history
- volafox
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  avada_post_views_count: '1463'
  fusion_builder_content_backup: |-
    <h3>1. 글을 시작하며..</h3>
    최근에 이런저런 사건사고가 터지면서 디지털 포렌식이라는 용어가 언론에 자주 나타나고 있다. 최근의 사용자가 아날로그보다 디지털에 점점 의존도가 높아지면서 대부분의 범죄 사고 분석의 주요 증거물이 됨으로 더더욱 중요도가 높아지면서 생겨난 자연스러운 현상이다.

    침해사고 대응 시나 기업 내 정보 유출 사고에서는 어떠한 사용자(Who)가 어떠한 시스템(Where)을 어느 시점(When)에 어떤 행동(What)을 어떻게(How) 제어했는가가 중요한 요소가 된다.  디지털 수사관은 이러한 정보를 효과적으로 얻기 위해 다양한 디지털 증거 수집 도구를 활용하여 데이터를 수집한다.

    디지털 증거를 수집하는 방법은 사건 현장에서 지속적으로 변경되는 데이터의 현재 상태를 신속하게 수집하기 위한 라이브 포렌식(Live Forensics)와 시스템의 현재 상태 수집, 라이브 데이터의 무결성 보장, 추 후 디스크 분석과 연계를 위한 메모리 포렌식(Memory Forensics), 마지막으로 가장 많은 디지털 증거를 가지며, 가장 많은 분석 시간이 할애되는 디스크 포렌식(Disk Forensics)로 크게 3가지로 나눠볼 수 있다.

    <a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-9.47.45-PM.png"><img class="aligncenter size-full wp-image-513" alt="Screen Shot 2013-11-29 at 9.47.45 PM" src="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-9.47.45-PM.png" width="648" height="359" /></a>
    <p style="text-align: center;"><strong>Figure 1. 각 기술의 포용 범위</strong></p>
    최근에는 메모리 포렌식 기술이 라이브 포렌식에서 수집하는 대부분의 디지털 증거를 포용하면서 이러한 증거의 무결성 보장을 위한 다양한 기능을 추가 제공하고, 라이브 포렌식에 비해 상대적으로 높은 시스템 무결성을 보장하다보니 메모리와 디스크 포렌식만 수행하는 경우도 종종 있다.

    특히, 메모리 분석에서는 스크립트 수준의 라이브 데이터 분석보다 더 많은 정보를 산출해 낼 수 있기 때문에 필자는 주변 분석가들에게 시스템 분석 과정에서 꼭 메모리 분석을 수행하기를 권고하고 있다.

    오늘 포스팅에서는 단순 라이브 데이터 스크립트에서는 수집할 수 없는 쉘 히스토리(shell history)의 시간 정보(timestamp)를 메모리에서 추출하는 방법에 대해 작성하고자 한다. 여기에선 여러 쉘 중 가장 주변에서 접하기 쉬우면서 발전된 쉘인 bash(bourne-again shell)을 대상으로 진행한다.

    &nbsp;
    <h3>2. 히스토리</h3>
    히스토리란 어쩌고저쩌고 하는 정의는 history manual page에서 확인하면 될 것 같고 여기에선 이게 뭔지만 간단하게 정리하도록 한다.

    shell에 히스토리 기능은 모든 쉘에 존재하는 것이 아닌, 쉘에서 제공하는 여러 옵션 기능 중 하나이다. 당장 우분투나 CentOS와 같은 리눅스 배포판이나 Mac OS X의 터미널에서 history 명령어를 치면, 다음과 같이 그동안 쉘에서 입력했던 명령어를 확인할 수 있다.
    <pre class="lang:default theme:twilight" title="history">n0fate@n0fate-MacBook-Air: ~$ history
    1  clear
    2  sudo vmmap 248
    3  ls
    4  cd
    5  ls
    6  cd Dropbox/
    7  ls
    ...[SNIP]...
    133  net use
    134  sudo port install smbclient
    135  sudo port install sambaclient
    136  sudo port install samba-client
    137  ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
    138  brew install smbclient
    139  brew install sambaclient
    140  sudo brew install sambaclient
    141  uname -a
    142  clear
    143  history
    n0fate@n0fate-MacBook-Air: ~$</pre>
    우리가 리눅스를 쓰면서 터미널에서 위/아래 방향 키로 이전/다음 명령어를 확인하는 것이 다 이러한 히스토리 정보를 기반으로 진행되는 것이다. 문제는 이러한 히스토리 정보가 명령어를 입력한 순서대로 저장되고 있으나, 해당 명령어가 언제 실행되었는지는 알지 못한다는 점이다. 라이브 포렌식에서는 단순히 명령어를 실행한 순서만을 확인할 수 있기 때문에, 메모리 분석을 하지 않는다면 해당 명령어를 실행한 정확한 시간을 판단할 수 없게 된다. 결론적으로 메모리에서 분석하면 다음과 같은 결과를 확인할 수 있다.
    <pre class="lang:default theme:twilight" title="dumping history with memory forensics">586    bash Fri Nov 15 05:31:41 2013 ls
    586    bash Fri Nov 15 05:31:41 2013 ls
    586    bash Fri Nov 15 05:31:41 2013 sudo mv *.bin ~
    586    bash Fri Nov 15 05:31:41 2013 ./osxpmem -f raw historyc.bin
    586    bash Fri Nov 15 05:31:41 2013 sudo ./osxpmem -f raw historyc.bin
    586    bash Fri Nov 15 05:31:43 2013 ls
    586    bash Fri Nov 15 05:31:41 2013 python vol.py -i ../historyc.bin -o bash_history
    586    bash Fri Nov 15 05:31:41 2013 sudo reboot
    586    bash Fri Nov 15 05:31:41 2013 ls -al
    586    bash Fri Nov 15 05:31:48 2013 tar xvf OSXPMem-RC1.tar
    586    bash Fri Nov 15 05:31:54 2013 mv OSXPMem /tmp</pre>
    각 bash 프로세스에서 실행한 명령어와 시간 정보를 확인할 수 있다. 몇몇 명령어의 시간이 동일한 이유는 뒤에 다루도록 하겠다.

    &nbsp;
    <h3>3. 히스토리 저장소</h3>
    히스토리가 어떤 정보를 가지고 있는지 확인하려면 bash 바이너리가 history 자료구조를 어떠한 알고리즘으로 관리하는지 확인해야 한다. bash 뿐만 아니라 대부분의 쉘은 오픈소스이므로, 프로그램 소스코드에서 확인할 수 있다. 프로그램 소스코드에서 히스토리를 관리하는 구조체는 다음과 같이 확인할 수 있다.
    <pre class="lang:default theme:twilight" title="history entry">typedef void * histdata_t;
    typedef struct _hist_entry {
    char *line;
    char *timestamp;
    histdata_t data;
    } HIST_ENTRY;</pre>
    <pre class="lang:default theme:twilight" title="history states">/*
     * A structure used to pass around the current state of the history.
     */
    typedef struct _hist_state {
      HIST_ENTRY **entries; /* Pointer to the entries themselves. */
      int offset;           /* The location pointer within this array. */
      int length;           /* Number of elements within this array. */
      int size;             /* Number of slots allocated to this array. */
      int flags;
    } HISTORY_STATE;</pre>
    hist_state 구조체는 각 히스토리를 총괄하여 관리하는 구조체로, 히스토리 엔트리(hist_entry)에 대한 2차원 배열 포인터와 배열 내 엔트리 갯수, 크기 등의 정보를 가지고 있다. 2차원 배열 내에는 각각의 히스토리 엔트리의 포인터 주소가 담겨져 있다. 히스토리 엔트리는 사용자가 입력한 명령어(line)와 시간정보(timestamp)를 문자열로 가지고 있다. histdata 타입 데이터는 아무런 정보를 가지고 있지 않아서 어떠한 정보인지 판단하지 않았다.

    <a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-18.43.33.png"><img class="aligncenter size-full wp-image-505" alt="Screen Shot 2013-11-29 at 18.43.33" src="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-18.43.33.png" width="732" height="368" /></a>

    &nbsp;
    <p style="text-align: center;"><strong>Figure 2. History Storage</strong></p>
    즉, 이 포인터 배열을 찾으면, 메모리에 존재하는 모든 히스토리 엔트리를 추출할 수 있다.

    &nbsp;
    <h3>4. 메모리 이미지에서 히스토리 추출</h3>
    자 그럼 이러한 데이터를 메모리에서 추출해보겠다.  일단 히스토리 데이터는 bash의 힙영역(Malloc Zone)에 위치한다. 힙 영역은 예전에 키체인 분석에서 사용된 힙 영역 식별 기법을 이용하였다.  각 히스토리 엔트리는 히스토리 엔트리 구조체와 실제 데이터가 같은 힙 영역에 위치한다는 점을 이용하여 히스토리를 패턴 기반으로 추출할 수 있다.

    <a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-18.55.14.png"><img class="aligncenter size-full wp-image-506" alt="Screen Shot 2013-11-29 at 18.55.14" src="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-18.55.14.png" width="732" height="385" /></a>

    &nbsp;
    <p style="text-align: center;"><strong>Figure 3. Dumping Bash history in Bash heap space</strong></p>
    <strong>Figure 3</strong>와 같이 특정 힙영역에서 두 개의 연속된 포인터 크기의 변수가 해당 힙영역 내의 가상 주소(0x7fdf68d00000 ~ 0x7fdf68d00000)를 가리키고 있고, 이 중 첫 번째 포인터가 가리키는 데이터의 첫 바이트가 '#'이라면, 이를 히스토리 엔트리로 판단할 수 있다. 시간 정보는 Unix Time이 문자열로 박혀 있으므로 추출 과정에서 데이터 변환 작업이 필요하다.

    &nbsp;
    <h3>5. Implementation</h3>
    위의 내용을 토대로 volafox에 'bash_history'라는 플러그인을 구현하였다. subversion에서 최신 버전을 다운받으면 사용할 수 있다. 현재 64비트 기반 Mac OS X의 메모리 이미지에서만 정상적인 덤프가 가능하나 Mountain Lion부터는 64비트 커널만 존재하므로 별 문제는 없다.
    <pre class="lang:default theme:twilight" title="history entry">n0fate@n0fate-MacBook-Air: ~/volafox$ python vol.py -i 12F37.bin -o bash_history
    [+] PID : 748, PROCESS: bash, HISTORY COUNT: 500
    [+] PID : 1119, PROCESS: bash, HISTORY COUNT: 0
    PID PROCESS TIME (UTC+0) CMD
    748 bash Wed Oct 23 10:26:47 2013 ls
    748 bash Wed Oct 23 10:26:55 2013 cp 12F37x64.overlay overlays
    748 bash Wed Oct 23 10:26:57 2013 cd overlays
    748 bash Wed Oct 23 10:26:57 2013 ls
    748 bash Thu Oct 03 06:54:22 2013 cd Documents/
    748 bash Thu Oct 03 07:24:16 2013 df -hl | grep 'disk0s2' | awk '{print $4"/"$2" free ("$5" used)"}'
    ...[SNIP]...
    748 bash Thu Oct 03 06:54:16 2013 dir
    748 bash Thu Oct 03 06:54:16 2013 ls
    748 bash Thu Oct 03 06:54:16 2013 cd sqlite3-dbx/
    748 bash Thu Oct 03 06:54:16 2013 ls
    748 bash Thu Oct 03 06:54:16 2013 clear
    748 bash Thu Oct 03 06:54:16 2013 ls
    748 bash Thu Oct 03 06:54:16 2013 history | grep sqlite3
    n0fate@n0fate-MacBook-Air: ~/volafox$</pre>
    위와 같이 각 명령어를 입력한 시간과 해당 명령어가 추출된다. 시간 정보가 함께 추출되기 때문에 사용자가 어느 시점에 명령어를 입력했는지 파악할 수 있다.

    &nbsp;
    <h3>6. Case Study</h3>
    여기에선 이슈 상황과 각 케이스에서 어떠한 결과가 도출되는지 확인해보도록 한다.
    <h4> 1. Timestamp가 같은 히스토리 엔트리가 다수 존재하는데?</h4>
    "5. Implementation"에서 추출된 히스토리 엔트리를 보면, PID 748인 프로세스의 명령어 일부가 동일한 시간에 실행된 것으로 표현되어 있다. bash 프로세스는 히스토리 파일(eg. .bash_history)을 메모리에 맵핑하여 새로운 히스토리 엔트리가 추가될 때마다 입력한 명령어를 파일에 저장한다. 그리고 프로세스를 종료한 후, 다시 실행하면 해당 파일을 메모리에 맵핑하고, 모든 명령어를 히스토리 엔트리 구조체에 맞춰 저장한다. 문제는 히스토리 파일에는 시간 정보(timestamp)가 존재하지 않기 때문에 구조체의 시간 정보를 저장하는 부분에 저장할 데이터가 없게 된다. 그리하여 bash는 해당 히스토리 엔트리에 대해서는 히스토리 파일을 메모리에 맵핑한 시점의 시간으로 일괄등록한다. 즉, 프로세스 생성시간을 기준으로 1~2초 내에 시간이 동일한 명령어 셋(set)이 있다면, 이전에 히스토리 파일에 저장된 명령어로 판단해도 된다.

    &nbsp;
    <h4>2. 사용자가 'history -c' 명령어로 히스토리를 초기화 한 경우?</h4>
    히스토리를 악의적인 목적으로 초기화한 경우, 시간 정보만 두고 입력한 명령어의 16~18바이트를 0x00으로 덮어씌운다. 즉, 디스크 분석을 수행하지 않는 이상 메모리에서는 제거된 명령어 셋을 복구할 수 없다.
    <p style="text-align: center;"><a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-19.14.17.png"><img class="size-full wp-image-507 aligncenter" alt="" src="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-19.14.17.png" width="713" height="342" /></a></p>
    <p style="text-align: center;"><strong>Figure 4. Case Study : Clearing History</strong></p>
     위 그림과 같이 해당 데이터가 와이핑(wiping)되기 때문에 일부 유실된 데이터만 확보할 수 있다. 만약에 메모리를 분석했는데 다음과 같은 결과를 확인한다면, 히스토리를 삭제 명령을 수행했다고 유추할 수 있다.

    &nbsp;
    <h4> 3.  여러 bash 프로세스 중 하나의 프로세스에서 히스토리를 초기화한 경우?</h4>
    여러 bash 프로세스 중에서 하나의 bash에서 히스토리 초기화 명령을 입력하는 경우가 있다. 예를 들어 공격자가 원격으로 쉘을 획득한 경우, 일부러 자신이 생성한 bash 프로세스에서 히스토리를 초기화 명령을 실행할 수 있을 것이다. 다음은 PID 619인 bash 프로세스에서 히스토리를 초기화하고 PID 769 프로세스를 생성하여 명령어를 몇 개 실행한 후 메모리를 분석한 결과이다.
    <p style="text-align: center;"><a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-19.24.41.png"><img class="aligncenter  wp-image-508" alt="" src="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-19.24.41.png" width="822" height="506" /></a></p>
    <p style="text-align: center;"><strong>Figure 5. Case Study: Clearing History in each bash </strong></p>
    각각의 프로세스는 별도의 히스토리 저장소를 관리/운용하기 때문에 PID 619 프로세스는 모든 명령어가 초기화 되더라도 기존의 다른 프로세스는 아무런 영향이 없다. 단, 히스토리 제거 과정에서 히스토리 파일의 데이터도 삭제되기 때문에 새롭게 생성된 프로세스는 기존에 저장된 히스토리 정보를 추출할 수 없게 된다.

    &nbsp;
    <h3>7. 결론</h3>
    메모리에서 bash의 히스토리 정보를 추출하는 방법은 쉘스크립트를 이용하여 히스토리 파일을 접근하여 수집하는 데이터 뿐만 아니라 각 명령어를 실행한 실행 정보를 획득할 수 있어서 시간 정보가 중요한 디지털 포렌식 분야에 아주 유용한 정보로 활용될 수 있다. 물론 'history -c' 명령어로 인해 메모리의 데이터도 유실될 수 있긴 하지만, 해당 명령어를 실행했는지 여부도 파악이 가능하기 때문에 가능한 많은 정보가 필요한 포렌식 분야에서는 이 또한 소중한 정보로 활용될 수 있다.

    사실 이 외에도 다양한 케이스('/dev/null 로 히스토리를 전부 저장하지 않는 경우에는 메모리에 어떠한 형태로 남는가?')가 있으나 이는 추 후 별도의 포스팅으로 다루도록 하겠다.

    &nbsp;
    <h3 style="text-align: left;">8. Reference</h3>
    <p itemprop="name">MoVP II - 3.3 - Automated Linux/Android Bash History Scanning, http://volatility-labs.blogspot.kr/2013/05/movp-ii-33-automated-linuxandroid-bash.html</p>
    <p itemprop="name">Extracting typing history in Unix Memory Image, http://forensicinsight.org</p>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<h3>1. 글을 시작하며..</h3>
<p>최근에 이런저런 사건사고가 터지면서 디지털 포렌식이라는 용어가 언론에 자주 나타나고 있다. 최근의 사용자가 아날로그보다 디지털에 점점 의존도가 높아지면서 대부분의 범죄 사고 분석의 주요 증거물이 됨으로 더더욱 중요도가 높아지면서 생겨난 자연스러운 현상이다.</p>
<p>침해사고 대응 시나 기업 내 정보 유출 사고에서는 어떠한 사용자(Who)가 어떠한 시스템(Where)을 어느 시점(When)에 어떤 행동(What)을 어떻게(How) 제어했는가가 중요한 요소가 된다.  디지털 수사관은 이러한 정보를 효과적으로 얻기 위해 다양한 디지털 증거 수집 도구를 활용하여 데이터를 수집한다.</p>
<p>디지털 증거를 수집하는 방법은 사건 현장에서 지속적으로 변경되는 데이터의 현재 상태를 신속하게 수집하기 위한 라이브 포렌식(Live Forensics)와 시스템의 현재 상태 수집, 라이브 데이터의 무결성 보장, 추 후 디스크 분석과 연계를 위한 메모리 포렌식(Memory Forensics), 마지막으로 가장 많은 디지털 증거를 가지며, 가장 많은 분석 시간이 할애되는 디스크 포렌식(Disk Forensics)로 크게 3가지로 나눠볼 수 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-9.47.45-PM.png"><img class="aligncenter size-full wp-image-513" alt="Screen Shot 2013-11-29 at 9.47.45 PM" src="{{ site.baseurl }}/assets/Screen-Shot-2013-11-29-at-9.47.45-PM.png" width="648" height="359" /></a></p>
<p style="text-align: center;"><strong>Figure 1. 각 기술의 포용 범위</strong></p>
<p>최근에는 메모리 포렌식 기술이 라이브 포렌식에서 수집하는 대부분의 디지털 증거를 포용하면서 이러한 증거의 무결성 보장을 위한 다양한 기능을 추가 제공하고, 라이브 포렌식에 비해 상대적으로 높은 시스템 무결성을 보장하다보니 메모리와 디스크 포렌식만 수행하는 경우도 종종 있다.</p>
<p>특히, 메모리 분석에서는 스크립트 수준의 라이브 데이터 분석보다 더 많은 정보를 산출해 낼 수 있기 때문에 필자는 주변 분석가들에게 시스템 분석 과정에서 꼭 메모리 분석을 수행하기를 권고하고 있다.</p>
<p>오늘 포스팅에서는 단순 라이브 데이터 스크립트에서는 수집할 수 없는 쉘 히스토리(shell history)의 시간 정보(timestamp)를 메모리에서 추출하는 방법에 대해 작성하고자 한다. 여기에선 여러 쉘 중 가장 주변에서 접하기 쉬우면서 발전된 쉘인 bash(bourne-again shell)을 대상으로 진행한다.</p>
<p>&nbsp;</p>
<h3>2. 히스토리</h3>
<p>히스토리란 어쩌고저쩌고 하는 정의는 history manual page에서 확인하면 될 것 같고 여기에선 이게 뭔지만 간단하게 정리하도록 한다.</p>
<p>shell에 히스토리 기능은 모든 쉘에 존재하는 것이 아닌, 쉘에서 제공하는 여러 옵션 기능 중 하나이다. 당장 우분투나 CentOS와 같은 리눅스 배포판이나 Mac OS X의 터미널에서 history 명령어를 치면, 다음과 같이 그동안 쉘에서 입력했던 명령어를 확인할 수 있다.</p>
<pre class="lang:default theme:twilight" title="history">n0fate@n0fate-MacBook-Air: ~$ history
1  clear
2  sudo vmmap 248
3  ls
4  cd
5  ls
6  cd Dropbox/
7  ls
...[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][SNIP]...
133  net use
134  sudo port install smbclient
135  sudo port install sambaclient
136  sudo port install samba-client
137  ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
138  brew install smbclient
139  brew install sambaclient
140  sudo brew install sambaclient
141  uname -a
142  clear
143  history
n0fate@n0fate-MacBook-Air: ~$</pre>
<p>우리가 리눅스를 쓰면서 터미널에서 위/아래 방향 키로 이전/다음 명령어를 확인하는 것이 다 이러한 히스토리 정보를 기반으로 진행되는 것이다. 문제는 이러한 히스토리 정보가 명령어를 입력한 순서대로 저장되고 있으나, 해당 명령어가 언제 실행되었는지는 알지 못한다는 점이다. 라이브 포렌식에서는 단순히 명령어를 실행한 순서만을 확인할 수 있기 때문에, 메모리 분석을 하지 않는다면 해당 명령어를 실행한 정확한 시간을 판단할 수 없게 된다. 결론적으로 메모리에서 분석하면 다음과 같은 결과를 확인할 수 있다.</p>
<pre class="lang:default theme:twilight" title="dumping history with memory forensics">586    bash Fri Nov 15 05:31:41 2013 ls
586    bash Fri Nov 15 05:31:41 2013 ls
586    bash Fri Nov 15 05:31:41 2013 sudo mv *.bin ~
586    bash Fri Nov 15 05:31:41 2013 ./osxpmem -f raw historyc.bin
586    bash Fri Nov 15 05:31:41 2013 sudo ./osxpmem -f raw historyc.bin
586    bash Fri Nov 15 05:31:43 2013 ls
586    bash Fri Nov 15 05:31:41 2013 python vol.py -i ../historyc.bin -o bash_history
586    bash Fri Nov 15 05:31:41 2013 sudo reboot
586    bash Fri Nov 15 05:31:41 2013 ls -al
586    bash Fri Nov 15 05:31:48 2013 tar xvf OSXPMem-RC1.tar
586    bash Fri Nov 15 05:31:54 2013 mv OSXPMem /tmp</pre>
<p>각 bash 프로세스에서 실행한 명령어와 시간 정보를 확인할 수 있다. 몇몇 명령어의 시간이 동일한 이유는 뒤에 다루도록 하겠다.</p>
<p>&nbsp;</p>
<h3>3. 히스토리 저장소</h3>
<p>히스토리가 어떤 정보를 가지고 있는지 확인하려면 bash 바이너리가 history 자료구조를 어떠한 알고리즘으로 관리하는지 확인해야 한다. bash 뿐만 아니라 대부분의 쉘은 오픈소스이므로, 프로그램 소스코드에서 확인할 수 있다. 프로그램 소스코드에서 히스토리를 관리하는 구조체는 다음과 같이 확인할 수 있다.</p>
<pre class="lang:default theme:twilight" title="history entry">typedef void * histdata_t;
typedef struct _hist_entry {
char *line;
char *timestamp;
histdata_t data;
} HIST_ENTRY;</pre>
<pre class="lang:default theme:twilight" title="history states">/*
 * A structure used to pass around the current state of the history.
 */
typedef struct _hist_state {
  HIST_ENTRY **entries; /* Pointer to the entries themselves. */
  int offset;           /* The location pointer within this array. */
  int length;           /* Number of elements within this array. */
  int size;             /* Number of slots allocated to this array. */
  int flags;
} HISTORY_STATE;</pre>
<p>hist_state 구조체는 각 히스토리를 총괄하여 관리하는 구조체로, 히스토리 엔트리(hist_entry)에 대한 2차원 배열 포인터와 배열 내 엔트리 갯수, 크기 등의 정보를 가지고 있다. 2차원 배열 내에는 각각의 히스토리 엔트리의 포인터 주소가 담겨져 있다. 히스토리 엔트리는 사용자가 입력한 명령어(line)와 시간정보(timestamp)를 문자열로 가지고 있다. histdata 타입 데이터는 아무런 정보를 가지고 있지 않아서 어떠한 정보인지 판단하지 않았다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-18.43.33.png"><img class="aligncenter size-full wp-image-505" alt="Screen Shot 2013-11-29 at 18.43.33" src="{{ site.baseurl }}/assets/Screen-Shot-2013-11-29-at-18.43.33.png" width="732" height="368" /></a></p>
<p>&nbsp;</p>
<p style="text-align: center;"><strong>Figure 2. History Storage</strong></p>
<p>즉, 이 포인터 배열을 찾으면, 메모리에 존재하는 모든 히스토리 엔트리를 추출할 수 있다.</p>
<p>&nbsp;</p>
<h3>4. 메모리 이미지에서 히스토리 추출</h3>
<p>자 그럼 이러한 데이터를 메모리에서 추출해보겠다.  일단 히스토리 데이터는 bash의 힙영역(Malloc Zone)에 위치한다. 힙 영역은 예전에 키체인 분석에서 사용된 힙 영역 식별 기법을 이용하였다.  각 히스토리 엔트리는 히스토리 엔트리 구조체와 실제 데이터가 같은 힙 영역에 위치한다는 점을 이용하여 히스토리를 패턴 기반으로 추출할 수 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-18.55.14.png"><img class="aligncenter size-full wp-image-506" alt="Screen Shot 2013-11-29 at 18.55.14" src="{{ site.baseurl }}/assets/Screen-Shot-2013-11-29-at-18.55.14.png" width="732" height="385" /></a></p>
<p>&nbsp;</p>
<p style="text-align: center;"><strong>Figure 3. Dumping Bash history in Bash heap space</strong></p>
<p><strong>Figure 3</strong>와 같이 특정 힙영역에서 두 개의 연속된 포인터 크기의 변수가 해당 힙영역 내의 가상 주소(0x7fdf68d00000 ~ 0x7fdf68d00000)를 가리키고 있고, 이 중 첫 번째 포인터가 가리키는 데이터의 첫 바이트가 '#'이라면, 이를 히스토리 엔트리로 판단할 수 있다. 시간 정보는 Unix Time이 문자열로 박혀 있으므로 추출 과정에서 데이터 변환 작업이 필요하다.</p>
<p>&nbsp;</p>
<h3>5. Implementation</h3>
<p>위의 내용을 토대로 volafox에 'bash_history'라는 플러그인을 구현하였다. subversion에서 최신 버전을 다운받으면 사용할 수 있다. 현재 64비트 기반 Mac OS X의 메모리 이미지에서만 정상적인 덤프가 가능하나 Mountain Lion부터는 64비트 커널만 존재하므로 별 문제는 없다.</p>
<pre class="lang:default theme:twilight" title="history entry">n0fate@n0fate-MacBook-Air: ~/volafox$ python vol.py -i 12F37.bin -o bash_history
[+] PID : 748, PROCESS: bash, HISTORY COUNT: 500
[+] PID : 1119, PROCESS: bash, HISTORY COUNT: 0
PID PROCESS TIME (UTC+0) CMD
748 bash Wed Oct 23 10:26:47 2013 ls
748 bash Wed Oct 23 10:26:55 2013 cp 12F37x64.overlay overlays
748 bash Wed Oct 23 10:26:57 2013 cd overlays
748 bash Wed Oct 23 10:26:57 2013 ls
748 bash Thu Oct 03 06:54:22 2013 cd Documents/
748 bash Thu Oct 03 07:24:16 2013 df -hl | grep 'disk0s2' | awk '{print $4"/"$2" free ("$5" used)"}'
...[SNIP]...
748 bash Thu Oct 03 06:54:16 2013 dir
748 bash Thu Oct 03 06:54:16 2013 ls
748 bash Thu Oct 03 06:54:16 2013 cd sqlite3-dbx/
748 bash Thu Oct 03 06:54:16 2013 ls
748 bash Thu Oct 03 06:54:16 2013 clear
748 bash Thu Oct 03 06:54:16 2013 ls
748 bash Thu Oct 03 06:54:16 2013 history | grep sqlite3
n0fate@n0fate-MacBook-Air: ~/volafox$</pre>
<p>위와 같이 각 명령어를 입력한 시간과 해당 명령어가 추출된다. 시간 정보가 함께 추출되기 때문에 사용자가 어느 시점에 명령어를 입력했는지 파악할 수 있다.</p>
<p>&nbsp;</p>
<h3>6. Case Study</h3>
<p>여기에선 이슈 상황과 각 케이스에서 어떠한 결과가 도출되는지 확인해보도록 한다.</p>
<h4> 1. Timestamp가 같은 히스토리 엔트리가 다수 존재하는데?</h4>
<p>"5. Implementation"에서 추출된 히스토리 엔트리를 보면, PID 748인 프로세스의 명령어 일부가 동일한 시간에 실행된 것으로 표현되어 있다. bash 프로세스는 히스토리 파일(eg. .bash_history)을 메모리에 맵핑하여 새로운 히스토리 엔트리가 추가될 때마다 입력한 명령어를 파일에 저장한다. 그리고 프로세스를 종료한 후, 다시 실행하면 해당 파일을 메모리에 맵핑하고, 모든 명령어를 히스토리 엔트리 구조체에 맞춰 저장한다. 문제는 히스토리 파일에는 시간 정보(timestamp)가 존재하지 않기 때문에 구조체의 시간 정보를 저장하는 부분에 저장할 데이터가 없게 된다. 그리하여 bash는 해당 히스토리 엔트리에 대해서는 히스토리 파일을 메모리에 맵핑한 시점의 시간으로 일괄등록한다. 즉, 프로세스 생성시간을 기준으로 1~2초 내에 시간이 동일한 명령어 셋(set)이 있다면, 이전에 히스토리 파일에 저장된 명령어로 판단해도 된다.</p>
<p>&nbsp;</p>
<h4>2. 사용자가 'history -c' 명령어로 히스토리를 초기화 한 경우?</h4>
<p>히스토리를 악의적인 목적으로 초기화한 경우, 시간 정보만 두고 입력한 명령어의 16~18바이트를 0x00으로 덮어씌운다. 즉, 디스크 분석을 수행하지 않는 이상 메모리에서는 제거된 명령어 셋을 복구할 수 없다.</p>
<p style="text-align: center;"><a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-19.14.17.png"><img class="size-full wp-image-507 aligncenter" alt="" src="{{ site.baseurl }}/assets/Screen-Shot-2013-11-29-at-19.14.17.png" width="713" height="342" /></a></p>
<p style="text-align: center;"><strong>Figure 4. Case Study : Clearing History</strong></p>
<p> 위 그림과 같이 해당 데이터가 와이핑(wiping)되기 때문에 일부 유실된 데이터만 확보할 수 있다. 만약에 메모리를 분석했는데 다음과 같은 결과를 확인한다면, 히스토리를 삭제 명령을 수행했다고 유추할 수 있다.</p>
<p>&nbsp;</p>
<h4> 3.  여러 bash 프로세스 중 하나의 프로세스에서 히스토리를 초기화한 경우?</h4>
<p>여러 bash 프로세스 중에서 하나의 bash에서 히스토리 초기화 명령을 입력하는 경우가 있다. 예를 들어 공격자가 원격으로 쉘을 획득한 경우, 일부러 자신이 생성한 bash 프로세스에서 히스토리를 초기화 명령을 실행할 수 있을 것이다. 다음은 PID 619인 bash 프로세스에서 히스토리를 초기화하고 PID 769 프로세스를 생성하여 명령어를 몇 개 실행한 후 메모리를 분석한 결과이다.</p>
<p style="text-align: center;"><a href="http://forensic.n0fate.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-29-at-19.24.41.png"><img class="aligncenter  wp-image-508" alt="" src="{{ site.baseurl }}/assets/Screen-Shot-2013-11-29-at-19.24.41.png" width="822" height="506" /></a></p>
<p style="text-align: center;"><strong>Figure 5. Case Study: Clearing History in each bash </strong></p>
<p>각각의 프로세스는 별도의 히스토리 저장소를 관리/운용하기 때문에 PID 619 프로세스는 모든 명령어가 초기화 되더라도 기존의 다른 프로세스는 아무런 영향이 없다. 단, 히스토리 제거 과정에서 히스토리 파일의 데이터도 삭제되기 때문에 새롭게 생성된 프로세스는 기존에 저장된 히스토리 정보를 추출할 수 없게 된다.</p>
<p>&nbsp;</p>
<h3>7. 결론</h3>
<p>메모리에서 bash의 히스토리 정보를 추출하는 방법은 쉘스크립트를 이용하여 히스토리 파일을 접근하여 수집하는 데이터 뿐만 아니라 각 명령어를 실행한 실행 정보를 획득할 수 있어서 시간 정보가 중요한 디지털 포렌식 분야에 아주 유용한 정보로 활용될 수 있다. 물론 'history -c' 명령어로 인해 메모리의 데이터도 유실될 수 있긴 하지만, 해당 명령어를 실행했는지 여부도 파악이 가능하기 때문에 가능한 많은 정보가 필요한 포렌식 분야에서는 이 또한 소중한 정보로 활용될 수 있다.</p>
<p>사실 이 외에도 다양한 케이스('/dev/null 로 히스토리를 전부 저장하지 않는 경우에는 메모리에 어떠한 형태로 남는가?')가 있으나 이는 추 후 별도의 포스팅으로 다루도록 하겠다.</p>
<p>&nbsp;</p>
<h3 style="text-align: left;">8. Reference</h3>
<p itemprop="name">MoVP II - 3.3 - Automated Linux/Android Bash History Scanning, http://volatility-labs.blogspot.kr/2013/05/movp-ii-33-automated-linuxandroid-bash.html</p>
<p itemprop="name">Extracting typing history in Unix Memory Image, http://forensicinsight.org</p>
<p>[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
