# 리액트는 항상 단방향으로 데이터가 흐를까 ? useImperativeHandle

## 📖 단방향 vs 양방향 데이터 바인딩

리액트는 데이터의 <mark style="background-color:orange;">단방향</mark> 흐름을 중요하게 여깁니다.\
이는 상위 컴포넌트에서 하위 컴포넌트로 `props` 를 통해 데이터를 전달하는방식으로, <mark style="background-color:orange;">데이터 흐름을 예측 가능하게 만들어</mark> 관리가 용이합니다

```tsx
function App() {
    const [count, setCount] = useState(0);
    return (
        <div>
            <Counter count={count}/>
            <button onClick={() => setCount(count+1)}>Increase</button>
        </div>
    );
}

function Counter({ count }){
    return <p>count : {count}</p>
}
```

그렇다면 리액트는 항상 **단방향 데이터 바인딩** 만을 지원하는 것일까요?

> 결론부터 말하자면 리액트도 양방향 데이터 바인딩을 지원합니다



### 📖 useImperativeHandle

### ✏️ ref 와 forwardRef

`useImperativeHandle` 을 이해하기 위해서는 먼저 `React.forwardRef` 를 먼저 알아야 합니다.

`ref` 는 `useRef` 에서 반환한 객체로,\
컴포넌트 또는 DOM 요소의 ref 라는 특별한 props 에 넣어, HTML 에 접근하는 용도로 흔히 사용됩니다.

만약, `ref` 를 상위 컴포넌트에서 하위 컴포넌트로 전달하려면 어떻게 해야 할까요?

```jsx
function App()
    const ref = useRef<HTMLInputElement>(null);
    
    return (
        <div>
            <Input ref={ref}/>
            
        </div>
    );
}
```



아님

forwardRef&#x20;

useImperativeHandle 훅 사용하면

자식 컴포넌트에서 부모 컴포넌트로 함수를 전달 할 수 있음

근데 리액트에서 별로 추천안함

단방향으로 설계되었기 때문.

ContextAPI 나 Redux 와 같은 전역상태 라이브러리 쓰는게 좋음.

