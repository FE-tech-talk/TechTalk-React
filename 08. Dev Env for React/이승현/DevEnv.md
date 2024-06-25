# <mark style="background: #FF5582A6;">ESLint</mark>

## <mark style="background: #FFB86CA6;">정적 코드 분석</mark>

코드의 실행과는 별개로 코드 자체만으로 잠재적 버그를 야기하는 부분을 찾아내어 사전에 수정하는 분석을 `정적 코드 분석` 이라고 한다.

### ESLint 동작 방식

#### 1. JS를 문자열로 읽는다.
#### 2. JS를 분석하는 parser로 코드를 구조화한다.

자주 사용하는 parser로는 espree가 있다. 
   
https://astexplorer.net/
위 사이트로 어떻게 구조화되는지 확인할 수 있다. (AST explorer)

parser를 espree로 한 결과
![](Pasted%20image%2020240617191611.png)

변수, 함수, 함수명을 분석하는 것이 아니라 코드의 정확한 위치와 같은 세세한 정보를 분석해서 알려준다. 그렇게 코드의 줄바꿈, 들여쓰기 등을 파악한다.

TS도 @typescript-eslinttypescript-estree 라고 하는 espree 기반 parser가 존재한다.

![](Pasted%20image%2020240617191548.png)
위 사진을 보면 JS와 parsing된 부분은 비슷하지만 여기에 typeAnnotation이 추가된 것을 확인할 수 있다.

#### 3. 구조화한 트리를 AST(Abstract Syntax Tree)라 하고 이를 규칙과 대조한다.

이러한 규칙들의 모음을 plugins라고 한다. debugger를 개발 과정에서만 사용하게 만드는 규칙 등등 다양한 규칙이 존재한다.

debugger를 espree로 분석한 모습
```json
{
  "type": "Program",
  "start": 0,
  "end": 8,
  "body": [
    {
      "type": "DebuggerStatement",
      "start": 0,
      "end": 8
    }
  ],
  "sourceType": "module"
}
```

no-debugger 규칙
```json
// no-debugger
module.exports = {
	meta: { // 해당 규칙과 관련된 메타 정보
		type: 'problem',
		docs: {
			description: 'Disallow the use of `debugger`',
			recommended: true,
			url: 'https://eslint.org/docs/rules/no-debugger',
		},
		fixabble: null, // docs, eslint --fix로 수정했을 때 수정가능한지 여부
		schema: [],
		messages: { // 경고 문구
			unexpeccted: "Unexpected 'debugger' sstatement.",
		},
	},
	create(context){ // 문제점 확인하는 함수
		return {
			DebuggerStatement(node){
				context.report({
					node,
					messageId: 'unexpected',
				})
			},
		}
	},
}
```

#### 4. report 혹은 fix한다.

## <mark style="background: #FFB86CA6;">eslint-plugin</mark>

규칙을 모아놓은 패키지, ex) `eslint-plugin-import`, `eslint-plugin-react`

## <mark style="background: #FFB86CA6;">eslint-config</mark>

eslint-plugin을 모아놓고 한 세트로 제공하는 패키지
(여러 프로젝트에 걸쳐 동일하게 사용할 수 있는 ESLint 관련 설정을 제공하는 패키지)

### eslint-config와 eslint-plugin의 네이밍 규칙

1. eslint-plugin-* 처럼 한 단어가 뒤에 붙은 prefix가 고정인 이름을 가져야 한다. (`ex: eslint-plugin-naver`)
2. `@titicaca/eslint-config-triple` 처럼 Scope를 앞에 붙는 것은 허용된다.

보통 `eslint-config`는 개인 개발자보다는 회사에서 만든 후 배포하는 형태가 일반적이기 때문에 남의 것 쓰면 해결되는 경우가 많다.

### 유명한 eslint-config

#### eslint-config-airbnb

500여 명이 유지보수하고 있으며 가장 인기있는 config

#### @titicaca/triple-config-kit

한국 커뮤니티에서 가장 유지보수가 활발하고 많이 쓰이는 config (인터파크트리플에서 개발함.)

1. airbnb와는 다르게 자체적으로 개발하였다.
2. 외부로 제공하는 규칙에 대한 테스트 코드가 존재
3. frontend 규칙도 별도로 제공하기 때문에 Node.js 환경 또는 리액트 환경에 맞는 규칙을 적용할 수 있다.

다른 prettier, stylelint도 모노레포로 만들어서 관리하고 있어서 설치하기 용이하다.

#### eslint-config-next

Next.js로 작성된 코드라면 꼭 필요한 config이다.

## <mark style="background: #FFB86CA6;">나만의 ESLint 규칙 만들기</mark>

