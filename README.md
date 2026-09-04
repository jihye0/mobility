# K-Mobility 브릿지 재단 | 협력사 지원사업 대시보드 (GitHub Pages 버전)

기존 Apps Script 웹앱을 **화면(GitHub Pages) + 데이터 API(Apps Script)** 구조로 분리한 버전입니다.

```
[방문자 브라우저]
   └─ GitHub Pages: index.html + config.js   ← 화면(정적)
        └─ fetch ─▶ Apps Script 웹앱(Api.gs, 새 프로젝트)  ← 구글시트 읽기 / 조회수 기록
                        └─ Google Sheets (지원사업 시트)
```

| 파일 | 위치 | 역할 |
|---|---|---|
| `Api.gs` | **새** Apps Script 프로젝트 | JSON API (데이터 조회, 조회수·원문보기수 +1) |
| `index.html` | GitHub 저장소 | 대시보드 화면 (기존 디자인 그대로) |
| `config.js` | GitHub 저장소 | Apps Script 웹앱 URL 한 줄 설정 |
| `.nojekyll` | GitHub 저장소 | GitHub Pages가 파일을 그대로 서빙하도록 하는 빈 파일 |

---

## 1단계. Apps Script를 API로 배포 (기존 프로젝트는 그대로 둠)

기존 Apps Script 프로젝트(Code.gs + Index.html)는 **전혀 건드리지 않습니다.**
새 프로젝트를 하나 더 만들어 API 전용으로 배포합니다.

1. https://script.google.com 접속 → **새 프로젝트**
2. 왼쪽 파일 목록의 `Code.gs` 를 클릭해 이름을 **`Api`** 로 바꿉니다 (아무 이름이나 가능, `.gs`는 자동).
3. 내용을 이 폴더의 **`Api.gs`** 로 전부 교체하고 저장합니다.
   - 시트 ID(`API_SPREADSHEET_ID`)는 기존과 동일하게 들어 있습니다.
4. 상단 프로젝트 이름(제목 없는 프로젝트)을 예: `지원사업 API` 로 바꿉니다.
5. **배포 → 새 배포**
   - 유형 선택(⚙️): **웹 앱**
   - 다음 사용자 인증 정보로 실행: **나**
   - 액세스 권한이 있는 사용자: **모든 사용자** (익명 포함) ← 반드시 이 값
   - 처음 배포 시 구글 계정 권한 승인 창이 뜹니다 (시트 접근 허용).
6. 배포 후 나오는 **웹 앱 URL** 을 복사합니다.
   `https://script.google.com/macros/s/AKfycb.../exec` 형태이며, 기존 웹앱 URL과 다른 새 주소입니다.
7. 브라우저에서 `웹앱URL?action=data` 를 열어 JSON이 보이면 성공입니다.

> **왜 새 프로젝트인가?** Apps Script는 한 프로젝트의 모든 `.gs` 파일이 함수 이름을 공유합니다.
> 기존 프로젝트에 `Api.gs`를 추가하면 `doGet` 이 두 개가 되어 어느 쪽이 실행될지 보장되지 않습니다.
> (그래도 만약을 위해 `Api.gs` 내부 함수·상수는 모두 `api`/`API_` 접두어를 써서 기존 코드와 겹치지 않게 했습니다.)

> **Api.gs를 수정할 때마다** 배포 → 배포 관리 → ✏️ → 버전: **새 버전** → 배포 를 해야 반영됩니다.
> (URL은 그대로 유지됩니다.)

### API 요약

| 요청 | 설명 |
|---|---|
| `GET ?action=data` | 관리자 점검 = O 인 지원사업 목록 (3분 캐시) |
| `GET ?action=view&key=<사업명 또는 원문URL>` | 조회수 +1 |
| `GET ?action=original&key=<사업명 또는 원문URL>` | 원문보기수 +1 |
| `GET ?action=refresh` | 캐시 비우고 목록 다시 생성 |

---

## 2단계. config.js 에 API URL 입력

`config.js` 를 열어 1단계에서 복사한 URL을 붙여넣습니다.

```js
window.APP_CONFIG = {
  API_URL: "https://script.google.com/macros/s/AKfycb.../exec"
};
```

---

## 3단계. GitHub 저장소 만들고 올리기

1. GitHub 로그인 → **New repository**
   - 이름 예: `support-programs` (아무 이름 가능, 소문자 권장)
   - **Public** 선택 (무료 GitHub Pages는 Public 저장소만 가능)
   - Create repository
2. **Add file → Upload files** 로 아래 4개 파일을 올립니다.
   - `index.html`
   - `config.js`
   - `.nojekyll`
   - `README.md` (선택)
3. **Commit changes** 클릭.

> 이후 화면을 수정할 땐 저장소에서 `index.html` 을 열어 ✏️ 편집 → Commit 하면 1~2분 내에 반영됩니다.

---

## 4단계. GitHub Pages 켜기

1. 저장소 → **Settings → Pages**
2. Build and deployment
   - Source: **Deploy from a branch**
   - Branch: **main** / **/(root)** → Save
3. 1~3분 후 상단에 주소가 표시됩니다.
   `https://<계정명>.github.io/<저장소명>/`

이 주소가 방문자에게 공유할 새 대시보드 URL입니다.

---

## 확인 및 문제 해결

| 증상 | 원인 / 해결 |
|---|---|
| 화면 상단에 "API 연결 설정이 필요합니다" 노란 안내 | `config.js` 의 `API_URL` 이 아직 예시 문구입니다. 웹앱 URL로 바꾸고 Commit. |
| "API 응답이 JSON 형식이 아닙니다" | 웹앱 배포의 액세스 권한이 **모든 사용자** 가 아니면 구글 로그인 페이지(HTML)가 반환됩니다. 배포 관리에서 액세스를 바꾸고 새 버전으로 재배포. |
| 데이터가 옛것으로 보임 | Apps Script 캐시 3분 + 브라우저 새로고침. 즉시 반영은 `웹앱URL?action=refresh` 한 번 호출. |
| 조회수가 시트에 안 올라감 | 시트 1행에 `조회수`, `조회수_마지막기록`, `원문보기수`, `원문보기_마지막기록` 컬럼이 있는지 확인. F12 → Console 에 실패 사유가 표시됩니다. |
| 페이지 404 | Pages 활성화 후 몇 분 대기. 파일명이 정확히 `index.html` (소문자)인지 확인. |

### 바뀐 점 (기존 Apps Script 버전 대비)

- 기존 프로젝트는 그대로 두고, 새 프로젝트의 `doGet()` 이 HTML 대신 **JSON** 을 반환합니다 (`ContentService`).
- 새 API가 안정적으로 동작하는 것을 확인한 뒤, 기존 웹앱은 원할 때 배포 해제하면 됩니다.
- `google.script.run` 호출이 모두 **`fetch(API_URL?...)`** 로 바뀌었습니다.
- `<?!= initialDataJson ?>` 템플릿 문법이 제거되고, 페이지 로드 시 API에서 데이터를 받아옵니다.
- 디자인·필터·정렬·모달·모바일 CSS 등 나머지 동작은 그대로입니다.
- 사용자 지정 도메인(예: `support.kmbridge.org`)이 필요하면 Settings → Pages → Custom domain 에서 설정할 수 있습니다.
