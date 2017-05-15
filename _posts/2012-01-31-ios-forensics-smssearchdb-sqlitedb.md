---
layout: post
title: 'iOS Forensics : SMSSearchdb.sqlitedb'
date: 2012-01-31 21:17:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- iOS Forensics
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/01/ipad-forensics-smssearchdbsqlitedb.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/6205449831824045619
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1465'
  fusion_builder_content_backup: <b><i>* 이 글은 iPad2 IOS5.0.1 버전을 기준으로 작성되었기 때문에 다른
    버전에서는 그 내용이 다를 수 있습니다. 혹시 다른 버전에서 본 파일이 존재하지 않는다면 댓글로 알려주시면 감사하겠습니다 :-)</i></b><br
    /><br />기존에 가지고 있던 아이패드2를 이번에 나온 ios5용 탈옥으로 핵(hack)하면서 포렌식 적으로 재미난 것이 무엇이 있을까
    확인해보던 중 SMS와 관련된 재미있는 내용을 확인하였다. 아이패드의 기본 사용자 계정인 mobile의 홈 디렉터리를 기준으로 '~/Library/Spotlight/'
    로 이동해보면 다음과 같이 두 디렉터리를 볼 수 있다.<br /><br /><br /><span>n0fates-iPad:~/Library/Spotlight
    mobile$ ls</span><br /><span>com.apple.MobileSMS/  com.apple.SpotlightTopHits/</span><br
    /><br />Spotlight는 보통 아이폰의 검색 기능을 위해 존재하는 데몬으로 연락처나, 문자메시지, 지원하는 몇몇 문서 포맷에 대해,
    다양한 정보를 인덱싱하여 저장하고 있는 데이터베이스이다. 고유의 포맷을 가지고 있으며, 그 포맷이 알려져 있지 않은 상태이다. 보통은 하나의
    디렉터리에 Index0, Index1 이런식으로 데몬의 고유의 관리 방식에 맞춰서 다 수의 파일을 만들어두는게 일반적으로 맥에서 보았던 모습인데,
    IOS의 경우엔 MobileSMS라는 디렉터리가 별도로 유지되고 있었다. 디렉터의 내용은 다음과 같다.<br /><br /><br /><span>n0fates-iPad:~/Library/Spotlight/com.apple.MobileSMS
    mobile$ ls -al</span><br /><span>total 96</span><br /><span>drwxr-xr-x 2 mobile
    mobile   170 Jan 31 18:28 ./</span><br /><span>drwxr-xr-x 4 mobile mobile   136
    Oct 30 15:33 ../</span><br /><span>-rw-r--r-- 1 mobile mobile 69632 Jan 31 18:28
    SMSSearchdb.sqlitedb</span><br /><span>-rw-r--r-- 1 mobile mobile 24576 Jan 31
    18:28 SMSSearchidx.spotlight</span><br /><span>-rw-r--r-- 1 mobile mobile    
    0 Jan 31 18:28 updates.SMSSearch.spotlight</span><br /><br />디렉터리 내부에는 sqlite
    포맷의 파일 하나와 스팟라이트 포맷의 파일이 두개 존재하였다. 실제로도 각 포맷은 확장자 명에 맞는 포맷 구조를 가지고 있었다.<br /><br
    />SMSSearchdb.sqlitedb 파일을 뷰어로 확인하면 다음과 같이 나타난다.<br /><br /><div><a href="http://3.bp.blogspot.com/-i7mSzAJqiM4/Tyf34-NkAhI/AAAAAAAAAPE/5DGFpnlyIug/s1600/Capture.PNG"
    imageanchor="1"><img border="0" height="229" src="http://3.bp.blogspot.com/-i7mSzAJqiM4/Tyf34-NkAhI/AAAAAAAAAPE/5DGFpnlyIug/s640/Capture.PNG"
    width="640"></a></div><div><br /></div><div><br /></div><div>Contents 테이블에는 고유
    ID인 ROWID와 연락처 정보(iMessage의 경우엔 메일주소도 가능)와 각각의 메시지에 대한 고유값인 GUID를 가진 external
    id 가 존재하였다. 그리고 대화를 나눈 사용자와 대화를 나눈 내용이 무엇인지도 기록되어 있었다. 실제 데이터를 삭제 시엔 문자 메시지도 함께
    삭제되지만, sqlite의 카빙 기법은 이미 많이 알려져 있기 때문에 카빙 기술을 통해 복구할 수 있을 것이다.</div><div><br /></div><div>보통
    맥 환경의 spotlight는 이렇게 명시적인 sqlite포맷을 가지진 않는다. 보통 스마트폰에서 사람들이 많이 하는 문자열 검색 대상이 문자
    메시지이다보니 별도로 sqlite 형태의 관리를 하는 것이 아닌가하는 생각이 든다.</div><div><br /></div><div>현재 아이폰에서는
    이 파일이 존재하는지 확인하진 않았지만, 같은 IOS 계열이기 때문에 존재할 것이라 생각된다. IOS에서 수발신한 SMS의 경우엔 많은 논문에서
    보통 sms.db 파일로 Library/SMS/ 에 위치해 있는 것으로 많이 알려져 있었기 때문에, 안티-포렌식을 위해 해당 파일 자체를 제거해버리는
    상황이 발생할 수 있는데, 이런 경우에 파일 복구 단계를 진행할 필요 없이 해당 파일을 추출하여 SMS를 확인할 수 있기 때문에, 분석 속도를
    높이는데 도움이 될 것이라 생각한다. :)</div><br /><br /><div>n0fate's Forensic Space :)</div>
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p><b><i>* 이 글은 iPad2 IOS5.0.1 버전을 기준으로 작성되었기 때문에 다른 버전에서는 그 내용이 다를 수 있습니다. 혹시 다른 버전에서 본 파일이 존재하지 않는다면 댓글로 알려주시면 감사하겠습니다 :-)</i></b></p>
<p>기존에 가지고 있던 아이패드2를 이번에 나온 ios5용 탈옥으로 핵(hack)하면서 포렌식 적으로 재미난 것이 무엇이 있을까 확인해보던 중 SMS와 관련된 재미있는 내용을 확인하였다. 아이패드의 기본 사용자 계정인 mobile의 홈 디렉터리를 기준으로 '~/Library/Spotlight/' 로 이동해보면 다음과 같이 두 디렉터리를 볼 수 있다.</p>
<p><span>n0fates-iPad:~/Library/Spotlight mobile$ ls</span><br /><span>com.apple.MobileSMS/  com.apple.SpotlightTopHits/</span></p>
<p>Spotlight는 보통 아이폰의 검색 기능을 위해 존재하는 데몬으로 연락처나, 문자메시지, 지원하는 몇몇 문서 포맷에 대해, 다양한 정보를 인덱싱하여 저장하고 있는 데이터베이스이다. 고유의 포맷을 가지고 있으며, 그 포맷이 알려져 있지 않은 상태이다. 보통은 하나의 디렉터리에 Index0, Index1 이런식으로 데몬의 고유의 관리 방식에 맞춰서 다 수의 파일을 만들어두는게 일반적으로 맥에서 보았던 모습인데, IOS의 경우엔 MobileSMS라는 디렉터리가 별도로 유지되고 있었다. 디렉터의 내용은 다음과 같다.</p>
<p><span>n0fates-iPad:~/Library/Spotlight/com.apple.MobileSMS mobile$ ls -al</span><br /><span>total 96</span><br /><span>drwxr-xr-x 2 mobile mobile   170 Jan 31 18:28 ./</span><br /><span>drwxr-xr-x 4 mobile mobile   136 Oct 30 15:33 ../</span><br /><span>-rw-r--r-- 1 mobile mobile 69632 Jan 31 18:28 SMSSearchdb.sqlitedb</span><br /><span>-rw-r--r-- 1 mobile mobile 24576 Jan 31 18:28 SMSSearchidx.spotlight</span><br /><span>-rw-r--r-- 1 mobile mobile     0 Jan 31 18:28 updates.SMSSearch.spotlight</span></p>
<p>디렉터리 내부에는 sqlite 포맷의 파일 하나와 스팟라이트 포맷의 파일이 두개 존재하였다. 실제로도 각 포맷은 확장자 명에 맞는 포맷 구조를 가지고 있었다.</p>
<p>SMSSearchdb.sqlitedb 파일을 뷰어로 확인하면 다음과 같이 나타난다.</p>
<div><a href="http://3.bp.blogspot.com/-i7mSzAJqiM4/Tyf34-NkAhI/AAAAAAAAAPE/5DGFpnlyIug/s1600/Capture.PNG" imageanchor="1"><img border="0" height="229" src="{{ site.baseurl }}/assets/Capture.PNG" width="640" /></a></div>
<div></div>
<div></div>
<div>Contents 테이블에는 고유 ID인 ROWID와 연락처 정보(iMessage의 경우엔 메일주소도 가능)와 각각의 메시지에 대한 고유값인 GUID를 가진 external id 가 존재하였다. 그리고 대화를 나눈 사용자와 대화를 나눈 내용이 무엇인지도 기록되어 있었다. 실제 데이터를 삭제 시엔 문자 메시지도 함께 삭제되지만, sqlite의 카빙 기법은 이미 많이 알려져 있기 때문에 카빙 기술을 통해 복구할 수 있을 것이다.</div>
<div></div>
<div>보통 맥 환경의 spotlight는 이렇게 명시적인 sqlite포맷을 가지진 않는다. 보통 스마트폰에서 사람들이 많이 하는 문자열 검색 대상이 문자 메시지이다보니 별도로 sqlite 형태의 관리를 하는 것이 아닌가하는 생각이 든다.</div>
<div></div>
<div>현재 아이폰에서는 이 파일이 존재하는지 확인하진 않았지만, 같은 IOS 계열이기 때문에 존재할 것이라 생각된다. IOS에서 수발신한 SMS의 경우엔 많은 논문에서 보통 sms.db 파일로 Library/SMS/ 에 위치해 있는 것으로 많이 알려져 있었기 때문에, 안티-포렌식을 위해 해당 파일 자체를 제거해버리는 상황이 발생할 수 있는데, 이런 경우에 파일 복구 단계를 진행할 필요 없이 해당 파일을 추출하여 SMS를 확인할 수 있기 때문에, 분석 속도를 높이는데 도움이 될 것이라 생각한다. :)</div>
<div>n0fate's Forensic Space :)</div>
