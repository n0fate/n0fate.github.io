---
layout: post
title: 'DFIRCON 2014 : Memory Forensics (Part II) - irykmmww.dll'
date: 2014-02-10 21:12:50.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- Memory Forensics
tags:
- malware analysis
- memory forensics
- rootkit
- windows
author: "n0fate"
---
<p>이번 글부터 메모리 분석 과정에서 악성코드로 추정되는 3개의 파일을 분석하기로 한다. Part I에서 여러 개의 바이너리를 덤프했지만 동일한 DLL이 다른 프로세스에 인젝션되어 있었으며, 결국 3개의 악성코드(DLL, D1L, SYS)로 분류된다. 여기에서는 가장 많은 프로세스에 인젝션(explorer.exe 및 그 하위 프로세스)된 irykmmww.dll 을 분석해보고자 한다. 분석에는 가장 코드 복구가 잘 된 "explorer.exe(PID: 1672)"에 인젝션된 바이너리로 선정하였다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/irykmmww.dll_.jpg"><img class="aligncenter size-full wp-image-811" alt="irykmmww.dll" src="{{ site.baseurl }}/assets/irykmmww.dll_.jpg" width="1024" height="768" /></a></p>
<p>한가지 유의할 점은 여기서 덤프한 바이너리는 모두 메모리에서 덤프한 것이다. 즉, 일부 코드가 유실되거나 변경되어 있을 수 있으며, 내부 호출 함수의 심볼 정보도 대부분 유실되어 있다. 그렇다보니 추측으로 알아내야하는 데이터도 존재할 수 있다.</p>
<h2>1. 주요 기능</h2>
<p>이 DLL은 현재 D1L에 의해 인젝션 되는 것으로 추정되며, 크게 다음 기능을 가진다.</p>
<ol>
<li>다른 프로세스가 DLL에 있는 Export 함수 호출 시 전역 키보드 후킹</li>
<li>Internet Explorer 정보유출</li>
<li>Office OutLook, FoxMail, Outlook Express 계정 정보 유출</li>
<li>MSN, QQ, ICQ 메시지 유출</li>
</ol>
<p>이 DLL은 자동 실행 기능을 보유하고 있진 않으며, 클라이언트에 존재하는 정보를 유출하는 가장 중요한 임무를 가진다. 익스포트 함수 호출을 제외한 모든 후킹은 스레드 형태로 동작한다.</p>
<h2>2. 다른 프로세스가 DLL에 있는 Export 함수 호출 시 전역 키보드 후킹</h2>
<p>D1L이 익스포트 함수를 호출하면,  OpenWindowStation API로 윈도우 스테이션을 열고, OpenDesktop으로 현재 데스크탑을  오픈한 후, SetWindowsHookEx로 키보드 후킹을 수행한다.</p>
<pre class="lang:c++ decode:true" title="SK - Export Function">int __cdecl SK(LPCSTR lpString2)
{
  DWORD v1; // eax@1
  HWINSTA v2; // eax@1
  HDESK v3; // eax@3

  byte_9775C8 = 2;
  GetProcessWindowStation();
  v1 = GetCurrentThreadId();
  GetThreadDesktop(v1);
  byte_9775C8 = 30;
  v2 = OpenWindowStationA("winsta0", 0, 895u);
  if ( v2 )
  {
    byte_9775C8 += 42;
    SetProcessWindowStation(v2);
  }
  v3 = OpenDesktopA("Default", 1u, 0, 0x400001CFu);
  if ( v3 )
    SetThreadDesktop(v3);
  byte_9775C8 *= 32;
  lstrcpyA("C:DOCUME~1demoLOCALS~1Tempirykmmww.log", lpString2);
  if ( !dword_97A504 )
    dword_97A504 = SetWindowsHookExA(2, KeyboardMessageHook, hmod, 0);
  if ( !hhk )
    hhk = SetWindowsHookExA(4, ImmKeyboardHook, hmod, 0);
  return SetWindowHooks();
}</pre>
<p>보통 SetWindowsHookEx만으로 후킹을 거는 것에 비하면 앞에 여러 API를 호출하는 것을 볼 수 있다. SetWindowsHookEx API는 Thread ID를 필요로하는데, 보통 악성코드는 FindWindow로 이 ID를 획득한다. 이 API는 애플리케이션이 아닌 윈도우 서비스에서 실행된다면,  서비스는 윈도우 스테이션과 데스크탑이 정의되지 않았기 때문에 실패를 리턴하게 된다. 이 악성코드는 이러한 상황을 고려하여 서비스에서도 동작이 가능한 전역 후킹을 설정하였다. 다음은 위 코드를 실제 코드화한 것이다.</p>
<pre class="lang:default decode:true" title="SetWindowsHookEX() as a Windows service">// reference
// http://cboard.cprogramming.com/windows-programming/144588-%5Bwin7%5D-setwindowshookex-windows-service-setthreaddesktop.html
//
HHOOK hHook = NULL;
HHOOK ImmHook = NULL;
HHOOK IMEHook = NULL;
HINSTANCE hmod = hinstDLL;

