# 2. 코드 작성

페이지는 Power Pages VS Code 확장으로 로컬에서 고치고 업로드하는 방식으로 관리했다.
직접 작성한 CSS·Liquid·JavaScript·흐름 식의 요지를 적는다.

## 로그인 화면 — CSS 재정의

Power Pages는 기본 인증 화면(`Account/SignIn`)의 마크업을 바꿀 수 없다.
CSS로 덮는 것 말고 선택지가 없어서, 세 가지를 했다.

**1. 기본 UI 요소를 숨긴다**

```css
.page-heading,.nav-tabs,.checkbox,h2,.required,
.field-validation-error,
form a[href*="ForgotPassword"]{display:none!important}
```

**2. 외부 인증 수단을 감춘다**

```css
button[value="MicrosoftEntraID"],button[name*="Microsoft"],
a[href*="MicrosoftEntra"],a[href*="External"]{display:none!important}
```

> 이건 **화면에서 감춘 것이지 비활성화한 것이 아니다.**
> 인증 수단을 실제로 막으려면 사이트 설정에서 해당 공급자를 꺼야 한다.

**3. 카드형 레이아웃으로 다시 짠다**

```css
.signin-card{max-width:520px;margin:0 auto;background:#fff;
  border-radius:14px;padding:50px 45px;
  box-shadow:0 4px 20px rgba(0,0,0,.1)}
```

Bootstrap의 `.row` 음수 마진이 카드 밖으로 삐져나와,
`margin-left/right:0!important`와 `box-sizing:border-box`를 여러 선택자에 걸어 눌렀다.
라벨도 테마에 따라 숨겨져서 `display:block` + `visibility:visible` + `opacity:1`로 되살렸다.
기본 테마 위에 `!important`를 겹겹이 쌓는 방식이라 깔끔하지는 않다.

## 주문 조회 — Liquid 서버 렌더링

클라이언트 Web API 조회를 대신해, 서버에서 주문을 읽어 data 속성에 심는다([1. 설계](01-design.md)).

```liquid
{% fetchxml order_q %}
<fetch top="1">
  <entity name="{PREFIX}_digitalprintingorder">
    <attribute name="{PREFIX}_ordernumber" /> ...
    <filter><condition attribute="{PREFIX}_digitalprintingorderid"
                       operator="eq" value="{{ request.params['id'] }}" /></filter>
  </entity>
</fetch>
{% endfetchxml %}
<div id="serverOrderData" hidden data-ordernumber="{{ o.{PREFIX}_ordernumber }}" ...></div>
```

상세 화면, 인쇄 화면, 주문 목록의 진행 상태 필터까지 같은 방식으로 바꿨다.

### 옮기면서 걸린 것

| 문제 | 원인 | 조치 |
|---|---|---|
| 날짜가 `Y-0-11`로 표시 | Liquid `date` 필터가 이 값에 기대대로 동작하지 않음 | 필터를 빼고 원본 값을 넘김 |
| 원본 값이 `2026-08-11 오전 12:00:00` | 서버가 한국어 로캘 문자열로 렌더링, JS `new Date()`가 못 읽음 | 공백 앞 `yyyy-MM-dd`만 잘라 씀 |
| 인쇄 시간이 비어 보임 | 같은 로캘 문자열(`오전 1:18:31`)을 `Date`로 파싱 실패 | 오전/오후를 직접 분해하는 파서를 만들고, 저장값이 UTC라 표시할 때 +9시간 |
| 목록 전체를 JSON으로 심었더니 파싱 오류 | 출력해 보니 `json` 필터가 **값에 따옴표를 붙이지 않았다** (`"id": 76d0…`) | 값을 직접 큰따옴표로 감싸고 `replace`로 따옴표만 이스케이프 |
| 그 `replace`가 Liquid 오류 | 첫 인자를 정규식으로 해석해 백슬래시 하나가 잘못된 패턴이 됨 | 백슬래시 이스케이프는 빼고 따옴표만 처리 |
| 진행 상태 필터가 안 먹음 | 선택 항목(OptionSet) 열은 `.value`를 붙여야 숫자 값이 나옴 | `{{ o.{PREFIX}_delivery.value }}` |

## 프로필 — 기본 제공 페이지에 스크립트 붙이기

`/profile`은 Power Pages가 기본 제공하는 페이지라 **편집할 페이지 소스가 비어 있다.**
그래서 사이트 전체에 깔리는 `Footer` 웹 템플릿에 스크립트를 넣고,
경로가 `/profile`일 때만 동작하게 했다.

- 대상 입력란은 연락처 표준 열 id(`address1_line1`, `address1_postalcode`, `telephone1`)로 찾는다
- 폼이 늦게 그려질 수 있어 **0.5초 간격으로 최대 15초까지** 다시 찾는다
- 화면 크기 조정 CSS는 `body[data-sitemap-state*="/profile"]`로 범위를 한정해 다른 페이지에 새지 않게 했다
- 저장 후 이동: 저장 성공 알림을 감지하면 로그아웃 URL에 `returnUrl`로 로그인 화면을,
  그 뒤에 다시 `ReturnUrl`로 주문확인 페이지를 이어 붙였다.
  처음엔 로그인 화면으로 보내기만 했는데 세션이 남아 로그인된 상태로 보였기 때문이다

