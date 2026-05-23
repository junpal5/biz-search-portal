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

## 수정 시 주의사항

- `beforeNav()` 함수는 각 `.nav-card`의 `onclick="return beforeNav()"` 에서 호출됨. 제거하지 말 것.
- 비밀번호 변경 시 `LOGIN_PWD` 상수 1곳만 수정.
- localStorage 사용 금지 — sessionStorage만 허용.
