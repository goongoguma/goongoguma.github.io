---
layout: post
title: 합성컴포넌트 패턴에서의 점 표기법(번역)
---

[원문출처](https://medium.com/risan/react-component-with-dot-notation-7a9853dbf33b)


이 글은 [스택오버플로우](https://stackoverflow.com/questions/49256472/react-how-to-extend-a-component-that-has-child-components-and-keep-them/49258038#answer-49258038)에 올라온 질문에 내가 답한 내용이다.

점 표기법을 이용해 접근이 가능한 리액트 컴포넌트는 어떻게 만들어질수 있을까? 

아래의 코드를 살펴보자. Menu 컴포넌트가 있고 자식 컴포넌트인 Menu.Item이 있다. 

```react
const App = () => (
  <Menu>
    <Menu.Item>Home</Menu.Item>
    <Menu.Item>Blog</Menu.Item>
    <Menu.Item>About</Menu.Item>
  </Menu>
);
```

점 표기법으로 접근이 가능한 하위 컴포넌트를 가진 Menu 컴포넌트를 만들수 있을까?

사실 이것은 흔한 패턴이다. 그리고 사실 하위 컴포넌트가 아닌 하나의 컴포넌트에 붙어있는 다른 하나의 컴포넌트일뿐이다. 

예시를 위헤 Menu컴포넌트를 menu.js파일에서 사용해보겠다.

첫번째로 두개의 컴포넌트를 각각 해당 모듈파일에서 정의해보겠다.

```react
// menu.js
import React from 'react';

export const MenuItem = ({ children }) => <li>{children}</li>;

export default const Menu = ({ children }) => <ul>{children}</ul>;
```

이것은 간단한 함수형 컴포넌트다. Menu는 ul태그인 부모 컴포넌트이고 MenuItem은 해당 컴포넌트의 자식 컴포넌트다. 그렇다면 이 두개의 컴포넌트를 아래와 같이 사용할수 있다.

```react
import React from 'react';
import { render } from 'react-dom';
import Menu, { MenuItem } from './menu';

const App = () => (
  <Menu>
    <MenuItem>Home</MenuItem>
    <MenuItem>Blog</MenuItem>
    <MenuItem>About</MenuItem>
  </Menu>
);

render(<App />, document.getElementById('root'));
```

점 표기법은 어디있을까? MenuItem 컴포넌트를 점 표기법으로 접근가능하게 만들어주기 위해, MenuItem 컴포넌트를 Menu 컴포넌트의 static 프로퍼티로 붙여줄수있다.

그렇게 하기 위해서, Menu 컴포넌트를 함수가 아닌 클래스 컴포넌트로 바꿔주겠다.

```react
// menu.js
import React, { Component } from 'react';

export default const MenuItem = ({ children }) => <li>{children}</li>;

export default class Menu extends Component {
  static Item = MenuItem;

  render() {
    return (
      <ul>{this.props.children}</ul>
    );
  }
}
```
이제 MenuItem 컴포넌트를 불러오기 위해 점 표기법을 사용할 수 있다.

```react
import React from 'react';
import { render } from 'react-dom';
import Menu from './menu';

const App = () => (
  <Menu>
    <Menu.Item>Home</Menu.Item>
    <Menu.Item>Blog</Menu.Item>
    <Menu.Item>About</Menu.Item>
  </Menu>
);

render(<App />, document.getElementById('root'));
```

이제 MenuItem 컴포넌트의 정의를 Menu 클래스 컴포넌트에서 직접적으로 정의할수 있다.

```react
import React, { Component } from 'react';

export default class Menu extends Component {
  static Item = ({ children }) => <li>{children}</li>;

  render() {
    return (
      <ul>{this.props.children}</ul>
    );
  }
}
```


<!-- ## Google Analytics

Specify `ga_analytics` in your `_config.yml` and restart the server to add Google Analytics tracking code. -->

<!-- ```ruby
# Google Analytics example
ga_analytics: UA-000000-0
``` -->
