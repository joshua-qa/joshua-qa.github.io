---
layout: post
title: "[Spring Boot] Active Profile 목록을 사용할 때는 순서에 주의하자"
date: 2020-09-13 14:56:37
tags: Spring
categories: Dev
---

## [Spring Boot] Active Profile 목록을 사용할 때는 순서에 주의하자

오랜만에 블로그 글을 쓴다.

이직 후 새로운 업무를 맡게 되었고, 그 중 아래와 같은 업무가 있었다. (구체적인 내용은 대외비로 서술하지 않음)

'대량의 파일을 삭제하는 배치'

열심히 사수님의 조언을 받아 완성 시켰고, 운영 서버에 배포해서 수행해본 결과 별다른 문제가 발생하지 않은 것처럼 보였다.

로컬 환경이랑 스테이지 환경에서도 성공적으로 돌아갔으니, 운영 서버에서도 당연히 잘 돌아갈 것이라 생각하며 열심히 돌렸는데 문제가 발생하고 말았다.

삭제 요청을 보낸 곳에서 성공적으로 삭제됐다고 응답이 돌아왔고, DB에도 업데이트를 쳤는데 정작 삭제 대상 서버에서는 파일 갯수가 줄어들지 않았다고 연락이 온 것이다.  
(삭제 요청을 보낸 서버에서는 파일이 존재하지 않는 경우 성공으로 응답을 보내는 구조였다)

| ![pepe](/images/0913-pepe.jpg) |

처음엔 내 코드에 문제가 없다고 판단하고 전혀 다른 쪽에서 원인을 찾고 있었는데, 운영 서버에 로깅을 몇가지 추가해서 배포한 뒤 문제점을 바로 발견하였다.

다행히 수정한 뒤에는 정상적으로 배치 수행 및 파일 삭제가 진행되고 있으며, 꼼꼼하지 못했던 나의 문제점을 반성하며 이 글을 작성하게 되었다.


## 문제 발생 원인

삭제할 파일이 있는 디렉토리의 경우, 스테이지 서버 용도로 쓰는 곳과 운영 환경 용도로 쓰는 디렉토리가 따로 있었는데 규칙이 아래와 같았다.

운영 환경 예시) `directory`, 스테이지 환경 예시) `directory-stg`

나는 이 규칙에 맞게 path 설정을 하기 위해서 아래와 같은 코드를 작성하였다.

```java
public String getProfileSuffix() {
    return "prod".equals(Arrays.stream(env.getActiveProfiles()).findFirst().orElse("local")) ? "" : "-stg";
}
```

보는 사람에 따라 지적할 포인트가 많을지도 모르겠다.

내가 생각하는 가장 큰 문제는 **activeProfiles 배열에 들어있는 객체 순서에 지나치게 의존**한다는 것이다.

하나 더 언급하자면, 배치 구동 시 argument에 넣은 `-Dspring.profiles.active` 값이 최우선으로 나올 것이라 믿은 것이 문제였다.

원인을 분석해본 결과 `application.yml`에 아래와 같은 설정이 포함되어 있었는데, 이 설정값이 위 코드에 큰 영향을 줬다.

```
spring:
    profiles:
        include: redis
    ...
```


편의를 위해, 해당 상황을 재현할 수 있는 코드를 간단하게 만들어보았다.

