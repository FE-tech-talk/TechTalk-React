## 웹사이트와 성능

진입하는데 0~2초가 가장 전환율이 좋고
평균 리소스 요청 수는 50회 미만인 것이 좋다.
평균적으로 웹 페이지 전체를 요청하는데 15.3초가 걸린다.

구글은 Core Web Vital 이라고 하는 핵심 웹 지표를 발표하였다.

## Core Web Vital

### LCP : Largest Contentful Paint (최대 콘텐츠풀 페인트)

페이지가 처음으로 로드를 시작한 시점부터 뷰포트 내부에서 가장 큰 이미지 또는 텍스트를 렌더링하는 데 걸리는 시간

ViewPort : 현재 노출되는 화면

#### 큰 이미지와 텍스트

1. `<img>`
2. `<svg>` 내부의 `<image>`
3. poster 속성을 사용하는 `<video>`
4. url()을 통해 불러온 배경 이미지가 있는 요소
5. 텍스트와 같이 인라인 텍스트 요소를 포함하고 있는 블록 레벨 요소
	1. 이 블록 레벨 요소에는 `<p>`, `<div>` 등이 포함된다.

#### 기준 점수

2.5초 이내로 응답이 오면 좋음, 4초 이내 보통, 그 이상은 나쁨

#### 개선 방안

1. LCP에는 다운로드가 필요한 리소스보다 텍스트 노출로 채우는게 좋다.
2. 이미지 불러오는 법
	1. `<img>`: 프리로드 스캐너에 의해 발견되어 HTML 파싱을 차단하지 않고 병렬적으로 리소스 다운
	2. `<svg>`: 모든 리소스를 다 불러온 이후에 이미지를 불러옴, 프리로드 스캐너를 사용하지 않기 때문에 사용하지 않는 것이 좋다.
	3. `<video>` 의 poster : poster를 넣으면 프리로드 스캐너, 안 넣으면 실제로 로딩해서 첫번째 프레임을 넣기 때문에 LCP에 좋지 않다.
	4. `background-image` : `url():background-image`를 비롯해서 CSS에 있는 리소스는 항상 느리다. LCP에 관련해서는 중요한 리소스를 사용하지 않는 것이 좋다.

#### 조심할 사항

1. 이미지 무손실 압축
2. loading=lazy : LCP를 중요하지 않은 리소스로 만들 수 있다. 별로 중요하지 않는 이미지에만 사용하자.
3. 각종 애니메이션
4. 클라이언트에서 빌드하지 않는것, 서버에서 빌드해온 html을 프리로드 스캐너가 바로 읽어서 LCP로 가져가는 것이 이상적이다. LCP를 useEffect 같은걸로 로드하면 리액트 코드를 파싱하고 API 요청을 보내고 응답을 받는 만큼 늦어진다.
5. 중요한 리소스, 큰 리소스는 같은 origin에서 다루도록 한다.

### FID : First Input Delay (최초 입력 지연)

#### 정의

웹사이트의 반응 속도, 최초 입력 지연 (First Input Delay)
모든 입력에 대해 측정하는 것이 아니라,, 최초의 입력 하나에 대해서만 응답 지연이 얼마나 걸리는지 판단
#### 의미

##### RAIL

###### Response

사용자의 입력에 대한 반응 속도, 50ms 미만으로 이벤트를 처리할 것

###### Animation

애니메이션의 각 프레임을 10ms 이하로 생성할 것

###### Idle

유휴 시간을 극대화해 페이지가 50ms 이내에 사용자 입력에 응답하도록 할 것 

###### Load

5초 이내에 콘텐츠를 전달하고 인터랙션을 준비할 것

FID는 여기서 R 기반인 것이다.

함수 처리 완료가 아니라 실행을 기준으로 측정한다. Event Timing API을 사용하면 실행 시간을 측정할 수 있다.

#### 기준 점수

좋은 점수 : 100ms 이내
보통 : 300ms 이내
나쁨 : 그 이상

#### 개선 방법

1. 실행에 오래 걸리는 긴 작업 분리 : 메인 스레드를 사용하는 작업을 분리하거나 다른 곳으로 옮겨서 처리 (suspense, lazy 혹은 next의 dynamic)
2. 자바스크립트 코드 최소화 : 개발자 도구 > 도구 더보기 > 커버리지 에서 기록, 새로고침 하면 필요 없는 코드 확인 가능
3. 타사 자바스크립트 코드 실행의 지연 : defer, async 등으로 script의 실행을 지연

### CLS : Cumulative Layout Shift (누적 레이아웃 이동)

