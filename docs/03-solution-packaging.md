# 3. 솔루션 배포

## managed 로 내보냈다

`solution.xml`의 `<Managed>1</Managed>` — **managed 솔루션**이다.

unmanaged 와 달리 구성요소가 잠기므로 대상 환경에서 임의 수정이 되지 않고,
솔루션을 제거하면 그 안의 구성요소가 함께 정리된다.
납품 환경에는 managed로 넣는 것이 기본이다.

## 버전

내보낸 파일이 셋 남아 있다.

```
1.0.0.1   →   1.0.0.3   →   1.0.0.4
```

managed 솔루션은 **같은 버전으로 덮어쓸 수 없다.** 수정할 때마다 버전을 올려 다시 내보낸다.

## 솔루션 구성

`1.0.0.4` 기준 313개 파일이다.

```
solution.xml               솔루션 정의·구성요소 목록        (134KB)
customizations.xml         테이블·폼·뷰 등 스키마           (1.8MB)
Formulas/                  수식 열 정의
Workflows/                 클라우드 흐름 5개 (JSON)
powerpagecomponents/       포털 구성요소 267개 + 웹파일
maskingrules/              Power Pages 기본 마스킹 규칙 6종
Assets/ · metadataforarchivals/ · [Content_Types].xml
```

압축을 풀면 22.8MB, zip 상태로 8.7MB다.
`customizations.xml` 한 파일이 1.8MB를 차지하는데,
테이블 스키마와 폼·뷰 정의가 전부 여기 한 덩어리로 들어가기 때문이다.

## 솔루션 밖에서 관리한 흐름

주문번호 채번과 담당자 통보 흐름은 솔루션과 **별도 zip으로도 내보내져 있다.**
각각 `Microsoft.Flow/flows/<GUID>/definition.json` 구조를 가진 흐름 패키지다.

내보내기 매니페스트를 보면 흐름 자체는 `Update`,
API와 연결(connection)은 `Existing`으로 잡혀 있다.
**연결은 패키지에 담기지 않고 가져오는 환경의 기존 연결에 붙인다**는 뜻이다.
그래서 흐름만 따로 옮길 때는 대상 환경에 Dataverse·Office 365 연결이
먼저 있어야 한다.
