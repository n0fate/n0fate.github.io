---
layout: post
title: OS X Yosemite(10.10) Forensic Artifacts
date: 2014-09-11 15:41:42.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags:
- Call History
- SMS
- Yosemite
- 문자
- 연락처
- 요세미티
author: "n0fate"
---

# English Version

<h2 class="">Introduction</h2>
<p>The new version of OS X, Yosemite (version 10), focus on refreshed design with deeper iOS8 integration. In this post, we are watching new features on Yosemite and learn more about where can we find these artifacts on it. All tests are based on Yosemite Developer Preview 7. First of all, features of Yosemite as following:</p>
<ul>
<li>iCloud Drive : You can read/write various file on iCloud as Dropbox.</li>
<li>Hand-off : Due to the integration between iOS and OS X, you can be continuously working iOS to OS X or otherwise.</li>
<li>Phone Call and SMS/MMS message : If you have iPhone, you can send and receive SMS message and phone call right on your Mac. Now you can make and receive iPhone call on your Mac</li>
</ul>
<p><img class="full aligncenter" src="{{ site.baseurl }}/assets/1410416600_thumb.png" alt="" width="332" height="103" align="middle" /></p>
<p>So, from now on we find new artifacts in OS X Yosemite.</p>
<p>&nbsp;</p>
<h2 class="">Analysis</h2>
<h3 class="">Call History</h3>
<p>Because OS X is integration with iOS, related information with phone call is stored on OS X if call history and so on. The call history of OS X is stored as below:</p>
<p>"~/Library/Application Support/CallHistoryDB/CallHistory.storedata"</p>
<p>But, FaceTime on OS X doesn't used it at analysis point. It seems to software bug or develop a related code yet I guess. The important artifacts on it as following:</p>
<ul>
<li>ZCALLRECORD table : It has several information of relevant call history. For example, phone reception(ZANSEWRED), contacts Type(FaceTime, voice or video call, etc), timestamps of Incoming/Outgoing call(ZDATE), call duration(ZDURATION), device identifier(ZDEVICE_ID), contacts name(ZNAME), contacts number(ZADDRESS) and so on</li>
<li>Z_METADATA : management of metadata such as registered device information.</li>
</ul>
<h3 class="">SMS messages</h3>
<p>SMS message, feature of Yosemite, is used on Messages Application(Messages.app)<br />
<img class="full aligncenter" src="{{ site.baseurl }}/assets/1410416660_thumb.png" alt="" width="416" height="207" align="middle" /><br />
SMS messages is stored on artifacts of 'Messages' application on Mavericks. The database file path of 'Messages' is below:</p>
<p>"~/Library/Messages/chat.db"</p>
<p>We can show records of 'SMS' type on database. The next figure show 'SMS' record on it using <a href="https://github.com/n0fate/walitean" target="_blank">walitean</a>.<br />
<img class="full aligncenter" src="{{ site.baseurl }}/assets/1555f05bdb8ea4e60e267d04917b555f.png" alt="" width="603" height="369" align="middle" data-en-media-hash="1555f05bdb8ea4e60e267d04917b555f" /></p>
<p>If element in the 'service' column of 'message' table is equal to 'SMS', records are incoming or outgoing SMS.</p>
<p>&nbsp;</p>
<h2 class="">Notes</h2>
<p>New artifacts described above are used to SQLite of WAL(Write-Ahead Logging) enabled. General SQLite database browser doesn't show journaling data such as deleted record since working time. So you need to analyze the time for WAL. I recommend the following posts.</p>
<ul>
<li>SQLite WAL (Write-Ahead Logging) - http://forensic.n0fate.com/?p=1077</li>
<li>THE FORENSIC IMPLICATIONS OF SQLITE’S WRITE AHEAD LOG - http://www.cclgroupltd.com/the-forensic-implications-of-sqlites-write-ahead-log/</li>
</ul>

# 한국어 버전

