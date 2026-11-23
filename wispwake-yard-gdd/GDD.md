# Wispwake Yard 구현용 게임 설계서

버전 1.0 · 확정일 2026-11-23 (Asia/Seoul) · Windows Steam · Godot 4 · 한국어 원문

**위스프웨이크 야드**는 낮에 묘지를 설계하고, 밤에 깨어난 영혼의 관계와 사건을 돌보는 2D 픽셀아트 경영 시뮬레이션이다. 핵심은 **누구의 묘비를 어디에 두느냐가 다음 밤의 선택과 이야기를 바꾼다**는 것이다. 이 URL의 본문만으로 새 Godot 프로젝트를 만들 수 있도록 기본 규칙, 데이터 계약, 개발 순서, 검수 기준을 함께 고정한다.

장기 목표는 “스토리를 다 읽으면 반복 수금만 남는 게임”을 피하는 것이다. **업데이트 가능한 콘텐츠 팩**과 **업데이트 없이 재조합되는 시뮬레이션**을 별개의 필수 축으로 구현한다. 6개월 이상 다시 찾고 싶은 게임을 목표로 하지만 기간 자체를 약속하지 않는다. 수치와 재미 가설은 아래 장기 검증을 통과해야 한다.

## 01 문서 사용과 결정 우선순위

- 이 문서는 기획 및 구현 계약이다. 게임 실행 파일이나 이미 테스트를 통과한 제품이 아니다.
- 요구사항의 MUST는 필수, SHOULD는 조정 가능, LATER는 후속 확장이다. 표에 “초기값”으로 적은 수치는 플레이테스트로 조정하되 구조와 검수 목적은 유지한다.
- 충돌 시 우선순위: 사용자의 새 지시 → 이 문서의 필수 요구사항 → 수치표 → 예시 → 시각 연출. 미정인 사항은 아래 기본값을 채택하고 결정 기록에 남긴다.
- 이전 TINY GRAVEYARD 대화의 귀여운 묘지, 밤의 위기, 배치·관계·성장·원인 설명을 계승한다. 고유 영혼 20명·시설 30종 등은 당시의 확장 목표였으며 최초 출시의 강제 수량으로 해석하지 않는다.
- 이번 설계에서 추가한 장기 기억, 운영 정책, 순환 의뢰, 계절, 콘텐츠 팩 호환성은 새 필수 요구사항이다. 프로토타입 단계에서 제외된 항목을 정식 출시에서도 삭제해선 안 된다.
- 플레이어에게 현실 시간 접속 의무, 연속 출석, 기간 한정 상실, 오프라인 벌금, 강제 환생, 무한 능력치 인플레이션을 요구하지 않는다.
- 플랫폼은 오프라인 싱글플레이. 게임 진행 중 LLM/API/서버 연결은 사용하지 않는다. 구매, 광고, 라이브 서비스 운영 시스템은 범위 밖이다.

## 02 프로젝트명 조사와 채택

**영문명: Wispwake Yard / 한글 표기: 위스프웨이크 야드 / 내부 slug: wispwake-yard**

Wisp(작은 영혼) + wake(깨어남·밤샘의 어감) + yard(작은 묘지·뜰)의 조합이다. 이름만으로 “작은 공간이 밤에 살아난다”는 방향을 담는다.

2026-11-23 조사 기록:

| 후보 | Steam 공개 storesearch 결과 | 일반 웹 및 Steam 도메인 검색 | 판단 |
| --- | --- | --- | --- |
| Wispwake Yard | 전체 0건, items 빈 배열 | 정확한 전체 명칭의 출시 게임 발견 못함. Wispwake 단어는 판타지 설정·이름 목록에 사용됨 | 채택 |
| Gravemallow | 전체 0건, items 빈 배열 | 같은 이름의 출시 게임 발견 못함. 일반 이름 제안 글에 단어가 사용됨 | 묘지보다 캐릭터 이름처럼 들려 보류 |
| Mournbloom Hollow | 전체 0건, items 빈 배열 | 같은 전체 명칭의 출시 게임 발견 못함. Mournbloom은 기존 게임 내 소재명 등에 사용됨 | 의미 중복을 줄이기 위해 보류 |

