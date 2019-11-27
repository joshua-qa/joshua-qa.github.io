---
layout: post
title: "[Android] Toast 중복 현상 해결"
date: 2017-02-10 12:10:33
tags:
- android
- toast
- issue
- java
categories:
- Dev
---

## 발생한 상황
* 서비스 중인 안드로이드 앱에서, 짧은 시간동안 여러 개의 토스트가 노출되는 상황이 있었는데 각각 설정한 duration만큼 노출 시간을 잡아 먹어서 토스트 메시지가 너무 오랫동안 보이는 문제가 있었다.
* 새로운 토스트가 노출되어야 하는 상황일 때, 기존에 떠있던 토스트를 사라지게 한 뒤에 바로 띄우고 싶었다.

## 해결 방법
* 기존의 코드는 다음과 같이 구현되어 있었다.
```java
Toast.makeText(act, message, Toast.LENGTH_SHORT).show();
```

변경한 코드는 클래스에 mToast라는 Toast 객체를 만들어주고, makeText와 show를 분리했다. 그리고 이미 떠있는 토스트가 있는 경우 cancel을 해주도록 처리했다.

```java
public static Toast mToast;
...
...
public void shortMessage(final String message) {
    Runnable r = new Runnable() {
        @Override
        public void run() {
            Toast.makeText(act, message, Toast.LENGTH_SHORT).show();
            if(mToast != null) mToast.cancel();
                mToast = Toast.makeText(act, message, Toast.LENGTH_SHORT);
                mToast.show();
            }
        };
        runOnMainThread(r);
    }
}
```



변경해주고 나서 확인해보니, 의도한대로 동작하는 것을 확인했다 :)

참고 사이트 : http://stackoverflow.com/questions/12922516/how-to-prevent-multiple-toast-overlaps

http://stackoverflow.com/questions/10070108/can-i-cancel-previous-toast-when-i-want-to-show-an-other-toast
