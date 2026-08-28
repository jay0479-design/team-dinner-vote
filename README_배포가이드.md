# 회식 날짜 설문 달력 — GitHub Pages 배포 가이드

정적 페이지(GitHub Pages)에서는 서버가 없으므로, 투표 공유를 위해 무료 백엔드인
**Firebase Realtime Database**를 사용합니다. 아래 순서대로 진행하세요.

---

## 1단계. Firebase 프로젝트 만들기 (약 5분)

1. https://console.firebase.google.com 접속 → **프로젝트 추가**
2. 프로젝트 이름 입력 (예: `team-dinner-vote`) → 애널리틱스는 꺼도 됨 → 생성
3. 왼쪽 메뉴 **빌드 > Realtime Database** → **데이터베이스 만들기**
   - 위치: `asia-southeast1` 등 아무거나
   - 보안 규칙: 일단 **테스트 모드**로 시작
4. **규칙(Rules)** 탭에서 아래로 교체 후 게시:

```json
{
  "rules": {
    "dinner-2026-09": {
      ".read": true,
      ".write": true
    },
    "$other": { ".read": false, ".write": false }
  }
}
```

> ⚠️ 이 규칙은 링크를 아는 누구나 읽기/쓰기가 가능합니다.
> 사내 간단 설문 용도로는 충분하지만, 민감한 정보는 절대 저장하지 마세요.
> 설문이 끝나면 `.write`를 `false`로 바꿔 결과를 잠글 수 있습니다.

## 2단계. 웹 앱 등록 및 구성값 복사

1. 프로젝트 개요 옆 ⚙️ → **프로젝트 설정** → 일반 탭 아래 **내 앱** → 웹(`</>`) 추가
2. 앱 닉네임 입력 → 등록 (호스팅 체크 불필요)
3. 표시되는 `firebaseConfig` 값 중 아래 5개를 복사

## 3단계. index.html에 구성값 입력

`index.html` 상단 스크립트의 `FIREBASE_CONFIG`를 본인 값으로 교체:

```javascript
var FIREBASE_CONFIG={
  apiKey:"AIza...",
  authDomain:"team-dinner-vote.firebaseapp.com",
  databaseURL:"https://team-dinner-vote-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId:"team-dinner-vote",
  appId:"1:1234:web:abcd"
};
```

> 참고: Firebase 웹 apiKey는 비밀키가 아니라 프로젝트 식별자로,
> 클라이언트 코드에 노출되는 것이 정상입니다. 접근 제어는 위의 DB 규칙이 담당합니다.
> 단, `databaseURL`은 콘솔에 표시된 본인 리전 주소를 그대로 써야 합니다.

## 4단계. GitHub에 올리고 Pages 켜기

```bash
# 기존 저장소에 추가하는 경우
git add index.html README_배포가이드.md
git commit -m "feat: 9월 회식 날짜 설문 달력 추가"
git push origin main
```

새 저장소라면:

```bash
git init
git add .
git commit -m "init: 회식 날짜 설문 달력"
git branch -M main
git remote add origin https://github.com/<본인계정>/<저장소명>.git
git push -u origin main
```

GitHub 저장소 → **Settings > Pages** →
Source: `Deploy from a branch`, Branch: `main` / `(root)` → Save.
1~2분 후 `https://<본인계정>.github.io/<저장소명>/` 에서 접속 가능합니다.

## 5단계. 동작 확인

- 페이지 상단에 빨간 "Firebase 설정이 필요합니다" 경고가 보이면 → 3단계 미완료
- 결과 영역에 "실시간 연결됨" + 빨간 점이 깜빡이면 → 정상
- 브라우저 두 개를 열고 한쪽에서 제출하면 다른 쪽에 **즉시** 반영되는지 확인

---

## 기능 요약

| 기능 | 동작 |
|---|---|
| 날짜 선택 | 2026년 9월 평일만 선택 가능. 주말·추석 연휴(9/24~26) 비활성 |
| 제출/수정 | 이름 입력 후 제출. 기존 응답자는 자동으로 불러와 "응답 수정"으로 전환 |
| 응답 취소 | 본인 응답 삭제 (확인창 후) |
| 실시간 | Firebase 리스너로 폴링 없이 즉시 반영. 변경된 날짜 셀 하이라이트 |
| 결과 | 득표순 막대그래프, 날짜별 가능 인원 명단, 최다 득표일 강조 |

## 알려진 한계 (설계상 의도된 트레이드오프)

- 로그인이 없어 **이름만으로 식별**합니다. 동명이인 구분 불가, 타인 이름으로 수정/삭제 가능.
- DB 규칙이 공개 쓰기라서 링크가 외부에 퍼지면 누구나 참여할 수 있습니다.
- 사내 인증이 필요해지면 Firebase Anonymous/Google Auth 추가가 다음 단계입니다.

## 디자인 출처

모든 컬러·타이포·radius 값은 Figma `Component` 기준점(node `2031-6499`)에서
추출한 토큰만 사용했습니다. (Primary/Pm1 `#e4002b`, Pretendard 등)
