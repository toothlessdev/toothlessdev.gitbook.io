# \[DEBUG] Vite SSR & Redux Persist, TypeError: storage.getItem is not a function

프론트엔드 팀원들이 아직 NextJS 에 대한 이해가 부족했지만,\
Vite 을 이용해 프로젝트를 생성하되, Vite SSR Plugin 을 적용해 사전렌더링된 페이지에서 CSR 을 적용해보기로 했다

상태관리는 Redux Toolkit 을 사용하고, 인증인가 토큰 또한 Redux 에 함께 관리하기 위해 Redux Persist 를 사용했다

```javascript
/* eslint-disable no-undef */
import express from "express";
import fs from "fs";
import path from "path";
import { createServer as createViteServer } from "vite";

const PORT = process.env.PORT || 5173;
const BASE = process.env.BASE || "/";
const PRODUCTION = process.env.NODE_ENV === "production";

const TEMPLATE = PRODUCTION ? fs.readFileSync("./dist/client/index.html", { encoding: "utf-8" }) : "";
const SSR_MANIFEST = PRODUCTION ? fs.readFileSync("./dist/client/ssr-manifest.json", { encoding: "utf-8" }) : undefined;

const app = express();
let vite;

// ? Add vite or respective production middlewares
if (!PRODUCTION) {
    vite = await createViteServer({
        server: { middlewareMode: true },
        appType: "custom",
    });
    app.use(vite.middlewares);
} else {
    const sirv = (await import("sirv")).default;
    const compression = (await import("compression")).default;
    app.use(compression());
    app.use(
        BASE,
        sirv("./dist/client", {
            extensions: [],
            gzip: true,
        }),
    );
}

app.use("*", async (req, res, next) => {
    if (req.originalUrl === "/favicon.ico") {
        return res.sendFile(path.resolve("./public/logo.svg"));
    }
    let template, render;

    try {
        if (!PRODUCTION) {
            template = fs.readFileSync(path.resolve("./index.html"), "utf-8");
            template = await vite.transformIndexHtml(req.originalUrl, template);
            render = (await vite.ssrLoadModule("/src/entry-server.tsx")).render;
        } else {
            template = TEMPLATE;
            render = (await import("./dist/server/entry-server.js")).render;
        }
        const rendered = await render({ path: req.originalUrl }, SSR_MANIFEST);
        const html = template.replace(`<!--app-html-->`, rendered ?? "");
        res.status(200).setHeader("Content-Type", "text/html").end(html);
    } catch (error) {
        console.log(error);
        vite.ssrFixStacktrace(error);
        next(error);
    }
});

app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});

```





## ❎ 에러

Redux Persist 를 적용하기 전에는 Vite SSR 이 서버사이드에서 Pre-Render 가 잘 진행하다가\
Persist 를 적용한 뒤부터 오류가 발생함

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

오류가 발생해서 catch 까지 됐는데, 오류의 근원지를 찾기 위해 try catch 문을 제거하고 다시 실행함

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>



## ✳️ 원인

Redux Persist 에서 문제가 발생한 것은 확실하고, 거슬러 올라가보니

### 1. redux-persist/lib/getStoredState.js

`redux-persist/lib/getStoredState.js` 에서 `storage.getItem()` 을 호출하고 있었는데 여기서 버그가 발생한것 같다.

```javascript
"use strict";

exports.__esModule = true;
exports.default = getStoredState;

var _constants = require("./constants");

function getStoredState(config) {
  // .. 어쩌구 저쩌구
  var storage = config.storage;
  // .. 어쩌구 저쩌구

  return storage.getItem(storageKey).then(function (serialized) {
    // .. 어쩌구 저쩌구
  });
}
```

그리고 이 `storage.getItem()` 은 `getStoredState` 가 호출하고,\
이 함수는 exports.default 로 내보내 져서 `persistReducer` 에서 호출된다.



### 2. redux-persist/lib/persistReducer.js

`redux-persist/lib/persistReducer.js` 에서 exports.default 로 내보내지는 persistReducer 는

