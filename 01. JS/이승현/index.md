## JS 동등 비교

리액트 컴포넌트의 렌더링이 일어나는 이유 중 하나는 props의 동등 비교이다.
(객체의 얕은 비교를 기반으로 이루어진다.)

이외에 가상 DOM과 실제 DOM의 비교, 리액트 컴포넌트가 렌더링할지를 판단하는 방법, 변수나 함수의 메모이제이션 등 모두 동등 비교를 기반으로 한다.

## Data Type

| 타입                   | 이름      | 설명                                                                                                                                                                                                                      |
| ---------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Primitive type         | boolean   | true와 false만 가질 수 있는 데이터 타입                                                                                                                                                                                   |
|                        | null      | 아직 값이 없거나 비어 있는 값                                                                                                                                                                                             |
|                        | undefined | 선언한 후 값을 할당하지 않은 변수 또는 값이 주어지지 않은 인수에 자동으로 할당되는 값                                                                                                                                     |
|                        | number    | -2<sup>53</sup>+1 부터 2<sup>53</sup> 사이의 값을 지정할 수 있다.<br>(`Number.MIN_SAFE_INTEGER` ~ `Number.MAX_SAFE_INTEGER`)<br>2진수, 8진수, 16진수는 자료형이 number이기 때문에 사용될 때는 10진수로 변환되서 사용된다. |
|                        | string    | 불변 타입의 원시 타입                                                                                                                                                                                                     |
|                        | symbol    | 중복되지 않는 어떠한 고유한 값을 나타내기 위해 만들어졌다.                                                                                                                                                                |
|                        | BigInt    | number의 숫자 제한을 극복하기 위해 타입,                                                                                                                                                                                  |
| object/ reference type | object    |                                                                                                                                                                                                                           |

## Primitive type

객체가 아닌 모든 타입

## falsy

| 값              | 타입           | 설명                                                     |
| --------------- | -------------- | -------------------------------------------------------- |
| false           | Boolean        |                                                          |
| 0, -0, 0n, 0x0n | Number, BigInt | 부호나 소수점 유무와 상관 없이 0은 falsy                 |
| NaN             | Number         | 숫자가 아닌 Number는 falsy                               |
| '', "", ``      | String         | 문자열이 falsy하기 위해서는 반드시 공백이 없는 빈 문자열 |
| null            | null           |                                                          |
| undefined       | undefined      |                                                          |
falsy가 아닌 값은 전부 truthy
(고로 객체와 배열은 비어 있어도 true이다.)

## BigInt

```js
Math.pow(2,53) === Math.pow(2,53)+1 // true
```

기존 number는 2<sup>53</sup>이 넘으면 허용 범위를 넘어 이상하게 계산되었다.

```js
const bigInt = 7289347823984n; // n을 붙히거나
const bigInt = BigInt('7289347823984'); // BigInt() 함수를 사용해서 표현한다.
```

## Symbol

```js
const key1 = Symbol('key'); // Synbol() 함수를 사용해서 표현한다.
const key2 = Symbol('key'); 

key1 === key2; // false

// 동일한 값을 사용하기 위해서는 Symbol.for를 활용한다.
Symbol.for('hello') === Symbol.for('hello') //true
```

## Primitive와 Object의 차이

## 값을 저장하는 방식

| 구분      | Primitive            | Object                                                       |
| --------- | -------------------- | ------------------------------------------------------------ |
| 저장 형태 | 불변 형태로 저장<br> | 변경 가능한 형태로 저장<br>(값 복사시 값이 아닌 참조를 전달) |

[[1. Data Type]]

// 문제 1 
```js
var hello = {
	greet: 'hello, world';
}
var hi = {
	greet: 'hello, world';
}

console.log(hello === hi); // false
console.log(hello.greet === hi.greet) // true
```
예제에서 hello와 hi가 같은 값을 가진 객체일 때 hello == hi 는 false인데 hello.프로퍼티 == hi.프로퍼티 는 true인 이유

객체 간의 비교는 값이 같더라도 `false`가 나올 수 있다.

## 비교를 위한 방법

| 이름      | 설명                                                                                |
| --------- | :---------------------------------------------------------------------------------- |
| Object.is | 형변환 없이 타입이 다르면 false                                                     |
| ==        | 양쪽이 같은 타입이 아니라면 형변환을 한 후 비교                                     |
| ===       | -, +사이, Number.NaN과 NaN 사이, NaN과 0/0 사이를 구분하지 못하는 Object.is와 같다. |
Object.is도 객체 비교에서는 다른 두 비교방법과 유사하다.

