---
layout: post
title: "jQuery 탈출기"
date: 2017-08-17 00:00:00 +0900
categories: javascript
tags: javascript
author: Myoungho Pak
---

최근에 `Plain Bagel`이란 팀에 프리랜서로 일하게 되면서 자막 에디터를 만들게 되었다. jQuery만을 이용해 작업하였는데 이를 javascript로
다시 작성하면서 느낀점을 적어본다.

elixir 기반의 Phoenix framework를 사용해서 만든 어드민이 있었고. 그 어드민에 자막 에디터를 붙여야하는 상황이었다.
어찌저찌 기술 스택을 고르다보니 프로젝트 하나를 몽땅 jQuery로 만들게 되었는데.
다 만들고나니 jQuery보다 javascript가 더 성능이 좋다는 얘기를 어디서 들은 적이 있어서 개발 기한도 많이 남았겠다.
jQuery를 걷어내고 javascript로 프로젝트를 재작성해보았다.

이 작업을 진행하면서 내가 원했던 것은 `성능 향상`이었다.

jQuery가 그리 느린건 아니었지만 불러오는 자막의 길이가 길면 길수록 DOM을 랜더링하는 시간이 늘어나서 문제가 있었다.
자막이 700개가 넘어가면 한 4초 정도 걸리는데, 700개라는 상황은 절대 오지 않을만 한 상황이었긴 하지만 그냥 호기심이 생겼다.
그래서 자바스크립트로 바꾸면 이를 해결할 수 있지 않을까하는 막연한 짐작으로 작업을 시작하게되었다.

# TL;DR

1. 성능의 개선을 목적으로 했지만 실패했다.
2. jQuery를 완전히 걷어내는 것은 힘들었다. 어느정도 jQuery를 혼용해서 사용하는 부분이 있는데,
  - jQuery를 이용하는게 더 빠른 경우.
  - 사용한 라이브러리의 디펜던시가 jQuery라서 어쩔 수 없는 경우.
3. 생각보다 jQuery 없이 짜는 것이 어렵지 않았다.
  - 대부분 바꿔야하는 것은 Selector 부분이다.
  - 내가 지금 짠 jQuery 코드를 어떻게 바꿔야할지 보여주는 가이드라인들이 존재했다.

결론적으로, jQuery가 꼭 필요한 상황이 아니라면 Vanilla Javascript로 코드를 작성하는 것도 나쁘지 않은 것 같다.
성능도 더 빠르고 syntax도 그렇게 어렵지 않고 나라면 앞으로 그냥 Javascript를 쓸 것이다. 굳이 jQuery를 불러와 쓸 필요가 없지 않은가.

# 1. 성능의 개선을 목적으로 했지만 실패했다.

일단 어느 함수에서 병목이 생기는지 알아내기위해 프로파일링 도구를 사용해보았다.

![profiling 결과 - 1](/assets/images/profiling-1.png)

프로파일링 결과를 보면 DOM랜더링 하기전 3~4초 정도 로딩시간이 생기는데 이를 자세히 보면

![profiling 결과 - 2](/assets/images/profiling-2.png)

addPlain이란 함수가 반복적으로 실행되는 것을 알 수 있다. (자막이 700개라면 이 함수가 700번 불리는 것이다)
이 함수는 자막 파일을 읽고 정규식으로 parsing 해서 자막을 Edit하는 Block(아래 캡처 참조)을 만들어 내는 함수이다.
![Edinting block](/assets/images/edit-block.png)

![profiling 결과 - 3](/assets/images/profiling-3.png)
더 안으로 파고 들어가면 initTimePicker라는 함수가 대부분의 시간을 잡아먹는 것을 알 수 있다.

자막 시간을 입력할 수 있는 input을 init해주는 함수인데 bootstrapTimePicker라는 라이브러리를 Custom해서 사용하고있다.

즉 느린 이유는 jQuery 때문이 아니라 라이브러리에서 timePicker를 init하는 시간이 너무 오래걸렸기 때문이었다.

