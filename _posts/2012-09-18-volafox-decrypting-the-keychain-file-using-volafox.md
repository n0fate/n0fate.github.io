---
layout: post
title: 'volafox : decrypting the keychain file using volafox'
date: 2012-09-18 16:11:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Memory Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/09/volafox-decrypting-keychain-file-using.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/8236219222788022346
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '956'
  fusion_builder_content_backup: |-
    <div>Mac OS X에 대해 연구를 하신 분들은 아시겠지만, 맥은 사용자 패스워드 관리에 키체인(Keychain)을 이용합니다. 이 파일은 당연히 암호화 되어 있으며, 복호화를 할 경우엔 사용자가 등록한 여러 계정에 대한 계정명과 패스워드를 획득할 수 있는 중요 데이터 입니다.</div><div>키체인에 저장되는 데이터는 주로 애플 메일 계정 정보, 메신저 계정 정보, SSH 접속 계정 정보 등 사용자가 보관한 모든 정보가 있습니다.</div><div><br /></div>이 키체인과 관련하여 얼마전에 트위터를 통해 <a href="http://juusosalonen.com/post/30923743427/breaking-into-the-os-x-keychain" target="_blank">Breaking In To The OS X Keychain</a> 이라는 글이 돌았습니다. 글의 내용을 간단하게 요약하면 다음과 같습니다.<br /><div><ol><li>OS X Lion 부터는 키체인 정보에 접근할 때 매번 물어보는 것이 아니라 처음에 한번 인증하고나면 사용자에게 접근 허용 여부만을 물어봄.</li><li>이는 사용자 패스워드 정보 또는 사용자 패스워드를 기반으로하는 특정 키를 메모리에 저장하고 있을 가능성이 높음.</li><li>확인해보니 메모리에는 패스워드는 존재하지 않으며, 이는 특정 키를 사용하고 있음을 의미함.</li><li>블로거가 분석해보니 24바이트(192bit)의 마스터 키를 사용함을 알 수 있었음.</li><li>securityd 프로세스의 힙 영역 중 MALLOC_TINY 영역(1메가 크기)에 키를 저장함을 확인 (루트 권한 필요)</li><li>특정 시그너처를 이용하여(0x18) 마스터 키 후보(master key candidate)를 찾음</li><li>마스터 키로 keychain 파일의 encrypted wrapping key가 올바르게 복호화 되는지 확인해서 제대로된 master key를 찾음.</li><li>keychain 파일 복호화 함.</li></ol></div><div><br /></div><div>나름 간단하게 요약 했는데도 복잡하네요. 한줄로 요약하면, '활성 시스템에서 사용자 패스워드 한번 입력하면, 키체인 마스터 키를 획득할 수 있고, 이것으로 키체인을 복호화할 수 있다.' 입니다. 단 여기에는 sudo 권한이 필요하다는 치명적인 조건이 필요합니다. </div><div><br /></div><div>사실 사용자 패스워드를 알면 이렇게 할 것도 없이 다음 그림과 같이 '키체인 접근'이라는 프로그램을 이용해서도 패스워드를 바로 복호화할 수 있습니다.</div><div><br /></div><div><a href="http://2.bp.blogspot.com/-5gGAoIQgbHQ/UFgX0Pc9nvI/AAAAAAAAAsg/7jX2ZS-VBHY/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2012-09-18+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+3.41.59.png" imageanchor="1"><img border="0" height="173" src="http://2.bp.blogspot.com/-5gGAoIQgbHQ/UFgX0Pc9nvI/AAAAAAAAAsg/7jX2ZS-VBHY/s400/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2012-09-18+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+3.41.59.png" width="400"></a></div><div><br /></div><div><br /></div><div>그런데 이 기술을 포렌식 분야에 적용하면 얘기가 달라집니다. 최근에 디스크 용량 이슈 또는 루트킷으로 인한 라이브 정보의 신뢰성 이슈가 있다보니 메모리 이미징을 뜨는 경우가 많아지고 있는데요, 메모리 이미징 시 FW 케이블을 이용하면, 사용자 권한 여부는 상관없이 라이브 상태만 유지되고 있다면 별다른 제약없이 이미징을 수행할 수 있습니다.</div><div>여기서 중요한 점은 이 과정에서 사용자 권한을 알 필요가없다는 점입니다. 과정은 다음과 같습니다.</div><div><br /></div><div><ul><li>라이브 시스템에 FW를 연결하여 메모리 이미징을 수행한다. (권한 필요 없음, 케이블을 연결해야 하므로, 시스템 물리적 접근 필요)</li><li>시스템을 강제 종료하고 디스크 이미징을 수행한다. (권한 필요 없음, 시스템 분해 필요)</li><ul><li>필요 시엔 FW를 이용하여 Target Disk Mode로 부팅 후 keychain 파일만 추출한다.(분해 필요 없음)</li></ul></ul></div><div><br /></div><div>이 두 과정에는 별도의 권한이 필요 없기 때문에 조사관은 두 이미지를 추출하여 키체인을 복호화할 수 있게 됩니다. 앞서 과정에서 결론적으로 키체인 복호화에 필요한 데이터는 다음과 같습니다.</div><div><br /></div><div><ul><li>securityd 프로세스의 MALLOC_TINY 힙 영역에 있는 마스터 키 후보군(192bit)</li><li>디스크 이미지에 있는 사용자의 키체인 파일(~/Library/Keychain/login.keychain)</li></ul></div><div>첫 번째 키는 메모리 이미지를 분석해서 securityd의 프로세스 영역을 추출한 다음에 얻을 수 있습니다. volafox project의 svn revision에는 앞서 링크한 블로그 작성된 절차를 volafox의 메모리 분석에 적용한 keychaindump 명령어로 획득할 수 있습니다. 다음은 실행 예제입니다.</div><div><br /></div><div><div><span>n0fates-MacBook-Pro:volafox n0fate$ python vol.py -i ~/Desktop/Mac OS X 10.8-c139ff1b.vmem -o keychaindump</span></div><div><span><br /></span></div><div><span>[+] Virtual Memory Map Information</span></div><div><span> [-] Virtual Address Start Point: 0x1099e1000</span></div><div><span> [-] Virtual Address End Point: 0x7fffffe00000</span></div><div><span> [-] Number of Entries: 92</span></div><div><span><br /></span></div><div><span>[+] Generating Process Virtual Memory Maps</span></div><div><span> [-] Region from 0x1099e1000 to 0x109aea000 (r-x, max rwx;)</span></div><div><span> [-] Region from 0x109aea000 to 0x109af7000 (rw-, max rwx;)</span></div><div><span> [-] Region from 0x109af7000 to 0x109b12000 (r--, max rwx;)</span></div><div><span> [-] Region from 0x109b12000 to 0x109b13000 (r--, max rwx;)</span></div><div><span> ...</span></div><div><span> [-] Region from 0x10a0ce000 to 0x10b2f3000 (r--, max r-x;)</span></div><div><span> [-] Region from 0x7fedc9c00000 to 0x7fedc9d00000 (rw-, max rwx;)</span></div><div><span> [-] Region from 0x7fedc9d00000 to 0x7fedc9e00000 (rw-, max rwx;)</span></div><div><span> [-] Region from 0x7fedc9e00000 to 0x7fedc9f00000 (rw-, max rwx;)</span></div><div><span> [-] Region from 0x7fedca000000 to 0x7fedca0e5000 (rw-, max rwx;)</span></div><div><span> [-] Region from 0x7fedca0e5000 to 0x7fedca0e7000 (rw-, max rwx;)</span></div><div><span> [-] Region from 0x7fedca0e7000 to 0x7fedca0f8000 (rw-, max rwx;)</span></div><div><span> [-] Region from 0x7fedca0f8000 to 0x7fedca0fa000 (rw-, max rwx;)</span></div><div><span> ...</span></div><div><span> [-] Region from 0x7fffc0000000 to 0x7fffffe00000 (r--, max rwx;)</span></div><div><span> [-] Region from 0x7fffffe00000 to 0x7fffffe01000 (r--, max r--;)</span></div><div><span> [-] Region from 0x7ffffffca000 to 0x7ffffffcb000 (r-x, max r-x;)</span></div><div><span><br /></span></div><div><span>[+] Find MALLOC_TINY heap range (guess)</span></div><div><span> [-] range 0x7fedc9c00000-0x7fedc9d00000</span></div><div><span> [-] range 0x7fedc9d00000-0x7fedc9e00000</span></div><div><span> [-] range 0x7fedc9e00000-0x7fedc9f00000</span></div><div><span> [-] range 0x7fedcb000000-0x7fedcb100000</span></div><div><span> [-] range 0x7fedcb100000-0x7fedcb200000</span></div><div><span> [-] range 0x7fedcb200000-0x7fedcb300000</span></div><div><span><br /></span></div><div><span>[*] Search for keys in range 0x7fedc9c00000-0x7fedc9d00000 complete. master key candidates : 4</span></div><div><span>[*] Search for keys in range 0x7fedc9d00000-0x7fedc9e00000 complete. master key candidates : 4</span></div><div><span>[*] Search for keys in range 0x7fedc9e00000-0x7fedc9f00000 complete. master key candidates : 4</span></div><div><span>[*] Search for keys in range 0x7fedcb000000-0x7fedcb100000 complete. master key candidates : 5</span></div><div><span>[*] Search for keys in range 0x7fedcb100000-0x7fedcb200000 complete. master key candidates : 5</span></div><div><span>[*] Search for keys
    in range 0x7fedcb200000-0x7fed
    cb300000 complete. master key candidates : 5</span></div><div><span><br /></span></div><div><span>[*] master key candidate: ACEFBB5CECE8398E780016DAF685E39A8E36FDE8EC8B6B6E</span></div><div><span>[*] master key candidate: C7A8DC12E05F1BEEF70228C304E6567CFDA07BEAB86606C7</span></div><div><span>[*] master key candidate: E340E90C2EC1E55BC116A2775624FC3D4ADD35E248E543C1</span></div><div><span>[*] master key candidate: 9AB40E7378E195335019C5A3577123D5AC814C7D22588B2F</span></div><div><span>[*] master key candidate: 9AB40E7378E195335019C5A3577123D5AC814C7D22588B2F</span></div></div><div><br /></div><div>확인된 마스터 키 후보를 이용하여 원저작자의 <a href="https://github.com/juuso/keychaindump" target="_blank">keychaindump</a>를 volafox 출력에 맞게 수정한 <a href="https://github.com/n0fate/keychaindump" target="_blank">memimgkcdump</a>로 복호화를 수행할 수 있습니다.</div><div><br /></div><div><div><span>n0fates-MacBook-Pro:keychaindump n0fate$ ./memimgkcdump E340E90C2EC1E55BC116A2775624FC3D4ADD35E248E543C1 ~/login.keychain</span></div><div><span><br /></span></div><div><span>[*] Trying to decrypt wrapping key in /Users/n0fate/login.keychain</span></div><div><span>[*] Trying master key candidate: E340E90C2EC1E55BC116A2775624FC3D4ADD35E248E543C1</span></div><div><span>[+] Found master key: E340E90C2EC1E55BC116A2775624FC3D4ADD35E248E543C1</span></div><div><span>[+] Found wrapping key: 9ab40e7378e195335019c5a3577123d5ac814c7d22588b2f</span></div><div><span><br /></span></div><div><span>xxx@livx.com:pop3.livx.xxx:xxxxx</span></div><div><span>xxx@livx.com:smtp.livx.xxx:xxxxx</span></div></div><div><br /></div><div>다음과 같이 키체인에 저장된 사용자 계정과 패스워드를 확인할 수 있습니다.</div><div><br /></div><div>이 방법을 이용하면 키체인에 등록된 모든 사용자의 계정 정보와 패스워드를 추출할 수 있으며, 특정 서버에 SSH 또는 SFTP를 이용하여 접근 여부와 계정 정보를 모두 수집할 수 있습니다.</div><div><br /></div><div>아직 MALLOC_TINY 영역을 추측해서 추출하는 등, 추후 에러가 발생할 소지는 많지만, <b>선의의</b> 목적으로 포렌식 기술을 사용하는 분들에게 이 방법이 큰 도움이 되길 바랍니다. :-)</div><div><br /></div><div><b><i>해당 모듈은 Mac OS X Lion, Mountain Lion 운영체제의 인텔 64비트 아키텍처 환경에서 테스트가 완료되었습니다.</i></b> </div><div>n0fate's Forensic Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<div>Mac OS X에 대해 연구를 하신 분들은 아시겠지만, 맥은 사용자 패스워드 관리에 키체인(Keychain)을 이용합니다. 이 파일은 당연히 암호화 되어 있으며, 복호화를 할 경우엔 사용자가 등록한 여러 계정에 대한 계정명과 패스워드를 획득할 수 있는 중요 데이터 입니다.</div>
