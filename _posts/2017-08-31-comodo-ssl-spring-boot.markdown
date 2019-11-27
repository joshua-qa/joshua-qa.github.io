---
layout: post
title: GOGETSSL에서 구매한 COMODO Positive SSL을 Spring Boot에 적용하기
tags:
  - SSL
  - HTTPS
  - Springboot
categories:
  - Dev
date: 2017-08-31 14:35:52
---

#### COMODO Positive SSL 인증서를 Spring Boot에 적용하기

이 포스팅을 쓰기까지 많은 시간이 걸렸습니다.

SSL 인증서에 대한 개념이 부족한 상태에서 접근하여 발생한 문제가 많은데, 드디어 해결을 했고 어떻게 했는지 작성해보려고 합니다.

#### 무엇을 하려고 했는가?

Spring Boot로 작성한 LINE 메신저 BOT을 개인 서버에 배포했는데, 실제로 이 것을 사용하려면 https로 통신할 수 있어야 합니다.

>   ### Do I need to have a server to create a LINE bot?
>
>   Yes, you need to have a server with a SSL certificate to communicate with the LINE Platform. You can get free SSL certifcates using a service such as Let's Encrypt or StartSSL. It's also possible to try out the LINE Bot SDK using Heroku or another similar cloud service.
>
>   ### Do I need to use SSL on my server?
>
>   Yes, you have to use SSL on your server. Also note that self-signed certificates are not acceptable. If you encounter issues related to your SSL configuration, you should check that your SSL certificate chain is complete and that your intermediate certificates are correctly installed on your server.

[참고한 링크](https://github.com/line/line-bot-faq#do-i-need-to-have-a-server-to-create-a-line-bot)

Apache2에 SSL 인증서를 적용하는 일은 쉬웠지만, Spring Boot나 Tomcat에는 적용해본 적이 없어서 어렵고 생소하게 느껴졌습니다.

#### 왜 어렵게 느꼈는가?

저는 [GOGETSSL](https://www.gogetssl.com/) 에서 COMODO Positive SSL 인증서를 구매했고, 메일로 다음과 같은 파일이 발송되었습니다.

*   `(도메인명).crt`
*   `(도메인명).ca-bundle`

그 외에 가지고 있는 파일은 이미 개인 서버에서 만들어놓은 key 파일이었습니다.

Spring Boot나 Tomcat에 SSL 인증서를 적용하려면 해당 파일을 조합하여 Java keystore(jks) 파일을 생성하는 것이 필요했고, 여기서부터 막히기 시작했어요. SSL 인증서에 대한 개념이 부족하다는 걸 바로 느꼈습니다.

1.  ca-bundle 파일이 무슨 역할을 하는 것인지 좀처럼 이해할 수 없었다.
2.  수십개의 블로그 및 사이트를 찾아서 메뉴얼을 보고 따라해보려고 했으나, 막상 jks 파일을 만들어서 적용하고 보면 오류가 발생했다.

#### 어떻게 해결했는가?

1.ca-bundle 파일이 무슨 역할을 하는지 공부했습니다.

>   CA bundle is a file that contains root and intermediate certificates. The certificate issued for your domain constitutes the certificates’ chain with a CA bundle.

루트 인증서 및 중간 인증서 내용을 가지고 있는 것이 ca-bundle 파일이었습니다.



2.crt 파일과 ca-bundle 파일을 합치고, 이를 key와 조합하여 jks로 만들었습니다.

마지막으로 참고를 했던 [페이지](https://www.securesign.kr/guides/SSL-Certificate-Convert-Format)에서, crt 및 key를 조합하여 pfx 파일을 만들고 이것을 jks로 변환하는 과정이 설명되어 있었습니다.


이 작업을 위해서는 우선, `(도메인명).crt` 파일과 `(도메인명).ca-bundle` 파일을 합친 crt 파일이 필요합니다.

```
cat mitsuha_me.crt mitsuha_me.ca-bundle > mitsuha.crt
```

다음으로, openssl과 keytool을 이용하여 pfx 생성 -> jks로 변환 과정을 거치면 됩니다.

```
openssl pkcs12 -export -in (위에서 만든 crt파일 명).crt -inkey private.key -out cert.pfx

keytool -importkeystore -srckeystore cert.pfx -srcstoretype pkcs12 -destkeystore cert.jks -deststoretype jks
```



3.TrustStore jks를 만들었습니다.

TrustStore에 대해서는 다음 링크들을 참고했습니다.

*   [KeyStore와 TrustStore란?](http://btsweet.blogspot.kr/2014/06/tls-ssl.html)
*   [[SSL] Spring Boot를 이용해 HTTPS 연동하기](http://seongtak-yoon.tistory.com/10)

```
keytool -export -alias (alias 이름) -keystore (keystore 파일명) -rfc -file server.cer

keytool -import -alias (alias 이름) -file server.cer -keystore trustcert.jks
```

여기서 alias 이름은 keystore에 부여된 별칭인데, 특별히 뭔가 하지 않았다면 `1`이나 `mykey` 등의 별칭이 붙어있을겁니다. 이 alias 이름은 Spring Boot 설정을 할 때도 필요합니다.



4.Spring Boot에 적용 후 테스트 했습니다.

`application.properties` 파일을 열어 필요한 정보를 추가해줍니다.

```
server.ssl.key-store= keystore(jks) 파일 경로
server.ssl.key-store-password= keystore 파일 만들 때 설정한 비밀번호
server.ssl.key-password= 키 비밀번호
server.ssl.key-alias= alias 이름

server.ssl.trust-store= truststore 파일 경로
server.ssl.trust-store-password= truststore 파일 만들 때 설정한 비밀번호
```

배포 후 테스트 해본 결과, 다음과 같이 인증서 적용이 되었음을 확인할 수 있었습니다.

![COMODO SSL](/images/comodossl.png)



#### 이 과정에서 어떤 문제가 있었는가?

얼핏 보면 깔끔하게 진행된 것 같지만, 중간에 두 시간정도 문제가 있었습니다.

SSL 인증서 적용을 몇몇 브라우저(IE, Chrome, Firefox)에서 확인했고 이제 LINE 개발자 페이지에서 Webhook URL 테스트만 남았는데, 놀랍게도 다음과 같은 오류를 뱉으며 실패했습니다.

```
SSL certificate is invalid.
```

솔직히 이 생각 밖에 들지 않았습니다. '브라우저들이 괜찮다고 해준 인증서가 여기서는 안된다고?'

어디서 발생했는지도 모르는 문제점을 찾아서 각종 시도를 해봤는데, 원인은 의외로 간단했습니다.

crt 파일과 key 파일을 합쳐서 pfx를 만들고, 이걸 jks로 변환했는데.. 처음 시도할 때는 여기서 ca-bundle을 합치는 과정을 빼먹었던 것입니다. ca-bundle 없이 `(도메인명).crt`와 private key 파일만 합치기!

그럼에도 불구하고 브라우저는 인증서가 멀쩡하다고 얘기해준 것이고, '아 이제 잘 동작하겠구나!' 로 생각해버린 저의 잘못이었던거죠.

crt파일과 ca-bundle을 합치는 작업까지 해준 뒤에는 이상없이 동작하는 것을 확인하였습니다.



#### 교훈

*   메뉴얼에서 설명하는 용어가 뭔지 모르겠으면 기본 개념부터 찾아보려고 노력하자
*   잘 된 것 같은데 문제가 있다면 내가 어떤 실수를 저질렀는지 단계별로 고민해보자!
