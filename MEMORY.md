# MEMORY.md

## 프로젝트 개요
커플(경우 + 새롬)의 공동 자산을 관리하는 가계부 웹앱 — "벌쓰 (벌고쓰는 우리의 서사)"

## 기술 스택
- **프론트엔드**: 순수 HTML/CSS/JS (프레임워크 없음, 단일 `index.html`)
- **DB/백엔드**: Supabase (PostgreSQL + Realtime)
- **폰트**: Google Fonts — Gowun Dodum (본문), Black Han Sans (헤딩)
- **차트**: Chart.js (추정, chartInstance 사용)
- **배포**: Vercel

## 핵심 아키텍처 결정
- 모든 UI/로직이 `index.html` 한 파일에 집약
- 전역 `state` 객체로 앱 상태 관리 (React/Vue 없음)
- 페이지 전환은 `.page` / `.page.active` CSS 클래스로 처리
- Supabase Realtime으로 두 사람이 동시에 보는 데이터 동기화
- `localStorage`에 세션(playerName, household_id 등) 저장

## 데이터 구조 (state)
```
state = {
  accounts[],        // { id, bank, name, balance, rate, owner('경우'|'새롬'|'공동'), prevBalance }
  transactions[],    // { id, emoji, name, meta, amount, type('in'|'out'), ts, manual }
  wealthHistory[],   // { date, val }
  weeklyProgress,    // { 경우: { done, checkedAt, totalBalance, prevTotal }, 새롬: ... }
  weeklyStep,
  weeklyPlayer,
  entryMode,
  lastWeeklyTs,
  interestAccrued,
}
```

## 페이지 구성 (nav-tabs)
- 대시보드 (자산 합계, 차트, 계좌 목록, 최근 거래)
- 수입/지출 입력
- 주간 체크 (경우 / 새롬 각자 잔액 확인)
- 주간 경쟁 (지출 적은 사람 승리)

## 색상 토큰 (CSS vars)
| 변수 | 용도 |
|------|------|
| `--accent` `--accent2` | 경우 (블루) |
| `--accent3` `--gold` | 새롬 (골드/앰버) |
| `--danger` | 지출 (빨강) |
| `--bg` `--surface` `--surface2` | 배경 레이어 |

## 코드 컨벤션
- 함수명: camelCase, 동사로 시작 (`bootApp`, `dbLoadAll`, `renderChart`)
- DB 함수 접두사: `db` (e.g. `dbSaveTransaction`, `dbUpdateAccountBalance`)
- 금액 표시: `fmt(n)` 유틸 사용 → `₩1,234,567`
- 고유 ID 생성: `uid()` (timestamp + random)
- 섹션 구분자: `// ===== SECTION =====`

## 알려진 제약사항
- 단일 HTML 파일이라 코드 분리 없음 — 파일 길이 매우 김 (1500줄+)
- Supabase key가 소스에 하드코딩됨 (publishable key이므로 의도적)
- 차트 데이터(wealthHistory)는 더미 노이즈 포함 — 실제 스냅샷과 혼용

## 하지 말 것
- 별도 JS/CSS 파일로 분리 제안 금지 (단일 파일 구조가 의도된 설계)
- 프레임워크(React 등) 도입 제안 금지
- `state` 객체를 직접 외부에서 조작하지 말 것 — db 함수 통해 저장 후 state 갱신
