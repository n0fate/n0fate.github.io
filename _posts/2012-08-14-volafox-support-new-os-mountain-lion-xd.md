---
layout: post
title: 'volafox : Support New OS!! Mountain Lion xD'
date: 2012-08-14 10:39:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/08/volafox-support-new-os-mountain-lion-xd.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/405726472750972596
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '992'
  fusion_builder_content_backup: '이전에 포스팅한 <a href="http://feedbeef.blogspot.kr/2012/08/volafox-volafox-08-release.html"
    target="_blank">volafox : volafox 0.8 release</a>를 공개하고 한 달간은 다른 것좀 보면서 쉴까 했는데,
    몇 일뒤에 마운틴 라이언이 공개되었습니다.<br />사실 애플이 라이언을 출시하고 성능이나 안정성 면에서 욕을 많이 먹은 상태에서 내놓은 운영체제(외국
    IT 관련 블로그 보면 아시겠지만 생각보다 많이 까였습니다.)이다보니 크게 구조가 변경되지 않았을 거란 전제를 두고 테스트를 해봤는데, 일단
    0.8은 제대로 동작하는 기능이 한개도 없더군요-_-; 본 포스팅에선 주요 키워드를 놓고 삽질한 내용에 대해 작성해보겠습니다.<br /><br
    /><h4><b>1. KASLR</b></h4>volafox는 Chris Leat가 제안한 방법을 이용하여 커널 버전 및 아키텍처를 판단합니다.
    커널 소스의 ''<a href="http://opensource.apple.com/source/xnu/xnu-2050.7.9/osfmk/i386/lowmem_vectors.s"
    target="_blank">lowmem_vector.s</a>''에 정의된 ''Catfish '' 시그너처를 찾고 커널 버전 문자열을 저장한
    후에 데이터 구조체의 크기를 확인하여 32비트인지 64비트인지 확인합니다.<br /><br />도구에선 이 부분을 해석하는 과정에서부터 문제가
    발생합니다. 라이언까지만해도 이 위치는 고정적이였고, 저희 도구는 그 위치를 통해 메모리 이미지가 linear 포맷인지 검증하는 루틴이 있었습니다.
    문제는 이 부분이 마운틴 라이언에서는 매번 컴퓨터가 재부팅할 때마다 다른 주소를 가리키다보니 잘못된 이미지라고 판단하는 문제가 있었습니다.
    즉, 커널 심볼 정보 주소가 Randomization 된다는 의미이지요. 그래서 이 부분을 확인해보니, 커널 심볼이 아니라 커널 이미지의 로딩
    주소가 랜덤성을 가진다는 사실을 알 수 있었습니다.<br /><br /><i>(추 후에 <a href="http://twitter.com/beist"
    target="_blank">beist</a>님이 마운틴 라이언에 KASLR과 Kernel Stack Cookies가 추가되었다는 정보를 알려주어서
    더 명백해졌네요. 감사합니다. thx beist!)</i><br /><br />참고로 KASLR은 iOS6에서도 적용될 기술로, 위에서 설명한
    것과 같이 커널 자체를 매번 로드할 때마다 다른 주소로 로드되도록 하는 방법입니다. 이 기법을 이용하면, 단순히 심볼 주소를 이용하여 해당
    커널 자원에 접근하는 방법은 그 효용성이 상당히 떨어지게 되죠.<br /><br />(추측이지만, 지금 생각하기에는 Mac OS X에서는 KASLR을
    EFI를 이용하여 구현하지 않았을까 생각됩니다. EFI가 ExitBootService()를 호출하면서 커널 이미지가 로드되는데 이 때 베이스
    주소를 Randomization하는 것이 가장 편하거든요)<br /><br />여튼 일단 원인은 파악을 했는데, 문제는 베이스 주소를 정량적으로
    판단하는 방법이였습니다. 그냥 단순히 기존 ''lowmem_vector.s''가 위치한 주소를 고정으로 박아서 산출하는 것도 한 방법이겠지만,
    좀 더 새련된 방법을 찾고 싶었습니다. 그래서 이것 저것 보던 와중에 커널 심볼 중에 ''_lowGlo''라는 심볼이 있음을 확인하였고, 이
    심볼의 주소가 lowmem_vector.s의 ''Catfish '' 시그너처를 가리키고 있음을 확인하였습니다. 이 정보를 이용하면 다음과 같은
    방법으로 베이스 주소를 찾을 수 있겠죠.<br /><br /><span>Kernel Base Address = ''Catfish '' Signature
    Address - (lowGlo Symbol Address % 0xFFFFFF80)</span><br /><br />이 방법을 활용하니 어떤
    상황에서도 정상적인 커널 베이스 주소를 확인할 수 있었습니다. 이젠 심볼 주소에 이 베이스 주소를 더해주기만하면, 페이지 테이블을 통해 물리
    메모리 주소를 산출할 수 있게 되는 것이죠 :)<br /><br /><br /><h4><b>2. 두 개의 PML4 테이블 심볼</b></h4>기존엔
    64비트 환경에서 사용하는 페이지 테이블 구조인 IA-32e with PML4의 시작 주소를 가리키는''IdlePML4''만 존재했던 심볼이
    마운틴 라이언에선 BootPML4 와 IdlePML4 두개로 표시되었습니다. 이 이유는 1번의 KASLR 때문입니다. KASLR은 어떤 주소로
    커널이 로드될지 알 수 없기 때문에, BootPML4가 IdlePML4와 같이 커널이 로드된 후에 커널의 여러 자원의 위치를 파악하는데 필요한
    요소를 추적하기 위한 정보를 가지고 있습니다. 즉, 커널 이미지를 추적하기 위한 정보는 BootPML4가 가지고 있는 것이지요. 그림으로 표현하면
    다음과 같습니다.<br /><br /><div><a href="http://3.bp.blogspot.com/-7SO4nmmgNNY/UCmqJMSNhLI/AAAAAAAAAr8/lUdX1xK3YxQ/s1600/1.PNG"
    imageanchor="1"><img border="0" height="316" src="http://3.bp.blogspot.com/-7SO4nmmgNNY/UCmqJMSNhLI/AAAAAAAAAr8/lUdX1xK3YxQ/s640/1.PNG"
    width="640"></a></div><br /><br />즉, BootPML4를 기반으로 페이지 테이블을 작성하고, (IdlePML4 주소
    + 커널 베이스 주소)를 추적하면 IdlePML4의 페이지 테이블 시작 주소를 추적할 수 있게 됩니다. 그러면, 이 페이지 테이블을 기반으로
    다른 데이터를 분석할 수 있게 되는 것이지요.<br /><br />1번과 2번 항목을 적용한 코드는 다음과 같습니다.<br /><br /><br
    /><br /><span><span> </span>if self.build[0:2] == ''12'': # Mountain Lion</span><br
    /><span><span> </span>    self.base_address = self.catfishlocation - (self.symbol_list[''_lowGlo'']
    % 0xFFFFFF80)</span><br /><span><span> </span>    self.bootpml4 = (self.symbol_list[''_BootPML4'']
    % 0xFFFFFF80) + self.base_address</span><br /><span><span> </span>    self.boot_pml4_pt
    = IA32PML4MemoryPae(FileAddressSpace(self.mempath), self.bootpml4)</span><br /><span><span>
    </span>    </span><br /><span><span> </span>    idlepml4_ptr = self.boot_pml4_pt.read(self.symbol_list[''_IdlePML4'']+self.base_address,
    8)</span><br /><span><span> </span>    self.idlepml4 = struct.unpack(''=Q'', idlepml4_ptr)[0]</span><br
    /><br /><br /><br /><h4><b>3. 마운틴 라이언은 64비트만 지원</b></h4>마운틴 라이언은 64비트만 지원합니다.
    EFI이미지와 커널 이미지도 64비트용 바이너리만 저장하고 있습니다. 그래서 이전 버전보다 작업량이 절반으로 줄어들다보니, 모듈 테스트나 수정이
    훨씬 용이하였습니다. 1번과 2번의 코드에서 0xFFFFFF80으로 나눈 나머지 값을 가져오는건 다 이 64비트 주소를 실제 물리 메모리 주소에
    맞게 변경하기 위해서였습니다. 제 입장에선 작업량이 준 것은 환영할 만한 일입니다.<br /><br /><br /><h4><b>마무리..</b></h4>현재
    위 3가지 요소가 반영되어 모든 데이터 구조를 해석 가능한 volafox는 SVN에 올라가 있습니다. 어제 라이언 분석 시 발생하는 마이너
    버그를 수정해서 현재는 리비전84(r.84)이네요. 다행히도 많은 시간 투자 없이 해결이 가능했습니다. 앞으로는 당분간 기존에 벌려놓았던 수
    많은 ''experiment''의 결과를 내는데 주력해야겠습니다. :)<br /><br /><br /><div>n0fate''s Forensic
    Space :)</div>'
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>이전에 포스팅한 <a href="http://feedbeef.blogspot.kr/2012/08/volafox-volafox-08-release.html" target="_blank">volafox : volafox 0.8 release</a>를 공개하고 한 달간은 다른 것좀 보면서 쉴까 했는데, 몇 일뒤에 마운틴 라이언이 공개되었습니다.<br />사실 애플이 라이언을 출시하고 성능이나 안정성 면에서 욕을 많이 먹은 상태에서 내놓은 운영체제(외국 IT 관련 블로그 보면 아시겠지만 생각보다 많이 까였습니다.)이다보니 크게 구조가 변경되지 않았을 거란 전제를 두고 테스트를 해봤는데, 일단 0.8은 제대로 동작하는 기능이 한개도 없더군요-_-; 본 포스팅에선 주요 키워드를 놓고 삽질한 내용에 대해 작성해보겠습니다.</p>
<h4><b>1. KASLR</b></h4>
<p>volafox는 Chris Leat가 제안한 방법을 이용하여 커널 버전 및 아키텍처를 판단합니다. 커널 소스의 '<a href="http://opensource.apple.com/source/xnu/xnu-2050.7.9/osfmk/i386/lowmem_vectors.s" target="_blank">lowmem_vector.s</a>'에 정의된 'Catfish ' 시그너처를 찾고 커널 버전 문자열을 저장한 후에 데이터 구조체의 크기를 확인하여 32비트인지 64비트인지 확인합니다.</p>
<p>도구에선 이 부분을 해석하는 과정에서부터 문제가 발생합니다. 라이언까지만해도 이 위치는 고정적이였고, 저희 도구는 그 위치를 통해 메모리 이미지가 linear 포맷인지 검증하는 루틴이 있었습니다. 문제는 이 부분이 마운틴 라이언에서는 매번 컴퓨터가 재부팅할 때마다 다른 주소를 가리키다보니 잘못된 이미지라고 판단하는 문제가 있었습니다. 즉, 커널 심볼 정보 주소가 Randomization 된다는 의미이지요. 그래서 이 부분을 확인해보니, 커널 심볼이 아니라 커널 이미지의 로딩 주소가 랜덤성을 가진다는 사실을 알 수 있었습니다.</p>
<p><i>(추 후에 <a href="http://twitter.com/beist" target="_blank">beist</a>님이 마운틴 라이언에 KASLR과 Kernel Stack Cookies가 추가되었다는 정보를 알려주어서 더 명백해졌네요. 감사합니다. thx beist!)</i></p>
<p>참고로 KASLR은 iOS6에서도 적용될 기술로, 위에서 설명한 것과 같이 커널 자체를 매번 로드할 때마다 다른 주소로 로드되도록 하는 방법입니다. 이 기법을 이용하면, 단순히 심볼 주소를 이용하여 해당 커널 자원에 접근하는 방법은 그 효용성이 상당히 떨어지게 되죠.</p>
<p>(추측이지만, 지금 생각하기에는 Mac OS X에서는 KASLR을 EFI를 이용하여 구현하지 않았을까 생각됩니다. EFI가 ExitBootService()를 호출하면서 커널 이미지가 로드되는데 이 때 베이스 주소를 Randomization하는 것이 가장 편하거든요)</p>
<p>여튼 일단 원인은 파악을 했는데, 문제는 베이스 주소를 정량적으로 판단하는 방법이였습니다. 그냥 단순히 기존 'lowmem_vector.s'가 위치한 주소를 고정으로 박아서 산출하는 것도 한 방법이겠지만, 좀 더 새련된 방법을 찾고 싶었습니다. 그래서 이것 저것 보던 와중에 커널 심볼 중에 '_lowGlo'라는 심볼이 있음을 확인하였고, 이 심볼의 주소가 lowmem_vector.s의 'Catfish ' 시그너처를 가리키고 있음을 확인하였습니다. 이 정보를 이용하면 다음과 같은 방법으로 베이스 주소를 찾을 수 있겠죠.</p>
<p><span>Kernel Base Address = 'Catfish ' Signature Address - (lowGlo Symbol Address % 0xFFFFFF80)</span></p>
<p>이 방법을 활용하니 어떤 상황에서도 정상적인 커널 베이스 주소를 확인할 수 있었습니다. 이젠 심볼 주소에 이 베이스 주소를 더해주기만하면, 페이지 테이블을 통해 물리 메모리 주소를 산출할 수 있게 되는 것이죠 :)</p>
<p>
<h4><b>2. 두 개의 PML4 테이블 심볼</b></h4>
<p>기존엔 64비트 환경에서 사용하는 페이지 테이블 구조인 IA-32e with PML4의 시작 주소를 가리키는'IdlePML4'만 존재했던 심볼이 마운틴 라이언에선 BootPML4 와 IdlePML4 두개로 표시되었습니다. 이 이유는 1번의 KASLR 때문입니다. KASLR은 어떤 주소로 커널이 로드될지 알 수 없기 때문에, BootPML4가 IdlePML4와 같이 커널이 로드된 후에 커널의 여러 자원의 위치를 파악하는데 필요한 요소를 추적하기 위한 정보를 가지고 있습니다. 즉, 커널 이미지를 추적하기 위한 정보는 BootPML4가 가지고 있는 것이지요. 그림으로 표현하면 다음과 같습니다.</p>
<div><a href="http://3.bp.blogspot.com/-7SO4nmmgNNY/UCmqJMSNhLI/AAAAAAAAAr8/lUdX1xK3YxQ/s1600/1.PNG" imageanchor="1"><img border="0" height="316" src="{{ site.baseurl }}/assets/1.PNG" width="640" /></a></div>
<p>즉, BootPML4를 기반으로 페이지 테이블을 작성하고, (IdlePML4 주소 + 커널 베이스 주소)를 추적하면 IdlePML4의 페이지 테이블 시작 주소를 추적할 수 있게 됩니다. 그러면, 이 페이지 테이블을 기반으로 다른 데이터를 분석할 수 있게 되는 것이지요.</p>
<p>1번과 2번 항목을 적용한 코드는 다음과 같습니다.</p>
<p><span><span> </span>if self.build[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][0:2] == '12': # Mountain Lion</span><br /><span><span> </span>    self.base_address = self.catfishlocation - (self.symbol_list['_lowGlo'] % 0xFFFFFF80)</span><br /><span><span> </span>    self.bootpml4 = (self.symbol_list['_BootPML4'] % 0xFFFFFF80) + self.base_address</span><br /><span><span> </span>    self.boot_pml4_pt = IA32PML4MemoryPae(FileAddressSpace(self.mempath), self.bootpml4)</span><br /><span><span> </span>    </span><br /><span><span> </span>    idlepml4_ptr = self.boot_pml4_pt.read(self.symbol_list['_IdlePML4']+self.base_address, 8)</span><br /><span><span> </span>    self.idlepml4 = struct.unpack('=Q', idlepml4_ptr)[0]</span></p>
<h4><b>3. 마운틴 라이언은 64비트만 지원</b></h4>
<p>마운틴 라이언은 64비트만 지원합니다. EFI이미지와 커널 이미지도 64비트용 바이너리만 저장하고 있습니다. 그래서 이전 버전보다 작업량이 절반으로 줄어들다보니, 모듈 테스트나 수정이 훨씬 용이하였습니다. 1번과 2번의 코드에서 0xFFFFFF80으로 나눈 나머지 값을 가져오는건 다 이 64비트 주소를 실제 물리 메모리 주소에 맞게 변경하기 위해서였습니다. 제 입장에선 작업량이 준 것은 환영할 만한 일입니다.</p>
<p>
<h4><b>마무리..</b></h4>
<p>현재 위 3가지 요소가 반영되어 모든 데이터 구조를 해석 가능한 volafox는 SVN에 올라가 있습니다. 어제 라이언 분석 시 발생하는 마이너 버그를 수정해서 현재는 리비전84(r.84)이네요. 다행히도 많은 시간 투자 없이 해결이 가능했습니다. 앞으로는 당분간 기존에 벌려놓았던 수 많은 'experiment'의 결과를 내는데 주력해야겠습니다. :)</p>
<p>
<div>n0fate's Forensic Space :)</div>
<p>[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
