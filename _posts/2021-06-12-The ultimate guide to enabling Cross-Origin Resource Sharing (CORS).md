---
layout: post
title: The ultimate guide to enabling Cross-Origin Resource Sharing (CORS) (번역)
---

다음과 같은 경우를 생각해보겠습니다: 여러분이 <mark>fetch()</mark>를 사용하여 여러분의 웹사이트에서 사용할 데이터를 API로부터 가져오려고 하지만 에러가 발생하게 됩니다. 

콘솔창을 열었더니 여러분이 서버에 보낸 요청이 CORS 정책에 의해 막혔음을 알리는 "No <mark>Access-Control-Allow-Origin</mark> header is present on the requested resource" 혹은 "The <mark>Access-Control-Allow-Origin</mark> header has a value <mark>(some_url)</mark>that is not equal to the supplied origin" 메세지가 빨간 글씨로 나타났습니다. 

![CORS-1](/assets/CORS-1.png)

익숙해 보이죠? 스택오버플로우에 올라온 <mark>cors</mark>태그가 달린 질문이 10,000개가 넘을 정도로 이 문제는 프론트엔드 개발자와 백엔드 개발자를 괴롭히는 가장 일반적인 문제중 하나입니다. 그래서, 정확히 CORS 정책은 무엇이며 왜 우리는 이러한 에러를 자주 접하는걸까요?

What is Cross-Origin Resource Sharing (CORS)?
=====================================================

흥미롭게도, 이것은 위에서 보여드렸던대로 에러라기 보다는 예상했던 동작입니다. 우리가 사용하는 웹 브라우저들은 리소스를 다른 출처들(origins)과 공유하는것을 제한하는 **동일 출처 정책(same-origin policy)**을 시행합니다. [교차 출처 리소스 공유(Cross-origin resource sharing)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) 혹은 CORS는 이 장벽을 극복할 수 있는 메커니즘 입니다. CORS를 이해하기 위해서, 먼저 동일 출처 정책을 이해하고 왜 필요한지 알아봅시다.

The same-origin policy
=============================

간단하게 말하자면, 동일 출처 정책은 브라우저에 포함된 "낯선 사람에게 말하지 마시오"의 웹 버전입니다. 

오늘날의 웹 브라우저들은 하나의 출처에서 <mark>XMLHttpRequest</mark>와 <mark>fetch</mark>가 다른 출처 리소스와의 상호작용을 제한하는 동일 출처 정책을 따르고 있습니다. 그런데 출처(origin)란 정확히 무엇일까요?

출처는 스키마, 도메인 그리고 포트의 조합입니다. 스키마는 HTTP, HTTPS, FTP 혹은 그 외 기타가 될 수 있으며 비슷하게 포트는 유효한 포트 숫자가 될 수 있습니다. 동일 출처 정책은 기본적으로 이러한 스키마, 도메인 그리고 포트가 일치하는 요청들(requests)입니다. 아래의 예제를 한번 볼까요.

출처를 <mark>http://localhost:3000</mark>으로 가정을 해봅시다. 요청들은 다음과 같이 동일 출처 혹은 교차 출처 요청으로 분류될 수 있습니다.

![CORS-2](/assets/CORS-2.png)

이것이 왜 작동되고 있는 여러분의 단일 페이지 어플리케이션(SPA)인 <mark>http://localhost:3000</mark>에서 서버인 <mark>http://localhost:5000</mark> 혹은 그 외의 포트에 API 호출을 할 수 없는 이유입니다.

또한, 출처 <mark>https://mywebsite.com</mark>에서 출처 <mark>https://api.mywebsite.com</mark>으로의 요청도 교차 사이트 요청(cross-site requests)으로 간주되고 있습니다. 두번째 출처가 하위 도메인(subdomain)인데도 말이죠. 

