# javascript confusions

## 참고

* unified guide
  * [mozilla](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide)
  * [w3schools](https://www.w3schools.com/js/default.asp)

* 요약
  * [basics 1](https://velog.io/@chyori/JavaScript-%EA%B8%B0%EB%B3%B8-%EB%AC%B8%EB%B2%95-%EC%B4%9D-%EC%A0%95%EB%A6%AC)
  * [basics 2](https://ko.javascript.info)
  * [basics 3](https://d-craftshop.tistory.com/211)

## equals

**`==` (동등 연산자)** - 타입 변환 후 비교

```javascript
5 == '5'    // true (문자열 '5'가 숫자로 변환됨)
0 == false  // true
null == undefined  // true
```

**`===` (일치 연산자)** - 타입 변환 없이 비교

```javascript
5 === '5'   // false (타입이 다름)
0 === false // false
null === undefined  // false
```

* 권장 사용:
  * **항상 `===` 사용 권장** (타입 안정성)
  * `==`은 의도치 않은 타입 변환으로 버그 발생 가능

## quotes

* `''`와 `""`는 동일 표현
* ``` `` ```는 개선된 문자열 표현

### 1. **Single Quotes (`''`)** - 작은따옴표

```javascript
let str = 'Hello World';
let message = 'It\'s a nice day';  // 이스케이프 필요
```

* 가장 기본적인 문자열 표현
* 내부에 작은따옴표 사용 시 이스케이프(`\'`) 필요

### 2. **Double Quotes (`""`)** - 큰따옴표

```javascript
let str = "Hello World";
let message = "It's a nice day";  // 이스케이프 불필요
let quote = "He said \"Hi\"";     // 이스케이프 필요
```

* 작은따옴표와 기능적으로 동일
* 내부에 큰따옴표 사용 시 이스케이프(`\"`) 필요

### 3. **Backticks (``` `` ```)** - 백틱 (Template Literals)

```javascript
// 변수 보간 (Interpolation)
let name = 'John';
let greeting = `Hello, ${name}!`;  // "Hello, John!"

// 여러 줄 문자열
let multiLine = `Line 1
Line 2
Line 3`;

// 표현식 사용
let result = `2 + 2 = ${2 + 2}`;  // "2 + 2 = 4"

// 함수 호출
let upper = `Hello ${name.toUpperCase()}`;
```

#### **핵심 차이점:**

| 특징 | `''` / `""` | ``` `` ``` |
|------|-------------|------------|
| 변수 보간 | ❌ 불가능 | ✅ `${variable}` 가능 |
| 여러 줄 | ❌ `\n` 필요 | ✅ 직접 개행 가능 |
| 표현식 평가 | ❌ 불가능 | ✅ `${expression}` 가능 |
| 이스케이프 | 해당 따옴표만 필요 | 백틱만 이스케이프 필요 |

* 권장 사용:
  * 일반 문자열: '' 또는 ""
  * 변수 삽입/여러 줄/표현식: ``

## var vs let vs const

* **`var`**: 함수 스코프, 호이스팅 발생, 재선언 가능 (사용 비권장)
* **`let`**: 블록 스코프, 재할당 가능
* **`const`**: 블록 스코프, 재할당 불가능 (객체 속성은 변경 가능)

## 화살표 함수(Arrow Function) vs 일반 함수 (Normal Function)

요약:

| 특징 | 일반 함수 | 화살표 함수 |
|------|----------|------------|
| `this` 바인딩 | 호출 시점에 결정 | 선언 시점의 상위 스코프 |
| 생성자 사용 | ✅ 가능 | ❌ 불가능 |
| `arguments` | ✅ 사용 가능 | ❌ 없음 (rest 파라미터 사용) |
| 메서드 정의 | ✅ 권장 | ❌ 비권장 |
| 콜백 함수 | `bind` 필요할 수 있음 | ✅ 간결하고 편리 |

### syntax

* 화살표 함수

```js
f = () => {}
```

* 일반 함수

```js
f = function() {}
```

### 1. **`this` 바인딩 차이**

```javascript
// 일반 함수: this가 호출 시점에 결정됨
const obj1 = {
  name: 'John',
  greet: function() {
    console.log(`Hello, ${this.name}`);
  }
};
obj1.greet();  // "Hello, John" - this는 obj1을 가리킴

// 화살표 함수: this가 선언 시점의 상위 스코프를 가리킴
const obj2 = {
  name: 'Jane',
  greet: () => {
    console.log(`Hello, ${this.name}`);
  }
};
obj2.greet();  // "Hello, undefined" - this는 obj2가 아닌 상위 스코프를 가리킴
```

### 2. **이벤트 핸들러에서의 차이**

```javascript
// 일반 함수: this가 이벤트 대상 요소
button.addEventListener('click', function() {
  console.log(this);  // <button> 요소
  this.classList.add('clicked');  // 정상 작동
});

// 화살표 함수: this가 상위 스코프
button.addEventListener('click', () => {
  console.log(this);  // Window 객체 (또는 상위 스코프)
  this.classList.add('clicked');  // 에러 발생
});
```

### 3. **콜백에서 유용한 화살표 함수**

```javascript
class Counter {
  constructor() {
    this.count = 0;
  }
  
  // 일반 함수 사용 시 - bind 필요
  startRegular() {
    setInterval(function() {
      this.count++;  // 에러! this가 Counter를 가리키지 않음
      console.log(this.count);
    }.bind(this), 1000);  // bind 필요
  }
  
  // 화살표 함수 사용 시 - bind 불필요
  startArrow() {
    setInterval(() => {
      this.count++;  // 정상 작동! this가 Counter를 가리킴
      console.log(this.count);
    }, 1000);
  }
}

const counter = new Counter();
counter.startArrow();  // 1, 2, 3, ...
```

### 4. **생성자 함수로 사용 불가**

```javascript
// 일반 함수: 생성자로 사용 가능
function Person(name) {
  this.name = name;
}
const john = new Person('John');  // 정상 작동

// 화살표 함수: 생성자로 사용 불가
const PersonArrow = (name) => {
  this.name = name;
};
const jane = new PersonArrow('Jane');  // TypeError: PersonArrow is not a constructor
```

### 5. **arguments 객체**

```javascript
// 일반 함수: arguments 객체 사용 가능
function sum() {
  console.log(arguments);  // [1, 2, 3, 4, 5]
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4, 5);  // 15

// 화살표 함수: arguments 객체 없음
const sumArrow = () => {
  console.log(arguments);  // ReferenceError: arguments is not defined
};

// 대신 rest 파라미터 사용
const sumRest = (...args) => {
  console.log(args);  // [1, 2, 3, 4, 5]
  return args.reduce((a, b) => a + b, 0);
};
sumRest(1, 2, 3, 4, 5);  // 15
```

### 6. **메서드 정의 시 주의**

```javascript
const calculator = {
  value: 0,
  
  // 잘못된 방법: 화살표 함수
  addWrong: (n) => {
    this.value += n;  // this가 calculator가 아님!
    return this.value;
  },
  
  // 올바른 방법 1: 일반 함수
  add: function(n) {
    this.value += n;  // this가 calculator
    return this.value;
  },
  
  // 올바른 방법 2: 단축 메서드 구문
  subtract(n) {
    this.value -= n;  // this가 calculator
    return this.value;
  }
};

calculator.add(5);       // 5
calculator.subtract(2);  // 3
calculator.addWrong(10); // NaN (this.value가 undefined)
```

**권장 사용:**

- **객체 메서드**: 일반 함수 또는 단축 메서드 구문
- **콜백 함수**: 화살표 함수 (특히 `this` 바인딩이 필요한 경우)
- **생성자 함수**: 일반 함수만 가능

## null vs undefined

* **`undefined`**: 값이 할당되지 않음
* **`null`**: 의도적으로 빈 값
