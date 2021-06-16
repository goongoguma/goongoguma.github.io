---
layout: post
title: Fix the slow render before you fix the re-render (번역)
---

어떻게 하면 리액트앱의 랜더링을 최적화 할 수 있을까요?

성능(Performance)은 진지한 이슈죠 그리고 개발자는 앱의 성능을 최대한 빠르게 만들 필요가 있습니다.
이것은 최적화의 효율성뿐만 아니라 코드의 복잡성(미래에 얼마나 빨리 개선되고 바뀔 수 있는지)에도 큰 영향을 미칩니다.

리액트에서 최적화를 말할 때, 많이 나오는 이야기가 '리랜더링(re-rendering)'의 최적화입니다. 
우리가 같은 것을 생각하고 있는지 한번 확인해볼까요?

<iframe src="https://codesandbox.io/embed/musing-franklin-4ewgg?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="musing-franklin-4ewgg"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

버튼을 클릭할때마다 리랜더링이 발생하게됩니다. 그런데 과연 "리랜더링"이란 무엇일까요?

What is a re-render?
===========================

리액트가 세상에 처음으로 나왔을때, 많은 사람들은 리액트의 "가상 돔(Virtual DOM)"덕분에 기존의 UI 라이브러리들보다는 성능 향상에 집중했습니다. 그 당시 가장 유명했던 UI 라이브러리의 대부분은 DOM을 직접적으로 업데이트하거나 DOM을 업데이트 하지만 업데이트가 필요한 모든 컴포넌트를 순차적으로 업데이트하는 방식이었습니다. 다시 말씀드리자면 이렇게 요약될 수 있습니다.

1. DOM을 업데이트함에 있어서 느리다(element.appendChild(childElement)을 호출하는것처럼요).

2. 1번이 많이 할수록 성능에 이슈가 생긴다.

3. 어떤 성능 이슈들은 필요한 업데이트를 한번 함으로써 피할 수 있다.

4. 만약 DOM 업데이트를 일괄적으로 처리한다면 연속적으로 DOM을 업데이트 할 때 발생했던 성능 이슈를 줄일 수 있다. 

