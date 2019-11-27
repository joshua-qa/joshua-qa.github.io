---
layout: post
title: Java 인접리스트 구현하기
date: 2018-10-01 00:35:49
tags: 
 - java
 - PS
categories: 
 - java
 - PS
---

### Java 인접리스트 구현

그래프 문제를 풀 때는 그래프 모델링을 해야하는데, 보통 잘 알려진 방법은 두 가지가 있다.

1. 인접행렬
2. 인접리스트

백준 오프라인 강의 들을 때 간선리스트라는 개념도 배웠지만, 잘 알려진게 아니고 나도 당시에 이해하고 넘어간게 아니라서 생략..



어지간한 문제 풀다보면 인접리스트를 많이 사용하게 되는데, 솔직히 자바로 짜면 C++보다 아주 조금 더 귀찮다.

`List<Integer>` 배열을 만들던지, 아니면 List안에 `List<Integer>`를 집어넣어서 표현하던지 하고, 각각 반복문 돌려서 `new ArrayList<>()` 같은 처리도 해줘야 하고...

꼭 이렇게 해줘야한다는 규칙은 없지만 다른 사람들 코드를 봐도 보통 이 두가지를 많이 쓰는 것 같다.

여튼 개인적으로 문제 풀 때 써먹기 위해서 (그리고 솔직히 매번 입력해주는게 귀찮아서) 클래스를 간단하게 짰다.



```java
class Graph {
    private List<List<Integer>> graph;
    public Graph(int initSize) {
        this.graph = new ArrayList<>();
        for(int i = 0; i < initSize+1; i++) {
            graph.add(new ArrayList<>());
        }
    }
    public List<List<Integer>> getGraph() {
        return this.graph;
    }
    public List<Integer> getNode(int x) {
        return this.graph.get(x);
    }
    public void put(int x, int y) {
        graph.get(x).add(y);
        graph.get(y).add(x);
    }
    public void putSingle(int x, int y) {
        graph.get(x).add(y);
    }
}
```



메서드 이름도 나 혼자 알아보게 짰고, 전체적으로 되게 엉성하다.

각각 설명하자면 다음과 같다.

* `public Graph(int initSize)`
  * initSize만큼의 연결 정보를 담을 수 있는 그래프를 생성한다.
  * for문을 initSize+1 만큼 돌려주는 이유는, 그래프 문제에서 들어오는 인풋이 1~n 기준인 경우가 많고 getNode를 할 때 편하게 가져오기 위해서 실제 개수보다 한개 더 많게 ArrayList를 만들어주는 것이다. (ArrayList는 첫 인덱스가 0이니까..)
* `public List<List<Integer>> getGraph()`
  * graph를 리턴해준다. 별로 쓸 일은 없음.
* `public List<Integer> getNode(int x)`
  * x번째 노드를 가져와준다. 이렇게 해놓으니까 여러 상황에서 써먹을 수 있어서 좋았다.
* `public void put(int x, int y)`
  * 무방향 그래프 (x가 y로 연결되어있고 y도 x로 연결되어있고 그런거) 일때는 x랑 y 각각에 서로를 등록해줘야 하는데 이게 귀찮아서 만든 메서드.
* `public void putSingle(int x, int y)`
  * x번째 노드에 y를 등록해준다.

사실 근데 이 정도만 있어도 딱히 못 쓸건 없고, 굉장히 편하다. 문제가 요구하는 내용에 맞게 메서드 추가하거나 자료형 바꿔주거나 해서 쓰면 된다. (다익스트라를 풀기 위해 Integer대신에 개인적으로 만든 Vertex 클래스를 쓴다던가)

문제 풀때마다 일일히 인접리스트 구현 코드 작성하다보면 정말 가끔 기억이 안나서 코딩 실수 할 때가 있었는데 이런식으로 해놓으니 편하더라.

워낙 간단한 내용이라 이 코드 자체가 다른 사람에게 도움이 될 것 같지는 않고, 실제 상황 (기업 코딩 테스트, 집에서 혼자 백준 풀어보기, 코드포스 참가 등)에서 시간을 절약하기 위해 필요한 자료구조들을 미리 구현해놓는 것은 중요하다고 생각해서 써봤다.

이것을 사용해서 푼 문제는 아래에서 확인해볼 수 있다.

[코드 링크](https://github.com/joshua-qa/PS/blob/master/BOJ/14000/14619.java)
