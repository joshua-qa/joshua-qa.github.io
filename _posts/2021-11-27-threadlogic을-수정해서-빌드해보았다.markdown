---
layout: post
title: "[Java] ThreadLogic를 수정해서 빌드해보았다"
date: 2021-11-27 12:08:35
tags: 
 - Java
categories: Dev
---

최근, 회사에서 쓰레드 덤프 분석을 위해 ThreadLogic을 쓰고 있다.

fastthread.io, TDA, IBM TMDA 등의 툴을 같이 써보기도 했으나, 아무래도 ThreadLogic이 분석 기능 및 권고 사항 알려주는 기능이 잘되어있어서 이쪽을 1순위로 활용하게 되었다.

업데이트도 멈췄고, 인터페이스가 조금 오래된 감이 있지만 큰 불편함 없이 쓰고 있었는데 며칠 전 한 가지 단점을 발견했다.

팀 내 공유할 문서 작성을 하는 도중 ThreadLogic에 출력된 권고 사항을 일부 복사할 일이 있었는데, 아무리 해도 텍스트 복사가 동작하지 않는 것이었다.

참고로 나는 업무용 랩탑으로 맥북을 쓰고 있는데, Command-C 단축키도 동작하지 않고 프로그램 메뉴에도 `[편집]` 메뉴가 없다보니 결국 일일히 타이핑하게 되었다.

![드래그한 텍스트 복사 불가](/images/threadlogic1.png)

보통 이런 일이 생긴다면 해당 Github Repo에 이슈를 등록해서 수정해달라고 하는게 우선이겠지만.. ThreadLogic의 경우 마지막 업데이트가 2018년이었고 어떠한 이슈나 PR도 등록된 적이 없는 것을 확인했기 때문에 내가 고쳐 쓰는게 빠르겠다는 결론이 나왔다.

다행히 어렵지 않게 수정할 수 있었고, 그 과정에서 알게된 것들을 적어보려고 한다.

## 원인 파악

우선, 아래와 같은 의문을 해소할 필요가 있었다.

1) ThreadLogic은 아예 드래그한 텍스트의 복사 기능을 지원하지 않는건가?
2) 그렇지 않다면, Command-C 단축키를 사용한 텍스트 복사만 지원하지 않는건가?

코드를 확인해본 결과, 첫번째 의문은 금방 알 수 있었다.

따로 복사를 방지하는 코드가 삽입되어 있지 않았고, `JEditTextArea` 클래스에는 아예 `copy()`, `paste()` 메서드까지 구현되어 있는 것을 확인했기 때문이다.

해당 메서드를 호출하는 PopupMenu 클래스를 확인해보니, 마우스 우클릭으로 호출하는 팝업 메뉴에 대한 구현부로 추정되는 코드를 발견할 수 있었다.  
-> 정작 해당 팝업 메뉴를 호출하는 방법은 찾지 못했다. 프로그램 내에서 마우스 우클릭을 이곳 저곳에 시도해봐도, `ThreadLogic` 클래스의 `actionPerformed` 메서드에 있는 메뉴가 나올뿐...

다행히 발견한 코드에서 Ctrl-C 단축키에 대한 코드를 확인할 수 있었다.

```java
copyMenuItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_C, KeyEvent.CTRL_DOWN_MASK));
```

두번째 의문도 해소되었고, Ctrl-C 단축키를 사용하면 텍스트 복사를 할 수 있다는 것도 알게되었다.

다만 내가 복사 기능을 사용하려던 컴포넌트는 `JEditorPane` 이었으며 이쪽과는 크게 상관 없는 것 같다는 것을 나중에 깨달았지만 코드 분석을 한 덕분에 Ctrl-C 지원 여부를 알게 된 것만으로도 큰 수확이었다.

## 수정 작업

사실 상식적으로 생각해보면, 애초에 이런 코드 분석을 할 필요 없이 Ctrl-C 단축키를 한 번이라도 눌러보면 되는 일이었다. 

**그러나 누가 MacOS 환경에서 Command-C 대신 Ctrl-C 단축키를 사용하겠는가?**

당연히 생각하지 못했고.. 이렇게 된 김에 Command-C 단축키를 지원하도록 코드를 수정하자! 라고 마음 먹었다.

위에 언급한 내용대로, 내가 복사 기능을 사용하고 싶었던 컴포넌트는 알고보니 `JEditorPane` 이었다. 그런데 처음에는 이걸 알지 못했고, 내가 복사하고 싶은 텍스트가 어떤 컴포넌트를 사용해서 출력하는 것인지 분석할 필요가 있었다.

ThreadLogic을 처음 실행하면 `Tip of the day`, `Actions`, `Help` 요소를 가지고 있는 페이지가 나온다. 

아마 이 페이지가 쓰레드 덤프 권고사항을 보여주는 화면과 동일한 컴포넌트를 사용할 것으로 예상되어서 찾아본 결과 `ThreadLogic` 클래스에서 해당 코드를 발견했다.

