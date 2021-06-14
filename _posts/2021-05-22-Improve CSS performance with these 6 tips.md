---
layout: post
title: Improve CSS performance with these 6 tips (번역)
---

Improve CSS performance with these 6 tips
===================================================

Introduction
=======================

CSS는 웹사이트 디자인에서 프레젠테이션 계층에 있습니다. 올바르게 사용한다면 사용자의 입장에서 아름다움을 선사하고 HTML 마크업에 맞게 잘 사용되는 느낌을 전달할 수 있습니다. 올바르지 않게 사용한다면, 안좋은 사용자 경험과 사이트의 속도와 성능에 악영향을 끼치는 결과를 발생시킬 수 있습니다.

이 글에서 여러분은 웹사이트에 악영향을 끼치는 일반적인 CSS 작성법들을 어떻게 피할 수 있는지 배우게 될 겁니다. 글이 끝날때 즈음이면 여러분은 더욱 빠르고 사용자 경험에 있어서 향상된 웹사이트를 만들 수 있을겁니다 - 모든 개발자들이 원하는것이죠!

여러분의 코드를 향상시키고 웹사이트의 성능을 올리기 위해 아래 6개의 CSS 주제들을 다룰겁니다. 

1. 간단한 선택자 사용하기 (Write simple selectors)
2. 과도한 애니메이션 자제하기 (Avoid excessive animations)
3. 애니메이션을 언제 사용할지 알기 (Know when to animate expensive properties)
4. @import의 사용 (@import statement)
5. 파일 사이즈 최적화 하기 Optimize your file sizes)
6. base64 비트맵 이미지 사용 피하기 (Avoid base64 bitmap images)

A quick note on why website performance matters
====================================================

웹사이트의 성격 그리고 다양한 브라우저에서 잘 작동하는지와는 관계없이, 웹사이트는 빨리 로드되야 합니다.
만약 로드가 빠르지 않다면, 유저는 다른 웹사이트를 찾아 떠나겠죠.

만약 이런일이 서비스나 상품을 판매하는 여러분의 웹사이트에서 발생한다면 매출의 감소와 연결됩니다. 왜냐하면 인터넷에서는 정말 많은 다양한 선택 옵션들이 존재하고 여러분이 사용자에게 다른곳에 가보라는 명확한 암시를 했기 때문입니다. 더 나아가 이런 사용자들은 다른 사용자에게 여러분의 웹사이트를 비추천 할것이고 결국에는 페이지 조회수의 감소로 이어질겁니다.

이제 시작해보도록 하겠습니다!

1. 간단한 선택자 사용하기 (Write simple selectors)
=========================================================

CSS는 HTML을 선택하고 스타일을 주기 위해 정말 많고 유연한 옵션들을 가지고 있습니다. 수년에 걸쳐, 전문가들은 간단한 선택자들을 사용하라고 개발자들에게 조언했습니다. 브라우저의 부하를 줄이고 코드를 깨끗하고 간결하게 유지하기 위해서 말이죠.

다음에 보여드릴 예시는 단순성이 무엇인지 보여줍니다:
```css
.hero-image {
    width: 70%
}
```

하지만 CSS는 위의 코드를 이렇게도 쓸 수 있게 해줍니다.
```css
main > div.blog-section + article > * {
    /* Code here */
}
```

해당 예시에서, 브라우저는 선택자를 오른쪽부터 왼쪽까지 파싱할겁니다. 전체 선택자인 <mark>*</mark>부터 시작해서 <mark>main</mark> 선택자까지 주욱 읽는거죠. 이런 코드는 브라우저에게 있어 평소보다 일이 더 많아진겁니다. 비록 선택자를 파싱하는데 있어 1000분의 1초밖에 안되는 차이점을 이야기 하고 있지만, 이런 시간들은 선택자 메소드가 여러분의 스타일 시트에서 여러번 발생할 때 더해지게 됩니다.

더군다나, 선택자가 길어지면 길어질수록 스타일 시트의 전체적인 크기에 더 많은 바이트(bytes)가 더해집니다. 

2. 과도한 애니메이션 자제하기 (Avoid excessive animations)
==============================================================

이제 CSS에서 자체적으로 애니메이션 사용이 가능하므로 웹사이트에 애니메이션 효과를 넣어주기 위해 자바스크립트를 사용하지 않아도 됩니다. 이제 웹사이트에 애니메이션 추가가 좀 더 간편해지게 되었고 이 뜻은, 올바르게만 사용한다면 더 나은 사용자 경험을 만들어 낼 수 있다는 겁니다. 

