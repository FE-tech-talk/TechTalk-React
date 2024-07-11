# XSS (Cross-Site-Scripting)

웹사이트에 Script를 삽입해 실행할 수 있는 취약점

```html
<p>글 작성</p>
<script>
	alert("XSS")// 실행 된다.
</script>
```

## dangerouslySetInnerHTML prop

```tsx
function App(){
	return <div dangerouslySetInnerHTML = {{__html: '문자열'}}
}
```

innerHTML을 특정한 내용으로 교체할 수 있는 방법, 단 들어가는 문자열에 한계가 없어서 검증이 더 필요하다.

## useRef를 활용한 직접 삽입

useRef인스턴스.current의 innerHTML 프로퍼티를 이용해서 직접 삽입할 수 있으나 이전 방식과 같은 한계가 존재한다.

## XSS 문제를 피하는 방법

제 3자가 삽입할 수 있는 HTML을 안전한 HTML코드로 한 번 치환하는 것이 확실하다.

sanitize 또는 escape라고 하는 과정인데 관련 라이브러리로는 다음 3가지가 있다.

1. DomPurify
2. sanitize-html (허용 목록을 작성하는 allow-list 방식)
3. js-xss

사용자의 컨텐츠를 보여줄 때 뿐만 아니라 저장할 때 sanitize해주면 이후에 일일이 과정을 거치지 않아도 편하게 보여줄 수 있다.

다른 이야기로 게시판 같은 사이트에서 쿼리 스트링을 가져오는 경우에도 사용자가 입력한 데이터를 가져오는 것이기 때문에 일단 의심하고 escape 처리 하는 것이 좋다.

리액트에서는 기본적으로 escape를 수행하는데 dangerouslySetInnerHTML이나 prop으로 넘겨주는 경우에는 원본이 필요하다는 판단하에 escape를 수행하지 않는다.

## 서버 컴포넌트 주의

서버 컴포넌트가 클라이언트 컴포넌트에 반환하는 props는 반드시 필요한 값으로만 제한되어야 한다.
(props의 값이 유효하지 않을 경우의 처리를 서버측에서 처리해주는 것이 효율적이다. 처리하지 않으면 서버단에서 할 수 있었던 리다이렉트를 클라이언트단에서 하게 된다.)

## `<a>` 태그의 값에 제한을 두자

a 태그의 href 속성은 value로 이동시킬 수 있게 해준다.

javascript: 을 넣게 되면 javascript: alert() 처럼 js를 실행시켜줄 수 있는데 이는 안티 패턴으로 볼 수 있다. (button과 이벤트 핸들러가 있기에)

만약 사용자가 href를 직접 넣을 수 있게 된다면 XSS 문제가 생기게 된다.

이를 분류하기 위해 href의 protocol을 http, https 로 고정시키는 함수가 필요하다.

```tsx
//...
if (["http","https"].includes(href)) isSafe=true;
//...
```

## HTTP 보안 헤더 설정하기

브라우저가 렌더링하는 내용과 관련된 보안 취약점을 미연에 방지하기 위해 브라우저와 함께 작동하는 헤더를 의미

1. Strict-Transport-Security : 모든 사이트가 HTTPS를 통해 접근해야 하며, HTTP로 접근하는 경우 https로 변경되게 한다.
2. X-XSS-Protection : XSS 취약점이 발견되면 페이지 로딩을 중단하는 현재 사파리와 구형 브라우저에서만 제공되는 기능
3. X-Frame-Options : iframe, embed, frame, object 내부에서 렌더링을 허용할지를 나타낼 수 있다.
4. Permissions-Policy : 웹사이트에서 사용할 수 있는 기능과 사용할 수 없는 기능을 명시적으로 선언하는 헤더 (geolocation, camera 등등)
5. X-Content-Type-Options : MIME (Multipurpose Internet Mail Extensions (css, json, jpg 등등)) 의 유형을 브라우저에서 임의로 변경할 수 없게 설정 (브라우저가 파일을 읽는 방식을 고정)
6. Referrer-Policy : HTTP의 Referer 헤더 (현재 요청을 보낸 주소) 에 담을 수 있는 데이터를 제한하는 규칙
7. Content-Security-Policy : 다양한 보안 위협을 막기 위해 설계된 헤더

Next, NGINX 에서도 기본적으로 위의 보안 헤더를 설정해줄 수 있다.

https://securityheaders.com에서 보안 헤더의 상황을 확인할 수 있다.

## 취약점이 있는 패키지 사용을 피하자

security.snyk.io 에 접속하면 라이브러리의 취약점을 한눈에 파악할 수 있다.

항상 개발자는 패치 수정에 민감해야 한다.

## OWASP TOP 10

Open WorldWide Application Security Project 라는 오픈 소스 웹 애플리케이션 보안 프로젝트

웹에서 발생할 수 있는 정보 노출, 악성 스크립트, 보안 취약점 등을 연구하며 주기적으로 10대 취약점을 공개한다.

이를 통해 웹사이트를 점검해보는 것이 좋다.

