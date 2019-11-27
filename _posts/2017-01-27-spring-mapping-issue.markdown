---
layout: post
title: "[Spring] RequestMapping이 잘 안된 이야기"
date: 2017-01-27 02:59:08
tags:
- spring
- requestmapping
- tomcat
- issue
- STS
categories:
- Dev

---

## 발생한 일

* 스프링 책 공부하면서 Controller / View 작성을 실습하는데, RequestMapping 및 잡다한 세팅을 제대로 했음에도 불구하고 해당 URI에서 404 에러가 나왔다.
* 이후 web.xml 확인, root-context.xml / servlet-context.xml 한번씩 체크하고 검색하는데 한시간 넘게 허비했다 ㅠㅠ

`
No mapping found for HTTP request with URI [/board/register] in DispatcherServlet with name 'dispatcher'
`

## 해결 방법

솔직히 해결 방법이라고 쓰기도 민망하지만, 의외의 방법으로 해결했다..

1) Project 메뉴 -> Clean.. 기능 실행

2) Servers 탭에서 Tomcat 우클릭 -> Clean.. 기능 실행

3) 톰캣 내렸다 올려서 다시 확인

예전에 maven 문제 생겼을 때도 .m2 폴더 삭제나 maven clean으로 허무하게 해결한 적이 몇 번 있었는데 이번이 그런 느낌이었다.

다음부터는 알 수 없는 문제가 발생했을 때, 우선 Clean부터 해봐야겠다는 생각이 들은 한시간이었다..