하지만 과도하게 사용했을 경우, 목적을 가지고 여러분의 웹사이트를 방문한 사용자에게 방해가 될 수 있습니다. 그리고 또 하나 명심해야할 부분은 여러분이 작성한 애니메이션 하나하나마다 파싱하는데 시간이 걸리므로 과도하게 사용할 경우 웹사이트의 속도가 저해되거나 멈출 수도 있습니다. 

3. 애니메이션을 언제 사용할지 알기 (Know when to animate expensive properties)
===============================================================================

간단합니다: 전체적인 페이지를 다시 레이아웃하게 만드는 CSS 속성은 애니메이션을 적용해서는 안됩니다. 이러한 속성들은 일반적으로 "비싸다"라고 하는데요. 왜냐하면 이런 속성들은 웹사이트에 상당한 로드 시간을 걸리게 하기 때문입니다. 몇몇 예시를 살펴보자면:
- <mark>margin</mark>
- <mark>padding</mark>
- <mark>height</mark>
- <mark>width</mark>

이유는 여러분이 예를들어 <mark>margin</mark>과 같은 속성을 바꾸고 단일 DOM 요소들의 차원을 수정한다면 다른 모든 요소들도 바뀌는 요인이 되어버립니다. 

<mark>opacity</mark>와 <mark>transform</mark>과 같은 속성들은 애니메이션을 적용해도 됩니다. 왜냐하면 이런 속성들은 다른 요소들의 레이아웃에 영향을 주지 않으니까요. 이것은 웹 브라우저가 이러한 계산을 GPU에게 넘김으로써 계산을 더욱 빠르게 할 수 있기에 가능합니다. 

특정한 다른 CSS 속성들도 주로 사용되지만 이것들은 그려지는데 더 오래 걸립니다. 이런 속성들은 아래의 내용들을 포함합니다.
- <mark>:nth-child</mark>
- <mark>box-shadow</mark>
- <mark>border-radius</mark>
- <mark>position: fixed</mark>

이런 속성들이 수백개가 스타일 시트에 작성되었다면, 여러분의 웹사이트의 성능에 영향을 미칠겁니다. 이런 속성들을 조금만 아껴서 사용해주세요. 

4. @import의 사용 (@import statement)
========================================

<mark>@import</mark>는 대부분 폰트와 같은 자산들(assets)을 포함시키기 위해 사용됩니다. 그러나 다른 CSS 파일들을 포함시킬 수도 있습니다. CSS는 랜더링 차단방식입니다(render-blocking). 즉, 폰트나 다른 CSS 파일들을 불러오기 위해 <mark>@import</mark>를 사용할때, 브라우저는 자산(asset)을 가져온 후 남은 CSS 코드들을 계속해서 처리합니다. 

```js
    /* styles.css */
    /**
    * The browser would fetch base.css before
    * processing the remaining code in styles.css
    */
    @import url("base.css");
```
자산이 폰트 파일일 때, 브라우저는 폰트를 다운로드 받을 동안 시스템에서 사용 가능한 폰트를 사용합니다. 
다운로드가 완료된 후, 브라우저는 시스템에서 사용하고 있는 폰트를 다운받은 폰트로 바꿉니다. 그러므로, 사용자는 사이트의 컨텐츠를 읽을 때, 갑자기 폰트가 달라지는 현상을 겪게됩니다. 이것은 사용자 경험상 좋지 않습니다. 

아래는 <mark>@import</mark>로 폰트를 로드하는 예시입니다. 

```js
    /**
    * Example of loading a font with the
    * @import statement.
    * The font is only available after it downloads.
    */
    @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@100&display=swap');
```
<mark>@import</mark>를 사용하는 대신, 폰트를 로드하기 위해 HTML <mark>head</mark>태그에 <mark>link</mark>태그 삽입을 권장합니다.

```html
    <link rel="preload" as="font" href="https://fonts.googleapis.com/css?family=Open+Sans" crossorigin="anonymous">
```

여기 link 태그에서 <mark>rel="preload"</mark>과 <mark>as="font"</mark>는 브라우저에게 최대한 빨리 폰트를 다운로드 하라고 알려줍니다. 또한 사용자가 동일한 폰트를 다운로드를 함으로써 발생되는 대역폭 낭비를 방지하기 위해 미리 로드된 폰트 파일이 CSS의 폰트 파일과 일치하는지 확인할 수 있습니다. 

5. 파일 사이즈 최적화 하기 Optimize your file sizes)
======================================================

웹 디자인과 개발에서, 크기는 중요합니다. 이미지나 HTML 아니면 자바스크립트 파일, 혹은 다른 미디어 자산들(media assts)을 다룰때, 한가지 황금률이 존재합니다: 항상 압축하는것입니다.