```js
Object.is({},{}) // false

const a = {
	hello: 'hi',
}
const b = a

Object.is(a,b) //true
a === b // true
```

## React 동등 비교

리액트에서는 `Object.is`를 이용해서 동등 비교를 진행한다.

ES6에서 제공하는 기능이기에 Polyfill을 함께 사용한다.

```ts
function is(x: any, y: any){
	return (
		(x === y && (x !== 0 || 1/x === 1/y)) || (x !==x && y !== y) // esssslint-disable-line no-self-compare
	)
}

const objectIs: (x: any y: any) => boolean =
	typeof Object.is === 'function' ? Object.is : is

export default objectIs
```

리액트에서는 `Object.is`를 기반으로 `shallowEqual` 이라는 함수를 만들어 사용한다.

```ts
import is from "./objectIs"; // 위 코드의 is를 가져온다.

import hasOwnProperty from "./hasOwnProperty";

function shallowEqual(objA: mixed, objB: mixed): boolean {
  if (is(objA, objB)) { // Object.is(objA, objB) true면 true
    return true;
  }

  if (
    typeof objA !== "object" || // 둘중 하나가 object가 아니거나 null이면 false
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  ) {
    return false;
  }

  const keysA = Object.keys(objA); // key를 꺼냄
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) { // key의 수가 다르면 false
    return false;
  }

  for (let i = 0; i < keysA.length; i++) {
    const currentKey = keysA[i];
    if (
      !hasOwnProperty.call(objB, currentKey) || 
      // A의 프로퍼티 키를 objB를 this로 해서 실행 시켰을 때 objB의 고유 프로퍼티 키로 취급하지 않고 
      // (해당 키가 objB에도 있는지 확인)
      !is(objA[currentKey], objB[currentKey])
	  // A의 프로퍼티값와 B의 프로퍼티값이 동일하지 않으면
    ) {
      return false; // false
    }
  }

  return true; // 아니면 true
}

export default shallowEqual;
```

결론은 Object.is로 비교를 수행한 후에

key를 꺼내서 다시 한번 비교를 수행한다. 

(객체 간의 얕은 비교를 한번 더 수행 (첫 번째 깊이에 존재하는 값만 비교) )

```js
Object.is({hello: 'world'},{hello: 'world'}) // false

shallowEqual({hello: 'world'},{hello:'world'}) // true

// 2 depth를 넘어가면 비교를 수행하지 않는다.
shallowEqual({hello: {hi:'world'}},{hello:{hi:'world'}}) // false
```

이는 리액트에서 Props가 얕은 비교만으로 충분히 비교가 가능하기 때문이다.

```ts
type Props = {
	hello: string
}
function HelloComponent(props: Props){
	return <h1>{props.hello}</h1>
}
// ...

function App(){
	return <HelloComponent hello="hi!" />
}
```

props는 객체이고 리액트는 이걸 기준으로 렌더링을 하기 때문에 얕은 비교를 사용해서 구현한 것이다.

```ts
import { memo, useEffect, useEffect } from "react";

type Props = { // 타입 지정
  counter: number;
};

const Component = memo((props: Props) => { // 1 depth Props
  useEffect(() => {
    console.log("Component has been rendered");
  });

  return <h1>{props.counter}</h1>;
});

type DeeperProps = {
  counter: {
    counter: number;
  };
};

const DeeperComponent = memo((props: DeeperProps) => { // 2 depth Props
  useEffect(() => {
    console.log("DeeperComponent has been rendered!");
  });

  return <h1>{props.counter.counter}</h1>;
});

export default function App() {
  const [, setCounter] = useState(0);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }
  return (
    <div className="App">
      <Component counter={100} />
      <DeeperComponent counter={{ counter: 100 }} />
      <button onClick={handleClick}>+</button> 
      {// 강제 렌더링을 위한 버튼}
    </div>
  );
}
```

button을 누르면 2 depth props인 DeeperProps가 연결된 memo의 경우 메모이제이션이 정상적으로 이루어지지 않는 것을 확인할 수 있다.

## 함수

작업을 수행하거나 값을 계산하는 등의 과정을 표현하고, 이를 하나의 블록으로 감싸서 실행 단위로 만들어 놓은 것

