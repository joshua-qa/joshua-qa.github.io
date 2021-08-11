---
layout: post
title: "[Spring WebFlux] Multipart 파일 업로드 - 한글 파일명 이슈"
date: 2021-08-11 20:49:07
tags: 
 - Spring
 - WebFlux
categories: Dev
---

## [Spring WebFlux] Multipart 파일 업로드 - 한글 파일명 이슈

최근, 회사 업무에서 WebFlux를 매우 많이 사용하고 있다.

그동안 많이 익숙해지기도 했고, 재미도 있어서 다행이지만.. 기존에 쓰던 Spring Web MVC와 다르게 동작할 때가 종종 있어서 고생할 때가 있다.

이번에 발생한 이슈도 그런 요소 중 하나여서, 간만에 블로그 글을 작성해보고자 한다.

# 발생한 이슈

예를 들어, 아래와 같이 업로드할 파일을 Multipart form data로 전달 받는 Endpoint가 컨트롤러에 있다고 가정하자.

```java
@PostMapping
public Mono<String> upload(@RequestPart("file") Mono<FilePart> file, ServerWebExchange exchange) {
    // 파일명 반환
    return file.map(FilePart::filename);
}
```

만약 이 Endpoint에 요청을 보낼 때, 한글파일명을 가진 파일을 업로드하면 어떻게 동작할까?

당연히 업로드한 파일명이 깨지지 않고 표시될 것이라 예상했으나..

![response](/images/210811_1.png)

깨진 파일명이 반환되는 것을 확인할 수 있다.

# 문제 발생 원인

해당 문제를 디버깅해보니, spring-web의 MultipartParser 클래스에서 아래와 같은 코드를 발견할 수 있었다.

```java
private HttpHeaders parseHeaders() {
    ...
	DataBuffer joined = this.buffers.get(0).factory().join(this.buffers);
	this.buffers.clear();
	String string = joined.toString(StandardCharsets.ISO_8859_1);
	...
}
```

Header를 파싱하는 과정에서 Charset를 ISO-8859-1로 지정하여 인코딩하는 것을 확인할 수 있었다.

참고로 Postman, Insomnia 등의 툴을 사용하여 요청을 보내는 경우, File Part에 붙어있는 Content-Disposition 헤더는 아래와 같다.

![header](/images/210811_2.png)

# 해결책

**한줄 요약 : 해당 이슈는 spring-web 5.3.0 ~ 5.3.5에서만 발생하며, 5.3.6 이상으로 업데이트하면 해결 (spring-boot-starter-webflux 2.4.5 이상)**

처음 생각했던 방법은 파일명이 필요한 로직에서 직접 UTF-8로 인코딩해서 쓰는 것이었다.

그런데 해당 이슈가 다른 환경에 발생하는지도 확인해보고 싶어서 집에 있는 윈도우 PC에 테스트용 프로젝트를 작성하고 구동해본 결과, 정상적으로 깨지지 않은 파일명이 반환되는 것을 보고 깜짝 놀랐다.

확인해본 결과 해당 문제가 발생했던 서비스의 Spring Boot 버전은 2.4.3이었고, 내가 구성한 테스트용 프로젝트는 Spring Boot 버전이 2.5.3이라는 차이점이 존재했다.

위에서 확인했던 `parseHeaders()` 메서드의 소스코드를 확인해본 결과, 아래와 같이 변경된 것을 확인할 수 있었다.

```java
private HttpHeaders parseHeaders() {
	...
	DataBuffer joined = this.buffers.get(0).factory().join(this.buffers);
	this.buffers.clear();
	String string = joined.toString(MultipartParser.this.headersCharset);
	...
}
```

ISO-8859-1을 무조건 사용하도록 구현되어있던 기존과 달리, 별도로 headersCharset을 세팅하여 인코딩에 사용하는 것으로 추측해볼 수 있다.

그렇다면 이 headerCharset을 세팅하는 곳은 어디일까?

`org.springframework.http.codec.multipart` 패키지에 있는 `DefaultPartHttpMessageReader` 클래스를 살펴보자.

Spring 공식 Repository에서 최신 소스를 살펴보면 아래와 같이 Default Charset을 지정하는 코드가 있는 것을 확인할 수 있다.

