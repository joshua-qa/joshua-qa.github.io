---
layout: post
title: "[Android Studio] git commit한 것 취소하기"
date: 2017-02-01 18:56:02
tags:
- android
- android studio
- git
- commit
- RESET HEAD
categories:
- Dev
---

## 발생한 상황

* Android Studio에서 작업한 것들을 git으로 커밋 했는데, 커밋 대상이 아닌 파일을 포함하는 바람에 커밋 취소를 하고 싶었다.



## 해결 방법

`VCS 메뉴 -> Git -> Reset HEAD...`  선택 후 노출되는 창에서 To Commit에 `HEAD^` 를 써주고 리셋하면 된다.

기본적으로 HEAD가 입력되어 있는데 이 상태로 리셋을 하면 안되고 반드시 ^을 붙여줘야 한다.

나같은 경우는 커밋 2개를 취소해야됐는데, Reset HEAD^ 두번 해주고 다시 커밋했다.



## 느낀 점

* git 사용법 어느정도는 알았다고 생각했는데, 좀 더 잘 공부해야될 것 같다.
* 그냥 SourceTree로 관리할래..