동일 출처 정책으로 브라우저는 클라이언트에게 공유 되어지는 교차 출처 요청으로부터 응답을 자동적으로 방지합니다. 이것은 보안상의 이유로 좋습니다! 하지만 모든 웹사이트가 이렇게 악의적인 것은 아니며 다른 출처에서 데이터를 가져와야 하는 여러 시나리오가 있습니다. 특히, 다양한 어플리케이션이 서로 다른 출처에서 호스팅 되는 현대 [microservice architecture](https://blog.logrocket.com/methods-for-microservice-communication/)시대에는 더욱 그렇습니다.

이 내용은 CORS를 깊이 살펴보고 교차 출처 요청을 허용하기 위해 CORS 사용법을 배울 수 있는 좋은 단게 입니다. 

Allowing cross-site requests with CORS
============================================

브라우저가 다른 출처와 리소스 공유하는 것을 허락하지 않는다는것을 배웠지만 리소스 공유를 할 수 있는 예시들은 수 없이 많습니다. 어떻게 가능할까요?
이 부분이 CORS가 등장하는 곳입니다.

CORS는 다른 출처와의 리소스 공유를 가능하게 해주는 HTTP 헤더 기반의 프로토콜입니다. HTTP 헤더와 함께, CORS 또한 간단하지 않은(non-simple) 요청을 위해  <mark>OPTIONS</mark>메소드를 사용하는 브라우저의 사전 전달(pre-flight) 요청에 의존합니다. 간단한 요청과 사전 전달 요청에 관해 나중에 더 자세히 설명하겠습니다.

HTTP 헤더는 CORS 메커니즘에서 있어 중요한 부분이기 때문에, 이러한 헤더들을 살펴보고 각각 무엇을 의미하는지 알아보겠습니다.

<mark>Access-Control-Allow-Origin</mark>
============================================

<mark>Access-Control-Allow-Origin</mark> 응답 헤더(Access-Control-Allow-Origin response header)는 아마도 CORS 메커니즘에 의해 설정된 가장 중요한 HTTP 헤더일 것입니다. 이 헤더의 값은 리소스들에게 접근할 수 있는 출처들로 구성되어있습니다. 
만약 이 헤더가 응답 헤더에 없다면, CORS가 서버에 설정되어있지 않다는 뜻입니다. 

헤더가 존재한다면, 해당 헤더는 요청 헤더의 <mark>Origin</mark> 헤더와 값을 확인합니다. 만약 값이 일치한다면, 요청은 성공적으로 수행될 것이며 리소스는 공유됩니다. 일치하지 않는다면, 브라우저는 CORS 에러로 응답합니다. 

공공 API의 경우와 같이 모든 출처의 리소스 접근을 허락해야 하는 경우, <mark>Access-Control-Allow-Origin</mark> 헤더는 서버에서 <mark>*</mark>로 설정할 수 있습니다. 
리소스의 접근을 특정 출처로 제한하려면 헤더는 <mark>https://mywebsite.com</mark>와 같은 클라이언트 출처의 완전한 도메인으로 설정될 수 있습니다.

<mark>Access-Control-Allow-Methods</mark>
===========================================

<mark>Access-Control-Allow-Methods</mark> 응답 헤더(Access-Control-Allow-Methods response header)는 사용이 가능한 HTTP 메소드나 <mark>GET</mark>, <mark>POST</mark>, 그리고 <mark>PUT</mark>과 같이 서버가 응답할 수 있는 HTTP 메소드들의 목록을 명시하기 위해 사용합니다. 

이 헤더는 사전 전달된(pre-flighted)요청에 보내는 응답(response)안에 존재합니다. 만약 여러분이 요청으로 보낸 HTTP 메소드가 허용한 메소드들의 목록이 없을 경우, CORS 에러를 보냅니다. 
이 방법은 유저가 <mark>POST</mark>, <mark>PUT</mark>, <mark>PATCH</mark> 혹은 <mark>DELETE</mark> 요청을 통해 데이터 수정하는 것을 제한하고 싶을 때 굉장히 유용합니다.

<mark>Access-Control-Allow-Headers</mark>
===========================================

<mark>Access-Control-Allow-Headers</mark> 응답 헤더는 여러분의 요청이 가질 수 있는 허용된 HTTP 헤더의 목록을 나타냅니다. <mark>x-auth-token</mark>과 같은 커스텀 헤더들(custom headers)을 지원하기 위해, 서버에서 CORS를 적절하게 설정할 수 있습니다.

허용된 헤더가 아닌 다른 헤더로 이루어진 요청들은 CORS 에러를 발생시킵니다. <mark>Access-Control-Allow-Methods</mark> 헤더와 비슷하게, 이 헤더는 사전 전달된 요청들에 대한 응답으로 사용됩니다. 

<mark>Access-Control-Max-Age</mark>
======================================

사전 전달된(pre-flighted) 요청은 <mark>OPTIONS</mark> HTTP 메소드를 사용하여 브라우저가 먼저 서버에게 요청해야 합니다. 이 이후에 안전하다고 판단이 되어야 메인 요청을 보낼 수 있습니다. 그러나, <mark>OPTIONS</mark>를 사전 전달 요청을 호출할 때마다 보내는 것은 비효율적입니다. 

이것을 방지하기 위해 서버는 브라우저가 사전 전달 요청의 결과를 특정 시간까지 캐싱하는것을 허락하는 <mark>Access-Control-Max-Age</mark> 헤더 (Access-Control-Max-Age header)로 응답합니다. 이 헤더의 값은 델타 초 단위의 시간입니다.

전반적으로, 아래의 내용은 CORS 응답 헤더의 모양입니다. 

```js
Access-Control-Allow-Origin: <allowed_origin> | *
Access-Control-Allow-Methods: <method> | [<method>]
Access-Control-Allow-Headers: <header> | [<header>]
Access-Control-Max-Age: <delta-seconds>
```

Simple requests vs. pre-flighted requests
============================================

CORS 사전 전달(preflight)을 작동시키지 않은 요청들은 단순 요청 범주에 속합니다. 그러나, 단순 요청으로 판단된 후에는 특정 조건을 만족시켜야 합니다. 이러한 조건들은:

1. 요청의 HTTP 메소들은 이 중 하나여야 함: <mark>GET</mark>, <mark>Post</mark> 혹은 <mark>HEAD</mark>
2. 요청 헤더들은 사용자 에이전트가 설정한 헤더와 상관 없이 <mark>Accept</mark>, <mark>Accept-Language</mark>, <mark>Content-Language</mark> 그리고 <mark>Content-Type</mark>과 같은 CORS 허용 목록 헤더(CORS safe-listed headers)여야만 합니다. 
3. <mark>Content-Type</mark>헤더는 이 세개의 값중 하나여야만 합니다: <mark>application/x-www-form-urlencoded</mark>, <mark>multipart/form-data</mark> 혹은 <mark>text/plain</mark>
4. <mark>XMLHttpRequest</mark>를 사용하는 경우 <mark>XMLHttpRequest.upload</mark> 프로퍼티에 의해 반환된 객체에는 이벤트 리스터가 등록되지 않습니다. 
5. 요청에는 <mark>ReadableStream</mark> 객체를 사용하지 않습니다. 

이러한 조건 중 하나라도 충족하지 못한다면, 해당 요청은 사전 전달 요청으로 간주됩니다. 이러한 요청의 경우, 브라우저는 우선 OPTIONS 메소드를 사용하여 다른 출처로 요청을 보내야 합니다. 

이 방법은 실제 요청을 서버로 보내도 안전한지 확인하기 위해 사용됩니다. 실제 요청을 허락하거나 거절하는것은 사전 전달된 요청(pre-flighted request)에 대한 응답 헤더에 달려있습니다. 만약 이러한 응답 헤더와 메인 요청의 헤더가 일치하지 않는다면, 요청은 수행되지 않습니다.

Enabling CORS
=================

위에서 예시를 들었던것 처럼 CORS 에러를 처음 마주한 상황을 가정해보겠습니다. 리소스가 호스팅이 되는 서버에 접근 권한이 있는지에 따라 이 문제를 해결할 방법은 다양합니다. 우리는 이 상황을 두가지로 요약할 수 있습니다.

1. 백엔드에 접근할 수 있거나 백엔드 개발자를 아는 경우
2. 프론트엔드만 관리하고 백엔드 서버에 접근할 수 없는 경우

If you have access to the backend:
====================================

CORS는 단순히 HTTP 헤더 기반 메커니즘이기 때문에 서로 다른 출처들이 리소스를 공유할 수 있게 하기 위해 적절한 헤더로 응답하도록 서버를 수정할 수 있습니다. 위에서 이야기한 CORS 헤더를 보고 따라서 설정해보세요.

[Node.js](https://nodejs.org/en/) + [Express.js](https://expressjs.com/) 개발자들을 위해, npm에서 <mark>cors</mark> 미들웨어를 설치할 수 있습니다.

다음은 CORS 미들웨어와 함께 Express 웹 프레임 워크를 사용하는 예시입니다:

```js
  const express = require('express');
  const cors = require('cors');
  const app = express();

  app.use(cors());

  app.get('/', (req, res) => {
    res.send('API running with CORS enabled');
  });

  app.listen(5000, console.log('Server running on port 5000'));
```

CORS 설정으로 이루어진 객체를 넘기지 않는다면, 아래와 같은 디폴트 설정을 사용합니다. 

```js
  {
    "origin": "*",
    "methods": "GET,HEAD,PUT,PATCH,POST,DELETE",
    "preflightContinue": false,
    "optionsSuccessStatus": 204
  }
```

아래는 서버에 <mark>https://yourwebsite.com</mark>로부터 헤더가 <mark>Content-Type</mark>이고 사전 전달 캐시 시간(preflight cache time)이 10분인 <mark>Authorization</mark>을 가지고 있는 <mark>GET</mark> 요청만을 허용하는 CORS를 설정할 수 있는지 예시입니다.

```js
  app.use(cors({
    origin: 'https://yourwebsite.com',
    methods: ['GET'],
    allowedHeaders: ['Content-Type', 'Authorization'],
    maxAge: 600
  }));
```

위의 코드는 Express.js와 Node.js에만 해당 되지만 개념은 동일하게 유지됩니다. 여러분이 선택한 프로그래밍 언어와 프레임워크를 사용하여 응답에 CORS를 수동적으로 설정하거나 동일한 커스텀 미들웨어를 생성할 수 있습니다. 

If you only have access to the frontend:
===========================================

공공 API 같이 백엔드 서버에 접근할 수 없는 경우도 많이 있습니다. 여기서 우리가 받는 응답에는 헤더를 추가할 수 없죠. 그러나 프록시 요청에 CORS 헤더를 추가하는 프록시 서버는 사용할 수 있습니다.
[cors-anywhere](https://github.com/Rob--W/cors-anywhere) 프로젝트는 위와 같은 일을 할 수 있게 만들어 주는 Node.js 리버스 프록시(reverse proxy)입니다. 프록시 서버는 <mark>https://cors-anywhere.herokuapp.com/</mark>에서 이용이 가능하지만 리포지토리를 클론하고 [Heroku](https://www.heroku.com/)와 같은 무료 플랫폼을 통해 배포함으로써 여러분 자신만의 프록시 서버를 만들 수 있습니다. 

해당 메소드에서, 아래와 같이 서버에 직접적으로 요청을 보내는 대신에:

```js
fetch('https://jsonplaceholder.typicode.com/posts');
```

단순히 API의 URL의 시작에 프록시 서버 주소를 붙이면 됩니다.

```js
fetch('https://cors-anywhere.herokuapp.com/https://jsonplaceholder.typicode.com/posts');
```

Conclusion
============

교차 사이트 위조 공격(cross-site forgery attacks)를 방지하기 위한 동일 출처 정책(same-origin policy)의 고마움을 배우면서, CORS를 왜 사용해야 하는지 알아봤습니다. 콘솔창에 나타난 빨간 글씨의 CORS 메세지가 사라지지 않는 동안에, 여러분은 이제 백엔드에서든 프론트엔드에서든 이러한 메세지와 싸울 지식을 갖추었습니다. 

원문 : [The ultimate guide to enabling Cross-Origin Resource Sharing (CORS)](https://blog.logrocket.com/the-ultimate-guide-to-enabling-cross-origin-resource-sharing-cors/)
