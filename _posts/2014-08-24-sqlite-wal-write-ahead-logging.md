---
layout: post
title: SQLite WAL (Write-Ahead Logging)
date: 2014-08-24 22:15:17.000000000 +09:00
type: post
published: true
status: publish
categories:
- OS Artifacts
- Tools
tags:
- SQLite
- WAL
- walitean
meta:
  _edit_last: '1'
  slide_template: default
  pyre_video: ''
  pyre_full_width: 'no'
  pyre_sidebar_position: default
  pyre_display_header: 'yes'
  pyre_transparent_header: default
  pyre_displayed_menu: default
  pyre_display_footer: default
  pyre_display_copyright: default
  pyre_fimg_width: auto
  pyre_fimg_height: auto
  pyre_image_rollover_icons: linkzoom
  pyre_link_icon_url: ''
  pyre_related_posts: default
  pyre_slider_position: default
  pyre_slider_type: 'no'
  pyre_slider: '0'
  pyre_wooslider: '0'
  pyre_revslider: '0'
  pyre_elasticslider: '0'
  pyre_fallback: ''
  pyre_page_bg_layout: default
  pyre_page_bg: ''
  pyre_page_bg_color: ''
  pyre_page_bg_full: 'no'
  pyre_page_bg_repeat: repeat
  pyre_wide_page_bg: ''
  pyre_wide_page_bg_color: ''
  pyre_wide_page_bg_full: 'no'
  pyre_wide_page_bg_repeat: repeat
  pyre_header_bg: ''
  pyre_header_bg_color: ''
  pyre_header_bg_full: 'no'
  pyre_header_bg_repeat: repeat
  pyre_page_title: default
  pyre_page_title_text: 'yes'
  pyre_page_title_custom_text: ''
  pyre_page_title_custom_subheader: ''
  pyre_page_title_height: ''
  pyre_page_title_bar_bg: ''
  pyre_page_title_bar_bg_retina: ''
  pyre_page_title_bar_bg_full: default
  pyre_page_title_bar_bg_color: ''
  pyre_page_title_bg_parallax: default
  avada_post_views_count: '4124'
  fusion_builder_status: inactive
  _thumbnail_id: '1080'
  sbg_selected_sidebar: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_replacement: a:1:{i:0;s:12:"Blog Sidebar";}
  sbg_selected_sidebar_2: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_2_replacement: a:1:{i:0;s:0:"";}
  fusion_builder_content_backup: "<h2>소개</h2>\n과거 데이터베이스는 갑작스런 이벤트로 인해 데이터베이스가 손상되는
    것을 방지하기 위해 트랜잭션(transaction)에 저널링(Jorunaling)을 사용하였다. 데이터베이스의 저널링 기법은 롤백 저널(Rollback
    Journal)이라고 불리며, 데이터베이스 파일 내에 저널링 데이터 보관 영역을 두고, 필요할 때마다 원래 데이터베이스 영역에 저장(commit)하는
    방법을 사용하였다. 하지만 롤백 저널은 레코드 조회와 삽입을 동시에 수행할 수가 없는 문제(Database Lock)가 있었다. 이에 최근에
    사용되는 트랜젝션 기법이 WAL(Write-Ahead Logging) 기법이다. WAL은 원본 데이터베이스에 변경을 가하지 않고 모든 변경
    내역을 WAL의 페이지에 기록한다. 동일한 레코드를 수정하더라도 새로운 페이지에 기록하여 히스토리(history)를 남긴다. 특정 갯수 이상의
    페이지가 생성되면, 체크포인트(checkpoint) 이벤트를 발생하여 최신 데이터를 데이터베이스에 반영한다. 이에 동시접근 시 발생하는 데이터베이스
    락(Lock) 문제가 해결된다.\nWAL은 성능 향상을 위해 메모리에 맵핑되어 있으며, SHM(Shared Memory Map)을 통해 맵핑
    테이블을 관리한다.\n최근 임베디드 운영체제나 Mac OS X에서는 대부분의 데이터 관리를 SQLite를 사용한다. 특히 임베디드에서 사용하는
    주소록, 문자메시지 등의 데이터는 SQLite를 사용하는 것이 대부분이다. 그리고 최근에 사용되는 임베디드 운영체제에서는 이 SQLite의
    저널링에 WAL을 사용한다. 임베디드의 경우에는 동작 중인 시스템을 루팅하여 획득하거나, 백업 파일을 분석할 수 있을 것이며, 데스크탑 운영체제의
    경우에는 동작 중인 운영체제의 전원을 차단하고 디스크 이미지를 생성할 수 있을 것이다.\n<ul>\n\t<li><strong>동작 중인 운영체제의
    디스크 이미지를 생성하는 경우</strong> - 현재 동작 중인 상태의 이미지이기 때문에 SQLite 데이터베이스 파일 뿐만 아니라 WAL,
    SHM이 함께 있다. 특히 트랜잭션 로그는 전부 WAL에 기록되기 때문에 꼭 분석해야 한다. OS X의 경우에는 연락처, 이메일, 메신저와
    같은 애플리케이션이 WAL 데이터를 가진다.</li>\n\t<li><strong>백업파일을 분석하는 경우</strong> - 데이터베이스는
    연결이 종료되어 파일이 닫히거나, 1000개 이상의 페이지가 WAL 파일에 존재하면 자동으로 체크포인트가 발생되어 최신 데이터를 데이터베이스에
    저장한다. 이 경우에는 기존 SQLite 분석 방법을 그대로 사용할 수 있다. 하지만 트랜젝션 로그의 분석이 불가능하다는 문제점을 가진다.</li>\n</ul>\n<h2></h2>\n<h2>지원하는
    데이터베이스</h2>\nPostgreSQL, SQLite 3.7.0 이상, MongoDB와 같이 오픈소스 데이터베이스는 WAL를 지원한다.\n\n&nbsp;\n<h2>저널링
    사용 방법</h2>\n데이터베이스의 기본 저널은 롤백 저널이다. SQLite는 journal_mode라는 옵션으로 설정할 수 있다.\n<pre
    class=\"lang:sh decode:true \"># PRAGMA journal_mode=WAL;</pre>\n롤백 저널로 되돌릴려면
    저널 모드를 삭제하면 된다.\n<pre class=\"lang:sh decode:true\"># PRAGMA journal_mode=DELETE;</pre>\n<h2></h2>\n<h2>데이터
    로깅 방식</h2>\nWAL은 데이터베이스에 추가/삭제/수정되는 정보를 저장한다. 초기에 SQLite에 WAL 저널 모드를 설정하면 다음과
    같다.\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/2d61abb6b1cbec065008fc99315a012c.png\"><img
    class=\"aligncenter  wp-image-1078\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/2d61abb6b1cbec065008fc99315a012c.png\"
    alt=\"2d61abb6b1cbec065008fc99315a012c\" width=\"311\" height=\"221\" /></a>\n여기에서
    사용자가 페이지3에 있는 레코드의 데이터를 추가한다면 WAL의 첫 번째 페이지에 페이지3의 내용이 기록된다.\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/71733a9eaff6cb01677ed871a30ba1a1.png\"><img
    class=\"aligncenter  wp-image-1079\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/71733a9eaff6cb01677ed871a30ba1a1.png\"
    alt=\"71733a9eaff6cb01677ed871a30ba1a1\" width=\"328\" height=\"226\" /></a>\n\n그
    후 페이지2의 내용이 변경되면 WAL의 두 번째 페이지에 새로운 데이터를 기록한다.\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/d54e4aee5ffa3675d2f468ca428bf179.png\"><img
    class=\"aligncenter  wp-image-1080\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/d54e4aee5ffa3675d2f468ca428bf179.png\"
    alt=\"d54e4aee5ffa3675d2f468ca428bf179\" width=\"313\" height=\"215\" /></a>\n다시
    페이지3의 내용을 수정하면 3번째 페이지에 다시 세 번째 페이지를 추가한다. 즉, WAL에는 순차적으로 페이지의 수정된 내용을 기록(logging)한다.\n<a
    href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/cc9f134d8ed3b4d0d950e22614fa2f22.png\"><img
    class=\"aligncenter  wp-image-1081\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/cc9f134d8ed3b4d0d950e22614fa2f22.png\"
    alt=\"cc9f134d8ed3b4d0d950e22614fa2f22\" width=\"330\" height=\"221\" /></a>\n만약
    페이지가 1000개가 되거나, 애플리케이션이 체크포인트 메시지를 발생하면 가장 최근 정보를 가지는 페이지를 데이터베이스에 반영하고 모든 데이터를
    삭제한다.\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/21c3f7117118845d5d52b0defd39ceb7.png\"><img
    class=\"aligncenter  wp-image-1082\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/21c3f7117118845d5d52b0defd39ceb7.png\"
    alt=\"21c3f7117118845d5d52b0defd39ceb7\" width=\"493\" height=\"215\" /></a>\nSQLite
    데이터베이스 관리 도구는 WAL모드로 동작 중인 데이터베이스를 제대로 분석하지 못한다. 이는 데이터베이스의 상태가 Dirty이다보니 셀을 추적하는
    과정에서 문제가 발생하기 때문으로 보인다. 제대로 분석하려면 체크포인트를 통해 WAL의 데이터가 전부 데이터베이스에 이관된 후에 분석해야 한다.
    문제는 위 그림과 같이 이 과정에서 기존 데이터베이스의 페이지와 WAL에 있는 오래된 페이지의 정보가 삭제된다는 점이다. 이에 체크포인트 이전의
    데이터베이스 파일과 WAL 파일에 대한 분석을 해야 데이터베이스가 닫히기 전에 삭제된 레코드의 정보를 복구할 수 있다.\n\n&nbsp;\n<h2>파일
    포맷 분석</h2>\nWAL은 WAL 파일 헤더와 각 페이지를 프레임으로 구분하여 프레임헤더가 있는 간단한 구성을 가지고 있다. 헤더에는 WAL의
    시그너처와 버전정보 페이지 크기 등의 정보를 가지며, 프레임 헤더에는 페이지 번호와 체크섬 값 등의 정보를 가진다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_file_header.png\"><img
    class=\"aligncenter size-full wp-image-1087\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_file_header.png\"
    alt=\"wal_file_header\" width=\"211\" height=\"224\" /></a>\n\nWAL 파일은 파일 헤더를
    두고 데이터베이스에 새롭게 생성된 페이지를 프레임(frame)이라고 부른다. 트랜젝션이 일어날 때마다 변경된 페이지를 새로운 프레임으로 작성한다.
    즉 실제 데이터베이스 파일은 체크포인트가 발생할 때까지 전혀 사용되지 않는다. WAL 파일 헤더 정보는 다음과 같다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_header.png\"><img
    class=\"aligncenter size-full wp-image-1088\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_header.png\"
    alt=\"wal_header\" width=\"326\" height=\"75\" /></a>\n\nWAL 파일 고유 시그니처(0x377f0682,
    0x377f06833)가 있고 버전과 페이지 크기 등 WAL 파일 전체를 관리할 수 있는 정보가 있다.\n페이지 크기는 frame의 크기를
    말하며, frame 헤더를 뺀 실제 페이지 데이터의 크기를 말한다. SQLite DB의 페이지 크기와 동일하게 설정된다. 시퀀스 넘버(Sequence
    Number)는 체크포인트가 발생할 때마다 1씩 증가하며 0부터 시작한다. salt1, salt2는 인터넷 정보에 따르면, salt1은 WAL
    파일이 생성될 때 랜덤 값이 저장되며 매 체크포인트 때마다 1씩 증가된다고 되어 있다. salt2의 경우에는 매 체크포인트 때마다 새롭게 생성되는
    해시 값 역할을 한다.  이 정보를 이용할 수 있을 것 같아 보이지만, 실제론 체크포인트 시점에 이전 데이터가 데이터베이스로 다 밀려 들어가므로,
    분석 시점에는 최종 체크포인트 이 후의 프레임만 WAL 파일에 남게되어 크게 의미가 없어진다.. (CCL Group에서는 이전 프레임이 남는다고
    하는데 이건 좀 더 확인해봐야할 듯..) 각 프레임에 붙어있는 헤더인 WAL 프레임 헤더의 구조는 다음과 같다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_frame_header.png\"><img
    class=\"aligncenter size-full wp-image-1089\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_frame_header.png\"
    alt=\"wal_frame_header\" width=\"320\" height=\"70\" /></a>\n\n각 프레임에는 해당 프레임(페이지)가
    데이터베이스의 몇 번째 페이지에 들어가야하는지 번호를 저장하는 페이지 넘버(page number)와 체크포인트 시, 해당 프레임이 커밋되야하는지에
    대한 정보를 저장하는 'end of transaction' 필드가 있다. 만약 해당 페이지가 체크포인트 대상이 아니라면 0이 저장된다.\n\n&nbsp;\n<h2>walitean
    (https://github.com/n0fate/walitean)</h2>\n필자가 개발한 WAL 전문 분석 도구이다. WAL파일만 받아 분석하며,
    페이지에 있는 레코드 중복과 상관없이 모든 레코드를 추출하여, 같은 컬럼 셋을 가지는 레코드를 묶어 테이블 형태로 추출한다. 콘솔 전용 도구이며,
    파이썬으로 개발되었기 때문에 모든 플랫폼에서 동작 가능하다. 터미널에 출력하는 방식(raw)과 tsv(tab-seperate values)
    형식을 지원한다. 오픈소스이며 GPL2 라이센스를 가진다.\n<pre class=\"lang:sh decode:true\" title=\"walitean\">$
    python walitean.py\nCopyright by n0fate (n0fate@n0fate.com)\npython walitean.py
    [-i SQLITE WAL FILE] [-f TYPE(raw, tsv)] [-o FILENAME(if type is tsv)]\nn0fates-Mac-mini:walitean
    n0fate$</pre>\n-f 옵션으로 raw, tsv를 구분한다. tsv를 줬을 경우엔 -o로 파일 명을 지정해주면 TSV 파일을 생성한다.
    다음은 OS X의 메신저 앱의 WAL 파일 분석 결과의 일부이다.\n<pre class=\"lang:sh decode:true\">$ python
    walitean.py -i ~/Library/Messages/chat.db-wal -f raw\n[+] Output Type : Standard
    Output(Terminal)\ntext                                         int         int
    \                                                                                                                                            text
    \       text       text int                                                                   text
    int\nEE9EF301-AFD0-482F-9DE8-4CA20FBC1ADE   -81433575   -47879143                                                          ~/Library/Messages/Attachments/a3/03/EE9EF301-AFD0-482F-9DE8-4CA20FBC1ADE/IMG_4478.jpeg
    public.jpeg image/jpeg   5                                                          IMG_4478.jpeg
    None\n5BB8B7C4-0FD0-438A-B11A-ABEAB4E056F7   -81433575     2518041                                                        ~/Library/Messages/Attachments/84/04/5BB8B7C4-0FD0-438A-B11A-ABEAB4E056F7/IMG_4478-1.jpeg
    public.jpeg image/jpeg   5                                                        IMG_4478-1.jpeg
    None\nBF186DC4-74F6-4B87-AA47-56065268B7FE -1545310695 -1394315751                                                           ~/Library/Messages/Attachments/ed/13/BF186DC4-74F6-4B87-AA47-56065268B7FE/IMG_9129.png
    \ public.png  image/png   5                                                           IMG_9129.png
    None\n</pre>\n현재 raw로 출력하거나 tsv로 저장하는 경우에는 BLOB의 바이트 데이터를 깔끔하게 정리할 수 없어서 출력하지
    않도록 하였다. 데이터베이스에 저장하는 방식으로 변경 시 추가할 예정이다.\n\n앞으로 진행 예정 사항은 다음과 같다.\n- SQLiteDB를
    입력으로 받아 해당 데이터베이스의 컬럼 이름을 추출하여 새로운 데이터베이스를 생성, 거기에 WAL 로그 추출 레코드를 저장하는 기능 예정 (Blob
    데이터도 출력되도록 같이 수정)\n- 중복 레코드 제거 기능 추가 예정\n\n&nbsp;\n<h2>결론</h2>\n저널링 정보는 최근 데이터
    베이스에 수정된 정보를 기록했기 때문에 최근 사용자가 한 행위를 추적할 수 있다는 이점을 가진다. 특히, 시스템을 재부팅하지 않고 사용하는
    스마트폰의 경우에는 대부분의 정보가 WAL에 기록되어 있기 때문에 더 분석이 필요하다고 할 수 있다. 또한 사용자가 메시지를 삭제하더라도 삭제이전의
    페이지 정보가 그대로 남겨져 있기 때문에 사용자의 안티포렌식 기법에도 대응할 수 있다고 할 수 있다. 저널링 정보는 사용자의 행위 기록을 남기므로
    전체 디스크의 타임라인을 구성할 때도  유용한 아티팩트가 될 수 있다.  SQLite 데이터베이스 분석 시에도 꼭 한번씩 분석하여 소중한 정보를
    놓치지 않도록 하자. 마지막으로 그 외 관련 상용도구 정보를 남기며 글을 마친다.\n\n&nbsp;\n<h2>그외 관련 도구</h2>\nSQLite
    데이터베이스가 사용자 정보를 보관하는 애플리케이션(연락처, 이메일 등)의 유용한 정보를 보관하고 있다보니, 이미 몇가지 상용도구(epilog,
    Oxygen Forensic SQLite Viewer)가 나와있다.\n<h3>Epilog</h3>\nSQLite 전문 분석 도구이다.데이터베이스도
    복구해주며, FreePage에 있는 레코드도 복구해준다. 데이터베이스 저널은 롤백저널과 WAL을 둘 다 지원한다. 단, 도구 자체의 버그인지
    올바른 데이터베이스임에도 분석을 못해주거나, FreePage Analysis를 활성화해야지만 데이터를 올바르게 분석해주는 버그가 존재한다.
    트라이얼 버전은 제공하지 않는다. 윈도우 플랫폼만 지원한다.\n\n&nbsp;\n<h3>Forensic SQLite Viewer, Oxygen</h3>\nSQLite
    DB 분석을 해주며, “Recover deleted records\" 버튼을 누르면 WAL 데이터를 분석하여 각 테이블에 레코드를 추가한다.
    임베디드 전문 포렌식 회사답게 상당히 뛰어난 퀄리티를 보여준다. 단, 데이터를 WAL에서 추출했다는 표기가 없어서 해당 데이터가 원래 DB에
    있었는지를 확인하는 것이 불편한 점이다. 홈페이지에서 30일 트라이얼 버전을 받을 수 있으니, 한번쯤 사용해보길 권한다. 윈도우 플랫폼만 지원한다.\n<a
    href=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/DataViewerSQLiteDeletedData.png\"><img
    class=\"aligncenter  wp-image-1084\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/08/DataViewerSQLiteDeletedData.png\"
    alt=\"DataViewerSQLiteDeletedData\" width=\"723\" height=\"527\" /></a>\n\n&nbsp;\n<h2>관련
    링크</h2>\nThe Forensic Implications of SQLite's Write Ahead Log, http://www.cclgroupltd.com/the-forensic-implications-of-sqlites-write-ahead-log/,
    CCL Group Ltd"
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<h2>소개</h2>
<p>과거 데이터베이스는 갑작스런 이벤트로 인해 데이터베이스가 손상되는 것을 방지하기 위해 트랜잭션(transaction)에 저널링(Jorunaling)을 사용하였다. 데이터베이스의 저널링 기법은 롤백 저널(Rollback Journal)이라고 불리며, 데이터베이스 파일 내에 저널링 데이터 보관 영역을 두고, 필요할 때마다 원래 데이터베이스 영역에 저장(commit)하는 방법을 사용하였다. 하지만 롤백 저널은 레코드 조회와 삽입을 동시에 수행할 수가 없는 문제(Database Lock)가 있었다. 이에 최근에 사용되는 트랜젝션 기법이 WAL(Write-Ahead Logging) 기법이다. WAL은 원본 데이터베이스에 변경을 가하지 않고 모든 변경 내역을 WAL의 페이지에 기록한다. 동일한 레코드를 수정하더라도 새로운 페이지에 기록하여 히스토리(history)를 남긴다. 특정 갯수 이상의 페이지가 생성되면, 체크포인트(checkpoint) 이벤트를 발생하여 최신 데이터를 데이터베이스에 반영한다. 이에 동시접근 시 발생하는 데이터베이스 락(Lock) 문제가 해결된다.<br />
WAL은 성능 향상을 위해 메모리에 맵핑되어 있으며, SHM(Shared Memory Map)을 통해 맵핑 테이블을 관리한다.<br />
최근 임베디드 운영체제나 Mac OS X에서는 대부분의 데이터 관리를 SQLite를 사용한다. 특히 임베디드에서 사용하는 주소록, 문자메시지 등의 데이터는 SQLite를 사용하는 것이 대부분이다. 그리고 최근에 사용되는 임베디드 운영체제에서는 이 SQLite의 저널링에 WAL을 사용한다. 임베디드의 경우에는 동작 중인 시스템을 루팅하여 획득하거나, 백업 파일을 분석할 수 있을 것이며, 데스크탑 운영체제의 경우에는 동작 중인 운영체제의 전원을 차단하고 디스크 이미지를 생성할 수 있을 것이다.</p>
<ul>
<li><strong>동작 중인 운영체제의 디스크 이미지를 생성하는 경우</strong> - 현재 동작 중인 상태의 이미지이기 때문에 SQLite 데이터베이스 파일 뿐만 아니라 WAL, SHM이 함께 있다. 특히 트랜잭션 로그는 전부 WAL에 기록되기 때문에 꼭 분석해야 한다. OS X의 경우에는 연락처, 이메일, 메신저와 같은 애플리케이션이 WAL 데이터를 가진다.</li>
<li><strong>백업파일을 분석하는 경우</strong> - 데이터베이스는 연결이 종료되어 파일이 닫히거나, 1000개 이상의 페이지가 WAL 파일에 존재하면 자동으로 체크포인트가 발생되어 최신 데이터를 데이터베이스에 저장한다. 이 경우에는 기존 SQLite 분석 방법을 그대로 사용할 수 있다. 하지만 트랜젝션 로그의 분석이 불가능하다는 문제점을 가진다.</li>
</ul>
<h2></h2>
<h2>지원하는 데이터베이스</h2>
<p>PostgreSQL, SQLite 3.7.0 이상, MongoDB와 같이 오픈소스 데이터베이스는 WAL를 지원한다.</p>
<p>&nbsp;</p>
<h2>저널링 사용 방법</h2>
<p>데이터베이스의 기본 저널은 롤백 저널이다. SQLite는 journal_mode라는 옵션으로 설정할 수 있다.</p>
<pre class="lang:sh decode:true "># PRAGMA journal_mode=WAL;</pre>
<p>롤백 저널로 되돌릴려면 저널 모드를 삭제하면 된다.</p>
<pre class="lang:sh decode:true"># PRAGMA journal_mode=DELETE;</pre>
<h2></h2>
<h2>데이터 로깅 방식</h2>
<p>WAL은 데이터베이스에 추가/삭제/수정되는 정보를 저장한다. 초기에 SQLite에 WAL 저널 모드를 설정하면 다음과 같다.<br />
<a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/2d61abb6b1cbec065008fc99315a012c.png"><img class="aligncenter  wp-image-1078" src="{{ site.baseurl }}/assets/2d61abb6b1cbec065008fc99315a012c.png" alt="2d61abb6b1cbec065008fc99315a012c" width="311" height="221" /></a><br />
여기에서 사용자가 페이지3에 있는 레코드의 데이터를 추가한다면 WAL의 첫 번째 페이지에 페이지3의 내용이 기록된다.<br />
<a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/71733a9eaff6cb01677ed871a30ba1a1.png"><img class="aligncenter  wp-image-1079" src="{{ site.baseurl }}/assets/71733a9eaff6cb01677ed871a30ba1a1.png" alt="71733a9eaff6cb01677ed871a30ba1a1" width="328" height="226" /></a></p>
<p>그 후 페이지2의 내용이 변경되면 WAL의 두 번째 페이지에 새로운 데이터를 기록한다.<br />
<a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/d54e4aee5ffa3675d2f468ca428bf179.png"><img class="aligncenter  wp-image-1080" src="{{ site.baseurl }}/assets/d54e4aee5ffa3675d2f468ca428bf179.png" alt="d54e4aee5ffa3675d2f468ca428bf179" width="313" height="215" /></a><br />
다시 페이지3의 내용을 수정하면 3번째 페이지에 다시 세 번째 페이지를 추가한다. 즉, WAL에는 순차적으로 페이지의 수정된 내용을 기록(logging)한다.<br />
<a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/cc9f134d8ed3b4d0d950e22614fa2f22.png"><img class="aligncenter  wp-image-1081" src="{{ site.baseurl }}/assets/cc9f134d8ed3b4d0d950e22614fa2f22.png" alt="cc9f134d8ed3b4d0d950e22614fa2f22" width="330" height="221" /></a><br />
만약 페이지가 1000개가 되거나, 애플리케이션이 체크포인트 메시지를 발생하면 가장 최근 정보를 가지는 페이지를 데이터베이스에 반영하고 모든 데이터를 삭제한다.<br />
<a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/21c3f7117118845d5d52b0defd39ceb7.png"><img class="aligncenter  wp-image-1082" src="{{ site.baseurl }}/assets/21c3f7117118845d5d52b0defd39ceb7.png" alt="21c3f7117118845d5d52b0defd39ceb7" width="493" height="215" /></a><br />
SQLite 데이터베이스 관리 도구는 WAL모드로 동작 중인 데이터베이스를 제대로 분석하지 못한다. 이는 데이터베이스의 상태가 Dirty이다보니 셀을 추적하는 과정에서 문제가 발생하기 때문으로 보인다. 제대로 분석하려면 체크포인트를 통해 WAL의 데이터가 전부 데이터베이스에 이관된 후에 분석해야 한다. 문제는 위 그림과 같이 이 과정에서 기존 데이터베이스의 페이지와 WAL에 있는 오래된 페이지의 정보가 삭제된다는 점이다. 이에 체크포인트 이전의 데이터베이스 파일과 WAL 파일에 대한 분석을 해야 데이터베이스가 닫히기 전에 삭제된 레코드의 정보를 복구할 수 있다.</p>
<p>&nbsp;</p>
<h2>파일 포맷 분석</h2>
<p>WAL은 WAL 파일 헤더와 각 페이지를 프레임으로 구분하여 프레임헤더가 있는 간단한 구성을 가지고 있다. 헤더에는 WAL의 시그너처와 버전정보 페이지 크기 등의 정보를 가지며, 프레임 헤더에는 페이지 번호와 체크섬 값 등의 정보를 가진다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_file_header.png"><img class="aligncenter size-full wp-image-1087" src="{{ site.baseurl }}/assets/wal_file_header.png" alt="wal_file_header" width="211" height="224" /></a></p>
<p>WAL 파일은 파일 헤더를 두고 데이터베이스에 새롭게 생성된 페이지를 프레임(frame)이라고 부른다. 트랜젝션이 일어날 때마다 변경된 페이지를 새로운 프레임으로 작성한다. 즉 실제 데이터베이스 파일은 체크포인트가 발생할 때까지 전혀 사용되지 않는다. WAL 파일 헤더 정보는 다음과 같다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_header.png"><img class="aligncenter size-full wp-image-1088" src="{{ site.baseurl }}/assets/wal_header.png" alt="wal_header" width="326" height="75" /></a></p>
<p>WAL 파일 고유 시그니처(0x377f0682, 0x377f06833)가 있고 버전과 페이지 크기 등 WAL 파일 전체를 관리할 수 있는 정보가 있다.<br />
페이지 크기는 frame의 크기를 말하며, frame 헤더를 뺀 실제 페이지 데이터의 크기를 말한다. SQLite DB의 페이지 크기와 동일하게 설정된다. 시퀀스 넘버(Sequence Number)는 체크포인트가 발생할 때마다 1씩 증가하며 0부터 시작한다. salt1, salt2는 인터넷 정보에 따르면, salt1은 WAL 파일이 생성될 때 랜덤 값이 저장되며 매 체크포인트 때마다 1씩 증가된다고 되어 있다. salt2의 경우에는 매 체크포인트 때마다 새롭게 생성되는 해시 값 역할을 한다.  이 정보를 이용할 수 있을 것 같아 보이지만, 실제론 체크포인트 시점에 이전 데이터가 데이터베이스로 다 밀려 들어가므로, 분석 시점에는 최종 체크포인트 이 후의 프레임만 WAL 파일에 남게되어 크게 의미가 없어진다.. (CCL Group에서는 이전 프레임이 남는다고 하는데 이건 좀 더 확인해봐야할 듯..) 각 프레임에 붙어있는 헤더인 WAL 프레임 헤더의 구조는 다음과 같다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/wal_frame_header.png"><img class="aligncenter size-full wp-image-1089" src="{{ site.baseurl }}/assets/wal_frame_header.png" alt="wal_frame_header" width="320" height="70" /></a></p>
<p>각 프레임에는 해당 프레임(페이지)가 데이터베이스의 몇 번째 페이지에 들어가야하는지 번호를 저장하는 페이지 넘버(page number)와 체크포인트 시, 해당 프레임이 커밋되야하는지에 대한 정보를 저장하는 'end of transaction' 필드가 있다. 만약 해당 페이지가 체크포인트 대상이 아니라면 0이 저장된다.</p>
<p>&nbsp;</p>
<h2>walitean (https://github.com/n0fate/walitean)</h2>
<p>필자가 개발한 WAL 전문 분석 도구이다. WAL파일만 받아 분석하며, 페이지에 있는 레코드 중복과 상관없이 모든 레코드를 추출하여, 같은 컬럼 셋을 가지는 레코드를 묶어 테이블 형태로 추출한다. 콘솔 전용 도구이며, 파이썬으로 개발되었기 때문에 모든 플랫폼에서 동작 가능하다. 터미널에 출력하는 방식(raw)과 tsv(tab-seperate values) 형식을 지원한다. 오픈소스이며 GPL2 라이센스를 가진다.</p>
<pre class="lang:sh decode:true" title="walitean">$ python walitean.py
Copyright by n0fate (n0fate@n0fate.com)
python walitean.py [fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][-i SQLITE WAL FILE] [-f TYPE(raw, tsv)] [-o FILENAME(if type is tsv)]
n0fates-Mac-mini:walitean n0fate$</pre>
<p>-f 옵션으로 raw, tsv를 구분한다. tsv를 줬을 경우엔 -o로 파일 명을 지정해주면 TSV 파일을 생성한다. 다음은 OS X의 메신저 앱의 WAL 파일 분석 결과의 일부이다.</p>
<pre class="lang:sh decode:true">$ python walitean.py -i ~/Library/Messages/chat.db-wal -f raw
[+] Output Type : Standard Output(Terminal)
text                                         int         int                                                                                                                                             text        text       text int                                                                   text int
EE9EF301-AFD0-482F-9DE8-4CA20FBC1ADE   -81433575   -47879143                                                          ~/Library/Messages/Attachments/a3/03/EE9EF301-AFD0-482F-9DE8-4CA20FBC1ADE/IMG_4478.jpeg public.jpeg image/jpeg   5                                                          IMG_4478.jpeg None
5BB8B7C4-0FD0-438A-B11A-ABEAB4E056F7   -81433575     2518041                                                        ~/Library/Messages/Attachments/84/04/5BB8B7C4-0FD0-438A-B11A-ABEAB4E056F7/IMG_4478-1.jpeg public.jpeg image/jpeg   5                                                        IMG_4478-1.jpeg None
BF186DC4-74F6-4B87-AA47-56065268B7FE -1545310695 -1394315751                                                           ~/Library/Messages/Attachments/ed/13/BF186DC4-74F6-4B87-AA47-56065268B7FE/IMG_9129.png  public.png  image/png   5                                                           IMG_9129.png None
</pre>
<p>현재 raw로 출력하거나 tsv로 저장하는 경우에는 BLOB의 바이트 데이터를 깔끔하게 정리할 수 없어서 출력하지 않도록 하였다. 데이터베이스에 저장하는 방식으로 변경 시 추가할 예정이다.</p>
<p>앞으로 진행 예정 사항은 다음과 같다.<br />
- SQLiteDB를 입력으로 받아 해당 데이터베이스의 컬럼 이름을 추출하여 새로운 데이터베이스를 생성, 거기에 WAL 로그 추출 레코드를 저장하는 기능 예정 (Blob 데이터도 출력되도록 같이 수정)<br />
- 중복 레코드 제거 기능 추가 예정</p>
<p>&nbsp;</p>
<h2>결론</h2>
<p>저널링 정보는 최근 데이터 베이스에 수정된 정보를 기록했기 때문에 최근 사용자가 한 행위를 추적할 수 있다는 이점을 가진다. 특히, 시스템을 재부팅하지 않고 사용하는 스마트폰의 경우에는 대부분의 정보가 WAL에 기록되어 있기 때문에 더 분석이 필요하다고 할 수 있다. 또한 사용자가 메시지를 삭제하더라도 삭제이전의 페이지 정보가 그대로 남겨져 있기 때문에 사용자의 안티포렌식 기법에도 대응할 수 있다고 할 수 있다. 저널링 정보는 사용자의 행위 기록을 남기므로 전체 디스크의 타임라인을 구성할 때도  유용한 아티팩트가 될 수 있다.  SQLite 데이터베이스 분석 시에도 꼭 한번씩 분석하여 소중한 정보를 놓치지 않도록 하자. 마지막으로 그 외 관련 상용도구 정보를 남기며 글을 마친다.</p>
<p>&nbsp;</p>
<h2>그외 관련 도구</h2>
<p>SQLite 데이터베이스가 사용자 정보를 보관하는 애플리케이션(연락처, 이메일 등)의 유용한 정보를 보관하고 있다보니, 이미 몇가지 상용도구(epilog, Oxygen Forensic SQLite Viewer)가 나와있다.</p>
<h3>Epilog</h3>
<p>SQLite 전문 분석 도구이다.데이터베이스도 복구해주며, FreePage에 있는 레코드도 복구해준다. 데이터베이스 저널은 롤백저널과 WAL을 둘 다 지원한다. 단, 도구 자체의 버그인지 올바른 데이터베이스임에도 분석을 못해주거나, FreePage Analysis를 활성화해야지만 데이터를 올바르게 분석해주는 버그가 존재한다. 트라이얼 버전은 제공하지 않는다. 윈도우 플랫폼만 지원한다.</p>
<p>&nbsp;</p>
<h3>Forensic SQLite Viewer, Oxygen</h3>
<p>SQLite DB 분석을 해주며, “Recover deleted records" 버튼을 누르면 WAL 데이터를 분석하여 각 테이블에 레코드를 추가한다. 임베디드 전문 포렌식 회사답게 상당히 뛰어난 퀄리티를 보여준다. 단, 데이터를 WAL에서 추출했다는 표기가 없어서 해당 데이터가 원래 DB에 있었는지를 확인하는 것이 불편한 점이다. 홈페이지에서 30일 트라이얼 버전을 받을 수 있으니, 한번쯤 사용해보길 권한다. 윈도우 플랫폼만 지원한다.<br />
<a href="http://forensic.n0fate.com/wp-content/uploads/2014/08/DataViewerSQLiteDeletedData.png"><img class="aligncenter  wp-image-1084" src="{{ site.baseurl }}/assets/DataViewerSQLiteDeletedData.png" alt="DataViewerSQLiteDeletedData" width="723" height="527" /></a></p>
<p>&nbsp;</p>
<h2>관련 링크</h2>
<p>The Forensic Implications of SQLite's Write Ahead Log, http://www.cclgroupltd.com/the-forensic-implications-of-sqlites-write-ahead-log/, CCL Group Ltd[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
