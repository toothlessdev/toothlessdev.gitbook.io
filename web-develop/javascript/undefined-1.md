# 연산자와 타입변환, 단축평가

> 이 포스트는 모던 자바스크립트 Deep Dive 를 공부하면서 정리한 글입니다



자바스크립트의 연산자의 종류에는 크게\
산술연산자, 할당연산자, 비교연산자, 삼항 조건 연산자, 논리연산자, 쉼표연산자, 그룹연산자, typeof 연산자, 지수연산자 가 있습니다

## 📖 산술연산자

### ✏️ 이항 산술연산자

\+ - \* / %\
이항 연산자는 연산 이후, 피연산자의 값을 바꾸지 않고 새로운 값을 만들어냅니다.

### ✏️ 단항 산술연산자

1. \++ (증가 산술연산자)
2. \-- (감소 산술연산자)

증가 / 감소 산술연산자는 연산자의 위치에 따라 전위 / 후위 증가 / 감소 산술연산자로 나뉩니다.\
전위 연산자는 값을 증가/감소 후 연산을 수행하고\
후위 연산자는 연산을 수행하고 값을 증가/감소합니다

3. \+ 단항연산자

<mark style="background-color:orange;">숫자 타입이 아닌 피연산자에 + 단항 연산자를 사용하면 숫자 타입으로 변환</mark>된 값을 반환합니다.

```javascript
console.log(+"123"); // 123

console.log(true);  // 1
console.log(false); // 0

console.log("string"); // NaN
```

4. \- 단항연산자

피연산자의 부호를 반전한 값을 반환합니다.\
마찬가지로 <mark style="background-color:orange;">숫자 타입이 아닌 피연산자에 - 단항 연산자를 사용하면 숫자 타입으로 변환</mark>된 값을 반환합니다.

5. \+ 문자열 연결 연산자

<mark style="background-color:orange;">피연산자중 하나 이상이 문자열</mark>일 경우, 문자열 연결 연산자로 동작합니다.



### &#x20;🔗 JS 에서 banana 출력하기

```javascript
console.log('b' + 'a' + +'🍌' + 'a');
```

해설) 숫자 타입이 아닌 '🍌' 에 + 단항연산자를 사용하면 숫자 타입으로 변환된 값을 반환하는데,\
'🍌' 는 숫자타입으로 변환이 안되기 때문에 NaN 이 나오고 앞에 있는 문자열 연결 연산자와 결합되어\
`baNaNa` 가 출력된다



## 📖 할당연산자&#x20;

