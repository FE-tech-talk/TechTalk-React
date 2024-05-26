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
const bButton = document.getElementById("a");
  
function heavyJobWithClosure() {
  const longArr = Array.from({ length: 10000000 }, (_, i) => i + 1);
  return function () {
    console.log(longArr.length);
  };
}
  
const innerFunc = heavyJobWithClosure();
bButton.addEventListener("click", function () {
  innerFunc();
});
```

크롬 개발자 도구로 보면 클로저를 활용하는 쪽이 압도적으로 부정적인 영향을 미친다.

![[closer.jpg]]
40mb를 받고 시작하는 클로저의 모습을 볼 수 있다.

클로저의 기본 원리에 따라 클로저가 선언된 순간 내부 함수는 외부 함수의 선언적인 환경을 기억하고 있어야 하므로 이를 어디에서 사용하는지 여부에 관계 없이 저장해 둔다.

위 코드로 보면 longArr가 어디에 사용될지 모르기 때문에 메모리에 일단 저장하면서 메모리에 영향을 미치는 것이다.

일반 함수의 경우 클릭시 스크립트 실행이 클로저보다 길게 걸리지만 클릭과 동시에 선언, 스코프 내에서 길이를 구하는 작업까지 끝내기 때문에 메모리에 영향을 주지 않는다.

![[closer2.jpg]]
배열을 메모리에 갖고 있지 않은 일반 함수의 모습을 볼 수 있다.

클로저의 개념은 외부 함수를 기억하고 이를 내부 함수에서 가져다 쓰는 메커니즘인데 이는 성능에 영향을 미친다. 클로저에 꼭 필요한 

작업만 남겨두지 않는다면 메모리를 불필요하게 잡아먹는 결과를 야기할 수 있고, 마찬가지로 클로저 사용을 적절한 스코프로 가둬두지 

않는다면 성능에 악영향을 미친다.

## 이벤트 루프와 비동기 통신의 이해

JS는 싱글 스레드에서 작동한다. 그래서 한 번에 하나의 작업만 동기 방식으로 처리할 수 있다.

동기(synchronous) : 직렬 방식으로 작업을 처리하는 것, 요청이 시작하고 응답을 받은 후에야 다음 작업 처리

비동기(Asynchronous) : 병렬 방식으로 작업을 처리하는 것, 요청이 시작한 후 응답과 관계 없이 다음 작업 처리

사용자가 검색어를 입력해 검색을 위한 네트워크 요청이 발생한 순간에도 다른 작업을 처리할 수 있다. (비동기적 작업 방식)

리액트에서는 16 버전에 접어들면서 비동기식으로 작동하는 방법이 소개 되었다.

## 싱글 스레드 자바스크립트

과거, 프로그램을 실행하는 단위가 오직 프로세스 뿐이었다.

process : 프로그램을 구동해 프로그램의 상태가 메모리 상에서 실행되는 작업 단위

현재, 하나의 프로그램에 여러가지 작업이 필요해 졌고 더 작은 실행 단위인 thread가 탄생했다.

thread: 하나의 process에는 여러 개의 thread를 만들 수 있고, thread 끼리는 메모리를 공유할 수 있다. 여러 작업 동시 수행

JS는 기본적으로 싱글 쓰레드 이다.

## JS가 멀티 쓰레드가 아닌 이유

1. 멀티 쓰레드는 내부적으로 처리가 복잡하며 같은 자원에 대해 여러 번 수정하는 등 동시성 문제가 발생할 수 있기에 이에 대한 처리가 필요하다.
2. 각각 격리되어 있는 process와 다르게 하나의 thread가 문제가 생기면 다른 thread도 문제가 발생할 수 있다.
3. JS이 멀티 스레딩을 지원해서 동시에 여러 쓰레드가 DOM을 조작할 수 있었다면 메모리 공유로 인해 동시에 같은 자원에 접근하게 되고  이 때문에 타이밍 이슈가 발생할 수 있고 DOM 표시에 큰 문제를 야기할 수 있다.

## 싱글 쓰레드

자바스크립트 코드의 실행이 하나의 스레드에서 순차적으로 이루어진다는 것을 의미

하나의 작업이 끝나기 전까지는 뒤이은 작업이 실행되지 않는다.

JS에서는 동기식과 다르게 비동기식 함수를 선언하면 요청 > 응답 과정과 상관 없이 여러 작업을 동시에 수행한다.

이 과정은 "이벤트 루프" 라는 개념을 통해 설명할 수 있다.

## 이벤트 루프

JS 런타임 외부에서 자바스크립트의 비동기 실행을 돕기 위해 만들어진 장치

런타임 외부라 하면 V8 같은 js 런타임 엔진 같은 외부 요소들을 뜻한다.

## 호출 스택

## 동기적 코드

call stack은 JS에서 수행해야 할 코드나 함수를 순차적으로 담아두는 스택

```js
function bar(){
	console.log('bar');
}

