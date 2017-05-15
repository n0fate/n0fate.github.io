--- layout: post title: Mac OS X Artifact (DS_Store) date: 2013-12-08 12:55:00.000000000 +09:00 type: post published: true status: publish categories: - OS Artifacts tags: - artifacts - ds_store author: login: n0fate email: rapfer@gmail.com display_name: n0fate first_name: '' last_name: '' ---

Mac OS X를 많이 사용하다보면, 일반 사용자의 눈에는 보이진 않지만 사용자 폴더에 숨겨져 있는 파일이 하나 있다. 바로 ".DS_Store"라는 파일인데, Mac OS X를 사용한다면 터미널을 열어서 'ls -al' 명령으로 간단하게 확인할 수 있다.

```bash
n0fate@n0fate-MacBook-Air: ~$ ls -al
total 20462328
drwxr-xr-x@  77 n0fate  staff         2618 Dec  7 19:02 .
drwxr-xr-x    6 root    admin          204 Oct 23 20:08 ..
-rw-------    1 n0fate  staff            3 Nov  4 22:20 .CFUserTextEncoding
-rw-r--r--@   1 n0fate  staff        24580 Dec  7 17:28 .DS_Store
drwx------    8 n0fate  staff          272 Dec  7 18:46 .Trash
-rw-------    1 n0fate  staff         9401 Dec  7 17:34 .bash_history
-rw-r--r--    1 n0fate  staff          924 Nov  2 18:14 .bash_profile
....
```

오늘은 매일 보기는 했지만 아무 생각없이 넘겨버렸던 파일의 역할이 무엇인지 알아보고 여기서 얻을만한 포렌식 정보가 무엇인지 알아보도록 한다.

## 1\. DS_Store?

