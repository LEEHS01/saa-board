# SAA 스터디 상황판 — 설치 안내

`index.html` 파일 하나가 전부입니다. 아래 순서대로 하면 조원들이 **계정 없이 주소만으로** 보고,
조장이 저장하면 **모두의 화면에 즉시** 반영됩니다.

전체 20~30분 정도 걸립니다. 비용은 없습니다 (Firebase 무료 요금제 안에서 충분).

---

## 1. Firebase 프로젝트 만들기

1. https://console.firebase.google.com 접속 → 구글 계정으로 로그인
2. **프로젝트 만들기** 클릭
3. 이름: `saa-study-board` (아무거나 가능)
4. **Google 애널리틱스는 사용 안 함**으로 끄기 → 만들기

## 2. 웹 앱 등록하고 설정값 복사

1. 프로젝트 개요 화면에서 **`</>`** (웹) 아이콘 클릭
2. 앱 닉네임: `board` → **앱 등록**
3. 화면에 나오는 `firebaseConfig` 값을 복사

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "saa-study-board.firebaseapp.com",
  projectId: "saa-study-board",
  ...
};
```

4. `index.html`을 메모장으로 열어 맨 위 `FIREBASE_CONFIG` 의 `"여기에-붙여넣기"` 자리를
   위 값으로 각각 바꿔 저장

> **apiKey는 비밀번호가 아닙니다.** 브라우저에 노출되는 게 정상이고, 공개 저장소에 올라가도 괜찮습니다.
> 실제 보안은 아래 4번의 Firestore 규칙이 담당합니다.

## 3. Firestore 데이터베이스 만들기

1. 왼쪽 메뉴 **빌드 → Firestore Database** → **데이터베이스 만들기**
2. 위치: **asia-northeast3 (서울)**
3. **프로덕션 모드에서 시작** 선택 → 만들기
4. 상단 **규칙** 탭으로 이동해서 전체를 아래로 교체하고 **게시**

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /board/state {
      allow read: if true;
      allow write: if request.auth != null
                   && request.auth.token.email == '조장-구글계정@gmail.com';
    }
  }
}
```

> `조장-구글계정@gmail.com` 자리에 **조장이 로그인할 구글 계정 주소**를 넣으세요.
> 읽기는 전체 공개, 쓰기는 이 계정 하나만 됩니다.
> 나중에 편집자를 늘리려면 `in ['a@gmail.com', 'b@gmail.com']` 형태로 바꾸면 됩니다.

## 4. 구글 로그인 켜기

1. 왼쪽 메뉴 **빌드 → Authentication** → **시작하기**
2. 로그인 방법에서 **Google** 선택 → 사용 설정 → 지원 이메일 고르고 **저장**

## 5. GitHub Pages에 올리기

1. https://github.com 로그인 → 우측 상단 **+** → **New repository**
2. 이름: `saa-board`, **Public** 선택 → **Create repository**
3. **Add file → Upload files** → `index.html` 끌어다 놓기 → **Commit changes**
4. **Settings → Pages**
   - Source: **Deploy from a branch**
   - Branch: **main** / **(root)** → **Save**
5. 1~2분 뒤 주소가 뜹니다:
   `https://<깃허브아이디>.github.io/saa-board/`

## 6. 승인된 도메인 추가 (로그인이 되게)

1. Firebase 콘솔 → **Authentication → 설정 → 승인된 도메인**
2. **도메인 추가** → `<깃허브아이디>.github.io` 입력 → 추가

## 7. 초기 데이터 올리기

1. 5번의 주소를 열기
2. 우측 상단 **조장 로그인** → 구글 로그인
3. **초기 데이터 올리기** 버튼이 보이면 클릭 (지금까지의 출결 기록이 올라갑니다)
4. 버튼이 사라지고 **편집** 버튼이 나오면 완료

## 8. 조원들에게 주소 공유

5번 주소만 알려주면 됩니다. **계정도, 로그인도 필요 없습니다.**

---

## 쓰는 법

- **우측 상단 점** — 초록: 실시간 연결됨 / 노랑: 저장 중 / 빨강: 연결 끊김
- **편집** → 출결 체크·사유 입력 → **저장하기 · N건**
- 저장하면 열려 있는 모든 화면이 **새로고침 없이 즉시** 바뀝니다
- 저장 안 한 변경이 있는 동안에는 남이 저장해도 내 입력이 덮어써지지 않습니다

## 내용 수정

일정표 진도·소요·비고, 휴무 전환, 다음 회차로 이월, 인원 탈퇴·합류, 정기 불참 등록은
전부 화면 안에서 됩니다. 화면 구조 자체(열 추가, 새 기능)를 바꿀 때만 `index.html`을 고쳐서
GitHub에 다시 업로드하면 됩니다 — 데이터는 Firestore에 있으니 안 날아갑니다.

## 문제가 생기면

| 증상 | 확인할 것 |
| --- | --- |
| 빨간 점 + "Firebase 설정 필요" | 2번에서 설정값을 안 넣었거나 오타 |
| 로그인 팝업이 막힘 | 브라우저 팝업 차단 해제 |
| 로그인 후 `auth/unauthorized-domain` | 6번의 승인된 도메인 추가 누락 |
| 저장 시 "저장 권한이 없습니다" | 3번 규칙의 이메일과 로그인한 계정이 다름 |
| 주소가 404 | Pages 배포에 1~2분 걸립니다. 폴더 없이 루트에 `index.html`이 있는지 확인 |
