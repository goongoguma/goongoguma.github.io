---
layout: post
title: When to useMemo and useCallback (번역)
---

_최적화에는 비용이 있기 마련이며 무조건 유익한것은 아닙니다. 이 글에서는 useMemo와 useCallback을 사용함으로써 발생되는 비용과 혜택을 설명해보겠습니다_


<iframe src="https://codesandbox.io/embed/late-cdn-z2zcc?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="late-cdn-z2zcc"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

위의 사탕 자판기는 이렇게 동작합니다.

```js
  function CandyDispenser() {
    const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
    const [candies, setCandies] = React.useState(initialCandies)
    const dispense = candy => {
      setCandies(allCandies => allCandies.filter(c => c !== candy))
    }
    return (
      <div>
        <h1>Candy Dispenser</h1>
        <div>
          <div>Available Candy</div>
          {candies.length === 0 ? (
            <button onClick={() => setCandies(initialCandies)}>refill</button>
          ) : (
            <ul>
              {candies.map(candy => (
                <li key={candy}>
                  <button onClick={() => dispense(candy)}>grab</button> {candy}
                </li>
              ))}
            </ul>
          )}
        </div>
      </div>
    )
  }
```
이제 질문을 하나 드리겠습니다. 위의 코드를 수정해 볼건데요, 수정 전의 코드와 수정 후의 코드중 성능면에서 어떤 코드가 더 나은지 선택해주시면 되겠습니다. 

React.useCallback
```js
  const dispense = React.useCallback(candy => {
    setCandies(allCandies => allCandies.filter(c => c !== candy))
  }, [])
```
그리고 기존의 dispense 함수입니다.
```js
  const dispense = candy => {
    setCandies(allCandies => allCandies.filter(c => c !== candy))
  }
```
자, 질문입니다. 위 두개의 코드중 어떤 코드의 성능이 더 좋을까요? 

<iframe src="https://codesandbox.io/embed/pedantic-cloud-zeiy2?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="pedantic-cloud-zeiy2"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

왜 useCallback의 사용이 더 나쁠까?
---------------------------------------

리액트를 사용하다보면 인라인 함수가 성능에 문제가 될 수 있기에 useCallback을 사용해 성능을 개선해야 한다는 말을 많이 들었습니다. 그런데 어떻게 useCallback을 안쓰는 것만 못할수가 있을까요?

우리가 위의 예제, 아니 리액트를 떠나서 생각해야 할 부분이 있습니다. 
**모든 라인에 있는 코드는 실행이 될때 비용을 수반합니다**

이 글을 읽으시는 분들의 수월한 이해를 위해 해당 useCallback 예제를 조금만 바꿔보도록 하겠습니다.
```js
  const dispense = candy => {
    setCandies(allCandies => allCandies.filter(c => c !== candy))
  }

  const dispenseCallback = React.useCallback(dispense, [])
```

그리고 아래는 초기 dispense 함수입니다.
```js
  const dispense = candy => {
    setCandies(allCandies => allCandies.filter(c => c !== candy))
  }
```

차이점이 보이시나요? 

```js
  const dispense = candy => {
      setCandies(allCandies => allCandies.filter(c => c !== candy))
    }

  + const dispenseCallback = React.useCallback(dispense, [])
```
맞습니다. 두개의 예시에서 dispense 함수는 같은 일을 수행하지만 useCallback 버전의 예시가 더 많은 일을 하고있습니다. useCallback 버전은 함수를 정의하는 일 뿐만 아니라 다양한 일(프로퍼티의 셋팅/논리적인 표현식의 실행)을 위해 배열([])을 정의해줘야 하죠.

그래서 두개의 예시에서 컴포넌트가 매순간 랜더링 될때마다 자바스크립트는 메모리에 함수를 정의하게 되며 useCallback이 어떻게 사용되는지에 따라 메모리에 더 많은 함수가 정의 될 수가 있는겁니다.

