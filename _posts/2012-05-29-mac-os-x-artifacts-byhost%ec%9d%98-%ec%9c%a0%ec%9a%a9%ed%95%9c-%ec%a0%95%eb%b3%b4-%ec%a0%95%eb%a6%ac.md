---
layout: post
title: 'Mac OS X Artifacts : ByHost의 유용한 정보 정리'
date: 2012-05-29 12:51:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags: []
meta:
  blogger_blog: forensic.n0fate.com
  blogger_author: Kyeongsik Lee
  blogger_permalink: /2012/05/mac-os-x-artifacts-byhost.html
  _blogger_self: https://www.blogger.com/feeds/7620918615785302711/posts/default/3051355343857373014
  _edit_last: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  avada_post_views_count: '1126'
  fusion_builder_content_backup: '<div></div>앞선 포스팅에 설명한 것처럼 ByHost 디렉터리엔 여러가지 유용한
    정보를 가진 plist파일이 존재한다. 디렉터리에 존재하는 plist 파일은 보통 동일 파일명의 lockfile 확장자를 가진 0바이트 파일을
    가지고 있다. lockfile은 Mac OS X 라이언에서 추가된 개념으로 이 확장자를 가지고 있는 경우엔, 하나의 서비스가 해당 plist를
    오픈하여 작업하는 동안에 다른 서비스는 그 plist를 읽을 수만 있게 된다. 이는 여러 서비스가 동시에 파일을 수정하여 생기는 문제를 해결하기
    위함으로 스노우 레오파드 이전엔 각 서비스마다 수정할 plist를 하나 복제해서 고유의 식별자를 가진 확장자를 추가하고, 해당 파일을 수정한
    후, 이를 원본 파일에 통합(merge)하는 방법을 사용했었다.<br /><br />이번 포스팅에선 이 ByHost 디렉터리에 존재하는 여러가지
    정보 중 포렌식적으로 유용한 정보만을 선정하여 간략하게 정리하였다. TimeMachine과 같은 경우처럼 내용이 제대로 식별되지 않은 부분도
    존재하며, 식별 후에 해당 내용을 수정할 예정이다.<br /><br />여러 포렌식 분석관에게 도움이 되길 바란다. :)<br /><br /><b>디렉터리
    : ~/Library/Preferences/ByHost/</b><br /><br /><br /><br /><table cellpadding="0"
    cellspacing="0"><tbody><tr><td valign="top"><div><b>파일 명</b></div></td><td valign="top"><div><b>내용</b></div></td></tr><tr><td
    valign="top"><div>com.apple.AddressBook.sync.<hardware UUID>.plist</div></td><td
    valign="top"><div>맥의 주소록과 iCloud간의 최종 동기화 날짜를 저장함.</div></td></tr><tr><td valign="top"><div>com.apple.airport.agent.<hardware
    UUID>.plist</div></td><td valign="top"><div>에어포트 프로그램의 noWideAreaBrowsing</div></td></tr><tr><td
    valign="top"><div>com.apple.Bluetooth.<hardware UUID>.plist</div></td><td valign="top"><div>블루투스
    디바이스 설정 정보. 블루투스 버전과 페어링 종료 시간(DeviceExpiration), OBEX를 통한 파일 전송 시 저장되는 디렉터리 정보,
    최근에 페어링한 5개의 블루투스 장치의 주소정보와 시간을 가짐</div></td></tr><tr><td valign="top"><div>com.apple.coreservices.appleidauthenticationinfo.<hardware
    UUID>.plist</div></td><td valign="top"><div>애플 아이디 인증관련된 정보. 로그인한 아이디와 최초/최종 로그인
    날짜, 해시된패스워드참조정보(확인되지 않음)(HashedPasswordRef), 인증서 정보를 가짐.</div></td></tr><tr><td
    valign="top"><div>com.apple.finder.<hardware UUID>.plist</div></td><td valign="top"><div>같은
    네트워크에 연결된 시스템간에 무선으로 파일 전송을 지원하는 AirDrop 기술 지원을 위해 디바이스 식별 ID를 저장함</div></td></tr><tr><td
    valign="top"><div>com.apple.HIToolbox.<hardware UUID>.plist</div></td><td valign="top"><div>키보드
    언어 설정 정보를 가짐</div></td></tr><tr><td valign="top"><div>com.apple.iCal.helper.<hardware
    UUID>.plist</div></td><td valign="top"><div>iCal과 iCloud간의 최종 동기화 시간을 가짐</div></td></tr><tr><td
    valign="top"><div>com.apple.iChat.Jabber.<hardware UUID>.plist</div></td><td valign="top"><div>iChat으로
    연결한 애플ID 정보, 사용자 이름, 등록한 사진 등</div></td></tr><tr><td valign="top"><div>com.apple.iChat.Subnet.<hardware
    UUID>.plist</div></td><td valign="top"><div>iChat으로 연결한 애플ID 정보, 사용자 이름, 등록한 사진,
    자동 로그인 여부, 활성화한 계정 등</div></td></tr><tr><td valign="top"><div>com.apple.iChat.Yahoo.<hardware
    UUID>.plist</div></td><td valign="top"><div>iChat에 등록한 야후 계정에 대한 정보</div></td></tr><tr><td
    valign="top"><div>com.apple.imservice.iMessage.<hardware UUID>.plist</div></td><td
    valign="top"><div>iMessage 프로그램에 등록한 사용자 계정 정보</div></td></tr><tr><td valign="top"><div>com.apple.loginwindow.<hardware
    UUID>.plist</div></td><td valign="top"><div>시스템 재부팅 시 현재 작업 상태를 유지하기 위해 현재 구동
    중인 프로그램 정보를 저장</div></td></tr><tr><td valign="top"><div>com.apple.preference.display2.<hardware
    UUID>.plist</div></td><td valign="top"><div>연결한 각 외부모니터에 대한 정보를 따로 저장함. 애플은 연결한
    외부모니터마다 별도의 설정 정보를 관리함.</div></td></tr><tr><td valign="top"><div>com.apple.scheduler.<hardware
    UUID>.plist</div></td><td valign="top"><div>소프트웨어 업데이트 날짜와 체크 주기 정보</div></td></tr><tr><td
    valign="top"><div>com.apple.screensaver.<hardware UUID>.plist</div></td><td valign="top"><div>스크린세이버
    설정 정보를 가짐</div></td></tr><tr><td valign="top"><div>com.apple.SubmitDIagInfo.<hardware
    UUID>.plist</div></td><td valign="top"><div>맥 운영체제 설치 시점에 나오는 License Agreement를
    승인한 시점을 의미하며, 최초 설치 시점을 유추할 수 있음</div></td></tr><tr><td valign="top"><div>com.apple.systemuiserver.<hardware
    UUID>.plist</div></td><td valign="top"><div>맥에서 기본으로 제공되는 서비스 중 사용자가 임의로 메뉴바에서
    삭제한 서비스.(예. 타임머신 아이콘 삭제)</div></td></tr><tr><td valign="top"><div>com.apple.TimeMachine.<hardware
    UUID></div></td><td valign="top"><div>타임머신 설정과 관련된 정보(현재 디바이스의 UUID나 TimeMachine으로
    설정한 디바이스의 UUID로 유추됨)</div></td></tr><tr><td valign="top"><div>com.google.GoogleContactSync.<hardware
    UUID>.plist</div></td><td valign="top"><div>애플에서 제공하는 AddressBook에 구글 계정을 등록할
    경우, 동기화 정보를 저장한다. 최종 동기화 시간과 구글에서 가져온 모든 연락처 정보를 가짐</div></td></tr><tr><td valign="top"><div>MicrosoftRegistrationDB.<hardware
    UUID>.plist</div></td><td valign="top"><div>마이크로소프트 오피스 설치 시 생성되며, 윈도의 레지스트리 구조를
    plist에 표현하여 관리함. 최근 시작 문서와 같은 정보는 존재하지 않음.</div></td></tr></tbody></table><br
    /><div><br /></div><div></div><br /><br /><br /><div>n0fate''s Forensic Space
    :)</div>'
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<div></div>
<p>앞선 포스팅에 설명한 것처럼 ByHost 디렉터리엔 여러가지 유용한 정보를 가진 plist파일이 존재한다. 디렉터리에 존재하는 plist 파일은 보통 동일 파일명의 lockfile 확장자를 가진 0바이트 파일을 가지고 있다. lockfile은 Mac OS X 라이언에서 추가된 개념으로 이 확장자를 가지고 있는 경우엔, 하나의 서비스가 해당 plist를 오픈하여 작업하는 동안에 다른 서비스는 그 plist를 읽을 수만 있게 된다. 이는 여러 서비스가 동시에 파일을 수정하여 생기는 문제를 해결하기 위함으로 스노우 레오파드 이전엔 각 서비스마다 수정할 plist를 하나 복제해서 고유의 식별자를 가진 확장자를 추가하고, 해당 파일을 수정한 후, 이를 원본 파일에 통합(merge)하는 방법을 사용했었다.</p>
<p>이번 포스팅에선 이 ByHost 디렉터리에 존재하는 여러가지 정보 중 포렌식적으로 유용한 정보만을 선정하여 간략하게 정리하였다. TimeMachine과 같은 경우처럼 내용이 제대로 식별되지 않은 부분도 존재하며, 식별 후에 해당 내용을 수정할 예정이다.</p>
<p>여러 포렌식 분석관에게 도움이 되길 바란다. :)</p>
<p><b>디렉터리 : ~/Library/Preferences/ByHost/</b></p>
<table cellpadding="0" cellspacing="0">
<tbody>
<tr>
<td valign="top">
<div><b>파일 명</b></div>
</td>
<td valign="top">
<div><b>내용</b></div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.AddressBook.sync.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>맥의 주소록과 iCloud간의 최종 동기화 날짜를 저장함.</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.airport.agent.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>에어포트 프로그램의 noWideAreaBrowsing</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.Bluetooth.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>블루투스 디바이스 설정 정보. 블루투스 버전과 페어링 종료 시간(DeviceExpiration), OBEX를 통한 파일 전송 시 저장되는 디렉터리 정보, 최근에 페어링한 5개의 블루투스 장치의 주소정보와 시간을 가짐</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.coreservices.appleidauthenticationinfo.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>애플 아이디 인증관련된 정보. 로그인한 아이디와 최초/최종 로그인 날짜, 해시된패스워드참조정보(확인되지 않음)(HashedPasswordRef), 인증서 정보를 가짐.</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.finder.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>같은 네트워크에 연결된 시스템간에 무선으로 파일 전송을 지원하는 AirDrop 기술 지원을 위해 디바이스 식별 ID를 저장함</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.HIToolbox.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>키보드 언어 설정 정보를 가짐</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.iCal.helper.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>iCal과 iCloud간의 최종 동기화 시간을 가짐</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.iChat.Jabber.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>iChat으로 연결한 애플ID 정보, 사용자 이름, 등록한 사진 등</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.iChat.Subnet.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>iChat으로 연결한 애플ID 정보, 사용자 이름, 등록한 사진, 자동 로그인 여부, 활성화한 계정 등</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.iChat.Yahoo.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>iChat에 등록한 야후 계정에 대한 정보</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.imservice.iMessage.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>iMessage 프로그램에 등록한 사용자 계정 정보</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.loginwindow.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>시스템 재부팅 시 현재 작업 상태를 유지하기 위해 현재 구동 중인 프로그램 정보를 저장</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.preference.display2.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>연결한 각 외부모니터에 대한 정보를 따로 저장함. 애플은 연결한 외부모니터마다 별도의 설정 정보를 관리함.</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.scheduler.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>소프트웨어 업데이트 날짜와 체크 주기 정보</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.screensaver.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>스크린세이버 설정 정보를 가짐</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.SubmitDIagInfo.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>맥 운영체제 설치 시점에 나오는 License Agreement를 승인한 시점을 의미하며, 최초 설치 시점을 유추할 수 있음</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.systemuiserver.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>맥에서 기본으로 제공되는 서비스 중 사용자가 임의로 메뉴바에서 삭제한 서비스.(예. 타임머신 아이콘 삭제)</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.apple.TimeMachine.<hardware uuid /></div>
</td>
<td valign="top">
<div>타임머신 설정과 관련된 정보(현재 디바이스의 UUID나 TimeMachine으로 설정한 디바이스의 UUID로 유추됨)</div>
</td>
</tr>
<tr>
<td valign="top">
<div>com.google.GoogleContactSync.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>애플에서 제공하는 AddressBook에 구글 계정을 등록할 경우, 동기화 정보를 저장한다. 최종 동기화 시간과 구글에서 가져온 모든 연락처 정보를 가짐</div>
</td>
</tr>
<tr>
<td valign="top">
<div>MicrosoftRegistrationDB.<hardware uuid>.plist</hardware></div>
</td>
<td valign="top">
<div>마이크로소프트 오피스 설치 시 생성되며, 윈도의 레지스트리 구조를 plist에 표현하여 관리함. 최근 시작 문서와 같은 정보는 존재하지 않음.</div>
</td>
</tr>
</tbody>
</table>
<p>
<div></div>
<div></div>
<p>
<div>n0fate's Forensic Space :)</div>
