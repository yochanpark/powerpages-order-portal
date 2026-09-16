# Power Pages 회원 포털 구축

Microsoft **Power Pages**로 만든 외부 사용자용 회원 포털이다.
Dataverse를 데이터 저장소로 쓰고, Power Automate 흐름이 주문 채번과 담당자 통보를 맡는다.

고객사 납품 건이라 산출물(managed solution, 이미지, 실데이터)은 올리지 않는다.
**무엇을 어떻게 만들었는지만 기록한 문서**다.

---

## 구성

| 영역 | 내용 |
|---|---|
| 포털 | Power Pages 사이트 — 웹페이지·템플릿·웹파일 등 구성요소 **267개** |
| 테마 | `theme.css` / `portalbasictheme.css` + Bootstrap. 로고·배경 이미지 22종 |
| 인증 | 로그인 페이지 커스터마이징(`Account/SignIn`), 로그인 안내 메일 템플릿 |
| 데이터 | Dataverse 테이블(게시자 접두어 `{PREFIX}_`) + 수식 열 정의 |
| 자동화 | 솔루션 내장 클라우드 흐름 **5개** + 별도 관리 흐름 **2개** |
| 기타 | PWA 매니페스트, `robots.txt`, 데이터 **마스킹 규칙 6종** |

솔루션은 **managed**로 1.0.0.1 → 1.0.0.3 → 1.0.0.4까지 버전을 올려 배포했다.

---

## 문서

| 문서 | 내용 |
|---|---|
| [1. 포털 구성](docs/01-portal-structure.md) | 사이트 구성요소, 테마, 인증 페이지 커스터마이징 |
| [2. 주문 자동화 흐름](docs/02-order-flows.md) | 주문번호 채번, 발주번호 갱신 시 관리자 통보 |
| [3. 솔루션 배포](docs/03-solution-packaging.md) | managed 솔루션 구성과 버전 관리 |

---

## 식별정보

| 자리표시자 | 원래 값의 성격 |
|---|---|
| `고객사` | 포털을 발주한 단체 |
| `{PREFIX}_` | Dataverse 게시자 접두어 |
| `{FLOW_ID}` | 클라우드 흐름 ID |
