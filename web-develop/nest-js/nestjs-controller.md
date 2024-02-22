# NestJS Controller

## 📖 Controller란?

Controller 는 들어오는 HTTP 요청을 받고, 처리하여 HTTP 응답을 보내주는 역할을 합니다.\
Controller 는 엔드포인트에 대한 라우팅, Service 레이어와 상호작용을 합니다.

### ✏️ nest cli 로 controller 생성하기

`nest g controller [NAME]` 또는 `nest generate controller [NAME]` 을 통해 `[NAME]` 에 해당하는 controller 를 생성할 수 있습니다.

(또는 `nest g resource [NAME] --no-spec` 을 이용해 리소스 전체를 생성할 수도 있습니다)

기본적으로 `.spec` 의 테스트 파일이 함께 생성되는데 `--no-spec` 옵션을 추가하면 테스트 파일 없이 생성할 수 있습니다.

## 📖 NestJS Controller

### ✏️ 요청 Path 매핑하기

Controller 에서 해당 엔트포인트를 처리하는 함수와 매핑하기 위해서 두가지 방법을 사용 할 수 있습니다

1. `@controller` 의 파라미터로 엔드포인트를 넘겨줍니다.

모든 라우트의 엔드포인트에 파라미터로 넘긴 문자열을 접두어로 붙이는 역할을 합니다

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller("post")
export class PostController {
    @Get()
    public readAllPosts() {
        return //...
    }
}
```

2. HTTP Method 데코레이터에 지정할 수 있습니다

해당 컨트롤러 메서드에만 적용하는 역할을 합니다.

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller()
export class PostController {
    @Get("post")
    public readAllPosts() {
        return //...
    }
}
```

둘다 `/post` 로 `GET` 요청을 날린경우에 동작합니다



### ✏️ HTTP Method

REST API 에는&#x20;

## 📖 nest cli 을 통ㅎ

## 🔗 참고자료

[https://medium.com/@mohitu531/nestjs-7c0eb5655bde](https://medium.com/@mohitu531/nestjs-7c0eb5655bde)
