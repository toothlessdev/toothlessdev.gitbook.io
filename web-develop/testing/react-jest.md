# React, Jest 테스팅 환경설정

## 📖 Typescript, React, Jest 테스팅 환경 설정

{% embed url="https://jestjs.io/docs/getting-started" %}

### ✏️ Jest Configuration File 설정 및 jest 설치

```bash
npm init jest@latest
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

명령어로 기본적인 `jest.config.ts` 파일을 생성하고,

```bash
npm install -D jest
npm install -D @types/jest
```

명령어를 사용해 `jest` 와 `@types/jest` 패키지를 설치해 주고,\
간단한 테스트 코드를 작성하고 테스트 스크립트를 실행하면,

> **Error: Jest: 'ts-node' is required for the TypeScript configuration files.**

오류가 발생합니다

###

### ✏️ ts-node 설치

`ts-node` 는 TS 파일을 JS 로 컴파일 하여 즉시 실행해주는 패키지 입니다.\
NodeJS 환경에서 TS 파일을 별도의 컴파일 과정없이 즉시 실행할 수 있습니다.

`TypeScript` 와 `Jest` 를 사용해 테스팅 환경을 구축할때,\
`jest.config.ts` 파일을 작성하거나, TS 로 작성된 테스트 파일을 실행하려면, Jest 는 TS 를 이해하고 실행할 수 있어야 합니다.

`ts-node` 는 `Jest` 가 TS 로 작성된 설정 파일을 읽고, JS 로 변환해 실행 할 수 있도록 도와줍니다

```bash
npm install -D ts-node
```

###

### ✏️ Cannot find name 'test' ... Cannot find name 'describe'

> **Cannot find name 'test'. Do you need to install type definitions for a test runner? Try `npm i --save-dev @types/jest`**

`@types/jest` 를 설치해도, TS Compiler 가 해당 타입을 인식하지 못하는경우,\
`tsconfig` 의 `compilerOptions` 에 문제가 있을 확률이 높습니다

[https://www.typescriptlang.org/tsconfig/#typeRoots](https://www.typescriptlang.org/tsconfig/#typeRoots)\
[https://www.typescriptlang.org/tsconfig/#types](https://www.typescriptlang.org/tsconfig/#types)

`tsconfig` `compilerOptions` 의 `types` 와 `typeRoots` 프로퍼티가 존재한다면,\
해당 프로퍼티에 `"jest"` 가 포함되어있는지 확인해야 합니다

> ❗️ `types` , `typeRoots` 프로퍼티가 존재하지 않는 경우, 기본적으로 모든 "`@types`" 패키지가 컴파일에 포함되지만,
>
> 1. &#x20;`types` 프로퍼티가 존재하는 경우, 해당 배열에 존재하는 패키지만 포함되고,
> 2. `typeRoots` 프로퍼티가 존재하는 경우, 해당 하위 패키지만 포함됩니다