CSS 파일 사이즈를 줄이세요. 여러분의 웹 사이트에 있는 이미지들은 로드되는 속도를 줄이기 위해 최적화 되어야 합니다. 이럴때는 [TinyPNG](https://tinypng.com/)와 같은 온라인 도구를 사용할 수 있죠. 아니면 여러분이 직접 이미지를 만든다면, 포토샵에서 웹용으로 저장하기와 같은 도구를 사용하세요.

6. base64 비트맵 이미지 사용 피하기 (Avoid base64 bitmap images)
=====================================================================

Base64 이미지는 웹페이지에 이미지를 삽입하는 방법중 하나입니다. [Harry Roberts](https://csswizardry.com/)와 같은 전문가들은 왜 base64 이미지가 성능에 좋지 않은지 많은 이유를 보여줬습니다: 
 
- CSS 파일의 전체 크기를 크게 늘립니다.
- 어떻게 사용되는지 여부에 관계없이 다운로드 됩니다.
- Base64로 코드화 된 결과들은 기본 이미지 파일 사이즈보다 큽니다.
- 브라우저는 이미지를 사용하기 전에 base64의 전체 문자열을 파싱해야 합니다.

아래를 보시면, base64로 [전환되기 전과 후](https://www.base64-image.de/)의 이미지 사이즈를 확인하실 수 있습니다. 

![Improve CSS performance with these 6 tips1](/assets/Improve CSS performance with these 6 tips1.jpg)

열 네줄의 코드로 세개의 base64 이미지를 CSS 파일에 추가하면 어떻게 되는지 한번 확인해보세요: 

```css
    /**
    * Base64 code truncated.
    */
    @media screen and (min-width: 20em) {

        html {
        background-image:  url('data:image/png;base64,iVBO ...');
        }
        
        footer {
        background-image:  url('data:image/png;base64,iVBO ...');
        }

        .non-existence-class {
        background-image:  url('data:image/png;base64,iVBO ...');
        }

    }
```

파일의 크기가 500KB를 넘습니다. 크기만 한게 아니라 사용자의 브라우저에서 이미지를 사용유무에 관계없이 다운로드 하는데에 시간이 걸릴겁니다. 

![Improve CSS performance with these 6 tips1](/assets/Improve CSS performance with these 6 tips2.jpg)

한편으로는 아래의 코드에서 브라우저는 뷰포인트에 따라 이미지를 다운로드 합니다.

```js
    html {
        padding: 2em;
        background-image: url("images/asnim_mobile.jpg");
    }

    @media screen and (min-width: 20em) {
        html {
        background-image: url("images/asnim_tablet.jpg");
        }
    }

    @media screen and (min-width: 48em) {
        html {
        background-image: url("images/asnim.jpg");
        background-size: cover;
        }
    }
```

다음 단계들을 수행하며 확인할 수 있습니다.

1. 위의 코드처럼 서로 다른 세개의 브레이크포인트에 서로 다른 크기의 이미지를 만듭니다. 
2. 해당 이미지를 배경 이미지로 사용하는 코드를 복붙합니다.
3. 브라우저를 실행하고 개발자 도구의 네트워크 탭을 열어봅니다.
4. 브라우저의 뷰포트의 사이즈를 조절해보면서 확인해봅니다. 

Conclusion
==============

여기서 배운 내용들을 가지고 미래의 웹 프로젝트에 적용을 해 볼때 더 나은 웹 세계에 기여하는것임을 알아두세요!
만약 여러분이 생각하기에 성능을 개선시켜야 하는 웹사이트가 있다면 위에서 배운 내용들을 가지고 코드 리팩토링을 해보시고 아래 댓글에 어떻게 됐는지 알려주세요.

Further reading
================

추가적으로 CSS와 일반 웹 성능 업데이트에 몇 가지 팁을 찾아보세요:

- [ How to use web fonts in CSS](https://blog.logrocket.com/how-to-use-web-fonts-in-css-a0326f4d6a4d/)
- [Improve site performance by inlining your CSS](https://blog.logrocket.com/improve-site-performance-inlining-css/)
- [Using CDNs to optimize website performance](https://blog.logrocket.com/using-cdns-to-optimize-website-performance/)
- [A guide to improving web accessibility with CSS](https://blog.logrocket.com/a-guide-to-improving-web-accessibility-with-css/)


원문 : [Improve CSS performance with these 6 tips by Habdul Hazeez](https://blog.logrocket.com/improve-css-performance-these-6-tips/)