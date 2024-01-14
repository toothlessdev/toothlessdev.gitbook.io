# \[NestJS] DI / IOC 의존성 주입과 제어의 역전

Dependency Injection (의존성 주입) 과 Inversion Of Control (제어의 역전) 에 대해 알아보기 전에

NestJS 의 요청 라이프사이클과 Controller, Service, Repository 아키텍쳐에 대해 간단히 알아 보고난 뒤에

DI 와 IOC 가 어떻게 쓰이는지에 대해 알아보자.

## 📖 NestJS 의 요청라이프사이클

### ✏️ 요청에서부터 응답까지

NestJS 에서 클라이언트로부터의 요청을 받아서 응답을 보내는데까지는 일련의 과정을 거치게 되는데 이것을 _**요청 라이프 사이클**_ 라고 부른다.

![](https://velog.velcdn.com/images/toothless042/post/c044c009-1ed5-46c6-8e23-8202d8491ee3/image.png)

### ✏️ Controller, Service, Repository 아키텍쳐

Nest-Cli 에 `nest g resource --no-spec` 을 이용해서 REST API 에 해당하는 리소스를 만들면 자동으로 module, controller, service 를 만들어준다.

간단하게 Controller, Service, Repository 아키텍쳐에 대해 살펴보자면

> **Controller** 에서는 요청에 해당하는 엔트포인트와 HTTP Method 가 정의되어있고, 클라이언트의 요청을 받고 응답을 반환하는 역할을 한다.

> **Service** 는 Controller로부터 받은 요청을 처리하고, 데이터 처리, 검증, 변환 및 저장과 같은 비즈니스 관련 로직을 수행한다.

> **Repository**는 데이터베이스와의 상호작용을 관리하는 역할을 한다.

## 📖 Dependency Injection 의존성 주입

### ✏️ Controller 에서 Service 인스턴스의 사용?

실제 NestJS 의 Controller 를 살펴보면, 이상한 점이 하나 있다. Service 에 대한 인스턴스가 없고 단순히 생성자로부터 Service 를 받아오는 코드만 작성되어있는데도

```ts
@Controller()
export class MyController {
  constructor(private readonly myService: MyService) {}
}
```

우리는 Controller 에서 Service 를 사용할 수 있다 이것을 가능하게 해주는 것이 **Dependency Injection 의존성 주입** 이다.

### ✏️ 일반적인 인스턴스화

```ts
export class A {
  b = new B();
}

export class B {
  
}
```

클래스 A, B 가 있고, 클래스 A 에서 클래스 B 를 사용하는 경우 일반적으로 클래스 A의 내부의 필드에 B 인스턴스를 생성해서 사용한다.

### ✏️ Dependency Injection 의존성 주입

위의 예제에서 클래스 A는 클래스 B의 인스턴스를 필요로 하기 때문에

> 클래스 A 는 클래스 B 에 의존하고 있다. 의존성을 가지고 있다

라고 한다.

클래스가 서로 의존성을 가지고 있는경우, 의존 대상이 변하게되면 의존하는 클래스에 영향을 끼치게 된다.

그렇다면 의존성 주입을 적용한 경우 어떻게 동작하는지 알아보자.

```ts
export class A {
  private b;
  constructor(b: B) { this.b = b; }
  // 또는 constructor(private readonly b: B) {}
}

export class B {
}
```

위의 코드를 보면 클래스 A 에는 따로 클래스 B 에 대한 인스턴스를 필드로 가지고 있지 않다.

> 대신, 클래스 A 가 생성될때 **외부**에서 B의 인스턴스가 생성이되고, 해당 B 의 인스턴스를 클래스 A 의 생성자로 넣어준다 **(의존성주입, DI)**

## 📖 Inversion Of Control 제어의 역전

### ✏️ Provider

> Providers are a fundamental concept in Nest. Many of the basic Nest classes may be treated as a provider – services, repositories, factories, helpers, and so on. The main idea of a provider is that _\*\*it can be injected as a dependency; \*\*_ this means objects can create various relationships with each other, and the function of "wiring up" these objects can largely be delegated to the Nest runtime system. [NestJS 공식문서 - Providers](https://docs.nestjs.com/providers)

NestJS 에서 다른 클래스에 의존성으로 주입될 수 있는 것들을 **Provider** 라고 한다. 그리고 이런 의존성 주입은 NestJS 의 런타임 시스템에 위임된다

### ✏️ Inversion Of Control 제어의 역전

NestJS 에서 다른 클래스들이 의존하고 있는 Provider 들의 인스턴스의 생성 & 삭제 & 주입을 프레임워크가 담당하는것을 **Inversion Of Control (제어의 역전)** 이라고 한다.

![](https://velog.velcdn.com/images/toothless042/post/7a2eeb1e-630b-4814-a8d1-b6f9904b6144/image.png)

즉, Dependency Injection 을 위해 Provider 클래스들을 NestJS의 **IOC Container** 에 등록하고, 의존성을 주입하고, 클래스 인스턴스의 생명주기를 관리한다.

DI 와 IOC 를 통해개발자는 의존성이 있는 클래스의 생성 & 폐기를 신경쓸 필요가 없다는 장점이 있다

## 📖 NestJS 에서의 DI / IOC

NestJS 의 Provider 중, Service 가 Controller 로 주입되는 과정을 예시를 통해 살펴보면서 이해해보자

생성된 Controller 를 보면, Service Provider 를 사용하기 위해 NestJS IOC Container 로 부터 생성자로 주입받을 수 있도록 작성되어있다.

```ts
import { Controller } from '@nestjs/common';
import { MyService } from './my.service';

@Controller('my')
export class MyController {
  constructor(private readonly myService: MyService) {}
}
```

### ✏️ @Injectable() 어노테이션

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class MyService {}
```

먼저 Service 를 NestJS IOC Container 가 주입 하고 관리할 수 있도록 설정하기위해 `@Injectable()` 데코레이터를 사용한다.

### ✏️ @Module() 어노테이션

`@Injectable()` 로 주입가능하도록 설정 후, Module 에서 클래스를 Provider 로 등록해야한다.

```ts
import { Module } from '@nestjs/common';
import { MyService } from './my.service';
import { MyController } from './my.controller';

@Module({
  imports: [],
  exports: [],
  controllers: [MyController],
  providers: [MyService],
})
export class MyModule {}
```

.module.ts 파일의 `@Module()` 어노테이션에서는 여러 속성들을 받을 수 있다. [NestJS 공식문서 - Modules](https://docs.nestjs.com/modules)

> **Providers** : the providers that will be instantiated by the Nest injector and that may be shared at least across this module (현재 모듈 내부에서만 의존성 주입을 할 수 있도록 인스턴스화한다)

> **Controllers** : the set of controllers defined in this module which have to be instantiated (현재 모듈에서 인스턴스화 해야하는 컨트롤러 클래스)

> **imports** : the list of imported modules that export the providers which are required in this module (현재 모듈에서 다른 모듈에서 exports 로 내보낸 Provider 를 주입받을때 사용)

> **exports** : the subset of providers that are provided by this module and should be available in other modules which import this module. You can use either the provider itself or just its token (provide value) (현재 모듈에서 다른 모듈로 Provider 를 공유할 때 사용)

### ✏️ NestJS 에서의 DI / IOC

![](https://velog.velcdn.com/images/toothless042/post/1ec7d4ca-ec2b-494b-9dc7-730b00460d8e/image.png)

정리하자면, NestJS 에서 의존성 주입을 위해서는 해당 클래스를 `@Injectable()` 어노테이션을 사용해 IOC Container 가 관리할 수 있도록 설정하고,

`@Module()` 의 `providers:[]` 에 해당 클래스를 Provider 로써 등록 해주면 된다

이런 Provider 들의 인스턴스 생성, 삭제, 주입은 NestJS IOC Container (프레임워크) 에 의해 관리되는데 이를 Inversion Of Control 이라고 한다

## 📎 참고자료

[NestJS 공식문서 - Providers](https://docs.nestjs.com/providers)&#x20;

[NestJS 공식문서 - Modules](https://docs.nestjs.com/modules)
