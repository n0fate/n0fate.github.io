---
layout: post
title: Time Machine Forensics [1]
date: 2010-12-02 17:26:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2010/12/time-machine-forensics-1.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/7543021555065078087
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '860'
  fusion_builder_content_backup: |-
    Mac OS X 는 '<a href="http://en.wikipedia.org/wiki/Time_Machine_(software)">Time Machine</a>'이라고 불리는 디스크 백업 유틸리티를 내장하고 있다.<br /><br /><div><a href="http://2.bp.blogspot.com/_KYUsDAgl5oc/TPdNjwbMJgI/AAAAAAAAAHc/mJCxKEOy75U/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA+2010-12-02+4.39.23+PM.png" imageanchor="1"><img border="0" height="265" src="http://2.bp.blogspot.com/_KYUsDAgl5oc/TPdNjwbMJgI/AAAAAAAAAHc/mJCxKEOy75U/s400/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA+2010-12-02+4.39.23+PM.png" width="400"></a></div><br /><br />스노우 레오파드를 처음 설치했다면, 타임머신은 기본적으로 비 활성화 되어있다. 타임머신은 USB, Firewire나 AFP(Apple File-transfer Protocol)을 지원하는 무선 네트워크를 가진 장치(타임캡슐)를 백업 디스크로 선택할 수 있도록 구성되어 있다.<br /><br />포렌식 관점에서 타임머신의 구성을 확인하기 위해 하나의 USB 외장 디스크에 가상 머신을 생성하여 연결하였고, '백업 디스크로 지정'하여 총 6회 정도의 백업을 수행하였다. 백업 목적의 디스크는 처음에 디스크로 지정되면, 타임머신에서 사용할 수 있도록 디스크 이미지를 프로그램에 맞게 재 포맷을 수행한다.<br /><br />이미징한 디스크는 Mac에서 쉽게 접근할 수 있도록 구성되어 있다. 처음 포맷을 수행할 때 타임 머신 관리자는 파일 시스템을 HFS Plus로 포맷한다. 내부에는 맥 운영체제로 인해 생성되는 파일 시스템 이벤트관련 디렉터리(.fseventsd)와 인덱스를 위한 디렉터리(.Spotlight-V100)이 <s>존재하며, '.HFS+ Private Directory Data_'와 '___HFS+ Private Data' 디렉터리를 추가적으로 가지고</s> 있다.<br />------------------------------------------------------------------------------------------------<br /><br /><span>forensic-iMac:Time Machine Backups n0fate$ ls -al</span><br /><span>total 16</span><br /><span>drwxrwxr-x  5 n0fate  staff   408 Dec  2 16:58 .</span><br /><span>drwxrwxrwt@ 5 root    admin   170 Dec  2 16:57 ..</span><br /><span>-rw-r--r--@ 1 n0fate  staff  6148 Dec  2 16:58 .DS_Store</span><br /><span>drwx------  3 n0fate  staff   102 Oct 23 18:47 .Spotlight-V100</span><br /><span>d-wx-wx-wt  3 n0fate  staff   102 Dec  2 16:57 .Trashes</span><br /><span>-rw-r--r--@ 1 n0fate  staff     0 Dec  2 16:57 .com.apple.timemachine.donotpresent</span><br /><span>drwx------  2 n0fate  staff   102 Dec  2 16:58 .fseventsd</span><br /><span>drwxr-xr-x+ 3 n0fate  staff   136 Dec  2 12:38 Backups.backupdb</span><br /><span>forensic-iMac:Time Machine Backups n0fate$ </span><br /><div>------------------------------------------------------------------------------------------------</div><div>우리가 원하는 실질적인 데이터는 'Backups.backupdb' 디렉터리에 존재할 것이니 해당 디렉터리에 직접 접근하면, 타임 머신을 연결한 시스템 이름의 디렉터리를 확인할 수 있다.</div><div>------------------------------------------------------------------------------------------------</div><div><div><span>forensic-iMac:Backups.backupdb n0fate$ ls -al</span></div><div><span>total 7</span></div><div><span>drwxr-xr-x+ 3 n0fate  staff  136 Dec  2 12:38 .</span></div><div><span>drwxrwxr-x  5 n0fate  staff  408 Dec  2 16:58 ..</span></div><div><span>drwxr-xr-x@ 9 n0fate  staff  340 Dec  1 01:02 n0fate's MacBook Pro</span></div><div><span>forensic-iMac:Backups.backupdb n0fate$ </span></div></div><div>------------------------------------------------------------------------------------------------</div><div>디렉터리에 접근하면 백업한 시점을 디렉터리 이름으로 설정하여 독립적인 백업 디렉터리를 생성하고,  가장 최근에 백업한 디렉터리를 'Lastest'로 심볼릭 링크를 걸어 접근할 수 있도록 되어 있다.</div><div>------------------------------------------------------------------------------------------------</div><div><div><span>forensic-iMac:n0fate's MacBook Pro n0fate$ ls -al</span></div><div><span>total 8</span></div><div><span>drwxr-xr-x@ 9 n0fate  staff  340 Dec  1 01:02 .</span></div><div><span>drwxr-xr-x+ 3 n0fate  staff  136 Dec  2 12:38 ..</span></div><div><span>drwxr-xr-x@ 3 n0fate  staff  204 Oct 23 20:02 2010-10-23-200209</span></div><div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov  6 22:44 2010-11-06-224436</span></div><div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov  7 13:52 2010-11-07-135232</span></div><div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov  7 14:41 2010-11-07-144113</span></div><div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov  7 14:58 2010-11-07-145853</span></div><div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov 22 00:19 2010-11-22-001900</span></div><div><span>drwxr-xr-x@ 3 n0fate  staff  204 Dec  1 01:01 2010-12-01-010147</span></div><div><span>lrwxr-xr-x  1 n0fate  staff   17 Dec  1 01:01 Latest -> 2010-12-01-010147</span></div><div><span>forensic-iMac:n0fate's MacBook Pro n0fate$ </span></div></div><div>------------------------------------------------------------------------------------------------</div><div>그리고 내부로 접근하면 해당 시점에 Snow Leopard가 설치된 디스크를 이름으로 한 디렉터리가 나타나고 한 스텝 더 진행하면 타임 머신을 설정한 시스템의 루트 디렉터리와 동일한 정보를 지닌 파일이 나타난다.</div><div>------------------------------------------------------------------------------------------------</div><div><div><span>forensic-iMac:2010-12-01-010147 n0fate$ ls</span></div><div><span>SnowLeopardHDD</span></div><div><span>forensic-iMac:2010-12-01-010147 n0fate$ cd SnowLeopardHDD/</span></div><div><span>forensic-iMac:SnowLeopardHDD n0fate$ ls</span></div><div><span>Applications</span></div><div><span>Developer</span></div><div><span>...</span></div><div><span>usr</span></div><div><span>var</span></div><div><span>사용 설명서와 정보</span></div><div><span>forensic-iMac:SnowLeopardHDD n0fate$ </span></div></div><div>------------------------------------------------------------------------------------------------</div><div>중요한 사실은 파일의 내용이 수정되지 않더라도 다음 타임 머신 백업 때 해당 파일을 다시 복사해서 보관하는 구조로 되어있다는 점이다. 이러한 사실은 타임 머신 백업 디스크에 한번이라도 백업을 수행했다거나, 한 시점의 디렉터리 구조가 살아 있다고 한다면, 주 시스템의 무결성을 해치지 않고서 논리적 관점에서 해당 디스크 정보를 획득한 것과 같은 효과를 얻을 수 있음을 의미한다.</div><div><br /></div><div>타임머신은 한 번만 설정해두면, 자동으로 디스크의 변경 사항을 정해진 시간에 따라 백업을 수행하도록 되어있다. 이러한 특징을 이용하면, 현재 상태의 디스크에서 확인할 수 없는 삭제 파일에 대한 메타데이터 정보 및 파일의 내용을 타임 머신 디스크를 통해 확인할 수 있기 때문에, 삭제된 파일에 접근한다는 측면에서 포렌식 적으로 유용한 정보가 될 수 있다.</div><div><br /></div><div>또 다른 관점에서, 만약 해당 시스템의 무결성을 고려하지 않고 논리적으로 데이터를 수집하고 싶다면, 별도의 스크립트를 사용할 필요 없이 외장 디스크만을 연결하여 디스크의 전체 내용을 논리적으로 수집하는 것도 생각해볼 수 있다. 이러한 수집을 콘솔에서 수행하기 위해서는 다음 명령어를 입력하면 된다.</div><div>------------------------------------------------------------------------------------------------</div><div><span>"/System/Library/CoreServices/backupd.bundle/Contents/Resour
    ces/backupd-helper &"</span></div><div>---------
    ---------------------------------------------------------------------------------------</div><div>현재까지 확인한 부분은 이정도이다. 단 위에서 줄로 그은 몇몇 디렉터리가 윈도우의 Mac Drive 어플리케이션에서만 보이는 관계로 저 부분이 본래 존재하는지에 대한 것을 추가적으로 확인할 필요가 있다. 이러한 부분에 대해서는 좀더 삽질해보고 포스팅하도록 하겠다. :)</div><div>n0fate's Forensic Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>Mac OS X 는 '<a href="http://en.wikipedia.org/wiki/Time_Machine_(software)">Time Machine</a>'이라고 불리는 디스크 백업 유틸리티를 내장하고 있다.</p>
