---
layout: post
title: 메모리에서 덤프한 KEXT 분석 시 심볼 정보 맞춰주기
date: 2014-12-03 16:41:36.000000000 +09:00
type: post
published: true
status: publish
categories:
- Malware Analysis
- Memory Forensics
tags:
- IDA
- KEXT
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<h3 id="1.-서론"><a href="#1.-서론" name="1.-서론"></a>1. 서론</h3>
<p>메모리에서 덤프한 파일을 IDA와 같은 디스어셈블러로 분석할 때 가장 곤란한 점은 제거된 임포트 함수 정보이다. 임포트 함수 정보는 심볼 테이블에 정의되어 있으며, 이 테이블에는 프로그램이 임포트할 라이브러리와 함수 이름이 정의되어 있다. 프로그램은 메모리에 로드되어 프로세스화 되는 과정에서 임포트 함수의 메모리 주소를 찾는데 이 테이블을 사용한다.</p>
<p>메모리에서 추출한 KEXT를 분석할 때는 해당 파일을 직접 분석할 때와 다르게 다음 사항을 고려해야 한다.</p>
<ol>
<li>심볼 테이블 제거</li>
<li>Call opcode의 오퍼랜드를 임포트 함수의 주소로 대치</li>
<li>KASLR로 인한 커널 함수의 주소 변경</li>
</ol>
<h3 id="2.-이슈"><a href="#2.-이슈" name="2.-이슈"></a>2. 이슈</h3>
<h4 id="1.-심볼-테이블-제거"><a href="#1.-심볼-테이블-제거" name="1.-심볼-테이블-제거"></a>1. 심볼 테이블 제거</h4>
<p>KEXT와 같은 커널 영역 데이터는 함수 주소를 맞춰준 후, 심볼 정보를 제거한다. 정확하게는 불필요한 커맨드(섹션)을 제거한다. 그러다보니 덤프한 바이너리 파일에는 코드 영역에 작성된 호출한 커널 주소만 남게 된다. 심볼 정보가 모두 유실되므로, 바이너리만 보고서는 분석에 한계를 가지게 된다.</p>
<h4 id="2.-call-오퍼랜드-값-변경"><a href="#2.-call-오퍼랜드-값-변경" name="2.-call-오퍼랜드-값-변경"></a>2. call 오퍼랜드 값 변경</h4>
<p>바이너리 내 코드 영역에도 변화가 생기는데, CALL opcode 뒤의 오퍼랜드가 0x00에서 임포트 주소로 변경된다. 예를 들어 커널 함수인 OSMalloc을 호출한다면 IDA에서는 다음과 같이 보인다.</p>
<pre><code>00000624 E8 48 23 00 00 CALL _OSMalloc
</code></pre>
<p>이 정보는 디스어셈블러에서 호출하는 함수 주소를 바이너리에 있는 심볼 주소와 매핑한 것으로 실제 헥사코드로 보면 다음과 같다.</p>
<pre><code>00000620 89 C3 XX XX E8 00 00 00 00
...
00002340 .... _OSMalloc
</code></pre>
<p>KEXT가 메모리에 로드되는 시점에 오퍼랜드 정보가 변경된다. IOKit은 KEXT 로드 시점에 심볼 테이블을 확인하여 임포트하는 커널 함수 이름을 식별하고, 그 이름을 토대로 메모리 상의 커널 함수의 위치를 알아낸다. 그리고 이 위치를 call opcode의 오퍼랜드로 적용한다. 메모리에서 덤프한 커널 코드를 보면 다음과 같다.</p>
<pre><code>00000620 89 C3 XX XX E8 28 64 CC 7C
</code></pre>
<p>이 주소에 해당하는 커널 함수를 찾기<br />
위한 방법으로 커널의 심볼 테이블을 이용한다. 커널 이미지의 심볼에는 각 커널 함수의 이름과 가상 주소 정보를 가진다. 덤프한 KEXT에서 호출하는 주소와 커널 심볼 주소를 매칭함으로 올바른 커널 함수명을 알아낼 수있다. 이 작업은 OS X Lion까지는 정상적으로 적용할 수 있으나, 최신 버전에서는 적용할 수 없다. 그 이유는 KASLR 때문이다.</p>
<h4 id="3.-kaslr(kernel-address-space-layout-randomization)"><a href="#3.-kaslr(kernel-address-space-layout-randomization)" name="3.-kaslr(kernel-address-space-layout-randomization)"></a>3. KASLR(Kernel address space layout randomization)</h4>
<p>마운틴 라이언 이상(10.8)의 운영체제에는 KASLR 기술이 적용되어, 부팅할 때마다 커널의 베이스 주소가 변경된다. 베이스 주소의 변경은 주요 함수의 엔트리 포인트 변경으로 이어져서 재부팅할 때마다 KEXT가 임포트하는 커널 함수의 주소가 변경된다. 결국 커널의 베이스 주소에 맞춰 심볼 주소를 재정의하지 않으면, 덤프한 KEXT 분석 시 함수 식별이 힘들어진다.</p>
<h3 id="3.-해결방안"><a href="#3.-해결방안" name="3.-해결방안"></a>3. 해결방안</h3>
<p>이 문제를 해결하기 위해 커널 이미지에 있는 심볼 테이블 정보를 획득하고, 심볼 테이블에 있는 각 심볼 레코드의 주소 값에 KASLR로 변경된 커널 베이스 주소를 적용하여 실질적인 커널 가상 주소를 산출한다.</p>
<pre><code>for symrec in symtable:
    print '0x%.8x'%(symrec-&gt;vaddr + kernelbaseaddr)
