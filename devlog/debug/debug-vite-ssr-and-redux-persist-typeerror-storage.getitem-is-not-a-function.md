# \[DEBUG] Vite SSR & Redux Persist, TypeError: storage.getItem is not a function

새로 진행하는 프로젝트에서 검색엔진 최적화가 필요할것 같아 NextJS 프레임워크를 사용하기로 했으나, 같은 팀 프론트엔드 팀원들이 아직 NextJS 에 대한 이해가 부족하기도 하고 프레임워크를 새로 학습하는데 시간이 많이 걸릴 것으로 생각되었습니다.

따라서 Vite SSR 을 적용해, 서버 사이드에서 렌더링된 페이지를 응답으로 보내주고,\
해당 페이지에 React Hydration 이후 Client Side Rendering 을 적용하기로 했습니다.

상태관리는 Redux Toolkit 을 사용하고,\
인증인가 토큰 또한 Redux 에 함께 관리하기 위해 Redux Persist 를 사용했습니다.

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

Vite SSR 가 서버사이드에서 렌더링을 한 페이지를 응답으로 잘 전송하다가,\
Redux Persist 를 사용하면서 부터 서버사이드 렌더링에 오류가 발생했습니다.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

오류가 발생해서 catch 까지 됐는데, \
`vite.ssrFixStacktrace( )` 에서 오류가 발생해,\
오류의 근원지를 찾기 위해 try catch 문을 제거하고 다시 실행 했습니다.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>



## ✳️ 원인

Redux Persist 에서 문제가 발생한 것은 확실하고, 거슬러 올라가보니

### 1. redux-persist/lib/getStoredState.js

`redux-persist/lib/getStoredState.js` 에서 `storage.getItem()` 을 호출하고 있었는데 여기서 버그가 발생한것으로 보였습니다.

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

이 `storage.getItem()` 은 `getStoredState` 가 호출하고,\
이 함수는 exports.default 로 내보내져 `persistReducer` 에서 호출됩니다.



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

실제 `store.ts` 에서 사용되어지고,\
해당 persistConfig 의 storage 에서 발생되는 원인임으로 좁혀 졌습니다.



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

해당 storage 가 구현된 부분을 확인해보니\
WEB API 인, localStorage 에 대한 타입을 정의해둔 곳이었습니다.\


### 4. Server Side 에서 WEB API ( local storage ) 가 없음

Server Side 에서 WEB API 인 Local Storage 가 없어서,\
Vite 이 Pre-Render 시 redux-persist 가 `getItem` 을 이용해 상태를 LocalStorage 로 부터 가져오는 것이 불가능하기 때문이었습니다.



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
Redux Store 또한 client.store.ts, server.store.ts 로 작성하여,\
클라이언트 사이드 / 서버사이드에서 `store` 를 분기하였습니다.

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

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

하지만, 서버에서 사전 렌더링한 HTML 에 hydration 이 다르게 되어\
클라이언트에서 렌더링이 한번 더 일어났고, 두번씩이나 렌더링 되었습니다



### 2. import.meta.env.SSR 로 reducer 다르게 할당하기

`storage.getItem()` 은 서버사이드 렌더링시 \
Redux Persist 의 `persist/REHYDRATE` action 을 dispatch 할때 내부적으로 사용되고 있었습니다.

따라서

> Server Side 에서 사전렌더링시에 실행될 때는 Root Reducer 를,
>
> Client Side 에서 렌더링시에는 persistReducer 를 리턴하도록 코드를 추가했습니다



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

로 수정 후 빌드 & 실행했습니다



결과는 ... ?

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

오류는 안생기는데, 응답으로 받은 HTML 을 확인하니\
사전렌더링에 실패했습니다



### 3. persistor.pause( )

Redux-Persist 는 번들이 로드 되면,\
`persist/PERSIST` 와 `persist/REHYDRATE` 액션이 디스패치되어\
localStorage, sessionStorage 로 부터 serialize 된 상태를 불러옵니다

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

따라서, \
애플리케이션의 진입점에서 서버인지 클라이언트인지 분기하고,\
서버인경우 `persistor.pause()` 를 이용해 `persist/PAUSE` 액션을 dispatch 하여,\
`persist/PERSIST` 그리고 `persist/REHYDRATE` 액션이 디스패치되는 것을 막았습니다



### 4. manualPersist

```typescript
export const persistor = persistStore(store, { manualPersist: true });
```

공식문서를 찾아보니,\
`persistStore` 에서 `{ manualPersist : true }` 옵션을 넣어주어 `persist/PERSIST` 액션이 dispatch 되는것을 막을 수 있었습니다.



결과는..?

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

SERVER SIDE PERSIST 가 실행되는것으로 보아 `persit/PAUSE` 액션은 dispatch 되었지만,\
`persist/PAUSE` 액션이 dispatch 되기 전에 `redux-persist failed to create sync storage.` 오류가 뜨는 것으로 보아, pause 이전에 storage 를 확인하는 것을 알 수 있었습니다.



