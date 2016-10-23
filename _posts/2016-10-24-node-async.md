---
layout: post
title: "NodeJS와 비동기"
date: 2016-10-23 00:00:00 +0900
categories: node 
tags: async
author: Myoungho Pak
---

NodeJS를 처음 접하는 사람에겐 비동기적이라는 말이 이해하기가 어렵다. 그렇다면 과연 비동기적인 것이 무엇일까?

# 머릿말

NodeJs를 처음 접하는 사람들이라면 누구나 듣지만 정작 이해하기 힘든 것이 하나 있다.
바로 `비동기적이다`라는 말인데 처음 접하는 사람들은 이게 여간 정확하게 알기 힘들다.

그래서 이번 글에선 NodeJs의`비동기`라는 키워드를 가지고 내가 이해하지 못했던 것들과 왜 이해하지 못했었는지 의식의 흐름에 따라 글을 적어보려한다.

# 예제 코드부터 들여다보자

```javascript
import fs from "fs";

console.log("I'm First!");

// practice.txt에는 "I'm Second!"라는 문장이 적혀있다.
fs.readFile("./practice.txt", (err, res) => { 
  console.log(res);
});

console.log("I'm Third!");
```

이 예제는 NodeJs를 처음 접하면 당연히 한번쯤 보게되어있는 예제인데 console.log로 출력되는 문장을 차례대로 적어보면

```javascript
I'm First!

I'm Third!

I'm Second!
```

다음과 같은 순서로 적히게 된다.

많은 동기적 프로그래밍을 해왔던 사람들(나를 포함해서)은 이 코드를 보고서 당연히 

```javascript
I'm First!

I'm Second!

I'm Third!
```

와 같은 결과를 원했겠지만 실제론 그렇지 않았다.

당시에는 그냥 막연하게 아 NodeJs는 `비동기`적이어서 그렇구나~ 하고 넘어갔었는데 그건 당연히 아니였다.

# Code Smell

막연하게 비동기라는 키워드를 가지고 정확한 이해없이 프로그램을 짜다보면 직면하는 두려움이 있다.

그 두려움은 바로 코드의 실행 순서를 명확하게 알지 못한다는 것인데, 이런식으로 코드를 짜다보면 어찌어찌 잘 돌아가는 코드를 짠다 해도
내가 짠 코드가 무조건 잘 작동 될 것이라는 확신이 없어진다.

나의 이러한 두려움을 조금이라도 없애기 위해 내가 했던 아주 `무식한` 방법이 있는데 그건 바로 Promise의 `.then`을 남발하는 것이었다.

이 당시의 나는 `.then`은 어떤 코드실행이 끝난 후 아래의 코드를 실행해라 정도의 의미로 받아드리고 있었다.

물론 틀린 말은 아니지만 나의 이해수준의 정도에선 틀린 말이었다.

그래서 코드의 실행 순서를 명확하게 알지 못하니까 `이 부분은 무조건 순서를 보장해줘야해!` 하는 부분에 모조건`.then`을 남발했다.

```javascript
myApiCallFunction() // res로 [1,2,3,4,5] 를 주는 Api를 호출하는 function이다. 
  .then(res => {
    let temp = res;
    temp.forEach(num => {
      num += 1;
    })
    return temp; 
  })
  .then(res => {
    let temp2 = res;
    temp2.forEach(num => {
      num += "입니다.";
    })
    return temp2; // ['2입니다.','3입니다.','4입니다.','5입니다.','6입니다.']
  })
  .catch(err => {
    alert(err); 
  });
```

과장을 조금 보태자면 이러한 코드를 작성하고 있었다. 두번째 forEach문 보다 첫번째 forEach문을 무조건 먼저 실행시켜주고 싶었기 때문이었다.

# 그래서 비동기가 뭔대?

NodeJS는 multi Thread 방식의 문제점을 보완하기위해 Single Thread + Non-blocking I/O 방식을 도입한 FrameWork이다.

그리고 우리는 이것을 추상적으로 흔히 NodeJS는 비동기적으로 돌아간다고 얘기한다.

# CPU Bound, I/O Bound

일단 Node Js의 비동기를 설명하기 전에 알아야할 개념들이 있다. 이는 CPU bound 와 I/O Bound이며 설명은 아래와 같다.

- CPU Bound
  - CPU 자원을 사용하는 Task
  - Javascript V8 Engine에서 처리된다.
  - 작업 속도가 빠르다
  - I/O Bound를 제외한 대부분의 javascript 코드와 연산들이 이에 해당한다.
- I/O Bound
  - Input/Output 즉 Disk, Network, Database와 관련된 Task
  - Event Queue에 Message 형식으로 쌓이며 Event Loop가 돌면서 Event Queue에 쌓인 Task들을 처리한다.
  - 작업이 대부분 네트워크등 통신을 해야하기 떄문에 다소 느리다.


# Event Loop

![event Loop](/assets/images/event-loop.jpg)

`Event Loop`는 NodeJS의 싱글 쓰레드에서 돌아가며 I/O Bound 작업들을 비동기적으로 처리해주기 위해서 필요하다.

Client에서 I/O Bound 요청이 온다면 이 요청들은 Message 형태로 Event Queue에 저장된다.

`Event Loop`는 Event Queue에 있는 Task들을 Pop하여 Non-Blocking 방식으로 Kernel에 처리를 요청하며 작업이 끝난 Task들을 감지하고 `Callback function`을 호출한다.

내부적으로 Non-Blocking function을 지원하지 않는 I/O task들을  Multi Thread Pool로 처리해주기도 한다.
 
# Non-blocking I/O Model
 
