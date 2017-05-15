---
layout: post
title: volafunx 0.2 beta1 Release!
date: 2011-04-29 17:34:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2011/04/volafunx-02-beta1-release.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/3965737032201116329
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '715'
  fusion_builder_content_backup: <table align="center" cellpadding="0" cellspacing="0"><tbody><tr><td><a
    href="http://4.bp.blogspot.com/-yU_uoWB1pcs/Tbp3-ZsmFmI/AAAAAAAAAI0/DP3vPhAUgh4/s1600/Logo.png"
    imageanchor="1"><img border="0" src="http://4.bp.blogspot.com/-yU_uoWB1pcs/Tbp3-ZsmFmI/AAAAAAAAAI0/DP3vPhAUgh4/s1600/Logo.png"></a></td></tr><tr><td><b><쓸
    때마다 저작권 걱정하는 volafox의 아이콘></b></td></tr></tbody></table><br />4월도 거의 끝나가네요 :)<br
    />거의 안쓰시는 버림받은 도구인 volafox 0.2가 베타1 버전이 공개되었습니다.<br />변화된 기능은 다음과 같습니다.<br /><br
    /><br /><table><tbody><tr><th align="left" valign="top">Description:</th><td align="left"
    valign="top"><pre>* KLD 리스트 출력 시 발생한 문제 해결<br /> * 이젠 BSD linker 내부의 모듈 정보도 함께
    확인할 수 있습니다.(-v 옵션)<br /> * volafunx는 이제 네트워크 일부 정보(포트)를 추출할 수 있습니다.</pre><pre></pre></td></tr></tbody></table><br
    />현재 inpcb 구조체에서 네트워크 정보를 추출 중인데, IP에 대한 정보는 존재하지 않는 것을 확인하였습니다. netstat는 아마 다른
    구조체와 맵핑을 하는 것 같은데, 이 부분에 대해 좀 더 증명이 필요할 것 같습니다. 이 기술이 완료되면 바로 beta2로 넘어갈 예정입니다.<br
    />네트워크 정보 추출 기능은 5월달 내로 예상하고 있으나, 이는 엄청나게 높은 확률로 변경될 수 있습니다.<br /><br />또한 저번주부터
    BSD 루트킷의 은닉 기법들을 체크하여, 각 기능을 우회하여 올바른 정보를 수집할 수 있는지 테스트하고 있습니다.<br />조만간 다양한 루트킷
    기법을 정리하여, 각 기법을 어떤식으로 탐지할 수 있는지에 대한 블로깅을 하겠습니다.<br /><br /><a href="http://code.google.com/p/volafox/downloads/detail?name=volafunx-0.2-beta.zip&can=2&q=">volafunx
    0.2 beta1 Download Link</a><div>n0fate's Forensic Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<table align="center" cellpadding="0" cellspacing="0">
<tbody>
<tr>
<td><a href="http://4.bp.blogspot.com/-yU_uoWB1pcs/Tbp3-ZsmFmI/AAAAAAAAAI0/DP3vPhAUgh4/s1600/Logo.png" imageanchor="1"><img border="0" src="{{ site.baseurl }}/assets/Logo.png" /></a></td>
</tr>
<tr>
<td><b><쓸 때마다 저작권 걱정하는 volafox의 아이콘></b></td>
</tr>
</tbody>
</table>
<p>4월도 거의 끝나가네요 :)<br />거의 안쓰시는 버림받은 도구인 volafox 0.2가 베타1 버전이 공개되었습니다.<br />변화된 기능은 다음과 같습니다.</p>
<p>
<table>
<tbody>
<tr>
<th align="left" valign="top">Description:</th>
<td align="left" valign="top">
<pre>* KLD 리스트 출력 시 발생한 문제 해결<br /> * 이젠 BSD linker 내부의 모듈 정보도 함께 확인할 수 있습니다.(-v 옵션)<br /> * volafunx는 이제 네트워크 일부 정보(포트)를 추출할 수 있습니다.</pre>
<pre></pre>
</td>
</tr>
</tbody>
</table>
<p>현재 inpcb 구조체에서 네트워크 정보를 추출 중인데, IP에 대한 정보는 존재하지 않는 것을 확인하였습니다. netstat는 아마 다른 구조체와 맵핑을 하는 것 같은데, 이 부분에 대해 좀 더 증명이 필요할 것 같습니다. 이 기술이 완료되면 바로 beta2로 넘어갈 예정입니다.<br />네트워크 정보 추출 기능은 5월달 내로 예상하고 있으나, 이는 엄청나게 높은 확률로 변경될 수 있습니다.</p>
<p>또한 저번주부터 BSD 루트킷의 은닉 기법들을 체크하여, 각 기능을 우회하여 올바른 정보를 수집할 수 있는지 테스트하고 있습니다.<br />조만간 다양한 루트킷 기법을 정리하여, 각 기법을 어떤식으로 탐지할 수 있는지에 대한 블로깅을 하겠습니다.</p>
<p><a href="http://code.google.com/p/volafox/downloads/detail?name=volafunx-0.2-beta.zip&can=2&q=">volafunx 0.2 beta1 Download Link</a>
<div>n0fate's Forensic Space :)</div>