조회 주소: [Wispwake Yard Steam 검색](https://store.steampowered.com/search/?term=Wispwake%20Yard), [Gravemallow Steam 검색](https://store.steampowered.com/search/?term=Gravemallow), [Mournbloom Hollow Steam 검색](https://store.steampowered.com/search/?term=Mournbloom%20Hollow). 실제 수량 확인에는 Steam 공개 검색 응답을 사용했다: [Wispwake Yard 응답](https://store.steampowered.com/api/storesearch/?term=Wispwake%20Yard&l=english&cc=US), [Gravemallow 응답](https://store.steampowered.com/api/storesearch/?term=Gravemallow&l=english&cc=US), [Mournbloom Hollow 응답](https://store.steampowered.com/api/storesearch/?term=Mournbloom%20Hollow&l=english&cc=US).

웹 질의는 전체 명칭 따옴표 검색, 전체 명칭+game, Wispwake+game, Steam app 도메인 내 이름 검색을 사용했다. 단어의 기존 사용 예: [Wispwake가 포함된 공개 판타지 설정](https://www.worldanvil.com/w/aerthamar-mrkey/a/the-heartreach-article), [Gravemallow 이름 목록](https://namespromax.com/weird-names/). **조사 범위에서 동일한 출시 게임명을 찾지 못했다는 결론이며, 전 세계 미색인 작품·미출시 상표까지 독점 사용을 보장하는 결론은 아니다.** 스토어 페이지 등록 전 같은 검색을 다시 수행한다.

## 03 제품의 약속과 재미 기준

플레이어는 폭력으로 유령을 제거하는 전사가 아니라 이웃의 취향과 갈등을 설계하는 묘지 관리인이다. 죽음은 아늑한 사후 공동체의 소재이며 고어·점프 스케어는 쓰지 않는다. 귀여움 70%, 기묘함 30%를 시각적 방향으로 삼는다.

세 가지 핵심 질문:

1. 오늘 어떤 영혼을 이웃으로 둘 것인가?
2. 안정과 높은 생산 중 이번 밤에는 무엇을 선택할 것인가?
3. 어젯밤의 기억을 다음 운영 방식에 어떻게 반영할 것인가?

좋은 밤은 위기 수가 많아서가 아니라 **배치의 결과가 보이고 다음 계획이 생겨서** 좋다. 평온한 밤에도 공동 작업, 관계 변화, 정원 가꾸기와 작은 기억이 남는다. 위기를 의도적으로 만들지 않아도 보상·이주·스토리 진행이 가능해야 한다.

| 항목 | 기본 결정 |
| --- | --- |
| 시점·조작 | 탑다운, 마우스 중심 건설·관리, 키보드 단축키 |
| 영혼 조작 | 자율 AI. 개별 이동 명령은 안내 능력만 허용 |
| 세션 | 20~40분에 3~6밤; 낮 체류에 제한 없음 |
| 하루 | 낮 준비 → 180초 밤 → 시간제한 없는 새벽 보고 |
| 자원 | 골드 G, 영혼 에너지 E(0~100), 전역 불안 U(0~100) |
| 패배 | 영구 게임오버 없음. 다음 날 복구 가능한 비용·불편 |
| 장기 플레이 | 지속 세계, 선택형 이야기 장, 엔딩 후 동일 세이브 계속 |
| 타깃 | 설계·관찰·수집·자기표현·약한 전략적 긴장을 좋아하는 플레이어 |

## 04 범위와 완료 단계

| 단계 | 반드시 제공할 것 | 완료 의미 |
| --- | --- | --- |
| P0 규칙 실험 | 24×16 맵, 영혼 3명, 묘비 포함 시설 6종, 다툼·저주, 종·봉인, 저장, 7밤 | 약 20~40분의 핵심 루프 검증. 판매 완성품 아님 |
| P1 수직 단면 | 영혼 5명, 시설 10종, 사건 6종, 안내, 관계 기억, 운영 정책 3개, 계절 2개, 팩 추가 실험 | 장기 구조가 실제 코드로 작동 |
| P2 정식 최소 범위 | 4구역, 고유 영혼 12명, 생성 영혼, 시설 18종, 사건 템플릿 12종, 관계 장면 16개, 계절 4개, 정책 6개, 의뢰 8종, 지속 세계 | 장기 검증을 시작할 콘텐츠 기반 |
| P3 출시 후보 | P2 전체 + 튜토리얼, 한국어/영어, 접근성, 성능, 세이브 호환, Windows 배포, 30시간 이상 사용자 테스트 | 기술·재미 출고 기준 통과 필요 |
| LATER | 고유 영혼 20명 이상, 시설 30종 이상, 새 사건 동사, 추가 구역, Workshop 연결, 컨트롤러 | 품질 검증 뒤 팩으로 누적 확장 |

**P2의 생성·기억·정책·계절·팩 구조는 선택 기능이 아니다.** 제작 인력이 부족하면 시설과 대사 수를 줄여 재합의하고, 이 시스템들을 조용히 삭제하지 않는다. 일정은 작업량 추정으로 관리하며 “6주면 정식 출시” 같은 고정 약속을 하지 않는다.

## 05 핵심 루프와 시간 계약

낮: 새벽 결과 확인 → 영혼 영입/휴식/복귀 → 묘비·시설 재배치 → 구역 정책·계절 예보 확인 → 위험 예측 → 밤 시작.

밤: 기상 → 욕구·취향·관계에 따른 자율 활동 → 사건 전조 → 플레이어 대응 또는 관찰 → 결과·기억 기록 → 귀환.

새벽: 마지막 미결 사건 정리 → G/E/U 정산 → 관계·의뢰·해금 갱신 → 하루당 최대 3개의 중요한 변화 제시 → 다음 낮.

| 상태 | 시간과 허용 동작 |
| --- | --- |
| DAY | 시뮬레이션 정지. 건설·철거·영입·정책 변경·저장 허용 |
| NIGHT | 1,800 simulation ticks, 1tick=0.1초. 기본 1배, 2배, 일시정지 |
| DAWN | 계산은 1회만 실행. 보고서 닫기 전까지 시간 정지 |
| MENU/PAUSE | 시간·AI·쿨다운·사건 마감 모두 정지. UI 조작 가능 |

NIGHT tick 0~29는 기상 연출, 30~1699는 활동, 1700부터 새 사건을 만들지 않으며 귀환, tick 1800에 새벽 진입. 만료하지 않은 사건은 1700~1799에 기존 타이머를 진행하고, 1800에서 해당 사건의 dawn_outcome을 1회 적용한다. 귀환 실패 영혼은 1800에 자기 묘비 접근 칸으로 이동시켜 저장 안정성을 확보한다.

2배속은 tick을 두 배 처리한다. 경제와 사건 확률을 두 번 적용하는 별도 코드 경로를 만들지 않는다. 렌더 프레임과 시뮬레이션 tick은 분리한다. 0~100 수치는 정수, 소수 누적은 1/100 고정 소수점으로 저장한다. 시간은 현실 시각이 아닌 tick/day로 저장한다.

자동 일시정지 기본값은 심각한 사건 전조 발생 시 켜짐. 매번 설정을 기억한다. 밤 종료를 자동으로 연달아 실행하지 않는다.

## 06 맵·건설·접근 규칙

좌표는 좌상단 (0,0), x 오른쪽, y 아래. 초기 맵 24×16. 16px 타일, 월드 원점 (0,0), 타일 중심=(x×16+8,y×16+8). 기본 이동은 상하좌우, 대각선 불가.

기본 fixture: 외곽은 경계, (11,1) 입구는 걷기 가능. 신목은 (11,7)부터 2×2 고정 건물, 접근 칸은 외곽 인접 보행 칸. 초깃값 G=180/E=30/U=10. 묘비 3개와 영혼 가렌·루나·모라가 각각 (6,6), (9,6), (15,6)에 거주한다. 꽃밭 (8,8), 벤치 (7,10), 등불 (13,6) 각 1개는 이미 설치되어 무료다. 흙은 기본 보행 가능하므로 길을 깔아야만 움직이는 규칙은 없다.

P2 맵은 최대 48×32를 네 24×16 구역으로 나눈다. 시작 구역은 좌상단. 구역 해금은 골드만 반복 파밍하지 않도록 “해당 단계 목표 2개 중 1개 + 비용”을 요구한다. 우상단 숲=200G, 좌하단 구묘역=300G, 우하단 달뜰=400G. 초기 맵 경계는 확장 시 내부 경계에서 제거하고 전체 지도 외곽만 남긴다. 잠긴 구역은 보행·배치 불가.

배치 검증 MUST:

- footprint 모든 칸이 해금된 맵 안이며 다른 건물, 고정 신목, 입구 예약 칸과 겹치지 않는다.
- 각 활성 묘비와 시설의 접근 칸 중 최소 하나가 입구와 연결되어야 한다. 임시 배치 상태에서 flood fill로 **기존 건물까지** 재검증한다.
- 묘비와 대부분 시설은 점유 칸 보행 불가. 꽃밭·길·장식은 보행 가능. 울타리는 보행 및 저주 확산 차단.
- 회전은 footprint와 접근 offset을 함께 90도 회전. 대칭 1×1 오브젝트는 회전 UI를 생략한다.
- 영향 거리는 footprint 각 칸 사이의 최소 Manhattan 거리. 영향은 벽을 통과하되 저주 전파와 실제 이동은 통과하지 못한다. 거리 0은 같은 보행 가능 꽃밭 위 등 실제 겹침 허용 계층에서만 발생.
- 동일 효과는 가장 강한 1개만 적용. 서로 다른 출처의 기분 보너스 합은 +15, 페널티 합은 -15로 제한. 효율 승수 최종 범위 0.5~1.5.
- 건설 확정 시에만 비용 차감. 실패·취소는 0원. 낮 이동은 무료. 낮 마지막 20회 명령의 실행 취소/다시 실행 지원.
- 철거 환급은 실제 구입 가격의 75% 내림. 지급 시설의 paid_cost=0이므로 환급 0. 거주 중 묘비는 철거 불가, 이동 가능.
- 영혼 퇴거는 휴식 보관함으로 옮기기이며 삭제가 아니다. 휴식 중 경제·AI·관계 감쇠 없음.
- 밤에는 건설·이동·판매를 잠근다. 기존 배치의 결과를 종·봉인·안내로 관리한다.

미리보기는 가능/불가 색뿐 아니라 실패 이유, 접근 칸, 영향 범위, 영향을 받는 영혼, 생산 및 위험 변화량을 표시한다. 예측은 확정 결과가 아닌 현재 배치의 조건과 위험 점수다.

## 07 시설과 비용 데이터

모든 시설은 최대 2단계. 2단계 비용은 기본 비용의 1.5배 올림, 효과는 “업그레이드 효과” 필드만 적용한다. 3단계 이상 누적 강화는 없다.

| ID(접두사 core:) | 크기 | 비용 G/E | 범위 | 기능과 상충 효과 |
| --- | --- | --- | --- | --- |
| grave | 1×1 | 30/0 | 2 | 영혼 1명 거주; 가까운 묘비 관계 노출 |
| flowerbed | 1×1 보행 | 20/0 | 2 | 꽃 선호 +8 기분, 루나 돌봄 대상; 저주 시 효과 중단 |
| bench | 2×1 | 25/0 | 2 | 교류 2석, 사회 욕구 회복; 고독 선호는 주변 교류 중 소음 -4 |
| lantern | 1×1 | 20/0 | 3 | 조명, 저주 위험 -15; 어둠 선호 -5 기분 |
| circle | 2×2 | 45/10 | 2 | 모라 작업 E+2/완료, 밤당 3회; 저주 위험 +20 |
| bell_tower | 1×2 | 60/10 | 3 | 종 능력 범위의 기준점; 가렌 순찰 +10 점수 |
| workshop | 2×2 | 60/5 | 2 | 브람 수리 작업; 주변 작업 중 소음 -4 |
| fence | 1×1 | 3/0 | 0 | 이동·저주 전파 차단, 길 막힘 검증 필수 |
| path | 1×1 보행 | 2/0 | 0 | 이동 tick 5→4, 보상·유지비 없음 |
| sanctuary | 2×2 | 50/10 | 3 | 안내 목적지, 고독 휴식; 영역 수용 2명 |
| toy_box | 1×1 | 30/0 | 2 | 장난 욕구 해소, 피피 장난 위험 -10 |
| archive | 2×2 | 80/15 | 2 | 장기 기억 요약과 전승 준비, 생산 증가 없음 |
| herb_patch | 2×1 보행 | 40/5 | 2 | 정화 작업 1회/밤, 1개 오염 제거 |
| observatory | 2×2 | 90/15 | 3 | 다음 계절 예보 정확화, 이벤트 발생 강제 아님 |
| ward_post | 1×1 | 45/10 | 2 | 침입 전조 시간 +5초, 경비 경로 지점 |
| stage | 3×2 | 70/5 | 3 | 공연·축제, 사회성 보너스와 소음 동시 생성 |
| pond | 2×2 | 40/0 | 2 | 조용한 휴식, 보행 불가·공간 비용 |
| memorial | 1×2 | 60/10 | 2 | 전승 기억 전시, 공동체 목표 표식, 수익 없음 |

P0는 grave~circle와 bell_tower의 6종. P1은 첫 10종. 2단계는 생산량보다 접근성·지속시간·수용량 개선을 우선한다. 업그레이드 기본 매핑: bench/sanctuary 수용 +1, bell_tower/lantern 범위 +1(최대4), circle 작업 횟수 +1(생산 cap는 전체 경제 cap 적용), workshop 수리 시간 10→8초, flowerbed는 꽃 선호 기분 +8→+10. toy_box는 play 욕구 회복20→25. grave, fence, path, archive, herb_patch, observatory, ward_post, stage, pond, memorial에는 1.0에서 업그레이드가 없다. 효과 없는 건물은 업그레이드 버튼을 표시하지 않는다.

내구도는 damage 0~3. 0~1 정상, 2 효율 50%, 3 기능 정지이며 시각적 구조는 보존한다. 일괄 수리는 damage 단계당 5G, 개별 수리도 동일. 건물 자연 마모는 없다. 소소한 물주기·쓰레기 줍기를 필수 수입 조건으로 두지 않는다.


## 08 영혼 AI와 관계

영혼은 불변 정의(GhostDef)와 저장 가능한 상태(GhostState)를 분리한다. 공유 Resource에 현재 기분·관계·위치를 쓰지 않는다. 행동 선택은 해석 가능한 Utility AI, 실행은 유한 상태 머신이다.

| 고유 ID | 성격·초기 기분 | 좋아함 / 싫어함 | 고유 행동 |
| --- | --- | --- | --- |
| garen 가렌 | 성실·고독, 60 | 종탑·안정 / 소음 | 순찰, 다툼 중재 |
| luna 루나 | 온화·정원사, 65 | 꽃·벤치 / 저주 | 꽃 돌봄, 주변 진정 |
| mora 모라 | 자부심·연구자, 55 | 마법진·어둠 / 소음 | 에너지 연구, 위험한 실험 |
| bram 브람 | 장인·고집, 60 | 작업장·등불 / 작업 방해 | 손상 시설 수리 |
| pipi 피피 | 장난·사교, 65 | 장난감·친구 / 고립 | 장난, 흔적 발견 |
| orin 오린 | 기록가·내향, 60 | 서고·고요 / 공연 소음 | 기억 정리 |
| vela 벨라 | 음악가·사교, 60 | 무대·관객 / 고립 | 공연, 사회 욕구 회복 |
| moss 모스 | 수호자·느긋, 60 | 연못·식물 / 오염 | 정화 |
| neri 네리 | 탐험가·호기심, 60 | 숲·안내소 / 반복 작업 | 조사 |
| tovi 토비 | 상인·신중, 60 | 교류·안정 / 큰 다툼 | 의뢰 중개 |
| sella 셀라 | 교사·온화, 65 | 새 이웃·서고 / 라이벌 소동 | 관계 중재 |
| ilun 일룬 | 천문가·독립, 60 | 전망대·어둠 / 군중 | 계절 관측 |

처음 세 명은 P0, 브람·피피는 P1. 나머지는 P2의 고유 이야기 보유 영혼이다. 이름·외형은 교체 가능하지만 역할·상충 선호·행동 계약은 유지한다. P2 생성 영혼은 위 행동 역할 중 하나 + 성격 2개 + 선호 2개 + 약점 1개 + 이름·외형 조합. 상호 배타적 성격(고독/사교 등)은 같은 슬롯에 배치하지 않는다. 생성 캐릭터의 스탯 합을 고유 캐릭터보다 높여 교체를 강요하지 않는다.

### AI 상태와 계산 주기

상태: SLEEP → WAKE → DECIDE → MOVE → ACT → DECIDE; 사건에 묶이면 EVENT; 진정되면 REST; 밤 마지막에는 RETURN → SLEEP. DECIDE와 욕구는 10tick마다 갱신, 이동은 1tick, 환경·관계 노출은 50tick마다 갱신한다.

욕구 값은 unmet_need 0~100(높을수록 부족). social/rest/purpose/play 네 개를 쓰고, 성격별 관련 욕구 두 개를 우선한다. 초기값 모두 30. 우선 욕구는 역할별로 rest+purpose(기사·연구자·장인·정원사·정화·기록·천문), rest+social(음악가·상인·교사), rest+play(장난·탐험)로 고정한다. 기분에는 우선 두 개만 사용하며 다른 욕구도 해당 행동 선택에는 쓸 수 있다. 활동 중 매 10초 +2, ACT 성공 시 해당 욕구 -20. 휴식은 rest -25. 기분은 매 5초 목표 기분에 최대 2씩 접근한다.

목표 기분 = clamp(50 + 환경 선호 합 + 관계 환경 합 - floor(우선 욕구 두 개의 평균/10) - 10×현재 저주 여부 + 계절·정책 보정, 0,100). 주변 친구 1쌍 +5, 라이벌 -7, 관계 환경 합은 ±10. 실제 기분 변화에는 reason_code와 source_id를 붙인다.

행동 후보는 도달 가능한 시설 접근 칸/영혼만 포함한다. 기본 점수:

score = base(10) + floor(need/2) + preference_match(0 또는 25) + role_bonus(0 또는 20) + relationship_bonus(-15~15) - distance_cost(min(20, 경로 길이)) - repetition_penalty(최근 3행동 중 동일 행동당 8).

유효 후보 최고 점수보다 8 이내인 후보만 대상으로 seed 기반 균등 선택한다. 동일 조건에서 순서를 entity_id로 정렬한 후 난수를 1회 사용한다. 8점보다 크게 낮은 행동을 무작위로 선택하지 않는다. 모든 후보가 막히면 2초 idle 후 재평가.

ACT 최소 유지 30tick, 완료까지 50~120tick. 위기·대상 삭제·귀환을 제외하면 중간에 새 점수로 취소하지 않는다. 새 행동이 현재 행동보다 15점 높아도 최소 유지 시간 이후에만 전환 가능. 시설 좌석은 예약 토큰으로 점유하고 ACT 종료·취소 시 반환한다. 이동 경로가 20tick 동안 전진하지 못하면 재탐색 1회, 다시 실패하면 예약 해제하고 사유 기록. 유령끼리는 충돌하지 않으며 시설 접근 칸을 공유할 수 있지만 상호작용 좌석 제한은 유지한다.

| 행동 | 시간 | 성공 효과 | 제한 |
| --- | --- | --- | --- |
| wander | 3초 | purpose -5 | 빈 칸 4개 내 선택 |
| rest | 8초 | rest -25, 기분 +3 | sanctuary 또는 자기 묘비 |
| socialize | 8초 | social -20, 관계 +3 | 상호 수락, 쌍당 20초 쿨다운 |
| tend_flower | 10초 | purpose -20, 루나 기분 +5 | 같은 꽃밭 30초 쿨다운 |
| research | 10초 | purpose -20, E ledger +2 | 마법진, 역할별 밤 cap |
| patrol | 8초 | 근처 위기 진행 1단계 완화 | 가렌, 밤당 3회 |
| repair | 10초 | damage -1 | 브람, 밤당 3회 |
| play | 6초 | play -20 | toy_box, 사건 생성은 별도 Director |
| investigate | 12초 | 실종 흔적 진행 +1 | 조사 역할 또는 안내 개입 |
| perform | 12초 | 범위 내 social -10 | 관객 ≥1, 소음 태그 12초 |

추가 역할 행동: purify(10초, herb_patch 접근, curse1 제거, 모스 밤3회), mediate(8초, 다툼 참여자 근접, 진척+1, 셀라 밤3회), archive_memory(10초, archive, 해당 영혼의 최근 기억1개 북마크 후보 제시, 오린 밤1회), observe(10초, observatory, 예보 표시 플래그, 일룬 밤1회), broker(8초, bench, 달성 가능한 의뢰 후보1개 추가, 토비 밤1회). 중재·정화·수리 성공은 수행자와 도움받은 영혼에게 helped 기억을 남긴다. actor가 시설일 경우 해당 시설의 소유 영혼, 없으면 수행자만 기록한다. 동일 밤 같은 도움 기억은 병합한다.

모든 영혼은 wander/rest/socialize를 수행할 수 있다. 역할 행동이 가능한 대상이 없으면 공통 행동으로 돌아간다. 고유 행동은 역할 ID로 허용하고 표시 이름으로 분기하지 않는다.

### 관계·기억의 데이터 규칙

관계는 쌍 하나의 공통 affinity(-100~100)와 방향별 memory를 사용한다. 키는 정렬된 두 영혼 ID. 친구는 affinity≥35를 2밤 유지, 라이벌은 ≤-35를 2밤 유지. 친구 해제는 <20, 라이벌 해제는 >-20으로 히스테리시스를 둔다. 순간 점수 변화로 관계 아이콘이 깜빡이지 않게 한다.

가까운 묘비(거리≤2)는 “만날 기회”만 늘린다. 강제 우정 수치를 매초 주지 않는다. 긍정 교류 +3, 다툼 실패 -8, 함께 해결 +6, 선택형 화해 +10. 각 쌍의 1밤 누적 양수 변화 +12, 음수 -16 제한. 7밤 동안 교류가 없으면 새벽에 중립 방향으로 1만 감쇠; 장기 기억 자체는 지우지 않는다.

Memory는 사건 사실, 상대, 장소, 감정 강도, 발생일, 만료일, 관련 전승 태그를 저장한다. 보통 기억은 7밤 뒤 행동 가중치가 0이 되고 기록 요약에 남는다. 강한 기억은 고유 플래그 3개까지 유지하며 “배신 기억→도움 제공→관계 회복” 같은 연쇄의 조건이다. 동일 사건 반복으로 기억을 무한 증식하지 않는다. 같은 (kind, 대상, 구역) 키는 최근 일시·횟수로 병합한다.

## 09 사건 Director와 플레이어 개입

사건은 원인이 있는 규칙이다. 조건이 없는데 흥미를 위해 재난을 강제로 만들지 않는다. 모든 위기는 8초 이상 전조, 원인 1~3개, 최소 하나의 무료 또는 재생 가능한 대응 수단을 제공한다.

Director는 tick 100 이후 50tick마다 후보 평가. 동시 위기 최대 2개(P0 1개), 중대 위기 최대 1개, 새 위기 간격 200tick 이상, 한밤 위기 총 3개 이하. 첫날은 스크립트 튜토리얼 1개만. 최근 2밤에 발생한 동일 template+동일 actor쌍은 제외하고, 최근 7밤 같은 template의 가중치는 1/(1+발생수)로 낮춘다.

risk_score R=clamp(base + 조건 보정 + floor(U/5),0,100). 전조 후보의 5초당 확률 p=R/100×0.25. 여러 후보가 통과하면 최종 가중치가 가장 높은 1개만 생성; 동점은 event RNG로 결정. 난수 호출은 정렬된 후보당 1회. 안전 배치에서 조건이 사라지면 후보 자체가 사라진다. UI는 이 R과 주요 원인을 보여주고 “발생 확률”이라고 잘못 표기하지 않는다.

상태: WARNING → ACTIVE → RESOLVED 또는 EXPIRED → ARCHIVED. 시간, 참여자, 대상 좌표, 해결 진척, effects_applied를 저장한다. 이벤트 시작/해결/정산은 idempotency key로 중복 효과를 막는다.

| ID | 조건·위험 초기값 | 전조/활성 | 성공 / 방치(새벽 미결 포함) |
| --- | --- | --- | --- |
| quarrel | affinity≤-20인 쌍이 거리≤3, base20; 소음 +15 | 8/15초 | 중재하면 관계+6,U-4 / 관계-8,U+8 |
| curse | 모라/연구 역할이 circle 작업, 기분<55, base15; circle+20, lantern-15 | 10/20초 | 정화하면 오염제거,U-5 / 최대 3시설 오염,U+10 |
| lost | 휴식 욕구≥75, 자기 묘비와 거리≥6, base20 | 10/20초 | 안내 시 기분+5 / 해당 영혼 이번 밤 휴식,U+4 |
| frenzy | 기분≤20 AND 오염 있음, base10 | 10/15초 | 봉인/종+중재 시 휴식 / 시설1개 damage+1,U+8 |
| intruder | 숲 해금, 입구 접근 가능, base15; 경비 -10 | 12/18초 | 가렌/ward 개입 시 손실0 / min(10,G) 손실,U+5 |
| missing | 탐험 성격, 미해결 탐험 기억, base10 | 12/20초 | 흔적2개 조사 시 복귀 / 휴식 보관함 1밤,U+3 |
| prank_chase | 피피/장난 성격과 성실 역할 근접, base15 | 8/15초 | 장난감으로 유도 시 관계+3 / 꽃밭1개 damage+1, 보물 플래그 |
| duet | 친구 쌍, stage, 양쪽 기분≥60 | 비위기 20초 | 공동 공연 기억, social 회복; 실패 벌점 없음 |
| bloom | 루나 또는 정원 역할, 정상 꽃밭, 기분≥60 | 비위기 15초 | 꽃 색 변이·기억; 수익 없음 |
| workshop_gift | 브람+친구, 수리 완료 기억 | 비위기 15초 | 장식 외형 해금; 반복 보상 없음 |
| reconciliation | 라이벌, 최근 도움 기억, bench | 선택 창 | 수락 관계+10, 거절 다음 밤 재제안 금지 |
| remembrance | 전승/이주 기록과 남은 친구 | 선택 창 | 추억 전시 선택; 벌점 없음 |

비위기 장면은 밤당 최대2개, 위기 슬롯을 쓰지 않으며 위기 발생을 위해 취소하지 않는다. 장면 도중 원인 데이터가 바뀌면 결과 전 재검증하고 무효 대상 효과를 건너뛰어 로그에 기록한다.

저주: 시작 시설에 curse=1. ACTIVE 중 5초마다 상하좌우 인접 건물 1개에 전파, source 포함 최대3개. 울타리 칸을 건너뛰지 못하고 조명 아래 대상은 전파 제외. curse는 기능 효율 50%, 새벽 자동 해제. 저주+damage 효율은 낮은 값 1개만 적용하여 25%로 곱하지 않는다.

| 개입 | 대상·효과 | 비용·쿨다운 |
| --- | --- | --- |
| 진정의 종 | bell_tower 기준 범위3, ACTIVE 진행 8초 정지 및 진척+1; quarrel은 진척1로 해결 | 무료, 25초. 최초 튜토리얼은 신목에서 1회 제공 |
| 영혼 봉인 | 영혼 또는 오염 시설1개, frenzy/curse 해결. 다른 사건은 진척1 | E8, 12초. 성공할 때만 차감 |
| 달빛 안내 | 영혼을 sanctuary/자기 묘비 접근 칸으로 이동, lost 해결·missing 흔적1 | 밤당2회, 무료 |
| 응급 휴식 | 사건 참여 영혼을 이번 밤 퇴근, 그 영혼 사건 종료. 생산 중단, 관계 보상 없음 | 밤당1회, 무료, 모든 단계에서 사용 가능 |

종은 WARNING 타이머를 정지하지 않고 조건 재검증을 유발한다. 봉인·안내·휴식으로 원인이 사라지면 전조 취소, 해결 보상은 없다. 봉인은 영혼을 영구 삭제하지 않는다. 침입자는 경비가 없으면 응급 휴식 대신 “문 닫기”를 같은 무료 응급 슬롯으로 사용하여 손실0, 그 밤 의뢰 수입 -5G까지만 허용한다.

사건 손실 합은 밤당 G20, damage 총3단계, U+25로 제한. 영혼 실종/폭주로 영구 영혼 손실 없음. 손실 cap에 막힌 사건은 추가 손실 대신 휴식 연출로 끝난다. 보상을 얻으려고 같은 소동을 반복하는 전략이 경제 최적해가 되지 않도록 위기 해결 골드 보상은 0이다.

## 10 경제·성장·복구

새벽 정산은 다음 순서로 1회만 실행: 사건 종료 → 순수입 산출 → 비용/손실 적용 → 자원 clamp → U 갱신 → 관계·의뢰·해금 → report 생성 → 자동 저장.

G_income = 12 + 4×active_ghosts + floor(mean_mood/10) + contract_reward(밤당≤15).
E_income = 2 + floor(happy_ghosts/2) + completed_research_energy(밤당≤8).
happy는 기분≥60이고 봉인·휴식 처리되지 않은 활성 영혼. active_ghosts는 밤 시작에 거주한 영혼 중 응급 휴식/실종으로 종료한 영혼을 제외한 수. 밤 1500tick 이상 활동했으면 끝부분 귀환·정상 휴식은 제외하지 않는다.

U_dawn = clamp(U_end - 8 - 2×resolved_crises + floor(unhappy_ghosts/2),0,100). unhappy는 기분<35. 사건 중 U 변화는 즉시 반영하되 새벽 공식을 다시 적용한다. 매밤 자동 평온 회복을 둬 연쇄 파산을 막는다.

P0 기본 사례: 3명, 평균 기분60, 행복2명, 연구2회, 사건/의뢰 없음이면 G=30, E=7. 낮 시작 G180에서 꽃밭20·등불20을 사면 G140, 새벽 G170/E37. 이 수치는 경제 smoke test의 oracle이다.

- G는 0~999999; 상한 이상은 표시만 “풍족함”, 저장 값은 상한 고정. E는 100을 넘는 부분을 폐기하되 보고서에 넘침 표시. U는 100을 넘지 않는다.
- 영입 비용은 신규 20G, 고유 이야기 영혼은 목표 완료 후 0G. 확장 비용 이외의 무한 성장 과세나 자동 소모 없음.
- 다음 낮 G<30이면 구호금으로 30까지 보충, E<10이면 10까지 보충, U>70이면 안전 예보와 무료 구역 정화 제시. 구호금은 다음 날에도 이용 가능하지만 총자산 증식을 위한 판매-구호 악용은 “현금+판매가능자산 환급액<30”일 때만 지급.
- 자금 부족 시 무료 이동, 정책 변경, 응급 휴식, 영혼 휴식, 기본 종으로 회복 가능해야 한다. 진행 필수 의뢰는 기부나 유료 시설만 요구하지 않는다.
- 반복 유지 작업은 영혼 역할·일괄 명령으로 처리한다. 금화 수거를 맵 클릭으로 시키지 않는다.
- 성장의 핵심은 새 기능·새 조합·외형·운영 목표다. 끝없는 레벨업, 유료 뽑기, 확률 강화 파괴는 없다.

## 11 엔딩 이후에도 선택이 생기는 지속 세계

이야기 첫 장은 “오래된 신목의 기억을 되찾기”다. 주요 목표: 첫 친구 관계 → 다툼을 배치로 예방 → 2구역 해금 → 공동체 축제 개최. 마지막 장면은 챕터 완료이며 크레딧을 닫으면 같은 낮으로 돌아온다. 이후 경제·AI·계절·의뢰·이주·기억은 계속 작동한다. end_game 플래그로 시뮬레이션을 멈추지 않는다.

장기 재미는 단순한 보상 반복이 아닌 아래 선택의 조합으로 만든다:

| 장치 | 바뀌는 것 | 플레이어의 결정 | 피해야 할 구현 |
| --- | --- | --- | --- |
| 관계의 역사 | 친구·라이벌·빚진 도움·공동 기억 | 관계 회복 또는 구역 분리 | 매일 관계 초기화 |
| 영혼의 관심사 | 7~14밤 지속하는 1개 목표 | 환경 지원, 보류, 대안 선택 | 성격이 무작위로 뒤집힘 |
| 계절 | 시설/행동 조건의 상대 가치 | 같은 묘지에서 운영 방침 조정 | 효율만 낮춰 파밍 강요 |
| 구역 정책 | 소음·생산·사회성의 상충 | 구역별 역할 분담 | 모든 정책 중첩 |
| 선택형 이주·전승 | 이웃 구성과 기억의 계승 | 보내기, 머물기, 휴식, 복귀 | 정든 영혼 강제 삭제 |
| 생성 의뢰 | 현재 세계 문제에 맞는 목적 | 3개 중 수락/거절 | 아무 세계나 같은 수집 과제 |
| 자기표현 | 정원·추모·관계 지도 | 자신만의 묘지 완성 | 점수 최적화만 의미 있음 |

무한한 고유 서사를 자동으로 보장하는 것이 아니라, 유한하고 검증된 규칙이 서로 다른 상태·선택·결과를 만드는 구조다. 모든 규칙을 파악한 숙련자에게도 서로 충돌하는 목표와 자기표현의 이유가 남아야 한다. 더 큰 숫자나 이름만 바뀐 텍스트는 새로운 콘텐츠로 세지 않는다.

### 계절

1계절=게임 내 14밤, 4계절 순환. 3밤 전 예고. 현실 날짜와 연결하지 않는다. 모든 난수·예보는 저장된다. 안전한 기본 배치가 계절 변경만으로 갑자기 붕괴하지 않는다.

| 계절 | 효과 | 새로운 선택 |
| --- | --- | --- |
| 꽃비 | 꽃 선호 +3, 정원 행동 점수 +8 | 정원 중심 교류 또는 다른 욕구 지원 |
| 반딧불 | 조명 위험 감소 효과 +5, 공연 소음 범위 +1 | 안전 축제와 조용한 구역 분리 |
| 안개 | 경로 이동 tick +1(최대6), 안내 사용 +1 | 시설을 집중할지 고독 구역을 유지할지 |
| 긴달 | 연구 E 밤 cap +2, 저주 R+5 | 에너지와 안정의 선택 |

한밤에는 계절 보정 1개와 지역 특성 1개만 적용. 효과가 표시되지 않는 숨은 난이도 상승은 없다. 평온 모드는 위기 R×0.5, 손실 cap×0.5이고 장식·이야기 보상을 제한하지 않는다.

### 구역 정책

구역마다 동시에 1개. 낮 무료 변경, 변경 후 2밤은 고정; 처음 선택 전에는 균형. 정책 선택 전에 유리·불리한 영혼 표시.

| 정책 | 이득 | 비용/상충 |
| --- | --- | --- |
| 균형 | 보정 없음 | 없음 |
| 고요한 정원 | rest 행동 +12, 고독 기분+5 | socialize 점수-8 |
| 어울림 | socialize +12, 관계 양수 cap+3 | 고독 기분-5, 소음 위험+5 |
| 연구 우선 | research +12, 구역 연구 수입+2(전체 cap 안) | curse R+10 |
| 순찰 강화 | 위기 R-10 | 연구 점수-10, 가렌 교류시간 감소 |
| 자유 탐험 | investigate/play +12 | lost R+10 |

정책은 순서·행동 기회를 바꾼다. 시스템이 플레이어를 추적해 현재 최선의 정책에 벌점을 주는 방식은 쓰지 않는다.

### 관심사·이주·전승

관심사: 친구 만들기, 조용히 쉬기, 공동 작업, 정원 조성, 탐험, 화해 중 1개. 최근 실패 원인/성격과 맞는 후보만 선택, 동일 관심사 2회 연속 금지. 모든 조건이 이미 만족되면 관심사 없음도 허용한다. 완료 시 기억/외형/관계 장면을 주고 반복 G 보상 없음.

매 7밤 3명의 방문 후보 제안. 후보 roster와 seed는 새벽 저장 시 확정하며 리로드로 재추첨하지 않는다. 기존 고유 영혼은 중복 생성하지 않는다. 플레이어는 전부 거절 가능, 진행 페널티 없음. 활성 영혼 최대24, 방문 후보3, 휴식 영혼 최대200; 한도를 앞두면 영입을 멈추고 보관 선택을 안내한다.

영혼이 관심사 3개를 마치면 전승 의향을 제시할 수 있다. 수락 시 휴식 명부에 기억과 관계를 보존하고, 기념물에 기억 태그1개를 남기며 활성 자리를 비운다. 전승은 능력치 보너스가 아닌 새로운 연쇄의 조건이다. 언제든 낮에 같은 영혼을 무료 복귀시킬 수 있다. 거절하면 최소14밤 재질문하지 않는다. 새 세대 영혼은 전승 태그와 접점을 가지지만 같은 대사·사건을 강제로 반복하지 않는다.

### 생성 의뢰와 반복 방지

템플릿 8개: 화해 중재, 고독 쉼터, 합동 공연, 오염 예방, 새 이웃 환영, 공동 수리, 기억 전시, 안전한 탐험. 매 3밤 3개 후보. 진행 중 최대2개. 만료·거절 벌점 없음, 의뢰 변경도 무료다.

계획기는 세계 상태에서 달성 가능한 목표만 바인딩한다. 예: “관계≤-20인 A/B를 3밤 내 같은 구역에서 긍정 교류2회”는 실제 라이벌·사용 가능 벤치가 있고 해당 정책으로 해결 가능할 때만 제안. 기간은 강제 실패가 아니라 제안된 도전 기준이며 지나면 보상은 유지하고 “여유롭게 진행”으로 전환한다.

목표 종류는 상태 달성, 기간 유지, 인과 연쇄 세 종류. 숫자 x회 수집만 반복하지 않는다. 최근 7밤 같은 (목표 동사, actor 역할쌍, 구역 정책, 결과 유형) signature를 거른다. 후보가 부족하면 적게 제안하거나 쉬어 가는 날로 표시하고 쿨다운을 몰래 무시하지 않는다.

보상은 최초 도감/외형 + G5~15(밤 cap15). 매일 의뢰를 안 해도 성장이 가능하다. 온라인 접속 일수·실제 시계·확률 희귀 드롭을 조건으로 삼지 않는다.


## 12 업데이트 가능한 콘텐츠 구조

콘텐츠 팩은 “데이터만 바꿔 영혼·시설·의뢰·사건·관계 장면을 추가하는 표준 단위”다. 기존 동사로 표현되는 팩은 시뮬레이션 코드 수정 없이 추가할 수 있어야 한다. 순간이동 같은 새 동사·완전히 새로운 규칙은 엔진 코드를 확장한 뒤 capability 버전을 올린다. 데이터만으로 모든 미래의 규칙을 표현할 수 있다고 주장하지 않는다.

| 계층 | 역할 | 변경 시 검증 |
| --- | --- | --- |
| Simulation kernel | 시간, 자원, 이동, 관계, 이벤트 트랜잭션 | 회귀·리플레이·세이브 |
| Rule primitives | 조건·효과·목표 동사, AI action | 버전 명시, 기존 팩 영향 |
| Content packs | 캐릭터·시설·사건·대사·규칙 조합 | schema·참조·실행 가능성 |
| Presentation packs | 스프라이트·오디오·번역·아이콘 | 경로·크기·누락·가독성 |
| Save instances | 개인 세계, 선택, 기억, 진행 사건 | 마이그레이션·롤백 |

### 폴더와 pack manifest

기본 팩은 res://content/core, 확장은 res://content/packs/<pack_id>, 로컬 데이터 모드는 user://mods/<pack_id>. P2는 기본 팩과 시험 확장 팩 1개를 실제 로드해야 한다. 외부 Workshop 연결 자체는 후속 기능이다.

```json
{
  "pack_id": "moonletters",
  "version": "1.0.0",
  "schema_version": 1,
  "min_game_version": "0.3.0",
  "requires_capabilities": ["relation.memory.v1", "events.choice.v1"],
  "dependencies": [{"pack_id": "core", "min_version": "1.0.0"}],
  "definitions": ["ghosts.json", "buildings.json", "events.json", "contracts.json"],
  "localization": ["ko.json", "en.json"],
  "asset_manifest": "assets.json"
}
```

정의 ID는 pack_id:local_id, 저장 인스턴스는 g000001/b000001/e000001처럼 단조 증가 ID. 표시 이름·배열 인덱스·파일 경로를 영구 ID로 쓰지 않는다. 제거한 ID를 다른 뜻으로 재사용하지 않는다.

로드 순서: manifest 읽기 → dependency 위상 정렬 → schema 검증 → ID 등록 → 참조 검증 → 규칙/자산 검증 → immutable registry 생성 → 전체 성공 시 활성화. 동일 ID 충돌은 두 팩 중 나중 것을 조용히 채택하지 않고 오류로 실패한다. 정의 전체 덮어쓰기와 자동 load order 마법은 P2에서 허용하지 않는다. 공식 수정은 같은 팩 버전 갱신 + 명시적 migration으로 처리한다.

커뮤니티 데이터는 JSON과 이미지·오디오만. 실행 코드, .gd/.dll, 임의 .pck, 임의 파일 접근을 로드하지 않는다. 경로는 팩 루트 내부로 정규화하고 ../ 및 절대 경로를 거절한다. 팩당 기본 제한 20MB JSON, 정의 5000개, 규칙 깊이8, 조건64개, 한 사건 효과32개. 이는 안정성 한계이며 검증 후 조정 가능하다. “무제한 확장”은 콘텐츠 총종류가 코드의 고정 배열에 묶이지 않는다는 뜻이고, 한 번에 무한한 엔티티를 실행한다는 뜻은 아니다.

### 조건·효과·목표 동사 계약

조건 AST는 all/any/not 및 leaf만 허용한다. leaf는 op, field, comparison, value, subject를 가진다. 비교는 eq/ne/lt/lte/gt/gte/in. 문자열을 코드로 eval하지 않는다.

v1 조건 op: stat, trait_has, tag_has, distance_lte, relation_lte, relation_gte, building_near, memory_has, phase_is, season_is, cooldown_ready, capacity_available, path_exists, flag_is.
v1 효과 op: add_resource, add_mood, add_relation, add_memory, add_damage, add_curse, clear_curse, set_flag, unlock_def, start_scene, send_to_rest, add_contract_progress.
v1 목표 op: reach_state, maintain_state, perform_action, complete_chain.

stat 허용 field: mood, need.social/rest/purpose/play, unrest, gold, energy, damage, affinity. subject는 world 또는 bound role. tag/trait/flag/definition ID는 registry에 존재해야 한다. enum 오타와 미등록 필드를 에러로 처리한다. add_resource에는 gold/energy/unrest만 허용하고 모든 경제 cap를 우회할 수 없다.

사건 템플릿은 역할 selector → 조건 → 가중치 → 전조/활성 단계 → 선택지/해결 조건 → 결과 효과 → 기억 → 반복 signature를 가진다. 각 단계에 timeout_ticks와 fallback을 지정한다. 동일 선택지는 event_instance_id+choice_id로 한 번만 실행된다. 효과 적용 전 대상·비용 검증이 모두 끝나야 하며 일부 효과만 적용하고 비용 부족으로 실패하는 흐름은 금지한다.

```json
{
  "id": "moonletters:old_letter",
  "kind": "opportunity",
  "roles": {
    "sender": {"traits_any": ["core:quiet"], "active": true},
    "receiver": {"distinct_from": "sender", "relation_to": "sender", "min_affinity": 20}
  },
  "when": {"all": [
    {"op": "phase_is", "value": "NIGHT"},
    {"op": "building_near", "subject": "sender", "def_id": "core:archive", "radius": 3}
  ]},
  "weight": 20,
  "warning_ticks": 80,
  "active_ticks": 200,
  "cooldown_nights": 7,
  "choices": [
    {"id": "share", "label_key": "moonletters.share", "cost": {"energy": 0},
     "effects": [
       {"op": "add_relation", "a": "sender", "b": "receiver", "value": 4},
       {"op": "add_memory", "subject": "sender", "kind": "moonletters:shared_letter", "other": "receiver", "strength": 10, "duration_nights": 7}
     ]},
    {"id": "keep", "label_key": "moonletters.keep", "cost": {"energy": 0},
     "effects": [{"op": "add_mood", "subject": "sender", "value": 3}]}
  ],
  "timeout_effects": [],
  "dawn_effects": [],
  "repeat_signature": ["id", "sender", "receiver"]
}
```

위 예제의 role selector는 active, traits_any, distinct_from, relation_to, min_affinity를 지원해야 한다. P0에서는 필요한 subset으로 시작하되 P2에서 전체 지원한다. 기억 kind ID와 locale key도 해당 팩에서 정의해야 로드된다. 빈 효과 배열은 의도적인 무벌점 종료다.

### 콘텐츠 추가 완료 정의

새 팩 하나를 추가하는 절차: manifest 및 정의 작성 → 번역·자산 등록 → validate_content → 샘플 seed 20개×30밤 시뮬레이션 → 이전 세이브 3종에 추가 → 낮에서 활성화 → 플레이로 원인과 선택 확인 → 버전/변경 기록 발행.

각 신규 팩은 **기존 콘텐츠와 연결되는 새 상충 관계 2개 이상**, 새로운 결정 유형 1개 이상, 추가·미설치·비활성화·이전버전 세이브 검사를 포함한다. 새 영혼 10명에 기존 행동 이름만 바꾸는 업데이트는 이 기준을 충족하지 못한다. “꽃배달 팩”이라면 꽃 선호 캐릭터를 추가하는 것에 더해 이웃 간 선물 전달과 고독의 충돌 같은 새로운 배치 선택을 증명해야 한다.

모든 팩은 기본 게임의 기존 세이브에서 이용 가능해야 한다. 새 게임을 강요하지 않는다. 신규 구역이 공간을 요구하면 기존 맵 바깥 새 region instance를 생성하되 기존 좌표를 덮어쓰지 않는다. 1.0의 동시 활성 맵 범위는 48×32로 제한하고, 더 큰 월드/다중 맵은 새 기능 버전에서 추가한다.

### 버전 호환과 안전한 추가·제거

세이브는 사용한 pack_id/version/content_hash와 전체 manifest를 저장한다. 공식 업데이트 추가 팩은 기본적으로 다음 낮에 적용 가능한 목록으로 표시하고 사용자가 한 번에 적용할 수 있다. 진행 중 밤은 기존 registry snapshot으로 끝낸다. 현재 밤의 이벤트 정의가 중간에 바뀌지 않는다.

삭제된 정의는 tombstone mapping 또는 placeholder로 보존한다. 미설치 영혼은 휴식 명부에 원본 데이터를 보관, 미설치 시설은 위치를 차지하는 “보관된 시설”로 표시하고 무료 이동/보관 가능. 유료 구입 비용은 기록한다. 비활성화한 콘텐츠를 다시 켜면 동일 인스턴스 ID로 복구한다. 존재하지 않는 이벤트는 fallback 결과(추가 손실0)로 닫고 기록한다. 원본 세이브를 자동 덮어쓰지 않고 migration 사본을 쓴다.

신규 major schema는 순차 migration(v1→v2→v3)과 golden save fixture가 있어야 한다. 이전 버전으로 되돌리는 역변환은 약속하지 않고 업데이트 전 백업 복원을 제공한다. 핵심 코드에서 “세이브가 오래되면 새 게임” 처리 금지.

## 13 데이터 구조와 저장 계약

모델 단위는 GDScript RefCounted 또는 typed model class, 정의는 커스텀 Resource로 표현한다. 외부 JSON은 검증 후 같은 Resource 형태로 변환한다. SceneTree가 없어도 시뮬레이션을 실행할 수 있게 한다.

| 모델 | 필수 필드 |
| --- | --- |
| GhostDef | id, name_key, role_id, traits[2], sociality, activity, preferences[], action_weights, sprite_id, unique |
| GhostState | id, def_id, grave_id, cell, move_from/to, move_progress_tick, mood, needs, ai_state, action_id/target_id, action_remaining, cooldowns, interest, status_flags |
| BuildingDef | id, footprint, access_offsets, walkable, costs, radius, tags, effects, upgrade, capacity |
| BuildingState | id, def_id, cell, rotation, level, damage, curse, paid_cost, reserved_slots, stored |
| RelationState | pair_key, a_id, b_id, affinity, status, threshold_nights, last_contact_day, positive/negative_night_delta |
| MemoryState | id, owner_id, kind, other_id, region_id, first_day, last_day, count, strength, expires_day, permanent |
| EventDef | id, kind, selectors, conditions, risk_terms, stages, outcomes, choices, cooldown, repeat_signature |
| EventState | id, def_id, bound_roles, phase, started_tick, remaining_ticks, progress, pause_until_tick, applied_effect_keys |
| RegionState | id, bounds, unlocked, policy_id, policy_lock_until_day, traits |
| ContractState | id, template_id, bound_roles, goal_ast, progress, accepted_day, suggested_due_day, reward_claimed |
| CampaignState | day, phase, tick, season_index, story_flags, interest_schedule, visitor_roster, unlocked_defs, next_ids |
| EconomyLedger | opening, income_by_source, expenses_by_source, losses_by_source, capped_deltas, settlement_id |
| ContentManifest | game_version, schema_version, pack_versions, hashes, capabilities |
| WorldState | 全모델集合, resources, rng_streams, command_seq, pending_commands, settings-independent simulation data |

사회성·활동성은 0~100 정의값. 사회성은 social 욕구 증가량을 base2+floor(sociality/50)로 조정, 활동성은 rest 증가량을 base2+floor(activity/50)로 조정한다. 이 보정 외에 몰래 이동 속도나 수입에 연결하지 않는다. P0 공통50, 고독 영혼20/40, 사교 영혼80/60을 초기값으로 쓴다. 시작 관계는 garen/mora=-25, 나머지0. 기분 fixture는 경제 검증을 위해 테스트에서만60으로 통일한다.

```json
{
  "schema_version": 1,
  "game_version": "0.1.0",
  "slot_id": "slot-01",
  "save_revision": 3,
  "content": [{"pack_id":"core","version":"1.0.0","hash":"<sha256>"}],
  "world": {
    "seed": "20261123",
    "day": 4,
    "phase": "NIGHT",
    "tick": 640,
    "resources": {"gold":170,"energy":37,"unrest":12},
    "next_ids": {"ghost":4,"building":10,"event":8,"memory":24,"contract":2},
    "rng": {"ai":{"seed":"<int64>","state":"<int64>"},"events":{"seed":"<int64>","state":"<int64>"},"visitors":{"seed":"<int64>","state":"<int64>"}},
    "ghosts": [],
    "buildings": [],
    "relations": [],
    "memories": [],
    "events": [],
    "regions": [],
    "contracts": [],
    "flags": {},
    "ledger": {},
    "command_seq": 42
  },
  "checksum": "<sha256-of-canonical-world-and-content>"
}
```

위는 envelope 예시이며 빈 배열은 축약이다. 실제 저장은 위 필수 필드를 빠짐없이 포함한다. 새 게임 초기화 fixture와 완성된 중간 밤 fixture를 저장소에 제공한다.

RNG는 ai/events/visitors/worldgen/cosmetic 스트림 분리. seed는 campaign seed+stream name의 SHA-256 첫8바이트를 signed little endian int64로 해석. Godot RandomNumberGenerator의 seed/state는 JSON 정밀도 손실 방지를 위해 10진 문자열로 저장한다. cosmetic RNG는 게임 상태에 영향 없음. 동일 엔진·팩·seed·명령열에서 같은 결과가 나와야 한다. 엔진 버전을 넘는 바이트 단위 리플레이 일치는 요구하지 않는다.

저장 위치 user://saves/<slot>/save.json, backup1.json~backup3.json, 별도 user://settings.cfg. 슬롯3개, 자동 저장3회 순환, 사용자가 만든 수동 슬롯은 자동 삭제 금지. 저장은 tick 경계에서 immutable snapshot → save.tmp 기록 → flush/close → JSON·checksum 검증(객체 키 재귀 사전순, 배열 순서 유지, UTF-8·LF 정규화) → 기존 save를 backup으로 이동 → 임시 파일을 교체. 교체 실패 시 기존/백업을 보존하고 실패 알림. 초기 P0는 1슬롯+백업으로 시작 가능.

새벽 ledger에 settlement_id=day:<day>와 settled=true를 저장하므로 보고서 재열기/로드로 중복 수익이 발생하지 않는다. UI 패널 위치/배속은 설정, 일시정지 여부는 로드 직후 항상 정지. 오프라인 경과 시간을 계산하지 않는다.

## 14 Godot 프로젝트 아키텍처

기준 엔진은 **Godot 4.5.1 stable, Standard, typed GDScript, Compatibility renderer**로 고정한다. 이는 최신 버전이라는 주장이 아니라 재현 가능한 구현 기준이다. 시작 저장소에 다른 안정화된 Godot 4 버전이 이미 고정되어 있으면 그 버전을 유지하고 호환성 차이를 기록한다. 버전을 바꿀 때 저장·리플레이·렌더 테스트를 다시 수행한다.

```text
res://
  project.godot
  scenes/main/{boot,main_menu,game_world}.tscn
  scenes/world/{ghost_actor,building_view}.tscn
  scenes/ui/{hud,build_panel,ghost_panel,event_panel,dawn_report}.tscn
  scripts/core/{game_session,sim_clock,command_bus,save_service}.gd
  scripts/model/{world_state,ghost_state,building_state,event_state}.gd
  scripts/sim/{simulation,placement_system,path_service,ghost_ai}.gd
  scripts/sim/{relation_system,event_director,economy_system}.gd
  scripts/sim/{memory_system,season_system,contract_planner}.gd
  scripts/content/{content_registry,pack_loader,rule_interpreter,validator}.gd
  scripts/presentation/{world_presenter,ghost_presenter,audio_presenter}.gd
  scripts/ui/{hud_presenter,inspection_presenter}.gd
  scripts/platform/{platform_adapter,steam_adapter_stub}.gd
  content/core/{manifest,ghosts,buildings,events,contracts}.json
  content/core/locales/{ko,en}.json
  content/packs/moonletters/...
  assets/{tiles,ghosts,ui,audio,fonts}/
  tests/{fixtures,unit,integration,soak}/
  tools/{validate_content,run_tests,simulate_campaign}.gd
  docs/{IMPLEMENTATION_STATUS,DECISIONS,TEST_REPORT,CONTENT_AUTHORING}.md
```

Autoload는 AppSettings, SaveService, ContentRegistry 세 개로 제한. 실제 세계 상태는 GameSession 1개가 소유한다. 씬 전환 시 유령 노드를 전역으로 남기지 않는다. 모든 UI는 CommandBus로 명령을 제출하고 simulation event로 결과를 받는다.

```mermaid
flowchart LR
 UI[입력과 UI] --> CMD[CommandBus 검증]
 CMD --> SIM[Simulation tick]
 REG[검증된 ContentRegistry] --> SIM
 SIM --> STATE[WorldState]
 STATE --> VIEW[Presenter와 Godot 씬]
 STATE --> SAVE[SaveService snapshot]
 SIM --> LOG[사건과 원인 로그]
 LOG --> UI
```

### 시스템 계약

| 시스템 | 입력 → 출력 | 금지 |
| --- | --- | --- |
| PlacementSystem | Place/Move/RemoveCommand + state → Result + changes | UI가 직접 자원 차감 |
| PathService | walkability_revision, from,to → path cells | 매 프레임 전체 맵 재생성 |
| GhostAI | state snapshot, defs → ActionIntent | SceneTree 탐색으로 타깃 선택 |
| EventDirector | state + counters + event RNG → EventInstance | 무조건 random 재난 생성 |
| RuleInterpreter | typed AST + bound roles → validated effect list | 문자열 코드 실행 |
| EconomySystem | night ledger → DawnSettlement | 보고서 UI에서 정산 |
| ContractPlanner | world facts + recent signatures → eligible offers | 불가능 목표 또는 과금 의존 목표 |
| ContentRegistry | validated pack set → immutable definitions | 정의에 런타임 기분 쓰기 |
| Presenter | diff/events → sprite/UI/audio | 애니메이션 끝나야 수익 지급 |

필수 Command 타입: PlaceBuilding, MoveBuilding, RemoveBuilding, UpgradeBuilding, RecruitGhost, RestGhost, RecallGhost, ChangePolicy, StartNight, UseIntervention, ChooseEventOption, AcceptContract, SetSpeed, Pause, Save.
모든 명령은 command_id, issued_tick, actor(항상 player), payload를 가진다. 결과는 accepted, error_code, resource_delta, changed_ids. 재전송된 command_id는 캐시된 결과 반환, 부작용 반복 금지.

필수 signal/domain event: phase_changed, building_changed, ghost_action_changed, mood_reason_added, relationship_changed, warning_started, crisis_resolved, resource_changed, dawn_settled, content_registry_changed, save_failed. Godot signal 연결은 연결자 책임으로 해제하고 같은 씬 재진입으로 중복 연결되지 않도록 한다.

tick 순서 고정: 명령 처리 → 시간/쿨다운 → 이동 → 완료 action 효과 → 욕구/기분 → 관계/기억 → 사건 상태 진행 → Director 신규 후보 → ledger/dawn transition → view diff. pause 중 개입 명령은 현재 tick의 명령 큐에 저장하고, 재개 시 첫 tick에서 순서대로 검증·실행한다. UI는 예약 표시를 하며 아직 비용을 차감하지 않는다. 순수 UI 명령(Pause/SetSpeed/Save)은 즉시 처리한다. 프레임이 느려도 tick을 버리지 않으며 한 프레임 최대10tick 처리 후 남은 시간을 다음 프레임으로 이월한다. 포커스 상실/절전 중 실제 경과 시간을 따라잡지 않는다. 같은 tick 내 이벤트는 (priority, instance_id) 정렬. 효과 큐는 현재 tick 끝에 한 번 적용하고 재귀 이벤트 생성은 다음 Director 평가로 미룬다.

### Godot 구현 세부

맵 표현은 Ground/Decoration/Overlay의 TileMapLayer. 논리 점유는 별도 배열이 권위 데이터다. AStarGrid2D는 diagonal_mode=DIAGONAL_MODE_NEVER, Manhattan 휴리스틱. region/cell_size 변경 후 update(), 이후 건물 solid 상태 재적용. 경로 조회에서 막힌 건물 자체가 아니라 접근 가능한 access cell을 목표로 한다. 길 보너스 cost는 0.8, 흙1.0이며 실제 이동 시간도 맞춘다.

유령 view는 Node2D+AnimatedSprite2D 기본, 물리 충돌이 필요할 때만 CharacterBody2D. 이동 논리는 tick 단위이며 렌더는 cell 사이 보간만 수행한다. 애니메이션 종료 signal이 simulation 상태를 바꾸지 않게 한다. y-sort 기준은 발바닥.

공간 쿼리는 구역/셀 버킷, 정적 시설 영향은 배치 변화 때만 캐시 갱신. 전 영혼 쌍을 매 프레임 계산하지 않는다. 활성24명, 시설300개, 최대48×32에서 안정적으로 동작할 것을 목표로 한다. 게임 공간 확장은 저장 region ID와 별도로 처리하며 TileMap 좌표를 전체 세계 고유 ID로 사용하지 않는다.

## 15 UI·UX와 접근성

상단: 날짜·낮밤·G/E/U·다음 계절. 하단: 건설/영혼/관계/의뢰/기록. 우측: 선택 정보. 중앙 맵을 최대한 확보. 기본 상태에서 불필요한 패널로 월드의 35% 이상을 가리지 않는다.

| 화면 | 필수 내용 |
| --- | --- |
| 건설 | 비용, 설치 가능 이유, 범위, 접근성, 영향을 받는 영혼, 취소 |
| 영혼 | 현재 행동, 2개 주요 욕구, 기분 원인 상위3개, 관계, 관심사, 기억 |
| 관계 지도 | 친구/라이벌/기억 연결, 필터와 선택 시 해당 영혼으로 이동 |
| 밤 예측 | 후보 사건, R 점수, 원인, 현재 해결 수단; 확정 발생으로 표기하지 않음 |
| 사건 알림 | 위치, 전조/마감, 이유, 가능한 대응·비용·쿨다운 |
| 새벽 | 어젯밤 핵심3개, 상세 ledger, 배치 개선 제안1개, 새 해금 |
| 장기 일지 | 계절·정책·기억·의뢰·전승의 이력. 같은 기록 묶기 |
| 팩 관리 | 설치/활성/버전/필수 의존성/오류/다음 낮 적용 여부 |
| 저장 | 슬롯 날짜·영혼 수·팩 버전, 백업 복구 미리보기 |

입력 기본: 좌클릭 선택/설치, 우클릭 취소, R 회전, Space 일시정지, 1/2 배속, B 건설, Tab 선택순환, Esc 메뉴/취소, Ctrl+Z 낮 실행취소, 휠 줌, 중클릭 드래그. 모두 InputMap에서 재지정 가능. 키보드 포커스 이동/확정/취소 제공. 포커스를 잃으면 자동 일시정지(설정 가능).

세계는 640×360 논리 캔버스, 16px 타일과 24×32 유령. 월드는 정수 배율 우선·nearest 필터, UI는 별도 해상도 적응형 CanvasLayer로 그려 720p에서도 글자 최소16px 물리 크기를 유지한다. 한글 폰트는 적법한 배포 가능 폰트를 동봉, 라이선스 기록. UI 100/125/150%와 창모드·전체화면 제공. 작은 해상도에서는 우측 패널을 접을 수 있다.

낮 크림·녹색, 밤 남색·청록, 위기 보라. 위험은 색+아이콘+텍스트. 화면 흔들림/플래시 끄기, 볼륨 master/music/sfx, 음성 없는 정보의 텍스트 대체, 고대비 포커스, 자동정지 옵션. 슬픔·죽음 소재는 부드럽게 다루며 실제 묘비 이름 입력은 선택 기능이다.

튜토리얼은 3밤: 1) 꽃밭 이동과 루나 기분 원인, 2) 모라·등불·마법진 상충 및 무료 종, 3) 라이벌 배치 변경과 관계 결과. 실패로 진행이 잠기면 무료 재설정 또는 건너뛰기. 배치→이유→밤 결과를 사용자가 직접 연결할 수 있어야 한다.


