---
layout: post
title: "[Spring WebFlux] 스프링 버전 업그레이드 (5.3.13, 5.3.14) 이후 파일 업로드 퍼미션 오류 발생 해결하기"
date: 2022-01-01 17:48:21
tags: 
 - Java
 - Spring
 - WebFlux
categories: Dev
---

# TL;DR
* Spring-web 5.3.13/5.3.14 버전 (Spring Boot 버전 기준 2.4.13, 2.5.7, 2.6.0) 기준으로 FilePart의 `transferTo()` 메서드는 주의해서 사용할 필요가 있다.
    * 특히 maxInMemorySize 설정값을 초과하는 파일을 업로드하는 경우, `transferTo()` 메서드를 사용했을 때 의도하지 않은 퍼미션 값으로 저장될 수 있다.
    * 파일을 저장하고 싶다면 `DataBufferUtils.write(Publisher<DataBuffer> source, Path destination, OpenOption... options)`를 사용해보자.
* 스프링 프레임워크 6 버전에 멀티파트 임시파일에 대한 자동 정리 기능을 넣으려고 준비중인 것으로 보이며, 아직까진 해당 기능이 없기 때문에 임시파일 정리를 신경쓸 필요가 있다.

# 발생 이슈
Spring Boot 2.5.5 / WebFlux 기반으로 운영되고 있던 API 서버를 최근 Spring Boot 2.5.8로 버전업했다.

배포한 뒤로 한동안 문제가 없어서 괜찮은줄 알았는데, 간헐적으로 동작에 이상이 있다는 제보를 받았다.

구체적인 증상을 기술할 수는 없으나, 해당 서버를 통해 업로드 하는 파일 중 일부 파일의 퍼미션만 상이한 것이 문제였다.

똑같은 과정으로 업로드하는 파일인데 어째서 일정 확률로 퍼미션에 문제가 생기는가? 구체적인 케이스를 파악해본 결과, 특정 용량을 초과하는 파일을 멀티파트로 업로드했을 때 발생하는 이슈였다.