function baz(){
	console.log('baz');
}

function foo(){
	console.log('foo');
	bar()
	baz()
}

foo()
```
![[Deep dive 1_240525_215758.jpg]]

`이벤트 루프`는 호출 스택이 비어 있는지 확인하고 비어 있지 않다면 JS엔진을 이용해 해당 코드를 실행한다.

이는 순차적으로 단일 스레드에서 일어난다. (동시에 일어날 수 없다.)

## 비동기적 코드

```js
function bar() {
  console.log("bar");
}
  
function baz() {
  console.log("baz");
}
  
function foo() {
  console.log("foo");
  setTimeout(bar(), 0);
  baz();
}
  
foo();
```

![[Deep dive 1_240526_075025.jpg]]

위 그림을 보면 setTimeout이 0초 후에 실행되는 걸로 코드에는 적혀 있지만 비동기기 때문에 보장되지 않는다는 것을 알 수 있다.

## Task Queue

선택된 큐 중 실행 가능한 가장 오래된 task를 가져와야 하기 때문에 `set` 자료구조를 사용한다.

Task queue에서 실행해야할 task는 비동기 함수의 callback function이나 event handler를 뜻한다.

이벤트 루프에는 Task queue가 최소 한개 이상 있다. 그래서 이벤트 루프의 역할은 다음과 같이 정리할 수 있다.

> 호출 스택에 실행 중인 코드가 있는지 그리고 task queue에 대기 중인 함수가 있는지 반복해서 확인
> 
> 호출 스택이 비어 있고 task queue에 작업이 대기중이라면 가장 오래된 것부터 순차적으로 호출 스택에 올린다.

 setTimeout이나 fetch같은 네트워크 요청, 이런 비동기 함수의 수행은 task queue가 할당하는 별도의 thread에서 수행된다. (브라우저나 Node.js같은 Web API에서 해당 코드가 실행되고 콜백이 task queue로 들어간다.)

## Micro Task Queue

이벤트 루프의 구성 요소중 하나이며 Task queue와 다른 task를 처리한다.

`Promis`같은걸 처리하며 task queue보다 우선권을 갖는다.

micro task queue가 비어야 task queue가 실행될 수 있다.

```js
function foo() {
  console.log("foo");
}

function bar() {
  console.log("bar");
}

function baz() {
  console.log("baz");
}

setTimeout(foo, 0);

Promise.resolve().then(bar).then(baz);
```

## Task Queue와 Micro Task Queue의 차이

| 이름             | 대표 작업                                                    |
| ---------------- | :----------------------------------------------------------- |
| task queue       | setTimeout, setInterval, setImmediate                        |
| micro task queue | process.nextTick, Promises, queueMicroTask, MutationObserver |
## 렌더링 시기

```js
for (let i = 0; i <= 100000; i++) {
  창.innerHTML = i;
} // 100000
```

동기식 코드는 모든 호출이 끝나고 렌더링을 한다.

```js
for (let i = 0; i <= 100000; i++) {
  setTimeout(() => {
    창.innerHTML = i;
  }, 0);
} // 0 1 2 3...
```

Task Queue 코드는 Task Queue에 콜백이 들어가기 전까지 대기 시간 후에 호출 스택에 하나씩 올라오면서 하나 될때마다 렌더링을 한다.

```js
for (let i = 0; i <= 100000; i++) {
  queueMicrotask(() => {
    창.innerHTML = i;
  }, 0);
} // 100000
```

Micro Task Queue 코드는 동기 코드와 같이 마지막에 렌더링이 일어난다.

```js
console.log("a");
  
setTimeout(() => {
  console.log("b");
}, 0);

Promise.resolve().then(() => {
  console.log("c");
});
  
window.requestAnimationFrame(() => { 
// 브라우저에 다음 리페인트 전에 콜백 함수 호출을 가능하게 하는 메소드
  console.log("d");
});

