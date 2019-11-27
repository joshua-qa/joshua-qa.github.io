---
layout: post
title: "[Android] Android Emulator(AVD)에 Charles Proxy 세팅하기"
date: 2017-02-22 20:38:00
tags:
- charles
- Android
- AVD
categories:
- Dev
---

안드로이드 앱 개발하는중에, 로그 찍는것만으로는 확인하기 어려운 부분이 있어서 Charles Proxy로 확인해보려고 세팅을 했다.

기존에 테스트용으로 쓰던 기기를 친구 빌려주고 AVD로 가상 기기 켜서 작업하고 있었는데, 생각해보니 AVD에는 Charles 연결해본적이 없었다.

세팅할 때 주의할 점이 하나 있어서 메모.

#### 세팅 하는 법

* Android Studio에서 AVD Manager로 가상기기 실행하는 경우, 프록시 설정하는 부분이 보이지 않았다. 스택오버플로우를 찾아보니 커맨드라인에서 다음과 같이 해주면 된다고 나와있었다.

`emulator -http-proxy [IP주소:포트] @기기명`

* 기기명을 입력하기전에 emulator -list-avds로 한번 확인해주는 것이 좋다. 띄어쓰기가 이름에 들어있는 기기는 언더바로 채워지기 때문에.. (나같은 경우 Nexus 5 API 24로 등록한 기기가 `Nexus_5_API_24` 로 되어있었다)
* Charles Proxy를 먼저 실행해놓은 상태에서 해당 명령어를 입력해야 프록시 적용된 상태로 기기 실행이 가능하다.
* IP주소랑 포트명은 Charles Proxy의 Help > SSL Proxying > Install Charles Root Certificate on a .... 메뉴에서 확인해서 입력한다.
* 기기 실행이 완료된 이후, 인증서를 받아서 설치해야하는데 가상기기 내부에 있는 크롬을 실행해서 chls.pro/ssl 접근한 뒤 받으면 된다. 보안 설정을 하라고 나올 것인데, 적당히 PIN번호 설정 해주고 인증서 설치하면 완료.
* Proxy > SSL Proxying Settings에서 Enable SSL Proxying 체크 및 프록시 캡쳐할 주소 등록해주면 된다.
