---
layout: post
title: React Hooks - The good, the bad, and the ugly (번역)
---

훅(Hooks)는 리액트 버전 16.8의 배포와 함께 리액트의 컴포넌트 만드는 방법을 바꿔보겠다는 목표로 등장했습니다. 시간이 조금 지났고 훅은 많은곳에서 사용되고 있습니다. 훅은 성공했을까요?

초반에는 클래스 컴포넌트를 제거하려는 방법으로 훅이 출시되었습니다. 클래스 컴포넌트의 가장 큰 문제는 만드는데 어렵다는것이었죠. <mark>componentDidMount</mark>와 같은 라이프 사이클 이벤트들이 담긴 로직을 [higher-order components](https://reactjs.org/docs/higher-order-components.html)와 [renderProps](https://reactjs.org/docs/render-props.html)와 같은 패턴을 이용해 재사용하는건 과장되게 말하자면 어색한 패턴이었습니다. 훅의 가장 큰 장점은 교차적으로 맡은 부분들을 분리하고 합칠 수 있는 능력에 있다고 할 수 있습니다.

# The good

훅은 상태(state)를 캡슐화 하고 로직을 공유하는것에 특화되어있습니다. [react-router](https://reactrouter.com/web/guides/quick-start)와 [react-redux](https://react-redux.js.org/)와 같은 라이브러리 패키지들은 훅 덕분에 간단하면서도 깨끗한 API를 가질 수 있게되었죠.

아래는 옛날방식의 [connet API](https://react-redux.js.org/api/connect)를 이용해 만든 예시입니다.

```js
import React from "react";
import { Dispatch } from "redux";
import { connect } from "react-redux";
import { AppStore, User } from "../types";
import { actions } from "../actions/constants";
import { usersSelector } from "../selectors/users";

const mapStateToProps = (state: AppStore) => ({
  users: usersSelector(state),
});

const mapDispatchToProps = (dispatch: Dispatch) => {
  return {
    addItem: (user: User) =>
      dispatch({ type: actions.ADD_USER, payload: user }),
  };
};

const UsersContainer: React.FC<{
  users: User[],
  addItem: (user: User) => void,
}> = (props) => {
  return (
    <>
      <h1>HOC connect</h1>
      <div>
        {users.map((user) => {
          return (
            <User user={user} key={user.id} dispatchToStore={props.addItem} />
          );
        })}
      </div>
    </>
  );
};

export default connect(mapStateToProps, mapDispatchToProps)(UsersContainer);
```

위와 같은 코드는 많고 반복적입니다. <mark>mapStateToProps</mark>와 <mark>mapDispatchToProps</mark>를 쓰는건 귀찮기도 하지요.

아래는 훅을 사용해 리팩토링한 코드입니다.

```js
import React from "react";
import { useSelector, useDispatch } from "react-redux";
import { AppStore, User } from "../types";
import { actions } from "../actions/constants";

export const UsersContainer: React.FC = () => {
  const dispatch = useDispatch();
  const users: User[] = useSelector((state: AppStore) => state.users);

  return (
    <>
      <h1>Hooks</h1>
      {users.map((user) => {
        return <User user={user} key={user.id} dispatchToStore={dispatch} />;
      })}
    </>
  );
};
```

차이점은 명확합니다. 훅은 더 깨끗하고 간단한 API를 제공합니다. 또한 모든것을 컴포넌트를 감쌀 필요가 없게하므로 큰 승리입니다.

# The bad

[useEffect 훅](https://blog.logrocket.com/guide-to-react-useeffect-hook/)은 함수를 첫번째 매개변수를 받고 종속성 배열(dependency array)을 두번째 매개변수로 받습니다.

```js
import React, { useEffect, useState } from "react";

export function Home() {
  const args = ["a"];
  const [value, setValue] = useState(["b"]);

  useEffect(() => {
    setValue(["c"]);
  }, [args]);

  console.log("value", value);
}
```

위의 코드는 <mark>useEffect</mark>훅을 끊임없이 호출할겁니다. 바로 아래의 단순한 변수 할당때문에 말이죠.

```js
const args = ["a"];
```

매순간 컴포넌트가 새로 랜더될때, 리액트는 전에 발생했던 랜더로부터 종속성 배열을 계속 복사합니다. 현재의 종속성 배열과 전의 종속성 배열을 비교하는거죠. 각각의 비교되는 요소들은 [Object.is 메소드](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)를 이용해 useEffect가 새로운 값과 함께 다시 작동이 되어야 할지 결정됩니다. 객체들은 값이 아닌 참조에 비교되어집니다. 변수인 <mark>args</mark>는 매 랜더될때마다 새로운 객체가 되어질것이고 전에 있던 객체와 다른 메모리의 주소를 가지고 있겠죠.

갑작스럽게도, 변수 할당이 함정이 될 수 있습니다. 불행하게도 종속성 배열과 관련되어 정말 많은 수의 함정이 존재합니다. 인라인으로 화살표 함수를 만듦으로써 종속성 배열이 이와 같은 운명에 처해질 수 있죠.

해결책은, 물론, 훅을 더 사용하는겁니다:

```js
import React, { useEffect, useState, useRef } from "react";

export function Home() {
  const [value, setValue] = useState(["b"]);
  const { current: a } = useRef(["a"]);
  useEffect(() => {
    setValue(["c"]);
  }, [a]);
}
```

갑자기 표준 자바스크립트 코드를 수많은 [useRef](https://blog.logrocket.com/usestate-vs-useref/), [useMemo](https://blog.logrocket.com/rethinking-hooks-memoization/) 혹은 [useCallback](https://blog.logrocket.com/react-usememo-vs-usecallback-a-pragmatic-guide/)훅으로 감싸기 복잡하고 어색해집니다. [eslint-plugin-react-hooks 플러그인](https://www.npmjs.com/package/eslint-plugin-react-hooks)을 사용해서 코드를 직관적이고 이해하기 쉽게 만들 수 있지만 버그가 아예 없어지지는 않으며 ESLint 플러그인은 필수가 아닌 보충물이어야 합니다.

# The ugly

저는 최근에 리액트 훅인 [react-abortable-fetch](https://github.com/dagda1/cuttingedge/tree/main/packages/react-abortable-fetch)를 만들었습니다. 코드를 보시면 <mark>useRef</mark>, <mark>useCallback</mark>이나 <mark>useMemo</mark>를 <mark>runner</mark>이라는 함수로 감쌌습니다. 그런데 이렇게 하는건 좋은 경험은 아니었습니다.

```js
const runner = useCallback(() => {
  task.current = run(function* (scope) {
    counter.current += 1;
    send(start);

    try {
        for (const job of fetchClient.current.jobs) {
          const {
            fetch: {
              request,
              init,
              contentType,
              onQuerySuccess = parentOnQuerySuccess,
              onQueryError = parentOnQueryError,
            },
          } = job;

          timeoutRef.current = timeout ? timeout : undefined;
```

결과적으로 종속성 배열이 꽤 커졌으며 코드가 변경되게 되면 최신상태로 항상 유지해야했기 때문에 번거롭게 되어버렸습니다.

```js
}, [
    send,
    timeout,
    onSuccess,
    parentOnQuerySuccess,
    parentOnQueryError,
    retryAttempts,
    fetchType,
    acc,
    retryDelay,
    onError,
    abortable,
    abortController,
  ]);
```

결국, 저는 <mark>useMemo</mark>를 사용하여 훅 함수의 반환 값을 기억해야(memoize) 했습니다. 그리고 물론, 또 다른 종속성 배열을 조작해야 했구요:

```js
const result: QueryResult<R> = useMemo(() => {
    switch (machine.value as FetchStates) {
      case 'READY':
        return {
          state: 'READY',
          run: runner,
          reset: resetable,
          abort: aborter,
          data: undefined,
          error: undefined,
          counter: counter.current,
        };
      case 'LOADING':
        return {
          state: 'LOADING',
          run: runner,
          reset: resetable,
          abort: aborter,
          data: undefined,
          error: undefined,
          counter: counter.current,
        };
      case 'SUCCEEDED':
        return {
          state: 'SUCCEEDED',
          run: runner,
          reset: resetable,
          abort: aborter,
          data: machine.context.data,
          error: undefined,
          counter: counter.current,
        };
      case 'ERROR':
        return {
          state: 'ERROR',
          error: machine.context.error,
          data: undefined,
          run: runner,
          reset: resetable,
          abort: aborter,
          counter: counter.current,
        };
    }
  }, [machine.value, machine.context.data, machine.context.error, runner, resetable, aborter]);
```

# Execution order

["훅의 규칙"](https://reactjs.org/docs/hooks-rules.html)에 나와있듯이 훅은 각각 같은 순서로 실행이 되어야 합니다.

_훅을 반복문이나 조건문 혹은 중첩된 함수 안에서 사용하지 마세요_

리액트 개발자들이 이벤트 핸들러 안에서 훅이 실행이 되어 질 수 있다는것을 예상못한건 좀 이상합니다.

일반적인것은 순서에서 벗어나 실행이 될 수 있는 훅에서 함수를 반환하는 것입니다.

```js
const { run, state } = useFetch(`/api/users/1`, { executeOnMount: false });

return (
  <button
    disabled={state !== "READY"}
    onClick={() => {
      run();
    }}
  >
    DO IT
  </button>
);
```

# The verdict

전에 언급한 <mark>react-redux</mark>코드의 단순화는 매력적이고 결과적으로 코드를 간소화 합니다.
훅은 이전의 코드보다는 적은 코드를 필요로 하며 이렇게 함으로써 쉽게 만들 수 있게 합니다.

훅의 장점은 단점을 압도합니다만, 훅의 완전한 승리는 아닙니다. 훅은 우아하고 영리한 아이디어지만, 이것을 [실제로 사용하기 위해서는 조금 힘들 수 있습니다](https://blog.logrocket.com/react-hooks-frustrations/).
수동적으로 종속 그래프를 조절하고 적절한 장소를 기억하는건(memoize) 아마도 대부분 문제의 근원이 되는것이며, 훅을 사용하는데 있어 다시 한번 생각하게 합니다. 실행을 일시적으로 중지하고 다시 시작할 수 있는 생성자 함수의 사용이 더 나을 수도 있습니다.

클로저(Closures)는 문제를 이해하는데에 키가 될 수도 있고 함정이 될 수도 있습니다. 오래된 클로저는 최신에 업데이트가 되지 않은 변수를 참조할 수 있습니다. 클로저의 지식은 훅을 처음 사용하는데에 있어 장애물이며 이것을 알고 디버깅할 수 있는 능력이 필요합니다.

원문: [React Hooks: The good, the bad, and the ugly by Paul Cowan](https://blog.logrocket.com/react-hooks-the-good-the-bad-and-the-ugly/)