```ts
function sum(a: number, b: number): number {
  return a + b;
}
sum(10, 24);
```

```ts
function Component(props){
	return <div>{props.hello}</div>
}
```

## 함수를 정의하는 4가지 방법

## 함수 선언문

```js
function add(a,b){
	return a+b
}
```

statement (어떠한 값도 표현되지 않았다.)

expression이 아님(무언가 값을 산출하는 구문을 expression이라고 함.)

```js
const sum = function sum(a,b){
	return a+b
}

sum(10,24); // 34
```

위와 같은 경우 JS 엔진이 statement를 문맥에 따라서 expression으로 처리한 것이다. 

이름을 가진 함수 리터럴은 코드 문맥에 따라 선언문, 표현식 둘 다로써 사용되어질 수 있다.

## 함수 표현식

`일급 객체` : 다른 객체들에 일반적으로 적용 가능한 연산을 모두 지원하는 객체

함수는 다른 함수의 매개변수, 반환값, 할당 전부 다 되기 때문에 일급 객체라고 할 수 있다.

```js
const sum = function (a,b){
	return a+b;
}

sum(10,24) // 34
```

보통 표현식으로 쓰일 때는 이름을 빼고 사용하는 것이 좋다.

```js
const sum = function add (a,b){
	// 현재 실행중인 함수를 참조할 수 있는데 실제 프로덕션에서는 쓰면 안된다.
	console.log(arguments.callee.name)
	return a + b
}

add(10,24) // Uncaught ReferenceError: add is not defined
```

add를 sum에 담는 로직인데 외부에서 add를 호출하려고 하면 할 수 없게 된다. 

이 때문에 이름을 빼는 것이 좋다는 것이다.

## 표현식과 선언식의 차이 : Hoisting

[[2. Execution Context]]

// 문제 2
실행 컨텍스트에서 호이스팅을 일으키는 요소는?

(EnvironmentRecord가 미리 함수 선언문, 변수명을 메모리에 등록해놓고 실행 컨텍스트가 수행되는 작용)
함수에 대한 선언을 실행 전에 미리 메모리에 등록하는 작업을 의미한다.

함수 표현식은 변수에 함수를 할당하기 때문에 호이스팅이 일어날 때 해당 변수만 (`var`일 경우에만) 호이스팅되고 그 변수는 `undefined` 상태로 초기화 된다.

```js
console.log(typeof hello === "undefined");
  
hello();
  
var hello = function () {
  console.log("hello");
};
  
hello();
```

## Function 생성자

```js
const add = new Function("a", "b", "return a+ b");

add(10, 24);
```

실제 코딩에서 사용되진 않는다. (클로저도 안 생김)

## 화살표 함수

ES6 신 문법

```js
const add = (a,b)=>{
	return a+b
}

const add = (a,b)=> a+b
```

그냥 함수와는 다른 특징을 갖고 있다.

1. constructor를 사용할 수 없다. 생성자 함수로 화살표 함수를 사용하는 것은 불가능하다.
```js
const Car = (name) => {
	this.name = name;
}

const myCar = new Car('하이')// Uncaught TypeError: Car is not a constructor
```

2. arguments가 존재하지 않는다.
```js
function hello(){
	console.log(arguments)
}

// Arguments(3)[1,2,3,callee:ff, Symbol(Symbol.iterator):f]
hello(1,2,3)

const hi = ()=>{
	console.log(arguments)
}

// Uncaught ReferenceError: arguments is not defined
hi(1,2,3)
```

 3. this Binding이 다르게 동작함.

일반 함수는 호출 방식에 따라서 `this`가 바뀌게 됨. (함수로써 호출되면 전역 객체, 메소드로써 호출되면 호출한 객체 자체)

화살표 함수는 상위 스코프의 `this`를 무조건 따른다.

클래스 컴포넌트 기반으로 알아보자.

```js
class Component extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      counter: 1,
    };
  }

  functionCountUp() {
    console.log(this); // undefined
    this.setState((prev) => ({ counter: prev.counter + 1 }));
  }

  ArrowFunctionCountUp = () => {
    console.log(this); // class Component
    this.setState((prev) => ({ counter: prev.counter + 1 }));
  };
  render() {
    return (
      <div>
        <button onClick={this.functionCountUp}>일반 함수</button>
        {/* Cannot read properties of undefined (reading 'setState') */}
        <button onClick={this.ArrowFunctionCountUp}>화살표 함수</button>
        {/* 정상 작동 */}
      </div>
    );
  }
}
```

