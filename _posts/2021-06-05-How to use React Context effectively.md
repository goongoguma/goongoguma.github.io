---
layout: post
title: How to use React Context effectively (번역)
---

[리액트의 어플리케이션 상태 관리(Application State Management)](https://kentcdodds.com/blog/application-state-management-with-react)에서 어떻게 하면 리액트 어플리케이션에서의 로컬 state와 context의 조합이 상태관리에 있어 도움을 주는지 이야기 했습니다. 몇 개의 예시를 보여드리면서 몇몇 부분과 어떻게 하면 리액트의 context consumer를 효과적으로 만들어, 발생할 수 있는 문제를 피하고 개발자 경험과 여러분이 어플리케이션이나 라이브러리를 위해 만든 context 객체의 유지성을 향상시킬 수 있는지도 강조했습니다. 

_[리액트의 어플리케이션 상태 관리(Application State Management)](https://kentcdodds.com/blog/application-state-management-with-react)를 읽고 마주치는 모든 state 공유 문제를 해결하기 위해 context를 쓰지말아주세요. 그러나 context가 필요한 순간에, 이 블로그 포스트가 어떻게 하면 효과적으로 context를 사용할 수 있는지 도움을 주었으면 합니다. 또한, context는 앱 전체에서 전역으로 사용될 필요가 없다는것을 알아두세요. 하지만 context는 트리(tree)의 한 부분에 적용될 수 있으며 앱 안에서 논리적으로 분리된 여러개의 context들을 가질 수 있습니다(아마도 그럴겁니다)._

첫째로 <mark>src/count-context.js</mark>안에 파일을 생성하고 context를 생성해 볼겁니다.

```js
  // src/count-context.js
  import * as React from 'react'
  const CountContext = React.createContext()
```
우선, <mark>CountContext</mark>는 초기값(initial value)을 가지고 있지 않습니다. 만약 제가 초기값을 원했었다면 <mark>React.createContext({count: 0})</mark>이렇게 호출했을 겁니다. 그런데 저는 기본값(defaultValue)을 넣어주지 않았고 이것은 의도된겁니다. <mark>기본값</mark>은 아래와 같은 상황에서만 유용합니다.

```js
  function CountDisplay() {
    const {count} = React.useContext(CountContext)
    return <div>{count}</div>
  }
  ReactDOM.render(<CountDisplay />, document.getElementById('⚛️'))
```

<mark>CountContext</mark>에 기본값이 없기 때문에 <mark>useContext</mark>에 구조 분해 할당을 사용해서 값을 리턴한 부분에 에러줄이 뜰겁니다. 왜냐하면 기본값이 undefined인데 undefined는 구조 분해 할당이 될 수 없으니까요. 

누구도 런타임 에러를 좋아하지 않죠. 그래서 여러분은 런타임 에러를 피하기 위해 기본값을 추가할겁니다. 하지만 위의 예시처럼 실제적인 값이 없는데 context를 왜 사용하는 것일까요? 
만일 제공되는 기본값을 사용한다면 좋은 결과를 얻을 수 없습니다. 여러분이 어플리케이션에 context를 만들고 사용하는 99퍼센트의 경우에는 context의 consumer(<mark>useContext</mark>를 사용하는것들)가 유용한 값들을 제공해주는 provider 안에 랜더되기를 원합니다. 

_기본값이 유용한 상황들이 있긴합니다만 대부분의 경우 필요하거나 유용하지는 않습니다_

[리액트 공식문서](https://reactjs.org/docs/context.html#reactcreatecontext)에서는 기본값을 제공하는건 "컴포넌트가 감싸인 형태가 아닌 분리된 상황에서 테스트 하는데에는 유용할 수 있다."라고 제안합니다. 이렇게 하는것이 가능하다는건 맞는 말이지만, 필요한 컨텍스트로 컴포넌트를 감싸는것보다 낫다는 부분에 있어서는 동의하지 않습니다. 
여러분이 어플리케이션에서 사용하지 않는 기능을 테스트 할 때마다 자신감이 줄어들 수 있음을 기억하세요. [테스트가 필요한 이유가 있지만](https://kentcdodds.com/blog/the-merits-of-mocking), 이 부분은 해당 이유에 속하지 않습니다. 

_만약 여러분이 타입스크립트를 사용하고 있다면, 기본값을 제공하지 않는건 <mark>React.useContext</mark>를 사용함에 있어 굉장히 귀찮을 수 있지만 어떻게 그 문제를 피하는지도 보여드릴겁니다. 계속 읽어주세요!_

The Custom Provider Component
===================================

그럼 계속 진행해보겠습니다. 해당 context 모듈을 유용하게 만들기 위해서는 Provider를 사용하고 값을 제공해주는 컴포넌트를 노출시킬 필요가 있습니다. 해당 컴포넌트들은 아래와 같습니다.

```js
  function App() {
    return (
      <CountProvider>
        <CountDisplay />
        <Counter />
      </CountProvider>
    )
  }

  ReactDOM.render(<App />, document.getElementById('⚛️'))
```

그렇다면 위와 같이 사용할 수 있는 컴포넌트들을 만들어 보겠습니다.

```js
  // src/count-context.js
  import * as React from 'react'

  const CountContext = React.createContext()
  
  function countReducer(state, action) {
    switch (action.type) {
      case 'increment': {
        return {count: state.count + 1}
      }
      case 'decrement': {
        return {count: state.count - 1}
      }
      default: {
        throw new Error(`Unhandled action type: ${action.type}`)
      }
    }
  }

  function CountProvider({children}) {
    const [state, dispatch] = React.useReducer(countReducer, {count: 0})
    // NOTE: you *might* need to memoize this value
    // Learn more in http://kcd.im/optimize-context
    const value = {state, dispatch}
    return <CountContext.Provider value={value}>{children}</CountContext.Provider>
  }
  export {CountProvider}
```

_위의 복잡한 예시는 실제로 어떻게 사용되는지 보여주기 위해 제가 인위적으로 어렵게 만들어 놓은 예시입니다. **그렇다고 항상 이렇게 복잡하지는 않습니다!** 여러분에게 맞는 시나리오에 따라 <mark>useState</mark>를 사용하셔도 됩니다. 어떤 Provider들은 예시처럼 짧고 간단하고 또 다른 것들은 해당 예시보다도 더 많은 hook들을 포함하고 있습니다._

The Custom Consumer Hook
==============================

제가 평소에 본 context 사용에 대한 API들은 아래와 같습니다. 

```js
  import * as React from 'react'
  import {SomethingContext} from 'some-context-package'
  function YourComponent() {
    const something = React.useContext(SomethingContext)
  }
```

그러나 위의 예시는더 나은 사용자 경험의 기회를 놓친거같습니다. 
대신에, 제 생각에는 이렇게 사용해야 할 것 같습니다.

```js
  import * as React from 'react'
  import {useSomething} from 'some-context-package'
  function YourComponent() {
    const something = useSomething()
  }
```

지금 보여드린 예시는 아래에 제가 보여드릴 예시처럼 몇 가지의 작업을 수행할 수 있다는 장점이 있습니다.

```js
  // src/count-context.js
  import * as React from 'react'

  const CountContext = React.createContext()

  function countReducer(state, action) {
    switch (action.type) {
      case 'increment': {
        return {count: state.count + 1}
      }
      case 'decrement': {
        return {count: state.count - 1}
      }
      default: {
        throw new Error(`Unhandled action type: ${action.type}`)
      }
    }
  }

  function CountProvider({children}) {
    const [state, dispatch] = React.useReducer(countReducer, {count: 0})
    // NOTE: you *might* need to memoize this value
    // Learn more in http://kcd.im/optimize-context
    const value = {state, dispatch}
    return <CountContext.Provider value={value}>{children}</CountContext.Provider>
  }

  function useCount() {
    const context = React.useContext(CountContext)
    if (context === undefined) {
      throw new Error('useCount must be used within a CountProvider')
    }
    return context
  }
  export {CountProvider, useCount}
```

첫번째로 <mark>useCount</mark>훅은 가장 가까운 <mark>CountProvider</mark>에서부터 제공된 context 값들을 갖기 위해 <mark>React.useContext</mark>를 사용합니다. 
하지만 만약에 값이 존재하지 않을 경우, <mark>CountProvider</mark> 내에서 렌더된 함수형 컴포넌트 안에 있는 훅이 호출되지 않았다는 메세지를 보냅니다. 이것은 실수인게 분명하므로, 에러 메세지 제공은 중요합니다. _#FailFast_

The Custom Consumer Component
===================================

만약 여러분이 훅을 사용할 수 있다면 해당 섹션은 넘어가셔도 됩니다. 그러나 만약 16.8.0버전 이상의 리액트를 지원해야 하거나 클래스형 컴포넌트에서 Context의 사용이 필요하다고 생각되는 경우 context consumers를 위해 render-prop 기반 API를 사용하여 비슷하게 작업을 수행하는 방법은 다음과 같습니다. 

```js
  function CountConsumer({children}) {
    return (
      <CountContext.Consumer>
        {context => {
          if (context === undefined) {
            throw new Error('CountConsumer must be used within a CountProvider')
          }
          return children(context)
        }}
      </CountContext.Consumer>
    )
  }
```

그리고 아래는 클래스 컴포넌트에서의 사용법입니다.

```js
  class CounterThing extends React.Component {
    render() {
      return (
        <CountConsumer>
          {({state, dispatch}) => (
            <div>
              <div>{state.count}</div>
              <button onClick={() => dispatch({type: 'decrement'})}>
                Decrement
              </button>
              <button onClick={() => dispatch({type: 'increment'})}>
                Increment
              </button>
            </div>
          )}
        </CountConsumer>
      )
    }
  }
```

훅이 나오기 전에 저는 위의 예시처럼 사용했고 잘 작동했습니다. 하지만 훅을 사용하실 수 있으면 위의 예시처럼 사용하라고 추천하지는 않겠습니다. 훅이 훨씬 더 나으니까요.

TypeScript
==============

타입스크립트를 사용할 때 <mark>기본값</mark>을 사용하지 않음으로써 발생하는 문제를 피할 수 있는 방법을 보여드리기로 약속했죠. 보세요! 제가 제안하는 방식으로 해봄으로써 이런 문제를 기본적으로 피할 수 있습니다! 사실 문제도 아니에요. 확인해보시죠:

```js
  // src/count-context.tsx
  import * as React from 'react'
  type Action = {type: 'increment'} | {type: 'decrement'}
  type Dispatch = (action: Action) => void
  type State = {count: number}
  type CountProviderProps = {children: React.ReactNode}

  const CountStateContext = React.createContext<
    {state: State; dispatch: Dispatch} | undefined
  >(undefined)

  function countReducer(state: State, action: Action) {
    switch (action.type) {
      case 'increment': {
        return {count: state.count + 1}
      }
      default: {
        throw new Error(`Unhandled action type: ${action.type}`)
      }
    }
  }

  function CountProvider({children}: CountProviderProps) {
    const [state, dispatch] = React.useReducer(countReducer, {count: 0})
    // NOTE: you *might* need to memoize this value
    // Learn more in http://kcd.im/optimize-context
    const value = {state, dispatch}
    return (
      <CountStateContext.Provider value={value}>
        {children}
      </CountStateContext.Provider>
    )
  }

  function useCount() {
    const context = React.useContext(CountStateContext)
    if (context === undefined) {
      throw new Error('useCount must be used within a CountProvider')
    }
    return context
  }

  export {CountProvider, useCount}
```
이렇게 함으로써, 누구든 undefined 확인을 해야 할 필요 없이 <mark>useCount</mark>를 사용할 수 있죠, 왜냐하면 우리가 해주었거든요!

[여기서 확인하실 수 있습니다.](https://codesandbox.io/s/bitter-night-i5mhj)

What about dispatch type typos?
=================================

이 시점에서, 리덕스는 소리를 지를겁니다: "이봐, 액션 생성자(action creators)는 어디있는거야?!" 만약에 여러분이 액션 생성자를 사용한다고 해도 괜찮습니다만, 저는 액션 생성자를 좋아해 본적이 없습니다.
저는 항상 액션 생성자는 불필요한 요약이라고 느꼈습니다. 또한, 만약 여러분이 타입스크립트를 사용하고 있으며 액션이 잘 정의되어 있다면, 굳이 필요하지는 않습니다. 자동완성과 인라인 타입 에러가 발생하니까요!

![react-context1](/assets/react-context1.png)

![react-context2](/assets/react-context2.png)

저는 <mark>dispatch</mark>함수를 위의 방식처럼 넘기는 것을 좋아합니다. 이런 방식의 또 다른 장점은, <mark>dispatch</mark>함수는 생성된 컴포넌트의 생명주기 동안 안전하기에 여러분은 <mark>dispatch</mark>함수를 useEffect의 종속성 배열에 넘길 필요가 없다는 겁니다(있으나 없으나 큰 차이는 없습니다).

만약 여러분이 자바스크립트(코드)를 입력하지 않을 경우(코드를 입력하지 않았을 경우를 생각해야합니다. 만약 그런적이 없었다면 말이죠.), 놓친 액션 타입 때문에 나오는 에러는 안전장치 입니다. 또한 다음 섹션을 읽어보세요 도움이 될테니까요. 

What about async actions?
=============================

정말로 좋은 질문입니다. 만약 여러분이 비동기 요청을 해야하고 이 요청 과정에서 여러가지들을 dispatch 해야하는 처하게 된다면 어떻게 해야할까요?
물론 컴포넌트가 호출되는 시점에서 할 수 있긴 합니다만 이런 일을 해야하는 컴포넌트들에게 모든 구성 요소들을 수동적으로 연결하는 것은 굉장히 귀찮은 일입니다. 

제가 제안하는 것은 <mark>dispatch</mark>함수와 함께 여러분이 필요한 데이터를 받는 context 모듈 안에 도우미 함수(helper function)를 만드는 겁니다. 그리고 그 도우미 함수가 모든것을 책임지게 만드는 것이죠. [my Advanced React Patterns workshop](https://kentcdodds.com/workshops/advanced-react-patterns)에 예시가 있습니다:

```js
  // user-context.js
  async function updateUser(dispatch, user, updates) {
    dispatch({type: 'start update', updates})
    try {
      const updatedUser = await userClient.updateUser(user, updates)
      dispatch({type: 'finish update', updatedUser})
    } catch (error) {
      dispatch({type: 'fail update', error})
    }
  }
  export {UserProvider, useUser, updateUser}
```

그리고 이렇게 사용할 수 있습니다:

```js
  // user-profile.js
  import {useUser, updateUser} from './user-context'
  function UserSettings() {
    const [{user, status, error}, userDispatch] = useUser()
    function handleSubmit(event) {
      event.preventDefault()
      updateUser(userDispatch, user, formState)
    }
    // more code...
  }
```

저는 이 패턴이 굉장히 만족스러우며 만약 여러분의 회사에서 이것을 배우고 싶다면 [알려주세요](https://kentcdodds.com/contact/) (아니면 다음에 제가 주최하는 워크샵 [대기자 리스트에 추가](https://kentcdodds.com/workshops/advanced-react-patterns)해주세요)!

Conclusion
=============

그래서 아래가 코드의 최종 버전입니다. 

```js
  // src/count-context.js
  import * as React from 'react'

  const CountContext = React.createContext()
  function countReducer(state, action) {
    switch (action.type) {
      case 'increment': {
        return {count: state.count + 1}
      }
      case 'decrement': {
        return {count: state.count - 1}
      }
      default: {
        throw new Error(`Unhandled action type: ${action.type}`)
      }
    }
  }

  function CountProvider({children}) {
    const [state, dispatch] = React.useReducer(countReducer, {count: 0})
    // NOTE: you *might* need to memoize this value
    // Learn more in http://kcd.im/optimize-context
    const value = {state, dispatch}
    return <CountContext.Provider value={value}>{children}</CountContext.Provider>
  }

  function useCount() {
    const context = React.useContext(CountContext)
    if (context === undefined) {
      throw new Error('useCount must be used within a CountProvider')
    }
    return context
  }

  export {CountProvider, useCount}
```

[여기서 작동하는 실제 예시를 확인할 수 있습니다.](https://codesandbox.io/s/react-codesandbox-je6cc)

여기서 <mark>CountContext</mark>를 export 하지 않습니다. 이것은 의도한 겁니다. 저는 context 값을 제공하는 방법과 이를 소비하는 방법 한 가지씩만 사용합니다.
이를 통해 사람들이 context의 값을 그대로 사용하고 있는지 확인하고 제가 만든 consumer들에게 유용한 값들을 제공할 수 있습니다. 

이 글이 도움이 되었으면 좋겠습니다! 기억해주세요:

1. 마주치는 모든 state 공유 문제를 해결하기위해 context를 사용하지 말아주세요.

2. Context는 꼭 앱 전역에서 사용되는 전역상태일 필요가 없습니다 하지만 여러분이 만든 트리 한 부분에서 적용될 수는 있습니다.

3. 앱에서 논리적으로 분리된 여러가지의 context를 가질 수 있습니다.

행운을 빕니다!

원문 : [How to use React Context effectively](https://kentcdodds.com/blog/how-to-use-react-context-effectively)