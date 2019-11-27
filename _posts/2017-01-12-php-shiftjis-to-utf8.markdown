---
layout: post
title: SHIFT-JIS를 UTF-8로 바꾸기 (PHP)
date: 2017-01-12 17:22:08
tags:
- info
- php
- shift-jis
- utf-8
categories:
- Dev
---

해외 사이트에 있는 데이터를 ajax로 크롤링해서 php에 form 전송을 했는데, 받아온 json값을 확인해보니 인코딩이 깨져있었다.

내 서버가 인코딩 세팅이 안되어있나 싶어서 확인해보니 UTF-8로 잘 되어있었고, 해당 사이트를 확인해본 결과 SHIFT-JIS를 사용하고 있었다.

일단 iconv를 사용하여 컨버팅을 했더니 성공!

```php
$json = $_POST['Detail'];
$converted_string = iconv("shift-jis","utf-8",$json);
$result = str_replace("¥", "", $converted_string);
```

세번째 줄은 편의상 넣었다. 받아온 json 값을 컨버팅해주고 나니 `¥` 문자가 많이 들어있어서 제외 처리..
