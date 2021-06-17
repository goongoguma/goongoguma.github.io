---
layout: post
title: 10 Useful JavaScript Coding Techniques That You Should Use (번역)
---

현재 자바스크립트는 다양한 곳에서 사용되고 있습니다. 자바스크립트를 사용하는 개발자로서, 지금 당장은 다른 언어를 배울 이유는 없습니다. 자바스크립트를 사용해 거의 모든 것을 다 할 수 있으니까요(웹 개발, 모바일, 데스크탑 등등). 거기다가, 저는 자바스크립트를 정말 사랑합니다. 왜냐하면 이 언어는 친근하면서도 강력하기까지 하니까요.

자바스크립트의 생태계는 정말 거대하죠. 개발을 할 때 시간을 절약하기 위해 사용이 가능한 정말 많은 라이브러리와 프레임워크들이 있습니다. 하지만 시간이 충분하다면 라이브러리를 사용하는 대신에 직접 만들어 보는것이 더 낫습니다. 

다행스럽게도, 자바스크립트는 여러분이 코딩 목표를 쉽게 달성하기 위해 사용할 수 있는 많은 특징과 기술을 가지고 있습니다. 그래서 이 글에서는, 여러분이 개발자로서 사용할 수 있는 간단한 자바스크립트 코딩 기술을 살펴보도록 하겠습니다.

# 1. Get the last element of an array

자바스크립트를 이용해서 코딩을 하다보면 배열의 마지막 요소를 구해야 하는 경우가 많이 있습니다. 이렇게 하기 위해서는 요소에 접근할 때, 배열의 length 속성을 사용하면 됩니다. 

위의 예시입니다:

```js
  let numbersArr = [4, 8, 9, 34, 100];
  numbersArr[numbersArr.length - 1]; //return 100
```

배열의 인덱스는 0에서부터 시작하기 때문에, 마지막 요소에 접근하기 위해서 배열의 length에 -1을 해줍니다.

# 2. Random number in a specific range

아시다시피, 0과 1사이의 숫자를 무작위로 반환하기 위해 <mark>Math.random()</mark> 메소드를 사용합니다. 

하지만 가끔씩, 특정 범위내의 무작위 숫자를 구해야 하는 경우가 있죠. 예를 들어, 0이나 100사이의 무작위 숫자를 구해야 하는것처럼요.

위의 예시입니다.

```js
  // Random number between 0 and 4.
  Math.floor(Math.random() * 5);
  // Random number between 0 and 49.
  Math.floor(Math.random() * 50);
  // Random number between 0 and 309.
  Math.floor(Math.random() * 310);
```

# 3. Flattening multi-dimensional array

ES2017에서 개발 된 <mark>flat()</mark> 메소드를 사용해서 다중 배열을 쉽게 평면화 할 수 있습니다.

<mark>flat()</mark> 메소드는 배열을 얼만큼 평면화 하고싶은지 정의하는 하나의 매개변수(평면화의 레벨)를 받습니다.

위의 예시입니다:

```js
  let arr = [5, [1, 2], [4, 8]];
  arr.flat(); //returns [5, 1, 2, 4, 8]
  let twoLevelArr = [4, ["John", 7, [5, 9]]]
  twoLevelArr.flat(2); //returns [4, "John", 7, 5, 9]
```

# 4. Check for multiple conditions

하나의 if로 여러개의 조건문을 확인하길 원한다면 배열 메소드인 <mark>includes()</mark>를 사용하는 편이 좋습니다.

위의 예시입니다:

```js
  let name = "John";
  //Bad way.
  if(name === "John" || name === "Ben" || name === "Chris"){
    console.log("included")
  }
  //Better way.
  if(["John", "Ben", "Chris"].includes(name)){
    console.log("included")
  }
```

보시다시피, 위 예시에서의 <mark>includes()</mark> 메소드는 코드를 깔끔하게 만들기 위한 사용하는 축약입니다. 이렇게 사용하는 것을 추천합니다.

# 5. Extract unique values

가끔은 배열 안에서 중복되는 요소를 제거해야하는 경우가 있습니다.

중복되는 요소를 제거하기 위해서, 자바스크립트의 전개 구문(spread operator)과 함께 set 객체를 사용하면 됩니다.

