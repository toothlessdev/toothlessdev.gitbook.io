# React Query

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
