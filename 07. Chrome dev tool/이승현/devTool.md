---
title: "Dev tool"
excerpt: "리액트 과거, 현재, 미래"
toc: true
toc_sticky: true
categories:
  - web
last_modified_at: 2024-06-15T18:26:00
header:
  teaser: /assets/images/Logo/reactdeepdive.jpg
---

애플리케이션에서 발생하는 버그와 디버깅 이슈는 리액트 밖에서 일어날 때도 있다.

자바스크립트 메모리, 네트워크, 소스, 실제 HTML 및 CSS 등 리액트 이외에 환경의 디버깅을 사용하려면 브라우저 개발자 도구가 필요하다.

# Chrome Dev Tool

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615101247.png)

위 사진처럼 dev tool은 해당 페이지의 다양한 정보를 보여준다.

이러한 개발자 도구는 시크릿 모드로 보기를 권장하는데

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615101714.png)

위처럼 확장 프로그램이 전역 변수를 저장시킬 수 있기 때문이다 만약 시크릿 모드라면

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615101851.png)

시크릿 모드에서 사용가능한 확장 프로그램이 없다면 확장 프로그램이 저장시키는 전역 변수는 존재하지 않게 된다.

## Element

현재 웹페이지를 구성하고 있는 HTML, CSS 등의 정보를 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615102351.png)

### Element 화면

현재 웹페이지를 구성하는 HTML을 나타낸다. 

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615102820.png)

요소를 추가하거나 삭제, 수정할 수 있고 중단점을 이용해서 변화를 일으킨 코드를 추적할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615103356.png)

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615103504.png)

해당 부분을 php를 사용했고 이와 관련된 코드가 무엇인지 확인할 수 있다.

### Element  정보

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615104728.png)

Styles: 요소와 관련된 스타일 정보 확인
Computed: 패딩, 보더, 마진, 각종 CSS 적용 결괏값을 알 수 있는 탭이다. 결과적으로 어떤 결과물인지 확인
Layout: css 그리드나 레이아웃과 관련된 정보 확인
Event listeners: 이벤트 리스너들 확인, Ancestor를 해제하면 정확히 요소에 지정된 이벤트만 볼 수 있다.
DOM Breakpoints: 중단점이 있는지 알려주는 탭
Properties: 해당 요소가 가지고 있는 모든 속성값, js의 .attributes와의 차이는 지정된 속성 뿐만 아니라 모든 속성값이 나온다는 것
Accessibility: 웹 이용에 어려움을 겪는 장애인, 노약자를 위한 스크린리더기 등이 활용하는 값 (seo시 이거 보면 될듯)

## Source

웹 애플리케이션을 불러오기 위해 실행하거나 참조된 모든 파일을 확인할 수 있다.
JS, CSS, HTML, Font 여러가지 파일 정보를 확인할 수 있다.

### 해당 페이지에서 생성된 파일 확인

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615105916.png)

inflearn 사이트로 가서 개발자도구 > ctrl+p 를 누른 모습이다.
이걸로 파일에 들어가면 실제 코드 내용을 확인할 수 있다.

프로덕션 모드일 때는 파일이 압축되어 있어 확인하기 어렵지만 개발 모드에서는 리액트 코드도 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615111323.png)

실제 코드처럼 중단점을 찍은 후에 디버깅할 수도 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615111411.png)

### 코드 내용 확인

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615112247.png)

Source 의 오른쪽에는 위와 같은 정보가 있다.

Watch : 감시하고 싶은 변수를 선언하고, 해당 변수의 정보를 확인할 수 있는 메뉴
Breakpoints : 현재 웹사이트에서 추가한 중단점 확인, 소스탭에서 추가한 모든 중단점 확인 가능
Scope : 로컬 스코프, 클로저, 전역 스코프 등 확인
Call Stack : 현재 중단점의 콜스택 확인
Global Listeners : 전역 스코프에 추가된 리스너 목록
XHR/fetch Breakpoints, DOM Breakpoints, Event Listener Breakpoints, CSP Violation Breakpoints : 다양한 종류의 중단점 확인

## Network

해당 페이지를 접속하는 순간부터 발생하는 모든 네트워크 관련 작동이 기록된다.
외부 데이터와 통신하는 정보가 다 들어가 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615113622.png)

3번째 줄에 Fetch/XHR, Doc 등으로 tag를 만들어놨는데 이걸로 네트워크 요청의 종류를 필터링할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615113806.png)

특정 네트워크의 상태를 확인할 수도 있다.

개발자 모드로 실행되면 웹 소켓을 통해 핫 리로딩 된다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615120838.png)