위의 예시입니다:

```js
  const languages = ['JavaScript', 'Python', 'Python', 'JavaScript', 'HTML', 'Python'];
  const uniqueLanguages = [...new Set(languages)];
  console.log(uniqueLanguages);
  //prints: ["JavaScript", "Python", "HTML"]
```

# 6. Run an event only once

만약 이벤트가 딱 한번 만 발생하길 원한다면, <mark>addEventListener()</mark>의 세번째 매개 변수에 <mark>once</mark> 옵션을 사용할 수 있습니다.

위의 예시입니다:

```js
  document.body.addEventListener('click', () => {
    console.log('Run only once');
  }, { once: true });
```

위의 예시와 같이, 옵션을 <mark>true</mark>로 설정했으며 이벤트는 단 한번만 실행됩니다. 

# 7. Sum all numbers in an array

배열안에 있는 모든 숫자를 더하는 가장 쉬운 방법은 자바스크립트의 reduce 메소드를 사용하는 겁니다. 

위의 예시입니다:

```js
  let numbers = [6, 9 , 90, 120, 55];
  numbers.reduce((a, b)=> a + b, 0); //returns 280
```

만약 reduce에 대해 더 배우기를 원하신다면 제가 쓴 아래의 글을 확인해보세요.

[The Reduce Method in JavaScript Explained With Examples](https://javascript.plainenglish.io/the-reduce-method-in-javascript-explained-with-examples-6232b85e47f6)

<mark>8. Sum numbers inside an array of objects</mark>
=============================================

객체 형태로 이루어진 배열에서 안에 있는 숫자들을 더하는 방법은 일반 배열에서 사용하는 방법과 다릅니다.

이 상황에서 <mark>reduce()</mark> 메소드는 여전히 사용할 수 있지만, reduce 콜백 안에서 객체를 반환해야 합니다. 

아래의 예시는 나이와 이름 프로퍼티로 이루어진 유저 객체의 배열입니다. 여기서 모든 나이의 값을 더한 총합을 구하고 싶습니다: 

```js
  const users  = [
    {name: "John", age: 25},
    {name: "Chris", age: 20},
    {name: "James", age: 31},
    ]
    
    users.reduce(function(a, b){
      return {age: a.age + b.age}
    }).age;
  //returns 76.
```

보시다시피, 반환된 객체의 나이 프로퍼티에 접근할 수 있습니다. 그런 다음 전체 나이의 숫자를 포함하고 있는 객체를 반환하기 때문에 reduce 메소드에 다시 접근합니다. 

# 9. The keyword “in”

예를들어 객체안에 프로퍼티들이 정의가 되었거나 배열안의 요소가 포함되었는지 확인하기 위해서 자바스크립트의 <mark>in</mark> 키워드를 사용하세요. 

위의 예시입니다: 

```js
  const employee = {
    name: "Chris",
    age: 25
  }
  "name" in employee; //returns true.
  "age" in employee;  //returns true.
  "experience" in employee; //retuens false.
```

# 10. From number to an array of digits

숫자들을 숫자들의 배열로 바꾸기 위해 전개 구문, map 메소드, 그리고 <mark>parseInt()</mark> 메소드를 사용할 수 있습니다.

위의 예시입니다:

```js
const toArray = num => [...`${num}`].map(elem=> parseInt(elem))

toArray(1234); //returns [1, 2, 3, 4]
toArray(758999); //returns [7, 5, 8, 9, 9, 9]
```

보시다시피, 빈 배열 안에서 <mark>num</mark> 매개 변수에 전개 구문을 아용하며 map 메소드로 하나하나의 요소들을 <mark>parseInt()</mark>로 사용해 숫자로 만들고 있습니다. 

Conclusion
============

이 내용들이 제가 여러분들과 공유한 간단한 자바스크립트 코딩 기술입니다. 항상 이 기술들이 필요한 상황이 있을겁니다. 

읽어주셔서 감사합니다. 이 내용들이 유용했기를 바랍니다.

원문 : [10 Useful JavaScript Coding Techniques That You Should Use by Mehdi Aoussiad](https://javascript.plainenglish.io/10-useful-javascript-coding-techniques-that-you-should-use-e8e7960e08ed)
