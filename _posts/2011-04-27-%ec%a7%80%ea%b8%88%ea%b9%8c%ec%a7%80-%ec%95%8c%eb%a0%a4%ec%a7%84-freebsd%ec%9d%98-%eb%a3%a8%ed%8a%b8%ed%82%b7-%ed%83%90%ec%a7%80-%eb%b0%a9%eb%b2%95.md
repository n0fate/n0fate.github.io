---
layout: post
title: 지금까지 알려진 FreeBSD의 루트킷 탐지 방법
date: 2011-04-27 17:42:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /feeds/7620918615785302711/posts/default/4994719496433718056
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/4994719496433718056
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '800'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p><span>FreeBSD의 정보 은닉 기법은 시스템 콜 주소를 수정하거나, 새로운 콜 함수를 추가하여 호출하는 등의 행위를 한다.</span><br /><span><br /></span><br /><span>이러한 역할을 수행하기 위해 루트킷은 FreeBSD의 모듈 개념인 KLD(Kernel Loadable Driver)로 구현되어 있다. 이러한 BSD의 모듈 제어를 위한 명령어는 크게 3가지가 존재한다.</span></p>
<ul>
<li><span>kldload - BSD의 커널 모듈을 로드하기 위한 명령</span></li>
<li><span>kldstat - BSD의 linker files 목록을 확인하기 위한 명령(-v 옵션으로 세부 모듈 정보 확인)</span></li>
<li><span>kldunload - BSD의 커널 모듈을 언로드하기 위한 명령</span></li>
</ul>
<p>kldstat 명령을 통해 나오는 결과는 linker files라 부르며, 세부 커널 모듈들을 연결시켜주는 존재이다. 이는 보통 BSD에서 KLD를 작성 시 겉 껍데기와 같은 역할을 수행한다.</p>
<p>여기서 우리가 루트킷을 탐지하는 방법은 두 가지를 생각해볼 수 있다. 첫 번째는 kldstat 명령을 통해 비 정상적으로 로드된 모듈이 있는지 확인하여 해당 모듈을 언로드하고, 경로 정보를 통해 추출 및 분석을 수행하는 것이 있으며, 두 번째는 시스템 콜의 무결성 여부를 판단하여, 변조된 시스템 콜 함수의 주소를 페이지 단위로 추출하여, 함수의 역할을 파악하는 방법이 있다.</p>
<p>두 방법 다 일반적인 루트킷 감염 시스템에서 탐지를 위해 많이 사용되지만</p>
