# Tanstack React Query 개념, useQuery 로 데이터 페칭하기

React Query (Tanstack Query) 는 React 에서 비동기 상태를 쉽게 처리할 수 있도록 도와주는 라이브러리입니다.

{% embed url="https://tanstack.com/query/latest/docs/framework/react/overview" %}

> Powerful asynchronous state management for\
> TS/JS, React, Solid, Vue, Svelte and Angular



## 📖 기존 데이터페칭 (서버 상태 관리) 의 단점

### ✏️ 복잡한 코드작성

기존에 서버로부터 데이터페칭을 하려면,\
컴포넌트 내에서 fetch 의 isLoading 상태 / error 상태 / data 상태 를 저장하는 state 를 만들고,\
useEffect 내부에서 fetch 함수에 대한 비동기 호출 처리 후 위의 상태를 조작해야합니다.

```tsx
export default function MyComponent() {
    const [isLoading, setIsLoading] = useState<boolean>(false);
    const [data, setData] = useState<ResponseBody | undefined>(undefined);
    const [error, setError] = useState<Error>();
    
    useEffect(() => {
        setIsLoading(true);
        
        fetch(API_URL)
            .then((response) => {
                return response.json();
            })
            .then((data) => {
                setData(data);
            })
            .catch((err) => {
                setError(err);
            })
            .finally(() => {
                setIsLoading(false);
            })
    }, [])
    
    return <></>
}
```



### ✏️ 복잡한 기능구현

사실 위의 데이터페칭 코드는 useFetch 와 같이 커스텀 훅으로 추상화 하여 사용하면, 어느정도 반복되는 코드 작성을 줄일 수 있습니다

```typescript
import { useEffect, useState } from "react";

export const useFetch = <ResponseBody>(API_URL: string, options?: RequestInit) => {
    const [isLoading, setIsLoading] = useState<boolean>(false);
    const [data, setData] = useState<ResponseBody | null>(null);
    const [isError, setIsError] = useState<boolean>(false);
    const [error, setError] = useState<Error | null>(null);

    useEffect(() => {
        fetch(API_URL, options)
            .then((response) => {
                return response.json();
            })
            .then((data) => {
                setData(data);
            })
            .catch((err) => {
                setIsError(true);
                setError(err);
            })
            .finally(() => {
                setIsLoading(false);
            });
    }, [API_URL, options]);

    return { isLoading, isError, data, error };
};
```

하지만,&#x20;

> 다른 화면을 보다가 다시 웹페이지에 포커스된경우 refetch 를 트리거 한다거나,
>
> 네트워크 상태가 변경되었을때 refetch 를 트리거 한다거나,
>
> POST 요청 후 GET 요청을 트리거 하고싶을때,
>
> 캐싱을 사용하여 성능 향상을 하고 싶을때 ...

등과 같은 상황에서 해당 기능들을 구현하기 위해서는 많은 시간이 소요됩니다.

**React Query (Tanstack Query) 를 이용하면, 서버상태와 비동기 처리를 효과적으로 관리하고,**\
**위 기능들을 손쉽게 사용할 수 있습니다**



## 📖 Tanstack React Query 를 사용하기에 앞서..

### ✏️ QueryClient

그렇다면 Tanstack React Query 는 캐싱, 중복 요청제거와 같은 기능을 어떻게 제공할까요?

Tanstack Query 는 먼저 애플리케이션이 시작되는 시점에 `QueryClient` 인스턴스를 생성하고,\
`QueryClientProvider` 를 통해 앱 전체를 감싸는 것으로 시작됩니다.

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

// 1 - QueryClient 인스턴스 생성
const queryClient = new QueryClient();

