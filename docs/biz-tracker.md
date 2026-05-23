# biz-tracker.html — 사업자 휴·폐업 조회

## 개요

- **원본 레포**: `junpal5/biz-tracker` (원본 CLAUDE.md 참조)
- **현재 버전**: 페이지 내 `VERSION_HISTORY` 참조 (biz-tracker 원본 기준 v2.0.1)
- **API**: 국세청 사업자등록 상태조회 공공데이터 포털 API

## 포털 통합 처리 내역

원본 `index.html` 에서 다음을 수정하여 `biz-tracker.html`로 통합:

1. **로그인 화면 제거**: `#loginScreen` HTML 및 관련 CSS/JS 완전 삭제
2. **sessionStorage 인증 체크 추가** (`</head>` 직전):
   ```javascript
   (function() {
     if (!sessionStorage.getItem('portal_auth')) {
       window.location.replace('index.html');
     }
   })();
   ```
3. **포털 복귀 버튼 추가**: 헤더 좌상단 `<a href="index.html" class="portal-back-btn">← 포털로</a>`

## 디자인 시스템 (Linear)

원본은 MiniMax 디자인이었으나, 포털 통합 시 Linear 디자인 시스템으로 재적용.

### 주요 CSS 토큰
```css
--color-canvas:        #010102;   /* 다크 배경 */
--color-surface-1:     #0d0e12;
--color-surface-2:     #141519;
--color-surface-3:     #1a1b21;
--color-hairline:      #23252a;
--color-ink:           #f7f8f8;   /* 밝은 텍스트 */
--color-charcoal:      #d0d6e0;
--color-steel:         #8a8f98;
--color-primary:       #5e6ad2;   /* 라벤더 블루 */
--color-primary-hover: #828fff;
--color-brand-coral:   #f87171;   /* 상태 뱃지 등 */
--rounded-md:          8px;
--rounded-lg:          12px;
--font: 'Inter', -apple-system, BlinkMacSystemFont, system-ui, sans-serif;
```

### 디자인 원칙
- 다크 캔버스 (`#010102`) 기반
- 라벤더 블루 (`#5e6ad2`) 단일 액센트
- 4단계 서피스 레이어 (surface-1 ~ surface-3)
- 그림자 없음, 그레이디언트 없음
- 버튼: `rounded-md` (8px), 카드: `rounded-lg` (12px)

## API 스펙 (국세청 NTS)

- **URL**: `POST https://api.odcloud.kr/api/nts-businessman/v1/status?serviceKey=...`
- **Body**: `{ "b_no": ["0000000000", ...] }` ← 문자열 배열
- **배치 크기**: 1회 최대 100건
- **상태코드**: `01` 계속사업자 · `02` 휴업자 · `03` 폐업자
- **CORS**: 브라우저 직접 호출 시 오류 가능 (API 키 필요)

## 앱 기능 요약

| 단계 | 설명 |
|------|------|
| Step 1 | API 인증키 입력 (password 타입, 보기/숨기기 토글) |
| Step 2 | XLSX 업로드 → 헤더 자동 감지 → 사업자번호 열 선택 (A~G, 최대 7개) |
| Step 3 | 1건 사전 검증 → 100건 배치 API 호출 → 프로그레스바 → 결과 테이블 |
| 결과 | 원본 + `사업자상태`/`상태코드`/`세금종류` 열 → XLSX 다운로드 |

### 헤더 자동 감지 로직
`XLSX.utils.sheet_to_json(ws, { header: 1 })` → 비어있지 않은 셀이 2개 이상인 첫 행을 헤더로 사용. 빈 셀은 `열N`으로 대체.

## 수정 시 주의사항

- 원본 레포 `junpal5/biz-tracker`의 기능과 싱크를 맞출 때, 원본 CLAUDE.md의 버전 관리 패턴 참조.
- 버전 히스토리는 HTML 내 `VERSION_HISTORY` 배열 + `.ver-chip` 텍스트를 함께 업데이트.
- Linear 디자인 유지 — MiniMax(원본)이나 PostHog 디자인 혼용 금지.
