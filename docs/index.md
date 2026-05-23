# index.html — 포털 진입 페이지

## 역할

- 비밀번호 인증 게이트 (`gallup1974`)
- 인증 성공 시 `sessionStorage.setItem('portal_auth', '1')`
- 3개 서비스 카드로 각 서브 페이지 네비게이션

## 화면 구조

```
#loginScreen  — 비밀번호 입력 화면 (미인증 시 표시)
#portalMain   — 서비스 선택 화면 (인증 후 표시)
  └── 3개 .nav-card: biz-tracker.html / biznoapi.html / ksic.html
```

## 핵심 JS 패턴

```javascript
const LOGIN_PWD = 'gallup1974';

// 로그인
function submitLogin() {
  const pwd = document.getElementById('loginPwd').value;
  if (pwd === LOGIN_PWD) {
    sessionStorage.setItem('portal_auth', '1');
    document.getElementById('loginScreen').style.display = 'none';
    document.getElementById('portalMain').style.display = 'block';
  } else {
    document.getElementById('loginError').style.display = 'block';
  }
}

// 서비스 카드 클릭 전 sessionStorage 재설정 (안전 장치)
function beforeNav() {
  sessionStorage.setItem('portal_auth', '1');
  return true;
}

// 이미 인증된 세션이면 로그인 화면 건너뜀
if (sessionStorage.getItem('portal_auth')) {
  document.getElementById('loginScreen').style.display = 'none';
  document.getElementById('portalMain').style.display = 'block';
}
```

## 버전 관리

### 구조
- `VERSION_HISTORY` 배열: 최신 버전이 **배열 맨 앞**
- 하단 좌측 fixed 버전 칩 (`.ver-chip#verChip`): 로그인 성공 후에만 `.visible` 클래스 추가로 노출
- 버전 칩 클릭 → 업데이트 내역 모달 (`.ver-modal-overlay`)

### 버전 업데이트 시
1. `VERSION_HISTORY` 배열 **맨 앞**에 새 항목 추가:
   ```javascript
   { version: 'x.x.x', date: 'YYYY-MM-DD', changes: ['변경사항 ...'] }
   ```
2. 버전 칩 HTML의 버전 텍스트 업데이트: `<button class="ver-chip" ...> v1.0.0`

### 작업 완료 후 버전 추천 방법
→ 루트 `CLAUDE.md`의 **"버전 관리 — 작업 완료 후 버전 추천 워크플로우"** 참조.
git diff로 변경 내역을 파악한 뒤 패치/마이너/메이저 중 추천을 제시하고, 사용자가 선택하면 업데이트한다.

### 버전 표시 조건
- `showVerChip()` — `submitLogin()` 성공 시 호출
- `showVerChip()` — 이미 `sessionStorage('portal_auth')` 있을 때도 호출
- 로그인 화면(`#loginScreen`) 상태에서는 버전 칩 숨김

## 수정 시 주의사항

- `beforeNav()` 함수는 각 `.nav-card`의 `onclick="return beforeNav()"` 에서 호출됨. 제거하지 말 것.
- 비밀번호 변경 시 `LOGIN_PWD` 상수 1곳만 수정.
- localStorage 사용 금지 — sessionStorage만 허용.