export default function App() {
    return (
        <QueryClientProvider client={queryClient}>
            { ... }
        </QueryClientProvider>
    );
}
```

이 `QueryClient` 는 React Context API 를 통해 Provider 내부에서 QueryClient 를 사용할 수 있게 되며,\
캐싱된 데이터는 QueryClient 의 `QueryCache` 에 내부적으로 저장됩니다.



## 📖 useQuery

`useQuery` 훅을 사용하면 서버 상태와 비동기 HTTP <mark style="background-color:orange;">**GET**</mark> 요청을 처리할 수 있습니다

주의해야할 점은, Tanstack Query 는 **HTTP 요청을 전송하는 로직이 내장되어 있지 않다**는점입니다.\
대신, 요청을 <mark style="background-color:orange;">**관리하는**</mark> 로직을 제공합니다

```tsx
import { useQuery } from "@tanstack/react-query";

export default function MyComponent = () => {
    const { isPending, data, isError, error } = useQuery({
        queryKey: [],
        queryFn: () => {}
    })
}
```

useQuery 훅을 사용하기 전에, \
반드시 포함해야하는 두가지 프로퍼티 `queryKey` 와 `queryFn` 에 대해 살펴보겠습니다



### ✏️ queryKey

모든 useQuery 에는 queryKey 배열이 할당되어있는데,\
Tanstack Query 는 이 <mark style="background-color:orange;">**queryKey**</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">를 기반으로 요청으로 생성된 데이터를 캐싱</mark>합니다.

이 queryKey 는 어떠한 값으로도 구성될 수 있습니다.\
하지만 queryKey 를 구성하는데 몇가지 주의해야할 점이 있습니다



> 1. **Query Keys are hashed deterministically**

❗️객체의 property 순서가 다른경우, 관계없이 동일한 쿼리 키로 취급됩니다&#x20;

```tsx
useQuery({ queryKey : ["posts", { id, page }], ... })
useQuery({ queryKey : ["posts", { page, id }], ... })
```

❗️배열의 순서가 다른경우, _**서로 다른**_  쿼리 키로 취급됩니다

```tsx
useQuery({ queryKey : ["posts", page, id], ... })
useQuery({ queryKey : ["posts", id, page], ... })
```



> 2. **If your query Function depends on a variable, include it in your query key**

queryKey 를 기반으로 데이터가 캐싱되고, 데이터를 식별하기 때문에,

❗️`queryKey` 에는 `queryFn` 에서 사용되는 변수가 포함되어야 합니다.

예를 들어, 특정 `id` 를 갖는 게시글 post 를 가져오는 경우 또는 제목으로 게시글을 검색하는경우, postId 와 searchKey 를 queryKey 로 넘겨주어야 합니다

```tsx
const usePostById = (postId : number) => {
    return useQuery({
        queryKey: ["posts", { id : postId }],
        queryFn: () => readPostById(postId)
    });
}
```

```tsx
const useSearchPost = (key : string) => {
    return useQuery({
        queryKey: ["posts", { searchKey : key }],
        queryFn: () => searchPost(key)
    });
}
```

{% embed url="https://tanstack.com/query/latest/docs/framework/react/guides/query-keys" %}

### ✏️ queryFn

queryFn 에는, `Promise` 를 반환하는, 실제 요청을 처리하는 비동기 함수를 전달해 주면 됩니다.\
return 되는 `Promise` 는 데이터를 resolve 하거나, 오류를 발생시키거나 reject 해야합니다.

❗️이때 비동기 함수를 호출하는 것이 아닌, 함수 객체 자체를 전달해야 함에 주의해야합니다.

따라서, 요청에 특정 body 나 param 이 필요한 경우, 화살표함수로 함수 객체를 만들어 넘겨주어야 합니다

```tsx
useQuery({
    queryKey: ["posts", { postId : id }],
    queryFn: () => readPostById(id)
    // queryFn : readPostById(id) 비동기 함수를 호출하면 안됨!
})
```



### ✏️ 이 외 여러가지 부가적인 옵션들

1. <mark style="background-color:orange;">**staleTime**</mark>** (기본값 : 0)**

staleTime 은 캐시된 데이터가 stale (오래된) 상태가 될때 까지의 시간입니다\
데이터가 stale 상태가 되기 전까지는 추가적인 데이터 요청을 하지 않고, 캐싱된 값을 사용합니다.

2. <mark style="background-color:orange;">**gcTime**</mark>** (기본값 : 5분, 300,000 ms)**

gcTime 은 캐시된 데이터가 삭제되기 까지의 시간입니다\
데이터가 stale 상태가 된 후, gcTime 이 지나면 캐시된 메모리에서 제거됩니다.\
이를 통해 캐시된 데이터의 메모리 사용을 관리하고 최적화 할 수 있습니다.

3. **onSuccess, onError**

데이터 요청이 성공했을 때 / 에러가 발생했을 때 호출되는 콜백함수 입니다

```tsx
useQuery({
    queryKey: ["posts"],
    queryFn: readPosts,
    onSuccess: (data) => { },
    onError: (err) => { }
})
```

4. **enabled**

쿼리가 활성화 될지의 여부를 제어할 수 있습니다. 이를통해 특정 조건에서 쿼리를 비활성화 할 수 있습니다

```tsx
useQuery({
    queryKey: ["posts"],
    queryFn: () => readPostById(postId),
    enabled: postId !== null, // postId 가 null 이 아닌 경우 쿼리가 활성화 됩니다
})
```

...



### ✏️ useQuery 가 반환하는 여러 값

1. <mark style="background-color:orange;">**isPending**</mark>** : queryFn 이 리턴하는 Promise 객체의 pending 상태를 반환합니다**
2. <mark style="background-color:orange;">**data**</mark>** : queryFn 이 성공적으로 Promise 를 resolve 하는 경우, resolve 된 result 를 반환합니다**
3. <mark style="background-color:orange;">**isError, error**</mark>** : queryFn 에서 오류가 발생하거나 Promise 가 reject 된 경우, error 를 반환합니다**
4. <mark style="background-color:orange;">**refetch**</mark>** : 해당 쿼리에 대해 재요청을 진행합니다**

...





{% embed url="https://tanstack.com/query/latest/docs/framework/react/reference/useQuery" %}

## 📖 Invalidate Query 쿼리 무효화

### ✏️ Invalidate Query

`queryClient.invalidateQueries` 를 사용하면, 조건에 맞는 쿼리를 무효화 할 수 있습니다.

예를들어, 특정 게시글을 작성 POST (mutate) 한 이후, 게시글 목록에 해당하는 쿼리를 무효화 (stale 상태로 만들고) 합니다.

```tsx
const { isPending, mutate } = useMutation({
    mutationFn: createPost,
    onSuccess: () => {
        queryClient.invalidateQueries({ queryKey: ["posts"] })
        // "posts" 를 queryKey 로 가지고 있는 query 들을 stale 상태로 만듭니다
    }
});

