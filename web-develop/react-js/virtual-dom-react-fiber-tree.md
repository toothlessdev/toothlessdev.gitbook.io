# Virtual DOM , React Fiber Tree와 리액트 렌더링

리액트의 렌더링 과정을 공부하면서,

Virtual DOM 이 실제 DOM 을 복사해서 변경된 내역만 바꾼다는데

> 복사하는데 많은 오버헤드가 걸리지는 않을까? 복사하고 변경된 노드를 비교하는게 더 오랜 시간이 걸리지 않을까?
>
> 그렇다면 왜 Virtual DOM 을 사용하는걸까? 왜 더 빠른걸까?

라는 의문이 들었고,\
의문을 해결하기위해 리액트의 Virtual DOM 과 관련된 내용을 찾아보면서 정리한 포스트입니다



이번 포스트에서는 리액트에서 Virtual DOM 이 왜 도입되었고, 실제 DOM 과 비교한 Virtual DOM 의 동작방식에 대해서 알아보고 리액트에서 Virtual DOM 을 개선한 React Fiber Tree 에 대해서 알아보겠습니다.



## 📖 브라우저의 렌더링 과정

Virtual DOM 은 왜 도입되었을까요 ?\
그러기 위해서는 브라우저의 렌더링 과정과, DOM API 에 대해서 알아야 합니다.

Chrome, Safari 에 들어가는 WebKit 엔진은 대략적으로 다음과 같은 과정을 거쳐 렌더링합니다.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

> 1. 먼저 HTML 을 브라우저의 렌더엔진이 파싱해서 <mark style="background-color:orange;">**DOM Tree**</mark> 를 생성합니다
> 2. CSS 파일, Inline-Style 을 파싱하고 <mark style="background-color:orange;">**CSSOM Tree**</mark> 를 생성합니다.
> 3. DOM Tree 와 CSSOM Tree 를 결합하여 <mark style="background-color:orange;">**Render Tree**</mark> 를 만듭니다.
> 4. Render Tree 의 각 노드들이 위치와 크기를 계산하는 <mark style="background-color:orange;">**Layout**</mark> 과정을 거친 뒤,
> 5. Layout 에서 계산된 스타일을 기반으로 실제로 화면에 그리는 <mark style="background-color:orange;">**Paint**</mark> 작업을 실행합니다.



### ✏️ DOM API 를 이용한 DOM 조작

`document.getElementbyId()` 또는 `document.querySelector()` 와 같은 DOM API 를 이용해 DOM 을 직접 조작하여 화면을 업데이트 하면, 위의 과정이 반복됍니다.

이 과정에서 많은 연산이 수행되고, 비용이 크기 때문에 브라우저 성능을 저하시킬 수 있습니다.



### ✏️ CSR 과 SPA

예전에는 서버에서 데이터와 함께 완전한 페이지를 HTML 로 만들어 주는 SSR(Server Side Rendering) 방식을 사용했습니다.

이때는 DOM 에 동적인 변화가 적었고, DOM API 를 사용하여 조작하는것이 크게 문제가 되지 않았습니다.

하지만, 새로운 페이지를 요청해서 UX 가 떨어지는 SSR 을 개선하고자 SPA(Single Page Application) 과 함께 CSR(Client Side Rendering) 방식이 사용되면서 DOM 의 업데이트가 많이 발생하는 웹 애플리케이션이 점점 더 많아지게 되었습니다.

따라서 DOM API 로 직접 DOM 을 조작하는것, 렌더링 과정에 최적화의 필요성이 대두되었습니다.





## 📖 리액트의 초기 렌더링



### ✏️ Render - 1 : 컴포넌트 트리의 순회

리액트에서 렌더링은 컴포넌트 트리의 루트에서 시작해 업데이트가 필요한 노드를 찾기 위해\
<mark style="background-color:orange;">**컴포넌트 트리를 순회**</mark>하는 작업에서 시작됍니다.

함수형 컴포넌트 경우 함수형 컴포넌트를 실행 `Component(props)` 하고,\
클래스형 컴포넌트의 경우 컴포넌트 인스턴스의 render 함수를 호출합니다`component.render()`

