# RxJS 와 반응형 프로그래밍

rxjs 는 뭐하는라이브러리인가 ?

함수형 프로그래밍 : 변수 사용을 지양하고, 순수함수를 활용해 프로그래밍하는것

예를들어 정수 배열에서 짝수만 걸러낸 후, 최초 5개의 배열 엘리먼트를 선택해 제곱한 결과값을 합하는 코드를 짜야된다고 했을때, 보통

```javascript
const arr : number[] = [
    1,2,3,4,5,6,7,8,9,10
];

let count : number = 0;
let sum : number = 0;

for(let index = 0; index < arr.length - 1 && count < 5 ; i++) {
    if(arr[index] % 2 === 0) {
        sum += Math.pow(arr[index], 2)
        count++;
    }
}
```

다음과 같이 작성할건데,

count sum 프로그래밍 실행중 값이 바뀔 수 있는 벼눗를 사용



소프트웨어가 커지고 버ㅗㄱ잡해지면, 코드에 노출된 변수들은 해당 소프트웨어가 돌아가는 맥락, 상태값으로 작용하는데,

코딩시 변수에 잘못된 값을 할당 할 수도 있고,,, 멀티스레딩환경에서는 락을 걸어놓지 않으면 오류가 발생할 수 있음

따라서 함수형 프로그래밍에서는 변수를 쓰지 않고, 순수함수를 활용해서 데이터를 처리한다

함수 내부에서는 변수들이 사용되지만, 캡슐화 추상화되어 노출되지 않기 때문에 오류 방지 가능. + 스레드 동시접근 / 교착문제 ... 예방가능

함수형 프로그래밍에서는 과정을 선언하는 형태,, 일반적으로 함수형 프로그래밍은 선언형 프로그래밍의 특성도 갖음



rxjs 구성요소

1. observable : 관찰될 수 있는것, 관찰되는 대상. 연속적으로 발행된 갑 = 스트림
2. pipe : &#x20;



스르팀은 파이프를 거치면서 operator 를 거침 (순수함수)

옵저ㅂㅓ는 파이프에서 나오는 값을 기다리고 받아 최종작업 실행(subscribe)



왜쓰냐 ?

평면적 배열 뿐만 아니라, 시간흐름, 사용자 동작, 네트워크 요청 ,.. 을 모두 스트림으로 만들어

파이프라인에 흘려보내 처리함



반으영ㅇ = 함수형 = 선언형..



