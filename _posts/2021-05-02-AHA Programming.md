---
layout: post
title: AHA Programming (번역)
---

_DRY의 위험성, WET의 거미줄, AHA의 대단함_

[Watch my talk: AHA Programming](https://youtu.be/wuVy7rwkCfc)

DRY
==============================
[DRY(Don't Reapet Yourself)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)은 오래된 소프트웨어의 원칙으로써 위키피디아는 다음과 같이 요약하고있습니다.

_Every piece of knowledge must have a single, unambiguous, authoritative representation within a system _

DRY가 일반적으로 좋은 관행이라는 부분에 있어서 저는 동의합니다(비록 위에서 말한 DRY의 정의가 권장하는것보다는 덜 독단적인것 같지만요). [코드 복사](https://en.wikipedia.org/wiki/Duplicate_code)(복사/붙여넣기로 알려져있고, 기본적으로는 DRY 원칙과는 반대되죠)를 하면서 겪었던 가장 큰 문제는 하나의 코드에서 버그를 발견하고 수정하게되면 같은 버그가 다른 곳에서도 발견된다는 것이고 또 다시 그곳도 고쳐야 한다는 것이었죠.

한번은 많이 사용되기에 복사 붙여넣기 방식으로 만들어졌던 코드를 인수인계 받은적이 있었는데요 무려 다른 여덞곳에서 버그를 고쳐야했던적이 있었습니다!😱 그 귀찮음이란! 그 코드를 함수로 만들어서 필요한 곳에서 호출되게 만들었더라면 굉장히 도움이 되었을텐데요.

WET
===============================
"Write Everything Twice"의 줄임말로 알려진 WET이라는 또 다른 프로그래밍 다른 개념이 있습니다. 이 개념도 비슷하게 독단적이고 좀 권위적입니다. [Conlin Durbin](https://twitter.com/CallMeWuz)은 다음과 같이 정의했습니다.

_You can ask yourself "Haven't I written this before?" two times, but never three._

위에서 제가 말씀드렸던 코드베이스에서 코드복사보다 좀 더 안좋은, 과한 abstraction이 있었습니다. AngularJS의 코드였고 여러 AngularJS의 컨트롤러에서 this를 함수에 전달해주는 코드가 있었는데 이 코드는 컨트롤러 인스턴스에 기능을 더 추가하기 위해 this에 (프로토타입에다가) 메소드와 프로퍼티들을 덕지덕지 붙이는 일을 하고있었습니다. 아마 프로토타입의 상속기능을 흉내 내본거라고 생각합니다. 그런데 이런 코드는 __굉장히__ 복잡하고 이해하기도 어려워서 코드를 수정하는게 겁이 났습니다. 

위의 코드는 세군데 이상에서 재사용이 되고 _있었는데요_, 여기에서는 코드 abstraction은 좋지 않았고 차라리 복사 붙여넣기를 사용했으면 했었네요.

AHA
===============================
AHA는 [Cher Scarlett](https://twitter.com/)이라는 개발자에게서 찾은건데요 아래의 뜻을 말합니다.

_Avoid Hasty Abstractions_

제가 생각하기에 위의 개념은 아래의 개념을 설명한 [Sandi Metz](https://twitter.com/sandimetz)가 잘 서술한 것 같습니다.

_prefer duplication over the wrong abstraction_

이 개념은 굉장히 중요하기에 다시한번 읽어주셨으면 합니다. 그리고 Sandi의 블로그에 들어가서 해당 개념을 설명한 블로그 포스트: [The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)을 읽어보세요.

그리고 저는 여기에 또 다른 중요한 개념을 덧붙이고 싶네요.

_Optimize for change first_

제가 생각하기에 중요한 부분은 미래에 코드가 어떻게 될지 모른다는거죠. 일주일동안 퍼포먼스의 향상을 위해 코드를 최적화 할 수도 있고 새로운 추상화(code abstraction)을 위해 다음날 잘못됐다는걸 깨달아서 처음부터 다시 만들어야 할 API를 생각해 낼 수도 있으며 만들어진 코드에서 어떤 한 부분이 더 이상 필요 없어질수도 있죠. 확실한 것은 상황은 아마도 바뀔것이고 만일 바뀌지 않는다면 코드를 수정할 필요가 없다는거죠. 그래서 코드가 어떻게 보이든 무슨 상관이죠?

자, 오해하지 말아주세요. 저는 코드를 난장판으로 만들라는 이야기가 아닙니다. 제가 말씀드리고자 하는건 작성된 코드들이 미래에 어떻게 바뀌어야 할지 모른다는것을 염두해 두어야 한다는겁니다.

그래서 저는 코드 복사에 대해 괜찮다고 생각합니다. 해당 코드가 어떻게 사용되고 작동되는지 알기만 한다면 말이죠.
함수에 맞는 인자를 만드는 코드는 어디가 다를까? 코드가 동작하는 여러 부분들을 보다보면 공통성이라는 녀석은 코드를 공통적으로 사용하라고 소리칠거고 그때가 바로 code abstraction을 하기 위한 타이밍입니다. 

그런데 만약에 code abstraction을 일찍 사용했더라면 만들어진 함수나 컴포넌트가 사용성을 위해 완벽하다고 생각이 들것이고 새로운 케이스가 발생하게 된다면 만들어진 코드를 수정만 하면 되겠죠. 그런데 이건 만들어진 어플리케이션 전체가 abstraction(이라 읽고 조건문과 반복문으로 떡칠됨)이 될때까지 계속 될겁니다.😂 😭

몇 년전에 한 회사에서 코드 리뷰를 위해 고용된적이 있습니다. 그때에 저는 코드의 abstraction을 위해 [jsinspect](https://github.com/danielstjules/jsinspect)라고 해서 복붙한 코드를 확인해주는 라이브러리를 사용했습니다.
정말로 많은 중복된 코드들이 있었고 저의 관점에서 abstraction이 어떻게 되어야 할지 한눈에 보였죠.

"AHA Programming"에 있어 __제가 생각하기에 중요한 것은__ abstraction을 고집하지 말라는 것입니다. 
abstraction은 이렇게 사용하는 것이 옳다고 생각할때 사용하세요 그리고 abstraction을 사용하는 것이 맞다고 생각할때까지는 중복되는 코드를 무서워 하지 마세요.

Conclustion
============================
저는 신중하고 실용적인 소프트웨어 원칙의 접근은 중요하다고 생각합니다.
그래서 저는 AHA를 DRY나 WET보다도 선호합니다. AHA는 코드 작성자가 함수나 모듈 코드를 abstract할때 조금 더 신경 쓰도록 의도합니다. 

이 글이 도움이 됐으면 좋겠습니다. 만약 코드를 작성할때 나쁜 abstraction의 수렁에 갇혔다면 힘내세요!
Sandi가 그 수렁에서 벗어나도록 발판을 줄테니까요. [이 글을 읽어보세요](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction).

행운을 빕니다!

원문 : [AHA Programming by Kent C. Dodds](https://kentcdodds.com/blog/aha-programming)




