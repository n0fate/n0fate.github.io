---
layout: post
title: KDFS 챌린지에 바라는 점
date: 2015-12-11 12:30:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Challenge &amp; Conference
tags:
- KDFS
meta:
  avada_post_views_count: '1340'
  _edit_last: '1'
  slide_template: default
  fusion_builder_status: inactive
  pyre_show_first_featured_image: 'no'
  pyre_portfolio_width_100: default
  pyre_video: ''
  pyre_fimg_width: ''
  pyre_fimg_height: ''
  pyre_image_rollover_icons: default
  pyre_link_icon_url: ''
  pyre_post_links_target: 'no'
  pyre_related_posts: default
  pyre_share_box: default
  pyre_post_pagination: default
  pyre_author_info: default
  pyre_post_meta: default
  pyre_post_comments: default
  pyre_main_top_padding: ''
  pyre_main_bottom_padding: ''
  pyre_hundredp_padding: ''
  pyre_slider_position: default
  pyre_slider_type: 'no'
  pyre_slider: '0'
  pyre_wooslider: '0'
  pyre_revslider: '0'
  pyre_elasticslider: '0'
  pyre_fallback: ''
  pyre_avada_rev_styles: default
  pyre_display_header: 'yes'
  pyre_header_100_width: default
  pyre_header_bg: ''
  pyre_header_bg_color: ''
  pyre_header_bg_opacity: ''
  pyre_header_bg_full: 'no'
  pyre_header_bg_repeat: repeat
  pyre_displayed_menu: default
  pyre_display_footer: default
  pyre_display_copyright: default
  pyre_footer_100_width: default
  pyre_sidebar_position: default
  pyre_sidebar_bg_color: ''
  pyre_page_bg_layout: default
  pyre_page_bg: ''
  pyre_page_bg_color: ''
  pyre_page_bg_full: 'no'
  pyre_page_bg_repeat: repeat
  pyre_wide_page_bg: ''
  pyre_wide_page_bg_color: ''
  pyre_wide_page_bg_full: 'no'
  pyre_wide_page_bg_repeat: repeat
  pyre_page_title: default
  pyre_page_title_text: default
  pyre_page_title_text_alignment: default
  pyre_page_title_100_width: default
  pyre_page_title_custom_text: ''
  pyre_page_title_text_size: ''
  pyre_page_title_custom_subheader: ''
  pyre_page_title_custom_subheader_text_size: ''
  pyre_page_title_font_color: ''
  pyre_page_title_height: ''
  pyre_page_title_mobile_height: ''
  pyre_page_title_bar_bg: ''
  pyre_page_title_bar_bg_retina: ''
  pyre_page_title_bar_bg_color: ''
  pyre_page_title_bar_borders_color: ''
  pyre_page_title_bar_bg_full: default
  pyre_page_title_bg_parallax: default
  pyre_page_title_breadcrumbs_search_bar: default
  _wp_old_slug: kdfs-%ea%b2%b0%ea%b3%bc%eb%a5%bc-%eb%b3%b4%ea%b3%a0-%ec%95%84%ec%89%ac%ec%9a%b4-%ec%a0%90-%eb%b3%b4%ea%b3%a0%ec%84%9c
  sbg_selected_sidebar: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_replacement: a:1:{i:0;s:0:"";}
  sbg_selected_sidebar_2: a:1:{i:0;s:1:"0";}
  sbg_selected_sidebar_2_replacement: a:1:{i:0;s:0:"";}
author:
  login: n0fate
  email: rapfer@gmail.com
  display_name: n0fate
  first_name: ''
  last_name: ''
