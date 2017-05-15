---
layout: post
title: FreeBSD Rootkit Part1 - Understanding KLD & Skeleton Code
date: 2011-05-01 22:42:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
tags:
- freebsd
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2011/05/freebsd-rootkit-part1-understanding-kld.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/8666281662937643157
  avada_post_views_count: '791'
  _edit_last: '1'
  slide_template: ''
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>FreeBSD는 현재 리눅스 2.6 커널과 마찬가지로, 단일 커널에 주요 기능을 부여하고, 3rd party 개발자가 추가로 원하는 기능이나 디바이스 드라이버를 손쉽게 추가하기 위해 동적 커널 링커를 제공한다. 동적 커널 링커를 연결함으로 시스템에 새로운 모듈을 추가하더라도 별도의 시스템 재부팅이 필요하지 않아 개발자 뿐만 아니라 서버 관리자에게 개발 편의성과 서버의 가용성을 높인다.</p>
<p>동적 커널 링커는 리눅스와 BSD 그리고 Mac OS X에서 각각 다른 이름으로 불리지만, 기능이 거의 유사하기 때문에 커널의 메모리 영역을 접근할 수 있고 루트 권한으로 작동한다는 점을 이용하여, 악의적인 사용자들은 루트킷의 다양한 테크닉을 적용하기 위한 목적으로 사용되고 있다. 이에 지속적인 블로그 포스팅을 통해 루트킷 기법을 설명하고 이를 방어하기 위한 다양한 방법에 대해 생각해보고자 한다. 금일은 간단하게 KLD를 이해하고 뼈대 코드를 작성하는 시간을 가지도록 하겠다.</p>
<p>KLD는 FreeBSD 3.x 시절에 LKM(Loadable Kernel Module)로 불리던 것으로 KLD로 변경되면서 현재와 같이 하나의 링커 파일에 다양한 모듈이 추가될 수 있다. 이는 BSD에서 로드된 동적 커널 링커의 목록인 kldstat를 이용하여 확인할 수 있다.</p>
<pre name="code">[n0fate@FreeBSD ~/rootkit/basic]$ kldstat -v<br />Id Refs Address    Size     Name<br />1    1 0xc0400000 bd97b4   kernel (/boot/kernel/kernel)<br />Contains modules:<br />Id Name<br />94 ataraid<br />364 if_lo<br />351 elf32<br />352 shell<br />336 pseudofs<br /></pre>
<p>이러한 커널 링커의 특징으로 여러 모듈을 각각 로드할 필요 없이, 하나의 컴포넌트(링커 파일)로 생성하여 관리할 수 있어 관리의 효용성을 높일 수 있다. 이 외에도 LKM과 KLD의 차이점에 대한 내용은 FreeBSD 포럼에 올라온 <a href="http://www.kr.freebsd.org/doc/KLD-Programming/">동적 커널 링커(KLD) 프로그래밍 튜토리얼</a>을 통해 확인할 수 있다. 링크의 사이트에도 추가적인 skeleton code를 제공하고 기본 코드 작성을 위해 세세하게 코드를 설명하였기 때문에, 개발에 관심이 있다면 해당 문서를 탐독하는 것도 공부에 도움이 될 수 있다. 단 FreeBSD 4.0을 기반으로 작성하였기 때문에 약간 변경된 점이 있다는 것은 고려해야 할 것이다. </p>
<p>이제 실제 FreeBSD의 코드를 확인해 보자. 모든 코드는 FreeBSD 7~8까지 호환될 것으로 생각한다. 실제 개발을 하면서 보고 싶다면, FreeBSD 설치 시 Developer버전으로 설치하는 것이 좋다.</p>
<pre name="code">#include <sys param.h><br />#include <sys module.h><br />#include <sys kernel.h><br />#include <sys systm.h><br /><br />static int<br />load(struct  module* module, int cmd, void *arg)<br />{<br /> int error = 0;<br /><br /> switch(cmd) {<br /> case MOD_LOAD:<br />  uprintf("loadn");<br />  break;<br /><br /> case MOD_UNLOAD:<br />  uprintf("unloadn");<br />  break;<br /><br /> default:<br />  error = EOPNOTSUPP;<br />  break;<br /> }<br /><br /> return error;<br />}<br /><br />static moduledata_t hello_mod = {<br /> "hello",<br /> load,<br /> NULL<br />};<br /><br />DECLARE_MODULE(hello, hello_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);<br /></sys></sys></sys></sys></pre>
<p>코드를 컴파일하기 위한 Makefile은 다음과 같다.</p>
<pre name="code">SRCS = hello.c<br />KMOD = hello<br /><br />.include <bsd.kmod.mk><br /></bsd.kmod.mk></pre>
<p>빌드는 간단히 make를 통해 가능하며, kldload ./hello.ko 를 통해 모듈을 로드할 수 있다.
<pre name="code">[n0fate@FreeBSD ~/rootkit/basic]$ make<br />[n0fate@FreeBSD ~/rootkit/basic]$ sudo kldload ./hello.ko<br />Password:<br />load<br />[n0fate@FreeBSD ~/rootkit/basic]$ kldstat -v<br />Id Refs Address    Size     Name<br /> 1    3 0xc0400000 bd97b4   kernel (/boot/kernel/kernel)<br /> Contains modules:<br />  Id Name<br />  94 ataraid<br />  364 if_lo<br />  351 elf32<br />  352 shell<br />  336 pseudofs<br />  365 if_tun<br />  427 elink<br />  363 if_gif<br />  375 mld<br />  374 igmp<br />...<br />...<br />...<br />  373 wlan_sta<br />  337 g_dev<br />  362 if_firewire<br />  360 ether<br />  440 x86bios<br /> 2    1 0xc1f97000 2000     hello.ko (./hello.ko)<br /> Contains modules:<br />  Id Name<br />  462 hello<br />[n0fate@FreeBSD ~/rootkit/basic]$ <br /></pre>
<p>코드가 정상적으로 로드되었다.
<div>n0fate's Forensic Space :)</div>
