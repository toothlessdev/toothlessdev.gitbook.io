# tsyringe로 리액트에서 싱글톤 패턴을 사용해 의존성 주입하기

## ❎ 사건의 발단

1. MVC 아키텍쳐의 한계를 느껴서 Flux 아키텍쳐가 나왔다고? MVC 아키텍쳐가 뭐지?
2. Model View Controller 로 나누는거구나! React 에서는 계층을 나눠서 사용못하나?
3. 아 리액트는 View 만 담당하는 Component 기반의 라이브러리구나
4. NestJS 에서는 Controller , Service, Repository ... 로 단위를 나눠서 관리하네
5. Dependency Injection 을 이용해서 싱글톤 패턴을 구현했구나
6. 리액트에서는 비즈니스 로직을 다루는데 있어서 정해진 패턴이 없네..
7. 그래서 그런가 커스텀훅으로 데이터페칭을 재사용해도 이벤트핸들러에서는 재사용하지를 못하고 복잡해진다..

그렇다면??

> API 호출을 추상화한 클래스를 만들고,
>
> 사용되는 로직을 추상화한 서비스 클래스를 만들고
>
> 서비스 클래스에서 API 호출을 추상화한 인스턴스를 생성자로 주입받아서 사용할수 있지 않을까??

에서 시작되었다



구글링을 해보니까 리액트에서 이렇게 의존성 주입으로 서비스 레이어를 나누는 코드는 없는것 같다\
그래서 그런지 정말 이렇게 해도되나 라는 생각이 계속 들었다

어쩌면 이게 정말로 잘못된 방법일 수도 있겠지만\
우선 현재 진행하고 있는 프로젝트에서 데이터 페칭이 여기 저기서 파편화 되는 문제가 있기도했고

> 그냥 refactor 브랜치 하나 파서 뭐 잘못되면 아님말고 (브랜치 갖다 버리기) 를 시전하지 뭐

라는 생각이 들어 도전해보기로 했다!



## **✅ tsyringe**

npm 라이브러리중에 타입스크립트를 이용해서 의존성주입과 싱글톤 패턴을 구현하는데 도움을 주는 tsyringe 라는 라이브러리를 발견했고 사용하기로 했다\
보니까 NestJS 의 `@Injectable() @InjectRepository()` 와 비슷하게 데코레이터를 이용해서 구현된 라이브러리 인듯

&#x20;**0. 혹시 모를 상황에 대비해 브랜치를 판다**

```bash
git branch refactor/tsyringe
```

1. **먼저 tsyringe 라이브러리를 설치해준다**

```bash
npm install tsyringe
```

2. **TypeScript 에서 데코레이터와 reflect-metadata 를 위한 설정을 한다**

```bash
npm install reflect-metadata
```

```json
// tsconfig.json
{
    "compilerOptions" : {
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}
```

3. **entry point 의 최상단에 reflect-metadata 를 추가한다**

Vite 로 만든 리액트 애플리케이션이기 때문에 `main.tsx` 에 추가했다

```tsx
// main.tsx
import "reflect-metadata";
```

4. **HTTP 요청을 추상화한 Api 클래스를 만들어준다**

원래 있었던 함수들을 Api 라는 클래스의 메서드로 묶었다

<pre class="language-typescript"><code class="lang-typescript"><strong>// src/api/api.ts
</strong><strong>
</strong><strong>export default class Api {
</strong>    public async Get&#x3C;ResponseBody>(url: string, token?: string): Promise&#x3C;ResponseBody> {
        const headers: HeadersInit = token ? { Authorization: `Bearer ${token}` } : {};
        const repsonse = await fetch(API_BASE_URL + url, {
            method: "GET",
            headers: headers,
        });
        if (!repsonse.ok) throw new Error(`GET Request Failed ${url}`);
        return repsonse.json() as Promise&#x3C;ResponseBody>;
    }

    public async Post&#x3C;RequestBody, ResponseBody>(url: string, body: RequestBody, token?: string): Promise&#x3C;ResponseBody> {
        const headers: HeadersInit = token
            ? { Authorization: `Bearer ${token}`, "Content-Type": "application/json" }
            : { "Content-Type": "application/json" };
        const response = await fetch(API_BASE_URL + url, {
            method: "POST",
            headers: headers,
            body: JSON.stringify(body),
        });
        if (!response.ok) throw new Error(`POST Request Failed ${url}`);
        return response.json() as Promise&#x3C;ResponseBody>;
    }
    // ...
}
</code></pre>

