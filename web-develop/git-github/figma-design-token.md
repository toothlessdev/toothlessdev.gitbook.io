# Figma Design Token 으로 디자인 변경사항 코드에 반영하기

> 이 포스트는 [https://docs.tokens.studio/](https://docs.tokens.studio/) 를 참고하여 작성하였습니다



## 📖 컴포넌트와 디자인시스템

웹 애플리케이션을 개발할때, 프론트엔드 개발자와 디자이너 간의 협업은 중요합니다.\
인터페이스 (UI) 의 일관된 디자인은 사용자 경험 (UX) 뿐만 아니라, 개발자 경험 (DX) 에도 영향을 미칩니다.

이런 측면에서 디자인 시스템과 컴포넌트 기반 개발은 개발과 디자인 간의 간극을 줄이고, 재사용성과 일관성을 유지하는데 중요한 역할을 합니다.

> ❓ 컴포넌트란 : UI의 재사용 가능한 독립적인 부분
>
> ❓ 디자인 시스템 : 일관된 사용자 경험을 위해 사용되는 **전체적인 규칙과 원칙의 집합으로,** 타이포그래피, 색상 팔레트, UI 패턴 (버튼, 카드, 입력 필드 ...), 아이콘, 일러스트레이션, 상호작용 규칙 등이 포함됩니다



### ✏️ 디자인 시스템과 디자인 토큰

디자인 시스템은 단순한 스타일 가이드가 아닌, <mark style="background-color:orange;">**어떻게 생기고 작동해야 하는지**</mark>를 정의하는 규칙 집합입니다.

<mark style="background-color:orange;">**피그마 디자인 토큰**</mark>을 활용하면 디자인 시스템의 모든 요소들을 코드와 바로 연결할 수 있어, 디자이너와 개발자 간의 원활한 협업을 가능하게 합니다.

디자인 토큰은 색상, 폰트, 간격, 그림자 등의 속성을 변수로 관리할 수 있게 도와줍니다.\
이를 프론트엔드 코드에 바로 반영함으로써, **디자인 변경사항이 즉시 코드에 적용되고, 디자인의 일관성을 유지하면서도 변경 사항을 빠르게 반영**할 수 있습니다.

이를 통해 수동으로 스타일을 반영하는 번거로움을 없애고, 모든 팀원이 동일한 디자인 기준을 따라갈 수 있도록 보장합니다.

> **🎨 디자인 토큰 Design Token**
>
> Design tokens are the visual design atoms of the design system — specifically, they are named entities that store visual design attributes. We use them in place of hard-coded values (such as hex values for color or pixel values for spacing) in order to maintain a scalable and consistent visual system for UI development. [Salesforce, Lightning Design System](https://www.lightningdesignsystem.com/design-tokens/)
>
> 디자인 토큰은 디자인 시스템의 시각적 디자인 원자, 특히 시각적 디자인 속성을 저장하는 개체로 명명됩니다. UI 개발을 위해 확장 가능하고 일관된 시각적 시스템을 유지하기 위해 하드 코딩된 값(예: 색상의 경우 헥스 값 또는 간격의 경우 픽셀 값) 대신 사용합니다.





## 📖 Figma Token Studio 를 사용해 디자인 토큰 관리하기

### ✏️ Figma Token Studio

{% embed url="https://docs.tokens.studio/" %}

Figma Token Studio 플러그인을 사용하면, 디자인 토큰을 손쉽게 관리하고 코드에 반영할 수 있습니다.

**Figma 에서 Plugin > 'Tokens Studio for Figma' 를 실행합니다.**

<figure><img src="../../.gitbook/assets/image (37).png" alt="" width="375"><figcaption></figcaption></figure>

Color, Font Size, Sizing, Spacing, Border Radius 등에 대한 디자인 토큰을 등록하고,\
피그마에서 선택된 객체에 적용하거나, 페이지에서 해당 토큰을 사용하는 모든 객체에 변경사항을 반영하는 등의 작업을 할 수 있습니다.

이를 통해 일관된 색상, 사이즈 등을 유지하고 반영할 수 있습니다.

> **🔗 참고**
>
> 토큰 생성 : [https://docs.tokens.studio/tokens/creating-tokens](https://docs.tokens.studio/tokens/creating-tokens)\
> 토큰 적용 : [https://docs.tokens.studio/tokens/applying-tokens](https://docs.tokens.studio/tokens/applying-tokens)



### ✏️ Github 를 사용해 Design Token 자동화하기

Figma Token Studio 를 사용하면 간편하게 클릭 몇번으로 디자인 토큰 수정사항을 코드에 반영할 수 있습니다.

1. **Settings > Developer Settings 에 들어가 Fine-Grained Token 을 발행합니다.**

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

2. **토큰이름과 만료날짜를 선택하고, 특정 리포지토리에 대한 접근을 허용하기위해 Repository Access 접속 후 Only select repositories 에서 프론트엔드 리포지토리를 선택합니다**

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

3. **권한 설정에서 Contents 를 'Read and Write' 로 설정합니다. 해당 토큰이 리포지토리에 접근해 디자인 토큰을 읽고 쓰기 위함입니다.**

<figure><img src="../../.gitbook/assets/스크린샷 2024-09-19 오후 3.40.44.png" alt=""><figcaption></figcaption></figure>

4. **Figma 의 Tokens Studio for Figma 로 돌아와 Settings 탭에 접속, Sync Providers 에서 새로운 Sync Provider 를 추가합니다**

<figure><img src="../../.gitbook/assets/image (34).png" alt="" width="375"><figcaption></figcaption></figure>

5. **Github 를 선택하고, 이름과 이전에 발급했던 Access Token 을 붙여넣고, repository 에는 owner/repo 에 맞게 작성해 줍니다. Branch 에는 토큰을 업데이트할 브랜치명을 설정합니다.**

<figure><img src="../../.gitbook/assets/image (35).png" alt="" width="375"><figcaption></figcaption></figure>

6. **새로운 디자인 토큰을 생성하고, 푸시 버튼을 눌러 깃허브에 업데이트 합니다**

| 디자인 토큰 업데이트                                                                  | 깃허브에 푸시                                                                                              | 커밋, 브랜치 설정                                                                   |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| <img src="../../.gitbook/assets/image (41).png" alt="" data-size="original"> | <img src="../../.gitbook/assets/스크린샷 2024-09-19 오후 5.11.17.png" alt="" data-size="original"> | <img src="../../.gitbook/assets/image (42).png" alt="" data-size="original"> |

7. **디자인 토큰을 푸시하면 pull request 에서 토큰의 변경사항을 검토하고 코드에 반영할 수 있습니다.**

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