애초에 느린 이유에 대한 가설이 틀렸기 때문에 목적은 달성하지 못했다.

문제를 정확히 파악하는게 선행되었어야 하는데 너무 해보고 싶었던 것이라 무작정 해버렸더니 실패했다..

(그래도 얼마정도 성능 향상은 있었다. [javascript selector vs jquery selector](https://jsperf.com/jquery-vs-javascript-performance-comparison/22))

# 2. jQuery를 완전히 걷어내는 것은 힘들었다.

목적은 jQuery를 완전히 걷어내는 것이였는데 아래 두가지 이유때문에 실패했다.

1. jQuery를 이용하는게 더 빠른 경우.
- 이런경우는 거의 없지만 html() vs innerHTML 이라면 jquery의 html() 메소드가 더 빠르다.
- 참고 [jQuery .append() vs jQuery .html() vs javascript innerHTML](https://jsperf.com/jquery-append-vs-html-list-performance/20)
2. 사용한 라이브러리의 디펜던시가 jQuery라서 어쩔 수 없는 경우.
- 애초에 javascript로만 처음부터 작성했었더라면 다른 라이브러리를 찾거나 직접 만들었겠지만. 이미 사용하고 있던것을 다른 것으로 교체하는 것은 거의 불가능해서 그만두었다.

# 3. 생각보다 jQuery를 없이 짜는 것이 어렵지 않았다.

## 아래에 몇가지 경우를 소개하겠다.

### Selector
```javascript
// jQuery
$('div.selector')
$('div#selector')
$('div.selector').find('.btn')

// javascript
document.getElementsByClassName('selector')
docuemnt.getElementById('selector')
document.querySelector('div.selector').querySelector('.btn')
```


### .empty()
```javascript
// jQuery
$('div').empty()

// javascript
document.querySelector('div').innerHTML = ''
```


### .remove()
```javascript
// jQuery
$('div').remove()

// javascript
const div = document.querySelector('div')
div.parentNode.removeChild(div)
```


### .addClass(), .removeClass()
```javascript
// jQuery
$('div').addClass('hidden')
$('div').removeClass('hidden')

// javascript
const div = document.querySelector('div')
div.classList.add('hidden')
div.classList.remove('hidden')
```


### .hasClass()
```javascript
// jQuery
$('div').hasClass('hidden')

// javascript
document.querySelector('div').classList.contains('hidden')
```


### .html()
```javascript
// jQuery
$('div').html(string)

// javascript
document.querySelector('div').innerHTML = string
```


### .text()
```javascript
// jQuery
$('div').text() // get
$('div').text(string) // set

// javascript
const div = document.querySelector('div')
div.textContent // get
div.textContent = string // set
```


### $.each
```javascript
// jQuery
$.each(array, (i, item) => {
  // do something
})

// javascript
array.forEach((item, i) => {
  // do something
})
```


### .append()
```javascript
// jQuery
$('div').append(element)

// javascript
document.querySelector('div').append(element)
```


### .clone()
```javascript
// jQuery
$('div').clone()

// javascript
document.querySelector('div').cloneNode(true)
```

내가 사용한 것은 이런 것들인데 더 참고할 것이 있다면 아래 두 page를 확인하자.

1. [http://youmightnotneedjquery.com/](http://youmightnotneedjquery.com/)
2. [http://codeblog.cz/vanilla](http://codeblog.cz/vanilla/)


### 4. 다른점

jQuery에서는 복수개의 DOM을 찾으면 코드 한줄로 모든 DOM에 관한 것을 함께 변경해주지만 javascript는 일일히 루프를 돌며 처리를 해줘야 한다.

```javascript
// jQuery
$('div').addClass('hidden')

// javascript
const divs = document.querySelectorAll('div')
divs.forEach((div, index) => {
  div.classList.add('hidden')
})
```

### 마무리

1. 문제해결 할때는 충동적으로 하지말고 문제 원인 파악을 먼저 제대로 하자.
2. 앞으로는 프로젝트 셋팅할 때 jQuery 부터 설치하는 습관을 버리고 javascript로 작성해보자.
