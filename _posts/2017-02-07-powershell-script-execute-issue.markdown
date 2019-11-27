---
layout: post
title: "[Elixir] PowerShell에서 mix 명령어 실행이 안되는 경우 해결방법"
date: 2017-02-07 23:28:31
tags:
- Elixir
- mix
- PowerShell
categories:
- Dev
---

## 발생한 상황

* Elixir 프로젝트를 생성하기 위해 커맨드라인 (PowerShell)에서 `mix new 프로젝트명` 을 입력했는데, PSSecurityException이 발생하며 실행되지 않았다.



## 해결 방법

* PowerShell의 스크립트 실행 권한을 변경해줘야한다.

  * ExecutionPolicy 명령어 입력했을 때 'Restricted'라고 나오면 스크립트 실행 권한이 없는 상태이며, 'Unrestricted'라고 나오면 스크립트 실행 권한이 있는 것이다.
  * 관리자 계정으로 PowerShell을 실행한 후, Set-ExecutionPolicy Unrestricted 명령어 입력해주면 권한이 변경된다. 현재 사용중인 사용자 계정에서만 변경하려고 하는 경우 currentuser를 뒤에 붙여주면 된다.

권한 변경 이후 mix 사용이 정상적으로 되는 것을 확인했다.
