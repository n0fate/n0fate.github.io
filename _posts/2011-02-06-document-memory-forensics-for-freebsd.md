---
layout: post
title: '[Document] Memory forensics for FreeBSD'
date: 2011-02-06 21:03:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2011/02/freebsd.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/3544972333672074090
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '687'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>BSD환경은 기업의 서버에서 많이 사용됨에 따라 침해사고 조사 분석의 대상이 될 수 있습니다.<br />본 문서는 제가 개발한 FreeBSD환경의 메모리 포렌식 도구인 volafunx에 대한 소개와 각 기능이 어떤식으로 구현되었는지에 대한 정보를 획득할 수 있습니다.</p>
<p>제가 아는 지식을 기준으로 작성되어 있기 때문에 틀린 부분도 많을 것입니다. 많은 BSD 고수님들이 사랑의 태클(?)을 걸어주시면 문서의 퀄리티를 높이는데 큰 도움이 될 것 같습니다. ;)</p>
<p>문서는 지속적인 업데이트를 수행할 예정이며, 차 후 통합 메모리 포렌식 가이드로 이어나갈 생각을 하고 있습니다.<br />디지털 포렌식 기술을 이용하거나 연구하시는 분들에게 많은 도움이 되었으면 합니다. :)</p>
<p>다운로드(Ver. 1): <a href="http://dl.dropbox.com/0/view/av1scna36wpksbh/shared/Memory%20forensics%20for%20FreeBSD.pdf">Memory Forensics for FreeBSD</a>
<div>n0fate's Forensic Space :)</div>
