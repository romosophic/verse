# MEMORY.md

## 프로젝트 개요
커플(경우 + 새롬)의 공동 자산을 관리하는 가계부 웹앱 — "벌쓰 (벌고쓰는 우리의 서사)"
- 배포: https://our-verse.vercel.app
- GitHub: `romosophic/verse` (main 브랜치 push → Vercel 자동 배포)

## 기술 스택
- **프론트엔드**: 순수 HTML/CSS/JS (프레임워크 없음, 단일 `index.html`)
- **DB/백엔드**: Supabase (PostgreSQL + Realtime) — `https://jfegtbghvcivsvuaqzjj.supabase.co`
- **폰트**: x12y12pxMaruMinya (로컬 woff2, `./fonts/x12y12pxMaruMinyaHangul.woff2`), html base 18px
- **차트**: Chart.js
- **배포**: Vercel

## 핵심 아키텍처 결정
- 모든 UI/로직이 `index.html` 한 파일에 집약
- 전역 `state` 객체로 앱 상태 관리 (React/Vue 없음)
- 페이지 전환은 `.page` / `.page.active` CSS 클래스로 처리
- Supabase Realtime으로 두 사람이 동시에 보는 데이터 동기화
- `localStorage`에 세션(playerName, householdId, inviteCode 등) 저장

## 데이터 구조 (state)
```
state = {
  accounts[],        // { id, bank, name, balance, rate, owner('경우'|'새롬'), prevBalance, _dbId }
  transactions[],    // { id, emoji, name, meta, amount, type('in'|'out'), ts, manual, autoDeducted, isAdjust }
  wealthHistory[],   // { date, val }
  weeklyProgress,    // { 경우: { done, checkedAt, totalBalance, prevTotal }, 새롬: ... }
  ...
}
```

## DB 테이블
- `households`: `invite_code`, `invite_code_active` (boolean, 기본 true)
- `users`: 사용자 (경우/새롬)
- `household_members`: 가계-유저 연결
- `accounts`: `bank`, `name`, `balance`, `prev_balance`, `rate`, `owner`, `is_active`
- `transactions`: `type`(in/out), `category`, `source`(manual/weekly_auto/balance_adjust), `is_manual`, `is_auto_deduct`
- `snapshots`: 자산 추이 스냅샷

## 페이지 구성 (nav-tabs, showPage id)
- `dashboard`: 총 자산, 자산 추이 차트, 이자 ticker, 최근 거래
- `weekly`: 주간 체크인 (경우/새롬 잔액 확인 + 경쟁)
- `income`: 수입/지출 입력
- `txns`: 전체 거래내역
- `assets`: 자산 관리 (자산 현황 카드 + 초대코드 관리 — 접힘/펼침)

## 색상 토큰 (CSS vars)
| 변수 | 용도 | 값 |
|------|------|-----|
| `--bg` | 배경 | `#FFFAF0` 아이보리 |
| `--surface` / `--surface2` | 카드 배경 | `#FFFFFF` / `#F5EFE0` |
| `--accent` / `--accent2` | 경우 (초록) | `#81912F` / `#677525` |
| `--accent3` / `--gold` | 새롬 (골드) | `#C8900A` / `#A07008` |
| `--income` | 수입 (파랑) | `#2A7EC8` |
| `--danger` | 지출 (빨강) | `#E03F4F` |
| `--text` / `--muted` | 텍스트 | `#1E1A0E` / `#7A7260` |

## 코드 컨벤션
- 함수명: camelCase, 동사로 시작 (`bootApp`, `dbLoadAll`, `renderChart`)
- DB 함수 접두사: `db` (e.g. `dbSaveTransaction`, `dbUpdateAccountBalance`)
- 금액 표시: `fmt(n)` 유틸 사용 → `₩1,234,567`
- 섹션 구분자: `// ===== SECTION =====`

## 주요 함수
- `bootApp()`: DB 로드 → 렌더링 시작
- `showPage(id)`: 페이지 전환
- `renderAccounts()`: 자산 카드 렌더 (클릭 시 수정/삭제 모달)
- `dbSaveTransaction()`: source 필드로 거래 구분 (balance_adjust = 기초자산수정)
- `loadInviteStatus()` / `toggleInviteCode()`: 초대코드 활성/비활성 관리
- `openEditBalanceModal(acc)` / `saveEditedBalance()`: 자산 잔액·이율 수정
- `deleteAccount()`: 자산 삭제 (is_active = false)

## 하지 말 것
- 별도 JS/CSS 파일로 분리 제안 금지 (단일 파일 구조가 의도된 설계)
- 프레임워크(React 등) 도입 제안 금지
- `state` 객체를 직접 외부에서 조작하지 말 것 — db 함수 통해 저장 후 state 갱신
- 계좌 소유자 옵션에 '공동' 추가 금지 (경우/새롬만)
