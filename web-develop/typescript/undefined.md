# 기초 타입 이론

{% embed url="https://www.youtube.com/watch?v=xesy1i67OWI" %}



## 📖 Type 이란?

어떤 Symbol (변수명, 상수명 ...) 에 엮인,\
메모리 공간에 존재할 수 있는,\
값의 집합과\
그 값들이 가질 수 있는 성질

> **예) 3.141592 : number**
>
> 값 3.141592 는 타입 number 에 속한다



## 📖 SubType 이란?

> **예) `{ x : number; y? : string } <: { x : number }`**&#x20;
>
> 타입 `{ x : number; y? : string } 은 타입 { x : number }` 의 서브/슈퍼 타입 이다

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



## 📖 타입은 부분 순서집합 (Partially Ordered Set) 이다!

서로 다른 두 타입은 4가지 경우의 관계를 가질 수 있습니다!

> 1. **`a :> b` a 가 b 의 SuperType**
> 2. **`a <: b` a 가 b 의 SubType**
> 3. **`a :=: b` 서로 SubType**
> 4. **`a :!=: b` 무관계**

예를 들어

> 1. `number :> 42`
> 2. `symbol | string <: number | symbol | string`
> 3. `{ x? : number } :=: { x : number | undefined }`
> 4. `number :!=: { x: string }`

와 같은 경우를 들 수 있습니다!



## 📖 타입과 값의 대입
