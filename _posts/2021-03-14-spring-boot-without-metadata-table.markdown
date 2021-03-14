---
layout: post
title: "[Spring Batch] 메타 테이블 없이 스프링 배치 구성하기"
date: 2021-03-14 15:25:06
tags: Spring, Spring Batch
categories: Dev
---

## [Spring Batch] 메타 테이블 없이 스프링 배치 구성하기

최근 스프링 배치로 간단하게 진행한 프로젝트가 있었는데, 이 과정에서 알게 된 사실이 있어 공유해보려고 한다.

스프링 배치는 어플리케이션 코드만 작성해서 사용할 수 있는게 아니라, 메타 데이터 테이블이라는게 있어야 정상적으로 사용할 수 있다. (이하 내용에서는 메타 테이블로 설명하겠다)

이에 대한 설명은 창천향로님의 블로그에 아주 자세히 설명되어 있기 때문에, 해당 게시글의 `2-3. MySQL 환경에서 Spring Batch 실행해보기` 파트를 읽어보는 것을 추천한다.

[게시글 링크](https://jojoldu.tistory.com/325?category=902551) 

위 내용을 읽어 봤으면 한 가지 의문이 생길지도 모르겠다.

> '그럼 메타 테이블을 만들지 않고 스프링 배치 쓰려면 어떻게 해야하나요?'

참고로 이번에 작업한 배치 프로그램에서 사용한 DB는 MySQL이며, 별도로 협의한 내용에 따라 메타 테이블 없이 작업하기로 결정했다.

datasource로 사용하는 DB가 해당 프로젝트 전용으로 쓰는 DB도 아니었거니와, 메타 테이블을 사용하기 위하여 H2 DB를 별도로 사용할 생각도 없었기 때문이다.

운영 환경에 배포해서 계속 사용하고자 한다면 메타 테이블은 필수 불가결한 사항이기 때문에 충분히 고려하여 구성했겠지만.. 개발 환경에서만 돌려보고 종료시킬 프로젝트는 그럴 필요가 없기도 했다.

그럼 이제 `메타 테이블 없이 스프링 배치 구성하는 법`을 한번 알아보자.

## MapJobRepositoryFactoryBean 사용하기

처음 작업한 방식은 아래와 같다.

```java
public class BatchApplication extends DefaultBatchConfigurer {
... (중략) ...

@Override
protected JobRepository createJobRepository() throws Exception {
    MapJobRepositoryFactoryBean factory = new MapJobRepositoryFactoryBean();
    factory.setTransactionManager(new ResourcelessTransactionManager());
    factory.afterPropertiesSet();
    return factory.getObject();
}
```

`MapJobRepositoryFactoryBean`과 `ResourcelessTransactionManager`를 가지고 JobRepository를 구성하는 방법이다.

MapJobRepositoryFactoryBean의 [javadoc](https://docs.spring.io/spring-batch/docs/4.2.x/api/org/springframework/batch/core/repository/support/MapJobRepositoryFactoryBean.html)을 읽어보면 아래와 같은 설명이 나온다.

> A FactoryBean that automates the creation of a SimpleJobRepository using non-persistent in-memory DAO implementations. This repository is only really intended for use in testing and rapid prototyping. In such settings you might find that ResourcelessTransactionManager is useful (as long as your business logic does not use a relational database). Not suited for use in multi-threaded jobs with splits, although it should be safe to use in a multi-threaded step.

비 영구적인 인메모리 DAO 구현이며 `테스트 및 프로토타이핑`에만 사용하기 위한 것이라고 나와있다.

[내부 코드](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/repository/support/MapJobRepositoryFactoryBean.java) 를 살펴보면 알겠지만, 별도의 datasource 의존 없이 ConcurrentHashMap으로 메타 데이터를 관리하는 것을 알 수 있다.

-> [MapJobExecutionDao](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/repository/dao/MapJobExecutionDao.java), [MapJobInstanceDao](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/repository/dao/MapJobInstanceDao.java), [MapStepExecutionDao](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/repository/dao/MapStepExecutionDao.java), [MapExecutionContextDao](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/repository/dao/MapExecutionContextDao.java) 를 같이 참고하자.

MapJobRepositoryFactoryBean을 사용하면 배치 프로그램이 실행되고 있는 동안에만 해당 데이터가 유지되며, 종료한 뒤에는 없어지니 간단한 테스트 용도로는 적합하다고 할 수 있겠다.

다만 몇 가지 주의해야할 사항이 있는데, 이에 대해서는 아래 `주의할 점` 파트에서 설명하려고 한다.

## 더 간편한 방법

위 방법과 동일하지만, 더 간단한 방법이 있다.

[해당 방법을 설명한 StackOverflow 링크](https://stackoverflow.com/questions/25077549/spring-batch-without-persisting-metadata-to-database)

위에 제시한 방법대로 `createJobRepository()` 메서드를 오버라이드 할 필요도 없이, `setDataSource()` 메서드를 오버라이드하여 비워놓기만 하면 의도한대로 동작하는 것을 확인할 수 있다.

```java
public class BatchSampleApplication extends DefaultBatchConfigurer {

    @Override
    public void setDataSource(DataSource dataSource) {
        // 여기를 비워놓는다
    }
```

아래 명시한 [예제 Repo](https://github.com/joshua-qa/batch-without-metatable)를 받아서 실행해보면 로그를 확인할 수 있는데, 그 내용을 확인해보자.

![image](/images/spring-batch-without-metatable-log.png)

WARN 로그만 읽어보면 된다. datasource와 tranaction manager가 없어서 `Map based JobRepository`와 `ResourcelessTransactionManager`를 사용한다는 내용이다.

여기서 말하는 `Map based JobRepository`가 바로 위에서 언급한 MapJobRepositoryFactoryBean으로 만들어지는 JobRepository다.

그럼, 왜 이런 동작이 발생하는지 내부 구현을 살펴보자.

## 내부 구현

[DefaultBatchConfigurer 코드](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/configuration/annotation/DefaultBatchConfigurer.java) 를 살펴보면 아래와 같은 initialize 작업을 발견할 수 있다. (편의상 일부 내용만 가져왔다)

```java
if(dataSource == null) {
    logger.warn("No datasource was provided...using a Map based JobRepository");

    if(getTransactionManager() == null) {
        logger.warn("No transaction manager was provided, using a ResourcelessTransactionManager");
        this.transactionManager = new ResourcelessTransactionManager();
    }

    MapJobRepositoryFactoryBean jobRepositoryFactory = new MapJobRepositoryFactoryBean(getTransactionManager());
    jobRepositoryFactory.afterPropertiesSet();
    this.jobRepository = jobRepositoryFactory.getObject();

    MapJobExplorerFactoryBean jobExplorerFactory = new MapJobExplorerFactoryBean(jobRepositoryFactory);
    jobExplorerFactory.afterPropertiesSet();
    this.jobExplorer = jobExplorerFactory.getObject();
} else {
    ... 생략 ...
}
```

초기화 과정에서 datasource를 발견하지 못하면 MapJobRepositoryFactoryBean을 사용하며, Transaction manager도 없는 경우 ResourcelessTransactionManager도 같이 사용하는 것을 확인할 수 있다.

위에서 보여줬던 코드와 별 차이 없는 모습이다.

다만 이 과정을 거치기 위해서는 주입 받을 수 있는 datasource가 존재하지 않아야하며, 배치 사용을 위해 설정한 datasource가 있는 경우 setDataSource 메서드를 오버라이드 하여 아무런 동작을 하지 않게 만들어줘야 한다.

## 예제 Repo

[https://github.com/joshua-qa/batch-without-metatable](https://github.com/joshua-qa/batch-without-metatable)


## 주의할 점

해당 방식을 사용하는 것에 대해 주의할 점이 있다.

### 최근 버전에서는 `MapJobrepositoryFactoryBean`이 deprecated 상태임

[스프링 배치 4.3.1 문서의 MapJobRepositoryFactoryBean 설명](https://docs.spring.io/spring-batch/docs/current/api/org/springframework/batch/core/repository/support/MapJobRepositoryFactoryBean.html)을 한번 읽어보자.

> Deprecated. as of v4.3 in favor or using the JobRepositoryFactoryBean with an in-memory database. Scheduled for removal in v5.0.

인메모리 DB와 함께 `JobRepositoryFactoryBean`을 사용할 것을 권고하고 있다. 심지어 5.0에서는 제거할 예정이 있다고 한다.

클래스 설명에도 적혀있지만, 분할이 있는 멀티스레드 환경 작업에는 적합하지 않기 때문에 더이상 사용하지 않는 것이 아닐까 추측해본다.

따라서 이 방식은 스프링 배치 4.x 버전까지만 유효하며, 5.0 부터는 사용할 수 없으니 참고하시기 바란다.

### 운영 환경에서 사용하지 말기

이 방식은 어디까지나 테스트 및 프로토타이핑에 적합한 방식이기 때문에 실제 운영 환경에서 구동해야 하는 배치인 경우 반드시 메타 테이블을 사용하는 것이 좋다.

## 마무리

이전 회사에서 php5 환경으로 구성된 배치를 유지보수할 일이 있었는데, 각종 예외처리로 점철된 코드와 번거로운 구동 방식에 당황했던 기억이 난다.

그에 비해 스프링 배치는 훨씬 간편하게 코드를 작성할 수 있었고, 비즈니스 로직에만 집중하면 되기 때문에 너무나도 편하다는 생각이 들었다.

한편으로는 제대로 사용하기위해 공부해야할 것이 많다는 생각도 들었고, 앞으로도 많이 만들어보며 학습해야 될 것 같다.

어지간한 트러블슈팅이나 설정 방법은 StackOverflow나 좋은 사이트들(baeldung이라던가..)을 참고하면 해결할 수 있지만, 레퍼런스와 내부 구현 코드를 확인하며 구성하면 더욱 확신을 얻을 수 있으므로 같이 읽어보도록 하자.
