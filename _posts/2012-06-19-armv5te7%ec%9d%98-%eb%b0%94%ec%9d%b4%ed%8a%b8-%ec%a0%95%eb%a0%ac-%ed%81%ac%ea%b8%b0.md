---
layout: post
title: ARMv5TE7의 바이트 정렬 크기
date: 2012-06-19 10:53:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- ARM
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/06/armv5te7.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/2395731175763183666
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1021'
  fusion_builder_content_backup: '요즘 임베디드 쪽 삽질 동기부여를 위해 강의를 하나 듣고 있다.<br /><div><br
    /></div><div>대부분이 인텔 아키텍처와 상당히 유사해서 100 중 30정도만 듣다가, 심심한차에 ARM의 바이트 정렬은 몇 바이트를
    기준으로 하는지가 궁금해졌다. 테스트에서 사용하는 머신이 32비트 ARM 코어(ARMv5TE7)이다보니 인텔과 같이 4바이트 일 것 같긴한데,
    증명 차원에서 예제 코드를 작성하여 확인하였다.</div><div><br /></div><div>개발 툴킷은 ARM Software Development
    Toolkit(SDT)를 사용하였다.</div><div><br /></div><div>샘플 코드는 다음과 같다.</div><div><br /></div><div><br
    /></div><div><div><span>#include <stdio.h></span></div><div><span><br /></span></div><div><span>typedef
    struct _test{</span></div><div><span><span> </span>char ch;</span></div><div><span><span>
    </span>int data;</span></div><div><span>} TEST;</span></div><div><span><br /></span></div><div><span>int
    main(void)</span></div><div><span>{</span></div><div><span><span> </span>TEST
    test;</span></div><div><span><span> </span>test.ch = ''c'';</span></div><div><span><span>
    </span>test.data = 3;</span></div><div><span><span> </span>printf("size: 0x%pn",
    &(test.ch));</span></div><div><span><span> </span>printf("size: 0x%pn", &(test.data));</span></div><div><span>}</span></div><div><br
    /></div><div><br /></div></div><div>간단하게 구조체 내의 두 바이트의 주소를 확인하는 코드이다. 구조체를 전역으로
    선언한게 아니기 때문에 해당 구조체는 스택에 할당될 것이다.</div><div><br /></div><div>디버거를 통해 확인하였다.</div><div><br
    /></div><div><a href="http://2.bp.blogspot.com/-vLwAwTgGPrE/T9_bR27rqoI/AAAAAAAAATg/-nbO9BgkCmM/s1600/arm2.png"
    imageanchor="1"><img border="0" height="409" src="http://2.bp.blogspot.com/-vLwAwTgGPrE/T9_bR27rqoI/AAAAAAAAATg/-nbO9BgkCmM/s640/arm2.png"
    width="640"></a></div><div><br /></div><div><br /></div><div>역시 예상대로 4바이트 정렬을
    사용한다 :-)</div><div><br /></div><div><br /></div><div><br /></div><div>n0fate''s
    Forensic Space :)</div>'
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>요즘 임베디드 쪽 삽질 동기부여를 위해 강의를 하나 듣고 있다.
<div></div>
<div>대부분이 인텔 아키텍처와 상당히 유사해서 100 중 30정도만 듣다가, 심심한차에 ARM의 바이트 정렬은 몇 바이트를 기준으로 하는지가 궁금해졌다. 테스트에서 사용하는 머신이 32비트 ARM 코어(ARMv5TE7)이다보니 인텔과 같이 4바이트 일 것 같긴한데, 증명 차원에서 예제 코드를 작성하여 확인하였다.</div>
<div></div>
<div>개발 툴킷은 ARM Software Development Toolkit(SDT)를 사용하였다.</div>
<div></div>
<div>샘플 코드는 다음과 같다.</div>
<div></div>
<div></div>
<div>
<div><span>#include <stdio.h /></span></div>
<div><span><br /></span></div>
<div><span>typedef struct _test{</span></div>
<div><span><span> </span>char ch;</span></div>
<div><span><span> </span>int data;</span></div>
<div><span>} TEST;</span></div>
<div><span><br /></span></div>
<div><span>int main(void)</span></div>
<div><span>{</span></div>
<div><span><span> </span>TEST test;</span></div>
<div><span><span> </span>test.ch = 'c';</span></div>
<div><span><span> </span>test.data = 3;</span></div>
<div><span><span> </span>printf("size: 0x%pn", &(test.ch));</span></div>
<div><span><span> </span>printf("size: 0x%pn", &(test.data));</span></div>
<div><span>}</span></div>
<div></div>
<div></div>
</div>
<div>간단하게 구조체 내의 두 바이트의 주소를 확인하는 코드이다. 구조체를 전역으로 선언한게 아니기 때문에 해당 구조체는 스택에 할당될 것이다.</div>
<div></div>
<div>디버거를 통해 확인하였다.</div>
<div></div>
<div><a href="http://2.bp.blogspot.com/-vLwAwTgGPrE/T9_bR27rqoI/AAAAAAAAATg/-nbO9BgkCmM/s1600/arm2.png" imageanchor="1"><img border="0" height="409" src="{{ site.baseurl }}/assets/arm2.png" width="640" /></a></div>
<div></div>
<div></div>
<div>역시 예상대로 4바이트 정렬을 사용한다 :-)</div>
<div></div>
<div></div>
<div></div>
<div>n0fate's Forensic Space :)</div>
