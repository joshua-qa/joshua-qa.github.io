---
layout: post
title: javascript-debugger
date: 2017-04-11 13:51:25
categories:
- Dev
tags:
- javascript
---

## 발생한 일
별도로 작성한 스크립트를 iOS 사파리나 크롬에서 북마크에 넣어놓고 실행할 일이 있는데, 어느날 이 스크립트가 정상적으로 동작하지 않는 이슈를 발견했다.
원래는 잘 됐던 스크립트였으나.. 최근 들어서 실행하는 순간 브라우저를 멈추게 하는데 디버깅을 해서 원인을 찾아내야하는 상황이었다. 모바일 환경에서의 웹 디버깅에 대한 경험도 거의 없었고, 스크립트 실행한 직후에 멈추는 현상이어서 어떤식으로 접근해야할지 잘 몰랐다.

## 해결 방법
Debugger 함수를 사용해서 중단점을 설정하는 방법이 있었다.
그동안 디버깅이라고 하면 브레이크 포인트를 잡고 돌려보거나, console.log등을 사용한 방식밖에 몰랐는데 이 방법을 사용하면 내가 실행하려는 스크립트에 대해 중단점을 지정할 수 있는 것이었다.

```javascript
var test = 1234;
debugger;
console.log(test);
```
이렇게 쓰고 스크립트를 실행하면, 아래와 같이 중단점이 동작하게 된다.
(스크립트 실행할 때 개발자 도구가 열려있어야 한다)

![debugger](/images/debugger_1.png)

예제 실행은 w3schools 페이지를 참고해도 된다.
[Example](https://www.w3schools.com/js/tryit.asp?filename=tryjs_debugger)


## 느낀 점
* 디버깅 기법에 대해서는 제대로 기억하자.