지난번 이 내용들을 이해하기위해 트위터 투표를 진행했습니다.

![When to useMemo and useCallback](/assets/useMemoCallback.jpg)
_몇몇 분들께서 설명이 제대로 되어있지 않았다고 말씀해주셨습니다. 그 이유로 잘못된 정답을 고르신분들께 사과드립니다. 그런데 저는 이미 정답을 알고있었어요._

그리고 또 하나 말씀드리고 싶은것은 컴포넌트가 두번째로 랜더됐을때 기존에 있던 dispense 함수는 가비지 컬렉터가 되며 새로운 함수가 생성이 됩니다. 그런데 useCallback을 사용하게 된다면 기존의 함수는 가비지 컬렉터가 되질않고 새로운 함수가 생성이 되어버리죠. 즉, 메모리 사용 측면에서 비효율적이라는 것입니다. 

관련된 내용으로 만약 useCallback의 dependency 배열안의 종속 값들을 사용한다면 리액트는 전에 생성된 함수의 참조(reference)로 배열 안에 있는 값을 계속 가지고있을겁니다. 왜냐하면 메모이제이션은 전과 같은 종속 값들을 받게되는 경우, 전에 가지고 있던 값들을 저장해서 그대로 리턴해준다는 뜻이니까요. 
이미 눈치채셨을수도 있지만 리액트는 동일성 체크를 위해 종속된 값들의 참조를 가지고 있다는 뜻입니다. 

useMemo는 어떻게 다르면서도 비슷할까?
-------------------------------------------------

useMemo는 어떤 타입의 값이든 메모이제이션의 사용을 가능케 한다는 부분을 제외하곤 useCallback과 비슷합니다. 
useMemo는 값을 리턴하는 함수를 받고 해당 함수의 리턴 값이 필요할때만 사용됩니다. (보통 dependency 배열안의 종속값들이 렌더시에 변화할때마다 한번 발생합니다.)

그래서 만약에 initialCandies 배열이 랜더될때마다 다시 만들기 싫다면 이렇게 만들 수 있겠네요.

```js
  - const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']

  + const initialCandies = React.useMemo(
  +  () => ['snickers', 'skittles', 'twix', 'milky way'],
  +  [],
  + )
```
initialCandies 배열이 랜더시에 다시 만들어지는 문제는 해결하기는 했지만 이렇게 해서 발생하는 효율은 그렇게 좋지는 않습니다. 오히려 코드가 좀 더 복잡해지기만 할 뿐이죠. 사실 useMemo를 사용하는게 더 비효율적일수 있어요. 왜냐하면 위에서 말했듯이 함수를 호출하면서 코드가 메모리에 할당될 테니까요.

위의 예시를 어떻게 하면 더 효율적으로 수정할 수 있을까요? 
```js
  + const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']

    function CandyDispenser() {
  -   const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
      const [candies, setCandies] = React.useState(initialCandies)
```
그런데 항상 이렇게 할수있는게 아니겠죠. 왜냐하면 함수에서 쓰이는 값들은 props를 통해 내려온걸수도 있고 함수 안에서 선언되어야하는 변수일수도 있을테니까요.

제가 말하고자 하는 부분은 근데 이게 아닙니다. 위의 코드를 최적화 함으로서 얻어지는 효율은 너무나 작아서 어떻게 하면 프로젝트를 개선할수 있을까를 생각하며 시간을 보내는편이 훨씬 나을겁니다. 

그래서 중요한것은?
-----------------------------------

- 중요한 부분은 이겁니다.

**성능 개선은 공짜가 아닙니다. 항상 비용이 들기 마련이고 성능개선을 함으로써 얻어지는 이득이 꼭 그 비용을 상쇄할수 있는건 아니라는겁니다.**

그러므로 최적화는 책임감을 가지고 사용해야합니다.

그렇다면 언제 useMemo와 useCallback을 사용해야 할까요?
------------------------------------------------------------------