하단에는 페이지를 불러오는 기간 동안 발생한 request의 건수, 다운로드한 업로드 리소스의 크기를 확인할 수 있다.

모바일일 경우 리소스의 크기만큼 모바일 네트워크 비용을 지불해야 하고 속도에도 영향을 미치기 때문에 건수와 크기는 최적화할 필요가 있다.

`gzip`이나 `brotli`를 활용해서 리소스를 압축하거나 이미지 최적화를 할 필요가 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615121057.png)

왼쪽 위에 설정을 누르고 Screenshots를 누르면 웹페이지 로딩을 네트워크 요청 흐름에 따라 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615121335.png)

시간별로 웹페이지를 스크린샷처럼 찍어서 보여줄 수 있다.

### Network에서 확인해야 하는 것

1. 불필요한 요청 또는 중복되는 요청이 없는가?
2. 웹페이지 구성에 필요한 리소스 크기가 너무 크지 않은가?
3. 리소스를 불러오는 속도는 적절한가?
4. 너무 속도가 오래 걸리는 리소스는 없는가?
5. 리소스가 올바른 우선순위로 다운로드 되어 웹페이지가 자연스럽게 만들어지는가?

## Memory

메모리 관련 정보 확인, 애플리케이션에서 발생하는 메모리 누수, 속도 저하, 웹페이지 프리징 현상을 확인할 수 있는 도구이다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615122034.png)

React-dev-tool과 비슷하게 프로파일링 작업을 거쳐야 원하는 정보를 볼 수 있다.

위에서부터 다음과 같은 프로파일링 종류이다.

Heap snapshot : 메모리 상황을 사진 찍듯이 촬영할 수 있다.
Allocation instrumentation of timeline : 시간의 흐름에 따라 메모리의 변화를 볼 수 있다. 로딩이 되는 과정의 메모리 변화, 상호작용했을 때 메모리의 변화 과정 확인
Allocation sampling : 메모리 공간을 차지하고 있는 자바스크립트 함수 확인

### Select JavaScript VM instance

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615122625.png)

디버깅하고 있는 JS VM 환경을 선택해서 실제 해당 페이지가 자바스크립트 힙을 얼마나 점유하고 있는지 확인할 수 있다. 

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615122833.png)

Google의 예시이다. JS 실행에 따라서 실시간으로 크기가 변하며 이 크기만큼 브라우저에 부담을 주기 때문에 눈여겨 볼만하다.

### Heap snapshot

현재 페이지의 메모리 상태를 확인해 볼 수 있는 메모리 프로파일 도구

```jsx
const DUMMY_LIST = []

export default function App(){
	function handleClick(){
		Array.from({ length: 10_000_000 }).forEach((_,idx)=>
			DUMMY_LIST.push(Math.random()*idx);
		)
	}
	alert('complete');
	return <button onClick={handleClick}>BUG</button>
}//컴포넌트 외부에 있는 배열에 천만 개의 랜덤한 값을 push하는 컴포넌트
```

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615124114.png)

누르기 전 메모리 모습

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615124158.png)

누른 후의 메모리 모습 세번째 줄에 array에 어마무시한 크기가 들어간 것을 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615124324.png)

이런 식으로 Objects allocated between snapshot 1 and snapshot 2을 눌러주면

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615124451.png)

둘의 교집합만 나오게 되는데 array의 토글을 열면 그 안의 객체들을 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615124752.png)

위처럼 전역 객체로 지정하면 어떤 값이 들어있는지도 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615124939.png)

window의 temp1에 저장된다. shallow size와 Retained Size는 참조값을 포함하냐 안하냐의 차이 이다.

```js
function Y(){
	this.j = 5;
}
function X(){
	this.x = 3;
	this.y = new Y();
}

export default function App(){
	function handleClick(){
		instances.push(new X())
	}
	return <button onClick={handleClick}>+</button>
}
```

이때 X는 52 100, Y는 48 48의 Shallow Size, Retained Size를 갖는다.

X는 Y를 참조하고 있으나 Y는 변수만 갖고 있기 때문에 X만 참조 값을 포함해서 Retained Size가 100인 것이다.

메모리 누수가 의심될 경우 Retained Size와 Shallow Size의 차이가 큰 객체를 찾는 것이 좋다. (다수의 객체를 참조하고 있어 누수 위험이 높음)

이번엔 useCallback으로 재할당되는지 여부를 디버깅으로 확인해보자.

