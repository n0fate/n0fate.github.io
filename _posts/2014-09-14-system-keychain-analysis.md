---
layout: post
title: System Keychain Analysis (Extracting confidential information of Wi-Fi and
  SMB on OS X)
date: 2014-09-14 14:40:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags:
- Chainbreaker
- Keychain
- Mac OS X
author: "n0fate"
---

# English Version

<h2 class="">Introduction</h2>
<p class="">Keychain system on OS X manage confidential information. OS X Keychain system manage default three keychain files. System keychain(/Library/Keychain/System.keychain) has stored confidential information for OS Security(e.g. certificates signed Apple root certificate, Wi-Fi SSID and key). Each user has user keychain(login.keychain) that stores application authentication(ID and password), En/Decryption Key(e.g. iMessage, iCloud, Filevault2), Session Key(Facebook, Twitter) and so on. After OS X Mavericks, iCloud Keychain is added on OS X for synchronization with iOS and OS X. It holds several information on iOS(e.g. Wi-Fi SSID and key, Accounts on contacts, calendars, apple mail, safari).<br />
<img class="aligncenter full" title="" src="{{ site.baseurl }}/assets/1410664668_thumb.png" alt="" align="middle" /></p>
<p>Prior mentioned, OS X memory forensic tool, volafox, can dump master key on user keychain. So in this post, I explain decryption process of System keychain.<br />
Aforementioned system keychain store confidential information for OS management<br />
OS requires certificates of Kerberos, apple system, software signature for Security, Wi-Fi SSID/password for Airport(apple network device) and ID/password on file server for continuously services. Especially, Wi-Fi confidential information is very nice artifacts because investigator helpfully determines first connection time and lastly key modification time on wireless AP. These will be connected with user activity.</p>
<h2 class="">Extracting the master key of system keychain</h2>
<p class="">System keychain is stored on “/Library/Keychain/System.Keychain”. It has same structure with user keychain. So, you can dump confidential information with Chainbreaker. Decryption process of Chainbreaker explains this paper : <a title="Keychain-Analysis-with-Mac-OS-X-Memory-Forensics" href="http://forensic.n0fate.com/wp-content/uploads/2012/12/Keychain-Analysis-with-Mac-OS-X-Memory-Forensics.pdf" target="_blank">Keychain Analysis with Mac OS X Memory Forensics</a><br />
The master key of system keychain is stored on "SystemKey" file in the “/private/var/db” (Apple says, SystemKey file is kSystemUnlockFile). The "SystemKey" is UnlockBlob file format.</p>
<h3 class="">Common Blob</h3>
<p><img class="aligncenter full" title="" src="{{ site.baseurl }}/assets/1410672682_thumb.png" alt="" align="middle" /></p>
<p class="">magicNumber(4bytes) : Signature of common blob structure(0xFADE0711)<br />
Current Version (4bytes) : keychain version. default value is 0x00(version_MacOS_10_0)</p>
<h3 class="">UnlockBlob</h3>
<p><img class="full aligncenter" title="" src="{{ site.baseurl }}/assets/1410672633_thumb.png" alt="" align="middle" /><br />
Master Key(24bytes) : The master key is a 24byte DES key(192bits)<br />
Signature (16bytes) : Signature for checking correct DB key with the master key</p>
<p>The master key stored it is not encrypted. Now you can easily decrypt system keychain using this.</p>
<h2 class="">System keychain decryption using Chainbreaker</h2>
<p>GUI version of Chainbreaker can decrypt system keychain according to the below procedures.<br />
- Load a system keychain file<br />
- checking a checkbox said 'Is a Master Key'<br />
- Write hex-code on Editbox said 'Key' and click 'Analysis' button.</p>
<p>The result of analysis is following:<br />
<img class="aligncenter full" title="" src="{{ site.baseurl }}/assets/1410672965_thumb.png" alt="" align="middle" /></p>
<p class="">Table of generic password stores SSID, key, Create time, last modification time on wireless AP. Table of internet password stores server account, password, mounted volume path on Samba.</p>
<p>You can dump decrypted confidential information on system keychain using chainbreaker <a title="Chainbreaker console" href="https://github.com/n0fate/chainbreaker" target="_blank">console version</a> as following:</p>
<pre class="lang:sh decode:true " title="chainbreaker">$ python chainbreaker.py -i ~/System.keychain -k xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
 [-] DB Key