바벨에서도 이러한 this의 특징을 확인할 수 있다.

```js
// 바벨로 변환 전
const hello = ()=>{
  console.log(this)
}

// 바벨로 변환
var _this = void 0
  
var hello = function hello(){
  console.log(_this)
}
```

## 리액트에서 자주 쓰이는 함수의 종류

## 즉시 실행 함수

IIFE (Imediate Invoked Function Expression)  함수를 정의하고 그 순간 즉시 실행되는 함수

```js
(function (a,b){
 return a+b
})(10, 24); // 34

((a,b)=>{
	return a+b
  },
)(10,24) // 34
```

한번 선언하고 호출된 이후부터는 더 이상 재호출이 불가능하기 때문에 이름을 붙이지 않는다.

글로벌 스코프를 오염시키지 않는 독립적인 함수 스코프를 운용할 수 있게 된다.

재사용되지 앟는 함수이고 단 한번만 실행된다는 특징으로 리팩토링에 도움을 줄 수 있다.

## 고차 함수

함수를 인수로 받거나 결과로 새로운 함수를 반환시키는 함수 (Higher Order Function)

```js
const doubledArray = [1,2,3].map((item)=> item * 2)

doubledArray // [2,4,6]

// 함수를 반환하는 고차 함수의 예
const add = function (a){
	// a가 존재하는 클로저를 생성
	return function (b){
		// b를 인수로 받아 두 합을 반환하는 또 다른 함수를 생성
		return a + b
	}
}

add(1)(3)
```

이러한 특징으로 Higher Order Component 를 만들 수도 있는데 

컴포넌트 내부에서 공통으로 관리되는 로직을 분리해 관리할 수 있어 효율적 리팩토링이 가능하다.

## 함수를 사용할 때 주의 사항

## 함수의 sideEffect를 줄여라

sideEffect : 함수 내의 작동으로 인해 함수가 아닌 함수 외부에 영향을 끼치는 것

순수 함수 : sideEffect가 없는 함수, 어떠한 상황에서든 동일한 인수를 받으면 동일한 결과를 반환하는 함수

```js
function PureComponent(props) {
  const { a, b } = props;
  return <div>{a + b}</div>;
}
```

위 컴포넌트는 a+b한 후 HTMLDivElement로 렌더링하는 순수한 함수 컴포넌트이다.

외부에 어떤 영향을 미치지 않았고 어디서든 동일한 인수에는 동일한 결과를 반환한다.

하지만 순수한 함수로만 코드를 작성할 수는 없다.

API호출하는 로직이 컴포넌트 내에 있다면 외부에 영향(HTTP request)을 미쳤기 때문에 순수할 수 없다.

`console.log`로도 외부에 영향(콘솔 창에 영향), HTML 문서의 title을 바꿨다면 이것 또한 외부에 영향이라고 할 수 있다.

웹 개발에서 Side Effect는 필수적인 요소임과 동시에 줄여야 하는 존재인 것이다.

리액트적으로 보면 `useEffect`의 사용을 줄이는 것이 side Effect를 줄이는 노력중 하나라고 할 수 있다.

이를 통해 함수의 역할을 좁히고 버그를 줄이며 컴포넌트의 안정성을 높일 수 있다.

## 가능한 한 함수를 작게 만들어라

ESLint에는 `max-lines-per-function`이라는 규칙이 있다.

함수당 코드의 길이가 길어질 수록 추적하기 힘들어지고 품질이 떨어진다는 것이다.

하나의 함수에서 너무나 많은 일을 하지 않게 하는 것이 좋다는 거다.
(Do One Thing and Do It Well)

## 누구나 이해할 수 있는 이름을 붙여라

Terser로 한글로 변수명, 함수명 작성해도 되게 만들어 줄 수 있음.

useEffect나 useCallback등 훅에 넘겨주는 콜백 함수에 네이밍을 붙여주면 가독성에 도움이 된다.

```js
useEffect(function apiRequest(){
	// ... do something
}, [])
```

기능적으로 다른 건 크게 없지만 (디버깅 이외에) 가독성에 도움이 된다.

## 클래스

16.8 이전까지는 리액트의 모든 컴포넌트는 Class였다.

과거의 코드를 읽고 개선하기 위해서는 클래스에 대한 내용 역시 알고 있어야 한다.

## 개념