```typescript
import { persistReducer, persistStore } from "redux-persist";

import { authSlice } from "./slice/auth.slice";
import { pageSlice } from "./slice/page.slice";
import { userInterfaceSlice } from "./slice/userInterface.slice";
import { configureStore, combineReducers } from "@reduxjs/toolkit";
import storage from "redux-persist/lib/storage";

const persistConfig = {
    key: "root",
    storage,
    whitelist: ["auth"],
};

const rootReducer = combineReducers({
    auth: authSlice.reducer,
    userInterface: userInterfaceSlice.reducer,
    page: pageSlice.reducer,
});

export const store = configureStore({
    reducer: persistReducer(persistConfig, rootReducer),
});

export type RootState = ReturnType<typeof store.getState>;

export const persistor = persistStore(store);

```

실제 `store.ts` 에서 사용되어지고, 해당 persistConfig 의 storage 에서 발생되는 원인임으로 좁혀 졌다



### 3. redux-persist/lib/storage

```typescript
declare module "redux-persist/es/storage" {
  import { WebStorage } from "redux-persist/es/types";

  const localStorage: WebStorage;
  export default localStorage;
}

declare module "redux-persist/lib/storage" {
  export * from "redux-persist/es/storage";
  export { default } from "redux-persist/es/storage";
}

```

해당 storage 가 있는곳을 보니 WEB API 인, localStorage 에 대한 타입을 정의해둔 곳이었다\


### 4. Server Side 에서 WEB API ( local storage ) 가 없음

Server Side 에서 Local Storage 가 없어서, Vite 이 Pre-Render 시 redux-persist 가 getItem 을 이용해 상태를 LocalStorage 로 부터 가져오는 것이 불가능하기 때문으로 추정됨



## ❎ 삽질...

### 1. entry-client.tsx, entry-server.tsx 에서 각각 다른 store 를 사용

```tsx
ReactDOM.hydrateRoot(
    document.getElementById("app") as HTMLElement,
    <BrowserRouter>
        <Provider store={store}>
            <PersistGate loading={null} persistor={persistor}>
                <App />
            </PersistGate>
        </Provider>
    </BrowserRouter>,
);
```

```tsx
export const render = ({ path }: IRenderProps) => {
    return ReactDOMServer.renderToString(
        <Provider store={store}>
            <StaticRouter location={path}>
                <App />
            </StaticRouter>
        </Provider>,
    );
};
```

로 entry-client.tsx entry-server.tsx 파일을 만들고,\
Redux Store 도 client.store.ts, server.store.ts 로 만들었다

<pre class="language-typescript"><code class="lang-typescript"><strong>// client.store.ts
</strong><strong>import { persistReducer, persistStore } from "redux-persist";
</strong>
import { authSlice } from "./slice/auth.slice";
import { pageSlice } from "./slice/page.slice";
import { userInterfaceSlice } from "./slice/userInterface.slice";
import { configureStore, combineReducers } from "@reduxjs/toolkit";
import storage from "redux-persist/lib/storage";

const persistConfig = {
    key: "root",
    storage,
    whitelist: ["auth"],
};

const rootReducer = combineReducers({
    auth: authSlice.reducer,
    userInterface: userInterfaceSlice.reducer,
    page: pageSlice.reducer,
});

export const store = configureStore({
    reducer: persistReducer(persistConfig, rootReducer),
});

export type RootState = ReturnType&#x3C;typeof store.getState>;

export const persistor = persistStore(store);

</code></pre>

```typescript
// server.store.ts
import { authSlice } from "./slice/auth.slice";
import { pageSlice } from "./slice/page.slice";
import { userInterfaceSlice } from "./slice/userInterface.slice";
import { configureStore, combineReducers } from "@reduxjs/toolkit";

const rootReducer = combineReducers({
    auth: authSlice.reducer,
    userInterface: userInterfaceSlice.reducer,
    page: pageSlice.reducer,
});

export const store = configureStore({
    reducer: rootReducer,
});

export type RootState = ReturnType<typeof store.getState>;

```

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

서버에서 사전 렌더링한 html 에 hydration 이 다르게 되어 두번씩이나 렌더링 됨



### 2. import.meta.env.SSR 로 reducer 다르게 할당하기