mutate();
```



### ✏️ Query Invalidation VS refetch

그렇다면 Query 를 Invalidate 하는 것과, useQuery 가 반환하는 refetch 를 호출하는 것에는 어떤 차이가 있고, 무엇을 사용해야 할까요?

공식 라이브러리의 Discussion 을 살펴보면,

{% embed url="https://github.com/TanStack/query/discussions/2468" %}

> invalidation is a more "smart" refetching. Refetching will always refetch, even if there are no observers for the query. invalidation will just mark them as stale so that they refetch the next time an observer mounts. For queries with active observers, there is no difference as far as I'm aware.

즉,

Query Invalidation 은, stale 상태로 만들어 해당 쿼리가 사용이 될때 (예를 들어 해당 데이터를 사용하는 컴포넌트가 마운트 될 때), 새로운 데이터를 가져오고,\
refetch 는 항상 데이터를 새로 가져옵니다.







## 🔗 참고자료

[https://velog.io/@kandy1002/React-Query-%ED%91%B9-%EC%B0%8D%EC%96%B4%EB%A8%B9%EA%B8%B0](https://velog.io/@kandy1002/React-Query-%ED%91%B9-%EC%B0%8D%EC%96%B4%EB%A8%B9%EA%B8%B0)

[https://tkdodo.eu/blog/inside-react-query](https://tkdodo.eu/blog/inside-react-query)

\
