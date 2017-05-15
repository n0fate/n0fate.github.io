---
layout: post
title: 'volafox project: distorm library'
date: 2011-07-24 20:30:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2011/07/volafox-project-distorm-library.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/2714712275802247599
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1614'
  fusion_builder_content_backup: distorm library는 Intel & AMD 환경의 코드의 디스어셈블 기능을 제공하는
    라이브러리로 상용과 프리웨어로 나뉩니다. volafox project에서 해당 내용을 다루는 이유는 이전 포스트에 설명했던 인라인 함수 후킹을
    탐지하기 위한 방안인 각 시스템 콜 영역의 해시 값을 산출하여 비교하는 방법에 이용하기 위함입니다.<br />코드의 시작 부분은 문제가 되지
    않지만, 코드의 끝을 의미하는 RETN의 명령을 올바르게 해석하기 위해선 디스어셈블을 통해 RETN에 해당 하는 코드인 C3이 opcode인지
    operand인지 확인하는 부분이 필요하기 때문입니다.<br />이에 본 포스팅에서는 간단하게 distorm을 이용하여 특정 콜 넘버의 함수를
    디스어셈블 하는 과정을 담도록 하겠습니다. 참고로 distorm library의 설치는 'setup.py install'을 통해 간단히 설치할
    수 있습니다(맥과 윈도우에서 테스트 되었습니다).<br /><img alt="distorm1.png" border="0" height="123"
    src="http://lh4.ggpht.com/-2f4oYNhGyyA/Tiv9o_ChVII/AAAAAAAAALA/mD87uV27tf8/distorm1.png?imgmax=800"
    title="distorm1.png" width="400"><br />Mac에서 에디터로 작업하다보니 코드가 깔끔하게 붙지 않아서 그냥 이미지로
    올립니다.<br /><br />위 함수의 인풋인자는 리스트 구조의 심볼 목록과 시스템 콜 넘버를 받습니다. 콜 넘버가 일치하면 해당 시스템
    콜 테이블의 시스템 콜 함수의 주소 값을 가지는 'sysent[call_number].sy_call' 의 커널 가상 주소로부터 300바이트의
    데이터를  받아옵니다. 그리고 읽은 데이터를 32비트형태의 디스어셈블 코드로 디코딩 하여 리스트 l에 저장합니다. 여기서 Decode16Bits로
    하면 16비트 instruction으로 해석하며 Decode64Bits로 하면 64비트 Instruction으로 해석합니다.<br />Decode
    함수의 첫번째 인자는 차 후 각 명령어에 대한 올바른 가상 주소를 표현하기 위한 Base Address를 지정합니다. 위의 예에선 'sysent[call_number].sy_call'
    가 베이스 주소가 됩니다.<br />l 리스트의 한 레코드의 각 블럭 정보는 다음과 같습니다.<br /><blockquote><em><span>[Base
    Address+Code Offset][sizeof(raw_code)][raw_code][Decoded Disassemble Instruction]</span></em></blockquote>이런
    정보를 잘 정리해서 뿌려주는게 위의 코드가 되겠습니다. 해석 종료는 '해석한 디스어셈블 명령'이 RET인지를 체크하여 결정합니다.<br />위의
    과정을 통해 2번째 시스템 콜(fork)을 해석한 화면은 아래와 같습니다.<br /><img alt="distorm2.png" border="0"
    height="283" src="http://lh6.ggpht.com/-krL1MvYWmAE/TiwB5dd8ETI/AAAAAAAAALI/ssWbkxRtO8M/distorm2.png?imgmax=800"
    title="distorm2.png" width="400"><br />코드가 정상적으로 동작하는 것을 확인할 수 있습니다.<br />해당 모듈은
    아직 volafox project에 추가하지 않았습니다. 현재 이것저것 다른 일을 수행하다보니 우선은 inline function hooking
    탐지 모듈이 완벽하게 구현되면 업로드할 예정입니다. distorm library를 설치함으로 인해 예전 volafox project의 장점인
    python만 설치하고 동작할 수 있는 간결성은 떨어졌지만, 현존하는 시스템 콜 함수 후킹 기법을 모두 탐지할 수 있기 때문에 루트킷 탐지에
    더욱 효율성이 높아질 수 있을 것이라 생각합니다.<div>n0fate's Forensic Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>distorm library는 Intel & AMD 환경의 코드의 디스어셈블 기능을 제공하는 라이브러리로 상용과 프리웨어로 나뉩니다. volafox project에서 해당 내용을 다루는 이유는 이전 포스트에 설명했던 인라인 함수 후킹을 탐지하기 위한 방안인 각 시스템 콜 영역의 해시 값을 산출하여 비교하는 방법에 이용하기 위함입니다.<br />코드의 시작 부분은 문제가 되지 않지만, 코드의 끝을 의미하는 RETN의 명령을 올바르게 해석하기 위해선 디스어셈블을 통해 RETN에 해당 하는 코드인 C3이 opcode인지 operand인지 확인하는 부분이 필요하기 때문입니다.<br />이에 본 포스팅에서는 간단하게 distorm을 이용하여 특정 콜 넘버의 함수를 디스어셈블 하는 과정을 담도록 하겠습니다. 참고로 distorm library의 설치는 'setup.py install'을 통해 간단히 설치할 수 있습니다(맥과 윈도우에서 테스트 되었습니다).<br /><img alt="distorm1.png" border="0" height="123" src="{{ site.baseurl }}/assets/distorm1.png?imgmax=800" title="distorm1.png" width="400" /><br />Mac에서 에디터로 작업하다보니 코드가 깔끔하게 붙지 않아서 그냥 이미지로 올립니다.</p>