클래스 : 특정한 객체를 만들기 위한 일종의 템플릿과 같은 개념, 특정한 형태의 객체를 반복적으로 만들기 위해 사용되는 것

```js
class Car {
  name: string;
  // contstructor 는 생성자, 최초 생성시 어떤 인수를 받을 지 결정
  constructor(name: string) {
    this.name = name;
  }

  // method
  honk() {
    console.log(`${this.name}이 경적을 울립니다.`);
  }

  // static method
  static hello() {
    console.log("저는 자동차입니다.");
  }

  // setter
  set age(value) {
    this.carAge = value;
  }

  // getter
  get age() {
    return this.carAge;
  }
}

// myCar라는 Car instance 생성
const myCar = new Car("자동차");

// 메서드 호출
myCar.honk();

// 정적 메서드는 클래스에서 직접 호출한다.
Car.hello();

// 정적 메서드는 클래스로 만든 객체에서는 호출할 수 없다.
myCar.hello(); // Uncaught TypeError: myCar.hello is not a function

// setter를 만들면 값을 할당할 수 있다.
myCar.age = 32;

// getter로 값을 가져올 수 있다.
console.log(myCar.age, myCar.name); // 32 자동차
```

## constructor

생성자 그 자체, 객체를 생성하는 데 사용하는 특수한 메서드

단 하나만 존재할 수 있고 생략해도 되는 로직이면 생략해도 된다.

## Property

클래스로 인스턴스를 생성할 때 내부에 정의할 수 있는 속성값

JS 자체적으로는 ES2019의 `#`prefix를 이용한 private 프로퍼티를 제외하면 다 public이고

TS에서는 protected, private, public을 명시적으로 사용가능하다.

## getter와 setter

`get`과 `set` 선언어를 이용해서 구현한다.

getter는 무언가를 가져올 때 사용되고 setter는 무언가를 클래스 필드에 할당할 때 사용한다.

```js
// setter
set age(value) {
  this.carAge = value;
}

// getter
get age() {
  return this.carAge;
}
```

```js
class Car {
  name: string;
  constructor(name: string) {
    this.name = name;
  }

  get firstCharacter() {
    return this.name[0];
  }

  set firstCharacter(char) {
    this.name = [char, ...this.name.slice(1)].join("");
  }
}

const myCar = new Car("자동차");
myCar.firstCharacter;
myCar.firstCharacter = "차";
console.log(myCar.firstCharacter, myCar.name); 
// 차, 차동차(firstCharacter에 의해서)
```

## instance method

클래스 내부에서 선언한 메서드

실제로 JS prototype에 선언되어 프로토타입 메서드로도 불림.

```ts
class Car {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  hello() {
    console.log(`안녕하세요, ${this.name}입니다.`);
  }
}
  
const myCar = new Car("테슬라");
console.log(myCar.hello());
```

위 코드처럼 새롭게 생성한 객체에서 클래스에서 선언한 hello 인스턴스 메서드에 접근할 수 있는 것을 확인할 수 있다.

이는 프로토타입에 메서드가 선언되었기 때문이다.

[[6. Prototype]]

```js
Object.getPrototypeOf(myCar); // {constructor: f, hello: f}
myCar.__proto__ // {constructor: f, hello: f}
```

위의 두가지 방법으로 프로토타입을 확인할 수 있는데 `__proto__`의 경우 호환성을 지키기 위해서만 존재하는 기능이기 때문에 지양하는 것이 좋다.

이러한 프로토타입을 타고 최상위 객체인 Object까지 특정 프로퍼티를 찾는 과정을 프로토타입 체이닝 이라고 한다.

프로토타입 체이닝으로 인해서 클래스에 선언한 `hello()` Method를 instance에서 호출할 수 있고, 이 메서드 내부에서 `this`도 접근해 사용할 수 있게 된다.

## static method

클래스의 이름으로 호출할 수 있는 메서드

```js
class Car {
	static hello(){
	console.log('hi')
	}
}

const myCar = new Car();

myCar.hello(); // Uncaught TypeError: myCar.hello is not a function
Car.hello(); // 안녕하세요!
```

여기서의 `this`는 인스턴스가 아닌 클래스 자신을 가르키기 때문에 다른 method처럼 this를 활용할 수 없다.

react class component life cycle method인 static getDerivedStateFromProps(props, state)에서는 this.state에 접근할 수 없다.

