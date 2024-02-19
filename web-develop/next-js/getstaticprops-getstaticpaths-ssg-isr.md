# getStaticProps, getStaticPaths 로 SSG ISR 적용하기

## 📖 일반적인 React 의 렌더링

일반적인 리액트 앱의 경우 빈 HTML 에 JavaScript 를 이용해 클라이언트측에서 렌더링을 진행합니다.\
데이터를 서버로부터 로딩할때는 추가적인 JavaScript 가 실행됩니다.

이는

> 1. 자바스크립트 번들 크기가 커질 경우, Hydration, Data Fetching 에 시간이 오래 소요되어 LCP (Largest Contentful Paint) 에 좋지 못합니다.
> 2.



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

