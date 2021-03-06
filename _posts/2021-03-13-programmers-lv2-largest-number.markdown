---
layout: post
title: "[프로그래머스] 레벨 2 - 가장 큰 수"
date: 2021-03-13 22:12:13
tags: PS
categories: PS
---

## 프로그래머스 레벨 2 - 가장 큰 수 (https://programmers.co.kr/learn/courses/30/lessons/42746)

두 달 전쯤, 프로그래머스 스킬 체크를 잠깐 해본 적이 있었다.

레벨1은 별 문제 없이 통과했으나, 레벨2에서 생각지도 못한 문제를 만나 고전했던 기억이 난다. 내가 레벨2를 너무 우습게 봤던 것일까.. 😂

그 문제는 바로 `가장 큰 수` 문제였다.

당시 충격이 심했던 것인지 한동안 잊고 살다가, 오늘 레벨2를 재도전 했다. 다행히 별 어려움 없이 통과할 수 있었으며 `가장 큰 수` 문제도 쉽게 해결했다.

풀어서 기분은 좋았지만 쉬운 문제를 너무 어렵게 접근했던 것 같아서 많은 생각이 들었고, 까먹기 전에 풀이를 기록하려고 한다.

## 접근한 방법 (처음)

처음 접근했던 방식은, 입력받은 숫자들을 자리수별로 꼼꼼하게 비교하여 내림차순 정렬하는 것이었다.

가령 `[3, 30, 34]` 라는 숫자를 가지고 `가장 큰 수`를 만들기 위해서는 맨앞 자리에 있는 숫자도 중요하겠지만, 그 숫자를 구성하는 중간 자리와 끝 자리 숫자들도 서열을 일일히 비교해야 된다고 판단했던 것이다.

결국 이 방식으로 접근하니 Comparator 코드가 비정상적으로 길어졌고, 일부 케이스만 통과하는 문제점이 발생했다.

해당 방식을 적용하려면 모든 경우의 수를 고려하여 처리해야했는데..................... 잘 생각해보면 레벨2짜리 문제에 이정도까지 고려할 필요도 없으며 예외 케이스가 너무 많았다.

## 접근한 방법 (오늘)

이 문제의 특징을 다시 한 번 열심히 생각해보았다.

특징1) 주어진 숫자를 분해할 수 없다.

특징2) 주어진 숫자가 A랑 B라고 가정했을 때 [A를 B앞에 붙이던가, B뒤에 붙이던가] 두가지 행위만 가능하다.

특징3) 주어진 숫자가 [A, B, C]이고 이 문제에서 요구하는 정답을 만들 수 있는 서열이 [C, A, B] 라고 할 때, 이 목록에서 A를 제거하거나 새로운 숫자 D를 추가한다고 하더라도 C가 B보다 앞선다는 사실은 변함이 없다.

특징4) 결국 `특징2`의 내용을 기반으로 했을 때 두 숫자를 비교하는 기준은 '앞에 붙이는게 유리함? 뒤에 붙이는게 유리함?'만 판단하면 된다.

여기까지 생각하고나서, '아차' 싶었다. 기존에 내가 너무 어렵게 생각하고 접근했다는 걸 깨달았고, 아래와 같이 간단하게 compare를 짰다.

```java
Comparator<Integer> integerComparator = (o1, o2) -> {
    String s1 = String.valueOf(o1) + o2;
    String s2 = String.valueOf(o2) + o1;
    return Integer.parseInt(s2) - Integer.parseInt(s1);
};
```

위 Comparator를 사용하여 정렬하는 경우, 아래와 같은 동작이 발생한다.

```
input : [30, 34]

s1 = "30" + "34" = "3034"
s2 = "34" + "30" = "3430"

-> 3034 < 3430
-> sort result : [34, 30]
```

## 주의할 점
이 문제는 주의할 점이 하나 더 있는데 input에 중복된 숫자가 들어올 수 있다는 것이다.

즉, [0, 0, 0, 0] 같은 극단적인 입력값이 들어올 수 있다는 걸 충분히 유의하고 예외처리를 진행해야한다.

정렬된 리스트를 단순히 하나씩 합친 뒤에 리턴하게 된다면, 위 입력값 기준으로 "0000"이 나오는데 실제로는 "0"을 리턴해야하니 문제가 발생한다.

해당 케이스를 반드시 체크하고 리턴할 수 있도록 신경쓰자.

간단하게 처리하고 싶다면, 리턴하려는 문자열의 맨앞자리가 0인 경우 "0"을 리턴하도록 처리해주면 편하다. 0보다 큰 숫자가 한개라도 존재했다면 맨앞자리가 0일 수 없으니까.

## 풀면서 그렸던 것

![image](/images/largest_number.png)

## 느낀 점

문제가 가진 성질을 제대로 파악하고, 단순하게 접근할 수 있는지 파악하는게 중요하다는 걸 다시 한 번 느끼는 계기였다.

오랜만에 문제 풀이에 재미를 느낄 수 있던 시간이었고 블로그에도 풀이를 더 많이 올릴 수 있도록 노력해야겠다.

## 전체 코드

[링크](https://github.com/joshua-qa/PS/blob/master/programmers/level2/LargestNumber.java)