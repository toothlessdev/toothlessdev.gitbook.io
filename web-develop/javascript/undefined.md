# 변수와 데이터 타입

> 이 포스트는 모던 자바스크립트 Deep Dive 를 공부하면서 정리한 글입니다



## 📖 변수와 상수

변수는 하나의 값을 저장하기 위해 확보한 메모리 공간 또는 메모리 공간을 식별하기 위해 붙인 이름을 말합니다.

프로그래밍에서 값을 저장하고 참조하는, 값의 위치를 가리키는 이름입니다.

변수는 컴파일러 또는 인터프리터에 의해 메모리 공간의 주소로 치환되어 실행되는데, 덕분에 개발자는 변수를 통해 안전하게 값에 접근할 수 있습니다



### ✏️ 변수의 선언

변수를 선언한다는것은, 값을 저장하기 위한 메모리 공간을 확보하고, 식별자와 메모리 공간을 연결하는 과정입니다.

자바스크립트에서 변수는 다음과 같이 선언할 수 있습니다.

```javascript
var myVariable;
let myVariable;
const myVariable;
```

다른 언어에서 지역변수를 선언하고 값을 읽으면 쓰레기값이 나오는 반면,

자바스크립트에서는 자바스크립트 엔진에 의해 식별자가 가리키는 실제 메모리 공간에는 undefined 가 할당됍니다

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

자바스크립트 엔진은

> 1. 선언단계 : 변수 이름 (식별자) 을 실행 컨텍스트 (Execution Context) 에 등록해, 변수의 존재를 알리고
> 2. 초기화 단계 : 값을 저장하기 위한 메모리 공간을 확보한 뒤, undefined 를 할당해 초기화합니다



### ✏️ 호이스팅

```javascript
console.log(myVariable);
var myVariable; // 또는 let myVariable 또는 const myVariable
```

변수를 선언하기 전에, 변수에 접근하면 ReferenceError (참조 에러) 가 발생할 것 같지만, 실제로는 undefined 가 출력됍니다.

자바스크립트에서 소스코드는 실행하기전 (런타임 이전) <mark style="background-color:orange;">**평가과정**</mark> (실행 컨텍스트를 생성하고, 변수 / 함수 등의 선언문을 실행해 생성된 식별자를 실행 컨텍스트의 스코프에 등록하는 과정) 을 거치기 때문에,

변수 선언이 소스코드의 어디에 위치하던 상관없이 먼저 실행됍니다.

이렇게 변수 선언문이 코드의 맨 앞으로 끌어 올려 진 것 처럼 동작하는 것을 <mark style="background-color:orange;">**호이스팅**</mark> 이라고 합니다.\
변수 뿐만 아니라, 자바스크립트 에서 선언하는 <mark style="background-color:orange;">**모든 식별자는 호이스팅**</mark> 됍니다.



### ✏️ 값의 할당

```javascript
var myVariable = "값";
```

변수에 값을 할당 할 때는 대입연산자(=) 를 사용합니다.

변수 선언은 소스코드 실행 전, 런타임 이전 평가 과정에서 실행되지만,\
값의 할당은 소스코드가 순차적으로 실행되는 시점인 런타임에 실행됍니다.

따라서, 변수의 선언과 값의 할당을 한번에 하더라도, 분리되어 실행됍니다.



### ✏️ 값의 재할당

```javascript
myVariable = "재할당";
```

변수에 값을 재할당하게 되면, 해당 식별자의 값은 재할당한 값으로 변경됍니다.

이때, **기존에 있던 메모리에 값이 변경되는 것이 아니라**,\
<mark style="background-color:orange;">**새로운 메모리 공간을 확보하고 재할당한 값을 저장합니다!**</mark>

> 함수형 프로그래밍 언어는 변수값 변경을 금지합니다
>
> 변수의 값을 변경하는 대신, 새로운 메모리 공간을 할당하고, 변경된 값을 저장합니다

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

기존에 있던 메모리 공간은 자바스크립트의 <mark style="background-color:orange;">**가비지콜렉터**</mark>에 의해 할당이 해제됍니다.



### ✏️ 상수

var, let 으로 선언된 변수와 다르게 const 로 선언된 상수는 재할당이 불가능합니다.

