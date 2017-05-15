---
layout: post
title: 'Mac OS X Artifacts : "Reopen windows when logging back in"'
date: 2012-05-29 10:24:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/05/mac-os-x-artifacts-reopen-windows-when.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/2351243858969994051
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '830'
  fusion_builder_content_backup: Mac OS X Lion 부터는 운영체제 재부팅 시 현재 활성화된 프로그램을 재시작하는
    기능을 가지고 있다. 이러한 요소는 사용자에게 작업의 연속성을 부여하기 위함으로, 하이버네이션을 사용하지 않는 데스크톱 맥에서 특히 유용한
    기능이라 할 수 있다.<br /><br /><div><a href="http://1.bp.blogspot.com/-fYaPZqZpG9M/T8QkQHvrHZI/AAAAAAAAARQ/qBKzdQ99wfw/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2012-05-29+%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB+10.19.40.png"
    imageanchor="1"><img border="0" src="http://1.bp.blogspot.com/-fYaPZqZpG9M/T8QkQHvrHZI/AAAAAAAAARQ/qBKzdQ99wfw/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2012-05-29+%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB+10.19.40.png"></a></div><br
    /><br />이 기능을 비활성화 시키기 위해 구글링(<a href="http://osxdaily.com/2011/08/25/disable-reopen-windows-when-logging-back-in-in-mac-os-x-lion-completely/"
    target="_blank">Disable "Reopen windows when logging back in</a>)을 해본 분들은 알겠지만,
    맥은 이러한 정보를 "Library/Preferences/ByHost/" 디렉터리에 com.apple.loginwindow.<hardware
    UUID>.plist 에 저장한다. 이 내용을 plist editor로 확인하면 다음과 같다.<br /><br /><div><a href="http://3.bp.blogspot.com/-p9_Gy965WxQ/T8Qp5WYKGgI/AAAAAAAAARc/hO6wVaDrDj4/s1600/plist.jpg"
    imageanchor="1"><img border="0" src="http://3.bp.blogspot.com/-p9_Gy965WxQ/T8Qp5WYKGgI/AAAAAAAAARc/hO6wVaDrDj4/s1600/plist.jpg"></a></div><br
    /><div></div><br />Background와 Hide는 애플리케이션이 실행될 떄의 옵션이며, 실제로 유용한 정보는 BundleID와
    Path이다. 재부팅 시 실행시킬 프로그램 목록은 매번 갱신되어 저장되기 때문에 시스템에 현재 실행된 프로그램을 파악하는데도 유용하게 사용될
    수 있다. 또한 휘발성 데이터가 아니기 때문에 디스크 이미징 후 분석 시에도 사용자가 시스템에서 실행한 프로그램을 파악하는데 유용하게 사용될
    수 있다.<br /><br />이 디렉터리엔 이 요소 외에도 포렌식적으로 유용한 여러가지 정보(구글 연락처 정보 등)가 존재한다. 앞으로 시간을
    내어서 이러한 내용에 대해서도 하나하나 다뤄보도록 하겠다. :)<br /><br /><br /><br /><div>n0fate's Forensic
    Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>Mac OS X Lion 부터는 운영체제 재부팅 시 현재 활성화된 프로그램을 재시작하는 기능을 가지고 있다. 이러한 요소는 사용자에게 작업의 연속성을 부여하기 위함으로, 하이버네이션을 사용하지 않는 데스크톱 맥에서 특히 유용한 기능이라 할 수 있다.</p>
<div><a href="http://1.bp.blogspot.com/-fYaPZqZpG9M/T8QkQHvrHZI/AAAAAAAAARQ/qBKzdQ99wfw/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2012-05-29+%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB+10.19.40.png" imageanchor="1"><img border="0" src="{{ site.baseurl }}/assets/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2012-05-29+%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB+10.19.40.png" /></a></div>
<p>이 기능을 비활성화 시키기 위해 구글링(<a href="http://osxdaily.com/2011/08/25/disable-reopen-windows-when-logging-back-in-in-mac-os-x-lion-completely/" target="_blank">Disable "Reopen windows when logging back in</a>)을 해본 분들은 알겠지만, 맥은 이러한 정보를 "Library/Preferences/ByHost/" 디렉터리에 com.apple.loginwindow.<hardware uuid>.plist 에 저장한다. 이 내용을 plist editor로 확인하면 다음과 같다.</p>
<div><a href="http://3.bp.blogspot.com/-p9_Gy965WxQ/T8Qp5WYKGgI/AAAAAAAAARc/hO6wVaDrDj4/s1600/plist.jpg" imageanchor="1"><img border="0" src="{{ site.baseurl }}/assets/plist.jpg" /></a></div>
<p>
<div></div>
<p>Background와 Hide는 애플리케이션이 실행될 떄의 옵션이며, 실제로 유용한 정보는 BundleID와 Path이다. 재부팅 시 실행시킬 프로그램 목록은 매번 갱신되어 저장되기 때문에 시스템에 현재 실행된 프로그램을 파악하는데도 유용하게 사용될 수 있다. 또한 휘발성 데이터가 아니기 때문에 디스크 이미징 후 분석 시에도 사용자가 시스템에서 실행한 프로그램을 파악하는데 유용하게 사용될 수 있다.</p>
<p>이 디렉터리엔 이 요소 외에도 포렌식적으로 유용한 여러가지 정보(구글 연락처 정보 등)가 존재한다. 앞으로 시간을 내어서 이러한 내용에 대해서도 하나하나 다뤄보도록 하겠다. :)</p>
<div>n0fate's Forensic Space :)</div>
<p></hardware></p>