Event Loop에서 I/O bound Task들을 kenel에 요청하게 되는데 이때 Non-blocking I/O Model이라는 방식을 사용하게 된다.

![Non-blocking I/O Model](/assets/images/non-blocking.jpg)

여기서 중요한 것은 application level의 recvfrom(read)를 통해서 kenel 단위로 system call을 하게된다.
반환 받을 것이 있다면 data를 받고 반환 받을 것이 없다면 EWOULDBLOCK이라는 상태를 반환 받는다는 것이 중요하다. 
EWOULDBLOCK이라는 상태를 반환 받게되면 `진행중` 이라는 의미이며 Blocking 방식과 다르게 Data를 받을 때까지 Thread가 멈춰서 대기하지 않아도 되고 
다음 Process의 작업을 할 수 있게 된다.

하지만 EWOULDBLOCK이라는 상태를 반환 받은 I/O task들을 갱신해주기 위해서 recvfrom을 계속 loop 돌며 data를 받을 때 까지 감시할 필요가 있는데,
NodeJS에서는 이러한 것들을 처리해주기 위해 내부적으로 [libuv](https://github.com/libuv/libuv)라는 것을 사용하고 있다.

이 libuv는 비동기적인 I/O Task 처리를 위한 모든 것들을 제공해준다.

# Single Thread인 NodeJS가 blocking I/O를 사용했다면?

만약 싱글 쓰레드에서 blocking I/O를 사용했다면 상대적으로 처리 시간이 긴 I/O Bound작업을 처리할 때 I/O Task가 끝날때 까지 Thread는 다음 작업을 진행하지 못하고
대기하게 되는데 그렇다면 CPU자원은 사용하지 않고 놀고있는 상태가 될 것이다.

이러한 문제점을 해결하기위해 NodeJS는 Non-Blocking 방식을 이용하므로써 놀고있는 CPU자원까지 끌어다 쓸 수 있게되었다. 그래서 흔히들 Node JS는 빠르다라고 얘기 할 수 있게 되었다.
 
# Multi Thread Pool
기본적으로 NodeJS는 `Single Thread`를 사용한다고 알려져있지만 내부적으로는 추가로 Multi Thread Pool을 이용한다.

이는 일부 Non-blocking을 자체 지원하지 않는 I/O에 대해서 내부적으로 Non-blocking 방식으로 I/O를 처리하기 위해서 이용하는 Thread이다.
Event Loop은 CallBack function을 붙여 thread들에게 요청을 할당하게된다.

실제로 요청을 처리하는 Thread는 `Event Loop` 단일 쓰레드가 맞다.
 
# 내가 했던 실수를 고쳐보자
 
즉 내가 앞에서 Promise의 `.then`으로 forEach문 2개를 나누어 준것은 의미가 없는 짓이었다.
왜냐하면 비동기적으로 처리될리 없는 V8을 이용한 CPU Bound Task들이 비동기적으로 호출 될 것을 우려하여 
`.then`으로 논리적일 뿐인 동기적 코드를 작성했기 때문이다.

위에서 실수라 했던 코드를 리펙토링 한다면 아래와 같이 쓸 수 있겠다.

```javascript
myApiCallFunction() // res로 [1,2,3,4,5] 를 주는 Api를 호출하는 function이다. 
  .then(res => {
    let temp = res;
    temp.forEach(num => {
      num += 1;
      num += "입니다.";
    }) 
    return temp; 
  })
  .catch(err => {
    alert(err); 
  });
```

# CallBack Hell

이러한 비동기적인 코드를 작성하다보면 우리는 언제 I/O Bound Task들이 끝나고 Callback이 호출됐는지 알 수 없다. 
그래서 우리는 동기적인 코드가 필요할 때 task가 끝나면 반드시 호출되는 callback들을 이용해야한다.

즉 여러개의 I/O Bound Task를 동기적으로 처리하고 싶다면 Callback function 내에서 다른 I/O Bound Task를 호출해야하고
이러한 동기적 호출이 몇개나 더 필요하다면 Callback안에 Callback 그리고 더많은 Callback들을 이용해야한다. 

우리는 이것을 `CallBack Hell`이라고 부른다.

# TL:DR

결론적으로 NodeJS는 코드를 순차적으로 읽어가며 `CPU Bound task`를 만나면 V8 엔진을 통해 바로 값을 계산하고,
`I/O Bound task`를 만나면 Event Queue에 등록하며 Event Loop에 의해 요청이 처리된다.

이때 `I/O Bound task`의 요청은 언제 끝날지 알 수 없으며 다만 끝나면 무조건 호출되는 `CallBack Function`이 있다.

Node Js는 `비동기적이다`라는 말은 I/O Bound Task의 요청을 던져두고 다른 작업 혹은 요청을 하다가.
요청이 처리되면 `나도 모르는 시점`에 결과가 callback으로 호출되는 점을 두고 말하는 것이라고 할 수 있겠다.
 
# 마치며
 
나는 이 글을 쓰면서 나와 같은 실수를 했던 사람이 좀더 정확하고 자세하게 Node JS의 비동기에대해 알아가길 원했다. 다만 글을 쓰면서 내가 생각했던 것 보다 내가 모르고 있는게 많았고 그것을 깊게 공부하면서 초보에게 이해하기 쉽게 얘기하고 싶다는 목표를 달성하지 못한게 아닌가 싶기도 하다. 

다음 글에서는 이 비동기적인 코드들을 동기적으로 작성할 수 있게 도와주는 Library들을 소개하고싶다.
현재 내가 주로 쓰고있는 Promise부터 generator pattern 그리고 ES7에 들어갈 async/await 까지 얘기해보려한다.

내 글이 읽고 있는 당신에게 도움이 되었으면 좋겠다.
