# 리액트 개발 도구

react-dev-tools

리액트 앱 개발을 위해서 리액트 팀이 만든 개발 도구

![](http://codefug.github.io/assets/images/2024-06-10/Pasted image 20240610161243.png)

## 활용

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610161637.png)

리액트 개발자 툴을 다운로드하면 위와 같은 두가지 선택지가 더 생긴다.

하나는 Component, 하나는 Profiler이다.

### 컴포넌트

Componet에서는 React App의 컴포넌트 트리를 확인할 수 있다. props, hooks 등 다양한 정보를 확인 가능하다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610161514.png)

개발자 도구와 같이 Hover했을 때 해당 컴포넌트의 정보를 확인할 수 있다. 
(개발자 도구와 다른 버튼으로 해야한다. 바로 아래 있음.)

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610161856.png)

#### 컴포넌트 트리

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610162302.png)

리액트 앱은 트리 구조로 이루어져 있는데 이 트리를 리액트 개발자 도구에서 확인할 수 있다.

여기서 Anonymous라는 노드가 있는데 이는 익명 컴포넌트를 뜻한다.

함수 선언식이나 함수 표현식으로 생성한 컴포넌트는 정상적으로 함수명을 표시하지만 그렇지 않을 경우 다음의 로직을 따른다.

1. default export의 경우 `_default`로 처리된다. (16.8기준, 9부터는 임의의 값으로 추론)
2. memo로 감싼 익명 컴포넌트의 경우 익명 처리된다.
3. 고차 컴포넌트는 명칭을 추론하지 못한다. (16.8기준 9부터는 임의의 값으로 추론)

16.9부터 해결되는 부분이 있긴 하지만 `_c5` 같은 임의의 값이기 때문에 기명 함수를 사용하는 것이 좋다.

혹은 displayName을 적용하는 방법이 있다.

고차 컴포넌트가 특히 효율적이다. (dynamic하게 displayName을 설정할 수 있다.)

```jsx
function withHigherOrderComponent(WrappedComponent){
	class WithHigherOrderComponent extends React.Component {
	}

	WithHigherOrderComponent.dissplayName = `WithHigherOrderComponent(${getDisplayName(WrappedComponent)})`;

	return WithHigherOrderComponent
}

function getDisplayName(WrappedddComponent){
	return WrappedComponent.displayName || WrappedComponent.name || 'Component'
}
```

사용되어지는 컴포넌트의 displayName 혹은 name 혹은 'Component'를 반환하여 고차 컴포넌트에 담을 수 있고 이는 담아지는 컴포넌트와 고차 컴포넌트를 분리할 수 있게 되서 개발 모드에서 편해진다.

terser 등의 압축 도구 같은 것들이 컴포넌트 명을 난수화할 수 있고 Component.displayName의 경우 빌드 도구가 삭제할 가능성도 있어서 개발 모드에서만 사용하는 것이 좋다.

#### 컴포넌트명과 props

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610164323.png)

컴포넌트를 선택했을 때 해당 컴포넌트에 대한 자세한 정보를 보여주는 영역이다.

##### 컴포넌트명과 key

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610164615.png)

누가 봐도 컴포넌트들을 mapping해서 구현했을 것 같은 부분을 가져왔다. 역시나였다.

특이한 점은 key값이 드러나있다는 것이다. 리액트에서 key값은 props로도 전달되지 않으며 파이버 트리에서 스위칭할 때 비교를 위해서만 쓰는 줄 알았지만 이렇게 개발자 도구에도 나타나 있었다.

위에서부터 보면 Anonymous라는 컴포넌트명을 가졌고 각 부분들에 key값이 나와있는 것을 확인할 수 있다.

여기서 Anonymous가 빨간색인 이유는 strict mode로 렌더링되지 않았다는 것을 의미한다.

##### 컴포넌트 도구

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610164952.png)

아까 props부분에서 오른쪽 세 개의 아이콘을 확인해보자.

###### Inspect the matching DOM element

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610165421.png)

해당 컴포넌트가 DOM element에서는 어떻게 렌더링 됐는지 확인

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610165908.png)

###### View source for this element

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610165510.png)

콘솔 창에 해당 컴포넌트의 정보를 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610170039.png)

해당 컴포넌트의 props, hooks, nodes 를 볼 수 있다.

###### View source for this element

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610165548.png)

해당 컴포넌트의 소스코드를 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610170606.png)

소스코드는 프로덕션 모드에서 빌드되어 최대로 압축되어 있다. `{}` 버튼을 누르면 그나마 볼 수 있게 나온다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610170629.png)

##### 컴포넌트 props

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610200919.png)

###### Copy value to clipboard

props의 값을 클립보드에 복사한다.

###### Store as global variable

`window.$r`에 해당 정보를 담는다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610201128.png)

위 사진처럼 내가 복사한 걸 임의의 변수명 ($reactTemp0)에 담아서 콘솔에서 사용할 수 있게 된다.

###### Go to definition

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610201250.png)

값이 함수일 경우 소스코드로 이동하는 Go to definition이 존재한다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610201359.png)
(해당 함수가 있는 소스코드로 이동한 모습)

##### 컴포넌트 hooks

사용 중인 훅 정보를 확인할 수 있다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610201925.png)

State, Reducer, Context, Memo, Callback, Ref, id, LayoutEffect, Effect 등이 존재한다.

훅에 넘겨주는 함수를 익명 함수 대신 기명 함수로 써야 하는 이유도 여기에 있다.