### 1. 기존 규칙 변경 : import React를 제거해서 번들 크기를 줄이기 (React 17부터 JSX Runtime으로 인해 사라진 문법)

npm run start로 시작한 것과 npm run build로 만든 JS파일은 동일한 크기가 되는데 이는 번들러가 처리해준 것이다. 

트리 쉐이킹으로 처리할 수는 있지만 불필요한 import React를 제거하면 트리 쉐이킹의 시간을 줄일 수 있게 되기 때문에 빌드 속도 향상에 도움이 될 것이다.

기존 규칙을 커스터마이징해서 이슈를 감지할 수 있다. `no-restricted-imports` 를 활용해보자.

```js
module.exports = {
	rules: {
		'no-restricted-imports' : [
			'error',
			{
				//paths에 금지시킬 모듈을 추가한다.
				paths:[
					{
						// 모듈명
						name: 'react',
						// 모듈 이름
						importNames: ['default'],
						// 경고 메시지
						message:
							"import React form 'react' is not needed now."
					}
				]
			}
		]
	}
}
```

```terminal
npm run lint // 린트 실행 명령어
// 린트에 맞지 않는 코드들 어느 파일 몇 줄에 있는지 확인할 수 있게 된다.
```

예상대로 잘 되는 것을 확인할 수 있을 것이다.

이와 같이 트리쉐이킹 되지 않는 라이브러리들이 import 되는 것을 방지할 수도 있다.

```js
module.exports = {
	rules: {
		'no-restricted-imports' : [
			'error',
			{
				//paths에 금지시킬 모듈을 추가한다.
				paths:[
					{
						// 모듈명
						name: 'lodash',
						// 경고 메시지
						message: 'lodash는 트리쉐이킹되지 않습니다. lodash/* 형식으로 import 해주세요.'
							
					}
				]
			}
		]
	}
}
```

### 2. 새 규칙 만들기 : new Date를 금지시키는 규칙

new Date() 는 기기의 시간을 리턴해준다. 서버의 시간만을 사용하게 하기 위해서 

espree에서 AST를 어떻게 만드는지 분석해서 새 규칙을 만들어 보자.

```js
new Date();
```

이는 AST로 

```json
{
  "type": "Program",
  "start": 0,
  "end": 11,
  "body": [
    {
      "type": "ExpressionStatement", // 해당 코드의 표현식 전체
      "start": 0,
      "end": 11,
      "expression": { // ExpressionStatement에 어떤 표현이 들어가 있는지 확인
        "type": "NewExpression", // 해당 표현이 어떤 타입인지 (NewExpression = 생성자)
        "start": 0,
        "end": 10,
        "callee": { // 생성자를 표현한 표현식에서 생성자의 이름
          "type": "Identifier",
          "start": 4,
          "end": 8,
          "name": "Date"
        },
        "arguments": [] // 생성자를 표현한 표현식에서 생성자에 전달하는 인수를 나타낸다.
      }
    }
  ],
  "sourceType": "module"
}
```

이렇게 표현된다. 이제 ESLint 규칙으로 만들어보자.

```json
module.exports = {
	meta: { // 해당 규칙과 관련된 메타 정보
		type: 'suggestion',
		docs: {
			description: 'Disallow use of the new Date()',
			recommended: false,
		},
		fixabble: 'code', // docs, eslint --fix로 수정했을 때 수정가능한지 여부
		schema: [],
		messages: { // 경고 문구
			message: "Unexpected 'debugger' sstatement.",
		},
	},
	create(context){ // 문제점 확인하는 함수
		return {
			NewExpression: function (node){ // NewExpression을 찾았을 때
				if (node.callee.name === 'Date' && node.arguments.length === 0){ // 해당 생성자가 Date인지 확인
					context.report({ // 리포트
						node: node, // 문제가 되는 노드
						messageId: 'message', // 노출하고 싶은 message (meta.messages에서 가져온다.)
						fix: function (fixer){ // fix를 이용해서 자동으로 수정한다.
							return fixer.replaceText(node, 'ServerDate()')
						}
					})
				}
			}
		}
	},
}
```

create 함수는 객체를 반환해야 하는데 여기서 코드 스멜을 감지할 선택자나 이벤트명 등을 선언할 수 있다.

이후 `yo`와 `generate-eslint`를 이용해서 eslint-plugin 형태로 배포할 수 있다.

## <mark style="background: #FFB86CA6;">주의할 점</mark>

### 1. Prettier와 충돌

Prettier는 코드의 포매팅을 도와주는 도구이다. (줄바꿈, 들여쓰기, 작은 따옴표와 큰 따옴표 등을 담당)

