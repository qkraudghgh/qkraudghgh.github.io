---
layout: post
title: "Jekyll로 블로그 만들기"
date: 2016-07-02 00:00:00 +0900
categories: Jekyll
tags: blog
author: Myoungho Pak
---

마음만 먹고 있다가 저도 드디어 Jekyll을 이용해서 블로그를 만들었습니다.
Jekyll로 블로그를 만들고 싶은데 어떻게 해야할 지 막막한 분들이 보면 좋을 것 같네요.
각종 **Trouble Shooting**은 최하단에 정리해 두었으니 하다가 막히는 부분이 있으면 참고하셔도 좋습니다.

# 앞서
해당 글은 Jekyll 설치부터 포스팅 하는 법 그리고 [github](https://github.com)에 호스팅 하는 법 까지 나와 있습니다.
Ruby를 모르셔도 블로그를 만들고 호스팅하는데 문제 없지만 [Git](https://rogerdudler.github.io/git-guide/index.ko.html)을 모른다면 조금 힘들 수 도 있습니다.
또한 모든 내용은 **Mac**을 기준으로 작성하였습니다!

# Jekyll ?

[Jekyll 공식 한글 문서](https://jekyllrb-ko.github.io/)를 살펴보면 Jekyll은 Ruby의 Gem으로 제공되며
Markdown 이나 HTML등을 이용하여 정적인 웹페이지를 만들어 줍니다.

# Jekyll을 설치해봅시다.

버전에따라 다르지만 mac에는 Ruby가 기본적으로 설치되어있고
Ruby의 패키지 매니저인 Gem을 이용하여 Jekyll을 설치할 수 있습니다.

```bash
$ gem install jekyll
```

permission error는 `sudo`를 붙여 해결합시다.

# Jekyll Theme

이제 블로그를 만들 모든 준비가 되었습니다.
[Jekyll Theme](http://jekyllthemes.org/)에서 원하는 테마를 다운로드 합니다.

다운로드 한 후 해당 폴더에 들어가 Jekyll을 이용하여 로컬에 블로그를 띄워봅시다.

로컬에 블로그를 띄우기 전에 해야할 것이 하나 있습니다. 그것은 **jekyll**에서 사용하는 라이브러리들을 설치해야 합니다!
해당 Theme에서 사용하고있는 라이브러리 들은 `_config.yml`의 최상단에 적혀있습니다.

```markdown
# Build settings
gems:
  - jemoji
  - jekyll-paginate
  - jekyll-feed
  - jekyll-mentions
  - jekyll-redirect-from
```

저의 경우에는 이런식으로 5가지의 라이브러리가 필요합니다. `gem install jemoji`와 같은 명령어를 이용하여 모두 설치해줍시다!

해당 라이브러리들의 설치가 끝난 후엔 아래의 명령어로 로컬에 띄워보세요.
만약 `nokogiri`관련 에러가 난다면 하단에 있는 **TroubleShooting**을 참고하세요!

```bash
$ Jekyll serve
```

local에서는 [http://127.0.0.1:4000](http://127.0.0.1:4000)에서 확인하실 수 있습니다.

# _config.yml

이제 여러분은 **_config.yml**을 수정하여 나만의 블로그를 만들 수 있습니다.

_config.yml이 밑에 있는 거와 동일 할 수는 없지만 참고용으로 적어두었습니다.

```markdown
# Site settings
title: "Myoungho Pak"
description: "쓰고싶은거 쓰는 블로그"
email: 'qkraudghgh@gmail.com'
#blog logo
logo: "https://qkraudghgh.github.io/assets/images/profile.JPG"
# blog cover
cover: "https://qkraudghgh.github.io/assets/images/main.JPG"

authors:
   Myoungho Pak:
     name: Myoungho Pak
     image: "https://qkraudghgh.github.io/assets/images/profile.JPG"
```

해당 블로그에서 이용하는 `Author`에 대한 정보는 모두 이 곳에서 수정 가능합니다.

저는 image를 모두 assets/images/ 폴더를 만들어 관리하고 있습니다. 

위 스크립트를 보면 모두 url주소가 포함된 image 주소가 들어가 있지만
`/assets/images/main.JPG` 같은 상대경로도 사용 가능합니다.

페이스북이나 트위터, 슬랙 등에서 meta tag가 잘 작동하게하려면 상대경로를 사용하시는 것이 좋습니다.

# Post 하기

Theme에 따라 다르겠지만 기본적으로 모든 `Posts`들은 _posts 폴더에 파일을 만드는 것으로 포스팅 할 수 있습니다.

파일 이름은 특별한 규칙에 의해서만 생성되어야 합니다.

```markdown
year-month-day-postsName.md
```
지금 포스트는 `2016-07-02-jekyll-blog-setup.md`의 형태를 띄고 있습니다.

`year-month-day-postsName.md` 상단엔 아래와 같은 내용의 스크립트가 들어가야합니다.

```markdown
---
layout: post
title: "Jekyll로 블로그 만들기"
date: 2016-07-02 00:00:00 +0900
categories: Jekyll
tags: blog
author: Myoungho Pak
---
```

그리고 두어줄 띄어준 후에 Markdown으로 Posting 해주면 됩니다!

Markdown에대한 정보는 [이 곳](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)에서 확인하실 수 있습니다.

# GitHub에 호스팅 하기

GitHub에 호스팅 하는 것도 무척 간단합니다.
Github는 [github Page](https://pages.github.com/)라는 것을 이용해 Repository를 웹 페이지로 만들어주는 서비스를 제공하고 있습니다.
우리는 이걸 이용해서 블로그를 호스팅 할 것이고 그러기 위해선 Repository를 하나 생성해야 합니다. Public이던 Private이던 상관 없습니다.

github에 `username.github.io`의 Repo를 하나 만듭니다.

![github profile 참고](https://qkraudghgh.github.io/assets/images/username.png)

`username`은 위 사진에서 **qkraudghgh**에 해당하는 부분입니다.

제 경우엔 `qkraudghgh.github.io`이고 이는 repo의 이름이 되는 동시에 블로그의 url이 될 것입니다.

그 다음엔 해당 repo와 local 을 remote 해주어야 합니다.

```bash
$ git init
$ git remote add origin your_git_repo_URL
$ git push origin master
```

해당 명령어까지 마치고 나면 이제 Jekyll을 이용한 블로그 만들기부터 호스팅이 끝이 납니다.

`userName.github.io`를 들어가 자신이 만든 blog를 확인해 봅시다.

# Trouble Shooting

- `gem install jemoji`시에 error가 나는 경우

```bash
sudo gem install nokogiri -- \
    --use-system-libraries \
    --with-iconv-dir=/path/to/dir \
    --with-zlib-dir=/path/to/dir \
    --with-xml2-config=/path/to/xml2-config \
    --with-xslt-config=/path/to/xslt-config
```

해당 에러는 El Captian에서 생길 수 있는 문제라고 합니다. `nokogiri`가 제대로 설치되지 않는 문제인데
위의 스크립트를 이용해 설치하면 해결됩니다.

- `Jekyll serve`를 자주 사용합시다.

이는 `origin`에 push하기 전 블로그에 게시될 모습을 미리 살펴볼 수 있고 수정할 수 있도록 해줍니다.

또한 `watch`기능이 제공되어 파일이 수정될 때 마다 번거롭게 서버를 껐다 켰다할 필요없이 새로고침 만으로
변경사항을 적용해 줍니다.

자주 이용하세요!

# 마무리

글이라는 건 참 어려운 것 같습니다. 남에게 도움이 되는 글을 쓰기란 더욱 어렵네요. 앞으로도 시간날때마다 포스팅을 해볼 생각입니다.

이상한 점이 있거나 잘못 된 부분이 있다면 메일이나 [9xd](https://9xd.slack.com)에서 DM주시면 시간날 때 수정하겠습니다.

감사합니다.