익명일 경우 `f() {}` 로 나타나고, 기명일 경우 `f 이름 () {}` 이 된다. 기명 함수를 넘기면 더 정확하게 확인할 수 있는 것이다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610202656.png)

코드로 기명함수를 표현하면 다음과 같다.

```jsx
useEffect(()=>{}) // xxx
useEffection(function 함수이름(){}) // ooo
```

##### 컴포넌트를 렌더링한 주체, rendered by

프로덕션에서는 `react-dom`이라고만 나오지만 개발환경에서는 다르다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610204025.png)

react-dom > hydrateRoot > App > BoardsPage 이렇게 렌더링 되었음을 알 수 있다.

### Profiler

리액트가 렌더링하는 과정에서 발생하는 상황을 확인하기 위한 도구

어떤 컴포넌트가 렌더링되었고 몇 차례 일어났으며 어떤 작업이 오래 걸렸는지 확인할 수 있다!!

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610204442.png)
<s>자랑스러운 내 과제를 꺼내왔다</s>
위처럼 렌더링이 얼마나 걸렸고 각 부분들은 어떤 것 때문에 업데이트 되었는지 확인할 수 있다.

#### 설정

##### General

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610204708.png)

설정 창에서 저 Highlight updates when components render를 키면 렌더링 시 하이라이트 표시를 할 수 있다.

{% include video id="955968361" provider="vimeo" %}

##### Debugging

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610205103.png)

리액트 엄격모드에서는 순수 컴포넌트인지 확인하기 위해서 개발 환경때 렌더링을 무조건 두번씩 하는데

`Hide logs during second render in Strict Mode` 옵션을 키면 한번만 렌더링된다.

##### Profiler

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610205318.png)

`Record why each component rendered while profiling` 으로 각 렌더링마다 각 컴포넌트의 렌더링 이유를 기록한다.

#### 프로파일링

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610205813.png)

왼쪽부터 순서대로 아이콘의 설명을 해보겠다.

##### 활용

###### `Start Profiling` 

프로파일링이 시작되며 한번 더 누르면 프로파일링이 끝나게 된다.
###### `Reload and Start profiling` 

웹페이지가 새로고침되면서 이와 동시에 프로파일링이 시작된다.
###### `Stop Profiling` 

현재 내용을 모두 지운다.
###### `Load profile` 

프로파일링 결과를 불러온다.
###### `Save profile` 

프로파일링 결과를 JSON 파일 형식으로 다운로드한다.

{% include video id="955968403" provider="vimeo" %}

(Reload and Start profiling을 이용해서 프로파일링한 모습)

###### `Flamegraph`

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610210623.png)

렌더 커밋별로 어떠한 작업이 일어났는지 나타낸다. 즉 리액트 컴포넌트 트리 구조를 확인할 수 있다.

너비가 넓을수록 오래 걸렸다는 것이고 당연하게도 가장 넓은건 루트 컴포넌트이다.

초록색일수록 빠르게, 노란색일수록 느리게 렌더링된 컴포넌트이며 회색은 아예 렌더링되지 않은 컴포넌트이다.
![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610214552.png)
![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610211527.png)

###### `Ranked`

렌더링하는데 오래걸린만큼 위에 있는 순서로 나열

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610212134.png)

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610212117.png)

###### `Timeline`

시간이 지남에 따라 컴포넌트에서 어떤 일이 일어났는지 확인(React 18 이상만 가능)

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610212643.png)

시간 단위로 무엇이 언제 렌더링되었고 유휴 시간은 어느정도인지 확인할 수 있다.

##### 실제 적용

실제 실습을 통해서 프로파일링을 통한 디버깅 해보겠다.

{% include video id="955968438" provider="vimeo" %}

위 영상에서 본 것처럼 이미지를 하나 업로드해서 preview를 보여줄 경우, addboard라는 큰 컴포넌트가 렌더링되는 것을 알 수 있다.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610214837.png)

이를 컴포넌트 탭에서 누가 렌더링했는지 확인하면

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610215256.png)

14 State의 변경 때문이라고 나온다. 14 State를 component tab에서 확인해보자.

![](http://codefug.github.io/assets/images/2024-06-10/Pasted%20image%2020240610215205.png)

코드로 보면 14 State의 정체가 preview임을 알 수 있다.

```tsx
export default function Addboard() {
  const {
    register,
    formState: { errors, isDirty, isValid },
    handleSubmit,
  } = useForm<IFormInput>();
  const onSubmit: SubmitHandler<IFormInput> = (d) => {
    postArticle({ ...d, image: preview });
  };
  const { isDesktop } = useScreenDetector();
  const [preview, setPreview] = useState("");
// ...
```

preview라는 부모 state가 바뀜에 따라 부모 컴포넌트 리렌더링 > 자식 컴포넌트 리렌더링 이런 순서로 리렌더링된 것을 디버깅으로 확인할 수 있었다.

이를 해결하기 위해서는 자식 컴포넌트 내에서 preview를 바꾼 후에 부모 컴포넌트에 preview를 올려주면 되는데 자식 컴포넌트에서 부모 컴포넌트로 preview를 올리는 로직은 단방향 데이터 흐름에 반하는 일이기 때문에 로직을 이해하는 선에서 마무리하고 디버깅을 종료하였다.

## 출처

<a src="https://wikibook.co.kr/react-deep-dive">React Deep Dive. 김용찬, chapter 6</a>