페이지의 생명주기동안 발생하는 모든 예기치 않은 이동에 대한 지표를 계산하는 것

적을수록 좋은 웹사이트이다.

#### 의미

초기 렌더링시의 요소의 위치에 영향을 주는 무언가 (useEffect로 컴포넌트 생성) 로 인한 레이아웃 이동이 일어나면 CLS에서 안 좋은 점수를 받는다.

뷰포트 내부의 요소만 측정하기에 요소 추가가 무조건 레이아웃 이동으로 간주되지는 않는다.

다음의 조건을 따른다.

1. 영향분율 : 레이아웃 이동이 발생한 요소의 전체 높이와 뷰포트 높이의 비율 ( 뷰포트 높이 100, 레이아웃으로 이동 10, 레이아웃 이동으로 발생한 요소의 높이 10 > 영향분율 = 0.2)
2. 거리분율 : 레이아웃 이동이 발생한 요소가 뷰포트 대비 얼마나 이동했는지 ( 레이아웃 이동 10, 전체 뷰포트 100 > 거리분율 = 0.1 )

최종 점수는 `영향분율 * 거리분율`이다.

개발자 도구의 성능에서 해당 내용을 실험할 수 있다.

#### 기준 점수

좋음 : 0.1
보통 : 0.25
나쁨 : 그 이상

#### 개선 방안

1. 삽입이 예상되는 요소를 위한 추가적인 공간 확보 : useLayoutEffect로 미리 공간 넣기 (동기적으로 작동하기 때문에 로딩이 오래걸리는 것과 같이 보일 수 있다, 스켈레톤, 서버사이드 렌더링으로 대처 (미리 HTML을 제공))
2. 폰트 로딩 최적화 (FOUT (Flash Of Unstyled Test : 기본 폰트가 보이다가 폰트가 적용), FOIT (Flash Of Invisible Text : 텍스트 없다가 폰트 로딩되고 페이지 렌더링) 같은 현상 때문에 폰트는 레이아웃 이동에 영향을 준다.)
	1. `<link rel=preload>` : 리소스를 미리 가져오기 때문에 폰트를 처리해준다.
	2. `font-family: optional` : 폰트를 불러올 수 있는 방법은 다음 5가지 이다.
		1. `auto` : 브라우저가 결정 
		2. `block` : 폰트가 로딩되기 전까지 렌더링을 중단한다.
		3. `swap` : FOUT 방식, fallback font로 먼저 렌더링 > 웹 폰트 적용
		4. `fallback` : 100ms간 텍스트 안 보이고 > fallback font로 렌더링 3초 안에 안되면 fallback으로 지속
		5. `optional` : fallback이랑 비슷한데 0.1초 내에 폰트가 다운로드돼 있거나 캐시돼 있지 않다면 fallback font를 사용한다.
3. 적절한 이미지 크기 설정 ( width:100% height:auto로 적절하지 않게 잡게 되면 CLS점수에 나쁜 영향을 준다. (이미지가 로딩되기 전까지 크게 잡아놓고 > 로딩 후 이미지를 맞춤) )
	1. width, height 지정 : aspect-ratio로 인해서 위의 상황이 발생하는 것인데 가로세율 비율을 자동으로 맞춰주는 것이다. `<img width="1600" height="900"/>`으로 설정하면 aspect-ratio가 자동으로 `1600/900` 으로 맞춰진다.
	2. `srcset` : 크기가 다른 여러 개의 이미지를 미리 준비하고 브라우저 상황에 맞게 이미지를 바꾸는 속성 `<img srcset="image-1000.jpg 1000w, image-2000.jpg 2000w"`

### TTFB : Time To First Byte (최초 바이트까지의 시간)

브라우저가 웹페이지의 첫 번째 바이트를 수신하는 데 걸리는 시간

600ms 이상 걸릴 경우 개선이 필요

SSR에서 중요한 지표 (서버에서 작업 > TTFB이기 때문)

###  FCP : First Contentful Paint (최초 콘텐츠풀 시간)

페이지가 로드되기 시작한 시점부터 페이지 콘텐츠의 일부가 화면에 렌더링될 때까지의 시간을 측정

좋음 : 1.8sec
보통 : 3.0sec
나쁨 : 이상

1. 최초 바이트까지의 시간을 개선
2. 렌더링을 가로막는 리소스 (JS, CSS) 최소화
3. Above the Fold에 대한 최적화(가장 먼저 보이는 영역)
4. 페이지 리다이렉트 최소화
5. DOM 크기 최소화 (DOM 노드는 1500개 미만, 깊이는 32단게 정도까지만, 부모 노드는 자식 노드를 60개 정도만)