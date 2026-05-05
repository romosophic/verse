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
  accounts[],        // { id, bank, name, balance, rate, owner('경우'|'새롬'), prevBalance, accountNumber, _dbId }
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
- `accounts`: `bank`, `name`, `balance`, `prev_balance`, `rate`, `owner`, `is_active`, `account_number` (text, nullable)
- `transactions`: `type`(in/out), `category`, `source`(manual/weekly_auto/balance_adjust), `is_manual`, `is_auto_deduct`
- `snapshots`: 자산 추이 스냅샷

## 페이지 구성 (nav-tabs, showPage id)
- `dashboard`: 총 자산, 자산 추이 차트, 이자 ticker, 최근 거래
- `weekly`: 주간 체크인 (경우/새롬 잔액 확인 + 경쟁)
- `income`: 수입/지출 입력
- `txns`: 전체 거래내역
- `assets`: 자산 관리 — 정렬 가능한 테이블(소유자/은행/계좌명/계좌번호/잔액/이율/수정), ＋추가 버튼(섹션 우상단), 초대코드 관리(접힘/펼침)

## 자산 테이블 (assets 페이지)
- 컬럼 헤더 클릭으로 오름/내림차순 정렬
- 경우: 초록 왼쪽 보더 (`owner-k`), 새롬: 골드 왼쪽 보더 (`owner-s`)
- ✏️ 아이콘 클릭 → `openEditBalanceModal(acc)` (수정/삭제 모달)
- 계좌번호는 뒷 4자리만 표시 (`···1234`)
- `editAccById(id)`: 테이블 행에서 수정 모달 여는 헬퍼

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
- 금액 표시: `fmt(n)` 유틸 → `₩1,234,567`
- 금액 입력: `type="text" inputmode="numeric" oninput="fmtAmtInput(this)"` — **절대 `type="number"` 사용 금지**
- 금액 읽기: `parseAmt('inputId')` — 쉼표 제거 후 파싱
- 금액 세팅: `fmtRaw(n)` — 숫자를 쉼표 포맷 문자열로 변환
- 이율 등 소수점 필드만 `type="number"` 유지
- 섹션 구분자: `// ===== SECTION =====`

## 주요 함수
- `bootApp()`: DB 로드 → 렌더링 시작
- `showPage(id)`: 페이지 전환 (assets 시 `renderAccounts()` 호출)
- `renderAccounts()`: 자산 테이블 렌더, `_accSort`로 정렬 상태 관리
- `sortAccounts(col)`: 테이블 컬럼 정렬 토글
- `editAccById(id)`: id로 계좌 찾아 수정 모달 열기
- `openEditBalanceModal(acc)` / `saveEditedBalance()`: 잔액·이율·계좌번호 수정
- `deleteAccount()`: 자산 삭제 (is_active = false)
- `dbSaveTransaction()`: source 필드로 거래 구분 (balance_adjust = 기초자산수정)
- `loadInviteStatus()` / `toggleInviteCode()`: 초대코드 활성/비활성 관리
- `confirmQAUnchanged(accountIdx)`: 주간체크 잔액 변동 없음 즉시 확인
- `fmtAmtInput(el)`: 금액 입력 필드 실시간 쉼표 포맷
- `parseAmt(id)`: 쉼표 포함 문자열 → 숫자 파싱
- `fmtRaw(n)`: 숫자 → 쉼표 포맷 문자열 (input value 세팅용)

## 하지 말 것
- 별도 JS/CSS 파일로 분리 제안 금지 (단일 파일 구조가 의도된 설계)
- 프레임워크(React 등) 도입 제안 금지
- `state` 객체를 직접 외부에서 조작하지 말 것 — db 함수 통해 저장 후 state 갱신
- 계좌 소유자 옵션에 '공동' 추가 금지 (경우/새롬만)
- 금액 입력 필드에 `type="number"` 사용 금지 — 쉼표 포맷 깨짐
