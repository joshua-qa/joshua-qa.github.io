---
layout: post
title: "[Spring Boot] JPA @EntityGraph 사용 시 Entity graph specified is not applicable to the entity 이슈"
date: 2020-11-26 22:22:11
tags: Spring
categories: Dev
---

## [Spring Boot] JPA @EntityGraph 사용 시 Entity graph specified is not applicable to the entity 이슈

최근에 업무를 진행하면서, 담당하는 API 서버의 스프링 부트 버전을 2.3.3으로 올릴 일이 생겼다.

기존에 1.5.2였던 서비스를 2.3.3으로 올리는 것이었기 때문에 신경쓸 포인트가 생각보다 많았는데 그 과정에서 겪은 '사소한 문제'들을 조금씩 올려볼까 한다.

이 서비스의 경우 JPA를 사용하고 있고, 일부 Repository 메서드에 `@EntityGraph`가 붙어있는 상황이었다.

전환 과정에서 HikariCP쪽 설정이나 잘 잡아주면 되겠지라고 생각했더니, 작업 완료한 코드를 운영 환경에 배포하자마자 아래와 같은 WARN 로그가 올라오기 시작했다.

```
o.h.engine.internal.TwoPhaseLoad : Entity graph specified is not applicable to the entity [XXXXXXXXXXXXXX(XXX=aa, XXX=bb, XXX=null)]. Ignored.
```

사실 이 문제는 개발 환경에서 발견할 수 있었던 문제지만, 주로 ERROR 로그에만 집중하고 WARN은 거의 신경쓰지 않았던 까닭에 놓친 것이었다.  
게다가 실제 동작에는 이상 없던 상황이었다. 어쨌든, 기존에 발생하지 않았던 오류 로그임은 분명하기에 원인을 파악하였다.


## 문제 발생 원인

다행히 해당 문제에 대한 SO 질문을 찾는데에는 그리 오랜 시간이 걸리지 않았다.
https://stackoverflow.com/questions/63485177/spring-data-jpa-using-entitygraph-causes-entity-graph-specified-is-not-appli

`Spring Boot Starter Data JPA 2.3.3.RELEASE` 버전의 경우 `hibernate-core:5.4.20.Final` 의존성이 있는데 이 버전에서 발생하는 버그라고 한다.

해당 버그에 대한 JIRA 이슈는 아래와 같다.

https://hibernate.atlassian.net/browse/HHH-14124

https://hibernate.atlassian.net/browse/HHH-14212

모든 내용을 자세히 이해한 것은 아니지만 Fetch Graph 관련된 구현으로 발생한 문제였으며, 이를 원래대로 되돌리고 일부 코드를 추가하면서 픽스된 것으로 확인되었다.

자세히 들여다보고 싶었지만 많은 시간이 걸릴 것으로 예상되어 더 깊게 알아보진 않았다. (나중에 여유가 된다면 꼭..)

해당 이슈를 수정한 PR은 [이 링크](https://github.com/hibernate/hibernate-orm/pull/3548) 를 참고하자.

수정 버전은 5.4.22.Final이며 아래 세가지 방법 중 하나를 선택하여 적용하면 된다.

1) 다운그레이드  
-> 5.4.18 이하로 다운그레이드 하면 해당 현상이 발생하지 않는다고 한다. [참고 링크](https://stackoverflow.com/a/63769461)

1) 5.4.22.Final로 업그레이드  
-> 가장 단순하게 해결할 수 있는 방법이어서 나는 이 것을 적용했다.

3) `Spring Boot Starter Data JPA 2.3.5.RELEASE` 버전 이상으로 업그레이드  
-> hibernate-core 5.4.22.Final 버전을 사용하는 것을 확인 가능 (https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa/2.3.5.RELEASE 참고)

## 결론

내가 가져다 쓴 것들이 항상 잘 동작할 것이라고 생각하면 안되며, WARN 로그도 꼼꼼히 살펴보자는 교훈을 얻게 되었다.

앞으로도 스프링 부트 버전 올릴 때 의존성 버전 같은거 더욱 신경 써서 작업해야겠다 :)