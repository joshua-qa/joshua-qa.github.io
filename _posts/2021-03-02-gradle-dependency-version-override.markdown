---
layout: post
title: "[Gradle 6.x] 멀티 모듈 환경에서 특정 의존성 버전 Override하기"
date: 2021-03-02 20:34:34
tags: 
 - Gradle
 - Spring
categories: Dev
---

## [Gradle 6.x] 멀티 모듈 환경에서 특정 의존성 버전 Override하기

요즘 개발하다보면 가장 속썩이는 것이 바로 Gradle이다.

무슨 문제가 발생했을 때, 공식 문서를 보고 따라해도 잘 모르겠고.. 그렇다고 이곳저곳 찾아서 나온 방법을 적용해보자니 사람마다 설명하는 해결책이 천차만별이라 어떤 것이 맞는지는 내가 직접 빌드해보면서 알아내야 하는 문제가 있다.

덕분에 오늘도 많은 시간을 Gradle 트러블슈팅으로 보냈고, 날잡아서 Gradle을 제대로 학습해야겠다는 생각이 들었다.


## 발생한 문제

현재 진행중인 스프링부트 프로젝트에서 `spring-data-elasticsearch`를 사용하고 있는데, Pre-Release 버전에 추가된 기능 때문에 해당 의존성의 버전을 변경할 일이 생겼다.

```
---- api ---- implementation project(":domain")
|
---- core
|
---- domain ---- api("org.springframework:spring-data-elasticsearch")
```

프로젝트 구조를 요약하면 위 모습과 같다. (실제로는 더 복잡하지만 일부 내용만 포함)

spring-data-elasticsearch 4.2.0-M4에서 [search_after 지원 관련 PR](https://github.com/spring-projects/spring-data-elasticsearch/pull/1691)이 반영되었고, 해당 기능을 사용하기 위해 4.2.0-M4로 버전을 override 했다.

refresh 이후 버전이 바뀔거라고 생각하였으나, 실제로는 그렇지 않았고 spring-boot에서 잡아주는 버전으로 지정되는 문제가 발생해서 해결을 시도하게 되었다.

## 시도한 방법

가장 쉬운 해결 방법은 서브모듈에 의존성을 선언하지 않고 루트에 있는 build.gradle에 선언해주는 것이다.

그러나, 이 방법을 사용하면 domain 모듈에만 들어가도 되는 `spring-data-elasticsearch` 의존성을 다른 서브모듈도 참조하게되는 문제가 있어서 별로 이 방법을 쓰고 싶지 않았다.

일단 해당 코드는 걷어내고, [레퍼런스 문서](https://docs.gradle.org/current/userguide/dependency_downgrade_and_exclude.html)에 있는 방법을 적용해보았다.

### strictly 적용
```
dependencies {
    implementation('org.springframework:spring-data-elasticsearch') {
        version {
            strictly '4.2.0-M4'
        }
    }
}
```
별 차이 없었다.

### force 적용
```
dependencies {
    implementation('org.springframework:spring-data-elasticsearch:4.2.0-M4') {
        force = true
    }
}
```
이 방법도 원하는대로 동작하지 않았다.

### resolutionStrategy.force 적용
```
configurations {
    compileClasspath {
        resolutionStrategy.force 'org.springframework:spring-data-elasticsearch:4.2.0-M4'
    }
}
```
이것마저 동작하지 않는 것을 보고, 내 gradle 버전이 문제인지.. 아니면 공식문서가 문제인지 고민하기 시작했다.

## 해결 방안

아래 링크를 참조하여 해결할 수 있었다.

https://stackoverflow.com/questions/28444016/how-can-i-force-gradle-to-set-the-same-version-for-two-dependencies

해당 링크에 있는 답변 중, `resolutionStrategy.eachDependency` 구문을 사용했더니 정상적으로 동작하는 것을 확인하였다.

```
configurations.all {
  resolutionStrategy.eachDependency { details ->
    if (details.requested.name == 'spring-data-elasticsearch') {
      details.useVersion "4.2.0-M4"
    }
  }
}
```

의존성 목록을 모두 순회하면서, if 조건을 만족하는 의존성이 발견되는 경우 사용 버전을 강제로 지정하는 것이다.

## 참고 문서

위 단락에 명시한 stackoverflow 링크를 찾기 위하여 검색을 다시해보던 도중에 해당 내용을 설명한 레퍼런스 문서를 발견했다.

작업하던 당시에는 발견하지 못했는데, 블로그에 글 쓰는 도중에 발견했다...

https://docs.gradle.org/current/userguide/resolution_rules.html

## 느낀 점

어느 툴이라고 안그렇겠냐만은, gradle이 유독 버전을 많이 타는 것 같다.

뭔가 문제가 발생했을 때 해결책을 찾아서 적용해보면 빌드 오류가 발생하는 일이 너무 많고, 그에 대한 원인 추적도 어렵게 느껴졌다.

책을 보던지, 레퍼런스 문서를 보던지 하면서 빡세게 학습하는 수 밖에 없겠다. (이 글 역시 좀 더 공부해본 뒤에 수정할 수 있음)

혹시 나처럼 이런 문제가 발생했는데 검색해서 나오는 방법으로 해결이 안되는 경우 도움 되었으면 좋겠다.