결국 서버사이드에서 사전렌더링시에 Web Storage API 가 없어서 발생하는 문제이고,\
`storage.getItem()` 은 사전렌더링시 Redux Persist 의 `persist/REHYDRATE` action 을 dispatch 할때 내부적으로 사용됨.

따라서

> Server Side 에서 사전렌더링시에 실행될 때는 Root Reducer 를,
>
> Client Side 에서 렌더링시에는 persistReducer 를 리턴하도록 작성했다



{% embed url="https://ko.vitejs.dev/guide/ssr#conditional-logic" %}

Vite 에서는 SSR 환경인지 `import.meta.env.SSR` 라는 환경변수를 이용해 조건을 분기할 수 있어서

```typescript
// store.ts

// ...

let store;
if (import.meta.env.SSR) {
    store = configureStore({
        reducer: rootReducer,
    });
} else {
    store = configureStore({
        reducer: persistReducer(persistConfig, rootReducer),
    });
}

export { store };

// ...

```

로 수정 후 빌드 & 실행했다



결과는 ... ?

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

오류는 안생기는데, 페이지 소스보기를 누르니 사전렌더링에 실패했다.



### 3. persistor.pause( ) 와 persistStore 의 manualPersist&#x20;

```typescript
export default function App() {
    if (typeof window === "undefined") {
        console.log("SERVER SIDE PERSIST");
        persistor.pause();
    } else {
        console.log("CLIENT SIDE PERSIST");
        persistor.persist();
    }
```

애플리케이션의 진입점에서 서버인지 클라이언트인지 분기하고,\
서버인경우 `persistor.pause()` 를 이용해 `persist/PAUSE` 액션을 dispatch 합니다

```typescript
export const persistor = persistStore(store, { manualPersist: true });
```

이후, `persistStore` 에서 `{ manualPersist : true }` 옵션을 넣어주어 `persist/PERSIST` 액션이 dispatch 되는것을 막습니다.

결과는..?

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

SERVER SIDE PERSIST 가 실행되는것으로 보아 `persit/PAUSE` 액션은 dispatch 되었지만,\
`persist/PAUSE` 액션이 dispatch 되기 전에 `redux-persist failed to create sync storage.` 오류가 뜨는 것으로 보아, pause 이전에 storage 가 생성되는것을 확인 할 수 있습니다.



### 4. Server Side Storage 구현하기

`redux-persist failed to create sync storage` 를 출력하는 부분을 찾아보겠습니다

```typescript
export default function getStorage(type: string): Storage {
  const storageType = `${type}Storage`
  if (hasStorage(storageType)) return (self as unknown as { [key: string]: Storage })[storageType]
  else {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        `redux-persist failed to create sync storage. falling back to noop storage.`
      )
    }
    return noopStorage
  }
}
```

getStorage.ts 에서 해당 오류를 출력하고, hasStorage, storage 가 없는경우 개발환경에서 출력되는 메시지였습니다.

따라서 서버측에서 사용될 serverSideStorage 를 구현하고,\
환경에 맞게 serverSideStorage, createWebStorage 로 분기하였습니다

```typescript
const serverSideStorage = () => {
    return {
        getItem: (key: string): Promise<null> => {
            console.log("Server Side getItem() called");
            return new Promise((resolve) => {
                resolve(null);
            });
        },
        setItem: (key: string, item: string): Promise<void> => {
            console.log("Server Side setItem() called");
            return new Promise((resolve) => {
                resolve();
            });
        },
    };
};
```

```typescript
const persistStorage = import.meta.env.SSR ? serverSideStorage() : createWebStorage("local");
```

```typescript
const persistConfig = {
    key: "root",
    storage: persistStorage,
    whitelist: ["auth"],
};
```

`redux-persist failed to create sync storage.` 오류는 발생하지 않는데,\
사전렌더링이 제대로 되지 않습니다

Sync Storage 는 잘 생성이 되는데... 사전렌더링이 제대로 되지 않은걸 보니,\
남은건 PersistGate 컴포넌트 때문에 Hydration 시에 Mismatch 가 발생하는 것으로 보입니다



### 5. PersistGate 뜯어보기

