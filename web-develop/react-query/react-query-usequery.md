# React Query 개념, useQuery 로 데이터 페칭하기

React Query 는 React 에서 HTTP 요청을 쉽게 처리할 수 있도록 도와주는 라이브러리입니다.

> Powerful asynchronous state management for
>
> TS/JS, React, Solid, Vue, Svelte and Angular



## 📖 기존 데이터페칭 (서버 상태 관리) 의 단점

### ✏️ 복잡한 코드작성

기존에 데이터페칭을 하려면, fetch 의 상태 / error 상태 / data 상태 를 저장하는 state 를 만들고,\
useEffect 내부에서 fetch 함수를 비동기 호출 후 위의 상태를 조작해야한다.

```tsx
export default function MyComponent() {
    const [isLoading, setIsLoading] = useState<boolean>(false);
    const [data, setData] = useState<ResponseBody | undefined>(undefined);
    const [error, setError] = useState<Error>();
    
    useEffect(() => {
        const request = async () => {
            setIsLoading(true);
            const response = await fetch(API_URL);
            if(!response.ok) throw new Error();
            const d = await response.json();
            setData(d);
            setIsLoading(false);
        }
        try {
            request();
        } catch(e) {
            setError(e);
        }
    }, [])
    
    return <></>
}
```



### ✏️ 복잡한 기능구현

사실 위의 데이터페칭 코드는 useFetch 와 같이 커스텀 훅을 이용하면 어느정도 반복되는 코드 작성을 줄일 수 있다.\
하지만, 다른 화면을 보다가 다시 웹페이지에 포커스된경우 refetch 를 트리거 한다거나, POST 요청 후 GET 요청을 트리거 하고싶을때... 등과 같은 상황에서 해당 기능들을 구현하기 위해서는 많은 시간이 걸린다.

React Query (Tanstack Query) 를 이용하면,

> 1. Caching (캐싱)
> 2.



코드가 간결해짐

ispending, data, error ... 상태 만들고, useEffect 에서 http 요청 보내고...

커스텀 훅으로 재사용할 수 있음

하지만, 고급 기능이 빠져있음

refetch.. 캐싱 ...



상태관리를 비롯한 긴 코드 작성할 필요없고

캐시처리, ... 과 같은 기능들이 기본적으로 내장되어 있음



npm install @tanstack/react-query



```tsx
import { useQuery } from "@tanstack/react-query";

export default function MyComponent () {
    
}
```

useQuery 훅은 자체적으로 http 요청 전송,

필요한 이벤트 데이터 가져오고 로딩상태에 대한 정보를 제공함

useQuery 에 object를 전달

tanstack 쿼리는 http 요청 전송하는 로직이 내장되어잇지않음.

요청을 관리하는 로직을 제공함

요청에 관련된 데이터,, 발생가능한 오류 추적 ,,,



```tsx
const { isPending, data, isError, error, refetch } = useQuery({
    queryKey : ["", ...] // 모든 http 요청은 쿼리 키가 존재. 이 키를 이용해 요청으로 생성된 데이터를 캐시 처리함. 
    // 동일 요청을 나중에 전송시, 쿼리 키를 이용해 응답을 재사용 가능
    queryFn : // 쿼리함수.. 를 이용해 실제 요청을 보낼 코드
})
```

실제 요청 보내는 함수는 fetch 나 axios 사용해야함

queryFn 는 promise 를 반환하는 http 요청을 핸들링하는 함수를 넣러야함



```tsx
export const fetchPosts = async () => {
    const response = await fetch(URL);
    if(!response.ok) {
         const error = new Error();
         error.code = response.status;
         error.info = await response.json();
         throw error;
    }
    const data = await response.json();
    return data;
}
```

QueryClientProvider 로 감싼다 root 컴포넌트

```tsx
import { QueryClientProvider, QueryClient } from "@tanstack/react-query";

const queryClient = new QueryClient();

export default function App() {
    return (
        <QueryClientProvider client={queryClient}>
            ...
        </QueryClientProvider/>
    );
}
```





## 🔗 참고자료

[https://velog.io/@kandy1002/React-Query-%ED%91%B9-%EC%B0%8D%EC%96%B4%EB%A8%B9%EA%B8%B0](https://velog.io/@kandy1002/React-Query-%ED%91%B9-%EC%B0%8D%EC%96%B4%EB%A8%B9%EA%B8%B0)\