useMemo와 useCallback이 훅으로 만들어진건 여러 이유가 있습니다.

1. 참조 동일성 (Referential equality)
2. 비용이 많이 드는 계산 (Computationally expensive calculations)

참조 동일성 (Referential equality)
---------------------------------------------------------------------

- 자바스크립트/프로그래밍 초보라도 아래의 코드를 이해하는데는 오래 걸리지 않을겁니다. 
```js
  true === true // true
  false === false // true
  1 === 1 // true
  'a' === 'a' // true
  {} === {} // false
  [] === [] // false
  () => {} === () => {} // false
  const z = {}
  z === z // true
  // NOTE: React actually uses Object.is, but it's very similar to ===
```
위의 코드는 자세히 설명은 안하겠지만 리액트의 함수형 컴포넌트 안에서 정의된 객체들은 같은 프로퍼티와 같은 값들을 가지고 있을지라도 같은 참조를 바라보고 있지 않다는 사실은 중요하죠.

리액트에서는 참조 동일성을 생각해야 하는 두가지 경우가 있습니다. 

Dependencies lists
--------------------------------------------
- 예시를 보도록 할까요?

_아래의 예시는 이해를 돕기위한 코드입니다. 코드보다는 제가 설명하려는 개념에 집중해주셨으면 합니다_

```js
  function Foo({bar, baz}) {
    const options = {bar, baz}
    React.useEffect(() => {
      buzz(options)
    }, [options]) // we want this to re-run if bar or baz change
    return <div>foobar</div>
  }

  function Blub() {
    return <Foo bar="bar value" baz={3} />
  }
```
위의 코드에 문제가 있는데요 useEffect는 options라는 변수를 대상으로 랜더되는 순간마다 참조 동일성 체크를 할겁니다. 그렇게 된다면 options는 매 순간마다 새로 만들어지게 되므로 컴포넌트가 랜더되고 options가 바뀌었는지 체크할때 항상 true로 판별이 되겠죠? 
즉, useEffect 콜백은 options 안의 bar와 baz가 바뀌었을때 호출되는게 아니라 랜더되는 순간마다 호출이 된다는겁니다.

이 문제를 해결하기 위해 두가지를 고쳐야 합니다.

```js
// option 1
  function Foo({bar, baz}) {
    React.useEffect(() => {
      const options = {bar, baz}
      buzz(options)
    }, [bar, baz]) // we want this to re-run if bar or baz change
    return <div>foobar</div>
  }
```
정말 좋은 방법이죠. 만약에 예제가 진짜였다면 저는 이렇게 고쳤을겁니다. 

그런데 만약 bar나 baz가 객체/배열/함수와 같이 원시타입이 아닌 경우 어떻게 할까요?
```js
  function Blub() {
    const bar = () => {}
    const baz = [1, 2, 3]
    return <Foo bar={bar} baz={baz} />
  }
```
위의 경우가 바로 useCallback과 useMemo가 만들어진 이유입니다. 그래서 이렇게 아래처럼 고치면 될거같네요.

```js
  function Foo({bar, baz}) {
    React.useEffect(() => {
      const options = {bar, baz}
      buzz(options)
    }, [bar, baz])
    return <div>foobar</div>
  }
  
  function Blub() {
    const bar = React.useCallback(() => {}, [])
    const baz = React.useMemo(() => [1, 2, 3], [])
    return <Foo bar={bar} baz={baz} />
  }
```
_참고로 useEffect, useLayoutEffect, useCallback, useMemo에 사용되는  dependencies 배열에 똑같이 적용됩니다._

React.memo
--------------------------
_아래의 예시는 이해를 돕기위한 코드입니다. 코드보다는 제가 설명하려는 개념에 집중해주셨으면 합니다_