<h2 class="">소개</h2>
<p>OS X의 새로운 버전인 요세미티(10.10)은 iOS와 OS X간의 통합과 전체적인 디자인 변화에 초점을 두었다. 이번 운영체제는 더 많은 iOS 사용자가 애플 제품을 사용하도록 유도하기 위한 애플의 포석으로 보인다. 이번 포스팅에서는 요세미티 중 포렌식적으로 유용한 아티팩트를 살펴보고, 이 중 몇가지 아티팩트를 어디에서 찾을 수 있는지 정리하였다. 본 내용은 요세미티 최신 버전인 DP7(Developer Preview 7)을 기준으로 작성되었다. 요세미티에는 여러가지 기능이 추가되었지만 이 중 포렌식적으로 유용한 기능은 다음과 같다.</p>
<ul>
<li>아이클라우드 드라이브 : 아이클라우드를 드롭박스처럼 사용할 수 있다.</li>
<li>핸드오프(Hand-off) : 아이폰에서 하는 작업을 맥에서 이어하거나 맥에서하던 작업을 아이폰에서 연속적으로할 수 있다.</li>
<li>iOS 연동성 : SMS와 전화기능을 맥에서도 사용할 수 있다. iMessage를 이용하여 문자 메시지를 발송할 수 있으며, 아이폰으로 전화가 오는 경우 맥에서도 함께 표시되어 맥에서 전화를 받을 수 있다. 또한 OS X에서 Facetime 애플리케이션을 이용하여 아이폰을 통한 전화걸기도 가능하다.</li>
</ul>
<p><img class="full aligncenter" src="{{ site.baseurl }}/assets/1410416600_thumb.png" alt="" width="332" height="103" align="middle" /></p>
<p>이번 포스팅에서는 이 중 iOS 연동 아티팩트가 어디에 저장되는지 확인하였다.</p>
<p>&nbsp;</p>
<h2 class="">분석</h2>
<h3 class="">통화기록</h3>
<p>iOS와 맥의 통화 기능이 연동된다면, 이와 관련된 정보가 시스템에 보존될 수 있다. 전화 기록 중 가장 유용한 기능은 최근 통화 목록이다. OS X는 이러한 통화 기록을 위해 "~/Library/Application Support/CallHistoryDB/CallHistory.storedata” 파일을 관리한다. 하지만 이 데이터베이스를 실제로 사용하지 않으며, FaceTime 애플리케이션도 최근 연락 기록을 유지하지 않는다. 현재 버전(DP7)에서 아직 해당 기능을 구현하지 못한 것으로 생각된다. 데이터베이스 테이블 정보는 다음과 같이 유추된다.</p>
<ul>
<li>ZCALLDBPROPERTIES : 데이터베이스에 저장할 속성정보를 보관한다.</li>
<li>ZCALLRECORD : 실제 연락처 기록이 저장되는 공간으로 판단된다. 가장 중요한 정보로 전화 수신 여부(ZANSWERED), 연락처 종류(FactTime 음성/영상, 3g 전화), 날짜(ZDATE), 통화 시간(ZDURATION), 장치 식별자(ZDEVICE_ID), 연락한 사람 이름(ZNAME), 연락처(ZADDRESS) 정보가 기록될 것으로 보인다.</li>
<li>Z_METADATA : 등록한 디바이스 정보 등 메타데이터 정보 유지에 사용된다.</li>
<li>Z_PRIMARYKEY : 데이터베이스의 키를 관리하기 위한 테이블이다.</li>
</ul>
<p>이 파일을 분석하면, 아이폰의 통화 내역을 추출하지 못하는 상황에서도 정보를 수집할 수 있다.</p>
<p>&nbsp;</p>
<h3 class="">문자 메시지</h3>
<p>요세미티의 문자 메시지 기능은 아이메시지에서 사용할 수 있다.</p>
<p><img class="full aligncenter" src="{{ site.baseurl }}/assets/1410416660_thumb.png" alt="" width="416" height="207" align="middle" /></p>
<p class="">SMS 정보는 아이메시지 데이터베이스에 새로운 타입을 추가하는 형태로 저장된다. 즉, 기존 아이메시지 분석 방법을 그대로 적용하면 된다. 기존에 아이메시지 데이터베이스는 "~/Library/Messages/chat.db" 에 저장되어 있다. 이 데이터베이스에는 아이메시지 정보 뿐만 아니라, SMS 메시지 내역도 포함되어 있다. 다음은 <a href="https://github.com/n0fate/walitean" target="_blank">walitean</a>을 사용하여 채팅 데이터베이스의 테이블 정보를 확인한 것이다.</p>
<p><img class="full aligncenter" src="{{ site.baseurl }}/assets/1555f05bdb8ea4e60e267d04917b555f.png" alt="" width="603" height="369" align="middle" data-en-media-hash="1555f05bdb8ea4e60e267d04917b555f" /></p>
<p>위 그림과 같이 테이블에 정보를 표시하고 기존 아이메시지와 데이터를 똑같이 처리한다. 기존 아이메시지와 동일한 방법으로 분석하면 되므로 크게 정리할 내용은 없다.</p>
<h2 class="">유의사항</h2>
<p>위 두 아티팩트는 SQLite 데이터베이스를 사용하며, 두 SQLite가 저널링 기능인 WAL(Write-Ahead Logging)을 사용한다. 이에 WAL에 대한 별도의 분석이 필요하다. 이에 대한 포스팅은 <a title="undefined" href="http://forensic.n0fate.com/?p=1077" target="_blank">SQLite WAL (Write-Ahead Logging)</a>을 참고하면, 구조와 분석 방법을 확인할 수 있을 것이다.<br />
