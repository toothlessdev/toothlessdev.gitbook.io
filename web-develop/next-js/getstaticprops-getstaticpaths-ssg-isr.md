# getStaticProps, getStaticPaths 로 SSG ISR 적용하기

## 📖 일반적인 React 의 렌더링

일반적인 리액트 앱의 경우 빈 HTML 에 JavaScript 를 이용해 클라이언트측에서 렌더링을 진행합니다.\
데이터를 서버로부터 로딩할때는 추가적인 JavaScript 가 실행됩니다.

이는

> 1. 자바스크립트 번들 크기가 커질 경우, Hydration, Data Fetching 에 시간이 오래 소요되어 LCP (Largest Contentful Paint) 에 좋지 못합니다.
> 2. 검색엔진이 크롤링 하는 웹 페이지는 Javascript 실행 전에는 텅 비어있어, 검색엔진최적화 (SEO, Search Engine Optimization) 에 좋지 못합니다.



## 📖 Next JS 의 Pre-Rendering

NextJS 는 위의 문제를 Pre-Rendering, 사전 렌더링을 통해 해결합니다.\
User 로 부터 요청이 오면, Pre-Rendered 사전 렌더링된 페이지를 응답으로 보내줍니다.

NextJS 는 페이지와, 페이지에 들어있는 데이터를 포함한 페이지 전체를 사전 렌더링하고,\
Hydrate, Fetch 등이 포함된 JavaScript 번들 또한 재전송합니다.

Pre-Rendering 은 User 로 부터 최초의 요청이 들어올때만 동작하고,\
이후에는 React 가 DOM 을 조작하며 일반적인 Single Page Application 처럼 동작합니다.



## 📖 getStaticProps 를 통한 SSG



## 📖 동적 라우트에서 getStaticPaths 를 통한 SSG



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

[https://github.com/leerob/react2025/blob/8192be271bf3a52c1207b519ec33a38bcc7c99a5/docs/pages/fundamentals/next.md](https://github.com/leerob/react2025/blob/8192be271bf3a52c1207b519ec33a38bcc7c99a5/docs/pages/fundamentals/next.md)