[..SNIP..]
[+] Symmetric Key Table: 0x000064f4
[+] Generic Password Record
 [-] RecordSize : 0x000000e4
 [-] Record Number : 0x00000000
 [-] SECURE_STORAGE_GROUP(SSGP) Area : 0x0000002c
 [-] Create DateTime: 20140816021533Z
 [-] Last Modified DateTime: 20140816021533Z
 [-] Description : AirPort network password
 [-] Creator :
 [-] Type :
 [-] PrintName : [BLANK]
 [-] Alias :
 [-] Account : [BLANK]
 [-] Service : AirPort
 [-] Password
00000000:  xx xx xx xx xx xx xx  xx                       xxxxxxxx</pre>
<p>&nbsp;</p>
<h2 class="">Conclusion</h2>
<p>The user keychain stores application confidential information, keys and certificates on each user. So, investigator can't acquire wifi SSID/key, mounted volume password and so on related OS management. If you are using this analysis method to extract it, you will get these on disk image only. You can analyze user activity or access other system with id/password on file sharing services.</p>
<p>&nbsp;</p>
<p>Reference : https://github.com/andrewdotn/chainbreaker/commit/b07b6840ed4013264199df7ef5a35282b591f5a1<br />

# 한국어 버전

<h2 class="">소개</h2>
<p class="">OS X의 키체인 시스템은 시스템 운영에 필요한 기밀 정보를 저장하는 시스템 키체인(System.keychain)과 각 사용자 관련 기밀 정보 및 애플리케이션의 기밀 정보를 저장하는 유저 키체인(login.keychain)으로 분류할 수 있다. OS X 매버릭스부터는 아이폰의 키체인 정보와 연동하는 아이클라우드 키체인(keychain-db2.db)이 추가되면서 아이폰에 저장된 무선 네트워크 정보, 메일, 연락처, 일정, 사파리에서 저장한 계정 정보를 공유할 수 있게 되었다. 이 외에도 시스템의 루트 인증서 정보를 저장하는 키체인도 있다.</p>
<p><img class="aligncenter full" title="" src="{{ site.baseurl }}/assets/1410664668_thumb.png" alt="" align="middle" /></p>
<p class="">기존 페이퍼를 통해 설명한 키체인의 마스터키 추출 방법은 사용자 키체인의 마스터키를 추출하는 방법이였다. 이번에는 시스템 키체인의 마스터 키를 통해 시스템 키체인을 복호화하는 방법을 알아보겠다.</p>
<p>시스템 키체인은 운영체제에 의해 사용되는 영역으로 커버로스, 애플 시스템 인증서, 소프트웨어 서명 인증서와 같은 운영체제 보안을 위한 것 외에도 사용자가 저장한 파일 공유 서버의 계정 및 패스워드 정보, OS X의 무선 장치인 AirPort에서 사용하기 위한 무선 네트워크의 SSID와 비밀번호를 보관한다. 특히 무선 네트워크 정보의 경우에는 사용자가 SSID를 등록한 시간, 마지막 비밀번호 변경 시간을 파악할 수 있으므로, 사용자의 맥 사용정보와 이동경로 유추에 도움이될 수 있다.</p>
<h2 class="">시스템 키체인 마스터키 추출</h2>
<p class="">시스템 키체인은 “/Library/Keychain”에 있다. 데이터 관리 방법은 사용자 키체인 관리 방법과 동일하므로, 시스템 키체인의 마스터 키만 알아내면 <a title="Keychain-Analysis-with-Mac-OS-X-Memory-Forensics" href="http://forensic.n0fate.com/wp-content/uploads/2012/12/Keychain-Analysis-with-Mac-OS-X-Memory-Forensics.pdf" target="_blank">동일한 방법으로 분석</a>할 수 있다.<br />
시스템 키체인의 마스터키는 “/private/var/db”에 “SystemKey”(kSystemUnlockFile) 파일에 보관되어 있다. 이 파일의 구조 Common Blob + UnlockBlob의 형태를 가진다.</p>
<h3 class="">Common Blob</h3>
<p><img class="aligncenter full" title="" src="{{ site.baseurl }}/assets/1410672682_thumb.png" alt="" align="middle" /></p>
<p class="">magicNumber(4bytes) : 0xFADE0711<br />
Current Version (4bytes) : 키체인 버전 정보. 기본 값인 0x00(version_MacOS_10_0)</p>
<h3 class="">UnlockBlob</h3>
<p><img class="full aligncenter" title="" src="{{ site.baseurl }}/assets/1410672633_thumb.png" alt="" align="middle" /><br />
Master Key(24bytes) : 192비트의 마스터 키<br />
Signature (16bytes) : DB키에 맞는 마스터 키인지 확인하기 위한 Signature</p>
<p>여기에서 마스터 키를 추출하면 기존 체인브레이커로 분석할 수 있다.</p>
<h2 class="">체인 브레이커에서 확인</h2>
<p>현재 개발 중인 체인브레이커에서는 ‘Is a Master Key?’에 체크하고 Master Key의 24바이트 헥사코드를 넣어주면 분석할 수 있다. 분석 결과는 다음과 같다.</p>
<p><img class="aligncenter full" title="" src="{{ site.baseurl }}/assets/1410672965_thumb.png" alt="" align="middle" /></p>
<p class="">Generic Password에는 각 무선 AP에 연결했던 SSID가 저장되어 있으며, 생성 날짜와 최종 수정 시간을 통해 무선 AP 최초 연결 시점과 패스워드 변경 시점을 파악할 수 있다.<br />
Internet Password에는 SMB로 연결했던 서버의 계정 및 패스워드 정보, 마운트한 볼륨 정보를 확인할 수 있다.</p>
<p>체인브레이커 <a title="Chainbreaker console" href="https://github.com/n0fate/chainbreaker" target="_blank">콘솔 버전</a>에서는 다음과 같이 확인할 수 있다.</p>
<pre class="lang:sh decode:true " title="chainbreaker">$ python chainbreaker.py -i ~/System.keychain -k xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
 [-] DB Key