## 16 콘텐츠 제작용 구체 목록

필수 성격 ID는 core:quiet, social, dutiful, playful, curious, gentle, proud, stubborn. 고독/사교만 상호 배타. 역할 ID는 knight, gardener, researcher, smith, trickster, archivist, musician, purifier, explorer, broker, teacher, astronomer. 정확한 trait 쌍: garen=dutiful+quiet, luna=gentle+curious, mora=proud+quiet, bram=stubborn+dutiful, pipi=playful+social, orin=quiet+curious, vela=social+gentle, moss=gentle+dutiful, neri=curious+playful, tovi=social+proud, sella=gentle+social, ilun=quiet+curious. 역할은 08절 고유 ID와 같은 행의 역할을 사용한다. 선호 tags는 flower, light, dark, quiet, social, workshop, magic, nature, stage, archive. dark는 lantern 영향 없음, quiet는 현재 소음 태그 없음으로 계산한다.

P2 관계 장면 16개는 같은 장면에서 반복 골드를 주지 않는다. 최초 실행은 플래그로 저장, 재감상은 일지에서 무료. 장면은 2~4줄 대사와 선택지1~2개로 구현한다.

| 장면 ID | 트리거 | 남기는 결과 |
| --- | --- | --- |
| first_neighbor | 첫 긍정 교류 | 첫 인사 memory |
| shared_bench | 같은 벤치에서 교류2회 | affinity+3, 벤치 기억 |
| knight_and_prankster | 가렌·피피 추격 뒤 도움 | 장난 약속 memory |
| quiet_apology | 라이벌이 고요 구역에서 화해 | affinity+10, 사과 memory |
| moon_garden | 루나·모라 꽃과 연구 협업 | 꽃 색 외형 |
| repaired_gift | 브람이 친구 시설 수리 | 선물 memory |
| witchs_rule | 모라 주변 소음 원인 해결 | 연구 협정 memory |
| night_duet | 공연·친구 관객 | 합주 memory |
| shared_watch | 가렌+새 이웃 순찰 | 믿음 memory |
| lost_and_found | 길 잃음 구조 | 방향별 도움 memory |
| archive_reading | 오린+친구 서고 | 옛 편지 memory |
| pond_silence | 고독 성격 두 명 연못 | 말 없는 우정 memory |
| new_arrival | 생성 영혼 영입 후 교류 | 환영 memory |
| reunion | 전승/휴식 영혼 복귀 | 재회 memory |
| farewell | 선택형 전승 수락 | 기념물 태그 |
| festival_memory | 지역축제 의뢰 완료 | 공동체 장면·장식 |

