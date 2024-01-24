# \[DEBUG] ReduxToolkit ThunkAction dispatch 할때 타입 에러해결

## ❎ 에러

Redux Toolkit 의 ThunkAction 과 typescript 사용할때 dispatch 할때 에러가 발생함

```typescript
export const FetchPostsThunk = () => {
    return async (dispatch: Dispatch) => {
        try {
            dispatch(PostAction.setIsPending(true));
        } catch (err) {
            const response = await fetch(API_URL);
            const data = (await response.json()) as IGetAllPostsResponseBody;
            dispatch(PostAction.setData(data));
        } finally {
            dispatch(PostAction.setIsPending(false));
        }
    };
};
```

```typescript
const dispatch: Dispatch = useDispatch();

useEffect(() => {
    dispatch(FetchPostsThunk());
}, []);
```

<mark style="color:red;">Argument of type '(dispatch: Dispatch) => Promise' is not assignable to parameter of type 'UnknownAction'.</mark>

## ✳️ 원인

{% embed url="https://stackoverflow.com/questions/72396293/argument-of-type-dispatch-dispatchshopdispatchtypes-promisevoid-is-n" %}

> The problem is that `const dispatch = useDispatch();` only knows about the basic `Dispatch` type from the `redux` core. That `Dispatch` type does not know that thunks exist - it only accepts plain action objects. So, when you try to dispatch a thunk, it (correctly) errors.
>
> The fix is to follow our "Usage with TS" guidelines for correctly inferring an `AppDispatch` type from `store.dispatch`, and then defining pre-typed hooks that have the thunk types baked in:

useDispatch 가 리턴하는 Dispatch 타입이 기본적인 Action 만 dispatch 인식 하도록 되어있어 순수한 Action Object 만 인자로 받을 수 있다.

**ThunkAction 을 사용하여 Dispatch 할때, store.dispatch 에 대한 타입을 지정해주어야 한다**

## **✅ 해결**

store.ts 에서 store.dispatch 에 대한 타입을 export 해주고,

```typescript
export type RootDispatch = typeof store.dispatch;
```

```tsx
const dispatch: RootDispatch = useDispatch();

useEffect(() => {
    dispatch(FetchPostsThunk());
}, [dispatch]);
```

dispatch 에 대한 타입을 지정해주면 된다
