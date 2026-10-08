# app-fast 변경 내역 (테니스 게임로그 2.0 빠른 버전)

`old/`의 동작하는 앱(index.html, app.js, style.css, guide.html, logo.png)을 `app-fast/`로 복사한 뒤, **추가 방식으로만** 신규 기능을 넣었습니다. 빌드 없이 `app-fast/index.html`을 더블클릭(file://)해서 열면 됩니다. 외부 라이브러리·CDN·ES 모듈은 없습니다.

## 1. 기능별 추가 내용

| # | 기능 | 추가/변경한 함수·요소 |
|---|---|---|
| 1 | 평균 점수 차이 | 순수 함수 `calcAvgScoreDiff(matches)`, 보조 `getMatchScoreDiff`, `formatScoreDiff`(`+1.4` / `-0.8` / `0.0` / 없으면 `-`). `addResultToGroup`에 선택 인자(match)를 추가하고 `buildSortedNameStats`·`calcByCourt`가 결과 객체에 `avgScoreDiff`를 덧붙임(기존 필드는 그대로). 대시보드 카드 `평균 점수 차이`(`renderDashboardSummary`), 통계 탭 전체 카드(`renderOverallStats`), 파트너/상대/구장/포지션 표의 `점수차` 열. 기존 `avgDifference`(NTRP 전력차)는 건드리지 않음 |
| 2 | 대시보드 카드 | `최근 5경기`(`calcRecent(matches, 5)` 재사용), `열세 경기 승률`(`calcNtrpUpsetStats().underdog.winRate`, 없으면 `-`). 기존 카드 뒤에 덧붙임 |
| 3 | 경기 상세 화면 | `openMatchDetail`, `closeMatchDetail`, `createMatchDetailView`, `setRecordsBodyHidden`, `formatIntScoreDiff`. `createMatchCard`에 카드 본문 클릭·Enter(`tabindex=0`) 연결(내부 버튼 클릭은 상세를 열지 않음). 상세: 날짜, 결과 배지, 큰 점수 `6 : 3`, `점수 차이 +3`, 구장/파트너/상대/포지션/시간/메모, NTRP 분석 블록(`createNtrpAnalysisBlock` 재사용), `← 목록으로`, `수정`(상세를 닫고 기존 `startEditMatch` 호출), `삭제`(기존 `handleDeleteMatch` 호출 후 삭제됐으면 목록 복귀). 다른 탭으로 이동하면 상세가 닫힘(`activateTab`에 3줄 추가). `index.html`에 `#match-detail` 추가. data-testid: `match-card`, `detail-back-button`, `detail-score`, `detail-score-diff`, `detail-edit-button`, `detail-delete-button`, `match-detail` |
| 4 | NTRP 구간별 승률 | 순수 함수 `calcOpponentNtrpBandStats(matches)`(상대팀 평균 NTRP 기준, `available:true` 경기만, 2.5 미만 행은 경기가 있을 때만), `renderOpponentNtrpBandStats`, `index.html`의 `#stats-ntrp-band`(전력 열세 섹션 바로 뒤). 열: 구간/경기/승/패/무/승률. 빈 문구는 요청대로 |
| 5 | 최고 파트너 | 순수 함수 `pickBestPartner(partnerStats)`(승+패 3 이상 중 승률 최고, 동률이면 경기 수 → 평균 점수차 → 이름). 파트너 표에서 `★` + 노란 행(`best-partner-row`) + 표 아래 안내 `★ 최소 3경기 이상 중 승률이 가장 높은 파트너` |
| 6 | 상대별 최근 결과 | `calcByOpponent` 결과에 `recentResults`(최신순 최대 5개, `sortMatchesByRecent` 사용) 추가. 상대 표 `최근 결과` 열에 승/패/무 칩(`createResultChips`) |
| 7 | 그래프(CSS만) | `renderDashboardGraphs`, `index.html`의 `#dashboard-graphs`(요약 카드 아래). 최근 10경기 칩(왼쪽이 가장 최근) + 월별 승률 막대(최근 최대 6개월, 값은 막대 위, 승률이 `-`인 달은 빈 막대) |
| 8 | 문구/설명서 | 개발자의 한마디에 `26.10.08 - 평균 점수 차이, 경기 상세, NTRP 구간별 승률, 최고 파트너, 상대별 최근 결과, 그래프 추가` 추가. `guide.html`에 `2.0 신규 기능` 섹션과 목차 항목 추가 |

CSS는 `style.css` 맨 끝에 신규 규칙만 덧붙였습니다(녹색 #2e7d32 테마, 상세 화면 버튼은 min-height 44px).

## 2. 보존 항목 (변경하지 않음)

- localStorage 키와 저장 형식, 백업/복원 형식(backupVersion 1), 기존 화면·메시지, 기존 함수명·요소 id.
- 새 코드는 ES5 스타일(var/function)이며 `?.`·`??` 없음. 사용자/백업 유래 텍스트는 `textContent`로만 렌더링. `innerHTML`은 기존처럼 `""`로 비울 때만 사용.
- `old/`는 읽기만 했습니다. `app/`, `tests/`, `package*.json`, `.gitignore`, `aidlc-state.md`는 건드리지 않았고, `aidlc-docs/audit.md`에는 항목 1개만 append 했습니다.

## 3. 검증한 것 (실제 실행 결과)

- `node --check app-fast/app.js`: 기능별로 확인했고 최종본도 통과(stderr 비어 있음).
- vm 샌드박스 순수 함수 테스트(임시 스크립트, 삭제함): **77개 단언 모두 통과**.
  - `calcAvgScoreDiff`(점수 없는 경기 제외, 반올림), `formatScoreDiff`(`+1.4`/`-0.8`/`0.0`/`-`)
  - `calcByPartner/Opponent/Venue/Court`의 `avgScoreDiff`, `calcByOpponent`의 `recentResults`(최신순, 5개 제한, 같은 경기 상대 이름 중복 1회)
  - `pickBestPartner` 동률 규칙(승률 → 경기 수 → 평균 점수차 → 이름), 3경기 미만/무승부만 있는 경우 null
  - `calcOpponentNtrpBandStats`(경계값 2.5/3.0/4.0, 2.5 미만 행 조건부, NTRP 미등록 제외, 내 NTRP 없으면 빈 배열)
  - **기존 함수 회귀 비교**: 같은 입력으로 `old/app.js`와 `app-fast/app.js`를 모두 로드해 `calcTotals`, `calcRecent(5/10)`, `calcStreak`, `calcWinRate`, `calcMonthly`, `calcMonthlyWithNtrp`, `calcByPartner/Opponent/Venue/Court`, `calcByPartnerWithNtrp`, `calcByOpponentWithNtrp`, `calcNtrpUpsetStats`, `calcNtrpDifficultyStats` 결과가 (신규 필드를 제외하면) 동일함을 확인.
- 가짜 DOM 스모크 테스트(임시 스크립트, 삭제함): **28개 단언 통과**. 대시보드 신규 카드·그래프(칩 7개, 막대 높이 %), 통계 탭 신규 열·★ 행·힌트·구간 표·빈 문구, 카드 5장 렌더, 수정 버튼 클릭 시 상세 안 열림, 카드 본문 클릭/Enter로 상세 열림(폼·목록 숨김), `← 목록으로` 복귀, 상세 삭제 후 목록 복귀, 경기 0건일 때 그래프 비움, 메모에 넣은 `<b>x</b>`가 글자 그대로 표시됨(innerHTML 미사용).
- 정적 검색: `app-fast/app.js`에 `?.`·`??` 없음, `innerHTML`은 전부 `= ""`(비우기)만 사용.
- `old/` 무변경: `old/logo.png`와 `app-fast/logo.png`의 SHA-256 동일. `old/`의 파일 수정 시각이 이번 작업 이전 값(가장 늦은 `index.html`이 14:09)이라 이번 작업에서 쓰지 않았음. (이 환경에는 git이 없어 `git status`로는 확인하지 못함)

## 4. 검증하지 못한 것

- **실제 브라우저에서 열어보지 못했습니다.** 레이아웃(그래프 막대, 표 가로 스크롤, 노란 행, 상세 화면 모양), 터치 동작, file:// 로딩, 실제 localStorage 연동은 확인하지 못했습니다. 아래 체크리스트로 직접 확인해 주세요.
- 가짜 DOM은 필요한 DOM 일부만 흉내 낸 것이라 실제 브라우저 동작과 다를 수 있습니다.

## 5. 시간 때문에 생략한 기능

없음. 8개 항목을 모두 구현했습니다. (단 위 4번의 브라우저 확인은 미완료입니다.)

## 6. 직접 확인용 클릭 체크리스트 (10항목)

1. `app-fast/index.html`을 더블클릭해서 엽니다. 사용자명 창이 뜨면 이름을 입력하고 시작합니다. 콘솔에 오류가 없는지 봅니다(F12).
2. 대시보드 `데이터 복원`으로 기존 백업(JSON)을 복원합니다. `데이터를 복원했습니다.` 메시지가 뜨는지 확인합니다.
3. 대시보드 요약 카드에 `최근 5경기`, `열세 경기 승률`, `평균 점수 차이`가 보이고, 기존 카드(총 경기, 승/무/패, 승률, 연승/연패, 최근 10경기, 요즘 폼)도 그대로인지 확인합니다.
4. 요약 카드 아래 그래프를 봅니다. 최근 10경기 칩(왼쪽이 가장 최근)과 월별 승률 막대(최근 6개월 이내, 값이 막대 위에 표시)가 보이는지 확인합니다. 폰 화면 폭에서도 확인합니다.
5. `경기 기록` 탭에서 카드 본문을 눌러 상세 화면을 엽니다(키보드는 Tab으로 카드에 이동 후 Enter). 날짜, 결과 배지, 큰 점수, `점수 차이`, NTRP 분석이 보이는지 확인합니다.
6. 상세에서 `← 목록으로`를 눌러 같은 페이지로 돌아오는지 확인합니다(2페이지에서 열었다가 돌아와 보세요).
7. 상세의 `수정`을 눌러 폼에 값이 채워지고 저장이 되는지 확인합니다. 카드의 `수정`/`삭제` 버튼은 상세 없이 예전처럼 바로 동작하는지도 확인합니다.
8. 상세 또는 카드에서 `삭제`를 눌러 확인창이 뜨고, 삭제 후 목록이 갱신되는지 확인합니다. 확인창에서 취소하면 상세에 그대로 남아야 합니다.
9. `통계` 탭: 전체 통계의 `평균 점수 차이` 카드, 파트너 표의 `점수차` 열과 ★ 노란 행(3경기 이상인 파트너가 있을 때)과 표 아래 안내 문구, 상대 표의 `점수차`·`최근 결과` 칩, 구장/포지션 표의 `점수차`, `상대팀 NTRP 구간별 승률` 표를 확인합니다.
10. `개발자의 한마디`에 새 줄이 있는지, `사용 설명서 보기`에서 `2.0 신규 기능` 섹션이 보이는지, `데이터 백업`으로 받은 파일을 다시 복원해도 문제가 없는지 확인합니다.

## 7. 데이터 공유 관련 주의

localStorage는 브라우저·출처(origin)별로 따로 저장됩니다. `file://`로 열었을 때 `old/index.html`과 `app-fast/index.html`이 같은 데이터를 공유하는지는 브라우저마다 다릅니다(경로별로 분리하는 브라우저도 있고 공유하는 브라우저도 있음). 데이터를 옮길 때는 기존 앱에서 `데이터 백업` → 새 앱에서 `데이터 복원`을 사용하세요. GitHub Pages 등에 올리면 주소가 달라져 데이터가 따로 저장되므로 같은 방법으로 옮깁니다.
