# 브라우저 렌더링 과정 How to render in Webkit

WebKit 이 무엇인가

오픈소스 웹 엔진 시플플로 개발됌

사파리 크롬 안드로이드 ... 가 웹킷을 사용한 브라우저이다



Webkit components

webkit : thin, os interaction layer&#x20;

webcore : rendering, layout, painting, dom, bindings, ...

javascript core / v8 : js engine, js execution

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

platform = all the hooks for talking to the operating system

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>



브라우저는 뭘하냐



loading ? get the data into our engine

loading is very complicated.

split between platform layer + webcore itself&#x20;

most of the code is inside webcore loader



<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>





frame loader client : webcore 가 실제 네트워크 요청 수행하기 위해 webkit 계층과 다시 통신하는 방법ㅂ

두가지 로드

loading frame vs resources

1. loadingframe

세 단계

* policy stage (block pop ups, cross process navigation)

webkiit 에게 이창을 열도록 허용하시겠습니까 ?

더이상ㄷ 동일 도메인 네임이 없는 탭으로 이동합니다 ?

* provisional phase

다운로드를 처리할것인지, 해당 로드를 커밋해서 현재 콘텐츠를 대체할것인지

링크를 클릭하면 해당 링크가 커밋된 로드인 경우 현재 페이지 콘텐츠가 대체됌

이후 parse ...&#x20;







## 🔗 참고자료

브라우저 작동방식\
[https://web.dev/articles/howbrowserswork?hl=ko](https://web.dev/articles/howbrowserswork?hl=ko)

How to Render in WebKit - Google For Developers\
[https://www.youtube.com/watch?v=RVnARGhhs9w\&ab\_channel=GoogleforDevelopers](https://www.youtube.com/watch?v=RVnARGhhs9w\&ab\_channel=GoogleforDevelopers)



