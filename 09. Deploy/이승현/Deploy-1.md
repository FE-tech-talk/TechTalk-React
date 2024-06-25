# create-next-app 없이 수동 세팅

```terminal
npm init // package.json을 만드는 cli

npm i react react-dom next // 핵심 라이브러리 설치

npm i @types/react @types/react-dom eslint eslint-config-next typescript --save-dev
// 타입 관련 라이브러리 설치
```

## tsconfig.json 작성

```json
// tsconfig.json
{
	$schema: "https://json.schemastore.org/tsconfig.json" // tsconfig의 자동완성 지원
}
```

위와 같이 schema라는 건 schemastore에서 제공하는 .eslintrc, .prettierrc 와 같이 JSON 방식으로 작성하는 라이브러리를 지원하는 도구이다.

## next.config.js 작성

Next.js 자체의 설정을 위한 파일이다. github 저장소에서 사용 중인 버전에 들어가서 사용 가능한 옵션을 확인할 수 있다.

## ESLint, Prettier 설정

eslint-config-next는 코드에 있을 잠재적인 문제를 확인하기만 하고 띄어쓰기나 줄바꿈과 같이 코드 스타일링을 정의해 주지는 않는다. `@titicaca/eslint-config-triple` 추천

.eslintignore, .prettierignore 에 .next, node_modules를 넣어서 정적 분석 대상에서 제외시키면 좋다.

# 깃허브 활용

## Github Actions

### CI (Continuous Integration)

코드의 변화를 모으고 관리하는 코드 중앙 저장소에서 여러 기여자가 기여한 코드를 지속적으로 빌드하고 테스트해 코드의 정합성을 확인하는 과정

### 구성

러너 : 액션이 실행되는 서버
액션 : 하나의 작업 단위
이벤트 : 실행을 일으키는 이벤트
	pull_request
	issues
	push
	schedule
잡 : 하나의 러너에서 실행되는 스텝의 모음
스텝 : 하나하나의 작업 (셸 명령어, 다른 액션)

name: 액션의 이름
run-name: 필수가 아닌 타이틀, github.actor로 누가 했는지 보여줌
on : 언제 액션을 실행할지 정의
jobs: 액션을 수행할 잡

aciton/checkout@v4 최신 커밋으로 checkout

calibreapp/image-actions : 이미지 압축 관리 최적화 액션
lirantal/is-website-vulnerable : 특정 웹사이트 방문, 라이브러리 취약점 존재 여부 확인
Lighthouse CI : 라이트 하우스 기반 실행

### Github Dependabot으로 보안 취약점 해결하기

#### package.json의 dependencies

##### 버전

주.부.수

주 : 기존 버전과 호환되지 않게 api가 바뀜
부 : 기능 추가
수 : 버그 수정

##### 버전들의 규칙

`react@^16.0.0` : 16 ~ 17

`react@16.0.0` : 16에서만 호환

`react@~16.0.0` : 16.0.0부터 16.1.0 미만

##### dependencies

npm 프로젝트를 운영하는 데 필요한 자신 외의 npm 라이브러리를 정의해 둔 목록

dependencies : 프로젝트에 필요한 패키지
devDependencies : 개발 단계에서 필요한 패키지
peerDependencies : 호환성으로 인해 필요한 패키지