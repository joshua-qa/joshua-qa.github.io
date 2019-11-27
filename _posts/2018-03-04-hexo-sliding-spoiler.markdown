---
layout: post
title: "[hexo] 스포일러 방지기능 (hide/show) 넣기"
date: 2018-03-04 16:35:22
tags: hexo
categories: hexo
---

### hexo-sliding-spoiler

네이버 블로그나 티스토리 등에서 많이 쓰는 기능 중에, `hide/show` 기능이 있다.

너무 길어질 수 있는 설명이나 스포일러가 될 수 있는 내용을 가려두고, show 버튼 같은 것을 눌렀을 때 보이도록 하는 것인데 hexo에서도 사용가능한지 찾아봤다.



다행히 누군가 이미 만들어놓은 [hexo-sliding-spolier](https://www.npmjs.com/package/hexo-sliding-spoiler) 라는 플러그인이 있어서 설치했고, 사용해보니 매우 만족스럽다.

```
npm install hexo-sliding-spoiler --save
```

사용 방법은 작성하려는 md파일에 다음과 같이 넣어주면 된다.

```
숨길 내용물
```



링크를 참고하면 DEMO gif가 있으니 참고하면 된다.

