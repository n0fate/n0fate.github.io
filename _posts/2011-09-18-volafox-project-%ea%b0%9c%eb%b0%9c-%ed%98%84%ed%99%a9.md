---
layout: post
title: 'volafox Project: 개발 현황'
date: 2011-09-18 23:49:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2011/09/volafox.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/8961500022634681364
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1170'
  fusion_builder_content_backup: 오랜만에 포스팅합니다. 그간 회사 일과 다양한 외부 프로젝트를 준비 중이다보니 포스팅이
    뜸하게 되었네요.<br />현재 0.6버전 이 후에도 계속 개발되고 있으며 많은 점이 바뀌고 있습니다.<br />대표적으로 변경된 점은 다음과
    같습니다.<br /><br /><strong>1. 네트워크 정보 추출 기능</strong><br />현재 이 기능은 베타로 모든 데이터를 가져오진
    못하고 있습니다. 하지만 네트워크 정보가 휘발성 정보이기 때문에, 침해사고 발생 시 유용하게 사용될 수 있을 것 같아서 일단 추가는 해 두었습니다.(아직
    버그가 존재합니다.)<br /><a href="http://lh6.ggpht.com/-BtTfr9YPY64/TnYE-CKTENI/AAAAAAAAALw/I4ba1b3c_nw/s1600-h/image%25255B4%25255D.png"><img
    alt="image" border="0" height="150" src="http://lh4.ggpht.com/-06LK-LIskrE/TnYE_psDJ9I/AAAAAAAAAL0/uYoqmGWNqsU/image_thumb%25255B2%25255D.png?imgmax=800"
    title="image" width="400"></a><br /><br /><strong>2. 커널 심볼 정보의 DB화(overlays)</strong><br
    />이 방법은 Chris Leats가 제게 보내준 것으로, 커널 이미지에서 추출한 심볼 정보를 미리 생성하여 보관하여 필요에 따라 overlay
    데이터를 바로 끌어다 사용할 수 있게 해주는 기능이라 할 수 있습니다. 이로인해 분석 속도를 획기적으로 높일 수 있었습니다. pickle 라이브러리로
    간단하게 구현할 수 있었습니다.<br /><br /><strong>3. 메모리 이미지 내에서 커널 버전 정보 추출</strong><br />이
    방법도 Chris Leats가 알려준 유용한 내용입니다.(Thx Chris Leats :) ) 가상 주소 0x2000에 존재하는 eyecatcher(‘Catfish’)를
    이용하여 오프셋에 위치한 운영체제 버전 정보, 다윈 커널 버전 정보, KEXT 시작 주소 정보를 획득할 수 있습니다. 시그너처를 탐색하여 운영체제
    버전 정보를 획득하면, 해당 메모리 이미지에 맞는 Overlay 데이터를 알려줄 수 있기 때문에, 분석하는 입장에서 맥 운영체제의 버전 정보를
    별도로 얻을 필요가 없어지게 됩니다. 이러한 점은 맥 메모리 포렌식의 무결성을 높여줄 수 있는 요소가 되었습니다.<br /><a href="http://lh6.ggpht.com/-DcwbeeB5LEE/TnYFA_uQlBI/AAAAAAAAAL4/XmIBLNEjkaA/s1600-h/image%25255B10%25255D.png"><img
    alt="image" border="0" height="84" src="http://lh4.ggpht.com/-O-k6npP4sdk/TnYFBrQbCvI/AAAAAAAAAL8/-zws7FEkysk/image_thumb%25255B6%25255D.png?imgmax=800"
    title="image" width="320"></a><br /><br /><strong>4. Mac Memory Reader를 이용하여 추출한
    메모리 이미지 분석 기능</strong><br />기존 volafox는 linear한 맥 메모리 만을 분석할 수 있었습니다. 이는 Firewire로
    추출한 메모리 또는 vmware상의 맥에 대한 메모리 이미지 분석만을 할 수 있게되어 분석 범위에 한계를 가지게 되는 문제가 있었습니다.(맥북에어나
    맥북은 firewire가 없음). 이번에 Mac Memory Reader에서 추출한 정보의 분석 기능이 추가됨에 따라 이제 아무런 문제 없이
    모든 맥의 메모리 정보를 분석할 수 있습니다.<br /><br />이것 외에도 추가적인 여러 기능을 추가 중에 있습니다. 그리고 새로운 소식도
    있습니다. 뉴욕에 거주 중인 Forensic Researcher 두 분(@hajimeinoue, @osxmem)이 volafox project에
    함께하고 있습니다. 맥 쪽으로 연구를 많이 하는 친구들이기 때문에 여러모로 도움이 많이 될 듯 합니다. :)<div>n0fate's Forensic
    Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>오랜만에 포스팅합니다. 그간 회사 일과 다양한 외부 프로젝트를 준비 중이다보니 포스팅이 뜸하게 되었네요.<br />현재 0.6버전 이 후에도 계속 개발되고 있으며 많은 점이 바뀌고 있습니다.<br />대표적으로 변경된 점은 다음과 같습니다.</p>
