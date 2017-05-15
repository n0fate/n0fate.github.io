---
layout: post
title: Google Drive Forensics (Mac OS X)
date: 2013-08-21 17:31:36.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
tags:
- Cloud Storage
- Google Drive
- Mac OS X
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  _oembed_d69e8097cc08a244d4a8483691e48f76: '{{unknown}}'
  _oembed_24ce92f4dec68d5dfe89f16517666463: '{{unknown}}'
  _oembed_5994afcd6c93a51eca5b5cdc130124ff: '{{unknown}}'
  _oembed_11c76d6ffa5e4a3602f94b599b9f546d: '{{unknown}}'
  avada_post_views_count: '2534'
  _oembed_3b2362459dd50c4f023c84105fc7664b: '{{unknown}}'
  fusion_builder_content_backup: "최근에 디지털 포렌식 동향 관련한 자료를 수집하다보니, 작년부터 올해까지는 클라우드 스토리지(Cloud
    Storage)에 대한 포렌식 분석 기술이 상당히 많이 올라와 있음을 알게되었다.\n\n최근에는 스마트폰이나 노트북 등 한명의 사용자가 다수의
    스마트 디바이스를 제어하다보니, 하나의 작업을 여러 디바이스에서 같이 해야하는 필요성이 높아졌다. 클라우드 스토리지는 이러한 점을 충족시킬
    수 있는 제품으로,  예전에는 Dropbox, Copy와 같은 해당 솔루션만을 제공하는 전문업체에서만 서비스를 제공했으나, 올해부터 구글,
    마이크로소프트, 아마존 등의 대형 IT 업체가 이러한 서비스를 제공하기 시작했다.\n\n클라우드 스토리지에 대한 분석 글은 매우 잘 되어있고,
    많이 공개되어 있다. 하지만 대부분의 분석 글이 Windows 중심으로 기술되다보니, 다른 운영체제 특히 클라이언트로 윈도 다음으로 많이 사용되는
    Mac OS X에서는 어떠한 정보가 어느 위치에 있는지에 대한 정보를 알 수 없었다. 이에 본 포스팅에서는 이러한 클라우드 스토리지 중 하나인
    구글 드라이브의 아티팩트를 확인해보고자 한다. 물론 땡기면, 드롭박스나 아마존 클라우드 스토리지도 진행할 예정이다.\n\n일단 클라우드 스토리지가
    어떤 개념인지 간단히 알아보도록 하자.\n<h2>1. 클라우드 스토리지</h2>\n클라우드 스토리지는 기업형 네트워킹 저장소(networked
    enterprise storage) 모델로 데이터가 사용자의 컴퓨터에만 남는 것이 아니라 가상화된 스토리지에도 함께 저장되도록 한다. 가상화된
    스토리지는 대형 데이터센터에서 여러 개의 스토리지를 묶어서 관리되며, 클라이언트가 저장하는 데이터를 보관한다. 최근의 클라우드 스토리지는 사용자
    요청 시 데이터를 롤백하거나, 복원하는 기능도 내장하고 있다.\n\n이러한 스토리지 서비스는 크게 다음과 같은 요소에 디지털 흔적을 남긴다.\n<ul>\n\t<li>웹브라우저
    내역 : 클라우드 스토리지는 클라이언트를 통한 접근 말고도, 웹 브라우저를 이용한 접근이 가능하다. 웹브라우저 포렌식 도구로 이러한 정보를
    수집할 수 있다.</li>\n\t<li>애플리케이션 설치 정보 : 대부분의 클라우드 스토리지는 사용자 편의를 위해 클라이언트를 제공한다. 이러한
    클라이언트도 소프트웨어이기 때문에 애플리케이션 설치 정보가 존재한다.</li>\n\t<li>동기화된 파일 정보 및 컨텐츠 : 클라이언트가 존재하는
    클라우드 스토리지 서비스는 각 로컬 시스템에 실제 파일을 저장하고, 다른 시스템에서 파일을 변경할 경우 각 로컬 시스템의 파일을 변경하는 방법을
    사용한다. 이에 사용자가 명시적으로 Unlink 하지 않는 이상, 해당 파일이 디스크에 존재하므로, 디스크 포렌식 기법으로 분석이 가능하다.</li>\n</ul>\n&nbsp;\n<h2>2.
    구글 드라이브</h2>\n구글 드라이브는 2012년 4월에 공개된 파일 저장소이며 동기화 서비스를 제공하는 클라우드 스토리지이다. 구글 드라이브의
    특징은 다음과 같다.\n<ul>\n\t<li>클라우드 스토리지</li>\n\t<li>파일 공유 : 특정 파일을 구글에서 검색 가능하도록 공개(Public)하거나,
    특정 사용자에게 접근 링크를 제공(Private)하는 식의 파일 공유가 가능하다.</li>\n\t<li>문서 편집 기능 제공 : 구글은 예전부터
    구글은 <a title=\"Google Docs\" href=\"http://docs.google.com\" target=\"_blank\">Google
    Docs</a>라는 서비스를 통해 웹기반으로 doc, xls, ppt를 수정하는 기능을 제공했다.</li>\n\t<li>협동 작업 : 여러
    시스템에서 문서 편집이 가능하며, 여러 사용자가 수정이 가능하다. 여러 사용자가 접근하는 문서의 경우, 수정한 사람의 구글 메일 주소가 나타난다.</li>\n</ul>\n구글
    드라이브는 기본적으로 15기가를 제공하며, 기존의 많은 구글 독스 사용자가 이용하기 때문에 생각보다 많은 사용자가 이용하는 서비스이다. 특히,
    구글 드라이브는 파일 포맷 구분없이 모든 파일을 업로드하고 공유할 수 있다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-1.44.09.png\"><img
    class=\"aligncenter size-full wp-image-449\" alt=\"Google Docs\" src=\"http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-1.44.09.png\"
    width=\"771\" height=\"439\" /></a>\n\n&nbsp;\n<h2>3. 포렌식 아티펙트 분석</h2>\n클라우드 스토리지
    포렌식을 진행할 땐 공통적으로 다음과 같은 요소를 식별해야 한다.\n<ol>\n\t<li>사용자 계정 정보(아이디, 패스워드, 이메일 등)</li>\n\t<li>클라우드에
    저장된 파일의 메타 정보</li>\n\t<li>클라우드에 저장된 파일의 컨텐츠</li>\n</ol>\n여기에선 각 요소 별로 분석을 진행하도록
    한다.\n<h3>1. 사용자 계정 정보</h3>\n구글 드라이브는 사용자 계정 정보를 Mac OS X의 키체인 시스템(Keychain System)에
    저장한다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-2.02.55.png\"><img
    class=\"aligncenter size-full wp-image-451\" alt=\"Keychain : Google Drive\" src=\"http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-2.02.55.png\"
    width=\"533\" height=\"348\" /></a>\n\n키체인 시스템에 대한 설명은 이전에도 많이 했으니 여기에선 간략하게 설명하겠다.\n<blockquote>키체인
    시스템은 Mac OS X에서 사용하는 수많은 애플리케이션의 패스워드를 통합 관리하기 위한 것으로 단순히 계정을 통합 관리하는 것 뿐만 아니라,
    높은 보안 수준으로 관리하고 있다. 이 시스템을 이용함으로, 각 애플리케이션은 키체인에 새로운 키 엔트리를 생성하고 그곳에 사용자 아이디와
    패스워드를 저장한다. 애플리케이션은 각 엔트리에 접근할 때마다 사용자의 허락을 받아야 한다. 사용자가 선택할 수 있는 건 크게 3가지이다.\n\n<strong>항상
    허용</strong> : 해당 애플리케이션의 정보가 키 엔트리에 저장되어, 앞으로 키 엔트리에 해당 애플리케이션이 접근하는 경우에는 사용자
    고지 없이 접근이 가능하다.\n\n<strong>허용</strong> : 1회성 허용으로, 추 후 재접근 시 다시 사용자에게 접근을 요청한다.\n\n<strong>거부</strong>
    : 애플리케이션이 접근하지 못하도록 한다. 1회성이다.</blockquote>\n키체인 시스템에 저장된 키 엔트리는 chainbreaker로
    분석할 수 있다. chainbreaker로 사용자의 'login.keychain' 파일을 분석하여, 구글 드라이브의 사용자 ID와 Password를
    평문으로 획득할 수 있다. chainbreaker를 이용한 분석 방법은 이 블로그에 작성한 다음 연재를 통해 확인할 수 있다.\n<ul>\n\t<li>Keychain
    Forensics : <a title=\"Keychain Forensics : Part I\" href=\"http://forensic.n0fate.com/?p=158\"
    target=\"_blank\">Part I</a>, <a title=\"Keychain Forensics : Part II\" href=\"http://forensic.n0fate.com/?p=157\"
    target=\"_blank\">Part II</a>, <a title=\"Keychain Forensics : Part III\" href=\"http://forensic.n0fate.com/?p=156\"
    target=\"_blank\">Part III</a></li>\n</ul>\n&nbsp;\n<h3>2. 클라우드에 저장된 파일의 메타 정보</h3>\n구글
    드라이브는 클라우드에 저장되는 각 파일의 메타 정보를 하나의 데이터베이스 형태로 유지한다. 해당 데이터베이스는 \"/Users/&lt;username&gt;/Library/Application
    Support/Google/Drive\"에 위치한다.\n<blockquote>drwxr-xr-x 15 n0fate staff 510 8 3
    19:56 .\ndrwxr-xr-x 3 n0fate staff 102 7 23 16:51 ..\n-rw------- 1 n0fate staff
    3245 8 3 19:56 cacerts\ndrwxr-xr-x 5 n0fate staff 170 8 3 19:56 cloud_graph\ndrwxr-xr-x
    2 n0fate staff 68 7 23 16:51 CrashReports\ndrwxr-xr-x 3 n0fate staff 102 7 23
    16:52 FinderExt.bundle\n-rw-r--r-- 1 n0fate staff 0 8 3 19:56 lockfile\ndrwxr-xr-x
    3 n0fate staff 102 7 23 16:52 mach_inject_bundle_stub.bundle\n-rw-r--r-- 1 n0fate
    staff 20480 7 23 16:51 snapshot.db\n-rw-r--r-- 1 n0fate staff 32768 8 4 14:26
    snapshot.db-shm\n-rw-r--r-- 1 n0fate staff 829000 8 3 19:56 snapshot.db-wal\n-rw-r--r--
    1 n0fate staff 5120 8 3 19:56 sync_config.db\n-rw-r--r-- 1 n0fate staff 32768
    8 4 14:26 sync_config.db-shm\n-rw-r--r-- 1 n0fate staff 12608 8 3 19:56 sync_config.db-wal\n-rw-r--r--
    1 n0fate staff 2124254 8 21 13:37 sync_log.log</blockquote>\n이 디렉터리에는 2개의 데이터베이스에
    주요 정보를 저장한다.\n<ul>\n\t<li>snapshot.db : 구글 클라우드 서버에 있는 파일 엔트리와 각 파일 간의 관계(부모-자식
    관계, 폴더도 엔트리로 관리되기 때문임), 로컬 시스템에 있는 파일 엔트리와 각 파일 간의 관계(부모-자식 관계)</li>\n\t<li>sync_config.db
    : 등록한 사용자의 구글 아이디(email address), 소프트웨어가 설치된 경우, 소프트웨어 버전</li>\n</ul>\n그런데 한가지
    특이한 점이 있다. 위에서 설명한 두 개의 데이터베이스를 보면, 동일한 이름의 'db', 'db-wal', 'db-shm' 파일이 존재한다.
    file 명령어로 확인해보면 db는 sqlite database이고 나머지 두 파일은 그냥 data로 출력된다.\n<blockquote>n0fate-MacBook-Air:Drive
    n0fate$ file snapshot.db*\nsnapshot.db: SQLite 3.x database\nsnapshot.db-shm:
    data\nsnapshot.db-wal: data\nn0fate-MacBook-Air:Drive n0fate$</blockquote>\n그리고
    db 파일은 스키마 정의만 있을 뿐 안에 아무런 데이터가 저장되어 있지 않으며, sqlite db brower에서도 아무런 정보를 보여주지
    못한다.  이는 WAL(Write-Ahead Logging)이 활성화된 SQLite3 Database이기 때문이다.\n\nWAL은 SQLite3
    3.7.0에서부터 등장한 개념으로 모든 데이터 트랜젝션을 데이터베이스 파일에 직접 저장하지 않고, Shared Memory Map에 맵핑한
    Write-Ahead Logging에 저장하고 필요할 때 한번에 DB 파일에 한번에 커밋하는 기능을 제공한다. 대부분의 데이터 트랜잭션을 메모리에서
    처리하고, 애플리케이션의 데이터베이스 정보 요청도 전부 메모리에서 처리하므로 시스템 성능이 향상된다. WAL에 저장된 모든 데이터는 checkpoint라는
    명령을 통해서 DB에 저장된다. WAL도 SQLite3 DB와 마찬가지로 페이지 단위로 곡안을 할당하며, 1000페이지가 가득차게 되면 자동으로
    checkpoint 명령을 실행하여 WAL의 데이터를 정리한다.\n\nWAL은 SQLite3 DB Browser에서 지원하지 않으므로, checkpoint명령으로
    DB에 저장하게 만들어야 한다. 다음과 같은 명령으로 작업을 진행하면 된다.\n<blockquote>n0fate-MacBook-Air:Drive
    n0fate$ sqlite3 snapshot.db\nSQLite version 3.7.12 2012-04-03 19:43:07\nEnter
    \".help\" for instructions\nEnter SQL statements terminated with a \";\"\nsqlite&gt;
    PRAGMA journal_mode=DELETE;\ndelete\nsqlite&gt; .quit\n\nn0fate-MacBook-Air:Drive
    n0fate$ ls -al | grep snapshot.db\n-rw-r--r-- 1 n0fate staff 48128 8 21 16:40
    snapshot.db\n-rw-r--r-- 1 n0fate staff 32768 8 21 16:39 snapshot.db-shm\n-rw-r--r--
    1 n0fate staff 0 8 21 16:40 snapshot.db-wal\nn0fate-MacBook-Air:Drive n0fate$</blockquote>\n명령어를
    수행하면 WAL 파일이 0바이트가되면서 모든 트랙잭션이 반영되어 snapshot.db에 저장된다. 단 이 방법의 경우, WAL에서 저장한 트랜잭션
    정보가 반영 안되기 때문에 삭제된 레코드를 복구하고자 할 때는 WAL에 대한 별도의 분석이 필요하다. 외국에서 이 부분에 대한 연구가 진행되고
    있는데, 추 후 시간이 좀 나면 정리해보겠다.\n\nSQLite DB Browser로 보면 다음과 같이 테이블 정보를 확인할 수 있다.\n\n<a
    href=\"http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-4.46.20.png\"><img
    class=\"aligncenter size-full wp-image-457\" alt=\"SQLite DB Browser\" src=\"http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-4.46.20.png\"
    width=\"702\" height=\"574\" /></a>\n\ncloud_entry와 local_entry 테이블에는 각각 다음 정보가
    저장된다.\n\nTable Name : cloud_entry\n[table]\nColumn, Description\nfilename, 구글
    클라우드에 저장된 파일명\nmodified, 수정된 시간 (UTC+0)\ncreated, 생성된 시간 (UTC+0)\ndoc_type, 문서
    타입. Folder(0) 기타 파일(1) Google PPT(2) Unknown(3) Google Form(4) Google Drawing(5)
    Google Document(6) Google Table(7)\nremoved, 파일 삭제 여부. 추 후 구글 드라이브는 삭제했던 파일을 복원하는
    기능을 가지고 있음\nurl, 편집할 컨텐츠를 저장한 URL\nsize, 웹 상의 파일  크기 정보. 구글 문서 형식(gdoc/gsheet/gslide)은
    무조건 0으로 저장\nshared, 다른 사용자에게 공유 여부\n[/table]\n\nTable Name : local_entry\n[table]\nColumn,
    Description\nfilename, 로컬에 저장된 파일명\nmodified, 수정된 시간 (UTC+0)\nsize, 로컬에 저장된 파일의
    크기\n[/table]\n\n&nbsp;\n<h3> 4. 클라우드에 저장된 파일의 컨텐츠</h3>\n구글 드라이브는 보통 공유 디렉터리를 \"/Users/&lt;username&gt;/Google
    Drive/\"에 위치한다. 구글 드라이브에 저장되는 파일은 메타 정보만 보관하는 파일(gdoc, gsheet, gslide)와 컨텐츠 전체를
    로컬에 보관하는 파일(그 외 모든 파일)로 나눌 수 있다.\n<h4>1. 메타 정보만 보관하는 파일</h4>\n구글에서 생성한 document,
    excel, powerpoint는 각각 gdoc, gsheet, gslide 확장자로 로컬에 생성된다. 이 파일은 실제 파일 컨텐츠를 저장하진
    않으며, URL과 Resource ID 정보만 담고 있다. 다음은 하나의 gdoc 파일의 내용을 확인한 것이다.\n<blockquote>{\"url\":
    \"https://docs.google.com/document/d/1op6cnFcnjgyhN9hdLKwxGpAwXgc1AV5vhs7DJGvzwYs/edit?usp=docslist_api\",
    \"resource_id\": \"document:1op6cnFcnjgyhN9hdLKwxGpAwXgc1AV5vhs7DJGvzwYs\"}</blockquote>\n구글
    드라이브는 이 파일을 사용자가 더블클릭했을 경우, 웹 브라우저를 실행하면서 URL의 내용을 전달한다. 사용자는 새롭게 띄워진 웹브라우저에서
    접속된 구글 드라이브의 문서 편집기로 데이터를 수정할 수 있다.\n<h4>2. 로컬에 보관하는 파일</h4>\n1에서 설명한 구글 드라이브에서
    만든 문서 파일을 제외한 모든 파일은 드롭박스와 같이 파일 자체를 그대로 저장한다. 이 파일은 디스크 포렌식 기술로 접근하여 데이터를 추출
    분석하면 된다.\n\n&nbsp;\n<h2>4. 결론</h2>\n구글 드라이브는 웹에서 문서 편집이 가능하며, 드롭박스와 같이 일반 파일도
    관리할 수 있는 편리한 클라우드 스토리지이다. SQLite 데이터베이스에 메타데이터 정보를 가지고 있으며, 클라우드의 데이터와 로컬 데이터의
    메타 정보를 각각 다른 테이블에 저장하여, 포렌식 타임라인 분석에 유용하게 사용될 수 있다.\n\n필요 시에는 키체인 정보 복호화 시도를 통해,
    구글의 메타데이터만 존재하는 구글 문서 파일의 컨텐츠를 확인할 수도 있으며, 구글의 다른 서비스(유투브, 이메일, 캘린더) 등의 정보를 접근할
    수도 있다.\n\n&nbsp;\n<h2> Reference</h2>\nhttp://sysforensics.org/2012/05/google-drive-forensics-notes.html\n\n&nbsp;\n\n&nbsp;"
  fusion_builder_converted: 'yes'
  _oembed_e6e29ccaa3dcbf11eb2651a0f5d5ecbd: '{{unknown}}'
  _oembed_4d9b2cd4f20909cc1e4b721622a26921: '{{unknown}}'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>최근에 디지털 포렌식 동향 관련한 자료를 수집하다보니, 작년부터 올해까지는 클라우드 스토리지(Cloud Storage)에 대한 포렌식 분석 기술이 상당히 많이 올라와 있음을 알게되었다.</p>
<p>최근에는 스마트폰이나 노트북 등 한명의 사용자가 다수의 스마트 디바이스를 제어하다보니, 하나의 작업을 여러 디바이스에서 같이 해야하는 필요성이 높아졌다. 클라우드 스토리지는 이러한 점을 충족시킬 수 있는 제품으로,  예전에는 Dropbox, Copy와 같은 해당 솔루션만을 제공하는 전문업체에서만 서비스를 제공했으나, 올해부터 구글, 마이크로소프트, 아마존 등의 대형 IT 업체가 이러한 서비스를 제공하기 시작했다.</p>
<p>클라우드 스토리지에 대한 분석 글은 매우 잘 되어있고, 많이 공개되어 있다. 하지만 대부분의 분석 글이 Windows 중심으로 기술되다보니, 다른 운영체제 특히 클라이언트로 윈도 다음으로 많이 사용되는 Mac OS X에서는 어떠한 정보가 어느 위치에 있는지에 대한 정보를 알 수 없었다. 이에 본 포스팅에서는 이러한 클라우드 스토리지 중 하나인 구글 드라이브의 아티팩트를 확인해보고자 한다. 물론 땡기면, 드롭박스나 아마존 클라우드 스토리지도 진행할 예정이다.</p>
<p>일단 클라우드 스토리지가 어떤 개념인지 간단히 알아보도록 하자.</p>
<h2>1. 클라우드 스토리지</h2>
<p>클라우드 스토리지는 기업형 네트워킹 저장소(networked enterprise storage) 모델로 데이터가 사용자의 컴퓨터에만 남는 것이 아니라 가상화된 스토리지에도 함께 저장되도록 한다. 가상화된 스토리지는 대형 데이터센터에서 여러 개의 스토리지를 묶어서 관리되며, 클라이언트가 저장하는 데이터를 보관한다. 최근의 클라우드 스토리지는 사용자 요청 시 데이터를 롤백하거나, 복원하는 기능도 내장하고 있다.</p>
<p>이러한 스토리지 서비스는 크게 다음과 같은 요소에 디지털 흔적을 남긴다.</p>
<ul>
<li>웹브라우저 내역 : 클라우드 스토리지는 클라이언트를 통한 접근 말고도, 웹 브라우저를 이용한 접근이 가능하다. 웹브라우저 포렌식 도구로 이러한 정보를 수집할 수 있다.</li>
<li>애플리케이션 설치 정보 : 대부분의 클라우드 스토리지는 사용자 편의를 위해 클라이언트를 제공한다. 이러한 클라이언트도 소프트웨어이기 때문에 애플리케이션 설치 정보가 존재한다.</li>
<li>동기화된 파일 정보 및 컨텐츠 : 클라이언트가 존재하는 클라우드 스토리지 서비스는 각 로컬 시스템에 실제 파일을 저장하고, 다른 시스템에서 파일을 변경할 경우 각 로컬 시스템의 파일을 변경하는 방법을 사용한다. 이에 사용자가 명시적으로 Unlink 하지 않는 이상, 해당 파일이 디스크에 존재하므로, 디스크 포렌식 기법으로 분석이 가능하다.</li>
</ul>
<p>&nbsp;</p>
<h2>2. 구글 드라이브</h2>
<p>구글 드라이브는 2012년 4월에 공개된 파일 저장소이며 동기화 서비스를 제공하는 클라우드 스토리지이다. 구글 드라이브의 특징은 다음과 같다.</p>
<ul>
<li>클라우드 스토리지</li>
<li>파일 공유 : 특정 파일을 구글에서 검색 가능하도록 공개(Public)하거나, 특정 사용자에게 접근 링크를 제공(Private)하는 식의 파일 공유가 가능하다.</li>
<li>문서 편집 기능 제공 : 구글은 예전부터 구글은 <a title="Google Docs" href="http://docs.google.com" target="_blank">Google Docs</a>라는 서비스를 통해 웹기반으로 doc, xls, ppt를 수정하는 기능을 제공했다.</li>
<li>협동 작업 : 여러 시스템에서 문서 편집이 가능하며, 여러 사용자가 수정이 가능하다. 여러 사용자가 접근하는 문서의 경우, 수정한 사람의 구글 메일 주소가 나타난다.</li>
</ul>
<p>구글 드라이브는 기본적으로 15기가를 제공하며, 기존의 많은 구글 독스 사용자가 이용하기 때문에 생각보다 많은 사용자가 이용하는 서비스이다. 특히, 구글 드라이브는 파일 포맷 구분없이 모든 파일을 업로드하고 공유할 수 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-1.44.09.png"><img class="aligncenter size-full wp-image-449" alt="Google Docs" src="{{ site.baseurl }}/assets/&#49828;&#53356;&#47536;&#49399;-2013-08-21-&#50724;&#54980;-1.44.09.png" width="771" height="439" /></a></p>
<p>&nbsp;</p>
<h2>3. 포렌식 아티펙트 분석</h2>
<p>클라우드 스토리지 포렌식을 진행할 땐 공통적으로 다음과 같은 요소를 식별해야 한다.</p>
<ol>
<li>사용자 계정 정보(아이디, 패스워드, 이메일 등)</li>
<li>클라우드에 저장된 파일의 메타 정보</li>
<li>클라우드에 저장된 파일의 컨텐츠</li>
</ol>
<p>여기에선 각 요소 별로 분석을 진행하도록 한다.</p>
<h3>1. 사용자 계정 정보</h3>
<p>구글 드라이브는 사용자 계정 정보를 Mac OS X의 키체인 시스템(Keychain System)에 저장한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-2.02.55.png"><img class="aligncenter size-full wp-image-451" alt="Keychain : Google Drive" src="{{ site.baseurl }}/assets/&#49828;&#53356;&#47536;&#49399;-2013-08-21-&#50724;&#54980;-2.02.55.png" width="533" height="348" /></a></p>
<p>키체인 시스템에 대한 설명은 이전에도 많이 했으니 여기에선 간략하게 설명하겠다.</p>
<blockquote><p>키체인 시스템은 Mac OS X에서 사용하는 수많은 애플리케이션의 패스워드를 통합 관리하기 위한 것으로 단순히 계정을 통합 관리하는 것 뿐만 아니라, 높은 보안 수준으로 관리하고 있다. 이 시스템을 이용함으로, 각 애플리케이션은 키체인에 새로운 키 엔트리를 생성하고 그곳에 사용자 아이디와 패스워드를 저장한다. 애플리케이션은 각 엔트리에 접근할 때마다 사용자의 허락을 받아야 한다. 사용자가 선택할 수 있는 건 크게 3가지이다.</p>
<p><strong>항상 허용</strong> : 해당 애플리케이션의 정보가 키 엔트리에 저장되어, 앞으로 키 엔트리에 해당 애플리케이션이 접근하는 경우에는 사용자 고지 없이 접근이 가능하다.</p>
<p><strong>허용</strong> : 1회성 허용으로, 추 후 재접근 시 다시 사용자에게 접근을 요청한다.</p>
<p><strong>거부</strong> : 애플리케이션이 접근하지 못하도록 한다. 1회성이다.</p></blockquote>
<p>키체인 시스템에 저장된 키 엔트리는 chainbreaker로 분석할 수 있다. chainbreaker로 사용자의 'login.keychain' 파일을 분석하여, 구글 드라이브의 사용자 ID와 Password를 평문으로 획득할 수 있다. chainbreaker를 이용한 분석 방법은 이 블로그에 작성한 다음 연재를 통해 확인할 수 있다.</p>
<ul>
<li>Keychain Forensics : <a title="Keychain Forensics : Part I" href="http://forensic.n0fate.com/?p=158" target="_blank">Part I</a>, <a title="Keychain Forensics : Part II" href="http://forensic.n0fate.com/?p=157" target="_blank">Part II</a>, <a title="Keychain Forensics : Part III" href="http://forensic.n0fate.com/?p=156" target="_blank">Part III</a></li>
</ul>
<p>&nbsp;</p>
<h3>2. 클라우드에 저장된 파일의 메타 정보</h3>
<p>구글 드라이브는 클라우드에 저장되는 각 파일의 메타 정보를 하나의 데이터베이스 형태로 유지한다. 해당 데이터베이스는 "/Users/&lt;username&gt;/Library/Application Support/Google/Drive"에 위치한다.</p>
<blockquote><p>drwxr-xr-x 15 n0fate staff 510 8 3 19:56 .<br />
drwxr-xr-x 3 n0fate staff 102 7 23 16:51 ..<br />
-rw------- 1 n0fate staff 3245 8 3 19:56 cacerts<br />
drwxr-xr-x 5 n0fate staff 170 8 3 19:56 cloud_graph<br />
drwxr-xr-x 2 n0fate staff 68 7 23 16:51 CrashReports<br />
drwxr-xr-x 3 n0fate staff 102 7 23 16:52 FinderExt.bundle<br />
-rw-r--r-- 1 n0fate staff 0 8 3 19:56 lockfile<br />
drwxr-xr-x 3 n0fate staff 102 7 23 16:52 mach_inject_bundle_stub.bundle<br />
-rw-r--r-- 1 n0fate staff 20480 7 23 16:51 snapshot.db<br />
-rw-r--r-- 1 n0fate staff 32768 8 4 14:26 snapshot.db-shm<br />
-rw-r--r-- 1 n0fate staff 829000 8 3 19:56 snapshot.db-wal<br />
-rw-r--r-- 1 n0fate staff 5120 8 3 19:56 sync_config.db<br />
-rw-r--r-- 1 n0fate staff 32768 8 4 14:26 sync_config.db-shm<br />
-rw-r--r-- 1 n0fate staff 12608 8 3 19:56 sync_config.db-wal<br />
-rw-r--r-- 1 n0fate staff 2124254 8 21 13:37 sync_log.log</p></blockquote>
<p>이 디렉터리에는 2개의 데이터베이스에 주요 정보를 저장한다.</p>
<ul>
<li>snapshot.db : 구글 클라우드 서버에 있는 파일 엔트리와 각 파일 간의 관계(부모-자식 관계, 폴더도 엔트리로 관리되기 때문임), 로컬 시스템에 있는 파일 엔트리와 각 파일 간의 관계(부모-자식 관계)</li>
<li>sync_config.db : 등록한 사용자의 구글 아이디(email address), 소프트웨어가 설치된 경우, 소프트웨어 버전</li>
</ul>
<p>그런데 한가지 특이한 점이 있다. 위에서 설명한 두 개의 데이터베이스를 보면, 동일한 이름의 'db', 'db-wal', 'db-shm' 파일이 존재한다. file 명령어로 확인해보면 db는 sqlite database이고 나머지 두 파일은 그냥 data로 출력된다.</p>
<blockquote><p>n0fate-MacBook-Air:Drive n0fate$ file snapshot.db*<br />
snapshot.db: SQLite 3.x database<br />
snapshot.db-shm: data<br />
snapshot.db-wal: data<br />
n0fate-MacBook-Air:Drive n0fate$</p></blockquote>
<p>그리고 db 파일은 스키마 정의만 있을 뿐 안에 아무런 데이터가 저장되어 있지 않으며, sqlite db brower에서도 아무런 정보를 보여주지 못한다.  이는 WAL(Write-Ahead Logging)이 활성화된 SQLite3 Database이기 때문이다.</p>
<p>WAL은 SQLite3 3.7.0에서부터 등장한 개념으로 모든 데이터 트랜젝션을 데이터베이스 파일에 직접 저장하지 않고, Shared Memory Map에 맵핑한 Write-Ahead Logging에 저장하고 필요할 때 한번에 DB 파일에 한번에 커밋하는 기능을 제공한다. 대부분의 데이터 트랜잭션을 메모리에서 처리하고, 애플리케이션의 데이터베이스 정보 요청도 전부 메모리에서 처리하므로 시스템 성능이 향상된다. WAL에 저장된 모든 데이터는 checkpoint라는 명령을 통해서 DB에 저장된다. WAL도 SQLite3 DB와 마찬가지로 페이지 단위로 곡안을 할당하며, 1000페이지가 가득차게 되면 자동으로 checkpoint 명령을 실행하여 WAL의 데이터를 정리한다.</p>
<p>WAL은 SQLite3 DB Browser에서 지원하지 않으므로, checkpoint명령으로 DB에 저장하게 만들어야 한다. 다음과 같은 명령으로 작업을 진행하면 된다.</p>
<blockquote><p>n0fate-MacBook-Air:Drive n0fate$ sqlite3 snapshot.db<br />
SQLite version 3.7.12 2012-04-03 19:43:07<br />
Enter ".help" for instructions<br />
Enter SQL statements terminated with a ";"<br />
sqlite&gt; PRAGMA journal_mode=DELETE;<br />
delete<br />
sqlite&gt; .quit</p>
<p>n0fate-MacBook-Air:Drive n0fate$ ls -al | grep snapshot.db<br />
-rw-r--r-- 1 n0fate staff 48128 8 21 16:40 snapshot.db<br />
-rw-r--r-- 1 n0fate staff 32768 8 21 16:39 snapshot.db-shm<br />
-rw-r--r-- 1 n0fate staff 0 8 21 16:40 snapshot.db-wal<br />
n0fate-MacBook-Air:Drive n0fate$</p></blockquote>
<p>명령어를 수행하면 WAL 파일이 0바이트가되면서 모든 트랙잭션이 반영되어 snapshot.db에 저장된다. 단 이 방법의 경우, WAL에서 저장한 트랜잭션 정보가 반영 안되기 때문에 삭제된 레코드를 복구하고자 할 때는 WAL에 대한 별도의 분석이 필요하다. 외국에서 이 부분에 대한 연구가 진행되고 있는데, 추 후 시간이 좀 나면 정리해보겠다.</p>
<p>SQLite DB Browser로 보면 다음과 같이 테이블 정보를 확인할 수 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2013/08/스크린샷-2013-08-21-오후-4.46.20.png"><img class="aligncenter size-full wp-image-457" alt="SQLite DB Browser" src="{{ site.baseurl }}/assets/&#49828;&#53356;&#47536;&#49399;-2013-08-21-&#50724;&#54980;-4.46.20.png" width="702" height="574" /></a></p>
<p>cloud_entry와 local_entry 테이블에는 각각 다음 정보가 저장된다.</p>
<p>Table Name : cloud_entry<br />
[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][table]<br />
Column, Description<br />
filename, 구글 클라우드에 저장된 파일명<br />
modified, 수정된 시간 (UTC+0)<br />
created, 생성된 시간 (UTC+0)<br />
doc_type, 문서 타입. Folder(0) 기타 파일(1) Google PPT(2) Unknown(3) Google Form(4) Google Drawing(5) Google Document(6) Google Table(7)<br />
removed, 파일 삭제 여부. 추 후 구글 드라이브는 삭제했던 파일을 복원하는 기능을 가지고 있음<br />
url, 편집할 컨텐츠를 저장한 URL<br />
size, 웹 상의 파일  크기 정보. 구글 문서 형식(gdoc/gsheet/gslide)은 무조건 0으로 저장<br />
shared, 다른 사용자에게 공유 여부<br />
[/table]</p>
<p>Table Name : local_entry<br />
[/fusion_builder_column][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][table]<br />
Column, Description<br />
filename, 로컬에 저장된 파일명<br />
modified, 수정된 시간 (UTC+0)<br />
size, 로컬에 저장된 파일의 크기<br />
[/table]</p>
<p>&nbsp;</p>
<h3> 4. 클라우드에 저장된 파일의 컨텐츠</h3>
<p>구글 드라이브는 보통 공유 디렉터리를 "/Users/&lt;username&gt;/Google Drive/"에 위치한다. 구글 드라이브에 저장되는 파일은 메타 정보만 보관하는 파일(gdoc, gsheet, gslide)와 컨텐츠 전체를 로컬에 보관하는 파일(그 외 모든 파일)로 나눌 수 있다.</p>
<h4>1. 메타 정보만 보관하는 파일</h4>
<p>구글에서 생성한 document, excel, powerpoint는 각각 gdoc, gsheet, gslide 확장자로 로컬에 생성된다. 이 파일은 실제 파일 컨텐츠를 저장하진 않으며, URL과 Resource ID 정보만 담고 있다. 다음은 하나의 gdoc 파일의 내용을 확인한 것이다.</p>
<blockquote><p>{"url": "https://docs.google.com/document/d/1op6cnFcnjgyhN9hdLKwxGpAwXgc1AV5vhs7DJGvzwYs/edit?usp=docslist_api", "resource_id": "document:1op6cnFcnjgyhN9hdLKwxGpAwXgc1AV5vhs7DJGvzwYs"}</p></blockquote>
<p>구글 드라이브는 이 파일을 사용자가 더블클릭했을 경우, 웹 브라우저를 실행하면서 URL의 내용을 전달한다. 사용자는 새롭게 띄워진 웹브라우저에서 접속된 구글 드라이브의 문서 편집기로 데이터를 수정할 수 있다.</p>
<h4>2. 로컬에 보관하는 파일</h4>
<p>1에서 설명한 구글 드라이브에서 만든 문서 파일을 제외한 모든 파일은 드롭박스와 같이 파일 자체를 그대로 저장한다. 이 파일은 디스크 포렌식 기술로 접근하여 데이터를 추출 분석하면 된다.</p>
<p>&nbsp;</p>
<h2>4. 결론</h2>
<p>구글 드라이브는 웹에서 문서 편집이 가능하며, 드롭박스와 같이 일반 파일도 관리할 수 있는 편리한 클라우드 스토리지이다. SQLite 데이터베이스에 메타데이터 정보를 가지고 있으며, 클라우드의 데이터와 로컬 데이터의 메타 정보를 각각 다른 테이블에 저장하여, 포렌식 타임라인 분석에 유용하게 사용될 수 있다.</p>
<p>필요 시에는 키체인 정보 복호화 시도를 통해, 구글의 메타데이터만 존재하는 구글 문서 파일의 컨텐츠를 확인할 수도 있으며, 구글의 다른 서비스(유투브, 이메일, 캘린더) 등의 정보를 접근할 수도 있다.</p>
<p>&nbsp;</p>
<h2> Reference</h2>
<p>http://sysforensics.org/2012/05/google-drive-forensics-notes.html</p>
<p>&nbsp;</p>
<p>&nbsp;[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
