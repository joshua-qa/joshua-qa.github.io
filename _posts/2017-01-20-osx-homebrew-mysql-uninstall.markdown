---
layout: post
title: Homebrew로 설치한 mysql을 삭제하기
date: 2017-01-20 20:48:10
tags:
- osx
- homebrew
- mysql
categories:
- Dev
---

맥에서도 mysql을 쓰려고 homebrew를 이용해서 설치해놨는데, 그 때 어떻게 설정했는지 몰라도 root 비밀번호가 안맞는 문제가 발생했다. (설치한 기억만 있고 비밀번호 설정한 기억은 없는데 말이다)

별도의 방법으로 접근해서 root 비밀번호 변경하는 방법도 찾아내서 해봤는데 잘 안되고.. 지우고 다시 설치해서 해봐도 뭐가 이상하게 안되고 ㅠㅠ 결국은 brew uninstall mysql로 검색해서 나온 완전 삭제 방법을 이용했다.

[링크](https://coderwall.com/p/os6woq/uninstall-all-those-broken-versions-of-mysql-and-re-install-it-with-brew-on-mac-mavericks)

1) 위 링크에 나온대로 찌꺼기들을 삭제 처리해주고 재부팅

2) brew install mysql

3) mysql.server start

4) mysql_secure installation

해주니 정상적으로 세팅하는 화면으로 넘어갔다.

구글링해서 나온 페이지들을 보고 따라해도 세팅이 잘 안될 경우는 완전히 삭제해주고 다시 설치해서 하는 것이 제일 나은 것 같다.