컴포넌트가 반환하는 값은 일반적으로 JSX 를 반환하게 되는데

```jsx
return <MyComponent prop1={10} prop2={"value"}>자식 컴포넌트</MyComponent>
```

빌드시에 [Babel](https://babeljs.io/docs/babel-plugin-transform-react-jsx) 과 같은 트랜스파일러에 의해 `React.createElement()` 로 트랜스파일링됍니다.

```jsx
return React.createElement(MyComponent, {prop1: 10, prop2: "value"}, "자식 컴포넌트");
```

`React.createElement()` 는 다시 React Element 에 해당하는 자바스크립트 객체를 반환합니다

```jsx
{ type : MyComponent, props : { prop1: 10, prop2: "value"}, children: ["자식 컴포넌트"] }
```



### ✏️ Render - 2. 가상 DOM 의 생성과 Reconciliation

리액트는 컴포넌트 트리 전체를 순회하며 렌더되는 자바스크립트 객체를 수집해 <mark style="background-color:orange;">**가상 DOM**</mark> 을 만듭니다.







초기렌더링에서 가상 DOM 을 기반으로 실제 DOM 에 반영합니다

위의 예시에서 봤던 것 처럼 가상 DOM (Virtual DOM) 은 \
실제 DOM 속성은 가지고 있지만 DOM API 는 가지고 있지 않기 때문에 가볍습니다.



## 📖 리액트의 리렌더링

### ✏️ 렌더링 큐에 렌더링 등록하기

첫 렌더링이 완료 된 후, 리액트에게 리렌더링을 큐에 등록하도록 지시하는 방법에는

> 1. 함수형 컴포넌트에서 useState 의 setState 호출
> 2. 함수형 컴포넌트에서 useReducer 의 dispatch 호출
> 3. 클래스형 컴포넌트에서 this.setState() 호출
> 4. 클래스형 컴포넌트에서 this.forceUpdate() 호출

이 있습니다.



### ✏️ 리렌더링

리액트에서 일반적으로 <mark style="background-color:orange;">**상위 컴포넌트가 리렌더링되면, 모든 하위컴포넌트를 순회하며 리렌더링**</mark>됍니다.\
이때 렌더링은 실제 DOM 을 변경해아하는지 여부를 확인하는 절차입니다.

```jsx
<ComponentA>
    <ComponentB>
        <ComponentC>
        </ComponentC>
    </ComponentB>    
</ComponentA>
```

컴포넌트가 다음과 같이 구성되어있고, `ComponentB` 에서 useState 의 setState 를 호출하여

> 1. 리렌더링을 렌더링 큐에 삽입하면,
> 2. 컴포넌트 트리 최상단에서 렌더링을 시작합니다
> 3. ComponentA 는 업데이트가 불필요함을 확인합니다
> 4. ComponentB 는 업데이트가 필요함을 확인하고 렌더링합니다.
> 5. ComponentC 는 업데이트가 불필요하지만 부모가 렌더링되었기 때문에, 이또한 재렌더링 됍니다.







## 📖 Virtual DOM 은 왜 가벼울까?

1. Batch Update
2.  &#x20;dom 은 class, style ... 등의 실제 dom 속성은 갖고 있지만

    getElement by id 와 같은 dom api 는 가지고 있지 않기 때문에 가볍다



렌더링이란 props, 상태를 기반으로 리액트 컴포넌트



무거운 작업은 아니지만, 매번 돔을 저런식으로 하는 작업은 시간이 걸림



리액트는 재렌더링이 발생 상황에서

새로운 가상돔 생성.

렌더링 이전의 가상돔, 업데이트 이후 가상돔을 비교해서&#x20;

바뀐 엘리먼트를 찾음 (diffing) : 빠른 알고리즘으로 구성!



diffing 을 이용해 바뀐 부분들만 적용 reconciliation



batch update 덕분에 빠름!

변경된 모든 엘리먼트를 한번에 적용





### ⭐️ 헷갈리는것

* 컴포넌트 호출은 Render phase에서 실행되며 호출이 곧 화면 반영을 나타내는 것은 아님을 알려 드렸습니다. 이전 포스트에서 언급한 용어인 렌더링에 빗대어 보자면 컴포넌트가 리-렌더링 된다는 말은 컴포넌트가 호출되고 그 결과가 VDOM에 반영된다는 것이지 DOM에 마운트되어 페인트 된다는 뜻은 아닙니다





* v-dom 의 동작

브라우저가 실제 dom 트리 생성 > 화면에 ui. 를 렌더 (vdom 복사됌)

dom 노드 변화가 생기면 새로운 vdom 생성

변화마다 새로운 vdom 만드는건 비효율적이라고 생각할 수 있지만,

dom 노드 조작하는것의 비효율성은 돔 트리를 업데이트 하는 과정에서 발생하는 것이 아니라,

렌더링 하는 과정에서 비싼 비용이 드는 것임

vdom 은 메모리 상에서 트리를 변경하는 일이기 때문에 빠름 (렌더링 안함)



vdom 내부 구현체 보면 diff(previous: VTree, current:VTree) -> PatchObject

변경전, 변경후 변화된 부분만 확인



실제 돔에 변경된 부분을 적용

patch(rootNode : DOMNode, patches: PatchObject) => DOMNode newRootNode

루트노드와ㅓ 변경된 부분을 인자로 받아, 실제돔 노드에 적용



v dom 은 결국 버퍼링 / 캐싱의 역할을  한다고 보면됌.&#x20;

dom 조작마다 브라우저 렌더링 프로세스 진행하는 것이 아닌,&#x20;

변화된 부분을 실제돔에 적용해 한번만 렌더링을 해서 성능이 최적화됌



* react v dom

jsx 는 babel 에 의해 js 로 변환됌 > createElement > js 객체로 변환됌 (type : 돔 태그 이름, Props : jsx 포함된 속성)

js 객체로 vdom tree 생성됌

해당 객체로 render 함수를 호출하면 실제 돔이됌



* 재조정

리액트 상태변화시 화면에 vdom 을 활용해 dom 을 업데이트 하는 과정

vdom 은 ui 의 이상적인 또는 가상적인 표현을 메모리에 저장하고, react dom 과 같은 라이브러리에 의해 실제 돔과 동기화하는 프로그래밍 개념입니다.

재조정 = vdom, dom 을 비교하고 일치시키는 과정이라고 생각하면됌

변경 이전, 이후 vdom 트리 스냅샷을 비교(diffing)해 변화된 부분만 감지한 후, 변경된 부분만 실제 돔에 적용



재조정이 일어나는 2가지 : 리렌더링 후 element type 이 바뀔시 / \~ 동일 엘리먼트 순서가 바뀔시



* diffing

type 이라는 key 값을 갖는다고 했음

1. type 이 이전 이후 동일한 경우 : 변경된 속성만 변경!
2. \~ 다른경우 : 이전트리 삭제하고 새로운 트리를 만듦

list 렌더시 key prop 이 있어야한다는 워닝 메시지  재조정?때문임

이전 v dom 과 새로 생성된 v dom 비교

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



마지막 하나만 새로 추가되었기때문에 추가된 노드만 새로 그림



<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

첫번째 위치에 추가됌. 모든 요소가 제자리에 위치 하지 않앗다 생각하고 자식 노드를 새로 그리게 됌

key prop 을 식별자로 서 사용

자식노드가 key prop 을 가지고 있으면 key 값으로 이전, 다음 트리를 변경.

첫번째에 들어와도 문제없이 추가된 노드만 그릴수잇음



key prop 은 변경되지 않는 유일한 값이 들어와야 함

배열 index 사용하면 , 새로운 아이템이 추가되는 경우, 인덱스가&#x20;



* 함수 호출 = render, vdom 반영 = 커밋
*

fiber node = 상태가 저장되는 공간

useState 만나면 initialState 를 fiber 노드에 저장

setState 실행시 파이버 노드의 상태를 변경 > 리렌더링

fiber node 값을 state 에 가져옴







렌더 커밋

렌더 = 컴포넌트 렌더링, 변경사항 계산

커밋 = 변경사항 돔에 적용

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



여기서 이해해야 할 핵심은 "렌더링"은 "DOM 업데이트"와 같지 않으며 결과적으로 어떠한 가시적인 변경도 일어나지 않고 컴포넌트가 렌더링될 수 있다는 것입니다.

* 컴포넌트가 지난번과 동일한 렌더 출력을 반환해 변경이 필요하지 않을 수 있습니다.
* 동시 렌더링에서 리액트는 컴포넌트를 여러 번 렌더링할 수 있지만 다른 업데이트로 인해 현재 수행 중인 작업이 무효화되는 경우 렌더 출력을 버립니다.



* 리액트에 리렌더링을 큐에 등록하도록 지시하는 방법

1. useState setState 호출
2. useReducer dispatch 호출
3. this.setState()
4. this.forceUpdate()



일반적으로 상위컴포넌트 렌더시 모든 하위컴포넌트도 순회하ㅁㅕ 렌더링됌

일반적으로 props 가 바뀌었는지 여부 신경쓰지 않음

무조건 부모 렌더링시 자식도 렌더링됌



fiber : react 16에서 도입되엇고 18버전에서 사용됌. 이전에는 stack 아키텍쳐

스택과 다르게 fiber 는 나중에 들어간걸 먼저꺼낼필요없음

왜 도입 ?

스택에서 꺼낸다 = 변화를 적용한다 = 렌더링

순서가 정해져있다면 렌더링 순서를 유연하게 가져갈 수 없음



## 🔗 참고자료

Babel Docs - Transform React JSX\
[https://babeljs.io/docs/babel-plugin-transform-react-jsx](https://babeljs.io/docs/babel-plugin-transform-react-jsx)

Virtual DOM 과 Internals - React Docs\
[https://ko.legacy.reactjs.org/docs/faq-internals.html?trk=article-ssr-frontend-pulse\_little-text-block](https://ko.legacy.reactjs.org/docs/faq-internals.html?trk=article-ssr-frontend-pulse\_little-text-block)

React Fiber Architecture\
[https://github.com/acdlite/react-fiber-architecture](https://github.com/acdlite/react-fiber-architecture)

React Fiber 아키텍쳐 분석 - Naver D2\
[https://d2.naver.com/helloworld/2690975](https://d2.naver.com/helloworld/2690975)

React 톺아보기 - Preview\
[https://goidle.github.io/react/in-depth-react-preview/](https://goidle.github.io/react/in-depth-react-preview/)

(번역) 리액트 렌더링 동작에 대한 (거의) 완벽한 가이드\
[https://velog.io/@superlipbalm/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior#%EB%A0%8C%EB%8D%94%EB%A7%81-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EA%B0%9C%EC%9A%94](https://velog.io/@superlipbalm/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior#%EB%A0%8C%EB%8D%94%EB%A7%81-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EA%B0%9C%EC%9A%94)

재조정 (Reconciliation) - React Docs\
[https://ko.legacy.reactjs.org/docs/reconciliation.html](https://ko.legacy.reactjs.org/docs/reconciliation.html)

(번역) 리액트에 대해서 그 누구도 제대로 설명하기 어려운 것 - 왜 Virtual DOM 인가\
[https://velopert.com/3236](https://velopert.com/3236)

Vanila JS 로 가상돔 (Virtual DOM) 만들기\
[https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Virtual-DOM/](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Virtual-DOM/)

React Lifecycle Method Diagram\
[https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

React Hooks Lifecycle\
[https://wavez.github.io/react-hooks-lifecycle/](https://wavez.github.io/react-hooks-lifecycle/)

How Does React Actually Work? ReactJS Deep Dive #1\
[https://www.youtube.com/watch?v=7YhdqIR2Yzo\&ab\_channel=PhilipFabianek](https://www.youtube.com/watch?v=7YhdqIR2Yzo\&ab\_channel=PhilipFabianek)