<div>키체인에 저장되는 데이터는 주로 애플 메일 계정 정보, 메신저 계정 정보, SSH 접속 계정 정보 등 사용자가 보관한 모든 정보가 있습니다.</div>
<div></div>
<p>이 키체인과 관련하여 얼마전에 트위터를 통해 <a href="http://juusosalonen.com/post/30923743427/breaking-into-the-os-x-keychain" target="_blank">Breaking In To The OS X Keychain</a> 이라는 글이 돌았습니다. 글의 내용을 간단하게 요약하면 다음과 같습니다.
<div>
<ol>
<li>OS X Lion 부터는 키체인 정보에 접근할 때 매번 물어보는 것이 아니라 처음에 한번 인증하고나면 사용자에게 접근 허용 여부만을 물어봄.</li>
<li>이는 사용자 패스워드 정보 또는 사용자 패스워드를 기반으로하는 특정 키를 메모리에 저장하고 있을 가능성이 높음.</li>
<li>확인해보니 메모리에는 패스워드는 존재하지 않으며, 이는 특정 키를 사용하고 있음을 의미함.</li>
<li>블로거가 분석해보니 24바이트(192bit)의 마스터 키를 사용함을 알 수 있었음.</li>
<li>securityd 프로세스의 힙 영역 중 MALLOC_TINY 영역(1메가 크기)에 키를 저장함을 확인 (루트 권한 필요)</li>
<li>특정 시그너처를 이용하여(0x18) 마스터 키 후보(master key candidate)를 찾음</li>
<li>마스터 키로 keychain 파일의 encrypted wrapping key가 올바르게 복호화 되는지 확인해서 제대로된 master key를 찾음.</li>
<li>keychain 파일 복호화 함.</li>
</ol>
</div>
<div></div>
<div>나름 간단하게 요약 했는데도 복잡하네요. 한줄로 요약하면, '활성 시스템에서 사용자 패스워드 한번 입력하면, 키체인 마스터 키를 획득할 수 있고, 이것으로 키체인을 복호화할 수 있다.' 입니다. 단 여기에는 sudo 권한이 필요하다는 치명적인 조건이 필요합니다. </div>
<div></div>
<div>사실 사용자 패스워드를 알면 이렇게 할 것도 없이 다음 그림과 같이 '키체인 접근'이라는 프로그램을 이용해서도 패스워드를 바로 복호화할 수 있습니다.</div>
<div></div>
<div><a href="http://2.bp.blogspot.com/-5gGAoIQgbHQ/UFgX0Pc9nvI/AAAAAAAAAsg/7jX2ZS-VBHY/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2012-09-18+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+3.41.59.png" imageanchor="1"><img border="0" height="173" src="{{ site.baseurl }}/assets/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA+2012-09-18+%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE+3.41.59.png" width="400" /></a></div>
<div></div>
<div></div>
<div>그런데 이 기술을 포렌식 분야에 적용하면 얘기가 달라집니다. 최근에 디스크 용량 이슈 또는 루트킷으로 인한 라이브 정보의 신뢰성 이슈가 있다보니 메모리 이미징을 뜨는 경우가 많아지고 있는데요, 메모리 이미징 시 FW 케이블을 이용하면, 사용자 권한 여부는 상관없이 라이브 상태만 유지되고 있다면 별다른 제약없이 이미징을 수행할 수 있습니다.</div>
<div>여기서 중요한 점은 이 과정에서 사용자 권한을 알 필요가없다는 점입니다. 과정은 다음과 같습니다.</div>
<div></div>
<div>
<ul>
<li>라이브 시스템에 FW를 연결하여 메모리 이미징을 수행한다. (권한 필요 없음, 케이블을 연결해야 하므로, 시스템 물리적 접근 필요)</li>
<li>시스템을 강제 종료하고 디스크 이미징을 수행한다. (권한 필요 없음, 시스템 분해 필요)</li>
</ul>
<ul>
<li>필요 시엔 FW를 이용하여 Target Disk Mode로 부팅 후 keychain 파일만 추출한다.(분해 필요 없음)</li>
</ul>
</div>
<div></div>
<div>이 두 과정에는 별도의 권한이 필요 없기 때문에 조사관은 두 이미지를 추출하여 키체인을 복호화할 수 있게 됩니다. 앞서 과정에서 결론적으로 키체인 복호화에 필요한 데이터는 다음과 같습니다.</div>
<div></div>
<div>
<ul>
<li>securityd 프로세스의 MALLOC_TINY 힙 영역에 있는 마스터 키 후보군(192bit)</li>
<li>디스크 이미지에 있는 사용자의 키체인 파일(~/Library/Keychain/login.keychain)</li>
</ul>
</div>
<div>첫 번째 키는 메모리 이미지를 분석해서 securityd의 프로세스 영역을 추출한 다음에 얻을 수 있습니다. volafox project의 svn revision에는 앞서 링크한 블로그 작성된 절차를 volafox의 메모리 분석에 적용한 keychaindump 명령어로 획득할 수 있습니다. 다음은 실행 예제입니다.</div>
<div></div>
<div>
<div><span>n0fates-MacBook-Pro:volafox n0fate$ python vol.py -i ~/Desktop/Mac OS X 10.8-c139ff1b.vmem -o keychaindump</span></div>
<div><span><br /></span></div>
<div><span>[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][+] Virtual Memory Map Information</span></div>
<div><span> [-] Virtual Address Start Point: 0x1099e1000</span></div>
<div><span> [-] Virtual Address End Point: 0x7fffffe00000</span></div>
<div><span> [-] Number of Entries: 92</span></div>
<div><span><br /></span></div>
<div><span>[+] Generating Process Virtual Memory Maps</span></div>
<div><span> [-] Region from 0x1099e1000 to 0x109aea000 (r-x, max rwx;)</span></div>
<div><span> [-] Region from 0x109aea000 to 0x109af7000 (rw-, max rwx;)</span></div>
<div><span> [-] Region from 0x109af7000 to 0x109b12000 (r--, max rwx;)</span></div>
<div><span> [-] Region from 0x109b12000 to 0x109b13000 (r--, max rwx;)</span></div>
<div><span> ...</span></div>
<div><span> [-] Region from 0x10a0ce000 to 0x10b2f3000 (r--, max r-x;)</span></div>
<div><span> [-] Region from 0x7fedc9c00000 to 0x7fedc9d00000 (rw-, max rwx;)</span></div>
<div><span> [-] Region from 0x7fedc9d00000 to 0x7fedc9e00000 (rw-, max rwx;)</span></div>
<div><span> [-] Region from 0x7fedc9e00000 to 0x7fedc9f00000 (rw-, max rwx;)</span></div>
<div><span> [-] Region from 0x7fedca000000 to 0x7fedca0e5000 (rw-, max rwx;)</span></div>
<div><span> [-] Region from 0x7fedca0e5000 to 0x7fedca0e7000 (rw-, max rwx;)</span></div>
<div><span> [-] Region from 0x7fedca0e7000 to 0x7fedca0f8000 (rw-, max rwx;)</span></div>
<div><span> [-] Region from 0x7fedca0f8000 to 0x7fedca0fa000 (rw-, max rwx;)</span></div>
<div><span> ...</span></div>
<div><span> [-] Region from 0x7fffc0000000 to 0x7fffffe00000 (r--, max rwx;)</span></div>
<div><span> [-] Region from 0x7fffffe00000 to 0x7fffffe01000 (r--, max r--;)</span></div>
<div><span> [-] Region from 0x7ffffffca000 to 0x7ffffffcb000 (r-x, max r-x;)</span></div>
<div><span><br /></span></div>
<div><span>[+] Find MALLOC_TINY heap range (guess)</span></div>
<div><span> [-] range 0x7fedc9c00000-0x7fedc9d00000</span></div>
<div><span> [-] range 0x7fedc9d00000-0x7fedc9e00000</span></div>
<div><span> [-] range 0x7fedc9e00000-0x7fedc9f00000</span></div>
<div><span> [-] range 0x7fedcb000000-0x7fedcb100000</span></div>
<div><span> [-] range 0x7fedcb100000-0x7fedcb200000</span></div>
<div><span> [-] range 0x7fedcb200000-0x7fedcb300000</span></div>
<div><span><br /></span></div>
<div><span>[*] Search for keys in range 0x7fedc9c00000-0x7fedc9d00000 complete. master key candidates : 4</span></div>
<div><span>[*] Search for keys in range 0x7fedc9d00000-0x7fedc9e00000 complete. master key candidates : 4</span></div>
<div><span>[*] Search for keys in range 0x7fedc9e00000-0x7fedc9f00000 complete. master key candidates : 4</span></div>
<div><span>[*] Search for keys in range 0x7fedcb000000-0x7fedcb100000 complete. master key candidates : 5</span></div>
<div><span>[*] Search for keys in range 0x7fedcb100000-0x7fedcb200000 complete. master key candidates : 5</span></div>
<div><span>[*] Search for keys<br />
in range 0x7fedcb200000-0x7fed<br />
cb300000 complete. master key candidates : 5</span></div>
<div><span><br /></span></div>
<div><span>[*] master key candidate: ACEFBB5CECE8398E780016DAF685E39A8E36FDE8EC8B6B6E</span></div>
<div><span>[*] master key candidate: C7A8DC12E05F1BEEF70228C304E6567CFDA07BEAB86606C7</span></div>
<div><span>[*] master key candidate: E340E90C2EC1E55BC116A2775624FC3D4ADD35E248E543C1</span></div>
<div><span>[*] master key candidate: 9AB40E7378E195335019C5A3577123D5AC814C7D22588B2F</span></div>
<div><span>[*] master key candidate: 9AB40E7378E195335019C5A3577123D5AC814C7D22588B2F</span></div>
</div>
<div></div>
<div>확인된 마스터 키 후보를 이용하여 원저작자의 <a href="https://github.com/juuso/keychaindump" target="_blank">keychaindump</a>를 volafox 출력에 맞게 수정한 <a href="https://github.com/n0fate/keychaindump" target="_blank">memimgkcdump</a>로 복호화를 수행할 수 있습니다.</div>
<div></div>
<div>
<div><span>n0fates-MacBook-Pro:keychaindump n0fate$ ./memimgkcdump E340E90C2EC1E55BC116A2775624FC3D4ADD35E248E543C1 ~/login.keychain</span></div>
<div><span><br /></span></div>
<div><span>[*] Trying to decrypt wrapping key in /Users/n0fate/login.keychain</span></div>
<div><span>[*] Trying master key candidate: E340E90C2EC1E55BC116A2775624FC3D4ADD35E248E543C1</span></div>
<div><span>[+] Found master key: E340E90C2EC1E55BC116A2775624FC3D4ADD35E248E543C1</span></div>
<div><span>[+] Found wrapping key: 9ab40e7378e195335019c5a3577123d5ac814c7d22588b2f</span></div>
<div><span><br /></span></div>
<div><span>xxx@livx.com:pop3.livx.xxx:xxxxx</span></div>
<div><span>xxx@livx.com:smtp.livx.xxx:xxxxx</span></div>
</div>
<div></div>
<div>다음과 같이 키체인에 저장된 사용자 계정과 패스워드를 확인할 수 있습니다.</div>
<div></div>
<div>이 방법을 이용하면 키체인에 등록된 모든 사용자의 계정 정보와 패스워드를 추출할 수 있으며, 특정 서버에 SSH 또는 SFTP를 이용하여 접근 여부와 계정 정보를 모두 수집할 수 있습니다.</div>
<div></div>
<div>아직 MALLOC_TINY 영역을 추측해서 추출하는 등, 추후 에러가 발생할 소지는 많지만, <b>선의의</b> 목적으로 포렌식 기술을 사용하는 분들에게 이 방법이 큰 도움이 되길 바랍니다. :-)</div>
<div></div>
<div><b><i>해당 모듈은 Mac OS X Lion, Mountain Lion 운영체제의 인텔 64비트 아키텍처 환경에서 테스트가 완료되었습니다.</i></b> </div>
<div>n0fate's Forensic Space :)</div>
<p>[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