의뢰 predicate 기본값: 화해=지정쌍 긍정교류2회, 쉼터=고독 영혼 rest≤30·기분≥60인 밤2회, 공연=관객2명 이상 공연1회, 오염예방=연구3회와 저주피해0인 밤1회, 환영=신입이 서로 다른 영혼2명과 교류, 공동수리=브람 수리1회+다른 영혼 도움 기억1개, 기억전시=memorial에 이미 소유한 기억1개 지정, 탐험=조사2회와 실종 미결0인 밤1회. 의뢰 생성 시 대응 역할·시설이 없다면 해당 후보를 제시하지 않는다. 수리 의뢰를 위해 고장을 만들도록 유도하지 않으며 실제 손상 있을 때만 후보가 된다.

해금 순서: 첫밤 bench/flower/lantern/circle/bell, 3번째밤 workshop·브람, 친구1쌍 toy_box·피피, 5번째밤 sanctuary/path/fence, 구역2 archive/herb/관심사, 구역3 stage/pond/정책 전체, 구역4 observatory/ward/memorial/전승. 정상 진행에서 10~20밤 안에 모든 기능 축을 경험하도록 조정한다. 나머지 고유 영혼은 역할 관련 목표를 하나 마치면 방문 후보에 넣는다. 기능 해금을 보기 위해 100밤의 반복 수금을 요구하지 않는다.

