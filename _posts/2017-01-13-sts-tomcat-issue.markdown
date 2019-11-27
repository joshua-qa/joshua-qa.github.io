---
layout: post
title: STS에서 실행한 톰캣 프로세스를 종료하기
date: 2017-01-13 01:47:05
tags:
- STS
- tomcat
categories:
- Dev
---

## 오류 현상

STS 종료가 제대로 안되어서 강제 종료를 했는데, STS에서 구동시킨 톰캣은 종료처리가 되어있지 않았다.

재 실행 한 뒤에 톰캣 Start 시도해보니 이미 구동중인 서버가 있어서 안된다고 메시지 노출..



## 해결 방법

다음 URL을 참고하여 조치했다.

http://stackoverflow.com/questions/4341082/how-to-kill-tomcat-when-running-it-from-eclipse

여러가지 방법이 명시되어 있으나, 윈도우 기준으로는 JAVA에 관련된 프로세스를 종료 시키는 것이 가장 편한 것 같다.

프로세스 종료 후 STS에서 톰캣 Start 시도해본 결과 성공!