```java
private JEditorPane htmlPane;

....(중략)....

// Create the HTML viewing pane.
InputStream is = ThreadLogic.class.getResourceAsStream("doc/welcome.html");

....(중략)....

htmlPane = new JEditorPane();
String welcomeText = parseWelcomeURL(is);
htmlPane.setContentType("text/html");
htmlPane.setText(welcomeText);
toolTip = htmlPane.createToolTip();
```

`welcome.html` 페이지를 불러온 뒤, 해당 컴포넌트에 띄우는 것을 확인했다.

이후 스크롤을 내려보니 `htmlPane.addHyperlinkListener` 밑으로 HyperlinkListener를 구현하는 코드가 있었고, 각 기능 별로 해당 컴포넌트를 재활용하여 출력해야하는 페이지들을 띄우는 것으로 추정할 수 있었다.

그렇다면 이제 해당 컴포넌트에 대하여 Command-C 단축키를 매핑해야되는데, 이 내용에 대해서는 아래 문서를 참고하여 진행할 수 있었다.

http://mirror.informatimago.com/next/developer.apple.com/ja/technotes/tn2042.html

특정 플랫폼에 국한되지 않고, 여러 플랫폼에 유효한 방법으로 Ctrl (MacOS에서는 Command) 마스크를 구현하기 위해서는 `Toolkit.getDefaultToolkit().getMenuShortcutKeyMask()` 메서드를 호출하면 된다고 나와있다.

어떻게 작업해야되는지 알았으니, 이제 아래와 같은 코드를 작성해보자. 위에 언급했던 코드가 들어있는 ThreadLogic->init 메서드 끝 부분에 코드를 넣어주었다.

```java
htmlPane.getInputMap().put(KeyStroke.getKeyStroke(KeyEvent.VK_C, Toolkit.getDefaultToolkit().getMenuShortcutKeyMask()), new DefaultEditorKit.CopyAction());
```

해당 코드를 삽입하고 빌드를 해본 결과, 각각의 페이지에서 Command-C 단축키를 이용한 텍스트 복사가 가능한 것을 확인했다!

프로젝트를 Fork 떠온 뒤에 내가 작성한 코드도 커밋했으니, 이제 Jar 파일을 빌드해보자.

## 빌드 하기

해당 프로젝트의 경우 Ant로 구성되어있었다. 그동안 Maven과 Gradle만 써보았기 때문에 Ant는 조금 생소했는데, `build.xml`을 열어보니 그리 어렵지 않게 느껴졌다.

인텔리제이는 기본적으로 Ant 플러그인을 지원하고 있는데, 우측의 Ant 탭에서 Task 목록에 있는 jar를 클릭한 상태로 `Alt+Enter` 를 누르면 `Build File Properties` 팝업을 띄울 수 있다.

프로퍼티를 따로 넣어주지 않아도 빌드는 가능하다. 그렇지만 ThreadLogic-{Revision}.jar 이라는 이름으로 빌드가 진행되며, 버전 정보가 없으니 별로 깔끔해보이지 않아서 여기에 `revision` 프로퍼티를 넣어주었다.

마지막 버전이 `2.5.2`였기 때문에 나는 `2.5.2.1`로 지정하고 빌드를 진행했다.

![프로퍼티 수정](/images/threadlogic2.png)

어차피 나 혼자 빌드해서 사용할 것이었기 때문에 구체적인 버저닝 규칙은 신경 쓰지 않았다는 점 양해 부탁드린다.

재밌는 점이 있었다면 MacOS에서 빌드 할 때는 별 이상없이 빌드 진행되던 것이 윈도우에서 에러가 난다는 것이었다.

그 원인은, 빌드 과정에서 실행하는 스크립트로 `git-revision.sh` 파일이 있는데 이걸 윈도우에서 실행하지 못해서 발생하는 문제였다.

어차피 이 것도 빌드 과정에서 필수는 아니라고 생각했기 때문에 윈도우로 빌드할 때는 해당 Task를 제외하고 진행해주었다.

```xml
// AS-IS
<target depends="load-git-revinfo,build-subprojects,build-project,jar" name="build" />
// TO-BE
<target depends="build-subprojects,build-project,jar" name="build" />
```

## 해당 빌드 (JAR 파일) 다운받기

[링크](https://github.com/joshua-qa/threadlogic/releases/tag/v2.5.2.1)

따로 확인해보진 않았지만, 만일 문제 되는 요소가 있다면 삭제하고 혼자서 쓸 생각이다.

## 결론

매우 사소한 문제점이었지만, 스스로 해결할 수 있어서 조금 뿌듯했다.

2년 전에도 인텔리제이의 `HighlightBracketPair` 플러그인이 PHP를 지원하지 않는 문제점이 있어서 개인적으로 수정해서 썼는데 오랜만에 그 때 생각이 나서 좋았다.

이번에는 복사 기능에 대한 단축키 지원만 수정했으나, 앞으로 기회가 된다면 이런 식으로 수정해보는 기회를 더 가져야겠다.

**ThreadLogic을 MacOS에서 사용할 때 나와 동일한 이유로 불편함을 느꼈거나, 자신이 사용하는 Java Swing 기반의 프로그램이 동일한 문제점을 가지고 있어서 수정하고 싶다면 이 글이 도움 될 수 있을 것 같다.**