그래서 리액트 팀은 일괄적 DOM 업데이트를 하기로 결정했습니다. 그래서 만약 30번의 DOM 업데이트가 필요한 상태변경이 일어났을 경우, 하나하나 업데이트 되는것이 아닌 한번에 DOM이 업데이트가 됩니다. 이런 일괄적인 업데이트를 하기 위해서, DOM 업데이트에 대한 주도권이 필요하므로 DOM이 어떻게 생겼는지 설명하는 React.createElement((JSX가 무엇)[https://kentcdodds.com/blog/what-is-jsx]인지 알려주는)가 있고, 리액트는 요소들(elements)을 DOM에 랜더하기 위해 다시 한번 함수를 호출합니다. 그리고 나서 전에 마지막으로 랜더했던 리액트의 요소들과 새로 만들어진 리액트의 요소들을 비교합니다. 여기에서 리액트는 DOM에게 어떤 부분이 업데이트가 되어야 하는지 알려주고, 최대한 최적화된 상태에서 업데이트를 진행합니다. DOM을 업데이트 하는 과정을 "committing"이라고 불립니다. 왜냐하면 사용자가 "랜더"한 리액트의 요소들을 DOM에 커밋하고 있기 때문입니다. 

이 부분은 다른 굉장히 중요한 리액트의 특별함이고 잊지 말았으면 합니다(이름에서 살짝 오해의 소지가 있으니 제대로 이해하고 가기를 바랍니다). "랜더(render)"는 리액트에서 리액트의 요소들을 얻기 위해 함수를 호출할 때 발생합니다. 
"조화(Reconfiliation)"는 리액트가 호출한 요소들과 그 전에 랜더되었던 리액트 요소들을 비교했을 때 발생합니다. 
"커밋(commit)"은 리액트가 요소들의 차이점을 가지고 DOM을 업데이트 했을 때 발생합니다. 

```html
render → reconciliation → commit
      ↖                   ↙
           state change
```

Unnecessary re-renders
=================================

단지 컴포넌트가 다시 랜더된다고해서, DOM 업데이트가 발생한다는 것은 아닙니다. 아래 예시를 볼까요:

```js
  function Foo() {
    return <div>FOO!</div>
  }
  function Counter() {
    const [count, setCount] = React.useState(0)
    const increment = () => setCount(c => c + 1)
    return (
      <>
        <Foo />
        <button onClick={increment}>{count}</button>
      </>
    )
  }
```
버튼을 누를때마다 Foo 함수가 호출이 되지만 그것을 나타내는 DOM은 다시 랜더되지 않습니다. 이런 이유로 해당 컴포넌트에서는 DOM 업데이트가 발생하지 않죠. 이것을 흔히 "불필요한 리랜더(unnecessary re-render)"라고 불립니다. 

불행하게도, "랜더"와 "커밋"의 차이점에서 많은 혼란이 발생하고 있습니다. 많은 사람들은 "DOM은 느리다"라고 알고 있습니다(적어도 한번쯤은 들어봤을겁니다). 그러나 대부분은 컴포넌트가 리랜더링이 된다고 해서 돔이 업데이트 되는것은 아니다 라는건 모르고 있죠. 바로 이런 오해 때문에, 컴포넌트 리랜더링이 사실적으로 DOM 업데이트가 필요하지 않을때 컴포넌트가 랜더됨으로써 성능에 장애물이 된다고 믿고있습니다. 

이런 오해가 어떤 케이스에서는 확실한 문제가 될 수 있습니다만 일반적으로 모바일 브라우저와 같은 작은 디바이스에서도 객체를 생성하고(render phase) 생성된 객체를 비교하는건(reconsiliation phase) 굉장히 빠르죠. 그래서 리랜더링의 문제점이 무엇일까요?

Slow renders
==================

자바스크립트가 "랜더"와 "조화" 단계를 매우 빠르게 다룬다는것을 생각해 볼때, 왜 나의 앱은 불필요한 리랜더링이 발생할때 멈추는걸까요? 이런 상황에서 문제는 아마도 불필요한 리랜더링 때문이라고 말해드릴 수 있으나 일반적으로 느린 랜더링 속도때문에 발생할 확률이 더 높습니다. 랜더되는 과정에서 코드가 무엇인가를 하기 때문에 속도를 느리게 만든다는거죠. 이 문제가 어디서 발생하는지 찾고 고치는게 급선무입니다. 문제를 고치고 나서 앱을 다시 실행시키고 불필요한 리랜더링 이슈가 아직 남아있는지 확인해보세요. 

사실상, 만약에 느린 랜더링문제를 고치지 않고 리랜더링 되는 횟수만을 줄이게 된다면 상황은 더 악화되고 코드는 더 복잡해 지겠지요.

이 부분을 제가 말하고 싶은 걸 수도 있습니다. 만약 눈을 감을때마다 내 얼굴을 때려야 한다고 생각해보세요 😉 🤛 🥴. 아마 이렇게 생각이 될겁니다: "눈을 최대한 깜빡이지 않는게 좋겠다!" 무슨 말인지 이해했나요? 저는 당신이 눈을 감을때마다 얼굴에 주먹질을 그만해야한다고 말했죠! 그러니까 나쁜일(느린 랜더)이 발생하는 일을 줄이는것보다 나쁜일을 아예 없애고 원할때마다 눈을 깜빡(랜더)이는거죠.

How to fix slow renders
============================

그래서 느린 랜더링들(slow renders)을 고치는것이 우선이라는 결론을 냈습니다. 그 후에 리랜더링이 문제가 되는지 결정할수 있죠. 그래서 어떻게 하면 느린 랜더링를 고칠 수 있을까요? 
사용자 경험에 있어서 어느 부분이 안좋은 문제를 일으키는지 알 수 있을겁니다. 주로 탭을 열거나 버튼을 클릭하거나 텍스트 필드에 텍스트를 입력할때 발생하죠.
여기서 이렇게 하시면 됩니다: 브라우저의 개발자 도구를 열고 performance 탭을 열어 앱이 어떻게 동작하는지 알아보세요. 앱과 상호작용을 해보시고 다시 멈춰보세요. 예를 들어: 

<div class="center">
  <blockquote class="twitter-tweet"><p lang="en" dir="ltr">Quickly determine JavaScript performance bottlenecks of interactions in your app via the Performance tab of <a href="https://twitter.com/ChromeDevTools?ref_src=twsrc%5Etfw">@ChromeDevTools</a>.<br><br>There&#39;s a LOT more to this stuff, but learning how to use these tools is super helpful! <a href="https://t.co/UbTR6ZUUNT">pic.twitter.com/UbTR6ZUUNT</a></p>&mdash; Kent C. Dodds (@kentcdodds) <a href="https://twitter.com/kentcdodds/status/1171158009277403136?ref_src=twsrc%5Etfw">September 9, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

일단 어떤 부분(아니면 설치된 dependency)에서 제일 오래 걸리는 부분을 찾게되면 해당 문제를 고치고 나서 다시 한번 검사해보세요. 그리고 향상된(아니면 반대로) 성능을 확인해보세요. 그리고 리액트 DevTool도 잊지마세요. 정말 좋답니다!
<div class="center">
  <blockquote class="twitter-tweet"><p lang="en" dir="ltr">⚛️🛠 Prototype of a new Profiler feature, &quot;Scheduled by&quot;, enumerating which fibers triggered the current commit (which ones called set state).<br><br>Would this be useful? Could it be more useful? <a href="https://t.co/7AvVHB0wPY">pic.twitter.com/7AvVHB0wPY</a></p>&mdash; Brian Vaughn 🖤 (@brian_d_vaughn) <a href="https://twitter.com/brian_d_vaughn/status/1126950967201546240?ref_src=twsrc%5Etfw">May 10, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

Conclusion
==============

랜더링이 100%가 필요한지는 중요하지 않습니다. 만약 랜더링되는 속도가 느리다면 사용자 경험에 있어서 나쁜 경험을 만들겁니다.
눈을 감을때마다 얼굴에 주먹질을 하지 마세요. 느린 랜더링의 원인을 먼저 고치세요. 그런 다음에 불필요한 리랜더링(만약 필요하다면)을 고치세요. 행운을 빕니다!

원문: [Fix the slow render before you fix the re-render by Kent C. Dodds](https://kentcdodds.com/blog/fix-the-slow-render-before-you-fix-the-re-render)