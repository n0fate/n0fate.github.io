---
layout: post
title: memory forensics for FreeBSD (Korean)
date: 2011-01-10 10:23:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /feeds/7620918615785302711/posts/default/6891898788505899438
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/6891898788505899438
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '719'
  fusion_builder_content_backup: <span><본 페이지는 지속적으로 업데이트가 될 예정 입니다.></span><br /><br
    />최근에 짬을 내서 메모리 포렌식에 여러 보안 관계자 분들이 쉽게 접근할 수 있도록 메모리 포렌식 문서를 작성하고 있습니다.<br /><br
    />우선은 제가 개발한 도구를 소개하는 목적으로 FreeBSD 운영체제의 메모리 포렌식에 대해 정리를 하고 있는 상태이며, 문서의 서두에도
    작성하였지만, 그 내용을 점차 늘릴 생각을 하고 있습니다.<br /><br />크게는 Mac OS X를 추가하고, 그 다음에 리눅스를 추가할
    생각이며, 윈도우는 <a href="http://baadc0de.blogspot.com/">xnetblue</a>님이 알아서 잘 해주실거라
    생각하고 있습니다 :)<br /><br /><a href="https://files.ucloud.com/pf/D85908_83009_650220">memory
    forensics for FreeBSD</a> (Korean)<br /><br />문서는 pdf 포맷으로 되어 있으며, 한글로 작성하였고,
    우선은 A4에 맞춰 작성하였습니다. 차 후 킨들용으로 별도로 컨버팅도 할 생각입니다 :)<br /><br />문서에는 틀린 점도 존재하고 내용이
    부족한 부분도 있을 것입니다. 글을 보신 분들이 많이 피드백을 주시면 감사하겠습니다.<br /><br /><문서 이력><br />2011.
    Jan. 10. 최초 문서 발행
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p><span><본 페이지는 지속적으로 업데이트가 될 예정 입니다.></span></p>
<p>최근에 짬을 내서 메모리 포렌식에 여러 보안 관계자 분들이 쉽게 접근할 수 있도록 메모리 포렌식 문서를 작성하고 있습니다.</p>
<p>우선은 제가 개발한 도구를 소개하는 목적으로 FreeBSD 운영체제의 메모리 포렌식에 대해 정리를 하고 있는 상태이며, 문서의 서두에도 작성하였지만, 그 내용을 점차 늘릴 생각을 하고 있습니다.</p>
<p>크게는 Mac OS X를 추가하고, 그 다음에 리눅스를 추가할 생각이며, 윈도우는 <a href="http://baadc0de.blogspot.com/">xnetblue</a>님이 알아서 잘 해주실거라 생각하고 있습니다 :)</p>
<p><a href="https://files.ucloud.com/pf/D85908_83009_650220">memory forensics for FreeBSD</a> (Korean)</p>
<p>문서는 pdf 포맷으로 되어 있으며, 한글로 작성하였고, 우선은 A4에 맞춰 작성하였습니다. 차 후 킨들용으로 별도로 컨버팅도 할 생각입니다 :)</p>
<p>문서에는 틀린 점도 존재하고 내용이 부족한 부분도 있을 것입니다. 글을 보신 분들이 많이 피드백을 주시면 감사하겠습니다.</p>
<p><문서 이력><br />2011. Jan. 10. 최초 문서 발행</p>
