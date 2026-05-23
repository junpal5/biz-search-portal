# ksic.html — 한국표준산업분류 검색 + 엑셀 변환

## 개요

- **데이터**: 한국표준산업분류(KSIC) 제11차 개정 전체 분류 데이터
- **원본**: 업로드된 단일 HTML 파일 (외부 레포 없음)
- **포털 통합**: sessionStorage 인증 체크 + 포털 복귀 버튼 추가

## 포털 통합 처리 내역

1. **sessionStorage 인증 체크** (`</head>` 직전):
   ```javascript
   (function() {
     if (!sessionStorage.getItem('portal_auth')) {
       window.location.replace('index.html');
     }
   })();
   ```
2. **포털 복귀 버튼**: 헤더 영역에 `<a href="index.html">← 포털로</a>` 추가
3. **엑셀 변환 탭 추가**: 기존 3개 탭(검색/코드조회/계층탐색)에 4번째 "엑셀 변환" 탭 신규 추가

## 탭 구조

```javascript
// switchTab() 내 탭 배열
['search', 'code', 'tree', 'excel']
```

| 탭 ID | 설명 |
|-------|------|
| `search` | 키워드로 KSIC 항목 검색 |
| `code` | 코드로 직접 조회 (숫자 2~5자리 또는 영문 A~U) |
| `tree` | 대분류 선택 후 계층 탐색 |
| `excel` | 엑셀 업로드 → 분류 코드 자동 추가 → 다운로드 |

## DATA 배열 구조

```javascript
const DATA = [
  { code: "A",     name: "농업, 임업 및 어업",   level: 1, major: "A" },
  // ...
  { code: "01",    name: "농업",                level: 2, major: "A" },
  { code: "011",   name: "작물 재배업",           level: 3, major: "A" },
  { code: "0111",  name: "식량작물 재배업",        level: 4, major: "A" },
  { code: "01110", name: "식량작물 재배업",        level: 5, major: "A" },
  // ...
];
```

### level 값 의미
| level | 설명 | code 형식 |
|-------|------|----------|
| 1 | 대분류 | 영문 1자 (A~U, 총 21개) |
| 2 | 중분류 | 숫자 2자리 |
| 3 | 소분류 | 숫자 3자리 |
| 4 | 세분류 | 숫자 4자리 |
| 5 | 세세분류 | 숫자 5자리 |

## 엑셀 변환 기능 상세

### 사용 라이브러리
- **SheetJS** `xlsx-0.20.3` CDN: `https://cdn.sheetjs.com/xlsx-0.20.3/package/dist/xlsx.full.min.js`

### 변환 로직

#### 조회 맵 (초기화 시 생성)
```javascript
const MAJOR_BY_NAME = {};   // 대분류명 → DATA 항목
const MAJOR_BY_CODE = {};   // 대분류코드(A~U) → DATA 항목
const DETAIL_BY_NAME = {};  // 세세분류명 → DATA 항목
const DETAIL_BY_CODE = {};  // 세세분류코드(5자리) → DATA 항목
const ALL_BY_NAME = {};     // 전체 이름 → DATA 항목

DATA.forEach(d => {
  ALL_BY_NAME[d.name] = d;
  if (d.level === 1) { MAJOR_BY_NAME[d.name] = d; MAJOR_BY_CODE[d.code] = d; }
  else if (d.level === 5) { DETAIL_BY_NAME[d.name] = d; DETAIL_BY_CODE[d.code] = d; }
});
```

#### 자동 감지 (detectExcelLevel)
- 샘플 최대 100행을 읽어 대분류 hit 수 vs 세세분류 hit 수 비교
- `majorScore >= detailScore` → 대분류로 판정
- 사용자가 수동으로 변경 가능

#### 대분류 변환 시 추가 열
- **`대분류 코드`**: 영문 알파벳 (A~U)

#### 세세분류 변환 시 추가 열
- **`대분류`**: 대분류명 (한국어)
- **`대분류 코드`**: 영문 알파벳 (A~U)
- **`세세분류 코드`**: 5자리 숫자

### UI 흐름
1. 엑셀 파일 업로드 (drag & drop 또는 클릭) — `.xlsx` / `.xls`
2. 분류명이 있는 열 선택 (드롭다운)
3. 자동 감지: 대분류 / 세세분류 판별 → 배지로 표시 + 수동 변경 가능
4. "변환 미리보기" (상위 5행)
5. "변환 결과 다운로드 (XLSX)" 버튼

## 수정 시 주의사항

- `DATA` 배열은 수만 건 이상의 KSIC 데이터 → 직접 편집 금지, grep으로만 확인
- `switchTab()` 수정 시 탭 배열 `['search', 'code', 'tree', 'excel']` 유지 필수
- SheetJS CDN 버전 `xlsx-0.20.3` 고정 (버전 업그레이드 시 API 호환성 확인 필요)