### 4. Server Side Storage 구현하기

Redux Persist 라이브러리에서,\
`redux-persist failed to create sync storage` 를 출력하는 부분을 찾아보았습니다

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

`getStorage.ts` 에서 해당 오류를 출력하고, `hasStorage`, storage 가 없는경우 개발환경에서 출력되는 메시지였습니다.

따라서 서버측에서 사용될 serverSideStorage 를 구현하고,\
환경에 맞게 serverSideStorage, createWebStorage 로 분기하는 코드를 작성하였습니다

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
사전렌더링이 제대로 되지 않았습니다.

Sync Storage 는 잘 생성이 되는데... 사전렌더링이 제대로 되지 않은걸 보니,\
남은건 PersistGate 컴포넌트 때문인 것으로 원인이 좁혀졌습니다.



### 5. PersistGate 뜯어보기

redux-persist 원본 소스코드의 `/src/integration/react.ts` 에서\
`PersistGate` 컴포넌트를 뜯어보았습니다

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



이부분이 핵심인것 같습니다\
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



### 3. 서버사이드에서의 localStorage 구현

```typescript
import createWebStorage from "redux-persist/lib/storage/createWebStorage";

const serverSideStorage = () => {
    return {
        // eslint-disable-next-line @typescript-eslint/no-unused-vars
        getItem: (_key: string): Promise<string> => {
            return new Promise((resolve) => {
                resolve("{}");
            });
        },
        // eslint-disable-next-line @typescript-eslint/no-unused-vars
        setItem: (_key: string, _item: string): Promise<void> => {
            return new Promise((resolve) => {
                resolve();
            });
        },
    };
};

export const persistStorage = typeof window !== "undefined" ? createWebStorage("local") : serverSideStorage();
```

서버사이드에서 텅빈 상태를 되돌려주는 `serverSideStorage` 를 구현했고,\
window 객체가 정의되지 않은 서버사이드 환경에서는 `serverSideStorage` 를,\
클라이언트 환경에서는 `createWebStorage` 를 이용해 localStorage 를 반환해주었습니다



### 4. hydrateRoot 이후, persist/PERSIST 액션 디스패치

```tsx
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";

import App from "./App";

const root = ReactDOM.hydrateRoot(
    document.getElementById("app") as HTMLElement,
    <BrowserRouter>
        <App isClient={false} />
    </BrowserRouter>,
);

root.render(
    <BrowserRouter>
        <App isClient={true} />
    </BrowserRouter>,
);
```

entry-client.tsx 에서, hydrateRoot 를 이용한 Hydration 시에는 isClient 로 false props 를 넘겨주고, 하이드레이션이 끝난 후, true 를 넘겨 주도록 코드를 작성하였습니다



```tsx
export default function App({ isClient }: { isClient: boolean }) {
    useEffect(() => {
        if (isClient) persistor.persist();
    }, [isClient]);
```

이후 App.tsx 에서 isClient 인 경우, `persist.persist()` 를 이용해 `persist/PERSIST` 액션이 디스패치 되도록 설정하였습니다.\


### 5. 결과

서버사이드에서 렌더링된 HTML 파일이 응답으로 오는 것을 확인 할 수 있고,

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

\


<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

렌더링 이후 `persist/PERSIST` `persist/REHYDRATE` 액션이 잘 디스패치 되어,\
localStorage 로 부터 상태를 복구 하는 것을 확인 할 수 있습니다



<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

\
React Profiler 를 이용해 확인해보니,\
`hydrateRoot()` 이후, `PersistGate` 의 `bootstrapped` 상태가 변경되면서\
localStorage 로 부터 상태를 잘 가져와 렌더링하는것도 확인 할 수 있습니다.



## ✅ 요약

### 버그원인

서버 사이드 렌더링시에, WEB API 인 localStorage 의 부재로,\
bootstrapped 상태가 false 로 되어 로딩 상태를 렌더링하여 응답으로 반환

### 버그해결

1. 서버 사이드 렌더링시 localStorage 가 없는 에러를 방지하는데 사용될 가짜 `serverSideStorage` 함수 생성
2. PersistGate 에서 window 객체가 정의되지 않은 서버사이드 환경에서, loading 컴포넌트가 아닌, children 을 렌더링하도록 수정
3. persistConfig 에 manualPersist 프로퍼티를 이용해 서버사이드에서 `persist/PERSIST` , `persist/REHYDRATE` 액션이 자동으로 디스패치 되지 않도록 수정
4. entry-client.tsx 에서 Hydration 이후, render 시 isClient props 를 true 로 넘겨주어,\
   클라이언트 사이드에서 `persist/PERSIST` 액션이 디스패치 되도록 수정



