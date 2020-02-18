---
layout: post
title: OS X El Capitan 이후 디스크 이미징
date: 2019-12-30 22:10:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Digital Forensics
tags: []
author: "n0fate"
comments: true
---

본래 macOS(편의상 모든 운영체제는 이 이름으로 표기)의 디스크 이미징 방법은 리눅스의 그것과 동일하게 디스크 장치를 입력으로한 덤프 방법을 사용하였다. 그런데, 이 방법이 OS X Elcapitan 이후부터 문제가 발생하였다.

최초 발단은 모 기업에 다니시는 유정호 형님에게 전화와서 blackbag 도구로 macOS 디스크 이미징을 하려고하는데 안되더라라는 내용이였다. 그리고 최근에도 모 분석가분께서 macOS Catalina에서 모 macOS 포렌식 도구로 이미징이 안된다는 연락을 받았다. 결국 이 문제는 OS X El Capitan에서 추가된 SIP(System Integrity Protection)으로 인해 발생한 문제였다.

## SIP이란?
SIP는 "Secure Integrity Protection"의 약어로 시스템의 주요 코드의 무결성을 유지하기 위해 추가된 기능이다. 이 기능은 크게 다음과 같은 일을 수행한다.

* macOS에 인가되지 않은(비 서명된) KEXT 로드 차단
* 특정 디렉터리 내의 바이너리 임의 수정(쓰기) 차단
* 특정 디렉터리와 시스템 파일의 컨텐츠 접근 차단
  * System-Only Locations(File  System  Protections in System Integrity Protection Guide  by Apple)
 	  * /bin
 	  * /sbin
 	  * /usr
 	  * /System
  
"/System" 디렉터리 내의 하위 디렉터리 중에는 운영체제 기본  Kernel Extension이 위치하고 있으며, "/dev/rdisk*" 장치도 이 익스텐션을 통해 생성되므로 SIP에 의해 보호를 받게 된다. 이로 인해 dd로 읽기 접근 시 오류가 발생한다. SIP의 활성화 여부는 터미널에서 다음 명령어를 입력하여 확인할 수 있다.

```bash
Computer:~ n0fate$ csrutil status
System Integrity Protection status: enabled.
Computer:~ n0fate$
```

현재 SW 디스크 덤프에 대해 이 문제를 해결하려면 SIP를 비활성화하는 방법 밖에 없다. SIP의 비활성화는 맥의 리커버리 모드(Recovery Mode)로 진입하여 할 수 있다.

macOS의 리커버리 모드 진입은 시스템 부팅 시점에 command+R을 누르고 있으면 진입이 가능하다.

이렇게 진입한 복구 모드에서 "Utility" -> "Terminal"을 눌러 실행한 후 다음과 같이 입력한다.

```bash
# csrutil disable
Successfully disabled System Integrity Protection. Please restart the machine for the changes to take effect.
```

시스템을 재부팅하면 "/dev/rdisk*"에 접근이 가능해지므로 디스크 덤프를 수행할 수 있다.

## 분석 시 고려사항

* 참고로 현재 blackbag의 도구는 이렇게 하더라도 디스크가 이미징이 안된다고 하니, ftkimager나 dd를 이용하는 것을 추천한다.
* 디스크 덤프 시에는 용량을 고려하여 분할압축을 하는 경우가 많은데, 이 경우 blackbag에서 해당 디스크를 마운트하지 못하는 문제를 가지고 있다. 이 문제를 해결하려면 ftkimager를 통해 하나의 읽기 전용 가상 디스크로 마운트하고 이를 blackbag에서 입력으로 받아 처리할 수 있다고 하니 참고하시기 바란다.
    - 유정호 형님에게 2019.R3 버전부터 문제없이 진행된다는 제보를 받았다!

## Reference
[System_Integrity_Protection_Guide](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40016462-CH1-DontLinkElementID_15)
