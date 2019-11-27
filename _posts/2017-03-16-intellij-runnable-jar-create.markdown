---
layout: post
title: "[IntelliJ] 실행 가능한 jar 만들기"
date: 2017-03-16 22:46:17
categories:
- Dev
tags:
- Intellij
- java
---

## 발생한 일

트위터 봇을 만든 뒤, ec2에 배포하려고 jar 파일을 생성했는데 Manifest 오류가 나며 서버에서 실행되지 않았다.

확인해본 결과 소스 폴더에 제대로 Manifest 관련 파일이 들어있었고 메인 클래스 설정도 잘 잡혀있었다.



## 해결 방법

인텔리제이에서 기본으로 만들어주는 Manifest 파일이 Maven을 사용할 때는 제대로 인식되지 않아 발생하는 문제였다.

https://intellij-support.jetbrains.com/hc/en-us/community/posts/206872335-How-to-create-executable-JAR-using-Intellij-

이 링크에서 젯브레인 측의 답변을 보면, 다음과 같이 답변하고 있다.

`If you're using Maven you need to put META-INF directory under main/resources folder instead of main/java.`

anifest 파일이 들어있는 META-INF 디렉토리를 main/resources로 이동한 뒤 빌드하면 해결된다.