'DS_Store'를 분석하기 전에 도대체 이 파일이 뭔지부터 알아봐야 한다. 다행히도 위키에 이 파일에 대한 정의가 잘 작성되어 있다. 위키의 정의에 따르면,  이 파일은 Desktop Services Store(DS_Store)로 애플에서 정의한   파일 포맷이다. 숨김 파일로 존재하며, 파인더(Finder)에서 필요한 폴더의 속성 정보(아이콘의 위치, 배경 이미지 등)을 저장한다[[1]](http://en.wikipedia.org/wiki/.DS_Store). 즉, 이 파일은 폴더에서 해당 파일이 어떻게 보여질지에 대한 메타데이터 정보를 가지는 것이 주요 목적이다. 또한 HFS+ 파일 시스템에선 저장하지 못하는 데이터이지만 파인더에서 필요한 경우에도 이 파일에 데이터를 저장한다. 대표적인 것이 스팟라이트 코멘트(Spotlight Comments)이다.

'DS_Store'는 사용자가 접근하는 모든 폴더에 해당 파일을 생성한다. 하물며, SMB나 AFP(Apple File Protocol)로 접근하는 경우에도 해당 폴더에 파일을 생성한다. 이제 전체적인 파일의 구조를 알아보고 분석 방법과 포렌식에서 유용한 정보를 알아본다.

## 2\. DS_Store 파일 포맷

사실 DS_Store의 구조는 어느정도 알려져 있다. 최초에 Mozilla Wiki에 Mark Mentovai가 기본적인 구조가 서술되어 있다 [[2]](https://wiki.mozilla.org/DS_Store_File_Format). 그리고 해당 파일을 좀 더 분석하여 Wim Lewis가 파일 포맷과 함께 관련 펄(Perl) 라이브러리를 CPAN에 등록하였다 [[3]](http://search.cpan.org/~wiml/Mac-Finder-DSStore/DSStoreFormat.pod). 사실상 DS_Store 구조를 분석하기 위한 데이터는 이미 준비되었다고 보면 된다. DS_Store의 구조를 간략하게 살펴보면 그림 1과 같다.

[![Screen Shot 2013-12-08 at 10.50.12]({{ site.baseurl }}/assets/Screen-Shot-2013-12-08-at-10.50.12.png)](http://forensic.n0fate.com/wp-content/uploads/2013/12/Screen-Shot-2013-12-08-at-10.50.12.png)

**Figure 1\. DS_Store Structure **

 DS_Store 파일은 B-Tree(Binary Tree) 구조로 데이터를 유지한다. 각각의 레코드를 노드로 관리하고 있으며 Root, non-leaf, Leaf 노드로 나눠서 관리한다. 각 레코드에는 UTF-16포맷의 파일/폴더 명과 저장할 구조체 정보와 데이터 타입이 저장된다.

Structure ID는 해당 레코드가 어떠한 구조체가 저장되는지에 대한 정보를 가진다. 해당 폴더에 배경을 저장하거나, 아이콘 설정 또는 아이콘의 위치 정보, 해당 폴더를 열었을 때의 파인더 위치, View(List, Detail 등) 정보 등이 있다. 이러한 정보 외에도 논리 및 물리적 파일 크기, 스팟라이트의 코멘트(Spotlight Comments)가 있다. 각각의 정보는 4개의 문자로 이루어진 'FourCharCode'로 구성된다. 예를 들면, 배경 설정과 관련된 데이터가 저장되어 있다면 'BKGD'라는 4개의 문자로 표현한다. 현재까지 알려진 구조체 종류는 다음과 같다.

*   BKGD : 12바이트 blob으로 해당 디렉터리에 보여진 배경에 대한 정보를 가진다.
*   Iloc : 파일 아이콘의 위치 정보를 가진다.
*   bwsp : 바이너리 plist 정보를 가진다. plist에는 사이드바의 넓이, 사이드바/툴바/상태 바 출력 여부 등의 정보를 가진다.
*   cmmt : 스팟라인트 코멘트 정보를 저장한다.
*   icvo : 아이콘 미리보기와 관련된 설정을 저장한다.
*   icvp : 아이콘 출력과 관련된 plist를 저장한다. 여기에는 아이콘 미리보기 활성화 여부, 아이콘의 위치, 크기 정보 등을 저장한다.
*   icvt : 아이콘 출력 시 보여지는 텍스트 크기를 정의한다.
*   logS, lg1S : 해당 디렉터리의 논리적 크기를 저장한다.
*   phyS, ph1S : 해당 디렉터리의 물리적 크기를 저장한다.
*   modD, moDD : 타임스탬프 정보를 저장한다. 수정시간이 저장된다.

DataType은 데이터 저장 타입을 의미한다. 이 데이터도 4바이트로 표현되며 long, shor, blob, ustr(Unicode String) 등으로 표현된다.

## 3\. 포렌식적으로 유용한 데이터 선별

사실 DS_Store는 포렌식적으로 유용한 데이터가 많지는 않다. 대부분이 파인더에서 출력에 필요한 정보이고 어떤 사건사고와 관련될만한 정보로 보기에는  부족하다. 그리고 해당 파일이 삭제되거나 추가되는 것이 즉각적으로 DS_Store에 반영되기 때문에, 어떠한 파일이 삭제되었을 때 추가분석에 활용하기에도 제한이 따른다. 그래도 이 중에 도움이 될만한 정보를 꼽아보자면 스팟라이트 코멘트 정보를 저장하는 'cmmt'와, 파일의 수정 시간이 기록된 modD or moDD, 파일의 논리적/물리적 크기를 저장하는 logS or phyS를 들 수 있다.

스팟라이트 코멘트는 파인더에서 특정 파일이나 폴더의 정보 확인('Get Info')에서 내용을 추가/수정/삭제할 수 있으며, 검색 시 사용자가 자유롭게 텍스트 메시지를 입력할 수 있어서 추 후 검색 시 hit rate를 높이는데 활용한다.

[![Screen Shot 2013-12-08 at 11.40.30]({{ site.baseurl }}/assets/Screen-Shot-2013-12-08-at-11.40.30.png)](http://forensic.n0fate.com/wp-content/uploads/2013/12/Screen-Shot-2013-12-08-at-11.40.30.png)

** Figure 2\. 'Get Info' 데이터의 일부**

이 정보는 디스크 어느 곳에도 남지 않고 DS_Store에만 존재하는 사용자 흔적이기 때문에 포렌식적으로 유용하게 사용될 수 있는 정보이다.

파일의 수정시간이 기록되는 modD or moDD는 해당 정보가 DS_Store에 저장된 후에 사용자가 고의적으로 파일의 생성/수정 시간 등을 조작했을 경우 조작여부를 판별할 수 있는 정보가 될 수 있다. 최근에 공격자들이 자주 사용하는 안티 포렌식 기법 중 하나로 시간 조작(timestamp manipulation)이 있다보니 특히 주목할 만한 데이터가 된다. 시간정보는 HFS+ Big-Endian 과 같은 기준을 따르며, 1초에 대한 interval까지 고려되기 때문에 1초*65536 의 값이 저장된다.

디렉터리의 논리적/물리적 크기 정보는 디스크 포렌식과 함께 부가적인 정보로 활용 가능할 것이다.

또한, SMB를 통해 윈도우와 Mac OS X가 파일을 공유하는 경우 Mac OS X에서 접근하여 DS_Store 파일이 생성된 후에 윈도우 시스템에서 공유 디렉터리의 특정 파일을 삭제한다면, DS_Store에는 해당 정보가 남기 때문에 Mac OS X과 윈도우를 혼용하는 기업에서 SMB와 같은 파일 공유 서버를 분석하는 상황이 발생할 경우, 매우 유용한 아티펙트로 활용할 수 있다.

## 4\. DS_Store 분석 방법

DS_Store 데이터 분석을 위한 가장 좋은 방법은 Perl 라이브러리를 활용하는 것이다. CPAN에 이와 관련 라이브러리가 존재한다. Mac OS X 사용자라면 다음과 같은 절차를 통해 설치할 수 있다.

```
perl -MCPAN -e shell # cpan shell을 실행한다.

install Mac::PropertyList # DS_Store에 저장된 바이너리 Plist를 분석하기 위한 모듈을 설치한다.

install Mac::Finder::DSStore # DS_Store 분석 모듈을 설치한다.
```

모듈 설치가 완료되면, DS_Store 분석 모듈을 다운로드해서 예제 코드를 실행해볼 수 있다.

```bash
n0fate@n0fate-MacBook-Air: ~$ wget http://search.cpan.org/CPAN/authors/id/W/WI/WIML/Mac-Finder-DSStore-1.00.tar.gz
n0fate@n0fate-MacBook-Air: ~$ tar xzf Mac-Finder-DSStore-1.00.tar.gz  
```
DS_Store에는 총 3개의 예제 파일이 있으며, 이 중 DS_Store를 perl 스크립트 구조로 덤프해주는 'dsstore_dump.pl'로 파일의 내용을 확인하겠다.

```bash
n0fate@n0fate-MacBook-Air: ~$ cd Mac-Finder-DSStore-1.00

n0fate@n0fate-MacBook-Air: ~/Mac-Finder-DSStore-1.00$ cd examples/

n0fate@n0fate-MacBook-Air: ~/Mac-Finder-DSStore-1.00/examples$ perl dsstore_dump.pl 

Usage: dsstore_dump.pl [options] /path/to/.DS_Store > result.pl

--raw-aliases Do not interpret alias data

--resolve-aliases Resolve aliases to filenames using Mac OS calls

--parse-aliases Parse aliases using Mac::Alias::Parse

--raw-plists Do not interpret bplist data

--parse-plists Parse bplists using Mac::PropertyList

 

The default alias handling is --resolve-aliases, but this requires

the Mac::Files module to be installed.

n0fate@n0fate-MacBook-Air: ~/Mac-Finder-DSStore-1.00/examples$

n0fate@n0fate-MacBook-Air: ~/Mac-Finder-DSStore-1.00/examples$ perl dsstore_dump.pl --parse-plists ~/.DS_Store 

#!/usr/bin/perl -w

 

use Mac::Finder::DSStore qw( writeDSDBEntries makeEntries );

use Mac::PropertyList::WriteBinary qw ( );

&writeDSDBEntries("/Users/n0fate/.DS_Store",

    &makeEntries(".",

        bwsp => Mac::PropertyList::WriteBinary::as_string(bless( {

         "ShowSidebar" => bless( do{(my $o = "true")}, 'Mac::PropertyList::true' ),

         "ShowStatusBar" => bless( do{(my $o = "false")}, 'Mac::PropertyList::false' ),

         "ShowTabView" => $VAR1->{"ShowStatusBar"},

....[SNIP]...

    &makeEntries("Google x{1103}x{1173}x{1105}x{1161}x{110b}x{1175}x{1107}x{1173}",

        bwsp => Mac::PropertyList::WriteBinary::as_string(bless( {

         "ShowSidebar" => bless( do{(my $o = "true")}, 'Mac::PropertyList::true' ),

         "ShowStatusBar" => bless( do{(my $o = "false")}, 'Mac::PropertyList::false' ),

         "ShowTabView" => $VAR1->{"ShowStatusBar"},

         "WindowBounds" => bless( do{(my $o = "{{658, 205}, {770, 438}}")}, 'Mac::PropertyList::string' ),

         "SidebarWidth" => bless( do{(my $o = 145)}, 'Mac::PropertyList::integer' ),

         "ShowToolbar" => $VAR1->{"ShowSidebar"},

         "ShowPathbar" => $VAR1->{"ShowStatusBar"},

         "ContainerShowSidebar" => $VAR1->{"ShowSidebar"}

       }, 'Mac::PropertyList::dict' )),

        lg1S => 113674640,

        moDD => "226647880368128",

        modD => "226647880368128",

        ph1S => 113762304,

        vSrn => 1,

        vstl => "clmv"

...
```

위와 같이 파일이 정상적으로 분석되었으며, Plist까지 올바르게 출력하는 모습을 볼 수 있다. 단, 아직 분석가가 보기엔 깔끔한 구조가 아니다. 시간정보는 추출한 시간 값을 1초에 대한 interval인 65536으로 나눈 후에 HFS+ 파일시스템의 시간 정보와 동일하게 분석한다. 예를 들어 위 코드에서 'Google 드라이브' 폴더의 경우에는 다음과 같이 계산된다.

```
moDD => "226647880368128"

226647880368128 / 65536 => 3458372198 => 0xCE229266h
```

해당 시간을 dorumug의 time_maker.py로 변환하면 다음과 같다 [[4]](http://dorumugs.blogspot.kr/2013/02/tools-time-maker.html).

```
n0fate@n0fate-MacBook-Air: ~/Downloads$ python time_maker.py -d hfs32_big_h -i 0xCE229266

-----------------------------------------------------------------

  1 o' clock is 3600 seconds

  1 day is 86400 seconds

  1 year is 8760 hours

  1 year is 31536000 seconds

  1 second is 1000000000 nano seconds

  1 second is 1000000 micro seconds

  1 second is 1000 milli seconds

-----------------------------------------------------------------

                User Input Time -  0xCE229266

         User Input Time Format -  hfs32_big_h

            Decode Inputed Time -  2013-08-03 10:56:38

-----------------------------------------------------------------

n0fate@n0fate-MacBook-Air: ~/Downloads$
```

정상적으로 시간 정보를 확인할 수 있다.

## 5\. 결론

DS_Store는 파인더가 접근하는 모든 디렉터리에 생성되는 숨겨진 파일이다. 이 파일에는 파인더에서 필요한 다양한 정보를 저장하고 있으며, 포렌식적으로 유용한 몇몇 정보도 있기 때문에 분석에 유용하게 활용될 수 있다. 단, 파일/디렉터리의 정보가 변경될 때 바로 DS_Store에서 해당 파일 정보가 삭제되거나 추가되기 때문에 사실 디스크 포렌식을 하는 입장에서 크게 유용하진 않을 수도 있다. 하지만, 디스크 포렌식과 함께 교차분석을 수행하거나 DS_Store에만 저장되는 스팟라이트 코멘트 정보를 통해 유용한 정보를 획득할 수도 있다.

원래 디지털 포렌식이라는 분야는 이런 작은 요소에서부터 예상치 못한 증거를 획득할 수 있기 때문에 알아 둔다면 분명 도움이 될 것이라 생각한다. :-)

## References

1\. .DS_Store, Wikipedia, http://en.wikipedia.org/wiki/.DS_Store

2\. DS Store File Format, MozillaWiki, https://wiki.mozilla.org/DS_Store_File_Format

3\. DSStoreFormat, CPAN,http://search.cpan.org/~wiml/Mac-Finder-DSStore/DSStoreFormat.pod

4\. Tools - Time Maker, Archaeological Dig for Digital Forensics, http://dorumugs.blogspot.kr/2013/02/tools-time-maker.html