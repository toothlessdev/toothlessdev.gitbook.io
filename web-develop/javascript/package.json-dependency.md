---
description: >-
  npm 으로 디자인시스템을 만들고 배포하면서, peerDependency 와 devDependency 와 같은 개념들을 공부하며 정리한
  포스트입니다
---

# package.json 과 dependency

## 📖 Dependency 란?

> **dependency** : 의존 / 의존성

프로젝트를 하다보면 dependency 라는 단어를 많이 들어보셨을 것입니다.\
dependency 를 사전에서 찾아보면 '의존' , '의존성' 이라는 뜻으로 나와 있습니다.

이와 유사하게, 프로그래밍에서 'dependency (의존성)' 은 <mark style="background-color:orange;">**소프트웨어 구성요소가 다른 구성요소나 라이브러리에 의존하는 관계**</mark>를 의미합니다.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

위 그림에서 Project 에 `Package A` , `Package B` , `Package C` 를 설치를 해 개발을 한다고 했을때,\
Project 는 `Package A` , `Package B` , `Pacakge C` 에 대한 의존성을 가지고 있다 라고 말합니다



### ✏️ NodeJS 와 NPM

NodeJS 에서는 기본적으로 npm 이라고 하는 Node Package Manager 를 사용해 패키지들을 관리합니다\
(이 외에도 yarn, pnpm 과 같은 패키지 매니저들이 있습니다)



## 📖 dependencies



## 📖 peerDependency

Peer Dependency 는 JS 와 NodeJS 환경에서 사용되는 개념으로, \
특정 패키지가 정상적으로 작동하기 위해 함께 사용되는 다른 패키지의 특정 버전을 요구하는 것을 명시합니다

이를 통해

1. 호환성 보장 : 여러 패키지가 동일 라이브러리의 호환 가능 버전을 사용함으로써 충돌을 방지하고,
2. 중복 방지 : 동일한 라이브러리의 여러 버전을 설치해 발생할 수 있는 문제를 피할 수 있습니다.

### ✏️ peer Dependency 예시

말로만 들어서는 어떤 개념인지 헷갈려서 예시를 통해 살펴보겠습니다