```jsx
function Component({ number }: { number: number }) {
  const callbackHandleClick = useCallback(() => {
    console.log(number);
  }, [number]);

  const noCallbackHandleClick = () => {
    console.log(number);
  };

  return (
    <>
      <button onClick={callbackHandleClick}>call</button>
      <button onClick={noCallbackHandleClick}>no</button>
    </>
  );
}

function App(){
	const [toggle, setToggle] = useState(false);
	//...
	return (<>
	<button onClick={()=>{setToggle(()=>!toggle)}}>재렌더링</button>
	<Component number={5} />
	</>)
}
```

내려주는 number props는 고정이지만 부모 컴포넌트인 App이 재렌더링됨에 따라 자식 컴포넌트인 Component도 재렌더링 된다. 이때 useCallback을 사용한 callbackHandleClick은 재호출되지 않는 것을 확인해보자.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615163357.png)

이전과 이후의 snap shot을 비교해보면 위와 같다. callbackHandleClick()은 없고 noCallbakHandleClick()만 존재하는 것을 확인할 수 있다.

아래 () @163297 은 익명 함수이다. (setToggle의 콜백)

```js
setToggle(function toggle(){return !toggle})
```

기명 함수로 적어주면 다음과 같이 된다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615164016.png)
### Allocation instrumentation on timeline

시간의 흐름에 따라 메모리 변화 기록

구간을 정해서 어떠한 변화가 있었는지 그 객체를 확인해볼 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615164419.png)

전역 변수로 지정도 할 수 있다.
![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615164611.png)

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615164544.png)

## Allocation sampling

시간의 흐름에 따라 발생하는 메모리 점유를 확인하는데 JS 실행 스택별로 분석할 수 있고 이 분석을 함수 단위로 한다.

다음의 코드가 있다고 하자

```jsx
const DUMMY: Array<string> = [];

function App() {
  const [toggle, setToggle] = useState(false);
  return (
  // ...
  <button
          onClick={() => {
            Array.from({ length: 3000 }).forEach(function addDummy() {
              DUMMY.push(Math.random().toString());
            });
            setToggle(function toggle(prev) {
              return !prev;
            });
          }}
        >
  )
```

버튼을 누르면 DUMMY에 3000개의 데이터가 random().toString() 을 거쳐서 들어가게 된다.

sampling을 돌리면 toString이 가장 많이 차지한 것을 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615165927.png)

맨 오른쪽 파일 경로로 소스코드에 접근할 수도 있다.

## Next.js 환경 디버깅하기

클라이언트 사이드의 메모리 누수는 디바이스에 따라서 다르고 그 영향은 해당 디바이스에만 닿지만 서버 사이드의 메모리 누수는 서버 자체에 부담을 줘서 모든 서비스에 영향을 주게 된다.

dev tool로 서버단도 디버깅이 가능하다.

### Next.js 디버그 모드

#### 디버그 모드 실행

```json
// package.json
{"scripts": { "dev" : NODE_OPTIONS='--inspect' next dev }}
```

```terminal
npm run dev

여러가지 나오고

이후 web socket 주소가 나옴
```

##### window의 경우는 다름

https://nextjs.org/docs/pages/building-your-application/configuring/debugging

cross-env를 설치한 후에
https://www.npmjs.com/package/cross-env

```json
{ "scripts": { "dev": "cross-env NODE_OPTIONS='--inspect' next dev" }}
```

이렇게 해야 실행된다.

#### 개발자 도구 열기

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615171833.png)

chrome://inspect 로 이동하면 위와 같은 화면이 나온다.

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615173042.png)

Open dedicated DevTools for Node 클릭

![](http://codefug.github.io/assets/images/2024-06-15/Pasted%20image%2020240615173118.png)

이제 해당 VM 인스턴스 (node 서버 환경) 를 활용해서 디버깅을 할 수 있다.

가상 시나리오를 세워보자. 서버에 트래픽을 발생시켜서 확인할 수 있다.

`ab` 라는 오픈소스 도구는 웹서버 성능 검사 도구로 HTTP 서버의 성능을 벤치마킹할 수 있는 도구다.

```terminal
ab -k -c 50 -n 10000 "http://127.0.0.1:3000/"
```

##### 윈도우용 ab 설치

https://stackoverflow.com/questions/49893994/apachebench-installation-on-windows-10

위 커맨드로 해당 url을 향해서 50개의 요청을 10,000회 보낼 수 있게 된다.

똑같이 서버단도 디버깅을 하면 메모리가 증가하는 것을 확인할 수 있다.

SSR에서 메모리가 누수되면 서버에서 계속 쌓여서 과부하가 오기 때문에 순수함수로 구현해야만 한다.

>
> 모던 리액트 딥다이브 스터디를 위해서 정리하고 살을 붙힌 내용입니다.
>
> 잘못된 내용이 있다면 말씀 부탁드립니다. 감사합니다.
>