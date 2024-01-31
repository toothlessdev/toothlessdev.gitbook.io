# 리액트는 항상 단방향으로 데이터가 흐를까 ? useImperativeHandle

아님

forwardRef&#x20;

useImperativeHandle 훅 사용하면

자식 컴포넌트에서 부모 컴포넌트로 함수를 전달 할 수 있음

근데 리액트에서 별로 추천안함

단방향으로 설계되었기 때문.

ContextAPI 나 Redux 와 같은 전역상태 라이브러리 쓰는게 좋음.