[코드 저장소 링크](https://github.com/joshua-qa/blog-example-active-profiles)

`spring.profiles.include` 설정 값이 있는 상태에서 argument에 profile을 별도로 넣고 구동해보자.

아래와 같은 순서로 active profile 목록이 찍히게 된다.

```
active profiles : [redis, prod]
```

include에 있는 값이 먼저 찍힌다. 위에 있는 코드도, 의도한 것과는 다르게 findFirst() 의 결과로 `redis`를 가져오게 될 것이다.

그동안 개발을 하면서도 `spring.profiles.include` 값을 사용해본 적이 없기에 전혀 예상하지 못한 결과였다.

`spring.profiles.active`를 먼저 가져온 뒤에 include에 있는 값은 뒤에 덧붙일줄 알았는데, 왜 이렇게 동작하는 것일까?


## 동작 원리

`SpringApplication` 클래스의 `run()` 메서드를 보면 아래와 같은 코드가 있다.

`ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);`

여기에 Breakpoint를 찍고 열심히 디버깅을 해도 되겠지만.. 시간을 절약하기 위해 IntelliJ의 기능을 활용하여 `spring.profiles.include`가 들어있는 클래스 위치를 찾아보자.  
(Find in Path -> Scope -> All Places로 설정 후 검색)

`ConfigFileApplicationListener` 클래스에서 `public static final String INCLUDE_PROFILES_PROPERTY = "spring.profiles.include";` 라는 코드가 보인다.

`INCLUDE_PROFILES_PROPERTY` 값을 사용하는 메서드를 추적한 뒤, 해당 메서드를 호출한 곳을 타고 올라가면 `load()` 라는 메서드가 나온다.

```java
List<Document> documents = loadDocuments(loader, name, resource);
if (CollectionUtils.isEmpty(documents)) {
    if (this.logger.isTraceEnabled()) {
        StringBuilder description = getDescription("Skipped unloaded config ", location, resource,
                profile);
        this.logger.trace(description);
    }
    continue;
}
List<Document> loaded = new ArrayList<>();
for (Document document : documents) {
    if (filter.match(document)) {
        addActiveProfiles(document.getActiveProfiles());
        addIncludedProfiles(document.getIncludeProfiles());
        loaded.add(document);
    }
}
```

`loadDocuments()` 메서드에서 active profile / include profile을 가져와 세팅하고 그걸 하나씩 순회하여 프로필을 등록해준다.

문제의 코드인 `addIncludedProfiles()` 메서드는 아래와 같다.

```java
// ConfigFileApplicationListener:594 (spring-boot:2.3.3.RELEASE)

private void addIncludedProfiles(Set<Profile> includeProfiles) {
    LinkedList<Profile> existingProfiles = new LinkedList<>(this.profiles);
    this.profiles.clear();
    this.profiles.addAll(includeProfiles);
    this.profiles.removeAll(this.processedProfiles);
    this.profiles.addAll(existingProfiles);
}
```

이미 있는 active profiles 목록을 리셋하고 새로 세팅해주는 것을 볼 수 있다.

1) `this.profiles`를 가져와 existingProfiles에 백업 후 내용물 삭제 (clear 호출)
2) includeProfiles를 `this.profiles`에 삽입
3) 백업해둔 existingProfiles를 `this.profiles`에 삽입

최종적으로 `this.profiles`에 들어있는 값의 순서는 `includeProfiles값 -> 기존 activeProfiles값` 가 될 것이다.

include에 대한 더 자세한 내용은 [공식 문서](https://docs.spring.io/spring-boot/docs/1.2.0.M1/reference/html/boot-features-profiles.html) 를 참고하자.


## 결론

해당 문제의 경우 결국 activeProfiles 배열에 의존하지 않도록 아예 새로운 코드를 작성하여 해결했고, 지금은 문제가 발생하지 않고 있다.

(배열을 하나씩 순회하여 일치하는 값이 있는지 확인해도 되었겠지만...)

사실 이런 실수는 해서 안되는 것이고, 지금도 꼼꼼하지 못했던 그 때의 나를 매우 반성하고 있다.

다행히 모든 분들이 이 문제점을 신속히 해결할 수 있는 방안들을 친절하고 빠르게 제시해주셨으며, 그 덕분에 패닉 상태에 빠지지 않고 문제를 해결할 수 있었다.

앞으로 `어설프게 아는 것 / 확실하지 않은 것`들은 더욱 정확한 내용을 살펴보고 개발 업무에 임해야겠다는 생각이 들었다.


# 예제 코드 저장소 링크
https://github.com/joshua-qa/blog-example-active-profiles