---
layout: post
title: rdisk0 vs disk0
date: 2012-02-13 22:00:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/02/rdisk0-vs-disk0.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/6638123731661962185
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1003'
  fusion_builder_content_backup: 오늘 터미널에서 이것 저것 보던 중에 Mac OS X의 /dev 에는 rdisk0와 disk0
    가 존재함을 알았다. 보통 리눅스 계열의 경우엔 sda나 hda 디바이스가 존재해서 해당 디바이스의 이미지를 dd를 이용해서 덤프할 수 있으나,
    Mac OS X에서는 이런 장치가 두 개가 존재하는 것이였다. 둘이 차이점이 뭔지 확인해보니 그 답은 Mac OS X Internal 책에
    있었다.<br /><br />우선 /dev/rdisk0는 raw device로 character device이며, 데이터를 별도의 운영체제의
    버퍼 캐시를 통하지 않고 raw 형태로 전송한다고 한다.  이는 버퍼 캐시의 Invalidating없이 디스크 파티션을 생성하거나, 파일 시스템을
    생성, 이미 존재하는 파일 시스템의 복구를 수행할 수 있도록 해준다. 보통 리눅스 파일 시스템에 있는 디스크 장치는 맥에서 /dev/rdisk0라고
    볼 수 있으며, 책에도 해당 장치에 I/O 요청 시에는 디스크의 블록 크기와 오프셋이 필요하다고 작성되어 있다.<br /><br />반대로
    /dev/disk0의 경우엔 운영체제의 버퍼 캐시를 통해 데이터를 전송한다. 여기서 버퍼 캐시는 물리 장치의 데이터 청크의 위치 정보를 가지는
    블록 넘버와 장치의 인덱스 정보를 가지고 있으며, 애플리케이션이 디스크의 데이터를 접근하여 사용하는 경우가 많은데, 이 때 운영체제는 disk0를
    이용하여 캐시된 데이터를 유지하고 이를 애플리케이션에 제공하는 형태로 사용할 수 있도록 하였다.<br /><br />물론 raw device는
    저수준의 데이터 접근이 가능하기 때문에, mmap()함수를 통해서도 동일한 역할을 수행할 순 있다고 한다.<br /><br /><strong>결론은
    dd를 이용해서 맥에서 디스크 이미징을 할려면, /dev/rdisk0 를 입력으로 받아 처리해야 된다 :-)</strong><div>n0fate's
    Forensic Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>오늘 터미널에서 이것 저것 보던 중에 Mac OS X의 /dev 에는 rdisk0와 disk0 가 존재함을 알았다. 보통 리눅스 계열의 경우엔 sda나 hda 디바이스가 존재해서 해당 디바이스의 이미지를 dd를 이용해서 덤프할 수 있으나, Mac OS X에서는 이런 장치가 두 개가 존재하는 것이였다. 둘이 차이점이 뭔지 확인해보니 그 답은 Mac OS X Internal 책에 있었다.</p>
<p>우선 /dev/rdisk0는 raw device로 character device이며, 데이터를 별도의 운영체제의 버퍼 캐시를 통하지 않고 raw 형태로 전송한다고 한다.  이는 버퍼 캐시의 Invalidating없이 디스크 파티션을 생성하거나, 파일 시스템을 생성, 이미 존재하는 파일 시스템의 복구를 수행할 수 있도록 해준다. 보통 리눅스 파일 시스템에 있는 디스크 장치는 맥에서 /dev/rdisk0라고 볼 수 있으며, 책에도 해당 장치에 I/O 요청 시에는 디스크의 블록 크기와 오프셋이 필요하다고 작성되어 있다.</p>
<p>반대로 /dev/disk0의 경우엔 운영체제의 버퍼 캐시를 통해 데이터를 전송한다. 여기서 버퍼 캐시는 물리 장치의 데이터 청크의 위치 정보를 가지는 블록 넘버와 장치의 인덱스 정보를 가지고 있으며, 애플리케이션이 디스크의 데이터를 접근하여 사용하는 경우가 많은데, 이 때 운영체제는 disk0를 이용하여 캐시된 데이터를 유지하고 이를 애플리케이션에 제공하는 형태로 사용할 수 있도록 하였다.</p>
<p>물론 raw device는 저수준의 데이터 접근이 가능하기 때문에, mmap()함수를 통해서도 동일한 역할을 수행할 순 있다고 한다.</p>
<p><strong>결론은 dd를 이용해서 맥에서 디스크 이미징을 할려면, /dev/rdisk0 를 입력으로 받아 처리해야 된다 :-)</strong>
<div>n0fate's Forensic Space :)</div>