[..SNIP..]
[+] Symmetric Key Table: 0x000064f4
[+] Generic Password Record
 [-] RecordSize : 0x000000e4
 [-] Record Number : 0x00000000
 [-] SECURE_STORAGE_GROUP(SSGP) Area : 0x0000002c
 [-] Create DateTime: 20140816021533Z
 [-] Last Modified DateTime: 20140816021533Z
 [-] Description : AirPort network password
 [-] Creator :
 [-] Type :
 [-] PrintName : [BLANK]
 [-] Alias :
 [-] Account : [BLANK]
 [-] Service : AirPort
 [-] Password
00000000:  xx xx xx xx xx xx xx  xx                       xxxxxxxx</pre>
<p>&nbsp;</p>
<h2 class="">결론</h2>
<p>사용자 키체인에는 사용자 애플리케이션 등 특정 사용자에게 국한된 기밀 정보를 추출하는데 도움을 받을 수 있다. 하지만, 무선 연결 정보나 마운트된 볼륨 정보 등, 운영체제 자체적으로 사용하는 정보를 추출할 수 없는 문제점을 가지고 있었다. 이 분석 방법을 사용하면, 분석가는 시스템에 남겨진 무선 AP와 원격 서버 연결 계정 정보를 추출함으로 행위 분석이나 추가적인 시스템 접근이 용이해지게 된다. 본 분석이 여러 보안인에게 도움이 되길 바란다.</p>
<p>&nbsp;</p>
<p>Reference : https://github.com/andrewdotn/chainbreaker/commit/b07b6840ed4013264199df7ef5a35282b591f5a1<br />

