## 1. 자바스크립트의 동등 비교

자바스크립트의 데이터 타입

- 원시타입
  - boolean : 참과 거짓
  - null : 아직 값이 없거나 비어있는 값 (typeof null은 object), 명시적으로 비어 있음을 나타내는 값
  - undefined : 할당하지 않은 변수 또는 값이 주어지지 않은 인수에 자동으로 할당되는 값, 선언됐지만 할당되지 않은 값
  - number : -(2^53 - 1) ~ 2^53 - 1 사이의 값
  - string : 텍스트 타입의 데이터
  - symbol : 중복되지 않는 어떠한 고유한 값
  - bigint : number보다 더 큰 수 저장 가능
- 객체 타입
  - object : 원시 타입 이외의 타입, (ex. 배열, 함수, 정규식, 클래스)

### 값을 저장하는 방식의 차이

객체 자체를 비교하면 false, 원시값인 내부 속성 값을 비교하면 동일

### Object.is

Quiz 1.

```js
Number.NaN === NaN; // ?
Object.is(Number.NaN, NaN); // ?
Object.is({}, {});
```

### 리액트에서의 동등 비교

얕은 비교로 Object.is를 사용한다

## 2. 함수

### 함수를 저장하는 4가지 방법

1. 함수 선언문

```js
function add(a, b) {
  return a + b;
}
```

2. 함수 표현식

```js
const add = function (a, b) {
  return a + b;
};
```

함수 선언문 vs 함수 표현식
함수 선언문 호이스팅 가능, 함수 표현식 호이스팅 불가능

3. Function 생성자

클로저 불가능, eval만큼 실제 코딩에서 사용되지 않는 방법이다.

```js
const add = new Function("a", "b", "return a + b");
```

4. 화살표 함수

```js
const add = (a, b) => {
  return a + b;
};
```

constructor 사용 불가능, arguments 존재 X
this. 일반 함수는 전역 객체를, 화살표 함수는 상위 스코프의 this를 따르게 된다.

### 다양한 함수

즉시 실행 함수 (Immediately Invoked Function Expression, IIFE)

```js
(function (a, b) {
  return a + b;
})(1, 2);
```

고차 함수

Array.prototype.map

```js
const add = function (a) {
  return function (b) {
    return a + b;
  };
};

add(1)(3);
```

### 함수 주의 사항

1. 부수 효과 억제

외부에 영향을 끼치는 것(부수 효과)을 최소화해야한다.

2. 함수를 작게

3. 누구나 이해할 수 있는 이름 붙이기

## 3. 클래스

### 구성 요소

1. constructor
   생성자로 객체를 생성하는데 사용하는 메서드, 하나만 존재

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}
```

2. 프로퍼티

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}

const jun = new Person("JH"); // 프로퍼티 값을 넘겨줌
```

3. getter와 setter

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  get getName() {
    return this.name[0];
  }

  set setName(char) {
    this.name = [char, ...this.name.slice(1)].join("");
  }
}

const jun = new Person("JH"); // 프로퍼티 값을 넘겨줌

jun.getName; // J
jun.setName = "A";

console.log(jun.name); // AH
```

4. 인스턴스 메서드

클래스 내부에 선언한 메서드, prototype에 선언되므로 프로토타입 메서드로 불리기도 한다.

5. 정적 메서드

함수 앞에 static을 붙여 인스턴스가 아닌 클래스 이름으로 호출할 수 있는 메서드이다.

6. 상속

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}

class Admin extends Person {
  constructor(name) {
    super(name);
  }

  hello() {
    console.log("어드민이에요");
  }
}

const admin = new Admin("JJH");
admin.hello();
```

### 클래스와 함수의 관계

생성자 함수로 유사하게 재현 가능

## 4. 클로저

### 클로저란?

함수와 함수가 선언된 Lexical Scope의 조합

```js
function add() {
  const a = 1;
  function innerAdd() {
    const b = 2;
    console.log(a + b);
  }
  innerAdd();
}

add();
```

### 스코프