</code></pre>
<p>그리고, 디스어셈블러의 스크립트를 이용하여 call opcode의 operand 값에 맞는 심볼 정도를 코멘트 형태로 기입할 수 있다. volafox에 플러그인으로 개발한 모듈은 <a href="https://code.google.com/p/volafox/source/browse/trunk/volafox/plugins/export_table_symbol.py">dumpsym 명령어</a><br />
로, KASLR로 변경된 커널 심볼 주소를 정리하여 json의 덤프 기능으로 파일로 떨구는 기능을 제공한다. 이 때 키를 address로 하고, 값을 심볼 명으로 하여, 주소 기반의 매칭을 수행할 수 있도록 재정의하였다.</p>
<p>IDA에서는 idapython 스크립트를 구현하였다. <a href="https://github.com/n0fate/idapython">makecommsyscallref.py</a> 스크립트는 바이너리의 함수 영역을 검색하여 call opcode 뒤에 "near ptr"이 붙어 있을 경우 그 뒤의 주소 값을 파싱하여 덤프한 심볼 주소 정보와 일치하는 심볼을 찾는다.</p>
<pre><code>import json
import idc

'''
License : GPLv2
Author : n0fate (n0fate@n0fate.com), forensic.n0fate.com
Name : makecommsyscallref.py
Requirements : This script need to open symbol file. The symbol file is output of volafox 'dumpsym' option. volafox : code.google.com/p/volafox
Site : github.com/n0fate/idascript
It can comment a kernel symbol name considered as KASLR to callee address if symbol isn't identified for analyzing the KEXT dumped in memory
Dumped symbol template of volafox : dictionary {address(hex):name(string), address(hex):name(string), ...}
'''

FILENAME = AskFile(0, '*.*', 'open symbol file')

d2 = json.load(open(FILENAME))

for f in Functions():
    func = idaapi.get_func(f)
    for head in Heads(func.startEA,func.endEA):
        if GetMnem(head) == "call":
            if(GetOpnd(head,0).startswith('near ptr')):
                try:
                    funcname = str(d2[hex(GetOperandValue(head,0))])[1:]    # remove a prefix('_')
                    print '%s: 0x%.8X (click here:%x)'%(funcname, GetOperandValue(head,0), head)
                    idc.MakeComm(head, funcname)
                except KeyError:
                    print 'Could not find function: 0x%.8X (click here:%x)'%(GetOperandValue(head,0), head)
</code></pre>
<p>함수 코멘트 시에는 심볼의 prefix인 "_"를 제거하도록 하였다. IDA에서 이 스크립트를 실행하고 덤프한 심볼 정보를 적용하면 된다.</p>
<h3 id="4.-한계점"><a href="#4.-한계점" name="4.-한계점"></a>4. 한계점</h3>
<p>앞의 해결방안을 통해 덤프한 KEXT를 효과적으로 분석이 가능하다. 단, 이 방법의 경우, 커널이 아닌 다른 KEXT에 있는 함수를 호출하는 경우에는 추적할 수 없는 문제점을 가지고 있다. 이 문제를 해결하려면 모든 KEXT의 익스포트 심볼을 분석해야하는데, 시도는 가능하지만, 도구의 분석 시간이 많이 소모될 수 있다.</p>