직접 규칙을 안 겹치게 선언하거나 JS에는 ESlint, 그외는 eslint-plugin-prettier를 사용하는 방법이 있다.

### 2. 규칙에 대한 예외 처리, 그리고 react-hooks/no-exhaustive-deps

`eslint-disable-` 주석을 통해서 특정 규칙을 임의로 제외시킨다.

```js
// 특정 줄만 제외
console.log("hw") // eslint-disable-line no-console

// 다음 줄 제외
// eslint-disable-next-line no-console
console.log("hw")

// 특정 여러 줄 제외
/*eslint-disable no-console*/
console.log("hw")
/*eslint-enable no-console*/

// 파일 전체 제외
/*eslint-disable no-console*/
console.log("hw")
```

useEffect 사용시 내부적으로 상태에 의존하고 있는데 deps에서 규칙이 위반된다고 나올 경우 어떤 경우든 문제이다. 규칙을 끌지 특정 부분만 무시할지 신중하게 점검해야 한다.

### 3. ESLint 버전 충돌

ESLint의 버전이 맞지 않을 경우 내부적으로 버전이 틀려서 찾지 못했다는 오류를 마주할 수 있다.

# <mark style="background: #FF5582A6;">React Test Library</mark>

 백엔드의 테스트 : 서버나 디비에서 원하는 데이터를 올바르게 가져올 수 있는지, 데이터 수정 간 교착 상태나 경쟁 상태가 발생하지는 않는지 데이터 손실은 없는지, 특정 상황에서 장애가 발생하지 않는지 등을 확인
 
 프론트엔드의 테스트 : 일반적인 사용자와 동일하거나 유사한 환경에서 수행

프론트엔드는 HTML, CSS 같은 디자인 요소, 사용자의 인터랙션, 브라우저에서 발생하는 다양한 시나리오 등등 테스팅하기 번거로워서 다양한 라이브러리가 존재한다.
(컴포넌트, 함수 수준의 유닛 테스트, 사용자 작동을 모두 흉내내서 테스트 등등)

그중 가장 유명한게 React Testing Library이다.

## <mark style="background: #FFB86CA6;">React Testing Library</mark>

DOM Testing Library 기반이다.

### <mark style="background: #FFF3A3A6;">DOM Testing Library</mark>

jsdom 기반이다.
jsdom은 Node.js 같은 HTML없이 js만 존재하는 환경에서도 HTML과 DOM을 사용할 수 있도록 하는 라이브러리

#### jsdom 예시
```js
const jsdom = require('jsdom')

const { JSDOM } = jsdom
const dom = new JSDOM(`<!DOCTYPE html><p>Hello world</p>`)

console.log(dom.window.document.querySelector('p').textContent) // "Hello World"
```

### <mark style="background: #FFF3A3A6;">자바스크립트 테스트의 기초</mark>

1. 테스트할 함수나 모듈을 선정
2. 함수나 모듈이 반환하길 기대하는 값 선정
3. 실제 반환 값 기입
4. 기대값과 반환값이 일치하는지 확인
5. 일치시 성공 불일치시 에러

Node.js는 assert라는 모듈을 제공한다. 하지만 무엇을 테스트했는지 무슨 테스트를 어떻게 수행했는지 실제 테스트 정보를 확인하려면 다른 라이브러리가 필요하다.

### <mark style="background: #FFF3A3A6;">Testing Framework</mark>

어설션을 기반으로 테스트 + 도움이 될만한 정보 전달

Jest, Mocha, Karma, Jasmine 등등이 있다.

#### <mark style="background: #BBFABBA6;">Jest</mark>

Jest의 실제 동작 과정을 보자.
```js
// math.js
function sum(a,b){
	return a+b
}

module.exports = {
	sum,
}
```

```js
// math.test.js
const {sum} = require('./math')

test('두 인수가 덧셈이 되어야 한다.', ()=>{
	expect(sum(1,2)).toBe(3) // 통과
})

test('두 인수가 덧셈이 되어야 한다.', ()=>{
	expect(sum(2,2)).toBe(3) // 에러
})
```

```terminal
npm run test

> jest
```

#### 리액트 컴포넌트 테스트 코드 작성하기

1. 컴포넌트 렌더링
2. 필요에 의해 컴포넌트에서 액션 수행
3. 기대값과 실제값 비교

##### 프로젝트 생성

cra의 경우 기본 장착 되있긴 함.

```terminal
npx create-react-app react-test --template typescript
```

App.test.tsx 파일 생성 확인

```js
import React from 'react'
import {render, screen} from '@testing-library/react'
import App from './App'

test('renders learn react link',()=> {
	render(<App />) // App 렌더링
	const linkElement = screen.getByText(/learn react/i) // learn react 문자열을 가진 DOM 요소 찾음.
	expect(linkElement).toBeInTheDocument() // DOM 요소가 document 내부에 있는지 확인
})
```