전역 스코프 : 전역 레벨에 선언
함수 스코프 : {}블록이 스코프 범위를 결정하지 않고 함수 레벨 스코프를 따른다. (var만, let은 블록 스코프)

### 클로저 활용

Quiz

```js
function Counter() {
  var counter = 0;

  return {
    increase: function () {
      return ++counter;
    },
  };
}

var c = Counter();

console.log(c.increase()); // ?
console.log(c.counter); // ?
```

## 5. 이벤트 루프와 비동기 통신

### 자바스크립트는 싱글 스레드

### 이벤트 루프

이벤트 루프로 싱글 스레드인 자바스크립트를 비동기로 처리할 수 있다 (런타임 외부에서 비동기 실행을 돕기 위해 만들어진 장치)

setTimeout은 태스크 큐로 들어간다.(set형태)
비동기 함수 또한 메인 스레드가 아닌 태스크 큐가 할당되는 별도의 스레드에서 수행

### 태스크 큐와 마이크로 태스크 큐

태스크 큐 : setTimeout, setInterval, setImmediate
마이크로 태스크 큐 : process.nextTick, Promise, queueMicroTask, MutationObserver

마이크로 태스크 큐가 우선권이 있다.

브라우저 렌더링 작업은 마이크로 태스크 큐와 태스크 큐 사이에서 일어난다.

## 6. 리액트에서 자주 사용하는 자바스크립트 문법

### 구조 분해 할당

Quiz

```js
const array = [1, 2, 3, 4, 5];
const [first, second, ...arrayRest] = array; // first? second? arrayRest?

const [a = 1, b = 1, c = 1, d = 1] = [undefined, null, 0, ""]; // a? b? c? d?
```

```js
const key = "a";
const object = {
  a: 1,
  b: 1,
};

const { [key]: a } = object; // a = 1
```

### 전개 구문

```js
const obj1 = {
  a: 1,
  b: 2,
};

const obj2 = {
  c: 3,
  d: 4,
};

const newObj = { ...obj1, ...obj2 };
```

### map, filter, reduce, forEach

map: 인수로 전달받은 배열과 똑같은 길이의 새로운 배열 반환 메서드

```js
const arr = [1, 2, 3];
const addArr = arr.map((item) => item + 1);
```

filter: truthy 조건을 만족하는 경우에만 해당 원소 반환

```js
const arr = [1, 2, 3];
const oddArr = arr.filter((item) => item % 2 === 1); // [1, 3]
```

reduce: 초깃값에 누적하여 결과 반환

```js
const arr = [1, 2, 3];
const sum = arr.reduce((res, item) => {
  return result + item;
}, 0); // 6 0은 res의 초깃값
```

forEach: 콜백 함수를 받아 실행, break, return을 이용해도 순회를 멈출 수 없다.

```js
const arr = [1, 2, 3];

arr.forEach((item) => console.log(item));
```

map과의 차이: map은 반환값 O, forEach는 X

### 삼항 조건 연산자

조건문 ? 참일 때 값 : 거짓일 때 값

## 7. 타입 스크립트

### 타입 스크립트 활용법

1. any대신 unknwon을 활용하자

존재가 불가능 할 때 (ex. string & number) never가 사용된다

2. 타입 가드 적극 활용하기

instanceof (특정 클래스의 인스턴스인지)
typeof (특정 요소에 대한 자료형)
in (객체에 키가 존재하는지)

3. 제네릭

단일 타입이 아닌 다양한 타입에 대응할 수 있는 도구

```js
function getFirstLast<T>(list: T[]): [T, T] {
  return [list[0], list[list.length - 1]];
}

const [first, last] = getFirstLast([1, 2, 3]); // first와 last는 number

const [firstS, lastS] = getFirstLast(["a", "b", "c"]); // firstS와 lastS는 string
```

4. 인덱스 시그니처
   객체의 키를 정의

```js
type Hi = {
  [key: string]: string,
};

const hi: Hi = {
  hello: "hello",
  hi: "hi",
};
```

### 타입스크립트 전환

tsconfig.json 설정
JSDoc과 @ts-check로 점진적으로 전환
@types 모듈 설치
파일 단위로 조금씩 전환
