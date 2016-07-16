---
layout: post
title: "MySQL ERROR 2002 문제"
date: 2016-07-16 00:00:00 +0900
categories: mysql 
tags: DB
author: Myoungho Pak
---

요즘 MySQL을 공부하면서 오랜만에 `terminal`에서 `mysql -uroot`를 실행하였더니
아래와 같은 Error가 발생했습니다. 이를 해결한 일련의 과정을 서술합니다.

```Shell
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/usr/local/mysql/mysql.socket' (2)
```

# 에러를 해결하기 까지..

위의 에러를 보아하니 `mysql server`가 구동이 안되어 있는 것이 문제 같았습니다.

때문에 아래의 명렁어를 실행시켜 server 구동을 해보았습니다.

```Shell
$ mysql.server start

Starting MySQL
......................................
```

계속해서 점만 늘어나고 Success가 뜨지 않았습니다. 혹시나 MySQL 데몬의 문제일까 하고 아래의 명령어를 실행했습니다.

```Shell
$ mysqld
```

그랬더니 아래의 Error가 나왔습니다.

```Shell
[ERROR] InnoDB: Unable to lock ./ibdata1 error: 35
[Note] InnoDB: Check that you do not already have another mysqld process using the same InnoDB data or log files.
```

해석을 해보니 MySQL 데몬이 이미 구동되어있어서 새로 데몬을 구동할 수 없는 문제였고 저는 MySQL 데몬을 종료시키기로 했습니다.

일단 프로세스상 구동되어있는 MySQL 데몬의 PID를 알아내기 위하여 `lsof`란 명령어를 이용하였습니다.

```Shell
$ lsof -i TCP:3306
```

mysql은 포트번호 3306을 이용하므로 위의 명령어를 실행시켰습니다. `lsof`에대한 자세한 내용은 [링크](https://ko.wikipedia.org/wiki/Lsof)를 참조하시기 바랍니다.

해당명령어를 이용하니 mysqld가 이미 구동되어있는 것을 확인 할 수 있었고 이녀석의 PID또한 알아낼 수 있었습니다. 아래와 같은 명령어로 프로세스를 종료시켜줍니다.

```Shell
$ kill your_pid
```

그리고나서 다시 MySQL을 구동시켜봅시다!

```Shell
$ mysql.server start

Starting MySQL
. SUCCESS!
```

성공입니다.

# 마치며

가끔 잘 돌아가던 것들이 나중에 다시 이용하려면 에러가 나는 경험을 간혹 할 때가 있습니다.
그래도 얼마 시간들이지 않아서 다행이네요!
