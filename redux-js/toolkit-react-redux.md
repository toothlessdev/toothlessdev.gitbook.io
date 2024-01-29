# Toolkit 없이 React 에서 Redux 사용하기

Toolkit 없이 Redux 만을 이용해 상태관리를 하는 방법에 대해 알아보겠습니다.\
이 포스트는 Redux 의 핵심개념 Flux 아키텍쳐에 대한 이해가 있다는 가정하에 작성되었습니다

Container : Redux 와 연결되어있는, Redux 의 일부 상태와 Dispatcher 를 포함하는 컴포넌트

> 1. Redux Store 생성
> 2. Redux Action Type 정의 및 Action Creator / Reducer 구현
> 3. React Component 구현
> 4. Container 만들기 (mapStateToProps, mapDispatchToProps, connect)
> 5. Provider 로 store 전달하기
> 6. Container Component 렌더링



## 📖 패키지 설치

```
npm install redux react-redux
```

react-redux redux를 react 에서 사용하기 위해 를 연결해주는 binding 라이브러리



## 📖 React-Redux

### ✏️ Provider 컴포넌트

```jsx
<Provider store={store}>
    <App/>
</Provider>
```

Provider 컴포넌트 : 하위 컴포넌트들이 Redux Store 에 접근 할 수 있게 해주는 컴포넌트입니다.

store prop에 Redux Store 를 넘기고, 하위 컴포넌트가 Redux Store 에 접근할 수 있도록 최상위 컴포넌트를 Provider 컴포넌트로 감싸주면 됍니다

