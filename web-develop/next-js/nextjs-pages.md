# NextJS Pages 라우터



NextJS 에서는 프로젝트 생성시 Page 라우터 또는 App 라우터를 사용할 수 있습니다.\
이번 포스트에서는 NextJS 의 Page 라우터에 대해 소개하고자 합니다.



## 📖 React Router Dom

리액트는 라이브러리이기 때문에 Client Side Routing 을 위해서는 React Router Dom 이라는 패키지를 별도로 설치해야합니다.

```tsx
<BrowserRouter>
    <Routes>
        <Route path="page" element={<Page/>}>
        // ...
    </Routes>
</BrowserRouter>
```

React Router Dom 에서는 `createBrowserRouter` 또는 `<BrowserRouter>` 등과 같은 컴포넌트 / 함수를 이용해서 라우터를 만든후, 경로에 해당하는 페이지 컴포넌트를 선언해야합니다.



## 📖 NextJS Page Router 의 라우팅

### ✏️ 파일시스템을 이용한 라우팅

React Router Dom 과 다르게 NextJS 는 `pages` 라는 특별한 디렉토리를 기준으로,\
<mark style="background-color:orange;">**파일시스템에서 라우팅**</mark> 구조를 만들어냅니다.

다음 예시와 같이 NextJS 에서는 Pages 디렉토리에서부터 해당 파일 까지의 경로를 이용해 라우팅 Path 를 결정하고,\
해당 파일에서 <mark style="background-color:orange;">**export default**</mark> 하는 컴포넌트를 페이지 컴포넌트로 인식합니다.

> src/pages/index.jsx -> **domain.com/**
>
> src/pages/about.jsx -> **domain.com/about**
>
> src/pages/posts/index.jsx -> **domain.com/posts**
>
> src/pages/posts/\[postId].jsx -> **domain.com/posts/:postId** (동적 Path 에 대한 라우트)

NextJS 에서 하위 디렉토리를 만들어 _**index.jsx**_ 를 만드는것과 상위폴더에서 _**페이지이름.jsx**_ 로 만드는 것 둘다 동일한 라우트를 생성해줍니다.



### ✏️ 동적 라우팅 경로 추가하기

동적 라우팅 경로에 대한 페이지 컴포넌트를 추가하기 위해서는\
해당 컴포넌트가 구현된 jsx / tsx 파일을 `[id].jsx` 와 같이 저장해줍니다.

<mark style="background-color:orange;">**useRouter**</mark> 훅을 사용하면 동적 라우트에서 실제로 URL 에 입력한 값을 가져올 수 있습니다.



```tsx
export default function Page() {
    const router = useRouter();

    console.log(router.pathname);
    console.log(router.query);
    
    return <></>
}
```

<mark style="background-color:orange;">**pathname**</mark> 은 해당 페이지 컴포넌트가 위치한 pages 디렉토리에서부터의 파일경로를 리턴합니다

<mark style="background-color:orange;">**query**</mark> 에는 동적인 경로에 대한 구체적인 값이 객체 형태로 들어있습니다.\
query 객체의 키값은 컴포넌트가 구현된 파일을 저장할때 \[ ] 안에 들어있는 값이 됍니다.

`[postId].jsx` 인 경우 query 객체는 `{ postId : "" }` 가 됍니다.



`[...params].jsx` 와 같이 Spread 문법을 사용해 동적 라우트의 쿼리값을 가져오는 경우,\
query 객체는 각 동적경로를 배열로 받아옵니다



### ✏️ 404 Not Found Page 추가하기

`pages/404.jsx` 에 라우팅시 존재하지 않는 페이지 컴포넌트를 표시할 수 있습니다.





## 📖 NextJS 의 Client Side Routing

`<a href=""></a>` 를 이용하여 페이지를 이동하는경우, 새로운 HTTP 요청을 보내게 되고\
State, Context, 전역 상태가 초기화됍니다.



### ✏️ Link 컴포넌트

NextJS 에서 Client Side Routing 을 위해서는`<a>` 대신 `<Link>` 컴포넌트를 사용해야 합니다.

```tsx
import Link from "next/link";

export function NavBar () {
    return (
        <nav>
            <ul>
                <li>
                    <Link href="/">Home</Link>
                </li>
                <li>
                    <Link href="/about">About</Link>
                </li>
                <li>
                    <Link href="/posts">Posts</Link>
                </li>
            </ul>
        </nav>
    );
}
```

Link 의 props

href : 이동할 경로입니다. 문자열 또는 경로 객체가 들어갈 수 있습니다.

```tsx
<Link href={{
    pathname : "/path/[id]",
    query: {id : path.id}
}}/>
```

**replace** : 기본값은 false 이고, true 일 경우 Link 로 이동시 브라우저 히스토리 스택이 교체되고,\
이전 방문한 페이지 라우트가 교체되어 뒤로가기가 불가능해집니다.

**scroll** : 기본값은 true 이고 Link 이동시 스크롤이 최상단으로 설정됍니다. false 일 경우 이전 페이지 스크롤 위치가 유지됍니다.



### ✏️ useRouter

컴포넌트가 아닌, 이벤트 리스너와 같은 곳에서 네비게이팅이 필요할 경우 `useRouter` 훅을 사용하면 됍니다.\
훅이기 때문에 함수형 컴포넌트에서 선언하여 사용가능합니다.

```tsx
import { useRouter } from "next/router";

export default function MyComponent() {
    const router = useRouter();
    
    const handler = (e: Event) => {
        router.push();
        router.replace();
    }
}
```

**router.push();**\
브라우저 히스토리 스택에 다음 페이지에 대한 라우트를 푸시 합니다. 이전 페이지로 돌아갈 수 있습니다.

**router.replace();**\
브라우저 히스토리 스택을 교체합니다. 따라서 이동후 뒤돌아가기가 불가능합니다.

push 와 replace 도 동일하게 이동할 경로에 대한 문자열 또는 경로 객체가 들어갈 수 있습니다.





## 🔗 참고자료

[https://nextjs.org/docs/pages/building-your-application/routing/pages-and-layouts](https://nextjs.org/docs/pages/building-your-application/routing/pages-and-layouts)\
[https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes](https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes)