int SetWindowsHooks()
{
    if(IMEHook == NULL)
        IMEHook = SetWindowsHookEx(WH_GETMESSAGE, KeyboardHook_IME, hmod, NULL);
    return -1;
}

int SK(LPCSTR lpString2)
{
    HWINSTA hWS;
    HDESK hDT;

    // Connect to the window station
    hWS = OpenWindowStation("Winsta0", FALSE, MAXIMUM_ALLOWED);
    if(hWS)
    {
        SetProcessWindowStation(hWS);
     }
    //Connect to the desktop
    hDT = OpenDesktop("Default", 1, FALSE, MAXIMUM_ALLOWED);
    if(hDT)
        SetThreadDesktop (hDT);
    strcpy(filename, lpString2); // logfilename
    if(hHook == NULL) // SK가 처음 실행된 경우..
        hHook = SetWindowsHookEx(WH_KEYBOARD, KeyboardMessageHook, hmod, NULL);
    if(ImmHook == NULL) // SK가 처음 실행된 경우..
        ImmHook = SetWindowsHookEx(WH_CALLWNDPROC, ImmKeyboardHook, hmod, NULL);
    return SetWindowHooks();
}</pre>
<p><span style="line-height: 1.5em;">악성코드는 사용자가 키입력을 할 때 발생할 때와 시스템에서 윈도우 프로시저로 메시지를 전달하기 전에 후킹 프로시저를 등록하여 키보드 입력을 가로채도록 하였다. 그리고 마지막에 SetWindowHooks 함수를 호출하여 메시지큐에 메시지를 가져오는 시점을 후킹하였다.</span></p>
<h3>- WH_KEYBOARD 후킹</h3>
<p>일반적인 키보드 후킹을 수행한다. wParam의 파라미터를 체크하여 일반적인 키 입력(1~0 or atoz 등)이면 GetKeyboardState() -&gt; ToAscii()로 문자를 추출한 후, irykmmww.log 파일에 로그를 남긴다.  만약에 일반적인 키 입력 아닌 경우, wParam 값에 들어있는 메시지에 해당하는 기능 키(화살표, 스페이스바, 엔터 등)로 변환하여 로그를 남긴다. 각 wParam 메시지는 다음과 같이 변환된다.</p>
<ul>
<li>0x20 -&gt; Space bar (" ")</li>
<li>0x8 -&gt; Back-space ("[fusion_builder_container hundred_percent="yes" overflow="visible"][fusion_builder_row][fusion_builder_column type="1_1" background_position="left top" background_color="" border_size="" border_color="" border_style="solid" spacing="yes" background_image="" background_repeat="no-repeat" padding="" margin_top="0px" margin_bottom="0px" class="" id="" animation_type="" animation_speed="0.3" animation_direction="left" hide_on_mobile="no" center_content="no" min_height="none"][&lt;=]")</li>
<li>0xD -&gt; CarriageReturn-LineFeed ("rn")</li>
<li>0x28 -&gt; Under Arrow ("[V]")</li>
<li>0x26 -&gt; Upper Arrow ("[^]")</li>
<li>0x25 -&gt; Left Arrow ("[&lt;-]")</li>
<li>0x27 -&gt; Right Arrow ("[-&gt;]")</li>
</ul>
<p>이 외의 메시지는 CallNextHookEx로 원래 프로시저에게 처리되도록 한다.</p>
<h3>- WH_CALLWNDPROC 후킹</h3>
<p>lParam의 메시지가 IME 변경 입력인 경우, 즉 WM_IME_COMPOSITION 메시지인 경우에 대해 키보드 후킹을 진행한다.  예를 들어 하나의 한글의 경우 한글의 자음과 모음이 입력될 때마다 메시지가 발생된다. 이 프로시저는 이 메시지를 가로채서 입력된 키를 획득한다.</p>