<p><strong>1. 네트워크 정보 추출 기능</strong><br />현재 이 기능은 베타로 모든 데이터를 가져오진 못하고 있습니다. 하지만 네트워크 정보가 휘발성 정보이기 때문에, 침해사고 발생 시 유용하게 사용될 수 있을 것 같아서 일단 추가는 해 두었습니다.(아직 버그가 존재합니다.)<br /><a href="http://lh6.ggpht.com/-BtTfr9YPY64/TnYE-CKTENI/AAAAAAAAALw/I4ba1b3c_nw/s1600-h/image%25255B4%25255D.png"><img alt="image" border="0" height="150" src="{{ site.baseurl }}/assets/image_thumb%25255B2%25255D.png?imgmax=800" title="image" width="400" /></a></p>
<p><strong>2. 커널 심볼 정보의 DB화(overlays)</strong><br />이 방법은 Chris Leats가 제게 보내준 것으로, 커널 이미지에서 추출한 심볼 정보를 미리 생성하여 보관하여 필요에 따라 overlay 데이터를 바로 끌어다 사용할 수 있게 해주는 기능이라 할 수 있습니다. 이로인해 분석 속도를 획기적으로 높일 수 있었습니다. pickle 라이브러리로 간단하게 구현할 수 있었습니다.</p>
<p><strong>3. 메모리 이미지 내에서 커널 버전 정보 추출</strong><br />이 방법도 Chris Leats가 알려준 유용한 내용입니다.(Thx Chris Leats :) ) 가상 주소 0x2000에 존재하는 eyecatcher(‘Catfish’)를 이용하여 오프셋에 위치한 운영체제 버전 정보, 다윈 커널 버전 정보, KEXT 시작 주소 정보를 획득할 수 있습니다. 시그너처를 탐색하여 운영체제 버전 정보를 획득하면, 해당 메모리 이미지에 맞는 Overlay 데이터를 알려줄 수 있기 때문에, 분석하는 입장에서 맥 운영체제의 버전 정보를 별도로 얻을 필요가 없어지게 됩니다. 이러한 점은 맥 메모리 포렌식의 무결성을 높여줄 수 있는 요소가 되었습니다.<br /><a href="http://lh6.ggpht.com/-DcwbeeB5LEE/TnYFA_uQlBI/AAAAAAAAAL4/XmIBLNEjkaA/s1600-h/image%25255B10%25255D.png"><img alt="image" border="0" height="84" src="{{ site.baseurl }}/assets/image_thumb%25255B6%25255D.png?imgmax=800" title="image" width="320" /></a></p>
<p><strong>4. Mac Memory Reader를 이용하여 추출한 메모리 이미지 분석 기능</strong><br />기존 volafox는 linear한 맥 메모리 만을 분석할 수 있었습니다. 이는 Firewire로 추출한 메모리 또는 vmware상의 맥에 대한 메모리 이미지 분석만을 할 수 있게되어 분석 범위에 한계를 가지게 되는 문제가 있었습니다.(맥북에어나 맥북은 firewire가 없음). 이번에 Mac Memory Reader에서 추출한 정보의 분석 기능이 추가됨에 따라 이제 아무런 문제 없이 모든 맥의 메모리 정보를 분석할 수 있습니다.</p>
<p>이것 외에도 추가적인 여러 기능을 추가 중에 있습니다. 그리고 새로운 소식도 있습니다. 뉴욕에 거주 중인 Forensic Researcher 두 분(@hajimeinoue, @osxmem)이 volafox project에 함께하고 있습니다. 맥 쪽으로 연구를 많이 하는 친구들이기 때문에 여러모로 도움이 많이 될 듯 합니다. :)
<div>n0fate's Forensic Space :)</div>