<p>위 함수의 인풋인자는 리스트 구조의 심볼 목록과 시스템 콜 넘버를 받습니다. 콜 넘버가 일치하면 해당 시스템 콜 테이블의 시스템 콜 함수의 주소 값을 가지는 'sysent[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][call_number].sy_call' 의 커널 가상 주소로부터 300바이트의 데이터를  받아옵니다. 그리고 읽은 데이터를 32비트형태의 디스어셈블 코드로 디코딩 하여 리스트 l에 저장합니다. 여기서 Decode16Bits로 하면 16비트 instruction으로 해석하며 Decode64Bits로 하면 64비트 Instruction으로 해석합니다.<br />Decode 함수의 첫번째 인자는 차 후 각 명령어에 대한 올바른 가상 주소를 표현하기 위한 Base Address를 지정합니다. 위의 예에선 'sysent[call_number].sy_call' 가 베이스 주소가 됩니다.<br />l 리스트의 한 레코드의 각 블럭 정보는 다음과 같습니다.<br />
<blockquote><em><span>[Base Address+Code Offset][sizeof(raw_code)][raw_code][Decoded Disassemble Instruction]</span></em></p></blockquote>
<p>이런 정보를 잘 정리해서 뿌려주는게 위의 코드가 되겠습니다. 해석 종료는 '해석한 디스어셈블 명령'이 RET인지를 체크하여 결정합니다.<br />위의 과정을 통해 2번째 시스템 콜(fork)을 해석한 화면은 아래와 같습니다.<br /><img alt="distorm2.png" border="0" height="283" src="{{ site.baseurl }}/assets/distorm2.png?imgmax=800" title="distorm2.png" width="400" /><br />코드가 정상적으로 동작하는 것을 확인할 수 있습니다.<br />해당 모듈은 아직 volafox project에 추가하지 않았습니다. 현재 이것저것 다른 일을 수행하다보니 우선은 inline function hooking 탐지 모듈이 완벽하게 구현되면 업로드할 예정입니다. distorm library를 설치함으로 인해 예전 volafox project의 장점인 python만 설치하고 동작할 수 있는 간결성은 떨어졌지만, 현존하는 시스템 콜 함수 후킹 기법을 모두 탐지할 수 있기 때문에 루트킷 탐지에 더욱 효율성이 높아질 수 있을 것이라 생각합니다.
<div>n0fate's Forensic Space :)</div>
<p>[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
