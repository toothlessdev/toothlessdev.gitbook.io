# Redux Persist 뜯어보기

## 📖 Redux Persist 사용법

Redux Toolkit 과 Redux Persist 를 사용하면 자바스크립트 메모리에 저장되어 새로고침시 날아가는 상태들을 LocalStorage 또는 SessionStorage 에 저장할 수 있습니다.

[https://github.com/rt2zz/redux-persist#basic-usage](https://github.com/rt2zz/redux-persist#basic-usage)

### ✏️ 1. persistConfig 설정

```typescript
import storage from "redux-persist/lib/storage";

const persistConfig = {
    key: "root",
    storage: storage,
    whitelist: ["auth"],
};
```

persistor 의 식별자 key 값을 지정하고,\
사용할 storage 를 redux-persist/lib 로 부터 불러옵니다. 저는 localstorage 로 설정했습니다

whitelist 에서는 reducer 중 auth reducer 만 영구저장하도록 설정했습니다.



### ✏️ 2. persistReducer  와 persistStore 만들기

```typescript
const rootReducer = combineReducers({
    // ...
});

export const store = configureStore({
    reducer: persistReducer(persistConfig, rootReducer),
});

export const persistor = persistStore(store);
```

combineReducers 로 reducer 를 묶고, 해당 rootReducer 를 persistReducer 에 넣어줍니다

그리고 reducer 로 만들어진 store 를 persistStore 를 이용해 persistor 로 만들어 내보냅니다.



### ✏️ 3. PersistGate 설정

애플리케이션의 진입점에 PersistGate 컴포넌트를 설정해줍니다

```tsx
<Provider store={store}>
    <PersistGate loading={null} persistor={persistor}>
        <App/>
    </PersistGate>
</Provider>
```

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

Redux Devtool 을 확인해보면, 애플리케이션 초기 실행시에 Redux Persist 에 의해\
`persist/PERSIST` 액션과 `persist/REHYDRATE` 액션이 dispatch 되는것을 알 수 있습니다



## 📖 persistReducer 뜯어보기

[https://github.com/rt2zz/redux-persist](https://github.com/rt2zz/redux-persist) 를 clone 하여\
실제 dispatch 되는 액션들이 들어있는 persistReducer 함수 를 들여다 보겠습니다.



1. 먼저 config 의 설정값을 검사합니다. 필수 설정 프로퍼티값이 있는지... 등을 검사합니다

```typescript
export default function persistReducer<S, A extends Action>(
  config: PersistConfig<S>,
  baseReducer: Reducer<S, A>
): Reducer<S & PersistPartial, AnyAction> {
  if (process.env.NODE_ENV !== 'production') {
    if (!config) throw new Error('config is required for persistReducer')
    if (!config.key) throw new Error('key is required in persistor config')
    if (!config.storage)
      throw new Error(
        "redux-persist: config.storage is required. Try using one of the provided storage engines `import storage from 'redux-persist/lib/storage'`"
      )
  }
```



2. version, stateReconciler, getStoredState, timeout 등을 가져옵니다

```typescript
const version =
  config.version !== undefined ? config.version : DEFAULT_VERSION
const stateReconciler =
  config.stateReconciler === undefined
    ? autoMergeLevel1
    : config.stateReconciler
const getStoredState = config.getStoredState || defaultGetStoredState
const timeout =
  config.timeout !== undefined ? config.timeout : DEFAULT_TIMEOUT

```



3. `_persistoid` `_purge` `_paused` 값을 초기화합니다

\_persistoid 는 persist + oid (object identifier), 영구저장할 Object 를 저장하는것으로 추정됩니다

```typescript
let _persistoid: Persistoid | null = null
let _purge = false
let _paused = true
const conditionalUpdate = (state: any) => {
  // update the persistoid only if we are rehydrated and not paused
  state._persist.rehydrated &&
    _persistoid &&
    !_paused &&
    _persistoid.update(state)
  return state
}
```



4. 이후 액션을 처리할 Reducer 를 리턴합니다

```typescript
return (state: any, action: any) => // ...
```

해당 리듀서의 로직을 한번 뜯어보겠습니다



***







## 📖 PERSIST Action 뜯어보기

Redux Persist 에서 dispatch 할 수 있는 action 의 type 은\
`PERSIST` `PURGE` `FLUSH` `PAUSE` `REHYDRATE` 가 있습니다

```typescript
return (state: any, action: any) => {
  const { _persist, ...rest } = state || {}
  const restState: S = rest
  // ...
```

우선 state 에서 \_persist 를 제외한 객체를 rest 로 받습니다

### ✏️ \_rehydrate

PERSIST Action 은\
영구저장 storage 로 부터 state 를 복구하는 액션입니다

```typescript
  if (action.type === PERSIST) {
    let _sealed = false
    const _rehydrate = (payload: any, err?: Error) => {
      // dev warning if we are already sealed
      // .... (중략) ....

    // @NOTE PERSIST resumes if paused.
    _paused = false
    
    // @NOTE only ever create persistoid once, ensure we call it at least once, even if _persist has already been set
    if (!_persistoid) _persistoid = createPersistoid(config)
```

이후 createPersistoid 를 이용해 영구저장할 객체를 생성합니다.

### ✏️ createPersistoid.ts

잠시 createPersistoid 를 한번 들여다보면,\
config 로 부터 storage, key 를 이용해 저장할 storage 와 해당 storage 에서 저장한 상태를 식별자로 사용할 key 값을 만듭니다.

```typescript
// createPersistoid.ts
export default function createPersistoid(config: PersistConfig<any>): Persistoid {
  // defaults
  const blacklist: string[] | null = config.blacklist || null
  const whitelist: string[] | null = config.whitelist || null
  const transforms = config.transforms || []
  const throttle = config.throttle || 0
  const storageKey = `${
    config.keyPrefix !== undefined ? config.keyPrefix : KEY_PREFIX
  }${config.key}`
  const storage = config.storage
```

```typescript
  let serialize: (x: any) => any
  if (config.serialize === false) {
    serialize = (x: any) => x
  } else if (typeof config.serialize === 'function') {
    serialize = config.serialize
  } else {
    serialize = defaultSerialize
  }
```

이후, serialize 를 받는데, 설정을 하지 않을 경우,

```typescript
function defaultSerialize(data: any) {
  return JSON.stringify(data)
}
```

상태값들을 JSON.stringify 를 이용해 문자열로 치환합니다.\
실제로 Persist 를 적용시키고 LocalStorage 를 확인하면 문자열로 치환되어 저장된 것을 확인 할 수 있습니다.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>



다시 persistReducer.ts 로 돌아가보겠습니다

이미 PERSIST 액션을 dispatch 된 경우, 기본 reducer 만 dispatch 합니다

```typescript
    // @NOTE PERSIST can be called multiple times, noop after the first
    if (_persist) {
      // We still need to call the base reducer because there might be nested
      // uses of persistReducer which need to be aware of the PERSIST action
      return {
        ...baseReducer(restState, action),
        _persist,
      };
    }
    // .... (중략) ....

    action.register(config.key)

```



이후, persistReducer 최상단에서 봤던 getStoredState 로 부터\
복구된, resoredState 를 이용해 여러 처리를 하는것을 알 수 있습니다.

<pre class="language-typescript"><code class="lang-typescript"><strong>  getStoredState(config).then(
</strong>      restoredState => {
      // ...
</code></pre>

getStoredState 를 config 로 넘겨주지 않으면\
defaultGetStoredState 를 사용하는것을 persistReducer.ts 상단에서 확인했습니다

```typescript
const getStoredState = config.getStoredState || defaultGetStoredState
```



### ✏️ getStoredState.ts

`defaultGetStoredState` 는 `getStoredState.ts` 로 부터 import 됩니다.

`getStoredState.ts` 를 잠시 들여다 보겠습니다.

<pre class="language-typescript"><code class="lang-typescript"><strong>  getStoredState(config).then(
</strong>      restoredState => {
        if (restoredState) {
          // eslint-disable-next-line @typescript-eslint/no-unused-vars
          const migrate = config.migrate || ((s, _) => Promise.resolve(s))
          migrate(restoredState as any, version).then(
            migratedState => {
              _rehydrate(migratedState)
            },
            migrateErr => {
              if (process.env.NODE_ENV !== 'production' &#x26;&#x26; migrateErr)
                console.error('redux-persist: migration error', migrateErr)
              _rehydrate(undefined, migrateErr)
            }
          )
        }
      },
      err => {
        _rehydrate(undefined, err)
      }
    )

    return {
      ...baseReducer(restState, action),
      _persist: { version, rehydrated: false },
    }
  }
</code></pre>





```
 else if (action.type === PURGE) {
    _purge = true
    action.result(purgeStoredState(config))
    return {
      ...baseReducer(restState, action),
      _persist,
    }
  } else if (action.type === FLUSH) {
    action.result(_persistoid && _persistoid.flush())
    return {
      ...baseReducer(restState, action),
      _persist,
    }
  } else if (action.type === PAUSE) {
    _paused = true
  } else if (action.type === REHYDRATE) {
    // noop on restState if purging
    if (_purge)
      return {
        ...restState,
        _persist: { ..._persist, rehydrated: true },
      }

    // @NOTE if key does not match, will continue to default else below
    if (action.key === config.key) {
      const reducedState = baseReducer(restState, action)
      const inboundState = action.payload
      // only reconcile state if stateReconciler and inboundState are both defined
      const reconciledRest: S =
        stateReconciler !== false && inboundState !== undefined
          ? stateReconciler(inboundState, state, reducedState, config)
          : reducedState

      const newState = {
        ...reconciledRest,
        _persist: { ..._persist, rehydrated: true },
      }
      return conditionalUpdate(newState)
    }
  }

  // if we have not already handled PERSIST, straight passthrough
  if (!_persist) return baseReducer(state, action)

  // run base reducer:
  // is state modified ? return original : return updated
  const newState = baseReducer(restState, action)
  if (newState === restState) return state
  return conditionalUpdate({ ...newState, _persist })
}
}
```



```typescript
return (state: any, action: any) => {
  const { _persist, ...rest } = state || {}
  const restState: S = rest

  if (action.type === PERSIST) {
    let _sealed = false
    const _rehydrate = (payload: any, err?: Error) => {
      // dev warning if we are already sealed
      if (process.env.NODE_ENV !== 'production' && _sealed)
        console.error(
          `redux-persist: rehydrate for "${
            config.key
          }" called after timeout.`,
          payload,
          err
        )

      // only rehydrate if we are not already sealed
      if (!_sealed) {
        action.rehydrate(config.key, payload, err)
        _sealed = true
      }
    }
    timeout &&
      setTimeout(() => {
        !_sealed &&
          _rehydrate(
            undefined,
            new Error(
              `redux-persist: persist timed out for persist key "${
                config.key
              }"`
            )
          )
      }, timeout)

    // @NOTE PERSIST resumes if paused.
    _paused = false

    // @NOTE only ever create persistoid once, ensure we call it at least once, even if _persist has already been set
    if (!_persistoid) _persistoid = createPersistoid(config)

    // @NOTE PERSIST can be called multiple times, noop after the first
    if (_persist) {
      // We still need to call the base reducer because there might be nested
      // uses of persistReducer which need to be aware of the PERSIST action
      return {
        ...baseReducer(restState, action),
        _persist,
      };
    }

    if (
      typeof action.rehydrate !== 'function' ||
      typeof action.register !== 'function'
    )
      throw new Error(
        'redux-persist: either rehydrate or register is not a function on the PERSIST action. This can happen if the action is being replayed. This is an unexplored use case, please open an issue and we will figure out a resolution.'
      )

    action.register(config.key)

    getStoredState(config).then(
      restoredState => {
        if (restoredState) {
          // eslint-disable-next-line @typescript-eslint/no-unused-vars
          const migrate = config.migrate || ((s, _) => Promise.resolve(s))
          migrate(restoredState as any, version).then(
            migratedState => {
              _rehydrate(migratedState)
            },
            migrateErr => {
              if (process.env.NODE_ENV !== 'production' && migrateErr)
                console.error('redux-persist: migration error', migrateErr)
              _rehydrate(undefined, migrateErr)
            }
          )
        }
      },
      err => {
        _rehydrate(undefined, err)
      }
    )

    return {
      ...baseReducer(restState, action),
      _persist: { version, rehydrated: false },
    }
  } else if (action.type === PURGE) {
    _purge = true
    action.result(purgeStoredState(config))
    return {
      ...baseReducer(restState, action),
      _persist,
    }
  } else if (action.type === FLUSH) {
    action.result(_persistoid && _persistoid.flush())
    return {
      ...baseReducer(restState, action),
      _persist,
    }
  } else if (action.type === PAUSE) {
    _paused = true
  } else if (action.type === REHYDRATE) {
    // noop on restState if purging
    if (_purge)
      return {
        ...restState,
        _persist: { ..._persist, rehydrated: true },
      }

    // @NOTE if key does not match, will continue to default else below
    if (action.key === config.key) {
      const reducedState = baseReducer(restState, action)
      const inboundState = action.payload
      // only reconcile state if stateReconciler and inboundState are both defined
      const reconciledRest: S =
        stateReconciler !== false && inboundState !== undefined
          ? stateReconciler(inboundState, state, reducedState, config)
          : reducedState

      const newState = {
        ...reconciledRest,
        _persist: { ..._persist, rehydrated: true },
      }
      return conditionalUpdate(newState)
    }
  }

  // if we have not already handled PERSIST, straight passthrough
  if (!_persist) return baseReducer(state, action)

  // run base reducer:
  // is state modified ? return original : return updated
  const newState = baseReducer(restState, action)
  if (newState === restState) return state
  return conditionalUpdate({ ...newState, _persist })
}
}
```