특정 무언가를 가진 HTML 요소를 가져오는 함수
1. getBy... : 인수의 조건에 맞는 요소 반환, 실패시 에러
2. getAllBy...: 인수의 조건에 맞는 복수 개의 요소 반환, 실패시 에러
3. findBy ... : Promise를 반환하는 getBy, 기본값 1000ms 타임 아웃, 비동기로 찾는다는 것 (비동기 액션 이후 요소를 찾을 때 사용), 실패시 에러
4. findAllBy... : findBy의 복수형, 실패시 에러
5. queryBy... : getBy와 같은데 실패시 null 반환
6. queryAllBy... : query의 복수형, 실패시 null

##### 정적 컴포넌트

별도의 상태가 존재하지 않아 항상 같은 결과를 반환하는 컴포넌트
이를 테스트하기 위해서는 렌더링 후에 테스트를 원하는 요소를 찾아 테스트를 수행하면 된다.

책에 있는 코드들을 확인하면 다음의 메소드들을 정리해볼 수 있다.

`beforeEach` : 각 테스트를 수행하기 전에 실행하는 함수

`describe` : 비슷한 속성을 가진 테스트를 하나의 그룹으로 묶는 역할

`it` : test의 alias이다.

`getByTestId` : testid는 리액트 테스팅 라이브러리의 예약어이다. get 등의 선택자로 선택하기 어렵거나 곤란한 요소를 선택하기 위해 사용할 수 있다. DOM 요소의 dataset(비표준 attribute)을 이용해서 사용하며 data-testid=~ 로하면 getByTestId, findByTestId 등으로 선택할 수 있다.

```html
<p data-testid = "something">무언가</p>
```

이외에도 `toHaveAttribute`, `toBeVisible` 등으로 직관적인 메소드들이 많다.

##### 동적 컴포넌트

아무런 상태값이 없는 순수한 컴포넌트는 테스트하기 간편하다. (stateless component)
하지만 상태값이 있는 컴포넌트, useState를 사용해 상태값을 관리하는 컴포넌트의 경우 다르게 접근해야 한다.
###### useState를 통해 사용자가 입력을 변경하는 컴포넌트

Testing Library에서는 입력을 흉내 내고, 또 state의 변화에 따른 컴포넌트의 변화를 테스트한다.
(책 참고, userEvent.type 같은 다양한 동작관련 메소드 들이 있다.)

##### 비동기 이벤트가 발생하는 컴포넌트

fetch처럼 복잡한 함수의 경우(header 설정, response 종류 등등) 모킹이 어려워서 테스트에 모든 시나리오를 적용하기 어렵다. 

이를 해결하기 위해 실제 네트워크를 가로채고 응답을 보내는 라이브러리인 MSW를 사용한다.

fetch를 이용한 비동기 컴포넌트를 사용할 때의 중점은 fetch 응답 모킹, findBy를 활용한 비동기 요청이 끝난 후 렌더링 비교이다.

#### 사용자 정의 훅 테스트

`react-hooks-testing-library` 를 활용해서 사용자 정의 훅을 테스트한다.

#### 테스트를 작성하기에 앞서 고려해야 할 점

##### 테스트 커버리지

소프트웨어가 얼마나 테스트됐는지를 나타내는 지표

얼마나 많은 코드가 테스트되고 있는지를 나타내는 지표이지만 잘되고 있는지 여부를 판별할 수는 없다.

##### 테스트를 하는 판단 조건

실무에서는 QA에 의존하는 경우가 많다. 여유롭게 테스트 코드를 작성하고 운영하기가 쉽지 않다.

테스트 코드를 작성하기 전에 최우선적으로 애플리케이션에서 가장 취약하거나 중요한 부분을 파악하는 것이 중요하다.

ex) 결제 파트

테스트 코드는 단순 코드 작성만으로는 쉽게 이룰 수 없는 목표인 소프트웨어 품질에 대한 확신을 얻기 위해 작성하는 것이다.

##### 테스팅의 종류

Unit Test : 각각의 코드나 컴포넌트가 독립적으로 분리된 환경에서 의도된 대로 정확히 작동하는지 검증하는 테스트

Integration Test : 유닛 테스트를 통과한 여러 컴포넌트가 묶여서 하나의 기능으로 정상적으로 작동하는지 확인하는 테스트

End to End Test : E2E 테스트라 하며, 실제 사용자처럼 작동하는 로봇을 활용해 애플리케이션의 전체적인 기능을 확인하는 테스트