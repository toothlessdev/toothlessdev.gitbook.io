---
icon: circle-exclamation-check
---

# The error below may be caused by using the wrong test env... Consider using the "jsdom"

## ❎ 에러

> **The error below may be caused by using the wrong test env... Consider using the "jsdom"**

리액트 훅을 테스트 하기 위해 `jest` 와 `@testing-library/react` 를 사용하던 중 다음과 같은 오류가 발생하였습니다.



## ✅ 해결

{% embed url="https://jestjs.io/docs/configuration#testenvironment-string" %}

Jest 테스트 러너는 가상의 환경에서 테스팅을 진행하는데,\
이 환경은 `jest.config.ts` 의 `testEnvironment` 프로퍼티 값으로 설정 가능합니다.

Jest의 기본 환경은 Node.js 환경 (`node`) 이기 때문에, 리액트를 사용해 웹 앱을 구축하는 경우 대신 `jsdom` 을 통해 브라우저와 유사한 환경을 사용할 수 있습니다.

```typescript
import type { Config } from "jest";

const config: Config = {
    // The test environment that will be used for testing
    testEnvironment: "jsdom",
}
```



이후 다시 테스트를 실행하면&#x20;

> **Error: Test environment jest-environment-jsdom cannot be found**

라는 에러가 발생하는데&#x20;

```bash
npm install -D jest-environment-jsdom
```

를 통해 테스트 환경을 devDependency 로 설정해주면 정상적으로 테스팅이 진행됩니다