// a c d b
// 브라우저 렌더링은 Micro task queue와 task queue 사이에서 일어난다.
```

## React와 관련된 JS 문법

브라우저의 종류는 다양하기 때문에 모든 최신 문법을 사용할 수 없다. 

이를 해결하기 위해서 바벨이라는 도구가 등장하였고 이 도구는 최신 문법을 설정된 버전으로 트랜스파일링해주는 역할을 한다.

## 구조 분해 할당

배열 또는 객체의 값을 분해해 개별 변수에 즉시 할당하는 것

## 배열의 구조 분해 할당

```js
const array = [1,2,3,4,5]

const [first, second, third, ...arrayRest] = array;
// first 1
// second 2
// third 3
// arrayRest [4,5]
```

배열의 구조 분해 할당은 ,의 위치에 따라 값이 결정된다.

```js
const array = [1,2,3,4,5]

const [first, , , ,fifth] = array;
// first 1
// fifth 5
```

기본값을 넣을 수도 있다. (배열의 길이가 짧거나 값이 없는 경우에 사용되는 값)

```js
const array = [1,2,3,4,5]

const [first = 10, , , ,fifth] = array;
// first 1
// fifth 5

const [a=1, b=1, c=1, d=1, e=1] = [undefined, null, 0, '']
a // 1
b // null
c // 0
d // ''
e // 1
```

js에서 기본값을 사용할 수 있는 경우는 `undefined`일 때뿐이다.

```js
const array = [1, 2, 3, 4, 5];
const [first, second, third, ...arrayRest] = array;

var array2 = [1, 2, 3, 4, 5];
const first2 = array[0];
const second2 = array[0];
const third2 = array[0];
const arrayRest2 = array.slice(3);
```

배열의 구조 분해 할당은 바벨에서 위와 같이 사용된다.

## 객체의 구조 분해 할당

객체에서 값을 꺼내온 뒤 할당하는 것

배열 구조 분해 할당과는 객체 내부 이름으로 꺼내온다는 차이가 있다.

```js
const object = {
  a: 1,
  b: 2,
  c: 3,
  d: 4,
  e: 5,
};

// 기본값도 넣을 수 있다.
const { a, b, c, z = 10 } = object;
// a 1
// b 2
// c 3
// z 10
// objectRest = {d:4, e:5}

// 이름을 변경해서 할당하는 것도 가능하며
const { a:first, b:second, c:third, ...objectRest2} = object;
// first 1
// second 2
// third 3
// objectRest2 = {d:4, e:5}


const key = 'a';
//computed property key도 사용 가능하다. 대신 네이밍이 필요하다.
const {[key]: a} = object;
```

리액트 컴포넌트인 props에서 값을 꺼내올 때 사용하는 방식이다.

```js
function Sample({a,b}){
	return a+b;
}

Sample({a:3, b:5}) // 8
```

바벨에서 트랜스파일링시 번들링 크기가 굉장히 크기 때문에 ES5을 고려해야한다면 lodash.omit이나 rambda.omt 같은 라이브러리를 사용해보면 좋다.

## Spread Syntax

배열이나 객체, 문자열과 같이 순회할 수 있는 값에 대해 말 그대로 전개해 간결하게 사용할 수 있는 구문

## Array Spread Syntax

```js
const arr1 = ["a", "b"];
const arr2 = arr1;
  
arr1 === arr2; // true
  
const arr1 = ["a", "b"];
const arr2 = [...arr1];

arr1 === arr2;
// 값만 복사되고 참조는 다르기에 false
```

기존 배열에 영향을 미치지 않고 배열을 복사할 수 있다.

## Object Spread Syntax

```js
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
  
const newObj = { ...obj1, ...obj2 };
// { "a":1, "b":2, "c":3, "d":4 }

const obj3 = { c: 5 };
```

같은 프로퍼티가 있으면 뒤에 있는 객체로 덮어 씌운다.

```js
// 순서에 따라서 다른 객체가 될 수 있으니 주의해야 한다.
const newObj2 = {...newObj, ...obj3};
// { "a":1, "b":2, "c":5, "d":4 }
const newObj2 = {...obj3, ...newObj};
// { "a":1, "b":2, "c":3, "d":4 }
```

바벨에서는 아래와 같이 구현된다.

```js
// 트랜스파일링 전
var arr1 = ['a','b'];
var arr2 = [...arr1, 'c','d','e'];

