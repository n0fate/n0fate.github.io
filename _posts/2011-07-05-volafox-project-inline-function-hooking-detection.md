---
layout: post
title: 'volafox project: inline function hooking detection'
date: 2011-07-05 09:57:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2011/07/volafox-project-inline-function-hooking.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/2445834213236363471
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '768'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>volafox 프로젝트로 만들어진 도구에서 시스템 콜을 조작하는 루트킷의 정보 변조 기법을 탐지하기 위해 업데이트를 하고 있습니다.<br />사실 단순히 시스템 콜의 핸들러 주소를 변경하는 System call Hooking은 현재 해당 프로젝트에서도 어느정도 탐지해 낼 수 있습니다만, inline function hooking 기술의 경우엔 시스템 콜 함수의 포인터 주소를 변형하는 기술이 아닌 함수 자체에 앞의 x 바이트를 변조시키거나 함수 전체를 evil stuff code를 덮어버리는 형식으로 진행되는 것이라, 코드 자체에 대한 변조 여부를 탐지해야 합니다.<br />해당 기술을 탐지하기 위해 volafox 프로젝트에 추가될 내용은 다음과 같습니다.<br />
<blockquote>아래부터는 FreeBSD 메모리 분석 도구인 volafunx를 기준으로 설명합니다. 본 기법을 BSD기반에 먼저 적용하고 효과가 있으면 맥용 도구에도 적용할 예정입니다.</p></blockquote>
<p>syscall_info에 KLD 목록 출력 시 사용했던 -v옵션을 적용하여, 활성화 시 inline function hooking을 탐지하기 위해 커널 이미지에서 추출한 시스템 콜 핸들러 주소(커널 이미지 내의 심볼 정보)와 코드 베이스 주소(커널 이미지 헤더에 정의), 그리고 커널이 로드된 주소(일반적으로 0xC0000000)을 이용하여, 각 심볼에 맞는 코드 영역을 추출하여, 해시 값을 산출하여, 시스템 콜의 핸들러 함수의 해시 값과 비교하는 방법을 구현할 예정입니다.<br />커널 이미지와 메모리 이미지 상의 시작 주소와 끝 주소는 다음과 같이 정의합니다.
<ul>
<li>커널 이미지 상의 시작 주소: 시스템 콜 핸들러 주소 - 커널이 로드된 주소 -코드 베이스 주소</li>
<li>커널 이미지 상의 끝 주소: 읽은 코드 바이트가 <s>C3(</s>retn<s>)</s>인 경우, 코드의 끝으로 정의</li>
</ul>
<p>
<ul>
<li>메모리 이미지 상의 시작 주소: sysent 구조체의 핸들러 함수의 주소</li>
<li>메모리 이미지 상의 끝 주소: 커널 이미지 상의 끝 주소를 판단하는 방법과 동일</li>
</ul>
<p>해당 정보를 이용하면 inline function hooking의 변조를 탐지할 수 있을 것으로 생각됩니다.(물론 좀더 확인할 필요는 있습니다.) 단지 이 방법은 성능이 엄청나게 떨어지는 문제가 있기 때문에 미리 커널에서 정보를 빼내 xml 형태로 구성하여 메모리 분석 도구 자체는 xml에서 정보를 읽어들여 분석하는 방법도 생각하고 있습니다.(사실 이 부분은 전부터 생각했던 문제인데, 자꾸 손을 안대고 있네요. 지금도 volafunx는 커널 이미지에서 심볼 가져오는게 엄청 느리지요. 저의 발적화로 인해 xD)<br />본 기능에 대한 이론적인 베이스는 정리가 되었으니 구현만 하면 될 것 같습니다. 조만간 좀더 강력해진 도구를 보여드릴 수 있겠네요 :)
<div>n0fate's Forensic Space :)</div>
