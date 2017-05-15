---
layout: post
title: '[제1회 FI 세미나] 사이버전 악성코드의 특징 질의응답'
date: 2014-08-11 10:52:48.000000000 +09:00
type: post
published: true
status: publish
categories:
- Challenge &amp; Conference
- Malware Analysis
tags:
- Cyber warfare
- F-INSIGHT
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
  pyre_fimg_width: ''
  pyre_fimg_height: ''
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
  avada_post_views_count: '2223'
  sbg_selected_sidebar: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_replacement: a:1:{i:0;s:12:"Blog Sidebar";}
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p>제 1회 포렌식 인사이트 세미나에서 발표된 내용 중 질문에 대한 답변을 작성하였습니다. 여기에 작성된 내용은 저의 의견이며, 특정 집단의 생각과는 무관하오니 참조 정도 하시면 좋을 것 같습니다.<br />
&nbsp;<br />
<strong>Q. 세미나 자료를 받을 수 있을까요?</strong><br />
A. 여러가지 문제(?)가 있어서 검토 후에 올리도록하겠습니다.</p>
<div><strong>Q. 우리나라를 공격하기위해 만들어진 사이버전 악성코드가 있나요???</strong></div>
<div>A. 현재까지 우리나라에서 발생하는 공격은 첩보행위나 무력화보다는 일반 사용자의 클라이언트 파괴를 목적으로 활동했습니다.</div>
<div></div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 크라이시스 악성코드에서는 호스트 운영체제에서 vmware 외에 아무런 행동도 하지 않으면 감염될 가능성이 없나요? 그렇다면 vmware 게스트에서 취약점을 통해 호스트를 감염시키는 사이버전 악성코드가 있었나요?</strong></div>
<div>A. 크라이시스 악성코드는 자바 제로데이를 통해 전파되었으며, 호스트 시스템에서도 첩보활동을 수행합니다. 즉, 호스트/게스트 운영체제에서 활동합니다. 게스트의 취약점을 통해 공격한 사이버전 악성코드는 발견된 사례가 없습니다.</div>
<div></div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 여러 OS에서 범용으로 작동하는 악성코드의 경우... 일반적으로 windows / linux / macosx 에서 모두 동작하는 프로그램이 따로 존재하나요? 보통은 플랫폼에 따라서 다른 포팅이 있는데.. 그렇다면 백신상에 OS를 탐지하는 루틴을 발견하면 차단 시키는 백신상에서의 대응법은 효과가 없을까요?</strong></div>
<div>A. 자바 취약점을 통해 실행된 경우 자바 코드 상에서 운영체제를 판단합니다. 운영체제 탐지 루틴 자체는 정상적인 코드를 수행하기 때문에 안티바이러스에서 잡을 수 있는 방법은 없습니다. 악성코드는 운영체제를 판단하고 그에 맞는 악성코드 바이너리를 떨어트려 실행합니다. 즉, 운영체제 별로 프로그램을 다 따로 가지고 있습니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 미디어에서 말하는 북한소행(?)의 악성코드가 실제 있는지. 분석하신 사례가 있다면 특징이 어떤지 알고 싶습니다</strong></div>
<div>A. 실제로 발견됐다고 확답드릴 순 없습니다.단, 북한입장에서는 악성코드만큼 정보 수집하기 용이한 방법도 없을 듯 하네요.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 특정 악성코드가 VM ware를 감염시키다고 하셨는데요 스냅샷 기능으로 이전으로 돌렸을때도 계속 감염이 되어 있는 상태도 있나요?</strong></div>
<div>A. 감염 전 스냅샷이 있다면 문제 없습니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 토렌토 등으로 자료를 받을 때. 아니면 기사를 보기위해 접속했을때 악성코드가 침투했을때 백신으로 발견되지않은 악성코드는 일반사용자가 발견하는 방법은 없나요? 발표자님 말씀대로 그냥 모르는채 계속 사용하는 수밖에없는건가요??..</strong></div>
<div>A. 일반사용자로써는 방법은 없습니다. 단, 이런 사이버전 악성코드의 특징은 감염된 시스템이 공격 타겟이 아니라면 활동을 안하는 특징도 가지고 있습니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 한가지 더.. 일부 말웨어 분석 업무 하시는 분들은 안티바이러 무용론을 말씀하시기도 하는데요. 발표자님의 개인적인 생각을 듣고 싶습니다. 개인이 공격에 대응할 수 있는 방법에는 어떤게 있을런지요</strong></div>
<div>A. 안티바이러스는 사전 예방이 아닌, 알려진 공격을 방어하는데 사용된다고 보시면 됩니다. 사이버전 악성코드와 다르게 일반적인 악성코드는 이미 알려진 취약점을 이용하는 사례도 많으며, 다양한 공격 징후 포착에도 도움을 주고 있습니다. 안티바이러스는 현재 개인이 보안 지식이 없더라도 공격에 대응할 수 있는 유용한 방법 중 하나입니다. 만약 좀 더 보안을 고민하신다면, 안티바이러스+방화벽 형태를 추천합니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 발표에서 언급된 악성코드는 하나의 프로젝트팀을 구성해 오랜 시간 동안 제작하고, 테스트를 통해 탐지되지 않도록 제작된다고 들었습니다. 이러한 악성코드들은 어떻게 탐지/발견 되었고, 세상에 알려지게 됐는지 궁금합니다.</strong></div>
<div>A. 다양한 상황에 의해 발견됩니다. 예상치 못하게 감염 시스템에서 에러가 발생하거나, 특정 국가에서 새로운 도메인으로 다수가 접근하는 경우가 있습니다. 또한, 다른 악성코드 침해사고 분석 과정에서 우연히 발견되기도 합니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 악성코드 제작의 조직화와 지원을 언급하셨는데 밝혀진 지원 업체 또는 국가가 있나요? 분석과 정보 수집 중에 본 비하인드 스토리(음모론)을 듣고 싶습니다.</strong></div>
<div>A. 스턱스넷 패밀리는 군사 초 강대국에서 한 것으로 알려져 있습니다. 발표에서 설명 드렸지만 Stuxnet은 이란 원자력 발전소 마비, Duqu는 차세대 스턱스넷을 위한 정보 수집, Gauss와 Flame은 중동지역 자금 흐름을 통해 중동 지원의 배후를 찾기 위함으로 알려져 있습니다.</div>
<div>역으로 Careto는 대부분 미국+유렵을 감염시켰습니다. 중동지역의 첩보활동으로 추정됩니다.</div>
<div>Havex는 유렵 발전소를 대상으로 OPC 서버 정보 수집을 수행합니다. 배후나 비하인드 스토리에 대한 자세한 정보는 CrowdStrike의 2013 Annual Report를 읽어보시는 걸 추천합니다 :-)</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 혹시 이런 사이버전에 사용되는 악성코드로 인해 확전 혹은 사이버전 이후 무력행위로 이어진 케이스가 있나요?</strong></div>
<div>A. 미국에서 무력으로 대응하겠다 공표를 했지만 대부분의 악성코드는 배후 자체를 단정할 수 없기 때문에 무력행위로 이어진 사례는 없습니다. 명확한 증거가 있지 않는 이상 쉽지 않은 결정이지요.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 사이버전과 사이버테러, 사이버범죄 악성코드는 어떻게 분류하나요!?</strong><br />
A. 사이버전은 말그래도 사이버 세상의 전쟁입니다. 인프라 파괴,마비/첩보를 의미합니다.</div>
<div>사이버테러는 민간에서 사용하는 클라이언트 공격 정보로 볼 수 있습니다 국내에서 발생한 3.20이 대표적인 케이스입니다.</div>
<div>사이버범죄는 사이버에서 발생하는 민/형사 범죄를 말합니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 악성코드와 악성코드 제작자 또는 집단을 특정 지을 수 있는 판단 근거나 방법론은!?</strong></div>
<div>A. 악성코드 상에 남는 디버그 메시지, 컴파일 날짜, 어셈블리 코드의 일치성 등으로 판단합니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 오늘 의하신 악성코드의 원제작자들이 정확히 밝혀진 사례가 있나요? 그리고 앞으로 이런 악성코드에 대한 대비책은 무엇일까요?</strong></div>
<div>A. 오늘 발표된 악성코드는 원제작자가 밝혀진 사례는 없습니다. 밝혀지지 않는 것이 사이버전 악성코드의 가장 큰 매력(?)이지요. 사이버전에 사용된 악성 파일은 대부분 장기적인 테스트를 거쳤기 때문에 더더욱 발견하기 힘듭니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 보안프로그램이 악성코드인지 검사를 할 때 일정 크기이상일 경우 검사를 못하는것으로 알고있는데요. 악성코드 뒤에 더미를 많이 붙여서 제한된 크기를 넘겨서 유포를 하면 보안업체에선 이런 악성코드를 어떻게 탐지하나요?</strong></div>
<div>A. 그런가요? 국내산의 특징일진 모르겠는데 어짜피 시그너처 기반이기 때문에 파일크기는 상관이 없는 것으로 알고 있습니다 :-) AV업체 소속이 아니라 정확한 답변은 어렵네요.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. Crisis가 vmware를 감염시킬수있다고하였는데 좀더 자세히 다시 설명해주실수있나요..!?</strong></div>
<div>A. 질의 응답 시간에 답한 것 같네요 :-)</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 한국을 대상으로 배포된 악성코드를 분석하신 사례가 있으시다면 간단하게 소개해주실 수 있을까요? 그리고 이러한 악성코드가 배포되고 있을때 빠르게 수집할 수 있는 좋은 방안이 있을지도 궁금합니다.</strong></div>
<div>A. 한국으로 ‘사이버전’이라 명명할 만한 사례는 없던 것 같습니다. ‘사이버 테러’는 있었지만요.. 현재까지 빠르게 찾기 위한 방법은 자동 동적 분석 도구가 유일한 방법일 것 같습니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 여태까지 발견된 사이버전 악성코드들의 특징에 관한 발표를 하셨는데 앞으로 개발될 사이버전 악성코드에 대한 생각이 궁금합니다!</strong></div>
<div>A. 첩보 행위는 계속될 것입니다. 윈도우에서 제공하는 API와 같이 합법(?)적인 방법을 사용하기 때문에 잡기도 힘들 것이고요. 문제는 이게 자국의 감시로 이어질 수 있다는 점입니다. 미국의 PRISM 과 마찬가지로 각 나라에서 통제를 목적으로 사용할 경우에는 더 큰 문제가 될 수 있습니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 다양한 악성코드를 많이 분석해보신 것 같은데, 개인적으로 샘플파일 구하시나요? 아니면 직업적으로 수행하시는 프로젝트의 일환으로 다양한 악성코드를 접하실 수 있는 기회가 있으신지요? 악성코드 구하시는 방법을 알려주세요~</strong></div>
<div>A. 구글은 우리의 친구입니다 :D 또는 주변 지인을 통하고 있습니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 분석된 악성코드는 행위나 특징 등을 프로파일링하는데 어떤식으로 정보들을 저장하고 악성코드 간의 연관관계를 정리하나요?</strong></div>
<div>A. 저희도 아직 명확한 프로파일링 기준을 정하진 않고 있습니다만, 행위나 형태 감염 국가 통계 국가간의 관계를 기반으로 하면 연관관계 파악에 도움이될 것 같습니다. 최근에는 악성코드 활동 시점과 국가간의 관계 정보를 통해 추론을 진행하기도 합니다. 국가간의 관계는 뉴스를 통해 확보가 가능하고요.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 모듈화된 악성코드가 침투대상의 보호기제들을 우회할 수 있던 특징에 대해 궁금합니다</strong></div>
<div>A. 우회 방법은 생각보다 간단합니다. 시그니처 기반이나 문자열 추출로 악성 행위를 의심할 수 있는 데이터가 있을 경우 XOR와 곱셈(MUL)으로 인코딩합니다. 또한 네트워크 기반 탐지 시스템을 우회하기 위해 모든 통신을 암호화합니다. 최근엔 공개키 암호화를 많이해서 사후 트래픽 분석이 더 까다로워지고 있습니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 탐지우회를 검증후 제작된 악성코드들의 최초 탐지 배경 및 방법에 대한 내용이 궁굼합니다.</strong></div>
<div>A. 위에 답변 참조바랍니다 :D</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 사이버전 관련 악성코드들은 모듈화가 잘 이루어져있다고 말씀해 주셨는데요 공부하는 입장에서 해샘플의 모듈중 한두개는 찾을수 있지만 구조를 파악할만한 전체 모듈을 구하기는 힘들다고 생각합니다 참고할만한 자료나 출처같은 것이 있나요?</strong></div>
<div>A. 말씀하신대로 분석을 진행하다보면 최종 임무를 수행하는 모듈을 습득하지 못할 수 있습니다. 이 경우, 샘플의 해시 값을 이용하여 악성코드 샘플 제공 사이트를 찾아봅니다. 이 경우에도 나타나지 않으면, 보고서의 내용을 참조하고 있습니다. Kaspersky, Eset, Symantec과 같은 백신 업체에서 나오는 보고서는 내용이 상세하게 잘 작성되어 있기 때문에 보고서만으로도 내용 파악이 가능합니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 소름끼치는 고도화된 악성코드에 대해 설명해 주신것 잘 들었습니다. 보통 개발 단계에서 분석을 방해 하는 것보다 탐지 자체를 방해하는 쪽으로 초점을 맞춘다 하셨는데 혹시 특정 포렌식 도구를 방해할 목적으로 동작하는 악성코드를 분석하신 적 있으신가요?</strong></div>
<div>A. 특정 포렌식 도구에 대한 방해는 많이 하지 않습니다. 이는 어찌됐던 탐지되는 순간 분석은 언젠가는 될 것이라는 판단 때문으로 보입니다. 단, 발표에서 설명드렸던 것처럼 역추적을 방지하기 위한 안티-포렌식 행위는 하고 있습니다.</div>
<div></div>
<p>&nbsp;</p>
<div><strong>Q. 범죄전반적으로 포렌식이 많이 사용 되는데 가장 시간이 많이 소요되는 분야는 어느 부분인가요?</strong></div>
<div>A. 포렌식에서 소요가 많이되는건 숨겨진 의도 파악입니다. 저장장치 분석으로 의중을 파악하더라도 본래 숨겨진 의도를 파악하지 못할 수 있습니다. 의도를 파악하려면 결국 악성코드를 상세분석할 수밖에 없고요. 사실 디지털 포렌식이라는게 가설을 사실로 입증시키기 위함이다보니, 좀 더 분석의 신뢰도를 높이려면 추출된 샘플의 상세한 분석이 필요합니다.</div>