5. **Api 클래스가 런타임에 주입될 수 있도록 컨테이너에 등록해준다**

```tsx
import "reflect-metadata";

import ReactDOM from "react-dom/client";
import App from "./App.tsx";
import { BrowserRouter } from "react-router-dom";

import { container } from "tsyringe";
import Api from "./api/api.ts";

container.register("Api", { useClass: Api });

//...
```

6. **실제 요청하는 API 호출을 추상화한 클래스를 만들어준다**

지금 하던 프로젝트에서 강의 목록들을 조회하는 곳 중에 테스트용으로 해보기로했다\
(ReponseType 인터페이스도 디렉토리를 나눴다)

<pre class="language-typescript"><code class="lang-typescript"><strong>// src/api/service/lecture.service.ts
</strong><strong>
</strong><strong>import { inject, injectable } from "tsyringe";
</strong>import Api from "../api";

import { IGetAllLectureResponse } from "../interface/lecture.interface";

@injectable()
export default class LectureService {
    constructor(
        @inject("Api")
        private readonly api: Api,
    ) {}

    public async getAllLectures() {
        return this.api.Get&#x3C;IGetAllLectureResponse>("/lectures");
    }
}
</code></pre>

`@Injectable()` 을 이용해 LectureService 클래스가 주입받을 수 있게 해주었고,\
`@Inject()` 를 이용해 생성자로 Api 클래스를 주입받도록 했다.



7. **되나?**

간단하게 원래 커스텀 훅 `useRequest` 로 호출하던 페이지에서 사용해보았다

```tsx
export default function LecturePage() {
    // ...
    const [status, setStatus] = useState<STATUS>(STATUS.IDLE);
    const [data, setData] = useState<IGetAllLectureResponse | null>(null);

    useEffect(() => {
        (async () => {
            setStatus(STATUS.PENDING);
            const lectureService = container.resolve(LectureService);
            const fetchedData = await lectureService.getAllLectures<IGetAllLectureResponse>();
            setData(fetchedData);
            setStatus(STATUS.SUCCESS);
        })();
    }, []);
    
    // ...
}
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

잘된다!



8. **추후 해볼것**

service 를 인자로 받아 사용하는 useAPI 훅을 만들면 좋을것 같다.\




## ✳️ 느낀점

장점은 파편화된 데이터페칭 코드들을 service 에서 관리할 수 있어 좋은것 같다



NestJS 의 IOC 컨테이너에서 `@Injectable()` , `@InjectRepository()` 와 같이 클래스를 생성하고 데코레이터를 이용해 의존성을 주입하는것에서 생각이 떠올라

비슷한 데코레이터를 사용하는 tsyringe 라이브러리를 사용했는데...

생각해보니 자바스크립트에서는 객체 리터럴을 이용해 객체를 생성할 수 있는데\
그냥 객체리터럴을 이용해 export 하고, Service 에서 api 객체 리터럴을 주입해도 되지 않았을까 하는생각이 든다



각각 어떤 장단점이 있는지 고민을 해야할 필요가 있을것 같고.. 아직 자바스크립트에 대한 기본기가 많이 부족하단 생각이 들고 얼른 딥다이브책으로 기본기를 쌓아야겠다\
(아직 자바스크립트에서 객체 리터럴, 클래스 인스턴스 ... 에 대한 차이도 모를뿐더러 TypeScript 데코레이터에 대해서는 더더욱 모른다)





## 🔗 참고자료

\[Typescript] DI using tsyringe\
[https://velog.io/@milkcoke/Typescript-DI-using-tsyringe#singleton](https://velog.io/@milkcoke/Typescript-DI-using-tsyringe#singleton)

Singleton Pattern - Patterns.dev\
[https://www.patterns.dev/vanilla/singleton-pattern](https://www.patterns.dev/vanilla/singleton-pattern)

NPMJS - Tsyringe\
[https://www.npmjs.com/package/tsyringe](https://www.npmjs.com/package/tsyringe)

