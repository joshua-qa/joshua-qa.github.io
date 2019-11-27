---
layout: post
title: "[windows10] Vagrant 설치 후 booting VM에 실패했을 때"
date: 2017-02-18 17:45:26
tags:
- Vagrant
- Windows
categories:
- Dev
---

## 발생한 일

* windows10(최신 업데이트까지 다 한 버전)에 vagrant를 설치하고나서 ubuntu 세팅을 하려고 했는데, virtualbox 설치 과정까지 다 끝내고 `vagrant up`을 하니 Booting VM에 실패했다는 메시지가 나왔다.

		The guest machine entered an invalid state while waiting for it
		to boot. Valid states are 'starting, running'. The machine is in the
		'poweroff' state. Please verify everything is configured
		properly and try again.

		If the provider you're using has a GUI that comes with it,
		it is often helpful to open that and watch the machine, since the
		GUI often has more helpful error messages than Vagrant can retrieve.
		For example, if you're using VirtualBox, run `vagrant up` while the
		VirtualBox GUI is open.

		The primary issue for this error is that the provider you're using
		is not properly configured. This is very rarely a Vagrant issue.

* 확인을 위해 virtualbox를 실행해보니 아무것도 나오지 않았고, 제어판 -> 관리 도구 -> 이벤트 뷰어를 살펴본 결과 APPCRASH가 발생하며 실행되지 않은 로그를 발견했다.

## 해결 방법
* 검색해본 결과 windows10에서 같은 현상을 겪은 유저들이 많이 있었다. virtualbox의 최신 버전을 설치 후 해결했다는 글을 발견해서 바로 시도했다.
  * https://www.virtualbox.org/wiki/Testbuilds 여기서 베타 버전을 받아서 설치한 뒤 재부팅 -> vagrant up 재시도
  * 이후 정상적으로 동작하는 것을 확인했다.


![vagrant](/images/vagrant.png)
