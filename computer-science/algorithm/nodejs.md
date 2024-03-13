# NodeJS 로 코딩테스트 문제 해결하기

> NodeJS 를 이용해서 코딩테스트 문제 해결을 시도하면서 배운것들을 정리하는 포스트입니다

## 📖 입출력

NodeJS 에서 표준 출력은 `consoole.log()` 을 사용하고,\
표준 입력은 따로 존재하지 않아 npm 패키지를 설치해야합니다.

NodeJS 를 이용해 백준 / 프로그래머스와 같은 문제를 풀기위해서는 다음과 같이 입력을 받아야 합니다

```javascript
const fs = require("fs");
let input = fs.readFileSync("/dev/stdin");
```

백준과 같은 채점 사이트에 업로드 전 로컬에서 테스트하기 위해서는 `/dev/stdin` 이 아닌,\
`.txt` 파일을 통해 입력을 받아야 합니다

```javascript
const fs = require("fs");

let input = fs
    .readFileSync(process.platform === "linux" ? "/dev/stdin" : "./input.txt")
    .toString()
    .trim();
```

이후 <mark style="background-color:orange;">**형식에 맞게 직접 타입을 변환하거나 Array 프로토타입의 메서드를 이용하여 입력을 가공**</mark>해주어야 합니다.



## 📖 아스키 코드 변환

### ✏️ 문자를 아스키코드로 변환

```javascript
"A".charCodeAt();    // 65
"abc".charCodeAt(2); // 99 (2번째 인덱스인 'c' 에 대한 아스키코드 99가 출력됩니다
```

### ✏️ 아스키코드를 문자로 변환

```javascript
String.fromCharCode(65);    // 'A'
```



## 📖 배열 Array

### ✏️ 연속된 숫자의 배열 만드는 방법

1. for 문 사용

```typescript
let arr = [];
for(let i = 0; i < SIZE; i++) arr.push(i);
```

2. Array.fill 사용

```typescript
Array(SIZE).fill().map((v,i) => i);
// Array(SIZE).fill() : undefined SIZE 개인 배열 생성
// Array.map() : 순회하며 index 가 i 인 배열 생성
```

3. Array.from, Array.keys 사용

```typescript
Array.from(Array(SIZE).keys());
// Array(SIZE).keys() 로 Iterator 객체를 생성후
// Array.from 으로 해당 Iterator 를 배열로 생성
```

4. Array.from length 사용

```typescript
Array.from({ length: SIZE }, (v, k) => k);
```



### ✏️ 2차원 배열

1. Array( ).fill( ).map()

```javascript
let arr = new Array(ROWS).fill().map(() => new Array());
```