(코드 링크 : https://github.com/spring-projects/spring-framework/blob/v5.3.9/spring-web/src/main/java/org/springframework/http/codec/multipart/DefaultPartHttpMessageReader.java#L78 )

```java
private Charset headersCharset = StandardCharsets.UTF_8;
```

그리고 해당 클래스의 `read()` 메서드를 확인해보면 headerCharset 필드를 포함하여 parse 호출하는 것을 볼 수 있다.

해당 이슈의 경우 spring-web 5.3.6 버전에서 [수정](https://github.com/spring-projects/spring-framework/releases/tag/v5.3.6)되었는데, 당시 등록되었던 Github 이슈는 [이곳](https://github.com/spring-projects/spring-framework/issues/26736)을 참고하자.

수정이 진행된 [커밋](https://github.com/spring-projects/spring-framework/commit/d83fb099149bd9178ee7d103c6da1ea52152c1cc)을 확인해보면, 5.3.6이라는 버전 주석과 함께 아래와 같은 설명을 확인할 수 있다.

![comment](/images/210811_3.png)

대충 디폴트로 UTF-8을 쓴다는 얘기인데, RFC 7578 문서 링크가 들어있다. 해당 문서를 잠시 살펴보자.

(참고로 링크 텍스트는 5.2라고 써있지만 5.1 항목을 봐야된다)

자세히 읽어보진 않았지만 필요한 내용만 추려본다면, 해석하려는 데이터에 별도로 Charset이 명시되지 않은 경우 UTF-8을 사용하라는 내용으로 보인다.

`spring-boot-starter-webflux` 를 사용하여 의존성을 잡은 경우, `spring-web` 5.3.6 버전을 사용하는 2.4.5 버전 이상으로 올려야하니 참고하자.

해당 버전으로 업데이트 한 뒤, 더이상 문제가 발생하지 않는 것을 확인했다.

# 여기서만 발생하는 문제인가?

## Spring Web MVC

동일한 요청을 Spring Web MVC로 구성하고 테스트해본 결과, 이쪽에서는 의도한대로 동작하고 있었다.

내부적으로 코드를 확인해본 결과, 아래와 같은 내용을 확인할 수 있었다.

* 기존 Spring 환경의 경우, StandardMultipartHttpServletRequest.java → parseRequest 에서 Content-Disposition Header값을 불러올 때 이미 UTF-8로 인코딩된 문자열이 넘어오고 있음
    * Spring Boot에서 제공하는 HttpEncodingAutoConfiguration에서 CharacterEncodingFilter를 세팅할 때 org.springframework.boot.web.servlet.server.Encoding 클래스에서 Charset을 가져와서 설정
    * 여기서 디폴트로 제공하는 설정이 UTF-8임
* MultiPart 파싱 과정에서 (Tomcat 기준) FileItemIteratorImpl → init() 메서드가 multiPartStream에 headerEncoding을 미리 세팅하고, 이후 findNextItem() 내에서 multi.readHeaders()를 호출할 때에는 그 headerEncoding을 가져와서 Content-Disposition String을 인코딩함

## Spring WebFlux 구버전

`spring-web` 5.3.6 버전에서 수정됐음을 확인한 뒤, 아래와 같은 의문점이 생겼다.

"기존에 WebFlux가 존재했던 모든 버전에서 이런 문제가 발생한 것일까?"

정답은 "아니다"였다.

해당 문제와 관련 있는 `DefaultPartHttpMessageReader` 클래스의 경우 `spring-web` 5.3 버전에서 처음 등장했으며, 그 전 버전까지는 Multipart 처리를 `SynchronossPartHttpMessageReader` 클래스가 맡아서 하고 있었다.

그리고 해당 클래스의 경우 UTF-8에 대한 처리가 제대로 되어 있기 때문에, Spring Boot 2.1.0 으로 다운그레이드 후 테스트 해본 결과 정상 동작하는 것을 확인했다.

* Header의 Content-Type에 charset이 명시된 경우 해당 charset을 사용하여 header string을 인코딩하고, 그렇지 않은 경우 default charset으로 UTF-8을 사용함
    * charset 지정 코드 : https://github.com/spring-projects/spring-framework/blob/5.2.x/spring-web/src/main/java/org/springframework/http/codec/multipart/SynchronossPartHttpMessageReader.java#L255
    * 해당 charset을 사용하여 header를 파싱하는 코드 : https://github.com/synchronoss/nio-multipart/blob/master/nio-multipart-parser/src/main/java/org/synchronoss/cloud/nio/multipart/NioMultipartParser.java#L516

이 클래스의 경우 synchronoss에서 만든 `nio-multipart` 라는 라이브러리에 포함되어 있으니, 관심 있으면 [해당 Repo](https://github.com/synchronoss/nio-multipart) 를 참고하자.


# 결론

이슈 대응 과정에서 여러모로 배운 점이 많아 뿌듯했다.

그리고 위에서 언급했던 내용인데, Javadoc 주석에 달린 링크가 실제로는 섹션 5.1인데 5.2로 적혀있는 이슈가 존재한다.

작성 과정에서 발견하여 소소하게 [PR](https://github.com/spring-projects/spring-framework/pull/27260) 을 등록했다. 별 내용은 아니지만 처음 올려본 것이라 만족!