## 주문서 저장

| 변경 | 이유 |
|---|---|
| 새 주문 저장에 `Prefer: return=representation` | 응답 헤더에서 새 행 id를 못 뽑아 다음 요청이 `(...)()`로 나가 404가 났다. 응답 본문에서 id를 받는다 |
| 내지 사양을 드롭다운으로 바꾸면서 조회 코드도 `setValue` → `setSelect` | 입력 방식이 바뀌면 값을 채우는 쪽도 같이 바꿔야 한다 |
| 지운 입력란의 JS 참조·저장 payload 열 정리 | HTML만 지우고 JS를 남기면 빈 값으로 처리돼 에러는 안 나지만, 남은 참조는 함께 정리했다 |

> 한 번은 입력란을 정리하면서 **페이지 하단 스크립트 전체(데이터 조회·제출)가 통째로 빠진 채** 반영됐다.
> 파일을 통째로 바꿀 때는 스크립트 블록이 남아 있는지부터 확인한다.

### 거래처는 거래처 테이블에서 (2026-05-12)

주문서의 거래처 목록을 주문 테이블 안의 값에서 모으고 있었다. 거래처 테이블(`accounts`)에서 읽어
조회 열로 연결하도록 바꿨다.

```javascript
// 목록
fetch("/_api/accounts?$select=accountid,name")
// 저장 — 조회 열은 탐색 속성 이름으로 연결
payload["{PREFIX}_Account@odata.bind"] = "/accounts(" + accountId + ")";
```

처음엔 저장이 403(`90040106`, 거래처를 주문과 **연결할 권한 없음**)으로 막혔다.
조회 열을 연결하려면 주문 테이블뿐 아니라 **대상 테이블(거래처)에도** 테이블 권한과 Web API 사이트 설정이 있어야 한다.

## 주문 목록 — 상태 필터와 페이징 (2026-05-13 ~ 15)

주문현황 열 머리에 `전체 / 진행중 / 완료` 토글을 두고, 누르면 그 상태만 **1페이지부터** 다시 보여야 했다.
기본 목록(엔티티 리스트)의 페이징은 현재 페이지 행만 거르므로, 필터를 켜면 페이지 수가 맞지 않았다.

| 단계 | 내용 |
|---|---|
| 상태 값 | Web API로 주문현황 열(선택 항목: 진행 중 / 완료)을 읽어 행의 `data-id`와 짝지음 |
| 건수 | `$count=true`가 동작하지 않아 `$top=5000`으로 받아 셈 |
| 페이징 | 필터 상태일 때는 직접 만든 페이징을 그리고, 기본 페이징은 CSS로 숨김(`전체`일 때만 표시) |
| 중복 렌더링 | 목록이 다시 그려질 때 감시(MutationObserver)가 페이징을 또 그려 버튼이 두 벌 생김 → 그리는 동안 감시를 끊었다 다시 연결 |

기존 화면 배치(토글이 열 머리 안에 있는 것)는 그대로 두는 것이 조건이었다.
이 목록 필터는 8월 운영 환경의 403 이후 Liquid 서버 렌더링으로 옮겼다(위 절).

## 주문확인 화면 — 확대·축소에 깨지는 표 (2026-05-20)

브라우저를 확대·축소하면 표가 넘쳤다. 스크립트(`forceTableWidth()`)가 표 폭을 픽셀로 고정하고 있었다.
고정 폭 스크립트를 걷어 내고 CSS로 바꿨다 — `width: 100%`, `table-layout: auto`, 가로 넘침은 `overflow-x: auto`.
그 뒤 상태 토글 버튼이 세로로 쌓여서 토글 묶음만 `flex-wrap: nowrap` + 열 머리 최소 폭(160px)으로 고정했다.

## 거래처 등록 검증

거래처를 만드는 경로가 두 곳(주문서 입력의 등록 창, 프로필의 등록 창)이라 둘 다 넣었다.
빈 값은 원래 막혀 있었고, 아래 세 규칙을 더했다.

```js
if (/^(DI|OI)-?\d+$/i.test(name)) { /* 주문번호 형식 차단 */ }
if (/^\d+$/.test(name))           { /* 숫자만 차단 */ }
if (name.length < 2)              { /* 너무 짧음 차단 */ }
```

## 오류 표시

실패 시 요청 본문·HTTP 응답·스택을 화면 디버그 영역에 보여주던 것을
**콘솔에만 남기고, 화면에는 일반 안내 문구만** 표시하도록 바꿨다(Site Checker의 애플리케이션 오류 노출 항목).

## 흐름 식

채번 흐름은 설정 테이블을 유형명으로 조회한다.

```
entityName: {PREFIX}_ordernumberconfigs
$filter:    {PREFIX2}_typename eq '@{outputs('OrderType')}'
```

설정 테이블의 열 접두어가 주문 테이블과 다르다(`{PREFIX2}_` vs `{PREFIX}_`).
솔루션 작업 중 게시자가 둘 섞였기 때문이라, 조회식을 쓸 때 접두어를 꼭 확인한다.

통보 메일 본문은 **인라인 스타일 HTML 표**로 썼다.
메일 클라이언트가 `<style>` 블록을 자주 지우므로 스타일을 태그마다 직접 붙였다.