// 트랜스파일링 후
var arr1 = ['a','b'];
var arr2 = [].concat(arr1, ['c','d','e'])
```

위의 경우는 간단하지만 객체 분해 할당과 같이 트랜스파일링되서 번들링 크기가 커지는 경우가 있기 때문에 

트랜스파일링이 필요할 경우 주의해야 한다.

## Object shorthand assignment

객체를 선언할 때 객체에 넣고자 하는 key, value를 가지고 있는 변수가 이미 있다면 간결하게 넣어주는 방식

```js
const a = 1;
const b = 2;

const obj = {
	a,
	b,
}

// {a:1, b:2}
```

트랜스파일 이후에도 큰 부담이 없다. `const obj = {a:a, b:b}`

## Array prototype method

## map

인수로 전달받은 배열과 똑같은 길이의 새로운 배열을 반환

```js
const arr = [1, 2, 3, 4, 5];
const doubledArr = arr.map((v) => v * 2);
```

리액트에서는 컴포넌트를 특정 배열을 기반으로 반환할 때 사용한다.

```jsx
const arr = [1, 2, 3, 4, 5];
const Elements = arr.map((item) => {
  return <Fragment key={item}>{item}</Fragment>;
});
```

## filter

인수로 받은 콜백 함수가 truthy 조건일 경우에만 해당 원소를 반환하는 메서드

```js
const arr = [1, 2, 3, 4, 5];
const evenArr = arr.filter((v) => v % 2 === 0);
```

## reduce

콜백 함수, 초깃값을 갖고 배열이나 객체, 또는 그 외의 다른 무언가를 반환

```js
const arr = [1, 2, 3, 4, 5];
const plusArr = arr.reduce((result, item) => {
  return result + item;
}, 0);
```

콜백 함수의 (reducer) 리턴값은 다음 콜백 함수의 result가 되며 이 로직대로 최종 누적값이 리턴된다.

number 뿐만 아니라 array를 object로 변환하는 것처럼 다양하게 사용할 수 있지만 직관적이지 않아서 적절하게 사용하는 것이 필요하다.

filter와 map의 조합으로 처리해도 되는 경우가 많다. (하지만 두번 순회하기에 성능은 더 안 좋다.)

```js
const arr = [1, 2, 3, 4, 5];
  
const result1 = arr.filter((item) => item % 2 === 0).map((item) => item * 100);
  
const result2 = arr.reduce((result, item) => {
  if (item % 2 === 0) {
    result.push(item * 2);
  }
  return result;
}, []);
```
## forEach

배열을 순회하면서 콜백 함수를 실행하는 method

```js
const arr = [1, 2, 3];
  
arr.forEach((item) => console.log(item));
// 1,2,3
```

반환값은 `undefined`로 의미가 없다.

프로세스 종료, 에러 리턴 이외에는 순회를 멈출 수 없다.

```js
function run() {
  const arr = [1, 2, 3];
  arr.forEach((v) => {
    console.log(v);
    if (item === 1) {
      console.log("finished");
      return;
    }
  });
}

run(); // 1 finished 2 3
```

무조건 O(n)이 실행된다는 이야기이기 때문에 최적화시 고려해야할 사항 중 하나이다.

## 삼항 조건 연산자

```js
const value = 10;
const result = value & (2 === 0) ? "even" : "odd";
// even
```

React에서는 조건부로 렌더링 하기 위해 사용한다.

```jsx
function Component({ condition }) {
  return <>{condition ? "참" : "거짓"}</>;
}
```

가독성이 안 좋기 때문에 삼항 조건 연산자를 중첩해서 사용하는 것은 지양하는 것이 좋다.

## TS

동적 언어인 JS는 실행했을 때만 에러를 확인할 수 있는데 TS는 타입 체크를 정적으로 런타임이 아닌 빌드(트랜스파일) 타임에 수행할 수 있게 해주어 코드 안정성을 늘린다.

```ts
function test(a:number, b:number){
	return a/b;
}

