---
layout: post
title: spring에 log4j 연결하다가 발생한 오류
date: 2017-01-13 19:44:18
tags:
- Spring
- Log4j
categories:
- Dev
---

## 오류 현상

`코드로 배우는 스프링 웹 프로젝트 `책을 보고 실습하다가, Log4J를 연동하는 내용이 있었는데 많은 오류를 뱉어내며 동작하지 않았다.

에러 로그를 확인해본 결과 DriverClassName 관련 내용이 나왔지만, 몇 번을 확인해봐도 오타는 없었다 ㅠㅠ



## 해결 방법

세팅 파일 넣은 폴더 경로가 잘못되어서 발생한 일이었다. (logback.xml, log4jdbc.log4j2.properties)

src/main/resources 폴더가 있고, src/main/webapp/resources 폴더가 있는데 제대로 확인하지 않고 webapp/resources에 넣은 것이 화근이었던 것..

해당 파일들을 src/main/resources 폴더로 이동한 뒤 이상없음을 확인했다.

얼핏 같아 보이는 경로일수록 잘 확인해야된다는 교훈을 얻었다 ㅠㅠ
