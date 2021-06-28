---
layout: post
title: Everything New in React 18 (번역)
---

_리액트 18 알파가 출시되었으며 리액트 18에는 많은 기능들이 추가되었습니다. 리액트 17에서는 많은 변화를 느끼지는 못했지만 리액트 18은 다를겁니다. 이 글에서는 리액트 18에서의 주요 업데이트를 살펴보도록 하겠습니다._

<h3>What's new?</h3>

# 1. The New Root API

아래의 이미지는 우리가 평소에 보던 코드입니다.

![react18-1](/assets/react18-1.png)

reactDOM.render를 App 컴포넌트에 전달한 다음 <mark>document.getElementById</mark>와 root 요소를 전달합니다. 이렇게 하면 app 컴포넌트를 root 요소안에 간단히 랜더링 할 수 있습니다.

아래 새로운 방법입니다.

![react18-2](/assets/react18-2.png)

새로운 메소드인 create root와 같은 root 변수가 있습니다. 이 변수가 root 요소에 전달되어 진 후, root.render를 호출하고 app 컴포넌트를 전달합니다.

app 컴포넌트가 만들어지는 것은 같으나 방식은 다릅니다. 예전에는 레거시 root API를 호출했고 아직 리액트 18에서 작동하지만 이 글에서 나올 동시적인 특징들(concurrent features)을 포함한 리액트 18의 향상된 기능들을 사용하는 새로운 root API를 사용함으로써 어느 순간부터 사용이 권장되지는 않을겁니다(deprecated).

# 2. Suspense

새로운 업데이트에서, suspense 기능의 완전한 지원을 받게됩니다. suspense라는 이름이 말해주고 있듯이, 이 기능은 랜더가 완료될 때까지 어떤것을 보류하는걸 말합니다.

![react18-3](/assets/react18-3.png)

위의 예시에서, 랜더링 되기전에 데이터를 가져오기 위해 시간이 필요한 컴포넌트가 있습니다. suspense는 데이터가 반환되고 컴포넌트가 랜더되기까지 fallback을 사용할 겁니다.

여기서 중요한 것은 suspense안의 컴포넌트가 데이터를 기다리지 않으며 랜더될 준비를 마칠때까지 보류한다는 겁니다.

이런 suspense의 기능은 서버 사이드 랜더링(server-side rendering)에서 굉장히 유용합니다. 현재 ssr에서는 완전히 랜더된 HTML을 받지만, 유저와 상호작용이 가능해지기 전에 브라우저가 자바스크립트를 로드하고 전체적인 페이지를 동적으로 만들어줘야 합니다(hydrate). suspense는 리액트 18의 예제를 사용해서 로드되는 시간을 크게 단축할 수 있습니다.

![react18-4](/assets/react18-4.png)

위를 보시면 페이지 로딩을 위한 navbar와 sidebar, post와 comment 컴포넌트가 있습니다. comments 컴포넌트는 사이트가 상호작용이 가능하기 전까지는 로드될 필요가 없죠. 그래서 comments 컴포넌트를 보류(suspend)할겁니다. 그래야지 사용자가 글을 읽을 수 있으니까요. 그리고 comments를 백그라운드에서 로드할겁니다.

# 3. Automatic Batching

리액트 17과 그 이전 버전의 일괄(batch) 리랜더링(re-render)은 클릭과 같은 브라우저 이벤트가 발생하는 동안 업데이트를 합니다.

리액트 17은 데이터를 가져온 다음, 상태를 업데이트해야하는 경우 리랜더링을 일괄 처리하지 않습니다. 리액트 18에서는 만약 여러분이 새로운 create root API를 사용한다면, 모든 상태 업데이트는 자동적으로 일괄 진행될것입니다. 언제 발생했는지와는 상관없이 말이죠.

만약 일괄적인 업데이트를 원하지 않는 중요한 컴포넌트가 있다면, <mark>ReactDOM.flushSync()</mark>를 사용해 비적용 시킬 수 있습니다.

# 4. startTransition API

현재의 웹 페이지를 반응형으로 유지할 수 있게 도와주며 동시에 무거운 non-blocking UI의 업데이트를 가능케 해주는 이 새로운 API는 새로 출시되었습니다.

이 API는 리액트에게 업데이트의 우선순위를 알려주어 UI가 반응형으로 유지되는것을 도와줍니다.

타이핑, 호버(hover), 클릭과 같은 급한 업데이트(urgent updates)들의 props/functions 호출은 아래와 같이 이뤄집니다.

![react18-5](/assets/react18-5.png)

급하지 않은 무거운 UI 업데이트는, 아래와 같이 startTransition API를 이용해 감싸면 됩니다.

![react18-6](/assets/react18-6.png)

# 5. Suspense List

<mark>SuspenseList</mark> 컴포넌트는 값 forward, backward 혹은 together와 함께 <mark>revealOrder</mark> prop을 받습니다.

![react18-7](/assets/react18-7.png)

위의 이미지처럼 Card 컴포넌트가 forward 방향으로 표시됩니다(데이터를 가져올때까지, LoadingSpinner 컴포넌트가 돌아갑니다). 이와 비슷하게, <mark>backwards</mark>는 Card 컴포넌트들을 반대 방향으로 표시하며, together prop은 모든것을 "같이" 랜더합니다.

# 6. useDeferredValue

![react18-8](/assets/react18-8.png)

<mark>useDeferredValue</mark>는 state 값과 밀리세컨드(ms)의 타임아웃을 받습니다. 그리고 최대 시간동안 지연할 수 있는 값의 유예된 버전을 반환합니다.

유저의 입력값에 따라 당장 랜더링이 필요할때 그리고 데이터 가져오는것을 기다려야 할때, 해당 기능은 인터페이스를 반응형으로 유지하기 위해 자주 사용됩니다.

리액트 18과 reactDOM은 아래와 같이 바로 설치할 수 있습니다.

![react18-9](/assets/react18-9.png)

# Wrapping Up

리액트 18은 현재 알파버전이 출시되었으며 아직 production에는 적합하지 않습니다. 하지만 이러한 기능들을 배우기 시작하는것이 좋습니다.

리액트 18은 몇 달 후, public 베타버전이 될겁니다.

읽어주셔서 감사합니다. 🙌

마음껏 방문해주세요. 👇

[GitHub](https://github.com/Push9828) <br />
[Twitter](https://twitter.com/PushkarThakur28) <br />
[LinkedIn](www.linkedin.com/in/pushkarthakur28) <br />

원문: [Everything New in React 18 by Pushkar Thakur](https://javascript.plainenglish.io/everything-new-in-react-18-db459c2608de)