redux-persist 의 `/src/integration/react.ts` 에서 `PersistGate` 컴포넌트를 뜯어보겠습니다

```tsx
export class PersistGate extends PureComponent<Props, State> {
    static defaultProps = {
        children: null,
        loading: null,
    };

    state = {
        bootstrapped: false,
    };
    _unsubscribe?: () => void;
```

우선 children 과 loading 을 props 로 받습니다



이부분이 핵심인것 같습닌다\
컴포넌트가 마운트 될때 `handlePersistorState()` 가 호출되는데,\
`bootstrapped` 라는 값을 `persistor.getState()` 로 부터 가져옵니다



`bootstrapped` 는 persistor 의 상태에서 가져오는데,\
이 `persistor` 는 컴포넌트 사용시 props 로 넘겨받게 되고, `persistStore` 에 의해 생성됩니다

`persistor/REHYDRATE` 액션이 디스패치되면, `bootstrapped` 는 true 가 됩니다.

```tsx
    componentDidMount(): void {
        this._unsubscribe = this.props.persistor.subscribe(this.handlePersistorState);
        this.handlePersistorState();
    }

    handlePersistorState = (): void => {
        const { persistor } = this.props;
        const { bootstrapped } = persistor.getState();
        if (bootstrapped) {
            if (this.props.onBeforeLift) {
                Promise.resolve(this.props.onBeforeLift()).finally(() => this.setState({ bootstrapped: true }));
            } else {
                this.setState({ bootstrapped: true });
            }
            this._unsubscribe && this._unsubscribe();
        }
    };

```





위에서 확인했던 `this.state.boostrapped` 가 참이면,\
즉 `persist/REHYDRATE` 액션이 디스패치 되었다면, `this.props.children` 을 렌더링하고\
아닌경우 `this.props.loading` 을 렌더링합니다

```tsx

    componentWillUnmount(): void {
        this._unsubscribe && this._unsubscribe();
    }

    render(): ReactNode {
        if (process.env.NODE_ENV !== "production") {
            if (typeof this.props.children === "function" && this.props.loading)
                console.error(
                    "redux-persist: PersistGate expects either a function child or loading prop, but not both. The loading prop will be ignored.",
                );
        }
        if (typeof this.props.children === "function") {
            return this.props.children(this.state.bootstrapped);
        }

        return this.state.bootstrapped ? this.props.children : this.props.loading;
    }
}
```



서버사이드에서 사전렌더링시에, `persist/REHYDRATE` 는 WEB Storage API 가 없어서\
localStorage, sessionStorage ... 로 부터 상태를 가져오지 못하고,\
따라서 `this.props.loading` 상태가 출력됩니다.



공식문서에 loading props 에 null 을 넣으라고 되어있어,\
사전렌더링시에 텅빈 HTML 이 출력되었었는데,&#x20;

```markup
<!doctype html>
<html lang="ko">
    ...
    
    <body>
        <div id="app">
            loading
        </div>
        <script type="module" src="/src/entry-client.tsx"></script>
    </body>
</html>
```

실제로 loading props 에 "loading" 을 넣으니, 다음과 같이 loading 이 사전 렌더링 되었습니다



## ✅ 해결

### 1. Custom PersistGate 생성

Redux-Persist 리포지토리에서 `/src/integration/react` 의 `PersistGate` 를 복사해왔습니다

