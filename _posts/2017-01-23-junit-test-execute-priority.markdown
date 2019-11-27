---
layout: post
title: Junit 테스트 실행 순서 맞추기
date: 2017-01-23 21:01:34
tags:
- junit
- test
- unit test
categories:
- Dev
---

## 오류 현상

클래스 하나에 여러개의 테스트 코드를 넣고 [Run as -> Junit Test] 를 실행했는데, 작성한 순서대로 테스트가 동작하지 않았다.

공부중인 책에서 기본적인 CRUD 코드를 작성한 뒤에 테스트 코드도 CRUD 순서로 작성했는데, 정작 테스트 실행은 Read가 Create보다 먼저 되어서 Null pointer 오류 발생. (Create로 글을 하나 DB에 등록하고 그걸 Read해오는 코드였는데, 테스트 실행 당시 등록된게 없었으므로...)

STS 기준으로 Outline 탭에서 메소드를 우클릭하고 각각 테스트 따로 실행하면 문제가 없지만, 번거롭기도 했고 한번에 실행시켜보고 싶었다.

## 발생 원인 / 해결 방법

Junit은 테스트 코드의 실행 순서를 보장하지 않는다고 한다. 그동안 Selenium Webdriver 코드를 작성하면서도 몰랐던 사실이다.. (클래스 한개에 @Test를 딱 한개씩만 넣었었음)

하지만 4.11부터 테스트의 순서를 지정할 수 있어서, 해당 옵션을 지정해주고 메소드 이름을 맞춰주면 의도한대로 테스트 실행이 가능하다.

- 기본값 (보장안함) / JVM 반환 순서대로 (실행마다 다를 수 있음) / 각 메소드의 이름 순으로

이렇게 세가지 옵션이 있는데, 각 메소드의 이름 순으로 맞춰주는 방법을 선택했다.

```java
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class BoardDAOTest {
	// inject 생략
	@Test
	public void TestA() throws Exception {
	    ...
	}

	@Test
	public void TestB() throws Exception {
	    ...
	}

	...
}
```

http://junit.org/junit4/javadoc/latest/org/junit/runners/MethodSorters.html

이곳에 자세한 설명이 있어서 도움이 되었다.

원래 사용했던 메소드 이름은 `testCreate, testRead, testUpdate, testDelete` 였는데 중간에 숫자를 넣어서 순서를 맞춰줬더니 테스트가 순서대로 돌아가는 것을 확인할 수 있었다.

![](/images/junit.png)

## 느낀 점

가급적이면 한 테스트가 다른 테스트에 의존하지 않도록 분리해주거나, 순서를 잘 정해주는 식으로 구현해야 될 것 같다.

단위 테스트 도구로 TestNG도 사용해본 적이 있었는데, 이런 경우에 어떻게 처리하는지 찾아봐야겠다.
