---
layout: post
title: "Hubot 슬랙과 연동하기"
date: 2016-07-16 19:00:00 +0900
categories: hubot
tags: slack
author: Myoungho Pak
---

이번 포스팅에서는 Hubot을 설치하고 heroku에 배포하여 Slack에 연동시키는 과정을 적어보려합니다.

# 머릿말

처음 [Frientrip](https://www.frip.co.kr)에 들어갔을때 저는 말로만 듣던 slack을 처음 써봤습니다. 그리고 Frientrip에서는[Hubot](https://hubot.github.com/)과 Slack을 연동하여 다양한 용도로 사용하고 있었습니다. 점심먹고 난 후 간단한 내기부터 웹훅을 이용한 Github과 Trello 연동까지 프로그래밍 세계에 발을 들인지 얼마 안된 저는 신세계를 경험했죠. 입사후 몇일 동안 소스코드를 보면서 Hubot 스크립트도 덩달아 훑어봤었는데 그 것이 제가 9xd에서 Hubot을 운영하게된 계기가 된 것 같습니다.

얼마전에 MS 최고 경영자인 [사티아 나델라](https://ko.wikipedia.org/wiki/%EC%82%AC%ED%8B%B0%EC%95%84_%EB%82%98%EB%8D%B8%EB%9D%BC)는 아래와 같은 말을 하기도 했습니다.

> 앱(app)의 시대가 가고 인간과 대화하는 인공지능(AI) 봇(bot)의 시대가 왔다.

물론 오늘 얘기하려하는 Hubot은 AI까지는 미치지 못하겠지만 어느정도 bot이 이런 것이구나 하는 정도는 경험해 볼 수 있지 않을까 싶습니다.

# 앞서

- 이 글은 Mac OSX 10.11(El capitan)을 기준으로 작성 되었습니다. 
- 다룹니다!
  - Node를 설치하고 Hubot 설치하기
  - Heroku에 Heroku Toolbelt를 이용하여 Hubot 배포하기
  - Heroku에 배포한 Hubot을 Slack에 연동하기 
- 안다룹니다!
  - Heroku 가입하기
  - Git 사용법
  - Hubot 기능 추가하기
  - Heroku Dyno가 잠들지 않게하기

# Hubot 이란?

**Hubot is your company's robot.** 공식 홈페이지에 나와있는 소개입니다. 실제로 머릿말에서 말했듯이 Frientrip에서도 다양한 기능으로 업무에 이용하고 있습니다.
IRC나 Slack등 채팅 App들에 연동이 가능하며 간단한 명령어를 통해 미리 짜둔 Script를 실행시켜 원하는 결과 값을 얻어낼 수 있습니다.

# Node 설치하기

이 [링크](https://nodejs.org/ko/)에 접속하여 메인에서 LTS 버전을 다운 받습니다. 휴봇은 Node.js의 package manager인 NPM을 통해 설치하실 수 있습니다.
설치가 완료되었다면 터미널에 아래 명령어를 입력하여 제대로 설치되어있는지 확인합니다.

```Shell
$ node -v
v4.2.3
$ npm -v
v2.14.7
```

위 두 명령어를 통해 버전을 확인 할 수 있으면 node 설치가 완료된 것입니다.

# Hubot 설치하기

npm 으로 hubot generator를 설치합니다.

```Shell
$ npm install -g yo generator-hubot
```

설치가 완료되면 hubot을 init해줄 폴더를 만들고 해당 Dir로 향합니다.

```Shell
$ mkdir myHobot && cd myHubot
```

디렉토리에 들어갔다면 아래 명령어를 통해 hubot을 만들어줍니다.

```Shell
$ yo Hubot
                     _____________________________
                    /                             \
   //\              |      Extracting input for    |
  ////\    _____    |   self-replication process   |
 //////\  /_____\   \                             /
 ======= |[^_/\_]|   /----------------------------
  |   | _|___@@__|__
  +===+/  ///     \_\
   | |_\ /// HUBOT/\\
   |___/\//      /  \\
         \      /   +---+  
          \____/    |   |
           | //|    +===+
            \//      |xx|

? Owner 박명호 <qkraudghgh@naver.com>
? Bot name myhubot
? Description A simple helpful robot for your Company
? Bot adapter slack
```

Owner와 Bot name 등은 아무렇게나 해도 되지만 Bot adapter는 꼭 **slack**으로 해주시기 바랍니다. 

명령어를 모두 입력하고 나면 로컬에서 테스트를 해보실 수 있습니다.

# 로컬에서 테스트해보기

```Shell
$ bin/hubot
myhubot> myhubot ping
myhubot> pong
```

위 처럼 bot의 이름 뒤에 ping을 붙여 엔터를 치면 봇이 `pong`하고 반응하는 것을 볼 수 있습니다.

# Slack에서 토큰 발급받기

웹에서 슬랙에 로그인을 해주신 다음 hubot을 만들고 싶은 Team의 app으로 갑니다. 로그인 후 [https://your-app.slack.com/apps](https://your-app.slack.com/apps)
이 링크로 들어가셔도 좋습니다. 그리고 화면 중간에 보이는 검색창에 `hubot` 을 검색하여 hubot을 slack에 추가해 준다음 **Add Configuration**을 통해
봇을 만들어 줍시다. Name은 원하는 것으로 등록해 주세요 이 곳에서 Hubot의 프로필 사진이나, 설명등을 바꿀 수 있습니다. 일단 여기서 제일 중요한
Api Token을 기억해 주세요.

```Shell
xoxb-4*********4-Q4*****Z0**FgP*****jT**w
```

위와 같은 형식의 토큰을 발견하실 수 있습니다. 이는 Heroku와 slack을 연동 시킬 때 필요한 Token입니다.

# heroku toolbelt 사용하여 dyno 만들기

heroku에 이미 가입하신 것을 전제로 하겠습니다.

[Heroku Toolbelt](https://toolbelt.heroku.com/)에서 OSX에 맞는 toolbelt를 설치해주세요 설치가 완료됐다면 아래와 같은 명령어를 실행해주세요.

```Shell
$ heroku login
      # 로그인 합니다.
$ heroku create
      # heroku dyno를 만듭니다.
      # 이때 만들어진 http://*****.herokuapp.com을 기억해두세요
```

# Heroku와 slack 연동하기

계속해서 heroku toolbelt를 이용하여 설정해줍니다.

```Shell
$ heroku config:add HUBOT_URL=http://*****.herokuapp.com
$ heroku config:add HUBOT_SLACK_TOKEN=xoxb-4*********4-Q4*****Z0**FgP*****jT**w
$ git init
$ git add .
$ git commit -m "my first hubot commit"
$ git push heroku master
```

배포가 끝나면 slack에 접속하여 hubot에게 ping을 날려봅시다.

# TroubleShooting

- heroku toolbelt에서 해당 dyno에 연결되지 않은 경우

```Shell
$ heroku git:remote -a your-dyno-name
```

해당 명령어를 통해 git과 heroku dyno를 remote 시켜줍니다.

- slack bot에 녹색 불이 안들어오는 경우
  - Token을 제대로 설정했는지 확인해 보세요.
    - [heroku](https://heroku.com)에 로그인하면 app setting을 다시 하실 수 있습니다.
  - 배포가 제대로 안 된경우
    - Heroku와 slack연동하기의 일련의 과정들을 다시 시도해 보세요

# 마치며

휴봇에 추가 기능을 넣는 것은 다루지 않았지만 CoffeeScript로 script를 작성하여 해당폴더 `./script`에 넣어주시고 다시 배포해주시면 기능이 추가됩니다.
이미 많은 기능들이 hubot 내부에 있으며 disabled 되어 있는 기능들은 `./node_modules/hubot-scripts/src/scripts`에서 확인하신 후 `external-scripts.json`에 추가해 주시면 따로 무언가를 설치할 필요없이 기능을 추가하실 수 있습니다.

또한 [9xd-hubot](https://github.com/qkraudghgh/9xd-bot)에 접속하셔서 여러가지 script를 확인해보셔도 좋습니다.

감사합니다.

