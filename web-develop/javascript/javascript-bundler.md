# JavaScript Bundler 란?

## 📖 Javascript Bundler?

### ✏️ 번들러란?

> **Bundle? 의 사전적 의미 :** 꾸러미, 묶음, 보따리, 다발 ...

웹 애플리케이션을 만들고자 할때, 복잡하고 다양한 모듈과 라이브러리를 사용하여 개발을 진행합니다.

예를들어, 프론트엔드에서는 `ReactJS` 를 사용하여 Single Page Application 을 만들고, Client Side Routing 을 위해 `ReactRouterDOM` 을, 전역 상태 관리를 위해 `Redux` 등을 사용합니다.

개발시에는 컴포넌트, 훅 등의 구성요소를 각각의 파일로 나누어 개발하면 관리가 쉽지만, 브라우저에서 많은 파일을 개별적으로 로드하게 되면 **성능 문제**가 발생 할 수 있습니다.



### ✏️ Bundler 를 사용하지 않는다면 어떤 문제가 발생할까?

1. <mark style="background-color:orange;">**HTTP 오버헤드와 서버 부하 증가**</mark>

각각의 JS 파일을 브라우저에서 로드할때 HTTP 요청이 필요하고, 파일이 많아질수록 네트워크 요청 수가 증가합니다.\
많은 네트워크 요청은 HTTP Header 등으로 인해 전체적인 네트워크 사용량이 증가하고, 서버에 부하가 걸려 성능이 저하 될 수 있습니다

2. <mark style="background-color:orange;">**의존성 모듈의 로드 순서 및 중복**</mark>

모듈간 의존성이 있는 경우 올바른 순서대로 로드되지 않으면 오류가 발생할 수 있습니다.

<figure><img src="../../.gitbook/assets/image.png" alt="" width="563"><figcaption></figcaption></figure>

다음과 같은 의존성 트리가 있을때, `Module C` 를 불러오기 전, `Module D` 가 로드되지 않는다면, undefined 오류가 발생할 수 있습니다.

또, `Module D` 가 중복적으로 로드 되어, 불필요한 네트워크 요청이 발생하거나, 서로 다른 인스턴스가 생성되어 각 인스턴스가 서로 다른 상태를 유지할 수 있습니다.





## 📖 Bundler 의 기능

Bundler 는 여러 JS 파일을 하나로 묶어주는 것 외에도, 다향한 기능을 사용해 문제를 해결합니다.

### ✏️ 중복 모듈 문제 (Tree Shaking, Deduplication)

1. <mark style="background-color:orange;">**Tree Shaking 트리 쉐이킹**</mark>

나무를 흔들면 썩은 나뭇잎이 떨어지는 것 처럼,\
불필요한 코드나 사용하지 않는 모듈을 제거하여, 번들 파일에서 중복 모듈이 포함되지 않도록 합니다.

<figure><img src="../../.gitbook/assets/image (2).png" alt="" width="563"><figcaption></figcaption></figure>

다음과 같은 의존성 트리와 패키지 내부에 모듈이 있을때, 사용하지 않는 `Foo()` `Baz()` 는 트리쉐이킹 되어 번들에 포함되지 않습니다.



2. <mark style="background-color:orange;">**DeDuplication 모듈 병합**</mark>

번들러는 동일 모듈이 여러 곳에서 참조 되더라도, 하나의 모듈로 통합하여 중복을 제거합니다.



### ✏️ 의존성 관리 (Dependency Graph, Version Resolution)

1. <mark style="background-color:orange;">**Dependency Graph 의존성 그래프**</mark>

번들러는 모듈/패키지 간의 관계를 의존성 그래프로 분석합니다.\
중복된 모듈은 제거되고, 올바른 순서로 로드되어, 런타임 오류를 방지합니다

<figure><img src="../../.gitbook/assets/image (4).png" alt="" width="563"><figcaption></figcaption></figure>

가장 의존도가 낮은 `Module D` -> `Module E` -> `Module A` -> `Module B` -> `Module C` 순서로 로드됩니다.

2. <mark style="background-color:orange;">**Version Resolution 버전 관리**</mark>

번들러는 동일한 모듈의 여러 버전이 존재할 때, 최적의 버전을 선택하고 중복되지 않도록 관리합니다\
이를 통해 버전 충돌문제를 해결할 수 있습니다.



### ✏️ HTTP 오버헤드 문제

1. <mark style="background-color:orange;">**Code Splitting 코드 분할, Lazy Loading 지연 로딩**</mark>

Bundler 를 사용하여 여러 패키지와 파일들을 하나로 합치면, 모든 모듈, 패키지 들이 하나의 큰 번들파일에 포함됩니다.

이는, 번들 파일을 다운로드, 파싱, 실행하는데 많은 시간을 필요로 하게 되고, 브라우저의 성능이 저하될 수 있으며 초기페이지 속도 로드 저하 등 사용자 경험에 악영향을 끼치게 됩니다.



<figure><img src="../../.gitbook/assets/image (5).png" alt="" width="563"><figcaption><p>get-p-frontend (Rollup Bundle Analyzer)</p></figcaption></figure>

번들러는 코드분할과 지연로딩을 통해 하나의 번들 파일을 여러개의 파일로 분리하고, <mark style="background-color:orange;">**(Code Splitting)**</mark>\
분할된 코드는 사용자가 서비스를 이용하는 중 해당 코드가 필요해지는 시점에 로드 <mark style="background-color:orange;">**(Lazy Load)**</mark> 되어 실행됩니다.





2. <mark style="background-color:orange;">**Minification 코드 압축**</mark>

Minification 은 JS 코드를 압축하여 파일 크기를 최소화 하는 기법입니다. 코드에서 필요한 공백, 주석, 줄바꿈, 긴 변수명 등을 제거하고 압축하여 파일 크기를 줄입니다.





## 🔗 참고자료

[https://dev.to/sayanide/the-what-why-and-how-of-javascript-bundlers-4po9](https://dev.to/sayanide/the-what-why-and-how-of-javascript-bundlers-4po9)\
[https://snipcart.com/blog/javascript-module-bundler](https://snipcart.com/blog/javascript-module-bundler)\
[https://medium.com/webpack/webpack-and-rollup-the-same-but-different-a41ad427058c](https://medium.com/webpack/webpack-and-rollup-the-same-but-different-a41ad427058c)\
[https://dev.to/mustafamilyas/brief-explanation-of-javascript-module-bundler-b1k](https://dev.to/mustafamilyas/brief-explanation-of-javascript-module-bundler-b1k)

