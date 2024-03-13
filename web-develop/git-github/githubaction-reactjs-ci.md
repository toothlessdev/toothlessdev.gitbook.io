# GithubAction 으로 ReactJS 프로젝트 CI 구축하기

프로젝트를 진행하며 브랜치를 나누어 코드를 작성하다가, 해당코드를 한 브랜치로 Merge 할 때,\
버그가 있는 코드를 올리거나, 코드 컨벤션에 맞지 않는 코드를 올리면,\
빌드시에 오류 발견이 늦어질 수도 있고, 개발 생산성이 떨어지거나, 배포시에 오류가 발생 할 수 있습니다.

## 📖 CI (Continuous Integration)

> 소프트웨어 공학에서 지속적 통합(continuous integration, CI)은 지속적으로 품질 관리(Quality Control)를 적용하는 프로세스를 실행하는 것이다. - 작은 단위의 작업, 빈번한 적용. 지속적인 통합은 모든 개발을 완료한 뒤에 품질 관리를 적용하는 고전적인 방법을 대체하는 방법으로서 소프트웨어의 질적 향상과 소프트웨어를 배포하는데 걸리는 시간을 줄이는데 초점이 맞추어져 있다&#x20;
>
> ([https://ko.wikipedia.org/wiki/%EC%A7%80%EC%86%8D%EC%A0%81\_%ED%86%B5%ED%95%A9](https://ko.wikipedia.org/wiki/%EC%A7%80%EC%86%8D%EC%A0%81\_%ED%86%B5%ED%95%A9))

CI 를 통해

> 1. 조기에 오류 탐지
> 2. 상시 진행 상황을 확인하면서 더 유익한 피드백 적용 가능
> 3. 팀 협업 증진&#x20;
> 4. 효과적인 시스템 통합&#x20;
> 5. 병합 및 테스트에서 동시 변경의 필요성 감소
> 6. 시스템 테스트 과정에서 오류 건수 감소
> 7. 상시 업데이트 된 시스템을 대상으로 테스트 가능

등의 장점이 있습니다.



## 📖 Typescript, Eslint, Prettier

React 프로젝트에서는

<mark style="background-color:orange;">TypeScript</mark> 를 사용해 타입 안정성을 보장하고,\
<mark style="background-color:orange;">Eslint</mark> 를 이용해 코딩 컨벤션에 위배되는 코드나 안티 패턴을 검출하고,\
<mark style="background-color:orange;">Prettier</mark> 를 통해 규칙에 맞게 코드를 포맷팅 합니다.

Github Action 과 TypeScript Compile, Eslint, Prettier 를 이용해 CI 파이프라인을 구축할 수 있습니다.

### ✏️ eslint

`.eslintrc.cjs` 파일에 eslint 관련 규칙을 설정합니다.

기본적으로 package.json 의 scripts 에 lint 가 정의되어있습니다.\
`npm run lint` 를 통해 EsLint 로 코드 컨벤션에 위배되는 코드나 안티 패턴을 검출 할 수 있습니다.

### ✏️ typescript

`npm run build` 시에 타입스크립트 컴파일 이후 빌드를 실행합니다.

CI 환경에서 타입스크립트 컴파일시 오류를 검증하기 위해 `compile` 스크립트를 만들어 줍니다.\
이제 `npm run compile` 을 통해 타입스크립트 컴파일을 할 수 있습니다.

```json
{
    // ...
    "scripts" : {
        "compile" : "tsc",
        // ...
    }
}
```

### ✏️ prettier

우선 prettier 코드 포매터를 개발의존성 패키지로 설치합니다

```sh
npm install prettier --save-dev
```

prettier 코드 포매터에 해당하는 `prettier` 스크립트도 만들어 줍니다.\
이제 `npm run prettier` 을 통해 코드 포매팅을 할 수 있습니다.

```json
{
    // ...
    "scripts" : {
        "prettier" : "prettier --write **/*.{ts,tsx}",
        // ...
    }
}
```





## 📖 CI Workflow

Github Action 에서는 Event 에 의해 Workflow 라는 자동화된 프로세스 단위가 실행됍니다.\
Workflow 파일은 `.yaml` `.yml` 로 작성되고, Github Repository 의 `.github/workflows` 디렉토리에 저장됍니다.

### ✏️ Event

Workflow 가 실행될 조건을 정의합니다.\
develop 브랜치에 PR 이 요청될 경우 트리거 되도록 설정하였습니다.

```yaml
on:
    pull_request:
        branches: ["main"]
```



### ✏️ Job

Workflow 는 여러 Job 으로 구성되고 하나의 Job 은 하나의 Runner 인스턴스에서 실행됍니다.

공통적으로 `ubuntu-latest` 환경에서 실행되도록 설정하였고,\
`actions/setup-node` 와 `actions/checkout` 을 이용해 NodeJS 를 설치하고 Code 를 Checkout 합니다.

`npm ci` 를 통해 package-lock.json 의 의존성 패키지들을 설치하고,\
package.json 에 정의해둔 스크립트를 실행합니다

각 job 들은 `needs` 를 통해 이전 Job 과의 종속 관계를 가지도록 설정해줍니다.



### ✏️ CI.yml

```yaml
name: CI

on:
    pull_request:
        branches: ["develop"]

jobs:
    compile:
        runs-on: ubuntu-latest

        steps:
            - name: Install NodeJS
              uses: actions/setup-node@v2
              with:
                  node-version: 20

            - name: Code Checkout
              uses: actions/checkout@v3

            - name: Install Dependencies
              run: npm ci

            - name: TypeScript Compile
              run: npm run compile

    lint:
        runs-on: ubuntu-latest
        needs: compile

        steps:
            - name: Install NodeJS
              uses: actions/setup-node@v2
              with:
                  node-version: 20

            - name: Code Checkout
              uses: actions/checkout@v3

            - name: Install Dependencies
              run: npm ci

            - name: ES Lint
              run: npm run lint

    prettier:
        runs-on: ubuntu-latest
        needs: lint

        steps:
            - name: Install NodeJS
              uses: actions/setup-node@v2
              with:
                  node-version: 20

            - name: Code Checkout
              uses: actions/checkout@v3

            - name: Install Dependencies
              run: npm ci

            - name: Code Format
              run: npm run prettier
```



<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

develop 브랜치로 Pull Request 요청을 날릴 경우, Workflow 가 잘 실행됌을 알 수 있습니다.

##

## 🔗 참고자료

[https://aws.amazon.com/ko/devops/continuous-integration/](https://aws.amazon.com/ko/devops/continuous-integration/)\
[https://www.daleseo.com/github-actions-checkout/](https://www.daleseo.com/github-actions-checkout/)\
[https://www.daleseo.com/github-actions-setup-node/](https://www.daleseo.com/github-actions-setup-node/)\
[https://www.daleseo.com/github-actions-jobs/](https://www.daleseo.com/github-actions-jobs/)\
[https://www.daleseo.com/github-actions-triggers/](https://www.daleseo.com/github-actions-triggers/)
