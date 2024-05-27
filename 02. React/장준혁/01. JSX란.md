## 1. JSX란?

JSX는 리액트에 종속되는 구문이 아닌 XML과 유사한 내장형 구문으로 독자적인 문법으로 보는 것이 옳다.
하지만 표준 코드가 아닌 메타에서 임의로 만든 새로운 문법이기 때문에 트랜스파일러를 거쳐야한다.

### JSX의 정의

JSXElement, JSXAttributes, JSXChildren, JSXStrings 4가지 컴포넌트 기반으로 구성돼 있다.

1. JSX Element: HTML의 요소(element)와 비슷한 역할

   - JSXOpeningElemnt <JSXElement Attribute(opt)>
   - JSXClosingElement </JSXElement>
   - JSXSelfClosingElement <JSXElement Attribute(opt) />
   - JSXFragment <></>

   1. JSXElementName: JSX Element의 요소 이름으로 쓸 수 있는 것
      - JSXIdentifier: 내부에서 사용할 수 있는 식별자, 자바스크립트 식별자 규칙과 동일 ($, \_ 이외 다른 특수문자로 불가능, 숫자 시작 불가능)
      - JSXNamespaceName: :로 최대 한개를 묶어 식별자로 사용할 수 있다.
      - JSXMemberExpression .로 여러개를 묶어 식별자로 사용할 수 있다.

```js
// JSXIdentifier 예제
// 가능
function Valid() {
    return <$></$>
}

// 불가능
function Invalid() {
    return <1></1>
}
```

2. JSXAttributes: JSXElement에 부여할 수 있는 속성을 의미

   - JSXSpreadAttribute: 자바스크립트의 전개연산자처럼 사용할 수 있다. (ex. {...AssignExpression})
   - JSXAttribute: 속성을 나타내는 키와 값으로 표현 가능, JSXAttributeName은 키 값, JSXAttributeValue는 키에 할당할 수 있는 값.
   - JSXAttributeValue는 큰 따옴표, 작은 따옴표, 자바스크립트 값을 할당하는 { value }, JSX요소 자체를 사용할 수 있다.

```js
// JSXAttributeName 예제
function Valid() {
  return <DropDown.Title canuse:key="value"></DropDown.Title>;
}
```

3. JSXChildren: JSXElement의 자식 값

   - JSXChild: 기본 단위로 0개 이상 가질 수 있다
     - JSXText: {, <, >, }를 제외한 문자열
     - JSXElement: 값으로 다른 JSX 요소가 들어갈 수 있다
     - JSXFragment: <></>를 사용할 수 있다.
     - { JSXChildExpression (optional) }: JSXChildExpression은 자바스크립트의 AssignmentExpression이다.

4. JSXStrings: HTML에서 사용 가능한 문자열은 모두 JSXStrings에서도 가능

### JSX 예제

```js
// 하나의 요소
const ComponentA = <A>Hello</A>;
// 자식 없이 SelfClosingTag
const ComponentB = <A />;
// 옵션을 { }와 전개 연산자로 넣을 수 있다.
const ComponentC = <A {...{ required: true }} />;
// 옵션의 값으로 JSXElement를 넣을 수 있다.
const ComponentD = (
  <A>
    <B optionalChild={<>Hi</>} />
  </A>
);
```

### JSX는 어떻게 자바스크립트에서 변환될까?

React.createElement()를 이용하고 파라미터로 태그, 속성와 값, 콘텐트를 주어 자바스크립트로 변환한다.
리액트 17, 바벨 7.9.0 이후 버전에서 추가된 자동 런타임으로 (0, \_jsxRuntime.jsx)을 이용한다.
