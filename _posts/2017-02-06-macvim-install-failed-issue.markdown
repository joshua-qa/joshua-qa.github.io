---
layout: post
title: '[OS X] Homebrew로 Macvim 설치 실패 후 해결한 이야기'
date: 2017-02-06 19:41:09
tags:
- homebrew
- macvim
- xcode
- issue
categories:
- Dev
---

## 발생한 문제

* MacVim 8.0을 설치해보고 싶어서 brew로 설치 시도했는데, 빌드 관련 오류가 발생하며 설치에 실패했다.
* homebrew 트러블슈팅 페이지를 참고해서 brew doctor도 해보고 이것저것 해봤으나 해결되지 않았다.



## 해결 방법

오류 메시지를 검색해본 결과 이미 같은 문제로 질문을 올린 사람이 있었다.

(https://github.com/nodejs/node-gyp/issues/569)



1. Install Xcode
2. Run `sudo xcode-select -s /Applications/Xcode.app/Contents/Developer`

우선 나같은 경우 xcode 및 커맨드라인 도구가 설치되어 있는 상태였기 때문에 2번에 적혀있는 명령어를 입력해봤다.

명령어 입력 후 별다른 반응은 없었지만, 이후 다시 설치 시도해보니 성공했다.

설치 완료 후 잘 됐는지 확인해보려면 mvim —version을 써주면 된다.
