# 4. 인프라

환경 구성, 솔루션 패키징, 개발 → 운영 이관, 사이트 보안 설정이다.

## 환경

| 환경 | 유형 | 지역 | 비고 |
|---|---|---|---|
| 개발 | 개발자 환경 | 미국 | 사이트는 평가판(개발자 웹사이트) |
| 운영 | 프로덕션(Dataverse 포함) | 미국 | 이관 중 **새로 만들었다** — 아래 |

## 솔루션 패키징

**managed 솔루션**(`solution.xml`의 `<Managed>1</Managed>`)으로 내보냈다.
managed는 **같은 버전으로 덮어쓸 수 없어** 수정할 때마다 버전을 올려 다시 내보냈다.

```
1.0.0.1   →   1.0.0.3   →   1.0.0.4
```

`1.0.0.4` 기준 313개 파일, 압축을 풀면 22.8MB(zip 8.7MB)다.

```
solution.xml               솔루션 정의·구성요소 목록        (134KB)
customizations.xml         테이블·폼·뷰 등 스키마           (1.8MB)
Formulas/                  수식 열 정의
Workflows/                 클라우드 흐름 5개 (JSON)
powerpagecomponents/       포털 구성요소 267개 + 웹파일
maskingrules/              Power Pages 기본 마스킹 규칙 6종
Assets/ · metadataforarchivals/ · [Content_Types].xml
```

- 포털 구성요소는 `powerpagecomponents/<GUID>/` 아래 XML로, 웹파일은 실제 파일이 `filecontent/`에 함께 나온다.
  사이트 전체를 솔루션에 담아 환경 간에 옮길 수 있다
- `customizations.xml` 한 파일이 1.8MB — 테이블 스키마와 폼·뷰 정의가 전부 한 덩어리로 들어간다
- `maskingrules/`의 6종(`Email`·`SocialSecurityNumber`·`Date_Hyphen` 등)은 **Power Pages 기본 제공 규칙**이 내보낼 때 따라온 것이다.
  솔루션 내용물을 볼 때 어디까지가 플랫폼 기본값인지 구분해 둔다

### 흐름을 따로 내보낼 때

채번·통보 흐름은 별도 zip(`Microsoft.Flow/flows/<GUID>/definition.json`)으로도 내보냈다.
매니페스트에서 흐름은 `Update`, API·연결은 `Existing` —
**연결은 패키지에 담기지 않고 가져오는 환경의 기존 연결에 붙는다.**
흐름만 옮길 때는 대상 환경에 Dataverse·Office 365 연결이 먼저 있어야 한다.

## 개발 → 운영 이관

목표는 하나였다 — **사이트를 공개(Public)로 바꿔 URL만으로 로그인 화면에 닿게 하는 것.**

### 개발 환경에서는 공개로 바꿀 수 없다

```
액세스 제한 비프로덕션 사이트의 사이트 표시 유형을 공용으로 변경할 수 없습니다.
```

사이트가 평가판이라 **프로덕션으로 변환**해야 공개가 열리는데, 변환도 막혔다.
Dataverse 환경 자체가 **개발자 환경**이라 프로덕션 사이트를 둘 수 없었다.
**환경을 옮기는 것**이 선결 과제가 됐다.

### 시도 1 — 파이프라인

배포를 누르기 전에 파이프라인이 잡은 **솔루션의 개체 목록부터 열어 봤다.**
솔루션 이름은 사이트와 같았지만 안에는 주문 테이블·흐름·연결 참조 6개뿐,
**웹사이트·웹 페이지·웹 역할 같은 사이트 구성요소는 하나도 없었다.**
그대로 배포했으면 사이트 없이 테이블만 올라갔을 것이다.
사이트 구성요소를 직접 추가해 배포했지만 환경 설정 오류로 실패했다.

```
0x80040265 - Issue detected with the environment configuration for this pipeline.
```

대상 환경을 관리형 환경으로 바꾸고 파이프라인을 다시 만들어도 같은 오류였다.

### 시도 2 — managed 솔루션 직접 가져오기

```
Failed to resolve app information for following apps: PowerPages_Core
```

대상 환경에 Power Pages가 설치된 적이 없었고, 빈 사이트를 만들어 설치하려 하자
**"환경 영역(지역)을 지원하지 않는다"**는 메시지가 나왔다.
기존 운영 환경은 국내 지역, 개발 환경은 미국 지역이었다.

### 해결 — 같은 지역에 운영 환경을 새로 만들었다

환경의 지역은 만든 뒤에 바꿀 수 없다. 개발 환경과 **같은 미국 지역**에
Dataverse를 포함한 프로덕션 환경을 새로 만들고 managed 솔루션을 가져왔다.
가져오기가 성공했고, 사이트를 프로덕션으로 변환한 뒤 공개로 바꿨다.

> 새 환경에서 처음 만든 빈 사이트도 공개가 막혀 있었다.
> 지역과 별개로 **사이트는 항상 평가판으로 시작하고, 변환을 거쳐야 공개가 열린다.**

### 솔루션에 안 따라오는 것들

| 빠진 것 | 증상 | 조치 |
|---|---|---|
| 채번·통보 흐름 2개 | 솔루션에 없었다 | 솔루션에 추가해 다시 내보내고 가져왔다 |
| 채번 설정 테이블의 **행** | 채번 흐름 조건이 항상 거짓 — 조회 결과 0건 | 개발 환경의 설정 행 3개(유형별 접두어·현재 번호)를 운영에 입력 |
| 관리자 계정 | — | 포털에서 가입한 연락처에 관리자 웹 역할을 연결 |

**솔루션은 구조만 옮기고 데이터는 옮기지 않는다.**

## 사이트 보안 설정

운영 중 장애와 Site Checker 점검을 거쳐 정한 최종 값이다. 경위는 [5. 유지보수](05-maintenance.md)에 있다.

| 사이트 설정 | 값 |
|---|---|
| `HTTP/Content-Security-Policy` | 아래 |
| `HTTP/X-Content-Type-Options` | `nosniff` |
| `Webapi/{PREFIX}_digitalprintingorder/fields` | 와일드카드 대신 페이지가 쓰는 열만 쉼표로 나열 |

```
script-src t1.daumcdn.net content.powerapps.com 'self' 'unsafe-inline';
style-src  content.powerapps.com 'self' 'unsafe-inline'
```

- **실제로 적용되는 CSP는 Studio 화면이 아니라 사이트 설정 레코드다.** Studio에는 추가한 외부 도메인만 보였다
- `content.powerapps.com`은 Power Pages가 jQuery·Bootstrap 등을 받아 오는 플랫폼 도메인이다. CSP를 직접 쓰면 이것까지 넣어야 한다
- Web API 필드 목록은 **대소문자까지 그대로 비교한다.** 조회 열은 논리 이름(`{PREFIX}_account`)과
  탐색 속성 이름(`{PREFIX}_Account`, 저장 시 `@odata.bind`)을 **둘 다** 넣는다
