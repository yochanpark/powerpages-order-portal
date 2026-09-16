# 1. 포털 구성

## 구성요소

Power Pages 사이트는 Dataverse 테이블에 저장되는 구성요소의 묶음이다.
이 사이트는 **267개**의 구성요소로 이루어져 있고, 타입은 **20종**에 걸쳐 있다.
가장 많은 타입이 74개, 그다음이 52개·28개 순이다.

솔루션으로 내보내면 `powerpagecomponents/<GUID>/` 아래에 구성요소 XML이,
웹파일인 경우 실제 파일이 `filecontent/`에 함께 나온다.
사이트 전체를 솔루션에 담아 환경 간에 옮길 수 있다.

## 테마

| 파일 | 역할 |
|---|---|
| `bootstrap.min.css` | 기본 그리드·컴포넌트 |
| `portalbasictheme.css` | Power Pages 기본 테마 |
| `theme.css` | 이 사이트 전용 재정의 |

로고(상단 메뉴용·64px)와 로그인/인쇄 화면용 이미지 20여 종을 웹파일로 올려 참조했다.
`PWAManifest.json`과 `robots.txt`도 웹파일로 들어가 있다.

## 로그인 화면 커스터마이징

`Account/SignIn` 화면을 CSS 재정의로 바꿨다. 실제로 하는 일은 셋이다.

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

로컬 계정 로그인만 쓰는 사이트라 Entra ID·외부 공급자 버튼을 가렸다.

> 이건 **화면에서 감춘 것이지 비활성화한 것이 아니다.**
> 인증 수단을 실제로 막으려면 사이트 설정에서 해당 공급자를 꺼야 한다.
> CSS는 보이지 않게 할 뿐이고 접근 통제가 아니다.

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
Power Pages는 기본 인증 화면의 마크업을 바꿀 수 없어 CSS로 덮는 것 말고 선택지가 없었다.

## 솔루션에 함께 들어온 것

`maskingrules/` 아래에 마스킹 규칙 6종이 들어 있다.

```
Email · Email_HideName · SocialSecurityNumber
SocialSecurityNumber_ShowLastFourDigits · Date_Hyphen · Date_Slash
```

**이건 직접 만든 것이 아니라 Power Pages가 기본 제공하는 표준 규칙**이고,
솔루션을 내보낼 때 함께 따라온 것이다. 기록해 두는 이유는
솔루션 내용물을 볼 때 어디까지가 플랫폼 기본값인지 구분해야 하기 때문이다.
