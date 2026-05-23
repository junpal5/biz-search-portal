# biznoapi.html — 비즈노 기반 사업자 상세정보 조회

## 개요

- **원본 레포**: `junpal5/biz-tracker-biznoapi` (원본 CLAUDE.md 참조)
- **현재 버전**: v1.4.0 (페이지 하단 버전 칩 + `VERSION_HISTORY` 배열 참조)
- **API**: bizno.net (무료: `fapi`, 유료: `papi`)
- **업데이트 기준**: 원본 레포의 `index.html`을 그대로 가져와 포털 통합만 적용

## ⚠️ 디자인 변경 금지

이 파일은 `biz-tracker-biznoapi/index.html` **원본 디자인을 그대로 보존**해야 한다.  
외부 디자인 시스템(PostHog, Linear 등) 적용 금지.

### 원본 디자인 시스템 (MiniMax)
```css
--color-primary:     #0A0A0A;   /* 검정 */
--color-canvas:      #FFFFFF;   /* 흰색 배경 */
--color-surface:     #F5F5F5;
--color-hairline:    #E5E5E5;
--color-ink:         #1A1A1A;
--color-charcoal:    #404040;
--color-steel:       #888888;
--color-brand-coral: #FF4B2B;   /* 버전 칩 dot, 프로그레스 바 */
--color-brand-blue:  #2563EB;   /* 링크, 포커스 링 */
--rounded-full: 9999px;
--font: 'DM Sans', 'Inter', -apple-system, sans-serif;
```
- 버튼: `border-radius: var(--rounded-full)` (pill 형태)
- 카드: `border-radius: var(--rounded-xl)` + `border: 1px solid var(--color-hairline)`

## 포털 통합 처리 내역

원본 대비 수정 사항:

1. **로그인 화면 제거**: `#loginScreen` HTML div 및 관련 CSS (`.login-card`, `.btn-login` 등), JS (`LOGIN_PWD` 상수, `submitLogin()`, `toggleLoginPwd()`, `initLogin` IIFE) 완전 삭제
2. **sessionStorage 인증 체크 추가** (`</head>` 직전):
   ```javascript
   (function() {
     if (!sessionStorage.getItem('portal_auth')) {
       window.location.replace('index.html');
     }
   })();
   ```
3. **homeScreen 초기 노출**: `display:none` → `display:flex` (포털에서 인증 후 바로 표시)
4. **포털 복귀 버튼 추가**: homeScreen 좌상단 절대 위치
   ```html
   <a href="index.html" style="position:absolute; top:20px; left:20px; ...">← 포털로</a>
   ```
   - API 조회/웹 조회 페이지에서는 헤더의 `← 홈` 버튼으로 homeScreen 복귀 (포털 버튼 없음 — 의도적 설계)

## 화면 구조

```
#homeScreen     — 조회 방식 선택 (← 포털로 버튼 있음)
  ├── goApiPage()       → #apiPage     (← 홈 버튼으로 homeScreen 복귀)
  └── goWebLookupPage() → #webLookupPage (← 홈 버튼으로 homeScreen 복귀)
```

## bizno.net API 스펙

### 엔드포인트
- 무료: `https://bizno.net/api/fapi`
- 유료: `https://bizno.net/api/papi`

### 요청 파라미터
| 파라미터 | 설명 | 값 |
|---------|------|-----|
| `key` | API 인증키 (필수) | `BIZNO_API_KEY` 상수 (HTML 내 하드코딩) |
| `gb` | 검색 유형 | 1=사업자등록번호, 2=법인등록번호, 3=상호명 |
| `q` | 검색어 (**`query`가 아님**) | 사업자번호 또는 상호명 |
| `type` | 응답 형식 | `json` 고정 |

### 응답 주요 필드
| 필드 | 설명 |
|------|------|
| `bno` | 사업자등록번호 |
| `company` | 상호명 |
| `ceo` | 대표자 |
| `BsttCd` | 사업자상태코드 (01=계속, 02=휴업, 03=폐업) |
| `bstt` | 사업자상태명칭 |
| `TaxTypeCd` | 과세유형코드 |
| `taxtype` | 과세유형명칭 |
| `EndDt` | 폐업일 |

### 오류 처리
- `resultCode < 0` → 오류 (`checkApiError()` 함수 처리)
- `items`가 빈 문자열 `""` 또는 `null` 가능 → `parseItems()` 함수 처리

## 주요 함수 계층

```
callBizno(query, gb)        — 원시 API 호출 (q= 파라미터)
  ├── searchByName(name)    — gb=3 (상호명 검색)
  ├── getDetail(bizno)      — gb=1 (사업자번호 직접 조회)
  └── searchByNameRaw(name, gb) — 테스트용
```

### 조회 흐름
1. 엑셀 업로드 → 헤더 파싱 → 열 선택
2. **사업자번호 있음** → `getDetail(bizno)` (gb=1)
3. **사업자번호 없음** → `searchByName(name)` (gb=3) → 1건 자동 확정 / 다건 disambiguation UI
4. 확정 사업자번호 → `getDetail()` → 결과 테이블 + XLSX 다운로드

## 원본 레포 업데이트 시 포털 반영 방법

1. `biz-tracker-biznoapi/index.html` 원본을 받아 `/tmp/biznoapi_original.html` 저장
2. Python 스크립트로 포털 통합 패치 적용 (로그인 제거 + sessionStorage + homeScreen + 포털 버튼)
3. `biz-search-portal/biznoapi.html` 교체

```python
# 핵심 패치 포인트 (Python 문자열 치환 방식)
# 1. 로그인 CSS 제거 (#loginScreen, .login-card, .btn-login 블록)
# 2. homeScreen: display:none → display:flex, position:relative 추가
# 3. homeScreen 첫 자식으로 ← 포털로 <a> 태그 삽입
# 4. loginScreen HTML 블록 제거
# 5. LOGIN_PWD 상수 제거
# 6. initLogin IIFE 제거
# 7. submitLogin + toggleLoginPwd 함수 제거
# 8. </head> 직전에 sessionStorage 인증 체크 <script> 삽입
```

## 버전 관리

### 업데이트 절차
1. `biznoapi.html` 내 `VERSION_HISTORY` 배열 **맨 앞**에 새 항목 추가
2. 하단 버전 칩 HTML 텍스트 업데이트 (`.ver-chip` 버튼 내 `vX.X.X`)

### 작업 완료 후 버전 추천 방법
→ 루트 `CLAUDE.md`의 **"버전 관리 — 작업 완료 후 버전 추천 워크플로우"** 참조.

```bash
git diff HEAD~1 HEAD -- biznoapi.html   # 변경 내역 파악
```

변경 내역을 검토한 뒤 패치/마이너/메이저 추천을 제시하고, 사용자가 선택하면 즉시 반영한다.

> **주의**: `biznoapi.html`을 원본 레포(`biz-tracker-biznoapi/index.html`)로 교체할 경우 버전 번호는 원본 기준을 따름.

## 버전 히스토리 (포털 통합 기준)
| 버전 | 날짜 | 주요 변경 |
|------|------|----------|
| 1.4.0 | 2026-05-21 | 홈 화면 + "사업자등록번호 조회" 페이지 추가 |
| 1.3.0 | 2026-05-20 | FIELD_KO 전면 확장, Excel 컬럼 순서 고정 |
| 1.2.0 | 2026-05-14 | API 키 입력창 제거, 테스트 패널 상단 이동 |
| 1.1.0 | 2026-05-14 | API 파라미터 수정 (query→q) |
| 1.0.0 | 2026-05-13 | 최초 출시 |