```tsx
import { PureComponent, ReactNode } from "react";
import { Persistor } from "redux-persist";

type Props = {
    onBeforeLift?: () => void;
    children: ReactNode | ((state: boolean) => ReactNode);
    loading: ReactNode;
    persistor: Persistor;
};

type State = {
    bootstrapped: boolean;
};

export class PersistGate extends PureComponent<Props, State> {
    static defaultProps = {
        children: null,
        loading: null,
    };

    state = {
        bootstrapped: false,
    };
    _unsubscribe?: () => void;

    componentDidMount(): void {
        this._unsubscribe = this.props.persistor.subscribe(this.handlePersistorState);
        this.handlePersistorState();
    }

    handlePersistorState = (): void => {
        const { persistor } = this.props;
        const { bootstrapped } = persistor.getState();
        if (bootstrapped) {
            if (this.props.onBeforeLift) {
                Promise.resolve(this.props.onBeforeLift()).finally(() => this.setState({ bootstrapped: true }));
            } else {
                this.setState({ bootstrapped: true });
            }
            this._unsubscribe && this._unsubscribe();
        }
    };

    componentWillUnmount(): void {
        this._unsubscribe && this._unsubscribe();
    }

    render(): ReactNode {
        if (process.env.NODE_ENV !== "production") {
            if (typeof this.props.children === "function" && this.props.loading)
                console.error(
                    "redux-persist: PersistGate expects either a function child or loading prop, but not both. The loading prop will be ignored.",
                );
        }
        if (typeof this.props.children === "function") {
            return this.props.children(this.state.bootstrapped);
        }

        /* =============================== 추가된 코드 =============================== */
        if (typeof window === "undefined") return this.props.children;
        /* 서버사이드에서는 persist/HYDRATE 가 정상적으로 일어나지 않아 항상 loading 이 렌더링됩니다 */
        /* 따라서 window 객체가 정의되지 않은 서버사이드에서는 this.props.children 을 렌더링합니다 */
        /* =============================== 추가된 코드 =============================== */
        
        return this.state.bootstrapped ? this.props.children : this.props.loading;
    }
}

```

마지막 `render()` 에서,\
`if(typeof window === "undefined") return this.props.children;` 을 이용해\
서버사이드에서는 `persist/REHYDRATE` 액션의 dispatch 에 상관없이 `children props` 를 리턴하는 코드를 추가했습니다.



### 2. store.ts 의 persistStore 에서 { manualPersist: true } 지정

```typescript
export const persistor = persistStore(store, { manualPersist: true } as PersistorOptions);
```

{% embed url="https://github.com/rt2zz/redux-persist?tab=readme-ov-file#persiststorestore-config-callback" %}

> * **config** _object_ (typically null)
>   * If you want to avoid that the persistence starts immediately after calling `persistStore`, set the option manualPersist. Example: `{ manualPersist: true }` Persistence can then be started at any point with `persistor.persist()`. You usually want to do this if your storage is not ready when the `persistStore` call is made.

공식문서에 따르면 `{ manualPersist : true }` 를 지정해주면,\
`persistor.persist()` 를 수동으로 호출할때 PERSIST, REHYDRATE 액션이 디스패치된다고 합니다

서버사이드에서는 PERSIST, REHYDRATE 는 사용이 불가하므로\
persistStore 에서 옵션으로 `{ manualPersist : true }` 를 넣어 자동으로 액션 디스패치 되는것을 막아줍니다



### 3. 진입점 App.tsx 에서 persistor.persist( ) 호출

이제 사전렌더링은 정상적으로 작동합니다\


<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

하지만 아직 Web Storage <- -> Redux Persist 사이에 동기화가 자동으로 되지않아,\
수동으로 `persistor.persist()` 를 호출해야 합니다.

```tsx
export default function App() {
    if (typeof window !== "undefined") {
        persistor.persist();
    } else {
        persistor.pause();
    }
    
    return (
        <>
            <Provider store={store}>
                <PersistGate loading={"Loading"} persistor={persistor}>
                    <Routes>
                        <Route path="/" element={<MainLayout />}>
                            <Route path="/" element={<HomePage />}></Route>
                            <Route path="auth/signin" element={<SignInPage />}></Route>
                            <Route path="auth/signup" element={<SignUpPage />}></Route>
                            <Route path="auth/findpw" element={<FindPasswordPage />}></Route>
                        </Route>
                    </Routes>
                </PersistGate>
            </Provider>
        </>
    );
}
```

앱의 진입점에서 클라이언트 사이드인 경우,\
`persistor.persist()` 를 호출해서 Redux Persist 가 저장소와 상태를 동기화 할 수 있도록 설정합니다

이후, PersistGate 의 loading props 로 null 이 아닌 값을 넣어주어,\






[https://ko.react.dev/reference/react-dom/client/hydrateRoot#usage](https://ko.react.dev/reference/react-dom/client/hydrateRoot#usage)

\






