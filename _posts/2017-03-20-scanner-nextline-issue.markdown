---
layout: post
title: "[Java] Scanner nextLine() 사용 시 주의할 점"
date: 2017-03-20 14:29:44
categories:
- Dev
tags:
- java
---

## 발생한 문제

BOJ에서 Java로 알고리즘 문제를 푸는데, 다음과 같은 형식으로 입력을 받는 문제가 있었다.

	2
	OOXXOXXOOO
	OXOXOXOXOXOXOX
케이스가 몇 개인지 입력 받고, 그 뒤에는 String을 입력 받는 것인데 여기서 문제가 발생했다.

```java
Scanner scan = new Scanner(System.in);
int num = scan.nextInt();
String[] ox = new String[num];

for (int i = 0; i < num; i++) {
	ox[i] = scan.nextLine();
}
```

위와 같은 상황에서 ox[0]에는 첫 번째 입력한 String이 들어가지 않고, ox[1]에 들어가버린 것이다.

원인을 살펴보니 숫자 입력 단계에서 발생한 개행 문자가 ox[0]에 들어가고 String 입력은 ox[1]에 들어가기 시작한 것이었다.



## 해결 방법

검색 결과 여러가지 해결 방법이 있었지만.. 제일 간단하게 해결할 수 있는 방법은 scan.nextLine을 중간에 하나 넣어서 개행 문자를 미리 읽어버리도록 하는 것이었다.

```java
Scanner scan = new Scanner(System.in);
int num = scan.nextInt();
String[] ox = new String[num];
scan.nextLine();

for (int i = 0; i < num; i++) {
	ox[i] = scan.nextLine();
}
```

이렇게 구현해서 해결!



알고리즘 문제 풀다보면 가끔씩 이런 일이 발생하니 잘 기억해둬야겠다.