// tsc로 이 코드를 JS로 트랜스파일하면 다음과 같은 에러가 난다.
test('안녕하세요', '하이' )
// Argument of type 'string' iss not assignable to parameter of type 'number'
```

## React에서 TS 활용

## any 대신 unknown 사용

`any`는  TS를 포기하겠다는 것이다. 따라서 대신 `unknown`을 사용하는 것이 좋은데 이는 모든 값을 할당할 수는 있지만 바로 사용할 수는 없게 된다.

```ts
function doSomething(callback: unknown){
	callback(); // 'callback' is of type 'unknown'
}
```

아래와 같이 callback의 타입을 지정해줘야 사용할 수 있다.

```ts
function doSomething(callback: unknown){
	if (typeof callback === 'function'){
		callback()
		return
	}

	throw new Error('callback은 함수여야 합니다.')
}
```

`never`는 어떤 타입도 들어올 수 없을 때를 의미하는데

```ts
type what1 = string & number; // never
type what2 = ('hello' | 'hi' ) & 'react'; // never
```

Class Component를 선언할 때 props는 없지만 state가 존재하는 상황에서 이 빈 props, 정확히는 어떠한 props도 받아들이지 않는다는 뜻으로 사용이 가능하다.

```tsx
// key string지만 value never 가 들어가서 어떠한 값도 들어갈 수 없게 된다.
type Props = Record<string, never>;
type State = {
  counter: 0;
};

class SampleComponent extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      counter: 0,
    };
  }
  render() {
    return <>...</>;
  }
}
  
export default function App() {
  return (
    <>
      {/* OK */}
      <SampleComponent />
      {/* Type 'string' is not assignable to type 'never' */}
      <SampleComponent hello="world" />
    </>
  );
}
```

## type guard 적극 활용

## instanceof 와 typeof

`instanceof`는 지정한 인스턴스가 특정 클래스의 인스턴스인지 확인할 수 있는 연산자이다.

```ts
class UnAuthorizedError extends Error {
  constructor() {
    super();
  }
  
  get message() {
    return "인증 실패";
  }
}
  
class UnExpectedError extends Error {
  constructor() {
    super();
  }
  
  get message() {
    return "예상치 못한 에러";
  }
}
  
async function fetchSomethingg() {
  try {
    const response = await fetch("url");
    return await response.json();
  } catch (e) {
    // e는 unknown
  
    if (e instanceof UnAuthorizedError) {
      // do something
    }
  
    if (e instanceof UnExpectedError) {
      // do something
    }
  
    throw e;
  }
}
```

unknown인 e를 타입 가드를 통해 타입을 좁힘으로써 원하는 처리 내용을 추가할 수 있다.

typeof 연산자는 앞서 예제에서 볼 수 있었던 것처럼 특정 요소에 대해 자료형을 확인하는 데 사용된다.

```ts
function logging(value: string | undefined){
	if (typeof value === 'string'){
		console.log(value)
	}

	if (typeof value === 'undefined'){
		// nothing to do
		return
	}
}
```

## in

property in object로 사용되는데, 어떤 객체에 키가 존재하는지 확인하는 용도로 사용

```ts
interface Student {
  age: number;
  score: number;
}
  
interface Teacher {
  name: string;
}
  
function doSchool(person: Student | Teacher) {
  if ("age" in person) {
    person.age; // Student
    person.score;
  }
  
  if ("name" in person) {
    person.name; // Teacher
  }
}
```

## Generic

함수나 클래스 내부에서 단일 타입이 아닌 다양한 타입에 대응할 수 있도록 도와주는 도구

제네릭을 사용하면 타입만 다른 비슷한 작업을 하는 컴포넌트를 단일 제네릭 컴포넌트로 선언해 간결하게 작성할 수 있다.

```ts
function getFirstAndLast<T>(list: T[]): [T, T] {
  return [list[0], list[list.length - 1]];
}
  
const [first, last] = getFirstAndLast([1, 2, 3, 4, 5]);
  
first; // number
last; // number
  
const [first, last] = getFirstAndLast(["a", "b", "c", "d", "e"]);
  
first; // string
last; // string
```

React에서 제네릭을 사용하는 코드는 useState 같은 것들이 있다.

```tsx
function Component(){
  // state: string
  const [state, setState] = useState<string>('')
}
```

제네릭은 여러 개 사용할 수 있다.

```tsx
function multipleGeneric<First, Last>(a1: First, a2: Last): [First, Last] {
  return [a1, a2];
}
  
const [a, b] = multipleGeneric<string, boolean>("string", true);

a // string
b // boolean
```

## Index Signiture

`[typeName: type]:type`으로 키에 원하는 타입을 부여할 수 있다. 

```ts
type Hello = {
  [key: string]: string;
};