# 원인
우선 [이슈](https://github.com/spring-projects/spring-framework/issues/27613) 및 [PR](https://github.com/spring-projects/spring-framework/commit/0c7e0002504f728d3ca3e182406d19ed3c4c9973)을 읽어보자.

이 이슈를 이해하려면 WebFlux 기반의 서비스에서 `multipart/form-data`를 이용하여 요청할 때, body에 포함된 파일을 어떻게 처리하는지 살펴봐야한다.

해당 이슈를 설명하기 위해, `spring-web 5.3.12 까지의 동작` 과 `spring-web 5.3.13, 5.3.14의 동작` 을 나누어서 설명하겠다.

## spring-web 5.3 ~ 5.3.12까지의 동작
`DefaultParts` 클래스 내부에 `DefaultFilePart` 내부 클래스가 존재하며, Multipart body로 들어온 파트중 파일에 해당하는 파트를 해당 객체로 변환한다.

`DefaultFilePart` 클래스의 경우 `transferTo()` 메서드를 사용하여 특정 Path에 파일을 저장할 수 있도록 지원하는데, 해당 메서드는 `DataBufferUtils.write()` 유틸 메서드를 사용하여 해당 기능을 구현하고 있다.

5.3.12 버전까지는 업로드 시도한 파일 용량과 관계없이 `DefaultFilePart` 객체로 변환하고 있다.

## spring-web 5.3.13 ~ 5.3.14의 동작
`org.springframework.http.codec.multipart` 패키지에 대한 리팩토링이 진행되었고, 아래와 같은 변화가 생겼다.

* `DefaultFilePart` 클래스의 부모인 `DefaultPart` 클래스의 변화
    * 기존에는 content 프로퍼티가 `Flux<DataBuffer>` 타입이었으나, Content 인터페이스를 신설하고 content 프로퍼티를 Content 타입으로 변경
    * `Content` 인터페이스의 경우 `FluxContent`, `FileContent` 두개의 구현체가 있으며 아래와 같은 차이점이 있다.
        * `FluxContent` : `Flux<DataBuffer>`를 갖는 객체
            * 요청으로 들어온 파일 용량이 maxInMemorySize를 초과하지 않는 경우 임시파일을 생성하지 않고 해당 객체로 관리
        * `FileContent` : `Path`, `Scheduler`를 갖는 객체
            * 요청으로 들어온 파일 용량이 maxInMemorySize를 초과하는 경우, 해당 환경의 임시폴더에 임시파일로 저장한 뒤 해당 객체로 관리
            * Path의 경우 해당 임시파일의 경로이며, Scheduler의 경우 Blocking Operation (여기서는 File I/O 관련 메서드들)를 처리하기 위해 할당할 스케줄러(스레드) 정보를 갖는다.
                * 참고로 이 스케줄러는 `DefaultPartHttpMessageReader` 클래스에서 [확인](https://github.com/spring-projects/spring-framework/blob/main/spring-web/src/main/java/org/springframework/http/codec/multipart/DefaultPartHttpMessageReader.java#L74)할 수 있다. `blockingOperationScheduler` 프로퍼티를 확인해보면 `Schedulers.boundedElastic()`을 사용하고 있다.
* maxInMemorySize (default 256kb)를 초과하는 파일을 Multipart body에 담아 요청한 경우
    * `FileContent` 객체로 관리되며, `transferTo()` 메서드를 호출하면 `Files.copy()` 메서드를 별도 스케줄러에 할당하여 파일 복사를 실행한다.
* 초과하지 않는 파일을 담아 요청한 경우
    * `FluxContent` 객체로 관리되며, `transferTo()` 메서드를 호출하면 `DataBufferUtils.write()` 메서드를 사용하여 AsynchronousFileChannel을 열고 파일을 전송한다.

## 변경된 코드로 인해 발생하는 문제

내 업무용 맥북 및 회사 서비스가 올라가는 서버를 기준으로, maxInMemorySize를 초과하는 파일을 첨부하여 멀티파트 요청을 하면 지정된 임시폴더 하위에 `spring-multipart-92384579682759843` 같은 형태로 폴더가 만들어지고 그 하위에 해당 file part에 대한 임시파일이 생성된다.

![image](/images/multipart-temp-location.jpg)

위 스크린샷을 보면 알겠지만, 임시 파일은 600 퍼미션으로 생성된다. 이는 리눅스 서버 환경에서도 동일하다.

이렇게 저장된 파일을 별도의 퍼미션 설정 없이 `Files.copy()` 호출해서 그대로 복사하고 있으니, 복사된 위치의 파일도 똑같이 600 퍼미션으로 들어가는 것이다.

기존 5.3.12에서는 복사된 파일의 퍼미션을 확인했을 때 644로 나오는 것을 확인하였다.

* (5.3.12 까지) DefaultFilePart->transferTo() 사용 결과
    * 복사된 파일의 퍼미션 : 644
* (5.3.13 버전 이후) FluxContent->transferTo() 사용 결과
    * 복사된 파일의 퍼미션 : 644
* (5.3.13 버전 이후) FileContent->transferTo() 사용 결과
    * 복사된 파일의 퍼미션 : 600

참고로 `DataBufferUtils.write()` 메서드와 `Files.copy()` 메서드는 완전히 다른 메서드이다. 어째서 이번 수정 코드에 `Files.copy()`를 적용하였는지 개인적으로는 매우 의문이 든다.

## 기존에는 이런 문제가 없었는가?
퍼미션 관련해서는 불편함을 겪을 일이 없었다. 대신 다른 문제가 존재한다.

5.3.12 시절의 코드를 잠시 살펴보자.

![image](/images/spring-5312-defaultpart.png)

`transferTo()` 메서드를 호출하는 경우, 첨부 파일에 대한 content(DataBuffer)를 가져와서 지정된 경로에 write 처리한다.

첨부 파일 데이터를 읽어들여서 만든 DataBuffer를 이용하여 파일을 생성하는 것이므로, 임시 폴더에 생성된 임시 파일의 퍼미션이 어땠는지와 소유권은 어땠는지 신경쓸 필요가 없다.

그렇다면 기존 로직의 다른 문제는 어떤 것인가?

**바로 `임시 파일을 삭제하는 로직이 없다'는 것이다.**

자동으로 정리해주지도 않고, Content나 Part 관련된 객체에서 사용 끝낸 임시 파일을 지워주는 로직도 없다.

아래 스크린샷은 내가 개인적으로 작성한 WebFlux 기반 미니 프로젝트로 파일 업로드 테스트를 한 뒤, 어플리케이션을 종료한 결과이다.

![image](/images/multipart-tempfile-not-delete.png)

정상적으로 업로드가 완료되었고 해당 요청에 대한 Response 송신도 완료되었으니 close 처리가 되면서 임시파일도 자연스럽게 정리될거라 믿었지만, 실제로는 그렇지 않다.

어플리케이션을 종료한 이후에도 해당 임시 폴더에 접근 가능하며 파일도 정상적으로 남은 것을 확인할 수 있었다.

해당 이슈를 해결하기 위해 5.3.13 부터는 파일을 정리할 수 있도록 `delete()` 메서드가 [추가](https://github.com/spring-projects/spring-framework/blob/v5.3.13/spring-web/src/main/java/org/springframework/http/codec/multipart/DefaultParts.java#L316) 되었고, 스프링 프로젝트 팀에서는 6.0 버전에서 멀티파트 임시 파일에 대한 자동 정리 옵션을 만들겠다고 답변했다. (https://github.com/spring-projects/spring-framework/issues/27613#issuecomment-957720481)


# 해결책

문제점을 파악하였으니 이제 해결책을 알아보자.

이 글을 작성한 이유는 `transferTo()` 메서드를 사용했을 때 의도하지 않은 퍼미션 값으로 파일 저장되는 문제를 알아보기 위함이었으나, 위에 기재했듯이 임시 파일 정리에 대한 이슈도 존재한다.

두가지 모두 해결책을 적어보겠다.

## 변경된 transferTo() 메서드로 인한 퍼미션 문제 해결하기

더이상 `transferTo()` 를 사용하지 않고, 기존 로직과 동일한 코드를 작성하여 해결하였다.

```java
// AS-IS
private Mono<Void> writeFile(FilePart part) {
    return part.transferTo(Paths.get("./test/testfile.txt"));
}

// TO-BE
private Mono<Void> writeFile(FilePart part) {
    return DataBufferUtils.write(part.content(), Paths.get("./test/testfile.txt"));
}
```

## 파일 저장 후 멀티파트 임시 파일 정리하기

5.3.12 버전까지는 임시 파일을 정리하는 기능이 없으니 알아서 구현해야한다.

5.3.13 버전 이상으로 올릴 수 있다면 `Part::delete` 메서드를 사용하자.

```java
private Mono<Void> writeFile(FilePart part) {
    return DataBufferUtils.write(part.content(), Paths.get("./test/testfile.txt")).then(part.delete());
}
```

* `part.delete()` 호출 직전
![before](/images/multipart-tempfile-delete-before.png)
* `part.delete()` 호출 이후
![after](/images/multipart-tempfile-delete-after.png)


# 기타
멀티파트 임시 파일 및 임시 파일에 대한 FileContent 객체가 생성되는 것이 불편한 사람들도 있을 것이다.

몇몇 메서드 호출 과정에서 File I/O가 발생하기 때문이다. 게다가 별도로 스케줄러를 할당해야하는 Blocking feature이기도 하다.

그렇다면 이 상황 자체를 회피할 수 있는 방법이 있을까? 답은 간단하다.

스프링 프로젝트 팀에서는 maxInMemorySize 설정을 증가시켜서 임시 파일 생성을 회피할 수 있다고 알려주고 있다.  
(https://github.com/spring-projects/spring-framework/issues/27633#issuecomment-959128482)

가령 서비스 정책 상으로 업로드 최대 제한이 5MB라고 한다면, 해당 설정을 5MB보다 크게 지정해서 사용자들이 업로드하는 파일은 무조건 임시 파일 생성 없이 `FluxContent` 객체로 생성 및 관리되도록 제어할 수 있는 것이다.

개인적으로 이 방법은 조금 위험한 방법이라고 생각한다.

특정 상황에서는 좋은 방법이 될 수 있겠으나, 서버 여유 자원이 넉넉하지 않은 상황에서는 문제가 될 수 있기 때문이다.

각자의 상황이 다른 법이니 꼭 필요한지 검토해보고 사용하자.

# 마무리
이번에 log4j2 이슈 및 logback 이슈를 대응하며, 갑작스럽게 스프링 부트 버전을 업그레이드한 곳이 많을 것이다.

마이너 버전 업그레이드는 솔직히 큰 문제가 없을줄 알았는데 이런 식으로 문제를 겪고나니 릴리즈 노트를 더욱 꼼꼼히 읽어봐야겠다는 생각이 들었다.

원래 이 글은 12월 30일에 작성하여 게시할 예정이었으나, 좀 더 확실하게 조사하여 올리고 싶었기 때문에 시간이 더 걸렸다.

앞으로도 WebFlux 관련하여 공유할만한 이슈가 있는 경우 블로그에 작성해보도록 하겠다. :)