<div><a href="http://2.bp.blogspot.com/_KYUsDAgl5oc/TPdNjwbMJgI/AAAAAAAAAHc/mJCxKEOy75U/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA+2010-12-02+4.39.23+PM.png" imageanchor="1"><img border="0" height="265" src="{{ site.baseurl }}/assets/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA+2010-12-02+4.39.23+PM.png" width="400" /></a></div>
<p>스노우 레오파드를 처음 설치했다면, 타임머신은 기본적으로 비 활성화 되어있다. 타임머신은 USB, Firewire나 AFP(Apple File-transfer Protocol)을 지원하는 무선 네트워크를 가진 장치(타임캡슐)를 백업 디스크로 선택할 수 있도록 구성되어 있다.</p>
<p>포렌식 관점에서 타임머신의 구성을 확인하기 위해 하나의 USB 외장 디스크에 가상 머신을 생성하여 연결하였고, '백업 디스크로 지정'하여 총 6회 정도의 백업을 수행하였다. 백업 목적의 디스크는 처음에 디스크로 지정되면, 타임머신에서 사용할 수 있도록 디스크 이미지를 프로그램에 맞게 재 포맷을 수행한다.</p>
<p>이미징한 디스크는 Mac에서 쉽게 접근할 수 있도록 구성되어 있다. 처음 포맷을 수행할 때 타임 머신 관리자는 파일 시스템을 HFS Plus로 포맷한다. 내부에는 맥 운영체제로 인해 생성되는 파일 시스템 이벤트관련 디렉터리(.fseventsd)와 인덱스를 위한 디렉터리(.Spotlight-V100)이 <s>존재하며, '.HFS+ Private Directory Data_'와 '___HFS+ Private Data' 디렉터리를 추가적으로 가지고</s> 있다.<br />------------------------------------------------------------------------------------------------</p>
<p><span>forensic-iMac:Time Machine Backups n0fate$ ls -al</span><br /><span>total 16</span><br /><span>drwxrwxr-x  5 n0fate  staff   408 Dec  2 16:58 .</span><br /><span>drwxrwxrwt@ 5 root    admin   170 Dec  2 16:57 ..</span><br /><span>-rw-r--r--@ 1 n0fate  staff  6148 Dec  2 16:58 .DS_Store</span><br /><span>drwx------  3 n0fate  staff   102 Oct 23 18:47 .Spotlight-V100</span><br /><span>d-wx-wx-wt  3 n0fate  staff   102 Dec  2 16:57 .Trashes</span><br /><span>-rw-r--r--@ 1 n0fate  staff     0 Dec  2 16:57 .com.apple.timemachine.donotpresent</span><br /><span>drwx------  2 n0fate  staff   102 Dec  2 16:58 .fseventsd</span><br /><span>drwxr-xr-x+ 3 n0fate  staff   136 Dec  2 12:38 Backups.backupdb</span><br /><span>forensic-iMac:Time Machine Backups n0fate$ </span>
<div>------------------------------------------------------------------------------------------------</div>
<div>우리가 원하는 실질적인 데이터는 'Backups.backupdb' 디렉터리에 존재할 것이니 해당 디렉터리에 직접 접근하면, 타임 머신을 연결한 시스템 이름의 디렉터리를 확인할 수 있다.</div>
<div>------------------------------------------------------------------------------------------------</div>
<div>
<div><span>forensic-iMac:Backups.backupdb n0fate$ ls -al</span></div>
<div><span>total 7</span></div>
<div><span>drwxr-xr-x+ 3 n0fate  staff  136 Dec  2 12:38 .</span></div>
<div><span>drwxrwxr-x  5 n0fate  staff  408 Dec  2 16:58 ..</span></div>
<div><span>drwxr-xr-x@ 9 n0fate  staff  340 Dec  1 01:02 n0fate's MacBook Pro</span></div>
<div><span>forensic-iMac:Backups.backupdb n0fate$ </span></div>
</div>
<div>------------------------------------------------------------------------------------------------</div>
<div>디렉터리에 접근하면 백업한 시점을 디렉터리 이름으로 설정하여 독립적인 백업 디렉터리를 생성하고,  가장 최근에 백업한 디렉터리를 'Lastest'로 심볼릭 링크를 걸어 접근할 수 있도록 되어 있다.</div>
<div>------------------------------------------------------------------------------------------------</div>
<div>
<div><span>forensic-iMac:n0fate's MacBook Pro n0fate$ ls -al</span></div>
<div><span>total 8</span></div>
<div><span>drwxr-xr-x@ 9 n0fate  staff  340 Dec  1 01:02 .</span></div>
<div><span>drwxr-xr-x+ 3 n0fate  staff  136 Dec  2 12:38 ..</span></div>
<div><span>drwxr-xr-x@ 3 n0fate  staff  204 Oct 23 20:02 2010-10-23-200209</span></div>
<div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov  6 22:44 2010-11-06-224436</span></div>
<div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov  7 13:52 2010-11-07-135232</span></div>
<div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov  7 14:41 2010-11-07-144113</span></div>
<div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov  7 14:58 2010-11-07-145853</span></div>
<div><span>drwxr-xr-x@ 3 n0fate  staff  204 Nov 22 00:19 2010-11-22-001900</span></div>
<div><span>drwxr-xr-x@ 3 n0fate  staff  204 Dec  1 01:01 2010-12-01-010147</span></div>
<div><span>lrwxr-xr-x  1 n0fate  staff   17 Dec  1 01:01 Latest -> 2010-12-01-010147</span></div>
<div><span>forensic-iMac:n0fate's MacBook Pro n0fate$ </span></div>
</div>
<div>------------------------------------------------------------------------------------------------</div>
<div>그리고 내부로 접근하면 해당 시점에 Snow Leopard가 설치된 디스크를 이름으로 한 디렉터리가 나타나고 한 스텝 더 진행하면 타임 머신을 설정한 시스템의 루트 디렉터리와 동일한 정보를 지닌 파일이 나타난다.</div>
<div>------------------------------------------------------------------------------------------------</div>
<div>
<div><span>forensic-iMac:2010-12-01-010147 n0fate$ ls</span></div>
<div><span>SnowLeopardHDD</span></div>
<div><span>forensic-iMac:2010-12-01-010147 n0fate$ cd SnowLeopardHDD/</span></div>
<div><span>forensic-iMac:SnowLeopardHDD n0fate$ ls</span></div>
<div><span>Applications</span></div>
<div><span>Developer</span></div>
<div><span>...</span></div>
<div><span>usr</span></div>
<div><span>var</span></div>
<div><span>사용 설명서와 정보</span></div>
<div><span>forensic-iMac:SnowLeopardHDD n0fate$ </span></div>
</div>
<div>------------------------------------------------------------------------------------------------</div>
<div>중요한 사실은 파일의 내용이 수정되지 않더라도 다음 타임 머신 백업 때 해당 파일을 다시 복사해서 보관하는 구조로 되어있다는 점이다. 이러한 사실은 타임 머신 백업 디스크에 한번이라도 백업을 수행했다거나, 한 시점의 디렉터리 구조가 살아 있다고 한다면, 주 시스템의 무결성을 해치지 않고서 논리적 관점에서 해당 디스크 정보를 획득한 것과 같은 효과를 얻을 수 있음을 의미한다.</div>
<div></div>
<div>타임머신은 한 번만 설정해두면, 자동으로 디스크의 변경 사항을 정해진 시간에 따라 백업을 수행하도록 되어있다. 이러한 특징을 이용하면, 현재 상태의 디스크에서 확인할 수 없는 삭제 파일에 대한 메타데이터 정보 및 파일의 내용을 타임 머신 디스크를 통해 확인할 수 있기 때문에, 삭제된 파일에 접근한다는 측면에서 포렌식 적으로 유용한 정보가 될 수 있다.</div>
<div></div>
<div>또 다른 관점에서, 만약 해당 시스템의 무결성을 고려하지 않고 논리적으로 데이터를 수집하고 싶다면, 별도의 스크립트를 사용할 필요 없이 외장 디스크만을 연결하여 디스크의 전체 내용을 논리적으로 수집하는 것도 생각해볼 수 있다. 이러한 수집을 콘솔에서 수행하기 위해서는 다음 명령어를 입력하면 된다.</div>
<div>------------------------------------------------------------------------------------------------</div>
<div><span>"/System/Library/CoreServices/backupd.bundle/Contents/Resour<br />
ces/backupd-helper &"</span></div>
<div>---------<br />
---------------------------------------------------------------------------------------</div>
<div>현재까지 확인한 부분은 이정도이다. 단 위에서 줄로 그은 몇몇 디렉터리가 윈도우의 Mac Drive 어플리케이션에서만 보이는 관계로 저 부분이 본래 존재하는지에 대한 것을 추가적으로 확인할 필요가 있다. 이러한 부분에 대해서는 좀더 삽질해보고 포스팅하도록 하겠다. :)</div>
<div>n0fate's Forensic Space :)</div>