---
<p><strong>참고로 본 글은 챌린지에 참여한 참여자나 출제팀을 무시하거나 비난하는 것이 아닌 고민해야할 점을 다룬 글입니다. 그 분들의 보고서는 제 것보다 뛰어났으며, 출제팀도 많은 고민을 통해 이러한 판단 기준을 세운 것으로 보입니다. :-)</strong></p>
<h2>시작</h2>
<p>KDFS 2015 결과가 나왔습니다. 아쉽게도 저는 수상하진 못했네요. 하하. 일단 수상한 다른 분들에게 감사의 말을 전합니다. 수상하신 분들의 보고서를 보니 제가 작성한 페이지보다 훨씬 많은 내용을 다루고 있었습니다+_+. 고생한게 보이는 보고서 였습니다. 그리고 수상 보고서를 쭉 보다보니 요번 포렌식 챌린지에서 출제팀과 평가팀의 의도가 파악이 되더군요.</p>
<p>이 챌린지는 '<strong>침해사고 당한 공유기의 디지털 포렌식 분석 보고서</strong>’ 가 아닌 ‘<strong>침해사고 당한 공유기 분석 기술 보고서</strong>’였다는 점입니다. (물론 의도는 출제팀에서만 알겠습니다만 보고서를 보는 입장에서는 그랬습니다.) 그래서 이 부분에 아쉬운 점을 적어보고자 합니다.<br />
<strong><br />
- 혹시나 필요하실까하여 </strong><strong>제 보고서는 Docs 에 올려두었습니다.</strong></p>
<h3>1. 포렌식 분석 보고서인가 침해사고 기술 보고서인가?</h3>
<p>본 챌린지는 펌웨어를 분석해서 증거를 채증하고 이에 대한 결함을 다루는 챌린지입니다. 생소한 기기가 아닌 잘 알려진 공유기 하나를 대상으로 하다보니 실제 케이스보다 분석이 쉬운 편이였구요. 또한 저용량 임베디드 시스템인 만큼 로깅이 잘되는 것도 아니다보니 포렌식적으로 타임라인을 구성하거나 하는 분석도 무의미했습니다. 이런 점으로 인해 문제에 다양한 포렌식 기술이 적용되기 힘들 수밖에 없었죠.<br />
문제 자체도 펌웨어에 저장된 악성코드를 찾고 그 기능에 중점을 두었을 뿐 침입 경로를 유추하라는 내용은 없었습니다. 이 부분은 전에 'KDFS 2015 챌린지 후기’에서도 아쉬웠음을 밝힌 내용인데요. 그로인해 대부분의 대회 참여자들이 문제를 완벽하게 풀게 된 듯 합니다 ㅎㅎ. (아는 분은 일주일만에 다 풀기도 했다.) 즉 단순 문제 풀이만으로 평가한다면 모두가 만점이 될 수 밖에 없는 것이지요.. 이에 출제팀 측은 문제 자체를 다 맞춘 상황에서 보고서의 퀄리티를 중점적으로 파악할 수 밖에 없었을 것이라 생각되네요. 여기에서 출제팀과 저와의 생각의 차이가 있던 것 같습니다. 물론 제 보고서에서 틀린 점이 있을 수도 있긴 하지만요. ;-)<br />
저는 포렌식 보고서에는 코드의 스크린샷의 무의미하다고 생각했습니다. 분석 보고서에는 발생한 증거(본 챌린지에서는 문제)만 다루면 되지 그에 분석 과정을 일일히 코드로 다루면 오히려 보고서를 보는 사람의 입장에서 보고자하는 내용이 한눈에 들어오지 않기 때문이라는 생각에서였지요.<br />
처음에 포렌식 증거 분석 보고서를 작성할 때 이 부분 때문에 고생을 하다보니 더 그랬던 것 같습니다. 그 때 항상 들었던 이야기가 법원 증거로 제출되는 분석 보고서는 디지털 포렌식의 기본 요소를 지키면서 디지털 포렌식을 모르는 사람이 보더라도 이해할 수 있도록 작성해야 한다는 것이였는데, 이 과정을 거치면서 보고서의 많은 내용이 제거되더군요. 당연히 핵심적인 스크린샷과 설명을 제외한 내용이 문서에서 제거될 것이구요. 물론 필자가 그 때 당시 보고서 수준으로 작성한 것은 아닙니다.... 문제 자체가 기술적인 부분을 요구했으니..<br />
그래서 풀이 보고서의 '<strong>문제 풀이</strong>' 파트를 따로 두어, 코드에 대한 내용을 최대한 배재하고, 정 필요하면 상세보고서를 보라고 했습니다. 공개된 수상 보고서를 보면 제가 작성한 기본적인 내용을 포함하고 그걸 뒷받침하기 위해서인지 많은 코드 스샷과 그에 대한 내용을 기술했습니다. 그런데 필자의 생각은 이게 들어가는게 맞냐는 것입니다. 역으로 생각하면 들어가면 안되는 내용이 보고서에 들어간게 아닌가? 아니면 내가 잘못알고 있는 것인가? 하는 생각이 들더군요.</p>
<h3>2. 추가 공격 시나리오 서술의 범위?</h3>
<p>추가 공격 시나리오를 작성하라는 문제가 있습니다. 이 부분에서도 평가가 많이 갈린 것 같은데요. 저의 경우에는 포렌식 챌린지이니, 본 케이스에만 맞춰서 발생할 수 있는 현재 상태에서 공격자가 할 수 있는 <strong>추가</strong> 공격 시나리오를 생각했습니다. 문제를 풀어본 사람은 알겠지만 블로그를 C&amp;C 형태로 사용해서 이미지 파일에 있는 임무 코드를 변경할 수 있습니다.(고 판단했습니다). 그러면, 공격자는 임무 코드를 변경해서 다음 부팅 시 다른 임무를 수행하게 할 수 있을 것이다라고 판단했죠. 그 외에 네트워크 모니터링을 통한 정보 탈취, DNS 스푸핑 등의 요소는 사실 현재 침해사고 케이스를 벗어나는 것이라 생각했다. (사실 요건 디지털 포렌식 챌린지랑은 약간 벗어난 듯 합니다 ㅎㅎ)<br />
하지만 수상 보고서를 보면, 공격자가 쉘을 가질 수 있으니.. 를 전제로 할 수 있는 모든 것이 작성된 보고서가 많았습니다. 제가 말하고자 하는 부분은 이 내용이 공격자가 이 환경 기반으로 어떤 공격을 또 할 수 있을까가 되어야 하지 않냐는 것입니다. 그로 인해 파생되는 공격은 (필자가 생각하는) 추가 공격 시나리오의 범위 밖이며, 사실 이 내용을 쓰면 이 것만 2~300쪽을 쓸 수 있을 정도로 방대한데 그 것을 넣음으로 침해사고의 핵심 내용이 가려지는 것이 아닐까? 하는 생각이 들었습니다.</p>
<h2>이 문제를 해결방안이 딱히 없다는게 문제</h2>
<p>출제 경험이 있던 사람으로써 말하자면, 디지털 포렌식은 문제로 내는게 쉽진 않습니다. 단순히 정답을 맞추느냐 맞추지 못했느냐의 관점으로 하면 포렌식의 절차의 연속성과 과정의 신뢰성, 그리고 포렌식에서 가장 중요한 보고서 작성에 대한 내용이 들어가지 않기 때문입니다. 그렇다고 지금같은 챌린지를하게되면, 보고서에 주관적인 판단이 들어가게 되고, 들어가지 말아야할 정보가 보고서에 들어갔다고 해서 그걸 오히려 감점을 주게된다면, 공정성의 문제가 발생하게 됩니다. 특히 이번 챌린지는 기간이 상당히 길었기 때문에 평가 시 이 문제가 더 커지게 된 듯합니다.<br />
가장 <strong>이상적인</strong> 침해사고 챌린지는 아예 해킹 대회처럼 일정 인원으로 팀을 구성하여 이틀의 시간을 투자해서 보고서까지 제출(인원 및 시간에 대한 제한)하는 것이라 생각합니다만.. 온라인 대회는 인원에 대한 제한이 불가능하다는 것이 문제이지요..<br />
국내에 디지털 포렌식 챌린지가 몇개 있었지만, KDFS에서 하는 챌린지는 가장 공정한 평가를 수행하고 논란의 여지가 적다고 생각합니다. 위 2개의 사항에 대해서는 아직도 고개가 갸우뚱하게 되지만 말이죠.^^ 앞으로도 다양한 상황을 고려하여 좀 더 나은 챌린지가 되길 기대하면서 글을 마칩니다.</p>
