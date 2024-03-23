# getStaticProps, getStaticPaths 로 SSG ISR 적용하기

## 📖 일반적인 React 의 렌더링

일반적인 리액트 앱의 경우 빈 HTML 에 JavaScript 를 이용해 클라이언트측에서 렌더링을 진행합니다.\
데이터를 서버로부터 로딩할때는 추가적인 JavaScript 가 실행됩니다.

이는

> 1. 자바스크립트 번들 크기가 커질 경우, Hydration, Data Fetching 에 시간이 오래 소요되어 LCP (Largest Contentful Paint) 에 좋지 못합니다.
> 2. 검색엔진이 크롤링 하는 웹 페이지는 Javascript 실행 전에는 텅 비어있어, 검색엔진최적화 (SEO, Search Engine Optimization) 에 좋지 못합니다.



## 📖 Next JS 의 Pre-Rendering

NextJS 는 위의 문제를 <mark style="background-color:orange;">Pre-Rendering, 사전 렌더링</mark>을 통해 해결합니다.\
User 로 부터 요청이 오면, Pre-Rendered 사전 렌더링된 페이지를 응답으로 보내줍니다.

NextJS 는 페이지와, 페이지에 들어있는 데이터를 포함한 페이지 전체를 사전 렌더링하고,\
<mark style="background-color:orange;">Hydrate, Fetch 등이 포함된 JavaScript 번들 또한 재전송</mark>합니다.

> Pre-Rendering 된 페이지의 응답은 User 로 부터 최초의 요청이 들어올때만 동작하고,\
> 이후에는 React 가 DOM 을 조작하며 일반적인 Single Page Application 처럼 동작합니다.



## 📖 getStaticProps 를 통한 Static Site Generation

Static Site Generation (SSG) 정적 사이트 생성은, 빌드타임에 Pre-Rendered 사전 렌더링된 페이지를 생성합니다.\
NextJS 는 페이지를 구성하는 데이터를 포함한 HTML 코드를 사전 렌더링합니다.

빌드 타임에 페이지가 사전 렌더링 되기 때문에, 배포 이후 구축된 페이지는 서버나 앱을 실행시키는 CDN 을 통해 캐시 될 수 있습니다.

Page 컴포넌트 파일에 `getStaticProps()` 를 export 하는 경우, NextJS 는 정적으로 렌더링되어야 하는 페이지로 인식합니다.

```tsx
export default function Page(props : InferGetStaticPropsType<typeof getStaticProps>) {

}

export async function getStaticProps(context : GetStaticPropsContext) {
    return { props: {} };
}
```

`getStaticProps` 내부에 작성된 코드는 Client 로 전송되는 JavaScript 번들에 포함되지 않기 때문에, Server Side 에서 실행되는 코드를 작성할 수 있습니다.



## 📖 동적 라우트에서 getStaticPaths 를 통한 SSG

NextJS 에서 모든 페이지는 Pre-Render 되지만, `[id].tsx [...slug].tsx` 와 같은 동적 라우트에 대한 페이지들은 Pre-Render 되지 않습니다.

동적 라우트에 대한 페이지는 여러 페이지로 이루어지기 때문에,\
NextJS 는 얼마나 많은 페이지들을 사전 렌더링해야할지 모릅니다.

동적인 라우트에 대해서 NextJS 가 페이지들을 사전렌더링 하도록 `getStaticPaths` 를 사용해 인식해주어야 합니다.

getStaticPaths 는 paths, fallback 프로퍼티를 포함한 객체를 리턴해야합니다.

```tsx
// pages/[id].tsx
export async function getStaticPaths() {
    return {
        paths : [
            { params : { id : 1 } },
            { params : { id : 2 } },
            { params : { id : 3 } }
        ],
        fallback : // true, false, "blocking"
    }
}
```

### ✏️ fallback

> fallback : false

getStaticPaths 가 리턴하지 않는, <mark style="background-color:orange;">사전렌더링되지 않은 페이지는 404.tsx 로 리다이렉트</mark> 됍니다.\
`next build` 가 실행될때, `fallback : false` 인 경우, `getStaticPaths` 가 리턴하는 동적 페이지에 한해서만 Pre-Render 됍니다.

사전 렌더링 해야할 동적 라우트의 페이지의 수가 적거나, 새로운 동적 라우트 페이지의 추가가 적은경우에 좋습니다.



> fallback : true

getStaticPaths 가 리턴하지 않는, <mark style="background-color:orange;">사전 렌더링되지 않은 페이지는 fallback 페이지를 리턴</mark>합니다.\
해당 동적 페이지에서는 `router.isFallback` 을 이용해 해당 페이지가 fallback 상태인지 확인할 수 있습니다.

```tsx
import { useRouter } from "next/router";
export default function Page() {
    const router : NextRouter = useRouter();
    
    if(router.isFallback) {
        return <Loading/>
    }
    return <></>
}
```



> fallback : "blocking"

dd



## 📖 getStaticProps 에 revalidate 를 통한 ISR



```javascript
export async function getStaticProps(context) {
    // ...
    return {
        props : {
            
        },
        revalidate : ,// seconds;
        notFound : true / false, // getStaticProps 의 fetch / db 중 오류발생시 notFound:true 리턴
        redirect : {
            destination : "/route"
        }
    }
}
```

context 를 이용해 구체적인 매개변수 값을 구할 수 있음

```javascript
const { params } = context;
```

동적 페이지를 위한 getStaticPath

NextJS 는 페이지를 사전 렌더링함

동적 페이지에서 어떤 페이지가 사전 렌더링 되어야 하는지,

어떤 값에 대한 페이지가 사전 렌더링 되어야 하는지 알아야함

```tsx
export async function getStaticPaths() {
    return {
        paths: [
            { params: {  } }
        ],
        fallback: true / false / "blocking"
    }
}
```

동적 페이지의 어떤 구ㅈ체적인 인스턴스를 사전 생성할지 알려주는 함수





## 🔗 참고자료

[https://nextjs.org/docs/pages/building-your-application/data-fetching/get-static-props](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-static-props)[https://github.com/leerob/react2025/blob/8192be271bf3a52c1207b519ec33a38bcc7c99a5/docs/pages/fundamentals/next.md](https://github.com/leerob/react2025/blob/8192be271bf3a52c1207b519ec33a38bcc7c99a5/docs/pages/fundamentals/next.md)
