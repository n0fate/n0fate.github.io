---
layout: post
title: Mac OS X Disk Imaging using Single User Mode
date: 2012-03-22 12:37:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Disk Forensics
tags:
- disk forensics
- singleboot
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/03/mac-os-x-disk-imaging-using-single-mode.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/6767134654760893517
  avada_post_views_count: '1838'
  _edit_last: '1'
  slide_template: ''
  fusion_builder_content_backup: '오늘 오전에 <a href="http://baadc0de.blogspot.com/" target="_blank">baadc0de</a>한테
    연락이 왔다. 내용인 즉슨, 맥 운영체제의 패스워드를 모르는 경우 디스크 이미징을 수행하는 방법을 묻는 내용이였다. 사실 디스크를 물리적으로
    추출하여 이미징하는 방법이 있지만, 요청 사례의 경우엔 맥북에어였는데 이 장비에 있는 SSD는 일반적으로 사용되는 SATA 인터페이스와 다른
    구조를 가지고 있다.<br /><img alt="Mba 2011 ssd ogrady2" border="0" height="137" src="http://lh5.ggpht.com/-Alb-7ND0njc/T2qeU3ov5fI/AAAAAAAAAQs/xJ-JfO8UxN0/mba-2011-ssd-ogrady2.jpeg?imgmax=800"
    title="mba-2011-ssd-ogrady2.jpeg" width="500"><br /><span>(reference: http://www.zdnet.com/blog/apple/owc-mercury-on-the-go-enclosure-for-macbook-air-ssd-review/11540)</span><br
    /><br />이 인터페이스는 MINI PCI EXPRESS(ZIF-40핀 인터페이스)라고 불리며, OWC에선 이를 위한 외장 케이스도 있었다.
    :)<br /><br /><img alt="Owc mercury enclosure for macbook air ssd" border="0"
    height="334" src="http://lh6.ggpht.com/-0lFKiMhcIWw/T2qeWMw7o3I/AAAAAAAAAQ0/sqTwoby9pRo/owc-mercury-enclosure-for-macbook-air-ssd.jpeg?imgmax=800"
    title="owc-mercury-enclosure-for-macbook-air-ssd.jpeg" width="500"><br /><span>(reference: http://www.zdnet.com/blog/apple/owc-mercury-on-the-go-enclosure-for-macbook-air-ssd-review/11540)</span><br
    /><br />여튼 이런 상황에 이미징 장치가 SATA, IDE 인터페이스만 가지고 있다면, 별도의 인터페이스를 구매하는 등 까다로운 상황이
    발생할 수 있다.<br /><i><span>코멘트. 포렌식 수사관이신 soomie kim님의 말씀에 의하면, 넷북을 받았는데 MINI PCI
    EX 인터페이스를 사용하여서, 삼성 서비스센터를 통해 MINI PCI EX to SATA PCB를 대여해서 처리한적이 있다고 한다. 최근 울트라씬이다
    뭐다해서 얇은 노트북이 많이 나오다보니, 점점 해당 인터페이스를 접하는 경우가 많을 것이라 생각한다. 만약 어댑터를 구매하고자한다면, Mini
    PCI-e SSD to 2.5" SATA Converter Adapter라는 제품을 찾아보기 바란다.</span></i><br /><br />이
    경우 소프트웨어적인 방법을 활용해야하는데, 맥의 사용자 패스워드를 모르는 상태에서 소프트웨어 기반 디스크 이미징을 수행하기 위해선 다음과 같은
    방법을 생각해볼 수 있다.<br /><ul><li>타겟디스크모드(TDM)를 활용한 디스크 접근</li><li>싱글모드 부팅 후 DD를 이용한
    디스크 이미징</li><li>싱글모드 부팅 후 FW(Firewire)를 이용한 이미지</li></ul>우선 첫 번째 방법은 맥에서만 지원하는
    타겟 디스크 모드를 활용하여, 논리적 디스크에 접근하는 방법으로 FW단자를 활용하여 접근을 수행한다. 이 부분은 맥 포렌식에서 많이 논의된
    부분이니 추가 설명은 생략하겠다. 이 방법은 맥에서 기본적으로 지원하는 기능인만큼 사용하긴 편하지만, 디스크가 RW 속성을 모두 가진 상태에서
    마운트하는 문제가 있다.<br /><br />두 번째 방법은 전통 유닉스 시스템과 같이 싱글모드로 부팅해서 DD로 이미징하는 방법이다. Mac
    OS X는 디스크를 RO(Read-Only)로 마운트하기 때문에, 나름 무결성을 유지시켜 줄 수 있다. 단 DD를 사용하려면, 외부 인터페이스를
    마운트하더라도 루트 디바이스가 RW(Read-Write) 속성으로 마운트 되어야 하기 때문에, 결국 쓰기 권한을 줄 수밖에 없는 문제가 있다.
    하지만, 첫 번째 방법에 비해, 별도의 장치 필요없이 외장하드만 있으면 처리된다는 점은 매력적으로 다가온다.<br /><br />세 번째 방법은
    아직 테스트해보진 않은 기능으로 싱글모드로 부팅해서 FW로 연결하여 대상시스템을 이미징하는 방법이다. 이 방법은 첫 번째와 동일하게 다른 시스템과
    FW케이블이 필요하며, 아직 별도 검증이 필요한 방법이다. 이 방법은 좀더 연구가 필요하다.<br /><br />이번 글에선 두 번째 방법을
    활용하여 싱글 모드에 진입하여 DD 이미징를 생성할 수 있는 환경을 구성하는 방법을 알아보겠다.<br /><br />Mac OS X는 유닉스
    기반(엄밀히 말하면 Mach+BSD) 구조를 갖추고 있기 때문에, 유닉스와 동일하게 싱글 사용자 모드 접근이 가능하다. 맥의 싱글모드 부팅은
    리눅스와 다르게 키 입력으로 진입 가능하다. 키는 회색 화면이 뜨는 시점에 Command + S 를 누른다.<br /><br /><img alt="IMAG0295"
    border="0" height="338" src="http://lh4.ggpht.com/-ZkgJFQxocXQ/T2qeR6buQ_I/AAAAAAAAAQc/p4LGsnzCYOQ/IMAG0295.jpg?imgmax=800"
    title="IMAG0295.jpg" width="600"><br /><br />맥은 루트 권한을 가진 쉘을 제공하지만, 재미있게도, read-only
    권한으로 파일시스템을 마운트시켜 사용자에게 제공한다. 이는 나름 일반 유저가 주요 파일을 임의로 수정하여, 시스템에 문제를 발생 시키는 것을
    방지하는 차원인 것 같다.<br /><br />문제는 읽기 전용으로 마운트 됨으로인해 USB 장치를 마운트할 영역을 만들 수 없다는 점이다.
    이 문제를 해결하기 위해선 루트 디바이스에 쓰기 권한을 추가해야 한다. 애플은 루트 디바이스에 쓰기 권한을 주는 방법을 위 그림과 같이 친절하게
    표시해두었다. 다음과 같이 콘솔에서 명령을 입력한다.<br /><br /><span>:/ root#/sbin/fsck -fy # 파일시스템
    일관성 체크</span><br /><span>:/ root#/sbin/mount -uw / # 부트 볼륨에 쓰기권한을 추가해서 재-마운트함.</span><br
    /><br />이 두 명령어를 사용하면, 루트 디바이스에 쓰기 속성이 추가되어, 파일을 쓸 수 있다. 이제 폴더를 만들고 USB를 연결하여
    마운트하면, 정상적으로 마운트되는 것을 볼 수 있다.<br /><br /><img alt="IMAG0296" border="0" height="338"
    src="http://lh3.ggpht.com/-D4FW-AWLki4/T2qeTvom9bI/AAAAAAAAAQk/779jJF97HpA/IMAG0296.jpg?imgmax=800"
    title="IMAG0296.jpg" width="600"><br />위 예제에선 FAT32로 포맷된 외장디스크를 연결하였기 때문에 마운트
    타입을 msdos로 설정하였다. 저 옵션은 각 타입에 맞춰 설정하면 된다.<br /><br />이제 맥의 dd명령을 이용하여 이미징을 수행하면,
    싱글모드를 통한 디스크 이미징이 완료된다. :-)<br /><br />이 글을 쓰면서 드는 생각은 Firewire를 이용하면, Read-Only
    filesystem 상태에서 이미징을 뜰 수 있지 않을까란 생각이 든다. 이 방법이 가능하다면, 포렌식의 중요 요소인 무결성 유지 부분에서
    위 방법보다 나은 점수를 받을 수 있을 것이다.<br /><br />다음엔 FW를 이용하여 이미징을 테스트해보고 그 결과를 올려보겠다 :-)<div>n0fate''s
    Forensic Space :)</div>'
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>오늘 오전에 <a href="http://baadc0de.blogspot.com/" target="_blank">baadc0de</a>한테 연락이 왔다. 내용인 즉슨, 맥 운영체제의 패스워드를 모르는 경우 디스크 이미징을 수행하는 방법을 묻는 내용이였다. 사실 디스크를 물리적으로 추출하여 이미징하는 방법이 있지만, 요청 사례의 경우엔 맥북에어였는데 이 장비에 있는 SSD는 일반적으로 사용되는 SATA 인터페이스와 다른 구조를 가지고 있다.<br /><img alt="Mba 2011 ssd ogrady2" border="0" height="137" src="{{ site.baseurl }}/assets/mba-2011-ssd-ogrady2.jpeg?imgmax=800" title="mba-2011-ssd-ogrady2.jpeg" width="500" /><br /><span>(reference: http://www.zdnet.com/blog/apple/owc-mercury-on-the-go-enclosure-for-macbook-air-ssd-review/11540)</span></p>
<p>이 인터페이스는 MINI PCI EXPRESS(ZIF-40핀 인터페이스)라고 불리며, OWC에선 이를 위한 외장 케이스도 있었다. :)</p>
<p><img alt="Owc mercury enclosure for macbook air ssd" border="0" height="334" src="{{ site.baseurl }}/assets/owc-mercury-enclosure-for-macbook-air-ssd.jpeg?imgmax=800" title="owc-mercury-enclosure-for-macbook-air-ssd.jpeg" width="500" /><br /><span>(reference: http://www.zdnet.com/blog/apple/owc-mercury-on-the-go-enclosure-for-macbook-air-ssd-review/11540)</span></p>
<p>여튼 이런 상황에 이미징 장치가 SATA, IDE 인터페이스만 가지고 있다면, 별도의 인터페이스를 구매하는 등 까다로운 상황이 발생할 수 있다.<br /><i><span>코멘트. 포렌식 수사관이신 soomie kim님의 말씀에 의하면, 넷북을 받았는데 MINI PCI EX 인터페이스를 사용하여서, 삼성 서비스센터를 통해 MINI PCI EX to SATA PCB를 대여해서 처리한적이 있다고 한다. 최근 울트라씬이다 뭐다해서 얇은 노트북이 많이 나오다보니, 점점 해당 인터페이스를 접하는 경우가 많을 것이라 생각한다. 만약 어댑터를 구매하고자한다면, Mini PCI-e SSD to 2.5" SATA Converter Adapter라는 제품을 찾아보기 바란다.</span></i></p>
<p>이 경우 소프트웨어적인 방법을 활용해야하는데, 맥의 사용자 패스워드를 모르는 상태에서 소프트웨어 기반 디스크 이미징을 수행하기 위해선 다음과 같은 방법을 생각해볼 수 있다.
<ul>
<li>타겟디스크모드(TDM)를 활용한 디스크 접근</li>
<li>싱글모드 부팅 후 DD를 이용한 디스크 이미징</li>
<li>싱글모드 부팅 후 FW(Firewire)를 이용한 이미지</li>
</ul>
<p>우선 첫 번째 방법은 맥에서만 지원하는 타겟 디스크 모드를 활용하여, 논리적 디스크에 접근하는 방법으로 FW단자를 활용하여 접근을 수행한다. 이 부분은 맥 포렌식에서 많이 논의된 부분이니 추가 설명은 생략하겠다. 이 방법은 맥에서 기본적으로 지원하는 기능인만큼 사용하긴 편하지만, 디스크가 RW 속성을 모두 가진 상태에서 마운트하는 문제가 있다.</p>
<p>두 번째 방법은 전통 유닉스 시스템과 같이 싱글모드로 부팅해서 DD로 이미징하는 방법이다. Mac OS X는 디스크를 RO(Read-Only)로 마운트하기 때문에, 나름 무결성을 유지시켜 줄 수 있다. 단 DD를 사용하려면, 외부 인터페이스를 마운트하더라도 루트 디바이스가 RW(Read-Write) 속성으로 마운트 되어야 하기 때문에, 결국 쓰기 권한을 줄 수밖에 없는 문제가 있다. 하지만, 첫 번째 방법에 비해, 별도의 장치 필요없이 외장하드만 있으면 처리된다는 점은 매력적으로 다가온다.</p>
<p>세 번째 방법은 아직 테스트해보진 않은 기능으로 싱글모드로 부팅해서 FW로 연결하여 대상시스템을 이미징하는 방법이다. 이 방법은 첫 번째와 동일하게 다른 시스템과 FW케이블이 필요하며, 아직 별도 검증이 필요한 방법이다. 이 방법은 좀더 연구가 필요하다.</p>
<p>이번 글에선 두 번째 방법을 활용하여 싱글 모드에 진입하여 DD 이미징를 생성할 수 있는 환경을 구성하는 방법을 알아보겠다.</p>
<p>Mac OS X는 유닉스 기반(엄밀히 말하면 Mach+BSD) 구조를 갖추고 있기 때문에, 유닉스와 동일하게 싱글 사용자 모드 접근이 가능하다. 맥의 싱글모드 부팅은 리눅스와 다르게 키 입력으로 진입 가능하다. 키는 회색 화면이 뜨는 시점에 Command + S 를 누른다.</p>
<p><img alt="IMAG0295" border="0" height="338" src="{{ site.baseurl }}/assets/IMAG0295.jpg?imgmax=800" title="IMAG0295.jpg" width="600" /></p>
<p>맥은 루트 권한을 가진 쉘을 제공하지만, 재미있게도, read-only 권한으로 파일시스템을 마운트시켜 사용자에게 제공한다. 이는 나름 일반 유저가 주요 파일을 임의로 수정하여, 시스템에 문제를 발생 시키는 것을 방지하는 차원인 것 같다.</p>
<p>문제는 읽기 전용으로 마운트 됨으로인해 USB 장치를 마운트할 영역을 만들 수 없다는 점이다. 이 문제를 해결하기 위해선 루트 디바이스에 쓰기 권한을 추가해야 한다. 애플은 루트 디바이스에 쓰기 권한을 주는 방법을 위 그림과 같이 친절하게 표시해두었다. 다음과 같이 콘솔에서 명령을 입력한다.</p>
<p><span>:/ root#/sbin/fsck -fy # 파일시스템 일관성 체크</span><br /><span>:/ root#/sbin/mount -uw / # 부트 볼륨에 쓰기권한을 추가해서 재-마운트함.</span></p>
<p>이 두 명령어를 사용하면, 루트 디바이스에 쓰기 속성이 추가되어, 파일을 쓸 수 있다. 이제 폴더를 만들고 USB를 연결하여 마운트하면, 정상적으로 마운트되는 것을 볼 수 있다.</p>
<p><img alt="IMAG0296" border="0" height="338" src="{{ site.baseurl }}/assets/IMAG0296.jpg?imgmax=800" title="IMAG0296.jpg" width="600" /><br />위 예제에선 FAT32로 포맷된 외장디스크를 연결하였기 때문에 마운트 타입을 msdos로 설정하였다. 저 옵션은 각 타입에 맞춰 설정하면 된다.</p>
<p>이제 맥의 dd명령을 이용하여 이미징을 수행하면, 싱글모드를 통한 디스크 이미징이 완료된다. :-)</p>
<p>이 글을 쓰면서 드는 생각은 Firewire를 이용하면, Read-Only filesystem 상태에서 이미징을 뜰 수 있지 않을까란 생각이 든다. 이 방법이 가능하다면, 포렌식의 중요 요소인 무결성 유지 부분에서 위 방법보다 나은 점수를 받을 수 있을 것이다.</p>
<p>다음엔 FW를 이용하여 이미징을 테스트해보고 그 결과를 올려보겠다 :-)
<div>n0fate's Forensic Space :)</div>
