---
layout: post
title: 'Keychain Forensics : Part II'
date: 2013-06-06 22:52:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2013/06/keychain-forensic-part-ii.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/2066483451539564937
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1470'
  fusion_builder_content_backup: '<br /><div><br />이전 과정에서는 키체인에 대한 소개와 메모리 이미지 수집
    방법을 알아보았다. 그 다음 단계로 키체인 파일이 저장되는 위치를 알아보고, 파일을 수집하는 방법을 고민해봐야한다.</div><div><br
    /></div><div>키체인 파일은 어디에 저장이 될까? 키체인 파일은 크게 다음 3 곳에 저장되는 것으로 알려져 있다.</div><ul><li>~/Library/Keychain/login.keychain
    : Mac OS X는 각 사용자마다 고유의 키체인 파일을 저장한다. 디지털 포렌식 관점에서 가장 유용한 정보를 담고 있는 파일로, 사용자가
    생성한 모든 기밀 정보를 보관하고 있다.</li><li>/Library/Keychain/System.keychain : 시스템 키체인 파일로
    운영체제마다 하나의 파일을 가지고 있다. 이 파일에는 애플스토어의 애플리케이션을 인증하기 위한 애플의 인증서와 같이 시스템 운영체제 필요한
    키나 인증서를 가지고 있다.</li><li>/Network/Library/Keychain : 현재까지 저자의 환경에서 확인된 사례는 없으나,
    소규모 네트워크에서 키체인을 네트워크로 공유하여 사용함으로, 특정 서비스를 개별 인증할 필요없이 접근하기 위해 제공하는 파일로 판단된다.</li></ul>키체인
    파일을 수집하는 방법을 생각해봐야 한다. 대부분의 포렌식 케이스에서는 디스크 이미지를 수집하기 때문에 해당 경로에서 파일을 추출해내면 되지만,
    최근 수사영장에서 증거 수집의 범위를 좁혀나가는 상황에서는 특정 파일만 수집하여 분석하는 것을 요구하고 있다. 이러한 상황을 고려하여 논리적인
    데이터를 추출 방법도 고민해 봐야 한다. Mac OS X에서 논리적으로 데이터를 수집하는 방법은 다음과 같다.<br /><ol><li>파일
    복사를 이용한 데이터 수집 : 가장 평범하면서도 가장 고전적인 방법으로, 그냥 외부 저장장치를 연결하고 해당 파일을 복사하는 방법이다. 어떻게
    보면 가장 정확하게 데이터를 수집할 수 있는 방법이지만, 사용자 인증을하여 시스템에 로그인하여 데이터를 수집해야하는 문제점이 있다. 또한 권한
    문제를 가지고 있기 때문에, 파일의 권한을 확인해야 한다. 보통 키체인 파일은 로그인한 사용자의 키체인 파일만을 수집할 수 있으며, 다른 키체인
    파일은 루트 권한을 가지고 있어야 한다.</li><li>FW를 이용한 논리적 데이터 수집 : Firewire 케이블을 이용하여 두 Mac OS
    X 시스템을 연결하고 수집 대상 시스템을 Target Disk Mode(TDM, Command+T)으로 부팅한다. 수집 대상 시스템에는 FW
    로고가 나타나며, 수집 시스템에 논리적 디스크로 마운트된다. 그 다음에 해당 파일을 복사한다. 이 방법은 파일 복사와 동일하게 데이터를 수집하며,
    사용자 인증없이 키체인 파일을 수집할 수 있는 장점을 가진다. 하지만, FW 단자가 없는 경우엔 인터페이스 자체가 없기 때문에 해당 방법이
    무용지물이 된다. 하지만 외장하드를 연결해서 데이터를 복사하는 방법보다는 좀 더 세련된 방법으로 볼 수 있다.</li><li>싱글 모드 부팅
    후 데이터 수집 : Mac OS X도 유닉스 시스템 중 하나이기 때문에, 싱글 모드로 부팅하여 데이터를 수집할 수 있다. 싱글모드 부팅 방법은
    예전에 블로그에서 설명한 적이 있으니, 해당 내용을 참조하도록 한다. [<a href="http://forensic.n0fate.com/2012/03/mac-os-x-disk-imaging-using-single-mode.html"
    target="_blank" title="">ref</a>] 외부 저장장치는 FAT, HFS+가 읽기/쓰기가 가능하므로, 파일 시스템이 둘
    중에 하나인 외부 저장장치를 활용하도록 한다.</li></ol>위의 세가지 방법 중에 하나의 방법을 이용하여 데이터를 수집하고나면, 메모리
    이미지를 수집하거나, 사용자에게 패스워드를 물어봐야 한다. 논리적 데이터 수집 방법 중 (1)의 방법을 이용했다면, 패스워드를 알고 있으나,
    (2)나 (3) 방법일 경우에는 패스워드를 알지 못하므로, 수집한 메모리 이미지로 분석해야 한다.<br /><br />다음 과정에서는 키체인의
    파일 포맷에 대한 소개와 분석 방법을 알아보겠다.<br /><br /><div>n0fate''s Forensic Space :)</div>'
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>
<div>이전 과정에서는 키체인에 대한 소개와 메모리 이미지 수집 방법을 알아보았다. 그 다음 단계로 키체인 파일이 저장되는 위치를 알아보고, 파일을 수집하는 방법을 고민해봐야한다.</div>
<div></div>
<div>키체인 파일은 어디에 저장이 될까? 키체인 파일은 크게 다음 3 곳에 저장되는 것으로 알려져 있다.</div>
<ul>
<li>~/Library/Keychain/login.keychain : Mac OS X는 각 사용자마다 고유의 키체인 파일을 저장한다. 디지털 포렌식 관점에서 가장 유용한 정보를 담고 있는 파일로, 사용자가 생성한 모든 기밀 정보를 보관하고 있다.</li>
<li>/Library/Keychain/System.keychain : 시스템 키체인 파일로 운영체제마다 하나의 파일을 가지고 있다. 이 파일에는 애플스토어의 애플리케이션을 인증하기 위한 애플의 인증서와 같이 시스템 운영체제 필요한 키나 인증서를 가지고 있다.</li>
<li>/Network/Library/Keychain : 현재까지 저자의 환경에서 확인된 사례는 없으나, 소규모 네트워크에서 키체인을 네트워크로 공유하여 사용함으로, 특정 서비스를 개별 인증할 필요없이 접근하기 위해 제공하는 파일로 판단된다.</li>
</ul>
<p>키체인 파일을 수집하는 방법을 생각해봐야 한다. 대부분의 포렌식 케이스에서는 디스크 이미지를 수집하기 때문에 해당 경로에서 파일을 추출해내면 되지만, 최근 수사영장에서 증거 수집의 범위를 좁혀나가는 상황에서는 특정 파일만 수집하여 분석하는 것을 요구하고 있다. 이러한 상황을 고려하여 논리적인 데이터를 추출 방법도 고민해 봐야 한다. Mac OS X에서 논리적으로 데이터를 수집하는 방법은 다음과 같다.
<ol>
<li>파일 복사를 이용한 데이터 수집 : 가장 평범하면서도 가장 고전적인 방법으로, 그냥 외부 저장장치를 연결하고 해당 파일을 복사하는 방법이다. 어떻게 보면 가장 정확하게 데이터를 수집할 수 있는 방법이지만, 사용자 인증을하여 시스템에 로그인하여 데이터를 수집해야하는 문제점이 있다. 또한 권한 문제를 가지고 있기 때문에, 파일의 권한을 확인해야 한다. 보통 키체인 파일은 로그인한 사용자의 키체인 파일만을 수집할 수 있으며, 다른 키체인 파일은 루트 권한을 가지고 있어야 한다.</li>
<li>FW를 이용한 논리적 데이터 수집 : Firewire 케이블을 이용하여 두 Mac OS X 시스템을 연결하고 수집 대상 시스템을 Target Disk Mode(TDM, Command+T)으로 부팅한다. 수집 대상 시스템에는 FW 로고가 나타나며, 수집 시스템에 논리적 디스크로 마운트된다. 그 다음에 해당 파일을 복사한다. 이 방법은 파일 복사와 동일하게 데이터를 수집하며, 사용자 인증없이 키체인 파일을 수집할 수 있는 장점을 가진다. 하지만, FW 단자가 없는 경우엔 인터페이스 자체가 없기 때문에 해당 방법이 무용지물이 된다. 하지만 외장하드를 연결해서 데이터를 복사하는 방법보다는 좀 더 세련된 방법으로 볼 수 있다.</li>
<li>싱글 모드 부팅 후 데이터 수집 : Mac OS X도 유닉스 시스템 중 하나이기 때문에, 싱글 모드로 부팅하여 데이터를 수집할 수 있다. 싱글모드 부팅 방법은 예전에 블로그에서 설명한 적이 있으니, 해당 내용을 참조하도록 한다. [fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][<a href="http://forensic.n0fate.com/2012/03/mac-os-x-disk-imaging-using-single-mode.html" target="_blank" title="">ref</a>] 외부 저장장치는 FAT, HFS+가 읽기/쓰기가 가능하므로, 파일 시스템이 둘 중에 하나인 외부 저장장치를 활용하도록 한다.</li>
</ol>
<p>위의 세가지 방법 중에 하나의 방법을 이용하여 데이터를 수집하고나면, 메모리 이미지를 수집하거나, 사용자에게 패스워드를 물어봐야 한다. 논리적 데이터 수집 방법 중 (1)의 방법을 이용했다면, 패스워드를 알고 있으나, (2)나 (3) 방법일 경우에는 패스워드를 알지 못하므로, 수집한 메모리 이미지로 분석해야 한다.</p>
<p>다음 과정에서는 키체인의 파일 포맷에 대한 소개와 분석 방법을 알아보겠다.</p>
<div>n0fate's Forensic Space :)</div>
<p>[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
