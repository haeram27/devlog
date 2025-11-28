# 함수

## 1. **함수는 일급 객체(First-Class Object)**

```javascript
// 함수는 객체이므로 속성을 가질 수 있음
function myFunc() {
  console.log('Hello');
}

myFunc.customProp = '함수에 속성 추가';
myFunc.count = 0;

console.log(myFunc.customProp);  // "함수에 속성 추가"
console.log(typeof myFunc);       // "function"
console.log(myFunc instanceof Object);  // true
```

## 2. **객체 생성 방법 3가지**

### 방법 1: 객체 리터럴 (가장 간단)
```javascript
// 직접 객체 생성
const person = {
  name: 'John',
  age: 30,
  greet() {
    console.log(`Hello, ${this.name}`);
  }
};

person.greet();  // "Hello, John"
```

### 방법 2: 생성자 함수 + new
```javascript
// 함수를 생성자로 사용
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

const john = new Person('John', 30);
john.greet();  // "Hello, John"
```

### 방법 3: ES6 클래스 (내부적으로는 생성자 함수)
```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  greet() {
    console.log(`Hello, ${this.name}`);
  }
}

const jane = new Person('Jane', 25);
jane.greet();  // "Hello, Jane"
```

## 3. **함수 == 객체인가?**

```javascript
// 함수는 특별한 종류의 객체
function add(a, b) {
  return a + b;
}

// 함수는 호출 가능(callable)한 객체
console.log(add(2, 3));  // 5 (함수로서 호출)

// 동시에 객체처럼 속성을 가질 수 있음
add.description = '두 수를 더하는 함수';
add.version = '1.0';

console.log(add.description);  // "두 수를 더하는 함수" (객체로서 접근)
console.log(add.name);         // "add" (내장 속성)
console.log(add.length);       // 2 (매개변수 개수)

// 타입 확인
console.log(typeof add);              // "function"
console.log(add instanceof Object);   // true
console.log(add instanceof Function); // true
```

## 4. **함수의 이중성**

```javascript
function MyFunction() {
  console.log('호출됨');
}

// 1. 함수로서 호출
MyFunction();  // "호출됨"

// 2. 생성자로서 호출
const instance = new MyFunction();  // "호출됨" + 객체 생성

// 3. 객체로서 사용
MyFunction.staticProp = '정적 속성';
console.log(MyFunction.staticProp);  // "정적 속성"

// 4. 메서드로 전달
const obj = {
  method: MyFunction
};
obj.method();  // "호출됨"
```

## 5. **함수는 객체이므로 다른 함수에 전달 가능**

```javascript
// 콜백 함수
function processArray(arr, callback) {
  for (let item of arr) {
    callback(item);
  }
}

function printItem(item) {
  console.log(item);
}

// 함수를 인자로 전달 (함수가 객체이므로 가능)
processArray([1, 2, 3], printItem);
// 1
// 2
// 3

// 익명 함수로도 전달 가능
processArray(['a', 'b', 'c'], function(item) {
  console.log(item.toUpperCase());
});
// A
// B
// C
```

## 6. **함수를 반환하는 함수 (클로저)**

```javascript
// 함수가 객체이므로 반환 가능
function createMultiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

## 7. **생성자 함수의 특별한 점**

```javascript
function Car(brand) {
  // new와 함께 호출되면
  // 1. 빈 객체가 자동으로 생성됨
  // 2. this가 그 객체를 가리킴
  
  this.brand = brand;
  
  // 3. this가 자동으로 반환됨
}

// new 없이 호출 (일반 함수로서)
const result1 = Car('Hyundai');
console.log(result1);  // undefined
console.log(window.brand);  // 'Hyundai' (전역 객체에 할당됨)

// new와 함께 호출 (생성자로서)
const result2 = new Car('Kia');
console.log(result2);  // Car { brand: 'Kia' }
console.log(result2.brand);  // 'Kia'
```

## 8. **Function 생성자로 함수 생성**

```javascript
// 함수도 객체이므로 생성자로 생성 가능
const add = new Function('a', 'b', 'return a + b');
console.log(add(2, 3));  // 5

// 위는 아래와 동일
function add2(a, b) {
  return a + b;
}
```

## 9. **프로토타입 체인**

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

const john = new Person('John');

// 프로토타입 체인 확인
console.log(john.__proto__ === Person.prototype);  // true
console.log(Person.prototype.__proto__ === Object.prototype);  // true
console.log(Object.prototype.__proto__);  // null

// john -> Person.prototype -> Object.prototype -> null
```

## 핵심 정리:

| 개념 | 설명 |
|------|------|
| **함수는 객체** | ✅ 모든 함수는 `Object`를 상속받음 |
| **함수 ≠ 일반 객체** | 함수는 **호출 가능(callable)**한 특수 객체 |
| **객체 생성** | 리터럴, 생성자 함수, 클래스 모두 가능 |
| **생성자 함수** | `new` 키워드와 함께 사용하여 객체 생성 |
| **일급 객체** | 변수 할당, 인자 전달, 반환값 사용 가능 |

## JavaScript의 특징:

```javascript
// JavaScript는 프로토타입 기반 객체지향 언어
// 클래스도 내부적으로는 함수

class MyClass {
  constructor(value) {
    this.value = value;
  }
}

console.log(typeof MyClass);  // "function"
console.log(MyClass instanceof Function);  // true

// 클래스도 결국 생성자 함수의 syntactic sugar
```

**결론:**
- 함수는 **특별한 종류의 객체** (호출 가능한 객체)
- 객체 생성은 **함수를 통해서도 가능**하지만, 리터럴이나 클래스로도 가능
- `함수 == 객체`는 아니지만, **함수 ⊂ 객체** (함수는 객체의 특수한 형태)