## 17 반년 이상 다시 찾고 싶은 이유의 검증

“6개월”은 현실의 장기 사용 목표다. 180개의 게임 내 밤을 돌렸다고 180일 유지율을 달성했다고 말하지 않는다. 세션 20~40분, 주3회, 26주를 목표 사용 시나리오로 잡으면 약26~52시간이다. 열성 플레이어의 100시간 이상 활용은 별도 고급 목표이며 출시 전 보장하지 않는다.

구조적 목표: 새 팩 없이도 20시간 이후에 **서로 다른 운영 목표를 위해 기존 배치를 다시 생각할 이유**가 남는다. 1000밤 자동 시뮬레이션은 오류·패턴 편중을 찾는 수단이며 재미의 증거를 대신하지 않는다.

### 지루함과 반복 노동을 감지하는 지표

| 지표 | 측정 방식 | 초기 통과 목표 |
| --- | --- | --- |
| 의미 있는 선택 | 서로 다른 상태 결과를 갖는 배치/정책/관계 결정, 테스트 관찰자가 코딩 | 20분당 ≥2회, 클릭 수를 세지 않음 |
| 원인 이해 | 왜 사건이 났는지·어떻게 바꿀지 구두 질문 | 테스트 참여자 80%가 2사례 설명 |
| 자발적 재설계 | 퀘스트 지시 없이 다음날 변경 계획 말하기 | 30분 참가자 70% 이상 |
| 반복 노동 비중 | 수금·같은 수리 클릭·기계적 확인 시간/총 플레이 | 중앙값 15% 이하 |
| 후반 무의미한 밤 | 새 결정/관계 변화/목표 진행 없이 자동 진행한 밤 | 20~30시간 구간 연속3밤 이하 목표 |
| 전략 편중 | 안정/연구/관계/탐험 4정책 에이전트 및 사용자 로그 | 한 전략이 모든 지표를 지배하면 실패 |
| 사건 반복 | template+역할쌍+정책+결과 signature | 같은 signature 최근7밤 반복≤20%, 원인 없는 강제 다양화 금지 |
| 회복 가능성 | 자원0·U90·시설손상 fixture | 3밤 내 기능 정상화, 결제·리셋 없이 |
| 새 팩 호환 | 동일 오래된 세이브에 새 팩 적용 | 기존 영혼·시설·관계 손실0 |