const hello: Hello = {
	hello: 'hello',
	hi: 'hi',
}

hello['hi'] // hi
hello['안녕'] //undefined
```

하지만 위 코드처럼 string같은 넓은 범위의 타입을 key type으로 주게 되면 존재하지 않는 키에 접근할 수 있게 되버릴 수 있다.

따라서 동적으로 선언시키는 것을 지양하고 객체의 타입도 좁혀주는 것이 좋다.

## Record 사용

```ts
type Hello = Record<"hello" | "hi", string>;
const hello: Hello = {
  hello: "hello",
  hi: "hi",
};
```

## 타입을 사용해서 좁힌 index signature


```ts
type  Hello = {{[key in 'hello'|'hi']: string}}
const hello: Hello = {
  hello: 'hello',
  hi: 'hi',
}
```

## mapping시 주의사항

```ts
Object.keys(hello).map((key)=>{
  const value = hello[key]
  return value
}) // Element implicity  has an 'any' type because expression of type 'string' can't be used to index type 'Hello'.
```

string이라는 타입을 Hello type 인덱스로 사용할 수 없어서 any type으로 다 설정이 될 것이라는 건데

이는 다음 코드를 보면 알 수 있다.

```ts
const result = Object.keys(hello) // string[]
```

hello의 키를 뽑은 result가 `string[]`로 지정되어 있어 string을 프로퍼티 키로 넣을 수 없다고 하는 것이다.

다음 세가지 해결방법이 존재한다.

1. as로 keyof type을 지정해주는 것
```ts
(Object.keys(hello) as Array<keyof Hello>).map((key) => {
  const value = hello[key];
  return value;
});
```

2. 이를 이용하여  함수를 만들어서 추상화할 수 있다.
```ts
// 타입 가드 함수를 만드는 방법
function keysof<T extends Object>(obj: T): Array<keyof T> {
  return Array.from(Object.keys(obj)) as Array<keyof T>;
}
// T extends Object로 obj는 Object내에서만 들어갈 수 있게 제한

keysof(hello).map((key)=>{
  const value = hello[key]
  return value
})
```

3. 가져온 키를 단언하는 방법
```ts
Object.keys(hello).map((key)=>{
  const value = hello[key as keyof Hello]
  return value
})
```

## Object.keys를 string[]을 내보내도록 만든 이유

Duck Typing: 오리처럼 걷고 헤엄치고 소리 내면 무엇이든 오리라고 부를 수 있다. (객체가 필요한 변수와 메서드만 지니고 있으면 해당 타입에 속하도록 인정)

JS는 객체의 타입에 구애 받지 않고 객체의 타입에 열려 있다. 그래서 TS도 그 특징을 맞춰 줘야 하며 모든 키가 들어올 수 있다는 가능성이 열려 있는 key에 포괄적으로 대응하기 위해 `string[]`으로 타입을 제공하는 것이다.

## TS 전환 가이드

## tsconfig.json 먼저 작성하기

```json
{
	"compilerOptions" :{
		"outDir": "./dist",
		// .ts나 .js가 만들어진 결과를 넣어두는 폴더
		"allowJs": true,
		// .js 파일을 허용하는지 여부
		"target": "es5"
		// 결과물이 될 JS 버전 지정
	},
	"include": [."/src/**/*"]
	// 트랜스파일할 JS와 TS 파일 지정
}
```

## JSDoc와 @ts-check를 활용해 점진적으로 전환하기

```js
// @ts-check

/**
* @type {string}
*/
const str = true
```

// @ts-check와 JSDoc으로 자바스크립트임에도 타입을 제한시킬 수 있다.

하지만 손이 많이 가기 때문에 ts로 바로 작업하는 편이 빠르다.

## 타입 기반 라이브러리를 위한 @types 모듈 설치

`@types` TS로 작성되지 않은 코드에 대한 타입을 제공하는 라이브러리

ex) React의 경우 @types/react, @types/react-dom 에 정의

Cannot find module 'lodash' or its corresponding type declarations 라는 오류 메시지를 import에서 보면 이 라이브러리를 설치해야 되는 상태인 것이다.

## 파일 단위로 조금씩 전환하기

상수나 유틸처럼 별도의 의존성이 없는 파일부터 시작하자

converter가 라이브러리로 존재 하기는 하지만 TS 지식 향상에서는 좋은 생각이 아니다.