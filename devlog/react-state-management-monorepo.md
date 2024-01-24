# React-State-Management을 위한 MonoRepo 구성

2024년 1월 목표인 redux, redux-saga, redux-toolkit 에 대해 공부하기위해 React-State-Management 리포지토리를 생성했다

거기에 리액트에서 상태관리를 공부하며 배운내용을 실습하며 정리할 계획이다.

react-redux, react-redux-toolkit, react-query 라는 폴더를 만들고 각각의 폴더에서 npm create vite 을 하려고 했는데 node\_modules 크기가 너무 커지고 중복되어, 이걸 해결할 방법이 없을까 생각을 하다, MonoRepo 라는걸 어디서 주워 들어서 한번 사용해 볼것이다



1. package.json 생성하기

```
npm init -y
```

2. 생성된 package.json 에 workspaces 추가하기

````
```json
{
    ...
    "private": true,
    "workspaces": [
        "common",
        "react-redux",
        "react-redux-toolkit",
        "react-query"
    ]
}

```
````

3. 각각 디렉토리 만들기
4. Workspace 초기화 하기

```
npm init -w ./common
npm init -w ./react-redux
npm init -w ./react-redux-toolkit
npm init -w ./react-query
```

5. npm install

```
npm install
```



WorkSpace 에 패키지 설치하는 경우

```
npm install <PACKAGE> -w <WORKSPACE>
```

WorkSpace 에 스크립트 실행하는 경우

```
npm run <SCRIPT> -w <WORKSPACE>
```



모든 Workspace 에서 vite, typescript, react 를 사용할 수 있도록&#x20;

common 디렉토리를 만들고 패키지를 설치해주었다