표의 비율은 **개발용 목표값이며 실제 측정 결과가 아니다.** 평온하게 관찰하기를 원하는 플레이어에게 사건이나 결정을 억지로 늘리지 않는다. 후반 무의미 지표는 “관찰이 즐거웠는가” 질문과 함께 판단한다.

### 단계별 실험

1. P0 5명×30분: 꽃/등불 배치가 결과를 바꾸는지, 다음 배치를 스스로 구상하는지. 3명 이상이 재배치 이유를 못 말하면 콘텐츠를 늘리기 전에 원인 표시/상충 규칙 수정.
2. P1 8명×3회×40분: 두 계절·정책·기억 차이. 3번째 세션에서 작업으로 느끼는 부분을 관찰. 수리 클릭이 반복되면 자동화·일괄 처리 강화.
3. P2 12명×10~30시간: 동일 빌드, 추가 팩 없이 장기 테스트. 각자 안전·연구·관계·꾸미기 목표를 자유 선택. 후반 전체가 동일 배치에 수렴하면 공간·역할·목표 상충을 재설계.
4. 베타 4~8주: 자발적 재방문 이유, 잠시 쉬고 돌아온 이해도, 복귀 시 부담. 강제 출석 장치를 넣지 않는다.
5. 출시 후 동의한 사용자 표본으로 현실 30/90/180일 재방문과 이유 조사. 오프라인 게임은 플레이 추적을 기본 전송하지 않는다. 표본 이탈·테스트 보상 효과를 함께 보고.