아래의 코드를 볼까요?
```js
  function CountButton({onClick, count}) {
    return <button onClick={onClick}>{count}</button>
  }

  function DualCounter() {
    const [count1, setCount1] = React.useState(0)
    const increment1 = () => setCount1(c => c + 1)
    const [count2, setCount2] = React.useState(0)
    const increment2 = () => setCount2(c => c + 1)
    return (
      <>
        <CountButton count={count1} onClick={increment1} />
        <CountButton count={count2} onClick={increment2} />
      </>
    )
  }
```
두개의 버튼중 하나의 버튼이라도 클릭이 된다면 DualCounter의 상태(state)는 변하게 되고 두개의 CountButton 컴포넌트도 리랜더링을 하게 됩니다. 그런데 실질적으로는 클릭한 함수의 컴포넌트만 다시 랜더 되어야하지 않을까요?
이것을 우리는 "불필요한 리랜더"(unnecessary re-render)라고 부릅니다.

**하지만 대부분의 경우 불필요한 리랜더를 크게 신경쓰지 않아도 됩니다.** 리액트는 굉장히 빠르고 불필요한 리랜더를 해결하는것보다 중요한 일들이 있으니까요. 사실 곧 보여드릴 최적화가 필요한 예시는 필자가 리액트를 가지고 일하는 기간동안 한번도 본적이 없을 정도로 굉장히 희박한 케이스입니다. 

그러나 상호작용이 가능한 그래프나 차트, 애니메이션등과 같이 랜더링이 발생할때 상당한 시간이 걸리게되는 상황들도 있습니다. 다행히도 리액트의 실용적인 속성 덕분에 해결할 수 있는 방법이 존재합니다.
```js
const CountButton = React.memo(function CountButton({onClick, count}) {
  return <button onClick={onClick}>{count}</button>
})
```

이제 리액트는 CountButton의 props가 변할때만 다시 랜더링합니다! 와! 그런데 아직 끝난게 아니에요.
위에서 이야기한 참조 동일성(Referential equality)을 기억하시나요? 
DualCounter 함수형 컴포넌트에서 함수 increment1과 increment2 함수를 선언했는데요 이 말은 즉, DualCounter 컴포넌트가 랜더링 될때마다 안에서 선언한 함수들은 새로 만들어질것이고 리액트는 두개의 CountButton 컴포넌트를 어쨌거나 다시 랜더링 할거라는거죠. 

그래서 아래의 예시는 useCallback과 useMemo을 사용해 함수의 재생성과 변수의 재선언을 방지할 수 있는 개선된 코드입니다. 

```js
  const CountButton = React.memo(function CountButton({onClick, count}) {
    return <button onClick={onClick}>{count}</button>
  })

  function DualCounter() {
    const [count1, setCount1] = React.useState(0)
    const increment1 = React.useCallback(() => setCount1(c => c + 1), [])
    const [count2, setCount2] = React.useState(0)
    const increment2 = React.useCallback(() => setCount2(c => c + 1), [])
    return (
      <>
        <CountButton count={count1} onClick={increment1} />
        <CountButton count={count2} onClick={increment2} />
      </>
    )
```

이렇게해서 CountButton의 "불필요한 리랜더"를 방지할 수 있습니다. 

다시 말씀드리자면 저는 React.memo(그리고 memo의 친구들인 PureComponent와 shouldComponentUpdate)를 기준없이 사용하는걸 반대합니다. 왜냐하면 최적화에는 비용이 따르기 마련이며 코드를 작성하는 사람은 memo의 사용으로 인한 비용과 그리고 그에 따른 이득을 생각하여 memo의 사용이 실질적으로 나에게 도움이 될것인지, 그리고 항상 코드가 의도한 대로 동작하여 memo를 사용함에 있어서 오는 이점을 취할 수 있을 것인지를 확인해야 합니다. 