인스턴스를 생성하지 않아도 사용할 수 있고 여러 곳에서 재사용이 가능하다는 장점이 있다. 애플리케이션 전역에서 사용하는 유틸 함수를 정적 메서드로 많이 활용한다.

## inheritance

리액트에서 클래스 컴포넌트를 만들기 위해 extends React.Component, extends React.PureComponent를 선언한다.

extends는 기존 클래스를 상속받아서 자식 클래스에서 이 상속받은 클래스를 기반으로 확장하는 개념이다.

```js
class Car {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  
  honk() {
    console.log(`${this.name}가 경적을 울린다.`);
  }
}
  
class Truck extends Car {
  constructor(name: string) {
    super(name);
  }
  load() {
    console.log("짐 싣기");
  }
}
  
const myCar = new Car("차");
myCar.honk(); // 차가 경적을 울린다.
  
const myTruck = new Truck("테슬라");
myTruck.honk(); // 테슬라가 경적을 울린다.
```

## 클래스와 함수의 관계

클래스는 ES6에 나온 개념으로 이전에는 프로토타입으로 작동 방식을 동일하게 구현했었다.

```js
class Car {
  constructor(name) {
    this.name = name;
  }
  
  honk() {
    console.log(`${this.name}이 경적을 올립니다.`);
  }
  
  static hello() {
    console.log("저는 자동차입니다.");
  }
  
  set age(value) {
    this.carAge = value;
  }
  
  get age() {
    return this.carAge;
  }
}
```

위 코드를 바벨로 트랜스파일하면 다음 코드로 정리할 수 있다.

```js
var Car = (function () {
  function Car(name) {
    this.name = name;
  }

  // prototype method
  Car.prototype.honk = function () {
    console.log(`${this.name}이 경적을 울립니다!`);
  };

  // static method
  Car.hello = function () {
    console.log("저는 자동차입니다.");
  };

  // Car 객체에 속성을 직접 정의했다.
  Object.defineProperty(Car, "age", {
    get: function () { // get과 set은 예약어이다. 접근잦, 설정자로 사용할 수 있다.
      return this.carAge;
    },
    set: function (value) {
      this.carAge = value;
    },
  });

  return Car;
})();
```

## 클로저

함수 컴포넌트에 대한 이해는 클로저에 달려 있다.

함수 컴포넌트의 구조와 작동 방식, 훅의 원리, 의존성 배열 등 함수 컴포넌트의 대부분의 기술이 모두 클로저에 의존하고 있다.

## 개념

> 함수와 함수가 선언된 어휘적 환경(Lexical Scope)의 조합 

```js
function add() {
  const a = 10;
  function innerAdd() {
    const b = 20;
    console.log(a + b); // 외부의 a와 내부의 b를 결합하여 출력
  }
  innerAdd();
}
  
add();
```

innerAdd가 add 안에 선언되었기 때문에 a에 접근할 수 있게 되었다.

선언된 어휘적 환경이란 변수가 코드 내부에서 어디서 선언됐는지를 말하는 것이다.

`this`와 다르게 코드가 작성된 순간에 정적으로 결정된다.

## 스코프

변수의 유효 범위

## Global Scope

전역 레벨에서 선언하는 것

```js
var global = 'global scope';

function hello(){
	console.log(global)
}

console.log(global) // global scope
hello()
console.log(global === window.global) // true
```
## Function Scope

JS는 기본적으로 함수 레벨 스코프를 따른다.

```js
if (true){
	var global = 'global scope'
}

console.log(global) // 'global scope'
console.log(global === window.global) // true
```

global은 `{}` 안에 정의되어 있지만 밖에서도 접근이 가능한 것을 확인할 수 잇다.

```js
function hello(){
	var local = 'local variable'
	console.log(local) // local variable
}

hello()
console.log(local) // Uncaught ReferenceError: local is not defined
```

함수 단위일 경우 함수 레벨 스코프에 따라서 밖에서는 접근이 되지 않게 된다.

## 중첩 스코프일 경우

```js
var x = 10;
  
function foo() {
  var x = 100;
  console.log(x); //100
  
  function bar() {
    var x = 1000;
    console.log(x); // 1000
  }
  bar();
}
  
console.log(x); // 10
foo(); // 100 1000
```

x의 위치에 따라 값이 달라지게 된다.

## 클로저의 활용

```js
function outerFunction() {
  var x = "hello";
  function innerFunction() {
    console.log(x);
  }
  
  return innerFunction;
}
  
const innerFunction = outerFunction();
innerFunction();
```