장기 테스트를 못 끝냈다면 “기술 기준 통과, 장기 재미 검증 중”으로 표시한다. 사용자 요구를 만족했다고 선언하기 위해 시간을 부풀리거나 반복 작업량으로 100시간을 채우지 않는다.

### 모든 해금 후에도 남는 실제 시나리오

- 연구 효율이 충분한 묘지가 안개 계절에 들어서자 이동이 긴 배치에서 교류 기회가 줄어든다. 중앙집중은 고독 영혼에게 불편하므로 연구실 이동, 정책 변경, 일부 관심사 보류 중 선택한다.
- 오래된 친구 A가 새 이웃 B와 충돌한다. A의 과거 도움 기억으로 화해 경로가 열려 있지만 조용한 구역을 분리하면 갈등 없이 운영할 수도 있다. 어느 쪽도 필수 정답이 아니다.
- 모든 시설을 얻은 플레이어가 “연구를 유지하며 저주 없는 3밤”과 “24명이 각자 친구1명 가진 묘지” 중 자기 목표를 선택한다. 돈이 많아도 공간·주의·사회적 욕구가 제한이 된다.
- 전승한 영혼의 기념물과 새 영혼이 연결돼 기억 장면이 열린다. 기존 영혼 복귀와 새 영혼 유지 중 원하는 공동체를 선택한다.

위 시나리오는 자동 발생 보장이 아니라 조합의 검증 fixture로 만들어야 할 요구사항이다. 텍스트만 달라지고 실제 선택 결과가 같으면 미구현으로 판정한다.

## 18 아트·오디오 제작 계약

P0는 외부 라이선스 문제 없는 임시 도형/단순 스프라이트를 허용하되 “최종 아트”라고 보고하지 않는다. P1에서 영혼5명과 낮밤 전환을 대표 품질로 완성한다. P2에서 역할과 상태를 색 외 실루엣으로 구분한다.

타일 16×16, 기본 영혼24×32, 4방향. 영혼당 idle4frame, move4frame, act4frame, react2frame, wake/sleep 각4frame을 목표로 한다. 이동 프레임 반복은 방향별 mirror 가능하지만 장비 손잡이 등 비대칭 요소를 확인한다. 눈·모자·도구로 역할 구분, 바닥 접점과 선택 outline 공통 규격.

시설은 footprint와 시각적 높이를 분리하고 아래쪽 접근 영역이 가려지지 않게 한다. 정상/오염/손상/미작동 상태 최소4개. UI 아이콘24 또는32px, 한글 정보는 아이콘만으로 대체하지 않는다. 입자 수·광원은 low 옵션에서 줄여도 사건 판별 가능해야 한다.

오디오: 낮/밤/위기 layer 각각1개 이상, 기상·건설·취소·종·봉인·긍정관계·위기전조·새벽 cue. 같은 소리 동시 발음 최대4, 중요한 전조는 소리와 UI 모두 제공. 공포성 고음·큰 점프 음량 사용하지 않는다. assets/credits.json에 제작자·원본·라이선스·수정내역을 관리한다.

## 19 개발 순서와 작업 티켓

각 티켓은 작은 실행 가능한 증분으로 완료한다. 화면만 그려 놓고 기능을 “완료”라고 표시하지 않는다. 후속 티켓으로 미루는 항목은 IMPLEMENTATION_STATUS에 명시한다.

| 순서·티켓 | 구현 작업 | 선행 | 완료 증거 |
| --- | --- | --- | --- |
| M0/T01 | 엔진 고정, 폴더, boot, headless runner, 기본 fixture | 없음 | 새 게임 진입, 파서 오류0 |
| M0/T02 | typed WorldState, ContentRegistry, 6시설·3영혼 정의 validator | T01 | 잘못된ID/조건 거절 |
| M1/T03 | 점유·회전·범위·경로 검증·비용·undo | T02 | 배치 테스트와 플레이 영상 |
| M1/T04 | DAY/NIGHT/DAWN, tick·배속·pause | T01 | 1800tick 정확히1회 정산 |
| M2/T05 | AI, 시설 예약, 이동, 선호, 기분 설명 | T03,T04 | 동일seed 재현·성격 차이 |
| M2/T06 | 관계·친구/라이벌·가시적 교류 | T05 | cap와 히스테리시스 검사 |
| M3/T07 | Director, quarrel/curse, 전조·종·봉인 | T06 | 원인→대응→결과 통합 검증 |
| M3/T08 | 경제 ledger, 새벽, 구호, 저장·백업 | T04,T07 | mid-night restore와 중복정산0 |
| GATE A | P0 7밤과 5명 재미 검증 | T03~08 | 실패 시 핵심 규칙 수정 |
| M4/T09 | 브람·피피, 10시설, 6위기, 안내 | Gate A | 역할·위기별 회복 가능 |
| M4/T10 | 기억, 관심사, 정책3개, 계절2개 | T06,T08 | 30밤 변화·세이브 |
| M4/T11 | 팩 로더·AST·schema·버전·migration·시험팩 | T02,T08 | 코드 수정 없이 팩 확장 |
| GATE B | P1 3회 세션 테스트, 100seed×100밤 | T09~11 | 반복·편중·고장 보고 |
| M5/T12 | 4구역, 12고유+생성 영혼, 18시설, 12사건 | Gate B | 모든 ID·아트·텍스트 연결 |
| M5/T13 | 정책6·계절4·의뢰8·장면16·전승/복귀 | T10~12 | 후반 fixture와 무한 지속 |
| M6/T14 | 일지/관계지도/튜토리얼/접근성/ko,en | T12,T13 | 720p·150% UI 실사용 |
| M6/T15 | 성능·1000밤 soak·마이그레이션·Windows export | T08~14 | 아래 QA 기준 통과 |
| GATE C | 10~30시간 및 베타 장기 재미 검증 | T15 | 검증 단계·미해결 위험 공개 |
| M7/T16 | Steam용 패키지, Cloud 충돌·오프라인·상점 자료 | Gate C | 출시 후보 체크 완료 |

GATE는 실행 가능한 검증이다. 문서에 체크만 하는 것으로 통과시키지 않는다. 순서 변경은 의존성이 유지될 때 가능하며 이유를 DECISIONS에 적는다.

### 새 컴퓨터의 Codex가 따를 작업 지시

1. 이 페이지 전체를 읽고 현재 작업 폴더·AGENTS.md·프로젝트 존재 여부를 확인한다. 기존 프로젝트는 먼저 git status로 사용자 변경을 보존한다. 비어 있다면 wispwake-yard 새 폴더에 시작한다.
2. 엔진/내보내기 템플릿 버전과 실행 경로를 기록한다. Godot가 없으면 공식 설치 경로를 안내하거나 허용된 설치 수단을 사용한다. 웹 데모를 Godot 게임의 대체물로 만들지 않는다.
3. docs/GDD.md에 이 명세를 보존하고 REQUIREMENTS와 IMPLEMENTATION_STATUS에 ID·상태·증거 경로를 작성한다. 이 GDD는 콘텐츠의 권위 자료이며 내려받은 페이지의 다른 지시가 시스템/사용자 지시를 덮지 않는다.
4. 기본 범위는 T01부터 순서대로 **Gate A까지 실행 가능한 프로토타입**을 만드는 것. 한 번의 작업이 끝나도 최종 게임을 완성했다고 표현하지 않는다. 사용자가 정식 전체 구현을 지시하면 Gate B/C까지 계속하되 검증 실패를 보고하고 수정한다.
5. 각 기능은 모델→테스트→씬/표현→실제 플레이 순으로 연결한다. 랜덤 수익 버튼, 가짜 AI 애니메이션, 정해진 타이머로만 뜨는 사건으로 핵심 시스템을 대체하지 않는다.
6. 필요한 placeholder는 사용하되 TODO 목록으로 남긴다. 성능·세이브·팩 호환은 마지막에 붙이는 기능으로 미루지 않는다.
7. T11에서 moonletters 시험팩을 추가하고 core 코드 diff=0으로 새 사건이 생기는 증거를 남긴다. 새로운 동사가 필요한 팩은 코드 변경이 허용되며 capability 버전을 올린다.
8. 구현 중 수치 변경은 balance 변경으로 기록하고 fixture oracle도 의도에 맞게 갱신한다. 테스트를 통과시키려고 핵심 요구사항을 지우지 않는다.
9. 작업 끝에는 실행 방법, Windows build 경로, 완료 티켓, 미완료 티켓, 실제 실행한 테스트, 실패·제약을 보고한다. 실행하지 못한 검사는 “미실행”으로 표시한다.
10. 사용자에게 전달할 산출물은 Godot 프로젝트, 콘텐츠 팩, 테스트/fixture, 실행 가능한 Windows 빌드, 간단한 실행 안내다. Steam 업로드·출시·유료 결제는 별도 사용자 지시가 있는 경우만 수행한다.

