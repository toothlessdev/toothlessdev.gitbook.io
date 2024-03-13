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

REST API 에서는 자원에 대한 행위를 Method 로 표현하고\
Method 에는 `GET` `POST` `PUT` `PATCH` `DELETE` 가 있습니다.

[rest-api-rest.md](../web/rest-api-rest.md "mention")



### ✏️ @Param 으로 동적 파라미터 받아오기

HTTP Method 데코레이터로 `:` 과 함께 문자열을 넘겨주는경우, 동적 파라미터에 대한 요청을 처리할 수 있습니다.\
Controller 메서드의 파라미터로 `@Param()` 을 전달하는경우, 요청시 동적 파라미터를 받아올 수 있습니다.

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller("posts")
export class PostController {
    @Get(":id")
    public readPostById(@Param("id") id : string) {
        return //...
    }
}
```

`/posts/1` 로 요청을 보내는 경우, id 로 1 을 받을 수 있습니다.



### ✏️ @Body 로 요청 Body 받아오기

Controller 메서드의 파라미터로 `@Body()` 를 전달하는경우, 요청시 Body 값을 받아올 수 있습니다.

```typescript
import { Controller, Post } from '@nestjs/common';

@Controller("posts")
export class PostController {
    @Post()
    public createPost(@Body("title") title:string, @Body("content") content:string) {
        return //...
    }
}
```

DTO (Data Transfer Object) 를 통해 요청 Body 값을 전달 할 수도 있습니다.

```typescript
export class CreatePostDto {
    title:string;
    content:string;
}

@Controller("posts")
export class PostController {
    @Post()
    public createPost(@Body() createPostDto:CreatePostDto) {
        return //...
    }
}
```



### ✏️ 요청, 응답 객체 받아오기

`@Request()` 또는 `@Req()` 를 통해 요청 객체를 받아올 수 있습니다.\
`@Response()` 또는 `@Res()` 를 통해 응답 객체를 받아올 수 있습니다.



## 🔗 참고자료

[https://medium.com/@mohitu531/nestjs-7c0eb5655bde](https://medium.com/@mohitu531/nestjs-7c0eb5655bde)