```javascript
const constant = "CONSTANT";
```





## 📖 데이터 타입

자바스크립트의 모든 값은 데이터 값을 갖고,\
크게 원시 타입 (Primitive Type) 과 객체 타입 (Object Type) 으로 구분됍니다

> 원시 타입 : Number (숫자), String (문자열), Boolean, undefined, null, symbol
>
> 객체 타입 : Object, function, Array ...

### ✏️ 숫자 Number 타입

자바스크립트에서는 <mark style="background-color:orange;">**정수와 실수를 구분하지 않고**</mark>, 모든 숫자 타입을 64비트의 부동소수점을 사용합니다.\
정수 / 실수 / 2진수 / 8진수 / 16진수는 모두 메모리에 64비트 부동소수점 형식의 2진수로 저장됍니다.

숫자 타입에서는 양의 무한대, 음의 무한대, 산술 연산 불가를 나타내는 값도 표현 가능합니다.

```javascript
Infinity
-Infinity
NaN // 산술 연산 불가
```



### ✏️ 문자열 String 타입

문자열은 작은 따옴표(''), 큰 따옴표(""), 백틱(\`\`) 으로 나타낼 수 있습니다.

```javascript
var singleQuote = '문자열';
var doubleQuote = "문자열";
var templateLiteral = `문자열`;
```

문자열을 char(문자)의 배열 또는 포인터로 표현하는 C / C++ , 객체로 표현하는 Java 와 다르게

```cpp
char *pStr = "문자열"; // C/C++
char str[] = "문자열"; // C/C++
```

자바스크립트에서는 <mark style="background-color:orange;">**원시타입이자 변경 불가능한 값**</mark>으로 표현됍니다!

```javascript
var string = "This is a String";
string[0] = "A";
console.log(string); // This is a String
```



백틱으로 표현한 문자열을 <mark style="background-color:orange;">**템플릿 리터럴**</mark> 이라고하고,\
멀티라인 문자열, $ 를 이용해서 표현식을 삽입할 수 있습니다.

```javascript
const expression = "표현식";

var templateLiteral = `
    여러 줄에 걸쳐서
    문자열을 작성할 수 있고
    이렇게 ${expression} 을 넣을 수 있습니다.
    템플릿 리터럴의 공백이나 줄바꿈은 유지 됍니다.
`;
```



### ✏️ 불리언 Boolean 타입

불리언 데이터 타입은, 참과 거짓을 나타내는 자료형으로 true, false 가 있습니다.



### ✏️ undefined

변수를 선언하고 값을 할당하지 않으면 undefined 로 초기화됍니다.



### ✏️ null

변수에 값이 없다는 것을 명시하고 싶을 때 null 을 사용합니다.

변수에 null 을 재할당하면, <mark style="background-color:orange;">기존에 참조하던 값을 참조하지 않게되고</mark>, 자바스크립트 엔진의 가비지 컬렉터가 아무도 참조하지 않는 메모리 공간에 대해 <mark style="background-color:orange;">가비지 컬렉션을 수행</mark>합니다.



### ✏️ Symbol 타입

symbol 타입은, 변경 불가능한 원시 타입의 값으로, 다른 값과 중복되지 않습니다.

주로 이름 충돌할 위험이 없는 객체의 유일한 프로퍼티 키를 만들기 위해 사용합니다.



## 📖 동적 타이핑

자바스크립트에서 변수는 선언이 아니라 할당에 의해 타입이 결정되는 <mark style="background-color:orange;">**동적 타이핑 언어**</mark>입니다.

명시적으로 타입을 선언해주고, 할당 할 수 있는 값의 종류가 정해진 정적 타이핑언어와 다르게,\
값을 할당하는 시점에 타입이 동적으로 결정되고, 변수의 타입은 언제든지 재할당을 통해 변경이 가능합니다



## 🔗 참고자료

모던 자바스크립트 딥다이브 Ch04 변수\
모던 자바스크립트 딥다이브 Ch05 데이터 타입

모던 JavaScript 튜토리얼 - 변수와 상수\
[https://ko.javascript.info/variables](https://ko.javascript.info/variables)
