---
layout: post
title: volafox의 최근 근황
date: 2011-04-29 17:47:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2011/04/volafox.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/5926123129427462130
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '789'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>원래 메인 모듈은 volafox인데 BSD분석 모듈인 volafunx가 더 진행이 되고 있습니다.</p>
<p>이 이유는 다른게 아니라, 프로세스 덤프 기능 때문입니다.</p>
<p>BSD와 다르게 Mac OS X는 커널이 32비트이고 데이터 관리를 32비트로 하지만, 32비트와 64비트 프로세스가 동시에 로드될 수 있는 구조로 되어있습니다. 현재까지 파악한 바로는 PML4(Page Map Level 4)로 페이징하나 Page Directory Pointer Table로 페이징하나 기준 테이블의 위치는 다르지만, 동일한 주소를 가지도록 만들었기 때문인 것 같습니다.</p>
<p>우선은 PML4인 Intel 64비트 운영체제를 기반으로 한 주소 변환 기능 구현을 완료하였기 때문에, 빠른 시일 내에 프로세스 덤프 기능이 추가될 것 같습니다.</p>
<p>원래 글을 따로 쓰진 않았는데, volafunx에 비해 너무 천대 받는 것 같아서 글을 올리게 되었네요. :)</p>
<p>다들 좋은 금요일 저녁 되시길!
<div>n0fate's Forensic Space :)</div>
