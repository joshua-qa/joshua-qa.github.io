---
layout: post
title: "[ALGOSPOT] 여행 짐 싸기 (PACKING)"
tags:
  - PS
categories:
  - PS
date: 2017-10-22 21:34:39
---

#### ALGOSPOT - 여행 짐 싸기 (PACKING)

할만하다고 생각해서 도전해봤다.

분명히 DP 문제를 어느정도 풀어본 경험이 있지만.. Knapsack 문제는 익숙하지 않아서 시간이 좀 걸렸다.

일단 이 문제는 일반적인 Knapsack 문제랑 비슷하면서도 신경써줘야 할 점이 하나 더 있는데, 최적값을 달성하기 위해 조합한 물건들의 리스트를 출력해줘야 한다.

처음에 이것 때문에 설계가 오래걸렸는데 배열을 하나 더 만들어주면 해결된다.

*   `DP[i][j]` = i번째까지 무게 j만큼 적절하게 골라서 담았을 때, 최대 가치값(절박도)
*   `used[i][j]` = 무게 j만큼 골라서 담았을 때, i를 사용했는지 체크

대충 이런식으로 설계했고, 문제 푸는데 사용한 점화식은 여기에 쓰지 않고 생략하려고 한다.

점화식이 잘 세워지지 않는 경우 가지고 있는 알고리즘 서적을 참고하거나, [GeeksforGeeks](http://www.geeksforgeeks.org/knapsack-problem/) 사이트 설명을 참고 하는 것을 매우 추천한다. (설명 보면서 공책에 배열 전개도 그려보면, 왜 이렇게 해야하는건지 이해도 잘 되고 좋다)

DP배열을 다 채웠으면, 최대 가치값(이 문제에서는 절박도)이 되는 무게값을 저장해뒀다가 그걸 이용해서 사용한 물건을 출력해주면 된다.

예를 들어서 가장 마지막으로 담은 물건이 30번째 물건이고, 캐리어에 최대 가치값을 만족하도록 채워넣은 무게값이 320이라고 하면 `used[30][320]` 배열에 1이 체크되어 있을 것이다.

그럼 스택이나 리스트에 30을 넣어주고, `used[30-1][320 - weight[30]]` 이런 식으로 감소해가면서 사용한 물건을 찾으면 된다. 해당 부분 코드는 다음과 같다.

```java
int index = n;
while(maxValueWeight >= 0 && index >= 0) {
  if(used[maxValueWeight][index] == 0) {
    index--;
    continue;
  } else {
    storeList.push(list[index]);
    maxValueWeight -= weight[index];
    index--;
  }
}

sb.append(max + " " + storeList.size()).append("\n");
while(!storeList.empty()) {
  sb.append(storeList.pop()).append("\n");
}
```

간만에 굉장히 재밌는 문제였고, 앞으로도 DP를 자주 풀어봐야겠다는 생각이 들었다.



[전체 코드 링크](https://github.com/joshua-qa/PS/blob/master/algospot/packing.java)