[=, +=, -=, \*=, /=, %=\
](#user-content-fn-1)[^1]우항에 있는 피연산자의 평가 결과를 좌항에 할당합니다.

```javascript
a = b = c = d = 0;
```

연쇄 할당은 우측에서 좌측으로 진행됩니다.



## 📖 비교연산자

### ✏️ 동등 비교연산자

`==` 는 비교시 <mark style="background-color:orange;">암묵적 타입변환을 통해 타입을 일치 시킨 후</mark> 같은 값인지 비교합니다

### ✏️ 일치 비교연산자

`===` 는 <mark style="background-color:orange;">타입도 동일하고 값도 같은 경우</mark>에 참을 반환합니다

### ✏️ NaN 비교

NaN 은 자신과 일치하지 않는 유일한 값으로, 표현식이 NaN 인지 확인하려면 `Number.isNaN` 을 사용해야합니다

```javascript
console.log(NaN === NaN); // false
console.log(Number.isNaN(NaN)); // true
console.log(Number.isNaN(10)); // false
console.log(Number.isNaN(1 + undefined)); // true
```

### ✏️ 대소 비교



## 📖 삼항연산자

`(조건식) ? (조건식이 참일때 반환값) : (조건식이 거짓일때 반환값)`

조건식에 들어오는 값은 boolean 으로 암묵적 타입 변환됍니다

### ✏️ 삼항 연산자는 표현식이다

삼항 연산자와 if else 문의 차이점은\
삼항 연산자 표현식은 값처럼 사용할 수 있지만, if.. else 문은 값처럼 사용할 수 없습니다

(if ... else if ... else 는 표현문 statement 이다)



## 📖 논리연산자

`||` OR 논리합 연산자\
`&&` AND 논리곱 연산자\
`!` NOT 부정 연산자



## 📖 typeof 연산자

피연산자의 데이터타입을 문자열\
`"string", "number", "boolean", "undefined", "symbol", "object", "function"` \
으로 반환합니다

```javascript
typeof ""            // "string"
typeof 1             // "number"
typeof NaN           // "number"
typeof true          // "boolean"
typeof undefined     // "undefined"
typeof Symbol()      // "symbol"
typeof null          // "object"
typeof []            // "object"
typeof {}            // "object"
typeof new Date()    // "object"
typeof /test/        // "object"
typeof function() {} // "function"
```

⭐️ `typeof null` 은 `"object"` 를 반환한다\
⭐️ 따라서 값이 null 인지 확인할 때는 `variable === null` 를 사용해야한다



## 📖 연산자 우선순위

... > 단항연산자 > 이항연산자 > 비교연산자 > 논리연산자 > 삼항연산자 > 대입연산자 > ...

<table><thead><tr><th width="165">우선순위</th><th width="356">연산자</th></tr></thead><tbody><tr><td>1</td><td>( )</td></tr><tr><td>2</td><td>new(args...) [] () ?.</td></tr><tr><td>3</td><td>new()</td></tr><tr><td>4</td><td>x++ x--</td></tr><tr><td>5</td><td>!x +x -x ++x --x typeof delete</td></tr><tr><td>6</td><td>**</td></tr><tr><td>7</td><td>* / %</td></tr><tr><td>8</td><td>+ -</td></tr><tr><td>9</td><td>&#x3C; &#x3C;= > >= in instanceof</td></tr><tr><td>10</td><td>== !== === !==</td></tr><tr><td>11</td><td>??</td></tr><tr><td>12</td><td>&#x26;&#x26;</td></tr><tr><td>13</td><td>||</td></tr><tr><td>14</td><td>? ... : ...</td></tr><tr><td>15</td><td>= += -= *= ...</td></tr><tr><td>16</td><td>,</td></tr></tbody></table>

***

## 📖 타입변환

타입변환에는 두가지 종류가 있습니다

개발자가 의도적으로 값의 타입을 변환하는 <mark style="background-color:orange;">명시적 타입 변환 (타입 캐스팅)</mark>,\
자바스크립트 엔진에 의해 값이 자동 변환되는 <mark style="background-color:orange;">암묵적 타입 변환 (타입 강제 변환)</mark> 입니다

### ✏️ 암묵적 타입변환

1. **문자열 연결 연산자**

\+ 연산자에 피연산자중 하나 이상이 문자열이면 문자열 연결 연산자로 동작합니다

```javascript
0 + ''   // "0"
-0 + ''  // "0"
1 + ''   // "1"
NaN + '' // "NaN"
Infinity + '' // "Infinity"
-Infinity + '' // "-Infinity"

true + '' // "true"
false + '' // "false"

null + '' // "null"
undefined + '' // "undefined"
(Symbol()) + '' // Cannot convert a Symbol to a string

({}) + '' // "[object Object]"
Math + '' // "[object Math]"
[] + ''   // ""
[10, 20] + '' // "10,20"
(function(){}) + '' // "function(){}"
Array + '' // "function Array() { [native code] }"
```

2. **숫자 타입 변환**

산술 연산자는 숫자 값을 반환합니다. 피연산자를 숫자 타입으로 변환할 수 없는 경우에는 `NaN` 을 반환합니다

```javascript
1 - "1" // 0
1 * 10 // 10
1 / 'one' // NaN
```

\+ 단항 연산자를 사용하면 숫자 타입의 값이 아니면, 숫자 타입의 값으로 암묵적 타입 변환을 수행합니다

```javascript
+"" // 0
+"0" // 0
+"1" // 1
+"string" // NaN

+true // 1
+false // 0

+null // 0
+undefined // 0
+Symbol() // Cannot Convert a Symbol value to a number

+{} // NaN
+[] // 0
+[10,20] // NaN
+(function(){}) // NaN
```

3. **불리언 타입으로 변환**

제어문 또는 삼항 연산 조건자의 조건식은 평가시에 불리언 타입으로 암묵적 타입 변환을 수행합니다

```javascript
if("") //true
if(true) // true
if(0) // false
if("str") // true
if(null) // false
```



자바스크립트 엔진은 불리언 타입이 아닌 값을 Truthy 값 (참으로 평가되는 값)\
또는 Falsy 값 (거짓으로 평가되는 값) 으로 구분합니다

`false` `undefined` `null` `0, -0` `NaN` `''` 은 <mark style="background-color:orange;">**Falsy**</mark> 값으로 취급됩니다

❗️`{ }` 와 `[ ]` 는 모두 <mark style="background-color:orange;">**Truthy**</mark> 값으로 취급됩니다



### ✏️ 명시적 타입변환

개발자의 의도에 따라 명시적으로 타입을 변경하기 위해서는

1. 표준 빌트인 생성자 함수를 new 연산자 없이 호출
2. 빌트인 메서드를 사용
3. 암묵적 타입 변환 이용

를 이용할 수 있습니다



1. 문자열 타입으로 변환

```javascript
String(1) // "1"
String(NaN) // "NaN"
String(Infinity) // "Infinity"
String(true) // "true"
String(false) // "false"

(1).toString() // "1"
(NaN).toString() // "NaN"
(Infinity).toString() // "Infinity"

(true).toString() // "true"
(false).toString() // "false"

1 + "" // "1"
NaN + "" // "NaN"
Infinity + "" // "Infinity"
true + "" // "true"
false + "" // "false"
```

2. 숫자 타입으로의 변환

```javascript
Number("0") // 0
Number("-1") // -1
Number("10.53") // 10.53

Number(true) // 1
Number(false) // 0

parseInt("0") // 0
parseInt("-1") // -1
parseInt("10.53") // 10.53

+"0" // 0
+"-1" // -1
+"10.53" // 10.53

+true // 1
+false // 0

"0" * 1 // 0
"-1" * 1 // -1
"10.53" * 1 // 10.53

true * 1 // 1
false * 1 // 0
```

3. 불리언 타입으로 변환

```javascript
Boolean("x") //true
Boolean("") //false
Boolean("false") //true

Boolean(0) //false
Boolean(1) //true
Boolean(NaN) //false
Boolean(Infinity) //true

Boolean(null) //false
Boolean(undefined) //false

Boolean({}) //true❗️
Boolean([]) //true❗️

!!"x" //true
!!"" //false
!!"false" //true

!!0 //false
!!1 //true
!!NaN //false
!!Infinity //true

!!null //false
!!undefined //false

!!{} //true❗️
!![] //true❗️
```



## 📖 단축평가

### ✏️ 논리 연산자를 사용한 단축평가

논리 연산자 `||` `&&` 표현식의 평가 결과는 불리언 값이 아닐 수 있습니다

| 단축 평가 표현식           | 평가 결과    |
| ------------------- | -------- |
| true \|\| anything  | true     |
| false \|\| anything | anything |
| true && anything    | anything |
| false && anything   | false    |



### ✏️ 옵셔널 체이닝 연산자

옵셔널 체이닝 연산자는 `?.` 좌항의 피연산자가 null 또는 undefined 인 경우, `undefined` 를 반환하고,\
그렇지 않으면 우항의 프로퍼티 참조를 이어갑니다



### ✏️ null 병합 연산자

null 병합 연산자는 `??` 좌항의 피연산자가 `null` 또는 `undefined` 인 경우,\
우항의 피연산자를 반환하고, 그렇지 않은 경우 좌항의 피연산자를 반환합니다

`||` 는 Falsy 한 값이면 우항의 피연산자를 반환합니다





[^1]: 
