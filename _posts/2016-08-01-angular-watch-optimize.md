---
layout: post
title: "Angular 양방향 데이터 바인딩과 최적화"
date: 2016-08-01 19:00:00 +0900
categories: angular
tags: angular
author: Myoungho Pak
---

이번 포스팅에서는 angular의 양방향 데이터 바인딩과 성능 최적화 방법에 대해 적어보려합니다.

# 머릿말
최근 [프립](https://www.frip.co.kr) 사이트가 느려지고 멈추는 현상이 발생했습니다.
프리징과 느려짐은 웹을 이용하지 못할 정도로 유저들을 괴롭히고 있었죠.

결론적으로 문제는 Angular를 제대로 알지 못하고 써온 코드들이 복합적인 원인이 되어 `Memory Leak`을 내고 있었습니다.

이번 포스팅에서는 이러한 `Memory Leak`을 해결하기위해 어떤 최적화를 진행하였는지 얘기하겠습니다.


# 앞서

이 글에서는 아래의 3가지를 다룰 생각입니다.

1. angular에서 말하는 `양방향 데이터 바인딩`이 무엇이고 어떻게 동작하는지
2. 이 양방향 데이터 바인딩이 가져다준 단점
3. 성능을 개선할 수 있는 몇가지 방안

# Angular context 와 양방향 데이터 바인딩

angular는 기본적으로 `angular context`를 통해 Event를 처리하고 DOM을 업데이트합니다.

javascript context에서 `angular context`에 진입하기 위해선 `$apply()`라는 특정 메소드가 필요합니다.

![angular context 구조](/assets/images/angular-context.png)

`$apply()`를 통해서 angular context에 진입하고나면 `$evalAsync queue`와 `$watch list`로 이루어진 `$digest loop`를 돌게됩니다.
`digest loop`는 무한 loop를 방지하기위해 10회를 돌고나면 더이상 loop하지 않고 angular context를 빠져나오게 됩니다.

- [`$evalAsync queue`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$evalAsync)는 새로운 $digest loop를 생성하지 않기위해 필요한 스케쥴 job입니다. 특정한 expression이 발생했을 때 이미 기존 $digest loop가 돌고 있다면 해당 작업은 새로운 $digest loop를 생성하는 것이 아니라 `$evalAsync queue`에 스케쥴로 추가되어 있다가 현재 $digest cycle이 다음 loop를 돌때 $watch list에 추가되어 $watch function에 의해 처리됩니다. 

- `$watch list`는 $watch fucntion에 의해 감지되어야 할 것들의 집합입니다. $watch list에 등록되는 경우는 아래 6가지 경우가 해당됩니다.
  - `$scope.$watch`
  - `{{ "{{  " }}}}` type bindings
  - 대부분의 directives (i.e. `ng-show`)
  - Scope variables scope: `{bar: '='}`
  - Filters `{{ "{{ value | myFilter " }}}}`
  - `ng-repeat`


`$watch function`은 $watch list에 등록되어있는 model들을 하나하나 `dirty checking`(newValue와 oldValue를 비교하여 바뀌면 newValue로 update)하게 됩니다.

바로 이 `$watch function`에 의해서 양방향 데이터 바인딩이 일어나게 됩니다.

# 양방향 데이터 바인딩이 가져다준 단점

우리는 이러한 앵귤러의 life cycle에 의해서 양방향 데이터 바인딩이 어떻게 작동하는지 알아보았습니다.

model에 값을 update해주고 자동으로 view render까지 해주는 이 양방향 데이터 바인딩에도 단점은 존재합니다.

문제는 바로 $digest loop에 있습니다. 우리가 뷰를 구성할 때 수 많은 양방향 데이터 바인딩을 이용한다면 $watch list에 있는 수많은 model들을 하나하나 dirtyChecking해야 할 것이고 이것은 곳 앵귤러의 성능 저하에 아주 직접적인 영향을 끼칩니다.

그래서 우리는 무분별한 양방향 데이터 바인딩을 지양하고 꼭 필요한 부분에만 양방향 데이터 바인딩을 이용해야 합니다.

# 성능을 향상 시키기 위한 최적화 방안

- `{{ "{{  " }}}}` type bindings 에서 `::`를 같이 사용합니다.
  - `{{ "{{ value " }}}}` =>  `{{ "{{:: value  " }}}}` 
  - 양방향 데이터 바인딩은 우리에가 많은 이점을 가져다주지만 제대로 사용하지 않는다면 추후에 성능문제로 우리들을 괴롭힐 것입니다.
그래서 양방향 데이터 바인딩은 꼭 필요한 경우에만 사용해야 합니다.
- ng-repeat을 사용할 땐 `track by`를 함께 사용합니다. 
  - track by 는 ng-repeat이 DOM을 다시 그릴지 알려주는 속성입니다.
  - `<div ng-repeat="customer in custmoers track by customer.id">` 와 같이 써주면 customer.id가 변하지 않는 이상 $digest loop를 돌아도 DOM을 새로 그리지 않습니다.
- ng-show와 ng-hide 대신 `ng-if`를 사용합니다.
  - ng-show와 ng-hide는 expression을 통해 DOM을 감추거나 보여줍니다. 이는 `display: none;` 을 이용하여 동작하므로 DOM을 그립니다.
  - ng-if는 expression을 통해 DOM을 감추거나 보여주지만 만약 expression에 맞지 않는 다면 DOM을 아예 그리지 않습니다.
- $apply 대신 `$digest`를 대신 사용합니다.
  - $apply는 $rootScope.$digest()를 뜻합니다.
  - Global하게 $digest loop가 필요한 것이 아니라면 $digest()를 호출하는게 좋습니다.
  - $digest()는 해당 scope와 자식 scope들만으로 digest cycle을 만듭니다.
- $httpProvider.useApplyAsync를 사용합니다.
  - 해당 메소드는 multiple http responses가 생겼을때 이를 하나로 합쳐줍니다.
  - http response는 그 response가 각각 $apply()를 호출하므로 이를 하나로 합쳐준다는 것은 상당한 성능 향상이 있습니다. 
  - controller단에서 아래와 같이 사용하실 수 있습니다.

```coffeescript
app.config ($httpProvider) ->
  $httpProvider.useApplyAsync true
  return
```

# 성능을 최적화 시키고 난 후..

![최적화 전 TimeLine](/assets/images/optimize-pre.png)

- 최적화 전 js Heap이 GC에 의해 제대로 줄어들지 않고 서서히 늘어나는 모습을 볼 수 있었습니다.

![최적화 후 TimeLine](/assets/images/optimize-after.png)

- 최적화 후 js Heap이 GC에 의해 제대로 정리되고 있는 모습을 볼 수 있었습니다.

# 마치며

매번 아무생각 없이 앵귤러를 사용해오다가 문제에 직면하고 나서야 앵귤러를 제대로 공부하게된 것 같습니다. 아직 앵귤러의 일부분밖에 이해하지 못했지만 적어도 앞으로 앵귤러를 사용할 때 이전보다 더 능숙하게 사용할 수 있을 것 같은 느낌이 듭니다!