```c
ImmKeyboardHook(int code, WPARAM wParam, LPARAM lParam)
{
    if((MSG*)lParam-&gt;message == WM_IME_COMPOSITION)
    {
        hWnd = GetFocus();
        hImg = ImmGetContext(hWnd);
        strLen = ImmGetCompositionString(hImg, GCS_RESULTSTR, NULL, NULL);
        ImmGetCompositionString(hImg, GCS_RESULTSTR, String1, strLen+1);
        if(IsDBCSLeadByte(char((MSG*)lParam-&gt;wParam)))
        {
            WriteLog(String1); // KeyLogging
        }
        if(hImg)
            ImmReleaseContext(hWnd, hImg);
    }
    return CallNextHookEx();
}
```

<p>&nbsp;</p>
<h3>-  WH_GETMESSAGE 후킹</h3>
<p>메시지를 얻어오는 레벨에서 후킹을 걸기위해 존재한다. WM_IMG_COMPOSITION 메시지가 아니거나 정상적인 코드가 아니면 CallNextHookEx로 다음 후킹 프로시저로 전달한다. 그 외에는 WH_CALLPROC의 경우와 같이 IME 문자를 가져온 다음, 로깅한다.</p>
<p>&nbsp;</p>
<h2>3. Internet Explorer 정보 유출</h2>
<p>DLL이 인젝션된 곳이 인터넷 익스플로러라면, COM 객체를 생성시키고,  인터넷 익스플로러가 접근하는 사이트에서 사용자가 입력한 ID Password 필드를 유출하는 것으로 판단된다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/startaddress.png"><img class="aligncenter size-full wp-image-797" alt="startaddress" src="{{ site.baseurl }}/assets/startaddress.png" width="739" height="319" /></a></p>
<p>모든 정보는 irykmmww.log 에 기록하며, 다음과 같은 포맷으로 기록한다.</p>
<blockquote><p>Time : YEAR-MONTH-DAY HOUR:MINUTE:SECOND</p>
<p>Name: Internet Explorer</p>
<p>痰빵츰샀角페儉瓊슥돨코휭: [ID] // TEXT 필드의 값, 중국어라 올바르게 해석이 안되네요.</p>
<p>PassWD: [PASSWORD]</p></blockquote>
<p>&nbsp;</p>
<h2>4. Office OutLook, FoxMail, Outlook Express 계정 정보 유출</h2>
<p>DLL이 인젝션된 프로세스가 오피스 아웃룩이나 FoxMail, 아웃룩 익스프레스라면, 레지스트리의 Protected Storage에 있는 정보를 유출한다. 이를 위해 psapi.dll에 있는 "PStoreCreateInstance" API를 사용한다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/EnumPStorage.png"><img class="aligncenter size-full wp-image-798" alt="EnumPStorage" src="{{ site.baseurl }}/assets/EnumPStorage.png" width="692" height="241" /></a></p>
<p>각 요소에서 유출하는 정보는 다음과 같다.</p>
<ul>
<li>Office Outlook : HTTPMail User Name and Password / POP3 User Name, Server and Password</li>
<li>FoxMail : POP3 User Name, Server and Password</li>
<li>Outlook Express :POP3 User Name, Server and Password</li>
<li>IE Password-Protected Sites : ID and Auto-Complete Password</li>
<li>MSN Explorer : MSN ID and Password</li>
</ul>
<p>Protected Storage 코드를 분석해보면 인터넷에 있는 특정 사이트에 있는 소스코드를 그대로 복사해 온 것으로 보인다.  저 위의 데이터는 생각보다 많은 사용자가 윈도우 시스템에 저장하여 사용하기 때문에 악용이 가능한바, 별도의 코드나 사이트 주소는 올리지 않도록 하겠다.</p>
<h2>5.  MSN 메신저, QQ, ICQ 메시지 유출</h2>
<p>이 악성코드는  주요 메신저에서 사용자가 입력하는 메시지를 가로챈다. 대상은 MSN 메신저(msmsgs.exe), QQ(QQ.exe), ICQ(icq.exe, icqlite.exe)이다.</p>
<p><a href="http://forensic.n0fate.com/wp-content/uploads/2014/02/ApplicationDataLeak.png"><img class="aligncenter size-full wp-image-799" alt="ApplicationDataLeak" src="{{ site.baseurl }}/assets/ApplicationDataLeak.png" width="1081" height="465" /></a></p>
<p>유출 방법은 다음과 같다.</p>
<ul>
<li>MSN Messenger : GetWindowLong API에 GWL_ID(윈도우 식별자 획득)로 윈도우 ID를 얻어온 다음, 식별자가 1003, 1005이면, SendMessage에 WM_GETTEXT 메시지를 전달하여 텍스트 메시지를 가져온다. 식별자 번호는 공격자가 직접 MSN 메신저를 분석하여 얻은 채팅 또는 계정 입력 부분과 관련된 윈도우 ID로 판단된다.</li>
<li>QQ :  GetClassName API로 클래스 명 얻어온다음 그 클래스가 EditBox일 경우, GetWindowLong에 GWL_STYLE(윈도우 스타일)로 윈도우 스타일을 얻어온다. 스타일이 SWP_FRAMECHANGED가 아니라면, SendMessage에 WM_GETTEXT 메시지를 전달하여 텍스트 메시지를 가져온다. 식별자 번호는 공격자가 직접 QQ를 분석하여 얻은 채팅 또는 계정 입력 부분과 관련된 윈도우 ID로 판단된다.</li>
<li>ICQ : GetWindowLong API에 GWL_ID(윈도우 식별자 획득)로 윈도우 ID를 얻어온 다음, 식별자가 1001, 2109, 3010이면, SendMessage에 WM_GETTEXT 메시지를 전달하여 텍스트 메시지를 가져온다. 식별자 번호는 공격자가 직접 ICQ를 분석하여 얻은 채팅 또는 계정 입력 부분과 관련된 윈도우 ID로 판단된다.</li>
</ul>
<h2>6. 마치며..</h2>
<p>지금까지 irykmmww.dll 분석을 알아보았다. 이 DLL은 개인 정보 유출에 중점을 둔 모듈로 자기자신이 디바이스 드라이버를 로드하거나 D1L 파일을 인젝션시키는 기능은 제공하지 않는다. 즉, 메모리만 보면 해당 악성코드는 최초 인젝션된 악성 DLL이 아니라는 결론을 내릴 수 있다. 글의 서두에서 언급했지만 다른 악성 파일인 irykmmww.d1l은 오히려 이 DLL을 호출(익스포트된 SK 함수)하고 있었다. Part III에서는 irykmmww.d1l을 알아보고, 이 두 DLL 간의 연관관계를 알아보도록 하겠다.