비용이 많이 드는 계산 (Computationally expensive calculations)
--------------------------------------------------------------------
비용이 많이 드는 계산의 경우도 useMemo가 훅으로 만들어진 또 다른 이유입니다.(useCallback은 제외)
useMemo의 사용은 아래와 같은 장점을 가지고 있습니다.
```js
  const a = {b: props.b}
```

이것을 lazy하게 받아봅시다.
```js
  const a = React.useMemo(() => ({b: props.b}), [props.b])
```
위의 예시는 그렇게 유용하지는 않지만 동기적으로 복잡한 값을 계산하는 함수가 있다고 생각해봅시다.
```js
  function RenderPrimes({iterations, multiplier}) {
    const primes = calculatePrimes(iterations, multiplier)
    return <div>Primes! {primes}</div>
  }
```
iterations와 multiplier가 어떤 일을 하는지 봐서 아시겠지만 예제 함수의 계산 속도는 꽤나 느릴겁니다. 그리고 여기서 우리가 할 수 있는 일은 없어요. 하드웨어에 마법을 걸어 속도를 빠르게 만들지는 못하겠죠. 
하지만 useMemo를 사용해 연속으로 같은 값을 다시 계산하지 않도록 만들어 속도를 향상시키는 방법은 있습니다.
```js
  function RenderPrimes({iterations, multiplier}) {
    const primes = React.useMemo(() => calculatePrimes(iterations, multiplier), [
      iterations,
      multiplier,
    ])
    return <div>Primes! {primes}</div>
  }
```
이 방법이 먹히는 이유는 비록 컴포넌트가 매번 랜더될때마다 소수(primes)를 계산하는 함수를 정의했지만, 리액트는 소수의 값이 필요할 때만 해당 함수를 호출하기 때문입니다. 덧붙이자면 리액트는 또한 전에 입력되었던 값을 저장하고 있으며 같은 입력값에 한하여 같은 리턴값을 내보냅니다. 이렇게 메모이제이션은 동작합니다. 

결론
-----------------------

저는 이 글을 "모든 추상화(그리고 성능 최적화)에는 비용이 든다"라고 말하며 마치겠습니다.
[the AHA Programming principle](https://kentcdodds.com/blog/aha-programming)를 적용해보시고 전과 후를 비교해보세요. 이득없이 비용이 발생되는 상황에서 도움이 될겁니다. 

분명히 말씀드리자면 useCallback과 useMemo를 사용함으로써 
1. 동료가 보기에 코드가 더 복잡해 질 수 있고 
2. dependencies 배열을 잘못 사용할수도 있으며 
3. 내부 훅을 호출함으로써 성능상 안쓰느니 못하게 만들 수도 있고
4. dependency들과 memoized된 값들이 가비지 컬랙터가 안되게 만들수도 있습니다.
굳이 성능상 이점을 원한다면 위 비용들의 발생을 감수할수도 있지만 **손익분실 계산이 최우선이 되어야 합니다.**

관련 글:

- React FAQ: ["Are Hooks slow because of creating functions in render?"](https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render)
- [Ryan Florence: React, Inline Functions, and Performance](https://twitter.com/ryanflorence)

추신. 만약 hook을 사용해서 움직이는것이 걱정되고 클래스 컴포넌트에서 함수를 메서드로 사용하는 방식이 아닌  강제적으로 함수형 컴포넌트에서 함수들을 정의한다고 생각하는 분들이 계시다면 맨 처음부터 메서드들은 render 함수 안에서 정의하고 있었다는 사실을 생각해주시기 바랍니다.
```js
  class FavoriteNumbers extends React.Component {
    render() {
      return (
        <ul>
          {this.props.favoriteNumbers.map(number => (
            // TADA! This is a function defined in the render method!
            // Hooks did not introduce this concept.
            // We've been doing this all along.
            <li key={number}>{number}</li>
          ))}
        </ul>
      )
    }
  }
```
원문 : [When to useMemo and useCallback by Kent C. Dodds](https://kentcdodds.com/blog/usememo-and-usecallback)