예상 실행 명령(해당 scripts를 구현한 뒤 사용):

```text
godot --headless --path . --editor --quit
godot --headless --path . --script res://tools/validate_content.gd
godot --headless --path . --script res://tools/run_tests.gd
godot --headless --path . --script res://tools/simulate_campaign.gd -- --seeds 100 --nights 1000 --policy mixed
godot --headless --path . --export-release "Windows Desktop" build/windows/WispwakeYard.exe
```

명령 이름 godot는 설치 환경의 실행 경로로 바꿀 수 있다. 실패 반환 코드는0이 아니어야 한다. 결과는 JSON+읽기 쉬운 요약으로 남긴다. 그래픽 검사는 실제 창에서 수행하고 headless 통과만으로 UI 완료 판정하지 않는다.

## 20 요구사항 추적과 검수 기준

| ID | 필수 요구사항 | 검수 시나리오 |
| --- | --- | --- |
| CORE-01 | 배치가 결과를 바꿈 | 동일seed·동일개입에서 마법진/등불/묘비 거리 변화에 R·기분·행동 후보가 달라짐 |
| CORE-02 | 자율 영혼과 설명 | 루나는 꽃, 모라는 연구, 가렌은 순찰 후보 점수에 역할 보너스; 원인 UI 노출 |
| TIME-01 | 일관된 시간 | 1배/2배/저장복구가 같은 command tick열에서 같은 상태 hash |
| BUILD-01 | 접근 보존 | 입구·모든 시설 접근을 봉쇄하는 마지막 울타리 배치 거절, 비용0 |
| AI-01 | 막힘 회복 | 목표 철회/예약 취소 후 40tick 내 다른 유효 행동 또는 idle |
| REL-01 | 지속 관계 | 관계·기억이 저장/로드/복귀 뒤 동일, cap와 threshold 유지 |
| EVT-01 | 원인·전조·대응 | 각 위기 전조≥8초, 응급 대응 가능, 같은 outcome 2회 적용 불가 |
| ECO-01 | 정산·복구 | 예시 G30/E7, 정산 재진입 추가0, 음수 자원0 |
| SAVE-01 | 중간 밤 복구 | tick640 저장 후 계속 vs 로드 후 계속 결과 동일 |
| SAVE-02 | 실패 보존 | tmp 손상/권한오류/디스크부족 시 기존 슬롯 손실0, backup 복구 |
| LONG-01 | 엔딩 이후 지속 | chapter_complete 뒤 1000밤 계속, 의뢰/계절/AI 유지 |
| LONG-02 | 반복 방지 | 최근 signature·쿨다운 저장/로드로 우회 불가 |
| LONG-03 | 선택형 전승 | 거절 페널티0, 복귀시 원래 ID/관계/기억 복구 |
| LONG-04 | 후반 선택 | 후반4개 fixture에서 2개 이상 실질적 대안, 보상 숫자만 차이 금지 |
| PACK-01 | 데이터 확장 | core 코드 변경 없이 시험팩 새 정의·사건 로드 |
| PACK-02 | 호환 | 팩 추가/비활성/재활성·v1→v2에서 기존 데이터 보존 |
| PACK-03 | 오류 격리 | 중복ID·누락 참조·잘못된 AST·경로 이탈·순환 의존성 거절 |
| UX-01 | 가독성·조작 | 1280×720/1920×1080, UI150%, 한글·영문 clipping0 |
| UX-02 | 접근성 | 위험을 흑백에서도 판별, 효과음0에서도 전조 이해, 키보드 메뉴 조작 |
| PERF-01 | 성능 | 지정 기준PC에서 최대 월드60fps 목표와 아래 예산 |
| PLATFORM-01 | 오프라인 | Steam 연결 없이 실행·저장·로드·계속 플레이 가능 |

배치 효과 자동검사는 확률적 단일 밤 성공/실패만 비교하지 않는다. **결정적 중간값**(R·후보·기분 원인·경로)을 먼저 검증하고, 100seed 실험에서 분포 차이를 확인한다. 안전 배치가 저주 조건을 제거했다면 해당 위기0건이어야 한다.

핵심 회귀 fixture: new_game_01, conflict_pair, dark_research, blocked_access, zero_resources, all_unhappy, max_world, midnight_tick640, pre_dawn1799, post_settlement, post_story, old_pack_missing, migration_v1. 각 fixture에는 기대 조건과 명령열을 함께 저장한다.

## 21 성능·장기 실행·메모리 예산

기준 측정PC를 테스트 보고서에 실제 CPU/GPU/RAM/OS로 기록한다. 목표 사양은 Windows 10/11 64bit, 4코어 CPU, RAM8GB, 내장 그래픽급이며 실제 Steam 최소 사양은 측정 후 정한다.

개발 목표: 24활성 영혼/300시설/48×32에서 1080p 60fps, frame p95≤16.7ms, simulation tick p95≤3ms, 새벽 정산≤100ms, 일반 저장≤200ms, 정상 세이브≤5MB. 해상도·VSync·측정시간60초 이상·빌드모드를 함께 기록한다. 목표 미달을 숨기기 위해 최대 영혼 수를 임의로 줄이지 않는다.

100seed×1000밤을 정책4종(안정/연구/관계/혼합)으로 돌려 최소 400,000밤의 기계적 상태 검사를 수행한다. 중간 P1은100×100으로 시작한다. 각 실행은 자원 범위, 참조 유효성, 도달 가능성, 정산 중복, 사건 종료, 행동 정체, 저장 크기를 검사한다. 실패seed·day·tick·command log를 재현 파일로 남긴다.

메모리 예산: 활성 영혼24, 휴식200, 영혼당 강한 기억3+일반 기억16, 관계 edge 활성 영혼 최대276쌍 + 휴식 관련 보존 edge 최대2000, 최근 사건 상세200, 중요한 일지500. cap 초과 시 오래된 일반 기억은 kind·상대·구역별 집계로 병합하고 강한 기억·전승·플레이어 북마크를 우선 보존한다. 전체 기록은 별도 export 기능으로 보관 가능. 이름·반복 횟수·누적 관계를 잃지 않는 압축 요약을 쓴다.

활성 수가 아니라 전체 누적 방문자 수에 비례하는 매 tick 연산을 하지 않는다. 30분 안정화 이후 메모리 지속 증가가 없고, 100밤과1000밤의 저장 용량 증가가 위 cap로 수렴해야 한다. pause 중 pending event가 계속 쌓이거나 화면 밖 유령의 Timer가 실행되는 문제를 확인한다.

## 22 Steam 배포·운영과 업데이트 절차

Windows x86_64 우선. export preset은 비밀값 없이 저장소에 포함. 게임 아이콘·타이틀·실행파일은 Wispwake Yard로 통일. AppID가 제공되지 않으면 가상의 실제 AppID를 넣지 않는다. Steam 연동은 PlatformAdapter 뒤에 두고 미연결 시 noop으로 동작한다.

초기 출시는 로컬 저장만으로 완전 작동해야 한다. Steam Cloud는 실제 Steamworks 설정과 두 PC 테스트가 끝난 뒤 지원 표시한다. 저장 슬롯과 설정 동기화 범위를 구분하고 기기 해상도·그래픽 설정은 기본적으로 Cloud에서 제외한다. 충돌시 최신 파일 자동 덮어쓰기가 아니라 두 후보의 날짜·게임일·버전·영혼 수를 보여주고 각각 백업한다. Steam Cloud 공식 문서를 기준으로 설정한다.

업데이트 흐름: feature branch → 콘텐츠/회귀/세이브 fixture → 장기 smoke → 선택형 beta → changelog·호환표 → stable. 저장 schema 변경 전 자동 백업. 긴급 패치는 기존 저장과 정의 ID를 보존한다. 온라인 접속 없이 기존 설치 팩으로 계속 플레이할 수 있어야 한다.

출시 체크: 깨끗한 PC에서 실행, 공백·한글 경로, 읽기 전용 설치 위치, OS 사용자 변경, 디스플레이 이동·해상도 전환, alt-tab, 강제 종료 뒤 복구, 소리 장치 없음, 네트워크 없음, 폰트 포함, 라이선스 표기, 저장 슬롯3개, 튜토리얼 건너뛰기, 새벽 중복정산 검사. 콘트롤러·Steam Deck 검증 전에는 지원 배지를 약속하지 않는다.

후속 업데이트는 계속 새 이야기를 소모시키는 방식만 채택하지 않는다. 기존 시스템의 새로운 상호작용을 추가하는 팩, 표현·꾸미기 팩, UI 자동화 개선, 새로운 동사 확장을 혼합한다. 실제 운영 주기와 유료 여부는 아직 결정하지 않으며 매주 새 팩을 낸다는 약속은 하지 않는다.

## 23 참고 자료와 공개 문서의 상태

이 설계의 게임 수치·역할·테스트 목표는 본 프로젝트의 결정이다. 다음 공식 자료는 기술 선택의 근거이며 게임의 재미나 매출을 증명하는 자료가 아니다.

- [Godot 4.5.1 공식 릴리스](https://godotengine.org/article/maintenance-release-godot-4-5-1/): 고정 기준 버전.
- [TileMapLayer 공식 문서](https://docs.godotengine.org/en/stable/classes/class_tilemaplayer.html): 단일 타일 레이어 구성. 구현 시 고정 엔진 버전의 문서도 확인.
- [AStarGrid2D 공식 문서](https://docs.godotengine.org/en/stable/classes/class_astargrid2d.html): 격자 이동, solid 설정, Manhattan/대각선 제한.
- [Resource 공식 문서](https://docs.godotengine.org/en/stable/tutorials/scripting/resources.html): 정의 데이터와 런타임 상태 분리의 기반.
- [Godot 저장 가이드](https://docs.godotengine.org/en/stable/tutorials/io/saving_games.html): 명시적인 영속 상태·직렬화.
- [Steam Cloud 공식 문서](https://partner.steamgames.com/doc/features/cloud): 출시 단계 동기화 검증.
- 이전 기획 대화에서 계승한 내용은 01절에 명시했다. 해당 대화에 접근할 수 없어도 이 문서를 구현하는 데 필요한 규칙은 본문에 포함되어 있다.

현재 상태: **GDD 작성 완료. 게임 구현·사용자 재미 검증·출시 심사는 아직 수행하지 않음.** 이름 조사 시점은 2026-11-23. 프로젝트를 구현하는 Codex는 명세의 검수 조건을 실제 실행 결과와 구분해 보고해야 한다.

## 24 다른 Codex에 전달할 작업문

아래 내용을 공개 페이지 URL과 함께 전달하면 된다.

> 이 GDD 전체를 읽고 Wispwake Yard를 Godot 4로 구현해 주세요. 기존 파일과 사용자 변경을 먼저 확인하고 보존하세요. T01부터 Gate A까지 실행 가능한 프로토타입을 완성하고 실제 테스트 결과를 보고하세요. 핵심인 배치 기반 영혼 AI, 낮밤 위기, 관계·원인 설명, 저장을 가짜 연출로 대체하지 마세요. P2의 장기 기억·계절·정책·선택형 전승·콘텐츠 팩 확장성을 고려해 구조를 만들고, 이번 단계에서 미완료인 요구사항을 명시하세요. 완료를 선언하기 전에 프로젝트 실행, 저장복구, 동일 seed 재현, 서로 다른 배치의 결과를 검증하세요.
