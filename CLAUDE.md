# biz-search-portal 통합 포털 가이드

## 프로젝트 개요

- **앱 이름**: 사업체 정보 검색 통합 포털
- **배포**: GitHub Pages — `https://junpal5.github.io/biz-search-portal/`
- **레포**: `junpal5/biz-search-portal`
- **구조**: 빌드 시스템 없음. 4개 HTML 파일 직접 편집 후 커밋/푸시

### 통합된 서비스 (원본 레포 → 포털 파일명)
| 원본 레포 | 포털 파일 | 서비스 설명 |
|---|---|---|
| `junpal5/biz-tracker` | `biz-tracker.html` | 사업자 휴·폐업 조회 (국세청 NTS API) |
| `junpal5/biz-tracker-biznoapi` | `biznoapi.html` | 비즈노 기반 사업자 상세정보 조회 |
| *(업로드된 HTML)* | `ksic.html` | 한국표준산업분류(KSIC) 검색 + 엑셀 변환 |
| *(신규)* | `index.html` | 포털 진입 / 비밀번호 인증 / 서비스 선택 |

---

## 인증 시스템 (CRITICAL)

### ⚠️ localStorage 사용 절대 금지
인증 상태는 **반드시 `sessionStorage`** 만 사용한다. `localStorage`는 절대 사용하지 않는다.

### 인증 흐름
```
index.html (포털)
  → 비밀번호 'gallup1974' 입력
  → sessionStorage.setItem('portal_auth', '1')
  → 3개 서비스 카드 노출

각 서비스 페이지 (biz-tracker.html / biznoapi.html / ksic.html)
  → 페이지 로드 즉시 sessionStorage.getItem('portal_auth') 체크
  → 없으면 window.location.replace('index.html') 리다이렉트
```

### 각 서비스 페이지 상단 JS (공통 패턴)
```javascript
(function() {
  if (!sessionStorage.getItem('portal_auth')) {
    window.location.replace('index.html');
  }
})();
```

### index.html 핵심 코드
```javascript
const LOGIN_PWD = 'gallup1974';

function submitLogin() {
  if (pwd === LOGIN_PWD) {
    sessionStorage.setItem('portal_auth', '1');
    // 로그인 화면 숨기고 포털 메인 표시
  }
}

function beforeNav() {
  // 서비스 카드 클릭 직전 sessionStorage를 확실히 설정
  sessionStorage.setItem('portal_auth', '1');
  return true;
}

// 이미 인증된 세션이면 로그인 화면 건너뜀
if (sessionStorage.getItem('portal_auth')) { ... }
```

---

## 파일 구조

```
biz-search-portal/
├── index.html        # 포털 진입 (비밀번호 인증 + 서비스 네비게이션)
├── biz-tracker.html  # 사업자 휴폐업 조회
├── biznoapi.html     # 비즈노 기반 사업자 상세정보 조회
└── ksic.html         # 한국표준산업분류 검색 + 엑셀 변환
```

각 파일은 CSS·JS 모두 인라인 단일 HTML. 편집 시 전체 Read 금지 — `offset/grep`으로 필요한 부분만 읽는다.

---

## Git 워크플로우

### ⚠️ 중요 주의사항
- **git commit signing 비활성화 필수**: `git config commit.gpgsign false`
- **MCP GitHub `create_pull_request` 403 오류**: PAT를 사용한 curl로 PR 생성
- **브랜치 히스토리**: main과 feature 브랜치는 공통 조상을 공유해야 PR 생성 가능

### PAT 설정
```bash
git remote set-url origin https://ghp_<PAT>@github.com/junpal5/biz-search-portal.git
git config commit.gpgsign false
```

### 브랜치 기반 작업 (권장)
```bash
# feature 브랜치 생성 (반드시 main에서 분기)
git checkout main && git pull origin main
git checkout -b feature/my-change

# 작업 후
git add <파일>
git commit -m "설명"
git push -u origin feature/my-change
```

### PR 생성 (curl 사용 — MCP 사용 불가)
```bash
curl -X POST \
  -H "Authorization: token ghp_<PAT>" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/junpal5/biz-search-portal/pulls \
  -d '{
    "title": "PR 제목",
    "head": "feature/my-change",
    "base": "main",
    "body": "설명"
  }'
```

### main 직접 push (긴급 수정 시)
```bash
git add <파일>
git commit -m "fix: 설명"
git push origin main
```

---

## 각 서비스 페이지 요약

각 페이지 상세 내용은 `docs/` 디렉토리 참조:
- `docs/index.md` — 포털 진입 페이지
- `docs/biz-tracker.md` — 사업자 휴폐업 조회
- `docs/biznoapi.md` — 비즈노 기반 사업자 상세정보 조회
- `docs/ksic.md` — 한국표준산업분류 검색 + 엑셀 변환

---

## 버전 관리

버전 칩(`.ver-chip`)이 있는 파일: `index.html`, `biz-tracker.html`, `biznoapi.html`  
(`ksic.html`은 버전 칩 없음)

### 업데이트 절차
1. 해당 HTML 내 `VERSION_HISTORY` 배열 **맨 앞**에 새 항목 추가
2. `.ver-chip` HTML의 버전 텍스트 업데이트

### 작업 완료 후 버전 추천 워크플로우 (절대 생략 금지)

Claude Code로 작업 완료 후, 사용자가 버전 업데이트를 요청하면:

**1단계 — 변경 내역 파악**
```bash
git diff HEAD~1 HEAD -- <파일명>   # 직전 커밋과의 diff
git log --oneline -5               # 최근 커밋 목록
```

**2단계 — 버전 유형 추천**

| 선택 | 버전 변화 | 적합한 경우 |
|------|-----------|-------------|
| 패치 | x.x.**+1** | 오탈자·스타일 미세 조정·사소한 버그 수정 |
| 마이너 | x.**+1**.0 | 새 기능 추가·기존 기능 개선·UI 변경 |
| 메이저 | **+1**.0.0 | 전체 구조 변경·대규모 리디자인·아키텍처 개편 |
| 버전 유지 | 변경 없음 | 임시 수정·테스트·문서만 수정 |

**3단계 — 사용자에게 제시**

변경 내역을 검토한 뒤 아래 형식으로 추천을 제시한다:
```
이번 작업 내역: [변경 요약]
추천: 마이너 업데이트 (v1.0.0 → v1.1.0)
이유: [새 기능/개선 내용이므로]

다른 선택지: 패치(v1.0.1) / 메이저(v2.0.0) / 버전 유지
```

사용자가 선택하면 즉시 `VERSION_HISTORY` 배열과 버전 칩 텍스트를 업데이트하고 커밋한다.

---

## GitHub Pages 배포

main 브랜치 push → GitHub Pages 자동 빌드 (약 1~2분 소요).
설정: Settings → Pages → Source: main branch / root.
