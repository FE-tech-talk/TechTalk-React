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

# 리액트 배포

## Saas 서비스
### netlify

정적 웹사이트 배포 서비스

배포 관련 알림, Preview 같은 관련 추가 기능이 존재한다.

### vercel

알림, 프리뷰 배포, 실제 배포의 성공 실패 여부 등등을 확인할 수 있다.
nextjs의 api가 존재하나면 해당 api호출시 로그를 확인할 수 있다. (서버리스 함수로 이를 인식)

### DigitalOcean

AWS와 GC Platform과 비슷하게 조금 더 다양한 클라우드 컴퓨팅 서비스를 제공한다.

## Dockerize

서비스 운영에 필요한 애플리케이션을 격리해 컨테이너로 만드는 데 이용하는 소프트웨어

> 도커는 개발자가 모던 애플리케이션을 구축, 공유, 실행하는 것을 도와줄 수 있도록 설계된 플랫폼

컨테이너 단위로 앱을 패키징하고 내부에서 앱이 실행되도록 한다.

### 용어

#### image

컨테이너를 만드는 데 사용되는 템플릿 Dockerfile이 필요하다.

#### container

도커의 이미지를 실행한 상태, 독립된 공간.

#### Dockerfile

이미지 파일을 정의하는 파일 (빌드시 이미지 생성)

#### tag

이미지를 식별할 수 있는 레이블 값, name: tagName 으로 구성되어 있다. `ubuntu:latest`

#### repository

이미지를 모아두는 장소

#### registry

repository에 접근할 수 있게 해주는 서비스 `Docker Hub`

### Command

```
docker build
docker push
docker tag
docker inspect
docker run
docker ps
docker rm: docker rm {이미지명}
docker stop {이미지명}
```

GUI로도 있으나 CLI 환경에서 쓰이게 될 수 있으니 알아두는 것이 좋다.

## Docker Image Deployment

DockerHub나 다른 AWS, GCP 에서 이미지를 업로드할 수 있으나 DockerHub만 무료이고 나머지는 비용이 발생한다.