위 코드를 보면 innerFunction 안에는 x라는 변수가 없지만 

해당 함수가 선언된 어휘적 환경, 즉 outerFunction에는 x라는 변수가 존재하며 접근할 수 있게 된다.

## JS에서의 클로저

```js
var counter = 0

function handleClick(){
	counter++
}
```

전역 스코프일 경우 counter는 누구나 수정하고 접근할 수 있다.

리액트도 이렇게 되어 있으면 누구나 망가뜨릴 수 있게 되기 때문에 리액트에서는 클로저를 사용해서 클로저 내부에서만 접근할 수 있게 하였다.

```js
function Counter() {
  var counter = 0;
  
  return {
    increase: function () {
      return counter++;
    },
    decrease: function () {
      return counter--;
    },
    counter: function () {
      console.log("counter에 접근!");
      return counter;
    },
  };
}
  
var c = Counter();
  
console.log(c.increase()); // 1
console.log(c.increase()); // 2
console.log(c.decrease()); // 1
console.log(c.counter()); // 1
```

위 코드와 같이 클로저로 접근할 수 있는 함수들을 담은 객체를 리턴함으로써 다음의 장점이 생긴다.

1. 외부에서는 변수에 직접 접근 못하게 만들 수 있다. (counter())
2. 변수를 무분별하게 수정하는 것을 제한했다. (increase(), decrease())

## 리액트에서의 클로저

## useState

```jsx
function Component(){
	const [state, setState] = useState()
	
	function handleClick(){
	// useState 호출은 위에서 끝났짖만
	// setState는 계속 내부의 최신값을 알고 있다. (prev) 
	// 클로저를 통해서 값을 가지고 있는 것이다.
	setState((prev)=>prev+1)
	}
	
	//...
}
```

외부 함수(useState)가 반환한 내부 함수(setState)는 외부 함수(useState)의 호출이 끝남음에도 자신이 선언된 외부 함수가 선언된 어휘적

환경(state가 있는 스코프)를 기억하기 때문에 state 값을 활용할 수 있게 된다.

## 클로저 주의사항

```js
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000); 
} // 5 5 5 5 5
```

클로저를 이용해서 구현한 코드인데 5만 나오게 된다. 

이유는 `var`과  JS가 함수레벨 스코프를 기본으로 갖기 때문이다.

`var`도 함수레벨 스코프기에 위 코드는 전역 스코프에서 돌아간다. 즉 `var i`는 전역 스코프에 걸려 있다.

모든 for문을 돌고 task queue에 있는 setTimeout을 실행하면 이미 5로 되어 있다. 
(전역 스코프에 있는 i는 for문을 돌리면서 올라가기 때문에)

## 해결 방법

## `let`을 사용해서 i를 블록레벨 스코프로 바꾸기

```js
for (let i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
} // 0 1 2 3 4
```

전역 스코프가 아닌 블록 안에서 `let i` 가 선언되었기 때문에 가능한 일이다.

## 클로저를 이용해서 해결

```js
for (var i = 0; i < 5; i++) {
  setTimeout(
    (function (sec) {
      return function () {
        console.log(sec);
      };
    })(i),
    i * 1000
  );
}
```

for문 내부에서 즉시 실행 익명 함수를 선언했다.

for문이 반복될 때마다 setTimeout의 콜백 함수를 가져오는 과정에서 즉시 실행 함수를 선언&호출 하게 되고 이는 실행 컨텍스트를 각 for문마다 만들게 됨. 

실행이 끝나면 리턴으로 function ()을 가져오면서 setTimeout의 콜백 함수를 채우게 된다.

콜백이 끝나도 각 for문의 function (sec)은 닫혀있지만 살아있는 형태로 sec의 값을 유지시키고 

그 결과로 클로저로 찍은 함수 function ()은 sec을 의도적으로 활용할 수 있게 된다.

![[Deep dive 1_240525_134937.jpg]]

## 클로저를 사용하는데 드는 비용

클로저는 생성될 때마다 그 선언적 환경을 기억해야 하므로 추가로 비용이 발생한다.

```js
// 일반 함수로 처리
const aButton = document.getElementById("a");
  
function heavyJob() {
  const longArr = Array.from({ length: 10000000 }, (_, i) => i + 1);
  console.log(longArr.length);
}
  
aButton.addEventListener("click", heavyJob);

// 클로저를 이용한 처리

```

